# evaluate_disk_savings.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/pruning/_experimental/data_sparsifier/benchmarks/evaluate_disk_savings.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure. This specific file centers on `evaluate_disk_savings.py`. The implementation is performance-facing and focuses on measurement fidelity or result reporting. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。 该文件具体围绕 `evaluate_disk_savings.py` 展开。 该实现面向性能分析，重点在于测量可信度或结果报告。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-17 / 第 1-17 行
```python
# mypy: allow-untyped-defs
import argparse
import copy
import os
import time
import zipfile
from zipfile import ZipFile

import pandas as pd  # type: ignore[import]
from dlrm_utils import get_dlrm_model, get_valid_name  # type: ignore[import]

import torch
from torch.ao.pruning._experimental.data_sparsifier import DataNormSparsifier


def create_attach_sparsifier(model, **sparse_config):
    """Create a DataNormSparsifier and the attach it to the model embedding layers
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.ao.pruning._experimental.data_sparsifier:DataNormSparsifier; standard-library helpers such as argparse, copy, os, time; external packages such as pandas, dlrm_utils:get_dlrm_model, dlrm_utils:get_valid_name. Key callable entry points in this range include `create_attach_sparsifier`. They package a focused unit of behavior behind named helpers or APIs. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.ao.pruning._experimental.data_sparsifier:DataNormSparsifier；标准库辅助模块，如 argparse, copy, os, time；外部包，如 pandas, dlrm_utils:get_dlrm_model, dlrm_utils:get_valid_name。 这一段的重要可调用入口包括 `create_attach_sparsifier`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 19-33 / 第 19-33 行
```python
    Args:
        model (nn.Module)
            layer of the model that needs to be attached to the sparsifier
        sparse_config (Dict)
            Config to the DataNormSparsifier. Should contain the following keys:
                - sparse_block_shape
                - norm
                - sparsity_level
    """
    data_norm_sparsifier = DataNormSparsifier(**sparse_config)
    for name, parameter in model.named_parameters():
        if "emb_l" in name:
            valid_name = get_valid_name(name)
            data_norm_sparsifier.add_data(name=valid_name, data=parameter)
    return data_norm_sparsifier
```
- **EN**: Key callable entry points in this range include `create_attach_sparsifier`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `create_attach_sparsifier`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 36-53 / 第 36-53 行
```python
def save_model_states(
    state_dict,
    sparsified_model_dump_path,
    save_file_name,
    sparse_block_shape,
    norm,
    zip=True,
):
    """Dumps the state_dict() of the model.

    Args:
        state_dict (Dict)
            The state_dict() as dumped by dlrm_s_pytorch.py. Only the model state will be extracted
            from this dictionary. This corresponds to the 'state_dict' key in the state_dict dictionary.
            >>> model_state = state_dict["state_dict"]
        save_file_name (str)
            The filename (not path) when saving the model state dictionary
        sparse_block_shape (Tuple)
```
- **EN**: Key callable entry points in this range include `save_model_states`. They package a focused unit of behavior behind named helpers or APIs. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `save_model_states`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 54-68 / 第 54-68 行
```python
            The block shape corresponding to the data norm sparsifier. **Used for creating save directory**
        norm (str)
            type of norm (L1, L2) for the datanorm sparsifier. **Used for creating save directory**
        zip (bool)
            if True, the file is zip-compressed.
    """
    folder_name = os.path.join(sparsified_model_dump_path, str(norm))

    # save model only states
    folder_str = f"config_{sparse_block_shape}"
    model_state = state_dict["state_dict"]
    model_state_path = os.path.join(folder_name, folder_str, save_file_name)

    os.makedirs(os.path.dirname(model_state_path), exist_ok=True)
    torch.save(model_state, model_state_path)
```
- **EN**: Key callable entry points in this range include `save_model_states`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `save_model_states`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 70-80 / 第 70-80 行
```python
    if zip:
        zip_path = model_state_path.replace(".ckpt", ".zip")
        with ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as zip:
            zip.write(model_state_path, save_file_name)
        os.remove(model_state_path)  # store it as zip, remove uncompressed
        model_state_path = zip_path

    model_state_path = os.path.abspath(model_state_path)
    file_size = os.path.getsize(model_state_path)
    file_size = file_size >> 20  # size in mb
    return model_state_path, file_size
```
- **EN**: Key callable entry points in this range include `save_model_states`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `save_model_states`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 83-100 / 第 83-100 行
```python
def sparsify_model(path_to_model, sparsified_model_dump_path):
    """Sparsifies the embedding layers of the dlrm model for different sparsity levels, norms and block shapes
    using the DataNormSparsifier.
    The function tracks the step time of the sparsifier and the size of the compressed checkpoint and collates
    it into a csv.

    Note::
        This function dumps a csv sparse_model_metadata.csv in the current directory.

    Args:
        path_to_model (str)
            path to the trained criteo model ckpt file
        sparsity_levels (List of float)
            list of sparsity levels to be sparsified on
        norms (List of str)
            list of norms to be sparsified on
        sparse_block_shapes (List of tuples)
            List of sparse block shapes to be sparsified on
```
- **EN**: Key callable entry points in this range include `sparsify_model`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `sparsify_model`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 101-116 / 第 101-116 行
```python
    """
    sparsity_levels = [sl / 10 for sl in range(10)]
    sparsity_levels += [0.91, 0.92, 0.93, 0.94, 0.95, 0.96, 0.97, 0.98, 0.99, 1.0]

    norms = ["L1", "L2"]
    sparse_block_shapes = [(1, 1), (1, 4)]

    device = torch.device("cuda") if torch.cuda.is_available() else torch.device("cpu")

    print("Running for sparsity levels - ", sparsity_levels)
    print("Running for sparse block shapes - ", sparse_block_shapes)
    print("Running for norms - ", norms)

    orig_model = get_dlrm_model()
    saved_state = torch.load(path_to_model, map_location=device)
    orig_model.load_state_dict(saved_state["state_dict"])
```
- **EN**: Key callable entry points in this range include `sparsify_model`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `sparsify_model`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 118-135 / 第 118-135 行
```python
    orig_model = orig_model.to(device)
    step_time_dict = {}

    stat_dict: dict[str, list] = {
        "norm": [],
        "sparse_block_shape": [],
        "sparsity_level": [],
        "step_time_sec": [],
        "zip_file_size": [],
        "path": [],
    }
    for norm in norms:
        for sbs in sparse_block_shapes:
            if norm == "L2" and sbs == (1, 1):
                continue
            for sl in sparsity_levels:
                model = copy.deepcopy(orig_model)
                sparsifier = create_attach_sparsifier(
```
- **EN**: Key callable entry points in this range include `sparsify_model`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `sparsify_model`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 136-149 / 第 136-149 行
```python
                    model, sparse_block_shape=sbs, norm=norm, sparsity_level=sl
                )

                t1 = time.time()
                sparsifier.step()
                t2 = time.time()

                step_time = t2 - t1
                norm_sl = f"{norm}_{sbs}_{sl}"
                print(f"Step Time for {norm_sl}=: {step_time} s")

                step_time_dict[norm_sl] = step_time

                sparsifier.squash_mask()
```
- **EN**: Key callable entry points in this range include `sparsify_model`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `sparsify_model`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 151-166 / 第 151-166 行
```python
                saved_state["state_dict"] = model.state_dict()
                file_name = f"criteo_model_norm={norm}_sl={sl}.ckpt"
                state_path, file_size = save_model_states(
                    saved_state, sparsified_model_dump_path, file_name, sbs, norm=norm
                )

                stat_dict["norm"].append(norm)
                stat_dict["sparse_block_shape"].append(sbs)
                stat_dict["sparsity_level"].append(sl)
                stat_dict["step_time_sec"].append(step_time)
                stat_dict["zip_file_size"].append(file_size)
                stat_dict["path"].append(state_path)

    df = pd.DataFrame(stat_dict)
    filename = "sparse_model_metadata.csv"
    df.to_csv(filename, index=False)
```
- **EN**: Key callable entry points in this range include `sparsify_model`. They package a focused unit of behavior behind named helpers or APIs. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `sparsify_model`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 168-179 / 第 168-179 行
```python
    print(f"Saved sparsified metadata file in {filename}")


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--model-path", "--model_path", type=str)
    parser.add_argument(
        "--sparsified-model-dump-path", "--sparsified_model_dump_path", type=str
    )
    args = parser.parse_args()

    sparsify_model(args.model_path, args.sparsified_model_dump_path)
```
- **EN**: Key callable entry points in this range include `sparsify_model`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `sparsify_model`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure.
  - CN: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Pruning and sparsity**
  - EN: Controls parameter masking, sparse layouts, or pruning schedules that reduce effective model size.
  - CN: 控制参数掩码、稀疏布局或剪枝计划，以降低模型的有效规模。
- **Data loading**
  - EN: Coordinates dataset traversal, batching, multiprocessing workers, and sample collation.
  - CN: 协调数据集遍历、批处理、多进程 worker 以及样本拼装。
- **Benchmarking**
  - EN: Focuses on repeatable measurement, result comparison, and performance diagnostics.
  - CN: 聚焦可重复的性能测量、结果比较与性能诊断。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.ao.pruning._experimental.data_sparsifier:DataNormSparsifier`
- **Python standard library / Python 标准库**: `argparse`, `copy`, `os`, `time`, `zipfile`, `zipfile:ZipFile`
- **Third-party packages / 第三方包**: `pandas`, `dlrm_utils:get_dlrm_model`, `dlrm_utils:get_valid_name`
- **Primary symbols / 核心符号**: `create_attach_sparsifier`, `save_model_states`, `sparsify_model`
