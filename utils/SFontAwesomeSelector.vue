<template>
  <v-autocomplete
    v-model="selectedIcon"
    outlined
    :rules="[(v) => !!v || $t('singleEventView.iconRequired')]"
    :items="filteredIcons"
    :filter="iconFilter"
    :label="$t('fontAwesomeSelect.selectLabel')"
    hide-details
  >
    <template #selection="data">
      <font-awesome-icon v-if="data.item" :icon="data.item.iconName" />
    </template>

    <template #item="data">
      <v-list-item-avatar>
        <font-awesome-icon v-if="data.item" :icon="data.item.iconName" />
      </v-list-item-avatar>
      <v-list-item-content>
        <v-list-item-title>{{ data.item.iconName }}</v-list-item-title>
      </v-list-item-content>
    </template>
  </v-autocomplete>
</template>

<script>
// https://github.com/drmovi/vue-icon-picker/blob/master/src/vue-icon-picker.vue
// stolen ;) but better to integrate it directly to vuetify and not depending on external lib
import { library } from '@fortawesome/fontawesome-svg-core'
import { fas } from '@fortawesome/free-solid-svg-icons'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
library.add(fas)

export default {
  name: 'SFontAwesomeSelect',

  components: {
    FontAwesomeIcon,
  },

  props: { value: { type: String, default: '', required: false } },

  data() {
    return {
      search: '',
      selectedIcon: null,
      icons: { ...fas },
    }
  },

  computed: {
    filteredIcons: function () {
      return Object.keys(this.icons)
        .filter(
          (key) =>
            key !== 'faFontAwesomeLogoFull' &&
            this.icons[key].iconName.includes(this.search)
        )
        .map((key) => this.icons[key])
    },
  },

  watch: {
    selectedIcon() {
      this.$emit('input', this.selectedIcon)
    },
  },

  mounted() {
    if (this.value) {
      this.selectedIcon = this.value
    }
  },

  methods: {
    iconFilter(item, queryText) {
      return item.iconName.includes(queryText)
    },
  },
}
</script>
