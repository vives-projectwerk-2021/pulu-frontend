<template>
  <div>
    <div class="d-flex justify-space-between pa-5">
      <h2>
        Live data: {{ liveValues.time.substr(0, 10) }}
        {{ liveValues.time.substr(11, 5) }}
      </h2>
      <v-btn @click="fold" elevation="0" fab plain small>
        <v-icon>mdi-chevron-down</v-icon>
      </v-btn>
    </div>

    <v-container>
      <v-row>
        <v-col cols="12" sm="6" md="3" v-for="(moisture, index) in liveValues.sensors.moisture" :key="moisture.key">
          <MoistureCard
            :percentage="Math.round(moisture.value*10)/10"
            :level="index"
          >
          </MoistureCard>
        </v-col>
        <v-col  cols="12" sm="6" md="3"  v-for="(temp, index) in liveValues.sensors.temperature" :key="temp.key">
          <SensorValueCard
            sensor=" Temperature"
            :value="Math.round(temp.value*10)/10"
            unit="°C"
            :level="index"
            icon="mdi-thermometer"
            color="red"
          >
          </SensorValueCard>
        </v-col>
        <v-col  cols="12" sm="6" md="3" >
          <SensorValueCard
            sensor=" Light intensity"
            :value="Math.round(liveValues.sensors.light.value*10)/10"
            unit="Lumens"
            icon="mdi-brightness-4"
            color="yellow"
          >
          </SensorValueCard>
        </v-col>
        <v-col cols="12" sm="6" md="3" >
          <SensorValueCard
            sensor=" Battery level"
            :value="Math.round(liveValues.sensors.voltage.battery.value*10)/10"
            unit="%"
            icon="mdi-battery"
            color="light-green"
          >
          </SensorValueCard>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import MoistureCard from "@/components/MoistureCard";
import SensorValueCard from "@/components/SensorValueCard";
export default {
  name: "LiveData",
  props: ["liveValues"],
  components: {
    MoistureCard,
    SensorValueCard,
  },
  data() {
    return {
      displayContent: true,
    };
  },
  methods: {
    fold() {
      this.displayContent = !this.displayContent;
    },
  },
};
</script>
