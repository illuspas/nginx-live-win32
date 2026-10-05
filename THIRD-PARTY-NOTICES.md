# 第三方组件与许可证声明

本发布包 **v2.0.1**（构建时间 `2026-10-05 21:50:16 +08:00`）包含下列组件，各组件版权归其
作者所有，按下列许可证分发。许可证全文随包附于 `LICENSES/` 目录。

| 组件 | 版本 | 许可证 | 全文 |
|---|---|---|---|
| nginx | 1.31.6 | BSD 2-Clause | [LICENSES/nginx.txt](LICENSES/nginx.txt) |
| nginx-live-module | 2.0.0 | Apache License 2.0 | [LICENSES/apache-2.0.txt](LICENSES/apache-2.0.txt) |
| nginx-live 服务管理器（nginx_service.exe） | 见 BUILD-INFO.txt | Apache License 2.0 | [LICENSES/apache-2.0.txt](LICENSES/apache-2.0.txt) |
| OpenSSL | 4.0.3 | Apache License 2.0 | [LICENSES/openssl.txt](LICENSES/openssl.txt) |
| PCRE2 | 10.49 | BSD 3-Clause（含 PCRE2 例外条款） | [LICENSES/pcre2.txt](LICENSES/pcre2.txt) |
| zlib | 1.3.1 | zlib License | [LICENSES/zlib.txt](LICENSES/zlib.txt) |

## nginx

```
Copyright (C) 2002-2021 Igor Sysoev
Copyright (C) 2011-2026 Nginx, Inc.
All rights reserved.
```

BSD 2-Clause License。本包内含 nginx 可执行文件，按该许可证要求随包复现其版权声明与
免责声明，完整条款见 [LICENSES/nginx.txt](LICENSES/nginx.txt)。

## nginx-live-module

Copyright (C) illuspas，Apache License 2.0。

本模块的 `rtmp { server { application { } } }` 配置结构、指令语义与部分实现思路沿用或
参考了 **nginx-rtmp-module**（Copyright (c) 2012-2014, Roman Arutyunyan，BSD 2-Clause）
的公开设计与约定；若本模块包含源自该项目的代码，则该部分适用 BSD 2-Clause，其版权与
免责声明须随分发保留。完整声明见模块仓库的 `NOTICE` 文件。

## OpenSSL / PCRE2 / zlib

均以源码形式编译进 `nginx.exe`。OpenSSL 自 3.0 起采用 Apache License 2.0；PCRE2 为
BSD 3-Clause（含 PCRE2 例外条款）；zlib 为 zlib License。许可证全文见上表。

## 商标

nginx 是 Nginx, Inc. 的注册商标。本发布包是独立的第三方构建，与 Nginx, Inc. 不存在
隶属、赞助或背书关系。

---

本文件由构建脚本生成（模板：`msvc-build\THIRD-PARTY-NOTICES.template.md`，执行：`publish.cmd`），
发布包内的副本请勿手工编辑。构建来源信息（版本、编译参数、exe SHA256）见同目录 `BUILD-INFO.txt`。
