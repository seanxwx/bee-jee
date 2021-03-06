<template>
  <div class="note-editor container-fluid py-3">
    <div v-if="note._id" class="row note-editor-container">
      <div class="col-12 form-group">
        <input
          type="text"
          class="title"
          placeholder="Title"
          @input="handleTitleUpdate"
          :value="note.title"
        />
      </div>
      <div class="col-12 form-group note-content-editor-container">
        <editor
          :initialValue="note.content"
          :options="editorOptions"
          height="100%"
          :initialEditType="getEditType(note)"
          ref="editor"
          previewStyle="tab"
          @change="handleContentUpdate"
        />
      </div>
    </div>
  </div>
</template>

<script>
import { Editor } from '@toast-ui/vue-editor';
import { toEditType, toContentType } from '../helpers/api';
import 'codemirror/lib/codemirror.css';
import '@toast-ui/editor/dist/toastui-editor.css';

export default {
  components: {
    Editor,
  },
  data() {
    return {
      editorOptions: {
        usageStatistics: false,
      },
    };
  },
  computed: {
    note() {
      return this.$store.getters.selectedNote;
    },
    defaultEditorEditType() {
      return this.$store.getters.config('defaultEditorEditType');
    },
  },
  methods: {
    getEditType(note) {
      return note.contentType ? toEditType(note.contentType) : this.defaultEditorEditType;
    },
    handleTitleUpdate(value) {
      this.$store.dispatch('changeNoteTitle', {
        _id: this.note._id,
        title: value,
      });
    },
    handleContentUpdate() {
      if (!this.note._id) {
        return;
      }
      const tuiEditor = this.$refs.editor;
      const contentType = toContentType(tuiEditor.editor.currentMode);
      let content = () => {
        return '';
      };
      switch (contentType) {
        case 'html':
          content = () => tuiEditor.invoke('getHtml');
          break;
        case 'markdown':
          content = () => tuiEditor.invoke('getMarkdown');
          break;
      }
      this.$store.dispatch('changeNoteContent', {
        _id: this.note._id,
        content,
        contentType,
      });
    },
  },
  created() {
    if (this.$route.params.id) {
      this.$store.dispatch('setSelectedNote', {
        _id: this.$route.params.id,
      });
    }
  },
}
</script>
