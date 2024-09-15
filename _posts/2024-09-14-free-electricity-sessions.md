# How to use the Octopus Power Ups and Free Electricity Session JSON objects in Home Assistant

## Octopus Power Ups and Free Electricity Session API

Octopus Power Ups and Octopus Free Electricity sessions don't, currently, have official API support.  Instead you get sent an email a day or so before the session starts with details of the times.  This is fine for normal people, but what if you've got a smart home with an electric car charger, or an electric immersion heater, or a large battery coupled with your solar system?  How can you automate actions like battery charging when the electricity is free?

I have created a set of Google Apps Script scripts which scrape the email and then convert it to a JSON object containing a standard `datetime` for the start and end of the sessions.  This JSON object is publicly available hosted on Github's infrastructure, and you can make free use of it to integrate in to Home Assistant.  You can read more about how that works here: <add link>
The script has correctly dealt with the changing formats used by Octopus and 

Everyone has different requirements from their Home Assistant automations, but I thought it would be worth writing up how I do it to demonstrate how straight forward it can be, and how flexible it is.

