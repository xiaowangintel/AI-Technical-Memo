# engine_info_bootstrap_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/engine_info_bootstrap_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements engine info bootstrap server logic for runtime entrypoints and service adapters. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 引擎 info bootstrap 服务器 相关的逻辑，并服务于 运行时入口与服务适配层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17: Import runtime dependencies / 导入运行时依赖
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

import logging
import threading
from typing import Dict, Optional, Tuple
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 19-21: Import runtime dependencies / 导入运行时依赖
```python
import uvicorn
from fastapi import FastAPI, HTTPException
from fastapi.responses import PlainTextResponse
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 23-23: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 26-35: Register helpers or handlers / 注册辅助逻辑或处理器
```python
class EngineInfoBootstrapServer:
    """Lightweight HTTP server for per-rank model info registration.

    Runs in a daemon thread on node_rank==0. Each ModelRunner registers its
    info via HTTP PUT after model initialization. The Engine
    accesses the collected info directly in-process; external consumers can
    query via HTTP GET.

    Currently supports transfer engine memory registration info.
    """
```
**EN:** This block wires callbacks, registries, or handlers into the surrounding runtime.
**CN:** 该代码块把回调、注册表或处理器接入周边运行时流程。

### Lines 37-72: Initialize EngineInfoBootstrapServer / 初始化 EngineInfoBootstrapServer
```python
    def __init__(self, host: str, port: int):
        self.host = host
        self.port = port

        # Storage: {tp_rank: (session_id, weights_info_dict)}
        self.transfer_engine_info: Dict[int, Tuple] = {}
        self.lock = threading.Lock()

        app = FastAPI()

        @app.get("/health")
        def health():
            return PlainTextResponse("OK")

        @app.put("/register_transfer_engine_info")
        def register_transfer_engine_info(data: dict):
            try:
                tp_rank = data["tp_rank"]
                info = data["transfer_engine_info"]
                session_id = info["session_id"]
                weights_info_dict = info["weights_info_dict"]

                with self.lock:
                    self.transfer_engine_info[tp_rank] = (
                        session_id,
                        weights_info_dict,
                    )

                logger.info(
                    f"Registered transfer engine info for tp_rank={tp_rank}, "
                    f"session_id={session_id}"
                )
                return PlainTextResponse("OK")
            except Exception as e:
                logger.error(f"Failed to register engine info: {e}")
                raise HTTPException(status_code=400, detail=str(e))
```
**EN:** This block implements the initializer `__init__(host, port)` for `EngineInfoBootstrapServer`. It prepares the object state and connects the instance to the surrounding engine info bootstrap server workflow.
**CN:** 该代码块实现 `EngineInfoBootstrapServer` 的初始化方法 `__init__(host, port)`。它负责准备对象状态，并把实例接入 引擎 info bootstrap 服务器 相关的运行流程。

### Lines 73-97: Initialize EngineInfoBootstrapServer (continued) / 初始化 EngineInfoBootstrapServer（续）
```python

        @app.get("/get_transfer_engine_info")
        def get_transfer_engine_info(rank: int):
            if rank < 0:
                raise HTTPException(status_code=400, detail="Invalid rank parameter")

            with self.lock:
                info = self.transfer_engine_info.get(rank)

            if info is None:
                raise HTTPException(
                    status_code=404,
                    detail=f"No transfer engine info for rank {rank}",
                )

            return {"rank": rank, "remote_instance_transfer_engine_info": list(info)}

        config = uvicorn.Config(app, host=host, port=port, log_level="warning")
        self._server = uvicorn.Server(config)
        self._thread = threading.Thread(
            target=self._server.run,
            daemon=True,
        )
        self._thread.start()
        logger.info(f"EngineInfoBootstrapServer started on {host}:{port}")
```
**EN:** This block implements the initializer `__init__(host, port)` for `EngineInfoBootstrapServer`. It prepares the object state and connects the instance to the surrounding engine info bootstrap server workflow.
**CN:** 该代码块实现 `EngineInfoBootstrapServer` 的初始化方法 `__init__(host, port)`。它负责准备对象状态，并把实例接入 引擎 info bootstrap 服务器 相关的运行流程。

### Lines 99-101: Implement close / 实现close
```python
    def close(self):
        self._server.should_exit = True
        self._thread.join(timeout=5)
```
**EN:** This block implements the method `close()` on `EngineInfoBootstrapServer`. It focuses on handling the engine info bootstrap server responsibilities represented by `close`, so the class can advance the engine info bootstrap server workflow in a self-contained way.
**CN:** 该代码块实现 `EngineInfoBootstrapServer` 上的方法 `close()`。它围绕 `close` 所承担的 引擎 info bootstrap 服务器 相关职责展开，使该类能够独立推进相应流程。

### Lines 103-105: Implement get transfer engine info / 实现get transfer 引擎 info
```python
    def get_transfer_engine_info(self, rank: int) -> Optional[Tuple]:
        """Direct in-process access for co-located HTTP server (no HTTP round-trip)."""
        return self.transfer_engine_info.get(rank)
```
**EN:** This block implements the method `get_transfer_engine_info(rank)` on `EngineInfoBootstrapServer`. It focuses on Direct in-process access for co-located HTTP server (no HTTP round-trip)., so the class can advance the engine info bootstrap server workflow in a self-contained way.
**CN:** 该代码块实现 `EngineInfoBootstrapServer` 上的方法 `get_transfer_engine_info(rank)`。它围绕 `get_transfer_engine_info` 所承担的 引擎 info bootstrap 服务器 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: EngineInfoBootstrapServer
- **Domain focus / 领域焦点**: engine info bootstrap server / 引擎 info bootstrap 服务器
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: logging, threading, typing
- **Third-party / 第三方库**: fastapi, fastapi.responses, uvicorn
- **Local Modules / 本地模块**: None / 无
