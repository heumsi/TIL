참고한 링크

- https://medium.com/vis-gl/deckgl-and-mapbox-better-together-47b29d6d4fb1
- https://deck.gl/docs/api-reference/mapbox/overview#using-with-pure-js
- https://github.com/heumsi/deckgl-jupyter/tree/master/deckgljupyter/templates
- https://deck.gl/docs/api-reference/geo-layers/trips-layer

```html
<html>
  <head>
    <script
      src="https://code.jquery.com/jquery-3.5.1.js"
      integrity="sha256-QWo7LDvxbWT2tbbQ97B53yJnYU3WhH/C8ycbRAkjPDc="
      crossorigin="anonymous"
    ></script>
    <!-- deck.gl standalone bundle -->
    <script src="https://unpkg.com/deck.gl@^8.1.0/dist.min.js"></script>

    <style type="text/css">
      body {
        margin: 0;
        padding: 0;
      }
      #container {
        width: 100vw;
        height: 100vh;
        z-index: 1;
      }
      .slidecontainer {
        width: 300px; /* Width of the outside container */
        z-index: 999;
        background-color: aquamarine;
        position: fixed;
        left: 0;
        top: 0;
      }

      /* The slider itself */
      .slider {
        -webkit-appearance: none; /* Override default CSS styles */
        appearance: none;
        width: 100%; /* Full-width */
        height: 25px; /* Specified height */
        background: #d3d3d3; /* Grey background */
        outline: none; /* Remove outline */
        opacity: 0.7; /* Set transparency (for mouse-over effects on hover) */
        -webkit-transition: 0.2s; /* 0.2 seconds transition on hover */
        transition: opacity 0.2s;
      }

      /* Mouse-over effects */
      .slider:hover {
        opacity: 1; /* Fully shown on mouse-over */
      }

      /* The slider handle (use -webkit- (Chrome, Opera, Safari, Edge) and -moz- (Firefox) to override default look) */
      .slider::-webkit-slider-thumb {
        -webkit-appearance: none; /* Override default look */
        appearance: none;
        width: 25px; /* Set a specific slider handle width */
        height: 25px; /* Slider handle height */
        background: #4caf50; /* Green background */
        cursor: pointer; /* Cursor on hover */
      }

      .slider::-moz-range-thumb {
        width: 25px; /* Set a specific slider handle width */
        height: 25px; /* Slider handle height */
        background: #4caf50; /* Green background */
        cursor: pointer; /* Cursor on hover */
      }
    </style>
    <script src="https://api.tiles.mapbox.com/mapbox-gl-js/v1.10.0/mapbox-gl.js"></script>
    <link
      href="https://api.tiles.mapbox.com/mapbox-gl-js/v1.10.1/mapbox-gl.css"
      rel="stylesheet"
    />
    <script type="text/javascript">
      const { MapboxLayer, TripsLayer } = deck;
      mapboxgl.accessToken =
        "pk.eyJ1IjoiaGV1bXNpIiwiYSI6ImNqYng3ZW0xYTJsZHQycXBhM2F1bm9yMXIifQ.kV_zhG36r5EIXXT__LzlCw";
    </script>

    <script src="https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-language/v0.10.1/mapbox-gl-language.js"></script>
  </head>

  <body>
    <div id="container"></div>
    <div class="slidecontainer">
      <input
        type="range"
        min="1592021170"
        max="1593319453"
        value="1592021170"
        class="slider"
        id="myRange"
      />
    </div>
  </body>

  <script type="text/javascript">
    const trip_data = [
      {
        waypoints: [
          { coordinates: [127.206846, 37.529978], timestamp: 1592021170 },
          { coordinates: [127.724522, 37.398283], timestamp: 1592049781 },
          { coordinates: [127.028764, 37.55874], timestamp: 1592060025 },
          { coordinates: [127.04568, 37.684594], timestamp: 1592099136 },
          { coordinates: [128.631595, 38.117094], timestamp: 1592120052 },
          {
            coordinates: [128.28618400000002, 37.915329],
            timestamp: 1592129815,
          },
          { coordinates: [127.057492, 37.675301], timestamp: 1592143356 },
          {
            coordinates: [127.07783899999998, 37.54392],
            timestamp: 1592200444,
          },
          { coordinates: [127.510112, 37.831604], timestamp: 1592355942 },
          { coordinates: [127.528256, 37.748289], timestamp: 1592359605 },
          { coordinates: [127.054899, 37.549042], timestamp: 1592396026 },
          {
            coordinates: [126.92818500000001, 37.561913],
            timestamp: 1592636778,
          },
          {
            coordinates: [126.958399, 37.476409000000004],
            timestamp: 1592650951,
          },
          { coordinates: [126.948024, 37.466663], timestamp: 1592711952 },
          { coordinates: [127.068149, 37.538725], timestamp: 1592814380 },
          { coordinates: [127.068149, 37.538725], timestamp: 1592816404 },
          { coordinates: [128.591454, 37.122398], timestamp: 1592880924 },
          { coordinates: [127.724535, 37.398275], timestamp: 1592895252 },
          {
            coordinates: [127.05493799999999, 37.548991],
            timestamp: 1592903509,
          },
          {
            coordinates: [127.04318300000001, 37.560827],
            timestamp: 1592921511,
          },
          {
            coordinates: [127.07306399999999, 37.538501000000004],
            timestamp: 1592970625,
          },
          { coordinates: [127.043124, 37.560705], timestamp: 1592976368 },
          { coordinates: [127.067447, 37.530404], timestamp: 1592977045 },
          { coordinates: [127.055199, 37.548952], timestamp: 1592999951 },
          {
            coordinates: [127.01228799999998, 37.568771000000005],
            timestamp: 1593003768,
          },
          {
            coordinates: [126.89031000000001, 37.507373],
            timestamp: 1593238943,
          },
          { coordinates: [126.770324, 37.846677], timestamp: 1593311950 },
          { coordinates: [126.692704, 37.712975], timestamp: 1593318130 },
          { coordinates: [126.693896, 37.789178], timestamp: 1593319453 },
          {
            coordinates: [126.699155, 37.788940000000004],
            timestamp: 1593327865,
          },
          {
            coordinates: [126.89442, 37.499272999999995],
            timestamp: 1593334407,
          },
          {
            coordinates: [127.09616799999999, 37.510905],
            timestamp: 1593425764,
          },
          { coordinates: [127.06867, 37.517641], timestamp: 1593429643 },
          { coordinates: [127.06867, 37.517641], timestamp: 1593439848 },
        ],
      },
    ];

    const map = new mapboxgl.Map({
      container: "container",
      style: "mapbox://styles/mapbox/dark-v10",
      center: [126.73, 37.715],
      zoom: 10,
    });

    // var language = new MapboxLanguage({
    //   defaultLanguage: "ko",
    // });
    // map.addControl(language);

    var myDeckLayer;
    map.on("load", () => {
      myDeckLayer = new MapboxLayer({
        id: "trips_layer",
        type: TripsLayer,
        data: trip_data,
        getPath: (d) => d.waypoints.map((p) => p.coordinates),
        getTimestamps: (d) => d.waypoints.map((p) => p.timestamp),
        getColor: [253, 128, 93],
        getWidth: 200,
        opacity: 0.8,
        widthMinPixels: 10,
        widthMaxPixels: 100,
        rounded: true,
        trailLength: 200,
        currentTime: 1593429643,
      });
      map.addLayer(myDeckLayer);
    });

    $(document).ready(() => {
      var slider = $("#myRange");

      slider.change(() => {
        console.log(myDeckLayer);
        console.log(myDeckLayer.props.currentTime);

        myDeckLayer.setProps({
          currentTime: parseInt(slider.val()),
          //   currentTime: 1592977045,
        });
      });
    });
  </script>
</html>

```

