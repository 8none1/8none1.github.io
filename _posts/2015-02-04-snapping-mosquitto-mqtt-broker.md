---
id: 598
title: 'Snapping Mosquitto MQTT broker'
date: '2015-02-04T17:32:44+00:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=598'
permalink: /2015/02/04/snapping-mosquitto-mqtt-broker/
authorsure_include_css:
    - ''
authorsure_hide_author_box:
    - ''
categories:
    - linux
    - 'Making the world a better place'
    - RaspberryPi
    - Ubuntu
---

*As part of my ever expanding home automation system I wanted to use MQTT to publish data on my network. With the release of the Raspberry Pi 2 I can run Ubuntu Core to create a reliable, secure and easily updated server which is a perfect fit for requirements of an MQTT broker and general HA controller. I asked some Ubuntu friends to help me package Mosquitto as a Snap, and in return I would write down how we did it. Here’s the story…*

Start by reading this: [https://developer.ubuntu.com/en/snappy/](https://developer.ubuntu.com/en/snappy/ "https://developer.ubuntu.com/en/snappy/")

In summary; a Snappy application is secure because it’s wrapped with AppArmor. It’s easier to install and upgrade because everything is packaged in a single file and installed to a single location. That location is backed-up before you install a new version, and so if the installation goes wrong you can revert to the previous version easily by copying the original files back (or rather, Snappy will do all of that for you). Simplifying things slightly there are two types of Snappy “application”: Apps and Frameworks. Frameworks can extend the OS and provide a mediation layer to access shared resources. Apps are your more traditional top-level items which can use the provided frameworks, or bundle everything they need in to their Snap. This makes things much easier for app providers because they are now in charge – they can be assured that no library will change underneath them. This is a huge benefit!

## Let’s get Mosquitto snapped.

### 1. Install QEMU to run an Ubuntu Core machine

[http://www.ubuntu.com/cloud/tools/snappy#snappy-local](http://www.ubuntu.com/cloud/tools/snappy#snappy-local "http://www.ubuntu.com/cloud/tools/snappy#snappy-local")

First we install the KVM hypervisor:

```
sudo apt-get install qemu-kvm
```

Then check everything is as it should be with:

```
kvm-ok
```

Now download the latest Ubuntu Core image from here: [http://cdimage.ubuntu.com/ubuntu-core/preview/](http://cdimage.ubuntu.com/ubuntu-core/preview/ "http://cdimage.ubuntu.com/ubuntu-core/preview/") At the time of writing this is the newest x86-64 image: [http://cdimage.ubuntu.com/ubuntu-core/preview/ubuntu-core-alpha-02\_amd64-virt.img](http://cdimage.ubuntu.com/ubuntu-core/preview/ubuntu-core-alpha-02_amd64-virt.img "http://cdimage.ubuntu.com/ubuntu-core/preview/ubuntu-core-alpha-02_amd64-virt.img")

Then launch the virtual machine. This command port forwards 8022 on your local machine to 22 on the virtual machine, so you can SSH to port 8022 on localhost and actually connect to the Ubuntu Core machine. It gives the Core machine 512MB of RAM, nicely achievable on a modest budget (The Pi2 has 1 GB). We also forward port 1883 from to the VM, which will allow us to connect to the Mosquitto server on our VM once it’s all installed.

```
kvm -m 512 -redir :8022::22 -redir :1883::1883 <vm image file>
```

Once it’s booted you can connect to it with SSH. The username and password are “ubuntu”.

```
ssh -p 8022 ubuntu@localhost
```

To make things a bit easier, why not use key authentication? On your host machine:

```
ssh-copy-id -p 8022 ubuntu@localhost
```

We should also upgrade our Ubuntu Core VM before we start. SSH in to your box and run:

```
sudo snappy update
sudo reboot
```

### 2. Build Mosquitto in the right way

Back on your host (not the virtual machine you just created above) create some directories to hold the code and download the latest stable source and the build dependencies for Mosquitto:

```
sudo apt-get install build-essential cmake
sudo apt-get build-dep mosquitto
```

```
mkdir -p mosquitto/install mosquitto/build
```

```
cd mosquitto
```

```
wget http://mosquitto.org/files/source/mosquitto-1.3.5.tar.gz
```

```
tar xvzf mosquitto-1.3.5.tar.gz
```

```
cd build
```

Time to build Mosquitto. Before you run the commands below, a bit of background information. The cmake line will force cmake to install the binaries to the location specified with INSTALL\_PREFIX, rather than /usr/local. This is required to bundle all of the binaries and other files to the “install” directory we created above, making it possible to package as a Snappy.

```
cmake -DCMAKE_INSTALL_PREFIX=`readlink -f ../install/` ../mosquitto-1.3.5
```

```
make -j`nproc`

make install
```

nproc spits out the number of processor cores you have, so the make line above will use as many processor cores as you have available. It’s not required, and for Mosquitto which is fairly small it’s not worth worrying about, but for a bigger job this is quite handy.

If you look in the “../install” directory you’ll see a familiar structure containing all the goodies needed by Mosquitto.

### 3. Find the libraries needed and copy them in to your Snappy project

Change in to the install/lib directory and use ldd to display the linked libraries for the two main .so files:

```
ldd lib/libmosquitto.so.1.3.5 lib/libmosquittopp.so.1.3.5 | grep '=>' | awk '{ print $1 }' | sort | uniq
```

This uses ldd to show the libraries required by Mosquitto, and then sorts them in to a nice list. You’ll see something like this:

```
libcares.so.2
libcrypto.so.1.0.0
libc.so.6
libdl.so.2
libgcc_s.so.1
libmosquitto.so.1
libm.so.6
libpthread.so.0
librt.so.1
libssl.so.1.0.0
libstdc++.so.6
linux-vdso.so.1
```

Now, on the Ubuntu Core machine we can run this little script:

```
for i in `cat`; do find /lib /usr/lib -name $i; done
```

Copy the list from the previous command to the clipboard and then paste it in to terminal where this command is running and hit Ctrl-D to submit the list. The script will then search Ubuntu Core for the libraries required. If it finds them they will be displayed, if it doesn’t then they are not available in Ubuntu Core by default and will need to be included in your Snappy package.

`linux-vdso` is the Linux kernel and is available on every Linux system by default, so we don’t need to provide that specifically.

`libssl, libcrypto, libpthread, librt, libc `and` libdl` are all available in Ubuntu Core by default – so we don’t need those either.

That leaves just `libcares` to be copied in to our package.

```
 cp /usr/lib/x86_64-linux-gnu/libcares.so.2.1.0 .
```

We should already be in the ‘lib’ directory, hence the ‘.’ above. We are copying libcares in to the lib directory of our Snap, and when we run the Snap we will pass in the library path to make sure Mosquitto can find it. More on this later.

### 4. Add the meta data required for the Snappy package

Reference: [https://developer.ubuntu.com/en/snappy/guides/packaging-format-apps/](https://developer.ubuntu.com/en/snappy/guides/packaging-format-apps/ " https://developer.ubuntu.com/en/snappy/guides/packaging-format-apps/")

Create the meta data directory inside the install directory (change to the install directory, it should just be cd ..):

```
mkdir meta
```

Create the package.yaml file:

```
nano meta/package.yaml
```

And this is what we’re putting in it:

```
name: mosquitto.willcooke
architecture: amd64
version: 1.3.5
icon:
services:
 - name: mosquitto
 start: ./sbin/mosquitto.sh
ports:
 required: 1883
```

Information about these fields and what they mean is available in the reference linked to above, but they are easily understandable. A comment on the name though, you need to append .&lt;yournamespace&gt; where your namespace is as you select in your Ubuntu myapps account. One thing to mention, you can see that to start our Snap we are calling a shell script. This allows us to pass in extra options to Mosquitto when it runs.

Next we need to create a readme file:

```
nano meta/readme.md
```

This file needs to contain at least a couple of non-blank lines. Here’s what we put in it:

```
This is a Snappy package for Mosquitto MQTT broker.
```

```
Information about Mosquitto is available here:  http://mosquitto.org/
```

```
Information about MQTT is available here: http://mqtt.org/
```

We also need to configure our Mosquitto server, by editing the conf file. Most of the settings can be left as default, so we will create a new conf file with only the bits in we need.

```
mv etc/mosquitto/mosquitto.conf etc/mosquitto/mosquitto.conf.ori
```

```
nano etc/mosquitto/mosquitto.conf
```

Add these two lines:

```
user root
persistence_location /var/apps/mosquitto/current/
```

We need to change this to run as root. Since our Snap will be confined there is no risk here. I expect the ability to run as non-root users when using Snappy will be improved, but really it’s not necessary.

We also need to add a small shell script to start Mosquitto with the right options. Create a file in install/sbin called mosquitto.sh:

```
nano sbin/mosquitto.sh
```

And add this:

```
<pre style="padding-left: 30px;">#!/bin/sh
LD_LIBRARY_PATH=./lib:$LD_LIBRARY_PATH exec ./sbin/mosquitto -c etc/mosquitto/mosquitto.conf
```

We are specifying where to find the extra libraries we require and where to find the conf file. Make that file executable:

```
chmod +x sbin/mosquitto.sh
```

### 5. Build your Snappy package

Add the Snappy PPA to get the build tools, and then install them:

```
sudo add-apt-repository ppa:snappy-dev/beta
sudo apt-get update
sudo apt-get dist-upgrade
sudo apt install snappy-tools
```

In your `install` directory run:

```
snappy build .
```

If you see an error about ImportError: No module named ‘click.repository’ then you likely have a clash between the Click library version in the SDK team PPA and the version in the Snappy PPA. This will be fixed soon, but in the meantime I would suggest installing ppa-purge via apt-get and then running `sudo ppa-purge ppa:ubuntu-sdk-team/ppa`.

If you see an error about “expected &lt;block end&gt;” in the package.yaml check the whitespace in the file. It’s likely a copy and paste error.

### 6. Install your Snappy package

Once you have your .snap file you can install it to your virtual machine like this:

```
snappy-remote --url=ssh://localhost:8022 install ./mosquitto_1.3.5_amd64.snap
```

### 7. Test your Snappy package

If everything has gone to plan Mosquitto should now be running on your virtual machine. In order to test you’ll need to write a test Publisher and Subscriber. I used the Python Paho library.

Here’s an example Publisher:

```
#!/usr/bin/python
```

```
import paho.mqtt.client as mqtt
from datetime import datetime
from time import sleep
```

```
def send_mqtt(topic, message):
 log("Sending MQTT")
 log("Topic: "+topic)
 log("Message: "+message)
 mqttc.reconnect()
 mqttc.publish(topic, message)
 mqttc.loop() 
 mqttc.disconnect()
```

```
print "Time server starting up...."
mqttc = mqtt.Client("python_pub")
mqttc.connect("localhost", 1883)
```

```
while True:
 tstr = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
 send_mqtt("/information/time",tstr)
 sleep(10)
```

And here’s an example Subscriber:

```
#!/usr/bin/python
```

```
import paho.mqtt.client as mqtt
import datetime
```

```
def on_connect(client, userdata, rc):
 print "Connected with result code "+str(rc)
 client.subscribe("#")
 
def on_message(client, userdata, msg):
 print "Topic: ", msg.topic+'\nMessage: '+str(msg.payload)
 
 
client = mqtt.Client()
client.on_connect = on_connect
client.on_message = on_message
```

```
client.connect("localhost", 1883, 60)
```

```
client.loop_forever()
```

## What’s next?

We’ve built a Snappy package for amd64 (or whatever your native architecture is), but we really need to be cross-architecture to give people the best choice of platform on which to use the package. This involves cross compiling, which can be tricky to put it mildly.

I spoke to [Alexander Sack](https://plus.google.com/+AlexanderSack/posts "https://plus.google.com/+AlexanderSack/posts"), the Director of Ubuntu Core, and asked what was coming next for Snappy and I was very excited to hear about easier cross-compilation methods as well as a cool script to help automate gathering the libraries in to your package. I’ll find out more about these and follow up with another post about

## Special Thanks

A huge “Thank You!” to [Saviq](https://plus.google.com/113078171667682980510/posts "https://plus.google.com/113078171667682980510/posts") and [Didrocks](https://www.google.com/+DidierRoche "https://www.google.com/+DidierRoche") for doing the actual work and letting me watch.

## Where to get Snappy Mosquitto

amd64 version: [https://myapps.developer.ubuntu.com/dev/click-apps/ubuntu/1500/](https://myapps.developer.ubuntu.com/dev/click-apps/ubuntu/1500/ "https://myapps.developer.ubuntu.com/dev/click-apps/ubuntu/1500/")  
armhf version: [https://myapps.developer.ubuntu.com/dev/click-apps/ubuntu/1502/](https://myapps.developer.ubuntu.com/dev/click-apps/ubuntu/1502/ "https://myapps.developer.ubuntu.com/dev/click-apps/ubuntu/1502/") (please note, I haven’t been able to test the ARM version because of a lack of hardware. If it doesn’t work let me know and I can fix it.)