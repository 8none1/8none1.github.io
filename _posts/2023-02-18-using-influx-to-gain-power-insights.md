---
layout: post
title: InfluxDB and Growatt SPH inverters
date: '2023-02-18T12:18:00+01:00'
author: will
categories:
    - 'Making the world a better place'
highlighter: none
---

# Using data for fun and profit

## Electricity prices

In the UK wholesale electricity prices are tightly coupled to gas prices.  This means that we are currently paying around 40p per kWh, despite wind often providing more electricity than gas.  The whole system is fundamentally flawed with little political will to fix it. A good way to try and beat the system is to install solar panels.

For around £16,000 you could install a 9kWp solar system and around 13 kWh of battery storage.  On my east facing array in December, solar was able to provide around 30% of my electricity needs.  When coupled with a battery, one could import off-peak electricity to fill most of the gap. Some days would be better than others, but it should all work out in the end.  For around half the year I expect to be importing zero electricity from the grid during peak times. In fact I expect to be a net exporter of electricity from around May to September but I haven't had the panels up long enough to know for sure. 

Solar panel installs currently benefit from zero VAT.  This is a good thing of course, but imagine if the government gave every household a £5000 grant for a solar panel system of any size.

## Gathering the data - Growatt SPH Inverters

Growatt are manufacturers of well priced, decently spec'd solar inverters and batteries.  They do have some representation in the UK and their warranty period is good, often 10 years or more.  How easy it will be to claim on this warranty remains to be seen, but so far I have no concerns.

One of the nice things about the Growatt inverters is the management system.  You purchase a wifi dongle which connects to the inverter and sends the data off to Growatt.  You can then log in to a web based system and get graphs of all the metrics you need and apply configuration changes remotely.  They also provide an app.

People have created custom firmware for these dongles to allow you to output that data to MQTT for use in your own system.  Check out this Github project for more information:  [https://github.com/octal-ip/ESP07_Growatt_SPF_3500-5000_ES_Monitor](https://github.com/octal-ip/ESP07_Growatt_SPF_3500-5000_ES_Monitor)
I think it's specific to the SPF line of inverters but could be easily adapted to support SPH inverters if needed.

I have taken a different route.  The inverter also includes an RS485 port which you can use at the same time as the wifi dongle.  I purchased an Elfin EW11 wifi to RS485 bridge and wrote a small integration for Node-RED to gather the metrics I need.  You can read more information about that project in this Github repo:  [https://github.com/8none1/growatt_sph_nodered](https://github.com/8none1/growatt_sph_nodered)

The end result is that I have my metrics being gathered every 30 seconds from the inverter, converted in to a JSON payload and sent to MQTT.  The I use Telegraf to collect the metrics and send them in to InfluxDB Cloud 2.

Once in InfluxDB I can build dashboards to visualise the data in the way that I want to see it.  This dashboard is currently built in the Dashboard system built in to InfluxDB Cloud.

![Dashboard Screenshot](/assets/img/solar_dashboard.png)

(I will be porting it to a local Grafana instance at some point, and attempting to port the Flux code to SQL or InfluxQL, probably InfluxQL since the time-series specific functions like `integral` are much easier in InfluxQL).

## Processing the data

Octopus Energy are a progressive energy company who offer a number of tariffs which have the potential to save you money.  You have to do your homework first though.
A couple of tariffs of interest to me are Agile Octopus and Octopus Flux.

Agile gives you advance warning of the cost of electricity broken down in to 30 min blocks each day.  If you have a battery or solar panels and are able to avoid using power during the peak periods you could bring the average cost per kWh down to something like 29p per kWh (saving about 25%).

Flux gives you 3 hours off-peak (2am to 5am) of electricity at 21p kWh, a normal rate of 35p kWh and a super-peak (4pm to 7pm) at 49p kWh.  If you can charge your battery during the off-peak time and avoid the super-peak period altogether, maybe this will work out cheaper in the long run?  Flux also gives very good export rates, but I'm not factoring that in just yet.  Octopus seem to let you change between tariffs without protracted lock-in periods.

My expectation is that these tariffs will have been designed so that they are very closely matched in cost to the average consumer. But, there's only one way to find out which is best for me.

In order to predict how much my bill will be on Agile I need to break my power usage down in to 30 minute chunks throughout the day.  I sample my power usage every 30 seconds, so I need to pick a day of typical energy usage, group the data in to 30 minute windows, and using [Riemann Sum](https://en.wikipedia.org/wiki/Riemann_sum) convert that in to kWh per 30 minutes.  I can then download the 30 minute prices from Octopus and hopefully come to a total for the day.

Since my data is already in InfluxDB Cloud, I will make use of the [integral](https://docs.influxdata.com/flux/v0.x/stdlib/universe/integral/) function in Flux to make this easier.

The function I came up with is below.  This is using the JSON data from my Node-RED function (linked to above) so your field names might be different.
This is complicated somewhat by me having two inverters.  Something to be aware of with the SPH inverters is that they don't count energy used to charge the battery from the grid in their "local load" calculations.  Also, if the local load is being supplied by the batteries the second inverter without the battery is unaware of that load.

This graph may explain it better:

![Local Load Graph](/assets/img/local_load.png)

The blue and purple lines are for each of the two inverters.  The blue line is the one without the battery, and it sees the battery being charged between 00:30 and 04:00 ish (this just happens to be when I charge the battery).  During that same period the purple inverter only reports the load from the house, and removes the battery charging power from it's measurements.  Once the local load is being supplied from the battery or the solar panels on the "purple" inverter the "blue" inverter can't see the load and reports zero.
Once both inverters are supplying power things get even more confusing.
I can't get perfect data using only my inverter metrics, but it's close enough.  I take the maximum of the purple and blue inverters and use that.  

## Convert instantaneous power readings in to energy usage using InfluxDB

```
from(bucket: "<your bucket>")
    |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
    |> filter(fn: (r) => r._measurement == "solar")
    |> filter(fn: (r) => r._field == "pLocalLoadTotal")
    |> truncateTimeColumn(unit: 30s) // The timestamps on the data points are about 15 seconds apart as first one inverter is read, and then the other. This rounds them to the nearest 30 seconds and gets them in sync.
    |> group(columns: ["_time"]) // Now the time stamps match, we group the data in to lots of small tables each containing two points, one for each inverter for each normalised sample point
    |> max() // and take the biggest of the two.
    |> group() // We re-group in to one table
    |> drop(columns: ["host", "serialNumber", "topic"]) // and remove data we don't need to shrink the size a bit.
    |> window(every: 30m) // Now we re-divide the tables in to 30 minute chunks as used by Octopus
    |> integral(unit: 60m) // and calculate the total Wh used during that period with `integral`
    |> group() // re-group in to a single table
    |> map(fn: (r) => ({r with _value: r._value / 1000.0})) // and convert watts to kWh
```


## Next steps

Now I have the data I can easily create a spreadsheet to calculate the costs for a typical day.  I'll write that up next time.
Meanwhile, I like Tim's videos on YouTube creating a spreadsheet to end all spreadsheets for Octopus calculations.  At the time of writing this is most up-to-date one:  [https://www.youtube.com/watch?v=iLFrykG49Uk](https://www.youtube.com/watch?v=iLFrykG49Uk)

