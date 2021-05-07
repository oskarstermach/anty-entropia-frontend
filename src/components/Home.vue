<template>
  <div class="landingpage">
    <div class="wrapper">
      <div class="section section-hero section-shaped">
        <div class="shape shape-style-3 shape-default">
          <span class="span-150"></span>
          <span class="span-50"></span>
          <span class="span-50"></span>
          <span class="span-75"></span>
          <span class="span-100"></span>
          <span class="span-75"></span>
          <span class="span-50"></span>
          <span class="span-100"></span>
          <span class="span-50"></span>
          <span class="span-100"></span>
        </div>
        <div class="page-header">
          <div class="container shape-container d-flex align-items-center py-lg">
            <div class="col px-0">
              <div class="row align-items-center justify-content-center">
                <div class="col-lg-6 text-center">
                  <h1 class="text-white display-2">Spójność ostateczna: anty-entropia</h1>
                  <h2 class="display-4 font-weight-normal text-white">System powinien się składać z 11 serwerów</h2>
                  <div class="btn-wrapper mt-4">
                    <a href="" class="btn btn-success btn-icon mt-3 mb-sm-0">
                      <span class="btn-inner--text">Start new server</span>
                    </a>
                  </div>
                  <br>
                  <DebugLog/>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="separator separator-bottom separator-skew zindex-100">
          <svg x="0" y="0" viewBox="0 0 2560 100" preserveAspectRatio="none" version="1.1"
               xmlns="http://www.w3.org/2000/svg">
            <polygon class="fill-white" points="2560 0 2560 100 0 100"></polygon>
          </svg>
        </div>
      </div>
    </div>
    <div class="section features-6">
      <div class="container">
        <div class="row align-items-center">
          <table class="table">
            <thead>
            <tr>
              <th class="text-center">#</th>
              <th>Address</th>
              <th>Status</th>
              <th>Values</th>
              <th class="text-right">Change</th>
              <th class="text-right">Actions <input v-model="value" type="number" placeholder="value">
              </th>

            </tr>
            </thead>
            <tbody>
            <tr v-for="(server,index) in this.servers" :key="index">
              <td class="text-center">{{ index }}</td>
              <td>{{ server.serverName }}</td>
              <td><span class="badge badge-success">ONLINE</span></td>
              <td>{{ server.values }}</td>
              <td class="text-right">{{ server.change }}</td>

              <td class="td-actions text-right">

                <button type="button" @click="sendValue(server.serverName)" rel="tooltip"
                        class="btn btn-primary btn-sm " data-original-title="" title="">
                  ADD
                </button>
                <button type="button" rel="tooltip" @click="deleteValue(server.serverName)"
                        class="btn btn-primary btn-sm " data-original-title="" title="">
                  REMOVE
                </button>
                <button type="button" rel="tooltip" class="btn btn-danger btn-icon btn-sm "
                        @click="turnOff(server.serverName)"
                        data-original-title=""
                        title=""> OFF
                  <i class="ni ni-button-power"></i>
                </button>
              </td>
            </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <footer class="footer">
      <div class="container">
        <div class="row row-grid align-items-center mb-5">
          <div class="col-lg-6">
            <h3 class="text-primary font-weight-light mb-2">Projekt</h3>
            <h4 class="mb-0 font-weight-light">Systemy odporne na błędy</h4>
          </div>
        </div>
        <hr>
        <div class="row align-items-center justify-content-md-between">
          <div class="col-md-6">
            <div class="copyright">
              Projekt został wykonany przez: <a href="" target="_blank"></a>
            </div>
          </div>
          <div class="col-md-6">
            <ul class="nav nav-footer justify-content-end">
              <li class="nav-item">
                <a href="" class="nav-link" target="_blank">Oskar Stermach</a>
              </li>
              <li class="nav-item">
                <a href="" class="nav-link" target="_blank">Adam Snoch</a>
              </li>
              <li class="nav-item">
                <a href="" class="nav-link" target="_blank">Maciej Świercz</a>
              </li>
              <li class="nav-item">
                <a href="" class="nav-link" target="_blank">Piotr Walczak</a>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </footer>
  </div>

</template>


<script>
import Socket from "./socket"
import ValueSocket from "./ValueSocket";
import axios from "axios";
import DebugLog from "./DebugLog";


export default {
  name: "HomePage",
  components: {DebugLog},
  data() {
    return {
      servers: [],
      value: 0
    }
  },
  methods: {
    async getConfiguration() {
      await axios.get("http://localhost:8080/server/configurationSimple")
        .then(result => this.servers = result.data.map(server => ({serverName: server})))

      await this.servers.forEach(server => axios.get("http://" + server.serverName + "/values").then(result => server.values = result.data))
    },
    sendValue(server) {
      axios.put('http://' + server + '/values/' + this.value).then(result => console.log(result))
    },
    deleteValue(server) {
      axios.delete('http://' + server + '/values/' + this.value).then(result => console.log(result))
    },
    turnOff(server) {
      axios.post('http://' + server + '/actuator/shutdown').then(result => console.log(result))
    },
    getServers() {
      this.servers = [...new Set(this.servers)]
    },

    handleStateMessage(msg) {
      let parsedEvent = JSON.parse(msg)
      this.servers = [...new Set(this.servers)]

      if (parsedEvent.state === 'CONNECTED') {
        const index = this.servers.findIndex(x => x.serverName === parsedEvent.serverName);
        if (index === -1) {
          const obj = {
            serverName: parsedEvent.serverName
          };
          this.servers.push(obj)
          this.getServers()
        }

      } else if (parsedEvent.state === 'DISCONNECTED') {
        const index = this.servers.findIndex(x => x.serverName === parsedEvent.serverName);

        if (index > -1) {
          this.servers.splice(index, 1)

        }
      }
    },
    handleValueMessage(msg) {
      let parseValues = JSON.parse(msg)
      if (parseValues.result !== undefined) {

        const index = this.servers.findIndex(x => x.serverName === parseValues.serverName);
        console.log("emituje na eventhub o" + msg);
        this.$eventHub.$emit('log-msg', {
          timestamp: new Date(),
          serverIndex: index,
          serverName: parseValues.serverName,
          changed: parseValues.change,
          current: parseValues.result,
          type: parseValues.type
        });
        console.log("wemitowane");
        this.servers[index] = {
          serverName: parseValues.serverName,
          values: parseValues.result,
          change: parseValues.change
        }
      }
    }


  },
  created() {
    Socket.$on("state", this.handleStateMessage)
    ValueSocket.$on("value", this.handleValueMessage)
  },
  beforeDestroy() {
    Socket.$off("state", this.handleStateMessage)
    ValueSocket.$off("value", this.handleValueMessage)
  },
  beforeMount() {
    this.getConfiguration()
  },

}
</script>
