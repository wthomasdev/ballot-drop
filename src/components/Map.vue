
<template>
  <div>
    <gmap-map
        ref="gmap"
        :center="center"
        :zoom="12"
        style="width:100%;  height: 100vh;"
        @click="closeInfoWindows()"
    >

      <gmap-marker
          :key="index"
          v-for="(m, index) in locations"
          :position="{lat: parseFloat(m.lat), lng: parseFloat(m.lng)}"
          @click="toggleInfoWindow(m,index)">
      </gmap-marker>

      <gmap-info-window
          :options="infoOptions"
          :position="infoWindowPos"
          :opened="infoWinOpen"
          @closeclick="infoWinOpen=false"
      >
        <div v-html="infoContent"></div>
      </gmap-info-window>

    </gmap-map>
  </div>
</template>

<script>



export default {
  name: 'GoogleMap',
  props: {
    // county_fips: String,
    locations: {
      type: Array,
      required: true
    }
  },
  data: function(){
    return {
      // locations: null,

      //a default center for the map
      center: {lat: 39.7392, lng: -104.9903},
      map: null,
      infoContent: '',
      infoWindowPos: {
        lat: 0,
        lng: 0
      },
      infoWinOpen: false,
      currentMidx: null,
      //optional: offset infowindow so it visually sits nicely on top of our marker
      infoOptions: {
        pixelOffset: {
          width: 0,
          height: -35
        }


      }
    }
  },
  methods: {
    toggleInfoWindow: function (marker, idx) {
      this.infoWindowPos = {lat: parseFloat(marker.lat), lng: parseFloat(marker.lng)};
      this.infoContent = this.getInfoWindowContent(marker);


      //check if its the same marker that was selected if yes toggle
      if (this.currentMidx == idx) {
        this.infoWinOpen = !this.infoWinOpen;
      }
      //if different marker set infowindow to open and reset current marker index
      else {
        this.infoWinOpen = true;
        this.currentMidx = idx;
      }
    },

    getInfoWindowContent: function (marker) {
      return (`
<div class="">
  <div class="card-content">
    <div class="media">
      <div class="media-content">
        <strong>${marker.Name}</strong>
      </div>
    </div>
    <div class="content">
      ${marker.Address} <br />
      ${marker.City}, ${marker.['State (from County Absentee Voting Data)']} ${marker.Zip}
      <hr />
      Hours: ${marker.Hours}
    </div>
  </div>
</div>`);
    },

    closeInfoWindows: function() {
      this.infoWinOpen = false;
    }
  },

  mounted() {
    this.$refs.gmap.$mapPromise.then((map) => {
      const bounds = new window.google.maps.LatLngBounds()
      for (let m of this.locations) {
        bounds.extend({lat: parseFloat(m.lat), lng: parseFloat(m.lng)})
      }
      map.fitBounds(bounds);

      // Deal with zoom delay on load
      setTimeout(function(){
        if (map.getZoom() > 18 ) map.setZoom(16);
      },
        100
      );

    });
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>


</style>
