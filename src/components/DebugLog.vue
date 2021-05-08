// TODO Dodać styl do textarea i wyłączyć zawijanie; Dodać ukrywanie i pokazywanie logu;
// Może zmienić czcionkę na bardziej konsolową
<template>
  <div>
  <textarea id="debuglog" name="debuglog" ref="logbox" rows="5" cols="40">
  </textarea>
  </div>
</template>

<script>
export default {
  name: "DebugLog",
  data() {
    return {
      separator: '; ',
      counter: 1,
      logArchive: ""
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
      let logText = this.counter + this.separator
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
      let logText = this.counter + this.separator
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
      return stamp.getHours() + ":" + stamp.getMinutes() + ":" + stamp.getSeconds() + "." + stamp.getMilliseconds()
    },
    openStorage () {
      return JSON.parse(localStorage.getItem('log_data'));
    },
    saveStorage (data) {
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
    }
  }
}
</script>

<style scoped>

</style>

