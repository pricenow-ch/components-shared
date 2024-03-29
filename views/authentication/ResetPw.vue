<template>
  <div>
    <v-layout
      class="wrap justify-center z-index-7 mt-12 pt-12"
    >
      <v-flex class="xs12 md7">
        <v-card>
          <v-layout class="wrap justify-center px-6 pb-6">
            <v-flex class="xs12">
              <!-- headline -->
              <v-layout class="pt-12 pb-2">
                <v-flex class="text-center xs12">
                  <div class="headline">
                    {{ $t('resetPw.headline') }}
                  </div>
                </v-flex>
              </v-layout>

              <!-- 1. step: E-Mail to reset pw -->
              <div v-if="firstStep">
                <!-- explanation -->
                <v-layout class="pb-6 justify-center">
                  <v-flex>
                    <div class="content-4 text-left">
                      {{ $t('resetPw.explanation') }}
                    </div>
                  </v-flex>
                </v-layout>

                <!-- mail input field -->
                <v-layout class="mb-4 wrap">
                  <v-flex class="xs12 md6">
                    <v-form ref="emailForm" @submit.prevent="">
                      <v-text-field
                        ref="mail"
                        v-model="mail"
                        :rules="$store.getters.getMailRules()"
                        :label="$t('general.email')"
                        required
                        outlined
                        @keyup.enter="resetPw()"
                      />
                    </v-form>
                  </v-flex>
                  <v-flex class="xs12 md6 text-center text-md-right">
                    <v-btn outlined large @click="resetPw()">
                      {{ $t('resetPw.resetPwButton') }}
                    </v-btn>
                  </v-flex>
                </v-layout>
              </div>

              <!-- 2. step: input code and new pw -->
              <div v-if="!firstStep" class="pb-6">
                <v-form ref="pwForm" @submit.prevent="">
                  <!-- security code from mail goes here -->
                  <v-layout>
                    <v-flex>
                      <v-text-field
                        v-model="code"
                        :rules="[(v) => !!v || this.$t('resetPw.codeRequired')]"
                        :label="$t('resetPw.resetCode')"
                        outlined
                        required
                      />
                    </v-flex>
                  </v-layout>

                  <!-- new pw1 -->
                  <v-layout justify-center wrap>
                    <v-flex>
                      <v-text-field
                        v-model="password1"
                        :rules="passwordRulesWithMessage"
                        :label="$t('resetPw.newPw')"
                        :append-icon="showPw ? 'visibility_off' : 'visibility'"
                        :type="showPw ? 'text' : 'password'"
                        outlined
                        required
                        @click:append="showPw = !showPw"
                      />
                    </v-flex>
                  </v-layout>

                  <!-- new pw2 -->
                  <v-layout justify-center wrap>
                    <v-flex>
                      <v-text-field
                        v-model="password2"
                        :rules="passwordEqualWithMessage"
                        :label="$t('resetPw.newPwRepeat')"
                        :append-icon="showPw ? 'visibility_off' : 'visibility'"
                        :type="showPw ? 'text' : 'password'"
                        outlined
                        required
                        @click:append="showPw = !showPw"
                      />
                    </v-flex>
                  </v-layout>

                  <!-- reset pw button -->
                  <v-layout justify-center class="pt-6">
                    <v-flex xs12 sm8 md4>
                      <v-btn class="ml-0" outlined large @click="sendNewPw()">
                        {{ $t('profile.changePassword') }}
                      </v-btn>
                    </v-flex>
                  </v-layout>
                </v-form>
              </div>
            </v-flex>
          </v-layout>
        </v-card>
      </v-flex>
    </v-layout>
  </div>
</template>

<script>
import PasswordRulesMixin from '../../mixins/PasswordRulesMixin.vue'
import { shopInstance } from '../../../classes-shared/utils/axiosInstance'

export default {
  name: 'ResetPw',

  mixins: [PasswordRulesMixin],

  data() {
    return {
      code: '',
      mail: '',
      firstStep: true,
      showPw: false,
    }
  },

  methods: {
    // 1. step to reset the pw: send email to get confirmation code
    resetPw() {
      if (this.$refs.emailForm.validate()) {
        /* global EventBus axios */
        EventBus.$emit('spinnerShow')

        shopInstance()
          .patch(`/user/${this.mail}/initPasswordReset`)
          .then(() => {
            EventBus.$emit(
              'notify',
              this.$root.$t('notify.pwResetCodeSentToMail'),
              'success'
            )

            // change to 2. step
            this.firstStep = false
          })
          .catch((error) => {
            if (error.response.status === 400) {
              EventBus.$emit('notify', this.$root.$t('resetPw.userNotFound'))
            } else {
              EventBus.$emit('notify')
            }
          })
          .finally(() => {
            EventBus.$emit('spinnerHide')
          })
      }
    },

    // 2. step to reset the pw: send code received via email and send new pw
    sendNewPw() {
      if (this.$refs.pwForm.validate()) {
        /* global EventBus axios */
        EventBus.$emit('spinnerShow')

        shopInstance()
          .patch(`/user/${this.mail}/passwordReset`, {
            passwordResetKey: this.code,
            password: this.password1,
          })
          .then(() => {
            EventBus.$emit(
              'notify',
              this.$root.$t('notify.passwordResetSuccessfull'),
              'success'
            )

            // route user to login form
            this.$router.push({ name: 'login' })
          })
          .catch((error) => {
            if (error.response.status === 400) {
              // wrong key was sent
              EventBus.$emit(
                'notify',
                this.$root.$t('notify.wrongCodeSent'),
                'error'
              )
            } else {
              EventBus.$emit('notify')
            }
          })
          .finally(() => {
            EventBus.$emit('spinnerHide')
          })
      }
    },
  },
}
</script>
