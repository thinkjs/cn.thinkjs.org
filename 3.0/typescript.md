## TypeScript

[TypeScript](http://www.typescriptlang.org/) 是一种由微软开发的自由和开源的编程语言。它是 JavaScript 的一个超集，向这个语言添加了可选的静态类型，在大型项目里非常有用。

ThinkJS 3.2 开始支持了创建 TypeScript 类型的项目，并且开发时会自动编译、自动更新，无需手工编译等复杂的操作。

### 创建 TypeScript 项目

[think-cli](http://github.com/thinkjs/think-cli)  版本 1.1.0 以后可以通过指定 `--typescript` 或者 `-t` 参数来创建 TypeScript 项目：

```sh
thinkjs new thinkjs_demo -t
```

用 `-t` 创建完后的项目会在根目录生成 tsconfig.json, 后续的命令会对应生成 .ts 的文件。
