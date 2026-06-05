# transformer_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/loader/component_loaders/transformer_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the component loading layer. It centers on `TransformerLoader`, and `_server_args_for_transformer_component`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于组件加载层。它围绕 `TransformerLoader` 和 `_server_args_for_transformer_component` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: module setup and imports / 模块初始化与导入
```python
import copy
import logging
from typing import Any

import torch

from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.loader.component_loaders.component_loader import (
    ComponentLoader,
)
from sglang.multimodal_gen.runtime.loader.fsdp_load import maybe_load_fsdp_model
from sglang.multimodal_gen.runtime.loader.transformer_load_utils import (
    resolve_transformer_quant_load_spec,
    resolve_transformer_safetensors_to_load,
)
from sglang.multimodal_gen.runtime.loader.utils import _normalize_component_type
from sglang.multimodal_gen.runtime.models.registry import ModelRegistry
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.hf_diffusers_utils import (
    get_diffusers_component_config,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import get_log_level, init_logger
from sglang.srt.utils import is_npu
```
**EN:** This block establishes the module context and imports `copy`, `logging`, `typing`, `torch`, `sglang.multimodal_gen.runtime.distributed`, and `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `copy`、`logging`、`typing`、`torch`、`sglang.multimodal_gen.runtime.distributed` 和 `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`。这些依赖为后续实现提供所需符号。

### Lines 25-27: supporting statements / 辅助语句
```python
_is_npu = is_npu()

logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_is_npu`, and `logger`. The code collaborates with `is_npu`, and `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_is_npu` 和 `logger` 等名称。 代码会与 `is_npu` 和 `init_logger` 协同工作。

### Lines 30-51: `_server_args_for_transformer_component` implementation / `_server_args_for_transformer_component` 实现
```python
def _server_args_for_transformer_component(
    server_args: ServerArgs, component_name: str
) -> ServerArgs:
    """Mask global quantized override flags for secondary transformer components."""
    if component_name != "transformer_2":
        return server_args

    if (
        server_args.transformer_weights_path is None
        and server_args.nunchaku_config is None
    ):
        return server_args

    component_server_args = copy.copy(server_args)
    component_server_args.transformer_weights_path = None
    component_server_args.nunchaku_config = None
    logger.info(
        "Ignoring global transformer_weights_path for %s; keep it on the base "
        "checkpoint unless a per-component override path is provided.",
        component_name,
    )
    return component_server_args
```
**EN:** This block defines function `_server_args_for_transformer_component`. Mask global quantized override flags for secondary transformer components. Key calls include `copy.copy`, and `logger.info`. The implementation branches on conditions. Parameters such as `server_args`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_server_args_for_transformer_component`。 它用于处理 server args for transformer component 相关逻辑。 关键调用包括 `copy.copy` 和 `logger.info`。 实现中包含条件分支。 本段逻辑主要由 `server_args` 和 `component_name` 等参数驱动。

### Lines 54-56: `TransformerLoader` class overview / `TransformerLoader` 类概览
```python
class TransformerLoader(ComponentLoader):
    """Shared loader for (video/audio) DiT transformers."""
```
**EN:** This block defines class `TransformerLoader`. Shared loader for (video/audio) DiT transformers. It inherits from `ComponentLoader`.
**CN:** 该代码块定义了类 `TransformerLoader`。 它用于封装 transformer loader 相关行为。 它继承自 `ComponentLoader`。

### Lines 57-58: supporting statements / 辅助语句
```python
    component_names = ["transformer", "audio_dit", "video_dit"]
    expected_library = "diffusers"
```
**EN:** This block gathers supporting statements inside `TransformerLoader`. It updates names such as `component_names`, and `expected_library`.
**CN:** 该代码块汇集了位于 `TransformerLoader` 内部的辅助语句。 它会更新 `component_names` 和 `expected_library` 等名称。

### Lines 60-155: `load_customized` implementation / `load_customized` 实现
```python
    def load_customized(
        self, component_model_path: str, server_args: ServerArgs, component_name: str
    ):
        """Load the transformer based on the model path, and inference args."""
        component_server_args = _server_args_for_transformer_component(
            server_args, component_name
        )

        # 1. hf config
        config = get_diffusers_component_config(component_path=component_model_path)

        safetensors_list = resolve_transformer_safetensors_to_load(
            component_server_args, component_model_path
        )

        # 2. dit config
        # Config from Diffusers supersedes sgl_diffusion's model config
        component_name = _normalize_component_type(component_name)
        server_args.model_paths[component_name] = component_model_path
        if component_name in ("transformer", "video_dit"):
            pipeline_dit_config_attr = "dit_config"
        elif component_name in ("audio_dit",):
            pipeline_dit_config_attr = "audio_dit_config"
        else:
            raise ValueError(f"Invalid module name: {component_name}")
        dit_config = getattr(server_args.pipeline_config, pipeline_dit_config_attr)
        dit_config.update_model_arch(config)

        cls_name = config.pop("_class_name")
        model_cls, _ = ModelRegistry.resolve_model_cls(cls_name)

        quant_spec = resolve_transformer_quant_load_spec(
            hf_config=config,
            server_args=component_server_args,
            safetensors_list=safetensors_list,
            component_model_path=component_model_path,
            model_cls=model_cls,
            cls_name=cls_name,
        )

        logger.info(
            "Loading %s from %s safetensors file(s) %s, param_dtype: %s",
            cls_name,
            len(safetensors_list),
            f": {safetensors_list}" if get_log_level() == logging.DEBUG else "",
            quant_spec.param_dtype,
        )
        # prepare init_param
        init_params: dict[str, Any] = {
            "config": dit_config,
            "hf_config": config,
            "quant_config": quant_spec.runtime_quant_config,
        }
        if (
            init_params["quant_config"] is None
            and component_server_args.transformer_weights_path is not None
        ):
            logger.warning(
                f"transformer_weights_path provided, but quantization config not resolved, which is unexpected and likely to cause errors"
            )
        else:
            logger.debug("quantization config: %s", init_params["quant_config"])

        # Load the model using FSDP loader
        model = maybe_load_fsdp_model(
            model_cls=model_cls,
            init_params=init_params,
            weight_dir_list=safetensors_list,
            device=get_local_torch_device(),
            hsdp_replicate_dim=server_args.hsdp_replicate_dim,
            hsdp_shard_dim=server_args.hsdp_shard_dim,
            cpu_offload=component_server_args.dit_cpu_offload,
            pin_cpu_memory=component_server_args.pin_cpu_memory,
            fsdp_inference=component_server_args.use_fsdp_inference,
            param_dtype=quant_spec.param_dtype,
            reduce_dtype=torch.float32,
            output_dtype=None,
            strict=False,
        )

        # post-hooks (e.g., patch scales (nunchaku))
        for post_load_hook in quant_spec.post_load_hooks:
            post_load_hook(model)

        # considering the existent of mixed-precision models (e.g., nunchaku)
        if (
            next(model.parameters()).dtype != quant_spec.param_dtype
            and quant_spec.param_dtype
        ):
            logger.warning(
                "Model dtype does not match expected param dtype, %s vs %s",
                next(model.parameters()).dtype,
                quant_spec.param_dtype,
            )

        return model
```
**EN:** This block defines method `load_customized` on `TransformerLoader`. Load the transformer based on the model path, and inference args. Key calls include `_server_args_for_transformer_component`, `get_diffusers_component_config`, `resolve_transformer_safetensors_to_load`, `_normalize_component_type`, and `getattr`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `component_model_path`, `server_args`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `TransformerLoader` 的方法 `load_customized`。 它用于加载customized。 关键调用包括 `_server_args_for_transformer_component`、`get_diffusers_component_config`、`resolve_transformer_safetensors_to_load`、`_normalize_component_type` 和 `getattr`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `component_model_path`、`server_args` 和 `component_name` 等参数驱动。

## Key Concepts / 关键概念
- `_server_args_for_transformer_component`: Mask global quantized override flags for secondary transformer components. / 顶层函数，用于处理 server args for transformer component 相关逻辑。
- `TransformerLoader`: Shared loader for (video/audio) DiT transformers. / 核心类，用于封装 transformer loader 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `copy`, `logging`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`, `sglang.multimodal_gen.runtime.loader.fsdp_load`, `sglang.multimodal_gen.runtime.loader.transformer_load_utils`, `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.models.registry`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.srt.utils`

- **Total lines / 总行数**: 155
