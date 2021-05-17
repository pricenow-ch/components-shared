<!-- navigation component for single views like SingleProductView.vue -->
<template>
  <v-row>
    <v-col class="col-6">
      <v-btn
        :color="color"
        large
        outlined
        :class="{ 'white--text': color !== 'primary' }"
        :min-width="buttonMinWidth"
        @click="goBack()"
      >
        <span class="fal mr-4" :class="icon" />
        {{ buttonText || defaultButtonText }}
      </v-btn>
    </v-col>

    <!-- product title -->
    <v-col class="col-6 text-right headline align-self-center">
      <span class="fal mr-2" :class="iconTitle" />
      {{ title }}
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'TopNavigation',

  props: {
    // route name to go back
    routeName: {
      type: String,
      required: false,
      default: null,
    },

    // alternative to the route name
    // route object
    routeObject: {
      type: Object,
      required: false,
      default: null,
    },

    // title on the top right
    title: {
      type: String,
      required: false,
      default: null,
    },

    // button color
    color: {
      type: String,
      required: false,
      default: 'primary',
    },

    buttonText: {
      type: String,
      required: false,
      default: null,
    },

    buttonMinWidth: {
      type: Number,
      required: false,
      default: null,
    },

    // fa icon (eg. fa-chevron-left)
    icon: {
      type: String,
      required: false,
      default: 'fa-chevron-left',
    },

    iconTitle: {
      type: String,
      required: false,
      default: null,
    },
  },

  computed: {
    defaultButtonText() {
      return this.$t('general.back')
    },
  },

  methods: {
    goBack() {
      if (this.routeObject) this.$router.push(this.routeObject)
      else if (this.routeName) this.$router.push({ name: this.routeName })
      else throw new Error('No routing available!')
    },
  },
}
</script>
