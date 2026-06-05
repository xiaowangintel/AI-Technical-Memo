# metadata.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/spec_decode/metadata.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `SpecDecodeMetadata` for the V1 `spec_decode` subsystem. / 为 V1 的 `spec_decode` 子系统实现 `SpecDecodeMetadata`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from dataclasses import dataclass

import numpy as np
import torch
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `SpecDecodeMetadata` class / `SpecDecodeMetadata` 类
```python
@dataclass
class SpecDecodeMetadata:
    # [num_tokens]
    draft_token_ids: torch.Tensor
    # [batch_size]
    num_draft_tokens: list[int]
    # [batch_size]
    cu_num_draft_tokens: torch.Tensor
    # [batch_size]
    cu_num_sampled_tokens: torch.Tensor
    # [num_tokens]
    target_logits_indices: torch.Tensor
    # [batch_size]
    bonus_logits_indices: torch.Tensor
    # [num_tokens + batch_size]
    logits_indices: torch.Tensor
```
**EN:** Uses `@dataclass` to package related state for `SpecDecodeMetadata`. Typical fields include `draft_token_ids`, `num_draft_tokens`, `cu_num_draft_tokens`, `cu_num_sampled_tokens`, `target_logits_indices`, `bonus_logits_indices`.
**CN:** `SpecDecodeMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `draft_token_ids`, `num_draft_tokens`, `cu_num_draft_tokens`, `cu_num_sampled_tokens`, `target_logits_indices`, `bonus_logits_indices`。

### `SpecDecodeMetadata.__post_init__` method / `SpecDecodeMetadata.__post_init__` 方法
```python
    def __post_init__(self):
        self.max_spec_len = max(self.num_draft_tokens)
```
**EN:** This method implements `__post_init__` within `SpecDecodeMetadata`. Key calls include `max`. It touches state such as `max_spec_len`.
**CN:** 该方法会实现 `__post_init__`，其作用域位于`SpecDecodeMetadata`。 关键调用包括 `max`。 它会读写 `max_spec_len` 等状态。

### `SpecDecodeMetadata.make_dummy` method / `SpecDecodeMetadata.make_dummy` 方法
```python
    @classmethod
    def make_dummy(
        cls,
        draft_token_ids: list[list[int]],
        device: torch.device,
    ) -> "SpecDecodeMetadata":
        batch_size = len(draft_token_ids)
        num_draft_tokens = [len(ids) for ids in draft_token_ids]
        num_sampled_tokens = [len(ids) + 1 for ids in draft_token_ids]
        flattened_draft_token_ids = sum(draft_token_ids, [])
        num_tokens = len(flattened_draft_token_ids)

        draft_token_ids_tensor = torch.tensor(
            flattened_draft_token_ids, dtype=torch.int32, device=device
        )
        cu_num_draft_tokens = np.cumsum(num_draft_tokens, dtype=np.int32)
        cu_num_draft_tokens_tensor = torch.from_numpy(cu_num_draft_tokens).to(device)
        cu_num_sampled_tokens = np.cumsum(num_sampled_tokens, dtype=np.int32)
        cu_num_sampled_tokens_tensor = torch.from_numpy(cu_num_sampled_tokens).to(
            device
        )

        target_logits_indices = torch.zeros(
            num_tokens, dtype=torch.int32, device=device
        )
        bonus_logits_indices = torch.zeros(batch_size, dtype=torch.int32, device=device)
        logits_indices = torch.zeros(
            num_tokens + batch_size, dtype=torch.int32, device=device
        )
        return cls(
            draft_token_ids=draft_token_ids_tensor,
            num_draft_tokens=num_draft_tokens,
            cu_num_draft_tokens=cu_num_draft_tokens_tensor,
            cu_num_sampled_tokens=cu_num_sampled_tokens_tensor,
            target_logits_indices=target_logits_indices,
            bonus_logits_indices=bonus_logits_indices,
            logits_indices=logits_indices,
        )
```
**EN:** This method implements `make_dummy` within `SpecDecodeMetadata`. Key calls include `len`, `sum`, `tensor`, `cumsum`, `to`, `zeros`.
**CN:** 该方法会实现 `make_dummy`，其作用域位于`SpecDecodeMetadata`。 关键调用包括 `len`, `sum`, `tensor`, `cumsum`, `to`, `zeros`。

## Key Concepts / 关键概念
- `SpecDecodeMetadata`: central class or interface in this module. / `SpecDecodeMetadata`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`
- External / 外部依赖: `numpy`, `torch`
