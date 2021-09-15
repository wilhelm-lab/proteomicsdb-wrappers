<template>
  <parallelplot
    v-if="violinModel"
    :chartData="violinModel"
    :plotWidth="minWidth"
    :plotHeight="minHeight"
    @keyChange="keyChange"
  />
</template>

<script>
import parallelplot from "@/vue-d3-components/D3ParallelGeneric";

export default {
  name: "sensitivityparallelplot",
  components: {
    parallelplot: parallelplot,
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
    violinModel: {
      type: Array,
      default: () => [],
    },
  },
  methods: {
    keyChange: function (selectedLinesForBarPlot, selectedKey) {
      this.$emit("keyChange", selectedLinesForBarPlot, selectedKey);
    },
  },
};
</script>
