# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/structured_output/__init__.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `StructuredOutputManager` for the V1 `structured_output` subsystem. / 为 V1 的 `structured_output` 子系统实现 `StructuredOutputManager`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import itertools
import multiprocessing
from collections.abc import Iterable
from concurrent.futures import Future, ThreadPoolExecutor
from typing import TYPE_CHECKING

from vllm.config import VllmConfig
from vllm.logger import init_logger
from vllm.reasoning import ReasoningParserManager
from vllm.tokenizers import cached_tokenizer_from_config
from vllm.utils.import_utils import LazyLoader
from vllm.v1.structured_output.backend_guidance import GuidanceBackend
from vllm.v1.structured_output.backend_types import (
    StructuredOutputBackend,
    StructuredOutputGrammar,
)
from vllm.v1.structured_output.backend_xgrammar import XgrammarBackend

if TYPE_CHECKING:
    import numpy as np
    import numpy.typing as npt
    import torch

    from vllm.reasoning import ReasoningParser
    from vllm.v1.request import Request
else:
    torch = LazyLoader("torch", globals(), "torch")


logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `StructuredOutputManager` class / `StructuredOutputManager` 类
```python
class StructuredOutputManager:
    """Engine-level manager for structured output requests."""
```
**EN:** Introduces the `StructuredOutputManager` class. Core methods include `__init__`, `_get_reasoner`, `grammar_init`, `_create_grammar`, `_fill_bitmasks`, `_async_submit_fill_bitmask`. Docstring signal: Engine-level manager for structured output requests.
**CN:** 这里定义 `StructuredOutputManager` 类。核心方法包括 `__init__`, `_get_reasoner`, `grammar_init`, `_create_grammar`, `_fill_bitmasks`, `_async_submit_fill_bitmask`。

### `StructuredOutputManager.__init__` method / `StructuredOutputManager.__init__` 方法
```python
    def __init__(self, vllm_config: VllmConfig):
        self.backend: StructuredOutputBackend | None = None
        # We only store the class of the reasoner in the manager.
        # The parser instance is request-scoped because some reasoning parsers
        # depend on per-request chat-template kwargs.
        self.reasoner_cls: type[ReasoningParser] | None = None
        self.vllm_config = vllm_config

        # When in external_launcher mode, async grammar compilation causes deadlocks
        # due to external_launcher mode having a scheduler for each TP rank.
        # Async grammar compilation causes the
        # WAITING_FOR_STRUCTURED_OUTPUT_GRAMMAR → WAITING transition to
        # happen at different times on different TP ranks,
        # breaking the determinism assumption that external_launcher relies on.
        self._use_async_grammar_compilation = (
            vllm_config.parallel_config.distributed_executor_backend
            != "external_launcher"
        )

        self._grammar_bitmask: torch.Tensor | None = None
        self._full_mask = torch.tensor(-1, dtype=torch.int32)

        max_batch_size = self.vllm_config.scheduler_config.max_num_seqs
        self.fill_bitmask_parallel_threshold = 128
        if self.fill_bitmask_parallel_threshold < max_batch_size:
            self.fill_bitmask_parallel_batch_size = 16
            # Use:
            # - at least 1 CPU
            # - at most half the number of CPUs or 8, whichever is less
            max_workers = max(1, min(multiprocessing.cpu_count() // 2, 8))
            self.executor_for_fillmask = ThreadPoolExecutor(max_workers=max_workers)

        if not self.vllm_config.model_config.skip_tokenizer_init:
            # The default max_workers if not specified is the number of
            # CPUs * 5, which is way too high since these tasks are CPU-bound,
            # not I/O bound. We also know we would never dominate CPU usage
            # with just grammar compilation, so we set it to half the number
            # of CPUs.
            max_workers = max(1, (multiprocessing.cpu_count() + 1) // 2)
            self.executor = ThreadPoolExecutor(max_workers=max_workers)
            self.tokenizer = cached_tokenizer_from_config(
                model_config=self.vllm_config.model_config
            )
            reasoning_parser_plugin = (
                self.vllm_config.structured_outputs_config.reasoning_parser_plugin
            )
            if reasoning_parser_plugin and len(reasoning_parser_plugin) > 3:
                ReasoningParserManager.import_reasoning_parser(reasoning_parser_plugin)

            reasoning_parser = (
                self.vllm_config.structured_outputs_config.reasoning_parser
            )
            if reasoning_parser:
                self.reasoner_cls = ReasoningParserManager.get_reasoning_parser(
                    reasoning_parser
                )

        self.enable_in_reasoning = (
            self.vllm_config.structured_outputs_config.enable_in_reasoning
        )
```
**EN:** This method initializes the object state within `StructuredOutputManager`. Key calls include `tensor`, `max`, `ThreadPoolExecutor`, `cached_tokenizer_from_config`, `min`, `import_reasoning_parser`. It touches state such as `backend`, `reasoner_cls`, `vllm_config`, `_use_async_grammar_compilation`, `_grammar_bitmask`, `_full_mask`, `fill_bitmask_parallel_threshold`, `enable_in_reasoning`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`StructuredOutputManager`。 关键调用包括 `tensor`, `max`, `ThreadPoolExecutor`, `cached_tokenizer_from_config`, `min`, `import_reasoning_parser`。 它会读写 `backend`, `reasoner_cls`, `vllm_config`, `_use_async_grammar_compilation`, `_grammar_bitmask`, `_full_mask`, `fill_bitmask_parallel_threshold`, `enable_in_reasoning` 等状态。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `StructuredOutputManager._get_reasoner` method / `StructuredOutputManager._get_reasoner` 方法
```python
    def _get_reasoner(self, request: "Request") -> "ReasoningParser | None":
        structured_req = request.structured_output_request
        if structured_req is None or self.reasoner_cls is None:
            return None

        if structured_req.reasoner is None:
            # Lazily build the request-local parser so the structured-output
            # gate observes the same template kwargs used by the frontend.
            parser_kwargs = structured_req.reasoning_parser_kwargs or {}
            structured_req.reasoner = self.reasoner_cls(
                tokenizer=self.tokenizer,
                **parser_kwargs,
            )
        return structured_req.reasoner
```
**EN:** This method implements `_get_reasoner` within `StructuredOutputManager`. Key calls include `reasoner_cls`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_get_reasoner`，其作用域位于`StructuredOutputManager`。 关键调用包括 `reasoner_cls`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `StructuredOutputManager.grammar_init` method / `StructuredOutputManager.grammar_init` 方法
```python
    def grammar_init(self, request: "Request") -> None:
        if request.structured_output_request is None:
            return

        if TYPE_CHECKING:
            assert (
                request.sampling_params is not None
                and request.sampling_params.structured_outputs is not None
            )

        # Initialize the backend the first time it is needed.
        #
        # NOTE: We only support a single backend. We do NOT support different
        # backends on a per-request basis in V1 (for now, anyway...).
        # _backend is set in Processor._validate_structured_output
        if self.backend is None:
            assert request.sampling_params is not None
            backend = request.sampling_params.structured_outputs._backend
            vocab_size = self.vllm_config.model_config.get_vocab_size()
            if backend == "xgrammar":
                self.backend = XgrammarBackend(
                    self.vllm_config,
                    tokenizer=self.tokenizer,
                    vocab_size=vocab_size,
                )
            elif backend == "guidance":
                self.backend = GuidanceBackend(
                    self.vllm_config,
                    tokenizer=self.tokenizer,
                    vocab_size=vocab_size,
                )
            elif backend == "outlines":
                from vllm.v1.structured_output.backend_outlines import OutlinesBackend

                self.backend = OutlinesBackend(
                    self.vllm_config,
                    tokenizer=self.tokenizer,
                    vocab_size=vocab_size,
                )
            elif backend == "lm-format-enforcer":
                from vllm.v1.structured_output.backend_lm_format_enforcer import (  # noqa: E501
                    LMFormatEnforcerBackend,
                )

                self.backend = LMFormatEnforcerBackend(
                    self.vllm_config,
                    tokenizer=self.tokenizer,
                    vocab_size=vocab_size,
                )
            else:
                raise ValueError(f"Unsupported structured output backend: {backend}")

        if self._use_async_grammar_compilation:
            grammar = self.executor.submit(self._create_grammar, request)
        else:
            grammar = self._create_grammar(request)  # type: ignore[assignment]
        request.structured_output_request.grammar = grammar  # type: ignore[assignment]
```
**EN:** This method implements `grammar_init` within `StructuredOutputManager`. Key calls include `get_vocab_size`, `submit`, `_create_grammar`, `XgrammarBackend`, `GuidanceBackend`, `OutlinesBackend`. It touches state such as `backend`. The control flow contains 8 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `grammar_init`，其作用域位于`StructuredOutputManager`。 关键调用包括 `get_vocab_size`, `submit`, `_create_grammar`, `XgrammarBackend`, `GuidanceBackend`, `OutlinesBackend`。 它会读写 `backend` 等状态。 控制流包含 8 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `StructuredOutputManager._create_grammar` method / `StructuredOutputManager._create_grammar` 方法
```python
    def _create_grammar(self, request: "Request") -> StructuredOutputGrammar:
        key = request.structured_output_request.structured_output_key  # type: ignore[union-attr]

        # Note that the request was validated in the engine core client,
        # so at this point we know it is a supported type of request.
        #
        # TODO: we still need to handle xgrammar compilation failures,
        # though it should be unlikely as we test that up front as well.
        request_type, grammar_spec = key

        assert self.backend is not None
        return self.backend.compile_grammar(request_type, grammar_spec)
```
**EN:** This method implements `_create_grammar` within `StructuredOutputManager`. Key calls include `compile_grammar`.
**CN:** 该方法会实现 `_create_grammar`，其作用域位于`StructuredOutputManager`。 关键调用包括 `compile_grammar`。

### `StructuredOutputManager.grammar_bitmask` method / `StructuredOutputManager.grammar_bitmask` 方法
```python
    def grammar_bitmask(
        self,
        requests: dict[str, "Request"],
        structured_output_request_ids: list[str],
        scheduled_spec_decode_tokens: dict[str, list[int]],
    ) -> "npt.NDArray[np.int32] | None":
        # Prepare the structured output bitmask for this batch.
        if not structured_output_request_ids:
            return None

        max_num_spec_tokens = 0
        if self.vllm_config.speculative_config is not None:
            max_num_spec_tokens = (
                self.vllm_config.speculative_config.num_speculative_tokens
            )

        if self._grammar_bitmask is None:
            assert self.backend is not None
            max_batch_size = self.vllm_config.scheduler_config.max_num_seqs

            # Allocate a bitmask for each token needing to be checked:
            # one for each speculative position, and one more for the
            # bonus token / non-speculative token.
            self._grammar_bitmask = self.backend.allocate_token_bitmask(
                max_batch_size * (1 + max_num_spec_tokens)
            )

        # Generate a batched bitmask for all structured output requests.
        # When speculative decoding is enabled, we need to include multiple
        # masks for each request, one for each possible bonus token position.
        # These are stored inline in the tensor and unpacked by the gpu runner.
        cumulative_index = 0

        # Optimized parallel filling of bitmasks for
        # non-spec, large-batch-size cases
        if (
            len(structured_output_request_ids) > self.fill_bitmask_parallel_threshold
            and max_num_spec_tokens == 0
        ):
            promises = []
            batch = []
            for req_id in structured_output_request_ids:
                request = requests[req_id]
                structured_output_request = request.structured_output_request
                if TYPE_CHECKING:
                    assert structured_output_request is not None
                    assert structured_output_request.grammar is not None
                grammar = structured_output_request.grammar

                apply_bitmask = self.should_fill_bitmask(request)
    # ... omitted for brevity ...
                apply_bitmask = self.should_fill_bitmask(request)

                state_advancements = 0
                req_tokens = scheduled_spec_decode_tokens.get(req_id, ())
                for token in itertools.chain(req_tokens, (-1,)):
                    self._fill_bitmasks(((grammar, cumulative_index, apply_bitmask),))
                    if token == -1:
                        # Stop advancing the grammar once we hit a padding token.
                        apply_bitmask = False
                    if apply_bitmask and not grammar.is_terminated():
                        accepted = grammar.accept_tokens(req_id, [token])
                        assert accepted, (token, req_id, scheduled_spec_decode_tokens)
                        state_advancements += 1
                    cumulative_index += 1
                if state_advancements > 0:
                    grammar.rollback(state_advancements)

        bitmask_tensor = self._grammar_bitmask
        if cumulative_index < bitmask_tensor.shape[0]:
            bitmask_tensor = bitmask_tensor[:cumulative_index]

        # After finishing with the xgrammar operations, we convert to
        # np.ndarray, because that is much more efficient for serialization
        # and deserialization when sending this to the GPU workers.
        return bitmask_tensor.numpy()
```
**EN:** This method implements `grammar_bitmask` within `StructuredOutputManager`. Key calls include `numpy`, `allocate_token_bitmask`, `len`, `should_fill_bitmask`, `append`, `result`. It touches state such as `_grammar_bitmask`. The control flow contains 12 branch(es) and 4 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `grammar_bitmask`，其作用域位于`StructuredOutputManager`。 关键调用包括 `numpy`, `allocate_token_bitmask`, `len`, `should_fill_bitmask`, `append`, `result`。 它会读写 `_grammar_bitmask` 等状态。 控制流包含 12 个分支和 4 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `StructuredOutputManager.should_fill_bitmask` method / `StructuredOutputManager.should_fill_bitmask` 方法
```python
    def should_fill_bitmask(self, request: "Request") -> bool:
        # NOTE (Hanchen) if enable_in_reasoning is True, it means that
        # the model needs to be constrained in reasoning. So we should always
        # enable the bitmask filling.
        reasoner = self._get_reasoner(request)
        if reasoner is not None:
            if self.enable_in_reasoning:
                return True
            assert request.structured_output_request is not None
            if request.structured_output_request.reasoning_ended is None:
                # This should be removed here, but since `openai_gptoss`
                # is an independent code path, it is kept for now.
                # After unifying the `openai_gptoss` and non-`openai_gptoss` styles,
                # it can be removed.
                request.structured_output_request.reasoning_ended = (
                    reasoner.is_reasoning_end(request.prompt_token_ids or [])
                )
            return request.structured_output_request.reasoning_ended
        return True
```
**EN:** This method implements `should_fill_bitmask` within `StructuredOutputManager`. Key calls include `_get_reasoner`, `is_reasoning_end`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `should_fill_bitmask`，其作用域位于`StructuredOutputManager`。 关键调用包括 `_get_reasoner`, `is_reasoning_end`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `StructuredOutputManager.should_advance` method / `StructuredOutputManager.should_advance` 方法
```python
    def should_advance(self, request: "Request") -> bool:
        if not request.use_structured_output:
            return False

        # To determine whether we can advance the FSM.
        # Supports thinking usage where we skip the reasoning components.
        if TYPE_CHECKING:
            assert request.structured_output_request is not None
            assert request.structured_output_request.grammar is not None
        # by default, we should always advance
        # for cases that don't use thinking mode.
        reasoner = self._get_reasoner(request)
        if reasoner is None:
            return True

        # if the model needs structured in reasoning, we should advance
        if self.enable_in_reasoning:
            return True

        structured_req = request.structured_output_request
        if structured_req.reasoning_ended:
            return True

        # Check if reasoning ends in *this* step
        delta_from = request.num_computed_tokens - request.num_output_placeholders
        all_token_ids = request.all_token_ids
        start = (
            delta_from if delta_from >= 0 else max(len(all_token_ids) + delta_from, 0)
        )
        if reasoner.is_reasoning_end_streaming(
            all_token_ids, itertools.islice(all_token_ids, start, None)
        ):
            # Reasoning just ended, so we shouldn't advance til
            # next pass
            structured_req.reasoning_ended = True

        return False
```
**EN:** This method implements `should_advance` within `StructuredOutputManager`. Key calls include `_get_reasoner`, `is_reasoning_end_streaming`, `max`, `islice`, `len`. The control flow contains 7 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `should_advance`，其作用域位于`StructuredOutputManager`。 关键调用包括 `_get_reasoner`, `is_reasoning_end_streaming`, `max`, `islice`, `len`。 控制流包含 7 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `StructuredOutputManager.clear_backend` method / `StructuredOutputManager.clear_backend` 方法
```python
    def clear_backend(self) -> None:
        if self.backend is not None:
            self.backend.destroy()
```
**EN:** This method implements `clear_backend` within `StructuredOutputManager`. Key calls include `destroy`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `clear_backend`，其作用域位于`StructuredOutputManager`。 关键调用包括 `destroy`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `StructuredOutputManager`: central class or interface in this module. / `StructuredOutputManager`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `itertools`, `multiprocessing`, `collections`, `concurrent`, `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.logger`, `vllm.reasoning`, `vllm.tokenizers`, `vllm.utils.import_utils`, `vllm.v1.structured_output.backend_guidance`, `vllm.v1.structured_output.backend_types`, `vllm.v1.structured_output.backend_xgrammar`, `vllm.v1.request`, `vllm.v1.structured_output.backend_outlines`, `vllm.v1.structured_output.backend_lm_format_enforcer`
