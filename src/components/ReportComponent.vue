<template>
    <div>
        <form class="ui segment large form">
            <div class="ui message light red" v-show="error">></div>
            <div class="ui segment">
                <p>Relatar um caso de incêndio</p>
                <div class="field">
                    <div class="ui right icon input large">
                        <input type="text" ref="autocompleteInput" placeholder="Seu endereço" id="endereco_forms">

                        <button type="button" class="ui button" style="margin: 0; min-width: 20px;">
                            <i class="map marker link icon" style="margin: 0; color: white;"> </i>
                        </button>
                        
                    </div>
                    
                </div>
                <p>Caso seja mais facil, clique no mapa para selecionar a área desejada</p>
                <div class="field" style="border: 1px solid black;">
                    <div id="forms_map"></div>
                </div>
            </div>
            <button type="button" @click="addReport" class="ui button" >
                Reportar
            </button>
        </form>
    </div>
</template>

<script>
import axios from 'axios'
import { Loader } from '@googlemaps/js-api-loader';
import { onMounted, ref } from 'vue';
import { initializeApp } from "firebase/app";
import { getFirestore } from "firebase/firestore";
import { collection, addDoc } from "firebase/firestore";

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

export default{
    name:'ReportComponent',
    methods: {
        addReport(){
            let endereco = document.getElementById("endereco_forms").value;
            var formatted_address = endereco
            endereco = endereco.replaceAll(' ', '%20');
            axios.get("https://maps.googleapis.com/maps/api/geocode/json?address=" + endereco +  "&key=AIzaSyDFyHoukxX9LkF9c_r0gI7TNQbEYdwNt0U")
                .then(response =>{
                    var latitude = response.data.results[0].geometry.location.lat;
                    var longitude = response.data.results[0].geometry.location.lng;
                    const docRef = addDoc(collection(db, "reports"),{
                        endereco : formatted_address,
                        lat : latitude,
                        long : longitude,
                    });
                    console.log("Document written with ID: ", docRef.id);
            })
            
        }
    },
    setup(){

        var user_position;
        const loader = new Loader({
            apiKey: 'AIzaSyDFyHoukxX9LkF9c_r0gI7TNQbEYdwNt0U',
            version: 'weekly',
            libraries: ['places', 'visualization']
        });
        const autocompleteInput = ref(null);
        onMounted(async () => {
            await loader
            .load()
            .then(google => { 
                const map = ref([]);
                const autocomplete = new google.maps.places.Autocomplete(document.getElementById('endereco_forms'));

                autocomplete.addListener('place_changed', () => {
                    const place = autocomplete.getPlace();
                    setCenter();
                    console.log(place.formatted_address);
                });
                function setCenter(){
                let endereco_forms = document.getElementById("endereco_forms").value;
                endereco_forms = endereco_forms.replaceAll(' ', '%20');
                axios.get("https://maps.googleapis.com/maps/api/geocode/json?address=" + endereco_forms +  "&key=AIzaSyDFyHoukxX9LkF9c_r0gI7TNQbEYdwNt0U")
                .then(response =>{
                    var latitude = response.data.results[0].geometry.location.lat;
                    var longitude = response.data.results[0].geometry.location.lng;
                    map.value.setCenter(new google.maps.LatLng(latitude, longitude));

                    user_position.setPosition(new google.maps.LatLng(response.data.results[0].geometry.location.lat, response.data.results[0].geometry.location.lng))
                })
                };
                function initmap(){
                    navigator.geolocation.getCurrentPosition(
                        position => {
                            
                            map.value = new google.maps.Map(document.getElementById('forms_map'), {
                                center: { lat: position.coords.latitude, lng: position.coords.longitude },
                                zoom: 17,
                                disableDefaultUI: true,
                            });
                            user_position = new google.maps.Marker({
                                position: { lat: position.coords.latitude, lng: position.coords.longitude },
                                map, 
                                title: "Hello World!",
                            });
                            user_position.setMap(map.value);
                            map.value.addListener('click', select_position);
                        },
                    )
                }; initmap();
                function select_position(clickEvent){
                    const position = clickEvent.latLng;
                    set_Center_on_Marker(position)
                    user_position.setPosition(clickEvent.latLng);

                    user_position.setMap(map.value);
                };
                function set_Center_on_Marker(position){
                    map.value.setCenter(position);
                }
                return {
                autocompleteInput
                };
        })
        .catch(error =>{
            console.log(error);
        })
        .then(function () {
        })
    })
    },
}
</script>