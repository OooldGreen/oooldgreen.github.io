---
title: "Full Stack Helsinki 2" #标题
date: 2025-11-30T19:00:44+01:00 #创建时间
lastmod: 2025-11-30T19:00:44+01:00 #更新时间
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

#     relative: false
---

# Part 5 (继续前端)

## Eslint 检查

- 安装、初始化：
    ```
    npm install eslint @eslint/js --save-dev
    npx eslint --init
    npm install --save-dev @stylistic/eslint-plugin-js
    ```
- 配置文件：
    ```
    // eslint.config.js
    import globals from 'globals'
    import js from '@eslint/js'
    import stylisticJs from '@stylistic/eslint-plugin-js'

    export default [
        // 预定义配置
        js.configs.recommended,，
        {
            files: ['**/*.js'],
            languageOptions: {
                sourceType: 'commonjs',
                globals: { ...globals.node },
                ecmaVersion: 'latest',
            },
            // 插件
            plugins: { 
                '@stylistic/js': stylisticJs,
            },
            // 规则
            rules: { 
                '@stylistic/js/indent': ['error', 2],
                '@stylistic/js/linebreak-style': ['error', 'unix'],
                '@stylistic/js/quotes': ['error', 'single'],
                '@stylistic/js/semi': ['error', 'never'],
                eqeqeq: 'error',
                'no-trailing-spaces': 'error',
                'object-curly-spacing': ['error', 'always'],
                'arrow-spacing': ['error', { before: true, after: true }],
                'no-console': 'off'
            }, 
            // 忽略一些文件
            { 
                ignores: ['dist/**'], 
            }
        }
        
    ]

    ```
- 建立运行 eslint 脚本
    ```
    {
        // package.json
        "scripts": {
            "start": "node index.js",
            "lint": "eslint ."
        },
     }
    ```
- 运行检查
    ```
    npm run lint
    ```

## 测试 React 应用

Jest 和 Vitest 十分相似

- 安装 Vitest 和相关库
    ```
    npm install --save-dev vitest jsdom
    npm install --save-dev @testing-library/react @testing-library/jest-dom
    npm install --save-dev @testing-library/user-event
    ```
- 写测试脚本
    ```
    // package.json
    {
        "scripts": {
            "test": "vitest run"
        }
    }
    ```

- 建立测试配置文件 testSetup.js
    ```
    import { afterEach } from 'vitest'
    import { cleanup } from '@testing-library/react'
    import '@testing-library/jest-dom/vitest'

    afterEach(() => {
    cleanup()
    })
    ```

- 扩展 vite.config.js
    ```
    export default defineConfig({
    // ...
    test: {
        environment: 'jsdom',
        globals: true,
        setupFiles: './testSetup.js', 
    }
    })
    ```
- 编写测试
- 运行测试
    ```
    npm test
    ```

- Tip: 让 Eslint 闭嘴：
    ```
    // eslint.config.js
    export default [
    // ...
    {
        files: ['**/*.test.{js,jsx}'],
        languageOptions: {
            globals: {
                ...globals.vitest
            }
        }
    }
    ]
    ```
### 写测试

#### 查找 content 的方法

1. getByText: 
    - 搜索只含作为参数提供的文本且不包含其他内容的元素: `const element = screen.getByText('xxx')`
    - 查找包含特定文本的元素: `const element = screen.getByText('Does not work anymore :(', { exact: false })`
2. findByText: 
    返回的是一个 promise: `const element = await screen.findByText('Does not work anymore :(')`
3. getByTestId: 在元素中添加 <p data-testid='001'></p>
4. queryByText: 
    该方法会返回元素，但如果未找到，则不会引发异常, 可以使用该方法来确保某些内容没有被渲染到组件中
    ```
    render(<Note note={note} />)

    const element = screen.queryByText('do not want this thing to be rendered')
    expect(element).toBeNull()
    ```
5. querySelector:
    ```
    const { container } = render(<Note note={note} />)

    const div = container.querySelector('.note')
    expect(div).toHaveTextContent(
        'Component testing is done with react-testing-library'
    )
    ```
#### 模拟用户输入

```
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import Note from './Note'

// ...

test('clicking the button calls event handler once', async () => {
  const note = {
    content: 'Component testing is done with react-testing-library',
    important: true
  }
  
  // mock 函数
  const mockHandler = vi.fn()

  render(

    <Note note={note} toggleImportance={mockHandler} />
  )

  // session 启动以与呈现的组件进行交互
  const user = userEvent.setup()
  // 测试基于呈现组件的文本找到按钮并单击元素
  const button = screen.getByText('make not important')
  // 模拟点击按钮
  await user.click(button)

  expect(mockHandler.mock.calls).toHaveLength(1)
})
```

### 测试覆盖率

`npm test -- --coverage`

## 端到端测试 (Playwright)

Playwright文档[https://playwright.dev/docs/intro]

- 安装，建立一个独立的项目
    ```
    npm init playwright@latest
    ```
- package.json 配置
    ```
    {
        "scripts": {
            "test": "playwright test",
            "test:report": "playwright show-report",
        },
        // ...
    }
    ```

    ```
    // 后端脚本创建
    {
        // ...
        "scripts": {
            "start:test": "cross-env NODE_ENV=test node --watch index.js"
        },
    }
    ```
- 减少测试失败等待时间
    ```
    // playwright.config.js
    export default defineConfig({
        // ...
        timeout: 3000,
        // 文件执行是否并行，默认为true需要修改
        fullyParallel: false,
        // 默认就有此设置
        workers: 1,
        use: {
            baseURL: 'http://localhost:5173',
            // ...
        },
    })
    ```
- 测试
    ```
    npm run

    // 打开测试报告
    npm run test:report

    // 通过图形界面运行测试
    npm test -- --ui
    // 运行测试保持跟踪查看器
    npm run test -- --trace on
    // 查看跟踪
    npx playwright show-report

    // 可以通过命令行参数定义要使用的浏览器引擎：
    npm test -- --project chromium

    // 测试生成器，录制测试
    npx playwright codegen http://localhost:5173/
    ```

# Part 6 (状态管理： Redux)

## Redux 基本介绍

Redux 是一个全局状态管理系统

### 安装 redux
```
npm install redux
npm install --save-dev deep-freeze
npm install react-redux
// 简化管理状态
npm install @reduxjs/toolkit
```

### Redux 的使用：其中 react-redux 提供了 Provider
```
// main.jsx

import { createStore, combineReducers } from 'redux'
import noteReducer from './reducers/noteReducer'
import { Provider } from 'react-redux'
import { configureStore } from '@reduxjs/toolkit'

const store = configureStore({
  reducer: {
    notes: noteReducer,
  }
})

// const store = createStore(reducer)

ReactDOM.createRoot(document.getElementById('root')).render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

### 创建 reducer 用于改变状态，创建动作可以写在同一个文件中
```
// noteReducer.js
const initialState = [
    ...
]

const noteReducer = (state = [], action) => {
    switch(action.type) {
        case 'NEW_NOTE':
        return [...state, action.data]
        case 'TOGGLE_IMPORTANCE':
        // ...
        default:
        return state
    }
}

export const createNote = (content) => {
    return {
        type: 'NEW_NOTE',
        data: {
            content,
            important: false,
            id: generateId()
        }
    }
}

export default noteReducer
```

或者使用 toolkit 中的 createSlice 直接创建：

```
// noteReducer.js

import { createSlice } from '@reduxjs/toolkit'

const initialState = [
    ...
]

const generateId = () => { ... }

const noteSlice = createSlice({
  name: 'notes',
  initialState,
  reducers: {
    createNote(state, action) {
      const content = action.payload
      state.push({
        content,
        important: false,
        id: generateId(),
      })
    },
  },
})

export const { createNote } = noteSlice.actions
export default noteSlice.reducer

```

### 在 App 组件（或者是单独的 component 中）中使用 dispatch 更改状态, selector 访问数据
```
import { createNote } from './reducers/noteReducer'
import { useSelector, useDispatch } from 'react-redux'

const App = () => {
    const dispatch = useDispatch()
    const notes = useSelector(state => state)

    const addNote = (event) => {
        event.preventDefault()
        const content = event.target.note.value
        event.target.note.value = ''
        dispatch(createNote(content))
    }

    return (
        <div>
            <form onSubmit={addNote}>
                <input name="note" />
                <button type="submit">add</button>
            </form>
            // ...
        </div>
    )
}

export default App

```

## React Query
在处理复杂业务时，使用 react query 处理缓存和同步可以减少 useState 和 useEffect 的使用。
因此在实际应用中，react query 可以用于处理核心的数据流，useState 则用于存储局部（组件内部）有用的状态

```
npm install @tanstack/react-query
```

```
// main.jsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
const queryClient = new QueryClient()

createRoot(document.getElementById('root')).render(
  <QueryClientProvider client={queryClient}>
    <App />
  </QueryClientProvider>
)
```

```
// app.jsx
import { useQuery } from '@tanstack/react-query'
```
```
export const createNote = async (newNote) => {
  const options = {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(newNote)
  }
 
  const response = await fetch(baseUrl, options)
 
  if (!response.ok) {
    throw new Error('Failed to create note')
  }
 
  return await response.json()
}
```

## useContext

使用 Context 处理不怎么变但到处都要用到的数据，减少 props 在组件间的传递

```
// CounterContext.jsx
import { createContext } from 'react'
const CounterContext = createContext()
export default CounterContext
```

```
// 在 App.jsx 中使用事例
// 用一个组件包裹 App
import CounterContext from './CounterContext'

const App = () => {
  const [counter, counterDispatch] = useReducer(counterReducer, 0)

  return (
    <CounterContext.Provider value={{ counter, counterDispatch }}>
      <div>
        // ...
      </div>
    </CounterContext.Provider>
  )
}
```

```
// 使用 context 中存储的数据
import { useContext } from 'react'
import CounterContext from './CounterContext'

const Display = () => {
  const { counter } = useContext(CounterContext)

  return <div>{counter}</div>
}

```

## useReducer

改变局部状态

```
import { useReducer } from 'react'

const counterReducer = (state, action) => {
  switch (action.type) {
    case 'INC':
      return state + 1
    case 'DEC':
      return state - 1
    case 'ZERO':
      return 0
    default:
      return state
  }
}

const App = () => {
  const [counter, counterDispatch] = useReducer(counterReducer, 0)

  return (
    <div>
      <div>{counter}</div>
      <div>
        <button onClick={() => counterDispatch({ type: 'INC' })}>+</button>
        <button onClick={() => counterDispatch({ type: 'DEC' })}>-</button>
        <button onClick={() => counterDispatch({ type: 'ZERO' })}>0</button>
      </div>
    </div>
  )
}

export default App
```

# Part 7