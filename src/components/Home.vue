<template>
 <div>
    <header class="h-[40vh]">
        <div class="w-96 mx-auto pt-4">
            <h1 class="text-center font-bold text-lg text-white">IP Address Tracker</h1>
            <div class="flex my-6 overflow-hidden rounded-xl">
                <input class="p-3 grow focus:outline-none text-sm" v-model="ip" type="text" placeholder="Search for any IP address or domain">
                <div class="bg-black w-[12%] hover:bg-gray-800 focus:bg-slate-800 cursor-pointer transition-all duration-200 flex justify-center items-center"
                @click="getIp">
                    <img src="../assets/images/icon-arrow.svg" alt="" class="w-1/5 h-1/4">
                </div>
            </div>
        </div>
    </header>
    <div id="map" class="h-[60vh]"></div>
 </div>
</template>
 
<script>
import "leaflet/dist/leaflet.css";
import L from "leaflet";

 export default{ 
 name: "Home",
 data(){
    return {
        api_key: 'at_0UdHagaH9Uxou2gSySG8rdS3ijWVw',
        ip: '',
        addressObj: {},
    }
 },
 methods: {
    
    initMap(){
        var map = L.map('map').setView([51.505, -0.09], 13);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 19,
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(map);

        var myIcon = L.icon({
            iconUrl: '../src/assets/images/icon-location.svg',
            iconSize: [25, 30],
            iconAnchor: [22, 64],
            popupAnchor: [-10, -56],
        });
        L.marker([51.5, -0.09], {icon: myIcon}).addTo(map)
            .bindPopup('A pretty CSS3 popup.<br> Easily customizable.')
            .openPopup();
    },
    getIp(){
        var url = 'https://geo.ipify.org/api/v2/country?apiKey=' + this.api_key + '&ipAddress=' + this.ip;
        if(this.ip.length > 0){
            fetch(url)
            .then(response => response.json())
            .then(data => {
                this.addressObj = data;
                console.log(this.addressObj);
            })
            .catch(error => console.log(error));
        }else{
            alert('please input IP address!')
        }
    }
 },
 mounted(){
    this.initMap(); // initialize map on component mount
    
 },

}
</script>
<style scoped>
header{
    background: url('../assets/images/pattern-bg.png') no-repeat;
    background-size: cover;
}

</style>
