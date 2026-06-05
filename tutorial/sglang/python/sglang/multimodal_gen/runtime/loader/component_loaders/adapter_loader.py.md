# adapter_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/loader/component_loaders/adapter_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the component loading layer. It centers on `AdapterLoader`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于组件加载层。它围绕 `AdapterLoader` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: module setup and imports / 模块初始化与导入
```python
from safetensors.torch import load_file as safetensors_load_file

from sglang.multimodal_gen.configs.models.adapter.ltx_2_connector import (
    LTX2ConnectorConfig,
)
from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.loader.component_loaders.component_loader import (
    ComponentLoader,
)
from sglang.multimodal_gen.runtime.loader.utils import (
    _list_safetensors_files,
    set_default_torch_dtype,
    skip_init_modules,
)
from sglang.multimodal_gen.runtime.models.registry import ModelRegistry
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.hf_diffusers_utils import (
    get_diffusers_component_config,
)
from sglang.multimodal_gen.utils import PRECISION_TO_TYPE
```
**EN:** This block establishes the module context and imports `safetensors.torch`, `sglang.multimodal_gen.configs.models.adapter.ltx_2_connector`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`, `sglang.multimodal_gen.runtime.loader.utils`, and `sglang.multimodal_gen.runtime.models.registry`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `safetensors.torch`、`sglang.multimodal_gen.configs.models.adapter.ltx_2_connector`、`sglang.multimodal_gen.runtime.distributed`、`sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`、`sglang.multimodal_gen.runtime.loader.utils` 和 `sglang.multimodal_gen.runtime.models.registry`。这些依赖为后续实现提供所需符号。

### Lines 23-30: `AdapterLoader` class overview / `AdapterLoader` 类概览
```python
class AdapterLoader(ComponentLoader):
    """Loader for small adapter-style modules (e.g., LTX-2 connectors).

    This loader intentionally avoids FSDP sharding and just:
    1) Instantiates the module from `config.json`.
    2) Loads a single safetensors state_dict.
    """
```
**EN:** This block defines class `AdapterLoader`. Loader for small adapter-style modules (e.g., LTX-2 connectors). This loader intentionally avoids FSDP sharding and just: 1) Instantiates the module from `config.json`. It inherits from `ComponentLoader`.
**CN:** 该代码块定义了类 `AdapterLoader`。 它用于封装 adapter loader 相关行为。 它继承自 `ComponentLoader`。

### Lines 31-32: supporting statements / 辅助语句
```python
    component_names = ["connectors"]
    expected_library = "diffusers"
```
**EN:** This block gathers supporting statements inside `AdapterLoader`. It updates names such as `component_names`, and `expected_library`.
**CN:** 该代码块汇集了位于 `AdapterLoader` 内部的辅助语句。 它会更新 `component_names` 和 `expected_library` 等名称。

### Lines 34-74: `load_customized` implementation / `load_customized` 实现
```python
    def load_customized(
        self, component_model_path: str, server_args: ServerArgs, *args
    ):
        config = get_diffusers_component_config(component_path=component_model_path)

        cls_name = config.pop("_class_name", None)
        if cls_name is None:
            raise ValueError(
                "Model config does not contain a _class_name attribute. "
                "Only diffusers format is supported."
            )

        config.pop("_diffusers_version", None)
        config.pop("_name_or_path", None)

        server_args.model_paths["connectors"] = component_model_path

        model_cls, _ = ModelRegistry.resolve_model_cls(cls_name)

        target_device = get_local_torch_device()
        default_dtype = PRECISION_TO_TYPE[server_args.pipeline_config.dit_precision]

        with set_default_torch_dtype(default_dtype), skip_init_modules():
            connector_cfg = LTX2ConnectorConfig()
            connector_cfg.update_model_arch(config)
            model = model_cls(connector_cfg).to(
                device=target_device, dtype=default_dtype
            )

        safetensors_list = _list_safetensors_files(component_model_path)
        if not safetensors_list:
            raise ValueError(f"No safetensors files found in {component_model_path}")
        if len(safetensors_list) != 1:
            raise ValueError(
                f"Found {len(safetensors_list)} safetensors files in {component_model_path}, expected 1"
            )

        loaded = safetensors_load_file(safetensors_list[0])
        model.load_state_dict(loaded, strict=False)

        return model
```
**EN:** This block defines method `load_customized` on `AdapterLoader`. It loads customized. Key calls include `get_diffusers_component_config`, `config.pop`, `ModelRegistry.resolve_model_cls`, `get_local_torch_device`, and `_list_safetensors_files`. The implementation branches on conditions, uses context-managed resources. Parameters such as `component_model_path`, and `server_args` drive the behavior in this section.
**CN:** 该代码块定义了 `AdapterLoader` 的方法 `load_customized`。 它用于加载customized。 关键调用包括 `get_diffusers_component_config`、`config.pop`、`ModelRegistry.resolve_model_cls`、`get_local_torch_device` 和 `_list_safetensors_files`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `component_model_path` 和 `server_args` 等参数驱动。

## Key Concepts / 关键概念
- `AdapterLoader`: Loader for small adapter-style modules (e.g., LTX-2 connectors). / 核心类，用于封装 adapter loader 相关行为。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `safetensors.torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.adapter.ltx_2_connector`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`, `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.models.registry`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 74
