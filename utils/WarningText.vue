<template>
  <v-row :class="justify" class="flex-nowrap align-center no-gutters">
    <v-col class="col-auto py-0">
      <span
        class="fal fa-exclamation-triangle grey--text text--lighten-1 fa-2x"
      />
    </v-col>
    <v-col class="col-10 font-weight-thin content-4 pl-4">
      <slot />
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'WarningText',
  props: {
    justify: {
      type: String,
      required: false,
      default: 'justify-center',
    },
  },
}
</script>
