best practice
=============

### project

- 目录结构

**NOTE** : _目录结构可以根据情况变更, `Makefile`, `public` etc;
     
        ├── .dockerignore   // build docker images 时 ignore 文件
        ├── .editorconfig   // 编辑器格式统一工具(editorconfig) 配置文件
        ├── .env            // 环境变量 require('dotenv').load()加载
        ├── .git            // Git repo 文件件
        ├── .gitignore   
        ├── .npmrc          // 项目 npm 配置文件
        ├── Dockerfile      // docker image
        ├── Jenkinsfile     // jenkins pipeline file
        ├── package.json    // package.json
        ├── test            // 测试目录
        │   ├── data
        ├── src             // 项目源代码
        │   ├── app
        │   │   └── app.js
        │   ├── middleware
        │   │   └── redis-cache.js
        │   ├── models
        │   │   ├── comments.js
        │   │   ├── groups.js
        │   │   └── users.js
        │   │   └── index.js
        │   ├── controllers
        │   │   ├── comments.js
        │   │   ├── groups.js
        │   │   └── users.js
        │   │   └── index.js
        │   ├── utils
        │   │   └── logger.js
        │   │   └── index.js
        │   ├── config
        │   │   └── config.js
        │   │   └── config.production.js
        │   │   └── config.development.js
        │   └── www         // 项目运行主文件 
        │   ├── server.js
        └── yarn.lock       // yarn 工具产生的 package lock


- 编辑器控制编码风格

工具(*`editorconfig`*), 配置文件(*`.editorconfig`*) 确保在代码编辑的过程中 `style` 的统一, 配置如下:

```ini
root = true

[*]
charset = utf-8
insert_final_newline = true
trim_trailing_whitespace = true

[*.{json,js}]
indent_size = 2
indent_style = space
```


- 合理利用`npm`和 `package.json`

    * `npmrc`

    ```ini
    registry=https://registry.npm.taobao.org
    save=true
    save-exact=true
    ...
    ```

    * 充分利用 `package.json` + `npm` 

    ```javascript
    {
      "name": "backend",
      "version": "1.0.0",
      "main": "index.js",
      "private": true,
      "license": "MIT",
      "scripts": {
        "fix": "standard --fix",
        "precommit": "npm fix && npm test",
        "prepush": "npm test",
        "test": "mocha",
        "ddp": "npm ddp"
      },
      "dependencies": {
        "bookshelf": "^0.10.4",
        "dotenv": "4.0.0",
        "knex": "^0.13.0",
        "pg": "^7.3.0",
        "restify": "^6.0.1"
      },
      "devDependencies": {
        "babel-preset-env": "^1.6.0",
        "chai": "^4.1.2",
        "nodemon": "1.12.1",
        "standard": "10.0.3",
        "supertest": "^3.0.0",
        "husky": "0.14.3"
      },
      "nodemonConfig": {
        "ignore": [
          "test/*.js",
          "node_modules"
        ]
      }
    }
    ```

- 减少 `docker image` 大小

    * use alpine linux as base image
    * set docker env NODE_ENV=production
    * run `npm ddp` command before build docker images
    * docker run `rm -f /var/cache/apk/*`

- config: 通过 NODE_ENV 来 merge 各个环境下的 config Context

- JavaScript文件使用无BOM的UTF-8编码

- 行尾符强制使用LR

- 文件名采用test-my-app.js这样的命名方式。
    ​


### 代码异常处理

- 类`connectjs`的 web framework 的错误需要统一为 `return next(err)`处理
- request response 的错误处理需要单独重新继承 Error 类型, Error 中需要包换 errcode, message
- errcode 和 message 需要在单独文件中进行 mapping 声明; 方便查询

例如:

```javascript
module.exports = ClientErrorMapping = {
    E4001: 'xxx 错误',
    E4002: 'xxx 错误',
    E4003: 'xxx 错误',
    E4004: 'xxx 错误'
};
```

- 后端产品代码中 __不能__ 出现`console.log()`, 调试时使用 `debug` 模块替代, log 使用 log 库代替; 




###**以下部分为 Standard JS 之外的扩展：**

### style guide

- node 版本: node version 使用 lastest `LTS` 版本, 目前为 `6.11.4`, 使用`nvm` 作为`node version` 控制工具;

- code lint 使用 `standard.js`;减少 style guide config 的分歧;
  eslint, jslint 配置要花不少功夫;

  ​
#### code style(以下关键字或是与语言环境均指 ES6

- 使用 immutablejs 控制数据对象的改变;

ex:

```javascript
const { Map } = require('immutable')
const map1 = Map({ a: 1, b: 2, c: 3 })
const map2 = map1.set('b', 50)
map1.get('b') + " vs. " + map2.get('b') // 2 vs. 50
```




- 所用文件头中使用`use strict` 标注严格模式;
```javascript
'use strict'
```

- 使用`let` 或是 `const` 代替 `var` 声明变量, 全程禁止使用`var`;


- 常量: 使用 `const`标识, 全局变量名称需要用全大写，局部变量继续遵循 camel;
```javascript
const PROJECT_NAME = 'oraclize'
```

- 类型判断优先使用`Object.prototype.toString`:

```js
const toString = Object.prototype.toString;

function isString (str) {
  return toString.call(str) === '[object String]'
}
```
- 使用`===` 代替 `==`; `null`,`undefined`, `NaN`判断除外, `NaN` 使用 `isNaN(val)` 判断, `null` 和`undefined` 使用 `==` 判断;


- `true` 或是 `false` 的判断, 请用 `===` 判断, ex:
```javascript
if (a === true) { doSomting()}
```

- 使用单引号不使用双引号, `object` 中的属性不使用引号;

- require 赋值时尽量使用`match pattern`, 且使用`const`标识;
```javascript
const {join} = require('path')
let joinPath = join(__dirname, 'public')
```

- 每行语句不得超过 *80* 字符; 

- 语句块或是函数的左大括号不换行, 右大括号需换行;
```javascript
function test() {
  debug('this is test')
}
```

- 只在确认为同步的代码中使用`try{}..catch(){}`, 否则用`event` 或是 `callback` 处理并且error必须处理;

**bad**
```javascript

try{
  let content = fs.readFile('text.txt')
}catch(e) {
  
}
```

- 使用 ES6中的`class`, 中替换 ES5中的 constrctor function, __NOTE__: 继承时不要忘记`static property`;

```javascript
const Utils = require('lodash')
class B {}
class A extends B {}
Utils.extends(A, B) // static 继承
```

- 在 ES6 中 `this` 之前 先调用`super()`;

- *禁止* 使用`eval()`, `with`;

- node.js 中使用 `process.nextTick()` 代替 `setTimeout(doSomting, 0)`

- 多行字符串请使用 es6 字符串模板;

- 构造`Array` 或是 `Object` 直接使用 `let arr = []` 或是 `let obj = {}` 或是 `let obj = Object.create(null)` , 禁止使用构造函数;

- 禁止修改内置对象的`prototype`;

- 多行注释, 使用 `/* */` 格式, 单行注释`//`

```js
/*
 * this is commit 
 */
 function doSometing() {
    // somthing variable
    let name = 'welcome'
    debug(name)
 }
```

- 单分支 `if ... else ... ` 使用 `?:` 代替, 三目运算符需要分行, 多分支使用`switch...case..` 代替;

## Error and exception catch

### `try...catch..finally`

```javascript
const str = '{"b:100}'
try {
  const obj = JSON.parse(str)
} catch(err) {
  debug(err.message || err.stack)
}
```

### `Promise catch`

```javascript
const catchRejectError = Promise.reject(new Error('test error'))
catchRejectError.then().catch((err) => {
  debug(err.message || err.stack)
})

```

### `error event`

```javascript
const http  = require('http')
const debug = require('debug')('somthing:xx')

const server = http.createServer((req, res) => {
  res.end('hello, world')
})

const port = parseInt(process.env.PORT || 3000, 10)
const onError = (err) => debug(err.message || err.stack)
server.listen(port)
server.on('error', onError)
```


### `process uncatch error`

```javascript
const noop = function(){}
process.on('uncaughtException', noop)
```


### `error in req-res loop`

```javascript

'use strict'

try{require('dotenv').load()} catch(e) {console.log(e.message)}

import http from 'http'
import express from 'express'
import debug from 'debug'

const toString = Object.prototype.toString;

const ErrMapping = {

  'AUTH_ERR': {
    errCode: 'E4001',
    errMsg: 'auth error'
  },

  'EXISTS_ERR': {
    errCode: 'E4002',
    errMsg: 'source not exists'
  },

  'ACCESS_ERR': {
    errCode: 'E4003',
    errMsg: 'source 访问受限'
  },

  'UNKNOWN_ERR': {
    errCode: 'E5005',
    errMsg: '未知错误'
  }
}

//subclass builtin Error Object
class PreError extends Error {
  constructor(code, message) {
    super(code, message)
    this.name = 'PreError'
    this.code = code
    this.message = message
    const hasCaptureStackTrace = typeof Error.captureStackTrace === 'function'
    if (hasCaptureStackTrace === true) {
      Error.captureStackTrace(this, PreError)
    } else {
      this.stack = new Error(message).stack
    }
  }
}

const getSpecialError = (desc='UNKNOWN_ERR') => {
  const errInfo = ErrMapping[desc];
  return new PreError(errInfo.errCode, errInfo.errMsg)
}

const logger = debug('xinlian:www')

const app = express()

const router = new express.Router()

router.get('/', (req, res, next) => res.json({ version: 'v1', info: process.env.HOME }))

router.get('/auth', (req, res, next) => next(getSpecialError('AUTH_ERR')))
router.get('/exists', (req, res, next) => next(getSpecialError('EXISTS_ERR')))
router.get('/access', (req, res, next) => next(getSpecialError('ACCESS_ERR')))


app.use('/api/v1', router)

app.use((err, req, res, next) => {
  return res.json({ code: err.code, message: err.message })
})


const server = http.createServer(app)

const normalPort = (val) => {
  const port = parseInt(val, 10)
  if (isNaN(port)) return val
  if (port > 0) return port
  return false
}



const port = normalPort(process.env.PORT || 3000)

const onError = (err) => {
  logger(err.message || err.stack)
  process.exit()
}

const onListening = () => {
  logger(server.address().port)
}

server.listen(port)

server.on('error', onError)
server.on('listening', onListening)

const onLoop = (err) => {
  logger(err.message || err.stack)
}

process.on('uncaughtException', onLoop)
```

### `exception retry; example: lookup dns`

```javascript
const dns = require('dns');
const retry = require('retry');
 
function faultTolerantResolve(address, cb) {
  const operation = retry.operation();
  operation.attempt((currentAttempt) => {
    dns.resolve(address, (err, addresses) => {
      if (operation.retry(err)) { return}
      cb(err ? operation.mainError() : null, addresses)
    })
  })
}
 
faultTolerantResolve('nodejs.org', (err, addresses) => {debug(err, addresses)})
```

## code style detail

- 属性不允许加双引号或单引号，也是为了防止关键字或标识符误用的情况

  ```js
  // good
  var obj = {
      a: 1,
      b: 2,
      c: 3
  };
   
  // bad
  var obj = {
      "a" : 1,
      "b" : 2,
      "c" : 3
  };
  ```

- "," 和 ";" 前不允许有空格

- 在函数调用、函数声明、括号表达式、属性访问、if / for / while / switch / catch 等语句中，() 和 [] 内紧贴括号部分不允许有空格

  ```js
  // good
  callFunc(param1, param2, param3);
   
  save(this.list[this.indexes[i]]);
   
  needIncream && (variable += increament);
   
  if (num > list.length) {
  }
   
  while (len--) {
  }

  // bad
  callFunc( param1, param2, param3 );
  save( this.list[ this.indexes[ i ] ] );
  needIncreament && ( variable += increament );
   
  if ( num > list.length ) {
  }
  while ( len-- ) {
  }
  ```

- 单行声明的数组与对象，如果包含元素，{} 和 [] 内紧贴括号部分不允许包含空格

  声明包含元素的数组与对象，只有当内部元素的形式较为简单时，才允许写在一行。元素复杂的情况，还是应该换行书写。

  ```js
  // good
  var arr1 = [];
  var arr2 = [1, 2, 3];
  var obj1 = {};
  var obj2 = {name: 'obj'};
  var obj3 = {
      name: 'obj',
      age: 20,
      sex: 1
  };
   
  // bad
  var arr1 = [ ];
  var arr2 = [ 1, 2, 3 ];
  var obj1 = { };
  var obj2 = { name: 'obj' };
  var obj3 = {name: 'obj', age: 20, sex: 1};
  ```

- 行尾不得有多余的空格

- 长字符串换行用 + 连接符换行书写，或用 ，禁止使用 \，因为可能产生各种错误，参见[Google规范](https://google.github.io/styleguide/javascriptguide.xml?showone=Multiline_string_literals#Multiline_string_literals)

- 运算符处换行时，运算符必须在新行的行首

  ```js
  // good
  if (user.isAuthenticated()
      && user.isInRole('admin')
      && user.hasAuthority('add-admin')
      || user.hasAuthority('delete-admin')
  ) {
      // Code
  }
   
  var result = number1 + number2 + number3
      + number4 + number5;

  // bad
  if (user.isAuthenticated() &&
      user.isInRole('admin') &&
      user.hasAuthority('add-admin') ||
      user.hasAuthority('delete-admin')) {
      // Code
  }

  var result = number1 + number2 + number3 +
      number4 + number5; 
  ```

- 在函数声明、函数表达式、函数调用、对象创建、数组创建、for语句等场景中，不允许 , 或 ; 出现在行首

  ```js
  // good
  var obj = {
      a: 1,
      b: 2,
      c: 3
  };
   
  foo(
      aVeryVeryLongArgument,
      anotherVeryLongArgument,
      callback
  );

  // bad
  var obj = {
      a: 1
      , b: 2
      , c: 3
  };
   
  foo(
      aVeryVeryLongArgument
      , anotherVeryLongArgument
      , callback
  );
  ```

- 不同行为或逻辑的语句集，使用空行隔开，更易阅读

  ```Js
  function setStyle(element, property, value) {
      if (element == null) {
          return;
      }
   
      element.style[property] = value;
  } 
  ```

- 在语句的行长度超过 80 时，根据逻辑条件合理缩进

  ```Js
  // 较复杂的逻辑条件组合，将每个条件独立一行，逻辑运算符放置在行首进行分隔，或将部分逻辑按逻辑组合进行分隔。
  // 建议最终将右括号 ) 与左大括号 { 放在独立一行，保证与 `if` 内语句块能容易视觉辨识。
  if (user.isAuthenticated()
      && user.isInRole('admin')
      && user.hasAuthority('add-admin')
      || user.hasAuthority('delete-admin')
  ) {
      // Code
  }

  // 按一定长度截断字符串，并使用 + 运算符进行连接。
  // 分隔字符串尽量按语义进行，如不要在一个完整的名词中间断开。
  // 特别的，对于 HTML 片段的拼接，通过缩进，保持和 HTML 相同的结构。
  var html = '' // 此处用一个空字符串，以便整个 HTML 片段都在新行严格对齐
      + '<article>'
      +     '<h1>Title here</h1>'
      +     '<p>This is a paragraph</p>'
      +     '<footer>Complete</footer>'
      + '</article>';

  // 也可使用数组来进行拼接，相对 `+` 更容易调整缩进。
  var html = [
      '<article>',
          '<h1>Title here</h1>',
          '<p>This is a paragraph</p>',
          '<footer>Complete</footer>',
      '</article>'
  ];
  html = html.join('');

  // 当参数过多时，将每个参数独立写在一行上，并将结束的右括号 ) 独立一行。
  // 所有参数必须增加一个缩进。
  foo(
      aVeryVeryLongArgument,
      anotherVeryLongArgument,
      callback
  );

  // 也可以按逻辑对参数进行组合，譬如分为“模板”和“数据”两块。
  zDreamer.format(
      dateFormatTemplate,
      year, month, date, hour, minute, second
  );
   
  // 当函数调用时，如果有一个或以上参数跨越多行，应当每一个参数独立一行。
  // 这通常出现在匿名函数或者对象初始化等作为参数时，如setTimeout函数等。
  setTimeout(
      function () {
          alert('hello');
      },
      200
  );
   
  order.data.read(
      'id=' + me.model.id,
      function (data) {
          me.attchToModel(data.result);
          callback();
      },
      300
  );
  // 链式调用较长时采用缩进进行调整。
  $('#items')
      .find('.selected')
      .highlight()
      .end();
   
  // 三元运算符由3部分组成，因此其换行应当根据每个部分的长度不同，形成不同的情况。
  var result = thisIsAVeryVeryLongCondition
      ? resultA : resultB;
   
  var result = condition
      ? thisIsAVeryVeryLongResult
      : resultB; 
   
  // 数组和对象初始化的混用，严格按照每个对象的 `{` 和结束 `}` 在独立一行的风格书写。
  var array = [
  {
    // ...
  },
  {
    // ...
  }];    
  ```

- if / else / for / do / while 语句中，即使只有一行，也不得省略块 {...} 

  ```Js
  // good
  if (condition) {
      callFunc();
  }

  // bad
  if (condition) callFunc()
  if (condition)
      callFunc(); 
  ```

- IIFE 必须在函数表达式外添加 (，非 IIFE 不得在函数表达式外添加 (

  IIFE = Immediately-Invoked Function Expression.

  额外的 ( 能够让代码在阅读的一开始就能判断函数是否立即被调用，进而明白接下来代码的用途。而不是一直拖到底部才恍然大悟。

  ```Js
  // good
  var task = (function () {
    // Code
    return result;
  })();

  // bad
  var task = function () {
    // Code
    return result;}(); 
  ```

- 任何变量禁止命名为类似于 i , j , k , q 这样的无实际意义还易看错的词，必须表意明确且清晰易懂

- 函数可选参数包装成 "**optional**" 命名的 Object

  ```Js
  function hear(theBells, optional) { ... } 
  ```

- 类的方法/属性使用Camel命名法，私有方法/属性名前额外增加"_"

  ```Js
  function TextNode(valueName, engine) {
    this._valueName = valueName;
    this.engine = engine;
  }
   
  TextNode.prototype.clone = function () {
    return this;
  };

  TextNode.prototype._doThing = function () {
    return this;}; 
  ```

- 命名空间使用Camel命名方法

  ```Js
  equipments.heavyWeapons = {}; 
  ```

- 枚举名使用Pascal命名法；枚举变量使用全部字母大写，单词间下划线分隔的命名方式

  ```Js
  var TargetState = {
    READING: 1,
    WAIT_READING: 2
  }; 
  ```

- 由多个单词组成的缩写词，在命名中，根据当前命名法和出现的位置，所有字母的大小写与首字母的大小写保持一致

  ```Js
  function XMLParser() { 
  }

  function insertHTML(element, html) {
  }
  var httpRequest = new HTTPRequest(); 
  ```

- 类名使用名词

- 函数名使用动宾短语

- boolean类型的变量使用is或has开头

- Promise对象用动宾短语的进行时表达

- 变量名、函数名等命名的时候，多用对仗词，如add/remove

- 代码尽量做到自注释

- 对于内部实现、不容易理解的逻辑说明、摘要信息等，我们可能需要编写细节注释，细节注释遵循单行注释的格式

- 有时我们会使用一些特殊标记进行说明。特殊标记必须使用单行注释的形式

  下面列举了一些常用标记：

  - **TODO: 有功能待实现。此时需要对将要实现的功能进行简单说明。**


  - **FIXME: 该处代码运行没问题，但可能由于时间赶或者其他原因，需要修正。此时需要对如何修正进行简单说明。**


  - **HACK: 为修正某些问题而写的不太好或者使用了某些诡异手段的代码。此时需要对思路或诡异手段进行描述。**


  - **WARN: 该处存在陷阱。此时需要对陷阱进行描述。**


  - **NOTE: 该处需要注意。针对非上述几种情况而言的重要。**

- 除开特定情况，所有函数、变量、类、常量、枚举等必须在合理规划的命名空间内

- 严格区分内部命名空间和第三方库的命名空间，禁止任何侵入

- 对于常用或层次很长的第三方库对象或类考虑在函数域或对象中增加别名变量去转接下，且该别名和对象名或类名保持一致

  ```Js
  myApp.main = function () {
      var MyClass = some.long.namespace.MyClass;
      var staticHelper = some.long.namespace.MyClass.staticHelper;
      staticHelper(Object.Create(MyClass));
  };
  ```

- 同时除开通过该别名获取其内部枚举或常量的情况外，禁止任何访问其方法或属性的操作

  ```js
  // good
  myApp.main = function () {
    var Fruit = some.long.namespace.Fruit;
    switch (Fruit) {
      case Fruit.APPLE:
        ...
        case Fruit.BANANA:
          ...
    }
  };

  // bad
  myApp.main = function () {
    var MyClass = some.long.namespace.MyClass;
    MyClass.staticHelper(null);
  };
  ```

- **慎用this：仅在对象构造函数、方法、闭包中使用。**

  this的语义很特别，有时它引用一个全局对象(大多数情况下)， 调用者的作用域(使用 eval 时)，DOM树中的节点(添加事件处理函数时)，新创建的对象(使用一个构造器), 或者其他对象(如果函数被 call()或apply()).使用时很容易出错, 所以只有在下面两个情况时才能使用: 

  - 构造函数中


  - 对象的方法(包括创建的闭包)中

- 特别的，浮点数做相等比较时切记勿用===直接比较，统一使用number.toFixed(10)；转换后再用===比较。

  特殊情况下位数可以调整。

  ```Js
  var num1 = 2.323 - 1.31;
  var num2 = 2.1 + 1.2;
   
  if (num1.toFixed(10) === num2.toFixed(10)) {
  }
  ```

- 使用描述性的条件变量来做判断

  ```Js
  // good
  var isValidPassword = password.length >= 4 && /^(?=.*\d).{4,}$/.test(password);

  if (isValidPassword) {
    console.log('winning');
  }
   
  // bad
  if (password.length >= 4 && /^(?=.*\d).{4,}$/.test(password)) {
    console.log('losing');} 
  ```

- 不要在循环体中包含函数表达式，事先将函数提取到循环体外。
  循环体中的函数表达式，运行过程中会生成循环次数个函数对象。

  ```Js
  // good
  var clicker = function () {
    // ......
  }

  for (var i = 0, len = elements.length; i < len; i++) {
    var element = elements[i];
    addListener(element, 'click', clicker);
  }

  // bad
  for (var i = 0, len = elements.length; i < len; i++) {
    var element = elements[i];
    addListener(element, 'click', function () {});
  } 
  ```

- 对循环内多次使用的不变值，在循环外用变量缓存

  ```js
  // good
  var width = wrap.offsetWidth + 'px';
  for (var i = 0, len = elements.length; i < len; i++) {
    var element = elements[i];
    element.style.width = width;
    // ......
  }

  // bad
  for (var i = 0, len = elements.length; i < len; i++) {
    var element = elements[i];
    element.style.width = wrap.offsetWidth + 'px';
    // ......
  }
  ```

- Boolean、Number和String之间的基本转换统一用Boolean()、Number()、String()这三个转型函数完成。(切勿带上new！)

  注意：Number()遇到字符串中有非数字的字符时会返回NaN，且只支持十进制

  ```Js
  var num = Number('2.534'); var str = String(3.12); var boo = Boolean(2);
  ```

- 对转换成Number有高要求时，浮点数使用parseFloat()，整数使用第二个参数指定进制的parseInt(x, decimalism)

  ```Js
  parseFloat(str); parseInt(str, 10);
  ```

- number去除小数点，使用Math.floor / Math.round / Math.ceil，不使用 parseInt

  ```Js
  // good 
  var num = 3.14; 
  Math.ceil(num);  
  // bad 
  var num = 3.14; 
  parseInt(num, 10);
  ```

- 字符串取某个字符时使用xx.charAt()，而不是xx[]，否则容易取到 undefined

- 鼓励自定义.toString()方法，但一定要保证该方法不会失败或产生任何副作用
  譬如若是调用触发了assert，可assert为了输出正确log会再去调用.toString()，这就导致死循环

- 创建新对象必须使用对象字面量 {} 或Object.Create()，禁止使用new Object()等。

- 属性访问时，尽量使用 . 。仅当涉及到动态属性名访问时用 [x]

- for/in遍历对象时, 使用hasOwnProperty()过滤掉继承过来的属性，最好能用ES5里的新方法 .Keys() 等代替。

- 构造函数调用时必须带上new，且函数名首字母必须大写。

- 使用数组字面量 [] 创建新数组；除非想要创建的是指定长度的数组，否则禁止使用new Array()。

- 遍历数组使用for不使用for/in。
  数组对象可能存在数字以外的属性，这种情况下for/in不会得到正确结果。

- 不因为性能的原因自己实现数组排序功能，尽量使用数组的 sort 方法（**非 ASCII** 需要自己采用 **[String.localeCompare](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare)** 实现排序函数）

  自己实现的常规排序算法，在性能上并不优于数组默认的 sort 方法。以下两种场景可以自己实现排序：

  1. 需要稳定的排序算法，达到严格一致的排序结果。
  2. 数据特点鲜明，适合使用桶排。

- 清空数组使用array.length = 0

- 永远不要将Array用作map/hash/associative等数据结构，它们有自己的专用实现对象。但Array可用作多维数组。
  Array约定禁止使用非整型作为索引值，应防止滥用。

- 数组新增和取值多用"push/pop"函数对。

- 除特殊说明外，所有情况下建议使用函数表达式，而不是函数声明语句。

- 当函数没有函数返回值得时候，要返回this。

- 在有标准函数且其无明显问题的情况下，优先选用标准函数而非第三方库。

- 一个函数的长度控制在50行以内，最长不要超过100行。
  将过多的逻辑单元混在一个大函数中，易导致难以维护。一个清晰易懂的函数应该完成单一的逻辑单元。复杂的操作应进一步抽取，通过函数的调用来体现流程。
  特定算法等不可分割的逻辑允许例外。

- 一个函数的参数控制在3个以内，否则用 Object 包一下。
  除去不定长参数以外，函数具备不同逻辑意义的参数建议控制在3个以内，过多参数会导致维护难度增大。

- 通过"options"对象传递非数据输入型参数

  有些函数的参数并不是作为算法的输入，而是对算法的某些分支条件判断之用，此类参数建议通过一个"options"对象作为参数传递。

  这种方式有几个显著的优势:

  * Boolean型的配置项具备名称，从调用的代码上更易理解其表达的逻辑意义。

  - 当配置项有增长时，无需无休止地增加参数个数，不会出现removeElement(element, true, false, false, 3)这样难以理解的调用代码。


  - 当部分配置参数可选时，多个参数的形式非常难处理重载逻辑，而使用一个对象只需判断属性是否存在，实现得以简化。

  ```Js
  function removeElement(targetElem, options) {
      targetElem.parent.removeChild(targetElem);

      if (options.removeEventListeners) {
          targetElem.clearEventListeners();
      }
  }
  ```

- 闭包容易造成内存泄露，要避免闭包中的循环引用。

  ```Js
  // element.onclick里面包含了闭包函数的引用，闭包中虽然没有使用element，但是也保留了对element的引用(闭包函数会保留调用函数变量的引用)。
   
  // bad
  function foo(element, a, b) {
    element.onclick = function() { /* uses a and b */ };
  }

  // good
  function foo(element, a, b) {
    element.onclick = bar(a, b);
  }

  function bar(a, b) {
    return function() { /* uses a and b */ };} 
  ```

- 使用 IIFE 避免 **Lift** 效应。
  在引用函数外部变量时，函数执行时外部变量的值由运行时决定而非定义时，最典型的场景如下：

  ```js
  var tasks = [];
  for (var i = 0; i < 5; i++) {
    tasks[tasks.length] = function () {
      console.log('Current cursor is at ' + i);
    };
  }

  var len = tasks.length;
  while (len--) {
    tasks[len]();
  }
  ```

  以上代码对 tasks 中的函数的执行均会输出 Current cursor is at 5，往往不符合预期。
  此现象称为 **Lift** 效应 。解决的方式是通过额外加上一层闭包函数，将需要的外部变量作为参数传递来解除变量的绑定关系：

  ```Js
  var tasks = [];
  for (var i = 0; i < 5; i++) {
    // 注意有一层额外的闭包，把参数 i 提前传入。
    tasks[tasks.length] = (function (i) {
      return function () {
        console.log('Current cursor is at ' + i);
      };
    })(i);
  }

  var len = tasks.length;
  while (len--) {
    tasks[len]();
  } 
  ```

- 尽量不要嵌套闭包函数太深，防止混乱。

  ```Js
  // good
  setTimeout(function() {
    client.connect(afterConnect);
  }, 1000);

  function afterConnect() {
    console.log('winning');
  }
   
  // bad
  setTimeout(function() {
    client.connect(function() {
      console.log('losing');
    });}, 1000); 
  ```

- 尽量减少匿名函数，函数表达式或闭包函数尽量加上函数名，方便堆栈追踪和性能分析。

- 自定义事件的事件名必须全小写。
  在 JavaScript 广泛应用的浏览器环境，绝大多数 DOM 事件名称都是全小写的。为了遵循大多数 JavaScript 开发者的习惯，在设计自定义事件时，事件名也应该全小写。

- 自定义事件只能有一个event参数。如果事件需要传递较多信息，应仔细设计事件对象。

  一个事件对象的好处有：

  1. 顺序无关，避免事件监听者需要记忆参数顺序。
  2. 每个事件信息都可以根据需要提供或者不提供，更自由。
  3. 扩展方便，未来添加事件信息时，无需考虑会破坏监听器参数形式而无法向后兼容。

- 设计自定义事件时，应考虑禁止默认行为

  常见禁止默认行为的方式有两种:

  1. 事件监听函数中return false。
  2. 事件对象中包含禁止默认行为的方法，如preventDefault。

- 若无特别说明，禁用delete，用 x = null; 代替。

- 避免修改外部传入的对象。（**用 immutable 解决**）
  JavaScript 因其脚本语言的动态特性，当一个对象未被seal或freeze时，可以任意添加、删除、修改属性值。
  但是随意地对非自身控制的对象进行修改，很容易造成代码在不可预知的情况下出现问题。
  因此，设计良好的组件、函数应该避免对外部传入的对象的修改。

  ```Js
  // selectNode方法修改了由外部传入的datasource对象。如果datasource用在其它场合（如另一个Tree实例）下，会造成状态的混乱。
  function Tree(datasource) {
    this.datasource = datasource;
  }

  Tree.prototype.selectNode = function (id) {
    // 从datasource中找出节点对象
    var node = this.findNode(id);
    if (node) {
      node.selected = true;
      this.flushView();
    }
  };
  ```

  对于此类场景，需要使用额外的对象来维护，使用由自身控制，不与外部产生任何交互的selectedNodeIndex对象来维护节点的选中状态，不对datasource作任何修改。

  ```Js
  function Tree(datasource) {
    this.datasource = datasource;
    this.selectedNodeIndex = {};
  }

  Tree.prototype.selectNode = function (id) {
    // 从datasource中找出节点对象
    var node = this.findNode(id);
    if (node) {
      this.selectedNodeIndex[id] = true;
      this.flushView();
    }
  };
  ```

  除此之外，也可以通过deepClone等手段将自身维护的对象与外部传入的分离，保证不会相互影响。

- **禁止任何形式的 Warning 存在且被提交。**

- **强制异常安全编程模型。**

  使用throw new Error();

- HTML和CSS的文件和文件夹采用全小写加"-"短横线连接的方式命名。
  my-test-page.html，my-test-page.css，这样利于SEO和输入。

- HTML中的类和id名采用全小写加"-"短横线连接的方式命名。
  "<p class="my-test-class" id="my-test-id" /p"。

- **Node的错误和异常处理请参考**[**NodeJS错误处理最佳实践**](http://code.oneapm.com/nodejs/2015/04/13/nodejs-errorhandling/)**。**

- "&&"和"||"的妙用：

  ```Js
  // bad
  function foo(opt_win) {
    var win;
    if (opt_win) {
      win = opt_win;
    } else {
      win = window;
    }
    // ...
  }

  // good
  function foo(opt_win) {
    var win = opt_win || window;
    // ...
  }

  // bad
  if (node) {
    if (node.kids) {
      if (node.kids[index]) {
        foo(node.kids[index]);
      }
    }
  }

  // good
  if (node && node.kids && node.kids[index]) {
    foo(node.kids[index]);
  }
  ```

- 禁止使用Object.freeze, Object.preventExtensions, Object.seal。

  参见[Node.js规范](https://github.com/felixge/node-style-guide#objectfreeze-objectpreventextensions-objectseal-with-eval)。



### JS 前后端错误和异常处理规范：

