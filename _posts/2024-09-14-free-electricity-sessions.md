# How to use the Octopus Power Ups and Free Electricity Session JSON objects in Home Assistant

## Octopus Power Ups and Free Electricity Session API

Octopus Power Ups and Octopus Free Electricity sessions don't, currently, have official API support.  Instead you get sent an email a day or so before the session starts with details of the times.  This is fine for normal people, but what if you've got a smart home with an electric car charger, or an electric immersion heater, or a large battery coupled with your solar system?  How can you automate actions like battery charging when the electricity is free?

I have created a set of Google Apps Script scripts which scrape the email and then convert it to a JSON object containing a standard `datetime` for the start and end of the sessions.  This JSON object is publicly available hosted on Github's infrastructure, and you can make free use of it to integrate in to Home Assistant.  You can read more about how that works here: (https://www.whizzy.org/2024-01-24-powerups-api/)
The script has correctly dealt with the changing formats used by Octopus in their emails, and we will with daylight savings, strange durations, etc.  It's been robust so far.

Everyone has different requirements from their Home Assistant automations, but I thought it would be worth writing up how I do it to demonstrate how straight forward it can be, and how flexible it is.  You will have to edit your `configuration.yaml` though.  But trust me, it's not scary.

## 1. The JSON object data source

The scripts mentioned above populate JSON files which are then published on the web.  You can fetch these files as often as you like, but they are only updated a maximum of once an hour - so there's really no point.  Every 15 minutes will be plenty.

There are two JSON files:
 - (https://www.whizzy.org/octopus_powerups/powerup.json)
 - (https://www.whizzy.org/octopus_powerups/free_electricity_session.json)

The first one contains information about Power Ups in the eastern part of the [UK Power Networks](https://www.ukpowernetworks.co.uk/) region.  If you live in "the east of England" then you're /probably/ in this area.  I live in Bedfordshire, and my friend in Suffolk usually has the same Power Ups as me, but there has been one instance where he had one and I didn't.  *Note:*  if you use this data source to get information about Power Ups you must still complete the form in the email from Octopus in order to qualify.  This is a requirement of Octopus.

The second file contains details of Octopus Free Electricity Sessions.  These are nationwide and do not need you to complete a form.  Once you're registered with Octoplus you get the Free Electricity Session.

Most of the time the JSON objects at these URLs will contain `null` data, because there isn't a Power Up or Free Electricity Session known.  As soon as one (or more) is known about, then the JSON object will be a list of JSON objects.  Each object contains two keys: `start` and `end`.

Here is an example:

```json
[{"start":"2024-09-14T12:00:00.000Z","end":"2024-09-14T13:00:00.000Z"}]
```

As you can see, this is a list with one element.  The element contains the start time and end time `datetime` strings in the UTC timezone.  The timing picked up from the Octopus emails is converted to UTC.  This makes dealing with daylights savings a lot easier, and it's also easy to deal with this in Home Assistant.
In the case where there are multiple sessions the list will contain an element for each session up to a maximum of three.  They will always be ordered from soonest to latest.  When one session ends it will be removed from the list within one hour.  When there are no sessions left it will revert to `null` entries.

The Home Assistant sensors detailed below can be used for either data source. If you want to use both, then you should create one full set of sensors for each data source, with some tweaks like changing the name etc.

## 2. Polling the information from the web and importing it into a Home Assistant sensor

The sensor definition below creates a `REST` sensor which polls the URL above every 900 seconds.  It extracts two pieces of information; the `start` and `end` times as shown above.  This sensor on it's own is very useful, it is used primarily to expose the session data in to Home Assistant where it can be used by other things (more on this later).  It takes the first element in the list, so the soonest session.

Open your `configuration.yaml` file in a text editor or use the Home Assistant configuration editor if you have it installed.  You can find more information about the `configuration.yaml` file on [this page](https://www.home-assistant.io/docs/configuration/) in the Home Assistant docs.

Here is the full sensor definition.  If you already have a `sensor:` section in your configuration file, then omit the `sensor:` line and add the rest of this sensor definition to your existing `sensor:` section, making sure the indentation is consistent with your existing file.  (If you already have manual sensors defined in your configuration file then this will make sense to you.  If you don't then you can just copy & paste this to the end of the file and it will probably just work.)

```text
sensor:
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

What does this do?  It creates a sensor which polls the URL containing the JSON object every 900 seconds.  It tries to extract `start` and `end` from the zero'th element (i.e. the first one) of the file hosted at that URL. That's it!  Every 15 minutes it will check to see if anything has changed, and if it has then an entity in Home Assistant called "Power Up Times" will have two attributes named `start` and `end`.  The contents of those attributes will be a string.  That string is a datetime formatted time as parsed from the Octopus email.  

You don't need to do anything to keep this updated.  Every 15 minutes Home Assistant will check and update itself if necessary.

So now you have the information in Home Assistant you need to be able to trigger automations with it.  So on to the next section.

## 3. Triggering an automation at the start of the session

