# 确认包名称是否匹配

## 工作流中的配置

在 `.github/workflows/build-and-push.yml` 中：

```yaml
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository_owner }}/linknlink-remote-frpc
```

## 计算实际的包名称

1. **获取仓库所有者**：
   - 您的 GitHub 用户名是：`Acmen0102` 或 `acmen0102`
   - GitHub 会自动转换为小写，所以是：`acmen0102`

2. **计算 IMAGE_NAME**：
   ```
   IMAGE_NAME = github.repository_owner / linknlink-remote-frpc
   IMAGE_NAME = acmen0102 / linknlink-remote-frpc
   ```

3. **完整的镜像地址**：
   ```
   ghcr.io/acmen0102/linknlink-remote-frpc
   ```

## 如何确认包名称

### 方法 1：查看 GitHub Packages 页面

1. 访问您的仓库：`https://github.com/Acmen0102/linknlink-remote`
2. 点击右侧的 **Packages**（如果有）
3. 或者直接访问：`https://github.com/Acmen0102/linknlink-remote/pkgs`
4. 查看包名称，应该是：`linknlink-remote-frpc`
5. 完整路径应该是：`acmen0102/linknlink-remote-frpc`

### 方法 2：查看镜像标签

访问以下 URL 查看包信息：
```
https://github.com/Acmen0102/linknlink-remote/pkgs/container/linknlink-remote-frpc
```

如果这个页面可以访问，说明包名称是正确的。

### 方法 3：使用 Docker 命令验证

在本地尝试拉取镜像：

```bash
# 测试包是否存在
docker pull ghcr.io/acmen0102/linknlink-remote-frpc:amd64

# 或者使用 manifest inspect（不需要登录）
docker manifest inspect ghcr.io/acmen0102/linknlink-remote-frpc:amd64
```

如果命令成功，说明包名称正确。

### 方法 4：查看工作流运行日志

1. 访问 GitHub Actions：`https://github.com/Acmen0102/linknlink-remote/actions`
2. 查看最新的工作流运行
3. 在日志中查找类似以下内容：
   ```
   IMAGE_NAME: acmen0102/linknlink-remote-frpc
   ```
4. 或者查看构建的镜像标签，应该是：
   ```
   ghcr.io/acmen0102/linknlink-remote-frpc:amd64
   ghcr.io/acmen0102/linknlink-remote-frpc:aarch64
   ...
   ```

## 验证清单

- [ ] 包名称：`linknlink-remote-frpc`
- [ ] 完整路径：`acmen0102/linknlink-remote-frpc`
- [ ] 镜像地址：`ghcr.io/acmen0102/linknlink-remote-frpc`
- [ ] 可以在 GitHub Packages 页面看到
- [ ] 可以拉取镜像或查看 manifest

## 如果包名称不匹配

### 情况 1：包名称不同

如果实际的包名称是 `linknlink-frpc` 而不是 `linknlink-remote-frpc`：

1. **选项 A**：修改工作流中的 IMAGE_NAME
   ```yaml
   IMAGE_NAME: ${{ github.repository_owner }}/linknlink-frpc
   ```

2. **选项 B**：重新构建，使用正确的名称

### 情况 2：用户名大小写问题

GitHub 会自动将用户名转换为小写，所以：
- 如果您的用户名是 `Acmen0102`
- 包名称会是 `acmen0102/linknlink-remote-frpc`（小写）

这是正常的，GitHub 会自动处理。

## 当前配置总结

**工作流配置：**
```yaml
REGISTRY: ghcr.io
IMAGE_NAME: acmen0102/linknlink-remote-frpc
```

**完整镜像地址：**
```
ghcr.io/acmen0102/linknlink-remote-frpc:{arch}
```

**config.json 中的配置应该是：**
```json
{
  "image": "ghcr.io/acmen0102/linknlink-remote-frpc:{arch}"
}
```

## 快速验证命令

```bash
# 检查包是否存在
curl -I https://ghcr.io/v2/acmen0102/linknlink-remote-frpc/manifests/amd64

# 查看所有标签（需要认证）
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://ghcr.io/v2/acmen0102/linknlink-remote-frpc/tags/list
```
