****************************************************************************
MapLibre with Svelte
****************************************************************************

1. Introduction
=================

This is a tutorial to display a map an interactive map with Svelte using MapLibre GL JS.
Based on the amptiler tutorial: https://docs.maptiler.com/svelte/maplibre-gl-js/how-to-use-maplibre-gl-js/

The web map will have the following functionalities:

- Visualize and query a Vector Tile layer from Catalonia municipalities
- Controll which municipalities are visible through a sheet hosted on a GoogleSpreadSheet.
- Append data to each municipality entity. Data available on IDESCAT API.


2. Create an app
=================

First, we are going to create a Svelte app.
To create a new Svelte project, run in the command-line:

.. code-block:: console

  npx degit sveltejs/template my-svelte-map



Now navigate to the newly created project folder **my-svelte-map**:


.. code-block:: console

  cd my-svelte-map


Inside this folder, run ``npm install`` to install the dependencies.

To start the local environment, run ``npm run dev``. An type the address on your prefered browser: ``http://localhost:5000``.

You'll see the app in your browser.


3. Install MapLibre and set up
================================

To install MapLibre GL, navigate to the project folder and run the command:

.. code-block:: console

  npm i maplibre-gl


Navigate to the ``src`` folder and delete the props name of the ``main.js`` file. Your ``main.js`` file should look like this:

.. code-block:: javascript

  import App from './App.svelte';

  const app = new App({
  target: document.body,
  props: {}
  });

  export default app;


Delete all the content of the App.svelte file and write the following lines in the App.svelte file

.. code-block:: javascript

  <script>

  </script>

  <div class="app">
    This is my map App
  </div>

  <style>
  .app {
    text-align: center;
  }
  </style>

You shold visualize the changes in your browser.


4. Create a navbar component
================================

In this step, we will create a simple heading navbar component.

Create a new folder called ``components`` indide the ``src`` folder.

Create a new file called ``Navbar.svelte`` inside the ``components`` folder and write these lines:

.. code-block:: javascript

  <div class="heading">
  <h1>This is my map App</h1>
  </div>

  <style>
  .heading {
    margin: 0;
    padding: 0px;
    background-color: black;
    color: white;
  }

  .heading > h1 {
    padding: 20px;
    margin: 0;
  }
  </style>


Finally, to display the **Navbar** we need to import the Navbar component and add it to our main component ``App.svelte``.

Import the navbar component into ``App.svelte`` script block:


.. code-block:: javascript

  <script>
    import Navbar from './components/Navbar.svelte';
  </script>

Replace the text *This is my map App* with ``<Navbar/>``. Your ``App.svelte`` file should look like this:

.. code-block:: javascript

  <script>
    import Navbar from './components/Navbar.svelte';
  </script>

  <div class="app">
    <Navbar />
  </div>

  <style>
    .app {
      text-align: center;
    }
  </style>

Now you should see the black navbar at the top of your browser.

5. Create a map component
===========================

Now we are going to create the map component.

Create a new file called ``Map.svelte`` inside the ``components`` folder and write these lines of code:


.. code-block:: javascript

  <script>
    import { onMount, onDestroy } from 'svelte'
    import { Map } from 'maplibre-gl';
    import 'maplibre-gl/dist/maplibre-gl.css';

    let map;
    let mapContainer;

    onMount(() => {

      const initialState = { lng: 1.4, lat: 41.6, zoom: 7 };

      map = new Map({
        container: mapContainer,
        style: `https://demotiles.maplibre.org/style.json`,
        center: [initialState.lng, initialState.lat],
        zoom: initialState.zoom
      });

    });

    onDestroy(() => {
      map.remove();
    });
  </script>

  <div class="map-wrap">
    <div class="map" id="map" bind:this={mapContainer}></div>
  </div>

  <style>

    .map-wrap {
      position: relative;
      width: 100%;
      height: calc(100vh - 77px); /* calculate height of the screen minus the heading */
    }

    .map {
      position: absolute;
      width: 100%;
      height: 100%;
    }

    .watermark {
      position: absolute;
      left: 10px;
      bottom: 10px;
      z-index: 999;
    }
  </style>

1. The ``container`` option sets the DOM element in which the map will be rendered. We'll assign the ``mapContainer`` ref expected by our component to an HTML element, which will act as a container. Keep in mind that the reference to ``mapContainer`` can only be used after the execution of the ``onMount`` lifecycle function.
2. THe ``style`` option defines what style is the map going to use.
3, The ``center`` and ``zoom`` options set the starting position of the map.
4. The ``onDestroy`` fuction does the cleanup that should occur when the instance is destroyed.


To display the Map we need to import the map component and add it to our main component ``App.svelte``.

Import the map component into the ``App.svelte`` script block

.. code-block:: javascript

  <script>
    import Navbar from './components/Navbar.svelte';
    import Map from './components/Map.svelte';
  </script>


And add the ``<Map/>`` just below the Navbar in the template section. The template block should look like this

.. code-block:: javascript

  <div class="app">
    <Navbar />
    <Map />
  </div>


5. Map Controls
===========================

We'll add navigation controls to our map.

Add the ``NavigationControl`` next to the Map object import from MapLibre GL.

.. code-block:: javascript

  import { Map, NavigationControl } from 'maplibre-gl';

And, just after the initialization of the map, on ``Map.svelte`` file, add the following line:

.. code-block:: javascript

  map.addControl(new NavigationControl(), 'top-right');


6. Add a VectorTile Layer
===========================

We'll add an open data source of vector data provided by the ICGC, with a dataset of municipalities of Catalonia.
Using this data source, we'll create and style two different layers (one for associated municipalites, and another one for none associated municipalities), and later we'll upload them to the map.


``ASSOCIATED_MUNICIPALITIES`` and ``NO_ASSOCIATED_MUNICIPALITES`` are stored in the javascript code as constant variables. Later will show how to get this data from a google spreadsheet.

This is the code:

.. code-block:: javascript

  <script>
    import { onMount, onDestroy } from 'svelte'
    import { Map, NavigationControl } from 'maplibre-gl';
    import 'maplibre-gl/dist/maplibre-gl.css';

    let map;
    let mapContainer;

    onMount(() => {

      const initialState = { lng: 1.4, lat: 41.6, zoom: 7 };

      const ASSOCIATED_MUNICIPALITIES = [
      	"in",
      	"name",
      	'Abrera', 'Àger', 'Agramunt', 'Agullana', 'Aiguaviva', 'Alàs i Cerc',
      ]

      ASSOCIATED_MUNICIPALITIES.push('Olot', 'Sant Joan les Fonts')

      const NO_ASSOCIATED_MUNICIPALITIES = [
      	"in",
      	"name",
      	'Abrera', 'Àger', 'Agramunt', 'Agullana', 'Aiguaviva', 'Alàs i Cerc',
      ]


      map = new Map({
        container: mapContainer,
        style: `https://demotiles.maplibre.org/style.json`,
        center: [initialState.lng, initialState.lat],
        zoom: initialState.zoom
      });

      map.addControl(new NavigationControl(), 'top-right');

      map.on('load', function () {
        map.addSource('municipalities', {
            type: 'vector',
            url: 'https://openicgc.github.io/divisions_administratives.json'
          });
      		// layer associated municipalities
          map.addLayer({
            'id': 'associated-munis',
            'type': 'fill',
            'source': 'municipalities',
            'source-layer': 'boundary',
            "filter": [

      				"all",
              [
      					"==",
      					"escala",
      					"1M"
      				],
      				[
      					"==",
      					"class",
      					"municipi"
      				],

      				ASSOCIATED_MUNICIPALITIES
      			],
            'paint': {
              "fill-opacity": 0.9,
              "fill-color": "green",
              "fill-outline-color": "grey"
            }
        	});

      		// layer not associated municipalities
      		map.addLayer({
            'id': 'no-associated-munis',
            'type': 'fill',
            'source': 'municipalities',
            'source-layer': 'boundary',
            "filter": [
      				"all",
              [
      					"==",
      					"escala",
      					"1M"
      				],
      				NO_ASSOCIATED_MUNICIPALITIES
      			],
            'paint': {
              "fill-opacity": 0.8,
              "fill-color": "red",
              "fill-outline-color": "grey"
            }
        	});

      });

      // change pointer on mouse over states
      map.on('mouseenter', 'associated-munis', function () {
      	map.getCanvas().style.cursor = 'pointer';
      });

      map.on('mouseleave', 'associated-munis', function () {
      	map.getCanvas().style.cursor = '';
      });

    });

    onDestroy(() => {
      map.remove();
    });
  </script>

  <div class="map-wrap">
    <div class="map" id="map" bind:this={mapContainer}></div>
  </div>

  <style>

    .map-wrap {
      position: relative;
      width: 100%;
      height: calc(100vh - 77px); /* calculate height of the screen minus the heading */
    }

    .map {
      position: absolute;
      width: 100%;
      height: 100%;
    }

    .watermark {
      position: absolute;
      left: 10px;
      bottom: 10px;
      z-index: 999;
    }
  </style>


7. A Google Spread Sheet to control the map
===============================================

Sometimes, the person in charge to update a data visualization is not a web developer with knowledge of javascript or any programming language.
In that case, using a graphic interface can be a solution to control the entities to display on the map.

Now we are going to use a Google Spread Sheet to control the visible municipalities on the map.

1. Create a google sheet, with the name ``map_municipalities``.
2. Fill this google sheet with the same content of this excel file: :download:`municipalities_cat.xlsx<municipalities_cat.xlsx>`
3. Make **public** the goolge sheet.
4. The url of this file contains the ``id``. Copy this id as we'll use later.


Add the following code to the **Map** component, just before creating the map. We'll create a function ``loadSheet`` to load this google sheet, which will be invoced when DOM is loaded.
Replace the **sheetId** and **sheetName** with yours.


.. code-block:: javascript

  const sheetId = 'your-sheet-id';
  const base = `https://docs.google.com/spreadsheets/d/${sheetId}/gviz/tq?`;
  const sheetName = 'your-name';
  const query = encodeURIComponent('Select *')
  const url = `${base}&sheet=${sheetName}&tq=${query}`

  const data = []
  document.addEventListener('DOMContentLoaded', loadSheet)

  function loadSheet() {
    	fetch(url)
      	.then(res => res.text())
        .then(rep => {
        	//Remove additional text and extract only JSON:
          const jsonData = JSON.parse(rep.substring(47).slice(0, -2));

          const colz = [];
          //Extract column labels
          jsonData.table.cols.forEach((heading) => {
          	if (heading.label) {
            	let column = heading.label;
              colz.push(column);
            }
          })
          //extract row data:
          jsonData.table.rows.forEach((rowData) => {
          	const row = {};
            colz.forEach((ele, ind) => {
            	row[ele] = (rowData.c[ind] != null) ? rowData.c[ind].v : '';
            })
            data.push(row);
    				if (row.ASSOCIATED == true) ASSOCIATED_MUNICIPALITIES.push(row.NOM_MUNICIPI);
    				if (row.NO_ASSOCIATED == true) NO_ASSOCIATED_MUNICIPALITIES.push(row.NOM_MUNICIPI);

          })
        })
    }


Now, the google sheet controls the displayed municipalities.


8. Show information on a popup
================================

To create a *popup*, we ust import the ``Popup`` class.
Update the **import** at the ``Map.svelte`` component.

.. code-block:: javascript

  import { Map, NavigationControl, Popup } from 'maplibre-gl';


And add the following code after ``map.addLayer``.
This will open a popup when click over an **associated municipality** (not over none associated municipalities), with information about the entity. And also it will append data from IDESCAT.

.. code-block:: javascript

  map.on('click', 'associated-munis', e => {
    const features = map.queryRenderedFeatures(e.point, { layers: ["associated-munis"] });

    // dades de l'spreadsheet amb la info del municipi
    let obj = data.find(o => o.ID_MUNICIPI == features[0].properties.codi);

    const myFunction = function (indicadores) {

    new Popup()
      .setLngLat(e.lngLat)
      .setHTML(`població total` + ' ' + indicadores.i[0].v.split(',')[0] + ' url ' + obj.URL)
      .addTo(map);
		};

    // aquí agafem les dades de l'IDESCAT
    fetch('https://api.idescat.cat/emex/v1/dades.json?id='+features[0].properties.codi+'&tipus=mun&i=f171,f36,f42') // IDESCAT API from selected municipi
      .then(response => response.json())
      .then(
        data => myFunction(data.fitxes.indicadors)
      );
    })


9. Final code
================

Finally, this is the ``Map.svelte`` code:

.. code-block:: javascript

  <script>
    import { onMount, onDestroy } from 'svelte'
    import { Map, NavigationControl, Popup } from 'maplibre-gl';
    import 'maplibre-gl/dist/maplibre-gl.css';



    let map;
    let mapContainer;

    onMount(() => {

      const initialState = { lng: 1.4, lat: 41.6, zoom: 7 };

      const ASSOCIATED_MUNICIPALITIES = [
      	"in",
      	"name",
      	'Abrera', 'Àger', 'Agramunt', 'Agullana', 'Aiguaviva', 'Alàs i Cerc',
      ]

      ASSOCIATED_MUNICIPALITIES.push('Olot', 'Sant Joan les Fonts')

      const NO_ASSOCIATED_MUNICIPALITIES = [
      	"in",
      	"name",
      	'Abrera', 'Àger', 'Agramunt', 'Agullana', 'Aiguaviva', 'Alàs i Cerc',
      ]

      const sheetId = '1ideZJ3bDKUnqa44Zl8jVclDdlYYDHzGr3DgSV-c3z9w';
      const base = `https://docs.google.com/spreadsheets/d/${sheetId}/gviz/tq?`;
      const sheetName = 'map_municipalities';
      const query = encodeURIComponent('Select *')
      const url = `${base}&sheet=${sheetName}&tq=${query}`

      const data = []
      document.addEventListener('DOMContentLoaded', loadSheet)

      function loadSheet() {
      	fetch(url)
        	.then(res => res.text())
          .then(rep => {
          	//Remove additional text and extract only JSON:
            const jsonData = JSON.parse(rep.substring(47).slice(0, -2));

            const colz = [];
            //Extract column labels
            jsonData.table.cols.forEach((heading) => {
            	if (heading.label) {
              	let column = heading.label;
                colz.push(column);
              }
            })
            //extract row data:
            jsonData.table.rows.forEach((rowData) => {
            	const row = {};
              colz.forEach((ele, ind) => {
              	row[ele] = (rowData.c[ind] != null) ? rowData.c[ind].v : '';
              })
              data.push(row);
      				if (row.ASSOCIATED == true) ASSOCIATED_MUNICIPALITIES.push(row.NOM_MUNICIPI);
      				if (row.NO_ASSOCIATED == true) NO_ASSOCIATED_MUNICIPALITIES.push(row.NOM_MUNICIPI);

            })
          })
      }

      map = new Map({
        container: mapContainer,
        style: `https://demotiles.maplibre.org/style.json`,
        center: [initialState.lng, initialState.lat],
        zoom: initialState.zoom
      });

      map.addControl(new NavigationControl(), 'top-right');

      map.on('load', function () {
        map.addSource('municipalities', {
            type: 'vector',
            url: 'https://openicgc.github.io/divisions_administratives.json'
          });
      		// layer associated municipalities
          map.addLayer({
            'id': 'associated-munis',
            'type': 'fill',
            'source': 'municipalities',
            'source-layer': 'boundary',
            "filter": [

      				"all",
              [
      					"==",
      					"escala",
      					"1M"
      				],
      				[
      					"==",
      					"class",
      					"municipi"
      				],

      				ASSOCIATED_MUNICIPALITIES
      			],
            'paint': {
              "fill-opacity": 0.9,
              "fill-color": "green",
              "fill-outline-color": "grey"
            }
        	});

      		// layer not associated municipalities
      		map.addLayer({
            'id': 'no-associated-munis',
            'type': 'fill',
            'source': 'municipalities',
            'source-layer': 'boundary',
            "filter": [
      				"all",
              [
      					"==",
      					"escala",
      					"1M"
      				],
      				NO_ASSOCIATED_MUNICIPALITIES
      			],
            'paint': {
              "fill-opacity": 0.8,
              "fill-color": "red",
              "fill-outline-color": "grey"
            }
        	});

          map.on('click', 'associated-munis', e => {
      			const features = map.queryRenderedFeatures(e.point, { layers: ["associated-munis"] });

      			// dades de l'spreadsheet amb la info del municipi
      			let obj = data.find(o => o.ID_MUNICIPI == features[0].properties.codi);

      			const myFunction = function (indicadores) {

      				new Popup()
      		      .setLngLat(e.lngLat)
      		      .setHTML(`població total` + ' ' + indicadores.i[0].v.split(',')[0] + ' url ' + obj.URL)
      		      .addTo(map);

      			};

      			// aquí agafem les dades de l'IDESCAT
      			fetch('https://api.idescat.cat/emex/v1/dades.json?id='+features[0].properties.codi+'&tipus=mun&i=f171,f36,f42') // IDESCAT API from selected municipi
      			  .then(response => response.json())
      			  .then(
      					data => myFunction(data.fitxes.indicadors)
      			);
      	})

      });

      // change pointer on mouse over states
      map.on('mouseenter', 'associated-munis', function () {
      	map.getCanvas().style.cursor = 'pointer';
      });

      map.on('mouseleave', 'associated-munis', function () {
      	map.getCanvas().style.cursor = '';
      });

    });

    onDestroy(() => {
      map.remove();
    });
  </script>

  <div class="map-wrap">
    <div class="map" id="map" bind:this={mapContainer}></div>
  </div>

  <style>

    .map-wrap {
      position: relative;
      width: 100%;
      height: calc(100vh - 77px); /* calculate height of the screen minus the heading */
    }

    .map {
      position: absolute;
      width: 100%;
      height: 100%;
    }

    .watermark {
      position: absolute;
      left: 10px;
      bottom: 10px;
      z-index: 999;
    }
  </style>
