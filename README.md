nginx-live-win32
================

## 简介

nginx-live-win32 是 [nginx](https://nginx.org/) + [nginx-live-module](https://github.com/illuspas/nginx-live-module) 的 Windows 绿色包（解压即用，免安装），
实现高性能、低延迟的开源直播流媒体服务器，支持 **RTMP / Enhanced RTMP / HTTP-FLV** 推流与播放。

包内另附 `nginx_service.exe`：把 nginx 注册为 Windows 服务，并提供系统托盘管理界面。

版本号、编译参数、exe 大小与 SHA256 由构建脚本自动生成在 **BUILD-INFO.txt**，
请以该文件为准（不要手工编辑、也不要在这里手写版本）。

## 特性

* RTMP 推流/播放（兼容 OBS、FFmpeg 等）
* Enhanced RTMP v1：原生支持 HEVC (H.265)、AV1、VP9、Opus，无需转码直接转发
* HTTP-FLV 推流（POST）/ 播放（GET），HTTP/1.1 chunked
* WS/WSS-flv 不支持（当前版本仅 HTTP-FLV）
* GOP 缓存，播放端从最近关键帧秒开（≤ 1 个 GOP）
* 访问控制：`allow/deny publish|play`（支持 CIDR，RTMP 与 HTTP-FLV 共用）
* JSON 状态接口 `/stat`：流列表、编码格式、码量、客户端数等实时统计
* 慢消费者保护：待发队列超限丢帧重同步，持续停滞断开单个订阅者，不影响他人
* 静态文件服务（nginx 原生能力，`html/` 目录）
* Windows 服务 + 托盘管理（安装、启动、停止、卸载）

## 下载

* **推荐**：到 [Releases](../../releases) 页下载 `nginx-live-<版本>-win32.zip`（绿色包，解压即用），
  同页附 `SHA256SUMS.txt` 校验和；每个版本的构建来源与组件版本见 Release 说明。
* 也可以直接 clone 本仓库：仓库内提交的就是同一份可运行包。

## 使用方法

### 快速开始

* 直接运行：双击 `nginx.exe`
* 作为服务/托盘管理：运行 `nginx_service.exe`（安装、启动、停止、卸载、打开配置/页面）
* 命令行停止：`nginx.exe -s stop`
* 重新加载配置：`nginx.exe -s reload`

默认端口（见 `conf/nginx.conf`）：

| 端口 | 协议 | 用途 |
|---|---|---|
| 1935 | RTMP | 推流 / 拉流，application `live` |
| 8080 | HTTP | HTTP-FLV 拉流 / 推流、状态页 |

### 推流

OBS：设置 → 推流 → 自定义服务器，填 `rtmp://<服务器IP>:1935/live`，串流密钥任意（即流名）。

FFmpeg：

```bash
# H.264 + AAC
ffmpeg -re -i input.mp4 -c copy -f flv rtmp://127.0.0.1:1935/live/mystream

# HEVC/AV1/VP9/Opus 源直接 copy（Enhanced RTMP 封装，无需转码）
ffmpeg -re -i input_265.mp4 -c copy -f flv rtmp://127.0.0.1:1935/live/hevc
```

也支持 HTTP-FLV 推流（POST）：

```bash
ffmpeg -re -i input.mp4 -c copy -f flv http://127.0.0.1:8080/live/mystream
```

### 播放

```bash
# RTMP
ffplay rtmp://127.0.0.1:1935/live/mystream

# HTTP-FLV
ffplay http://127.0.0.1:8080/live/mystream.flv
```

浏览器播放可配合支持 HTTP-FLV 的播放器，如
[NodePlayer.js](https://www.nodemedia.cn/product/nodeplayer-js/)（原生支持 Enhanced-FLV H.265）。

### 状态查看

```bash
curl -s http://127.0.0.1:8080/stat
```

返回 JSON：应用与流列表、是否在推、客户端数、音视频编码、收发字节数、运行时长等。

## 配置说明

`conf/nginx.conf` 为配置文件实例：

```nginx
rtmp {
    server {
        listen 1935;              # RTMP 监听
        gop_cache  on;            # GOP 缓存（秒开）
        chunk_size 65535;

        application live {
            live             on;  # 推拉开关
            # allow publish 127.0.0.1;   # 可选：访问控制
            # deny  publish all;
        }
    }
}

http {
    server {
        listen 8080;

        location ~ ^/[a-z0-9_]+/.+(\.flv)?$ {
            live_flv  on;         # HTTP-FLV 推/拉流端点 /<app>/<stream>[.flv]
        }

        location = /stat {
            live_stat  on;        # JSON 状态接口
        }
    }
}
```

常用指令（完整说明见模块文档 `docs/usage.md`）：

| 指令 | 默认 | 说明 |
|------|------|------|
| `listen <addr:port>` | — | RTMP 监听地址，可多条 |
| `live on\|off` | `off` | 关闭时发布/播放返回 503 |
| `gop_cache on\|off` | `on` | GOP 缓存，秒开 |
| `chunk_size <n>` | `65535` | RTMP 出站 chunk size |
| `timeout <t>` | `60s` | 握手/发送停滞/发布空闲超时 |
| `allow/deny [publish\|play] <addr>\|all` | 放行 | 按书写顺序匹配，支持 CIDR |
| `live_flv on\|off` | `off` | HTTP 端启用 HTTP-FLV 推/拉 |
| `live_stat on\|off` | `off` | HTTP 端启用 `/stat` |

注意：流表为 per-worker，生产环境建议保持 `worker_processes 1`。

## 支持的编码

| 类型 | legacy FLV | Enhanced RTMP v1（FourCC） |
|------|-----------|---------------------------|
| 视频 | H.264 | HEVC `hvc1`、AV1 `av01`、VP9 `vp09` |
| 音频 | AAC、MP3 等 | Opus、ac-3、ec-3、`.mp3`、fLaC |

## 已知限制

* 不支持 Flash 时代的旧式 RTMP 扩展 H.265（如 `cdn_cn` flv_265 标准）
* AMF3 命令（objectEncoding=3）不支持
* reload 后新 worker 流表为空，已有流对新连接不可见（长连接场景可配置 `worker_shutdown_timeout`）

## 许可与来源

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
