<template>
  <v-dialog
    :value="value"
    width="50%"
    overlay-color="white"
    :overlay-opacity="0.9"
    @input="
      (value) => {
        this.$('update:value', value)
      }
    "
  >
    <v-card>
      <v-toolbar class="mb-4 d-flex justify-space-between" flat>
        <v-btn icon @click="close">
          <i class="far fa-2x fa-times-circle"></i>
        </v-btn>
        <v-toolbar-title>
          <span v-if="mode === 'addToList'">
            {{ productDefinition.getTitle() }}
            {{ $t('bookmarks.addListTitle') }}
          </span>
          <span v-if="mode === 'editLists'">
            {{ $t('bookmarks.editListsTitle') }}
          </span>
        </v-toolbar-title>
      </v-toolbar>

      <v-card-text>
        <v-row v-if="mode === 'addToList' || mode === 'editLists'">
          <v-col cols="12">
            <v-expansion-panels
              v-model="panel"
              flat
              :readonly="mode === 'addToList'"
            >
              <v-expansion-panel
                v-for="(bookmark, key) in bookmarks"
                :key="key"
                :disabled="containsProdDef(key) && mode !== 'editLists'"
              >
                <v-expansion-panel-header @click="addProdDefToList(key)">
                  <span>
                    {{ bookmark.name }}
                  </span>
                  <v-btn
                    v-if="mode === 'editLists'"
                    class="mr-8"
                    color="error"
                    max-width="1rem"
                    x-small
                    icon
                    @click="removeList(key)"
                  >
                    <i class="far fa-times-circle"></i>
                  </v-btn>
                  <template v-if="mode === 'addToList'" #actions>
                    <v-badge
                      v-if="bookmark.prodDefIds.length"
                      color="accent"
                      :content="bookmark.prodDefIds.length || 0"
                    >
                    </v-badge>
                    <v-badge v-else color="warning" :content="'0'"> </v-badge>
                  </template>
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                  <v-list v-if="bookmark.prodDefIds.length > 0">
                    <v-list-item
                      v-for="(id, prodDefKey) in bookmark.prodDefIds"
                      :key="id"
                    >
                      <v-list-item-content>
                        {{ prodDefTranslation(id) }}
                      </v-list-item-content>

                      <v-list-item-action>
                        <v-btn
                          icon
                          color="error"
                          @click="removeProdDef(key, prodDefKey)"
                        >
                          <i class="far fa-times-circle"></i>
                        </v-btn>
                      </v-list-item-action>
                    </v-list-item>
                  </v-list>
                  <v-alert v-else type="warning">
                    {{ $t('bookmarks.noProdDefs') }}
                  </v-alert>
                </v-expansion-panel-content>
              </v-expansion-panel>
            </v-expansion-panels>
          </v-col>
          <v-col v-if="bookmarks.length === 0" cols="12">
            <v-alert type="warning">
              {{ $t('bookmarks.noLists') }}
            </v-alert>
          </v-col>
        </v-row>

        <v-row v-if="mode === 'create'">
          <v-col cols="12">
            <v-form ref="form" v-model="createFormValid">
              <v-text-field
                v-model="newListName"
                :label="$t('bookmarks.listName')"
                outlined
                :rules="[
                  createListRules.required,
                  createListRules.unique,
                  createListRules.maxLength,
                ]"
              ></v-text-field>
            </v-form>
          </v-col>
        </v-row>
      </v-card-text>

      <v-divider></v-divider>

      <v-card-actions class="d-flex justify-center">
        <v-btn
          v-if="mode === 'addToList'"
          color="primary"
          outlined
          @click="mode = 'create'"
        >
          <i class="far fa-plus-square mr-2"></i>
          {{ $t('bookmarks.newList') }}
        </v-btn>
        <v-btn
          v-if="mode === 'create'"
          color="primary"
          outlined
          @click="mode = 'addToList'"
        >
          {{ $t('bookmarks.abort') }}
        </v-btn>
        <v-btn
          v-if="mode === 'create'"
          color="success"
          outlined
          :disabled="!createFormValid"
          @click="createList"
        >
          {{ $t('bookmarks.create') }}
        </v-btn>
        <v-btn
          v-if="mode === 'addToList'"
          color="info"
          outlined
          @click="mode = 'editLists'"
        >
          <i class="fas fa-pen mr-2"></i>
          {{ $t('bookmarks.editLists') }}
        </v-btn>
        <v-btn v-if="mode === 'editLists'" color="info" outlined @click="back">
          <i class="fas fa-chevron-left mr-2"></i>
          {{ $t('bookmarks.back') }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import ProductDefinition from '@/classes-shared/products/ProductDefinition'
import { mapState } from 'vuex'
import BookmarkList from '@/classes-shared/bookmark/BookmarkList'
import _ from 'lodash'
export default {
  name: 'ProductDefinitionBookmarks',

  props: {
    /**
     * the productdefinition object
     * which should be added to
     * favorites list
     */
    productDefinition: {
      type: ProductDefinition,
      required: true,
    },
    /**
     * All prod defs to fetch their
     * translations
     */
    productDefinitions: {
      type: Array,
      required: true,
    },
    /**
     * v-model to open/close the
     * dialog
     */
    value: {
      type: Boolean,
      required: true,
    },
  },

  data() {
    return {
      mode: 'addToList',
      newListName: null,
      createFormValid: true,
      panel: [],
      createListRules: {
        required: (value) => !!value || this.$t('bookmarks.required'),
        maxLength: (value) =>
          value?.length <= 50 || this.$t('bookmarks.maxChars'),
        unique: () => {
          const index = _.findIndex(this.bookmarks, (bookmark) => {
            return bookmark.name === this.newListName
          })
          if (index === -1) {
            return true
          }

          return this.$t('bookmarks.notUnique')
        },
      },
    }
  },

  computed: {
    ...mapState('UserProductDefinitionBookmarks', ['bookmarks']),
  },

  mounted() {
    if (Object.keys(this.bookmarks).length === 0) {
      this.$store.dispatch('UserProductDefinitionBookmarks/loadBookmarks')
    }
  },

  methods: {
    close() {
      this.$emit('input', false)
    },
    back() {
      this.mode = 'addToList'
      this.panel = []
    },
    async createList() {
      const bookmarks = this.bookmarks
      bookmarks.push(new BookmarkList({ name: this.newListName }))
      this.$store.commit(
        'UserProductDefinitionBookmarks/setBookmarks',
        bookmarks
      )
      await this.$store.dispatch(
        'UserProductDefinitionBookmarks/updateBookmarks'
      )
      this.newListName = null
      this.mode = 'addToList'
      this.$emit('changed', this.bookmarks)
    },
    async addProdDefToList(listIndex) {
      if (this.mode !== 'addToList') return
      if (this.containsProdDef(listIndex)) return
      const bookmarks = this.bookmarks
      bookmarks[listIndex].prodDefIds.push(this.productDefinition.getId())
      this.$store.commit(
        'UserProductDefinitionBookmarks/setBookmarks',
        bookmarks
      )
      await this.$store.dispatch(
        'UserProductDefinitionBookmarks/updateBookmarks'
      )
      this.$emit('input', false)
      this.$emit('changed', this.bookmarks)
    },
    containsProdDef(bookmarkKey) {
      return this.bookmarks[bookmarkKey].prodDefIds.includes(
        this.productDefinition.getId()
      )
    },
    async removeList(listIndex) {
      let bookmarks = this.bookmarks
      bookmarks.splice(listIndex, 1)
      this.$store.commit(
        'UserProductDefinitionBookmarks/setBookmarks',
        bookmarks
      )
      await this.$store.dispatch(
        'UserProductDefinitionBookmarks/updateBookmarks'
      )
      this.$emit('changed', this.bookmarks)
    },
    async removeProdDef(bookmarkKey, prodDefKey) {
      let bookmarks = this.bookmarks
      bookmarks[bookmarkKey].prodDefIds.splice(prodDefKey, 1)
      this.$store.commit(
        'UserProductDefinitionBookmarks/setBookmarks',
        bookmarks
      )
      await this.$store.dispatch(
        'UserProductDefinitionBookmarks/updateBookmarks'
      )
    },
    prodDefTranslation(prodDefId) {
      const prodDef = _.find(this.productDefinitions, function (pd) {
        return pd.getId() === prodDefId
      })
      return (
        this.capitalize(prodDef.getProduct().destinations[0].slug) +
        ' - ' +
        prodDef.getProduct().getTitle() +
        ' - ' +
        prodDef.getTitle()
      )
    },
    capitalize(s) {
      if (typeof s !== 'string') return ''
      return s.charAt(0).toUpperCase() + s.slice(1)
    },
  },
}
</script>

<style scoped lang="scss">
>>> .v-dialog {
  border: none;
  .v-card {
    padding: 1rem;
  }
}
</style>
