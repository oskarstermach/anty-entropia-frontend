<template>
  <div>
    <div style="height:calc(100vh - 50px);">
      <RelationGraph ref="seeksRelationGraph" :options="graphOptions" :on-node-click="onNodeClick"
                     :on-line-click="onLineClick"/>
    </div>
  </div>
</template>

<script>
import RelationGraph from 'relation-graph'
import axios from "axios";

export default {
  name: 'RelationServerGraph',
  components: {RelationGraph},
  data() {
    return {
      graphOptions: {
        allowSwitchLineShape: true,
        allowSwitchJunctionPoint: true,
        defaultJunctionPoint: 'border'
      },
      addresses: [],
      configurations: []
    }
  },
  mounted() {
    this.getConfiguration()
      .then(this.showSeeksGraph)
  },
  methods: {
    async getConfiguration() {
      await axios.get("http://localhost:8080/server/configurationSimple")
        .then(result => this.addresses = result.data)

      await axios.get("http://localhost:8080/server/configuration")
        .then(result => this.configurations = result.data
          .map(cfg => ({
            address: cfg.address,
            neighbours: cfg.neighbourAddresses
          })))
    },
    showSeeksGraph() {
      let nodes = this.addresses.map(el => ({
        id: el,
        text: el,
        borderColor: 'yellow',
        width: 150
      }))

      let links = this.configurations
        .flatMap(el => el.neighbours
          .map(ne => {
            return ({from: el.address, to: ne, color: '#67C23A'})
          }))

      console.log(links)


      var __graph_json_data = {
        rootId: this.addresses[0],
        nodes: nodes,
        links: links
      }


      this.$refs.seeksRelationGraph.setJsonData(__graph_json_data, (seeksRGGraph) => {
      })
    },
    onNodeClick(nodeObject, $event) {
      console.log('onNodeClick:', nodeObject)
    },
    onLineClick(lineObject, $event) {
      console.log('onLineClick:', lineObject)
    }
  },

}
</script>
