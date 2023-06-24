<template>
    
    <div>
        <div>
            <section id="forms"  v-show="report">
                <button id="return_map" @click="dismissReport">
                    
                </button>
                <div id="forms_report">
                        <ReportComponent/>
                </div>
            </section>
            <section class="ui three column left grid" style="position:relative; z-index: 2; margin: 10px;">
                <div class="column">
                    <form class="ui segment large form">
                        <div class="ui message light red" v-show="error">{{error}}></div>
                        <div class="ui segment">
                            <p>Visualização de casos de incêndio</p>
                            <div class="field">
                                <div class="ui right icon input large" :class="{loading:spinner}">
                                    <input type="text" ref="autocompleteInput" placeholder="Seu endereço" id="endereco" v-model="address">
                                    <i class="map marker link icon" @click="locatorButtonPressed"> </i>
                                </div>
                            </div>
                        </div>
                    </form>
                </div>
                <div class="column">
                    <button id="report" @click="callReport" class="ui button">
                        <IconLogo/>
                    </button>
                </div>
                <div class="column">
                    
                </div>
            </section>
            <section id="map">

            </section>
        </div>
    </div>
</template>

<script setup>
    import { initializeApp } from "firebase/app";
    import { getFirestore } from "firebase/firestore";
    import { collection, getDocs } from "firebase/firestore";
    import { onMounted, ref } from 'vue';
    import { Loader } from '@googlemaps/js-api-loader';
    const loader = new Loader({
        apiKey: 'AIzaSyDFyHoukxX9LkF9c_r0gI7TNQbEYdwNt0U',
        version: 'weekly',
        libraries: ['places', 'visualization']
    });

    const map = ref([]);
    let heatmapLayer;
    const autocompleteInput = ref(null);
    //const autocomplete = ref ([]);

    const firebaseConfig = {
        apiKey: "AIzaSyDFyHoukxX9LkF9c_r0gI7TNQbEYdwNt0U",
        authDomain: "onfirepuc.firebaseapp.com",
        projectId: "onfirepuc",
        storageBucket: "onfirepuc.appspot.com",
        messagingSenderId: "118780534932",
        appId: "1:118780534932:web:97df285c7beff2c81c5481",
        measurementId: "G-89NP98DGHQ"
    };

    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);

    
    onMounted(async () => {

        var heatMapDataLat = [];
        var heatMapDataLng = [];
        var i = 0;
        const querySnapshot = await getDocs(collection(db, "reports"));
            querySnapshot.forEach((doc) => {
                console.log(i)
            get_Ocorrences(doc.data().lat, doc.data().long, i);
            i = i + 1;
        });
        function get_Ocorrences(lat, long, i){
            heatMapDataLat[i] = lat;
            heatMapDataLng[i] = long;
        }


        await loader
        .load()
        .then(google => {

            

            const autocomplete = new google.maps.places.Autocomplete(autocompleteInput.value);

            autocomplete.addListener('place_changed', () => {
                const place = autocomplete.getPlace();
                setCenter();
                // Ação a ser executada quando um local é selecionado
                console.log(place.formatted_address);
            });
            function setCenter(){
                let endereco = document.getElementById("endereco").value;
                endereco = endereco.replaceAll(' ', '%20');
                axios.get("https://maps.googleapis.com/maps/api/geocode/json?address=" + endereco +  "&key=AIzaSyDFyHoukxX9LkF9c_r0gI7TNQbEYdwNt0U")
                .then(response =>{
                    var latitude = response.data.results[0].geometry.location.lat;
                    var longitude = response.data.results[0].geometry.location.lng;
                    map.value.setCenter(new google.maps.LatLng(latitude, longitude));
                })
            }
            function initmap(){
                navigator.geolocation.getCurrentPosition(
                    position => {
                        map.value = new google.maps.Map(document.getElementById('map'), {
                            center: { lat: position.coords.latitude, lng: position.coords.longitude },
                            zoom: 16,
                            disableDefaultUI: true,
                        });
                        var heatMapData = [];
                        var count = 0
                        while(count < i ){
                            console.log(count);
                            var lat = heatMapDataLat[count];
                            var long = heatMapDataLng[count];
                            heatMapData[count] = new google.maps.LatLng(lat, long);
                            count = count + 1
                        }
                        
                        heatmapLayer = new google.maps.visualization.HeatmapLayer({
                        data: heatMapData,
                        radius: 30
                        });
                        heatmapLayer.setMap(map.value);
                    },
                )
            } initmap();
            return {
                autocompleteInput
            };  
        })
        .catch(error => {
            console.log(error)
        })
        .then(function () {
        })
        
    })
</script>

<script>
const screenHeight = window.screen.height;
export {screenHeight};
import axios from 'axios'
import IconLogo from './icons/IconLogo.vue'
import ReportComponent from './ReportComponent.vue'

export default {
    name: 'MapsComponent',
    components: {
        IconLogo,
        ReportComponent
    },

    data() {
        return{
            report: false,
            address: "",
            error: "",
            spinner: false,
        }
    },

    methods: {
        dismissReport(){
            this.report = false;
        },
        callReport(){
            this.report = true;
        },
        locatorButtonPressed(){
            this.spinner = true;
            if(navigator.geolocation){
                navigator.geolocation.getCurrentPosition(
                    position => {
                        this.getAddressFrom(position.coords.latitude, position.coords.longitude);
                    },
                    error => {
                        this.error = error.message;
                        console.log(this.error.message);
                        this.spinner = false;
                    }
                );
            }
            else{
                var error = "Seu navegador nao suporta a geolocalização, por favor a informe abaixo"
                this.error = error.message;
                this.spinner = false;
            }
        },


        getAddressFrom(lat, long){
            axios.get("https://maps.googleapis.com/maps/api/geocode/json?latlng="
             + lat + "," + long + "&key=AIzaSyDFyHoukxX9LkF9c_r0gI7TNQbEYdwNt0U")
             .then(response => {
                if(response.data.error_message) {
                    this.error = response.data.error_message
                    console.log(response.data.error_message);
                } else{
                    this.address = response.data.results[0].formatted_address;
                    console.log(response.data.results[0].formatted_address);
                }
                this.spinner = false;
             })
             .catch(error => {
                this.error = error.message;
                this.spinner = false;
                console.log(error.message);
             })
        }
    },
}
</script>