# scheduler_mixin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/scheduler_mixin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `SchedulerDisaggMixin`, `_is_tensor_like`, and `_to_json_serializable`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Mixin that adds disaggregated diffusion scheduling to the Scheduler. Extracted from scheduler.py to keep the core scheduler lean. / 该文件属于解耦运行时层。它围绕 `SchedulerDisaggMixin`、`_is_tensor_like` 和 `_to_json_serializable` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-58: module setup and imports / 模块初始化与导入
```python
"""Mixin that adds disaggregated diffusion scheduling to the Scheduler.

Extracted from scheduler.py to keep the core scheduler lean.
All transfer, compute, and event-loop logic for disaggregated roles
(encoder / denoiser / decoder) lives here.
"""

from __future__ import annotations

import contextlib
import dataclasses
import inspect
import json
import logging
import pickle
import queue
import threading
import time
from typing import TYPE_CHECKING, Any

import torch
import zmq

from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
from sglang.multimodal_gen.runtime.disaggregation.roles import RoleType
from sglang.multimodal_gen.runtime.disaggregation.transport.buffer import (
    TransferTensorBuffer,
)
from sglang.multimodal_gen.runtime.disaggregation.transport.codec import (
    send_tensors,
)
from sglang.multimodal_gen.runtime.disaggregation.transport.engine import (
    create_transfer_engine,
)
from sglang.multimodal_gen.runtime.disaggregation.transport.manager import (
    DiffusionTransferManager,
)
from sglang.multimodal_gen.runtime.disaggregation.transport.protocol import (
    TRANSFER_MAGIC,
    TransferAllocatedMsg,
    TransferDoneMsg,
    TransferMsgType,
    TransferPushedMsg,
    TransferRegisterMsg,
    decode_transfer_msg,
    encode_transfer_msg,
    is_transfer_message,
)
from sglang.multimodal_gen.runtime.pipelines_core import Req
from sglang.multimodal_gen.runtime.pipelines_core.diffusion_scheduler_utils import (
    clone_scheduler_runtime,
)
from sglang.multimodal_gen.runtime.utils.common import get_zmq_socket
from sglang.multimodal_gen.runtime.utils.distributed import broadcast_pyobj
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.runtime.utils.trace_wrapper import DiffStage, trace_slice
from sglang.srt.observability.trace import TraceReqContext
```
**EN:** This block establishes the module context and imports `__future__`, `contextlib`, `dataclasses`, `inspect`, `json`, and `logging`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`contextlib`、`dataclasses`、`inspect`、`json` 和 `logging`。这些依赖为后续实现提供所需符号。

### Lines 60-127: supporting statements / 辅助语句
```python
if TYPE_CHECKING:
    from sglang.multimodal_gen.runtime.managers.scheduler import Scheduler

logger = init_logger(__name__)

# ---------------------------------------------------------------------------
# Field extraction: split Req into tensors (transfer buffer) and scalars (JSON)
# ---------------------------------------------------------------------------

# Fields that should never be transferred (non-serializable, internal, or receiver rebuilds)
_EXCLUDE_FIELDS = frozenset(
    {
        "sampling_params",
        "generator",
        "modules",
        "metrics",
        "extra_step_kwargs",
        "extra",
        "condition_image",
        "vae_image",
        "pixel_values",
        "preprocessed_image",
        "image_embeds",
        "original_condition_image_size",
        "vae_image_sizes",
        "output",
        "audio",
        "audio_sample_rate",
        "trajectory_timesteps",
        "trajectory_latents",
        "trajectory_audio_latents",
        "timestep",
        "step_index",
        # Request scheduler is a local runtime object cloned from the pipeline
        # scheduler template. It may hold live mutable state and is not JSON-safe.
        "scheduler",
        "prompt_template",
        "max_sequence_length",
        # trace_ctx holds live OTel SDK objects that aren't JSON-serializable.
        # We propagate tracing across the JSON hop via a separate, JSON-safe
        # ``_trace_state`` scalar field built from ``TraceReqContext.__getstate__``
        # (same W3C carrier SRT relies on for pickle transport) and rebuild it
        # on the receiver in ``_build_disagg_req``.
        "trace_ctx",
    }
)

# Sampling-params fields that should never be transferred across roles:
#   - data_type / supported_resolutions: enums / non-JSON classvars reconstructed on the receiver
#   - teacache_params: model-specific object, not JSON-safe
#   - output_* / save_output / return_*: output-side concerns owned by the decoder role
#
# Everything else on SamplingParams is forwarded automatically via a field-walk
# below; this keeps new request-level features (e.g. Qwen-Image's
# true_cfg_scale, guidance_rescale, cfg_normalization, ...) from silently
# getting dropped just because nobody remembered to add them to a whitelist.
_SAMPLING_PARAMS_EXCLUDE_FIELDS = frozenset(
    {
        "data_type",
        "supported_resolutions",
        "teacache_params",
    }
)

_BASE_SP_DEFAULTS: dict[str, Any] = {}
for _f in dataclasses.fields(SamplingParams):
    if _f.default is not dataclasses.MISSING:
        _BASE_SP_DEFAULTS[_f.name] = _f.default
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, `_EXCLUDE_FIELDS`, `_SAMPLING_PARAMS_EXCLUDE_FIELDS`, and `_BASE_SP_DEFAULTS`. The code collaborates with `init_logger`, `frozenset`, and `dataclasses.fields`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger`、`_EXCLUDE_FIELDS`、`_SAMPLING_PARAMS_EXCLUDE_FIELDS` 和 `_BASE_SP_DEFAULTS` 等名称。 代码会与 `init_logger`、`frozenset` 和 `dataclasses.fields` 协同工作。

### Lines 130-135: `_is_tensor_like` implementation / `_is_tensor_like` 实现
```python
def _is_tensor_like(value) -> bool:
    if isinstance(value, torch.Tensor):
        return True
    if isinstance(value, list) and value and isinstance(value[0], torch.Tensor):
        return True
    return False
```
**EN:** This block defines function `_is_tensor_like`. It handles is tensor like logic. Key calls include `isinstance`. The implementation branches on conditions. Parameters such as `value` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_is_tensor_like`。 它用于处理 is tensor like 相关逻辑。 关键调用包括 `isinstance`。 实现中包含条件分支。 本段逻辑主要由 `value` 等参数驱动。

### Lines 138-149: `_to_json_serializable` implementation / `_to_json_serializable` 实现
```python
def _to_json_serializable(value):
    if isinstance(value, torch.Tensor):
        return value.tolist()
    if isinstance(value, (list, tuple)):
        converted = []
        for item in value:
            if isinstance(item, torch.Tensor):
                converted.append(item.tolist())
            else:
                converted.append(item)
        return converted
    return value
```
**EN:** This block defines function `_to_json_serializable`. It converts to json serializable. Key calls include `isinstance`, `value.tolist`, `converted.append`, and `item.tolist`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `value` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_to_json_serializable`。 它用于转换为json serializable。 关键调用包括 `isinstance`、`value.tolist`、`converted.append` 和 `item.tolist`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `value` 等参数驱动。

### Lines 152-158: `_is_default` implementation / `_is_default` 实现
```python
def _is_default(value, field_info) -> bool:
    if field_info.default is not dataclasses.MISSING:
        return value == field_info.default
    if field_info.default_factory is not dataclasses.MISSING:
        if isinstance(value, (list, dict)) and len(value) == 0:
            return True
    return False
```
**EN:** This block defines function `_is_default`. It handles is default logic. Key calls include `isinstance`, and `len`. The implementation branches on conditions. Parameters such as `value`, and `field_info` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_is_default`。 它用于处理 is default 相关逻辑。 关键调用包括 `isinstance` 和 `len`。 实现中包含条件分支。 本段逻辑主要由 `value` 和 `field_info` 等参数驱动。

### Lines 161-170: `_extract_extra_fields` implementation / `_extract_extra_fields` 实现
```python
def _extract_extra_fields(extra: dict, scalar_fields: dict) -> None:
    """Extract JSON-serializable entries from Req.extra into scalar_fields."""
    for key, value in extra.items():
        if key.startswith("_"):
            continue
        try:
            json.dumps(value)
            scalar_fields[f"_extra_{key}"] = value
        except (TypeError, ValueError, OverflowError):
            pass
```
**EN:** This block defines function `_extract_extra_fields`. Extract JSON-serializable entries from Req.extra into scalar_fields. Key calls include `extra.items`, `key.startswith`, and `json.dumps`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `extra`, and `scalar_fields` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_extract_extra_fields`。 它用于处理 extract extra fields 相关逻辑。 关键调用包括 `extra.items`、`key.startswith` 和 `json.dumps`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `extra` 和 `scalar_fields` 等参数驱动。

### Lines 173-201: `_init_request_scheduler_from_template` implementation / `_init_request_scheduler_from_template` 实现
```python
def _init_request_scheduler_from_template(
    scheduler_template: Any, req: Req, device: torch.device
) -> None:
    scheduler = clone_scheduler_runtime(scheduler_template)
    extra_kwargs = {}
    mu = req.extra.get("mu") if hasattr(req, "extra") else None
    if mu is not None:
        extra_kwargs["mu"] = mu

    set_timesteps_params = inspect.signature(scheduler.set_timesteps).parameters
    timesteps = getattr(req, "timesteps", None)
    sigmas = getattr(req, "sigmas", None)
    num_steps = getattr(req, "num_inference_steps", None)

    if sigmas is not None and "sigmas" in set_timesteps_params:
        if isinstance(sigmas, torch.Tensor):
            sigmas = sigmas.detach().cpu()
        scheduler.set_timesteps(sigmas=sigmas, device=device, **extra_kwargs)
    elif timesteps is not None and "timesteps" in set_timesteps_params:
        if isinstance(timesteps, torch.Tensor):
            timesteps = timesteps.detach().cpu()
        scheduler.set_timesteps(timesteps=timesteps, device=device, **extra_kwargs)
    elif num_steps is not None:
        scheduler.set_timesteps(num_steps, device=device, **extra_kwargs)
    else:
        return

    req.scheduler = scheduler
    req.timesteps = scheduler.timesteps
```
**EN:** This block defines function `_init_request_scheduler_from_template`. It initializes request scheduler from template. Key calls include `clone_scheduler_runtime`, `getattr`, `hasattr`, `req.extra.get`, and `inspect.signature`. The implementation branches on conditions. Parameters such as `scheduler_template`, `req`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_init_request_scheduler_from_template`。 它用于初始化request scheduler from template。 关键调用包括 `clone_scheduler_runtime`、`getattr`、`hasattr`、`req.extra.get` 和 `inspect.signature`。 实现中包含条件分支。 本段逻辑主要由 `scheduler_template`、`req` 和 `device` 等参数驱动。

### Lines 204-209: `_init_disagg_request_scheduler` implementation / `_init_disagg_request_scheduler` 实现
```python
def _init_disagg_request_scheduler(self: Scheduler, req: Req) -> None:
    scheduler_template = self.worker.pipeline.get_module("scheduler")
    if scheduler_template is None:
        return
    device = torch.device(f"cuda:{self.worker.local_rank}")
    _init_request_scheduler_from_template(scheduler_template, req, device)
```
**EN:** This block defines function `_init_disagg_request_scheduler`. It initializes disagg request scheduler. Key calls include `self.worker.pipeline.get_module`, `torch.device`, and `_init_request_scheduler_from_template`. The implementation branches on conditions. Parameters such as `req` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_init_disagg_request_scheduler`。 它用于初始化disagg request scheduler。 关键调用包括 `self.worker.pipeline.get_module`、`torch.device` 和 `_init_request_scheduler_from_template`。 实现中包含条件分支。 本段逻辑主要由 `req` 等参数驱动。

### Lines 212-310: `extract_transfer_fields` implementation / `extract_transfer_fields` 实现
```python
def extract_transfer_fields(req) -> tuple[dict, dict]:
    """Extract all transferable fields from a Req, split into tensors and scalars."""
    tensor_fields = {}
    scalar_fields = {}
    _debug_transfer = logger.isEnabledFor(logging.DEBUG)

    for f in dataclasses.fields(req):
        if f.name in _EXCLUDE_FIELDS:
            continue

        value = getattr(req, f.name, None)
        if value is None:
            continue
        if _is_default(value, f):
            continue

        if _is_tensor_like(value):
            tensor_fields[f.name] = value
        else:
            try:
                scalar_fields[f.name] = _to_json_serializable(value)
            except (TypeError, ValueError):
                pass

    extra = getattr(req, "extra", None)
    if extra:
        _extract_extra_fields(extra, scalar_fields)

    sp = getattr(req, "sampling_params", None)
    if sp is not None:
        # Forward every non-default, JSON-safe SamplingParams field, not a
        # narrow whitelist. Previously only a handful of fields were carried
        # across roles, which silently dropped per-request config like
        # Qwen-Image's true_cfg_scale (and any future feature added to
        # SamplingParams). Using a field-walk keeps the disagg boundary
        # feature-complete without needing to edit this list.
        for f in dataclasses.fields(sp):
            name = f.name
            if name in _SAMPLING_PARAMS_EXCLUDE_FIELDS:
                continue
            if name in scalar_fields:
                # Req-level field already took precedence (or upstream Req
                # explicitly set it).
                continue
            value = getattr(sp, name, None)
            if value is None:
                continue
            base_default = _BASE_SP_DEFAULTS.get(name, dataclasses.MISSING)
            if base_default is not dataclasses.MISSING and value == base_default:
                continue
            try:
                scalar_fields[name] = _to_json_serializable(value)
            except (TypeError, ValueError):
                pass

    if getattr(req, "generator", None) is not None:
        seed = getattr(req, "seed", None)
        if seed is not None:
            scalar_fields["seed"] = _to_json_serializable(seed)

    if _debug_transfer:
        import torch as _torch

        for _n, _t in tensor_fields.items():
            if isinstance(_t, _torch.Tensor):
                _sz = _t.nelement() * _t.element_size()
                logger.debug(
                    "transfer_field %s shape=%s dtype=%s size=%d",
                    _n,
                    list(_t.shape),
                    _t.dtype,
                    _sz,
                )
            elif isinstance(_t, list):
                for _i, _ti in enumerate(_t):
                    if isinstance(_ti, _torch.Tensor):
                        _sz = _ti.nelement() * _ti.element_size()
                        logger.debug(
                            "transfer_field %s[%d] shape=%s dtype=%s size=%d",
                            _n,
                            _i,
                            list(_ti.shape),
                            _ti.dtype,
                            _sz,
                        )

    # Propagate OTel trace context over the JSON hop. TraceReqContext.__getstate__
    # reduces the live context to a JSON-safe dict (W3C traceparent/tracestate in
    # root_span_context). Receiver rebuilds via __setstate__ in _build_disagg_req.
    trace_ctx = getattr(req, "trace_ctx", None)
    if trace_ctx is not None and getattr(trace_ctx, "tracing_enable", False):
        try:
            trace_state = trace_ctx.__getstate__()
            if trace_state and trace_state.get("tracing_enable"):
                scalar_fields["_trace_state"] = trace_state
        except Exception as e:
            logger.debug("Failed to export trace state: %s", e)

    return tensor_fields, scalar_fields
```
**EN:** This block defines function `extract_transfer_fields`. Extract all transferable fields from a Req, split into tensors and scalars. Key calls include `logger.isEnabledFor`, `dataclasses.fields`, `getattr`, `_is_default`, and `_is_tensor_like`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `req` drive the behavior in this section.
**CN:** 该代码块定义了函数 `extract_transfer_fields`。 它用于处理 extract transfer fields 相关逻辑。 关键调用包括 `logger.isEnabledFor`、`dataclasses.fields`、`getattr`、`_is_default` 和 `_is_tensor_like`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `req` 等参数驱动。

### Lines 320-320: supporting statements / 辅助语句
```python
_LIST_MARKER_KEY = "__is_list__"
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_LIST_MARKER_KEY`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_LIST_MARKER_KEY` 等名称。

### Lines 323-344: `_pack_tensor_fields_for_broadcast` implementation / `_pack_tensor_fields_for_broadcast` 实现
```python
def _pack_tensor_fields_for_broadcast(tensor_fields: dict) -> dict:
    """Pack ``tensor_fields`` into a structure ``broadcast_tensor_dict`` accepts.

    ``broadcast_tensor_dict`` understands dict-of-tensor values (recursively),
    but not list-of-tensor values. Several Req fields (``prompt_embeds``,
    ``image_embeds``, ...) are lists of tensors, so we encode each list as a
    nested dict whose tensors are keyed by their stringified index, with a
    sentinel ``__is_list__`` flag to disambiguate from real nested dicts.
    """
    packed: dict = {}
    for key, value in tensor_fields.items():
        if isinstance(value, torch.Tensor):
            packed[key] = value
        elif isinstance(value, list):
            sub: dict = {_LIST_MARKER_KEY: True}
            for i, item in enumerate(value):
                if isinstance(item, torch.Tensor):
                    sub[str(i)] = item
            packed[key] = sub
        # Anything else (e.g. None, scalars) is intentionally dropped — the
        # scalar_fields broadcast covers non-tensor metadata.
    return packed
```
**EN:** This block defines function `_pack_tensor_fields_for_broadcast`. Pack ``tensor_fields`` into a structure ``broadcast_tensor_dict`` accepts. ``broadcast_tensor_dict`` understands dict-of-tensor values (recursively), but not list-of-tensor values. Key calls include `tensor_fields.items`, `isinstance`, `enumerate`, and `str`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `tensor_fields` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_pack_tensor_fields_for_broadcast`。 它用于打包tensor fields for broadcast。 关键调用包括 `tensor_fields.items`、`isinstance`、`enumerate` 和 `str`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `tensor_fields` 等参数驱动。

### Lines 347-357: `_unpack_tensor_fields_from_broadcast` implementation / `_unpack_tensor_fields_from_broadcast` 实现
```python
def _unpack_tensor_fields_from_broadcast(packed: dict) -> dict:
    """Inverse of :func:`_pack_tensor_fields_for_broadcast`."""
    out: dict = {}
    for key, value in packed.items():
        if isinstance(value, dict) and value.get(_LIST_MARKER_KEY) is True:
            indexed = [(int(k), v) for k, v in value.items() if k != _LIST_MARKER_KEY]
            indexed.sort(key=lambda kv: kv[0])
            out[key] = [v for _, v in indexed]
        else:
            out[key] = value
    return out
```
**EN:** This block defines function `_unpack_tensor_fields_from_broadcast`. Inverse of :func:`_pack_tensor_fields_for_broadcast`. Key calls include `packed.items`, `isinstance`, `indexed.sort`, `value.get`, and `int`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `packed` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_unpack_tensor_fields_from_broadcast`。 它用于解包tensor fields from broadcast。 关键调用包括 `packed.items`、`isinstance`、`indexed.sort`、`value.get` 和 `int`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `packed` 等参数驱动。

### Lines 360-366: `SchedulerDisaggMixin` class overview / `SchedulerDisaggMixin` 类概览
```python
class SchedulerDisaggMixin:
    """Disaggregated diffusion scheduling: transfer, compute, event loops."""

    # ------------------------------------------------------------------
    # Initialization
    # ------------------------------------------------------------------
```
**EN:** This block defines class `SchedulerDisaggMixin`. Disaggregated diffusion scheduling: transfer, compute, event loops.
**CN:** 该代码块定义了类 `SchedulerDisaggMixin`。 它用于封装 scheduler disagg mixin 相关行为。

### Lines 367-390: `_init_disagg_state` implementation / `_init_disagg_state` 实现
```python
    def _init_disagg_state(self: Scheduler, server_args, local_rank: int) -> None:
        """Initialize all disaggregation state, sockets, and transfer infrastructure."""
        from sglang.multimodal_gen.runtime.disaggregation.metrics import DisaggMetrics

        self._disagg_role = server_args.disagg_role
        self._disagg_timeout_s = float(getattr(server_args, "disagg_timeout", 600))
        self._disagg_metrics = None
        self._disagg_mode = getattr(server_args, "disagg_mode", False)
        self._pool_work_pull = None
        self._pool_result_push = None
        self._transfer_manager = None
        self._transfer_stream = None
        self._rdma_push_queue = None
        self._rdma_push_thread = None
        self._rdma_push_zmq = None
        self._compute_ready_queue = None
        self._recv_prefetch_thread = None

        if self._disagg_role != RoleType.MONOLITHIC:
            self._disagg_metrics = DisaggMetrics(role=self._disagg_role.value)
            device = torch.device(f"cuda:{local_rank}")
            self._transfer_stream = torch.cuda.Stream(device=device)
            self._init_disagg_sockets()
            self._init_disagg_transfer_manager()
```
**EN:** This block defines method `_init_disagg_state` on `SchedulerDisaggMixin`. Initialize all disaggregation state, sockets, and transfer infrastructure. Key calls include `float`, `getattr`, `DisaggMetrics`, `torch.device`, and `torch.cuda.Stream`. The implementation branches on conditions. Parameters such as `server_args`, and `local_rank` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_init_disagg_state`。 它用于初始化disagg state。 关键调用包括 `float`、`getattr`、`DisaggMetrics`、`torch.device` 和 `torch.cuda.Stream`。 实现中包含条件分支。 本段逻辑主要由 `server_args` 和 `local_rank` 等参数驱动。

### Lines 392-426: `_init_disagg_sockets` implementation / `_init_disagg_sockets` 实现
```python
    def _init_disagg_sockets(self: Scheduler):
        """Initialize ZMQ sockets for disaggregated mode (DiffusionServer-mediated).

        Only rank 0 creates ZMQ sockets. Non-rank-0 processes participate
        via NCCL broadcast from rank 0 (see _disagg_recv_work).
        """
        if self.gpu_id != 0:
            logger.info(
                "Pool mode %s rank %d: no ZMQ sockets (non-rank-0)",
                self._disagg_role.value.upper(),
                self.gpu_id,
            )
            return

        sa = self.server_args

        # PULL: receive work from DiffusionServer
        self._pool_work_pull, _ = get_zmq_socket(
            self.context,
            zmq.PULL,
            sa.pool_work_endpoint,
            bind=True,
            max_bind_retries=5,
            same_port=True,
        )
        # PUSH: send results to DiffusionServer
        self._pool_result_push, _ = get_zmq_socket(
            self.context, zmq.PUSH, sa.pool_result_endpoint, bind=False
        )
        logger.info(
            "Disagg %s rank 0: work_pull=%s, result_push=%s",
            self._disagg_role.value.upper(),
            sa.pool_work_endpoint,
            sa.pool_result_endpoint,
        )
```
**EN:** This block defines method `_init_disagg_sockets` on `SchedulerDisaggMixin`. Initialize ZMQ sockets for disaggregated mode (DiffusionServer-mediated). Only rank 0 creates ZMQ sockets. Key calls include `get_zmq_socket`, `logger.info`, and `self._disagg_role.value.upper`. The implementation branches on conditions.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_init_disagg_sockets`。 它用于初始化disagg sockets。 关键调用包括 `get_zmq_socket`、`logger.info` 和 `self._disagg_role.value.upper`。 实现中包含条件分支。

### Lines 428-546: `_init_disagg_transfer_manager` implementation / `_init_disagg_transfer_manager` 实现
```python
    def _init_disagg_transfer_manager(self: Scheduler):
        """Initialize TransferManager for transfer mode (rank 0 only).

        Creates a TransferTensorBuffer (pinned memory pool) and a
        BaseTransferEngine, then wraps them in a DiffusionTransferManager.
        Also sends a transfer_register message to DiffusionServer.
        """
        if self.gpu_id != 0:
            return

        sa = self.server_args

        # Pool size: configurable, default 256 MiB
        pool_size = getattr(sa, "disagg_transfer_pool_size", 256 * 1024 * 1024)

        # Create transfer engine.
        # NOTE: self.gpu_id is the role-internal rank (0..num_role_gpus-1),
        # not the physical GPU index. In disagg mode with --base-gpu-id > 0,
        # the physical device is self.worker.local_rank. Mooncake needs the
        # physical index to pin the right NIC and register GPUDirect buffers.
        hostname = getattr(sa, "disagg_p2p_hostname", "127.0.0.1")
        ib_device = getattr(sa, "disagg_ib_device", None)
        physical_gpu_id = self.worker.local_rank
        engine = create_transfer_engine(
            hostname=hostname,
            gpu_id=physical_gpu_id,
            ib_device=ib_device,
        )

        # Use GPU buffer when engine supports GPUDirect RDMA, CPU pinned otherwise
        device = f"cuda:{physical_gpu_id}" if engine.supports_gpu_direct else "cpu"
        buffer = TransferTensorBuffer(
            pool_size=pool_size, device=device, role_name=self._disagg_role.value
        )

        # Create transfer manager
        self._transfer_manager = DiffusionTransferManager(engine=engine, buffer=buffer)

        # Pre-allocate receive slots for receivers (denoiser/decoder)
        self._preallocated_slots: dict[int, object] = {}
        preallocated_slot_info = []
        if self._disagg_role in (RoleType.DENOISER, RoleType.DECODER):
            capacity = getattr(sa, "disagg_prealloc_slots", 2)
            typical_size = 64 * 1024 * 1024  # 64 MiB per slot
            for i in range(capacity):
                slot = buffer.allocate(typical_size, f"prealloc_{i}")
                if slot is not None:
                    self._preallocated_slots[i] = slot
                    preallocated_slot_info.append(
                        {
                            "offset": slot.offset,
                            "size": slot.size,
                            "slot_id": i,
                            "addr": self._transfer_manager.pool_data_ptr + slot.offset,
                        }
                    )
            if preallocated_slot_info:
                logger.info(
                    "Transfer %s: pre-allocated %d receive slots",
                    self._disagg_role.value.upper(),
                    len(preallocated_slot_info),
                )

        # Register with DiffusionServer.
        # Include our own work_endpoint so DS can key the peer by URL index,
        # not by registration order (startup order is not guaranteed to match
        # --encoder/denoiser/decoder-urls ordering).
        register_msg = TransferRegisterMsg(
            role=self._disagg_role.value,
            session_id=self._transfer_manager.session_id,
            pool_ptr=self._transfer_manager.pool_data_ptr,
            pool_size=self._transfer_manager.pool_size,
            work_endpoint=sa.pool_work_endpoint,
            preallocated_slots=preallocated_slot_info,
        )
        self._pool_result_push.send_multipart(encode_transfer_msg(register_msg))
        logger.info(
            "Transfer %s: registered with DS (session=%s, pool=%d bytes, prealloc=%d)",
            self._disagg_role.value.upper(),
            self._transfer_manager.session_id,
            pool_size,
            len(preallocated_slot_info),
        )

        # RDMA push thread for sender roles (encoder/denoiser)
        if self._disagg_role in (RoleType.ENCODER, RoleType.DENOISER):
            self._rdma_push_queue = queue.Queue(maxsize=4)
            self._rdma_push_zmq, _ = get_zmq_socket(
                self.context,
                zmq.PUSH,
                sa.pool_result_endpoint,
                bind=False,
            )
            self._rdma_push_thread = threading.Thread(
                target=self._rdma_push_loop,
                daemon=True,
                name=f"rdma-push-{self._disagg_role.value}",
            )
            self._rdma_push_thread.start()
            logger.info(
                "Transfer %s: RDMA push thread started",
                self._disagg_role.value.upper(),
            )

        # Recv prefetch thread for receiver roles (denoiser/decoder)
        # Rank 0 only (bg thread does ZMQ recv + load; multi-rank gets
        # scalar fields via broadcast_pyobj from the main thread).
        if self._disagg_role in (RoleType.DENOISER, RoleType.DECODER):
            self._compute_ready_queue = queue.Queue(maxsize=4)
            self._recv_prefetch_thread = threading.Thread(
                target=self._recv_prefetch_loop,
                daemon=True,
                name=f"recv-prefetch-{self._disagg_role.value}",
            )
            self._recv_prefetch_thread.start()
            logger.info(
                "Transfer %s: recv prefetch thread started",
                self._disagg_role.value.upper(),
            )
```
**EN:** This block defines method `_init_disagg_transfer_manager` on `SchedulerDisaggMixin`. Initialize TransferManager for transfer mode (rank 0 only). Creates a TransferTensorBuffer (pinned memory pool) and a BaseTransferEngine, then wraps them in a DiffusionTransferManager. Key calls include `getattr`, `create_transfer_engine`, `TransferTensorBuffer`, `DiffusionTransferManager`, and `TransferRegisterMsg`. The implementation branches on conditions, iterates over collections or steps.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_init_disagg_transfer_manager`。 它用于初始化disagg transfer manager。 关键调用包括 `getattr`、`create_transfer_engine`、`TransferTensorBuffer`、`DiffusionTransferManager` 和 `TransferRegisterMsg`。 实现中包含条件分支，会遍历集合或步骤。

### Lines 552-584: `_rdma_push_loop` implementation / `_rdma_push_loop` 实现
```python
    def _rdma_push_loop(self: Scheduler):
        """Background thread: execute RDMA push + notify DS.

        Runs push_to_peer (blocking RDMA) on a dedicated thread so the
        main event loop can immediately start processing the next request.
        """
        role_name = self._disagg_role.value.upper()
        while True:
            item = self._rdma_push_queue.get()
            if item is None:
                break  # Shutdown signal
            request_id, dest_session_id, dest_addr, transfer_size = item
            try:
                success = self._transfer_manager.push_to_peer(
                    request_id=request_id,
                    dest_session_id=dest_session_id,
                    dest_addr=dest_addr,
                    transfer_size=transfer_size,
                )
                if success:
                    self._transfer_manager.free_staged(request_id)

                pushed_msg = TransferPushedMsg(request_id=request_id)
                self._rdma_push_zmq.send_multipart(encode_transfer_msg(pushed_msg))

                if not success:
                    logger.error(
                        "Transfer %s: RDMA push failed for %s", role_name, request_id
                    )
            except Exception:
                logger.exception(
                    "Transfer %s: RDMA push thread error for %s", role_name, request_id
                )
```
**EN:** This block defines method `_rdma_push_loop` on `SchedulerDisaggMixin`. Background thread: execute RDMA push + notify DS. Runs push_to_peer (blocking RDMA) on a dedicated thread so the main event loop can immediately start processing the next request. Key calls include `self._disagg_role.value.upper`, `self._rdma_push_queue.get`, `self._transfer_manager.push_to_peer`, `TransferPushedMsg`, and `self._rdma_push_zmq.send_multipart`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_rdma_push_loop`。 它用于处理 rdma push loop 相关逻辑。 关键调用包括 `self._disagg_role.value.upper`、`self._rdma_push_queue.get`、`self._transfer_manager.push_to_peer`、`TransferPushedMsg` 和 `self._rdma_push_zmq.send_multipart`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。

### Lines 586-626: `_recv_prefetch_loop` implementation / `_recv_prefetch_loop` 实现
```python
    def _recv_prefetch_loop(self: Scheduler):
        """Background thread: recv transfer messages and prefetch tensor loads.

        For transfer_ready: loads tensors + builds Req in this thread, then
        enqueues the ready-to-compute item. This allows loading of request N+1
        to overlap with compute of request N on the main thread.

        For transfer_alloc/push: passes them through to the main thread for handling.
        """
        role_name = self._disagg_role.value.upper()
        while self._running:
            try:
                raw_frames = self._pool_work_pull.recv_multipart()
                frames = [bytes(f) for f in raw_frames]

                msg = decode_transfer_msg(frames)
                msg_type = msg.get("msg_type", "")

                if msg_type == TransferMsgType.READY:
                    # Prefetch: load tensors + build Req in this thread
                    item = self._prefetch_transfer_ready(msg)
                    self._compute_ready_queue.put(("transfer_compute", item))
                elif msg_type == TransferMsgType.PUSH:
                    # Handle push directly in prefetch thread — it only
                    # enqueues to the RDMA bg thread (thread-safe queue).
                    # Critical for pipeline parallelism: if deferred to the
                    # main thread, this gets blocked behind the next request's
                    # GPU compute, preventing the previous request's output
                    # from reaching the decoder.
                    self._handle_transfer_push(msg)
                else:
                    # alloc and other messages: pass to main thread
                    # (alloc sends on _pool_result_push which isn't thread-safe)
                    self._compute_ready_queue.put(("transfer_control", frames))

            except zmq.ZMQError as e:
                if not self._running:
                    break
                logger.error("Transfer %s recv prefetch: ZMQ error: %s", role_name, e)
            except Exception:
                logger.exception("Transfer %s recv prefetch: error", role_name)
```
**EN:** This block defines method `_recv_prefetch_loop` on `SchedulerDisaggMixin`. Background thread: recv transfer messages and prefetch tensor loads. For transfer_ready: loads tensors + builds Req in this thread, then enqueues the ready-to-compute item. Key calls include `self._disagg_role.value.upper`, `self._pool_work_pull.recv_multipart`, `decode_transfer_msg`, `msg.get`, and `bytes`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_recv_prefetch_loop`。 它用于处理 recv prefetch loop 相关逻辑。 关键调用包括 `self._disagg_role.value.upper`、`self._pool_work_pull.recv_multipart`、`decode_transfer_msg`、`msg.get` 和 `bytes`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。

### Lines 628-675: `_prefetch_transfer_ready` implementation / `_prefetch_transfer_ready` 实现
```python
    def _prefetch_transfer_ready(self: Scheduler, msg: dict) -> tuple:
        """Load tensors from transfer buffer and build Req for a transfer_ready message.

        Called from the recv prefetch thread. Loads on _transfer_stream
        and builds the Req, so the main thread can start compute immediately.

        Returns (req, load_event, request_id, role_name, prealloc_slot_id).
        """
        request_id = msg["request_id"]
        manifest = msg.get("manifest", {})
        scalar_fields = msg.get("scalar_fields", {})
        role_name = self._disagg_role.value.upper()

        if self._disagg_metrics:
            self._disagg_metrics.record_request_start(request_id)

        # Pre-allocated slot handling
        prealloc_slot_id = scalar_fields.pop("_prealloc_slot_id", None)
        if (
            prealloc_slot_id is not None
            and prealloc_slot_id in self._preallocated_slots
        ):
            slot = self._preallocated_slots[prealloc_slot_id]
            self._transfer_manager.register_prealloc_as_receive(request_id, slot)

        # Load tensors on transfer_stream (non-blocking)
        local_device = f"cuda:{self.worker.local_rank}"
        tensors, load_event = self._transfer_manager.load_tensors_async(
            request_id,
            manifest,
            device=local_device,
            stream=self._transfer_stream,
        )

        # NOTE: Do NOT free the receive slot here. The async load is still
        # in progress. The slot must remain valid until the main thread waits
        # on load_event. Freeing is done in _disagg_prefetch_event_loop.

        # Build Req (CPU work, overlapped with load)
        req = self._build_disagg_req(scalar_fields, tensors)

        # NOTE: Do NOT call scheduler_mod.set_timesteps() here!
        # This runs on the prefetch thread. set_timesteps mutates shared
        # scheduler state (self.sigmas), which would corrupt the currently
        # running denoising loop on the main thread. Deferred to main thread
        # in _disagg_prefetch_event_loop, right before compute.

        return (req, load_event, request_id, role_name, prealloc_slot_id, scalar_fields)
```
**EN:** This block defines method `_prefetch_transfer_ready` on `SchedulerDisaggMixin`. Load tensors from transfer buffer and build Req for a transfer_ready message. Called from the recv prefetch thread. Key calls include `msg.get`, `self._disagg_role.value.upper`, `scalar_fields.pop`, `self._transfer_manager.load_tensors_async`, and `self._build_disagg_req`. The implementation branches on conditions. Parameters such as `msg` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_prefetch_transfer_ready`。 它用于处理 prefetch transfer ready 相关逻辑。 关键调用包括 `msg.get`、`self._disagg_role.value.upper`、`scalar_fields.pop`、`self._transfer_manager.load_tensors_async` 和 `self._build_disagg_req`。 实现中包含条件分支。 本段逻辑主要由 `msg` 等参数驱动。

### Lines 681-713: `_broadcast_to_all_ranks` implementation / `_broadcast_to_all_ranks` 实现
```python
    def _broadcast_to_all_ranks(self: Scheduler, data):
        """Broadcast *data* from rank 0 to all other ranks.

        Rank 0 passes the real payload; non-rank-0 passes ``None``.
        Broadcasts through all applicable groups (SP, CFG, TP).
        """
        sa = self.server_args

        if sa.sp_degree != 1:
            data = broadcast_pyobj(
                data,
                self.worker.sp_group.rank,
                self.worker.sp_cpu_group,
                src=self.worker.sp_group.ranks[0],
            )

        if sa.enable_cfg_parallel:
            data = broadcast_pyobj(
                data,
                self.worker.cfg_group.rank,
                self.worker.cfg_cpu_group,
                src=self.worker.cfg_group.ranks[0],
            )

        if sa.tp_size > 1:
            data = broadcast_pyobj(
                data,
                self.worker.tp_group.rank,
                self.worker.tp_cpu_group,
                src=self.worker.tp_group.ranks[0],
            )

        return data
```
**EN:** This block defines method `_broadcast_to_all_ranks` on `SchedulerDisaggMixin`. Broadcast *data* from rank 0 to all other ranks. Rank 0 passes the real payload; non-rank-0 passes ``None``. Key calls include `broadcast_pyobj`. The implementation branches on conditions. Parameters such as `data` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_broadcast_to_all_ranks`。 它用于处理 broadcast to all ranks 相关逻辑。 关键调用包括 `broadcast_pyobj`。 实现中包含条件分支。 本段逻辑主要由 `data` 等参数驱动。

### Lines 715-717: `_is_multi_rank` implementation / `_is_multi_rank` 实现
```python
    def _is_multi_rank(self: Scheduler) -> bool:
        sa = self.server_args
        return sa.sp_degree != 1 or sa.tp_size > 1 or sa.enable_cfg_parallel
```
**EN:** This block defines method `_is_multi_rank` on `SchedulerDisaggMixin`. It handles is multi rank logic.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_is_multi_rank`。 它用于处理 is multi rank 相关逻辑。

### Lines 719-738: `_broadcast_tensor_dict_to_all_ranks` implementation / `_broadcast_tensor_dict_to_all_ranks` 实现
```python
    def _broadcast_tensor_dict_to_all_ranks(
        self: Scheduler, tensor_dict: dict | None
    ) -> dict | None:
        """Broadcast a tensor dict from rank 0 to non-rank-0 via NCCL.

        Uses ``GroupCoordinator.broadcast_tensor_dict`` which ships tensor
        metadata over the CPU group and the tensor payload over the device
        (NCCL) group, so large GPU buffers never bounce through CPU.
        """
        sa = self.server_args

        if sa.sp_degree != 1:
            tensor_dict = self.worker.sp_group.broadcast_tensor_dict(tensor_dict, src=0)
        if sa.enable_cfg_parallel:
            tensor_dict = self.worker.cfg_group.broadcast_tensor_dict(
                tensor_dict, src=0
            )
        if sa.tp_size > 1:
            tensor_dict = self.worker.tp_group.broadcast_tensor_dict(tensor_dict, src=0)
        return tensor_dict
```
**EN:** This block defines method `_broadcast_tensor_dict_to_all_ranks` on `SchedulerDisaggMixin`. Broadcast a tensor dict from rank 0 to non-rank-0 via NCCL. Uses ``GroupCoordinator.broadcast_tensor_dict`` which ships tensor metadata over the CPU group and the tensor payload over the device (NCCL) group, so large GPU buffers never bounce through CPU. Key calls include `self.worker.sp_group.broadcast_tensor_dict`, `self.worker.cfg_group.broadcast_tensor_dict`, and `self.worker.tp_group.broadcast_tensor_dict`. The implementation branches on conditions. Parameters such as `tensor_dict` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_broadcast_tensor_dict_to_all_ranks`。 它用于处理 broadcast tensor dict to all ranks 相关逻辑。 关键调用包括 `self.worker.sp_group.broadcast_tensor_dict`、`self.worker.cfg_group.broadcast_tensor_dict` 和 `self.worker.tp_group.broadcast_tensor_dict`。 实现中包含条件分支。 本段逻辑主要由 `tensor_dict` 等参数驱动。

### Lines 740-794: `_broadcast_req_to_all_ranks` implementation / `_broadcast_req_to_all_ranks` 实现
```python
    def _broadcast_req_to_all_ranks(self: Scheduler, req: Req | None) -> Req | None:
        """Broadcast a fully-loaded Req (scalars + GPU tensors) from rank 0.

        Required for multi-rank denoiser/decoder in disagg mode: only rank 0
        owns the TransferManager and RDMA-loads tensors into GPU memory. All
        other ranks must see the same Req before entering ``execute_forward``,
        otherwise REPLICATED stages (e.g. denoising) blow up on empty tensor
        fields because ``ParallelExecutor`` never broadcasts the batch for
        that paradigm.

        Tensor fields travel over NCCL (stays on GPU); scalar fields travel
        as a small pickled object over the CPU group.
        """
        if not self._is_multi_rank():
            return req

        is_rank0 = self.gpu_id == 0

        if is_rank0:
            assert req is not None, "rank 0 must pass a loaded Req"
            tensor_fields, scalar_fields = extract_transfer_fields(req)
            packed_tensors = _pack_tensor_fields_for_broadcast(tensor_fields)
        else:
            scalar_fields = None
            packed_tensors = None

        # 1. Scalars via CPU pyobj broadcast.
        scalar_fields = self._broadcast_to_all_ranks(scalar_fields)

        # 2. Tensors via NCCL broadcast — keeps GPU buffers on device.
        packed_tensors = self._broadcast_tensor_dict_to_all_ranks(packed_tensors)

        if is_rank0:
            return req

        tensor_fields = _unpack_tensor_fields_from_broadcast(packed_tensors or {})
        # Move tensors onto this rank's physical device. The broadcast
        # allocates receive tensors on the receiver's default CUDA device
        # (set via torch.cuda.set_device(local_rank) during init), which is
        # already the right physical GPU — the .to() is effectively a no-op
        # but makes the invariant explicit for future readers.
        local_device = torch.device(f"cuda:{self.worker.local_rank}")
        for key, value in list(tensor_fields.items()):
            if isinstance(value, torch.Tensor):
                tensor_fields[key] = value.to(local_device, non_blocking=True)
            elif isinstance(value, list):
                tensor_fields[key] = [
                    (
                        t.to(local_device, non_blocking=True)
                        if isinstance(t, torch.Tensor)
                        else t
                    )
                    for t in value
                ]
        return self._build_disagg_req(scalar_fields or {}, tensor_fields)
```
**EN:** This block defines method `_broadcast_req_to_all_ranks` on `SchedulerDisaggMixin`. Broadcast a fully-loaded Req (scalars + GPU tensors) from rank 0. Required for multi-rank denoiser/decoder in disagg mode: only rank 0 owns the TransferManager and RDMA-loads tensors into GPU memory. Key calls include `self._broadcast_to_all_ranks`, `self._broadcast_tensor_dict_to_all_ranks`, `_unpack_tensor_fields_from_broadcast`, `torch.device`, and `list`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `req` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_broadcast_req_to_all_ranks`。 它用于处理 broadcast req to all ranks 相关逻辑。 关键调用包括 `self._broadcast_to_all_ranks`、`self._broadcast_tensor_dict_to_all_ranks`、`_unpack_tensor_fields_from_broadcast`、`torch.device` 和 `list`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `req` 等参数驱动。

### Lines 800-814: `_disagg_recv_work` implementation / `_disagg_recv_work` 实现
```python
    def _disagg_recv_work(self: Scheduler) -> list[bytes] | None:
        """Receive work frames in pool mode, with multi-rank broadcast.

        Rank 0: recv from ZMQ PULL socket, broadcast to other ranks.
        Non-rank-0: receive via NCCL broadcast from rank 0.

        Returns list of bytes frames, or None on shutdown.
        """
        if self.gpu_id == 0:
            raw_frames = self._pool_work_pull.recv_multipart()
            frames = [bytes(f) for f in raw_frames]
        else:
            frames = None

        return self._broadcast_to_all_ranks(frames)
```
**EN:** This block defines method `_disagg_recv_work` on `SchedulerDisaggMixin`. Receive work frames in pool mode, with multi-rank broadcast. Rank 0: recv from ZMQ PULL socket, broadcast to other ranks. Key calls include `self._broadcast_to_all_ranks`, `self._pool_work_pull.recv_multipart`, and `bytes`. The implementation branches on conditions.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_disagg_recv_work`。 它用于处理 disagg recv work 相关逻辑。 关键调用包括 `self._broadcast_to_all_ranks`、`self._pool_work_pull.recv_multipart` 和 `bytes`。 实现中包含条件分支。

### Lines 816-906: `_disagg_prefetch_event_loop` implementation / `_disagg_prefetch_event_loop` 实现
```python
    def _disagg_prefetch_event_loop(self: Scheduler, role_name: str) -> None:
        """Event loop for transfer receiver roles with recv prefetch thread (rank 0).

        The recv thread reads from ZMQ and prefetches tensor loads.
        This loop reads from _compute_ready_queue:
          - "transfer_compute": load already done, wait_event + free slot
              → broadcast scalar_fields to non-rank-0 → compute
          - "transfer_control": alloc/push messages, handle on main thread
              → broadcast "skip" so non-rank-0 doesn't hang
          - queue timeout: broadcast "skip"
          - shutdown: broadcast None
        """
        is_multi_rank = (
            self.server_args.sp_degree != 1
            or self.server_args.tp_size > 1
            or self.server_args.enable_cfg_parallel
        )

        while self._running:
            try:
                try:
                    msg_type, data = self._compute_ready_queue.get(timeout=1.0)
                except queue.Empty:
                    if is_multi_rank:
                        self._broadcast_to_all_ranks(("skip",))
                    continue

                if msg_type == "transfer_compute":
                    # Load already done by recv thread
                    req, load_event, request_id, rn, prealloc_slot_id, scalar_fields = (
                        data
                    )
                    # Wait for load to complete on compute stream
                    if load_event is not None:
                        torch.cuda.current_stream().wait_event(load_event)
                    # Now safe to free the receive slot
                    if prealloc_slot_id is not None:
                        with self._transfer_manager._lock:
                            self._transfer_manager._pending_receives.pop(
                                request_id, None
                            )
                    else:
                        self._transfer_manager.free_receive_slot(request_id)
                    # Broadcast the full Req (scalar + tensor fields) to
                    # non-rank-0 ranks. Tensors ride NCCL on the SP/CFG/TP
                    # groups so downstream REPLICATED stages (e.g. denoising)
                    # see identical inputs on every rank — without this, the
                    # non-rank-0 ranks would enter execute_forward with empty
                    # prompt_embeds and fail verify_input.
                    if is_multi_rank:
                        self._broadcast_to_all_ranks(("compute",))
                        self._broadcast_req_to_all_ranks(req)
                    # Init scheduler timesteps on main thread (safe — no
                    # concurrent denoising loop can be running here).
                    if self._disagg_role == RoleType.DENOISER:
                        _init_disagg_request_scheduler(self, req)
                    # Run compute
                    if self._disagg_role == RoleType.DENOISER:
                        self._disagg_denoiser_compute(req, request_id, rn)
                    elif self._disagg_role == RoleType.DECODER:
                        self._disagg_decoder_compute(req, request_id, rn)

                elif msg_type == "transfer_control":
                    # alloc, push messages — handle on main thread (rank 0 only)
                    if is_multi_rank:
                        self._broadcast_to_all_ranks(("skip",))
                    self._handle_transfer_msg(data)

                self._consecutive_error_count = 0

            except Exception as e:
                self._consecutive_error_count += 1
                logger.error(
                    "Pool %s rank %d prefetch loop: error (attempt %d/%d): %s",
                    role_name,
                    self.gpu_id,
                    self._consecutive_error_count,
                    self._max_consecutive_errors,
                    e,
                    exc_info=True,
                )
                if self._consecutive_error_count >= self._max_consecutive_errors:
                    raise RuntimeError(
                        f"Pool {role_name} rank {self.gpu_id} terminated after "
                        f"{self._max_consecutive_errors} consecutive errors: {e}"
                    ) from e

        # Shutdown: notify non-rank-0 to exit
        if is_multi_rank:
            self._broadcast_to_all_ranks(None)
        self._cleanup_disagg()
```
**EN:** This block defines method `_disagg_prefetch_event_loop` on `SchedulerDisaggMixin`. Event loop for transfer receiver roles with recv prefetch thread (rank 0). The recv thread reads from ZMQ and prefetches tensor loads. Key calls include `self._cleanup_disagg`, `self._broadcast_to_all_ranks`, `self._compute_ready_queue.get`, `logger.error`, and `torch.cuda.current_stream.wait_event`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths, uses context-managed resources. Parameters such as `role_name` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_disagg_prefetch_event_loop`。 它用于处理 disagg prefetch event loop 相关逻辑。 关键调用包括 `self._cleanup_disagg`、`self._broadcast_to_all_ranks`、`self._compute_ready_queue.get`、`logger.error` 和 `torch.cuda.current_stream.wait_event`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `role_name` 等参数驱动。

### Lines 908-957: `_disagg_non_rank0_event_loop` implementation / `_disagg_non_rank0_event_loop` 实现
```python
    def _disagg_non_rank0_event_loop(self: Scheduler) -> None:
        """Event loop for non-rank-0 receivers in multi-rank prefetch mode.

        Blocks on broadcast from rank 0:
          - ("compute", scalar_fields): build minimal Req → execute_forward
          - ("skip",): continue (rank 0 handled a control msg or timed out)
          - None: shutdown, exit loop
        """
        role_name = self._disagg_role.value.upper()
        logger.info(
            "Pool %s rank %d: entering non-rank-0 prefetch loop",
            role_name,
            self.gpu_id,
        )

        while True:
            try:
                msg = self._broadcast_to_all_ranks(None)

                if msg is None:
                    # Shutdown signal
                    break

                if isinstance(msg, tuple) and len(msg) >= 1 and msg[0] == "compute":
                    # Participate in the companion tensor broadcast so this
                    # rank sees the full Req (scalars + GPU tensors). Without
                    # the tensor half, REPLICATED stages would see empty
                    # prompt_embeds on non-rank-0 and fail verify_input.
                    req = self._broadcast_req_to_all_ranks(None)
                    self._disagg_compute_non_rank0(req)
                # else: ("skip",) — continue

            except Exception as e:
                self._consecutive_error_count += 1
                logger.error(
                    "Pool %s rank %d non-rank-0 loop: error (attempt %d/%d): %s",
                    role_name,
                    self.gpu_id,
                    self._consecutive_error_count,
                    self._max_consecutive_errors,
                    e,
                    exc_info=True,
                )
                if self._consecutive_error_count >= self._max_consecutive_errors:
                    raise RuntimeError(
                        f"Pool {role_name} rank {self.gpu_id} terminated after "
                        f"{self._max_consecutive_errors} consecutive errors: {e}"
                    ) from e

        self._cleanup_disagg()
```
**EN:** This block defines method `_disagg_non_rank0_event_loop` on `SchedulerDisaggMixin`. Event loop for non-rank-0 receivers in multi-rank prefetch mode. Blocks on broadcast from rank 0: - ("compute", scalar_fields): build minimal Req → execute_forward - ("skip",): continue (rank 0 handled a control msg or timed out) - None: shutdown, exit loop Key calls include `self._disagg_role.value.upper`, `logger.info`, `self._cleanup_disagg`, `self._broadcast_to_all_ranks`, and `isinstance`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_disagg_non_rank0_event_loop`。 它用于处理 disagg non rank0 event loop 相关逻辑。 关键调用包括 `self._disagg_role.value.upper`、`logger.info`、`self._cleanup_disagg`、`self._broadcast_to_all_ranks` 和 `isinstance`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。

### Lines 959-1046: `_disagg_event_loop` implementation / `_disagg_event_loop` 实现
```python
    def _disagg_event_loop(self: Scheduler) -> None:
        """Event loop for all roles in pool mode (DiffusionServer-mediated).

        Multi-rank support:
        - Rank 0 receives from ZMQ, broadcasts to other ranks via NCCL
        - All ranks process work (execute_forward with SP/TP sharding)
        - Only rank 0 sends results back to DiffusionServer

        Transfer:
        - Transfer control messages (transfer_alloc, transfer_push) are rank-0-only.
        - transfer_ready is broadcast to all ranks for compute.
        - Encoder receives pickled Req, runs compute, stages output for transfer.
        - Denoiser/decoder only receive transfer control messages.

        Receiver prefetch paths:
        - Rank 0: _disagg_prefetch_event_loop (reads from compute_ready_queue)
        - Non-rank-0 in multi-rank: _disagg_non_rank0_event_loop (broadcast)
        - Encoder (any rank): existing _disagg_recv_work while loop below
        """
        role_name = self._disagg_role.value.upper()
        is_rank0 = self.gpu_id == 0
        is_multi_rank = (
            self.server_args.sp_degree != 1
            or self.server_args.tp_size > 1
            or self.server_args.enable_cfg_parallel
        )
        use_prefetch = self._compute_ready_queue is not None
        logger.info(
            "Pool mode %s rank %d event loop started " "(multi_rank=%s, prefetch=%s)",
            role_name,
            self.gpu_id,
            is_multi_rank,
            use_prefetch,
        )

        # Rank 0 receiver with prefetch queue → prefetch event loop
        if use_prefetch:
            self._disagg_prefetch_event_loop(role_name)
            return

        # Non-rank-0 receiver in multi-rank → broadcast-based loop
        if (
            not is_rank0
            and is_multi_rank
            and self._disagg_role in (RoleType.DENOISER, RoleType.DECODER)
        ):
            self._disagg_non_rank0_event_loop()
            return

        while self._running:
            try:
                # All ranks receive work (rank 0 via ZMQ, others via broadcast)
                frames = self._disagg_recv_work()

                # Transfer dispatch: check on ALL ranks (frames are broadcast)
                if self._is_transfer_frames(frames):
                    if is_rank0:
                        # Rank 0: handle all transfer messages
                        self._handle_transfer_msg(frames)
                    else:
                        # Non-rank-0: only participate in transfer_ready compute
                        self._handle_transfer_non_rank0(frames)
                elif self._disagg_role == RoleType.ENCODER:
                    self._disagg_encoder_step(
                        send_tensors,
                        frames=frames,
                    )

                self._consecutive_error_count = 0

            except Exception as e:
                self._consecutive_error_count += 1
                logger.error(
                    "Pool %s rank %d: error (attempt %d/%d): %s",
                    role_name,
                    self.gpu_id,
                    self._consecutive_error_count,
                    self._max_consecutive_errors,
                    e,
                    exc_info=True,
                )
                if self._consecutive_error_count >= self._max_consecutive_errors:
                    raise RuntimeError(
                        f"Pool {role_name} rank {self.gpu_id} terminated after "
                        f"{self._max_consecutive_errors} consecutive errors: {e}"
                    ) from e

        self._cleanup_disagg()
```
**EN:** This block defines method `_disagg_event_loop` on `SchedulerDisaggMixin`. Event loop for all roles in pool mode (DiffusionServer-mediated). Multi-rank support: - Rank 0 receives from ZMQ, broadcasts to other ranks via NCCL - All ranks process work (execute_forward with SP/TP sharding) - Only rank 0 sends results back to DiffusionServer Transfer: - Transfer control messages (transfer_alloc, transfer_push) are rank-0-only. Key calls include `self._disagg_role.value.upper`, `logger.info`, `self._cleanup_disagg`, `self._disagg_prefetch_event_loop`, and `self._disagg_non_rank0_event_loop`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_disagg_event_loop`。 它用于处理 disagg event loop 相关逻辑。 关键调用包括 `self._disagg_role.value.upper`、`logger.info`、`self._cleanup_disagg`、`self._disagg_prefetch_event_loop` 和 `self._disagg_non_rank0_event_loop`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。

### Lines 1048-1065: `_cleanup_disagg` implementation / `_cleanup_disagg` 实现
```python
    def _cleanup_disagg(self: Scheduler):
        """Clean up all pool mode resources (sockets, threads, transfer manager)."""
        # Shutdown RDMA push thread
        if self._rdma_push_queue is not None:
            self._rdma_push_queue.put(None)
        if self._rdma_push_thread is not None:
            self._rdma_push_thread.join(timeout=5)
        if self._rdma_push_zmq is not None:
            self._rdma_push_zmq.close()
        # Recv prefetch thread stops when self._running = False
        if self._recv_prefetch_thread is not None:
            self._recv_prefetch_thread.join(timeout=5)
        if self._transfer_manager is not None:
            self._transfer_manager.cleanup()
        if self._pool_work_pull is not None:
            self._pool_work_pull.close()
        if self._pool_result_push is not None:
            self._pool_result_push.close()
```
**EN:** This block defines method `_cleanup_disagg` on `SchedulerDisaggMixin`. Clean up all pool mode resources (sockets, threads, transfer manager). Key calls include `self._rdma_push_queue.put`, `self._rdma_push_thread.join`, `self._rdma_push_zmq.close`, `self._recv_prefetch_thread.join`, and `self._transfer_manager.cleanup`. The implementation branches on conditions.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_cleanup_disagg`。 它用于处理 cleanup disagg 相关逻辑。 关键调用包括 `self._rdma_push_queue.put`、`self._rdma_push_thread.join`、`self._rdma_push_zmq.close`、`self._recv_prefetch_thread.join` 和 `self._transfer_manager.cleanup`。 实现中包含条件分支。

### Lines 1071-1074: `_is_transfer_frames` implementation / `_is_transfer_frames` 实现
```python
    @staticmethod
    def _is_transfer_frames(frames: list) -> bool:
        """Check if ZMQ multipart frames carry a transfer control message."""
        return is_transfer_message(frames)
```
**EN:** This block defines method `_is_transfer_frames` on `SchedulerDisaggMixin`. Check if ZMQ multipart frames carry a transfer control message. Key calls include `is_transfer_message`. Parameters such as `frames` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_is_transfer_frames`。 它用于处理 is transfer frames 相关逻辑。 关键调用包括 `is_transfer_message`。 本段逻辑主要由 `frames` 等参数驱动。

### Lines 1076-1100: `_handle_transfer_msg` implementation / `_handle_transfer_msg` 实现
```python
    def _handle_transfer_msg(self: Scheduler, frames: list) -> None:
        """Dispatch a transfer control message to the appropriate handler (rank 0)."""
        msg = decode_transfer_msg(frames)
        msg_type = msg.get("msg_type", "")
        request_id = msg.get("request_id", "")

        logger.debug(
            "Transfer %s: received %s for %s",
            self._disagg_role.value.upper(),
            msg_type,
            request_id,
        )

        if msg_type == TransferMsgType.ALLOC:
            self._handle_transfer_alloc(msg)
        elif msg_type == TransferMsgType.PUSH:
            self._handle_transfer_push(msg)
        elif msg_type == TransferMsgType.READY:
            self._handle_transfer_ready(msg)
        else:
            logger.warning(
                "Transfer %s: unknown message type %s",
                self._disagg_role.value.upper(),
                msg_type,
            )
```
**EN:** This block defines method `_handle_transfer_msg` on `SchedulerDisaggMixin`. Dispatch a transfer control message to the appropriate handler (rank 0). Key calls include `decode_transfer_msg`, `msg.get`, `logger.debug`, `self._disagg_role.value.upper`, and `self._handle_transfer_alloc`. The implementation branches on conditions. Parameters such as `frames` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_handle_transfer_msg`。 它用于处理 handle transfer msg 相关逻辑。 关键调用包括 `decode_transfer_msg`、`msg.get`、`logger.debug`、`self._disagg_role.value.upper` 和 `self._handle_transfer_alloc`。 实现中包含条件分支。 本段逻辑主要由 `frames` 等参数驱动。

### Lines 1102-1116: `_handle_transfer_non_rank0` implementation / `_handle_transfer_non_rank0` 实现
```python
    def _handle_transfer_non_rank0(self: Scheduler, frames: list) -> None:
        """Handle transfer messages on non-rank-0 workers.

        Only transfer_ready requires non-rank-0 participation (for compute).
        transfer_alloc and transfer_push are rank-0-only operations — skip them.
        """
        msg = decode_transfer_msg(frames)
        msg_type = msg.get("msg_type", "")

        if msg_type == TransferMsgType.READY:
            # Non-rank-0 has no TransferManager, so rank 0 loads tensors from
            # the RDMA buffer and broadcasts the full Req (scalars + tensors)
            # over NCCL. Participate in the matching broadcast here.
            req = self._broadcast_req_to_all_ranks(None)
            self._disagg_compute_non_rank0(req)
```
**EN:** This block defines method `_handle_transfer_non_rank0` on `SchedulerDisaggMixin`. Handle transfer messages on non-rank-0 workers. Only transfer_ready requires non-rank-0 participation (for compute). Key calls include `decode_transfer_msg`, `msg.get`, `self._broadcast_req_to_all_ranks`, and `self._disagg_compute_non_rank0`. The implementation branches on conditions. Parameters such as `frames` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_handle_transfer_non_rank0`。 它用于处理 handle transfer non rank0 相关逻辑。 关键调用包括 `decode_transfer_msg`、`msg.get`、`self._broadcast_req_to_all_ranks` 和 `self._disagg_compute_non_rank0`。 实现中包含条件分支。 本段逻辑主要由 `frames` 等参数驱动。

### Lines 1119-1149: `_handle_transfer_alloc` implementation / `_handle_transfer_alloc` 实现
```python
    def _handle_transfer_alloc(self: Scheduler, msg: dict) -> None:
        """Handle transfer_alloc: allocate a receive slot and reply with transfer_allocated."""
        request_id = msg["request_id"]
        data_size = msg.get("data_size", 0)

        pending = self._transfer_manager.allocate_receive_slot(request_id, data_size)
        if pending is None:
            logger.error(
                "Transfer %s: failed to allocate receive slot for %s (%d bytes)",
                self._disagg_role.value.upper(),
                request_id,
                data_size,
            )
            return

        allocated_msg = TransferAllocatedMsg(
            request_id=request_id,
            session_id=self._transfer_manager.session_id,
            pool_ptr=self._transfer_manager.pool_data_ptr,
            slot_offset=pending.slot.offset,
            slot_size=pending.slot.size,
        )
        self._pool_result_push.send_multipart(encode_transfer_msg(allocated_msg))

        logger.debug(
            "Transfer %s: allocated receive slot for %s (offset=%d, size=%d)",
            self._disagg_role.value.upper(),
            request_id,
            pending.slot.offset,
            pending.slot.size,
        )
```
**EN:** This block defines method `_handle_transfer_alloc` on `SchedulerDisaggMixin`. Handle transfer_alloc: allocate a receive slot and reply with transfer_allocated. Key calls include `msg.get`, `self._transfer_manager.allocate_receive_slot`, `TransferAllocatedMsg`, `self._pool_result_push.send_multipart`, and `logger.debug`. The implementation branches on conditions. Parameters such as `msg` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_handle_transfer_alloc`。 它用于处理 handle transfer alloc 相关逻辑。 关键调用包括 `msg.get`、`self._transfer_manager.allocate_receive_slot`、`TransferAllocatedMsg`、`self._pool_result_push.send_multipart` 和 `logger.debug`。 实现中包含条件分支。 本段逻辑主要由 `msg` 等参数驱动。

### Lines 1151-1193: `_handle_transfer_push` implementation / `_handle_transfer_push` 实现
```python
    def _handle_transfer_push(self: Scheduler, msg: dict) -> None:
        """Handle transfer_push: RDMA push staged data to peer, reply with transfer_pushed.

        If RDMA push thread is active, enqueue non-blocking.
        Otherwise fall back to blocking push (e.g., during shutdown).
        """
        request_id = msg["request_id"]
        dest_session_id = msg.get("dest_session_id", "")
        dest_addr = msg.get("dest_addr", 0)
        transfer_size = msg.get("transfer_size", 0)

        if self._rdma_push_queue is not None:
            # Non-blocking: enqueue to RDMA push thread
            self._rdma_push_queue.put(
                (
                    request_id,
                    dest_session_id,
                    dest_addr,
                    transfer_size,
                )
            )
            return

        # Fallback: blocking push on main thread
        success = self._transfer_manager.push_to_peer(
            request_id=request_id,
            dest_session_id=dest_session_id,
            dest_addr=dest_addr,
            transfer_size=transfer_size,
        )

        if success:
            self._transfer_manager.free_staged(request_id)

        pushed_msg = TransferPushedMsg(request_id=request_id)
        self._pool_result_push.send_multipart(encode_transfer_msg(pushed_msg))

        if not success:
            logger.error(
                "Transfer %s: RDMA push failed for %s",
                self._disagg_role.value.upper(),
                request_id,
            )
```
**EN:** This block defines method `_handle_transfer_push` on `SchedulerDisaggMixin`. Handle transfer_push: RDMA push staged data to peer, reply with transfer_pushed. If RDMA push thread is active, enqueue non-blocking. Key calls include `msg.get`, `self._transfer_manager.push_to_peer`, `TransferPushedMsg`, `self._pool_result_push.send_multipart`, and `self._rdma_push_queue.put`. The implementation branches on conditions. Parameters such as `msg` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_handle_transfer_push`。 它用于处理 handle transfer push 相关逻辑。 关键调用包括 `msg.get`、`self._transfer_manager.push_to_peer`、`TransferPushedMsg`、`self._pool_result_push.send_multipart` 和 `self._rdma_push_queue.put`。 实现中包含条件分支。 本段逻辑主要由 `msg` 等参数驱动。

### Lines 1195-1262: `_handle_transfer_ready` implementation / `_handle_transfer_ready` 实现
```python
    def _handle_transfer_ready(self: Scheduler, msg: dict) -> None:
        """Handle transfer_ready: load tensors from buffer, run compute, send result.

        Overlap tensor load with Req construction and scheduler init.
        After the RDMA data arrives:
        1. Start load on transfer_stream (non-blocking)
        2. Build Req from scalar fields + tensors (CPU, overlapped)
        3. Init scheduler timesteps if denoiser (CPU, overlapped)
        4. Wait for load before compute
        5. Run the role's compute
        """

        request_id = msg["request_id"]
        manifest = msg.get("manifest", {})
        scalar_fields = msg.get("scalar_fields", {})
        role_name = self._disagg_role.value.upper()

        if self._disagg_metrics:
            self._disagg_metrics.record_request_start(request_id)

        # If using a pre-allocated slot, register it as pending receive
        prealloc_slot_id = scalar_fields.pop("_prealloc_slot_id", None)
        if (
            prealloc_slot_id is not None
            and prealloc_slot_id in self._preallocated_slots
        ):
            slot = self._preallocated_slots[prealloc_slot_id]
            self._transfer_manager.register_prealloc_as_receive(request_id, slot)

        # 1. Start load on transfer_stream (non-blocking)
        local_device = f"cuda:{self.worker.local_rank}"
        tensors, load_event = self._transfer_manager.load_tensors_async(
            request_id,
            manifest,
            device=local_device,
            stream=self._transfer_stream,
        )

        # 2. Build Req from scalar fields + tensors (CPU work, overlapped)
        req = self._build_disagg_req(scalar_fields, tensors)

        # 3. Init scheduler timesteps if denoiser (CPU work, overlapped)
        if self._disagg_role == RoleType.DENOISER:
            _init_disagg_request_scheduler(self, req)

        # 4. Wait for load before compute (GPU must see the data)
        if load_event is not None:
            torch.cuda.current_stream().wait_event(load_event)

        # 5. Free receive slot after load completes (data is on compute GPU)
        if prealloc_slot_id is not None:
            # Pre-allocated slot: just remove from pending receives, don't free buffer
            with self._transfer_manager._lock:
                self._transfer_manager._pending_receives.pop(request_id, None)
        else:
            self._transfer_manager.free_receive_slot(request_id)

        # 6. In multi-rank mode, broadcast the fully-loaded Req to the other
        # ranks so REPLICATED stages see identical inputs everywhere. See
        # the prefetch-loop variant for the matching receiver broadcast.
        if self._is_multi_rank():
            self._broadcast_req_to_all_ranks(req)

        # 7. Run compute
        if self._disagg_role == RoleType.DENOISER:
            self._disagg_denoiser_compute(req, request_id, role_name)
        elif self._disagg_role == RoleType.DECODER:
            self._disagg_decoder_compute(req, request_id, role_name)
```
**EN:** This block defines method `_handle_transfer_ready` on `SchedulerDisaggMixin`. Handle transfer_ready: load tensors from buffer, run compute, send result. Overlap tensor load with Req construction and scheduler init. Key calls include `msg.get`, `self._disagg_role.value.upper`, `scalar_fields.pop`, `self._transfer_manager.load_tensors_async`, and `self._build_disagg_req`. The implementation branches on conditions, uses context-managed resources. Parameters such as `msg` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_handle_transfer_ready`。 它用于处理 handle transfer ready 相关逻辑。 关键调用包括 `msg.get`、`self._disagg_role.value.upper`、`scalar_fields.pop`、`self._transfer_manager.load_tensors_async` 和 `self._build_disagg_req`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `msg` 等参数驱动。

### Lines 1268-1291: `_disagg_compute_non_rank0` implementation / `_disagg_compute_non_rank0` 实现
```python
    def _disagg_compute_non_rank0(self: Scheduler, req: Req) -> None:
        """Non-rank-0 compute: enter execute_forward with a Req received via
        NCCL broadcast from rank 0.

        The Req already contains tensor fields materialized on this rank's
        GPU (see ``_broadcast_req_to_all_ranks``), so REPLICATED stages such
        as denoising have non-empty prompt_embeds and verify_input passes.

        Used by both the non-prefetch path (:meth:`_handle_transfer_non_rank0`)
        and the prefetch non-rank-0 loop
        (:meth:`_disagg_non_rank0_event_loop`).
        """
        if self._disagg_role == RoleType.DENOISER:
            # Initialize scheduler timesteps (same as rank 0)
            _init_disagg_request_scheduler(self, req)

            with self._disagg_trace_dispatch(req):
                self.worker.execute_forward([req], return_req=True)

        elif self._disagg_role == RoleType.DECODER:
            req.save_output = False
            req.return_file_paths_only = False
            with self._disagg_trace_dispatch(req):
                self.worker.execute_forward([req])
```
**EN:** This block defines method `_disagg_compute_non_rank0` on `SchedulerDisaggMixin`. Non-rank-0 compute: enter execute_forward with a Req received via NCCL broadcast from rank 0. The Req already contains tensor fields materialized on this rank's GPU (see ``_broadcast_req_to_all_ranks``), so REPLICATED stages such as denoising have non-empty prompt_embeds and verify_input passes. Key calls include `_init_disagg_request_scheduler`, `self._disagg_trace_dispatch`, and `self.worker.execute_forward`. The implementation branches on conditions, uses context-managed resources. Parameters such as `req` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_disagg_compute_non_rank0`。 它用于处理 disagg compute non rank0 相关逻辑。 关键调用包括 `_init_disagg_request_scheduler`、`self._disagg_trace_dispatch` 和 `self.worker.execute_forward`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `req` 等参数驱动。

### Lines 1293-1341: `_build_disagg_req` implementation / `_build_disagg_req` 实现
```python
    def _build_disagg_req(self: Scheduler, scalar_fields: dict, tensors: dict) -> Req:
        """Reconstruct a Req from transfer scalar fields and loaded GPU tensors.

        Initializes all dataclass field defaults first, then overlays
        scalar and tensor fields from the transfer message.
        """
        # Pop _trace_state before the generic setattr loop so it doesn't land
        # on the Req as a stray attribute.
        trace_state = scalar_fields.pop("_trace_state", None)

        req = object.__new__(Req)
        # Initialize all dataclass fields with their defaults
        for f in dataclasses.fields(Req):
            if f.default is not dataclasses.MISSING:
                object.__setattr__(req, f.name, f.default)
            elif f.default_factory is not dataclasses.MISSING:
                object.__setattr__(req, f.name, f.default_factory())
        # Ensure sampling_params is not None so __getattr__ delegation works
        object.__setattr__(req, "sampling_params", SamplingParams())
        # Restore _extra_* prefixed fields into req.extra dict
        extra_keys = [k for k in scalar_fields if k.startswith("_extra_")]
        for key in extra_keys:
            req.extra[key[len("_extra_") :]] = scalar_fields.pop(key)
        for key, value in scalar_fields.items():
            setattr(req, key, value)
        # Set tensor fields
        for key, value in tensors.items():
            setattr(req, key, value)
        # Recreate torch.Generator from seed (not serializable over transfer)
        seed = scalar_fields.get("seed")
        if seed is not None:
            if isinstance(seed, list):
                req.generator = [
                    torch.Generator(device="cpu").manual_seed(int(item))
                    for item in seed
                ]
            else:
                req.generator = torch.Generator(device="cpu").manual_seed(int(seed))
        # Rebuild trace_ctx from the propagated __getstate__ dict so this role's
        # spans nest under the sender's trace (same mechanism SRT uses via pickle).
        if trace_state and trace_state.get("tracing_enable"):
            try:
                ctx = object.__new__(TraceReqContext)
                ctx.__setstate__(trace_state)
                req.trace_ctx = ctx
            except Exception as e:
                logger.debug("Failed to rebuild trace_ctx from _trace_state: %s", e)
        req.validate()
        return req
```
**EN:** This block defines method `_build_disagg_req` on `SchedulerDisaggMixin`. Reconstruct a Req from transfer scalar fields and loaded GPU tensors. Initializes all dataclass field defaults first, then overlays scalar and tensor fields from the transfer message. Key calls include `scalar_fields.pop`, `object.__new__`, `dataclasses.fields`, `object.__setattr__`, and `scalar_fields.items`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `scalar_fields`, and `tensors` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_build_disagg_req`。 它用于构建disagg req。 关键调用包括 `scalar_fields.pop`、`object.__new__`、`dataclasses.fields`、`object.__setattr__` 和 `scalar_fields.items`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `scalar_fields` 和 `tensors` 等参数驱动。

### Lines 1343-1366: `_disagg_trace_dispatch` implementation / `_disagg_trace_dispatch` 实现
```python
    @contextlib.contextmanager
    def _disagg_trace_dispatch(self: Scheduler, req: Req):
        """Wrap a disagg role's worker.execute_forward in the tracing lifecycle.

        Mirrors the monolithic path in ``scheduler._handle_generation``: rebuild
        the thread context under the (potentially remote) root_span_context that
        was propagated in via ``_trace_state`` / pickle, then emit a
        ``scheduler_dispatch`` span for this role with ``thread_finish_flag``
        so the thread span closes when compute returns. If tracing is disabled
        (TraceNullContext), everything is a no-op.
        """
        ctx = getattr(req, "trace_ctx", None)
        if ctx is None:
            yield
            return
        # Disagg receive (__setstate__) and compute may run on different
        # threads (e.g. recv-prefetch vs scheduler main). Align the ctx's pid
        # with the current compute thread so __create_thread_context's
        # threads_info lookup resolves via the local registration.
        if getattr(ctx, "tracing_enable", False):
            ctx.pid = threading.get_native_id()
        ctx.rebuild_thread_context()
        with trace_slice(ctx, DiffStage.SCHEDULER_DISPATCH, thread_finish_flag=True):
            yield
```
**EN:** This block defines method `_disagg_trace_dispatch` on `SchedulerDisaggMixin`. Wrap a disagg role's worker.execute_forward in the tracing lifecycle. Mirrors the monolithic path in ``scheduler._handle_generation``: rebuild the thread context under the (potentially remote) root_span_context that was propagated in via ``_trace_state`` / pickle, then emit a ``scheduler_dispatch`` span for this role with ``thread_finish_flag`` so the thread span closes when compute returns. Key calls include `getattr`, `ctx.rebuild_thread_context`, `threading.get_native_id`, and `trace_slice`. The implementation branches on conditions, uses context-managed resources. Parameters such as `req` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_disagg_trace_dispatch`。 它用于处理 disagg trace dispatch 相关逻辑。 关键调用包括 `getattr`、`ctx.rebuild_thread_context`、`threading.get_native_id` 和 `trace_slice`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `req` 等参数驱动。

### Lines 1368-1439: `_disagg_denoiser_compute` implementation / `_disagg_denoiser_compute` 实现
```python
    def _disagg_denoiser_compute(
        self: Scheduler, req: Req, request_id: str, role_name: str
    ) -> None:
        """Run denoiser compute in transfer mode, then stage output for decoder.

        Note: Scheduler timestep init is done in _handle_transfer_ready
        to overlap with tensor loading.
        """
        # Run denoising
        start_time = time.monotonic()
        with self._disagg_trace_dispatch(req):
            result = self.worker.execute_forward([req], return_req=True)
        duration_s = time.monotonic() - start_time

        if not isinstance(result, Req):
            error_msg = getattr(result, "error", "denoiser error")
            done_msg = TransferDoneMsg(request_id=request_id, error=str(error_msg))
            self._pool_result_push.send_multipart(encode_transfer_msg(done_msg))
            if self._disagg_metrics:
                self._disagg_metrics.record_request_failed(request_id)
            return

        # Stage denoiser output for decoder transfer (async staging)
        tensor_fields, scalar_fields = extract_transfer_fields(result)

        # 1. Stage tensors on transfer_stream (non-blocking)
        staged, stage_event = self._transfer_manager.stage_tensors_async(
            request_id=request_id,
            tensor_fields=tensor_fields,
            scalar_fields=scalar_fields,
            stream=self._transfer_stream,
        )

        if staged is None:
            done_msg = TransferDoneMsg(
                request_id=request_id,
                error="Failed to stage denoiser output for decoder",
            )
            self._pool_result_push.send_multipart(encode_transfer_msg(done_msg))
            if self._disagg_metrics:
                self._disagg_metrics.record_request_failed(request_id)
            return

        # 2. Build done_data dict while staging runs (CPU work, overlapped)
        done_data = {
            "msg_type": "transfer_done",
            "request_id": request_id,
            "staged_for_decoder": True,
            "session_id": self._transfer_manager.session_id,
            "pool_ptr": self._transfer_manager.pool_data_ptr,
            "slot_offset": staged.slot.offset if staged.slot else 0,
            "data_size": staged.slot.size if staged.slot else 0,
            "manifest": staged.manifest,
            "scalar_fields": staged.scalar_fields,
        }
        msg_bytes = json.dumps(done_data, separators=(",", ":")).encode("utf-8")

        # 3. Wait for staging to complete before sending
        if stage_event is not None:
            stage_event.synchronize()

        # 4. Send transfer_done with staged info
        self._pool_result_push.send_multipart([TRANSFER_MAGIC, msg_bytes])

        if self._disagg_metrics:
            self._disagg_metrics.record_request_complete(request_id)

        logger.debug(
            "Transfer DENOISER: processed %s in %.2f s, staged for decoder",
            request_id,
            duration_s,
        )
```
**EN:** This block defines method `_disagg_denoiser_compute` on `SchedulerDisaggMixin`. Run denoiser compute in transfer mode, then stage output for decoder. Note: Scheduler timestep init is done in _handle_transfer_ready to overlap with tensor loading. Key calls include `time.monotonic`, `extract_transfer_fields`, `self._transfer_manager.stage_tensors_async`, `json.dumps.encode`, and `self._pool_result_push.send_multipart`. The implementation branches on conditions, uses context-managed resources. Parameters such as `req`, `request_id`, and `role_name` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_disagg_denoiser_compute`。 它用于处理 disagg denoiser compute 相关逻辑。 关键调用包括 `time.monotonic`、`extract_transfer_fields`、`self._transfer_manager.stage_tensors_async`、`json.dumps.encode` 和 `self._pool_result_push.send_multipart`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `req`、`request_id` 和 `role_name` 等参数驱动。

### Lines 1441-1496: `_disagg_decoder_compute` implementation / `_disagg_decoder_compute` 实现
```python
    def _disagg_decoder_compute(
        self: Scheduler, req: Req, request_id: str, role_name: str
    ) -> None:
        """Run decoder compute in transfer mode, send result to DS.

        Decoder result is sent as raw ZMQ multipart frames (same format as
        relay mode) so DiffusionServer handles it via _handle_decoder_result_frames
        without hex/JSON overhead.
        """

        # Check for upstream error
        disagg_error = getattr(req, "_disagg_error", None)
        if disagg_error:
            if self._pool_result_push is not None:
                send_tensors(
                    self._pool_result_push,
                    {},
                    {
                        "request_id": request_id,
                        "error": f"Upstream error: {disagg_error}",
                    },
                )
            return

        req.save_output = False
        req.return_file_paths_only = False

        start_time = time.monotonic()
        with self._disagg_trace_dispatch(req):
            output_batch = self.worker.execute_forward([req])
        duration_s = time.monotonic() - start_time

        # Send result as raw ZMQ frames (no TRANSFER_MAGIC prefix).
        # DiffusionServer will route it through _handle_decoder_result_frames,
        # the same path as relay mode.
        tensor_fields = {}
        scalar_fields = {"request_id": request_id}
        if output_batch.output is not None:
            tensor_fields["output"] = output_batch.output
        if output_batch.audio is not None:
            tensor_fields["audio"] = output_batch.audio
        if output_batch.audio_sample_rate is not None:
            scalar_fields["audio_sample_rate"] = output_batch.audio_sample_rate
        if output_batch.error is not None:
            scalar_fields["error"] = output_batch.error

        if self._pool_result_push is not None:
            send_tensors(self._pool_result_push, tensor_fields, scalar_fields)

        if self._disagg_metrics:
            if output_batch.error:
                self._disagg_metrics.record_request_failed(request_id)
            else:
                self._disagg_metrics.record_request_complete(request_id)

        logger.debug("Transfer DECODER: processed %s in %.2f s", request_id, duration_s)
```
**EN:** This block defines method `_disagg_decoder_compute` on `SchedulerDisaggMixin`. Run decoder compute in transfer mode, send result to DS. Decoder result is sent as raw ZMQ multipart frames (same format as relay mode) so DiffusionServer handles it via _handle_decoder_result_frames without hex/JSON overhead. Key calls include `getattr`, `time.monotonic`, `logger.debug`, `self._disagg_trace_dispatch`, and `self.worker.execute_forward`. The implementation branches on conditions, uses context-managed resources. Parameters such as `req`, `request_id`, and `role_name` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_disagg_decoder_compute`。 它用于处理 disagg decoder compute 相关逻辑。 关键调用包括 `getattr`、`time.monotonic`、`logger.debug`、`self._disagg_trace_dispatch` 和 `self.worker.execute_forward`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `req`、`request_id` 和 `role_name` 等参数驱动。

### Lines 1498-1555: `_disagg_encoder_step` implementation / `_disagg_encoder_step` 实现
```python
    def _disagg_encoder_step(
        self: Scheduler,
        send_tensors_fn,
        frames=None,
    ):
        """Single encoder step in pool mode."""
        # Receive: [request_id_bytes, pickled_req_bytes]
        if frames is None:
            frames = self._pool_work_pull.recv_multipart()
        pickled_req = frames[-1]
        reqs = pickle.loads(pickled_req)
        if not isinstance(reqs, list):
            reqs = [reqs]

        req = reqs[0]
        request_id = getattr(req, "request_id", "unknown")

        if self._disagg_metrics:
            self._disagg_metrics.record_request_start(request_id)

        # Run encoder stages
        with self._disagg_trace_dispatch(req):
            req_result = self.worker.execute_forward(reqs, return_req=True)

        if not isinstance(req_result, Req):
            # Error — send error via scalar fields (rank 0 only)
            if self._pool_result_push is not None:
                error_msg = getattr(req_result, "error", "encoder error")
                send_tensors_fn(
                    self._pool_result_push,
                    {},
                    {"request_id": request_id, "_disagg_error": str(error_msg)},
                )
            if self._disagg_metrics:
                self._disagg_metrics.record_request_failed(request_id)
            return

        # Pack and send encoder output (rank 0 only sends)
        tensor_fields, scalar_fields = extract_transfer_fields(req_result)

        if self._pool_result_push is not None:
            if self._transfer_manager is not None:
                # Transfer mode: stage tensors to TransferBuffer, send transfer_staged
                self._disagg_encoder_transfer_stage(
                    request_id, tensor_fields, scalar_fields
                )
            else:
                # Fallback: send error (transfer manager not initialized)
                send_tensors_fn(
                    self._pool_result_push,
                    {},
                    {"request_id": request_id, "_disagg_error": "No transfer manager"},
                )

        if self._disagg_metrics:
            self._disagg_metrics.record_request_complete(request_id)

        logger.debug("Pool ENCODER: processed %s", request_id)
```
**EN:** This block defines method `_disagg_encoder_step` on `SchedulerDisaggMixin`. Single encoder step in pool mode. Key calls include `pickle.loads`, `getattr`, `extract_transfer_fields`, `logger.debug`, and `self._pool_work_pull.recv_multipart`. The implementation branches on conditions, uses context-managed resources. Parameters such as `send_tensors_fn`, and `frames` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_disagg_encoder_step`。 它用于处理 disagg encoder step 相关逻辑。 关键调用包括 `pickle.loads`、`getattr`、`extract_transfer_fields`、`logger.debug` 和 `self._pool_work_pull.recv_multipart`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `send_tensors_fn` 和 `frames` 等参数驱动。

### Lines 1557-1601: `_disagg_encoder_transfer_stage` implementation / `_disagg_encoder_transfer_stage` 实现
```python
    def _disagg_encoder_transfer_stage(
        self: Scheduler, request_id: str, tensor_fields: dict, scalar_fields: dict
    ) -> None:
        """Stage encoder output and send transfer_staged to DS.

        Overlap staging with metadata JSON serialization.
        """
        # 1. Stage tensors on transfer_stream (non-blocking)
        staged, stage_event = self._transfer_manager.stage_tensors_async(
            request_id=request_id,
            tensor_fields=tensor_fields,
            scalar_fields=scalar_fields,
            stream=self._transfer_stream,
        )

        if staged is None:
            # Staging failed — send error via relay as fallback
            send_tensors(
                self._pool_result_push,
                {},
                {"request_id": request_id, "_disagg_error": "Transfer staging failed"},
            )
            if self._disagg_metrics:
                self._disagg_metrics.record_request_failed(request_id)
            return

        # 2. Build transfer metadata dict while staging runs (CPU work, overlapped)
        staged_data = {
            "msg_type": "transfer_staged",
            "request_id": request_id,
            "data_size": staged.slot.size if staged.slot else 0,
            "manifest": staged.manifest,
            "session_id": self._transfer_manager.session_id,
            "pool_ptr": self._transfer_manager.pool_data_ptr,
            "slot_offset": staged.slot.offset if staged.slot else 0,
            "scalar_fields": staged.scalar_fields,
        }
        msg_bytes = json.dumps(staged_data, separators=(",", ":")).encode("utf-8")

        # 3. Wait for staging to complete before sending (buffer must be ready)
        if stage_event is not None:
            stage_event.synchronize()

        # 4. Send transfer staged message
        self._pool_result_push.send_multipart([TRANSFER_MAGIC, msg_bytes])
```
**EN:** This block defines method `_disagg_encoder_transfer_stage` on `SchedulerDisaggMixin`. Stage encoder output and send transfer_staged to DS. Overlap staging with metadata JSON serialization. Key calls include `self._transfer_manager.stage_tensors_async`, `json.dumps.encode`, `self._pool_result_push.send_multipart`, `send_tensors`, and `stage_event.synchronize`. The implementation branches on conditions. Parameters such as `request_id`, `tensor_fields`, and `scalar_fields` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerDisaggMixin` 的方法 `_disagg_encoder_transfer_stage`。 它用于处理 disagg encoder transfer stage 相关逻辑。 关键调用包括 `self._transfer_manager.stage_tensors_async`、`json.dumps.encode`、`self._pool_result_push.send_multipart`、`send_tensors` 和 `stage_event.synchronize`。 实现中包含条件分支。 本段逻辑主要由 `request_id`、`tensor_fields` 和 `scalar_fields` 等参数驱动。

## Key Concepts / 关键概念
- `_is_tensor_like`: Top-level function that handles is tensor like logic. / 顶层函数，用于处理 is tensor like 相关逻辑。
- `_to_json_serializable`: Top-level function that converts to json serializable. / 顶层函数，用于转换为json serializable。
- `_is_default`: Top-level function that handles is default logic. / 顶层函数，用于处理 is default 相关逻辑。
- `_extract_extra_fields`: Extract JSON-serializable entries from Req.extra into scalar_fields. / 顶层函数，用于处理 extract extra fields 相关逻辑。
- `_init_request_scheduler_from_template`: Top-level function that initializes request scheduler from template. / 顶层函数，用于初始化request scheduler from template。
- `_init_disagg_request_scheduler`: Top-level function that initializes disagg request scheduler. / 顶层函数，用于初始化disagg request scheduler。
- `extract_transfer_fields`: Extract all transferable fields from a Req, split into tensors and scalars. / 顶层函数，用于处理 extract transfer fields 相关逻辑。
- `_pack_tensor_fields_for_broadcast`: Pack ``tensor_fields`` into a structure ``broadcast_tensor_dict`` accepts. / 顶层函数，用于打包tensor fields for broadcast。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `contextlib`, `dataclasses`, `inspect`, `json`, `logging`, `pickle`, `queue`, `threading`, `time`
- **Third-party / 第三方依赖**: `torch`, `zmq`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.disaggregation.roles`, `sglang.multimodal_gen.runtime.disaggregation.transport.buffer`, `sglang.multimodal_gen.runtime.disaggregation.transport.codec`, `sglang.multimodal_gen.runtime.disaggregation.transport.engine`, `sglang.multimodal_gen.runtime.disaggregation.transport.manager`, `sglang.multimodal_gen.runtime.disaggregation.transport.protocol`, `sglang.multimodal_gen.runtime.pipelines_core`, `sglang.multimodal_gen.runtime.pipelines_core.diffusion_scheduler_utils`, `sglang.multimodal_gen.runtime.utils.common`, `sglang.multimodal_gen.runtime.utils.distributed`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 1601
