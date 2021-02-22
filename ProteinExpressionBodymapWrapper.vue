<template>
  <v-card :height="height" :max-height="maxHeight">
    <v-toolbar v-if="showToolbar" :color="color" dark class="mb-2" elevation="2">
      <v-card-title>Protein Expression</v-card-title>
    </v-toolbar>
  <bodymap id='expressionBodymap'
    :data="bodyMapData"
    :selectedOrganism="selectedOrganism"
    @organSelected="organSelected"
    ref="bodymap"
    class="pt-2"
  />
  </v-card>
</template>

<script>
import axios from 'axios';
import bodymap from '@/vue-d3-components/Bodymap';
import Helper from '@/vue-d3-component-wrappers/common-lib/Helper.js';

export default {
  name: 'proteinExpressionBodymap',
  components: {
    bodymap
  },
  props: {
    minWidth: {
      type: Number,
      default: 400
    },
    maxHeight: {
      type: String,
      default: '750px'
    },
    height: {
      type: String,
      default: '750px'
    },
    showToolbar: {
      type: Boolean,
      default: false
    },
    color: {
      type: String,
      default: 'black'
    },
    proteinId: {
      type: String,
      default: null
    },
    proteinAccession: {
      type: String,
      default: null
    },
    omicsType: {
      type: String,
      default: null
    },
    quantification: {
      type: String,
      default: null
    },
    tissue_category: {
      type: String,
      default: null
    },
    scope: {
      type: String,
      default: null
    },
    group_by_tissue: {
      type: String,
      default: null
    },
    calculation: {
      type: String,
      default: null
    },
    selectedOrganism: {
      type: Object,
      default: () => {}
    }
  },
  data: function () {
    return {
      bodyMapData: []
    };
  },
  computed: {
  },
  watch: {
    omicsType: function() {
      if (this.tissue_category && this.tissue_category !== '') {
        this.getData();
      }
    },
    calculation: function() {
      if(this.tissue_category && this.tissue_category !== '') {
        this.getData();
      }
    },
    tissue_category: function() {
      if (this.tissue_category && this.tissue_category !== '') {
        this.getData();
      }
    }

  },
  methods: {
    getData: function () {  
      let that = this;

      let urlDatasets = 'https://www.proteomicsdb.org/proteomicsdb/logic/getExpressionWrapper.xsjs'
      axios.get(urlDatasets, { params: {
          omics: that.omicsType,
          protein_id: that.proteinId,
          quantification: that.quantification,
          tissue_category: that.tissue_category,
          scope: that.scope,
          group_by_tissue: that.group_by_tissue,
          calculation: that.calculation
      }})
      .then(function (response) {
        that.bodyMapData = response.data;
        that.$emit('dataLoaded', response.data);
      })
    },
    toggleOrgan: function (name) {
      this.$refs.bodymap.toggleOrgan(name);
    },
    organSelected: function (data) {
      this.$emit('organSelected', data)
    },
    getSVG: function () {
      return this.$refs.bodymap.getSVG();
    },
    getCSV: function () {
      var aColumnMapping = [];
      var title = '';
      if (this.omicsType === 'Proteomics') {
        aColumnMapping = [{
          jsonProperty: 'TISSUE_ID',
          columnName: 'Tissue ID'
        }, {
          jsonProperty: 'TISSUE_NAME',
          columnName: 'Tissue'
        }, {
          jsonProperty: 'SAP_SYNONYM',
          columnName: 'Tissue Synonym'
        }, {
          jsonProperty: 'UNNORMALIZED_INTENSITY',
          columnName: 'Average Unnormalized Intensity'
        }, {
          jsonProperty: 'NORMALIZED_INTENSITY',
          columnName: 'Average Normalized Intensity'
        }, {
          jsonProperty: 'MAX_NORMALIZED_INTENSITY',
          columnName: 'Maximum Normalized Intensity'
        }, {
          jsonProperty: 'MIN_NORMALIZED_INTENSITY',
          columnName: 'Minimum Normalized Intensity'
        }, {
          jsonProperty: 'SAMPLES',
          columnName: 'Number of Samples'
        }];
        title = 'Protein Expression: ' + this.proteinAccession;
      } else {
        aColumnMapping = [{
          jsonProperty: 'TISSUE_ID',
          columnName: 'Tissue ID'
        }, {
          jsonProperty: 'TISSUE_NAME',
          columnName: 'Tissue'
        }, {
          jsonProperty: 'SAP_SYNONYM',
          columnName: 'Tissue Synonym'
        }, {
          jsonProperty: 'NORMALIZED_INTENSITY',
          columnName: 'Average ' + this.calculation
        }, {
          jsonProperty: 'MAX_NORMALIZED_INTENSITY',
          columnName: 'Maximum ' + this.calculation
        }, {
          jsonProperty: 'MIN_NORMALIZED_INTENSITY',
          columnName: 'Minimum ' + this.calculation
        }, {
          jsonProperty: 'NUMBER_OF_SAMPLES',
          columnName: 'Number of Samples'
        }];
        title = 'mRNA Expression: ' + this.proteinAccession;
      }
      return Helper.downloadCsvFromJson(this.bodyMapData, title, aColumnMapping);
    }
  },
  mounted() {
    this.getData();
  }
}
</script>
