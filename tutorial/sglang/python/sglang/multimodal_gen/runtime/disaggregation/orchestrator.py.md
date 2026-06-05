# orchestrator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/orchestrator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `_EncoderTTAEntry`, `_TransferRequestState`, and `_RoleTTAEntry`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Central request router for disaggregated diffusion pipelines. / 该文件属于解耦运行时层。它围绕 `_EncoderTTAEntry`、`_TransferRequestState` 和 `_RoleTTAEntry` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-34: module setup and imports / 模块初始化与导入
```python
"""Central request router for disaggregated diffusion pipelines."""

import json
import logging
import pickle
import threading
import time
from collections import deque
from dataclasses import dataclass

import zmq

from sglang.multimodal_gen.runtime.disaggregation.dispatch_policy import (
    PoolDispatcher,
)
from sglang.multimodal_gen.runtime.disaggregation.request_state import (
    RequestState,
    RequestTracker,
)
from sglang.multimodal_gen.runtime.disaggregation.roles import RoleType
from sglang.multimodal_gen.runtime.disaggregation.transport.codec import (
    unpack_tensors,
)
from sglang.multimodal_gen.runtime.disaggregation.transport.protocol import (
    TransferAllocMsg,
    TransferMsgType,
    TransferPushMsg,
    TransferReadyMsg,
    decode_transfer_msg,
    encode_transfer_msg,
    is_transfer_message,
)
from sglang.multimodal_gen.runtime.utils.common import get_zmq_socket
```
**EN:** This block establishes the module context and imports `json`, `logging`, `pickle`, `threading`, `time`, and `collections`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `json`、`logging`、`pickle`、`threading`、`time` 和 `collections`。这些依赖为后续实现提供所需符号。

### Lines 36-36: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `logging.getLogger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `logging.getLogger` 协同工作。

### Lines 40-40: `_EncoderTTAEntry` class overview / `_EncoderTTAEntry` 类概览
```python
class _EncoderTTAEntry:
```
**EN:** This block defines class `_EncoderTTAEntry`. It encapsulates encoder ttaentry behavior.
**CN:** 该代码块定义了类 `_EncoderTTAEntry`。 它用于封装 encoder ttaentry 相关行为。

### Lines 41-43: supporting statements / 辅助语句
```python
    request_id: str
    client_identity: bytes
    payload: bytes
```
**EN:** This block gathers supporting statements inside `_EncoderTTAEntry`. It updates names such as `request_id`, `client_identity`, and `payload`.
**CN:** 该代码块汇集了位于 `_EncoderTTAEntry` 内部的辅助语句。 它会更新 `request_id`、`client_identity` 和 `payload` 等名称。

### Lines 47-47: `_TransferRequestState` class overview / `_TransferRequestState` 类概览
```python
class _TransferRequestState:
```
**EN:** This block defines class `_TransferRequestState`. It encapsulates transfer request state behavior.
**CN:** 该代码块定义了类 `_TransferRequestState`。 它用于封装 transfer request state 相关行为。

### Lines 48-59: supporting statements / 辅助语句
```python
    sender_session_id: str = ""
    sender_pool_ptr: int = 0
    sender_slot_offset: int = 0
    data_size: int = 0
    manifest: dict = None
    scalar_fields: dict = None
    receiver_session_id: str = ""
    receiver_pool_ptr: int = 0
    receiver_slot_offset: int = 0
    sender_instance: int = -1
    receiver_instance: int = -1
    prealloc_slot_id: int | None = None
```
**EN:** This block gathers supporting statements inside `_TransferRequestState`. It updates names such as `sender_session_id`, `sender_pool_ptr`, `sender_slot_offset`, `data_size`, `manifest`, and `scalar_fields`.
**CN:** 该代码块汇集了位于 `_TransferRequestState` 内部的辅助语句。 它会更新 `sender_session_id`、`sender_pool_ptr`、`sender_slot_offset`、`data_size`、`manifest` 和 `scalar_fields` 等名称。

### Lines 61-65: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        if self.manifest is None:
            self.manifest = {}
        if self.scalar_fields is None:
            self.scalar_fields = {}
```
**EN:** This block defines method `__post_init__` on `_TransferRequestState`. It post-processes init. The implementation branches on conditions.
**CN:** 该代码块定义了 `_TransferRequestState` 的方法 `__post_init__`。 它用于后处理init。 实现中包含条件分支。

### Lines 69-69: `_RoleTTAEntry` class overview / `_RoleTTAEntry` 类概览
```python
class _RoleTTAEntry:
```
**EN:** This block defines class `_RoleTTAEntry`. It encapsulates role ttaentry behavior.
**CN:** 该代码块定义了类 `_RoleTTAEntry`。 它用于封装 role ttaentry 相关行为。

### Lines 70-71: supporting statements / 辅助语句
```python
    request_id: str
    transfer_state: _TransferRequestState | None = None
```
**EN:** This block gathers supporting statements inside `_RoleTTAEntry`. It updates names such as `request_id`, and `transfer_state`.
**CN:** 该代码块汇集了位于 `_RoleTTAEntry` 内部的辅助语句。 它会更新 `request_id` 和 `transfer_state` 等名称。

### Lines 74-79: `DiffusionServer` class overview / `DiffusionServer` 类概览
```python
class DiffusionServer:
    """Global pipeline orchestrator for N:M:K disaggregated diffusion.

    Capacity-aware dispatch with FreeBufferSlots per instance and TTA queues.
    """
```
**EN:** This block defines class `DiffusionServer`. Global pipeline orchestrator for N:M:K disaggregated diffusion. Capacity-aware dispatch with FreeBufferSlots per instance and TTA queues.
**CN:** 该代码块定义了类 `DiffusionServer`。 它用于封装 diffusion server 相关行为。

### Lines 80-157: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        frontend_endpoint: str,
        encoder_work_endpoints: list[str],
        denoiser_work_endpoints: list[str],
        decoder_work_endpoints: list[str],
        encoder_result_endpoint: str,
        denoiser_result_endpoint: str,
        decoder_result_endpoint: str,
        dispatch_policy_name: str = "round_robin",
        timeout_s: float = 600.0,
        encoder_capacity: int = 4,
        denoiser_capacity: int = 2,
        decoder_capacity: int = 4,
        p2p_mode: bool = True,
    ):
        self._frontend_endpoint = frontend_endpoint
        self._encoder_work_endpoints = encoder_work_endpoints
        self._denoiser_work_endpoints = denoiser_work_endpoints
        self._decoder_work_endpoints = decoder_work_endpoints
        self._encoder_result_endpoint = encoder_result_endpoint
        self._denoiser_result_endpoint = denoiser_result_endpoint
        self._decoder_result_endpoint = decoder_result_endpoint

        self._num_encoders = len(encoder_work_endpoints)
        self._num_denoisers = len(denoiser_work_endpoints)
        self._num_decoders = len(decoder_work_endpoints)
        self._timeout_s = timeout_s

        self._tracker = RequestTracker()
        self._dispatcher = PoolDispatcher(
            num_encoders=self._num_encoders,
            num_denoisers=self._num_denoisers,
            num_decoders=self._num_decoders,
            policy_name=dispatch_policy_name,
        )

        self._context = zmq.Context(io_threads=2)
        self._running = False
        self._ready = threading.Event()
        self._thread: threading.Thread | None = None

        self._pending: dict[str, bytes] = {}  # request_id -> client ZMQ identity
        self._lock = threading.Lock()

        # FreeBufferSlots per instance
        self._encoder_free_slots = [encoder_capacity] * self._num_encoders
        self._denoiser_free_slots = [denoiser_capacity] * self._num_denoisers
        self._decoder_free_slots = [decoder_capacity] * self._num_decoders

        # TTA queues per role type
        self._encoder_tta: deque[_EncoderTTAEntry] = deque()
        self._denoiser_tta: deque[_RoleTTAEntry] = deque()
        self._decoder_tta: deque[_RoleTTAEntry] = deque()

        self._transfer_mode = p2p_mode
        self._transfer_state: dict[str, _TransferRequestState] = {}

        # Per-instance registration: instance_idx -> {session_id, pool_ptr, pool_size}
        # Keyed by the same index used to build the PUSH work-socket list
        # (i.e. the index into --encoder/denoiser/decoder-urls). The index is
        # resolved from the registering instance's work_endpoint so the control
        # plane (work PUSH) and the data plane (RDMA session_id / pool_ptr /
        # preallocated slots) stay consistent regardless of startup order.
        self._encoder_peers: dict[int, dict] = {}
        self._denoiser_peers: dict[int, dict] = {}
        self._decoder_peers: dict[int, dict] = {}

        # work_endpoint -> index lookup tables, built from the --*-urls args
        self._encoder_endpoint_to_idx = {
            ep: i for i, ep in enumerate(encoder_work_endpoints)
        }
        self._denoiser_endpoint_to_idx = {
            ep: i for i, ep in enumerate(denoiser_work_endpoints)
        }
        self._decoder_endpoint_to_idx = {
            ep: i for i, ep in enumerate(decoder_work_endpoints)
        }
```
**EN:** This block defines method `__init__` on `DiffusionServer`. It initializes the instance state. Key calls include `len`, `RequestTracker`, `PoolDispatcher`, `zmq.Context`, and `threading.Event`. Parameters such as `frontend_endpoint`, `encoder_work_endpoints`, `denoiser_work_endpoints`, `decoder_work_endpoints`, and `encoder_result_endpoint` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `len`、`RequestTracker`、`PoolDispatcher`、`zmq.Context` 和 `threading.Event`。 本段逻辑主要由 `frontend_endpoint`、`encoder_work_endpoints`、`denoiser_work_endpoints`、`decoder_work_endpoints` 和 `encoder_result_endpoint` 等参数驱动。

### Lines 159-161: `tracker` implementation / `tracker` 实现
```python
    @property
    def tracker(self) -> RequestTracker:
        return self._tracker
```
**EN:** This block defines method `tracker` on `DiffusionServer`. It handles tracker logic.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `tracker`。 它用于处理 tracker 相关逻辑。

### Lines 163-165: `dispatcher` implementation / `dispatcher` 实现
```python
    @property
    def dispatcher(self) -> PoolDispatcher:
        return self._dispatcher
```
**EN:** This block defines method `dispatcher` on `DiffusionServer`. It handles dispatcher logic.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `dispatcher`。 它用于处理 dispatcher 相关逻辑。

### Lines 167-189: `start` implementation / `start` 实现
```python
    def start(self) -> None:
        if self._running:
            return
        self._running = True
        self._thread = threading.Thread(
            target=self._event_loop,
            name="DiffusionServer",
            daemon=True,
        )
        self._thread.start()
        logger.info(
            "DiffusionServer started: frontend=%s, "
            "%d encoder(s), %d denoiser(s), %d decoder(s), policy=%s, "
            "capacity=(%d/%d/%d)",
            self._frontend_endpoint,
            self._num_encoders,
            self._num_denoisers,
            self._num_decoders,
            type(self._dispatcher.encoder_policy).__name__,
            self._encoder_free_slots[0] if self._encoder_free_slots else 0,
            self._denoiser_free_slots[0] if self._denoiser_free_slots else 0,
            self._decoder_free_slots[0] if self._decoder_free_slots else 0,
        )
```
**EN:** This block defines method `start` on `DiffusionServer`. It handles start logic. Key calls include `threading.Thread`, `self._thread.start`, `logger.info`, and `type`. The implementation branches on conditions.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `start`。 它用于处理 start 相关逻辑。 关键调用包括 `threading.Thread`、`self._thread.start`、`logger.info` 和 `type`。 实现中包含条件分支。

### Lines 191-193: `wait_ready` implementation / `wait_ready` 实现
```python
    def wait_ready(self, timeout: float = 30.0) -> bool:
        """Block until the event loop has bound all sockets, or *timeout* elapses."""
        return self._ready.wait(timeout=timeout)
```
**EN:** This block defines method `wait_ready` on `DiffusionServer`. Block until the event loop has bound all sockets, or *timeout* elapses. Key calls include `self._ready.wait`. Parameters such as `timeout` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `wait_ready`。 它用于处理 wait ready 相关逻辑。 关键调用包括 `self._ready.wait`。 本段逻辑主要由 `timeout` 等参数驱动。

### Lines 195-199: `stop` implementation / `stop` 实现
```python
    def stop(self) -> None:
        self._running = False
        if self._thread is not None:
            self._thread.join(timeout=5.0)
            self._thread = None
```
**EN:** This block defines method `stop` on `DiffusionServer`. It handles stop logic. Key calls include `self._thread.join`. The implementation branches on conditions.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `stop`。 它用于处理 stop 相关逻辑。 关键调用包括 `self._thread.join`。 实现中包含条件分支。

### Lines 201-276: `_event_loop` implementation / `_event_loop` 实现
```python
    def _event_loop(self) -> None:
        frontend, _ = get_zmq_socket(
            self._context, zmq.ROUTER, self._frontend_endpoint, bind=True
        )

        encoder_pushes: list[zmq.Socket] = []
        for i, ep in enumerate(self._encoder_work_endpoints):
            sock, _ = get_zmq_socket(self._context, zmq.PUSH, ep, bind=False)
            encoder_pushes.append(sock)

        denoiser_pushes: list[zmq.Socket] = []
        for i, ep in enumerate(self._denoiser_work_endpoints):
            sock, _ = get_zmq_socket(self._context, zmq.PUSH, ep, bind=False)
            denoiser_pushes.append(sock)

        decoder_pushes: list[zmq.Socket] = []
        for i, ep in enumerate(self._decoder_work_endpoints):
            sock, _ = get_zmq_socket(self._context, zmq.PUSH, ep, bind=False)
            decoder_pushes.append(sock)

        encoder_result_pull, _ = get_zmq_socket(
            self._context, zmq.PULL, self._encoder_result_endpoint, bind=True
        )
        denoiser_result_pull, _ = get_zmq_socket(
            self._context, zmq.PULL, self._denoiser_result_endpoint, bind=True
        )
        decoder_result_pull, _ = get_zmq_socket(
            self._context, zmq.PULL, self._decoder_result_endpoint, bind=True
        )

        poller = zmq.Poller()
        poller.register(frontend, zmq.POLLIN)
        poller.register(encoder_result_pull, zmq.POLLIN)
        poller.register(denoiser_result_pull, zmq.POLLIN)
        poller.register(decoder_result_pull, zmq.POLLIN)

        self._encoder_pushes = encoder_pushes
        self._denoiser_pushes = denoiser_pushes
        self._decoder_pushes = decoder_pushes
        self._frontend = frontend

        self._ready.set()

        all_sockets = (
            [frontend, encoder_result_pull, denoiser_result_pull, decoder_result_pull]
            + encoder_pushes
            + denoiser_pushes
            + decoder_pushes
        )

        try:
            while self._running:
                events = dict(poller.poll(timeout=10))

                self._handle_timeouts()

                if frontend in events:
                    self._handle_client_request(frontend)

                if encoder_result_pull in events:
                    self._handle_role_result(encoder_result_pull, RoleType.ENCODER)

                if denoiser_result_pull in events:
                    self._handle_role_result(denoiser_result_pull, RoleType.DENOISER)

                if decoder_result_pull in events:
                    self._handle_role_result(decoder_result_pull, RoleType.DECODER)

                self._drain_all_queues()

        except Exception:
            logger.exception("DiffusionServer event loop error")
        finally:
            for sock in all_sockets:
                sock.close()
            self._context.destroy(linger=0)
```
**EN:** This block defines method `_event_loop` on `DiffusionServer`. It handles event loop logic. Key calls include `get_zmq_socket`, `enumerate`, `zmq.Poller`, `poller.register`, and `self._ready.set`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_event_loop`。 它用于处理 event loop 相关逻辑。 关键调用包括 `get_zmq_socket`、`enumerate`、`zmq.Poller`、`poller.register` 和 `self._ready.set`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。

### Lines 278-293: `_handle_role_result` implementation / `_handle_role_result` 实现
```python
    def _handle_role_result(self, result_pull: zmq.Socket, role: RoleType) -> None:
        try:
            frames = result_pull.recv_multipart(zmq.NOBLOCK, copy=True)
        except zmq.Again:
            return

        if is_transfer_message(frames):
            self._handle_transfer_result(frames, role)
            return

        if role == RoleType.DECODER:
            self._handle_decoder_result_frames(frames)
        else:
            # Non-transfer frames from encoder/denoiser are error results
            # sent via send_tensors (e.g., _disagg_error).
            self._handle_role_error_frames(frames, role)
```
**EN:** This block defines method `_handle_role_result` on `DiffusionServer`. It handles handle role result logic. Key calls include `is_transfer_message`, `result_pull.recv_multipart`, `self._handle_transfer_result`, `self._handle_decoder_result_frames`, and `self._handle_role_error_frames`. The implementation branches on conditions, handles exceptional paths. Parameters such as `result_pull`, and `role` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_handle_role_result`。 它用于处理 handle role result 相关逻辑。 关键调用包括 `is_transfer_message`、`result_pull.recv_multipart`、`self._handle_transfer_result`、`self._handle_decoder_result_frames` 和 `self._handle_role_error_frames`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `result_pull` 和 `role` 等参数驱动。

### Lines 295-328: `_handle_role_error_frames` implementation / `_handle_role_error_frames` 实现
```python
    def _handle_role_error_frames(self, frames: list, role: RoleType) -> None:
        """Handle non-transfer error results from encoder/denoiser roles."""
        try:
            tensor_fields, scalar_fields = unpack_tensors(frames, device="cpu")
        except Exception as e:
            logger.warning(
                "DiffusionServer: failed to unpack non-transfer frames from %s: %s",
                role.value,
                e,
            )
            return

        request_id = scalar_fields.get("request_id")
        disagg_error = scalar_fields.get("_disagg_error")

        if request_id and disagg_error:
            logger.error(
                "DiffusionServer: %s error for %s: %s",
                role.value,
                request_id,
                disagg_error,
            )
            self._complete_with_error(request_id, f"{role.value} error: {disagg_error}")
        elif request_id:
            logger.warning(
                "DiffusionServer: non-transfer frames from %s for %s without error",
                role.value,
                request_id,
            )
        else:
            logger.warning(
                "DiffusionServer: non-transfer frames from %s without request_id",
                role.value,
            )
```
**EN:** This block defines method `_handle_role_error_frames` on `DiffusionServer`. Handle non-transfer error results from encoder/denoiser roles. Key calls include `scalar_fields.get`, `unpack_tensors`, `logger.error`, `self._complete_with_error`, and `logger.warning`. The implementation branches on conditions, handles exceptional paths. Parameters such as `frames`, and `role` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_handle_role_error_frames`。 它用于处理 handle role error frames 相关逻辑。 关键调用包括 `scalar_fields.get`、`unpack_tensors`、`logger.error`、`self._complete_with_error` 和 `logger.warning`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `frames` 和 `role` 等参数驱动。

### Lines 330-391: `_handle_client_request` implementation / `_handle_client_request` 实现
```python
    def _handle_client_request(self, frontend: zmq.Socket) -> None:
        try:
            parts = frontend.recv_multipart(zmq.NOBLOCK)
        except zmq.Again:
            return

        if len(parts) < 3:
            return

        client_identity = parts[0]
        payload = parts[-1]

        try:
            reqs = pickle.loads(payload)
        except (pickle.UnpicklingError, EOFError):
            logger.warning("DiffusionServer: failed to deserialize request")
            return

        if not isinstance(reqs, list):
            reqs = [reqs]

        req = reqs[0]

        if isinstance(req, dict) or not hasattr(req, "request_id"):
            # Send empty reply so REQ socket doesn't hang
            try:
                frontend.send_multipart(
                    [client_identity, b"", pickle.dumps({"status": "ignored"})],
                    zmq.NOBLOCK,
                )
            except zmq.Again:
                pass
            return

        request_id = getattr(req, "request_id", None)
        if request_id is None:
            request_id = f"ds-{time.monotonic()}"

        try:
            self._tracker.submit(request_id)
        except ValueError:
            logger.warning("DiffusionServer: duplicate request_id %s", request_id)
            return

        with self._lock:
            self._pending[request_id] = client_identity

        try:
            self._tracker.transition(request_id, RequestState.ENCODER_WAITING)
        except ValueError:
            pass
        self._encoder_tta.append(
            _EncoderTTAEntry(
                request_id=request_id,
                client_identity=client_identity,
                payload=payload,
            )
        )
        logger.debug(
            "DiffusionServer: queued %s to encoder_tta",
            request_id,
        )
```
**EN:** This block defines method `_handle_client_request` on `DiffusionServer`. It handles handle client request logic. Key calls include `getattr`, `self._encoder_tta.append`, `logger.debug`, `frontend.recv_multipart`, and `len`. The implementation branches on conditions, handles exceptional paths, uses context-managed resources. Parameters such as `frontend` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_handle_client_request`。 它用于处理 handle client request 相关逻辑。 关键调用包括 `getattr`、`self._encoder_tta.append`、`logger.debug`、`frontend.recv_multipart` 和 `len`。 实现中包含条件分支，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `frontend` 等参数驱动。

### Lines 393-451: `_handle_decoder_result_frames` implementation / `_handle_decoder_result_frames` 实现
```python
    def _handle_decoder_result_frames(self, frames: list) -> None:
        from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import (
            OutputBatch,
        )

        request_id = self._extract_request_id(frames)
        if request_id is None:
            logger.warning("DiffusionServer: decoder result missing request_id")
            return

        logger.debug("DiffusionServer: decoder result %s", request_id)
        record = self._tracker.get(request_id)
        if record and record.decoder_instance is not None:
            self._decoder_free_slots[record.decoder_instance] += 1

        tensor_fields, scalar_fields = unpack_tensors(frames, device="cpu")

        output_batch = OutputBatch(
            output=tensor_fields.get("output"),
            audio=tensor_fields.get("audio"),
            audio_sample_rate=scalar_fields.get("audio_sample_rate"),
            error=scalar_fields.get("error"),
        )

        try:
            if output_batch.error:
                self._tracker.transition(
                    request_id, RequestState.FAILED, error=output_batch.error
                )
            else:
                self._tracker.transition(request_id, RequestState.DONE)
        except ValueError:
            pass

        with self._lock:
            client_identity = self._pending.pop(request_id, None)

        if client_identity is None:
            logger.warning(
                "DiffusionServer: no pending client for decoder result %s",
                request_id,
            )
            self._tracker.remove(request_id)
            return

        try:
            self._frontend.send_multipart(
                [client_identity, b"", pickle.dumps(output_batch)]
            )
        except zmq.ZMQError as e:
            logger.error(
                "DiffusionServer: failed to send result for %s: %s",
                request_id,
                e,
            )

        logger.debug("DiffusionServer: returned result for %s", request_id)
        self._transfer_state.pop(request_id, None)
        self._tracker.remove(request_id)
```
**EN:** This block defines method `_handle_decoder_result_frames` on `DiffusionServer`. It handles handle decoder result frames logic. Key calls include `self._extract_request_id`, `logger.debug`, `self._tracker.get`, `unpack_tensors`, and `OutputBatch`. The implementation branches on conditions, handles exceptional paths, uses context-managed resources. Parameters such as `frames` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_handle_decoder_result_frames`。 它用于处理 handle decoder result frames 相关逻辑。 关键调用包括 `self._extract_request_id`、`logger.debug`、`self._tracker.get`、`unpack_tensors` 和 `OutputBatch`。 实现中包含条件分支，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `frames` 等参数驱动。

### Lines 453-475: `_dispatch_to_encoder` implementation / `_dispatch_to_encoder` 实现
```python
    def _dispatch_to_encoder(
        self, request_id: str, payload: bytes, encoder_idx: int
    ) -> None:
        self._encoder_free_slots[encoder_idx] -= 1

        try:
            self._tracker.transition(
                request_id,
                RequestState.ENCODER_RUNNING,
                encoder_instance=encoder_idx,
            )
        except ValueError:
            pass

        self._encoder_pushes[encoder_idx].send_multipart(
            [request_id.encode("utf-8"), payload]
        )
        logger.debug(
            "DiffusionServer: dispatched %s to encoder[%d] (free=%d)",
            request_id,
            encoder_idx,
            self._encoder_free_slots[encoder_idx],
        )
```
**EN:** This block defines method `_dispatch_to_encoder` on `DiffusionServer`. It handles dispatch to encoder logic. Key calls include `self._encoder_pushes.send_multipart`, `logger.debug`, `self._tracker.transition`, and `request_id.encode`. The implementation handles exceptional paths. Parameters such as `request_id`, `payload`, and `encoder_idx` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_dispatch_to_encoder`。 它用于处理 dispatch to encoder 相关逻辑。 关键调用包括 `self._encoder_pushes.send_multipart`、`logger.debug`、`self._tracker.transition` 和 `request_id.encode`。 实现中处理异常路径。 本段逻辑主要由 `request_id`、`payload` 和 `encoder_idx` 等参数驱动。

### Lines 477-480: `_drain_all_queues` implementation / `_drain_all_queues` 实现
```python
    def _drain_all_queues(self) -> None:
        self._drain_encoder_tta()
        self._drain_denoiser_tta()
        self._drain_decoder_tta()
```
**EN:** This block defines method `_drain_all_queues` on `DiffusionServer`. It handles drain all queues logic. Key calls include `self._drain_encoder_tta`, `self._drain_denoiser_tta`, and `self._drain_decoder_tta`.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_drain_all_queues`。 它用于处理 drain all queues 相关逻辑。 关键调用包括 `self._drain_encoder_tta`、`self._drain_denoiser_tta` 和 `self._drain_decoder_tta`。

### Lines 482-490: `_drain_encoder_tta` implementation / `_drain_encoder_tta` 实现
```python
    def _drain_encoder_tta(self) -> None:
        while self._encoder_tta:
            idx = self._dispatcher.select_encoder_with_capacity(
                self._encoder_free_slots
            )
            if idx is None:
                break
            entry = self._encoder_tta.popleft()
            self._dispatch_to_encoder(entry.request_id, entry.payload, idx)
```
**EN:** This block defines method `_drain_encoder_tta` on `DiffusionServer`. It handles drain encoder tta logic. Key calls include `self._dispatcher.select_encoder_with_capacity`, `self._encoder_tta.popleft`, and `self._dispatch_to_encoder`. The implementation branches on conditions, iterates over collections or steps.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_drain_encoder_tta`。 它用于处理 drain encoder tta 相关逻辑。 关键调用包括 `self._dispatcher.select_encoder_with_capacity`、`self._encoder_tta.popleft` 和 `self._dispatch_to_encoder`。 实现中包含条件分支，会遍历集合或步骤。

### Lines 492-502: `_drain_denoiser_tta` implementation / `_drain_denoiser_tta` 实现
```python
    def _drain_denoiser_tta(self) -> None:
        while self._denoiser_tta:
            idx = self._dispatcher.select_denoiser_with_capacity(
                self._denoiser_free_slots
            )
            if idx is None:
                break
            entry = self._denoiser_tta.popleft()
            self._transfer_dispatch_to_denoiser(
                entry.request_id, entry.transfer_state, idx
            )
```
**EN:** This block defines method `_drain_denoiser_tta` on `DiffusionServer`. It handles drain denoiser tta logic. Key calls include `self._dispatcher.select_denoiser_with_capacity`, `self._denoiser_tta.popleft`, and `self._transfer_dispatch_to_denoiser`. The implementation branches on conditions, iterates over collections or steps.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_drain_denoiser_tta`。 它用于处理 drain denoiser tta 相关逻辑。 关键调用包括 `self._dispatcher.select_denoiser_with_capacity`、`self._denoiser_tta.popleft` 和 `self._transfer_dispatch_to_denoiser`。 实现中包含条件分支，会遍历集合或步骤。

### Lines 504-514: `_drain_decoder_tta` implementation / `_drain_decoder_tta` 实现
```python
    def _drain_decoder_tta(self) -> None:
        while self._decoder_tta:
            idx = self._dispatcher.select_decoder_with_capacity(
                self._decoder_free_slots
            )
            if idx is None:
                break
            entry = self._decoder_tta.popleft()
            self._transfer_dispatch_to_decoder(
                entry.request_id, entry.transfer_state, idx
            )
```
**EN:** This block defines method `_drain_decoder_tta` on `DiffusionServer`. It handles drain decoder tta logic. Key calls include `self._dispatcher.select_decoder_with_capacity`, `self._decoder_tta.popleft`, and `self._transfer_dispatch_to_decoder`. The implementation branches on conditions, iterates over collections or steps.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_drain_decoder_tta`。 它用于处理 drain decoder tta 相关逻辑。 关键调用包括 `self._dispatcher.select_decoder_with_capacity`、`self._decoder_tta.popleft` 和 `self._transfer_dispatch_to_decoder`。 实现中包含条件分支，会遍历集合或步骤。

### Lines 516-521: `_extract_request_id` implementation / `_extract_request_id` 实现
```python
    def _extract_request_id(self, frames: list) -> str | None:
        try:
            metadata = json.loads(frames[0])
            return metadata.get("scalar_fields", {}).get("request_id")
        except (json.JSONDecodeError, IndexError, TypeError):
            return None
```
**EN:** This block defines method `_extract_request_id` on `DiffusionServer`. It handles extract request id logic. Key calls include `json.loads`, `metadata.get.get`, and `metadata.get`. The implementation handles exceptional paths. Parameters such as `frames` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_extract_request_id`。 它用于处理 extract request id 相关逻辑。 关键调用包括 `json.loads`、`metadata.get.get` 和 `metadata.get`。 实现中处理异常路径。 本段逻辑主要由 `frames` 等参数驱动。

### Lines 523-554: `_complete_with_error` implementation / `_complete_with_error` 实现
```python
    def _complete_with_error(self, request_id: str, error_msg: str) -> None:
        from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import (
            OutputBatch,
        )

        logger.error("DiffusionServer: %s — %s", request_id, error_msg)

        try:
            self._tracker.transition(request_id, RequestState.FAILED, error=error_msg)
        except ValueError:
            pass

        with self._lock:
            client_identity = self._pending.pop(request_id, None)

        if client_identity is None:
            self._tracker.remove(request_id)
            return

        error_batch = OutputBatch(error=error_msg)
        try:
            self._frontend.send_multipart(
                [client_identity, b"", pickle.dumps(error_batch)]
            )
        except zmq.ZMQError as e:
            logger.error(
                "DiffusionServer: failed to send error for %s: %s",
                request_id,
                e,
            )

        self._tracker.remove(request_id)
```
**EN:** This block defines method `_complete_with_error` on `DiffusionServer`. It handles complete with error logic. Key calls include `logger.error`, `OutputBatch`, `self._tracker.remove`, `self._tracker.transition`, and `self._pending.pop`. The implementation branches on conditions, handles exceptional paths, uses context-managed resources. Parameters such as `request_id`, and `error_msg` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_complete_with_error`。 它用于处理 complete with error 相关逻辑。 关键调用包括 `logger.error`、`OutputBatch`、`self._tracker.remove`、`self._tracker.transition` 和 `self._pending.pop`。 实现中包含条件分支，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `request_id` 和 `error_msg` 等参数驱动。

### Lines 556-580: `_handle_timeouts` implementation / `_handle_timeouts` 实现
```python
    def _handle_timeouts(self) -> None:
        timed_out = self._tracker.find_timed_out(self._timeout_s)
        for request_id in timed_out:
            # Free the slot for the timed-out request
            record = self._tracker.get(request_id)
            if record:
                self._free_slot_for_record(record)

            self._complete_with_error(
                request_id,
                f"DiffusionServer timeout: request {request_id} "
                f"not completed within {self._timeout_s}s",
            )

        if timed_out:
            timed_set = set(timed_out)
            self._encoder_tta = deque(
                e for e in self._encoder_tta if e.request_id not in timed_set
            )
            self._denoiser_tta = deque(
                e for e in self._denoiser_tta if e.request_id not in timed_set
            )
            self._decoder_tta = deque(
                e for e in self._decoder_tta if e.request_id not in timed_set
            )
```
**EN:** This block defines method `_handle_timeouts` on `DiffusionServer`. It handles handle timeouts logic. Key calls include `self._tracker.find_timed_out`, `self._tracker.get`, `self._complete_with_error`, `set`, and `deque`. The implementation branches on conditions, iterates over collections or steps.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_handle_timeouts`。 它用于处理 handle timeouts 相关逻辑。 关键调用包括 `self._tracker.find_timed_out`、`self._tracker.get`、`self._complete_with_error`、`set` 和 `deque`。 实现中包含条件分支，会遍历集合或步骤。

### Lines 582-598: `_free_slot_for_record` implementation / `_free_slot_for_record` 实现
```python
    def _free_slot_for_record(self, record) -> None:
        if (
            record.state in (RequestState.ENCODER_RUNNING, RequestState.ENCODER_DONE)
            and record.encoder_instance is not None
        ):
            self._encoder_free_slots[record.encoder_instance] += 1
        if (
            record.state
            in (RequestState.DENOISING_RUNNING, RequestState.DENOISING_DONE)
            and record.denoiser_instance is not None
        ):
            self._denoiser_free_slots[record.denoiser_instance] += 1
        if (
            record.state == RequestState.DECODER_RUNNING
            and record.decoder_instance is not None
        ):
            self._decoder_free_slots[record.decoder_instance] += 1
```
**EN:** This block defines method `_free_slot_for_record` on `DiffusionServer`. It handles free slot for record logic. The implementation branches on conditions. Parameters such as `record` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_free_slot_for_record`。 它用于处理 free slot for record 相关逻辑。 实现中包含条件分支。 本段逻辑主要由 `record` 等参数驱动。

### Lines 600-620: `_handle_transfer_result` implementation / `_handle_transfer_result` 实现
```python
    def _handle_transfer_result(self, frames: list, role: RoleType) -> None:
        try:
            msg = decode_transfer_msg(frames)
        except (ValueError, Exception) as e:
            logger.error("DiffusionServer: failed to decode transfer message: %s", e)
            return

        msg_type = msg.get("msg_type")

        if msg_type == TransferMsgType.REGISTER:
            self._handle_transfer_register(msg)
        elif msg_type == TransferMsgType.STAGED:
            self._handle_transfer_staged(msg)
        elif msg_type == TransferMsgType.ALLOCATED:
            self._handle_transfer_allocated(msg)
        elif msg_type == TransferMsgType.PUSHED:
            self._handle_transfer_pushed(msg)
        elif msg_type == TransferMsgType.DONE:
            self._handle_transfer_done(msg, role)
        else:
            logger.warning("DiffusionServer: unknown transfer msg_type=%s", msg_type)
```
**EN:** This block defines method `_handle_transfer_result` on `DiffusionServer`. It handles handle transfer result logic. Key calls include `msg.get`, `decode_transfer_msg`, `self._handle_transfer_register`, `logger.error`, and `self._handle_transfer_staged`. The implementation branches on conditions, handles exceptional paths. Parameters such as `frames`, and `role` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_handle_transfer_result`。 它用于处理 handle transfer result 相关逻辑。 关键调用包括 `msg.get`、`decode_transfer_msg`、`self._handle_transfer_register`、`logger.error` 和 `self._handle_transfer_staged`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `frames` 和 `role` 等参数驱动。

### Lines 622-680: `_handle_transfer_register` implementation / `_handle_transfer_register` 实现
```python
    def _handle_transfer_register(self, msg: dict) -> None:
        try:
            role = RoleType.from_string(msg.get("role", ""))
        except ValueError:
            logger.warning(
                "DiffusionServer transfer: unknown role in register: %s",
                msg.get("role"),
            )
            return

        work_endpoint = msg.get("work_endpoint", "")
        if role == RoleType.ENCODER:
            endpoint_to_idx = self._encoder_endpoint_to_idx
            peers = self._encoder_peers
        elif role == RoleType.DENOISER:
            endpoint_to_idx = self._denoiser_endpoint_to_idx
            peers = self._denoiser_peers
        elif role == RoleType.DECODER:
            endpoint_to_idx = self._decoder_endpoint_to_idx
            peers = self._decoder_peers
        else:
            logger.warning(
                "DiffusionServer transfer: unsupported role in register: %s", role
            )
            return

        idx = endpoint_to_idx.get(work_endpoint)
        if idx is None:
            # Fail loudly: without a URL match, the control plane (work PUSH)
            # and data plane (RDMA dest) would drift silently.
            logger.error(
                "DiffusionServer transfer: register for role=%s with unknown "
                "work_endpoint=%r (known=%s); dropping registration",
                role.value,
                work_endpoint,
                list(endpoint_to_idx.keys()),
            )
            return

        info = {
            "session_id": msg.get("session_id", ""),
            "pool_ptr": msg.get("pool_ptr", 0),
            "pool_size": msg.get("pool_size", 0),
            "work_endpoint": work_endpoint,
        }
        prealloc = msg.get("preallocated_slots", [])
        info["free_preallocated_slots"] = list(prealloc)
        peers[idx] = info

        logger.info(
            "DiffusionServer transfer: registered %s[%d] work_endpoint=%s "
            "session=%s pool_ptr=%#x prealloc=%d",
            role,
            idx,
            work_endpoint,
            info["session_id"],
            info["pool_ptr"],
            len(prealloc),
        )
```
**EN:** This block defines method `_handle_transfer_register` on `DiffusionServer`. It handles handle transfer register logic. Key calls include `msg.get`, `endpoint_to_idx.get`, `list`, `logger.info`, and `RoleType.from_string`. The implementation branches on conditions, handles exceptional paths. Parameters such as `msg` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_handle_transfer_register`。 它用于处理 handle transfer register 相关逻辑。 关键调用包括 `msg.get`、`endpoint_to_idx.get`、`list`、`logger.info` 和 `RoleType.from_string`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `msg` 等参数驱动。

### Lines 682-711: `_handle_transfer_staged` implementation / `_handle_transfer_staged` 实现
```python
    def _handle_transfer_staged(self, msg: dict) -> None:
        request_id = msg["request_id"]
        logger.debug("DiffusionServer transfer: encoder staged %s", request_id)
        record = self._tracker.get(request_id)
        encoder_idx = record.encoder_instance if record else 0

        p2p = _TransferRequestState(
            sender_session_id=msg.get("session_id", ""),
            sender_pool_ptr=msg.get("pool_ptr", 0),
            sender_slot_offset=msg.get("slot_offset", 0),
            data_size=msg.get("data_size", 0),
            manifest=msg.get("manifest", {}),
            scalar_fields=msg.get("scalar_fields", {}),
            sender_instance=encoder_idx,
        )
        self._transfer_state[request_id] = p2p

        # Encoder slot freed later in _handle_transfer_pushed after RDMA completes
        try:
            self._tracker.transition(request_id, RequestState.ENCODER_DONE)
        except ValueError:
            pass

        try:
            self._tracker.transition(request_id, RequestState.DENOISING_WAITING)
        except ValueError:
            pass
        self._denoiser_tta.append(
            _RoleTTAEntry(request_id=request_id, transfer_state=p2p)
        )
```
**EN:** This block defines method `_handle_transfer_staged` on `DiffusionServer`. It handles handle transfer staged logic. Key calls include `logger.debug`, `self._tracker.get`, `_TransferRequestState`, `self._denoiser_tta.append`, and `self._tracker.transition`. The implementation handles exceptional paths. Parameters such as `msg` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_handle_transfer_staged`。 它用于处理 handle transfer staged 相关逻辑。 关键调用包括 `logger.debug`、`self._tracker.get`、`_TransferRequestState`、`self._denoiser_tta.append` 和 `self._tracker.transition`。 实现中处理异常路径。 本段逻辑主要由 `msg` 等参数驱动。

### Lines 713-756: `_try_fast_path_push` implementation / `_try_fast_path_push` 实现
```python
    def _try_fast_path_push(
        self,
        request_id: str,
        p2p: _TransferRequestState,
        receiver_peer_info: dict,
        sender_pushes: list,
        receiver_role_label: str,
        receiver_idx: int,
    ) -> bool:
        """Try to dispatch via a pre-allocated receive slot (fast path).

        If the receiver already registered a free prealloc slot large enough
        for this transfer, claim it and send a ``TransferPushMsg`` directly
        to the sender so RDMA can start immediately. Returns True when the
        fast path is used; False when the caller must fall back to the
        round-trip alloc path.
        """
        free_slots = receiver_peer_info.get("free_preallocated_slots", [])
        if not (free_slots and free_slots[0].get("size", 0) >= p2p.data_size):
            return False

        slot_info = free_slots.pop(0)
        p2p.receiver_session_id = receiver_peer_info.get("session_id", "")
        p2p.receiver_pool_ptr = receiver_peer_info.get("pool_ptr", 0)
        p2p.receiver_slot_offset = slot_info["offset"]
        p2p.prealloc_slot_id = slot_info.get("slot_id")

        push_msg = TransferPushMsg(
            request_id=request_id,
            dest_session_id=p2p.receiver_session_id,
            dest_addr=slot_info["addr"],
            transfer_size=p2p.data_size,
        )
        sender_pushes[p2p.sender_instance].send_multipart(encode_transfer_msg(push_msg))
        logger.debug(
            "DiffusionServer transfer: fast-path push to %s[%d] for %s "
            "(prealloc slot %s, %d bytes)",
            receiver_role_label,
            receiver_idx,
            request_id,
            slot_info.get("slot_id"),
            p2p.data_size,
        )
        return True
```
**EN:** This block defines method `_try_fast_path_push` on `DiffusionServer`. Try to dispatch via a pre-allocated receive slot (fast path). If the receiver already registered a free prealloc slot large enough for this transfer, claim it and send a ``TransferPushMsg`` directly to the sender so RDMA can start immediately. Key calls include `receiver_peer_info.get`, `free_slots.pop`, `slot_info.get`, `TransferPushMsg`, and `sender_pushes.send_multipart`. The implementation branches on conditions. Parameters such as `request_id`, `p2p`, `receiver_peer_info`, `sender_pushes`, and `receiver_role_label` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_try_fast_path_push`。 它用于处理 try fast path push 相关逻辑。 关键调用包括 `receiver_peer_info.get`、`free_slots.pop`、`slot_info.get`、`TransferPushMsg` 和 `sender_pushes.send_multipart`。 实现中包含条件分支。 本段逻辑主要由 `request_id`、`p2p`、`receiver_peer_info`、`sender_pushes` 和 `receiver_role_label` 等参数驱动。

### Lines 758-777: `_send_slow_path_alloc` implementation / `_send_slow_path_alloc` 实现
```python
    def _send_slow_path_alloc(
        self,
        request_id: str,
        p2p: _TransferRequestState,
        receiver_pushes: list,
        receiver_idx: int,
        source_role: str,
    ) -> None:
        """Ask the receiver to allocate a slot (slow path).

        Used when the receiver has no free prealloc slot large enough. The
        receiver will respond with ``transfer_allocated``; see
        :meth:`_handle_transfer_allocated`.
        """
        alloc_msg = TransferAllocMsg(
            request_id=request_id,
            data_size=p2p.data_size,
            source_role=source_role,
        )
        receiver_pushes[receiver_idx].send_multipart(encode_transfer_msg(alloc_msg))
```
**EN:** This block defines method `_send_slow_path_alloc` on `DiffusionServer`. Ask the receiver to allocate a slot (slow path). Used when the receiver has no free prealloc slot large enough. Key calls include `TransferAllocMsg`, `receiver_pushes.send_multipart`, and `encode_transfer_msg`. Parameters such as `request_id`, `p2p`, `receiver_pushes`, `receiver_idx`, and `source_role` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_send_slow_path_alloc`。 它用于处理 send slow path alloc 相关逻辑。 关键调用包括 `TransferAllocMsg`、`receiver_pushes.send_multipart` 和 `encode_transfer_msg`。 本段逻辑主要由 `request_id`、`p2p`、`receiver_pushes`、`receiver_idx` 和 `source_role` 等参数驱动。

### Lines 779-809: `_transfer_dispatch_to_denoiser` implementation / `_transfer_dispatch_to_denoiser` 实现
```python
    def _transfer_dispatch_to_denoiser(
        self, request_id: str, p2p: _TransferRequestState, denoiser_idx: int
    ) -> None:
        self._denoiser_free_slots[denoiser_idx] -= 1
        p2p.receiver_instance = denoiser_idx

        try:
            self._tracker.transition(
                request_id,
                RequestState.DENOISING_RUNNING,
                denoiser_instance=denoiser_idx,
            )
        except ValueError:
            pass

        peer_info = self._denoiser_peers.get(denoiser_idx, {})
        if not self._try_fast_path_push(
            request_id=request_id,
            p2p=p2p,
            receiver_peer_info=peer_info,
            sender_pushes=self._encoder_pushes,
            receiver_role_label="denoiser",
            receiver_idx=denoiser_idx,
        ):
            self._send_slow_path_alloc(
                request_id=request_id,
                p2p=p2p,
                receiver_pushes=self._denoiser_pushes,
                receiver_idx=denoiser_idx,
                source_role="encoder",
            )
```
**EN:** This block defines method `_transfer_dispatch_to_denoiser` on `DiffusionServer`. It handles transfer dispatch to denoiser logic. Key calls include `self._denoiser_peers.get`, `self._tracker.transition`, `self._try_fast_path_push`, and `self._send_slow_path_alloc`. The implementation branches on conditions, handles exceptional paths. Parameters such as `request_id`, `p2p`, and `denoiser_idx` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_transfer_dispatch_to_denoiser`。 它用于处理 transfer dispatch to denoiser 相关逻辑。 关键调用包括 `self._denoiser_peers.get`、`self._tracker.transition`、`self._try_fast_path_push` 和 `self._send_slow_path_alloc`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `request_id`、`p2p` 和 `denoiser_idx` 等参数驱动。

### Lines 811-844: `_handle_transfer_allocated` implementation / `_handle_transfer_allocated` 实现
```python
    def _handle_transfer_allocated(self, msg: dict) -> None:
        request_id = msg["request_id"]
        p2p = self._transfer_state.get(request_id)
        if p2p is None:
            logger.warning(
                "DiffusionServer transfer: no state for allocated %s", request_id
            )
            return

        p2p.receiver_session_id = msg.get("session_id", "")
        p2p.receiver_pool_ptr = msg.get("pool_ptr", 0)
        p2p.receiver_slot_offset = msg.get("slot_offset", 0)

        dest_addr = p2p.receiver_pool_ptr + p2p.receiver_slot_offset
        push_msg = TransferPushMsg(
            request_id=request_id,
            dest_session_id=p2p.receiver_session_id,
            dest_addr=dest_addr,
            transfer_size=p2p.data_size,
        )

        sender_idx = p2p.sender_instance
        record = self._tracker.get(request_id)
        if record and record.state in (
            RequestState.DECODER_RUNNING,
            RequestState.DECODER_WAITING,
        ):
            self._denoiser_pushes[sender_idx].send_multipart(
                encode_transfer_msg(push_msg)
            )
        else:
            self._encoder_pushes[sender_idx].send_multipart(
                encode_transfer_msg(push_msg)
            )
```
**EN:** This block defines method `_handle_transfer_allocated` on `DiffusionServer`. It handles handle transfer allocated logic. Key calls include `self._transfer_state.get`, `msg.get`, `TransferPushMsg`, `self._tracker.get`, and `logger.warning`. The implementation branches on conditions. Parameters such as `msg` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_handle_transfer_allocated`。 它用于处理 handle transfer allocated 相关逻辑。 关键调用包括 `self._transfer_state.get`、`msg.get`、`TransferPushMsg`、`self._tracker.get` 和 `logger.warning`。 实现中包含条件分支。 本段逻辑主要由 `msg` 等参数驱动。

### Lines 846-903: `_handle_transfer_pushed` implementation / `_handle_transfer_pushed` 实现
```python
    def _handle_transfer_pushed(self, msg: dict) -> None:
        request_id = msg["request_id"]
        logger.debug("DiffusionServer transfer: pushed %s", request_id)
        p2p = self._transfer_state.get(request_id)
        if p2p is None:
            logger.warning(
                "DiffusionServer transfer: no state for pushed %s", request_id
            )
            return

        # Use record state (not sender_idx) to determine sender role,
        # because encoder and denoiser can share the same instance index.
        record = self._tracker.get(request_id)
        if record and record.state in (
            RequestState.DENOISING_RUNNING,
            RequestState.DENOISING_WAITING,
            RequestState.DENOISING_DONE,
        ):
            if record.encoder_instance is not None:
                self._encoder_free_slots[record.encoder_instance] += 1
        elif record and record.state in (
            RequestState.DECODER_RUNNING,
            RequestState.DECODER_WAITING,
        ):
            if record.denoiser_instance is not None:
                self._denoiser_free_slots[record.denoiser_instance] += 1

        scalar_fields = dict(p2p.scalar_fields) if p2p.scalar_fields else {}
        if p2p.prealloc_slot_id is not None:
            scalar_fields["_prealloc_slot_id"] = p2p.prealloc_slot_id
        ready_msg = TransferReadyMsg(
            request_id=request_id,
            manifest=p2p.manifest,
            slot_offset=p2p.receiver_slot_offset,
            scalar_fields=scalar_fields,
        )

        receiver_idx = p2p.receiver_instance
        record = self._tracker.get(request_id)
        if record and record.state in (
            RequestState.DENOISING_RUNNING,
            RequestState.DENOISING_WAITING,
        ):
            self._denoiser_pushes[receiver_idx].send_multipart(
                encode_transfer_msg(ready_msg)
            )
        elif record and record.state in (
            RequestState.DECODER_RUNNING,
            RequestState.DECODER_WAITING,
        ):
            self._decoder_pushes[receiver_idx].send_multipart(
                encode_transfer_msg(ready_msg)
            )

        logger.debug(
            "DiffusionServer transfer: notified receiver for %s (data ready)",
            request_id,
        )
```
**EN:** This block defines method `_handle_transfer_pushed` on `DiffusionServer`. It handles handle transfer pushed logic. Key calls include `logger.debug`, `self._transfer_state.get`, `self._tracker.get`, `TransferReadyMsg`, and `logger.warning`. The implementation branches on conditions. Parameters such as `msg` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_handle_transfer_pushed`。 它用于处理 handle transfer pushed 相关逻辑。 关键调用包括 `logger.debug`、`self._transfer_state.get`、`self._tracker.get`、`TransferReadyMsg` 和 `logger.warning`。 实现中包含条件分支。 本段逻辑主要由 `msg` 等参数驱动。

### Lines 905-926: `_recycle_prealloc_slot` implementation / `_recycle_prealloc_slot` 实现
```python
    def _recycle_prealloc_slot(
        self, p2p: _TransferRequestState, role: RoleType
    ) -> None:
        if p2p is None or p2p.prealloc_slot_id is None:
            return
        receiver_idx = p2p.receiver_instance
        if role == RoleType.DENOISER:
            peer_info = self._denoiser_peers.get(receiver_idx, {})
        elif role == RoleType.DECODER:
            peer_info = self._decoder_peers.get(receiver_idx, {})
        else:
            return
        free_list = peer_info.get("free_preallocated_slots", [])
        free_list.append(
            {
                "offset": p2p.receiver_slot_offset,
                "size": p2p.data_size,
                "slot_id": p2p.prealloc_slot_id,
                "addr": p2p.receiver_pool_ptr + p2p.receiver_slot_offset,
            }
        )
        p2p.prealloc_slot_id = None
```
**EN:** This block defines method `_recycle_prealloc_slot` on `DiffusionServer`. It handles recycle prealloc slot logic. Key calls include `peer_info.get`, `free_list.append`, `self._denoiser_peers.get`, and `self._decoder_peers.get`. The implementation branches on conditions. Parameters such as `p2p`, and `role` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_recycle_prealloc_slot`。 它用于处理 recycle prealloc slot 相关逻辑。 关键调用包括 `peer_info.get`、`free_list.append`、`self._denoiser_peers.get` 和 `self._decoder_peers.get`。 实现中包含条件分支。 本段逻辑主要由 `p2p` 和 `role` 等参数驱动。

### Lines 928-994: `_handle_transfer_done` implementation / `_handle_transfer_done` 实现
```python
    def _handle_transfer_done(self, msg: dict, role: RoleType) -> None:
        request_id = msg.get("request_id", "")
        logger.debug(
            "DiffusionServer transfer: done %s role=%s",
            request_id,
            role.value,
        )
        error = msg.get("error")
        p2p = self._transfer_state.get(request_id)

        if role == RoleType.DENOISER:
            record = self._tracker.get(request_id)

            if p2p is not None:
                self._recycle_prealloc_slot(p2p, RoleType.DENOISER)

            if error:
                if record and record.denoiser_instance is not None:
                    self._denoiser_free_slots[record.denoiser_instance] += 1
                self._complete_with_error(request_id, f"Denoiser error: {error}")
                return

            try:
                self._tracker.transition(request_id, RequestState.DENOISING_DONE)
            except ValueError:
                pass

            if p2p is not None and msg.get("staged_for_decoder"):
                # Denoiser slot freed later in _handle_transfer_pushed
                p2p.sender_session_id = msg.get("session_id", "")
                p2p.sender_pool_ptr = msg.get("pool_ptr", 0)
                p2p.sender_slot_offset = msg.get("slot_offset", 0)
                p2p.data_size = msg.get("data_size", 0)
                p2p.manifest = msg.get("manifest", {})
                p2p.scalar_fields = msg.get("scalar_fields", {})
                p2p.sender_instance = record.denoiser_instance if record else 0

                try:
                    self._tracker.transition(request_id, RequestState.DECODER_WAITING)
                except ValueError:
                    pass
                self._decoder_tta.append(
                    _RoleTTAEntry(request_id=request_id, transfer_state=p2p)
                )
            else:
                if record and record.denoiser_instance is not None:
                    self._denoiser_free_slots[record.denoiser_instance] += 1

        elif role == RoleType.DECODER:
            if p2p is not None:
                self._recycle_prealloc_slot(p2p, RoleType.DECODER)

            record = self._tracker.get(request_id)
            if record and record.decoder_instance is not None:
                self._decoder_free_slots[record.decoder_instance] += 1

            if error:
                self._complete_with_error(request_id, f"Decoder error: {error}")
            else:
                try:
                    self._tracker.transition(request_id, RequestState.DONE)
                except ValueError:
                    pass

                self._transfer_return_to_client_from_msg(request_id, msg)

            self._transfer_state.pop(request_id, None)
```
**EN:** This block defines method `_handle_transfer_done` on `DiffusionServer`. It handles handle transfer done logic. Key calls include `msg.get`, `logger.debug`, `self._transfer_state.get`, `self._tracker.get`, and `self._recycle_prealloc_slot`. The implementation branches on conditions, handles exceptional paths. Parameters such as `msg`, and `role` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_handle_transfer_done`。 它用于处理 handle transfer done 相关逻辑。 关键调用包括 `msg.get`、`logger.debug`、`self._transfer_state.get`、`self._tracker.get` 和 `self._recycle_prealloc_slot`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `msg` 和 `role` 等参数驱动。

### Lines 996-1026: `_transfer_dispatch_to_decoder` implementation / `_transfer_dispatch_to_decoder` 实现
```python
    def _transfer_dispatch_to_decoder(
        self, request_id: str, p2p: _TransferRequestState, decoder_idx: int
    ) -> None:
        self._decoder_free_slots[decoder_idx] -= 1
        p2p.receiver_instance = decoder_idx

        try:
            self._tracker.transition(
                request_id,
                RequestState.DECODER_RUNNING,
                decoder_instance=decoder_idx,
            )
        except ValueError:
            pass

        peer_info = self._decoder_peers.get(decoder_idx, {})
        if not self._try_fast_path_push(
            request_id=request_id,
            p2p=p2p,
            receiver_peer_info=peer_info,
            sender_pushes=self._denoiser_pushes,
            receiver_role_label="decoder",
            receiver_idx=decoder_idx,
        ):
            self._send_slow_path_alloc(
                request_id=request_id,
                p2p=p2p,
                receiver_pushes=self._decoder_pushes,
                receiver_idx=decoder_idx,
                source_role="denoiser",
            )
```
**EN:** This block defines method `_transfer_dispatch_to_decoder` on `DiffusionServer`. It handles transfer dispatch to decoder logic. Key calls include `self._decoder_peers.get`, `self._tracker.transition`, `self._try_fast_path_push`, and `self._send_slow_path_alloc`. The implementation branches on conditions, handles exceptional paths. Parameters such as `request_id`, `p2p`, and `decoder_idx` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_transfer_dispatch_to_decoder`。 它用于处理 transfer dispatch to decoder 相关逻辑。 关键调用包括 `self._decoder_peers.get`、`self._tracker.transition`、`self._try_fast_path_push` 和 `self._send_slow_path_alloc`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `request_id`、`p2p` 和 `decoder_idx` 等参数驱动。

### Lines 1028-1052: `_transfer_return_to_client_from_msg` implementation / `_transfer_return_to_client_from_msg` 实现
```python
    def _transfer_return_to_client_from_msg(self, request_id: str, msg: dict) -> None:
        from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import (
            OutputBatch,
        )

        with self._lock:
            client_identity = self._pending.pop(request_id, None)

        if client_identity is None:
            self._tracker.remove(request_id)
            return

        output_batch = OutputBatch(error=msg.get("error"))

        try:
            self._frontend.send_multipart(
                [client_identity, b"", pickle.dumps(output_batch)]
            )
        except zmq.ZMQError as e:
            logger.error(
                "DiffusionServer transfer: failed to send result for %s: %s",
                request_id,
                e,
            )
        self._tracker.remove(request_id)
```
**EN:** This block defines method `_transfer_return_to_client_from_msg` on `DiffusionServer`. It handles transfer return to client from msg logic. Key calls include `OutputBatch`, `self._tracker.remove`, `self._pending.pop`, `self._frontend.send_multipart`, and `msg.get`. The implementation branches on conditions, handles exceptional paths, uses context-managed resources. Parameters such as `request_id`, and `msg` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `_transfer_return_to_client_from_msg`。 它用于处理 transfer return to client from msg 相关逻辑。 关键调用包括 `OutputBatch`、`self._tracker.remove`、`self._pending.pop`、`self._frontend.send_multipart` 和 `msg.get`。 实现中包含条件分支，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `request_id` 和 `msg` 等参数驱动。

### Lines 1054-1076: `get_stats` implementation / `get_stats` 实现
```python
    def get_stats(self) -> dict:
        with self._lock:
            pending_count = len(self._pending)
        return {
            "role": "diffusion_server",
            "transfer_mode": self._transfer_mode,
            "num_encoders": self._num_encoders,
            "num_denoisers": self._num_denoisers,
            "num_decoders": self._num_decoders,
            "pending_requests": pending_count,
            "dispatch_policy": type(self._dispatcher.encoder_policy).__name__,
            "encoder_free_slots": list(self._encoder_free_slots),
            "denoiser_free_slots": list(self._denoiser_free_slots),
            "decoder_free_slots": list(self._decoder_free_slots),
            "encoder_tta_depth": len(self._encoder_tta),
            "denoiser_tta_depth": len(self._denoiser_tta),
            "decoder_tta_depth": len(self._decoder_tta),
            "transfer_active_transfers": len(self._transfer_state),
            "encoder_peers": len(self._encoder_peers),
            "denoiser_peers": len(self._denoiser_peers),
            "decoder_peers": len(self._decoder_peers),
            "tracker": self._tracker.snapshot(),
        }
```
**EN:** This block defines method `get_stats` on `DiffusionServer`. It retrieves stats. Key calls include `len`, `list`, `self._tracker.snapshot`, and `type`. The implementation uses context-managed resources.
**CN:** 该代码块定义了 `DiffusionServer` 的方法 `get_stats`。 它用于获取stats。 关键调用包括 `len`、`list`、`self._tracker.snapshot` 和 `type`。 实现中使用上下文管理资源。

## Key Concepts / 关键概念
- `_EncoderTTAEntry`: Primary class that encapsulates encoder ttaentry behavior. / 核心类，用于封装 encoder ttaentry 相关行为。
- `_TransferRequestState`: Primary class that encapsulates transfer request state behavior. / 核心类，用于封装 transfer request state 相关行为。
- `_RoleTTAEntry`: Primary class that encapsulates role ttaentry behavior. / 核心类，用于封装 role ttaentry 相关行为。
- `DiffusionServer`: Global pipeline orchestrator for N:M:K disaggregated diffusion. / 核心类，用于封装 diffusion server 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `pickle`, `threading`, `time`, `collections`, `dataclasses`
- **Third-party / 第三方依赖**: `zmq`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.disaggregation.dispatch_policy`, `sglang.multimodal_gen.runtime.disaggregation.request_state`, `sglang.multimodal_gen.runtime.disaggregation.roles`, `sglang.multimodal_gen.runtime.disaggregation.transport.codec`, `sglang.multimodal_gen.runtime.disaggregation.transport.protocol`, `sglang.multimodal_gen.runtime.utils.common`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`

- **Total lines / 总行数**: 1076
