<template>
  <div>
    <!-- Mobile -->
    <a-list
      v-if="isMobile()"
      :dataSource="formattedSheets"
      :loading="list.loading"
      :pagination="false"
      itemLayout="vertical"
      size="large"
    >
      <a-list-item :key="index" slot="renderItem" slot-scope="item, index">
        <template slot="actions">
          <span>
            <a-icon type="eye" />
            {{ item.visits }}
          </span>
          <span @click="handleShowSheetComments(item)">
            <a-icon type="message" />
            {{ item.commentCount }}
          </span>
          <a-dropdown :trigger="['click']" placement="topLeft">
            <span>
              <a-icon type="bars" />
            </span>
            <a-menu slot="overlay">
              <a-menu-item v-if="item.status === 'PUBLISHED' || item.status === 'DRAFT'">
                <a href="javascript:void(0);" @click="handleEditClick(item)">编辑</a>
              </a-menu-item>
              <a-menu-item v-else-if="item.status === 'RECYCLE'">
                <a-popconfirm
                  :title="'你确定要发布【' + item.title + '】页面？'"
                  cancelText="取消"
                  okText="确定"
                  @confirm="handleEditStatusClick(item.id, 'PUBLISHED')"
                >
                  <a href="javascript:void(0);">还原</a>
                </a-popconfirm>
              </a-menu-item>
              <a-menu-item v-if="item.status === 'PUBLISHED' || item.status === 'DRAFT'">
                <a-popconfirm
                  :title="'你确定要将【' + item.title + '】页面移到回收站？'"
                  cancelText="取消"
                  okText="确定"
                  @confirm="handleEditStatusClick(item.id, 'RECYCLE')"
                >
                  <a href="javascript:void(0);">回收站</a>
                </a-popconfirm>
              </a-menu-item>
              <a-menu-item v-else-if="item.status === 'RECYCLE'">
                <a-popconfirm
                  :title="'你确定要永久删除【' + item.title + '】页面？'"
                  cancelText="取消"
                  okText="确定"
                  @confirm="handleDeleteClick(item.id)"
                >
                  <a href="javascript:void(0);">删除</a>
                </a-popconfirm>
              </a-menu-item>
              <a-menu-item>
                <a href="javascript:void(0);" rel="noopener noreferrer" @click="handleShowSheetSettings(item)">设置</a>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </template>
        <template slot="extra">
          <span>
            <a-badge :status="item.statusProperty.status" :text="item.statusProperty.text" />
          </span>
        </template>
        <a-list-item-meta>
          <template slot="description">
            {{ item.createTime | moment }}
          </template>
          <span
            slot="title"
            style="max-width: 300px;display: block;white-space: nowrap;overflow: hidden;text-overflow: ellipsis;"
          >
            <a v-if="item.status === 'PUBLISHED'" :href="item.fullPath" class="no-underline" target="_blank">
              <a-tooltip :title="'点击访问【' + item.title + '】'" placement="top">{{ item.title }}</a-tooltip>
            </a>
            <a
              v-else-if="item.status === 'DRAFT'"
              class="no-underline"
              href="javascript:void(0)"
              @click="handlePreview(item.id)"
            >
              <a-tooltip :title="'点击预览【' + item.title + '】'" placement="topLeft">{{ item.title }}</a-tooltip>
            </a>
            <a v-else class="no-underline" disabled href="javascript:void(0);">
              {{ item.title }}
            </a>
          </span>
        </a-list-item-meta>
        <span> {{ item.summary }}... </span>
      </a-list-item>
    </a-list>

    <!-- Desktop -->
    <a-table
      v-else
      :columns="customColumns"
      :dataSource="formattedSheets"
      :loading="list.loading"
      :pagination="false"
      :rowKey="sheet => sheet.id"
      :scrollToFirstRowOnChange="true"
    >
      <span slot="sheetTitle" slot-scope="text, record">
        <a v-if="record.status === 'PUBLISHED'" :href="record.fullPath" class="no-underline" target="_blank">
          <a-tooltip :title="'点击访问【' + text + '】'" placement="top">{{ text }}</a-tooltip>
        </a>
        <a
          v-else-if="record.status === 'DRAFT'"
          class="no-underline"
          href="javascript:void(0)"
          @click="handlePreview(record.id)"
        >
          <a-tooltip :title="'点击预览【' + text + '】'" placement="topLeft">{{ text }}</a-tooltip>
        </a>
        <a v-else class="no-underline" disabled href="javascript:void(0);">
          {{ text }}
        </a>
      </span>

      <span slot="status" slot-scope="statusProperty">
        <a-badge :status="statusProperty.status" :text="statusProperty.text" />
      </span>

      <span
        slot="commentCount"
        slot-scope="text, record"
        class="cursor-pointer"
        @click="handleShowSheetComments(record)"
      >
        <a-badge
          :count="record.commentCount"
          :numberStyle="{ backgroundColor: '#f38181' }"
          :overflowCount="999"
          :showZero="true"
        />
      </span>

      <span slot="visits" slot-scope="visits">
        <a-badge :count="visits" :numberStyle="{ backgroundColor: '#00e0ff' }" :overflowCount="9999" :showZero="true" />
      </span>

      <span slot="createTime" slot-scope="createTime">
        <a-tooltip placement="top">
          <template slot="title">
            {{ createTime | moment }}
          </template>
          {{ createTime | timeAgo }}
        </a-tooltip>
      </span>

      <span slot="action" slot-scope="text, sheet">
        <a
          v-if="sheet.status === 'PUBLISHED' || sheet.status === 'DRAFT'"
          href="javascript:void(0);"
          @click="handleEditClick(sheet)"
          >编辑</a
        >

        <a-popconfirm
          v-else-if="sheet.status === 'RECYCLE'"
          :title="'你确定要发布【' + sheet.title + '】？'"
          cancelText="取消"
          okText="确定"
          @confirm="handleEditStatusClick(sheet.id, 'PUBLISHED')"
        >
          <a href="javascript:void(0);">还原</a>
        </a-popconfirm>

        <a-divider type="vertical" />

        <a-popconfirm
          v-if="sheet.status === 'PUBLISHED' || sheet.status === 'DRAFT'"
          :title="'你确定要将【' + sheet.title + '】页面移到回收站？'"
          cancelText="取消"
          okText="确定"
          @confirm="handleEditStatusClick(sheet.id, 'RECYCLE')"
        >
          <a href="javascript:void(0);">回收站</a>
        </a-popconfirm>

        <a-popconfirm
          v-else-if="sheet.status === 'RECYCLE'"
          :title="'你确定要永久删除【' + sheet.title + '】页面？'"
          cancelText="取消"
          okText="确定"
          @confirm="handleDeleteClick(sheet.id)"
        >
          <a href="javascript:void(0);">删除</a>
        </a-popconfirm>
        <a-divider type="vertical" />
        <a href="javascript:void(0);" @click="handleShowSheetSettings(sheet)">设置</a>
      </span>
    </a-table>
    <div class="page-wrapper">
      <a-pagination
        :current="pagination.page"
        :defaultPageSize="pagination.size"
        :pageSizeOptions="['10', '20', '50', '100']"
        :total="pagination.total"
        class="pagination"
        showLessItems
        showSizeChanger
        @change="handlePageChange"
        @showSizeChange="handlePageSizeChange"
      />
    </div>
    <SheetSettingModal
      :loading="sheetSettingLoading"
      :post="selectedSheet"
      :savedCallback="onSheetSavedCallback"
      :visible.sync="sheetSettingVisible"
      @onClose="selectedSheet = {}"
    >
      <template #extraFooter>
        <a-button :disabled="selectPreviousButtonDisabled" @click="handleSelectPrevious">
          上一篇
        </a-button>
        <a-button :disabled="selectNextButtonDisabled" @click="handleSelectNext">
          下一篇
        </a-button>
      </template>
    </SheetSettingModal>
    <TargetCommentDrawer
      :id="selectedSheet.id"
      :description="selectedSheet.summary"
      :target="`sheets`"
      :title="selectedSheet.title"
      :visible="sheetCommentVisible"
      @close="onSheetCommentsClose"
    />
  </div>
</template>
<script>
import { mixin, mixinDevice } from '@/mixins/mixin.js'
import SheetSettingModal from './SheetSettingModal'
import TargetCommentDrawer from '../../comment/components/TargetCommentDrawer'
import apiClient from '@/utils/api-client'

const customColumns = [
  {
    title: '标题',
    dataIndex: 'title',
    ellipsis: true,
    scopedSlots: { customRender: 'sheetTitle' }
  },
  {
    title: '状态',
    className: 'status',
    dataIndex: 'statusProperty',
    scopedSlots: { customRender: 'status' }
  },
  {
    title: '评论量',
    dataIndex: 'commentCount',
    scopedSlots: { customRender: 'commentCount' }
  },
  {
    title: '访问量',
    dataIndex: 'visits',
    scopedSlots: { customRender: 'visits' }
  },
  {
    title: '发布时间',
    dataIndex: 'createTime',
    scopedSlots: { customRender: 'createTime' }
  },
  {
    title: '操作',
    width: '180px',
    scopedSlots: { customRender: 'action' }
  }
]

const sheetStatus = {
  PUBLISHED: {
    color: 'green',
    status: 'success',
    text: '已发布'
  },
  DRAFT: {
    color: 'yellow',
    status: 'warning',
    text: '草稿'
  },
  RECYCLE: {
    color: 'red',
    status: 'error',
    text: '回收站'
  }
}

export default {
  name: 'CustomSheetList',
  mixins: [mixin, mixinDevice],
  components: {
    SheetSettingModal,
    TargetCommentDrawer
  },
  data() {
    return {
      customColumns,
      sheetStatus,

      list: {
        data: [],
        loading: false,
        total: 0,
        hasPrevious: false,
        hasNext: false,
        params: {
          page: 0,
          size: 10
        }
      },
      selectedSheet: {},
      sheetSettingVisible: false,
      sheetSettingLoading: false,
      sheetCommentVisible: false
    }
  },
  computed: {
    formattedSheets() {
      return this.list.data.map(sheet => {
        sheet.statusProperty = this.sheetStatus[sheet.status]
        return sheet
      })
    },
    pagination() {
      return {
        page: this.list.params.page + 1,
        size: this.list.params.size,
        total: this.list.total
      }
    },
    selectPreviousButtonDisabled() {
      const index = this.list.data.findIndex(sheet => sheet.id === this.selectedSheet.id)
      return index === 0 && !this.list.hasPrevious
    },
    selectNextButtonDisabled() {
      const index = this.list.data.findIndex(sheet => sheet.id === this.selectedSheet.id)
      return index === this.list.data.length - 1 && !this.list.hasNext
    }
  },
  created() {
    this.handleListSheets()
  },
  destroyed() {
    if (this.sheetSettingVisible) {
      this.sheetSettingVisible = false
    }
  },
  beforeRouteLeave(to, from, next) {
    if (this.sheetSettingVisible) {
      this.sheetSettingVisible = false
    }
    next()
  },
  methods: {
    async handleListSheets(enableLoading = true) {
      try {
        if (enableLoading) {
          this.list.loading = true
        }

        const { data } = await apiClient.sheet.list(this.list.params)

        this.list.data = data.content
        this.list.total = data.total
        this.list.hasPrevious = data.hasPrevious
        this.list.hasNext = data.hasNext
      } catch (e) {
        this.$log.error(e)
      } finally {
        this.list.loading = false
      }
    },
    handleEditClick(sheet) {
      this.$router.push({ name: 'SheetEdit', query: { sheetId: sheet.id } })
    },
    handleEditStatusClick(sheetId, status) {
      apiClient.sheet
        .updateStatusById(sheetId, status)
        .then(() => {
          this.$message.success('操作成功！')
        })
        .finally(() => {
          this.handleListSheets()
        })
    },
    handleDeleteClick(sheetId) {
      apiClient.sheet
        .delete(sheetId)
        .then(() => {
          this.$message.success('删除成功！')
        })
        .finally(() => {
          this.handleListSheets()
        })
    },
    handleShowSheetSettings(sheet) {
      apiClient.sheet.get(sheet.id).then(response => {
        this.selectedSheet = response.data
        this.sheetSettingVisible = true
      })
    },
    handleShowSheetComments(sheet) {
      apiClient.sheet.get(sheet.id).then(response => {
        this.selectedSheet = response.data
        this.sheetCommentVisible = true
      })
    },
    handlePreview(sheetId) {
      apiClient.sheet.getPreviewLinkById(sheetId).then(response => {
        window.open(response.data, '_blank')
      })
    },

    /**
     * Handle page change
     */
    handlePageChange(page = 1) {
      this.list.params.page = page - 1
      this.handleListSheets()
    },

    /**
     * Handle page size change
     */
    handlePageSizeChange(current, size) {
      this.$log.debug(`Current: ${current}, PageSize: ${size}`)
      this.list.params.page = 0
      this.list.params.size = size
      this.handleListSheets()
    },
    onSheetCommentsClose() {
      this.sheetCommentVisible = false
      this.selectedSheet = {}
      setTimeout(() => {
        this.handleListSheets(false)
      }, 500)
    },
    onSheetSavedCallback() {
      this.handleListSheets(false)
    },
    /**
     * Select previous sheet
     */
    async handleSelectPrevious() {
      const index = this.list.data.findIndex(post => post.id === this.selectedSheet.id)
      if (index > 0) {
        this.sheetSettingLoading = true
        const response = await apiClient.sheet.get(this.list.data[index - 1].id)
        this.selectedSheet = response.data
        this.sheetSettingLoading = false
        return
      }
      if (index === 0 && this.list.hasPrevious) {
        this.list.params.page--
        await this.handleListPosts()
        this.sheetSettingLoading = true
        const response = await apiClient.sheet.get(this.list.data[this.list.data.length - 1].id)
        this.selectedSheet = response.data
        this.sheetSettingLoading = false
      }
    },
    /**
     * Select next sheet
     */
    async handleSelectNext() {
      const index = this.list.data.findIndex(post => post.id === this.selectedSheet.id)
      if (index < this.list.data.length - 1) {
        this.sheetSettingLoading = true
        const response = await apiClient.sheet.get(this.list.data[index + 1].id)
        this.selectedSheet = response.data
        this.sheetSettingLoading = false
        return
      }
      if (index === this.list.data.length - 1 && this.list.hasNext) {
        this.list.params.page++
        await this.handleListPosts()
        this.sheetSettingLoading = true
        const response = await apiClient.sheet.get(this.list.data[0].id)
        this.selectedSheet = response.data
        this.sheetSettingLoading = false
      }
    }
  }
}
</script>
