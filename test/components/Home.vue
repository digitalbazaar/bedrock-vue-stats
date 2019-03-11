<template>
  <q-page
    class="column gutter-md background"
    padding>
    <div class="row">
      <h4>Gauge Chart</h4>
      <span class="col-4">
        <br-gauge-chart
          title="CPU"
          :color="colors().cpu"
          :max="charts.maxCPU"
          unit="CPUS"
          :last="charts.lastCPU" />
      </span>
    </div>
    <div>
      <h4>Static Line Chart</h4>
    <br-time-series-chart
      id="load-avg"
      :line="colors().line"
      :fill="colors(0.8).cpu"
      :max="charts.maxCPU"
      :series="charts.loadavg"
      label="CPU Usage" />
    </div>
    <div>
      <h4>Real time Line Chart </h4>
    <br-time-series-chart
      id="fs-used"
      :line="colors().line"
      :fill="colors(0.8).disk"
      :max="charts.maxDISK"
      :series="charts.fssize"
       realtime
      label="Disk Space in GB" />
    </div>
  </q-page>
</template>
<script>
/*!
 * Copyright (c) 2019 Digital Bazaar, Inc. All rights reserved.
 */
'use strict';

// FIXME: chartjs is loaded as a global in index.js as a hack
import {StatsService} from 'bedrock-web-stats';
import {BrGaugeChart, BrTimeSeriesChart} from 'bedrock-vue-stats';

export default {
  name: 'Home',
  components: {BrGaugeChart, BrTimeSeriesChart},
  data() {
    return {
      charts: {last: {}},
    };
  },
  beforeCreate() {
    this._statsService = new StatsService({poll: 2000});
  },
  mounted() {
    this._statsService.subscribe({id: 'demo', updater: this.subscriber});
  },
  methods: {
    colors(alpha = 1) {
      return {
        ram: `rgba(0, 0, 184, ${alpha})`,
        cpu: `rgba(100, 199, 85, ${alpha})`,
        disk: `rgba(200, 44, 146, ${alpha})`,
        line: `rgba(100,100,100, ${alpha})`
      };
    },
    subscriber(charts) {
      if(!charts) {
        return false;
      }
      this.$set(this, 'charts', charts);
    }
  }
};
</script>
<style>
main.background {
  background-color: 'black';
}
</style>
