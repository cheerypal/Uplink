<template>
  <!--Room Page - similar to the Dash-->
  <div id="room">
    <!--Top Navbar (Website)-->
    <NavTop class="top-show" :userToken="userToken" />
    <div class="bottom-show">
      <div class="logo-back fixed-top">
        <h5 class="logo">
          <router-link class="links-top" :to="{ name: 'dashboard' }">uplink</router-link>
        </h5>
      </div>
    </div>
    <!--Take the roomName from props and store as title for the room page - demo data is also entered-->
    <Summary
      :userToken="this.userToken"
      :sumTitle="roomName"
      energy="200"
      temperature="18"
      solar="1000"
    />
    <div class="container">
      <div id="rooms">
        <div class="sub-title-wrapper">
          <h3 class="h1-titles text-center">Devices</h3>
          <hr />
        </div>
        <div class="flex-rooms">
          <!--Loop through all the devices by device name and generate cards for each of them-->
          <div v-for="device in roomDevices" :key="device.deviceID">
            <!--Data that the Device component uses to show the card.-->
            <Device
              :deviceID="device.deviceID"
              :deviceName="device.deviceName"
              :deviceImage="device.deviceImage"
              :deviceEnergy="device.deviceWattage"
              :deviceType="device.deviceType"
              :roomID="roomID"
            />
          </div>
        </div>
        <hr />
        <!--Additional Components.-->
        <div class="text-center">
          <router-link
            class="advanced-links text-center links"
            :to="{ name: 'allDevicesPage' }"
          >See All Devices</router-link>
        </div>
      </div>
    </div>
    <!--Bottom Navbar (Mobile and Tablet)-->
    <NavBottom class="bottom-show" :userToken="userToken" />
  </div>
</template>
<script>
//All the components that are needed for the roomPage
import Summary from "../helpers/roomSummary";
import Device from "../helpers/deviceCard";

import NavTop from "../navbars/navbar-top";
import NavBottom from "../navbars/navbar-bottom";

//URL for getting all the devices
let url = "http://localhost:5552/getDevices";
//URL for getting all rooms
let url1 = "http://localhost:5552/getRooms";
export default {
  name: "Room",
  components: {
    //Initialise the components
    Summary,
    Device,
    NavTop,
    NavBottom
  },
  props: ["userToken"], //props to confirm the room the page is showing
  data() {
    return {
      roomName: this.$route.params.name,
      devices: [], //all devices stored in the database
      rooms: [], //all rooms stored in the database
      roomDevices: [], //finished array containing the device for the room and the device icon
      roomID: 0
    };
  },

  mounted: function() {
    fetch(url, {
      mode: "cors",
      method: "GET",
      headers: { Authorization: this.userToken }
    }) //first fetch gets all the devices and stores in devices array
      .then(response => {
        return response.json();
      })
      .then(jsonData => {
        this.devices = jsonData;
        fetch(url1, {
          mode: "cors",
          method: "GET",
          headers: { Authorization: this.userToken }
        }) //second fetch gets all the rooms and stored in rooms array
          .then(response => {
            return response.json();
          })
          .then(jsonData => {
            this.rooms = jsonData;
            console.log(this.rooms);
            //parses data to only get devices for the room the page is showning
            for (let key in this.rooms) {
              if (this.roomName === this.rooms[key].room_name) {
                this.roomID = this.rooms[key].room_id;
                console.log(this.roomID);
              }
            }
            for (let key in this.devices) {
              if (this.roomID === this.devices[key].device_room) {
                //Loop to get the icon that matches the second half of the device name
                this.roomDevices.push({
                  //generate a JSON of the device name and icon and store in roomDevices array
                  deviceID: this.devices[key].device_id,
                  deviceName: this.devices[key].device_name,
                  deviceImage: pairImg(this.devices[key].device_type),
                  deviceWattage: this.devices[key].device_wattage,
                  deviceType: this.devices[key].device_type
                });
              }
            }
          });
      });
  }
};

//picks image name for a select device.
function pairImg(deviceType) {
  if (deviceType == 1) {
    return "heating";
  }
  if (deviceType == 2) {
    return "fridge";
  }
  if (deviceType == 3) {
    return "solarpanel";
  }
  if (deviceType == 4) {
    return "light-bulb";
  }
  if (deviceType == 5) {
    return "doorbell";
  }
  if (deviceType == 6) {
    return "solarpanel";
  }
  if (deviceType == 7) {
    return "security-camera";
  }
  if (deviceType == 8) {
    return "tv";
  }
  if (deviceType == 9) {
    return "plug";
  }
}
</script>
