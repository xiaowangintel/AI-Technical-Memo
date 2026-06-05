# adaptive_runtime_state.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/adaptive_runtime_state.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements speculative decoding infrastructure, such as workers, metadata objects, CUDA-graph runners, or shared utilities. It coordinates draft-model execution with target-model verification to improve throughput. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23: Module header, imports, and shared constants
```python
import logging
from dataclasses import dataclass
from typing import TYPE_CHECKING, Protocol

from sglang.srt.speculative.adaptive_spec_params import (
    AdaptiveSpeculativeParams,
    load_adaptive_config,
)

if TYPE_CHECKING:
    from sglang.srt.layers.attention.base_attn_backend import AttentionBackend
    from sglang.srt.model_executor.cpu_graph_runner import CPUGraphRunner
    from sglang.srt.model_executor.cuda_graph_runner import CudaGraphRunner
    from sglang.srt.speculative.eagle_draft_cuda_graph_runner import (
        EAGLEDraftCudaGraphRunner,
    )
    from sglang.srt.speculative.eagle_draft_extend_cuda_graph_runner import (
        EAGLEDraftExtendCudaGraphRunner,
    )

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 25-49: SpecRuntimeState class declaration
```python
class SpecRuntimeState:
    """A complete set of runtime resources bound to a specific speculative
    decoding configuration.

    Each decode round runs three stages — draft, verify, extend — and every
    stage has shape-dependent resources (attention backends and CUDA graphs)
    that must match the current configuration.  Switching adaptive steps
    means swapping the entire state atomically.
    """

    # -- Configuration (determines shapes for all stages) --
    speculative_num_steps: int
    speculative_num_draft_tokens: int

    # -- Draft stage: draft model multi-step autoregressive generation --
    draft_attn_backend: "AttentionBackend | None"
    cuda_graph_runner: "EAGLEDraftCudaGraphRunner | None"

    # -- Verify stage: target model one-pass tree verification --
    target_attn_backend: "AttentionBackend"
    target_graph_runner: "CudaGraphRunner | CPUGraphRunner | None"

    # -- Extend stage: draft model KV cache catch-up after verify --
    draft_extend_attn_backend: "AttentionBackend | None"
    cuda_graph_runner_for_draft_extend: "EAGLEDraftExtendCudaGraphRunner | None"
```
**EN:** This block declares the `SpecRuntimeState` class, which exists to store configuration or metadata. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `SpecRuntimeState` 类，其职责是存储配置或元数据。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 52-56: AdaptiveSpecWorker class declaration
```python
class AdaptiveSpecWorker(Protocol):
    """Protocol that a worker must implement to use AdaptiveController."""

    speculative_num_steps: int
```
**EN:** This block declares the `AdaptiveSpecWorker` class, which exists to coordinate staged runtime execution. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `AdaptiveSpecWorker` 类，其职责是协调分阶段运行时执行。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 57-59: AdaptiveSpecWorker.build_adaptive_runtime_state method
```python
    def build_adaptive_runtime_state(
        self, speculative_num_steps: int, speculative_num_draft_tokens: int
    ) -> SpecRuntimeState: ...
```
**EN:** This block uses `AdaptiveSpecWorker.build_adaptive_runtime_state` to construct helper objects or layouts. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AdaptiveSpecWorker.build_adaptive_runtime_state` 来构建辅助对象或布局。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 61-61: AdaptiveSpecWorker.apply_runtime_state method
```python
    def apply_runtime_state(self, state: SpecRuntimeState) -> None: ...
```
**EN:** This block uses `AdaptiveSpecWorker.apply_runtime_state` to apply a transformation to tensors or state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AdaptiveSpecWorker.apply_runtime_state` 来对张量或状态应用变换。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 64-76: AdaptiveController class declaration
```python
class AdaptiveController:
    """Facade that owns adaptive decision-making and runtime state switching.

    Works with any worker that implements ``AdaptiveSpecWorker`` protocol:
      - ``build_adaptive_runtime_state(steps, draft_tokens)`` → runtime state
      - ``apply_runtime_state(state)`` → apply it to the worker

    The worker only needs to:
      1. Call ``register()`` for the initial state, then ``init_states()``
         once during startup.
      2. Call ``on_verify_complete(num_correct_drafts_per_req)`` after each decode verify.
    """
```
**EN:** This block declares the `AdaptiveController` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `AdaptiveController` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 77-84: AdaptiveController initializer
```python
    def __init__(self, worker: AdaptiveSpecWorker, config_path: str | None = None):
        self.worker = worker
        cfg = load_adaptive_config(config_path)
        self.params = AdaptiveSpeculativeParams(
            initial_steps=worker.speculative_num_steps,
            config=cfg,
        )
        self._states: dict[int, SpecRuntimeState] = {}
```
**EN:** This block initializes the `AdaptiveController` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `AdaptiveController` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 86-88: AdaptiveController.candidate_steps method
```python
    @property
    def candidate_steps(self) -> list[int]:
        return self.params.candidate_steps
```
**EN:** This block uses `AdaptiveController.candidate_steps` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AdaptiveController.candidate_steps` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 90-96: AdaptiveController.register method
```python
    def register(self, state: SpecRuntimeState, steps: int | None = None) -> None:
        """Register a pre-built runtime state.

        *steps* defaults to ``state.speculative_num_steps`` when not given.
        """
        key = steps if steps is not None else state.speculative_num_steps
        self._states[key] = state
```
**EN:** This block uses `AdaptiveController.register` to register components for later lookup. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AdaptiveController.register` 来注册组件以供后续查找。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 98-108: AdaptiveController.init_states method
```python
    def init_states(self) -> None:
        """Build and register runtime states for all candidate steps."""
        for steps in self.params.candidate_steps:
            if steps in self._states:
                continue
            state = self.worker.build_adaptive_runtime_state(
                speculative_num_steps=steps,
                speculative_num_draft_tokens=steps + 1,
            )
            self._states[steps] = state
        self._activate(self.params.current_steps)
```
**EN:** This block uses `AdaptiveController.init_states` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AdaptiveController.init_states` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 110-113: AdaptiveController.on_verify_complete method
```python
    def on_verify_complete(self, num_correct_drafts_per_req: list[int]) -> None:
        """Feed verify results; switch runtime state if EMA warrants it."""
        if self.params.update(num_correct_drafts_per_req):
            self._activate(self.params.current_steps)
```
**EN:** This block uses `AdaptiveController.on_verify_complete` to verify draft results against the target model. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AdaptiveController.on_verify_complete` 来根据目标模型校验草稿结果。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 115-121: AdaptiveController._activate method
```python
    def _activate(self, speculative_num_steps: int) -> None:
        state = self._states.get(speculative_num_steps)
        if state is None:
            raise ValueError(
                f"Missing adaptive runtime state for steps={speculative_num_steps}"
            )
        self.worker.apply_runtime_state(state)
```
**EN:** This block uses `AdaptiveController._activate` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AdaptiveController._activate` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Adaptive speculative control / 自适应推测控制
- Speculative decoding / 推测解码
- Graph-captured execution paths / 图捕获执行路径

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.layers.attention.base_attn_backend`
- `sglang.srt.model_executor.cpu_graph_runner`
- `sglang.srt.model_executor.cuda_graph_runner`
- `sglang.srt.speculative.adaptive_spec_params`
- `sglang.srt.speculative.eagle_draft_cuda_graph_runner`
- `sglang.srt.speculative.eagle_draft_extend_cuda_graph_runner`
### External / 外部
- `dataclasses` (stdlib)
- `logging` (stdlib)
- `typing` (stdlib)
