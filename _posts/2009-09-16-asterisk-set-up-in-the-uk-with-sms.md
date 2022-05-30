---
id: 209
title: 'Asterisk set up in the UK with SMS'
date: '2009-09-16T22:15:02+01:00'
author: will
excerpt: 'Why you need to buy a TDM400 instead of a AX100p.'
layout: post
guid: 'http://www.whizzy.org/?p=209'
permalink: /2009/09/16/asterisk-set-up-in-the-uk-with-sms/
enclosure:
    - "http://www.whizzy.org/wp-content/uploads/2009/09/holdmusic.mp3\n202680\naudio/mpeg"
categories:
    - asterisk
    - linux
    - tv
---

**EDIT: 25 Sept 2010  
The below specifically refers to Asterisk 1.4. Go to the bottom for an update on 1.6**

Why do I always get myself in to these situations?

Many years ago I knocked up a set of scripts to record TV programs from a DVB-T card. Getting the TV card working was really hard work. I had to hack about with the driver source code, compile custom kernels, build endless versions of the driver, and so on and so on and so on. It was a pain in the arse, which was compounded by the fact that at the time very few other people (end users) were trying to get these cards working, so there wasn’t much community support. My scripts were great and everything and I could record TV, taxes were low and life was, on the whole, pretty good.

Then [Stuart](http://www.kryogenix.org) put me on to MythTV. It quickly became evident that although my scripts were functional they were nothing compared to Myth. So I moved over. Again with the pain.

The basics were there but DVB support was in it’s infancy and it was a lot of hard work getting it working. I persevered and learnt a lot about MythTV in the process. Yay me. At the beginning MythTV was a foreign country. By the time I’d got it working the way I wanted it was my home town.

And so it will be with Asterisk, I hope.

<span style="text-decoration: underline;">Things I have learnt about Asterisk</span>

1. The cheap FXO cards they sell on eBay are more trouble than they are worth.
2. The documentation on the [voip-info site](http://www.voip-info.org/wiki/) is often out of date.
3. UK Caller ID works with the TDM400’s out of the box.
4. Sending &amp; receiving SMS’s does work, but it’s a bitch, and unreliable.
5. The logic in Asterisks’ dial plan (extensions.conf et al) is illogical to me
6. FreePBX is both a blessing and a curse.
7. I love it.

In a bit more detail then:

1\. The FXO cards you can get from eBay for about 20 quid work perfectly well. For making calls. But, there are a few drawbacks. They don’t support polarity reversal which is the method by which incoming calls and Caller ID are announced to Asterisk. You’ll still get ringing indications and be able to make calls, but you won’t get Caller ID. Or rather, you can get Caller ID but you need to patch the Zaptel drivers and the Asterisk source. The patches are old and don’t apply perfectly. They are also seemingly unsupported now, in that no one cares if you are having problems with them. I got the patches applied but Asterisk wouldn’t compile and I don’t know/care enough to fix it. I was in the market for an ATA anyway so I put that 40 quid towards the cost of the TDM400p11 from [NovaVox](http://www.novavox.co.uk/products/analogue-cards/a400p.html) and if I hadn’t bought the [AX-100](http://www.x100p.eu/product_info.php?products_id=39) then the initial outlay for a fully supported card isn’t so bad. It also works out cheaper to add another FXS port the to TDM400 than it does to buy another ATA. The AX-100 also has a US spec. “[hybrid](http://en.wikipedia.org/wiki/Telephone_hybrid)” in it which means that the [unbalanced](http://en.wikipedia.org/wiki/Balun) UK spec phone lines and the AX-100 are not perfectly matched which can cause wicked echo. The TDM400 has a tunable hybrid, and that’s a good thing. All that said – for 20quid I still think it’s worth getting one to play with. Ask me about doing a swap, if you’ve got any cool toys you don’t want (I’ve got a DVB-T card going spare as well!).

2\. The [VOIP-info.org](http://www.voip-info.org/) site is a really good source of data. The website might be ugly, but there is a lot of Asterisk data on there. Unfortunately there is little in the way of information, and some of the data is way out of date. It acts as a good reference point but isn’t for the beginner because most pages assume a lot of prior knowledge. Sure – I could go in and edit the Wiki, but I don’t know if what I’m doing is the best way of doing it, or even the correct way to do it. So for now that’s what this blog post is for.

3\. BT send the Caller ID information before the first ring. The process goes something like this:

```
Line polarity reversed -> Caller ID sent as FSK (sort of modem tones) -> Phone starts to ring -> Call Connected -> Charlie Brown's Teacher -> Call Hung Up -> Line polarity reversed
```

You can see the importance of being able to detect polarity reversal in the call set-up and tear-down. The AX100 simply doesn’t do it. The TDM400 does, and it’s fully supported out of the box, as is UK style Caller ID, where the ID is sent before the first ring. The hack for the AX100 keeps a buffer running all the time and then once a call comes in it looks in this buffer for the caller ID FSK and decodes it. Shonky, I think you’ll agree. My advice is that if you want to do caller ID in the UK buy a TDM400. If you are a masochist then feel free to try with the AX100, I’d love to read your HowTo once you’ve got it working.

4\. SMS. What a bitch. I recently discovered my [subscription with BT](http://www.productsandservices.bt.com/consumerProducts/displayTopic.do?topicId=25504) gives me 200 free SMS text messages a month. So to make sure I was getting best value from the phone subscription I went out and spent 100quid on a PCI card and then spent hours and hours and hours fiddling about trying to get it working. I don’t know why, I probably send about 3 text’s a month from my mobile, the whole [“Because it’s there”](http://en.wikipedia.org/wiki/George_Mallory) thing I guess. A pox on you, Mallory.

Last night I finally got it working semi-reliably for incoming and outgoing texts. I can send and receive from my mobile, but Stuart doesn’t receive my messages. Ho hum.

<span style="text-decoration: underline;">**Firstly, receiving SMS’s.** </span> Now, I don’t really understand the extensions.conf language. It’s all a bit, well, wrong, to my eyes. All I can tell you is that this is how I got it working. **NOTE: This WILL NOT work for you if you just copy and paste.** Sorry about that. Perhaps someone can help me re-write it so that it does?

I’m using [FreePBX](http://www.freepbx.org/) as a GUI and it rewrites some of the Asterisk config files for you. I’m not sure if the context \[from-pstn\] is one that I created, or a standard one. What you need to find out is what route incoming calls from the PSTN take i.e. which contexts they pass through before they start ringing on your internal phones. You can do this my running the Asterisk console and turning verbosity up to about 6 (core set verbose 6) and then calling in from outside. You should see a trace of what contexts get called as the call comes in.

For example, my call flow goes something like this:

```
[from-zaptel] -> [from-pstn] -> [macro-user-callerid] -> 600@[ext-group]
```

Where 600 is an extensions number I set up with FreePBX to ring all internal phones.

I chose to test for SMS’s in the \[from-pstn\] context. You can visualise what the \[from-pstn\] context does when a call comes in by using the Asterisk CLI. Use the command “dialplan show s@from-pstn” to see the instructions that would be followed when \[from-pstn\] gets called at position “s”, or start.

Originally, mine went something like this:

```
[ Included context 'ext-did-0001' created by 'pbx_config' ]
's' => Set(__FROM_DID=${EXTEN})                   [pbx_config]
Gosub(cidlookup|cidlookup_1|1)             [pbx_config]
ExecIf($[ "${CALLERID(name)}" = "" ] |Set|CALLERID(name)=${CALLERID(num)}) [pbx_config]
Set(__CALLINGPRES_SV=${CALLINGPRES_${CALLINGPRES}}) [pbx_config]
SetCallerPres(allowed_not_screened)        [pbx_config]
Goto(ext-group|600|1)                      [pbx_config]
```
You can follow the call progression through this quite easily. We jump in at position (or priority) 1, we set a variable, then we jump to some cidlookup sub-routine, then if the CALLERID(name) is blank we set it to be the same as the caller id number, then something about CALLINGPRES, not sure what that does, ditto the next line, and then we go to ext-group|600|1. Ahhaaa! I know that 600 is the group that I call to ring all the phones, so that must be where the call is handed off to other contexts or functions that let you do talking to people. Since an incoming SMS doesn’t need to get as far as ringing on a phone, it makes sense to me to interrupt the call flow during \[from-pstn\].

So I added \[from-pstn-custom\] to /etc/asterisk/extensions\_custom.conf like this:

```
[from-pstn-custom]
exten => s,3,GotoIf($["${CALLERID(num)}" = "08005875290"]?will-sms,s,1)
exten => s,4,Verbose("Not an incoming SMS")
exten => s,5,Gosub(cidlookup,cidlookup_1,1)
exten => s,n,ExecIf($[ "${CALLERID(name)}" = "" ] ,Set,CALLERID(name)=${CALLERID(num)})
exten => s,n,Set(__CALLINGPRES_SV=${CALLINGPRES_${CALLINGPRES}})
exten => s,n,SetCallerPres(allowed_not_screened)
exten => s,n,Goto(ext-group,600,1)

[will-sms]
exten => s,1,Verbose(=============Entered Will SMS)
exten => s,n,Answer()
exten => s,n,Wait(2)
exten => s,n,SMS(default|a)
exten => s,n,Verbose(=============Done with Will SMS)
exten => s,n,Hangup(16)
```

How does it work? The \[from-pstn-custom\] overrides the \[from-pstn\] from the main extensions.conf file provided by FreePBX and adds a line that branches to a context called \[will-sms\] which then uses the SMS application to receive the SMS and then hangup the phone. (Actually, I think SMS hangs up for you. But, you know, whatever)

In \[from-pstn\] in extensions.conf is a line that “includes” the config from “from-pstn-custom”. By virtue of being imported, the instructions in \[from-pstn-custom\] take precedence over the other config, <span style="text-decoration: underline;">because it is imported first</span>. The config files are read line by line in the order they appear in the file. But, the imported instructions do not completely replace the config that is already there. They are sort of merged. That’s why I’ve copied a load of bits from the original call flow in to my new \[from-pstn-custom\]. If I left them out, then some bits of my new context would be applied and some wouldn’t. The way I got it to work was to replicate the call flow to the point of it branching off to \[ext-group\] in my new context. If you do a “dialplan show s@from-pstn” now, you see this:

```
[ Included context 'from-pstn-custom' created by 'pbx_config' ]
's' =>            3. GotoIf($["${CALLERID(num)}" = "08005875290"]?will-sms|s|1) [pbx_config]
4. Verbose("Not an incoming SMS")             [pbx_config]
6. Gosub(cidlookup|cidlookup_1|1)             [pbx_config]
7. ExecIf($[ "${CALLERID(name)}" = "" ] |Set|CALLERID(name)=${CALLERID(num)}) [pbx_config]
[pbx_config]
9. Set(__CALLINGPRES_SV=${CALLINGPRES_${CALLINGPRES}}) [pbx_config]
10. SetCallerPres(allowed_not_screened)       [pbx_config]
11. Goto(ext-group|600|1)                     [pbx_config]

[ Included context 'ext-did-0001' created by 'pbx_config' ]
's' =>            1. Set(__FROM_DID=${EXTEN})                   [pbx_config]
2. Gosub(cidlookup|cidlookup_1|1)             [pbx_config]
3. ExecIf($[ "${CALLERID(name)}" = "" ] |Set|CALLERID(name)=${CALLERID(num)}) [pbx_config]
4. Set(__CALLINGPRES_SV=${CALLINGPRES_${CALLINGPRES}}) [pbx_config]
5. SetCallerPres(allowed_not_screened)        [pbx_config]
6. Goto(ext-group|600|1)                      [pbx_config]

[ Included context 'ext-did-catchall' created by 'pbx_config' ]
'_.' =>           1. Noop(Catch-All DID Match - Found ${EXTEN} - You probably want a DID for this.) [pbx_config]
2. Goto(ext-did|s|1)                          [pbx_config]
```
You can see my new \[from-pstn-custom\] context at the start, along with the copied commands from ext-did-001 to ensure that they run in the way I expected. The upshot of this is that I now branch to my new context \[will-sms\] when the incoming caller ID is the BT 0800 number from where SMS’s originate. You should note that the 0800 number can change if you have more than one incoming SMS box registered, so don’t do that. Just register one incoming address (more on this later).

The \[will-sms\] is pretty straight forward. Answer the phone, wait 2 seconds, and then start the SMS application which does all the clever noises. The key here is Wait. Without it SMS reception is intermittent at best. The SMS(default\|a) tells the SMS app to receive in to the default queue (this works, don’t understand fully why) and to do an “a” for answer. I had a load of problems with the example from VOIP-info where it passes in the extension number, which is “s” for start. If the SMS application sees an “s” it thinks it means “send”. That won’t work!

If you manage to bodge all this in to your dial plan then you should find your incoming SMS’s in /var/spool/asterisk/sms/mtrx

Make sure that /var/spool/asterisk is writeable by the same user as Asterisk is running as.

<span style="text-decoration: underline;">**Sending SMS’s.**</span> Was a whole bunch of no fun as well. Especially if you don’t understand Asterisk properly.

The command line I’m using to send an SMS is:

```
smsq --motx-channel=Zap/2/17094009 -d <phone number>  -m "<message>"
```

That works because zap channel 2 is my PSTN line. You might need to change this. 17094009 is the SMSC centre number for BT to accept SMS messages for delivery to other networks.

When I was trying to get sending working, which I actually did first, I couldn’t reliably get Asterisk to talk to the SMSC. Before I had the receive working properly the BT system would ring me up and I’d answer the phone, then not hearing a carrier the BT system would hang up again. This gave me an idea, if I could emulate the carrier then I could hear what the BT system was sending down the line. I sent an SMS to the home phone, it rang in on one of the extensions and I whistled at it. Because I am just so [1337](http://en.wikipedia.org/wiki/Leet) I hit the right tone and I could hear FSK noises coming down the line, but they were really really quiet. I tweaked the gain up a little bit in the /etc/asterisk/zapata.conf file:

```
rxgain=2
txgain=0
```

and tried again. It was louder! Sending a test message to “00000” with the text of “test” causes the BT system to send a message back to you (you need to send “register” to 00000 before BT will send you messages as text instead of reading them to you by the way – but if it can’t successfully deliver the message back to you to say it has received your “register” message it will ignore the request and you’ll keep getting the messages read to you by a friendly robot – hence the need to get receiving working first). It sent! I saw some hex coming from the SMS application in the Asterisk log. I sent a text to my mobile, and it arrived. Yay! problem solved.

5\. Asterisk’s dial plan. Bonkers. I still can’t make sense of them. Oh well, I’ll get there in the end.

6\. FreePBX is brilliant if you want to get Asterisk configured with some very complex applications (think voicemail) in a matter of minutes. But, if it makes editing the config files by hand a bit more complicated because you have to use xxxx\_custom files. Thus, all the replication of code when as above when you don’t understand how it all works.

7\. Check out my hold music hacked together from various online sources, and no doubt subject to various copyright restrictions. I’ve compressed the hell out of it, if you want a better quality version for your own amusement let me know.

[Hold Music](/wp-content/uploads/2009/09/holdmusic.mp3)

I crack myself up, I really do.

Also – think of all the cool things you could do if you could phone or text your computer and get it to do things for you.

Now I’m off to write myself an gmail &lt;-&gt; SMS gateway doodad.

Ok – with Asterisk 1.6 the above is slightly less relevant. Try this in your dialplan instead:

```
[will-sms]
exten => s,1,Verbose(=============Entered Will SMS)
exten => s,n,Answer()
exten => s,n,SMS(default,ap(1500))
exten => s,n,Verbose(=============Done with Will SMS)
exten => s,n,Hangup(16)
```

The SMS application has the option to add a pause which I've set to 1500 ms.  Seems to work.
