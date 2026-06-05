# encoder_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/mm/encoder_cache.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `EncoderCache` for the V1 `worker/gpu/mm` subsystem. / 为 V1 的 `worker/gpu/mm` 子系统实现 `EncoderCache`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import torch

from vllm.multimodal.inputs import MultiModalFeatureSpec
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.multimodal.inputs`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.multimodal.inputs` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `EncoderCache` class / `EncoderCache` 类
```python
class EncoderCache:
```
**EN:** Introduces the `EncoderCache` class. Core methods include `__init__`, `add_request`, `remove_request`, `reset_mm_cache`, `reset_encoder_cache`, `free_encoder_cache`.
**CN:** 这里定义 `EncoderCache` 类。核心方法包括 `__init__`, `add_request`, `remove_request`, `reset_mm_cache`, `reset_encoder_cache`, `free_encoder_cache`。

### `EncoderCache.__init__` method / `EncoderCache.__init__` 方法
```python
    def __init__(self):
        # req_id -> MM features
        self.mm_features: dict[str, list[MultiModalFeatureSpec]] = {}
        # MM hash -> encoder outputs
        self.encoder_outputs: dict[str, torch.Tensor] = {}
```
**EN:** This method initializes the object state within `EncoderCache`. It touches state such as `mm_features`, `encoder_outputs`.
**CN:** 该方法会初始化对象状态，其作用域位于`EncoderCache`。 它会读写 `mm_features`, `encoder_outputs` 等状态。

### `EncoderCache.add_request` method / `EncoderCache.add_request` 方法
```python
    def add_request(
        self, req_id: str, mm_features: list[MultiModalFeatureSpec]
    ) -> None:
        self.mm_features[req_id] = mm_features
```
**EN:** This method implements `add_request` within `EncoderCache`. It touches state such as `mm_features`.
**CN:** 该方法会实现 `add_request`，其作用域位于`EncoderCache`。 它会读写 `mm_features` 等状态。

### `EncoderCache.remove_request` method / `EncoderCache.remove_request` 方法
```python
    def remove_request(self, req_id: str) -> None:
        self.mm_features.pop(req_id, None)
```
**EN:** This method implements `remove_request` within `EncoderCache`. Key calls include `pop`.
**CN:** 该方法会实现 `remove_request`，其作用域位于`EncoderCache`。 关键调用包括 `pop`。

### `EncoderCache.reset_mm_cache` method / `EncoderCache.reset_mm_cache` 方法
```python
    def reset_mm_cache(self) -> None:
        """
        Clear the multi-modal cache that was used during profiling,
        but no longer needed during inference.
        """
        # TODO: Implement MM budget for encoder dummy run
        pass
```
**EN:** This method implements `reset_mm_cache` within `EncoderCache`. The docstring frames it as: Clear the multi-modal cache that was used during profiling, but no longer needed during inference.
**CN:** 该方法会实现 `reset_mm_cache`，其作用域位于`EncoderCache`。

### `EncoderCache.reset_encoder_cache` method / `EncoderCache.reset_encoder_cache` 方法
```python
    def reset_encoder_cache(self) -> None:
        """Clear the GPU-side encoder cache storing vision embeddings.

        This should be called when model weights are updated to ensure
        stale embeddings computed with old weights are not reused.
        """
        self.encoder_outputs.clear()
```
**EN:** This method implements `reset_encoder_cache` within `EncoderCache`. The docstring frames it as: Clear the GPU-side encoder cache storing vision embeddings. Key calls include `clear`.
**CN:** 该方法会实现 `reset_encoder_cache`，其作用域位于`EncoderCache`。 关键调用包括 `clear`。

### `EncoderCache.free_encoder_cache` method / `EncoderCache.free_encoder_cache` 方法
```python
    def free_encoder_cache(self, mm_hash: str) -> None:
        self.encoder_outputs.pop(mm_hash, None)
```
**EN:** This method implements `free_encoder_cache` within `EncoderCache`. Key calls include `pop`.
**CN:** 该方法会实现 `free_encoder_cache`，其作用域位于`EncoderCache`。 关键调用包括 `pop`。

## Key Concepts / 关键概念
- `EncoderCache`: central class or interface in this module. / `EncoderCache`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.multimodal.inputs`
