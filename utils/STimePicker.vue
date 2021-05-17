<!-- wraps and extends vuetify's time picker with an editable header -->
<template>
  <v-row>
    <v-col class="col-12 pa-0">
      <v-row no-gutters>
        <!-- editable header -->
        <v-text-field
          v-mask="mask"
          outlined
          :value="time"
          :label="label"
          append-icon="far fa-clock fa-md"
          :disabled="disabled"
          :rules="rules"
          @input="change"
        />
      </v-row>

      <v-row no-gutters class="justify-center">
        <!-- vuetify's time picker -->
        <v-time-picker
          v-if="showClock"
          no-title
          format="24hr"
          :value="time"
          color="primary"
          :disabled="disabled"
          class="mt-n2 elevation-0"
          @input="change"
        />
      </v-row>
    </v-col>
  </v-row>
</template>

<script>
import { mask } from 'vue-the-mask'

export default {
  name: 'STimePicker',

  directives: {
    mask,
  },

  props: {
    time: {
      type: String,
      required: true,
      default: '00:00',
    },

    label: {
      type: String,
      required: false,
      default: '',
    },

    disabled: {
      type: Boolean,
      required: false,
      default: false,
    },

    rules: {
      type: Array,
      required: false,
      default: () => {
        return []
      },
    },

    showClock: {
      type: Boolean,
      required: false,
      default: true,
    },
  },

  data: () => ({
    mask: '##:##',
  }),

  methods: {
    change(time) {
      this.$emit('update:time', time)
    },
  },
}
</script>
