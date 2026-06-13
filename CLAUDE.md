# CLAUDE.md

本文档为 Claude Code (claude.ai/code) 提供在本仓库中工作时的指导。

## 项目概览

这是一个 [Bruno](https://www.usebruno.com/) API 集合（collection），使用 YAML 格式定义了针对 mathplore.com 平台的接口请求。本项目没有构建系统、包管理器或测试运行器，该集合直接由 Bruno 桌面应用或 Bruno CLI 消费使用。

## 集合结构

- `opencollection.yml` — 集合根配置。定义了集合级别的认证方式，以及一个 `before-request` 脚本，用于自动登录并获取 JWT Token。
- `environments/*.yml` — 环境变量定义文件（包含 `teaching-url`、`sale-url`、`system-url`、`token`）。
- 根目录下的 `*.yml` — 单个 API 请求定义文件。

## 认证流程

集合采用 API Key 认证方式，请求头中携带 `authorization`，其值为变量 `{{token}}`。

`opencollection.yml` 中包含一个 `before-request` 脚本，会在每次发起请求前自动执行登录：
- 向 `https://test-api.mathplore.com/system-base/web/login` 发送 POST 请求
- 使用硬编码的加密用户名和密码
- 登录成功后，通过 `bru.setVar('token', ...)` 将 `res.data.data.tokenValue` 存入 `token` 变量

如需修改登录接口、凭据或 Token 提取路径，请直接更新 `opencollection.yml` 中的脚本。

## 环境配置

| 环境 | 文件 | 用途 |
|---|---|---|
| 本地 | `environments/本地.yml` | 本地开发环境（`localhost:8082/8083/8084`） |
| 测试 | `environments/测试.yml` | 测试环境（`test-api.mathplore.com`） |

各环境中定义的变量：
- `teaching-url` — teaching-web 服务的基础地址
- `sale-url` — sale-web 服务的基础地址
- `system-url` — system-base 服务的基础地址
- `token` — JWT Token，由 `before-request` 脚本自动填充

## 请求文件格式

每个 API 请求都是一个独立的 YAML 文件，结构如下：

```yaml
info:
  name: <请求名称>
  type: http
  seq: <序号>

http:
  method: GET|POST|...
  url: "{{变量}}/路径"
  params:
    - name: 参数名
      value: "参数值"
      type: query
  auth: inherit  # 继承集合级别的认证配置

settings:
  encodeUrl: true
  timeout: 0
```

请求通常使用 `auth: inherit`，以自动携带集合级别配置的 `authorization` 请求头。
