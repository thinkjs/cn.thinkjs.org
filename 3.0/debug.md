## 断点调试

### 在 VSCode 下断点调试

* 修改 `src/config/config.js`（多模块项目为 `src/common/config/config.js`），添加 `workers: 1` 配置
* 在项目根目录下添加文件 `debug.js` （与 development.js 文件同级），内容如下：

```js
const InspectorProxy = require('inspector-proxy');
const proxy = new InspectorProxy({ port: 9999 });
const childProcess = require('child_process');

const instance = childProcess.fork('./development.js', {
  execArgv: [ '--inspect' ]
})
instance.on('message', msg => {
  if(msg.act === 'inspectPort' && msg.port) {
    proxy.start({ debugPort: msg.port });
  }
})
instance.on('exit', () => proxy.end());
```
* `npm install inspector-proxy` 安装依赖
* 添加 VS Code 的调试文件 `.vscode/launch.json`，内容如下：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Attach Worker",
      "type": "node",
      "request": "attach",
      "restart": true,
      "port": 9999
    }
  ]
}
```
* 命令行下通过 `node debug.js` 启动服务，然后在 VS Code 里打开断点调试。

### 在 WebStorm 下断点调试

WebStorm 下调试比较简单，直接在根目录 `development.js` 文件上右键选择 debug 启动即可。

![](https://p0.ssl.qhimg.com/t015babb1309bbc9cf7.png)

关于断点调试的更多内容可查看 [#716](https://github.com/thinkjs/thinkjs/issues/716#issuecomment-337449445)。
