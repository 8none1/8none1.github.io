---
id: 6
title: 'Absolute beginners guide to Google Maps Javascript'
date: '2009-05-08T14:05:04+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=6'
permalink: /2009/05/08/absolute-beginners-guide-to-google-maps-javascript/
authorsure_include_css:
    - ''
authorsure_hide_author_box:
    - ''
categories:
    - 'Google Maps'
    - Javascript
---

# <span style="color: #339966;">STOP! This page is quite out of date! [Click here to read an up-to-date version</span>](http://www.whizzy.org/2013/12/absolute-beginners-guide-to-google-maps-javascript-v3/ "Absolute beginners guide to Google Maps JavaScript v3").</span>

So – you’ve got an idea for site that makes use of the Google Maps API but you lack any knowledge of Javascript whatsoever – where do you start? Up until about 6 months ago I didn’t know, so I went about learning how and now I’m going to write it up in to a tutorial so you can find out too. This guide is aimed at beginners of Javascript and DOM scripting, some knowledge of HTML is assumed. We will assume that you are running the HTML from your local computer to start with, before uploading it to a server. I would recommend that if you use Firefox, that you install the Firebug addon: <https://addons.mozilla.org/en-US/firefox/addon/1843> Chromium comes with the same functionality built in, just hit f12.

### Part 1. Getting that draggable map on to your web page.

The Google maps API really does a lot to make it easy to get the map to appear on your page, but you need to lay a few of the foundations first. The steps involved in getting the map on to your page are:

1. Getting a Google Maps API key
2. Creating a correctly formatted HTML page
3. Loading the necessary JavaScript libraries
4. Writing your own JavaScript code to get the map to actually appear
5. A bit more code to make the map into something useful

All of the above should be achievable in about 60 minutes.

#### Part 1.1 Getting a Google Maps API key

Nice and easy. Go to:

[https://developers.google.com/maps/documentation/javascript/tutorial#api\_key](https://developers.google.com/maps/documentation/javascript/tutorial#api_key) (opens in a new window)

and follow the instructions.

While you don’t actually **need** an API key, it will help you to track usage and if your map page hits the big time you’ll already be set up.

Got your API key? Good, let’s carry on.

#### Part 1.2 Writing the basic HTML

The best thing about standard HTML is that there is so much variety. In order to try and guarantee a consistent layout across browsers you need to make sure that your page is rendered in ‘Standards Compliant’ mode as opposed to ‘Quirks’ mode. To do this you need to specify a DOCTYPE at the top of your HTML file. Google recommends that you use this XHTML DOCTYPE:

```
urn:schemas-microsoft-com:vml
```

so your HTML file should have that line right at the top. Hand in hand with this goes an ‘xmlns’ attribute on the &lt;html&gt; element in the page, like this:

```
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:v="urn:schemas-microsoft-com:vml">
```

We are using this particular line because we might want to make use of the PolyLine features of Google Maps in the future. More on this in a later guide, for now just believe me.

It’s also important to specify the correct character encoding. Google Maps by default uses UTF-8 and you should do the same unless you have a reason not to. This is the example given by Google:

```
<meta http-equiv="content-type" content="text/html; charset=utf-8"/>
```

but this goes *inside* the HTML head area.

If we put all these together in to a very basic HTML file it would look something like this:

```
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:v="urn:schemas-microsoft-com:vml">
  <head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8"/>
    <title>My first Google Map</title>
  </head>
  <body>
    <div id="map" style="width: 800px; height: 800px; margin: auto; background-color: red">
      This is where the map will soon appear.
    </div>
  </body>
</html>
```

Copy and paste this in to a text editor and then save that file as something like ‘maps.html’ and open it in your browser. You won’t be amazed to see an 800 x 800 pixel red box in the middle of the screen. But the really important thing is that this is valid XHTML meaning the browser will render it in ‘standards compliant’ mode.

So far so good. You’ve got a Google Maps API key and now you’ve got an area on the page in which the maps can be displayed, now you just need get that map on to the page.

#### **Part 1.3 Loading the Javascript libraries**

Google to the rescue. They host a set of [AJAX](http://en.wikipedia.org/wiki/Ajax_(programming)) APIs that we can use to do some really clever tricks, but first we need to use the loader to get access to them. This is simply a case of loading the Google AJAX API Loader with this line inside the &lt;head&gt; section of the HTML page:

```
  ...
    <script type="text/javascript" src="http://www.google.com/jsapi?key=XXXXXXXXXXXX"></script>
  ...
</head>
```

where ‘XXXXXXXXX’ should be your API key.  
This loads the Google loader, which we can then use to load the Google Maps API (and in the future any other Google hosted APIs such as jQuery which we’ll get to another day). Again, this code is located in the &lt;head&gt; section of the HTML, below the line that loads the Google loader. So, now our &lt;head&gt; section looks like this:

```
  ...
    <script type="text/javascript" src="http://www.google.com/jsapi?key=XXXXXXXXXXXX"></script>
    <script type="text/javascript">
      google.load("maps","2", {"other_params":"sensor=false"});
    </script>
  ...
</head>
```

What’s just happened here, is that you’ve just written a line of Javascript! It’s not a very long or complicated line but it opens up a world of Javascript wonder to us. There are six things we should mention about that one little line:

1. Javascript inside an HTML file must be surrounded by `<script>` tags. To make sure it’s a valid tag the opening one must say what sort of thing is inside. In our case ‘text/javascript’
2. The ‘google.’ bit becomes available after we loaded jsapi from www.google.com at the start of this section. This is referred to as the google namespace. 
3. The google namespace has many methods. One of these methods is ‘load’ which is separated from the ‘google’ bit with a ‘.’ and accepts a few arguments.
4. The arguments are passed inside the brackets. In our case we are passing the arguments ‘maps’ and ‘2’. ‘maps’ is the API we want to load, and ‘2’ is the version. Version 2 is a bit of a generic version number and actual version number that represents ‘2’ changes frequently. For example, at the moment 2 represents version 2.97
5. We also have to tell Google that we’re not using a sensor (e.g. GPS) to determine user location. I don’t know why this should matter, but Google say you must tell them, so tell them we will. This is a sort of dictionary where we link keywords with values. In this case the keyword is ‘other\_params’ and the value is ‘sensor=false’.
6. The line ends with a semicolon. You may see Javascript written where semicolons don’t appear at the end of a line. Some browsers are smart enough to realise that a new line represents the end of a line of code, others are not. So for now just always stick a semicolon on the end of a line, except where you are closing a curly bracket and then immediately closing a normal bracket.

Add these two script elements to your HTML file in the &lt;head&gt; section, below the meta tag and above the title, and reload the page. If you’ve done it correctly you won’t notice any difference and Firebug should show no errors.

#### Part 1.4 Making the map appear

Now we’re going to write the code to make the basic map appear on your page, in 5 extra lines.

```
...
<script type="text/javascript">
  google.load("maps","2", {"other_params":"sensor=false"});
  function init() {
    map = new google.maps.Map2(document.getElementById("map"));
    map.setCenter(new google.maps.LatLng(54.124634, -3.237029),17);
  };
  google.setOnLoadCallback(init);
</script>
...
```

These extra lines of code go between the same &lt;script&gt; tags as the line we used to load the the Google Maps API and must come after the google.load line otherwise the google.maps namespace won’t exist. First we create a new function called init. As you’d expect – the function won’t do anything until it’s called, we’re just creating it so we can use it in a minute. The function doesn’t take any arguments, hence the empty set of brackets. The actual code inside the function lives between two curly brackets { } and of course each line ends with a semicolon.

First we declare a variable called map. Notice that we don’t say ‘**var** map = new google.maps.Map2….’. If we did use var then our ‘map’ would only be accessible from within the function that declared it (init in our case). Since we are going to access the map from many other functions we’ll create ‘map’ as a global variable by not using var.

The variable ‘map’ becomes a new instance of the class ‘Map2’ from within the google.maps namespace. The Map2 class takes a ‘constructor’ which specifies the node within the document that will be used to display the map. In our case, as is most common, that node is a div element. We pass the getElementById function the Id of our div and it returns a reference that we can then pass to Map2 as the required constructor.

Or, put differently, ‘map’ becomes a new copy of a thing called Map2 which Google have provided for us. The new copy needs to know where to put the map, so we tell it which div we want to use.

Now our variable ‘map’ has been constructed it is an object of class Map2 and as such it posses certain predefined ‘methods’ and ‘events’. A methods is a way of doing things, and events happen. These are already in place and all we have to do is either tell them what to do, or listen to what they have to say.

The first method we make use of is setCenter (note the spelling Euro people). Unsurprisingly the setCenter method sets the centre point for the map. It takes as an argument a special Google LatLng object. How do I know this you ask? Everything you need to know about the Google Maps API can be found here (centred on the setCenter method): <http://code.google.com/apis/maps/documentation/reference.html#GMap2.setCenter>

(You need to be aware that the API refers to the original G namespace, not the google.maps namespace. So in the API wherever you see a capital G in front of something, you can probably assume you should drop the G and replace it with ‘google.maps.’ You can argue that we should stick to using the G namespace since that’s what the docs are written in, but the conversion is really easy, and I expect that all new API docs will use the full google namespace. **UPDATE:** Google have announced that indeed everything will be moving to the google namespace in the future. <http://googlegeodevelopers.blogspot.com/2009/05/announcing-google-maps-api-v3.html>)

From the API reference you can see the method also accepts an optional (note the ‘?’ in the API reference) zoom and type. Since it is optional will ignore the type for now, and only pass a LatLng object and a zoom level. It’s easy to create our LatLng object, we just instanciate (create) a new google.maps.LatLng class with the latitude and longitude passed to it as numbers. You can see we do all of this inline, we could equally have declared a new variable, this time with the var prefix, like this:

```
var myLatLng = new google.maps.LatLng(54.124634, -3.237029);
```

and then passed that to setCenter instead:

```
map.setCenter(myLatLng,17);
```

As you’ve probably already worked out the ’17’ is the optional zoom value.

Notice we close the function with another close-curly-brackets and a semicolon on a new line with the same indention as the line where we started the function, and the contents of the function indented from there.

The last of our Javascript lines causes our init function to get called and make the map appear. When you are working with elements in the page, for example a div with and Id of ‘map’, using Javascript you are interacting with the DOM, Document Object Model. This is the browsers internal table of what makes up the web page you are looking at and it gets created as the HTML source loads. If you’re at the end of a particularly slow internet connection, say a mobile phone, and the page you are loading is quite large it could take a minute or two before all the HTML is loaded. Before all the HTML is loaded the browser can’t finish building the DOM. If you tried to reference an element in on the page before it had loaded you would get an error something along the lines of “the element your are referring is undefined”. To combat this the browser can tell you when the DOM is ready for you to use and to make things even easier the Google loader provides a nice easy way for you to call your functions when the DOM is ready.

From the google namespace we have a method called setOnLoadCallback which takes a function as an argument. So we pass it our function called init, and sit back and relax as the browser loads the source, gets the DOM ready, signals that the DOM is ready, the onLoadCallback method gets triggered and in turn calls our init() function which replaces the div called map with a draggable Google Map. So add the above few lines in to your HTML and reload your browser. If you’ve done it right you should see a map!

In 21 lines of text we’ve created a Google map in the middle of our page. It’s pretty basic, in that we can only drag it about and double click to zoom in or double right click to zoom out, but consider how little we had to do to make it work.

#### Part 1.5 Doing a bit more with the map</span>

We’ve got a map, but it’s missing a few key parts like the zoom slider and the buttons to switch between the different map types and a marker indicating a point on the map.

Remember how we could have created a variable to hold our google.maps.LatLng object, but that we didn’t need to? Well, now we’re going to need to reference that LatLng in a couple of places, so it makes sense to keep it in a variable, just to save a bit of typing. To do this we’re going to rewrite the init() function. The new function looks like this:

```
function init() {
  map = new google.maps.Map2(document.getElementById("map"));
  var myLatLng = new google.maps.LatLng(54.124634, -3.237029);
  map.setCenter(myLatLng,17);
  var myMarker = new google.maps.Marker(myLatLng, {"draggable":"true","title":"A place on earth"});
  map.addControl(new google.maps.LargeMapControl());
  map.addControl(new google.maps.MapTypeControl());
  map.setMapType(G_SATELLITE_MAP);
  map.addOverlay(myMarker);
};
```

The way we create the map object stays the same, we create a new instance of class google.maps.Map2 and tell it which div we want to use to display the map. The next line declares our new myLatLng variable which will be local to the init function because we use ‘var myLatLng….’ and then we set the centre of the map using our myLatLng variable and set the zoom level to 17.

Next we declare another new variable called myMarker. This is a new instance of class google.maps.Marker, and to instantiate it we need to pass it a location in google.maps.LatLng format as per the map centre and some additional options. Since we have used the same LatLng object for the Marker as we did for the setCenter the Marker will appear in the middle of the map. Again, all of this information is found in the API reference:

<http://code.google.com/apis/maps/documentation/reference.html#GMarker>

where we can see that the options are listed under GMarkerOptions. These options are passed to Marker in the dictionary style format we have seen before, that is – enclosed in curly brackets we have pairs of keywords and values. The keywords and values are separated by a colon, and each pair is separated from the next with a comma. In our example we are only passing two options:

```
{"draggable":"true","title":"A place on earth"}
```

If we wanted to add another option, say “bouncy” we’d just add it to the end:

```
{"draggable":True","title":"A place on earth","bouncy":"true"}
```

If you read the API reference you’ll see that we don’t need to pass “bouncy”:”true” though, as any marker that is “draggable” is also “bouncy” by default, but if you wanted to turn off “bouncy” you could say that “bouncy”:”false”.

We’ve now created our Marker object, but we haven’t added it to the map yet. First of all though we add the zoom slider with directional buttons and the buttons to allow us to switch between the various map types (see the API reference under ‘GMapType’ – ‘Constants’ for the various types available). To add these controls to our map we use the addControl() method of a google.maps.Map2 class, in our case the variable ‘map’. From the API we see that the addControl method has a constructor which requires a GControl object and an optional position. If we then look up the GControl object in the API reference we see that it too requires constructing (that is, it is not a constant – it must be summoned from somewhere before we use it). The API shows that the GControl class constructors don’t require any further information in order to be created – as shown by the empty set of brackets, so to instantiate (summon) a GControl object we just create a ‘new’ one:

```
new google.maps.LargeMapControl()
```

will do that for us. Remember if something in the API starts with an uppercase G we replace that G for ‘google.maps.’ (note the trailing dot). Since the addControl method requires this new instance of a GControl class as a constructor we just pass it straight in:

```
map.addControl(new google.maps.LargeControl());
```

We could equally say:

```
var myLargeControl = new google.maps.LargeControl();
map.addControl(myLargeControl);
```

but MyLargeControl just sounds too rude.

We repeat this process to add the controls to switch between map types:

```
map.addControl(new google.maps.mapTypeControl());
```

Next we change the default map type to the satellite view instead of the default map which only shows roads. We use the setMapType method of a google.maps.Map2 class (our variable map). From the API we see that setMapType requires a GMapType. We don’t know what a GMapType is, so we click on the link in the API which takes us here:

<http://code.google.com/apis/maps/documentation/reference.html#GMapType>

and have a quick read. Straight away it says that Google have provided some predefined map types, so scroll down to “Constants” and you can see a list of these predefined map types. We’re going to use the G\_SATELLITE\_MAP type and because it’s a constant we can just refer directly to it:

```
map.setMapType(G_SATELLITE_MAP);
```

Then last of all we add the marker we created earlier to the map:

```
map.addOverlay(myMarker);
```

addOverlay() is a method of google.maps.Map2 and accepts the marker as an overlay to add. Each marker on the map is an overlay.

And there you have it. A map on your web page in only 27 lines including the HTML itself. The final HTML file looks like this:

```

<html xmlns="http://www.w3.org/1999/xhtml" xmlns:v="urn:schemas-microsoft-com:vml">
<head>
  <meta http-equiv="content-type" content="text/html; charset=utf-8"/>
  <script type="text/javascript" src="http://www.google.com/jsapi?key=xxx"></script>
  <script type="text/javascript">
    google.load("maps","2", {"other_params":"sensor=false"});
    function init() {
      map = new google.maps.Map2(document.getElementById("map"));
      var myLatLng = new google.maps.LatLng(54.124634, -3.237029);
      var myMarker = new google.maps.Marker(myLatLng, {"draggable":"true","title":"A place on earth"});
      map.setCenter(myLatLng,17);
      map.addControl(new google.maps.LargeMapControl());
      map.addControl(new google.maps.MapTypeControl());
      map.setMapType(G_SATELLITE_MAP);
      map.addOverlay(myMarker);
    };
    google.setOnLoadCallback(init);
  </script>
  <title>My first Google Map</title>
</head>
<body onunload="google.maps.Unload()">
  <div id="map" style="width: 800px; height: 800px; margin: auto; background-color: red">
    This is where the map will soon appear.
  </div>
</body>
</html>
```

Note – we have slightly modified the &lt;body&gt; to include an additional method from the google.maps namespace. This function ensures that when you close the browser window containing your map everything is tidied up. This prevents memory leaks in browsers such as IE6. You should always include this in your code when you are using Google Maps.

Challenge 1. As a bit of fun, why not try to add one of these to the map:

<http://code.google.com/apis/maps/documentation/reference.html#GControlImpl.GOverviewMapControl>

Challenge 2. See if you can trim the code by one line and increase the available map types at the same time:

<http://code.google.com/apis/maps/documentation/reference.html#GMap2.setUIToDefault>

## Coming in Part 2…

In part two we’ll cover more things you can do with markers such as:

- Create a function to add markers to the map.
- Giving markers pop-up info windows containg text
- Reading information back out of the marker

## Useful References

[Google Maps API Concepts  
http://code.google.com/apis/maps/documentation/](http://code.google.com/apis/maps/documentation/ "Google Maps API Concepts")

[Google Maps API Reference  
http://code.google.com/apis/maps/documentation/reference.html](http://code.google.com/apis/maps/documentation/reference.html "API Reference")

[Google Maps Examples  
http://code.google.com/apis/maps/documentation/examples/index.html](http://code.google.com/apis/maps/documentation/examples/index.html "Great examples of Google Maps")
