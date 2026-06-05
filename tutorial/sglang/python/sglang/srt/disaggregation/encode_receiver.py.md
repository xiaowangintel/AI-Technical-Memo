# encode_receiver.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/encode_receiver.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements the encode-side receiver that accepts and interprets incoming transfer operations. It connects remote metadata exchange with local memory operations. / 该文件实现了编码侧接收器，用于接收并解释传入的传输操作，把远端元数据交换与本地内存操作连接起来。

## Line-by-Line Analysis / 逐行分析
### Lines 1-44: Imports and module setup
```python
import asyncio
import itertools
import logging
import pickle
import random
import threading
import time
import uuid
from abc import ABC, abstractmethod
from collections import OrderedDict, defaultdict
from enum import IntEnum
from http import HTTPStatus
from typing import TYPE_CHECKING, Dict, List, Optional

import aiohttp
import numpy as np
import torch
import zmq
import zmq.asyncio
from transformers import PretrainedConfig

from sglang.srt.distributed.parallel_state import (
    GroupCoordinator,
    get_mooncake_transfer_engine,
)
from sglang.srt.environ import envs
from sglang.srt.managers.io_struct import GenerateReqInput, TokenizedGenerateReqInput
from sglang.srt.managers.multimodal_processor import get_mm_processor, import_processors
from sglang.srt.managers.schedule_batch import Modality, Req
from sglang.srt.server_args import ServerArgs
from sglang.srt.utils import ImageData
from sglang.srt.utils.hf_transformers_utils import get_processor
from sglang.srt.utils.network import (
    NetworkAddress,
    get_local_ip_auto,
    get_zmq_socket_on_host,
)

logger = logging.getLogger(__name__)

if TYPE_CHECKING:
    from sglang.srt.managers.scheduler import Scheduler


```
**EN:** This block gathers the imports and module-level setup for encode-side transfer receiving workflow. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与编码侧传输接收流程相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 45-50: Function `_grpc_target`
```python
def _grpc_target(url: str) -> str:
    if url.startswith("grpc://"):
        return url[len("grpc://") :]
    if url.startswith("grpcs://"):
        raise ValueError("grpcs:// is not supported; use grpc://")
    return url
```
**EN:** This block defines the function `_grpc_target`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_grpc_target`. Notable operations include `startswith`, `ValueError`.
**CN:** 这一段定义了function `_grpc_target`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_grpc_target`。 值得注意的操作包括 `startswith`、`ValueError`。

### Lines 53-58: Function `_normalize_embedding_ports`
```python
def _normalize_embedding_ports(embedding_port):
    if embedding_port is None:
        return []
    if isinstance(embedding_port, list):
        return embedding_port
    return [embedding_port]
```
**EN:** This block defines the function `_normalize_embedding_ports`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_normalize_embedding_ports`.
**CN:** 这一段定义了function `_normalize_embedding_ports`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_normalize_embedding_ports`。

### Lines 61-78: Function `_grpc_scheduler_receive_url`
```python
def _grpc_scheduler_receive_url(target, req_id, receive_url, receive_count):
    import grpc
    from smg_grpc_proto import sglang_encoder_pb2, sglang_encoder_pb2_grpc

    timeout_secs = envs.SGLANG_ENCODER_GRPC_TIMEOUT_SECS.get()
    channel = grpc.insecure_channel(target)
    stub = sglang_encoder_pb2_grpc.SglangEncoderStub(channel)
    try:
        stub.SchedulerReceiveUrl(
            sglang_encoder_pb2.SchedulerReceiveUrlRequest(
                req_id=req_id,
                receive_url=receive_url,
                receive_count=receive_count,
            ),
            timeout=timeout_secs,
        )
    finally:
        channel.close()
```
**EN:** This block defines the function `_grpc_scheduler_receive_url`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_grpc_scheduler_receive_url`. Notable operations include `get`, `insecure_channel`, `SglangEncoderStub`, `SchedulerReceiveUrl`.
**CN:** 这一段定义了function `_grpc_scheduler_receive_url`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_grpc_scheduler_receive_url`。 值得注意的操作包括 `get`、`insecure_channel`、`SglangEncoderStub`、`SchedulerReceiveUrl`。

### Lines 81-104: Function `_grpc_encode_request`
```python
def _grpc_encode_request(target, encode_request):
    import grpc
    from smg_grpc_proto import sglang_encoder_pb2, sglang_encoder_pb2_grpc

    timeout_secs = envs.SGLANG_ENCODER_GRPC_TIMEOUT_SECS.get()
    channel = grpc.insecure_channel(target)
    stub = sglang_encoder_pb2_grpc.SglangEncoderStub(channel)
    try:
        response = stub.Encode(
            sglang_encoder_pb2.EncodeRequest(
                mm_items=encode_request["mm_items"],
                req_id=encode_request["req_id"],
                num_parts=encode_request["num_parts"],
                part_idx=encode_request["part_idx"],
                prefill_host=encode_request["prefill_host"],
                embedding_port=_normalize_embedding_ports(
                    encode_request["embedding_port"]
                ),
            ),
            timeout=timeout_secs,
        )
        return response
    finally:
        channel.close()
```
**EN:** This block defines the function `_grpc_encode_request`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_grpc_encode_request`. Notable operations include `get`, `insecure_channel`, `SglangEncoderStub`, `Encode`.
**CN:** 这一段定义了function `_grpc_encode_request`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_grpc_encode_request`。 值得注意的操作包括 `get`、`insecure_channel`、`SglangEncoderStub`、`Encode`。

### Lines 107-126: Function `_grpc_send_request`
```python
def _grpc_send_request(target, request_json):
    import grpc
    from smg_grpc_proto import sglang_encoder_pb2, sglang_encoder_pb2_grpc

    timeout_secs = envs.SGLANG_ENCODER_GRPC_TIMEOUT_SECS.get()
    channel = grpc.insecure_channel(target)
    stub = sglang_encoder_pb2_grpc.SglangEncoderStub(channel)
    try:
        stub.Send(
            sglang_encoder_pb2.SendRequest(
                req_id=request_json["req_id"],
                prefill_host=request_json["prefill_host"],
                embedding_port=request_json["embedding_port"],
                session_id=request_json["session_id"],
                buffer_address=request_json["buffer_address"],
            ),
            timeout=timeout_secs,
        )
    finally:
        channel.close()
```
**EN:** This block defines the function `_grpc_send_request`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_grpc_send_request`. Notable operations include `get`, `insecure_channel`, `SglangEncoderStub`, `Send`.
**CN:** 这一段定义了function `_grpc_send_request`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_grpc_send_request`。 值得注意的操作包括 `get`、`insecure_channel`、`SglangEncoderStub`、`Send`。

### Lines 129-129: Class `EmbeddingData` declaration
```python
class EmbeddingData:
```
**EN:** This block declares the class `EmbeddingData` and establishes its responsibility inside encode-side transfer receiving workflow. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `EmbeddingData`.
**CN:** 这一段声明了类 `EmbeddingData`，并说明它在编码侧传输接收流程中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `EmbeddingData`。

### Lines 130-132: Method `__init__` signature and setup
```python
    def __init__(
        self,
        req_id,
```
**EN:** This block defines the method `__init__` on `EmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `EmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 133-156: Method `__init__` logic (part 1)
```python
        num_parts,
        part_idx,
        grid_dim,
        modality,
        embedding=None,
        embedding_shape=None,
        error_msg=None,
        error_code=None,
        **kwargs,
    ):
        self.req_id = req_id
        self.num_parts = num_parts
        self.part_idx = part_idx
        self.grid_dim = grid_dim
        self.modality = modality
        self.embedding = embedding
        self.send_time = None
        self.dtype = embedding.dtype if embedding is not None else None
        if embedding_shape is not None:
            self.shape = embedding_shape
        else:
            self.shape = list(embedding.shape) if embedding is not None else None
        self.error_msg = error_msg
        self.error_code = error_code
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。

### Lines 157-159: Method `__init__` logic (part 2)
```python
        # Store additional metadata (e.g., video_timestamps for qwen3_vl)
        for key, value in kwargs.items():
            setattr(self, key, value)
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `metadata`, `items`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `metadata`、`items`。

### Lines 161-163: Method `get_grid`
```python
    def get_grid(self):
        """Get the grid dimension of the embedding, used for image/video/audio."""
        return self.grid_dim
```
**EN:** This block defines the method `get_grid` on `EmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `get_grid`.
**CN:** 这一段定义了method `get_grid`（属于 `EmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `get_grid`。

### Lines 165-166: Method `get_embedding`
```python
    def get_embedding(self):
        return self.embedding
```
**EN:** This block defines the method `get_embedding` on `EmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `get_embedding`.
**CN:** 这一段定义了method `get_embedding`（属于 `EmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `get_embedding`。

### Lines 168-169: Method `__repr__`
```python
    def __repr__(self):
        return f"EmbeddingData(req_id={self.req_id}, num_parts={self.num_parts}, part_idx={self.part_idx}) error_msg={self.error_msg}"
```
**EN:** This block defines the method `__repr__` on `EmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `__repr__`. Notable operations include `EmbeddingData`.
**CN:** 这一段定义了method `__repr__`（属于 `EmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `__repr__`。 值得注意的操作包括 `EmbeddingData`。

### Lines 171-187: Method `copy_without_embedding`
```python
    def copy_without_embedding(self):
        new_data = EmbeddingData(
            req_id=self.req_id,
            num_parts=self.num_parts,
            part_idx=self.part_idx,
            grid_dim=self.grid_dim,
            modality=self.modality,
            embedding=None,
            embedding_shape=self.shape,
            error_msg=self.error_msg,
            error_code=self.error_code,
        )
        for key, value in self.__dict__.items():
            if key.startswith("_") or key == "embedding":
                continue
            setattr(new_data, key, value)
        return new_data
```
**EN:** This block defines the method `copy_without_embedding` on `EmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `copy_without_embedding`. Notable operations include `EmbeddingData`, `items`, `startswith`.
**CN:** 这一段定义了method `copy_without_embedding`（属于 `EmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `copy_without_embedding`。 值得注意的操作包括 `EmbeddingData`、`items`、`startswith`。

### Lines 188-212: Module-level constants and helper logic
```python


# Modality -> (list attr name, whether to flatten grid for that list)
_MODALITY_GRID_ATTRS = {
    Modality.IMAGE: ("img_grid_thw", False),
    Modality.VIDEO: ("video_grid_thw", False),
    Modality.AUDIO: ("audio_feature_lens", True),
}
# Per-part video metadata for EPD. Tensor attrs cat on dim=0 across parts;
# others chain as lists. video_meta_attrs_for(model_type) resolves the active
# set per instance so non-MiMo runs skip the MiMo audio fields entirely.
_GENERAL_VIDEO_META_ATTRS = (
    "video_timestamps",
    "second_per_grid_ts",
)
# MiMo-VL audio-in-video fields; appended only when model_type is MiMo.
_MIMO_VIDEO_AUDIO_META_ATTRS = (
    "video_audio_feature_lens",
    "video_audio_segment_lens_flat",
    "video_audio_per_video_num_units",
    "video_audio_embedding",
)
_VIDEO_META_TENSOR_ATTRS = ("video_audio_feature_lens", "video_audio_embedding")


```
**EN:** This block contains module-level constants, helpers, or documentation for encode-side transfer receiving workflow. It prepares shared state that later classes and functions build on. Notable operations include `video_meta_attrs_for`.
**CN:** 这一段包含与编码侧传输接收流程相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `video_meta_attrs_for`。

### Lines 213-218: Function `video_meta_attrs_for`
```python
def video_meta_attrs_for(model_type: Optional[str]) -> tuple:
    """Video-meta attrs for model_type. MiMo appends its audio-in-video fields."""
    attrs = _GENERAL_VIDEO_META_ATTRS
    if model_type and "mimo" in model_type.lower():
        attrs = attrs + _MIMO_VIDEO_AUDIO_META_ATTRS
    return attrs
```
**EN:** This block defines the function `video_meta_attrs_for`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `video_meta_attrs_for`. Notable operations include `lower`.
**CN:** 这一段定义了function `video_meta_attrs_for`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `video_meta_attrs_for`。 值得注意的操作包括 `lower`。

### Lines 221-243: Function `_cat_grid`
```python
def _cat_grid(dims, flatten_items=False):
    """Concatenate non-None grid entries; supports tensor/ndarray/list inputs."""

    def _to_tensor(g):
        if isinstance(g, torch.Tensor):
            return g.cpu() if g.is_cuda else g
        if isinstance(g, np.ndarray):
            return torch.from_numpy(g)
        return torch.as_tensor(g)

    valid = []
    for g in dims:
        if g is None:
            continue
        t = _to_tensor(g)
        if flatten_items:
            t = t.flatten()
        elif t.ndim == 0:
            # Keep cat semantics stable for scalar-like metadata.
            t = t.unsqueeze(0)
        valid.append(t)

    return torch.cat(valid, dim=0) if valid else None
```
**EN:** This block defines the function `_cat_grid`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_cat_grid`, `_to_tensor`. Notable operations include `cpu`, `from_numpy`, `as_tensor`, `_to_tensor`.
**CN:** 这一段定义了function `_cat_grid`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_cat_grid`、`_to_tensor`。 值得注意的操作包括 `cpu`、`from_numpy`、`as_tensor`、`_to_tensor`。

### Lines 246-246: Class `MultiModalEmbeddingData` declaration
```python
class MultiModalEmbeddingData(EmbeddingData):
```
**EN:** This block declares the class `MultiModalEmbeddingData` and establishes its responsibility inside encode-side transfer receiving workflow. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `MultiModalEmbeddingData`.
**CN:** 这一段声明了类 `MultiModalEmbeddingData`，并说明它在编码侧传输接收流程中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `MultiModalEmbeddingData`。

### Lines 247-249: Method `__init__` signature and setup
```python
    def __init__(
        self,
        part_idx,
```
**EN:** This block defines the method `__init__` on `MultiModalEmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `MultiModalEmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 250-273: Method `__init__` logic (part 1)
```python
        num_parts,
        req_id,
        grid_dim,
        modality,
        embedding,
        embedding_shape,
        model_type: Optional[str] = None,
        **kwargs,
    ):
        super().__init__(
            req_id,
            num_parts,
            part_idx,
            grid_dim,
            modality,
            embedding,
            embedding_shape,
            **kwargs,
        )
        self.video_meta_attrs = video_meta_attrs_for(model_type)
        self.img_grid_thw = [None] * num_parts
        self.video_grid_thw = [None] * num_parts
        self.audio_feature_lens = [None] * num_parts
        self.modality_list = [
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `__init__`, `video_meta_attrs_for`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `__init__`、`video_meta_attrs_for`。

### Lines 274-288: Method `__init__` logic (part 2)
```python
            modality if part_idx == i else None for i in range(num_parts)
        ]
        self.ready_list = [i == part_idx for i in range(num_parts)]
        self.embedding_list = [
            embedding if i == part_idx else None for i in range(num_parts)
        ]
        self.embedding_shape_list = [
            embedding_shape if i == part_idx else None for i in range(num_parts)
        ]
        for attr in self.video_meta_attrs:
            setattr(self, attr, [None] * num_parts)

        self._set_part_grid(part_idx, modality, self.get_grid())
        if modality == Modality.VIDEO:
            self._set_video_meta_for_part(part_idx, kwargs)
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `_set_part_grid`, `get_grid`, `_set_video_meta_for_part`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `_set_part_grid`、`get_grid`、`_set_video_meta_for_part`。

### Lines 290-297: Method `_set_part_grid`
```python
    def _set_part_grid(self, part_idx, modality, grid):
        """Set the grid for one part according to modality (IMAGE/VIDEO/AUDIO)."""
        spec = _MODALITY_GRID_ATTRS.get(modality)
        if spec is None:
            raise ValueError(f"Invalid modality: {modality}")
        attr_name, flatten = spec
        value = grid.flatten() if flatten else grid
        getattr(self, attr_name)[part_idx] = value
```
**EN:** This block defines the method `_set_part_grid` on `MultiModalEmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_set_part_grid`. Notable operations include `modality`, `get`, `ValueError`, `flatten`.
**CN:** 这一段定义了method `_set_part_grid`（属于 `MultiModalEmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_set_part_grid`。 值得注意的操作包括 `modality`、`get`、`ValueError`、`flatten`。

### Lines 299-308: Method `_set_video_meta_for_part`
```python
    def _set_video_meta_for_part(self, part_idx, source):
        """Copy video_timestamps and second_per_grid_ts from source (dict or object)."""
        for attr_name in self.video_meta_attrs:
            val = (
                source.get(attr_name)
                if isinstance(source, dict)
                else getattr(source, attr_name, None)
            )
            if val is not None:
                getattr(self, attr_name)[part_idx] = val
```
**EN:** This block defines the method `_set_video_meta_for_part` on `MultiModalEmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_set_video_meta_for_part`. Notable operations include `source`, `get`.
**CN:** 这一段定义了method `_set_video_meta_for_part`（属于 `MultiModalEmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_set_video_meta_for_part`。 值得注意的操作包括 `source`、`get`。

### Lines 310-316: Method `from_embedding_data` signature and setup
```python
    @classmethod
    def from_embedding_data(
        cls,
        embedding_data: EmbeddingData,
        model_type: Optional[str] = None,
    ):
        """Create MultiModalEmbeddingData from an EmbeddingData instance."""
```
**EN:** This block defines the method `from_embedding_data` on `MultiModalEmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `from_embedding_data`.
**CN:** 这一段定义了method `from_embedding_data`（属于 `MultiModalEmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `from_embedding_data`。

### Lines 317-335: Method `from_embedding_data` logic (part 1)
```python
        # Only forward known optional attrs (e.g. video metadata) so they land on the instance
        extra = {}
        for attr in video_meta_attrs_for(model_type):
            val = getattr(embedding_data, attr, None)
            if val is not None:
                extra[attr] = val
        mm_data = cls(
            part_idx=embedding_data.part_idx,
            num_parts=embedding_data.num_parts,
            req_id=embedding_data.req_id,
            grid_dim=embedding_data.grid_dim,
            modality=embedding_data.modality,
            embedding=embedding_data.embedding,
            embedding_shape=embedding_data.shape,
            model_type=model_type,
            **extra,
        )
        mm_data.send_time = embedding_data.send_time
        return mm_data
```
**EN:** This block continues `from_embedding_data` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `attrs`, `video_meta_attrs_for`, `cls`.
**CN:** 这一段延续了 `from_embedding_data` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `attrs`、`video_meta_attrs_for`、`cls`。

### Lines 337-338: Method `__repr__`
```python
    def __repr__(self):
        return f"MultiModalEmbeddingData(req_id={self.req_id}, num_parts={self.num_parts}, part_idx={self.part_idx}, modality={self.modality})"
```
**EN:** This block defines the method `__repr__` on `MultiModalEmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `__repr__`. Notable operations include `MultiModalEmbeddingData`.
**CN:** 这一段定义了method `__repr__`（属于 `MultiModalEmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `__repr__`。 值得注意的操作包括 `MultiModalEmbeddingData`。

### Lines 340-347: Method `get_embedding`
```python
    def get_embedding(self, is_concat=False):
        if is_concat:
            groups = defaultdict(list)
            for i, e in enumerate(self.embedding_list):
                if e is not None:
                    groups[self.modality_list[i]].append(e)
            return {mod: torch.cat(tensors, dim=0) for mod, tensors in groups.items()}
        return self.embedding_list
```
**EN:** This block defines the method `get_embedding` on `MultiModalEmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `get_embedding`. Notable operations include `defaultdict`, `append`, `cat`, `items`.
**CN:** 这一段定义了method `get_embedding`（属于 `MultiModalEmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `get_embedding`。 值得注意的操作包括 `defaultdict`、`append`、`cat`、`items`。

### Lines 349-351: Method `ready`
```python
    @property
    def ready(self):
        return sum(self.ready_list) == self.num_parts
```
**EN:** This block defines the method `ready` on `MultiModalEmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `ready`.
**CN:** 这一段定义了method `ready`（属于 `MultiModalEmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `ready`。

### Lines 353-373: Method `get_mm_extra_meta`
```python
    def get_mm_extra_meta(self):
        """Build kwargs for mm_processor.get_mm_data() from grid and optional video meta."""
        kwargs = {
            "img_grid_thw": _cat_grid(self.img_grid_thw),
            "video_grid_thw": _cat_grid(self.video_grid_thw),
            "audio_feature_lens": _cat_grid(
                self.audio_feature_lens, flatten_items=True
            ),
        }
        for attr in self.video_meta_attrs:
            lst = getattr(self, attr, None)
            if not lst:
                continue
            valid = [a for a in lst if a is not None]
            if not valid:
                continue
            if attr in _VIDEO_META_TENSOR_ATTRS:
                kwargs[attr] = torch.cat(valid, dim=0)
            else:
                kwargs[attr] = list(itertools.chain(*valid))
        return kwargs
```
**EN:** This block defines the method `get_mm_extra_meta` on `MultiModalEmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `get_mm_extra_meta`. Notable operations include `get_mm_data`, `_cat_grid`, `cat`, `chain`.
**CN:** 这一段定义了method `get_mm_extra_meta`（属于 `MultiModalEmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `get_mm_extra_meta`。 值得注意的操作包括 `get_mm_data`、`_cat_grid`、`cat`、`chain`。

### Lines 375-390: Method `add`
```python
    def add(self, embedding_data: EmbeddingData):
        if self.req_id != embedding_data.req_id:
            logger.warning(
                f"Dropping embedding data with mismatched req_id: "
                f"expected {self.req_id}, got {embedding_data.req_id}"
            )
            return False
        assert not self.ready_list[embedding_data.part_idx]
        pid = embedding_data.part_idx
        self.ready_list[pid] = True
        self.modality_list[pid] = embedding_data.modality
        self.embedding_list[pid] = embedding_data.get_embedding()
        self.embedding_shape_list[pid] = embedding_data.shape
        self._set_part_grid(pid, embedding_data.modality, embedding_data.get_grid())
        if embedding_data.modality == Modality.VIDEO:
            self._set_video_meta_for_part(pid, embedding_data)
```
**EN:** This block defines the method `add` on `MultiModalEmbeddingData`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `add`. Notable operations include `warning`, `get_embedding`, `_set_part_grid`, `get_grid`.
**CN:** 这一段定义了method `add`（属于 `MultiModalEmbeddingData`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `add`。 值得注意的操作包括 `warning`、`get_embedding`、`_set_part_grid`、`get_grid`。

### Lines 393-393: Class `WaitingImageRequestStatus` declaration
```python
class WaitingImageRequestStatus(IntEnum):
```
**EN:** This block declares the class `WaitingImageRequestStatus` and establishes its responsibility inside encode-side transfer receiving workflow. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `WaitingImageRequestStatus`.
**CN:** 这一段声明了类 `WaitingImageRequestStatus`，并说明它在编码侧传输接收流程中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `WaitingImageRequestStatus`。

### Lines 394-397: Supporting state inside `WaitingImageRequestStatus`
```python
    FAIL = -1
    PENDING = 0
    SUCCESS = 1
    TIMEOUT = -2
```
**EN:** This block adds supporting state or helper logic inside `WaitingImageRequestStatus`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `WaitingImageRequestStatus` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 400-402: Function `create_part_req_id`
```python
def create_part_req_id(original_req_id: str, part_idx: int) -> str:
    """Create a unique part request ID by appending part index suffix."""
    return f"{original_req_id}_local_part_{part_idx}"
```
**EN:** This block defines the function `create_part_req_id`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `create_part_req_id`.
**CN:** 这一段定义了function `create_part_req_id`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `create_part_req_id`。

### Lines 405-409: Function `extract_original_req_id`
```python
def extract_original_req_id(part_req_id: str) -> str:
    """Extract the original request ID from a part request ID."""
    if "_local_part_" in part_req_id:
        return part_req_id.rsplit("_local_part_", 1)[0]
    return part_req_id
```
**EN:** This block defines the function `extract_original_req_id`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `extract_original_req_id`. Notable operations include `rsplit`.
**CN:** 这一段定义了function `extract_original_req_id`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `extract_original_req_id`。 值得注意的操作包括 `rsplit`。

### Lines 412-432: Function `calculate_modality_num_parts`
```python
def calculate_modality_num_parts(modalities, num_items_assigned):
    """
    Calculate total number of parts and number of parts per modality.

    Args:
        modalities: List of modalities in order
        num_items_assigned: Dictionary mapping modality to list of assignment counts per encoder

    Returns:
        Tuple of (total_num_parts, modality_num_parts_dict)
        - total_num_parts: Total number of parts across all modalities
        - modality_num_parts: Dictionary mapping modality to number of parts for that modality
    """
    total_num_parts = 0
    modality_num_parts = {}
    for modality in modalities:
        num_items_assigned_modality = num_items_assigned.get(modality)
        num_parts = sum(1 for x in num_items_assigned_modality if x != 0)
        modality_num_parts[modality] = num_parts
        total_num_parts += num_parts
    return total_num_parts, modality_num_parts
```
**EN:** This block defines the function `calculate_modality_num_parts`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `calculate_modality_num_parts`. Notable operations include `of`, `get`.
**CN:** 这一段定义了function `calculate_modality_num_parts`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `calculate_modality_num_parts`。 值得注意的操作包括 `of`、`get`。

### Lines 433-435: Module-level constants and helper logic
```python


# For zmq_to_scheduler
```
**EN:** This block contains module-level constants, helpers, or documentation for encode-side transfer receiving workflow. It prepares shared state that later classes and functions build on.
**CN:** 这一段包含与编码侧传输接收流程相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。

### Lines 436-436: Class `WaitingImageRequest` declaration
```python
class WaitingImageRequest:
```
**EN:** This block declares the class `WaitingImageRequest` and establishes its responsibility inside encode-side transfer receiving workflow. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `WaitingImageRequest`.
**CN:** 这一段声明了类 `WaitingImageRequest`，并说明它在编码侧传输接收流程中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `WaitingImageRequest`。

### Lines 437-439: Method `__init__` signature and setup
```python
    def __init__(
        self,
        rid: str,
```
**EN:** This block defines the method `__init__` on `WaitingImageRequest`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `WaitingImageRequest`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 440-463: Method `__init__` logic (part 1)
```python
        recv_req: TokenizedGenerateReqInput,
        mm_processor,
        encoder_urls,
        host_name,
        receive_count,
    ):
        self.rid = rid
        self.recv_req = recv_req
        self.mm_inputs = None
        self.error = None
        self.thread = None
        self.mm_processor = mm_processor
        self.encoder_urls = encoder_urls
        self.host_name = host_name
        self.receive_count = receive_count
        self.num_items_assigned = recv_req.num_items_assigned
        self.embedding_port, self.recv_socket = get_zmq_socket_on_host(
            zmq.Context(), zmq.PULL, host=host_name
        )
        logger.info(f"Waiting for input {self.embedding_port = }")
        self.recv_embedding_data = None
        # ok=1 pending=0 fail=-1
        self.status = WaitingImageRequestStatus.PENDING
        self.error_msg = None
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `get_zmq_socket_on_host`, `Context`, `info`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `get_zmq_socket_on_host`、`Context`、`info`。

### Lines 464-465: Method `__init__` logic (part 2)
```python
        self.error_code = None
        self.start_time = time.time()
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `time`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `time`。

### Lines 467-469: Method `send_encode_request` signature and setup
```python
    def send_encode_request(self):
        async def _send_single_request(session, url, payload):
            try:
```
**EN:** This block defines the method `send_encode_request` on `WaitingImageRequest`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `send_encode_request`, `_send_single_request`.
**CN:** 这一段定义了method `send_encode_request`（属于 `WaitingImageRequest`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `send_encode_request`、`_send_single_request`。

### Lines 470-488: Method `send_encode_request` logic (part 1)
```python
                async with session.post(url, json=payload) as response:
                    response.raise_for_status()
                    return await response.text()
            except Exception as e:
                logger.error(f"Failed to send request to {url}: {e}")
                raise

        async def send_embedding_port(req_id, receive_count, host_name, embedding_port):
            async with aiohttp.ClientSession(
                timeout=aiohttp.ClientTimeout(total=1800)
            ) as session:
                tasks = []
                logger.info(f"{self.num_items_assigned = } ")

                # Calculate part_idx_offset similar to encode() method
                modalities = list(self.num_items_assigned.keys())
                _, modality_num_parts = calculate_modality_num_parts(
                    modalities, self.num_items_assigned
                )
```
**EN:** This block continues `send_encode_request` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Definitions introduced here include `send_embedding_port`. Notable operations include `post`, `raise_for_status`, `text`, `error`.
**CN:** 这一段延续了 `send_encode_request` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 此处引入的定义包括 `send_embedding_port`。 值得注意的操作包括 `post`、`raise_for_status`、`text`、`error`。

### Lines 489-516: Method `send_encode_request` logic (part 2)
```python

                part_idx_offset = 0
                for modality in modalities:
                    assigned_nums = self.num_items_assigned[modality]
                    num_parts = modality_num_parts[modality]
                    cum_idx = 0
                    for idx, assigned_num in enumerate(assigned_nums):
                        if assigned_num == 0:
                            continue
                        part_idx = part_idx_offset + cum_idx
                        part_req_id = create_part_req_id(req_id, part_idx)
                        encoder_url = self.encoder_urls[idx]
                        target_url = f"{encoder_url}/scheduler_receive_url"
                        payload = {
                            "req_id": part_req_id,  # use part_req_id to match encode request
                            "receive_count": receive_count,
                            "receive_url": NetworkAddress(
                                host_name, embedding_port
                            ).to_host_port_str(),
                            "modality": modality.name,
                        }
                        logger.info(
                            f"Preparing to send to {target_url} with part_req_id={part_req_id}"
                        )
                        task = _send_single_request(session, target_url, payload)
                        tasks.append(task)
                        cum_idx += 1
                    part_idx_offset += num_parts
```
**EN:** This block continues `send_encode_request` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `create_part_req_id`, `NetworkAddress`, `to_host_port_str`, `info`.
**CN:** 这一段延续了 `send_encode_request` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `create_part_req_id`、`NetworkAddress`、`to_host_port_str`、`info`。

### Lines 517-537: Method `send_encode_request` logic (part 3)
```python

                if not tasks:
                    logger.info("No tasks to send.")
                    return
                logger.info(f"Concurrently sending {len(tasks)} requests...")
                results = await asyncio.gather(*tasks, return_exceptions=True)

                for i, result in enumerate(results):
                    if isinstance(result, Exception):
                        logger.error(f"Request {i} failed: {result}")
                    else:
                        logger.debug(f"Request {i} succeeded.")

        asyncio.run(
            send_embedding_port(
                self.recv_req.rid,
                self.receive_count,
                self.host_name,
                self.embedding_port,
            )
        )
```
**EN:** This block continues `send_encode_request` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `info`, `gather`, `error`, `debug`.
**CN:** 这一段延续了 `send_encode_request` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `info`、`gather`、`error`、`debug`。

### Lines 539-541: Method `_try_recv_mm_data` signature and setup
```python
    def _try_recv_mm_data(self):
        if self.status != WaitingImageRequestStatus.PENDING:
            return
```
**EN:** This block defines the method `_try_recv_mm_data` on `WaitingImageRequest`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_try_recv_mm_data`.
**CN:** 这一段定义了method `_try_recv_mm_data`（属于 `WaitingImageRequest`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_try_recv_mm_data`。

### Lines 542-568: Method `_try_recv_mm_data` logic (part 1)
```python
        while self.recv_embedding_data is None or not self.recv_embedding_data.ready:
            try:
                parts = self.recv_socket.recv_multipart(flags=zmq.NOBLOCK, copy=False)
            except zmq.Again:
                # No data available yet, wait a bit and retry
                return
            recv_obj: EmbeddingData = pickle.loads(parts[0])
            if getattr(recv_obj, "error_msg", None) is not None:
                logger.warning(
                    f"Received error signal from encoder for {self.rid}: {recv_obj.error_msg} {recv_obj.error_code = }"
                )
                self.error_msg = recv_obj.error_msg
                self.error_code = recv_obj.error_code
                self.status = WaitingImageRequestStatus.FAIL
                self.recv_socket.close()
                return

            # Extract original req_id from part_req_id and drop stale payloads
            # that may arrive on a reused ZMQ port after a prior request aborted.
            original_req_id = extract_original_req_id(recv_obj.req_id)
            if original_req_id != self.recv_req.rid:
                logger.warning(
                    f"Dropping stale embedding data: expected rid={self.recv_req.rid}, "
                    f"got rid={recv_obj.req_id} (likely from ZMQ port reuse)"
                )
                continue
            recv_obj.req_id = original_req_id
```
**EN:** This block continues `_try_recv_mm_data` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `recv_multipart`, `loads`, `warning`, `close`.
**CN:** 这一段延续了 `_try_recv_mm_data` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `recv_multipart`、`loads`、`warning`、`close`。

### Lines 569-592: Method `_try_recv_mm_data` logic (part 2)
```python

            buffer = parts[1].buffer if hasattr(parts[1], "buffer") else parts[1]
            recv_obj.embedding = (
                torch.frombuffer(buffer, dtype=recv_obj.dtype)
                .reshape(recv_obj.shape)
                .clone()
            )

            if self.recv_embedding_data is None:
                self.recv_embedding_data = MultiModalEmbeddingData.from_embedding_data(
                    recv_obj, model_type=self.model_type
                )
            else:
                self.recv_embedding_data.add(recv_obj)

        recv_embedding = self.recv_embedding_data.get_embedding(is_concat=True)
        mm_inputs = self.mm_processor.get_mm_data(
            self.recv_req.input_text,
            recv_embedding,
            **self.recv_embedding_data.get_mm_extra_meta(),
        )
        self.recv_req.mm_inputs = mm_inputs
        self.recv_req.input_ids = mm_inputs.input_ids
        self.status = WaitingImageRequestStatus.SUCCESS
```
**EN:** This block continues `_try_recv_mm_data` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `frombuffer`, `reshape`, `clone`, `from_embedding_data`.
**CN:** 这一段延续了 `_try_recv_mm_data` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `frombuffer`、`reshape`、`clone`、`from_embedding_data`。

### Lines 593-593: Method `_try_recv_mm_data` logic (part 3)
```python
        self.recv_socket.close()
```
**EN:** This block continues `_try_recv_mm_data` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `close`.
**CN:** 这一段延续了 `_try_recv_mm_data` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `close`。

### Lines 596-596: Class `WaitingImageRequestGrpc` declaration
```python
class WaitingImageRequestGrpc(WaitingImageRequest):
```
**EN:** This block declares the class `WaitingImageRequestGrpc` and establishes its responsibility inside encode-side transfer receiving workflow. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `WaitingImageRequestGrpc`.
**CN:** 这一段声明了类 `WaitingImageRequestGrpc`，并说明它在编码侧传输接收流程中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `WaitingImageRequestGrpc`。

### Lines 597-599: Method `send_encode_request` signature and setup
```python
    def send_encode_request(self):
        async def send_embedding_port(req_id, receive_count, host_name, embedding_port):
            tasks = []
```
**EN:** This block defines the method `send_encode_request` on `WaitingImageRequestGrpc`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `send_encode_request`, `send_embedding_port`.
**CN:** 这一段定义了method `send_encode_request`（属于 `WaitingImageRequestGrpc`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `send_encode_request`、`send_embedding_port`。

### Lines 600-619: Method `send_encode_request` logic (part 1)
```python
            # gRPC image-only: flatten modality dict to flat list
            assigned = list(self.num_items_assigned.values())[0]
            logger.info(f"num_items_assigned={assigned}")

            for idx, assigned_num in enumerate(assigned):
                if assigned_num == 0:
                    continue
                encoder_url = self.encoder_urls[idx]
                receive_url = f"{host_name}:{embedding_port}"
                target_url = f"{encoder_url}/SchedulerReceiveUrl"
                logger.info(f"Preparing to send to {target_url}")
                tasks.append(
                    asyncio.to_thread(
                        _grpc_scheduler_receive_url,
                        _grpc_target(encoder_url),
                        req_id,
                        receive_url,
                        receive_count,
                    )
                )
```
**EN:** This block continues `send_encode_request` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `values`, `info`, `append`, `to_thread`.
**CN:** 这一段延续了 `send_encode_request` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `values`、`info`、`append`、`to_thread`。

### Lines 620-640: Method `send_encode_request` logic (part 2)
```python

            if not tasks:
                logger.info("No tasks to send.")
                return
            logger.info(f"Concurrently sending {len(tasks)} requests...")
            results = await asyncio.gather(*tasks, return_exceptions=True)

            for i, result in enumerate(results):
                if isinstance(result, Exception):
                    logger.error(f"Request {i} failed: {result}")
                else:
                    logger.debug(f"Request {i} succeeded.")

        asyncio.run(
            send_embedding_port(
                self.recv_req.rid,
                self.receive_count,
                self.host_name,
                self.embedding_port,
            )
        )
```
**EN:** This block continues `send_encode_request` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `info`, `gather`, `error`, `debug`.
**CN:** 这一段延续了 `send_encode_request` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `info`、`gather`、`error`、`debug`。

### Lines 643-650: Function `_determine_tensor_transport_mode`
```python
def _determine_tensor_transport_mode(server_args):
    is_cross_node = server_args.dist_init_addr

    if is_cross_node:
        # Fallback to default CPU transport for multi-node
        return "default"
    else:
        return "cuda_ipc"
```
**EN:** This block defines the function `_determine_tensor_transport_mode`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_determine_tensor_transport_mode`.
**CN:** 这一段定义了function `_determine_tensor_transport_mode`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_determine_tensor_transport_mode`。

### Lines 653-653: Class `MMReceiverBase` declaration
```python
class MMReceiverBase(ABC):
```
**EN:** This block declares the class `MMReceiverBase` and establishes its responsibility inside encode-side transfer receiving workflow. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `MMReceiverBase`.
**CN:** 这一段声明了类 `MMReceiverBase`，并说明它在编码侧传输接收流程中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `MMReceiverBase`。

### Lines 654-656: Method `__init__` signature and setup
```python
    def __init__(
        self,
        server_args: ServerArgs,
```
**EN:** This block defines the method `__init__` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 657-680: Method `__init__` logic (part 1)
```python
        dtype: Optional[torch.dtype] = None,
        hf_config: Optional[PretrainedConfig] = None,
        pp_rank: Optional[int] = None,
        tp_rank: Optional[int] = None,
        tp_group: Optional[GroupCoordinator] = None,
        scheduler: Optional["Scheduler"] = None,
    ):
        self.context = zmq.asyncio.Context(20)
        self.encoder_transfer_backend = server_args.encoder_transfer_backend
        self.encode_urls = server_args.encoder_urls
        self.recv_timeout = envs.SGLANG_ENCODER_RECV_TIMEOUT.get()
        self.host = get_local_ip_auto(server_args.host)
        self.model_type = (
            getattr(hf_config, "model_type", "").lower()
            if hf_config is not None
            else None
        )
        if self.encoder_transfer_backend == "mooncake":
            self.dtype = dtype
            self.embeddings_engine = get_mooncake_transfer_engine()
            if self.embeddings_engine is None:
                from sglang.srt.distributed.device_communicators.mooncake_transfer_engine import (
                    init_mooncake_transfer_engine,
                )
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `Context`, `get`, `get_local_ip_auto`, `lower`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `Context`、`get`、`get_local_ip_auto`、`lower`。

### Lines 681-712: Method `__init__` logic (part 2)
```python

                self.embeddings_engine = init_mooncake_transfer_engine(
                    hostname=self.host,
                    ib_device=(
                        server_args.disaggregation_ib_device
                        or server_args.mooncake_ib_device
                    ),
                )
            self.embeddings_buffer = dict()
        elif self.encoder_transfer_backend == "zmq_to_scheduler":
            self.pp_rank = pp_rank
            self.tp_rank = tp_rank
            self.tp_size = server_args.tp_size
            self.tp_group = tp_group
            self.nnodes = server_args.nnodes
            self.hostname = get_local_ip_auto()
            self.waiting_list: List[WaitingImageRequest] = []
            self.scheduler = scheduler
            self.wait_timeout = envs.SGLANG_ENCODER_RECV_TIMEOUT.get()
            if hf_config is not None:
                transport_mode = _determine_tensor_transport_mode(server_args)
                import_processors("sglang.srt.multimodal.processors")
                _processor = None
                try:
                    _processor = get_processor(
                        server_args.tokenizer_path,
                        tokenizer_mode=server_args.tokenizer_mode,
                        trust_remote_code=server_args.trust_remote_code,
                        revision=server_args.revision,
                        use_fast=not server_args.disable_fast_image_processor,
                        tokenizer_backend=server_args.tokenizer_backend,
                    )
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `init_mooncake_transfer_engine`, `get_local_ip_auto`, `get`, `_determine_tensor_transport_mode`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `init_mooncake_transfer_engine`、`get_local_ip_auto`、`get`、`_determine_tensor_transport_mode`。

### Lines 713-744: Method `__init__` logic (part 3)
```python
                except ValueError as e:
                    error_message = str(e)
                    if "does not have a slow version" in error_message:
                        logger.info(
                            f"Processor {server_args.tokenizer_path} does not have a slow version. Automatically use fast version"
                        )
                        _processor = get_processor(
                            server_args.tokenizer_path,
                            tokenizer_mode=server_args.tokenizer_mode,
                            trust_remote_code=server_args.trust_remote_code,
                            revision=server_args.revision,
                            use_fast=True,
                            tokenizer_backend=server_args.tokenizer_backend,
                        )
                    else:
                        raise e

                # Skip mm_pool if not adaptive dispatch to encoder
                enable_adaptive_dispatch_to_encoder = (
                    server_args.enable_adaptive_dispatch_to_encoder
                )
                self.mm_processor = get_mm_processor(
                    hf_config,
                    server_args,
                    _processor,
                    transport_mode,
                    model_config=(
                        getattr(self.scheduler, "model_config", None)
                        if self.scheduler is not None
                        else None
                    ),
                    skip_mm_pool=not enable_adaptive_dispatch_to_encoder,
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `info`, `get_processor`, `get_mm_processor`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `info`、`get_processor`、`get_mm_processor`。

### Lines 745-745: Method `__init__` logic (part 4)
```python
                )
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。

### Lines 747-749: Method `process_waiting_requests`
```python
    @abstractmethod
    def process_waiting_requests(self, recv_reqs):
        pass
```
**EN:** This block defines the method `process_waiting_requests` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `process_waiting_requests`.
**CN:** 这一段定义了method `process_waiting_requests`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `process_waiting_requests`。

### Lines 751-753: Method `recv_mm_data` signature and setup
```python
    async def recv_mm_data(
        self, request_obj, mm_processor, prompt, need_wait_for_mm_inputs=True
    ):
```
**EN:** This block defines the method `recv_mm_data` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `recv_mm_data`.
**CN:** 这一段定义了method `recv_mm_data`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `recv_mm_data`。

### Lines 754-777: Method `recv_mm_data` logic (part 1)
```python
        req_id = None
        try:
            if len(self.encode_urls) == 0 or not need_wait_for_mm_inputs:
                return None
            req_id = uuid.uuid4().hex
            embedding_port, recv_socket = get_zmq_socket_on_host(
                self.context, zmq.PULL, host=self.host
            )
            mm_data = self._extract_url_data(request_obj)
            modalities = [m.get("modality") for m in mm_data]
            logger.info(
                f"[{req_id}] Sending encode request to E, "
                f"modalities={modalities}, num_items={len(mm_data)}"
            )
            send_time = time.monotonic()
            asyncio.create_task(
                self.encode(req_id, mm_data, embedding_port, "encode", "send")
            )
            result = await asyncio.wait_for(
                self._recv_mm_data(req_id, recv_socket, mm_processor, prompt),
                timeout=self.recv_timeout,
            )
            elapsed = time.monotonic() - send_time
            logger.info(f"[{req_id}] Received embedding from E in {elapsed:.3f}s")
```
**EN:** This block continues `recv_mm_data` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `uuid4`, `get_zmq_socket_on_host`, `_extract_url_data`, `get`.
**CN:** 这一段延续了 `recv_mm_data` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `uuid4`、`get_zmq_socket_on_host`、`_extract_url_data`、`get`。

### Lines 778-784: Method `recv_mm_data` logic (part 2)
```python
            return result
        except asyncio.TimeoutError:
            elapsed = time.monotonic() - send_time
            logger.warning(f"[{req_id}] Embedding recv timeout after {elapsed:.3f}s")
            if req_id is not None:
                self._cleanup_mooncake_buffer(req_id)
            return None
```
**EN:** This block continues `recv_mm_data` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `monotonic`, `warning`, `_cleanup_mooncake_buffer`.
**CN:** 这一段延续了 `recv_mm_data` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `monotonic`、`warning`、`_cleanup_mooncake_buffer`。

### Lines 786-799: Method `_cleanup_mooncake_buffer`
```python
    def _cleanup_mooncake_buffer(self, req_id):
        if self.encoder_transfer_backend != "mooncake":
            return
        if not hasattr(self, "embeddings_buffer"):
            return
        embeddings = self.embeddings_buffer.pop(req_id, None)
        if embeddings is None:
            return
        try:
            self.embeddings_engine.deregister(embeddings.data_ptr())
        except Exception:
            logger.exception(
                "mooncake: failed to deregister buffer for req_id=%s", req_id
            )
```
**EN:** This block defines the method `_cleanup_mooncake_buffer` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_cleanup_mooncake_buffer`. Notable operations include `pop`, `deregister`, `data_ptr`, `exception`.
**CN:** 这一段定义了method `_cleanup_mooncake_buffer`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_cleanup_mooncake_buffer`。 值得注意的操作包括 `pop`、`deregister`、`data_ptr`、`exception`。

### Lines 801-803: Method `_recv_mm_data` signature and setup
```python
    async def _recv_mm_data(self, req_id, recv_socket, mm_processor, prompt):
        if req_id is None:
            return None
```
**EN:** This block defines the method `_recv_mm_data` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_recv_mm_data`.
**CN:** 这一段定义了method `_recv_mm_data`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_recv_mm_data`。

### Lines 804-835: Method `_recv_mm_data` logic (part 1)
```python

        recv_embedding = None

        recv_embedding_data: MultiModalEmbeddingData = None

        try:
            while recv_embedding_data is None or not recv_embedding_data.ready:
                parts = await recv_socket.recv_multipart(copy=False)
                if not parts:
                    continue
                recv_obj: EmbeddingData = pickle.loads(parts[0])
                if getattr(recv_obj, "error_msg", None) is not None:
                    logger.warning(
                        f"Encoder error for req_id={req_id}: {recv_obj.error_msg} "
                        f"error_code={getattr(recv_obj, 'error_code', None)}"
                    )
                    self._cleanup_mooncake_buffer(req_id)
                    return None
                logger.debug("recv_obj=%s", recv_obj)
                # Extract original req_id from part_req_id
                part_req_id = recv_obj.req_id
                original_req_id = extract_original_req_id(part_req_id)
                # Update recv_obj.req_id to original for aggregation
                recv_obj.req_id = original_req_id
                if self.encoder_transfer_backend == "zmq_to_tokenizer":
                    if len(parts) < 2:
                        logger.error(
                            "zmq_to_tokenizer expected 2-part message, got %d parts",
                            len(parts),
                        )
                        return None
                    buffer = (
```
**EN:** This block continues `_recv_mm_data` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `recv_multipart`, `loads`, `warning`, `_cleanup_mooncake_buffer`.
**CN:** 这一段延续了 `_recv_mm_data` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `recv_multipart`、`loads`、`warning`、`_cleanup_mooncake_buffer`。

### Lines 836-874: Method `_recv_mm_data` logic (part 2)
```python
                        parts[1].buffer if hasattr(parts[1], "buffer") else parts[1]
                    )
                    # Clone so we don't depend on ZMQ buffer after next recv.
                    recv_obj.embedding = (
                        torch.frombuffer(buffer, dtype=recv_obj.dtype)
                        .reshape(recv_obj.shape)
                        .clone()
                    )
                if recv_embedding_data is None:
                    recv_embedding_data = MultiModalEmbeddingData.from_embedding_data(
                        recv_obj, model_type=self.model_type
                    )
                else:
                    recv_embedding_data.add(recv_obj)

            if self.encoder_transfer_backend == "mooncake":
                if req_id not in self.embeddings_buffer:
                    logger.error(
                        "mooncake: embeddings_buffer missing req_id=%s", req_id
                    )
                    return None
                raw_buffer = self.embeddings_buffer.pop(req_id)
                self.embeddings_engine.deregister(raw_buffer.data_ptr())
                byte_offset = 0
                for i in range(recv_embedding_data.num_parts):
                    shape = recv_embedding_data.embedding_shape_list[i]
                    if shape is None:
                        continue
                    part_bytes = (
                        shape[0]
                        * shape[1]
                        * torch.tensor([], dtype=self.dtype).element_size()
                    )
                    recv_embedding_data.embedding_list[i] = (
                        raw_buffer[byte_offset : byte_offset + part_bytes]
                        .view(self.dtype)
                        .reshape(shape)
                    )
                    byte_offset += part_bytes
```
**EN:** This block continues `_recv_mm_data` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `frombuffer`, `reshape`, `clone`, `from_embedding_data`.
**CN:** 这一段延续了 `_recv_mm_data` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `frombuffer`、`reshape`、`clone`、`from_embedding_data`。

### Lines 875-885: Method `_recv_mm_data` logic (part 3)
```python

            recv_embedding = recv_embedding_data.get_embedding(is_concat=True)

            mm_inputs = mm_processor.get_mm_data(
                prompt,
                recv_embedding,
                **recv_embedding_data.get_mm_extra_meta(),
            )
            return mm_inputs
        finally:
            recv_socket.close()
```
**EN:** This block continues `_recv_mm_data` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `get_embedding`, `get_mm_data`, `get_mm_extra_meta`, `close`.
**CN:** 这一段延续了 `_recv_mm_data` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `get_embedding`、`get_mm_data`、`get_mm_extra_meta`、`close`。

### Lines 887-888: Method `send_encode_request`
```python
    def send_encode_request(self, obj):
        self._send_encode_request(obj)
```
**EN:** This block defines the method `send_encode_request` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `send_encode_request`. Notable operations include `_send_encode_request`.
**CN:** 这一段定义了method `send_encode_request`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `send_encode_request`。 值得注意的操作包括 `_send_encode_request`。

### Lines 890-913: Method `_send_encode_request`
```python
    def _send_encode_request(self, obj):
        mm_data = self._extract_url_data(obj)
        if obj.rid is None:
            obj.rid = uuid.uuid4().hex
        if mm_data and self.encode_urls:
            logger.info(f"Processing {len(mm_data)} mm items for request {obj.rid}")
            obj.need_wait_for_mm_inputs = True

            num_items_assigned = self._assign_items_by_modality(
                mm_data, len(self.encode_urls)
            )
            obj.num_items_assigned = num_items_assigned
            encode_thread = threading.Thread(
                target=self._run_encode_in_thread,
                args=(
                    obj.rid,
                    mm_data,
                    "encode",
                    num_items_assigned,
                    None,
                ),
                daemon=True,
            )
            encode_thread.start()
```
**EN:** This block defines the method `_send_encode_request` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_send_encode_request`. Notable operations include `_extract_url_data`, `uuid4`, `info`, `_assign_items_by_modality`.
**CN:** 这一段定义了method `_send_encode_request`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_send_encode_request`。 值得注意的操作包括 `_extract_url_data`、`uuid4`、`info`、`_assign_items_by_modality`。

### Lines 914-915: Supporting state inside `MMReceiverBase`
```python

    # For zmq_to_scheduler
```
**EN:** This block adds supporting state or helper logic inside `MMReceiverBase`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `MMReceiverBase` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 916-918: Method `_process_waiting_requests` signature and setup
```python
    def _process_waiting_requests(self, recv_reqs, waiting_cls):
        new_recv_reqs = []
        for recv_req in recv_reqs:
```
**EN:** This block defines the method `_process_waiting_requests` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_process_waiting_requests`.
**CN:** 这一段定义了method `_process_waiting_requests`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_process_waiting_requests`。

### Lines 919-937: Method `_process_waiting_requests` logic (part 1)
```python
            if (
                isinstance(recv_req, TokenizedGenerateReqInput)
                and recv_req.need_wait_for_mm_inputs is True
            ):
                waiting_req = waiting_cls(
                    rid=recv_req.rid,
                    recv_req=recv_req,
                    mm_processor=self.mm_processor,
                    encoder_urls=self.encode_urls,
                    host_name=self.hostname,
                    receive_count=self.tp_size,
                )
                waiting_req.send_encode_request()
                self.waiting_list.append(waiting_req)
            else:
                new_recv_reqs.append(recv_req)

        if len(self.waiting_list) == 0:
            return new_recv_reqs, []
```
**EN:** This block continues `_process_waiting_requests` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `waiting_cls`, `send_encode_request`, `append`.
**CN:** 这一段延续了 `_process_waiting_requests` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `waiting_cls`、`send_encode_request`、`append`。

### Lines 938-953: Method `_process_waiting_requests` logic (part 2)
```python

        current_time = time.time()
        local_status = []
        for waiting_req in self.waiting_list:
            waiting_req._try_recv_mm_data()
            if current_time - waiting_req.start_time > self.wait_timeout:
                waiting_req.status = WaitingImageRequestStatus.TIMEOUT
            local_status.append(waiting_req.status)

        local_status = torch.tensor(local_status, device="cpu", dtype=torch.int32)

        torch.distributed.all_reduce(
            local_status,
            op=torch.distributed.ReduceOp.MIN,
            group=self.tp_group.cpu_group,
        )
```
**EN:** This block continues `_process_waiting_requests` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `time`, `_try_recv_mm_data`, `append`, `tensor`.
**CN:** 这一段延续了 `_process_waiting_requests` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `time`、`_try_recv_mm_data`、`append`、`tensor`。

### Lines 954-984: Method `_process_waiting_requests` logic (part 3)
```python

        new_waiting = []
        abort_reqs = []
        for i, waiting_req in enumerate(self.waiting_list):
            status_value = local_status[i].item()
            if status_value == WaitingImageRequestStatus.SUCCESS:
                new_recv_reqs.append(waiting_req.recv_req)
            elif status_value == WaitingImageRequestStatus.FAIL:
                logger.error(
                    f"Waiting request {waiting_req.rid} failed: {waiting_req.error_msg} {waiting_req.error_code = }"
                )
                abort_reqs.append(
                    (
                        self.create_req(waiting_req.recv_req),
                        waiting_req.error_msg,
                        waiting_req.error_code,
                    )
                )
            elif status_value == WaitingImageRequestStatus.TIMEOUT:
                logger.error(
                    f"Timed out waiting for image embeddings for request {waiting_req.rid}"
                )
                abort_reqs.append(
                    (
                        self.create_req(waiting_req.recv_req),
                        f"Timeout waiting for image embedding after {self.wait_timeout}s",
                        HTTPStatus.REQUEST_TIMEOUT,
                    )
                )
            else:  # status_value == WaitingImageRequestStatus.PENDING
                new_waiting.append(waiting_req)
```
**EN:** This block continues `_process_waiting_requests` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `item`, `append`, `error`, `create_req`.
**CN:** 这一段延续了 `_process_waiting_requests` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `item`、`append`、`error`、`create_req`。

### Lines 985-987: Method `_process_waiting_requests` logic (part 4)
```python

        self.waiting_list = new_waiting
        return new_recv_reqs, abort_reqs
```
**EN:** This block continues `_process_waiting_requests` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow.
**CN:** 这一段延续了 `_process_waiting_requests` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。

### Lines 989-1004: Method `_run_encode_in_thread`
```python
    def _run_encode_in_thread(
        self, req_id, mm_data, endpoint_encode, num_items_assigned, embedding_port
    ):
        try:
            asyncio.run(
                self.encode(
                    req_id=req_id,
                    mm_data=mm_data,
                    embedding_port=embedding_port,
                    endpoint_encode=endpoint_encode,
                    endpoint_send=None,
                    num_items_assigned=num_items_assigned,
                )
            )
        except Exception as e:
            logger.error(f"Encode failed for request {req_id}: {e}", exc_info=True)
```
**EN:** This block defines the method `_run_encode_in_thread` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_run_encode_in_thread`. Notable operations include `run`, `encode`, `error`.
**CN:** 这一段定义了method `_run_encode_in_thread`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_run_encode_in_thread`。 值得注意的操作包括 `run`、`encode`、`error`。

### Lines 1006-1008: Method `create_req` signature and setup
```python
    def create_req(self, recv_req: TokenizedGenerateReqInput):
        req = Req(
            recv_req.rid,
```
**EN:** This block defines the method `create_req` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `create_req`. Notable operations include `Req`.
**CN:** 这一段定义了method `create_req`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `create_req`。 值得注意的操作包括 `Req`。

### Lines 1009-1032: Method `create_req` logic (part 1)
```python
            recv_req.input_text,
            recv_req.input_ids,
            recv_req.sampling_params,
            return_logprob=recv_req.return_logprob,
            top_logprobs_num=recv_req.top_logprobs_num,
            token_ids_logprob=recv_req.token_ids_logprob,
            stream=recv_req.stream,
            lora_id=recv_req.lora_id,
            input_embeds=recv_req.input_embeds,
            custom_logit_processor=recv_req.custom_logit_processor,
            require_reasoning=recv_req.require_reasoning,
            return_hidden_states=recv_req.return_hidden_states,
            return_routed_experts=recv_req.return_routed_experts,
            routed_experts_start_len=recv_req.routed_experts_start_len,
            eos_token_ids=self.scheduler.model_config.hf_eos_token_id,
            bootstrap_host=recv_req.bootstrap_host,
            bootstrap_port=recv_req.bootstrap_port,
            bootstrap_room=recv_req.bootstrap_room,
            disagg_mode=self.scheduler.disaggregation_mode,
            routed_dp_rank=recv_req.routed_dp_rank,
            disagg_prefill_dp_rank=recv_req.disagg_prefill_dp_rank,
            vocab_size=self.scheduler.model_config.vocab_size,
            priority=recv_req.priority,
            metrics_collector=(
```
**EN:** This block continues `create_req` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow.
**CN:** 这一段延续了 `create_req` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。

### Lines 1033-1041: Method `create_req` logic (part 2)
```python
                self.scheduler.metrics_collector
                if self.scheduler.metrics_reporter.enable_metrics
                else None
            ),
            http_worker_ipc=recv_req.http_worker_ipc,
            dllm_config=self.scheduler.dllm_config,
        )
        req.tokenizer = self.scheduler.tokenizer
        return req
```
**EN:** This block continues `create_req` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow.
**CN:** 这一段延续了 `create_req` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。

### Lines 1043-1050: Method `allocate_embedding_buffer`
```python
    async def allocate_embedding_buffer(self, req_id, total_bytes):
        embeddings = torch.empty(total_bytes, dtype=torch.uint8)
        self.embeddings_engine.register(
            embeddings.data_ptr(),
            embeddings.nbytes,
        )
        self.embeddings_buffer[req_id] = embeddings
        return embeddings.data_ptr()
```
**EN:** This block defines the method `allocate_embedding_buffer` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `allocate_embedding_buffer`. Notable operations include `empty`, `register`, `data_ptr`.
**CN:** 这一段定义了method `allocate_embedding_buffer`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `allocate_embedding_buffer`。 值得注意的操作包括 `empty`、`register`、`data_ptr`。

### Lines 1052-1066: Method `_assign_items_by_modality` signature and setup
```python
    def _assign_items_by_modality(
        self, mm_data, encoder_num, random_shuffle=True
    ) -> Dict:
        """
        Assign multimodal items across encoders by modality with cross-modality load balancing.

        Args:
            mm_data: List of multimodal data items, each with a "modality" key
            encoder_num: Number of encoders
            random_shuffle: Whether to shuffle the encoder indices

        Returns:
            Dictionary mapping modality to list of assignment counts per encoder
            Format: {modality: [count_for_encoder_0, count_for_encoder_1, ...]}
        """
```
**EN:** This block defines the method `_assign_items_by_modality` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_assign_items_by_modality`.
**CN:** 这一段定义了method `_assign_items_by_modality`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_assign_items_by_modality`。

### Lines 1067-1094: Method `_assign_items_by_modality` logic (part 1)
```python
        encode_idx = list(range(encoder_num))
        if random_shuffle:
            random.shuffle(encode_idx)
        # Get unique modalities with order preserved
        modalities = list(dict.fromkeys(mm_item.get("modality") for mm_item in mm_data))
        # Use OrderedDict to explicitly maintain modality order
        num_items_assigned = OrderedDict()
        current_offset = 0

        for modality in modalities:
            mm_data_modality = [
                mm_item for mm_item in mm_data if mm_item.get("modality") == modality
            ]
            num_items = len(mm_data_modality)
            if num_items == 0:
                continue

            base = num_items // len(encode_idx)
            remainder = num_items % len(encode_idx)
            # Rotate assignments based on current_offset to balance load across modalities
            assignments = [0] * len(encode_idx)
            for i in range(len(encode_idx)):
                # keep shuffle order when assigning items to encoders
                pos_in_shuffled = (current_offset + i) % len(encode_idx)
                actual_encoder_idx = encode_idx[pos_in_shuffled]
                assignments[actual_encoder_idx] = base + (1 if i < remainder else 0)
            num_items_assigned[modality] = assignments
            current_offset = (current_offset + remainder) % len(encode_idx)
```
**EN:** This block continues `_assign_items_by_modality` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `shuffle`, `fromkeys`, `get`, `OrderedDict`.
**CN:** 这一段延续了 `_assign_items_by_modality` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `shuffle`、`fromkeys`、`get`、`OrderedDict`。

### Lines 1095-1096: Method `_assign_items_by_modality` logic (part 2)
```python

        return num_items_assigned
```
**EN:** This block continues `_assign_items_by_modality` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow.
**CN:** 这一段延续了 `_assign_items_by_modality` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。

### Lines 1098-1100: Method `_extract_url_data` signature and setup
```python
    def _extract_url_data(self, request_obj) -> List[Dict]:
        def flatten_mm_items(items):
            if not isinstance(items, list):
```
**EN:** This block defines the method `_extract_url_data` on `MMReceiverBase`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_extract_url_data`, `flatten_mm_items`.
**CN:** 这一段定义了method `_extract_url_data`（属于 `MMReceiverBase`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_extract_url_data`、`flatten_mm_items`。

### Lines 1101-1117: Method `_extract_url_data` logic (part 1)
```python
                return [items]

            flat = []
            for item in items:
                if isinstance(item, (list, tuple)):
                    flat.extend(flatten_mm_items(list(item)))
                else:
                    flat.append(item)
            return flat

        def to_raw_url(mm_item):
            if isinstance(mm_item, ImageData):
                return mm_item.url
            if isinstance(mm_item, dict):
                # tolerate {"url": ...} shaped payloads
                return mm_item.get("url", mm_item)
            return mm_item
```
**EN:** This block continues `_extract_url_data` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Definitions introduced here include `to_raw_url`. Notable operations include `extend`, `flatten_mm_items`, `append`, `get`.
**CN:** 这一段延续了 `_extract_url_data` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 此处引入的定义包括 `to_raw_url`。 值得注意的操作包括 `extend`、`flatten_mm_items`、`append`、`get`。

### Lines 1118-1135: Method `_extract_url_data` logic (part 2)
```python

        mm_data = []
        for attr, modality in [
            ("image_data", Modality.IMAGE),
            ("video_data", Modality.VIDEO),
            ("audio_data", Modality.AUDIO),
        ]:
            mm_items = getattr(request_obj, attr, None)
            if mm_items:
                mm_items = flatten_mm_items(mm_items)
                for mm_item in mm_items:
                    mm_data.append(
                        {
                            "url": to_raw_url(mm_item),
                            "modality": modality,
                        }
                    )
        return mm_data
```
**EN:** This block continues `_extract_url_data` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `flatten_mm_items`, `append`, `to_raw_url`.
**CN:** 这一段延续了 `_extract_url_data` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `flatten_mm_items`、`append`、`to_raw_url`。

### Lines 1138-1138: Class `MMReceiverHTTP` declaration
```python
class MMReceiverHTTP(MMReceiverBase):
```
**EN:** This block declares the class `MMReceiverHTTP` and establishes its responsibility inside encode-side transfer receiving workflow. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `MMReceiverHTTP`.
**CN:** 这一段声明了类 `MMReceiverHTTP`，并说明它在编码侧传输接收流程中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `MMReceiverHTTP`。

### Lines 1139-1157: Method `__init__`
```python
    def __init__(
        self,
        server_args: ServerArgs,
        dtype: Optional[torch.dtype] = None,
        hf_config: Optional[PretrainedConfig] = None,
        pp_rank: Optional[int] = None,
        tp_rank: Optional[int] = None,
        tp_group: Optional[GroupCoordinator] = None,
        scheduler: Optional["Scheduler"] = None,
    ):
        super().__init__(
            server_args,
            dtype=dtype,
            hf_config=hf_config,
            pp_rank=pp_rank,
            tp_rank=tp_rank,
            tp_group=tp_group,
            scheduler=scheduler,
        )
```
**EN:** This block defines the method `__init__` on `MMReceiverHTTP`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `MMReceiverHTTP`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 1158-1159: Supporting state inside `MMReceiverHTTP`
```python

    # For zmq_to_scheduler
```
**EN:** This block adds supporting state or helper logic inside `MMReceiverHTTP`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `MMReceiverHTTP` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 1160-1161: Method `process_waiting_requests`
```python
    def process_waiting_requests(self, recv_reqs):
        return self._process_waiting_requests(recv_reqs, WaitingImageRequest)
```
**EN:** This block defines the method `process_waiting_requests` on `MMReceiverHTTP`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `process_waiting_requests`. Notable operations include `_process_waiting_requests`.
**CN:** 这一段定义了method `process_waiting_requests`（属于 `MMReceiverHTTP`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `process_waiting_requests`。 值得注意的操作包括 `_process_waiting_requests`。

### Lines 1163-1165: Method `encode` signature and setup
```python
    async def encode(
        self,
        req_id,
```
**EN:** This block defines the method `encode` on `MMReceiverHTTP`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `encode`.
**CN:** 这一段定义了method `encode`（属于 `MMReceiverHTTP`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `encode`。

### Lines 1166-1195: Method `encode` logic (part 1)
```python
        mm_data,
        embedding_port,
        endpoint_encode,
        endpoint_send,
        num_items_assigned=None,
    ):
        if len(mm_data) == 0:
            return

        # get unique modalities with order preserved
        modalities = [mm_item.get("modality") for mm_item in mm_data]
        modalities = list(dict.fromkeys(modalities))
        encode_requests = []

        if num_items_assigned is None:
            num_items_assigned = self._assign_items_by_modality(
                mm_data, len(self.encode_urls)
            )

        # Calculate total num_parts across all modalities
        total_num_parts, modality_num_parts = calculate_modality_num_parts(
            modalities, num_items_assigned
        )

        part_idx_offset = 0
        for modality in modalities:
            num_items_assigned_modality = num_items_assigned.get(modality)
            mm_data_modality = [
                mm_item for mm_item in mm_data if mm_item.get("modality") == modality
            ]
```
**EN:** This block continues `encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `get`, `fromkeys`, `_assign_items_by_modality`, `calculate_modality_num_parts`.
**CN:** 这一段延续了 `encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `get`、`fromkeys`、`_assign_items_by_modality`、`calculate_modality_num_parts`。

### Lines 1196-1224: Method `encode` logic (part 2)
```python

            num_parts = modality_num_parts[modality]
            cum_num_items = 0
            cum_idx = 0
            for idx, assigned_num in enumerate(num_items_assigned_modality):
                if assigned_num == 0:
                    continue
                part_idx = part_idx_offset + cum_idx
                part_req_id = create_part_req_id(req_id, part_idx)
                encode_requests.append(
                    {
                        "encoder_idx": idx,
                        "mm_items": [
                            mm_item.get("url")
                            for mm_item in mm_data_modality[
                                cum_num_items : cum_num_items + assigned_num
                            ]
                        ],
                        "num_parts": total_num_parts,
                        "part_idx": part_idx,
                        "req_id": part_req_id,  # use part_req_id to avoid key collision
                        "modality": modality.name,  # convert enum to string for json serialization
                        "prefill_host": self.host,
                        "embedding_port": embedding_port,
                    }
                )
                cum_idx += 1
                cum_num_items += assigned_num
            part_idx_offset += num_parts
```
**EN:** This block continues `encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `create_part_req_id`, `append`, `get`.
**CN:** 这一段延续了 `encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `create_part_req_id`、`append`、`get`。

### Lines 1225-1254: Method `encode` logic (part 3)
```python

        async with aiohttp.ClientSession(
            timeout=aiohttp.ClientTimeout(
                total=1800
            )  # Add timeout for request reliability
        ) as session:
            # Send encode requests

            tasks = [
                session.post(
                    f"{self.encode_urls[encode_request['encoder_idx']]}/{endpoint_encode}",
                    json=encode_request,
                )
                for encode_request in encode_requests
            ]

            responses = await asyncio.gather(*tasks)
            for response in responses:
                if response.status != 200:
                    try:
                        err_data = await response.json()
                        msg = err_data.get("message", "Unknown encoder error")
                    except:
                        msg = await response.text()

                    logger.error(f"Encoder returned error {response.status}: {msg}")
                    return
            response_json_list_unsort = [
                await response.json() for response in responses
            ]
```
**EN:** This block continues `encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `ClientSession`, `ClientTimeout`, `post`, `gather`.
**CN:** 这一段延续了 `encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `ClientSession`、`ClientTimeout`、`post`、`gather`。

### Lines 1255-1286: Method `encode` logic (part 4)
```python

            # zmq backend: return is None
            if None in response_json_list_unsort:
                return

            # mooncake backend: send bootstrap info

            embedding_size_list_sort = [None for _ in range(total_num_parts)]
            response_json_list_sort = [None for _ in range(total_num_parts)]
            for response_json in response_json_list_unsort:
                idx = response_json["part_idx"]
                embedding_size_list_sort[idx] = response_json["embedding_size"]
                response_json_list_sort[idx] = response_json

            total_embedding_bytes = sum(
                s for s in embedding_size_list_sort if s is not None
            )
            offset = 0
            metadata_tasks = []
            buffer_address = await self.allocate_embedding_buffer(
                req_id,
                total_embedding_bytes,
            )
            for idx in range(len(tasks)):
                response_json = response_json_list_sort[idx]
                buffer_address_adjust = offset + buffer_address
                response_json.update(
                    {
                        "session_id": self.embeddings_engine.session_id,
                        "buffer_address": buffer_address_adjust,
                    }
                )
```
**EN:** This block continues `encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `allocate_embedding_buffer`, `update`.
**CN:** 这一段延续了 `encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `allocate_embedding_buffer`、`update`。

### Lines 1287-1294: Method `encode` logic (part 5)
```python
                metadata_tasks.append(
                    session.post(
                        f"{self.encode_urls[response_json['encoder_idx']]}/{endpoint_send}",
                        json=response_json,
                    )
                )
                offset += embedding_size_list_sort[idx]
            await asyncio.gather(*metadata_tasks)
```
**EN:** This block continues `encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `append`, `post`, `gather`.
**CN:** 这一段延续了 `encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `append`、`post`、`gather`。

### Lines 1297-1297: Class `MMReceiverGrpc` declaration
```python
class MMReceiverGrpc(MMReceiverBase):
```
**EN:** This block declares the class `MMReceiverGrpc` and establishes its responsibility inside encode-side transfer receiving workflow. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `MMReceiverGrpc`.
**CN:** 这一段声明了类 `MMReceiverGrpc`，并说明它在编码侧传输接收流程中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `MMReceiverGrpc`。

### Lines 1298-1316: Method `__init__`
```python
    def __init__(
        self,
        server_args: ServerArgs,
        dtype: Optional[torch.dtype] = None,
        hf_config: Optional[PretrainedConfig] = None,
        pp_rank: Optional[int] = None,
        tp_rank: Optional[int] = None,
        tp_group: Optional[GroupCoordinator] = None,
        scheduler: Optional["Scheduler"] = None,
    ):
        super().__init__(
            server_args,
            dtype=dtype,
            hf_config=hf_config,
            pp_rank=pp_rank,
            tp_rank=tp_rank,
            tp_group=tp_group,
            scheduler=scheduler,
        )
```
**EN:** This block defines the method `__init__` on `MMReceiverGrpc`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `__init__`. Notable operations include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `MMReceiverGrpc`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `__init__`。

### Lines 1318-1324: Method `build_and_send_encode_request`
```python
    def build_and_send_encode_request(self, image_urls, rid):
        encode_req = GenerateReqInput(
            image_data=[ImageData(url=url) for url in image_urls],
            rid=rid,
        )
        self.send_encode_request(encode_req)
        return encode_req
```
**EN:** This block defines the method `build_and_send_encode_request` on `MMReceiverGrpc`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `build_and_send_encode_request`. Notable operations include `GenerateReqInput`, `ImageData`, `send_encode_request`.
**CN:** 这一段定义了method `build_and_send_encode_request`（属于 `MMReceiverGrpc`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `build_and_send_encode_request`。 值得注意的操作包括 `GenerateReqInput`、`ImageData`、`send_encode_request`。

### Lines 1325-1326: Supporting state inside `MMReceiverGrpc`
```python

    # For zmq_to_scheduler
```
**EN:** This block adds supporting state or helper logic inside `MMReceiverGrpc`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `MMReceiverGrpc` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 1327-1328: Method `process_waiting_requests`
```python
    def process_waiting_requests(self, recv_reqs):
        return self._process_waiting_requests(recv_reqs, WaitingImageRequestGrpc)
```
**EN:** This block defines the method `process_waiting_requests` on `MMReceiverGrpc`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `process_waiting_requests`. Notable operations include `_process_waiting_requests`.
**CN:** 这一段定义了method `process_waiting_requests`（属于 `MMReceiverGrpc`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `process_waiting_requests`。 值得注意的操作包括 `_process_waiting_requests`。

### Lines 1330-1332: Method `encode` signature and setup
```python
    async def encode(
        self,
        req_id,
```
**EN:** This block defines the method `encode` on `MMReceiverGrpc`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `encode`.
**CN:** 这一段定义了method `encode`（属于 `MMReceiverGrpc`），介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `encode`。

### Lines 1333-1357: Method `encode` logic (part 1)
```python
        mm_data,
        embedding_port,
        endpoint_encode,
        endpoint_send,
        num_items_assigned=None,
    ):
        if not mm_data:
            return

        # gRPC currently only supports image; flatten new dict formats to simple lists
        if mm_data and isinstance(mm_data[0], dict):
            non_image = [
                item.get("modality")
                for item in mm_data
                if item.get("modality") != Modality.IMAGE
            ]
            if non_image:
                raise NotImplementedError(
                    f"gRPC encode only supports IMAGE modality, got: {non_image}"
                )
            img_data = [item.get("url") for item in mm_data]
        else:
            img_data = mm_data
        if isinstance(num_items_assigned, dict):
            num_items_assigned = list(num_items_assigned.values())[0]
```
**EN:** This block continues `encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `get`, `NotImplementedError`, `values`.
**CN:** 这一段延续了 `encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `get`、`NotImplementedError`、`values`。

### Lines 1358-1386: Method `encode` logic (part 2)
```python

        encode_requests = []
        if num_items_assigned is None:
            encode_idx = list(range(len(self.encode_urls)))
            random.shuffle(encode_idx)
            num_items_assigned = [
                (idx + len(img_data)) // len(self.encode_urls) for idx in encode_idx
            ]
        num_parts = sum(1 for x in num_items_assigned if x != 0)
        cum_num_items = 0
        cum_idx = 0
        for idx, assigned_num in enumerate(num_items_assigned):
            if assigned_num == 0:
                continue
            start = cum_num_items
            end = cum_num_items + assigned_num
            encode_requests.append(
                {
                    "encoder_idx": idx,
                    "mm_items": img_data[start:end],
                    "num_parts": num_parts,
                    "part_idx": cum_idx,
                    "req_id": req_id,
                    "prefill_host": self.host,
                    "embedding_port": embedding_port,
                }
            )
            cum_idx += 1
            cum_num_items += assigned_num
```
**EN:** This block continues `encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `shuffle`, `append`.
**CN:** 这一段延续了 `encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `shuffle`、`append`。

### Lines 1387-1416: Method `encode` logic (part 3)
```python

        grpc_tasks = [
            asyncio.to_thread(
                _grpc_encode_request,
                _grpc_target(self.encode_urls[encode_request["encoder_idx"]]),
                encode_request,
            )
            for encode_request in encode_requests
        ]
        grpc_responses = await asyncio.gather(*grpc_tasks)
        response_json_unsorted = []
        for encode_request, response in zip(encode_requests, grpc_responses):
            if self.encoder_transfer_backend == "zmq_to_scheduler":
                response_json_unsorted.append(None)
                continue
            response_json_unsorted.append(
                {
                    "req_id": encode_request["req_id"],
                    "prefill_host": encode_request["prefill_host"],
                    "embedding_port": encode_request["embedding_port"],
                    "encoder_idx": encode_request["encoder_idx"],
                    "part_idx": encode_request["part_idx"],
                    "embedding_size": response.embedding_size,
                    "embedding_len": response.embedding_len,
                    "embedding_dim": response.embedding_dim,
                }
            )

        if None in response_json_unsorted:
            return
```
**EN:** This block continues `encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `to_thread`, `_grpc_target`, `gather`, `append`.
**CN:** 这一段延续了 `encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `to_thread`、`_grpc_target`、`gather`、`append`。

### Lines 1417-1446: Method `encode` logic (part 4)
```python

        embedding_size_by_part = [None for _ in range(num_parts)]
        response_json_sorted = [None for _ in range(num_parts)]
        for response_json in response_json_unsorted:
            idx = response_json["part_idx"]
            embedding_size_by_part[idx] = response_json["embedding_size"]
            response_json_sorted[idx] = response_json

        total_embedding_bytes = sum(s for s in embedding_size_by_part if s is not None)
        offset = 0
        buffer_address = await self.allocate_embedding_buffer(
            req_id,
            total_embedding_bytes,
        )
        grpc_metadata_tasks = []
        for response_json in response_json_sorted:
            response_json.update(
                {
                    "session_id": self.embeddings_engine.session_id,
                    "buffer_address": offset + buffer_address,
                }
            )
            grpc_metadata_tasks.append(
                asyncio.to_thread(
                    _grpc_send_request,
                    _grpc_target(self.encode_urls[response_json["encoder_idx"]]),
                    response_json,
                )
            )
            offset += embedding_size_by_part[response_json["part_idx"]]
```
**EN:** This block continues `encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `allocate_embedding_buffer`, `update`, `append`, `to_thread`.
**CN:** 这一段延续了 `encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `allocate_embedding_buffer`、`update`、`append`、`to_thread`。

### Lines 1447-1449: Method `encode` logic (part 5)
```python

        if grpc_metadata_tasks:
            await asyncio.gather(*grpc_metadata_tasks)
```
**EN:** This block continues `encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `gather`.
**CN:** 这一段延续了 `encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `gather`。

### Lines 1452-1469: Function `_validate_transport_mode`
```python
def _validate_transport_mode(transport_mode: str, encoder_urls):
    if transport_mode == "grpc":
        invalid_prefix = "http://"
        error_msg = (
            "EPD MMReceiver: grpc mode requires grpc:// encoder URLs. "
            "Set SGLANG_ENCODER_MM_RECEIVER_MODE=http for http:// URLs."
        )
    elif transport_mode == "http":
        invalid_prefix = "grpc://"
        error_msg = (
            "EPD MMReceiver: http mode requires http:// encoder URLs. "
            "Set SGLANG_ENCODER_MM_RECEIVER_MODE=grpc for grpc:// URLs."
        )
    else:
        return

    if any(url.startswith(invalid_prefix) for url in encoder_urls):
        raise ValueError(error_msg)
```
**EN:** This block defines the function `_validate_transport_mode`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `_validate_transport_mode`. Notable operations include `startswith`, `ValueError`.
**CN:** 这一段定义了function `_validate_transport_mode`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `_validate_transport_mode`。 值得注意的操作包括 `startswith`、`ValueError`。

### Lines 1470-1477: Module-level constants and helper logic
```python


_MM_RECEIVER_BY_MODE = {
    "grpc": MMReceiverGrpc,
    "http": MMReceiverHTTP,
}


```
**EN:** This block contains module-level constants, helpers, or documentation for encode-side transfer receiving workflow. It prepares shared state that later classes and functions build on.
**CN:** 这一段包含与编码侧传输接收流程相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。

### Lines 1478-1480: Function `create_mm_receiver` signature and setup
```python
def create_mm_receiver(
    server_args: ServerArgs,
    dtype: Optional[torch.dtype] = None,
```
**EN:** This block defines the function `create_mm_receiver`. It introduces the parameters, setup steps, and the main entry point for this piece of encode-side transfer receiving workflow. Definitions introduced here include `create_mm_receiver`.
**CN:** 这一段定义了function `create_mm_receiver`，介绍了参数、初始化步骤，以及这部分编码侧传输接收流程逻辑的主要入口。 此处引入的定义包括 `create_mm_receiver`。

### Lines 1481-1504: Function `create_mm_receiver` logic (part 1)
```python
    hf_config: Optional[PretrainedConfig] = None,
    pp_rank: Optional[int] = None,
    tp_rank: Optional[int] = None,
    tp_group: Optional[GroupCoordinator] = None,
    scheduler: Optional["Scheduler"] = None,
    transport_mode: Optional[str] = None,
):
    if transport_mode is None:
        transport_mode = envs.SGLANG_ENCODER_MM_RECEIVER_MODE.get()
        logger.debug(f"MMReceiver transport_mode from env: {transport_mode}")

    _validate_transport_mode(transport_mode, server_args.encoder_urls)
    logger.info(f"EPD MMReceiver: using transport_mode={transport_mode}")

    receiver_cls = _MM_RECEIVER_BY_MODE.get(transport_mode)
    if receiver_cls is None:
        raise ValueError(f"Unsupported transport_mode: {transport_mode}")
    return receiver_cls(
        server_args,
        dtype=dtype,
        hf_config=hf_config,
        pp_rank=pp_rank,
        tp_rank=tp_rank,
        tp_group=tp_group,
```
**EN:** This block continues `create_mm_receiver` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow. Notable operations include `get`, `debug`, `_validate_transport_mode`, `info`.
**CN:** 这一段延续了 `create_mm_receiver` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。 值得注意的操作包括 `get`、`debug`、`_validate_transport_mode`、`info`。

### Lines 1505-1506: Function `create_mm_receiver` logic (part 2)
```python
        scheduler=scheduler,
    )
```
**EN:** This block continues `create_mm_receiver` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding encode-side transfer receiving workflow workflow.
**CN:** 这一段延续了 `create_mm_receiver` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输接收流程工作流。

## Key Concepts / 关键概念
- `_grpc_target`: Function that performs grpc target for the surrounding workflow. / `_grpc_target`：在周边工作流中执行“gRPCtarget”相关任务的函数。
- `_normalize_embedding_ports`: Function that performs normalize embedding ports for the surrounding workflow. / `_normalize_embedding_ports`：在周边工作流中执行“normalizeembeddingports”相关任务的函数。
- `_grpc_scheduler_receive_url`: Function that performs grpc scheduler receive url for the surrounding workflow. / `_grpc_scheduler_receive_url`：在周边工作流中执行“gRPCschedulerreceiveurl”相关任务的函数。
- `_grpc_encode_request`: Function that performs grpc encode request for the surrounding workflow. / `_grpc_encode_request`：在周边工作流中执行“gRPC编码请求”相关任务的函数。
- `_grpc_send_request`: Function that performs grpc send request for the surrounding workflow. / `_grpc_send_request`：在周边工作流中执行“gRPCsend请求”相关任务的函数。
- `EmbeddingData`: Class that encapsulates embedding data behavior in this module. / `EmbeddingData`：封装与“embeddingdata”相关行为的类。
- `video_meta_attrs_for`: Function that performs video meta attrs for for the surrounding workflow. / `video_meta_attrs_for`：在周边工作流中执行“videometaattrsfor”相关任务的函数。
- `_cat_grid`: Function that performs cat grid for the surrounding workflow. / `_cat_grid`：在周边工作流中执行“catgrid”相关任务的函数。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `itertools`, `logging`, `pickle`, `random`, `threading`, `time`, `uuid`, `abc`, `collections`, `enum`, `http`, `typing`
- **External packages / 外部依赖**: `aiohttp`, `numpy`, `torch`, `zmq`, `transformers`, `grpc`, `smg_grpc_proto`
- **Internal modules / 内部模块**: `sglang.srt.distributed.parallel_state`, `sglang.srt.environ`, `sglang.srt.managers.io_struct`, `sglang.srt.managers.multimodal_processor`, `sglang.srt.managers.schedule_batch`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.srt.utils.network`, `sglang.srt.managers.scheduler`, `sglang.srt.distributed.device_communicators.mooncake_transfer_engine`
