<template>
  <v-autocomplete
    :value="country"
    :items="countryList"
    :rules="getRules()"
    :label="required ? $t('SCountry.countryMandatory') : $t('SCountry.country')"
    :item-value="'code'"
    :item-text="'native'"
    validate-on-blur
    persistent-hint
    :outlined="outlined"
    @input="
      (country) => {
        this.$emit('update:country', country)
      }
    "
    @blur="$emit('update:country', country)"
  />
</template>

<script>
import { countries } from 'countries-list'

export default {
  name: 'SCountry',

  props: {
    // decide whether a country has to bee selected
    required: {
      type: Boolean,
      required: false,
      default: false,
    },

    // preset country
    country: {
      required: false,
      type: Object,
      default: null,
    },

    outlined: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  data() {
    return {}
  },

  computed: {
    countryList() {
      let list = []

      Object.entries(countries).forEach((country) => {
        list.push({
          name: country[1].name,
          native: country[1].native,
          code: country[0],
        })
      })

      // set custom countries at the beginning (switzerland, netherlands, germany, belgium, austria, france, italy, lichtenstein)
      let firstSelectCountryCodes = [
        'CH',
        'DE',
        'NL',
        'BE',
        'FR',
        'IT',
        'AT',
        'LI',
      ]

      // get the first selected country objects
      let firstSelectedCountryObjects = []
      firstSelectCountryCodes.forEach((countryCode) => {
        firstSelectedCountryObjects.push(
          list.find((country) => {
            return country.code === countryCode
          })
        )
      })

      // all countries without the first selected
      let countryListWithoutFirstSelectedCountries = list.filter((country) => {
        return !firstSelectCountryCodes.includes(country.code)
      })

      // merge the countries
      return [
        ...firstSelectedCountryObjects,
        ...countryListWithoutFirstSelectedCountries,
      ]
    },
  },

  methods: {
    getRules() {
      if (this.required) {
        return [(v) => !!v || this.$t('SCountry.countryIsMandatory')]
      } else {
        return []
      }
    },
  },
}
</script>
