# layerwise.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/model_loader/reload/layerwise.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements layerwise support for the `reload` portion of vLLM. / 为 vLLM 的 `reload` 子目录实现与 layerwise 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-30)
```python
import inspect

from collections.abc import Callable

from functools import wraps

from weakref import WeakKeyDictionary, WeakSet

import torch

from vllm.config import ModelConfig

from vllm.logger import init_logger

from vllm.model_executor.layers.attention import Attention, MLAAttention

from vllm.model_executor.layers.quantization.base_config import QuantizeMethodBase

from vllm.model_executor.model_loader.weight_utils import default_weight_loader

from .meta import (
    SKIP_TENSORS,
    capture_layer_to_meta,
    get_numel_loaded,
    materialize_layer,
    restore_layer_on_meta,
)

from .types import LayerReloadingInfo

from .utils import (
    get_info_size,
    get_layer_params_buffers,
    get_layer_size,
    get_layer_tensors,
    has_device_tensors,
# ... omitted for brevity ...
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 32-53)
```python
logger = init_logger(__name__)

__all__ = [
    "get_layerwise_info",
    "record_metadata_for_reloading",
    "initialize_layerwise_reload",
    "finalize_layerwise_processing",
    "finalize_layerwise_reload",
]

LAYERWISE_INFO: WeakKeyDictionary[torch.nn.Module, LayerReloadingInfo] = (
    WeakKeyDictionary()
)

LOADING_LAYERS: WeakSet[torch.nn.Module] = WeakSet()
```
**EN:** This constant/configuration block defines `logger`, `__all__`, `LAYERWISE_INFO`, `LOADING_LAYERS`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`, `__all__`, `LAYERWISE_INFO`, `LOADING_LAYERS`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Function `get_layerwise_info` (lines 56-67)
```python
def get_layerwise_info(layer: torch.nn.Module) -> LayerReloadingInfo:
    """
    Get information related to restoring and layerwise processing. If no previous
    information existed, a new entry is constructed
    """
    if layer not in LAYERWISE_INFO:
        LAYERWISE_INFO[layer] = LayerReloadingInfo(
            restore_metadata=({}, {}),
            restore_device=torch.get_default_device(),
        )

    return LAYERWISE_INFO[layer]
```
**EN:** Function `get_layerwise_info` provides a reusable helper around the module's main workflow. The docstring highlights: Get information related to restoring and layerwise processing. Key calls such as `LayerReloadingInfo`, `torch.get_default_device` show the concrete execution path.
**CN:** Function `get_layerwise_info` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Get information related to restoring and layerwise processing. 像 `LayerReloadingInfo`, `torch.get_default_device` 这样的关键调用展示了该代码块的具体执行路径。

### Function `record_metadata_for_reloading` (lines 70-80)
```python
def record_metadata_for_reloading(model: torch.nn.Module):
    """
    Record layer metadata needed for later reloading.

    Stores parameter and buffer metadata as meta tensors for restoration.
    Must be called before `initialize_layerwise_reload`.
    """
    for layer in model.modules():
        info = get_layerwise_info(layer)
        info.restore_metadata = capture_layer_to_meta(layer)
        info.restore_device = torch.get_default_device()
```
**EN:** Function `record_metadata_for_reloading` handles loading or retrieval of external/internal data. The docstring highlights: Record layer metadata needed for later reloading. Key calls such as `model.modules`, `get_layerwise_info`, `capture_layer_to_meta`, `torch.get_default_device` show the concrete execution path.
**CN:** Function `record_metadata_for_reloading` 负责加载或获取外部/内部数据。 文档字符串强调：Record layer metadata needed for later reloading. 像 `model.modules`, `get_layerwise_info`, `capture_layer_to_meta`, `torch.get_default_device` 这样的关键调用展示了该代码块的具体执行路径。

### Function `initialize_layerwise_reload` (lines 84-117)
```python
def initialize_layerwise_reload(model: torch.nn.Module):
    """
    Set up layerwise weight loading with deferred processing.

    Must be called after `record_metadata_for_reloading`. This function:
    1. Saves current kernel tensors for later copying
    2. Restores layer parameters/buffers from metadata (on meta device)
    3. Wraps weight loaders to defer processing until all weights are loaded

    When all weights for a layer are loaded, the wrapped loaders will:
    1. Materialize the layer onto the target device
    2. Load all cached weights
    3. Run quantization processing if applicable
    4. Copy processed values back to original tensor storage
    """
    # disable torchao reloading to avoid infinite recursion
    model._original_do_torchao_reload = getattr(model, "_do_torchao_reload", False)
    model._do_torchao_reload = False

    for layer in model.modules():
        info = get_layerwise_info(layer)

        # Skip if the layer has already been initialized
        if info.can_load():
            continue
    # ... omitted for brevity ...
        # Wrap weight loaders to buffer loading
        initialize_online_processing(layer)
```
**EN:** Function `initialize_layerwise_reload` handles loading or retrieval of external/internal data. The docstring highlights: Set up layerwise weight loading with deferred processing. Key calls such as `getattr`, `model.modules`, `get_layerwise_info`, `info.can_load`, `get_layer_params_buffers` show the concrete execution path.
**CN:** Function `initialize_layerwise_reload` 负责加载或获取外部/内部数据。 文档字符串强调：Set up layerwise weight loading with deferred processing. 像 `getattr`, `model.modules`, `get_layerwise_info`, `info.can_load`, `get_layer_params_buffers` 这样的关键调用展示了该代码块的具体执行路径。

### Function `initialize_online_processing` (lines 120-140)
```python
def initialize_online_processing(layer: torch.nn.Module):
    """
    Wrap a layer's weight loaders with online processing loaders.
    Called by either `initialize_layerwise_reload` or an online quantization scheme,
    prevents double wrapping in the case of online quantization + reloading

    :param layer: layer whose parameter weight loaders will be wrapped
    """
    info = get_layerwise_info(layer)

    # Track loading progress to determine when to process/copy
    info.load_numel = 0
    info.load_numel_total = get_layer_size(layer)

    # Wrap each parameter's weight loader
    # Note that nested wrapping will occur for shared tensors
    for name, tensor in get_layer_tensors(layer).items():
        if name in SKIP_TENSORS:
            continue
        if _get_weight_loader(tensor).__name__ != "online_process_loader":
            tensor.weight_loader = make_online_process_loader(layer, name)
```
**EN:** Function `initialize_online_processing` constructs derived objects, runtime state, or helper structures. The docstring highlights: Wrap a layer's weight loaders with online processing loaders. Key calls such as `get_layerwise_info`, `get_layer_size`, `get_layer_tensors(layer).items`, `get_layer_tensors`, `_get_weight_loader` show the concrete execution path.
**CN:** Function `initialize_online_processing` 负责构造派生对象、运行时状态或辅助结构。 文档字符串强调：Wrap a layer's weight loaders with online processing loaders. 像 `get_layerwise_info`, `get_layer_size`, `get_layer_tensors(layer).items`, `get_layer_tensors`, `_get_weight_loader` 这样的关键调用展示了该代码块的具体执行路径。

### Function `make_online_process_loader` (lines 143-213)
```python
def make_online_process_loader(layer: torch.nn.Module, param_name: str) -> Callable:
    """Create a wrapped weight loader that defers processing."""
    info = get_layerwise_info(layer)
    param = getattr(layer, param_name)
    original_loader = _get_original_loader(param)
    loader_signature = inspect.signature(original_loader)

    @wraps(original_loader, assigned=("__doc__", "__annotations__"))
    def online_process_loader(*args, **kwargs):
        if not info.can_load():
            # Unfortunately, some qconfigs are set up to load the same weight
            # multiple times. For example, CT_WNA16 loads `weight_shape` for
            # each of the qkv partitions. This results in layers loading extra
            # weights (beyond load_numel_total) after it's already processed.
            #
            # Best solution is to ensure that `load_numel_total` reflects the
            # actual number of weights loaded, either by modifying qconfigs to
            # create as many weights as loaded (see padding issue as well)
            # or maybe capturing how many weights are loaded on first pass
            #
            # For now, `load_numel_total` is still safe to use as long as
            # there's no way to reach `load_numel_total` without loading all
            # necessary weights. `weight_shape` is very small, so this is safe.
            # see Limitations(4)
            logger.debug("%s: Excessive loading", layer.__class__.__name__)
    # ... omitted for brevity ...

    return online_process_loader
```
**EN:** Function `make_online_process_loader` handles loading or retrieval of external/internal data. The docstring highlights: Create a wrapped weight loader that defers processing. Key calls such as `get_layerwise_info`, `getattr`, `_get_original_loader`, `inspect.signature`, `info.can_load` show the concrete execution path.
**CN:** Function `make_online_process_loader` 负责加载或获取外部/内部数据。 文档字符串强调：Create a wrapped weight loader that defers processing. 像 `get_layerwise_info`, `getattr`, `_get_original_loader`, `inspect.signature`, `info.can_load` 这样的关键调用展示了该代码块的具体执行路径。

### Function `finalize_layerwise_processing` (lines 216-271)
```python
def finalize_layerwise_processing(model: torch.nn.Module, model_config: ModelConfig):
    """
    Apply processing to any layers which were not layerwise processed during loading.
    This includes attention layers and layers which have weight elements which are not
    loaded (due to padding).

    This function should be applied after `initialize_layerwise_reload` is applied
    unwrap the layerwise weight loaders.

    :param model: model to finalize processing for
    :param model_config: config needed for applying processing to attention layers
    """
    if hasattr(model, "_original_do_torchao_reload"):
        model._do_torchao_reload = model._original_do_torchao_reload

    deferred_attn: list[tuple[torch.nn.Module, LayerReloadingInfo]] = []

    for layer in model.modules():
        info = get_layerwise_info(layer)
        if not info.can_load():
            info.reset()
            continue

        # Attention/MLA layers are processed after all other layers
        if isinstance(layer, (Attention, MLAAttention)):
    # ... omitted for brevity ...

    LOADING_LAYERS.clear()
```
**EN:** Function `finalize_layerwise_processing` provides a reusable helper around the module's main workflow. The docstring highlights: Apply processing to any layers which were not layerwise processed during loading. Key calls such as `hasattr`, `model.modules`, `get_layerwise_info`, `info.can_load`, `info.reset` show the concrete execution path.
**CN:** Function `finalize_layerwise_processing` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Apply processing to any layers which were not layerwise processed during loading. 像 `hasattr`, `model.modules`, `get_layerwise_info`, `info.can_load`, `info.reset` 这样的关键调用展示了该代码块的具体执行路径。

### Function `finalize_layerwise_reload` (lines 274-275)
```python
def finalize_layerwise_reload(*args, **kwargs):
    finalize_layerwise_processing(*args, **kwargs)
```
**EN:** Function `finalize_layerwise_reload` handles loading or retrieval of external/internal data. Key calls such as `finalize_layerwise_processing` show the concrete execution path.
**CN:** Function `finalize_layerwise_reload` 负责加载或获取外部/内部数据。 像 `finalize_layerwise_processing` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_finalize_attention_layer` (lines 278-292)
```python
def _finalize_attention_layer(
    layer: torch.nn.Module, info: LayerReloadingInfo, model_config: ModelConfig
) -> None:
    if info.load_numel > 0 and info.kernel_tensors is not None:
        # Reload with new scale weights from checkpoint
        _place_kernel_tensors(layer, info)
        _reload_attention_scales(layer, info)
    elif info.load_numel > 0 or info.kernel_tensors is None:
        raise ValueError(
            "Layerwise loading of attention layers is not supported. "
            "Attention must always process after linears."
        )
    else:
        _place_kernel_tensors(layer, info)
    layer.process_weights_after_loading(model_config.dtype)
```
**EN:** Function `_finalize_attention_layer` provides a reusable helper around the module's main workflow. Key calls such as `_place_kernel_tensors`, `_reload_attention_scales`, `ValueError`, `layer.process_weights_after_loading` show the concrete execution path.
**CN:** Function `_finalize_attention_layer` 为模块主流程提供可复用的辅助逻辑。 像 `_place_kernel_tensors`, `_reload_attention_scales`, `ValueError`, `layer.process_weights_after_loading` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_reload_attention_scales` (lines 295-317)
```python
def _reload_attention_scales(layer: torch.nn.Module, info: LayerReloadingInfo) -> None:
    """Load and process attention scale weights (k_scale, v_scale, etc.)
    during reload.

    Assumes dtype/shapes of attention tensors do not change during
    processing, since we use .data.copy_() to preserve kernel tensor
    references."""
    quant_method = getattr(layer, "quant_method", None)
    if quant_method is None:
        return

    # Re-create scale Parameters with sentinel values so unloaded scales
    # are correctly detected by process_weights_after_loading
    quant_method.create_weights(layer)

    for name, args in info.loaded_weights:
        param = getattr(layer, name)
        args.arguments["param"] = param
        _get_weight_loader(param)(*args.args, **args.kwargs)

    quant_method.process_weights_after_loading(layer)

    _copy_and_restore_kernel_tensors(layer, info)
```
**EN:** Function `_reload_attention_scales` handles loading or retrieval of external/internal data. The docstring highlights: Load and process attention scale weights (k_scale, v_scale, etc.) during reload. Key calls such as `getattr`, `quant_method.create_weights`, `_get_weight_loader`, `quant_method.process_weights_after_loading`, `_copy_and_restore_kernel_tensors` show the concrete execution path.
**CN:** Function `_reload_attention_scales` 负责加载或获取外部/内部数据。 文档字符串强调：Load and process attention scale weights (k_scale, v_scale, etc.) during reload. 像 `getattr`, `quant_method.create_weights`, `_get_weight_loader`, `quant_method.process_weights_after_loading`, `_copy_and_restore_kernel_tensors` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_layerwise_process` (lines 320-359)
```python
def _layerwise_process(layer: torch.nn.Module, info: LayerReloadingInfo):
    """
    Finalize layer loading after all weights have been buffered.

    This function:
    1. Materializes the layer onto the target device
    2. Loads all buffered weights
    3. Runs quantization processing if applicable
    4. Copies processed values back to original tensor storage
    """
    # Materialize layer tensors onto device
    materialize_layer(layer, info)

    # Reset online quantization flag so process_weights_after_loading
    # will run again during reload
    if hasattr(layer, "_already_called_process_weights_after_loading"):
        delattr(layer, "_already_called_process_weights_after_loading")

    # Unwrap layerwise loading wrappers
    for param in get_layer_tensors(layer).values():
        param.weight_loader = _get_original_loader(param)

    # Load all buffered weights into materialized layer (using original loaders)
    for name, args in info.loaded_weights:
        param = getattr(layer, name)
    # ... omitted for brevity ...
    info.reset()
    logger.debug("%s: Processed", layer.__class__.__name__)
```
**EN:** Function `_layerwise_process` provides a reusable helper around the module's main workflow. The docstring highlights: Finalize layer loading after all weights have been buffered. Key calls such as `materialize_layer`, `hasattr`, `delattr`, `get_layer_tensors(layer).values`, `get_layer_tensors` show the concrete execution path.
**CN:** Function `_layerwise_process` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Finalize layer loading after all weights have been buffered. 像 `materialize_layer`, `hasattr`, `delattr`, `get_layer_tensors(layer).values`, `get_layer_tensors` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_get_original_loader` (lines 362-368)
```python
def _get_original_loader(tensor: torch.Tensor) -> Callable:
    """Return the weight loader with any layerwise wrappers removed"""
    loader = _get_weight_loader(tensor)
    while loader.__name__ == "online_process_loader":
        loader = loader.__wrapped__  # type: ignore[union-attr]

    return loader
```
**EN:** Function `_get_original_loader` handles loading or retrieval of external/internal data. The docstring highlights: Return the weight loader with any layerwise wrappers removed Key calls such as `_get_weight_loader` show the concrete execution path.
**CN:** Function `_get_original_loader` 负责加载或获取外部/内部数据。 文档字符串强调：Return the weight loader with any layerwise wrappers removed 像 `_get_weight_loader` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_get_weight_loader` (lines 371-372)
```python
def _get_weight_loader(tensor: torch.Tensor):
    return getattr(tensor, "weight_loader", default_weight_loader)
```
**EN:** Function `_get_weight_loader` handles loading or retrieval of external/internal data. Key calls such as `getattr` show the concrete execution path.
**CN:** Function `_get_weight_loader` 负责加载或获取外部/内部数据。 像 `getattr` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_copy_and_restore_kernel_tensors` (lines 375-387)
```python
def _copy_and_restore_kernel_tensors(layer: torch.nn.Module, info: LayerReloadingInfo):
    """Copy processed values into original kernel tensor storage and restore
    kernel tensor references on the layer. Preserves cudagraph references."""
    assert info.kernel_tensors is not None
    parameters, buffers = info.kernel_tensors
    for name, param in parameters.items():
        param.data.copy_(getattr(layer, name))
    for name, buffer in buffers.items():
        if name not in layer._buffers:
            continue
        buffer.data.copy_(getattr(layer, name))

    _place_kernel_tensors(layer, info)
```
**EN:** Function `_copy_and_restore_kernel_tensors` provides a reusable helper around the module's main workflow. The docstring highlights: Copy processed values into original kernel tensor storage and restore kernel tensor references on the layer. Key calls such as `parameters.items`, `param.data.copy_`, `getattr`, `buffers.items`, `buffer.data.copy_` show the concrete execution path.
**CN:** Function `_copy_and_restore_kernel_tensors` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Copy processed values into original kernel tensor storage and restore kernel tensor references on the layer. 像 `parameters.items`, `param.data.copy_`, `getattr`, `buffers.items`, `buffer.data.copy_` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_place_kernel_tensors` (lines 390-399)
```python
def _place_kernel_tensors(layer: torch.nn.Module, info: LayerReloadingInfo):
    for name in get_layer_tensors(layer):
        delattr(layer, name)

    assert info.kernel_tensors is not None
    parameters, buffers = info.kernel_tensors
    for name, param in parameters.items():
        layer.register_parameter(name, param)
    for name, buffer in buffers.items():
        layer.register_buffer(name, buffer)
```
**EN:** Function `_place_kernel_tensors` provides a reusable helper around the module's main workflow. Key calls such as `get_layer_tensors`, `delattr`, `parameters.items`, `layer.register_parameter`, `buffers.items` show the concrete execution path.
**CN:** Function `_place_kernel_tensors` 为模块主流程提供可复用的辅助逻辑。 像 `get_layer_tensors`, `delattr`, `parameters.items`, `layer.register_parameter`, `buffers.items` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import inspect`, `from collections.abc import Callable`, `from functools import wraps`, `from weakref import WeakKeyDictionary, WeakSet`
- **Third-party / 第三方**: `import torch`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import ModelConfig`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.attention import Attention, MLAAttention`, `from vllm.model_executor.layers.quantization.base_config import QuantizeMethodBase`, `from vllm.model_executor.model_loader.weight_utils import default_weight_loader`, `from .meta import SKIP_TENSORS, capture_layer_to_meta, get_numel_loaded, materialize_layer, restore_layer_on_meta`, `from .types import LayerReloadingInfo`, `from .utils import get_info_size, get_layer_params_buffers, get_layer_size, get_layer_tensors, has_device_tensors`
