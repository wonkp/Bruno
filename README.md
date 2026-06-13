# Bruno API 集合

本项目是使用 [Bruno](https://www.usebruno.com/) 维护的 API 测试集合，用于对 mathplore.com 平台的后端接口进行调试与验证。

## 环境要求

- [Bruno](https://www.usebruno.com/downloads) 桌面客户端，或
- [Bruno CLI](https://docs.usebruno.com/bru-cli/overview) (`npm install -g @usebruno/cli`)

## 项目结构

```
.
├── opencollection.yml          # 集合根配置（认证脚本、全局配置）
├── environments/
│   ├── 本地.yml                  # 本地开发环境变量
│   └── 测试.yml                  # 测试环境变量
├── 查询邀请人信息.yml             # 接口请求定义
├── 触发分析流程.yml               # 接口请求定义
└── ...                          # 其他 .yml 请求文件
```

## 快速开始

### 1. 使用 Bruno 桌面客户端

1. 打开 Bruno，选择「打开集合」（Open Collection），定位到本仓库根目录。
2. 在右上角切换目标环境（本地 / 测试）。
3. 任意选择一个请求，点击「发送」即可。集合会自动执行登录脚本获取 Token。

### 2. 使用 Bruno CLI

```bash
# 运行整个集合（默认使用测试环境）
bru run --env 测试

# 运行单个请求
bru run "查询邀请人信息" --env 测试
```

## 认证机制

集合采用自动登录机制：

- 每个请求发送前，会触发 `opencollection.yml` 中的 `before-request` 脚本。
- 脚本自动向 `/system-base/web/login` 接口发送登录请求（使用加密凭据）。
- 登录成功后，将返回的 `tokenValue` 写入环境变量 `token`。
- 所有请求通过 `auth: inherit` 自动携带 `authorization` 请求头。

> **注意**：如需修改登录账号或密码，请编辑 `opencollection.yml` 中的脚本内容。

## 环境变量说明

| 变量名 | 说明 | 本地环境示例 | 测试环境示例 |
|---|---|---|---|
| `teaching-url` | 教学服务基础地址 | `http://localhost:8083/teaching-web` | `https://test-api.mathplore.com/teaching-web` |
| `sale-url` | 销售服务基础地址 | `http://localhost:8084/sale-web` | `https://test-api.mathplore.com/sale-web` |
| `system-url` | 系统服务基础地址 | `http://localhost:8082/system-base` | `https://test-api.mathplore.com/system-base` |
| `token` | JWT Token | 由脚本自动填充 | 由脚本自动填充 |

## 添加新接口

在仓库根目录下新建 `.yml` 文件，参考已有文件格式：

```yaml
info:
  name: 接口名称
  type: http
  seq: 1

http:
  method: GET
  url: "{{sale-url}}/your/api/path"
  params:
    - name: param
      value: "value"
      type: query
  auth: inherit

settings:
  encodeUrl: true
  timeout: 0
```

完成后在 Bruno 客户端中右键「重新加载集合」即可看到新请求。
