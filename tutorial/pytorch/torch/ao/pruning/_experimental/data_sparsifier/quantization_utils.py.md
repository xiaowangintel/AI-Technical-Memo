# quantization_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/pruning/_experimental/data_sparsifier/quantization_utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure. This specific file centers on `quantization_utils.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。 该文件具体围绕 `quantization_utils.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
```python
# mypy: allow-untyped-defs

import torch
import torch.nn as nn
from torch.ao.pruning.sparsifier.utils import fqn_to_module, module_to_fqn


SUPPORTED_MODULES = {nn.Embedding, nn.EmbeddingBag}


def _fetch_all_embeddings(model):
    """Fetches Embedding and EmbeddingBag modules from the model"""
    embedding_modules = []
    stack = [model]
    while stack:
        module = stack.pop()
        for _, child in module.named_children():
            fqn_name = module_to_fqn(model, child)
```
- **EN**: Key callable entry points in this range include `_fetch_all_embeddings`. They package a focused unit of behavior behind named helpers or APIs. Named constants such as `SUPPORTED_MODULES` centralize shared configuration or sentinel values. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_fetch_all_embeddings`，它们把聚焦的行为封装成具名辅助函数或 API。 `SUPPORTED_MODULES` 等具名常量把共享配置或哨兵值集中定义在一起。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 19-34 / 第 19-34 行
```python
            if type(child) in SUPPORTED_MODULES:
                embedding_modules.append((fqn_name, child))
            else:
                stack.append(child)
    return embedding_modules


def post_training_sparse_quantize(
    model,
    data_sparsifier_class,
    sparsify_first=True,
    select_embeddings: list[nn.Module] | None = None,
    **sparse_config,
):
    """Takes in a model and applies sparsification and quantization to only embeddings & embeddingbags.
    The quantization step can happen before or after sparsification depending on the `sparsify_first` argument.
```
- **EN**: Key callable entry points in this range include `_fetch_all_embeddings`, `post_training_sparse_quantize`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `_fetch_all_embeddings`, `post_training_sparse_quantize`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 36-48 / 第 36-48 行
```python
    Args:
        - model (nn.Module)
            model whose embeddings needs to be sparsified
        - data_sparsifier_class (type of data sparsifier)
            Type of sparsification that needs to be applied to model
        - sparsify_first (bool)
            if true, sparsifies first and then quantizes
            otherwise, quantizes first and then sparsifies.
        - select_embeddings (List of Embedding modules)
            List of embedding modules to in the model to be sparsified & quantized.
            If None, all embedding modules with be sparsified
        - sparse_config (Dict)
            config that will be passed to the constructor of data sparsifier object.
```
- **EN**: Key callable entry points in this range include `post_training_sparse_quantize`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `post_training_sparse_quantize`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 50-64 / 第 50-64 行
```python
    Note:
        1. When `sparsify_first=False`, quantization occurs first followed by sparsification.
            - before sparsifying, the embedding layers are dequantized.
            - scales and zero-points are saved
            - embedding layers are sparsified and `squash_mask` is applied
            - embedding weights are requantized using the saved scales and zero-points
        2. When `sparsify_first=True`, sparsification occurs first followed by quantization.
            - embeddings are sparsified first
            - quantization is applied on the sparsified embeddings
    """
    data_sparsifier = data_sparsifier_class(**sparse_config)

    # if select_embeddings is None, perform it on all embeddings
    if select_embeddings is None:
        embedding_modules = _fetch_all_embeddings(model)
```
- **EN**: Key callable entry points in this range include `post_training_sparse_quantize`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `post_training_sparse_quantize`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 66-82 / 第 66-82 行
```python
    else:
        embedding_modules = []
        if not isinstance(select_embeddings, list):
            raise AssertionError(
                "the embedding_modules must be a list of embedding modules"
            )
        for emb in select_embeddings:
            if type(emb) not in SUPPORTED_MODULES:
                raise AssertionError(
                    "the embedding_modules list must be an embedding or embedding bags"
                )
            fqn_name = module_to_fqn(model, emb)
            if fqn_name is None:
                raise AssertionError(
                    "the embedding modules must be part of input model"
                )
            embedding_modules.append((fqn_name, emb))
```
- **EN**: Key callable entry points in this range include `post_training_sparse_quantize`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `post_training_sparse_quantize`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 84-99 / 第 84-99 行
```python
    if sparsify_first:
        # sparsify
        for name, emb_module in embedding_modules:
            valid_name = name.replace(".", "_")
            data_sparsifier.add_data(name=valid_name, data=emb_module)

        data_sparsifier.step()
        data_sparsifier.squash_mask()

        # quantize
        for _, emb_module in embedding_modules:
            # pyrefly: ignore [bad-argument-type]
            emb_module.qconfig = torch.ao.quantization.float_qparams_weight_only_qconfig

        torch.ao.quantization.prepare(model, inplace=True)
        torch.ao.quantization.convert(model, inplace=True)
```
- **EN**: Key callable entry points in this range include `post_training_sparse_quantize`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `post_training_sparse_quantize`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 101-117 / 第 101-117 行
```python
    else:
        # quantize
        for _, emb_module in embedding_modules:
            # pyrefly: ignore [bad-argument-type]
            emb_module.qconfig = torch.ao.quantization.float_qparams_weight_only_qconfig

        torch.ao.quantization.prepare(model, inplace=True)
        torch.ao.quantization.convert(model, inplace=True)

        # retrieve scale & zero_points
        quantize_params: dict[str, dict] = {
            "scales": {},
            "zero_points": {},
            "dequant_weights": {},
            "axis": {},
            "dtype": {},
        }
```
- **EN**: Key callable entry points in this range include `post_training_sparse_quantize`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `post_training_sparse_quantize`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 119-133 / 第 119-133 行
```python
        for name, _ in embedding_modules:
            quantized_emb = fqn_to_module(model, name)
            if quantized_emb is None:
                raise AssertionError(f"quantized embedding {name} not found in model")

            quantized_weight = quantized_emb.weight()  # type: ignore[operator]
            quantize_params["scales"][name] = quantized_weight.q_per_channel_scales()
            quantize_params["zero_points"][name] = (
                quantized_weight.q_per_channel_zero_points()
            )
            quantize_params["dequant_weights"][name] = torch.dequantize(
                quantized_weight
            )
            quantize_params["axis"][name] = quantized_weight.q_per_channel_axis()
            quantize_params["dtype"][name] = quantized_weight.dtype
```
- **EN**: Key callable entry points in this range include `post_training_sparse_quantize`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `post_training_sparse_quantize`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 135-152 / 第 135-152 行
```python
            # attach data to sparsifier
            data_sparsifier.add_data(
                name=name.replace(".", "_"),
                data=quantize_params["dequant_weights"][name],
            )

        data_sparsifier.step()
        data_sparsifier.squash_mask()

        for name, _ in embedding_modules:
            quantized_emb = fqn_to_module(model, name)
            if quantized_emb is None:
                raise AssertionError(f"quantized embedding {name} not found in model")
            requantized_vector = torch.quantize_per_channel(
                quantize_params["dequant_weights"][name],
                scales=quantize_params["scales"][name],
                zero_points=quantize_params["zero_points"][name],
                dtype=quantize_params["dtype"][name],
```
- **EN**: Key callable entry points in this range include `post_training_sparse_quantize`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `post_training_sparse_quantize`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 153-156 / 第 153-156 行
```python
                axis=quantize_params["axis"][name],
            )

            quantized_emb.set_weight(requantized_vector)  # type: ignore[operator]
```
- **EN**: Key callable entry points in this range include `post_training_sparse_quantize`. They package a focused unit of behavior behind named helpers or APIs. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `post_training_sparse_quantize`，它们把聚焦的行为封装成具名辅助函数或 API。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure.
  - CN: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Fake quantization**
  - EN: Simulates quantized numerics during training while keeping tensors in differentiable forms.
  - CN: 在训练阶段模拟量化数值行为，同时保持张量处于可求导形式。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Pruning and sparsity**
  - EN: Controls parameter masking, sparse layouts, or pruning schedules that reduce effective model size.
  - CN: 控制参数掩码、稀疏布局或剪枝计划，以降低模型的有效规模。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.nn`, `torch.ao.pruning.sparsifier.utils:fqn_to_module`, `torch.ao.pruning.sparsifier.utils:module_to_fqn`
- **Primary symbols / 核心符号**: `_fetch_all_embeddings`, `post_training_sparse_quantize`
