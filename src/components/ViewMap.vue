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
            data: [],
            originData: null,
            wrapper : null,
            map : null,
            marker : null,
            placeMarker : [],
            mapOptions : null,
            mouseLatlng : null,
            bounds : null,
            swLatlng : null,
            neLatlng : null,
        }
    }, 
    methods : {
        drawGeoMap() {
            var _this = this;
            
            this.wrapper = document.querySelector(".mapWrapper");

            navigator.geolocation.getCurrentPosition((pos) => {
                this.drawKakaoMap(pos.coords.latitude, pos.coords.longitude);
                this.drawMyPlace();
                this.map.addOverlayMapTypeId(kakao.maps.MapTypeId.TRAFFIC);
                this.clickMarker();
                this.changePlaceMarker();
            });
        },

        async fetchData(la, ma) {
            var _this = this;
            var imageSrc = "https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/markerStar.png"; 
            var response = await axios.get(`${process.env.VUE_APP_REST_API}?lat=${ma}&lng=${la}`);
            
            if (response.data.data != null) {
                _this.data = [];
                
                _this.resetMarker();

                response.data.data.map(item => {
                    _this.data.push({ title :  item.statNm, latlng: new window.kakao.maps.LatLng(item.lat, item.lng)});
                });
    
               this.data.map((item, idx) => {
                    var imageSize = new window.kakao.maps.Size(24, 35); 
                    var markerImage = new window.kakao.maps.MarkerImage(imageSrc, imageSize); 
                    
                    var marker = new window.kakao.maps.Marker({
                        map: _this.map,
                        position: _this.data[idx].latlng,
                        title : _this.data[idx].title,
                        image : markerImage,
                    });

                    _this.placeMarker.push(marker);
                });
            } else { 
                return;
            }
        },

        changePlaceMarker() {
            var _this = this;

            window.kakao.maps.event.addListener(this.map, "bounds_changed", () => {
                _this.bounds = _this.map.getBounds();
                _this.swLatlng = _this.bounds.getSouthWest();
                _this.neLatlng = _this.bounds.getNorthEast();
            });
        },

        drawKakaoMap(lat, lng) {
            this.mapOptions = {
                center: new window.kakao.maps.LatLng(lat, lng),
                level: 3,
            };

            this.map = new window.kakao.maps.Map(this.wrapper, this.mapOptions);
        },

        drawMyPlace() {
            this.marker = new window.kakao.maps.Marker({
                position: this.map.getCenter(),
            });

            this.marker.setMap(this.map);
        },

        clickMarker() {
            var _this = this;
            
            window.kakao.maps.event.addListener(_this.map, "click", (mouseEvent) => {
                _this.mouseLatlng = mouseEvent.latLng;
                _this.marker.setPosition(_this.mouseLatlng);
                
                _this.fetchData(_this.mouseLatlng.La, _this.mouseLatlng.Ma);
            });
        },

        resetMarker() {
            this.placeMarker.map(item => {
                item.setMap(null);
            });
        }
    },
    mounted() {
        if (!window.kakao || !window.kakao.maps) {
            const scriptTag = document.createElement("script");
            scriptTag.src = `//dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=${process.env.VUE_APP_KAKAO_API_KEY}`;
            // dapi.kakao.com/v2/maps/sdk.js?appkey=APIKEY&libraries=services,clusterer,drawing


            scriptTag.addEventListener("load", () => {
                window.kakao.maps.load(this.drawGeoMap);
            });

            document.head.appendChild(scriptTag);
        } else {
            this.drawGeoMap();
        }
    }
}
</script>

<style scoped>
    
</style>