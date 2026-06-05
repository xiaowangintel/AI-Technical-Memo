# aux_plugins.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/token_aligner/smart/aux_plugins.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on heuristic token alignment. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于启发式 token 对齐。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

from abc import ABC, abstractmethod
from typing import Optional

import torch

from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    PositionalSeqId,
    SeqId,
    SGLangSeqId,
    TokenAlignerStepAux,
)
from sglang.srt.debug_utils.comparator.dims_spec import TokenLayout
from sglang.srt.debug_utils.comparator.log_sink import log_sink
from sglang.srt.debug_utils.comparator.output_types import InfoLog
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 21-23: Define class `_AuxFrameworkPlugin` and class context / 定义类 `_AuxFrameworkPlugin`及类上下文
```python
class _AuxFrameworkPlugin(ABC):
    @property
    @abstractmethod
```
**EN:** This section introduces `_AuxFrameworkPlugin`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_AuxFrameworkPlugin`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 24-24: Implement method `name` for `_AuxFrameworkPlugin` / 为 `_AuxFrameworkPlugin` 实现方法 `name`
```python
    def name(self) -> str: ...
```
**EN:** Method `name` implements behavior on `_AuxFrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `name` 为 `_AuxFrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 28-28: Implement method `tensor_names` for `_AuxFrameworkPlugin` / 为 `_AuxFrameworkPlugin` 实现方法 `tensor_names`
```python
    def tensor_names(self) -> frozenset[str]: ...
```
**EN:** Method `tensor_names` implements behavior on `_AuxFrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `tensor_names` 为 `_AuxFrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 32-32: Implement method `non_tensor_names` for `_AuxFrameworkPlugin` / 为 `_AuxFrameworkPlugin` 实现方法 `non_tensor_names`
```python
    def non_tensor_names(self) -> frozenset[str]: ...
```
**EN:** Method `non_tensor_names` implements behavior on `_AuxFrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `non_tensor_names` 为 `_AuxFrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 35-36: Implement method `cp_sharded_names` for `_AuxFrameworkPlugin` / 为 `_AuxFrameworkPlugin` 实现方法 `cp_sharded_names`
```python
    def cp_sharded_names(self) -> frozenset[str]:
        return frozenset()
```
**EN:** Method `cp_sharded_names` implements behavior on `_AuxFrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `cp_sharded_names` 为 `_AuxFrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 39-41: Implement method `discriminating_names` for `_AuxFrameworkPlugin` / 为 `_AuxFrameworkPlugin` 实现方法 `discriminating_names`
```python
    def discriminating_names(self) -> frozenset[str]:
        """Field names unique to this framework (excluding shared names like input_ids)."""
        return frozenset()
```
**EN:** Method `discriminating_names` implements behavior on `_AuxFrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `discriminating_names` 为 `_AuxFrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 44-44: Implement method `detect_layout` for `_AuxFrameworkPlugin` / 为 `_AuxFrameworkPlugin` 实现方法 `detect_layout`
```python
    def detect_layout(self, raw: dict[int, dict[str, object]]) -> TokenLayout: ...
```
**EN:** Method `detect_layout` implements behavior on `_AuxFrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `detect_layout` 为 `_AuxFrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 47-49: Implement method `compute_step_aux` for `_AuxFrameworkPlugin` / 为 `_AuxFrameworkPlugin` 实现方法 `compute_step_aux`
```python
    def compute_step_aux(
        self, step_data: dict[str, object], *, layout: TokenLayout, step: int
    ) -> TokenAlignerStepAux: ...
```
**EN:** Method `compute_step_aux` implements behavior on `_AuxFrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `compute_step_aux` 为 `_AuxFrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 52-54: Implement method `has_required_names` for `_AuxFrameworkPlugin` / 为 `_AuxFrameworkPlugin` 实现方法 `has_required_names`
```python
    def has_required_names(self, names: set[str]) -> bool:
        """Whether the minimum set of aux names needed for alignment is present."""
        ...
```
**EN:** Method `has_required_names` implements behavior on `_AuxFrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `has_required_names` 为 `_AuxFrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 57-58: Implement method `all_names` for `_AuxFrameworkPlugin` / 为 `_AuxFrameworkPlugin` 实现方法 `all_names`
```python
    def all_names(self) -> frozenset[str]:
        return self.tensor_names | self.non_tensor_names
```
**EN:** Method `all_names` implements behavior on `_AuxFrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `all_names` 为 `_AuxFrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 60-67: Implement method `extract_global_seq_lens` for `_AuxFrameworkPlugin` / 为 `_AuxFrameworkPlugin` 实现方法 `extract_global_seq_lens`
```python
    def extract_global_seq_lens(
        self, step_data: dict[str, object]
    ) -> Optional[list[int]]:
        """Extract per-seq token counts from loaded step data.

        Returns None if this framework doesn't support THD / no relevant data available.
        """
        return None
```
**EN:** Method `extract_global_seq_lens` implements behavior on `_AuxFrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `extract_global_seq_lens` 为 `_AuxFrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 69-73: Implement method `infer_cp_sharded_dims` for `_AuxFrameworkPlugin` / 为 `_AuxFrameworkPlugin` 实现方法 `infer_cp_sharded_dims`
```python
    def infer_cp_sharded_dims(self, name: str, ndim: int) -> str:
        """Infer dims string for a CP-sharded aux tensor based on its ndim."""
        raise NotImplementedError(
            f"infer_cp_sharded_dims not implemented for {type(self).__name__}"
        )
```
**EN:** Method `infer_cp_sharded_dims` implements behavior on `_AuxFrameworkPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `infer_cp_sharded_dims` 为 `_AuxFrameworkPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 79-80: Define class `_SGLangPlugin` and class context / 定义类 `_SGLangPlugin`及类上下文
```python
class _SGLangPlugin(_AuxFrameworkPlugin):
    @property
```
**EN:** This section introduces `_SGLangPlugin`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_SGLangPlugin`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 81-82: Implement method `name` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `name`
```python
    def name(self) -> str:
        return "sglang"
```
**EN:** Method `name` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `name` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 85-86: Implement method `tensor_names` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `tensor_names`
```python
    def tensor_names(self) -> frozenset[str]:
        return frozenset({"input_ids", "positions", "seq_lens", "req_pool_indices"})
```
**EN:** Method `tensor_names` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `tensor_names` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 89-90: Implement method `non_tensor_names` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `non_tensor_names`
```python
    def non_tensor_names(self) -> frozenset[str]:
        return frozenset({"rids"})
```
**EN:** Method `non_tensor_names` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `non_tensor_names` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 93-94: Implement method `cp_sharded_names` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `cp_sharded_names`
```python
    def cp_sharded_names(self) -> frozenset[str]:
        return frozenset({"input_ids", "positions"})
```
**EN:** Method `cp_sharded_names` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `cp_sharded_names` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 97-98: Implement method `discriminating_names` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `discriminating_names`
```python
    def discriminating_names(self) -> frozenset[str]:
        return frozenset({"seq_lens", "positions", "req_pool_indices", "rids"})
```
**EN:** Method `discriminating_names` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `discriminating_names` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 100-101: Implement method `has_required_names` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `has_required_names`
```python
    def has_required_names(self, names: set[str]) -> bool:
        return "input_ids" in names and "seq_lens" in names
```
**EN:** Method `has_required_names` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `has_required_names` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 103-104: Implement method `detect_layout` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `detect_layout`
```python
    def detect_layout(self, raw: dict[int, dict[str, object]]) -> TokenLayout:
        return TokenLayout.T
```
**EN:** Method `detect_layout` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `detect_layout` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 106-116: Implement method `extract_global_seq_lens` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `extract_global_seq_lens`
```python
    def extract_global_seq_lens(
        self, step_data: dict[str, object]
    ) -> Optional[list[int]]:
        if not self.cp_sharded_names:
            return None

        seq_lens = step_data.get("seq_lens")
        if not isinstance(seq_lens, torch.Tensor):
            return None

        return seq_lens.tolist()
```
**EN:** Method `extract_global_seq_lens` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `extract_global_seq_lens` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 118-128: Implement method `infer_cp_sharded_dims` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `infer_cp_sharded_dims`
```python
    def infer_cp_sharded_dims(self, name: str, ndim: int) -> str:
        """Infer dims for CP-sharded aux tensors.

        NOTE: assumes zigzag ordering — natural-order CP without explicit dims
        will be mishandled. Callers should set dims explicitly for non-zigzag CP.
        """
        if ndim == 1:
            return "t[cp:zigzag]"
        raise ValueError(
            f"SGLang: cannot infer dims for CP-sharded '{name}' with ndim={ndim}"
        )
```
**EN:** Method `infer_cp_sharded_dims` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `infer_cp_sharded_dims` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 130-162: Implement method `compute_step_aux` for `_SGLangPlugin` / 为 `_SGLangPlugin` 实现方法 `compute_step_aux`
```python
    def compute_step_aux(
        self, step_data: dict[str, object], *, layout: TokenLayout, step: int
    ) -> TokenAlignerStepAux:
        input_ids = step_data["input_ids"]
        positions = step_data["positions"]
        seq_lens = step_data["seq_lens"]
        rids_raw = step_data.get("rids")

        assert isinstance(
            input_ids, torch.Tensor
        ), f"input_ids: expected Tensor, got {type(input_ids)}"
        assert isinstance(
            positions, torch.Tensor
        ), f"positions: expected Tensor, got {type(positions)}"
        assert isinstance(
            seq_lens, torch.Tensor
        ), f"seq_lens: expected Tensor, got {type(seq_lens)}"

        seq_lens_list: list[int] = seq_lens.tolist()
        num_seqs: int = len(seq_lens_list)

        seq_ids: list[SeqId]
        if rids_raw is not None and isinstance(rids_raw, (list, tuple)):
            seq_ids = [SGLangSeqId(rid=str(r)) for r in rids_raw]
        else:
            seq_ids = [PositionalSeqId(step=step, seq_index=i) for i in range(num_seqs)]

        return TokenAlignerStepAux(
            input_ids=input_ids.tolist(),
            positions=positions.tolist(),
            seq_lens=seq_lens_list,
            seq_ids=seq_ids,
        )
```
**EN:** Method `compute_step_aux` implements behavior on `_SGLangPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `compute_step_aux` 为 `_SGLangPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 168-169: Define class `_MegatronPlugin` and class context / 定义类 `_MegatronPlugin`及类上下文
```python
class _MegatronPlugin(_AuxFrameworkPlugin):
    @property
```
**EN:** This section introduces `_MegatronPlugin`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_MegatronPlugin`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 170-171: Implement method `name` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `name`
```python
    def name(self) -> str:
        return "megatron"
```
**EN:** Method `name` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `name` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 174-175: Implement method `tensor_names` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `tensor_names`
```python
    def tensor_names(self) -> frozenset[str]:
        return frozenset({"input_ids", "position_ids", "cu_seqlens_q", "cu_seqlens_kv"})
```
**EN:** Method `tensor_names` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `tensor_names` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 178-179: Implement method `non_tensor_names` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `non_tensor_names`
```python
    def non_tensor_names(self) -> frozenset[str]:
        return frozenset({"qkv_format"})
```
**EN:** Method `non_tensor_names` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `non_tensor_names` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 182-183: Implement method `cp_sharded_names` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `cp_sharded_names`
```python
    def cp_sharded_names(self) -> frozenset[str]:
        return frozenset({"input_ids", "position_ids"})
```
**EN:** Method `cp_sharded_names` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `cp_sharded_names` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 186-187: Implement method `discriminating_names` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `discriminating_names`
```python
    def discriminating_names(self) -> frozenset[str]:
        return frozenset({"cu_seqlens_q", "cu_seqlens_kv", "qkv_format"})
```
**EN:** Method `discriminating_names` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `discriminating_names` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 189-190: Implement method `has_required_names` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `has_required_names`
```python
    def has_required_names(self, names: set[str]) -> bool:
        return "input_ids" in names
```
**EN:** Method `has_required_names` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `has_required_names` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 192-202: Implement method `extract_global_seq_lens` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `extract_global_seq_lens`
```python
    def extract_global_seq_lens(
        self, step_data: dict[str, object]
    ) -> Optional[list[int]]:
        if not self.cp_sharded_names:
            return None

        cu_seqlens_q = step_data.get("cu_seqlens_q")
        if not isinstance(cu_seqlens_q, torch.Tensor):
            return None

        return (cu_seqlens_q[1:] - cu_seqlens_q[:-1]).tolist()
```
**EN:** Method `extract_global_seq_lens` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `extract_global_seq_lens` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 204-216: Implement method `infer_cp_sharded_dims` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `infer_cp_sharded_dims`
```python
    def infer_cp_sharded_dims(self, name: str, ndim: int) -> str:
        """Infer dims for CP-sharded aux tensors.

        NOTE: assumes zigzag ordering — natural-order CP without explicit dims
        will be mishandled. Callers should set dims explicitly for non-zigzag CP.
        """
        if ndim == 1:
            return "t[cp:zigzag]"
        if ndim == 2:
            return "b s[cp:zigzag]"
        raise ValueError(
            f"Megatron: cannot infer dims for CP-sharded '{name}' with ndim={ndim}"
        )
```
**EN:** Method `infer_cp_sharded_dims` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `infer_cp_sharded_dims` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 218-239: Implement method `detect_layout` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `detect_layout`
```python
    def detect_layout(self, raw: dict[int, dict[str, object]]) -> TokenLayout:
        for step_data in raw.values():
            if (qkv_format := step_data.get("qkv_format")) is not None:
                fmt = qkv_format if isinstance(qkv_format, str) else str(qkv_format)
                if "bshd" in fmt.lower():
                    return TokenLayout.BS
                return TokenLayout.T

            input_ids = step_data.get("input_ids")
            if isinstance(input_ids, torch.Tensor) and input_ids.ndim == 2:
                return TokenLayout.BS

        log_sink.add(
            InfoLog(
                category="layout_detection_fallback",
                message=(
                    "Megatron layout detection: no qkv_format or 2D input_ids found, "
                    "falling back to T"
                ),
            )
        )
        return TokenLayout.T
```
**EN:** Method `detect_layout` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `detect_layout` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 241-277: Implement method `compute_step_aux` for `_MegatronPlugin` / 为 `_MegatronPlugin` 实现方法 `compute_step_aux`
```python
    def compute_step_aux(
        self, step_data: dict[str, object], *, layout: TokenLayout, step: int
    ) -> TokenAlignerStepAux:
        input_ids: torch.Tensor = step_data["input_ids"]
        is_bshd: bool = layout == TokenLayout.BS

        # BSHD [B, S] → flat [B*S]; THD [T] stays as-is
        flat_ids: list[int] = input_ids.reshape(-1).tolist()

        if (cu_seqlens_q := step_data.get("cu_seqlens_q")) is not None:
            seq_lens_list: list[int] = (cu_seqlens_q[1:] - cu_seqlens_q[:-1]).tolist()
        elif is_bshd:
            seq_lens_list = [input_ids.shape[1]] * input_ids.shape[0]
        else:
            seq_lens_list = [input_ids.shape[0]]

        if (position_ids := step_data.get("position_ids")) is not None:
            flat_positions: list[int] = position_ids.reshape(-1).tolist()
        elif is_bshd:
            flat_positions = list(range(input_ids.shape[1])) * input_ids.shape[0]
        else:
            flat_positions = _infer_positions(
                seq_lens=torch.tensor(seq_lens_list)
            ).tolist()

        num_seqs: int = len(seq_lens_list)
        seq_ids: list[SeqId] = [
            PositionalSeqId(step=step, seq_index=seq_index)
            for seq_index in range(num_seqs)
        ]

        return TokenAlignerStepAux(
            input_ids=flat_ids,
            positions=flat_positions,
            seq_lens=seq_lens_list,
            seq_ids=seq_ids,
        )
```
**EN:** Method `compute_step_aux` implements behavior on `_MegatronPlugin`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `compute_step_aux` 为 `_MegatronPlugin` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 282-284: Declare module-level symbols such as `_plugins`, `AUX_NAMES` / 声明模块级符号，例如 `_plugins`, `AUX_NAMES`
```python
_plugins: list[_AuxFrameworkPlugin] = [_SGLangPlugin(), _MegatronPlugin()]

AUX_NAMES: frozenset[str] = frozenset().union(*(p.all_names for p in _plugins))
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 290-292: Implement helper `_infer_positions` / 实现辅助函数 `_infer_positions`
```python
def _infer_positions(*, seq_lens: torch.Tensor) -> torch.Tensor:
    """Infer positions when position_ids is missing (THD only)."""
    return torch.cat([torch.arange(int(slen.item())) for slen in seq_lens])
```
**EN:** Function `_infer_positions` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_infer_positions` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_AuxFrameworkPlugin`, `_SGLangPlugin`, `_MegatronPlugin`, `_infer_positions`
- **Module role / 模块角色**: Heuristic token alignment / 启发式 token 对齐
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `abc`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.log_sink`, `sglang.srt.debug_utils.comparator.output_types`
