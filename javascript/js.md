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

-  config: 通过 NODE_ENV 来 merge 各个环境下的 config Context


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


### style guide

- node 版本: node version 使用 lastest `LTS` 版本, 目前为 `6.11.4`, 使用`nvm` 作为`node version` 控制工具;

- code lint 使用 `standard.js`;减少 style guide config 的分歧;
  eslint, jslint 配置要花不少功夫;
  
  
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


- 常量: 使用 `const`  标识, 变量名称需要用大写来;
```javascript
const PROJECT_NAME='oraclize'
```

- 类型判断优先使用`Object.prototype.toString`:

```js
const toString = Object.prototype.toString;

function isString(str) {
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