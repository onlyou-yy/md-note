## mustache模板引擎

历史上曾经出现的数据变为视图的方法

### 纯 DOM 法

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>01_数据变为视图-纯DOM法</title>
</head>

<body>
  <ul id="list"></ul>
  <script>
    var arr = [
      { name: '小明', age: 12, sex: '男' },
      { name: '小红', age: 11, sex: '女' },
      { name: '小强', age: 13, sex: '男' },
    ]
    var list = document.getElementById('list')
    for (let i = 0; i < arr.length; i++) {
      // 每遍历一项，都要用 DOM 方法去创建 li 标签
      let oLi = document.createElement('li')
      // 创建 hd 这个 div
      let hdDiv = document.createElement('div')
      hdDiv.className = 'hd'
      hdDiv.innerText = arr[i].name + '的基本信息'
      oLi.appendChild(hdDiv)
      // 创建 bd 这个 div
      let bdDiv = document.createElement('div')
      bdDiv.className = 'bd'
      // bdDiv.innerText = arr[i].name + '的基本信息'
      // 创建 3 个 p
      let p1 = document.createElement('p')
      p1.innerText = '姓名：' + arr[i].name
      bdDiv.appendChild(p1)
      let p2 = document.createElement('p')
      p2.innerText = '年龄：' + arr[i].age
      bdDiv.appendChild(p2)
      let p3 = document.createElement('p')
      p3.innerText = '性别：' + arr[i].sex
      bdDiv.appendChild(p3)
      oLi.appendChild(bdDiv)
      // 创建的节点是孤儿节点，所以必须要上树才能让用户看见
      list.appendChild(oLi)
    }
  </script>
</body>
</html>
```

### 数组 join 法

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>02_数据变为视图-数组join法</title>
</head>

<body>
  <ul id="list">
  </ul>
  <script>
    var arr = [
      { name: '小明', age: 12, sex: '男' },
      { name: '小红', age: 11, sex: '女' },
      { name: '小强', age: 13, sex: '男' },
    ]
    var list = document.getElementById('list')
    // 遍历 arr 数组，每遍历一项，就以字符串的视角将HTML字符串添加到list中
    for (let i = 0; i < arr.length; i++) {
      list.innerHTML += [
        '<li>',
        '  <div class="hd">' + arr[i].name + '的信息</div>',
        '  <div class="bd">',
        '    <p>姓名：' + arr[i].name + '</p>',
        '    <p>年龄：' + arr[i].age + '</p>',
        '    <p>性别：' + arr[i].sex + '</p>',
        '  </div>',
        '</li>'
      ].join('')
    }
  </script>
</body>
</html>
```

### ES6 的反引号法

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>03_数据变为视图-ES6反引号法</title>
</head>

<body>
  <ul id="list">
  </ul>
  <script>
    var arr = [
      { name: '小明', age: 12, sex: '男' },
      { name: '小红', age: 11, sex: '女' },
      { name: '小强', age: 13, sex: '男' },
    ]
    var list = document.getElementById('list')
    // 遍历 arr 数组，每遍历一项，就以字符串的视角将HTML字符串添加到list中
    for (let i = 0; i < arr.length; i++) {
      list.innerHTML += `
        <li>
          <div class="hd">${arr[i].name}的基本信息</div>
          <div class="bd">
            <p>姓名：${arr[i].name}</p>
            <p>年龄：${arr[i].age}</p>
            <p>性别：${arr[i].sex}</p>
          </div>
        </li>
      `
    }
  </script>
</body>
</html>
```

## mustache 的基本使用

### mustache 库简介

+ mustache 官方 git：https://github.com/janl/mustache.js
+ mustache 是 “胡子” 的意思，因为它的嵌入标记 {{ }} 非常像胡子
+ {{ }} 的语法也被 Vue 沿用
+ mustache 是最早的模板引擎库，比 Vue 诞生的早多了，它的底层实现机理在当时是非常有创造性的、轰动性的，为后续模板引擎的发展提供了崭新的思路

### mustache 的基本使用

+ 必须引入 `mustache` 库，可以在 https://www.bootcdn.cn/ 上找到它
+ `mustache` 的模板语法非常简单，比如前述案例的模板语法如下：

```html
{{#arr}}
  <li>
    <div class="hd">{{name}}的基本信息</div>
    <div class="bd">
      <p>姓名：{{name}}</p>
      <p>年龄：{{age}}</p>
      <p>性别：{{sex}}</p>
    </div>
  </li>
{{/arr}}
```

+ 循环对象数组

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>04_数据变为视图-mustache模板引擎</title>
  <script src="https://cdn.bootcdn.net/ajax/libs/mustache.js/4.1.0/mustache.js"></script>
</head>

<body>
  <div id="container"></div>
  <script>
    var templateStr = `
      <ul id="list">
        {{#arr}}
        <li>
          <div class="hd">{{name}}的基本信息</div>
          <div class="bd">
            <p>姓名：{{name}}</p>
            <p>年龄：{{age}}</p>
            <p>性别：{{sex}}</p>
          </div>
        </li>
        {{/arr}}
      </ul>
    `
    var data = {
      arr: [
        { name: '小明', age: 12, sex: '男' },
        { name: '小红', age: 11, sex: '女' },
        { name: '小强', age: 13, sex: '男' },
      ]
    }
    var domStr = Mustache.render(templateStr, data)
    var container = document.getElementById('container')
    container.innerHTML = domStr
  </script>
</body>
</html>
```

+ 不循环

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>05_数据变为视图-mustache模板引擎-不循环</title>
  <script src="https://cdn.bootcdn.net/ajax/libs/mustache.js/4.1.0/mustache.js"></script>
</head>

<body>
  <div id="container"></div>
  <h1></h1>
  <script>
    var templateStr = `
      <h1>我买了一个{{thing}}，好{{mood}}啊</h1>
    `
    var data = {
      thing: '华为手机',
      mood: '开心'
    }
    var domStr = Mustache.render(templateStr, data)
    var container = document.getElementById('container')
    container.innerHTML = domStr
  </script>
</body>
</html>
```

+ 循环简单数组

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>06_数据变为视图-mustache模板引擎-循环简单数组</title>
  <script src="https://cdn.bootcdn.net/ajax/libs/mustache.js/4.1.0/mustache.js"></script>
</head>

<body>
  <div id="container"></div>
  <h1></h1>
  <script>
    var templateStr = `
      <ul>
        {{#arr}}
          <li>{{.}}</li>
        {{/arr}}  
      </ul>
    `
    var data = {
      arr: ['苹果', '梨子', '香蕉']
    }
    var domStr = Mustache.render(templateStr, data)
    var container = document.getElementById('container')
    container.innerHTML = domStr
  </script>
</body>
</html>
```

+ 数组的嵌套情况

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>07_数据变为视图-mustache模板引擎-数组的嵌套情况</title>
  <script src="https://cdn.bootcdn.net/ajax/libs/mustache.js/4.1.0/mustache.js"></script>
</head>

<body>
  <div id="container"></div>
  <h1></h1>
  <script>
    var templateStr = `
      <ul>
        {{#arr}}
          <li>{{name}}的爱好是：
              <ol>
                {{#hobbies}}
                  <li>{{.}}</li>
                {{/hobbies}}
              </ol>
          </li>
        {{/arr}}  
      </ul>
    `
    var data = {
      arr: [
        { name: '小明', age: 12, hobbies: ['游泳', '羽毛球'] },
        { name: '小红', age: 11, hobbies: ['编程', '写作文', '看报纸'] },
        { name: '小强', age: 13, hobbies: ['打台球'] }
      ]
    }
    var domStr = Mustache.render(templateStr, data)
    var container = document.getElementById('container')
    container.innerHTML = domStr
  </script>
</body>
</html>
```

+ 布尔值

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>08_数据变为视图-mustache模板引擎-布尔值</title>
  <script src="./mustache.js"></script>
</head>

<body>
  <div id="container"></div>
  <h1></h1>
  <script>
    var templateStr = `
    {{#m}}
      <h1>哈哈哈</h1>
    {{/m}}
    `
    var data = {
      m: true
    }
    var domStr = Mustache.render(templateStr, data)
    var container = document.getElementById('container')
    container.innerHTML = domStr
  </script>
</body>
</html>
```

- `script` 模板写法

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>09_数据变为视图-mustache模板引擎-模板写法</title>
  <script src="./mustache.js"></script>
</head>

<body>
  <div id="container"></div>

  <!-- 模板 -->
  <script type="text/template" id="mytemplate">
    <ul id="list">
        {{#arr}}
        <li>
          <div class="hd">{{name}}的基本信息</div>
          <div class="bd">
            <p>姓名：{{name}}</p>
            <p>年龄：{{age}}</p>
            <p>性别：{{sex}}</p>
          </div>
        </li>
        {{/arr}}
      </ul>
  </script>

  <script>
    var templateStr = document.getElementById('mytemplate').innerHTML
    var data = {
      arr: [
        { name: '小明', age: 12, sex: '男' },
        { name: '小红', age: 11, sex: '女' },
        { name: '小强', age: 13, sex: '男' },
      ]
    }
    var domStr = Mustache.render(templateStr, data)
    var container = document.getElementById('container')
    container.innerHTML = domStr
  </script>
</body>
</html>
```

## mustache 的底层核心机理

### `replace()` 方法实现简单地模板数据填充

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>10_正则表达式实现简单的模板数据填充</title>
</head>

<body>
  <div id="container"></div>

  <script>
    var templateStr = '<h1>我买了一个{{thing}}，好{{mood}}</h1>'
    var data = {
      thing: '华为手机',
      mood: '开心'
    }
    // 最简单的模板引擎实现机理，利用的是正则表达式中的 replace() 方法
    // replace() 的第二个参数可以是一个函数，这个函数提供捕获的东西的参数，就是 $1
    // 结合data对象，即可进行智能的替换
    function render(templateStr, data) {
      return templateStr.replace(/\{\{(\w+)\}\}/g, function (findStr, $1) {
        return data[$1]
      })
    }
    var domStr = render(templateStr, data)
    var container = document.getElementById('container')
    container.innerHTML = domStr
  </script>
</body>
</html>
```

### mustache 库的机理

![1630836404730](mustache模板引擎/1630836404730.png)

#### 什么是 `tokens`？

- `tokens` 就是**JS的嵌套数组**，说白了，就是**模板字符串的JS表示**
- **它是“抽象语法书”、“虚拟节点”等等的开山鼻祖**
- 模板字符串`<h1>我买了一个{{thing}}，好{{mood}}啊</h1>`

tokens

```js
[
  ["text", "<h1>我买了一个"],
  ["name", "thing"],
  ["text", "好"],
  ["name", "mood"],
  ["text", "啊</h1>"]
]
```

#### 循环情况下的 `tokens`

当模板字符串中有循环存在时，它将被编译为**嵌套更深**的 `tokens`

模板字符串

```html
<div>
  <ul>
    {{#arr}}
    <li>{{.}}</li>
    {{/arr}}
  </ul>
</div>
```

tokens

```js
[
  ["text", "<div><ul>"],
  [
    "#",
    "arr", 
    [
      ["text", "li"],
      ["name", "."],
      ["text": "</li>"]
    ]
  ],
  ["text", "</ul></div>"]
]

```

#### 双重循环下的 `tokens`

当循环是双重的，那么 `tokens` 会更深一层

模板字符串

```html
<div>
  <ol>
    {{#students}}
    <li>
      学生{{name}}的爱好是
      <ol>
        {{#hobbies}}
        <li>{{.}}</li>
        {{/hobbies}}
      </ol>
    </li>
    {{/students}}
  </ol>
</div>
```

tokens

```js
[
  ["text", "<div><ol>"],
  ["#", "students",
    [
      ["text", "<li>学生"],
      ["name", "name"],
      ["text", "的爱好是<ol>"],
      ["#", "hobbies",
        [
          ["text", "<li>"],
          ["name", "."],
          ["text", "</li>"]
        ]
      ],
      ["text", "</ol></li>"]
    ]
  ],
  ["text", "</ol></div>"]
]
```

#### `mustache` 库底层重点要做2个事情

1. 将模板字符串编译为 `tokens` 形式
2. 将 `tokens` 结合数据，解析为 `dom` 字符串

## 手写实现 mustache 库

`www/index.html`

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <h1>你好!!!</h1>
  <script src="xuni/bundle.js"></script>
  <script>
    var templateStr = '<h1>我买了一个{{thing}}，好{{mood}}啊</h1>'
    var data = {
      thing: '华为手机',
      mood: '开心'
    }
    SGG_TemplateEngine.render(templateStr, data)
  </script>
</body>
</html>
```

`src/index.js`

```js
import Scanner from './Scanner'
window.SGG_TemplateEngine = {
  render(templateStr, data) {
    console.log('render函数被调用，命令Scanner工作')
    // 实例化一个扫描器 构造时候提供一个参数，这个参数就是模板字符串
    // 也就是说这个扫描器就是针对这个模板字符串工作的
    var scanner = new Scanner(templateStr)
    while (scanner.pos !== templateStr.length) {
      var words = scanner.scanUtil('{{')
      console.log(words)
      scanner.scan('{{')
      words = scanner.scanUtil('}}')
      console.log(words)
      scanner.scan('}}')
    }
  }
}
```

`src/Scanner.js`

```js
/*
扫描器类
*/
export default class Scanner {
  constructor(templateStr) {
    // console.log('我是Scanner', templateStr)
    // 将模板字符串写到实例身上
    this.templateStr = templateStr
    // 指针
    this.pos = 0
    // 尾巴，一开始就是模板字符串的原文
    this.tail = templateStr
  }

  // 功能弱，就是走过指定的内容
  scan(tag) {
    if (this.tail.indexOf(tag) === 0) {
      // tag 有多长，比如 {{ 长度是2，就让指针后移几位
      this.pos += tag.length
      // 尾巴也要变，改变尾巴为从当前指针这个字符开始，到最后的全部字符
      this.tail = this.templateStr.substring(this.pos)
    }
  }

  // 让指针进行扫描，直到遇见指定内容结束，并且能够返回结束之前路过的文字
  scanUtil(stopTap) {
    // 记录一下执行本方法的时候pos的值
    const pos_backup = this.pos
    // 当尾巴的开头不是 stopTag的时候，就说明没有扫描到stopTag
    // 写 && 很有必要，防止越界
    while (this.tail.indexOf(stopTap) !== 0 && !this.eos()) {
      this.pos++
      // 改变尾巴为从当前指针这个字符开始，到最后的全部字符
      this.tail = this.templateStr.substr(this.pos)
    }
    return this.templateStr.substring(pos_backup, this.pos)
  }

  // 指针是否已经到头，返回布尔值，end of string
  eos() {
    return this.pos >= this.templateStr.length
  }
}
```

### 生成 `tokens` 数组

`www/index.html` 把测试数据换掉

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <h1>你好!!!</h1>
  <script src="xuni/bundle.js"></script>
  <script>
    var templateStr = `
      <div>
        <ol>
          {{#students}}
          <li>
            学生{{name}}的爱好是
            <ol>
              {{#hobbies}}
              <li>{{.}}</li>
              {{/hobbies}}
            </ol>
          </li>
          {{/students}}
        </ol>
      </div>
    `
    var data = {
      thing: '华为手机',
      mood: '开心'
    }
    SGG_TemplateEngine.render(templateStr, data)
  </script>
</body>
</html>
```

`src/index.js` 实现一个将模板字符串解析成 `tokens` 的方法 `parseTemplateToTokens()`

```js
import parseTemplateToTokens from './parseTemplateToTokens'
window.SGG_TemplateEngine = {
  // 渲染方法
  render(templateStr) {
    // 调用 parseTemplateToTokens 函数，让模板字符串能够变成 tokens 数组
    var tokens = parseTemplateToTokens(templateStr)
    console.log(tokens)
  }
}
```

新建文件 `src/parseTemplateToTokens`

```js
import Scanner from './Scanner'
import nestTokens from './nestTokens'
/**
 * 将模板字符串变为 tokens 数组
 * @param {string} templateStr
 */
export default function parseTemplateToTokens(templateStr) {
  var tokens = []
  // 实例化一个扫描器 构造时候提供一个参数，这个参数就是模板字符串
  var scanner = new Scanner(templateStr)
  var words
  // 让扫描器工作
  while (!scanner.eos()) {
    // 收集开始标记之前的文字
    words = scanner.scanUtil('{{')
    // 存起来
    if (words !== '') tokens.push(['text', words])
    // 过双大括号
    scanner.scan('{{')
    // 收集开始标记之前的文字
    words = scanner.scanUtil('}}')
    // 存起来
    if (words !== '') {
      // 这个 words 就是 {{}} 中间的东西，判断一下首字符
      if (words[0] === '#') {
        // 存起来，从下标为1的项开始存，因为下标为0的项是#
        tokens.push(['#', words.substring(1)])
      } else if (words[0] === '/') {
        // 存起来，从下标为1的项开始存，因为下标为0的项是/
        tokens.push(['/', words.substring(1)])
      } else {
        tokens.push(['name', words])
      }
    }
    // 过双大括号
    scanner.scan('}}')
  }
  // 返回折叠收集的 tokens
  return nestTokens(tokens)
}
```

`src/nestTokens.js` 嵌套 `tokens` 的折叠处理

```js
/**
  * 函数的功能是折叠 tokens，将#和/之间的tokens能够整合起来
  * 作为它的下标为3的项
  * @param {array} tokens
  */
export default function nestTokens(tokens) {
    // 结果数组
    var nestedTokens = []
    // 栈结构，存放小tokens，栈顶（靠近端口的，最新进入的）的tokens数组中当前操作的这个tokens小数组
    var sections = []
    // 收集器，天生指向nestedTokens结果数组，引用类型值，所以指向的是同一个数组
    // 收集器的指向会变化，当遇见#的时候，收集器会指向这个token的下标为2的新数组
    var collector = nestedTokens
    for (let i = 0; i < tokens.length; i++) {
        let token = tokens[i]
        switch (token[0]) {
            case '#':
                // 收集器中放入这个 token
                collector.push(token)
                // 入栈
                sections.push(token)
                // 收集器要换人。给token添加下标为2的项，并且让收集器指向它
                collector = token[2] = []
                break
            case '/':
                // 出栈 pop()会返回刚刚弹出的项
                sections.pop()
                // 改变收集器为栈结构队尾（队尾是栈顶）那项的下标为2的数组
                collector =
                    sections.length > 0 ? sections[sections.length - 1][2] : nestedTokens
                break
            default:
                // 甭管当前的collector是谁，可能是结果nestedTokens，也可能是某个token的下标为2的数组，甭管是谁，推入collector即可
                collector.push(token)
                break
        }
    }
    return nestedTokens
}
```

### 将 `tokens` 结合数据，解析为 `dom` 字符串

+ `#` 标记的 `tokens`，需要递归处理它的下标为2的小数组
+ `www/index.html` 将返回的 `domStr` 添加到 `dom` 树里

```html
  <!DOCTYPE html>
  <html lang="en">

  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
  </head>

  <body>
    <div id="container"></div>
    <script src="xuni/bundle.js"></script>
    <script>
      var templateStr = `
        <div>
          <ol>
            {{#students}}
            <li>
              学生{{name}}的爱好是
              <ol>
                {{#hobbies}}
                <li class='myli'>{{.}}</li>
                {{/hobbies}}
              </ol>
            </li>
            {{/students}}
          </ol>
        </div>
      `
      var data = {
        students: [
          { name: '小明', hobbies: ['游泳', '健身'] },
          { name: '小红', hobbies: ['足球', '篮球', '羽毛球'] },
          { name: '小强', hobbies: ['吃饭', '睡觉'] },
        ]
      }
      var domStr = SGG_TemplateEngine.render(templateStr, data)
      var container = document.getElementById('container')
      container.innerHTML = domStr
    </script>
  </body>
  </html>
```

`src/index.js` 此时，`render()` 函数返回生成的 `domStr`

```js
import parseTemplateToTokens from './parseTemplateToTokens'
import renderTemplate from './renderTemplate'

window.SGG_TemplateEngine = {
  // 渲染方法
  render(templateStr, data) {
    // 调用 parseTemplateToTokens 函数，让模板字符串能够变成 tokens 数组
    var tokens = parseTemplateToTokens(templateStr)
    // 调用 renderTemplate 函数，让 tokens数组变为 dom 字符串
    var domStr = renderTemplate(tokens, data)
    return domStr
  }
}
```

`src/renderTemplate.js` 让 `tokens` 数组变为 `dom` 字符串

```js
import lookup from './lookup'
import parseArray from './parseArray'
/**
 * 让 tokens数组变为 dom 字符串
 * @param {array} tokens
 * @param {object} data
 */
export default function renderTemplate(tokens, data) {
  // 结果字符串
  var resultStr = ''
  // 遍历 tokens
  for (let i = 0; i < tokens.length; i++) {
    let token = tokens[i]
    // 看类型
    if (token[0] === 'text') {
      resultStr += token[1] // 拼起来
    } else if (token[0] === 'name') {
      // 如果是 name 类型，那么就直接使用它的值，当然要用 lookup
      // 防止这里有 “a.b.c” 有逗号的形式
      resultStr += lookup(data, token[1])
    } else if (token[0] === '#') {
      resultStr += parseArray(token, data)
    }
  }
  return resultStr
}
```

`src/parseArray.js` 解析数组及嵌套数组

```js
import lookup from './lookup'
import renderTemplate from './renderTemplate'
/**
 * 处理数组，结合 renderTemplate 实现递归
 * 这个函数接受的参数是token 而不是 tokens
 * token 是什么，就是一个简单的 ['#', 'students', []]
 *
 * 这个函数要递归调用 renderTemplate 函数
 * 调用的次数由 data 的深度决定
 */
export default function parseArray(token, data) {
  // 得到整体数据data中这个数组要使用的部分
  var v = lookup(data, token[1])
  // 结果字符串
  var resultStr = ''
  // 遍历v数组，v一定是数组
  // 遍历数据
  for (let i = 0; i < v.length; i++) {
    // 这里要补一个 “.” 属性的识别
    resultStr += renderTemplate(token[2], v[i])
  }
  return resultStr
}
```

`src/lookup.js`

```js
/**
 * 功能是可以在 dataObj 对象中，用连续点符号的 keyName 属性
 * 比如，dataObj是
 * {
 *    a: {
 *      b: {
 *        c: 100
 *      }
 *    }
 * }
 * 那么 lookup(dataObj, 'a.b.c') 结果就是 100
 * @param {object} dataObj
 * @param {string} keyName
 */
export default function lookup(dataObj, keyName) {
  /*
  // 看看 keyName 中有没有 . 符号
  if (keyName.indexOf('.') !== -1 && keyName !== '.') {
    // 如果有点符号，那么拆开
    var keys = keyName.split('.')
    // 这只一个临时变量，用于周转，一层一层找下去
    var temp = dataObj
    // 每找一层，更新临时变量
    for (let i = 0; i < keys.length; i++) {
      temp = temp[keys[i]]
    }
    return temp
  }
  // 如果这里没有 . 符号
  return dataObj[keyName]
  */
  // 这里其实可以不用加是否包含 . 符号的判断 因为 'abc'.split('.') = ["abc"]
  // 只有一个元素不影响最终结果，不影响循环语句最终结果
  // 另外，这里的特征是：当前的值要依赖前一个的值，所以可以用 reduce 累加器
  // 一行代码搞定
  return keyName !== '.'
    ? keyName
        .split('.')
        .reduce((prevValue, currentKey) => prevValue[currentKey], dataObj)
    : dataObj
}
```

`src/parseTemplateToTokens.js` 添加智能处理空格的逻辑

```js
import Scanner from './Scanner'
import nestTokens from './nestTokens'
/**
 * 将模板字符串变为 tokens 数组
 * @param {string} templateStr
 */
export default function parseTemplateToTokens(templateStr) {
  var tokens = []
  // 实例化一个扫描器 构造时候提供一个参数，这个参数就是模板字符串
  var scanner = new Scanner(templateStr)
  var words
  // 让扫描器工作
  while (!scanner.eos()) {
    // 收集开始标记之前的文字
    words = scanner.scanUtil('{{')
    // 存起来
    if (words !== '') {
      // 尝试写一下去掉空格，智能判断是普通文字的空格，还是标签中的空格
      // 标签中的空格不能去掉，比如 <div class="box"><></div> 不能去掉class前面的空格
      let isInJJH = false
      // 空白字符串
      var _words = ''
      for (let i = 0; i < words.length; i++) {
        // 判断是否在标签里
        if (words[i] === '<') {
          isInJJH = true
        } else if (words[i] === '>') {
          isInJJH = false
        }
        if (!/\s/.test(words[i])) {
          _words += words[i]
        } else {
          // 如果这项是空格，只有当它在标签内的时候，才拼接上
          if (isInJJH) {
            _words += words[i]
          }
        }
      }
      tokens.push(['text', _words])
    }
    // 过双大括号
    scanner.scan('{{')
    // 收集开始标记之前的文字
    words = scanner.scanUtil('}}')
    // 存起来
    if (words !== '') {
      // 这个 words 就是 {{}} 中间的东西，判断一下首字符
      if (words[0] === '#') {
        // 存起来，从下标为1的项开始存，因为下标为0的项是#
        tokens.push(['#', words.substring(1)])
      } else if (words[0] === '/') {
        // 存起来，从下标为1的项开始存，因为下标为0的项是/
        tokens.push(['/', words.substring(1)])
      } else {
        tokens.push(['name', words])
      }
    }
    // 过双大括号
    scanner.scan('}}')
  }
  // 返回折叠收集的 tokens
  return nestTokens(tokens)
}
```



转载自[Vue2.x-mustache模板引擎（笔记）](https://blog.csdn.net/wanghuan1020/article/details/113699725)

源码参考[https://github.com/4xii/Small-knowledge-base/tree/master/Vue%E6%BA%90%E7%A0%81](https://github.com/4xii/Small-knowledge-base/tree/master/Vue%E6%BA%90%E7%A0%81)