<template>
  <div>
    <div class="modal-body">
      <p class="announce">
        認証コードを入力してください
      </p>
      <p class="description">
        SMSに記載の6桁の認証コードを入力してください
      </p>
      <form class="signup-form" @keypress.enter.prevent="onSubmit">
        <div class="signup-form-group" :class="{ 'error': hasPhoneNumberError }">
          <label class="signup-form-label">認証コード</label>
          <input
            class="signup-form-input"
            type="text"
            placeholder="123456"
            maxlength="6"
            autofocus
            @input="setAuthCode"
            @blur="showError('authCode')"
            @focus="resetError('authCode')">
        </div>
      </form>
    </div>
    <div class="modal-footer">
      <p class="error-message">{{errorMessage}}</p>
      <p class="error-message" v-if="showErrorInvalidAuthCode">認証コードは6文字でご入力ください</p>
      <p class="error-message" v-if="showErrorAuthCodeNumeric">認証コードは数字でご入力ください</p>
      <app-button class="to-next-step-button" :disabled="invalidSubmit" @click="onSubmit">
        認証コードを送信する
      </app-button>
      <p class="back-to-input-phone-number">
        電話番号の入力に戻る場合は<span class="link" @click="backToInputPhoneNumber">こちら</span>
      </p>
    </div>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import { ADD_TOAST_MESSAGE } from 'vuex-toast'
import { required, minLength, maxLength, numeric } from 'vuelidate/lib/validators'
import AppButton from '../atoms/AppButton'

export default {
  data() {
    return {
      errorMessage: ''
    }
  },
  components: {
    AppButton
  },
  computed: {
    showErrorInvalidAuthCode() {
      return (
        this.requestPhoneNumberVerifyModal.inputAuthCode.formError.authCode &&
        (!this.$v.requestPhoneNumberVerifyModal.inputAuthCode.formData.authCode.minLength ||
          !this.$v.requestPhoneNumberVerifyModal.inputAuthCode.formData.authCode.maxLength)
      )
    },
    showErrorAuthCodeNumeric() {
      return (
        this.requestPhoneNumberVerifyModal.inputAuthCode.formError.authCode &&
        !this.$v.requestPhoneNumberVerifyModal.inputAuthCode.formData.authCode.numeric
      )
    },
    invalidSubmit() {
      return this.$v.requestPhoneNumberVerifyModal.inputAuthCode.formData.$invalid
    },
    hasUserIdOrEmailError() {
      return (
        this.requestPhoneNumberVerifyModal.inputAuthCode.formError.userIdOrEmail &&
        this.$v.requestPhoneNumberVerifyModal.inputAuthCode.formData.userIdOrEmail.$error
      )
    },
    hasPhoneNumberError() {
      return (
        this.requestPhoneNumberVerifyModal.inputAuthCode.formError.authCode &&
        this.$v.requestPhoneNumberVerifyModal.inputAuthCode.formData.authCode.$error
      )
    },
    ...mapGetters('user', ['requestPhoneNumberVerifyModal'])
  },
  validations: {
    requestPhoneNumberVerifyModal: {
      inputAuthCode: {
        formData: {
          authCode: {
            required,
            minLength: minLength(6),
            maxLength: maxLength(6),
            numeric
          }
        }
      }
    }
  },
  methods: {
    setAuthCode(e) {
      this.setRequestPhoneNumberVerifyInputAuthCodeAuthCode({ authCode: e.target.value })
    },
    showError(type) {
      this.$v.requestPhoneNumberVerifyModal.inputAuthCode.formData[type].$touch()
      this.showRequestPhoneNumberVerifyInputAuthCodeError({ type })
    },
    resetError(type) {
      this.$v.requestPhoneNumberVerifyModal.inputAuthCode.formData[type].$reset()
      this.hideRequestPhoneNumberVerifyInputAuthCodeError({ type })
    },
    async onSubmit() {
      if (this.invalidSubmit) return
      const { authCode: code } = this.requestPhoneNumberVerifyModal.inputAuthCode.formData
      try {
        await this.verifySMSCode({ code })
        await this.refreshUserSession()
        this.setRequestPhoneNumberVerifyModal({ isShow: false })
        this.setRequestPhoneNumberVerifyInputAuthCodeModal({ isShow: false })
        await this.setCurrentUserInfo()
        this.sendNotification({ text: '電話番号の登録が完了しました' })
      } catch (error) {
        let errorMessage = ''
        switch (error.code) {
          default:
            errorMessage = 'エラーが発生しました。入力内容をご確認ください'
            break
        }
        this.errorMessage = errorMessage
      }
    },
    backToInputPhoneNumber() {
      this.setRequestPhoneNumberVerifyInputAuthCodeModal({ isShow: false })
      this.setRequestPhoneNumberVerifyInputPhoneNumberModal({ isShow: true })
    },
    ...mapActions({
      sendNotification: ADD_TOAST_MESSAGE
    }),
    ...mapActions('user', [
      'setRequestPhoneNumberVerifyInputAuthCodeModal',
      'setRequestPhoneNumberVerifyInputAuthCodeAuthCode',
      'showRequestPhoneNumberVerifyInputAuthCodeError',
      'hideRequestPhoneNumberVerifyInputAuthCodeError',
      'setRequestPhoneNumberVerifyModal',
      'verifySMSCode',
      'refreshUserSession',
      'setCurrentUserInfo'
    ])
  }
}
</script>

<style lang="scss" scoped>
.modal-body {
  margin: 0 auto;

  .announce {
    @include default-text();
    font-size: 14px;
    margin: 60px 0 0;
    text-align: center;
  }

  .description {
    @include default-text();
    margin: 20px 0 0;
    text-align: center;
  }

  .signup-form {
    margin: 60px auto 0;
    max-width: 400px;
    width: 80%;

    &-label {
      color: #030303;
      font-size: 14px;
      line-height: 20px;
    }

    &-input {
      border: none;
      border-bottom: 1px dotted #232538;
      border-radius: 0;
      margin-bottom: 30px;
      padding: 5px 0;
      width: 100%;

      &::-webkit-input-placeholder {
        color: #cecece;
        font-size: 14px;
        letter-spacing: 0.05em;
      }

      &:focus {
        outline: 0;
      }
    }

    .error {
      .signup-form {
        &-label {
          color: #f06273;
        }

        &-input {
          border-bottom: 1px dotted #f06273;
        }
      }
    }
  }
}

.modal-footer {
  width: 270px;
  margin: 90px auto 40px;

  .to-next-step-button {
    margin: 20px auto 0;
  }

  .back-to-input-phone-number {
    @include default-text();
    text-align: right;

    .link {
      @include default-link();
    }
  }

  .error-message {
    color: #f06273;
    font-size: 12px;
    width: 100%;
  }
}

@media screen and (max-width: 320px) {
  .modal-body {
    .signup-form {
      margin-top: 30px;

      &-input {
        margin-bottom: 10px;
      }
    }
  }
}
</style>
