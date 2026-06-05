# evaluate_model_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/pruning/_experimental/data_sparsifier/benchmarks/evaluate_model_metrics.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure. This specific file centers on `evaluate_model_metrics.py`. Dataset iteration, batching, and worker orchestration are central concerns here. The implementation is performance-facing and focuses on measurement fidelity or result reporting.
- **Purpose (CN) / 用途（中文）**: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。 该文件具体围绕 `evaluate_model_metrics.py` 展开。 数据集迭代、批处理以及 worker 协调是这里的核心关注点。 该实现面向性能分析，重点在于测量可信度或结果报告。
## Line-by-Line Analysis / 逐行分析

### Lines 1-14 / 第 1-14 行
```python
# mypy: allow-untyped-defs
import argparse

import numpy as np  # type: ignore[import]
import pandas as pd  # type: ignore[import]
import sklearn  # type: ignore[import]
from dlrm_s_pytorch import unpack_batch  # type: ignore[import]
from dlrm_utils import (  # type: ignore[import]
    dlrm_wrap,
    fetch_model,
    make_test_data_loader,
)

import torch
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch; standard-library helpers such as argparse; external packages such as numpy, pandas, sklearn, dlrm_s_pytorch:unpack_batch. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch；标准库辅助模块，如 argparse；外部包，如 numpy, pandas, sklearn, dlrm_s_pytorch:unpack_batch。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 17-33 / 第 17-33 行
```python
def inference_and_evaluation(dlrm, test_dataloader, device):
    """Perform inference and evaluation on the test dataset.
    The function returns the dictionary that contains evaluation metrics such as accuracy, f1, auc,
    precision, recall.
    Note: This function is a rewritten version of ```inference()``` present in dlrm_s_pytorch.py

    Args:
        dlrm (nn.Module)
            dlrm model object
        test_data_loader (torch dataloader):
            dataloader for the test dataset
        device (torch.device)
            device on which the inference happens
    """
    nbatches = len(test_dataloader)
    scores = []
    targets = []
```
- **EN**: Key callable entry points in this range include `inference_and_evaluation`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. This section supports data traversal, worker behavior, or sample batching in the input pipeline.
- **CN**: 这一段的重要可调用入口包括 `inference_and_evaluation`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 这一段支撑输入流水线中的数据遍历、worker 行为或样本批处理。

### Lines 35-50 / 第 35-50 行
```python
    for i, testBatch in enumerate(test_dataloader):
        # early exit if nbatches was set by the user and was exceeded
        if nbatches > 0 and i >= nbatches:
            break

        X_test, lS_o_test, lS_i_test, T_test, _, _ = unpack_batch(testBatch)
        # forward pass
        X_test, lS_o_test, lS_i_test = dlrm_wrap(
            X_test, lS_o_test, lS_i_test, device, ndevices=1
        )

        Z_test = dlrm(X_test, lS_o_test, lS_i_test)
        S_test = Z_test.detach().cpu().numpy()  # numpy array
        T_test = T_test.detach().cpu().numpy()  # numpy array
        scores.append(S_test)
        targets.append(T_test)
```
- **EN**: Key callable entry points in this range include `inference_and_evaluation`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. This section supports data traversal, worker behavior, or sample batching in the input pipeline.
- **CN**: 这一段的重要可调用入口包括 `inference_and_evaluation`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 这一段支撑输入流水线中的数据遍历、worker 行为或样本批处理。

### Lines 52-69 / 第 52-69 行
```python
    scores = np.concatenate(scores, axis=0)
    targets = np.concatenate(targets, axis=0)
    metrics = {
        "recall": lambda y_true, y_score: sklearn.metrics.recall_score(
            y_true=y_true, y_pred=np.round(y_score)
        ),
        "precision": lambda y_true, y_score: sklearn.metrics.precision_score(
            y_true=y_true, y_pred=np.round(y_score)
        ),
        "f1": lambda y_true, y_score: sklearn.metrics.f1_score(
            y_true=y_true, y_pred=np.round(y_score)
        ),
        "ap": sklearn.metrics.average_precision_score,
        "roc_auc": sklearn.metrics.roc_auc_score,
        "accuracy": lambda y_true, y_score: sklearn.metrics.accuracy_score(
            y_true=y_true, y_pred=np.round(y_score)
        ),
        "log_loss": lambda y_true, y_score: sklearn.metrics.log_loss(
```
- **EN**: Key callable entry points in this range include `inference_and_evaluation`. They package a focused unit of behavior behind named helpers or APIs. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `inference_and_evaluation`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 70-87 / 第 70-87 行
```python
            y_true=y_true, y_pred=y_score
        ),
    }

    all_metrics = {}
    for metric_name, metric_function in metrics.items():
        all_metrics[metric_name] = round(metric_function(targets, scores), 3)

    return all_metrics


def evaluate_metrics(test_dataloader, sparse_model_metadata):
    """Evaluates the metrics the sparsified metrics for the dlrm model on various sparsity levels,
    block shapes and norms. This function evaluates the model on the test dataset and dumps
    evaluation metrics in a csv file [model_performance.csv]
    """
    metadata = pd.read_csv(sparse_model_metadata)
    device = torch.device("cuda") if torch.cuda.is_available() else torch.device("cpu")
```
- **EN**: Key callable entry points in this range include `inference_and_evaluation`, `evaluate_metrics`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. This section supports data traversal, worker behavior, or sample batching in the input pipeline.
- **CN**: 这一段的重要可调用入口包括 `inference_and_evaluation`, `evaluate_metrics`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这一段支撑输入流水线中的数据遍历、worker 行为或样本批处理。

### Lines 89-104 / 第 89-104 行
```python
    metrics_dict: dict[str, list] = {
        "norm": [],
        "sparse_block_shape": [],
        "sparsity_level": [],
        "precision": [],
        "recall": [],
        "f1": [],
        "roc_auc": [],
        "accuracy": [],
        "log_loss": [],
    }

    for _, row in metadata.iterrows():
        norm, sbs, sl = row["norm"], row["sparse_block_shape"], row["sparsity_level"]
        model_path = row["path"]
        model = fetch_model(model_path, device)
```
- **EN**: Key callable entry points in this range include `evaluate_metrics`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `evaluate_metrics`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 106-119 / 第 106-119 行
```python
        model_metrics = inference_and_evaluation(model, test_dataloader, device)
        key = f"{norm}_{sbs}_{sl}"
        print(key, "=", model_metrics)

        metrics_dict["norm"].append(norm)
        metrics_dict["sparse_block_shape"].append(sbs)
        metrics_dict["sparsity_level"].append(sl)

        for key, value in model_metrics.items():
            if key in metrics_dict:
                metrics_dict[key].append(value)

    sparse_model_metrics = pd.DataFrame(metrics_dict)
    print(sparse_model_metrics)
```
- **EN**: Key callable entry points in this range include `evaluate_metrics`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. This section supports data traversal, worker behavior, or sample batching in the input pipeline.
- **CN**: 这一段的重要可调用入口包括 `evaluate_metrics`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 这一段支撑输入流水线中的数据遍历、worker 行为或样本批处理。

### Lines 121-137 / 第 121-137 行
```python
    filename = "sparse_model_metrics.csv"
    sparse_model_metrics.to_csv(filename, index=False)
    print(f"Model metrics file saved to {filename}")


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--raw-data-file", "--raw_data_file", type=str)
    parser.add_argument("--processed-data-file", "--processed_data_file", type=str)
    parser.add_argument("--sparse-model-metadata", "--sparse_model_metadata", type=str)

    args = parser.parse_args()

    # Fetch test data loader
    test_dataloader = make_test_data_loader(
        args.raw_data_file, args.processed_data_file
    )
```
- **EN**: Key callable entry points in this range include `evaluate_metrics`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. This section supports data traversal, worker behavior, or sample batching in the input pipeline.
- **CN**: 这一段的重要可调用入口包括 `evaluate_metrics`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 这一段支撑输入流水线中的数据遍历、worker 行为或样本批处理。

### Lines 139-140 / 第 139-140 行
```python
    # Evaluate metrics
    evaluate_metrics(test_dataloader, args.sparse_model_metadata)
```
- **EN**: This chunk continues the implementation of `evaluate_metrics`, filling in the details of its control flow or data handling. This section supports data traversal, worker behavior, or sample batching in the input pipeline.
- **CN**: 这一段延续了 `evaluate_metrics` 的实现，继续补充其控制流或数据处理细节。 这一段支撑输入流水线中的数据遍历、worker 行为或样本批处理。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure.
  - CN: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。
- **Pruning and sparsity**
  - EN: Controls parameter masking, sparse layouts, or pruning schedules that reduce effective model size.
  - CN: 控制参数掩码、稀疏布局或剪枝计划，以降低模型的有效规模。
- **Data loading**
  - EN: Coordinates dataset traversal, batching, multiprocessing workers, and sample collation.
  - CN: 协调数据集遍历、批处理、多进程 worker 以及样本拼装。
- **Benchmarking**
  - EN: Focuses on repeatable measurement, result comparison, and performance diagnostics.
  - CN: 聚焦可重复的性能测量、结果比较与性能诊断。
- **Serialization safety**
  - EN: Keeps persisted data exchange bounded, predictable, and easier to validate.
  - CN: 让持久化数据交换保持有界、可预测且更易验证。
- **inference_and_evaluation**
  - EN: `inference_and_evaluation` is a representative function that exposes or coordinates an important action in this module.
  - CN: `inference_and_evaluation` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`
- **Python standard library / Python 标准库**: `argparse`
- **Third-party packages / 第三方包**: `numpy`, `pandas`, `sklearn`, `dlrm_s_pytorch:unpack_batch`, `dlrm_utils:dlrm_wrap`, `dlrm_utils:fetch_model`, `dlrm_utils:make_test_data_loader`
- **Primary symbols / 核心符号**: `inference_and_evaluation`, `evaluate_metrics`
