<template>
  <div id="app">
    <xhz-tinymce
        id="myEdit"
        ref="editor"
        v-model="msg"
        :defIconSet="set"
        :disabled="disabled"
        @onClick="onClick"
        :upload-method="uploadEditorPasteImg"
    />
  </div>
</template>

<script>
import XhzTinymce from './components/XhzTinymce.vue'

export default {
  name: 'App',
  components: {
    XhzTinymce
  },
  data(){
    return {
      msg: '',
      disabled: false,
      set: [
        {
          name: 'myImage',
          event: {
            // 按钮，名
            text: '图片上传',
            // 是否显示图标
            icon: 'image',
            type: 'button',
            //onclick事件
            onAction: function () {
              this.onEditUploadImg()
            },
          }
        },
        {
          name: 'myVideo',
          event: {
            // 按钮，名
            text: '视频上传',
            // 是否显示图标
            icon: 'embed',
            type: 'button',
            //onclick事件
            onAction: function () {
              this.onEditUploadVideo()
            },
          }
        }
      ]
    }
  },
  methods: {
    onEditUploadImg() {
      const input = document.createElement('input')
      input.type = 'file'
      input.accept = '.png,.jpg'
      input.click()
      input.addEventListener('change', () => {
        if (input.files[0].size / 1024 / 1024 < 2) {
          if (input.files[0].type.indexOf('image') !== -1) {
            console.log(11111)
          }
        }
      })
    },
    onEditUploadVideo(){
      const input = document.createElement('input')
      input.type = 'file'
      input.accept = '.mp4,.rmvb'
      input.click()
      input.addEventListener('change', () => {
        if (input.files[0].size / 1024 / 1024 < 500) {
          if (input.files[0].type.indexOf('image') !== -1) {
            console.log(2222)
          }
        }
      })
    },
    uploadEditorPasteImg(file) {
      console.log(file)
      let image =
          '<img src=' +
          'http://lyj.gxzf.gov.cn/xwzx/xxkb/W020210906584424620138.jpg' +
          ' style="max-width: 100%" />'
      return image
    },
    // 内容
    getContent () {
      console.log('内容', this.msg)
    },
    // 鼠标单击的事件
    onClick (e, editor) {
      console.log('Element clicked')
      console.log(e)
      console.log(editor)
    },
    // 清空内容
    clear () {
      console.log(this.$refs.editor)
      this.$refs.editor.clear()
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
