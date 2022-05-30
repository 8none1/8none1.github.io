---
id: 457
title: 'Absolute beginners guide to Google Maps JavaScript v3'
date: '2013-12-17T14:58:55+00:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=457'
permalink: /2013/12/17/absolute-beginners-guide-to-google-maps-javascript-v3/
authorsure_include_css:
    - ''
authorsure_hide_author_box:
    - ''
categories:
    - 'Google Maps'
    - Javascript
    - 'Making the world a better place'
---

Since I first published my [HowTo](http://www.whizzy.org/2009/05/absolute-beginners-guide-to-google-maps-javascript/ "Absolute beginners guide to Google Maps Javascript") and the [subsequent](http://www.whizzy.org/2009/05/absolute-begineers-guide-to-google-maps-javascript-part-2/ "Absolute beginners guide to Google Maps Javascript Part 2") follow up for novices to get a Google Map on a web page it’s been the most popular post on my site by quite a margin. Sadly it’s been resting on its laurels, and is now quite out of date and indeed broken. So spurred on by my recent server replacement and attempt to revitalise this blog I present the new, improved, and generally working… Beginners Guide To Google Maps JavaScript v3.

The premise is simple; you’ve got a requirement to put a map on your site. You don’t have the first clue how to do it. You follow these instructions and you get you map. Hopefully you’ll pick up enough to tweak the map to your requirements, if not – please ask in the comments and I’ll try and help you. I’m assuming that you’ve got a bit of HTML experience, and that you’ve got a JavaScript debugger available to you (either Firebug for Firefox or Developer Tools built in to Chrome).

I’ve based a lot of this on the official Google tutorial here: <https://developers.google.com/maps/documentation/javascript/tutorial>. (I’m pretty sure they borrowed that from me in the first place, so it’s only fair 😉 ).

# Part 1. Getting the map on the page

Get yourself an API key for Google Maps from here: [https://developers.google.com/maps/documentation/javascript/tutorial#api\_key](https://developers.google.com/maps/documentation/javascript/tutorial#api_key)

Let’s start from the end and work back to the beginning. Save this as an HTML document, change the API key, open it in your browser and you’ll have a map. I’ve saved you a bit of time by already centring it on Barrow-in-Furness bus depot.

\[js\]  
&lt;!DOCTYPE html&gt;  
&lt;html&gt;  
 &lt;head&gt;  
&lt;title&gt;My first Google Map&lt;/title&gt;  
 &lt;meta name="viewport" content="initial-scale=1.0, user-scalable=no" /&gt;  
 &lt;style type="text/css"&gt;  
 html { height: 100% }  
 body { height: 100%; margin: 0; padding: 0 }  
 #map-canvas { height: 100% }  
 &lt;/style&gt;  
 &lt;script type="text/javascript"  
 src="https://maps.googleapis.com/maps/api/js?key=XXXXXXXXXXXXXXXX&amp;sensor=false"&gt;  
 &lt;/script&gt;  
 &lt;script type="text/javascript"&gt;  
 function initialize() {  
 var myLatLng = new google.maps.LatLng(54.124634, -3.237029)  
 var mapOptions = {  
 center: myLatLng,  
 zoom: 17,  
 mapTypeId: google.maps.MapTypeId.SATELLITE  
 };

 var map = new google.maps.Map(document.getElementById("map-canvas"),  
 mapOptions);

 var myMarker = new google.maps.Marker({  
 position: myLatLng,  
 map: map,  
 title: "A place on Earth",  
 draggable: true,  
 })  
 }

 google.maps.event.addDomListener(window, ‘load’, initialize);  
 &lt;/script&gt;  
 &lt;/head&gt;  
 &lt;body&gt;  
 &lt;div id="map-canvas"/&gt;  
 &lt;/body&gt;  
&lt;/html&gt;  
\[/js\]

Copy and paste that in to a text editor, replace XXXXXXXXXXXXXXXX with your own API key, save it somewhere and the load it up in your browser. You should see a satellite style map with a marker in the middle. You’re done. Simple eh? Read on learn a bit about how it works, and what you can do to change the appearance.

## Part 1.1 Understanding the basic HTML

In order to try and guarantee a consistent layout across browsers you need to make sure that your page is rendered in ‘Standards Compliant’ mode as opposed to ‘Quirks’ mode. To do this you need to specify a DOCTYPE at the top of your HTML file. We’re using a very simple “html” DOCTYPE which tells the browser that we’re HTML5. In HTML4.x there were a plethora of variations – thankfully we don’t need to care about them anymore. HTML5 is the way to go, and so the only DOCTYPE we care about is “html”.

We set the title of the page, and then we set some initial viewport settings to help mobile browsers render the page correctly. This is widely regarded as a good thing, and you can learn more about it from here: <http://webdesign.tutsplus.com/tutorials/htmlcss-tutorials/quick-tip-dont-forget-the-viewport-meta-tag/>

Skipping over the style and scripts for a moment, we create the main body of the page with a single div element in it. We give it the id “map-canvas”, and then we close off the bottom of the body and mark the end of the HTML. You won’t be surprised to read that the div we’ve just created will be where the map will soon appear.

## Part 1.2 All about style

In standard HTML5 (remember the DOCTYPE from above) there are a few specifics you need to know about CSS. If an element specifies a size as a percentage, then that percentage is calculated from the parent objects size. Imagine you have a nested DIV, called DIV2. It lives inside DIV1. Where DIV1 has a fixed size of 500px by 500px, then DIV2 knows that 100% high equals 500px, but what if DIV1 didn’t have a size specified? In that case, in standards mode, DIV2 would decide that 100% high equals zero px – because it doesn’t know any better. This has caught people out a few times. In order to make sure that all our DIVs can inherit a size correctly we set the height of the entire HTML page and the BODY to be 100%. This is calculated by the browser when the page loads, and then can flow down to the elements within the page correctly.

Once we have the parent elements size set correctly (the page, and the body) we can style our map DIV to be 100% high safe in the knowledge that it has enough information to render and the correct size and not 0 px high.

## Part 1.3 The Meat Section

Now we’re going to look at the actual JavaScript and understand what it’s doing and in which order.

First of all, we have the scripts in the HEAD tag. The browser will load the head part of the HTML first and your scripts will be loaded before the page is fully rendered. Any functionality that you make available in your scripts should be available to the rest of the page when it comes to need it. This is generally the right way to do it.

The first SCRIPT tag takes care of loading the Google Maps code. We tell the browser that the content of the script is text/javascript and then where to find it. There are a couple of parameters we pass in to the script through the URL. The first one is “key” – this is your simple API key for access Google Maps (see above for details of where to get this key). The second parameter is “sensor”. This is required and must be “false” if you’re not using a GPS (or similar) to work out where you are. In our case, we’re just picking a point on and saying “centre the map here” – so we use false. If were using a GPS to centre the map on our current location, then this would be “true”.

This script gets loaded by the browser and now we can start to make use of the Google Maps JavaScript APIs in the rest of our page.

The second script is a bit more complex, but should be easy to understand:

We create a new function called “initialize”.  
Inside that function we create an object called myLatLng. We use the Google provided API google.maps.LatLng() to create an object which can be understood by the rest of the Google Maps API and pass in the co-ordinates of Barrow-in-Furness bus depot. We use “var” to limit that object’s “scope” to within the “initialize” function – that is to say, we won’t be able to get access to that particular myLatLng from other functions on the page.  
Next we create another thing called mapOptions. The format of this is as per the spec here: [https://developers.google.com/maps/documentation/javascript/reference?hl=en#MapOptions  ](https://developers.google.com/maps/documentation/javascript/reference?hl=en#MapOptions)

There are loads of options, most of which we don’t need to worry about, so we’re just setting a few key options: where the map is centered, how much it is zoomed in, and the type of map we see. The map types are provided by Google as a set of constants, which are identifiable by being all in upper case. In our case we’re using SATELLITE, but we could also use HYBRID, ROADMAP or TERRAIN.

Once we’ve set up the various mapOptions we create a new var called “map” which is an instance of a Map object as provided by the Google APIs. We pass in to it the id of the HTML object where we want the map to appear, as we created in section 1.2, and we pass in the options var which we just created. This is enough information for the Google APIs to set up the map as we want it and put it on the page.

The last thing we do in our example is add a marker. A marker is the indicator which you use to highlight a point on the map. Google provide a lot of icons and colours for us to use, but the default is the red tear-drop one, so we’ll stick with that for now.

To create a marker we create a new instance of google.Maps.Marker and set up some of the options as we did for the map itself. We tell it the position for the marker to appear. We use the “myLatLng” object we created earlier. You might notice that we are using the myLatLng object twice in our example. Once as the centre point for the map, and once for the position of the marker. You can probably deduce from this that the marker will appear in the centre of the map. We also tell the marker which map it should be added to. We only have one map on our page, but if we had many this is how you’d add a marker to the correct map. We give it a title, which is simply a string and we make it draggable by setting the draggable option to “true”. You can read more about the marker options here: <https://developers.google.com/maps/documentation/javascript/reference?hl=en#MarkerOptions>

That’s very nearly it for our first simple map. The last thing to do is use a DOM listener to trigger the above JavaScript when the page loads, and so make our map and marker appear.

google.maps.event.addDomListener is provided via the Google APIs and we pass in three pieces of information. window is the object provided by the browser. The ‘load’ event is actually is separate from the “onload” event you might have read about. The load event signifies that the page is fully rendered and any JavaScript which wants to manipulate the DOM can begin work, and that’s what we want to do. We want to swap the empty div with the id of “map” with the actual map. So once the page is indeed loaded the command will execute the “initialize” function and all the magic will happen.

And that’s it. We’re done. We’ve got a map on the page centred at our chosen location, and there is a little marker to show a specific part of the map.

If you compare this to the original [Beginners Guide](http://www.whizzy.org/2009/05/absolute-beginners-guide-to-google-maps-javascript/ "Absolute beginners guide to Google Maps Javascript") I think you’ll say that this new version of the API is even easier to use. I will try and find time of the next few months to jot down a few notes on doing more interesting things with the map but I hope that this will get you started.