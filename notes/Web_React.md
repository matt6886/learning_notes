# Web-React

## Vite

Vite 是一个前端构建工具，用于快发开发和构建Web应用。

```shell
npm create vite@latest my-app -- --template react-ts
cd my-app
npm run dev
```



## Eslint

Eslint是一个用于检查JavaScript/TypeScript的代码质量和风格的工具。

**主要功能**

1. **发现潜在错误**

   比如未使用的变量、语法错误、错误的 API 使用等。

2. **统一代码风格**

   比如强制使用单引号、缩进为 2 个空格、禁止 var 等。

3. **协助团队协作**

   避免团队成员写出风格不一致、质量不一的代码。

4. **与 Prettier 搭配** 自动格式化代码，保持风格一致。

**如何使用**

```shell
npm install eslint -D
npx eslint --init
```

**运行并检查代码**

```shell
npx eslint src/
```

**搭配prettier使用**

```shell
npm install -D prettier eslint-config-prettier eslint-plugin-prettier
```

**`eslint-plugin-prettier`**

让 Prettier 的格式规则变成 ESLint 的规则.

把 Prettier 的格式检查集成到 ESLint 中.

所以当你运行 eslint 时也能检查格式问题（不需要单独跑 prettier）

**`eslint-config-prettier`**

解决 ESLint 和 Prettier 的冲突

有些 ESLint 规则（比如缩进、换行）和 Prettier 冲突，会导致“你修好了 ESLint，Prettier 又改回来”

这个配置专门用来 **关闭与 Prettier 冲突的 ESLint 规则**

```shell
extends: ['plugin:prettier/recommended']
```

这个推荐配置相当于：

```shell
extends: [
  'plugin:prettier/recommended', // 启用 plugin 和 config
  'prettier'                     // 关闭冲突的规则
]
```



## Prettier

**Prettier** 是一个“**代码格式化工具**”，用于自动统一和整理你的代码格式。

它并不检查代码逻辑对错（像 ESLint 那样），而是关注于**代码外观的一致性**。

**作用**

| **功能**         | **说明**                                             |
| ---------------- | ---------------------------------------------------- |
| ✅ 统一格式       | 自动调整缩进、引号、分号、换行等，让团队代码风格一致 |
| ✅ 提高效率       | 无需手动整理格式，专注写逻辑                         |
| ✅ 避免争论       | 减少团队中关于“代码风格”无意义的争吵                 |
| ✅ 与 ESLint 搭配 | 结合使用可做到“检查 + 格式化”双保险                  |

**如何使用**

1. 安装prettier

```shell
npm install --D prettier
```

2. 创建 .prettierrc 文件（或在 package.json 中添加 "prettier" 字段）：
3. 格式化整个项目

```shell
npx prettier . --write
```

只检查不修改（用于 CI 检查）：

```shell
npx prettier . --check
```

**代码格式化顺序**

1. 没有项目配置，只装了 VS Code 的 Prettier 插件

- 如果你：
  - 没有 .prettierrc 文件
  - 没有在 package.json 中配置 "prettier" 字段
  - VS Code 安装了 Prettier 插件并设置为默认格式化工具

> 🔧 **格式化将会使用 Prettier 的默认规则**，即插件内置规则：

2. 项目中配置了.prettierrc或在package.json中设置了"prettier"

> ✅ **VS Code 的 Prettier 插件会优先使用项目配置**，忽略插件内置默认值。

- 这意味着团队协作时，**每个人格式化出来的效果都一样**，不会因为个人 VS Code 设置不同而有差异。
- 所以团队项目中 **一定建议放 .prettierrc 或配置在 package.json 中**。



## 为Typescript项目设置路径别名

```json
// tsconfig.json
"baseUrl": ".",
"paths": {
  "@/*": ["src/*"]
}
```

baseUrl: 告诉 TypeScript：**模块路径的根目录是当前项目根目录**（即 tsconfig.json 所在的位置, 所有 paths 中的映射，都会以这个路径为基础来解析

paths: 是 **路径别名映射表**，它定义了：你写的别名（如 @/xxx）对应实际的路径。

**在Vite中配置使用**

如果你使用的是 Vite + TypeScript 项目，要让路径别名在开发环境中也生效，还需要在 vite.config.ts 中配置：

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```



## Common libraries

### `react-helmet-async`

react-helmet-async 是一个用于 **在 React 应用中动态设置 HTML <head> 内容**（如 <title>、<meta>、<link> 等）的库，支持服务端渲染（SSR）和异步渲染。

**作用**

让你可以在组件中动态控制页面的 <head> 信息，比如：

- 设置页面标题 <title>
- 设置描述 <meta name="description">
- 添加 favicon、OG 标签、SEO 信息等

**为什么不用原生的title**

在单页应用（SPA）中，只有一个 index.html，原生的 <title> 是静态的。

你需要根据不同页面或组件动态设置标题和 meta 信息，比如：

```ts
<Route path="/about" element={<About />} />
<Route path="/contact" element={<Contact />} />
```

每个页面都要有不同的 <title> 和 <meta>，这时就需要用 react-helmet-async。

**使用**

```tsx
// root component
import { HelmetProvider } from 'react-helmet-async';

const helmetContext = {};

<HelmetProvider context={helmetContext}>
  <App />
</HelmetProvider>
```

```tsx
// other component
import { Helmet } from 'react-helmet-async';

function About() {
  return (
    <>
      <Helmet>
        <title>About Us - MyApp</title>
        <meta name="description" content="Learn more about us." />
      </Helmet>
      <h1>About Page</h1>
    </>
  );
}
```



### **`@mantine`**

@mantine 是一个**现代 React UI 组件库**，用于快速构建美观、响应式的 Web 应用界面。

**简介**

- 🧱 **提供大量 UI 组件**（按钮、表单、表格、模态框等）
- 🎨 **自带主题系统**，支持自定义颜色、字体、间距等
- ⚡ **支持 TypeScript**，类型完善
- 🧩 **模块化设计**，你可以只安装你需要的部分
- 📱 **默认支持响应式**，适合做后台管理、仪表盘、表单系统等

**安装**

```shell
npm install @mantine/core @mantine/hooks @mantine/dates @mantine/carousel embla-carousel embla-carousel-react
```

**使用**

```ts
import { Button } from '@mantine/core';

export default function Demo() {
  return <Button color="blue">点击我</Button>;
}
```



### `@tanstack/react-router`

@tanstack/react-router 是一个现代的、**类型安全的 React 路由库**，由 TanStack 团队打造（他们也开发了 React Query）。它的目标是成为比 react-router-dom 更灵活、可组合、类型强的替代方案。

**作用**

用来在 React 应用中进行 **页面导航和 URL 路由控制**，比如：

- 定义不同的页面（路由）
- 根据 URL 展示对应组件
- 支持嵌套路由、懒加载、动态参数等
- 自动类型推导（不再需要写 useParams() 手动断言）

**安装**

```shell
npm install @tanstack/react-router
```

**使用**

* 方法1

1. 安装依赖

```shell
npm install @tanstack/react-router
npm install -D @tanstack/router-cli
```

2. 创建CLI配置文件

在项目根目录下创建tanstack-router.config.ts

```ts
export default {
	routesDirectory: 'src/routes',
	out: 'src/routeTree.gen.ts',
};
```

3. 创建页面组件文件

在 src/routes/ 下创建

src/routes/index.tsx（默认对应路径 /）

```tsx
import React from 'react'

export default function Home() {
  return <h1>Welcome to Home</h1>
}
```

创建布局文件

```tsx
// src/__root.tsx
import { createRootRoute } from '@tanstack/react-router';

const Root = () => {
	return <div>Root</div>;
};

export const Route = createRootRoute({
	component: Root,
});
```

4. 生成路由树文件

```shell
npx tsr generate
```

5. 在项目中使用生成的路由树

```tsx
//src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { HelmetProvider } from 'react-helmet-async';
import { MantineProvider, Modal, createTheme } from '@mantine/core';
import './index.css';
import { createRouter, RouterProvider } from '@tanstack/react-router';
import { routeTree } from './routeTree.gen.ts';

const theme = createTheme({
	fontFamily: 'Nunito Sans Variable',
	components: {
		Modal: Modal.extend({
			defaultProps: {
				removeScrollProps: { enabled: false },
			},
		}),
	},
});

const router = createRouter({ routeTree });

// 可选：注册路由以获得路由安全
declare module '@tanstack/react-router' {
	interface Register {
		router: typeof router;
	}
}

createRoot(document.getElementById('root')!).render(
	<StrictMode>
		<HelmetProvider>
			<MantineProvider theme={theme}>
				<RouterProvider router={router} />
			</MantineProvider>
		</HelmetProvider>
	</StrictMode>,
);
```

* 方法二

1. 安装依赖

```shell
npm install @tanstack/react-router
npm install -D @tanstack/router-plugin @tanstack/react-router-devtools
```

2. 配置vite插件

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';
import { TanStackRouterVite } from '@tanstack/router-plugin/vite';

// https://vite.dev/config/
export default defineConfig({
	plugins: [
		react(),
		TanStackRouterVite({
			// 可选：你可以自定义 routes 目录、生成路径等
			routesDirectory: 'src/routes',
		}),
	],
	resolve: {
		alias: {
			'@': path.resolve(__dirname, '/src'),
		},
	},
});
```

3. 运行`npm run dev`自动生成`routeTree.gen.ts`



### `@tanstack/router-devtools`

开发调试工具

作用类似于 React Query Devtools，用于查看当前的路由信息、路由状态、匹配情况等。



### `@tanstack/router-plugin`

插件机制支持库

这个是给 @tanstack/router 增加 **插件系统** 支持的底层模块，比如未来可插入的数据加载、权限控制、动画等功能。

目前这个库大多数情况下是内部使用或高级用户用来开发自定义插件的。



### `ts-essentials`

ts-essentials 是一个专门为 **TypeScript 项目** 提供 **高级类型工具** 的库。它的目标是 **扩展 TypeScript 的类型能力**，为开发者提供更强大、更实用的类型辅助工具，简化类型定义，增强类型安全。

**安装**

```shell
npm install ts-essentials
```

**功能**

它提供了很多有用的 **类型工具（Utility Types）**，下面是一些常用的类型：

| **工具类型**        | **说明**                                                     |
| ------------------- | ------------------------------------------------------------ |
| DeepPartial<T>      | 递归地把对象类型 T 的所有属性设为可选                        |
| DeepReadonly<T>     | 递归地把对象类型 T 的所有属性设为只读                        |
| NonEmptyArray<T>    | 表示至少有一个元素的数组 [T, ...T[]]                         |
| ValueOf<T>          | 获取一个对象所有 value 的联合类型                            |
| Exact<T, Shape>     | 保证对象 T **不包含** 额外属性，只能匹配 Shape 的精确结构    |
| Opaque<Type, Token> | 创建“品牌类型”，用于区别语义相同但逻辑不同的基本类型（例如 ID vs Email） |
| Merge<T, U>         | 合并两个对象类型                                             |
| StrictExtract<T, U> | 更严格的 Extract，避免意外类型兼容                           |

```ts
import { DeepPartial, NonEmptyArray } from 'ts-essentials';

type User = {
  name: string;
  address: {
    city: string;
    zip: string;
  };
};

const userPatch: DeepPartial<User> = {
  address: {
    city: 'New York',
  },
};

const users: NonEmptyArray<string> = ['Alice']; // ✅ 至少 1 个元素
```



### `@tabler/icons-react`

@tabler/icons-react 是一个图标库，提供了 **React 组件形式的 Tabler Icons 图标**，适用于在 React、Next.js、Vite、Mantine 等现代 React 项目中使用。

**作用**

这是一个 **轻量级 SVG 图标库**，专为 React 项目设计，图标风格是现代、简约、线性（outline-style）的。

Tabler 图标由 https://tabler.io/icons 提供，该网站维护了数千个免费开源图标。

**安装**

```shell
npm install @tabler/icons-react
```

**使用**

```tsx
import { IconHome, IconUser } from '@tabler/icons-react';

function Demo() {
  return (
    <div>
      <IconHome size={24} stroke={2} />
      <IconUser size={32} stroke={1.5} color="blue" />
    </div>
  );
}
```



### `i18next react-i18next i18next-browser-languagedector`

这三个库经常搭配在 React 项目中使用，用于实现**国际化（i18n）功能**。

1. i18next（基础库）

首先，它们都基于 i18next，这是一个强大、灵活、可扩展的 **JavaScript 国际化引擎**，支持：

- 多语言资源管理
- 插值（变量替换，如 Hello {{name}}）
- 语言切换
- 语言回退（fallback）

但它是纯 JS 库，不跟 React 绑定。

2. react-i18next是什么？

这是 i18next 的 **React 绑定库**，用来让你在 React 中更方便地使用国际化功能。

- 提供 <I18nextProvider> 给 React 项目包裹上下文
- 提供 useTranslation() Hook，让你在组件中方便地获取翻译字符串
- 支持组件插值、懒加载、动态翻译等

```ts
import { useTranslation } from 'react-i18next';

const MyComponent = () => {
  const { t } = useTranslation();
  return <h1>{t('welcome_message')}</h1>; // 自动根据语言显示“欢迎”或“Welcome”
};
```

3. i18next-browser-languagedetector 是什么？

这是一个用于**自动检测用户语言**的插件。

- 在用户首次访问时自动检测浏览器设置、cookie、localStorage 等
- 自动选择对应语言资源（如 en, zh, fr）
- 配合 i18next 使用即可自动初始化合适语言

```ts
import i18n from 'i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

i18n
  .use(LanguageDetector) // 插件注册
  .init({
    resources: {...},
    fallbackLng: 'en', // 如果检测失败，就用英文
  });
```

| **库名**                         | **作用**       | **用途**                         |
| -------------------------------- | -------------- | -------------------------------- |
| i18next                          | 国际化核心引擎 | 管理语言资源、翻译逻辑           |
| react-i18next                    | React 框架绑定 | 提供 useTranslation() 等方便 API |
| i18next-browser-languagedetector | 检测用户语言   | 自动识别浏览器语言/本地设置等    |



### `@ts-rest/core`

@ts-rest/core 是一个用于 **在 TypeScript 中定义和共享后端 API 类型** 的库，它的目标是：

> **让前端和后端通过类型共享自动对齐，避免手动维护接口定义。**

可以理解为：

它类似于一个「**类型安全的 REST API 描述器**」，前端通过它可以获得**自动推导的接口类型和请求方法**，而不需要手动写 Axios 请求和接口类型。

1. 作用

- **定义 API 结构（路径、方法、请求体、响应体）**
- **自动生成类型安全的请求函数**
- **前后端共享接口定义文件**
- 支持 OpenAPI 和后续构建 SDK

2. 流程

* **定义接口 schema（通常在后端或共享代码中）**

```ts
// contract.ts
import { initContract } from '@ts-rest/core';

const c = initContract();

export const userContract = c.router({
  getUser: {
    method: 'GET',
    path: '/users/:id',
    responses: {
      200: c.type<{ id: string; name: string }>(),
      404: c.type<{ message: string }>(),
    },
  },
  createUser: {
    method: 'POST',
    path: '/users',
    body: c.type<{ name: string }>(),
    responses: {
      201: c.type<{ id: string; name: string }>(),
    },
  },
});
```

* **在后端绑定逻辑（比如用 Express）**

```ts
// backend.ts
import express from 'express';
import { createExpressEndpoints } from '@ts-rest/express';
import { userContract } from './contract';

const app = express();
app.use(express.json());

const router = {
  getUser: async ({ params }) => {
    if (params.id === '123') {
      return { status: 200, body: { id: '123', name: 'Alice' } };
    }
    return { status: 404, body: { message: 'Not found' } };
  },
  createUser: async ({ body }) => {
    return { status: 201, body: { id: '456', name: body.name } };
  },
};

createExpressEndpoints(userContract, router, app);
```

* **在前端直接使用类型安全的 API 调用：**

```ts
// frontend.ts
import { initClient } from '@ts-rest/core';
import { userContract } from './contract'; // 可与后端共享的文件

const client = initClient(userContract, {
  baseUrl: 'http://localhost:3000',
  baseHeaders: {},
});

const userRes = await client.getUser({ params: { id: '123' } });

if (userRes.status === 200) {
  console.log(userRes.body.name); // 类型自动推导为 string
}
```

**总结**

@ts-rest/core 的核心优势：

| **优势**               | **说明**                       |
| ---------------------- | ------------------------------ |
| 🧠 类型自动推导         | 请求参数、响应体都是类型安全的 |
| 🧩 前后端共享 contract  | 不再手动维护接口类型           |
| 🔒 支持 REST 风格的接口 | 和常规的 REST API 一致         |
| 🔧 可生成 SDK、OpenAPI  | 可以扩展成更大的工具链         |



### `zustand`

zustand 是一个 **轻量级、高性能的状态管理库**，主要用于 React 或 React Native 项目中。

它的目标是：

✅ 更简单

✅ 更少样板代码

✅ 更好性能（无多余 re-render）

✅ 支持中大型应用的状态管理

> Zustand 是 React 应用中用于共享状态（全局 state）的工具，功能类似 Redux，但语法更简洁，使用更灵活。

**Zustand 的核心特点：**

| **性**            | **说明**                                 |
| ----------------- | ---------------------------------------- |
| 不需要 Provider   | 使用时不需要像 Redux 一样包裹 <Provider> |
| 简洁 API          | 只需要一个 create() 来创建 store         |
| 支持中间件        | 可使用 devtools、persist（持久化）等插件 |
| 性能好            | 自动避免不必要的组件更新（按需订阅）     |
| 支持 TypeScript   | 类型友好，非常适合大型项目               |
| React Native 兼容 | 非常适合 React Native 开发               |

**基本使用步骤**

1. **安装**

```shell
npm install zustand
```

2. **创建 store（共享状态）：**

```ts
// store.js
import { create } from 'zustand'

const useCounterStore = create((set) => ({
  count: 0,
  increase: () => set((state) => ({ count: state.count + 1 })),
  decrease: () => set((state) => ({ count: state.count - 1 })),
}))
```

3. **在组件中使用：**

```tsx
import React from 'react'
import { View, Text, Button } from 'react-native'
import useCounterStore from './store'

export default function Counter() {
  const { count, increase, decrease } = useCounterStore()

  return (
    <View>
      <Text>Count: {count}</Text>
      <Button title="+" onPress={increase} />
      <Button title="-" onPress={decrease} />
    </View>
  )
}
```

4. **更复杂用法示例：Auth Store**

```ts
// authStore.js
import { create } from 'zustand'

const useAuthStore = create((set) => ({
  user: null,
  login: (userData) => set({ user: userData }),
  logout: () => set({ user: null }),
}))
```

组件中使用：

```ts
const user = useAuthStore((state) => state.user)
const login = useAuthStore((state) => state.login)
```

5. **中间件扩展（如持久化）：**

```shell
npm install zustand-middleware
```

```ts
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

const useAuthStore = create(
  persist(
    (set) => ({
      user: null,
      login: (user) => set({ user }),
      logout: () => set({ user: null }),
    }),
    {
      name: 'auth-storage', // localStorage key 或 AsyncStorage key
    }
  )
)
```

**适用场景：**

✅ 适合中小项目或中大型项目的全局状态管理（比 Redux 更简洁）

✅ 适用于 React Native 和 Web

✅ 替代 useContext + useReducer 的更优方案



### `idb-keyval`

idb-keyval 是一个用于浏览器的 **轻量级封装库**，用于更简单地操作 **IndexedDB**。

> **一句话介绍：** idb-keyval 让你像使用 localStorage 一样简单地使用 IndexedDB 来进行本地数据存储，但它是异步的、支持更大数据量。

**什么是 IndexedDB？**

IndexedDB 是浏览器提供的一个本地数据库，支持结构化数据、本地持久化存储。

但原生 API 非常复杂，所以 idb-keyval 把它封装得很简单易用。

**使用场景：**

- 在前端本地存储结构化数据
- 离线缓存
- 本地数据同步
- 替代 localStorage，用于更大、更复杂数据结构

**基本用法：**

1. **安装：**

```shell
npm install idb-keyval
```

2. **基本用法：**

```ts
import { set, get, del, clear, keys } from 'idb-keyval'

// 写入
await set('username', 'matt')

// 读取
const username = await get('username')

// 删除
await del('username')

// 清空全部
await clear()

// 获取所有 key
const allKeys = await keys()
```

3. **存储对象**

```ts
await set('user', { id: 1, name: 'Alice' })

const user = await get('user')
console.log(user.name) // Alice
```

4. **自定义数据库/表（store）**

```ts
import { Store, get, set } from 'idb-keyval'

const customStore = new Store('my-db', 'my-store')

await set('token', 'abc123', customStore)

const token = await get('token', customStore)
```

**注意事项：**

- 所有操作都是 **Promise 异步**
- 数据结构必须是结构化可序列化的（不能包含函数、DOM 等）
- 存储容量远大于 localStorage，适合缓存大数据

**在 React 中使用（示例）：**

```ts
import { useEffect, useState } from 'react'
import { get, set } from 'idb-keyval'

function App() {
  const [name, setName] = useState('')

  useEffect(() => {
    get('name').then((val) => {
      if (val) setName(val)
    })
  }, [])

  const save = () => {
    set('name', name)
  }

  return (
    <>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button onClick={save}>Save</button>
    </>
  )
}
```



### `ramda`

Ramda 是一个功能强大的 **函数式编程库（Functional Programming library）**，专为 JavaScript 设计，强调 **纯函数**、**不可变性** 和 **函数组合（composition）**。

> **Ramda** 让你更简单、优雅、声明式地处理数组、对象、函数组合等复杂逻辑，避免副作用，代码更清晰、可测试。

**Ramda 的核心特点：**

| **特性**     | **描述**                               |
| ------------ | -------------------------------------- |
| 函数式       | 一切操作用纯函数实现，避免副作用       |
| 不改变输入值 | 所有函数返回新的数据，不修改原数据     |
| 自动柯里化   | 函数可以部分应用参数（简化组合）       |
| 组合性强     | 支持 compose / pipe 将多个函数串联起来 |
| 支持对象操作 | 可以深层处理对象、路径等数据结构       |

**安装 Ramda：**

```shell
npm install ramda
```

**基本使用示例**

1. **引入方式：**

```ts
import * as R from 'ramda'
```

2. **柯里化函数（Curried Functions）**

```ts
const add = R.add(1)
console.log(add(2)) // 3
```

> **柯里化函数**：把接收多个参数的函数，变成**每次只接收一个参数**的函数。
>
> ```ts
> function add(a, b) {
>   return a + b
> }
> 
> add(2, 3) // 输出 5
> ```



3. **函数组合 compose/pipe**

```ts
const double = x => x * 2
const increment = x => x + 1

const composed = R.compose(double, increment) // 从右往左
console.log(composed(3)) // (3 + 1) * 2 = 8

const piped = R.pipe(increment, double) // 从左往右
console.log(piped(3)) // same: 8
```

**4. 操作数组**

```ts
const nums = [1, 2, 3, 4, 5]
const result = R.filter(n => n % 2 === 0, nums)
console.log(result) // [2, 4]
```

5. **操作对象**

```ts
const person = { name: 'Alice', age: 25 }

const getAge = R.prop('age')
console.log(getAge(person)) // 25
```

6. **深层访问/修改对象属性（lens）**

```ts
const user = { name: 'Tom', address: { city: 'Beijing' } }

const cityLens = R.lensPath(['address', 'city'])

const city = R.view(cityLens, user) // 获取
const updated = R.set(cityLens, 'Shanghai', user) // 设置（不会改变原对象）
console.log(updated) // { name: 'Tom', address: { city: 'Shanghai' } }
```





## TypeScript

### 交叉类型: &

& 表示 **交叉类型**（intersection type）👉 用于**合并多个类型的属性**

```ts
type A = { name: string };
type B = { age: number };
type C = A & B; // C = { name: string; age: number }
```

当 & 用在基本类型（比如 string | number）时

```ts
type T = string & number;
```

这个例子中，string 和 number 是**基础类型**，它们没有交集，**交叉结果是 never**：

```ts
type T = never;
```

当 & 用在字符串字面量和基本类型时

```ts
type A = 'abc' & string;  // => 'abc'
type B = string & number; // => never
type C = 'abc' & number;  // => never
```

**强制一个类型必须是字符串**

```ts
type P = string | number;
type SafeP = P & string; // => string
```

> **把宽泛的联合类型缩窄成纯字符串**，常用于类型安全保护，比如处理模板字符串推断、键名映射等。



### 联合类型: |

```ts
type A = { type: 'cat'; meow: () => void };
type B = { type: 'dog'; bark: () => void };
type C = A | B; // 只能是猫或狗
```



### extend用法

在 TypeScript 中，extends 是非常强大的关键字之一，它在 **类型系统中扮演“继承”、“约束”和“条件判断”** 等多个角色，取决于具体语境

1. **泛型约束（限制类型范围）**

```ts
function printLength<T extends { length: number }>(value: T) {
  console.log(value.length);
}
```

- T extends { length: number } 表示：**T 必须具有 length 属性**
- 所以只能传入有 .length 的对象，如字符串、数组、类数组对象等。

```ts
printLength('hello');       // ✅ string 有 length
printLength([1, 2, 3]);     // ✅ 数组有 length
printLength(42);            // ❌ 报错：number 没有 length
```

2. **类型继承（类似类的继承）**

```ts
type Animal = { name: string };
type Dog = Animal & { breed: string }; // 或者：interface Dog extends Animal
```

- Dog 继承了 Animal 的属性，并添加了 breed。
- 这就是基本的**类型继承**。

3. **条件类型判断 + infer 联合使用**

```5s
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
```

- T extends (...args: any[]) => infer R 表示：如果 T 是函数类型，就提取返回值 R。
- 否则返回 never。

```ts
type A = GetReturnType<() => string>; // string
type B = GetReturnType<number>;       // never（不是函数）
```

4. **实际应用对比总结**

| **用法类型**     | **示例**                             | **含义**                   |
| ---------------- | ------------------------------------ | -------------------------- |
| 泛型约束         | <T extends { id: number }>           | 限制 T 必须含有 id: number |
| 类型继承         | type B = A & { extra: string }       | B 包含 A 所有属性          |
| 条件类型 + infer | T extends (...args) => infer R ? ... | 用于类型提取和判断         |

5. **高级例子：联合类型条件判断**

```ts
type IsString<T> = T extends string ? true : false;

type A = IsString<'hello'>; // true
type B = IsString<number>;  // false
```

> extends 可以像 if 一样使用，判断一个类型是否属于另一个类型的子集。



### infer用法

infer 是 TypeScript 的一种**类型推导关键字**，用于在 extends 条件类型中提取某部分的类型，并赋予一个名字，以便后续使用。

> **infer X 表示“推导出一个类型，命名为 X”**

用法场景一般是：**从复杂类型中提取类型的一部分**，例如函数参数类型、返回值、数组元素、元组类型、字符串部分等。

1. **提取数组元素类型**

```ts
type ElementType<T> = T extends (infer U)[] ? U : never;
```

```ts
type A = ElementType<string[]>; // A = string
type B = ElementType<number[]>; // B = number
type C = ElementType<boolean>;  // C = never （不是数组）
```

- 如果 T 是一个数组类型（如 string[]），就提取数组元素类型。
- infer U 会推导出 string 或 number 等，作为返回值。

2. **提取函数返回值类型**

```5s
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
```

```ts
type A = ReturnType<() => number>;        // number
type B = ReturnType<(x: string) => void>; // void
```

- 如果 T 是函数类型，则提取它的返回值类型 R
- infer R 就是提取函数的返回类型

3. **提取 Promise 的内部值类型**

```ts
type UnwrapPromise<T> = T extends Promise<infer R> ? R : T;
```

```ts
type A = UnwrapPromise<Promise<string>>; // string
type B = UnwrapPromise<number>;          // number（不是 Promise，不推导）
```

4. **提取字符串模板的某部分**

```ts
type ExtractId<T> = T extends `user/$${infer Id}` ? Id : never;

type A = ExtractId<'user/$123'>; // '123'
type B = ExtractId<'user/abc'>;  // never
```

5. **总结**

| **示例代码**                          | **作用**                |
| ------------------------------------- | ----------------------- |
| T extends Array<infer U>              | 提取数组元素类型        |
| T extends (...args: any[]) => infer R | 提取函数返回值          |
| T extends Promise<infer P>            | 提取 Promise 包裹的类型 |
| T extends \/path/$${infer Param}``    | 提取字符串参数部分      |



### never理解

never 是 TypeScript 中一个非常特殊、底层的类型，它表示 **“永远不会发生的值”**。理解它的方式可以从几个角度来展开

1. **函数永不返回（抛出异常或死循环）**

```ts
function throwError(): never {
  throw new Error("Something went wrong!");
}
```

- throwError 函数永远不会有“返回值”。
- 它 **抛出异常之后就终止了程序的执行路径**，所以其返回类型是 never。

2. **死循环**

```ts
function infiniteLoop(): never {
  while (true) {}
}
```

- 永远执行、永不返回的函数，也是 never 类型。

3. **类型保护失败（永远不会出现的分支）**

```ts
type Shape = { kind: 'circle' } | { kind: 'square' };

function handleShape(shape: Shape) {
  if (shape.kind === 'circle') {
    // 处理 circle
  } else if (shape.kind === 'square') {
    // 处理 square
  } else {
    const _exhaustiveCheck: never = shape; // ❌ 报错，说明你漏了某个类型
  }
}
```

- 如果以后 Shape 增加了新的 kind 类型，比如 'triangle'，这里会立刻报错，提示你**没处理完全**。
- 这叫 **穷尽检查**，非常常用于 **类型安全控制流程**。

4. **联合类型差集中使用**

```ts
type A = string | number;
type B = string;
type C = Exclude<A, B>; // number
```

如果你 Exclude<string, string>，结果是：

```ts
type D = Exclude<string, string>; // never
```

> 说明：从 string 中排除 string，结果就没有了，就是 never。

**为什么需要 never**

- 帮助你发现逻辑错误（如穷尽检查）。
- 使得类型系统更健壮、安全。
- 在泛型和条件类型中用于**表示不可能、错误、终止逻辑的分支**。

**小结**

| **用途**                | **例子**                          | **含义**           |
| ----------------------- | --------------------------------- | ------------------ |
| 抛异常/死循环的函数返回 | function f(): never { throw ... } | 永远不会返回       |
| 类型不成立/差集         | Exclude<'a', 'a'> -> never        | 类型排除后为空     |
| 类型检查兜底            | const x: never = value;           | 提示逻辑未处理完全 |



### NonNullable用法

NonNullable<T> 是 TypeScript 提供的一个**内置工具类型**，它的作用是：

> **从类型 T 中剔除掉 null 和 undefined。**

```ts
NonNullable<T>
```

1. **基本用法**

```ts
type A = string | null | undefined;
type B = NonNullable<A>; // => string
```

- 原本类型 A 是 string | null | undefined
- 使用 NonNullable<A> 后，结果是 string

2. **和函数参数一起用**

```ts
function printLength(text: NonNullable<string | null | undefined>) {
  console.log(text.length);
}

printLength("Hello"); // ✅
printLength(null);    // ❌ 报错
printLength(undefined); // ❌ 报错
```

> 你强制告诉 TypeScript：我不接受 null 或 undefined。

3. **配合泛型使用**

```ts
type MyType<T> = NonNullable<T>;

type Result = MyType<number | null>; // => number
```

4. **在组件 props 中使用**

```ts
type Props = {
  name: string | null;
};

function MyComponent(props: { name: NonNullable<Props['name']> }) {
  return <div>{props.name}</div>;
}
```

- 保证 name 是必填的，且不能为 null。



### as const用法

在 TypeScript 中，as const 是一个非常实用的类型断言，它的作用是告诉编译器将一个表达式的类型 **保持为最字面量（literal）形式**，而不是更宽泛的类型。

1. **字面量值保持不变**

```ts
const direction = "left" as const;
// 推断为 "left"，不是 string
```

2. **用于对象**

```ts
const status = {
	code: 200,
	message: "ok",
} as const;
```

类型推断为

```ts
{
  readonly code: 200;
  readonly message: "ok";
}
```

相比之下如果不加 as const：

```ts
const status = {
	code: 200,
	message: "ok",
};
// 推断为 { code: number; message: string; }
```

3. **配合typeof构建枚举类型** 

```ts
const roles = ["admin", "user", "guest"] as const;

type Role = typeof roles[number]; 
// "admin" | "user" | "guest"
```

**总结**

| **作用**          | **说明**                                   |
| ----------------- | ------------------------------------------ |
| 保持字面量类型    | 避免被自动推断为 string、number 等宽泛类型 |
| 自动变为 readonly | 对数组和对象变成不可修改                   |
| 构造精确类型      | 在类型定义中更准确                         |



### Record用法

Record 是 TypeScript 提供的一个**内置泛型工具类型**，常用于**构造具有固定 key 和对应 value 类型的对象类型**。

```ts
Record<Keys, Type>
```

- Keys: 要作为对象键的联合类型（例如 'a' | 'b' | 'c'）。
- Type: 所有键对应的值的类型。

1. **基本使用**

```ts
type Role = 'admin' | 'user' | 'guest';

type RoleDescriptions = Record<Role, string>;
```

等价于：

```ts
type RoleDescriptions = {
  admin: string;
  user: string;
  guest: string;
}
```

你可以这样使用它：

```ts
const descriptions: RoleDescriptions = {
  admin: 'Administrator',
  user: 'Regular User',
  guest: 'Guest User',
};
```

2. **与联合类型配合**

```ts
type Status = 'success' | 'error' | 'loading';

const statusColors: Record<Status, string> = {
  success: 'green',
  error: 'red',
  loading: 'gray',
};
```

3. **嵌套使用**

```ts
type Languages = 'en' | 'zh';
type Keys = 'welcome' | 'logout';

type Translations = Record<Languages, Record<Keys, string>>;
```

等价于：

```ts
type Translations = {
  en: {
    welcome: string;
    logout: string;
  };
  zh: {
    welcome: string;
    logout: string;
  };
}
```

**应用场景**

| **场景**           | **用途**                            |
| ------------------ | ----------------------------------- |
| 构造映射表         | 将一组固定的 key 映射到特定类型的值 |
| 保证对象结构完整性 | 所有 key 都必须被定义               |
| 替代手写对象类型   | 使用联合类型动态构造                |

你可以结合其他类型工具使用：

```ts
type OptionalSettings = Partial<Record<'theme' | 'layout', string>>;

// 相当于：
type OptionalSettings = {
  theme?: string;
  layout?: string;
}
```



### keyof用法

在 TypeScript 中，keyof 是一个 **类型操作符（type operator）**，它用于提取一个类型的所有键名（key），并将它们组成一个**联合类型（union type）**。

```ts
keyof T
```

其中 T 是一个对象类型，keyof T 会返回这个对象所有键的集合（以联合类型的形式表示）。

1. **基础用法**

```ts
type Person = {
  name: string;
  age: number;
};

type Keys = keyof Person; // "name" | "age"
```

keyof Person 的结果是 "name" | "age"，即 Person 类型中所有属性名组成的联合类型。

2. **结合泛型**

```ts
function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: 'Tom', age: 30 };

const name = getValue(person, 'name'); // 推断为 string
const age = getValue(person, 'age');   // 推断为 number
```

解释：

- K extends keyof T 表示 K 必须是 T 的键名之一。
- T[K] 表示 key 对应的值的类型。
- 这样写可以让函数在传入不同 key 时自动推断出不同的值类型（即 name 得到 string，age 得到 number）。

3. **配合映射类型**

```ts
type Person = {
  name: string;
  age: number;
};

type ReadonlyPerson = {
  readonly [K in keyof Person]: Person[K];
};
```

解释：

- [K in keyof Person]：遍历 Person 所有的键名（“name” | “age”）
- Person[K]：获取每个键对应的类型
- 最终生成的 ReadonlyPerson 类型如下：

```ts
type ReadonlyPerson = {
  readonly name: string;
  readonly age: number;
}
```

4. **与 typeof 联合使用**

```ts
const user = {
  id: 1,
  name: 'Alice',
};

type User = typeof user;     // 自动生成类型
type UserKeys = keyof User;  // "id" | "name"
```

**keyof 的作用**

| **功能**           | **说明**                         |
| ------------------ | -------------------------------- |
| 获取类型的所有 key | 以联合类型形式返回               |
| 搭配泛型           | 限制函数只能使用合法属性         |
| 搭配映射类型       | 实现深拷贝、只读、可选等结构变换 |
| 与 typeof 配合     | 动态生成类型后提取键名           |



### [variables: {...}] 理解

[variables: {...}] 是 **TypeScript 中的元组类型（tuple）语法的一种形式**，它让你能够在定义函数时精确描述参数结构。

1. 什么是 [variables: {...}]？

这是 **元组类型** 的写法，它表示：

- 函数必须接收一个参数（位置固定）
- 这个参数是一个对象类型
- variables 是这个参数的命名（仅为了可读性）

```ts
function fn(...args: [variables: { name: string }]) {}
```

这里：

- args 是一个数组（元组）；
- 元组中 **只有一个值**；
- 那个值的名字是 variables，类型是 { name: string }。

2. 这是哪种语法？标准定义在哪里？

这是 TypeScript **4.0 以后引入**的功能之一，叫作：

**Named Tuple Elements（命名元组元素）**

在 TypeScript 中，我们可以为元组中的每一项命名，使得语义更清晰。

```ts
type SingleArg = [user: { name: string }];
```

这和下面写法是一样的，只是更具可读性：

```ts
type SingleArg = [{ name: string }];
```

3. 在函数中怎么用

```ts
function greet(...args: [user: { name: string }]) {
  console.log(`Hello, ${args[0].name}`);
}
```

调用

```ts
greet({ name: 'Matt' }); // ✅
```



