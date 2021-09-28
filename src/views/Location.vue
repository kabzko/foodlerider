<template>
    <ion-page>
        <header class="grid">
            <div class="col-1">
                <ion-back-button default-href="/" color="primary"></ion-back-button>
            </div>
            <div class="col-2">
                <ion-title>
                    <span>{{ Route + " " + Street + " " + City + " " + Province + " " + Country }}</span>
                </ion-title>
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
import { IonContent, IonPage, IonBackButton } from "@ionic/vue";
import { useRoute } from "vue-router";

export default({
    components: {
        IonContent,
        IonPage,
        IonBackButton,
    },
    setup() {
        const route = useRoute();
        const lat = route.params.lat;
        const lng = route.params.lng;
        return {
            lat,
            lng,
        }
    },
    data() {
        return {
            Map: "",
            Route: "",
            Street: "",
            City: "",
            Province: "",
            Country: "",
        }
    },
    mounted() {
        this.Map = new window.google.maps.Map(this.$refs.MapDivRef, {
            zoom: 17,
            clickableIcons: false,
            disableDefaultUI: true,
            disableDoubleClickZoom: true,
            mapTypeControl: false,
            scaleControl: false,
            center: { lat: parseFloat(this.lat), lng: parseFloat(this.lng) }
        });
        new window.google.maps.Marker({
            position: { lat: parseFloat(this.lat), lng: parseFloat(this.lng) },
            map: this.Map,
            icon: {
                url: "../assets/icon/IAMMarker.png",
                scaledSize: new window.google.maps.Size(30, 40),
            }
        });
        new window.google.maps.Geocoder().geocode({
            "latLng": { lat: parseFloat(this.lat), lng: parseFloat(this.lng) }
        }, res => {
            if (res[1].address_components.length == 6) {
                this.Route = res[1].address_components[0].short_name + " " + res[1].address_components[1].short_name;
                this.Street = res[1].address_components[2].short_name;
                this.City = res[1].address_components[3].short_name;
                this.Province = res[1].address_components[4].short_name;
                this.Country = res[1].address_components[5].short_name;
            } else if (res[1].address_components.length == 5) {
                this.Route = res[1].address_components[0].short_name;
                this.Street = res[1].address_components[1].short_name;
                this.City = res[1].address_components[2].short_name;
                this.Province = res[1].address_components[3].short_name;
                this.Country = res[1].address_components[4].short_name;
            } else if (res[1].address_components.length == 4) {
                this.Street = res[1].address_components[0].short_name;
                this.City = res[1].address_components[1].short_name;
                this.Province = res[1].address_components[2].short_name;
                this.Country = res[1].address_components[3].short_name;
            } else if (res[1].address_components.length == 3) {
                this.City = res[1].address_components[0].short_name;
                this.Province = res[1].address_components[1].short_name;
                this.Country = res[1].address_components[2].short_name;
            } else if (res[1].address_components.length == 2) {
                this.Province = res[1].address_components[0].short_name;
                this.Country = res[1].address_components[1].short_name;
            } else {
                this.Country = res[1].address_components[0].short_name;
            }
        }).catch(err => {
            console.log(err);
        });
    },
    unmounted() {
        this.Map = "";
    },
})
</script>

<style lang="scss" scoped>
    ion-content {
        top: -6px !important;
    }
    ion-content::part(background) {
        height: calc(100% + 6px) !important;   
    }
    ion-content::part(scroll) {
        height: calc(100% + 6px) !important;   
    }
    .map-size {
        width: 100%;
        height: 100%;
        background: azure;
    }
</style>