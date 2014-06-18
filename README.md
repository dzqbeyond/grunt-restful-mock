

# grunt-restful-mock

  __  __  ___   ____ _  __
 |  \/  |/ _ \ / ___| |/ /
 | |\/| | | | | |   | ' / 
 | |  | | |_| | |___| . \ 
 |_|  |_|\___/ \____|_|\_\

> 模拟 AJAX 请求返回的 JSON 数据，帮助前端工程师减少对后端接口的依赖，在接口规范的基础之上，实现与后端并行开发。

主要有以下特性：

- 根据数据模板随机生产数据
- 支持 RESTful 风格的 API
- 模拟 HTTPOnly 的 Cookie
- 模拟 HTTP 响应的状态码
- 模拟 HTTP 请求的网络延时

## 开始使用

该插件需要 Grunt `~0.4.5`，如果你还没有使用过 [Grunt](http://gruntjs.com/)，请移步 [Grunt 新手上路](http://gruntjs.com/getting-started)。

熟悉 Grunt 的运作之后，你可以使用如下命令来安装本插件：

```shell
npm install grunt-restful-mock --save-dev
```

安装之后，通过下面代码来加载本插件：

```js
grunt.loadNpmTasks('grunt-restful-mock');
```

## "mock" 任务

### 综述
在你的项目的 Gruntfile 文件中，添加名为 `mock` 的配置节：

```js
grunt.initConfig({
  mock: {
    options: {
      // Task-specific options go here.
    },
    your_target: {
      // Target-specific file lists and/or options go here.
    },
  },
});
```

### 选项（Options）

#### options.protocol
Type: `String`
默认值: `"http"`

HTTP 请求的协议，可选值有：`"http"` 和 `"https"`

#### options.port
Type: `String`
默认值: `'6000'`

端口号

#### options.delay
Type: `Integer`
默认值: `0`

网络延时毫秒数，默认值为 0，收到请求之后立即响应，在处理 AJAX 请求和响应时，可以根据实际需要配置一定的网络延时。

#### options.statusCode
Type: `Integer`
默认值: `200`

响应的状态码，默认值为 200，表示成功的响应。

#### options.debug
Type: `Boolean`
默认值: `false`

默认关闭调试模式，在终端中只显示每次请求的 API 路径；如果打开调试模式（true），将在终端中显示每次请求和响应的详细信息，包括请求的 URL、参数，响应的状态码、Cookie、数据等。

#### options.watch
Type: `Array`
默认值: `[]`

监视的路由文件列表，当文件改动时，自动重启 mock 任务，关于路由文件后面会有详细介绍。

#### options.sensitive、options.strict 和 options.end

这三个选项是 `Path-to-RegExp` 组件的选项，本插件使用的是 `Path-to-RegExp` 来解析 RESTful 的 URL，选项的含义可以[参看这里](https://github.com/component/path-to-regexp#usage)

#### options.cookie
Type: `Object|Array`
默认值: `null`

全局级别的 cookie 设置，在 options 中的 cookie 设置将出现在每个 API 的响应结果中。在某些情况下需要配置全局的 cookie，例如在需要在每次响应后，从 cookie 中取到用户的 ID，就可以在 options 中将该 cookie 配置为全局的。

cookie 中的 options 选项可以[请参考](https://github.com/defunctzombie/node-cookie#more)。

cookie 项可以是一个对象或数组，下面分别看看两种配置：

- 对象格式：

```js
cookie: {
    // 下面是 cookie 名和对应的值
    id: 123,
    name: 'John',

    // cookie 的选项，可选项
    // 注意：这里的设置将用于上面两个 cookie
    // 如果对不同的 cookie 想使用不同的设置，可以使用后面介绍的数组格式
    options:{
        // cookie 的有效期，这里是一小时
        maxAge: 1000 * 60 * 60,
        domain: 'some.com',
        path: '/cookie/path'
    }
}
```
- 数组格式：使用数组格式可以方便将一些 cookie 分类设置，分别设置不同的选项

```js
cookie: [
    {
        id: 123,
        options: {
            maxAge: 1000 * 60 * 60,
            domain: 'some.com',
            path: '/cookie/path'
        }
    },
    {
        name: 'John'
    }
]
```



#### options.route
Type: `Object`
默认值: `null`

定义 API 的路由以及返回数据的模板。

### 数据模板的语法规范

数据模板的语法部分参考了 [mockjson](http://experiments.mennovanslooten.nl/2010/mockjson/) 和 [mockjs](http://mockjs.com/) 的设计，感谢原作者。

**数据模板中的每条数据由三部分构成：属性名、生成规则、属性值**

```js
// 属性名    name
// 生成规则  rule
// 属性值    value
'name|rule': value
```

注意：
  - 属性名 和 生成规则 之间用 `|` 分隔
  - 生成规则 是可选的
  - 生成规则 有 7 种格式：
    
    1. `'name|min-max': value`    
    2. `'name|count': value`
    3. `'name|min-max.dmin-dmax': value`
    4. `'name|min-max.dcount': value`
    5. `'name|count.dmin-dmax': value`
    6. `'name|count.dcount': value`
    7. `'name|+step': value`
  - **生成规则 的 含义 需要依赖 属性值 才能确定**
  - 属性值 中可以含有 `@占位符`
  - 属性值 还指定了最终值的初始值和类型 

#### 生成规则和示例

##### 属性值是字符串 **String**

1. `'name|min-max': 'value'` 通过重复 `'value'` 生成一个字符串，重复次数大于等于 `min`，小于等于 `max`
2. `'name|count': 'value'` 通过重复 `'value'` 生成一个字符串，重复次数等于 `count`

##### 属性值是数字 **Number**

1. `'name|+1': 100` 属性值自动加 `1`，初始值为 `100`
2. `'name|1-100': 100` 生成一个大于等于 `1` 小于等于 `100` 的整数，属性值 `100` 只用来确定类型
3. `'name|1-100.1-10': 100` 生成一个浮点数，整数部分大于等于 `1` 小于等于 `100`，小数部分保留 `1` 到 `10` 位
4. `'name|1-100.2': 100` 生成一个浮点数，整数部分大于等于 `1` 小于等于 `100`，小数部分保留 `2` 位
5. `'name|100.1-10': 100` 生成一个浮点数，整数部分大于等于 `100`，小数部分保留 `1` 到 `10` 位
6. `'name|100.2': 100` 生成一个浮点数，整数部分大于等于 `100`，小数部分保留 `2` 位

  示例：
  ```js
  {
     'number1|1-100.1-10': 1,
     'number2|123.1-10': 1,
     'number3|123.3': 1,
     'number4|123.10': 1.123
  }
  // =>
  {
     "number1": 12.92,
     "number2": 123.51,
     "number3": 123.777,
     "number4": 123.1231091814
  }
  ```

##### 属性值是布尔型 **Boolean**

1. `'name|1': value` 随机生成一个布尔值，值为 `true` 的概率是 `1/2`，值为 `false` 的概率是 `1/2`
2. `'name|min-max': value` 随机生成一个布尔值，值为 `value` 的概率是 `min / (min + max)`，值为 `!value` 的概率是 `max / (min + max)`

##### 属性值是对象 **Object**

1. `'name|min-max': {}` 从属性值 `{}` 中随机选取 `min` 到 `max` 个属性
2. `'name|count': {}` 从属性值 `{}` 中随机选取 `count` 个属性

##### 属性值是数组 **Array**

1. `'name|1': [{}, {} ...]` 从属性值 `[{}, {} ...]` 中随机选取 `1` 个元素，作为最终值
2. `'name|min-max': [{}, {} ...]` 通过重复属性值 `[{}, {} ...]` 生成一个新数组，重复次数大于等于 `min`，小于等于 `max`
3. `'name|count': [{}, {} ...]` 通过重复属性值 `[{}, {} ...]` 生成一个新数组，重复次数为 `count`

##### 属性值是函数 **Function**

`'name': function(){}` 执行函数 `function(){}`，取其返回值作为最终的属性值，上下文为 `'name'` 所在的对象

#### 数据占位符

占位符只是在属性值字符串中占个位置，并不出现在最终的属性值中。占位符的格式为：

```js
name|rule: @占位符 // 没有参数时可以省略括号
name|rule: @占位符(参数 [, 参数])
name|rule: @占位符(参数, @占位符(参数，参数)) // 嵌套使用
```
**注意：**
- 占位符和规则可以同时使用
- 占位符可以嵌套使用
- 属性值的类型由占位符的返回值决定
- 占位符不区分大小写，但是为了明确区分是占位符，推荐采用全大写的方式

##### 内置占位符

- [@int(min, max)](#native-placeholder-1)
- [@integer(min, max)](#native-placeholder-1)
- [@natural(min, max)](#native-placeholder-2)
- [@bool(min, max, cur)](#native-placeholder-3)
- [@boolean(min, max, cur)](#native-placeholder-3)
- [@float(min, max, dMin, dMax)](#native-placeholder-4)
- [@char(pool)](#native-placeholder-5)
- [@character(pool)](#native-placeholder-5)
- [@str(pool, min, max)](#native-placeholder-6)
- [@string(pool, min, max)](#native-placeholder-6)
- [@range(start, stop, step)](#native-placeholder-7)
- [@capitalize(word)](#native-placeholder-8)
- [@upper(str)](#native-placeholder-9)
- [@lower(str)](#native-placeholder-10)
- [@pick(arr)](#native-placeholder-11)
- [@shuffle(arr)](#native-placeholder-12)
- [@randomDate(min, max)](#native-placeholder-13)
- [@formatDate(data, format)](#native-placeholder-14)
- [@parseDate(...)](#native-placeholder-15)
- [@date(date, format)](#native-placeholder-16)
- [@time(date, format)](#native-placeholder-17)
- [@datetime(date, format)](#native-placeholder-18)
- [@now(unit, format)](#native-placeholder-19)
- [@color](#native-placeholder-20)
- [@male_first_name](#native-placeholder-21)
- [@female_first_name](#native-placeholder-22)
- [@last_name](#native-placeholder-23)
- [@name(middleName)](#native-placeholder-24)
- [@word(min, max)](#native-placeholder-25)
- [@sentence(min, max)](#native-placeholder-26)
- [@title(min, max)](#native-placeholder-27)
- [@paragraph(min, max)](#native-placeholder-28)
- [@lorem](#native-placeholder-29)
- [@lorem_ipsum](#native-placeholder-30)
- [@tld](#native-placeholder-31)
- [@domain(tld)](#native-placeholder-32)
- [@email(domain)](#native-placeholder-33)
- [@url](#native-placeholder-34)
- [@ip](#native-placeholder-35)
- [@mobile](#native-placeholder-36)
- [@zip(len)](#native-placeholder-37)
- [@zipcode(len)](#native-placeholder-37)
- [@lang](#native-placeholder-38)
- [@language](#native-placeholder-38)
- [@countryList](#native-placeholder-39)
- [@provinceList](#native-placeholder-40)
- [@randomArea(join)](#native-placeholder-41)
- [@d4](#native-placeholder-42)
- [@d6](#native-placeholder-43)
- [@d8](#native-placeholder-44)
- [@d12](#native-placeholder-45)
- [@d20](#native-placeholder-46)
- [@d50](#native-placeholder-47)
- [@d100](#native-placeholder-48)
- [@guid](#native-placeholder-49)
- [@id](#native-placeholder-50)
- [@inc(step)](#native-placeholder-51)
- [@increment(step)](#native-placeholder-51)
- [@fromData(path, data)](#native-placeholder-52)

<a name="native-placeholder-1" id="native-placeholder-1"></a>
1. @int(min, max) 和 @integer(min, max)

  参数：
  - `min` 可选，缺省值 `-9007199254740992`
  - `max` 可选，缺省值 `9007199254740992`

  在 `min` 和 `max` 之间生成一个随机整数，等价于 `name|min-max: 100`

<a name="native-placeholder-2" id="native-placeholder-2"></a>
2. @natural(min, max)
  
  参数：
  - `min` 可选，缺省值 `0`
  - `max` 可选，缺省值 `9007199254740992`

  在 `min` 和 `max` 之间生成一个随机正整数，等价于 `name|min-max: 100`

<a name="native-placeholder-3" id="native-placeholder-3"></a>
3. @bool(min, max, cur) 和 @boolean(min, max, cur)

  参数：
  - `min` 可选，缺省值 `1`
  - `max` 可选，缺省值 `1`
  - `cur` 可选，省略时将随机产生一个 `bool` 值

  随机生成一个布尔值，值为 `cur` 的概率是 `min / (min + max)`，值为 `!cur` 的概率是 `max / (min + max)`，等价于 `'name|min-max': cur`

<a name="native-placeholder-4" id="native-placeholder-4"></a>
4. @float(min, max, dMin, dMax)

  参数：
  - `min`  可选，缺省值 `-9007199254740992`
  - `max`  可选，缺省值 `9007199254740992`
  - `dMin` 可选，缺省值 `0`
  - `dMax` 可选，缺省值 `17`
  
  生成一个浮点数，整数部分大于等于 `min` 小于等于 `max`，小数部分保留 `dMin` 到 `dMax` 位，等价于 `'name|1-100.1-10': 100`

<a name="native-placeholder-5" id="native-placeholder-5"></a>
5. @char(pool) 和 @character(pool)
  参数
  - `pool` 预设字符，可选，预定义的 pool 有：
    - lower : 'abcdefghijklmnopqrstuvwxyz'
    - upper : 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
    - number: '0123456789'
    - symbol: '!@#$%^&*()[]'
    
    省略时的默认值是上述四种 pool 的组合
  
  从 `pool` 中随机选择一个字符作为返回的字符，使用示例：
  - @char() // 使用默认值
  - @char('lower')
  - @char('upper')
  - @char('number')
  - @char('symbol')
  - @char('alpha') // alpha = lower + upper
  - @char('ABCDefgh') // 自定义的 pool
  
<a name="native-placeholder-6" id="native-placeholder-6"></a>
6. @str(pool, min, max) 和 @string(pool, min, max)

  参数：
  - `pool` 同上
  - `min` 字符串的最小长度，可选，缺省值 `0`
  - `max` 字符串的最大长度，可选，缺省值 `9007199254740992`
  
  从 `pool` 候选字符中，随机生成一个长度在 `min` 到 `max` 之间的字符串
  
<a name="native-placeholder-7" id="native-placeholder-7"></a>
7. @range(start, stop, step)

  参数：
  - `start` 开始位置，可选，缺省值 `0`
  - `stop`  结束位置，必选
  - `step`  自增步长，可选，缺省值 `1`
  
  从 `start` 开始，每次自增 `step`，直到 `stop` 结束，生成一个数组
 
<a name="native-placeholder-8" id="native-placeholder-8"></a>
8. @capitalize(word)

  参数：
  - `word` 必选 
  
  将 `word` 首字母大写

<a name="native-placeholder-9" id="native-placeholder-9"></a>
9. @upper(str)
  
  参数：
   - `str` 必选
   
  将 `str` 中的字符转换为大写字母

<a name="native-placeholder-10" id="native-placeholder-10"></a>
10. @lower(str)
  
  参数：
   - `str` 必选
   
  将 `str` 中的字符转换为小写字母

<a name="native-placeholder-11" id="native-placeholder-11"></a>
11. @pick(arr)

  参数：
  - `arr` 字符串或数组，必选
  
  从数组或字符串中随机返回其中一个字符或数组项

<a name="native-placeholder-12" id="native-placeholder-12"></a>
12. @shuffle(arr)

  参数：
  - `arr` 字符串或数组，必选
  
  随机打乱字符串或数组中的字符或数组项

<a name="native-placeholder-13" id="native-placeholder-13"></a>
13. @randomDate(min, max)

  参数：
  - `min` 最小毫秒数，可选，缺省值 `0`
  - `max` 最大毫秒数，可选，缺省值 `(new Date()).getTime()`
  缺省值
  从 `min` 到 `max` 之间随机产生一个日期
  
<a name="native-placeholder-14" id="native-placeholder-14"></a>
14. @formatDate(data, format)

  参数：
  - `date` 必选，将由 [moment](http://momentjs.com/) 格式化为日期格式，然后调用其 `format` 方法
  - `format` 必选
  
  注意：两个参数具体的传值可以查看 [moment 官网](http://momentjs.com/)

<a name="native-placeholder-15" id="native-placeholder-15"></a>
15. @parseDate(...)

  该占位符直接返回 `moment(arguments)`，所以参数可以参考 [moment 官网](http://momentjs.com/)
  
<a name="native-placeholder-16" id="native-placeholder-16"></a>
16. @date(date, format)

  参数：
  - `date` 可选，缺省时将由 `randomDate` 随机生成一个
  - `format` 可选，缺省值 `'YYYY-MM-DD'`，其他格式参考 [moment 官网](http://momentjs.com/)
  
  格式化日期为指定的日期格式
  
<a name="native-placeholder-17" id="native-placeholder-17"></a>
17. @time(date, format)

  参数：
  - `date` 可选，缺省时将由 `randomDate` 随机生成一个
  - `format` 可选，缺省值 `'HH:mm:ss'`，其他格式参考 [moment 官网](http://momentjs.com/)
  
  格式化日期为指定的时间格式

<a name="native-placeholder-18" id="native-placeholder-18"></a>
18. @datetime(date, format)

  参数：
  - `date` 可选，缺省时将由 `randomDate` 随机生成一个
  - `format` 可选，缺省值 `'YYYY-MM-DD HH:mm:ss'`，其他格式参考 [moment 官网](http://momentjs.com/)
  
  格式化日期为指定的日期时间格式
  
<a name="native-placeholder-19" id="native-placeholder-19"></a>
19. @now(unit, format)

  参数：
  - `unit` 可选，参考 [moment 官网](http://momentjs.com/)
  - `format` 可选，缺省值 `'YYYY-MM-DD HH:mm:ss'`，其他格式参考 [moment 官网](http://momentjs.com/)
  
  按照指定格式返回当前时间

<a name="native-placeholder-20" id="native-placeholder-20"></a>
20. @color

  生成随机颜色值，例如：`'#080900'`

<a name="native-placeholder-21" id="native-placeholder-21"></a>
21. @male_first_name
  
  随机返回一个男性名

<a name="native-placeholder-22" id="native-placeholder-22"></a>
22. @female_first_name
  
  随机返回一个女性名

<a name="native-placeholder-23" id="native-placeholder-23"></a>
23. @last_name

  随机返回一个姓
  
<a name="native-placeholder-24" id="native-placeholder-24"></a>
24. @name(middleName)

  参数：
  - `middleName` 可选
  
  随机返回一个名字

<a name="native-placeholder-25" id="native-placeholder-25"></a>
25. @word(min, max)

  参数：
  - `min` 可选
  - `max` 可选
  
  两个参数都省略时，返回长度为 `3` 到 `7` 之间的一个由随机字符组成的单词
  
  只有一个参数时，返回该参数长度的随机单词
  
  两个参数都没省略时，返回长度为 `min` 到 `max` 之间的一个随机单词
  
<a name="native-placeholder-26" id="native-placeholder-26"></a>
26. @sentence(min, max)

  参数：
  - `min` 可选
  - `max` 可选
  
  两个参数都省略时，返回由 `3` 到 `7` 个随机单词组成的句子（首个单词首字母大写）
  
  只有一个参数时，返回该参数个数的随机单词组成的句子
  
  两个参数都没省略时，返回长度为 `min` 到 `max` 个随机单词组成的句子

<a name="native-placeholder-27" id="native-placeholder-27"></a>
27. @title(min, max)

参数：
  - `min` 可选
  - `max` 可选
  
  两个参数都省略时，返回由 `3` 到 `7` 个随机单词组成的标题（每个单词的首字母大写）
  
  只有一个参数时，返回该参数个数的随机单词组成的标题
  
  两个参数都没省略时，返回长度为 `min` 到 `max` 个随机单词组成的标题

<a name="native-placeholder-28" id="native-placeholder-28"></a>
28. @paragraph(min, max)

参数：
  - `min` 可选
  - `max` 可选
  
  两个参数都省略时，返回由 `3` 到 `7` 个随机句子组成的段落
  
  只有一个参数时，返回该参数个数的随机句子组成的段落
  
  两个参数都没省略时，返回长度为 `min` 到 `max` 个随机句子组成的段落

<a name="native-placeholder-29" id="native-placeholder-29"></a>
29. @lorem

  返回一个 lorem 随机单词

<a name="native-placeholder-30" id="native-placeholder-30"></a>
30. @lorem_ipsum

  返回一个 lorem 随机段落

<a name="native-placeholder-31" id="native-placeholder-31"></a>
31. @tld

  返回一个随机域名后缀 ( com、net、me、org... )

<a name="native-placeholder-32" id="native-placeholder-32"></a>
32. @domain(tld)

  参数
  - `tld` 可选，省略时将随机产生一个域名后缀
  
  返回一个随机域名
  
<a name="native-placeholder-33" id="native-placeholder-33"></a>
33. @email(domain)

  参数
  - `domain` 可选，省略时将随机产生一个域名
  
  返回一个随机邮箱

<a name="native-placeholder-34" id="native-placeholder-34"></a>
34. @url

  随机生成一个 URL
  
<a name="native-placeholder-35" id="native-placeholder-35"></a>
35. @ip

  随机生成一个 IP 地址
  
<a name="native-placeholder-36" id="native-placeholder-36"></a>
36. @mobile

  随机生成一个大陆的手机号码
  
<a name="native-placeholder-37" id="native-placeholder-37"></a>
37. @zip(len) 和 @zipcode(len)

  参数：
  - `len` 可选，邮政编码的长度，缺省值为 `6`
  
  随机生成一个邮政编码

<a name="native-placeholder-38" id="native-placeholder-38"></a>
38. @lang 和 @language

  随机返回一个语言的名称

<a name="native-placeholder-39" id="native-placeholder-39"></a>
39. @countryList

  返回国家数组
  
<a name="native-placeholder-40" id="native-placeholder-40"></a>
40. @provinceList

  返回中国的省份数组  

<a name="native-placeholder-41" id="native-placeholder-41"></a>
41. @randomArea(join)

  参数：
  - `join` 分隔符，可选，缺省为 `'-'`

  返回一个随机的省市区所代表的地址

<a name="native-placeholder-42" id="native-placeholder-42"></a>
42. @d4

  返回 `1` 到 `4` 之间的随机数

<a name="native-placeholder-43" id="native-placeholder-43"></a>
43. @d6

  返回 `1` 到 `6` 之间的随机数
  
<a name="native-placeholder-44" id="native-placeholder-44"></a>
44. @d8

  返回 `1` 到 `8` 之间的随机数

<a name="native-placeholder-45" id="native-placeholder-45"></a>
45. @d12

  返回 `1` 到 `12` 之间的随机数

<a name="native-placeholder-46" id="native-placeholder-46"></a>
46. @d20

  返回 `1` 到 `20` 之间的随机数

<a name="native-placeholder-47" id="native-placeholder-47"></a>
47. @d50

  返回 `1` 到 `50` 之间的随机数

<a name="native-placeholder-48" id="native-placeholder-48"></a>
48. @d100

  返回 `1` 到 `100` 之间的随机数
  
<a name="native-placeholder-49" id="native-placeholder-49"></a>
49. @guid

  随机生成一个 GUID，不重复

<a name="native-placeholder-50" id="native-placeholder-50"></a>
50. @id

  随机生成一个 ID，一般情况不会重复

<a name="native-placeholder-51" id="native-placeholder-51"></a>
51. @inc(step) 和 @increment(step)

  参数：
  - `step` 自增步长，缺省为 `1`
  
   返回从 `0` 开始的自增数，每次调用自增 `step`

<a name="native-placeholder-52" id="native-placeholder-52"></a>
52. @fromData(path, data)

  参数：
  - `path` 数据的路径，必选，例如 `'address.province'` 表示返回 `data.address.province` 的值
  - `data` 数据对象，可选，缺省为 HTTP 请求的参数

  返回 `data` 中的指定数据


### 使用示例

#### 基本使用示例

在 options 选项中定义 route 配置。


```js
grunt.initConfig({
  mock: {
      your_target: {
          options: {
              // 定义路由规则
              route: {
                 // API 的路径
                 '/path/to/API': {
                    // 处理 get 请求
                    get: {
                        // 这里我定义该 API 的 get 请求延时 500ms
                        delay: 500,
                        // 返回的 cookie
                        cookie: {
                            // 返回 cookie 的键值
                            id: 123,
                            username: 'John',
                            // cookie 的选项，该选项将应用于以上的 cookie
                            options:{
                                // cookie 的有效期，这里是一小时
                                maxAge: 1000 * 60 * 60,

                            }
                        },
                        // 返回的数据
                        data: {
                            code: 200,
                            username: 'John',
                            email: 'John@company.com'
                        }
                    },
                    post: {

                        cookie:{
                        },
                        data:{
                        }
                    },
                    delete:{
                    },
                 }
              }
          }
      }
  }
});
```

#### Custom Options
In this example, custom options are used to do something else with whatever else. So if the `testing` file has the content `Testing` and the `123` file had the content `1 2 3`, the generated result in this case would be `Testing: 1 2 3 !!!`

```js
grunt.initConfig({
  restful_mock: {
    options: {
      separator: ': ',
      punctuation: ' !!!',
    },
    files: {
      'dest/default_options': ['src/testing', 'src/123'],
    },
  },
});
```
