<template>
  <popular-article-list/>
</template>

<script>
import { mapActions } from 'vuex'
import { ADD_TOAST_MESSAGE } from 'vuex-toast'
import PopularArticleList from '~/components/pages/PopularArticleList'

export default {
  components: {
    PopularArticleList
  },
  async fetch({ store }) {
    const topic = 'crypto'
    await store.dispatch('article/getTopics')
    store.dispatch('article/resetArticleData')
    await store.dispatch('article/getPopularArticles', { topic })
  },
  async mounted() {
    try {
      const { code } = this.$route.query

      this.$router.replace('/articles/popular?topic=crypto')

      if (!code) return
      const { hasUserId, status } = await this.$store.dispatch('user/checkAuthByLine', {
        code
      })
      if (!hasUserId) {
        this.$store.dispatch('user/setSignUpAuthFlowModal', { showSignUpAuthFlowModal: true })
        this.$store.dispatch('user/setSignUpAuthFlowInputUserIdModal', { isShow: true })
        return
      }
      await this.$store.dispatch('user/getUserSession')
      if (status === 'login') return
      this.$store.dispatch('user/setSignUpAuthFlowModal', { showSignUpAuthFlowModal: true })
      this.$store.dispatch('user/setSignUpAuthFlowCompletedAuthModal', { isShow: true })
    } catch (error) {
      const { message } = error.response.data
      switch (message) {
        case 'EmailExistsException':
          this.sendNotification({
            text: 'ご利用いただいた外部サービスに紐づくメールアドレスは既に登録されています'
          })
          this.$store.dispatch('user/setSignUpModal', { showSignUpModal: true })
          break
        case 'NotRegistered':
          this.sendNotification({
            text: 'アカウントが登録されていません。新規登録を行ってください',
            type: 'warning'
          })
          break
        default:
          this.sendNotification({
            text: 'エラーが発生しました。お手数ですが、時間をおいて再度お試しください',
            type: 'warning'
          })
          break
      }
    }
  },
  methods: {
    ...mapActions({
      sendNotification: ADD_TOAST_MESSAGE
    })
  }
}
</script>
