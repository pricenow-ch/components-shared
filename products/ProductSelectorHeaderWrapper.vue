<template>
  <div v-show="show">
    <product-type-selector />
  </div>
</template>

<script>
import ProductTypeSelector from './ProductTypeSelector'
export default {
  name: 'ProductSelectorHeaderWrapper',
  components: { ProductTypeSelector },
  data() {
    return {
      // all routes where the product type selector has to be displayed
      routes: ['overviewPricing', 'pricingDetails'],
    }
  },
  computed: {
    show() {
      return this.routes.includes(this.$route.name)
    },
  },
}
</script>
