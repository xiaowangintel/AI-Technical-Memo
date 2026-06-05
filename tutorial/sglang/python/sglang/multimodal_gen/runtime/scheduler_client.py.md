# scheduler_client.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/scheduler_client.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for scheduler client in the multimodal generation stack. Key symbols include `run_zeromq_broker`, `SchedulerClient`, `AsyncSchedulerClient`. / 该模块包含多模态生成体系中与 scheduler client 相关的运行时支持代码。 关键符号包括 `run_zeromq_broker`, `SchedulerClient`, `AsyncSchedulerClient`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and module setup / 导入与模块初始化
```python
import pickle
import time
from typing import Any

import zmq
import zmq.asyncio

from sglang.multimodal_gen.runtime.ipc_array import materialize_file_refs
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import OutputBatch
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 16-46: Async function `run_zeromq_broker` / 异步函数 `run_zeromq_broker`
```python
async def run_zeromq_broker(server_args: ServerArgs):
    """
    This function runs as a background task in the FastAPI process.
    It listens for TCP requests from offline clients (e.g., DiffGenerator).
    """
    ctx = zmq.asyncio.Context()
    socket = ctx.socket(zmq.REP)
    broker_endpoint = f"tcp://127.0.0.1:{server_args.broker_port}"
    socket.bind(broker_endpoint)
    logger.info(f"ZMQ Broker is listening for offline jobs on {broker_endpoint}")

    while True:
        try:
            # 1. Receive a request from an offline client
# ...
            try:
                await socket.send(pickle.dumps({"status": "error", "message": str(e)}))
            except Exception:
                pass
```
**EN:** This async function drives `run_zeromq_broker` with inputs such as `server_args`. This function runs as a background task in the FastAPI process.
**CN:** 这个异步函数负责 `run_zeromq_broker`，主要处理 `server_args` 等输入。 文档字符串说明：This function runs as a background task in the FastAPI process.

### Lines 49-123: Class `SchedulerClient` / 类 `SchedulerClient`
```python
class SchedulerClient:
    """
    A synchronous, singleton client for communicating with the Scheduler service.
    Designed for use in DiffGenerator, where synchronous usage is preferred
    """

    def __init__(self):
        self.context = None
        self.scheduler_socket = None
        self.server_args = None

    def initialize(self, server_args: ServerArgs):
        if self.context is not None and not self.context.closed:
            logger.warning("SchedulerClient is already initialized. Re-initializing.")
# ...
            self.scheduler_socket = None
        if self.context:
            self.context.term()
            self.context = None
```
**EN:** This class models `SchedulerClient`. A synchronous, singleton client for communicating with the Scheduler service. Important methods include `__init__`, `initialize`, `forward`, `ping`.
**CN:** 该类实现 `SchedulerClient`。 文档字符串指出：A synchronous, singleton client for communicating with the Scheduler service. 其中较重要的方法包括 `__init__`, `initialize`, `forward`, `ping`。

### Lines 126-206: Class `AsyncSchedulerClient` / 类 `AsyncSchedulerClient`
```python
class AsyncSchedulerClient:
    """
    An asynchronous, singleton client for communicating with the Scheduler service.
    Designed for use in asynchronous environments like FastAPI entrypoints.

    To support high concurrency, it creates a new REQ socket for each request
    rather than sharing a single one (which would cause ZMQ state errors).
    """

    def __init__(self):
        self.context = None
        self.server_args = None

    def initialize(self, server_args: ServerArgs):
# ...
        """Closes the socket and terminates the context."""
        if self.context:
            self.context.term()
            self.context = None
```
**EN:** This class models `AsyncSchedulerClient`. An asynchronous, singleton client for communicating with the Scheduler service. Important methods include `__init__`, `initialize`, `forward`, `ping`.
**CN:** 该类实现 `AsyncSchedulerClient`。 文档字符串指出：An asynchronous, singleton client for communicating with the Scheduler service. 其中较重要的方法包括 `__init__`, `initialize`, `forward`, `ping`。

### Lines 207-211: Top-level configuration / 顶层配置
```python


# Singleton instances for easy access
async_scheduler_client = AsyncSchedulerClient()
sync_scheduler_client = SchedulerClient()
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 214-224: Function `_materialize_output_batch_file_refs` / 函数 `_materialize_output_batch_file_refs`
```python
def _materialize_output_batch_file_refs(output_batch: Any) -> None:
    if not isinstance(output_batch, OutputBatch):
        return

    start_time = time.perf_counter()
    output_batch.output = materialize_file_refs(output_batch.output)
    if output_batch.metrics is not None:
        output_batch.metrics.record_stage(
            "SchedulerClient.materialize_file_refs",
            time.perf_counter() - start_time,
        )
```
**EN:** This function drives `_materialize_output_batch_file_refs` with inputs such as `output_batch`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_materialize_output_batch_file_refs`，主要处理 `output_batch` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- FastAPI request handling / FastAPI 请求处理
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Command-line interface / 命令行接口
- Symbol `run_zeromq_broker` anchors the module API / 符号 `run_zeromq_broker` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.ipc_array`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `zmq`, `zmq.asyncio`
- **Stdlib / 标准库**: `pickle`, `time`, `typing`
