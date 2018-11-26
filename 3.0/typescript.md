## TypeScript

[TypeScript](http://www.typescriptlang.org/) 是一种由微软开发的自由和开源的编程语言。它是 JavaScript 的一个超集，向这个语言添加了可选的静态类型，在大型项目里非常有用。

ThinkJS 3.2 开始支持了创建 TypeScript 类型的项目，并且开发时会自动编译、自动更新，无需手工编译等复杂的操作。如果希望了解更多实现细节，请参考 [ThinkJS 3.0 如何实现对 TypeScript 的支持](https://zhuanlan.zhihu.com/p/31057738)

### 创建 TypeScript 项目

[think-cli](http://github.com/thinkjs/think-cli)  版本 2.1.1 以后可以以下命令来创建 TypeScript 项目：

```sh
thinkjs new project-name typescript
```

### 引入 Extend 模块定义
用 think-cli 生成 TypeScript 项目模板之后（下文统称项目模板），会自动生成 `src/index.ts` 文件, 在这里需要配置项目用到了哪些 Extend 模块，这样 TS 的智能感知才会生效。
``` js
import * as ThinkJS from '../node_modules/thinkjs';

// 项目 Extend 模块
import './extend/controller';
import './extend/logic';
import './extend/context';
import './extend/think';
import './extend/service';
import './extend/application';
import './extend/request';
import './extend/response'; 

// 外部 Extend 模块
import 'think-view';
import 'think-model';
import 'think-i18n';
// 更多 extend 模块 参考 [think-awesome](https://github.com/thinkjs/think-awesome)

export const think = ThinkJS.think;
```

### 获取 model 和 service 类型
```js
// in controller
import { think } from 'thinkjs';
import SomeService from '../service/someservice';
import SomeModel from '../model/somemodel';

export default class extends think.Controller {
  indexAction() {
    const serviceInstance = think.service('someservice') as SomeService;
    const modelInstance = think.model('somemodel') as SomeModel;
  }
}
```
有参数的 service 推荐直接用 new 来实例化，这样可以在编辑器里直接看到参数类型，代码也更简洁。
```js
// in controller
import { think } from 'thinkjs';
import SomeService from '../service/someservice';

export default class extends think.Controller {
  indexAction() {
    const serviceInstance = new SomeService(param1, param2);
  }
}
```

### TSLint
TypeScript 的项目编写风格与 JavaScript 的非常接近，只要经过一段时间的上手就能适应。我们还基于 ThinkJS 项目的特点配置了一套 TSLint 的规则包含在项目模板里。使用 TSLint 能更快速的在团队里实施规范，并保护代码。

### 编译部署
在开发环境可以使用 `think-typescript` 编译，还支持 `tsc` 直接编译, 编译后的代码和 JS 版本是通用的。
