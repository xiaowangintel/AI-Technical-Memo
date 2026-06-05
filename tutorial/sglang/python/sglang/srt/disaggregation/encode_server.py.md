# encode_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/encode_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file orchestrates the encode-side transfer server, including request admission, transfer setup, and interaction with the memory/cache subsystems. / 该文件负责编排编码侧传输服务端，包括请求接入、传输初始化以及与内存/缓存子系统的交互。

## Line-by-Line Analysis / 逐行分析
### Lines 1-69: Imports and module setup
```python
import asyncio
import concurrent.futures
import ctypes
import logging
import multiprocessing as mp
import os
import pickle
import time
import traceback
from http import HTTPStatus
from typing import Dict, List, Optional, Set, Tuple, Union

import aiohttp
import numpy as np
import torch
import uvicorn
import zmq
import zmq.asyncio
from fastapi import FastAPI
from fastapi.responses import ORJSONResponse, Response
from transformers import AutoProcessor

from sglang.srt.configs.device_config import DeviceConfig
from sglang.srt.configs.load_config import LoadConfig
from sglang.srt.configs.model_config import ModelConfig
from sglang.srt.constants import HEALTH_CHECK_RID_PREFIX
from sglang.srt.disaggregation.encode_receiver import (
    EmbeddingData,
    video_meta_attrs_for,
)
from sglang.srt.distributed.parallel_state import (
    get_default_distributed_backend,
    get_mooncake_transfer_engine,
    get_tp_group,
    init_distributed_environment,
    initialize_model_parallel,
)
from sglang.srt.environ import envs
from sglang.srt.layers.dp_attention import initialize_dp_attention
from sglang.srt.managers.io_struct import ProfileReq, ProfileReqInput, ProfileReqType
from sglang.srt.managers.schedule_batch import Modality, MultimodalDataItem
from sglang.srt.mem_cache.multimodal_cache import EmbeddingResult, MultiModalStaticCache
from sglang.srt.model_loader import get_model
from sglang.srt.multimodal.processors.qwen_vl import preprocess_video
from sglang.srt.server_args import (
    PortArgs,
    ServerArgs,
    set_global_server_args_for_scheduler,
)
from sglang.srt.utils import (
    configure_logger,
    load_audio,
    load_image,
    load_video,
    random_uuid,
)
from sglang.srt.utils.network import (
    NetworkAddress,
    config_socket,
    get_local_ip_auto,
    get_zmq_socket,
)

logger = logging.getLogger(__name__)

HEALTH_CHECK_TIMEOUT = 10

# Minimal 32x32 black PNG for health check dummy encode
MINIMUM_PNG_PICTURE_BASE64 = "iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAACXBIWXMAAA7EAAAOxAGVKw4bAAAAbUlEQVRYhe3VsQ2AMAxE0Y/lIgNQULD/OqyCMgCihCKSG4yRuKuiNH6JLsoEbMACOGBcua9HOR7Y6w6swBwMy0qLTpkeI77qdEBpBFAHBBDAGH8WrwJKI4AAegUCfAKgEgpQDvh3CR3oQCuav58qlAw73kKCSgAAAABJRU5ErkJggg=="
```
**EN:** This block gathers the imports and module-level setup for encode-side transfer server orchestration. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与编码侧传输服务编排相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 70-85: Module-level constants and helper logic
```python

# Minimal WAV: 16kHz mono 16-bit PCM, 160 samples (0.01s) of silence
MINIMUM_WAV_SILENCE_BASE64 = "UklGRmQBAABXQVZFZm10IBAAAAABAAEAgD4AAAB9AAACABAAZGF0YUABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=="

rid_lock = asyncio.Lock()
rid_to_receive_endpoint: Dict[str, List[str]] = dict()
rid_to_receive_count: Dict[str, int] = dict()
rid_to_err_msg: Dict[str, str] = dict()
cond_dict_lock = asyncio.Lock()
rid_to_cond: Dict[str, asyncio.Condition] = {}

use_image_processor_gpu = (
    int(os.getenv("SGLANG_ENCODER_IMAGE_PROCESSOR_USE_GPU", "0")) == 1
)


```
**EN:** This block contains module-level constants, helpers, or documentation for encode-side transfer server orchestration. It prepares shared state that later classes and functions build on. Notable operations include `samples`, `Lock`, `getenv`.
**CN:** 这一段包含与编码侧传输服务编排相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `samples`、`Lock`、`getenv`。

### Lines 86-86: Class `MMError` declaration
```python
class MMError(Exception):
```
**EN:** This block declares the class `MMError` and establishes its responsibility inside encode-side transfer server orchestration. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `MMError`.
**CN:** 这一段声明了类 `MMError`，并说明它在编码侧传输服务编排中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `MMError`。

### Lines 87-90: Method `__init__`
```python
    def __init__(self, message, code=HTTPStatus.INTERNAL_SERVER_ERROR):
        self.message = message
        self.code = code
        super().__init__(self.message)
```
**EN:** This block defines the method `__init__` on `MMError`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `MMError`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 93-93: Class `BadRequestError` declaration
```python
class BadRequestError(MMError):
```
**EN:** This block declares the class `BadRequestError` and establishes its responsibility inside encode-side transfer server orchestration. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `BadRequestError`.
**CN:** 这一段声明了类 `BadRequestError`，并说明它在编码侧传输服务编排中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `BadRequestError`。

### Lines 94-95: Method `__init__`
```python
    def __init__(self, message):
        super().__init__(message, code=HTTPStatus.BAD_REQUEST)
```
**EN:** This block defines the method `__init__` on `BadRequestError`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `BadRequestError`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 98-98: Class `InternalError` declaration
```python
class InternalError(MMError):
```
**EN:** This block declares the class `InternalError` and establishes its responsibility inside encode-side transfer server orchestration. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `InternalError`.
**CN:** 这一段声明了类 `InternalError`，并说明它在编码侧传输服务编排中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `InternalError`。

### Lines 99-100: Method `__init__`
```python
    def __init__(self, message):
        super().__init__(message, code=HTTPStatus.INTERNAL_SERVER_ERROR)
```
**EN:** This block defines the method `__init__` on `InternalError`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `InternalError`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 103-103: Class `TensorWrapper` declaration
```python
class TensorWrapper:
```
**EN:** This block declares the class `TensorWrapper` and establishes its responsibility inside encode-side transfer server orchestration. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `TensorWrapper`.
**CN:** 这一段声明了类 `TensorWrapper`，并说明它在编码侧传输服务编排中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `TensorWrapper`。

### Lines 104-105: Supporting state inside `TensorWrapper`
```python
    """Wrapper to keep tensor alive while exposing buffer for zero-copy."""

```
**EN:** This block adds supporting state or helper logic inside `TensorWrapper`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `TensorWrapper` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 106-116: Method `__init__`
```python
    def __init__(self, tensor):
        # Ensure tensor is on CPU and contiguous
        if tensor.is_cuda:
            tensor = tensor.cpu()
        if not tensor.is_contiguous():
            tensor = tensor.contiguous()

        # Keep tensor reference
        self.tensor = tensor
        self.shape = list(tensor.shape)
        self.dtype = tensor.dtype
```
**EN:** This block defines the method `__init__` on `TensorWrapper`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `__init__`. Notable operations include `cpu`, `is_contiguous`, `contiguous`.
**CN:** 这一段定义了method `__init__`（属于 `TensorWrapper`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `cpu`、`is_contiguous`、`contiguous`。

### Lines 118-123: Method `__buffer__`
```python
    def __buffer__(self):
        data_ptr = self.tensor.data_ptr()
        total_bytes = self.tensor.numel() * self.tensor.element_size()
        c_obj = (ctypes.c_char * total_bytes).from_address(data_ptr)
        c_obj._keep_alive_ref = self
        return memoryview(c_obj)
```
**EN:** This block defines the method `__buffer__` on `TensorWrapper`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `__buffer__`. Notable operations include `data_ptr`, `numel`, `element_size`, `from_address`.
**CN:** 这一段定义了method `__buffer__`（属于 `TensorWrapper`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `__buffer__`。 值得注意的操作包括 `data_ptr`、`numel`、`element_size`、`from_address`。

### Lines 126-136: Function `_convert`
```python
def _convert(data):
    if isinstance(data, torch.Tensor):
        return data
    elif isinstance(data, np.ndarray):
        return torch.tensor(data)
    elif isinstance(data, list) and isinstance(data[0], np.ndarray):
        return torch.tensor(np.array(data))
    elif isinstance(data, list) and isinstance(data[0], (int, float)):
        return torch.tensor(data)
    else:
        return data
```
**EN:** This block defines the function `_convert`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_convert`. Notable operations include `tensor`, `array`.
**CN:** 这一段定义了function `_convert`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_convert`。 值得注意的操作包括 `tensor`、`array`。

### Lines 137-152: Module-level constants and helper logic
```python


_mm_grid_attrs = {
    # Kimi K2.5 HF processor uses grid_thws (see base_processor.ATTR_NAME_TO_MODALITY).
    Modality.IMAGE: ["image_grid_thw", "image_grid_hws", "grid_thws"],
    Modality.VIDEO: ["video_grid_thw"],
    Modality.AUDIO: ["audio_feature_lens_raw"],
}

_mm_feature_attrs = {
    Modality.IMAGE: ["pixel_values"],
    Modality.VIDEO: ["pixel_values_videos"],
    Modality.AUDIO: ["input_features"],
}


```
**EN:** This block contains module-level constants, helpers, or documentation for encode-side transfer server orchestration. It prepares shared state that later classes and functions build on. Notable operations include `grid_thws`.
**CN:** 这一段包含与编码侧传输服务编排相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `grid_thws`。

### Lines 153-165: Function `_get_mm_grid_dim`
```python
def _get_mm_grid_dim(mm_inputs, modality, model_type: Optional[str] = None):
    # Kimi K2.5 vision processor only emits `grid_thws`; prefer it over generic keys
    # so we never pick a mis-typed or stale `image_grid_hws` field from kwargs.
    attrs = _mm_grid_attrs[modality]
    if (model_type or "").lower() in [
        "kimi_k25",
        "kimi_vl",
    ] and modality == Modality.IMAGE:
        attrs = ("grid_thws", "image_grid_thw", "image_grid_hws")
    for attr in attrs:
        if attr in mm_inputs and mm_inputs[attr] is not None:
            return mm_inputs[attr]
    raise ValueError(f"Grid dim ({_mm_grid_attrs[modality]}) not found in {mm_inputs}")
```
**EN:** This block defines the function `_get_mm_grid_dim`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_get_mm_grid_dim`. Notable operations include `lower`, `ValueError`, `dim`.
**CN:** 这一段定义了function `_get_mm_grid_dim`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_get_mm_grid_dim`。 值得注意的操作包括 `lower`、`ValueError`、`dim`。

### Lines 168-174: Function `_get_mm_feature`
```python
def _get_mm_feature(mm_inputs, modality):
    for attr in _mm_feature_attrs[modality]:
        if attr in mm_inputs:
            return mm_inputs[attr]
    raise ValueError(
        f"Feature attrs ({_mm_feature_attrs[modality]}) not found in {mm_inputs}"
    )
```
**EN:** This block defines the function `_get_mm_feature`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_get_mm_feature`. Notable operations include `ValueError`, `attrs`.
**CN:** 这一段定义了function `_get_mm_feature`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_get_mm_feature`。 值得注意的操作包括 `ValueError`、`attrs`。

### Lines 177-179: Function `_build_mm_aux_data`
```python
def _build_mm_aux_data(mm_inputs, model_type=None):
    # Video aux metadata, scoped to model_type's video-meta attrs.
    return {attr: mm_inputs.get(attr) for attr in video_meta_attrs_for(model_type)}
```
**EN:** This block defines the function `_build_mm_aux_data`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_build_mm_aux_data`. Notable operations include `get`, `video_meta_attrs_for`.
**CN:** 这一段定义了function `_build_mm_aux_data`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_build_mm_aux_data`。 值得注意的操作包括 `get`、`video_meta_attrs_for`。

### Lines 182-182: Class `MMEncoder` declaration
```python
class MMEncoder:
```
**EN:** This block declares the class `MMEncoder` and establishes its responsibility inside encode-side transfer server orchestration. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `MMEncoder`.
**CN:** 这一段声明了类 `MMEncoder`，并说明它在编码侧传输服务编排中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `MMEncoder`。

### Lines 183-185: Method `__init__` signature and setup
```python
    def __init__(
        self,
        server_args: ServerArgs,
```
**EN:** This block defines the method `__init__` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 186-218: Method `__init__` logic (part 1)
```python
        schedule_path=None,
        dist_init_method=None,
        rank: int = 0,
    ):
        logger.info(f"init MMEncoder {rank}/{server_args.tp_size}")
        self.server_args = server_args
        set_global_server_args_for_scheduler(server_args)
        self.rank = rank
        self.profiler = EncoderProfiler(rank)
        self._load_mm_processor(server_args)

        self.model_config = ModelConfig.from_server_args(
            server_args,
        )
        self.load_config = LoadConfig(
            load_format=server_args.load_format,
            download_dir=server_args.download_dir,
            model_loader_extra_config=server_args.model_loader_extra_config,
            remote_instance_weight_loader_seed_instance_ip=server_args.remote_instance_weight_loader_seed_instance_ip,
            remote_instance_weight_loader_seed_instance_service_port=server_args.remote_instance_weight_loader_seed_instance_service_port,
            remote_instance_weight_loader_send_weights_group_ports=server_args.remote_instance_weight_loader_send_weights_group_ports,
        )
        self.model_type = getattr(
            self.model_config.hf_config, "model_type", "unknown"
        ).lower()

        self.device = server_args.device
        self.gpu_id = server_args.base_gpu_id + rank

        self.device_config = DeviceConfig(
            device=self.device,
            gpu_id=self.gpu_id,
        )
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `info`, `set_global_server_args_for_scheduler`, `EncoderProfiler`, `_load_mm_processor`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `info`、`set_global_server_args_for_scheduler`、`EncoderProfiler`、`_load_mm_processor`。

### Lines 219-249: Method `__init__` logic (part 2)
```python

        torch.get_device_module(self.device).set_device(self.gpu_id)

        self.use_image_processor_gpu = (
            use_image_processor_gpu and not server_args.disable_fast_image_processor
        )
        self._build_vision_config(server_args.mm_process_config)

        init_distributed_environment(
            backend=get_default_distributed_backend(self.device),
            world_size=server_args.tp_size,
            rank=rank,
            distributed_init_method=dist_init_method,
            local_rank=rank,
        )
        initialize_model_parallel(tensor_model_parallel_size=server_args.tp_size)
        initialize_dp_attention(server_args, self.model_config)

        self.model = get_model(
            model_config=self.model_config,
            load_config=self.load_config,
            device_config=self.device_config,
        )

        self.context = zmq.asyncio.Context(2)
        self.sync_context = zmq.Context()  # Reuse sync context for thread pool
        self.executor = concurrent.futures.ThreadPoolExecutor(max_workers=10)

        embedding_cache_size = int(os.environ.get("SGLANG_VLM_CACHE_SIZE_MB", "4096"))
        self.mm_cache = MultiModalStaticCache(embedding_cache_size * 1024 * 1024)
        self.mm_cache_lock = asyncio.Lock()
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `get_device_module`, `set_device`, `_build_vision_config`, `init_distributed_environment`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `get_device_module`、`set_device`、`_build_vision_config`、`init_distributed_environment`。

### Lines 250-281: Method `__init__` logic (part 3)
```python

        self.io_executor = concurrent.futures.ThreadPoolExecutor(
            max_workers=int(os.environ.get("SGLANG_ENCODER_MM_LOAD_WORKERS", 4))
        )
        self.send_timeout = envs.SGLANG_ENCODER_SEND_TIMEOUT.get()

        if schedule_path is not None:
            self.schedule_socket = get_zmq_socket(
                self.context, zmq.PULL, schedule_path, True
            )
        self.background_tasks: Set[asyncio.Task] = set()

        if self.server_args.enable_mm_global_cache:
            from sglang.srt.mem_cache.storage.mooncake_store.embedding_cache_controller import (
                EmbeddingCacheController,
            )

            hidden_dims = self._infer_embedding_dims()
            self.mm_global_cache = EmbeddingCacheController(
                rank,
                server_args.tp_size,
                hidden_dims=hidden_dims,
                tp_group=get_tp_group().cpu_group,
                all_rank_get=False,
            )
        else:
            self.mm_global_cache = None

        if self.rank == 0:
            logger.info(
                f"Using transfer backend: {self.server_args.encoder_transfer_backend}"
            )
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `ThreadPoolExecutor`, `get`, `get_zmq_socket`, `import`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `ThreadPoolExecutor`、`get`、`get_zmq_socket`、`import`。

### Lines 282-303: Method `__init__` logic (part 4)
```python

            if self.server_args.encoder_transfer_backend == "mooncake":
                self.local_ip = get_local_ip_auto()

                self.engine = get_mooncake_transfer_engine()
                if self.engine is None:
                    from sglang.srt.distributed.device_communicators.mooncake_transfer_engine import (
                        init_mooncake_transfer_engine,
                    )

                    self.engine = init_mooncake_transfer_engine(
                        hostname=self.local_ip,
                        gpu_id=self.gpu_id,
                        ib_device=(
                            self.server_args.disaggregation_ib_device
                            or self.server_args.mooncake_ib_device
                        ),
                    )

            self.embedding_to_send = dict()

        logger.info(f"rank {rank} init finish ")
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `get_local_ip_auto`, `get_mooncake_transfer_engine`, `import`, `init_mooncake_transfer_engine`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `get_local_ip_auto`、`get_mooncake_transfer_engine`、`import`、`init_mooncake_transfer_engine`。

### Lines 305-306: Method `_infer_embedding_dims` signature and setup
```python
    def _infer_embedding_dims(self) -> dict:
        """Infer per-modality embedding dimensions from hf_config at init time."""
```
**EN:** This block defines the method `_infer_embedding_dims` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_infer_embedding_dims`.
**CN:** 这一段定义了method `_infer_embedding_dims`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_infer_embedding_dims`。

### Lines 307-329: Method `_infer_embedding_dims` logic (part 1)
```python
        default = self.model_config.hidden_size
        hf_cfg = self.model_config.hf_config
        thinker_cfg = getattr(hf_cfg, "thinker_config", None)
        dims = {
            Modality.IMAGE: default,
            Modality.VIDEO: default,
            Modality.AUDIO: default,
        }

        vision_cfg = getattr(thinker_cfg, "vision_config", None) or getattr(
            hf_cfg, "vision_config", None
        )
        if vision_cfg is not None:
            out_hs = getattr(vision_cfg, "out_hidden_size", None)
            if out_hs is not None:
                ds = getattr(vision_cfg, "deepstack_visual_indexes", None)
                vis_dim = (
                    out_hs * (1 + len(ds))
                    if isinstance(ds, (list, tuple)) and ds
                    else out_hs
                )
                dims[Modality.IMAGE] = vis_dim
                dims[Modality.VIDEO] = vis_dim
```
**EN:** This block continues `_infer_embedding_dims` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow.
**CN:** 这一段延续了 `_infer_embedding_dims` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。

### Lines 330-342: Method `_infer_embedding_dims` logic (part 2)
```python

        audio_cfg = getattr(thinker_cfg, "audio_config", None) or getattr(
            hf_cfg, "audio_config", None
        )
        if audio_cfg is not None:
            for attr in ("output_dim", "d_model"):
                val = getattr(audio_cfg, attr, None)
                if val and int(val) > 0:
                    dims[Modality.AUDIO] = int(val)
                    break

        logger.info(f"Global cache embedding dims: {dims}")
        return dims
```
**EN:** This block continues `_infer_embedding_dims` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `in`, `info`.
**CN:** 这一段延续了 `_infer_embedding_dims` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `in`、`info`。

### Lines 344-348: Method `_build_vision_config` signature and setup
```python
    def _build_vision_config(self, mm_process_config):
        """
        Validate vision config, used for image/video/audio.
        If not provided, keep default values.
        """
```
**EN:** This block defines the method `_build_vision_config` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_build_vision_config`.
**CN:** 这一段定义了method `_build_vision_config`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_build_vision_config`。

### Lines 349-363: Method `_build_vision_config` logic (part 1)
```python
        self.vision_config = (
            mm_process_config.get("vision_config", {})
            if mm_process_config is not None
            else {}
        )
        for modality_str in ["image", "video", "audio"]:
            if not self.vision_config.get(modality_str, None):
                self.vision_config[modality_str] = {}
            if self.use_image_processor_gpu:
                self.vision_config[modality_str]["device"] = self.device

            if modality_str == "video":
                video_defaults = {"fps": 2.0, "max_frames": 768, "min_frames": 4}
                for k, v in video_defaults.items():
                    self.vision_config["video"].setdefault(k, v)
```
**EN:** This block continues `_build_vision_config` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `get`, `items`, `setdefault`.
**CN:** 这一段延续了 `_build_vision_config` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `get`、`items`、`setdefault`。

### Lines 364-387: Method `_build_vision_config` logic (part 2)
```python

            if modality_str == "audio":
                if "return_attention_mask" not in self.vision_config["audio"]:
                    self.vision_config["audio"]["return_attention_mask"] = True
                if "padding" not in self.vision_config["audio"]:
                    if self.model_type == "qwen2_audio":
                        # For Qwen2Audio, use padding="max_length"
                        # (same as https://github.com/huggingface/transformers/blob/main/src/transformers/models/qwen2_audio/processing_qwen2_audio.py#L93)
                        self.vision_config["audio"]["padding"] = "max_length"
                    else:
                        self.vision_config["audio"]["padding"] = True
                if "truncation" not in self.vision_config["audio"]:
                    # keep same logic as base_processor.py
                    if (
                        hasattr(self, "audio_processor")
                        and self.audio_processor is not None
                    ):
                        if self.audio_processor.__class__.__name__ in {
                            "Gemma3nProcessor",
                            "GlmAsrProcessor",
                            "Qwen2AudioProcessor",
                            "Qwen3OmniMoeProcessor",
                        }:
                            self.vision_config["audio"]["truncation"] = False
```
**EN:** This block continues `_build_vision_config` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow.
**CN:** 这一段延续了 `_build_vision_config` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。

### Lines 389-393: Method `_load_mm_processor` signature and setup
```python
    def _load_mm_processor(self, server_args: ServerArgs):
        """
        Load image/video/audio processor separately,
        avoid issues with AutoProcessor not recognizing certain models
        """
```
**EN:** This block defines the method `_load_mm_processor` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_load_mm_processor`.
**CN:** 这一段定义了method `_load_mm_processor`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_load_mm_processor`。

### Lines 394-416: Method `_load_mm_processor` logic (part 1)
```python
        from transformers import AutoImageProcessor, AutoVideoProcessor

        try:
            self.image_processor = AutoImageProcessor.from_pretrained(
                server_args.tokenizer_path or server_args.model_path,
                trust_remote_code=server_args.trust_remote_code,
                revision=server_args.revision,
                use_fast=not server_args.disable_fast_image_processor,
            )
        except Exception as e:
            logger.warning(f"Failed to load image processor: {e}")
            self.image_processor = None

        try:
            self.video_processor = AutoVideoProcessor.from_pretrained(
                server_args.tokenizer_path or server_args.model_path,
                trust_remote_code=server_args.trust_remote_code,
                revision=server_args.revision,
                use_fast=not server_args.disable_fast_image_processor,
            )
        except Exception as e:
            logger.warning(f"Failed to load video processor: {e}")
            self.video_processor = None
```
**EN:** This block continues `_load_mm_processor` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `from_pretrained`, `warning`.
**CN:** 这一段延续了 `_load_mm_processor` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `from_pretrained`、`warning`。

### Lines 417-436: Method `_load_mm_processor` logic (part 2)
```python

        try:
            # Note: AutoProcessor is used for audio processor
            _audio_proc = AutoProcessor.from_pretrained(
                server_args.tokenizer_path or server_args.model_path,
                trust_remote_code=server_args.trust_remote_code,
                revision=server_args.revision,
                use_fast=not server_args.disable_fast_image_processor,
            )
            if not hasattr(_audio_proc, "feature_extractor"):
                logger.warning(
                    "Loaded AutoProcessor has no feature_extractor attribute, "
                    "audio processing will be unavailable."
                )
                self.audio_processor = None
            else:
                self.audio_processor = _audio_proc
        except Exception as e:
            logger.warning(f"Failed to load audio processor: {e}")
            self.audio_processor = None
```
**EN:** This block continues `_load_mm_processor` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `from_pretrained`, `warning`.
**CN:** 这一段延续了 `_load_mm_processor` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `from_pretrained`、`warning`。

### Lines 438-449: Method `_load_single_item` signature and setup
```python
    def _load_single_item(
        self,
        data,
        modality: Modality,
        frame_count_limit=None,
        audio_sample_rate: Optional[int] = None,
        discard_alpha_channel=True,
    ):
        """
        Load a single multimodal data.
        If data is precomputed, returns directly.
        Static method that can be pickled for multiprocessing"""
```
**EN:** This block defines the method `_load_single_item` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_load_single_item`.
**CN:** 这一段定义了method `_load_single_item`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_load_single_item`。

### Lines 450-469: Method `_load_single_item` logic (part 1)
```python
        if isinstance(data, dict):
            return data
        try:
            if modality == Modality.IMAGE:
                img, _ = load_image(data, False)
                if (
                    discard_alpha_channel
                    and not isinstance(img, torch.Tensor)
                    and img.mode != "RGB"
                ):
                    # Needed only when `img` is a PIL image
                    img = img.convert("RGB")
                return img
            elif modality == Modality.VIDEO:
                return load_video(data, frame_count_limit)
            elif modality == Modality.AUDIO:
                return load_audio(data, audio_sample_rate)

        except Exception as e:
            raise RuntimeError(f"Error while loading data {data}: {e}")
```
**EN:** This block continues `_load_single_item` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `load_image`, `convert`, `load_video`, `load_audio`.
**CN:** 这一段延续了 `_load_single_item` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `load_image`、`convert`、`load_video`、`load_audio`。

### Lines 471-485: Method `submit_data_loading_tasks`
```python
    def submit_data_loading_tasks(self, items, modalities):
        futures = []
        task_info = []

        for data, modality in zip(items, modalities):
            if modality is not None:
                futures.append(
                    self.io_executor.submit(
                        self._load_single_item,
                        data,
                        modality,
                    )
                )
                task_info.append((modality, data))
        return futures, task_info
```
**EN:** This block defines the method `submit_data_loading_tasks` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `submit_data_loading_tasks`. Notable operations include `append`, `submit`.
**CN:** 这一段定义了method `submit_data_loading_tasks`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `submit_data_loading_tasks`。 值得注意的操作包括 `append`、`submit`。

### Lines 487-509: Method `_get_feat_extract_output_lengths`
```python
    def _get_feat_extract_output_lengths(self, feature_lens):
        """
        Computes the output length of the convolutional layers and the output length of the audio encoder
        """
        # qwen2_audio/qwen2.5_omni
        if self.model_type in ["qwen2_audio", "qwen2_5_omni"]:
            input_length = (feature_lens - 1) // 2 + 1
            return (input_length - 2) // 2 + 1
        # qwen3_asr / qwen3_omni_moe (same audio encoder architecture)
        elif self.model_type in ["qwen3_asr", "qwen3_omni_moe"]:
            input_lengths_leave = feature_lens % 100
            feat_lengths = (input_lengths_leave - 1) // 2 + 1
            output_lengths = (
                ((feat_lengths - 1) // 2 + 1 - 1) // 2 + 1 + (feature_lens // 100) * 13
            )
            return output_lengths
        else:
            # fallback to original HF audio sample logic for other models
            logger.warning(
                f"Fallback to original HF audio sample logic for {self.model_type}"
            )
            input_length = (feature_lens - 1) // 2 + 1
            return (input_length - 2) // 2 + 1
```
**EN:** This block defines the method `_get_feat_extract_output_lengths` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_get_feat_extract_output_lengths`. Notable operations include `qwen3_omni_moe`, `warning`.
**CN:** 这一段定义了method `_get_feat_extract_output_lengths`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_get_feat_extract_output_lengths`。 值得注意的操作包括 `qwen3_omni_moe`、`warning`。

### Lines 511-513: Method `_flatten_and_load_videos` signature and setup
```python
    async def _flatten_and_load_videos(self, mm_items):
        if not isinstance(mm_items, (list, tuple)):
            mm_items = [mm_items]
```
**EN:** This block defines the method `_flatten_and_load_videos` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_flatten_and_load_videos`.
**CN:** 这一段定义了method `_flatten_and_load_videos`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_flatten_and_load_videos`。

### Lines 514-537: Method `_flatten_and_load_videos` logic (part 1)
```python

        futures, _ = self.submit_data_loading_tasks(
            mm_items, [Modality.VIDEO] * len(mm_items)
        )
        async_futures = [asyncio.wrap_future(f) for f in futures]
        video_items = await asyncio.gather(*async_futures)

        video_processor_kwargs = {}
        if "qwen" in self.model_type:
            # for qwen-series model, do sample frames before preprocess
            video_processed = [
                await preprocess_video(
                    video, video_config=self.vision_config.get("video", {})
                )
                for video in video_items
            ]
            videos, video_metadata = map(list, zip(*video_processed))
            video_processor_kwargs["do_sample_frames"] = False
            if video_metadata:
                video_processor_kwargs["video_metadata"] = video_metadata
            return videos, video_processor_kwargs
        else:
            raise NotImplementedError(
                f"Video processing is not supported for {self.model_type} model."
```
**EN:** This block continues `_flatten_and_load_videos` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `submit_data_loading_tasks`, `wrap_future`, `gather`, `preprocess_video`.
**CN:** 这一段延续了 `_flatten_and_load_videos` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `submit_data_loading_tasks`、`wrap_future`、`gather`、`preprocess_video`。

### Lines 538-538: Method `_flatten_and_load_videos` logic (part 2)
```python
            )
```
**EN:** This block continues `_flatten_and_load_videos` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow.
**CN:** 这一段延续了 `_flatten_and_load_videos` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。

### Lines 540-546: Method `_flatten_and_load_data_by_modality` signature and setup
```python
    async def _flatten_and_load_data_by_modality(self, mm_items, modality):
        """
        Flatten mm_items structure, load multimodal data concurrently, and restore original structure.

        Returns:
            Same structure as load_mm_items would return, support for image/audio
        """
```
**EN:** This block defines the method `_flatten_and_load_data_by_modality` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_flatten_and_load_data_by_modality`.
**CN:** 这一段定义了method `_flatten_and_load_data_by_modality`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_flatten_and_load_data_by_modality`。

### Lines 547-569: Method `_flatten_and_load_data_by_modality` logic (part 1)
```python
        # Handle single mm_item (not a list)
        if not isinstance(mm_items, (list, tuple)):
            futures, _ = self.submit_data_loading_tasks([mm_items], [modality])
            return await asyncio.wrap_future(futures[0])

        # Handle nested list (list of lists)
        if len(mm_items) > 0 and isinstance(mm_items[0], (list, tuple)):
            # Flatten nested structure
            flat_data = []
            flat_indices = []  # Track which group each item belongs to
            for group_idx, item_group in enumerate(mm_items):
                for item in item_group:
                    flat_data.append(item)
                    flat_indices.append(group_idx)

            # Submit all tasks concurrently
            futures, _ = self.submit_data_loading_tasks(
                flat_data, [modality] * len(flat_data)
            )

            # Wait for all tasks to complete asynchronously
            async_futures = [asyncio.wrap_future(f) for f in futures]
            results = await asyncio.gather(*async_futures)
```
**EN:** This block continues `_flatten_and_load_data_by_modality` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `mm_item`, `submit_data_loading_tasks`, `wrap_future`, `append`.
**CN:** 这一段延续了 `_flatten_and_load_data_by_modality` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `mm_item`、`submit_data_loading_tasks`、`wrap_future`、`append`。

### Lines 570-585: Method `_flatten_and_load_data_by_modality` logic (part 2)
```python

            # Restore nested structure
            nested_results = [[] for _ in range(len(mm_items))]
            for idx, result in zip(flat_indices, results):
                nested_results[idx].append(result)

            return nested_results

        # Handle simple list
        else:
            futures, _ = self.submit_data_loading_tasks(
                mm_items, [modality] * len(mm_items)
            )
            # Wait for all tasks to complete asynchronously
            async_futures = [asyncio.wrap_future(f) for f in futures]
            return await asyncio.gather(*async_futures)
```
**EN:** This block continues `_flatten_and_load_data_by_modality` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `append`, `submit_data_loading_tasks`, `wrap_future`, `gather`.
**CN:** 这一段延续了 `_flatten_and_load_data_by_modality` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `append`、`submit_data_loading_tasks`、`wrap_future`、`gather`。

### Lines 587-594: Method `get_num_patches`
```python
    def get_num_patches(
        self, grid: Union[torch.Tensor, List[int]], modality: Modality
    ) -> int:
        """Calculate number of raw patches (before merge/sampling). Used for pixel_values slicing."""
        if modality == Modality.AUDIO:
            return int(grid.item())
        else:
            return int(grid[0] * grid[1] * grid[2])
```
**EN:** This block defines the method `get_num_patches` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `get_num_patches`. Notable operations include `patches`, `item`.
**CN:** 这一段定义了method `get_num_patches`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `get_num_patches`。 值得注意的操作包括 `patches`、`item`。

### Lines 596-604: Method `_kimi_tokens_from_patch_grid`
```python
    def _kimi_tokens_from_patch_grid(self, grid: Union[torch.Tensor, List[int]]) -> int:
        """MoonViT + tpool: output len is (h//mh)*(w//mw); temporal dim is pooled (not t*h*w/merge^2)."""
        if isinstance(grid, torch.Tensor):
            flat = grid.flatten()
            _t, h, w = (int(x) for x in flat[:3].tolist())
        else:
            _t, h, w = int(grid[0]), int(grid[1]), int(grid[2])
        merge_h, merge_w = self.model_config.hf_config.vision_config.merge_kernel_size
        return (h * w) // (merge_h * merge_w)
```
**EN:** This block defines the method `_kimi_tokens_from_patch_grid` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_kimi_tokens_from_patch_grid`. Notable operations include `is`, `pooled`, `flatten`, `tolist`.
**CN:** 这一段定义了method `_kimi_tokens_from_patch_grid`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_kimi_tokens_from_patch_grid`。 值得注意的操作包括 `is`、`pooled`、`flatten`、`tolist`。

### Lines 606-620: Method `get_num_tokens`
```python
    def get_num_tokens(
        self, grid: Union[torch.Tensor, List[int]], modality: Modality
    ) -> int:
        """Calculate number of tokens (after 2x2 merge). Used for mm_embedding slicing."""
        if modality == Modality.AUDIO:
            input_length = self.get_num_patches(grid, modality)
            return self._get_feat_extract_output_lengths(input_length)
        else:
            if (
                self.model_type in ["kimi_k25", "kimi_vl"]
                and modality == Modality.IMAGE
            ):
                return self._kimi_tokens_from_patch_grid(grid)
            merge_size = getattr(self.image_processor, "merge_size", 2)
            return self.get_num_patches(grid, modality) // (merge_size**2)
```
**EN:** This block defines the method `get_num_tokens` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `get_num_tokens`. Notable operations include `tokens`, `get_num_patches`, `_get_feat_extract_output_lengths`, `_kimi_tokens_from_patch_grid`.
**CN:** 这一段定义了method `get_num_tokens`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `get_num_tokens`。 值得注意的操作包括 `tokens`、`get_num_patches`、`_get_feat_extract_output_lengths`、`_kimi_tokens_from_patch_grid`。

### Lines 622-631: Method `slice_embedding`
```python
    def slice_embedding(
        self, mm_embedding: torch.Tensor, grid_thw: List, modality: Modality
    ) -> List[torch.Tensor]:
        """Slice a concatenated embedding tensor into individual image embeddings."""
        slices, offset = [], 0
        for grid in grid_thw:
            count = self.get_num_tokens(grid, modality)
            slices.append(mm_embedding[offset : offset + count])
            offset += count
        return slices
```
**EN:** This block defines the method `slice_embedding` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `slice_embedding`. Notable operations include `get_num_tokens`, `append`.
**CN:** 这一段定义了method `slice_embedding`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `slice_embedding`。 值得注意的操作包括 `get_num_tokens`、`append`。

### Lines 633-646: Method `_calculate_hashes_from_features`
```python
    def _calculate_hashes_from_features(
        self, mm_feature: torch.Tensor, grid_thw: List, modality: Modality
    ) -> List[str]:
        """CPU Task: Compute hashes based on processed feature patches."""
        hashes, offset = [], 0
        logger.info(f"{mm_feature.shape=} with {modality=}")
        for grid in grid_thw:
            num_patches = self.get_num_patches(grid, modality)
            feature_slice = mm_feature[offset : offset + num_patches]
            tmp_item = MultimodalDataItem(modality=modality, feature=feature_slice)
            tmp_item.set_pad_value()
            hashes.append(tmp_item.hash)
            offset += num_patches
        return hashes
```
**EN:** This block defines the method `_calculate_hashes_from_features` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_calculate_hashes_from_features`. Notable operations include `info`, `get_num_patches`, `MultimodalDataItem`, `set_pad_value`.
**CN:** 这一段定义了method `_calculate_hashes_from_features`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_calculate_hashes_from_features`。 值得注意的操作包括 `info`、`get_num_patches`、`MultimodalDataItem`、`set_pad_value`。

### Lines 648-658: Method `_encode_missing` signature and setup
```python
    async def _encode_missing(
        self,
        mm_feature: torch.Tensor,
        mm_inputs: dict,
        indices: List[int],
        modality: Modality = Modality.IMAGE,
        get_feature_fn=None,
    ) -> List[torch.Tensor]:
        """
        GPU Task: Run ViT inference ONLY on the subset of mm items missing from the cache.
        """
```
**EN:** This block defines the method `_encode_missing` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_encode_missing`.
**CN:** 这一段定义了method `_encode_missing`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_encode_missing`。

### Lines 659-679: Method `_encode_missing` logic (part 1)
```python
        grid_thw = _get_mm_grid_dim(mm_inputs, modality, self.model_type)

        # 1. Slice mm_feature to get only the patches for missing mm items
        sub_feature_list = []
        offsets = [0]
        curr = 0
        for g in grid_thw:
            curr += self.get_num_patches(g, modality)
            offsets.append(curr)

        for idx in indices:
            sub_feature_list.append(mm_feature[offsets[idx] : offsets[idx + 1]])

        sub_feature = torch.cat(sub_feature_list, dim=0)

        mm_item = MultimodalDataItem.from_dict(
            {
                "modality": modality,
                "feature": _convert(sub_feature),
            }
        )
```
**EN:** This block continues `_encode_missing` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `_get_mm_grid_dim`, `get_num_patches`, `append`, `cat`.
**CN:** 这一段延续了 `_encode_missing` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `_get_mm_grid_dim`、`get_num_patches`、`append`、`cat`。

### Lines 680-696: Method `_encode_missing` logic (part 2)
```python

        for k, v in mm_inputs.items():
            if k in _mm_feature_attrs.get(modality, []):
                continue
            val = _convert(v)
            if k in _mm_grid_attrs.get(modality, []):
                mm_item.set(k, val[indices])
            else:
                mm_item.set(k, val)

        with torch.inference_mode():
            new_embeddings = get_feature_fn([mm_item]).cpu()
            if new_embeddings.ndim != 2:
                new_embeddings = new_embeddings.reshape(-1, new_embeddings.shape[-1])

        sub_grids = [grid_thw[i] for i in indices]
        return self.slice_embedding(new_embeddings, sub_grids, modality)
```
**EN:** This block continues `_encode_missing` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `items`, `get`, `_convert`, `inference_mode`.
**CN:** 这一段延续了 `_encode_missing` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `items`、`get`、`_convert`、`inference_mode`。

### Lines 698-700: Method `encode_with_global_cache` signature and setup
```python
    async def encode_with_global_cache(
        self,
        mm_items,
```
**EN:** This block defines the method `encode_with_global_cache` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `encode_with_global_cache`.
**CN:** 这一段定义了method `encode_with_global_cache`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `encode_with_global_cache`。

### Lines 701-734: Method `encode_with_global_cache` logic (part 1)
```python
        modality: Modality,
        req_id: str,
        num_parts: int,
        part_idx: int,
        hashes: Optional[List[str]] = None,
    ) -> torch.Tensor:
        # mm_inputs: dict
        mm_inputs, get_feature_fn = await self._process_mm_items(mm_items, modality)
        grid_thw = _get_mm_grid_dim(mm_inputs, modality, self.model_type)
        mm_feature = _convert(_get_mm_feature(mm_inputs, modality))
        num_items = len(grid_thw)

        # Step 1: Rank 0 checks global cache and broadcasts hit/miss mask to all ranks.
        if self.rank == 0:
            if hashes is None:
                mm_hashes = self._calculate_hashes_from_features(
                    mm_feature, grid_thw, modality
                )
            else:
                mm_hashes = hashes
            exist_mask = await self.mm_global_cache.batch_is_exist(mm_hashes)
            mask_tensor = torch.tensor(
                [1 if e else 0 for e in exist_mask], dtype=torch.int32
            )
        else:
            mm_hashes = None
            mask_tensor = torch.zeros(num_items, dtype=torch.int32)

        if self.server_args.tp_size > 1:
            torch.distributed.broadcast(
                mask_tensor,
                src=0,
                group=self.mm_global_cache.prefetch_tp_group,
            )
```
**EN:** This block continues `encode_with_global_cache` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `_process_mm_items`, `_get_mm_grid_dim`, `_convert`, `_get_mm_feature`.
**CN:** 这一段延续了 `encode_with_global_cache` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `_process_mm_items`、`_get_mm_grid_dim`、`_convert`、`_get_mm_feature`。

### Lines 735-762: Method `encode_with_global_cache` logic (part 2)
```python

        exist_mask = [m.item() == 1 for m in mask_tensor]
        missing_indices = [i for i, e in enumerate(exist_mask) if not e]
        hit_indices = [i for i, e in enumerate(exist_mask) if e]

        # Step 2: All ranks run ViT together on cache-miss images.
        new_slices = []
        if missing_indices:
            new_slices = await self._encode_missing(
                mm_feature, mm_inputs, missing_indices, modality, get_feature_fn
            )

        # Step 3: Rank 0 prefetches cache-hit embeddings from global cache.
        prefetch_status = torch.tensor([1], dtype=torch.int32)

        if self.rank == 0:
            if hit_indices:
                hit_hashes = [mm_hashes[i] for i in hit_indices]
                hit_tokens = [
                    self.get_num_tokens(grid_thw[i], modality) for i in hit_indices
                ]
                self.mm_global_cache.prefetch(req_id, hit_hashes, hit_tokens, modality)

                try:

                    async def _wait_prefetch():
                        while not self.mm_global_cache.check_prefetch_progress(req_id):
                            await asyncio.sleep(0.005)
```
**EN:** This block continues `encode_with_global_cache` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Definitions introduced here include `_wait_prefetch`. Notable operations include `item`, `_encode_missing`, `tensor`, `get_num_tokens`.
**CN:** 这一段延续了 `encode_with_global_cache` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 此处引入的定义包括 `_wait_prefetch`。 值得注意的操作包括 `item`、`_encode_missing`、`tensor`、`get_num_tokens`。

### Lines 763-794: Method `encode_with_global_cache` logic (part 3)
```python

                    await asyncio.wait_for(_wait_prefetch(), timeout=60.0)
                except (asyncio.TimeoutError, Exception) as e:
                    logger.error(
                        f"Prefetch failed for req {req_id}: {e}. "
                        f"Falling back to ViT for {len(hit_indices)} hit items."
                    )
                    prefetch_status[0] = 0

        # Step 4: Broadcast prefetch result to all ranks so they stay in sync.
        if self.server_args.tp_size > 1:
            torch.distributed.broadcast(
                prefetch_status,
                src=0,
                group=self.mm_global_cache.prefetch_tp_group,
            )

        # Step 5: If prefetch failed, all ranks fallback to ViT for the hit mm items.
        if prefetch_status.item() == 0 and hit_indices:
            logger.info(
                f"Req {req_id}: Prefetch failed, all ranks running ViT fallback "
                f"for {len(hit_indices)} mm items."
            )
            fallback_slices = await self._encode_missing(
                mm_feature, mm_inputs, hit_indices, modality, get_feature_fn
            )
        else:
            fallback_slices = None

        # Step 6: Rank 0 assembles final embedding and prepares for sending.
        if self.rank == 0:
            final_slices = [None] * num_items
```
**EN:** This block continues `encode_with_global_cache` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `wait_for`, `_wait_prefetch`, `error`, `broadcast`.
**CN:** 这一段延续了 `encode_with_global_cache` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `wait_for`、`_wait_prefetch`、`error`、`broadcast`。

### Lines 795-827: Method `encode_with_global_cache` logic (part 4)
```python

            for i, idx in enumerate(missing_indices):
                final_slices[idx] = new_slices[i]

            # Fill in cache-hit embeddings (from prefetch or fallback)
            if prefetch_status.item() == 1 and hit_indices:
                cached_slices = self.mm_global_cache.get_embeddings(
                    [mm_hashes[i] for i in hit_indices]
                )
                for i, idx in enumerate(hit_indices):
                    final_slices[idx] = cached_slices[i]
            elif fallback_slices is not None:
                for i, idx in enumerate(hit_indices):
                    final_slices[idx] = fallback_slices[i]

            mm_embedding = torch.cat(final_slices, dim=0)

            # Background insert: store newly computed embeddings into global cache.
            # Includes both original misses and fallback-recomputed hits.
            all_new_hashes = [mm_hashes[i] for i in missing_indices]
            all_new_slices = list(new_slices)
            if fallback_slices is not None:
                all_new_hashes += [mm_hashes[i] for i in hit_indices]
                all_new_slices += list(fallback_slices)

            if all_new_hashes:

                async def _background_insert():
                    await asyncio.to_thread(
                        self.mm_global_cache.insert_batch,
                        all_new_hashes,
                        all_new_slices,
                    )
```
**EN:** This block continues `encode_with_global_cache` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Definitions introduced here include `_background_insert`. Notable operations include `embeddings`, `item`, `get_embeddings`, `cat`.
**CN:** 这一段延续了 `encode_with_global_cache` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 此处引入的定义包括 `_background_insert`。 值得注意的操作包括 `embeddings`、`item`、`get_embeddings`、`cat`。

### Lines 828-851: Method `encode_with_global_cache` logic (part 5)
```python

                task = asyncio.create_task(_background_insert())
                self.background_tasks.add(task)
                task.add_done_callback(self.background_tasks.discard)

            aux_data = _build_mm_aux_data(mm_inputs, self.model_type)
            self.embedding_to_send[req_id] = EmbeddingData(
                req_id,
                num_parts,
                part_idx,
                grid_thw,
                modality,
                mm_embedding,
                **aux_data,
            )
            return (
                mm_embedding.nbytes,
                mm_embedding.shape[0],
                mm_embedding.shape[1],
                None,
                None,
            )
        else:
            return (0, 0, 0, None, None)
```
**EN:** This block continues `encode_with_global_cache` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `create_task`, `_background_insert`, `add`, `add_done_callback`.
**CN:** 这一段延续了 `encode_with_global_cache` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `create_task`、`_background_insert`、`add`、`add_done_callback`。

### Lines 853-857: Method `_flatten_and_load_audios`
```python
    async def _flatten_and_load_audios(self, mm_items):
        """
        Flatten mm_items structure, load audios concurrently, and restore original structure.
        """
        return await self._flatten_and_load_data_by_modality(mm_items, Modality.AUDIO)
```
**EN:** This block defines the method `_flatten_and_load_audios` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_flatten_and_load_audios`. Notable operations include `_flatten_and_load_data_by_modality`.
**CN:** 这一段定义了method `_flatten_and_load_audios`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_flatten_and_load_audios`。 值得注意的操作包括 `_flatten_and_load_data_by_modality`。

### Lines 859-863: Method `_flatten_and_load_images`
```python
    async def _flatten_and_load_images(self, mm_items):
        """
        Flatten mm_items structure, load images concurrently, and restore original structure.
        """
        return await self._flatten_and_load_data_by_modality(mm_items, Modality.IMAGE)
```
**EN:** This block defines the method `_flatten_and_load_images` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_flatten_and_load_images`. Notable operations include `_flatten_and_load_data_by_modality`.
**CN:** 这一段定义了method `_flatten_and_load_images`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_flatten_and_load_images`。 值得注意的操作包括 `_flatten_and_load_data_by_modality`。

### Lines 865-881: Method `_calculate_timestamps`
```python
    def _calculate_timestamps(self, indices, video_fps: float, merge_size: int = 2):
        """Calculate timestamps for video frames, used for qwen3_vl models."""
        # refer to https://github.com/huggingface/transformers/blob/main/src/transformers/models/qwen3_vl/processing_qwen3_vl.py#L255
        if not isinstance(indices, list):
            indices = indices.tolist()
        if len(indices) % merge_size != 0:
            indices.extend(
                indices[-1] for _ in range(merge_size - len(indices) % merge_size)
            )
        timestamps = [idx / video_fps for idx in indices]
        # Frames are merged by merge_size, so we need to average the timestamps
        # between the first/last frame within the temporal patch
        timestamps = [
            (timestamps[i] + timestamps[i + merge_size - 1]) / 2
            for i in range(0, len(timestamps), merge_size)
        ]
        return timestamps
```
**EN:** This block defines the method `_calculate_timestamps` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_calculate_timestamps`. Notable operations include `tolist`, `extend`.
**CN:** 这一段定义了method `_calculate_timestamps`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_calculate_timestamps`。 值得注意的操作包括 `tolist`、`extend`。

### Lines 883-894: Method `_flatten_nested_items`
```python
    @staticmethod
    def _flatten_nested_items(items):
        if not isinstance(items, (list, tuple)):
            return [items]

        flat = []
        for item in items:
            if isinstance(item, (list, tuple)):
                flat.extend(MMEncoder._flatten_nested_items(item))
            else:
                flat.append(item)
        return flat
```
**EN:** This block defines the method `_flatten_nested_items` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_flatten_nested_items`. Notable operations include `extend`, `_flatten_nested_items`, `append`.
**CN:** 这一段定义了method `_flatten_nested_items`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_flatten_nested_items`。 值得注意的操作包括 `extend`、`_flatten_nested_items`、`append`。

### Lines 896-897: Method `_normalize_kimi_encoder_images` signature and setup
```python
    def _normalize_kimi_encoder_images(self, images):
        """Normalize Kimi image inputs for the image processor call."""
```
**EN:** This block defines the method `_normalize_kimi_encoder_images` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_normalize_kimi_encoder_images`.
**CN:** 这一段定义了method `_normalize_kimi_encoder_images`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_normalize_kimi_encoder_images`。

### Lines 898-921: Method `_normalize_kimi_encoder_images` logic (part 1)
```python
        from PIL import Image as PILImage

        def wrap_one(img):
            if isinstance(img, dict) and img.get("type") in ("image", "video_chunk"):
                return [img]
            if isinstance(img, PILImage.Image):
                return [{"type": "image", "image": img}]
            return [img]

        if not images:
            return images

        # Disagg may supply nested lists from grouped routing.
        images = self._flatten_nested_items(images)

        # Kimi-VL image processor expects a flat list of concrete images.
        if self.model_type == "kimi_vl":
            normalized = []
            for img in images:
                if (
                    isinstance(img, dict)
                    and img.get("type") == "image"
                    and "image" in img
                ):
```
**EN:** This block continues `_normalize_kimi_encoder_images` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Definitions introduced here include `wrap_one`. Notable operations include `get`, `in`, `_flatten_nested_items`.
**CN:** 这一段延续了 `_normalize_kimi_encoder_images` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 此处引入的定义包括 `wrap_one`。 值得注意的操作包括 `get`、`in`、`_flatten_nested_items`。

### Lines 922-946: Method `_normalize_kimi_encoder_images` logic (part 2)
```python
                    inner = img["image"]
                    if isinstance(inner, (list, tuple)):
                        normalized.extend(self._flatten_nested_items(inner))
                    else:
                        normalized.append(inner)
                else:
                    normalized.append(img)
            return normalized

        # Kimi-K2.5 vision processor expects media dicts.
        normalized = []
        for img in images:
            wrapped = wrap_one(img)
            for media in wrapped:
                # Some pipelines may produce {"type": "image", "image": [PIL]}.
                # Split it into one media item per concrete image object.
                if (
                    isinstance(media, dict)
                    and media.get("type") == "image"
                    and isinstance(media.get("image"), (list, tuple))
                ):
                    for inner in self._flatten_nested_items(media["image"]):
                        normalized.append({**media, "image": inner})
                else:
                    normalized.append(media)
```
**EN:** This block continues `_normalize_kimi_encoder_images` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `extend`, `_flatten_nested_items`, `append`, `wrap_one`.
**CN:** 这一段延续了 `_normalize_kimi_encoder_images` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `extend`、`_flatten_nested_items`、`append`、`wrap_one`。

### Lines 947-948: Method `_normalize_kimi_encoder_images` logic (part 3)
```python

        return normalized
```
**EN:** This block continues `_normalize_kimi_encoder_images` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow.
**CN:** 这一段延续了 `_normalize_kimi_encoder_images` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。

### Lines 950-970: Method `_process_mm_items`
```python
    async def _process_mm_items(self, mm_items, modality):
        model_preprocessor = getattr(self.model, "preprocess_mm_for_encoder", None)

        if modality == Modality.IMAGE:
            processor_input = await self._process_image_items(
                mm_items, model_preprocessor
            )
        elif modality == Modality.VIDEO:
            processor_input = await self._process_video_items(
                mm_items, model_preprocessor
            )
        elif modality == Modality.AUDIO:
            processor_input = await self._process_audio_items(
                mm_items, model_preprocessor
            )
        else:
            raise ValueError(f"Unsupported modality: {modality}")

        target = self.model.thinker if hasattr(self.model, "thinker") else self.model
        get_feature_method = getattr(target, f"get_{modality.name.lower()}_feature")
        return processor_input, get_feature_method
```
**EN:** This block defines the method `_process_mm_items` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_process_mm_items`. Notable operations include `_process_image_items`, `_process_video_items`, `_process_audio_items`, `ValueError`.
**CN:** 这一段定义了method `_process_mm_items`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_process_mm_items`。 值得注意的操作包括 `_process_image_items`、`_process_video_items`、`_process_audio_items`、`ValueError`。

### Lines 972-981: Method `_process_image_items`
```python
    async def _process_image_items(self, mm_items, model_preprocessor):
        if not (self.image_processor or model_preprocessor):
            raise ValueError("No image processor available")
        images = await self._flatten_and_load_images(mm_items)
        if model_preprocessor:
            return model_preprocessor(images, Modality.IMAGE, self.vision_config)
        image_config = self.vision_config.get("image", {})
        if self.model_type in ["kimi_k25", "kimi_vl"]:
            images = self._normalize_kimi_encoder_images(images)
        return self.image_processor(images=images, **image_config)
```
**EN:** This block defines the method `_process_image_items` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_process_image_items`. Notable operations include `not`, `ValueError`, `_flatten_and_load_images`, `model_preprocessor`.
**CN:** 这一段定义了method `_process_image_items`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_process_image_items`。 值得注意的操作包括 `not`、`ValueError`、`_flatten_and_load_images`、`model_preprocessor`。

### Lines 983-985: Method `_process_video_items` signature and setup
```python
    async def _process_video_items(self, mm_items, model_preprocessor):
        if model_preprocessor:
            return model_preprocessor(mm_items, Modality.VIDEO, self.vision_config)
```
**EN:** This block defines the method `_process_video_items` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_process_video_items`. Notable operations include `model_preprocessor`.
**CN:** 这一段定义了method `_process_video_items`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_process_video_items`。 值得注意的操作包括 `model_preprocessor`。

### Lines 986-1010: Method `_process_video_items` logic (part 1)
```python
        if not self.video_processor:
            raise ValueError("No video processor available")

        videos, video_processor_kwargs = await self._flatten_and_load_videos(mm_items)
        processor_input = self.video_processor(videos=videos, **video_processor_kwargs)

        # Get additional video metadata
        if (
            self.model_type
            in [
                "qwen3_vl",
                "qwen3_vl_moe",
                "qwen3_5",
                "qwen3_5_moe",
                "intern_s2_preview",
            ]
            and video_processor_kwargs.get("video_metadata", None) is not None
        ):
            video_metadata = video_processor_kwargs["video_metadata"]
            try:
                merge_size = (
                    self.model_config.hf_config.vision_config.spatial_merge_size
                )
            except (AttributeError, KeyError):
                merge_size = 2  # Default merge_size
```
**EN:** This block continues `_process_video_items` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `ValueError`, `_flatten_and_load_videos`, `video_processor`, `get`.
**CN:** 这一段延续了 `_process_video_items` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `ValueError`、`_flatten_and_load_videos`、`video_processor`、`get`。

### Lines 1011-1037: Method `_process_video_items` logic (part 2)
```python

            video_timestamps = []
            for metadata in video_metadata:
                video_fps = metadata.get("fps", None) or 24  # original video fps
                frames_indices = metadata.get("frames_indices", None)
                timestamps = self._calculate_timestamps(
                    frames_indices, video_fps, merge_size
                )
                video_timestamps.append(timestamps)
            processor_input["video_timestamps"] = video_timestamps
        elif (
            self.model_type in ["qwen2_5_vl", "qwen2_5_omni", "qwen3_omni_moe"]
            and processor_input.get("video_grid_thw", None) is not None
        ):
            video_grid_thw = processor_input["video_grid_thw"]
            try:
                temporal_patch_size = self.video_processor.temporal_patch_size
            except AttributeError:
                temporal_patch_size = 2  # Default temporal_patch_size
            fps_list = [
                self.vision_config.get("video", {}).get("fps", None) or 2
            ] * len(video_grid_thw)
            second_per_grid_ts = [(temporal_patch_size / fps) for fps in fps_list]
            second_per_grid_ts_tensor = torch.tensor(
                second_per_grid_ts, dtype=torch.float32
            )
            processor_input["second_per_grid_ts"] = second_per_grid_ts_tensor
```
**EN:** This block continues `_process_video_items` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `get`, `_calculate_timestamps`, `append`, `elif`.
**CN:** 这一段延续了 `_process_video_items` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `get`、`_calculate_timestamps`、`append`、`elif`。

### Lines 1038-1039: Method `_process_video_items` logic (part 3)
```python

        return processor_input
```
**EN:** This block continues `_process_video_items` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow.
**CN:** 这一段延续了 `_process_video_items` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。

### Lines 1041-1060: Method `_process_audio_items`
```python
    async def _process_audio_items(self, mm_items, model_preprocessor):
        if model_preprocessor:
            return model_preprocessor(mm_items, Modality.AUDIO, self.vision_config)
        if not self.audio_processor:
            raise ValueError("No audio processor available")

        audios = await self._flatten_and_load_audios(mm_items)
        audio_config = self.vision_config.get("audio", {})
        processor_input = self.audio_processor.feature_extractor(audios, **audio_config)
        processor_input["feature_attention_mask"] = processor_input.pop(
            "attention_mask"
        )
        # convert to same format as image/video
        input_lengths = torch.tensor(
            processor_input["feature_attention_mask"].sum(-1), dtype=torch.long
        )
        processor_input["audio_feature_lens_raw"] = input_lengths
        output_lengths = self._get_feat_extract_output_lengths(input_lengths)
        processor_input["audio_feature_lens"] = output_lengths
        return processor_input
```
**EN:** This block defines the method `_process_audio_items` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_process_audio_items`. Notable operations include `model_preprocessor`, `ValueError`, `_flatten_and_load_audios`, `get`.
**CN:** 这一段定义了method `_process_audio_items`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_process_audio_items`。 值得注意的操作包括 `model_preprocessor`、`ValueError`、`_flatten_and_load_audios`、`get`。

### Lines 1062-1064: Method `_encode` signature and setup
```python
    async def _encode(self, mm_items, modality: Modality) -> torch.Tensor:
        try:
            mm_inputs, get_feature_fn = await self._process_mm_items(mm_items, modality)
```
**EN:** This block defines the method `_encode` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_encode`. Notable operations include `_process_mm_items`.
**CN:** 这一段定义了method `_encode`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_encode`。 值得注意的操作包括 `_process_mm_items`。

### Lines 1065-1083: Method `_encode` logic (part 1)
```python
        except NotImplementedError as e:
            raise InternalError(f"Not implemented error: {str(e)}")
        except Exception as e:
            raise BadRequestError(f"Failed to process mm items: {str(e)}")
        try:
            # support mm_cache
            mm_embedding = None
            mm_hash = None

            mm_item = MultimodalDataItem.from_dict(
                {
                    "modality": modality,
                    "feature": _convert(_get_mm_feature(mm_inputs, modality)),
                }
            )
            for k, v in mm_inputs.items():
                if k in _mm_feature_attrs[modality]:
                    continue
                mm_item.set(k, _convert(v))
```
**EN:** This block continues `_encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `InternalError`, `BadRequestError`, `from_dict`, `_convert`.
**CN:** 这一段延续了 `_encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `InternalError`、`BadRequestError`、`from_dict`、`_convert`。

### Lines 1084-1106: Method `_encode` logic (part 2)
```python

            if self.server_args.enable_prefix_mm_cache:
                mm_item.set_pad_value()
                mm_hash = MultiModalStaticCache.combine_hashes([mm_item.hash])
                async with self.mm_cache_lock:
                    mm_cache = self.mm_cache.get([mm_item.hash])
                    if mm_cache is not None:
                        mm_embedding = mm_cache.embedding

            if mm_embedding is None:
                with torch.inference_mode():
                    mm_embedding: torch.Tensor = get_feature_fn([mm_item])
                    mm_embedding = mm_embedding.cpu()
                if len(mm_embedding.shape) != 2:
                    mm_embedding = mm_embedding.reshape(-1, mm_embedding.shape[-1])

            if self.server_args.enable_prefix_mm_cache:
                async with self.mm_cache_lock:
                    self.mm_cache.set(mm_hash, EmbeddingResult(embedding=mm_embedding))
            if self.profiler is not None:
                self.profiler.step()

            aux_data = _build_mm_aux_data(mm_inputs, self.model_type)
```
**EN:** This block continues `_encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `set_pad_value`, `combine_hashes`, `get`, `inference_mode`.
**CN:** 这一段延续了 `_encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `set_pad_value`、`combine_hashes`、`get`、`inference_mode`。

### Lines 1107-1121: Method `_encode` logic (part 3)
```python

            if modality == Modality.VIDEO and mm_inputs.get("video_audio_features"):
                target = (
                    self.model.thinker if hasattr(self.model, "thinker") else self.model
                )
                encode_video_audio_fn = getattr(target, "encode_video_audio", None)
                if encode_video_audio_fn is not None:
                    audio_embedding = encode_video_audio_fn(mm_inputs)
                    if audio_embedding is not None:
                        aux_data["video_audio_embedding"] = audio_embedding
                else:
                    logger.warning(
                        "Videos carry audio tracks but model has no "
                        "encode_video_audio; dropping audio for EPD encoding."
                    )
```
**EN:** This block continues `_encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `get`, `encode_video_audio_fn`, `warning`.
**CN:** 这一段延续了 `_encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `get`、`encode_video_audio_fn`、`warning`。

### Lines 1122-1131: Method `_encode` logic (part 4)
```python

            return (
                _get_mm_grid_dim(mm_inputs, modality, self.model_type),
                mm_embedding,
                aux_data,
            )
        except BadRequestError as e:
            raise BadRequestError(f"Bad request error: {str(e)}")
        except Exception as e:
            raise InternalError(f"Internal encoding error: {str(e)}")
```
**EN:** This block continues `_encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `_get_mm_grid_dim`, `BadRequestError`, `InternalError`.
**CN:** 这一段延续了 `_encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `_get_mm_grid_dim`、`BadRequestError`、`InternalError`。

### Lines 1133-1135: Method `_send` signature and setup
```python
    async def _send(
        self,
        embedding: torch.Tensor,
```
**EN:** This block defines the method `_send` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `_send`.
**CN:** 这一段定义了method `_send`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `_send`。

### Lines 1136-1157: Method `_send` logic (part 1)
```python
        mm_data: EmbeddingData,
        session_id=None,
        buffer_address=None,
        prefill_host=None,
        embedding_port=None,
        url=None,
    ):
        if self.server_args.encoder_transfer_backend == "mooncake":
            self.engine.register(embedding.data_ptr(), embedding.nbytes)
            self.engine.transfer_sync(
                session_id, embedding.data_ptr(), buffer_address, embedding.nbytes
            )
            self.engine.deregister(embedding.data_ptr())

            mm_data.embedding = None

        # Send ack/data
        if url is not None:
            endpoint = NetworkAddress.parse(url).to_tcp()
        else:
            endpoint = NetworkAddress(prefill_host, embedding_port).to_tcp()
        logger.info(f"{endpoint = }")
```
**EN:** This block continues `_send` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `register`, `data_ptr`, `transfer_sync`, `deregister`.
**CN:** 这一段延续了 `_send` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `register`、`data_ptr`、`transfer_sync`、`deregister`。

### Lines 1158-1184: Method `_send` logic (part 2)
```python

        # Serialize data
        if self.server_args.encoder_transfer_backend == "mooncake":
            serialized_data = pickle.dumps(mm_data)
            buffer = None
        else:
            new_mm_data = mm_data.copy_without_embedding()
            if new_mm_data.error_msg is not None:
                buffer = None
                serialized_data = pickle.dumps(new_mm_data)
            else:
                embedding_tensor = TensorWrapper(mm_data.embedding)
                serialized_data = pickle.dumps(new_mm_data)
                buffer = embedding_tensor.__buffer__()

        # Use thread pool executor for parallel ZMQ send operations
        def send_with_socket():
            sock = self.sync_context.socket(zmq.PUSH)
            config_socket(sock, zmq.PUSH)
            try:
                sock.connect(endpoint)
                if buffer is not None:
                    sock.send_multipart([serialized_data, buffer], copy=False)
                else:
                    sock.send_multipart([serialized_data], copy=False)
            finally:
                sock.close()
```
**EN:** This block continues `_send` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Definitions introduced here include `send_with_socket`. Notable operations include `dumps`, `copy_without_embedding`, `TensorWrapper`, `__buffer__`.
**CN:** 这一段延续了 `_send` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 此处引入的定义包括 `send_with_socket`。 值得注意的操作包括 `dumps`、`copy_without_embedding`、`TensorWrapper`、`__buffer__`。

### Lines 1185-1186: Method `_send` logic (part 3)
```python

        await asyncio.get_event_loop().run_in_executor(self.executor, send_with_socket)
```
**EN:** This block continues `_send` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `get_event_loop`, `run_in_executor`.
**CN:** 这一段延续了 `_send` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `get_event_loop`、`run_in_executor`。

### Lines 1188-1190: Method `encode` signature and setup
```python
    async def encode(self, mm_items, modality: Modality, req_id, num_parts, part_idx):
        try:
            grid_dim, mm_embedding, aux_data = await self._encode(mm_items, modality)
```
**EN:** This block defines the method `encode` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `encode`. Notable operations include `_encode`.
**CN:** 这一段定义了method `encode`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `encode`。 值得注意的操作包括 `_encode`。

### Lines 1191-1214: Method `encode` logic (part 1)
```python

            if self.rank == 0:
                mm_data = EmbeddingData(
                    req_id,
                    num_parts,
                    part_idx,
                    grid_dim,
                    modality,
                    mm_embedding,
                    **aux_data,
                )
                self.embedding_to_send[req_id] = mm_data
            return (
                mm_embedding.nbytes,
                mm_embedding.shape[0],
                mm_embedding.shape[1],
                None,
                None,
            )
        except Exception as e:
            error_code = getattr(e, "code", HTTPStatus.INTERNAL_SERVER_ERROR)
            error_msg = str(e)
            logger.error(f"Rank {self.rank} encode failed: {error_msg} {error_code = }")
            if self.rank == 0:
```
**EN:** This block continues `encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `EmbeddingData`, `error`.
**CN:** 这一段延续了 `encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `EmbeddingData`、`error`。

### Lines 1215-1226: Method `encode` logic (part 2)
```python
                mm_data = EmbeddingData(
                    req_id,
                    num_parts,
                    part_idx,
                    None,
                    modality,
                    error_msg=error_msg,
                    error_code=error_code,
                )
                self.embedding_to_send[req_id] = mm_data
                logger.debug(f"Created error EmbeddingData: {mm_data}")
            return 0, 0, 0, error_msg, error_code
```
**EN:** This block continues `encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `EmbeddingData`, `debug`.
**CN:** 这一段延续了 `encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `EmbeddingData`、`debug`。

### Lines 1227-1228: Supporting state inside `MMEncoder`
```python

    # For zmq_to_tokenizer zmq_to_scheduler and mooncake
```
**EN:** This block adds supporting state or helper logic inside `MMEncoder`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `MMEncoder` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 1229-1240: Method `send`
```python
    async def send(
        self, req_id, prefill_host, embedding_port, session_id=None, buffer_address=None
    ):
        mm_data: EmbeddingData = self.embedding_to_send[req_id]
        await self._send(
            mm_data.embedding,
            mm_data,
            session_id=session_id,
            buffer_address=buffer_address,
            prefill_host=prefill_host,
            embedding_port=embedding_port,
        )
```
**EN:** This block defines the method `send` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `send`. Notable operations include `_send`.
**CN:** 这一段定义了method `send`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `send`。 值得注意的操作包括 `_send`。

### Lines 1241-1242: Supporting state inside `MMEncoder`
```python

    # For zmq_to_scheduler
```
**EN:** This block adds supporting state or helper logic inside `MMEncoder`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `MMEncoder` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 1243-1245: Method `send_with_url` signature and setup
```python
    async def send_with_url(
        self,
        req_id,
```
**EN:** This block defines the method `send_with_url` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `send_with_url`.
**CN:** 这一段定义了method `send_with_url`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `send_with_url`。

### Lines 1246-1262: Method `send_with_url` logic (part 1)
```python
    ):
        mm_data = self.embedding_to_send.get(req_id)
        if not mm_data:
            return
        sent_urls: Set[str] = set()
        all_tasks: List[Tuple[asyncio.Task, str]] = []
        start_time = asyncio.get_running_loop().time()
        timeout = self.send_timeout
        cond = await get_condition(req_id)

        try:
            while True:
                async with rid_lock:
                    current_targets = rid_to_receive_endpoint.get(req_id, set()).copy()
                    expected_count = rid_to_receive_count.get(req_id)

                new_targets = current_targets - sent_urls
```
**EN:** This block continues `send_with_url` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `get`, `get_running_loop`, `time`, `get_condition`.
**CN:** 这一段延续了 `send_with_url` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `get`、`get_running_loop`、`time`、`get_condition`。

### Lines 1263-1288: Method `send_with_url` logic (part 2)
```python

                if new_targets:
                    logger.info(
                        f"Found {len(new_targets)} new endpoints for {req_id}. Starting tasks..."
                    )
                    for url in new_targets:
                        task = asyncio.create_task(
                            self._send(
                                mm_data.embedding,
                                mm_data,
                                url=url,
                            )
                        )
                        all_tasks.append((task, url))
                        sent_urls.add(url)  # Mark as handled immediately
                if expected_count is not None and len(sent_urls) >= expected_count:
                    logger.info(
                        f"All {expected_count} endpoints initiated for {req_id}. Breaking loop."
                    )
                    break
                remaining = timeout - (asyncio.get_running_loop().time() - start_time)
                if remaining <= 0:
                    logger.error(
                        f"[{req_id}] Timeout! Sent {len(sent_urls)}/{expected_count}"
                    )
                    break
```
**EN:** This block continues `send_with_url` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `info`, `create_task`, `_send`, `append`.
**CN:** 这一段延续了 `send_with_url` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `info`、`create_task`、`_send`、`append`。

### Lines 1289-1311: Method `send_with_url` logic (part 3)
```python

                async with cond:
                    try:
                        await asyncio.wait_for(cond.wait(), timeout=remaining)
                    except asyncio.TimeoutError:
                        continue

            if all_tasks:
                logger.info(
                    f"Loop finished. Awaiting completion of {len(all_tasks)} sending tasks..."
                )
                tasks_only = [t[0] for t in all_tasks]
                results = await asyncio.gather(*tasks_only, return_exceptions=True)

                # Process results and log errors
                for i, result in enumerate(results):
                    url = all_tasks[i][1]  # Retrieve URL associated with the task
                    if isinstance(result, Exception):
                        logger.error(f"Failed to send to {url}: {result}")
                    else:
                        logger.debug(f"Successfully sent to {url}")

            logger.info(f"All tasks completed for req_id: {req_id}")
```
**EN:** This block continues `send_with_url` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `wait_for`, `wait`, `info`, `gather`.
**CN:** 这一段延续了 `send_with_url` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `wait_for`、`wait`、`info`、`gather`。

### Lines 1312-1320: Method `send_with_url` logic (part 4)
```python

        finally:
            logger.info(f"Cleaning up resources for req_id {req_id}")
            async with rid_lock:
                rid_to_receive_endpoint.pop(req_id, None)
                rid_to_receive_count.pop(req_id, None)
            async with cond_dict_lock:
                rid_to_cond.pop(req_id, None)
            self.embedding_to_send.pop(req_id, None)
```
**EN:** This block continues `send_with_url` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `info`, `pop`.
**CN:** 这一段延续了 `send_with_url` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `info`、`pop`。

### Lines 1322-1331: Method `get_embedding_port`
```python
    async def get_embedding_port(self, prefill_url):
        async with aiohttp.ClientSession(
            timeout=aiohttp.ClientTimeout(total=1800)
        ) as session:
            response = await session.post(
                f"{prefill_url}/embedding_bootstrap",
                json={"embedding_port": None},
            )
            response_json = await response.json()
            return response_json["embedding_port"]
```
**EN:** This block defines the method `get_embedding_port` on `MMEncoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `get_embedding_port`. Notable operations include `ClientSession`, `ClientTimeout`, `post`, `json`.
**CN:** 这一段定义了method `get_embedding_port`（属于 `MMEncoder`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `get_embedding_port`。 值得注意的操作包括 `ClientSession`、`ClientTimeout`、`post`、`json`。

### Lines 1334-1334: Class `EncoderProfiler` declaration
```python
class EncoderProfiler:
```
**EN:** This block declares the class `EncoderProfiler` and establishes its responsibility inside encode-side transfer server orchestration. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `EncoderProfiler`.
**CN:** 这一段声明了类 `EncoderProfiler`，并说明它在编码侧传输服务编排中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `EncoderProfiler`。

### Lines 1335-1341: Method `__init__`
```python
    def __init__(self, rank: int):
        self.rank = rank
        self.profiler = None
        self.steps_left = None
        self.output_dir = None
        self.prefix = None
        self.profile_id = None
```
**EN:** This block defines the method `__init__` on `EncoderProfiler`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `EncoderProfiler`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 1343-1345: Method `start` signature and setup
```python
    def start(self, obj: ProfileReq):
        if self.profiler is not None:
            return False, "profiling already running"
```
**EN:** This block defines the method `start` on `EncoderProfiler`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `start`.
**CN:** 这一段定义了method `start`（属于 `EncoderProfiler`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `start`。

### Lines 1346-1362: Method `start` logic (part 1)
```python

        output_dir = obj.output_dir or os.getenv("SGLANG_TORCH_PROFILER_DIR", "/tmp")
        os.makedirs(output_dir, exist_ok=True)
        self.output_dir = output_dir
        self.prefix = obj.profile_prefix or "encoder"
        self.profile_id = str(time.time())

        activities = obj.activities or ["CPU", "GPU"]
        torch_activities = []
        if "CPU" in activities:
            torch_activities.append(torch.profiler.ProfilerActivity.CPU)
        if "GPU" in activities:
            torch_activities.append(torch.profiler.ProfilerActivity.CUDA)

        profile_memory = "MEM" in activities
        if not torch_activities and not profile_memory:
            return False, "no supported activities"
```
**EN:** This block continues `start` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `getenv`, `makedirs`, `time`, `append`.
**CN:** 这一段延续了 `start` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `getenv`、`makedirs`、`time`、`append`。

### Lines 1363-1375: Method `start` logic (part 2)
```python

        self.profiler = torch.profiler.profile(
            activities=torch_activities,
            with_stack=True if obj.with_stack is None else obj.with_stack,
            record_shapes=False if obj.record_shapes is None else obj.record_shapes,
            profile_memory=profile_memory,
        )
        self.profiler.start()
        self.steps_left = obj.num_steps
        logger.info(
            f"Encoder profiling started. output_dir={self.output_dir} profile_id={self.profile_id}"
        )
        return True, None
```
**EN:** This block continues `start` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `profile`, `start`, `info`.
**CN:** 这一段延续了 `start` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `profile`、`start`、`info`。

### Lines 1377-1384: Method `step`
```python
    def step(self):
        if self.profiler is None:
            return
        self.profiler.step()
        if self.steps_left is not None:
            self.steps_left -= 1
            if self.steps_left <= 0:
                self.stop()
```
**EN:** This block defines the method `step` on `EncoderProfiler`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `step`. Notable operations include `step`, `stop`.
**CN:** 这一段定义了method `step`（属于 `EncoderProfiler`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `step`。 值得注意的操作包括 `step`、`stop`。

### Lines 1386-1396: Method `stop`
```python
    def stop(self):
        if self.profiler is None:
            return False, "profiling not running"
        self.profiler.stop()
        filename = f"{self.prefix}-rank{self.rank}-{self.profile_id}.trace.json"
        trace_path = os.path.join(self.output_dir, filename)
        self.profiler.export_chrome_trace(trace_path)
        logger.info("Encoder profiling saved to: %s", trace_path)
        self.profiler = None
        self.steps_left = None
        return True, None
```
**EN:** This block defines the method `stop` on `EncoderProfiler`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `stop`. Notable operations include `stop`, `join`, `export_chrome_trace`, `info`.
**CN:** 这一段定义了method `stop`（属于 `EncoderProfiler`），介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `stop`。 值得注意的操作包括 `stop`、`join`、`export_chrome_trace`、`info`。

### Lines 1397-1403: Module-level constants and helper logic
```python


app = FastAPI()
encoder: Optional[MMEncoder] = None
send_sockets: List[zmq.Socket] = []


```
**EN:** This block contains module-level constants, helpers, or documentation for encode-side transfer server orchestration. It prepares shared state that later classes and functions build on. Notable operations include `FastAPI`.
**CN:** 这一段包含与编码侧传输服务编排相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `FastAPI`。

### Lines 1404-1406: Async Function `run_encoder` signature and setup
```python
async def run_encoder(
    server_args: ServerArgs, schedule_path, dist_init_method, rank: int
):
```
**EN:** This block defines the async function `run_encoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `run_encoder`.
**CN:** 这一段定义了async function `run_encoder`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `run_encoder`。

### Lines 1407-1430: Async Function `run_encoder` logic (part 1)
```python
    encoder = MMEncoder(server_args, schedule_path, dist_init_method, rank)
    while True:
        request = await encoder.schedule_socket.recv_pyobj()
        if isinstance(request, ProfileReq):
            if request.type == ProfileReqType.START_PROFILE:
                if encoder.profiler is None:
                    encoder.profiler = EncoderProfiler(encoder.rank)
                encoder.profiler.start(request)
            else:
                encoder.profiler.stop()
        else:
            if encoder.mm_global_cache is not None:
                await encoder.encode_with_global_cache(
                    mm_items=request["mm_items"],
                    modality=Modality.from_str(request["modality"]),
                    req_id=request["req_id"],
                    num_parts=request["num_parts"],
                    part_idx=request["part_idx"],
                    hashes=request.get("hashes", None),
                )
            else:
                await encoder.encode(
                    mm_items=request["mm_items"],
                    modality=Modality.from_str(request["modality"]),
```
**EN:** This block continues `run_encoder` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `MMEncoder`, `recv_pyobj`, `EncoderProfiler`, `start`.
**CN:** 这一段延续了 `run_encoder` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `MMEncoder`、`recv_pyobj`、`EncoderProfiler`、`start`。

### Lines 1431-1434: Async Function `run_encoder` logic (part 2)
```python
                    req_id=request["req_id"],
                    num_parts=request["num_parts"],
                    part_idx=request["part_idx"],
                )
```
**EN:** This block continues `run_encoder` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow.
**CN:** 这一段延续了 `run_encoder` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。

### Lines 1437-1443: Function `launch_encoder`
```python
def launch_encoder(server_args, schedule_path, dist_init_method, rank):
    try:
        asyncio.run(run_encoder(server_args, schedule_path, dist_init_method, rank))
    except KeyboardInterrupt:
        logger.info(f"Exit rank {rank}")
    except Exception:
        traceback.print_exc()
```
**EN:** This block defines the function `launch_encoder`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `launch_encoder`. Notable operations include `run`, `run_encoder`, `info`, `print_exc`.
**CN:** 这一段定义了function `launch_encoder`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `launch_encoder`。 值得注意的操作包括 `run`、`run_encoder`、`info`、`print_exc`。

### Lines 1446-1448: Function `launch_server` signature and setup
```python
def launch_server(server_args: ServerArgs):
    configure_logger(server_args, prefix=" encode_server")
    global encoder
```
**EN:** This block defines the function `launch_server`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `launch_server`. Notable operations include `configure_logger`.
**CN:** 这一段定义了function `launch_server`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `launch_server`。 值得注意的操作包括 `configure_logger`。

### Lines 1449-1471: Function `launch_server` logic (part 1)
```python
    ctx = mp.get_context("spawn")
    zmq_ctx = zmq.Context(10)
    ipc_path_prefix = random_uuid()
    port_args = PortArgs.init_new(server_args)
    if server_args.dist_init_addr:
        na = NetworkAddress.parse(server_args.dist_init_addr)
        dist_init_method = na.to_tcp()
    else:
        dist_init_method = NetworkAddress(
            server_args.host or "127.0.0.1", port_args.nccl_port
        ).to_tcp()
    for rank in range(1, server_args.tp_size):
        schedule_path = f"ipc:///tmp/{ipc_path_prefix}_schedule_{rank}"
        send_sockets.append(
            get_zmq_socket(zmq_ctx, zmq.PUSH, schedule_path, bind=False)
        )
        ctx.Process(
            target=launch_encoder,
            args=(server_args, schedule_path, dist_init_method, rank),
            daemon=True,
        ).start()
    encoder = MMEncoder(server_args, dist_init_method=dist_init_method)
    uvicorn.run(app, host=server_args.host, port=server_args.port)
```
**EN:** This block continues `launch_server` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `get_context`, `Context`, `random_uuid`, `init_new`.
**CN:** 这一段延续了 `launch_server` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `get_context`、`Context`、`random_uuid`、`init_new`。

### Lines 1474-1478: Async Function `get_condition`
```python
async def get_condition(rid):
    async with cond_dict_lock:
        if rid not in rid_to_cond:
            rid_to_cond[rid] = asyncio.Condition()
        return rid_to_cond[rid]
```
**EN:** This block defines the async function `get_condition`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `get_condition`. Notable operations include `Condition`.
**CN:** 这一段定义了async function `get_condition`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `get_condition`。 值得注意的操作包括 `Condition`。

### Lines 1481-1483: Async Function `handle_encode_request` signature and setup
```python
@app.post("/encode")
async def handle_encode_request(request: dict):
    req_id = request["req_id"]
```
**EN:** This block defines the async function `handle_encode_request`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `handle_encode_request`. Notable operations include `post`.
**CN:** 这一段定义了async function `handle_encode_request`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `handle_encode_request`。 值得注意的操作包括 `post`。

### Lines 1484-1516: Async Function `handle_encode_request` logic (part 1)
```python
    start_time = time.monotonic()
    try:

        def start_background_send(req_id):
            task = asyncio.create_task(encoder.send_with_url(req_id=req_id))
            encoder.background_tasks.add(task)
            task.add_done_callback(encoder.background_tasks.discard)

        # broadcast request
        request.update({"enter_time": time.time()})
        for socket in send_sockets:
            socket.send_pyobj(request)
        if encoder.mm_global_cache is not None:
            nbytes, embedding_len, embedding_dim, error_msg, error_code = (
                await encoder.encode_with_global_cache(
                    mm_items=request["mm_items"],
                    modality=Modality.from_str(request["modality"]),
                    req_id=request["req_id"],
                    num_parts=request["num_parts"],
                    part_idx=request["part_idx"],
                    hashes=request.get("hashes", None),
                )
            )
        else:
            nbytes, embedding_len, embedding_dim, error_msg, error_code = (
                await encoder.encode(
                    mm_items=request["mm_items"],
                    modality=Modality.from_str(request["modality"]),
                    req_id=request["req_id"],
                    num_parts=request["num_parts"],
                    part_idx=request["part_idx"],
                )
            )
```
**EN:** This block continues `handle_encode_request` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Definitions introduced here include `start_background_send`. Notable operations include `monotonic`, `create_task`, `send_with_url`, `add`.
**CN:** 这一段延续了 `handle_encode_request` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 此处引入的定义包括 `start_background_send`。 值得注意的操作包括 `monotonic`、`create_task`、`send_with_url`、`add`。

### Lines 1517-1548: Async Function `handle_encode_request` logic (part 2)
```python

        if error_msg:
            if encoder.server_args.encoder_transfer_backend == "zmq_to_scheduler":
                if request["embedding_port"] is None:
                    start_background_send(req_id)
                else:
                    for port in request["embedding_port"]:
                        await encoder.send(
                            req_id=req_id,
                            prefill_host=request["prefill_host"],
                            embedding_port=port,
                        )
            return ORJSONResponse(
                status_code=error_code,
                content={"status": "error", "message": error_msg, "req_id": req_id},
            )
        if encoder.server_args.encoder_transfer_backend == "mooncake":
            del request["mm_items"]
            request.update(
                {
                    "embedding_size": nbytes,
                    "embedding_len": embedding_len,
                    "embedding_dim": embedding_dim,
                }
            )
            return ORJSONResponse(content=request)
        elif encoder.server_args.encoder_transfer_backend == "zmq_to_scheduler":
            logger.info(f"{request['embedding_port'] = }")
            if request["embedding_port"] is None:
                await encoder.send_with_url(
                    req_id=request["req_id"],
                )
```
**EN:** This block continues `handle_encode_request` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `start_background_send`, `send`, `ORJSONResponse`, `update`.
**CN:** 这一段延续了 `handle_encode_request` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `start_background_send`、`send`、`ORJSONResponse`、`update`。

### Lines 1549-1580: Async Function `handle_encode_request` logic (part 3)
```python
            else:
                assert type(request["embedding_port"]) == list
                tasks = []
                for embedding_port in request["embedding_port"]:
                    tasks.append(
                        encoder.send(
                            req_id=request["req_id"],
                            prefill_host=request["prefill_host"],
                            embedding_port=embedding_port,
                        )
                    )
                await asyncio.gather(*tasks)
                encoder.embedding_to_send.pop(request["req_id"], None)
            return ORJSONResponse(content=None)
        elif encoder.server_args.encoder_transfer_backend == "zmq_to_tokenizer":
            await encoder.send(
                req_id=request["req_id"],
                prefill_host=request["prefill_host"],
                embedding_port=request["embedding_port"],
            )
            encoder.embedding_to_send.pop(request["req_id"], None)
            elapsed = time.monotonic() - start_time
            logger.info(
                f"[{req_id}] /encode completed in {elapsed:.3f}s, "
                f"modality={request['modality']}, tokens={embedding_len}"
            )
            return ORJSONResponse(content=None)
    except Exception as e:
        error_msg = str(e)
        logger.error(f"Unexpected error in encoder logic for {req_id}: {error_msg}")
        rid_to_err_msg[req_id] = error_msg
        return ORJSONResponse(
```
**EN:** This block continues `handle_encode_request` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `append`, `send`, `gather`, `pop`.
**CN:** 这一段延续了 `handle_encode_request` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `append`、`send`、`gather`、`pop`。

### Lines 1581-1587: Async Function `handle_encode_request` logic (part 4)
```python
            status_code=HTTPStatus.INTERNAL_SERVER_ERROR,
            content={
                "status": "error",
                "message": error_msg,
                "req_id": req_id,
            },
        )
```
**EN:** This block continues `handle_encode_request` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow.
**CN:** 这一段延续了 `handle_encode_request` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。

### Lines 1590-1601: Async Function `handle_send_request`
```python
@app.post("/send")
async def handle_send_request(request: dict):
    # mooncake backend
    await encoder.send(
        req_id=request["req_id"],
        prefill_host=request["prefill_host"],
        embedding_port=request["embedding_port"],
        session_id=request["session_id"],
        buffer_address=request["buffer_address"],
    )
    encoder.embedding_to_send.pop(request["req_id"], None)
    return ORJSONResponse(content=None)
```
**EN:** This block defines the async function `handle_send_request`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `handle_send_request`. Notable operations include `post`, `send`, `pop`, `ORJSONResponse`.
**CN:** 这一段定义了async function `handle_send_request`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `handle_send_request`。 值得注意的操作包括 `post`、`send`、`pop`、`ORJSONResponse`。

### Lines 1604-1616: Async Function `handle_scheduler_receive_url_request`
```python
@app.post("/scheduler_receive_url")
async def handle_scheduler_receive_url_request(request: dict):
    rid = request["req_id"]
    async with rid_lock:
        global rid_to_receive_endpoint
        if rid not in rid_to_receive_endpoint:
            rid_to_receive_endpoint[rid] = set()
            rid_to_receive_count[rid] = request["receive_count"]
        assert rid_to_receive_count[rid] == request["receive_count"]
        rid_to_receive_endpoint[rid].add(request["receive_url"])
    cond = await get_condition(rid)
    async with cond:
        cond.notify_all()
```
**EN:** This block defines the async function `handle_scheduler_receive_url_request`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `handle_scheduler_receive_url_request`. Notable operations include `post`, `add`, `get_condition`, `notify_all`.
**CN:** 这一段定义了async function `handle_scheduler_receive_url_request`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `handle_scheduler_receive_url_request`。 值得注意的操作包括 `post`、`add`、`get_condition`、`notify_all`。

### Lines 1619-1626: Async Function `health_generate` signature and setup
```python
@app.get("/health")
@app.get("/health_generate")
async def health_generate():
    """
    Health check endpoint for the encoder server.
    Performs a dummy encode to verify the encoder is functional.
    Returns 200 if the encoder is healthy, 503 otherwise.
    """
```
**EN:** This block defines the async function `health_generate`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `health_generate`. Notable operations include `get`.
**CN:** 这一段定义了async function `health_generate`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `health_generate`。 值得注意的操作包括 `get`。

### Lines 1627-1648: Async Function `health_generate` logic (part 1)
```python
    if encoder is None:
        return Response(status_code=503)

    # Skip the dummy encode when real requests are already in flight — the
    # ongoing traffic already proves liveness, matching the scheduler's
    # `is_fully_idle`-based health-check skip pattern.
    if encoder.embedding_to_send:
        return Response(status_code=200)

    # Pick the first available modality for the dummy encode
    if encoder.image_processor is not None:
        mm_items = [f"data:image/png;base64,{MINIMUM_PNG_PICTURE_BASE64}"]
        modality = Modality.IMAGE
    elif encoder.audio_processor is not None:
        mm_items = [f"data:audio/wav;base64,{MINIMUM_WAV_SILENCE_BASE64}"]
        modality = Modality.AUDIO
    else:
        # No processor available, fall back to liveness check only
        return Response(status_code=200)

    try:
        req_id = f"{HEALTH_CHECK_RID_PREFIX}_{time.time()}"
```
**EN:** This block continues `health_generate` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `Response`, `time`.
**CN:** 这一段延续了 `health_generate` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `Response`、`time`。

### Lines 1649-1672: Async Function `health_generate` logic (part 2)
```python

        dummy_request = {
            "mm_items": mm_items,
            "modality": modality.name,
            "req_id": req_id,
            "num_parts": 1,
            "part_idx": 0,
        }

        # Broadcast to other TP ranks so distributed ops stay in sync
        for socket in send_sockets:
            socket.send_pyobj(dummy_request)

        # Run encode on rank 0 with timeout
        _, _, _, error_msg, _ = await asyncio.wait_for(
            encoder.encode(
                mm_items=mm_items,
                modality=modality,
                req_id=req_id,
                num_parts=1,
                part_idx=0,
            ),
            timeout=HEALTH_CHECK_TIMEOUT,
        )
```
**EN:** This block continues `health_generate` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `send_pyobj`, `wait_for`, `encode`.
**CN:** 这一段延续了 `health_generate` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `send_pyobj`、`wait_for`、`encode`。

### Lines 1673-1688: Async Function `health_generate` logic (part 3)
```python

        # Clean up stored embedding
        encoder.embedding_to_send.pop(req_id, None)

        if error_msg:
            logger.error(f"Encoder health check failed: {error_msg}")
            return Response(status_code=503)

        return Response(status_code=200)

    except asyncio.TimeoutError:
        logger.error(f"Encoder health check timed out after {HEALTH_CHECK_TIMEOUT}s")
        return Response(status_code=503)
    except Exception as e:
        logger.error(f"Encoder health check failed: {e}")
        return Response(status_code=503)
```
**EN:** This block continues `health_generate` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `pop`, `error`, `Response`.
**CN:** 这一段延续了 `health_generate` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `pop`、`error`、`Response`。

### Lines 1691-1693: Async Function `start_profile_async` signature and setup
```python
@app.api_route("/start_profile", methods=["GET", "POST"])
async def start_profile_async(obj: Optional[ProfileReqInput] = None):
    if encoder is None:
```
**EN:** This block defines the async function `start_profile_async`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `start_profile_async`. Notable operations include `api_route`.
**CN:** 这一段定义了async function `start_profile_async`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `start_profile_async`。 值得注意的操作包括 `api_route`。

### Lines 1694-1717: Async Function `start_profile_async` logic (part 1)
```python
        return Response(content="encoder not ready\n", status_code=503)
    req = None
    if obj is None:
        req = ProfileReq(ProfileReqType.START_PROFILE)
    else:
        req = ProfileReq(
            type=ProfileReqType.START_PROFILE,
            output_dir=obj.output_dir,
            start_step=obj.start_step,
            num_steps=obj.num_steps,
            activities=obj.activities,
            with_stack=obj.with_stack,
            record_shapes=obj.record_shapes,
            profile_by_stage=obj.profile_by_stage,
            profile_id=str(time.time()),
            merge_profiles=obj.merge_profiles,
            profile_prefix=obj.profile_prefix,
            profile_stages=obj.profile_stages,
        )
    for socket in send_sockets:
        socket.send_pyobj(req)
    if encoder.profiler is None:
        encoder.profiler = EncoderProfiler(encoder.rank)
    ok, msg = encoder.profiler.start(req)
```
**EN:** This block continues `start_profile_async` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `Response`, `ProfileReq`, `time`, `send_pyobj`.
**CN:** 这一段延续了 `start_profile_async` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `Response`、`ProfileReq`、`time`、`send_pyobj`。

### Lines 1718-1726: Async Function `start_profile_async` logic (part 2)
```python
    if ok:
        detail = (
            f"Start profiling. output_dir={encoder.profiler.output_dir} "
            f"profile_id={encoder.profiler.profile_id}\n"
        )
        return Response(content=detail, status_code=200)
    return Response(
        content=(msg or "Start profiling failed.\n"), status_code=HTTPStatus.BAD_REQUEST
    )
```
**EN:** This block continues `start_profile_async` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer server orchestration workflow. Notable operations include `Response`.
**CN:** 这一段延续了 `start_profile_async` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务编排工作流。 值得注意的操作包括 `Response`。

### Lines 1729-1745: Async Function `stop_profile_async`
```python
@app.api_route("/stop_profile", methods=["GET", "POST"])
async def stop_profile_async():
    if encoder is None:
        return Response(content="encoder not ready\n", status_code=503)
    if encoder.profiler is None:
        return Response(
            content="profiling not initialized\n", status_code=HTTPStatus.BAD_REQUEST
        )
    req = ProfileReq(ProfileReqType.STOP_PROFILE)
    for socket in send_sockets:
        socket.send_pyobj(req)
    ok, msg = encoder.profiler.stop()
    if ok:
        return Response(content="Stop profiling.\n", status_code=200)
    return Response(
        content=(msg or "Stop profiling failed.\n"), status_code=HTTPStatus.BAD_REQUEST
    )
```
**EN:** This block defines the async function `stop_profile_async`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer server orchestration. Definitions introduced here include `stop_profile_async`. Notable operations include `api_route`, `Response`, `ProfileReq`, `send_pyobj`.
**CN:** 这一段定义了async function `stop_profile_async`，介绍了参数、初始化步骤，以及这部分编码侧传输服务编排逻辑的主要入口。 此处引入的定义包括 `stop_profile_async`。 值得注意的操作包括 `api_route`、`Response`、`ProfileReq`、`send_pyobj`。

## Key Concepts / 关键概念
- `MMError`: Class that encapsulates mmerror behavior in this module. / `MMError`：封装与“mmerror”相关行为的类。
- `BadRequestError`: Class that encapsulates bad request error behavior in this module. / `BadRequestError`：封装与“bad请求error”相关行为的类。
- `InternalError`: Class that encapsulates internal error behavior in this module. / `InternalError`：封装与“internalerror”相关行为的类。
- `TensorWrapper`: Class that encapsulates tensor wrapper behavior in this module. / `TensorWrapper`：封装与“tensorwrapper”相关行为的类。
- `_convert`: Function that performs convert for the surrounding workflow. / `_convert`：在周边工作流中执行“convert”相关任务的函数。
- `_get_mm_grid_dim`: Function that performs get mm grid dim for the surrounding workflow. / `_get_mm_grid_dim`：在周边工作流中执行“getmmgriddim”相关任务的函数。
- `_get_mm_feature`: Function that performs get mm feature for the surrounding workflow. / `_get_mm_feature`：在周边工作流中执行“getmmfeature”相关任务的函数。
- `_build_mm_aux_data`: Function that performs build mm aux data for the surrounding workflow. / `_build_mm_aux_data`：在周边工作流中执行“buildmmauxdata”相关任务的函数。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `concurrent`, `ctypes`, `logging`, `multiprocessing`, `os`, `pickle`, `time`, `traceback`, `http`, `typing`
- **External packages / 外部依赖**: `aiohttp`, `numpy`, `torch`, `uvicorn`, `zmq`, `fastapi`, `transformers`, `PIL`
- **Internal modules / 内部模块**: `sglang.srt.configs.device_config`, `sglang.srt.configs.load_config`, `sglang.srt.configs.model_config`, `sglang.srt.constants`, `sglang.srt.disaggregation.encode_receiver`, `sglang.srt.distributed.parallel_state`, `sglang.srt.environ`, `sglang.srt.layers.dp_attention`, `sglang.srt.managers.io_struct`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.multimodal_cache`, `sglang.srt.model_loader`, `sglang.srt.multimodal.processors.qwen_vl`, `sglang.srt.server_args`, `sglang.srt.utils`
- **More internal imports / 更多内部导入**: 3 additional module paths omitted for brevity. / 其余 3 个内部模块路径因篇幅原因未展开。
