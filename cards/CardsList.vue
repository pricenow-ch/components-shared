<template>
  <div>
    <v-data-table
      :headers="headers"
      :items="cards"
      :no-data-text="$t('cardsList.noCardsAvailable')"
      :hide-default-header="hideHeaders"
      hide-default-footer
    >
      <template #body="{ items }">
        <tbody>
          <!-- iterate cards -->
          <tr v-for="item in items" :key="item.getCard().id">
            <!-- delete button -->
            <td class="pr-0 pl-0">
              <span
                class="fa fa-trash error--text clickable"
                @click="deleteCard(item.getCard().id)"
              >
              </span>
            </td>

            <!-- card type -->
            <td class="pr-1 pl-1">
              {{ item.getCardTypeDescription() }}
            </td>
            <td class="pr-1">
              {{ item.getCardDescription() }}
            </td>
          </tr>
        </tbody>
      </template>
    </v-data-table>

    <v-row class="justify-center pb-4 pt-2">
      <v-col class="col-12 text-center">
        <v-btn
          outlined
          large
          color="primary"
          class="ml-0"
          @click="openAddCardModal()"
        >
          <v-icon class="mr-2" small>fa-plus-circle</v-icon>
          {{ $t('cardsList.addCard') }}
        </v-btn>
      </v-col>
    </v-row>

    <!-- add new card in edit mode in a separate modal -->
    <s-modal
      ref="newCardModal"
      :size="1"
      :headerText="$t('addNewProfile.newCardModalTitle')"
      :hideOnOk="false"
      @ok="addNewCard()"
      @cancel="$emit('cancelNewCard')"
    >
      <v-row>
        <v-col class="col-12">
          <card-adder
            ref="cardAdderEdit"
            :card="card"
            :available-card-types="availableCardTypes"
            @hideModal="$refs.newCardModal.hide()"
            @showModal="$refs.newCardModal.show()"
          />
        </v-col>
      </v-row>
    </s-modal>
  </div>
</template>

<script>
import CardAdder from './CardAdder.vue'
import Card from '../../classes-shared/card/Card.js'
import definitions from '../../../definitions'
import CardService from '@/classes-shared/card/CardService'
import { shopInstance } from '../../classes-shared/utils/axiosInstance'

export default {
  name: 'CardsList',

  components: {
    CardAdder,
  },

  props: {
    cards: {
      required: true,
      type: Array,
      default: () => {
        return []
      },
    },

    uid: {
      required: true,
      type: String,
    },

    // Array with available media types which can be added
    availableCardTypes: {
      type: Array,
      required: false,
      default: () => {
        return Object.keys(definitions.ticketMedia)
      },
    },

    hideHeaders: {
      required: false,
      type: Boolean,
      default: false,
    },

    // if component is used to add new user
    newUserMode: {
      required: false,
      type: Boolean,
      default: false,
    },
  },

  data() {
    return {
      card: new Card(),
      cardService: new CardService(),
    }
  },

  computed: {
    headers() {
      return [
        {
          text: '',
          sortable: false,
        },
        {
          text: this.$t('profile.cardType'),
          align: 'left',
          sortable: true,
          value: 'type',
        },
        {
          text: this.$t('cardsList.description'),
          align: 'left',
          sortable: false,
        },
      ]
    },
  },

  methods: {
    validate() {
      return (
        (this.newUserMode &&
          this.$refs.cardAdderEdit.validateIfUserWantsToAddCard()) ||
        (!this.newUserMode && this.$refs.cardAdderEdit.validate())
      )
    },

    // add a new card to the profile
    async addNewCard(uid = this.uid, card = this.card) {
      // check if the card fields are filled in correctly
      if (this.validate()) {
        if (await this.cardService.addCardToUser(card, uid)) {
          this.$emit('refresh', uid)
          this.$emit('addNewProfile:show')
          this.$refs.newCardModal.hide()
        }

        EventBus.$emit('cardList:cardAdded')
      }
    },

    openAddCardModal() {
      // emit action to get the possibility to catch it
      this.$emit('openAddCardModal')

      // open modal
      this.$refs.newCardModal.show()
    },

    // delete ski card from user
    deleteCard(cardId) {
      /* global EventBus axios */
      EventBus.$emit('spinnerShow')

      shopInstance()
        .delete(`/skicard/${this.uid}/${cardId}`)
        .then(() => {
          this.$emit('refresh')
          EventBus.$emit(
            'notify',
            this.$root.$t('addNewProfile.cardDelted'),
            'success'
          )
        })
        .catch(() => {
          EventBus.$emit('notify')
        })
        .finally(() => {
          EventBus.$emit('spinnerHide')
        })
    },
  },
}
</script>
