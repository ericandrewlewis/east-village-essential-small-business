<script>
    import {onMount} from 'svelte'
    import {mapView, data, details, filters} from '../stores'

    let container
    let map
    let layer

    function generateLayer(data, filter = () => true) {
        return L.geoJSON(data, {
            pointToLayer: feature => feature.properties.icon,
            onEachFeature: (feature, layer) => {
                const {merchantname, overallcategory} = feature.properties
                layer.bindPopup(`<p><strong>${merchantname}</strong></p><p>${overallcategory}</p>`)
                layer.on('click', () => details.set(feature.properties))
            },
            filter
        })
    }

    onMount(() => {
        const maxZoom = 20
        const minZoom = 14
        const bounds = L.latLngBounds([40.708, -74.02], [40.743, -73.961]);
        const attribution = '<a href="https://carto.com/">Carto</a> | <a href="https://www.openstreetmap.org/copyright">OSM</a> |  <a href="https://mapicons.mapsmarker.com/">Map Icons Collection</a>'


        map = L.map(container, {maxZoom, minZoom}).setView([40.7268, -73.9835], 15)
        // map.scrollWheelZoom.disable()

        L.tileLayer(
                'https://cartodb-basemaps-{s}.global.ssl.fastly.net/rastertiles/light_all/{z}/{x}/{y}{r}.png',
                {maxZoom, minZoom, bounds, attribution}
        ).addTo(map)

        //add map to store
        mapView.set(map)
    })

    $: if (map && $data) {
        layer = generateLayer($data).addTo(map);
    }

    $: {
        if (map && layer) {
            //update layer when filter functions changes
            map.removeLayer(layer);
            layer = generateLayer($data, feature => $filters.every(filter => filter(feature))).addTo(map);
            layer.addTo(map);
        }
    }


</script>

<div id="map" bind:this="{container}"></div>

<style>
    #map {
        width: 100%;
        height: 100%;
    }

    :global(.awesome-marker i) {
        font-size: 4px;
    }
</style>