nginx-live-win32
================

Windows 绿色包：nginx + nginx-live-module（RTMP / HTTP-FLV 直播），
另附 `nginx_service.exe`（把 nginx 注册为 Windows 服务，并提供托盘管理界面）。

版本号、编译参数、exe 大小与 SHA256 由构建脚本自动生成在 **BUILD-INFO.txt**，
请以该文件为准（不要手工编辑、也不要在这里手写版本）。

# 下载

* **推荐**：到 [Releases](../../releases/latest) 页下载 `nginx-live-<版本>-win32.zip`（绿色包，解压即用），
  同页附 `SHA256SUMS.txt` 校验和；每个版本的构建来源与组件版本见 Release 说明。
* 也可以直接 clone 本仓库：仓库内提交的就是同一份可运行包。

# 使用方法
* 直接运行：双击 `nginx.exe`
* 作为服务/托盘管理：运行 `nginx_service.exe`（安装、启动、停止、卸载、打开配置/页面）
* 命令行停止：`nginx.exe -s stop`

# 简要说明
`conf/nginx.conf` 为配置文件实例：
* RTMP 监听 1935 端口，application `live`（直播）
* HTTP 监听 8080 端口：
  * `:8080/stat` 查看 stream 状态
  * `:8080/<app>/<stream>.flv` HTTP-FLV 拉流，如 `http://localhost:8080/live/mystream.flv`

# 注意
不支持 exec

# 许可与来源
本包是 nginx + nginx-live-module + 服务管理器的组合二进制，各组件的许可证全文随包
附在 `LICENSES/` 目录，完整清单与版权声明见 `THIRD-PARTY-NOTICES.md`：

| 组件 | 许可证 |
|---|---|
| nginx | BSD 2-Clause |
| nginx-live-module、服务管理器 | Apache License 2.0 |
| OpenSSL | Apache License 2.0 |
| PCRE2 | BSD 3-Clause（含 PCRE2 例外条款） |
| zlib | zlib License |

nginx 是 Nginx, Inc. 的注册商标；本包是独立第三方构建，与 Nginx, Inc. 无隶属关系。

# 直播测试工具
内置了一个方便测试的pc端推流于播放的工具
![img](https://github.com/NodeMedia/NodeMediaDevClient/raw/master/QQ20160310-0.png)
源码在此:https://github.com/NodeMedia/NodeMediaDevClient

# 另一个选择，支持HTTP-FLV
基于Node.js实现,高性能,原生跨平台,支持RTMP/HTTP-FLV/GOPcache
https://github.com/illuspas/Node-Media-Server
