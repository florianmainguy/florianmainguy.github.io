---
layout: post
title:  "Mapbox Lab"
date:   2017-09-22 20:30:00
categories: jekyll update
---

<div id='map' style='width: 400px; height: 300px;'></div>
<script>
  mapboxgl.accessToken = 'pk.eyJ1IjoiY2hpZW5jaGllbjE2NjQiLCJhIjoiY2o3dzl2ODBhNWZpejM0cXA0emY2eWdwbyJ9.7MMOUqu-jnU_xAFxy9qttA';
  var map = new mapboxgl.Map({
    container: 'map',
    style: 'mapbox://styles/mapbox/outdoors-v10'
  });
</script>
