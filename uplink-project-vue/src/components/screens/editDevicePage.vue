<template>
  <div>
    <NavbarTop class="top-show" :userToken="userToken" />
    <div class="bottom-show">
      <div class="logo-back fixed-top">
        <h5 class="logo">
          <router-link class="links-top" :to="{ name: 'dashboard' }">uplink</router-link>
        </h5>
      </div>
    </div>
    <div id="editDevice">
      <div class="container">
        <div id="form-addDevice">
          <div class="flex-deviceDetails">
            <div class="card custom-cards-editDevices">
              <div class="img-cont">
                <img
                  :src="require(`../../assets/${deviceImage}.png`)"
                  alt="device icon"
                  class="device-img"
                />
              </div>
              <div class="text-wrapper">
                <h5 class="card-title text-center label-section">{{ deviceName }}</h5>
                <p class="card-text text-center">{{ deviceEnergy }} Watts</p>
              </div>
              <div class="device-cont">
                <b-form @submit="go">
                  <p class="label-section text-center">Add to Schedule</p>

                  <div class="col-sm-12">
                    <label for="input-device-room" class="label">Set Time</label>
                  </div>
                  <div class="col-sm-12">
                    <select
                      v-model="form.hour"
                      class="form-dropdown time-width"
                      required="required"
                    >
                      <option disabled value>Hours</option>
                      <option selected="selected" value="0">{{formatTimeFront(0)}}</option>
                      <option v-for="n in 24" :key="n" :value="n">
                        {{
                        formatTimeFront(n)
                        }}
                      </option>
                    </select>
                    :
                    <select
                      v-model="form.minute"
                      class="form-dropdown time-width"
                      required="required"
                    >
                      <option disabled value>Minutes</option>
                      <option selected="selected" alue="0">{{formatTimeFront(0)}}</option>
                      <option v-for="n in 60" :key="n" :value="n">
                        {{
                        formatTimeFront(n)
                        }}
                      </option>
                    </select>
                  </div>

                  <div class="form-rows">
                    <div class="col-sm-12">
                      <label for="input-device-room" class="label">Set Operation</label>
                    </div>
                    <div class="col-sm-12">
                      <select v-model="form.operation" class="form-dropdown" required="required">
                        <option disabled value>Please Select An Operation</option>
                        <option
                          v-for="op in operations"
                          :key="op.device_command_id"
                          :value="op.device_command_id"
                        >{{ op.device_command_name }}</option>
                      </select>
                    </div>
                  </div>
                  <div class="newRowSwitch">
                    <div class="form-rows">
                      <div class="col-sm-12">
                        <button class="form-buttons" type="submit">Add To Schedule</button>
                      </div>
                    </div>
                  </div>
                </b-form>
              </div>
            </div>

            <div class="card custom-cards-devicesDetails-schedule-edit">
              <h5 class="card-title text-center label-section">Delete Schedule Times</h5>
              <div class="form-rows" />
              <ul class="list-schedule">
                <li class="scheduleItem" v-for="command in scheduledCommands" :key="command.id">
                  {{ command.command }} at {{ command.hour }}:{{
                  command.minutes
                  }}
                  <br />
                  <span class="delete" @click="deleteScheduleItem(command.id)">Delete</span>
                </li>
              </ul>
              <div id="empty">{{ empty.emptySchedule }}</div>
              <div class="form-rows">
                <button
                  class="form-buttons-delete"
                  type="button"
                  @click="deleteFullSchedule"
                >Delete Full Schedule</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <NavbarBottom class="bottom-show" :userToken="userToken" />
  </div>
</template>
<script>
import NavbarTop from "../navbars/navbar-top";
import NavbarBottom from "../navbars/navbar-bottom";

export default {
  name: "addDevice",
  components: {
    NavbarTop,
    NavbarBottom
  },
  props: [
    "deviceID",
    "deviceName",
    "deviceImage",
    "deviceEnergy",
    "deviceType",
    "userToken"
  ],
  data() {
    return {
      form: {
        name: "",
        operation: "",
        month: "0",
        day: "0",
        hour: "",
        minute: ""
      },
      operations: [],
      device: "",
      scheduledCommands: [],
      empty: {
        emptySchedule: ""
      }
    };
  },
  methods: {
    //Add item to the schedule
    go(evt) {
      let url = "http://localhost:5552/insertRepeatTimer";
      console.log(this.form);
      fetch(url, {
        mode: "cors",
        method: "POST",
        headers: {
          Accept: "application/json",
          "Content-Type": "application/json",
          Authorization: this.userToken
        },
        body: JSON.stringify({
          type: "Day",
          month: this.form.month,
          day: this.form.day,
          hour: this.form.hour,
          minute: this.form.minute,
          command: this.form.operation,
          device_id: this.deviceID
        })
      })
        .then(response => {
          return response.json();
        })
        .then(jsonData => {
          console.log(jsonData);
          location.reload();
        });

      evt.preventDefault();
    },
    deleteScheduleItem(id) {
      //deletes the select scheduled item
      if (!confirm("Do really want to delete this scheduled event?")) {
        return false;
      }
      let url = "http://localhost:5552/deleteRepeatTimer?id=" + id;
      fetch(url, {
        mode: "cors",
        method: "GET",
        headers: {
          Authorization: this.userToken
        }
      })
        .then(response => {
          return response.json();
        })
        .then(jsonData => {
          console.log(jsonData);
          location.reload();
        });
    },
    deleteFullSchedule() {
      //deletes the the entire Schedule

      if (!confirm("Do really want to delete the entire schedule?")) {
        return false;
      }
      //loops through the schedule and deletes each schedule item one by one using the id
      for (let i in this.scheduledCommands) {
        let url =
          "http://localhost:5552/deleteRepeatTimer?id=" +
          this.scheduledCommands[i].id;
        fetch(url, {
          mode: "cors",
          method: "GET",
          headers: {
            Authorization: this.userToken
          }
        })
          .then(response => {
            return response.json();
          })
          .then(jsonData => {
            console.log(jsonData);
            location.reload();
          });
      }
    },
    formatTimeFront(time) {
      if (time < 10) {
        return "0" + time;
      }
      return time;
    }
  },
  mounted: function() {
    //Get command for device via icon thats displayed
    let url = "http://localhost:5552/getCommandsByDevice?id=" + this.deviceType;
    fetch(url, {
      mode: "cors",
      method: "GET",
      headers: {
        Authorization: this.userToken
      }
    })
      .then(response => {
        return response.json();
      })
      .then(jsonData => {
        this.operations = jsonData; //after we get commands find device ID
        console.log(this.operations);
        let url1 = "http://localhost:5552/getRepeatTimers?id=" + this.deviceID;

        fetch(url1, {
          mode: "cors",
          method: "GET",
          headers: {
            Authorization: this.userToken
          }
        })
          .then(response => {
            return response.json();
          })
          .then(jsonData => {
            console.log(jsonData);
            if (jsonData.length < 1) {
              this.empty.emptySchedule = "Schedule is Empty";
            }
            for (let key in jsonData) {
              if (
                jsonData[key].timer_repeat_command === 1 ||
                jsonData[key].timer_repeat_command === 3
              ) {
                jsonData[key].timer_repeat_command = "on";
              } else if (
                jsonData[key].timer_repeat_command === 2 ||
                jsonData[key].timer_repeat_command === 4
              ) {
                jsonData[key].timer_repeat_command = "off";
              }
              //creates a json of the each scheduled item and pushes to an array
              this.scheduledCommands.push({
                id: jsonData[key].timer_repeat_id,
                hour: formatTime(jsonData[key].timer_repeat_hour),
                minutes: formatTime(jsonData[key].timer_repeat_minute),
                command: capitalize(jsonData[key].timer_repeat_command)
              });
            }
            /*this.lastTime = {
              hour: jsonData[jsonData.length - 1].timer_repeat_hour,
              minutes: jsonData[jsonData.length - 1].timer_repeat_minute,
              command: jsonData[jsonData.length - 1].timer_repeat_command
            };*/
            console.log(this.scheduledCommands);
            this.scheduledCommands.sort((a, b) => {
              return a.hour - b.hour;
            });
          });
      });
  }
};

//Formating for the schedule
function capitalize(text) {
  return text.charAt(0).toUpperCase() + text.slice(1);
}

function formatTime(time) {
  if (time < 10) {
    return "0" + time;
  }
  return time;
}
</script>
