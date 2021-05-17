<template>
  <v-row v-if="itemsPerPage" class="fill-height align-center">
    <!-- items per page -->
    <v-col class="col-4">
      <v-select
        :value="itemsPerPage"
        :items="rowsPerPage"
        :label="$t('general.rowsPerPage')"
        @change="
          (itemsPerPage) => {
            this.$emit('update:itemsPerPage', itemsPerPage)
          }
        "
      >
      </v-select>
    </v-col>

    <!-- previous page -->
    <v-col class="col-3 text-center">
      <v-btn
        depressed
        fab
        x-small
        color="primary"
        :disabled="page === 1"
        @click="
          () => {
            this.$emit('previousPage')
          }
        "
      >
        <span class="fa fa-chevron-left"></span>
      </v-btn>
    </v-col>

    <!-- page -->
    <v-col
      v-if="!disablePageCountDisplay"
      class="col-2 content-4 text-center pa-0"
    >
      {{ page }} / {{ pageCount }}
    </v-col>

    <!-- next page -->
    <v-col class="col-3 text-center">
      <v-btn
        depressed
        fab
        x-small
        :disabled="page >= pageCount || disableNextArrow"
        color="primary"
        @click="
          () => {
            this.$emit('nextPage')
          }
        "
      >
        <span class="fa fa-chevron-right"></span>
      </v-btn>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'TableFooter',

  props: {
    // current displayed items per page
    itemsPerPage: {
      type: Number,
      required: true,
      default: 10,
    },
    // dont show the x pages of y info, used when you dont know how many pages are available
    disablePageCountDisplay: {
      type: Boolean,
      required: false,
      default: false,
    },

    // when no more pages are available disable that functionality
    disableNextArrow: {
      type: Boolean,
      required: false,
      default: false,
    },

    // current page
    page: {
      type: Number,
      required: true,
      default: 1,
    },

    // max pages
    pageCount: {
      type: Number,
      required: true,
      default: null,
    },
  },

  data() {
    return {
      rowsPerPage: [10, 20, 30, 40, 50],
    }
  },
}
</script>
