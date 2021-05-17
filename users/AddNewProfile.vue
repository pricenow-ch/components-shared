<template>
  <!-- modal with form -->
  <s-modal
    ref="addNewProfile"
    :size="1"
    :hideOnOk="false"
    :header-text="headerText"
    :ok-text="$t('general.save')"
    @hide="$emit('hide')"
    @cancel="$emit('cancel')"
    @ok="profileAction()"
  >
    <v-form ref="newProfile" @submit.prevent="">
      <v-row class="no-gutters ma-3">
        <!-- optional: company-->
        <v-col v-if="fields.company || fields.company === ''" class="col-12">
          <v-text-field
            v-model="company"
            outlined
            :rules="[
              (v) =>
                !mandatoryField('company') ||
                !!v ||
                this.$t('registration.companyRequired'),
            ]"
            :label="
              mandatoryField('company')
                ? $t('registration.companyStar')
                : $t('registration.company')
            "
          />
        </v-col>

        <!-- optional: salutation -->
        <v-col
          v-if="fields.salutations"
          class="col-12"
          :set="(salutations = fields.salutations)"
        >
          <v-radio-group
            v-model="salutations.salutation"
            row
            mandatory
            :rules="[
              (v) =>
                !mandatoryField('salutations') ||
                !!v ||
                this.$t('registrationForm.selectSalutation'),
            ]"
            :label="
              mandatoryField('salutations')
                ? $t('registrationForm.salutationStar')
                : $t('registrationForm.salutation')
            "
          >
            <v-radio
              v-for="(salutation, salutationIndex) in salutations.salutations"
              :key="salutationIndex"
              :label="salutation.getTitle()"
              :value="salutation"
            />
          </v-radio-group>
        </v-col>

        <!-- first name -->
        <v-col class="col-12">
          <v-text-field
            ref="firstName"
            v-model="fields.firstName"
            outlined
            validate-on-blur
            :rules="[
              (v) =>
                !mandatoryField('firstName') ||
                !!v ||
                this.$t('registration.firstNameRequired'),
            ]"
            :label="
              mandatoryField('firstName')
                ? $t('registration.firstName')
                : $t('general.firstName')
            "
          />
        </v-col>

        <!-- last name -->
        <v-col class="col-12">
          <v-text-field
            v-model="fields.lastName"
            outlined
            validate-on-blur
            :rules="[
              (v) =>
                !mandatoryField('lastName') ||
                !!v ||
                this.$t('registration.lastNameRequired'),
            ]"
            :label="
              mandatoryField('lastName')
                ? $t('registration.lastName')
                : $t('general.lastName')
            "
            required
          />
        </v-col>

        <!-- optional: address-->
        <v-col v-if="fields.address || fields.address === ''" class="col-12">
          <v-text-field
            v-model="fields.address"
            outlined
            :rules="[
              (v) =>
                !mandatoryField('address') ||
                !!v ||
                this.$t('registration.addressRequired'),
            ]"
            :label="
              mandatoryField('address')
                ? $t('registration.streetStar')
                : $t('general.address')
            "
          />
        </v-col>

        <!-- optional: zip and city-->
        <!-- zip -->
        <v-col
          v-if="fields.zip || fields.zip === ''"
          class="col-12 col-md-4 pr-md-1"
        >
          <v-text-field
            v-model="fields.zip"
            outlined
            :rules="[
              (v) =>
                !mandatoryField('zip') ||
                !!v ||
                this.$t('registration.zipRequired'),
            ]"
            :label="
              mandatoryField('zip') ? $t('registration.zip') : $t('general.zip')
            "
          />
        </v-col>
        <!-- optional: city -->
        <v-col
          v-if="fields.city || fields.city === ''"
          class="col-12 col-md-8 pl-md-1"
        >
          <v-text-field
            v-model="fields.city"
            outlined
            :rules="[
              (v) =>
                !mandatoryField('city') ||
                !!v ||
                this.$t('registration.cityRequired'),
            ]"
            :label="
              mandatoryField('city')
                ? $t('registration.cityStar')
                : $t('general.city')
            "
          />
        </v-col>

        <!-- optional: country-->
        <v-col v-if="fields.country || fields.country === ''" class="col-12">
          <s-country
            ref="country"
            outlined
            :required="mandatoryField('country')"
            :country.sync="fields.country"
          />
        </v-col>

        <!-- optional: phone -->
        <v-col v-if="fields.phone || fields.phone === ''" class="col-12">
          <v-text-field
            v-model="fields.phone"
            outlined
            :rules="[
              (v) =>
                !mandatoryField('phone') ||
                !!v ||
                this.$t('registration.phoneRequired'),
            ]"
            :label="
              mandatoryField('phone')
                ? $t('registration.phoneStar')
                : $t('general.phone')
            "
            type="tel"
          />
        </v-col>

        <!-- email -->
        <v-col v-show="fields.mail || fields.mail === ''" class="col-12">
          <v-text-field
            v-model="fields.mail"
            outlined
            :rules="[
              (v) =>
                !mandatoryField('mail') ||
                !!v ||
                this.$t('registration.eMailRequired'),
            ]"
            :label="
              mandatoryField('mail')
                ? $t('registration.email')
                : $t('general.email')
            "
            validate-on-blur
          />
        </v-col>

        <!-- birthday -->
        <v-col class="col-12">
          <s-birthday-picker
            :date.sync="fields.birthday"
            outlined
            :mandatory="mandatoryFields.includes('birthday')"
          />
        </v-col>
      </v-row>

      <!-- cards (only in edit mode)-->
      <v-row v-show="edit && showCardList" class="pt-4">
        <v-col class="col-12 py-0 text-center">
          <div class="headline">
            {{ $t('addNewProfile.cards') }}
          </div>
        </v-col>
      </v-row>

      <!-- cards list -->
      <v-row v-show="edit && showCardList" class="justify-center">
        <v-col class="col-12 py-0">
          <cards-list
            v-if="profile"
            ref="cardsList"
            :cards="profile.getCards()"
            :uid="uid"
            :available-card-types="availableCardTypes"
            hideHeaders
            @cancelNewCard="cancelNewCard()"
            @addNewProfile:show="addNewProfileShow()"
            @openAddCardModal="$refs.addNewProfile.hide()"
            @refresh="
              (uid) => {
                $emit('refresh', uid)
              }
            "
          />
        </v-col>
      </v-row>
    </v-form>

    <v-row v-if="!edit && !hideCardListSwitch" class="pt-10 mb-0">
      <v-col class="col-12 py-0">
        <v-switch
          :value="showCardList"
          color="primary"
          :label="showCardList ? $t('v6.addNoCard') : $t('v6.addCard')"
          @change="
            () => {
              this.$emit('update:showCardList', !showCardList)
            }
          "
        />
      </v-col>
    </v-row>

    <!-- card adder (not in edit mode: only while adding new profile)-->
    <v-row v-show="!edit && showCardList">
      <v-col class="col-12 py-0">
        <card-adder
          ref="cardAdder"
          :availableCardTypes="availableCardTypes"
          :hideAgb="hideAgb"
          :hide-camera="hideCamera"
          :card="card"
          :disable-swisspass="disableSwisspass"
          @hideModal="$refs.addNewProfile.hide()"
          @showModal="$refs.addNewProfile.show()"
        />
      </v-col>
    </v-row>
  </s-modal>
</template>

<script>
import CardAdder from '../cards/CardAdder.vue'
import CardsList from '../cards/CardsList.vue'
import Card from '../../classes/card/Card'
import SBirthdayPicker from '../utils/SBirthdayPicker'
import SCountry from '../utils/SCountry'
import User from '../../classes/user/User'
import definitions from '../../../definitions'
import { shopInstance } from '../../classes/utils/axiosInstance'

export default {
  name: 'AddNewProfile',

  components: {
    CardAdder,
    CardsList,
    SBirthdayPicker,
    SCountry,
  },

  props: {
    // existing profile?
    edit: {
      required: false,
      type: Boolean,
      default: false,
    },

    // user class instance
    profile: {
      required: false,
      type: User,
      default: null,
    },

    // Array with available media types which can be added
    availableCardTypes: {
      type: Array,
      required: false,
      default: () => {
        return Object.keys(definitions.ticketMedia)
      },
    },

    card: {
      required: false,
      type: Card,
      default: () => {
        return new Card()
      },
    },

    // adds additional form fields
    additionalFields: {
      required: false,
      type: Object,
      default: () => {
        return {
          address: null,
          city: null,
          zip: null,
          country: null,
          mail: null,
          phone: null,
          company: null,
          salutations: null,
        }
      },
    },

    // make input fields mandatory
    mandatoryFields: {
      type: Array,
      required: false,
      default: () => {
        return ['firstName', 'lastName', 'birthday']
      },
    },

    // custom header text
    headerText: {
      required: false,
      type: String,
      default: '',
    },

    // hides agb check field
    hideAgb: {
      required: false,
      type: Boolean,
      default: false,
    },

    hideCamera: {
      required: false,
      type: Boolean,
      default: false,
    },

    // show or hide card list
    showCardList: {
      required: false,
      type: Boolean,
      default: true,
    },

    // hide card list switch
    hideCardListSwitch: {
      requried: false,
      type: Boolean,
      default: false,
    },

    // make swisspass uneditable
    disableSwisspass: {
      type: Boolean,
      required: false,
      default: false,
    },

    // create new user as shadow user or not
    shadow: {
      type: Boolean,
      required: false,
      default: true,
    },

    // create the shadow user for another uid
    // needs a particular permission and changes the endpoint
    foreignUidForShadowUser: {
      type: Number,
      required: false,
      default: null,
    },

    // reset fields after adding a new user
    resetFields: {
      type: Boolean,
      required: false,
      default: false,
    },

    editOtherUsersProfile: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  data() {
    return {
      fields: {
        firstName: null,
        lastName: null,
        birthday: null,
        mail: null,
        oldMail: null,
        address: this.additionalFields.address,
        city: this.additionalFields.city,
        zip: this.additionalFields.zip,
        country: this.additionalFields.country,
        phone: this.additionalFields.phone,
        company: this.additionalFields.company,
        salutations: this.additionalFields.salutations,
      },
      skiCards: [],
    }
  },

  computed: {
    uid() {
      // if a profile (= User instance) passed as prop take it. Else fallback to the AppUser instance
      return this.profile
        ? this.profile.getId()
        : this.$store.getters.getAppUserInstance().getId()
    },
  },

  watch: {
    profile(newProfile) {
      this.updateProfileDataForForm(newProfile)
    },
  },

  mounted() {
    if (this.profile != null) this.updateProfileDataForForm(this.profile)
  },

  methods: {
    updateProfileDataForForm(newProfile) {
      if (newProfile) {
        this.fields.firstName = newProfile.getFirstName()
        this.fields.lastName = newProfile.getLastName()
        this.fields.birthday = newProfile.getBirthdate()
        this.fields.mail = newProfile.getMail()

        // hold old mail
        this.fields.oldMail = newProfile.getMail()
      }
    },
    addNewProfileShow() {
      if (this.edit) {
        this.$refs.addNewProfile.show()
      }
    },

    // check if this field is mandatory
    // short cut
    mandatoryField(field) {
      return this.mandatoryFields.find((tmpField) => tmpField === field)
    },

    // decide whether profile is to patch or to post
    profileAction() {
      if (this.edit) {
        this.updateProfile()
      } else {
        this.addNewProfile()
      }
    },

    // update user profile
    async updateProfile() {
      /* global EventBus axios */
      EventBus.$emit('spinnerShow')
      let data = {
        firstname: this.fields.firstName,
        surname: this.fields.lastName,
      }

      if (this.fields.birthday) data.birthdate = this.fields.birthday
      if (this.fields.company) data.company = this.fields.company
      if (this.fields.address) data.street = this.fields.address
      if (this.fields.city) data.city = this.fields.city
      if (this.fields.zip) data.zip = this.fields.zip
      if (this.fields.country) data.country = this.fields.country
      if (this.fields.company) data.company = this.fields.company
      if (this.fields.phone) data.phone = this.fields.phone

      try {
        await shopInstance().patch(
          `${
            this.editOtherUsersProfile ? '/admin' : ''
          }/user/${this.profile.getId()}/profileInfo`,
          data
        )

        if (this.oldMail === this.mail) {
          // update email, if there is a new one
          this.afterUserUpdate()
        } else {
          await this.updateMail()
        }
      } catch (e) {
        EventBus.$emit('notify')
      } finally {
        EventBus.$emit('spinnerHide')
      }
    },

    afterUserUpdate() {
      //refresh user profiles
      this.$emit(
        'refresh',
        new User({
          uid: this.profile.uid,
          firstName: this.fields.firstName,
          lastName: this.fields.lastName,
          birthdate: this.fields.birthdate,
          mail: this.fields.mail,
          phone: this.fields.phone,
          street: this.fields.address,
          address: this.fields.address,
          city: this.fields.city,
          zip: this.fields.zip,
          country: this.fields.country,
          skiCards: this.skiCards,
          company: this.fields.company,
          title: this.fields.salutations
            ? this.fields.salutation.salutationKey
            : null,
        })
      )

      // hide modal
      this.$refs.addNewProfile.hide()

      // inform user
      EventBus.$emit('notify', this.$t('addNewProfile.userUpdated'), 'success')
    },

    async updateMail() {
      /* global EventBus axios */
      EventBus.$emit('spinnerShow')

      let url = this.$store.getters.getCurrentDestinationInstance().getShopApi()
      if (this.editOtherUsersProfile) url += 'admin/'
      url += 'user/mailAddress'

      try {
        await axios.patch(url, {
          mail: this.fields.mail,
          uid: this.uid,
        })

        this.afterUserUpdate()
      } catch (e) {
        EventBus.$emit('notify')
      } finally {
        EventBus.$emit('spinnerHide')
      }
    },

    // close add new card modal and reopen add new profile modal
    cancelNewCard() {
      this.$refs.addNewProfile.show()
    },

    // open modal
    show() {
      this.$refs.addNewProfile.show()
    },

    // add new profile via api
    addNewProfile() {
      if (
        this.$refs.cardAdder.validateIfUserWantsToAddCard() &&
        this.$refs.newProfile.validate()
      ) {
        /* global EventBus axios */
        EventBus.$emit('spinnerShow')

        let url = this.$store.getters
          .getCurrentDestinationInstance()
          .getShopApi()
        url = this.foreignUidForShadowUser ? url + 'admin/user' : url + 'user'
        // prepare zip
        let zip =
          this.$refs.cardAdder?.getCard()?.getZip() || this.fields?.zip || null

        let payload = {
          firstname: this.fields.firstName,
          surname: this.fields.lastName,
          birthdate: this.fields.birthday || null,
          street: this.fields.address ? this.fields.address : '',
          zip: zip,
          city: this.fields.city ? this.fields.city : '',
          mail: this.fields.mail,
          shadow: this.shadow,
          country: this.fields.country || '',
          phone: this.fields.phone || '',
          agbAccepted: false,
          company: this.fields.company ? this.fields.company : '',
          title: this.fields.salutations
            ? this.salutations.salutation.salutationKey
            : null,
        }

        if (this.foreignUidForShadowUser && this.shadow) {
          payload.userId = this.foreignUidForShadowUser
        } else {
          payload.cardNumber = this.$refs.cardAdder.getCard().getCardNumber()
          payload.cardType = parseInt(this.$refs.cardAdder.getCard().getType())
          payload.cardDescription = this.$refs.cardAdder
            .getCard()
            .getCard().cardDescription
        }

        axios
          .post(url, payload)
          .then((response) => {
            this.$emit('refresh', response.data)

            // notify user about success
            EventBus.$emit(
              'notify',
              this.$root.$t('addNewProfile.newProfileAddedSuccessfully'),
              'success'
            )

            // hide modal
            this.$refs.addNewProfile.hide()

            // reset fields to have a clean form for adding next user
            if (this.resetFields) {
              this.fields.firstName = null
              this.fields.lastName = null
              this.fields.birthday = null
              this.fields.mail = null
              this.fields.country = null
              this.fields.address = null
              this.fields.city = null
              this.fields.zip = null
              this.fields.country = null
              this.fields.phone = null
              this.fields.company = null
            }
          })
          .catch((error) => {
            if (error.response.status === 409) {
              let user = new User(error.response.data.error)
              let userString = user.getFullName()
              if (user.getBirthdate()) userString + ', ' + user.getBirthdate()
              if (user.getCity())
                userString +
                  ' ' +
                  this.$root.$t('shopBackendBookingDetail.fromCity') +
                  ' ' +
                  user.getCity()
              EventBus.$emit(
                'notify',
                this.$root.$t('addNewProfile.mailAlreadyExists', {
                  user: userString,
                })
              )
            } else if (error.response.status === 410) {
              // person was linked (admin endpoint)
              this.$refs.addNewProfile.hide()
              EventBus.$emit(
                'notify',
                this.$root.$t('addNewProfile.userLinkedToRegion')
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
