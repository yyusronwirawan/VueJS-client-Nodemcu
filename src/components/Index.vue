<template>
  <div>
    <canvas id="planet-chart"></canvas>
    <!-- {{ this.data.datasets[0].data }} -->
  </div>
</template>

<script>
import Chart from 'chart.js'

export default {
  name: 'Index',
  data: function(){
    return{
      connection: null,
      // data_sensor: [1],
      type: "line",
      data: {
        labels: [],
        datasets: [
          {
            label: "Data Sensor",
            data: [],
            backgroundColor: "rgba(71, 183,132,.5)",
            borderColor: "#47b784",
            borderWidth: 3
          }
        ]
      },
      options: {
        responsive: true,
        lineTension: 1,
        scales: {
          yAxes: [
            {
              ticks: {
                beginAtZero: true,
                padding: 25
              }
            }
          ]
        }
      }
    }
  },
  created: function() {    
    const date = new Date();
    this.data.datasets[0].label = "Data Sensor " + date.toLocaleDateString('en-GB', { timeZone: 'Asia/Jakarta' });    
  },
  mounted() {
    const ctx = document.getElementById('planet-chart');
    const chart = new Chart(ctx, {
      type: this.type,
      data: this.data,
      options: this.options,
    });

    console.log("Starting connection to WebSocket Server")
    this.connection = new WebSocket("ws://localhost:8080")

    this.connection.onopen = (event) => {      
      console.log(event)
      console.log("Successfully connected to the echo websocket server...")
      const messageObject = {
        device_type: "pc-client",
      };    
      this.connection.send(JSON.stringify(messageObject));
    }

    this.connection.onmessage = (event) => {
      const date = new Date();      
      let time_now = date.toLocaleTimeString('en-GB', { timeZone: 'Asia/Jakarta' });
      // console.log(event)
      // this.data_sensor.push(parseInt(event.data));
      if(this.data.datasets[0].data.length >= 13 && this.data.labels.length >= 13){        
        this.data.datasets[0].data.shift();
        this.data.labels.shift();
        this.data.datasets[0].data.push(parseInt(event.data));   
        this.data.labels.push(time_now);
      }      
      else{
        this.data.datasets[0].data.push(parseInt(event.data));   
        this.data.labels.push(time_now);
      }
      chart.update();      
      // console.log(this.datasets);
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
