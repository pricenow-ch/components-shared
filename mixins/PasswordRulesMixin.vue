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
      passwordRules: [
        (v) => !!v || this.$t('registration.passwordWrong'),
        this.checkFirst,
      ],
      password1Rules: [
        (v) => !!v || this.$t('registration.passwordWrong'),
        this.checkPassword1,
      ],
    }
  },

  methods: {
    checkFirst() {
      return this.password2 == '' || this.checkPasswords()
    },

    checkPasswords() {
      return this.isPasswordValid() || this.$t('registration.passwordWrong')
    },

    checkPassword1() {
      return (
        this.isPasswordValid(false) || this.$t('registration.passwordWrong')
      )
    },

    // compare the two new Passwords
    isPasswordValid(checkEqual = true) {
      return (
        (checkEqual ? this.passwordEqual() : true) &&
        this.passwordLength() &&
        this.passwordHasNumber() &&
        // this.passwordHasSpecialCharacter() &&
        // this.passwordHasCapitalLetter() &&
        this.passwordHasSmallLetter() &&
        this.passwordHasCapitalLetter()
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
