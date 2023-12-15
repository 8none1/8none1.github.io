# Reverse engineering Bluetooth LE LED light controllers, or, How I Bricked My Christmas Lights

If it talks Bluetooth LE and it's got an app then it can, and indeed should, be connected up to my home automation system.

I've spent quite a lot of time reverse engineering various cheap and cheerful LED light strips in order to automate them.  The process is largely the same each time, and I quite enjoy it.  Last week I got the cheapest lights I've ever found [£2.38 plus tax for a Bluetooth LE controlled 5M non-addressable strip](https://www.aliexpress.com/item/1005005485885067.html) hooked up to Home Assistant in a few hours.
<https://github.com/8none1/bj_led>

I've had another set of addressable lights sat on my desk for a while and when I was decorating my office for Christmas I thought I'd spend a little bit of time getting them hooked up to Home Assistant using the BJ_LED code as a template.  Should be pretty easy, right?  Well, yes.  But also no.
These lights are a 10M long string of addressable LEDs and are controlled by the "iDeal LED" app.  The app is quite full featured and works well enough. The LEDs are, I think, WS2812 or similar.  I am, or rather was, quite pleased with these lights.  You can [get them on AliExpress](https://www.aliexpress.com/item/1005004829475855.html) of course.

![](/wp-content/uploads/2023/12/aliex_lights.jpg){:class="img-responsive"}

There now follows a cautionary tale.  A lot of the details have been left out for brevity, but there are no secrets here, more instructions are readily available on the web.  I appreciate that this is a bit "draw the rest of the owl", but hopefully some of the links will give you a place to start if you're interested in reverse engineering your own LED lights.

## Step 1. The bytes over the wire

The first step towards controlling the devices from your own software is to look at the bytes being sent over Bluetooth to the device from the app.  Very often the lights talk a simple protocol consisting of a number of bytes forming a header, some command bytes (turn on, turn off, change colour, etc) and a footer which is sometimes a checksum and often ignored.

Android makes this really easy for you.  On an Android device with developer mode enabled install the app for your lights.

On your computer, install [adb](https://developer.android.com/tools/adb).

On Android, go in to the developer settings and enable `Bluetooth HCI snoop`. This will log the Bluetooth bytes to a file which can be understood by [Wireshark](https://www.wireshark.org/).

Open the lights app and do something.  I suggest you start simple; turn the lights on and off five times.

Use `adb` to copy the logs to your computer. e.g. `adb pull sdcard/btsnoop_hci.log .`

Open that file up in Wireshark and you can see the exact bytes being sent to the device.  If you look at the values being sent you should be able to spot a pattern.  You're likely to find that for each time you turned the lights on or off a series of bytes was sent over the wire, and one of those byes would alternate between two values, probably a `1` and a `0`.  Representing on and off.  A useful Wireshark filter to apply would be something like:

```text
bluetooth.dst == ff:ff:ff:ff:ff:ff && btatt.opcode.method==0x12
```

Change MAC address to be the MAC of your lights.  `btatt.opcode.method==0x12` is a write from the Android device to the lights.

Congratulations, you are now a reverse engineer!

==Pro-tip:  You can speed things up a bit by using [tshark](https://tshark.dev/) instead of Wireshark.  What you really care about is the values being written to the LED controller.  `tshark -r <filename> -T fields -e btatt.value` will dump the payload to the terminal for easy interrogation.==

Sometimes your bytes will look like this:

```text
69 96 02 01 01
69 96 02 01 00
69 96 02 01 01
69 96 02 01 00
69 96 02 01 01
69 96 02 01 00
69 96 02 01 01
69 96 02 01 00
```

On, off, on, off, on, off, on, off.

Sometimes your bytes will look like this:

```text
84 dd 50 42 37 41 50 89 7a c8 2f 39 11 09 68 a8
79 d1 db a4 09 19 c2 46 a8 58 0a e7 d1 1b 78 84
84 dd 50 42 37 41 50 89 7a c8 2f 39 11 09 68 a8
79 d1 db a4 09 19 c2 46 a8 58 0a e7 d1 1b 78 84
84 dd 50 42 37 41 50 89 7a c8 2f 39 11 09 68 a8
79 d1 db a4 09 19 c2 46 a8 58 0a e7 d1 1b 78 84
84 dd 50 42 37 41 50 89 7a c8 2f 39 11 09 68 a8
```

There is still a repeating pattern here.  There are two distinct sets of bytes, one for on & one for off, but... what?  Why is it so noisy?
Who designs their protocol like this?
The answer is: someone who is trying to hide something.

## Step 2.  Replay attacks

If all you want to do is turn the lights on and off, then the incomprehensible but repeating series of bytes is probably good enough to control power on the device.  You can test this with `gatttool`.  It will allow you to connect to a BLE device and blindly send a series of bytes to it.  You will need to know which `handle` to send the bytes to, you can get this information from Wireshark.

![](/wp-content/uploads/2023/12/wireshark.jpg){:class="img-responsive"}

If you want to a bit more control then we will need to make sense of all those bytes, so let's go to the source...

## Step 3. Decompile the Android app

Download the apk for the app and open it up in [jadx](https://github.com/skylot/jadx).  Behold the secrets within!

In the case of the lights which I've been trying to get working I see a bunch of references to AES.  This would explain the seemingly complex protocol.
If indeed the data is encrypted there are a few things we know:

- The encrypted data doesn't change every time.  We can see some repeating patterns and the same command seems to generate the same bytes.  This suggests the data is encrypted with the same key each time, if it's AES then it's probably ECB encryption, and probably doesn't have an initialisation vector. i.e. a quick hack seemingly to annoy people like me.
- The data needs to be decrypted quickly on a low power MCU.  So hundreds of bits keys, not thousands.
- It's possible that the key is unique to each device (e.g. the MAC address is the key) but that would make production costs higher and more risky, so is unlikely.  This could be tested if you have multiple devices.
- From previous reverse engineering adventures, we know that the software is often MVP quality, and so the easiest solution (a fixed key) is the most likely.

The source makes use of a compiled AES library `libAES.so` which `jadx` can't help me with.

![](/wp-content/uploads/2023/12/jadx.jpg){:class="img-responsive"}
![](/wp-content/uploads/2023/12/jadx2.jpg){:class="img-responsive"}

This is where I got stuck.  For about 5 minutes.  I asked [@popey](https://ubuntu.social/@popey) and [@sil](https://mastodon.social/@sil) for some ideas.  @sil Googled some of the decompiled app code and found [this](https://habr-com.translate.goog/ru/articles/722412/?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=en-US&_x_tr_pto=wapp&_x_tr_hist=true) page.  On closer examination the code looks identical.  This chap used [ida free](https://hex-rays.com/ida-free/) to decompile the AES library and found the key embedded in it.  Let's try that key.

```python

from Crypto.Cipher import AES
key = [
        0x34,
        0x52,
        0x2A,
        0x5B,
        0x7A,
        0x6E,
        0x49,
        0x2C,
        0x08,
        0x09,
        0x0A,
        0x9D,
        0x8D,
        0x2A,
        0x23,
        0xF8
    ]

def decrypt_aes_ecb(ciphertext, key):
    cipher = AES.new(key, AES.MODE_ECB)
    plaintext = cipher.decrypt(ciphertext)
    return plaintext
```

When we try and decrypt the `on` and `off` packets we get:

```text
05 54 55 52 4E 01 00 00 00 00 00 00 00 00 00 00
05 54 55 52 4E 00 00 00 00 00 00 00 00 00 00 00
05 54 55 52 4E 01 00 00 00 00 00 00 00 00 00 00
05 54 55 52 4E 00 00 00 00 00 00 00 00 00 00 00
05 54 55 52 4E 01 00 00 00 00 00 00 00 00 00 00
05 54 55 52 4E 00 00 00 00 00 00 00 00 00 00 00
```

Success!  This is a lot more sensible.  A fixed header, byte 5 switching between a `1` and a `0` for on and off, and a bunch of zeros.

We can now decrypt all the packets being sent to the device and we can encrypt our own bytes so that we can duplicate the controls from the Android app in our own code.  It's pretty much mission accomplished at this point.

## Step 4. All the functions

What I tend to do now is work my way through the app carrying out each function and recording the bytes sent.  In order to give yourself some clues I highly recommend writing down exactly what you're doing, and doing each function multiple times.  Take your time, a slip of the finger on the screen can lead to time lost to trying to find a red herring.
Also, try to use some kind of separator between different functions, such as turning the lights on and off.  This gives you a demarcation point to try and line up your notes with the bytes being captured.

For example, your process might be:

```text
turn off, turn on - [start of function]
set to red
set to green
set to blue
set to red
set to green
set to blue
set to red
set to green
set to blue
turn off, turn on - [end of colour changing]
set brightness to 100%
set brightness to 50%
set brightness to 10%
set brightness to 50%
set brightness to 100%
turn off, turn on - [end of brightness]
```

This will help you to spot patterns in the data and see which bytes change depending on what you are doing.

If you have used `jadx` you can search the source code for the bytes you see and that might give you a faster way to locate the control code in the app.

## Step 5.  Automated e-waste generator

While I was working out all the functions of these lights I noticed that when changing the colours the maximum value ever sent from the app for red, green or blue was `0x1F`, which is a 5 bit number.  I've seen devices which use 5 bit colour before, but never in a set of cheapo LED lights, they are always 8 bit RGB or HSV.  That's odd, but OK.

Once I'd finished working out the rest of the functions I needed I went back to try something... what if I sent an 8 bit value for the colours instead?  All the colours used when setting an effect mode are 8 bit, so it might work.  Let's try it...

It worked!  It worked really well.  The colours were brighter, especially when using pure red, green or blue.  Great success!

Excited by my discovery I got to wondering what other secrets this light controller was hiding from me.  I wonder if there are any additional effects beyond the 10 that the app uses?
A good way to try this out would be a simple loop, right?

```python
    for n in range(20):
        print(f"Setting effect {n}")
        set_effect(n)
        time.sleep(20)
```

I ran this and watched 1 to 10.  So far so good, then it ticked over to 11 and AH HA!  I have found a secret mode!
Then it ticked over to 12 and... darkness.

Oh well, I guess there are only 11 effects, that's fine.  I'll reboot it and finish off the rest of the code.

And that was then end of my fun.  The lights never came back.  They don't advertise on Bluetooth any more and I can't connect to them.  I've tried holding down the button when turning them on.  I've left them unplugged over night to see if that helps, but no.  They are dead.  I guess I overflowed some buffer and I've corrupted the firmware.

All is not lost however.  The LEDs themselves are standard addressable LEDs so I can at least hook the string up to a different microcontroller and use them.

## Tell me how I can break my own lights

I did at least get most of the protocol worked out and have a Github project with a Home Assistant custom component in.  It works, but ya know, try it at your own risk.

(https://github.com/8none1/idealLED)
