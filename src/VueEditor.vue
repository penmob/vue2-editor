<template>
  <div id="quillWrapper">

      <div ref="quillContainer" id="quill-container"></div>

      <button v-if="useSaveButton" class="save-button"
        @click="saveContent">
        {{ buttonText ? buttonText : 'Save Content' }}
      </button>

      <div v-if="showPreview" ref="livePreview" class="ql-editor"></div>

  </div>
</template>

<script>
import Quill from 'quill'
require('../node_modules/quill/dist/quill.core.css')
require('../node_modules/quill/dist/quill.snow.css')

var defaultToolbar = [
  ['bold', 'italic', 'underline', 'strike'],
  ['blockquote', 'code-block'],

  [{ 'list': 'ordered'}, { 'list': 'bullet' }],

  [{ 'indent': '-1'}, { 'indent': '+1' }],
  [{ 'header': [1, 2, 3, 4, 5, 6, false] }],

  [{ 'color': [] }, { 'background': [] }],
  [{ 'font': [] }],
  [{ 'align': [] }],

  ['clean']
]

export default {
  name: 'vue-editor',
  props: {
    editorContent: String,
    placeholder: String,
    buttonText: String,
    editorToolbar: Array,
    useSaveButton: {
      type: Boolean,
      default () {
        return true
      }
    },
    showPreview: {
      type: Boolean,
      default () {
        return false
      }
    },
    readOnly: {
      type: Boolean,
      default () {
          return false
      }
    },
    theme: String,
    addTempHighlight: {
      type: Object,
      default () {
        return {
          rangeIndex: null,
          rangeLength: null,
        }
      }
    },
    removeTempHighlight: {
      type: Boolean,
      default () {
        return false
      }
    },
    addSuggestionHighlight: {
      type: Array,
      default () {
        return [{
          rangeIndex: null,
          rangeLength: null,
        }]
      }
    },
  },

  data: function () {
    return {
      quill: null,
      editor: null,
      toolbar: this.editorToolbar ? this.editorToolbar : defaultToolbar
    }
  },

  mounted: function () {
    const vm = this

    vm.quill = new Quill(vm.$refs.quillContainer, {
      modules: {
        toolbar: this.toolbar
      },
      placeholder: this.placeholder ? this.placeholder : '',
      readOnly: this.readOnly,
      theme: this.theme ? this.theme : 'snow'
    });

    vm.editor = document.querySelector('.ql-editor')

    if ( vm.$refs.livePreview !== undefined || false ) {

      vm.quill.on('text-change', function() {
        vm.$refs.livePreview.innerHTML = vm.editor.innerHTML
        vm.$emit('editor-updated', vm.editor.innerHTML)
      });

    } else {

      vm.quill.on('text-change', function() {
        vm.$emit('editor-updated', vm.editor.innerHTML)
      });

    }

    // register new highlight formats.
    let Inline = Quill.import('blots/inline');

    class tempHighlight extends Inline {
        static create() {
            return super.create();
        }
        static formats() {
            return true;
        }
    }
    tempHighlight.blotName = 'tempHighlight';
    tempHighlight.className = 'temp-highlight';
    tempHighlight.tagName = 'span';

    class suggestionHighlight extends Inline {
        static create() {
            return super.create();
        }
        static formats() {
            return true;
        }
    }
    suggestionHighlight.blotName = 'suggestionHighlight';
    suggestionHighlight.className = 'suggestion-highlight';
    suggestionHighlight.tagName = 'span';

    Quill.register(tempHighlight);
    Quill.register(suggestionHighlight);
  },

  watch: {
    editorContent: function () {
      this.editor.innerHTML = this.editorContent
    },
    readOnly: function () {
      this.readOnly ? this.quill.disable() : this.quill.enable();
    },
    addTempHighlight: function () {
      if (this.addTempHighlight.rangeIndex !== null && this.addTempHighlight.rangeLength !== null) {
        this.quill.formatText(this.addTempHighlight.rangeIndex, this.addTempHighlight.rangeLength, 'tempHighlight', true);
      }
    },
    removeTempHighlight: function () {
      if (this.removeTempHighlight) {
        this.quill.formatText(0, this.quill.getLength(), 'tempHighlight', false);
        this.$emit('tempHighlightRemoved');
      }
    },
    addSuggestionHighlight: function () {
      if (this.addSuggestionHighlight.length > 0) {
        for (let i = 0; i < this.addSuggestionHighlight.length; i++) {
          if (this.addSuggestionHighlight[i].rangeIndex !== null && this.addSuggestionHighlight[i].rangeLength !== null) {
            this.quill.formatText(this.addSuggestionHighlight[i].rangeIndex, this.addSuggestionHighlight[i].rangeLength, 'suggestionHighlight', true);
          }
        }
      }
    },
  },

  methods: {
    saveContent: function (value) {
      this.$emit('save-content', this.editor.innerHTML)
    },
  },

  events: {
    'removeTempHighlights': function () {
      console.log('in removeTempHighlights vue2-editor');
      console.log(this);
      console.log(this.quill);
      console.log(this.quill.getContents());

      this.quill.formatText(0, this.quill.getLength(), 'tempHighlight', false);

      console.log('removed:');
      console.log(this.quill.getContents());
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

#quill-container {
  height: 400px;
}

</style>
