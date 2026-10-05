nginx-rtmp-win32
================

Windows 绿色包：nginx + nginx-live-module（RTMP / HTTP-FLV 直播），
另附 `nginx_service.exe`（把 nginx 注册为 Windows 服务，并提供托盘管理界面）。

版本号、编译参数、exe 大小与 SHA256 由构建脚本自动生成在 **BUILD-INFO.txt**，
请以该文件为准（不要手工编辑、也不要在这里手写版本）。

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

# 直播测试工具
内置了一个方便测试的pc端推流于播放的工具
![img](https://github.com/NodeMedia/NodeMediaDevClient/raw/master/QQ20160310-0.png)
源码在此:https://github.com/NodeMedia/NodeMediaDevClient

# 另一个选择，支持HTTP-FLV
基于Node.js实现,高性能,原生跨平台,支持RTMP/HTTP-FLV/GOPcache
https://github.com/illuspas/Node-Media-Server
