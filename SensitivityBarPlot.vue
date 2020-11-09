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
import axios from 'axios'
import barplot from '@/vue-d3-components/GenericBarPlot'

export default {
  name: 'sensitivitybarplot',
  components: {
    barplot: barplot
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
    selectedDataset: {
      type: Object,
      default: null
    },
    selectedDrugs: {
      type: String,
      default: ''
    },
    selectedTissues: {
      type: String,
      default: ''
    },
    selectedKey: {
      type: String,
      default: ''
    },
    selectedLinesForBarPlot: {
      type: String,
      default: ''
    }
  },
  data: function () {
    return {
      barPlotModel: null
    }
  },
  computed: {
    barPlotTitle: function () {
      return this.selectedKey + "  values for selected drug/cell line combination"
    }
  },
  watch: {
    selectedDataset: function () {
      this.getData()
    },
    selectedDrugs: function () {
      this.getData()
    },
    selectedTissues: function () {
      this.getData()
    },
    selectedKey: function () {
      this.getData()
    },
    selectedLinesForBarPlot: function () {
      this.getData()
    }
  },
  methods: {
    getData: function () {  
      if (this.selectedKey) {
        let that = this
        
        let urlDatasets = '/proteomicsdb/logic/cellSelectivity/getDataForVioPlots.xsjs'
        axios.get(urlDatasets, {
          params: {
            dataset_id: that.selectedDataset.datasetId,
            drug_id: that.selectedDrugs,
            cell_line_id: that.selectedTissues,
            BIC: 100,
            R2: 0,
            Effect: 0,
            Switch: 1,
            model_ids: that.selectedLinesForBarPlot
          }})
          .then(function (response) {
            var selectedKeyBarData = response.data.filter(function(ViolinModel) {
                return ViolinModel.AttributeType === that.selectedKey;
              })[0];
            that.barPlotModel = selectedKeyBarData
          })
      } else {
        this.barPlotModel = null
      }
    },
    onBarSelected: function (obj) {
      this.$emit('send-message', obj)
    },
  }
}
</script>
