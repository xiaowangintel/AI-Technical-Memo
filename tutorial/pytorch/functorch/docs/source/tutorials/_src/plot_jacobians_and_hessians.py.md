# plot_jacobians_and_hessians.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `functorch/docs/source/tutorials/_src/plot_jacobians_and_hessians.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Configures documentation builds or tutorial scripts that explain functorch concepts to users.
- **Purpose (CN)**: 配置文档构建或教程脚本，用于向用户解释 functorch 概念。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```python
"""
=============================
Jacobians, hessians, and more
=============================

Computing jacobians or hessians are useful in a number of non-traditional
deep learning models. It is difficult (or annoying) to compute these quantities
efficiently using a standard autodiff system like PyTorch Autograd; functorch
provides ways of computing various higher-order autodiff quantities efficiently.
"""

from functools import partial

import torch
import torch.nn.functional as F
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch.nn.functional; standard-library modules such as functools for the logic below.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch.nn.functional；标准库模块，如 functools组织在一起，供下方逻辑使用。

### Lines 18-34
```python
torch.manual_seed(0)


######################################################################
# Setup: Comparing functorch vs the naive approach
# --------------------------------------------------------------------
# Let's start with a function that we'd like to compute the jacobian of.
# This is a simple linear function with non-linear activation.
def predict(weight, bias, x):
    return F.linear(x, weight, bias).tanh()


# Here's some dummy data: a weight, a bias, and a feature vector.
D = 16
weight = torch.randn(D, D)
bias = torch.randn(D)
x = torch.randn(D)
```
- **EN**: This chunk defines `predict`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `predict`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 36-52
```python
# Let's think of ``predict`` as a function that maps the input ``x`` from R^D -> R^D.
# PyTorch Autograd computes vector-Jacobian products. In order to compute the full
# Jacobian of this R^D -> R^D function, we would have to compute it row-by-row
# by using a different unit vector each time.
xp = x.clone().requires_grad_()
unit_vectors = torch.eye(D)


def compute_jac(xp):
    jacobian_rows = [
        torch.autograd.grad(predict(weight, bias, xp), xp, vec)[0]
        for vec in unit_vectors
    ]
    return torch.stack(jacobian_rows)


jacobian = compute_jac(xp)
```
- **EN**: This chunk defines `compute_jac`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `compute_jac`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 54-71
```python
# Instead of computing the jacobian row-by-row, we can use ``vmap`` to get rid
# of the for-loop and vectorize the computation. We can't directly apply vmap
# to PyTorch Autograd; instead, functorch provides a ``vjp`` transform:
from functorch import vjp, vmap


_, vjp_fn = vjp(partial(predict, weight, bias), x)
(ft_jacobian,) = vmap(vjp_fn)(unit_vectors)
if not torch.allclose(ft_jacobian, jacobian):
    raise AssertionError("ft_jacobian does not match jacobian")

# In another tutorial a composition of reverse-mode AD and vmap gave us
# per-sample-gradients. In this tutorial, composing reverse-mode AD and vmap
# gives us Jacobian computation! Various compositions of vmap and autodiff
# transforms can give us different interesting quantities.
#
# functorch provides ``jacrev`` as a convenience function that performs
# the vmap-vjp composition to compute jacobians. ``jacrev`` accepts an argnums
```
- **EN**: The import section wires together PyTorch-local modules such as functorch for the logic below. This chunk continues `compute_jac` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Branch dispatch chooses specialized behavior for different cases, node kinds, or runtime situations. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把PyTorch 本地模块，如 functorch组织在一起，供下方逻辑使用。 这一段延续了 `compute_jac`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 分支分发会为不同情况、节点类型或运行时场景选择专门行为。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 72-85
```python
# argument that says which argument we would like to compute Jacobians with
# respect to.
from functorch import jacrev


ft_jacobian = jacrev(predict, argnums=2)(weight, bias, x)
if not torch.allclose(ft_jacobian, jacobian):
    raise AssertionError("ft_jacobian does not match jacobian")

# Let's compare the performance of the two ways to compute jacobian.
# The functorch version is much faster (and becomes even faster the more outputs
# there are). In general, we expect that vectorization via ``vmap`` can help
# eliminate overhead and give better utilization of your hardware.
from torch.utils.benchmark import Timer
```
- **EN**: The import section wires together PyTorch-local modules such as functorch, torch.utils.benchmark for the logic below. This chunk continues `compute_jac` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Branch dispatch chooses specialized behavior for different cases, node kinds, or runtime situations. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把PyTorch 本地模块，如 functorch、torch.utils.benchmark组织在一起，供下方逻辑使用。 这一段延续了 `compute_jac`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 分支分发会为不同情况、节点类型或运行时场景选择专门行为。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 88-105
```python
without_vmap = Timer(stmt="compute_jac(xp)", globals=globals())
with_vmap = Timer(stmt="jacrev(predict, argnums=2)(weight, bias, x)", globals=globals())
print(without_vmap.timeit(500))
print(with_vmap.timeit(500))

# It's pretty easy to flip the problem around and say we want to compute
# Jacobians of the parameters to our model (weight, bias) instead of the input.
ft_jac_weight, ft_jac_bias = jacrev(predict, argnums=(0, 1))(weight, bias, x)

######################################################################
# reverse-mode Jacobian (jacrev) vs forward-mode Jacobian (jacfwd)
# --------------------------------------------------------------------
# We offer two APIs to compute jacobians: jacrev and jacfwd:
# - jacrev uses reverse-mode AD. As you saw above it is a composition of our
#   vjp and vmap transforms.
# - jacfwd uses forward-mode AD. It is implemented as a composition of our
#   jvp and vmap transforms.
# jacfwd and jacrev can be substituted for each other and have different
```
- **EN**: This chunk continues `compute_jac` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `compute_jac`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 106-116
```python
# performance characteristics.
#
# As a general rule of thumb, if you're computing the jacobian of an R^N -> R^M
# function, if there are many more outputs than inputs (i.e. M > N) then jacfwd is
# preferred, otherwise use jacrev. There are exceptions to this rule, but a
# non-rigorous argument for this follows:

# In reverse-mode AD, we are computing the jacobian row-by-row, while in
# forward-mode AD (which computes Jacobian-vector products), we are computing
# it column-by-column. The Jacobian matrix has M rows and N columns.
from functorch import jacfwd, jacrev
```
- **EN**: The import section wires together PyTorch-local modules such as functorch for the logic below. This chunk continues `compute_jac` and expands its control flow, data preparation, or emitted structure.
- **CN**: 导入区把PyTorch 本地模块，如 functorch组织在一起，供下方逻辑使用。 这一段延续了 `compute_jac`，继续展开其控制流、数据准备或生成结构。

### Lines 119-129
```python
# Benchmark with more inputs than outputs
Din = 32
Dout = 2048
weight = torch.randn(Dout, Din)
bias = torch.randn(Dout)
x = torch.randn(Din)

using_fwd = Timer(stmt="jacfwd(predict, argnums=2)(weight, bias, x)", globals=globals())
using_bwd = Timer(stmt="jacrev(predict, argnums=2)(weight, bias, x)", globals=globals())
print(f"jacfwd time: {using_fwd.timeit(500)}")
print(f"jacrev time: {using_bwd.timeit(500)}")
```
- **EN**: This chunk continues `compute_jac` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `compute_jac`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 131-141
```python
# Benchmark with more outputs than inputs
Din = 2048
Dout = 32
weight = torch.randn(Dout, Din)
bias = torch.randn(Dout)
x = torch.randn(Din)

using_fwd = Timer(stmt="jacfwd(predict, argnums=2)(weight, bias, x)", globals=globals())
using_bwd = Timer(stmt="jacrev(predict, argnums=2)(weight, bias, x)", globals=globals())
print(f"jacfwd time: {using_fwd.timeit(500)}")
print(f"jacrev time: {using_bwd.timeit(500)}")
```
- **EN**: This chunk continues `compute_jac` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `compute_jac`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 143-159
```python
######################################################################
# Hessian computation with functorch.hessian
# --------------------------------------------------------------------
# We offer a convenience API to compute hessians: functorch.hessian.
# Hessians are the jacobian of the jacobian, which suggests that one can just
# compose functorch's jacobian transforms to compute one.
# Indeed, under the hood, ``hessian(f)`` is simply ``jacfwd(jacrev(f))``
#
# Depending on your model, you may want to use ``jacfwd(jacfwd(f))`` or
# ``jacrev(jacrev(f))`` instead to compute hessians.
from functorch import hessian


# # TODO: make sure PyTorch has tanh_backward implemented for jvp!!
# hess0 = hessian(predict, argnums=2)(weight, bias, x)
# hess1 = jacfwd(jacfwd(predict, argnums=2), argnums=2)(weight, bias, x)
hess2 = jacrev(jacrev(predict, argnums=2), argnums=2)(weight, bias, x)
```
- **EN**: The import section wires together PyTorch-local modules such as functorch for the logic below. This chunk continues `compute_jac` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把PyTorch 本地模块，如 functorch组织在一起，供下方逻辑使用。 这一段延续了 `compute_jac`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 161-174
```python
######################################################################
# Batch Jacobian (and Batch Hessian)
# --------------------------------------------------------------------
# In the above examples we've been operating with a single feature vector.
# In some cases you might want to take the Jacobian of a batch of outputs
# with respect to a batch of inputs where each input produces an independent
# output. That is, given a batch of inputs of shape (B, N) and a function
# that goes from (B, N) -> (B, M), we would like a Jacobian of shape (B, M, N).
# The easiest way to do this is to sum over the batch dimension and then
# compute the Jacobian of that function:


def predict_with_output_summed(weight, bias, x):
    return predict(weight, bias, x).sum(0)
```
- **EN**: This chunk defines `predict_with_output_summed`, which implements one step in a functional transform, example, or package export flow. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `predict_with_output_summed`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 177-192
```python
batch_size = 64
Din = 31
Dout = 33
weight = torch.randn(Dout, Din)
bias = torch.randn(Dout)
x = torch.randn(batch_size, Din)

batch_jacobian0 = jacrev(predict_with_output_summed, argnums=2)(weight, bias, x)

# If you instead have a function that goes from R^N -> R^M but inputs that are
# batched, you compose vmap with jacrev to compute batched jacobians:

compute_batch_jacobian = vmap(jacrev(predict, argnums=2), in_dims=(None, None, 0))
batch_jacobian1 = compute_batch_jacobian(weight, bias, x)
if not torch.allclose(batch_jacobian0, batch_jacobian1):
    raise AssertionError("batch_jacobian0 does not match batch_jacobian1")
```
- **EN**: This chunk continues `predict_with_output_summed` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Branch dispatch chooses specialized behavior for different cases, node kinds, or runtime situations. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `predict_with_output_summed`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 分支分发会为不同情况、节点类型或运行时场景选择专门行为。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 194-198
```python
# Finally, batch hessians can be computed similarly. It's easiest to think about
# them by using vmap to batch over hessian computation, but in some cases the sum
# trick also works.
compute_batch_hessian = vmap(hessian(predict, argnums=2), in_dims=(None, None, 0))
batch_hess = compute_batch_hessian(weight, bias, x)
```
- **EN**: This chunk continues `predict_with_output_summed` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `predict_with_output_summed`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

## Key Concepts / 关键概念

- **Functional tensor transforms**
  - EN: Applies composable transforms such as batching, gradients, and tracing.
  - CN: 应用可组合的变换，例如 batching、梯度和 tracing。
- **Autograd generation**
  - EN: Produces or organizes derivative-related wrappers, bindings, and metadata.
  - CN: 生成或组织与求导相关的包装器、绑定与元数据。
- **Batching / vmap**
  - EN: Coordinates vectorized mapping rules and batched operator behavior.
  - CN: 协调向量化映射规则与 batched 算子行为。
- **Benchmarking**
  - EN: Measures runtime behavior to compare implementations or generate tuning data.
  - CN: 测量运行时行为，用于比较实现或生成调优数据。
- **Gradient transforms**
  - EN: Applies gradient-oriented transforms such as grad, vjp, or higher-order derivatives.
  - CN: 应用梯度相关变换，例如 grad、vjp 或高阶导数。
- **Jacobian/Hessian transforms**
  - EN: Works with Jacobian- or Hessian-style higher-order differentiation APIs.
  - CN: 处理 Jacobian/Hessian 风格的高阶微分 API。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch.nn.functional`, `functorch`, `torch.utils.benchmark`
- **Standard library / 标准库**: `functools`
- **Primary symbols / 核心符号**: `predict`, `compute_jac`, `predict_with_output_summed`
