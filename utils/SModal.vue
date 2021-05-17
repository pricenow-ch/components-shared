<template>
  <div>
    <v-dialog
      v-model="modal"
      :max-width="getSize()"
      :persistent="persistent"
      :scrollable="scrollable"
      overlay-color="white"
      :overlay-opacity="0.9"
      :fullscreen="fullscreen"
    >
      <v-card class="white px-0">
        <!-- title -->
        <v-card-title
          v-if="headerText !== null"
          class="headline px-0 pt-0"
          :class="classHeader"
        >
          <v-row class="justify-space-around" style="width: 100%">
            <v-col class="headline-standard py-0 word-no-wrap">
              {{ headerText }}
            </v-col>

            <!-- close cross -->
            <!-- emits hide via v-dialog @update:returnValue emit, see up -->
            <v-col
              v-if="showCancelHeader"
              class="text-right error--text clickable"
              @click="hide()"
            >
              <span class="fal fa-times" />
            </v-col>
          </v-row>
        </v-card-title>

        <v-card-text class="px-0" style="overflow-x: hidden">
          <slot>
            <!-- content goes here -->
          </slot>
        </v-card-text>

        <v-card-actions v-if="!hideButtons" style="overflow-x: hidden">
          <v-row justify-center class="no-gutters">
            <!-- cancel button -->
            <v-col
              v-if="!hideCancelButton"
              class="text-center d-flex justify-start"
              :class="cancelButtonClass"
            >
              <v-btn
                outlined
                :large="!smallButtons"
                :color="cancelColor"
                @click="cancel()"
              >
                <span :class="[cancelIcon]" />
                {{ getCancelText() }}
              </v-btn>
            </v-col>

            <!-- ok button -->
            <v-col
              v-if="!hideOkButton"
              class="text-center d-flex justify-end"
              :class="okButtonClass"
            >
              <v-btn
                outlined
                :large="!smallButtons"
                :color="okColor"
                :disabled="okButtonDisabled"
                @click="ok()"
              >
                <span :class="[okIcon]" />
                {{ getOkText() }}
              </v-btn>
            </v-col>
          </v-row>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  name: 'SModal',

  props: {
    size: {
      type: Number,
      required: false,
      default: 0,
    },
    headerText: {
      type: String,
      required: false,
      default: null,
    },
    cancelText: {
      type: String,
      required: false,
      default: null,
    },
    okText: {
      type: String,
      required: false,
      default: null,
    },

    // don't close modal after clicking ok
    hideOnOk: {
      type: Boolean,
      required: false,
      default: true,
    },

    // don't close modal after clicking cancel
    hideOnCancel: {
      type: Boolean,
      required: false,
      default: true,
    },

    hideButtons: {
      type: Boolean,
      required: false,
      default: false,
    },

    // disable ok button
    okButtonDisabled: {
      type: Boolean,
      required: false,
      default: false,
    },

    persistent: {
      type: Boolean,
      required: false,
      default: false,
    },
    hideOkButton: {
      type: Boolean,
      required: false,
      default: false,
    },
    hideCancelButton: {
      type: Boolean,
      required: false,
      default: false,
    },

    // add cancel header to modal
    showCancelHeader: {
      type: Boolean,
      required: false,
      default: false,
    },

    // style classes for the modal header
    classHeader: {
      type: String,
      required: false,
      default: '',
    },

    // style classes for the ok button
    okButtonClass: {
      type: String,
      required: false,
      default: 'xs6',
    },

    // style classes for the cancel button
    cancelButtonClass: {
      type: String,
      required: false,
      default: 'xs6',
    },

    // button or class for cancel button
    cancelIcon: {
      type: String,
      required: false,
      default: '',
    },

    okIcon: {
      type: String,
      required: false,
      default: '',
    },

    // style classes for the header text
    classHeaderText: {
      type: String,
      required: false,
      default: '',
    },

    // style class on ok button
    okColor: {
      type: String,
      required: false,
      default: 'primary',
    },

    cancelColor: {
      type: String,
      required: false,
      default: 'secondary',
    },

    // content is scrollable while buttons aren't
    scrollable: {
      type: Boolean,
      required: false,
      default: true,
    },

    // make buttons small
    smallButtons: {
      type: Boolean,
      required: false,
      default: false,
    },
    // passes fullscreen prop to vuetify's v-dialog
    // https://vuetifyjs.com/en/api/v-dialog/#props
    fullscreen: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  data() {
    return {
      modal: false,
    }
  },

  methods: {
    // click on ok button
    // hide modal and emit ok to parent component
    ok() {
      if (this.hideOnOk) {
        this.hide()
      }
      this.$emit('ok')
    },

    // click on cancel button
    // hide modal and emit cancel to parent
    cancel() {
      if (this.hideOnCancel) {
        this.hide()
      }
      this.$emit('cancel')
    },

    getCancelText() {
      return this.cancelText === null
        ? this.$t('general.cancel')
        : this.cancelText
    },

    getOkText() {
      return this.okText === null ? this.$t('general.ok') : this.okText
    },

    isDisplayed() {
      return this.modal
    },

    // get the size in px
    getSize() {
      switch (this.size) {
        case 0:
          return 310
        case 1:
          return 500
        case 2:
          return 700
        case 3:
          return 1200
        default:
          return 290
      }
    },

    // display the modal
    show() {
      this.modal = true
    },

    // emits 'hide' via @update:returnValue
    hide() {
      this.modal = false
    },

    // hide the modal
    emitHide() {
      this.$emit('hide')
    },
  },
}
</script>

<style lang="scss" scoped>
>>> .v-dialog {
  border: none;
  .v-card {
    padding: 1rem;
  }
}

.color-primary {
  color: var(--v-primary-base);
}
</style>
