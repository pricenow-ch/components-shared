<template>
  <div>
    <season-selector
      outlined
      prefix-icon='fal fa-hand-pointer'
      :seasons='seasonsForSeasonSelector'
      :selected-season-instance='selectedProductForProductSelector'
      @update:selectedSeasonInstance='onProductTypeSelected'
    />
    <product-selector
      outlined
      prefix-icon='fal fa-hand-pointer'
      :products-and-events='productsForProductSelector'
      :selected-product-or-event-instance='selectedProductForProductSelector'
      @update:selectedProductOrEventInstance='onProductTypeSelected'
    />
  </div>
</template>

<script>
import definitions from '../../../definitions'
import Products from '../../classes-shared/products/Products'
import Seasons from '@/classes-shared/season/Seasons'
import _ from 'lodash'
import ProductSelector from './ProductSelector'
import SeasonSelector from './SeasonSelector'

export default {
  name: 'ProductTypeSelector',
  components: { ProductSelector, SeasonSelector },
  props: {
    preselectedProductId: {
      type: Number,
      required: false,
      default: null,
    },
  },

  data() {
    return {
      // an object of product groups the user has permissions for
      availableProducts: null,
      availableSeasons: null,
    }
  },

  computed: {
    chosenProducts() {
      return this.$store.getters.getProducts()
    },
    // products for the product-selector
    productsForProductSelector() {
      if (this.availableProducts) {
        let tmpProducts = this.availableProducts.map((products) => products[0])
        return tmpProducts.sort((a, b) => a.getSortOrder() - b.getSortOrder())
      }
      return []
    },

    seasonsForSeasonSelector() {
      if (this.availableSeasons) {
        let tmpSeasons = this.availableSeasons.map((seasons) => seasons[0])
        return tmpSeasons
      }
      return []
    },

    selectedSeasonForProductSelector() {
      if (this.chosenProducts) {
        return this.chosenProducts.getFirstProduct()
      }
      return null
    },

    selectedProductForProductSelector() {
      if (this.chosenProducts) {
        return this.chosenProducts.getFirstProduct()
      }
      return null
    },
  },

  async mounted() {
    // todo: https://pricenow.atlassian.net/browse/T1-1043
    // get the userInstance to read the destinations available for the logged in user
    let userInstance = this.$store.getters.getAppUserInstance()
    const destinations = userInstance.userDestinationsInstance.getDestinationsWithPermissions(
      definitions.permissions.engine.PE_GET_PRICES,
    )

    const seasonsInstance = new Seasons()
    await seasonsInstance.loadSeasonsForAllDestinations(destinations)
    this.seasonsForSeasonSelector = seasonsInstance.getSeasons()
    console.log(this.seasonsForSeasonSelector)
    // fetch all products for each destination from api
    let productsInstance = new Products()
    await productsInstance.loadProductsForAllDestinations(destinations, true)

    // todo: can this be done within the Products class?
    const uniqueProducts = _.uniqBy(productsInstance.getProducts(true), 'id')
    let groupedProducts = uniqueProducts.reduce((r, a) => {
      r[a.type] = r[a.type] || []
      r[a.type].push(a)
      return r
    }, Object.create(null))

    // sort product type by product id
    this.availableProducts = _.sortBy(groupedProducts, (products) => {
      return products[0].id
    })

    // reset products
    this.$store.commit('setProducts')
    // return first product group as products instance
    if (this.availableProducts.length > 0) {
      this.availableProducts[0].map((productInstance) =>
        this.chosenProducts.addProduct(productInstance),
      )
    }
  },

  methods: {
    onProductTypeSelected(productInstance) {
      // get product type group
      let products = this.availableProducts.find(
        (productTypeGroup) =>
          productTypeGroup[0].getId() === productInstance.getId(),
      )

      // reset products
      this.$store.commit('setProducts')
      // transform products into products instance
      products.map((productInstance) =>
        this.chosenProducts.addProduct(productInstance),
      )
    },
  },
}
</script>
