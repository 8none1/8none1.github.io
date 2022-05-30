---
id: 146
title: 'Absolute beginners guide to Google Maps Javascript Part 2'
date: '2009-05-11T13:44:04+01:00'
author: will
layout: post
guid: 'http://www.whizzy.org/?p=146'
permalink: /2009/05/11/absolute-begineers-guide-to-google-maps-javascript-part-2/
authorsure_include_css:
    - ''
authorsure_hide_author_box:
    - ''
categories:
    - 'Google Maps'
    - Javascript
---

# <span style="color: #339966;">Stop! This page is out of date. [<span style="color: #339966;">Read this one first</span>](http://www.whizzy.org/2013/12/absolute-beginners-guide-to-google-maps-javascript-v3/ "Absolute beginners guide to Google Maps JavaScript v3").</span>


# Part 2. Doing a bit more with the Google Maps API

This guide follows on from this one: [http://www.whizzy.org/2009/05/absolute-begineers-guide-to-google-maps-javascript/](http://www.whizzy.org/2009/05/absolute-begineers-guide-to-google-maps-javascript/ "http://www.whizzy.org/2009/05/absolute-begineers-guide-to-google-maps-javascript/")  
You’ll need to install Firefox and the Firebug add-on: [https://addons.mozilla.org/en-US/firefox/addon/1843](https://addons.mozilla.org/en-US/firefox/addon/1843 "https://addons.mozilla.org/en-US/firefox/addon/1843")

If you’ve run through the previous guide you should have a little 27 line piece of HTML and JavaScript which gets you a basic draggable map on to your page with a marker in the centre. It’s not much, but it was easy to do and most of all it didn’t take us very long. Now we’re going to make the process of adding more markers to the map a bit easier, give each marker an infoWindow to display some text and see how to read some information back out of the marker object and finally we’ll give our marker a different icon to the standard red one.

1. Create a function to add Markers
2. Giving markers an infoWindow
3. Reading information back out of the marker
4. Changing the marker icon

## Part 2.1 Creating a function to add Markers

Why do you need a function to add markers when you could just copy &amp; paste the line we used in part 1. Imagine you have 50 markers added to your map, they would all have different co-ordinates but they might also have some common features such as “type” which might be ‘Bus Stop’. What would happen if you wanted to make a change to that title for all your makers, like adding a prefix of ‘Marker:’? If you used the copy &amp; paste method to create the 50 markers you would have to go through each of the 50 lines and make the change, but if you used a function to create the markers and moved the common attributes to that function you would only need to make the change in one place – inside the function. By doing this we are using the DRY principal – Don’t Repeat Yourself. It will allow us to specify all common attributes only once and so therefore make changing those common attributes very quick. Our new function is called addMarker and when added to our code from part 1, the &lt;script&gt; section looks like this:

```
<pre class="codestyle"><script type="text/javascript">
  google.load("maps","2", {"other_params":"sensor=false"});
  function init() {
    map = new google.maps.Map2(document.getElementById("map"));
    var myLatLng = new google.maps.LatLng(54.124634, -3.237029);
    map.setCenter(myLatLng,17);
    map.addControl(new google.maps.LargeMapControl());
    map.addControl(new google.maps.MapTypeControl());
    map.setMapType(G_SATELLITE_MAP);
    newMarker1 = addMarker(54.124634, -3.237029,'Barrow-in-Furness Bus Depot');
    newMarker2 = addMarker(54.124500, -3.237129,'Monster defence system');
  };
  function addMarker(lat,lng,marker_title) {
    var myLatLng = new google.maps.LatLng(lat,lng);
    marker_title = 'Marker:  '+marker_title;
    var myMarkerOptions = {"draggable":true, "title":marker_title};
    var myMarker = new google.maps.Marker(myLatLng, myMarkerOptions);
    map.addOverlay(myMarker);
    return myMarker;
  };
  google.setOnLoadCallback(init);
</script>
```

You can see the new addMarker function inserted between the closing curly bracket of init() and the line google.setOnLoadCallback(init);. The function accepts three things as shown by the comma separated values inside the brackets – lat,lng and title. These are variables which become accessible inside the function and will be set to the values passed in when the function is called, in the same order as they are passed in. For example, if I called the addMarker function like this:

```
<pre class="codestyle">addMarker('blue',42,'soup');
```

then lat would become a string ‘blue’, lng would become the number 42 and title would be a string ‘soup’. Obviously those values aren’t going to be valid for our real function as ‘blue’ is not a measurement of latitude – but you get the idea. We actually call the function from inside the init() function, like this:

```
<pre class="codestyle">newMarker1 = addMarker(54.124634, -3.237029,'Barrow-in-Furness Bus Depot');
newMarker2 = addMarker(54.124500, -3.237129,'Monster defence system');
```

You can see we actually pass in a real latitude, longitude and title string.  
Back inside the new function, the first thing we do is construct a new instance of a google.maps.LatLng class by passing it the lat and lng variables that were used when the function was called. We assign this to a local variable called myLatLng. Notice we use var myLatLng – this makes myLatLng local to our function, it’s not a global variable. Next we prepend ‘Marker: ‘ to the marker\_title variable and then we declare another function-local variable (note the var) which is our dictionary of keywords and values mated together with a colon and separated from each other with a comma. This also demonstrates why using a function is a better way of doing things. On the whole marker options are likely to the same for every marker you have on your map, so rather than specifying the same set of options for each marker we only do it once, inside the function. If we want to change the options we just have to change one line rather than one line per marker. If you wanted to add more options you would just add them to the end of the dictionary. For example, let’s say you wanted to make your markers not bounce up and down when you let go after dragging. From the API reference we can see that the option is “bouncy” and to turn it off we set it to false, so we just add that to the end:

```
<pre class="codestyle">var myMarkerOptions = {"draggable":"true", "title":marker_title, "bouncy":false};
```

You’ll notice that we don’t need to make “bouncy” true in order to get bouncy markers, from the API reference we can see that draggable makers are bouncy by default.

The next line creates a new instance of a google.maps.Marker class called myMarker and we use the variables we have defined above as constructors – myLatLng, which is an instance of a google.maps.LatLng and myMarkerOptions which is a dictionary of keywords and values. myMarker is now a fully formed marker that can be added to the map with map.addOverlay(myMarker) where map is our instance of google.maps.Map2 – the main Google Maps class.  
The last bit of our function is where things get a bit conceptual:

```
<pre class="codestyle">return myMarker;
```

When we instantiate the new marker we store a reference to it in the variable myMarker. myMarker is not actually the object, but rather a set of directions telling us where that marker is in memory. We can pass that reference around to tell functions which marker we are talking about when we perform actions on it, such as adding it to the map with addOverlay. We call addOverlay and tell it where to find the marker we have just created. A problem arises when we call the addMarker function for a second time, as the variable myMarker stops being directions to the first marker and starts being directions to the new marker. We have lost our directions to the first marker. It still exists in memory, and the map knows where it is, but we don’t. This is why we return the reference, or directions, at the end of our function – so we can keep a record of it outside of the addMarker function. When we call the function first time around:

```
<pre class="codestyle">newMarker1 = addMarker(54.124634, -3.237029,'Barrow-in-Furness Bus Depot');
```

the ‘return myMarker;’ passes the reference to the marker back out of the function and the ‘newMarker1 =’ accepts that returned reference and becomes a reference to the marker itself. Then, then next time we call the function:

```
<pre class="codestyle">newMarker2 = addMarker(54.124500, -3.237129,'Monster defence system');
```

newMarker2 becomes a reference to the second marker.

So there you have it. A simple little function that makes it a bit easier to add new markers. If you want a nice way of storing all these references and then being able to do things like loop through them – look up JavaScript arrays – but we’ll cover this in a later tutorial. Now we’ve got our function we can start adding more features to it.

## Part 2.2 Adding Info Windows to your markers.

An Info Window is a pop-up area inside that map that you can use to provide more information about markers. It usually appears when you click on a marker. There are many different types of infoWindow available with the Google Maps API, but for now we will be using an HTML infoWindow because it’s very easy to set up. If we are going to display an infoWindow we are going to need some information to display in it, so the first thing is to extend our function to accept a fourth incoming variable which can be used to pass in the contents of the infoWindow.

```
<pre class="codestyle">function addMarker(lat,lng,title,descr) {
```

This gives us a new variable inside the function called ‘descr’ which we will use as the contents of the infoWindow. As we are using an HTML infoWindow we could pass an HTML string in to our function when we call it, something like this:

```
<pre class="codestyle">newMarker1 = addMarker(54.124634, -3.237029,'Barrow-in-Furness Bus Depot','<b>Barrow-in-Furness Bus Depot</b><br />Under threat from monsters');
```

… or we could just pass in some plain text and HTMLifiy it inside the function, which is a much better idea as we can affect a global change much more easily inside the function. So we will just use a plain string:

```
<pre class="codestyle">newMarker1 = addMarker(54.124634, -3.237029,'Barrow-in-Furness Bus Depot', 'under threat from monsters');
```

which means we need to add the HTML inside the function. We will do this by associating a new variable to the marker object:

```
<pre class="codestyle">var myMarker = new google.maps.Marker(myLatLng, myMarkerOptions);
myMarker.infoHtml = '<b>'+title+'</b><br />'+descr
```

the ‘var myMarker…’ line is the same as it was above, I’ve included it here to show you where the new line appears, that is underneath the line where we instantiate myMarker. So, after we have created our myMarker we associate a new variable with it, called infoHTML. This is a very powerful function of JavaScript and we should explain exactly what’s happened here. We have created a new attribute or property within the myMarker object and assigned it a value. This property, or association, now lives with the myMarker object. We can add lots of extra information to myMarker very easily and use our reference to get access to them in this way. So anywhere that this marker goes, infoHtml will go with it. You can see that our HTML is made of the title in bold, a newline and the descr tagged on the end.

Now we have the HTML ready for the infoWindow we need to trigger the opening of that window. As you would expect this is triggered when you click on a marker. The Google Maps API makes it easy for us to hook in to this “click” event and to trigger our own code:

```
<pre class="codestyle">google.maps.Event.addListener(myMarker, "click", function() {
myMarker.openInfoWindowHtml(myMarker.infoHtml, {maxWidth: 300});
});
```

The Map2 class has a number of events that we can listen out for. We register our interest in a particular event by using the addListener method of Event. We are saying that when a “click” event is generated from source myMarker (where myMarker is a reference to the marker we are adding during this call of the function) then execute this function, and then the function to be run. That function, which is included inline, simply calls the openInfoWindowHtml method of myMarker with the infoHtml string we associated with the marker, and then the optional maxWidth keyword.

And that’s all there is to it. Now you have a function to create markers with infoWindows. The whole thing looks like this:

```
<pre class="codestyle">
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:v="urn:schemas-microsoft-com:vml">
  <head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8"/>
    <script type="text/javascript" src="http://www.google.com/jsapi?key=xxx"></script>
    <script type="text/javascript">
    google.load("maps","2", {"other_params":"sensor=false"});
      function init() {
        map = new google.maps.Map2(document.getElementById("map"));
        var myLatLng = new google.maps.LatLng(54.124634, -3.237029);
        map.setCenter(myLatLng,17);
        map.addControl(new google.maps.LargeMapControl());
        map.addControl(new google.maps.MapTypeControl());
        map.setMapType(G_SATELLITE_MAP);
        newMarker1 = addMarker(54.124634, -3.237029,'Barrow-in-Furness Bus Depot', 'Very tasty to monsters');
        newMarker2 = addMarker(54.124500, -3.237129,'Monster defence system', 'Chewits to deflect hungry monsters');
        };
      function addMarker(lat,lng,title,descr) {
        var myLatLng = new google.maps.LatLng(lat,lng);
        title = 'Marker:  '+title;
        var myMarkerOptions = {"draggable":true, "title":title, "bouncy":false};
        var myMarker = new google.maps.Marker(myLatLng, myMarkerOptions);
        myMarker.infoHtml = '<b>'+title+'</b><br />'+descr
        map.addOverlay(myMarker);
        google.maps.Event.addListener(myMarker, "click", function() {
          myMarker.openInfoWindowHtml(myMarker.infoHtml, {maxWidth: 300});
          });
        return myMarker;
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

## Part 2.3 Reading information back out of the marker.

Since our markers are draggable it would be useful to know where they have been moved to. We need somewhere to display that information and a means of reading the co-ordinates from the marker. Of course, this is made easy with the Google Maps API! First of all we’ll add a couple of text input areas to the HTML to use as somewhere to display the co-ordinates after a marker has been moved:

```
<pre class="codestyle"><div id="output">
  <label for="lat">Latitude:</label><input id="lat" type="text" value="Unknown" >
  <label for="lng">Longitude:</label><input id="lng" type="text" value="Unknown" >
</div>
```

Using text input boxes makes it easy to copy &amp; paste the results.  
Then we need a new function to read the information out of the marker. As before, the API provides a nice easy way for us to hook in, an event is generated when a marker has been dragged and then stops being dragged, called dragend (http://code.google.com/apis/maps/documentation/reference.html#GMarker.dragend). We add a listener to this event:

```
<pre class="codestyle">google.maps.Event.addListener(myMarker, "dragend", function() {
  var lat_box = document.getElementById("lat");
  var lng_box = document.getElementById("lng");
  var coords = this.getLatLng();
  lat_box.value = coords.lat();
  lng_box.value = coords.lng();
});
```

The structure of this section of our function is very similar to the infoWindow bit. We are adding a listener to the marker object that gets triggered when the dragend event is fired. This listener becomes part of the marker object referred to by myMarker. It works like this: the dragend event is fired when you release the mouse button having moved the marker, which in turns executes in inline function. We use “document.getElementById” and pass in the ID of the element to get a reference to elements in the DOM where we are going to write our output. Then we read the co-ordinates from the marker object and put them in the variable “coords”. To do this we refer to “this”. “this” is a special keyword in JavaScript, it refers to the owner of the function. So in our function, which is inline with the addListener method, the owner is set the marker itself, and the marker has a method getLatLng which returns a google.maps.LatLng, which in turn has a lat method and lng method.  
We combine all these things to set the contents of the two text boxes to be the latitude and longitude of the marker that was dragged.

## Part 2.4 Using your own icon

By default the Google Maps API provides you with nice red marker icon, which is fine for most maps. On my Potton Online website I needed to be able to show lots of different types of marker on one map (see [http://www.pottononline.net/map/](http://www.pottononline.net/map/ "http://www.pottononline.net/map/")). The most user-friendly way of doing this was to make each type of marker have it’s own icon. You won’t be surprised to hear that one of the options available when creating a new instance of a marker is “icon”. This “icon” option needs to be a google.maps.Icon class, which has a number of properties that define the icon to be used. Unfortunetly it’s not just a case of saying “Use this image to be the icon”, but it’s not a lot more complicated. We will create a function to create these google.maps.Icon instances and we will modify the addMarker function to accept these Icons so that the google.maps.Marker construction can use it.  
The new function to spit out google.maps.Icon instances looks like this:

```
<pre class="codestyle">function createIcon(iconUrl, shadowUrl) {
  var myIcon = new google.maps.Icon();
  myIcon.image = iconUrl;
  myIcon.shadow = shadowUrl;
  myIcon.size = new google.maps.Size(32,32);
  myIcon.shadowsize = new google.maps.Size(59,32);
  myIcon.iconAnchor = new google.maps.Point(16,32);
  myIcon.infoWindowAnchor = new google.maps.Point(16,0);
  return myIcon;
};
```

I hope that you’ll be able to look at this function and have a rough idea how it works. But let’s run through it anyway. The function accepts two inputs; iconUrl and shadowUrl. The icons on a map can have a shadow to give the impression that are standing up vertically from the map. This shadow is just an image with some clever stretching performed on it. The first thing we do is create a new empty instance of google.maps.Icon. We see from the API reference there are no required constructors, so we won’t use any. A google.maps.Icon has a number of properties, some of which are required, some of which aren’t. It isn’t immediately apparent to me which is which, so through trial and error I came up with this list. For now, just belive me. The first property is image. Fairly obviously this is required, and it’s a URL to a location where the image can be found. The same goes for shadow. size and shadowsize describe the size in pixels of the image and its associated shadow. These are of the type google.maps.Size which is just an x,y pair of numbers. We’re including them inline to save on typing. The iconAnchor and infoWindowAnchor are similar to the size properties. These describe a point on the icon where the it is anchored to the window and the infoWindow. These are also a special type, this time called Point. The last line returns the bundle of properties called myIcon back to the place where the function was called from. To call this function we use a line like this:

```
<pre class="codestyle">var icon = createIcon('http://maps.google.com/mapfiles/ms/micons/bus.png', 'http://maps.google.com/mapfiles/ms/micons/bus.shadow.png');
```

So “icon” now becomes an instance of google.maps.Icon and can be used when we instantiate our marker. This is done by passing the variable “icon” in to the addMarker function. When we call addMarker now, the line would look like this:

```
<pre class="codestyle">newMarker1 = addMarker(54.124634, -3.237029,'Barrow-in-Furness Bus Depot', 'Very tasty to monsters', icon);
```

You can see we’ve just tagged “icon” on the end of the data we are passing in to addMarker. Of course, this means that we need to modify the addMarker function to accept this extra information. So the whole addMarker function now looks like this:

```
<pre class="codestyle">function addMarker(lat,lng,title,descr,icon) {
  var myLatLng = new google.maps.LatLng(lat,lng);
  title = 'Marker:  '+title;
  var myMarkerOptions = {"draggable":true, "title":title, "bouncy":false, "icon":icon};
  var myMarker = new google.maps.Marker(myLatLng, myMarkerOptions);
  myMarker.infoHtml = '<b>'+title+'</b><br />'+descr
  map.addOverlay(myMarker);
  google.maps.Event.addListener(myMarker, "click", function() {
    myMarker.openInfoWindowHtml(myMarker.infoHtml, {maxWidth: 300});
    });
  google.maps.Event.addListener(myMarker, "dragend", function() {
    var lat_box = document.getElementById("lat");
    var lng_box = document.getElementById("lng");
    var coords = this.getLatLng();
    lat_box.value = coords.lat();
    lng_box.value = coords.lng();
    });
return myMarker;
};
```

The changes are on the first line, where we add ‘icon’ to the list of function inputs and we add another keyword to the myMarkerOptions line “icon” and the value icon which is the google.maps.Icon class we passed in when we called the function. And that’s it! Now your marker has an icon, which has it’s source as a URL to an image on the internet.

If we combine all these steps from Part 1 and Part 2 our entire HTML file now looks like this:

```
<pre class="codestyle">
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:v="urn:schemas-microsoft-com:vml">
  <head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8"/>
    <script type="text/javascript" src="http://www.google.com/jsapi?key=xxx"></script>
    <script type="text/javascript">
      google.load("maps","2", {"other_params":"sensor=false"});
      function init() {
        map = new google.maps.Map2(document.getElementById("map"));
        var myLatLng = new google.maps.LatLng(54.124634, -3.237029);
        map.setCenter(myLatLng,17);
        map.addControl(new google.maps.LargeMapControl());
        map.addControl(new google.maps.MapTypeControl());
        map.setMapType(G_SATELLITE_MAP);
        var icon = createIcon('http://maps.google.com/mapfiles/ms/micons/bus.png', 'http://maps.google.com/mapfiles/ms/micons/bus.shadow.png');
        newMarker1 = addMarker(54.124634, -3.237029,'Barrow-in-Furness Bus Depot', 'Very tasty to monsters', icon);
        var icon = createIcon('http://maps.google.com/mapfiles/ms/micons/snack_bar.png','http://maps.google.com/mapfiles/ms/micons/snack_bar.shadow.png');
        newMarker2 = addMarker(54.124500, -3.237129,'Monster defence system', 'Chewits to deflect hungry monsters', icon);
        };
      function addMarker(lat,lng,title,descr,icon) {
        var myLatLng = new google.maps.LatLng(lat,lng);
        title = 'Marker:  '+title;
        var myMarkerOptions = {"draggable":true, "title":title, "bouncy":false, "icon":icon};
        var myMarker = new google.maps.Marker(myLatLng, myMarkerOptions);
        myMarker.infoHtml = '<b>'+title+'</b><br />'+descr
        map.addOverlay(myMarker);
        google.maps.Event.addListener(myMarker, "click", function() {
          myMarker.openInfoWindowHtml(myMarker.infoHtml, {maxWidth: 300});
          });
        google.maps.Event.addListener(myMarker, "dragend", function() {
          var lat_box = document.getElementById("lat");
          var lng_box = document.getElementById("lng");
          var coords = this.getLatLng();
          lat_box.value = coords.lat();
          lng_box.value = coords.lng();
          });
        return myMarker;
        };
      function createIcon(iconUrl, shadowUrl) {
        var myIcon = new google.maps.Icon();
        myIcon.image = iconUrl;
        myIcon.shadow = shadowUrl;
        myIcon.size = new google.maps.Size(32,32);
        myIcon.shadowsize = new google.maps.Size(59,32);
        myIcon.iconAnchor = new google.maps.Point(16,32);
        myIcon.infoWindowAnchor = new google.maps.Point(16,0);
        return myIcon;
        };
      google.setOnLoadCallback(init);
    </script>
    <title>My first Google Map</title>
  </head>
  <body onunload="google.maps.Unload()">
    <div id="map" style="width: 800px; height: 800px; margin: auto; background-color: red">
      This is where the map will soon appear.
    </div>
    <div id="output">
      <label for="lat">Latitude:</label><input id="lat" type="text" value="Unknown" >
      <label for="lng">Longitude:</label><input id="lng" type="text" value="Unknown" >
    </div>
  </body>
</html>
```

At first glance that might look quite scary, but it’s really not that bad. You should be able to read through the script section and understand how it works.

## Additional

Hopefully you have installed the Firefox add-in called Firebug. If you have you can play around with the Google Maps API from the Javascript console built in to Firebug. Click on the little bug icon in the bottom left of the browser window and make sure the the console is enabled. Then you will see the console input area as shown by “&gt;&gt;&gt;” at the bottom of the screen. In to this section type:

```
<pre class="codestyle">newMarker1.hide();
```

and then

```
<pre class="codestyle">newMarker1.show();
```

Now you got something else to play with. You could use this to show only markers of a particular type. We’ll cover this next time.
