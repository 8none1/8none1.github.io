# Automating Power Ups

[Octopus Energy](https://octopus.energy/) are a UK energy supplier who are more progressive than the rest of the UK providers put together.  They provide tariffs for gas and electricity which enable time-of-use based billing which is very useful for people with large battery storage and/or electric vehicles where you can load up when the energy is cheapest (e.g. over night) reducing your average per kWh cost quite considerably.  The have built a reliable, easy to use REST api which allows you to get your per-30-minute prices a day ahead as well as querying your usage etc.  This has led to a number of API abstraction libraries and tools being made available by the community.  My favourite is the [Home Assistant integration](https://github.com/BottlecapDave/HomeAssistant-OctopusEnergy) from BottlecapDave.

The Home Assistant integration provides you with costs, unit pricing and most usefully the ability to trigger your own custom integrations when prices drop below a certain cost for a configurable amount of time.  This allows me to set a 4 hour window each day to charge my battery whenever the electricity is cheapest.  The integration calls these "target rate sensors" and they have proved very reliable, allowing me to charge for a total of 4 hours over the eight cheapest 30 minute slots per night.

## Power Ups

Octopus also have a scheme they call [Power Ups](https://octopus.energy/power-ups/) whereby if you live in a region which is operated by UK Power Networks (i.e. the East & South East) then whenever there is an excess of electricity available, for example when it's very windy or very sunny, that electricity is given away for free[^1].

To be brutally honest, the mechanisms around Power Ups are clunky.  At the moment there isn't an API.  Instead you get an email from Octopus about a day before the power up is available, and you have to click a link to a form and use that form to opt in.  If you don't opt in, you don't get free electricity.  Generally speaking this hasn't been a problem though.  I'm sat in front of the computer all day every day, so clicking a link doesn't present much of a challenge.

That said, it would still be very useful to have the start and end times of the Power Up available to me in an easy to consume format such as a JSON object.  So let's do that...

[^1]: You actually get billed for it, but then a credit is applied to your account for the same amount.

## GMail and Google Apps Script

I've used Apps Script for a few projects where I want something small hosting on the web for free and I want to integrate with an existing Google product.  In this case, my email is hosted in GMail and that will be the data source for the Power Up.  I want to be able to find the email from Octopus in the soup of the rest of my email, write a regex to find the relevant data, and then publish that data to the web.

Finding the email is pretty easy.  The App Script API has a `GMailApp.search` [method](https://developers.google.com/apps-script/reference/gmail/gmail-app#searchquery) which takes the same search that you would run in the UI.  So something like:

```from: hello@octopus.energy subject: "Power-ups: Opt in"```

should do the job[^2].  

```
function getPowerUpEmails() {
  var threads = GmailApp.search('from: hello@octopus.energy subject:"Power-ups: Opt in"',0, 3);
  var messages = [];
  threads.forEach(function(thread) {
    messages.push(thread.getMessages()[0]);
  });
  return messages;
}
```

will return an array containing the first 3 matching email objects.


[^2]: Yes, this is very fragile.  But at least it was quick.