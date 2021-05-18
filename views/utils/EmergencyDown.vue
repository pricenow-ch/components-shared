<template>
  <div class="emergency-overlay">
    <!-- content -->
    <v-row class="white--text mt-12 pt-12">
      <v-col class="col-12 text-center headline-large">
        {{ text }}
      </v-col>
    </v-row>
  </div>
</template>

<script>
export default {
  name: 'EmergencyDown',
  props: {
    text: {
      type: String,
      required: false,
      default: () => {
        return i18n.t('emergencyDown.text')
      },
    },
  },
}
</script>

<style scoped>
.emergency-overlay {
  z-index: 9999;
  position: fixed;
  top: 0;
  left: 0;
  height: 100%;
  width: 100vw;
  background-color: rgba(0, 0, 0, 0.8);
}
</style>
