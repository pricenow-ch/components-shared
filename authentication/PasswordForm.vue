<template>
  <div>
    <!-- TODO this form is only used ind confirmMail.vue so far. However, it's meant to use in profile.vue as well as in registrationForm.vue -->


    <!-- password input fields -->
    <v-form ref="passwordFormContainer" @submit.prevent="">
      <v-layout class="wrap">
        <!-- password1 -->
        <v-flex class="xs12">
          <v-text-field
            id="password"
            v-model="password1"
            :rules="passwordRulesWithMessage"
            :label="$t('registration.password')"
            :append-icon="showPassword ? 'visibility_off' : 'visibility'"
            :type="showPassword ? 'text' : 'password'"
            @focus="passwordTouched = true"
            @click:append="showPassword = !showPassword"
          ></v-text-field>
        </v-flex>

        <!-- password2 -->
        <v-flex class="xs12">
          <v-text-field
            v-model="password2"
            :rules="passwordEqualWithMessage"
            :label="$t('registration.passwordRepete')"
            :append-icon="showPassword ? 'visibility_off' : 'visibility'"
            :type="showPassword ? 'text' : 'password'"
            @click:append="showPassword = !showPassword"
          ></v-text-field>
        </v-flex>
      </v-layout>
    </v-form>
  </div>
</template>

<script>
import PasswordRulesMixin from '../mixins/PasswordRulesMixin.vue'

export default {
  name: 'PasswordForm',

  mixins: [PasswordRulesMixin],

  data() {
    return {
      // helper: don't show password policy, if not yet touched password input fields
      passwordTouched: false,

      // show password as text
      showPassword: false,

      // first password
      password1: '',

      // repeat password
      password2: '',
    }
  },

  methods: {
    // validate form
    validate() {
      return this.$refs.passwordFormContainer.validate()
    },

    // get password
    getPassword() {
      return this.password1
    },
  },
}
</script>

<style scoped></style>
