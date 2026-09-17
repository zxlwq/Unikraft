# 在Unikraft部署 LunaTV（个人用 + Upstash）
 - 网址：https://unikraft.com
 - 文档：https://unikraft.com/docs/introduction
 - 项目：https://github.com/MoonTechLab/LunaTV

# Unikraft特点

 - 🧠 最多 2 个 CPU 核心（2 vCPU）
 - 💾 最多 4GB 内存
 - 🖥️ 最多同时运行 2 个实例
 - 💽 有 8GB 存储空间
 - 📦 可以存 10GB 的镜像
 - ⏳ 可以闲置时自动“关机”，有请求时再快速启动

> 构建时拉取 `ghcr.io/moontechlab/lunatv:latest`，推到 Unikraft 仓库后运行。
> **存储用 Upstash**（云端 Redis），Unikraft 上只跑 1 个 LunaTV 实例，不另起 Kvrocks/Redis。
> 端口 `3000`。启动命令用 `/usr/local/bin/node`（勿用 `/usr/bin/node`，会 ENOENT 闪退）。
> 个人用可用 `scale-to-zero policy=idle`。
> 部署后是空壳，需在后台自行配置播放源。协议 CC BY-NC-SA，仅限非商业个人使用。
>
> **域名固定**：不要用 `run -p ...` 临时建 service（删实例域名会变）。先建持久 `lunatv-svc`，再用 `--service lunatv-svc` 挂载，重建实例域名不变，`SITE_BASE` 也可写死。

# 第零步：准备密钥（不要写进文档 / 不要提交 git）
1. 打开 https://upstash.com/ 注册并新建 Redis 数据库，复制 HTTPS ENDPOINT 和 TOKEN
2. 复制示例文件并只在本机填写：
```
cp lunatv.env.example lunatv.env
```
3. 编辑 `lunatv.env`（已在 `.gitignore`）。文档和 Kraftfile 里只留占位符。

> 若密码或 Token 曾经写进过公开 md / 聊天记录，到 Upstash 控制台 **Reset Token**，并改掉站长密码。

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
cd LunaTV
```
# 第四步：打包并推送
```
unikraft build . --output howatruha/lunatv:latest
```

> 若报 `Client.Timeout exceeded`，加长超时再试：
> ```
> unikraft build . --output howatruha/lunatv:latest --timeout 1h
> ```
>
> **换新镜像时只需删实例再建**，**不要删** `lunatv-svc`，否则域名会变。

# 第五步：创建持久 Service（只做一次，用来固定域名）
```
unikraft services create --metro sin \
  --name lunatv-svc \
  --service 443:3000/tls+http
```

查看并记下域名（形如 `https://lunatv-svc-xxxx.sin.unikraft.app`）：
```
unikraft services get lunatv-svc
```

# 第六步：首次运行（挂到 lunatv-svc）
```
set -a
source ./.env
set +a
```
```
unikraft run --metro sin \
  --name lunatv \
  -m 4G \
  --service lunatv-svc \
  --scale-to-zero policy=idle,cooldown-time=1000,stateful=true \
  -e USERNAME="$USERNAME" \
  -e PASSWORD="$PASSWORD" \
  -e NEXT_PUBLIC_STORAGE_TYPE="$NEXT_PUBLIC_STORAGE_TYPE" \
  -e UPSTASH_URL="$UPSTASH_URL" \
  -e UPSTASH_TOKEN="$UPSTASH_TOKEN" \
  -e NEXT_PUBLIC_SITE_NAME="$NEXT_PUBLIC_SITE_NAME" \
  -e SITE_BASE="$SITE_BASE" \
  --image howatruha/lunatv:latest
```

> 数据在 Upstash，一般**不需要** Unikraft volume。

# 更新镜像后重建实例（域名不变）
```
unikraft build . --output howatruha/lunatv:latest --no-cache --timeout 1h
```
# 只删实例，不要删 lunatv-svc
```
unikraft instance stop lunatv
```
```
unikraft instance rm lunatv
```
```
set -a
source ./.env
set +a
```
```
unikraft run --metro sin \
  --name lunatv \
  -m 4G \
  --service lunatv-svc \
  --scale-to-zero policy=idle,cooldown-time=1000,stateful=true \
  -e USERNAME="$USERNAME" \
  -e PASSWORD="$PASSWORD" \
  -e NEXT_PUBLIC_STORAGE_TYPE="$NEXT_PUBLIC_STORAGE_TYPE" \
  -e UPSTASH_URL="$UPSTASH_URL" \
  -e UPSTASH_TOKEN="$UPSTASH_TOKEN" \
  -e NEXT_PUBLIC_SITE_NAME="$NEXT_PUBLIC_SITE_NAME" \
  -e SITE_BASE="$SITE_BASE" \
  --image howatruha/lunatv:latest
```

# 首次访问
```
unikraft services get lunatv-svc
```
或
```
unikraft instance get lunatv
```
打开 `SITE_BASE` 对应地址，用 `USERNAME` / `PASSWORD` 登录后台配置播放源。

# 相关操作命令
# 查看运行中的实例（Instances）
```
unikraft instance list
```
# 查看实例详情（含域名）
```
unikraft instance get lunatv
```
# 查看 Service（固定域名）
```
unikraft services list
unikraft services get lunatv-svc
```
# 查看实例日志
```
unikraft instance logs lunatv
```
# 停止实例（Instances）
```
unikraft instance stop lunatv
```
# 启动已停止的实例（Instances）
```
unikraft instance start lunatv
```
# 重启实例（先停再起，实例需仍存在）
```
unikraft instance restart lunatv
```
# 删除实例（Instances）——不会删掉 lunatv-svc
```
unikraft instance rm lunatv
```
# 删除 Service（会丢掉固定域名，一般不用）
```
unikraft services delete lunatv-svc
```
# 列出所有镜像（Images）
```
unikraft image list
```
# 删除指定镜像（Images）
```
unikraft image rm howatruha/lunatv
```
