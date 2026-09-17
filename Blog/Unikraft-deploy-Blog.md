# 在Unikraft部署 Blog（nginx + Docker Hub 内容）
 - 网址：https://unikraft.com
 - 文档：https://unikraft.com/docs/introduction


# Unikraft特点

 - 🧠 最多 2 个 CPU 核心（2 vCPU）
 - 💾 最多 4GB 内存
 - 🖥️ 最多同时运行 2 个实例
 - 💽 有 8GB 存储空间
 - 📦 可以存 10GB 的镜像
 - ⏳ 可以闲置时自动“关机”，有请求时再快速启动


> 构建时从 Docker Hub 拉取 `zxlwq/blog:latest` 的 `/usr/share/html` 与 `/etc/nginx/conf.d/default.conf`。
> **nginx 配置只维护博客仓库一份**：`docker/default.conf`（勿在 `Blog/` 再放一份）。
> 端口 `3000`，站点目录 `/usr/share/html`。
> Unikraft 上必须用 `master_process off`（见 Kraftfile），且 `default.conf` 把日志打到 stdout/stderr，否则 Logs 会是 `No log output`。
> 可用 `scale-to-zero policy=idle`（有访问再唤醒；看日志前先打开一次站点）。
>
> **域名固定（重要）**：博客更新频繁时，不要用 `run -p ...` 临时建 service（删实例域名会变，Cloudflare `AI_ALLOWED_ORIGINS` 也要跟着改）。
> 先建持久 `blog-svc`，再用 `--service blog-svc` 挂载；换镜像只删实例、**不删** service，域名不变，Cloudflare **只配一次**。

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
cd Blog
```
# 第四步：打包并推送
```
unikraft build . --output fairsxto/blog:latest
```

> 若报 `Client.Timeout exceeded`，再执行几次即可。
>
若你先更新了 Docker Hub 上的 `Docker镜像`，这里需要加 `--no-cache` 重新拉 Hub 层再打包：
```
unikraft build . --output fairsxto/blog:latest --no-cache --timeout 1h
```

# 第五步：创建持久Service（只做一次，用来固定域名）
```
unikraft services create --metro sin \
  --name blog-svc \
  --service 443:3000/tls+http
```

查看并记下域名（形如 `https://blog-svc-xxxx.sin.unikraft.app`）：
```
unikraft services get blog-svc
```

# 第六步：首次运行（挂到 blog-svc）
```
unikraft run --metro sin \
  --name blog \
  -m 4G \
  --service blog-svc \
  --scale-to-zero policy=idle,cooldown-time=1000,stateful=true \
  --image fairsxto/blog:latest \
  --timeout 30m
```

> 纯静态站一般不需要 volume。站点文件在镜像的 `/usr/share/html` 里。

# 第七步：Cloudflare Pages 的 AI_ALLOWED_ORIGINS（只做一次）
把第五步记下的**固定域名**写入 Cloudflare Pages 环境变量 `AI_ALLOWED_ORIGINS`。
之后无论博客更新多少次、实例重建多少次，只要不删 `blog-svc`，就不用再改这项。

# 更新文章 / Hub 镜像后重建实例（域名不变）

# 1. Hub 有新内容时再 --no-cache；仅 Unikraft 包装变更可去掉 --no-cache
```
unikraft build . --output fairsxto/blog:latest --no-cache --timeout 1h
```
# 2. 只删实例，不要删 blog-svc
```
unikraft instance stop blog
```
```
unikraft instance rm blog
```

# 3. 仍挂到同一个 blog-svc → 域名与 AI_ALLOWED_ORIGINS 都不变
```
unikraft run --metro sin \
  --name blog \
  -m 4G \
  --service blog-svc \
  --scale-to-zero policy=idle,cooldown-time=1000,stateful=true \
  --image fairsxto/blog:latest \
  --timeout 30m
```

# 首次访问
```
unikraft services get blog-svc
```
或
```
unikraft instance get blog
```
打开返回的固定 `https://….sin.unikraft.app`。

# 常用命令
# 查看运行中的实例（Instances）
```
unikraft instance list
```
# 查看实例详情
```
unikraft instance get blog
```
# 查看 Service（固定域名）
```
unikraft services list
unikraft services get blog-svc
```
# 查看实例日志
```
unikraft instance logs blog
```
# 停止实例（Instances）
```
unikraft instance stop blog
```
# 启动已停止的实例（Instances）
```
unikraft instance start blog
```
# 重启实例（先停再起，实例需仍存在）
```
unikraft instance restart blog
```
# 删除实例（Instances）——不会删掉 blog-svc
```
unikraft instance rm blog
```
# 删除 Service（会丢掉固定域名，一般不用；删了就要重配 Cloudflare）
```
unikraft services delete blog-svc
```
# 列出所有卷（Volumes）
```
unikraft volume list
```
# 删除指定存储卷（Volumes）
```
unikraft volume rm blog-data
```
# 列出所有镜像（Images）
```
unikraft image list
```
# 删除指定镜像（Images）
```
unikraft image rm fairsxto/blog
```
