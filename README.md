<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1">
	
	<title>WEBGIS COVID JEMBER
	</title>
	
	<link rel="shortcut icon" type="image/x-icon" href="docs/images/favicon.ico" />

    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.8.0/dist/leaflet.css" integrity="sha512-hoalWLoI8r4UszCkZ5kL8vayOGVae1oxXe/2A4AO6J9+580uKHDO3JdHb7NzwwzK5xr/Fs0W40kiNHxM9vyTtQ==" crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.8.0/dist/leaflet.js" integrity="sha512-BB3hKbKWOc9Ez/TAwyWxNXeoV9c1v6FIeYiBieIWkpLjauysF18NzgR1MBNBXf8/KABdlkX68nAhlwcDFLGPCQ==" crossorigin=""></script>
	<link rel="shortcut icon" type="image/x-icon" href="docs/images/favicon.ico" />
    <link rel="stylesheet" href="assets/leaflet/leaflet.css" />
    <link rel="stylesheet" href="assets/leaflet.label.css" />
    <link href="docs/images/search_icon.png" rel="stylesheet">
    <link rel="stylesheet" href="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.css" />
    <script src="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.js"></script>


<style>
#map { height: 100vh; }
</style>

<style>
   .info
 {
        padding: 6px 8px;
        font: 14px/16px Verdana, Geneva, sans-serif;
        background: white;
        background: rgba(255, 255, 255, 0.8);
        box-shadow: 0 0 15px rgba(244, 255, 243, 0.2);
        border-radius: 5px;
 }

.legend {
        line-height: 18px;
        color: rgb(0, 0, 0);
    }

.legend i {
        width: 30px;
        height: 15px;
        float: left;
        margin-right: 10px;
        opacity: ;
    }
</style>
</head>
<body>

<div id='map'></div>

<script type="text/javascript" src="assets/covid.js"></script>

<script type="text/javascript">


//Basemapm Set View Lokasi
var map = L.map('map').setView([-8.195163154101227, 113.60332070477767], 10);

var OpenStreetMaps = L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}', {
    attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
    maxZoom: 18,
    id: 'mapbox/streets-v11',
    tileSize: 512,
    zoomOffset: -1,
    accessToken: 'pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NXVycTA2emYycXBndHRqcmZ3N3gifQ.rJcFIG214AriISLbB6B5aw'
});

var googleHybrid = L.tileLayer('http://{s}.google.com/vt/lyrs=s,h&x={x}&y={y}&z={z}',{
    maxZoom: 20,
    subdomains:['mt0','mt1','mt2','mt3']
});
var googleStreets = L.tileLayer('http://{s}.google.com/vt/lyrs=m&x={x}&y={y}&z={z}',{
    maxZoom: 20,
    subdomains:['mt0','mt1','mt2','mt3']
});

var googleSat = L.tileLayer('http://{s}.google.com/vt/lyrs=s&x={x}&y={y}&z={z}',{
    maxZoom: 20,
    subdomains:['mt0','mt1','mt2','mt3']
}); 

var googleTerrain = L.tileLayer('http://{s}.google.com/vt/lyrs=p&x={x}&y={y}&z={z}',{
    maxZoom: 20,
    subdomains:['mt0','mt1','mt2','mt3']
});

var baselayers = {
    "OSM" :OpenStreetMaps,
    "Google Hybrid": googleHybrid,
    "Google Streets" :googleStreets,
    "Google Sattelite" :googleSat,
    "Google Terrain" :googleTerrain
}

var group = L.layerGroup([OpenStreetMaps, googleHybrid, googleStreets, googleSat, googleTerrain]).addTo(map);

var layerControl = L.control.layers(baselayers).addTo(map);

// Search Box

var geocoder = L.Control.geocoder({
  defaultMarkGeocode: true
}).addTo(map);

// Information on Hover
	var info = L.control();

	info.onAdd = function (map) {
		this._div = L.DomUtil.create('div', 'info');
		this.update();
		return this._div;
	};

	info.update = function (props) {
		this._div.innerHTML = '<h4>Kasus Covid Kabupaten Jember</h4>' +  (props ?
			'<b>' +'Kecamatan ' + props.NAME_3 + '</b><br />' + 'Kasus: ' + props.Interval + ' / Kecamatan' + '</b><br />'+ 'Kategori: '+ props.Zona: 'Informasi Kasus Covid-19');
	};

	info.addTo(map);

// Home Button


// Symbology Warna (kalau angka gunakan d : angka tanpa petik)
	function getColor(d) {
		return d > 1000 ? '#800026' :
			d == 'Zona Tidak Ada Kasus'  ? '#3D9A07' :
			d == 'Zona Resiko Rendah'  ? '#ffff00' :
			d == 'Zona Resiko Sedang'   ? '#F18F01' :'#59FD02';
	}

	function style(feature) {
		return {
			weight: 1,
			opacity: 1,
			color: 'black',
			dashArray: '1',
			fillOpacity: 0.7,
			fillColor: getColor(feature.properties.Zona)
		};
	}

// Highlight
	function highlightFeature(e) {
		var layer = e.target;

		layer.setStyle({
			weight: 2,
			color: '#40E0D0',
			dashArray: '',
			fillOpacity: 0.7
		});

		if (!L.Browser.ie && !L.Browser.opera && !L.Browser.edge) {
			layer.bringToFront();
		}

		info.update(layer.feature.properties);
	}

	var geojson;

	function resetHighlight(e) {
		geojson.resetStyle(e.target);
		info.update();
	}

	function zoomToFeature(e) {
		map.fitBounds(e.target.getBounds());
	}

	function onEachFeature(feature, layer) {
		layer.on({
			mouseover: highlightFeature,
			mouseout: resetHighlight,
			click: zoomToFeature
		});
	}

// Load Data Geojson
	geojson = L.geoJson(data_covid, {
		style: style,
		onEachFeature: onEachFeature
	}).addTo(map);

	map.attributionControl.addAttribution('Population data &copy; <a href="http://census.gov/">US Census Bureau</a>');

// Legenda
var legend = L.control({position: 'bottomleft'});
legend.onAdd = function (map) {

var div = L.DomUtil.create('div', 'info legend');
labels = ['<strong>Legenda:</strong>'],
categories = ['Zona Tidak Ada Kasus','Zona Resiko Rendah','Zona Resiko Sedang'];

for (var i = 0; i < categories.length; i++) {

        div.innerHTML += 
        labels.push(
            '<i class="circle" style="background:' + getColor(categories[i]) + '"></i> ' +
        (categories[i] ? categories[i] : '+'));

    }
    div.innerHTML = labels.join('<br>');
return div;
};
legend.addTo(map);



</script>



</body>
</html>

