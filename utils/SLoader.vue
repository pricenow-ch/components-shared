<template>
  <v-row>
    <v-col class="col-12 text-center opacity">
      <!-- spinning circle -->
      <div class="mb-4">
        <span class="fal fa-3x fa-circle-notch fa-spin"></span>
      </div>

      <!-- waiting text -->
      <span v-if="text" class="headline-large">
        {{ text }}
      </span>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'SLoader',

  props: {
    text: {
      type: String,
      required: false,
      default: '',
    },
  },
}
</script>

<style scoped></style>
