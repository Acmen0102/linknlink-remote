# 修复 Manifest List 构建问题

## 问题说明

当前的 `:amd64` 等架构标签是 manifest list（多架构索引），而不是单个架构镜像。Home Assistant 需要的是单个架构镜像。

## 解决方案

### 步骤 1：删除所有旧的架构标签（必须！）

**在重新构建之前，必须先删除旧的 manifest list 标签。**

```bash
# 设置 Token
export GITHUB_TOKEN=your_token_here

# 删除所有架构标签
./delete_tags.sh delete-arch-tags y
```

这会删除：
- `:amd64`
- `:aarch64`
- `:armhf`
- `:armv7`
- `:i386`
- 以及相关的版本标签（`:latest-amd64`, `:1.0.0-amd64` 等）

### 步骤 2：提交工作流修复

工作流已经修改为：
1. 先推送单个架构标签（`:amd64`）
2. 然后分别推送其他标签（`:latest-amd64`, `:1.0.0-amd64`）
3. 这样可以确保每个标签都是单个镜像，不是 manifest list

```bash
git add .github/workflows/build-and-push.yml
git commit -m "fix: 修改构建配置，确保架构标签是单个镜像而不是 manifest list"
git push origin main
```

### 步骤 3：在 GitHub Actions 中重新构建

1. 访问 GitHub Actions 页面
2. 选择 "Build and Push Docker Images" 工作流
3. 点击 "Run workflow" 手动触发

### 步骤 4：验证构建结果

构建完成后，验证镜像：

```bash
docker manifest inspect ghcr.io/acmen0102/linknlink-remote-frpc:amd64
```

应该看到：
- `mediaType: "application/vnd.oci.image.manifest.v1+json"`（单个镜像）
- **不是** `"application/vnd.oci.image.index.v1+json"`（manifest list）
- 有 `architecture: "amd64"` 和 `os: "linux"` 字段
- **没有** `manifests` 数组

## 为什么需要先删除？

如果旧的 manifest list 还在：
1. Docker 可能会将新的单个镜像与旧的 manifest list 合并
2. 或者新的推送可能会失败
3. 或者会创建包含 `unknown` 平台的混合 manifest list

## 工作流修改说明

### 修改前（会创建 manifest list）：
```yaml
tags: |
  ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ matrix.arch.name }}
  ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.tags.outputs.tag }}-${{ matrix.arch.name }}
  ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.version.outputs.version }}-${{ matrix.arch.name }}
```
同时推送多个标签可能导致 Docker 创建 manifest list。

### 修改后（单个镜像）：
```yaml
# 步骤 1：先推送架构标签（单个镜像）
tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ matrix.arch.name }}

# 步骤 2：然后分别推送其他标签
docker pull $BASE_TAG
docker tag $BASE_TAG $NEW_TAG
docker push $NEW_TAG
```
分别推送确保每个标签都是单个镜像的引用。

## 验证清单

- [ ] 已删除所有旧的架构标签
- [ ] 已提交工作流修复
- [ ] 已手动触发 GitHub Actions 构建
- [ ] 构建成功完成
- [ ] 验证 `:amd64` 标签是单个镜像（不是 manifest list）
- [ ] Home Assistant 可以正常安装 addon

## 如果还有问题

1. **仍然看到 manifest list**：
   - 确认已删除所有架构标签
   - 检查是否有其他工作流或脚本在创建 manifest list

2. **构建失败**：
   - 查看 GitHub Actions 日志
   - 检查验证步骤的错误信息

3. **Home Assistant 仍然无法安装**：
   - 确认镜像已公开（public）
   - 检查 `config.json` 中的 `image` 字段格式

