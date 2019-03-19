<template>
  <div class="relative gauge-holder">
    <canvas
      ref="chart"
      :data="chartData" />
    <div class="absolute-center text-center">
      <p class="percent-holder">
        <span
          :style="style"
          class="gauge-percent">{{percentage}}<sup class="percent-sup">%</sup>
        </span>
      </p>
      <p
        class="gauge-info-line">
        {{used}} of {{max}} {{unit}}
      </p>
    </div>
  </div>
</template>

<script>
import Chart from 'chart.js';

export default {
  Name: 'GaugeChart',
  props: {
    labels: {
      type: Array,
      default() {
        return ['Used', 'Available'];
      }
    },
    title: {
      type: String,
      required: true
    },
    color: {
      type: String,
      required: true
    },
    last: {
      type: Number,
      default: () => 0.5
    },
    max: {
      type: String,
      default: () => ''
    },
    unit: {
      type: String,
      default: () => 'GB'
    }
  },
  data() {
    return {
      chart: null
    };
  },
  computed: {
    style() {
      return {
        color: this.color
      };
    },
    used() {
      return Math.ceil(this.max * this.last);
    },
    percentage() {
      return Math.ceil(this.last * 100);
    },
    chartData() {
      return this.setChartData(this.last);
    }
  },
  mounted() {
    this.chart = new Chart(this.$refs.chart, {
      type: 'doughnut',
      data: {
        labels: this.labels,
        datasets: [{
          data: [0.5, 0.5],
          backgroundColor: [this.color, 'rgba(228,223,223)']
        }]
      },
      options: {
        circumference: Math.PI,
        rotation: Math.PI,
        cutoutPercentage: 70, // percent
        legend: {
          display: false
        },
        tooltips: {
          enabled: false
        },
        title: {
          display: true,
          text: this.title
        }
      }
    });
    this.setChartData(this.last);
  },
  methods: {
    setChartData(latest) {
      if(!latest || !this.chart) {
        return false;
      }
      this.last = Number(latest).toFixed(2);
      const next = [this.last, (1 - this.last).toFixed(2)];
      this.chart.data.datasets[0].data = next;
      this.chart.update();
    }
  }
};
</script>

<style scoped>
.relative {
  position: relative;
}

.absolute-center {
  position:absolute;
  top: 85%;
  left: 50%;
  transform: translate(-50%, -50%);
}

p {
  font-size:  1rem;
  line-height: 0.25rem;
}

.gauge-percent {
  font-size: 2.5rem;
  margin-bottom: 1.5rem;
}

.percent-sup {
  color: black;
  font-size: 1rem;
}
</style>
