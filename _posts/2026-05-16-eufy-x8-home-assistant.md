---
layout: post
title: "Eufy X8 robot vacuum, local control, and a Home Assistant integration that actually works"
date: 2026-05-16
tags: [home-assistant, eufy, tuya, robot-vacuum, reverse-engineering, python, automation]
social-share: true
excerpt: "The Eufy X8 lives on the older Tuya platform, not the AIOT/protobuf cloud the rest of the Eufy range uses. The existing Home Assistant integrations don't know about this and silently fail. I wrote one that talks to it directly over LAN — no cloud after setup, full vacuum control, and a couple of fun automations to wrap it up."
---

## TL;DR

I bought a pair of [Eufy X8](https://www.eufy.com/products/eufy-x8) robot vacuums. The existing Home Assistant integrations work for the basics on these models, but none of them expose a `goto` service — and I wanted one specifically so I could send the robot to park next to the kitchen bin after a clean. So I wrote [github.com/8none1/eufy-x8](https://github.com/8none1/eufy-x8) — a HACS integration that talks to the X8 over Tuya v3.3 on the local network, exposes `goto`, and keeps working when the Anker cloud has a wobble.

## What the existing integrations actually do (and why I wrote another one)

The Eufy/Tuya HA ecosystem isn't a desert. There are integrations that work on the X8. They just don't do the one thing I wanted them to do.

Most modern Eufy robots — X10 Pro Omni, L60, G40 and friends — live on Anker's AIOT cloud. Their status updates come over an AWS IoT MQTT broker as protobuf-encoded blobs. The various community integrations (including my own [`eufy-clean`](https://github.com/8none1/eufy-clean) fork, itself a fork-of-a-fork going back to jeppesens and martijnpoppen) speak this dialect.

The X8 is older. It's on Eufy's Tuya platform, predating the AIOT migration, and the modern path doesn't really work for it: the Anker AIOT MQTT broker denies subscription for X8 device IDs (`SUBACK 0x80`). What *does* work is Eufy's older V1 cloud API, which has its own MQTT pipe and carries DPS status updates to clients. `eufy-clean` falls back to that path automatically for X8s, and I ran my robots on it happily for some time. It does the basics — start, stop, return, fan speed, battery, errors — and that's most of what you want from a robot vacuum.

What it does not do is `goto`. There is no service to send the robot to an (x, y) coordinate, which means there is no way to make it park next to the bin after a clean. And it has the same downside as anything cloud-based: when the Anker cloud has a wobble, your vacuum goes with it.

The original Tuya-based community integration, [`mitchellrj/eufy_robovac`](https://github.com/mitchellrj/eufy_robovac), is dormant — last commit was 2020. I did use it on the X8 for a while; at some point it stopped working, possibly a firmware change at the robot's end, and I never went back to debug.

So the gap I wanted to close was: local control over the X8, with a proper `goto` service, and no dependency on Anker's broker being up. The result speaks a simple AES-encrypted protocol on TCP/6668 directly to the robot, the same way the Eufy app does when both the app and the robot are on the same Wi-Fi. If you have the device ID and the 16-byte "local key", cloud is optional from there on out.

## What the integration does

It's a HACS-installable custom component for Home Assistant. You give it your Eufy account email and password once; it logs into the Eufy cloud, pulls down the device IDs and local keys for any X8s on the account, and from then on it speaks to the robot over LAN.

What you get for your robot:

- Full vacuum control — start, stop, pause, return-to-dock, locate
- Fan speed select — Low, Medium, High, Max
- Work mode select — Auto, Edge, Spot, No Sweep
- Live status — battery, cleaning time, cleaning area, activity, detailed status, errors
- BoostIQ and Auto-Return switches
- A **`goto` service** that sends the robot to a fixed (x, y) coordinate in its SLAM map — e.g. "park next to the bin"
- A **`locate_brief` service** that beeps for *n* seconds instead of the default ~60, so you can use it as a notification in automations

The protocol research, DPS table, and goto-coordinate fiddliness all live in [tools/CLAUDE.md](https://github.com/8none1/eufy-x8/blob/main/tools/CLAUDE.md) in the repo. There is also a `tools/` directory with standalone scripts for poking the robot manually, capturing goto coordinates, and so on — useful if you fancy reverse-engineering some more of it.

## The local key rotates

This was the first surprise. You authenticate once, fetch the local key, talk to the robot, everything works… until at some point a few hours later the robot decides to reconnect to the Eufy cloud and the cloud rotates its local key on the way out. Your next packet gets rejected with `InvalidKey` and the integration falls over.

The fix is to catch the `InvalidKey` exception, transparently re-authenticate to the Eufy cloud, fetch the new key, and retry the command. The integration does this automatically, so once it's set up it just keeps working. The only time you'll see it in the logs is on the line that says `Local key refreshed from cloud`.

## Goto coordinates and where to get them

The integration's `goto` service takes an (x, y) pair in the robot's persistent SLAM map. These are stable across reboots, firmware updates, and even battery changes — once you've captured a coordinate for a particular spot in your house, that coordinate will keep working until you reset the map. But neither the robot nor any cloud API will *tell* you what those coordinates are.

What you can do is sniff them off the wire. When you tap "Go to Location" in the Eufy app, the app sends the encrypted goto command to the robot directly over your LAN. `tools/intercept_goto.py` watches for that packet, decrypts it with your local key, and prints the (x, y) pair. You stash that in your automation YAML and use it forever.

(If you're tempted to use any other source of coordinates — path data, position telemetry, anything from the Tuya cloud — don't. The robot exposes several different coordinate spaces, most of them session-local, and feeding the wrong space into a goto sends the robot to a wholly unrelated point in your house. I know this because I tried it and it was funny but unproductive.)

## The thing I tried and gave up on: cleaning maps

The X8 doesn't expose its SLAM map directly the way the higher-end robots do. It clearly *has* one — the robot navigates intelligently, remembers where it's been, and accepts goto commands in a stable coordinate space — but none of that is offered to clients over either protocol.

What the Tuya cloud *does* offer is a `media.latest` endpoint that, on paper, returns recent path/position data for the robot. I spent a fairly significant chunk of time on this, hoping it would let me build an accumulating map entity for HA. The short version: it doesn't. The `media.latest v3.0` endpoint reliably returns exactly one record, the decoded fields don't form a recognisable map (one of them jumps by thousands of units between consecutive polls during a single clean), and what it actually encodes remains unknown. I monitored a full session — 112 polls, robot ran until the battery died — and got nothing usable out of it. The adjacent endpoints (`media.detail`, `images.tuyaeu.com`, `px.tuyaeu.com`) are all dead ends too.

I'm calling time on it for now. If anyone has cracked path or map data out of one of these robots, please tell me — there is a full write-up of everything I probed and what I got back in [tools/CLAUDE.md](https://github.com/8none1/eufy-x8/blob/main/tools/CLAUDE.md), so you don't have to repeat the dead-end work.

## The fun part: park-at-bin automation

Here's the thing the X8 *can't* do, but really should: when it finishes a clean and goes back to the dock, it should reposition itself somewhere convenient for you to empty the bin. Mine docks in a cupboard. Emptying it means getting on hands and knees, fishing it out, emptying it, then putting it back. If I could get it to park next to the kitchen bin instead, I'd save myself a minor back complaint every couple of days.

With the `goto` service this is straightforward. The pattern is:

1. After the robot finishes a clean and goes back to the dock, send a `goto` to the coordinates next to the kitchen bin
2. When it arrives, give a short beep so I know to come and empty it

That's two automations. There's one small wrinkle: the robot has no DPS field that says "I am at the bin" — it just has a generic "I have arrived somewhere" state. So if I ever add other `goto` commands in future, the beep would fire spuriously. The fix is a single `input_boolean` helper that automation 1 sets just before dispatching the goto, and automation 2 checks before beeping.

### Automation 1 — go to the bin when cleaning ends

Triggers when the granular status sensor transitions from "Cleaning" to "Returning to dock", and the robot has cleaned at least 25 m² (so a quick spot-clean or being moved off the dock doesn't fire it). It waits for the robot to actually dock before sending the goto, because the goto sequence is a `clear` + 35-second wait + `goto`, and sending that while the robot is still mid-journey gets ignored.

```yaml
alias: Downstairs vacuum - go to bin after clean
mode: single
trigger:
  - platform: state
    entity_id: sensor.downstairs_status
condition:
  - condition: template
    value_template: >
      {{ trigger.from_state.state | lower == 'cleaning'
         and trigger.to_state.state | lower == 'returning to dock' }}
  - condition: numeric_state
    entity_id: sensor.downstairs_cleaning_area
    above: 25
action:
  - wait_for_trigger:
      - platform: state
        entity_id: vacuum.downstairs
        to: docked
    timeout: "00:15:00"
    continue_on_timeout: false
  - service: input_boolean.turn_on
    target:
      entity_id: input_boolean.vacuum_headed_to_bin
  - service: eufy_x8.goto
    target:
      entity_id: vacuum.downstairs
    data:
      x: 2283   # your bin coordinates from intercept_goto.py
      y: -363
```

A note on the status sensor: the integration deliberately exposes two state-ish entities. The standard `vacuum.downstairs` entity uses Home Assistant's canonical vacuum states (`docked`, `cleaning`, `returning`, etc.), which is the right thing for HA's vacuum card to render. But for automations you want the more granular `sensor.downstairs_status` — it distinguishes "Cleaning", "Returning to dock", "Going to location", "Standby", and so on, which lets you write triggers that the canonical state would collapse together.

### Automation 2 — beep when the robot arrives at the bin

Triggers when the granular status sensor transitions from "Going to location" to "Standby" (= "I've arrived"), **and** the headed-to-bin flag is set. The beep is the `locate_brief` service, which I added specifically for this — the default locate beeps for nearly a minute, which is too long to be a useful notification.

```yaml
alias: Downstairs vacuum - beep when at bin
mode: single
trigger:
  - platform: state
    entity_id: sensor.downstairs_status
condition:
  - condition: template
    value_template: >
      {{ trigger.from_state.state | lower == 'going to location'
         and trigger.to_state.state | lower == 'standby' }}
  - condition: state
    entity_id: input_boolean.vacuum_headed_to_bin
    state: "on"
action:
  - service: input_boolean.turn_off
    target:
      entity_id: input_boolean.vacuum_headed_to_bin
  - service: eufy_x8.locate_brief
    target:
      entity_id: vacuum.downstairs
    data:
      duration: 5
```

Both automations do case-insensitive string comparisons on the state values, because Eufy have form for changing capitalisation between firmware revisions and I'd rather not have it break silently.

## What it can't do

In the interests of honesty:

- **Room cleaning is not supported.** Room-by-room cleaning on the X8 requires the AIOT MQTT path, which (as established) doesn't actually work for this device. The local protocol exposes a "room cleaning" command, but it returns "Failed" in every state I've tried. If anyone has cracked this, I'd love to hear from you.
- **No map.** As above — I tried, the path-data endpoint didn't give me anything usable, and I don't currently have a way forward.
- **The initial state has a delay.** Battery and status default to 0 / idle until the robot sends its first status push, which is normally within 1–2 minutes of waking. It's a one-time-per-HA-restart thing.

## How to get it

[github.com/8none1/eufy-x8](https://github.com/8none1/eufy-x8) — HACS-installable as a custom repository. The [README](https://github.com/8none1/eufy-x8/blob/main/README.md) has the full installation, configuration, and goto-coordinate-capture instructions. The protocol documentation in [tools/CLAUDE.md](https://github.com/8none1/eufy-x8/blob/main/tools/CLAUDE.md) is, I think, the most complete write-up of the X8's local protocol that exists publicly — feel free to use it as a starting point if you fancy adding support for another Tuya-platform robot vacuum.

It's been running both of mine — one downstairs (T2262), one upstairs (T2262EV) — for several weeks without manual intervention. Good enough for daily use, in my opinion. Bug reports welcome on the GitHub issue tracker.

---

*Acknowledgements: the protocol work builds on the older [eufy-clean](https://github.com/8none1/eufy-clean) project and the broader Tuya local protocol community. The integration ships its own self-contained Tuya v3.3 client; [tinytuya](https://github.com/jasonacox/tinytuya) was a useful reference while figuring out the framing, and the standalone scripts in `tools/` use it directly.*
