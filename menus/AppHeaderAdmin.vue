<!-- main header in the app -->
<template>
  <div>
    <!-- toolbar: menu, logo and user menu -->
    <v-row
      class="skinow-toolbar fluid elevation-4 align-center justify-center no-gutters"
      :class="{ 'fixed-toolbar-for-backend': backendMode }"
    >
      <!-- burger menu and menu btn -->
      <v-col
        v-if="!backendMode"
        class="col-2 col-sm-3 col-md-4 clickable pl-4 pl-sm-6"
        @click="drawer = !drawer"
      >
        <!-- burger lines -->
        <span class="headline">
          <span class="fa fa-bars" />
        </span>
        <span v-if="$vuetify.breakpoint.smAndUp" class="headline pl-2">
          {{ $t('layout.menu') }}
        </span>
      </v-col>

      <!-- logo -->
      <v-col
        class="clickable just col-4 pl-2"
      >
        <img
          class="destination-logo"
          :src="require('../../assets/destinations/logo/default.png')"
        />
      </v-col>

      <!-- product selection in backend mode -->
      <v-col
        v-if="backendMode && $store.getters.getAppUserInstance()"
        class="col-4 text-center"
      >
        <product-selector-header-wrapper />
      </v-col>

      <!-- basket, user and language -->
      <v-col class="pd-4 pr-sm-6 text-right">
        <!-- basket -->
        <v-btn
          v-if="!backendMode"
          text
          class="primary v-menu-header-height v-btn-header-position v-btn-border-bottom-left v-btn-min-width mr-1"
          @click="$router.push({ name: 'basketView' })"
        >
          <!-- basket icon -->
          <div class="headline-large">
            <span class="fa fa-shopping-cart"></span>
          </div>

          <!-- number of items in the basket -->
          <span
            v-if="countBasketEntries"
            class="headline-standard font-weight-bold shopping-cart-button"
          >
            {{ countBasketEntries }}
          </span>
        </v-btn>

        <!-- user is logged out -->
        <v-btn
          v-if="!$store.getters.isUserLoggedIn()"
          class="primary v-menu-header-height v-btn-header-position v-btn-min-width mr-md-1 mr-0"
          :class="[
            backendMode ? 'v-btn-border-bottom-left' : 'v-btn-border-none',
          ]"
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
              class="primary v-menu-header-height v-btn-header-position v-btn-min-width mr-md-1 mr-0"
              :class="[
                backendMode ? 'v-btn-border-bottom-left' : 'v-btn-border-none',
              ]"
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
            <v-list-item
              v-for="(route, index) in userProfileRoutes"
              :key="index"
              @click="$router.push(route)"
            >
              <v-list-item-title>
                <span class="content-3">
                  {{ $t('routingTitles.' + route.name) }}
                </span>
              </v-list-item-title>
            </v-list-item>

            <!-- logging out button -->
            <v-list-item>
              <v-list-item-title @click="initLogout()">
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

    <!-- burger menu content -->
    <v-navigation-drawer v-model="drawer" app disable-resize-watcher>
      <!-- show on desktop and mobile -->

      <!-- close menu -->
      <v-row class="justify-center">
        <v-col class="col-12 text-center">
          <v-btn text class="white content-3" @click="drawer = !drawer">
            <v-icon class="mr-2">fa-times</v-icon>
          </v-btn>
        </v-col>
      </v-row>

      <v-divider></v-divider>

      <!-- switch to shop backend -->
      <v-row v-if="goToBackendShop" class="justify-center">
        <v-col class="col-12 text-center">
          <v-btn
            text
            class="white content-3"
            @click="openBackend(goToBackendShop)"
          >
            {{ $t('appHeader.goToBackendShop') }}
          </v-btn>
        </v-col>
      </v-row>

      <v-divider v-if="goToBackendShop"></v-divider>

      <v-row v-if="goToBackendPe" class="justify-center">
        <v-col class="xs12 text-center">
          <v-btn
            text
            class="white content-3"
            @click="openBackend(goToBackendPe)"
          >
            {{ $t('appHeader.goToBackendPe') }}
          </v-btn>
        </v-col>
      </v-row>

      <v-divider v-if="goToBackendPe"></v-divider>

      <!-- for all users -->

      <!-- booking -->
      <v-row class="justify-center">
        <v-col class="text-center">
          <v-btn
            text
            class="white content-3"
            @click="selectBurgerMenuEntry('booking')"
          >
            {{ $t('layout.booking') }}
          </v-btn>
        </v-col>
      </v-row>

      <v-divider />

      <!-- iterate pages -->
      <div v-for="(page, pageIndex) in pages" :key="pageIndex">
        <v-row class="justify-center">
          <v-col class="col-12 text-center">
            <v-btn text class="white content-3" @click="goToPage(page)">
              {{ page.getTitle() }}
            </v-btn>
          </v-col>
        </v-row>

        <v-divider />
      </div>

      <!-- only mobile -->

      <!-- change language -->
      <v-row class="justify-center hidden-sm-and-up">
        <v-col class="col-12 text-center">
          <v-btn text class="white content-3" @click="showLanguageModal()">
            {{ $store.getters.getActualLanguage() }}
          </v-btn>
        </v-col>
      </v-row>
    </v-navigation-drawer>

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

    <!-- ask user really to stop the current booking process, because the basket will be deleted -->
    <s-modal
      ref="resetBasketModal"
      :size="1"
      :header-text="$t('v5.currentBasketWillBeDeleted')"
      @ok="logout(true)"
    >
      <v-row>
        <v-col class="col-12 content-3">
          <span class="fa fa-exclamation-triangle mr-4 primary--text" />
          {{ $t('v5.areYouSureDeletingBasket') }}
        </v-col>
      </v-row>
    </s-modal>
  </div>
</template>

<script>
import SModal from '../utils/SModal'
import ProductSelectorHeaderWrapper from '../products/ProductSelectorHeaderWrapper'

export default {
  name: 'AppHeader',

  components: {
    ProductSelectorHeaderWrapper,
    SModal,
  },

  props: {
    // show different header, if user is in backend
    backendMode: {
      type: Boolean,
      default: false,
      required: false,
    },
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
    userProfileRoutes: {
      async get() {
        // only evaluate if app user instance is already set
        if (this.$store.getters.getAppUserInstance()) {
          return await this.$store.getters
            .getRouterHelperInstance()
            .getAllUserProfileRoutes(true)
        } else return []
      },
      watch: ['$route'],
    },

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

    // logout from other components
    EventBus.$on('logout', (notify = true) => {
      this.logout(true, notify)
    })
  },

  beforeDestroy() {
    EventBus.$off('logout')
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

        axios
          .patch(
            this.$store.getters.getCurrentDestinationInstance().getShopApi() +
              'user/language',
            {
              language: this.selectedLanguageKey,
            }
          )
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

    /**
     * LOGOUT SECTION
     **/
    initLogout() {
      // did or do we use the basket...
      if (this.$store.getters.getBasketInstance().isBasketInUse()) {
        // ... then ask the user to delete it
        this.$refs.resetBasketModal.show()
      } else {
        this.logout()
      }
    },

    /**
     * this is the main logout logic
     */
    async logout(resetBookingProcess = true, notify = true) {
      EventBus.$emit('spinnerShow')

      // delete user and user authorization
      await this.$store.dispatch('destroyUser', resetBookingProcess)

      // delete cookies, if there are any
      if (this.$cookies.isKey('authorization')) {
        this.$cookies.remove('authorization')
      }

      if (this.$cookies.isKey('uid')) {
        this.$cookies.remove('uid')
      }

      // kill the products because we run into problems if a
      // user logs out and after that a new user logs in
      // without reloading the page -> then we have mixed destination products
      // in the store
      this.$store.commit('setProducts', null)

      // notify user
      if (notify)
        EventBus.$emit('notify', this.$t('notify.logoutSuccess'), 'success')

      // hide spinner
      EventBus.$emit('spinnerHide')

      // if we are in the default destination: go back to login page
      if (this.isDefaultDestination) this.$router.push({ name: 'login' })
      else if (this.$route.name !== 'booking')
        this.$router.push({ name: 'booking' })
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
