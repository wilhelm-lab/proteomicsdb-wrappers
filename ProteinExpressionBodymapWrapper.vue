<template>
  <v-card style="height:750px;max-height:750px;" class="pt-2">
  <bodymap id='expressionBodymap'
    :data="bodyMapData"
    :selectedOrganism="selectedOrganism"
    @organSelected="organSelected"
    ref="bodymap"
  />
  </v-card>
</template>

<script>
import axios from 'axios';
import bodymap from '@/vue-d3-components/Bodymap';

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
    minHeight: {
      type: Number,
      default: 400
    },
    proteinId: {
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
    }
  },
  mounted() {
    this.getData();
  }
}
</script>
