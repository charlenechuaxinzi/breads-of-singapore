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
