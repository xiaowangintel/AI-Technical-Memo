# launch_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/launch_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the multimodal generation module. It centers on `_find_available_port`, `kill_process_tree`, and `launch_server`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于多模态生成模块。它围绕 `_find_available_port`、`kill_process_tree` 和 `launch_server` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-26: module setup and imports / 模块初始化与导入
```python
import dataclasses
import multiprocessing as mp
import os
import signal
import sys
import threading

import psutil
import uvicorn

from sglang.multimodal_gen.runtime.disaggregation.orchestrator import (
    DiffusionServer,
)
from sglang.multimodal_gen.runtime.disaggregation.roles import RoleType
from sglang.multimodal_gen.runtime.entrypoints.http_server import create_app
from sglang.multimodal_gen.runtime.managers.gpu_worker import run_scheduler_process
from sglang.multimodal_gen.runtime.server_args import (
    ServerArgs,
    prepare_server_args,
    set_global_server_args,
)
from sglang.multimodal_gen.runtime.utils.common import is_port_available
from sglang.multimodal_gen.runtime.utils.logging_utils import configure_logger, logger
from sglang.srt.observability.trace import process_tracing_init, trace_set_thread_info
```
**EN:** This block establishes the module context and imports `dataclasses`, `multiprocessing`, `os`, `signal`, `sys`, and `threading`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`multiprocessing`、`os`、`signal`、`sys` 和 `threading`。这些依赖为后续实现提供所需符号。

### Lines 29-44: `_find_available_port` implementation / `_find_available_port` 实现
```python
def _find_available_port(
    start: int = 10000, avoid: set[int] | None = None, max_attempts: int = 100
) -> int:
    """Find an available port starting from *start*, skipping ports in *avoid*."""
    if avoid is None:
        avoid = set()
    port = max(1024, min(start, 65535))
    for _ in range(max_attempts):
        if port not in avoid and is_port_available(port):
            return port
        port += 1
        if port > 65535:
            port = 1024
    raise RuntimeError(
        f"No available port found after {max_attempts} attempts (start={start})"
    )
```
**EN:** This block defines function `_find_available_port`. Find an available port starting from *start*, skipping ports in *avoid*. Key calls include `max`, `range`, `RuntimeError`, `set`, and `min`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `start`, `avoid`, and `max_attempts` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_find_available_port`。 它用于处理 find available port 相关逻辑。 关键调用包括 `max`、`range`、`RuntimeError`、`set` 和 `min`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `start`、`avoid` 和 `max_attempts` 等参数驱动。

### Lines 47-83: `kill_process_tree` implementation / `kill_process_tree` 实现
```python
def kill_process_tree(parent_pid, include_parent: bool = True, skip_pid: int = None):
    """Kill the process and all its child processes."""
    # Remove sigchld handler to avoid spammy logs.
    if threading.current_thread() is threading.main_thread():
        signal.signal(signal.SIGCHLD, signal.SIG_DFL)

    if parent_pid is None:
        parent_pid = os.getpid()
        include_parent = False

    try:
        itself = psutil.Process(parent_pid)
    except psutil.NoSuchProcess:
        return

    children = itself.children(recursive=True)
    for child in children:
        if child.pid == skip_pid:
            continue
        try:
            child.kill()
        except psutil.NoSuchProcess:
            pass

    if include_parent:
        try:
            if parent_pid == os.getpid():
                itself.kill()
                sys.exit(0)

            itself.kill()

            # Sometime processes cannot be killed with SIGKILL (e.g, PID=1 launched by kubernetes),
            # so we send an additional signal to kill them.
            itself.send_signal(signal.SIGQUIT)
        except psutil.NoSuchProcess:
            pass
```
**EN:** This block defines function `kill_process_tree`. Kill the process and all its child processes. Key calls include `itself.children`, `threading.current_thread`, `threading.main_thread`, `signal.signal`, and `os.getpid`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `parent_pid`, `include_parent`, and `skip_pid` drive the behavior in this section.
**CN:** 该代码块定义了函数 `kill_process_tree`。 它用于终止process tree。 关键调用包括 `itself.children`、`threading.current_thread`、`threading.main_thread`、`signal.signal` 和 `os.getpid`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `parent_pid`、`include_parent` 和 `skip_pid` 等参数驱动。

### Lines 86-210: `launch_server` implementation / `launch_server` 实现
```python
def launch_server(server_args: ServerArgs, launch_http_server: bool = True):
    """
    Args:
        launch_http_server: False for offline local mode
    """
    configure_logger(server_args)

    # Start a new server with multiple worker processes
    logger.info("Starting server...")

    num_gpus = server_args.num_gpus
    processes = []

    # Pipes for master to talk to slaves
    task_pipes_to_slaves_w = []
    task_pipes_to_slaves_r = []
    for _ in range(num_gpus - 1):
        r, w = mp.Pipe(duplex=False)
        task_pipes_to_slaves_r.append(r)
        task_pipes_to_slaves_w.append(w)

    # Pipes for slaves to talk to master
    result_pipes_from_slaves_w = []
    result_pipes_from_slaves_r = []
    for _ in range(num_gpus - 1):
        r, w = mp.Pipe(duplex=False)
        result_pipes_from_slaves_r.append(r)
        result_pipes_from_slaves_w.append(w)

    # Launch all worker processes
    master_port = server_args.master_port
    scheduler_pipe_readers = []
    scheduler_pipe_writers = []

    for i in range(num_gpus):
        reader, writer = mp.Pipe(duplex=False)
        scheduler_pipe_writers.append(writer)
        if i == 0:  # Master worker
            process = mp.Process(
                target=run_scheduler_process,
                args=(
                    i,  # local_rank
                    i,  # rank
                    master_port,
                    server_args,
                    writer,
                    None,  # No task pipe to read from master
                    None,  # No result pipe to write to master
                    task_pipes_to_slaves_w,
                    result_pipes_from_slaves_r,
                ),
                name=f"sglang-diffusionWorker-{i}",
                daemon=True,
            )
        else:  # Slave workers
            process = mp.Process(
                target=run_scheduler_process,
                args=(
                    i,  # local_rank
                    i,  # rank
                    master_port,
                    server_args,
                    writer,
                    None,  # No task pipe to read from master
                    None,  # No result pipe to write to master
                    task_pipes_to_slaves_r[i - 1],
                    result_pipes_from_slaves_w[i - 1],
                ),
                name=f"sglang-diffusionWorker-{i}",
                daemon=True,
            )
        scheduler_pipe_readers.append(reader)
        process.start()
        processes.append(process)

    # Wait for all workers to be ready
    scheduler_infos = []
    for writer in scheduler_pipe_writers:
        writer.close()

    # Close unused pipe ends in parent process
    for p in task_pipes_to_slaves_w:
        p.close()
    for p in task_pipes_to_slaves_r:
        p.close()
    for p in result_pipes_from_slaves_w:
        p.close()
    for p in result_pipes_from_slaves_r:
        p.close()

    for i, reader in enumerate(scheduler_pipe_readers):
        try:
            data = reader.recv()
        except EOFError:
            logger.error(
                f"Rank {i} scheduler is dead. Please check if there are relevant logs."
            )
            processes[i].join()
            logger.error(f"Exit code: {processes[i].exitcode}")
            raise

        if data["status"] != "ready":
            raise RuntimeError(
                "Initialization failed. Please see the error messages above."
            )
        scheduler_infos.append(data)
        reader.close()

    logger.debug("All workers are ready")

    if launch_http_server:
        logger.info("Starting FastAPI server.")
        if server_args.webui:
            logger.info("Launch FastAPI server in another process because of webui.")
            http_server_process = mp.Process(
                target=launch_http_server_only,
                args=(server_args,),
                name=f"sglang-diffusion-webui",
                daemon=True,
            )
            http_server_process.start()
        else:
            launch_http_server_only(server_args)

    return processes
```
**EN:** This block defines function `launch_server`. Args: launch_http_server: False for offline local mode Key calls include `configure_logger`, `logger.info`, `range`, `enumerate`, and `logger.debug`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `server_args`, and `launch_http_server` drive the behavior in this section.
**CN:** 该代码块定义了函数 `launch_server`。 它用于处理 launch server 相关逻辑。 关键调用包括 `configure_logger`、`logger.info`、`range`、`enumerate` 和 `logger.debug`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `server_args` 和 `launch_http_server` 等参数驱动。

### Lines 213-412: `launch_pool_disagg_server` implementation / `launch_pool_disagg_server` 实现
```python
def launch_pool_disagg_server(
    server_args: ServerArgs,
    encoder_gpus: list[list[int]],
    denoiser_gpus: list[list[int]],
    decoder_gpus: list[list[int]],
    launch_http_server: bool = True,
):
    """Launch a pool-based disaggregated server with N:M:K independent role instances.

    DiffusionServer orchestrates the full pipeline, dispatching at every
    role transition (Encoder → Denoiser → Decoder).

    Args:
        server_args: Base server configuration
        encoder_gpus: List of GPU ID lists, one per encoder instance.
            e.g., [[0], [2]] for 2 encoder instances on GPUs 0 and 2.
        denoiser_gpus: List of GPU ID lists, one per denoiser instance.
            e.g., [[1], [3]] for 2 denoiser instances.
        decoder_gpus: List of GPU ID lists, one per decoder instance.
            e.g., [[0], [2]] for 2 decoder instances (can share with encoder).
        launch_http_server: Whether to launch the HTTP server.

    Example:
        launch_pool_disagg_server(server_args,
            encoder_gpus=[[0], [2]],
            denoiser_gpus=[[1], [3]],
            decoder_gpus=[[0], [2]],
        )
    """
    configure_logger(server_args)

    num_encoders = len(encoder_gpus)
    num_denoisers = len(denoiser_gpus)
    num_decoders = len(decoder_gpus)
    logger.info(
        "Starting pool disagg server: %d encoder(s), %d denoiser(s), %d decoder(s)...",
        num_encoders,
        num_denoisers,
        num_decoders,
    )

    host = server_args.host or "127.0.0.1"

    def find_port(start):
        return _find_available_port(start)

    # Allocate endpoints
    port_cursor = server_args.scheduler_port + 3000

    # Per-instance work endpoints (instance binds PULL, DS connects PUSH)
    encoder_work_endpoints = []
    for i in range(num_encoders):
        p = find_port(port_cursor)
        encoder_work_endpoints.append(f"tcp://{host}:{p}")
        port_cursor = p + 1

    denoiser_work_endpoints = []
    for i in range(num_denoisers):
        p = find_port(port_cursor)
        denoiser_work_endpoints.append(f"tcp://{host}:{p}")
        port_cursor = p + 1

    decoder_work_endpoints = []
    for i in range(num_decoders):
        p = find_port(port_cursor)
        decoder_work_endpoints.append(f"tcp://{host}:{p}")
        port_cursor = p + 1

    # Per-role-type result endpoints (DS binds PULL, instances connect PUSH)
    # Use deterministic convention: scheduler_port + {1,2,3}
    base_port = server_args.scheduler_port
    encoder_result_ep = f"tcp://{host}:{base_port + 1}"
    denoiser_result_ep = f"tcp://{host}:{base_port + 2}"
    decoder_result_ep = f"tcp://{host}:{base_port + 3}"

    logger.info(
        "Pool endpoints allocated: %d work + 3 result endpoints",
        num_encoders + num_denoisers + num_decoders,
    )

    # Launch all role instances
    all_processes = []

    role_configs = [
        (RoleType.ENCODER, encoder_gpus, encoder_work_endpoints, encoder_result_ep),
        (
            RoleType.DENOISER,
            denoiser_gpus,
            denoiser_work_endpoints,
            denoiser_result_ep,
        ),
        (RoleType.DECODER, decoder_gpus, decoder_work_endpoints, decoder_result_ep),
    ]

    for role_type, gpu_lists, work_eps, result_ep in role_configs:
        for inst_idx, gpu_ids in enumerate(gpu_lists):
            num_role_gpus = len(gpu_ids)

            # Per-role parallelism: use explicit overrides if set, else None (auto-derive)
            role_par = server_args.get_role_parallelism(role_type)

            role_overrides = {
                "disagg_role": role_type,
                "disagg_mode": True,
                "pool_work_endpoint": work_eps[inst_idx],
                "pool_result_endpoint": result_ep,
                "num_gpus": num_role_gpus,
                "warmup": role_type == RoleType.ENCODER,
                "scheduler_port": find_port(port_cursor),
                "master_port": find_port(port_cursor + 100),
                # Per-role parallelism (None = auto-derive from num_gpus)
                "tp_size": role_par["tp_size"],
                "sp_degree": role_par["sp_degree"],
                "ulysses_degree": role_par["ulysses_degree"],
                "ring_degree": role_par["ring_degree"],
            }
            port_cursor = role_overrides["master_port"] + 100

            base_dict = {
                f.name: getattr(server_args, f.name)
                for f in dataclasses.fields(server_args)
            }
            base_dict.update(role_overrides)
            base_dict.pop("pipeline_config", None)
            role_args = ServerArgs.from_kwargs(**base_dict)

            pool_ctx = mp.get_context("spawn")
            inst_readers = []

            # Spawn all ranks first — NCCL init blocks until all ranks connect
            for rank_idx in range(num_role_gpus):
                reader, writer = pool_ctx.Pipe(duplex=False)
                gpu_id = gpu_ids[rank_idx]

                process = pool_ctx.Process(
                    target=_run_disagg_role_process,
                    args=(gpu_id, rank_idx, rank_idx, role_args, writer, [], []),
                    name=f"sglang-pool-{role_type.value}-{inst_idx}-r{rank_idx}",
                    daemon=True,
                )
                process.start()
                all_processes.append(process)
                inst_readers.append(reader)

            # Wait for all ranks to be ready (after all are spawned)
            for rank_idx, reader in enumerate(inst_readers):
                try:
                    data = reader.recv()
                except EOFError:
                    logger.error(
                        "Pool %s[%d] rank %d is dead.",
                        role_type.value,
                        inst_idx,
                        rank_idx,
                    )
                    raise
                if data.get("status") != "ready":
                    raise RuntimeError(
                        f"Pool {role_type.value}[{inst_idx}] rank {rank_idx} "
                        "failed to initialize."
                    )
                reader.close()

            logger.info(
                "Pool %s[%d] ready on GPU(s) %s (work=%s)",
                role_type.value.upper(),
                inst_idx,
                gpu_ids,
                work_eps[inst_idx],
            )

    logger.info("All pool role instances ready")

    # Start DiffusionServer
    frontend_endpoint = f"tcp://{host}:{server_args.scheduler_port}"

    diffusion_server = DiffusionServer(
        frontend_endpoint=frontend_endpoint,
        encoder_work_endpoints=encoder_work_endpoints,
        denoiser_work_endpoints=denoiser_work_endpoints,
        decoder_work_endpoints=decoder_work_endpoints,
        encoder_result_endpoint=encoder_result_ep,
        denoiser_result_endpoint=denoiser_result_ep,
        decoder_result_endpoint=decoder_result_ep,
        dispatch_policy_name=server_args.disagg_dispatch_policy,
        timeout_s=float(server_args.disagg_timeout),
    )
    diffusion_server.start()

    if not diffusion_server.wait_ready(timeout=30.0):
        raise RuntimeError("DiffusionServer failed to bind sockets within 30 seconds")

    if launch_http_server:
        logger.info(
            "Starting FastAPI server (connected to DiffusionServer at port %d).",
            server_args.scheduler_port,
        )
        launch_http_server_only(server_args)

    return all_processes
```
**EN:** This block defines function `launch_pool_disagg_server`. Launch a pool-based disaggregated server with N:M:K independent role instances. DiffusionServer orchestrates the full pipeline, dispatching at every role transition (Encoder → Denoiser → Decoder). Key calls include `configure_logger`, `len`, `logger.info`, `range`, and `DiffusionServer`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `server_args`, `encoder_gpus`, `denoiser_gpus`, `decoder_gpus`, and `launch_http_server` drive the behavior in this section.
**CN:** 该代码块定义了函数 `launch_pool_disagg_server`。 它用于处理 launch pool disagg server 相关逻辑。 关键调用包括 `configure_logger`、`len`、`logger.info`、`range` 和 `DiffusionServer`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `server_args`、`encoder_gpus`、`denoiser_gpus`、`decoder_gpus` 和 `launch_http_server` 等参数驱动。

### Lines 415-441: `_run_disagg_role_process` implementation / `_run_disagg_role_process` 实现
```python
def _run_disagg_role_process(
    gpu_id: int,
    _local_rank: int,
    rank: int,
    server_args: ServerArgs,
    pipe_writer: mp.connection.Connection,
    task_pipes: list,
    result_pipes: list,
):
    """Entry point for a disagg role process.

    Uses the physical GPU index (gpu_id) as local_rank so that
    torch.cuda.set_device(local_rank) selects the correct GPU.
    This avoids relying on CUDA_VISIBLE_DEVICES remapping, which
    may not work if CUDA was pre-initialized in the parent process.
    """
    run_scheduler_process(
        local_rank=gpu_id,
        rank=rank,
        master_port=server_args.master_port,
        server_args=server_args,
        pipe_writer=pipe_writer,
        task_pipe_r=None,
        result_pipe_w=None,
        task_pipes_to_slaves=task_pipes,
        result_pipes_from_slaves=result_pipes,
    )
```
**EN:** This block defines function `_run_disagg_role_process`. Entry point for a disagg role process. Uses the physical GPU index (gpu_id) as local_rank so that torch.cuda.set_device(local_rank) selects the correct GPU. Key calls include `run_scheduler_process`. Parameters such as `gpu_id`, `_local_rank`, `rank`, `server_args`, and `pipe_writer` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_run_disagg_role_process`。 它用于运行disagg role process。 关键调用包括 `run_scheduler_process`。 本段逻辑主要由 `gpu_id`、`_local_rank`、`rank`、`server_args` 和 `pipe_writer` 等参数驱动。

### Lines 444-459: `launch_http_server_only` implementation / `launch_http_server_only` 实现
```python
def launch_http_server_only(server_args):
    if server_args.enable_trace:
        process_tracing_init(server_args.otlp_traces_endpoint, "sglang-diffusion")
        trace_set_thread_info("DiffHTTPServer")

    # set for endpoints to access global_server_args
    set_global_server_args(server_args)
    app = create_app(server_args)
    uvicorn.run(
        app,
        use_colors=True,
        log_level=server_args.log_level,
        host=server_args.host,
        port=server_args.port,
        reload=False,
    )
```
**EN:** This block defines function `launch_http_server_only`. It handles launch http server only logic. Key calls include `set_global_server_args`, `create_app`, `uvicorn.run`, `process_tracing_init`, and `trace_set_thread_info`. The implementation branches on conditions. Parameters such as `server_args` drive the behavior in this section.
**CN:** 该代码块定义了函数 `launch_http_server_only`。 它用于处理 launch http server only 相关逻辑。 关键调用包括 `set_global_server_args`、`create_app`、`uvicorn.run`、`process_tracing_init` 和 `trace_set_thread_info`。 实现中包含条件分支。 本段逻辑主要由 `server_args` 等参数驱动。

### Lines 462-467: `parse_url_string` implementation / `parse_url_string` 实现
```python
def parse_url_string(url_str: str) -> list[str]:
    """Parse a semicolon-separated URL string into a list.

    Example: "tcp://10.0.0.1:35000;tcp://10.0.0.2:35000" -> ["tcp://...", "tcp://..."]
    """
    return [u.strip() for u in url_str.split(";") if u.strip()]
```
**EN:** This block defines function `parse_url_string`. Parse a semicolon-separated URL string into a list. Example: "tcp://10.0.0.1:35000;tcp://10.0.0.2:35000" -> ["tcp://...", "tcp://..."] Key calls include `u.strip`, and `url_str.split`. Parameters such as `url_str` drive the behavior in this section.
**CN:** 该代码块定义了函数 `parse_url_string`。 它用于解析url string。 关键调用包括 `u.strip` 和 `url_str.split`。 本段逻辑主要由 `url_str` 等参数驱动。

### Lines 470-541: `launch_disagg_server` implementation / `launch_disagg_server` 实现
```python
def launch_disagg_server(server_args: ServerArgs):
    """Launch DiffusionServer head node + HTTP server (--disagg-role server).

    No GPU workers are spawned. Connects to remote role instances
    specified by --encoder-urls, --denoiser-urls, --decoder-urls.

    Result endpoints use deterministic convention:
        encoder result: scheduler_port + 1
        denoiser result: scheduler_port + 2
        decoder result: scheduler_port + 3
    """
    configure_logger(server_args)

    for name, val in [
        ("--encoder-urls", server_args.encoder_urls),
        ("--denoiser-urls", server_args.denoiser_urls),
        ("--decoder-urls", server_args.decoder_urls),
    ]:
        if val is None:
            raise ValueError(f"{name} is required for --disagg-role server")

    host = server_args.host or "127.0.0.1"
    base_port = server_args.scheduler_port

    encoder_work_endpoints = parse_url_string(server_args.encoder_urls)
    denoiser_work_endpoints = parse_url_string(server_args.denoiser_urls)
    decoder_work_endpoints = parse_url_string(server_args.decoder_urls)

    encoder_result_ep = f"tcp://{host}:{base_port + 1}"
    denoiser_result_ep = f"tcp://{host}:{base_port + 2}"
    decoder_result_ep = f"tcp://{host}:{base_port + 3}"

    frontend_endpoint = f"tcp://{host}:{base_port}"

    logger.info(
        "Starting DiffusionServer: %d encoder(s), %d denoiser(s), %d decoder(s)",
        len(encoder_work_endpoints),
        len(denoiser_work_endpoints),
        len(decoder_work_endpoints),
    )
    logger.info("  Frontend: %s", frontend_endpoint)
    logger.info("  Encoder work endpoints: %s", encoder_work_endpoints)
    logger.info("  Denoiser work endpoints: %s", denoiser_work_endpoints)
    logger.info("  Decoder work endpoints: %s", decoder_work_endpoints)
    logger.info(
        "  Result endpoints: encoder=%s, denoiser=%s, decoder=%s",
        encoder_result_ep,
        denoiser_result_ep,
        decoder_result_ep,
    )

    diffusion_server = DiffusionServer(
        frontend_endpoint=frontend_endpoint,
        encoder_work_endpoints=encoder_work_endpoints,
        denoiser_work_endpoints=denoiser_work_endpoints,
        decoder_work_endpoints=decoder_work_endpoints,
        encoder_result_endpoint=encoder_result_ep,
        denoiser_result_endpoint=denoiser_result_ep,
        decoder_result_endpoint=decoder_result_ep,
        dispatch_policy_name=server_args.disagg_dispatch_policy,
        timeout_s=float(server_args.disagg_timeout),
    )
    diffusion_server.start()

    if not diffusion_server.wait_ready(timeout=30.0):
        raise RuntimeError("DiffusionServer failed to bind sockets within 30 seconds")

    logger.info(
        "Starting HTTP server (connected to DiffusionServer at port %d).",
        base_port,
    )
    launch_http_server_only(server_args)
```
**EN:** This block defines function `launch_disagg_server`. Launch DiffusionServer head node + HTTP server (--disagg-role server). No GPU workers are spawned. Key calls include `configure_logger`, `parse_url_string`, `logger.info`, `DiffusionServer`, and `diffusion_server.start`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `server_args` drive the behavior in this section.
**CN:** 该代码块定义了函数 `launch_disagg_server`。 它用于处理 launch disagg server 相关逻辑。 关键调用包括 `configure_logger`、`parse_url_string`、`logger.info`、`DiffusionServer` 和 `diffusion_server.start`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `server_args` 等参数驱动。

### Lines 544-660: `launch_disagg_role` implementation / `launch_disagg_role` 实现
```python
def launch_disagg_role(server_args: ServerArgs):
    """Launch a standalone disaggregated role instance (--disagg-role encoder/denoising/decoder).

    The instance:
    1. Binds its work PULL socket on tcp://0.0.0.0:{scheduler_port}
    2. Connects its result PUSH socket to the DiffusionServer head node
       (derived from --disagg-server-addr + role offset)
    3. Spawns GPU worker processes for the assigned role.
    """
    configure_logger(server_args)

    role_type = server_args.disagg_role
    if server_args.disagg_server_addr is None:
        raise ValueError(
            "--disagg-server-addr is required for --disagg-role " f"{role_type.value}"
        )

    # Derive endpoints
    work_endpoint = server_args.derive_pool_work_endpoint()
    result_endpoint = server_args.derive_pool_result_endpoint()

    logger.info(
        "Starting disagg role: %s, num_gpus=%d",
        role_type.value,
        server_args.num_gpus,
    )
    logger.info("  Work endpoint (bind): %s", work_endpoint)
    logger.info("  Result endpoint (connect): %s", result_endpoint)
    logger.info(
        "  P2P: hostname=%s, ib_device=%s, pool_size=%d",
        server_args.disagg_p2p_hostname,
        server_args.disagg_ib_device,
        server_args.disagg_transfer_pool_size,
    )

    # Build role-specific ServerArgs
    # Use a different port for the scheduler's internal ROUTER socket to avoid
    # conflicting with the pool work PULL socket (both bind on scheduler_port).
    internal_scheduler_port = _find_available_port(
        start=server_args.scheduler_port + 100, avoid={server_args.scheduler_port}
    )

    role_par = server_args.get_role_parallelism(role_type)
    role_overrides = {
        "disagg_role": role_type,
        "disagg_mode": True,
        "pool_work_endpoint": work_endpoint,
        "pool_result_endpoint": result_endpoint,
        "warmup": role_type == RoleType.ENCODER,
        "scheduler_port": internal_scheduler_port,
        # Per-role parallelism (None = auto-derive from num_gpus)
        "tp_size": role_par["tp_size"],
        "sp_degree": role_par["sp_degree"],
        "ulysses_degree": role_par["ulysses_degree"],
        "ring_degree": role_par["ring_degree"],
    }

    base_dict = {
        f.name: getattr(server_args, f.name) for f in dataclasses.fields(server_args)
    }
    base_dict.update(role_overrides)
    base_dict.pop("pipeline_config", None)
    role_args = ServerArgs.from_kwargs(**base_dict)

    # Spawn GPU worker processes
    # NOTE: All ranks must be spawned before waiting for ready signals,
    # because NCCL init_process_group blocks until all ranks connect.
    num_gpus = server_args.num_gpus
    base_gpu_id = server_args.base_gpu_id
    pool_ctx = mp.get_context("spawn")
    processes = []
    readers = []

    for rank_idx in range(num_gpus):
        reader, writer = pool_ctx.Pipe(duplex=False)
        gpu_id = base_gpu_id + rank_idx

        process = pool_ctx.Process(
            target=_run_disagg_role_process,
            args=(gpu_id, rank_idx, rank_idx, role_args, writer, [], []),
            name=f"sglang-{role_type.value}-r{rank_idx}",
            daemon=True,
        )
        process.start()
        processes.append(process)
        readers.append(reader)

    # Wait for all ranks to be ready (after all are spawned)
    for rank_idx, reader in enumerate(readers):
        try:
            data = reader.recv()
        except EOFError:
            logger.error(
                "Role %s rank %d is dead.",
                role_type.value,
                rank_idx,
            )
            raise
        if data.get("status") != "ready":
            raise RuntimeError(
                f"Role {role_type.value} rank {rank_idx} failed to initialize."
            )
        reader.close()

    logger.info(
        "Role %s ready (%d GPU(s), work=%s)",
        role_type.value.upper(),
        num_gpus,
        work_endpoint,
    )

    # Block until interrupted
    try:
        for p in processes:
            p.join()
    except KeyboardInterrupt:
        logger.info("Role %s shutting down.", role_type.value)
```
**EN:** This block defines function `launch_disagg_role`. Launch a standalone disaggregated role instance (--disagg-role encoder/denoising/decoder). The instance: 1. Key calls include `configure_logger`, `server_args.derive_pool_work_endpoint`, `server_args.derive_pool_result_endpoint`, `logger.info`, and `_find_available_port`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `server_args` drive the behavior in this section.
**CN:** 该代码块定义了函数 `launch_disagg_role`。 它用于处理 launch disagg role 相关逻辑。 关键调用包括 `configure_logger`、`server_args.derive_pool_work_endpoint`、`server_args.derive_pool_result_endpoint`、`logger.info` 和 `_find_available_port`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `server_args` 等参数驱动。

### Lines 663-673: `dispatch_launch` implementation / `dispatch_launch` 实现
```python
def dispatch_launch(server_args: ServerArgs):
    """Route to the correct launch function based on --disagg-role."""
    role = server_args.disagg_role
    if role == RoleType.MONOLITHIC:
        launch_server(server_args)
    elif role == RoleType.SERVER:
        launch_disagg_server(server_args)
    elif role in (RoleType.ENCODER, RoleType.DENOISER, RoleType.DECODER):
        launch_disagg_role(server_args)
    else:
        raise ValueError(f"Unknown disagg_role: {role}")
```
**EN:** This block defines function `dispatch_launch`. Route to the correct launch function based on --disagg-role. Key calls include `launch_server`, `launch_disagg_server`, `launch_disagg_role`, and `ValueError`. The implementation branches on conditions. Parameters such as `server_args` drive the behavior in this section.
**CN:** 该代码块定义了函数 `dispatch_launch`。 它用于处理 dispatch launch 相关逻辑。 关键调用包括 `launch_server`、`launch_disagg_server`、`launch_disagg_role` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `server_args` 等参数驱动。

### Lines 676-682: supporting statements / 辅助语句
```python
if __name__ == "__main__":
    server_args = prepare_server_args(sys.argv[1:])

    try:
        dispatch_launch(server_args)
    finally:
        kill_process_tree(os.getpid(), include_parent=False)
```
**EN:** This block gathers supporting statements at module scope. The code collaborates with `prepare_server_args`, `dispatch_launch`, `kill_process_tree`, and `os.getpid`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 代码会与 `prepare_server_args`、`dispatch_launch`、`kill_process_tree` 和 `os.getpid` 协同工作。

## Key Concepts / 关键概念
- `_find_available_port`: Find an available port starting from *start*, skipping ports in *avoid*. / 顶层函数，用于处理 find available port 相关逻辑。
- `kill_process_tree`: Kill the process and all its child processes. / 顶层函数，用于终止process tree。
- `launch_server`: Args: launch_http_server: False for offline local mode / 顶层函数，用于处理 launch server 相关逻辑。
- `launch_pool_disagg_server`: Launch a pool-based disaggregated server with N:M:K independent role instances. / 顶层函数，用于处理 launch pool disagg server 相关逻辑。
- `_run_disagg_role_process`: Entry point for a disagg role process. / 顶层函数，用于运行disagg role process。
- `launch_http_server_only`: Top-level function that handles launch http server only logic. / 顶层函数，用于处理 launch http server only 相关逻辑。
- `parse_url_string`: Parse a semicolon-separated URL string into a list. / 顶层函数，用于解析url string。
- `launch_disagg_server`: Launch DiffusionServer head node + HTTP server (--disagg-role server). / 顶层函数，用于处理 launch disagg server 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `multiprocessing`, `os`, `signal`, `sys`, `threading`
- **Third-party / 第三方依赖**: `psutil`, `uvicorn`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.disaggregation.orchestrator`, `sglang.multimodal_gen.runtime.disaggregation.roles`, `sglang.multimodal_gen.runtime.entrypoints.http_server`, `sglang.multimodal_gen.runtime.managers.gpu_worker`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.common`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.srt.observability.trace`

- **Total lines / 总行数**: 682
