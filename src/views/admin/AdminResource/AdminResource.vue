<template>
  <div>
    <z-panel title="文章资源管理">
      <template v-slot:headerRight>
        <btn theme="primary" @click="handleAddResource">新增资源</btn>
      </template>
      <z-table :columns="columns" :data="tableData" :loading="isLoading" />
      <pagenation :all="pageTotal" :cur="page" :callback="handleChangePage" style="margin-top: 20px;" />
    </z-panel>

    <modal v-if="isShowResourceModal" @close="handleHideResourceModal">
      <h3 slot="header">{{ editMode === 'edit' ? '修改资源' : '添加资源' }}</h3>
      <div slot="body">
        <z-select v-model="formData.resourceTypeId" placeholder="请选择文章分类" :options="resourceTypeList" labelKey="name" valueKey="_id"></z-select>
        <input class="common-input" v-model="formData.name" type="text" placeholder="资源名称" />
        <input class="common-input" v-model="formData.url" type="text" placeholder="资源地址" />
        <textarea class="common-textarea" v-model="formData.desc" placeholder="资源描述" cols="30" rows="3"></textarea>
      </div>
      <div slot="footer">
        <btn theme="primary" :loading="isAddLoading || isEditLoading" long @click="handleSubmitResource">{{ editMode === 'edit' ? '确认修改' : '确认添加' }}</btn>
      </div>
    </modal>

    <modal v-if="isShowDeleteModal" @close="handleHideDeleteModal">
      <h3 slot="header">确认删除?</h3>
      <div slot="body">
        <p>确认删除名为 {{ currentRow.name }} 的资源吗?</p>
      </div>
      <div slot="footer">
        <btn theme="error" long @click="requestDeleteResource">确认删除</btn>
      </div>
    </modal>
  </div>
</template>

<script>
import api from '@/api/'
import { mapGetters, mapActions } from 'vuex'

import ZPanel from '@/components/base/ZPanel/ZPanel'
import ZTable from '@/components/base/Table/Table'
import Btn from '@/components/base/Btn/Btn'
import Pagenation from '@/components/base/Pagenation/Pagenation'
import Modal from '@/components/base/Modal/Modal'
import ZSelect from '@/components/base/ZSelect/ZSelect'

export default {
  name: 'AdminResource',
  components: {
    ZPanel,
    ZTable,
    Pagenation,
    Modal,
    Btn,
    ZSelect
  },
  data () {
    return {
      page: 1,
      limit: 10,
      pageTotal: 0,
      tableData: [],
      isResourceTypeLoading: false,
      resourceTypeList: [],
      isLoading: false,
      isAddLoading: false,
      isEditLoading: false,
      isDeleteLoading: false,
      currentRow: {},
      isShowDeleteModal: false,
      isShowResourceModal: false,
      editMode: 'add',
      formData: {},
      columns: [
        {
          title: '序号',
          type: 'index',
          width: 60
        },
        {
          title: '海报',
          key: 'poster',
          width: '80px',
          render: (h, parama) => {
            return h('img', {
              attrs: {
                src: parama.row.posterUrl
              },
              style: {
                width: '80px'
              }
            })
          }
        },
        {
          title: '名称',
          key: 'name',
          align: 'left'
        },
        {
          title: '资源分类',
          align: 'left',
          render: (h, params) => {
            return h('span', params.row.resourceTypeObj.name)
          }
        },
        {
          title: '地址',
          key: 'url',
          align: 'left'
        },
        {
          title: '简介',
          key: 'desc',
          minWidth: '100px',
          maxWidth: '400px;',
          align: 'left',
          width: '300px',
          render: (h, params) => {
            return h('span', params.row.desc || params.row.metaDesc)
          }
        },
        {
          title: '时间',
          render: (h, params) => {
            const createdAtFormat = this.$options.filters.dateFormatFilter(params.row.createdAt, 'YYYY-MM-DD HH:mm')
            const updatedAtFormat = this.$options.filters.dateFormatFilter(params.row.updatedAt, 'YYYY-MM-DD HH:mm')
            return h('div', [h('div', createdAtFormat), h('div', updatedAtFormat)])
          }
        },
        {
          title: '操作',
          render: (h, params) => {
            return h('div', [
              h(
                Btn,
                {
                  props: {
                    theme: 'primary',
                    size: 'small'
                  },
                  style: {
                    marginRight: '5px'
                  },
                  on: {
                    click: () => {
                      this.handleRowEdit(params.row)
                    }
                  }
                },
                '编辑'
              ),
              h(
                Btn,
                {
                  props: {
                    theme: 'error',
                    size: 'small'
                  },
                  style: {
                    marginRight: '5px'
                  },
                  on: {
                    click: () => {
                      this.handleRowDel(params.row)
                    }
                  }
                },
                '删除'
              )
            ])
          }
        }
      ]
    }
  },
  computed: {
    ...mapGetters([
      'userInfo'
    ])
  },
  mounted () {
    this.requestResourceTypeList()
    this.requestResourceList()
  },
  methods: {
    ...mapActions({
      toggleSignInModal: 'toggleSignInModal'
    }),

    /**
     * @desc 请求 获取文章资源类别列表
     */
    requestResourceTypeList () {
      this.isResourceTypeLoading = true
      api
        .GetResourceType()
        .then(res => {
          this.resourceTypeList = res.result
          this.isResourceTypeLoading = false
        })
        .catch(() => {
          this.isResourceTypeLoading = false
        })
    },

    /**
     * @desc 请求 获取文章资源列表
     */
    requestResourceList () {
      this.isLoading = true
      const params = {
        page: this.page,
        limit: this.limit
      }
      api
        .GetResource(params)
        .then(res => {
          this.tableData = res.result.list
          this.pageTotal = res.result.pages
          this.isLoading = false
        })
        .catch(() => {
          this.isLoading = false
        })
    },

    /**
     * @desc 请求 新增文章资源列表
     */
    requestAddResource () {
      const params = {
        ...this.formData
      }
      this.isAddLoading = true
      api
        .PostResource(params)
        .then(() => {
          this.isAddLoading = false
          this.$toast.success('添加成功！')
          this.handleHideResourceModal()
          this.requestResourceList()
        })
        .catch(() => {
          this.isAddLoading = false
        })
    },

    /**
     * @desc 请求 修改文章资源列表
     */
    requestEditResource () {
      const params = {
        ...this.formData,
        resourceId: this.currentRow._id
      }
      this.isEditLoading = true
      api
        .PutResource(params)
        .then(() => {
          this.isEditLoading = false
          this.$toast.success('修改成功！')
          this.handleHideResourceModal()
          this.requestResourceList()
        })
        .catch(() => {
          this.isEditLoading = false
        })
    },

    /**
     * @desc 请求 删除文章资源
     */
    requestDeleteResource () {
      this.isDeleteLoading = true
      api
        .DeleteResource({ resourceId: this.currentRow._id })
        .then(() => {
          this.isDeleteLoading = false
          this.$toast.success('删除成功！')
          this.handleHideDeleteModal()
          this.requestResourceList()
        })
        .catch(() => {
          this.isDeleteLoading = false
        })
    },

    /**
     * @desc 分页点击
     */
    handleChangePage (page) {
      this.page = page
      this.requestResourceList()
    },

    /**
     * @desc 清空表单值
     */
    handleClearFormData () {
      this.formData = {}
    },

    /**
     * @desc 回显表单值
     */
    handleRecoveryFormData (data) {
      this.formData = {
        name: data.name,
        url: data.url,
        oldUrl: data.url,
        desc: data.desc,
        metaDesc: data.metaDesc,
        resourceTypeId: [data.resourceTypeId]
      }
    },

    /**
     * @desc 新增文章资源
     */
    handleAddResource () {
      this.editMode = 'add'
      this.handleShowResourceModal()
    },

    /**
     * @desc 表格点击事件 编辑
     */
    handleRowEdit (row) {
      this.editMode = 'edit'
      this.currentRow = row
      if (this.handleValidateUserAuth()) {
        this.handleRecoveryFormData(row)
        this.handleShowResourceModal()
      }
    },

    /**
     * @desc 表格点击事件 删除
     */
    handleRowDel (row) {
      this.currentRow = row
      if (this.handleValidateUserAuth()) {
        this.handleShowDeleteModal()
      }
    },

    /**
     * @desc 检查表单数据是否合格
     */
    handleCheckFormData () {
      if (!this.formData.resourceTypeId) {
        this.$toast.error('请选择资源类别')
        return false
      } else if (!this.formData.name) {
        this.$toast.error('请填写资源名称')
        return false
      }
      return true
    },

    /**
     * @desc 验证是否已登录，是否为 admin 用户
     */
    handleValidateUserAuth () {
      let isUserAuth = false
      if (this.userInfo) {
        if (this.userInfo.userName === 'admin') {
          isUserAuth = true
        } else {
          this.$toast.warning('非admin，无权限！')
        }
      } else {
        this.$toast.info('请登录')
        this.toggleSignInModal(true)
      }
      return isUserAuth
    },

    /**
     * @desc 提交文章资源表单
     */
    handleSubmitResource () {
      if (!this.handleCheckFormData()) {
        return
      }
      if (!this.handleValidateUserAuth()) {
        return
      }
      if (this.editMode === 'edit') {
        this.requestEditResource()
      } else {
        this.requestAddResource()
      }
    },

    /**
     * @desc 显示文章资源表单弹框
     */
    handleShowResourceModal () {
      this.isShowResourceModal = true
    },

    /**
     * @desc 隐藏文章资源表单弹框
     */
    handleHideResourceModal () {
      this.isShowResourceModal = false
      this.handleClearFormData()
    },

    /**
     * @desc 显示删除文章资源弹框
     */
    handleShowDeleteModal () {
      this.isShowDeleteModal = true
    },

    /**
     * @desc 隐藏删除文章资源弹框
     */
    handleHideDeleteModal () {
      this.isShowDeleteModal = false
    }
  }
}

</script>

<style lang="less" scoped>
</style>
