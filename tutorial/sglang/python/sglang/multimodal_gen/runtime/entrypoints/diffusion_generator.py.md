# diffusion_generator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/entrypoints/diffusion_generator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the service entrypoint layer. It centers on `DiffGenerator`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: DiffGenerator module for sglang-diffusion. This module provides a consolidated interface for generating images/videos using diffusion models. / 该文件属于服务入口层。它围绕 `DiffGenerator` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 4-47: module setup and imports / 模块初始化与导入
```python
"""
DiffGenerator module for sglang-diffusion.

This module provides a consolidated interface for generating images/videos using
diffusion models.
"""

import dataclasses
import multiprocessing as mp
import os
import time
from contextlib import ExitStack
from typing import Any, List, Union

from sglang.multimodal_gen.configs.sample.sampling_params import (
    DataType,
    SamplingParams,
)
from sglang.multimodal_gen.runtime.entrypoints.utils import (
    GenerationResult,
    ListLorasReq,
    MergeLoraWeightsReq,
    SetLoraReq,
    ShutdownReq,
    UnmergeLoraWeightsReq,
    expand_request_outputs,
    format_lora_message,
    prepare_request,
    save_outputs,
)
from sglang.multimodal_gen.runtime.launch_server import launch_server
from sglang.multimodal_gen.runtime.pipelines_core import Req
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import OutputBatch
from sglang.multimodal_gen.runtime.scheduler_client import sync_scheduler_client
from sglang.multimodal_gen.runtime.server_args import PortArgs, ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import (
    GREEN,
    RESET,
    init_logger,
    log_batch_completion,
    log_generation_timer,
)
from sglang.multimodal_gen.runtime.utils.trace_wrapper import trace_req
from sglang.srt.observability.trace import process_tracing_init, trace_set_thread_info
```
**EN:** This block establishes the module context and imports `dataclasses`, `multiprocessing`, `os`, `time`, `contextlib`, and `typing`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`multiprocessing`、`os`、`time`、`contextlib` 和 `typing`。这些依赖为后续实现提供所需符号。

### Lines 49-59: supporting statements / 辅助语句
```python
logger = init_logger(__name__)

# TODO: move to somewhere appropriate
try:
    # Set the start method to 'spawn' to avoid CUDA errors in forked processes.
    # This must be done at the top level of the module, before any CUDA context
    # or other processes are initialized.
    mp.set_start_method("spawn", force=True)
except RuntimeError:
    # The start method can only be set once per program execution.
    pass
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`, and `mp.set_start_method`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 和 `mp.set_start_method` 协同工作。

### Lines 62-69: `DiffGenerator` class overview / `DiffGenerator` 类概览
```python
class DiffGenerator:
    """
    A unified class for generating images/videos using diffusion models.

    This class provides a simple interface for image/video generation with rich
    customization options, similar to popular frameworks like HF Diffusers.
    """
```
**EN:** This block defines class `DiffGenerator`. A unified class for generating images/videos using diffusion models. This class provides a simple interface for image/video generation with rich customization options, similar to popular frameworks like HF Diffusers.
**CN:** 该代码块定义了类 `DiffGenerator`。 它用于封装 diff generator 相关行为。

### Lines 70-85: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        server_args: ServerArgs,
    ):
        """
        Initialize the generator.

        Args:
            server_args: The inference arguments
        """
        self.server_args = server_args
        self.port_args = PortArgs.from_server_args(server_args)

        # The executor is now a client to the Scheduler service
        self.local_scheduler_process: list[mp.Process] | None = None
        self.owns_scheduler_client: bool = False
```
**EN:** This block defines method `__init__` on `DiffGenerator`. Initialize the generator. Args: server_args: The inference arguments Key calls include `PortArgs.from_server_args`. Parameters such as `server_args` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `PortArgs.from_server_args`。 本段逻辑主要由 `server_args` 等参数驱动。

### Lines 87-108: `from_pretrained` implementation / `from_pretrained` 实现
```python
    @classmethod
    def from_pretrained(
        cls,
        local_mode: bool = True,
        **kwargs,
    ) -> "DiffGenerator":
        """
        Create a DiffGenerator from a pretrained model.

        Priority level: Default pipeline config < User's pipeline config < User's kwargs
        """
        # If users also provide some kwargs, it will override the ServerArgs and PipelineConfig.

        if (server_args := kwargs.get("server_args", None)) is not None:
            if isinstance(server_args, ServerArgs):
                pass
            elif isinstance(server_args, dict):
                server_args = ServerArgs.from_kwargs(**server_args)
        else:
            server_args = ServerArgs.from_kwargs(**kwargs)

        return cls.from_server_args(server_args, local_mode=local_mode)
```
**EN:** This block defines method `from_pretrained` on `DiffGenerator`. Create a DiffGenerator from a pretrained model. Priority level: Default pipeline config < User's pipeline config < User's kwargs Key calls include `cls.from_server_args`, `isinstance`, `ServerArgs.from_kwargs`, and `kwargs.get`. The implementation branches on conditions. Parameters such as `local_mode` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `from_pretrained`。 它用于从…构造pretrained。 关键调用包括 `cls.from_server_args`、`isinstance`、`ServerArgs.from_kwargs` 和 `kwargs.get`。 实现中包含条件分支。 本段逻辑主要由 `local_mode` 等参数驱动。

### Lines 110-140: `from_server_args` implementation / `from_server_args` 实现
```python
    @classmethod
    def from_server_args(
        cls, server_args: ServerArgs, local_mode: bool = True
    ) -> "DiffGenerator":
        """
        Create a DiffGenerator with the specified arguments.

        Args:
            server_args: The inference arguments

        Returns:
            The created DiffGenerator
        """
        instance = cls(
            server_args=server_args,
        )
        if server_args.enable_trace:
            process_tracing_init(server_args.otlp_traces_endpoint, "sglang-diffusion")
            trace_set_thread_info("DiffGenerator")

        logger.info(f"Local mode: {local_mode}")
        if local_mode:
            instance.local_scheduler_process = instance._start_local_server_if_needed()
        else:
            # In remote mode, we just need to connect and check.
            sync_scheduler_client.initialize(server_args)
            instance._check_remote_scheduler()

        # In both modes, this DiffGenerator instance is responsible for the client's lifecycle.
        instance.owns_scheduler_client = True
        return instance
```
**EN:** This block defines method `from_server_args` on `DiffGenerator`. Create a DiffGenerator with the specified arguments. Args: server_args: The inference arguments Returns: The created DiffGenerator Key calls include `cls`, `logger.info`, `process_tracing_init`, `trace_set_thread_info`, and `instance._start_local_server_if_needed`. The implementation branches on conditions. Parameters such as `server_args`, and `local_mode` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `from_server_args`。 它用于从…构造server args。 关键调用包括 `cls`、`logger.info`、`process_tracing_init`、`trace_set_thread_info` 和 `instance._start_local_server_if_needed`。 实现中包含条件分支。 本段逻辑主要由 `server_args` 和 `local_mode` 等参数驱动。

### Lines 142-151: `_start_local_server_if_needed` implementation / `_start_local_server_if_needed` 实现
```python
    def _start_local_server_if_needed(
        self,
    ) -> list[mp.Process]:
        """Check if a local server is running; if not, start it and return the process handles."""
        # First, we need a client to test the server. Initialize it temporarily.
        sync_scheduler_client.initialize(self.server_args)

        processes = launch_server(self.server_args, launch_http_server=False)

        return processes
```
**EN:** This block defines method `_start_local_server_if_needed` on `DiffGenerator`. Check if a local server is running; if not, start it and return the process handles. Key calls include `sync_scheduler_client.initialize`, and `launch_server`.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `_start_local_server_if_needed`。 它用于处理 start local server if needed 相关逻辑。 关键调用包括 `sync_scheduler_client.initialize` 和 `launch_server`。

### Lines 153-164: `_check_remote_scheduler` implementation / `_check_remote_scheduler` 实现
```python
    def _check_remote_scheduler(self):
        """Check if the remote scheduler is accessible."""
        if not sync_scheduler_client.ping():
            raise ConnectionError(
                f"Could not connect to remote scheduler at "
                f"{self.server_args.scheduler_endpoint} with `local mode` as False. "
                "Please ensure the server is running."
            )
        logger.info(
            f"Successfully connected to remote scheduler at "
            f"{self.server_args.scheduler_endpoint}."
        )
```
**EN:** This block defines method `_check_remote_scheduler` on `DiffGenerator`. Check if the remote scheduler is accessible. Key calls include `logger.info`, `sync_scheduler_client.ping`, and `ConnectionError`. The implementation branches on conditions.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `_check_remote_scheduler`。 它用于检查remote scheduler。 关键调用包括 `logger.info`、`sync_scheduler_client.ping` 和 `ConnectionError`。 实现中包含条件分支。

### Lines 166-182: `_resolve_image_paths_per_prompt` implementation / `_resolve_image_paths_per_prompt` 实现
```python
    @staticmethod
    def _resolve_image_paths_per_prompt(
        prompts: list[str], image_paths: str | list[str] | None
    ) -> list[str | list[str] | None]:
        if len(prompts) <= 1:
            return [image_paths]

        if not isinstance(image_paths, list) or len(image_paths) <= 1:
            return [image_paths for _ in prompts]

        if len(image_paths) != len(prompts):
            raise ValueError(
                "When using multiple prompts with multiple input images, "
                "provide either one shared image or exactly one image per prompt."
            )

        return [[image_path] for image_path in image_paths]
```
**EN:** This block defines method `_resolve_image_paths_per_prompt` on `DiffGenerator`. It resolves image paths per prompt. Key calls include `len`, `ValueError`, and `isinstance`. The implementation branches on conditions. Parameters such as `prompts`, and `image_paths` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `_resolve_image_paths_per_prompt`。 它用于解析image paths per prompt。 关键调用包括 `len`、`ValueError` 和 `isinstance`。 实现中包含条件分支。 本段逻辑主要由 `prompts` 和 `image_paths` 等参数驱动。

### Lines 184-362: `generate` implementation / `generate` 实现
```python
    def generate(
        self,
        sampling_params_kwargs: dict | None = None,
        external_trace_header: dict[str, str] | None = None,
    ) -> GenerationResult | list[GenerationResult] | None:
        """Generate image(s)/video(s) based on the given prompt(s).

        Returns a single GenerationResult for a single prompt, a list for
        multiple prompts, or None when every request failed.
        """
        # 1. prepare requests
        prompts = self._resolve_prompts(
            sampling_params_kwargs.get("prompt"),
            sampling_params_kwargs.get("prompt_path"),
        )
        user_output_file_name = sampling_params_kwargs.get("output_file_name")

        if len(prompts) > 1 and user_output_file_name is not None:
            raise ValueError(
                "Cannot use multiple prompts with a fixed output_file_name. "
                "Either remove --output-file-name or use a single prompt."
            )

        sampling_params_orig = SamplingParams.from_user_sampling_params_args(
            self.server_args.model_path,
            server_args=self.server_args,
            **sampling_params_kwargs,
        )

        request_groups: list[list[Req]] = []
        image_paths_per_prompt = self._resolve_image_paths_per_prompt(
            prompts, sampling_params_orig.image_path
        )

        for i, p in enumerate(prompts):
            sampling_params = dataclasses.replace(
                sampling_params_orig,
                prompt=p,
                output_file_name=user_output_file_name,
                image_path=image_paths_per_prompt[i],
            )
            sampling_params._set_output_file_name()
            req = prepare_request(
                server_args=self.server_args,
                sampling_params=sampling_params,
                external_trace_header=external_trace_header,
            )
            request_groups.append(
                expand_request_outputs(
                    req,
                    num_prompts=len(prompts),
                    prompt_index=i,
                )
            )

        results: list[GenerationResult] = []
        total_start_time = time.perf_counter()
        global_output_index = 0

        for requests in request_groups:
            try:
                timer_prompt = [req.prompt for req in requests]
                logger.info("Processing %d grouped request(s)", len(requests))
                with ExitStack() as stack:
                    for req in requests:
                        stack.enter_context(trace_req(req.trace_ctx))
                    timer = stack.enter_context(
                        log_generation_timer(logger, timer_prompt)
                    )
                    output_batch = self._send_to_scheduler_and_wait_for_response(
                        requests
                    )
                    if output_batch.error:
                        raise Exception(f"{output_batch.error}")

                    if (
                        output_batch.output is None
                        and output_batch.output_file_paths is None
                    ):
                        logger.error("Received empty output from scheduler")
                        continue

                    if requests[0].save_output and requests[0].return_file_paths_only:
                        output_file_paths = output_batch.output_file_paths or []
                        self._validate_output_count(
                            len(output_file_paths), len(requests)
                        )
                        for idx, path in enumerate(output_file_paths):
                            req = requests[idx]
                            results.append(
                                GenerationResult(
                                    **self._result_common(
                                        req, output_batch, timer.duration, idx
                                    ),
                                    prompt_index=global_output_index + idx,
                                    output_file_path=path,
                                )
                            )
                    elif requests[0].data_type == DataType.MESH:
                        output_file_paths = output_batch.output_file_paths or []
                        self._validate_output_count(
                            len(output_file_paths), len(requests)
                        )
                        for idx, sample in enumerate(output_file_paths):
                            req = requests[idx]
                            results.append(
                                GenerationResult(
                                    **self._result_common(
                                        req, output_batch, timer.duration, idx
                                    ),
                                    prompt_index=global_output_index + idx,
                                    output_file_path=sample,
                                )
                            )
                    else:
                        self._validate_output_count(
                            len(output_batch.output), len(requests)
                        )
                        samples_out: list[Any] = []
                        audios_out: list[Any] = []
                        frames_out: list[Any] = []
                        save_outputs(
                            output_batch.output,
                            requests[0].data_type,
                            requests[0].fps,
                            requests[0].save_output,
                            lambda idx: requests[idx].output_file_path(1, 0),
                            audio=output_batch.audio,
                            audio_sample_rate=output_batch.audio_sample_rate,
                            samples_out=samples_out,
                            audios_out=audios_out,
                            frames_out=frames_out,
                            output_compression=requests[0].output_compression,
                            enable_frame_interpolation=requests[
                                0
                            ].enable_frame_interpolation,
                            frame_interpolation_exp=requests[0].frame_interpolation_exp,
                            frame_interpolation_scale=requests[
                                0
                            ].frame_interpolation_scale,
                            frame_interpolation_model_path=requests[
                                0
                            ].frame_interpolation_model_path,
                            enable_upscaling=requests[0].enable_upscaling,
                            upscaling_model_path=requests[0].upscaling_model_path,
                            upscaling_scale=requests[0].upscaling_scale,
                        )

                        for idx in range(len(samples_out)):
                            req = requests[idx]
                            results.append(
                                GenerationResult(
                                    **self._result_common(
                                        req, output_batch, timer.duration, idx
                                    ),
                                    samples=samples_out[idx],
                                    frames=frames_out[idx],
                                    audio=audios_out[idx],
                                    prompt_index=global_output_index + idx,
                                    output_file_path=req.output_file_path(1, 0),
                                )
                            )
            except Exception as e:
                logger.error("Generation failed: %s", e, exc_info=True)
            finally:
                global_output_index += len(requests)

        total_gen_time = time.perf_counter() - total_start_time
        if self.server_args.batching_max_size > 1:
            log_batch_completion(
                logger,
                len(results),
                total_gen_time,
            )
        self._log_summary(results)

        if not results:
            return None
        return results[0] if len(results) == 1 else results
```
**EN:** This block defines method `generate` on `DiffGenerator`. Generate image(s)/video(s) based on the given prompt(s). Returns a single GenerationResult for a single prompt, a list for multiple prompts, or None when every request failed. Key calls include `self._resolve_prompts`, `sampling_params_kwargs.get`, `SamplingParams.from_user_sampling_params_args`, `self._resolve_image_paths_per_prompt`, and `enumerate`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths, uses context-managed resources. Parameters such as `sampling_params_kwargs`, and `external_trace_header` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `generate`。 它用于生成函数。 关键调用包括 `self._resolve_prompts`、`sampling_params_kwargs.get`、`SamplingParams.from_user_sampling_params_args`、`self._resolve_image_paths_per_prompt` 和 `enumerate`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `sampling_params_kwargs` 和 `external_trace_header` 等参数驱动。

### Lines 364-385: `_resolve_prompts` implementation / `_resolve_prompts` 实现
```python
    def _resolve_prompts(
        self,
        prompt: str | list[str] | None,
        prompt_path: str | None = None,
    ) -> list[str]:
        """Collect prompts from the argument or from a prompt file."""
        path = prompt_path or self.server_args.prompt_file_path
        if path is not None:
            if not os.path.exists(path):
                raise FileNotFoundError(f"Prompt text file not found: {path}")
            with open(path, encoding="utf-8") as f:
                prompts = [line.strip() for line in f if line.strip()]
            if not prompts:
                raise ValueError(f"No prompts found in file: {path}")
            logger.info("Found %d prompts in %s", len(prompts), path)
            return prompts

        if prompt is None:
            return [" "]
        if isinstance(prompt, str):
            return [prompt]
        return list(prompt)
```
**EN:** This block defines method `_resolve_prompts` on `DiffGenerator`. Collect prompts from the argument or from a prompt file. Key calls include `isinstance`, `list`, `logger.info`, `os.path.exists`, and `FileNotFoundError`. The implementation branches on conditions, uses context-managed resources. Parameters such as `prompt`, and `prompt_path` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `_resolve_prompts`。 它用于解析prompts。 关键调用包括 `isinstance`、`list`、`logger.info`、`os.path.exists` 和 `FileNotFoundError`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `prompt` 和 `prompt_path` 等参数驱动。

### Lines 387-402: `_log_summary` implementation / `_log_summary` 实现
```python
    def _log_summary(self, results: list[GenerationResult]) -> None:
        if not results:
            return
        if self.server_args.warmup:
            total_duration_ms = results[0].metrics.get("total_duration_ms", 0)
            logger.info(
                f"Warmed-up request processed in {GREEN}%.2f{RESET} seconds (with warmup excluded)",
                total_duration_ms / 1000.0,
            )

        peak_memories = [r.peak_memory_mb for r in results if r.peak_memory_mb]
        if peak_memories:
            logger.info(
                f"Memory usage - Max peak: {max(peak_memories):.2f} MB, "
                f"Avg peak: {sum(peak_memories) / len(peak_memories):.2f} MB"
            )
```
**EN:** This block defines method `_log_summary` on `DiffGenerator`. It handles log summary logic. Key calls include `results.metrics.get`, `logger.info`, `max`, `sum`, and `len`. The implementation branches on conditions. Parameters such as `results` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `_log_summary`。 它用于处理 log summary 相关逻辑。 关键调用包括 `results.metrics.get`、`logger.info`、`max`、`sum` 和 `len`。 实现中包含条件分支。 本段逻辑主要由 `results` 等参数驱动。

### Lines 404-428: `_result_common` implementation / `_result_common` 实现
```python
    @staticmethod
    def _result_common(
        req: Req,
        output_batch: OutputBatch,
        generation_time: float,
        output_index: int | None = None,
    ) -> dict[str, Any]:
        metrics = output_batch.metrics
        if (
            output_index is not None
            and output_batch.metrics_list is not None
            and output_index < len(output_batch.metrics_list)
        ):
            metrics = output_batch.metrics_list[output_index]
        return dict(
            prompt=req.prompt,
            size=(req.height, req.width, req.num_frames),
            generation_time=generation_time,
            peak_memory_mb=output_batch.peak_memory_mb,
            metrics=metrics.to_dict() if metrics else {},
            trajectory_latents=output_batch.trajectory_latents,
            trajectory_timesteps=output_batch.trajectory_timesteps,
            rollout_trajectory_data=output_batch.rollout_trajectory_data,
            trajectory_decoded=output_batch.trajectory_decoded,
        )
```
**EN:** This block defines method `_result_common` on `DiffGenerator`. It handles result common logic. Key calls include `dict`, `len`, and `metrics.to_dict`. The implementation branches on conditions. Parameters such as `req`, `output_batch`, `generation_time`, and `output_index` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `_result_common`。 它用于处理 result common 相关逻辑。 关键调用包括 `dict`、`len` 和 `metrics.to_dict`。 实现中包含条件分支。 本段逻辑主要由 `req`、`output_batch`、`generation_time` 和 `output_index` 等参数驱动。

### Lines 430-435: `_validate_output_count` implementation / `_validate_output_count` 实现
```python
    @staticmethod
    def _validate_output_count(output_count: int, request_count: int) -> None:
        if output_count != request_count:
            raise RuntimeError(
                f"Expected {request_count} outputs, got {output_count} from scheduler"
            )
```
**EN:** This block defines method `_validate_output_count` on `DiffGenerator`. It validates output count. Key calls include `RuntimeError`. The implementation branches on conditions. Parameters such as `output_count`, and `request_count` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `_validate_output_count`。 它用于校验output count。 关键调用包括 `RuntimeError`。 实现中包含条件分支。 本段逻辑主要由 `output_count` 和 `request_count` 等参数驱动。

### Lines 437-441: `_send_to_scheduler_and_wait_for_response` implementation / `_send_to_scheduler_and_wait_for_response` 实现
```python
    def _send_to_scheduler_and_wait_for_response(self, batch: list[Req]) -> OutputBatch:
        """
        Sends a request to the scheduler and waits for a response.
        """
        return sync_scheduler_client.forward(batch)
```
**EN:** This block defines method `_send_to_scheduler_and_wait_for_response` on `DiffGenerator`. Sends a request to the scheduler and waits for a response. Key calls include `sync_scheduler_client.forward`. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `_send_to_scheduler_and_wait_for_response`。 它用于处理 send to scheduler and wait for response 相关逻辑。 关键调用包括 `sync_scheduler_client.forward`。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 444-451: `_send_lora_request` implementation / `_send_lora_request` 实现
```python
    def _send_lora_request(self, req: Any, success_msg: str, failure_msg: str):
        response = sync_scheduler_client.forward(req)
        if response.error is None:
            logger.info(success_msg)
            return response
        else:
            error_msg = response.error
            raise RuntimeError(f"{failure_msg}: {error_msg}")
```
**EN:** This block defines method `_send_lora_request` on `DiffGenerator`. It handles send lora request logic. Key calls include `sync_scheduler_client.forward`, `logger.info`, and `RuntimeError`. The implementation branches on conditions. Parameters such as `req`, `success_msg`, and `failure_msg` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `_send_lora_request`。 它用于处理 send lora request 相关逻辑。 关键调用包括 `sync_scheduler_client.forward`、`logger.info` 和 `RuntimeError`。 实现中包含条件分支。 本段逻辑主要由 `req`、`success_msg` 和 `failure_msg` 等参数驱动。

### Lines 453-492: `set_lora` implementation / `set_lora` 实现
```python
    def set_lora(
        self,
        lora_nickname: Union[str, List[str]],
        lora_path: Union[str, None, List[Union[str, None]]] = None,
        target: Union[str, List[str]] = "all",
        strength: Union[float, List[float]] = 1.0,
        merge_mode: str | None = None,
    ) -> None:
        """
        Set LoRA adapter(s) for the specified transformer(s).
        Supports both single LoRA (backward compatible) and multiple LoRA adapters.

        Args:
            lora_nickname: The nickname(s) of the adapter(s). Can be a string or a list of strings.
            lora_path: Path(s) to the LoRA adapter(s). Can be a string, None, or a list of strings/None.
            target: Which transformer(s) to apply the LoRA to. Can be a string or a list of strings.
                Valid values:
                - "all": Apply to all transformers (default)
                - "transformer": Apply only to the primary transformer (high noise for Wan2.2)
                - "transformer_2": Apply only to transformer_2 (low noise for Wan2.2)
                - "critic": Apply only to the critic model
            strength: LoRA strength(s) for merge, default 1.0. Can be a float or a list of floats.
            merge_mode: Optional LoRA merge mode: "auto", "merge", or "dynamic".
        """
        req = SetLoraReq(
            lora_nickname=lora_nickname,
            lora_path=lora_path,
            target=target,
            strength=strength,
            merge_mode=merge_mode,
        )
        nickname_str, target_str, strength_str = format_lora_message(
            lora_nickname, target, strength
        )

        self._send_lora_request(
            req,
            f"Successfully set LoRA adapter(s): {nickname_str} (target: {target_str}, strength: {strength_str})",
            "Failed to set LoRA adapter",
        )
```
**EN:** This block defines method `set_lora` on `DiffGenerator`. Set LoRA adapter(s) for the specified transformer(s). Supports both single LoRA (backward compatible) and multiple LoRA adapters. Key calls include `SetLoraReq`, `format_lora_message`, and `self._send_lora_request`. Parameters such as `lora_nickname`, `lora_path`, `target`, `strength`, and `merge_mode` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `set_lora`。 它用于设置lora。 关键调用包括 `SetLoraReq`、`format_lora_message` 和 `self._send_lora_request`。 本段逻辑主要由 `lora_nickname`、`lora_path`、`target`、`strength` 和 `merge_mode` 等参数驱动。

### Lines 494-506: `unmerge_lora_weights` implementation / `unmerge_lora_weights` 实现
```python
    def unmerge_lora_weights(self, target: str = "all") -> None:
        """
        Unmerge LoRA weights from the base model.

        Args:
            target: Which transformer(s) to unmerge.
        """
        req = UnmergeLoraWeightsReq(target=target)
        self._send_lora_request(
            req,
            f"Successfully unmerged LoRA weights (target: {target})",
            "Failed to unmerge LoRA weights",
        )
```
**EN:** This block defines method `unmerge_lora_weights` on `DiffGenerator`. Unmerge LoRA weights from the base model. Args: target: Which transformer(s) to unmerge. Key calls include `UnmergeLoraWeightsReq`, and `self._send_lora_request`. Parameters such as `target` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `unmerge_lora_weights`。 它用于处理 unmerge lora weights 相关逻辑。 关键调用包括 `UnmergeLoraWeightsReq` 和 `self._send_lora_request`。 本段逻辑主要由 `target` 等参数驱动。

### Lines 508-521: `merge_lora_weights` implementation / `merge_lora_weights` 实现
```python
    def merge_lora_weights(self, target: str = "all", strength: float = 1.0) -> None:
        """
        Merge LoRA weights into the base model.

        Args:
            target: Which transformer(s) to merge.
            strength: LoRA strength for merge, default 1.0.
        """
        req = MergeLoraWeightsReq(target=target, strength=strength)
        self._send_lora_request(
            req,
            f"Successfully merged LoRA weights (target: {target}, strength: {strength})",
            "Failed to merge LoRA weights",
        )
```
**EN:** This block defines method `merge_lora_weights` on `DiffGenerator`. Merge LoRA weights into the base model. Args: target: Which transformer(s) to merge. Key calls include `MergeLoraWeightsReq`, and `self._send_lora_request`. Parameters such as `target`, and `strength` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `merge_lora_weights`。 它用于合并lora weights。 关键调用包括 `MergeLoraWeightsReq` 和 `self._send_lora_request`。 本段逻辑主要由 `target` 和 `strength` 等参数驱动。

### Lines 523-531: `list_loras` implementation / `list_loras` 实现
```python
    def list_loras(self) -> dict:
        """List loaded LoRA adapters and current application status per module."""
        output = self._send_lora_request(
            req=ListLorasReq(),
            success_msg="Successfully listed LoRA adapters",
            failure_msg="Failed to list LoRA adapters",
        )
        # _send_lora_request already raises on error, so output.error is always None here
        return output.output or {}
```
**EN:** This block defines method `list_loras` on `DiffGenerator`. List loaded LoRA adapters and current application status per module. Key calls include `self._send_lora_request`, and `ListLorasReq`.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `list_loras`。 它用于处理 list loras 相关逻辑。 关键调用包括 `self._send_lora_request` 和 `ListLorasReq`。

### Lines 533-559: `_ensure_lora_state` implementation / `_ensure_lora_state` 实现
```python
    def _ensure_lora_state(
        self,
        lora_path: str | None,
        lora_nickname: str | None = None,
        merge_lora: bool = True,
    ) -> None:
        """
        Ensure the LoRA state matches the desired configuration.

        Note: This method does not cache client-side state. The server handles
        idempotent operations, so redundant calls are safe but may have minor overhead.
        """
        if lora_path is None:
            # Unmerge all LoRA weights when no lora_path is provided
            self.unmerge_lora_weights()
            return

        lora_nickname = lora_nickname or self.server_args.lora_nickname

        # Set the LoRA adapter (server handles idempotent logic)
        self.set_lora(lora_nickname, lora_path)

        # Merge or unmerge based on the merge_lora flag
        if merge_lora:
            self.merge_lora_weights()
        else:
            self.unmerge_lora_weights()
```
**EN:** This block defines method `_ensure_lora_state` on `DiffGenerator`. Ensure the LoRA state matches the desired configuration. Note: This method does not cache client-side state. Key calls include `self.set_lora`, `self.unmerge_lora_weights`, and `self.merge_lora_weights`. The implementation branches on conditions. Parameters such as `lora_path`, `lora_nickname`, and `merge_lora` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `_ensure_lora_state`。 它用于处理 ensure lora state 相关逻辑。 关键调用包括 `self.set_lora`、`self.unmerge_lora_weights` 和 `self.merge_lora_weights`。 实现中包含条件分支。 本段逻辑主要由 `lora_path`、`lora_nickname` 和 `merge_lora` 等参数驱动。

### Lines 561-580: `generate_with_lora` implementation / `generate_with_lora` 实现
```python
    def generate_with_lora(
        self,
        prompt: str | list[str] | None = None,
        sampling_params: SamplingParams | None = None,
        *,
        lora_path: str | None = None,
        lora_nickname: str | None = None,
        merge_lora: bool = True,
        **kwargs,
    ):
        self._ensure_lora_state(
            lora_path=lora_path, lora_nickname=lora_nickname, merge_lora=merge_lora
        )
        return self.generate(
            sampling_params_kwargs=dict(
                prompt=prompt,
                sampling_params=sampling_params,
                **kwargs,
            )
        )
```
**EN:** This block defines method `generate_with_lora` on `DiffGenerator`. It generates with lora. Key calls include `self._ensure_lora_state`, `self.generate`, and `dict`. Parameters such as `prompt`, and `sampling_params` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `generate_with_lora`。 它用于生成with lora。 关键调用包括 `self._ensure_lora_state`、`self.generate` 和 `dict`。 本段逻辑主要由 `prompt` 和 `sampling_params` 等参数驱动。

### Lines 582-606: `shutdown` implementation / `shutdown` 实现
```python
    def shutdown(self):
        """
        Shutdown the generator.
        If in local mode, it also shuts down the scheduler server.
        """
        # sends the shutdown command to the server
        if self.local_scheduler_process and self.owns_scheduler_client:
            try:
                sync_scheduler_client.forward(ShutdownReq())
            except Exception:
                pass

        if self.local_scheduler_process:
            for process in self.local_scheduler_process:
                process.join(timeout=10)
                if process.is_alive():
                    logger.warning(
                        f"Local worker {process.name} did not terminate gracefully, forcing."
                    )
                    process.terminate()
            self.local_scheduler_process = None

        if self.owns_scheduler_client:
            sync_scheduler_client.close()
            self.owns_scheduler_client = False
```
**EN:** This block defines method `shutdown` on `DiffGenerator`. Shutdown the generator. If in local mode, it also shuts down the scheduler server. Key calls include `sync_scheduler_client.close`, `sync_scheduler_client.forward`, `process.join`, `process.is_alive`, and `ShutdownReq`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `shutdown`。 它用于处理 shutdown 相关逻辑。 关键调用包括 `sync_scheduler_client.close`、`sync_scheduler_client.forward`、`process.join`、`process.is_alive` 和 `ShutdownReq`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。

### Lines 608-609: `__enter__` implementation / `__enter__` 实现
```python
    def __enter__(self):
        return self
```
**EN:** This block defines method `__enter__` on `DiffGenerator`. It opens a managed runtime context.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `__enter__`。 它用于建立受管理的运行时上下文。

### Lines 611-612: `__exit__` implementation / `__exit__` 实现
```python
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.shutdown()
```
**EN:** This block defines method `__exit__` on `DiffGenerator`. It closes the managed runtime context. Key calls include `self.shutdown`. Parameters such as `exc_type`, `exc_val`, and `exc_tb` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `__exit__`。 它用于关闭受管理的运行时上下文。 关键调用包括 `self.shutdown`。 本段逻辑主要由 `exc_type`、`exc_val` 和 `exc_tb` 等参数驱动。

### Lines 614-628: `__del__` implementation / `__del__` 实现
```python
    def __del__(self):
        owns_scheduler_client = bool(getattr(self, "owns_scheduler_client", False))
        local_scheduler_process = getattr(self, "local_scheduler_process", None)
        if owns_scheduler_client:
            logger.warning(
                "Generator was garbage collected without being shut down. "
                "Attempting to shut down the local server and client."
            )
            self.shutdown()
        elif local_scheduler_process:
            logger.warning(
                "Generator was garbage collected without being shut down. "
                "Attempting to shut down the local server."
            )
            self.shutdown()
```
**EN:** This block defines method `__del__` on `DiffGenerator`. It releases held resources during teardown. Key calls include `bool`, `getattr`, `logger.warning`, and `self.shutdown`. The implementation branches on conditions.
**CN:** 该代码块定义了 `DiffGenerator` 的方法 `__del__`。 它用于在销毁阶段释放占用资源。 关键调用包括 `bool`、`getattr`、`logger.warning` 和 `self.shutdown`。 实现中包含条件分支。

## Key Concepts / 关键概念
- `DiffGenerator`: A unified class for generating images/videos using diffusion models. / 核心类，用于封装 diff generator 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `multiprocessing`, `os`, `time`, `contextlib`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.entrypoints.utils`, `sglang.multimodal_gen.runtime.launch_server`, `sglang.multimodal_gen.runtime.pipelines_core`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.scheduler_client`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.utils.trace_wrapper`, `sglang.srt.observability.trace`

- **Total lines / 总行数**: 628
