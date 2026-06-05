# cifar10_opacus.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `functorch/examples/dp_cifar10/cifar10_opacus.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides runnable examples that demonstrate functorch transforms, compilation flows, and research-style use cases.
- **Purpose (CN)**: 提供可运行示例，展示 functorch 变换、编译流程以及研究型用例。
## Line-by-Line Analysis / 逐行分析

### Lines 1-23
```python
#!/usr/bin/env python3
# Copyright (c) Facebook, Inc. and its affiliates. All Rights Reserved

"""
Runs CIFAR10 training with differential privacy.
"""

import argparse
import logging
import shutil
import sys
from datetime import datetime, timedelta

import numpy as np
from opacus import PrivacyEngine
from torchvision import models, transforms
from torchvision.datasets import CIFAR10
from tqdm import tqdm

import torch
import torch.nn as nn
import torch.optim as optim
import torch.utils.data
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch.nn, torch.optim, torch.utils.data; third-party modules such as numpy, opacus, torchvision, and 2 more; standard-library modules such as argparse, logging, shutil, sys, and 1 more for the logic below.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch.nn、torch.optim、torch.utils.data；第三方模块，如 numpy、opacus、torchvision 等共 5 项；标准库模块，如 argparse、logging、shutil、sys 等共 5 项组织在一起，供下方逻辑使用。

### Lines 26-52
```python
logging.basicConfig(
    format="%(asctime)s:%(levelname)s:%(message)s",
    datefmt="%m/%d/%Y %H:%M:%S",
    stream=sys.stdout,
)
logger = logging.getLogger("ddp")
logger.setLevel(level=logging.INFO)


def save_checkpoint(state, is_best, filename="checkpoint.tar"):
    torch.save(state, filename)
    if is_best:
        shutil.copyfile(filename, "model_best.pth.tar")


def accuracy(preds, labels):
    return (preds == labels).mean()


def train(args, model, train_loader, optimizer, privacy_engine, epoch, device):
    start_time = datetime.now()

    model.train()
    criterion = nn.CrossEntropyLoss()

    losses = []
    top1_acc = []
```
- **EN**: This chunk defines `train`, which runs a training procedure that fits a heuristic or predictive model. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `train`，其作用是执行训练过程以拟合启发式或预测模型。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 54-76
```python
    for i, (images, target) in enumerate(tqdm(train_loader)):
        images = images.to(device)
        target = target.to(device)

        # compute output
        output = model(images)
        loss = criterion(output, target)
        preds = np.argmax(output.detach().cpu().numpy(), axis=1)
        labels = target.detach().cpu().numpy()

        # measure accuracy and record loss
        acc1 = accuracy(preds, labels)

        losses.append(loss.item())
        top1_acc.append(acc1)

        # compute gradient and do SGD step
        loss.backward()

        # make sure we take a step after processing the last mini-batch in the
        # epoch to ensure we start the next epoch with a clean state
        optimizer.step()
        optimizer.zero_grad()
```
- **EN**: This chunk continues `train` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `train`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 78-104
```python
        if i % args.print_freq == 0:
            if not args.disable_dp:
                epsilon, best_alpha = privacy_engine.accountant.get_privacy_spent(
                    delta=args.delta,
                    alphas=[1 + x / 10.0 for x in range(1, 100)] + list(range(12, 64)),
                )
                print(
                    f"\tTrain Epoch: {epoch} \t"
                    f"Loss: {np.mean(losses):.6f} "
                    f"Acc@1: {np.mean(top1_acc):.6f} "
                    f"(ε = {epsilon:.2f}, δ = {args.delta}) for α = {best_alpha}"
                )
            else:
                print(
                    f"\tTrain Epoch: {epoch} \t"
                    f"Loss: {np.mean(losses):.6f} "
                    f"Acc@1: {np.mean(top1_acc):.6f} "
                )
    train_duration = datetime.now() - start_time
    return train_duration


def test(args, model, test_loader, device):
    model.eval()
    criterion = nn.CrossEntropyLoss()
    losses = []
    top1_acc = []
```
- **EN**: This chunk defines `test`, which verifies invariants and catches incorrect states early. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `test`，其作用是验证不变量，并尽早捕获错误状态。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 106-132
```python
    with torch.no_grad():
        for images, target in tqdm(test_loader):
            images = images.to(device)
            target = target.to(device)

            output = model(images)
            loss = criterion(output, target)
            preds = np.argmax(output.detach().cpu().numpy(), axis=1)
            labels = target.detach().cpu().numpy()
            acc1 = accuracy(preds, labels)

            losses.append(loss.item())
            top1_acc.append(acc1)

    top1_avg = np.mean(top1_acc)

    print(f"\tTest set:Loss: {np.mean(losses):.6f} Acc@1: {top1_avg:.6f}")
    return np.mean(top1_acc)


def main():
    args = parse_args()

    if args.debug >= 1:
        logger.setLevel(level=logging.DEBUG)

    device = args.device
```
- **EN**: This chunk defines `main`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `main`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 134-159
```python
    if args.secure_rng:
        try:
            import torchcsprng as prng
        except ImportError as e:
            msg = (
                "To use secure RNG, you must install the torchcsprng package! "
                "Check out the instructions here: https://github.com/pytorch/csprng#installation"
            )
            raise ImportError(msg) from e

        generator = prng.create_random_device_generator("/dev/urandom")

    else:
        generator = None

    augmentations = [
        transforms.RandomCrop(32, padding=4),
        transforms.RandomHorizontalFlip(),
    ]
    normalize = [
        transforms.ToTensor(),
        transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010)),
    ]
    train_transform = transforms.Compose(
        augmentations + normalize if args.disable_dp else normalize
    )
```
- **EN**: The import section wires together third-party modules such as torchcsprng for the logic below. This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把第三方模块，如 torchcsprng组织在一起，供下方逻辑使用。 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 161-185
```python
    test_transform = transforms.Compose(normalize)

    train_dataset = CIFAR10(
        root=args.data_root, train=True, download=True, transform=train_transform
    )

    train_loader = torch.utils.data.DataLoader(
        train_dataset,
        batch_size=int(args.sample_rate * len(train_dataset)),
        generator=generator,
        num_workers=args.workers,
        pin_memory=True,
    )

    test_dataset = CIFAR10(
        root=args.data_root, train=False, download=True, transform=test_transform
    )
    test_loader = torch.utils.data.DataLoader(
        test_dataset,
        batch_size=args.batch_size_test,
        shuffle=False,
        num_workers=args.workers,
    )

    best_acc1 = 0
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 187-204
```python
    model = models.__dict__[args.architecture](
        pretrained=False, norm_layer=(lambda c: nn.GroupNorm(args.gn_groups, c))
    )
    model = model.to(device)

    if args.optim == "SGD":
        optimizer = optim.SGD(
            model.parameters(),
            lr=args.lr,
            momentum=args.momentum,
            weight_decay=args.weight_decay,
        )
    elif args.optim == "RMSprop":
        optimizer = optim.RMSprop(model.parameters(), lr=args.lr)
    elif args.optim == "Adam":
        optimizer = optim.Adam(model.parameters(), lr=args.lr)
    else:
        raise NotImplementedError("Optimizer not recognized. Please check spelling")
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 206-230
```python
    privacy_engine = None
    if not args.disable_dp:
        if args.clip_per_layer:
            # Each layer has the same clipping threshold. The total grad norm is still bounded by `args.max_per_sample_grad_norm`.
            n_layers = len(
                [(n, p) for n, p in model.named_parameters() if p.requires_grad]
            )
            max_grad_norm = [
                args.max_per_sample_grad_norm / np.sqrt(n_layers)
            ] * n_layers
        else:
            max_grad_norm = args.max_per_sample_grad_norm

        privacy_engine = PrivacyEngine(
            secure_mode=args.secure_rng,
        )
        clipping = "per_layer" if args.clip_per_layer else "flat"
        model, optimizer, train_loader = privacy_engine.make_private(
            module=model,
            optimizer=optimizer,
            data_loader=train_loader,
            noise_multiplier=args.sigma,
            max_grad_norm=max_grad_norm,
            clipping=clipping,
        )
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 232-252
```python
    # Store some logs
    accuracy_per_epoch = []
    time_per_epoch = []

    for epoch in range(args.start_epoch, args.epochs + 1):
        if args.lr_schedule == "cos":
            lr = args.lr * 0.5 * (1 + np.cos(np.pi * epoch / (args.epochs + 1)))
            for param_group in optimizer.param_groups:
                param_group["lr"] = lr

        train_duration = train(
            args, model, train_loader, optimizer, privacy_engine, epoch, device
        )
        top1_acc = test(args, model, test_loader, device)

        # remember best acc@1 and save checkpoint
        is_best = top1_acc > best_acc1
        best_acc1 = max(top1_acc, best_acc1)

        time_per_epoch.append(train_duration)
        accuracy_per_epoch.append(float(top1_acc))
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 254-278
```python
        save_checkpoint(
            {
                "epoch": epoch + 1,
                "arch": "Convnet",
                "state_dict": model.state_dict(),
                "best_acc1": best_acc1,
                "optimizer": optimizer.state_dict(),
            },
            is_best,
            filename=args.checkpoint_file + ".tar",
        )

    time_per_epoch_seconds = [t.total_seconds() for t in time_per_epoch]
    avg_time_per_epoch = sum(time_per_epoch_seconds) / len(time_per_epoch_seconds)
    metrics = {
        "accuracy": best_acc1,
        "accuracy_per_epoch": accuracy_per_epoch,
        "avg_time_per_epoch_str": str(timedelta(seconds=int(avg_time_per_epoch))),
        "time_per_epoch": time_per_epoch_seconds,
    }

    logger.info(
        "\nNote:\n- 'total_time' includes the data loading time, training time and testing time.\n- 'time_per_epoch' measures the training time only.\n"
    )
    logger.info(metrics)
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 281-308
```python
def parse_args():
    parser = argparse.ArgumentParser(description="PyTorch CIFAR10 DP Training")
    parser.add_argument(
        "-j",
        "--workers",
        default=2,
        type=int,
        metavar="N",
        help="number of data loading workers (default: 2)",
    )
    parser.add_argument(
        "--epochs",
        default=90,
        type=int,
        metavar="N",
        help="number of total epochs to run",
    )
    parser.add_argument(
        "--start-epoch",
        default=1,
        type=int,
        metavar="N",
        help="manual epoch number (useful on restarts)",
    )
    parser.add_argument(
        "-b",
        "--batch-size-test",
        default=256,
```
- **EN**: This chunk defines `parse_args`, which parses textual or structured input into internal representations. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `parse_args`，其作用是把文本或结构化输入解析为内部表示。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 309-336
```python
        type=int,
        metavar="N",
        help="mini-batch size for test dataset (default: 256)",
    )
    parser.add_argument(
        "--sample-rate",
        default=0.005,
        type=float,
        metavar="SR",
        help="sample rate used for batch construction (default: 0.005)",
    )
    parser.add_argument(
        "--lr",
        "--learning-rate",
        default=0.1,
        type=float,
        metavar="LR",
        help="initial learning rate",
        dest="lr",
    )
    parser.add_argument(
        "--momentum", default=0.9, type=float, metavar="M", help="SGD momentum"
    )
    parser.add_argument(
        "--wd",
        "--weight-decay",
        default=0,
        type=float,
```
- **EN**: This chunk continues `parse_args` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `parse_args`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 337-364
```python
        metavar="W",
        help="SGD weight decay",
        dest="weight_decay",
    )
    parser.add_argument(
        "-p",
        "--print-freq",
        default=10,
        type=int,
        metavar="N",
        help="print frequency (default: 10)",
    )
    parser.add_argument(
        "--resume",
        default="",
        type=str,
        metavar="PATH",
        help="path to latest checkpoint (default: none)",
    )
    parser.add_argument(
        "-e",
        "--evaluate",
        dest="evaluate",
        action="store_true",
        help="evaluate model on validation set",
    )
    parser.add_argument(
        "--seed", default=None, type=int, help="seed for initializing training. "
```
- **EN**: This chunk continues `parse_args` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `parse_args`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 365-392
```python
    )

    parser.add_argument(
        "--sigma",
        type=float,
        default=1.5,
        metavar="S",
        help="Noise multiplier (default 1.0)",
    )
    parser.add_argument(
        "-c",
        "--max-per-sample-grad_norm",
        type=float,
        default=10.0,
        metavar="C",
        help="Clip per-sample gradients to this norm (default 1.0)",
    )
    parser.add_argument(
        "--disable-dp",
        action="store_true",
        default=False,
        help="Disable privacy training and just train with vanilla SGD",
    )
    parser.add_argument(
        "--secure-rng",
        action="store_true",
        default=False,
        help="Enable Secure RNG to have trustworthy privacy guarantees."
```
- **EN**: This chunk continues `parse_args` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `parse_args`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 393-420
```python
        "Comes at a performance cost. Opacus will emit a warning if secure rng is off,"
        "indicating that for production use it's recommender to turn it on.",
    )
    parser.add_argument(
        "--delta",
        type=float,
        default=1e-5,
        metavar="D",
        help="Target delta (default: 1e-5)",
    )

    parser.add_argument(
        "--checkpoint-file",
        type=str,
        default="checkpoint",
        help="path to save check points",
    )
    parser.add_argument(
        "--data-root",
        type=str,
        default="../cifar10",
        help="Where CIFAR10 is/will be stored",
    )
    parser.add_argument(
        "--log-dir",
        type=str,
        default="/tmp/stat/tensorboard",
        help="Where Tensorboard log will be stored",
```
- **EN**: This chunk continues `parse_args` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `parse_args`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 421-441
```python
    )
    parser.add_argument(
        "--optim",
        type=str,
        default="SGD",
        help="Optimizer to use (Adam, RMSprop, SGD)",
    )
    parser.add_argument(
        "--lr-schedule", type=str, choices=["constant", "cos"], default="cos"
    )

    parser.add_argument(
        "--device", type=str, default="cuda", help="Device on which to run the code."
    )

    parser.add_argument(
        "--architecture",
        type=str,
        default="resnet18",
        help="model from torchvision to run",
    )
```
- **EN**: This chunk continues `parse_args` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `parse_args`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 443-468
```python
    parser.add_argument(
        "--gn-groups",
        type=int,
        default=8,
        help="Number of groups in GroupNorm",
    )

    parser.add_argument(
        "--clip-per-layer",
        "--clip_per_layer",
        action="store_true",
        default=False,
        help="Use static per-layer clipping with the same clipping threshold for each layer. Necessary for DDP. If `False` (default), uses flat clipping.",
    )
    parser.add_argument(
        "--debug",
        type=int,
        default=0,
        help="debug level (default: 0)",
    )

    return parser.parse_args()


if __name__ == "__main__":
    main()
```
- **EN**: This chunk continues `parse_args` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `parse_args`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

## Key Concepts / 关键概念

- **Functional tensor transforms**
  - EN: Applies composable transforms such as batching, gradients, and tracing.
  - CN: 应用可组合的变换，例如 batching、梯度和 tracing。
- **Model training**
  - EN: Runs data-driven training loops to fit heuristic or regression models.
  - CN: 运行数据驱动的训练循环，以拟合启发式或回归模型。
- **Gradient transforms**
  - EN: Applies gradient-oriented transforms such as grad, vjp, or higher-order derivatives.
  - CN: 应用梯度相关变换，例如 grad、vjp 或高阶导数。
- **save_checkpoint**
  - EN: `save_checkpoint` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `save_checkpoint` 是本文件声明、导出或驱动的显著符号之一。
- **accuracy**
  - EN: `accuracy` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `accuracy` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch.nn`, `torch.optim`, `torch.utils.data`
- **Third-party modules / 第三方模块**: `numpy`, `opacus`, `torchvision`, `torchvision.datasets`, `tqdm`, `torchcsprng`
- **Standard library / 标准库**: `argparse`, `logging`, `shutil`, `sys`, `datetime`
- **Primary symbols / 核心符号**: `save_checkpoint`, `accuracy`, `train`, `test`, `main`, `parse_args`
- **Note / 说明**: 15 imports were detected; only the first few are listed for readability. / 检测到 15 个导入项，为便于阅读这里只展示前若干项。
