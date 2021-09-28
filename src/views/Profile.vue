<template>
    <ion-page>
        <header class="grid">
            <div class="col-1">
                <ion-back-button default-href="/" color="primary"></ion-back-button>
            </div>
            <div class="col-2">
            </div>
            <div class="col-3">
                <ion-buttons>
                    <ion-button mode="ios" color="primary" @click="UpdatePassword()">
                        <ion-icon slot="icon-only" :icon="saveOutline"></ion-icon>
                    </ion-button>
                </ion-buttons>
            </div>
        </header>

        <ion-content>
            <div class="info">
                <ion-item lines="none">
                    <ion-label position="floating">Mobile</ion-label>
                    <ion-input type="number" v-model="Mobile" readonly></ion-input>
                </ion-item>
                <ion-item lines="none">
                    <ion-label position="floating">ID Number</ion-label>
                    <ion-input type="text" v-model="RiderCode" readonly></ion-input>
                </ion-item>
                <ion-item lines="none">
                    <ion-label position="floating">Firstname</ion-label>
                    <ion-input type="text" v-model="Firstname" readonly></ion-input>
                </ion-item>
                <ion-item lines="none">
                    <ion-label position="floating">Lastname</ion-label>
                    <ion-input type="text" v-model="Lastname" readonly></ion-input>
                </ion-item>
                <ion-item lines="none">
                    <ion-label position="floating">Password</ion-label>
                    <ion-input :type="EyePass" v-model="Password"></ion-input>
                    <ion-button class="eyebtn" @click.self="ShowHidePass()">
                        <ion-icon slot="icon-only" :icon="EyeIcon" color="primary"></ion-icon>
                    </ion-button>
                </ion-item>
            </div>
        </ion-content>
    </ion-page>
</template>

<script>
import { IonContent, IonPage, IonItem, IonInput, IonBackButton, IonLabel, IonIcon } from "@ionic/vue";
import { eyeOutline, eyeOffOutline, saveOutline } from "ionicons/icons";
import Global from "../components/Global";
import axios from "axios";

export default({
    components: {
        IonPage,
        IonContent,
        IonItem,
        IonLabel,
        IonIcon,
        IonInput,
        IonBackButton,
    },
    setup() {
        return {
            eyeOutline,
            eyeOffOutline,
            saveOutline
        }
    },
    data() {
        return {
            EyeIcon: this.eyeOffOutline,
            EyeStatus: false,
            EyePass: "password",
            Mobile: "",
            Firstname: "",
            Lastname: "",
            Password: "",
            RiderID: "",
            RiderCode: "",
        }
    },
    created() {
        this.RiderID = (JSON.parse(localStorage.rider))[0].id;
    },
    beforeMount() {
        this.LoadUserProfile();
    },
    methods: {
        LoadUserProfile() {
            Global.methods.Loading();
            axios.post(Global.methods.GetURL() + "/rider_profile", {
                riderid: this.RiderID,
            }).then(res => {
                this.Firstname = res.data[0].firstname;
                this.Lastname = res.data[0].lastname;
                this.Mobile = res.data[0].cellphone;
                this.RiderCode = res.data[0].rider_code;
            }).catch(err => {
                console.log(err);
            }).finally(() => {
                setTimeout(() => {
                    Global.methods.Unloading();
                }, 100);
            });
        },
        UpdatePassword() {
            if (this.Password != "") {
                axios.post(Global.methods.GetURL() + "/rider_update_password", {
                    riderid: this.RiderID,
                    password: this.Password,
                }).then(res => {
                    if (res.data == 1) {
                        this.LoadUserProfile();
                    }
                }).catch(err => {
                    console.log(err);
                });
            } else {
                //
            }
        },
        ShowHidePass() {
            if (this.EyeStatus == false) {
                this.EyeStatus = true;
                this.EyeIcon = this.eyeOutline;
                this.EyePass = "text";
            } else {
                this.EyeStatus = false;
                this.EyeIcon = this.eyeOffOutline;
                this.EyePass = "password";
            }
        },
    },
})
</script>

<style lang="scss" scoped>
    .info {
        margin: 15px;
        ion-item {
            border: 1px solid #c8c7cc;
            border-radius: 10px;
            margin: 10px 0;
        }
    }
    .eyebtn {
        position: absolute;
        right: 15px;
        top: 17px;
        padding: 0 !important;
        --background: transparent;
        z-index: 2;
        --padding-start: 0;
        --padding-end: 0;
        --padding-top: 0;
        --padding-bottom: 0;
        --box-shadow: 0;
    }
</style>