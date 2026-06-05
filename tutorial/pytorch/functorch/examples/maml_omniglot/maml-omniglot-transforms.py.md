# maml-omniglot-transforms.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `functorch/examples/maml_omniglot/maml-omniglot-transforms.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides runnable examples that demonstrate functorch transforms, compilation flows, and research-style use cases.
- **Purpose (CN)**: 提供可运行示例，展示 functorch 变换、编译流程以及研究型用例。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```python
#!/usr/bin/env python3
#
# Copyright (c) Facebook, Inc. and its affiliates.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
```
- **EN**: This comment block records assumptions, caveats, or usage guidance that frame the implementation below.
- **CN**: 这一段注释记录了后续实现所依赖的假设、注意事项或使用说明。

### Lines 17-32
```python
"""
This example shows how to use higher to do Model Agnostic Meta Learning (MAML)
for few-shot Omniglot classification.
For more details see the original MAML paper:
https://arxiv.org/abs/1703.03400

This code has been modified from Jackie Loong's PyTorch MAML implementation:
https://github.com/dragen1860/MAML-Pytorch/blob/master/omniglot_train.py

Our MAML++ fork and experiments are available at:
https://github.com/bamos/HowToTrainYourMAMLPytorch
"""

import argparse
import functools
import time
```
- **EN**: The import section wires together standard-library modules such as argparse, functools, time for the logic below. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries.
- **CN**: 导入区把标准库模块，如 argparse、functools、time组织在一起，供下方逻辑使用。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。

### Lines 34-48
```python
import matplotlib as mpl
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
from support.omniglot_loaders import OmniglotNShot

import torch
import torch.nn.functional as F
import torch.optim as optim
from torch import nn
from torch.func import functional_call, grad, vmap


mpl.use("Agg")
plt.style.use("bmh")
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch.nn.functional, torch.optim, torch.func; third-party modules such as matplotlib, matplotlib.pyplot, numpy, and 2 more for the logic below.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch.nn.functional、torch.optim、torch.func；第三方模块，如 matplotlib、matplotlib.pyplot、numpy 等共 5 项组织在一起，供下方逻辑使用。

### Lines 51-68
```python
def main():
    argparser = argparse.ArgumentParser()
    argparser.add_argument("--n-way", "--n_way", type=int, help="n way", default=5)
    argparser.add_argument(
        "--k-spt", "--k_spt", type=int, help="k shot for support set", default=5
    )
    argparser.add_argument(
        "--k-qry", "--k_qry", type=int, help="k shot for query set", default=15
    )
    argparser.add_argument("--device", type=str, help="device", default="cuda")
    argparser.add_argument(
        "--task-num",
        "--task_num",
        type=int,
        help="meta batch size, namely task num",
        default=32,
    )
    argparser.add_argument("--seed", type=int, help="random seed", default=1)
```
- **EN**: This chunk defines `main`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `main`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 69-86
```python
    args = argparser.parse_args()

    torch.manual_seed(args.seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(args.seed)
    np.random.seed(args.seed)

    # Set up the Omniglot loader.
    device = args.device
    db = OmniglotNShot(
        "/tmp/omniglot-data",
        batchsz=args.task_num,
        n_way=args.n_way,
        k_shot=args.k_spt,
        k_query=args.k_qry,
        imgsz=28,
        device=device,
    )
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 88-105
```python
    # Create a vanilla PyTorch neural network.
    inplace_relu = True
    net = nn.Sequential(
        nn.Conv2d(1, 64, 3),
        nn.BatchNorm2d(64, affine=True, track_running_stats=False),
        nn.ReLU(inplace=inplace_relu),
        nn.MaxPool2d(2, 2),
        nn.Conv2d(64, 64, 3),
        nn.BatchNorm2d(64, affine=True, track_running_stats=False),
        nn.ReLU(inplace=inplace_relu),
        nn.MaxPool2d(2, 2),
        nn.Conv2d(64, 64, 3),
        nn.BatchNorm2d(64, affine=True, track_running_stats=False),
        nn.ReLU(inplace=inplace_relu),
        nn.MaxPool2d(2, 2),
        nn.Flatten(),
        nn.Linear(64, args.n_way),
    ).to(device)
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 107-121
```python
    net.train()

    # We will use Adam to (meta-)optimize the initial parameters
    # to be adapted.
    meta_opt = optim.Adam(net.parameters(), lr=1e-3)

    log = []
    for epoch in range(100):
        train(db, net, device, meta_opt, epoch, log)
        test(db, net, device, epoch, log)
        plot(log)


# Trains a model for n_inner_iter using the support and returns a loss
# using the query.
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 122-135
```python
def loss_for_task(net, n_inner_iter, x_spt, y_spt, x_qry, y_qry):
    params = dict(net.named_parameters())
    buffers = dict(net.named_buffers())
    querysz = x_qry.size(0)

    def compute_loss(new_params, buffers, x, y):
        logits = functional_call(net, (new_params, buffers), x)
        loss = F.cross_entropy(logits, y)
        return loss

    new_params = params
    for _ in range(n_inner_iter):
        grads = grad(compute_loss)(new_params, buffers, x_spt, y_spt)
        new_params = {k: new_params[k] - g * 1e-1 for k, g in grads.items()}
```
- **EN**: This chunk defines `compute_loss`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `compute_loss`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 137-153
```python
    # The final set of adapted parameters will induce some
    # final loss and accuracy on the query dataset.
    # These will be used to update the model's meta-parameters.
    qry_logits = functional_call(net, (new_params, buffers), x_qry)
    qry_loss = F.cross_entropy(qry_logits, y_qry)
    qry_acc = (qry_logits.argmax(dim=1) == y_qry).sum() / querysz

    return qry_loss, qry_acc


def train(db, net, device, meta_opt, epoch, log):
    n_train_iter = db.x_train.shape[0] // db.batchsz

    for batch_idx in range(n_train_iter):
        start_time = time.time()
        # Sample a batch of support and query images and labels.
        x_spt, y_spt, x_qry, y_qry = db.next()
```
- **EN**: This chunk defines `train`, which runs a training procedure that fits a heuristic or predictive model. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `train`，其作用是执行训练过程以拟合启发式或预测模型。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 155-166
```python
        task_num, setsz, c_, h, w = x_spt.size()

        n_inner_iter = 5
        meta_opt.zero_grad()

        # In parallel, trains one model per task. There is a support (x, y)
        # for each task and a query (x, y) for each task.
        compute_loss_for_task = functools.partial(loss_for_task, net, n_inner_iter)
        qry_losses, qry_accs = vmap(compute_loss_for_task)(x_spt, y_spt, x_qry, y_qry)

        # Compute the maml loss by summing together the returned losses.
        qry_losses.sum().backward()
```
- **EN**: This chunk continues `train` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `train`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 168-185
```python
        meta_opt.step()
        qry_losses = qry_losses.detach().sum() / task_num
        qry_accs = 100.0 * qry_accs.sum() / task_num
        i = epoch + float(batch_idx) / n_train_iter
        iter_time = time.time() - start_time
        if batch_idx % 4 == 0:
            print(
                f"[Epoch {i:.2f}] Train Loss: {qry_losses:.2f} | Acc: {qry_accs:.2f} | Time: {iter_time:.2f}"
            )

        log.append(
            {
                "epoch": i,
                "loss": qry_losses,
                "acc": qry_accs,
                "mode": "train",
                "time": time.time(),
            }
```
- **EN**: This chunk continues `train` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `train`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 186-200
```python
        )


def test(db, net, device, epoch, log):
    # Crucially in our testing procedure here, we do *not* fine-tune
    # the model during testing for simplicity.
    # Most research papers using MAML for this task do an extra
    # stage of fine-tuning here that should be added if you are
    # adapting this code for research.
    params = dict(net.named_parameters())
    buffers = dict(net.named_buffers())
    n_test_iter = db.x_test.shape[0] // db.batchsz

    qry_losses = []
    qry_accs = []
```
- **EN**: This chunk defines `test`, which verifies invariants and catches incorrect states early. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `test`，其作用是验证不变量，并尽早捕获错误状态。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 202-218
```python
    for batch_idx in range(n_test_iter):
        x_spt, y_spt, x_qry, y_qry = db.next("test")
        task_num, setsz, c_, h, w = x_spt.size()

        # TODO: Maybe pull this out into a separate module so it
        # doesn't have to be duplicated between `train` and `test`?
        n_inner_iter = 5

        for i in range(task_num):
            new_params = params
            for _ in range(n_inner_iter):
                spt_logits = functional_call(net, (new_params, buffers), x_spt[i])
                spt_loss = F.cross_entropy(spt_logits, y_spt[i])
                grads = torch.autograd.grad(spt_loss, new_params.values())
                new_params = {
                    k: new_params[k] - g * 1e-1 for k, g in zip(new_params, grads)
                }
```
- **EN**: This chunk continues `test` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `test`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 220-237
```python
            # The query loss and acc induced by these parameters.
            qry_logits = functional_call(net, (new_params, buffers), x_qry[i]).detach()
            qry_loss = F.cross_entropy(qry_logits, y_qry[i], reduction="none")
            qry_losses.append(qry_loss.detach())
            qry_accs.append((qry_logits.argmax(dim=1) == y_qry[i]).detach())

    qry_losses = torch.cat(qry_losses).mean().item()
    qry_accs = 100.0 * torch.cat(qry_accs).float().mean().item()
    print(f"[Epoch {epoch + 1:.2f}] Test Loss: {qry_losses:.2f} | Acc: {qry_accs:.2f}")
    log.append(
        {
            "epoch": epoch + 1,
            "loss": qry_losses,
            "acc": qry_accs,
            "mode": "test",
            "time": time.time(),
        }
    )
```
- **EN**: This chunk continues `test` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `test`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 240-257
```python
def plot(log):
    # Generally you should pull your plotting code out of your training
    # script but we are doing it here for brevity.
    df = pd.DataFrame(log)

    fig, ax = plt.subplots(figsize=(6, 4))
    train_df = df[df["mode"] == "train"]
    test_df = df[df["mode"] == "test"]
    ax.plot(train_df["epoch"], train_df["acc"], label="Train")
    ax.plot(test_df["epoch"], test_df["acc"], label="Test")
    ax.set_xlabel("Epoch")
    ax.set_ylabel("Accuracy")
    ax.set_ylim(70, 100)
    fig.legend(ncol=2, loc="lower right")
    fig.tight_layout()
    fname = "maml-accs.png"
    print(f"--- Plotting accuracy to {fname}")
    fig.savefig(fname)
```
- **EN**: This chunk defines `plot`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `plot`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 258-262
```python
    plt.close(fig)


if __name__ == "__main__":
    main()
```
- **EN**: This chunk continues `plot` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `plot`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

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
- **Jacobian/Hessian transforms**
  - EN: Works with Jacobian- or Hessian-style higher-order differentiation APIs.
  - CN: 处理 Jacobian/Hessian 风格的高阶微分 API。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch.nn.functional`, `torch.optim`, `torch.func`
- **Third-party modules / 第三方模块**: `matplotlib`, `matplotlib.pyplot`, `numpy`, `pandas`, `support.omniglot_loaders`
- **Standard library / 标准库**: `argparse`, `functools`, `time`
- **Primary symbols / 核心符号**: `main`, `loss_for_task`, `compute_loss`, `train`, `test`, `plot`
- **Note / 说明**: 13 imports were detected; only the first few are listed for readability. / 检测到 13 个导入项，为便于阅读这里只展示前若干项。
