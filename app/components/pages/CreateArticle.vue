<template>
  <div class="create-article-container">
    <app-header />
    <edit-header-nav class="drafts" />
    <article-editor :putArticle="putArticle"/>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import AppHeader from '../organisms/AppHeader'
import EditHeaderNav from '../molecules/EditHeaderNav'
import ArticleEditor from '../atoms/ArticleEditor'

export default {
  components: {
    AppHeader,
    EditHeaderNav,
    ArticleEditor
  },
  computed: {
    ...mapGetters('article', ['articleId', 'title', 'body'])
  },
  methods: {
    ...mapActions('article', ['putDraftArticle']),
    async putArticle() {
      const { title, body, articleId } = this
      const article = { title, body }
      await this.putDraftArticle({ article, articleId })
    }
  }
}
</script>

<style lang="scss" scoped>
.create-article-container {
  display: grid;
  grid-template-rows: 100px 40px 50px 650px 75px;
  grid-template-columns: 1fr 640px 1fr;
  /* prettier-ignore */
  grid-template-areas:
    "app-header  app-header app-header"
    "nav         nav        nav       "
    "...         ...        ...       "
    "...         editor     ...       "
    "...         ...        ...       ";
}
</style>
