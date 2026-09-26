# Go（Scoop 安装）

> Go 官方版本管理工具 `g` 的安装见 [g.md](./g.md)

## 1. 搜索软件包

```powershell
scoop search go
```

## 2. 安装（单版本）

```powershell
# go 在官方 main 仓库中，安装最新稳定版
scoop install go
```

## 3. 验证安装

```powershell
go version
go env GOPATH
```

## 4. 配置国内模块代理和工作目录

```powershell
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
go env -w GOPATH=D:\code\go
```

## 5. 更新与卸载

```powershell
scoop update go
scoop uninstall go
```

## 6. 多版本安装与管理

### 6.1 添加 versions 仓库

历史/次新版本位于官方 `versions` 仓库，包名按 `go` + 主次版本号命名（如 `go124`、`go123`）：

```powershell
scoop bucket add versions
scoop search go
```

当前可用的版本包：`go113`、`go114`、`go115`、`go116`、`go117`、`go118`、`go119`、`go120`、`go121`、`go122`、`go123`、`go124`

### 6.2 并行安装多个版本

各版本安装到各自独立的 app 目录，可以同时存在：

```powershell
# main 仓库的最新版 + versions 仓库的指定次版本
scoop install go
scoop install go124
scoop install go123
```

> 注意：这些包都会生成同名的 `go.exe`、`gofmt.exe` shim。安装第二个版本时 Scoop 会提示 shim 冲突（`Creating shim for 'go'` 覆盖），PATH 中的 `go` 命令只会指向最后一次操作的版本。

### 6.3 查看已安装版本

```powershell
scoop list | Select-String go
# 输出示例：
# go     1.25.x   main
# go123  1.23.x   versions
# go124  1.24.13  versions
```

### 6.4 切换当前使用版本（scoop reset）

`scoop reset` 会重建指定 app 的 shim，使 PATH 中的 `go` 指向该版本：

```powershell
scoop reset go123      # 切换到 go123
go version             # go version go1.23.x windows/amd64

scoop reset go124      # 切换到 go124
scoop reset go         # 切换回 main 仓库最新版
```

> 新开终端窗口生效；也可用 `scoop reset *` 重置全部 app 的 shim。

### 6.5 不切换 shim，临时使用指定版本

直接用对应版本的绝对路径调用，避免改变全局默认：

```powershell
# current junction 指向该包当前版本
& "$env:SCOOP\apps\go123\current\bin\go.exe" version
& 'D:\Program Files\Scoop\apps\go123\current\bin\go.exe' version
```

也可在 PowerShell `$PROFILE` 中设置别名：

```powershell
Set-Alias go123 'D:\Program Files\Scoop\apps\go123\current\bin\go.exe'
Set-Alias go124 'D:\Program Files\Scoop\apps\go124\current\bin\go.exe'
```

### 6.6 各版本独立更新与卸载

```powershell
scoop update go124           # 只更新 1.24 的补丁版本（1.24.13 -> 1.24.14）
scoop uninstall go123        # 卸载 1.23，不影响其他版本
scoop uninstall go123 -p     # -p 同时删除 persist 目录（保留的用户数据）
```

> main 包 `go` 跟随最新稳定版；`go1xx` 包只更新对应次版本系列的补丁号，不会跨次版本升级。

### 6.7 方案选择

- 需要**频繁在多个 Go 版本间切换**：建议直接使用 [g](./g.md)，一条命令 `g use 1.23.5` 即可，无需手动 reset。
- 只需要**保留一两个备用版本、偶尔切换**：使用本节的 Scoop `go1xx` 包 + `scoop reset` 即可。

## 7. 安装后的目录结构

单版本：

```text
<ScoopDir>\
├── apps\
│   └── go\
│       ├── current -> app-x.y.z   # 指向当前版本的软链接（junction）
│       └── app-x.y.z\
│           ├── bin\
│           │   ├── go.exe
│           │   └── gofmt.exe
│           ├── src\
│           ├── lib\
│           └── ...
└── shims\                        # 自动加入 PATH 的命令垫片
    ├── go.exe
    └── gofmt.exe
```

多版本并行安装后：

```text
<ScoopDir>\
├── apps\
│   ├── go\                       # main 仓库最新版
│   │   ├── current -> app-1.25.x
│   │   └── app-1.25.x\
│   ├── go124\
│   │   ├── current -> 1.24.13
│   │   └── 1.24.13\bin\go.exe
│   └── go123\
│       ├── current -> 1.23.x
│       └── 1.23.x\bin\go.exe
└── shims\
    ├── go.exe                    # 由 scoop reset 决定指向哪个版本
    └── gofmt.exe
```
