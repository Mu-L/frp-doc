---
title: "配置文件"
weight: 1
---

frp 目前仅支持 ini 格式的配置文件，frps 和 frpc 各自支持不同的参数。

frps 主要配置服务端的一些通用参数，frpc 则需要额外配置每一个代理的详细配置。

## 格式

目前仅支持 ini 格式的配置，如下的示例配置将本地 SSH 服务穿透到公网。

frps 配置：

```ini
[common]
bind_port = 7000
```

frpc 配置：

```ini
[common]
server_addr = x.x.x.x
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

`[common]` 是固定名称的段落，用于配置通用参数。

`[ssh]` 仅在 frpc 中使用，用于配置单个代理的参数。代理名称必须唯一，不能重复。

同一个客户端可以配置多个代理。

## 模版渲染

配置文件支持使用环境变量进行模版渲染，模版格式采用 Go 的标准格式。

示例配置如下：

```ini
# frpc.ini
[common]
server_addr = {{ .Envs.FRP_SERVER_ADDR }}
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = {{ .Envs.FRP_SSH_REMOTE_PORT }}
```

启动 frpc 程序：

```
export FRP_SERVER_ADDR="x.x.x.x"
export FRP_SSH_REMOTE_PORT="6000"
./frpc -c ./frpc.ini
```

frpc 会自动使用环境变量渲染配置文件模版，所有环境变量需要以 `.Envs` 为前缀。

## 配置校验

通过执行 `frpc verify -c ./frpc.ini` 或 `frps verify -c ./frps.ini` 可以对配置文件中的参数进行预先校验。

```
frpc: the configuration file ./frpc.ini syntax is ok
```

如果出现此结果，则说明新的配置文件没有错误，否则会输出具体的错误信息。

## 配置拆分

通过 `includes` 参数可以在主配置中包含其他配置文件，从而实现将代理配置拆分到多个文件中管理。

```ini
# frpc.ini
[common]
server_addr = x.x.x.x
server_port = 7000
includes = ./confd/*.ini
```

```ini
# ./confd/test.ini
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

上述配置在 frpc.ini 中通过 includes 额外包含了 `./confd` 目录下所有的 ini 文件的代理配置内容，效果等价于将这两个文件合并成一个文件。

需要注意的是 includes 指定的文件中只能包含代理配置，common 段落的配置只能放在主配置文件中。

## 完整配置参数

由于 frp 目前支持的功能和配置项较多，未在文档中列出的功能参数可以在 [参考](/docs/reference) 中查看。
