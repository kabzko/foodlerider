<template>
    <ion-page>
        <header class="grid">
            <div class="col-1">
                <ion-back-button default-href="/" color="primary"></ion-back-button>
            </div>
            <div class="col-2">
                <ion-title>Orders</ion-title>
            </div>
            <div class="col-3">
            </div>
        </header>
        
        <ion-content>
            <ion-segment v-model="Segment">
                <ion-segment-button value="Complete">
                    <ion-label>Complete</ion-label>
                </ion-segment-button>
                <ion-segment-button value="Decline">
                    <ion-label>Decline</ion-label>
                </ion-segment-button>
            </ion-segment>
            <div v-if="Segment == 'Complete'">
                <ion-list v-if="CompleteOrders?.length != 0">
                    <ion-card v-for="order of CompleteOrders" v-bind:key="order">
                        <ion-grid>
                            <ion-row>
                                <ion-col class="ion-text-right">
                                    {{ order.created_at }}
                                </ion-col>
                            </ion-row>
                            <ion-row>
                                <ion-col>
                                    Order ID: {{ order.token }}
                                </ion-col>
                            </ion-row>
                            <ion-row>
                                <ion-col>
                                    ₱ {{ FormatPrice(order.price + order.fare) }}
                                </ion-col>
                                <ion-col class="ion-text-right">
                                    {{ order.name }}
                                </ion-col>
                            </ion-row>
                        </ion-grid>
                    </ion-card>
                </ion-list>
            </div>
            <div v-if="Segment == 'Decline'">
                <ion-list v-if="DeclineOrders?.length != 0">
                    <ion-card v-for="order of DeclineOrders" v-bind:key="order">
                        <ion-grid>
                            <ion-row>
                                <ion-col class="ion-text-right">
                                    {{ order.created_at }}
                                </ion-col>
                            </ion-row>
                            <ion-row>
                                <ion-col>
                                    Order ID: {{ order.token }}
                                </ion-col>
                            </ion-row>
                            <ion-row>
                                <ion-col>
                                    ₱ {{ FormatPrice(order.price + order.fare) }}
                                </ion-col>
                                <ion-col>
                                    {{ order.name }}
                                </ion-col>
                            </ion-row>
                        </ion-grid>
                    </ion-card>
                </ion-list>
            </div>
        </ion-content>
    </ion-page>
</template>

<script>
import { IonContent, IonPage, IonTitle, IonBackButton, IonList, IonSegment, IonSegmentButton } from "@ionic/vue";
import Global from "../components/Global";
import axios from "axios";

export default({
    components: {
        IonContent, 
        IonPage, 
        IonTitle, 
        IonBackButton, 
        IonList,
        IonSegment, 
        IonSegmentButton
    },
    created() {
        this.RiderID = (JSON.parse(localStorage.rider))[0].id;
    },
    beforeMount() {
        this.LoadOrders();
    },
    data() {
        return {
            RiderID: "",
            CompleteOrders: [],
            DeclineOrders: [],
            Segment: "Complete"
        }
    },
    methods: {
        FormatPrice(value) {
            const val = (value/1).toFixed(2).replace(",", ".")
            return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",")
        },
        LoadOrders() {
            Global.methods.Loading();
            axios.post(Global.methods.GetURL() + "/rider_complete_orders", {
                riderid: this.RiderID
            }).then(res => {
                res.data.complete.forEach(x => {
                    let total = 0;
                    if (x.selected_choices != "") {
                        JSON.parse(x.selected_choices).forEach(y => {
                            y.choices.forEach(z => {
                                if (z.checked == "true") {
                                    total += Number(z.price);
                                }
                            });
                        });
                    }
                    if (this.CompleteOrders.filter(s => s.id == x.id).length == 0) {
                        this.CompleteOrders.push({
                            id: x.id,
                            name: x.name,
                            token: x.token,
                            price: (x.price + total) * x.quantity,
                            created_at: x.datetime,
                            fare: x.fare,
                        });
                    } else {
                        this.CompleteOrders.filter(s => s.id == x.id)[0].price = this.CompleteOrders.filter(s => s.id == x.id)[0].price + ((x.price + total) * x.quantity);
                    }
                });
                res.data.decline.forEach(x => {
                    let total = 0;
                    JSON.parse(x.selected_choices).forEach(y => {
                        y.choices.forEach(z => {
                            if (z.checked == "true") {
                                total += Number(z.price);
                            }
                        });
                    });
                    if (this.DeclineOrders.filter(s => s.id == x.id).length == 0) {
                        this.DeclineOrders.push({
                            id: x.id,
                            name: x.name,
                            token: x.token,
                            price: (x.price + total) * x.quantity,
                            created_at: x.datetime,
                            fare: x.fare,
                        });
                    } else {
                        this.DeclineOrders.filter(s => s.id == x.id)[0].price = this.DeclineOrders.filter(s => s.id == x.id)[0].price + ((x.price + total) * x.quantity);
                    }
                });
            }).catch(err => {
                console.log(err);
            }).finally(() => {
                setTimeout(() => {
                    Global.methods.Unloading();
                }, 100);
            });
        }
    },
})
</script>

<style lang="scss" scoped>
    ion-card {
        margin: 10px;
        img {
            display: block;
        }
    }
    ion-grid {
        ion-row:nth-child(2) ion-col {
            font-size: 16px;
            font-weight: 600;
            color: black;
        }
        ion-row:nth-child(3) {
            ion-col:nth-child(1) {
                font-size: 16px;
                font-weight: 600;
                color: red;
            }
            ion-col:nth-child(2) {
                font-size: 16px;
                font-weight: 600;
                color: var(--ion-color-primary);
            }
        }
        ion-row:nth-child(2) ion-col:nth-child(1) {
            display: flex;
            align-items: center;
        }
        ion-row:nth-child(3) ion-col:nth-child(2) {
            display: flex;
            align-items: center;
            justify-content: flex-end !important;
        }
    }
    .empty {
        width: 75%;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        img {
            height: 100px;
        }
    }
</style>