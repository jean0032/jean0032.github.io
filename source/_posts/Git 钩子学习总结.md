---
title: Git 钩子学习总结
date: 2025-04-20 23:37:37
tags: [git]
categories: Git
---
### 一、Git 钩子概述

定义：Git 钩子（Hooks）是 Git 在特定事件（如提交、推送、合并等）发生时自动触发的脚本，用于自定义和自动化开发流程。

核心价值：

- 自动化流程：自动执行代码检查、测试、部署等操作。
- 规范约束：强制遵守代码风格、提交消息格式等规则。
- 效率提升：减少手动操作，降低人为错误。

存储位置：

- 项目本地钩子：`.git/hooks/` 目录下（默认模板，不会随仓库同步）。
- 服务端钩子：服务端仓库的 `.git/hooks/`（如 `post-receive` 用于部署）。

* * *

### 二、钩子类型与触发时机

#### 1\. 客户端钩子（本地触发）

|     |     |     |
| --- | --- | --- |
| 钩子名称 | 触发时机 | 常见用途 |
| `pre-commit` | 提交前（输入提交消息前） | 代码风格检查（ESLint）、禁止提交大文件 |
| `prepare-commit-msg` | 启动提交消息编辑器前 | 自动生成提交消息模板 |
| `commit-msg` | 提交消息输入完成后 | 校验提交消息格式（如符合正则规则） |
| `post-commit` | 提交完成后 | 通知、触发构建 |
| `pre-push` | 推送前 | 运行完整测试，阻止推送失败代码 |
| `pre-rebase` | 变基前 | 禁止对已推送的提交变基 |

&nbsp;

#### 2\. 服务端钩子（远程仓库触发）

|     |     |     |
| --- | --- | --- |
| 钩子名称 | 触发时机 | 常见用途 |
| `pre-receive` | 推送请求到达时 | 检查提交是否符合团队规则 |
| `update` | 每个分支推送前 | 更细粒度控制分支权限 |
| `post-receive` | 推送完成后 | 触发部署、通知CI/CD |

&nbsp;

* * *

### 三、典型使用场景

#### 1\. 代码质量保障

- `pre-commit` 示例： 禁止提交调试代码或大文件：

```
#!/bin/sh
# 检查是否包含 console.log
if git diff --cached | grep 'console.log'; then
  echo "Error: 提交内容包含调试代码！"
  exit 1
fi
```

#### 2\. 提交消息规范

- `commit-msg` 示例： 要求消息格式为 `[类型] 描述`（如 `[feat] 新增登录功能`）：

```
#!/bin/sh
message=$(cat $1)
if ! echo "$message" | grep -qE '^$$(feat|fix|docs)$$ .+'; then
  echo "提交消息格式错误！需为 [类型] 描述"
  exit 1
fi
```

#### 3\. 自动部署（经典场景）

- `post-receive` 示例： 推送后自动部署到服务器：

```
#!/bin/sh
TARGET_DIR="/var/www"
GIT_DIR=$(pwd)
# 强制检出到目标目录
git --work-tree=$TARGET_DIR --git-dir=$GIT_DIR checkout -f
# 重启服务
systemctl reload nginx
```

* * *

### 四、配置流程

#### 1\. 创建钩子脚本

```
# 进入项目.git/hooks目录
cd .git/hooks
# 创建pre-commit钩子（使用Shell/Python等任意语言）
vim pre-commit
# 添加执行权限
chmod +x pre-commit
```

#### 2\. 调试钩子

- 手动触发测试：

```
.git/hooks/pre-commit  # 直接运行脚本
```

- 日志调试：

```
# 在脚本开头添加日志记录
exec > /tmp/hook.log 2>&1
echo "$(date) 钩子启动"
```

#### 3\. 共享钩子（团队协作）

- 方案1：将钩子脚本存储在项目根目录（如 `scripts/hooks/`），通过符号链接关联：

```
ln -s ../../scripts/hooks/pre-commit .git/hooks/pre-commit
```

- 方案2：使用工具管理（如 [Husky](https://typicode.github.io/husky/)）：

```
# 安装Husky
npm install husky --save-dev
# 配置package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  }
}
```

* * *

### 五、高级技巧与注意事项

#### 1\. 钩子执行控制

- 跳过钩子：通过 `--no-verify` 参数临时绕过检查：

```
git commit -m "紧急修复" --no-verify
```

#### 2\. 错误处理

- 非零退出码：脚本返回非零值将中断 Git 操作。
- 用户友好提示：错误信息需明确指导如何修复。

#### 3\. 性能优化

- 轻量化操作：避免在 `pre-commit` 中运行耗时任务（如完整测试）。
- 增量检查：使用 `git diff --cached` 仅检查暂存区文件。

#### 4\. 安全注意事项

- 禁止直接修改代码：钩子应仅用于检查，避免自动修复导致意外结果。
- 服务端钩子权限：确保运行钩子的用户（如 `git`）有目标目录写入权限。

* * *

### 六、常见问题与解决

#### 1\. 钩子未生效

- 检查项：
    
- 文件路径和名称是否正确（如 `pre-commit` 不是 `pre-commit.sh`）
    
- 脚本是否有可执行权限（`chmod +x`）
    
- 脚本的退出码是否正确（成功返回0，失败返回非0）
    

#### 2\. 路径错误（如部署失败）

- 绝对路径：在钩子脚本中尽量使用绝对路径。
- 环境变量：检查 `$PATH` 是否包含所需命令（如 `npm`）。

#### 3\. 裸仓库配置

- 服务端仓库必须为裸仓库：

```
git clone --bare repo.git  # 创建裸仓库
git rev-parse --is-bare-repository  # 验证返回true
```

* * *

### 七、总结

Git 钩子是自动化开发流程的利器，但需注意：

- 明确目的：避免过度设计，优先解决核心痛点。
- 谨慎使用：服务端钩子可能影响团队协作，需充分测试。
- 文档化：团队共享钩子时需提供配置说明。
