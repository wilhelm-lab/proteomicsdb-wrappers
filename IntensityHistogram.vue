<template>
  <histogram
    v-if="chartData"
    id="intensityHistogram"
    :chartData="chartData"
    :selectedLines="selectedLines"
    :minHeight="minHeight"
    :minWidth="minWidth"
    :title="title"
    :xlabel="xlabel"
  />
</template>

<script>
import axios from "axios";
import histogram from "@/vue-d3-components/GenericHistogram";

export default {
  name: "intensityhistogram",
  components: {
    histogram: histogram,
  },
  props: {
    minWidth: {
      type: Number,
      default: 200,
    },
    minHeight: {
      type: Number,
      default: 200,
    },
    title: {
      type: String,
      default: "",
    },
    xlabel: {
      type: String,
      default: "",
    },
    sModelIds: {
      type: String,
      default: "",
    },
    selectedDataset: {
      type: Object,
      default: null,
    },
    selectedLines: {
      type: Array,
      default: null,
    },
  },
  data: function () {
    return {
      chartData: null,
    };
  },
  watch: {
    sModelIds: function () {
      this.getData();
    },
    selectedDataset: function () {
      this.getData();
    },
  },
  methods: {
    getData: function () {
      let that = this;
      let urlCurveData =
        this.$store.state.host +
        "/proteomicsdb/logic/ptmCurves/getIntensityData.xsjs";
      if (this.selectedDataset) {
        axios
          .get(urlCurveData, {
            params: {
              experiment_design_id: this.selectedDataset.experimentDesignId,
            },
          })
          .then(function (response) {
            that.chartData = response.data;
          });
      } else {
        this.chartData = null;
      }
    },
  },
};
</script>
