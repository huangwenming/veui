<template>
<article>
  <h1><code>&lt;veui-uploader&gt;</code></h1>
  <h2>图片上传模式，上传进度以文字百分比显示</h2>
  <veui-uploader
    v-model="files"
    type="image"
    name="file"
    action="/upload"
    :max-count="3"
    max-size="10mb"
    accept=".jpg,.jpeg,.gif"
    ui="horizontal"
    :payload="payload"
    progress="percent"
    @success="onSuccess"
    @failure="onFailure"
    @change="handleChange('files')"
    @statuschange="handleStatusChange"
  >
    <template slot="desc">
      请选择jpg,jpeg,gif图片，大小在10M以内，最多上传3张图
    </template>
  </veui-uploader>
  <h2>图片上传模式，上传进度以进度条显示</h2>
  <veui-uploader
    v-model="files1"
    type="image"
    request-mode="custom"
    name="file"
    :max-count="3"
    max-size="10mb"
    accept=".jpg,.jpeg,.gif"
    :payload="payload"
    ui="horizontal"
    progress="bar"
    :upload="upload"
    @success="onSuccess"
    @failure="onFailure"
    @change="handleChange('files1')"
    @statuschange="handleStatusChange"
  >
    <template slot="desc">
      请选择jpg,jpeg,gif图片，大小在10M以内，最多上传3张图
    </template>
  </veui-uploader>
  <h2>图片上传模式，增加额外操作按钮可以直接输入图片地址</h2>
  <veui-uploader
    v-model="filesExtra"
    type="image"
    name="file"
    action="/upload"
    :max-count="3"
    max-size="10mb"
    accept=".jpg,.jpeg,.gif"
    :payload="payload"
    ui="horizontal"
    progress="bar"
    class="extra-operation"
    @success="onSuccess"
    @failure="onFailure"
    @change="handleChange('filesExtra')"
    @statuschange="handleStatusChange"
  >
    <template slot="desc">
      请选择jpg,jpeg,gif图片，大小在10M以内，最多上传3张图
    </template>
    <template
      slot="extra-operation"
      slot-scope="file"
    >
      <veui-button
        :ref="`add-image${file.index !== undefined ? '-' + file.index : ''}`"
        :ui="file.src ? 'dark' : null"
        class="extra-operation-button"
        @click="openTooltip(file)"
      >
        输入图片地址
      </veui-button>
    </template>
  </veui-uploader>
  <veui-tooltip
    :target="tooltipTarget"
    :open="tooltipOpen"
    trigger="click"
  >
    <div class="extra-url">
      <veui-span>图片地址：</veui-span><veui-input v-model="imageSrc"/>
      <veui-button @click="addImage">
        确定
      </veui-button>
    </div>
  </veui-tooltip>
  <h2>文件上传模式</h2>
  <veui-uploader
    v-model="files2"
    name="file"
    action="/upload"
    :max-count="3"
    max-size="10mb"
    :payload="payload"
    ui="horizontal"
    progress="detail"
    @success="onSuccess"
    @failure="onFailure"
    @change="handleChange('files2')"
    @statuschange="handleStatusChange"
  >
    <template slot="desc">
      请选择文件，大小在10M以内，只能上传3个文件
    </template>
  </veui-uploader>
  <h2>文件上传模式，通过iframe上传</h2>
  <veui-uploader
    ref="iframeUploader"
    v-model="filesIframe"
    name="file"
    action="/uploadiframe"
    request-mode="iframe"
    :max-count="1"
    max-size="10mb"
    accept=".jpg,.jpeg,.gif"
    :payload="payload"
    :convert-response="convertResponse"
    @success="onSuccess"
    @failure="onFailure"
    @change="handleChange('filesIframe')"
    @statuschange="handleStatusChange"
  >
    <template slot="desc">
      请选择jpg,jpeg,gif图片，大小在10M以内，只能上传1张图
    </template>
  </veui-uploader>
</article>
</template>
<script>
import { Uploader, Button, Tooltip, Input, Span } from 'veui'
import ui from 'veui/mixins/ui'

export default {
  name: 'uploader-demo',
  components: {
    'veui-uploader': Uploader,
    'veui-button': Button,
    'veui-tooltip': Tooltip,
    'veui-input': Input,
    'veui-span': Span
  },
  mixins: [ui],
  data: function () {
    let files = [
      {
        name: 'demo-file1.jpg',
        src: 'https://www.baidu.com/img/bd_logo1.png',
        extraInfo: 123
      },
      {
        name: 'demo-file2.gif',
        src: 'http://nodejs.cn/static/images/logo.svg',
        extraInfo: 128
      }
    ]

    return {
      files,
      files1: files.slice(0),
      files2: files.slice(0),
      filesExtra: files.slice(0),
      filesIframe: {
        name: 'demo-file.txt',
        src: 'http://www.baidu.com'
      },
      payload: {
        year: '2017',
        month: '4'
      },
      currentImage: null,
      imageSrc: null,
      tooltipTarget: null,
      tooltipOpen: false,
      upload: (file, {onload, onprogress, onerror}) => {
        // onload(file: Object, data: Object)
        // onprogress(file: Object, progress: Object({loaded, total}))
        // onerror(file: Object, error: Object({reason}))
        let xhr = new XMLHttpRequest()
        file.xhr = xhr

        xhr.upload.onprogress = e => onprogress(file, e)
        xhr.onload = () => onload(file, JSON.parse(xhr.responseText))
        xhr.onerror = e => onerror(file, e)
        let formData = new FormData()
        formData.append('file', file)

        xhr.open('POST', '/upload', true)
        xhr.send(formData)
      }
    }
  },
  methods: {
    onSuccess (data) {
      console.log('Success event: ', data)
    },
    onFailure (data) {
      console.log('Failure event: ', data)
    },
    handleChange (name) {
      console.log('Change event: ', this[name])
    },
    handleStatusChange (status) {
      console.log('Total status is: ', status)
    },
    convertResponse (data) {
      return {
        status: data.code ? 'failure' : 'success',
        ...data.result
      }
    },
    openTooltip (file) {
      this.currentImage = file
      this.tooltipOpen = true
      this.tooltipTarget = `add-image${file.index !== undefined ? '-' + file.index : ''}`
    },
    addImage () {
      if (this.currentImage.index !== undefined) {
        this.$set(this.filesExtra, this.currentImage.index, { src: this.imageSrc })
      } else {
        this.filesExtra.push({ src: this.imageSrc })
      }
      this.currentImage = null
      this.imageSrc = null
      this.tooltipOpen = false
    }
  }
}
</script>

<style lang="less" scoped>
@import "~veui-theme-one/lib.less";

h2 {
  font-size: 16px;
  border-bottom: 1px solid #eee;
  padding-bottom: 10px;
  margin-top: 40px;
}

.extra-operation {
  /deep/ label.veui-button {
    .veui-position-center(50%, 30%) !important;
  }

  /deep/ .veui-button&-button {
    width: 120px;
    padding: 0;
    height: 28px;
    line-height: 26px;
    .veui-position-center(50%, 70%);
  }
}

.extra-url {
  & > * {
    vertical-align: middle;
  }

  .veui-button {
    margin-left: 5px;
  }
}
</style>
