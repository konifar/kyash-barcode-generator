<template>
  <v-app>
    <v-app-bar app title>
      <div class="d-flex align-center">
        <v-toolbar-title>Kyash Barcode Generator</v-toolbar-title>
      </div>
      <v-spacer></v-spacer>
      <v-btn href="https://github.com/konifar/kyash-barcode-generator" target="_blank" text>
        <span class="mr-2">GitHub</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-container>
        <v-row>
          <!-- Barcode reading area -->
          <v-col cols="12" xs="12" md="4">
            <QrcodeDropZone @detect="onDetect" @dragover="onDragOver" @init="onDragInit">
              <div class="drop-area" :class="{ 'dragover': isDragging }">
                <p>Drag Kyash barcode here</p>
                <QrcodeVue v-if="url !== ''" :value="url" :size="200" />
              </div>
            </QrcodeDropZone>

            <QrcodeCapture @decode="onDecode" class="my-4" />

            <v-alert v-if="errorMessage !== ''" border="left" dismissible outlined type="error">
              {{ errorMessage }}
            </v-alert>
          </v-col>

          <!-- Barcode data input area -->
          <v-col cols="12" xs="12" md="8">
            <v-text-field label="Id" v-model="id" />
            <v-select label="Action" :items="actionItems" v-model="action" />
            <v-text-field label="Amount" v-model="amount" />
            <v-text-field label="Message" v-model="message" />

            <v-alert v-if="url !== ''" border="left" color="blue" type="success" outlined>{{ url }}</v-alert>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script lang="ts">
import { Vue, Component, Prop } from 'vue-property-decorator'
import { QrcodeDropZone, QrcodeCapture } from "vue-qrcode-reader";
import QrcodeVue from "qrcode.vue";

const KYASH_DEEPLINK_PROTOCOL = "kyash:"
const KYASH_DEEPLINK_PATH_NAME = "//qr/u/"
const ACTIONS = ['send', 'request']

@Component({
  components: {
    QrcodeDropZone,
    QrcodeCapture,
    QrcodeVue
  }
})
export default class App extends Vue {
  public url = ""
  public id = ""
  public action = ACTIONS[0]
  public amount = ""
  public message = ""

  public errorMessage = ""
  public isDragging = false
  public actionItems = ACTIONS

  private onDecode(decodedString: string) {
    try {
      this.errorMessage = ""
      const url = this.validateBarcodeString(decodedString)
      this.url = url.href

      this.id = url.pathname.replace(KYASH_DEEPLINK_PATH_NAME, '')
      const params =url.searchParams 
      const action = params.get("action") || ACTIONS[0]
      if (action == "send" || action == "request") {
        this.action = action
      }
      this.amount = params.get("amount") || ""
      this.message = params.get("message") || ""
    } catch (e) {
      this.errorMessage = e.message
    }
  }

  private async onDetect(promise: Promise<any>) {
    try {
      const { content } = await promise
      this.onDecode(content)
    } catch (error) {
      if (error.name === 'DropImageFetchError') {
        this.errorMessage = 'Failed to load images.'
      } else if (error.name === 'DropImageDecodeError') {
        this.errorMessage = 'Failed to decode file. Maybe it\'s not an image.'
      } else {
        this.errorMessage = 'Ups, what kind of error is this?! ' + error.message
      }
    }
  }

  private onDragInit(promise: Promise<any>) {
    promise.catch(console.error)
  }

  private onDragOver(isDraggingOver: boolean) {
    this.isDragging = isDraggingOver
  }

  private validateBarcodeString(barcodeString: string): URL {
    try {
      const url = new URL(barcodeString)
      console.log(url.pathname)
      if (!url.protocol.startsWith(KYASH_DEEPLINK_PROTOCOL) 
        || !url.pathname.startsWith(KYASH_DEEPLINK_PATH_NAME)) {
        throw new Error("Invalid kyash deeplink url.");
      }
      return url
    } catch (e) {
      throw new Error("Invalid kyash deeplink url.");
    }
  }
}
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