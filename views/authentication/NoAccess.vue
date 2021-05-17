<template>
  <div>
    <v-row class="z-index-5 justify-center pt-12" style="height: 70vh">
      <v-col class="col-12 col-md-8 mt-12">
        <v-card outlined>
          <!-- text -->
          <v-row class="ma-2">
            <v-col class="col-12 pt-12 text-center">
              <div class="headline">
                <span class="fa fa-ban primary--text mr-2" />
                {{ $t('noAccess.noAccess') }}
              </div>
            </v-col>
          </v-row>

          <!-- button to login -->
          <v-row class="pb-6">
            <v-col class="col-12 pt-12 text-center">
              <v-btn
                outlined
                color="primary"
                large
                @click="$router.push({ name: 'login' })"
              >
                {{ $t('noAccess.goToLogin') }}
              </v-btn>
            </v-col>
          </v-row>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script>
export default {
  name: 'NoAccess',
}
</script>
