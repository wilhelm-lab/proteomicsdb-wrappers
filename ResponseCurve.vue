<template>
  <lineplot 
    v-if="responseCurveModel"
    id="ic50Plot"
    :curveParameters="responseCurveModel.parameter"
    :dataPoints="responseCurveModel.data"
    :minHeight="minHeight"
    :minWidth="minWidth"
    :title="title"
    :properties="responseCurveModel.properties"
    @update-curve-styles="updateCurveStyles"
    v-on="$listeners"
    />
</template>

<script>
import axios from 'axios'
import lineplot from '@/vue-d3-components/GenericLinePlot'

export default {
  name: 'responsecurve',
  components: {
    lineplot: lineplot
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
    sModelIds: {
        type: String,
        default: ''
    },
    selectedDataset: {
        type: Object,
        default: null
    },
    dataType: {
        type: String,
        default: 'cellSelectivity'
    }
  },
  data: function () {
    return {
      responseCurveModel: null
    }
  },
  watch: {
    sModelIds: function () {
      this.getData()
    },
    datasetId: function () {
      this.getData()
    }
  },
  methods: {
    getData: function () {  
      let that = this
      let urlCurveData = this.$store.state.host+'/proteomicsdb/logic/' + this.dataType + '/getCurveData.xsjs'
      if (this.sModelIds !== '' && this.selectedDataset) {
        axios.get(urlCurveData, {
          params: {
            dr_model_ids: this.sModelIds,
            dataset_id: this.selectedDataset.datasetId
          }})
          .then(function (response) {
            that.responseCurveModel = response.data
          })
      } else {
        this.responseCurveModel = null
      }
    },
    updateCurveStyles: function (oCurveStyleMapping) {
      this.$emit('update-curve-styles', oCurveStyleMapping)
    }
  }
}
</script>
