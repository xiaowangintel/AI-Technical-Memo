# lennard_jones.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `functorch/examples/lennard_jones/lennard_jones.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides runnable examples that demonstrate functorch transforms, compilation flows, and research-style use cases.
- **Purpose (CN)**: 提供可运行示例，展示 functorch 变换、编译流程以及研究型用例。
## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```python
# This example was adapted from https://github.com/muhrin/milad
# It is licensed under the GLPv3 license. You can find a copy of it
# here: https://www.gnu.org/licenses/gpl-3.0.en.html .

import torch
from torch import nn
from torch.func import jacrev, vmap
from torch.nn.functional import mse_loss
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch.func, torch.nn.functional for the logic below.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch.func、torch.nn.functional组织在一起，供下方逻辑使用。

### Lines 11-21
```python
sigma = 0.5
epsilon = 4.0


def lennard_jones(r):
    return epsilon * ((sigma / r) ** 12 - (sigma / r) ** 6)


def lennard_jones_force(r):
    """Get magnitude of LJ force"""
    return -epsilon * ((-12 * sigma**12 / r**13) + (6 * sigma**6 / r**7))
```
- **EN**: This chunk defines `lennard_jones_force`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `lennard_jones_force`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 24-35
```python
training_size = 1000
r = torch.linspace(0.5, 2 * sigma, steps=training_size, requires_grad=True)

# Create a bunch of vectors that point along positive-x
drs = torch.outer(r, torch.tensor([1.0, 0, 0]))
norms = torch.norm(drs, dim=1).reshape(-1, 1)
# Create training energies
training_energies = torch.stack(list(map(lennard_jones, norms))).reshape(-1, 1)
# Create forces with random direction vectors
training_forces = torch.stack(
    [force * dr for force, dr in zip(map(lennard_jones_force, norms), drs)]
)
```
- **EN**: This chunk continues `lennard_jones_force` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `lennard_jones_force`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 37-47
```python
model = nn.Sequential(
    nn.Linear(1, 16),
    nn.Tanh(),
    nn.Linear(16, 16),
    nn.Tanh(),
    nn.Linear(16, 16),
    nn.Tanh(),
    nn.Linear(16, 16),
    nn.Tanh(),
    nn.Linear(16, 1),
)
```
- **EN**: This chunk continues `lennard_jones_force` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `lennard_jones_force`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 50-56
```python
def make_prediction(model, drs):
    norms = torch.norm(drs, dim=1).reshape(-1, 1)
    energies = model(norms)

    network_derivs = vmap(jacrev(model))(norms).squeeze(-1)
    forces = -network_derivs * drs / norms
    return energies, forces
```
- **EN**: This chunk defines `make_prediction`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `make_prediction`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 59-66
```python
def loss_fn(energies, forces, predicted_energies, predicted_forces):
    return (
        mse_loss(energies, predicted_energies)
        + 0.01 * mse_loss(forces, predicted_forces) / 3
    )


optimiser = torch.optim.Adam(model.parameters(), lr=1e-3)
```
- **EN**: This chunk defines `loss_fn`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `loss_fn`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 68-76
```python
for epoch in range(400):
    optimiser.zero_grad()
    energies, forces = make_prediction(model, drs)
    loss = loss_fn(training_energies, training_forces, energies, forces)
    loss.backward(retain_graph=True)
    optimiser.step()

    if epoch % 20 == 0:
        print(loss.cpu().item())
```
- **EN**: This chunk continues `loss_fn` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `loss_fn`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

## Key Concepts / 关键概念

- **Functional tensor transforms**
  - EN: Applies composable transforms such as batching, gradients, and tracing.
  - CN: 应用可组合的变换，例如 batching、梯度和 tracing。
- **Batching / vmap**
  - EN: Coordinates vectorized mapping rules and batched operator behavior.
  - CN: 协调向量化映射规则与 batched 算子行为。
- **Gradient transforms**
  - EN: Applies gradient-oriented transforms such as grad, vjp, or higher-order derivatives.
  - CN: 应用梯度相关变换，例如 grad、vjp 或高阶导数。
- **Jacobian/Hessian transforms**
  - EN: Works with Jacobian- or Hessian-style higher-order differentiation APIs.
  - CN: 处理 Jacobian/Hessian 风格的高阶微分 API。
- **lennard_jones**
  - EN: `lennard_jones` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `lennard_jones` 是本文件声明、导出或驱动的显著符号之一。
- **lennard_jones_force**
  - EN: `lennard_jones_force` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `lennard_jones_force` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch.func`, `torch.nn.functional`
- **Primary symbols / 核心符号**: `lennard_jones`, `lennard_jones_force`, `make_prediction`, `loss_fn`
