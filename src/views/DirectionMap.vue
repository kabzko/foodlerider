<template>
    <ion-page>
        <header class="grid">
            <div class="col-1">
                <ion-buttons>
                    <ion-button @click="CloseModal()">
                        <ion-icon slot="icon-only" :icon="closeOutline" color="primary"></ion-icon>
                    </ion-button>
                </ion-buttons>
            </div>
            <div class="col-2">
            </div>
            <div class="col-3">
            </div>
        </header>
        <ion-content>
            <div class="map-size" ref="MapDivRef"></div>
        </ion-content>
    </ion-page>
</template>

<script>
import { IonContent, IonPage, IonButtons, IonIcon, IonButton, modalController } from "@ionic/vue";
import { closeOutline } from "ionicons/icons";

export default({
    components: {
        IonContent,
        IonPage,
        IonButton,
        IonButtons,
        IonIcon,
    },
    props: {
        storelat: Number,
        storelng: Number,
        customerlat: Number,
        customerlng: Number,
    },
    setup() {
        return {
            closeOutline,
        }
    },
    data() {
        return {
            Map: "",
        }
    },
    mounted() {
        this.Map = new window.google.maps.Map(this.$refs.MapDivRef, {
            zoom: 16,
            clickableIcons: false,
            disableDefaultUI: true,
            disableDoubleClickZoom: true,
            mapTypeControl: false,
            scaleControl: false,
            center: { lat: parseFloat(this.storelat), lng: parseFloat(this.storelng) }
        });
        this.DirectionMap(this.storelat, this.storelng, this.customerlat, this.customerlng);
    },
    unmounted() {
        this.Map = "";
    },
    methods: {
        DirectionMap(storelat, storelng, customerlat, customerlng) {
            this.DirectionsService = new window.google.maps.DirectionsService;
            this.DirectionsDisplay = new window.google.maps.DirectionsRenderer;
            this.DirectionsDisplay.setMap(this.Map);
            this.DirectionsService.route({
                origin: storelat + ", " + storelng,
                destination: customerlat + ", " + customerlng,
                travelMode: 'DRIVING'
            }, (response, status) => {
                if (status === 'OK') {
                    this.$refs.MapDivRef.style.height = "100%";
                    const route = response.routes[0].legs[0];
                    this.YouMarker(route.start_location);
                    this.createEndMarker(route.end_location);
                    this.DirectionsDisplay.setDirections(response);
                    this.DirectionsDisplay.setOptions({
                        preserveViewport: true,
                        suppressMarkers: true,
                        polylineOptions: {
                            geodesic: true,
                            strokeColor: '#009e4d',
                            strokeOpacity: 0.9,
                            strokeWeight: 5,
                        }
                    });
                    let infowindow = new window.google.maps.InfoWindow();
                    let arr = [];
                    infowindow.setContent(response.routes[0].legs[0].distance.text + "<br>" + response.routes[0].legs[0].duration.text);
                    response.routes[0].legs[0].steps.forEach((x, index) => {
                        arr.push({
                            steps: index,
                            latlng: x.lat_lngs.length,
                            distance: x.distance.value,
                        });
                    });
                    let steps = arr.find(function(x) { return x.distance == Math.max.apply(Math, arr.map(function(z) { return z.distance; }))}).steps;
                    let latlng = arr.find(function(x) { return x.distance == Math.max.apply(Math, arr.map(function(z) { return z.distance; }))}).latlng; 
                    infowindow.setPosition(response.routes[0].legs[0].steps[steps].lat_lngs[Math.round(latlng / 2)]);
                    infowindow.open(this.Map);
                    var bounds = new window.google.maps.LatLngBounds();
                    bounds.extend(new window.google.maps.LatLng(storelat, storelng));
                    bounds.extend(new window.google.maps.LatLng(customerlat, customerlng));
                    this.Map.fitBounds(bounds);
                }
            });
        },
        YouMarker(yLocation) {
            new window.google.maps.Marker({
                map: this.Map,
                position: yLocation,
                icon: {
                    url: "../assets/icon/StartMarker.png",
                    scaledSize: new window.google.maps.Size(30, 40),
                },
            });
            this.Map.setCenter(yLocation);
        },
        createEndMarker(mPosition) {
            new window.google.maps.Marker({
                map: this.Map,
                position: mPosition,
                icon: {
                    url: "../assets/icon/EndMarker.png",
                    scaledSize: new window.google.maps.Size(30, 40),
                },
            });
        },
        CloseModal() {
            modalController.dismiss();
        },
    },
})
</script>

<style lang="scss" scoped>
    .map-size {
        width: 100%;
        background: azure;
    }
    ion-content {
        top: -6px !important;
    }
    ion-content::part(background) {
        height: calc(100% + 6px) !important;   
    }
    ion-content::part(scroll) {
        height: calc(100% + 6px) !important;   
    }
</style>