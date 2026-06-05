# caching.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/caching.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Serialize, hash, save, and reload compiled artifacts for reusable compilation cache entries. / 对编译产物进行序列化、哈希、保存与重载，以复用编译缓存。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-25)
```python
import contextlib
import hashlib
import inspect
import os
import pickle
from collections.abc import Callable, Sequence
from typing import Any, Literal
from unittest.mock import patch

import torch
from torch._subclasses import FakeTensorMode
from torch.fx._graph_pickler import GraphPickler, Options
from torch.utils import _pytree as pytree

import vllm.envs as envs
from vllm.compilation.codegen import compile_execution_fn
from vllm.compilation.compiler_interface import get_inductor_factors
from vllm.compilation.counter import compilation_counter
from vllm.config import VllmConfig, get_current_vllm_config
from vllm.config.utils import hash_factors
from vllm.logger import init_logger
from vllm.utils.hashing import safe_hash
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in vLLM's compilation path.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在vLLM 的编译路径中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 34-34)
```python
logger = init_logger(__name__)
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout vLLM's compilation path.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在vLLM 的编译路径中复用。

### Class `StandaloneCompiledArtifacts` (lines 37-151)
```python
class StandaloneCompiledArtifacts:
    """Storage for standalone compiled artifacts with content-based deduplication.

    Deduplication works via a two-level indirection:
    1. `submodule_bytes` maps "{submod_name}_{shape}" -> SHA256 hash
    2. `submodule_bytes_store` maps SHA256 hash -> actual bytes

    When inserting, we compute the SHA256 hash of the bytes. If the hash
    already exists in `submodule_bytes_store`, we reuse the existing entry
    rather than storing duplicate bytes. This is common because submodules
    often compile to identical artifacts (e.g., identical transformer layers
    split on attn)
    """

    def __init__(self) -> None:
        # dict from submodule name to byte hash
        self.submodule_bytes: dict[str, str] = {}
        # dict from byte hash to bytes
        self.submodule_bytes_store: dict[str, bytes] = {}
        # dict from byte hash to loaded module
...
```
**EN:** Class `StandaloneCompiledArtifacts` holds a main piece of file-specific logic. It operates inside vLLM's compilation path. Storage for standalone compiled artifacts with content-based deduplication. Key methods include __init__, insert, get, get_loaded, size_bytes.
**CN:** 类 `StandaloneCompiledArtifacts` 承载了该文件中的一块核心逻辑，并服务于vLLM 的编译路径。 文档字符串强调：Storage for standalone compiled artifacts with content-based deduplication. 关键方法包括 __init__, insert, get, get_loaded, size_bytes。

### Function `patch_pytree_map_over_slice` (lines 155-163)
```python
def patch_pytree_map_over_slice():
    pytree._private_register_pytree_node(
        slice, lambda x: ([x.start, x.stop, x.step], None), lambda x, c: slice(*x)
    )

    try:
        yield
    finally:
        pytree._deregister_pytree_node(slice)
```
**EN:** Function `patch_pytree_map_over_slice` restores or patches behavior so the surrounding pipeline can reuse cached or compatible state. It fits into vLLM's compilation path.
**CN:** 函数 `patch_pytree_map_over_slice` 恢复或修补相关行为，使周边流程可以复用缓存或兼容状态，并嵌入vLLM 的编译路径。

### Class `VllmSerializableFunction` (lines 166-408)
```python
class VllmSerializableFunction(SerializableCallable):  # type: ignore[misc]
    """
    A wrapper around a compiled function by vllm. It will forward the tensor
    inputs to the compiled function and return the result.
    It also implements a serialization interface to support PyTorch's precompile
    with custom backend, so that we can save and load the compiled function on
    disk. There's no need to wrap around the compiled function if we don't want
    to serialize them in particular cases.
    Right now serialization for the custom backend is done via
    serializing the Dynamo fx graph plus example inputs.
    """

    def __init__(
        self,
        graph_module: torch.fx.GraphModule | bytes,
        example_inputs: Sequence[Any],
        prefix: str,
        optimized_call: Callable[..., Any],
        is_encoder: bool = False,
        vllm_backend: Any | None = None,
...
```
**EN:** Class `VllmSerializableFunction` holds a main piece of file-specific logic. It operates inside vLLM's compilation path. A wrapper around a compiled function by vllm. Key methods include __init__, __call__, serialize_graph_module, deserialize_graph_module, serialize_compile_artifacts.
**CN:** 类 `VllmSerializableFunction` 承载了该文件中的一块核心逻辑，并服务于vLLM 的编译路径。 文档字符串强调：A wrapper around a compiled function by vllm. 关键方法包括 __init__, __call__, serialize_graph_module, deserialize_graph_module, serialize_compile_artifacts。

### Function `reconstruct_serializable_fn_from_mega_artifact` (lines 411-562)
```python
def reconstruct_serializable_fn_from_mega_artifact(
    state: dict[str, Any],
    standalone_compile_artifacts: "StandaloneCompiledArtifacts",
    vllm_config: VllmConfig,
    sym_shape_indices_map: dict[str, list[int]],
    returns_tuple_map: dict[str, bool],
    fake_mode: FakeTensorMode,
) -> "VllmSerializableFunction":
    """Construct a VllmSerializableFunction from cached inductor artifacts.

    This function reconstructs a callable model from pre-compiled inductor
    artifacts without re-running the compilation. It:
    1. Loads all cached artifacts
    2. Builds compiled callables for each submodule/shape
    3. Creates PiecewiseBackend instances that dispatch to cached artifacts
    4. Wraps with cudagraph if needed
...
```
**EN:** Function `reconstruct_serializable_fn_from_mega_artifact` restores or patches behavior so the surrounding pipeline can reuse cached or compatible state. It fits into vLLM's compilation path. Construct a VllmSerializableFunction from cached inductor artifacts.
**CN:** 函数 `reconstruct_serializable_fn_from_mega_artifact` 恢复或修补相关行为，使周边流程可以复用缓存或兼容状态，并嵌入vLLM 的编译路径。 文档字符串说明：Construct a VllmSerializableFunction from cached inductor artifacts.

### Function `aot_compile_hash_factors` (lines 565-581)
```python
def aot_compile_hash_factors(vllm_config: VllmConfig) -> list[str]:
    factors = []
    # 0. factors come from the env, for example, The values of
    # VLLM_PP_LAYER_PARTITION will affect the computation graph.
    env_hash = hash_factors(envs.compile_factors())
    factors.append(env_hash)

    # 1. factors come from the vllm_config (it mainly summarizes how the
    #    model is created)
    config_hash = vllm_config.compute_hash()
    factors.append(config_hash)

    # 2. inductor factors if applicable
    if envs.VLLM_USE_MEGA_AOT_ARTIFACT:
        factors.extend(get_inductor_factors())

...
```
**EN:** Function `aot_compile_hash_factors` computes a digest or summary used for cache identity and reproducibility. It fits into vLLM's compilation path.
**CN:** 函数 `aot_compile_hash_factors` 计算用于缓存身份与可复现性的摘要或哈希，并嵌入vLLM 的编译路径。

### Function `_compute_code_hash_with_content` (lines 584-597)
```python
def _compute_code_hash_with_content(file_contents: dict[str, str]) -> str:
    items = list(sorted(file_contents.items(), key=lambda x: x[0]))
    hash_content = []
    for filepath, content in items:
        hash_content.append(filepath)
        if filepath == "<string>":
            # This means the function was dynamically generated, with
            # e.g. exec(). We can't actually check these.
            continue
        hash_content.append(content)
    result: str = safe_hash(
        "\n".join(hash_content).encode(), usedforsecurity=False
    ).hexdigest()
    return result
```
**EN:** Function `_compute_code_hash_with_content` computes a digest or summary used for cache identity and reproducibility. It fits into vLLM's compilation path.
**CN:** 函数 `_compute_code_hash_with_content` 计算用于缓存身份与可复现性的摘要或哈希，并嵌入vLLM 的编译路径。

### Function `_compute_code_hash` (lines 600-612)
```python
def _compute_code_hash(files: set[str]) -> str:
    logger.debug(
        "Traced files (to be considered for compilation cache):\n%s", "\n".join(files)
    )
    file_contents = {}
    for filepath in files:
        # Skip files that don't exist (e.g., <string>, <frozen modules>, etc.)
        if not os.path.isfile(filepath):
            file_contents[filepath] = ""
        else:
            with open(filepath) as f:
                file_contents[filepath] = f.read()
    return _compute_code_hash_with_content(file_contents)
```
**EN:** Function `_compute_code_hash` computes a digest or summary used for cache identity and reproducibility. It fits into vLLM's compilation path.
**CN:** 函数 `_compute_code_hash` 计算用于缓存身份与可复现性的摘要或哈希，并嵌入vLLM 的编译路径。

## Key Concepts / 关键概念
- **Compilation pipeline / 编译流水线**
  - **EN:** The file participates in turning FX graphs into optimized executable code or runtime wrappers.
  - **CN:** 该文件参与将 FX 图转换为优化后的可执行代码或运行时封装。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `import vllm.envs as envs`, `from vllm.compilation.codegen import compile_execution_fn`, `from vllm.compilation.compiler_interface import get_inductor_factors`, `from vllm.compilation.counter import compilation_counter`, `from vllm.config import VllmConfig, get_current_vllm_config`, `from vllm.config.utils import hash_factors`, `from vllm.logger import init_logger`, `from vllm.utils.hashing import safe_hash`
- **PyTorch / Torch 栈**: `import torch`, `from torch._subclasses import FakeTensorMode`, `from torch.fx._graph_pickler import GraphPickler, Options`, `from torch.utils import _pytree as pytree`
- **Stdlib / 标准库**: `import contextlib`, `import hashlib`, `import inspect`, `import os`, `import pickle`, `from collections.abc import Callable, Sequence`, `from typing import Any, Literal`, `from unittest.mock import patch`
