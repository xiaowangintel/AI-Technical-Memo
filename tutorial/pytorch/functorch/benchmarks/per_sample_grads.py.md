# per_sample_grads.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `functorch/benchmarks/per_sample_grads.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements benchmark drivers and result-processing utilities for functorch performance experiments.
- **Purpose (CN)**: 实现 functorch 性能实验所需的基准驱动与结果处理工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```python
import time

from opacus import PrivacyEngine
from opacus.utils.module_modification import convert_batchnorm_modules
from torchvision import models

import torch
import torch.nn as nn
from functorch import grad, make_functional, vmap
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch.nn, functorch; third-party modules such as opacus, opacus.utils.module_modification, torchvision; standard-library modules such as time for the logic below.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch.nn、functorch；第三方模块，如 opacus、opacus.utils.module_modification、torchvision；标准库模块，如 time组织在一起，供下方逻辑使用。

### Lines 12-22
```python
device = "cuda"
batch_size = 128
torch.manual_seed(0)

model_functorch = convert_batchnorm_modules(models.resnet18(num_classes=10))
model_functorch = model_functorch.to(device)
criterion = nn.CrossEntropyLoss()

images = torch.randn(batch_size, 3, 32, 32, device=device)
targets = torch.randint(0, 10, (batch_size,), device=device)
func_model, weights = make_functional(model_functorch)
```
- **EN**: Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 25-35
```python
def compute_loss(weights, image, target):
    images = image.unsqueeze(0)
    targets = target.unsqueeze(0)
    output = func_model(weights, images)
    loss = criterion(output, targets)
    return loss


def functorch_per_sample_grad():
    compute_grad = grad(compute_loss)
    compute_per_sample_grad = vmap(compute_grad, (None, 0, 0))
```
- **EN**: This chunk defines `functorch_per_sample_grad`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `functorch_per_sample_grad`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 37-42
```python
    start = time.time()
    result = compute_per_sample_grad(weights, images, targets)
    torch.cuda.synchronize()
    end = time.time()

    return result, end - start  # end - start in seconds
```
- **EN**: This chunk continues `functorch_per_sample_grad` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `functorch_per_sample_grad`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 45-51
```python
torch.manual_seed(0)
model_opacus = convert_batchnorm_modules(models.resnet18(num_classes=10))
model_opacus = model_opacus.to(device)
criterion = nn.CrossEntropyLoss()
for p_f, p_o in zip(model_functorch.parameters(), model_opacus.parameters()):
    if not torch.allclose(p_f, p_o):  # Sanity check
        raise AssertionError("Parameters do not match")
```
- **EN**: This chunk continues `functorch_per_sample_grad` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Branch dispatch chooses specialized behavior for different cases, node kinds, or runtime situations. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `functorch_per_sample_grad`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 分支分发会为不同情况、节点类型或运行时场景选择专门行为。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 53-59
```python
privacy_engine = PrivacyEngine(
    model_opacus,
    sample_rate=0.01,
    alphas=[10, 100],
    noise_multiplier=1,
    max_grad_norm=10000.0,
)
```
- **EN**: This chunk continues `functorch_per_sample_grad` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `functorch_per_sample_grad`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 62-73
```python
def opacus_per_sample_grad():
    start = time.time()
    output = model_opacus(images)
    loss = criterion(output, targets)
    loss.backward()
    torch.cuda.synchronize()
    end = time.time()
    expected = [p.grad_sample for p in model_opacus.parameters()]
    for p in model_opacus.parameters():
        delattr(p, "grad_sample")
        p.grad = None
    return expected, end - start
```
- **EN**: This chunk defines `opacus_per_sample_grad`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `opacus_per_sample_grad`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 76-85
```python
for _ in range(5):
    _, seconds = functorch_per_sample_grad()
    print(seconds)

result, seconds = functorch_per_sample_grad()
print(seconds)

for _ in range(5):
    _, seconds = opacus_per_sample_grad()
    print(seconds)
```
- **EN**: This chunk continues `opacus_per_sample_grad` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `opacus_per_sample_grad`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 87-98
```python
expected, seconds = opacus_per_sample_grad()
print(seconds)

result = [r.detach() for r in result]
print(len(result))

# TODO: The following shows that the per-sample-grads computed are different.
# This concerns me a little; we should compare to a source of truth.
# for i, (r, e) in enumerate(list(zip(result, expected))[::-1]):
#     if torch.allclose(r, e, rtol=1e-5):
#         continue
#     print(-(i+1), ((r - e)/(e + 0.000001)).abs().max())
```
- **EN**: This chunk continues `opacus_per_sample_grad` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `opacus_per_sample_grad`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

## Key Concepts / 关键概念

- **Functional tensor transforms**
  - EN: Applies composable transforms such as batching, gradients, and tracing.
  - CN: 应用可组合的变换，例如 batching、梯度和 tracing。
- **Batching / vmap**
  - EN: Coordinates vectorized mapping rules and batched operator behavior.
  - CN: 协调向量化映射规则与 batched 算子行为。
- **Benchmarking**
  - EN: Measures runtime behavior to compare implementations or generate tuning data.
  - CN: 测量运行时行为，用于比较实现或生成调优数据。
- **Gradient transforms**
  - EN: Applies gradient-oriented transforms such as grad, vjp, or higher-order derivatives.
  - CN: 应用梯度相关变换，例如 grad、vjp 或高阶导数。
- **compute_loss**
  - EN: `compute_loss` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `compute_loss` 是本文件声明、导出或驱动的显著符号之一。
- **functorch_per_sample_grad**
  - EN: `functorch_per_sample_grad` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `functorch_per_sample_grad` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch.nn`, `functorch`
- **Third-party modules / 第三方模块**: `opacus`, `opacus.utils.module_modification`, `torchvision`
- **Standard library / 标准库**: `time`
- **Primary symbols / 核心符号**: `compute_loss`, `functorch_per_sample_grad`, `opacus_per_sample_grad`
