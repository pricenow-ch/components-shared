<template>
  <div>
    <v-form ref="form" @submit.prevent="">
      <v-row class="pt-8 justify-center no-gutters">
        <v-col class="col-12 col-sm-8 z-index-5">
          <v-card outlined>
            <v-card-title>
              {{ $t('profile.personalInformation') }}
            </v-card-title>
            <v-card-text>
              <v-row>
                <!-- salutation -->
                <v-col class="col-12 py-0">
                  <v-radio-group
                    v-if="salutations.salutation"
                    v-model="salutations.salutation.salutationKey"
                    :label="$t('registrationForm.salutation')"
                    row
                    :rules="[
                      (v) => !!v || $t('registrationForm.selectSalutation'),
                    ]"
                  >
                    <v-radio
                      v-for="(
                        salutation, salutationIndex
                      ) in salutations.salutations"
                      :key="salutationIndex"
                      :label="salutation.getTitle()"
                      :value="salutation.salutationKey"
                    />
                  </v-radio-group>
                </v-col>

                <!-- firstname -->
                <v-col class="col-12 col-md-6 pr-md-2 py-0">
                  <v-text-field
                    v-model="firstName"
                    outlined
                    :rules="[(v) => !!v || this.$t('rule.inputRequired')]"
                    :label="$t('registration.firstName')"
                    required
                  >
                  </v-text-field>
                </v-col>

                <!-- lastname -->
                <v-col class="col-12 col-md-6 py-0">
                  <v-text-field
                    v-model="lastName"
                    outlined
                    :rules="[(v) => !!v || this.$t('rule.inputRequired')]"
                    :label="$t('registration.lastName')"
                    required
                  >
                  </v-text-field>
                </v-col>
              </v-row>

              <!-- new row -->
              <v-row class="pl-2 pr-2">
                <v-col class="col-12 col-md-6 pr-md-2 py-0">
                  <!-- birthday -->
                  <s-birthday-picker :date.sync="birthday" outlined />
                </v-col>

                <!-- country -->
                <v-col class="col-12 col-md-6 py-0">
                  <s-country
                    ref="country"
                    required
                    :country.sync="country"
                    outlined
                  />
                </v-col>
              </v-row>

              <!-- new row -->
              <v-row class="px-2">
                <!-- address -->
                <v-col class="col-12 col-md-6 pr-md-2 py-0">
                  <v-text-field
                    v-model="address"
                    outlined
                    :label="$t('registration.streetStar')"
                    :rules="[
                      (v) => !!v || this.$t('registration.addressRequired'),
                    ]"
                    validate-on-blur
                  >
                  </v-text-field>
                </v-col>

                <!-- zip -->
                <v-col class="col-12 col-md-2 pr-md-2 py-0">
                  <v-text-field
                    v-model="zip"
                    outlined
                    :rules="[(v) => !!v || this.$t('registration.zipRequired')]"
                    :label="$t('registration.zip')"
                  >
                  </v-text-field>
                </v-col>

                <!-- city -->
                <v-col class="col-12 col-md-4 py-0">
                  <v-text-field
                    v-model="city"
                    outlined
                    :label="$t('registration.cityStar')"
                    :rules="[
                      (v) => !!v || this.$t('registration.cityRequired'),
                    ]"
                  >
                  </v-text-field>
                </v-col>
              </v-row>

              <!-- new row -->
              <v-row class="px-2">
                <!-- phone -->
                <v-col class="col-12 col-md-6 pr-md-2 py-0">
                  <v-text-field
                    v-model="phone"
                    outlined
                    :label="$t('general.phone')"
                  >
                  </v-text-field>
                </v-col>

                <!-- email -->
                <v-col class="col-12 col-md-6 py-0">
                  <v-text-field
                    v-model="mail"
                    outlined
                    :label="$t('registration.email')"
                    @keydown="$refs.mailModal.show()"
                  >
                  </v-text-field>
                </v-col>
              </v-row>

              <!-- buttons -->
              <v-row class="py-4 px-2">
                <!-- save -->
                <v-col class="col-12 text-center py-0">
                  <v-btn large outlined color="primary" @click="saveProfile()">
                    {{ $t('general.save') }}
                  </v-btn>
                </v-col>
              </v-row>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>

      <!-- ski cards -->
      <v-row
        v-if="!isDefaultDestination && appUser"
        class="pt-8 justify-center no-gutters"
      >
        <v-col class="col-12 col-sm-8">
          <v-card outlined>
            <v-card-title>
              {{ $t('profile.myCards') }}
            </v-card-title>
            <v-card-text>
              <cards-list
                class="pl-2 pr-2"
                :cards="appUser.getCards()"
                :uid="appUser.getId()"
                @refresh="$store.dispatch('getMainUserFromAPI')"
              />
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-form>

    <!-- change password card -->
    <v-row class="pt-8 justify-center no-gutters">
      <v-col class="col-12 col-sm-8">
        <v-card outlined>
          <v-card-title>
            {{ $t('general.password') }}
          </v-card-title>
          <v-card-text>
            <show-password-rules
              vLayoutClass="align-end pb-6"
              vFlexClass="xs12 md3 offset-md9"
              passwordRulesAlignTopSmAndDown
              :isPasswordValid="isPasswordValid()"
              :passwordTouched="passwordTouched"
              :passwordLength="passwordLength()"
              :passwordHasNumber="passwordHasNumber()"
              :passwordHasLetters="
                passwordHasSmallLetter() && passwordHasCapitalLetter()
              "
              :passwordEqual="passwordEqual()"
            />
            <v-form ref="passwordForm" style="z-index: 0" @submit.prevent="">
              <v-row class="pt-6 pb-4 px-2">
                <!-- pw1 -->
                <v-col class="col-12 col-md-6 pr-2 py-0">
                  <v-text-field
                    id="password"
                    v-model="password1"
                    outlined
                    validate-on-blur
                    :rules="password1Rules"
                    :label="$t('registration.password')"
                    :append-icon="showPw ? 'visibility_off' : 'visibility'"
                    :type="showPw ? 'text' : 'password'"
                    required
                    @click:append="showPw = !showPw"
                    @focus="passwordTouched = true"
                  />
                </v-col>

                <!-- pw2 -->
                <v-col class="col-12 col-md-6 pr-2 py-0">
                  <v-text-field
                    v-model="password2"
                    outlined
                    validate-on-blur
                    :rules="passwordRules"
                    :label="$t('registration.passwordRepete')"
                    :append-icon="showPw ? 'visibility_off' : 'visibility'"
                    :type="showPw ? 'text' : 'password'"
                    required
                    @click:append="showPw = !showPw"
                  />
                </v-col>
                <!-- change password -->
                <v-col class="col-12 text-center py-0">
                  <v-btn
                    class="ml-0"
                    color="primary"
                    outlined
                    large
                    @click="changePassword()"
                  >
                    {{ $t('profile.changePassword') }}
                  </v-btn>
                </v-col>
              </v-row>
            </v-form>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- deactivate user -->
    <v-row class="pt-8 justify-center no-gutters">
      <v-col class="col-12 col-sm-8">
        <v-card outlined>
          <v-card-title>
            {{ $t('profile.deactivateProfile') }}
          </v-card-title>
          <v-card-text>
            <v-row>
              <v-col class="col-12 text-center">
                <v-btn
                  color="primary"
                  class="mt-4 mb-2"
                  outlined
                  @click="$refs.deactivateProfile.show()"
                >
                  <span class="fal fa-trash mr-2" />
                  {{ $t('profile.deactivateProfile') }}
                </v-btn>
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- back button for default destination -->
    <v-row v-if="isDefaultDestination" class="py-8 justify-center no-gutters">
      <v-col class="col-12 col-sm-8">
        <v-btn color="primary" large @click="$router.go(-1)">
          <span class="fa fa-chevron-left mr-2" />
          {{ $t('general.back') }}
        </v-btn>
      </v-col>
    </v-row>

    <!-- modal to change mail -->
    <s-modal
      ref="mailModal"
      :hide-on-ok="false"
      :size="2"
      :headerText="$t('profile.changeMailModalHeader')"
      @ok="checkMail()"
    >
      <v-form ref="newMailForm" @submit.prevent="">
        <v-row class="justify-center">
          <!-- email 1 -->
          <v-col class="col-12 col-md-6 pr-2">
            <v-text-field
              v-model="newMail1"
              :rules="$store.getters.getMailRules()"
              :label="$t('profile.newMailFirst')"
            >
            </v-text-field>
          </v-col>

          <!-- email 2 -->
          <v-col class="col-12 col-md-6 pr-2">
            <v-text-field
              v-model="newMail2"
              :rules="$store.getters.getMailRules()"
              :label="$t('profile.newMailRepeat')"
            >
            </v-text-field>
          </v-col>
        </v-row>
      </v-form>
    </s-modal>

    <!-- deactivate profile modal -->
    <s-modal
      ref="deactivateProfile"
      :size="1"
      :ok-text="$t('profile.deactivateProfile')"
      :header-text="$t('profile.reallyDeactivateProfile')"
      :hide-on-ok="false"
      ok-color="red"
      ok-icon="fa fa-trash mr-2"
      cancel-color="green"
      @ok="deactivateMyProfile()"
    >
      <v-form v-if="appUser" ref="deactivateModalForm" @submit.prevent="">
        <v-row>
          <v-col class="col-12">
            <span class="fal fa-exclamation-triangle mr-2 red--text" />
            <span v-html="$t('profile.reallyDeleteText')" />
          </v-col>

          <v-col class="col-6 mt-2">
            {{ $t('profile.enterYourNameForSecurityReasons') }} (<b>{{
              appUser.getFirstName()
            }}</b
            >):
          </v-col>
          <v-col class="col-6">
            <v-text-field
              v-model="securityName"
              :label="$t('profile.yourName')"
              outlined
              :rules="[
                (v) =>
                  (!!v && v === appUser.getFirstName()) ||
                  $t('profile.doesNotMatchWithYourName'),
              ]"
              validate-on-blur
            />
          </v-col>
        </v-row>
      </v-form>
    </s-modal>
  </div>
</template>

<script>
import PasswordRulesMixin from '../mixins/PasswordRulesMixin.vue'
import ShowPasswordRules from '@/components-shared/authentication/ShowPasswordRules.vue'
import CardsList from '@/components-shared/cards/CardsList.vue'
import SCountry from '@/components-shared/utils/SCountry'
import SBirthdayPicker from '@/components-shared/utils/SBirthdayPicker'
import Salutations from '@/classes/salutations/Salutations'
import { shopInstance } from '@/classes/utils/axiosInstance'

export default {
  name: 'Profile',

  components: {
    SCountry,
    ShowPasswordRules,
    CardsList,
    SBirthdayPicker,
  },

  mixins: [PasswordRulesMixin],

  data() {
    return {
      showPw: false,
      passwordTouched: false,
      favoriteSkiRessort: null,
      favoriteSkiMask: null,
      education: null,
      profession: null,
      perfectSkiDay: null,
      newMail1: null,
      newMail2: null,
      birthday: null,
      firstName: null,
      lastName: null,
      address: null,
      zip: null,
      city: null,
      country: null,
      phone: null,
      salutations: new Salutations(),
      securityName: null,
    }
  },

  computed: {
    mail() {
      if (this.appUser) return this.appUser.getMail()
      return null
    },

    appUser() {
      return this.$store.getters.getAppUserInstance()
    },
  },

  asyncComputed: {
    async isDefaultDestination() {
      return (
        this.$store.getters.getAppDestinationInstance() &&
        this.$store.getters.getAppDestinationInstance().isDefaultDestination()
      )
    },
  },

  mounted() {
    this.fetchUserData()
  },

  methods: {
    // change the password
    changePassword() {
      // check first the password constraints
      if (this.$refs.passwordForm.validate()) {
        // send new pw to api
        /* global EventBus axios */
        EventBus.$emit('spinnerShow')

        shopInstance()
          .patch(
            this.$store.getters.getCurrentDestinationInstance().getShopApi() +
              'user/password',
            {
              password: this.password1,
            }
          )
          .then(() => {
            EventBus.$emit(
              'notify',
              this.$root.$t('notify.passwordChangedSuccessfully'),
              'success'
            )
          })
          .catch(() => {
            EventBus.$emit('notify')
          })
          .finally(() => {
            EventBus.$emit('spinnerHide')
          })
      }
    },

    // delete my profile
    async deactivateMyProfile() {
      if (!this.$refs.deactivateModalForm.validate()) return
      if (!this.appUser.deactivateMyProfile()) return
      // close modal
      this.$refs.deactivateProfile.hide()
      // logout
      EventBus.$emit('logout')
    },

    checkMail() {
      if (this.$refs.newMailForm.validate()) {
        if (this.newMail1 === this.newMail2) {
          this.saveMail()
        } else {
          /* global EventBus */
          EventBus.$emit(
            'notify',
            this.$t('notify.mailsHaveToBeEquals'),
            'error'
          )
        }
      }
    },

    // fetch user data from store
    fetchUserData() {
      let user = this.$store.getters.getAppUserInstance()
      this.firstName = user.getFirstName()
      this.lastName = user.getLastName()
      this.birthday = user.getBirthdate()
      this.address = user.getAddress()
      this.zip =
        user.getZip() === 0
          ? null
          : this.$store.getters.getAppUserInstance().getZip()
      this.city = user.getCity()
      this.country = user.getCountry()
      this.phone = user.getPhone()
      this.perfectSkiDay = user.getSkiDay()
      this.favoriteSkiRessort = user.getSkiDestination()
      this.favoriteSkiMask = user.getSkiMask()
      this.education = user.getEducation()
      this.profession = user.getProfession()
      this.salutations = user.salutations
    },

    // save profil informations
    saveProfile() {
      if (this.$refs.form.validate()) {
        /* global EventBus axios */
        EventBus.$emit('spinnerShow')

        shopInstance()
          .patch(
            this.$store.getters.getCurrentDestinationInstance().getShopApi() +
              'user/' +
              this.$store.getters.getAppUserInstance().getId() +
              '/profileInfo',
            {
              firstname: this.firstName === '' ? null : this.firstName,
              surname: this.lastName === '' ? null : this.lastName,
              birthdate: this.birthday,
              street: this.address === '' ? null : this.address,
              zip: this.zip === null ? 0 : this.zip,
              city: this.city === '' ? null : this.city,
              country: this.country === '' ? null : this.country,
              phone: this.phone === '' ? null : this.phone,
              language: this.$store.getters.getActualLanguage(false),
              skiDay: this.perfectSkiDay === '' ? null : this.perfectSkiDay,
              skiDestination:
                this.favoriteSkiRessort === '' ? null : this.favoriteSkiRessort,
              skiMask:
                this.favoriteSkiMask === '' ? null : this.favoriteSkiMask,
              education: this.education === '' ? null : this.education,
              profession: this.profession === '' ? null : this.profession,
              title: this.salutations.salutation.salutationKey,
            }
          )
          .then(() => {
            this.$store.dispatch('getMainUserFromAPI')
            EventBus.$emit(
              'notify',
              this.$root.$t('notify.profileInfoSaved'),
              'success'
            )
          })
          .catch(() => {
            EventBus.$emit('notify')
          })
          .finally(() => {
            EventBus.$emit('spinnerHide')
          })
      }
    },

    saveMail() {
      /* global EventBus axios */
      EventBus.$emit('spinnerShow')

      shopInstance
        .patch(
          this.$store.getters.getCurrentDestinationInstance().getShopApi() +
            'user/mailAddress',
          {
            mail: this.newMail1,
            uid: this.$store.getters.getAppUserInstance().getId(),
          }
        )
        .then(() => {
          // get user data to store
          // todo: refactor
          this.$store.dispatch('getMainUserFromAPI')
          this.$refs.mailModal.hide()
        })
        .catch(() => {
          EventBus.$emit(
            'notify',
            this.$root.$t('notify.mailNotSaved'),
            'error'
          )
        })
        .finally(() => {
          EventBus.$emit('spinnerHide')
        })
    },
  },
}
</script>
