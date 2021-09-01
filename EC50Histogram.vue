<template>
  <histogram 
    v-if="chartData"
    id="ec50Histogram"
    :chartData="chartData"
    :plotHistogram="false"
    :plotKDE="true"
    :selectedLines="selectedLines"
    :minHeight="minHeight"
    :minWidth="minWidth"
    :title="title"
    :xlabel="xlabel"
    />
</template>

<script>
import axios from 'axios'
import histogram from '@/vue-d3-components/GenericHistogram'

export default {
  name: 'ec50histogram',
  components: {
    histogram: histogram
  },
  props: {
    minWidth: {
        type: Number,
        default: 200
    },
    minHeight: {
        type: Number,
        default: 200
    },
    title: {
        type: String,
        default: ''
    },
    xlabel: {
        type: String,
        default: ''
    },
    sModelIds: {
        type: String,
        default: ''
    },
    selectedDataset: {
        type: Object,
        default: null
    },
    selectedLines: {
        type: Array,
        default: null
    },
    minDose: {
        type: Number,
        default: 1e-20
    },
    maxDose: {
        type: Number,
        default: 1e20
    }
  },
  data: function () {
    return {
      chartData: null
    }
  },
  watch: {
    sModelIds: function () {
      this.getData()
    },
    selectedDataset: function () {
      this.getData()
    }
  },
  methods: {
    getData: function () {  
      let that = this
      let urlCurveData = this.$store.state.host + '/proteomicsdb/logic/ptmCurves/getEC50Data.xsjs'
      if (this.selectedDataset) {
        axios.get(urlCurveData, {
          params: {
            experiment_design_id: this.selectedDataset.experimentDesignId,
            min_dose: this.selectedDataset.minDose * this.getDoseUnitMultiplier(this.selectedDataset),
            max_dose: this.selectedDataset.maxDose * this.getDoseUnitMultiplier(this.selectedDataset)
          }})
          .then(function (response) {
            that.chartData = response.data
          })
      } else {
        this.chartData = null
      }
    },
    getDoseUnitMultiplier: function(dataset) {
      if (dataset.doseUnit === "nanomolar") {
        return 1e-9;
      } else {
        return 1.0;
      }
    }
  }
}
</script>
