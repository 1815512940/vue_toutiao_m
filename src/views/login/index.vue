<template>
  <div class="login-container">
    <!-- 导航栏 -->
    <van-nav-bar class="page-nav-bar"  title="登录" >
      <van-icon slot="left" name="cross" @click="$router.back()" />
    </van-nav-bar>

    <!-- 登录表单 -->
    <van-form @submit="onSubmit" ref="loginForm">
        <van-field
            v-model="user.mobile"
            name="mobile"
            placeholder="请输入手机号"
            maxlength="11"
            type="number"
            :rules="userFormRules.mobile"
        >
        <i slot="left-icon" class="toutiao toutiao-shouji"></i>
        </van-field>
        <van-field
            v-model="user.code"
            name="code"
            placeholder="请输入验证码"
            maxlength="6"
            :rules="userFormRules.code"
        >
            <i slot="left-icon" class="toutiao toutiao-yanzhengma"></i>
            <template #button>
                <van-count-down v-if="isCountDownShow" :time="1000 * 5" format="ss s" @finish="isCountDownShow = false" />
                <van-button v-else round class="send-sms-btn" size="small" type="default" @click="onSendSms">获取验证码</van-button>
            </template>
        </van-field>
        <div class="login-btn-wrap">
            <van-button class="login-btn" block type="info" native-type="submit">登录</van-button>
        </div>
    </van-form>
  </div>
</template>

<script>
import { login, sendSms } from '@/api/user.js'

export default {
  name: 'LoginIndex',
  components: {},
  props: {},
  data () {
    return {
      user: {
        mobile: '17811111111',
        code: '246810'
      },
      userFormRules: {
        mobile: [
          { required: true, message: '请输入用户名' },
          { pattern: /^1[3/5/7/8]\d{9}$/, message: '手机号格式错误' }
        ],
        code: [
          { required: true, message: '请输入验证码' },
          { pattern: /^\d{6}$/, message: '验证码错误' }
        ]
      },
      isCountDownShow: false
    }
  },
  computed: {},
  watch: {},
  created () {},
  mounted () {},
  methods: {
    async onSubmit () {
      const user = this.user

      this.$toast.loading({
        message: '加载中...',
        forbidClick: true, // 是否禁止背景点击
        duration: 0 // 持续时间b
      })
      try {
        const { data } = await login(user)
        this.$toast.success('登陆成功', data)
        this.$store.commit('setUser', data.data)

        this.$router.back()
      } catch (err) {
        if (err.response.status === 400) {
          this.$toast.fail('手机号或验证码错误')
        }
        this.$toast.fail('登录失败，请稍后重试')
      }
    },
    async onSendSms () {
      try {
        await this.$refs.loginForm.validate('mobile')
      } catch (err) {
        return console.log('验证失败', err)
      }

      this.isCountDownShow = true

      try {
        await sendSms(this.user.mobile)
        this.$toast('发送成功')
      } catch (err) {
        // 发送失败，关闭倒计时
        this.isCountDownShow = false
        if (err.response.status === 429) {
          return this.$toast('发送太频繁了，请稍后重试')
        }
        this.$toast('发送失败，请稍后重试')
      }
    }
  }
}
</script>

<style scoped lang="less">
.login-container {
  .toutiao {
    font-size: 37px;
  }
  .send-sms-btn {
    width: 158px;
    height: 46px;
    line-height: 46px;
    background-color: #ededed;
    font-size: 22px;
    color: #666;
  }
  .login-btn-wrap {
    padding: 53px 33px;
    .login-btn {
      background-color: #6db4fb;
      border: none;
      border-radius: 10px;
    }
  }
}
</style>
