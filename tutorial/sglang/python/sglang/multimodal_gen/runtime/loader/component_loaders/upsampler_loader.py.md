# upsampler_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/loader/component_loaders/upsampler_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the component loading layer. It centers on `UpsamplerLoader`, `_parse_hf_url`, and `_download_hf_file`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于组件加载层。它围绕 `UpsamplerLoader`、`_parse_hf_url` 和 `_download_hf_file` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module setup and imports / 模块初始化与导入
```python
import glob
import json
import os
import re

import safetensors
import torch
from safetensors.torch import load_file as safetensors_load_file

from sglang.multimodal_gen.runtime.loader.component_loaders.component_loader import (
    ComponentLoader,
)
from sglang.multimodal_gen.runtime.models.upsampler.latent_upsampler import (
    LatentUpsampler,
)
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.hf_diffusers_utils import maybe_download_model
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `glob`, `json`, `os`, `re`, `safetensors`, and `torch`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `glob`、`json`、`os`、`re`、`safetensors` 和 `torch`。这些依赖为后续实现提供所需符号。

### Lines 20-38: supporting statements / 辅助语句
```python
logger = init_logger(__name__)

UPSAMPLER_CONSTRUCTOR_KEYS = {
    "in_channels",
    "mid_channels",
    "num_blocks_per_stage",
    "dims",
    "spatial_upsample",
    "temporal_upsample",
    "spatial_scale",
    "rational_resampler",
}

_HF_BLOB_URL_RE = re.compile(
    r"https?://huggingface\.co/([^/]+/[^/]+)/blob/([^/]+)/(.*)"
)
_HF_RESOLVE_URL_RE = re.compile(
    r"https?://huggingface\.co/([^/]+/[^/]+)/resolve/([^/]+)/(.*)"
)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, `UPSAMPLER_CONSTRUCTOR_KEYS`, `_HF_BLOB_URL_RE`, and `_HF_RESOLVE_URL_RE`. The code collaborates with `init_logger`, and `re.compile`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger`、`UPSAMPLER_CONSTRUCTOR_KEYS`、`_HF_BLOB_URL_RE` 和 `_HF_RESOLVE_URL_RE` 等名称。 代码会与 `init_logger` 和 `re.compile` 协同工作。

### Lines 41-45: `_parse_hf_url` implementation / `_parse_hf_url` 实现
```python
def _parse_hf_url(path: str):
    m = _HF_BLOB_URL_RE.match(path) or _HF_RESOLVE_URL_RE.match(path)
    if m:
        return m.group(1), m.group(2), m.group(3)
    return None
```
**EN:** This block defines function `_parse_hf_url`. It parses hf url. Key calls include `_HF_BLOB_URL_RE.match`, `_HF_RESOLVE_URL_RE.match`, and `m.group`. The implementation branches on conditions. Parameters such as `path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_parse_hf_url`。 它用于解析hf url。 关键调用包括 `_HF_BLOB_URL_RE.match`、`_HF_RESOLVE_URL_RE.match` 和 `m.group`。 实现中包含条件分支。 本段逻辑主要由 `path` 等参数驱动。

### Lines 48-52: `_download_hf_file` implementation / `_download_hf_file` 实现
```python
def _download_hf_file(repo_id: str, filename: str, revision: str = "main") -> str:
    from huggingface_hub import hf_hub_download

    logger.info("Downloading %s from %s (revision=%s)", filename, repo_id, revision)
    return hf_hub_download(repo_id=repo_id, filename=filename, revision=revision)
```
**EN:** This block defines function `_download_hf_file`. It handles download hf file logic. Key calls include `logger.info`, and `hf_hub_download`. Parameters such as `repo_id`, `filename`, and `revision` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_download_hf_file`。 它用于处理 download hf file 相关逻辑。 关键调用包括 `logger.info` 和 `hf_hub_download`。 本段逻辑主要由 `repo_id`、`filename` 和 `revision` 等参数驱动。

### Lines 55-92: `_find_safetensors_file` implementation / `_find_safetensors_file` 实现
```python
def _find_safetensors_file(path: str) -> str:
    """Resolve path to a single safetensors file (local path, directory, HF URL, or HF repo id)."""
    if os.path.isfile(path) and path.endswith(".safetensors"):
        return path

    if os.path.isdir(path):
        files = sorted(glob.glob(os.path.join(path, "*.safetensors")))
        if len(files) == 1:
            return files[0]
        elif len(files) > 1:
            raise ValueError(
                f"Found {len(files)} safetensors files in {path}, expected 1"
            )

    hf = _parse_hf_url(path)
    if hf:
        repo_id, revision, filename = hf
        return _download_hf_file(repo_id, filename, revision)

    try:
        maybe_downloaded = maybe_download_model(path)
        if os.path.isdir(maybe_downloaded):
            files = sorted(glob.glob(os.path.join(maybe_downloaded, "*.safetensors")))
            if len(files) == 1:
                return files[0]
            elif len(files) > 1:
                raise ValueError(
                    f"Found {len(files)} safetensors files in {maybe_downloaded}, expected 1"
                )
    except Exception:
        pass

    raise FileNotFoundError(
        f"No safetensors file found at {path}. "
        "Provide a local .safetensors file, a directory containing one, "
        "a HuggingFace URL (https://huggingface.co/<repo>/blob/main/<path>), "
        "or a HuggingFace repo id."
    )
```
**EN:** This block defines function `_find_safetensors_file`. Resolve path to a single safetensors file (local path, directory, HF URL, or HF repo id). Key calls include `os.path.isdir`, `_parse_hf_url`, `FileNotFoundError`, `os.path.isfile`, and `path.endswith`. The implementation branches on conditions, handles exceptional paths. Parameters such as `path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_find_safetensors_file`。 它用于处理 find safetensors file 相关逻辑。 关键调用包括 `os.path.isdir`、`_parse_hf_url`、`FileNotFoundError`、`os.path.isfile` 和 `path.endswith`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `path` 等参数驱动。

### Lines 95-104: `_normalize_config` implementation / `_normalize_config` 实现
```python
def _normalize_config(raw: dict) -> dict:
    """Map diffusers / original-repo config fields to LatentUpsampler kwargs."""
    config = {k: v for k, v in raw.items() if k in UPSAMPLER_CONSTRUCTOR_KEYS}

    # diffusers uses rational_spatial_scale instead of rational_resampler + spatial_scale
    if "rational_spatial_scale" in raw and "rational_resampler" not in config:
        config["rational_resampler"] = True
        config.setdefault("spatial_scale", raw["rational_spatial_scale"])

    return config
```
**EN:** This block defines function `_normalize_config`. Map diffusers / original-repo config fields to LatentUpsampler kwargs. Key calls include `config.setdefault`, and `raw.items`. The implementation branches on conditions. Parameters such as `raw` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_normalize_config`。 它用于处理 normalize config 相关逻辑。 关键调用包括 `config.setdefault` 和 `raw.items`。 实现中包含条件分支。 本段逻辑主要由 `raw` 等参数驱动。

### Lines 107-152: `_infer_config_from_state_dict` implementation / `_infer_config_from_state_dict` 实现
```python
def _infer_config_from_state_dict(state_dict: dict[str, torch.Tensor]) -> dict:
    """Infer LatentUpsampler kwargs from weight shapes and key names.

    Works even when no config.json or safetensors metadata is available.
    """
    config: dict = {}

    w = state_dict.get("initial_conv.weight")
    if w is not None:
        config["mid_channels"] = w.shape[0]
        config["in_channels"] = w.shape[1]
        config["dims"] = 3 if w.ndim == 5 else 2

    num_blocks = sum(
        1
        for k in state_dict
        if k.startswith("res_blocks.") and k.endswith(".conv1.weight")
    )
    if num_blocks > 0:
        config["num_blocks_per_stage"] = num_blocks

    # Detect upsampler type from key patterns
    has_rational = any(k.startswith("upsampler.blur_down.") for k in state_dict)
    if has_rational:
        config["rational_resampler"] = True
        config["spatial_upsample"] = True
        config["temporal_upsample"] = False
        config["spatial_scale"] = 2.0
    else:
        up_w = state_dict.get("upsampler.0.weight")
        if up_w is not None and up_w.ndim == 5:
            ratio = up_w.shape[0] // up_w.shape[1]
            if ratio == 8:
                config["spatial_upsample"] = True
                config["temporal_upsample"] = True
            elif ratio == 2:
                config["spatial_upsample"] = False
                config["temporal_upsample"] = True
            else:
                config["spatial_upsample"] = True
                config["temporal_upsample"] = False
        else:
            config["spatial_upsample"] = True
            config["temporal_upsample"] = False

    return config
```
**EN:** This block defines function `_infer_config_from_state_dict`. Infer LatentUpsampler kwargs from weight shapes and key names. Works even when no config.json or safetensors metadata is available. Key calls include `state_dict.get`, `sum`, `any`, `k.startswith`, and `k.endswith`. The implementation branches on conditions. Parameters such as `state_dict` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_infer_config_from_state_dict`。 它用于推断config from state dict。 关键调用包括 `state_dict.get`、`sum`、`any`、`k.startswith` 和 `k.endswith`。 实现中包含条件分支。 本段逻辑主要由 `state_dict` 等参数驱动。

### Lines 155-191: `_load_config` implementation / `_load_config` 实现
```python
def _load_config(
    safetensors_path: str,
    original_path: str,
    state_dict: dict[str, torch.Tensor],
) -> dict:
    """Load upsampler config with fallback chain:
    1. safetensors metadata ("config" key) - original LTX-2 repo format
    2. sibling config.json - diffusers format
    3. config.json from HF (if original_path was a URL)
    4. infer from state dict shapes (always works)
    """
    with safetensors.safe_open(safetensors_path, framework="pt") as f:
        meta = f.metadata()
        if meta and "config" in meta:
            logger.info("Using config from safetensors metadata")
            return _normalize_config(json.loads(meta["config"]))

    config_json_path = os.path.join(os.path.dirname(safetensors_path), "config.json")
    if os.path.isfile(config_json_path):
        with open(config_json_path) as fp:
            logger.info("Using config from sibling config.json")
            return _normalize_config(json.load(fp))

    hf = _parse_hf_url(original_path)
    if hf:
        repo_id, revision, filename = hf
        config_filename = os.path.dirname(filename) + "/config.json"
        try:
            local = _download_hf_file(repo_id, config_filename, revision)
            with open(local) as fp:
                logger.info("Using config from HF config.json")
                return _normalize_config(json.load(fp))
        except Exception:
            pass

    logger.info("No explicit config found, inferring from state dict")
    return _infer_config_from_state_dict(state_dict)
```
**EN:** This block defines function `_load_config`. Load upsampler config with fallback chain: 1. safetensors metadata ("config" key) - original LTX-2 repo format 2. Key calls include `os.path.join`, `os.path.isfile`, `_parse_hf_url`, `logger.info`, and `_infer_config_from_state_dict`. The implementation branches on conditions, handles exceptional paths, uses context-managed resources. Parameters such as `safetensors_path`, `original_path`, and `state_dict` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_load_config`。 它用于加载config。 关键调用包括 `os.path.join`、`os.path.isfile`、`_parse_hf_url`、`logger.info` 和 `_infer_config_from_state_dict`。 实现中包含条件分支，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `safetensors_path`、`original_path` 和 `state_dict` 等参数驱动。

### Lines 194-194: `UpsamplerLoader` class overview / `UpsamplerLoader` 类概览
```python
class UpsamplerLoader(ComponentLoader):
```
**EN:** This block defines class `UpsamplerLoader`. It encapsulates upsampler loader behavior. It inherits from `ComponentLoader`.
**CN:** 该代码块定义了类 `UpsamplerLoader`。 它用于封装 upsampler loader 相关行为。 它继承自 `ComponentLoader`。

### Lines 195-196: supporting statements / 辅助语句
```python
    component_names = ["spatial_upsampler"]
    expected_library = "diffusers"
```
**EN:** This block gathers supporting statements inside `UpsamplerLoader`. It updates names such as `component_names`, and `expected_library`.
**CN:** 该代码块汇集了位于 `UpsamplerLoader` 内部的辅助语句。 它会更新 `component_names` 和 `expected_library` 等名称。

### Lines 198-199: `should_offload` implementation / `should_offload` 实现
```python
    def should_offload(self, server_args: ServerArgs, model_config=None):
        return server_args.vae_cpu_offload
```
**EN:** This block defines method `should_offload` on `UpsamplerLoader`. It determines whether to offload. Parameters such as `server_args`, and `model_config` drive the behavior in this section.
**CN:** 该代码块定义了 `UpsamplerLoader` 的方法 `should_offload`。 它用于判断是否offload。 本段逻辑主要由 `server_args` 和 `model_config` 等参数驱动。

### Lines 201-223: `load_customized` implementation / `load_customized` 实现
```python
    def load_customized(
        self,
        component_model_path: str,
        server_args: ServerArgs,
        component_name: str,
    ):
        safetensors_path = _find_safetensors_file(component_model_path)
        state_dict = safetensors_load_file(safetensors_path)
        config = _load_config(safetensors_path, component_model_path, state_dict)

        logger.info("Loading LatentUpsampler with config: %s", config)

        should_offload = self.should_offload(server_args)
        target_device = self.target_device(should_offload)

        with torch.device("meta"):
            model = LatentUpsampler(**config)

        model.load_state_dict(state_dict, assign=True)
        model = model.to(device=target_device, dtype=torch.bfloat16).eval()

        logger.info("Loaded LatentUpsampler to %s", target_device)
        return model
```
**EN:** This block defines method `load_customized` on `UpsamplerLoader`. It loads customized. Key calls include `_find_safetensors_file`, `safetensors_load_file`, `_load_config`, `logger.info`, and `self.should_offload`. The implementation uses context-managed resources. Parameters such as `component_model_path`, `server_args`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `UpsamplerLoader` 的方法 `load_customized`。 它用于加载customized。 关键调用包括 `_find_safetensors_file`、`safetensors_load_file`、`_load_config`、`logger.info` 和 `self.should_offload`。 实现中使用上下文管理资源。 本段逻辑主要由 `component_model_path`、`server_args` 和 `component_name` 等参数驱动。

## Key Concepts / 关键概念
- `_parse_hf_url`: Top-level function that parses hf url. / 顶层函数，用于解析hf url。
- `_download_hf_file`: Top-level function that handles download hf file logic. / 顶层函数，用于处理 download hf file 相关逻辑。
- `_find_safetensors_file`: Resolve path to a single safetensors file (local path, directory, HF URL, or HF repo id). / 顶层函数，用于处理 find safetensors file 相关逻辑。
- `_normalize_config`: Map diffusers / original-repo config fields to LatentUpsampler kwargs. / 顶层函数，用于处理 normalize config 相关逻辑。
- `_infer_config_from_state_dict`: Infer LatentUpsampler kwargs from weight shapes and key names. / 顶层函数，用于推断config from state dict。
- `_load_config`: Load upsampler config with fallback chain: 1. / 顶层函数，用于加载config。
- `UpsamplerLoader`: Primary class that encapsulates upsampler loader behavior. / 核心类，用于封装 upsampler loader 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `glob`, `json`, `os`, `re`
- **Third-party / 第三方依赖**: `safetensors`, `torch`, `safetensors.torch`, `huggingface_hub`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`, `sglang.multimodal_gen.runtime.models.upsampler.latent_upsampler`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 223
