# 在Unikraft部署哪吒监控 Dashboard
 - 项目：https://github.com/nezhahq/nezha
 - 文档：https://nezha.wiki/guide/dashboard.html
 - 网址：https://unikraft.com


# Unikraft特点

 - 🧠 最多 2 个 CPU 核心（2 vCPU）
 - 💾 最多 4GB 内存
 - 🖥️ 最多同时运行 2 个实例
 - 💽 有 8GB 存储空间
 - 📦 可以存 10GB 的镜像
 - ⏳ 可以闲置时自动“关机”，有请求时再快速启动


> 注意：只部署 **Dashboard 面板**。Agent 装在被监控的服务器上，不要装进 Unikraft。
>
> Agent 需持续上报，**必须关闭 scale-to-zero**（`policy=off`）。
>
> 默认版本 `latest`（跟随 GitHub 最新 Release）。固定版本可改 Dockerfile 的 `ARG VERSION=v2.3.4`。端口 `8008`，数据目录 `/dashboard/data`。

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
cd Nezha
```
# 第四步：打包并推送
```
unikraft build . \
  --output pustoas/nezha:latest \
  --no-cache \
  --timeout 1h
```

> 若报 `Client.Timeout exceeded`，去掉 `--no-cache` 再执行几次即可。
> 本地 metro 仓库（如 `index.sin.unikraft.cloud`）对 kernel 层常 `502`，不要用。
>
> **重新推送镜像后，必须先删再建实例**，否则会报名称冲突，且旧实例不会自动用上新镜像。见下方「更新镜像后重建实例」。

# 第五步：创建数据卷（已有可跳过）
```
unikraft volume create --metro sin --name nezha-data --size 1G
```
# 第六步：首次运行
```
unikraft run --metro sin \
  --name nezha \
  -m 4G \
  -p 443:8008/tls+http \
  --scale-to-zero policy=off \
  --volume nezha-data:/dashboard/data \
  --image pustoas/nezha:latest \
  --timeout 30m
```

# 更新镜像后重建实例
```
unikraft build . --output pustoas/nezha:latest --no-cache --timeout 1h
```
# 停止实例
```
unikraft instance stop nezha
```
# 删除实例
```
unikraft instance rm nezha
```
# 运行：
```
unikraft run --metro sin \
  --name nezha \
  -m 4G \
  -p 443:8008/tls+http \
  --scale-to-zero policy=off \
  --volume nezha-data:/dashboard/data \
  --image pustoas/nezha:latest \
  --timeout 30m
```

# 首次访问
```
unikraft instance get nezha
```
打开返回的 `https://….sin.unikraft.app`（管理后台一般是 `/dashboard`）。

# Agent 对接
1. 在面板系统设置里填写 Agent 对接地址：`你的域名:443`（Unikraft 对外是 HTTPS/443）。
2. 按面板生成的命令，在**被监控机器**上安装 Agent（不要装在 Unikraft 实例里）。
3. 若 Agent 连不上，检查 TLS/gRPC 是否走通；必要时参考官方文档调整对接方式：https://nezha.wiki/guide/agent.html

# 相关操作命令
# 查看运行中的实例（Instances）
```
unikraft instance list
```
# 查看实例详情（含域名）
```
unikraft instance get nezha
```
# 查看实例日志
```
unikraft instance logs nezha
```
# 停止实例（Instances）
```
unikraft instance stop nezha
```
# 启动已停止的实例（Instances）
```
unikraft instance start nezha
```
# 重启实例（先停再起，实例需仍存在）
```
unikraft instance restart nezha
```
# 删除实例（Instances）
```
unikraft instance rm nezha
```
# 列出所有卷（Volumes）
```
unikraft volume list
```
# 删除指定存储卷（Volumes）
```
unikraft volume rm nezha-data
```
# 列出所有镜像（Images）
```
unikraft image list
```
# 删除指定镜像（Images）
```
unikraft image rm pustoas/nezha
```
