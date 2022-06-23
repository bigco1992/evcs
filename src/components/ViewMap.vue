<template>
    <div class="flex flex-middle flex-center w100p h100p">
        <div class="w100p h100p mapWrapper"></div>
    </div>
</template>

<script>

export default {
    data() {
        return {
            wrapper : null,
            map : null,
            marker : null,
            mapOptions : null,
            mouseLatlng : null,
        }
    }, 
    methods : {
        drawMap() {
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

                // window.kakao.maps.event.addListener(this.map, 'click', function(mouseEvent) {        
                //     this.mouseLatlng = mouseEvent.latLng; 
                //     this.marker.setPosition(this.mouseLatlng);
                // });
            });
        },
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