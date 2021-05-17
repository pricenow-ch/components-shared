<template>
  <v-dialog v-model="isActive" width="300" hide-overlay persistent>
    <v-card color="primary" dark>
      <v-card-text class="text-center">
        {{ text }}
        <v-progress-linear indeterminate color="white" class="mb-0" />
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: 'Spinner',

  data() {
    return {
      text: '',
      stack: [],
      isActive: false,
    }
  },

  watch: {
    stack: {
      handler() {
        if (this.stack.length > 0) {
          this.isActive = true
        } else {
          EventBus.$emit('spinner:hidden')
          this.isActive = false
        }
      },
      deep: true,
    },
  },

  mounted() {
    /* global EventBus */
    EventBus.$on('spinnerShow', (text = this.$t('spinner.pleaseWait')) => {
      this.stack.push(text)
      this.text = text
    })

    EventBus.$on('spinnerHide', () => {
      this.stack.pop()
    })
  },

  beforeDestroy() {
    /* global EventBus */
    EventBus.$off('spinnerShow')
    EventBus.$off('spinnerHide')
  },
}
</script>
