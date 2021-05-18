<template>
  <div>
    <v-layout class="wrap justify-center align-center" style="height: 70vh">
      <v-flex class="xs12 md8">
        <v-card
          text
          :class="{
            'pa-2': $vuetify.breakpoint.smAndDown,
            'pa-5': $vuetify.breakpoint.mdAndUp,
          }"
        >
          <!-- confirm section -->
          <div v-if="!confirmationSuccess">
            <!-- no password setting needed -->
            <v-layout v-if="!needPassword" class="wrap">
              <v-flex class="xs12 text-center">
                <div class="headline">
                  {{ $t('v3.pleaseConfirmMail') }}
                </div>
              </v-flex>

              <!-- mail -->
              <v-flex class="xs12 text-center mt-6">
                <div class="content-3">
                  <span class="fas fa-envelope mr-2"></span>
                  {{ $route.params.mail }}
                </div>
              </v-flex>
            </v-layout>

            <!-- password setting required -->
            <v-layout v-else class="wrap justify-center">
              <v-flex class="xs12 text-center">
                <div class="headline">
                  {{ $t('v3.pleaseSetPw') }}
                </div>
              </v-flex>

              <!-- mail -->
              <v-flex class="xs12 text-center mt-6">
                <div class="content-3">
                  <span class="fas fa-envelope mr-2"></span>
                  {{ $route.params.mail }}
                </div>
              </v-flex>

              <!-- password form -->
              <v-flex class="xs12 md6 mt-6">
                <password-form ref="passwordForm"></password-form>
              </v-flex>
            </v-layout>

            <!-- button -->
            <v-layout class="wrap">
              <v-flex class="xs12 text-center mt-6">
                <v-btn text large @click="confirmMail()">
                  {{ $t('v3.confirm') }}
                </v-btn>
              </v-flex>
            </v-layout>
          </div>

          <!-- confirmation was successfull -->
          <v-layout v-else class="wrap">
            <v-flex class="xs12 text-center">
              <div class="headline">
                {{ $t('v3.mailSuccessfullyConfirmed') }}
              </div>
            </v-flex>

            <!-- mail -->
            <v-flex class="xs12 text-center mt-6">
              <div class="headline-large success--text">
                <span class="fa fa-check"></span>
              </div>
            </v-flex>

            <!-- button -->
            <v-flex class="xs12 text-center mt-12">
              <v-btn text large @click="$router.push({ name: 'login' })">
                {{ $t('v3.goToLogin') }}
              </v-btn>
            </v-flex>
          </v-layout>
        </v-card>
      </v-flex>
    </v-layout>
  </div>
</template>

<script>
import { shopInstance } from '../../../classes-shared/utils/axiosInstance'
import PasswordForm from '../../authentication/PasswordForm'
export default {
  name: 'ConfirmMail',
  components: { PasswordForm },
  data() {
    return {
      confirmationSuccess: false,

      // password needs to be set by user
      needPassword: false,
    }
  },

  mounted() {
    // decide whether to set a pw or not
    this.needPassword = this.$route.params.mode == 1
  },

  methods: {
    // TODO: Rückmeldung an User, falls Adresse bereits bestätigt wurde (api pending)
    // TODO: Rückmeldung an User, falls Passwort generiert wurde oder er ein neues setzen soll (api pending)
    confirmMail() {
      // check if pw is complete or not used
      if (!this.needPassword || this.$refs.passwordForm.validate()) {
        /* global EventBus axios */
        EventBus.$emit('spinnerShow')

        shopInstance()
          .patch('/user/mailValidation', {
            mail: this.$route.params.mail,
            mailConfirmationCode: this.$route.params.code,
            password: this.needPassword
              ? this.$refs.passwordForm.getPassword()
              : '',
          })
          .then(() => {
            this.confirmationSuccess = true
          })
          .catch((error) => {
            EventBus.$emit('notify', error.response.status)
          })
          .finally(() => {
            EventBus.$emit('spinnerHide')
          })
      }
    },
  },
}
</script>

<style scoped></style>
