# 在Unikraft部署Uptime Kuma
 - 项目：https://github.com/louislam/uptime-kuma
 - 网址：https://unikraft.com
 - 文档：https://unikraft.com/docs/introduction

# Unikraft特点

 - 🧠 最多 2 个 CPU 核心（2 vCPU）
 - 💾 最多 4GB 内存
 - 🖥️ 最多同时运行 2 个实例
 - 💽 有 8GB 存储空间
 - 📦 可以存 10GB 的镜像
 - ⏳ 可以闲置时自动“关机”，有请求时再快速启动

> 注意：Uptime Kuma 需要按间隔主动探测，**必须关闭 scale-to-zero**（`policy=off`）。
>
> 镜像为 `scratch` + 版本 `2.5.4`。Browser / Oracle / Mongo / MSSQL / gamedig / gRPC 等可选监控已 stub，HTTP/TCP/DNS/Ping 等仍可用。

# 第一步：安装官方cli
```
curl --proto '=https' --tlsv1.2 -fsSL https://unikraft.com/cli/install.sh | sh
```
# 第二步：登陆
```
unikraft login --no-browser
```
# 第三步：进入目录
```
cd Uptime-Kuma
```
# 第四步：打包并推送
```
unikraft build . \
  --output escau/uptime-kuma:latest \
  --no-cache \
  --timeout 1h
```

> 若报 `Client.Timeout exceeded`，去掉 `--no-cache` 再执行几次即可。
> 本地 metro 仓库（如 `index.sin.unikraft.cloud`）对 kernel 层常 `502`，不要用。
>
> **重新推送镜像后，必须先删再建实例**，否则会报 `An instance with the name 'uptime-kuma' already exists`，且旧实例不会自动用上新镜像。见下方「更新镜像后重建实例」。

# 第五步：创建数据卷（已有可跳过）
```
unikraft volume create --metro sin --name uptime-kuma-data --size 1G
```
# 第六步：首次运行
```
unikraft run --metro sin \
  --name uptime-kuma \
  -m 4G \
  -p 443:3001/tls+http \
  --scale-to-zero policy=off \
  --volume uptime-kuma-data:/app/data \
  --image escau/uptime-kuma:latest \
  --timeout 30m
```

# 更新镜像后重建实例
```
unikraft build . --output escau/uptime-kuma:latest --no-cache --timeout 1h
```
# 停止实例
```
unikraft instance stop uptime-kuma
```
# 删除实例
```
unikraft instance rm uptime-kuma
```
# 运行：
```
unikraft run --metro sin \
  --name uptime-kuma \
  -m 4G \
  -p 443:3001/tls+http \
  --scale-to-zero policy=off \
  --volume uptime-kuma-data:/app/data \
  --image escau/uptime-kuma:latest \
  --timeout 30m
```
# 首次访问
```
unikraft instance get uptime-kuma
```
打开返回的 `https://….sin.unikraft.app`，选择 SQLite，创建管理员账号。

# 常用命令
# 查看运行中的实例（Instances）
```
unikraft instance list
```
# 查看实例详情（含域名）
```
unikraft instance get uptime-kuma
```
# 查看实例日志
```
unikraft instance logs uptime-kuma
```
# 停止实例（Instances）
```
unikraft instance stop uptime-kuma
```
# 启动已停止的实例（Instances）
```
unikraft instance start uptime-kuma
```
# 重启实例（先停再起，实例需仍存在）
```
unikraft instance restart uptime-kuma
```
# 删除实例（Instances）
```
unikraft instance rm uptime-kuma
```
# 列出所有卷（Volumes）
```
unikraft volume list
```
# 删除指定存储卷（Volumes）
```
unikraft volume rm uptime-kuma-data
```
# 列出所有镜像（Images）
```
unikraft image list
```
# 删除指定镜像（Images）
```
unikraft image rm escau/uptime-kuma
```
