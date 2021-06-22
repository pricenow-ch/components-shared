<template>
  <div class="filters">
    <!-- attribute filters -->
    <template v-if="multipleFilters">
      <v-autocomplete
        v-if="items.length > 0"
        :key="id ? id : ''"
        ref="autocomplete"
        v-model="selectedItems"
        :clearable="clearable"
        :multiple="multiple"
        outlined
        :class="{ 'pl-4': marginleft }"
        chips
        small-chips
        :label="label"
        :items="items"
        :value="value ? value : null"
        :placeholder="placeholder ? placeholder : ''"
        :filter="customFilter"
        hide-details
        :disabled="disabled"
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
            class="chip--select-multi my-1 mx-1 px-2"
            small
            :disabled="disabled"
            @click="data.select"
            @click:clear="clearedFilter()"
            @click:close="chipClosed(data.item)"
            >{{
              translateFilterValues(data.item[chipLabel]) | capitalize
            }}</v-chip
          >
        </template>
        <template #item="data">
          <v-list-item-content>
            <v-list-item-title
              :inner-html.prop="
                translateFilterValues(data.item[selectLabel]) | capitalize
              "
            ></v-list-item-title>
          </v-list-item-content>
        </template>
      </v-autocomplete>
    </template>

    <!-- for destination filter -->
    <template v-else>
      <v-autocomplete
        v-if="items.length > 0"
        ref="autocomplete"
        v-model="selectedItems"
        :clearable="clearable"
        :multiple="multiple"
        outlined
        chips
        small-chips
        :label="label"
        :items="items"
        item-text="slug"
        item-value="id"
        return-object
        :value="value ? value : null"
        :class="{ 'pl-4': marginleft }"
        :filter="customFilter"
        :placeholder="placeholder ? placeholder : ''"
        hide-details
        :disabled="disabled"
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
            class="chip--select-multi"
            :disabled="disabled"
            @click="data.select"
            @click:clear="clearedFilter()"
            @click:close="chipClosed(data.item)"
            >{{ data.item.getName() }}</v-chip
          >
        </template>
        <template #item="data">
          <v-list-item-content>
            <v-list-item-title
              :inner-html.prop="data.item.getName()"
            ></v-list-item-title>
          </v-list-item-content>
        </template>
      </v-autocomplete>
    </template>
  </div>
</template>

<script>
export default {
  name: 'PricingFilter',
  props: {
    items: {
      type: Array,
      required: true,
    },
    multiple: {
      type: Boolean,
      required: false,
      default: false,
    },
    multipleFilters: {
      type: Boolean,
      required: false,
      default: false,
    },
    clearable: {
      type: Boolean,
      required: false,
      default: false,
    },
    label: {
      type: String,
      required: false,
      default: '',
    },
    value: {
      type: [Array, Object],
      required: false,
      default: () => [],
    },
    chipLabel: {
      type: String,
      required: false,
      default: 'label',
    },
    selectLabel: {
      type: String,
      required: false,
      default: 'label',
    },
    translate: {
      type: Boolean,
      required: false,
      default: true,
    },
    placeholder: {
      type: String,
      required: false,
      default: '',
    },
    values: {
      type: String,
      required: false,
      default: '',
    },
    id: {
      type: String,
      required: false,
      default: '',
    },
    marginleft: {
      type: Boolean,
      required: true,
    },
    disabled: {
      type: Boolean,
      required: false,
      default: false,
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

    /**
     * Emits an event on the pricing-filter component
     * @caller v-btn-toggle component pricing-filter
     * @param {object} selectedItem
     * @emits {event} change, payload: selectedItem
     */
    setItemSelection(selectedItem) {
      this.$emit('change', selectedItem)
    },

    translateFilterValues(key) {
      if (this.translate) {
        /* global i18n */
        return i18n.t(key)
      }
      return key
    },
    clearedFilter() {
      this.$emit('cleared')
    },
    chipClosed(closedItem) {
      let index = -1
      if (this.multipleFilters) {
        index = this.selectedItems.indexOf(closedItem.value)
      } else {
        index = this.selectedItems.indexOf(closedItem)
      }

      if (index >= 0) {
        this.selectedItems.splice(index, 1)
      }
      this.$emit('removed-chip', closedItem)
    },
    customFilter(item, queryText, itemText) {
      if (this.multipleFilters) {
        itemText = this.translateFilterValues(item.translation).toLowerCase()
      } else {
        itemText = this.translateFilterValues(item.slug).toLowerCase()
      }
      queryText = queryText.toLowerCase()
      return itemText.indexOf(queryText) > -1
    },
  },
}
</script>
