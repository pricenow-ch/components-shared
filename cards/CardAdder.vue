<template>
  <div style="width: 100%">
    <v-form
      v-if="card.hasOwnProperty('card')"
      ref="cardAdderForm"
      @submit.prevent=""
    >
      <!-- select card type -->
      <v-row>
        <v-col class="col-12 pt-0">
          <v-radio-group v-model="card.getCard().type" row hide-details>
            <v-radio
              v-if="availableCardTypes.includes('swisspass')"
              color="primary"
              :label="$t('general.swisspass')"
              :value="0"
            />
            <v-radio
              v-if="availableCardTypes.includes('skidata')"
              color="primary"
              :label="$t('general.skicard')"
              :value="1"
            />
            <v-radio
              v-if="availableCardTypes.includes('axess')"
              color="primary"
              :label="$t('cardAdder.smartCard')"
              :value="2"
            />
          </v-radio-group>
        </v-col>
      </v-row>

      <!-- card number and add button -->
      <v-row class="justify-space-around">
        <!-- card number -->
        <v-col
          :class="{
            'col-12 col-md-7': card.getType() != 0 || hideCamera,
            'col-8 col-md-6': card.getType() == 0 && !hideCamera,
          }"
          class="pr-2"
        >
          <v-text-field
            ref="cardNumber"
            v-model="card.getCard().cardNumber"
            :disabled="disableSwisspass"
            outlined
            :rules="[
              (v) => !!v || this.$t('cardAdder.cardNumberRequired'),
              (v) =>
                card.isValidKeyCardOrSwisspass() ||
                this.$t('skidata.keycardHasToStartWith01'),
            ]"
            :label="$t('cardAdder.cardNumber')"
            append-icon="s-icon-info"
            @keyup.enter="setZipCodeFocus()"
            @click:append="
              () => {
                $refs.cardNumberInfoModal.show()
              }
            "
          />
        </v-col>

        <!-- camera button -->
        <v-col
          v-if="card.getType() == 0 && !hideCamera"
          class="col-4 col-md-auto pl-2"
        >
          <v-btn fab outlined :disabled="false" @click="openQrModal()">
            <span class="fal fa-camera fa-2x" />
          </v-btn>
        </v-col>

        <!-- swisspass: zip code -->
        <v-col
          v-if="card.getType() == 0"
          :class="{
            'col-12 col-md-auto': !hideCamera,
            'col-12 col-md-auto': hideCamera,
          }"
        >
          <v-text-field
            ref="zipCode"
            v-model="card.getCard().zip"
            :disabled="disableSwisspass"
            outlined
            :rules="[(v) => !!v || this.$t('cardAdder.zipRequired')]"
            :label="$t('general.zip')"
            @keyup.enter="$emit('enter')"
          />
        </v-col>

        <!-- swiss pass agb -->
        <v-col v-if="card.getType() === 0 && !hideAgb" class="col-12 py-0">
          <v-checkbox
            v-model="swisspassAgb"
            :rules="[(v) => !!v || $t('cardAdder.acceptSwisspassAgb')]"
            validate-on-blur
            color="primary"
            class="pt-0 mt-0"
            @click="acceptSwisspassAgb()"
          >
            <template #label>
              <span class="clickable hover-underline" @click="openSwisspassAgb">
                {{ $t('cardAdder.swisspassAgb') }}
              </span>
            </template>
          </v-checkbox>
        </v-col>

        <!-- keycard or smart card: card description -->
        <v-col
          v-if="card.getType() == 1 || card.getType() == 2"
          class="col-12 col-md-5"
        >
          <v-text-field
            v-model="card.getCard().cardDescription"
            outlined
            :label="$t('cardAdder.cardDescription')"
            @keyup.enter="$emit('enter')"
          />
        </v-col>
      </v-row>
    </v-form>

    <!-- qr code reader for swisspass -->
    <s-modal
      ref="qrModal"
      :size="3"
      persistent
      hideOkButton
      @cancel="stopCamera()"
    >
      <v-row v-if="renderQr" class="justify-center">
        <v-col v-if="showWaitingText" class="col-12 text-center pt-6">
          <span class="headline">{{ $t('cardAdder.waitingForCamera') }}</span>
        </v-col>
        <qrcode-reader
          :paused="paused"
          @decode="onDecode"
          @init="loadQr"
        ></qrcode-reader>
      </v-row>
    </s-modal>

    <s-modal
      ref="cardNumberInfoModal"
      hide-cancel-button
      :ok-text="$t('general.close')"
      :size="1"
    >
      <!-- swisspass -->
      <v-row v-if="card.getType() == 0">
        <v-col class="col-12">
          <div class="content-2">
            {{ $t('v5.twoPossibilities') }}
          </div>

          <div class="mt-2 content-4">
            <span class="font-weight-bold"
              >{{ $t('v5.cameraCapturing') }}:</span
            >
            {{ $t('v5.scanQrCode') }}
          </div>

          <div class="mt-2 content-4">
            <span class="font-weight-bold"
              >{{ $t('v5.manualCapturing') }}:</span
            >
            {{ $t('v5.captureSwisspassNumber') }}
          </div>

          <!-- swisspass card example -->
          <img class="mt-2" width="100%" src="@/assets/imgs/swisspass.jpg" />
        </v-col>
      </v-row>

      <!-- keycard -->
      <v-row v-if="card.getType() == 1">
        <v-col class="col-12">
          <div class="content-2">
            {{ $t('v5.keyCardCatpuring') }}
          </div>
          <div class="content-4 mt-2">
            {{ $t('v5.keyCardInputExplanation') }}
          </div>

          <!-- key card image -->
          <img class="mt-2" width="100%" src="@/assets/imgs/keycard.jpg" />
        </v-col>
      </v-row>

      <!-- smart card axess -->
      <v-row v-if="card.getType() == 2">
        <v-col class="col-12">
          <div class="content-2">
            {{ $t('cardAdder.captureSmartCard') }}
          </div>
          <div class="content-4 mt-2">
            {{ $t('cardAdder.smartCardInputExplanation') }}
          </div>

          <!-- key card image -->
          <img class="mt-2" width="100%" src="@/assets/imgs/smartcard.jpg" />
        </v-col>
      </v-row>
    </s-modal>
  </div>
</template>

<script>
import { QrcodeReader } from 'vue-qrcode-reader'
import Card from '../../classes-shared/card/Card.js'
import definitions from '../../../definitions'

export default {
  name: 'CardAdder',

  components: {
    QrcodeReader,
  },

  props: {
    uid: {
      type: Number,
      default: function () {
        return this.$store.getters.getAppUserInstance().getId()
      },
      required: false,
    },

    card: {
      required: false,
      default: () => {
        return new Card()
      },
      type: Card,
    },

    // Array with available media types which can be added
    availableCardTypes: {
      type: Array,
      required: false,
      default: () => {
        return Object.keys(definitions.ticketMedia)
      },
    },

    // hide camera symbol
    hideCamera: {
      required: false,
      default: false,
      type: Boolean,
    },

    // hide agb field
    hideAgb: {
      type: Boolean,
      required: false,
      default: false,
    },

    // make swisspass uneditable
    disableSwisspass: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  data() {
    return {
      paused: false,
      renderQr: false,
      showWaitingText: true,
      swisspassAgb: null,
    }
  },

  watch: {
    availableCardTypes() {
      this.preselectMediaType()
    },
  },

  mounted() {
    this.preselectMediaType()
  },

  methods: {
    preselectMediaType() {
      let preselectMediaType = null
      if (
        this.availableCardTypes.length > 2 &&
        (this.availableCardTypes[0] === definitions.ticketMedia.paper ||
          this.availableCardTypes[0] === definitions.ticketMedia.printAtHome ||
          this.availableCardTypes[0] === definitions.ticketMedia.pickUp)
      ) {
        preselectMediaType = this.availableCardTypes[1]
      } else {
        preselectMediaType = this.availableCardTypes[0]
      }
      this.card.card.setCardType(
        Object.values(definitions.oldMediaType).findIndex(
          (oldMediaType) => oldMediaType === preselectMediaType
        )
      )
    },

    setZipCodeFocus() {
      this.$nextTick(() => {
        this.$refs.zipCode.focus()
      })
    },

    // open the swiss pass agb in a new tab / window
    acceptSwisspassAgb() {
      this.swisspassAgb = true
    },
    openSwisspassAgb() {
      window.open(
        'https://www.swisspass.ch/content/dam/swisspass/kmw/disclaimer/de/SwisspassPlus_Disclaimer_de.pdf'
      )
    },

    // sets focus on card number field
    focusCardNumberField() {
      this.$nextTick(() => {
        this.$refs.cardNumber.focus()
      })
    },

    // returns the mask string depending on swisspass or keycard
    getInputMask() {
      if (this.card.getType() === 0) return 'A##-###-###-###'
      // swisspass
      else if (this.card.getType() === 1) return '##-#### #### #### #### ####-#'
      else return undefined
    },

    getCard() {
      return this.card
    },

    // validate this form
    validateIfUserWantsToAddCard() {
      if (this.card.getCardNumber() || this.card.getZip()) {
        // user wants to add a card
        return this.$refs.cardAdderForm.validate()
      } else {
        return true
      }
    },

    validate() {
      return this.$refs.cardAdderForm.validate()
    },

    // wait for camera
    async loadQr(promise) {
      try {
        await promise
        this.showWaitingText = false
        this.canStopCamera = true
      } catch (error) {
        if (error.name === 'NotAllowedError') {
          // user denied camera access permisson
        } else if (error.name === 'NotFoundError') {
          // no suitable camera device installed
        } else if (error.name === 'NotSupportedError') {
          // page is not served over HTTPS (or localhost)
        } else if (error.name === 'NotReadableError') {
          // maybe camera is already in use
        } else if (error.name === 'OverconstrainedError') {
          // passed constraints don't match any camera. Did you requested the front camera although there is none?
        } else {
          // browser is probably lacking features (WebRTC, Canvas)
        }
      }
    },

    // stop camera
    stopCamera() {
      // load add new profile modal
      this.$emit('showModal')

      //stop camera
      this.renderQr = false

      // stop camera stream
      this.paused = true

      // hide qr modal
      this.$refs.qrModal.hide()
    },

    // start camera
    startCamera() {
      // hide add new profile modal
      this.$emit('hideModal')

      // load waiting text
      this.showWaitingText = true

      // load camera stream
      this.paused = false

      // load camera component
      this.renderQr = true

      // show camera modal
      this.$refs.qrModal.show()
    },

    // get qr code string
    onDecode(decodedString) {
      // parse swisspass string
      let parsedArray = decodedString.split('=')
      this.card.setCardNumber(parsedArray[parsedArray.length - 1])

      // stop camera
      this.stopCamera()
    },

    // open modal with qr reader, if swisspass is selected
    openQrModal() {
      if (this.card.getType() === 0) {
        this.startCamera()
      }
    },
  },
}
</script>
