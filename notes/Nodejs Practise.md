# Nodejs Practise

## 初始化项目

* 创建package.json，tsconfig.json文件

```shell
mkdir cms_bakcend
cd cms_backend
npm init -y // 生成package.json文件
npm install typescript
npx tsc --init // 生成tsconfig.json 文件
```

注意：这里执行`npx tsc --init`时，需要先执行`npm install typescript`，因为tsc是typescript的一个命令行工具，而`npx tsc --init`的作用时去项目里面（或者全局）找有没有可以执行的tsc命令，然后运行它。

`npx xxx` 并不会自动去下载xxx包，而是去项目或者全局找有没有可以执行的命令，但是确实有一些全局的包npx会自动下载并执行，因为其是**全局 CLI 工具，名字也是 npm 包名**。例如

`npx creat-react-app my-app`

* 添加`.gitignore`文件

* 初始化`eslint`配置文件

```shell
npx eslint --init
// 在这个过程中会安装包eslint, @eslint/js, globals, typescript-eslint
```

* 初始化`.prettierrc`配置文件

```shell
touch .prettierrc
touch .prettierignore
npm i eslint-config-prettier prettier 
```

* 安装`@types/node`

```shell
// @types/node 是 TypeScript 项目中为 Node.js 提供类型定义（Type Definitions） 的包。
npm install --save-dev @types/node
```

- @types/node 属于 **DefinitelyTyped** 项目，它是一个社区驱动的类型定义仓库，专门为 JavaScript 库写 TypeScript 类型。
- 所有 npm 包的类型定义通常都在 @types/xxx 中，比如：
  - @types/express
  - @types/lodash

## 安装express

```shell
npm i express
npm i -D @types/express 
```

* 安装nodemon

```shell
npm install --save-dev nodemon ts-node 
// ts-node 是一个可以直接运行 TypeScript 文件的工具。你可以把它理解成 TypeScript 版本的 node，让你不用先手动编译成 JavaScript，就能运行 .ts 文件。
```

在项目配置`nodemon.json`

```json
{
  "watch": ["src"],
  "ext": "ts",
  "ignore": ["dist"],
  "exec": "ts-node ./src/index.ts"
}
```

在`package.json`文件中添加脚本

```json
"scripts": {
		"dev": "nodemon",
  	...
	},
```

执行命令，运行server

```shell
npm run dev
```

```ts
import express from 'express';
const env = process.env.NODE_ENV || 'development';

console.log(`Server is runmning in ${env} mode!`);

const app = express();
const PORT = process.env.API_PORT || 3001;

app.listen(PORT, () => {
	console.log(`Server is runnint at port ${PORT} `);
});
```

> 在 Node.js 项目中，像 process.env.NODE_ENV 这种**环境变量**通常是在 **.env 文件**中定义的，或者在启动脚本（如 package.json 的 scripts）中设置。
>
> 方式一：
>
> ```json
> "scripts": {
>   "start:dev": "NODE_ENV=development ts-node src/index.ts",
>   "start:prod": "NODE_ENV=production ts-node src/index.ts"
> }
> ```
>
> 方式二：
>
> ```shell
> npm i dovenv
> ```
>
> ```tex
> // file .env
> NODE_ENV=development
> API_PORT=3001
> ```
>
> ```ts
> // index.ts
> import dotenv from 'dotenv';
> dotenv.config();
> const env = process.env.NODE_ENV;
> console.log(env); // development
> ```

## 注册CORS中间件

```shell
npm i cors
npm i --save-dev @types/cors
```

```ts
app.use(cors(corsOptions))
```

## 安装zod包

```shell
npm i zod
```

### 概念

zod 是一个 **TypeScript 优先** 的 **模式验证和数据校验库**，常用于在 Node.js 或前端项目中对数据进行 **结构验证（schema validation）** 和 **类型推导（type inference）**。

- Joi / Yup 的更现代、类型更友好的版本
- 用于校验输入的数据，比如：用户输入、请求体、环境变量、配置文件等
- 自动生成 TypeScript 类型，**不用写两份代码**

### **使用场景**

1. **校验 API 请求体**
2. **校验环境变量**
3. **校验表单数据**
4. **生成 TypeScript 类型（强类型 + 自动推导）**

```ts
type User = z.infer<typeof UserSchema>;
```

**基本用法**

```ts
import { z } from 'zod';

const UserSchema = z.object({
  name: z.string(),
  age: z.number().min(18),
  email: z.string().email(),
});

// 校验数据
const result = UserSchema.safeParse({
  name: 'Tom',
  age: 17,
  email: 'invalid-email',
});

console.log(result.success); // false
console.log(result.error?.errors); // 显示具体校验错误
```

**校验express请求体**

```ts
import express from 'express';
import { z } from 'zod';

const app = express();
app.use(express.json());

const UserSchema = z.object({
  username: z.string(),
  password: z.string().min(6),
});

app.post('/register', (req, res) => {
  const result = UserSchema.safeParse(req.body);

  if (!result.success) {
    return res.status(400).json({ errors: result.error.format() });
  }

  const data = result.data;
  // 现在 data 已经通过校验，可以安全使用
  res.send(`Welcome ${data.username}`);
});

app.listen(3001);
```

## 安装yaml包

```shell
npm i yaml
npm i --save-dev @types/yaml
```

### 概念

yaml 是 Node.js 中用来**读写 YAML 格式文件**的包。YAML（Yet Another Markup Language）是一种简洁、可读性强的配置语言，常用于配置文件，比如 .yml 或 .yaml 文件。

### 功能

主要两个功能：

1. **读取 YAML 文件（解析成 JavaScript 对象）**

```yaml
// config.yaml
port: 3000
debug: true
database:
  host: localhost
  port: 5432
```

```ts
// ts文件
import fs from 'fs';
import yaml from 'yaml';

const file = fs.readFileSync('./config.yml', 'utf8');
const config = yaml.parse(file);

console.log(config.port); // 输出: 3000
```

2. **把 JavaScript 对象写成 YAML 格式**

```ts
// ts文件
import yaml from 'yaml';
import fs from 'fs';

const data = {
  app: 'myApp',
  version: '1.0.0',
  features: ['auth', 'db', 'logs'],
};

const yamlString = yaml.stringify(data);
fs.writeFileSync('./output.yml', yamlString);
```

```yaml
// output.yal
app: myApp
version: 1.0.0
features:
  - auth
  - db
  - logs
```

### **典型应用场景**

- 读取 Kubernetes 配置
- 读取 CI/CD 配置（如 GitHub Actions 的 .yml 文件）
- 管理应用的环境配置（开发、生产环境分离）

## @codegenie/serverless-express

### 概念

@codegenie/serverless-express 是一个用于在 **Serverless（无服务器）架构**中运行 **Express 应用**的 Node.js 库。

### 作用

在 AWS Lambda 等 Serverless 平台中运行 Express 应用

Express 是一个传统的 Node.js Web 框架，通常运行在一个长期运行的服务器上（比如 EC2 或 Docker 容器）。但在 **Serverless** 架构（比如 AWS Lambda）中，请求是事件驱动的，Express 本身并不兼容这些平台。

@codegenie/serverless-express（以及它的前身 aws-serverless-express）提供了一个适配层，让你可以把现有的 Express 应用部署到 AWS Lambda 这类平台上，自动处理事件 → HTTP 请求 → Express 响应 的整个过程。

### 处理过程

1. **接收 AWS Lambda 的事件（event）和上下文（context）**
2. **将它们转换成一个 Express 能理解的 HTTP 请求**
3. **调用你的 Express 应用**
4. **把 Express 的响应转换回 Lambda 的返回格式**

```ts
import express from 'express';
import serverlessExpress from '@codegenie/serverless-express';

const app = express();

app.get('/', (req, res) => {
  res.send('Hello from Express on Lambda!');
});

// 导出给 Lambda handler 使用
export const handler = serverlessExpress({ app });
```

## AWS Lambda

### 概念

AWS Lambda 是 **Amazon Web Services（AWS）提供的无服务器计算服务**。简单来说，它允许你在 **无需管理服务器** 的情况下运行代码。

传统开发流程：

- 你得买服务器、部署代码、处理服务器宕机等问题。

用 AWS Lambda：

- 你只需要写代码，把它上传到 AWS，**AWS 帮你运行、扩展、维护服务器**，你只为用到的资源 **按调用次数和运行时间付费**。

### 核心特点

| **特性**            | **说明**                                                  |
| ------------------- | --------------------------------------------------------- |
| 🚫 无需服务器        | 不需要你部署或管理服务器（即“Serverless”）                |
| 🧩 按需执行          | 代码只在有请求时运行（比如 API 调用、文件上传、定时任务） |
| 💸 按使用计费        | 只为实际运行的时间付费（以毫秒计）                        |
| 🌍 可与 AWS 服务集成 | 可以被 S3、API Gateway、DynamoDB、EventBridge 等触发      |
| ♾️ 自动扩展          | 根据流量自动扩缩容，无需手动干预                          |

### 应用场景

| **场景**   | **示例**                                            |
| ---------- | --------------------------------------------------- |
| 后端 API   | 使用 API Gateway + Lambda 构建 RESTful API          |
| 定时任务   | 每天定时发邮件、清理数据（结合 EventBridge）        |
| 文件处理   | 用户上传文件到 S3 → Lambda 自动处理（压缩、转换等） |
| 数据处理   | DynamoDB 有数据更新 → Lambda 自动处理业务逻辑       |
| 聊天机器人 | 构建 Slack 或 Discord bot，后端逻辑用 Lambda 实现   |

## RESTful API

### 概念

**REST** 全称是 **Representational State Transfer（表现层状态转移）**，是一种软件架构风格。它不是一个标准，而是一种 **设计理念**，主要用于构建可扩展、易维护的 Web 服务。

当你设计的 API 遵循 REST 原则时，我们就称它为 **RESTful API**。

> **RESTful API 就是用 HTTP 协议定义的一组接口，让前端或客户端可以通过统一的方式访问或操作后端的数据。**

### RESTful API的核心概念

| **概念**                  | **说明**                                                     |
| ------------------------- | ------------------------------------------------------------ |
| **资源（Resource）**      | 服务器上的实体数据，比如用户、文章、图片等。                 |
| **URL（统一资源定位符）** | 每个资源对应一个唯一的路径，例如 /users 表示用户资源。       |
| **HTTP 方法**             | 不同的请求方法表示对资源的不同操作：GET、POST、PUT、DELETE。 |
| **无状态（Stateless）**   | 每次请求都是独立的，服务器不保存客户端状态。                 |

### 核心设计理念

1. 资源导向（Resource-Oriented）

- 一切都是资源，比如用户、文章、图片等。
- 每个资源使用唯一的 URL 来标识。
- URL 不应包含动词，只描述资源名。例如：
  - ✅ /users 表示用户资源。
  - ❌ /getAllUsers（不符合 REST 风格）。

2. 使用标准的 HTTP 方法（动作）

   | **方法** | **作用**         |
   | -------- | ---------------- |
   | GET      | 获取资源         |
   | POST     | 创建资源         |
   | PUT      | 更新资源（整体） |
   | PATCH    | 局部更新         |
   | DELETE   | 删除资源         |

3. 无状态（Stateless）

- 每个请求都是独立的，服务器不会记住客户端的任何状态。
- 所有需要的信息都应该在请求中提供（比如 token）。

4. 统一接口（Uniform Interface）

- 客户端和服务端之间的通信格式固定（如 JSON）。
- 接口风格一致，结构清晰。

5. 使用响应码表达结果

| **状态码** | **含义**                          |
| ---------- | --------------------------------- |
| 200        | OK，成功                          |
| 201        | Created，资源创建成功             |
| 204        | No Content，删除成功              |
| 400        | Bad Request，请求参数错误         |
| 401        | Unauthorized，未授权              |
| 404        | Not Found，资源不存在             |
| 500        | Internal Server Error，服务器错误 |

6. 可通过超链接发现资源（HATEOAS，进阶）

- 响应中包含下一步操作的链接。
- 比如：返回用户时，顺便提供“查看订单”的链接。

你想获取一个用户列表，那么就用：

```http
GET /users
```

你不需要调用什么 getUsers() 方法，RESTful API 的风格就是这么自然。

## 安装`jsonwebtoken`

```shell
npm i jsonwebtoken
npm i --save-dev @type/jsonwebtoken
```

## 安装`jwks-rsa`

```shell
npm i jwks-rsa
```

> **扩展express中Request类型的属性**

>  这段代码的作用是 **给 Express 的 Request 类型添加一个自定义的 user 字段**，以便在中间件或路由中使用它时不会被 TypeScript 报错。
>
> ```ts
> //这行看起来没用，但其实是为了“触发”对 express 模块的增强，告诉 TypeScript 我正在扩展它。
> import 'express';
> 
> //这是 Express 内部的核心模块，定义了 Request、Response 等类型。通过 declare module 来“声明扩展”。
> declare module 'express-serve-static-core' {
>   //这就是给原本的 Request 类型新增一个字段：user，类型是 string | undefined。
>   interface Request {
>     user?: string;
>   }
> }
> ```
>
> 这段代码是 TypeScript 中的一种 **模块扩展（Module Augmentation）** 技巧，用来给第三方库（这里是 express）中的类型加上自定义的字段。
>
> 使用这类 **模块扩展（Module Augmentation）** 时,注意点：
>
> 1. 文件名没有强制要求，但推荐命名为
>
> - express.d.ts
> - 或者 types/express/index.d.ts
> - 或放在 @types 目录下，比如：src/@types/express/index.d.ts
>
> 这样是为了让其他开发者（和 TypeScript 编译器）更容易识别这些是**类型扩展声明文件**。
>
> 2. 文件必须被 TypeScript 编译器“看到”
>
> 即：**这个文件要被包含在 tsconfig.json 中**
>
> 把它放在你的 src/ 或 types/ 文件夹下，**确保在 tsconfig.json 的 include 里有包含路径**：
>
> ```json
> {
>   "include": ["src/**/*.ts", "src/**/*.d.ts"]
> }
> ```
>
> 3. 不需要显式导入
>
> 不需要在每个用到 req.user 的文件里写 import './express.d.ts'，只要这个 .d.ts 文件在 tsconfig 的范围内，它就会自动生效。

## 安装`dynamoose`

```shell
npm i dynamoose
```

### 创建DynamoDB客户端对象

```ts
import dynamoose from 'dynamoose';
import { config } from 'dotenv';

// 加载 .env 配置（可选）
config();

// 判断是否本地环境
const isLocal = process.env.NODE_ENV !== 'production';

// 1. 创建 DynamoDB 客户端
const ddb = new dynamoose.aws.ddb.DynamoDB({
  region: process.env.AWS_REGION || 'us-east-1',
  endpoint: isLocal ? 'http://localhost:8000' : undefined,
  accessKeyId: process.env.AWS_ACCESS_KEY_ID || 'fakeMyKeyId',
  secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY || 'fakeSecretAccessKey',
});

// 2. 设置为默认客户端
dynamoose.aws.ddb.set(ddb);

// 3. 设置表默认配置
dynamoose.Table.defaults.set({
  create: isLocal, // 只有本地开发时自动创建表
  prefix: isLocal ? 'dev_' : '',
  waitForActive: false,
});

export default dynamoose;
```

>  **P<T,K>**
>
> ```ts
> // 从类型T中挑选几个属性K，组成一个新的类型
> // type IndexedAttributeValues<T, K keyof T> = Pick<T, K>
> export type IndexedAttributeValues<T, K extends keyof T> = {
> 	[P in K]: T[P];
> };
> ```

### 创建Model对象

```ts
// models/User.ts
import dynamoose from 'dynamoose';

const UserSchema = new dynamoose.Schema({
  id: {
    type: String,
    hashKey: true, // 主键
  },
  username: {
    type: String,
    index: {
      name: 'username-index',
      global: true, // GSI
    },
  },
  email: {
    type: String,
    index: {
      name: 'email-index',
      global: true, // GSI
    },
  },
}, {
  timestamps: true,
  saveUnknown: false,
});

export const UserModel = dynamoose.model('User', UserSchema);
```

```ts
// services/userService.ts
import { UserModel } from './models/User';
import { v4 as uuidv4 } from 'uuid';

// ✅ 创建用户
export const createUser = async (username: string, email: string) => {
  const newUser = await UserModel.create({
    id: uuidv4(),
    username,
    email,
  });
  return newUser;
};

// ✅ 查询用户（通过 username 索引）
export const getUserByUsername = async (username: string) => {
  const results = await UserModel.query('username').eq(username).exec();
  return results[0]; // username是唯一的就取第一个
};

// ✅ 查询用户（通过 email 索引）
export const getUserByEmail = async (email: string) => {
  const results = await UserModel.query('email').eq(email).exec();
  return results[0];
};

// ✅ 更新用户
export const updateUser = async (id: string, updates: Partial<{ username: string; email: string }>) => {
  const updated = await UserModel.update(id, updates);
  return updated;
};

// ✅ 删除用户
export const deleteUser = async (id: string) => {
  await UserModel.delete(id);
};
```

```ts
// test.ts
import { createUser, getUserByUsername } from './services/userService';

(async () => {
  const user = await createUser('john_doe', 'john@example.com');
  console.log('User created:', user);

  const found = await getUserByUsername('john_doe');
  console.log('Found by username:', found);
})();
```

需要注意：

- DynamoDB 中的 **Global Secondary Index (GSI)** 需要你在 AWS 控制台或者 CloudFormation 中预先配置好（如果不是使用 Dynamoose 自动建表的情况下）。
- 索引必须唯一时，你需要自己控制数据唯一性。

> Typescript 知识

1. **知识点1**

```ts
type GameUser = {
  id: string;
  userName: string;
  email: string;
  age: number;
};

type IndexedAttributes = 'userName' | 'email';

type IndexedAttributeValues<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

* IndexedAttributeValues<GameUser, IndexedAttributes>

这是一个类型工具，用来从 GameUser 中**提取出 IndexedAttributes（即 userName 和 email）字段**

```ts
{
  userName: string;
  email: string;
}
```

* [IndexedAttributes]

这是访问索引类型的一种方式。假设 IndexedAttributes = 'userName' | 'email'，那这一部分的意思是：

```ts
{ userName: string; email: string }['userName' | 'email']
```

你可以理解为：

```ts
string | string  // 即 => string
```

最终结果

```ts
value: string;
```

2. **知识点2**

```ts
interface GameUser {
  id: string;
  userName: string;
  email: string;
}
```

* 那么 keyof GameUser 的类型就是：

```ts
'id' | 'userName' | 'email'
```

* (keyof GameUser)[]

这表示一个数组，数组中的每一项必须是 GameUser 类型的键名之一。

等价于：

```ts
type GameUserKeys = 'id' | 'userName' | 'email';
const attributes: GameUserKeys[] = ['id', 'email']; // ✅ 合法
```

3. **知识点3**

```ts
type Partial<User> = {
  id?: string;
  userName?: string;
  email?: string;
}
```

Partial<T> 是 TypeScript 的一个**内置泛型工具类型**，它的作用是：

> 把传入类型 T 的**所有属性都变成可选（optional）属性**。

Partial<User> 让你**不用传完整的 User 对象，只传你需要的字段就行了**。在做表单、更新操作、构建 patch 类型接口时非常有用。

## AWS

### **@aws-sdk/client-cognito-identity-provider**

- **用途**：用于和 **Amazon Cognito** 服务交互，比如用户注册、登录、验证 token、管理用户池等。

- **何时需要**：当你用 Cognito 做用户认证（Auth）时。

AWS Cognito 是 Amazon 提供的一项 **身份验证与用户管理服务**，用于让你轻松实现：

- ✅ **用户注册与登录**
- ✅ **社交登录（Google、Facebook、Apple 等）**
- ✅ **多因素认证（MFA）**
- ✅ **令牌管理（JWT）**
- ✅ **用户信息存储**
- ✅ **与 AWS 其他服务集成（比如 S3、API Gateway、Lambda 等）**

**Cognito 的两个核心组件：**

1. User Pools（用户池）

用于**管理应用用户**（邮箱注册、手机号注册、密码、验证等）

👉 常用于 App 或网站登录注册功能。

比如你可以：

- 创建一个用户池
- 让用户用邮箱注册
- 验证邮箱
- 登录后返回一个 JWT token（用于访问 API）

**2.** Identity Pools（身份池）

用于**临时授权 AWS 服务权限**（比如 S3 上传文件），支持匿名或联合身份登录。

比如：

- 用户登录成功后，Cognito 给他分配临时的 AWS 凭证
- 他可以用这个凭证访问你 S3 上的图片

### **@aws-sdk/client-dynamodb**

- **用途**：用于操作 **DynamoDB** 数据库（底层 API），比如读写表数据。
- **何时需要**：当你直接操作 DynamoDB，而不使用简化封装（比如 dynamoose）时。

### **@aws-sdk/lib-dynamodb**

- **用途**：是对 client-dynamodb 的一个 **轻量封装**，提供了更方便的操作方法（比如 put, get, query），同时支持 TypeScript 更好。
- **何时需要**：推荐使用它来替代原始 client-dynamodb，更方便开发。

### **@aws-sdk/client-s3**

@aws-sdk/client-s3 是 AWS SDK v3 中专门用于操作 **Amazon S3**（对象存储服务）的客户端库，适用于 Node.js 或 Express 项目中上传、下载、删除文件等操作。

- **用途**：用于上传、下载、删除 **S3** 中的文件（比如图片、PDF等）。
- **何时需要**：当你需要用户上传头像、App 上传文件到云端、或读取某些静态资源时。

**安装aws-sdk/client-s3**

```shell
npm i @aws-sdk/client-s3
npm i multer multer-s3
npm i --save-dev @types/multer
```

**使用示例**

1. 创建 S3 客户端

```ts
import { S3Client } from "@aws-sdk/client-s3";

export const s3 = new S3Client({
  region: "us-east-1", // 替换成你的 region
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});
```

2. 上传文件的接口

```ts
// routes/upload.ts
import express, { Request, Response } from "express";
import multer from "multer";
import { PutObjectCommand } from "@aws-sdk/client-s3";
import { s3 } from "../lib/s3";

const router = express.Router();
const upload = multer({storage: multer.memoryStorage() }); // 用 multer 处理 multipart/form-data

router.post("/upload", upload.single("file"), async (req: Request, res: Response) => {
  const file = req.file;

  if (!file) return res.status(400).send("No file uploaded");

  const bucketName = "your-s3-bucket-name";
  const key = `uploads/${Date.now()}_${file.originalname}`;

  const command = new PutObjectCommand({
    Bucket: bucketName,
    Key: key,
    Body: file.buffer,
    ContentType: file.mimetype,
  });

  try {
    await s3.send(command);
    res.json({ message: "File uploaded successfully", key });
  } catch (error) {
    console.error("Upload error", error);
    res.status(500).json({ error: "Failed to upload file" });
  }
});

export default router;
```

上传文件到 Amazon S3 后，**你可以通过 Bucket 名 + Key 拼接出文件的公开 URL**（前提是该对象是公开访问的，或你有设置签名 URL）。

* 拼接 S3 公共访问地址（适用于设置为公开读的文件）

```ts
const bucketName = "your-bucket-name";
const region = "your-region"; // 比如 us-east-1
const key = "uploads/123456_file.png";

// 拼接出文件 URL
const url = `https://${bucketName}.s3.${region}.amazonaws.com/${key}`;
```

* 生成签名访问 URL（私有对象也能访问）

```ts
import { GetObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
import { s3 } from "./lib/s3";

const command = new GetObjectCommand({
  Bucket: bucketName,
  Key: key,
});

const signedUrl = await getSignedUrl(s3, command, { expiresIn: 3600 }); // 1小时有效
```

3. 下载文件

```ts
import { GetObjectCommand } from "@aws-sdk/client-s3";

const command = new GetObjectCommand({
  Bucket: "your-bucket",
  Key: "uploads/file.jpg"
});

const response = await s3.send(command);
const stream = response.Body;
```

4. 删除文件

```ts
import { DeleteObjectCommand } from "@aws-sdk/client-s3";

await s3.send(new DeleteObjectCommand({
  Bucket: "your-bucket",
  Key: "uploads/file.jpg"
}));
```

### **@aws-sdk/client-scheduler**

- **用途**：用于操作 **Amazon EventBridge Scheduler**，这是 AWS 的计划任务服务，可以定时调用 Lambda。
- **何时需要**：你需要定时触发 Lambda（比如每天凌晨执行一个任务），或者要替代 cron job 的时候。

**介绍**

这是 AWS 提供的 **SDK 客户端库**，用来操作 **Amazon EventBridge Scheduler** 服务。

✅ 简单说，它可以让你在 Node.js (比如 Express) 项目里直接用代码来：

- 创建定时任务（Schedule）
- 删除定时任务
- 查询定时任务
- 更新定时任务

这些定时任务可以触发各种 AWS 服务，比如 Lambda、Step Functions 等等。

```shell
npm install @aws-sdk/client-scheduler
```

```ts
// 引入 AWS Scheduler 客户端
import { SchedulerClient, CreateScheduleCommand } from "@aws-sdk/client-scheduler";

// 创建客户端实例
const schedulerClient = new SchedulerClient({
  region: "us-east-1", // 改成你自己的 AWS 区域
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

// Express 路由例子：创建定时任务
app.post('/create-schedule', async (req, res) => {
  try {
    const command = new CreateScheduleCommand({
      Name: "my-sample-schedule",
      ScheduleExpression: "rate(1 day)", // 每天执行一次
      FlexibleTimeWindow: { Mode: "OFF" }, // 不允许弹性时间窗口
      Target: {
        Arn: "arn:aws:lambda:us-east-1:123456789012:function:myLambdaFunction", // 你要触发的 Lambda 函数
        RoleArn: "arn:aws:iam::123456789012:role/SchedulerInvokeRole", // 调用 Lambda 的 IAM 角色
      },
    });
    const result = await schedulerClient.send(command);
    res.json({ message: 'Schedule created successfully', result });
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Failed to create schedule' });
  }
});
```

**什么是 IAM 权限？**

**IAM** 全称是 **Identity and Access Management**，是 AWS 的一个服务。

简单说，就是在 AWS 上：

✅ 管理“**谁**”可以访问你的资源（比如 EC2、S3、DynamoDB、Scheduler 等）

✅ 控制“**可以做什么**”操作（比如只允许读，不允许删）

👉 **IAM 权限**，就是指你为用户、程序、服务器分配的**访问 AWS 服务的权限**。

**举个例子：**

- 你有一个 Lambda 函数，想让它被 AWS Scheduler 定时调用。
- 那么 Lambda 需要一个 IAM 角色（Role），这个角色的权限中允许：
  - 被 Scheduler 调用（InvokeFunction）
- Scheduler 也需要一个 IAM 角色，它要有权限去触发你的 Lambda。

🔵 **没有设置好 IAM 权限**，即使代码没错，AWS 也会拒绝访问，报 AccessDenied 错误。

**如果要用 AWS Scheduler 定时调用 Lambda，IAM 需要这样配置：**

**1. 给** **Scheduler** **一个 IAM Role，它必须有的权限：**

- 允许 Scheduler **调用 Lambda 函数**

所以要创建一个 IAM 角色，**信任实体（Trust Entity）** 设置为：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "scheduler.amazonaws.com"   // 允许 AWS Scheduler 使用这个 Role
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

（这叫**信任策略 Trust Policy**，意思是：Scheduler 服务可以使用这个角色）

**2.给这个角色添加权限策略（Policy），比如允许调用 Lambda**

权限策略示例：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "lambda:InvokeFunction"
      ],
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:yourLambdaFunctionName"
    }
  ]
}
```

```ts
//•	创建一个 CreateScheduleCommand 对象。
//•	command 是你要发送给 AWS 的指令，告诉它要建一个新的定时任务。
const command = new CreateScheduleCommand({
  Name: "my-scheduled-task",
  //•	设定 任务的执行频率。
	//•	"rate(1 day)" 表示：每 1 天执行一次。
	//•	类似 cron 表达式，只不过是 rate 表达式，AWS Scheduler 支持的。
	//（比如：
	//•	"rate(5 minutes)" 每5分钟一次
	//•	"rate(7 days)" 每7天一次）
  ScheduleExpression: "rate(1 day)", // 每天一次
  //•	是否允许任务在一个弹性时间窗口内随机执行。
	//•	OFF 表示：不允许弹性，必须严格按设定的时间执行。
  FlexibleTimeWindow: { Mode: "OFF" },
  //•	Arn：指向要执行的目标，这里是某个 Lambda 函数 的完整 ARN 地址。
	//•	也就是说，每天按时去调用这个 Lambda 函数。
  //•	RoleArn：告诉 AWS 用哪个 IAM Role 的权限去执行任务。
	//•	这个 Role 需要有 调用 Lambda 函数 的权限（即 lambda:InvokeFunction）。
  //（如果没有 Role，会因权限不足，调用失败。）
  Target: {
    Arn: "arn:aws:lambda:us-east-1:123456789012:function:yourLambdaFunction",
    RoleArn: "arn:aws:iam::123456789012:role/yourSchedulerRole", // 这里是上面创建的 IAM Role
  },
});

await schedulerClient.send(command);
```









### **@aws-sdk/client-secrets-manager**

- **用途**：用于从 **AWS Secrets Manager** 获取安全存储的密钥、密码等敏感配置。
- **何时需要**：你把数据库密码、第三方 API 密钥保存在 AWS Secrets Manager 中，而不是硬编码在代码里。

### **@codegenie/serverless-express**

- **用途**：让你可以用 **Express 框架部署到 AWS Lambda**，封装了请求和响应的转换。
- **何时需要**：你用 Express 写好了 API，想部署为 **Serverless 架构（AWS Lambda）**，不需要自己写转接逻辑。

> **Typescript语法**

> [] as const 是 TypeScript 中的**类型断言**语法，意思是把数组断言为“不可变的常量元组（readonly tuple）”。
>
> ```ts
> const arr = ['apple', 'banana'] as const;  
> ```
>
> 这行代码有两个效果：
> 1.	变成 readonly（只读），不能再修改：
>
> ```ts
> arr[0] = 'orange'; // ❌ 报错：Cannot assign to '0' because it is a read-only property.
> ```
>
> 2. **元素的类型变成字面量类型**：
>
> ```ts
> type Fruit = typeof arr[number]; // 'apple' | 'banana'
> ```
>
> **as const常见用途**
>
> 1. 限制只能取特定字符串
>
> ```ts
> const fruits = ['apple', 'banana', 'orange'] as const;
> 
> type Fruit = typeof fruits[number]; 
> // "apple" | "banana" | "orange"
> ```
>
> 2. 用于 Redux、Zod、枚举、配置数组等场景
>
> ```ts
> const roles = ['admin', 'user', 'guest'] as const;
> 
> function hasRole(role: typeof roles[number]) {
>   // 参数 role 只能是 'admin' | 'user' | 'guest'
> }
> ```

## 安装`uuid`

```
npm i uuid
npm i --save-dev @types/uuid
```

- **uuid** 是一个非常流行的 npm 包，用来在你的项目里**生成全局唯一的 ID（UUID）**。
- UUID（全称：**Universally Unique Identifier**）是一个长度固定、**几乎不可能重复**的字符串。

```tex
550e8400-e29b-41d4-a716-446655440000
```

**为什么要用uuid？**

- 给数据库中的数据（比如用户、订单、任务等）生成唯一ID。
- 给上传的文件命名，防止文件名冲突。
- 标识某个操作、请求、会话（session）。
- 分布式系统中，保证不同机器也能生成不重复的 ID。

✅ 总之，用来**安全、快速、自动生成不会冲突的 ID**。

```ts
import { v4 as uuidv4 } from 'uuid';

const newId = uuidv4();
console.log(newId); 
// 输出类似这样：'5b1d5fc5-9a2a-4514-89e0-62f5faafd10d'
```

## 安装`date-fns`

```shell
npm i date-fns
npm i --save-dev @types/date-fns
```

**date-fns** 是一个非常流行的 JavaScript 时间处理库，

它的作用就是：

✅ **轻松地处理日期和时间**，比如：

- 格式化日期
- 日期加减（比如加 3 天）
- 比较两个日期
- 计算时间差
- 判断某个日期是否在某个范围内

而且：

- 体积小
- 只按需引入（tree-shaking 友好）
- 函数式风格，每个功能都是一个独立函数

**date-fns 是 Node.js/前端中处理日期最方便、轻量、现代的工具库，**

## 安装`node-cron`

```shell
npm i node-cron
```

**node-cron** 是一个在 **Node.js** 项目里用来

✅ **定时执行任务** 的库。

它可以按照你设定的规则（比如每天9点、每5分钟一次）

去**自动执行某段代码**。

👉 类似于 Linux 里的 **crontab 定时任务**，但是是纯 JavaScript 写的！

**🛠 常见使用场景**

- 每天凌晨清理数据库日志
- 每小时同步一次数据
- 定时备份文件
- 定时发送提醒邮件、推送通知
- 定时更新缓存数据

```ts
import express from 'express';
import cron from 'node-cron';

const app = express();
const PORT = 3000;

// 创建一个定时任务：每分钟执行一次
cron.schedule('* * * * *', () => {
  console.log(`[定时任务] 每分钟执行一次: ${new Date().toLocaleString()}`);
});

// 一个简单的 API
app.get('/', (req, res) => {
  res.send('Hello World! The server is running.');
});

// 启动服务器
app.listen(PORT, () => {
  console.log(`🚀 Server running on http://localhost:${PORT}`);
});
```

在 **Express 项目**里，node-cron 可以直接作为一个**后台定时调度器**，

管理任务，比如定期同步、备份、发送通知，非常实用！



