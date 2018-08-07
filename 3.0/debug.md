## 断点调试

### 在 VSCode 下断点调试

* 确保 VSCode 的版本 >= 1.22
* 添加 VS Code 的调试文件 `.vscode/launch.json`，内容如下：

  ```json
  {
    "version": "0.2.0",
    "configurations": [
      {
        "port": 9229,
        "type": "node",
        "restart": true,
        "request": "launch",
        "name": "ThinkJS Debug",
        "cwd": "${workspaceRoot}",
        "runtimeExecutable": "node",
        "autoAttachChildProcesses": true,
        "runtimeArgs": ["--inspect", "development.js"]
      }
    ]
  }
  ```
* 点击上面的调试按钮来启动服务。

![alt](https://p.ssl.qhimg.com/t01e177d7042059bc7b.png)

### 在 WebStorm 下断点调试

WebStorm 下调试比较简单，直接在根目录 `development.js` 文件上右键选择 debug 启动即可。

![](https://p0.ssl.qhimg.com/t015babb1309bbc9cf7.png)

### 使用 ndb 断点调试

ndb 是 Chrome 开发的针对 Node.js 的调试工具，使用如下命令进行安装：

```bash
npm install -g ndb
```

使用如下命令启动服务即可进入调试界面：

```bash
ndb npm start
```

调试界面如下，和 Chrome DevTools 的操作是类似的：

![](https://p1.ssl.qhimg.com/t01e31c899aa555ca99.jpg)

具体的调试方法可参考视频 [ndb 调试 Node.js](https://v.qq.com/x/page/t0746cv5e06.html) 以及文章[《使用 ndb 调试你的 Node.js 项目》](https://zhuanlan.zhihu.com/p/41315709)。

关于断点调试的更多内容可查看 [#716](https://github.com/thinkjs/thinkjs/issues/716#issuecomment-337449445)。
