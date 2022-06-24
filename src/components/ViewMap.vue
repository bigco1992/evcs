<template>
    <div class="flex flex-middle flex-center w100p h100p">
        <div class="w100p h100p mapWrapper"></div>
    </div>
</template>

<script>
import axios from "axios";

export default {
    data() {
        return {
            wrapper : null,
            map : null,
            marker : null,
            mapOptions : null,
            mouseLatlng : null,
            bounds : null,
            swLatlng : null,
            neLatlng : null,
        }
    }, 
    methods : {
        drawMap() {
            var _this = this;
            
            this.wrapper = document.querySelector(".mapWrapper");

            navigator.geolocation.getCurrentPosition((pos) => {
                this.mapOptions = {
                    center: new window.kakao.maps.LatLng(pos.coords.latitude, pos.coords.longitude),
                    level: 3
                };
                this.map = new window.kakao.maps.Map(this.wrapper, this.mapOptions);

                this.marker = new window.kakao.maps.Marker({
                    position: this.map.getCenter(),
                });

                this.marker.setMap(this.map);
                this.map.addOverlayMapTypeId(kakao.maps.MapTypeId.TRAFFIC);

                window.kakao.maps.event.addListener(_this.map, 'click', function(mouseEvent) {        
                    _this.mouseLatlng = mouseEvent.latLng; 
                    _this.marker.setPosition(_this.mouseLatlng);
                    // 위도
                    console.log(_this.mouseLatlng.La);
                    // 경도
                    console.log(_this.mouseLatlng.Ma);

                    _this.fetchData(_this.mouseLatlng.La, _this.mouseLatlng.Ma);
                });

                window.kakao.maps.event.addListener(this.map, 'bounds_changed', function() {             
                    _this.bounds = _this.map.getBounds();
                    _this.swLatlng = _this.bounds.getSouthWest();
                    _this.neLatlng = _this.bounds.getNorthEast();
                });
            });
        },

        async fetchData(la, ma) {
            const response = await axios.get(`?lat=${ma}&lng=${la}`);
            console.log(response);
        }
    },
    mounted() {
        if (!window.kakao || !window.kakao.maps) {
            const scriptTag = document.createElement("script");
            scriptTag.src = `//dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=${process.env.VUE_APP_KAKAO_API_KEY}`;

            scriptTag.addEventListener("load", () => {
                window.kakao.maps.load(this.drawMap);
            });

            document.head.appendChild(scriptTag);
        } else {
            this.drawMap();
        }
    }
}
</script>

<style scoped>
    
</style>