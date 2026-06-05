# perf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/metrics/perf.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Analytic flops/memory estimation module for transformer components, to help derive MFU (Model Flops Utilization) stats for a running model. / 该模块位于 `metrics` 子系统，主要围绕 `InvalidComponent`, `DebugPerfStats`, `PerfStats` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""
Analytic flops/memory estimation module for transformer components,
to help derive MFU (Model Flops Utilization) stats for a running model.
"""

import json
import time
from abc import ABC, abstractmethod
from collections.abc import Iterable
from dataclasses import asdict, dataclass
from typing import Any, Protocol

import prometheus_client
import torch
from pydantic import BaseModel, Field, ValidationError, model_validator
from typing_extensions import Self

import vllm.envs as envs
from vllm.config import VllmConfig
from vllm.logger import init_logger
from vllm.utils.torch_utils import (
    STR_DTYPE_TO_TORCH_DTYPE,
    get_dtype_size,
    get_kv_cache_torch_dtype,
)
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.metrics.utils import create_metric_per_engine

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `InvalidComponent` class / `InvalidComponent` 类
```python
class InvalidComponent(Exception):
    """
    Custom exception to indicate that a certain ComponentMetric is not
    applicable to the given VllmConfig.
    """

    pass
```
**EN:** Introduces the `InvalidComponent` class on top of `Exception`. Core methods include its methods defined below. Docstring signal: Custom exception to indicate that a certain ComponentMetric is not applicable to the given VllmConfig.
**CN:** 这里定义 `InvalidComponent` 类，其基类包括 `Exception`。核心方法包括 下方定义的方法。

### Module constants / 模块常量
```python
_QUANT_WEIGHT_BYTE_SIZE: dict[str, float] = {
    # FP8 methods (1 byte per weight)
    "fp8": 1,
    "fbgemm_fp8": 1,
    "ptpc_fp8": 1,
    "fp_quant": 1,
    "modelopt": 1,
    "modelopt_mxfp8": 1,
    # FP4 / INT4 methods (0.5 bytes per weight)
    "mxfp4": 0.5,
    "awq": 0.5,
    "awq_marlin": 0.5,
    "gptq": 0.5,
    "gptq_marlin": 0.5,
    "bitsandbytes": 0.5,
    "modelopt_fp4": 0.5,
    "petit_nvfp4": 0.5,
    "gguf": 0.5,
    "compressed-tensors": 0.5,
    "torchao": 0.5,
    "quark": 0.5,
    "moe_wna16": 0.5,
    "inc": 0.5,
    "cpu_awq": 0.5,
    "experts_int8": 1,
}
```
**EN:** Defines module-level constants or aliases such as `_QUANT_WEIGHT_BYTE_SIZE`, which are reused by later definitions.
**CN:** 定义 `_QUANT_WEIGHT_BYTE_SIZE` 等模块级常量或别名，供后续定义复用。

### `DebugPerfStats` class / `DebugPerfStats` 类
```python
@dataclass
class DebugPerfStats:
    ## Stats for debugging the metrics calculation
    calc_duration: float = 0.0  # time spent calculating these stats
    num_prefill_requests: int = 0
    num_decode_requests: int = 0
    context_breakdown: dict[str, int] | None = None
    num_flops_per_gpu_breakdown: dict[str, int] | None = None
    num_read_bytes_per_gpu_breakdown: dict[str, int] | None = None
    num_write_bytes_per_gpu_breakdown: dict[str, int] | None = None
```
**EN:** Uses `@dataclass` to package related state for `DebugPerfStats`. Typical fields include `calc_duration`, `num_prefill_requests`, `num_decode_requests`, `context_breakdown`, `num_flops_per_gpu_breakdown`, `num_read_bytes_per_gpu_breakdown`.
**CN:** `DebugPerfStats` 使用 `@dataclass` 打包相关状态。典型字段包括 `calc_duration`, `num_prefill_requests`, `num_decode_requests`, `context_breakdown`, `num_flops_per_gpu_breakdown`, `num_read_bytes_per_gpu_breakdown`。

### `PerfStats` class / `PerfStats` 类
```python
@dataclass
class PerfStats:
    num_flops_per_gpu: int = 0
    num_read_bytes_per_gpu: int = 0
    num_write_bytes_per_gpu: int = 0
    debug_stats: DebugPerfStats | None = None
```
**EN:** Uses `@dataclass` to package related state for `PerfStats`. Typical fields include `num_flops_per_gpu`, `num_read_bytes_per_gpu`, `num_write_bytes_per_gpu`, `debug_stats`.
**CN:** `PerfStats` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_flops_per_gpu`, `num_read_bytes_per_gpu`, `num_write_bytes_per_gpu`, `debug_stats`。

### `ExecutionContext` class / `ExecutionContext` 类
```python
@dataclass
class ExecutionContext:
    """
    Represents an execution context for a batch of requests.

    This class aggregates statistics across multiple requests in a batch,
    separately tracking prefill and decode phases.

    Example)
    - Batch with one full prefill (2048 tokens) and one decode (1 token, 8192 context):
      ctx = ExecutionContext()
      ctx.add(2048, 2048, is_prefill=True)
      ctx.add(1, 8192, is_prefill=False)
    """

    # Prefill phase statistics
    num_prefill_requests: int = 0
    prefill_num_tokens: int = 0  # sum of num_tokens for prefill requests
    prefill_context_len: int = 0  # sum of context_len for prefill requests
    prefill_token_context_product: int = 0  # sum of (num_tokens * context_len)

    # Decode phase statistics
    num_decode_requests: int = 0
    decode_num_tokens: int = 0  # sum of num_tokens for decode requests
    decode_context_len: int = 0  # sum of context_len for decode requests
    decode_token_context_product: int = 0  # sum of (num_tokens * context_len)
```
**EN:** Uses `@dataclass` to package related state for `ExecutionContext`. Typical fields include `num_prefill_requests`, `prefill_num_tokens`, `prefill_context_len`, `prefill_token_context_product`, `num_decode_requests`, `decode_num_tokens`.
**CN:** `ExecutionContext` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_prefill_requests`, `prefill_num_tokens`, `prefill_context_len`, `prefill_token_context_product`, `num_decode_requests`, `decode_num_tokens`。

### `ExecutionContext.add` method / `ExecutionContext.add` 方法
```python
    def add(self, num_tokens: int, context_len: int, is_prefill: bool) -> None:
        """Add a single request's statistics to this batch context."""
        if is_prefill:
            self.num_prefill_requests += 1
            self.prefill_num_tokens += num_tokens
            self.prefill_context_len += context_len
            self.prefill_token_context_product += num_tokens * context_len
        else:
            self.num_decode_requests += 1
            self.decode_num_tokens += num_tokens
            self.decode_context_len += context_len
            self.decode_token_context_product += num_tokens * context_len
```
**EN:** This method implements `add` within `ExecutionContext`. The docstring frames it as: Add a single request's statistics to this batch context. It touches state such as `num_prefill_requests`, `prefill_num_tokens`, `prefill_context_len`, `prefill_token_context_product`, `num_decode_requests`, `decode_num_tokens`, `decode_context_len`, `decode_token_context_product`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `add`，其作用域位于`ExecutionContext`。 它会读写 `num_prefill_requests`, `prefill_num_tokens`, `prefill_context_len`, `prefill_token_context_product`, `num_decode_requests`, `decode_num_tokens`, `decode_context_len`, `decode_token_context_product` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ExecutionContext.total_num_tokens` method / `ExecutionContext.total_num_tokens` 方法
```python
    def total_num_tokens(self) -> int:
        """Total number of tokens across all requests in the batch."""
        return self.prefill_num_tokens + self.decode_num_tokens
```
**EN:** This method implements `total_num_tokens` within `ExecutionContext`. The docstring frames it as: Total number of tokens across all requests in the batch.
**CN:** 该方法会实现 `total_num_tokens`，其作用域位于`ExecutionContext`。

### `ExecutionContext.num_logits_tokens` method / `ExecutionContext.num_logits_tokens` 方法
```python
    def num_logits_tokens(self) -> int:
        """Number of tokens that require logits computation (unembedding).

        For prefill, only the last token per request needs logits.
        For decode, all tokens need logits.
        """
        return self.num_prefill_requests + self.decode_num_tokens
```
**EN:** This method implements `num_logits_tokens` within `ExecutionContext`. The docstring frames it as: Number of tokens that require logits computation (unembedding).
**CN:** 该方法会实现 `num_logits_tokens`，其作用域位于`ExecutionContext`。

### `ExecutionContext.from_single_request` method / `ExecutionContext.from_single_request` 方法
```python
    @classmethod
    def from_single_request(
        cls, num_tokens: int, context_len: int, is_prefill: bool
    ) -> "ExecutionContext":
        """Create an ExecutionContext from a single request.

        This is a convenience method primarily for testing.
        """
        ctx = cls()
        ctx.add(num_tokens, context_len, is_prefill)
        return ctx
```
**EN:** This method reconstructs data from another representation within `ExecutionContext`. The docstring frames it as: Create an ExecutionContext from a single request. Key calls include `cls`, `add`.
**CN:** 该方法会从另一种表示恢复数据，其作用域位于`ExecutionContext`。 关键调用包括 `cls`, `add`。

### `ParsedArgs` class / `ParsedArgs` 类
```python
class ParsedArgs:
    """
    Syntactic sugar so that Parsers can use dot notations
    to access/update the parsed arguments.

    e.g.)
        args = ParsedArgs()
        args.x = 3
        args.y = args.x + 1
    """
```
**EN:** Introduces the `ParsedArgs` class. Core methods include `__getattr__`, `__setattr__`, `model_dump`. Docstring signal: Syntactic sugar so that Parsers can use dot notations to access/update the parsed arguments.
**CN:** 这里定义 `ParsedArgs` 类。核心方法包括 `__getattr__`, `__setattr__`, `model_dump`。

### `ParsedArgs.__getattr__` method / `ParsedArgs.__getattr__` 方法
```python
    def __getattr__(self, name: str) -> Any:
        raise AttributeError(f"'{type(self).__name__}' has no attribute '{name}'")
```
**EN:** This method implements `__getattr__` within `ParsedArgs`. Key calls include `AttributeError`, `type`.
**CN:** 该方法会实现 `__getattr__`，其作用域位于`ParsedArgs`。 关键调用包括 `AttributeError`, `type`。

### `ParsedArgs.__setattr__` method / `ParsedArgs.__setattr__` 方法
```python
    def __setattr__(self, name: str, value: Any) -> None:
        object.__setattr__(self, name, value)
```
**EN:** This method implements `__setattr__` within `ParsedArgs`. Key calls include `__setattr__`.
**CN:** 该方法会实现 `__setattr__`，其作用域位于`ParsedArgs`。 关键调用包括 `__setattr__`。

### `ParsedArgs.model_dump` method / `ParsedArgs.model_dump` 方法
```python
    def model_dump(self) -> dict[str, Any]:
        return vars(self).copy()
```
**EN:** This method implements `model_dump` within `ParsedArgs`. Key calls include `copy`, `vars`.
**CN:** 该方法会实现 `model_dump`，其作用域位于`ParsedArgs`。 关键调用包括 `copy`, `vars`。

### `Parser` class / `Parser` 类
```python
class Parser(Protocol):
```
**EN:** Declares the `Parser` interface. Downstream implementations are expected to provide methods such as `parse`.
**CN:** `Parser` 声明了一组接口约定。下游实现需要提供 `parse` 等方法。

### `Parser.parse` method / `Parser.parse` 方法
```python
    def parse(self, args: ParsedArgs, vllm_config: VllmConfig) -> ParsedArgs:
        """
        Parse the vllm config and update the current ParsedArgs and pass it on.
        If the parser isn't applicable to the vllm_config, it will do nothing.
        """
        ...
```
**EN:** This method implements `parse` within `Parser`. The docstring frames it as: Parse the vllm config and update the current ParsedArgs and pass it on.
**CN:** 该方法会实现 `parse`，其作用域位于`Parser`。

### `ParserChain` class / `ParserChain` 类
```python
class ParserChain:
    """
    Applies chain of parser in a sequential order.
    Later parsers might overwrite results from previous parsers,
    so parsers should be chained in the appropriate order if they
    are not mutually exclusive.
    """
```
**EN:** Introduces the `ParserChain` class. Core methods include `__init__`, `add_parser`, `parse`. Docstring signal: Applies chain of parser in a sequential order.
**CN:** 这里定义 `ParserChain` 类。核心方法包括 `__init__`, `add_parser`, `parse`。

### `ParserChain.__init__` method / `ParserChain.__init__` 方法
```python
    def __init__(self, *parsers: Parser) -> None:
        self.parsers = list(parsers)
```
**EN:** This method initializes the object state within `ParserChain`. Key calls include `list`. It touches state such as `parsers`.
**CN:** 该方法会初始化对象状态，其作用域位于`ParserChain`。 关键调用包括 `list`。 它会读写 `parsers` 等状态。

### `ParserChain.add_parser` method / `ParserChain.add_parser` 方法
```python
    def add_parser(self, parser: Parser) -> None:
        self.parsers.append(parser)
```
**EN:** This method implements `add_parser` within `ParserChain`. Key calls include `append`.
**CN:** 该方法会实现 `add_parser`，其作用域位于`ParserChain`。 关键调用包括 `append`。

### `ParserChain.parse` method / `ParserChain.parse` 方法
```python
    def parse(self, vllm_config: VllmConfig) -> ParsedArgs:
        args = ParsedArgs()
        for parser in self.parsers:
            args = parser.parse(args, vllm_config)
        return args
```
**EN:** This method implements `parse` within `ParserChain`. Key calls include `ParsedArgs`, `parse`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `parse`，其作用域位于`ParserChain`。 关键调用包括 `ParsedArgs`, `parse`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### Module constants / 模块常量
```python
_COMPONENT_METRICS_REGISTRY: dict[str, type["ComponentMetrics"]] = {}
```
**EN:** Defines module-level constants or aliases such as `_COMPONENT_METRICS_REGISTRY`, which are reused by later definitions.
**CN:** 定义 `_COMPONENT_METRICS_REGISTRY` 等模块级常量或别名，供后续定义复用。

### `ComponentMetrics` class / `ComponentMetrics` 类
```python
class ComponentMetrics(BaseModel, ABC):
    """
    Each concrete ComponentMetrics class is associated with:
    - fields that are required for metric derivation
      (fields are specified/validated through pydantic model)
    - parser to parse VllmConfig into fields
    - metric methods that derive flops/bytes for a given execution context
    """
```
**EN:** Declares the `ComponentMetrics` interface. Downstream implementations are expected to provide methods such as `component_type`, `get_parser`, `__init_subclass__`, `from_vllm_config`, `registered_metrics`, `get_num_flops_breakdown`.
**CN:** `ComponentMetrics` 声明了一组接口约定。下游实现需要提供 `component_type`, `get_parser`, `__init_subclass__`, `from_vllm_config`, `registered_metrics`, `get_num_flops_breakdown` 等方法。

### `ComponentMetrics.get_parser` method / `ComponentMetrics.get_parser` 方法
```python
    @classmethod
    @abstractmethod
    def get_parser(cls) -> ParserChain:
        """
        Return a ParserChain that provides values for all required fields.
        The returned parser chain must populate ParsedArgs with values for every
        field defined on this ComponentMetrics class. Missing fields will cause
        a ValidationError when from_vllm_config() is called.
        See individual Parser docstrings for which args they provide, and field
        comments on ComponentMetrics subclasses for which parser provides each field.
        """
        ...
```
**EN:** This method returns or derives a value within `ComponentMetrics`. The docstring frames it as: Return a ParserChain that provides values for all required fields.
**CN:** 该方法会返回或推导一个值，其作用域位于`ComponentMetrics`。

### `ComponentMetrics.from_vllm_config` method / `ComponentMetrics.from_vllm_config` 方法
```python
    @classmethod
    def from_vllm_config(cls, vllm_config: VllmConfig) -> Self:
        """
        Instantiate this class from VllmConfig.
        Raises ValidationError if parsing fails.
        """

        parser = cls.get_parser()
        parsed_args = parser.parse(vllm_config)
        try:
            return cls.model_validate(parsed_args.model_dump())
        except ValidationError as e:
            raise InvalidComponent(f"Invalid {cls.component_type()} config: {e}") from e
```
**EN:** This method reconstructs data from another representation within `ComponentMetrics`. The docstring frames it as: Instantiate this class from VllmConfig. Key calls include `get_parser`, `parse`, `model_validate`, `model_dump`, `InvalidComponent`, `component_type`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会从另一种表示恢复数据，其作用域位于`ComponentMetrics`。 关键调用包括 `get_parser`, `parse`, `model_validate`, `model_dump`, `InvalidComponent`, `component_type`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ComponentMetrics.get_num_flops_breakdown` method / `ComponentMetrics.get_num_flops_breakdown` 方法
```python
    @abstractmethod
    def get_num_flops_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]: ...
```
**EN:** This method returns or derives a value within `ComponentMetrics`.
**CN:** 该方法会返回或推导一个值，其作用域位于`ComponentMetrics`。

### `ComponentMetrics.get_read_bytes_breakdown` method / `ComponentMetrics.get_read_bytes_breakdown` 方法
```python
    @abstractmethod
    def get_read_bytes_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]: ...
```
**EN:** This method returns or derives a value within `ComponentMetrics`.
**CN:** 该方法会返回或推导一个值，其作用域位于`ComponentMetrics`。

### `BaseConfigParser` class / `BaseConfigParser` 类
```python
class BaseConfigParser(Parser):
    """
    Parses base model configuration.
    Provides: vocab_size, hidden_size, num_attention_heads, num_hidden_layers,
    weight_byte_size, activation_byte_size, dp_size, tp_size, pp_size, enable_ep
    """
```
**EN:** Introduces the `BaseConfigParser` class on top of `Parser`. Core methods include `parse`. Docstring signal: Parses base model configuration.
**CN:** 这里定义 `BaseConfigParser` 类，其基类包括 `Parser`。核心方法包括 `parse`。

### `BaseConfigParser.parse` method / `BaseConfigParser.parse` 方法
```python
    def parse(self, args: ParsedArgs, vllm_config: VllmConfig) -> ParsedArgs:
        model_config = vllm_config.model_config

        args.vocab_size = model_config.get_vocab_size()
        args.hidden_size = model_config.get_hidden_size()
        # NOTE: model_config.get_attention_heads() divide by TP
        # so we access field manually here to get total num_heads
        args.num_attention_heads = get_required(
            model_config.hf_text_config, "num_attention_heads"
        )
        args.num_hidden_layers = get_required(
            model_config.hf_text_config, "num_hidden_layers"
        )

        model_dtype = vllm_config.model_config.dtype

        if isinstance(model_dtype, torch.dtype):
            torch_dtype = model_dtype
        elif isinstance(model_dtype, str) and model_dtype in STR_DTYPE_TO_TORCH_DTYPE:
            torch_dtype = STR_DTYPE_TO_TORCH_DTYPE[model_dtype]
        else:
            # FIXME: handle this better
            logger.warning(
                "Unknown model_dtype %s, defaulting to bfloat16",
                model_dtype,
            )
            torch_dtype = torch.bfloat16

        args.weight_byte_size = get_dtype_size(torch_dtype)

        # FIXME: handle this better by parsing whether activations use
        # bf16, fp32, etc...
        args.activation_byte_size = 2

        args.dp_size = vllm_config.parallel_config.data_parallel_size
        args.tp_size = vllm_config.parallel_config.tensor_parallel_size
        args.pp_size = vllm_config.parallel_config.pipeline_parallel_size
        args.enable_ep = vllm_config.parallel_config.enable_expert_parallel

        return args
```
**EN:** This method implements `parse` within `BaseConfigParser`. Key calls include `get_vocab_size`, `get_hidden_size`, `get_required`, `isinstance`, `get_dtype_size`, `warning`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `parse`，其作用域位于`BaseConfigParser`。 关键调用包括 `get_vocab_size`, `get_hidden_size`, `get_required`, `isinstance`, `get_dtype_size`, `warning`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `BaseAttentionConfigParser` class / `BaseAttentionConfigParser` 类
```python
class BaseAttentionConfigParser(Parser):
    """
    Parses attention-specific configuration.
    Provides: num_key_value_heads, head_dim, cache_byte_size
    """
```
**EN:** Introduces the `BaseAttentionConfigParser` class on top of `Parser`. Core methods include `parse`. Docstring signal: Parses attention-specific configuration.
**CN:** 这里定义 `BaseAttentionConfigParser` 类，其基类包括 `Parser`。核心方法包括 `parse`。

### `BaseAttentionConfigParser.parse` method / `BaseAttentionConfigParser.parse` 方法
```python
    def parse(self, args: ParsedArgs, vllm_config: VllmConfig) -> ParsedArgs:
        model_config = vllm_config.model_config

        args.num_key_value_heads = model_config.get_total_num_kv_heads()
        args.head_dim = model_config.get_head_size()

        model_dtype = vllm_config.model_config.dtype
        cache_dtype = vllm_config.cache_config.cache_dtype

        kv_cache_torch_dtype = get_kv_cache_torch_dtype(cache_dtype, model_dtype)
        args.cache_byte_size = get_dtype_size(kv_cache_torch_dtype)

        return args
```
**EN:** This method implements `parse` within `BaseAttentionConfigParser`. Key calls include `get_total_num_kv_heads`, `get_head_size`, `get_kv_cache_torch_dtype`, `get_dtype_size`.
**CN:** 该方法会实现 `parse`，其作用域位于`BaseAttentionConfigParser`。 关键调用包括 `get_total_num_kv_heads`, `get_head_size`, `get_kv_cache_torch_dtype`, `get_dtype_size`。

### `AttentionQuantizationConfigParser` class / `AttentionQuantizationConfigParser` 类
```python
class AttentionQuantizationConfigParser(Parser):
    """
    Parses quantization configuration for attention layers.
    Overrides: weight_byte_size
    """
```
**EN:** Introduces the `AttentionQuantizationConfigParser` class on top of `Parser`. Core methods include `parse`. Docstring signal: Parses quantization configuration for attention layers.
**CN:** 这里定义 `AttentionQuantizationConfigParser` 类，其基类包括 `Parser`。核心方法包括 `parse`。

### `AttentionQuantizationConfigParser.parse` method / `AttentionQuantizationConfigParser.parse` 方法
```python
    def parse(self, args: ParsedArgs, vllm_config: VllmConfig) -> ParsedArgs:
        cfg = vllm_config.quant_config

        if cfg is None:
            return args

        quant_method = cfg.get_name()
        if quant_method in _QUANT_WEIGHT_BYTE_SIZE:
            args.weight_byte_size = _QUANT_WEIGHT_BYTE_SIZE[quant_method]
        else:
            raise InvalidComponent(
                f"Unsupported quantization method for attention metrics: {quant_method}"
            )

        return args
```
**EN:** This method implements `parse` within `AttentionQuantizationConfigParser`. Key calls include `get_name`, `InvalidComponent`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `parse`，其作用域位于`AttentionQuantizationConfigParser`。 关键调用包括 `get_name`, `InvalidComponent`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AttentionMetrics` class / `AttentionMetrics` 类
```python
class AttentionMetrics(ComponentMetrics):
    # From BaseConfigParser
    num_hidden_layers: int = Field(..., gt=0)
    hidden_size: int = Field(..., gt=0)
    num_attention_heads: int = Field(..., gt=0)
    activation_byte_size: int = Field(..., gt=0)
    tp_size: int = Field(..., gt=0)
    pp_size: int = Field(..., gt=0)

    # From BaseAttentionConfigParser
    num_key_value_heads: int = Field(..., gt=0)
    head_dim: int = Field(..., gt=0)
    cache_byte_size: int = Field(..., gt=0)

    # From BaseConfig Parser, overridden by AttentionQuantizationConfigParser
    weight_byte_size: int | float = Field(..., gt=0)

    # TODO: discern cases where we have mixture of different attention layer types
    # such as SWA, MLA, etc.
```
**EN:** Introduces the `AttentionMetrics` class on top of `ComponentMetrics`. Core methods include `component_type`, `get_parser`, `get_num_flops_breakdown`, `get_read_bytes_breakdown`, `get_write_bytes_breakdown`.
**CN:** 这里定义 `AttentionMetrics` 类，其基类包括 `ComponentMetrics`。核心方法包括 `component_type`, `get_parser`, `get_num_flops_breakdown`, `get_read_bytes_breakdown`, `get_write_bytes_breakdown`。

### `AttentionMetrics.get_parser` method / `AttentionMetrics.get_parser` 方法
```python
    @classmethod
    def get_parser(cls) -> ParserChain:
        return ParserChain(
            BaseConfigParser(),
            BaseAttentionConfigParser(),
            AttentionQuantizationConfigParser(),
        )
```
**EN:** This method returns or derives a value within `AttentionMetrics`. Key calls include `ParserChain`, `BaseConfigParser`, `BaseAttentionConfigParser`, `AttentionQuantizationConfigParser`.
**CN:** 该方法会返回或推导一个值，其作用域位于`AttentionMetrics`。 关键调用包括 `ParserChain`, `BaseConfigParser`, `BaseAttentionConfigParser`, `AttentionQuantizationConfigParser`。

### `AttentionMetrics.get_num_flops_breakdown` method / `AttentionMetrics.get_num_flops_breakdown` 方法
```python
    def get_num_flops_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]:
        L, D, q, kv, d = (
            self.num_hidden_layers,
            self.hidden_size,
            self.num_attention_heads,
            self.num_key_value_heads,
            self.head_dim,
        )
        T = ctx.total_num_tokens()
        TC = ctx.total_token_context_product()

        if per_gpu:
            L //= self.pp_size
            # tensor parallel along heads
            q = max(1, q // self.tp_size)
            kv = max(1, kv // self.tp_size)

        return {
            "qkv_proj": 2 * T * D * (q + 2 * kv) * d * L,
            "attn_qk": 2 * q * TC * d * L,
            "attn_av": 2 * q * TC * d * L,
            "out_proj": 2 * T * D * q * d * L,
        }
```
**EN:** This method returns or derives a value within `AttentionMetrics`. Key calls include `total_num_tokens`, `total_token_context_product`, `max`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`AttentionMetrics`。 关键调用包括 `total_num_tokens`, `total_token_context_product`, `max`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AttentionMetrics.get_read_bytes_breakdown` method / `AttentionMetrics.get_read_bytes_breakdown` 方法
```python
    def get_read_bytes_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]:
        L, D, q, kv, d = (
            self.num_hidden_layers,
            self.hidden_size,
            self.num_attention_heads,
            self.num_key_value_heads,
            self.head_dim,
        )
        T = ctx.total_num_tokens()

        if per_gpu:
            L //= self.pp_size
            # tensor parallel along heads
            q = max(1, q // self.tp_size)
            kv = max(1, kv // self.tp_size)

        read_bytes = {}

        read_bytes["qkv_input"] = T * D * self.activation_byte_size * L
        read_bytes["qkv_weight"] = int(D * (q + 2 * kv) * d * self.weight_byte_size * L)

        # Attention input reads differ between prefill and decode
        # Prefill: read Q, K, V activations (all in activation_byte_size)
        if ctx.prefill_num_tokens > 0:
            read_bytes["attn_input"] = (
                (ctx.prefill_num_tokens * q + 2 * ctx.prefill_context_len * kv)
                * d
                * self.activation_byte_size
                * L
            )

        # Decode: read Q activations + read K, V from cache (in cache_byte_size)
        if ctx.decode_num_tokens > 0:
            read_bytes["attn_input"] = read_bytes.get("attn_input", 0) + (
                ctx.decode_num_tokens * q * d * self.activation_byte_size * L
                + 2 * ctx.decode_context_len * kv * d * self.cache_byte_size * L
            )

        read_bytes["out_input"] = T * q * d * self.activation_byte_size * L
        read_bytes["out_weight"] = int(q * d * D * self.weight_byte_size * L)

        return read_bytes
```
**EN:** This method returns or derives a value within `AttentionMetrics`. Key calls include `total_num_tokens`, `int`, `max`, `get`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`AttentionMetrics`。 关键调用包括 `total_num_tokens`, `int`, `max`, `get`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AttentionMetrics.get_write_bytes_breakdown` method / `AttentionMetrics.get_write_bytes_breakdown` 方法
```python
    def get_write_bytes_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]:
        """Calculate write memory traffic for attention layers."""
        L, D, q, kv, d = (
            self.num_hidden_layers,
            self.hidden_size,
            self.num_attention_heads,
            self.num_key_value_heads,
            self.head_dim,
        )
        T = ctx.total_num_tokens()

        if per_gpu:
            L //= self.pp_size
            # tensor parallel along heads
            q = max(1, q // self.tp_size)
            kv = max(1, kv // self.tp_size)

        return {
            "qkv_output": T * (q + 2 * kv) * d * self.activation_byte_size * L,
            "kv_cache": 2 * T * kv * d * self.cache_byte_size * L,
            "out_output": T * D * self.activation_byte_size * L,
        }
```
**EN:** This method returns or derives a value within `AttentionMetrics`. The docstring frames it as: Calculate write memory traffic for attention layers. Key calls include `total_num_tokens`, `max`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`AttentionMetrics`。 关键调用包括 `total_num_tokens`, `max`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `BaseFfnConfigParser` class / `BaseFfnConfigParser` 类
```python
class BaseFfnConfigParser(Parser):
    """
    Parses FFN and MoE configuration.
    Provides: intermediate_size, num_experts, num_experts_per_tok,
    moe_intermediate_size, num_shared_experts, num_moe_layers
    """
```
**EN:** Introduces the `BaseFfnConfigParser` class on top of `Parser`. Core methods include `parse`. Docstring signal: Parses FFN and MoE configuration.
**CN:** 这里定义 `BaseFfnConfigParser` 类，其基类包括 `Parser`。核心方法包括 `parse`。

### `BaseFfnConfigParser.parse` method / `BaseFfnConfigParser.parse` 方法
```python
    def parse(self, args: ParsedArgs, vllm_config: VllmConfig) -> ParsedArgs:
        cfg = vllm_config.model_config.hf_config
        if hasattr(cfg, "text_config") and cfg.text_config is not None:
            cfg = cfg.text_config

        args.intermediate_size = getattr(cfg, "intermediate_size", args.hidden_size * 4)

        # Try different naming conventions.
        args.num_experts = vllm_config.model_config.get_num_experts()
        args.num_experts_per_tok = getattr_from_list(
            cfg, ["num_experts_per_tok", "moe_topk"], 0
        )
        args.moe_intermediate_size = getattr_from_list(
            cfg, ["moe_intermediate_size", "intermediate_size"], 0
        )
        args.num_shared_experts = getattr_from_list(
            cfg, ["n_shared_experts", "num_shared_experts"], 0
        )

        is_moe = args.num_experts != 0
        # Assume all MoE layers by default
        args.num_moe_layers = args.num_hidden_layers if is_moe else 0

        return args
```
**EN:** This method implements `parse` within `BaseFfnConfigParser`. Key calls include `getattr`, `get_num_experts`, `getattr_from_list`, `hasattr`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `parse`，其作用域位于`BaseFfnConfigParser`。 关键调用包括 `getattr`, `get_num_experts`, `getattr_from_list`, `hasattr`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FfnParallelParser` class / `FfnParallelParser` 类
```python
class FfnParallelParser(Parser):
    """
    Parses FFN parallelism configuration.

    Provides: ffn_tp_size, ffn_ep_size
    """
```
**EN:** Introduces the `FfnParallelParser` class on top of `Parser`. Core methods include `parse`. Docstring signal: Parses FFN parallelism configuration.
**CN:** 这里定义 `FfnParallelParser` 类，其基类包括 `Parser`。核心方法包括 `parse`。

### `FfnParallelParser.parse` method / `FfnParallelParser.parse` 方法
```python
    def parse(self, args: ParsedArgs, vllm_config: VllmConfig) -> ParsedArgs:
        # NOTE: ffn tp_size does not equal the tp_size parameter directly.
        # e.g.) If we use DP2TP4, ffn will use TP8 (or EP8 if EP is enabled.)
        if args.enable_ep:
            ffn_tp_size, ffn_ep_size = 1, args.dp_size * args.tp_size
        else:
            ffn_tp_size, ffn_ep_size = args.dp_size * args.tp_size, 1

        args.ffn_tp_size = ffn_tp_size
        args.ffn_ep_size = ffn_ep_size

        return args
```
**EN:** This method implements `parse` within `FfnParallelParser`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `parse`，其作用域位于`FfnParallelParser`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `InterleaveMoeLayerStepParser` class / `InterleaveMoeLayerStepParser` 类
```python
class InterleaveMoeLayerStepParser(Parser):
    """
    Parses interleave_moe_layer_step field for models like Llama4.

    Overrides: num_moe_layers
    """
```
**EN:** Introduces the `InterleaveMoeLayerStepParser` class on top of `Parser`. Core methods include `parse`. Docstring signal: Parses interleave_moe_layer_step field for models like Llama4.
**CN:** 这里定义 `InterleaveMoeLayerStepParser` 类，其基类包括 `Parser`。核心方法包括 `parse`。

### `InterleaveMoeLayerStepParser.parse` method / `InterleaveMoeLayerStepParser.parse` 方法
```python
    def parse(self, args: ParsedArgs, vllm_config: VllmConfig) -> ParsedArgs:
        cfg = vllm_config.model_config.hf_config
        if hasattr(cfg, "text_config") and cfg.text_config is not None:
            cfg = cfg.text_config

        if (
            hasattr(cfg, "interleave_moe_layer_step")
            and cfg.interleave_moe_layer_step > 0
        ):
            args.num_moe_layers = len(
                [
                    layer
                    for layer in range(args.num_hidden_layers)
                    if (layer + 1) % cfg.interleave_moe_layer_step == 0
                ]
            )

        return args
```
**EN:** This method implements `parse` within `InterleaveMoeLayerStepParser`. Key calls include `hasattr`, `len`, `range`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `parse`，其作用域位于`InterleaveMoeLayerStepParser`。 关键调用包括 `hasattr`, `len`, `range`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MoeLayerFreqParser` class / `MoeLayerFreqParser` 类
```python
class MoeLayerFreqParser(Parser):
    """
    Parses moe_layer_freq and first_k_dense_replace fields for models like Deepseek.

    Overrides: num_moe_layers
    """
```
**EN:** Introduces the `MoeLayerFreqParser` class on top of `Parser`. Core methods include `parse`. Docstring signal: Parses moe_layer_freq and first_k_dense_replace fields for models like Deepseek.
**CN:** 这里定义 `MoeLayerFreqParser` 类，其基类包括 `Parser`。核心方法包括 `parse`。

### `MoeLayerFreqParser.parse` method / `MoeLayerFreqParser.parse` 方法
```python
    def parse(self, args: ParsedArgs, vllm_config: VllmConfig) -> ParsedArgs:
        cfg = vllm_config.model_config.hf_config
        if hasattr(cfg, "text_config") and cfg.text_config is not None:
            cfg = cfg.text_config

        if hasattr(cfg, "moe_layer_freq") and hasattr(cfg, "first_k_dense_replace"):
            args.num_moe_layers = len(
                [
                    layer
                    for layer in range(args.num_hidden_layers)
                    if layer >= cfg.first_k_dense_replace
                    and layer % cfg.moe_layer_freq == 0
                ]
            )

        return args
```
**EN:** This method implements `parse` within `MoeLayerFreqParser`. Key calls include `hasattr`, `len`, `range`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `parse`，其作用域位于`MoeLayerFreqParser`。 关键调用包括 `hasattr`, `len`, `range`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FfnQuantizationConfigParser` class / `FfnQuantizationConfigParser` 类
```python
class FfnQuantizationConfigParser(Parser):
    """
    Parses quantization configuration for FFN layers.

    Overrides: weight_byte_size
    """
```
**EN:** Introduces the `FfnQuantizationConfigParser` class on top of `Parser`. Core methods include `parse`. Docstring signal: Parses quantization configuration for FFN layers.
**CN:** 这里定义 `FfnQuantizationConfigParser` 类，其基类包括 `Parser`。核心方法包括 `parse`。

### `FfnQuantizationConfigParser.parse` method / `FfnQuantizationConfigParser.parse` 方法
```python
    def parse(self, args: ParsedArgs, vllm_config: VllmConfig) -> ParsedArgs:
        cfg = vllm_config.quant_config

        if cfg is None:
            return args

        quant_method = cfg.get_name()
        if quant_method in _QUANT_WEIGHT_BYTE_SIZE:
            args.weight_byte_size = _QUANT_WEIGHT_BYTE_SIZE[quant_method]
        else:
            raise InvalidComponent(
                f"Unsupported quantization method for FFN metrics: {quant_method}"
            )

        return args
```
**EN:** This method implements `parse` within `FfnQuantizationConfigParser`. Key calls include `get_name`, `InvalidComponent`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `parse`，其作用域位于`FfnQuantizationConfigParser`。 关键调用包括 `get_name`, `InvalidComponent`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FfnMetrics` class / `FfnMetrics` 类
```python
class FfnMetrics(ComponentMetrics):
    # From BaseConfigParser
    num_hidden_layers: int = Field(..., gt=0)
    hidden_size: int = Field(..., gt=0)
    activation_byte_size: int = Field(..., gt=0)
    pp_size: int = Field(..., gt=0)

    # From FfnParallelParser
    ffn_tp_size: int = Field(..., gt=0)
    ffn_ep_size: int = Field(..., gt=0)

    # From BaseFfnConfigParser
    intermediate_size: int = Field(..., gt=0)
    num_experts: int = Field(0)
    num_experts_per_tok: int = Field(1)
    moe_intermediate_size: int = Field(0)
    num_shared_experts: int = Field(0)

    # From BaseConfigParser, can be overridden InterleaveMoeLayerStep or MoeLayerFreq
    num_moe_layers: int = Field(..., ge=0)

    # FIXME: might have to make this more granular
    # (i.e. dense_weight_byte_size, moe_routed_weight_byte_size,
    # moe_shared_weight_byte_size)
    # since it can differ from byte size of other components (e.g. attn)
    # and can differ even from each other.

    # From BaseConfigParser, can be overridden by FfnQuantizationConfigParser
    weight_byte_size: int | float = Field(..., gt=0)
```
**EN:** Introduces the `FfnMetrics` class on top of `ComponentMetrics`. Core methods include `validate_moe_fields`, `component_type`, `get_parser`, `get_num_flops_breakdown`, `get_read_bytes_breakdown`, `get_write_bytes_breakdown`.
**CN:** 这里定义 `FfnMetrics` 类，其基类包括 `ComponentMetrics`。核心方法包括 `validate_moe_fields`, `component_type`, `get_parser`, `get_num_flops_breakdown`, `get_read_bytes_breakdown`, `get_write_bytes_breakdown`。

### `FfnMetrics.validate_moe_fields` method / `FfnMetrics.validate_moe_fields` 方法
```python
    @model_validator(mode="after")
    def validate_moe_fields(self) -> Self:
        """Validate that MoE-related fields are properly set when num_moe_layers > 0."""
        if self.num_moe_layers > 0:
            assert self.num_experts, f"{self.num_experts=}"
            assert self.num_experts_per_tok, f"{self.num_experts_per_tok=}"
            assert self.moe_intermediate_size, f"{self.moe_intermediate_size=}"
        return self
```
**EN:** This method validates assumptions or constraints within `FfnMetrics`. The docstring frames it as: Validate that MoE-related fields are properly set when num_moe_layers > 0. Key calls include `model_validator`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会校验前提与约束，其作用域位于`FfnMetrics`。 关键调用包括 `model_validator`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FfnMetrics.get_num_flops_breakdown` method / `FfnMetrics.get_num_flops_breakdown` 方法
```python
    def get_num_flops_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]:
        """Calculate flops breakdown for FFN layers."""
        L, D, DI = self.num_hidden_layers, self.hidden_size, self.intermediate_size
        Lm, E, MI, S = (
            self.num_moe_layers,
            self.num_experts_per_tok,
            self.moe_intermediate_size,
            self.num_shared_experts,
        )
        T = ctx.total_num_tokens()

        Ld = L - Lm

        num_activated_tokens = T * E if E else 0

        if per_gpu:
            Ld //= self.pp_size
            Lm //= self.pp_size

            DI //= self.ffn_tp_size
            if MI is not None:
                MI //= self.ffn_tp_size
            if E:
                num_activated_tokens //= self.ffn_ep_size

        flops = {}

        # Dense FFN layers (SwiGLU: 3 linear layers: up, gate, down)
        if Ld:
            flops["dense_ffn"] = 2 * D * 3 * DI * T * Ld

        # MoE routed experts (each token activates E experts)
        if Lm and E:
            flops["routed_ffn"] = 2 * D * 3 * MI * num_activated_tokens * Lm

        # MoE shared experts (all S shared experts run for every token)
        if Lm and S:
            flops["shared_ffn"] = 2 * D * 3 * MI * S * T * Lm

        return flops
```
**EN:** This method returns or derives a value within `FfnMetrics`. The docstring frames it as: Calculate flops breakdown for FFN layers. Key calls include `total_num_tokens`. The control flow contains 7 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`FfnMetrics`。 关键调用包括 `total_num_tokens`。 控制流包含 7 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FfnMetrics.get_read_bytes_breakdown` method / `FfnMetrics.get_read_bytes_breakdown` 方法
```python
    def get_read_bytes_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]:
        """Calculate read memory traffic for FFN layers."""
        L, D, DI = self.num_hidden_layers, self.hidden_size, self.intermediate_size
        Lm, E, MI, S = (
            self.num_moe_layers,
            self.num_experts_per_tok,
            self.moe_intermediate_size,
            self.num_shared_experts,
        )
        T = ctx.total_num_tokens()
        num_experts = self.num_experts

        Ld = L - Lm

        num_activated_tokens = T * E if E else 0

        if per_gpu:
            Ld //= self.pp_size
            Lm //= self.pp_size

            DI //= self.ffn_tp_size
            if MI is not None:
                MI //= self.ffn_tp_size
            if E:
                num_activated_tokens //= self.ffn_ep_size
            if num_experts is not None:
                num_experts //= self.ffn_ep_size

        read_bytes = {}

        # Dense FFN layers (3 GEMMs: up, gate, down projections + SiLU activation)
        if Ld:
            read_bytes["dense_up_gate_input"] = int(
                T * D * self.activation_byte_size * Ld
            )
            read_bytes["dense_up_gate_weights"] = int(
                2 * D * DI * self.weight_byte_size * Ld
            )
            read_bytes["dense_silu_input"] = int(
                2 * T * DI * self.activation_byte_size * Ld
            )
            read_bytes["dense_down_input"] = int(
                T * DI * self.activation_byte_size * Ld
            )
            read_bytes["dense_down_weights"] = int(D * DI * self.weight_byte_size * Ld)

        if Lm:
            # MoE routed expert reads
            if E:
                # FIXME: Assume perfect load balancing for now.
                num_activated_experts = min(num_activated_tokens, num_experts)

                read_bytes["routed_up_gate_input"] = int(
                    num_activated_tokens * D * self.activation_byte_size * Lm
                )
                read_bytes["routed_up_gate_weights"] = int(
                    2 * D * MI * num_activated_experts * self.weight_byte_size * Lm
                )
                read_bytes["routed_silu_input"] = int(
                    2 * num_activated_tokens * MI * self.activation_byte_size * Lm
                )
                read_bytes["routed_down_input"] = int(
                    num_activated_tokens * MI * self.activation_byte_size * Lm
                )
                read_bytes["routed_down_weights"] = int(
                    D * MI * num_activated_experts * self.weight_byte_size * Lm
                )

            # MoE shared expert reads
            if S:
                read_bytes["shared_up_gate_input"] = int(
                    T * D * self.activation_byte_size * Lm
                )
                read_bytes["shared_up_gate_weights"] = int(
                    2 * D * MI * S * self.weight_byte_size * Lm
                )
                read_bytes["shared_silu_input"] = int(
                    2 * T * MI * S * self.activation_byte_size * Lm
                )
                read_bytes["shared_down_input"] = int(
                    T * MI * self.activation_byte_size * Lm
                )
                read_bytes["shared_down_weights"] = int(
                    D * MI * S * self.weight_byte_size * Lm
                )

        return read_bytes
```
**EN:** This method returns or derives a value within `FfnMetrics`. The docstring frames it as: Calculate read memory traffic for FFN layers. Key calls include `total_num_tokens`, `int`, `min`. The control flow contains 9 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`FfnMetrics`。 关键调用包括 `total_num_tokens`, `int`, `min`。 控制流包含 9 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FfnMetrics.get_write_bytes_breakdown` method / `FfnMetrics.get_write_bytes_breakdown` 方法
```python
    def get_write_bytes_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]:
        """Calculate write memory traffic for FFN layers."""
        L, D, DI = self.num_hidden_layers, self.hidden_size, self.intermediate_size
        Lm, E, MI, S = (
            self.num_moe_layers,
            self.num_experts_per_tok,
            self.moe_intermediate_size,
            self.num_shared_experts,
        )
        T = ctx.total_num_tokens()

        Ld = L - Lm

        num_activated_tokens = T * E if E else 0

        if per_gpu:
            Ld //= self.pp_size
            Lm //= self.pp_size

            DI //= self.ffn_tp_size
            if MI is not None:
                MI //= self.ffn_tp_size
            if E:
                num_activated_tokens //= self.ffn_ep_size

        write_bytes = {}

        # Dense FFN layers
        if Ld:
            write_bytes["dense_up_gate_output"] = int(
                2 * T * DI * self.activation_byte_size * Ld
            )
            write_bytes["dense_silu_output"] = int(
                T * DI * self.activation_byte_size * Ld
            )
            write_bytes["dense_down_output"] = int(
                T * D * self.activation_byte_size * Ld
            )

        # MoE outputs
        if Lm:
            if E:
                write_bytes["routed_up_gate_output"] = int(
                    2 * num_activated_tokens * MI * self.activation_byte_size * Lm
                )
                write_bytes["routed_silu_output"] = int(
                    num_activated_tokens * MI * self.activation_byte_size * Lm
                )
                write_bytes["routed_down_output"] = int(
                    num_activated_tokens * D * self.activation_byte_size * Lm
                )
            if S:
                write_bytes["shared_up_gate_output"] = int(
                    2 * T * S * MI * self.activation_byte_size * Lm
                )
                write_bytes["shared_silu_output"] = int(
                    T * S * MI * self.activation_byte_size * Lm
                )
                write_bytes["shared_down_output"] = int(
                    T * S * D * self.activation_byte_size * Lm
                )

        return write_bytes
```
**EN:** This method returns or derives a value within `FfnMetrics`. The docstring frames it as: Calculate write memory traffic for FFN layers. Key calls include `total_num_tokens`, `int`. The control flow contains 8 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`FfnMetrics`。 关键调用包括 `total_num_tokens`, `int`。 控制流包含 8 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UnembedMetrics` class / `UnembedMetrics` 类
```python
class UnembedMetrics(ComponentMetrics):
    # From BaseConfigParser
    hidden_size: int = Field(..., gt=0)
    vocab_size: int = Field(..., gt=0)
    weight_byte_size: int = Field(..., gt=0)
    activation_byte_size: int = Field(..., gt=0)

    tp_size: int
```
**EN:** Introduces the `UnembedMetrics` class on top of `ComponentMetrics`. Core methods include `component_type`, `get_parser`, `get_num_flops_breakdown`, `get_read_bytes_breakdown`, `get_write_bytes_breakdown`.
**CN:** 这里定义 `UnembedMetrics` 类，其基类包括 `ComponentMetrics`。核心方法包括 `component_type`, `get_parser`, `get_num_flops_breakdown`, `get_read_bytes_breakdown`, `get_write_bytes_breakdown`。

### `UnembedMetrics.get_parser` method / `UnembedMetrics.get_parser` 方法
```python
    @classmethod
    def get_parser(cls) -> ParserChain:
        return ParserChain(
            BaseConfigParser(),
        )
```
**EN:** This method returns or derives a value within `UnembedMetrics`. Key calls include `ParserChain`, `BaseConfigParser`.
**CN:** 该方法会返回或推导一个值，其作用域位于`UnembedMetrics`。 关键调用包括 `ParserChain`, `BaseConfigParser`。

### `UnembedMetrics.get_num_flops_breakdown` method / `UnembedMetrics.get_num_flops_breakdown` 方法
```python
    def get_num_flops_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]:
        """Calculate flops breakdown for unembedding layer."""
        D, V = self.hidden_size, self.vocab_size
        T = ctx.num_logits_tokens()

        if per_gpu:
            V //= self.tp_size

        return {
            "unembed": 2 * T * D * V,
        }
```
**EN:** This method returns or derives a value within `UnembedMetrics`. The docstring frames it as: Calculate flops breakdown for unembedding layer. Key calls include `num_logits_tokens`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`UnembedMetrics`。 关键调用包括 `num_logits_tokens`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UnembedMetrics.get_read_bytes_breakdown` method / `UnembedMetrics.get_read_bytes_breakdown` 方法
```python
    def get_read_bytes_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]:
        """Calculate read memory traffic for unembedding layer."""
        D, V = self.hidden_size, self.vocab_size
        T = ctx.num_logits_tokens()

        if per_gpu:
            V //= self.tp_size

        return {
            "input": T * D * self.activation_byte_size,
            "weight": D * V * self.weight_byte_size,
        }
```
**EN:** This method returns or derives a value within `UnembedMetrics`. The docstring frames it as: Calculate read memory traffic for unembedding layer. Key calls include `num_logits_tokens`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`UnembedMetrics`。 关键调用包括 `num_logits_tokens`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UnembedMetrics.get_write_bytes_breakdown` method / `UnembedMetrics.get_write_bytes_breakdown` 方法
```python
    def get_write_bytes_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]:
        """Calculate write memory traffic for unembedding layer."""
        V = self.vocab_size
        T = ctx.num_logits_tokens()

        if per_gpu:
            V //= self.tp_size

        return {
            "output": T * V * self.activation_byte_size,
        }
```
**EN:** This method returns or derives a value within `UnembedMetrics`. The docstring frames it as: Calculate write memory traffic for unembedding layer. Key calls include `num_logits_tokens`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`UnembedMetrics`。 关键调用包括 `num_logits_tokens`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ModelMetrics` class / `ModelMetrics` 类
```python
class ModelMetrics:
```
**EN:** Introduces the `ModelMetrics` class. Core methods include `__init__`, `is_enabled`, `get_num_flops`, `get_read_bytes`, `get_write_bytes`, `get_num_flops_breakdown`.
**CN:** 这里定义 `ModelMetrics` 类。核心方法包括 `__init__`, `is_enabled`, `get_num_flops`, `get_read_bytes`, `get_write_bytes`, `get_num_flops_breakdown`。

### `ModelMetrics.__init__` method / `ModelMetrics.__init__` 方法
```python
    def __init__(self, vllm_config: VllmConfig) -> None:
        """
        Parse vllm_config to instantiate metrics for each component.
        is_enabled() will return False if no component metrics could be instantiated.
        """

        self.vllm_config = vllm_config

        self.metrics: list[ComponentMetrics] = []
        for metric_cls in ComponentMetrics.registered_metrics():
            try:
                metric = metric_cls.from_vllm_config(vllm_config)
                self.metrics.append(metric)
                logger.info(
                    "Instantiated ComponentMetrics [%s] with (%s)",
                    metric.component_type(),
                    str(metric),
                )
            except InvalidComponent as e:
                logger.debug(
                    "Failed to instantiate %s from %s",
                    metric_cls.component_type(),
                    str(e),
                )
```
**EN:** This method initializes the object state within `ModelMetrics`. The docstring frames it as: Parse vllm_config to instantiate metrics for each component. Key calls include `registered_metrics`, `from_vllm_config`, `append`, `info`, `component_type`, `str`. It touches state such as `vllm_config`, `metrics`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`ModelMetrics`。 关键调用包括 `registered_metrics`, `from_vllm_config`, `append`, `info`, `component_type`, `str`。 它会读写 `vllm_config`, `metrics` 等状态。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `ModelMetrics.get_num_flops_breakdown` method / `ModelMetrics.get_num_flops_breakdown` 方法
```python
    def get_num_flops_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]:
        total = {}
        for metric in self.metrics:
            breakdown = metric.get_num_flops_breakdown(ctx, per_gpu)
            component = metric.component_type()
            prefixed = {f"{component}.{key}": val for key, val in breakdown.items()}
            total.update(prefixed)
        return total
```
**EN:** This method returns or derives a value within `ModelMetrics`. Key calls include `get_num_flops_breakdown`, `component_type`, `update`, `items`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`ModelMetrics`。 关键调用包括 `get_num_flops_breakdown`, `component_type`, `update`, `items`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `ModelMetrics.get_read_bytes_breakdown` method / `ModelMetrics.get_read_bytes_breakdown` 方法
```python
    def get_read_bytes_breakdown(
        self, ctx: ExecutionContext, per_gpu: bool = True
    ) -> dict[str, int]:
        total = {}
        for metric in self.metrics:
            breakdown = metric.get_read_bytes_breakdown(ctx, per_gpu)
            component = metric.component_type()
            prefixed = {f"{component}.{key}": val for key, val in breakdown.items()}
            total.update(prefixed)
        return total
```
**EN:** This method returns or derives a value within `ModelMetrics`. Key calls include `get_read_bytes_breakdown`, `component_type`, `update`, `items`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`ModelMetrics`。 关键调用包括 `get_read_bytes_breakdown`, `component_type`, `update`, `items`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `ModelMetrics.get_step_perf_stats_per_gpu` method / `ModelMetrics.get_step_perf_stats_per_gpu` 方法
```python
    def get_step_perf_stats_per_gpu(
        self, scheduler_output: SchedulerOutput
    ) -> PerfStats:
        """
        Calculate perf stats for the current step based on scheduled tokens.
        """

        t0 = time.monotonic()

        # Build a single batch context
        ctx = ExecutionContext()

        # Process new requests (these are in prefill phase)
        for new_req in scheduler_output.scheduled_new_reqs:
            req_id = new_req.req_id
            num_tokens = scheduler_output.num_scheduled_tokens.get(req_id, 0)
            if num_tokens == 0:
                continue

            # For new requests, context_len = num_computed_tokens + num_tokens
            # num_computed_tokens represents previously computed tokens in the sequence
            context_len = new_req.num_computed_tokens + num_tokens
            ctx.add(num_tokens, context_len, is_prefill=True)

        # Process cached requests (continuing requests)
        cached_reqs = scheduler_output.scheduled_cached_reqs
        for i, req_id in enumerate(cached_reqs.req_ids):
            num_tokens = scheduler_output.num_scheduled_tokens.get(req_id, 0)
            if num_tokens == 0:
                continue

            # For cached requests, we have the current num_computed_tokens
            num_computed_tokens = cached_reqs.num_computed_tokens[i]
            context_len = num_computed_tokens + num_tokens

            # Cached requests are typically in decode phase (num_tokens == 1)
            # unless they're doing chunked prefill (num_tokens > 1)
            is_prefill = num_tokens > 1
            ctx.add(num_tokens, context_len, is_prefill)

        num_flops_breakdown = self.get_num_flops_breakdown(ctx, True)
        read_bytes_breakdown = self.get_read_bytes_breakdown(ctx, True)
        write_bytes_breakdown = self.get_write_bytes_breakdown(ctx, True)
        perf_stats = PerfStats(
            sum(num_flops_breakdown.values()),
            sum(read_bytes_breakdown.values()),
            sum(write_bytes_breakdown.values()),
        )

        if envs.VLLM_DEBUG_MFU_METRICS:
            perf_stats.debug_stats = DebugPerfStats(
                time.monotonic() - t0,
                ctx.num_prefill_requests,
                ctx.num_decode_requests,
                asdict(ctx),
                num_flops_breakdown,
                read_bytes_breakdown,
                write_bytes_breakdown,
            )

        return perf_stats
```
**EN:** This method returns or derives a value within `ModelMetrics`. The docstring frames it as: Calculate perf stats for the current step based on scheduled tokens. Key calls include `monotonic`, `ExecutionContext`, `enumerate`, `get_num_flops_breakdown`, `get_read_bytes_breakdown`, `get_write_bytes_breakdown`. The control flow contains 3 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`ModelMetrics`。 关键调用包括 `monotonic`, `ExecutionContext`, `enumerate`, `get_num_flops_breakdown`, `get_read_bytes_breakdown`, `get_write_bytes_breakdown`。 控制流包含 3 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `PerfMetricsDebugLogging` class / `PerfMetricsDebugLogging` 类
```python
class PerfMetricsDebugLogging:
```
**EN:** Introduces the `PerfMetricsDebugLogging` class. Core methods include `__init__`, `reset`, `observe`, `log`.
**CN:** 这里定义 `PerfMetricsDebugLogging` 类。核心方法包括 `__init__`, `reset`, `observe`, `log`。

### `PerfMetricsDebugLogging.__init__` method / `PerfMetricsDebugLogging.__init__` 方法
```python
    def __init__(self):
        self.reset()
```
**EN:** This method initializes the object state within `PerfMetricsDebugLogging`. Key calls include `reset`.
**CN:** 该方法会初始化对象状态，其作用域位于`PerfMetricsDebugLogging`。 关键调用包括 `reset`。

### `PerfMetricsDebugLogging.reset` method / `PerfMetricsDebugLogging.reset` 方法
```python
    def reset(self):
        self.total_calc_duration: float = 0.0
        self.total_num_prefill_requests: int = 0
        self.total_num_decode_requests: int = 0
        self.total_num_batches: int = 0
        self.total_context_breakdown: dict[str, int] = {}
        self.total_num_flops_per_gpu_breakdown: dict[str, int] = {}
        self.total_read_bytes_per_gpu_breakdown: dict[str, int] = {}
        self.total_write_bytes_per_gpu_breakdown: dict[str, int] = {}
```
**EN:** This method implements `reset` within `PerfMetricsDebugLogging`. It touches state such as `total_calc_duration`, `total_num_prefill_requests`, `total_num_decode_requests`, `total_num_batches`, `total_context_breakdown`, `total_num_flops_per_gpu_breakdown`, `total_read_bytes_per_gpu_breakdown`, `total_write_bytes_per_gpu_breakdown`.
**CN:** 该方法会实现 `reset`，其作用域位于`PerfMetricsDebugLogging`。 它会读写 `total_calc_duration`, `total_num_prefill_requests`, `total_num_decode_requests`, `total_num_batches`, `total_context_breakdown`, `total_num_flops_per_gpu_breakdown`, `total_read_bytes_per_gpu_breakdown`, `total_write_bytes_per_gpu_breakdown` 等状态。

### `PerfMetricsDebugLogging.observe` method / `PerfMetricsDebugLogging.observe` 方法
```python
    def observe(self, debug_stats: DebugPerfStats) -> None:
        self.total_calc_duration += debug_stats.calc_duration
        self.total_num_prefill_requests += debug_stats.num_prefill_requests
        self.total_num_decode_requests += debug_stats.num_decode_requests
        self.total_num_batches += 1

        for dst, src in zip(
            [
                self.total_context_breakdown,
                self.total_num_flops_per_gpu_breakdown,
                self.total_read_bytes_per_gpu_breakdown,
                self.total_write_bytes_per_gpu_breakdown,
            ],
            [
                debug_stats.context_breakdown,
                debug_stats.num_flops_per_gpu_breakdown,
                debug_stats.num_read_bytes_per_gpu_breakdown,
                debug_stats.num_write_bytes_per_gpu_breakdown,
            ],
        ):
            assert isinstance(src, dict)
            for key, val in src.items():
                dst[key] = dst.get(key, 0) + val
```
**EN:** This method implements `observe` within `PerfMetricsDebugLogging`. Key calls include `zip`, `isinstance`, `items`, `get`. It touches state such as `total_calc_duration`, `total_num_prefill_requests`, `total_num_decode_requests`, `total_num_batches`. The control flow contains 0 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `observe`，其作用域位于`PerfMetricsDebugLogging`。 关键调用包括 `zip`, `isinstance`, `items`, `get`。 它会读写 `total_calc_duration`, `total_num_prefill_requests`, `total_num_decode_requests`, `total_num_batches` 等状态。 控制流包含 0 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `PerfMetricsDebugLogging.log` method / `PerfMetricsDebugLogging.log` 方法
```python
    def log(self, log_fn, log_prefix: str, delta_time: float):
        # pretty print breakdowns
        total_num_flops_per_gpu_breakdown = {
            k: f"{v / 1e12:.1f}TF"
            for k, v in self.total_num_flops_per_gpu_breakdown.items()
        }
        total_read_bytes_per_gpu_breakdown = {
            k: f"{v / 1e9:.1f}GB"
            for k, v in self.total_read_bytes_per_gpu_breakdown.items()
        }
        total_write_bytes_per_gpu_breakdown = {
            k: f"{v / 1e9:.1f}GB"
            for k, v in self.total_write_bytes_per_gpu_breakdown.items()
        }

        logger.debug(
            "%sMFU details: %s",
            log_prefix,
            json.dumps(
                {
                    "prefill_reqs": self.total_num_prefill_requests,
                    "decode_reqs": self.total_num_decode_requests,
                    "num_batches": self.total_num_batches,
                    "context_breakdown": self.total_context_breakdown,
                    "flops_breakdown": total_num_flops_per_gpu_breakdown,
                    "num_read_bytes_breakdown": total_read_bytes_per_gpu_breakdown,
                    "num_write_bytes_breakdown": (total_write_bytes_per_gpu_breakdown),
                    "duration": f"{delta_time:.1f}s",
                    "mfu_calc_overhead": (
                        f"{self.total_calc_duration / delta_time:.1%}"
                    ),
                },
                indent=2,
            ),
        )
```
**EN:** This method implements `log` within `PerfMetricsDebugLogging`. Key calls include `debug`, `dumps`, `items`.
**CN:** 该方法会实现 `log`，其作用域位于`PerfMetricsDebugLogging`。 关键调用包括 `debug`, `dumps`, `items`。

### `PerfMetricsLogging` class / `PerfMetricsLogging` 类
```python
class PerfMetricsLogging:
```
**EN:** Introduces the `PerfMetricsLogging` class. Core methods include `__init__`, `reset`, `observe`, `log`.
**CN:** 这里定义 `PerfMetricsLogging` 类。核心方法包括 `__init__`, `reset`, `observe`, `log`。

### `PerfMetricsLogging.__init__` method / `PerfMetricsLogging.__init__` 方法
```python
    def __init__(self, vllm_config: VllmConfig):
        self.vllm_config = vllm_config
        self.pp_size = vllm_config.parallel_config.pipeline_parallel_size

        self.debug_logging: PerfMetricsDebugLogging | None = None
        if envs.VLLM_DEBUG_MFU_METRICS:
            self.debug_logging = PerfMetricsDebugLogging()

        self.reset()
```
**EN:** This method initializes the object state within `PerfMetricsLogging`. Key calls include `reset`, `PerfMetricsDebugLogging`. It touches state such as `vllm_config`, `pp_size`, `debug_logging`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`PerfMetricsLogging`。 关键调用包括 `reset`, `PerfMetricsDebugLogging`。 它会读写 `vllm_config`, `pp_size`, `debug_logging` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PerfMetricsLogging.reset` method / `PerfMetricsLogging.reset` 方法
```python
    def reset(self):
        self.last_log_time = time.monotonic()

        self.total_num_flops_per_gpu: int = 0
        self.total_read_bytes_per_gpu: int = 0
        self.total_write_bytes_per_gpu: int = 0

        if self.debug_logging:
            self.debug_logging.reset()
```
**EN:** This method implements `reset` within `PerfMetricsLogging`. Key calls include `monotonic`, `reset`. It touches state such as `last_log_time`, `total_num_flops_per_gpu`, `total_read_bytes_per_gpu`, `total_write_bytes_per_gpu`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `reset`，其作用域位于`PerfMetricsLogging`。 关键调用包括 `monotonic`, `reset`。 它会读写 `last_log_time`, `total_num_flops_per_gpu`, `total_read_bytes_per_gpu`, `total_write_bytes_per_gpu` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PerfMetricsLogging.observe` method / `PerfMetricsLogging.observe` 方法
```python
    def observe(self, perf_stats: PerfStats) -> None:
        self.total_num_flops_per_gpu += perf_stats.num_flops_per_gpu
        self.total_read_bytes_per_gpu += perf_stats.num_read_bytes_per_gpu
        self.total_write_bytes_per_gpu += perf_stats.num_write_bytes_per_gpu

        if self.debug_logging:
            assert perf_stats.debug_stats is not None
            self.debug_logging.observe(perf_stats.debug_stats)
```
**EN:** This method implements `observe` within `PerfMetricsLogging`. Key calls include `observe`. It touches state such as `total_num_flops_per_gpu`, `total_read_bytes_per_gpu`, `total_write_bytes_per_gpu`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `observe`，其作用域位于`PerfMetricsLogging`。 关键调用包括 `observe`。 它会读写 `total_num_flops_per_gpu`, `total_read_bytes_per_gpu`, `total_write_bytes_per_gpu` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PerfMetricsLogging.log` method / `PerfMetricsLogging.log` 方法
```python
    def log(self, log_fn=logger.info, log_prefix: str = "") -> None:
        if not (
            self.total_num_flops_per_gpu
            or self.total_read_bytes_per_gpu
            or self.total_write_bytes_per_gpu
        ):
            return

        now = time.monotonic()
        delta_time = now - self.last_log_time

        if delta_time <= 0.0:
            avg_tflops_per_gpu = 0.0
            avg_gbps_per_gpu = 0.0
        else:
            avg_tflops_per_gpu = self.total_num_flops_per_gpu / delta_time / 1e12
            avg_gbps_per_gpu = (
                (self.total_read_bytes_per_gpu + self.total_write_bytes_per_gpu)
                / delta_time
                / 1e9
            )

        log_fn(
            "%sMFU: %.1f TF/s/GPU %.1f GB/s/GPU",
            log_prefix,
            avg_tflops_per_gpu,
            avg_gbps_per_gpu,
        )

        if self.debug_logging:
            self.debug_logging.log(log_fn, log_prefix, delta_time)

        self.reset()
```
**EN:** This method implements `log` within `PerfMetricsLogging`. Key calls include `monotonic`, `log_fn`, `reset`, `log`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `log`，其作用域位于`PerfMetricsLogging`。 关键调用包括 `monotonic`, `log_fn`, `reset`, `log`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PerfMetricsProm` class / `PerfMetricsProm` 类
```python
class PerfMetricsProm:
    """Record performance metrics in Prometheus.

    Average TFLOPS (tera floating-point operations per second) can be
    calculated using a PromQL query:

      rate(vllm:estimated_flops_per_gpu_total[1m]) / 1e12

    Average memory bandwidth in GB/s can be calculated using:

      (rate(vllm:estimated_read_bytes_per_gpu_total[1m]) +
       rate(vllm:estimated_write_bytes_per_gpu_total[1m])) / 1e9
    """

    _counter_cls = prometheus_client.Counter
```
**EN:** Introduces the `PerfMetricsProm` class. Core methods include `__init__`, `observe`. Docstring signal: Record performance metrics in Prometheus.
**CN:** 这里定义 `PerfMetricsProm` 类。核心方法包括 `__init__`, `observe`。

### `PerfMetricsProm.__init__` method / `PerfMetricsProm.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        labelnames: list[str],
        per_engine_labelvalues: dict[int, list[object]],
    ):
        counter_flops = self._counter_cls(
            name="vllm:estimated_flops_per_gpu_total",
            documentation=(
                "Estimated number of floating point operations per GPU "
                "(for Model Flops Utilization calculations)."
            ),
            labelnames=labelnames,
        )
        self.counter_flops = create_metric_per_engine(
            counter_flops, per_engine_labelvalues
        )

        counter_read_bytes = self._counter_cls(
            name="vllm:estimated_read_bytes_per_gpu_total",
            documentation=(
                "Estimated number of bytes read from memory per GPU "
                "(for Model Flops Utilization calculations)."
            ),
            labelnames=labelnames,
        )
        self.counter_read_bytes = create_metric_per_engine(
            counter_read_bytes, per_engine_labelvalues
        )

        counter_write_bytes = self._counter_cls(
            name="vllm:estimated_write_bytes_per_gpu_total",
            documentation=(
                "Estimated number of bytes written to memory per GPU "
                "(for Model Flops Utilization calculations)."
            ),
            labelnames=labelnames,
        )
        self.counter_write_bytes = create_metric_per_engine(
            counter_write_bytes, per_engine_labelvalues
        )
```
**EN:** This method initializes the object state within `PerfMetricsProm`. Key calls include `_counter_cls`, `create_metric_per_engine`. It touches state such as `counter_flops`, `counter_read_bytes`, `counter_write_bytes`.
**CN:** 该方法会初始化对象状态，其作用域位于`PerfMetricsProm`。 关键调用包括 `_counter_cls`, `create_metric_per_engine`。 它会读写 `counter_flops`, `counter_read_bytes`, `counter_write_bytes` 等状态。

### `PerfMetricsProm.observe` method / `PerfMetricsProm.observe` 方法
```python
    def observe(self, perf_stats: PerfStats, engine_idx: int = 0):
        if not (
            perf_stats.num_flops_per_gpu
            or perf_stats.num_read_bytes_per_gpu
            or perf_stats.num_write_bytes_per_gpu
        ):
            return
        self.counter_flops[engine_idx].inc(perf_stats.num_flops_per_gpu)
        self.counter_read_bytes[engine_idx].inc(perf_stats.num_read_bytes_per_gpu)
        self.counter_write_bytes[engine_idx].inc(perf_stats.num_write_bytes_per_gpu)
```
**EN:** This method implements `observe` within `PerfMetricsProm`. Key calls include `inc`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `observe`，其作用域位于`PerfMetricsProm`。 关键调用包括 `inc`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `get_required` function / `get_required` 函数
```python
def get_required(obj: object, attr: str):
    """Get an attr from an object, or throw a InvalidComponentError if it's not set."""
    if not hasattr(obj, attr):
        raise InvalidComponent(f"Missing required attr {attr} in config")
    return getattr(obj, attr)
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Get an attr from an object, or throw a InvalidComponentError if it's not set. Key calls include `getattr`, `hasattr`, `InvalidComponent`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `getattr`, `hasattr`, `InvalidComponent`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `getattr_from_list` function / `getattr_from_list` 函数
```python
def getattr_from_list(obj: object, attrs: list[str], default: object = None):
    """Try to get the first attr that exists in the object
    from a list of attrs. Otherwise return None."""
    for attr in attrs:
        if hasattr(obj, attr):
            return getattr(obj, attr)
    return default
```
**EN:** This function implements `getattr_from_list` within the module. The docstring frames it as: Try to get the first attr that exists in the object from a list of attrs. Key calls include `hasattr`, `getattr`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `getattr_from_list`，其作用域位于the module。 关键调用包括 `hasattr`, `getattr`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `InvalidComponent`: central class or interface in this module. / `InvalidComponent`：本模块中的核心类或接口。
- `DebugPerfStats`: central class or interface in this module. / `DebugPerfStats`：本模块中的核心类或接口。
- `PerfStats`: central class or interface in this module. / `PerfStats`：本模块中的核心类或接口。
- `ExecutionContext`: central class or interface in this module. / `ExecutionContext`：本模块中的核心类或接口。
- `ParsedArgs`: central class or interface in this module. / `ParsedArgs`：本模块中的核心类或接口。
- `Parser`: central class or interface in this module. / `Parser`：本模块中的核心类或接口。
- `ParserChain`: central class or interface in this module. / `ParserChain`：本模块中的核心类或接口。
- `ComponentMetrics`: central class or interface in this module. / `ComponentMetrics`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `json`, `time`, `abc`, `collections`, `dataclasses`, `typing`
- External / 外部依赖: `prometheus_client`, `torch`, `pydantic`, `typing_extensions`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.config`, `vllm.logger`, `vllm.utils.torch_utils`, `vllm.v1.core.sched.output`, `vllm.v1.metrics.utils`
