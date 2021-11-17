<template>
  <div class="tinymce-box">
    <editor
        v-model="myValue"
        :init="setting"
        :disabled="disabled"
        @click="onClick">
    </editor>
  </div>
</template>

<script>
// 文档 http://tinymce.ax-z.cn/
// 引入组件
import tinymce from 'tinymce/tinymce' // tinymce默认hidden，不引入不显示
import Editor from '@tinymce/tinymce-vue'

// 引入富文本编辑器主题的js和css
import 'tinymce/skins/content/default/content.css'
import 'tinymce/themes/silver/theme.min.js'
import 'tinymce/icons/default/icons' // 解决了icons.js 报错Unexpected token '<'

// 编辑器插件plugins
// 更多插件参考：https://www.tiny.cloud/docs/plugins/
import 'tinymce/plugins/image'// 插入上传图片插件
import 'tinymce/plugins/media'// 插入视频插件
import 'tinymce/plugins/table'// 插入表格插件
import 'tinymce/plugins/lists'// 列表插件
import 'tinymce/plugins/wordcount'// 字数统计插件
import 'tinymce/plugins/link'
import 'tinymce/plugins/code'
import 'tinymce/plugins/preview'
import 'tinymce/plugins/fullscreen'
import 'tinymce/plugins/help'
import 'tinymce/plugins/paste'
export default {
  components: {
    Editor
  },
  name: 'Tinymce',
  props: {
    // 默认的富文本内容
    value: {
      type: String,
      default: ''
    },
    // 基本路径，默认为空根目录，如果你的项目发布后的地址为目录形式，
    // 即abc.com/tinymce，baseUrl需要配置成tinymce，不然发布后资源会找不到
    baseUrl: {
      type: String,
      default: window.location.origin ? window.location.origin : ''
    },
    // 禁用
    disabled: {
      type: Boolean,
      default: false
    },
    plugins: {
      type: [String, Array],
      default: 'link lists image code table wordcount media preview fullscreen help powerpaste'
    },
    toolbar: {
      type: Array,
      default:()=>{
        return [
            'bold italic underline strikethrough | fontsizeselect | formatselect | forecolor backcolor | alignleft aligncenter alignright alignjustify | bullist numlist outdent indent blockquote lineheight | undo redo ',
          ' link unlink code lists table | removeformat | fullscreen preview |'
        ]
      }
    },
    defIconSet: {
      type: [Object,Array],
      default: () => {
        return undefined
      }
    },
    options: {
      type: Object,
      default: ()=>{
        return {}
      }
    },
    uploadMethod: {
      type: Function,
      default: ()=>{}
    }
  },
  data () {
    return {
      init: {
        language_url: `${this.baseUrl}/tinymce/langs/zh_CN.js`,
        language: 'zh_CN',
        skin_url: `${this.baseUrl}/tinymce/skins/ui/oxide`,
        // skin_url: 'tinymce/skins/ui/oxide-dark', // 暗色系
        convert_urls: false,
        height: 300,
        // content_css（为编辑区指定css文件）,加上就不显示字数统计了
        // content_css: `${this.baseUrl}tinymce/skins/content/default/content.css`,
        // （指定需加载的插件）
        plugins: this.plugins,
        toolbar: this.editorToolbar, // （自定义工具栏）

        statusbar: true, // 底部的状态栏
        menubar: 'file edit insert view format table tools help', // （1级菜单）最上方的菜单
        branding: false, // （隐藏右下角技术支持）水印“Powered by TinyMCE”
        paste_preprocess: (plugin, args) => {
          this.onEditorPaste(plugin, args)
        },
      },
      myValue: this.value
    }
  },
  computed: {
    setting() {
      let set = Object.assign(this.init,this.options)
      set.toolbar = this.editorToolbar
      set.setup = (editor) => { this.setUp(editor) }
      return set
    },
    editorToolbar() {
      if(this.defIconSet) {
        let toolbar = this.toolbar
        if(this.defIconSet instanceof Array) {
          let defIconName = ''
          this.defIconSet.forEach(item=>{
            defIconName += `${item.name} `
          })
          toolbar.push(defIconName)
          return toolbar
        } else if(this.defIconSet instanceof Object) {
          let length = toolbar.length - 1
          toolbar[length] += ` ${this.defIconSet.name}`
          return toolbar
        }
      }
      return this.toolbar
    },
  },
  mounted () {
    tinymce.init({})
  },
  methods: {
    // 添加相关的事件，可用的事件参照文档=> https://github.com/tinymce/tinymce-vue => All available events
    // 需要什么事件可以自己增加
    onClick (e) {
      this.$emit('onClick', e, tinymce)
    },
    // 可以添加一些自己的自定义事件，如清空内容
    clear () {
      this.myValue = ''
    },
    setUp(editor) {
      this.editor = editor
      if(this.defIconSet) {
        try {
          if(this.defIconSet instanceof Array) {
            this.defIconSet.forEach(item=>{
              this.registerEvent(editor,item)
            })
          } else if(this.defIconSet instanceof Object) {
            this.registerEvent(editor,this.defIconSet)
          } else {
            throw new Error('无效传值')
          }
        } catch (e) {
          throw new Error(e)
        }
      }
    },
    registerEvent(editor,set) {
      editor.ui.registry.addButton(set.name, set.event)
    },
    onEditorPaste(plugin,args) {
      let content = args.content
      if (args.content) {
        let _this = this
        let imgReg = /<img.*?(?:>|\/>)/gi //匹配图片中的img标签
        let srcReg = /src=['"]?([^'"]*)['"]?/i // 匹配图片中的src
        let str = content
        let arr = str.match(imgReg) //筛选出所有的img
        if (arr && arr.length !== 0) {
          args.content = ''
          let count = 0
          arr.forEach(async (item) => {
            let src = item.match(srcReg)
            if (src[1].indexOf('blob') !== -1) {
              const base64Data = await this.urlToBase64(src[1])
              const blob = await this.dataURLtoBlob(base64Data)
              const file = await this.blobToFile(blob, this.uuid(16) + '.jpg')
              const img = await this.uploadMethod(file)
              if (img) {
                count++
                content = content.replace(item.toString(), img)
                if (count === arr.length) {
                  _this.editor.insertContent(content)
                }
              }
            } else {
              args.content = content
            }
          })
        }
      }
    },
    uuid(length = 32) {
      const num = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890'
      let str = ''
      for (let i = 0; i < length; i++) {
        str += num.charAt(Math.floor(Math.random() * num.length))
      }
      return str
    },
    blobToFile(theBlob, fileName) {
      let files = new window.File([theBlob], fileName, { type: theBlob.type })
      return files
    },
    dataURLtoBlob(dataUrl) {
      let arr = dataUrl.split(','),
          mime = arr[0].match(/:(.*?);/)[1],
          bstr = atob(arr[1]),
          n = bstr.length,
          u8arr = new Uint8Array(n)
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n)
      }
      return new Blob([u8arr], { type: mime })
    },
    urlToBase64(url) {
      return new Promise((resolve, reject) => {
        let image = new Image()
        image.onload = function () {
          let canvas = document.createElement('canvas')
          canvas.width = this.naturalWidth
          canvas.height = this.naturalHeight
          // 将图片插入画布并开始绘制
          canvas.getContext('2d').drawImage(image, 0, 0)
          // result
          let result = canvas.toDataURL('image/png')
          resolve(result)
        }
        // CORS 策略，会存在跨域问题https://stackoverflow.com/questions/20424279/canvas-todataurl-securityerror
        image.setAttribute('crossOrigin', 'Anonymous')
        image.src = url
        // 图片加载失败的错误处理
        image.onerror = () => {
          reject(new Error('urlToBase64 error'))
        }
      })
    }
  },
  watch: {
    value (newValue) {
      this.myValue = newValue
    },
    myValue (newValue) {
      this.$emit('input', newValue)
    }
  }
}

</script>
<style scoped>

</style>
