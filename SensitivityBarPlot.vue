<template>
  <barplot
    v-if="barPlotModel"
    id="ic50Bar"
    :data="barPlotModel"
    :multiSelection="true"
    :minWidth="minWidth"
    :minHeight="minHeight"
    :title="barPlotTitle"
    @send-message="onBarSelected"
  />
</template>

<script>
import barplot from "@/vue-d3-components/GenericBarPlot";

export default {
  name: "sensitivitybarplot",
  components: {
    barplot: barplot,
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
    selectedKey: {
      type: String,
      default: "",
    },
    violinModel: {
      type: Array,
      default: () => [],
    },
  },
  computed: {
    barPlotModel: function () {
      if (this.selectedKey) {
        return this.violinModel.filter(
          (model) => model.AttributeType === this.selectedKey
        )[0];
      } else {
        return null;
      }
    },

    barPlotTitle: function () {
      return (
        this.selectedKey + "  values for selected drug/cell line combination"
      );
    },
  },
  methods: {
    onBarSelected: function (obj) {
      this.$emit("send-message", obj);
    },
  },
};
</script>
