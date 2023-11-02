# 阶段1

vue-loader-plugin 注入了 pitcher

```js
import 'demo.vue'
```

未命中 pitcher-loader 的逻辑，返回 void，忽略 pitch 阶段，进入 normal 阶段。

使用vue-loader直接处理

1. parse 解析源码，得到结构如下：

```js
{
  template: {
    type: "template",
    content: "\n<div>\n  <div><h1>hi</h1></div>\n  <p class=\"abc def\">hi</p>\n  <template v-if=\"ok\"><p class=\"test\">yo</p></template>\n  <svg><template><p></p></template></svg>\n</div>\n",
    start: 756,
    attrs: {
    },
    end: 921,
  },
  script: {
    type: "script",
    content: "//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n//\n\nexport default {\n  data () {\n    return { ok: true }\n  }\n}\n",
    start: 942,
    attrs: {
    },
    end: 1002,
  },
  styles: [
    {
      type: "style",
      content: "\n.test {\n  color: yellow;\n}\n.test:after {\n  content: 'bye!';\n}\nh1 {\n  color: green;\n}\n.anim {\n  animation: color 5s infinite, other 5s;\n}\n.anim-2 {\n  animation-name: color;\n  animation-duration: 5s;\n}\n.anim-3 {\n  animation: 5s color infinite, 5s other;\n}\n.anim-multiple {\n  animation: color 5s infinite, opacity 2s;\n}\n.anim-multiple-2 {\n  animation-name: color, opacity;\n  animation-duration: 5s, 2s;\n}\n\n@keyframes color {\n  from { color: red; }\n  to { color: green; }\n}\n@-webkit-keyframes color {\n  from { color: red; }\n  to { color: green; }\n}\n@keyframes opacity {\n  from { opacity: 0; }\n  to { opacity: 1; }\n}\n@-webkit-keyframes opacity {\n  from { opacity: 0; }\n  to { opacity: 1; }\n}\n.foo p >>> .bar {\n  color: red;\n}\n",
      start: 14,
      attrs: {
        scoped: true,
      },
      scoped: true,
      end: 736,
    },
  ],
  customBlocks: [
  ],
  errors: [
  ],
}
```

2. 因为没有 type 参数，所以不进入 selectBlock 函数处理（这是单独处理 template/script/style 的函数，会在第2次进入 vue-loader 时再使用）

template 代码块存在时，生成以下代码文本String

```js
templateImport = 'import { render, staticRenderFns } from "./scoped-css.vue?vue&type=template&id=0eaf4f08&scoped=true&"'
```

类似的 script，style 代码存在时，生成以下代码文本 String

```js
scriptImport = 'import script from "./scoped-css.vue?vue&type=script&lang=js&"\nexport * from "./scoped-css.vue?vue&type=script&lang=js&"'

stylesCode = 'import style0 from "./scoped-css.vue?vue&type=style&index=0&id=0eaf4f08&scoped=true&lang=css&"\n'
```

3个语句都是为了重新触发 webpack 对 import 的处理

通过第1次loader处理后，因为生成的code，仍然包含 import 语句，并且 .vue 的文件引入依然会使用 vue-loader 处理，分成了 3个 import，通过query参数区分实际逻辑。

第一次转换后，新的 code 如下：

```js
import { render, staticRenderFns } from "./scoped-css.vue?vue&type=template&id=0eaf4f08&scoped=true&"

import script from "./scoped-css.vue?vue&type=script&lang=js&"

export * from "./scoped-css.vue?vue&type=script&lang=js&"
import style0 from "./scoped-css.vue?vue&type=style&index=0&id=0eaf4f08&scoped=true&lang=css&"

/* normalize component */
import normalizer from "!../../lib/runtime/componentNormalizer.js"
var component = normalizer(
  script,
  render,
  staticRenderFns,
  false,
  null,
  "0eaf4f08",
  null
  )
  /* hot reload */
  if (module.hot) {
    var api = require("/Users/xiezhentian/Desktop/github/open-sources/vue-loader/node_modules/vue-hot-reload-api/dist/index.js")
    api.install(require('vue'))
    if (api.compatible) {
      module.hot.accept()
      if (!api.isRecorded('0eaf4f08')) {
        api.createRecord('0eaf4f08', component.options)
      } else {
        api.reload('0eaf4f08', component.options)
      }
      module.hot.accept(
        "./scoped-css.vue?vue&type=template&id=0eaf4f08&scoped=true&",
        function () {
          api.rerender('0eaf4f08', {
            render: render,
            staticRenderFns: staticRenderFns
          })
        })
      }
    }
    component.options.__file = "test/fixtures/scoped-css.vue"
```

执行顺序

```shell
For help, see: https://nodejs.org/en/docs/inspector
Debugger attached.
  console.log lib/plugin-webpack4.js:81
    初始化，注入 Pitcher Loader，

  console.log lib/index.js:36
    Normal Loader: 

  console.log lib/plugin-webpack4.js:86
    根据query.vue参数判定是否符合loader：?vue&type=template&id=0eaf4f08&scoped=true&

  console.log lib/loaders/pitcher.js:53
    Pitcher Loader: /Users/xiezhentian/Desktop/github/open-sources/vue-loader/lib/index.js??vue-loader-options!/Users/xiezhentian/Desktop/github/open-sources/vue-loader/test/fixtures/scoped-css.vue?vue&type=template&id=0eaf4f08&scoped=true&

  console.log lib/plugin-webpack4.js:86
    根据query.vue参数判定是否符合loader：?vue&type=template&id=0eaf4f08&scoped=true&

  console.log lib/index.js:36
    Normal Loader: ?vue&type=template&id=0eaf4f08&scoped=true&

 PASS  test/style.spec.js (5.566s)
```