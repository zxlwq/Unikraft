# 在Unikraft部署各类项目

 - 网址：https://unikraft.com
 - 文档：https://unikraft.com/docs/introduction

<p align="center">
  <img src="./unikraft.webp" alt="notes" />
</p>

---

# Unikraft特点

 - 🧠 最多 2 个 CPU 核心（2 vCPU）
 - 💾 最多 4GB 内存
 - 🖥️ 最多同时运行 2 个实例
 - 💽 有 8GB 存储空间
 - 📦 可以存 10GB 的镜像
 - ⏳ 可以闲置时自动“关机”，有请求时再快速启动

---

# 第一步：安装官方cli
```
curl --proto '=https' --tlsv1.2 -fsSL https://unikraft.com/cli/install.sh | sh
```
# 第二步：登陆: 打开url进行授权
```
unikraft login --no-browser
```
# 第三步：切换到项目目录 (可选)
```
cd alist
```
# 第四步：打包镜像 (可选)
```
unikraft build . --output zxlwqa/alist
```
> 使用Unikraft Images已打包好的镜像可跳过

# 第五步：创建数据卷：
```
unikraft volume create --metro sin --name alist-data --size 1G
```
# 第六步：运行：
```
unikraft run --metro sin \
  --name alist \
  -m 4G \
  -p 443:5244/tls+http \
  --scale-to-zero policy=idle,cooldown-time=1000,stateful=true \
  --volume alist-data:/app/data \
  --image zxlwqa/alist:latest
```

---


# 常用命令
## 查看运行中的实例（Instances）
```
unikraft instance list
```
## 查看实例详情（含域名）
```
unikraft instance get alist
```
## 查看实例日志
```
unikraft instance logs alist
```
## 停止实例（Instances）
```
unikraft instance stop alist
```
## 启动已停止的实例（Instances）
```
unikraft instance start alist
```
## 重启实例（先停再起，实例需仍存在）
```
unikraft instance restart alist
```
## 删除实例（Instances）
```
unikraft instance rm alist
```
## 列出所有卷（Volumes）
```
unikraft volume list
```
## 删除指定存储卷（Volumes）
```
unikraft volume rm alist-data
```
## 列出所有镜像（Images）
```
unikraft image list
```
## 删除指定镜像（Images）
```
unikraft image rm zxlwqa/alist
```
