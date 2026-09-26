# Go 多版本依赖管理

当多个项目要求不同 Go 版本（如项目 A 需要 Go 1.22、项目 B 需要 Go 1.24）时，有两种主流管理方式：

1. **g**：第三方版本管理工具，全局安装和切换 Go SDK
2. **GOTOOLCHAIN=auto**：Go 1.21+ 官方内置的工具链自动切换，按项目自动下载所需版本

基础概念先明确：

- `go.mod` 中的 `go 1.22`（go 指令）声明本模块要求的**最低 Go 版本**；`toolchain go1.24.3`（toolchain 指令，Go 1.21+）声明建议使用的具体工具链版本。
- 两种方式解决的都是"用哪个 Go SDK"的问题；而**模块依赖版本**（gin v1.9 还是 v1.10）始终由各项目的 `go.mod` / `go.sum` 独立锁定，与此无关。

---

## 方式一：g（第三方版本管理器）

项目地址：<https://github.com/voidint/g>，安装步骤见 [g.md](./g.md)。

### 工作原理

g 把多个 Go 版本下载到自己的主目录，通过修改 `GOROOT` 和符号链接（`~/.g/go` -> 当前版本）决定终端里 `go` 命令指向哪个版本：

```text
~\.g\
├── bin\g.exe
├── go -> versions\go1.22.x        # GOROOT 符号链接，g use 时切换指向
├── versions\
│   ├── go1.22.12\
│   └── go1.24.3\
└── downloads\                     # 安装包缓存
```

### 使用流程

```powershell
# 1.（可选）配置国内下载镜像，否则 g 从 go.dev 查询版本会超时
[Environment]::SetEnvironmentVariable("G_MIRROR", "https://golang.google.cn/dl/", "User")

# 2. 安装项目需要的版本（各自独立目录，可只装一次）
g install 1.22.12
g install 1.24.3

# 3. 进入项目前全局切换
g ls                    # 查看已安装版本，* 为当前版本
g use 1.22.12           # 全局切换（重建 GOROOT 软链接）

# 4. 在项目 A 目录构建
cd D:\code\project-a
go version              # go1.22.12
go build ./...

# 5. 切换到项目 B
g use 1.24.3
cd D:\code\project-b
go build ./...

# 维护
g prune                 # 清理旧版本（每个次版本系列只留最新）
g clean                 # 清理安装包缓存
g uninstall 1.22.12
```

### 特点

- 切换是**全局、手动**的：同一时刻所有终端默认用一个版本，每次换项目要执行 `g use`。
- 可管理**任意历史版本**（包括 Go 1.21 之前的版本），下载行为和目录完全由自己掌控。
- Windows 上依赖符号链接：需要开启「开发者模式」或以管理员身份运行，且 **g 的官方安装脚本要求 PowerShell 7+**（5.1 会报三元运算符解析错误，可手动安装，见 [g.md](./g.md)）。
- 适合：需要频繁在大量版本（含老版本）间切换、要求全局固定默认版本、CI/离线环境。

> 小技巧：配合 [direnv](https://direnv.net/) 等工具可在进入目录时自动执行 `g use`，实现"按项目自动切换"，但这属于额外配置，并非 g 原生能力。

---

## 方式二：GOTOOLCHAIN=auto（官方自动工具链）

从 **Go 1.21** 开始内置，默认开启。无需安装任何第三方工具。

### 工作原理

在模块目录执行任意 go 命令时，go 会读取 `go.mod` 的 `go` / `toolchain` 指令并与当前版本比较：

```text
当前 go 版本 >= 项目要求版本  → 直接用当前 go
当前 go 版本 <  项目要求版本  → 经 GOPROXY 自动下载所需 SDK → 用下载的工具链执行命令
```

下载的工具链以模块形式（`golang.org/toolchain`）存放在 `GOMODCACHE` 中，全局缓存、跨项目复用：

```text
GOMODCACHE\
└── golang.org\
    └── toolchain@v0.0.1-go1.24.3.windows-amd64\
        └── bin\go.exe
```

### 使用流程

```powershell
# 1. 确保使用 Go 1.21+ 作为引导版本，并确认开关（默认即 auto）
go version
go env GOTOOLCHAIN           # 输出 auto

# 2. 确保 GOPROXY 可达（工具链同样通过 GOPROXY 下载）
go env -w GOPROXY=https://goproxy.cn,direct

# 3. 直接进入任何项目构建，无需手动切换
cd D:\code\project-a         # go.mod: go 1.22
go build ./...               # 本机版本满足 → 用本机版本

cd D:\code\project-b         # go.mod: go 1.24
go build ./...               # 本机版本不足 → 自动下载 go1.24 并用它构建
```

升级/固定项目的工具链要求，直接用 `go get` 修改 go.mod：

```powershell
go get go@1.24               # 提高 go 指令到 1.24
go get toolchain@go1.24.3    # 固定/更新 toolchain 指令
```

### GOTOOLCHAIN 取值

```powershell
go env -w GOTOOLCHAIN=auto          # 默认：按需自动下载切换
go env -w GOTOOLCHAIN=local         # 只用本机 go，版本不足直接报错（不下载）
go env -w GOTOOLCHAIN=go1.22.5      # 强制固定使用某个版本
go env -w GOTOOLCHAIN=go1.22.0+auto # 默认用 1.22.0，但若项目要求更高则自动下载
go env -w GOTOOLCHAIN=go1.22.0+path # 默认用 1.22.0，允许使用 PATH 中找到的其他 go
go env -u GOTOOLCHAIN               # 恢复默认
```

### 特点

- **按项目、零手动操作**：切换发生在命令执行瞬间，每个项目各用各的，不存在"忘记 use 导致版本错误"。
- 只能管理 **Go 1.21+** 的项目（引导版本本身也必须是 1.21+；更老的项目无法借此管理）。
- 首次构建不满足要求的项目时需要联网下载；下载源就是 `GOPROXY`，国内配好 `goproxy.cn` 即可。
- 适合：绝大多数日常开发，尤其是项目版本差异由 `go.mod` 声明、希望开箱即用的场景。

---

## 两种方式对比

| 维度 | g | GOTOOLCHAIN=auto |
| --- | --- | --- |
| 来源 | 第三方（voidint/g） | Go 1.21+ 官方内置 |
| 切换方式 | 手动 `g use`，全局生效 | 自动，按项目 go.mod 即时切换 |
| 可管理版本 | 任意历史版本（含 < 1.21） | 仅 Go 1.21+ |
| 额外安装 | 需要安装 g | 无需额外工具 |
| 下载来源 | `G_MIRROR` 指定的镜像 | `GOPROXY` 指定的模块代理 |
| Windows 要求 | 符号链接权限（开发者模式/管理员）、PS7 安装脚本 | 无特殊要求 |
| 离线/CI 可控性 | 强（版本预先装好） | 需预热缓存或固定 `local` |
| 默认版本 | 由 `g use` 全局指定 | 本机安装的 go 即为默认 |

## 二者如何配合

它们可以共存，但要理解优先级：**只要 `GOTOOLCHAIN=auto` 生效，即使 g 已 `use` 某个版本，遇到要求更高版本的项目时 go 仍会自动下载另一套工具链**，g 的版本目录反而不被使用。

推荐组合：

- 用 g（或 Scoop）安装一个 **Go 1.21+ 作为默认引导版本**；
- 日常保持 `GOTOOLCHAIN=auto`，各项目自动适配；
- 只有以下情况临时设 `GOTOOLCHAIN=local`，把控制权完全交给 g：

```powershell
go env -w GOTOOLCHAIN=local   # 版本不足时报错，由你手动 g install / g use
# 验证完毕后恢复
go env -u GOTOOLCHAIN
```

## 选型建议

- **新项目为主、团队 go.mod 规范、不想手动切换** → `GOTOOLCHAIN=auto`（默认即可，最省心）。
- **需要维护 Go 1.20 及更早的老项目、要全局锁定版本、离线/CI 环境** → 使用 g。
- 二者并非互斥：默认 auto 处理绝大多数项目，g 用于老版本和特殊管控场景。

> 补充：还可以用 Scoop 的 `go1xx` 包（如 `go124`）+ `scoop reset` 管理版本，见 [go.md](./go.md) 第 6 节，适合"装一两个备用版本偶尔切换"的轻量场景。
