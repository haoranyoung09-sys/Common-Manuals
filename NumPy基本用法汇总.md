# NumPy 基本用法汇总

> 本文档系统整理了 NumPy 的核心用法，涵盖数组创建、索引切片、运算广播、统计排序、形状操作、随机数、线性代数等常见知识点，适用于入门与速查。

---

## 目录

1. [NumPy 简介](#1-numpy-简介)
2. [数组创建](#2-数组创建)
3. [数组属性与数据类型](#3-数组属性与数据类型)
4. [索引与切片](#4-索引与切片)
5. [数组运算与广播](#5-数组运算与广播)
6. [形状操作](#6-形状操作)
7. [统计与聚合](#7-统计与聚合)
8. [排序与搜索](#8-排序与搜索)
9. [随机数](#9-随机数)
10. [线性代数](#10-线性代数)
11. [文件 I/O](#11-文件-io)
12. [常用技巧与注意事项](#12-常用技巧与注意事项)

---

## 1. NumPy 简介

NumPy（Numerical Python）是 Python 科学计算的基础库，提供了高性能的多维数组对象 `ndarray` 以及对数组进行操作的各种函数。

**核心优势：**
- 内存连续存储，访问速度快
- 向量化运算，避免显式循环
- 广播机制，支持不同形状数组运算
- 丰富的数学、线性代数、随机数功能

```python
import numpy as np
```

---

## 2. 数组创建

### 2.1 从列表/元组创建

```python
# 一维数组
arr1 = np.array([1, 2, 3, 4, 5])
print(arr1)  # [1 2 3 4 5]

# 二维数组
arr2 = np.array([[1, 2, 3], [4, 5, 6]])
print(arr2)
# [[1 2 3]
#  [4 5 6]]

# 指定数据类型
arr_f = np.array([1, 2, 3], dtype=float)
print(arr_f)  # [1. 2. 3.]
```

### 2.2 常用创建函数

```python
# 全零数组
np.zeros((2, 3))           # 2行3列全零
np.zeros_like(arr)         # 形状与 arr 相同的全零数组

# 全一数组
np.ones((3, 3))            # 3行3列全一
np.ones_like(arr)

# 空数组（未初始化，速度快）
np.empty((2, 2))

# 填充指定值
np.full((2, 3), 7)         # 全部填充为 7
np.full_like(arr, 9)

# 单位矩阵
np.eye(4)                  # 4x4 单位矩阵
np.identity(3)

# 对角矩阵
np.diag([1, 2, 3])         # 对角线为 [1,2,3] 的方阵
```

### 2.3 序列生成

```python
# arange：类似 range，支持浮点数步长
np.arange(0, 10, 2)        # [0 2 4 6 8]
np.arange(0, 1, 0.1)       # [0.  0.1 0.2 ... 0.9]

# linspace：等间隔的 N 个数（包含端点）
np.linspace(0, 10, 5)      # [ 0.   2.5  5.   7.5 10. ]

# logspace：对数刻度等间隔
np.logspace(0, 3, 4)       # [   1.   10.  100. 1000.]

# geomspace：几何级数
np.geomspace(1, 1000, 4)   # [   1.   10.  100. 1000.]
```

### 2.4 随机数组创建（详见第 9 节）

```python
np.random.rand(3, 3)       # [0,1) 均匀分布
np.random.randn(3, 3)      # 标准正态分布
np.random.randint(0, 10, (2, 3))  # 随机整数
```

---

## 3. 数组属性与数据类型

### 3.1 核心属性

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

arr.shape       # (2, 3)       形状
arr.ndim        # 2            维度（轴数）
arr.size        # 6            元素总数
arr.dtype       # int64        数据类型
arr.itemsize    # 8            每个元素字节数
arr.nbytes      # 48           总字节数
arr.T           # 转置
```

### 3.2 数据类型（dtype）

| 类型 | 说明 |
|------|------|
| `int8`, `int16`, `int32`, `int64` | 有符号整数 |
| `uint8`, `uint16`, `uint32`, `uint64` | 无符号整数 |
| `float16`, `float32`, `float64` | 浮点数 |
| `complex64`, `complex128` | 复数 |
| `bool` | 布尔值 |
| `string_`, `unicode_` | 字符串 |

```python
# 转换数据类型
arr = np.array([1, 2, 3])
arr.astype(float)        # 转为 float64
arr.astype(np.float32)   # 转为 float32
arr.astype(str)          # 转为字符串

# 查看类型
np.int64(42)
np.float64(3.14)
```

---

## 4. 索引与切片

### 4.1 基本索引

```python
arr = np.arange(10)  # [0 1 2 3 4 5 6 7 8 9]

arr[0]      # 0
arr[-1]     # 9
arr[2:5]    # [2 3 4]
arr[:5]     # [0 1 2 3 4]
arr[5:]     # [5 6 7 8 9]
arr[::2]    # [0 2 4 6 8]  步长为2
arr[::-1]   # [9 8 7 6 5 4 3 2 1 0]  反转
```

### 4.2 多维索引

```python
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

arr2d[0, 1]        # 2（第0行第1列）
arr2d[0][1]        # 2（等价但稍慢）
arr2d[:2, 1:]      # [[2 3] [5 6]]
arr2d[:, 0]        # [1 4 7]  取第0列
arr2d[1, :]        # [4 5 6]  取第1行
```

### 4.3 布尔索引（掩码）

```python
arr = np.arange(10)
mask = arr > 5
print(mask)         # [False False False False False False  True  True  True  True]
print(arr[mask])    # [6 7 8 9]

# 多条件（注意用 & | ~，不能用 and/or/not）
arr[(arr > 2) & (arr < 8)]   # [3 4 5 6 7]
arr[(arr < 3) | (arr > 7)]   # [0 1 2 8 9]
arr[~(arr == 5)]             # 排除等于5的元素

# 二维布尔索引
data = np.random.randn(4, 3)
rows = np.array(['A', 'B', 'A', 'C'])
data[rows == 'A']            # 选出对应行
```

### 4.4 花式索引（Fancy Indexing）

```python
arr = np.arange(10)
arr[[0, 2, 4, 6]]      # [0 2 4 6]

# 二维花式索引
arr2d = np.arange(12).reshape(3, 4)
arr2d[[0, 2]]          # 取第0行和第2行
arr2d[[0, 2], [1, 3]]  # 取 (0,1) 和 (2,3) 位置的元素
arr2d[np.ix_([0, 2], [1, 3])]  # 取子矩阵
```

> **注意**：花式索引返回的是**副本**，而切片返回的是**视图**。

### 4.5 视图 vs 副本

```python
arr = np.arange(10)

# 切片是视图，修改会影响原数组
view = arr[2:5]
view[0] = 99
print(arr)   # [ 0  1 99  3  4  5  6  7  8  9]

# 副本：显式复制
copy = arr[2:5].copy()
copy[0] = 100
print(arr)   # 不受影响

# 花式索引是副本
fancy = arr[[0, 1, 2]]
fancy[0] = 200
print(arr)   # 不受影响
```

---

## 5. 数组运算与广播

### 5.1 逐元素运算

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

a + b        # [5 7 9]
a - b        # [-3 -3 -3]
a * b        # [ 4 10 18]
a / b        # [0.25 0.4  0.5 ]
a ** 2       # [1 4 9]
a % 2        # [1 0 1]
```

### 5.2 数学函数

```python
np.sqrt(a)       # 平方根
np.exp(a)        # e 的幂
np.log(a)        # 自然对数
np.log2(a)       # 以2为底
np.log10(a)      # 以10为底
np.sin(a)        # 正弦
np.cos(a)        # 余弦
np.tan(a)        # 正切
np.abs(a)        # 绝对值
np.round(a, 2)   # 四舍五入到2位小数
np.floor(a)      # 向下取整
np.ceil(a)       # 向上取整
np.mod(a, b)     # 取模
```

### 5.3 比较运算

```python
a = np.array([1, 2, 3])
b = np.array([0, 2, 4])

a == b       # [False  True False]
a != b       # [ True False  True]
a > b        # [ True False False]
a < b        # [False False  True]

np.array_equal(a, b)   # False，整体比较
np.allclose(a, b)      # 浮点数近似比较
```

### 5.4 广播机制

广播允许不同形状的数组进行运算，规则如下：
1. 从最右边的维度开始比较
2. 维度相等，或其中一个为 1，则兼容
3. 缺失的维度视为 1

```python
# 标量广播
a = np.array([1, 2, 3])
a + 10        # [11 12 13]

# 行向量与列向量
a = np.array([[1], [2], [3]])     # shape (3,1)
b = np.array([10, 20, 30])        # shape (3,)
a + b
# [[11 21 31]
#  [12 22 32]
#  [13 23 33]]

# 不兼容的形状会报错
try:
    np.array([1, 2, 3]) + np.array([1, 2])
except ValueError as e:
    print(e)  # operands could not be broadcast together
```

---

## 6. 形状操作

### 6.1 reshape 与 resize

```python
arr = np.arange(12)

# reshape：不改变数据，返回新视图（可能是副本）
arr.reshape(3, 4)
arr.reshape(2, -1)    # -1 自动计算

# resize：原地修改，填充或截断
arr2 = np.arange(5)
np.resize(arr2, (2, 3))   # 重复填充 [[0 1 2] [3 4 0]]
```

### 6.2 展平

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

arr.flatten()    # 返回副本 [1 2 3 4 5 6]
arr.ravel()      # 返回视图（优先）[1 2 3 4 5 6]
arr.reshape(-1)  # 等价于 ravel
```

### 6.3 转置与轴变换

```python
arr = np.arange(6).reshape(2, 3)

arr.T              # 转置
arr.transpose()    # 等价
arr.transpose(1, 0)  # 指定轴顺序

# 三维数组轴变换
arr3d = np.arange(24).reshape(2, 3, 4)
arr3d.transpose(0, 2, 1)   # 交换轴1和轴2
arr3d.swapaxes(1, 2)       # 等价
```

### 6.4 拼接与分割

```python
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

# 拼接
np.concatenate([a, b], axis=0)   # 垂直拼接
np.concatenate([a, b], axis=1)   # 水平拼接
np.vstack([a, b])                # 垂直（等价 axis=0）
np.hstack([a, b])                # 水平（等价 axis=1）
np.dstack([a, b])                # 深度拼接（第三维）

# 分割
arr = np.arange(12).reshape(3, 4)
np.split(arr, 3, axis=0)         # 垂直分成3份
np.vsplit(arr, 3)                # 等价
np.hsplit(arr, 2)                # 水平分成2份
```

### 6.5 添加与删除

```python
arr = np.array([1, 2, 3])

np.append(arr, [4, 5])           # [1 2 3 4 5]
np.insert(arr, 1, [10, 20])      # [ 1 10 20  2  3]
np.delete(arr, 1)                # [1 3]
```

---

## 7. 统计与聚合

### 7.1 基础统计函数

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

arr.sum()          # 21  总和
arr.mean()         # 3.5 平均值
arr.std()          # 标准差
arr.var()          # 方差
arr.min()          # 1  最小值
arr.max()          # 6  最大值
arr.argmin()       # 0  最小值索引
arr.argmax()       # 5  最大值索引
arr.cumsum()       # [ 1  3  6 10 15 21]  累加
arr.cumprod()      # [  1   2   6  24 120 720]  累乘
```

### 7.2 沿轴计算

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

arr.sum(axis=0)    # [5 7 9]        按列求和
arr.sum(axis=1)    # [ 6 15]        按行求和
arr.mean(axis=0)   # [2.5 3.5 4.5]  按列均值
arr.max(axis=1)    # [3 6]          按行最大值

# keepdims 保持维度
arr.sum(axis=0, keepdims=True)   # [[5 7 9]]  shape (1,3)
```

### 7.3 其他聚合

```python
arr = np.array([1, 2, 3, 4, 5])

np.median(arr)        # 中位数
np.percentile(arr, 50)  # 百分位数
np.quantile(arr, 0.5)   # 分位数
np.any(arr > 3)       # True  是否有任一满足
np.all(arr > 0)       # True  是否全部满足
np.unique(arr)        # 去重
```

---

## 8. 排序与搜索

### 8.1 排序

```python
arr = np.array([3, 1, 4, 1, 5, 9, 2, 6])

np.sort(arr)              # 返回排序后的副本 [1 1 2 3 4 5 6 9]
arr.sort()                # 原地排序

# 按轴排序
arr2d = np.array([[3, 1, 2], [6, 5, 4]])
np.sort(arr2d, axis=0)    # 按列排序
np.sort(arr2d, axis=1)    # 按行排序

# 返回排序索引
np.argsort(arr)           # [1 3 6 0 2 4 7 5]

# 按关键字排序（lexsort）
names = np.array(['Bob', 'Alice', 'Charlie'])
scores = np.array([85, 92, 78])
indices = np.lexsort((scores, names))  # 先按 names，再按 scores
names[indices]            # ['Alice' 'Bob' 'Charlie']
```

### 8.2 搜索

```python
arr = np.array([1, 3, 5, 7, 9])

np.where(arr > 4)              # 返回满足条件的索引 (array([2, 3, 4]),)
np.where(arr > 4, arr, -1)     # 条件替换 [−1 −1  5  7  9]

np.searchsorted(arr, [2, 5, 8])  # 二分查找插入位置 [1 2 4]

np.nonzero(arr)                # 非零元素索引
np.extract(arr > 3, arr)       # 提取满足条件的元素
```

---

## 9. 随机数

### 9.1 生成随机数组

```python
# 均匀分布 [0, 1)
np.random.rand(3, 3)

# 标准正态分布
np.random.randn(3, 3)

# 随机整数 [low, high)
np.random.randint(0, 10, size=(2, 3))

# 均匀分布浮点数 [low, high)
np.random.uniform(1.0, 2.0, size=5)

# 正态分布
np.random.normal(loc=0, scale=1, size=(2, 2))

# 二项分布
np.random.binomial(n=10, p=0.5, size=5)

# 泊松分布
np.random.poisson(lam=5, size=5)
```

### 9.2 随机选择与打乱

```python
arr = np.arange(10)

np.random.choice(arr, size=3)         # 随机选3个
np.random.choice(arr, size=3, replace=False)  # 不重复选取

np.random.shuffle(arr)               # 原地打乱
shuffled = np.random.permutation(arr)  # 返回打乱后的副本

np.random.seed(42)                    # 设置随机种子，保证可复现
```

---

## 10. 线性代数

### 10.1 基本运算

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

# 矩阵乘法
A @ B                  # 推荐写法
np.dot(A, B)           # 等价
np.matmul(A, B)        # 等价

# 点积（向量）
v = np.array([1, 2])
w = np.array([3, 4])
np.dot(v, w)           # 11

# 逐元素乘法
A * B                  # 不是矩阵乘法
```

### 10.2 矩阵属性

```python
np.linalg.det(A)       # 行列式
np.linalg.inv(A)       # 逆矩阵
np.linalg.matrix_rank(A)  # 秩
np.trace(A)            # 迹（对角线之和）
np.linalg.norm(A)      # 范数
np.linalg.eig(A)       # 特征值和特征向量
np.linalg.svd(A)       # 奇异值分解
```

### 10.3 解线性方程组

```python
# 解 Ax = b
A = np.array([[1, 1], [1, -1]])
b = np.array([5, 1])
x = np.linalg.solve(A, b)   # [3. 2.]

# 最小二乘
A = np.array([[1, 1], [1, 2], [1, 3]])
b = np.array([1, 2, 2])
x, residuals, rank, s = np.linalg.lstsq(A, b, rcond=None)
```

---

## 11. 文件 I/O

### 11.1 NumPy 原生格式

```python
arr = np.array([[1, 2], [3, 4]])

# 保存单个数组
np.save('data.npy', arr)
arr_loaded = np.load('data.npy')

# 保存多个数组
np.savez('data.npz', a=arr, b=arr*2)
data = np.load('data.npz')
print(data['a'], data['b'])

# 压缩保存
np.savez_compressed('data_compressed.npz', a=arr)
```

### 11.2 文本文件

```python
# 保存为文本
np.savetxt('data.txt', arr, delimiter=',', fmt='%.2f')

# 加载文本
arr = np.loadtxt('data.txt', delimiter=',')
arr = np.genfromtxt('data.txt', delimiter=',', skip_header=1)
```

---

## 12. 常用技巧与注意事项

### 12.1 向量化代替循环

```python
# 低效：循环
result = np.zeros(1000)
for i in range(1000):
    result[i] = i ** 2

# 高效：向量化
result = np.arange(1000) ** 2
```

### 12.2 条件赋值

```python
arr = np.arange(10)

# where 实现条件赋值
arr = np.where(arr > 5, arr * 10, arr)
# [ 0  1  2  3  4  5 60 70 80 90]

# clip 限制范围
arr.clip(2, 8)   # [2 2 2 3 4 5 8 8 8 8]
```

### 12.3 处理 NaN 和 Inf

```python
arr = np.array([1.0, np.nan, 3.0, np.inf, -np.inf])

np.isnan(arr)      # [False  True False False False]
np.isinf(arr)      # [False False False  True  True]
np.isfinite(arr)   # [ True False  True False False]

# 忽略 NaN 的统计
arr2 = np.array([1.0, np.nan, 3.0])
np.nanmean(arr2)   # 2.0
np.nansum(arr2)    # 4.0
np.nanmax(arr2)    # 3.0
```

### 12.4 视图与副本判断

```python
a = np.arange(10)
b = a[2:5]
c = a[[0, 1, 2]]

# 判断是否共享内存
np.shares_memory(a, b)   # True（视图）
np.shares_memory(a, c)   # False（副本）
```

### 12.5 性能优化

```python
# 1. 预分配数组
arr = np.empty(1000)
arr[:] = np.arange(1000)

# 2. 使用 in-place 运算
arr *= 2          # 比 arr = arr * 2 快，不创建新数组
arr += 1

# 3. 选择合适的 dtype
arr = np.array([1, 2, 3], dtype=np.int32)  # 比 int64 省内存

# 4. 避免不必要的复制
a = np.arange(10)
b = a.reshape(2, 5)   # 可能是视图
b = a.reshape(2, 5).copy()  # 确保是副本
```

---

## 附录：常用速查表

| 操作 | 函数/方法 |
|------|-----------|
| 创建数组 | `array`, `arange`, `linspace`, `zeros`, `ones`, `full` |
| 形状 | `reshape`, `flatten`, `ravel`, `transpose`, `T` |
| 拼接 | `concatenate`, `vstack`, `hstack`, `dstack` |
| 分割 | `split`, `vsplit`, `hsplit` |
| 数学 | `sqrt`, `exp`, `log`, `sin`, `cos`, `abs` |
| 统计 | `sum`, `mean`, `std`, `var`, `min`, `max`, `median` |
| 排序 | `sort`, `argsort`, `lexsort` |
| 索引 | `where`, `nonzero`, `searchsorted`, `extract` |
| 线性代数 | `dot`, `inv`, `det`, `eig`, `svd`, `solve` |
| 随机 | `rand`, `randn`, `randint`, `choice`, `shuffle` |
| 广播 | 形状规则：右对齐，维数相等或为1 |

---

> 本汇总涵盖了 NumPy 入门到进阶的核心知识点。建议结合实际数据处理场景多加练习，重点掌握**索引切片**、**广播机制**和**向量化运算**三大核心。
