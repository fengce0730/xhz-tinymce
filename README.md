# xhz-tinymce

![vue](https://img.shields.io/github/package-json/dependency-version/lpreterite/vue-tinymce/vue)
![tinymce](https://img.shields.io/github/package-json/dependency-version/lpreterite/vue-tinymce/tinymce)

提供给 xhz 前端开发者使用的 TinyMCE 组件（注：仅适用于vue-cli构建的项目）

## 目的

主要为了解决自定义图片上传以及复制粘贴时不保留格式的情况，适配当前业务场景。

## 如何使用

### 安装组件

```sh
yarn add xhz-tinymce
# or
npm install xhz-tinymce
```

### 引入

执行完安装命令后，将xhz-tinymce/public目录下的tinymce文件夹复制到项目public文件夹中，可使用如下简易命令：

```
cp -r node_modules/xhz-tinymce/public/tinymce/ public/
```

复制之后在index.html引入相关js

```
<script src="tinymce/tinymce.min.js"></script>
```

在页面中js引入：

```
import XhzTinymce from 'xhz-tinymce'

export default {
    name: 'Index',
    components: {
        XhzTinymce,
    },
}
```

引入标签：

```
<xhz-tinymce id="myEdit" ref="editor" v-model="msg" />
```



## 属性

| 名称            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| `:value`        | 类型`String`，作为文本内容传入编辑器，可以使用`v-model`实现双向绑定 |
| `:baseUrl`      | 基本路径，默认为空根目录，如果你的项目发布后的地址为目录形式，即abc.com/tinymce，baseUrl需要配置成tinymce，不然发布后资源会找不到 |
| `:disabled`     | 富文本编辑器禁用状态                                                     |
| `:plugin`       | 有默认值，常规情况下无需更改 |
| `:toolbar`      | 类型`String`、`Array`，作为string时编辑栏为一行显示，作为array时为多行显示。有默认值，常规情况下无需更改 |
| `:defIconSet`   | 类型`String`、`Array`，自定义按钮配置，可用于图片上传和其他自定义操作 |
| `:options`      | 类型`Object`，编辑器的配置项，默认为空，可根据tinymce文档添加新配置 |
| `:uploadMethod` | 类型`Function`,  粘贴复杂文本时对图片进行处理的方法，项目中app.vue中有相关代码示例。 |

## 更多使用细节

想了解更多内容请联系作者。

- 邮箱：fengce@gxxhz.com
- qq：1048438791
