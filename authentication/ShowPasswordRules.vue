<template>
  <v-layout
    v-if="!isPasswordValid && passwordTouched"
    class="wrap password-rules"
    :class="[
      vLayoutClass,
      {
        'fill-height': $vuetify.breakpoint.smAndUp,
        'password-rules-align-top-sm-and-down': passwordRulesAlignTopSmAndDown,
      },
    ]"
  >
    <v-flex :class="[vFlexClass]">
      <v-card class="pa-4 white">
        <!-- title -->
        <v-layout class="wrap mb-4">
          <v-flex class="xs12">
            <div class="content-2">
              {{ $t('registration.passwordConstraints') }}
            </div>
          </v-flex>
        </v-layout>

        <!-- constraints -->
        <v-layout class="wrap v-label content-3">
          <!-- is the password long enough? -->
          <v-flex v-if="!passwordLength" class="xs12 content-4 mb-4">
            <span class="fa fa-times error--text mr-2"></span>
            {{ $t('registration.passwordLength') }}
          </v-flex>

          <!-- does the password contain any number? -->
          <v-flex v-if="!passwordHasNumber" class="xs12 content-4 mb-4">
            <span class="fa fa-times error--text mr-2"></span>
            {{ $t('registration.passwordHasNumber') }}
          </v-flex>

          <!-- NOT IN USE does the password contain any special character? -->
          <v-flex
            v-if="false && !passwordHasSpecialCharacter"
            class="xs12 content-4 mb-4"
          >
            <span class="fa fa-times error--text mr-2"></span>
            {{ $t('registration.passwordHasSpecialCharacter') }}
          </v-flex>

          <!-- NOT IN USE does the password contain at least one capital letter? -->
          <v-flex
            v-if="false && !passwordHasCapitalLetter"
            class="xs12 content-4 mb-4"
          >
            <span class="fa fa-times error--text mr-2"></span>
            {{ $t('registration.passwordHasCapitalLetter') }}
          </v-flex>

          <!-- does the password contain at least one letter? -->
          <v-flex v-if="!passwordHasLetters" class="xs12 content-4 mb-4">
            <span class="fa fa-times error--text mr-2"></span>
            {{ $t('registration.passwordHasLetters') }}
          </v-flex>

          <!-- NOT IN USE does the password is not the uid? -->
          <v-flex
            v-if="false && !passwordDoesNotContainPersonalInformation"
            class="xs12 content-4 mb-4"
          >
            <span class="fa fa-times error--text mr-2"></span>
            {{ $t('registration.passwordIsNotUid') }}
          </v-flex>

          <!-- are the passwords the same? -->
          <v-flex v-if="!passwordEqual" class="xs12 content-4 mb-4">
            <span class="fa fa-times error--text mr-2"></span>
            {{ $t('registration.passwordEqual') }}
          </v-flex>
        </v-layout>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
export default {
  name: 'ShowPasswordRules',

  props: {
    passwordTouched: {
      type: Boolean,
      required: true,
      default: false,
    },

    isPasswordValid: {
      type: Boolean,
      required: true,
      default: false,
    },

    passwordLength: {
      type: Boolean,
      required: true,
      default: false,
    },

    passwordHasNumber: {
      type: Boolean,
      required: true,
      default: false,
    },

    passwordHasLetters: {
      type: Boolean,
      required: true,
      default: false,
    },

    passwordEqual: {
      type: Boolean,
      required: true,
      default: false,
    },

    // width and offset of the component
    vFlexClass: {
      type: String,
      required: false,
      default: 'xs12',
    },

    // component layout
    vLayoutClass: {
      type: String,
      required: false,
      default: '',
    },

    // allign password constraints top on mobile
    passwordRulesAlignTopSmAndDown: {
      type: Boolean,
      required: false,
      return: false,
    },
  },
}
</script>

<style lang="scss">
.password-rules {
  position: fixed;
  right: 0;
  top: 0;
  height: 100%;
  width: 100%;
}

@media screen and (max-width: $md) {
  .password-rules {
    z-index: 2;
    bottom: 0;
    top: inherit;
    height: inherit;
  }

  .password-rules-align-top-sm-and-down {
    top: 55px !important;
    height: auto !important;
    bottom: inherit !important;
  }
}
</style>
