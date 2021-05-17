<template>
  <v-navigation-drawer
    permanent
    :right="right"
    app
    clipped
    class="header-menu-bottom-level primary"
    width="75"
  >
    <!-- menu content -->
    <v-list class="mt-3">
      <v-list-item-group>
        <v-list-item
          v-for="route in routes"
          :key="route.title"
          :to="route"
          :disabled="isRouteLocked(route)"
          class="mb-2"
          :class="[isActiveRoute(route) ? 'white' : 'primary']"
        >
          <v-tooltip right :nudge-right="8" content-class="primary">
            <template #activator="{ on }">
              <v-list-item-content
                class="text-center"
                :class="[
                  isActiveRoute(route) ? 'primary--text' : 'white--text',
                ]"
                v-on="on"
              >
                <!-- regular icon -->
                <i
                  v-if="route.meta.icon.startsWith('fa-')"
                  :class="[route.meta.icon, 'fal']"
                  style="font-size: 1.5em"
                />

                <!-- s-icon -->
                <v-img
                  v-else-if="route.meta.icon.startsWith('s-icon-')"
                  class="py-0 mt-n2 mb-n2"
                  :src="getSIconSrc(route.meta.icon, isActiveRoute(route))"
                  contain
                  :aspect-ratio="1"
                  style="min-width: 130%; margin-left: -15%; fill: red"
                />

                <!-- lock -->
                <div class="mt-n10 ml-n6">
                  <span
                    v-if="isRouteLocked(route)"
                    style="font-size: 1.2em"
                    class="fal fa-lock"
                  >
                  </span>
                </div>
              </v-list-item-content>
            </template>
            <!-- route title as tooltip -->
            <span>{{ $t('routingTitles.' + route.name) }}</span>
          </v-tooltip>
        </v-list-item>

        <!-- switch between pe and shop backend -->
        <v-list-item
          v-if="switchRoute"
          class="mt-6 pt-3 border-top"
          @click="switchToRoute()"
        >
          <v-tooltip right :nudge-right="8" content-class="primary">
            <template #activator="{ on }">
              <v-list-item-content class="text-center white--text" v-on="on">
                <i class="fal fa-repeat" style="font-size: 1.5em" />
              </v-list-item-content>
            </template>
            <!-- description -->
            <span v-if="weAreInBackendShop">Price-Engine</span>
            <span v-else>Shop-Backend</span>
          </v-tooltip>
        </v-list-item>
      </v-list-item-group>
    </v-list>
  </v-navigation-drawer>
</template>

<script>
export default {
  name: 'BackendSideMenu',
  props: {
    color: {
      type: String,
      default: '#ffa',
      required: false,
    },
    right: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  computed: {
    // store shortcut
    routerHelper() {
      return this.$store.getters.getRouterHelperInstance()
    },

    weAreInBackendPe() {
      return this.$route.path.startsWith(this.routerHelper.getBackendPeUrl())
    },

    weAreInBackendShop() {
      return this.$route.path.startsWith(this.routerHelper.getBackendShopUrl())
    },
  },

  asyncComputed: {
    routes: {
      async get() {
        if (this.weAreInBackendPe) {
          // returning all routes with /backend/pricing/ in the path
          return this.routerHelper.getBackendPeRoutes(true, true, true)
        } else if (this.weAreInBackendShop) {
          // returning all routes with /backend/shop/ in the path
          return this.routerHelper.getBackendShopRoutes(true, true, true)
        }
      },
      watch: ['$route'],
    },

    switchRoute: {
      async get() {
        // are we in shop or pe backend ?
        if (this.weAreInBackendPe) {
          // we are in pe backend
          let shopBackendRoutes = await this.routerHelper.getBackendShopRoutes(
            true,
            false,
            true
          )

          // if we got any
          if (shopBackendRoutes.length) return shopBackendRoutes[0]
          else return null
        } else if (this.weAreInBackendShop) {
          const canISwitchToPricingDashboard = await this.$store.getters
            .getAppUserInstance()
            .doIHavePermissionForPricingDashboard()
          if (canISwitchToPricingDashboard) {
            return process.env.VUE_APP_PRICING_DASHBOARD_URL
          }
          return null
        } else {
          return null
        }
      },
      watch: ['$route'],
    },
  },

  methods: {
    isRouteLocked(route) {
      return (
        route.meta &&
        route.meta.withLock &&
        route.meta.onlyGrantedAccessForLockIcon
      )
    },

    switchToRoute() {
      if (this.switchRoute) {
        window.location.href = this.switchRoute
      }
    },

    isActiveRoute(route) {
      return this.$route.name === route.name
    },

    getSIconSrc(icon, isActive) {
      return require('@/assets/icons/shortCutSection/' +
        (isActive ? 'primary' : 'white') +
        '/' +
        icon +
        '.svg')
    },
  },
}
</script>
