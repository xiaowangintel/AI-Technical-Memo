# weight_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/model_loader/weight_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the model loading and weight management part of the SRT runtime and implements logic centered on `weight_utils`. The module docstring frames it as: "Utilities for downloading and initializing model weights." / 该模块属于 SRT 运行时的模型加载与权重管理部分，主要实现围绕 `weight_utils` 的逻辑。 它对外提供的主要入口包括 `probe_routed_expert_weight_dtype`, `_get_prefetch_block_size`, `get_lock`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60: Module imports, constants, and setup (part 1/2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/model_executor/model_loader/weight_utils.py

"""Utilities for downloading and initializing model weights."""

import collections
import concurrent.futures
import fnmatch
import glob
import hashlib
import itertools
import json
import logging
import os
import re
import struct
import tempfile
from collections import defaultdict
from pathlib import Path
from typing import (
    Any,
    Callable,
    Dict,
    Generator,
    Iterable,
    List,
    Optional,
    Tuple,
    Union,
)

import filelock
import huggingface_hub.constants
import numpy as np
import safetensors.torch
import torch
from huggingface_hub import HfFileSystem, hf_hub_download, snapshot_download
from pydantic import BaseModel, ConfigDict, ValidationInfo, model_validator
from tqdm.auto import tqdm

from sglang.srt.configs.load_config import LoadConfig
from sglang.srt.configs.model_config import ModelConfig
from sglang.srt.distributed import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    get_world_group,
)
from sglang.srt.layers.dp_attention import get_attention_tp_rank
from sglang.srt.layers.quantization import QuantizationConfig, get_quantization_config
from sglang.srt.layers.quantization.fp8 import Fp8Config
from sglang.srt.layers.quantization.modelopt_quant import (
    ModelOptFp4Config,
    ModelOptFp8Config,
)
from sglang.srt.model_loader.ci_weight_validation import (
    ci_download_with_validation_and_retry,
    ci_validate_and_cleanup_local_snapshot,
)
from sglang.srt.utils import (
```
**EN:** This range organizes module-level state and shared setup. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; manages model weights or checkpoints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；管理模型权重或检查点。

### Lines 61-87: Module imports, constants, and setup (part 2/2)
```python
    BAR_FORMAT,
    find_local_repo_dir,
    is_cpu,
    log_info_on_rank0,
    print_warning_once,
)
from sglang.srt.utils.common import is_cuda_alike
from sglang.utils import is_in_ci

try:
    from fastsafetensors import SafeTensorsFileLoader, SingleGroup
except ImportError as e:
    SafeTensorsFileLoader = SingleGroup = None

logger = logging.getLogger(__name__)

RUNAI_STREAMER_TENSOR_ATTR = "_sglang_runai_streamer_tensor"

# Matches routed-expert weight keys in both HF-style layouts
# (``...mlp.experts.<N>.{gate,up,down}_proj.weight``) and DeepSeek V4
# layouts (``...ffn.experts.<N>.w{1,2,3}.weight``). ``shared_experts`` is
# excluded because the index segment requires a digit after ``.experts.``.
_ROUTED_EXPERT_KEY_RE = re.compile(
    r"\.experts\.\d+\.(?:w[123]|down_proj|up_proj|gate_proj)\.weight$"
)


```
**EN:** This range organizes module-level state and shared setup. This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols; emits logs for diagnostics; prepares compilation-related behavior.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；处理与编译相关的行为。

### Lines 88-130: Function probe_routed_expert_weight_dtype
```python
def probe_routed_expert_weight_dtype(model_path: str) -> Optional[str]:
    """Return the safetensors dtype string (e.g. ``F8_E4M3``, ``U8``) of one
    routed-expert weight tensor, or ``None`` if the checkpoint is remote or has
    no matching key. Reads only the safetensors header of the relevant shard.
    """
    if not os.path.isdir(model_path):
        return None

    index_file = os.path.join(model_path, "model.safetensors.index.json")
    target_key = None
    target_shard_path = None

    if os.path.exists(index_file):
        with open(index_file) as f:
            index = json.load(f)
        weight_map = index.get("weight_map", {}) or {}
        for k, shard in weight_map.items():
            if _ROUTED_EXPERT_KEY_RE.search(k):
                target_key = k
                target_shard_path = os.path.join(model_path, shard)
                break
        if target_key is None:
            return None
    else:
        shards = sorted(Path(model_path).glob("*.safetensors"))
        if not shards:
            return None
        target_shard_path = str(shards[0])

    with open(target_shard_path, "rb") as f:
        (header_len,) = struct.unpack("<Q", f.read(8))
        header = json.loads(f.read(header_len))

    if target_key is not None:
        meta = header.get(target_key)
        return meta.get("dtype") if meta else None

    for k, meta in header.items():
        if k == "__metadata__" or not isinstance(meta, dict):
            continue
        if _ROUTED_EXPERT_KEY_RE.search(k):
            return meta.get("dtype")
    return None
```
**EN:** This callable implements `probe_routed_expert_weight_dtype`. It takes `model_path` and mainly implements probe routed expert weight dtype. The docstring states: "Return the safetensors dtype string (e.g." In this range it serializes or parses JSON payloads; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `probe_routed_expert_weight_dtype`。它接收 `model_path`，主要用于实现 probe routed expert weight dtype 相关逻辑。 在这一范围内，它会序列化或解析 JSON 数据；管理模型权重或检查点。

### Lines 131-136: Module-level constants and helpers
```python


# Block size for sequential checkpoint prefetch reads (page cache warming).
_PREFETCH_BLOCK_SIZE = None


```
**EN:** This range organizes module-level state and shared setup. In this range it manages model weights or checkpoints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会管理模型权重或检查点。

### Lines 137-143: Function _get_prefetch_block_size
```python
def _get_prefetch_block_size() -> int:
    global _PREFETCH_BLOCK_SIZE
    if _PREFETCH_BLOCK_SIZE is None:
        from sglang.srt.environ import envs

        _PREFETCH_BLOCK_SIZE = envs.SGLANG_PREFETCH_BLOCK_SIZE_MB.get() * 1024 * 1024
    return _PREFETCH_BLOCK_SIZE
```
**EN:** This callable implements `_get_prefetch_block_size` and mainly retrieves a value or derived view. In this range it sets up imports and shared symbols; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `_get_prefetch_block_size`，主要用于获取某个值或派生视图。 在这一范围内，它会建立导入关系并准备共享符号；读取环境变量驱动的配置。

### Lines 144-152: Module-level constants and helpers
```python


# use system-level temp directory for file locks, so that multiple users
# can share the same lock without error.
# lock files in the temp directory will be automatically deleted when the
# system reboots, so users will not complain about annoying lock files
temp_dir = tempfile.gettempdir()


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 153-164: Function get_lock
```python
def get_lock(
    model_name_or_path: str, cache_dir: Optional[str] = None, suffix: str = ""
):
    lock_dir = cache_dir or temp_dir
    os.makedirs(os.path.dirname(lock_dir), exist_ok=True)
    model_name = model_name_or_path.replace("/", "-")
    hash_name = hashlib.sha256(model_name.encode()).hexdigest()
    # add hash to avoid conflict with old users' lock files
    lock_file_name = hash_name + model_name + suffix + ".lock"
    # mode 0o666 is required for the filelock to be shared across users
    lock = filelock.FileLock(os.path.join(lock_dir, lock_file_name), mode=0o666)
    return lock
```
**EN:** This callable implements `get_lock`. It takes `model_name_or_path`, `cache_dir`, `suffix` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `get_lock`。它接收 `model_name_or_path`, `cache_dir`, `suffix`，主要用于获取某个值或派生视图。

### Lines 167-175: Function _shared_pointers
```python
def _shared_pointers(tensors):
    ptrs = defaultdict(list)
    for k, v in tensors.items():
        ptrs[v.data_ptr()].append(k)
    failing = []
    for _, names in ptrs.items():
        if len(names) > 1:
            failing.append(names)
    return failing
```
**EN:** This callable implements `_shared_pointers`. It takes `tensors` and mainly implements shared pointers.
**CN:** 这一可调用对象实现了 `_shared_pointers`。它接收 `tensors`，主要用于实现 shared pointers 相关逻辑。

### Lines 178-215: Function convert_bin_to_safetensor_file
```python
def convert_bin_to_safetensor_file(
    pt_filename: str,
    sf_filename: str,
) -> None:
    loaded = torch.load(pt_filename, map_location="cpu", weights_only=True)
    if "state_dict" in loaded:
        loaded = loaded["state_dict"]
    shared = _shared_pointers(loaded)
    for shared_weights in shared:
        for name in shared_weights[1:]:
            loaded.pop(name)

    # For tensors to be contiguous
    loaded = {k: v.contiguous() for k, v in loaded.items()}

    dirname = os.path.dirname(sf_filename)
    os.makedirs(dirname, exist_ok=True)

    from safetensors.torch import save_file

    save_file(loaded, sf_filename, metadata={"format": "pt"})

    # check file size
    sf_size = os.stat(sf_filename).st_size
    pt_size = os.stat(pt_filename).st_size
    if (sf_size - pt_size) / pt_size > 0.01:
        raise RuntimeError(f"""The file size different is more than 1%:
         - {sf_filename}: {sf_size}
         - {pt_filename}: {pt_size}
         """)

    # check if the tensors are the same
    reloaded = safetensors.torch.load_file(sf_filename)
    for k in loaded:
        pt_tensor = loaded[k]
        sf_tensor = reloaded[k]
        if not torch.equal(pt_tensor, sf_tensor):
            raise RuntimeError(f"The output tensors do not match for key {k}")
```
**EN:** This callable implements `convert_bin_to_safetensor_file`. It takes `pt_filename`, `sf_filename` and mainly converts data into another representation. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `convert_bin_to_safetensor_file`。它接收 `pt_filename`, `sf_filename`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；管理模型权重或检查点。

### Lines 218-222: Function replace_prefix
```python
def replace_prefix(key: str, prefix_mapping: dict[str, str]) -> str:
    for prefix, new_prefix in prefix_mapping.items():
        if key.startswith(prefix):
            key = key.replace(prefix, new_prefix, 1)
    return key
```
**EN:** This callable implements `replace_prefix`. It takes `key`, `prefix_mapping` and mainly implements replace prefix.
**CN:** 这一可调用对象实现了 `replace_prefix`。它接收 `key`, `prefix_mapping`，主要用于实现 replace prefix 相关逻辑。

### Lines 225-229: Function replace_substrings
```python
def replace_substrings(key: str, substring_mapping: dict[str, str]) -> str:
    for substr, new_substr in substring_mapping.items():
        if substr in key:
            key = key.replace(substr, new_substr)
    return key
```
**EN:** This callable implements `replace_substrings`. It takes `key`, `substring_mapping` and mainly implements replace substrings.
**CN:** 这一可调用对象实现了 `replace_substrings`。它接收 `key`, `substring_mapping`，主要用于实现 replace substrings 相关逻辑。

### Lines 232-232: Class DisabledTqdm
```python
class DisabledTqdm(tqdm):
```
**EN:** This range introduces `DisabledTqdm` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `DisabledTqdm`，并定义其后续方法依赖的结构或元数据。

### Lines 233-235: Method DisabledTqdm.__init__
```python
    def __init__(self, *args, **kwargs):
        kwargs["disable"] = True
        super().__init__(*args, **kwargs)
```
**EN:** This callable implements `DisabledTqdm.__init__`. It takes `*args`, `**kwargs` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `DisabledTqdm.__init__`。它接收 `*args`, `**kwargs`，主要用于初始化实例状态与默认值。

### Lines 236-238: Module-level logic
```python


# TODO(woosuk): Move this to other place.
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 239-298: Function get_quant_config (part 1/2)
```python
def get_quant_config(
    model_config: ModelConfig,
    load_config: LoadConfig,
    packed_modules_mapping: Dict[str, List[str]],
    remap_prefix: Dict[str, str] | None = None,
) -> QuantizationConfig:
    quant_cls = get_quantization_config(model_config.quantization)

    # GGUF doesn't have config file
    if model_config.quantization == "gguf":
        return quant_cls.from_config({})

    # Read the quantization config from the HF model config, if available.
    hf_quant_config = getattr(model_config.hf_config, "quantization_config", None)
    # some vision model may keep quantization_config in their text_config
    hf_text_config = getattr(model_config.hf_config, "text_config", None)
    if hf_quant_config is None and hf_text_config is not None:
        hf_quant_config = getattr(hf_text_config, "quantization_config", None)
    if hf_quant_config is None:
        # compressed-tensors uses a compressions_config
        hf_quant_config = getattr(model_config.hf_config, "compression_config", None)
    if hf_quant_config is not None:
        if not isinstance(hf_quant_config, dict):
            hf_quant_config = hf_quant_config.to_dict()
        hf_quant_config["packed_modules_mapping"] = packed_modules_mapping
        return quant_cls.from_config(hf_quant_config)

    # In case of bitsandbytes/QLoRA, get quant config from the adapter model.
    if model_config.quantization == "bitsandbytes":
        if (
            not load_config.model_loader_extra_config
            or "qlora_adapter_name_or_path" not in load_config.model_loader_extra_config
        ):
            return quant_cls.from_config({"adapter_name_or_path": ""})
        model_name_or_path = load_config.model_loader_extra_config[
            "qlora_adapter_name_or_path"
        ]
    else:
        model_name_or_path = model_config.model_path

    is_local = os.path.isdir(model_name_or_path)
    if not is_local:
        # Download the config files.
        with get_lock(model_name_or_path, load_config.download_dir):
            hf_folder = snapshot_download(
                model_name_or_path,
                revision=model_config.revision,
                allow_patterns="*.json",
                cache_dir=load_config.download_dir,
                local_files_only=huggingface_hub.constants.HF_HUB_OFFLINE,
                tqdm_class=DisabledTqdm,
            )
    else:
        hf_folder = model_name_or_path

    possible_config_filenames = quant_cls.get_config_filenames()

    # If the quantization config is not found, use the default config.
    if not possible_config_filenames:
        if model_config.quantization == "mxfp8":
```
**EN:** This callable implements `get_quant_config`. It takes `model_config`, `load_config`, `packed_modules_mapping`, `remap_prefix` and mainly retrieves a value or derived view. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `get_quant_config`。它接收 `model_config`, `load_config`, `packed_modules_mapping`, `remap_prefix`，主要用于获取某个值或派生视图。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 299-344: Function get_quant_config (part 2/2)
```python
            return Fp8Config(use_mxfp8=True, is_checkpoint_fp8_serialized=False)
        return quant_cls()

    config_files = glob.glob(os.path.join(hf_folder, "*.json"))

    quant_config_files = [
        f for f in config_files if any(f.endswith(x) for x in possible_config_filenames)
    ]
    if len(quant_config_files) == 0:
        raise ValueError(f"Cannot find the config file for {model_config.quantization}")
    if len(quant_config_files) > 1:
        raise ValueError(
            f"Found multiple config files for {model_config.quantization}: "
            f"{quant_config_files}"
        )

    quant_config_file = quant_config_files[0]
    with open(quant_config_file) as f:
        config = json.load(f)
        if remap_prefix is not None:
            exclude_modules = [
                replace_prefix(key, remap_prefix)
                for key in config["quantization"]["exclude_modules"]
            ]
            config["quantization"]["exclude_modules"] = exclude_modules
        config["packed_modules_mapping"] = packed_modules_mapping

        if model_config.quantization == "bitsandbytes":
            config["adapter_name_or_path"] = model_name_or_path
        elif model_config.quantization.startswith("modelopt") and (
            config.get("producer", {}).get("name", "").startswith("modelopt")
        ):
            quant_algo = config["quantization"]["quant_algo"]
            if quant_algo is None:
                # (yizhang2077) workaround for nvidia/Llama-4-Maverick-17B-128E-Eagle3
                if model_config.hf_config.architectures[0] != "LlamaForCausalLMEagle3":
                    raise ValueError(
                        f"Invalid quant_config, quantization method: {model_config.quantization},"
                        f"hf architectures: {model_config.hf_config.architectures[0]}. "
                    )
                return None
            elif quant_algo == "FP8" or model_config.quantization == "modelopt_fp8":
                return ModelOptFp8Config.from_config(config)
            elif "FP4" in quant_algo:
                return ModelOptFp4Config.from_config(config)
        return quant_cls.from_config(config)
```
**EN:** This callable implements `get_quant_config`. It takes `model_config`, `load_config`, `packed_modules_mapping`, `remap_prefix` and mainly retrieves a value or derived view. This chunk is part 2 of 2 for the same logical block. In this range it performs defensive checks on invalid state; serializes or parses JSON payloads; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `get_quant_config`。它接收 `model_config`, `load_config`, `packed_modules_mapping`, `remap_prefix`，主要用于获取某个值或派生视图。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会对非法状态执行防御性检查；序列化或解析 JSON 数据；管理模型权重或检查点。

### Lines 347-392: Function _check_index_files_exist
```python
def _check_index_files_exist(snapshot_dir: str) -> Tuple[bool, Optional[str]]:
    """
    Check if all files listed in safetensors index files actually exist on disk.

    This catches cases where the snapshot directory exists but files are missing
    (e.g., due to incomplete downloads or corrupted cache).

    Args:
        snapshot_dir: Path to the model snapshot directory

    Returns:
        Tuple of (all_exist, error_message)
    """
    index_files = [
        f for f in os.listdir(snapshot_dir) if f.endswith(".safetensors.index.json")
    ]

    if not index_files:
        return True, None  # Not a sharded model

    for index_file in index_files:
        index_path = os.path.join(snapshot_dir, index_file)
        if not os.path.exists(index_path):
            continue
        try:
            with open(index_path) as f:
                weight_map = json.load(f).get("weight_map", {})
            if not weight_map:
                continue
            required_files = set(weight_map.values())
            missing_files = [
                fn
                for fn in required_files
                if not os.path.exists(os.path.join(snapshot_dir, fn))
            ]
            if missing_files:
                return (
                    False,
                    f"Missing {len(missing_files)} file(s) from index {index_file}: "
                    f"{missing_files[:3]}{'...' if len(missing_files) > 3 else ''}",
                )
        except Exception as e:
            logger.warning("Failed to read index file %s: %s", index_file, e)
            continue

    return True, None
```
**EN:** This callable implements `_check_index_files_exist`. It takes `snapshot_dir` and mainly checks preconditions and compatibility. The docstring states: "Check if all files listed in safetensors index files actually exist on disk." In this range it sets up imports and shared symbols; emits logs for diagnostics; serializes or parses JSON payloads.
**CN:** 这一可调用对象实现了 `_check_index_files_exist`。它接收 `snapshot_dir`，主要用于检查前置条件与兼容性。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；序列化或解析 JSON 数据。

### Lines 395-454: Function _find_local_hf_snapshot_dir_unlocked (part 1/2)
```python
def _find_local_hf_snapshot_dir_unlocked(
    model_name_or_path: str,
    cache_dir: Optional[str],
    allow_patterns: List[str],
    revision: Optional[str] = None,
) -> Optional[str]:
    """Find local HF snapshot directory without locking.

    IMPORTANT: Caller MUST hold the model lock before calling this function
    to prevent race conditions during validation and cleanup.

    If the weights are already local, skip downloading and returns the path.
    """
    if os.path.isdir(model_name_or_path):
        return None

    found_local_snapshot_dir = None

    # Check custom cache_dir (if provided)
    if cache_dir:
        try:
            repo_folder = os.path.join(
                cache_dir,
                huggingface_hub.constants.REPO_ID_SEPARATOR.join(
                    ["models", *model_name_or_path.split("/")]
                ),
            )
            rev_to_use = revision
            if not rev_to_use:
                ref_main = os.path.join(repo_folder, "refs", "main")
                if os.path.isfile(ref_main):
                    with open(ref_main) as f:
                        rev_to_use = f.read().strip()
            if rev_to_use:
                rev_dir = os.path.join(repo_folder, "snapshots", rev_to_use)
                if os.path.isdir(rev_dir):
                    found_local_snapshot_dir = rev_dir
        except Exception as e:
            logger.warning(
                "Failed to find local snapshot in custom cache_dir %s: %s",
                cache_dir,
                e,
            )

    # Check default HF cache as well
    if not found_local_snapshot_dir:
        try:
            rev_dir = find_local_repo_dir(model_name_or_path, revision)
            if rev_dir and os.path.isdir(rev_dir):
                found_local_snapshot_dir = rev_dir
        except Exception as e:
            logger.warning("Failed to find local snapshot in default HF cache: %s", e)

    # if local snapshot exists, validate it contains at least one weight file
    # matching allow_patterns before skipping download.
    if found_local_snapshot_dir is None:
        return None

    # Check if snapshot dir exists (might have been cleaned by another process
    # before we acquired the lock)
```
**EN:** This callable implements `_find_local_hf_snapshot_dir_unlocked`. It takes `model_name_or_path`, `cache_dir`, `allow_patterns`, `revision` and mainly implements find local hf snapshot dir unlocked. The docstring states: "Find local HF snapshot directory without locking." This chunk is part 1 of 2 for the same logical block. In this range it emits logs for diagnostics; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `_find_local_hf_snapshot_dir_unlocked`。它接收 `model_name_or_path`, `cache_dir`, `allow_patterns`, `revision`，主要用于实现 find local hf snapshot dir unlocked 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会输出日志以便诊断；管理模型权重或检查点。

### Lines 455-510: Function _find_local_hf_snapshot_dir_unlocked (part 2/2)
```python
    if not os.path.isdir(found_local_snapshot_dir):
        return None

    local_weight_files: List[str] = []
    try:
        for pattern in allow_patterns:
            matched_files = glob.glob(os.path.join(found_local_snapshot_dir, pattern))
            for f in matched_files:
                # os.path.exists returns False for broken symlinks.
                if not os.path.exists(f):
                    continue
                local_weight_files.append(f)
    except Exception as e:
        logger.warning(
            "Failed to scan local snapshot %s with patterns %s: %s",
            found_local_snapshot_dir,
            allow_patterns,
            e,
        )
        local_weight_files = []

    # Check for missing files from index (lightweight, for all users)
    # This catches incomplete downloads before they cause cryptic load errors
    if local_weight_files:
        is_complete, error_msg = _check_index_files_exist(found_local_snapshot_dir)
        if not is_complete:
            log_info_on_rank0(
                logger,
                f"Local snapshot incomplete for {model_name_or_path}: {error_msg}. "
                f"Will download missing files.",
            )
            return None  # Triggers snapshot_download() which handles partial downloads

    # Only perform cache validation and cleanup in CI to avoid
    # unnecessary overhead for regular users
    if is_in_ci() and local_weight_files:
        is_valid = ci_validate_and_cleanup_local_snapshot(
            model_name_or_path, found_local_snapshot_dir, local_weight_files
        )
        if not is_valid:
            return None

    if len(local_weight_files) > 0:
        log_info_on_rank0(
            logger,
            f"Found local HF snapshot for {model_name_or_path} at "
            f"{found_local_snapshot_dir}; skipping download.",
        )
        return found_local_snapshot_dir
    else:
        log_info_on_rank0(
            logger,
            f"Local HF snapshot at {found_local_snapshot_dir} has no files matching "
            f"{allow_patterns}; will attempt download.",
        )
        return None
```
**EN:** This callable implements `_find_local_hf_snapshot_dir_unlocked`. It takes `model_name_or_path`, `cache_dir`, `allow_patterns`, `revision` and mainly implements find local hf snapshot dir unlocked. The docstring states: "Find local HF snapshot directory without locking." This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols; emits logs for diagnostics; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `_find_local_hf_snapshot_dir_unlocked`。它接收 `model_name_or_path`, `cache_dir`, `allow_patterns`, `revision`，主要用于实现 find local hf snapshot dir unlocked 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；管理模型权重或检查点。

### Lines 513-572: Function download_weights_from_hf (part 1/2)
```python
def download_weights_from_hf(
    model_name_or_path: str,
    cache_dir: Optional[str],
    allow_patterns: List[str],
    revision: Optional[str] = None,
    ignore_patterns: Optional[Union[str, List[str]]] = None,
    max_retries: int = 3,
) -> str:
    """Download model weights from Hugging Face Hub.

    Args:
        model_name_or_path (str): The model name or path.
        cache_dir (Optional[str]): The cache directory to store the model
            weights. If None, will use HF defaults.
        allow_patterns (List[str]): The allowed patterns for the
            weight files. Files matched by any of the patterns will be
            downloaded.
        revision (Optional[str]): The revision of the model.
        ignore_patterns (Optional[Union[str, List[str]]]): The patterns to
            filter out the weight files. Files matched by any of the patterns
            will be ignored.
        max_retries (int): Maximum number of download retries if corruption
            is detected. Defaults to 3.

    Returns:
        str: The path to the downloaded model weights.
    """
    # For local paths, no HF operations needed
    if os.path.isdir(model_name_or_path):
        return model_name_or_path

    # Use a SINGLE lock for the entire operation (validation + cleanup + download)
    # to prevent race conditions where:
    # 1. Process A validates, finds corruption, deletes corrupted file
    # 2. Process B validates, sees missing file, deletes ENTIRE cache
    # 3. Process A tries to download but cache is gone
    # By using one lock, validation/cleanup and download are atomic.
    with get_lock(model_name_or_path, cache_dir):
        # Check for valid local cache first (validates and cleans up if needed)
        path = _find_local_hf_snapshot_dir_unlocked(
            model_name_or_path, cache_dir, allow_patterns, revision
        )
        if path is not None:
            # Valid local cache found, skip download
            return path

        # In CI, skip HF API calls if we're in offline mode or want to avoid rate limits
        # But we already checked for local cache above, so if we're here we need to download
        if not huggingface_hub.constants.HF_HUB_OFFLINE:
            # Before we download we look at what is available:
            fs = HfFileSystem()
            file_list = fs.ls(model_name_or_path, detail=False, revision=revision)

            # depending on what is available we download different things
            for pattern in allow_patterns:
                matching = fnmatch.filter(file_list, pattern)
                if len(matching) > 0:
                    allow_patterns = [pattern]
                    break

```
**EN:** This callable implements `download_weights_from_hf`. It takes `model_name_or_path`, `cache_dir`, `allow_patterns`, `revision` and mainly constructs data from an external representation. The docstring states: "Download model weights from Hugging Face Hub." This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `download_weights_from_hf`。它接收 `model_name_or_path`, `cache_dir`, `allow_patterns`, `revision`，主要用于从外部表示构造数据。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；管理模型权重或检查点。

### Lines 573-596: Function download_weights_from_hf (part 2/2)
```python
        log_info_on_rank0(logger, f"Using model weights format {allow_patterns}")

        if not is_in_ci():
            # Simple download without validation for non-CI environments
            hf_folder = snapshot_download(
                model_name_or_path,
                allow_patterns=allow_patterns,
                ignore_patterns=ignore_patterns,
                cache_dir=cache_dir,
                tqdm_class=DisabledTqdm,
                revision=revision,
                local_files_only=huggingface_hub.constants.HF_HUB_OFFLINE,
            )
            return hf_folder
        else:
            # Only perform validation and retry in CI to avoid overhead for regular users
            return ci_download_with_validation_and_retry(
                model_name_or_path=model_name_or_path,
                allow_patterns=allow_patterns,
                ignore_patterns=ignore_patterns,
                cache_dir=cache_dir,
                revision=revision,
                max_retries=max_retries,
            )
```
**EN:** This callable implements `download_weights_from_hf`. It takes `model_name_or_path`, `cache_dir`, `allow_patterns`, `revision` and mainly constructs data from an external representation. The docstring states: "Download model weights from Hugging Face Hub." This chunk is part 2 of 2 for the same logical block. In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `download_weights_from_hf`。它接收 `model_name_or_path`, `cache_dir`, `allow_patterns`, `revision`，主要用于从外部表示构造数据。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会管理模型权重或检查点。

### Lines 599-630: Function download_safetensors_index_file_from_hf
```python
def download_safetensors_index_file_from_hf(
    model_name_or_path: str,
    index_file: str,
    cache_dir: Optional[str],
    revision: Optional[str] = None,
) -> None:
    """Download hf safetensors index file from Hugging Face Hub.

    Args:
        model_name_or_path (str): The model name or path.
        cache_dir (Optional[str]): The cache directory to store the model
            weights. If None, will use HF defaults.
        revision (Optional[str]): The revision of the model.
    """
    # Use file lock to prevent multiple processes from
    # downloading the same model weights at the same time.
    with get_lock(model_name_or_path, cache_dir):
        try:
            # Download the safetensors index file.
            hf_hub_download(
                repo_id=model_name_or_path,
                filename=index_file,
                cache_dir=cache_dir,
                revision=revision,
                local_files_only=huggingface_hub.constants.HF_HUB_OFFLINE,
            )
        # If file not found on remote or locally, we should not fail since
        # only some models will have index_file.
        except huggingface_hub.utils.EntryNotFoundError:
            logger.debug("No %s found in remote.", index_file)
        except huggingface_hub.utils.LocalEntryNotFoundError:
            logger.debug("No %s found in local cache.", index_file)
```
**EN:** This callable implements `download_safetensors_index_file_from_hf`. It takes `model_name_or_path`, `index_file`, `cache_dir`, `revision` and mainly constructs data from an external representation. The docstring states: "Download hf safetensors index file from Hugging Face Hub." In this range it sets up imports and shared symbols; emits logs for diagnostics; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `download_safetensors_index_file_from_hf`。它接收 `model_name_or_path`, `index_file`, `cache_dir`, `revision`，主要用于从外部表示构造数据。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；管理模型权重或检查点。

### Lines 631-637: Module-level logic
```python


# For models like Mistral-7B-v0.3, there are both sharded
# safetensors files and a consolidated safetensors file.
# Passing both of these to the weight loader functionality breaks.
# So, we use the index_file to
# look up which safetensors files should be used.
```
**EN:** This range organizes module-level state and shared setup. In this range it manages model weights or checkpoints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会管理模型权重或检查点。

### Lines 638-664: Function filter_duplicate_safetensors_files
```python
def filter_duplicate_safetensors_files(
    hf_weights_files: List[str], hf_folder: str, index_file: str
) -> List[str]:
    # model.safetensors.index.json is a mapping from keys in the
    # torch state_dict to safetensors file holding that weight.
    index_file_name = os.path.join(hf_folder, index_file)
    if not os.path.isfile(index_file_name):
        # NOTE: this is a trick of handling mistral model
        # skip the unsupported consolidated.safetensors file
        if len(hf_weights_files) == 2:
            hf_weights_files.sort()
            if hf_weights_files[0].endswith(
                "consolidated.safetensors"
            ) and hf_weights_files[1].endswith("model.safetensors"):
                return [hf_weights_files[1]]
        return hf_weights_files

    # Iterate through the weight_map (weight_name: safetensors files)
    # to identify weights that we should use.
    with open(index_file_name) as f:
        weight_map = json.load(f)["weight_map"]
    weight_files_in_index = set()
    for weight_name in weight_map:
        weight_files_in_index.add(os.path.join(hf_folder, weight_map[weight_name]))
    # Filter out any fields that are not found in the index file.
    hf_weights_files = [f for f in hf_weights_files if f in weight_files_in_index]
    return hf_weights_files
```
**EN:** This callable implements `filter_duplicate_safetensors_files`. It takes `hf_weights_files`, `hf_folder`, `index_file` and mainly implements filter duplicate safetensors files. In this range it sets up imports and shared symbols; serializes or parses JSON payloads; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `filter_duplicate_safetensors_files`。它接收 `hf_weights_files`, `hf_folder`, `index_file`，主要用于实现 filter duplicate safetensors files 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；序列化或解析 JSON 数据；管理模型权重或检查点。

### Lines 667-706: Function maybe_add_mtp_safetensors
```python
def maybe_add_mtp_safetensors(
    hf_weights_files: List[str], hf_folder: str, index_file: str, hf_config
) -> List[str]:
    """
    Auto-detect and add mtp.safetensors for GLM4Moe MTP/NextN models if:
    1. mtp.safetensors exists in the model directory
    2. mtp.safetensors is NOT in the index (checkpoint packaging bug)
    3. Model architecture is Glm4MoeForCausalLM with num_nextn_predict_layers > 0

    This works around incorrectly packaged FP4 checkpoints like
    baseten-admin/glm-4.7-fp4 where mtp.safetensors exists but
    isn't referenced in model.safetensors.index.json.
    """
    # Only apply for GLM4Moe architecture with nextn layers
    arch = getattr(hf_config, "architectures", [None])[0]
    num_nextn_layers = getattr(
        getattr(hf_config, "text_config", hf_config),
        "num_nextn_predict_layers",
        getattr(hf_config, "num_nextn_predict_layers", 0),
    )
    if not (
        arch in ["Glm4MoeForCausalLM", "Glm4MoeForCausalLMNextN"]
        and num_nextn_layers > 0
    ):
        return hf_weights_files

    # Check if mtp.safetensors exists and is not already in the file list
    mtp_path = os.path.join(hf_folder, "mtp.safetensors")
    if not os.path.isfile(mtp_path) or mtp_path in hf_weights_files:
        return hf_weights_files

    # mtp.safetensors exists but not in index - this is a bug
    logger.warning(
        f"Found mtp.safetensors but it's not referenced in {index_file}. "
        f"This is a checkpoint packaging bug. Auto-adding it for loading. "
        f"Please report this to the checkpoint provider."
    )

    # Add it to the files list
    return hf_weights_files + [mtp_path]
```
**EN:** This callable implements `maybe_add_mtp_safetensors`. It takes `hf_weights_files`, `hf_folder`, `index_file`, `hf_config` and mainly adds configuration entries or arguments. The docstring states: "Auto-detect and add mtp.safetensors for GLM4Moe MTP/NextN models if: 1." In this range it emits logs for diagnostics; serializes or parses JSON payloads; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `maybe_add_mtp_safetensors`。它接收 `hf_weights_files`, `hf_folder`, `index_file`, `hf_config`，主要用于添加配置项或参数。 在这一范围内，它会输出日志以便诊断；序列化或解析 JSON 数据；管理模型权重或检查点。

### Lines 709-725: Function filter_files_not_needed_for_inference
```python
def filter_files_not_needed_for_inference(hf_weights_files: List[str]) -> List[str]:
    """
    Exclude files that are not needed for inference.

    See https://github.com/huggingface/transformers/blob/v4.34.0/src/transformers/trainer.py#L227-L233
    """
    blacklist = [
        "training_args.bin",
        "optimizer.bin",
        "optimizer.pt",
        "scheduler.pt",
        "scaler.pt",
    ]
    hf_weights_files = [
        f for f in hf_weights_files if not any(f.endswith(x) for x in blacklist)
    ]
    return hf_weights_files
```
**EN:** This callable implements `filter_files_not_needed_for_inference`. It takes `hf_weights_files` and mainly implements filter files not needed for inference. The docstring states: "Exclude files that are not needed for inference." In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `filter_files_not_needed_for_inference`。它接收 `hf_weights_files`，主要用于实现 filter files not needed for inference 相关逻辑。 在这一范围内，它会管理模型权重或检查点。

### Lines 728-774: Function np_cache_weights_iterator
```python
def np_cache_weights_iterator(
    model_name_or_path: str,
    cache_dir: Optional[str],
    hf_folder: str,
    hf_weights_files: List[str],
) -> Generator[Tuple[str, torch.Tensor], None, None]:
    """Iterate over the weights in the model np files.

    Will dump the model weights to numpy files if they are not already dumped.
    """
    enable_tqdm = (
        not torch.distributed.is_initialized() or torch.distributed.get_rank() == 0
    )
    # Convert the model weights from torch tensors to numpy arrays for
    # faster loading.
    np_folder = os.path.join(hf_folder, "np")
    os.makedirs(np_folder, exist_ok=True)
    weight_names_file = os.path.join(np_folder, "weight_names.json")
    # Use file lock to prevent multiple processes from
    # dumping the same model weights to numpy at the same time.
    with get_lock(model_name_or_path, cache_dir):
        if not os.path.exists(weight_names_file):
            weight_names: List[str] = []
            for bin_file in tqdm(
                hf_weights_files,
                desc="Loading np_cache checkpoint shards",
                disable=not enable_tqdm,
                bar_format=BAR_FORMAT,
                position=tqdm._get_free_pos(),
            ):
                state = torch.load(bin_file, map_location="cpu", weights_only=True)
                for name, param in state.items():
                    param_path = os.path.join(np_folder, name)
                    with open(param_path, "wb") as f:
                        np.save(f, param.cpu().detach().numpy())
                    weight_names.append(name)
            with open(weight_names_file, "w") as f:
                json.dump(weight_names, f)

    with open(weight_names_file) as f:
        weight_names = json.load(f)

    for name in weight_names:
        param_path = os.path.join(np_folder, name)
        with open(param_path, "rb") as f:
            param = np.load(f)
        yield name, torch.from_numpy(param)
```
**EN:** This callable implements `np_cache_weights_iterator`. It takes `model_name_or_path`, `cache_dir`, `hf_folder`, `hf_weights_files` and mainly converts data into another representation. The docstring states: "Iterate over the weights in the model np files." In this range it sets up imports and shared symbols; coordinates distributed communication; serializes or parses JSON payloads.
**CN:** 这一可调用对象实现了 `np_cache_weights_iterator`。它接收 `model_name_or_path`, `cache_dir`, `hf_folder`, `hf_weights_files`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信；序列化或解析 JSON 数据。

### Lines 777-785: Function _prefetch_checkpoint_file
```python
def _prefetch_checkpoint_file(file_path: str) -> None:
    """Prefetch a checkpoint file into the OS page cache.

    Reads the file sequentially in 16 MB blocks so the kernel caches its pages
    before workers load the same file via mmap.
    """
    with open(file_path, "rb") as f:
        while f.read(_get_prefetch_block_size()):
            pass
```
**EN:** This callable implements `_prefetch_checkpoint_file`. It takes `file_path` and mainly checks preconditions and compatibility. The docstring states: "Prefetch a checkpoint file into the OS page cache." In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `_prefetch_checkpoint_file`。它接收 `file_path`，主要用于检查前置条件与兼容性。 在这一范围内，它会管理模型权重或检查点。

### Lines 788-847: Function _prefetch_all_checkpoints (part 1/2)
```python
def _prefetch_all_checkpoints(
    sorted_files: List[str],
    num_threads: int = 4,
) -> None:
    """Start prefetching checkpoint files into page cache in a background thread.

    When multiple ranks on the same node load the same checkpoint (e.g.
    DP-attention), each rank independently mmaps the same files, causing
    redundant NFS/Lustre reads. By distributing the prefetch across ranks
    (each rank reads 1/Nth of the shards), the total network I/O is reduced
    from N * checkpoint_size to 1 * checkpoint_size, with subsequent
    mmap accesses hitting the shared OS page cache.

    The prefetch runs in a background thread so that loading can start
    immediately and benefit from pages that have already been cached,
    rather than blocking until all files are prefetched. This pipelining
    naturally adapts to any RAM size — even if the full checkpoint does
    not fit in page cache, the prefetch thread stays ahead of the loader.
    """
    import asyncio
    import threading
    import time

    # Use node-local rank so that each node independently prefetches the
    # full checkpoint into its own page cache. Global rank would split files
    # across nodes, but page cache is not shared across nodes.
    if torch.distributed.is_initialized():
        world_group = get_world_group()
        local_rank = world_group.local_rank
        local_world_size = world_group.local_size or world_group.world_size
    else:
        local_rank = 0
        local_world_size = 1

    my_files = sorted_files[local_rank::local_world_size]
    total_for_rank = len(my_files)

    logger.info(
        "Rank %d: prefetching %d/%d checkpoint shards into page cache "
        "(background, %d local ranks sharing the work, %d threads per rank)...",
        local_rank,
        total_for_rank,
        len(sorted_files),
        local_world_size,
        num_threads,
    )

    async def _prefetch_all() -> None:
        semaphore = asyncio.Semaphore(num_threads)
        completed = 0
        next_log_pct = 10

        async def prefetch_one(path: str) -> None:
            nonlocal completed, next_log_pct
            try:
                async with semaphore:
                    await asyncio.to_thread(_prefetch_checkpoint_file, path)
                completed += 1
                if total_for_rank > 0 and next_log_pct <= 100:
                    pct = 100 * completed / total_for_rank
```
**EN:** This callable implements `_prefetch_all_checkpoints`. It takes `sorted_files`, `num_threads` and mainly checks preconditions and compatibility. The docstring states: "Start prefetching checkpoint files into page cache in a background thread." This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `_prefetch_all_checkpoints`。它接收 `sorted_files`, `num_threads`，主要用于检查前置条件与兼容性。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 848-877: Function _prefetch_all_checkpoints (part 2/2)
```python
                    if pct >= next_log_pct:
                        logger.info(
                            "Rank %d: prefetching checkpoint files: %d%% (%d/%d)",
                            local_rank,
                            next_log_pct,
                            completed,
                            total_for_rank,
                        )
                        next_log_pct += 10
            except Exception:
                logger.warning(
                    "Failed to prefetch checkpoint file %r.",
                    path,
                    exc_info=True,
                )

        await asyncio.gather(*(prefetch_one(p) for p in my_files))

    def _run_prefetch() -> None:
        start = time.perf_counter()
        asyncio.run(_prefetch_all())
        elapsed = time.perf_counter() - start
        logger.info(
            "Rank %d: prefetching checkpoint files into page cache "
            "finished in %.2fs",
            local_rank,
            elapsed,
        )

    threading.Thread(target=_run_prefetch, daemon=True).start()
```
**EN:** This callable implements `_prefetch_all_checkpoints`. It takes `sorted_files`, `num_threads` and mainly checks preconditions and compatibility. The docstring states: "Start prefetching checkpoint files into page cache in a background thread." This chunk is part 2 of 2 for the same logical block. In this range it emits logs for diagnostics; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `_prefetch_all_checkpoints`。它接收 `sorted_files`, `num_threads`，主要用于检查前置条件与兼容性。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会输出日志以便诊断；管理模型权重或检查点。

### Lines 880-895: Function _drop_file_cache_after_load
```python
def _drop_file_cache_after_load(path: str) -> None:
    """Release of checkpoint pages after weights have been copied out. Used to avoid CPU OOM in RL."""
    posix_fadvise = getattr(os, "posix_fadvise", None)
    dontneed = getattr(os, "POSIX_FADV_DONTNEED", None)
    if posix_fadvise is None or dontneed is None:
        return

    fd = None
    try:
        fd = os.open(path, os.O_RDONLY)
        posix_fadvise(fd, 0, 0, dontneed)
    except OSError as e:
        logger.debug("Failed to drop file cache for %s: %s", path, e)
    finally:
        if fd is not None:
            os.close(fd)
```
**EN:** This callable implements `_drop_file_cache_after_load`. It takes `path` and mainly loads external data or weights. The docstring states: "Release of checkpoint pages after weights have been copied out." In this range it emits logs for diagnostics; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `_drop_file_cache_after_load`。它接收 `path`，主要用于加载外部数据或权重。 在这一范围内，它会输出日志以便诊断；管理模型权重或检查点。

### Lines 898-932: Function safetensors_weights_iterator
```python
def safetensors_weights_iterator(
    hf_weights_files: List[str],
    disable_mmap: bool = False,
    prefetch: bool = False,
    prefetch_num_threads: int = 4,
    drop_cache_after_load: bool = False,
) -> Generator[Tuple[str, torch.Tensor], None, None]:
    """Iterate over the weights in the model safetensor files."""
    enable_tqdm = (
        not torch.distributed.is_initialized() or torch.distributed.get_rank() == 0
    )

    sorted_files = sorted(hf_weights_files)

    if prefetch and not disable_mmap:
        _prefetch_all_checkpoints(sorted_files, num_threads=prefetch_num_threads)

    for st_file in tqdm(
        sorted_files,
        desc="Loading safetensors checkpoint shards",
        disable=not enable_tqdm,
        bar_format=BAR_FORMAT,
        position=tqdm._get_free_pos(),
    ):
        if disable_mmap:
            with open(st_file, "rb") as f:
                result = safetensors.torch.load(f.read())
                for name in sorted(result.keys()):
                    yield name, result[name]
        else:
            with safetensors.safe_open(st_file, framework="pt", device="cpu") as f:
                for name in f.keys():
                    yield name, f.get_tensor(name)
        if drop_cache_after_load:
            _drop_file_cache_after_load(st_file)
```
**EN:** This callable implements `safetensors_weights_iterator`. It takes `hf_weights_files`, `disable_mmap`, `prefetch`, `prefetch_num_threads` and mainly converts data into another representation. The docstring states: "Iterate over the weights in the model safetensor files." In this range it coordinates distributed communication; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `safetensors_weights_iterator`。它接收 `hf_weights_files`, `disable_mmap`, `prefetch`, `prefetch_num_threads`，主要用于将数据转换为另一种表示。 在这一范围内，它会协调分布式通信；管理模型权重或检查点。

### Lines 935-987: Function fastsafetensors_weights_iterator
```python
def fastsafetensors_weights_iterator(
    hf_weights_files: List[str],
) -> Generator[Tuple[str, torch.Tensor], None, None]:
    """
    Iterate over the weights in the model safetensor files
    using fastsafetensor library to accelerate loading via GPU Direct Storage (if available).
    """
    if SafeTensorsFileLoader is None:
        raise ImportError(
            "Please install fastsafetensors via `pip install fastsafetensors`"
        )

    if torch.distributed.is_initialized():
        pg = torch.distributed.group.WORLD
    else:
        pg = SingleGroup()

    try:
        rank = pg.rank()
    except Exception:
        rank = 0

    device = torch.device(f"cuda:{rank}")

    weight_files_sub_lists = [
        hf_weights_files[i : i + pg.size()]
        for i in range(0, len(hf_weights_files), pg.size())
    ]

    _BAR_FORMAT = (
        "{l_bar}{bar}| {n_fmt}/{total_fmt} [{elapsed}<{remaining}, {rate_fmt}]"
    )

    for f_list in tqdm(
        weight_files_sub_lists,
        desc="Loading safetensors using Fastsafetensor loader",
        disable=False,
        bar_format=_BAR_FORMAT,
    ):
        loader = SafeTensorsFileLoader(pg, device)
        rank_file_map = {i: [f] for i, f in enumerate(f_list)}
        loader.add_filenames(rank_file_map)
        try:
            fb = loader.copy_files_to_device()
            try:
                keys = list(fb.key_to_rank_lidx.keys())
                for k in keys:
                    t = fb.get_tensor(k)
                    yield k, t
            finally:
                pass
        finally:
            loader.close()
```
**EN:** This callable implements `fastsafetensors_weights_iterator`. It takes `hf_weights_files` and mainly converts data into another representation. The docstring states: "Iterate over the weights in the model safetensor files using fastsafetensor library to accelerate loading via GPU Direct Storage (if available)." In this range it performs defensive checks on invalid state; coordinates distributed communication; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `fastsafetensors_weights_iterator`。它接收 `hf_weights_files`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信；管理模型权重或检查点。

### Lines 990-1031: Function multi_thread_safetensors_weights_iterator
```python
def multi_thread_safetensors_weights_iterator(
    hf_weights_files: List[str],
    max_workers: int,
    disable_mmap: bool = False,
    drop_cache_after_load: bool = False,
) -> Generator[Tuple[str, torch.Tensor], None, None]:
    """Multi-Thread iterate over the weights in the model safetensor files."""
    enable_tqdm = (
        not torch.distributed.is_initialized() or torch.distributed.get_rank() == 0
    )

    def _load_file(st_file: str):
        if disable_mmap:
            with open(st_file, "rb") as f:
                result = safetensors.torch.load(f.read())
        else:
            with safetensors.safe_open(st_file, framework="pt", device="cpu") as f:
                result = {k: f.get_tensor(k) for k in f.keys()}

        return st_file, result

    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = [executor.submit(_load_file, st_file) for st_file in hf_weights_files]

        if enable_tqdm:
            futures_iter = tqdm(
                concurrent.futures.as_completed(futures),
                total=len(hf_weights_files),
                desc="Multi-thread loading shards",
                disable=not enable_tqdm,
                bar_format=BAR_FORMAT,
            )
        else:
            futures_iter = concurrent.futures.as_completed(futures)

        for future in futures_iter:
            st_file, state_dict = future.result()
            for name, param in state_dict.items():
                yield name, param
            del state_dict
            if drop_cache_after_load:
                _drop_file_cache_after_load(st_file)
```
**EN:** This callable implements `multi_thread_safetensors_weights_iterator`. It takes `hf_weights_files`, `max_workers`, `disable_mmap`, `drop_cache_after_load` and mainly converts data into another representation. The docstring states: "Multi-Thread iterate over the weights in the model safetensor files." In this range it coordinates distributed communication; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `multi_thread_safetensors_weights_iterator`。它接收 `hf_weights_files`, `max_workers`, `disable_mmap`, `drop_cache_after_load`，主要用于将数据转换为另一种表示。 在这一范围内，它会协调分布式通信；管理模型权重或检查点。

### Lines 1034-1093: Function buffered_multi_thread_safetensors_weights_iterator (part 1/2)
```python
def buffered_multi_thread_safetensors_weights_iterator(
    hf_weights_files: List[str],
    max_workers: int,
    disable_mmap: bool = False,
    prefetch: bool = False,
    prefetch_num_threads: int = 4,
    drop_cache_after_load: bool = False,
) -> Generator[Tuple[str, torch.Tensor], None, None]:
    """Multi-threaded safetensor loader with bounded memory via a sliding window.

    At most (max_workers + 1) shard files are in-flight at any time:
    max_workers loading concurrently + 1 prefetched and ready to yield.
    Peak CPU RAM ≈ (max_workers + 2) × shard_file_size.
    """
    sorted_files = sorted(hf_weights_files)
    if prefetch and not disable_mmap:
        _prefetch_all_checkpoints(sorted_files, num_threads=prefetch_num_threads)
    enable_tqdm = (
        not torch.distributed.is_initialized() or torch.distributed.get_rank() == 0
    )

    def _load_file(st_file: str):
        if disable_mmap:
            with open(st_file, "rb") as f:
                result = safetensors.torch.load(f.read())
        else:
            with safetensors.safe_open(st_file, framework="pt", device="cpu") as f:
                result = {k: f.get_tensor(k) for k in f.keys()}
        return result

    # Sliding window: max_workers loading + 1 prefetched.
    buffer_size = max_workers + 1

    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        file_iter = iter(sorted_files)
        pending: collections.deque = collections.deque()

        # Seed the buffer.
        for st_file in itertools.islice(file_iter, buffer_size):
            pending.append(executor.submit(_load_file, st_file))

        with tqdm(
            total=len(hf_weights_files),
            desc="Multi-thread loading shards",
            disable=not enable_tqdm,
            bar_format=BAR_FORMAT,
            position=tqdm._get_free_pos(),
        ) as pbar:
            while pending:
                future = pending.popleft()
                state_dict = future.result()
                del future  # let GC reclaim the Future's internal result

                # Replenish: submit the next file to keep the buffer full.
                next_file = next(file_iter, None)
                if next_file is not None:
                    pending.append(executor.submit(_load_file, next_file))

                for name in sorted(state_dict.keys()):
                    yield name, state_dict[name]
```
**EN:** This callable implements `buffered_multi_thread_safetensors_weights_iterator`. It takes `hf_weights_files`, `max_workers`, `disable_mmap`, `prefetch` and mainly converts data into another representation. The docstring states: "Multi-threaded safetensor loader with bounded memory via a sliding window." This chunk is part 1 of 2 for the same logical block. In this range it coordinates distributed communication; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `buffered_multi_thread_safetensors_weights_iterator`。它接收 `hf_weights_files`, `max_workers`, `disable_mmap`, `prefetch`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会协调分布式通信；管理模型权重或检查点。

### Lines 1094-1095: Function buffered_multi_thread_safetensors_weights_iterator (part 2/2)
```python
                del state_dict
                pbar.update(1)
```
**EN:** This callable implements `buffered_multi_thread_safetensors_weights_iterator`. It takes `hf_weights_files`, `max_workers`, `disable_mmap`, `prefetch` and mainly converts data into another representation. The docstring states: "Multi-threaded safetensor loader with bounded memory via a sliding window." This chunk is part 2 of 2 for the same logical block.
**CN:** 这一可调用对象实现了 `buffered_multi_thread_safetensors_weights_iterator`。它接收 `hf_weights_files`, `max_workers`, `disable_mmap`, `prefetch`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 2/2 部分。

### Lines 1098-1115: Function _load_pt_file
```python
def _load_pt_file(bin_file: str) -> dict:
    """Load a PyTorch checkpoint file, handling legacy tar format.

    PyTorch 2.6 changed the default of weights_only from False to True.
    Legacy tar format files cannot be loaded with weights_only=True.
    This function tries weights_only=True first, then falls back to False
    for legacy tar format files from trusted sources (HuggingFace Hub).
    """
    try:
        return torch.load(bin_file, map_location="cpu", weights_only=True)
    except RuntimeError as e:
        if "legacy .tar format" in str(e):
            logger.warning(
                "Loading %s with weights_only=False (legacy tar format)",
                os.path.basename(bin_file),
            )
            return torch.load(bin_file, map_location="cpu", weights_only=False)
        raise
```
**EN:** This callable implements `_load_pt_file`. It takes `bin_file` and mainly loads external data or weights. The docstring states: "Load a PyTorch checkpoint file, handling legacy tar format." In this range it sets up imports and shared symbols; emits logs for diagnostics; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `_load_pt_file`。它接收 `bin_file`，主要用于加载外部数据或权重。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；管理模型权重或检查点。

### Lines 1118-1134: Function pt_weights_iterator
```python
def pt_weights_iterator(
    hf_weights_files: List[str],
) -> Generator[Tuple[str, torch.Tensor], None, None]:
    """Iterate over the weights in the model bin/pt files."""
    enable_tqdm = (
        not torch.distributed.is_initialized() or torch.distributed.get_rank() == 0
    )
    for bin_file in tqdm(
        hf_weights_files,
        desc="Loading pt checkpoint shards",
        disable=not enable_tqdm,
        bar_format=BAR_FORMAT,
        position=tqdm._get_free_pos(),
    ):
        state = _load_pt_file(bin_file)
        yield from state.items()
        del state
```
**EN:** This callable implements `pt_weights_iterator`. It takes `hf_weights_files` and mainly converts data into another representation. The docstring states: "Iterate over the weights in the model bin/pt files." In this range it sets up imports and shared symbols; coordinates distributed communication; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `pt_weights_iterator`。它接收 `hf_weights_files`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信；管理模型权重或检查点。

### Lines 1137-1164: Function multi_thread_pt_weights_iterator
```python
def multi_thread_pt_weights_iterator(
    hf_weights_files: List[str],
    max_workers: int,
) -> Generator[Tuple[str, torch.Tensor], None, None]:
    """Multi-Thread iterate over the weights in the model bin/pt files."""
    enable_tqdm = (
        not torch.distributed.is_initialized() or torch.distributed.get_rank() == 0
    )

    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = [
            executor.submit(_load_pt_file, bin_file) for bin_file in hf_weights_files
        ]

        if enable_tqdm:
            futures_iter = tqdm(
                concurrent.futures.as_completed(futures),
                total=len(hf_weights_files),
                desc="Multi-thread loading pt checkpoint shards",
                disable=not enable_tqdm,
                bar_format=BAR_FORMAT,
            )
        else:
            futures_iter = concurrent.futures.as_completed(futures)

        for future in futures_iter:
            state = future.result()
            yield from state.items()
```
**EN:** This callable implements `multi_thread_pt_weights_iterator`. It takes `hf_weights_files`, `max_workers` and mainly converts data into another representation. The docstring states: "Multi-Thread iterate over the weights in the model bin/pt files." In this range it sets up imports and shared symbols; coordinates distributed communication; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `multi_thread_pt_weights_iterator`。它接收 `hf_weights_files`, `max_workers`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信；管理模型权重或检查点。

### Lines 1167-1176: Function get_gguf_extra_tensor_names
```python
def get_gguf_extra_tensor_names(
    gguf_file: str, gguf_to_hf_name_map: Dict[str, str]
) -> List[str]:
    import gguf

    reader = gguf.GGUFReader(gguf_file)
    expected_gguf_keys = set(gguf_to_hf_name_map.keys())
    exact_gguf_keys = set([tensor.name for tensor in reader.tensors])
    extra_keys = expected_gguf_keys - exact_gguf_keys
    return [gguf_to_hf_name_map[key] for key in extra_keys]
```
**EN:** This callable implements `get_gguf_extra_tensor_names`. It takes `gguf_file`, `gguf_to_hf_name_map` and mainly retrieves a value or derived view. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `get_gguf_extra_tensor_names`。它接收 `gguf_file`, `gguf_to_hf_name_map`，主要用于获取某个值或派生视图。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 1179-1238: Function gguf_quant_weights_iterator (part 1/2)
```python
def gguf_quant_weights_iterator(
    gguf_file: str, gguf_to_hf_name_map: Dict[str, str]
) -> Generator[Tuple[str, torch.Tensor], None, None]:
    """
    Iterate over the quant weights in the model gguf files and convert
    them to torch tensors
    """

    import gguf

    reader = gguf.GGUFReader(gguf_file)

    # MoE expert weight name patterns
    MOE_WEIGHT_PATTERNS = {
        "ffn_gate_exps": "gate_proj",  # gate projection
        "ffn_up_exps": "up_proj",  # up projection
        "ffn_down_exps": "down_proj",  # down projection
    }

    # First pass: yield weight types
    for tensor in reader.tensors:
        weight_type = tensor.tensor_type
        tensor_name = tensor.name

        # Check if this is a MoE expert weight (packed format)
        is_moe_weight = any(
            pattern in tensor_name for pattern in MOE_WEIGHT_PATTERNS.keys()
        )

        if is_moe_weight:
            # MoE weights need special handling - extract layer_id and weight type
            # Format: blk.{layer_id}.ffn_gate_exps.weight
            import re

            match = re.match(r"blk\.(\d+)\.(ffn_\w+_exps)\.weight", tensor_name)
            if match:
                layer_id = int(match.group(1))
                weight_pattern = match.group(2)
                hf_weight_name = MOE_WEIGHT_PATTERNS.get(weight_pattern)

                if hf_weight_name and weight_type.name != "F32":
                    # Yield weight type for each expert
                    weight = tensor.data
                    num_experts = weight.shape[0]
                    for expert_id in range(num_experts):
                        hf_name = f"model.layers.{layer_id}.mlp.experts.{expert_id}.{hf_weight_name}.qweight_type"
                        yield hf_name, torch.tensor(weight_type)
        elif tensor_name in gguf_to_hf_name_map:
            # Normal weight handling
            name = gguf_to_hf_name_map[tensor_name]

            if weight_type.name != "F32":
                weight_type_name = name.replace("weight", "qweight_type")
                yield weight_type_name, torch.tensor(weight_type)

    # Second pass: yield actual weights
    for tensor in reader.tensors:
        weight = tensor.data
        weight_type = tensor.tensor_type
        tensor_name = tensor.name
```
**EN:** This callable implements `gguf_quant_weights_iterator`. It takes `gguf_file`, `gguf_to_hf_name_map` and mainly converts data into another representation. The docstring states: "Iterate over the quant weights in the model gguf files and convert them to torch tensors" This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `gguf_quant_weights_iterator`。它接收 `gguf_file`, `gguf_to_hf_name_map`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；管理模型权重或检查点。

### Lines 1239-1274: Function gguf_quant_weights_iterator (part 2/2)
```python

        # Check if this is a MoE expert weight (packed format)
        is_moe_weight = any(
            pattern in tensor_name for pattern in MOE_WEIGHT_PATTERNS.keys()
        )

        if is_moe_weight:
            # MoE weights: split packed format into individual expert weights
            import re

            match = re.match(r"blk\.(\d+)\.(ffn_\w+_exps)\.weight", tensor_name)
            if match:
                layer_id = int(match.group(1))
                weight_pattern = match.group(2)
                hf_weight_name = MOE_WEIGHT_PATTERNS.get(weight_pattern)

                if hf_weight_name:
                    # Packed format: [num_experts, ...]
                    num_experts = weight.shape[0]
                    for expert_id in range(num_experts):
                        expert_weight = weight[expert_id]

                        if weight_type.name != "F32":
                            hf_name = f"model.layers.{layer_id}.mlp.experts.{expert_id}.{hf_weight_name}.qweight"
                        else:
                            hf_name = f"model.layers.{layer_id}.mlp.experts.{expert_id}.{hf_weight_name}.weight"

                        yield hf_name, torch.tensor(expert_weight)
        elif tensor_name in gguf_to_hf_name_map:
            # Normal weight handling
            name = gguf_to_hf_name_map[tensor_name]

            if weight_type.name != "F32":
                name = name.replace("weight", "qweight")
            param = torch.tensor(weight)
            yield name, param
```
**EN:** This callable implements `gguf_quant_weights_iterator`. It takes `gguf_file`, `gguf_to_hf_name_map` and mainly converts data into another representation. The docstring states: "Iterate over the quant weights in the model gguf files and convert them to torch tensors" This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `gguf_quant_weights_iterator`。它接收 `gguf_file`, `gguf_to_hf_name_map`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；管理模型权重或检查点。

### Lines 1277-1289: Function convert_pyslice_to_tensor
```python
def convert_pyslice_to_tensor(x: Any) -> torch.Tensor:
    """convert PySafeSlice object from safetensors to torch.Tensor

    PySafeSlice object supports indexing, which is done before loading the
    actual tensor and can reduce the amount of memory being read into the
    memory. However, it does not support more advanced functionalities
    like `.view()` or `.t()`. Therefore, if we need to modify the loaded
    tensor with these more complicated operators, we need to convert to
    tensor first.
    """
    if not isinstance(x, torch.Tensor):
        x = x[:]
    return x
```
**EN:** This callable implements `convert_pyslice_to_tensor`. It takes `x` and mainly converts data into another representation. The docstring states: "convert PySafeSlice object from safetensors to torch.Tensor PySafeSlice object supports indexing, which is done before loading the actual tensor and can reduce the amount of memory being read into the memory." In this range it sets up imports and shared symbols; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `convert_pyslice_to_tensor`。它接收 `x`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；管理模型权重或检查点。

### Lines 1292-1310: Function default_weight_loader
```python
def default_weight_loader(param: torch.Tensor, loaded_weight: torch.Tensor) -> None:
    """Default weight loader."""
    try:
        if param.numel() == 1 and loaded_weight.numel() == 1:
            # Sometimes scalar values aren't considered tensors with shapes
            # so if both param and loaded_weight are a scalar,
            # "broadcast" instead of copy
            param.data.fill_(loaded_weight.item())
        else:
            assert param.size() == loaded_weight.size(), (
                f"Attempted to load weight ({loaded_weight.size()}) "
                f"into parameter ({param.size()})"
            )

            param.data.copy_(loaded_weight)
    except Exception:
        # NOTE: This exception is added for the purpose of setting breakpoint to
        # debug weight loading issues.
        raise
```
**EN:** This callable implements `default_weight_loader`. It takes `param`, `loaded_weight` and mainly loads external data or weights. The docstring states: "Default weight loader." In this range it performs defensive checks on invalid state; coordinates distributed communication; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `default_weight_loader`。它接收 `param`, `loaded_weight`，主要用于加载外部数据或权重。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信；管理模型权重或检查点。

### Lines 1313-1325: Function row_parallel_weight_loader
```python
def row_parallel_weight_loader(
    param: torch.Tensor, loaded_weight: torch.Tensor
) -> None:
    """Load weights that are row-parallelized."""
    tp_rank = get_tensor_model_parallel_rank()
    shard_dim = 0 if param.dim() != 1 else None

    if shard_dim is not None:
        shard_size = param.data.shape[shard_dim]
        start_idx = tp_rank * shard_size
        loaded_weight = loaded_weight.narrow(shard_dim, start_idx, shard_size)

    return default_weight_loader(param, loaded_weight)
```
**EN:** This callable implements `row_parallel_weight_loader`. It takes `param`, `loaded_weight` and mainly loads external data or weights. The docstring states: "Load weights that are row-parallelized." In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `row_parallel_weight_loader`。它接收 `param`, `loaded_weight`，主要用于加载外部数据或权重。 在这一范围内，它会管理模型权重或检查点。

### Lines 1326-1330: Module-level constants and helpers
```python


LoaderFunction = Callable[[torch.Tensor, torch.Tensor], torch.Tensor]


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 1331-1363: Function sharded_weight_loader
```python
def sharded_weight_loader(shard_axis: int) -> LoaderFunction:
    """Create a weight loader that shards the weights along the given axis"""

    def loader(param: torch.Tensor, loaded_weight: torch.Tensor) -> None:
        tp_rank = get_attention_tp_rank()

        shard_size = param.data.shape[shard_axis]
        start_idx = tp_rank * shard_size

        if (
            is_cpu()
            and (
                loaded_weight.size(0) % get_tensor_model_parallel_world_size() != 0
                or loaded_weight.size(0)
                < get_tensor_model_parallel_world_size() * shard_size
            )
            and loaded_weight.dim() == 1
        ):
            param_data = param.data  # view copy on param for uneven padding
            param_data, loaded_weight = narrow_padded_param_and_loaded_weight(
                param_data,
                loaded_weight,
                0,  # param_data_start
                start_idx,
                shard_axis,
                shard_size,
            )
            return default_weight_loader(param_data, loaded_weight)
        else:
            loaded_weight = loaded_weight.narrow(shard_axis, start_idx, shard_size)
            return default_weight_loader(param, loaded_weight)

    return loader
```
**EN:** This callable implements `sharded_weight_loader`. It takes `shard_axis` and mainly loads external data or weights. The docstring states: "Create a weight loader that shards the weights along the given axis" In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `sharded_weight_loader`。它接收 `shard_axis`，主要用于加载外部数据或权重。 在这一范围内，它会管理模型权重或检查点。

### Lines 1366-1376: Function composed_weight_loader
```python
def composed_weight_loader(
    loader: LoaderFunction, fn: Callable[[torch.Tensor], torch.Tensor]
) -> LoaderFunction:
    """Create a weight loader that post-processes the weights after loading"""

    def composed_loader(param: torch.Tensor, loaded_weight: torch.Tensor) -> None:
        loader(param, loaded_weight)
        param.data.copy_(fn(param))
        return

    return composed_loader
```
**EN:** This callable implements `composed_weight_loader`. It takes `loader`, `fn` and mainly loads external data or weights. The docstring states: "Create a weight loader that post-processes the weights after loading" In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `composed_weight_loader`。它接收 `loader`, `fn`，主要用于加载外部数据或权重。 在这一范围内，它会管理模型权重或检查点。

### Lines 1379-1413: Function runai_safetensors_weights_iterator
```python
def runai_safetensors_weights_iterator(
    hf_weights_files: List[str], is_distributed: bool = False, device: str = "cpu"
) -> Generator[Tuple[str, torch.Tensor], None, None]:
    """Iterate over the weights in the model safetensor files."""
    from runai_model_streamer import SafetensorsStreamer

    enable_tqdm = (
        not torch.distributed.is_initialized() or torch.distributed.get_rank() == 0
    )
    device = device if is_distributed and is_cuda_alike() else "cpu"

    with SafetensorsStreamer() as streamer:

        streamer.stream_files(
            hf_weights_files,
            device=device,
            is_distributed=is_distributed,
        )
        total_tensors = sum(
            len(tensors_meta)
            for tensors_meta in streamer.files_to_tensors_metadata.values()
        )

        tensor_iter = tqdm(
            streamer.get_tensors(),
            total=total_tensors,
            desc="Loading safetensors using Runai Model Streamer",
            bar_format=BAR_FORMAT,
            disable=not enable_tqdm,
            mininterval=2,
        )

        for name, tensor in tensor_iter:
            setattr(tensor, RUNAI_STREAMER_TENSOR_ATTR, True)
            yield name, tensor
```
**EN:** This callable implements `runai_safetensors_weights_iterator`. It takes `hf_weights_files`, `is_distributed`, `device` and mainly converts data into another representation. The docstring states: "Iterate over the weights in the model safetensor files." In this range it sets up imports and shared symbols; coordinates distributed communication; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `runai_safetensors_weights_iterator`。它接收 `hf_weights_files`, `is_distributed`, `device`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信；管理模型权重或检查点。

### Lines 1416-1437: Function set_runai_streamer_env
```python
def set_runai_streamer_env(load_config: LoadConfig):
    if load_config.model_loader_extra_config:
        extra_config = load_config.model_loader_extra_config

        if "concurrency" in extra_config and isinstance(
            extra_config.get("concurrency"), int
        ):
            os.environ["RUNAI_STREAMER_CONCURRENCY"] = str(
                extra_config.get("concurrency")
            )

        if "memory_limit" in extra_config and isinstance(
            extra_config.get("memory_limit"), int
        ):
            os.environ["RUNAI_STREAMER_MEMORY_LIMIT"] = str(
                extra_config.get("memory_limit")
            )

    runai_streamer_s3_endpoint = os.getenv("RUNAI_STREAMER_S3_ENDPOINT")
    aws_endpoint_url = os.getenv("AWS_ENDPOINT_URL")
    if runai_streamer_s3_endpoint is None and aws_endpoint_url is not None:
        os.environ["RUNAI_STREAMER_S3_ENDPOINT"] = aws_endpoint_url
```
**EN:** This callable implements `set_runai_streamer_env`. It takes `load_config` and mainly applies configuration to mutable state. In this range it reads environment-driven configuration; talks to external storage or service backends.
**CN:** 这一可调用对象实现了 `set_runai_streamer_env`。它接收 `load_config`，主要用于将配置写入可变状态。 在这一范围内，它会读取环境变量驱动的配置；与外部存储或服务后端交互。

### Lines 1440-1469: Function initialize_dummy_weights
```python
def initialize_dummy_weights(
    model: torch.nn.Module,
    low: float = -1e-3,
    high: float = 1e-3,
    seed: int = 1234,
) -> None:
    """Initialize model weights with random values.

    The model weights must be randomly initialized for accurate performance
    measurements. Additionally, the model weights should not cause NaNs in the
    forward pass. We empirically found that initializing the weights with
    values between -1e-3 and 1e-3 works well for most models.

    We use per-parameter random seed, so that dummy weights are consistent,
    even if the model is partitioned across multiple devices. When the seed
    is fixed, the random values generated by this function only depends on
    the parameter's number of elements and its data type.
    """
    for param in model.state_dict().values():
        if torch.is_floating_point(param):
            generator = torch.Generator(device=param.data.device)
            generator.manual_seed(seed)
            if torch.finfo(param.data.dtype).bits < 16:
                # uniform_ doesn't support < 16-bit datatypes (FP8)
                dtype = param.data.dtype
                tmp_param = param.data.to(torch.float16)
                tmp_param = tmp_param.uniform_(low, high, generator=generator).to(dtype)
                param.data.copy_(tmp_param)
            else:
                param.uniform_(low, high, generator=generator)
```
**EN:** This callable implements `initialize_dummy_weights`. It takes `model`, `low`, `high`, `seed` and mainly implements initialize dummy weights. The docstring states: "Initialize model weights with random values." In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `initialize_dummy_weights`。它接收 `model`, `low`, `high`, `seed`，主要用于实现 initialize dummy weights 相关逻辑。 在这一范围内，它会管理模型权重或检查点。

### Lines 1472-1531: Function maybe_remap_kv_scale_name (part 1/2)
```python
def maybe_remap_kv_scale_name(name: str, params_dict: dict) -> Optional[str]:
    """Remap the name of FP8 k/v_scale parameters.

    This function handles the remapping of FP8 k/v_scale parameter names.
    It detects if the given name ends with a suffix and attempts to remap
    it to the expected name format in the model. If the remapped name is not
    found in the params_dict, a warning is printed and None is returned.

    Args:
        name (str): The original loaded checkpoint parameter name.
        params_dict (dict): Dictionary containing the model's named parameters.

    Returns:
        str: The remapped parameter name if successful, or the original name
             if no remapping is needed.
        None: If the remapped name is not found in params_dict.
    """
    if name.endswith(".kv_scale"):
        print_warning_once(
            "DEPRECATED. Found kv_scale in the checkpoint. "
            "This format is deprecated in favor of separate k_scale and "
            "v_scale tensors and will be removed in a future release. "
            "Functionally, we will remap kv_scale to k_scale and duplicate "
            "k_scale to v_scale"
        )
        # NOTE: we remap the deprecated kv_scale to k_scale
        remapped_name = name.replace(".kv_scale", ".attn.k_scale")
        if remapped_name not in params_dict:
            print_warning_once(
                f"Found kv_scale in the checkpoint (e.g. {name}), "
                "but not found the expected name in the model "
                f"(e.g. {remapped_name}). kv_scale is "
                "not loaded."
            )
            return None
        return remapped_name

    possible_scale_names = [".k_scale", ".v_scale"]
    # Patterns where modelopt stores scales under k_proj/v_proj
    # but the model expects them under attn (RadixAttention)
    modelopt_attn_prefixes = [".self_attn.", ".mixer."]
    for scale_name in possible_scale_names:
        if name.endswith(scale_name):
            # Check if this is a modelopt-style scale under k_proj/v_proj
            matched_prefix = None
            for attn_prefix in modelopt_attn_prefixes:
                if f"{attn_prefix}{scale_name[1]}_proj{scale_name}" in name:
                    matched_prefix = attn_prefix
                    break

            if matched_prefix is not None:
                remapped_name = name.replace(
                    f"{matched_prefix}{scale_name[1]}_proj{scale_name}",
                    f"{matched_prefix}attn{scale_name}",
                )
            else:
                remapped_name = name.replace(scale_name, f".attn{scale_name}")
            if remapped_name not in params_dict:
                print_warning_once(
                    f"Found {scale_name} in the checkpoint (e.g. {name}), "
```
**EN:** This callable implements `maybe_remap_kv_scale_name`. It takes `name`, `params_dict` and mainly implements maybe remap kv scale name. The docstring states: "Remap the name of FP8 k/v_scale parameters." This chunk is part 1 of 2 for the same logical block. In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `maybe_remap_kv_scale_name`。它接收 `name`, `params_dict`，主要用于实现 maybe remap kv scale name 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会管理模型权重或检查点。

### Lines 1532-1550: Function maybe_remap_kv_scale_name (part 2/2)
```python
                    "but not found the expected name in the model "
                    f"(e.g. {remapped_name}). {scale_name} is "
                    "not loaded."
                )
                return None
            return remapped_name

    quark_scale_names = {
        ".q_proj.output_scale": ".attn.q_scale",
        ".k_proj.output_scale": ".attn.k_scale",
        ".v_proj.output_scale": ".attn.v_scale",
        "self_attn.prob_output_scale": ".attn.prob_scale",
    }
    for quark_scale_name, sglang_scale_name in quark_scale_names.items():
        if name.endswith(quark_scale_name):
            return name.replace(quark_scale_name, sglang_scale_name)

    # If there were no matches, return the untouched param name
    return name
```
**EN:** This callable implements `maybe_remap_kv_scale_name`. It takes `name`, `params_dict` and mainly implements maybe remap kv scale name. The docstring states: "Remap the name of FP8 k/v_scale parameters." This chunk is part 2 of 2 for the same logical block.
**CN:** 这一可调用对象实现了 `maybe_remap_kv_scale_name`。它接收 `name`, `params_dict`，主要用于实现 maybe remap kv scale name 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。

### Lines 1551-1553: Module imports, constants, and setup
```python


# Adapted from https://github.com/vllm-project/vllm/blob/68ad4e3a8d8a66fb2a43be57471ee13a8bec4ec0/vllm/model_executor/layers/quantization/schema.py
```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 1554-1561: Class KVCacheQuantSchema
```python
class KVCacheQuantSchema(BaseModel):
    dtype: str
    # Each key is a TP rank. Each value is a dictionary mapping a TP rank's
    # layer indices to their per-tensor KV cache scaling factor.
    # TODO: Consider pulling this and its validation methods out into its
    # own schema class (tricky as its members are variable)
    scaling_factor: Dict[int, Dict[int, float]]

```
**EN:** This range introduces `KVCacheQuantSchema` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `KVCacheQuantSchema`，并定义其后续方法依赖的结构或元数据。

### Lines 1562-1568: Method KVCacheQuantSchema.check_is_fp8
```python
    @model_validator(mode="after")
    def check_is_fp8(self) -> "KVCacheQuantSchema":
        assert self.dtype == "float8_e4m3fn", (
            "Loaded scaling factors intended for KV cache dtype = "
            f"{self.dtype} rather than float8_e4m3fn!"
        )
        return self
```
**EN:** This callable implements `KVCacheQuantSchema.check_is_fp8` and mainly checks preconditions and compatibility. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `KVCacheQuantSchema.check_is_fp8`，主要用于检查前置条件与兼容性。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1570-1590: Method KVCacheQuantSchema.check_tp_ranks
```python
    @model_validator(mode="after")
    def check_tp_ranks(self, info: ValidationInfo) -> "KVCacheQuantSchema":
        context = info.context
        if context:
            tp_size = context["tp_size"]
            num_hidden_layers = context["num_hidden_layers"]
            assert len(self.scaling_factor) == tp_size, (
                f"Loaded dictionary has TP size {len(self.scaling_factor)} "
                f"but LLM engine is currently running with TP size {tp_size}."
            )
            for tp_rank, layer_maps in self.scaling_factor.items():
                assert len(layer_maps) == num_hidden_layers, (
                    f"KV cache scales map for TP rank {tp_rank} is malformed. "
                    f"Expected {num_hidden_layers} layers, got "
                    f"{len(layer_maps)}."
                )
            for i in range(tp_size):
                assert (
                    i in self.scaling_factor
                ), f"KV cache scales map for TP rank {i} not found."
        return self
```
**EN:** This callable implements `KVCacheQuantSchema.check_tp_ranks`. It takes `info` and mainly checks preconditions and compatibility. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `KVCacheQuantSchema.check_tp_ranks`。它接收 `info`，主要用于检查前置条件与兼容性。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1592-1604: Method KVCacheQuantSchema.check_current_rank
```python
    @model_validator(mode="after")
    def check_current_rank(self, info: ValidationInfo) -> "KVCacheQuantSchema":
        context = info.context
        if context:
            tp_rank = context["tp_rank"]
            num_hidden_layers = context["num_hidden_layers"]
            layer_scales_map = self.scaling_factor[tp_rank]
            for i in range(num_hidden_layers):
                assert i in layer_scales_map, (
                    f"Could not find KV cache scales for layer {i} in "
                    f"TP rank {tp_rank}."
                )
        return self
```
**EN:** This callable implements `KVCacheQuantSchema.check_current_rank`. It takes `info` and mainly checks preconditions and compatibility. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `KVCacheQuantSchema.check_current_rank`。它接收 `info`，主要用于检查前置条件与兼容性。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1607-1613: Class QuantParamSchema
```python
class QuantParamSchema(BaseModel):
    # TODO: Generalize and extend with more fields
    # (e.g. weights/activations params) once functionality is enabled
    model_config = ConfigDict(protected_namespaces=())
    model_type: Optional[str]
    kv_cache: KVCacheQuantSchema

```
**EN:** This range introduces `QuantParamSchema` and defines the structure or metadata that its methods rely on. In this range it manages model weights or checkpoints.
**CN:** 这一段引入 `QuantParamSchema`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会管理模型权重或检查点。

### Lines 1614-1625: Method QuantParamSchema.check_model_type
```python
    @model_validator(mode="after")
    def check_model_type(self, info: ValidationInfo) -> "QuantParamSchema":
        context = info.context
        if context:
            model_type = context.get("model_type", None)
            if model_type is not None:
                assert model_type == self.model_type, (
                    f"Model type is {model_type} but loaded "
                    f"scaling factors belonging to different "
                    f"model type {self.model_type}!"
                )
        return self
```
**EN:** This callable implements `QuantParamSchema.check_model_type`. It takes `info` and mainly checks preconditions and compatibility. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `QuantParamSchema.check_model_type`。它接收 `info`，主要用于检查前置条件与兼容性。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1628-1668: Function kv_cache_scales_loader
```python
def kv_cache_scales_loader(
    filename: str,
    tp_rank: int,
    tp_size: int,
    num_hidden_layers: int,
    model_type: Optional[str],
) -> Iterable[Tuple[int, float]]:
    """
    A simple utility to read in KV cache scaling factors that have been
    previously serialized to disk. Used by the model to populate the appropriate
    KV cache scaling factors. The serialization should represent a dictionary
    whose keys are the TP ranks and values are another dictionary mapping layers
    to their KV cache scaling factors.
    """
    try:
        with open(filename) as f:
            context = {
                "model_type": model_type,
                "num_hidden_layers": num_hidden_layers,
                "tp_rank": tp_rank,
                "tp_size": tp_size,
            }
            schema_dct = json.load(f)
            schema = QuantParamSchema.model_validate(schema_dct, context=context)
            layer_scales_map = schema.kv_cache.scaling_factor[tp_rank]
            return layer_scales_map.items()
    except FileNotFoundError:
        logger.error("File or directory '%s' not found.", filename)
    except json.JSONDecodeError:
        logger.error("Error decoding JSON in file '%s'.", filename)
    except Exception:
        logger.error("An error occurred while reading '%s'.", filename)
    # This section is reached if and only if any of the excepts are hit
    # Return an empty iterable (list) => no KV cache scales are loaded
    # which ultimately defaults to 1.0 scales
    logger.warning(
        "Defaulting to KV cache scaling factors = 1.0 for all "
        "layers in TP rank %d as an error occurred during loading.",
        tp_rank,
    )
    return []
```
**EN:** This callable implements `kv_cache_scales_loader`. It takes `filename`, `tp_rank`, `tp_size`, `num_hidden_layers` and mainly loads external data or weights. The docstring states: "A simple utility to read in KV cache scaling factors that have been previously serialized to disk." In this range it emits logs for diagnostics; serializes or parses JSON payloads.
**CN:** 这一可调用对象实现了 `kv_cache_scales_loader`。它接收 `filename`, `tp_rank`, `tp_size`, `num_hidden_layers`，主要用于加载外部数据或权重。 在这一范围内，它会输出日志以便诊断；序列化或解析 JSON 数据。

### Lines 1671-1675: Function get_actual_shard_size
```python
def get_actual_shard_size(shard_size, weight_start, weight_end):
    if weight_end < weight_start:
        return 0

    return min(shard_size, weight_end - weight_start)
```
**EN:** This callable implements `get_actual_shard_size`. It takes `shard_size`, `weight_start`, `weight_end` and mainly retrieves a value or derived view. In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `get_actual_shard_size`。它接收 `shard_size`, `weight_start`, `weight_end`，主要用于获取某个值或派生视图。 在这一范围内，它会管理模型权重或检查点。

### Lines 1678-1685: Function reset_param_data_if_needed
```python
def reset_param_data_if_needed(param_data, dim, start, length):
    if length == 0:
        return

    assert length > 0, f"Length should be positive, but got {length}"

    param_data.narrow(dim, start, length).zero_()
    return
```
**EN:** This callable implements `reset_param_data_if_needed`. It takes `param_data`, `dim`, `start`, `length` and mainly applies configuration to mutable state. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `reset_param_data_if_needed`。它接收 `param_data`, `dim`, `start`, `length`，主要用于将配置写入可变状态。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1688-1722: Function narrow_padded_param_and_loaded_weight
```python
def narrow_padded_param_and_loaded_weight(
    param_data,
    loaded_weight,
    param_data_start,
    weight_start,
    dim,
    shard_size,
    narrow_weight=True,
):
    actual_shard_size = get_actual_shard_size(
        shard_size, weight_start, loaded_weight.size(dim)
    )

    if narrow_weight:
        if actual_shard_size > 0:
            loaded_weight = loaded_weight.narrow(dim, weight_start, actual_shard_size)
        else:
            # No real data to load; create a dummy tensor filled with zeros
            loaded_weight = torch.zeros_like(
                param_data.narrow(dim, param_data_start, actual_shard_size)
            )

    # [Note] Reset padded weights to zero.
    # If the actual shard size is less than the shard size, we need to reset
    # the padded param_data to zero and then copy the loaded_weight into it.
    reset_param_data_if_needed(
        param_data,
        dim,
        param_data_start + actual_shard_size,
        shard_size - actual_shard_size,
    )

    param_data = param_data.narrow(dim, param_data_start, actual_shard_size)

    return param_data, loaded_weight
```
**EN:** This callable implements `narrow_padded_param_and_loaded_weight`. It takes `param_data`, `loaded_weight`, `param_data_start`, `weight_start` and mainly adds configuration entries or arguments. In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `narrow_padded_param_and_loaded_weight`。它接收 `param_data`, `loaded_weight`, `param_data_start`, `weight_start`，主要用于添加配置项或参数。 在这一范围内，它会管理模型权重或检查点。

### Lines 1725-1752: Function pad_loaded_weight
```python
def pad_loaded_weight(loaded_weight, output_dim, output_sizes):
    # This function is for padding zeros when loaded_weight is less than output_sizes.
    # Most cases, sum(output_sizes) = loaded_weight.size(output_dim),
    # while in some TP cases like TP6, output_sizes will be padded, thus loaded_weight needs padding.
    total_output_size = sum(output_sizes)
    raw_output_size = loaded_weight.size(output_dim)
    if total_output_size > raw_output_size:
        loaded_weight_pad = []
        weight_split_size = [
            int(output_size / total_output_size * raw_output_size)
            for output_size in output_sizes
        ]
        assert (
            sum(weight_split_size) == raw_output_size
        ), f"Padding the loaded weight failed due to sizes are not divisible cleanly from {output_sizes} to {raw_output_size}"

        split_weight = loaded_weight.split_with_sizes(weight_split_size, dim=output_dim)
        for i, output_size in enumerate(output_sizes):
            pad_size = output_size - weight_split_size[i]
            target_pad_shape = list(loaded_weight.size())
            target_pad_shape[output_dim] = pad_size
            pad_tensor = torch.zeros(target_pad_shape).to(loaded_weight.dtype)
            loaded_weight_pad.append(
                torch.cat([split_weight[i], pad_tensor], dim=output_dim)
            )
        return torch.cat(loaded_weight_pad, dim=output_dim)
    else:
        return loaded_weight
```
**EN:** This callable implements `pad_loaded_weight`. It takes `loaded_weight`, `output_dim`, `output_sizes` and mainly loads external data or weights. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `pad_loaded_weight`。它接收 `loaded_weight`, `output_dim`, `output_sizes`，主要用于加载外部数据或权重。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；管理模型权重或检查点。

## Key Concepts / 关键概念
- `RUNAI_STREAMER_TENSOR_ATTR`: module constant or capability flag / 模块常量或能力标记
- `_ROUTED_EXPERT_KEY_RE`: module constant or capability flag / 模块常量或能力标记
- `probe_routed_expert_weight_dtype`: implements probe routed expert weight dtype / 实现 probe routed expert weight dtype 相关逻辑
- `_PREFETCH_BLOCK_SIZE`: module constant or capability flag / 模块常量或能力标记
- `_get_prefetch_block_size`: retrieves a value or derived view / 获取某个值或派生视图
- `get_lock`: retrieves a value or derived view / 获取某个值或派生视图
- `_shared_pointers`: implements shared pointers / 实现 shared pointers 相关逻辑
- `convert_bin_to_safetensor_file`: converts data into another representation / 将数据转换为另一种表示
- `replace_prefix`: implements replace prefix / 实现 replace prefix 相关逻辑
- `replace_substrings`: implements replace substrings / 实现 replace substrings 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections`, `concurrent.futures`, `fnmatch`, `glob`, `hashlib`, `itertools`, `json`, `logging`, `os`, `re`, `struct`, `tempfile` + 5 more
- **Third-party / 第三方**: `filelock`, `huggingface_hub.constants`, `numpy`, `safetensors.torch`, `torch`, `huggingface_hub`, `pydantic`, `tqdm.auto`, `fastsafetensors`, `gguf`, `runai_model_streamer`
- **Internal modules / 内部模块**: `sglang.srt.configs.load_config`, `sglang.srt.configs.model_config`, `sglang.srt.distributed`, `sglang.srt.layers.dp_attention`, `sglang.srt.layers.quantization`, `sglang.srt.layers.quantization.fp8`, `sglang.srt.layers.quantization.modelopt_quant`, `sglang.srt.model_loader.ci_weight_validation`, `sglang.srt.utils`, `sglang.srt.utils.common`, `sglang.utils`, `sglang.srt.environ`
