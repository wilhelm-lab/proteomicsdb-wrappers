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
import axios from 'axios'
import parallelplot from '@/vue-d3-components/D3ParallelGeneric'

export default {
  name: 'sensitivityparallelplot',
  components: {
    parallelplot: parallelplot
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
    }
  },
  data: function () {
    return {
      violinModel: null
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
    }
  },
  methods: {
    getData: function () {
      if (this.selectedDrugs && this.selectedTissues && !(this.selectedDrugs === '-1' && this.selectedTissues === 'not')) {
        let that = this
        let urlDatasets = this.$store.state.host+'/proteomicsdb/logic/cellSelectivity/getDataForVioPlots.xsjs'

        axios.get(urlDatasets, {
          params: {
            dataset_id: that.selectedDataset.datasetId,
            drug_id: that.selectedDrugs,
            cell_line_id: that.selectedTissues,
            BIC: 100,
            R2: 0,
            Effect: 0,
            Switch: 1
          }})
          .then(function (response) {
            that.violinModel = response.data
            that.$emit('modelUpdate', that.violinModel)
          })
      } else {
        this.violinModel = null
        this.$emit('modelUpdate', null)
        this.$emit('keyChange', null, null)
      }
    },
    keyChange: function (selectedLinesForBarPlot, selectedKey) {
      this.$emit('keyChange', selectedLinesForBarPlot, selectedKey)
    },
  }
}
</script>
