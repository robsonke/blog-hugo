---
author: admin
comments: true
date: 2011-07-01 15:56:36+00:00
layout: post
slug: simple-google-maps-routeplanner
title: Simple Google Maps routeplanner
wordpress_id: 256
categories:
- Software
tags:
- google
- jquery
- LinkedIn
- maps
---

Last week I was googling for a simple implementation of a routeplanner based on Google maps. Weirdly enough I didn't find a total solution or older ones based on the maps v2 API. So I decided to have a look at the API and In 15 minutes I had a working routeplanner. Which might explain why I found that less examples on the internet :). But for all other people who need a quick solution, I thought sharing would be good.

It's a very plain example based on the Google Maps v3 and a bit of jQuery but that can easily be avoided.

First the necessary html:
[code language="html"]

<script type="text/javascript" src="http://maps.google.com/maps/api/js?sensor=true"></script> 
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.6.1/jquery.min.js" type="text/javascript"></script>

<style>
  #map_canvas {
    height:500px;
    width:600px;
 }
 #directions {
   width:600px;
 }
</style>

<div id="map_canvas"></div>
<h2>Plan your trip</h2>
<form id="routeform">
  City or zipcode:
  <input type="text" name="routefrom" id="routefrom"/>
  <input type="submit" value="calculate"/>
</form>
<div id="directions"></div>
[/code]


And the Javascript:
[code language="javascript"]
$(document).ready(function()
{
  initialize();
  
  $('#routeform').submit(function(e)
  {
    calcRoute();
    return false;
  });

});

var directionDisplay;
var directionsService = new google.maps.DirectionsService();
var map;
var home = new google.maps.LatLng(52.0951, 5.109);

function initialize() 
{
  directionsDisplay = new google.maps.DirectionsRenderer();

  var myOptions = 
  {
    zoom:8,
    mapTypeId: google.maps.MapTypeId.ROADMAP,
    center: home
  }
  map = new google.maps.Map(document.getElementById("map_canvas"), myOptions);
  directionsDisplay.setMap(map);
  directionsDisplay.setPanel(document.getElementById("directions"));
  
  var myLatLng = new google.maps.LatLng(52.0951, 5.109);
  var beachMarker = new google.maps.Marker
  ({
      position: myLatLng,
      map: map
  });
}

function calcRoute() 
{
  var start = document.getElementById("routefrom").value;

  var request = 
  {
    origin:start, 
    destination:home,
    travelMode: google.maps.DirectionsTravelMode.DRIVING
  };
  directionsService.route(request, function(response, status) 
  {
    if (status == google.maps.DirectionsStatus.OK) 
    {
      directionsDisplay.setDirections(response);
    }
  });
}
[/code]

Which results in:







  




  
  



## Plan your trip



  City or zipcode:
  
  

  
  




  
  

