# linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/lora/linear.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `BaseLayerWithLoRA`, `VocabParallelEmbeddingWithLoRA`, and `ColumnParallelLinearWithLoRA`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `BaseLayerWithLoRA`、`VocabParallelEmbeddingWithLoRA` 和 `ColumnParallelLinearWithLoRA` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 5-34: module setup and imports / 模块初始化与导入
```python
import os

import torch
from torch import nn
from torch.distributed._composable.fsdp import (
    CPUOffloadPolicy,
    OffloadPolicy,
    fully_shard,
)
from torch.distributed.tensor import DTensor

from sglang.multimodal_gen.runtime.distributed import (
    get_local_torch_device,
    get_tp_rank,
    split_tensor_along_last_dim,
    tensor_model_parallel_all_gather,
    tensor_model_parallel_all_reduce,
)
from sglang.multimodal_gen.runtime.layers.linear import (
    ColumnParallelLinear,
    LinearBase,
    MergedColumnParallelLinear,
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from sglang.multimodal_gen.runtime.layers.vocab_parallel_embedding import (
    VocabParallelEmbedding,
)
from sglang.multimodal_gen.utils import get_mixed_precision_state
```
**EN:** This block establishes the module context and imports `os`, `torch`, `torch.distributed._composable.fsdp`, `torch.distributed.tensor`, `sglang.multimodal_gen.runtime.distributed`, and `sglang.multimodal_gen.runtime.layers.linear`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `os`、`torch`、`torch.distributed._composable.fsdp`、`torch.distributed.tensor`、`sglang.multimodal_gen.runtime.distributed` 和 `sglang.multimodal_gen.runtime.layers.linear`。这些依赖为后续实现提供所需符号。

### Lines 36-47: supporting statements / 辅助语句
```python
torch._dynamo.config.recompile_limit = 64


LORA_MERGE_CHUNK_BYTES = 32 * 1024 * 1024
LoRAWeightEntry = tuple[
    torch.nn.Parameter,
    torch.nn.Parameter,
    str | None,
    float,
    int | None,
    int | None,
]
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `LORA_MERGE_CHUNK_BYTES`, and `LoRAWeightEntry`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `LORA_MERGE_CHUNK_BYTES` 和 `LoRAWeightEntry` 等名称。

### Lines 50-50: `BaseLayerWithLoRA` class overview / `BaseLayerWithLoRA` 类概览
```python
class BaseLayerWithLoRA(nn.Module):
```
**EN:** This block defines class `BaseLayerWithLoRA`. It encapsulates base layer with lo ra behavior. It inherits from `nn.Module`.
**CN:** 该代码块定义了类 `BaseLayerWithLoRA`。 它用于封装 base layer with lo ra 相关行为。 它继承自 `nn.Module`。

### Lines 51-75: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        base_layer: nn.Module,
        lora_rank: int | None = None,
        lora_alpha: int | None = None,
    ):
        super().__init__()
        self.base_layer: nn.Module = base_layer

        self.merged: bool = False
        # Immutable base-weight snapshot; `to("cpu")` may alias CPU storage.
        # Use `clone()` so merge updates cannot mutate this backup tensor.
        self.cpu_weight = base_layer.weight.detach().to("cpu").clone()
        # indicates adapter weights don't contain this layer
        # (which shouldn't normally happen, but we want to separate it from the case of erroneous merging)
        # Default to True to prevent using uninitialized weights; set to False when weights are loaded
        self.disable_lora: bool = True
        self.lora_rank = lora_rank
        self.lora_alpha = lora_alpha
        self.lora_weights_list: list[LoRAWeightEntry] = []
        self.lora_path: str | None = None
        self.strength: float = 1.0

        self.lora_A = None
        self.lora_B = None
```
**EN:** This block defines method `__init__` on `BaseLayerWithLoRA`. It initializes the instance state. Key calls include `super.__init__`, `base_layer.weight.detach.to.clone`, `super`, `base_layer.weight.detach.to`, and `base_layer.weight.detach`. Parameters such as `base_layer`, `lora_rank`, and `lora_alpha` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseLayerWithLoRA` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`base_layer.weight.detach.to.clone`、`super`、`base_layer.weight.detach.to` 和 `base_layer.weight.detach`。 本段逻辑主要由 `base_layer`、`lora_rank` 和 `lora_alpha` 等参数驱动。

### Lines 77-79: `weight` implementation / `weight` 实现
```python
    @property
    def weight(self):
        return self.base_layer.weight
```
**EN:** This block defines method `weight` on `BaseLayerWithLoRA`. It handles weight logic.
**CN:** 该代码块定义了 `BaseLayerWithLoRA` 的方法 `weight`。 它用于处理 weight 相关逻辑。

### Lines 81-83: `bias` implementation / `bias` 实现
```python
    @property
    def bias(self):
        return getattr(self.base_layer, "bias", None)
```
**EN:** This block defines method `bias` on `BaseLayerWithLoRA`. It handles bias logic. Key calls include `getattr`.
**CN:** 该代码块定义了 `BaseLayerWithLoRA` 的方法 `bias`。 它用于处理 bias 相关逻辑。 关键调用包括 `getattr`。

### Lines 85-113: `forward` implementation / `forward` 实现
```python
    @torch.compile()
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        lora_A = self.lora_A
        lora_B = self.lora_B
        if isinstance(self.lora_B, DTensor):
            lora_B = self.lora_B.to_local()
            lora_A = self.lora_A.to_local()

        # TODO: Support multiple LoRA adapters when use not merged mode
        if not self.merged and not self.disable_lora:
            lora_dtype = lora_A.dtype
            x_lora = x.to(dtype=lora_dtype)
            lora_A_sliced = self.slice_lora_a_weights(
                lora_A.to(device=x.device, non_blocking=True)
            )
            lora_B_sliced = self.slice_lora_b_weights(
                lora_B.to(device=x.device, non_blocking=True)
            )
            delta = x_lora @ lora_A_sliced.T @ lora_B_sliced.T
            if self.lora_alpha != self.lora_rank:
                delta = delta * (
                    self.lora_alpha / self.lora_rank  # type: ignore
                )  # type: ignore
            delta = delta * self.strength
            out, output_bias = self.base_layer(x)
            return out + delta.to(dtype=out.dtype), output_bias
        else:
            out, output_bias = self.base_layer(x)
            return out, output_bias
```
**EN:** This block defines method `forward` on `BaseLayerWithLoRA`. It executes function. Key calls include `torch.compile`, `isinstance`, `self.lora_B.to_local`, `self.lora_A.to_local`, and `x.to`. The implementation branches on conditions. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseLayerWithLoRA` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `torch.compile`、`isinstance`、`self.lora_B.to_local`、`self.lora_A.to_local` 和 `x.to`。 实现中包含条件分支。 本段逻辑主要由 `x` 等参数驱动。

### Lines 115-116: `slice_lora_a_weights` implementation / `slice_lora_a_weights` 实现
```python
    def slice_lora_a_weights(self, A: torch.Tensor) -> torch.Tensor:
        return A
```
**EN:** This block defines method `slice_lora_a_weights` on `BaseLayerWithLoRA`. It handles slice lora a weights logic. Parameters such as `A` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseLayerWithLoRA` 的方法 `slice_lora_a_weights`。 它用于处理 slice lora a weights 相关逻辑。 本段逻辑主要由 `A` 等参数驱动。

### Lines 118-119: `slice_lora_b_weights` implementation / `slice_lora_b_weights` 实现
```python
    def slice_lora_b_weights(self, B: torch.Tensor) -> torch.Tensor:
        return B
```
**EN:** This block defines method `slice_lora_b_weights` on `BaseLayerWithLoRA`. It handles slice lora b weights logic. Parameters such as `B` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseLayerWithLoRA` 的方法 `slice_lora_b_weights`。 它用于处理 slice lora b weights 相关逻辑。 本段逻辑主要由 `B` 等参数驱动。

### Lines 121-177: `set_lora_weights` implementation / `set_lora_weights` 实现
```python
    def set_lora_weights(
        self,
        A: torch.Tensor,
        B: torch.Tensor,
        lora_path: str | None = None,
        strength: float = 1.0,
        clear_existing: bool = False,
        merge_weights: bool = True,
    ) -> None:
        """
        Set LoRA weights. Supports multiple LoRA adapters.

        Args:
            A: LoRA A weight tensor
            B: LoRA B weight tensor
            lora_path: Path to the LoRA adapter (for logging)
            strength: LoRA strength
            clear_existing: If True, clear existing LoRA weights before adding new one.
                          If False, append to existing list (for multi-LoRA support).
        """
        lora_A_param = torch.nn.Parameter(
            A
        )  # share storage with weights in the pipeline
        lora_B_param = torch.nn.Parameter(B)

        if clear_existing:
            self.lora_weights_list.clear()
            # Also clear backward compatibility attributes
            self.lora_A = None
            self.lora_B = None
            self.lora_path = None
            self.strength = 1.0

        # Add to list for multi-LoRA support
        self.lora_weights_list.append(
            (
                lora_A_param,
                lora_B_param,
                lora_path,
                strength,
                self.lora_rank,
                self.lora_alpha,
            )
        )

        # Set backward compatibility attributes to point to the last LoRA (for single LoRA case)
        # This ensures backward compatibility while supporting multiple LoRA
        self.lora_A = lora_A_param
        self.lora_B = lora_B_param
        self.lora_path = lora_path
        self.strength = strength

        self.disable_lora = False
        if merge_weights:
            self.merge_lora_weights()
        elif self.merged:
            self.unmerge_lora_weights()
```
**EN:** This block defines method `set_lora_weights` on `BaseLayerWithLoRA`. Set LoRA weights. Supports multiple LoRA adapters. Key calls include `torch.nn.Parameter`, `self.lora_weights_list.append`, `self.lora_weights_list.clear`, `self.merge_lora_weights`, and `self.unmerge_lora_weights`. The implementation branches on conditions. Parameters such as `A`, `B`, `lora_path`, `strength`, and `clear_existing` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseLayerWithLoRA` 的方法 `set_lora_weights`。 它用于设置lora weights。 关键调用包括 `torch.nn.Parameter`、`self.lora_weights_list.append`、`self.lora_weights_list.clear`、`self.merge_lora_weights` 和 `self.unmerge_lora_weights`。 实现中包含条件分支。 本段逻辑主要由 `A`、`B`、`lora_path`、`strength` 和 `clear_existing` 等参数驱动。

### Lines 179-235: `_merge_lora_into_data` implementation / `_merge_lora_into_data` 实现
```python
    @torch.no_grad()
    def _merge_lora_into_data(
        self,
        data: torch.Tensor,
        lora_list: list[LoRAWeightEntry],
    ) -> None:
        """
        Merge all LoRA adapters into the data tensor in-place.

        Args:
            data: The base weight tensor to merge LoRA into (modified in-place)
            lora_list: List of (lora_A, lora_B, lora_path, lora_strength, rank, alpha) tuples
        """
        # Merge all LoRA adapters in order
        for lora_A, lora_B, _, lora_strength, lora_rank, lora_alpha in lora_list:
            lora_A_sliced = self.slice_lora_a_weights(lora_A.to(data))
            lora_B_sliced = self.slice_lora_b_weights(lora_B.to(data))

            scale = lora_strength
            if (
                lora_alpha is not None
                and lora_rank is not None
                and lora_alpha != lora_rank
            ):
                scale *= lora_alpha / lora_rank

            if not isinstance(lora_B_sliced, torch.Tensor):
                lora_delta = lora_B_sliced @ lora_A_sliced
                if isinstance(lora_delta, torch.Tensor) and lora_delta.dim() > 2:
                    lora_delta = lora_delta.reshape(-1, lora_delta.shape[-1])
                data.add_(lora_delta, alpha=scale)
                continue

            if lora_A_sliced.dim() > 2 or lora_B_sliced.dim() > 2:
                lora_delta = lora_B_sliced @ lora_A_sliced
                if lora_delta.dim() > 2:
                    lora_delta = lora_delta.reshape(-1, lora_delta.shape[-1])
                data_2d = data.reshape(-1, data.shape[-1]) if data.dim() > 2 else data
                data_2d.add_(lora_delta, alpha=scale)
                continue

            data_2d = data.reshape(-1, data.shape[-1]) if data.dim() > 2 else data
            lora_B_2d = (
                lora_B_sliced.reshape(-1, lora_B_sliced.shape[-1])
                if lora_B_sliced.dim() > 2
                else lora_B_sliced
            )

            chunk_rows = max(
                1,
                LORA_MERGE_CHUNK_BYTES
                // (data_2d.shape[-1] * max(1, data_2d.element_size())),
            )
            for start in range(0, lora_B_2d.shape[0], chunk_rows):
                end = min(start + chunk_rows, lora_B_2d.shape[0])
                chunk_delta = lora_B_2d[start:end] @ lora_A_sliced
                data_2d[start:end].add_(chunk_delta, alpha=scale)
```
**EN:** This block defines method `_merge_lora_into_data` on `BaseLayerWithLoRA`. Merge all LoRA adapters into the data tensor in-place. Args: data: The base weight tensor to merge LoRA into (modified in-place) lora_list: List of (lora_A, lora_B, lora_path, lora_strength, rank, alpha) tuples Key calls include `torch.no_grad`, `self.slice_lora_a_weights`, `self.slice_lora_b_weights`, `max`, and `range`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `data`, and `lora_list` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseLayerWithLoRA` 的方法 `_merge_lora_into_data`。 它用于合并lora into data。 关键调用包括 `torch.no_grad`、`self.slice_lora_a_weights`、`self.slice_lora_b_weights`、`max` 和 `range`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `data` 和 `lora_list` 等参数驱动。

### Lines 237-246: `_should_merge_in_fp32` implementation / `_should_merge_in_fp32` 实现
```python
    def _should_merge_in_fp32(
        self,
        lora_list: list[LoRAWeightEntry],
    ) -> bool:
        if os.getenv("SGLANG_DIFFUSION_LORA_MERGE_FP32", "0") != "1":
            return False
        for _, _, lora_path, _, _, _ in lora_list:
            if lora_path and "distilled-lora" in lora_path.lower():
                return False
        return True
```
**EN:** This block defines method `_should_merge_in_fp32` on `BaseLayerWithLoRA`. It determines whether to merge in fp32. Key calls include `os.getenv`, and `lora_path.lower`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `lora_list` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseLayerWithLoRA` 的方法 `_should_merge_in_fp32`。 它用于判断是否merge in fp32。 关键调用包括 `os.getenv` 和 `lora_path.lower`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `lora_list` 等参数驱动。

### Lines 248-342: `merge_lora_weights` implementation / `merge_lora_weights` 实现
```python
    @torch.no_grad()
    def merge_lora_weights(self, strength: float | None = None) -> None:
        if strength is not None:
            self.strength = strength

        if self.disable_lora:
            return

        if self.merged:
            self.unmerge_lora_weights()

        # Use lora_weights_list if available, otherwise fall back to single LoRA for backward compatibility
        lora_list = self.lora_weights_list if self.lora_weights_list else []
        if not lora_list and self.lora_A is not None and self.lora_B is not None:
            lora_list = [
                (
                    self.lora_A,
                    self.lora_B,
                    self.lora_path,
                    self.strength,
                    self.lora_rank,
                    self.lora_alpha,
                )
            ]

        if not lora_list:
            raise ValueError("LoRA weights not set. Please set them first.")

        merge_in_fp32 = self._should_merge_in_fp32(lora_list)

        if isinstance(self.base_layer.weight, DTensor):
            mesh = self.base_layer.weight.data.device_mesh
            unsharded_base_layer = ReplicatedLinear(
                input_size=self.base_layer.input_size,
                output_size=self.base_layer.output_size,
                bias=getattr(self.base_layer, "bias", None) is not None,
                skip_bias_add=self.base_layer.skip_bias_add,
                params_dtype=self.base_layer.params_dtype,
                quant_config=self.base_layer.quant_config,
                prefix=self.base_layer.prefix,
            )
            # Using offload param is on CPU, so current_device is for "CPU -> GPU -> merge -> CPU"
            current_device = self.base_layer.weight.data.device
            data = self.base_layer.weight.data.to(
                get_local_torch_device()
            ).full_tensor()
            target_dtype = data.dtype
            if (
                merge_in_fp32
                and data.is_floating_point()
                and data.dtype != torch.float32
            ):
                data = data.to(torch.float32)

            self._merge_lora_into_data(data, lora_list)

            unsharded_base_layer.weight = nn.Parameter(
                data.to(current_device, dtype=target_dtype)
            )
            if isinstance(getattr(self.base_layer, "bias", None), DTensor):
                unsharded_base_layer.bias = nn.Parameter(
                    self.base_layer.bias.to(get_local_torch_device(), non_blocking=True)
                    .full_tensor()
                    .to(current_device)
                )

            offload_policy = (
                CPUOffloadPolicy() if "cpu" in str(current_device) else OffloadPolicy()
            )
            mp_policy = get_mixed_precision_state().mp_policy

            self.base_layer = fully_shard(
                unsharded_base_layer,
                mesh=mesh,
                mp_policy=mp_policy,
                offload_policy=offload_policy,
            )
        else:
            current_device = self.base_layer.weight.data.device
            data = self.base_layer.weight.data.to(get_local_torch_device())
            target_dtype = data.dtype
            if (
                merge_in_fp32
                and data.is_floating_point()
                and data.dtype != torch.float32
            ):
                data = data.to(torch.float32)

            self._merge_lora_into_data(data, lora_list)

            self.base_layer.weight.data = data.to(
                current_device, dtype=target_dtype, non_blocking=True
            )

        self.merged = True
```
**EN:** This block defines method `merge_lora_weights` on `BaseLayerWithLoRA`. It merges lora weights. Key calls include `torch.no_grad`, `self._should_merge_in_fp32`, `isinstance`, `self.unmerge_lora_weights`, and `ValueError`. The implementation branches on conditions. Parameters such as `strength` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseLayerWithLoRA` 的方法 `merge_lora_weights`。 它用于合并lora weights。 关键调用包括 `torch.no_grad`、`self._should_merge_in_fp32`、`isinstance`、`self.unmerge_lora_weights` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `strength` 等参数驱动。

### Lines 344-372: `unmerge_lora_weights` implementation / `unmerge_lora_weights` 实现
```python
    @torch.no_grad()
    # @torch.compile(dynamic=True)
    def unmerge_lora_weights(self) -> None:
        if self.disable_lora:
            return

        if not self.merged:
            raise ValueError(
                "LoRA weights not merged. Please merge them first before unmerging."
            )

        # avoid precision loss
        if isinstance(self.base_layer.weight, DTensor):
            device = self.base_layer.weight.data.device
            old_weight = self.base_layer.weight
            new_weight_data = self.cpu_weight.to(device, non_blocking=True)
            self.base_layer.weight = nn.Parameter(new_weight_data)
            del old_weight
        else:
            current_device = self.base_layer.weight.data.device
            cpu_weight_on_device = self.cpu_weight.to(current_device, non_blocking=True)
            self.base_layer.weight.data.copy_(cpu_weight_on_device)
            if (
                cpu_weight_on_device.data_ptr()
                != self.base_layer.weight.data.data_ptr()
            ):
                del cpu_weight_on_device

        self.merged = False
```
**EN:** This block defines method `unmerge_lora_weights` on `BaseLayerWithLoRA`. It handles unmerge lora weights logic. Key calls include `torch.no_grad`, `isinstance`, `ValueError`, `self.cpu_weight.to`, and `nn.Parameter`. The implementation branches on conditions.
**CN:** 该代码块定义了 `BaseLayerWithLoRA` 的方法 `unmerge_lora_weights`。 它用于处理 unmerge lora weights 相关逻辑。 关键调用包括 `torch.no_grad`、`isinstance`、`ValueError`、`self.cpu_weight.to` 和 `nn.Parameter`。 实现中包含条件分支。

### Lines 375-383: `VocabParallelEmbeddingWithLoRA` class overview / `VocabParallelEmbeddingWithLoRA` 类概览
```python
class VocabParallelEmbeddingWithLoRA(BaseLayerWithLoRA):
    """
    Vocab parallel embedding layer with support for LoRA (Low-Rank Adaptation).

    Note: The current version does not yet implement the LoRA functionality.
    This class behaves exactly the same as the base VocabParallelEmbedding.
    Future versions will integrate LoRA functionality to support efficient parameter fine-tuning.
    """
```
**EN:** This block defines class `VocabParallelEmbeddingWithLoRA`. Vocab parallel embedding layer with support for LoRA (Low-Rank Adaptation). Note: The current version does not yet implement the LoRA functionality. It inherits from `BaseLayerWithLoRA`.
**CN:** 该代码块定义了类 `VocabParallelEmbeddingWithLoRA`。 它用于封装 vocab parallel embedding with lo ra 相关行为。 它继承自 `BaseLayerWithLoRA`。

### Lines 384-388: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        base_layer: VocabParallelEmbedding,
    ) -> None:
        super().__init__(base_layer)
```
**EN:** This block defines method `__init__` on `VocabParallelEmbeddingWithLoRA`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `base_layer` drive the behavior in this section.
**CN:** 该代码块定义了 `VocabParallelEmbeddingWithLoRA` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `base_layer` 等参数驱动。

### Lines 390-393: `forward` implementation / `forward` 实现
```python
    def forward(self, input_: torch.Tensor) -> torch.Tensor:
        raise NotImplementedError(
            "We don't support VocabParallelEmbeddingWithLoRA yet."
        )
```
**EN:** This block defines method `forward` on `VocabParallelEmbeddingWithLoRA`. It executes function. Key calls include `NotImplementedError`. Parameters such as `input_` drive the behavior in this section.
**CN:** 该代码块定义了 `VocabParallelEmbeddingWithLoRA` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `NotImplementedError`。 本段逻辑主要由 `input_` 等参数驱动。

### Lines 396-396: `ColumnParallelLinearWithLoRA` class overview / `ColumnParallelLinearWithLoRA` 类概览
```python
class ColumnParallelLinearWithLoRA(BaseLayerWithLoRA):
```
**EN:** This block defines class `ColumnParallelLinearWithLoRA`. It encapsulates column parallel linear with lo ra behavior. It inherits from `BaseLayerWithLoRA`.
**CN:** 该代码块定义了类 `ColumnParallelLinearWithLoRA`。 它用于封装 column parallel linear with lo ra 相关行为。 它继承自 `BaseLayerWithLoRA`。

### Lines 397-403: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        base_layer: ColumnParallelLinear,
        lora_rank: int | None = None,
        lora_alpha: int | None = None,
    ) -> None:
        super().__init__(base_layer, lora_rank, lora_alpha)
```
**EN:** This block defines method `__init__` on `ColumnParallelLinearWithLoRA`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `base_layer`, `lora_rank`, and `lora_alpha` drive the behavior in this section.
**CN:** 该代码块定义了 `ColumnParallelLinearWithLoRA` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `base_layer`、`lora_rank` 和 `lora_alpha` 等参数驱动。

### Lines 405-442: `forward` implementation / `forward` 实现
```python
    def forward(self, input_: torch.Tensor) -> torch.Tensor:
        if self.merged or self.disable_lora:
            return self.base_layer(input_)

        lora_A = self.lora_A
        lora_B = self.lora_B
        if isinstance(self.lora_B, DTensor):
            lora_B = self.lora_B.to_local()
            lora_A = self.lora_A.to_local()

        bias = self.base_layer.bias if not self.base_layer.skip_bias_add else None
        output_parallel = self.base_layer.quant_method.apply(
            self.base_layer, input_, bias
        )
        if not self.merged and not self.disable_lora:
            lora_dtype = lora_A.dtype
            input_lora = input_.to(dtype=lora_dtype)
            lora_A_sliced = self.slice_lora_a_weights(
                lora_A.to(device=input_.device, non_blocking=True)
            )
            lora_B_sliced = self.slice_lora_b_weights(
                lora_B.to(device=input_.device, non_blocking=True)
            )
            delta_parallel = input_lora @ lora_A_sliced.T @ lora_B_sliced.T
            if self.lora_alpha != self.lora_rank:
                delta_parallel = delta_parallel * (
                    self.lora_alpha / self.lora_rank  # type: ignore
                )  # type: ignore
            delta_parallel = delta_parallel * self.strength
            output_parallel = output_parallel + delta_parallel.to(
                dtype=output_parallel.dtype
            )
        if self.base_layer.gather_output:
            output = tensor_model_parallel_all_gather(output_parallel)
        else:
            output = output_parallel
        output_bias = self.base_layer.bias if self.base_layer.skip_bias_add else None
        return output, output_bias
```
**EN:** This block defines method `forward` on `ColumnParallelLinearWithLoRA`. It executes function. Key calls include `isinstance`, `self.base_layer.quant_method.apply`, `self.base_layer`, `self.lora_B.to_local`, and `self.lora_A.to_local`. The implementation branches on conditions. Parameters such as `input_` drive the behavior in this section.
**CN:** 该代码块定义了 `ColumnParallelLinearWithLoRA` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `isinstance`、`self.base_layer.quant_method.apply`、`self.base_layer`、`self.lora_B.to_local` 和 `self.lora_A.to_local`。 实现中包含条件分支。 本段逻辑主要由 `input_` 等参数驱动。

### Lines 444-445: `slice_lora_a_weights` implementation / `slice_lora_a_weights` 实现
```python
    def slice_lora_a_weights(self, A: torch.Tensor) -> torch.Tensor:
        return A
```
**EN:** This block defines method `slice_lora_a_weights` on `ColumnParallelLinearWithLoRA`. It handles slice lora a weights logic. Parameters such as `A` drive the behavior in this section.
**CN:** 该代码块定义了 `ColumnParallelLinearWithLoRA` 的方法 `slice_lora_a_weights`。 它用于处理 slice lora a weights 相关逻辑。 本段逻辑主要由 `A` 等参数驱动。

### Lines 447-453: `slice_lora_b_weights` implementation / `slice_lora_b_weights` 实现
```python
    def slice_lora_b_weights(self, B: torch.Tensor) -> torch.Tensor:
        tp_rank = get_tp_rank()
        shard_size = self.base_layer.output_partition_sizes[0]
        start_idx = tp_rank * shard_size
        end_idx = (tp_rank + 1) * shard_size
        B = B[start_idx:end_idx, :]
        return B
```
**EN:** This block defines method `slice_lora_b_weights` on `ColumnParallelLinearWithLoRA`. It handles slice lora b weights logic. Key calls include `get_tp_rank`. Parameters such as `B` drive the behavior in this section.
**CN:** 该代码块定义了 `ColumnParallelLinearWithLoRA` 的方法 `slice_lora_b_weights`。 它用于处理 slice lora b weights 相关逻辑。 关键调用包括 `get_tp_rank`。 本段逻辑主要由 `B` 等参数驱动。

### Lines 456-456: `MergedColumnParallelLinearWithLoRA` class overview / `MergedColumnParallelLinearWithLoRA` 类概览
```python
class MergedColumnParallelLinearWithLoRA(ColumnParallelLinearWithLoRA):
```
**EN:** This block defines class `MergedColumnParallelLinearWithLoRA`. It encapsulates merged column parallel linear with lo ra behavior. It inherits from `ColumnParallelLinearWithLoRA`.
**CN:** 该代码块定义了类 `MergedColumnParallelLinearWithLoRA`。 它用于封装 merged column parallel linear with lo ra 相关行为。 它继承自 `ColumnParallelLinearWithLoRA`。

### Lines 457-463: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        base_layer: MergedColumnParallelLinear,
        lora_rank: int | None = None,
        lora_alpha: int | None = None,
    ) -> None:
        super().__init__(base_layer, lora_rank, lora_alpha)
```
**EN:** This block defines method `__init__` on `MergedColumnParallelLinearWithLoRA`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `base_layer`, `lora_rank`, and `lora_alpha` drive the behavior in this section.
**CN:** 该代码块定义了 `MergedColumnParallelLinearWithLoRA` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `base_layer`、`lora_rank` 和 `lora_alpha` 等参数驱动。

### Lines 465-466: `slice_lora_a_weights` implementation / `slice_lora_a_weights` 实现
```python
    def slice_lora_a_weights(self, A: torch.Tensor) -> torch.Tensor:
        return A
```
**EN:** This block defines method `slice_lora_a_weights` on `MergedColumnParallelLinearWithLoRA`. It handles slice lora a weights logic. Parameters such as `A` drive the behavior in this section.
**CN:** 该代码块定义了 `MergedColumnParallelLinearWithLoRA` 的方法 `slice_lora_a_weights`。 它用于处理 slice lora a weights 相关逻辑。 本段逻辑主要由 `A` 等参数驱动。

### Lines 468-474: `slice_lora_b_weights` implementation / `slice_lora_b_weights` 实现
```python
    def slice_lora_b_weights(self, B: torch.Tensor) -> torch.Tensor:
        tp_rank = get_tp_rank()
        # Since the outputs for both gate and up are identical, we use a random one.
        shard_size = self.base_layer.output_partition_sizes[0]
        start_idx = tp_rank * shard_size
        end_idx = (tp_rank + 1) * shard_size
        return B[:, start_idx:end_idx, :]
```
**EN:** This block defines method `slice_lora_b_weights` on `MergedColumnParallelLinearWithLoRA`. It handles slice lora b weights logic. Key calls include `get_tp_rank`. Parameters such as `B` drive the behavior in this section.
**CN:** 该代码块定义了 `MergedColumnParallelLinearWithLoRA` 的方法 `slice_lora_b_weights`。 它用于处理 slice lora b weights 相关逻辑。 关键调用包括 `get_tp_rank`。 本段逻辑主要由 `B` 等参数驱动。

### Lines 477-477: `QKVParallelLinearWithLoRA` class overview / `QKVParallelLinearWithLoRA` 类概览
```python
class QKVParallelLinearWithLoRA(ColumnParallelLinearWithLoRA):
```
**EN:** This block defines class `QKVParallelLinearWithLoRA`. It encapsulates qkvparallel linear with lo ra behavior. It inherits from `ColumnParallelLinearWithLoRA`.
**CN:** 该代码块定义了类 `QKVParallelLinearWithLoRA`。 它用于封装 qkvparallel linear with lo ra 相关行为。 它继承自 `ColumnParallelLinearWithLoRA`。

### Lines 478-484: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        base_layer: QKVParallelLinear,
        lora_rank: int | None = None,
        lora_alpha: int | None = None,
    ) -> None:
        super().__init__(base_layer, lora_rank, lora_alpha)
```
**EN:** This block defines method `__init__` on `QKVParallelLinearWithLoRA`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `base_layer`, `lora_rank`, and `lora_alpha` drive the behavior in this section.
**CN:** 该代码块定义了 `QKVParallelLinearWithLoRA` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `base_layer`、`lora_rank` 和 `lora_alpha` 等参数驱动。

### Lines 486-487: `slice_lora_a_weights` implementation / `slice_lora_a_weights` 实现
```python
    def slice_lora_a_weights(self, A: torch.Tensor) -> torch.Tensor:
        return A
```
**EN:** This block defines method `slice_lora_a_weights` on `QKVParallelLinearWithLoRA`. It handles slice lora a weights logic. Parameters such as `A` drive the behavior in this section.
**CN:** 该代码块定义了 `QKVParallelLinearWithLoRA` 的方法 `slice_lora_a_weights`。 它用于处理 slice lora a weights 相关逻辑。 本段逻辑主要由 `A` 等参数驱动。

### Lines 489-506: `slice_lora_b_weights` implementation / `slice_lora_b_weights` 实现
```python
    def slice_lora_b_weights(
        self, B: list[torch.Tensor]
    ) -> tuple[torch.Tensor, torch.Tensor]:
        tp_rank = get_tp_rank()
        B_q, B_kv = B
        base_layer = self.base_layer
        q_proj_shard_size = base_layer.q_proj_shard_size
        kv_proj_shard_size = base_layer.kv_proj_shard_size
        num_kv_head_replicas = base_layer.num_kv_head_replicas

        q_start_idx = q_proj_shard_size * tp_rank
        q_end_idx = q_start_idx + q_proj_shard_size

        kv_shard_id = tp_rank // num_kv_head_replicas
        kv_start_idx = kv_proj_shard_size * kv_shard_id
        kv_end_idx = kv_start_idx + kv_proj_shard_size

        return B_q[q_start_idx:q_end_idx, :], B_kv[:, kv_start_idx:kv_end_idx, :]
```
**EN:** This block defines method `slice_lora_b_weights` on `QKVParallelLinearWithLoRA`. It handles slice lora b weights logic. Key calls include `get_tp_rank`. Parameters such as `B` drive the behavior in this section.
**CN:** 该代码块定义了 `QKVParallelLinearWithLoRA` 的方法 `slice_lora_b_weights`。 它用于处理 slice lora b weights 相关逻辑。 关键调用包括 `get_tp_rank`。 本段逻辑主要由 `B` 等参数驱动。

### Lines 509-509: `RowParallelLinearWithLoRA` class overview / `RowParallelLinearWithLoRA` 类概览
```python
class RowParallelLinearWithLoRA(BaseLayerWithLoRA):
```
**EN:** This block defines class `RowParallelLinearWithLoRA`. It encapsulates row parallel linear with lo ra behavior. It inherits from `BaseLayerWithLoRA`.
**CN:** 该代码块定义了类 `RowParallelLinearWithLoRA`。 它用于封装 row parallel linear with lo ra 相关行为。 它继承自 `BaseLayerWithLoRA`。

### Lines 510-516: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        base_layer: RowParallelLinear,
        lora_rank: int | None = None,
        lora_alpha: int | None = None,
    ) -> None:
        super().__init__(base_layer, lora_rank, lora_alpha)
```
**EN:** This block defines method `__init__` on `RowParallelLinearWithLoRA`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `base_layer`, `lora_rank`, and `lora_alpha` drive the behavior in this section.
**CN:** 该代码块定义了 `RowParallelLinearWithLoRA` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `base_layer`、`lora_rank` 和 `lora_alpha` 等参数驱动。

### Lines 518-573: `forward` implementation / `forward` 实现
```python
    def forward(self, input_: torch.Tensor):
        if self.merged or self.disable_lora:
            return self.base_layer(input_)

        lora_A = self.lora_A
        lora_B = self.lora_B
        if isinstance(self.lora_B, DTensor):
            lora_B = self.lora_B.to_local()
            lora_A = self.lora_A.to_local()

        if self.base_layer.input_is_parallel:
            input_parallel = input_
        else:
            tp_rank = get_tp_rank()
            splitted_input = split_tensor_along_last_dim(
                input_, num_partitions=self.base_layer.tp_size
            )
            input_parallel = splitted_input[tp_rank].contiguous()
        output_parallel = self.base_layer.quant_method.apply(
            self.base_layer, input_parallel
        )
        if not self.merged and not self.disable_lora:
            lora_dtype = lora_A.dtype
            input_parallel_lora = input_parallel.to(dtype=lora_dtype)
            lora_A_sliced = self.slice_lora_a_weights(
                lora_A.to(device=input_parallel.device, non_blocking=True)
            )
            lora_B_sliced = self.slice_lora_b_weights(
                lora_B.to(device=input_parallel.device, non_blocking=True)
            )
            delta_parallel = input_parallel_lora @ lora_A_sliced.T @ lora_B_sliced.T
            if self.lora_alpha != self.lora_rank:
                delta_parallel = delta_parallel * (
                    self.lora_alpha / self.lora_rank  # type: ignore
                )  # type: ignore
            delta_parallel = delta_parallel * self.strength
            output_parallel = output_parallel + delta_parallel.to(
                dtype=output_parallel.dtype
            )

        if self.base_layer.reduce_results and self.base_layer.tp_size > 1:
            output_ = tensor_model_parallel_all_reduce(output_parallel)
        else:
            output_ = output_parallel

        if not self.base_layer.skip_bias_add:
            output = (
                output_ + self.base_layer.bias
                if self.base_layer.bias is not None
                else output_
            )
            output_bias = None
        else:
            output = output_
            output_bias = self.base_layer.bias
        return output, output_bias
```
**EN:** This block defines method `forward` on `RowParallelLinearWithLoRA`. It executes function. Key calls include `isinstance`, `self.base_layer.quant_method.apply`, `self.base_layer`, `self.lora_B.to_local`, and `self.lora_A.to_local`. The implementation branches on conditions. Parameters such as `input_` drive the behavior in this section.
**CN:** 该代码块定义了 `RowParallelLinearWithLoRA` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `isinstance`、`self.base_layer.quant_method.apply`、`self.base_layer`、`self.lora_B.to_local` 和 `self.lora_A.to_local`。 实现中包含条件分支。 本段逻辑主要由 `input_` 等参数驱动。

### Lines 575-581: `slice_lora_a_weights` implementation / `slice_lora_a_weights` 实现
```python
    def slice_lora_a_weights(self, A: torch.Tensor) -> torch.Tensor:
        tp_rank = get_tp_rank()
        shard_size = self.base_layer.input_size_per_partition
        start_idx = tp_rank * shard_size
        end_idx = (tp_rank + 1) * shard_size
        A = A[:, start_idx:end_idx].contiguous()
        return A
```
**EN:** This block defines method `slice_lora_a_weights` on `RowParallelLinearWithLoRA`. It handles slice lora a weights logic. Key calls include `get_tp_rank`, and `A.contiguous`. Parameters such as `A` drive the behavior in this section.
**CN:** 该代码块定义了 `RowParallelLinearWithLoRA` 的方法 `slice_lora_a_weights`。 它用于处理 slice lora a weights 相关逻辑。 关键调用包括 `get_tp_rank` 和 `A.contiguous`。 本段逻辑主要由 `A` 等参数驱动。

### Lines 583-584: `slice_lora_b_weights` implementation / `slice_lora_b_weights` 实现
```python
    def slice_lora_b_weights(self, B: torch.Tensor) -> torch.Tensor:
        return B
```
**EN:** This block defines method `slice_lora_b_weights` on `RowParallelLinearWithLoRA`. It handles slice lora b weights logic. Parameters such as `B` drive the behavior in this section.
**CN:** 该代码块定义了 `RowParallelLinearWithLoRA` 的方法 `slice_lora_b_weights`。 它用于处理 slice lora b weights 相关逻辑。 本段逻辑主要由 `B` 等参数驱动。

### Lines 587-593: `LinearWithLoRA` class overview / `LinearWithLoRA` 类概览
```python
class LinearWithLoRA(BaseLayerWithLoRA):
    """
    Wrapper for standard torch.nn.Linear to support LoRA.
    Unlike custom LinearBase classes, nn.Linear.forward() returns a single tensor,
    not a tuple of (output, bias).
    """
```
**EN:** This block defines class `LinearWithLoRA`. Wrapper for standard torch.nn.Linear to support LoRA. Unlike custom LinearBase classes, nn.Linear.forward() returns a single tensor, not a tuple of (output, bias). It inherits from `BaseLayerWithLoRA`.
**CN:** 该代码块定义了类 `LinearWithLoRA`。 它用于封装 linear with lo ra 相关行为。 它继承自 `BaseLayerWithLoRA`。

### Lines 594-600: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        base_layer: nn.Linear,
        lora_rank: int | None = None,
        lora_alpha: int | None = None,
    ) -> None:
        super().__init__(base_layer, lora_rank, lora_alpha)
```
**EN:** This block defines method `__init__` on `LinearWithLoRA`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `base_layer`, `lora_rank`, and `lora_alpha` drive the behavior in this section.
**CN:** 该代码块定义了 `LinearWithLoRA` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `base_layer`、`lora_rank` 和 `lora_alpha` 等参数驱动。

### Lines 602-632: `forward` implementation / `forward` 实现
```python
    @torch.compile()
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        lora_A = self.lora_A
        lora_B = self.lora_B
        if isinstance(self.lora_B, DTensor):
            lora_B = self.lora_B.to_local()
            lora_A = self.lora_A.to_local()

        # TODO: Support multiple LoRA adapters when use not merged mode
        if not self.merged and not self.disable_lora:
            lora_dtype = lora_A.dtype
            x_lora = x.to(dtype=lora_dtype)
            lora_A_sliced = self.slice_lora_a_weights(
                lora_A.to(device=x.device, non_blocking=True)
            )
            lora_B_sliced = self.slice_lora_b_weights(
                lora_B.to(device=x.device, non_blocking=True)
            )
            delta = x_lora @ lora_A_sliced.T @ lora_B_sliced.T
            if self.lora_alpha != self.lora_rank:
                delta = delta * (
                    self.lora_alpha / self.lora_rank  # type: ignore
                )  # type: ignore
            delta = delta * self.strength
            # nn.Linear.forward() returns a single tensor, not a tuple
            out = self.base_layer(x)
            return out + delta.to(dtype=out.dtype)
        else:
            # nn.Linear.forward() returns a single tensor
            out = self.base_layer(x)
            return out
```
**EN:** This block defines method `forward` on `LinearWithLoRA`. It executes function. Key calls include `torch.compile`, `isinstance`, `self.lora_B.to_local`, `self.lora_A.to_local`, and `x.to`. The implementation branches on conditions. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `LinearWithLoRA` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `torch.compile`、`isinstance`、`self.lora_B.to_local`、`self.lora_A.to_local` 和 `x.to`。 实现中包含条件分支。 本段逻辑主要由 `x` 等参数驱动。

### Lines 635-663: `wrap_with_lora_layer` implementation / `wrap_with_lora_layer` 实现
```python
def wrap_with_lora_layer(
    layer: nn.Module,
    lora_rank: int | None = None,
    lora_alpha: int | None = None,
) -> BaseLayerWithLoRA | None:
    """
    transform the given layer to its corresponding LoRA layer
    """
    supported_layer_types: dict[
        type[LinearBase] | type[nn.Linear], type[BaseLayerWithLoRA]
    ] = {
        # the order matters
        # VocabParallelEmbedding: VocabParallelEmbeddingWithLoRA,
        QKVParallelLinear: QKVParallelLinearWithLoRA,
        MergedColumnParallelLinear: MergedColumnParallelLinearWithLoRA,
        ColumnParallelLinear: ColumnParallelLinearWithLoRA,
        RowParallelLinear: RowParallelLinearWithLoRA,
        ReplicatedLinear: BaseLayerWithLoRA,
        nn.Linear: LinearWithLoRA,
    }
    for src_layer_type, lora_layer_type in supported_layer_types.items():
        if isinstance(layer, src_layer_type):  # type: ignore[arg-type]
            ret = lora_layer_type(
                layer,
                lora_rank=lora_rank,
                lora_alpha=lora_alpha,
            )
            return ret
    return None
```
**EN:** This block defines function `wrap_with_lora_layer`. transform the given layer to its corresponding LoRA layer Key calls include `supported_layer_types.items`, `isinstance`, and `lora_layer_type`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `layer`, `lora_rank`, and `lora_alpha` drive the behavior in this section.
**CN:** 该代码块定义了函数 `wrap_with_lora_layer`。 它用于处理 wrap with lora layer 相关逻辑。 关键调用包括 `supported_layer_types.items`、`isinstance` 和 `lora_layer_type`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `layer`、`lora_rank` 和 `lora_alpha` 等参数驱动。

### Lines 667-674: `replace_submodule` implementation / `replace_submodule` 实现
```python
def replace_submodule(
    model: nn.Module, module_name: str, new_module: nn.Module
) -> nn.Module:
    """Replace a submodule in a model with a new module."""
    parent = model.get_submodule(".".join(module_name.split(".")[:-1]))
    target_name = module_name.split(".")[-1]
    setattr(parent, target_name, new_module)
    return new_module
```
**EN:** This block defines function `replace_submodule`. Replace a submodule in a model with a new module. Key calls include `model.get_submodule`, `setattr`, `join`, and `module_name.split`. Parameters such as `model`, `module_name`, and `new_module` drive the behavior in this section.
**CN:** 该代码块定义了函数 `replace_submodule`。 它用于处理 replace submodule 相关逻辑。 关键调用包括 `model.get_submodule`、`setattr`、`join` 和 `module_name.split`。 本段逻辑主要由 `model`、`module_name` 和 `new_module` 等参数驱动。

## Key Concepts / 关键概念
- `BaseLayerWithLoRA`: Primary class that encapsulates base layer with lo ra behavior. / 核心类，用于封装 base layer with lo ra 相关行为。
- `VocabParallelEmbeddingWithLoRA`: Vocab parallel embedding layer with support for LoRA (Low-Rank Adaptation). / 核心类，用于封装 vocab parallel embedding with lo ra 相关行为。
- `ColumnParallelLinearWithLoRA`: Primary class that encapsulates column parallel linear with lo ra behavior. / 核心类，用于封装 column parallel linear with lo ra 相关行为。
- `MergedColumnParallelLinearWithLoRA`: Primary class that encapsulates merged column parallel linear with lo ra behavior. / 核心类，用于封装 merged column parallel linear with lo ra 相关行为。
- `QKVParallelLinearWithLoRA`: Primary class that encapsulates qkvparallel linear with lo ra behavior. / 核心类，用于封装 qkvparallel linear with lo ra 相关行为。
- `RowParallelLinearWithLoRA`: Primary class that encapsulates row parallel linear with lo ra behavior. / 核心类，用于封装 row parallel linear with lo ra 相关行为。
- `LinearWithLoRA`: Wrapper for standard torch.nn.Linear to support LoRA. / 核心类，用于封装 linear with lo ra 相关行为。
- `wrap_with_lora_layer`: transform the given layer to its corresponding LoRA layer / 顶层函数，用于处理 wrap with lora layer 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed._composable.fsdp`, `torch.distributed.tensor`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.vocab_parallel_embedding`, `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 674
