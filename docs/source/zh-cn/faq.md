title: 常见问题
---

如果下面的内容无法解决你的问题，请查看 [Egg issues](https://github.com/eggjs/egg/issues)。

## 为什么我的配置不生效？

框架的配置功能比较强大，有不同环境变量，又有框架/插件/应用等很多地方配置。

如果你分析问题时，想知道当前运行时使用的最终配置，可以查看下 `run/application_config.json` 和 `run/agent_config.json` 这两个文件。

也可参见[配置文件](https://eggjs.org/zh-cn/basics/config.html#配置结果)。

## 进程管理为什么没有选型 PM2 ？

1. PM2 模块本身复杂度很高，出了问题很难排查。我们认为框架使用的工具复杂度不应该过高，而 PM2 自身的复杂度超越了大部分应用本身。
2. 没法做非常深的优化。
3. 切实的需求问题，一个进程里跑 leader，其他进程代理到 leader 这种模式（[多进程模型](./core/cluster-and-ipc.md)），在企业级开发中对于减少远端连接，降低数据通信压力等都是切实的需求。特别当应用规模大到一定程度，这就会是刚需。egg 本身起源于蚂蚁金服和阿里，我们对标的起点就是大规模企业应用的构建，所以要非常全面。这些特性通过 PM2 很难做到。

进程模型非常重要，会影响到开发模式，运行期间的深度优化等，我们认为可能由框架来控制比较合适。

**如何使用 PM2 启动应用？**

尽管我们不推荐使用 PM2 启动，但仍然是可以做到的。

首先，在项目根目录定义启动文件：

```js
// server.js
const egg = require('egg');

const workers = Number(process.argv[2] || require('os').cpus().length);
egg.startCluster({
  workers,
  baseDir: __dirname,
});
```

这样，我们就可以通过 PM2 进行启动了：

```bash
pm2 start server.js
```

## 为什么会有 csrf 报错？

通常有两种 csrf 报错：

- `missing csrf token`
- `invalid csrf token`

Egg 内置的 [egg-security](https://github.com/eggjs/egg-security/) 插件默认对所有『非安全』的方法，例如 `POST`，`PUT`，`DELETE` 都进行 CSRF 校验。

请求遇到 csrf 报错通常是因为没有加正确的 csrf token 导致，具体实现方式，请阅读[安全威胁 CSRF 的防范](./core/security.md#安全威胁csrf的防范)。

## 本地开发时，修改代码后为什么 worker 进程没有自动重启？

没有自动重启的情况一般是在使用 Jetbrains 旗下软件（IntelliJ IDEA, WebStorm..），并且开启了 Safe Write 选项。

Jetbrains [Safe Write 文档](https://www.jetbrains.com/help/webstorm/2016.3/system-settings.html)中有提到：

> If this check box is selected, a changed file is first saved in a temporary file. If the save operation succeeds, the file being saved is replaced with the saved file. (Technically, the original file is deleted and the temporary file is renamed.)

由于使用了重命名导致文件监听的失效。解决办法是关掉 Safe Write 选项。（Settings | Appearance & Behavior | System Settings | Use "safe write" 路径可能根据版本有所不同）
