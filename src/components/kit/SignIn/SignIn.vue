<template>
  <modal v-if="showSignInModal" @close="handleCloseModel">
    <h3 slot="header">登录</h3>
    <div slot="body">
      <input class="common-input" v-model="formData.account" type="text" placeholder="用户名" style="width: 100%" />
      <input class="common-input" v-model="formData.password" type="password" placeholder="密码" style="width: 100%" />
      <div style="text-align: right;">
        <span style="font-size: 12px;color: #999;cursor: pointer;" @click="handleForgetPwd">忘记密码？</span>
      </div>
    </div>
    <div slot="footer">
      <btn theme="primary" long @click="login" :loading="isLoginLoading">确定登录</btn>
      <btn long @click="goToRegister" style="margin-top: 10px;">我要注册</btn>
    </div>
  </modal>
</template>

<script>
import api from '@/api/'
import webStore from '@/utils/storage'
import { mapGetters, mapActions } from 'vuex'

import Modal from '@/components/base/Modal/Modal'
import Btn from '@/components/base/Btn/Btn'
export default {
  name: 'SignIn',
  components: {
    Modal,
    Btn
  },
  data () {
    return {
      formData: {
        account: '',
        password: ''
      },
      isLoginLoading: false
    }
  },
  computed: {
    ...mapGetters([
      'showSignInModal'
    ])
  },
  watch: {},
  created () { },
  mounted () { },
  methods: {
    ...mapActions({
      toggleSignInModal: 'toggleSignInModal',
      handleChangeUserInfo: 'changeUserInfo'
    }),
    /**
     * @desc 请求 登录
     */
    requestLogin () {
      const params = {
        userName: this.formData.account,
        password: this.formData.password
      }
      this.isLoginLoading = true
      api
        .PostUserLogin(params)
        .then(res => {
          webStore.setUserInfo(res.result)
          this.handleChangeUserInfo(res.result)
          this.isLoginLoading = false
          this.$toast.success('登录成功')
          this.handleCloseModel()
        })
        .catch(() => {
          this.isLoginLoading = false
        })
    },

    /**
     * @desc 按钮点击事件 登录
     */
    login () {
      this.validateFormData().then(isvalid => {
        if (isvalid) {
          this.requestLogin()
        }
      })
    },

    /**
     * @desc 按钮点击事件 忘记密码
     */
    handleForgetPwd () {
      this.$router.push('/forget')
      this.$store.dispatch('toggleSignInModal', false)
      this.$store.dispatch('toggleSignUpModal', false)
    },

    /**
     * @desc 按钮点击事件 去注册
     */
    goToRegister () {
      this.$store.dispatch('toggleSignInModal', false)
      this.$store.dispatch('toggleSignUpModal', true)
    },

    /**
     * @desc 校验表单数据是否符合要求
     */
    validateFormData () {
      return new Promise(resolve => {
        if (!this.formData.account) {
          this.$toast.error('请输入昵称')
          return resolve(false)
        }
        if (!this.formData.password) {
          this.$toast.error('请输入密码')
          return resolve(false)
        }
        return resolve(true)
      })
    },

    /**
     * @desc 关闭弹框
     */
    handleCloseModel () {
      this.toggleSignInModal(false)
    }
  }
}
</script>

<style lang="less" scoped>
</style>
