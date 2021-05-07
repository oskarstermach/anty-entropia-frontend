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
      separator: '; '
    }
  },
  created() {
    this.$eventHub.$on('log-msg', this.addValueMsgToLog);
  },
  beforeDestroy(){
    this.$eventHub.$off('log-msg');
  },
  methods: {
    addValueMsgToLog(msg) {
      console.log("addToLog uruchomione");
      let logText = this.formatTimestamp(msg.timestamp) + this.separator
        + msg.serverIndex + " - " + msg.serverName + this.separator;
      if (msg.type === "ADDED") {
        logText += "Dodano " + msg.changed + this.separator;
      } else if (msg.type === "DELETED") {
        logText += "Usunięto " + msg.changed + this.separator;
      }
      logText += ("[" + msg.current + "]");
      this.$refs.logbox.value = this.$refs.logbox.value + logText + "\n";

    },
    formatTimestamp(stamp) {
      return stamp.getHours() + ":" + stamp.getMinutes() + ":" + stamp.getSeconds() + "." + stamp.getMilliseconds()
    }
  }
}
</script>

<style scoped>

</style>

