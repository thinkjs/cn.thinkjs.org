# 单元测试

一个 typescript 项目测试示例。

## 测试框架

### [Jest](https://jestjs.io/)

配置文件

```js
// jest.config.js

module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  bail: true,
  verbose: true,
  collectCoverage: false,
}
```

## 测试运行入口文件

- 用于`controller`

```js
// testing.js

const path = require('path')
const Application = require('thinkjs')

const instance = new Application({
  ROOT_PATH: __dirname,
  APP_PATH: path.join(__dirname, 'app'),
  proxy: true,
  env: 'testing',
})

instance.run()

instance.runInWorker({ port: 2333 })
module.exports = instance
```

- 用于非 `controller`

```js
// testing2.js

const path = require('path')
const Application = require('thinkjs')

const instance = new Application({
  ROOT_PATH: __dirname,
  APP_PATH: path.join(__dirname, 'app'),
  proxy: true,
  env: 'testing',
})

instance.run()
```

## 准备测试

### 测试目录结构

```
test
├── agent.js
├── controller
│   └── user.test.js
└── service
    └── UserService.test.js
```

### `agent`

```js
// agent.ts

import debugLib from 'debug'
import nodeFetch from 'node-fetch'

const baseUrl = `http://127.0.0.1:2333/`
const debug = debugLib('test')
const adminToken = ''

const fetchGet = async (pa: string, param = {}, token = adminToken) => {
  return nodeFetch(baseUrl + pa + '?' + new URLSearchParams(param), {
    method: 'get',
    headers: { Authorization: token },
  }).then(res => res.json())
}

const fetchPost = async (path: string, param = {}, token = adminToken) => {
  return nodeFetch(baseUrl + path, {
    method: 'post',
    headers: { 'Content-Type': 'application/json', Authorization: token },
    body: JSON.stringify(param),
  }).then(res => res.json())
}

export { fetchGet, fetchPost, debug }
```

### `controller` 测试

```js
// user.test.ts

import path from 'path'
import { think } from 'thinkjs'
import { debug, fetchGet } from '../agent'

require(path.join(process.cwd(), 'testing.js'))

const basePath = 'user/'

beforeAll(async done => {
  think.app.on('appReady', done)
})

describe(basePath, () => {
  it('用户测试', async () => {
    const pa = 'real'
    const body = await fetchGet(basePath + pa, {})
    debug('%s: %j', pa, body)
    expect(body.errno).toBe(0)
  })
})
```

### `service` 测试

```js
// UserService.test.ts

import path from 'path'
import { think } from 'thinkjs'
import UserService from '../../src/service/UserService'
import { debug } from '../agent'

require(path.join(process.cwd(), 'testing2.js'))

describe('statistic service', () => {
  const service = think.service('UserService') as UserService

  describe('用户测试', () => {
    const func = service.getInfo

    it('用户信息', async () => {
      const args: Parameters<typeof func> = [{ userId: 123 }]
      const result = await func.apply(service, args)
      debug('%s, %o, %j', func.name, args, result)
      expect(result).toBe(0)
    })
  })
})
```

## 运行测试

在 `package.json` 配置测试命令。

```
{
  "test": "export $(cat .env | xargs) && export THINK_UNIT_TEST=1 DEBUG=test* && jest --detectOpenHandles placeholder"
}
```
