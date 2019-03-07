## 线上部署

代码开发、测试完成后，需要部署到线上机器部署然后提供服务。

### 代码转译

如果项目中的代码是需要转译的，虽然在开发环境时会实时将 `src/` 目录转译到 `app/` 目录，但代码上线时建议执行 `npm run compile` 命令重新转译一下，避免一些意外的影响。或者在一个干净的目录拉取最新的代码，然后执行转译的命令。

如果修改了 [babel preset](/doc/3.0/babel.html#toc-2cb)，那么需要把 package.json 里的 compile 命令（babel src/ --presets think-node --out-dir app/）作对应的修改。

如果代码不需要转译，那么直接上线 `src/` 目录的代码即可。

### 生产环境

项目创建时，会自动在项目根目录下创建一个名为 `production.js` 的文件，该文件为生产环境运行的入口文件，定义的 `env` 为 `production`。切不可在生产环境把 `development.js` 作为入口文件来启动服务。

### 服务管理

#### Docker 

越来越多的业务使用容器服务来部署项目了，ThinkJS 项目要使用容器部署也非常简单。首先我们要创建业务镜像，以下是一个简单的 Dockerfile 文件，包括基础镜像，依赖安装，项目文件拷贝和项目启动四个部分。

```Dockerfile
FROM mhart/alpine-node:8.9.4

WORKDIR /animaris
COPY package.json /animaris/package.json
RUN npm i --production --registry=https://registry.npm.taobao.org

COPY src /animaris/src
COPY view /animaris/view
COPY www /animaris/www
COPY production.js /animaris/production.js

ENV DOCKER=true
EXPOSE 8360
CMD [ "node", "/animaris/production.js" ]
```

创建好构建脚本后使用如下命令进行构建：

```sh
docker build -t lizheming/animaris ./Dockerfile
```
之后使用如下命令运行镜像，即可使用 http://localhost:8360 访问网站：

```sh
docker run -p 8360:8360 lizheming/animaris
```

关于更多 Docker 部署 ThinkJS 相关内容可查看[《ThinkJS 项目构建 Docker 镜像
》](https://zhuanlan.zhihu.com/p/37082611)。

#### PM2

PM2 是一款专业管理 Node.js 服务的模块，建议在线上使用。使用 PM2 需要以全局的方式安装，如： 
```
sudo npm install -g pm2
```
安装完成后，命令行下会有 `pm2` 命令。

创建项目时，会在项目目录下创建名为 `pm2.json` 的配置文件，内容类似如下：

```json
{
  "apps": [{
    "name": "demo",
    "script": "production.js",
    "cwd": "/Users/welefen/Develop/git/thinkjs/demo",
    "max_memory_restart": "1G",
    "autorestart": true,
    "node_args": [],
    "args": [],
    "env": {}
  }]
}
```
将 `name` 字段改为项目名，`cwd` 字段改为线上项目的具体路径。

##### 项目启动

可以在项目根目录下执行 `pm2 start pm2.json` 来启动项目，执行完成后会显示如下的信息：

![](https://p5.ssl.qhimg.com/t011347d36ca082a2e4.jpg)

##### 项目重启

由于 Node.js 是自身启动服务运行的，所以当有代码更新后，需要重启服务才能让其生效。

最简单的办法可以通过 `pm2 restart pm2.json` 重启服务，但这种方式会导致服务临时性的中断（重启服务需要时间，重启过程中会导致无法处理用户的请求从而导致服务中断）。如果不想服务中断，那么可以通过发送信号的方式来重启方式，具体命令为：

```
pm2 sendSignal SIGUSR2 pm2.json
```
通过发送 `SIGUSR2` 信号，pm2 会将这个信号派发给框架，框架主进程捕获到这个信号后，会 fork 一个新的子进程提供服务，然后逐渐将之前的子进程重启，从而达到不中断服务重启的目的。

##### cluster 模式

框架会强制使用 cluster，然后使用 master/worker 的方式提供服务，所以就不能开启 `pm2` 中的 cluster 模式（如果开启，那么启动服务会直接报错退出）。

#### 手动管理进程

##### 项目启动

如果生产环境不想使用 PM2 来管理服务，那么可以手工通过脚本来管理，可以先在项目根目录下执行 `node production.js` 启动服务。

当访问服务没问题后，可以通过 `nohup node production.js &` 启动服务，通过 `nohup` 和 `&` 将服务在后台运行，执行后会看到类似下面的日志：

```
$ nohup node production.js &
[2] 1114
appending output to nohup.out
``` 

看到输出后，回车，执行 `exit` 命令退出当前终端，这样服务就在后台运行了。

启动完成后，可以通过 `ps aux | grep node` 查看具体的 node 进程情况：

```text
welefen           3971   0.0  0.3  3106048  46244 s001  S+   11:14AM   0:00.65 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3970   0.0  0.3  3106048  46064 s001  S+   11:14AM   0:00.64 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3969   0.0  0.3  3106040  46248 s001  S+   11:14AM   0:00.65 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3968   0.0  0.3  3106048  46400 s001  S+   11:14AM   0:00.65 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3967   0.0  0.3  3106048  46608 s001  S+   11:14AM   0:00.65 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3966   0.0  0.3  3106048  46432 s001  S+   11:14AM   0:00.65 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3965   0.0  0.3  3106040  46828 s001  S+   11:14AM   0:00.65 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3964   0.0  0.3  3106048  46440 s001  S+   11:14AM   0:00.64 /usr/local/bin/node /Users/welefen/demo/production.js
welefen           3963   0.0  0.2  3135796  40960 s001  S+   11:14AM   0:00.31 node production.js
```

其中前面几个为 fork 出来的子进程，最后一个为主进程。

##### 重启服务

当代码修改后，需要重启服务，最简单的办法就是找到主进程的 pid，然后通过 `kill -9 PID` 杀死进程然后重新启动。如果不想中断服务，那么可以给主进程发送 `SIGUSR2` 信号来完成：

```
kill -s USR2 PID
```

比如上面打印出来的日志中主进程的 pid 为 3963，那么可以通过 `kill -s USR2 3963` 来无中断重启服务。当然每次这么执行比较麻烦，可以包装成一个简单的脚本来执行。

```sh
#!/bin/sh
cd PROJECT_PATH; # 进入项目根目录
nodepid=`ps auxww | grep node | grep production.js | grep -v grep | awk '{print $2}' `
if [ -z "$nodepid" ]; then
    echo 'node service is not running'
    nohup node production.js > ~/file.log 2>&1 & 
else
    echo 'node service is running'
    kill -s USR2 $nodepid 2>/dev/null
    echo 'gracefull restart'
fi
```

### 使用 nginx

虽然 Node.js 自身可以直接创建 HTTP(S) 服务，但生产环境不建议直接把 Node 服务可以对外直接访问，而是在前面用 WebServer（如：nginx） 来挡一层，这样有多个好处：

* 可以更好做负载均衡，比如：同一个项目，启动多个端口的服务，用 nginx 做负载
* 静态资源使用 nginx 直接提供服务性能更高
* HTTPS 服务用 nginx 提供性能更高

创建项目时，会在项目根目录下创建了一个名为 `nginx.conf` 的配置文件：

```
server {
    listen 80;
    server_name example.com www.example.com;
    root /Users/welefen/Downloads/demo/www;
    set $node_port 8360;

    index index.js index.html index.htm;
    if ( -f $request_filename/index.html ){
        rewrite (.*) $1/index.html break;
    }
    if ( !-f $request_filename ){
        rewrite (.*) /index.js;
    }
    location = /index.js {
        proxy_http_version 1.1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass http://127.0.0.1:$node_port$request_uri;
        proxy_redirect off;
    }

    location ~ /static/ {
        etag         on;
        expires      max;
    }
}
```

项目中需要将 `server_name`、`root`、`port` 字段值根据实际情况配置，然后将配置文件软链到 nginx 的配置文件目录下，最后重启 nginx 服务即可（可以通过 `nginx -s reload` 重新加载配置文件）。

--------

如果还是想直接通过 Node.js 提供服务，也是可以的，可以直接监听 80 或者 443 的端口（部分环境下需要 `sudo` 执行才能监听这二个端口）。

### HTTPS

现代网站强制建议使用 HTTPS 访问，这样可以提供网站内容的安全性，避免内容被监听、篡改等问题。如果不愿意支付证书的费用，可以使用 [Let's Encrypt](https://letsencrypt.org/) 提供的免费 SSL/TLS 证书，可以参见文章 [Let's Encrypt，免费好用的 HTTPS 证书](https://imququ.com/post/letsencrypt-certificate.html)。

### 常见问题

#### 为什么上线后静态资源访问不了？

创建项目时，会自动生成中间件配置 `src/config/middleware.js`（多模块项目文件为 `src/common/config/middleware.js`），里面有个 resource 中间件用来处理静态资源的请求，但这个中间件默认只在开发环境下开启的，线上环境是关闭的，所以会看到上线后静态资源访问不了的情况。

线上环境静态资源请求推荐用 nginx 来处理，这样性能会更高，对 Node 服务的压力也会小一些。如果非要框架处理静态资源请求，那么可以把 `src/config/middleware.js` 里的配置打开即可。

```js
module.exports = [
  ...
  {
    handle: 'resource',
    enable: true // 始终开启，默认为 `enable: isDev` 表示只在开发环境下开启
  },
  ...
]
```
