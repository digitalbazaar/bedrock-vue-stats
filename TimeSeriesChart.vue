<template>
  <div>
    <div
      v-if="realtime"
      class="row q-pa-lg">
      <q-select
        v-model="timeUnit"
        class="col-2"
        :options="times"
        @input="changeTime()" />
      <span
        class="col-1 q-mx-lg"
        @click="pause">
        <q-icon
          v-if="!paused"
          size="2rem"
          color="grey"
          name="fas fa-pause-circle">
          <q-tooltip
            class="q-ml-sm"
            anchor="center right"
            self="center left">
            <div>Pause {{label}}</div>
          </q-tooltip>
        </q-icon>
        <q-icon
          v-else
          size="2rem"
          color="grey"
          name="fas fa-play-circle">
          <q-tooltip
            class="q-ml-sm"
            anchor="center right"
            self="center left">
            <div>Resume {{label}}</div>
          </q-tooltip>
        </q-icon>
      </span>
    </div>
    <canvas
      ref="canvas"
      :height="height" />
  </div>
</template>

<script>
import Chart from 'chart.js';
import streaming from 'chartjs-plugin-streaming';

export default {
  Name: 'TimeSeriesChart',
  props: {
    realtime: {
      type: Boolean,
      default: () => false
    },
    label: {
      type: String,
      default: () => 'Label'
    },
    line: {
      type: String,
      default: () => 'black'
    },
    fill: {
      type: String,
      default: () => 'grey'
    },
    series: {
      type: Array,
      required: true,
      default: () => {
        return [];
      }
    },
    max: {
      type: Number,
      default: () => 1
    },
    min: {
      type: Number,
      default: () => 0
    },
    height: {
      type: Number,
      default: () => 50
    },
    point: {
      type: Object,
      default: () => ({radius: 5, hitRadius: 10})
    },
    duration: {
      type: Number,
      default: () => 60000
    }
  },
  data() {
    return {
      paused: false,
      chart: null,
      timeUnit: 'minute',
      units: [
        'millisecond', 'second', 'minute',
        'hour', 'day', 'week', 'month',
        'quarter', 'year'
      ]
    };
  },
  computed: {
    times() {
      const options = this.units.map(u => ({label: u, value: u}));
      if(!this.realtime) {
        return options;
      }
      return options.slice(1, 3);
    }
  },
  watch: {
    series(newSeries) {
      newSeries.forEach(p => {
        p.y = p.y.toFixed(2);
        this.chart.data.datasets[0].data.push(p);
      });
      this.chart.update();
    },
    max(newMax, oldMax) {
      if(newMax !== oldMax) {
        this.updateMax(newMax);
      }
    }
  },
  mounted() {
    const realtime = {
      type: 'realtime',
      realtime: {
        duration: this.duration,
        pause: false
      },
      time: {
        unit: this.timeUnit,
      }
    };
    const time = {
      type: 'time',
      realtime: false,
    };
    const myChart = new Chart(this.$refs.canvas, {
      type: 'line',
      plugins: this.realtime ? [streaming] : undefined,
      data: {
        datasets: [{
          label: this.label,
          data: this.series,
          borderColor: this.line,
          backgroundColor: this.fill,
          pointRadius: this.point.radius,
          pointHitRadius: this.point.hitRadius
        }]
      },
      options: {
        scales: {
          xAxes: [this.realtime ? realtime : time],
          yAxes: [
            {
              ticks: {
                min: this.min,
                max: this.max,
                stepSize: 1
              }
            }
          ]
        },
        animation: {
          duration: 40
        },
        plugins: {
          streaming: this.realtime ? {frameRate: 10} : false
        }
      }
    });
    this.chart = myChart;
    this.updateMax(this.max);
  },
  methods: {
    pause() {
      const [axis] = this.chart.options.scales.xAxes;
      this.paused = !this.paused;
      axis.realtime.pause = this.paused;
      this.chart.update();
    },
    calcTimeStep() {
      const maxTicks = 25;
      const diff = this.series[this.series.length - 1].x - this.series[0].x;
      const second = 1000;
      const minute = second * 60;
      const hour = minute * 60;
      const day = hour * 24;
      const week = day * 7;
      const month = week * 4;
      const year = month * 12;
      const times = {
        millisecond: 1,
        second,
        minute,
        hour,
        day,
        week,
        month,
        year
      };
      let step = 1;
      const time = times[this.timeUnit];
      if(time) {
        const units = Math.ceil(diff / time);
        step = Math.ceil(units / maxTicks);
      }
      return step;
    },
    changeTime() {
      const [axis] = this.chart.options.scales.xAxes;
      axis.time.unit = this.timeUnit;
      axis.time.stepSize = this.calcTimeStep();
      this.chart.update();
    },
    updateMax(max) {
      const [axis] = this.chart.options.scales.yAxes;
      axis.ticks.max = max;
      axis.ticks.stepSize = Math.ceil(max / 4);
      this.chart.update();
    }
  }
};
</script>

<style scoped>
.q-tooltip {
  line-height: 0.2rem;
}
</style>
