# interpreter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/lang/interpreter.py`
- **Repository**: sgl-project/sglang
- **Purpose**: The interpreter that executes SGL programs / 该文件围绕上述模块职责展开实现，组织核心流程、辅助逻辑以及与相邻组件的协作。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module-level supporting statements
```python
"""The interpreter that executes SGL programs"""
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 2-2: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 3-3: Module-level supporting statements
```python
import asyncio
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 4-4: Module-level supporting statements
```python
import contextvars
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 5-5: Module-level supporting statements
```python
import copy
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 6-6: Module-level supporting statements
```python
import multiprocessing
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 7-7: Module-level supporting statements
```python
import queue
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 8-8: Module-level supporting statements
```python
import threading
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 9-9: Module-level supporting statements
```python
import uuid
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 10-10: Module-level supporting statements
```python
import warnings
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 11-11: Module-level supporting statements
```python
from concurrent.futures import ThreadPoolExecutor
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 12-12: Module-level supporting statements
```python
from contextlib import contextmanager
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 13-13: Module-level supporting statements
```python
from typing import Any, Callable, Dict, List, Optional
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 14-14: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 15-15: Module-level supporting statements
```python
import tqdm
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 16-16: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 17-17: Module-level supporting statements
```python
from sglang.global_config import global_config
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 18-34: Module-level supporting statements
```python
from sglang.lang.ir import (
    SglCommitLazy,
    SglConcateAndAppend,
    SglConstantText,
    SglExpr,
    SglExprList,
    SglGen,
    SglImage,
    SglRoleBegin,
    SglRoleEnd,
    SglSelect,
    SglSeparateReasoning,
    SglVariable,
    SglVarScopeBegin,
    SglVarScopeEnd,
    SglVideo,
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 35-39: Module-level supporting statements
```python
from sglang.utils import (
    encode_image_base64,
    encode_video_base64,
    get_exception_traceback,
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 40-41: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 42-54: run internal function
```python
def run_internal(state, program, func_args, func_kwargs, sync):
    try:
        state.ret_value = program.func(state, *func_args, **func_kwargs)
    except Exception as e:
        raise e
    finally:
        state.stream_executor.end()

    if sync:
        state.stream_executor.sync()

    if global_config.verbosity >= 2:
        print(state.text())
```
**EN:** This block uses `run_internal` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `run_internal` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 55-56: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 57-90: run program function
```python
def run_program(
    program,
    backend,
    func_args,
    func_kwargs,
    default_sampling_para,
    stream,
    sync=False,
    use_thread=True,
):
    if hasattr(backend, "endpoint"):
        backend = backend.endpoint
    assert backend is not None, "Please specify a backend"
    func_kwargs.update(program.bind_arguments)
    stream_executor = StreamExecutor(
        backend,
        func_kwargs,
        default_sampling_para,
        chat_template=None,
        stream=stream,
        num_api_spec_tokens=program.num_api_spec_tokens,
        use_thread=use_thread,
    )
    state = ProgramState(stream_executor)

    if stream:
        t = threading.Thread(
            target=run_internal, args=(state, program, func_args, func_kwargs, sync)
        )
        t.start()
        return state
    else:
        run_internal(state, program, func_args, func_kwargs, sync)
        return state
```
**EN:** This block uses `run_program` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `run_program` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 91-92: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 93-152: run program batch function (part 1/2)
```python
def run_program_batch(
    program,
    backend,
    batch_arguments,
    default_sampling_para,
    num_threads,
    progress_bar,
    generator_style=False,
):
    if hasattr(backend, "endpoint"):
        backend = backend.endpoint

    # Pre-cache the common prefix for a batch. The prefix is extracted by tracing the program.
    if global_config.enable_precache_with_tracing and len(batch_arguments) > 1:
        cache_program(program, backend)

    # Run all programs
    if num_threads == "auto":
        num_threads = max(96, multiprocessing.cpu_count() * 16)
    num_threads = min(num_threads, len(batch_arguments))

    if generator_style:
        return _run_program_batch_generator(
            program,
            backend,
            batch_arguments,
            default_sampling_para,
            num_threads,
            progress_bar,
        )

    # Original code path when generator_style=False
    if num_threads == 1:
        rets = []
        if progress_bar:
            for arguments in tqdm.tqdm(batch_arguments):
                rets.append(
                    run_program(
                        program,
                        backend,
                        (),
                        arguments,
                        default_sampling_para,
                        False,
                        True,
                    )
                )
        else:
            for arguments in batch_arguments:
                rets.append(
                    run_program(
                        program,
                        backend,
                        (),
                        arguments,
                        default_sampling_para,
                        False,
                        True,
                    )
                )
```
**EN:** This block uses `run_program_batch` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `run_program_batch` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 153-181: run program batch function (part 2/2)
```python
    else:
        if progress_bar:
            pbar = tqdm.tqdm(total=len(batch_arguments))

        with ThreadPoolExecutor(num_threads) as executor:
            futures = []
            for arguments in batch_arguments:
                futures.append(
                    executor.submit(
                        run_program,
                        program,
                        backend,
                        (),
                        arguments,
                        default_sampling_para,
                        False,
                        True,
                    )
                )
                if progress_bar:
                    futures[-1].add_done_callback(lambda _: pbar.update())

            rets = [f.result() for f in futures]
        rets[-1].sync()

        if progress_bar:
            pbar.close()

    return rets
```
**EN:** This block uses `run_program_batch` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `run_program_batch` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 182-183: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 184-239: run program batch generator function
```python
def _run_program_batch_generator(
    program,
    backend,
    batch_arguments,
    default_sampling_para,
    num_threads,
    progress_bar,
):
    """Helper function that yields results one by one using chunking to avoid overwhelming ThreadPoolExecutor."""
    if num_threads == 1:
        iterator = tqdm.tqdm(batch_arguments) if progress_bar else batch_arguments
        for arguments in iterator:
            yield run_program(
                program,
                backend,
                (),
                arguments,
                default_sampling_para,
                False,
                True,
            )
    else:
        pbar = tqdm.tqdm(total=len(batch_arguments)) if progress_bar else None

        # Process in chunks to avoid overwhelming ThreadPoolExecutor
        # Otherwise, ThreadPoolExecutor.submit will block after adding certain number of tasks
        # so we will never reach "yield" until all tasks are done
        chunk_size = 200

        with ThreadPoolExecutor(num_threads) as executor:
            for chunk_start in range(0, len(batch_arguments), chunk_size):
                chunk_end = min(chunk_start + chunk_size, len(batch_arguments))
                chunk_futures = []

                # Submit chunk of tasks
                for i in range(chunk_start, chunk_end):
                    future = executor.submit(
                        run_program,
                        program,
                        backend,
                        (),
                        batch_arguments[i],
                        default_sampling_para,
                        False,
                        True,
                    )
                    if pbar:
                        future.add_done_callback(lambda _: pbar.update())
                    chunk_futures.append(future)

                # Yield results from this chunk as they complete
                for future in chunk_futures:
                    yield future.result()

        if pbar:
            pbar.close()
```
**EN:** This block uses `_run_program_batch_generator` to execute the main control path. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_run_program_batch_generator` 来执行主要控制路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 240-241: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 242-247: cache program function
```python
def cache_program(program, backend):
    from sglang.lang.tracer import extract_prefix_by_tracing

    prefix = extract_prefix_by_tracing(program, backend)
    if prefix and len(prefix) > 64:
        backend.cache_prefix(prefix)
```
**EN:** This block uses `cache_program` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `cache_program` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 248-249: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 250-254: Module-level supporting statements
```python
_INCREMENTAL_STREAMING_META_INFO_KEYS = (
    "output_token_logprobs",
    "output_top_logprobs",
    "output_token_ids_logprobs",
)
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 255-256: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 257-271: merge stream meta info function
```python
def _merge_stream_meta_info(
    pending_meta_info: dict[str, Any] | None,
    meta_info: dict[str, Any],
) -> dict[str, Any]:
    if pending_meta_info is None:
        return meta_info

    merged_meta_info = dict(meta_info)
    for key in _INCREMENTAL_STREAMING_META_INFO_KEYS:
        if key not in meta_info and key not in pending_meta_info:
            continue
        merged_meta_info[key] = list(pending_meta_info.get(key, [])) + list(
            meta_info.get(key, [])
        )
    return merged_meta_info
```
**EN:** This block uses `_merge_stream_meta_info` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_merge_stream_meta_info` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 272-273: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 274-274: StreamExecutor class declaration
```python
class StreamExecutor:
```
**EN:** This block declares the `StreamExecutor` class, which exists to coordinate staged execution. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `StreamExecutor` 类，其职责是协调分阶段执行。它定义了本文件其余部分使用的结构与成员布局。

### Lines 275-275: Class-level supporting statements
```python
    """A stream executor that executes SGL expressions in a background thread."""
```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 276-276: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 277-336: StreamExecutor initializer (part 1/2)
```python
    def __init__(
        self,
        backend,
        arguments,
        default_sampling_para,
        chat_template,
        stream,
        num_api_spec_tokens=None,
        use_thread=True,
    ):
        from sglang.lang.backend.base_backend import BaseBackend

        self.sid = uuid.uuid4().hex
        self.backend: BaseBackend = backend
        self.arguments: Dict[str, Any] = arguments
        self.default_sampling_para = default_sampling_para
        self.stream = stream

        self.variables = {}  # Dict[name: str -> value: str]
        self.variable_event = {}  # Dict[name: str -> event: threading.Event]
        self.meta_info = {}  # Dict[name: str -> info: str]
        self.is_finished = False
        self.error_ = None

        # For completion
        self.text_ = ""  # The full text

        # For chat
        self.messages_ = []  # The messages in the OpenAI API format
        self.chat_template = chat_template or self.backend.get_chat_template()
        self.cur_role = None
        self.cur_role_begin_pos = None

        # For vision
        self.images_ = []
        self.cur_images = []

        # For fork/join
        self.fork_start_text_pos = None

        # For speculative execution
        self.num_api_spec_tokens = num_api_spec_tokens
        self.speculated_text = ""

        # Worker thread
        self.use_thread = use_thread
        if self.use_thread:
            self.queue = queue.Queue()

            def _run_worker_in_context():
                self._thread_worker_func()

            self.worker = threading.Thread(
                target=contextvars.copy_context().run, args=(_run_worker_in_context,)
            )
            self.worker.start()

        # For streaming
        if stream:
            self.stream_text_event = threading.Event()
```
**EN:** This block initializes the `StreamExecutor` object, wiring together the state and dependencies used by later methods. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块初始化 `StreamExecutor` 对象，连接后续方法使用的状态与依赖。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 337-340: StreamExecutor initializer (part 2/2)
```python
            self.stream_var_event = {}
        else:
            self.stream_text_event = None
            self.stream_var_event = None
```
**EN:** This block initializes the `StreamExecutor` object, wiring together the state and dependencies used by later methods. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块初始化 `StreamExecutor` 对象，连接后续方法使用的状态与依赖。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 341-341: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 342-348: StreamExecutor.submit method
```python
    def submit(self, expr: SglExpr):
        self._init_var_event(expr)

        if self.use_thread:
            self.queue.put(expr)
        else:
            self._execute(expr)
```
**EN:** This block uses `StreamExecutor.submit` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor.submit` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 349-349: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 350-352: StreamExecutor.sync method
```python
    def sync(self):
        if self.use_thread:
            self.queue.join()
```
**EN:** This block uses `StreamExecutor.sync` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor.sync` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 353-353: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 354-357: StreamExecutor.get_var method
```python
    def get_var(self, name):
        if name in self.variable_event:
            self.variable_event[name].wait()
        return self.variables[name]
```
**EN:** This block uses `StreamExecutor.get_var` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor.get_var` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 358-358: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 359-360: StreamExecutor.set_var method
```python
    def set_var(self, name, value):
        self.variables[name] = value
```
**EN:** This block uses `StreamExecutor.set_var` to update stored state. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor.set_var` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 361-361: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 362-368: StreamExecutor.get_meta_info method
```python
    def get_meta_info(self, name, timeout=None):
        if name in self.variable_event:
            got = self.variable_event[name].wait(timeout)
            if not got:
                raise TimeoutError(f"Timeout while waiting for event '{name}'")
        ret = self.meta_info.get(name, None)
        return ret
```
**EN:** This block uses `StreamExecutor.get_meta_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor.get_meta_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 369-369: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 370-402: StreamExecutor.fork method
```python
    def fork(
        self,
        size: int = 1,
        position_ids_offset: Optional[List[int]] = None,
    ):
        if size > 1 and str(self.text_):
            self.submit(SglCommitLazy())

        self.sync()
        size = int(size)

        exes = [
            StreamExecutor(
                self.backend,
                self.arguments,
                self.default_sampling_para,
                self.chat_template,
                self.stream,
            )
            for _ in range(size)
        ]
        for i in range(size):
            exes[i].variables = dict(self.variables)
            exes[i].text_ = str(self.text_)
            exes[i].messages_ = list(self.messages_)
            exes[i].cur_role = self.cur_role
            exes[i].cur_role_begin_pos = self.cur_role_begin_pos
            exes[i].fork_start_text_pos = len(self.text_)
            exes[i].images_ = list(self.images_)

            # TODO(ying): handle API speculative execution

        return exes
```
**EN:** This block uses `StreamExecutor.fork` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor.fork` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 403-403: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 404-406: StreamExecutor.text method
```python
    def text(self):
        self.sync()
        return self.text_
```
**EN:** This block uses `StreamExecutor.text` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor.text` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 407-407: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 408-410: StreamExecutor.messages method
```python
    def messages(self):
        self.sync()
        return self.messages_
```
**EN:** This block uses `StreamExecutor.messages` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor.messages` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 411-411: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 412-414: StreamExecutor.error method
```python
    def error(self):
        self.sync()
        return self.error_
```
**EN:** This block uses `StreamExecutor.error` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor.error` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 415-415: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 416-420: StreamExecutor.end method
```python
    def end(self):
        if self.use_thread:
            if self.worker.is_alive():
                self.queue.put(None)
        self.backend.end_program(self)
```
**EN:** This block uses `StreamExecutor.end` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor.end` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 421-421: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 422-459: StreamExecutor._thread_worker_func method
```python
    def _thread_worker_func(self):
        error = None

        while True:
            expr = self.queue.get()
            if expr is None:
                self.queue.task_done()
                break

            try:
                self._execute(expr)
            except Exception as e:
                warnings.warn(f"Error in stream_executor: {get_exception_traceback()}")
                error = e
                break
            self.queue.task_done()
            if self.stream_text_event:
                self.stream_text_event.set()

        # Clean the queue and events
        if error is not None:
            try:
                while True:
                    self.queue.task_done()
                    self.queue.get_nowait()
            except queue.Empty:
                pass
            for name in self.variable_event:
                self.variable_event[name].set()
            if self.stream_var_event:
                for name in self.stream_var_event:
                    self.stream_var_event[name].set()
            self.error_ = error

        if self.stream_text_event:
            self.stream_text_event.set()

        self.is_finished = True
```
**EN:** This block uses `StreamExecutor._thread_worker_func` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._thread_worker_func` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 460-460: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 461-503: StreamExecutor._execute method
```python
    def _execute(self, other):
        if isinstance(other, str):
            other = SglConstantText(other)

        assert isinstance(other, SglExpr), f"{other}"

        if isinstance(other, SglConstantText):
            self._execute_fill(other.value)
        elif isinstance(other, SglGen):
            self._execute_gen(other)
        elif isinstance(other, SglSelect):
            self._execute_select(other)
        elif isinstance(other, SglExprList):
            for x in other.expr_list:
                self._execute(x)
        elif isinstance(other, SglRoleBegin):
            self._execute_role_begin(other)
        elif isinstance(other, SglRoleEnd):
            self._execute_role_end(other)
        elif isinstance(other, SglImage):
            self._execute_image(other)
        elif isinstance(other, SglVideo):
            self._execute_video(other)
        elif isinstance(other, SglVariable):
            self._execute_variable(other)
        elif isinstance(other, SglVarScopeBegin):
            self._execute_var_scope_begin(other)
        elif isinstance(other, SglVarScopeEnd):
            self._execute_var_scope_end(other)
        elif isinstance(other, SglCommitLazy):
            self._execute_commit_lazy_operations(other)
        elif isinstance(other, SglConcateAndAppend):
            if (
                global_config.enable_parallel_encoding
                and self.backend.support_concate_and_append
            ):
                self._execute_concatenate_and_append_kv_cache(other)
            else:
                self._execute_concatenate_and_append_text(other)
        elif isinstance(other, SglSeparateReasoning):
            self._execute_separate_reasoning(other)
        else:
            raise ValueError(f"Unknown type: {type(other)}")
```
**EN:** This block uses `StreamExecutor._execute` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 504-504: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 505-522: StreamExecutor._execute_fill method
```python
    def _execute_fill(self, value: str, prefix=False):
        value = str(value)

        if (
            self.cur_role == "assistant"
            and self.num_api_spec_tokens is not None
            and self.backend.is_chat_model
            and not prefix
        ):
            self.backend.spec_fill(value)
            return

        if self.speculated_text.startswith(value):
            self.speculated_text = self.speculated_text[len(value) :]
        else:
            self.speculated_text = ""

        self.text_ += value
```
**EN:** This block uses `StreamExecutor._execute_fill` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_fill` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 523-523: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 524-531: StreamExecutor._execute_image method
```python
    def _execute_image(self, expr: SglImage):
        path = expr.path

        base64_data = encode_image_base64(path)

        self.images_.append((path, base64_data))
        self.cur_images.append((path, base64_data))
        self.text_ += self.chat_template.image_token
```
**EN:** This block uses `StreamExecutor._execute_image` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_image` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 532-532: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 533-541: StreamExecutor._execute_video method
```python
    def _execute_video(self, expr: SglVideo):
        path = expr.path
        num_frames = expr.num_frames

        base64_data = encode_video_base64(path, num_frames)

        self.images_.append((path, base64_data))
        self.cur_images.append((path, base64_data))
        self.text_ += self.chat_template.image_token
```
**EN:** This block uses `StreamExecutor._execute_video` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_video` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 542-542: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 543-591: StreamExecutor._spec_gen method
```python
    def _spec_gen(self, sampling_params):
        stop = sampling_params.stop
        max_new_tokens = sampling_params.max_new_tokens
        meta_info = {}

        def regen():
            nonlocal meta_info

            sampling_params.max_new_tokens = max(
                sampling_params.max_new_tokens, self.num_api_spec_tokens
            )
            sampling_params.stop = None
            self.speculated_text, meta_info = self.backend.generate(
                self, sampling_params=sampling_params
            )

        def find_stop():
            if isinstance(stop, str):
                return self.speculated_text.find(stop)
            elif isinstance(stop, (tuple, list)):
                pos = -1
                for stop_str in stop:
                    stop_pos = self.speculated_text.find(stop_str)
                    if stop_pos != -1 and (pos == -1 or stop_pos < pos):
                        pos = stop_pos
                return pos
            else:
                raise Exception("Wrong type of stop in sampling parameters.")

        if stop is None:
            if len(self.speculated_text) < max_new_tokens:
                regen()
            comp = self.speculated_text[:max_new_tokens]
            self.speculated_text = self.speculated_text[max_new_tokens:]
        elif isinstance(stop, (str, list, tuple)):
            if self.speculated_text == "":
                regen()
            stop_pos = find_stop()
            if stop_pos == -1:
                stop_pos = min(
                    sampling_params.max_new_tokens,
                    len(self.speculated_text),
                )
            comp = self.speculated_text[:stop_pos]
            self.speculated_text = self.speculated_text[stop_pos:]
        else:
            raise ValueError("Wrong type of stop in sampling parameters.")

        return comp, meta_info
```
**EN:** This block uses `StreamExecutor._spec_gen` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._spec_gen` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 592-592: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 593-645: StreamExecutor._execute_gen method
```python
    def _execute_gen(self, expr: SglGen):
        sampling_params = self._resolve_sampling_params(expr.sampling_params)
        name = expr.name
        if not self.stream:
            if self.num_api_spec_tokens is None:
                comp, meta_info = self.backend.generate(
                    self,
                    sampling_params=sampling_params,
                )

            else:
                if self.backend.is_chat_model:
                    # Speculative execution on models with only chat interface.
                    # Store the calls into a temporary list.
                    # They will be lazily executed later.
                    comp, meta_info = self.backend.generate(
                        self,
                        sampling_params=sampling_params,
                        spec_var_name=name,
                    )
                    return

                else:  # Speculative execution on models with completion interface
                    comp, meta_info = self._spec_gen(sampling_params)
            if isinstance(comp, list):
                self.text_ += comp[0]
            else:
                assert isinstance(comp, str)
                self.text_ += comp

            self.variables[name] = comp
            self.meta_info[name] = meta_info
            self.variable_event[name].set()
        else:
            assert (
                self.num_api_spec_tokens is None
            ), "stream is not supported with api speculative execution"
            generator = self.backend.generate_stream(
                self, sampling_params=sampling_params
            )

            self.variables[name] = ""
            self.stream_var_event[name].set()

            for comp, meta_info in generator:
                self.text_ += comp
                self.variables[name] += comp
                self.meta_info[name] = meta_info
                self.stream_var_event[name].set()
                self.stream_text_event.set()

            self.variable_event[name].set()
            self.stream_var_event[name].set()
```
**EN:** This block uses `StreamExecutor._execute_gen` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_gen` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 646-646: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 647-658: StreamExecutor._execute_select method
```python
    def _execute_select(self, expr: SglSelect):
        choices_decision = self.backend.select(
            self, expr.choices, expr.temperature, expr.choices_method
        )
        if expr.name is not None:
            name = expr.name
            self.variables[name] = choices_decision.decision
            self.meta_info[name] = choices_decision.meta_info
            self.variable_event[name].set()
            if self.stream_var_event:
                self.stream_var_event[name].set()
        self.text_ += choices_decision.decision
```
**EN:** This block uses `StreamExecutor._execute_select` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_select` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 659-659: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 660-663: StreamExecutor._execute_variable method
```python
    def _execute_variable(self, expr: SglVariable):
        src_executor = expr.source_stream_executor
        value = src_executor.get_var(expr.name)
        self._execute_fill(value)
```
**EN:** This block uses `StreamExecutor._execute_variable` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_variable` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 664-664: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 665-681: StreamExecutor._execute_role_begin method
```python
    def _execute_role_begin(self, expr: SglRoleBegin):
        assert self.cur_role is None, "Nested roles are not allowed."

        if len(self.messages_) == 0 and expr.role != "system":
            # Insert the default system message
            default_system = self.chat_template.default_system_prompt
            if default_system:
                self._execute_role_begin(SglRoleBegin("system"))
                self._execute_fill(default_system)
                self._execute_role_end(SglRoleEnd("system"))

        self.cur_role = expr.role

        prefix, _ = self.chat_template.get_prefix_and_suffix(expr.role, self.messages_)

        self._execute_fill(prefix, prefix=True)
        self.cur_role_begin_pos = len(self.text_)
```
**EN:** This block uses `StreamExecutor._execute_role_begin` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_role_begin` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 682-682: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 683-717: StreamExecutor._execute_role_end method
```python
    def _execute_role_end(self, expr: SglRoleEnd):
        if (
            self.cur_role == "assistant"
            and self.num_api_spec_tokens is not None
            and self.backend.is_chat_model
        ):
            # Execute the stored lazy generation calls
            self.backend.role_end_generate(self)
        self.cur_role = None

        new_text = self.text_[self.cur_role_begin_pos :].lstrip()

        _, suffix = self.chat_template.get_prefix_and_suffix(expr.role, self.messages_)
        self._execute_fill(suffix)

        if self.cur_images:
            # OpenAI vision API format
            last_msg = {
                "role": expr.role,
                "content": [{"type": "text", "text": new_text}],
            }
            for image_path, image_base64_data in self.cur_images:
                last_msg["content"].append(
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": f"data:image/jpeg;base64,{image_base64_data}"
                        },
                    }
                )
            self.messages_.append(last_msg)
            self.cur_images = []
        else:
            # OpenAI chat API format
            self.messages_.append({"role": expr.role, "content": new_text})
```
**EN:** This block uses `StreamExecutor._execute_role_end` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_role_end` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 718-718: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 719-720: StreamExecutor._execute_var_scope_begin method
```python
    def _execute_var_scope_begin(self, expr: SglVarScopeBegin):
        self.variables[expr.name] = int(len(self.text_))
```
**EN:** This block uses `StreamExecutor._execute_var_scope_begin` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_var_scope_begin` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 721-721: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 722-724: StreamExecutor._execute_var_scope_end method
```python
    def _execute_var_scope_end(self, expr: SglVarScopeEnd):
        self.variables[expr.name] = self.text_[self.variables[expr.name] :]
        self.variable_event[expr.name].set()
```
**EN:** This block uses `StreamExecutor._execute_var_scope_end` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_var_scope_end` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 725-725: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 726-727: StreamExecutor._execute_commit_lazy_operations method
```python
    def _execute_commit_lazy_operations(self, expr: SglCommitLazy):
        self.backend.commit_lazy_operations(self)
```
**EN:** This block uses `StreamExecutor._execute_commit_lazy_operations` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_commit_lazy_operations` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 728-728: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 729-736: StreamExecutor._execute_concatenate_and_append_text method
```python
    def _execute_concatenate_and_append_text(self, expr: SglConcateAndAppend):
        new_text = ""
        for s in expr.states:
            exe = s.stream_executor
            exe.sync()
            new_text += exe.text_[exe.fork_start_text_pos :]

        self._execute_fill(new_text)
```
**EN:** This block uses `StreamExecutor._execute_concatenate_and_append_text` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_concatenate_and_append_text` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 737-737: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 738-752: StreamExecutor._execute_concatenate_and_append_kv_cache method
```python
    def _execute_concatenate_and_append_kv_cache(self, expr: SglConcateAndAppend):
        self_len = len(self.text_)

        for i, s in enumerate(expr.states):
            exe = s.stream_executor
            exe.submit(SglCommitLazy())

        for i, s in enumerate(expr.states):
            exe = s.stream_executor
            exe.sync()
            assert exe.fork_start_text_pos == self_len
            self.text_ += exe.text_[exe.fork_start_text_pos :]

        src_rids = [state.stream_executor.sid for state in expr.states]
        self.backend.concatenate_and_append(src_rids, self.sid)
```
**EN:** This block uses `StreamExecutor._execute_concatenate_and_append_kv_cache` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_concatenate_and_append_kv_cache` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 753-753: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 754-786: StreamExecutor._execute_separate_reasoning method
```python
    def _execute_separate_reasoning(self, expr: SglSeparateReasoning):
        if self.stream:
            # separate reasoning for stream is not supported
            return

        if (
            self.cur_role == "assistant"
            and self.num_api_spec_tokens is not None
            and self.backend.is_chat_model
        ):
            # Execute the stored lazy generation calls
            self.backend.role_end_generate(self)

        from sglang.srt.parser.reasoning_parser import ReasoningParser

        reasoning_parser = ReasoningParser(expr.model_type)
        other = expr.expr
        if not other:
            return
        elif isinstance(other, SglGen) or isinstance(other, SglSelect):
            cur_text = self.get_var(other.name)
            reasoning, normal_text = reasoning_parser.parse_non_stream(cur_text)
            reasoning_name = expr.process_name_for_reasoning(other.name)
            self.set_var(other.name, normal_text)
            self.set_var(reasoning_name, reasoning)
            # the variable is ready to be used
            self.variable_event[reasoning_name].set()
            self.text_ = self.text_[: self.cur_role_begin_pos] + normal_text
        elif isinstance(other, SglExprList):
            for x in other.expr_list:
                self._execute_separate_reasoning(
                    SglSeparateReasoning(expr.model_type, x)
                )
```
**EN:** This block uses `StreamExecutor._execute_separate_reasoning` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._execute_separate_reasoning` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 787-787: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 788-797: StreamExecutor._init_var_event method
```python
    def _init_var_event(self, expr):
        if isinstance(
            expr, (SglGen, SglSelect, SglVarScopeBegin, SglSeparateReasoning)
        ):
            self.variable_event[expr.name] = threading.Event()
            if self.stream:
                self.stream_var_event[expr.name] = threading.Event()
        elif isinstance(expr, SglExprList):
            for e in expr.expr_list:
                self._init_var_event(e)
```
**EN:** This block uses `StreamExecutor._init_var_event` to initialize state and dependencies. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._init_var_event` 来初始化状态与依赖。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 798-798: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 799-846: StreamExecutor._resolve_sampling_params method
```python
    def _resolve_sampling_params(self, sampling_params):
        """
        Construct sampling param based on default + override values

        The default values of sampling are populated in `default_sampling_para` via sgl.function.run(...sampling_args)
        , and `sampling_params` contains the override values from sgl.gen().

        Here we use default_sampling_para as the base and override the values if they exist in `sampling_params`.
        It also extends the stop tokens based on the chat template.
        """

        # deepcopy is required because the dict has lists inside
        clone = copy.deepcopy(self.default_sampling_para)

        for item in [
            "max_new_tokens",
            "min_new_tokens",
            "n",
            "stop",
            "stop_token_ids",
            "stop_regex",
            "temperature",
            "top_p",
            "top_k",
            "min_p",
            "frequency_penalty",
            "presence_penalty",
            "ignore_eos",
            "return_logprob",
            "logprob_start_len",
            "top_logprobs_num",
            "return_text_in_logprobs",
            "dtype",
            "regex",
            "json_schema",
        ]:
            value = getattr(sampling_params, item, None)
            if value is not None:
                setattr(clone, item, value)

        if self.chat_template.stop_str:
            if clone.stop == ():
                clone.stop = []
            elif isinstance(clone.stop, str):
                clone.stop = [clone.stop]
            clone.stop += self.chat_template.stop_str

        return clone
```
**EN:** This block uses `StreamExecutor._resolve_sampling_params` to store configuration or metadata. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor._resolve_sampling_params` 来存储配置或元数据。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 847-847: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `StreamExecutor` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`StreamExecutor` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 848-849: StreamExecutor.__del__ method
```python
    def __del__(self):
        self.end()
```
**EN:** This block uses `StreamExecutor.__del__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `StreamExecutor.__del__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 850-851: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 852-852: ProgramState class declaration
```python
class ProgramState:
```
**EN:** This block declares the `ProgramState` class, which exists to store configuration or metadata. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `ProgramState` 类，其职责是存储配置或元数据。它定义了本文件其余部分使用的结构与成员布局。

### Lines 853-853: Class-level supporting statements
```python
    """The state of an SGL program."""
```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 854-854: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 855-856: ProgramState initializer
```python
    def __init__(self, stream_executor: StreamExecutor):
        self.stream_executor = stream_executor
```
**EN:** This block initializes the `ProgramState` object, wiring together the state and dependencies used by later methods.
**CN:** 该代码块初始化 `ProgramState` 对象，连接后续方法使用的状态与依赖。

### Lines 857-857: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 858-871: ProgramState._role_common method
```python
    def _role_common(self, name: str, expr: Optional[SglExpr] = None):
        if expr is not None:
            role_expr = SglExprList([SglRoleBegin(name), expr, SglRoleEnd(name)])
            self.stream_executor.submit(role_expr)
            return role_expr
        else:

            @contextmanager
            def role_scope():
                self.stream_executor.submit(SglRoleBegin(name))
                yield
                self.stream_executor.submit(SglRoleEnd(name))

            return role_scope()
```
**EN:** This block uses `ProgramState._role_common` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState._role_common` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 872-872: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 873-874: ProgramState.system method
```python
    def system(self, expr: Optional[SglExpr] = None):
        return self._role_common("system", expr)
```
**EN:** This block uses `ProgramState.system` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.system` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 875-875: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 876-877: ProgramState.user method
```python
    def user(self, expr: Optional[SglExpr] = None):
        return self._role_common("user", expr)
```
**EN:** This block uses `ProgramState.user` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.user` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 878-878: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 879-880: ProgramState.assistant method
```python
    def assistant(self, expr: Optional[SglExpr] = None):
        return self._role_common("assistant", expr)
```
**EN:** This block uses `ProgramState.assistant` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.assistant` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 881-882: Class-level supporting statements
```python

    @contextmanager
```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 883-886: ProgramState.var_scope method
```python
    def var_scope(self, name: str):
        self.stream_executor.submit(SglVarScopeBegin(name))
        yield
        self.stream_executor.submit(SglVarScopeEnd(name))
```
**EN:** This block uses `ProgramState.var_scope` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.var_scope` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 887-887: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 888-896: ProgramState.fork method
```python
    def fork(
        self,
        size: int = 1,
        position_ids_offset: Optional[List[int]] = None,
    ):
        stream_executors = self.stream_executor.fork(size, position_ids_offset)
        states = [ProgramState(x) for x in stream_executors]
        state_group = ProgramStateGroup(states, self)
        return state_group
```
**EN:** This block uses `ProgramState.fork` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.fork` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 897-898: Class-level supporting statements
```python

    @contextmanager
```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 899-904: ProgramState.copy method
```python
    def copy(self, position_ids_offset: Optional[List[int]] = None):
        state_group = self.fork(1, position_ids_offset)
        try:
            yield state_group[0]
        finally:
            state_group.join()
```
**EN:** This block uses `ProgramState.copy` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.copy` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 905-905: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 906-907: ProgramState.text method
```python
    def text(self):
        return self.stream_executor.text()
```
**EN:** This block uses `ProgramState.text` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.text` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 908-908: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 909-910: ProgramState.messages method
```python
    def messages(self):
        return self.stream_executor.messages()
```
**EN:** This block uses `ProgramState.messages` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.messages` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 911-911: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 912-913: ProgramState.sync method
```python
    def sync(self):
        return self.stream_executor.sync()
```
**EN:** This block uses `ProgramState.sync` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.sync` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 914-914: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 915-916: ProgramState.error method
```python
    def error(self):
        return self.stream_executor.error()
```
**EN:** This block uses `ProgramState.error` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.error` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 917-917: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 918-954: ProgramState.text_iter method
```python
    def text_iter(self, var_name: Optional[str] = None):
        if self.stream_executor.stream:
            prev = 0
            if var_name is None:
                event = self.stream_executor.stream_text_event
                while True:
                    event.wait()
                    event.clear()
                    out = str(self.stream_executor.text_[prev:])
                    prev += len(out)
                    if out:
                        yield out
                    if self.stream_executor.is_finished:
                        break
            else:
                event = None
                while not event:
                    if var_name in self.stream_executor.stream_var_event:
                        event = self.stream_executor.stream_var_event[var_name]
                    if self.stream_executor.is_finished:
                        yield ""
                        return

                while True:
                    event.wait()
                    event.clear()
                    out = str(self.stream_executor.variables[var_name][prev:])
                    prev += len(out)
                    if out:
                        yield out
                    if self.stream_executor.variable_event[var_name].is_set():
                        break
        else:
            if var_name is None:
                yield self.text()
            else:
                yield self.get_var(var_name)
```
**EN:** This block uses `ProgramState.text_iter` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.text_iter` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 955-955: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 956-1012: ProgramState.text_async_iter method
```python
    async def text_async_iter(
        self, var_name: Optional[str] = None, return_meta_data: bool = False
    ):
        loop = asyncio.get_running_loop()

        if self.stream_executor.stream:
            prev = 0
            if var_name is None:
                event = self.stream_executor.stream_text_event
                while True:
                    await loop.run_in_executor(None, event.wait)
                    event.clear()
                    out = str(self.stream_executor.text_[prev:])
                    prev += len(out)
                    if out:
                        yield out
                    if self.stream_executor.is_finished:
                        break
            else:
                event = None
                pending_meta_info = None
                while not event:
                    if var_name in self.stream_executor.stream_var_event:
                        event = self.stream_executor.stream_var_event[var_name]
                    if self.stream_executor.is_finished:
                        yield ""
                        return

                while True:
                    await loop.run_in_executor(None, event.wait)
                    event.clear()
                    out = str(self.stream_executor.variables[var_name][prev:])
                    meta_info = self.stream_executor.meta_info.get(var_name)
                    prev += len(out)
                    if out:
                        if return_meta_data:
                            assert meta_info is not None
                            merged_meta_info = _merge_stream_meta_info(
                                pending_meta_info,
                                meta_info,
                            )
                            pending_meta_info = None
                            yield out, merged_meta_info
                        else:
                            yield out
                    elif return_meta_data and meta_info is not None:
                        pending_meta_info = _merge_stream_meta_info(
                            pending_meta_info,
                            meta_info,
                        )
                    if self.stream_executor.variable_event[var_name].is_set():
                        break
        else:
            if var_name is None:
                yield self.text()
            else:
                yield self.get_var(var_name)
```
**EN:** This block uses `ProgramState.text_async_iter` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.text_async_iter` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1013-1013: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1014-1015: ProgramState.get_var method
```python
    def get_var(self, name):
        return self.stream_executor.get_var(name)
```
**EN:** This block uses `ProgramState.get_var` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.get_var` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1016-1016: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1017-1018: ProgramState.set_var method
```python
    def set_var(self, name, value):
        return self.stream_executor.set_var(name, value)
```
**EN:** This block uses `ProgramState.set_var` to update stored state. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.set_var` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1019-1019: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1020-1021: ProgramState.get_meta_info method
```python
    def get_meta_info(self, name):
        return self.stream_executor.get_meta_info(name)
```
**EN:** This block uses `ProgramState.get_meta_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.get_meta_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1022-1022: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1023-1027: ProgramState.__iadd__ method
```python
    def __iadd__(self, other):
        if other is None:
            raise ValueError("Tried to append None to state.")
        self.stream_executor.submit(other)
        return self
```
**EN:** This block uses `ProgramState.__iadd__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.__iadd__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1028-1028: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1029-1030: ProgramState.__getitem__ method
```python
    def __getitem__(self, name):
        return self.get_var(name)
```
**EN:** This block uses `ProgramState.__getitem__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.__getitem__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1031-1031: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1032-1033: ProgramState.__setitem__ method
```python
    def __setitem__(self, name, value):
        self.set_var(name, value)
```
**EN:** This block uses `ProgramState.__setitem__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.__setitem__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1034-1034: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1035-1036: ProgramState.__contains__ method
```python
    def __contains__(self, name):
        return name in self.stream_executor.variables
```
**EN:** This block uses `ProgramState.__contains__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.__contains__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1037-1037: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1038-1039: ProgramState.__del__ method
```python
    def __del__(self):
        self.stream_executor.end()
```
**EN:** This block uses `ProgramState.__del__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.__del__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1040-1040: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramState` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramState` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1041-1042: ProgramState.__repr__ method
```python
    def __repr__(self) -> str:
        return f"ProgramState({self.text()})"
```
**EN:** This block uses `ProgramState.__repr__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramState.__repr__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1043-1044: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1045-1045: ProgramStateGroup class declaration
```python
class ProgramStateGroup:
```
**EN:** This block declares the `ProgramStateGroup` class, which exists to store configuration or metadata. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `ProgramStateGroup` 类，其职责是存储配置或元数据。它定义了本文件其余部分使用的结构与成员布局。

### Lines 1046-1050: ProgramStateGroup initializer
```python
    def __init__(
        self, states: List[ProgramState], src_state: Optional[ProgramState] = None
    ):
        self.states = states
        self.src_state = src_state
```
**EN:** This block initializes the `ProgramStateGroup` object, wiring together the state and dependencies used by later methods.
**CN:** 该代码块初始化 `ProgramStateGroup` 对象，连接后续方法使用的状态与依赖。

### Lines 1051-1051: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramStateGroup` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramStateGroup` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1052-1076: ProgramStateGroup.join method
```python
    def join(self, mode: str = "gather_variable"):
        if mode == "gather_variable":
            # Copy variables back
            src_vars = self.src_state.stream_executor.variables
            src_var_set = set(src_vars.keys())
            for child_state in self.states:
                child_state.stream_executor.sync()
                child_vars = child_state.stream_executor.variables
                new_vars = set(child_vars.keys()) - src_var_set

                for k in new_vars:
                    if k in src_vars:
                        src_vars[k].append(child_vars[k])
                    else:
                        src_vars[k] = [child_vars[k]]
        elif mode == "concate_and_append":
            # Concatenate and append KV cache
            self.src_state += SglConcateAndAppend(self.states)
            # Need a sync here. Otherwise, `states` can be deleted.
            self.src_state.stream_executor.sync()
        else:
            raise ValueError(f"Invalid join mode: {mode}")

        for s in self.states:
            s.stream_executor.end()
```
**EN:** This block uses `ProgramStateGroup.join` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramStateGroup.join` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1077-1077: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramStateGroup` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramStateGroup` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1078-1079: ProgramStateGroup.__getitem__ method
```python
    def __getitem__(self, i: int):
        return self.states[i]
```
**EN:** This block uses `ProgramStateGroup.__getitem__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramStateGroup.__getitem__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1080-1080: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramStateGroup` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramStateGroup` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1081-1082: ProgramStateGroup.__setitem__ method
```python
    def __setitem__(self, i: int, value):
        assert self.states[i] == value
```
**EN:** This block uses `ProgramStateGroup.__setitem__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramStateGroup.__setitem__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1083-1083: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `ProgramStateGroup` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`ProgramStateGroup` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 1084-1098: ProgramStateGroup.__iadd__ method
```python
    def __iadd__(self, other):
        if isinstance(other, Callable):
            # lambda function
            for i in range(len(self.states)):
                self.states[i] += other(i)
        elif isinstance(other, SglExpr):
            for i in range(len(self.states)):
                self.states[i] += other
        elif isinstance(other, (list, tuple)):
            for i in range(len(self.states)):
                self.states[i] += other[i]
        else:
            raise ValueError(f"Invalid value: {other}")

        return self
```
**EN:** This block uses `ProgramStateGroup.__iadd__` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ProgramStateGroup.__iadd__` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Runtime support code / 运行时支撑代码

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.global_config`
- `sglang.lang.backend.base_backend`
- `sglang.lang.ir`
- `sglang.lang.tracer`
- `sglang.srt.parser.reasoning_parser`
- `sglang.utils`
### External / 外部
- `concurrent`
- `contextvars`
- `multiprocessing`
- `tqdm`
- `asyncio` (stdlib)
- `contextlib` (stdlib)
- `copy` (stdlib)
- `queue` (stdlib)
- `threading` (stdlib)
- `typing` (stdlib)
- `uuid` (stdlib)
- `warnings` (stdlib)
