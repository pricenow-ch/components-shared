<template>
  <v-row dense justify="start">
    <v-col class="col-auto">
      <season-selector
        v-if="showSeasonSelector && availableSeasons && availableSeasons.length"
        outlined
        prefix-icon="fal fa-calendar"
        :seasons="availableSeasons"
        :selected-season-instance="selectedSeasonForSeasonSelector"
        @update:selectedSeasonInstance="onSeasonSelected"
      />
    </v-col>
    <v-col class="col-auto">
      <product-selector
        outlined
        prefix-icon="fal fa-hand-pointer"
        :products-and-events="productsForProductSelector"
        :selected-product-or-event-instance="selectedProductForProductSelector"
        @update:selectedProductOrEventInstance="onProductTypeSelected"
      />
    </v-col>
  </v-row>
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
    showSeasonSelector: {
      type: Boolean,
      required: false,
      default: true,
    },
    loadExtendedProducts: {
      type: Boolean,
      required: false,
      default: false,
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
    selectedSeason() {
      return this.$store.getters.getSelectedSeason()
    },
    // products for the product-selector
    productsForProductSelector() {
      if (this.availableProducts) {
        const flatProducts = this.availableProducts.flat()
        return flatProducts.sort((a, b) => a.getSortOrder() - b.getSortOrder())
      }
      return []
    },

    selectedSeasonForSeasonSelector() {
      if (this.selectedSeason) {
        return this.selectedSeason
      }
      return this.availableSeasons[0]
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
      definitions.permissions.engine.PE_GET_PRICES
    )
    //fetch all seasons for available destinations
    const seasonsInstance = new Seasons()
    await seasonsInstance.loadSeasonsForAllDestinations(destinations)
    this.availableSeasons = seasonsInstance.getAllSeasons()
    //default current selected season to the first one
    this.$store.commit('setSelectedSeason', seasonsInstance.getFirstSeason())
    // fetch all products for each destination from api
    let productsInstance = new Products()
    await productsInstance.loadProductsForAllDestinations(
      destinations,
      true,
      this.loadExtendedProducts
    )

    // todo: can this be done within the Products class?
    const uniqueProducts = _.uniqBy(productsInstance.getProducts(true), 'id')
    const groupedProducts = _.groupBy(uniqueProducts, 'type')

    // sort product type by product id
    this.availableProducts = _.sortBy(groupedProducts, (products) => {
      return products[0].id
    })

    // reset products
    this.$store.commit('setProducts')
    // return first product group as products instance
    if (this.availableProducts.length > 0) {
      this.availableProducts[0].map((productInstance) =>
        this.chosenProducts.addProduct(productInstance)
      )
    }
  },

  methods: {
    onProductTypeSelected(productInstance) {
      // get product type group
      const product = this.availableProducts
        .flat()
        .find((product) => product.getId() === productInstance.getId())

      // reset products
      this.$store.commit('setProducts')
      // transform products into products instance
      this.chosenProducts.addProduct(product)
    },

    onSeasonSelected(seasonInstance) {
      this.onProductTypeSelected(this.selectedProductForProductSelector)
      this.$store.commit('setSelectedSeason', seasonInstance)
    },
  },
}
</script>
