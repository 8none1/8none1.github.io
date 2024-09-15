# How to use the Octopus Power Ups and Free Electricity Session JSON objects in Home Assistant

## Octopus Power Ups and Free Electricity Session API

Octopus Power Ups and Octopus Free Electricity sessions don't, currently, have official API support.  Instead you get sent an email a day or so before the session starts with details of the times.  This is fine for normal people, but what if you've got a smart home with an electric car charger, or an electric immersion heater, or a large battery coupled with your solar system?  How can you automate actions like battery charging when the electricity is free?

I have created a set of Google Apps Script scripts which scrape the email and then convert it to a JSON object containing a standard `datetime` for the start and end of the sessions.  This JSON object is publicly available hosted on Github's infrastructure, and you can make free use of it to integrate in to Home Assistant.  You can read more about how that works here: (<https://www.whizzy.org/2024-01-24-powerups-api/>)
The script has correctly dealt with the changing formats used by Octopus in their emails, and we will with daylight savings, strange durations, etc.  It's been robust so far.

Everyone has different requirements from their Home Assistant automations, but I thought it would be worth writing up how I do it to demonstrate how straight forward it can be, and how flexible it is.  You will have to edit your `configuration.yaml` though.  But trust me, it's not scary.

## 1. The JSON object data source

The scripts mentioned above populate JSON files which are then published on the web.  You can fetch these files as often as you like, but they are only updated a maximum of once an hour - so there's really no point.  Every 15 minutes will be plenty.

There are two JSON files:

- (<https://www.whizzy.org/octopus_powerups/powerup.json>)
- (<https://www.whizzy.org/octopus_powerups/free_electricity_session.json>)

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

{% raw %}
```yaml
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
{% endraw %}

What does this do?  It creates a sensor which polls the URL containing the JSON object every 900 seconds.  It tries to extract `start` and `end` from the zero'th element (i.e. the first one) of the file hosted at that URL. That's it!  Every 15 minutes it will check to see if anything has changed, and if it has then an entity in Home Assistant called "Power Up Times" will have two attributes named `start` and `end`.  The contents of those attributes will be a string.  That string is a datetime formatted time as parsed from the Octopus email.  

You don't need to do anything to keep this updated.  Every 15 minutes Home Assistant will check and update itself if necessary.

So now you have the information in Home Assistant you need to be able to trigger automations with it.  So on to the next section.

## 3. Triggering an automation at the start of the session

In the previous section we saw how to get the information about the sessions in to a sensor in Home Assistant.  The next phase is to do something with that information.  To do that we need to create another sensor, this time a _binary sensor_ which will turn `ON` and `OFF` at the start and end of the session.  We will make use of Home Assistant's `template` feature to create a dynamic and automatic sensor which doesn't require any manual intervention, it simply turns on and off by itself.  You can detect this change of state in your own automations to trigger actions.  More on that later.  To start with we will create a new `binary sensor` in the `template` section of the `configuration.yaml` file.

If you already have a `template:` section in your configuration file then you should omit that line below.  If you don't, then you should be able to copy & paste this to the end of your `configuration.yaml` file and it should work.  Make sure the indentation is correction for you file if you already have a `template` section.

{% raw %}
```yaml
template:
  - binary_sensor
    - name: "Free Electricity Session In Progress"
      state: >
        {% set n = now() | as_timestamp %}
        {% set st  = state_attr('sensor.octopus_power_up_times', 'start') | as_timestamp %}
        {% set end = state_attr('sensor.octopus_power_up_times', 'end')   | as_timestamp %}
        {% if n >= st and n < end %}
          True
        {% else %}
          False
        {% endif %}
```
{% endraw %}

To explain what this does, let's look at it line by line.  It's a `template` sensor and so uses the [Jinja2](https://palletsprojects.com/projects/jinja) syntax.  `state: >` says that the state of this sensor (it's a binary sensor, and so the state can only be ON or OFF) is set by a multiline script following.  The script first creates a variable `n` which is set to the value of `now()` (the current number of seconds since the [epoch](https://en.wikipedia.org/wiki/Unix_time)).
It then sets two other variables called `st` and `end`.  The values of these variables is the `start` string from the sensor we created above parsed as a `timestamp`.  The parsing is done for us by Jinja.
The result is that we have three variables each containing a number.  The value of that number is a time represented as a number of seconds since a fixed point in the past.

The script then compares `n` (the current time) to `st` and `end`.  If the value of `n` is greater than or equal to `st` AND less than `end` (i.e. the current time falls between the start and end of the session times) then the state is `True` (i.e. ON) otherwise the value is `False` (i.e. OFF).

That's it!  A sensor which switches between `ON` and `OFF` automatically when the current time is inside the bounds of the free electricity session.  Home Assistant knows that the script is using `now()` and so evaluates the script every second automatically.  You can use this change of state to trigger your automations without having to manually set anything.  It will just quietly update itself every second and you don't have to do a thing.

In your Home Assistant automation add a trigger "State. When the state of an entity (or attribute) changes."  Select the "Free Electricity Session In Progress" entity, and set From to "Off" and To to "On".  That trigger will fire at the start of every session.

PICTURES

## 4. Adding more information to the binary sensor

So far we have created a simple on/off sensor that can be used to trigger automations when it's state changes.  This is useful for a lot of automations, but what if we need to know some more information up front so we can configure external hardware.  For example, if you have a programmable electric car charger it might be useful to tell it how long to run for when the sensor triggers an automation.  We can work out that information using more Jinja scripting.

Below is a script which builds on the previous example.

{% raw %}
```yaml
template:
  - binary_sensor
    - name: "Free Electricity Session In Progress"
      state: >
        {% set n = now() | as_timestamp %}
        {% set st  = state_attr('sensor.octopus_power_up_times', 'start') | as_timestamp %}
        {% set end = state_attr('sensor.octopus_power_up_times', 'end')   | as_timestamp %}
        {% if n >= st and n < end %}
          True
        {% else %}
          False
        {% endif %}
      attributes:
        duration_mins: >
          {% set st  = state_attr('sensor.octopus_power_up_times', 'start') | as_timestamp %}
          {% set end = state_attr('sensor.octopus_power_up_times', 'end')   | as_timestamp %}
          {{ ((end - st) / 60) | int }}
        duration_remaining: >
          {% if this.state == 'on' %}
            {% set n = now() | as_timestamp %}
            {% set end = state_attr('sensor.octopus_power_up_times', 'end') | as_timestamp %}
            {{ ((end - n) / 60) | int }}
          {% else %}
            {{ False }}
          {% endif %}
        start_time: "{{state_attr('sensor.octopus_power_up_times', 'start') | as_datetime }}"
        end_time: "{{state_attr('sensor.octopus_power_up_times', 'end') | as_datetime }}"
```
{% endraw %}


You can see the `state` section is the same, but we have added a new `attributes` section.

`duration_mins` converts the start and end times to timestamps then subtracts one from the other and divides by 60.  This gives us the duration of the session in minutes.

`duration_remaining` calculates the number of minutes from `now()` to the end of the session in minutes.  Home Assistant will update this value every minute automatically for us.  If the session is not currently active then it will return False instead.

Finally `start_time` and `end_time` are parsed by Jinja and converted in to datetime format.  This means that these attributes contain the start and end times not as a string, but as a properly formatted datetime in your local timezone or daylight savings period.  This converts from the UTC timestamp _string_ in the JSON object in to a proper Home Assistant time zone aware datetime.  Easy!

You can access these attributes in your own scripts by using the `state_attr` function.  e.g.

{% raw %}
```yaml
{{ state_attr('binary_sensor.octopus_power_up_times', 'duration_remaining') }}
```
{% endraw %}

will return the number of minutes remaining in the current session.  You can test this in the Home Assistant Developer Tools section, under "Template".
