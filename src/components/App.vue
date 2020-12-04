<template>
  <Page androidStatusBarBackground="#474747">
    <ActionBar title="Realtime Location NSVUE"/>
    <StackLayout>
      <WrapLayout horizontalAlignment="center">
        <Button @tap="getDirections">Get Directions</Button>
        <Button @tap="clearRoute">Clear Route</Button>
        <Button @tap="startJourney">Start Journey</Button>
        <button @tap="endJourney">End Journey</button>
      </WrapLayout>

      <DockLayout>
        <MapView
          width="100%"
          height="85%"
          dock="top"
          :zoom="zoom"
          :latitude="origin.latitude"
          :longitude="origin.longitude"
          v-if="allowExecution"
          @mapReady="mapReady"
          @coordinateLongPress="locationSelected"
        />
        <TextView dock="bottom" :text="journeyDetails" editable="false"/>
      </DockLayout>
    </StackLayout>
  </Page>
</template>

<script>
import * as permissions from 'nativescript-permissions'
import * as platform from 'platform'
import  MapsHelper from "./MapsHelper.js";

export default {
    data() {
    return {
      origin: { latitude: 37.4220, longitude: -122.0840},
      destination: { latitude: 37.4220, longitude: -90.0840},
      journeyDetails: "Journey: Not started yet!",
      allowExecution: false,
      journeyStarted: false,
      mapView: null,
      zoom: 17,
      APIKEY: "AIzaSyCDrhtDQd0MFyuQKV6KpaqcCEB7I2Wo2dM"
    }
  },
  created: function() {
  /* dont run the android permissions routine for iOS */
  if (platform.isIOS) {
    this.allowExecution = true;
    return;
  }
  /* list of permissions needed */
  let permissionsNeeded = [
    android.Manifest.permission.ACCESS_FINE_LOCATION,
    android.Manifest.permission.ACCESS_COARSE_LOCATION
  ];
  /* showing up permissions dialog */
  permissions
    .requestPermissions(permissionsNeeded, "Give it to me!")
    .then(() => this.allowExecution = true)
    .catch(() => this.allowExecution = false);
},
methods: {
    mapReady(args) {
      this.mapView = args.object;
      this.enableMyLocationButton(true);
      this.addMarkerToMap(this.destinationMarker, true);
      this.addMarkerToMap(this.myLocationMarker, false);
    },
    locationSelected(args) {
      /* get coordinates of the point where user long pressed */
      let lat = args.position.latitude;
      let lng = args.position.longitude;
      /* set the obtained coordinates as destination coordinates */
      this.destination.latitude = lat;
      this.destination.longitude = lng;
      /* move the destination marker to the same coordinates */
      this.setMarker(this.destinationMarker, lat, lng);
    },
    getDirections() {
      /* hit Directions API - as origin and destination coordinates are set */
      this.hitDirectionsAPI().then(response => {
        /* draw route from encoded polyline points */
        this.drawRoute(response.encodedPolylinePoints);
        
        /* if jouney started, don't adjust the camera */
        if (this.journeyStarted)
          return;

        /* adjust camera to bring route into view */
          this.getRouteInView(
            response.northEastBounds,
            response.southWestBounds
          );
      });
    },
    clearRoute() {
      /* remove route drawn between locations on map */
      this.mapView.removeAllPolylines();
    },
    startJourney() {
      /* hide my location indicator and button */
      this.enableMyLocationButton(false);
      /* un-hide my location marker */
      this.myLocationMarker.visible = true;
      /* update journey details */
      this.journeyStarted = true;
      this.journeyDetails = "Journey started...";
      /* start watching for location changes and update the map and journey details accordingly */
      this.watchLocationAndUpdateJourney();
    },
    endJourney() {
      /* stop watching for location changes */
      this.clearWatch();
      /* remove the route drawn on map */
      this.clearRoute();
      /* hide my location marker  */
      this.myLocationMarker.visible = false;
      /* bring back my location button on screen */
      this.enableMyLocationButton(true);
      /* update journey details */
      this.journeyStarted = false;
      this.journeyDetails = "Destination Reached.";
    }
  },
  mixins: [ MapsHelper.MapsUIHelper, MapsHelper.DirectionsAPIHelper, MapsHelper.DistanceMatrixAPIHelper, MapsHelper.LocationHelper ]
};
</script>

<style>
Button {
  font-size: 9;
  background-color: #474747;
  color:white;
  width:25%;
}
ActionBar {
  background-color: #474747;
  color:white;
}
Page {
  background-color: #474747;
}
TextView {
  border-bottom-color: transparent;
  color:white;
  border-bottom-width: 1;
  padding: 15;
}
</style>
