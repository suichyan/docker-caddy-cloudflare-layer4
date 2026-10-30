# Caddy-Cloudflare

[English](README.md) | [简体中文](README.zh-CN.md)

[![Docker Hub](https://img.shields.io/badge/Docker%20Hub-suichyan%2Fcaddy--cloudflare--layer4-blue?style=flat)](https://hub.docker.com/r/suichyan/caddy-cloudflare-layer4)
[![version](https://img.shields.io/docker/v/suichyan/caddy-cloudflare-layer4?label=version)](https://hub.docker.com/r/suichyan/caddy-cloudflare-layer4/tags)
[![GitHub build status](https://img.shields.io/github/actions/workflow/status/suichyan/docker-caddy-cloudflare-layer4/docker.yml?style=flat)](https://github.com/suichyan/docker-caddy-cloudflare-layer4/actions/workflows/docker.yml)
[![License](https://img.shields.io/github/license/suichyan/docker-caddy-cloudflare-layer4)](https://github.com/suichyan/docker-caddy-cloudflare-layer4/blob/main/LICENSE)

> Project forked and simplified from https://github.com/IAreKyleW00t/docker-caddy-cloudflare

The official [Caddy](https://hub.docker.com/_/caddy) Docker image with two added modules: [caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare) for DNS-01 ACME validation and [mholt/caddy-l4](https://github.com/mholt/caddy-l4) for Layer 4 (TCP/UDP) proxying. This image does not change anything with Caddy except replacing the `caddy` binary. Built for `linux/amd64` only.

```sh
# Docker Hub
docker pull suichyan/caddy-cloudflare-layer4:latest

# GHCR
docker pull ghcr.io/suichyan/caddy-cloudflare-layer4:latest
```

## Tags

The following tags are available for the `suichyan/caddy-cloudflare-layer4` image.

- `latest`

## Usage

Since this is built off the official Docker image all of the same Volumes, Environment variables, etc. can be used with this container. Please refer to the official [Caddy](https://hub.docker.com/_/caddy) Docker image and [docs](https://caddyserver.com/docs/) for more information on using Caddy.

Simply create the container as usual and include your `CF_API_TOKEN` (email no longer required for API Tokens). We can utilizing Caddy's support for [Environment varaiables](https://caddyserver.com/docs/caddyfile/concepts#environment-variables) to pass these values into our `Caddyfile`.

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

Then set the global [acme_dns](https://caddyserver.com/docs/caddyfile/options#acme-dns) directive in your `Caddyfile`

```Caddyfile
tls {
    dns cloudflare {env.CF_API_TOKEN}
}
```

or via JSON

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

See the [caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare) module and [`tls`](https://caddyserver.com/docs/caddyfile/directives/tls#tls) directive for advanced usage.

### Creating a Cloudflare API Token

You can generate a Cloudflare API token via the Cloudflare web console using the following steps:

1. Login to your [Dashboard](https://dash.cloudflare.com/)
2. Go to [Account Profile](https://dash.cloudflare.com/profile) > [API Tokens](https://dash.cloudflare.com/profile/api-tokens)
3. Click "Create token" (Use the "Create Custom Token" option)
4. Grant the following permissions:
   - `Zone > Zone > Read`
   - `Zone > DNS > Edit`

## Layer 4

The [mholt/caddy-l4](https://github.com/mholt/caddy-l4) module ships a `layer4` app that proxies raw TCP and UDP — SSH, MQTT, game servers, or TLS traffic routed by SNI without terminating it at an HTTP server.

`layer4` is a top-level app, so it goes in the global options block instead of a site block:

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

Use a bare `:8443` for TCP and `udp/:8443` for UDP. Validate before deploying, since the Caddyfile syntax changed between caddy-l4 v0.2 and v0.3:

```sh
docker run --rm -v $PWD/Caddyfile:/etc/caddy/Caddyfile \
  suichyan/caddy-cloudflare-layer4:latest caddy validate --config /etc/caddy/Caddyfile
```

## Building

You can easily build the Docker image locally by doing

```sh
docker build -t caddy-cloudflare-layer4 .
```

## Contributing

Feel free to contribute and make things better by opening an [Issue](https://github.com/IAreKyleW00t/docker-caddy-cloudflare/issues) or [Pull Request](https://github.com/IAreKyleW00t/docker-caddy-cloudflare/pulls) at the parent project.

## License

See [LICENSE](https://github.com/suichyan/docker-caddy-cloudflare-layer4/blob/main/LICENSE).
