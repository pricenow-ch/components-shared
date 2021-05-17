<script>
export default {
  name: 'NavigationStoreMixin',
  computed: {
    storeObject() {
      return this.$store.getters
        .getNavigationStores()
        .getNavigationStoreByRouteObject(this.$route).storeObject
    },
  },
  // init navigation store
  beforeCreate() {
    this.$store.getters.getNavigationStores().initNavigationStore(this.$route)
  },
}
</script>
