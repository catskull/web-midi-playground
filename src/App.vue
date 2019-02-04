<template>
  <div id="app">
    <h3>Web MIDI Playground</h3>
    <template v-if="!clockInterval">
      <input v-model="clockTempo" type="number" placeholder="BPM"/>
      <button
        @click="startMidiCLock"
      >Start Clock</button>
    </template>
    <template v-else>
      <button
        @click="stopMidiClock"
      >Stop Clock</button>
    </template>

    <br>

    <input v-model="midiData.status" type="number" placeholder="Status"/>
    <input v-model="midiData.data1" type="number" placeholder="Data Byte 1"/>
    <input v-model="midiData.data2" type="number" placeholder="Data Byte 2"/>
    <button
      @click="sendMidiByte"
    >Send Byte</button>

    <p>Built with <a href="https://vuejs.org/">Vue.js</a></p>
    <p>and <a href="https://github.com/djipco/webmidi">WebMidi.js</a></p>
    <p>by <a href="https://github.com/catskull">catskull</a></p>
    <p>2019</p>
  </div>
</template>

<script>
import WebMidi from 'webmidi'

export default {
  name: 'app',
  data () {
    return {
      outputDevice: {},
      clockTempo: undefined,
      clockInterval: undefined,
      midiData: {
        status: null,
        data1: null,
        data2: null
      }
    }
  },
  computed: {
    bpmInMilliseconds () {
      return 60000 / parseInt(this.clockTempo)
    }
  },
  methods: {
    startMidiCLock () {
      this.clockInterval = setInterval(() => {
        this.sendMidiClock()
      }, this.bpmInMilliseconds / 24)
    },
    stopMidiClock () {
      clearInterval(this.clockInterval)
      this.clockInterval = undefined
    },
    sendMidiClock () {
      this.outputDevice.sendClock()
    },
    sendMidiByte () {
      this.outputDevice.send(this.midiData.status, [this.midiData.data1, this.midiData.data2])
    }
  },
  created () {
    WebMidi.enable(err => {
      if (err) {
        console.log('WebMidi not enabled', err)
      } else {
        this.outputDevice = WebMidi.outputs[0]
        this.outputDevice.sendStart()
        this.outputDevice.playNote(64, 1)
        this.outputDevice.sendClock()
      }
    }, 'enable sysex')
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
