<template>
    <div class="Hero">
        <div class="threecanvas">
            <Landscape :hour="{hour}" :minute="{minute}" :gmthour="{gmthour}" :gmtmin="{gmtmin}" :time="{time}" :sign="{sign}"/>
        </div>
    </div>
</template>

<script>
import Landscape from '@/components/subs/Landscape.vue'
import {data} from './../data/timezones.js'
export default {
  name: 'TimeDisplay',
  components: {
    Landscape
  },
  data() {
      return {
        intervalId:null,
        time:0,
        gmt:0,
        hour:0,
        minute:0,
        gmthour:0,
        gmtmin:0,
        randomMinute:0,
        sign:'+'
      }
  },
  methods: {
    updateTime() {
      //get random value 
      let currentHour = new Date().getUTCHours();
      let currentMinute = new Date().getUTCMinutes();
      let index = Math.floor(Math.random()*24);
      let timezoneobj = data[index];
      this.generateRandomMin(timezoneobj);
      //set gmt time 
      this.gmthour = timezoneobj.offset;
      this.gmtmin = this.randomMinute;

      //set the current time
     this.hour = currentHour + parseInt(timezoneobj.offset);
     if(this.hour < 0) {
       this.hour = 24 + this.hour;
     }
      this.minute = currentMinute + parseInt(this.randomMinute);
      if(this.minute > 60) {
        this.minute = this.minute - 60;
        this.hour = this.hour +1;
      }
      this.hour = this.hour%24;
      if(this.gmthour < 0) {
        this.gmthour = Math.abs(this.gmthour);
        this.sign = "-"
      } else {
        this.sign = '+'
      }
      this.time++;
    },
    generateRandomMin(timeObj) {
      this.randomMinute = Math.floor(Math.random()*60);
      for(let i=0;i<timeObj.minsEx.length;i++) {
        if(this.randomMinute == timeObj.minsEx[i]) {
          this.generateRandomMin()
          break;
        }
      }
    },
    fetchTime() {
      this.intervalId = setInterval(() => {
        this.updateTime()
      },60000)
    }
  },
  created() {
    //console.log(data);
    this.fetchTime();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.Hero {
    height: 100%;
    width: 100%;
    z-index: -1;
    background: black;
}
.threecanvas {
    z-index: 0;
    position: absolute;
    margin-top: -100px;
}
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
