---
layout: post
title: "Octopus Energy Free Electricity Sessions: Official GraphQL API!"
date: 2025-11-10
tags: [octopus-energy, home-assistant, api, graphql, automation]
social-share: true
---

## TL;DR

After months of scraping emails to track Octopus Energy's Free Electricity Sessions, I've discovered an **official GraphQL API** that provides reliable data. This is a significant improvement over regex text parsing for my Home Assistant automation.

**Key Updates:**
- ✅ **Free Electricity Sessions**: Official API available
- ⚠️ **Power Ups**: API exists but currently returns what looks like placeholder data
- 🔄 **Running both systems in parallel** during transition period - no action needed on your part

## The Email Scraping Era is Over (for Free Electricity Sessions)

Back in [January 2024](https://www.whizzy.org/2024-01-24-powerups-api/), I built a Google Apps Script solution to scrape Octopus Energy Power Up emails because there was no API. Then in [September 2024](https://www.whizzy.org/2024-09-14-free-electricity-sessions/), I extended this to cover Free Electricity Sessions.

While that solution worked, it had limitations:
- Fragile regex patterns that break when email format changes
- Dependent on email delivery timing
- Required Google Apps Script infrastructure
- Manual maintenance when things broke

But now there is an API for FES.

## Discovering the Official Octopus Energy GraphQL API

Octopus Energy has a **production GraphQL API** at `https://api.octopus.energy/v1/graphql/` that includes official support for flexibility campaigns including Free Electricity Sessions.

### What I Found

The API provides a `customerFlexibilityCampaignEvents` query that returns structured data for:
- **Free Electricity Sessions** (`free_electricity` campaign) - ✅ **Working perfectly**
- **Power Ups UKPN** (`power_ups_ukpn` campaign) - ⚠️ Data quality issues
- **Power Ups NGED** (`power_ups_nged` campaign) - ⚠️ Data quality issues  
- **Power Ups NPG** (`power_ups_npg` campaign) - ⚠️ Data quality issues
- **Power Ups Scotland** (`power_ups_scotland` campaign) - ⚠️ Data quality issues

### Free Electricity Sessions API: Production Ready

The Free Electricity Sessions API is **reliable and accurate**. It returns events with:
- Start time (ISO 8601 format)
- End time (ISO 8601 format)
- Event code
- Automatic filtering for future events only

**This is now the recommended method** for tracking Free Electricity Sessions!

### Power Ups API: Still in Development?

While exploring the API, I discovered that Power Ups *technically* have API support, but the data quality is questionable:

- ❌ Returns 24-hour windows (00:00 to 00:00 next day) instead of specific hours
- ❌ Missing historical events (e.g., my last real Power Up on Sept 5th, 2025 at 12-2pm BST doesn't appear)
- ❌ Events look like placeholders rather than real scheduling data

For now, **Power Ups will continue using the email scraping method** until the API data improves.  There hasn't been a Power Up for a while so next time one comes along we will be able to see what happens.

## Authentication: API Key Only

The new implementation uses **API key authentication** - no email/password required. You can generate an API key from your Octopus Energy account dashboard.

The system automatically discovers:
- Your account number
- Your property's MPAN (electricity meter point)
- Correctly filters import vs export meters (important for solar panel users)

**Just one environment variable needed**: `OCTOPUS_API_KEY`

## New JSON Endpoints

I'm running **both systems in parallel** during this transition:

### Free Electricity Sessions

**New (Recommended)**: GraphQL API-based
```
https://www.whizzy.org/octopus_powerups/free_electricity_session_graphql.json
```

**Legacy**: Email scraping
```
https://www.whizzy.org/octopus_powerups/free_electricity_session.json
```

### Power Ups (UKPN)

**GraphQL** (experimental - placeholder data):
```
https://www.whizzy.org/octopus_powerups/powerup_graphql.json
```

**Recommended**: Email scraping
```
https://www.whizzy.org/octopus_powerups/powerup.json
```

**All files use the same JSON format** for drop-in compatibility!

## Home Assistant Integration with Octopus Energy API

Here's how to integrate the new official Octopus Energy Free Electricity Sessions API with Home Assistant:

### Step 1: Add REST Sensor for Free Electricity Sessions

Update your `sensors.yaml`:

```yaml
- platform: rest
  name: "Free Electricity Session Times"
  unique_id: octopus_free_electricity_graphql
  resource: "https://www.whizzy.org/octopus_powerups/free_electricity_session_graphql.json"
  scan_interval: 900
  json_attributes_path: "$.[0]"
  json_attributes:
    - start
    - end
    - code
```

This sensor polls the official Octopus Energy API data every 15 minutes.

### Step 2: Create Binary Sensor to Track Active Sessions

Add to your `configuration.yaml`:

{% raw %}
```yaml
template:
  - binary_sensor:
      - name: "Free Electricity Session In Progress"
        unique_id: free_electricity_active
        state: >
          {% set n = now() | as_timestamp %}
          {% set st  = state_attr('sensor.free_electricity_session_times', 'start') | as_timestamp %}
          {% set end = state_attr('sensor.free_electricity_session_times', 'end')   | as_timestamp %}
          {% if n >= st and n < end %}
            True
          {% else %}
            False
          {% endif %}
        attributes:
          duration_mins: >
            {% set st  = state_attr('sensor.free_electricity_session_times', 'start') | as_timestamp %}
            {% set end = state_attr('sensor.free_electricity_session_times', 'end')   | as_timestamp %}
            {{ ((end - st) / 60) | int }}
          duration_remaining: >
            {% if this.state == 'on' %}
              {% set n = now() | as_timestamp %}
              {% set end = state_attr('sensor.free_electricity_session_times', 'end') | as_timestamp %}
              {{ ((end - n) / 60) | int }}
            {% else %}
              {{ 0 }}
            {% endif %}
          start_time: "{{state_attr('sensor.free_electricity_session_times', 'start') | as_datetime }}"
          end_time: "{{state_attr('sensor.free_electricity_session_times', 'end') | as_datetime }}"
          event_code: "{{state_attr('sensor.free_electricity_session_times', 'code')}}"
```
{% endraw %}

This creates a binary sensor that:
- Turns **ON** when a Free Electricity Session starts
- Turns **OFF** when the session ends
- Provides additional attributes like duration and remaining time

### Step 3: Create Home Assistant Automations

Trigger automations based on the Octopus Energy Free Electricity Session:

```yaml
automation:
  - alias: "Charge Battery During Free Electricity"
    trigger:
      - platform: state
        entity_id: binary_sensor.free_electricity_session_in_progress
        from: 'off'
        to: 'on'
    action:
      - service: switch.turn_on
        entity_id: switch.battery_charger
      - service: notify.mobile_app
        data:
          message: "Free Electricity Session started - charging battery!"
          
  - alias: "Stop Charging After Free Electricity"
    trigger:
      - platform: state
        entity_id: binary_sensor.free_electricity_session_in_progress
        from: 'on'
        to: 'off'
    action:
      - service: switch.turn_off
        entity_id: switch.battery_charger
      - service: notify.mobile_app
        data:
          message: "Free Electricity Session ended"
```

### For Power Ups (UKPN)

Keep using the existing email-based endpoint until the API improves:

```yaml
- platform: rest
  name: "Power Up Times"
  unique_id: octopus_power_up_times
  resource: "https://www.whizzy.org/octopus_powerups/powerup.json"
  scan_interval: 900
  json_attributes_path: "$.[0]"
  json_attributes:
    - start
    - end
```

The binary sensor configuration remains identical to my [previous guide](https://www.whizzy.org/2024-09-14-free-electricity-sessions/).

## How It Works

1. **GitHub Actions** runs Python scripts every 6 hours
2. Scripts authenticate with Octopus Energy GraphQL API using API key (stored as a Github secret)
3. Auto-discover account number and MPAN based on the user who created the API key
4. Query `customerFlexibilityCampaignEvents` for future events
5. Write JSON files to repository
6. Files automatically committed and hosted via GitHub Pages
7. Home Assistant polls JSON files every 15 minutes

## Technical Deep Dive: The GraphQL API

For the technically curious, here's the GraphQL query used:

```graphql
query GetFreeElectricitySessions($accountNumber: String!, $mpan: String!) {
  customerFlexibilityCampaignEvents(
    accountNumber: $accountNumber,
    supplyPointIdentifier: $mpan,
    campaignSlug: "free_electricity",
    first: 10
  ) {
    edges {
      node {
        code
        startAt
        endAt
      }
    }
  }
}
```

The API was added to Octopus Energy's GraphQL schema on **May 29, 2025** according to their [changelog](https://developer.octopus.energy/graphql/changelog/). This explains why it's so new and perhaps why Power Ups data quality is still being refined.

## Migration Guide: Switching to the New API

If you're using my previous email-based solution:

1. **Update your REST sensor URL** to point to `free_electricity_session_graphql.json`
3. **Keep existing binary sensors** - they work with both data sources
4. **Monitor both endpoints** for a few weeks to verify reliability
5. **For Power Ups**: Continue using the existing email-based endpoint

The JSON format is **100% compatible**, so no changes to your automations needed!

## Open Source and Community

All code is available on GitHub:
- Repository: [https://github.com/8none1/octopus_powerups](https://github.com/8none1/octopus_powerups)
- Python scripts in the `graphql/` directory
- Documentation of API discovery process
- GitHub Actions workflow for automatic updates

## What's Next?

I'm monitoring the Power Ups API data quality and will switch to it once Octopus Energy populates it with real event data instead of placeholders. 

In the meantime:
- ✅ Free Electricity Sessions: **Use GraphQL API**
- ⚠️ Power Ups: **Use email scraping**

## Conclusion

This is a huge improvement for anyone automating Octopus Energy flexibility events with Home Assistant! The official GraphQL API provides reliable, structured data for Free Electricity Sessions, eliminating the fragility of email scraping.

While Power Ups API support exists, the data quality needs improvement before I'd recommend using it. For now, we'll run both systems in parallel and keep you updated as the API matures.

**Happy automating!** 🔋⚡

---

**Links:**
- JSON Files: [Free Electricity (GraphQL)](https://www.whizzy.org/octopus_powerups/free_electricity_session_graphql.json) | [Power Ups](https://www.whizzy.org/octopus_powerups/powerup.json)
- GitHub: [octopus_powerups](https://github.com/8none1/octopus_powerups)
- Previous Posts: [Power Ups API](https://www.whizzy.org/2024-01-24-powerups-api/) | [Free Electricity Sessions](https://www.whizzy.org/2024-09-14-free-electricity-sessions/)
- [Octopus Energy](https://octopus.energy/)
- [Home Assistant Octopus Energy Integration](https://github.com/BottlecapDave/HomeAssistant-OctopusEnergy)

---

*Keywords: Octopus Energy, Free Electricity Sessions, API, GraphQL, Home Assistant, automation, Power Ups, UKPN, smart home, energy management, REST sensor, binary sensor*
