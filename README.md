# question_bank

题库管理系统 CLI —— 面向考试的题库管理命令行工具，支持知识图谱、语义搜索、智能练习和错题分析。

设计用于 **AI Agent 调用**，所有命令均支持 `--json` 原始 JSON 输出。

## 安装

```bash
npm install -g @d_moki/question_bank
```

## 快速开始

```bash
# 查看帮助
question_bank --help

# 健康检查
question_bank system health

# 注册 + 登录（token 自动缓存至 ~/.question_bank/config.json）
question_bank auth register --email a@example.com --username myuser --password mypass
question_bank auth login --username myuser --password mypass

# 查看当前用户信息
question_bank users me

# 查询数学知识点
question_bank kp list --subject 数学

# 搜索题目
question_bank search --query "二次函数"

# JSON 输出（供 Agent 解析）
question_bank --json questions list --subject 数学
```

## 配置

| 方式                           | 说明                  |
| ------------------------------ | --------------------- |
| `--base-url <url>`             | CLI 参数指定 API 地址 |
| `QUESTION_BANK_URL`            | 环境变量指定 API 地址 |
| `~/.question_bank/config.json` | 登录后自动缓存 token  |

默认 API 地址：`http://localhost:8000/api/v1`

## 命令参考

### 认证鉴权 `auth`

| 命令                                                                                     | 说明                 |
| ---------------------------------------------------------------------------------------- | -------------------- |
| `auth register --email --username --password [--display-name] [--grade] [--exam-target]` | 注册新账号           |
| `auth login -u <用户名> -p <密码>`                                                       | 登录，token 自动缓存 |
| `auth refresh --refresh-token <token>`                                                   | 刷新令牌             |
| `auth logout`                                                                            | 登出，清除本地 token |

### 用户管理 `users`

| 命令                                                                        | 说明                   |
| --------------------------------------------------------------------------- | ---------------------- |
| `users me`                                                                  | 获取当前用户信息       |
| `users update-me [--display-name] [--avatar-url] [--grade] [--exam-target]` | 更新个人信息           |
| `users list [--page] [--size] [--keyword]`                                  | 用户列表（管理员）     |
| `users get <user-id>`                                                       | 查看指定用户（管理员） |
| `users update <user-id> [--display-name] [--role] [--is-active]`            | 更新指定用户（管理员） |

### 知识点管理 `kp`

| 命令                                                                                    | 说明                 |
| --------------------------------------------------------------------------------------- | -------------------- |
| `kp list [--exam-type] [--subject] [--level] [--parent-id] [--is-leaf]`                 | 查询知识点列表       |
| `kp tree [--exam-type] [--subject] [--parent-id]`                                       | 获取知识点树形结构   |
| `kp get <kp-id>`                                                                        | 获取知识点详情       |
| `kp create --name --level [--code] [--parent-id] [--exam-type] [--subject] [--is-leaf]` | 创建知识点（管理员） |
| `kp update <kp-id> [--name] [--description] [--sort-order] [--is-leaf]`                 | 更新知识点（管理员） |
| `kp delete <kp-id>`                                                                     | 删除知识点（管理员） |
| `kp children <kp-id>`                                                                   | 获取子知识点         |
| `kp descendants <kp-id>`                                                                | 获取所有子孙知识点   |
| `kp ancestors <kp-id>`                                                                  | 获取祖先知识点       |

### 题目管理 `questions`

| 命令                                                                                                             | 说明                         |
| ---------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| `questions list [--page] [--size] [--subject] [--difficulty] [--question-type] [--exam-type] [--kp-id] [--year]` | 查询题目列表                 |
| `questions get <question-id>`                                                                                    | 获取题目详情（含答案、解析） |
| `questions create --question-type --difficulty --content --answer [--analysis] [--kp-ids] [--tag-ids]`           | 创建题目（管理员）           |
| `questions update <id> [--question-type] [--difficulty] [--content] [--answer]`                                  | 更新题目（管理员）           |
| `questions delete <question-id>`                                                                                 | 删除题目（管理员）           |

`--content` 和 `--answer` 接受 JSON 字符串（图片 base64 可直接传）。`--kp-ids` 和 `--tag-ids` 用逗号分隔。

### 搜索 `search`

```bash
question_bank search --query "关键词" \
  [--exam-type] [--subject] [--difficulty] [--question-type] \
  [--knowledge-point-id] [--year] [--page] [--size]
```

支持关键词 + 筛选条件的混合搜索，返回结果含分面聚合。

### 知识图谱 `graph`

| 命令                                             | 说明           |
| ------------------------------------------------ | -------------- |
| `graph prerequisites <kp-id> [--depth 1-5]`      | 前置依赖链     |
| `graph similar <question-id> [--limit]`          | 相似题目       |
| `graph weak-points <user-id> [--limit]`          | 用户薄弱知识点 |
| `graph learning-path <user-id> [--target-kp-id]` | 学习路径推荐   |
| `graph error-patterns <user-id>`                 | 错误模式分析   |

### 练习管理 `practice`

| 命令                                                                                                            | 说明                                                            |
| --------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `practice start [--session-type] [--focus-kp-id] [--question-ids] [--question-count]`                           | 开始练习（random / knowledge_focus / error_review / mock_exam） |
| `practice history`                                                                                              | 练习历史                                                        |
| `practice submit <session-id> --answers '<json-array>'`                                                         | 一次性提交答案                                                  |
| `practice answer --session-id --question-id --user-answer '<json>' [--time-spent-seconds] [--confidence-level]` | 逐题提交答案                                                    |

### 错题本 `error-book`

| 命令                                                            | 说明                           |
| --------------------------------------------------------------- | ------------------------------ |
| `error-book list [--page] [--size] [--mastered] [--error-type]` | 错题列表                       |
| `error-book stats`                                              | 错题统计                       |
| `error-book master <entry-id> --mastered <true/false>`          | 标记掌握状态                   |
| `error-book analyze`                                            | 综合分析（薄弱点、根因、建议） |

### 系统 `system`

| 命令            | 说明     |
| --------------- | -------- |
| `system health` | 健康检查 |

## Agent 调用示例

```bash
# 登录
question_bank --json auth login -u agent -p secret

# 获取所有数学知识点树（JSON 输出可直接解析）
question_bank --json kp tree --subject 数学

# 搜索题目
question_bank --json search --query "微积分" --difficulty "3,4,5" --size 10

# 获取题目详情（content 中的图片 base64 原样透传）
question_bank --json questions get <question-id>

# 分析用户薄弱点
question_bank --json graph weak-points <user-id> --limit 20

# 开始知识点专练
question_bank --json practice start --session-type knowledge_focus --focus-kp-id <kp-id> --question-count 5

# 提交单题答案
question_bank --json practice answer \
  --session-id <id> \
  --question-id <id> \
  --user-answer '{"selected": "B"}'

# 获取错题分析
question_bank --json error-book analyze
```

## 退出码

| 码  | 含义                            |
| --- | ------------------------------- |
| 0   | 成功                            |
| 1   | 通用错误（网络、IO、解析）      |
| 2   | 认证失败（未登录或 token 过期） |
| 3   | 404 未找到                      |
| 4   | 其他 API 错误                   |
