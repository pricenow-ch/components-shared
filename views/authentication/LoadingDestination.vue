<template>
  <v-row class="h-100 align-center">
    <v-col class="col-12 text-center">
      <span class="fal fa-circle-notch fa-spin fa-3x" />
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'LoadingDestination',
  data() {
    return {
      toRoute: null,
    }
  },
  computed: {
    appDestination() {
      return this.$store.getters.getAppDestinationInstance()
    },
  },
  watch: {
    appDestination() {
      this.toRoute()
    },
  },
  beforeMount() {
    this.toRoute = this.$route.params.toRoute
    if (this.$store.getters.getAppDestinationInstance()) {
      this.route()
    }
  },
  methods: {
    route() {
      this.$router.replace(this.toRoute)
    },
  },
}
</script>
