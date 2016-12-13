---
layout: post
title: Postgres 的启动 和 Mac 的开机启动脚本
---

记录一个今天遇到的很坑的问题：电脑重启后，postgres 没有重启，于是我去手动启动，然而一直启动失败。

网上有很多帖子说 postgres 启动命令为：

```
pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
```
注意这只是默认安装路径，你要找到你自己的 postgres 安装路径。
postgres 启动是要指明文件路径的，具体命令见以下（来自 `postgres --help`）：

```
pg_ctl start   [-w] [-t SECS] [-D DATADIR] [-s] [-l FILENAME] [-o "OPTIONS"]
```
我的 postgres 安装路径为：

```
~/Library/Application Support/Postgres/var-9.5
```
我一开始跑的命令如下：

```
pg_ctl -D ~/Library/Application Support/Postgres/var-9.5 -l ~/Library/Application Support/Postgres/var-9.5/postgres-server.log start
```
然而启动失败，报错如下：

```
pg_ctl: unrecognized operation mode "Support/Postgres/var-9.5"
```
这个报错一开始深深误导了我，我去查 unrecognized operation mode 查了好久，我怀疑是不是我的启动方式有问题。

后来我发现还有一种方式可以启动 postgres，不过是前台启动，于是我又死马当活马医地试了一下：

```
postgres -D ~/Library/Application Support/Postgres/var-9.5
```
启动也失败了，报错如下：

```
postgres: invalid argument: "Support/Postgres/var-9.5"
```
但是！这个报错比之前的良心太多了，因为通过 invalid argument 这个关键字我瞬间顿悟到了错误的根源，在于 `"Application Support"` 这个文件名包含了一个空格，导致启动命令以为传入了两个参数。

知道错误的原因后，通过以下启动命令成功启动 postgres:

```
pg_ctl -D "~/Library/Application Support/Postgres/var-9.5" -l "~/Library/Application Support/Postgres/var-9.5/postgres-server.log" start
```
---
---
在 Mac 上配置开机启动：(等我下次重启测试好结果再回来补)

~/Library/LaunchAgents

homebrew.mxcl.postgresql.plist

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>KeepAlive</key>
  <true/>
  <key>Label</key>
  <string>homebrew.mxcl.postgresql</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/opt/postgresql/bin/postgres</string>
    <string>-D</string>
    <string>/usr/local/var/postgres</string>
  </array>
  <key>RunAtLoad</key>
  <true/>
  <key>WorkingDirectory</key>
  <string>/usr/local</string>
  <key>StandardErrorPath</key>
  <string>/usr/local/var/log/postgres.log</string>
</dict>
</plist>
```
