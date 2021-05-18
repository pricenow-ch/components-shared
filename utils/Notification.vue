<template>
  <div v-show="notify" class="notification-container">
    <v-alert
      v-model="notify"
      dense
      elevation="2"
      transition="scale-transition"
      class="notification"
      dismissible
      :type="type"
    >
      {{ text }}
      <span v-html="htmlText" />
    </v-alert>

    <s-modal
      ref="modal"
      :size="1"
      :persistent="persistent"
      hide-cancel-button
      @ok="emitOk()"
      @hide="emitCancel()"
    >
      <v-layout class="wrap">
        <v-flex class="xs12">
          <div class="content-3 text-center">
            {{ text }}
            <span v-html="htmlText" />
          </div>
        </v-flex>
      </v-layout>
    </s-modal>
  </div>
</template>

<script>
export default {
  name: 'Notification',

  data() {
    return {
      text: '',
      type: 'success',
      notify: false,
      htmlText: '',
      persistent: false,
      uniqueIdentifier: null,
    }
  },

  mounted() {
    /* global EventBus */
    EventBus.$on(
      'notify',
      (
        text = this.$t('notify.standardError'),
        type = 'error',
        config = {
          duration: 3000,
          htmlText: '',
          uniqueIdentifier: null,
          noModal: false,
        }
      ) => {
        this.text = text
        this.type = type
        this.htmlText = config.hasOwnProperty('htmlText') ? config.htmlText : ''
        this.persistent = config.hasOwnProperty('persistent')
          ? config.persistent
          : false

        // already such an error message open?
        if (
          !config.uniqueIdentifier ||
          config.uniqueIdentifier != this.uniqueIdentifier
        ) {
          // if it is null or previous identifier is different from the current.
          // setting unique identifier
          // identifier, to avoid multiple errors popping up with the same message
          this.uniqueIdentifier = config.hasOwnProperty('uniqueIdentifier')
            ? config.uniqueIdentifier
            : null

          // type is error
          if (type === 'error' && !config.noModal) {
            this.$refs.modal.show()
          } else {
            this.notify = true
            setTimeout(
              () => {
                this.notify = false
              },
              config.hasOwnProperty('duration') ? config.duration : 3000
            )
          }
        }
      }
    )
  },

  beforeDestroy() {
    /* global EventBus */
    EventBus.$off('notify')
  },

  methods: {
    emitOk() {
      // reset unique identifier
      this.uniqueIdentifier = null

      /* global EventBus */
      EventBus.$emit('notification:ok')
    },

    emitCancel() {
      // reset unique identifier
      this.uniqueIdentifier = null

      /* global EventBus */
      EventBus.$emit('notification:hide')
    },
  },
}
</script>

<style lang="scss">
.notification-container {
  top: 15px;
  margin: auto;
  left: 0;
  right: 0;
  position: fixed;
  width: min-content;
  z-index: 8;
}
.notification {
  margin: auto;
  height: 40px;
  width: max-content;
}

@media screen and (max-width: $sm) {
  .notification {
    top: $header-height-xs;
  }
}
</style>
