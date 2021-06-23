<!-- main header in the app -->
<template>
  <div>
    <!-- toolbar: menu, logo and user menu -->
    <v-row
      class="skinow-toolbar fluid elevation-4 align-center justify-center no-gutters fixed-toolbar-for-backend"
    >
      <!-- logo -->
      <v-col class="clickable just col-4 pl-2">
        <img
          class="destination-logo"
          :src="require('../../assets/destinations/logo/default.png')"
        />
      </v-col>

      <!-- product selection in backend mode -->
      <v-col
        v-if="$store.getters.getAppUserInstance()"
        class="col-4 text-center"
      >
        <product-selector-header-wrapper />
      </v-col>

      <!-- basket, user and language -->
      <v-col class="pd-4 pr-sm-6 text-right">
        <!-- user is logged out -->
        <v-btn
          v-if="!$store.getters.isUserLoggedIn()"
          class="primary v-menu-header-height v-btn-header-position v-btn-min-width mr-md-1 mr-0 v-btn-border-bottom-left"
          text
          @click="$router.push({ name: 'login' })"
        >
          <!-- user icon -->
          <div class="headline-large">
            <span class="fa fa-user"></span>
          </div>
        </v-btn>

        <!-- user is logged in: show profile menu dropdown -->
        <v-menu v-if="$store.getters.isUserLoggedIn()" offset-y>
          <template #activator="{ on }">
            <v-btn
              text
              class="primary v-menu-header-height v-btn-header-position v-btn-min-width mr-md-1 mr-0 v-btn-border-bottom-left"
              v-on="on"
            >
              <!-- user icon -->
              <div class="headline-large">
                <span class="fa fa-user"></span>
              </div>
            </v-btn>
          </template>

          <!-- user menu -->
          <v-list>
            <v-list-item @click="$router.push({ name: 'profile' })">
              <v-list-item-title>
                <span class="content-3">
                  {{ $t('routingTitles.profile') }}
                </span>
              </v-list-item-title>
            </v-list-item>

            <!-- logging out button -->
            <v-list-item>
              <v-list-item-title @click="$store.dispatch('logout')">
                <span class="content-3 clickable">
                  {{ $t('layout.logout') }}
                </span>
              </v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>

        <!-- only desktop -->

        <!-- language -->
        <v-btn
          v-if="$vuetify.breakpoint.smAndUp"
          class="primary v-menu-header-height v-btn-header-position v-btn-border-bottom-right v-btn-min-width"
          text
          @click="showLanguageModal()"
        >
          {{ $store.getters.getActualLanguage() }}
        </v-btn>
      </v-col>
    </v-row>

    <!-- language selector -->
    <v-dialog v-model="changeLanguageModal" max-width="290">
      <v-card>
        <v-card-title class="content-3">{{
          $t('layout.selectLanguageModalHeader')
        }}</v-card-title>

        <v-card-text>
          <v-select
            v-model="selectedLanguageKey"
            :items="$store.getters.getLanguages()"
            :label="$t('layout.selectLanguage')"
            outlined
            hide-details
          />
        </v-card-text>

        <v-card-actions class="justify-center pt-0">
          <v-btn large outlined @click="changeLanguage()">
            {{ $t('general.ok') }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import ProductSelectorHeaderWrapper from '../products/ProductSelectorHeaderWrapper'
import { shopInstance } from '../../classes-shared/utils/axiosInstance'

export default {
  name: 'AppHeaderAdmin',

  components: {
    ProductSelectorHeaderWrapper,
  },

  data() {
    return {
      changeLanguageModal: false,
      drawer: false,
      selectedLanguageKey: '',
      selectedMenuEntry: Object,
    }
  },

  computed: {
    // shortcut
    isDefaultDestination() {
      return this.$store.getters
        .getAppDestinationInstance()
        .isDefaultDestination()
    },

    countBasketEntries() {
      return this.$store.getters.getBasketInstance().countBasketEntries()
    },

    // pages in the menu
    pages() {
      let pagesInstance = this.$store.getters.getPagesInstance()

      if (pagesInstance) return pagesInstance.getPages()
      else return []
    },
  },

  asyncComputed: {
    async goToBackendPe() {
      if (this.$store.getters.getAppUserInstance()) {
        return await this.$store.getters
          .getRouterHelperInstance()
          .getFirstBackendPeRouteWhereUserHasPermissionFor()
      }
    },

    async goToBackendShop() {
      if (this.$store.getters.getAppUserInstance()) {
        return await this.$store.getters
          .getRouterHelperInstance()
          .getFirstBackendShopRouteWhereuserHasPermissionFor()
      }
    },
  },

  mounted() {
    this.selectedLanguageKey = this.$store.getters.getActualLanguage(false) // get actual language Key
  },

  methods: {
    // show burger menu
    show() {
      this.drawer = true
    },

    hide() {
      this.drawer = false
    },

    showLanguageModal() {
      // hide burger menu first
      this.drawer = false

      // show modal
      this.changeLanguageModal = true
    },

    // burger menu entry was selected
    selectBurgerMenuEntry(route) {
      this.drawer = false
      this.$router.push({ name: route })
    },

    openBackend(route) {
      this.drawer = false
      this.$router.push({ name: route.name })
    },

    goToPage(page) {
      this.drawer = false

      if (page.getWeblink()) window.location.href = page.getWeblink()
      else this.$router.push({ name: 'apiPage', params: { id: page.getId() } })
    },

    changeLanguage() {
      // hide language modal
      this.changeLanguageModal = false

      // set new language in store
      this.$store.commit('setLanguage', this.selectedLanguageKey)

      if (this.$store.getters.isUserLoggedIn()) {
        /* global EventBus axios */
        EventBus.$emit('spinnerShow')

        shopInstance()
          .patch('user/language', {
            language: this.selectedLanguageKey,
          })
          .then(() => {
            EventBus.$emit(
              'notify',
              this.$root.$t('notify.languageChangedSuccessfully'),
              'success'
            )
          })
          .catch(() => {
            EventBus.$emit(
              'notify',
              this.$root.$t('notify.languageCouldNotBeSaved'),
              'error'
            )
          })
          .finally(() => {
            EventBus.$emit('spinnerHide')
          })
      }
    },
  },
}
</script>

<style lang="scss">
.destination-logo {
  height: $icon-medium;
}

.shopping-cart-button {
  margin-top: -19px;
  margin-left: -2px;
  margin-right: -10px;
}

.v-btn-header-position {
  margin-top: -18px !important;
}

@media screen and (max-width: $sm) {
  .v-btn-header-position {
    margin-top: -8px !important;
  }

  .destination-logo {
    height: $icon-medium-mobile;
  }
}

.v-menu-header-height {
  height: 45px !important;
  align-self: self-start !important;
}

.skinow-toolbar {
  height: $header-height-sm;
  background-color: white;
  width: 100vw;
  position: absolute;
  top: 0px;
  left: 0px;
}

@media screen and (max-width: $sm) {
  .skinow-toolbar {
    height: $header-height-xs;
  }
}

/** do not scroll the header in backend */
.fixed-toolbar-for-backend {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
}
</style>
