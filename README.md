# scoop_useage

Scoop 使用示例

## 项目结构

```text
scoop_useage/
├── .vscode/
│   └── settings.json   # VS Code 配置（已被 .gitignore 忽略）
├── apps/                       # 各软件安装示例
│   ├── go/
│   │   ├── go.md               # Go 安装（Scoop，完整示例）
│   │   └── g.md                # g 安装（Go 多版本管理工具 https://github.com/voidint/g）
│   ├── php.md                  # 待补充
│   ├── nginx.md                # 待补充
│   ├── mysql.md                # 待补充
│   ├── mariadb.md              # 待补充
│   ├── java.md                 # 待补充
│   └── npm.md                  # NPM / Node.js，待补充
├── .gitignore
├── install.ps1         # Scoop 官方安装脚本
└── README.md
```

## 安装指令

```powershell
# ScoopDir 为 Scoop 安装目录
# ScoopGlobalDir 为 Scoop 全局目录
# NoProxy 为是否禁用代理
.\install.ps1 -ScoopDir 'D:\Program Files\Scoop' -ScoopGlobalDir 'D:\GlobalScoopApps' -NoProxy
```

## 配置代理

### 安装时指定代理

```powershell
# 使用指定代理安装（无需认证）
.\install.ps1 -ScoopDir 'D:\Program Files\Scoop' -ScoopGlobalDir 'D:\GlobalScoopApps' `
  -Proxy 'http://127.0.0.1:7890'

# 使用当前 Windows 用户的凭据进行代理认证
.\install.ps1 -Proxy 'http://proxy.example.com:8080' -ProxyUseDefaultCredentials

# 手动输入代理账号密码
.\install.ps1 -Proxy 'http://proxy.example.com:8080' -ProxyCredential (Get-Credential)

# 绕过系统代理
.\install.ps1 -NoProxy
```

### 安装后配置 Scoop 代理

```powershell
# 设置代理
scoop config proxy 127.0.0.1:7890

# 代理需要认证时使用 user:password@host:port 格式
scoop config proxy username:password@proxy.example.com:8080

# 查看当前代理配置
scoop config proxy

# 取消代理
scoop config rm proxy
```

### 配置 Git 代理（可选）

Scoop 的 bucket（软件仓库）通过 git 拉取，如网络受限可同时为 git 设置代理：

```powershell
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# 取消 git 代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

## 软件安装示例

各软件的安装示例已拆分到 [apps](./apps) 目录：

| 软件 | 文档 | 状态 |
| --- | --- | --- |
| Go | [apps/go/go.md](./apps/go/go.md) | 完整 |
| g（Go 多版本管理） | [apps/go/g.md](./apps/go/g.md) | 完整 |
| PHP | [apps/php.md](./apps/php.md) | 待补充 |
| Nginx | [apps/nginx.md](./apps/nginx.md) | 待补充 |
| MySQL | [apps/mysql.md](./apps/mysql.md) | 待补充 |
| MariaDB | [apps/mariadb.md](./apps/mariadb.md) | 待补充 |
| Java | [apps/java.md](./apps/java.md) | 待补充 |
| NPM / Node.js | [apps/npm.md](./apps/npm.md) | 待补充 |
