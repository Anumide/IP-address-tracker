<template>
 <div>
    <!-- <div v-if="isLoading" ref="loader" class="z-[100] absolute w-full flex justify-center items-center h-screen top-0 left-0 bg-black bg-opacity-30 text-white font-semibold"> Loading... </div> -->
    <header class="h-[35vh]">
        <div class="w-[90%] max-w-sm mx-auto pt-4">
            <h1 class="text-center font-bold text-lg text-white">IP Address Tracker</h1>
            <div class="flex my-6 overflow-hidden rounded-xl">
                <input class="p-3 grow focus:outline-none text-sm" v-model="ip" type="text" placeholder="Search for any IP address" @keyup.enter="getIp">
                <div class="bg-black w-[12%] hover:bg-gray-800 focus:bg-slate-800 cursor-pointer transition-all duration-200 flex justify-center items-center"
                @click="getIp">
                    <img src="../assets/images/icon-arrow.svg" alt="" class="w-1/5 h-1/4">
                </div>
            </div>
        </div>
    </header>
    <div class="flex justify-center items-center absolute bg-white top-[23%] md:top-1/4 lg:top-[25%] z-20 rounded-xl w-[90%] md:w-4/5 max-w-4xl left-5 xl:left-[17%] md:inset-x-24 shadow-2xl" :class="{ 'h-32': isLoading }">
        <div 
            v-if="!isLoading"
            class="md:grid grid-cols-4 md:text-left md:divide-x divide-neutral-200 divide-solid p-6 md:py-6 md:px-0  text-center w-full"
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
        <div v-else class="loader">
        </div>
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
        fetchedIp: null,
        isLoading: true,
        map: null,
        myIcon: null
    }
 },
 methods: {
    
    initMap(){
        this.map = L.map('map').setView([this.lat, this.lng], 17);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 19,
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(this.map);

        this.myIcon = L.icon({
            iconUrl: 'https://cdn-icons-png.flaticon.com/128/3082/3082383.png',
            iconSize: [30, 30],
            iconAnchor: [22, 64],
            popupAnchor: [-7, -56],
        });
        L.marker([this.lat, this.lng], {icon: this.myIcon}).addTo(this.map)
            .bindPopup('Your Location')
            .openPopup();
    },
    getIp(){
        this.addressObj = JSON.parse(localStorage.data)
        if(this.ip.length > 0){ 
                let e = this.addressObj.map(e => e.ip)
                if(e.includes(this.ip)){
                    this.isLoading = false
                        this.fetchedIp = this.addressObj[e.indexOf(this.ip)].ip
                        this.lat = this.addressObj[e.indexOf(this.ip)].location.lat;
                        this.lng = this.addressObj[e.indexOf(this.ip)].location.lng;
                        this.ISP = this.addressObj[e.indexOf(this.ip)].isp;
                        this.timezone = 'UTC ' + this.addressObj[e.indexOf(this.ip)].location.timezone;
                        this.city = this.addressObj[e.indexOf(this.ip)].location.city;
                        this.country = this.addressObj[e.indexOf(this.ip)].location.country;
                        this.postalCode = this.addressObj[e.indexOf(this.ip)].location.postalCode;
                        L.marker([this.lat, this.lng], {icon: this.myIcon}).addTo(this.map)
                        .bindPopup('Your Location')
                        .openPopup();
                        this.map.flyTo(L.latLng(this.lat, this.lng))
                        this.ip = ''
                    }else{  // fetch the IP address if it hasn't been fetched yet
                    this.isLoading = true
                        let clientFetchedIp = async () =>{
                        const fetchedData = await this.fetchIp(this.ip)
                        return fetchedData;
                        }

                        clientFetchedIp().then(data => {
                            this.isLoading = false
                            this.data = JSON.parse(localStorage.data)
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
                            L.marker([this.lat, this.lng], {icon: this.myIcon}).addTo(this.map)
                            .bindPopup('Your Location')
                            .openPopup();
                            this.map.flyTo(L.latLng(this.lat, this.lng))
                            this.ip = ''
                        })
                        .catch(error => console.error(error));
                    }
        }else{
            alert('please input IP address!')
        }
    },
    getClientIp: async () => {
        const res = await fetch('https://api.ipify.org?format=json')
        const data = await res.json()
        return data.ip
    },
    fetchIp: async (ip) =>{
        const api_key = 'at_0UdHagaH9Uxou2gSySG8rdS3ijWVw'
        var url = 'https://geo.ipify.org/api/v2/country,city?apiKey=' + api_key + '&ipAddress=' + ip;
        const res = await fetch(url)
        const data = await res.json()
        return data
    }
 },
 mounted(){
     if(!localStorage.data){
        this.isLoading = true
        let clientFetchedIp = async () =>{
        const clientIp = await this.getClientIp()
        const fetchedData = await this.fetchIp(clientIp)
        return fetchedData;
        }

        clientFetchedIp().then(data => {
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
            this.initMap()
            this.isLoading = false                                                        
        })
        .then(err => console.error(err))
    }else{
        this.addressObj = JSON.parse(localStorage.data)
        this.getClientIp().then(ip => {
            let e = this.addressObj.map(e => e.ip)
            
            if(e.includes(ip)){
                    this.isLoading = false
                    this.lat = this.addressObj[e.indexOf(ip)].location.lat;
                    this.lng = this.addressObj[e.indexOf(ip)].location.lng;
                    this.ISP = this.addressObj[e.indexOf(ip)].isp;
                    this.timezone = 'UTC ' + this.addressObj[e.indexOf(ip)].location.timezone;
                    this.city = this.addressObj[e.indexOf(ip)].location.city;
                    this.country = this.addressObj[e.indexOf(ip)].location.country;
                    this.postalCode = this.addressObj[e.indexOf(ip)].location.postalCode;
                    this.fetchedIp = this.addressObj[e.indexOf(ip)].ip
                    this.initMap()
                }else{
                    this.isLoading = true
                    this.fetchIp(ip).then(data => {
                        this.data = JSON.parse(localStorage.data)
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
                        this.initMap()
                        this.isLoading = false
                    })
                }
        })
    }
 }

}
</script>
<style scoped>
header{
    background: url('../assets/images/pattern-bg.png') no-repeat;
    background-size: cover;
}

.loader{
    @apply w-8 h-8 rounded-full border-solid border-4 border-black/20 border-r-4 border-r-black/70;
    animation: spin 950ms infinite linear;
}

@keyframes spin {
    to{
        transform: rotate(360deg);
    }
}
</style>
