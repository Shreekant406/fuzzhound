# 🐕 FuzzHound

**FuzzHound** 是一款专为 API 安全测试设计的智能 Fuzz 工具，支持 Swagger/OpenAPI 文档自动解析，提供多种 Fuzz 模式和漏洞检测能力。像猎犬一样嗅探 API 中的安全漏洞！

---

## 💯 功能特性

### 核心功能
- **🔍 自动化 API 发现**：自动解析 Swagger 2.0 和 OpenAPI 3.0 文档，支持 JSON/YAML 格式
- **🎯 智能参数识别**：自动识别路径参数、查询参数、请求头、Cookie、请求体等
- **📦 枚举参数支持**：自动识别并测试 API 文档中定义的枚举参数，支持数量限制
- **🔗 $ref 引用解析**：完整支持 Swagger/OpenAPI 中的 `$ref` 引用解析

### Fuzz 测试能力
- **💥 用户名 Fuzz**：基于字典的用户名爆破测试，支持关键字匹配和全参数模式
- **🔐 密码 Fuzz**：基于字典的密码爆破测试，支持自定义字典和数量控制
- **🔢 数字型 Fuzz**：支持随机模式和范围遍历模式，可检测越权、IDOR 等漏洞
- **💉 SQL 注入检测**：
  - 三种检测模式：基础（10个payload）、智能（20个）、完整（155个）
  - 基线对比分析，自动计算风险评分
  - 支持 152 种 SQL 错误特征匹配
  - 智能去重和误报过滤
- **📁 文件上传支持**：自动识别文件上传接口，生成测试文件（支持图片、PDF、CSV等）

### 高级特性
- **🎨 灵活的参数控制**：
  - `--fuser 30`：关键字匹配 + 随机30个用户名
  - `--fuser all`：所有字符串参数 + 随机15个用户名
  - `--fuser all:100`：所有字符串参数 + 随机100个用户名
  - `--fuser all:all`：所有字符串参数 + 全部字典
- **🔥 一键全量测试**：`--fall` 参数启用所有 Fuzz 测试
- **🎯 状态码筛选**：
  - `--fuzz-status`：筛选显示的 Fuzz 结果状态码
  - `--fuzz-filter`：只对指定状态码的 API 进行 Fuzz 测试
- **⚡ 多线程并发**：支持自定义线程数，提高测试效率
- **🔒 认证支持**：支持 Bearer Token、API Key、Basic Auth、Cookie 等多种认证方式
- **🌐 代理支持**：支持 HTTP/HTTPS 代理，可配合 Burp Suite 使用
- **📊 详细报告**：生成 HTML 格式的测试报告，包含请求/响应详情
- **🎭 随机 User-Agent**：内置 User-Agent 池，模拟真实浏览器请求
- **🚫 黑名单过滤**：支持方法、路径、正则表达式黑名单，避免测试危险接口

---

## ✅ 安装

### 环境要求
- Python 3.7+
- pip

### 安装步骤

1. 克隆项目到本地：
   ```bash
   git clone https://github.com/RuoJi6/fuzzhound.git
   cd fuzzhound
   ```

2. 安装依赖：
   ```bash
   pip3 install -r requirements.txt
   ```

3. 验证安装：
   ```bash
   python3 fuzzhound.py --help
   ```

---

## 🥰 使用指南

### 基础用法

#### 1. 测试单个 API
```bash
# 基础测试
python3 fuzzhound.py -u http://example.com/api-docs

# 指定线程数和延迟
python3 fuzzhound.py -u http://example.com/api-docs -t 10 --delay 0.5
```

#### 2. 启用 Fuzz 测试
```bash
# 用户名 Fuzz（关键字匹配 + 随机15个）
python3 fuzzhound.py -u http://example.com/api-docs --fuser

# 密码 Fuzz（关键字匹配 + 随机30个）
python3 fuzzhound.py -u http://example.com/api-docs --fpass 30

# 用户名 Fuzz（所有字符串参数 + 全部字典）
python3 fuzzhound.py -u http://example.com/api-docs --fuser all:all

# SQL 注入检测（智能模式）
python3 fuzzhound.py -u http://example.com/api-docs --fpsql --sql-mode smart

# 一键启用所有 Fuzz
python3 fuzzhound.py -u http://example.com/api-docs --fall
```

#### 3. 高级用法
```bash
# 使用代理 + 认证 + 所有 Fuzz
python3 fuzzhound.py -u http://example.com/api-docs \
  --proxy http://127.0.0.1:8080 \
  --token "your-bearer-token" \
  --fall all \
  -t 15

# 只对返回 200 的 API 进行 Fuzz
python3 fuzzhound.py -u http://example.com/api-docs \
  --fall \
  --fuzz-filter 200

# 限制枚举参数测试数量
python3 fuzzhound.py -u http://example.com/api-docs \
  --enum-limit 10
```

### 配置文件

编辑 `config/config.yaml` 自定义默认配置：

```yaml
# 目标配置
target:
  base_url: "http://example.com"
  timeout: 10
  verify_ssl: false

# 请求配置
request:
  threads: 5
  delay: 1.5
  retry: 1

# Fuzz 配置
fuzz_username:
  enabled: false
  username_file: "config/usernames.txt"
  count: 15  # 默认随机挑选15个

fuzz_password:
  enabled: false
  password_file: "config/top100_password.txt"
  count: 15
```

---

## 📖 参数说明

### 基础参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-u, --url` | 目标 URL 或 API 文档路径 | `-u http://example.com/api-docs` |
| `-c, --config` | 配置文件路径 | `-c config/custom.yaml` |
| `-t, --threads` | 并发线程数 | `-t 10` |
| `--delay` | 请求间延迟（秒） | `--delay 0.5` |
| `--timeout` | 请求超时时间（秒） | `--timeout 30` |
| `--proxy` | HTTP/HTTPS 代理 | `--proxy http://127.0.0.1:8080` |
| `--token` | Bearer Token | `--token "your-token"` |
| `--debug` | 启用调试模式 | `--debug` |

### Fuzz 参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `--fall [MODE]` | 一键启用所有 Fuzz | `--fall` 或 `--fall all` |
| `--fuser [N\|all\|all:N\|all:all]` | 用户名 Fuzz | `--fuser 30` / `--fuser all:100` |
| `--fpass [N\|all\|all:N\|all:all]` | 密码 Fuzz | `--fpass all` / `--fpass all:all` |
| `--fnumber [N\|START-END\|all]` | 数字型 Fuzz | `--fnumber 20` / `--fnumber 1-1000` |
| `--fpsql [KEYWORDS]` | SQL 注入检测 | `--fpsql` 或 `--fpsql id,name` |
| `--sql-mode [MODE]` | SQL 检测模式 | `--sql-mode smart` |
| `--sql-payloads [N]` | 自定义 SQL payload 数量 | `--sql-payloads 50` |

### 筛选参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `--fuzz-status` | 筛选显示的 Fuzz 结果状态码 | `--fuzz-status 200,500` |
| `--fuzz-filter` | 只对指定状态码的 API 进行 Fuzz | `--fuzz-filter 200` |
| `--enum-limit` | 枚举参数测试数量限制 | `--enum-limit 10` |
| `--ignore-blacklist` | 忽略黑名单 | `--ignore-blacklist` |

---

## 🎯 使用场景

### 场景 1：快速安全扫描
```bash
# 对目标进行快速安全扫描，启用所有 Fuzz
python3 fuzzhound.py -u https://api.example.com/swagger.json --fall -t 15
```

### 场景 2：SQL 注入专项测试
```bash
# 使用完整模式进行 SQL 注入检测
python3 fuzzhound.py -u https://api.example.com/api-docs \
  --fpsql \
  --sql-mode full \
  --fuzz-filter 200
```

### 场景 3：用户名/密码爆破
```bash
# 使用自定义字典进行爆破
python3 fuzzhound.py -u https://api.example.com/api-docs \
  --fuser all:all \
  --fpass all:all \
  -t 20
```

### 场景 4：配合 Burp Suite 使用
```bash
# 通过 Burp 代理进行测试，方便查看详细请求
python3 fuzzhound.py -u https://api.example.com/api-docs \
  --proxy http://127.0.0.1:8080 \
  --fall \
  --debug
```

### 场景 5：越权测试
```bash
# 使用数字型 Fuzz 测试 IDOR 漏洞
python3 fuzzhound.py -u https://api.example.com/api-docs \
  --fnumber 1-10000 \
  --fuzz-filter 200
```

---

## 🔖 待办事项

- [x] **Swagger/OpenAPI 解析**：支持 Swagger 2.0 和 OpenAPI 3.0
- [x] **多种 Fuzz 模式**：用户名、密码、数字、SQL 注入
- [x] **智能参数匹配**：关键字匹配和全参数模式
- [x] **状态码筛选**：支持 Fuzz 前后的状态码筛选
- [x] **文件上传支持**：自动生成测试文件
- [x] **SQL 注入检测**：基线对比 + 错误匹配 + 风险评分
- [ ] **XSS 检测**：支持反射型和存储型 XSS 检测
- [ ] **命令注入检测**：支持系统命令注入检测
- [ ] **SSRF 检测**：支持服务端请求伪造检测
- [ ] **XXE 检测**：支持 XML 外部实体注入检测
- [ ] **JWT 测试**：支持 JWT 弱密钥、算法混淆等测试
- [ ] **GraphQL 支持**：支持 GraphQL API 测试
- [ ] **WebSocket 支持**：支持 WebSocket 接口测试
- [ ] **插件系统**：支持自定义 Fuzz 插件
- [ ] **数据持久化**：支持测试结果数据库存储
- [ ] **Web UI**：提供 Web 界面进行测试和查看结果

---

## 📊 输出示例

### 控制台输出
```
╔═══════════════════════════════════════════════════════════╗
║                                                           ║
║   🐕 FuzzHound - API 安全测试工具                         ║
║                                                           ║
║   支持 Swagger/OpenAPI 自动化测试和智能 Fuzz              ║
║                                                           ║
╚═══════════════════════════════════════════════════════════╝

🔍 自动检测到 API 文档路径: https://api.example.com/swagger.json
✓ 成功解析 25 个 API 接口

📍 阶段 1/2: 普通测试
[200]      1.23 KB    125ms GET     /api/users/{id} Get User Info
[401]      0.15 KB     45ms POST    /api/login User Login
[200]      2.45 KB    234ms GET     /api/products List Products

📍 阶段 2/2: Fuzz 测试
🔍 Fuzz前置筛选：只对状态码为 [200] 的API进行Fuzz测试
[💉SQL]    1.25 KB    156ms GET     /api/users/{id} [可能存在SQL注入]
  └─ Payload: id=1' OR '1'='1
  └─ 风险评分: 75/100
  └─ 检测到SQL错误: MySQL syntax error

✓ 测试完成！报告已保存到: output/report.html
```

### HTML 报告
生成的 HTML 报告包含：
- 测试统计信息
- 状态码分布
- 详细的请求/响应信息
- SQL 注入检测结果
- 风险评分和建议

---

## 🔧 自定义字典

### 用户名字典
编辑 `config/usernames.txt`：
```
admin
administrator
root
test
user
```

### 密码字典
编辑 `config/top100_password.txt`：
```
123456
password
admin
12345678
```

### SQL Payload
编辑 `config/sql_payloads_*.txt`：
- `sql_payloads_basic.txt`：基础 payload（10个）
- `sql_payloads_smart.txt`：智能 payload（20个）
- `sql_payloads_full.txt`：完整 payload（155个）

---

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

### 开发指南
1. Fork 本项目：https://github.com/RuoJi6/fuzzhound
2. 创建特性分支：`git checkout -b feature/your-feature`
3. 提交更改：`git commit -am 'Add some feature'`
4. 推送分支：`git push origin feature/your-feature`
5. 提交 Pull Request

---

## 📜 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

---

## 📧 联系方式

如有任何问题或建议，请通过以下方式联系：

- **GitHub Issues**: [提交 Issue](https://github.com/RuoJi6/fuzzhound/issues)

---

## ⭐ Star History

[![Star History Chart](https://api.star-history.com/svg?repos=RuoJi6/fuzzhound&type=Timeline)](https://star-history.com/#RuoJi6/fuzzhound&Timeline)

---

## 🙏 致谢

感谢所有为本项目做出贡献的开发者！

---

**⚠️ 免责声明**

本工具仅供安全研究和授权测试使用。使用本工具进行未经授权的测试是违法的。使用者需自行承担使用本工具的一切后果，作者不承担任何法律责任。

