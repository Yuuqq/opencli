# Browser Bridge 设置

Browser Bridge 是需要浏览器的命令的默认连接方式。如果命令需要登录，请在启用了扩展的同一 Chrome 配置中登录目标网站。公开 API 命令（如 `hackernews top`）无需 Browser Bridge。

OpenCLI 通过轻量级 **Browser Bridge** Chrome 扩展 + 微守护进程连接浏览器（零配置，自动启动）。

## 扩展安装

### 方法 1：下载预构建版本（推荐）

1. 前往 GitHub [Releases 页面](https://github.com/jackwener/opencli/releases) 下载最新的 `opencli-extension.zip`。
2. 解压后打开 `chrome://extensions`，启用**开发者模式**。
3. 点击**加载已解压的扩展程序**，选择解压后的文件夹。

### 方法 2：加载源码（开发者）

1. 打开 `chrome://extensions`，启用**开发者模式**。
2. 点击**加载已解压的扩展程序**，选择仓库中的 `extension/` 目录。

## 验证

使用 Browser Bridge 的命令会按需启动守护进程。`opencli doctor` 也可以启动它，但不会安装或启用扩展。

```bash
opencli doctor             # 检查守护进程、扩展和浏览器实际连接
opencli doctor --no-live   # 跳过浏览器连接实测，仍可能启动守护进程
opencli doctor --sessions  # 扩展已连接时，同时列出自动化工作区
```

## 工作原理

```
opencli CLI <--- HTTP ---> daemon <--- WebSocket ---> Browser Bridge 扩展
                       127.0.0.1:19825
```

CLI 通过 HTTP 向本地守护进程发送请求。守护进程通过 WebSocket 将请求转发给扩展，再把结果返回给 CLI。

## Daemon 生命周期

Daemon 在连续 **4 小时**未收到浏览器命令，且 Chrome 扩展未连接时自动退出。扩展保持连接时，daemon 可以继续运行。查看状态或日志不会重置空闲计时。

```bash
opencli daemon status    # 查看 daemon 状态（PID、运行时长、扩展连接、内存）
opencli daemon stop      # 优雅关停
opencli daemon restart   # 重启
```

通过 `OPENCLI_DAEMON_TIMEOUT` 环境变量覆盖超时时间（毫秒）。设为 `0` 则永不超时。
