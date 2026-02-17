# 第 0 章：环境准备与 NumPy 快速入门

> **本章目标**：搭建开发环境，掌握 NumPy 中与神经网络相关的核心操作。
> **预计时间**：1–2 小时

---

## 0.1 跨平台说明

本教程支持 **Windows**、**macOS** 和 **Linux** 三个平台。绝大多数 Python 代码完全跨平台，但终端命令有些差异。本教程的约定：

**终端命令格式**：当命令在所有平台都一样时，直接用一个代码块。当存在差异时，会标注各平台的写法。

**Python 命令**：

| 平台 | 命令 | 说明 |
|------|------|------|
| Windows | `python` | 一般安装后就是 `python` |
| macOS | `python3` | 系统自带的 `python` 可能是 2.x |
| Linux | `python3` | 同上 |

如果你用 **uv**（推荐，见附录 A），统一用 `uv run python` 即可，不用关心平台差异。

**路径分隔符**：Python 代码中统一用 `/`（Python 在 Windows 上也能正确处理 `/`），终端命令中 Windows 用 `\`，macOS/Linux 用 `/`。

---

## 0.2 安装 Python 与 NumPy

### 方式一：Anaconda（最简单，推荐新手）

从 https://www.anaconda.com/download 下载安装包。Anaconda 自带 Python、NumPy、Matplotlib 等常用库，**三个平台都支持**。

安装后验证：

```bash
python --version        # Windows
python3 --version       # macOS / Linux
```

### 方式二：uv（推荐，本作业使用）

uv 会自动管理 Python 版本和依赖，详见附录 A。安装 uv 后：

```bash
uv init mnist-homework && cd mnist-homework
uv add numpy matplotlib
```

### 方式三：手动安装 Python + pip

从 https://www.python.org/downloads/ 下载安装。然后：

```bash
# Windows
pip install numpy matplotlib

# macOS / Linux
pip3 install numpy matplotlib
```

我们还需要 MNIST 数据集。从 http://yann.lecun.com/exdb/mnist/ 下载以下四个文件：

```
train-images.idx3-ubyte    # 训练图片（60000张）
train-labels.idx1-ubyte    # 训练标签
t10k-images.idx3-ubyte     # 测试图片（10000张）
t10k-labels.idx1-ubyte     # 测试标签
```

放入项目目录下的 `MNIST_data` 文件夹：

```bash
# macOS / Linux
mkdir -p MNIST_data

# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path MNIST_data

# Windows (CMD)
mkdir MNIST_data
```

---

## 0.3 为什么是 NumPy？

神经网络的核心运算是**矩阵乘法**和**逐元素运算**。Python 自带的 list 做这些事情很慢，而 NumPy 底层用 C 语言实现，快 100 倍以上。

一个直觉：用 Python list 算 100 万次乘法需要几秒，NumPy 只需要几毫秒。

---

## 0.4 NumPy 核心操作速查

### 创建数组

```python
import numpy as np

# From list
a = np.array([1, 2, 3])          # shape: (3,)
M = np.array([[1, 2], [3, 4]])   # shape: (2, 2)

# Special arrays
zeros = np.zeros((3, 4))         # 3x4 all zeros
ones  = np.ones((2, 3))          # 2x3 all ones
rand  = np.random.randn(3, 4)    # 3x4 random (standard normal)
```

### 数组的"形状"（shape）

形状是 NumPy 最重要的概念之一。它告诉你数据的维度结构。

```python
x = np.random.randn(5, 3)
print(x.shape)    # (5, 3) — 5 行 3 列

# Reshape: change shape without changing data
flat = x.reshape(15)          # flatten to 1D
back = flat.reshape(5, 3)     # restore to 2D
```

**在神经网络中**：一张 28×28 的灰度图可以表示为 shape (28, 28) 的数组，也可以展平为 shape (784,) 的一维向量。

### 矩阵乘法（最重要！）

矩阵乘法是神经网络的心脏。

```python
# Matrix multiplication: @ operator or np.dot
A = np.random.randn(3, 4)    # 3x4 matrix
B = np.random.randn(4, 5)    # 4x5 matrix
C = A @ B                     # result: 3x5 matrix
print(C.shape)                # (3, 5)
```

**规则**：(m, n) @ (n, k) → (m, k)。中间的 n 必须匹配。

**直觉**：如果 A 的每一行代表一个"数据样本"，B 的每一列代表一个"权重组合"，那么 A @ B 就是把每个样本经过线性变换后的结果。这就是神经网络每一层在做的事情。

### 逐元素运算

```python
a = np.array([1, -2, 3, -4])

np.maximum(0, a)         # [1, 0, 3, 0] — this is ReLU!
np.exp(a)                # e^x for each element
np.log(a.clip(1e-12))    # log(x), clip to avoid log(0)
a > 0                    # [True, False, True, False]
```

### 广播（Broadcasting）

当两个数组形状不同时，NumPy 会自动"广播"较小的数组：

```python
X = np.random.randn(100, 784)   # 100 samples, 784 features
b = np.zeros((1, 784))          # 1 bias vector

result = X + b   # b is broadcast to (100, 784)
                  # each row of X gets the same b added
```

**在神经网络中**：偏置项 (bias) 就是这样加到每个样本上的。

### 求和与均值

```python
X = np.random.randn(5, 3)

X.sum()              # sum of ALL elements
X.sum(axis=0)        # sum along rows → shape (3,)
X.sum(axis=1)        # sum along columns → shape (5,)
X.mean(axis=0)       # mean along rows → shape (3,)

# keepdims preserves dimensions (useful for broadcasting)
X.sum(axis=1, keepdims=True)   # shape (5, 1) instead of (5,)
```

### 索引与花式索引

```python
labels = np.array([0, 3, 7, 2, 9])
probs  = np.random.rand(5, 10)    # 5 samples, 10 classes

# Get the probability assigned to the correct class for each sample
correct_probs = probs[np.arange(5), labels]
# This picks probs[0,0], probs[1,3], probs[2,7], probs[3,2], probs[4,9]
```

这个技巧在计算交叉熵损失时非常常用。

---

## 0.5 动手练习

**练习 1**：创建一个 shape 为 (100, 784) 的随机矩阵 X，和一个 shape 为 (784, 10) 的随机矩阵 W。计算 X @ W，验证结果的 shape 是 (100, 10)。

**练习 2**：实现一个函数，把整数标签 `[0, 3, 7]` 转换为 one-hot 编码（提示：先创建全零矩阵，再用花式索引填 1）：

```python
def one_hot(labels, num_classes=10):
    """
    Convert [0, 3, 7] to:
    [[1,0,0,0,0,0,0,0,0,0],
     [0,0,0,1,0,0,0,0,0,0],
     [0,0,0,0,0,0,0,1,0,0]]
    """
    # Your code here
    pass
```

**练习 3**：给定一个数组 `z = np.array([-2, -1, 0, 1, 2])`，写出把所有负数变成 0、正数保留的操作（一行代码）。恭喜，你刚刚实现了 ReLU 激活函数！

---

## 0.6 本章小结

你已经掌握了：

- NumPy 数组的创建与 shape 概念
- 矩阵乘法 `@`（神经网络每一层的核心）
- 逐元素运算（激活函数的基础）
- 广播机制（偏置项的加法）
- 花式索引（损失函数的计算）

这些就是用 NumPy 写神经网络所需的全部工具。下一章，我们从"什么是机器学习"开始，理解我们要解决的问题。
