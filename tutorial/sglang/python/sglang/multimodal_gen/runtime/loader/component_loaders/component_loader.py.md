# component_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/loader/component_loaders/component_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the component loading layer. It centers on `ComponentLoader`, `ImageProcessorLoader`, and `AutoProcessorLoader`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于组件加载层。它围绕 `ComponentLoader`、`ImageProcessorLoader` 和 `AutoProcessorLoader` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 5-44: module setup and imports / 模块初始化与导入
```python
import importlib
import os
import pkgutil
import traceback
from abc import ABC
from typing import Any, Type

import torch
from diffusers import AutoModel
from torch import nn
from transformers import AutoImageProcessor, AutoProcessor, AutoTokenizer

from sglang.multimodal_gen.configs.models import ModelConfig
from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.layers.attention.selector import (
    component_attn_backend_context_manager,
    get_component_attn_backend_context,
)
from sglang.multimodal_gen.runtime.loader.utils import (
    _normalize_component_type,
    component_name_to_loader_cls,
    get_memory_usage_of_component,
)
from sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload import (
    configure_layerwise_offload_modules,
    is_layerwise_offloaded_module,
)
from sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload_components import (
    LAYERWISE_OFFLOAD_ALL_COMPONENTS,
    LAYERWISE_OFFLOAD_DIT_GROUP,
    layerwise_component_matches_any_selection,
    normalize_layerwise_offload_components,
)
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.hf_diffusers_utils import (
    get_hf_config,
    prepare_diffusers_component_path_for_loading,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `importlib`, `os`, `pkgutil`, `traceback`, `abc`, and `typing`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `importlib`、`os`、`pkgutil`、`traceback`、`abc` 和 `typing`。这些依赖为后续实现提供所需符号。

### Lines 46-46: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 49-67: `_load_auto_tokenizer_with_roberta_processing_compat` implementation / `_load_auto_tokenizer_with_roberta_processing_compat` 实现
```python
def _load_auto_tokenizer_with_roberta_processing_compat(*args, **kwargs):
    from tokenizers import processors

    roberta_processing = processors.RobertaProcessing

    def roberta_processing_compat(*processor_args, **processor_kwargs):
        if "sep" in processor_kwargs and "cls" in processor_kwargs:
            sep = processor_kwargs.pop("sep")
            cls_token = processor_kwargs.pop("cls")
            return roberta_processing(
                sep, cls_token, *processor_args, **processor_kwargs
            )
        return roberta_processing(*processor_args, **processor_kwargs)

    processors.RobertaProcessing = roberta_processing_compat
    try:
        return AutoTokenizer.from_pretrained(*args, **kwargs)
    finally:
        processors.RobertaProcessing = roberta_processing
```
**EN:** This block defines function `_load_auto_tokenizer_with_roberta_processing_compat`. It loads auto tokenizer with roberta processing compat. Key calls include `roberta_processing`, `AutoTokenizer.from_pretrained`, and `processor_kwargs.pop`. The implementation branches on conditions, handles exceptional paths.
**CN:** 该代码块定义了函数 `_load_auto_tokenizer_with_roberta_processing_compat`。 它用于加载auto tokenizer with roberta processing compat。 关键调用包括 `roberta_processing`、`AutoTokenizer.from_pretrained` 和 `processor_kwargs.pop`。 实现中包含条件分支，处理异常路径。

### Lines 70-73: `ComponentLoader` class overview / `ComponentLoader` 类概览
```python
class ComponentLoader(ABC):
    """Base class for loading a specific type of model component."""

    # the list of possible name of the component in model_index.json, e.g., scheduler
```
**EN:** This block defines class `ComponentLoader`. Base class for loading a specific type of model component. It inherits from `ABC`.
**CN:** 该代码块定义了类 `ComponentLoader`。 它用于封装 component loader 相关行为。 它继承自 `ABC`。

### Lines 74-79: supporting statements / 辅助语句
```python
    component_names: list[str] = []

    # diffusers or transformers
    expected_library: str = ""

    _loaders_registered = False
```
**EN:** This block gathers supporting statements inside `ComponentLoader`. It updates names such as `component_names`, `expected_library`, and `_loaders_registered`.
**CN:** 该代码块汇集了位于 `ComponentLoader` 内部的辅助语句。 它会更新 `component_names`、`expected_library` 和 `_loaders_registered` 等名称。

### Lines 81-87: `__init_subclass__` implementation / `__init_subclass__` 实现
```python
    def __init_subclass__(cls, **kwargs):
        """
        register loaders, called when subclass is imported
        """
        super().__init_subclass__(**kwargs)
        for component_name in cls.component_names:
            component_name_to_loader_cls[component_name] = cls
```
**EN:** This block defines method `__init_subclass__` on `ComponentLoader`. register loaders, called when subclass is imported Key calls include `super.__init_subclass__`, and `super`. The implementation iterates over collections or steps.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `__init_subclass__`。 它用于初始化subclass。 关键调用包括 `super.__init_subclass__` 和 `super`。 实现中会遍历集合或步骤。

### Lines 89-91: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, device=None) -> None:
        self.device = device
        self.component_architecture: str | None = None
```
**EN:** This block defines method `__init__` on `ComponentLoader`. It initializes the instance state. Parameters such as `device` drive the behavior in this section.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `device` 等参数驱动。

### Lines 93-97: `should_offload` implementation / `should_offload` 实现
```python
    def should_offload(
        self, server_args: ServerArgs, model_config: ModelConfig | None = None
    ):
        # not offload by default
        return False
```
**EN:** This block defines method `should_offload` on `ComponentLoader`. It determines whether to offload. Parameters such as `server_args`, and `model_config` drive the behavior in this section.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `should_offload`。 它用于判断是否offload。 本段逻辑主要由 `server_args` 和 `model_config` 等参数驱动。

### Lines 99-107: `target_device` implementation / `target_device` 实现
```python
    def target_device(self, should_offload):
        if should_offload:
            return (
                torch.device("mps")
                if current_platform.is_mps()
                else torch.device("cpu")
            )
        else:
            return get_local_torch_device()
```
**EN:** This block defines method `target_device` on `ComponentLoader`. It handles target device logic. Key calls include `get_local_torch_device`, `current_platform.is_mps`, and `torch.device`. The implementation branches on conditions. Parameters such as `should_offload` drive the behavior in this section.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `target_device`。 它用于处理 target device 相关逻辑。 关键调用包括 `get_local_torch_device`、`current_platform.is_mps` 和 `torch.device`。 实现中包含条件分支。 本段逻辑主要由 `should_offload` 等参数驱动。

### Lines 109-112: `customized_load_kwargs_for_component` implementation / `customized_load_kwargs_for_component` 实现
```python
    def customized_load_kwargs_for_component(
        self, _server_args: ServerArgs, _component_name: str
    ) -> dict[str, Any]:
        return {}
```
**EN:** This block defines method `customized_load_kwargs_for_component` on `ComponentLoader`. It handles customized load kwargs for component logic. Parameters such as `_server_args`, and `_component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `customized_load_kwargs_for_component`。 它用于处理 customized load kwargs for component 相关逻辑。 本段逻辑主要由 `_server_args` 和 `_component_name` 等参数驱动。

### Lines 114-132: `_is_component_set_as_layerwise_load` implementation / `_is_component_set_as_layerwise_load` 实现
```python
    @staticmethod
    def _is_component_set_as_layerwise_load(
        server_args: ServerArgs, component_name: str
    ) -> bool:
        """if a component should be loaded in a layerwise-fashion"""
        selected_component_names = normalize_layerwise_offload_components(
            server_args.layerwise_offload_components
        )
        if selected_component_names is None:
            return False
        selected_component_names = set(selected_component_names)
        if LAYERWISE_OFFLOAD_ALL_COMPONENTS in selected_component_names:
            return True
        explicit_component_names = selected_component_names - {
            LAYERWISE_OFFLOAD_DIT_GROUP
        }
        return layerwise_component_matches_any_selection(
            component_name, explicit_component_names
        )
```
**EN:** This block defines method `_is_component_set_as_layerwise_load` on `ComponentLoader`. if a component should be loaded in a layerwise-fashion Key calls include `normalize_layerwise_offload_components`, `set`, and `layerwise_component_matches_any_selection`. The implementation branches on conditions. Parameters such as `server_args`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `_is_component_set_as_layerwise_load`。 它用于处理 is component set as layerwise load 相关逻辑。 关键调用包括 `normalize_layerwise_offload_components`、`set` 和 `layerwise_component_matches_any_selection`。 实现中包含条件分支。 本段逻辑主要由 `server_args` 和 `component_name` 等参数驱动。

### Lines 134-168: `_maybe_configure_layerwise_after_startup_cpu_staging` implementation / `_maybe_configure_layerwise_after_startup_cpu_staging` 实现
```python
    def _maybe_configure_layerwise_after_startup_cpu_staging(
        self,
        component: AutoModel,
        server_args: ServerArgs,
        component_name: str,
        load_kwargs: dict[str, Any],
    ) -> AutoModel:
        if not load_kwargs.get("cpu_offload_flag"):
            return component
        if not isinstance(component, nn.Module):
            return component

        # try to configure layerwise-offload with the component
        configured_components = configure_layerwise_offload_modules(
            {component_name: component},
            server_args,
            component_names=server_args.layerwise_offload_components,
            warn_missing=False,
        )
        if is_layerwise_offloaded_module(component):
            logger.info(
                "Configured layerwise offload for %s immediately after startup CPU staging",
                component_name,
            )
            return component

        logger.warning(
            "Layerwise startup CPU staging was requested for %s, but the loaded "
            "module did not enable layerwise offload. Moving it to GPU.",
            component_name,
        )
        # ensures the module is on GPU
        if component_name in configured_components:
            return component
        return component.to(get_local_torch_device())
```
**EN:** This block defines method `_maybe_configure_layerwise_after_startup_cpu_staging` on `ComponentLoader`. It handles maybe configure layerwise after startup cpu staging logic. Key calls include `configure_layerwise_offload_modules`, `is_layerwise_offloaded_module`, `logger.warning`, `component.to`, and `load_kwargs.get`. The implementation branches on conditions. Parameters such as `component`, `server_args`, `component_name`, and `load_kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `_maybe_configure_layerwise_after_startup_cpu_staging`。 它用于处理 maybe configure layerwise after startup cpu staging 相关逻辑。 关键调用包括 `configure_layerwise_offload_modules`、`is_layerwise_offloaded_module`、`logger.warning`、`component.to` 和 `load_kwargs.get`。 实现中包含条件分支。 本段逻辑主要由 `component`、`server_args`、`component_name` 和 `load_kwargs` 等参数驱动。

### Lines 170-264: `load` implementation / `load` 实现
```python
    def load(
        self,
        component_model_path: str,
        server_args: ServerArgs,
        component_name: str,
        transformers_or_diffusers: str,
    ) -> tuple[AutoModel, float]:
        """
        Template method that standardizes logging around the core load implementation.
        The priority of loading method is:
            1. load customized component
            2. load native diffusers/transformers component
        If all of the above methods failed, an error will be thrown

        """
        gpu_mem_before_loading = current_platform.get_available_gpu_memory()
        logger.info(
            "Loading %s from %s. avail mem: %.2f GB",
            component_name,
            component_model_path,
            gpu_mem_before_loading,
        )
        attn_backend = None
        component_attn_name = None
        if get_component_attn_backend_context() is None:
            attn_backend, matched_backend_key = (
                server_args.resolve_component_attention_backend(component_name)
            )
            component_attn_name = matched_backend_key or component_name
            if attn_backend is not None:
                logger.info(
                    "Using %s backend for component: %s",
                    attn_backend.name.lower(),
                    matched_backend_key,
                )
        try:
            with component_attn_backend_context_manager(
                attn_backend, component_name=component_attn_name
            ):
                load_kwargs = self.customized_load_kwargs_for_component(
                    server_args, component_name
                )
                component = self.load_customized(
                    component_model_path, server_args, component_name, **load_kwargs
                )
                # configure layerwise to make enough VRAM headroom
                component = self._maybe_configure_layerwise_after_startup_cpu_staging(
                    component, server_args, component_name, load_kwargs
                )
            source = "sgl-diffusion"
        except Exception as e:
            if "Unsupported model architecture" in str(e):
                logger.info(
                    f"Component: {component_name} doesn't have a customized version yet, using native version"
                )
            else:
                traceback.print_exc()
                logger.error(
                    f"Error while loading customized {component_name}, falling back to native version"
                )
            # fallback to native version
            with component_attn_backend_context_manager(
                attn_backend, component_name=component_attn_name
            ):
                component = self.load_native(
                    component_model_path, server_args, transformers_or_diffusers
                )
            should_offload = self.should_offload(server_args)
            target_device = self.target_device(should_offload)
            component = component.to(device=target_device)
            source = "native"
            logger.warning(
                "Native component %s: %s is loaded, performance may be sub-optimal",
                component_name,
                component.__class__.__name__,
            )

        if component is None:
            logger.error("Load %s failed", component_name)
            consumed = 0.0
        else:
            if isinstance(component, nn.Module):
                component = component.eval()
            current_gpu_mem = current_platform.get_available_gpu_memory()
            model_size = get_memory_usage_of_component(component) or "NA"
            consumed = gpu_mem_before_loading - current_gpu_mem
            logger.info(
                f"Loaded %s: %s ({source} version). model size: %s GB, consumed GPU mem: %.2f GB, avail GPU mem: %.2f GB",
                component_name,
                component.__class__.__name__,
                model_size,
                consumed,
                current_gpu_mem,
            )
        return component, consumed
```
**EN:** This block defines method `load` on `ComponentLoader`. Template method that standardizes logging around the core load implementation. The priority of loading method is: 1. Key calls include `current_platform.get_available_gpu_memory`, `logger.info`, `get_component_attn_backend_context`, `server_args.resolve_component_attention_backend`, and `logger.error`. The implementation branches on conditions, handles exceptional paths, uses context-managed resources. Parameters such as `component_model_path`, `server_args`, `component_name`, and `transformers_or_diffusers` drive the behavior in this section.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `load`。 它用于加载函数。 关键调用包括 `current_platform.get_available_gpu_memory`、`logger.info`、`get_component_attn_backend_context`、`server_args.resolve_component_attention_backend` 和 `logger.error`。 实现中包含条件分支，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `component_model_path`、`server_args`、`component_name` 和 `transformers_or_diffusers` 等参数驱动。

### Lines 266-301: `load_native` implementation / `load_native` 实现
```python
    def load_native(
        self,
        component_model_path: str,
        server_args: ServerArgs,
        transformers_or_diffusers: str,
    ) -> AutoModel:
        """
        Load the component using the native library (transformers/diffusers).
        """
        if transformers_or_diffusers == "transformers":
            from transformers import AutoModel

            config = get_hf_config(
                component_model_path,
                trust_remote_code=server_args.trust_remote_code,
                revision=server_args.revision,
            )
            return AutoModel.from_pretrained(
                component_model_path,
                config=config,
                trust_remote_code=server_args.trust_remote_code,
                revision=server_args.revision,
            )
        elif transformers_or_diffusers == "diffusers":
            from diffusers import AutoModel

            component_model_path = prepare_diffusers_component_path_for_loading(
                component_model_path
            )
            return AutoModel.from_pretrained(
                component_model_path,
                revision=server_args.revision,
                trust_remote_code=server_args.trust_remote_code,
            )
        else:
            raise ValueError(f"Unsupported library: {transformers_or_diffusers}")
```
**EN:** This block defines method `load_native` on `ComponentLoader`. Load the component using the native library (transformers/diffusers). Key calls include `get_hf_config`, `AutoModel.from_pretrained`, `prepare_diffusers_component_path_for_loading`, and `ValueError`. The implementation branches on conditions. Parameters such as `component_model_path`, `server_args`, and `transformers_or_diffusers` drive the behavior in this section.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `load_native`。 它用于加载native。 关键调用包括 `get_hf_config`、`AutoModel.from_pretrained`、`prepare_diffusers_component_path_for_loading` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `component_model_path`、`server_args` 和 `transformers_or_diffusers` 等参数驱动。

### Lines 303-311: `load_customized` implementation / `load_customized` 实现
```python
    def load_customized(
        self, component_model_path: str, server_args: ServerArgs, component_name: str
    ):
        """
        Load the customized version component, implemented and optimized in SGL-diffusion
        """
        raise NotImplementedError(
            f"load_customized not implemented for {self.__class__.__name__}"
        )
```
**EN:** This block defines method `load_customized` on `ComponentLoader`. Load the customized version component, implemented and optimized in SGL-diffusion Key calls include `NotImplementedError`. Parameters such as `component_model_path`, `server_args`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `load_customized`。 它用于加载customized。 关键调用包括 `NotImplementedError`。 本段逻辑主要由 `component_model_path`、`server_args` 和 `component_name` 等参数驱动。

### Lines 313-336: `_ensure_loaders_registered` implementation / `_ensure_loaders_registered` 实现
```python
    @classmethod
    def _ensure_loaders_registered(cls):
        """
        avoid multiple registration
        """
        if cls._loaders_registered:
            return

        package_dir = os.path.dirname(__file__)
        package_name = (
            __package__
            or "sglang.multimodal_gen.runtime.loader.component_loaders.component_loaders"
        )

        for _, name, _ in pkgutil.iter_modules([package_dir]):
            # skip importing self to avoid circular dependency issues
            if name == "component_loader":
                continue
            try:
                importlib.import_module(f".{name}", package=package_name)
            except ImportError as e:
                logger.warning(f"Failed to import loader component {name}: {e}")

        cls._loaders_registered = True
```
**EN:** This block defines method `_ensure_loaders_registered` on `ComponentLoader`. avoid multiple registration Key calls include `os.path.dirname`, `pkgutil.iter_modules`, `importlib.import_module`, and `logger.warning`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `_ensure_loaders_registered`。 它用于处理 ensure loaders registered 相关逻辑。 关键调用包括 `os.path.dirname`、`pkgutil.iter_modules`、`importlib.import_module` 和 `logger.warning`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。

### Lines 338-362: `resolve_transformers_or_diffusers` implementation / `resolve_transformers_or_diffusers` 实现
```python
    @classmethod
    def resolve_transformers_or_diffusers(
        self, transformers_or_diffusers: str, component_name: str
    ) -> str:
        # NOTE(FlamingoPg): special for LTX-2 models
        if component_name == "vocoder" or component_name == "connectors":
            transformers_or_diffusers = "diffusers"

        # NOTE(CloudRipple): special for MOVA models
        # TODO(CloudRipple): remove most of these special cases after unifying the loading logic
        if component_name in [
            "audio_vae",
            "audio_dit",
            "dual_tower_bridge",
            "video_dit",
        ]:
            transformers_or_diffusers = "diffusers"

        if (
            component_name == "scheduler"
            and transformers_or_diffusers == "mova.diffusion.schedulers.flow_match_pair"
        ):
            transformers_or_diffusers = "diffusers"

        return transformers_or_diffusers
```
**EN:** This block defines method `resolve_transformers_or_diffusers` on `ComponentLoader`. It resolves transformers or diffusers. The implementation branches on conditions. Parameters such as `transformers_or_diffusers`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `resolve_transformers_or_diffusers`。 它用于解析transformers or diffusers。 实现中包含条件分支。 本段逻辑主要由 `transformers_or_diffusers` 和 `component_name` 等参数驱动。

### Lines 364-405: `for_component_type` implementation / `for_component_type` 实现
```python
    @classmethod
    def for_component_type(
        cls,
        component_name: str,
        transformers_or_diffusers: str,
        component_architecture: str | None = None,
    ) -> "ComponentLoader":
        """
        Factory method to create a component loader for a specific component type.

        Args:
            component_name: Type of component (e.g., "vae", "text_encoder", "transformer", "scheduler")
            transformers_or_diffusers: Whether the component is from transformers or diffusers
        """
        cls._ensure_loaders_registered()

        # Map of component types to their loader classes and expected library
        component_name = _normalize_component_type(component_name)

        transformers_or_diffusers = cls.resolve_transformers_or_diffusers(
            transformers_or_diffusers, component_name
        )

        if component_name in component_name_to_loader_cls:
            loader_cls: Type[ComponentLoader] = component_name_to_loader_cls[
                component_name
            ]
            expected_library = loader_cls.expected_library
            # Assert that the library matches what's expected for this component type
            assert (
                transformers_or_diffusers == expected_library
            ), f"{component_name} must be loaded from {expected_library}, got {transformers_or_diffusers}"
            loader = loader_cls()
            loader.component_architecture = component_architecture
            return loader

        # For unknown component types, use a generic loader
        logger.warning(
            "No specific loader found for component type: %s. Using generic loader.",
            component_name,
        )
        return GenericComponentLoader(transformers_or_diffusers, component_architecture)
```
**EN:** This block defines method `for_component_type` on `ComponentLoader`. Factory method to create a component loader for a specific component type. Args: component_name: Type of component (e.g., "vae", "text_encoder", "transformer", "scheduler") transformers_or_diffusers: Whether the component is from transformers or diffusers Key calls include `cls._ensure_loaders_registered`, `_normalize_component_type`, `cls.resolve_transformers_or_diffusers`, `logger.warning`, and `GenericComponentLoader`. The implementation branches on conditions. Parameters such as `component_name`, `transformers_or_diffusers`, and `component_architecture` drive the behavior in this section.
**CN:** 该代码块定义了 `ComponentLoader` 的方法 `for_component_type`。 它用于处理 for component type 相关逻辑。 关键调用包括 `cls._ensure_loaders_registered`、`_normalize_component_type`、`cls.resolve_transformers_or_diffusers`、`logger.warning` 和 `GenericComponentLoader`。 实现中包含条件分支。 本段逻辑主要由 `component_name`、`transformers_or_diffusers` 和 `component_architecture` 等参数驱动。

### Lines 408-410: `ImageProcessorLoader` class overview / `ImageProcessorLoader` 类概览
```python
class ImageProcessorLoader(ComponentLoader):
    """Loader for image processor."""
```
**EN:** This block defines class `ImageProcessorLoader`. Loader for image processor. It inherits from `ComponentLoader`.
**CN:** 该代码块定义了类 `ImageProcessorLoader`。 它用于封装 image processor loader 相关行为。 它继承自 `ComponentLoader`。

### Lines 411-412: supporting statements / 辅助语句
```python
    component_names = ["image_processor"]
    expected_library = "transformers"
```
**EN:** This block gathers supporting statements inside `ImageProcessorLoader`. It updates names such as `component_names`, and `expected_library`.
**CN:** 该代码块汇集了位于 `ImageProcessorLoader` 内部的辅助语句。 它会更新 `component_names` 和 `expected_library` 等名称。

### Lines 414-417: `load_customized` implementation / `load_customized` 实现
```python
    def load_customized(
        self, component_model_path: str, server_args: ServerArgs, component_name: str
    ) -> Any:
        return AutoImageProcessor.from_pretrained(component_model_path, use_fast=True)
```
**EN:** This block defines method `load_customized` on `ImageProcessorLoader`. It loads customized. Key calls include `AutoImageProcessor.from_pretrained`. Parameters such as `component_model_path`, `server_args`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `ImageProcessorLoader` 的方法 `load_customized`。 它用于加载customized。 关键调用包括 `AutoImageProcessor.from_pretrained`。 本段逻辑主要由 `component_model_path`、`server_args` 和 `component_name` 等参数驱动。

### Lines 420-422: `AutoProcessorLoader` class overview / `AutoProcessorLoader` 类概览
```python
class AutoProcessorLoader(ComponentLoader):
    """Loader for auto processor."""
```
**EN:** This block defines class `AutoProcessorLoader`. Loader for auto processor. It inherits from `ComponentLoader`.
**CN:** 该代码块定义了类 `AutoProcessorLoader`。 它用于封装 auto processor loader 相关行为。 它继承自 `ComponentLoader`。

### Lines 423-424: supporting statements / 辅助语句
```python
    component_names = ["processor"]
    expected_library = "transformers"
```
**EN:** This block gathers supporting statements inside `AutoProcessorLoader`. It updates names such as `component_names`, and `expected_library`.
**CN:** 该代码块汇集了位于 `AutoProcessorLoader` 内部的辅助语句。 它会更新 `component_names` 和 `expected_library` 等名称。

### Lines 426-429: `load_customized` implementation / `load_customized` 实现
```python
    def load_customized(
        self, component_model_path: str, server_args: ServerArgs, component_name: str
    ) -> Any:
        return AutoProcessor.from_pretrained(component_model_path)
```
**EN:** This block defines method `load_customized` on `AutoProcessorLoader`. It loads customized. Key calls include `AutoProcessor.from_pretrained`. Parameters such as `component_model_path`, `server_args`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `AutoProcessorLoader` 的方法 `load_customized`。 它用于加载customized。 关键调用包括 `AutoProcessor.from_pretrained`。 本段逻辑主要由 `component_model_path`、`server_args` 和 `component_name` 等参数驱动。

### Lines 432-434: `TokenizerLoader` class overview / `TokenizerLoader` 类概览
```python
class TokenizerLoader(ComponentLoader):
    """Loader for tokenizers."""
```
**EN:** This block defines class `TokenizerLoader`. Loader for tokenizers. It inherits from `ComponentLoader`.
**CN:** 该代码块定义了类 `TokenizerLoader`。 它用于封装 tokenizer loader 相关行为。 它继承自 `ComponentLoader`。

### Lines 435-436: supporting statements / 辅助语句
```python
    component_names = ["tokenizer"]
    expected_library = "transformers"
```
**EN:** This block gathers supporting statements inside `TokenizerLoader`. It updates names such as `component_names`, and `expected_library`.
**CN:** 该代码块汇集了位于 `TokenizerLoader` 内部的辅助语句。 它会更新 `component_names` 和 `expected_library` 等名称。

### Lines 438-473: `load_customized` implementation / `load_customized` 实现
```python
    def load_customized(
        self, component_model_path: str, server_args: ServerArgs, component_name: str
    ) -> Any:
        # Some pipelines keep the slot name `tokenizer` in model_index.json even
        # when the declared class is a processor. e.g. FLUX.2:
        # `tokenizer: ["transformers", "PixtralProcessor"]`.
        # Honor the declared component class instead of guessing from the slot name.
        if (
            self.component_architecture is not None
            and self.component_architecture.endswith("Processor")
        ):
            return AutoProcessor.from_pretrained(component_model_path)

        # Qwen-Image's model_index declares Qwen2Tokenizer; using the fast class
        # changes text preprocessing and shifts official GT comparisons.
        use_fast = self.component_architecture != "Qwen2Tokenizer"
        try:
            return AutoTokenizer.from_pretrained(
                component_model_path,
                padding_side="right",
                use_fast=use_fast,
            )
        except TypeError as e:
            # tokenizers>=0.21 removed the `cls` kwarg from RobertaProcessing,
            # but some transformers CLIPTokenizer builds still pass it. Fall back
            # to the pure-Python (slow) tokenizer which avoids the rust path.
            if "RobertaProcessing" in str(e) and use_fast:
                logger.warning(
                    "Fast tokenizer failed (%s), retrying with use_fast=False", e
                )
                return _load_auto_tokenizer_with_roberta_processing_compat(
                    component_model_path,
                    padding_side="right",
                    use_fast=False,
                )
            raise
```
**EN:** This block defines method `load_customized` on `TokenizerLoader`. It loads customized. Key calls include `self.component_architecture.endswith`, `AutoProcessor.from_pretrained`, `AutoTokenizer.from_pretrained`, `logger.warning`, and `_load_auto_tokenizer_with_roberta_processing_compat`. The implementation branches on conditions, handles exceptional paths. Parameters such as `component_model_path`, `server_args`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `TokenizerLoader` 的方法 `load_customized`。 它用于加载customized。 关键调用包括 `self.component_architecture.endswith`、`AutoProcessor.from_pretrained`、`AutoTokenizer.from_pretrained`、`logger.warning` 和 `_load_auto_tokenizer_with_roberta_processing_compat`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `component_model_path`、`server_args` 和 `component_name` 等参数驱动。

### Lines 476-478: `GenericComponentLoader` class overview / `GenericComponentLoader` 类概览
```python
class GenericComponentLoader(ComponentLoader):
    """Generic loader for components that don't have a specific loader."""
```
**EN:** This block defines class `GenericComponentLoader`. Generic loader for components that don't have a specific loader. It inherits from `ComponentLoader`.
**CN:** 该代码块定义了类 `GenericComponentLoader`。 它用于封装 generic component loader 相关行为。 它继承自 `ComponentLoader`。

### Lines 479-484: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self, library="transformers", component_architecture: str | None = None
    ) -> None:
        super().__init__()
        self.library = library
        self.component_architecture = component_architecture
```
**EN:** This block defines method `__init__` on `GenericComponentLoader`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `library`, and `component_architecture` drive the behavior in this section.
**CN:** 该代码块定义了 `GenericComponentLoader` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `library` 和 `component_architecture` 等参数驱动。

### Lines 487-491: `PipelineComponentLoader` class overview / `PipelineComponentLoader` 类概览
```python
class PipelineComponentLoader:
    """
    Utility class for loading the components in a pipeline.
    """
```
**EN:** This block defines class `PipelineComponentLoader`. Utility class for loading the components in a pipeline.
**CN:** 该代码块定义了类 `PipelineComponentLoader`。 它用于封装 pipeline component loader 相关行为。

### Lines 492-527: `load_component` implementation / `load_component` 实现
```python
    @staticmethod
    def load_component(
        component_name: str,
        component_model_path: str,
        transformers_or_diffusers: str,
        server_args: ServerArgs,
        component_architecture: str | None = None,
    ):
        """
        Load a pipeline component.

        Args:
            component_name: Name of the component (e.g., "vae", "text_encoder", "transformer", "scheduler")
            component_model_path: Path to the component model
            transformers_or_diffusers: Whether the component is from transformers or diffusers
            component_architecture: the class name of the module
        """

        # Get the appropriate loader for this component type
        loader = ComponentLoader.for_component_type(
            component_name, transformers_or_diffusers, component_architecture
        )

        try:
            # Load the component
            return loader.load(
                component_model_path,
                server_args,
                component_name,
                transformers_or_diffusers,
            )
        except Exception as e:
            logger.error(
                f"Error while loading component: {component_name}, {component_model_path=}"
            )
            raise e
```
**EN:** This block defines method `load_component` on `PipelineComponentLoader`. Load a pipeline component. Args: component_name: Name of the component (e.g., "vae", "text_encoder", "transformer", "scheduler") component_model_path: Path to the component model transformers_or_diffusers: Whether the component is from transformers or diffusers component_architecture: the class name of the module Key calls include `ComponentLoader.for_component_type`, `loader.load`, and `logger.error`. The implementation handles exceptional paths. Parameters such as `component_name`, `component_model_path`, `transformers_or_diffusers`, `server_args`, and `component_architecture` drive the behavior in this section.
**CN:** 该代码块定义了 `PipelineComponentLoader` 的方法 `load_component`。 它用于加载component。 关键调用包括 `ComponentLoader.for_component_type`、`loader.load` 和 `logger.error`。 实现中处理异常路径。 本段逻辑主要由 `component_name`、`component_model_path`、`transformers_or_diffusers`、`server_args` 和 `component_architecture` 等参数驱动。

## Key Concepts / 关键概念
- `_load_auto_tokenizer_with_roberta_processing_compat`: Top-level function that loads auto tokenizer with roberta processing compat. / 顶层函数，用于加载auto tokenizer with roberta processing compat。
- `ComponentLoader`: Base class for loading a specific type of model component. / 核心类，用于封装 component loader 相关行为。
- `ImageProcessorLoader`: Loader for image processor. / 核心类，用于封装 image processor loader 相关行为。
- `AutoProcessorLoader`: Loader for auto processor. / 核心类，用于封装 auto processor loader 相关行为。
- `TokenizerLoader`: Loader for tokenizers. / 核心类，用于封装 tokenizer loader 相关行为。
- `GenericComponentLoader`: Generic loader for components that don't have a specific loader. / 核心类，用于封装 generic component loader 相关行为。
- `PipelineComponentLoader`: Utility class for loading the components in a pipeline. / 核心类，用于封装 pipeline component loader 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `importlib`, `os`, `pkgutil`, `traceback`, `abc`, `typing`
- **Third-party / 第三方依赖**: `torch`, `diffusers`, `transformers`, `tokenizers`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.attention.selector`, `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload_components`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 527
