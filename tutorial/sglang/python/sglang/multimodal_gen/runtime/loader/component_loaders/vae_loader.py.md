# vae_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/loader/component_loaders/vae_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the component loading layer. It centers on `VAELoader`, `_backfill_ltx2_audio_vae_latent_stats`, and `_convert_conv3d_weights_to_channels_last_3d`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于组件加载层。它围绕 `VAELoader`、`_backfill_ltx2_audio_vae_latent_stats` 和 `_convert_conv3d_weights_to_channels_last_3d` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: module setup and imports / 模块初始化与导入
```python
import importlib.util
import os

import torch
import torch.nn as nn
from safetensors.torch import load_file as safetensors_load_file

from sglang.multimodal_gen import envs
from sglang.multimodal_gen.configs.models import ModelConfig
from sglang.multimodal_gen.runtime.loader.component_loaders.component_loader import (
    ComponentLoader,
)
from sglang.multimodal_gen.runtime.loader.utils import (
    _list_safetensors_files,
    set_default_torch_dtype,
    skip_init_modules,
)
from sglang.multimodal_gen.runtime.models.registry import ModelRegistry
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.hf_diffusers_utils import (
    get_diffusers_component_config,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import PRECISION_TO_TYPE
```
**EN:** This block establishes the module context and imports `importlib.util`, `os`, `torch`, `torch.nn`, `safetensors.torch`, and `sglang.multimodal_gen`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `importlib.util`、`os`、`torch`、`torch.nn`、`safetensors.torch` 和 `sglang.multimodal_gen`。这些依赖为后续实现提供所需符号。

### Lines 27-27: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 30-40: `_backfill_ltx2_audio_vae_latent_stats` implementation / `_backfill_ltx2_audio_vae_latent_stats` 实现
```python
def _backfill_ltx2_audio_vae_latent_stats(
    loaded: dict[str, torch.Tensor], component_name: str
) -> None:
    if component_name != "audio_vae":
        return
    mean_key = "per_channel_statistics.mean-of-means"
    std_key = "per_channel_statistics.std-of-means"
    if "latents_mean" not in loaded and mean_key in loaded:
        loaded["latents_mean"] = loaded[mean_key]
    if "latents_std" not in loaded and std_key in loaded:
        loaded["latents_std"] = loaded[std_key]
```
**EN:** This block defines function `_backfill_ltx2_audio_vae_latent_stats`. It handles backfill ltx2 audio vae latent stats logic. The implementation branches on conditions. Parameters such as `loaded`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_backfill_ltx2_audio_vae_latent_stats`。 它用于处理 backfill ltx2 audio vae latent stats 相关逻辑。 实现中包含条件分支。 本段逻辑主要由 `loaded` 和 `component_name` 等参数驱动。

### Lines 43-59: `_convert_conv3d_weights_to_channels_last_3d` implementation / `_convert_conv3d_weights_to_channels_last_3d` 实现
```python
def _convert_conv3d_weights_to_channels_last_3d(module: nn.Module) -> int:
    """
    Convert Conv3d weights to channels_last_3d (NDHWC) memory format.
    Returns the number of Conv3d modules converted.
    """
    if not hasattr(torch, "channels_last_3d"):
        return 0
    num_converted = 0
    for m in module.modules():
        if isinstance(m, nn.Conv3d):
            try:
                m.weight.data = m.weight.data.to(memory_format=torch.channels_last_3d)
                num_converted += 1
            except Exception:
                # Best-effort; skip unsupported cases.
                continue
    return num_converted
```
**EN:** This block defines function `_convert_conv3d_weights_to_channels_last_3d`. Convert Conv3d weights to channels_last_3d (NDHWC) memory format. Returns the number of Conv3d modules converted. Key calls include `module.modules`, `hasattr`, `isinstance`, and `m.weight.data.to`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `module` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_convert_conv3d_weights_to_channels_last_3d`。 它用于转换conv3d weights to channels last 3d。 关键调用包括 `module.modules`、`hasattr`、`isinstance` 和 `m.weight.data.to`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `module` 等参数驱动。

### Lines 62-64: `VAELoader` class overview / `VAELoader` 类概览
```python
class VAELoader(ComponentLoader):
    """Shared loader for (video/audio) VAE modules."""
```
**EN:** This block defines class `VAELoader`. Shared loader for (video/audio) VAE modules. It inherits from `ComponentLoader`.
**CN:** 该代码块定义了类 `VAELoader`。 它用于封装 vaeloader 相关行为。 它继承自 `ComponentLoader`。

### Lines 65-66: supporting statements / 辅助语句
```python
    component_names = ["vae", "audio_vae", "video_vae"]
    expected_library = "diffusers"
```
**EN:** This block gathers supporting statements inside `VAELoader`. It updates names such as `component_names`, and `expected_library`.
**CN:** 该代码块汇集了位于 `VAELoader` 内部的辅助语句。 它会更新 `component_names` 和 `expected_library` 等名称。

### Lines 68-71: `should_offload` implementation / `should_offload` 实现
```python
    def should_offload(
        self, server_args: ServerArgs, model_config: ModelConfig | None = None
    ):
        return server_args.vae_cpu_offload
```
**EN:** This block defines method `should_offload` on `VAELoader`. It determines whether to offload. Parameters such as `server_args`, and `model_config` drive the behavior in this section.
**CN:** 该代码块定义了 `VAELoader` 的方法 `should_offload`。 它用于判断是否offload。 本段逻辑主要由 `server_args` 和 `model_config` 等参数驱动。

### Lines 73-180: `load_customized` implementation / `load_customized` 实现
```python
    def load_customized(
        self, component_model_path: str, server_args: ServerArgs, component_name: str
    ):
        """Load the VAE based on the model path, and inference args."""
        config = get_diffusers_component_config(component_path=component_model_path)
        class_name = config.pop("_class_name", None)
        assert (
            class_name is not None
        ), "Model config does not contain a _class_name attribute. Only diffusers format is supported."

        server_args.model_paths[component_name] = component_model_path

        if component_name in ("vae", "video_vae"):
            pipeline_vae_config_attr = "vae_config"
            pipeline_vae_precision = "vae_precision"
        elif component_name in ("audio_vae",):
            pipeline_vae_config_attr = "audio_vae_config"
            pipeline_vae_precision = "audio_vae_precision"
        else:
            raise ValueError(
                f"Unsupported module name for VAE loader: {component_name}"
            )
        vae_config = getattr(server_args.pipeline_config, pipeline_vae_config_attr)
        vae_precision = getattr(server_args.pipeline_config, pipeline_vae_precision)
        vae_config.update_model_arch(config)
        if hasattr(vae_config, "post_init"):
            # NOTE: some post init logics are only available after updated with config
            vae_config.post_init()

        should_offload = self.should_offload(server_args)
        target_device = self.target_device(should_offload)

        # Check for auto_map first (custom VAE classes)
        auto_map = config.get("auto_map", {})
        auto_model_map = auto_map.get("AutoModel")
        if auto_model_map:
            module_path, cls_name = auto_model_map.rsplit(".", 1)
            custom_module_file = os.path.join(component_model_path, f"{module_path}.py")
            spec = importlib.util.spec_from_file_location("_custom", custom_module_file)
            custom_module = importlib.util.module_from_spec(spec)
            spec.loader.exec_module(custom_module)
            vae_cls = getattr(custom_module, cls_name)
            vae_dtype = PRECISION_TO_TYPE[vae_precision]
            with set_default_torch_dtype(vae_dtype):
                vae = vae_cls.from_pretrained(
                    component_model_path,
                    revision=server_args.revision,
                    trust_remote_code=server_args.trust_remote_code,
                )
            vae = vae.to(device=target_device, dtype=vae_dtype)
            if (
                component_name in ("vae", "video_vae")
                and torch.cuda.is_available()
                and getattr(envs, "SGLANG_DIFFUSION_VAE_CHANNELS_LAST_3D", False)
            ):
                n = _convert_conv3d_weights_to_channels_last_3d(vae)
                if n > 0:
                    logger.info(
                        "VAE: converted %d Conv3d weights to channels_last_3d", n
                    )
            vae = current_platform.optimize_vae(vae)
            return vae

        # Load from ModelRegistry (standard VAE classes)
        with (
            set_default_torch_dtype(PRECISION_TO_TYPE[vae_precision]),
            skip_init_modules(),
        ):
            vae_cls, _ = ModelRegistry.resolve_model_cls(class_name)
            vae = vae_cls(vae_config).to(target_device)

        safetensors_list = _list_safetensors_files(component_model_path)
        safetensors_list = server_args.pipeline_config.select_vae_weight_files(
            safetensors_list=safetensors_list,
            component_model_path=component_model_path,
            component_name=component_name,
            vae_precision=vae_precision,
        )

        assert (
            len(safetensors_list) >= 1
        ), f"Found no safetensors files in {component_model_path}"
        loaded = {}
        for sf_path in safetensors_list:
            loaded.update(safetensors_load_file(sf_path))
        _backfill_ltx2_audio_vae_latent_stats(loaded, component_name)
        vae.load_state_dict(loaded, strict=False)

        state_keys = set(vae.state_dict().keys())
        loaded_keys = set(loaded.keys())
        missing_keys = sorted(state_keys - loaded_keys)
        unexpected_keys = sorted(loaded_keys - state_keys)
        if missing_keys:
            logger.warning("VAE missing keys: %s", missing_keys)
        if unexpected_keys:
            logger.warning("VAE unexpected keys: %s", unexpected_keys)

        if (
            component_name in ("vae", "video_vae")
            and torch.cuda.is_available()
            and getattr(envs, "SGLANG_DIFFUSION_VAE_CHANNELS_LAST_3D", False)
        ):
            n = _convert_conv3d_weights_to_channels_last_3d(vae)
            if n > 0:
                logger.info("VAE: converted %d Conv3d weights to channels_last_3d", n)

        vae = current_platform.optimize_vae(vae)
        return vae
```
**EN:** This block defines method `load_customized` on `VAELoader`. Load the VAE based on the model path, and inference args. Key calls include `get_diffusers_component_config`, `config.pop`, `getattr`, `vae_config.update_model_arch`, and `hasattr`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `component_model_path`, `server_args`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `VAELoader` 的方法 `load_customized`。 它用于加载customized。 关键调用包括 `get_diffusers_component_config`、`config.pop`、`getattr`、`vae_config.update_model_arch` 和 `hasattr`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `component_model_path`、`server_args` 和 `component_name` 等参数驱动。

## Key Concepts / 关键概念
- `_backfill_ltx2_audio_vae_latent_stats`: Top-level function that handles backfill ltx2 audio vae latent stats logic. / 顶层函数，用于处理 backfill ltx2 audio vae latent stats 相关逻辑。
- `_convert_conv3d_weights_to_channels_last_3d`: Convert Conv3d weights to channels_last_3d (NDHWC) memory format. / 顶层函数，用于转换conv3d weights to channels last 3d。
- `VAELoader`: Shared loader for (video/audio) VAE modules. / 核心类，用于封装 vaeloader 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `importlib.util`, `os`
- **Third-party / 第三方依赖**: `torch`, `torch.nn`, `safetensors.torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen`, `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`, `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.models.registry`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 180
