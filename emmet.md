# emmet代码生成

## 常规操作

* 生成父子节点 `>`

  ```html
  <!-- ul>li>p{123} -->
  <ul>
    <li>
      <p>123</p>
    </li>
  </ul>
  ```

* 生成兄弟节点 `+`

  ```html
  <!-- p{123}+p{456}+p{789} -->
  <p>123</p>
  <p>456</p>
  <p>789</p>
  ```

* 生成分组 `()`

  ```html
  <!-- (.article>p{123})+(.article>p{456}) -->
  <div class="article">
    <p>123</p>
  </div>
  <div class="article">
    <p>456</p>
  </div>
  ```

* 层级变化 `^`

  ```html
  <!-- div>p>img[src=test.jpg]^p{123}^^div{456} -->
  <div>
    <p><img src="test.jpg" alt=""></p>
    <p>123</p>
  </div>
  <div>456</div>
  ```

* 类名id `.#`

  ```html
  <!-- #container>.header+.main+.footer -->
  <div id="container">
    <div class="header"></div>
    <div class="main"></div>
    <div class="footer"></div>
  </div>
  ```

* 重复 `*`
  
  > `$`生成从1开始的数字，`$$`生成从01开始的数字，`$@-`倒序排列，`$3`从3开始的数字

  ```html
  <!-- ul>li.item_$$*5>p{$@-} -->
  <ul>
    <li class="item_1">
      <p>5</p>
    </li>
    <li class="item_2">
      <p>4</p>
    </li>
    <li class="item_3">
      <p>3</p>
    </li>
    <li class="item_4">
      <p>2</p>
    </li>
    <li class="item_5">
      <p>1</p>
    </li>
  </ul>
  ```

* 属性 `[]`

  ```html
  <!-- el-table[border data]>el-table-column[prop name]*4 -->
  <el-table border="" data="">
    <el-table-column prop="" name=""></el-table-column>
    <el-table-column prop="" name=""></el-table-column>
    <el-table-column prop="" name=""></el-table-column>
    <el-table-column prop="" name=""></el-table-column>
  </el-table>
  ```

* 文本 `{}`

  ```html
  <!-- p>a{click me}+{to continue} -->
  <p><a href="">click me</a>to continue</p>
  ```

## 神器，通过代码片段自动生成代码

> `command+shift+p`输入snippet或者code->首选项->用户代码片段创建对应格式片段

```json
{
  "vue_init": {
    "prefix": "vuec",
    "body": [
      "<template>",
      "\t<div>$0</div>",
      "</template>\r\r",
      "<script>",
      "export default {",
      "\tname: '${1:component_name}',",
      "\tcomponents: {},",
      "\tprops: {},",
      "\tdata() {",
      "\t\treturn {};",
      "\t},",
      "\tcomputed: {},",
      "\twatch: {},",
      "\tcreated() {},",
      "\tmounted() {},",
      "\tmethods: {}",
      "}",
      "</script>\r\r",
      "<style></style>"
    ],
    "description": "Vue component init"
  }
}
```

* 设置prefix用来调用

* body语法

  > 具体代码片段

  * 制表符`$0, $1, $2...`，可通过tab切换，`$0`为最终光标位置
  * 占位符`${1:placeholder}`用来设置`$1`位置默认值，代码生成后会选中占位字符便于更改，占位符可以嵌套`${1:another ${2:placeholder}}`
  * 选择`${1|one,two,three|}`插入代码段并选择占位符后，选项将提示用户选择其中一个值
  * 内置变量，详情见[这里](https://code.visualstudio.com/docs/editor/userdefinedsnippets#_variables)

* snippets转换工具

  [这里](https://github.com/aduck/snippets)