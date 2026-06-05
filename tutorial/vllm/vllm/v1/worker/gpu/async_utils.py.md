# async_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/async_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `AsyncOutput`, `AsyncPoolingOutput`, `async_copy_to_np` for the V1 `worker/gpu` subsystem. / 为 V1 的 `worker/gpu` 子系统实现 `AsyncOutput`, `AsyncPoolingOutput`, `async_copy_to_np`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import contextlib

import numpy as np
import torch

from vllm.v1.outputs import AsyncModelRunnerOutput, LogprobsTensors, ModelRunnerOutput
from vllm.v1.worker.gpu.sample.output import SamplerOutput
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.v1.outputs`, `vllm.v1.worker.gpu.sample.output`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.v1.outputs`, `vllm.v1.worker.gpu.sample.output` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `AsyncOutput` class / `AsyncOutput` 类
```python
class AsyncOutput(AsyncModelRunnerOutput):
```
**EN:** Introduces the `AsyncOutput` class on top of `AsyncModelRunnerOutput`. Core methods include `__init__`, `get_output`.
**CN:** 这里定义 `AsyncOutput` 类，其基类包括 `AsyncModelRunnerOutput`。核心方法包括 `__init__`, `get_output`。

### `AsyncOutput.__init__` method / `AsyncOutput.__init__` 方法
```python
    def __init__(
        self,
        model_runner_output: ModelRunnerOutput,
        sampler_output: SamplerOutput,
        num_sampled_tokens: torch.Tensor,
        main_stream: torch.cuda.Stream,
        copy_stream: torch.cuda.Stream,
    ):
        # NOTE(woosuk): We must retain references to the GPU tensors,
        # as the copy operations are performed on a different CUDA stream than
        # the one where the tensors were created.
        self.model_runner_output = model_runner_output
        self.sampler_output = sampler_output
        self.num_sampled_tokens = num_sampled_tokens
        self.copy_event = torch.cuda.Event()

        with stream(copy_stream, main_stream):
            copy_stream.wait_stream(main_stream)

            self.sampled_token_ids = async_copy_to_np(sampler_output.sampled_token_ids)
            self.logprobs_tensors: LogprobsTensors | None = None
            if sampler_output.logprobs_tensors is not None:
                self.logprobs_tensors = (
                    sampler_output.logprobs_tensors.to_cpu_nonblocking()
                )
            self.num_nans: np.ndarray | None = None
            if sampler_output.num_nans is not None:
                self.num_nans = async_copy_to_np(sampler_output.num_nans)
            self.num_sampled_tokens_np = async_copy_to_np(num_sampled_tokens)
            self.prompt_logprobs_dict = {
                k: v.to_cpu_nonblocking() if v is not None else None
                for k, v in self.model_runner_output.prompt_logprobs_dict.items()
            }
            self.copy_event.record(copy_stream)
```
**EN:** This method initializes the object state within `AsyncOutput`. Key calls include `Event`, `stream`, `wait_stream`, `async_copy_to_np`, `record`, `to_cpu_nonblocking`. It touches state such as `model_runner_output`, `sampler_output`, `num_sampled_tokens`, `copy_event`, `sampled_token_ids`, `logprobs_tensors`, `num_nans`, `num_sampled_tokens_np`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`AsyncOutput`。 关键调用包括 `Event`, `stream`, `wait_stream`, `async_copy_to_np`, `record`, `to_cpu_nonblocking`。 它会读写 `model_runner_output`, `sampler_output`, `num_sampled_tokens`, `copy_event`, `sampled_token_ids`, `logprobs_tensors`, `num_nans`, `num_sampled_tokens_np` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AsyncOutput.get_output` method / `AsyncOutput.get_output` 方法
```python
    def get_output(self) -> ModelRunnerOutput:
        self.copy_event.synchronize()

        # NOTE(woosuk): The following code is to ensure compatibility with
        # the existing model runner.
        # Going forward, we should keep the data structures as NumPy arrays
        # rather than Python lists.
        sampled_token_ids: list[list[int]] = self.sampled_token_ids.tolist()
        num_sampled_tokens: list[int] = self.num_sampled_tokens_np.tolist()
        for token_ids, num_tokens in zip(sampled_token_ids, num_sampled_tokens):
            del token_ids[num_tokens:]
        self.model_runner_output.sampled_token_ids = sampled_token_ids

        if self.num_nans is not None:
            self.model_runner_output.num_nans_in_logits = dict(
                zip(self.model_runner_output.req_ids, self.num_nans.tolist())
            )

        if self.logprobs_tensors is not None:
            self.model_runner_output.logprobs = self.logprobs_tensors.tolists()
        self.model_runner_output.prompt_logprobs_dict = self.prompt_logprobs_dict
        return self.model_runner_output
```
**EN:** This method returns or derives a value within `AsyncOutput`. Key calls include `synchronize`, `tolist`, `zip`, `dict`, `tolists`. It touches state such as `model_runner_output`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`AsyncOutput`。 关键调用包括 `synchronize`, `tolist`, `zip`, `dict`, `tolists`。 它会读写 `model_runner_output` 等状态。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `AsyncPoolingOutput` class / `AsyncPoolingOutput` 类
```python
class AsyncPoolingOutput(AsyncModelRunnerOutput):
```
**EN:** Introduces the `AsyncPoolingOutput` class on top of `AsyncModelRunnerOutput`. Core methods include `__init__`, `get_output`.
**CN:** 这里定义 `AsyncPoolingOutput` 类，其基类包括 `AsyncModelRunnerOutput`。核心方法包括 `__init__`, `get_output`。

### `AsyncPoolingOutput.__init__` method / `AsyncPoolingOutput.__init__` 方法
```python
    def __init__(
        self,
        model_runner_output: ModelRunnerOutput,
        pooler_output: torch.Tensor,
        is_valid: torch.Tensor | None,
        main_stream: torch.cuda.Stream,
        copy_stream: torch.cuda.Stream,
    ):
        self.model_runner_output = model_runner_output
        self.pooler_output = pooler_output
        self.is_valid = is_valid
        self.copy_event = torch.cuda.Event()

        with stream(copy_stream, main_stream):
            copy_stream.wait_stream(main_stream)
            self.pooler_output_cpu = self.pooler_output.to("cpu", non_blocking=True)
            if self.is_valid is not None:
                self.is_valid_cpu = self.is_valid.to("cpu", non_blocking=True)
            else:
                self.is_valid_cpu = None
            self.copy_event.record(copy_stream)
```
**EN:** This method initializes the object state within `AsyncPoolingOutput`. Key calls include `Event`, `stream`, `wait_stream`, `to`, `record`. It touches state such as `model_runner_output`, `pooler_output`, `is_valid`, `copy_event`, `pooler_output_cpu`, `is_valid_cpu`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`AsyncPoolingOutput`。 关键调用包括 `Event`, `stream`, `wait_stream`, `to`, `record`。 它会读写 `model_runner_output`, `pooler_output`, `is_valid`, `copy_event`, `pooler_output_cpu`, `is_valid_cpu` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AsyncPoolingOutput.get_output` method / `AsyncPoolingOutput.get_output` 方法
```python
    def get_output(self) -> ModelRunnerOutput:
        pooler_output = list(self.pooler_output_cpu.unbind(dim=0))
        self.copy_event.synchronize()
        if self.is_valid_cpu is not None:
            is_valid_cpu = self.is_valid_cpu.tolist()
            for i, is_valid in enumerate(is_valid_cpu):
                if not is_valid:
                    pooler_output[i] = None
        self.model_runner_output.pooler_output = pooler_output
        return self.model_runner_output
```
**EN:** This method returns or derives a value within `AsyncPoolingOutput`. Key calls include `list`, `synchronize`, `unbind`, `tolist`, `enumerate`. It touches state such as `model_runner_output`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`AsyncPoolingOutput`。 关键调用包括 `list`, `synchronize`, `unbind`, `tolist`, `enumerate`。 它会读写 `model_runner_output` 等状态。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `async_copy_to_np` function / `async_copy_to_np` 函数
```python
def async_copy_to_np(x: torch.Tensor) -> np.ndarray:
    return x.to("cpu", non_blocking=True).numpy()
```
**EN:** This function implements `async_copy_to_np` within the module. Key calls include `numpy`, `to`.
**CN:** 该函数会实现 `async_copy_to_np`，其作用域位于the module。 关键调用包括 `numpy`, `to`。

### `stream` function / `stream` 函数
```python
@contextlib.contextmanager
def stream(to_stream: torch.cuda.Stream, from_stream: torch.cuda.Stream):
    """Lightweight version of torch.cuda.stream() context manager which
    avoids current_stream and device lookups.
    """
    try:
        torch.cuda.set_stream(to_stream)
        yield
    finally:
        torch.cuda.set_stream(from_stream)
```
**EN:** This function implements `stream` within the module. The docstring frames it as: Lightweight version of torch.cuda.stream() context manager which avoids current_stream and device lookups. Key calls include `set_stream`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `stream`，其作用域位于the module。 关键调用包括 `set_stream`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `AsyncOutput`: central class or interface in this module. / `AsyncOutput`：本模块中的核心类或接口。
- `AsyncPoolingOutput`: central class or interface in this module. / `AsyncPoolingOutput`：本模块中的核心类或接口。
- `async_copy_to_np`: top-level helper or orchestration entry point. / `async_copy_to_np`：顶层辅助函数或编排入口。
- `stream`: top-level helper or orchestration entry point. / `stream`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `contextlib`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.v1.outputs`, `vllm.v1.worker.gpu.sample.output`
