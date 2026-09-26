# g（Go 多版本管理工具）

项目地址：<https://github.com/voidint/g>

`g` 是一个支持 Linux、macOS、Windows 的 Go 版本管理工具，可以安装、切换、卸载多个 Go 版本。

> 与官方 GOTOOLCHAIN=auto 方案的详细对比见 [version-management.md](./version-management.md)。

## 1. 安装

> g 暂无官方 Scoop 包，以下两种方式任选其一。

### 方式一：官方安装脚本（推荐，需 PowerShell 7+）

> 官方脚本使用了三元运算符（`? :`），只能在 **PowerShell 7（pwsh）及以上**运行。在 Windows PowerShell 5.1 中执行会报 `表达式或语句中包含意外的标记"?"` 解析错误。可用 `pwsh --version` 检查，未安装则 `scoop install pwsh`。

进入 PowerShell 7 后执行：

```powershell
iwr https://raw.githubusercontent.com/voidint/g/master/install.ps1 -useb | iex
```

安装完成后如提示 PATH 未生效，重新打开终端即可。

### 方式二：go install（需要先装有 Go）

若已通过 Scoop 安装了 Go（见 [go.md](./go.md)），可直接：

```powershell
go install github.com/voidint/g@latest
```

安装后 `g.exe` 位于 `%GOPATH%\bin`（如 `D:\code\go\bin`），需确保该目录在 PATH 中。

### 手动安装（兼容 PowerShell 5.1）

1. 创建目录：`mkdir ~/.g/bin`
2. 从 [releases](https://github.com/voidint/g/releases) 下载 Windows 压缩包，解压后将 `g.exe` 放入 `~/.g/bin`
3. 执行 `code $PROFILE` 打开 PowerShell 配置文件，加入：

```powershell
$env:GOROOT="$HOME\.g\go"
$env:Path=-join("$HOME\.g\bin;", "$env:GOROOT\bin;", "$env:Path")
```

4. 重新打开终端即可使用。若 `g` 与 git 别名冲突，可将 `g.exe` 改名为 `gvm.exe`

## 2. 配置国内镜像

大陆访问 Go 官网受限，通过 `G_MIRROR` 指定镜像：

```powershell
# 临时设置
$env:G_MIRROR = "https://golang.google.cn/dl/"

# 永久生效（写入用户环境变量）
[Environment]::SetEnvironmentVariable("G_MIRROR", "https://golang.google.cn/dl/", "User")
```

其他可用镜像：

- 阿里云：`https://mirrors.aliyun.com/golang/`
- 南京大学：`https://mirrors.nju.edu.cn/golang/`
- 中科大：`https://mirrors.ustc.edu.cn/golang/`

## 3. 使用

```powershell
# 查询可安装的稳定版本
g ls-remote stable

# 安装指定版本
g install 1.20.5

# 查询已安装版本（* 为当前使用版本）
g ls

# 切换到已安装的其他版本
g use 1.19.10

# 卸载指定版本
g uninstall 1.19.10

# 清理旧版本（每个次版本系列只保留最新，当前使用的不会被删）
g prune --dry-run   # 预览
g prune             # 执行

# 清理安装包缓存
g clean

# 查看 g 自身版本
g version

# g 自更新 / 自卸载
g self update
g self uninstall
```

## 4. 代理

g 支持通过环境变量设置代理：

```powershell
$env:HTTP_PROXY  = "http://127.0.0.1:7890"
$env:HTTPS_PROXY = "http://127.0.0.1:7890"
```

## 目录结构

默认主目录为 `~/.g`（可用 `G_HOME` 自定义，需同时设置 `G_EXPERIMENTAL=true`）：

```text
~\.g\
├── bin\
│   └── g.exe
├── go\                 # 当前启用版本（GOROOT，符号链接）
│   └── bin\go.exe
├── versions\           # 已下载的各版本 Go
│   ├── go1.19.10\
│   └── go1.20.5\
├── downloads\          # 安装包缓存（可用 g clean 清理）
└── env
```
