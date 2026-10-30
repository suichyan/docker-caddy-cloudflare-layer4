# Caddy-Cloudflare

[English](README.md) | [简体中文](README.zh-CN.md)

[![Docker Hub](https://img.shields.io/badge/Docker%20Hub-suichyan%2Fcaddy--cloudflare--layer4-blue?style=flat)](https://hub.docker.com/r/suichyan/caddy-cloudflare-layer4)
[![version](https://img.shields.io/docker/v/suichyan/caddy-cloudflare-layer4?label=version)](https://hub.docker.com/r/suichyan/caddy-cloudflare-layer4/tags)
[![GitHub build status](https://img.shields.io/github/actions/workflow/status/suichyan/docker-caddy-cloudflare-layer4/docker.yml?style=flat)](https://github.com/suichyan/docker-caddy-cloudflare-layer4/actions/workflows/docker.yml)
[![License](https://img.shields.io/github/license/suichyan/docker-caddy-cloudflare-layer4)](https://github.com/suichyan/docker-caddy-cloudflare-layer4/blob/main/LICENSE)

> 本项目从 https://github.com/IAreKyleW00t/docker-caddy-cloudflare fork 并精简而来。

基于官方 [Caddy](https://hub.docker.com/_/caddy) 镜像，额外加入两个模块：[caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare) 用于 DNS-01 ACME 校验，[mholt/caddy-l4](https://github.com/mholt/caddy-l4) 用于 Layer 4（TCP/UDP）代理。除替换 `caddy` 二进制外不修改镜像的任何内容。仅构建 `linux/amd64`。

```sh
# Docker Hub
docker pull suichyan/caddy-cloudflare-layer4:latest

# GHCR
docker pull ghcr.io/suichyan/caddy-cloudflare-layer4:latest
```

## 标签

`suichyan/caddy-cloudflare-layer4` 镜像提供以下标签：

- `latest`

## 使用

由于基于官方镜像构建，官方镜像的所有 Volume、环境变量等用法都原样适用。完整用法请参考官方 [Caddy](https://hub.docker.com/_/caddy) 镜像和[文档](https://caddyserver.com/docs/)。

按常规方式创建容器，并带上 `CF_API_TOKEN`（API Token 已不再需要邮箱）。利用 Caddy 对[环境变量](https://caddyserver.com/docs/caddyfile/concepts#environment-variables)的支持，把这些值传进 `Caddyfile` 即可。

```sh
docker run --rm -it \
  --name caddy \
  -p 80:80 \
  -p 443:443 \
  -v caddy_data:/data \
  -v caddy_config:/config \
  -v $PWD/Caddyfile:/etc/caddy/Caddyfile \
  -e CF_API_TOKEN=UhKLc...JD9jk \
  suichyan/caddy-cloudflare-layer4:latest
```

然后在 `Caddyfile` 里设置全局的 [acme_dns](https://caddyserver.com/docs/caddyfile/options#acme-dns) 指令：

```Caddyfile
tls {
    dns cloudflare {env.CF_API_TOKEN}
}
```

或用 JSON 形式：

```json
{
  "module": "acme",
  "challenges": {
    "dns": {
      "provider": {
        "name": "cloudflare",
        "api_token": "{env.CF_API_TOKEN}"
      }
    }
  }
}
```

进阶用法参见 [caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare) 模块和 [`tls`](https://caddyserver.com/docs/caddyfile/directives/tls#tls) 指令文档。

### 创建 Cloudflare API Token

在 Cloudflare 网页控制台按以下步骤生成 API Token：

1. 登录 [Dashboard](https://dash.cloudflare.com/)
2. 进入 [Account Profile](https://dash.cloudflare.com/profile) > [API Tokens](https://dash.cloudflare.com/profile/api-tokens)
3. 点击 "Create token"（选择 "Create Custom Token"）
4. 授予以下权限：
   - `Zone > Zone > Read`
   - `Zone > DNS > Edit`

## Layer 4

[mholt/caddy-l4](https://github.com/mholt/caddy-l4) 提供了 `layer4` 应用，可以代理原始 TCP 与 UDP 流量——SSH、MQTT、游戏服务器，或者按 SNI 分流而不在 HTTP 层终止的 TLS 流量。

`layer4` 是顶层应用，因此写在全局选项块里，而不是站点块里：

```Caddyfile
{
	layer4 {
		:8443 {
			route {
				proxy tcp/localhost:9443
			}
		}
	}
}
```

监听 TCP 用 `:8443`，监听 UDP 用 `udp/:8443`。部署前建议先校验配置，因为 caddy-l4 在 v0.2 到 v0.3 之间改过 Caddyfile 语法：

```sh
docker run --rm -v $PWD/Caddyfile:/etc/caddy/Caddyfile \
  suichyan/caddy-cloudflare-layer4:latest caddy validate --config /etc/caddy/Caddyfile
```

## 构建

本地构建镜像：

```sh
docker build -t caddy-cloudflare-layer4 .
```

## 贡献

欢迎到上游项目提 [Issue](https://github.com/IAreKyleW00t/docker-caddy-cloudflare/issues) 或 [Pull Request](https://github.com/IAreKyleW00t/docker-caddy-cloudflare/pulls)。

## 许可证

见 [LICENSE](https://github.com/suichyan/docker-caddy-cloudflare-layer4/blob/main/LICENSE)。
