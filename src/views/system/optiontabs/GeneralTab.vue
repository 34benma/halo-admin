<template>
  <div>
    <a-form-model ref="generalOptionsForm" :model="options" :rules="rules" :wrapperCol="wrapperCol" layout="vertical">
      <a-form-model-item label="博客标题：" prop="blog_title">
        <a-input v-model="options.blog_title" />
      </a-form-model-item>
      <a-form-model-item label="博客地址：" prop="blog_url">
        <a-input v-model="options.blog_url" placeholder="如：https://halo.run" />
      </a-form-model-item>
      <a-form-model-item label="Logo：" prop="blog_logo">
        <a-input v-model="options.blog_logo">
          <a slot="addonAfter" href="javascript:void(0);" @click="handleShowLogoSelector">
            <a-icon type="picture" />
          </a>
        </a-input>
      </a-form-model-item>
      <a-form-model-item label="Favicon：" prop="blog_favicon">
        <a-input v-model="options.blog_favicon">
          <a slot="addonAfter" href="javascript:void(0);" @click="handleShowFaviconSelector">
            <a-icon type="picture" />
          </a>
        </a-input>
      </a-form-model-item>
      <a-form-model-item label="页脚信息：" prop="blog_footer_info">
        <a-input
          v-model="options.blog_footer_info"
          :autoSize="{ minRows: 5 }"
          placeholder="支持 HTML 格式的文本"
          type="textarea"
        />
      </a-form-model-item>
      <a-form-model-item>
        <ReactiveButton
          :errored="errored"
          :loading="saving"
          erroredText="保存失败"
          loadedText="保存成功"
          text="保存"
          type="primary"
          @callback="$emit('callback')"
          @click="handleSaveOptions"
        ></ReactiveButton>
      </a-form-model-item>
    </a-form-model>

    <AttachmentSelectDrawer
      v-model="attachmentSelector.visible"
      :title="attachmentSelectorTitle"
      @listenToSelect="handleSelectAttachment"
    />
  </div>
</template>

<script>
export default {
  name: 'GeneralTab',
  props: {
    options: {
      type: Object,
      required: true
    },
    saving: {
      type: Boolean,
      default: false
    },
    errored: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      wrapperCol: {
        xl: { span: 8 },
        lg: { span: 8 },
        sm: { span: 12 },
        xs: { span: 24 }
      },
      attachmentSelector: {
        visible: false,
        field: ''
      },
      rules: {
        blog_title: [
          { required: true, message: '* 博客标题不能为空', trigger: ['change'] },
          { max: 1023, message: '* 字符数不能超过 1023', trigger: ['change'] }
        ],
        blog_url: [
          { required: true, message: '* 博客地址不能为空', trigger: ['change'] },
          { max: 1023, message: '* 字符数不能超过 1023', trigger: ['change'] }
        ],
        blog_logo: [{ max: 1023, message: '* 字符数不能超过 1023', trigger: ['change'] }],
        blog_favicon: [{ max: 1023, message: '* 字符数不能超过 1023', trigger: ['change'] }],
        blog_footer_info: [{ max: 1023, message: '* 字符数不能超过 1023', trigger: ['change'] }]
      }
    }
  },
  computed: {
    attachmentSelectorTitle() {
      if (this.attachmentSelector.field === 'blog_logo') {
        return '选择 Logo'
      } else if (this.attachmentSelector.field === 'blog_favicon') {
        return '选择 Favicon'
      }
      return ''
    }
  },
  destroyed: function() {
    if (this.attachmentSelector.visible) {
      this.attachmentSelector.visible = false
    }
  },
  beforeRouteLeave(to, from, next) {
    if (this.attachmentSelector.visible) {
      this.attachmentSelector.visible = false
    }
    next()
  },
  watch: {
    options(val) {
      this.$emit('onChange', val)
    }
  },
  methods: {
    handleSaveOptions() {
      const _this = this
      _this.$refs.generalOptionsForm.validate(valid => {
        if (valid) {
          this.$emit('onSave')
        }
      })
    },
    handleShowLogoSelector() {
      this.attachmentSelector.field = 'blog_logo'
      this.attachmentSelector.visible = true
    },
    handleShowFaviconSelector() {
      this.attachmentSelector.field = 'blog_favicon'
      this.attachmentSelector.visible = true
    },
    handleSelectAttachment(attachment) {
      this.$set(this.options, this.attachmentSelector.field, encodeURI(attachment.path))
      this.attachmentSelector.visible = false
    }
  }
}
</script>
