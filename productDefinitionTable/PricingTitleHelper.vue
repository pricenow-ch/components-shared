<template>
  <v-card-title class="py-0">
    <v-row class="align-center">
      <v-col class="col-auto pt-5">
        <slot />
      </v-col>
    </v-row>
  </v-card-title>
</template>

<script>
export default {
  name: 'PricingTitleHelper',
  props: {
    filterExpanded: {
      type: Boolean,
      required: true,
    },
  },
}
</script>
