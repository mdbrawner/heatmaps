# Heatmaps Tutorial

> Spring 2019 | Geography 4/572 | Geovisual Analytics
>
> **Instructor:** Bo Zhao  **Location:** Wilkinson 210 | **Time:** TR 1600 - 1650 |
> **Tutorial by:** Nichael Brawner

**Learning Objectives**

- Learn how to use the leaflet.heatmap plugin to visually aggregate data

#### JavaScript Libraries
### leaflet.heat
This tutorial will use `leaflet-heatmap.js` which is a javascript library that uses [simpleheat](https://github.com/mourner/simpleheat) under the hood to aggregate points. The `leaflet-heat.js` library can be found at the [leaflet.heat](https://github.com/Leaflet/Leaflet.heat) github page in the dist folder.

![](img/chart.png)
### Leaflet

This tutorial will also use the `leaflet` javascript library to render a map.


##### Other Libraries used:
- Leaflet
- jQuery
- Chroma

### Introduction
In this tutorial, we will produce a heatmap showing densitites of American kestrels overwintering in Oregon.  The American kestrel is a small falcon that breeds in open grasslands and is a partial migrant, meaning that some populations do not migrate, but many do.  Oregon is a hotspot for raptor species migrating from their breeding grounds further north to spend the winter.  Here we will just be looking at a single species.  This map will give us a visual representation of where kestrels are most populated in the state over the months of Decemnber, January and February.

#### Data
The data consist of counts of kestrels along survey routes taken over the 2017/2018 Winter season.  Each survey route is ~60 miles long and the surveys occur twice a month.  The point location data is taken from the centroid locations of each route. If you look at the GeoJSON `WRS_kestrels.geojson` in the assests folder, you will see that each route contains lat/long coordinate data and a count for the average number of kestrels on that route over the time period labeled as the property `Kestrels`.  We will use these two piece of data to build our heatmap.

### Code

##### Initialize a HTML page

As usuall, we will create an empty html page.

```html
<!DOCTYPE html>
<html>
    <head>
    </head>
    <body>
    </body>
</html>
```

##### Libraries and CSS Styling
Here in the head of the html we'll include the following libraries and css styling:

```html
<head>
    <title>Kestrels Heat Map </title>
    <style>
        html, body, #map { width: 100%; height: 100%; margin: 0; background: #fff}
    </style>

    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.4.0/dist/leaflet.css" />

    <script src="https://unpkg.com/leaflet@1.4.0/dist/leaflet.js"></script>
    <script src="http://leaflet.github.io/Leaflet.heat/dist/leaflet-heat.js"></script>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chroma-js/1.3.4/chroma.min.js"></script>
</head>
```
##### Create placeholders
In the first part of the body section, we define a placeholder for the map and the legend.

```html
<body>
  <div id="map"></div>
  <div id="panel"></div>

</body>
```

##### Create a map object and add base map

Now we will create a map object between `<script></script>` tags within the body.

```javascript
<script>

var map = L.map('map').setView([44.1, -120.5], 6);

L.tileLayer( 'http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; \<a href="http://openstreetmap.org"\>OpenStreetMap\</a\> Contributors',
    maxZoom: 10,
    minZoom: 3,
}).addTo(map);

</script>
```
##### Now we create the heat map

To do this, we will use the `getJSON` method and create a for loop to retrieve the data from the geoJSON file.  In the loop we need to create an array that has latitude, logitude and the count data, which will be read as intensity by the heatLayer.  Next we define the heatLayer calling kestrels_data and setting the parameters of the heat map.
All of this we do inside the script tags.
```javascript
var heat = null;

// Get GeoJSON and put on it on the map when it loads
$.getJSON("assets/WRS_kestrels.geojson",function(data){
    // set kestrels_data to the dataset, and add the kestrels GeoJSON layer to the map
    kestrels_data = [];
    for (var i = 0; i < data.features.length; i++ ) {
        kestrels_data[i] = [
            data.features[i].geometry.coordinates[1],
            data.features[i].geometry.coordinates[0],
            data.features[i].properties.Redtail]

    }



    heat = L.heatLayer(kestrels_data,{
        minOpacity: 0,
        radius: 35,
        blur: 7,
        maxZoom: 17,
        max:1,
        gradient: {0.07: 'blue', 0.172: 'lime',  0.272: 'yellow', 0.35: 'red'},
        // gradient: {0.05: 'white', 0.25: 'orange',  0.27: 'yellow', 0.3: 'red'}
    }).addTo(map);
  });
```

##### Creating a legend
Now that we have a heatmap, we would like to create a legend to demarcate what the colors mean.  We will create a simple lengend here dynamically generating a color scheme to match our map.  First we need to set the css for the div `panel` that we created earlier.  Add the following code to the head in the css tags.

###### Chart Total
```html
i {
  width: 0.5px;
  height: 16px;
  float: right;
}
#panel {
  position: absolute;
  height: 75px;
  z-index: 1000;
  padding: 5px;
  top: 10px;
  right: 10px;
  background-color: lightgray;
  text-align: center;
  opacity: 0.7;
  color: blue;

}

#panel p {
  font-size: 12px;
  line-height: 16px;
  margin: 0;
  text-align: center;
}
#panel p2 {
  font-size: 10px;
  line-height: 12px;
  margin: 0;
  text-align: center;
}
```
###### Chart Districts

```javacript
var chart2 = c3.generate({
    title: {
      text: 'Refugee Arrival by District (D)'
    },
    data: {
        x: 't',
        columns: [t, d1, d2, d3, d4, d5, d6,d7, d8, d9, d10, d11, d12],
        type: 'spline',
    },
    axis: {
        x: {
            type: 'timeseries',
            tick: {
                format: '%Y-%m'
            }
        },
        y: {
          tick: {
              format: ''
          },
          label: {
              text: '# of Refugees',
              position: 'outer-middle'
          }
        },
    },
    point: {
      r: 0,
      focus: {
        expand: {
          r: 2
        }
      }
    },
    zoom: {
      enabled: {
        type: "drag"
      },
      onzoomend: function(d) {
        chart.zoom(chart2.zoom());
      }
    },
    tooltip: {
      linked: true
    },
      bindto: "#district"
});
```
##### Add Geojson data onto map
This code allows you to add geojson data to the leaflet map created earlier in the tutorial.

```javacript
districts = L.geoJSON(datasets[1], {
    onEachFeature: function(feature, layer) {
      layer.bindPopup(feature.properties.NAME_EN)
    },
    style: {
      opacity: 0.8,
      fillOpacity: 0,
      weight: 2,
      color: "brown",
      dashArray: '6',
    }
  }).addTo(mymap);
```
