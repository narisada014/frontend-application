<template>
  <div class="modal-body">
    <div class="report-modal-content">
      <p class="confirm-text">
        本当に報告しますか？
      </p>
      <app-button class="report-button" @click="report">
        報告する
      </app-button>
      <button class="close-button" @click="closeModal">
        閉じる
      </button>
    </div>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import { ADD_TOAST_MESSAGE } from 'vuex-toast'
import AppButton from '../atoms/AppButton'
export default {
  components: {
    AppButton
  },
  computed: {
    ...mapGetters('report', ['articleReportModal'])
  },
  methods: {
    async report() {
      try {
        const { articleId } = this.$route.params
        const { reason } = this.articleReportModal.selectReason.formData
        const { originURL, freeText } = this.articleReportModal.inputFreeText.formData
        await this.postArticleFraud({ articleId, reason, originURL, freeText })
        this.sendNotification({ text: '報告しました' })
      } catch (error) {
        let text = 'エラーが発生しました。しばらく時間を置いて再度お試しください'
        if (error.response.data.message === 'Already exists') {
          text = 'すでに報告済みです'
        }
        this.sendNotification({ text, type: 'warning' })
      }
      this.setArticleReportConfirmationModal({ isShow: false })
      this.setArticleReportModal({ isShow: false })
      this.resetArticleReportData()
      document.querySelector('html').style.overflow = ''
      document.querySelector('body').style.overflow = ''
    },
    closeModal() {
      this.setArticleReportConfirmationModal({ isShow: false })
      this.setArticleReportModal({ isShow: false })
      this.resetArticleReportData()
      document.querySelector('html').style.overflow = ''
      document.querySelector('body').style.overflow = ''
    },
    ...mapActions({
      sendNotification: ADD_TOAST_MESSAGE
    }),
    ...mapActions('report', [
      'setArticleReportModal',
      'postArticleFraud',
      'setArticleReportConfirmationModal',
      'resetArticleReportData'
    ])
  }
}
</script>

<style lang="scss" scoped>
.confirm-text {
  color: #030303;
  font-size: 14px;
  font-weight: 500;
  line-height: 21px;
  text-align: center;
  padding: 50px 0 0;
  display: block;
}

.report-button {
  margin: 60px auto 0;
}

.close-button {
  background-color: #fff;
  border-radius: 30px;
  border: 1px solid #858dda;
  color: #858dda;
  cursor: pointer;
  display: block;
  height: 37px;
  margin: 20px auto 100px;
  width: 265px;
}

@media screen and (max-width: 550px) {
  .confirm-text {
    padding: 20vh 0 0;
  }
  .close-button {
    margin: 20px auto 30vh;
  }
}
</style>
