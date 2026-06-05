# dlrm_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/pruning/_experimental/data_sparsifier/benchmarks/dlrm_utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure. This specific file centers on `dlrm_utils.py`. Dataset iteration, batching, and worker orchestration are central concerns here. The implementation is performance-facing and focuses on measurement fidelity or result reporting. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。 该文件具体围绕 `dlrm_utils.py` 展开。 数据集迭代、批处理以及 worker 协调是这里的核心关注点。 该实现面向性能分析，重点在于测量可信度或结果报告。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行
```python
# mypy: allow-untyped-defs
import os
import zipfile

import numpy as np  # type: ignore[import]
from dlrm_data_pytorch import (  # type: ignore[import]
    collate_wrapper_criteo_offset,
    CriteoDataset,
)
from dlrm_s_pytorch import DLRM_Net  # type: ignore[import]

import torch
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch; standard-library helpers such as os, zipfile; external packages such as numpy, dlrm_data_pytorch:collate_wrapper_criteo_offset, dlrm_data_pytorch:CriteoDataset, dlrm_s_pytorch:DLRM_Net. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch；标准库辅助模块，如 os, zipfile；外部包，如 numpy, dlrm_data_pytorch:collate_wrapper_criteo_offset, dlrm_data_pytorch:CriteoDataset, dlrm_s_pytorch:DLRM_Net。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 15-30 / 第 15-30 行
```python
class SparseDLRM(DLRM_Net):
    """The SparseDLRM model is a wrapper around the DLRM_Net model that tries
    to use torch.sparse tensors for the features obtained after the ```interact_features()```
    call. The idea is to do a simple torch.mm() with the weight matrix of the first linear
    layer of the top layer.
    """

    def forward(self, dense_x, lS_o, lS_i):
        x = self.apply_mlp(dense_x, self.bot_l)  # dense features
        ly = self.apply_emb(lS_o, lS_i, self.emb_l, self.v_W_l)  # apply embedding bag
        z = self.interact_features(x, ly)

        z = z.to_sparse_coo()
        z = torch.mm(z, self.top_l[0].weight.T).add(self.top_l[0].bias)
        for layer in self.top_l[1:]:
            z = layer(z)
```
- **EN**: It introduces or extends class-level abstractions such as `SparseDLRM`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `SparseDLRM` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 32-49 / 第 32-49 行
```python
        return z


def get_valid_name(name):
    """Replaces '.' with '_' as names with '.' are invalid in data sparsifier"""
    return name.replace(".", "_")


def get_dlrm_model(sparse_dlrm=False):
    """Obtain dlrm model. The configs specified are based on the script in
    bench/dlrm_s_criteo_kaggle.sh. The same config is used to train the model
    for benchmarking on data sparsifier.
    """
    dlrm_model_config = {
        "m_spa": 16,
        "ln_emb": np.array(
            [
                1460,
```
- **EN**: It introduces or extends class-level abstractions such as `SparseDLRM`, which organize state and behavior for this subsystem. Key callable entry points in this range include `get_valid_name`, `get_dlrm_model`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 它引入或扩展了 `SparseDLRM` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `get_valid_name`, `get_dlrm_model`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 50-67 / 第 50-67 行
```python
                583,
                10131227,
                2202608,
                305,
                24,
                12517,
                633,
                3,
                93145,
                5683,
                8351593,
                3194,
                27,
                14992,
                5461306,
                10,
                5652,
                2173,
```
- **EN**: Key callable entry points in this range include `get_dlrm_model`. They package a focused unit of behavior behind named helpers or APIs. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `get_dlrm_model`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 68-85 / 第 68-85 行
```python
                4,
                7046547,
                18,
                15,
                286181,
                105,
                142572,
            ],
            dtype=np.int32,
        ),
        "ln_bot": np.array([13, 512, 256, 64, 16]),
        "ln_top": np.array([367, 512, 256, 1]),
        "arch_interaction_op": "dot",
        "arch_interaction_itself": False,
        "sigmoid_bot": -1,
        "sigmoid_top": 2,
        "sync_dense_params": True,
        "loss_threshold": 0.0,
```
- **EN**: Key callable entry points in this range include `get_dlrm_model`. They package a focused unit of behavior behind named helpers or APIs. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `get_dlrm_model`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 86-100 / 第 86-100 行
```python
        "ndevices": 1,
        "qr_flag": False,
        "qr_operation": "mult",
        "qr_collisions": 4,
        "qr_threshold": 200,
        "md_flag": False,
        "md_threshold": 200,
        "weighted_pooling": None,
        "loss_function": "bce",
    }
    if sparse_dlrm:
        dlrm_model = SparseDLRM(**dlrm_model_config)
    else:
        dlrm_model = DLRM_Net(**dlrm_model_config)
    return dlrm_model
```
- **EN**: Key callable entry points in this range include `get_dlrm_model`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `get_dlrm_model`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 103-118 / 第 103-118 行
```python
def dlrm_wrap(X, lS_o, lS_i, device, ndevices=1):
    """Rewritten simpler version of ```dlrm_wrap()``` found in dlrm_s_pytorch.py.
    This function simply moves the input tensors into the device and without the forward pass
    """
    if ndevices == 1:
        lS_i = (
            [S_i.to(device) for S_i in lS_i]
            if isinstance(lS_i, list)
            else lS_i.to(device)
        )
        lS_o = (
            [S_o.to(device) for S_o in lS_o]
            if isinstance(lS_o, list)
            else lS_o.to(device)
        )
    return X.to(device), lS_o, lS_i
```
- **EN**: Key callable entry points in this range include `dlrm_wrap`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `dlrm_wrap`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 121-138 / 第 121-138 行
```python
def make_test_data_loader(raw_data_file_path, processed_data_file):
    """Function to create dataset and dataloaders for the test dataset.
    Rewritten simpler version of ```make_criteo_and_loaders()``` from the dlrm_data_pytorch.py
    that makes the test dataset and dataloaders only for the ***kaggle criteo dataset***
    """
    test_data = CriteoDataset(
        "kaggle",
        -1,
        0.0,
        "total",
        "test",
        raw_data_file_path,
        processed_data_file,
        False,
        False,
    )
    test_loader = torch.utils.data.DataLoader(
        test_data,
```
- **EN**: Key callable entry points in this range include `make_test_data_loader`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. This section supports data traversal, worker behavior, or sample batching in the input pipeline.
- **CN**: 这一段的重要可调用入口包括 `make_test_data_loader`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 这一段支撑输入流水线中的数据遍历、worker 行为或样本批处理。

### Lines 139-151 / 第 139-151 行
```python
        batch_size=16384,
        shuffle=False,
        num_workers=7,
        collate_fn=collate_wrapper_criteo_offset,
        pin_memory=False,
        drop_last=False,
    )
    return test_loader


def fetch_model(model_path, device, sparse_dlrm=False):
    """This function unzips the zipped model checkpoint (if zipped) and returns a
    model object
```
- **EN**: Key callable entry points in this range include `make_test_data_loader`, `fetch_model`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `make_test_data_loader`, `fetch_model`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 153-169 / 第 153-169 行
```python
    Args:
        model_path (str)
            path pointing to the zipped/raw model checkpoint file that was dumped in evaluate disk savings
        device (torch.device)
            device to which model needs to be loaded to
    """
    if zipfile.is_zipfile(model_path):
        with zipfile.ZipFile(model_path, "r", zipfile.ZIP_DEFLATED) as zip_ref:
            zip_ref.extractall(os.path.dirname(model_path))
            unzip_path = model_path.replace(".zip", ".ckpt")
    else:
        unzip_path = model_path

    model = get_dlrm_model(sparse_dlrm=sparse_dlrm)
    model.load_state_dict(torch.load(unzip_path, map_location=device))
    model = model.to(device)
    model.eval()
```
- **EN**: Key callable entry points in this range include `fetch_model`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `fetch_model`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 171-175 / 第 171-175 行
```python
    # If there was a zip file, clean up the unzipped files
    if zipfile.is_zipfile(model_path):
        os.remove(unzip_path)

    return model
```
- **EN**: Key callable entry points in this range include `fetch_model`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段的重要可调用入口包括 `fetch_model`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

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

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`
- **Python standard library / Python 标准库**: `os`, `zipfile`
- **Third-party packages / 第三方包**: `numpy`, `dlrm_data_pytorch:collate_wrapper_criteo_offset`, `dlrm_data_pytorch:CriteoDataset`, `dlrm_s_pytorch:DLRM_Net`
- **Primary symbols / 核心符号**: `SparseDLRM`, `get_valid_name`, `get_dlrm_model`, `dlrm_wrap`, `make_test_data_loader`, `fetch_model`
