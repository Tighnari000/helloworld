<template>
  <div class="sms-verifition-input">
    <div class="sms-verifition-input-input">
      <input :value="value" @input="e=>$emit('input',e.target.value)" :placeholder="placeholder" maxLength="6" />
    </div>
    <div :class="{'sms-verifition-input-btn': true, 'sms-verifition-input-disabled': disabled, 'sms-verifition-input-count-down': countDown > 0 }">
      <button @click="sendSMS" :disabled="disabled">{{btnText}}{{countDown > 0 ? ` (${countDown})` : ''}}
      </button>
    </div>
    <Verify
      v-if="needCaptcha"
      @success="success"
      :mode="mode"
      :captchaType="captchaType"
      :imgSize="imgSize"
      :vSpace="vSpace"
      :explain="explain"
      ref="verify"
    ></Verify>
  </div>
</template>

<script>
//引入组件
import Verify from "./verifition/Verify"
import service from "./verifition/utils/axios"
import SMSVerifitionHandler from './sms-verifition-handler'

export default {
  name: 'app',
  props: {
    phoneNum: String,
    value: String,
    captchaApiBaseURL: String,
    smsApiBaseURL: String,
    phoneNumkey: String,
    sendSmsCodeKey: String,
    mode: {
      type: String,
      default: 'pop'
    },
    captchaType: {
      type: String,
      default: 'blockPuzzle'
    },
    vSpace: String,
    explain: String,
    imgSize: {
      type: Object,
      default () {
        const width = window.innerWidth < 720 ? Math.min(400, window.innerWidth - 50) : 310
        return {
          width: width + 'px',
          height: width / 2 + 'px'
        }
      }
    },
    placeholder: {
      type: String,
      default: '短信验证码'
    },
    btnText: {
      type: String,
      default: '发送'
    },
    onSuccess: Function,
    onError: Function,
    beforeSend: Function,
    channelId: String,
    clientId: String,
    templateId: String,
    modelId: String,
  },
  components: {
    Verify
  },
  data () {
    return {
      smsVerifitionHandler: new SMSVerifitionHandler({
        smsApiBaseURL: this.smsApiBaseURL,
        phoneNumkey: this.phoneNumkey,
        sendSmsCodeKey: this.sendSmsCodeKey,
        onSuccess: this.onSuccess,
        onError: this.onError,
        onCaptcha: this.onCaptcha,
        onCountDown: this.onCountDown,
        channelId: this.channelId,
        clientId: this.clientId,
        templateId: this.templateId,
        modelId: this.modelId,
      }),
      countDown: 0,
      needCaptcha: false
    }
  },
  computed: {
    disabled () {
      return this.countDown > 0 || !this.smsVerifitionHandler.validatePhoneNum(this.phoneNum)
    }
  },
  methods:{
    success (params) {
      // params 返回的二次验证参数, 和参数一起回传给登录接口，方便后台进行二次验证
      this.smsVerifitionHandler.onCaptchaSuccess(params)
    },
    onCaptcha () {
      this.needCaptcha = true
      this.$nextTick(() => {
        // 挂到bodyh上防止样式干扰
        if (this.mode === 'pop') {
          document.body.appendChild(this.$refs.verify.$el)
        }
        this.$refs.verify.show()
        this.$emit('onVerifyShow')
      })
    },
    onCountDown (value) {
      this.countDown = value
    },
    sendSMS () {
      if (this.beforeSend && this.beforeSend() === false) {
        return
      }
      this.$emit('input', '')
      this.smsVerifitionHandler.sendSMS({ phoneNum: this.phoneNum })
    }
  },
  created () {
    service.defaults.baseURL = this.captchaApiBaseURL
  },
  beforeDestroy () {
    this.$refs.verify && this.$refs.verify.$el.remove()
  }
}
</script>
