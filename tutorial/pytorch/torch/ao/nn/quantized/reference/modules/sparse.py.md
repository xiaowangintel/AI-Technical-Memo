# sparse.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/nn/quantized/reference/modules/sparse.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants. This specific file centers on `sparse.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior.
- **Purpose (CN) / 用途（中文）**: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。 该文件具体围绕 `sparse.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。
## Line-by-Line Analysis / 逐行分析

### Lines 1-11 / 第 1-11 行
```python
# mypy: allow-untyped-defs
from typing import Any

import torch.nn as nn
import torch.nn.functional as F
from torch import Tensor

from .utils import ReferenceQuantizedModule


__all__ = ["Embedding", "EmbeddingBag"]
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch.nn, torch.nn.functional, torch:Tensor, .utils:ReferenceQuantizedModule; standard-library helpers such as typing:Any. The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch.nn, torch.nn.functional, torch:Tensor, .utils:ReferenceQuantizedModule；标准库辅助模块，如 typing:Any。 `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 14-31 / 第 14-31 行
```python
class Embedding(nn.Embedding, ReferenceQuantizedModule):
    """A reference quantized Embedding module that fits into the
    FX Graph Mode Quantization workflow, activation will be floating point Tensor,
    we will store floating point weight as well in the module, but in forward we'll
    quantize and dequantize the weight before running the floating point functional
    embedding operator.
    """

    def __init__(
        self,
        num_embeddings: int,
        embedding_dim: int,
        padding_idx: int | None = None,
        max_norm: float | None = None,
        norm_type: float = 2.0,
        scale_grad_by_freq: bool = False,
        sparse: bool = False,
        _weight: Tensor | None = None,
```
- **EN**: It introduces or extends class-level abstractions such as `Embedding`, which organize state and behavior for this subsystem. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Embedding` 等类级抽象，用于组织该子系统的状态与行为。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 32-49 / 第 32-49 行
```python
        device=None,
        dtype=None,
        weight_qparams: dict[str, Any] | None = None,
    ) -> None:
        super().__init__(
            num_embeddings,
            embedding_dim,
            padding_idx,
            max_norm,
            norm_type,
            scale_grad_by_freq,
            sparse,
            _weight,
            # pyrefly: ignore [bad-argument-type]
            device,
            dtype,
        )
        self._init_weight_qparams(weight_qparams, device)
```
- **EN**: It introduces or extends class-level abstractions such as `Embedding`, which organize state and behavior for this subsystem. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Embedding` 等类级抽象，用于组织该子系统的状态与行为。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 51-64 / 第 51-64 行
```python
    def _get_name(self):
        return "QuantizedEmbedding(Reference)"

    def forward(self, input: Tensor) -> Tensor:
        weight_quant_dequant = self.get_weight()
        return F.embedding(
            input,
            weight_quant_dequant,
            self.padding_idx,
            self.max_norm,
            self.norm_type,
            self.scale_grad_by_freq,
            self.sparse,
        )
```
- **EN**: It introduces or extends class-level abstractions such as `Embedding`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Embedding` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 66-80 / 第 66-80 行
```python
    @classmethod
    def from_float(cls, mod, weight_qparams):
        return cls(
            mod.num_embeddings,
            mod.embedding_dim,
            mod.padding_idx,
            mod.max_norm,
            mod.norm_type,
            mod.scale_grad_by_freq,
            mod.sparse,
            mod.weight,
            mod.weight.device,
            mod.weight.dtype,
            weight_qparams,
        )
```
- **EN**: It introduces or extends class-level abstractions such as `Embedding`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `Embedding` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 83-100 / 第 83-100 行
```python
class EmbeddingBag(nn.EmbeddingBag, ReferenceQuantizedModule):
    """A reference quantized EmbeddingBag module that fits into the
    FX Graph Mode Quantization workflow, activation will be floating point Tensor,
    we will store floating point weight as well in the module, but in forward we'll
    quantize and dequantize the weight before running the floating point functional
    embedding operator.
    """

    def __init__(
        self,
        num_embeddings: int,
        embedding_dim: int,
        max_norm: float | None = None,
        norm_type: float = 2.0,
        scale_grad_by_freq: bool = False,
        mode: str = "mean",
        sparse: bool = False,
        _weight: Tensor | None = None,
```
- **EN**: It introduces or extends class-level abstractions such as `EmbeddingBag`, which organize state and behavior for this subsystem. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `EmbeddingBag` 等类级抽象，用于组织该子系统的状态与行为。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 101-118 / 第 101-118 行
```python
        include_last_offset: bool = False,
        padding_idx: int | None = None,
        device=None,
        dtype=None,
        weight_qparams: dict[str, Any] | None = None,
    ) -> None:
        super().__init__(
            num_embeddings,
            embedding_dim,
            max_norm,
            norm_type,
            scale_grad_by_freq,
            mode,
            sparse,
            _weight,
            include_last_offset,
            padding_idx,
            device,
```
- **EN**: It introduces or extends class-level abstractions such as `EmbeddingBag`, which organize state and behavior for this subsystem. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `EmbeddingBag` 等类级抽象，用于组织该子系统的状态与行为。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 119-136 / 第 119-136 行
```python
            dtype,
        )
        self._init_weight_qparams(weight_qparams, device)

    def _get_name(self):
        return "QuantizedEmbedding(Reference)"

    def forward(
        self,
        input: Tensor,
        offsets: Tensor | None = None,
        per_sample_weights: Tensor | None = None,
    ) -> Tensor:
        weight_quant_dequant = self.get_weight()
        return F.embedding_bag(
            input,
            weight_quant_dequant,
            offsets,
```
- **EN**: It introduces or extends class-level abstractions such as `EmbeddingBag`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `EmbeddingBag` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 137-154 / 第 137-154 行
```python
            self.max_norm,
            self.norm_type,
            self.scale_grad_by_freq,
            self.mode,
            self.sparse,
            per_sample_weights,
            self.include_last_offset,
            self.padding_idx,
        )

    @classmethod
    def from_float(cls, mod, weight_qparams, use_precomputed_fake_quant=False):
        return cls(
            mod.num_embeddings,
            mod.embedding_dim,
            mod.max_norm,
            mod.norm_type,
            mod.scale_grad_by_freq,
```
- **EN**: It introduces or extends class-level abstractions such as `EmbeddingBag`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `EmbeddingBag` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 155-163 / 第 155-163 行
```python
            mod.mode,
            mod.sparse,
            mod.weight,
            mod.include_last_offset,
            mod.padding_idx,
            mod.weight.device,
            mod.weight.dtype,
            weight_qparams,
        )
```
- **EN**: It introduces or extends class-level abstractions such as `EmbeddingBag`, which organize state and behavior for this subsystem. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `EmbeddingBag` 等类级抽象，用于组织该子系统的状态与行为。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements quantization-aware neural-network module wrappers, fused building blocks, and reference/quantized module variants.
  - CN: 实现面向量化的神经网络模块包装、融合构件以及参考/量化模块变体。
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
- **Embedding**
  - EN: `Embedding` is one of the main classes that structures the file's behavior.
  - CN: `Embedding` 是组织该文件行为的核心类之一。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch.nn`, `torch.nn.functional`, `torch:Tensor`, `.utils:ReferenceQuantizedModule`
- **Python standard library / Python 标准库**: `typing:Any`
- **Explicit exports / 显式导出**: `Embedding`, `EmbeddingBag`
- **Primary symbols / 核心符号**: `Embedding`, `EmbeddingBag`
