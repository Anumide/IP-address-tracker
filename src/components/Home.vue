<template>
 <div>
    <header class="h-[35vh]">
        <div class="w-[90%] max-w-sm mx-auto pt-4">
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
    <div 
        class="md:grid grid-cols-4 md:text-left md:divide-x divide-neutral-200 divide-solid absolute bg-white p-6 md:py-6 md:px-0 top-[23%] md:top-1/4 lg:top-[25%] z-20 rounded-xl text-center w-[90%] md:w-4/5 max-w-4xl left-5 xl:left-[17%] md:inset-x-24 shadow-2xl"
        >
        <ipInfo 
        class=""
        subTitle="IP ADDRESS"
        :title="fetchedIp" />
        <ipInfo 
        class=""
        subTitle="LOCATION"
        :title="city + ', ' + country + ' ' + postalCode" />
        <ipInfo 
        class=""
        subTitle="TIMEZONE"
        :title="timezone" />
        <ipInfo 
        class=""
        subTitle="ISP"
        :title="ISP" />
    </div>
    <div id="map" class="h-[65vh] z-10"></div>
 </div>
</template>
 
<script>
import ipInfo from './ipInfo.vue';
import "leaflet/dist/leaflet.css";
import L from "leaflet";

 export default{ 
 name: "Home",
 components: {
    ipInfo
 },
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
        fetchedIp: null
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
            // .bindPopup('A pretty CSS3 popup.<br> Easily customizable.')
            // .openPopup();
    },
    getIp(){
        var url = 'https://geo.ipify.org/api/v2/country,city?apiKey=' + this.api_key + '&ipAddress=' + this.ip;
            
        if(this.ip.length > 0){
                
            if(this.addressObj.length > 0){   // if addressObj is not empty, then check if the IP address has been fetched
                 this.addressObj.forEach(e => {
                    if(e.ip == this.ip){
                        this.fetchedIp = e.ip
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
                            this.lat = data.location.lat;
                            this.lng = data.location.lng;
                            this.ISP = data.isp;
                            this.timezone = 'UTC ' + data.location.timezone;
                            this.city = data.location.city;
                            this.country = data.location.country;
                            this.postalCode = data.location.postalCode;
                            this.fetchedIp = data.ip
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
                    this.lat = data.location.lat;
                    this.lng = data.location.lng;
                    this.ISP = data.isp;
                    this.timezone = 'UTC ' + data.location.timezone;
                    this.city = data.location.city;
                    this.country = data.location.country;
                    this.postalCode = data.location.postalCode;
                    this.fetchedIp = data.ip
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
     console.log(this.lat, this.lng, this.ISP, this.timezone, this.city, this.country, this.postalCode, this.fetchedIp);
    
 },
 created(){
    let clientIp
    let getClientIp = async () => {
        const res = await fetch('https://api.ipify.org?format=json')
        const data = await res.json()
        return data
    }
    if(!localStorage.data){
        getClientIp().then(data => clientIp = data.ip)
        var url = 'https://geo.ipify.org/api/v2/country,city?apiKey=' + this.api_key + '&ipAddress=' + clientIp;
       return fetch(url)
        .then(response => response.json())
        .then(data => {
            this.data.push(data)
            console.log(data)
            localStorage.data = JSON.stringify(this.data)
            this.addressObj = JSON.parse(localStorage.data)
            console.log(this.addressObj);
            this.lat = data.location.lat;
            this.lng = data.location.lng;
            this.ISP = data.isp;
            this.timezone = 'UTC ' + data.location.timezone;
            this.city = data.location.city;
            this.country = data.location.country;
            this.postalCode = data.location.postalCode;
            this.fetchedIp = data.ip
        })
        .catch(error => console.log(error));
    }else{
        this.addressObj = JSON.parse(localStorage.data)
        console.log(JSON.parse(localStorage.data))
        this.addressObj.forEach(e => {
        if(e.ip == clientIp){
            this.fetchedIp = e.ip
            this.lat = e.location.lat;
            this.lng = e.location.lng;
            this.ISP = e.isp;
            this.timezone = 'UTC ' + e.location.timezone;
            this.city = e.location.city;
            this.country = e.location.country;
            this.postalCode = e.location.postalCode;
            console.log(this.lat, this.lng);
        }else{
            var url = 'https://geo.ipify.org/api/v2/country,city?apiKey=' + this.api_key + '&ipAddress=' + clientIp;
           return fetch(url)
            .then(response => response.json())
            .then(data => {
                this.data.push(data)
                localStorage.data = JSON.stringify(this.data)
                this.addressObj = JSON.parse(localStorage.data)
                console.log(this.addressObj);
                this.lat = data.location.lat;
                this.lng = data.location.lng;
                this.ISP = data.isp;
                this.timezone = 'UTC ' + data.location.timezone;
                this.city = data.location.city;
                this.country = data.location.country;
                this.postalCode = data.location.postalCode;
                this.fetchedIp = data.ip
            })
            .catch(error => console.log(error));
        }
    })
    }
    // this.addressObj = data;
    
    // this.addressObj.forEach(e => console.log(e))
        //  fetch('https://api.ipify.org?format=json')
        // .then(response => response.json())
        // .then(data => console.log(data.ip))
 }

}
</script>
<style scoped>
header{
    background: url('../assets/images/pattern-bg.png') no-repeat;
    background-size: cover;
}

</style>
