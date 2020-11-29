<template>
  <v-app>
    <v-app-bar app title>
      <div class="d-flex align-center">
        <v-toolbar-title>Kyash Barcode Generator</v-toolbar-title>
      </div>
      <v-spacer></v-spacer>
      <v-btn
        href="https://github.com/konifar/kyash-barcode-generator"
        target="_blank"
        text
      >
        <span class="mr-2">GitHub</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-container>
        <v-row>
          <v-col cols="12" sm="12" md="4" style="background-color: #FFCDD2">
            <p v-if="error !== null" class="drop-error">
              {{ error }}
            </p>
            <QrcodeDropZone @detect="onDetect" @dragover="onDragOver" @init="logErrors">
              <div>
              <div class="drop-area" :class="{ 'dragover': dragover }">
                DROP SOME IMAGES HERE
              </div>
              <QrcodeVue :value="result" :size="200" />
              </div>
            </QrcodeDropZone>
            <QrcodeCapture @decode="onDecode" />
          </v-col>
          <v-col cols="12" sm="12" md="8" style="background-color: #F8BBD0">
            <p class="decode-result">Last result: <b>{{ result }}</b></p>
            <v-text-field
              v-model="result"
              label="Amount"
            />
            <v-text-field
              label="Message"
            />
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script lang="ts">
import Vue from "vue";
import { QrcodeDropZone, QrcodeCapture } from "vue-qrcode-reader";
import QrcodeVue from "qrcode.vue";
import HelloWorld from "./components/HelloWorld.vue";

export default Vue.extend({
  name: "App",

  components: {
    QrcodeDropZone,
    QrcodeCapture,
    QrcodeVue
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
    async onDetect (promise: Promise<any>) {
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