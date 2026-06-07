---
layout: post
title: "The hidden Home Assistant setting that frees your Bluetooth proxy connection slots"
date: 2026-06-07
tags: [home-assistant, bluetooth, ble, esphome, bluetooth-proxy, led-ble, polling]
social-share: true
excerpt: "My BLE light controllers were each holding a Bluetooth proxy connection open forever, quietly starving everything else of slots. The fix turned out to be a setting that already ships in Home Assistant, does exactly the right thing, and is almost impossible to find. Here's why you'd want it, why it hides, and three ways to switch it on."
---

## TL;DR

If you control Bluetooth devices through ESPHome Bluetooth proxies, some integrations connect once and then never let go, holding a precious proxy connection slot open forever. There is a built-in, per-device Home Assistant setting that fixes this: turn off **"Enable polling for changes"** on the config entry. The integration stops polling, the underlying library is finally allowed to drop the idle connection, and the slot is freed for something else. Commands reconnect on demand. The catch is that the toggle is buried behind Advanced Mode in a "System options" dialog that, depending on your frontend version, can be genuinely hard to find. If you can't find it, you can flip it via the WebSocket API instead.

## The problem: proxies have very few slots

[Bluetooth proxies](https://esphome.io/components/bluetooth_proxy.html) are one of the best things to happen to Home Assistant in years. A cheap ESP32 in a far corner of the house relays BLE advertisements and connections back to HA, so your Bluetooth devices no longer have to be within a few metres of the server. I have several dotted around.

The thing nobody tells you up front is how few *active connections* a proxy can hold at once. An ESP32 can typically manage three simultaneous BLE connections. Three. That's per proxy. Advertisements (passive data, like a thermometer broadcasting its reading) are effectively unlimited and cheap, but anything that needs a real connection (a light you send colour commands to, a lock you operate) occupies one of those three slots for as long as it stays connected.

So the question that matters is: when an integration connects to a device, **does it disconnect afterwards?**

For a lot of my kit, the answer was no.

## The discovery: a disconnect timer that never fires

I have a handful of cheap BLE LED controllers, the "Triones" sort, handled by the core [LED BLE](https://www.home-assistant.io/integrations/led_ble/) integration. I noticed my proxies were permanently at capacity, and these lights were the culprits. Each one was holding a connection open 24/7, even when I hadn't touched it in days.

What's interesting is that the underlying library, [`led-ble`](https://github.com/Bluetooth-Devices/led-ble), already does the right thing. It has an idle disconnect timer baked in:

```python
DISCONNECT_DELAY = 120
```

After two minutes of inactivity it tears the connection down on its own. Great. So why was my connection up forever?

Because the integration polls. The LED BLE coordinator asks each light for its current state every 15 seconds:

```python
update_interval = timedelta(seconds=UPDATE_SECONDS)  # 15
```

and every poll connects (if not already connected), which resets that 120-second disconnect timer. Fifteen is comfortably less than 120, so the timer is reset eight times before it could ever fire. The connection never goes idle long enough to be dropped. The poll is, in effect, a keep-alive I never asked for.

That's the whole bug in one sentence: **a library that would happily disconnect is prevented from doing so by a poll loop running faster than its idle timeout.** This pattern is not unique to LED BLE. A lot of the excellent `bleak`-based device libraries in the HA ecosystem follow exactly this shape, so if your proxies are full, this is worth understanding generally.

## Why you'd want to turn polling off

The fix is to stop polling. If the 15-second poll goes away, nothing resets the disconnect timer, the library drops the connection after two idle minutes, and the slot goes back into the pool. When you next send a command, the library reconnects on demand, does its thing, and disconnects again two minutes later.

The trade-off is real and worth stating plainly:

- **You lose background state refresh.** While disconnected, Home Assistant won't notice if you change the light some other way (a physical remote, the vendor's app). The state catches up the next time HA connects to send a command. For a light I only ever drive *from* Home Assistant, I do not care even slightly.
- **There's a small reconnect delay** on the first command after an idle period, maybe a second or two while it establishes the connection. Again, for a light, fine.

In exchange, a device that was hogging a slot 24/7 now uses one only for the couple of minutes around when you actually talk to it. Multiply that across several devices and several proxies and you go from "permanently full" to "loads of headroom".

## The genuinely interesting part: you don't need a code change

My first instinct was that this needed a patch to the integration: add an option, wire up the disconnect, propose a PR. I started down that road and then found something better.

Home Assistant's `DataUpdateCoordinator`, the base class that drives polling for a huge fraction of integrations, **already honours a per-config-entry setting called `pref_disable_polling`**. Look at the scheduler in `homeassistant/helpers/update_coordinator.py`:

```python
def _schedule_refresh(self) -> None:
    if self._update_interval_seconds is None:
        return
    if self.config_entry and self.config_entry.pref_disable_polling:
        return  # no next poll is scheduled
    ...
```

If that flag is set, the next poll is simply never scheduled. No code change, no custom option, no PR. The capability is generic and already shipping for any coordinator-based integration. LED BLE inherits it for free. The reason I'd never noticed is that the setting is, frankly, hidden.

I verified the whole chain on my live instance. With debug logging on and polling disabled on one light, the log tells the story perfectly:

```
11:03:01  Connected; RSSI: -84              <- single connect on reload
11:03:01  Finished fetching ... (success)   <- one refresh, then silence
   ... 120 seconds of nothing ...
11:05:01  Disconnecting after timeout of 120
11:05:01  Disconnected from device          <- connection released, slot freed
```

One connect, one refresh, two minutes of silence, then the library lets go. Exactly what I wanted, and nothing I had to write.

## Why it's so hard to discover

If this is built in and so useful, why does nobody know about it? Three reasons stack up.

1. **It's an Advanced Mode setting.** It does not appear at all unless you've enabled Advanced Mode in your user profile (click your name at the bottom of the sidebar, toggle it on). Most people never do.
2. **It lives in "System options", not the obvious place.** It is not on the device page, where you'd naturally look (that page only offers Rename, Disable, Delete). It's on the *config entry* row, behind a three-dot menu, in a sub-dialog called "System options". For an integration like LED BLE that creates one config entry per device, that's one more layer of "which thing am I clicking on" than you'd expect.
3. **The UI has moved around.** I'm on a recent build (2026.4) where, even with Advanced Mode on, I could not find the "System options" entry at all. The toggle is unconditionally present in the frontend source, so this looks like a layout/regression quirk between versions. The upshot is that the one route the documentation points you at may not exist in your particular build.

So you have an extremely useful, generic, already-implemented setting that is gated behind a mode most users don't enable, tucked into a dialog most users never open, in a spot that occasionally goes missing. No wonder proxies fill up.

## How to enable it

### The intended way (UI)

1. Profile (your name, bottom of the sidebar) and turn on **Advanced Mode**.
2. **Settings → Devices & Services → Integrations**, and click the integration's tile (not the Devices tab).
3. On the specific config entry's three-dot menu, choose **System options**.
4. Turn off **Enable polling for changes**. The entry reloads itself; no restart needed.

### The reliable way (WebSocket API)

If the menu item isn't there, or you have a lot of devices to do, go straight to the API that toggle calls anyway: the `config_entries/update` WebSocket command. It's admin-only, takes a `pref_disable_polling` boolean, and reloads the entry for you.

You'll need a [long-lived access token](https://www.home-assistant.io/docs/authentication/#your-account-profile) (profile, bottom of the page). Then a few lines of Python with the `websockets` package:

```python
import asyncio, json, websockets

URL = "ws://homeassistant.local:8123/api/websocket"
TOKEN = "YOUR_LONG_LIVED_TOKEN"

async def main():
    async with websockets.connect(URL) as ws:
        await ws.recv()  # auth_required
        await ws.send(json.dumps({"type": "auth", "access_token": TOKEN}))
        assert json.loads(await ws.recv())["type"] == "auth_ok"

        # List config entries so you can find the entry_id you want
        await ws.send(json.dumps({"id": 1, "type": "config_entries/get"}))
        while (msg := json.loads(await ws.recv())).get("id") != 1:
            pass
        for e in msg["result"]:
            if e["domain"] == "led_ble":
                print(e["entry_id"], e["title"], e["pref_disable_polling"])

        # Then disable polling on one of them
        await ws.send(json.dumps({
            "id": 2,
            "type": "config_entries/update",
            "entry_id": "PASTE_ENTRY_ID_HERE",
            "pref_disable_polling": True,
        }))
        while (msg := json.loads(await ws.recv())).get("id") != 2:
            pass
        print("done:", msg["success"])

asyncio.run(main())
```

Run it once per device. To reverse any of this, send the same command with `pref_disable_polling: False`.

### Worth knowing

- This is per config entry. If an integration bundles many devices into one entry, the switch is all-or-nothing for that entry.
- It is not specific to Bluetooth or to LED BLE. Any coordinator-based polling integration respects it, so it's a general lever for rate-limited cloud APIs too, not just proxy slots.
- The [official docs](https://www.home-assistant.io/docs/configuration/platform_options/) mention disabling polling, but it's easy to read past how much it can do for a constrained Bluetooth setup.

## Conclusion

The most satisfying outcome of debugging something is finding that the fix already exists and you just have to *turn it on*. My proxies went from permanently full to mostly idle by flipping one boolean per light, with no custom code, no fork, no PR. The only real problem with `pref_disable_polling` is how well Home Assistant has hidden one of its more useful settings. If your Bluetooth proxies are wedged at capacity, go and find it. Then, if you're like me, go and find it again via the API because the menu won't be where the docs say it is.
