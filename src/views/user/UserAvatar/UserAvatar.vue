<template>
  <card :padding="20" style="height: 100%;">
    <user-page-title title="修改头像" titleSub="提示：有一个好的头像会有意想不到的收获哦。"></user-page-title>

    <div class="z-row">
      <div class="z-col-sm-30 z-col-md-26 z-col-lg-22 z-col-xl-18">
        <div class="avatar-preview">
          <p class="avatar-preview-title">--- 头像预览 ---</p>

          <div class="avatar-preview-img avatar-preview-img-large" :style="{ 'background-image': 'url(' + previewImg + ')' }"></div>
          <p class="avatar-preview-tip">大尺寸头像：100x100像素</p>

          <div class="avatar-preview-img avatar-preview-img-middle" :style="{ 'background-image': 'url(' + previewImg + ')' }"></div>
          <p class="avatar-preview-tip">中尺寸头像：50x50像素</p>

          <div class="avatar-preview-img avatar-preview-img-small" :style="{ 'background-image': 'url(' + previewImg + ')' }"></div>
          <p class="avatar-preview-tip">小尺寸头像：30x30像素</p>
        </div>
      </div>

      <div class="z-col-sm-30 z-col-md-34 z-col-lg-38 z-col-xl-32">
        <upload class="upload-input" action="/api/upload" accept="image/*" :format="['png', 'jpeg', 'jpg', 'gif']" :data="uploadParams" :max-size="2048"
          :on-format-error="handleFormatError" :on-exceeded-size="handleMaxSize" :on-success="handleUploadSuccess">
          <btn long>上传头像</btn>
        </upload>
        <p class="upload-tip">仅支持JPG、GIF、JPEG、PNG格式、文件小于2M</p>

        <btn :theme="formData.avatar ? 'success' : 'default'" style="margin-top: 30px;" @click="handleConfirm">确定</btn>
      </div>
    </div>
  </card>
</template>

<script>
import Card from '@/components/base/Card/Card'
import UserPageTitle from '@/views/user/components/UserPageTitle'
import Btn from '@/components/base/Btn/Btn'
import Upload from '@/components/base/Upload/Upload'

import webStore from '@/utils/storage'
import api from '@/api/'
import { mapGetters, mapActions } from 'vuex'

export default {
  name: 'UserProfile',
  components: {
    Card,
    Btn,
    Upload,
    UserPageTitle
  },
  data () {
    return {
      uploadParams: { usedFor: 'avatar' },
      formData: {
        avatar: ''
      }
    }
  },
  computed: {
    ...mapGetters([
      'userInfo'
    ]),
    previewImg () {
      if (this.formData.avatar) {
        return this.formData.avatar
      }
      if (this.userInfo && this.userInfo.avatar) {
        return this.userInfo.avatar
      }
      return ''
    }
  },
  methods: {
    ...mapActions({
      handleChangeUserInfo: 'changeUserInfo'
    }),

    /**
     * @desc 上传 格式出错
     */
    handleFormatError (file) {
      this.$toast.warning(`文件 ${file.name} 格式不对, 请选择 JPG、GIF、JPEG or PNG.`, { duration: 4000 })
    },

    /**
     * @desc 上传 大小限制
     */
    handleMaxSize (file) {
      this.$toast.warning(`文件 ${file.name} 太大, 不可超过2M`)
    },

    /**
     * @desc 上传 成功
     */
    handleUploadSuccess (res) {
      this.formData.avatar = res.result.path
    },

    /**
     * @desc 确认 保存头像
     */
    handleConfirm () {
      if (!this.formData.avatar) {
        this.$toast.info('请先上传图片')
        return
      }
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
.avatar-preview {
  background-color: @colorBg;
  padding: 20px;
  text-align: center;
  color: @colorTextContent;
  border-radius: 3px;
  .avatar-preview-title {
    margin: 10px 0 20px 0;
  }
  .avatar-preview-tip {
    margin: 10px 0 20px 0;
  }
  .avatar-preview-img {
    border-radius: 100%;
    box-sizing: border-box;
    margin: auto;
    display: block;
    padding: 1px;
    border: 1px solid rgba(0, 0, 0, 0.05);
    background-color: #fff;
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;
  }
  .avatar-preview-img-large {
    width: 100px;
    height: 100px;
  }
  .avatar-preview-img-middle {
    width: 50px;
    height: 50px;
  }
  .avatar-preview-img-small {
    width: 30px;
    height: 30px;
  }
}
.upload-input {
  width: 180px;
}
.upload-tip {
  font-size: 12px;
  color: @colorTextSub;
}
</style>
