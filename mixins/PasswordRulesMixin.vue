<template>
  <div></div>
</template>

<script>
export default {
  name: 'PasswordRulesMixin',

  data() {
    return {
      password1: '',
      password2: '',
      passwordRulesWithMessage: [
        (v) => !!v || this.$t('registration.passwordRequired'),
        () => this.passwordLength() || this.$t('registration.passwordLength'),
        () =>
          this.passwordHasNumber() || this.$t('registration.passwordHasNumber'),
        () =>
          this.passwordHasLetters() ||
          this.$t('registration.passwordHasLetters'),
      ],
      passwordEqualWithMessage: [
        (v) => !!v || this.$t('registration.passwordRequired'),
        () => this.passwordEqual() || this.$t('registration.passwordEqual'),
      ],
    }
  },

  methods: {
    // compare the two new Passwords
    isPasswordValid(checkEqual = true) {
      return (
        (checkEqual ? this.passwordEqual() : true) &&
        this.passwordLength() &&
        this.passwordHasNumber() &&
        // this.passwordHasSpecialCharacter() &&
        // this.passwordHasCapitalLetter() &&
        this.passwordHasLetters()
      ) // &&
      // this.passwordDoesNotContainPersonalInformation()
    },

    passwordLength() {
      return this.password1.length >= 8
    },

    passwordEqual() {
      return this.password1 === this.password2
    },

    passwordHasNumber() {
      return /[0-9]+/.test(this.password1)
    },

    passwordHasSpecialCharacter() {
      return /[*@!#%&()^~{}_`´"·$/=?¿¡'ºª]+/.test(this.password1)
    },

    passwordHasCapitalLetter() {
      return /[A-Z]+/.test(this.password1)
    },

    passwordHasSmallLetter() {
      return /[a-z]+/.test(this.password1)
    },

    passwordHasLetters() {
      return this.passwordHasCapitalLetter() && this.passwordHasSmallLetter()
    },

    passwordDoesNotContainPersonalInformation() {
      return (
        !this.password1.includes(this.eMail) &&
        !this.password1.includes(this.firstName) &&
        !this.password1.includes(this.lastName)
      )
    },
  },
}
</script>
