<template>
    <ion-page>
        <header class="grid">
            <div class="col-1">
                <ion-menu-button autoHide="false" color="primary"></ion-menu-button>
            </div>
            <div class="col-2">
            </div>
            <div class="col-3">
                <ion-buttons>
                    <ion-button disabled style="opacity: 1;">
                        <ion-icon slot="icon-only" :icon="ellipse" :color="Color"></ion-icon>
                    </ion-button>
                </ion-buttons>
            </div>
        </header>
        <ion-content>
            <ion-card v-if="Status == 4" class="ion-padding" color="hold">
                You have pending case of remitting, Please fulfill the requirements to continue!
            </ion-card>
            <div v-if="Token != ''">
                <ion-card v-if="Status != 4">
                    <ion-grid>
                        <ion-row>
                            <ion-col size="5">
                                <b>Placed date:</b>
                            </ion-col>
                            <ion-col>
                                {{ DateTime }}
                            </ion-col>
                        </ion-row>
                        <ion-row>
                            <ion-col size="5">
                                <b>Ref ID:</b>
                            </ion-col>
                            <ion-col>
                                {{ Token }}
                            </ion-col>
                        </ion-row>
                        <ion-row>
                            <ion-col size="5">
                                <b>Pick up:</b>
                            </ion-col>
                            <ion-col>
                                {{ ShipFrom }}
                            </ion-col>
                        </ion-row>
                        <ion-row>
                            <ion-col size="5">
                                <b>Deliver to:</b>
                            </ion-col>
                            <ion-col>
                                {{ ShipTo }}
                            </ion-col>
                        </ion-row>
                        <ion-row v-if="DeliveryStatus == 'Accepted'">
                            <ion-col size="5">
                                <b>Buyer name:</b>
                            </ion-col>
                            <ion-col>
                                {{ Firstname + " " + Lastname }}
                            </ion-col>
                        </ion-row>
                        <ion-row v-if="DeliveryStatus == 'Accepted'">
                            <ion-col size="5">
                                <b>Buyer mobile:</b>
                            </ion-col>
                            <ion-col>
                                <a href="#" @click.prevent="CallNumber(PhoneNumber)">{{ PhoneNumber }}</a>
                            </ion-col>
                        </ion-row>
                        <ion-row v-if="DeliveryStatus == 'Accepted'">
                            <ion-col size="5">
                                <b>Buyer note:</b>
                            </ion-col>
                            <ion-col>
                                {{ Note }}
                            </ion-col>
                        </ion-row>
                        <ion-row v-if="DeliveryStatus == 'Accepted'">
                            <ion-col size="5">
                                <b>Payment to recieve:</b>
                            </ion-col>
                            <ion-col>
                                ₱ {{ FormatPrice(Price + Fare) }}
                            </ion-col>
                        </ion-row>
                        <ion-row size="12">
                            <ion-col>
                                <ion-button expand="full" @click="ShowMap()">Show Map</ion-button>
                            </ion-col>
                        </ion-row>
                    </ion-grid>
                </ion-card>
            </div>
            <div v-if="Token == ''">
                <div class="empty ion-text-center">
                    <img src="../assets/NoActiveDelivery.png">
                    <div>
                        <h2>No active delivery found!</h2>
                    </div>
                    <div>
                        <small>Go near from any partner restaurant or shops, You don't know they might need you there.</small>
                    </div>
                </div>
            </div>
        </ion-content>
        
        <ion-footer v-if="Token != ''">
            <ion-button mode="ios" expand="full" color="primary" v-if="DeliveryStatus == 'Pending'" @click="AcceptDelivery()">Accept delivery</ion-button>
            <ion-button mode="ios" expand="full" color="primary" v-if="DeliveryStatus == 'Pending'" @click="DeclineDelivery()">Decline delivery</ion-button>
            <ion-button mode="ios" expand="full" color="primary" v-if="DeliveryStatus == 'Accepted'" @click="CompleteDelivery()">Set as delivered</ion-button>
        </ion-footer>
    </ion-page>
</template>

<script>
// import { LocationAccuracy } from "@ionic-native/location-accuracy";
// import { Diagnostic } from "@ionic-native/diagnostic";
import { CallNumber } from '@ionic-native/call-number';
import { IonContent, IonPage, IonMenuButton, IonButton, IonButtons, modalController } from "@ionic/vue";
import { LocalNotifications } from '@ionic-native/local-notifications';
import { ellipse } from "ionicons/icons";
import DirectionMap from "./DirectionMap";
import Global from "../components/Global";
import axios from "axios";

export default({
    components: {
        IonContent, IonPage, IonMenuButton, IonButton, IonButtons
    },
    setup() {
        return {
            ellipse
        }  
    },
    data() {
        return {
            RiderID: "",
            Map: "",
            DeliveryID: "",
            OrderID: "",
            StoreID: "",
            DeliveryStatus: "",
            DirectionsService: "",
            DirectionsDisplay: "",
            DateTime: "",
            Token: "",
            ShipFrom: "",
            ShipTo: "",
            Firstname: "",
            Lastname: "",
            PhoneNumber: "",
            Note: "",
            Price: "",
            Fare: "",
            Color: "",
            StoreLat: "",
            StoreLng: "",
            CustomerLat: "",
            CustomerLng: "",
            Status: (JSON.parse(localStorage.rider))[0].status_id,
        }
    },
    created() {
        this.RiderID = (JSON.parse(localStorage.rider))[0].id;
        if (this.Status == 2) {
            this.Color = "online";
        } else if (this.Status == 3) {
            this.Color = "busy";
        } else {
            this.Color = "hold";
        }
    },
    mounted() {
        setTimeout(() => {
            // Diagnostic.isLocationEnabled().then((isEnabled) => {
            //     if(!isEnabled){
            //         LocationAccuracy.request(LocationAccuracy.REQUEST_PRIORITY_HIGH_ACCURACY).then(() => {
            //             this.LoadHome();
            //         });
            //     } else {
                    this.LoadHome();
            //     }
            // });
        }, 500);
    },
    methods: {
        LoadHome() {
            Global.methods.Loading();
            axios.post(Global.methods.GetURL() + "/rider_active_delivery", {
                riderid: this.RiderID,
            }).then(res => {
                localStorage.riderdelivery = res.data.delivery.length;
                if (res.data.delivery.length != 0) {
                    this.Price = 0;
                    this.DeliveryID = res.data.delivery[0].delivery_id;
                    this.DeliveryStatus = res.data.delivery[0].rider_order_status;
                    this.OrderID = res.data.delivery[0].order_id;
                    this.StoreID = res.data.delivery[0].store_id;
                    this.DateTime = res.data.delivery[0].datetime;
                    this.Token = res.data.delivery[0].token;
                    this.ShipFrom = res.data.delivery[0].ship_from;
                    this.ShipTo = res.data.delivery[0].ship_to;
                    this.Firstname = res.data.delivery[0].firstname;
                    this.Lastname = res.data.delivery[0].lastname;
                    this.PhoneNumber = res.data.delivery[0].phone_number;
                    this.Note = res.data.delivery[0].note;
                    this.Fare = res.data.delivery[0].fare;
                    this.StoreLat = res.data.delivery[0].storelat;
                    this.StoreLng = res.data.delivery[0].storelng;
                    this.CustomerLat = res.data.delivery[0].orderlat;
                    this.CustomerLng = res.data.delivery[0].orderlng;
                    res.data.delivery.forEach(x => {
                        let subtotal = 0;
                        if (x.selected_choices != "") {
                            JSON.parse(x.selected_choices).forEach(y => {
                                y.choices.forEach(z => {
                                    if (z.checked == "true") {
                                        subtotal += Number(z.price);
                                    }
                                });
                            });
                        }
                        this.Price += ((x.price + subtotal) * x.quantity);
                    });
                    localStorage.delivery = 1;
                } else {
                    this.Price = 0;
                    this.DeliveryID = "";
                    this.DeliveryStatus = "";
                    this.OrderID = "";
                    this.StoreID = "";
                    this.DateTime = "";
                    this.Token = "";
                    this.ShipFrom = "";
                    this.ShipTo = "";
                    this.Firstname = "";
                    this.Lastname = "";
                    this.PhoneNumber = "";
                    this.Note = "";
                    this.Fare = "";
                    this.StoreLat = "";
                    this.StoreLng = "";
                    this.CustomerLat = "";
                    this.CustomerLng = "";
                    localStorage.removeItem("delivery");
                }
                this.LoopReload();
            }).catch(err => {
                console.log(err);
            }).finally(() => {
                setTimeout(() => {
                    Global.methods.Unloading();
                }, 250);
            });
        },
        LoopReload() {
            let interval = true;
            setInterval(() => {
                if (interval == true) {
                    interval = false;
                    axios.post(Global.methods.GetURL() + "/rider_active_delivery", {
                        riderid: this.RiderID,
                    }).then(res => {
                        interval = true;
                        localStorage.rider = JSON.stringify(res.data.rider);
                        if (res.data.delivery.length != 0) {
                            if (res.data.delivery.length != localStorage.riderdelivery) {
                                LocalNotifications.schedule({
                                    id: 1,
                                    title: "You have new delivery",
                                    text: "Check it out now...",
                                    channel: "IncomingDelivery",
                                });
                            }
                            this.Price = 0;
                            this.DeliveryID = res.data.delivery[0].delivery_id;
                            this.DeliveryStatus = res.data.delivery[0].rider_order_status;
                            this.OrderID = res.data.delivery[0].order_id;
                            this.StoreID = res.data.delivery[0].store_id;
                            this.DateTime = res.data.delivery[0].datetime;
                            this.Token = res.data.delivery[0].token;
                            this.ShipFrom = res.data.delivery[0].ship_from;
                            this.ShipTo = res.data.delivery[0].ship_to;
                            this.Firstname = res.data.delivery[0].firstname;
                            this.Lastname = res.data.delivery[0].lastname;
                            this.PhoneNumber = res.data.delivery[0].phone_number;
                            this.Note = res.data.delivery[0].note;
                            this.Fare = res.data.delivery[0].fare;
                            this.StoreLat = res.data.delivery[0].storelat;
                            this.StoreLng = res.data.delivery[0].storelng;
                            this.CustomerLat = res.data.delivery[0].orderlat;
                            this.CustomerLng = res.data.delivery[0].orderlng;
                            res.data.delivery.forEach(x => {
                                let subtotal = 0;
                                if (x.selected_choices != "") {
                                    JSON.parse(x.selected_choices).forEach(y => {
                                        y.choices.forEach(z => {
                                            if (z.checked == "true") {
                                                subtotal += Number(z.price);
                                            }
                                        });
                                    });
                                }
                                this.Price += ((x.price + subtotal) * x.quantity);
                            });
                            localStorage.delivery = 1;
                        } else {
                            this.Price = 0;
                            this.DeliveryID = "";
                            this.DeliveryStatus = "";
                            this.OrderID = "";
                            this.StoreID = "";
                            this.DateTime = "";
                            this.Token = "";
                            this.ShipFrom = "";
                            this.ShipTo = "";
                            this.Firstname = "";
                            this.Lastname = "";
                            this.PhoneNumber = "";
                            this.Note = "";
                            this.Fare = "";
                            this.StoreLat = "";
                            this.StoreLng = "";
                            this.CustomerLat = "";
                            this.CustomerLng = "";
                            localStorage.removeItem("delivery");
                        }
                        localStorage.riderdelivery = res.data.delivery.length;
                    }).catch(err => {
                        console.log(err);
                    });
                }
            }, 1000);
        },
        AcceptDelivery() {
            axios.post(Global.methods.GetURL() + "/rider_accept_delivery", {
                riderid: this.RiderID,
                deliveryid: this.DeliveryID
            }).catch(err => {
                console.log(err);
            });
        },
        DeclineDelivery() {
            axios.post(Global.methods.GetURL() + "/rider_decline_delivery", {
                deliveryid: this.DeliveryID,
                storeid: this.StoreID,
                orderid: this.OrderID,
                riderid: this.RiderID
            }).then(res => {
                if (res.data == 1) {
                    setTimeout(() => {
                        localStorage.riderdelivery = localStorage.riderdelivery - 1;
                    }, 100);
                }
            }).catch(err => {
                console.log(err);
            });
        },
        CompleteDelivery() {
            axios.post(Global.methods.GetURL() + "/rider_complete_delivery", {
                deliveryid: this.DeliveryID,
                orderid: this.OrderID,
                riderid: this.RiderID
            }).then(res => {
                if (res.data == 1) {
                    setTimeout(() => {
                        localStorage.riderdelivery = localStorage.riderdelivery - 1;
                    }, 100);
                }
            }).catch(err => {
                console.log(err);
            });
        },
        CallNumber(number) {
            CallNumber.callNumber(number, true).then(() => {
            });
        },
        FormatPrice(value) {
            const val = (value/1).toFixed(2).replace(",", ".")
            return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",")
        },
        async ShowMap() {
            const modal = await modalController.create({
                component: DirectionMap,
                cssClass: "my-custom-class",
                componentProps: {
                    storelat: this.StoreLat,
                    storelng: this.StoreLng,
                    customerlat: this.CustomerLat,
                    customerlng: this.CustomerLng,
                }
            });
            await modal.present();
        },
    },
});
</script>

<style lang="scss" scoped>
    ion-footer {
        ion-button {
            margin: 5px;
        }
    }
    ion-grid {
        ion-row:last-child {
            ion-col:nth-child(1) {
                display: flex;
                align-items: center;
                justify-content: center;
                ion-button {
                    margin: 10px 0;
                }
            }
        }
        ion-row {
            ion-col:nth-child(1) {
                display: flex;
                align-items: center;
            }
        }
    }
    .empty {
        width: 75%;
        position: absolute;
        top: 45%;
        left: 50%;
        transform: translate(-50%, -50%);
        img {
            height: 250px;
        }
    }
</style>