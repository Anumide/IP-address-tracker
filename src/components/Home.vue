<template>
 <div>
    <header class="h-[40vh]">
        <div class="w-96 mx-auto pt-4">
            <h1 class="text-center font-bold text-lg text-white">IP Address Tracker</h1>
            <div class="flex my-6 overflow-hidden rounded-xl">
                <input class="p-3 grow focus:outline-none text-sm" v-model="ip" type="text" placeholder="Search for any IP address or domain">
                <div class="bg-black w-[12%] hover:bg-gray-800 focus:bg-slate-800 cursor-pointer transition-all duration-200 flex justify-center items-center"
                @click="getIp" @keydown.enter="getIp">
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
        // https://api.db-ip.com/v2/free/self for getting ip address of client
        api_key: 'at_0UdHagaH9Uxou2gSySG8rdS3ijWVw',
        ip: '',
        addressObj: {},
        data: [],
        lat: null,
        lng: null,
        city: null,
        country: null,
        postalCode: null,
        timezone: null,
        ISP: null,
    }
 },
 methods: {
    
    initMap(){
        var map = L.map('map').setView([this.lat, this.lng], 17);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 19,
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(map);

        var myIcon = L.icon({
            iconUrl: '../src/assets/images/icon-location.svg',
            iconSize: [30, 35],
            iconAnchor: [22, 64],
            popupAnchor: [-7, -56],
        });
        L.marker([this.lat, this.lng], {icon: myIcon}).addTo(map)
            .bindPopup('A pretty CSS3 popup.<br> Easily customizable.')
            .openPopup();
    },
    getIp(){
        var url = 'https://geo.ipify.org/api/v2/country,city?apiKey=' + this.api_key + '&ipAddress=' + this.ip;
            
        if(this.ip.length > 0){
                
            if(this.addressObj.length > 0){   // if addressObj is not empty, then check if the IP address has been fetched
                 this.addressObj.forEach(e => {
                    if(e.ip == this.ip){
                        this.lat = e.location.lat;
                        this.lng = e.location.lng;
                        this.ISP = e.isp;
                        this.timezone = 'UTC ' + e.location.timezone;
                        this.city = e.location.city;
                        this.country = e.location.country;
                        this.postalCode = e.location.postalCode;
                        console.log(this.lat, this.lng);
                    }else{  // fetch the IP address if it hasn't been fetched yet
                         fetch(url)
                        .then(response => response.json())
                        .then(data => {
                            this.data.push(data)
                            localStorage.data = JSON.stringify(this.data)
                            this.addressObj = JSON.parse(localStorage.data)
                        })
                        .catch(error => console.log(error));
                    }
                 })
            }else{
                fetch(url)
                .then(response => response.json())
                .then(data => {
                    this.data.push(data)
                    localStorage.data = JSON.stringify(this.data)
                    this.addressObj = JSON.parse(localStorage.data)
                    console.log(this.addressObj);
                })
                .catch(error => console.log(error));
            }
        }else{
            alert('please input IP address!')
        }
    }
 },
 mounted(){
    this.initMap(); // initialize map on component mount
     console.log(this.lat, this.lng, this.ISP, this.timezone, this.city, this.country, this.postalCode);
    
 },
 created(){
    this.addressObj = JSON.parse(localStorage.data)
    // this.addressObj = data;
    this.addressObj.forEach(e => {
        this.lat = e.location.lat;
        this.lng = e.location.lng;
        this.ISP = e.isp;
        this.timezone = 'UTC ' + e.location.timezone;
        this.city = e.location.city;
        this.country = e.location.country;
        this.postalCode = e.location.postalCode;
    })
    console.log(this.addressObj);
 }

}
</script>
<style scoped>
header{
    background: url('../assets/images/pattern-bg.png') no-repeat;
    background-size: cover;
}

</style>
