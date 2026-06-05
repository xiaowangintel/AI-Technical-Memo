# plot_per_sample_gradients.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `functorch/docs/source/tutorials/_src/plot_per_sample_gradients.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Configures documentation builds or tutorial scripts that explain functorch concepts to users.
- **Purpose (CN)**: 配置文档构建或教程脚本，用于向用户解释 functorch 概念。
## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```python
"""
==========================
Per-sample-gradients
==========================

What is it?
--------------------------------------------------------------------
Per-sample-gradient computation is computing the gradient for each and every
sample in a batch of data. It is a useful quantity in differential privacy
and optimization research.
"""
```
- **EN**: Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries.
- **CN**: 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。

### Lines 13-22
```python
import torch
import torch.nn as nn
import torch.nn.functional as F


torch.manual_seed(0)


# Here's a simple CNN
class SimpleCNN(nn.Module):
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch.nn, torch.nn.functional for the logic below. It introduces or extends SimpleCNN, which hold the primary data model or public surface for this slice of the file. This chunk continues `SimpleCNN` and expands its control flow, data preparation, or emitted structure.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch.nn、torch.nn.functional组织在一起，供下方逻辑使用。 它引入或扩展了 SimpleCNN，这些类型承载了本段的核心数据模型或对外接口。 这一段延续了 `SimpleCNN`，继续展开其控制流、数据准备或生成结构。

### Lines 23-28
```python
    def __init__(self):
        super().__init__()
        self.conv1 = nn.Conv2d(1, 32, 3, 1)
        self.conv2 = nn.Conv2d(32, 64, 3, 1)
        self.fc1 = nn.Linear(9216, 128)
        self.fc2 = nn.Linear(128, 10)
```
- **EN**: This chunk defines `__init__`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `__init__`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 30-41
```python
    def forward(self, x):
        x = self.conv1(x)
        x = F.relu(x)
        x = self.conv2(x)
        x = F.relu(x)
        x = F.max_pool2d(x, 2)
        x = torch.flatten(x, 1)
        x = self.fc1(x)
        x = F.relu(x)
        x = self.fc2(x)
        output = F.log_softmax(x, dim=1)
        output = x
```
- **EN**: This chunk defines `forward`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `forward`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 42-53
```python
        return output


def loss_fn(predictions, targets):
    return F.nll_loss(predictions, targets)


# Let's generate a batch of dummy data. Pretend that we're working with an
# MNIST dataset where the images are 28 by 28 and we have a minibatch of size 64.
device = "cuda"
num_models = 10
batch_size = 64
```
- **EN**: This chunk defines `loss_fn`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `loss_fn`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 54-63
```python
data = torch.randn(batch_size, 1, 28, 28, device=device)
targets = torch.randint(10, (64,), device=device)

# In regular model training, one would forward the batch of examples and then
# call .backward() to compute gradients:

model = SimpleCNN().to(device=device)
predictions = model(data)
loss = loss_fn(predictions, targets)
loss.backward()
```
- **EN**: This chunk continues `loss_fn` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `loss_fn`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 66-73
```python
# Conceptually, per-sample-gradient computation is equivalent to: for each sample
# of the data, perform a forward and a backward pass to get a gradient.
def compute_grad(sample, target):
    sample = sample.unsqueeze(0)
    target = target.unsqueeze(0)
    prediction = model(sample)
    loss = loss_fn(prediction, target)
    return torch.autograd.grad(loss, list(model.parameters()))
```
- **EN**: This chunk defines `compute_grad`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `compute_grad`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 76-83
```python
def compute_sample_grads(data, targets):
    sample_grads = [compute_grad(data[i], targets[i]) for i in range(batch_size)]
    sample_grads = zip(*sample_grads)
    sample_grads = [torch.stack(shards) for shards in sample_grads]
    return sample_grads


per_sample_grads = compute_sample_grads(data, targets)
```
- **EN**: This chunk defines `compute_sample_grads`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `compute_sample_grads`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 85-96
```python
# sample_grads[0] is the per-sample-grad for model.conv1.weight
# model.conv1.weight.shape is [32, 1, 3, 3]; notice how there is one gradient
# per sample in the batch for a total of 64.
print(per_sample_grads[0].shape)


######################################################################
# Per-sample-grads using functorch
# --------------------------------------------------------------------
# We can compute per-sample-gradients efficiently by using function transforms.
# First, let's create a stateless functional version of ``model`` by using
# ``functorch.make_functional_with_buffers``.
```
- **EN**: This chunk continues `compute_sample_grads` and expands its control flow, data preparation, or emitted structure.
- **CN**: 这一段延续了 `compute_sample_grads`，继续展开其控制流、数据准备或生成结构。

### Lines 97-107
```python
from functorch import grad, make_functional_with_buffers, vmap


fmodel, params, buffers = make_functional_with_buffers(model)


# Next, let's define a function to compute the loss of the model given a single
# input rather than a batch of inputs. It is important that this function accepts the
# parameters, the input, and the target, because we will be transforming over them.
# Because the model was originally written to handle batches, we'll use
# ``torch.unsqueeze`` to add a batch dimension.
```
- **EN**: The import section wires together PyTorch-local modules such as functorch for the logic below. This chunk continues `compute_sample_grads` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把PyTorch 本地模块，如 functorch组织在一起，供下方逻辑使用。 这一段延续了 `compute_sample_grads`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 108-118
```python
def compute_loss(params, buffers, sample, target):
    batch = sample.unsqueeze(0)
    targets = target.unsqueeze(0)
    predictions = fmodel(params, buffers, batch)
    loss = loss_fn(predictions, targets)
    return loss


# Now, let's use ``grad`` to create a new function that computes the gradient
# with respect to the first argument of compute_loss (i.e. the params).
ft_compute_grad = grad(compute_loss)
```
- **EN**: This chunk defines `compute_loss`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `compute_loss`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 120-131
```python
# ``ft_compute_grad`` computes the gradient for a single (sample, target) pair.
# We can use ``vmap`` to get it to compute the gradient over an entire batch
# of samples and targets. Note that in_dims=(None, None, 0, 0) because we wish
# to map ``ft_compute_grad`` over the 0th dimension of the data and targets
# and use the same params and buffers for each.
ft_compute_sample_grad = vmap(ft_compute_grad, in_dims=(None, None, 0, 0))

# Finally, let's used our transformed function to compute per-sample-gradients:
ft_per_sample_grads = ft_compute_sample_grad(params, buffers, data, targets)
for per_sample_grad, ft_per_sample_grad in zip(per_sample_grads, ft_per_sample_grads):
    if not torch.allclose(per_sample_grad, ft_per_sample_grad, atol=1e-6, rtol=1e-6):
        raise AssertionError("per_sample_grad does not match ft_per_sample_grad")
```
- **EN**: This chunk continues `compute_loss` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Branch dispatch chooses specialized behavior for different cases, node kinds, or runtime situations. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `compute_loss`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 分支分发会为不同情况、节点类型或运行时场景选择专门行为。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 133-138
```python
# A quick note: there are limitations around what types of functions can be
# transformed by vmap. The best functions to transform are ones that are
# pure functions: a function where the outputs are only determined by the inputs
# that have no side effects (e.g. mutation). vmap is unable to handle mutation of
# arbitrary Python data structures, but it is able to handle many in-place
# PyTorch operations.
```
- **EN**: This comment block records assumptions, caveats, or usage guidance that frame the implementation below.
- **CN**: 这一段注释记录了后续实现所依赖的假设、注意事项或使用说明。

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
- **Gradient transforms**
  - EN: Applies gradient-oriented transforms such as grad, vjp, or higher-order derivatives.
  - CN: 应用梯度相关变换，例如 grad、vjp 或高阶导数。
- **SimpleCNN**
  - EN: `SimpleCNN` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `SimpleCNN` 是本文件声明、导出或驱动的显著符号之一。
- **__init__**
  - EN: `__init__` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `__init__` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch.nn`, `torch.nn.functional`, `functorch`
- **Primary symbols / 核心符号**: `SimpleCNN`, `__init__`, `forward`, `loss_fn`, `compute_grad`, `compute_sample_grads`, `compute_loss`
