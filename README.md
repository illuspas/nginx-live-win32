nginx-live-win32
================

[简体中文](README_CN.md) | **English**

## Introduction

nginx-live-win32 is a portable ("green", unzip-and-run, no installation) Windows build of
[nginx](https://nginx.org/) + [nginx-live-module](https://github.com/illuspas/nginx-live-module).
It provides a high-performance, low-latency open-source live streaming server supporting
**RTMP / Enhanced RTMP / HTTP-FLV** publishing and playback.

The package also ships `nginx_service.exe`, which registers nginx as a Windows service and
provides a system tray management UI.

The version number, build flags, executable size and SHA256 are generated automatically by the
build scripts into **BUILD-INFO.txt**. That file is authoritative — do not edit it by hand, and
do not hard-code versions here.

## Features

* RTMP publishing/playback (compatible with OBS, FFmpeg, etc.)
* Enhanced RTMP v1: native HEVC (H.265), AV1, VP9 and Opus support, forwarded without transcoding
* HTTP-FLV publishing (POST) / playback (GET) with HTTP/1.1 chunked transfer
* WS/WSS-FLV publishing / playback
* GOP cache, so players start instantly from the most recent keyframe (≤ 1 GOP)
* Access control: `allow/deny publish|play` (CIDR supported, shared by RTMP and HTTP-FLV)
* JSON status endpoint `/stat`: live stream list, codecs, bitrate, client counts and more
* Slow-consumer protection: drop-and-resync when the send queue overflows, and disconnect a
  single stalled subscriber without affecting others
* Static file serving (native nginx capability, `html/` directory)
* Windows service + tray management (install, start, stop, uninstall)

## Download

* **Recommended**: download `nginx-live-<version>-win32.zip` from the [Releases](../../releases)
  page (portable package, unzip and run). The same page carries `SHA256SUMS.txt` checksums;
  build provenance and component versions for each release are in the release notes.
* Alternatively, clone this repository directly: what is committed here is the same runnable package.

## Usage

### Quick start

* Run directly: double-click `nginx.exe`
* Service / tray management: run `nginx_service.exe` (install, start, stop, uninstall, open
  config/pages)
* Stop from the command line: `nginx.exe -s stop`
* Reload configuration: `nginx.exe -s reload`

Default ports (see `conf/nginx.conf`):

| Port | Protocol | Purpose |
|---|---|---|
| 1935 | RTMP | Publish / play, application `live` |
| 8080 | HTTP | HTTP-FLV play / publish, status page |

### Publishing

OBS: Settings → Stream → Custom server, enter `rtmp://<server-ip>:1935/live`, and any stream key
(i.e. the stream name).

FFmpeg:

```bash
# H.264 + AAC
ffmpeg -re -i input.mp4 -c copy -f flv rtmp://127.0.0.1:1935/live/mystream

# HEVC/AV1/VP9/Opus sources copied as-is (Enhanced RTMP encapsulation, no transcoding)
ffmpeg -re -i input_265.mp4 -c copy -f flv rtmp://127.0.0.1:1935/live/hevc
```

HTTP-FLV publishing (POST) is also supported:

```bash
ffmpeg -re -i input.mp4 -c copy -f flv http://127.0.0.1:8080/live/mystream
```

### Playback

```bash
# RTMP
ffplay rtmp://127.0.0.1:1935/live/mystream

# HTTP-FLV
ffplay http://127.0.0.1:8080/live/mystream.flv
```

For browser playback, pair with an HTTP-FLV capable player such as
[NodePlayer.js](https://www.nodemedia.cn/product/nodeplayer-js/) (native Enhanced-FLV H.265 support).

### Status

```bash
curl -s http://127.0.0.1:8080/stat
```

Returns JSON: applications and stream list, whether publishing, client count, audio/video codecs,
bytes sent/received, uptime and more.

## Configuration

`conf/nginx.conf` is the example configuration:

```nginx
rtmp {
    server {
        listen 1935;              # RTMP listener
        gop_cache  on;            # GOP cache (instant start)
        chunk_size 65535;

        application live {
            live             on;  # publish/play switch
            # allow publish 127.0.0.1;   # optional: access control
            # deny  publish all;
        }
    }
}

http {
    server {
        listen 8080;

        location ~ ^/[a-z0-9_]+/.+(\.flv)?$ {
            live_flv  on;         # HTTP-FLV publish/play endpoint /<app>/<stream>[.flv]
        }

        location = /stat {
            live_stat  on;        # JSON status endpoint
        }
    }
}
```

Common directives (see the module docs `docs/usage.md` for the full reference):

| Directive | Default | Description |
|------|------|------|
| `listen <addr:port>` | — | RTMP listen address, may be repeated |
| `live on\|off` | `off` | When off, publish/play return 503 |
| `gop_cache on\|off` | `on` | GOP cache, instant start |
| `chunk_size <n>` | `65535` | RTMP outbound chunk size |
| `timeout <t>` | `60s` | Handshake / send-stall / publisher-idle timeout |
| `allow/deny [publish\|play] <addr>\|all` | allow | Matched in written order, CIDR supported |
| `live_flv on\|off` | `off` | Enable HTTP-FLV publish/play on the HTTP side |
| `live_stat on\|off` | `off` | Enable `/stat` on the HTTP side |

Note: the stream table is per-worker, so `worker_processes 1` is recommended in production.

## Supported codecs

| Type | Legacy FLV | Enhanced RTMP v1 (FourCC) |
|------|-----------|---------------------------|
| Video | H.264 | HEVC `hvc1`, AV1 `av01`, VP9 `vp09` |
| Audio | AAC, MP3, etc. | Opus, ac-3, ec-3, `.mp3`, fLaC |

## Known limitations

* Legacy Flash-era RTMP-extension H.265 (e.g. the `cdn_cn` flv_265 standard) is not supported
* AMF3 commands (objectEncoding=3) are not supported
* After a reload the new worker's stream table is empty, so existing streams are not visible to
  new connections (for long-lived connection scenarios, configure `worker_shutdown_timeout`)

## Supported clients

| Client | H.264 | HEVC | VP9 | AV1 |
| ------------ | ------------ |------------ |------------ |------------ |
| OBS_29.1+ | ✅ | ✅ | ❌ | ✅ |
| FFmpeg/FFplay_6.1+ | ✅ | ✅ | ✅ | ✅ |
| NodePlayer.js_1.0+ | ✅ | ✅ | ❌ | ❌ |
| NodeMediaClient_3.0+ | ✅ | ✅ | ❌ | ❌ |

### [QLive](https://play.google.com/store/apps/details?id=cn.nodemedia.qlive)
Free Android Live Streaming App

### [NodePlayer.js](https://www.nodemedia.cn/product/nodeplayer-js/) pure javascript implementation live streaming player
[Online Demo](http://demo.nodemedia.cn/)
- ASM.js, WASM, SIMD, WebWorker, WebCodecs, MediaSource multiple technical implementations
- H.264/H.265+AAC/G711 software and hardware decoder
- Ultra-low latency, Under extreme conditions less than 100 milliseconds
- Enhanced HTTP/WS-FLV Protocol, Natively support h.265
- Android/iOS/HarmonyOS/Chrome/Edge/Firefox/Safari, All modern browsers or platforms

### [NodePublisher.js](https://www.nodemedia.cn/demo/nodepublisher/) pure javascript implementation live streaming publisher
- WebSocket-FLV Protocol
- H.264+AAC hardware encoder
- Only chrome or chromium based browsers are supported at the moment
- wss is required

### [NodeMediaClient-iOS](https://github.com/NodeMedia/NodeMediaClient-iOS)  iOS live streaming player and publisher SDK
- Objective-C/Swift
- RTMP/HTTP-FLV/RTSP
- H.264/H.265+AAC/OPUS/G711
- Ultra-low latency, Under extreme conditions less than 100 milliseconds
- Enhanced RTMP/FLV Protocol, Natively support H.265/OPUS
- Built-in beauty filter

### [NodeMediaClient-Android](https://github.com/NodeMedia/NodeMediaClient-Android)  Android live streaming player and publisher SDK
- JAVA/Kotlin
- armv7/arm64/x86/x86_64
- RTMP/HTTP-FLV/RTSP
- H.264/H.265+AAC/OPUS/G711
- Ultra-low latency, Under extreme conditions less than 100 milliseconds
- Enhanced RTMP/FLV Protocol, Natively support H.265/OPUS
- Built-in beauty filter

### [expo-nodemediaclient](https://github.com/NodeMedia/expo-nodemediaclient)  Expo module for NodeMediaClient
- iOS and Android
- player and publisher

## License and provenance

This package is a combined binary of nginx + nginx-live-module + the service manager. The full
license texts for every component ship with the package in the `LICENSES/` directory; the complete
inventory and copyright notices are in `THIRD-PARTY-NOTICES.md`:

| Component | License |
|---|---|
| nginx | BSD 2-Clause |
| nginx-live-module, service manager | Apache License 2.0 |
| OpenSSL | Apache License 2.0 |
| PCRE2 | BSD 3-Clause (with the PCRE2 exception clause) |
| zlib | zlib License |

nginx is a registered trademark of Nginx, Inc.; this package is an independent third-party build
and is not affiliated with Nginx, Inc.
