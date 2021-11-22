<template>
  <div>
    <v-menu
      ref="datePicker"
      v-model="menu"
      :nudge-right="40"
      transition="scale-transition"
      min-width="290px"
      :close-on-content-click="false"
    >
      <template #activator="{ on }">
        <v-text-field
          :value="$options.filters.formatDate(date)"
          :label="label"
          :rules="rules"
          append-icon="event"
          readonly
          outlined
          :disabled="disabled"
          :hide-details="hideDetails"
          :dense="dense"
          @click:append="invertMenu()"
          v-on="on"
        />
      </template>
      <v-date-picker
        :value="date"
        :min="min"
        :max="max"
        color="primary"
        :locale="$store.getters.getLanguageLocale()"
        :first-day-of-week="1"
        :allowed-dates="allowedDatesFunction"
        @input="
          (date) => {
            $emit('update:date', date), (menu = false)
          }
        "
      />
    </v-menu>
  </div>
</template>

<script>
import moment from 'moment-timezone'

export default {
  name: 'SDatePicker',

  props: {
    label: {
      required: true,
      type: String,
      default: null,
    },
    rules: {
      required: false,
      type: Array,
      default: () => {
        return []
      },
    },

    // format: YYYY-MM-DD || [YYYY-MM-DD]
    date: {
      required: true,
      type: [Date, String],
      default: new Date(),
    },

    // date iso string
    min: {
      type: String,
      required: false,
      default: undefined,
    },

    // date iso string
    max: {
      type: String,
      required: false,
      default: undefined,
    },

    // Array with selectable dates
    allowedDates: {
      type: Array,
      required: false,
      default: () => [],
    },

    // disable date picker
    disabled: {
      type: Boolean,
      required: false,
      default: false,
    },

    // hide error texts beneath the input field
    hideDetails: {
      type: Boolean,
      required: false,
      default: false,
    },
    // vuetify standard
    dense: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  data() {
    return {
      menu: false,
      isFirstClick: true,
    }
  },

  mounted() {
    this.setFirstClickStartDate()
  },

  methods: {
    allowedDatesFunction(val) {
      // allowing all dates if array is empty
      if (!this.allowedDates.length) return true

      // restrict allowed dates
      return this.allowedDates.indexOf(val) !== -1
    },

    getDateInstance() {
      let year = parseInt(this.date.slice(0, 4))
      let month = parseInt(this.date.slice(5, 7))
      let date = parseInt(this.date.slice(8, 10))
      return new Date(year, month - 1, date)
    },

    setFirstClickStartDate() {
      if (this.isFirstClick && this.min && !this.date) {
        this.isFirstClick = false
        this.$emit('update:date', moment(this.min).format('YYYY-MM-DD'))
      }
    },
    invertMenu() {
      this.menu = !this.menu
    },
  },
}
</script>
