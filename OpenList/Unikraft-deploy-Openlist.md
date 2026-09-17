# 在Unikraft部署Openlist
 - 网址：https://unikraft.com
 - 文档：https://unikraft.com/docs/introduction

# Unikraft特点

 - 🧠 最多 2 个 CPU 核心（2 vCPU）
 - 💾 最多 4GB 内存
 - 🖥️ 最多同时运行 2 个实例
 - 💽 有 8GB 存储空间
 - 📦 可以存 10GB 的镜像
 - ⏳ 可以闲置时自动“关机”，有请求时再快速启动


# 第一步：安装官方cli
```
curl --proto '=https' --tlsv1.2 -fsSL https://unikraft.com/cli/install.sh | sh
```
# 第二步：登陆: 打开url进行授权
```
unikraft login --no-browser
```
# 第三步：切换到项目目录
```
cd openlist
```
# 第四步：打包镜像:
```
unikraft build . --output liuzxl/openlist
```

> **重新推送镜像后，必须先删再建实例**，否则会报 `An instance with the name 'openlist' already exists`，且旧实例不会自动用上新镜像。见下方「更新镜像后重建实例」。

# 第五步：创建数据目录：
```
unikraft volume create --metro sin --name openlist-data --size 1G
```
# 第六步：首次运行：
```
unikraft run --metro sin \
  --name openlist \
  -m 4G \
  -p 443:5244/tls+http \
  --scale-to-zero policy=idle,cooldown-time=1000,stateful=true \
  --volume openlist-data:/app/data \
  --image liuzxl/openlist:latest
```

# 更新镜像后重建实例
```
unikraft build . --output liuzxl/openlist
```
# 停止实例
```
unikraft instance stop openlist
```
# 删除实例
```
unikraft instance rm openlist
```
# 运行：
```
unikraft run --metro sin \
  --name openlist \
  -m 4G \
  -p 443:5244/tls+http \
  --scale-to-zero policy=idle,cooldown-time=1000,stateful=true \
  --volume openlist-data:/app/data \
  --image liuzxl/openlist:latest
```

# 常用命令
# 查看运行中的实例（Instances）
```
unikraft instance list
```
# 查看实例详情（含域名）
```
unikraft instance get openlist
```
# 查看实例日志
```
unikraft instance logs openlist
```
# 停止实例（Instances）
```
unikraft instance stop openlist
```
# 启动已停止的实例（Instances）
```
unikraft instance start openlist
```
# 重启实例（先停再起，实例需仍存在）
```
unikraft instance restart openlist
```
# 删除实例（Instances）
```
unikraft instance rm openlist
```
# 列出所有卷（Volumes）
```
unikraft volume list
```
# 删除指定存储卷（Volumes）
```
unikraft volume rm openlist-data
```
# 列出所有镜像（Images）
```
unikraft image list
```
# 删除指定镜像（Images）
```
unikraft image rm liuzxl/openlist
```
