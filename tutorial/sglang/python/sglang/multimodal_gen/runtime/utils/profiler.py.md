# profiler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/utils/profiler.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for profiler in the multimodal generation stack. Key symbols include `_resolve_profiler_log_dir`, `SGLDiffusionProfiler`. / 该模块包含多模态生成体系中与 profiler 相关的运行时支持代码。 关键符号包括 `_resolve_profiler_log_dir`, `SGLDiffusionProfiler`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Imports and module setup / 导入与模块初始化
```python
import gzip
import os

import torch

from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.multimodal_gen.runtime.utils.logging_utils import CYAN, RESET, init_logger

if current_platform.is_npu():
    import torch_npu

    patches = [
        ["profiler.profile", torch_npu.profiler.profile],
        ["profiler.schedule", torch_npu.profiler.schedule],
    ]
    torch_npu._apply_patches(patches)

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 21-29: Function `_resolve_profiler_log_dir` / 函数 `_resolve_profiler_log_dir`
```python
def _resolve_profiler_log_dir(log_dir: str | None) -> str:
    if log_dir is not None:
        return log_dir

    diffusion_profiler_dir = os.getenv("SGLANG_DIFFUSION_TORCH_PROFILER_DIR")
    if diffusion_profiler_dir:
        return diffusion_profiler_dir

    return os.getenv("SGLANG_TORCH_PROFILER_DIR", "./logs")
```
**EN:** This function drives `_resolve_profiler_log_dir` with inputs such as `log_dir`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resolve_profiler_log_dir`，主要处理 `log_dir` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 32-193: Class `SGLDiffusionProfiler` / 类 `SGLDiffusionProfiler`
```python
class SGLDiffusionProfiler:
    """
    A wrapper around torch.profiler to simplify usage in pipelines.
    Supports both full profiling and scheduled profiling.


    1. if profile_all_stages is on: profile all stages, including all denoising steps
    2. otherwise, if num_profiled_timesteps is specified: profile {num_profiled_timesteps} denoising steps. profile all steps if num_profiled_timesteps==-1
    """

    _instance = None

    def __init__(
        self,
# ...
            return True
        except Exception as e:
            logger.warning(f"Trace file integrity check failed: {e}")
            return False
```
**EN:** This class models `SGLDiffusionProfiler`. A wrapper around torch.profiler to simplify usage in pipelines. Important methods include `__init__`, `start`, `_step`, `step_stage`.
**CN:** 该类实现 `SGLDiffusionProfiler`。 文档字符串指出：A wrapper around torch.profiler to simplify usage in pipelines. 其中较重要的方法包括 `__init__`, `start`, `_step`, `step_stage`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Symbol `_resolve_profiler_log_dir` anchors the module API / 符号 `_resolve_profiler_log_dir` 构成该模块的核心 API
- Symbol `SGLDiffusionProfiler` anchors the module API / 符号 `SGLDiffusionProfiler` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `gzip`, `torch`, `torch_npu`
- **Stdlib / 标准库**: `os`
