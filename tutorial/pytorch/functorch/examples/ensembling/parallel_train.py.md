# parallel_train.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `functorch/examples/ensembling/parallel_train.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides runnable examples that demonstrate functorch transforms, compilation flows, and research-style use cases.
- **Purpose (CN)**: 提供可运行示例，展示 functorch 变换、编译流程以及研究型用例。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
import argparse
import math

import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.func import functional_call, grad_and_value, stack_module_state, vmap


# Adapted from http://willwhitney.com/parallel-training-jax.html , which is a
# tutorial on Model Ensembling with JAX by Will Whitney.
#
# The original code comes with the following citation:
# @misc{Whitney2021Parallelizing,
#     author = {William F. Whitney},
#     title = { {Parallelizing neural networks on one GPU with JAX} },
#     year = {2021},
#     url = {http://willwhitney.com/parallel-training-jax.html},
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch.nn, torch.nn.functional, torch.func; standard-library modules such as argparse, math for the logic below.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch.nn、torch.nn.functional、torch.func；标准库模块，如 argparse、math组织在一起，供下方逻辑使用。

### Lines 19-35
```python
# }

# GOAL: Demonstrate that it is possible to use eager-mode vmap
# to parallelize training over models.

parser = argparse.ArgumentParser(description="Functorch Ensembled Models")
parser.add_argument(
    "--device",
    type=str,
    default="cpu",
    help="CPU or GPU ID for this process (default: 'cpu')",
)
args = parser.parse_args()

DEVICE = args.device

# Step 1: Make some spirals
```
- **EN**: Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 38-54
```python
def make_spirals(n_samples, noise_std=0.0, rotations=1.0):
    ts = torch.linspace(0, 1, n_samples, device=DEVICE)
    rs = ts**0.5
    thetas = rs * rotations * 2 * math.pi
    signs = torch.randint(0, 2, (n_samples,), device=DEVICE) * 2 - 1
    labels = (signs > 0).to(torch.long).to(DEVICE)

    xs = (
        rs * signs * torch.cos(thetas)
        + torch.randn(n_samples, device=DEVICE) * noise_std
    )
    ys = (
        rs * signs * torch.sin(thetas)
        + torch.randn(n_samples, device=DEVICE) * noise_std
    )
    points = torch.stack([xs, ys], dim=1)
    return points, labels
```
- **EN**: This chunk defines `make_spirals`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `make_spirals`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 57-68
```python
points, labels = make_spirals(100, noise_std=0.05)


# Step 2: Define two-layer MLP and loss function
class MLPClassifier(nn.Module):
    def __init__(self, hidden_dim=32, n_classes=2):
        super().__init__()
        self.hidden_dim = hidden_dim
        self.n_classes = n_classes

        self.fc1 = nn.Linear(2, self.hidden_dim)
        self.fc2 = nn.Linear(self.hidden_dim, self.n_classes)
```
- **EN**: It introduces or extends MLPClassifier, which hold the primary data model or public surface for this slice of the file. This chunk defines `__init__`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 它引入或扩展了 MLPClassifier，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `__init__`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 70-86
```python
    def forward(self, x):
        x = self.fc1(x)
        x = F.relu(x)
        x = self.fc2(x)
        x = F.log_softmax(x, -1)
        return x


loss_fn = nn.NLLLoss()
model = MLPClassifier().to(DEVICE)


def train_step_fn(weights, batch, targets, lr=0.2):
    def compute_loss(weights, batch, targets):
        output = functional_call(model, weights, batch)
        loss = loss_fn(output, targets)
        return loss
```
- **EN**: This chunk defines `compute_loss`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `compute_loss`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 88-101
```python
    grad_weights, loss = grad_and_value(compute_loss)(weights, batch, targets)

    # NB: PyTorch is missing a "functional optimizer API" (possibly coming soon)
    # so we are going to re-implement SGD here.
    new_weights = {}
    with torch.no_grad():
        for key in grad_weights:
            new_weights[key] = weights[key] - grad_weights[key] * lr

    return loss, new_weights


# Step 4: Let's verify this actually trains.
# We should see the loss decrease.
```
- **EN**: This chunk continues `compute_loss` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `compute_loss`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 102-113
```python
def step4():
    global weights
    for i in range(2000):
        loss, weights = train_step_fn(dict(model.named_parameters()), points, labels)
        if i % 100 == 0:
            print(loss)


step4()

# Step 5: We're ready for multiple models. Let's define an init_fn
# that, given a number of models, returns to us all of the weights.
```
- **EN**: This chunk defines `step4`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `step4`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 116-124
```python
def init_fn(num_models):
    models = [MLPClassifier().to(DEVICE) for _ in range(num_models)]
    params, _ = stack_module_state(models)
    return params


# Step 6: Now, can we try multiple models at the same time?
# The answer is: yes! `loss` is a 2-tuple, and we can see that the value keeps
# on decreasing
```
- **EN**: This chunk defines `init_fn`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `init_fn`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 127-144
```python
def step6():
    parallel_train_step_fn = vmap(train_step_fn, in_dims=(0, None, None))
    batched_weights = init_fn(num_models=2)
    for i in range(2000):
        loss, batched_weights = parallel_train_step_fn(batched_weights, points, labels)
        if i % 200 == 0:
            print(loss)


step6()

# Step 7: Now, the flaw with step 6 is that we were training on the same exact
# data. This can lead to all of the models in the ensemble overfitting in the
# same way. The solution that http://willwhitney.com/parallel-training-jax.html
# applies is to randomly subset the data in a way that the models do not receive
# exactly the same data in each training step!
# Because the goal of this doc is to show that we can use eager-mode vmap to
# achieve similar things as JAX, the rest of this is left as an exercise to the reader.
```
- **EN**: This chunk defines `step6`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `step6`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 146-151
```python
# In conclusion, to achieve what http://willwhitney.com/parallel-training-jax.html
# does, we used the following additional items that PyTorch does not have:
# 1. NN module functional API that turns a module into a (state, state_less_fn) pair
# 2. Functional optimizers
# 3. A "functional" grad API (that effectively wraps autograd.grad)
# 4. Composability between the functional grad API and torch.vmap.
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
- **Model training**
  - EN: Runs data-driven training loops to fit heuristic or regression models.
  - CN: 运行数据驱动的训练循环，以拟合启发式或回归模型。
- **Gradient transforms**
  - EN: Applies gradient-oriented transforms such as grad, vjp, or higher-order derivatives.
  - CN: 应用梯度相关变换，例如 grad、vjp 或高阶导数。
- **DEVICE**
  - EN: `DEVICE` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `DEVICE` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch.nn`, `torch.nn.functional`, `torch.func`
- **Standard library / 标准库**: `argparse`, `math`
- **Primary symbols / 核心符号**: `DEVICE`, `make_spirals`, `MLPClassifier`, `__init__`, `forward`, `train_step_fn`, `compute_loss`, `step4`, `init_fn`, `step6`
