# npu_graph_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/graph_runner/npu_graph_runner.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for backend-specific runtime support inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的后端运行时支持支持。

## Line-by-Line Analysis / 逐行分析
### Lines 14-52: Module setup and shared state / 模块设置与共享状态
```python
"""Run the model with npu graph and torch.compile."""

from __future__ import annotations

import logging
import os
import threading
from contextlib import contextmanager
from pathlib import Path
from typing import TYPE_CHECKING, Dict, Optional, Union

import numpy as np
import torch

import sglang
from sglang.srt.configs.model_config import AttentionArch, is_deepseek_nsa
from sglang.srt.distributed.parallel_state import GroupCoordinator
from sglang.srt.environ import envs
from sglang.srt.model_executor.cuda_graph_runner import CudaGraphRunner
from sglang.srt.utils import (
    empty_context,
    get_bool_env_var,
    get_compiler_backend,
    is_npu,
)

is_npu = is_npu()

# ... omitted for brevity ...

if TYPE_CHECKING:
    from sglang.srt.model_executor.model_runner import ModelRunner

from sglang.srt.layers.logits_processor import LogitsProcessorOutput
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, PPProxyTensors
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `logging`, `os`, `threading`, `contextlib`, `pathlib`. It also defines symbols such as `is_npu`, `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `logging`, `os`, `threading`, `contextlib`, `pathlib`。 同时定义了 `is_npu`, `logger` 等符号，供后续逻辑使用。

### Lines 55-71: Function `patch_model_npu` / 函数 `patch_model_npu`
```python
@contextmanager
def patch_model_npu(
    model: torch.nn.Module,
    enable_compile: bool,
    num_tokens: int,
    tp_group: GroupCoordinator,
):
    if enable_compile:
        backend = get_compiler_backend("npugraph_ex")
        yield torch.compile(
            torch.no_grad()(model.forward),
            fullgraph=True,
            dynamic=False,
            backend=backend,
        )
    else:
        yield model.forward
```
**EN:** This function implements `patch_model_npu`. It primarily calls `get_compiler_backend`, `torch.compile`, `torch.no_grad` to complete its work. State updates are written into `backend`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `patch_model_npu`。 它主要通过调用 `get_compiler_backend`, `torch.compile`, `torch.no_grad` 来完成任务。 状态更新主要写入 `backend`。 实现中使用了条件分支。

### Lines 74-75: Class `NPUGraphRunner` declaration / 类 `NPUGraphRunner` 声明
```python
class NPUGraphRunner(CudaGraphRunner):
    """A NPUGraphRunner runs the forward pass of a model with npu graph and torch.compile."""
```
**EN:** This class establishes `NPUGraphRunner` as the main container/coordinator for the surrounding logic. It inherits from `CudaGraphRunner`. Its core interface includes methods such as `__init__`, `_init_arch_map`, `_create_device_graph`, `_capture_graph`, `_get_update_attr_name`, `_get_update_attr_type`.
**CN:** 该类将 `NPUGraphRunner` 定义为周边逻辑的主要封装体或协调者。 它继承自 `CudaGraphRunner`。 其核心接口包括 `__init__`, `_init_arch_map`, `_create_device_graph`, `_capture_graph`, `_get_update_attr_name`, `_get_update_attr_type` 等方法。

### Lines 77-84: Method `NPUGraphRunner.__init__` / 方法 `NPUGraphRunner.__init__`
```python
    def __init__(self, model_runner: ModelRunner):
        sglang.srt.model_executor.cuda_graph_runner.patch_model = patch_model_npu
        super().__init__(model_runner)
        self.update_attr_name = None
        self.update_attr_type = None
        self.model_runner = model_runner
        self._init_arch_map()
        self.use_fia = get_bool_env_var("ASCEND_USE_FIA", "False")
```
**EN:** This method implements `__init__` on `NPUGraphRunner`. It primarily calls `super.__init__`, `self._init_arch_map`, `get_bool_env_var`, `super` to complete its work. State updates are written into `sglang.srt.model_executor.cuda_graph_runner.patch_model`, `self.update_attr_name`, `self.update_attr_type`, `self.model_runner`, `self.use_fia`.
**CN:** 该方法（属于 `NPUGraphRunner`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `self._init_arch_map`, `get_bool_env_var`, `super` 来完成任务。 状态更新主要写入 `sglang.srt.model_executor.cuda_graph_runner.patch_model`, `self.update_attr_name`, `self.update_attr_type`, `self.model_runner`, `self.use_fia`。

### Lines 86-100: Method `NPUGraphRunner._init_arch_map` / 方法 `NPUGraphRunner._init_arch_map`
```python
    def _init_arch_map(self):
        if self.is_dllm:
            self.attr_name: Dict[str, str] = {
                AttentionArch.MLA: "actual_seq_lengths_kv",
                AttentionArch.MHA: "actual_seq_lengths_kv",
            }
        else:
            self.attr_name: Dict[str, str] = {
                AttentionArch.MLA: "actual_seq_lengths_kv",
                AttentionArch.MHA: "context_lens",
            }
        self.attr_type: Dict[str, Union[list, torch.Tensor]] = {
            AttentionArch.MLA: [],
            AttentionArch.MHA: torch.Tensor(),
        }
```
**EN:** This method implements `_init_arch_map` on `NPUGraphRunner`. It primarily calls `torch.Tensor` to complete its work. State updates are written into `self.attr_type`, `self.attr_name`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUGraphRunner`）实现了 `_init_arch_map`。 它主要通过调用 `torch.Tensor` 来完成任务。 状态更新主要写入 `self.attr_type`, `self.attr_name`。 实现中使用了条件分支。

### Lines 102-103: Method `NPUGraphRunner._create_device_graph` / 方法 `NPUGraphRunner._create_device_graph`
```python
    def _create_device_graph(self):
        return torch.npu.NPUGraph()
```
**EN:** This method implements `_create_device_graph` on `NPUGraphRunner`. It primarily calls `torch.npu.NPUGraph` to complete its work.
**CN:** 该方法（属于 `NPUGraphRunner`）实现了 `_create_device_graph`。 它主要通过调用 `torch.npu.NPUGraph` 来完成任务。

### Lines 105-121: Method `NPUGraphRunner._capture_graph` / 方法 `NPUGraphRunner._capture_graph`
```python
    def _capture_graph(self, graph, pool, stream, run_once_fn):
        if self.enable_torch_compile:
            skip_guard_context = torch.compiler.set_stance(skip_guard_eval_unsafe=True)
        else:
            skip_guard_context = empty_context()

        with (
            skip_guard_context,
            torch.npu.graph(
                graph,
                pool=pool,
                stream=stream,
                auto_dispatch_capture=True,
            ),
        ):
            out = run_once_fn()
        return out
```
**EN:** This method implements `_capture_graph` on `NPUGraphRunner`. It primarily calls `torch.compiler.set_stance`, `empty_context`, `torch.npu.graph`, `run_once_fn` to complete its work. State updates are written into `skip_guard_context`, `out`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该方法（属于 `NPUGraphRunner`）实现了 `_capture_graph`。 它主要通过调用 `torch.compiler.set_stance`, `empty_context`, `torch.npu.graph`, `run_once_fn` 来完成任务。 状态更新主要写入 `skip_guard_context`, `out`。 实现中使用了条件分支、上下文管理资源。

### Lines 123-124: Method `NPUGraphRunner._get_update_attr_name` / 方法 `NPUGraphRunner._get_update_attr_name`
```python
    def _get_update_attr_name(self):
        return self.attr_name[AttentionArch.MLA]
```
**EN:** This method implements `_get_update_attr_name` on `NPUGraphRunner`.
**CN:** 该方法（属于 `NPUGraphRunner`）实现了 `_get_update_attr_name`。

### Lines 126-127: Method `NPUGraphRunner._get_update_attr_type` / 方法 `NPUGraphRunner._get_update_attr_type`
```python
    def _get_update_attr_type(self):
        return self.attr_type[AttentionArch.MLA]
```
**EN:** This method implements `_get_update_attr_type` on `NPUGraphRunner`.
**CN:** 该方法（属于 `NPUGraphRunner`）实现了 `_get_update_attr_type`。

### Lines 129-135: Method `NPUGraphRunner._update_inputs` / 方法 `NPUGraphRunner._update_inputs`
```python
    def _update_inputs(self, seq_lens):
        if isinstance(self.update_attr_type, torch.Tensor):
            seq_lens = torch.from_numpy(np.array(seq_lens).astype(np.int32))

        self.graphs[self.bs].update(
            cpu_update_input=[{self.update_attr_name: seq_lens}]
        )
```
**EN:** This method implements `_update_inputs` on `NPUGraphRunner`. It primarily calls `isinstance`, `self.graphs.update`, `torch.from_numpy`, `np.array.astype`, `np.array` to complete its work. State updates are written into `seq_lens`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUGraphRunner`）实现了 `_update_inputs`。 它主要通过调用 `isinstance`, `self.graphs.update`, `torch.from_numpy`, `np.array.astype`, `np.array` 来完成任务。 状态更新主要写入 `seq_lens`。 实现中使用了条件分支。

### Lines 137-138: Method `NPUGraphRunner._cache_loc_dtype` / 方法 `NPUGraphRunner._cache_loc_dtype`
```python
    def _cache_loc_dtype(self):
        return torch.int32
```
**EN:** This method implements `_cache_loc_dtype` on `NPUGraphRunner`.
**CN:** 该方法（属于 `NPUGraphRunner`）实现了 `_cache_loc_dtype`。

### Lines 140-162: Method `NPUGraphRunner._init_profile_context_and_memory_record` / 方法 `NPUGraphRunner._init_profile_context_and_memory_record`
```python
    def _init_profile_context_and_memory_record(self):
        output_dir = os.path.join(
            os.getenv("SGLANG_TORCH_PROFILER_DIR", "/tmp"), "graph_capture_profile"
        )
        if not Path(output_dir).exists():
            Path(output_dir).mkdir(parents=True, exist_ok=True)
        logger.info(
            f"Profiling starts for graph capture for NPU. Traces will be saved to: {output_dir}"
        )
        experimental_config = torch_npu.profiler._ExperimentalConfig(
            export_type=[torch_npu.profiler.ExportType.Text],
            profiler_level=torch_npu.profiler.ProfilerLevel.Level1,
        )
        profile_context = profile(
            activities=[ProfilerActivity.CPU, ProfilerActivity.NPU],
            record_shapes=True,
            profile_memory=True,
            on_trace_ready=torch_npu.profiler.tensorboard_trace_handler(
                output_dir, async_mode=True
            ),
            experimental_config=experimental_config,
        )
        return profile_context
```
**EN:** This method implements `_init_profile_context_and_memory_record` on `NPUGraphRunner`. It primarily calls `os.path.join`, `logger.info`, `torch_npu.profiler._ExperimentalConfig`, `profile`, `os.getenv`, `Path.exists` to complete its work. State updates are written into `output_dir`, `experimental_config`, `profile_context`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUGraphRunner`）实现了 `_init_profile_context_and_memory_record`。 它主要通过调用 `os.path.join`, `logger.info`, `torch_npu.profiler._ExperimentalConfig`, `profile`, `os.getenv`, `Path.exists` 来完成任务。 状态更新主要写入 `output_dir`, `experimental_config`, `profile_context`。 实现中使用了条件分支。

### Lines 164-166: Method `NPUGraphRunner._post_process_after_profile` / 方法 `NPUGraphRunner._post_process_after_profile`
```python
    def _post_process_after_profile(self, prof_context):
        # for NPU, profile data will be saved to disk for further analysis.
        pass
```
**EN:** This method implements `_post_process_after_profile` on `NPUGraphRunner`.
**CN:** 该方法（属于 `NPUGraphRunner`）实现了 `_post_process_after_profile`。

### Lines 168-225: Method `NPUGraphRunner.replay` / 方法 `NPUGraphRunner.replay`
```python
    def replay(
        self,
        forward_batch: ForwardBatch,
        skip_attn_backend_init: bool = False,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ) -> Union[LogitsProcessorOutput, PPProxyTensors]:
        if not skip_attn_backend_init:
            self.replay_prepare(forward_batch, pp_proxy_tensors)
        else:
            # In speculative decoding, these two fields are still needed.
            self.buffers.input_ids[: self.raw_num_token].copy_(forward_batch.input_ids)
            self.buffers.positions[: self.raw_num_token].copy_(forward_batch.positions)
            if (
                envs.SGLANG_ENABLE_OVERLAP_PLAN_STREAM.get()
                and forward_batch.mrope_positions is not None
            ):
                self.buffers.mrope_positions[:, : self.raw_num_token].copy_(
                    forward_batch.mrope_positions
                )

        self.update_attr_name = self._get_update_attr_name()
        self.update_attr_type = self._get_update_attr_type()
        # Replay
        if not is_deepseek_nsa(self.model_runner.model_config.hf_config):
            if forward_batch.forward_mode.is_target_verify():
                seq_lens_cpu = forward_batch.seq_lens.cpu() + self.num_tokens_per_bs
                seq_lens = seq_lens_cpu.tolist() + [0] * (self.bs - self.raw_bs)
            else:
# ... omitted for brevity ...
                    else None
                ),
            )
        else:
            assert isinstance(output, PPProxyTensors)
            return PPProxyTensors({k: v[: self.bs] for k, v in output.tensors.items()})
```
**EN:** This method implements `replay` on `NPUGraphRunner`. It primarily calls `self._get_update_attr_name`, `self._get_update_attr_type`, `isinstance`, `self.replay_prepare`, `self.buffers.input_ids.copy_`, `self.buffers.positions.copy_` to complete its work. State updates are written into `self.update_attr_name`, `self.update_attr_type`, `output`, `thread`, `seq_lens_cpu`, `seq_lens`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUGraphRunner`）实现了 `replay`。 它主要通过调用 `self._get_update_attr_name`, `self._get_update_attr_type`, `isinstance`, `self.replay_prepare`, `self.buffers.input_ids.copy_`, `self.buffers.positions.copy_` 来完成任务。 状态更新主要写入 `self.update_attr_name`, `self.update_attr_type`, `output`, `thread`, `seq_lens_cpu`, `seq_lens`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `NPUGraphRunner`
- **Functions / 函数**: `patch_model_npu`, `__init__`, `_init_arch_map`, `_create_device_graph`, `_capture_graph`, `_get_update_attr_name`, `_get_update_attr_type`, `_update_inputs`
- **Themes / 主题**: `runner`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang`, `sglang.srt.configs.model_config`, `sglang.srt.distributed.parallel_state`, `sglang.srt.environ`, `sglang.srt.model_executor.cuda_graph_runner`, `sglang.srt.utils`, `sglang.srt.layers.logits_processor`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.model_executor.model_runner`
- **External / 外部依赖**: `numpy`, `torch`, `torch_npu`, `torch_npu.profiler`
- **Standard library / 标准库**: `__future__`, `logging`, `os`, `threading`, `contextlib`, `pathlib`, `typing`
