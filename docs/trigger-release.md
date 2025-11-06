# 触发 Release Workflow 的几种方法

## 方法 1: 使用 GitHub API (repository_dispatch)

### 前置要求

1. **生成 GitHub Personal Access Token (PAT)**
   - 访问：https://github.com/settings/tokens
   - 点击 "Generate new token" → "Generate new token (classic)"
   - 权限选择：
     - `repo` (完整仓库访问权限)
     - 或者 `workflow` (如果只用于触发 workflow)
   - 复制生成的 token

### 使用 curl 命令触发

```bash
curl -X POST \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  https://api.github.com/repos/acmen0102/linknlink-remote/dispatches \
  -d '{
    "event_type": "release",
    "client_payload": {
      "version": "1.0.1"
    }
  }'
```

### 使用 JavaScript/Node.js 触发

```javascript
const https = require('https');

const data = JSON.stringify({
  event_type: 'release',
  client_payload: {
    version: '1.0.1'
  }
});

const options = {
  hostname: 'api.github.com',
  path: '/repos/acmen0102/linknlink-remote/dispatches',
  method: 'POST',
  headers: {
    'Accept': 'application/vnd.github.v3+json',
    'Authorization': 'token YOUR_GITHUB_TOKEN',
    'Content-Type': 'application/json',
    'Content-Length': data.length
  }
};

const req = https.request(options, (res) => {
  console.log(`Status: ${res.statusCode}`);
  res.on('data', (d) => {
    process.stdout.write(d);
  });
});

req.on('error', (error) => {
  console.error(error);
});

req.write(data);
req.end();
```

### 使用 Python 触发

```python
import requests
import json

url = "https://api.github.com/repos/acmen0102/linknlink-remote/dispatches"
headers = {
    "Accept": "application/vnd.github.v3+json",
    "Authorization": "token YOUR_GITHUB_TOKEN",
    "Content-Type": "application/json"
}
data = {
    "event_type": "release",
    "client_payload": {
        "version": "1.0.1"
    }
}

response = requests.post(url, headers=headers, data=json.dumps(data))
print(f"Status: {response.status_code}")
print(response.text)
```

### 使用 GitHub CLI (gh) 触发

```bash
# 安装 GitHub CLI (如果还没有)
# macOS: brew install gh
# Linux: 查看 https://cli.github.com/manual/installation

# 登录
gh auth login

# 触发 workflow
gh api repos/acmen0102/linknlink-remote/dispatches \
  -X POST \
  -f event_type='release' \
  -f client_payload='{"version":"1.0.1"}'
```

## 方法 2: 在 GitHub Actions 页面手动触发 (workflow_dispatch)

1. 进入仓库的 **Actions** 页面
2. 在左侧选择 **Release** workflow
3. 点击右侧的 **Run workflow** 按钮
4. 输入版本号（例如：`1.0.1`）
5. 点击 **Run workflow** 执行

## 方法 3: 创建 GitHub Release 自动触发

1. 进入仓库的 **Releases** 页面
2. 点击 **Draft a new release** 或 **Create a new release**
3. 输入 Tag 名称（例如：`v1.0.1`）
4. 填写 Release 标题和描述
5. 点击 **Publish release**
6. Workflow 会自动运行，并从 tag 中提取版本号

## 验证触发是否成功

### 检查 Workflow 运行状态

1. 进入 **Actions** 页面
2. 查看 **Release** workflow 的运行记录
3. 点击运行记录查看详细日志

### API 响应说明

- **成功响应**: `204 No Content` (表示请求已接受)
- **错误响应**: 
  - `401 Unauthorized` - Token 无效或权限不足
  - `404 Not Found` - 仓库不存在或无权访问
  - `422 Unprocessable Entity` - 请求格式错误

## 安全建议

1. **保护 Token**:
   - 不要将 token 提交到代码仓库
   - 使用环境变量存储 token
   - 定期轮换 token

2. **使用环境变量示例**:
   ```bash
   # 在 ~/.bashrc 或 ~/.zshrc 中添加
   export GITHUB_TOKEN="your_token_here"
   
   # 使用
   curl -X POST \
     -H "Accept: application/vnd.github.v3+json" \
     -H "Authorization: token $GITHUB_TOKEN" \
     https://api.github.com/repos/acmen0102/linknlink-remote/dispatches \
     -d '{"event_type":"release","client_payload":{"version":"1.0.1"}}'
   ```

## 自动化脚本示例

### Bash 脚本

```bash
#!/bin/bash

# 配置
REPO="acmen0102/linknlink-remote"
TOKEN="${GITHUB_TOKEN}"
VERSION="${1:-1.0.1}"

# 检查 token
if [ -z "$TOKEN" ]; then
    echo "Error: GITHUB_TOKEN environment variable is not set"
    exit 1
fi

# 触发 workflow
echo "Triggering release workflow for version: $VERSION"
curl -X POST \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Authorization: token $TOKEN" \
  "https://api.github.com/repos/$REPO/dispatches" \
  -d "{
    \"event_type\": \"release\",
    \"client_payload\": {
      \"version\": \"$VERSION\"
    }
  }"

if [ $? -eq 0 ]; then
    echo "✓ Workflow triggered successfully"
else
    echo "✗ Failed to trigger workflow"
    exit 1
fi
```

使用方法：
```bash
chmod +x trigger-release.sh
export GITHUB_TOKEN="your_token"
./trigger-release.sh 1.0.1
```

## 常见问题

### Q: 为什么看不到 workflow 运行？

A: `repository_dispatch` 事件触发的 workflow 在 GitHub Actions 页面会显示，但可能需要等待几秒钟。如果使用 `workflow_dispatch` 触发，会立即显示。

### Q: 如何知道 workflow 是否成功触发？

A: 检查 API 响应状态码：
- `204` = 成功接受
- 其他状态码 = 查看错误信息

### Q: 可以传递更多数据吗？

A: 可以，在 `client_payload` 中可以传递任意 JSON 数据：
```json
{
  "event_type": "release",
  "client_payload": {
    "version": "1.0.1",
    "changelog": "修复了某个bug",
    "author": "username"
  }
}
```

在 workflow 中通过 `github.event.client_payload.changelog` 访问。

