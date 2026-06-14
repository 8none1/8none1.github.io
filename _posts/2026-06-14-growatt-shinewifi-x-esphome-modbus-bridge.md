---
layout: post
title: "The Growatt ShineWiFi-X 'USB' dongle is really a Modbus bridge, so I reflashed it with ESPHome"
date: 2026-06-14
tags: [home-assistant, esphome, growatt, modbus, solar, esp8266, reverse-engineering, mqtt]
social-share: true
excerpt: "I had a couple of spare Growatt ShineWiFi-X dongles and wanted them doing the same job as my EW11 RS485 bridges: feeding my Modbus-to-MQTT poller. The twist is that the inverter's 'USB' port isn't USB at all, it's TTL serial speaking Modbus. Here's how I put ESPHome on the ESP8266 inside to turn the dongle into a dumb, swappable serial-to-TCP bridge, plus the flashing gotchas that ate an afternoon."
---

## TL;DR

The Growatt ShineWiFi-X is a Wi-Fi dongle that plugs into the "USB" port on a Growatt inverter. Inside it is an ESP8266. The port looks like USB but it is not: the pins carry plain TTL serial, and what flows over that serial line is Modbus. So the dongle is really a serial-to-Wi-Fi Modbus gadget wearing a USB-shaped shell. I reflashed two spare ones with [ESPHome](https://esphome.io/) so they act as a **dumb serial-to-TCP bridge**, an [Elfin EW11](https://www.hi-flying.com/elfin-ew10-elfin-ew11) clone, and let my existing Python poller keep doing all the Modbus decoding. Firmware and notes are in my [growatt_modbus repo](https://github.com/8none1/growatt_modbus) under `shinewifi-bridge/`.

## The setup I already had

For a while now I have been pulling data off my two Growatt SPH inverters over Modbus TCP, decoding the registers in a small Python service, and publishing the result to MQTT for Home Assistant and Grafana. The transport in the middle was a pair of [Elfin EW11](https://www.hi-flying.com/elfin-ew10-elfin-ew11) modules: little RS485-to-TCP bridges wired to the inverters' RS485 terminals. The EW11 does one job, exposes the bus as a TCP socket on port 502, and my poller connects to it.

That works, but the EW11 is an extra box to power and wire. Growatt inverters also ship with their own Wi-Fi dongle, the ShineWiFi-X, which plugs straight into a port on the inverter. I had two spares doing nothing. Could they do the EW11's job and tidy the whole thing up? Short answer: yes.

## The "USB" port is a lie (in a useful way)

Here is the bit that trips people up, and the thing worth knowing if you ever poke at one of these. The ShineWiFi-X has a **USB Type-A connector**. The inverter has a matching USB-shaped socket. But there is no USB protocol involved at all. Those pins carry **TTL-level UART serial** at 115200 8N1, plus power. The USB connector is used purely as a convenient, robust physical plug.

What travels over that serial line is **Modbus RTU**. The ESP8266 inside the dongle is the Modbus master, and the inverter is the slave at address 1. This is exactly the same conversation the EW11 has over RS485, just on a different bit of wire:

```
EW11 today:        Inverter --RS485--> EW11      --(Wi-Fi)--> poller
ShineWiFi-X (new): Inverter --TTL UART--> ESP8266 --(Wi-Fi)--> poller
```

So when the excellent [OpenInverterGateway](https://github.com/OpenInverterGateway/OpenInverterGateway) project (well worth a look, it is where I got a lot of the hardware detail) talks about driving these inverters "over serial", that is what it means. There is no Modbus-over-USB witchcraft to worry about. It is serial Modbus the whole way down, and the connector shape is a red herring.

## Dumb bridge, not a smart dongle

ESPHome could have decoded the inverter natively on the ESP8266 and published sensors straight to Home Assistant. Plenty of people do exactly that. I deliberately did not, and I think the reasoning is worth spelling out.

All of my hard-won logic already lives in one tested Python codebase: the register map, the 32-bit register pairings, the asymmetric clock-sync quirk, and the Octopus Agile charge scheduling that writes back to the inverter. Re-implementing all of that in ESPHome YAML on a constrained ESP8266 would mean abandoning my single source of truth and porting control logic into lambdas. No thanks.

So I split it cleanly: **transport** (get bytes on and off the bus) lives on the dongle, **logic** (decode and decide) stays in Python. The dongle should "sit there doing nothing until asked", which is the whole point of an EW11. If it ever misbehaves I can pull it out, drop the EW11 back on the RS485 terminals, and nothing else in the stack changes. That swappability is worth a lot.

In ESPHome terms, "dumb bridge" means the [`stream_server`](https://github.com/oxan/esphome-stream-server) external component: a raw TCP-to-UART passthrough. No Modbus smarts on the device, it just shuttles bytes between port 502 and the serial pins.

```yaml
external_components:
  - source: github://oxan/esphome-stream-server
    components: [stream_server]

uart:
  id: inverter_uart
  rx_pin: GPIO3            # RXD0
  tx_pin: GPIO1            # TXD0
  baud_rate: 115200

stream_server:
  uart_id: inverter_uart
  port: 502

logger:
  baud_rate: 0             # UART0 is the inverter, so no serial logging
```

That `logger: baud_rate: 0` matters: the inverter is on the ESP8266's hardware UART (GPIO1/GPIO3), so if you leave serial logging on it scribbles all over your Modbus line. Logs still come out over the network.

## One framing gotcha

There is a subtlety that will bite you. `stream_server` is a *raw* passthrough, so the TCP socket carries raw Modbus **RTU** frames (with their CRC). That is "RTU over TCP". A normal Modbus **TCP** client expects the MBAP header instead, no CRC. My EW11s present proper Modbus TCP, which is why my poller spoke MBAP to them.

The fix is one line on the client side: tell pymodbus to use the RTU framer for that device.

```python
from pymodbus.client import ModbusTcpClient
from pymodbus import FramerType

client = ModbusTcpClient(host="192.168.x.x", port=502, framer=FramerType.RTU)
```

EW11 entries stay exactly as they were on the default framer; only the dongle entries opt into RTU. (A small aside while I was testing: a fresh pymodbus 3.13 has renamed the slave argument yet again, from `unit` to `slave` to `device_id`. Pin your versions.)

## Flashing it: where the afternoon went

The ESP8266 in my units turned out to be an ESP8266EX with 4MB of flash (I confirmed this with `esptool flash-id`, which reports the chip and flash size). In ESPHome/PlatformIO terms that is the `esp12e` board profile, a generic 4MB ESP8266 build target, not a claim about the exact module.

To flash one for the first time you have to get at the ESP's serial pins and drop it into the bootloader. The recipe:

1. **Back up the stock firmware first.** This is your undo button. `esptool ... read-flash 0x0 ALL stock.bin`. The image came back starting with the `0xE9` ESP8266 magic byte and full of `growatt Technology Co. Ltd.` strings, so I know it is a good dump. Keep it safe, it almost certainly contains your Wi-Fi credentials and cloud keys.
2. **Hold GPIO0 to GND while powering on.** GPIO0 is only sampled at reset, so it decides bootloader-vs-run at the instant power is applied.
3. Flash with esptool, or `esphome run`.
4. **Remove the GPIO0 jumper** and power-cycle so it boots your new firmware.

A few hard-won lessons, so you do not lose the same afternoon I did:

- **Pick one baud rate and keep it.** I had a perfectly good connection at 115200, bumped to 460800 to speed up a 4MB read, and instantly got `No serial data received`. Changing baud mid-session drops sync. 230400 is a safe, faster step if you want one.
- **If `flash-id` works, your wiring is fine.** It is a full round trip: the chip has to receive commands and reply with its ID, MAC and flash size. So if that succeeds, a later failure is a baud or bootloader-state issue, not a wiring fault. That one realisation saved a lot of pointless re-seating of jumpers.
- Most cheap USB-serial adapters do not wire up the auto-reset lines on these dongles, so esptool cannot reset the chip for you. You power-cycle it into the bootloader by hand, and pass `--before no-reset` so esptool does not wait around trying.

After the first serial flash, everything else is **OTA over Wi-Fi**, which is the real joy of ESPHome here. I have not had to open a dongle since. One small trap: `esphome upload` does *not* recompile, it pushes whatever binary already exists, so run `esphome compile` first (or use `esphome run`) when you have changed the config.

## LEDs and a button, for free

The ShineWiFi-X has three LEDs (green on GPIO0, red on GPIO2, blue on GPIO16, per the OpenInverterGateway sources) and a button. None of the LED pins clash with the UART, so I can use them. With the case on you mostly see the green one through a little diffuser window, so green carries the health signal: a steady ~1Hz heartbeat. If it is *moving*, the firmware loop is alive; if it freezes solid, something has wedged. Red I use as ESPHome's status LED, and blue lights while a poller is connected to port 502. Because my poller opens and closes the connection on every poll, the blue LED ends up blinking once per poll cycle all by itself, a free activity light.

The button is wired to the analog pin (A0), so I read it with an ADC and threshold it into a press. The plan is to fire a Home Assistant event on press so I can hang automations off it, and a long-press for a safe-mode reboot. Capturing useful inputs that are already sitting there for nothing is always worth it.

## Does it actually work?

Yes. With a dongle plugged into each inverter, a quick RTU-over-TCP read straight off the bridge returns live registers:

```
device1 (battery inverter) 192.168.x.204:502
  input[0..7]: [5, 0, 200, 921, 1, 0, 140, 798]
```

Decoded against the Growatt map that is status 5, PV string voltages around 92V and a small trickle of PV power, exactly right for a sunny evening on the way down. The full chain works: pymodbus to TCP to the ESP's `stream_server` to the UART to the inverter and back. Both dongles now sit on fixed DHCP-reserved addresses, advertise themselves over mDNS, and OTA happily.

(A note for the container crowd: those tidy `.local` mDNS names do not resolve from inside a slim Docker container, there is no Avahi in there, so the poller talks to the dongles by IP. A DHCP reservation keeps that address stable.)

## Where next

The dongles are doing the EW11's job now. The remaining step is the deliberate cut-over: point the poller (and, for the battery inverter, the control side) at the dongle with that one `framer: rtu` flag, and retire the EW11 on that inverter. Worth remembering that Modbus allows exactly one master per bus, so you do not want the EW11 and the dongle both polling the same inverter at once.

It is a satisfying little result: a spare cloud dongle, freed from the cloud, doing one honest job on my own network. And a reminder that a USB connector does not always mean USB.

If you want the firmware and the gory detail, it is all in [github.com/8none1/growatt_modbus](https://github.com/8none1/growatt_modbus).
