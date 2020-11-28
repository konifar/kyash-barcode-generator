<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />

        <v-img
          alt="Vuetify Name"
          class="shrink mt-1 hidden-sm-and-down"
          contain
          min-width="100"
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-name-dark.png"
          width="100"
        />
      </div>

      <v-spacer></v-spacer>

      <v-btn
        href="https://github.com/vuetifyjs/vuetify/releases/latest"
        target="_blank"
        text
      >
        <span class="mr-2">Latest Release</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <p class="decode-result">Last result: <b>{{ result }}</b></p>

      <p v-if="error !== null" class="drop-error">
        {{ error }}
      </p>

      <QrcodeDropZone @detect="onDetect" @dragover="onDragOver" @init="logErrors">
        <div class="drop-area" :class="{ 'dragover': dragover }">
          DROP SOME IMAGES HERE
        </div>
      </QrcodeDropZone>
      <QrcodeCapture @decode="onDecode" />
    </v-main>
  </v-app>
</template>

<script lang="ts">
import Vue from "vue";
import { QrcodeDropZone, QrcodeCapture } from "vue-qrcode-reader";
import HelloWorld from "./components/HelloWorld.vue";

export default Vue.extend({
  name: "App",

  components: {
    QrcodeDropZone,
    QrcodeCapture
  },

  data: () => ({
    result: "",
    error: "",
    dragover: false
  }),
  
  methods: {
    onDecode (decodedString: string) {
      this.result = decodedString
    },
    async onDetect (promise: Promise<string>) {
      try {
        const { content } = await promise
        this.result = content
        this.error = ""
      } catch (error) {
        if (error.name === 'DropImageFetchError') {
          this.error = 'Sorry, you can\'t load cross-origin images :/'
        } else if (error.name === 'DropImageDecodeError') {
          this.error = 'Ok, that\'s not an image. That can\'t be decoded.'
        } else {
          this.error = 'Ups, what kind of error is this?! ' + error.message
        }
      }
    },

    logErrors (promise: Promise<string>) {
      promise.catch(console.error)
    },

    onDragOver (isDraggingOver: boolean) {
      this.dragover = isDraggingOver
    }
  }
});
</script>

<style>
.drop-area {
  height: 300px;
  color: #fff;
  text-align: center;
  font-weight: bold;
  padding: 10px;

  background-color: rgba(0,0,0,.5);
}

.dragover {
  background-color: rgba(0,0,0,.8);
}

.drop-error {
  color: red;
  font-weight: bold;
}
</style>