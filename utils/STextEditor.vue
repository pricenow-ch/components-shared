<template>
  <div class="editor">
    <editor-menu-bar v-slot="{ commands, isActive }" :editor="editor">
      <div>
        <!-- history -->
        <v-btn icon class="mr-1" @click="commands.undo">
          <span class="fal fa-undo" />
        </v-btn>

        <v-btn icon class="mr-6" @click="commands.redo">
          <span class="fal fa-redo" />
        </v-btn>

        <!-- font -->
        <v-btn
          icon
          class="mr-1"
          :class="{ 'primary white--text': isActive.bold() }"
          @click="commands.bold"
        >
          <span class="fal fa-bold" />
        </v-btn>

        <v-btn
          icon
          class="mr-1"
          :class="{ 'primary white--text': isActive.italic() }"
          @click="commands.italic"
        >
          <span class="fal fa-italic" />
        </v-btn>

        <v-btn
          icon
          class="mr-1"
          :class="{ 'primary white--text': isActive.underline() }"
          @click="commands.underline"
        >
          <span class="fal fa-underline" />
        </v-btn>

        <v-btn
          icon
          class="mr-6"
          :class="{ 'primary white--text': isActive.strike() }"
          @click="commands.strike"
        >
          <span class="fal fa-strikethrough" />
        </v-btn>

        <!-- font size -->
        <v-btn
          icon
          class="mr-1"
          :class="{ 'primary white--text': isActive.heading({ level: 1 }) }"
          @click="commands.heading({ level: 1 })"
        >
          <span class="fal fa-h1" />
        </v-btn>

        <v-btn
          icon
          class="mr-1"
          :class="{ 'primary white--text': isActive.heading({ level: 2 }) }"
          @click="commands.heading({ level: 2 })"
        >
          <span class="fal fa-h2" />
        </v-btn>

        <v-btn
          icon
          class="mr-6"
          :class="{ 'primary white--text': isActive.heading({ level: 3 }) }"
          @click="commands.heading({ level: 3 })"
        >
          <span class="fal fa-h3" />
        </v-btn>

        <!-- list -->
        <v-btn
          icon
          class="mr-1"
          :class="{ 'primary white--text': isActive.bullet_list() }"
          @click="commands.bullet_list"
        >
          <span class="fal fa-list" />
        </v-btn>

        <v-btn
          icon
          class="mr-1"
          :class="{ 'primary white--text': isActive.ordered_list() }"
          @click="commands.ordered_list"
        >
          <span class="fal fa-list-ol" />
        </v-btn>
      </div>
    </editor-menu-bar>
    <editor-content
      class="border mt-4 pb-0"
      :class="{ 'validation-error': !validateState }"
      :editor="editor"
    />

    <v-row v-if="!validateState">
      <v-col class="red--text ml-2 content-5 font-weight-thin pt-1">
        {{ errorText }}
      </v-col>
    </v-row>
  </div>
</template>

<script>
import { Editor, EditorContent, EditorMenuBar } from 'tiptap'
import {
  Heading,
  OrderedList,
  BulletList,
  ListItem,
  TodoItem,
  TodoList,
  Bold,
  Italic,
  Link,
  Strike,
  Underline,
  History,
} from 'tiptap-extensions'
export default {
  components: {
    EditorContent,
    EditorMenuBar,
  },

  props: {
    // validation only works, if any error string is provided!
    errorText: {
      type: String,
      required: false,
      default: null,
    },
  },

  data() {
    return {
      // text editor content to set
      content: null,
      editor: new Editor({
        extensions: [
          new BulletList(),
          new Heading({ levels: [1, 2, 3] }),
          new ListItem(),
          new OrderedList(),
          new TodoItem(),
          new TodoList(),
          new Link(),
          new Bold(),
          new Italic(),
          new Strike(),
          new Underline(),
          new History(),
        ],
        onUpdate: ({ getHTML }) => {
          this.content = getHTML()
          this.$emit('update:content', this.content)
          this.$nextTick(() => {
            this.validate()
          })
        },
      }),

      // validate the text editor
      validateState: true,
    }
  },

  beforeDestroy() {
    this.editor.destroy()
  },

  methods: {
    // form validation
    // called from parent component
    validate() {
      if (this.errorText) {
        if (this.content && this.content !== '' && this.content !== '<p></p>') {
          this.validateState = true
          return true
        } else {
          this.validateState = false
          return false
        }
      }
      return true
    },

    setContent(content) {
      this.content = content
      this.editor.setContent(content)
    },

    getContent() {
      return this.content
    },
  },
}
</script>

<style lang="scss">
.ProseMirror {
  min-height: 200px;
  padding: 20px;
}

.validation-error {
  border-color: red;
  border-width: 3px;
  border-radius: 5px !important;
}
</style>
