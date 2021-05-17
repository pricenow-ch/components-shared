<template>
  <v-menu
    ref="birthdayMenu"
    v-model="menu"
    open-on-click
    :close-on-content-click="false"
    transition="scale-transition"
    offset-y
    min-width="290px"
    :nudge-right="40"
  >
    <template #activator="{ on }">
      <v-text-field
        ref="textField"
        v-mask="mask"
        :value="$options.filters.formatDate(date)"
        :label="label"
        :rules="rules"
        validate-on-blur
        prepend-icon="event"
        :outlined="outlined"
        @input="setDate"
        v-on="on"
      />
    </template>
    <v-date-picker
      ref="picker"
      :locale="$store.getters.getLanguageLocale()"
      scrollable
      no-title
      :value="date"
      :first-day-of-week="1"
      :max="new Date().toISOString().substr(0, 10)"
      @input="
        (date) => {
          $emit('update:date', date)
        }
      "
      @change="save"
    />
  </v-menu>
</template>

<script>
import { mask } from 'vue-the-mask'
import moment from 'moment'

export default {
  name: 'SBirthdayPicker',

  directives: {
    mask,
  },

  props: {
    date: {
      required: true,
      default: null,
      type: [Date, String],
    },
    // is the birthdate field mandatory or not
    mandatory: {
      type: Boolean,
      required: false,
      default: true,
    },
    outlined: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  data() {
    return {
      menu: false,
      regexRule: /^([0][1-9]|[1-2][0-9]|[3][0-1]).([0][1-9]|[1][0-2]).([1][9][0-9][0-9]|[2][0-3][0-9][0-9])$/,
      mask: '##.##.####',
      userMadeInput: false,
    }
  },

  computed: {
    rules() {
      if (this.mandatory || this.userMadeInput) {
        return [
          (v) => !!v || this.$t('addNewProfile.birthdayRequired'),
          (v) =>
            this.regexRule.test(v) ||
            this.$t('addNewProfile.wrongBirthdayFormat'),
        ]
      }
      return []
    },
    label() {
      if (this.mandatory) return this.$t('myProfiles.birthday')
      return this.$t('general.birthdate')
    },
  },

  watch: {
    menu(val) {
      val && setTimeout(() => (this.$refs.picker.activePicker = 'YEAR'))
    },
  },

  methods: {
    save(date) {
      this.$refs.birthdayMenu.save(date)

      // hack for proper validation
      this.$refs.textField.focus()
    },

    // set the date get from user input
    setDate(userInput) {
      this.userMadeInput = userInput || false
      if (this.regexRule.test(userInput)) {
        // fix 24.03.2020: manually parse date for firefox
        let splitedDate = userInput.split('.')
        let newDate = moment(
          splitedDate[2] + '-' + splitedDate[1] + '-' + splitedDate[0]
        ).format('YYYY-MM-DD')
        if (newDate === 'Invalid date') {
          this.$emit('update:date', null)
        } else {
          this.$emit('update:date', newDate)
          this.$refs.birthdayMenu.save(newDate)
        }
      }
    },
  },
}
</script>
