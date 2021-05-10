// TODO Dodać styl do textarea i wyłączyć zawijanie; Dodać ukrywanie i pokazywanie logu;
// Może zmienić czcionkę na bardziej konsolową
<template>
  <div>
    <div class="solid" v-show="show">
      <div>
      <textarea id="debuglog" style="overflow:hidden" name="debuglog" ref="logbox" rows="5" cols="60" wrap="soft" readonly="readonly">
      </textarea>
      </div>
      <div>
        <button type="button" @click="saveToFile('debuglog')" rel="tooltip"
                class="btn btn-success btn-sm " data-original-title="" title="">
          DOWNLOAD
        </button>
      </div>
      <br>
    </div>
    <button type="button" @click="show = !show" rel="tooltip"
            class="btn btn-default btn-sm " data-original-title="" title="">
      {{show ? "HIDE LOG" : "SHOW LOG"}}
    </button>

    <button type="button" @click="clearBox()" rel="tooltip"
           class="btn btn-success btn-sm " data-original-title="" title="">
       CLEAR
    </button>

  </div>
</template>

<script>
export default {
  name: "DebugLog",
  data() {
    return {
      separator: ' | ',
      counter: 1,
      logArchive: "",
      show: true
    }
  },
  created() {
    console.log("created");
    this.$eventHub.$on('log-msg', this.addValueMsgToLog);
    this.$eventHub.$on('log-state', this.addStateMsgToLog);
  },
  mounted() {
    console.log("mounted")
    let retrievedData = this.openStorage();
    if (retrievedData) {
      this.counter = retrievedData.counter;
      this.logArchive = retrievedData.logArchive;
    }
    this.$refs.logbox.value = this.logArchive;
  },
  beforeDestroy() {
    this.$eventHub.$off('log-msg');
    this.$eventHub.$off('log-state');
    console.log("beforeDestroy - removing from local storage")
    // Ten log w localstorage przetrwa odświeżanie strony, ale hot reload z ide nie
    localStorage.removeItem('log_data');
  },
  
  methods: {
    addValueMsgToLog(msg) {
      console.log("addToLog uruchomione");
      let logText = this.counter.toString().padStart(5, ' ') + this.separator
        + this.formatTimestamp(msg.timestamp) + this.separator
        + msg.serverIndex + " - " + msg.serverName + this.separator;
      if (msg.type === "ADDED") {
        logText += "Dodano " + msg.changed + this.separator;
      } else if (msg.type === "DELETED") {
        logText += "Usunięto " + msg.changed + this.separator;
      }
      logText += ("[" + msg.current + "]");
      this.updateLogArchive(logText)
    },
    addStateMsgToLog(msg) {
      console.log("addToLog uruchomione");
      let logText = this.counter.toString().padStart(5, ' ') + this.separator
        + this.formatTimestamp(msg.timestamp) + this.separator
        + msg.serverIndex + " - " + msg.serverName + this.separator;
      if (msg.type === "CONNECTED") {
        logText += "Połączony";
      } else if (msg.type === "DISCONNECTED") {
        logText += "Rozłączony";
      }
      this.updateLogArchive(logText)
    },
    formatTimestamp(stamp) {
      return stamp.getHours() + ":" + stamp.getMinutes() + ":" + stamp.getSeconds() + "." + stamp.getMilliseconds().toString().padStart(3, '0');
    },
    openStorage() {
      return JSON.parse(localStorage.getItem('log_data'));
    },
    saveStorage(data) {
      localStorage.setItem('log_data', JSON.stringify(data));
    },
    updateLogArchive(logText) {
      console.log("updating log archive")
      this.counter++;
      this.logArchive += logText + "\n";
      this.$refs.logbox.value = this.logArchive;
      this.saveStorage({
        counter: this.counter,
        logArchive: this.logArchive
      });
    },
    // https://forum.vuejs.org/t/saving-form-data/38714/2
    saveToFile(filename) {
      console.log("saving to file")
      let blob = new Blob([this.logArchive], {type: 'text/plain;charset=utf-8;'})
      if (navigator.msSaveBlob) { // IE 10+
        navigator.msSaveBlob(blob, filename)
      } else {
        let link = document.createElement('a')
        if (link.download !== undefined) { // feature detection
          // Browsers that support HTML5 download attribute
          let url = URL.createObjectURL(blob)
          link.setAttribute('href', url)
          link.setAttribute('download', filename)
          link.style.visibility = 'hidden'
          document.body.appendChild(link)
          link.click()
          document.body.removeChild(link)
        }
      }
    },

    clearBox(){
      this.logArchive = ' ';
      this.$refs.logbox.value = this.logArchive;
    }

  }
}
</script>

<style scoped>
div.solid {border-style: solid;}
textarea
{
  display: inline-block;
  resize: both;
  white-space: pre;
  overflow-wrap: normal;
  overflow-x: scroll;
  max-width: 100%;
  font-family: Menlo,Monaco,Consolas,"Liberation Mono","Courier New",monospace;
  font-size: 14px;

  -moz-border-bottom-colors: none;
  -moz-border-left-colors: none;
  -moz-border-right-colors: none;
  -moz-border-top-colors: none;
  background: none repeat scroll 0 0 rgba(0, 0, 0, 0.07);
  border-color: -moz-use-text-color #FFFFFF #FFFFFF -moz-use-text-color;
  border-image: none;
  border-radius: 6px 6px 6px 6px;
  border-style: none solid solid none;
  border-width: medium 1px 1px medium;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.12) inset;
  color: #ffffff;
  font-weight: bold;
  line-height: 1.4em;
  padding: 5px 8px;
  transition: background-color 0.2s ease 0s;
}

textarea:focus {
    background: none repeat scroll 0 0 #FFFFFF;
    outline-width: 0;
}
</style>

