# text_encoder_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/loader/component_loaders/text_encoder_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the component loading layer. It centers on `TextEncoderLoader`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于组件加载层。它围绕 `TextEncoderLoader` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-43: module setup and imports / 模块初始化与导入
```python
import dataclasses
import glob
import os
import re
from collections.abc import Generator, Iterable
from typing import cast

import torch
import torch.distributed as dist
from torch import nn
from torch.distributed import init_device_mesh
from transformers import AutoModel
from transformers.utils import SAFE_WEIGHTS_INDEX_NAME

from sglang.multimodal_gen.configs.models import EncoderConfig, ModelConfig
from sglang.multimodal_gen.configs.pipeline_configs.qwen_image import (
    QwenImageEditPipelineConfig,
)
from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.loader.component_loaders.component_loader import (
    ComponentLoader,
)
from sglang.multimodal_gen.runtime.loader.fsdp_load import shard_model
from sglang.multimodal_gen.runtime.loader.utils import (
    set_default_torch_dtype,
    skip_init_modules,
)
from sglang.multimodal_gen.runtime.loader.weight_utils import (
    filter_duplicate_safetensors_files,
    filter_files_not_needed_for_inference,
    pt_weights_iterator,
    safetensors_weights_iterator,
)
from sglang.multimodal_gen.runtime.models.registry import ModelRegistry
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.hf_diffusers_utils import (
    get_config,
    get_diffusers_component_config,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import PRECISION_TO_TYPE
from sglang.srt.environ import envs
```
**EN:** This block establishes the module context and imports `dataclasses`, `glob`, `os`, `re`, `collections.abc`, and `typing`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`glob`、`os`、`re`、`collections.abc` 和 `typing`。这些依赖为后续实现提供所需符号。

### Lines 45-45: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 48-50: `TextEncoderLoader` class overview / `TextEncoderLoader` 类概览
```python
class TextEncoderLoader(ComponentLoader):
    """Loader for text encoders."""
```
**EN:** This block defines class `TextEncoderLoader`. Loader for text encoders. It inherits from `ComponentLoader`.
**CN:** 该代码块定义了类 `TextEncoderLoader`。 它用于封装 text encoder loader 相关行为。 它继承自 `ComponentLoader`。

### Lines 51-52: supporting statements / 辅助语句
```python
    component_names = ["text_encoder"]
    expected_library = "transformers"
```
**EN:** This block gathers supporting statements inside `TextEncoderLoader`. It updates names such as `component_names`, and `expected_library`.
**CN:** 该代码块汇集了位于 `TextEncoderLoader` 内部的辅助语句。 它会更新 `component_names` 和 `expected_library` 等名称。

### Lines 54-68: `Source` class definition / `Source` 类定义
```python
    @dataclasses.dataclass
    class Source:
        """A source for weights."""

        model_or_path: str
        """The model ID or path."""

        prefix: str = ""
        """A prefix to prepend to all weights."""

        fall_back_to_pt: bool = True
        """Whether .pt weights can be used."""

        allow_patterns_overrides: list[str] | None = None
        """If defined, weights will load exclusively using these patterns."""
```
**EN:** This block defines method `Source` on `TextEncoderLoader`. A source for weights.
**CN:** 该代码块定义了 `TextEncoderLoader` 的方法 `Source`。 它用于封装 source 相关行为。

### Lines 70-82: `should_offload` implementation / `should_offload` 实现
```python
    def should_offload(self, server_args, model_config: ModelConfig | None = None):
        should_offload = server_args.text_encoder_cpu_offload
        if not should_offload:
            return False
        # _fsdp_shard_conditions is in arch_config, not directly on model_config
        arch_config = (
            getattr(model_config, "arch_config", model_config) if model_config else None
        )
        fsdp_shard_conditions = (
            getattr(arch_config, "_fsdp_shard_conditions", []) if arch_config else []
        )
        use_cpu_offload = should_offload and len(fsdp_shard_conditions) > 0
        return use_cpu_offload
```
**EN:** This block defines method `should_offload` on `TextEncoderLoader`. It determines whether to offload. Key calls include `getattr`, and `len`. The implementation branches on conditions. Parameters such as `server_args`, and `model_config` drive the behavior in this section.
**CN:** 该代码块定义了 `TextEncoderLoader` 的方法 `should_offload`。 它用于判断是否offload。 关键调用包括 `getattr` 和 `len`。 实现中包含条件分支。 本段逻辑主要由 `server_args` 和 `model_config` 等参数驱动。

### Lines 84-95: `customized_load_kwargs_for_component` implementation / `customized_load_kwargs_for_component` 实现
```python
    def customized_load_kwargs_for_component(
        self, server_args: ServerArgs, component_name: str
    ) -> dict[str, bool]:
        if ComponentLoader._is_component_set_as_layerwise_load(
            server_args, component_name
        ):
            logger.info(
                "Loading %s on CPU first because it is selected for layerwise offload",
                component_name,
            )
            return {"cpu_offload_flag": True}
        return {}
```
**EN:** This block defines method `customized_load_kwargs_for_component` on `TextEncoderLoader`. It handles customized load kwargs for component logic. Key calls include `ComponentLoader._is_component_set_as_layerwise_load`, and `logger.info`. The implementation branches on conditions. Parameters such as `server_args`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `TextEncoderLoader` 的方法 `customized_load_kwargs_for_component`。 它用于处理 customized load kwargs for component 相关逻辑。 关键调用包括 `ComponentLoader._is_component_set_as_layerwise_load` 和 `logger.info`。 实现中包含条件分支。 本段逻辑主要由 `server_args` 和 `component_name` 等参数驱动。

### Lines 97-117: `load_native` implementation / `load_native` 实现
```python
    def load_native(
        self,
        component_model_path: str,
        server_args: ServerArgs,
        transformers_or_diffusers: str,
    ):
        if transformers_or_diffusers != "transformers":
            return super().load_native(
                component_model_path, server_args, transformers_or_diffusers
            )

        encoder_idx = (
            1 if component_model_path.rstrip("/").endswith("text_encoder_2") else 0
        )
        encoder_dtype = server_args.pipeline_config.text_encoder_precisions[encoder_idx]
        return AutoModel.from_pretrained(
            component_model_path,
            trust_remote_code=server_args.trust_remote_code,
            revision=server_args.revision,
            torch_dtype=PRECISION_TO_TYPE[encoder_dtype],
        )
```
**EN:** This block defines method `load_native` on `TextEncoderLoader`. It loads native. Key calls include `AutoModel.from_pretrained`, `super.load_native`, `component_model_path.rstrip.endswith`, `super`, and `component_model_path.rstrip`. The implementation branches on conditions. Parameters such as `component_model_path`, `server_args`, and `transformers_or_diffusers` drive the behavior in this section.
**CN:** 该代码块定义了 `TextEncoderLoader` 的方法 `load_native`。 它用于加载native。 关键调用包括 `AutoModel.from_pretrained`、`super.load_native`、`component_model_path.rstrip.endswith`、`super` 和 `component_model_path.rstrip`。 实现中包含条件分支。 本段逻辑主要由 `component_model_path`、`server_args` 和 `transformers_or_diffusers` 等参数驱动。

### Lines 119-169: `_prepare_weights` implementation / `_prepare_weights` 实现
```python
    def _prepare_weights(
        self,
        model_name_or_path: str,
        fall_back_to_pt: bool,
        allow_patterns_overrides: list[str] | None,
    ) -> tuple[str, list[str], bool]:
        """Prepare weights for the model.

        If the model is not local, it will be downloaded."""
        # model_name_or_path = (self._maybe_download_from_modelscope(
        #     model_name_or_path, revision) or model_name_or_path)

        is_local = os.path.isdir(model_name_or_path)
        assert is_local, "Model path must be a local directory"

        use_safetensors = False
        index_file = SAFE_WEIGHTS_INDEX_NAME
        allow_patterns = ["*.safetensors", "*.bin"]

        if fall_back_to_pt:
            allow_patterns += ["*.pt"]

        if allow_patterns_overrides is not None:
            allow_patterns = allow_patterns_overrides

        hf_folder = model_name_or_path

        hf_weights_files: list[str] = []
        for pattern in allow_patterns:
            hf_weights_files += glob.glob(os.path.join(hf_folder, pattern))
            if len(hf_weights_files) > 0:
                if pattern == "*.safetensors":
                    use_safetensors = True
                break

        if use_safetensors:
            hf_weights_files = filter_duplicate_safetensors_files(
                hf_weights_files, hf_folder, index_file
            )
        else:
            hf_weights_files = filter_files_not_needed_for_inference(hf_weights_files)

        if len(hf_weights_files) == 0:
            raise RuntimeError(
                f"Cannot find any model weights with `{model_name_or_path}`"
            )

        if envs.SGLANG_SORT_WEIGHT_FILES.get():
            hf_weights_files.sort()

        return hf_folder, hf_weights_files, use_safetensors
```
**EN:** This block defines method `_prepare_weights` on `TextEncoderLoader`. Prepare weights for the model. If the model is not local, it will be downloaded. Key calls include `os.path.isdir`, `envs.SGLANG_SORT_WEIGHT_FILES.get`, `glob.glob`, `filter_duplicate_safetensors_files`, and `filter_files_not_needed_for_inference`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `model_name_or_path`, `fall_back_to_pt`, and `allow_patterns_overrides` drive the behavior in this section.
**CN:** 该代码块定义了 `TextEncoderLoader` 的方法 `_prepare_weights`。 它用于准备weights。 关键调用包括 `os.path.isdir`、`envs.SGLANG_SORT_WEIGHT_FILES.get`、`glob.glob`、`filter_duplicate_safetensors_files` 和 `filter_files_not_needed_for_inference`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `model_name_or_path`、`fall_back_to_pt` 和 `allow_patterns_overrides` 等参数驱动。

### Lines 171-191: `_get_weights_iterator` implementation / `_get_weights_iterator` 实现
```python
    def _get_weights_iterator(
        self,
        source: "Source",
        to_cpu: bool,
    ) -> Generator[tuple[str, torch.Tensor], None, None]:
        """get an iterator for the model weights based on the load format."""
        hf_folder, hf_weights_files, use_safetensors = self._prepare_weights(
            source.model_or_path,
            source.fall_back_to_pt,
            source.allow_patterns_overrides,
        )
        if use_safetensors:
            weights_iterator = safetensors_weights_iterator(
                hf_weights_files,
                to_cpu=to_cpu,
            )
        else:
            weights_iterator = pt_weights_iterator(hf_weights_files, to_cpu=to_cpu)

        # apply the prefix.
        return ((source.prefix + name, tensor) for (name, tensor) in weights_iterator)
```
**EN:** This block defines method `_get_weights_iterator` on `TextEncoderLoader`. get an iterator for the model weights based on the load format. Key calls include `self._prepare_weights`, `safetensors_weights_iterator`, and `pt_weights_iterator`. The implementation branches on conditions. Parameters such as `source`, and `to_cpu` drive the behavior in this section.
**CN:** 该代码块定义了 `TextEncoderLoader` 的方法 `_get_weights_iterator`。 它用于获取weights iterator。 关键调用包括 `self._prepare_weights`、`safetensors_weights_iterator` 和 `pt_weights_iterator`。 实现中包含条件分支。 本段逻辑主要由 `source` 和 `to_cpu` 等参数驱动。

### Lines 193-218: `_get_all_weights` implementation / `_get_all_weights` 实现
```python
    def _get_all_weights(
        self,
        model: nn.Module,
        model_path: str,
        to_cpu: bool,
    ) -> Generator[tuple[str, torch.Tensor], None, None]:
        primary_weights = TextEncoderLoader.Source(
            model_path,
            prefix="",
            fall_back_to_pt=getattr(model, "fall_back_to_pt_during_load", True),
            allow_patterns_overrides=getattr(model, "allow_patterns_overrides", None),
        )
        yield from self._get_weights_iterator(
            primary_weights,
            to_cpu,
        )

        secondary_weights = cast(
            Iterable[TextEncoderLoader.Source],
            getattr(model, "secondary_weights", ()),
        )
        for source in secondary_weights:
            yield from self._get_weights_iterator(
                source,
                to_cpu,
            )
```
**EN:** This block defines method `_get_all_weights` on `TextEncoderLoader`. It retrieves all weights. Key calls include `TextEncoderLoader.Source`, `cast`, `self._get_weights_iterator`, and `getattr`. The implementation iterates over collections or steps. Parameters such as `model`, `model_path`, and `to_cpu` drive the behavior in this section.
**CN:** 该代码块定义了 `TextEncoderLoader` 的方法 `_get_all_weights`。 它用于获取all weights。 关键调用包括 `TextEncoderLoader.Source`、`cast`、`self._get_weights_iterator` 和 `getattr`。 实现中会遍历集合或步骤。 本段逻辑主要由 `model`、`model_path` 和 `to_cpu` 等参数驱动。

### Lines 220-257: `load_customized` implementation / `load_customized` 实现
```python
    def load_customized(
        self,
        component_model_path: str,
        server_args: ServerArgs,
        component_name: str,
        cpu_offload_flag: bool | None = None,
    ):
        """Load the text encoders based on the model path, and inference args."""
        diffusers_pretrained_config = get_config(
            component_model_path, trust_remote_code=True
        )
        model_config = get_diffusers_component_config(
            component_path=component_model_path
        )

        # TODO(mick): had to throw an exception for different text-encoder arch
        encoder_index = self._extract_encoder_index(component_name)
        assert encoder_index < len(
            server_args.pipeline_config.text_encoder_configs
        ) and encoder_index < len(server_args.pipeline_config.text_encoder_precisions)

        encoder_config = server_args.pipeline_config.text_encoder_configs[encoder_index]
        encoder_config.update_model_arch(model_config)

        if encoder_index == 0:
            for key, value in diffusers_pretrained_config.__dict__.items():
                setattr(encoder_config.arch_config, key, value)
        encoder_dtype = server_args.pipeline_config.text_encoder_precisions[
            encoder_index
        ]
        # TODO(will): add support for other dtypes
        return self.load_model(
            component_model_path,
            encoder_config,
            server_args,
            encoder_dtype,
            cpu_offload_flag=cpu_offload_flag,
        )
```
**EN:** This block defines method `load_customized` on `TextEncoderLoader`. Load the text encoders based on the model path, and inference args. Key calls include `get_config`, `get_diffusers_component_config`, `self._extract_encoder_index`, `encoder_config.update_model_arch`, and `self.load_model`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `component_model_path`, `server_args`, `component_name`, and `cpu_offload_flag` drive the behavior in this section.
**CN:** 该代码块定义了 `TextEncoderLoader` 的方法 `load_customized`。 它用于加载customized。 关键调用包括 `get_config`、`get_diffusers_component_config`、`self._extract_encoder_index`、`encoder_config.update_model_arch` 和 `self.load_model`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `component_model_path`、`server_args`、`component_name` 和 `cpu_offload_flag` 等参数驱动。

### Lines 259-279: `_extract_encoder_index` implementation / `_extract_encoder_index` 实现
```python
    @staticmethod
    def _extract_encoder_index(component_name: str) -> int:
        """
        Map text encoder component names to zero-based indices.

        Examples:
        - text_encoder -> 0
        - text_encoder_2 -> 1
        - text_encoder_3 -> 2
        """
        match = re.search(r"_(\d+)$", component_name)
        if match is None:
            return 0

        suffix_num = int(match.group(1))
        if suffix_num <= 0:
            raise ValueError(
                f"Invalid text encoder component name '{component_name}': "
                "numeric suffix must be >= 1."
            )
        return suffix_num - 1
```
**EN:** This block defines method `_extract_encoder_index` on `TextEncoderLoader`. Map text encoder component names to zero-based indices. Examples: - text_encoder -> 0 - text_encoder_2 -> 1 - text_encoder_3 -> 2 Key calls include `re.search`, `int`, `match.group`, and `ValueError`. The implementation branches on conditions. Parameters such as `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `TextEncoderLoader` 的方法 `_extract_encoder_index`。 它用于处理 extract encoder index 相关逻辑。 关键调用包括 `re.search`、`int`、`match.group` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `component_name` 等参数驱动。

### Lines 281-391: `load_model` implementation / `load_model` 实现
```python
    def load_model(
        self,
        model_path: str,
        model_config: EncoderConfig,
        server_args: ServerArgs,
        dtype: str = "fp16",
        cpu_offload_flag: bool | None = None,
    ):
        # Determine CPU offload behavior and target device

        local_torch_device = get_local_torch_device()

        if not current_platform.is_cpu():
            fsdp_cpu_offload = self.should_offload(server_args, model_config)
            should_offload = (
                cpu_offload_flag if cpu_offload_flag is not None else fsdp_cpu_offload
            )
        else:
            fsdp_cpu_offload = False
            should_offload = False

        if should_offload and not current_platform.is_mps():
            model_device = torch.device("cpu")
        else:
            model_device = local_torch_device

        with set_default_torch_dtype(PRECISION_TO_TYPE[dtype]):
            with model_device, skip_init_modules():
                architectures = getattr(model_config, "architectures", [])
                model_cls, _ = ModelRegistry.resolve_model_cls(architectures)
                enable_image_understanding = (
                    True
                    if isinstance(
                        server_args.pipeline_config, QwenImageEditPipelineConfig
                    )
                    else False
                )
                model_config.enable_image_understanding = enable_image_understanding
                model = model_cls(model_config)

            weights_to_load = {name for name, _ in model.named_parameters()}
            loaded_weights = model.load_weights(
                self._get_all_weights(
                    model,
                    model_path,
                    to_cpu=should_offload,
                )
            )

            if should_offload:
                # Disable FSDP for MPS as it's not compatible
                if current_platform.is_mps():
                    logger.info(
                        "Disabling FSDP sharding for MPS platform as it's not compatible"
                    )
                    model = model.to(local_torch_device)
                elif fsdp_cpu_offload:
                    mesh = init_device_mesh(
                        current_platform.device_type,
                        mesh_shape=(1, dist.get_world_size()),
                        mesh_dim_names=("offload", "replicate"),
                    )
                    shard_model(
                        model,
                        cpu_offload=True,
                        reshard_after_forward=True,
                        mesh=mesh["offload"],
                        fsdp_shard_conditions=model_config.arch_config._fsdp_shard_conditions
                        or getattr(model, "_fsdp_shard_conditions", None),
                        pin_cpu_memory=server_args.pin_cpu_memory,
                    )
                else:
                    model = model.to("cpu")
            else:
                model = model.to(local_torch_device)
            # We only enable strict check for non-quantized models
            # that have loaded weights tracking currently.
            # if loaded_weights is not None:
            weights_not_loaded = weights_to_load - loaded_weights
            if weights_not_loaded:
                # NOTE:
                # If we silently continue with uninitialized weights, the text encoder can
                # produce NaNs/garbage embeddings that later fail stage verification in a
                # hard-to-debug way (e.g., `prompt_embeds` fails the NaN check).
                #
                # We allow a small set of known-optional parameters to be missing, but
                # default to strict behavior for the rest.
                allowed_missing_patterns = (
                    getattr(model, "_allowed_missing_weights_patterns", []) or []
                )
                unexpected_missing = {
                    n
                    for n in weights_not_loaded
                    if not any(pat in n for pat in allowed_missing_patterns)
                }
                if unexpected_missing:
                    raise ValueError(
                        "Following text encoder weights were not initialized from checkpoint: "
                        f"{sorted(unexpected_missing)}. "
                        "This usually indicates a checkpoint/model-arch mismatch or a broken "
                        "weight-name mapping. If these are truly optional, set "
                        "`model._allowed_missing_weights_patterns` to whitelist patterns."
                    )
                logger.warning(
                    "Following (allowed) text encoder weights were not initialized from "
                    "checkpoint: %s (allowed patterns: %s)",
                    sorted(weights_not_loaded),
                    allowed_missing_patterns,
                )

        return model
```
**EN:** This block defines method `load_model` on `TextEncoderLoader`. It loads model. Key calls include `get_local_torch_device`, `current_platform.is_cpu`, `self.should_offload`, `torch.device`, and `set_default_torch_dtype`. The implementation branches on conditions, uses context-managed resources. Parameters such as `model_path`, `model_config`, `server_args`, `dtype`, and `cpu_offload_flag` drive the behavior in this section.
**CN:** 该代码块定义了 `TextEncoderLoader` 的方法 `load_model`。 它用于加载model。 关键调用包括 `get_local_torch_device`、`current_platform.is_cpu`、`self.should_offload`、`torch.device` 和 `set_default_torch_dtype`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `model_path`、`model_config`、`server_args`、`dtype` 和 `cpu_offload_flag` 等参数驱动。

## Key Concepts / 关键概念
- `TextEncoderLoader`: Loader for text encoders. / 核心类，用于封装 text encoder loader 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `glob`, `os`, `re`, `collections.abc`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`, `transformers`, `transformers.utils`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.pipeline_configs.qwen_image`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`, `sglang.multimodal_gen.runtime.loader.fsdp_load`, `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.loader.weight_utils`, `sglang.multimodal_gen.runtime.models.registry`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 391
