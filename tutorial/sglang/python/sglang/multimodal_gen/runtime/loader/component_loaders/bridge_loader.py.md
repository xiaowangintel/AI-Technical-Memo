# bridge_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/loader/component_loaders/bridge_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the component loading layer. It centers on `BridgeLoader`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于组件加载层。它围绕 `BridgeLoader` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: module setup and imports / 模块初始化与导入
```python
from copy import deepcopy

import torch

from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.loader.component_loaders.component_loader import (
    ComponentLoader,
)
from sglang.multimodal_gen.runtime.loader.fsdp_load import maybe_load_fsdp_model
from sglang.multimodal_gen.runtime.loader.utils import _list_safetensors_files
from sglang.multimodal_gen.runtime.models.registry import ModelRegistry
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.hf_diffusers_utils import (
    get_diffusers_component_config,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import PRECISION_TO_TYPE
```
**EN:** This block establishes the module context and imports `copy`, `torch`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`, `sglang.multimodal_gen.runtime.loader.fsdp_load`, and `sglang.multimodal_gen.runtime.loader.utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `copy`、`torch`、`sglang.multimodal_gen.runtime.distributed`、`sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`、`sglang.multimodal_gen.runtime.loader.fsdp_load` 和 `sglang.multimodal_gen.runtime.loader.utils`。这些依赖为后续实现提供所需符号。

### Lines 19-19: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 22-24: `BridgeLoader` class overview / `BridgeLoader` 类概览
```python
class BridgeLoader(ComponentLoader):
    """Loader for MOVA dual tower bridge with FSDP support."""
```
**EN:** This block defines class `BridgeLoader`. Loader for MOVA dual tower bridge with FSDP support. It inherits from `ComponentLoader`.
**CN:** 该代码块定义了类 `BridgeLoader`。 它用于封装 bridge loader 相关行为。 它继承自 `ComponentLoader`。

### Lines 25-28: supporting statements / 辅助语句
```python
    pipeline_bridge_config_attr: str = "bridge_config"

    component_names = ["dual_tower_bridge"]
    expected_library = "diffusers"
```
**EN:** This block gathers supporting statements inside `BridgeLoader`. It updates names such as `pipeline_bridge_config_attr`, `component_names`, and `expected_library`.
**CN:** 该代码块汇集了位于 `BridgeLoader` 内部的辅助语句。 它会更新 `pipeline_bridge_config_attr`、`component_names` 和 `expected_library` 等名称。

### Lines 30-105: `load_customized` implementation / `load_customized` 实现
```python
    def load_customized(
        self, component_model_path: str, server_args: ServerArgs, component_name: str
    ):
        config = get_diffusers_component_config(component_path=component_model_path)
        hf_config = deepcopy(config)
        class_name = config.pop("_class_name", None)
        if class_name is None:
            raise ValueError(
                "Model config does not contain a _class_name attribute. "
                "Only diffusers format is supported."
            )
        server_args.model_paths[component_name] = component_model_path

        # Try to get bridge config from pipeline config, fallback to creating one
        bridge_config = getattr(
            server_args.pipeline_config, self.pipeline_bridge_config_attr, None
        )
        if bridge_config is not None:
            bridge_config.update_model_arch(config)
        else:
            # Create a minimal config from hf_config
            from sglang.multimodal_gen.configs.models.bridges.mova_dual_tower import (
                MOVADualTowerConfig,
            )

            bridge_config = MOVADualTowerConfig()
            bridge_config.update_model_arch(config)

        model_cls, _ = ModelRegistry.resolve_model_cls(class_name)

        # Find all safetensors files
        safetensors_list = _list_safetensors_files(component_model_path)
        if not safetensors_list:
            raise ValueError(f"No safetensors files found in {component_model_path}")

        default_dtype = PRECISION_TO_TYPE[server_args.pipeline_config.dit_precision]

        logger.info(
            "Loading %s from %s safetensors files, default_dtype: %s",
            class_name,
            len(safetensors_list),
            default_dtype,
        )

        # Use the FSDP loader when FSDP is requested or shard rules are declared.
        fsdp_shard_conditions = getattr(model_cls, "_fsdp_shard_conditions", None)
        if server_args.use_fsdp_inference or (
            server_args.hsdp_shard_dim is not None and fsdp_shard_conditions
        ):
            # Load with FSDP support
            model = maybe_load_fsdp_model(
                model_cls=model_cls,
                init_params={"config": bridge_config, "hf_config": hf_config},
                weight_dir_list=safetensors_list,
                device=get_local_torch_device(),
                hsdp_replicate_dim=server_args.hsdp_replicate_dim,
                hsdp_shard_dim=server_args.hsdp_shard_dim,
                cpu_offload=server_args.dit_cpu_offload,
                pin_cpu_memory=server_args.pin_cpu_memory,
                fsdp_inference=server_args.use_fsdp_inference,
                param_dtype=default_dtype,
                reduce_dtype=torch.float32,
                output_dtype=None,
                strict=False,
            )
        else:
            # Fallback to simple loading (for non-FSDP or legacy models)
            model = model_cls.from_pretrained(
                component_model_path, torch_dtype=default_dtype
            )
            model = model.to(device=get_local_torch_device(), dtype=default_dtype)

        total_params = sum(p.numel() for p in model.parameters())
        logger.info("Loaded bridge model with %.2fM parameters", total_params / 1e6)

        return model
```
**EN:** This block defines method `load_customized` on `BridgeLoader`. It loads customized. Key calls include `get_diffusers_component_config`, `deepcopy`, `config.pop`, `getattr`, and `ModelRegistry.resolve_model_cls`. The implementation branches on conditions. Parameters such as `component_model_path`, `server_args`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `BridgeLoader` 的方法 `load_customized`。 它用于加载customized。 关键调用包括 `get_diffusers_component_config`、`deepcopy`、`config.pop`、`getattr` 和 `ModelRegistry.resolve_model_cls`。 实现中包含条件分支。 本段逻辑主要由 `component_model_path`、`server_args` 和 `component_name` 等参数驱动。

## Key Concepts / 关键概念
- `BridgeLoader`: Loader for MOVA dual tower bridge with FSDP support. / 核心类，用于封装 bridge loader 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `copy`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`, `sglang.multimodal_gen.runtime.loader.fsdp_load`, `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.models.registry`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`, `sglang.multimodal_gen.configs.models.bridges.mova_dual_tower`

- **Total lines / 总行数**: 105
