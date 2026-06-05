# mooncake_embedding_store.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/storage/mooncake_store/mooncake_embedding_store.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the mooncake embedding store logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的Mooncake嵌入store相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Imports and setup / 导入与初始化
```python
import logging
from typing import Any, List

from sglang.srt.mem_cache.storage.mooncake_store.mooncake_store import MooncakeBaseStore
```
**EN:** Imports `logging`, `typing`, `sglang.srt.mem_cache.storage.mooncake_store.mooncake_store` and other helpers used by the surrounding scope.
**CN:** 导入 `logging`, `typing`, `sglang.srt.mem_cache.storage.mooncake_store.mooncake_store` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 6-6: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 9-9: MooncakeEmbeddingStore declaration / MooncakeEmbeddingStore 声明
```python
class MooncakeEmbeddingStore(MooncakeBaseStore):
```
**EN:** Declares the `MooncakeEmbeddingStore` class and connects it to `MooncakeBaseStore`.
**CN:** 声明 `MooncakeEmbeddingStore` 类，并将其关联到 `MooncakeBaseStore`。

### Lines 10-31: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        storage_config: Any = None,
    ):
        super().__init__()

        MooncakeDistributedStore = self._import_mooncake_store()
        self.store = MooncakeDistributedStore()
        self.config = self._load_config(storage_config)
        ret_code = self.store.setup(
            self.config.local_hostname,
            self.config.metadata_server,
# ... omitted for brevity ...
        if ret_code != 0:
            raise RuntimeError(f"Failed to setup Mooncake Embedding Store: {ret_code}")

        logger.info("Mooncake Embedding Store initialized successfully.")
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `MooncakeEmbeddingStore`. It validates error cases explicitly.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `MooncakeEmbeddingStore`。它会显式处理错误场景。

### Lines 33-34: get_key implementation / get_key 实现
```python
    def get_key(self, image_hash: str) -> str:
        return f"emb_{image_hash}"
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MooncakeEmbeddingStore`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MooncakeEmbeddingStore`。它会向调用方返回计算结果。

### Lines 36-41: batch_get implementation / batch_get 实现
```python
    def batch_get(
        self, hashes: List[str], ptrs: List[int], sizes: List[int]
    ) -> List[bool]:
        keys = [self.get_key(h) for h in hashes]
        results = self.store.batch_get_into(keys, ptrs, sizes)
        return [res > 0 for res in results]
```
**EN:** Implements the batch get routine for this scope. It belongs to `MooncakeEmbeddingStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch get例程。 该方法属于 `MooncakeEmbeddingStore`。它会向调用方返回计算结果。

### Lines 43-63: batch_put implementation / batch_put 实现
```python
    def batch_put(
        self, hashes: List[str], ptrs: List[int], sizes: List[int]
    ) -> List[bool]:
        keys = [self.get_key(h) for h in hashes]
        exists = self.store.batch_is_exist(keys)

        put_keys, put_ptrs, put_sizes, indices = [], [], [], []
        success_map = [True] * len(hashes)

        for i, status in enumerate(exists):
            if status != 1:
                put_keys.append(keys[i])
# ... omitted for brevity ...
            results = self.store.batch_put_from(put_keys, put_ptrs, put_sizes)
            for i, res in enumerate(results):
                success_map[indices[i]] = res == 0
        return success_map
```
**EN:** Implements the batch PUT routine for this scope. It belongs to `MooncakeEmbeddingStore`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch put例程。 该方法属于 `MooncakeEmbeddingStore`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 65-68: batch_is_exist implementation / batch_is_exist 实现
```python
    def batch_is_exist(self, hashes: List[str]) -> List[bool]:
        keys = [self.get_key(h) for h in hashes]
        results = self.store.batch_is_exist(keys)
        return [res == 1 for res in results]
```
**EN:** Implements the batch IS exist routine for this scope. It belongs to `MooncakeEmbeddingStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch is exist例程。 该方法属于 `MooncakeEmbeddingStore`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`MooncakeEmbeddingStore`**: Defines the `MooncakeEmbeddingStore` type and its core responsibilities. / 定义 `MooncakeEmbeddingStore` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `logging`, `typing`
- **Internal / 内部**: `sglang.srt.mem_cache.storage.mooncake_store.mooncake_store`
