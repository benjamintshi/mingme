<template>
  <card :padding="20" style="height: 100%;">
    <user-page-title title="个人资料" titleSub="请如实填写以下内容，让大家更好的交流互动。"></user-page-title>
    <form-item required label="用户名" tip="用户名不可更改">
      <input v-model.trim="formData.userName" class="form-item-input form-item-input-readonly" type="text" readonly />
    </form-item>
    <form-item label="昵称">
      <input v-model.trim="formData.nicName" class="form-item-input" type="text" placeholder="请填写昵称" />
    </form-item>
    <form-item required label="邮箱">
      <input v-model.trim="formData.email" class="form-item-input" type="text" placeholder="请填写邮箱" />
    </form-item>
    <form-item label="手机号">
      <input v-model.trim="formData.phone" class="form-item-input" type="text" placeholder="请输入手机号" />
    </form-item>
    <form-item label="个人说明">
      <textarea v-model.trim="formData.briefDesc" class="form-item-textarea" cols="60" rows="5" placeholder="介绍下自己吧"></textarea>
      <div style="color: #999;">{{ formData.briefDesc ? formData.briefDesc.length : 0 }} / 100</div>
    </form-item>
    <form-item>
      <btn theme="info" @click="handleUpdate" :loading="isEditLoading">更新个人资料</btn>
    </form-item>
  </card>
</template>

<script>
import webStore from '@/utils/storage'
import api from '@/api/'
import { validatorsExp } from '@/utils/validate'
import { mapGetters, mapActions } from 'vuex'

import Card from '@/components/base/Card/Card'
import UserPageTitle from '../components/UserPageTitle'
import FormItem from '@/components/base/FormItem/FormItem'
import Btn from '@/components/base/Btn/Btn'

export default {
  name: 'UserProfile',
  components: {
    Card,
    Btn,
    FormItem,
    UserPageTitle
  },
  data () {
    return {
      formData: {},
      isEditLoading: false
    }
  },
  computed: {
    ...mapGetters([
      'userInfo'
    ])
  },
  mounted () {
    const userInfo = webStore.getUserInfo()
    if (userInfo && userInfo.userName) {
      this.$set(this.formData, 'userName', userInfo.userName)
    }
    if (userInfo && userInfo.nicName) {
      this.$set(this.formData, 'nicName', userInfo.nicName)
    }
    if (userInfo && userInfo.phone) {
      this.$set(this.formData, 'phone', userInfo.phone)
    }
    if (userInfo && userInfo.email) {
      this.$set(this.formData, 'email', userInfo.email)
    }
    if (userInfo && userInfo.briefDesc) {
      this.$set(this.formData, 'briefDesc', userInfo.briefDesc)
    }
  },
  methods: {
    ...mapActions({
      handleChangeUserInfo: 'changeUserInfo'
    }),

    /**
     * @desc 更新个人资料
     */
    handleUpdate () {
      if (!this.handleValidete()) {
        return
      }
      this.requestUpdateUserInfo()
    },

    /**
     * @desc 校验参数
     */
    handleValidete () {
      const { userName, phone, email, briefDesc } = this.formData
      if (!userName) {
        this.$toast.error('请填写用户名！')
        return false
      }
      if (phone && !validatorsExp.phone.test(phone)) {
        this.$toast.error('请正确填写手机号！')
        return false
      }
      if (!email) {
        this.$toast.error('请填写邮箱！')
        return false
      }
      if (!validatorsExp.email.test(email)) {
        this.$toast.error('请正确填写邮箱！')
        return false
      }
      if (briefDesc && briefDesc.length > 100) {
        this.$toast.error('请用不超过100个字介绍自己')
        return false
      }
      return true
    },

    /**
     * @desc 请求 更新个人信息
     */
    requestUpdateUserInfo () {
      const params = {
        ...this.formData,
        userId: this.userInfo._id
      }
      this.isEditLoading = true
      api
        .PutUser(params)
        .then(res => {
          this.isEditLoading = false
          this.$toast.success('修改成功！')
          this.handleChangeUserInfo(res.result)
          webStore.setUserInfo(res.result)
        })
        .catch(() => {
          this.isEditLoading = false
        })
    }
  }
}
</script>

<style lang="less" scoped>
.form-item-input {
  padding: 8px 10px;
  width: 250px;
  font-size: 12px;
  color: @colorTextContent;
  outline: none;
  vertical-align: middle;
  border: 1px solid @colorBorder;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.05) inset;
}
.form-item-input-readonly {
  background-color: @colorTextSilver;
  color: @colorTextLight;
}
.form-item-textarea {
  padding: 8px 10px;
  font-size: 14px;
  line-height: 1.5;
  color: @colorTextContent;
  outline: none;
  vertical-align: middle;
  border: 1px solid @colorBorder;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.05) inset;
}
</style>
