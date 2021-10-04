<template>
  <v-menu bottom offset-y>
    <template #activator="{ on }">
      <!-- Button -->
      <v-btn
        large
        :outlined="outlined"
        :style="{ backgroundColor: outlined ? 'white' : '' }"
        color="primary"
        :loading="seasons.length === 0"
        max-width="100%"
        :disabled="disabled"
        v-on="on"
      >
        <!-- prefix icon -->
        <span v-if="prefixIcon" :class="prefixIcon" class="mr-2" />
        <div v-if="selectedSeasonInstance">
          {{ selectedSeasonInstance.name }}
        </div>
        <span v-else>
          <!-- todo add translation-->
          {{ 'select a season' }}
        </span>
      </v-btn>
    </template>
    <v-list subheader class="scrollable-list">
      <v-list-item
        v-for="(season, seasonIndex) in seasons"
        :key="seasonIndex"
        @click="clickedSeason(season)"
      >
        <v-list-item-content class="py-0" :class="{ 'mt-3': season === 0 }">
          <v-list-item-title>
            <v-row>
              <v-col class="col-12 ml-2">
                <span v-if="$store.getters.getExtTransLoaded()">
                  {{ season.name }}
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
import Season from '@/classes-shared/season/Season'

export default {
  name: 'SeasonSelector',
  props: {
    // An array with the seasons to display
    seasons: {
      type: Array,
      required: true,
    },

    selectedSeasonInstance: {
      type: [Season],
      required: false,
      default: null,
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

    disabled: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  methods: {
    clickedSeason(season) {
      this.$emit('update:selectedSeasonInstance', season)
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
