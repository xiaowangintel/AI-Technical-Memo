# encode_grpc_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/encode_grpc_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file exposes a gRPC server used by encode-side disaggregation services. It converts network requests into the local transfer workflow and response handling. / 该文件暴露了解码/编码解耦场景中编码侧使用的 gRPC 服务端入口，将网络请求转换为本地传输流程与响应处理。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40: Module-level constants and helper logic
```python
"""
gRPC Encoder Server for SGLang EPD (Encode-Prefill-Decode) mode.

This server provides gRPC-based encoding for multimodal inputs.

Usage:
    python -m sglang.launch_server --model-path <model> --encoder-only --grpc-mode
"""

import asyncio
import logging
import multiprocessing as mp
import traceback
from concurrent import futures
from typing import List

import grpc
import zmq
import zmq.asyncio
from grpc_health.v1 import health_pb2, health_pb2_grpc
from grpc_reflection.v1alpha import reflection
from smg_grpc_proto import sglang_encoder_pb2, sglang_encoder_pb2_grpc

from sglang.srt.disaggregation.encode_server import (
    MMEncoder,
    handle_scheduler_receive_url_request,
    launch_encoder,
)
from sglang.srt.managers.schedule_batch import Modality
from sglang.srt.server_args import PortArgs, ServerArgs
from sglang.srt.utils import random_uuid
from sglang.srt.utils.network import NetworkAddress, get_zmq_socket

logger = logging.getLogger(__name__)
SGLangEncoderServicer = sglang_encoder_pb2_grpc.SglangEncoderServicer
add_SGLangEncoderServicer_to_server = (
    sglang_encoder_pb2_grpc.add_SglangEncoderServicer_to_server
)


```
**EN:** This block contains module-level constants, helpers, or documentation for gRPC entrypoint for encode-side transfer service. It prepares shared state that later classes and functions build on. Notable operations include `EPD`, `import`, `getLogger`.
**CN:** 这一段包含与编码侧传输服务的 gRPC 入口相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `EPD`、`import`、`getLogger`。

### Lines 41-41: Class `EncoderHealthServicer` declaration
```python
class EncoderHealthServicer(health_pb2_grpc.HealthServicer):
```
**EN:** This block declares the class `EncoderHealthServicer` and establishes its responsibility inside gRPC entrypoint for encode-side transfer service. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `EncoderHealthServicer`.
**CN:** 这一段声明了类 `EncoderHealthServicer`，并说明它在编码侧传输服务的 gRPC 入口中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `EncoderHealthServicer`。

### Lines 42-49: Supporting state inside `EncoderHealthServicer`
```python
    """
    Standard gRPC health check service for encoder server.
    Implements grpc.health.v1.Health for Kubernetes probes.
    """

    OVERALL_SERVER = ""
    ENCODER_SERVICE = "sglang.grpc.encoder.SglangEncoder"

```
**EN:** This block adds supporting state or helper logic inside `EncoderHealthServicer`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `EncoderHealthServicer` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 50-51: Method `__init__`
```python
    def __init__(self):
        self._serving = False
```
**EN:** This block defines the method `__init__` on `EncoderHealthServicer`. It introduces the parameters, setup steps, and the main entry point for this piece of gRPC entrypoint for encode-side transfer service. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `EncoderHealthServicer`），介绍了参数、初始化步骤，以及这部分编码侧传输服务的 gRPC 入口逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 53-54: Method `set_serving`
```python
    def set_serving(self):
        self._serving = True
```
**EN:** This block defines the method `set_serving` on `EncoderHealthServicer`. It introduces the parameters, setup steps, and the main entry point for this piece of gRPC entrypoint for encode-side transfer service. Definitions introduced here include `set_serving`.
**CN:** 这一段定义了method `set_serving`（属于 `EncoderHealthServicer`），介绍了参数、初始化步骤，以及这部分编码侧传输服务的 gRPC 入口逻辑的主要入口。 此处引入的定义包括 `set_serving`。

### Lines 56-57: Method `set_not_serving`
```python
    def set_not_serving(self):
        self._serving = False
```
**EN:** This block defines the method `set_not_serving` on `EncoderHealthServicer`. It introduces the parameters, setup steps, and the main entry point for this piece of gRPC entrypoint for encode-side transfer service. Definitions introduced here include `set_not_serving`.
**CN:** 这一段定义了method `set_not_serving`（属于 `EncoderHealthServicer`），介绍了参数、初始化步骤，以及这部分编码侧传输服务的 gRPC 入口逻辑的主要入口。 此处引入的定义包括 `set_not_serving`。

### Lines 59-66: Method `Check`
```python
    async def Check(self, request, context) -> health_pb2.HealthCheckResponse:
        if self._serving:
            return health_pb2.HealthCheckResponse(
                status=health_pb2.HealthCheckResponse.SERVING
            )
        return health_pb2.HealthCheckResponse(
            status=health_pb2.HealthCheckResponse.NOT_SERVING
        )
```
**EN:** This block defines the method `Check` on `EncoderHealthServicer`. It introduces the parameters, setup steps, and the main entry point for this piece of gRPC entrypoint for encode-side transfer service. Definitions introduced here include `Check`. Notable operations include `HealthCheckResponse`.
**CN:** 这一段定义了method `Check`（属于 `EncoderHealthServicer`），介绍了参数、初始化步骤，以及这部分编码侧传输服务的 gRPC 入口逻辑的主要入口。 此处引入的定义包括 `Check`。 值得注意的操作包括 `HealthCheckResponse`。

### Lines 68-69: Method `Watch`
```python
    async def Watch(self, request, context):
        yield await self.Check(request, context)
```
**EN:** This block defines the method `Watch` on `EncoderHealthServicer`. It introduces the parameters, setup steps, and the main entry point for this piece of gRPC entrypoint for encode-side transfer service. Definitions introduced here include `Watch`. Notable operations include `Check`.
**CN:** 这一段定义了method `Watch`（属于 `EncoderHealthServicer`），介绍了参数、初始化步骤，以及这部分编码侧传输服务的 gRPC 入口逻辑的主要入口。 此处引入的定义包括 `Watch`。 值得注意的操作包括 `Check`。

### Lines 72-72: Class `SGLangEncoderServer` declaration
```python
class SGLangEncoderServer(SGLangEncoderServicer):
```
**EN:** This block declares the class `SGLangEncoderServer` and establishes its responsibility inside gRPC entrypoint for encode-side transfer service. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `SGLangEncoderServer`.
**CN:** 这一段声明了类 `SGLangEncoderServer`，并说明它在编码侧传输服务的 gRPC 入口中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `SGLangEncoderServer`。

### Lines 73-76: Supporting state inside `SGLangEncoderServer`
```python
    """
    gRPC service implementation for SGLang encoder.
    """

```
**EN:** This block adds supporting state or helper logic inside `SGLangEncoderServer`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `SGLangEncoderServer` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 77-85: Method `__init__`
```python
    def __init__(
        self,
        encoder: MMEncoder,
        send_sockets: List[zmq.Socket],
        server_args: ServerArgs,
    ):
        self.encoder = encoder
        self.send_sockets = send_sockets
        self.server_args = server_args
```
**EN:** This block defines the method `__init__` on `SGLangEncoderServer`. It introduces the parameters, setup steps, and the main entry point for this piece of gRPC entrypoint for encode-side transfer service. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `SGLangEncoderServer`），介绍了参数、初始化步骤，以及这部分编码侧传输服务的 gRPC 入口逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 87-89: Method `Encode` signature and setup
```python
    async def Encode(
        self, request: sglang_encoder_pb2.EncodeRequest, context
    ) -> sglang_encoder_pb2.EncodeResponse:
```
**EN:** This block defines the method `Encode` on `SGLangEncoderServer`. It introduces the parameters, setup steps, and the main entry point for this piece of gRPC entrypoint for encode-side transfer service. Definitions introduced here include `Encode`.
**CN:** 这一段定义了method `Encode`（属于 `SGLangEncoderServer`），介绍了参数、初始化步骤，以及这部分编码侧传输服务的 gRPC 入口逻辑的主要入口。 此处引入的定义包括 `Encode`。

### Lines 90-117: Method `Encode` logic (part 1)
```python
        try:
            request_dict = {
                "mm_items": list(request.mm_items),
                "req_id": request.req_id,
                "num_parts": request.num_parts,
                "part_idx": request.part_idx,
            }
            for socket in self.send_sockets:
                await socket.send_pyobj(request_dict)

            # gRPC encode is image-only; encoder.encode() requires modality
            (
                nbytes,
                embedding_len,
                embedding_dim,
                error_msg,
                error_code,
            ) = await self.encoder.encode(
                mm_items=list(request.mm_items),
                modality=Modality.IMAGE,
                req_id=request.req_id,
                num_parts=request.num_parts,
                part_idx=request.part_idx,
            )
            if error_msg is not None:
                context.set_code(grpc.StatusCode.INTERNAL)
                context.set_details(error_msg)
                return sglang_encoder_pb2.EncodeResponse()
```
**EN:** This block continues `Encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gRPC entrypoint for encode-side transfer service workflow. Notable operations include `send_pyobj`, `encode`, `modality`, `set_code`.
**CN:** 这一段延续了 `Encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务的 gRPC 入口工作流。 值得注意的操作包括 `send_pyobj`、`encode`、`modality`、`set_code`。

### Lines 118-141: Method `Encode` logic (part 2)
```python

            if self.server_args.encoder_transfer_backend == "mooncake":
                return sglang_encoder_pb2.EncodeResponse(
                    embedding_size=nbytes,
                    embedding_len=embedding_len,
                    embedding_dim=embedding_dim,
                )
            elif self.server_args.encoder_transfer_backend == "zmq_to_scheduler":
                embedding_ports = list(request.embedding_port)
                logger.info(f"embedding_port = {embedding_ports}")
                if not embedding_ports:
                    await self.encoder.send_with_url(req_id=request.req_id)
                else:
                    tasks = []
                    for embedding_port in embedding_ports:
                        tasks.append(
                            self.encoder.send(
                                req_id=request.req_id,
                                prefill_host=request.prefill_host,
                                embedding_port=embedding_port,
                            )
                        )
                    await asyncio.gather(*tasks)
                    self.encoder.embedding_to_send.pop(request.req_id, None)
```
**EN:** This block continues `Encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gRPC entrypoint for encode-side transfer service workflow. Notable operations include `EncodeResponse`, `info`, `send_with_url`, `append`.
**CN:** 这一段延续了 `Encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务的 gRPC 入口工作流。 值得注意的操作包括 `EncodeResponse`、`info`、`send_with_url`、`append`。

### Lines 142-162: Method `Encode` logic (part 3)
```python
                return sglang_encoder_pb2.EncodeResponse()
            elif self.server_args.encoder_transfer_backend == "zmq_to_tokenizer":
                embedding_port = (
                    request.embedding_port[0] if request.embedding_port else 0
                )
                await self.encoder.send(
                    req_id=request.req_id,
                    prefill_host=request.prefill_host,
                    embedding_port=embedding_port,
                )
                self.encoder.embedding_to_send.pop(request.req_id, None)
                return sglang_encoder_pb2.EncodeResponse()

            return sglang_encoder_pb2.EncodeResponse()

        except Exception as e:
            logger.error(f"Encode error: {e}")
            traceback.print_exc()
            context.set_code(grpc.StatusCode.INTERNAL)
            context.set_details(str(e))
            return sglang_encoder_pb2.EncodeResponse()
```
**EN:** This block continues `Encode` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gRPC entrypoint for encode-side transfer service workflow. Notable operations include `EncodeResponse`, `send`, `pop`, `error`.
**CN:** 这一段延续了 `Encode` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务的 gRPC 入口工作流。 值得注意的操作包括 `EncodeResponse`、`send`、`pop`、`error`。

### Lines 164-185: Method `Send`
```python
    async def Send(
        self, request: sglang_encoder_pb2.SendRequest, context
    ) -> sglang_encoder_pb2.SendResponse:
        try:
            await self.encoder.send(
                req_id=request.req_id,
                prefill_host=request.prefill_host,
                embedding_port=request.embedding_port,
                session_id=request.session_id if request.session_id else None,
                buffer_address=(
                    request.buffer_address if request.buffer_address else None
                ),
            )
            self.encoder.embedding_to_send.pop(request.req_id, None)
            return sglang_encoder_pb2.SendResponse()

        except Exception as e:
            logger.error(f"Send error: {e}")
            traceback.print_exc()
            context.set_code(grpc.StatusCode.INTERNAL)
            context.set_details(str(e))
            return sglang_encoder_pb2.SendResponse()
```
**EN:** This block defines the method `Send` on `SGLangEncoderServer`. It introduces the parameters, setup steps, and the main entry point for this piece of gRPC entrypoint for encode-side transfer service. Definitions introduced here include `Send`. Notable operations include `send`, `pop`, `SendResponse`, `error`.
**CN:** 这一段定义了method `Send`（属于 `SGLangEncoderServer`），介绍了参数、初始化步骤，以及这部分编码侧传输服务的 gRPC 入口逻辑的主要入口。 此处引入的定义包括 `Send`。 值得注意的操作包括 `send`、`pop`、`SendResponse`、`error`。

### Lines 187-205: Method `SchedulerReceiveUrl`
```python
    async def SchedulerReceiveUrl(
        self, request: sglang_encoder_pb2.SchedulerReceiveUrlRequest, context
    ) -> sglang_encoder_pb2.SchedulerReceiveUrlResponse:
        try:
            await handle_scheduler_receive_url_request(
                {
                    "req_id": request.req_id,
                    "receive_count": request.receive_count,
                    "receive_url": request.receive_url,
                }
            )
            return sglang_encoder_pb2.SchedulerReceiveUrlResponse()

        except Exception as e:
            logger.error(f"SchedulerReceiveUrl error: {e}")
            traceback.print_exc()
            context.set_code(grpc.StatusCode.INTERNAL)
            context.set_details(str(e))
            return sglang_encoder_pb2.SchedulerReceiveUrlResponse()
```
**EN:** This block defines the method `SchedulerReceiveUrl` on `SGLangEncoderServer`. It introduces the parameters, setup steps, and the main entry point for this piece of gRPC entrypoint for encode-side transfer service. Definitions introduced here include `SchedulerReceiveUrl`. Notable operations include `handle_scheduler_receive_url_request`, `SchedulerReceiveUrlResponse`, `error`, `print_exc`.
**CN:** 这一段定义了method `SchedulerReceiveUrl`（属于 `SGLangEncoderServer`），介绍了参数、初始化步骤，以及这部分编码侧传输服务的 gRPC 入口逻辑的主要入口。 此处引入的定义包括 `SchedulerReceiveUrl`。 值得注意的操作包括 `handle_scheduler_receive_url_request`、`SchedulerReceiveUrlResponse`、`error`、`print_exc`。

### Lines 208-210: Async Function `serve_grpc_encoder` signature and setup
```python
async def serve_grpc_encoder(server_args: ServerArgs):
    ctx = mp.get_context("spawn")
    zmq_ctx = zmq.asyncio.Context(10)
```
**EN:** This block defines the async function `serve_grpc_encoder`. It introduces the parameters, setup steps, and the main entry point for this piece of gRPC entrypoint for encode-side transfer service. Definitions introduced here include `serve_grpc_encoder`. Notable operations include `get_context`, `Context`.
**CN:** 这一段定义了async function `serve_grpc_encoder`，介绍了参数、初始化步骤，以及这部分编码侧传输服务的 gRPC 入口逻辑的主要入口。 此处引入的定义包括 `serve_grpc_encoder`。 值得注意的操作包括 `get_context`、`Context`。

### Lines 211-232: Async Function `serve_grpc_encoder` logic (part 1)
```python
    ipc_path_prefix = random_uuid()
    port_args = PortArgs.init_new(server_args)

    if server_args.dist_init_addr:
        na = NetworkAddress.parse(server_args.dist_init_addr)
        dist_init_method = na.to_tcp()
    else:
        dist_init_method = NetworkAddress(
            server_args.host or "127.0.0.1", port_args.nccl_port
        ).to_tcp()

    send_sockets: List[zmq.Socket] = []
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
```
**EN:** This block continues `serve_grpc_encoder` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gRPC entrypoint for encode-side transfer service workflow. Notable operations include `random_uuid`, `init_new`, `parse`, `to_tcp`.
**CN:** 这一段延续了 `serve_grpc_encoder` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务的 gRPC 入口工作流。 值得注意的操作包括 `random_uuid`、`init_new`、`parse`、`to_tcp`。

### Lines 233-252: Async Function `serve_grpc_encoder` logic (part 2)
```python

    encoder = MMEncoder(server_args, dist_init_method=dist_init_method)

    server = grpc.aio.server(
        futures.ThreadPoolExecutor(max_workers=10),
        options=[
            ("grpc.max_send_message_length", 1024 * 1024 * 256),
            ("grpc.max_receive_message_length", 1024 * 1024 * 256),
        ],
    )

    health_servicer = EncoderHealthServicer()
    health_pb2_grpc.add_HealthServicer_to_server(health_servicer, server)

    encoder_servicer = SGLangEncoderServer(
        encoder=encoder,
        send_sockets=send_sockets,
        server_args=server_args,
    )
    add_SGLangEncoderServicer_to_server(encoder_servicer, server)
```
**EN:** This block continues `serve_grpc_encoder` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gRPC entrypoint for encode-side transfer service workflow. Notable operations include `MMEncoder`, `server`, `ThreadPoolExecutor`, `EncoderHealthServicer`.
**CN:** 这一段延续了 `serve_grpc_encoder` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务的 gRPC 入口工作流。 值得注意的操作包括 `MMEncoder`、`server`、`ThreadPoolExecutor`、`EncoderHealthServicer`。

### Lines 253-274: Async Function `serve_grpc_encoder` logic (part 3)
```python

    SERVICE_NAMES = (
        sglang_encoder_pb2.DESCRIPTOR.services_by_name["SglangEncoder"].full_name,
        "grpc.health.v1.Health",
        reflection.SERVICE_NAME,
    )
    reflection.enable_server_reflection(SERVICE_NAMES, server)

    listen_addr = NetworkAddress(server_args.host, server_args.port).to_host_port_str()
    server.add_insecure_port(listen_addr)

    await server.start()
    logger.info(f"gRPC encoder server listening on {listen_addr}")

    health_servicer.set_serving()

    try:
        await server.wait_for_termination()
    except KeyboardInterrupt:
        logger.info("Shutting down gRPC encoder server...")
        health_servicer.set_not_serving()
        await server.stop(grace=5)
```
**EN:** This block continues `serve_grpc_encoder` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding gRPC entrypoint for encode-side transfer service workflow. Notable operations include `enable_server_reflection`, `NetworkAddress`, `to_host_port_str`, `add_insecure_port`.
**CN:** 这一段延续了 `serve_grpc_encoder` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的编码侧传输服务的 gRPC 入口工作流。 值得注意的操作包括 `enable_server_reflection`、`NetworkAddress`、`to_host_port_str`、`add_insecure_port`。

## Key Concepts / 关键概念
- `EncoderHealthServicer`: Class that encapsulates encoder health servicer behavior in this module. / `EncoderHealthServicer`：封装与“encoderhealthservicer”相关行为的类。
- `SGLangEncoderServer`: Class that encapsulates sglang encoder server behavior in this module. / `SGLangEncoderServer`：封装与“sglangencoder服务端”相关行为的类。
- `serve_grpc_encoder`: Function that performs serve grpc encoder for the surrounding workflow. / `serve_grpc_encoder`：在周边工作流中执行“servegRPCencoder”相关任务的函数。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `logging`, `multiprocessing`, `traceback`, `concurrent`, `typing`
- **External packages / 外部依赖**: `grpc`, `zmq`, `grpc_health`, `grpc_reflection`, `smg_grpc_proto`
- **Internal modules / 内部模块**: `sglang.srt.disaggregation.encode_server`, `sglang.srt.managers.schedule_batch`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.srt.utils.network`
