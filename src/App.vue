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
          <!-- Barcode reading area -->
          <v-col cols="12" xs="12" md="4">
            <QrcodeDropZone 
              @detect="onDetect" 
              @dragover="onDragOver" 
              @init="onDragInit">
              <div 
                class="drop-area" 
                :class="{ 'dragover': isDragging }">
                <p>Drag Kyash barcode here</p>
                <QrcodeVue 
                  v-if="url !== ''"
                  :value="url"
                  :size="200" />
              </div>
            </QrcodeDropZone>

            <QrcodeCapture @decode="onDecode" class="my-4" />

            <v-alert 
              v-if="errorMessage !== ''" 
              border="left"
              dismissible
              outlined
              type="error">
              {{ errorMessage }}
            </v-alert>
          </v-col>

          <!-- Barcode data area -->
          <v-col cols="12" xs="12" md="8">
            <p class="decode-result">Last result: <b>{{ url }}</b></p>
            <v-text-field
              v-model="url"
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
    url: "",
    errorMessage: "",
    isDragging: false
  }),
  
  methods: {
    onDecode (decodedString: string) {
      this.url = decodedString
    },
    async onDetect (promise: Promise<any>) {
      try {
        const { content } = await promise
        this.url = content
        this.errorMessage = ""
      } catch (error) {
        if (error.name === 'DropImageFetchError') {
          this.errorMessage = 'Failed to load images.'
        } else if (error.name === 'DropImageDecodeError') {
          this.errorMessage = 'Failed to decode file. Maybe it\'s not an image.'
        } else {
          this.errorMessage = 'Ups, what kind of error is this?! ' + error.message
        }
      }
    },

    onDragInit (promise: Promise<any>) {
      promise.catch(console.error)
    },

    onDragOver (isDraggingOver: boolean) {
      this.isDragging = isDraggingOver
    }
  }
});
</script>

<style>
.drop-area {
  height: 300px;
  width: 300px;
  color: #fff;
  text-align: center;
  font-weight: bold;
  padding: 10px;
  background-color: rgba(0,0,0,.5);
}

.dragover {
  background-color: rgba(0,0,0,.8);
}
</style>