---
layout: post
title:  "Mapbox - Brittany rivers"
date:   2017-10-16 20:30:00
categories: jekyll update
---

<div id='map'></div>
<script>
  mapboxgl.accessToken = 'pk.eyJ1IjoiY2hpZW5jaGllbjE2NjQiLCJhIjoiY2o3dzl2ODBhNWZpejM0cXA0emY2eWdwbyJ9.7MMOUqu-jnU_xAFxy9qttA';
  var map = new mapboxgl.Map({
    container: 'map',
    style: 'mapbox://styles/chienchien1664/cj8unftwxegn12so03au46wfy',
	center: [-3,48.2],
	zoom: 7.5
  });
</script>
A first play with Mapbox.

Here we can see Brittany and its rivers. With a customized style.