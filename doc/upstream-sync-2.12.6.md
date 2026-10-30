# 官方 v2.12.6 同步记录

同步日期：2026-10-30。

## 同步来源

- 来源：本地 `../lx-music-desktop`，对应 `lyswhut/lx-music-desktop`。
- 原基准：官方标签 `v2.12.2`，提交 `9c364b48`。
- 本次目标：官方 `v2.12.6`，提交 `ad95d5091c9ed689fa72b5e5c849df65f5a679ce`。
- 二开起点：`dccede5`（同步官方 2.12.2 更新）。
- 采用 v2.12.2、二开 HEAD、官方 v2.12.6 的三方文件比较。两个仓库没有共同的 Git 提交祖先，未直接合并无关历史。

## 本次更新

引入官方 v2.12.3 至 v2.12.6 的源码、依赖锁定、构建脚本和原生库更新，包括音源搜索与歌单、歌词解析、自动换源匹配、列表保存顺序、主题、下载元数据、同步连接重定向以及 Windows 窗口兼容修复。完整上游说明见根目录 `CHANGELOG.md`。

保留二开的播放详情页、唱片封面、长文本滚动、歌词加粗及动态背景开关；保留应用名、应用 ID、图标、数据目录、自定义协议和更新源。官方新增的韩语补齐了二开设置文案，旧版 Windows 提示指向 UR 发布页。

`package.json` 与 `package-lock.json` 使用官方 v2.12.6 的依赖和安装脚本，保留二开包名及仓库身份。发布版本号更新为 2.12.6，更新说明使用 UR 文案。

## 后续同步

下次以本次记录的官方提交 `ad95d5091c9ed689fa72b5e5c849df65f5a679ce` 为比较基准，分别对比二开代码和新的官方提交，保留定制改动。不要将二开版本号视为与官方文件完全一致的依据。

## 验证

- `npm run lint`：通过。
- `npm run build`：通过。主进程、主界面、独立桌面歌词和用户脚本四个构建模块均成功，输出位于 `dist/`。
- 三方文件核对：71 个上游独有变动文件与官方一致；43 个无需调整的定制文件保持原样。另对版本号、依赖锁文件、应用名、应用 ID、数据目录、协议、更新源和四种语言的定制文案做了检查。
- SQLite：运行项目安装后脚本，使用预编译库执行内存数据库查询，返回 `{ ok: 1 }`。
- `git diff --check`：通过。
- Electron 42.11.6：已完成下载与安装，使用 Electron 自带 Node.js 24.19.0 执行 SQLite 内存数据库查询，返回 `{ "ok": 1 }`。
- 已尝试 `npm run dev`：四个模块开发编译通过，但 Electron 随即退出，日志出现 `crashpad ... not connected`，未能确认图形界面正常启动；切换到工作区临时数据目录后结果相同，退出原因尚未确定。未执行真实音源播放或安装包验证。

本机为 Node.js 24.22.1、npm 11.19.1。直接 `npm ci` 时 SQLite 的自动原生编译因缺少 Python 失败，因此本次用 `npm ci --ignore-scripts` 安装依赖，再执行 `npm run postinstall` 复制上游提供的预编译库。Windows PowerShell 若拦截 `npm.ps1`，可使用 `npm.cmd` 执行上述 npm 命令。

### Electron 下载排查

当前执行环境直连 GitHub 返回 `connect EACCES`；经本机 `http://127.0.0.1:7897` 代理可访问 GitHub，并跳转至 `release-assets.githubusercontent.com`，分段下载测试成功。此前整文件下载曾出现 `TypeError: terminated`，仅能确定传输连接中断，无法据此判定是代理还是远端链路导致。

本次经代理分段下载官方 `electron-v42.11.6-win32-x64.zip`（149034251 字节），拼接后使用 Electron npm 包附带的校验值验证 SHA-256：`e1f1226e29ba84d247283d3b541715fe2cbdf19beeea99538f870c9c974b9836`。随后通过官方安装脚本从本地缓存安装成功，未更换版本、镜像或绕过校验。

该版本安装脚本使用 `electron_config_cache` 指定缓存目录。当前缓存位于工作区根目录 `.electron-cache/`，可在 PowerShell 中设置 `$env:electron_config_cache='D:\System\Desktop\lx\.electron-cache'` 后运行 `node node_modules/electron/install.js` 复用。网络下载需要代理时，设置 `$env:ELECTRON_GET_USE_PROXY='1'`，下载器将读取 `HTTP_PROXY` / `HTTPS_PROXY`。本次未修改系统持久环境变量。
