<template>
  <q-page
    v-if="!loading"
    class="column gutter-md background"
    padding>
    <div class="row">
      <h4>Gauge Chart</h4>
      <span class="col-4">
        <br-gauge-chart
          title="CPU"
          :color="colors().cpu"
          :max="cpu.chart.max"
          unit="CPUS"
          :last="cpu.chart.last" />
      </span>
    </div>
    <div>
      <h4>Real time Line Chart </h4>
      <br-time-series-chart
        id="mem-used"
        :line="colors().line"
        :fill="colors(0.8).ram"
        :max="ramseries.chart.max"
        :series="ramseries.chart.series"
        realtime
        label="RAM Usage GB" />
    </div>
    <div>
      <h4>Static Line Chart</h4>
      <br-time-series-chart
        id="static-chart"
        :line="colors().line"
        :fill="colors(0.8).disk"
        :max="8"
        :series="staticSeries"
        label="Static Example" />
    </div>
  </q-page>
</template>
<script>
/*!
 * Copyright (c) 2019 Digital Bazaar, Inc. All rights reserved.
 */
'use strict';

// FIXME: chartjs is loaded as a global in index.js as a hack
import {statsService, ChartController} from 'bedrock-web-stats';
import {BrGaugeChart, BrTimeSeriesChart} from 'bedrock-vue-stats';
import staticSeries from '../mocks/staticSeries.json';
import units from './units';

export default {
  name: 'Home',
  components: {BrGaugeChart, BrTimeSeriesChart},
  data() {
    return {
      staticSeries,
      charts: {last: {}},
      cpu: null,
      ramseries: null
    };
  },
  computed: {
    loading() {
      if(!this.cpu) {
        return true;
      }
      return this.cpu.loading;
    }
  },
  mounted() {
    this.$set(this, 'cpu', new ChartController(
      {
        type: 'pie',
        statsService,
        format: {
          prepare({latest}) {return latest.monitors.os.currentLoad;},
          last(p) {return p.avgload;},
          max(p) {return p.cpus.length;}
        }}));
    this.$set(this, 'ramseries', new ChartController(
      {
        type: 'realtime',
        statsService,
        format: {
          prepare({latest}) {return latest.monitors.os.mem;},
          y(p) {return p.active / units.gb;},
          max(p) {return p.total / units.gb;}
        }}));

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
