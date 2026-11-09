# docker-mirror

把国内拉不到的 Docker Hub 镜像（如 MinIO）搬运到你自己的 GHCR，然后本地从 GHCR 直连拉取。

## 为什么需要

国内免费 Docker 镜像源（DaoCloud、1ms.run 等）对 **MinIO 这类商业软件镜像做了合规拦截**（docker 客户端请求一律 403），导致 `minio/minio` 拉不下来。但 GitHub 的 CI 环境在海外、可以直连 Docker Hub，而本机网络可以直连 ghcr.io。于是用 GitHub Actions 做一次「中转」。

## 一次性搭建步骤

### 1. 创建仓库并推送

```bash
# 在 GitHub 网页新建一个空仓库，名字叫 docker-mirror（public 即可）
# 然后本地执行：
cd E:/AGI/WorkBuddy/2026-11-09-15-16-36/docker-mirror
git init
git add .
git commit -m "chore: add minio mirror workflow"
git branch -M main
git remote add origin https://github.com/<你的用户名>/docker-mirror.git
git push -u origin main
```

### 2. 触发 workflow

打开仓库的 **Actions** 页 → 左侧选 **"Mirror MinIO to GHCR"** → 右侧 **Run workflow** → 绿色按钮确认。等待 1-2 分钟跑完（绿色对勾）。

### 3. 把镜像包设为公开（关键一步，否则本地拉不到）

打开仓库 **Settings → Packages**（或直接访问 `https://github.com/<你的用户名>?tab=packages`）→ 点进 `minio` 包 → **Package settings → Danger Zone → Change visibility → Public**。

> 为什么：ghcr.io 的镜像包默认是 private 的，本地不登录就拉不到。改成 Public 后就能匿名拉取。

### 4. 本地拉取验证

```bash
docker pull ghcr.io/<你的用户名>/minio:latest
```

成功后，在 `secure-doc-qa/docker-compose.yml` 里把 minio 的 `image:` 从 `minio/minio:latest` 改成 `ghcr.io/<你的用户名>/minio:latest`，再 `docker compose up -d` 即可。

## 重复使用（搬其他被墙镜像）

改 `.github/workflows/mirror-minio.yml` 里的镜像名即可，或在同一 workflow 里加多行 `docker pull` / `docker tag` / `docker push`。
