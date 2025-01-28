# road-geomap

### References
1. https://docs.mapbox.com/mapbox-gl-js/example/timeline-animation/


### How to make this web map?
#### 1. Establish dataset.

Make a GeoJSON that represents the data. It must have the attribute `geometry` and can be generated using the latitude and longitude of the dataset. The GeoJSON can also have other attritubutes such as classification, datetime, and the link of the image.
#### 2. Start a leaflet project.

Leaflet is used to create interactive web maps. It can be used to display map data and create customized dynamic mapping applications. We can start by creating a folder that contains an `.html` file. This will contain our codebase. We can start a simple leaflet project by adding a map layer and add a marker.

```html
<html>
<head>
    <title>Simple Map</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
</head>
<body>
    <div id="map" style="height: 400px;"></div>
    <script>
        <div id="map"></div>
        // Initialize the map and set its view
        var map = L.map('map').setView([10.76976389, 106.7003694], 13);

        // Add OpenStreetMap tiles
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 19,
            attribution: '© OpenStreetMap contributors'
        }).addTo(map);

        // Add a marker
        L.marker([10.76976389, 106.7003694]).addTo(map)
            .bindPopup('A simple popup.<br> Easily customizable.')
            .openPopup();
    </script>
</body>
</html>
```
#### 3. Load the dataset.

We can add our dataset in the map to see if our data works. We will also add function later once we can properly load our data in the map. We can remove our marker at this point. The output of this step must show points on the map of the locations of the dataset.

```html
<body>
    <div id="map"></div>
    <script>
        // Rest of the code here

        // Get the content of the geojson from a remote server
        var geojsonLayer;
        var geojsonData;
        fetch('https://raw.githubusercontent.com/nikkopante/road-geomap/refs/heads/main/road_use_classification.geojson')
            .then(res => res.json())
            .then(data => {
                geojsonData = data;
                showDataset();
            });

        function showDataset() {
            geojsonLayer = L.geoJSON(geojsonData);
            geojsonLayer.addTo(map);
        };

    </script>
</body>
```
#### 4. Add a slider.

The slider will control or filter what shows on the map. To test this, you check the console of the browser.

```html
<body>
    <div id="map"></div>
    <div class="map-overlay top">
        <!-- Add a slider -->
        <div class="map-overlay-inner">
            <h2>Time Slider</h2>
            <label id="time"></label>
            <input id="slider" type="range" min="0" max="15" step="1" value="0">
        </div>
    </div>
    <script>
        // Rest of the code here

        const slider = document.getElementById('slider');
        const sliderValueDisplay = document.getElementById('time');

        sliderValueDisplay.textContent = parseInt(slider.value)

        slider.addEventListener('input', (e) => {
            const sliderValue = parseInt(slider.value);
            sliderValueDisplay.textContent = sliderValue;
            console.log("slider value", sliderValue) // To test if our browser reads the index of our slider
        });
        

    </script>
</body>
```

#### 5. Filter the dataset based on time
To filter the dataset, we have to make a function that is in sync with the time slider. In order to do this we have to make another dataset of datetime intervals. This can be hardcoded in the codebase `main.html` but it can also be separate file such as JSON. I have an `hourly_intervals.json` in this repo with this list of dictionary. The startdate and enddate can be used to filter what subset from our GeoJSON coincides with a time interval.
```json
[
    {
        "startdate":"2023-05-06T05:00:00",
        "enddate":"2023-05-06T06:00:00"
    },
    {
        "startdate":"2023-05-06T06:00:00",
        "enddate":"2023-05-06T07:00:00"
    },
]
```

Now we have to make it work with our time slider. Each time interval has an index beginning from 0, 1, 2, and so on. We also have to use fetch just like in our GeoJSON to get the data stream. 
```html
<body>

    <script>
        var timeIntervals;
        fetch('https://raw.githubusercontent.com/nikkopante/road-geomap/refs/heads/main/hourly_intervals.json')
            .then(res => res.json())
            .then(data => {
                timeIntervals = data;
                sliderValueDisplay.textContent = timeIntervals[0].startdate // Set initial display to index 0
            });

        slider.addEventListener('input', (e) => {
            const sliderValue = parseInt(slider.value);
            sliderValueDisplay.textContent = sliderValue;
        });
        
    </script>
</body>
```

The GeoJSON layer `var geojsonLayer;` is responsible for controlling what dataset to show on our map. We have to wrap it inside a function that filters dataset based on time.
```html
<body>
    <script>

            var geojsonLayer;
            function styleFeature(feature) {
                return {
                    radius: 8,
                    color: "#000",
                    weight: 1,
                    opacity: 1,
                    fillOpacity: 1
                };
            }

            var geojsonData;
            fetch('https://raw.githubusercontent.com/nikkopante/road-geomap/refs/heads/main/road_use_classification.geojson')
                .then(res => res.json())
                .then(data => {
                    geojsonData = data;
                    filterData(0); // Initially filter our dataset using time interval index 0
                });

            function filterData(sliderValue) {
                if (geojsonLayer) {
                    map.removeLayer(geojsonLayer);
                }
                geojsonLayer = L.geoJSON(geojsonData, {
                    filter: function(feature) {
                        console.log(timeIntervals[sliderValue].startdate, timeIntervals[sliderValue].enddate)
                        return feature.properties.DateTime >= timeIntervals[sliderValue].startdate && 
                                feature.properties.DateTime <= timeIntervals[sliderValue].enddate;
                    },
                    pointToLayer: function(feature, latlng) {
                        return L.circleMarker(latlng, styleFeature(feature));
                    },
                });
                geojsonLayer.addTo(map);
            }

        slider.addEventListener('input', (e) => {
            const sliderValue = parseInt(slider.value);
            sliderValueDisplay.textContent = sliderValue;
            filterData(sliderValue); // Call our function everytime the slider change its value
        });
        
    </script>
</body>
```
