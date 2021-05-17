<template>
  <v-menu bottom offset-y>
    <template #activator="{ on }">
      <!-- multiselect -->
      <v-row
        v-if="multiSelect"
        class="clickable border border-radius-standard"
        v-on="on"
      >
        <!-- loading products -->
        <v-col v-if="productsAndEvents.length === 0" class="col-10">
          <span class="fa fa-circle-notch fa-spin mr-2" />
          {{ $t('productSelector.loadingProducts') }}
        </v-col>

        <!-- no product selected -->
        <v-col v-else-if="selectedProducts.length === 0" class="col-10">
          {{ $t('shopBookingCalendar.selectProducts') }}
        </v-col>
        <!-- iterate selected products -->
        <v-col v-else class="col-10">
          <v-chip
            v-for="(selectedProduct, selectedProductIndex) in selectedProducts"
            :key="selectedProductIndex"
            class="clickable mr-1 mb-1"
            @click.stop.capture="removeSelectedProduct(selectedProduct)"
          >
            {{ selectedProduct.getTitle() }}
            <span class="fa fa-times ml-1" />
          </v-chip>
        </v-col>

        <v-col class="col-2 text-right">
          <span class="fal fa-chevron-down" />
        </v-col>
      </v-row>

      <!-- Button -->
      <v-btn
        v-else
        large
        :outlined="outlined"
        :style="{ backgroundColor: outlined ? 'white' : '' }"
        color="primary"
        :loading="productsAndEvents.length === 0"
        max-width="100%"
        v-on="on"
      >
        <!-- prefix icon -->
        <span v-if="prefixIcon" :class="prefixIcon" class="mr-2" />
        <div v-if="selectedProductOrEventInstance">
          {{
            displayProductEvent(selectedProductOrEventInstance)
              | maxLength(computedMaxProductNameLength)
          }}
        </div>
        <span v-else>
          {{ $t('createBooking.selectProductOrEvent') }}
        </span>
      </v-btn>
    </template>
    <v-list subheader class="scrollable-list">
      <v-list-item
        v-for="(productOrEvent, productOrEventIndex) in productsAndEvents"
        :key="productOrEventIndex"
        @click="clickedProductOrEvent(productOrEvent)"
      >
        <v-list-item-content
          :set="(mySubheader = subheader(productOrEventIndex))"
          class="py-0"
          :class="{ 'mt-3': productOrEventIndex === 0 }"
        >
          <v-list-item-subtitle
            v-if="mySubheader"
            inset
            :class="{ 'mt-4': productOrEventIndex > 0 }"
          >
            {{ mySubheader }}
          </v-list-item-subtitle>
          <v-divider v-if="mySubheader" class="mb-2" />

          <v-list-item-title
            :class="{
              'primary white--text': selectedProducts.find(
                (selectedProduct) =>
                  selectedProduct.getId() === productOrEvent.getId()
              ),
            }"
          >
            <v-row>
              <v-col class="col-12 ml-2">
                <span v-if="$store.getters.getExtTransLoaded()">
                  {{ displayProductEvent(productOrEvent) }}
                </span>
                <span v-else class="fal fa-circle-notch fa-spin" />
              </v-col>
            </v-row>
          </v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-list>
  </v-menu>
</template>

<script>
import Event from '../../classes/events/Event'
import Product from '../../classes/products/Product'

export default {
  name: 'ProductSelector',
  props: {
    // An array with all products and events to display
    productsAndEvents: {
      type: Array,
      required: true,
    },

    // you can make use of two modes:
    // 1.) single product or event mode (multiSelect: false): Only one product can be selected. In this case, use the "selectedProductOrEveventInstanc" prop in your parent component
    // 2.) multiple product mode (multiSelect: true): Multiple products can be selected. Use the "selectedProducts" prop in your parent component (Array with products instances)
    multiSelect: {
      type: Boolean,
      required: false,
      default: false,
    },

    selectedProductOrEventInstance: {
      type: [Product, Event],
      required: false,
      default: null,
    },

    selectedProducts: {
      type: Array,
      required: false,
      default: () => [],
    },

    // button style
    outlined: {
      type: Boolean,
      required: false,
      default: false,
    },

    // prefix icon
    prefixIcon: {
      type: String,
      required: false,
      default: null,
    },

    // max string length for the product name
    limitProductName: {
      type: Boolean,
      required: false,
      default: false,
    },
    maxProductNameLength: {
      type: Number,
      required: false,
      default: null,
    },
  },

  computed: {
    computedMaxProductNameLength() {
      if (this.maxProductNameLength) return this.maxProductNameLength
      if (this.limitProductName) {
        let breakpoint = this.$vuetify.breakpoint
        if (breakpoint.xl) return 28
        if (breakpoint.lgAndUp) return 20
        if (breakpoint.mdAndUp) return 15
        if (breakpoint.smAndUp) return 10
      }
      return 500
    },
  },

  methods: {
    clickedProductOrEvent(productOrEvent) {
      if (this.multiSelect) {
        // in multiselect no event should be passed
        if (productOrEvent instanceof Event)
          throw new Error('No event expected in multi select mode!')
        let indexOfClickedProduct = this.selectedProducts.indexOf(
          productOrEvent
        )
        let newSelectedProducts = []
        if (indexOfClickedProduct > -1) {
          // remove clicked product, because it already lives in the array
          newSelectedProducts = this.selectedProducts.filter(
            (selectedProduct) =>
              selectedProduct.getId() !== productOrEvent.getId()
          )
        } else {
          // add clicked product
          newSelectedProducts = [...this.selectedProducts, productOrEvent]
        }
        this.$emit('update:selectedProducts', newSelectedProducts)
      } else {
        this.$emit('update:selectedProductOrEventInstance', productOrEvent)
      }
    },

    removeSelectedProduct(product) {
      if (!this.multiSelect)
        throw new Error(
          'The method removeSelectedproduct() is only supported in multiselect mode so far.'
        )
      let newSelectedProducts = this.selectedProducts.filter(
        (selectedProduct) => selectedProduct.getId() !== product.getId()
      )
      this.$emit('update:selectedProducts', newSelectedProducts)
    },

    // helper method to display the products or events name in the dropdown
    displayProductEvent(item) {
      if (item instanceof Event) return item.title
      return this.$t(item.getTranslation())
    },

    subheader(index) {
      if (this.productsAndEvents.length > 1) {
        let currentProductOrEvent = this.productsAndEvents[index]
        let previousProductOrEvent = this.productsAndEvents[index - 1]
        // check for events
        if (currentProductOrEvent instanceof Event) {
          // was previous item also an event ?
          if (previousProductOrEvent instanceof Event) return false
          return this.$t('singleReport.events')
        }

        // first subtitle
        if (index === 0)
          return this.productsAndEvents[index].getProductCategoryTitle()
        return previousProductOrEvent.productCategory ===
          currentProductOrEvent.productCategory
          ? false
          : currentProductOrEvent.getProductCategoryTitle()
      }
      return false
    },
  },
}
</script>

<style lang="scss">
.scrollable-list {
  max-height: 100vh;
  overflow-y: auto;
}
</style>
