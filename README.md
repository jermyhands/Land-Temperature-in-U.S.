# Land-Temperature-in-U.S.
A web map that allows users to interact with temperature data between the years 1849-2013 in cities in the U.S.

<!DOCTYPE html>
<html>
<head>
    <title>Land Temperature in U.S. Cities</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://unpkg.com/leaflet@1.3.4/dist/leaflet.css" rel="stylesheet">
    <link rel="stylesheet" href="https://code.jquery.com/ui/1.9.2/themes/base/jquery-ui.css" type="text/css">
    <link rel="stylesheet" href="./style.css">
    <link rel="stylesheet" href="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.css" />
    
</head>
<body>
  <h1>Land Temperature in U.S. Cities</h1>
  
  
    <script src="https://unpkg.com/leaflet@1.3.4/dist/leaflet.js"></script>
    <script src="https://code.jquery.com/jquery-1.9.1.min.js"></script>
    <script src="https://code.jquery.com/ui/1.9.2/jquery-ui.js"></script>
    <script src="https://rawgit.com/dwilhelm89/LeafletSlider/master/SliderControl.js" type="text/javascript"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mustache.js/3.0.0/mustache.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.4.1/leaflet.markercluster.js"></script>
    <script src="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.js"></script>
    <div id="map" style="width: 90%; height: 500px; left: 75px; position: relative; "> </div>
  
    
    <script src="./script.js"></script>
</body>
</html>
----------------------------------------------------------------------------------------------------------------------------------------
/* global L */


var sliderControl = null;
var myMap = L.map('map').setView([39.82, -98.58], 4);
L.tileLayer('http://services.arcgisonline.com/arcgis/rest/services/World_Street_Map/MapServer/tile/{z}/{y}/{x}', {
    attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
}).addTo(myMap);

function getColor(d) {
    return  d >= 20   ? '#b2182b' : 
            d >= 15   ? '#f4a582' :
            d >= 10   ? '#fddbc7' :
            d >= 0    ? '#d1e5f0' :
            d <= 0    ? '#92c5de' :
            d <= -2   ? '#4393c3' :
            d <= -5   ? '#2166ac': 
            d <= -10  ? '#053061':
                        'black'; 
}

var url = "https://cdn.glitch.com/eacace4b-b435-40a2-a61a-448502b9b12f%2Fustemp_geo.geojson?1539202207545";

function getResp(response) {
    return response.json();
}



function getData(data) {
    console.log(data);
    var sliderControl = null;
    //Create a marker layer 
    var testlayer = L.geoJson(data, {
        pointToLayer: function (feature, latlng) {
            return L.circleMarker(
                latlng, {
                    fillColor: getColor(feature.properties.AverageTemperature), 
                    // bubblingMouseEvents: true,
                    fillOpacity: 1,
                    radius: 10,
                    weight: 0,
                }
            );
        },

        onEachFeature: function (feature, layer) {
            layer.bindPopup(
                "<h2>" + feature.properties.City + "</h2>" + "<p>" + "<h3>" +"Temp. in Celsius:"+ feature.properties.AverageTemperature + "</h3>" + 
                "<p>" + feature.properties.dt
            );
        }
    });

    sliderControl = L.control.sliderControl({
        position: "topright",
        layer: testlayer,
        timeAttribute: "dt",
        follow: false,
        range: false,
        alwaysShowDate : true,
    });

    
    myMap.addControl(sliderControl);

    //And initialize the slider
    sliderControl.startSlider();
}

fetch(url)
    .then(getResp)
    .then(getData)

var legend = L.control({position: 'bottomleft'});

legend.onAdd = function (myMap) {

    var div = L.DomUtil.create('div', 'info legend'),
        labels = ['<strong> THE TITLE </strong>'],
        grades = [-10, -5,-2, 0, 5, 10, 15, 20];

    // loop through our density intervals and generate a label with a colored square for each interval
    for (var i = 0; i < grades.length; i++) {
        div.innerHTML +=
            '<i style="background:' + getColor(grades[i] + 1) + '"></i> ' +
            grades[i] + (grades[i + 1] ? '&ndash;' + grades[i + 1] + '<br>' : '+');
    }

    return div;
};

legend.addTo(myMap);

  var geocoder = L.Control.geocoder({
        defaultMarkGeocode: false,
        position: "bottomright",
        placeholder: "Search a City",
        suggestMinLength: 3,
    })
    .on('markgeocode', function(e) {
        var bbox = e.geocode.bbox;
        var poly = L.polygon([
             bbox.getSouthEast(),
             bbox.getNorthEast(),
             bbox.getNorthWest(),
             bbox.getSouthWest()
        ]).addTo(myMap);
        myMap.fitBounds(poly.getBounds());
    })
    .addTo(myMap);
----------------------------------------------------------------------------------------------------------------------------------------
/* CSS files add styling rules to your content */

body {
  font-family:"Courier New", Courier, monospace;
  margin: 2em;
}

h1 {
  position: relative;
  color: #fff;
  text-shadow: 1px 0 0 #000, 0 -1px 0 #000, 0 1px 0 #000, -1px 0 0 #000;
  text-align: center;
  font-size: 16;
  background-color: rgba(64, 58, 58, 0.42);
  
  border-radius: 25px;
 
  margin-left: 300px;
  margin-right: 300px;
}

.legend i {
    width: 18px;
    height: 18px;
    float: left;
    margin-right: 8px;
    opacity: 0.7;
    background-color: white;
}
