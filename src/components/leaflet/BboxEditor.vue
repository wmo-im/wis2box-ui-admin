<template>
    <v-lazy>
        <v-img>
        <l-map
            ref="map"
            :zoom="1.5" 
            :center.sync="rectangle.center" 
            @ready="loadMapObjects()"
            style="height: 300px; width: 100%" 
        >
            <l-control-layers
                position="bottomleft"
                :collapsed="true"
                :sort-layers="true"
            />
            <l-tile-layer
                v-for="tileProvider in tileProviders"
                :key="tileProvider.name"
                :name="tileProvider.name"
                :visible="tileProvider.visible"
                :url="tileProvider.url"
                :attribution="tileProvider.attribution"
                layer-type="base"
            />
            <l-feature-group ref="features">
                <l-rectangle 
                    v-if="rectangle.visible" 
                    :autoPan="true"
                    :autofocus="true"
                    :bounds.sync="rectangle.bounds"
                    :draggable="true"
                ></l-rectangle>
            </l-feature-group>
        </l-map>
        </v-img>
    </v-lazy>
</template>

<script>
import { LMap, LTileLayer, LControlLayers, LRectangle, LFeatureGroup } from "vue2-leaflet"
import { LatLng, LatLngBounds } from "leaflet"
//todo import { LDraw } from 'leaflet-draw'
export default {
    name: "BboxEditor",
    components: {
        LMap,
        LTileLayer,
        LRectangle,
        LControlLayers,
        LFeatureGroup
    },
    props: {
        inputFeature: {},
    },
    data() {
        return {
            tileProviders: this.loadBasemaps(),
            rectangle: {
                visible: false,
                center: [0, 0],
                bounds: [[0, 0], [0, 0]]
            }
        }
    },
    watch: {
        bounds: {
            handler(input) {
                this.$emit('updated', [input.getNorth(), input.getEast(), input.getSouth(), input.getWest()])
            },
            deep: true
        },
        inputFeature: {
            handler(input) {
                if (input.length === 4) {
                    const zoom_factor = 0.05
                    var topleft = new LatLng(input[0], input[1])
                    var bottomright = new LatLng(input[2], input[3])
                    var tmp = new LatLngBounds(topleft, bottomright)
                    this.rectangle.center = tmp.getCenter()
                    this.rectangle.bounds = [tmp.getNorthWest(), tmp.getSouthEast()]
                    this.$refs.map.mapObject.fitBounds([
                        [input[0] + zoom_factor, input[1] + zoom_factor],
                        [input[2] - zoom_factor, input[3] - zoom_factor]
                    ])
                    this.rectangle.visible = true
                }
                else {
                    this.$refs.map.mapObject.setZoom(1.5)
                    this.rectangle.center = [0, 0]
                    this.rectangle.bounds = [[0, 0], [0, 0]]
                    this.rectangle.visible = false
                }
            },
            deep: true
        }
    },
    methods: {
        loadMapObjects() {
            this.$refs.map.mapObject.on("draw:created", (event) => {
                console.log(event)
            })
            this.$emit('loaded')
        },

        loadBasemaps() {
            return [
                {
                name: 'OpenStreetMap',
                visible: true,
                attribution:
                    '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
                url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
                },
                {
                name: 'OpenTopoMap',
                visible: false,
                url: 'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png',
                attribution:
                    'Map data: &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, <a href="http://viewfinderpanoramas.org">SRTM</a> | Map style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (<a href="https://creativecommons.org/licenses/by-sa/3.0/">CC-BY-SA</a>)',
                },
                {
                name: 'ESRI World Imagery',
                visible: false,
                url: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
                attribution: 'Tiles &copy; Esri &mdash; Source: Esri, i-cubed, USDA, USGS, AEX, GeoEye, Getmapping, Aerogrid, IGN, IGP, UPR-EGP, and the GIS User Community'
                }
            ]
        },
    },
}
</script>

<style>
@import "~leaflet/dist/leaflet.css";
</style>
