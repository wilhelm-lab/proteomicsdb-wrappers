<template>
  <v-container fluid>
    <v-row>
      <v-col cols="3">
        <v-card class="ma-1">
          <v-tabs
                v-model="UiModel.tabIndex"
                grow
                dense
                dark
                :background-color="$store.state.selectedOrganismShown.secondaryColor"
                >
                <v-tabs-slider :color="$store.state.selectedOrganismShown.primaryColor"></v-tabs-slider>
            <v-tab href="#tab-0">
              Relations
            </v-tab>
            <v-tab href="#tab-1" :disabled="!UiModel.visibleNode">
              Node information
            </v-tab>
            <v-tab href="#tab-2" :disabled="!UiModel.visibleRetrieval">
              Options
            </v-tab>
            <v-tabs-items v-model="UiModel.tabIndex">
              <v-tab-item :value="'tab-0'">
                <v-treeview
                          :items="RelationTypesModel" 
                          item-children="Children"
                          item-text="Label"
                          return-object
                          dense style="overflow-y: auto; max-height:500px"
                                >
                                <template v-slot:prepend="{ item }">
                                  <v-row>
                                    <div v-html="item.svg"></div>
                                    <v-checkbox v-if="item.Description" v-model="item.selected" :value="item.selected" :label="item.Name" dense class="mt-0 pt-0" @change="modifiedRelationSelection = true"></v-checkbox>
                                  </v-row>
                                </template>
                </v-treeview>
                <v-col align="middle">
                  <v-btn :color="$store.state.selectedOrganismShown.primaryColor" :loading="loading" :disabled="loading" class="ma-2 white--text" @click="runAnalysis">
                    Run analysis
                    <v-icon right>fas fa-running</v-icon>
                  </v-btn>
                </v-col>
              </v-tab-item>
              <v-tab-item :value="'tab-1'">
                <v-expansion-panels multiple dense  style="overflow-y: auto; max-height:500px" class="pa-1">
                  <v-expansion-panel v-for="(item, i) in NodeModel.nodes"
                                     :key="i"
                                     class="ma-1"
                                     color="secondary">
                    <v-expansion-panel-header>{{item.Label}}</v-expansion-panel-header>
                    <v-expansion-panel-content>
                      <v-divider/>
                        <v-list dense>
                          <v-list-item v-for="(child, j) in item.Children" :key="j">
                            <v-list-item-title>{{child.ResourceType}}</v-list-item-title>
                            <v-list-item-subtitle class="text-right">{{child.AccessionNumber}}</v-list-item-subtitle>
                          </v-list-item>
                          <v-divider/>
                            <v-list-item>
                              <v-list-item-title>Protein information</v-list-item-title>
                              <v-tooltip top>
                                <template v-slot:activator="{ on, attrs }">
                                  <v-btn :color="$store.state.selectedOrganismShown.primaryColor" outlined fab small class="ma-1" v-bind="attrs" v-on="on" @click="proteinLinkFormatter(item.UniProtName)"><v-icon>fas fa-info</v-icon></v-btn>
                                </template>
                                <span>Information</span>
                              </v-tooltip>
                              <v-tooltip top>
                                <template v-slot:activator="{ on, attrs }">
                                  <v-btn :disabled="!item.Expression" :color="$store.state.selectedOrganismShown.primaryColor" outlined fab small class="ma-1" @click="proteinLinkExpressionFormatter(item.UniProtName)" v-bind="attrs" v-on="on"><v-icon>fas fa-street-view</v-icon></v-btn>
                                </template>
                                <span>Expression</span>
                              </v-tooltip>
                              <v-tooltip top>
                                <template v-slot:activator="{ on, attrs }">
                                  <v-btn :disabled="!item.BC" :color="$store.state.selectedOrganismShown.primaryColor" outlined fab small class="ma-1" v-bind="attrs" v-on="on"  @click="proteinLinkBcFormatter(item.UniProtName)">BC</v-btn>
                                </template>
                                <span>Biochemical Assays</span>
                              </v-tooltip>
                              <v-tooltip top>
                                <template v-slot:activator="{ on, attrs }">
                                  <v-btn :color="$store.state.selectedOrganismShown.primaryColor" outlined fab small class="ma-1" v-bind="attrs" v-on="on" @click="proteinLinkNodeFormatter(item.UniProtName)"><v-icon>mdi mdi-apache-kafka</v-icon></v-btn>
                                </template>
                                <span>Interaction Network</span>
                              </v-tooltip>
                            </v-list-item>
                            <v-divider/>
                              <v-list-item>
                                <v-list-item-title>Protein analytics</v-list-item-title>
                                <v-tooltip top>
                                  <template v-slot:activator="{ on, attrs }">
                                    <v-btn :disabled="!item.CurveFit" :color="$store.state.selectedOrganismShown.primaryColor" outlined fab small class="ma-1" v-bind="attrs" v-on="on"  @click="proteinLinkDsFormatter(item.UniProtName)"><v-icon>mdi mdi-distribute-horizontal-center</v-icon></v-btn>
                                  </template>
                                  <span>Dose Selectivity</span>
                                </v-tooltip>
                                <v-tooltip top>
                                  <template v-slot:activator="{ on, attrs }">
                                    <v-btn :disabled="!item.CurveFit" :color="$store.state.selectedOrganismShown.primaryColor" outlined fab small class="ma-1" v-bind="attrs" v-on="on" @click="proteinLinkDsFormatter(item.UniProtName)">CT</v-btn>
                                  </template>
                                  <span>Combination Treatment</span>
                                </v-tooltip>
                              </v-list-item>
                            </v-list>
                          </v-expansion-panel-content>
                        </v-expansion-panel>
                      </v-expansion-panels>
                      <v-list-item class="mx-10 py-5">
                        <v-list-item-title>Combined analytics</v-list-item-title>
                        <v-tooltip top>
                          <template v-slot:activator="{ on, attrs }">
                            <v-btn :disabled="!NodeModel.bEnableHeatmap" :color="$store.state.selectedOrganismShown.primaryColor" outlined fab small class="ma-1" v-bind="attrs" v-on="on" @click="heatmapLinkFormatter(NodeModel.asUniProtNameHeatmap)"><v-icon>mdi mdi-table-column-width</v-icon></v-btn>
                          </template>
                          <span>Interactive Expression Heatmap</span>
                        </v-tooltip>
                        <v-tooltip top>
                          <template v-slot:activator="{ on, attrs }">
                            <v-btn :disabled="!NodeModel.bEnableCt" :color="$store.state.selectedOrganismShown.primaryColor" outlined fab small class="ma-1" v-bind="attrs" v-on="on" @click="proteinLinkCtFormatter(NodeModel.asUniProtNameCt)">CT</v-btn>
                          </template>
                          <span>Combination Treatment</span>
                        </v-tooltip>
                      </v-list-item>
                    </v-tab-item>
                    <v-tab-item :value="'tab-2'">
                      <v-list-item>
                        <v-list-item-title class="text-center" >Visible resources</v-list-item-title>
                      </v-list-item>
                      <v-list dense style="max-height:400px">
                        <v-list-item v-for="(item, i) in GraphRadioModel.Radios" :key="i">
                          <v-switch dense class="ml-7 mt-0 pt-0" v-model="item.State" :label="`${item.ResourceType} : ${!item.State ? 'hidden' : 'visible' }`" @change="onRadioSelection"></v-switch>
                        </v-list-item>
                      </v-list>
                      <v-divider/>
                        <v-col align="center">
                          <v-list-item>
                            <v-list-item-title class="text-center" >Downloadable content</v-list-item-title>
                          </v-list-item>
                          <v-btn dark :color="$store.state.selectedOrganismShown.primaryColor" class ="ma-2" @click="onSvgClick"><v-icon>fas fa-download</v-icon>SVG</v-btn>
                          <v-btn dark :color="$store.state.selectedOrganismShown.primaryColor" class ="ma-2" @click="onPngClick"><v-icon>fas fa-download</v-icon>PNG</v-btn>
                          <v-btn dark :color="$store.state.selectedOrganismShown.primaryColor" class ="ma-2" @click="onSifClick"><v-icon>fas fa-download</v-icon>SIF</v-btn>
                          <canvas id="canvasId" style=display:none></canvas>
                        </v-col>
                      </v-tab-item>
                    </v-tabs-items>
                  </v-tabs>
                </v-card>
              </v-col>

              <v-btn v-if="UiModel.visibleD3Button" :color="$store.state.selectedOrganismShown.primaryColor" fab outlined elevation="4" absolute style="top:140px;left:565px;" @click="onButtonPressDelete"><v-icon>far fa-trash-alt</v-icon></v-btn>
              <v-btn v-if="UiModel.visibleD3Button" :color="$store.state.selectedOrganismShown.primaryColor" :disabled="!UiModel.enableExpansion" fab outlined elevation="4" absolute style="top:210px;left:565px;" @click="onButtonPressExpand"><v-icon>fas fa-plus</v-icon></v-btn>
              <v-btn v-if="UiModel.visibleD3Button" :color="$store.state.selectedOrganismShown.primaryColor" fab outlined elevation="4" absolute style="top:280px;left:565px;" @click="onButtonPressUnfix"><v-icon>fas fa-unlock-alt</v-icon></v-btn>

              <v-col cols="9">
                <interactiongraph ref="interactionsgraph" 
                                  v-if="UiModel.visibleD3Button"
                                  :height="'600px'"
                                  :proteinId="proteinId"
                                  :selectedEdges="SelectionModel"
                                  @visibleRetrieval="onRetrievalChange"
                                  :accessionId="PlotInformationModel.AccessionId"
                                  :sRelationTypeIds="PlotInformationModel.sRelationTypeIds"
                                  :endpointAccession="this.$store.state.host+'/logic/pathways/getAccessionId.xsjs?protein_id=__parameter1__'"
                                  @disableNodeTab="disableNodeTab"
                                  @keyChange="onKeyChange"
                                  @showOverlay="onShowOverlay"
                                  @enableExpansion="onEnableExpansion"
                                  @triggerRadioSelection="onRadioSelection"
                                  @NodeModel="onNodeSelection"
                                  @radioButtons="setGraphRadioModel"
                                  :heatmapLinkGenerator="heatmapLinkFormatter"
                                  :endpoint="this.$store.state.host+'/logic/pathways/getPathwayGraphForProteinId.xsjs?accession_id=__parameter1__&amp;resource_type=__parameter2__&amp;relation_ids=__parameter3__&amp;offset=__parameter4__&amp;new_graph=__parameter5__&amp;sn_exist=__parameter6__'" />
              </v-col>
              <edgePopup :openDialog="isEdgeSelected" :inputData="overlayData" @closePopUp="disablePopUp"/>
            </v-row>
          </v-container>
</template>

<script>
import axios from 'axios'
import utils from '@/vue-d3-component-wrappers/common-lib/Utils.js'
import edgePopup from '@/views/popup/InteractionNetworkEdgeOverlay'
import interactiongraph from '@/vue-d3-components/InteractionNetwork'

export default {
  name: 'proteinInteractionGraph',
  components: {
    interactiongraph: interactiongraph,
    edgePopup: edgePopup
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
    proteinAccession: {
      type: String,
      default: null
    }
  },
  data: function () {
    return {
      RelationTypesModel: [],
      isNodeSelected: false,
      isEdgeSelected: false,
      isGraphShown: false,
      loader: null,
      loading: false,
      tab: 0,
      PlotInformationModel: {
        proteinId: this.proteinId,
        visible: false
      },
      overlayData: [{
        Source: '',
        Destination: '',
        Relations: []
      },
      {
        Source: '',
        Destination: '',
        Relations: []
      }],
      UiModel: {
        visibleD3Button: false,
        visibleEdge: false,
        visibleNode: false,
        tabIndex: 'tab-0',
        isRendered: false,
        visibleRetrieval: false,
        canvasHidden: '<canvas id=hiddenPNG style=display:none />'
      },
      SelectionModel: [],
      GraphRadioModel: { Radios: [] },
      NodeModel: [],
      modifiedRelationSelection: false,
      svgCss: [require('@/vue-d3-components/d3v4-selectable-zoomable-force-directed-graph.css.prdb')]
    }
  },
  watch: {
/*    loader () {
      const l = this.loader
      this[l] = !this[l]

      setTimeout(() => (this[l] = false), 3000)

      this.loader = null
    },*/
  },
  methods: {
    getAccessionId: function() {
      var that = this;
      axios.get(this.$store.state.host+'/proteomicsdb/logic/pathways/getAccessionId.xsjs', {params: {protein_id: that.proteinId}}).then(function (response) {
        that.PlotInformationModel.AccessionId = response.data.AccessionId;
      })
    },
    getRelations: function() {
      var that = this;
      this.modifiedRelationSelection = true;
      axios.get(this.$store.state.host+'/proteomicsdb/logic/pathways/getAllRelationTypes.xsjs', {}).then(function (response) {
        var aRelationTypes = response.data;
        aRelationTypes = aRelationTypes.map(function(dataset) {
          dataset.Children = dataset.Children.map(function(elements) {
            elements.value = elements.Name;
            elements.selected = elements.Selected > 0;
            elements.cheatCss = 'sapInteractionCheckbox' + elements.Level;
            var xDistance = elements.Level * 30 + 10;
            var svgConstructStart = '<svg height="21" width="' + xDistance + '">';
            var svgConstructStop = '</svg>';
            // CHEAT is necessary  for unique ids in the document
            var marker_base = '<marker id="CHEAT__ArrowTypeId__" viewBox="-10 -10 20 20" refX="0" refY="0" markerWidth="6" markerHeight="6" orient="auto"><path transform="scale(1)" fill="__ArrowColour__" d="__Marker__"></path></marker>';

            var marker = elements.EndArrowTypeId ? marker_base.replace('__ArrowTypeId__', elements.EndArrowTypeId).replace('__ArrowColour__', elements.EndArrowColour).replace('__Marker__', elements.EndMarker) : '';

            marker += elements.StartArrowTypeId ? marker_base.replace('__ArrowTypeId__', elements.StartArrowTypeId).replace('__ArrowColour__', elements.StartArrowColour).replace('__Marker__', elements.StartMarker) : '';
            marker += elements.MidArrowTypeId ? marker_base.replace('__ArrowTypeId__', elements.MidArrowTypeId).replace('__ArrowColour__', elements.MidArrowColour).replace('__Marker__', elements.MidMarker) : '';

            var defs = '<defs>' + marker + '</defs>';
            var marker_mid = elements.MidArrowTypeId ? ' marker-mid="url(#CHEAT__MidArrowTypeId__)" '.replace('__MidArrowTypeId__', elements.MidArrowTypeId) : '';
            var marker_end = elements.EndArrowTypeId ? ' marker-end="url(#CHEAT__EndArrowTypeId__)" '.replace('__EndArrowTypeId__', elements.EndArrowTypeId) : '';
            var marker_start = elements.StartArrowTypeId ? ' marker-mid="url(#CHEAT__StartArrowTypeId__)" '.replace('__StartArrowTypeId__', elements.StartArrowTypeId) : '';
            var path_base = '<path ' + marker_mid + marker_end + marker_start + ' class="linkRef" refY="-50" stroke="black" stroke-width="3px" d="M10,10L20,10L30,10"></path>';
            var path = path_base.replace('__ArrowTypeId__', elements.ArrowTypeId);

            elements.svg = svgConstructStart + defs + path + svgConstructStop;
            return elements;
          });
          return dataset;
        });

        that.RelationTypesModel = aRelationTypes;

      })
    },
    disablePopUp: function() {
      this.isEdgeSelected = false;
    },
    onEnableExpansion: function onEnableExpansion(oEvent) {
      var bEnable = oEvent.bEnable;
      this.UiModel.enableExpansion = bEnable;
    },
    runAnalysis: function(){
      if(this.modifiedRelationSelection){
        this.loader = 'loading';
        const l = this.loader;
        this[l] = !this[l];
        this.modifiedRelationSelection = false;
      }
      this.UiModel.visibleD3Button = true;
      this.UiModel.visibleEdge = false;
      this.UiModel.visibleNode = false;
      this.UiModel.tabIndex = 'tab-0';
      this.UiModel.isRendered = false;
      this.UiModel.visibleRetrieval = false;
      var sRelationTypeIds = this.getSelectedRelationTypeIds();
      this.PlotInformationModel.sRelationTypeIds = sRelationTypeIds;
      this.PlotInformationModel.visible = true;
      this.$emit('dataLoaded',null);
    },
    onRetrievalChange: function(data){
      this.UiModel.visibleRetrieval = data.value;
      const l = this.loader;
      this[l] = false;
      this.loader = null;
    },

    onKeyChange: function onKeyChange(oEvent) {
      var iTabId = oEvent.iTabId;
      this.UiModel.visibleEdge = iTabId === 1 ? false : true;
      this.UiModel.visibleNode = iTabId === 2 ? false : true;
      this.UiModel.tabIndex = "tab-"+iTabId;
    },

    nTitleFormatter: function proteinTitleFormatter(name, id) {
      return name + ' (' + id + ')';
    },

    onCheckBoxChange: function onCheckBoxChange() {},

    disableNodeTab: function disableNodeTab(oEvent) {
      var bDisableNode = oEvent.bDisable;
      this.UiModel.visibleNode = !bDisableNode;
      this.NodeModel = [];
      if ( this.UiModel.tabIndex === 'tab-1') {
        this.UiModel.tabIndex = 'tab-0';
        this.UiModel.enableExpansion = false;
      }
    },

    onShowOverlay: function onShowOverlay(oEvent) {
      var sSource = oEvent.source;
      var sTarget = oEvent.target;
      var sDirectional = oEvent.directional;
      var sArrowType = oEvent.arrowtype;
      var sRelation_ids = oEvent.relation_ids;
      var that = this;

      axios.get(this.$store.state.host+'/proteomicsdb/logic/pathways/getEdgeInfo.xsjs', {params: {
              source: sSource,
              target: sTarget,
              directional: sDirectional,
              arrowtype: sArrowType,
              relation_ids: sRelation_ids
            }}).then(function (response) {
              that.overlayData = response.data;
              that.isEdgeSelected = true;
      });
    },
    onSelectionModelChange: function onSelectionModelChange() {},

    onButtonNetwork: function onButtonNetwork() {
      window.open('tesgt', '_blank');
    },
    onButtonPressExpand: function onButtonPressExpand() {
      this.$refs.interactionsgraph.expandSelection();
    },
    onButtonPressDelete: function onButtonPressDelete() {
      this.$refs.interactionsgraph.deleteSelectionDirty();
    },
    onButtonPressUnfix: function onButtonPressUnfix() {
      this.$refs.interactionsgraph.unfixSelection();
    },
    onSvgClick: function() {
      var aPlots = [];
      aPlots.push(this.$refs.interactionsgraph.getSVG());
      utils.downloadSVGs(aPlots, 'InteractionsGraph: ', true, 'canvasId', this.svgCss);
    },
    onPngClick: function() {
      var aPlots = [];
      aPlots.push(this.$refs.interactionsgraph.getSVG());
      utils.downloadSVGs(aPlots, 'InteractionsGraph: ', false, 'canvasId', this.svgCss);
    },
    onSifClick: function() {
      var aCSVRows = this.$refs.interactionsgraph.getSif();
      var sFileName = this.proteinAccession;
      utils.downloadFile(aCSVRows, sFileName, 'sif');
    },
    proteinLinkFormatter: function proteinLinkFormatter(iProteinId) {
      window.open('/protein/summary/'+ iProteinId, '_blank');
    },

    proteinLinkBcFormatter: function proteinLinkBcFormatter(iProteinId) {
      window.open('/protein/assay/'+ iProteinId, '_blank');
    },

    proteinLinkNodeFormatter: function proteinLinkNodeFormatter(iProteinId) {
      window.open('/protein/interactions/'+ iProteinId, '_blank');
    },

    proteinLinkExpressionFormatter: function proteinLinkExpressionFormatter(iProteinId) {
      window.open('/protein/expression/'+ iProteinId, '_blank');
    },

    proteinLinkCtFormatter: function proteinLinkCtFormatter(asUniProtName) {
      if (typeof asUniProtName === 'string') {
        window.open('/analytics/combinationTreatment?protein_name='+ asUniProtName, '_blank');
      } else if (typeof asUniProtName === 'undefined') {
        return '';
      }
      window.open('/analytics/combinationTreatment?protein_name='+ asUniProtName.join(','), '_blank');
    },

    proteinLinkDsFormatter: function proteinLinkDsFormatter(sUniProtName) {
      window.open('/analytics/selectivity?protein_name=' + sUniProtName, '_blank');
    },

    // array consists of Data from D3 graph
    heatmapLinkFormatter: function heatmapLinkFormatter(asUniProtNameHeatmap) {
      if (typeof asUniProtNameHeatmap === 'undefined') {
        return '';
      }
      var iTaxCode = this.$store.state.cookie;
      window.open('/analytics/expressionHeatmap?proteins=' + asUniProtNameHeatmap.join('%3B') + '&quantification=1&biologicalSource=tissue%3Bfluid&calculationMethod=0&swissprotOnly=1&noIsoforms=1&taxcode=' + iTaxCode, '_blank');
    },

    setGraphRadioModel: function(oData) {
      this.GraphRadioModel.Radios = oData;
    },

    onRadioSelection: function onRadioSelection() {
      var aRadioStuff = this.GraphRadioModel.Radios;

      var aShownShapes = aRadioStuff.filter(function(d) {
        return d.State;
      }).map(function(d) {
        return d.Shape;
      });
      this.$refs.interactionsgraph.seek_and_hide(true, aShownShapes);
    },

    onNodeSelection: function(nodes) {
      this.NodeModel = nodes;
    },

    getSelectedRelationTypeIds: function getSelectedRelationTypeIds() {
      var aRelationTypes = this.RelationTypesModel;
      aRelationTypes = aRelationTypes.map(function(dataset) {
        var selectedElements = dataset.Children.filter(function(element) {
          return element.selected;
        });
        return selectedElements.map(function(element) {
          return element.RelationTypeId;
        });
      });
      return aRelationTypes.reduce(function(a, b) {
        return a.concat(b);
      }, []).join(',');
    },

  },
  computed: {
    selectedRelations: function() {
      var aRelationTypes = this.RelationTypesModel;
      aRelationTypes = aRelationTypes.map(function(dataset) {
        var selectedElements = dataset.Children.filter(function(element) {
          return element.selected;
        });
        return selectedElements.map(function(element) {
          return element.RelationTypeId;
        });
      });
      return aRelationTypes.reduce(function(a, b) {
        return a.concat(b);
      }, []).join(',');

    }
  },
  mounted() {
    this.getRelations();
    this.getAccessionId();
  }
}
</script>
<style scss>
@import '../vue-d3-components/d3v4-selectable-zoomable-force-directed-graph.css.prdb'
</style>
