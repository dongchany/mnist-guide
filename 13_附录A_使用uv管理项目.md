# 附录 A：使用 uv 管理项目

> **本章目标**：学会用 uv 创建项目、管理依赖、运行代码，把我们的 MNIST 项目变成一个规范的 Python 工程。
> **预计时间**：1 小时

---

## A.1 为什么用 uv？

传统的 Python 项目管理需要组合好几个工具：`python` 安装、`venv` 创建虚拟环境、`pip` 安装包、`pip freeze` 导出依赖……步骤多、容易出错。

**uv** 是一个用 Rust 编写的全能工具，把上面所有事情合并成一个命令行工具，而且比传统工具快 10–100 倍。它相当于 `pip + venv + pyenv + pip-tools` 的合体。

我们的 MNIST 作业就是用 uv 来管理的。

---

## A.2 安装 uv

### macOS / Linux

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

安装完成后，重启终端或执行 `source ~/.bashrc`（Linux）/ `source ~/.zshrc`（macOS）使 `uv` 命令生效。

### Windows

PowerShell（推荐）：

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

安装完成后重启 PowerShell。

### 所有平台通用：用 pip 安装

如果上面的方式遇到问题，也可以：

```bash
pip install uv       # Windows
pip3 install uv      # macOS / Linux
```

### 验证安装

```bash
uv --version
# 应显示 uv 0.6.x 或更高
```

---

## A.3 从零创建项目

### 初始化

```bash
uv init mnist-homework
cd mnist-homework
```

这会生成以下结构：

```
mnist-homework/
├── pyproject.toml      ← 项目配置（核心文件）
├── .python-version     ← 指定 Python 版本
├── README.md
└── hello.py            ← 示例文件（可删除）
```

### pyproject.toml 详解

这是项目的"身份证"，记录了所有配置信息：

```toml
[project]
name = "mnist-homework"
version = "0.1.0"
description = "MNIST digit classification with FC and CNN from scratch"
requires-python = ">=3.10"
dependencies = []
```

### 指定 Python 版本

```bash
uv python pin 3.12
# Creates/updates .python-version file
```

uv 会自动下载你需要的 Python 版本，不需要手动安装。

---

## A.4 管理依赖

### 添加依赖

```bash
# Add packages one by one
uv add numpy
uv add matplotlib

# Or add multiple at once
uv add numpy matplotlib
```

每次 `uv add`，uv 会做三件事：
1. 更新 `pyproject.toml` 的 `dependencies` 列表
2. 解析所有依赖关系，生成 `uv.lock`（锁定文件）
3. 同步虚拟环境（自动创建 `.venv` 并安装包）

### 添加开发依赖

测试和调试用的包不应该算作项目的正式依赖：

```bash
# Add as development dependencies
uv add --dev scikit-learn
uv add --dev pytest
```

这些会被放到 `pyproject.toml` 的 `[tool.uv.dev-dependencies]` 中。

### 查看当前依赖

```bash
uv tree
```

输出类似：

```
mnist-homework v0.1.0
├── numpy v2.2.3
├── matplotlib v3.10.0
│   ├── contourpy v1.3.1
│   ├── numpy v2.2.3
│   └── ...
└── scikit-learn v1.6.1 (dev)
    ├── numpy v2.2.3
    └── ...
```

### 移除依赖

```bash
uv remove some-package
```

---

## A.5 运行代码

### 基本用法

```bash
# Run a script
uv run python main.py

# Run tests
uv run python tests.py

# Run with verbose output
uv run python tests.py -v
```

`uv run` 确保代码在正确的虚拟环境中执行，所有依赖都已安装。你**不需要**手动激活虚拟环境。

### 也可以手动激活环境

如果你更习惯传统方式，uv 创建的虚拟环境和普通 venv 完全兼容：

```bash
# macOS / Linux
source .venv/bin/activate

# Windows (PowerShell)
.venv\Scripts\Activate.ps1

# Windows (CMD)
.venv\Scripts\activate.bat
```

激活后就可以直接用 `python` 命令运行脚本：

```bash
python main.py
python tests.py -v
```

> **提示**：如果你用 `uv run`，完全不需要手动激活环境。`uv run` 是推荐方式。

---

## A.6 同步与复现环境

### 从已有项目恢复

当你从 Git 克隆一个 uv 项目后，只需一条命令：

```bash
uv sync
```

这会读取 `uv.lock`，安装**完全相同版本**的所有依赖。任何人在任何机器上都能复现一模一样的环境。

### 只同步生产依赖（不含 dev）

```bash
uv sync --no-dev
```

---

## A.7 我们项目的完整 pyproject.toml

```toml
[project]
name = "mnist-homework"
version = "0.1.0"
description = "MNIST digit classification using FC and CNN networks from scratch"
readme = "README.md"
requires-python = ">=3.10"
dependencies = [
    "numpy>=2.0",
    "matplotlib>=3.8",
]

[tool.uv]
dev-dependencies = [
    "scikit-learn>=1.5",
]
```

**说明**：
- `numpy` 和 `matplotlib` 是运行项目必需的
- `scikit-learn` 只在测试中用来交叉验证指标，放在 dev 依赖里
- 我们没有依赖任何深度学习框架——这正是本作业的意义所在

---

## A.8 项目完整结构

```
mnist-homework/
├── pyproject.toml           ← 项目配置 + 依赖声明
├── uv.lock                  ← 依赖锁定文件（自动生成，提交到 Git）
├── .python-version          ← Python 版本
├── .venv/                   ← 虚拟环境（不提交到 Git）
├── MNIST_data/              ← 数据文件（不提交到 Git）
│   ├── train-images.idx3-ubyte
│   └── ...
├── activations.py
├── fc_network.py
├── cnn_network.py
├── metrics.py
├── data_loader.py
├── plotting.py
├── main.py
├── tests.py
├── generate_report.py
└── README.md
```

### .gitignore

```gitignore
.venv/
MNIST_data/
__pycache__/
*.pyc
output/

# OS-specific
.DS_Store          # macOS
Thumbs.db          # Windows
```

---

## A.9 常用命令速查表

| 场景 | 命令 |
|------|------|
| 创建新项目 | `uv init my-project` |
| 安装 Python | `uv python install 3.12` |
| 固定 Python 版本 | `uv python pin 3.12` |
| 添加依赖 | `uv add numpy` |
| 添加开发依赖 | `uv add --dev pytest` |
| 移除依赖 | `uv remove numpy` |
| 查看依赖树 | `uv tree` |
| 同步环境 | `uv sync` |
| 运行脚本 | `uv run python main.py` |
| 运行测试 | `uv run python tests.py -v` |
| 升级 uv 自身 | `uv self update` |
| 升级某个包 | `uv add --upgrade numpy` |

---

## A.10 uv vs 传统方式对比

### 传统方式（pip + venv）

```bash
# macOS / Linux
python3 -m venv .venv
source .venv/bin/activate
pip3 install numpy matplotlib
pip3 install scikit-learn
pip3 freeze > requirements.txt
python3 main.py
# 换台电脑...
pip3 install -r requirements.txt    # 版本可能对不上

# Windows
python -m venv .venv
.venv\Scripts\activate.bat
pip install numpy matplotlib
pip install scikit-learn
pip freeze > requirements.txt
python main.py
# 换台电脑...
pip install -r requirements.txt
```

### uv 方式（三个平台完全一样！）

```bash
uv init mnist-homework
cd mnist-homework
uv add numpy matplotlib
uv add --dev scikit-learn
uv run python main.py
# 换台电脑...
uv sync                            # 精确复现
```

不仅步骤更少、速度更快，而且**命令在所有平台上完全一致**——这是 uv 的一大优势。

---

## A.11 FAQ

**Q：uv 会替代 pip 吗？**
uv 提供了完整的 pip 兼容接口（`uv pip install`、`uv pip freeze` 等），可以作为 pip 的直接替代品。但推荐使用 uv 原生命令（`uv add`、`uv sync`），功能更强大。

**Q：`uv.lock` 需要提交到 Git 吗？**
**需要**。它确保团队成员和 CI 环境使用完全相同的依赖版本。

**Q：`.venv` 需要提交吗？**
**不需要**。虚拟环境可以通过 `uv sync` 随时重建。

**Q：Windows 上 PowerShell 报错"无法运行脚本"怎么办？**
这是 PowerShell 的执行策略限制。以管理员身份打开 PowerShell，运行：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

之后就可以正常使用 uv 和激活虚拟环境了。

**Q：Windows 上 `python` 命令打开了 Microsoft Store 怎么办？**
这是 Windows 10/11 的"应用别名"导致的。去 **设置 → 应用 → 应用执行别名**，关掉"python.exe"和"python3.exe"的应用安装程序。然后确保你的 Python 安装路径在 PATH 中。用 uv 则完全不用担心这个问题——`uv run python` 会自动找到正确的 Python。

**Q：我之前用 `requirements.txt`，能迁移吗？**
可以。在已有项目目录中运行 `uv init`，然后用 `uv add` 逐个添加之前 requirements.txt 中的包，或者直接：

```bash
# macOS / Linux
uv add $(cat requirements.txt | grep -v '^#' | tr '\n' ' ')

# Windows (PowerShell)
Get-Content requirements.txt | Where-Object { $_ -notmatch '^#' } | ForEach-Object { uv add $_ }
```

---

## A.12 动手练习

**练习 1**：用 uv 从零创建我们的 MNIST 项目，添加 numpy 和 matplotlib 依赖，运行 `uv run python -c "import numpy; print(numpy.__version__)"`。

**练习 2**：检查 `uv.lock` 文件内容，找到 numpy 的精确版本号。思考为什么需要锁定到精确版本。

**练习 3**：删除 `.venv` 目录，然后运行 `uv sync` 重建环境。验证所有代码依然能正常运行。
