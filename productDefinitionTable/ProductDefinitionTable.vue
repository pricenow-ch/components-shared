<template>
  <div>
    <div v-if="alignment == 'horizontal' && products">
      <product-definition-bookmarks
        v-if="
          availableProductDefinitions.length > 0 && bookmarkedProductDefinition
        "
        v-model="bookmarksModal"
        :product-definition="bookmarkedProductDefinition"
        :product-definitions="availableProductDefinitions"
        @changed="bookmarksChanged"
      />
      <!-- filter section -->
      <v-row class="filters py-3 flex-nowrap overflow-x" no-gutters>
        <pricing-bookmark-filter
          v-if="bookmarks.length > 0"
          ref="bookmarkFilter"
          :items="bookmarks"
          @change="filterByProdDefIds($event)"
          @cleared="filterByProdDefIds(null)"
          @removed-chip="removedBookmarkFilter($event)"
        >
        </pricing-bookmark-filter>
        <!-- filter for destinations -->
        <v-col v-if="!hideDestination" class="col-3">
          <pricing-filter
            id="destination"
            :items="destinations"
            :multiple="true"
            :translate="false"
            :clearable="true"
            :marginleft="true"
            :label="$t('attributes.region')"
            chipLabel="slug"
            selectLabel="slug"
            :value="destinations"
            :disabled="bookmarksFilterActive"
            @change="changeAttributeFilter('destinations', $event)"
            @cleared="filterCleared('destinations')"
            @removed-chip="removedChip('destinations', $event)"
          />
        </v-col>

        <!-- filters for attributes -->
        <v-col
          v-for="attribute in productsAttributes"
          :key="attribute.key"
          class="col-3"
        >
          <pricing-filter
            :id="attribute.key"
            :items="attribute.values"
            :multiple="true"
            :multipleFilters="true"
            :marginleft="true"
            :clearable="true"
            :label="$t('attributes.' + attribute.key)"
            chipLabel="translation"
            selectLabel="translation"
            :disabled="bookmarksFilterActive"
            @change="changeAttributeFilter(attribute.key, $event)"
            @cleared="filterCleared(attribute.key)"
            @removed-chip="removedChip(attribute.key, $event)"
          />
        </v-col>
      </v-row>

      <!-- table with filtered products -->
      <v-row no-gutters>
        <v-col cols="12">
          <v-card tile elevation="0">
            <!-- a height is needed here, so that the datatable body gets a scrollbar -->
            <v-data-table
              class="fixed-header theme-light"
              :headers="headers"
              :show-select="!limitProductDefinitionsSelections"
              :items="tableItems"
              :value="getDefinitionSelection"
              item-key="id"
              :fixed-header="true"
              :items-per-page="itemsPerPage"
              :footer-props="{
                itemsPerPageText: $t('general.rowsPerPage'),
                itemsPerPageAllText: $t('dataTable.allRows'),
              }"
              @item-selected="onSelectedItem"
              @toggle-select-all="onToggleAll"
            >
              <template #item="props">
                <tr>
                  <td>
                    <v-simple-checkbox
                      :color="getColor(props.item)"
                      :value="props.isSelected"
                      :disabled="
                        selectedDefinitions.length >= maxSelectableItems &&
                        !props.isSelected &&
                        limitProductDefinitionsSelections
                      "
                      @input="props.select"
                    />
                  </td>
                  <td class="clickable">
                    <i
                      v-if="!getBookmarkByProdDefId(props.item.getId())"
                      class="far fa-heart"
                      @click="openBookmarkModal(props.item)"
                    >
                    </i>
                    <i
                      v-else
                      class="fas fa-heart red--text"
                      @click="openBookmarkModal(props.item)"
                    >
                    </i>
                  </td>
                  <!-- Destinations -->
                  <td v-if="!hideDestination" class="py-0 opacity-50">
                    {{ props.item.destinations[0].slug | capitalize() }}
                  </td>
                  <td class="opacity-50">
                    {{ $t(props.item.translation) }}
                    <v-chip
                      v-if="props.item.isIndependent()"
                      class="ma-2"
                      x-small
                      outlined
                    >
                      {{ $t('v6.base') }}
                    </v-chip>
                  </td>
                  <td pa-0 style="min-width: 290px">
                    <v-container fluid grid-list-sm text-left pa-0 ma-0>
                      <v-row v-if="!pricesLoading" class="text-left">
                        <v-col
                          class="col-4 text-left mt-1 data-label-text"
                          style="min-width: 78px"
                        >
                          {{ props.item.getMinPrice() | currency }}
                        </v-col>
                        <v-col
                          class="col pa-0 justify-center d-flex align-content-center flex-wrap"
                        >
                          <span
                            v-if="getPriceByPath(props.item)"
                            class="font-weight-light"
                          >
                            {{
                              (getPriceByPath(props.item).customPrice ||
                                getPriceByPath(props.item).price) | currency
                            }}
                          </span>
                          <span
                            v-if="!getPriceByPath(props.item)"
                            class="font-weight-light opacity-50"
                          >
                            {{ $t('overviewPricing.undefined') }}
                          </span>
                          <v-progress-linear
                            v-if="getPriceByPath(props.item)"
                            class="pa-0 ma-0"
                            color="primary"
                            height="8"
                            rounded
                            :value="getPriceProgressValue(props.item)"
                          >
                          </v-progress-linear>
                          <v-progress-linear
                            v-if="!getPriceByPath(props.item)"
                            class="pa-0 ma-0"
                            color="primary"
                            height="8"
                            rounded
                            :value="0"
                          >
                          </v-progress-linear>
                        </v-col>
                        <v-col
                          style="min-width: 78px"
                          class="col-4 text-right mt-1 data-label-text"
                        >
                          {{ props.item.getMaxPrice() | currency }}
                        </v-col>
                      </v-row>
                      <v-skeleton-loader
                        v-if="pricesLoading"
                        type="text"
                        class="w-100"
                      />
                    </v-container>
                  </td>
                  <!-- Buchungsstände mit Progressbar -->
                  <td pa-0>
                    <v-container fluid grid-list-sm text-left pa-0 ma-0>
                      <v-row>
                        <!-- data is loading -->
                        <v-col v-if="realizedDemandLoading">
                          <v-skeleton-loader type="text" class="w-100" />
                        </v-col>

                        <!-- is basic product definition -->
                        <!-- v-else-if="props.item.isIndependent()" -->
                        <v-col v-else-if="false" class="col-12 text-center">
                          <v-row class="align-end">
                            <v-col class="col-9">
                              <span
                                v-if="bookingsNumber(props.item) >= 0"
                                class="font-weight-light opacity-50"
                              >
                                {{ bookingsNumber(props.item) }}
                              </span>

                              <v-progress-linear
                                v-if="bookingsNumber(props.item) >= 0"
                                class="pa-0 ma-0"
                                color="primary"
                                height="15"
                                rounded
                                :value="bookingsValue(props.item)"
                              />
                            </v-col>
                            <v-col class="col-3"> exp. n </v-col>
                          </v-row>
                        </v-col>

                        <!-- is related product definition -->
                        <v-col
                          v-else-if="
                            !realizedDemandLoading &&
                            !$store.getters.isOnlyStaticallyPriced()
                          "
                          class="col-12 text-center"
                        >
                          {{ bookingsNumber(props.item) }}
                          <span class="fal fa-ticket ml-2" />
                        </v-col>
                      </v-row>
                    </v-container>
                  </td>
                </tr>
              </template>
            </v-data-table>
          </v-card>
        </v-col>
      </v-row>
    </div>

    <div v-else-if="alignment == 'vertical' && products">
      <v-row class="ma-0 fill-height align-center white">
        <!-- show / hide filter -->
        <v-col class="col-12 filterBtnContainer">
          <v-btn
            outlined
            color="primary"
            large
            @click="showFilter = !showFilter"
          >
            <div v-if="!showFilter">
              <i class="fas fa-filter mr-4" />
              {{ $t('v6.showFilter') }}
            </div>
            <div v-else>
              <span>
                <i class="fa fa-filter" />
                <i class="fa fa-slash error--text ml-n5" />
              </span>
              {{ $t('v6.hideFilter') }}
            </div>
          </v-btn>
        </v-col>

        <!-- filter secion -->
        <v-expand-transition>
          <v-col v-show="showFilter" class="col-12 white">
            <pricing-bookmark-filter
              v-if="bookmarks.length > 0"
              ref="bookmarkFilter"
              class="mb-2 pa-0"
              :items="bookmarks"
              @change="filterByProdDefIds($event)"
              @cleared="filterCleared()"
              @removed-chip="removedBookmarkFilter($event)"
            >
            </pricing-bookmark-filter>
            <!-- destinations -->
            <pricing-filter
              id="destination"
              class="mb-2"
              :items="destinations"
              :multiple="true"
              :marginleft="false"
              :translate="false"
              :clearable="true"
              :label="$t('attributes.region')"
              chipLabel="slug"
              selectLabel="slug"
              :value="destinations"
              :disabled="bookmarksFilterActive"
              @change="changeAttributeFilter('destinations', $event)"
              @cleared="filterCleared('destinations')"
              @removed-chip="removedChip('destinations', $event)"
            />

            <!-- attributes -->
            <div v-for="attribute in productsAttributes" :key="attribute.key">
              <pricing-filter
                :id="attribute.key"
                class="mb-2"
                :items="attribute.values"
                :multiple="true"
                :multipleFilters="true"
                :marginleft="false"
                :clearable="true"
                :label="$t('attributes.' + attribute.key)"
                chipLabel="translation"
                selectLabel="translation"
                :disabled="bookmarksFilterActive"
                @change="changeAttributeFilter(attribute.key, $event)"
                @cleared="filterCleared(attribute.key)"
                @removed-chip="removedChip(attribute.key, $event)"
              />
            </div>
          </v-col>
        </v-expand-transition>

        <!-- products count indicator -->
        <v-col v-if="showTotalSelections" class="mr-n5">
          <v-alert v-if="getDefinitionSelection.length > 1" type="info">
            {{
              $t('overviewPricing.countSelectedDefinitions', {
                counter: getDefinitionSelection.length,
              })
            }}
          </v-alert>
        </v-col>

        <!-- products -->
        <v-col class="col-12 white">
          <v-data-table
            :headers="headers"
            :single-select="singleSelect"
            :show-select="!singleSelect && !limitProductDefinitionsSelections"
            :items="tableItems"
            :value="getDefinitionSelection"
            item-key="id"
            :items-per-page="itemsPerPage"
            :page.sync="page"
            hide-default-footer
            @item-selected="onSelectedItem"
            @toggle-select-all="onToggleAll"
          >
            <!-- content -->
            <template #item="props">
              <tr>
                <td>
                  <v-simple-checkbox
                    :color="getColor(props.item)"
                    :value="props.isSelected"
                    :disabled="
                      selectedDefinitions.length >= maxSelectableItems &&
                      !props.isSelected &&
                      limitProductDefinitionsSelections
                    "
                    @input="props.select"
                  />
                </td>
                <td
                  pa-0
                  :class="{
                    'font-weight-medium': props.item.isIndependent(),
                    'font-weight-thin text--secondary': !props.item.isIndependent(),
                  }"
                >
                  <span>
                    {{ $t(props.item.translation) }}
                  </span>
                </td>
                <td v-if="showBasePrice && getPriceByPath(props.item)" pa-0>
                  {{ getPriceByPath(props.item).getRealPrice() | currency }}
                </td>
              </tr>
            </template>

            <!-- footer -->
            <template #footer="{ props }">
              <table-footer
                :itemsPerPage.sync="itemsPerPage"
                :page="page"
                :pageCount="props.pagination.pageCount"
                @nextPage="
                  () => {
                    page++
                  }
                "
                @previousPage="
                  () => {
                    page--
                  }
                "
              >
              </table-footer>
            </template>
          </v-data-table>
        </v-col>
      </v-row>
    </div>
  </div>
</template>

<script>
import PricingFilter from '@/components-shared/productDefinitionTable/PricingFilter.vue'
import { mapGetters, mapState } from 'vuex'
import moment from 'moment'
import ProductDefinitionPrices from '@/classes-shared/products/ProductDefinitonPrices'
import ProductDefinitionCapacities from '@/classes-shared/expectedDemand/ProductDefinitonCapacities'
import TableFooter from '@/components-shared/utils/vuetifyHelper/TableFooter'
import definitions from '../../../definitions'
import store from '@/store/store'
import ProductDefinitionBookmarks from '@/components-shared/productDefinitionTable/ProductDefinitionBookmarks'
import PricingBookmarkFilter from '@/components-shared/productDefinitionTable/PricingBookmarkFilter'
import _ from 'lodash'

export default {
  name: 'ProductDefinitionTable',
  components: {
    TableFooter,
    PricingFilter,
    ProductDefinitionBookmarks,
    PricingBookmarkFilter,
  },
  props: {
    // the path to the store variable
    // that selections are committed to
    // e.g. 'OverviewPricing/setSelectedDefinitions'
    storeGetSelectedDefinitionsPath: {
      type: String,
      required: true,
    },
    storeSetSelectedDefinitionsPath: {
      type: String,
      required: true,
    },
    storeAddDefinitionPath: {
      type: String,
      required: true,
    },
    storeRemoveDefinitionPath: {
      type: String,
      required: true,
    },
    showTotalSelections: {
      type: Boolean,
      required: false,
      default: false,
    },
    singleSelect: {
      type: Boolean,
      required: false,
      default: true,
    },
    disableFilterTransition: {
      type: Boolean,
      required: false,
      default: false,
    },
    limitProductDefinitionsSelections: {
      type: Boolean,
      required: false,
      default: false,
    },
    disableSelectionColors: {
      type: Boolean,
      required: false,
      default: false,
    },
    products: {
      type: Object,
      default: () => {
        return null
      },
      required: true,
    },
    preselection: {
      type: Array,
      default: () => {
        return []
      },
      required: false,
    },
    alignment: {
      type: String,
      default: 'horizontal',
      required: true,
      validator: (value) => {
        return ['horizontal', 'vertical'].indexOf(value) !== -1
      },
    },
    selectedDate: {
      type: Date,
      required: false,
      validator: (value) => {
        return value instanceof Date
      },
      default: null,
    },
    maxSelectableItems: {
      type: Number,
      default: definitions.colors.lineChart.length,
    },
    isSingleSelect: {
      type: Boolean,
      default: false,
    },
    showBasePrice: {
      type: Boolean,
      default: false,
    },
    hideDependingProductDefinitions: {
      type: Boolean,
      default: false,
    },
    hideDestination: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
  data() {
    return {
      page: 1,
      itemsPerPage: 15,
      productsAttributes: [],
      filteredProductDefinitions: [],
      availableProductDefinitions: [],
      attributeFilters: {},
      colors: definitions.colors.lineChart.slice(),
      showFilter: false,
      bookmarkedProductDefinition: null,
      bookmarksModal: false,
      bookmarksFilterActive: false,
      realizedDemandHandler: new ProductDefinitionCapacities(),
      pricesHandler: new ProductDefinitionPrices(),
    }
  },
  computed: {
    ...mapState(['actualLanguage', 'languages']),
    ...mapGetters('UserProductDefinitionBookmarks', [
      'bookmarks',
      'getBookmarkByProdDefId',
    ]),

    selectedDefinitions() {
      return this.$store.getters[this.storeGetSelectedDefinitionsPath]
    },

    // get all available destinations for the selected products
    destinations() {
      if (this.products) {
        return this.products.getDestinations()
      }

      // return default destination
      return [store.getters.getCurrentDestinationInstance()]
    },

    // product definitions displayed in the table
    tableItems() {
      let prodDefs = []
      if (this.filteredProductDefinitions.length > 0) {
        prodDefs = this.filteredProductDefinitions
      } else if (
        this.filteredProductDefinitions.length === 0 &&
        Object.keys(this.attributeFilters).length > 0
      ) {
        return []
      } else {
        prodDefs = this.availableProductDefinitions
      }
      if (this.hideDependingProductDefinitions) {
        prodDefs = prodDefs.filter((prodDef) => {
          return prodDef.isIndependent()
        })
      }

      return _.sortBy(prodDefs, ['productId', 'durationDays', 'id'])
    },
    getDefinitionSelection() {
      return this.selectedDefinitions.map(({ item }) => item)
    },
    headers() {
      const headers = [
        {
          text: '',
          align: 'left',
          sortable: false,
          value: '',
        },
        {
          text: '',
          align: 'left',
          sortable: false,
          value: '',
        },
        {
          text: this.$t('overviewPricing.regions'),
          align: 'left',
          sortable: true,
          value: '',
        },
        {
          text: this.$t('overviewPricing.product'),
          align: 'left',
          sortable: true,
          value: 'name',
        },
        {
          text: this.selectedDate
            ? this.$t('overviewPricing.price') +
              ' (' +
              moment(this.selectedDate).format('DD. MM. YY') +
              ')'
            : '',
          align: 'center',
          sortable: true,
          value: 'price',
        },
        {
          text: this.selectedDate
            ? this.$t('overviewPricing.bookings') +
              ' (' +
              moment(this.selectedDate).format('DD. MM. YY') +
              ')'
            : '',
          align: 'center',
          sortable: false,
          value: '',
        },
      ]

      if (this.alignment === 'vertical') {
        let vertHeaders = []
        if (this.singleSelect) {
          vertHeaders.push(headers[0])
        }
        vertHeaders.push(headers[3])
        if (this.showBasePrice) {
          vertHeaders.push(headers[4])
        }
        return vertHeaders
      }

      if (this.$store.getters.isOnlyStaticallyPriced()) {
        headers.splice(5, 1)
      }

      if (this.hideDestination) {
        headers.splice(2, 1)
      }

      if (!this.limitProductDefinitionsSelections) {
        return headers.splice(1)
      }

      return headers
    },
    pricesLoading() {
      return this.pricesHandler.isLoading()
    },
    realizedDemandLoading() {
      return this.realizedDemandHandler.isLoading()
    },
  },
  watch: {
    selectedDate: {
      deep: true,
      handler() {
        this.fetchAdditionalData().then(() => {
          this.updateSelectedDefinitions()
        })
      },
    },
    products: {
      handler() {
        this.resetComponentState()
        this.setAvailableProductDefinitions()
        this.fetchAdditionalData().then(() => {
          this.updateSelectedDefinitions()
        })
        this.getAttributeNames()

        // set first product definition in table
        this.selectFirstProductDefinition()
      },
    },
    limitProductDefinitionsSelections() {
      if (
        this.limitProductDefinitionsSelections &&
        this.selectedDefinitions.length > this.maxSelectableItems
      ) {
        // unselect all
        this.onToggleAll({ value: false })
        // reset items per page to 15
        this.itemsPerPage = 15
      }
    },
  },
  async mounted() {
    await this.setAvailableProductDefinitions()
    this.fetchAdditionalData()
    this.getAttributeNames()
    if (this.preselection.length > 0) {
      this.$store.commit(
        this.storeSetSelectedDefinitionsPath,
        this.preselection
      )
      const len = this.colors.length - this.preselection.length
      this.colors.length = len < 0 ? 0 : len
    }
    // select default prod definition from the table
    if (
      this.availableProductDefinitions.length &&
      this.preselection.length === 0
    ) {
      this.selectFirstProductDefinition()
    }
    this.updateSelectedDefinitions()
  },
  methods: {
    // select the first product definition from the table e.g. on mounted
    selectFirstProductDefinition() {
      if (!this.tableItems || !this.tableItems.length) return
      // new strange object created
      this.colors = [...definitions.colors.lineChart]
      this.onSelectedItem({ item: this.tableItems[0], value: true })
    },
    resetBookmarkFilter() {
      if (this.$refs.bookmarkFilter) {
        this.$refs.bookmarkFilter.reset()
      }
    },
    // a particular filter was reset fully
    async filterCleared(label) {
      this.$store.commit(this.storeSetSelectedDefinitionsPath, [])
      this.resetBookmarkFilter()
      delete this.attributeFilters[label]
      await this.filterProducts()
    },

    // a chip was removed from a certain filter
    async removedChip(label, attributeToRemove) {
      this.resetBookmarkFilter()
      // get index to remove from filter
      let attributeValue = null

      if (label === 'destinations') attributeValue = attributeToRemove.id
      // destination filter mode
      else attributeValue = attributeToRemove[definitions.attributeValues.value]

      // index to remove from label filter
      let indexOfValue = this.attributeFilters[label].indexOf(attributeValue)

      // only remove filter attribute if we got an index
      if (indexOfValue !== -1) {
        if (this.attributeFilters[label].length <= 1)
          delete this.attributeFilters[label]
        else delete this.attributeFilters[label][indexOfValue]
        await this.filterProducts()
      }
    },

    // if a filter emits @change (adding or removing from the dropdown
    async changeAttributeFilter(label, selectedFilterItems) {
      this.resetBookmarkFilter()
      if (Array.isArray(selectedFilterItems) && selectedFilterItems.length) {
        // we still have attributes for this label
        let attributes = []
        for (let filterItem of selectedFilterItems) {
          // is the label === destination
          if (label === 'destinations') attributes.push(filterItem.getId())
          else attributes.push(filterItem) // simple attribute value
        }
        // save new attribute filter for further filtering
        this.$set(this.attributeFilters, label, attributes)
      } else delete this.attributeFilters[label] // no attribute for this label. reset label fully

      await this.filterProducts()
    },

    // after the filter was manipulated filter the products
    async filterProducts() {
      let filtered = await this.products.filterProductsByAttributesAndDestinations(
        this.attributeFilters
      )

      this.$set(this, 'filteredProductDefinitions', filtered)
      this.updateSelectedDefinitions()
      return Promise
    },

    async filterByProdDefIds(ids) {
      if (ids?.length > 0) {
        this.bookmarksFilterActive = true
        let filtered = await this.products.filterByProductDefIds(ids)

        this.$set(this, 'filteredProductDefinitions', filtered)
      } else {
        this.bookmarksFilterActive = false
        await this.filterProducts()
      }
      this.updateSelectedDefinitions()
      return Promise
    },

    removedBookmarkFilter(removedIds) {
      this.selectedDefinitions = []
      this.bookmarksFilterActive = false
      if (this.filteredProductDefinitions.length > 0) {
        let filtered = this.filteredProductDefinitions.filter((prodDef) => {
          return !removedIds.includes(prodDef.getId())
        })

        this.$set(this, 'filteredProductDefinitions', filtered)
        this.updateSelectedDefinitions()
      }
    },

    // get the color of an item
    getColor(item) {
      const selectedDefinition = this.selectedDefinitions.find(
        (selectedDefinition) => {
          return selectedDefinition.item.id === item.id
        }
      ) // find the object where it has a property item equal to our item
      if (!selectedDefinition)
        // if item isn't selected
        return this.$vuetify.theme.themes.light.grey
      return selectedDefinition.color
    },

    // product definition was selected from the table
    async onSelectedItem({ item, value: selected }) {
      if (selected) {
        // select
        if (this.isSingleSelect) {
          // oh no, we're out of colours D:
          // reset the colors array to have all colors again
          this.colors = definitions.colors.lineChart.slice()
        }

        if (this.isSingleSelect) {
          this.$store.commit(this.storeSetSelectedDefinitionsPath, [
            {
              item: item,
              color: this.disableSelectionColors ? null : this.colors.pop(),
            },
          ])
        }

        // check if product definition already is selected
        if (
          !this.selectedDefinitions.find(
            (tmpDefinition) => tmpDefinition.item.id === item.id
          )
        ) {
          this.$store.commit(this.storeAddDefinitionPath, {
            color: this.disableSelectionColors ? null : this.colors.pop(),
            item,
          })
        }
      } else {
        // deselect product definition
        const index = this.selectedDefinitions.findIndex(
          (selectedDefinition) => selectedDefinition.item.id === item.id
        )
        if (index === -1) {
          return
        }

        // return the color we deselected back to the array
        this.colors.push(this.selectedDefinitions[index].color)

        // remove our selected item from our array
        this.$store.commit(this.storeRemoveDefinitionPath, index)

        this.rematchColors()
      }
    },

    rematchColors() {
      if (
        this.colors.length > 0 &&
        this.selectedDefinitions.length > this.maxSelectableItems
      ) {
        _.sortBy(this.selectedDefinitions, [
          'item.productId',
          'item.durationDays',
          'item.id',
        ]).forEach((selectedDefinition) => {
          if (
            selectedDefinition.color === undefined &&
            this.colors.length > 0
          ) {
            const index = this.selectedDefinitions.findIndex(
              (definition) => definition.item.id === selectedDefinition.item.id
            )

            if (-1 !== index) {
              this.$store.commit(this.storeRemoveDefinitionPath, index)

              this.$store.commit(this.storeAddDefinitionPath, {
                color: this.disableSelectionColors ? null : this.colors.pop(),
                item: selectedDefinition.item,
              })
            }
          }
        })
      }
    },

    async loadPrices() {
      if (!this.selectedDate) return
      const prodIds = this.products.getProductIds()
      await this.pricesHandler.getAllProductDefinitionPricesOnDate(
        this.selectedDate,
        prodIds
      )
    },

    async loadRealizedDemand() {
      if (!this.selectedDate) return
      await this.realizedDemandHandler.getAllRealizedDemandForMultipleDestinations(
        this.selectedDate,
        this.selectedDate,
        this.destinations
      )
    },

    setAvailableProductDefinitions() {
      if (this.products) {
        this.$set(
          this,
          'availableProductDefinitions',
          this.products.getAllProductDefinitions()
        )
      }
    },

    // fetching possible attribute names depending on the available products
    async getAttributeNames() {
      if (this.products) {
        this.$set(
          this,
          'productsAttributes',
          await this.products.getAllAttributeNames()
        )
      }
    },

    // reset selected definitions if needed (correct description?)
    updateSelectedDefinitions() {
      let tempSelectedDefinitions = []
      let filteredItems = this.availableProductDefinitions

      // the filtered items equals the already filtered product definitions and not the available product definitions
      if (this.filteredProductDefinitions.length)
        filteredItems = this.filteredProductDefinitions

      // iterate selected product definitions
      for (let i = 0; i < this.selectedDefinitions.length; i++) {
        let selectedDefinition = this.selectedDefinitions[i].item
        let updatedSelectedDefinitions = filteredItems.filter((obj) => {
          return obj.id === selectedDefinition.id
        })

        // we still have at least one selected prod definition
        if (updatedSelectedDefinitions.length) {
          tempSelectedDefinitions.push(this.selectedDefinitions[i])
        } else this.selectFirstProductDefinition() // automatically set just the first available prod def
      }

      // reset colors (if colors were selected and are gone now by filtering => free these colors)
      this.colors = definitions.colors.lineChart.slice()

      for (let b = 0; b < tempSelectedDefinitions.length; b++) {
        this.colors = this.colors.filter(
          (color) => tempSelectedDefinitions[b].color !== color
        )
      }

      this.$store.commit(
        this.storeSetSelectedDefinitionsPath,
        tempSelectedDefinitions
      )
    },

    // getting additional data from api
    async fetchAdditionalData() {
      if (this.showBasePrice) {
        return this.loadPrices()
      } else {
        return Promise.all([this.loadPrices(), this.loadRealizedDemand()])
      }
    },

    resetComponentState() {
      this.productsAttributes = []
      this.filteredProductDefinitions = []
      this.availableProductDefinitions = []
      this.$store.commit(this.storeSetSelectedDefinitionsPath, [])
      this.attributeFilters = {}
      this.colors = []
    },

    onToggleAll(event) {
      if (event.value) {
        this.$store.commit(this.storeSetSelectedDefinitionsPath, [])
        this.colors = definitions.colors.lineChart.slice()
        if (this.filteredProductDefinitions.length > 0) {
          _.sortBy(this.filteredProductDefinitions, [
            'productId',
            'durationDays',
            'id',
          ]).forEach((prodDef) => {
            this.$store.commit(this.storeAddDefinitionPath, {
              item: prodDef,
              color: this.disableSelectionColors ? null : this.colors.pop(),
            })
          })
        } else {
          _.sortBy(this.availableProductDefinitions, [
            'productId',
            'durationDays',
            'id',
          ]).forEach((prodDef) => {
            this.$store.commit(this.storeAddDefinitionPath, {
              item: prodDef,
              color: this.disableSelectionColors ? null : this.colors.pop(),
            })
          })
        }
        // sets items per page to 'all'
        this.itemsPerPage = -1
      } else {
        this.$store.commit(this.storeSetSelectedDefinitionsPath, [])
        this.colors = definitions.colors.lineChart.slice()
      }
    },
    getPriceByPath(item) {
      return this.pricesHandler.getPriceByPath(
        item?.id,
        moment(this.selectedDate).format('YYYY-MM-DD')
      )
    },
    getRealizedDemandByPath(item) {
      return this.realizedDemandHandler.getRealizedDemandByPath(
        item?.id,
        moment(this.selectedDate).format('YYYY-MM-DD')
      )
    },
    bookingsValue(item) {
      return (
        (100 / parseInt(this.getRealizedDemandByPath(item)?.demandPrediction)) *
        parseInt(this.getRealizedDemandByPath(item)?.demand)
      )
    },
    bookingsNumber(item) {
      if (this.getRealizedDemandByPath(item)?.demand) {
        return this.getRealizedDemandByPath(item).demand
      }
      return 0
    },
    getPriceProgressValue(item) {
      const price = this.getPriceByPath(item)
      return (
        (100 / (item.getMaxPrice() - item.getMinPrice())) *
        (price.getRealPrice() - item.getMinPrice())
      )
    },
    openBookmarkModal(productDef) {
      this.bookmarkedProductDefinition = productDef
      this.bookmarksModal = true
    },
    bookmarksChanged(bookmarks) {
      if (bookmarks?.length === 0) {
        this.bookmarksModal = false
        this.bookmarksFilterActive = false
        this.filterByProdDefIds(null)
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.v-simple-checkbox--disabled {
  cursor: not-allowed;
}
.v-btn-toggle.vertical {
  flex-direction: column;
}

.v-btn-toggle > .v-btn.v-btn:first-child {
  border-radius: 0;
}
.v-btn-toggle > .v-btn.v-btn:last-child {
  border-radius: 0;
}
.v-select__slot {
  height: 57px !important;
}

.outlined {
  border: 1px solid lightcoral;
}
.button-list {
  float: right;
}
.v-btn--icon {
  border-radius: 30% !important;
}
.v-btn--icon::before {
  border-radius: 30% !important;
}
.bordered {
  border: 1px solid map-get($blue, 'base');
}
.product-filter {
  position: absolute;
  top: -60px;
  z-index: 999;
}
.product-filter > .v-btn {
  background-color: none;
}

.opacity-50 {
  opacity: 0.5;
}

.filterExpandBtnContainer {
  position: absolute;
  top: 10px;
  right: -15px;
}

.filterBtnContainer {
  display: flex;
  justify-content: center;
  align-items: center;
}

.overflow-x {
  overflow-x: auto;
}
</style>
