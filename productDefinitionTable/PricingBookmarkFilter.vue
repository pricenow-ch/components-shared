<template>
  <v-col>
    <v-autocomplete
      v-if="items.length > 0"
      ref="autocomplete"
      v-model="selectedItems"
      clearable
      multiple
      outlined
      chips
      small-chips
      :label="$t('bookmarks.bookmarkFilter')"
      :items="items"
      hide-details
      return-object
      item-value="name"
      item-text="name"
      dense
      @change="setItemSelection"
      @click:clear="clearedFilter()"
      @click:append="closeFilter()"
    >
      <template #selection="data">
        <v-chip
          v-bind="data.attrs"
          :input-value="data.selected"
          close
          small
          class="chip--select-multi my-1 mx-1 px-2"
          @click="data.select"
          @click:clear="clearedFilter()"
          @click:close="chipClosed(data)"
        >
          {{ data.item.name }}
        </v-chip>
      </template>
      <template #item="data">
        <v-list-item-content>
          <v-list-item-title
            :inner-html.prop="data.item.name"
          ></v-list-item-title>
        </v-list-item-content>
      </template>
    </v-autocomplete>
  </v-col>
</template>

<script>
export default {
  name: 'PricingBookmarkFilter',
  props: {
    items: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      selectedItems: null,
    }
  },
  methods: {
    closeFilter() {
      let autocomplete = this.$refs.autocomplete

      if (autocomplete.isMenuActive) {
        autocomplete.isMenuActive = false
        autocomplete.blur()
      } else {
        autocomplete.isMenuActive = true
        autocomplete.focus()
      }
    },
    setItemSelection(selectedItems) {
      let prodDefIds = []
      selectedItems.forEach((bookmark) => {
        prodDefIds = prodDefIds.concat(bookmark.prodDefIds)
      })
      this.$emit('change', prodDefIds)
    },

    clearedFilter() {
      this.$emit('cleared')
    },
    chipClosed(closedObj) {
      this.selectedItems.splice(closedObj.index, 1)
      this.$emit('removed-chip', closedObj.item.prodDefIds)
    },
    reset() {
      this.selectedItems = null
    },
  },
}
</script>
