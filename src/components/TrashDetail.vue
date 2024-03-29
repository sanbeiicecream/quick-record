<template>
  <div id="trash" class="detail">
    <div class="note-sidebar">
      <h3 class="notebook-title">回收站</h3>
      <div class="menu">
        <div>更新时间</div>
        <div>标题</div>
      </div>
      <ul class="notes">
        <li v-for="(note,index) in trashNotes" :key="index">
          <router-link :to="`/trash?noteId=${note.id}`">
            <span class="date">{{ note.updatedAtFriendly }}</span>
            <span class="title">{{ note.title }}</span>
          </router-link>
        </li>
      </ul>
    </div>
    <div class="note-detail">
      <div class="note-bar" v-if="true">
        <span> 创建日期: {{ currentTrashNote.createdAtFriendly }}</span>
        <span> | </span>
        <span> 更新日期: {{ currentTrashNote.updatedAtFriendly }}</span>
        <span> | </span>
        <span> 所属笔记本: {{ belongTo }}</span>

        <a class="btn action" @click="onRevert">恢复</a>
        <a class="btn action" @click="onDelete">彻底删除</a>
      </div>
      <div class="note-title">
        <span>{{ currentTrashNote.title }}</span>
      </div>
      <div class="editor">
        <div class="preview markdown-body" v-html="compiledMarkdown"></div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import {Component} from 'vue-property-decorator';
import {mapActions, mapGetters, mapMutations} from 'vuex';
import {Note} from '@/helpers/noteType';
import {Message, MessageBox} from 'element-ui';
import marked from 'marked';
Component.registerHooks([
  'beforeRouteUpdate',
]);
@Component({
  methods: {...mapActions(['checkLogin', 'getTrashNotes', 'revertTrashNote', 'deleteTrashNote', 'getNotebooks']), ...mapMutations(['setCurrentTrashNote'])},
  computed: mapGetters(['trashNotes', 'currentTrashNote', 'belongTo'])
})
export default class TrashDetail extends Vue {
  checkLogin!: ({path}: { path: string }) => Promise<void>;
  getTrashNotes!: () => Promise<void>;
  getNotebooks!: () => Promise<void>;
  deleteTrashNote!: ({noteId}: { noteId: number }) => Promise<void>;
  trashNotes!: Note[];
  currentTrashNote!: Note;
  revertTrashNote!: ({noteId}: { noteId: number }) => Promise<void>;
  setCurrentTrashNote!: ({currentTrashNoteId}?: { currentTrashNoteId: string }) => void;

  created() {
    this.checkLogin({path: 'login'});
    this.getTrashNotes().then(() => {
      this.getNotebooks().then(() => {
        if (this.$route.query.noteId) {
          this.setCurrentTrashNote({currentTrashNoteId: this.$route.query.noteId + ''});
        } else {
          this.setCurrentTrashNote();
        }
        this.$router.replace({query: {noteId: this.currentTrashNote.id + ''}}).catch(() => {return;});
      });
    });
  }

  get compiledMarkdown() {
    return marked(this.currentTrashNote.content || '');
  }

  onDelete() {
    if (this.trashNotes.length < 1) return;
    MessageBox.confirm('此操作将永久删除该文件, 是否继续?', '提示', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning'
    }).then(() => {
      return this.deleteTrashNote({noteId: this.currentTrashNote.id});
    }).then(() => {
      this.setCurrentTrashNote();
      this.$router.replace({path: '/trash', query: {noteId: this.currentTrashNote.id + ''}});
    }).catch(() => {
      Message({
        type: 'info',
        message: '已取消删除'
      });
    });
  }

  onRevert() {
    if (this.trashNotes.length < 1) return;
    this.revertTrashNote({noteId: this.currentTrashNote.id}).then(() => {
      this.setCurrentTrashNote();
      this.$router.replace({path: 'trash', query: {noteId: this.currentTrashNote.id + ''}});
    });
  }

  beforeRouteUpdate(to: any, from: any, next: any) {
    if (to.query.noteId === from.query.noteId) return;
    this.setCurrentTrashNote({currentTrashNoteId: to.query.noteId});
    next();
  }
}


</script>

<style lang="scss" scoped>
@import "~@/assets/style/noteSidebar.scss";
@import "~@/assets/style/noteDetail.scss";

#trash {
  display: flex;
  align-items: stretch;
  background-color: #fff;
  flex: 1;

  .note-bar {
    .action {
      float: right;
      margin-left: 10px;
      padding: 2px 4px;
      font-size: 12px;

    }
  }
}
</style>