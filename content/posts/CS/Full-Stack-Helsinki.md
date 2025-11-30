---
title: "Full Stack Helsinki 1" #标题
date: 2025-10-28 #创建时间
lastmod: 2025-10-28 #更新时间
categories: [""]
tags: [""]
description: "" #描述
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
# draft: false # 是否为草稿
# comments: true #是否展示评论
# showToc: true # 显示目录
# TocOpen: true # 自动展开目录
# hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
# disableShare: true # 底部不显示分享栏
# showbreadcrumbs: true #顶部显示当前路径
# cover:
#     image: "" #图片路径：posts/tech/文章1/picture.png
#     caption: "" #图片底部描述
#     alt: ""
#     relative: false
---
# Part 1
新建React项目
```
# npm 7+, un double tiret supplémentaire est nécessaire :
npm create vite@latest part1 -- --template react

cd part1
npm install

npm run dev
```

# Part 2
## useState

## 服务器

```
// 1. 建立db.json
// 2. 安装依赖
npm install axios
npm install json-server --save-dev

// 3. 开启服务器
json-server --watch db.json --port 3001

// 4. axios返回服务器端数据
axios.get('http://localhost:3001/notes')
```

## useEffect
axios返回一个promise，useEffect操作DOM

```
import {useEffect} from 'react'

const App = () => {
    const [notes, newNotes] = useState([])

    const hook = () => {
        axios
            .get('http://localhost:3001/notes')
            .then(response => {
            console.log('promise fulfilled')
            setNotes(response.data)
            })
    }
    useEffect(hook, []) // 第二个参数设置触发条件，[]即表示在第一次渲染时不执行
}
```
useEffect的第二个参数用于设置执行的频率，组件首次渲染后，当这个参数的值变化的时候，组件重新渲染。
因此当第二个参数为空数组时，参数不会变化，因此组件只在第一次渲染后执行效果。

我们可以给设置第二个参数，例如在根据汇率改变进行渲染，将此参数设置为`[currency]`

## Exercices 2.18-2.20
### 出bug需要注意的几点：
1. 函数的返回值需要return 否则不会显示的
2. 数据是Object的时候想要显示在页面上需要转换成数组再进行遍历，转换成数组可以使用`Object.values(object)`
3. 子组件接受的参数实际上是一个对象，需要记得解构（{props}）才能使用，直接使用会返回 undefined

### 程序改进
1. ！important ： 记得避免直接调用setState，防止出现无限循环
2. 使用完的数据记得清空
3. 分离小组件增加易读性和可维护性

# Part 3
## 服务器端 Node.js 和 Express
### Node.js

JS使用ES6模块，和JS不同，Node.js 使用 CommonJS，但两者使用起来几乎是一样的
```
// index.js

// 相当于import
const http = require('http')

// 使用createServer来创建一个服务器网络，请求使用200代码应答，标头设置和返回的网站内容设置如下
const app = http.createServer((request, response) => {
    response.writeHead(200, { 'Content-Type' : 'text/plain'})
    response.end('Hello World')
})

const PORT = 3001
app.listen(PORT)
console.log(`Server running on port ${PORT}`)

```
`index.js` 文件创建了服务器，监听前端请求，并返回数据给前端，通常都使用JSON

Hint: 前面用过的 json-server 适合用于测试前端，因为可以快速生成服务器和数据库（db.json），axios 自动生成了多种接口逻辑用于访问 json-server； Node 则更加自由，自己定义服务器逻辑，用于处理正式业务逻辑

### Express

插件安装：

```
npm install express

```

使用：
`app.get('/', (request, response) => {}) `中函数有两个参数，第一个参数 request 包含HTTP请求所有的信息，第二个参数 response 是请求的响应数据
```
const express = require('express')
const app = express()
app.use(express.json())

// 然后就可以使用了，得到的body参数可以直接被使用
app.post('/api/notes', (request, response) => {
    const body = request.body

    if(!body.content) {
        return response.status(400).json({
            error: 'content missing'
        })
    }

    const note = {
        content: body.content,
        important: body.important || false,
        date: new Date(),
        id: generateId()
    }
    notes = notes.concat(note)

    response.json(note)
})
```

作用：
- 用于快速搭建HTTP服务器，有了express，我们不再需要自己手动解析URL、方法，自己设置响应头等
- 使用request.body不需要拼接，直接使用

**HINT**
1. 从URL路径提取使用 `request.params`
2. 从URL的问好提取使用 `request.query`
3. 从请求体（POST/PUT数据内容）中提取使用 `request.body`


### nodemon
用于监督服务器端的变化并重新运行，这样就不用每次手动重启服务器

安装：`npm install --save-dev nodemon`
在`package.json`中：
```
    "devDependencies": {
        "nodemon": "^2.0.15"
    }
```

适合用于开发者模式
```
// package.json
{
  // ..
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  // ..
}

```

### Middleware

express、json-parser 都算是 midlleware

中间件通常定义三个参数，当运行完再进行其他函数的运行

```
const requestLogger = (request, response, next) => {
  console.log('Method:', request.method)
  console.log('Path:  ', request.path)
  console.log('Body:  ', request.body)
  console.log('---')
  next()
}
```

### 把前端文件复制到后端文件夹下

`npm run build` 生成一个 `dist` 文件夹，将这个文件夹复制到后端文件夹下：`cp -r dist ../phonebook-backend`，接着声明中间件 `app.use(express.static('dist'))`

这时可以将 baseURL 省略成一个相对 URL， 例如： `const baseUrl = '/api/persons`

在 Render 上优化部署如下：
```
{
  "scripts": {
    //...
    "build:ui": "rm -rf dist && cd ../frontend && npm run build && cp -r dist ../backend",
    "deploy:full": "npm run build:ui && git add . && git commit -m uibuild && git push"
  }
}
```
npm run build:ui 将前端生成文件夹复制到后端文件夹下
npm run deploy:full 包含更新后端的指令

## 数据库 - MongoDB

### 1. 配置
MongoDB 是一种文档型数据库，使用的时候现在 MongoDB Atlas 建立一个空数据库，调用代码可以封装在一个文件 `/models/person.js` 里：

```
// 先安装：npm install mongoose, 然后使用 mongoDB 数据库
const mongoose = require('mongoose')

/* 
    本地测试时可以在环境变量里配置 uri， 在实际运行中可以在 MongoDB 中配置环境变量（不要把密码单独写出来，或者报存在 github 仓库里）
*/
const url = process.env.MONGODB_URL

<!-- console.log('connection to', url) -->

// 连接数据库
mongoose.connect(url)
 .then(result => {
    console.log('connected to MongoDB')
 })
 .catch(error => {
    console.log('error connecting to MongoDB:', error.message)
 })

// 设置存储数据的数据类型
const personSchema = new mongoose.Schema({
    name: String,
    number: String
})

// 删除不想要展示在前端的数据
personSchema.set('toJSON', {
    transform: (document, returnedObject) => {
        returnedObject.id = returnedObject._id.toString()
        delete returnedObject._id
        delete returnedObject.__v
    }
})

module.exports = mongoose.model('Person', personSchema)

```

然后在 `index.js` 里使用它：

```
// 用于引用环境变量，要先安装 dotenv 哦
require('dotenv').config()
const Person = require('./models/person')
```

### 2. 一些调用方法

- 请求所有对象
    ```
    Person.find({}).then(person => {
        response.json(person)
    })
    ```
- 增加一个对象
    ```
    // 新建一个对象
    const person = new Person({
        ...
    })
    // 保存到数据库
    person.save().then(newPersons => {
        response.json(newPersons)
    })
    ```
- 删除一个对象
    ```
    Persons.findByIdAndDelete(id)
    ```
- 找到一个对象
    ```
    Persons.findById(id)
    ```
- 更新一个对象（注意这个方法是更改原内容而不是创建一个新内容）
    ```
    Person.findByIdAndUpdate(id, person, {new: true})
    ```

## 错误处理

要考虑用户请求的 id 不存在的情况？扔出错误，打印错误，也可以将错误传递给中间件

#### 返回代码含义

- 200 OK
- 400 bad request
- 404 not found
- 500 internal server error

#### errorHandle —— 将错误传递给 express

errorHandle 四个参数：

```
const errorHandle = (error, request, response, next) {
    console.error(eroor.message)

    // 如果是 id 问题
    if(error.name === 'CastError') {
        return response.status(400).send({error: 'malformatted id'})
    }

    // 使用 next 传递错误给 express
    next(error)
}

```

错误信息要放在最后

## 验证信息与ESLint

除了返回错误提示之外，还可以使用 mongoose 提供的验证功能，在定义数据类型时我们可以一并定义验证规则

```
const personSchema = new mongoose.Schema({
    name: {
        type: String,
        minLength: 5,
        required: true
    },
    number: String
})
```

# Part 4

## 贴一张推荐的项目构架写法

```
├── index.js
├── dist
│   └── ...
├── controllers
│   └── notes.js
├── models
│   └── note.js
├── package-lock.json
├── package.json
├── utils
│   ├── config.js
│   ├── logger.js
│   └── middleware.js  

```

## 使用 jest 测试

- 安装 jest `npm install --save-dev jest`
- 修改 `npm test` ：
    ```
    "scripts": {
        "test": "jest --verbose"
    }
    ```
- 还必须用 node 运行，所以要在 package.json 加上
```
{
 //...
 "jest": {
   "testEnvironment": "node"
 }
}
```

- 写测试

```
test('reverse of a', () => {
  const result = reverse('a')

  // expect 比较结果是否正确
  expect(result).toBe('a')
})

// 封装一个名叫 average 的测试
describe('average', () => {
  // tests
})

```

## 给后端写测试

Development Mode：

- package.json 增加 NODE_ENV 环境标识
    ```
    {
    // ...
    "scripts": {
        "start": "cross-env NODE_ENV=production node index.js",
        "dev": "cross-env NODE_ENV=development nodemon index.js",
        "build:ui": "rm -rf build && cd ../frontend/ && npm run build && cp -r build ../backend",
        "deploy": "fly deploy",
        "deploy:full": "npm run build:ui && npm run deploy",
        "logs:prod": "fly logs",
        "lint": "eslint .",
        "test": "cross-env NODE_ENV=test jest --verbose --runInBand"
    },
    // ...
    }
    ```
    `npm install cross-env` 使其可以在 windows 上运行（可以在不同的操作系统上用一致的方式设置环境变量）
- config.js
    ```
    // 开发者模式使用本地测试链接
    const MONGODB_URI = process.env.NODE_ENV === 'test' 
        ? process.env.TEST_MONGODB_URI
        : process.env.MONGODB_URI
    ```
- .env
    ```
    // 添加开发者模式测试链接
    TEST_MONGODB_URI=mongodb+srv://fullstack:thepasswordishere@cluster0.o1opl.mongodb.net/testNoteApp?retryWrites=true&w=majority
    ```
- supertest：用来测试 HTTP API 的工具，模拟各种真实的 HTTP 请求并返回数据（状态吗，响应体，响应头，JSON内容等） `npm install --save-dev supertest`
- 用 supertest 正常写测试，包裹成 api
    ```
    const supertest = require('supertest')
    const api = supertest(app)  // superagent 一个超级代理
    ```
    
可能的报错：
mongoosee 等待时间过长导致失败，可以：
1. 给 test 增加第三个参数
    ```
    test('notes are returned as json', async () => {
      await api
        .get('/api/blogs')
        .expect(200)
        // 这里使用正则表达式来匹配，因为响应头中还包含了其他信息：“application/json; charset=utf-8”
        .expect('Content-Type', /application\/json/)
    }, 100000) //增加第三个参数可以延长等待时间（默认5000ms），非必需动作，只是防止等待时间过长导致运行失败报错

    ```
2. 定义一个 `mongoose.set("bufferTimeoutMS", 30000)`(30s)


- 用 `require('express-async-errors')` 替代 `next try catch` 的使用，简化代码

### async/await Promise.all


## 用户管理

1. 创建一个 userSchema （包含用户名，密码，blog 的 type `type: mongoose.Schema.Types.ObjectId` 和 ref）
2. 博客定义中也增加用户的 type 和 ref
3. 创建 users 路由， post、get 等
4. 创建新笔记/博客的代码需要修改
5. populate() 让用户和博客内容显示在同一个页面

## 用户登录

1. 登录表单
2. 请求，验证用户名和密码
3. 生成 token
4. 后端响应状态码，同时一起返回 token 
5. 将 token 保存在 react 状态中

### 1. 登陆功能
- 安装 jasonwebtoken （生成 token）
- 登陆逻辑
    - 写一个单独的 loginRouter
    - findOne 在数据库中根据 username 搜索用户
    - 用 bcrypt.compare(password, user,passwordHash) 检查 password 
    - 如果用户或密码不正确，返回 401
    - 如果都正确，用 `jwt.sign(userForToken, process.env.SECRET, { expiresIn: 60*60 })` 创建一个 token，有效期 60*60s = 1h （先在 env 中配置 SECRET（任意值）），记得写错误处理 TokenExpiredError
    - 返回 200
    - 如果写 tokenExtractor 中间件必须要在路由之前调用
- 将登陆 js 添加到应用中

### 2. 将 token 发送到服务器（bearer）

- getTokenFrom 的到 authorization
- 添加新笔记的时候，用 `jwt.verify(getTokenFrom(request), process.env.SECRET)` 检查 token 有效性
- 如果 token 丢失或无效，将引发 JsonWebTokenError （自己写错误处理中间件）
- decodeToken 解码 token，其中包含 id 和 username 字段
- 如果 token 解码对象不包含用户 id，返回 401
- 如果 authorization 头正确，就可以创建新笔记/博客了
