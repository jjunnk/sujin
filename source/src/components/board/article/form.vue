<template>
  <v-container
    fluid
    :class="$vuetify.breakpoint.xs ? 'pa-0' : 'px-md-7 py-md-7 px-lg-12'"
  >
    <v-form>
      <v-card :loading="loading" class="">
        <v-toolbar color="transparent" dense flat>
          <v-toolbar-title>Article 작성</v-toolbar-title>
          <v-spacer />
          <v-btn icon @click="$router.push('/board/' + boardId)">
            <v-icon>mdi-arrow-left</v-icon>
          </v-btn>
          <v-btn icon @click="save" :disabled="!user">
            <v-icon>mdi-content-save</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text>
          <v-row>
            <v-col cols="12" sm="4" v-if="board">
              <v-combobox
                v-model="form.category"
                :items="board.categories"
                label="분류"
                outlined
                hide-details
              />
            </v-col>
            <v-col cols="12" sm="8" v-if="board">
              <v-combobox
                v-model="form.tags"
                :items="board.tags"
                label="태그"
                outlined
                hide-details
                multiple
                small
                small-chips
              />
            </v-col>
          </v-row>
          <v-text-field v-model="form.title" outlined label="제목"></v-text-field>
          <editor
            v-if="articleId === 'new'"
            :initialValue="form.content"
            ref="editor"
            initialEditType="wysiwyg"
            :options="{ hideModeSwitch: true }"
            height="500px"
          ></editor>
          <template v-else>
            <editor
              v-if="form.content"
              :initialValue="form.content"
              ref="editor"
              initialEditType="wysiwyg"
              :options="{ hideModeSwitch: true }"
              height="500px"
            ></editor>
            <v-container v-else>
              <v-row justify="center" align="center">
                <v-progress-circular indeterminate></v-progress-circular>
              </v-row>
            </v-container>
          </template>
        </v-card-text>
      </v-card>
    </v-form>
  </v-container>
</template>

<script>
import axios from "axios";
import getSummary from "@/util/getSummary";

export default {
  props: ["boardId", "articleId", "action"],
  data() {
    return {
      form: {
        category: "일반",
        tags: [],
        title: "",
        content: "",
      },
      exists: false,
      loading: false,
      ref: null,
      article: null,
      board: null,
    };
  },

  computed: {
    user() {
      return this.$store.state.user;
    },
    fireUser() {
      return this.$store.state.fireUser;
    },
  },
  watch: {
    boardId() {
      this.fetch();
    },
  },
  created() {
    this.fetch();
  },
  methods: {
    async fetch() {
      this.ref = this.$firebase.firestore().collection("boards").doc(this.boardId);
      const docBoard = await this.ref.get();
      this.board = docBoard.data(); // board 가져오기
      if (this.articleId === "new") return;
      const doc = await this.ref.collection("articles").doc(this.articleId).get();
      this.exists = doc.exists;
      if (!this.exists) return;
      const item = doc.data();
      this.article = item;
      this.form.title = item.title;
      this.form.category = item.category;
      this.form.tags = item.tags;
      const { data } = await axios.get(item.url);
      this.form.content = data;
    },
    async save() {
      if (!this.fireUser) throw Error("로그인이 필요합니다");
      if (!this.form.title) throw Error("제목은 필수 항목입니다");
      if (!this.form.category) throw Error("분류 필수 항목입니다");
      const md = this.$refs.editor.invoke("getMarkdown");
      if (!md) throw Error("내용은 필수 항목입니다");
      this.loading = true;

      try {
        const createdAt = new Date();
        const hash = "#";
        const doc = {
          title: this.form.title,
          category: this.form.category,
          tags: this.form.tags,
          updatedAt: createdAt,
          summary: getSummary(md, 300),
        };
        if (this.articleId === "new") {
          const id = createdAt.getTime().toString();
          const fn = id + "-" + this.fireUser.uid + ".md"; // 파일명
          const sn = await this.$firebase
            .storage()
            .ref()
            .child("boards")
            .child(this.boardId)
            .child(fn)
            .putString(md);
          doc.url = await sn.ref.getDownloadURL();
          doc.createdAt = createdAt;
          doc.commentCount = 0;
          doc.readCount = 0;
          doc.uid = this.$store.state.fireUser.uid;

          doc.user = {
            email: this.user.email,
            photoURL: this.user.photoURL,
            displayName: this.user.displayName,
          };
          doc.likeCount = 0;
          doc.likeUids = [];

          await this.ref.collection("articles").doc(id).set(doc);
          this.$router.push("/board/" + this.boardId);
        } else {
          const fn = this.articleId + "-" + this.article.uid + ".md"; // 파일명
          await this.$firebase
            .storage()
            .ref()
            .child("boards")
            .child(this.boardId)
            .child(fn)
            .putString(md);
          await this.ref.collection("articles").doc(this.articleId).update(doc);
          this.$router.push(this.$route.path);
        }
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>
