<template>
  <div>
    <v-row>
      <v-col class="col-12 px-0 py-1">
        <v-select
          v-if="!hideTimePeriod"
          v-model="selectedTimePeriod"
          :items="timePeriodValues"
          :label="$t('priceBulkManagement.repeat')"
          outlined
          item-text="display"
          item-value="value"
          :rules="[(v) => (!!v && v.length > 0) || $t('rule.inputRequired')]"
          hide-details
          :dense="dense"
        />
      </v-col>
      <v-col class="col-12 pb-1 px-0">
        {{ $t('priceBulkManagement.repeatEvery') }}
      </v-col>
    </v-row>
    <v-row v-if="selectedTimePeriod === 'months'">
      <v-col class="col-6">
        <v-text-field
          v-model="everyMonthCtr"
          type="number"
          append-outer-icon="add"
          prepend-icon="remove"
          @click:append-outer="increment"
          @click:prepend="decrement"
        >
        </v-text-field>
      </v-col>
      <v-col class="col-6 d-flex align-center">
        <span>
          {{ $t('priceBulkManagement.repeatEveryWeekday') }}
        </span>
      </v-col>
    </v-row>

    <v-row class="justify-flex-start px-0">
      <v-col
        v-for="weekday of weekdays"
        :key="weekday.value"
        class="col-auto px-1 pt-0"
        :style="{ width: '50px' }"
      >
        <v-checkbox
          v-model="selectedWeekdays"
          :value="weekday.value"
          hide-details
          class="mt-0"
          color="primary"
        >
          <template #label>
            <span class="content-4 font-weight-thin font-condensed">
              {{ weekday.name }}
            </span>
          </template>
        </v-checkbox>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import * as moment from 'moment'
import DateRange from '@/classes/dates/DateRange'
// plugin for momentjs
require('moment-recur')

export default {
  name: 'RuleBasedDateSelector',

  props: {
    dateRange: {
      type: DateRange,
      required: true,
    },
    // hide the dropdown to switch between weekly and monthly
    hideTimePeriod: {
      type: Boolean,
      required: false,
      default: false,
    },
    dense: {
      type: Boolean,
      required: false,
      default: true,
    },
  },

  data() {
    return {
      selectedTimePeriod: 'weeks',
      timePeriodValues: [
        {
          display: i18n.t('priceBulkManagement.weekly'),
          value: 'weeks',
        },
        {
          display: i18n.t('priceBulkManagement.monthly'),
          value: 'months',
        },
      ],
      weekdays: [
        { name: 'Mo', value: 1 },
        { name: 'Di', value: 2 },
        { name: 'Mi', value: 3 },
        { name: 'Do', value: 4 },
        { name: 'Fr', value: 5 },
        { name: 'Sa', value: 6 },
        { name: 'So', value: 7 },
      ],
      everyMonthCtr: 1,
      // https://momentjs.com/docs/#/manipulating/add/
      selectedWeekdays: [],
      selectedDates: [],
    }
  },

  watch: {
    selectedTimePeriod() {
      this.calculateSelectedDates()
    },
    selectedWeekdays() {
      this.calculateSelectedDates()
    },
    dateRange: {
      handler() {
        this.calculateSelectedDates()
      },
      deep: true,
    },
  },

  methods: {
    increment() {
      const val = parseInt(this.everyMonthCtr, 10)
      if (val < 5) {
        this.everyMonthCtr = val + 1
      }
      this.calculateSelectedDates()
    },
    decrement() {
      const val = parseInt(this.everyMonthCtr, 10)
      if (val > 1) {
        this.everyMonthCtr = val - 1
      }
      this.calculateSelectedDates()
    },
    calculateSelectedDates() {
      this.selectedDates = []
      if (this.selectedWeekdays.length > 0 && this.selectedTimePeriod) {
        const startMom = this.dateRange.getStartDateMoment()
        const endMom = this.dateRange.getEndDateMoment()
        if (this.selectedTimePeriod === 'weeks') {
          this.selectedDates = this.addWeekly(startMom, endMom)
        } else {
          this.selectedDates = this.addMonthly(startMom, endMom)
        }
      }
      this.$emit('changed', this.selectedDates)
    },
    addWeekly(start, end) {
      let occurrences = []
      this.selectedWeekdays.forEach((weekdayNr) => {
        let recurrence = moment(start)
          .recur(moment(end).add(1, 'd'))
          .every(weekdayNr)
          .daysOfWeek()

        occurrences = occurrences.concat(
          recurrence.all().map((dateEl) => {
            return moment(dateEl).format('YYYY-MM-DD')
          })
        )
      })

      return occurrences
    },
    addMonthly(start, end) {
      let occurrences = []

      this.selectedWeekdays.forEach((weekdayNr) => {
        let recurrence = moment(start)
          .recur(moment(end).add(1, 'd'))
          .every(weekdayNr)
          .daysOfWeek()
          .every(this.everyMonthCtr - 1)
          .weeksOfMonthByDay()

        occurrences = occurrences.concat(
          recurrence.all().map((dateEl) => {
            return moment(dateEl).format('YYYY-MM-DD')
          })
        )
      })

      return occurrences
    },
  },
}
</script>
