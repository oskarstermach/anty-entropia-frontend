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
    this.$eventHub.$on('log-msg', this.addValueMsgToLog);
  },
  mounted() {
    let retrievedData = this.openStorage();
    if (retrievedData) {
      this.counter = retrievedData.counter;
      this.logArchive = retrievedData.logArchive;
    }
    this.$refs.logbox.value = this.logArchive;
  },
  beforeDestroy() {
    console.log("beforeDestroy")
    // Ten log w localstorage przetrwa odświeżanie strony, ale ponowne uruchomienie apki już nie
    // (hot reload z ide też nie)
    this.$eventHub.$off('log-msg');
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
      this.counter++;
      this.logArchive += logText + "\n";
      this.$refs.logbox.value = this.logArchive;
      console.log({
        counter: this.counter,
        logArchive: this.logArchive
      });
      this.saveStorage({
        counter: this.counter,
        logArchive: this.logArchive
      });
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
  }
}
</script>

<style scoped>

</style>

