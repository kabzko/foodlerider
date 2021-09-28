<template>
    <ion-app>
        <ion-menu content-id="main" type="overlay" swipeGesture="false">
            <ion-content>
                <ion-list>
                    <ion-list-header>{{ Firstname + " " + Lastname }}</ion-list-header>
                    <ion-menu-toggle auto-hide="false">
                        <ion-item button lines="none" router-link="/profile">Profile</ion-item>
                    </ion-menu-toggle>
                    <ion-menu-toggle auto-hide="false">
                        <ion-item button lines="none" router-link="/orders">Orders</ion-item>
                    </ion-menu-toggle>
                    <ion-menu-toggle auto-hide="false" @click="() => $router.push({name: 'Location', params: {lat: Lat, lng: Lng }})">
                        <ion-item button lines="none">My Location</ion-item>
                    </ion-menu-toggle>
                    <ion-menu-toggle auto-hide="false">
                        <ion-item button lines="none" router-link="/helpcenter">Help Center</ion-item>
                    </ion-menu-toggle>
                    <ion-menu-toggle auto-hide="false" @click="Logout()">
                        <ion-item button lines="none">Log out</ion-item>
                    </ion-menu-toggle>
                </ion-list>
            </ion-content>
        </ion-menu>
        <ion-router-outlet id="main"></ion-router-outlet>
    </ion-app>
</template>

<script>
import { IonApp, IonRouterOutlet, toastController, IonList, IonListHeader, IonItem, IonContent, IonMenu, IonMenuToggle } from "@ionic/vue";
import { Geolocation } from "@ionic-native/geolocation";
// import { Plugins } from '@capacitor/core';
import Global from "./components/Global";
import axios from "axios";

// const { LocalNotifications } = Plugins;

export default({
    components: {
        IonApp,
        IonRouterOutlet,
        IonList, 
        IonItem, 
        IonContent, 
        IonMenu,
        IonMenuToggle,
        IonListHeader,
    },
    created() {
        // const channel = {
        //     id: 'IncomingDelivery',
        //     name: 'Incoming Delivery',
        //     importance: 5,
        //     visibility: 1,
        // };
        // LocalNotifications.createChannel(channel);
        if (typeof localStorage.rider !== "undefined") {
            this.RiderID = (JSON.parse(localStorage.rider))[0].id;
            this.Firstname = (JSON.parse(localStorage.rider))[0].firstname;
            this.Lastname = (JSON.parse(localStorage.rider))[0].lastname;
            Geolocation.getCurrentPosition().then(res => {
                this.Lat = res.coords.latitude;
                this.Lng = res.coords.longitude;
            });
            Geolocation.watchPosition().subscribe(data => {
                this.Lat = data.coords.latitude;
                this.Lng = data.coords.longitude;
                this.UpdateRiderLocation(data.coords.latitude, data.coords.longitude);
            });
        }
    },
    data() {
        return {
            RiderID: "",
            Firstname: "",
            Lastname: "",
            Lat: "",
            Lng: "",
        }
    },
    methods: {
        Logout() {
            if (localStorage.delivery != 1) {
                Global.methods.Loading();
                axios.post(Global.methods.GetURL() + "/rider_logout", {
                    riderid: this.RiderID,
                }).catch(err => {
                    console.log(err);
                }).finally(() => {
                    Global.methods.Unloading();
                    location.replace("/login");
                    localStorage.clear();
                });
            } else {
                this.openToast();
            }
        },
        UpdateRiderLocation(lat, lng) {
            axios.post(Global.methods.GetURL() + "/rider_update_location", {
                riderid: this.RiderID,
                lat: lat,
                lng: lng,
            }).catch(err => {
                console.log(err);
            });
        },
        async openToast() {
            const toast = await toastController.create({
                    message: "You still have active delivery, Please complete it before signing out.",
                    mode: "ios",
                    duration: 3000
                });
            return toast.present();
        },
    },
});
</script>

<style scoped>
    ion-list {
        padding: 20px 10px;
    }
    ion-list-header {
        font-size: 18px;
        font-weight: 600;
        color: var(--ion-color-primary);
    }
</style>