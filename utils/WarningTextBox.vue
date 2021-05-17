<template>
  <div v-if="showWarningText" class="warning-banner d-flex align-center">
    <v-row class="no-gutters px-4 px-sm-6 px-lg-0">
      <v-col class="py-0">
        <v-row class="no-gutters">
          <v-col class="pa-0">
            <warning-text justify="">{{ $t(warningText) }}</warning-text>
          </v-col>
          <v-col
            class="text-right clickable col-1 align-self-center"
            @click="toggleWarningText()"
          >
            <span class="icon-small">
              <span class="primary--text fa fa-times" />
            </span>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import WarningText from '@/components/shop/helper/WarningText'

export default {
  name: 'WarningTextBox',
  components: { WarningText },
  data() {
    return {
      showWarningText: false,
      warningText: null,
    }
  },

  mounted() {
    // listen whether to show warning text
    EventBus.$on('ShopModule.ReloadWarningText', (warningText) => {
      if (typeof warningText === 'undefined')
        this.toggleWarningText(null, false)
      else this.toggleWarningText(warningText, true)
    })
  },

  methods: {
    toggleWarningText(
      text = this.warningText,
      toggleTo = !this.showWarningText
    ) {
      if (toggleTo !== this.showWarningText || text !== this.warningText) {
        this.warningText = text
        this.showWarningText = toggleTo
        EventBus.$emit('WarningTextBox.heightChanged', this.showWarningText)
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.warning-banner {
  width: 100%;
  margin: 0;
  background-color: #93cceb;
}

.centered {
  padding-left: 5%;
  padding-right: 5%;
}

@media screen and (max-width: $lg) {
  .centered {
    padding-left: 0px;
    padding-right: 0px;
  }
}

@media screen and (max-width: $sm) {
  .centered {
    padding-left: 8px;
    padding-right: 8px;
  }
}
</style>
