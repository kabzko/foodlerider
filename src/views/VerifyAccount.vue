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
                    <ion-button @click="VerifyCode()" mode="ios" color="primary">
                        Confirm
                    </ion-button>
                </ion-buttons>
            </div>
        </header>

        <ion-content>
            <div class="container">
                <div class="ion-text-center">
                    <p>Your verification code is sent by SMS to</p>
                    <p>{{ Simnumber }}</p>
                </div>
                <div>
                    <ion-item lines="none" class="ion-text-center">
                        <mask-input refid="verifycode" mask="cpf" input="numeric" ph="* * * * * *"></mask-input>
                    </ion-item>
                </div>
                <div class="ion-text-center">
                    <p>Did not receive the code? <a href="#">Resend</a></p>
                </div>
            </div>
        </ion-content>
    </ion-page>
</template>

<script>
import { IonContent, IonPage, IonButtons, IonButton, IonBackButton, IonItem } from "@ionic/vue";
import { useRoute } from "vue-router";
import MaskInput from "../components/MaskInput.vue";
import Global from "../components/Global";
import axios from "axios";
import { Plugins } from "@capacitor/core";

const { Storage } = Plugins;

export default({
    name: "VerifyAccount",
    components: {
        IonContent,
        IonPage,
        IonButton,
        IonButtons,
        IonBackButton,
        IonItem, 
        MaskInput
    },
    setup() {
        const route = useRoute();
        const mobile = route.params.mobile;
        const firstname = route.params.firstname;
        const lastname = route.params.lastname;
        const password = route.params.password;
        return {
            mobile,
            firstname,
            lastname,
            password
        }
    },
    data() {
        return {
            GeneratedCode: 123456,
            Simnumber: this.mobile,
            UserID: "",
        }
    },
    async created() {
        const Session = await Storage.get({key: "users"});
        if (Session.value != null) {
            this.UserID = (JSON.parse(Session.value))[0].id;
            this.Name = (JSON.parse(Session.value))[0].firstname + (JSON.parse(Session.value))[0].lastname;
        }
        this.SendCode();
    },
    methods: {
        SendCode() {
            Global.methods.Loading();
            axios.post(Global.methods.GetURL() + "/send_code", {
                mobile: ((this.Simnumber.replace(/[()]/g, '')).replace(/\s/g, '')).replace('+', '')
            }).catch(err => {
                console.log(err);
            }).finally(() => {
                Global.methods.Unloading();
            });
        },
        VerifyCode() {
            if (MaskInput.methods.getData("verifycode").replace(/\s/g, "") == this.GeneratedCode) {
                Global.methods.Loading();
                axios.post(Global.methods.GetURL() + "/user_update", {
                    userid: this.UserID,
                    firstname: this.firstname,
                    lastname: this.lastname,
                    password: this.password
                }).catch(err => {
                    console.log(err);
                }).finally(() => {
                    this.$router.back();
                    Global.methods.Unloading();
                });
            }
        }
    }
})
</script>

<style scoped>
    .container {
        margin: 15px 0;
    }
    #verifycode {
        font-size: 30px;
    }
</style>