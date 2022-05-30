---
id: 297
title: 'Getting NUT to talk to LiebertPSP UPS'
date: '2010-10-06T17:17:07+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=297'
permalink: /2010/10/06/getting-nut-to-talk-to-liebertpsp-ups/
categories:
    - linux
---

I got hold of a couple of LiebertPSP UPSes and connected to my servers. They’re perfectly good for what I need them for, namely stopping my servers having the rug pulled out from underneath them with no notice if there is a power cut.

They don’t appear to have advanced functions like switching off the load independently of the supply which means you can’t power down the servers and then have the UPS switch them back on again once the power is restored and the battery has had a chance to charge, but I can live without that sort of thing.

I assumed that something somewhere would support these, and for the most part NUT does a bang up job. It’s a bit advanced for my needs but most of the work has already been done. In NUT the LiebertPSP is sort-of-supported by the Belkin USB driver with a few notable exceptions:

- The Online/On Battery indication doesn’t work
- The numeric values don’t work for things like Output Voltage

The numbers I can live without but the Online / On Battery is really rather important.

So I’ve cobbled together a new sub-driver using the tools supplied with Nut. It’s far from complete but it does fix most of the annoying problems.

</wp-content/uploads/2010/10/liebertpsp.tar.gz>

You’ll need to download the Nut source and put the .c and .h files in the above tarball in to the drivers directory, then you’ll need to apply this patch in the drivers directory to get the new driver included in the build and to stop the native Belkin driver from claiming the ID of the LiebertPSP:

</wp-content/uploads/2010/10/liebertpsp.diff>

Then recompile Nut.

If everything works you should see a bit more info from your UPS:

```
battery.type: PbAc
```

```
device.mfr: Emerson Network Power
```

```
device.model: LiebertPSP
```

```
device.serial:
```

```
device.type: ups
```

```
driver.name: usbhid-ups
```

```
driver.parameter.bus: 004
```

```
driver.parameter.pollfreq: 30
```

```
driver.parameter.pollinterval: 2
```

```
driver.parameter.port: auto
```

```
driver.version: 2.4.3-2519M
```

```
driver.version.data: LiebertPSP HID 0.1
```

```
driver.version.internal: 0.35
```

```
ups.input.frequency: 50.0
```

```
ups.input.voltage: 243
```

```
ups.mfr: Emerson Network Power
```

```
ups.model: LiebertPSP
```

```
ups.output.percentload: 54
```

```
ups.output.voltage: 240
```

```
ups.powersummary.capacitygranularity1: 1
```

```
ups.powersummary.capacitygranularity2: 1
```

```
ups.powersummary.capacitymode: 2
```

```
ups.powersummary.configvoltage: 12.0
```

```
ups.powersummary.designcapacity: 100
```

```
ups.powersummary.fullchargecapacity: 100
```

```
ups.powersummary.imanufacturer: 19
```

```
ups.powersummary.ioeminformation: 19
```

```
ups.powersummary.iproduct: 1
```

```
ups.powersummary.presentstatus.batterypresent: 1
```

```
ups.powersummary.presentstatus.belowremainingcapacitylimit: 0
```

```
ups.powersummary.presentstatus.charging: 1
```

```
ups.powersummary.presentstatus.discharging: 0
```

```
ups.powersummary.presentstatus.needreplacement: 0
```

```
ups.powersummary.presentstatus.shutdownimminent: 0
```

```
ups.powersummary.rechargeable: 1
```

```
ups.powersummary.remainingcapacity: 100
```

```
ups.powersummary.remainingcapacitylimit: 38
```

```
ups.powersummary.voltage: 14
```

```
ups.powersummary.warningcapacitylimit: 38
```

```
ups.productid: 0001
```

```
ups.serial:
```

```
ups.status: OL
```

```
ups.vendorid: 10af
```

Search hints:

Vendor ID: 0x10af

Product ID: 0x0001