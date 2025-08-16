# 自动备份程序

这是一个基于GitHub Actions的自动备份程序，用于定期备份网站数据。

## 功能特性

- 🕒 **定时备份**: 每天自动执行备份任务
- 🔐 **安全认证**: 支持用户名密码认证
- 📁 **自动管理**: 自动保存和管理备份文件
- 🚀 **GitHub Actions**: 完全基于GitHub Actions运行
- 🧹 **自动清理**: 自动清理旧备份文件（默认保留最近100个，可自定义）
- 🔍 **智能检测**: 自动检测数据变化，只在有变更时保存备份

## 快速开始

### 1. 配置GitHub仓库机密

在你的GitHub仓库中，进入 `Settings` -> `Secrets and variables` -> `Actions`，添加以下机密：

**必需配置**:
| 机密名称 | 说明 | 示例值 |
|---------|------|--------|
| `BACKUP_URL` | 网站完整URL（包含协议和端口） | `` |
| `BACKUP_USERNAME` | 登录用户名 | `your_username` |
| `BACKUP_PASSWORD` | 登录密码 | `your_password` |

**可选配置** (不设置将使用默认值):
| 机密名称 | 说明 | 示例值 | 默认值 |
|---------|------|--------|--------|
| `MAX_BACKUPS` | 最大保留备份文件数量 | `100` | `100` |
| `ENABLE_CHANGE_DETECTION` | 启用变更检测 | `true` | `true` |

### 2. 启用GitHub Actions

确保你的仓库已启用GitHub Actions功能。

**注意**: 程序会自动将你提供的URL拼接成完整的API路径：
- 输入: ``
- 自动拼接为: `/api/manage/sysConfig/backup?action=backup`
- 支持 HTTP 和 HTTPS 协议，会保持你指定的协议类型

### 3. 执行备份

备份程序会：
- **自动执行**: 每天北京时间02:00自动运行
- **手动触发**: 你也可以在Actions页面手动触发备份

## 文件结构

```
cloudflare-imgbed-auto-backup/
├── .github/
│   └── workflows/
│       └── backup.yml          # GitHub Actions工作流配置
├── backups/                    # 备份文件存储目录
│   ├── backup_20240816_100000.json
│   ├── backup_20240817_100000.json
│   └── latest_backup.json      # 最新备份文件
├── backup_script.py            # 主备份脚本
├── requirements.txt            # Python依赖
└── README.md                   # 说明文档
```

## 备份文件说明

- **带时间戳的备份文件**: `backup_YYYYMMDD_HHMMSS.json`
- **最新备份文件**: `latest_backup.json` (始终指向最新的备份)
- **自动清理**: 程序会自动保留最近的指定数量个备份文件（默认100个）

## 🔍 智能变更检测

程序默认启用智能变更检测功能，只有在数据发生变化时才会保存新的备份文件：

### 检测原理
- 使用MD5哈希算法计算JSON数据的指纹
- 将新下载的数据与最新备份文件进行对比
- 只有当哈希值不同时才保存新的备份文件

### 优势
- **节省存储空间**: 避免保存重复的备份文件
- **减少提交噪音**: GitHub仓库不会产生无意义的提交
- **提高效率**: 跳过无变化的备份操作

### 控制选项
- **启用**: `ENABLE_CHANGE_DETECTION=true` （默认）
- **禁用**: `ENABLE_CHANGE_DETECTION=false` （强制每次都备份）

### 特殊情况处理
- **首次备份**: 没有历史数据时，总是保存备份
- **计算错误**: 无法计算哈希时，为安全起见会强制保存备份
- **文件缺失**: 找不到最新备份文件时，会保存新的备份

## 🌐 HTTP/HTTPS 协议支持

程序支持 HTTP 和 HTTPS 两种协议，并能智能处理不同的网络环境：

### HTTP 协议特殊情况

#### 1. **混合内容问题**
- 在 GitHub Actions 环境中，HTTP 连接可能会受到安全策略限制
- 如果遇到连接问题，建议优先使用 HTTPS

#### 2. **端口配置**
- HTTP 默认端口：80
- HTTPS 默认端口：443
- 自定义端口：如 `http://example.com:8080/`
- 程序会保持你指定的端口号

#### 3. **代理和防火墙**
```bash
# 如果你的服务器在防火墙后，确保端口已开放
sudo ufw allow 40968  # Ubuntu/Debian
sudo firewall-cmd --permanent --add-port=40968/tcp  # CentOS/RHEL
```

#### 4. **SSL/TLS 证书问题**
- HTTP 连接不涉及证书验证
- 如果你的 HTTPS 使用自签名证书，可能需要特殊处理

#### 5. **连接超时设置**
- 程序默认超时时间：30秒
- 对于较慢的服务器，可能需要调整超时时间

### 协议选择建议

| 场景 | 推荐协议 | 说明 |
|------|----------|------|
| 生产环境 | HTTPS | 数据加密，更安全 |
| 测试环境 | HTTP | 配置简单，快速测试 |
| 内网环境 | HTTP | 无需证书，性能更好 |
| 公网环境 | HTTPS | 必须使用，防止数据泄露 |

### 常见问题解决

1. **"Connection refused" 错误**
   ```bash
   # 检查服务是否运行
   netstat -tulpn | grep 40968
   
   # 检查防火墙状态
   sudo ufw status
   ```

2. **"Certificate verify failed" 错误**
   - 这通常发生在 HTTPS 连接中
   - 确保证书有效或使用 HTTP 协议

3. **"Timeout" 错误**
   - 检查网络连接
   - 确认服务器响应时间
   - 考虑增加超时时间

## 本地运行

如果你想在本地运行备份脚本：

1. 安装依赖：
```bash
pip install -r requirements.txt
```

2. 设置环境变量：
```bash
export BACKUP_URL="http://"
export BACKUP_USERNAME="your_username"
export BACKUP_PASSWORD="your_password"
export MAX_BACKUPS="100"  # 可选，默认100
export ENABLE_CHANGE_DETECTION="true"  # 可选，默认true
```

3. 运行脚本：
```bash
python backup_script.py
```

## 故障排除

### 常见问题

1. **认证失败**
   - 检查用户名和密码是否正确
   - 确认GitHub机密设置正确

2. **网络连接问题**
   - 检查完整URL是否正确（如：http:///）
   - 确认服务器端口是否开放（如：40968）
   - 确认网站的完整API路径可访问
   - 确认网站没有IP限制
   - 如果使用HTTP，确保服务器支持HTTP连接
   - **HTTP特有问题**：某些网络环境可能阻止HTTP连接，建议先测试HTTPS

3. **协议相关问题**
   - **HTTP**: 检查是否被防火墙或代理阻止
   - **HTTPS**: 检查SSL证书是否有效
   - **端口**: 确认自定义端口（如40968）已正确开放
   - **超时**: 如果服务器响应慢，可能需要增加超时时间

3. **JSON解析错误**
   - 确认API返回的是有效的JSON格式
   - 检查API是否正常工作

4. **变更检测问题**
   - 如果怀疑变更检测有误，可以设置 `ENABLE_CHANGE_DETECTION=false` 强制备份
   - 检查 `latest_backup.json` 文件是否存在且格式正确

5. **配置错误**
   - 如果看到 `invalid literal for int()` 错误，检查 `MAX_BACKUPS` 是否设置了有效数字
   - 可选配置项如果不需要可以不设置，程序会使用默认值

### 本地测试方法

在配置GitHub Actions之前，建议先在本地测试连接：

```bash
# 测试HTTP连接
curl -v -u "username:password" "http:///api/manage/sysConfig/backup?action=backup"

# 测试HTTPS连接（如果支持）
curl -v -u "username:password" "https:///api/manage/sysConfig/backup?action=backup"

# 检查端口是否开放
telnet 11
```

### 查看日志

在GitHub Actions页面可以查看详细的执行日志，包括：
- 认证过程
- 下载进度
- 变更检测结果
- 文件保存状态
- 错误信息

## 自定义配置

你可以修改以下设置：

### 修改备份时间

编辑 `.github/workflows/backup.yml` 中的cron表达式：
```yaml
schedule:
  - cron: '0 18 * * *'  # 每天UTC时间18:00（北京时间02:00）
```

### 修改保留备份数量

编辑 `backup_script.py` 中的清理逻辑，或者在GitHub仓库机密中设置 `MAX_BACKUPS`：
```python
# 默认保留最近的100个备份文件
self.max_backups = int(os.getenv('MAX_BACKUPS', '100'))
```

### 禁用变更检测

如果你希望每次都强制保存备份（不管数据是否变化），可以设置：
```yaml
# 在GitHub仓库机密中设置
ENABLE_CHANGE_DETECTION: false
```

## 安全注意事项

1. **机密管理**: 永远不要在代码中硬编码用户名和密码
2. **访问权限**: 确保GitHub仓库的访问权限设置合适
3. **备份内容**: 注意备份文件可能包含敏感信息

## 贡献

欢迎提交Issues和Pull Requests来改进这个项目！

## 许可证

MIT License
