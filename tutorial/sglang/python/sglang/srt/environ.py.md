# environ.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/environ.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the core runtime configuration part of the SRT runtime and implements logic centered on `environ`. It exposes primary entry points such as `temp_set_env`, `EnvField`, `EnvTuple`. / 该模块属于 SRT 运行时的核心运行时配置部分，主要实现围绕 `environ` 的逻辑。 它对外提供的主要入口包括 `temp_set_env`, `EnvField`, `EnvTuple`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Module imports, constants, and setup
```python
import os
import subprocess
import warnings
from contextlib import ExitStack, contextmanager
from enum import IntEnum
from typing import Any, Optional


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 9-35: Function temp_set_env
```python
@contextmanager
def temp_set_env(*, allow_sglang: bool = False, **env_vars: Any):
    """Temporarily set environment variables, restoring originals on exit.

    By default, SGLANG_*/SGL_* keys are rejected — use ``Envs`` descriptors
    for those.  Pass ``allow_sglang=True`` only for special env vars that
    intentionally bypass ``environ.py``.
    """
    if not allow_sglang:
        for key in env_vars:
            if key.startswith("SGLANG_") or key.startswith("SGL_"):
                raise ValueError("temp_set_env should not be used for sglang env vars")

    backup = {key: os.environ.get(key) for key in env_vars}
    try:
        for key, value in env_vars.items():
            if value is None:
                os.environ.pop(key, None)
            else:
                os.environ[key] = str(value)
        yield
    finally:
        for key, value in backup.items():
            if value is None:
                os.environ.pop(key, None)
            else:
                os.environ[key] = value
```
**EN:** This callable implements `temp_set_env`. It takes `**env_vars` and mainly applies configuration to mutable state. The docstring states: "Temporarily set environment variables, restoring originals on exit." In this range it performs defensive checks on invalid state; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `temp_set_env`。它接收 `**env_vars`，主要用于将配置写入可变状态。 在这一范围内，它会对非法状态执行防御性检查；读取环境变量驱动的配置。

### Lines 38-40: Class EnvField
```python
class EnvField:
    _allow_set_name = True

```
**EN:** This range introduces `EnvField` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `EnvField`，并定义其后续方法依赖的结构或元数据。

### Lines 41-45: Method EnvField.__init__
```python
    def __init__(self, default: Any):
        self.default = default
        # NOTE: environ can only accept str values, so we need a flag to indicate
        # whether the env var is explicitly set to None.
        self._set_to_none = False
```
**EN:** This callable implements `EnvField.__init__`. It takes `default` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `EnvField.__init__`。它接收 `default`，主要用于初始化实例状态与默认值。

### Lines 47-49: Method EnvField.__set_name__
```python
    def __set_name__(self, owner, name):
        assert EnvField._allow_set_name, "Usage like `a = envs.A` is not allowed"
        self.name = name
```
**EN:** This callable implements `EnvField.__set_name__`. It takes `owner`, `name` and mainly applies configuration to mutable state. In this range it performs defensive checks on invalid state; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `EnvField.__set_name__`。它接收 `owner`, `name`，主要用于将配置写入可变状态。 在这一范围内，它会对非法状态执行防御性检查；读取环境变量驱动的配置。

### Lines 51-52: Method EnvField.parse
```python
    def parse(self, value: str) -> Any:
        raise NotImplementedError()
```
**EN:** This callable implements `EnvField.parse`. It takes `value` and mainly parses structured input. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `EnvField.parse`。它接收 `value`，主要用于解析结构化输入。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 54-72: Method EnvField.get
```python
    def get(self) -> Any:
        value = os.getenv(self.name)

        # Explicitly set to None
        if self._set_to_none:
            assert value == str(None)
            return None

        # Not set, return default
        if value is None:
            return self.default

        try:
            return self.parse(value)
        except ValueError as e:
            warnings.warn(
                f'Invalid value for {self.name}: {e}, using default "{self.default}"'
            )
            return self.default
```
**EN:** This callable implements `EnvField.get` and mainly retrieves a value or derived view. In this range it performs defensive checks on invalid state; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `EnvField.get`，主要用于获取某个值或派生视图。 在这一范围内，它会对非法状态执行防御性检查；读取环境变量驱动的配置。

### Lines 74-75: Method EnvField.is_set
```python
    def is_set(self):
        return self.name in os.environ
```
**EN:** This callable implements `EnvField.is_set` and mainly applies configuration to mutable state. In this range it reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `EnvField.is_set`，主要用于将配置写入可变状态。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 77-79: Method EnvField.set
```python
    def set(self, value: Any):
        self._set_to_none = value is None
        os.environ[self.name] = str(value)
```
**EN:** This callable implements `EnvField.set`. It takes `value` and mainly applies configuration to mutable state. In this range it reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `EnvField.set`。它接收 `value`，主要用于将配置写入可变状态。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 81-92: Method EnvField.override
```python
    @contextmanager
    def override(self, value: Any):
        backup_present = self.name in os.environ
        backup_value = os.environ.get(self.name)
        backup_set_to_none = self._set_to_none
        self.set(value)
        yield
        if backup_present:
            os.environ[self.name] = backup_value
        else:
            os.environ.pop(self.name, None)
        self._set_to_none = backup_set_to_none
```
**EN:** This callable implements `EnvField.override`. It takes `value` and mainly implements override. In this range it reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `EnvField.override`。它接收 `value`，主要用于实现 override 相关逻辑。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 94-96: Method EnvField.clear
```python
    def clear(self):
        os.environ.pop(self.name, None)
        self._set_to_none = False
```
**EN:** This callable implements `EnvField.clear` and mainly implements clear. In this range it reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `EnvField.clear`，主要用于实现 clear 相关逻辑。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 98-101: Method EnvField.__bool__
```python
    def __bool__(self):
        raise RuntimeError(
            "Please use `envs.YOUR_FLAG.get()` instead of `envs.YOUR_FLAG`"
        )
```
**EN:** This callable implements `EnvField.__bool__` and mainly implements bool. In this range it performs defensive checks on invalid state; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `EnvField.__bool__`，主要用于实现 bool 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；读取环境变量驱动的配置。

### Lines 103-106: Method EnvField.__len__
```python
    def __len__(self):
        raise RuntimeError(
            "Please use `envs.YOUR_FLAG.get()` instead of `envs.YOUR_FLAG`"
        )
```
**EN:** This callable implements `EnvField.__len__` and mainly implements len. In this range it performs defensive checks on invalid state; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `EnvField.__len__`，主要用于实现 len 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；读取环境变量驱动的配置。

### Lines 109-109: Class EnvTuple
```python
class EnvTuple(EnvField):
```
**EN:** This range introduces `EnvTuple` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `EnvTuple`，并定义其后续方法依赖的结构或元数据。

### Lines 110-111: Method EnvTuple.parse
```python
    def parse(self, value: str) -> tuple[str, ...]:
        return tuple(s.strip() for s in value.split(",") if s.strip())
```
**EN:** This callable implements `EnvTuple.parse`. It takes `value` and mainly parses structured input.
**CN:** 这一可调用对象实现了 `EnvTuple.parse`。它接收 `value`，主要用于解析结构化输入。

### Lines 114-114: Class EnvStr
```python
class EnvStr(EnvField):
```
**EN:** This range introduces `EnvStr` and defines the structure or metadata that its methods rely on. In this range it reads environment-driven configuration.
**CN:** 这一段引入 `EnvStr`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 115-116: Method EnvStr.parse
```python
    def parse(self, value: str) -> str:
        return value
```
**EN:** This callable implements `EnvStr.parse`. It takes `value` and mainly parses structured input.
**CN:** 这一可调用对象实现了 `EnvStr.parse`。它接收 `value`，主要用于解析结构化输入。

### Lines 119-119: Class EnvBool
```python
class EnvBool(EnvField):
```
**EN:** This range introduces `EnvBool` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `EnvBool`，并定义其后续方法依赖的结构或元数据。

### Lines 120-126: Method EnvBool.parse
```python
    def parse(self, value: str) -> bool:
        value = value.lower()
        if value in ["true", "1", "yes", "y"]:
            return True
        if value in ["false", "0", "no", "n"]:
            return False
        raise ValueError(f'"{value}" is not a valid boolean value')
```
**EN:** This callable implements `EnvBool.parse`. It takes `value` and mainly parses structured input. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `EnvBool.parse`。它接收 `value`，主要用于解析结构化输入。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 129-129: Class EnvInt
```python
class EnvInt(EnvField):
```
**EN:** This range introduces `EnvInt` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `EnvInt`，并定义其后续方法依赖的结构或元数据。

### Lines 130-134: Method EnvInt.parse
```python
    def parse(self, value: str) -> int:
        try:
            return int(value)
        except ValueError:
            raise ValueError(f'"{value}" is not a valid integer value')
```
**EN:** This callable implements `EnvInt.parse`. It takes `value` and mainly parses structured input. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `EnvInt.parse`。它接收 `value`，主要用于解析结构化输入。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 137-137: Class EnvFloat
```python
class EnvFloat(EnvField):
```
**EN:** This range introduces `EnvFloat` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `EnvFloat`，并定义其后续方法依赖的结构或元数据。

### Lines 138-142: Method EnvFloat.parse
```python
    def parse(self, value: str) -> float:
        try:
            return float(value)
        except ValueError:
            raise ValueError(f'"{value}" is not a valid float value')
```
**EN:** This callable implements `EnvFloat.parse`. It takes `value` and mainly parses structured input. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `EnvFloat.parse`。它接收 `value`，主要用于解析结构化输入。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 145-156: Class ToolStrictLevel
```python
class ToolStrictLevel(IntEnum):
    """
    Defines the strictness levels for tool call parsing and validation.

    OFF: No strict validation
    FUNCTION: Enables structural tag constraints for all tools
    PARAMETER: Enforces strict parameter validation for all tools
    """

    OFF = 0
    FUNCTION = 1
    PARAMETER = 2
```
**EN:** This range introduces `ToolStrictLevel` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Defines the strictness levels for tool call parsing and validation."
**CN:** 这一段引入 `ToolStrictLevel`，并定义其后续方法依赖的结构或元数据。

### Lines 159-218: Class Envs (part 1/9)
```python
class Envs:
    # fmt: off

    # Model & File Download
    SGLANG_USE_MODELSCOPE = EnvBool(False)
    SGLANG_SORT_WEIGHT_FILES = EnvBool(False)
    SGLANG_DISABLED_MODEL_ARCHS = EnvTuple(tuple())
    SGLANG_PREFETCH_BLOCK_SIZE_MB = EnvInt(16)

    # Logging Options
    SGLANG_LOG_GC = EnvBool(False)
    SGLANG_LOG_FORWARD_ITERS = EnvBool(False)
    SGLANG_LOG_MS = EnvBool(False)
    SGLANG_LOG_REQUEST_EXCEEDED_MS = EnvInt(-1)
    SGLANG_LOG_REQUEST_HEADERS = EnvTuple(tuple())
    SGLANG_LOG_SCHEDULER_STATUS_TARGET = EnvStr("")
    SGLANG_LOG_SCHEDULER_STATUS_INTERVAL = EnvFloat(60.0)

    # SGLang CI
    SGLANG_IS_IN_CI = EnvBool(False)
    SGLANG_IS_IN_CI_AMD = EnvBool(False)
    SGLANG_CUDA_COREDUMP = EnvBool(False)
    SGLANG_CUDA_COREDUMP_DIR = EnvStr("/tmp/sglang_cuda_coredumps")
    SGLANG_TEST_MAX_RETRY = EnvInt(None)

    # Constrained Decoding (Grammar)
    SGLANG_GRAMMAR_POLL_INTERVAL = EnvFloat(0.005)
    SGLANG_GRAMMAR_MAX_POLL_ITERATIONS = EnvInt(10000)
    SGLANG_DISABLE_OUTLINES_DISK_CACHE = EnvBool(False)

    # Test & Debug
    SGLANG_DETECT_SLOW_RANK = EnvBool(False)
    SGLANG_TEST_STUCK_DETOKENIZER = EnvFloat(0)
    SGLANG_TEST_STUCK_DP_CONTROLLER = EnvFloat(0)
    SGLANG_TEST_STUCK_SCHEDULER_INIT = EnvFloat(0)
    SGLANG_TEST_STUCK_TOKENIZER = EnvFloat(0)
    SGLANG_TEST_CRASH_AFTER_STREAM_OUTPUTS = EnvInt(0)
    IS_H200 = EnvBool(False)
    SGLANG_SET_CPU_AFFINITY = EnvBool(False)
    SGLANG_PROFILE_WITH_STACK = EnvBool(True)
    SGLANG_PROFILE_RECORD_SHAPES = EnvBool(True)
    SGLANG_PROFILE_V2 = EnvBool(False)
    SGLANG_RECORD_STEP_TIME = EnvBool(False)
    SGLANG_FORCE_SHUTDOWN = EnvBool(False)
    SGLANG_DEBUG_MEMORY_POOL = EnvBool(False)
    SGLANG_TEST_REQUEST_TIME_STATS = EnvBool(False)
    SGLANG_DISABLE_TP_MEMORY_INBALANCE_CHECK = EnvBool(False)
    SGLANG_SIMULATE_ACC_LEN = EnvFloat(-1)
    SGLANG_SIMULATE_ACC_METHOD = EnvStr("match-expected")
    SGLANG_TORCH_PROFILER_DIR = EnvStr("/tmp")
    SGLANG_OTLP_EXPORTER_SCHEDULE_DELAY_MILLIS = EnvInt(500)
    SGLANG_OTLP_EXPORTER_MAX_EXPORT_BATCH_SIZE = EnvInt(64)
    SGLANG_NATIVE_MOVE_KV_CACHE = EnvBool(False)
    SGLANG_ENABLE_TP_MEMORY_INBALANCE_CHECK = EnvBool(True)

    # Scheduler: memory leak test
    SGLANG_TEST_RETRACT = EnvBool(False)
    SGLANG_TEST_RETRACT_INTERVAL = EnvInt(3)
    SGLANG_TEST_RETRACT_NO_PREFILL_BS = EnvInt(2 ** 31)
    SGLANG_ENABLE_STRICT_MEM_CHECK_DURING_BUSY = EnvInt(0)
```
**EN:** This range introduces `Envs` and defines the structure or metadata that its methods rely on. This chunk is part 1 of 9 for the same logical block. In this range it reads environment-driven configuration; handles grammar or regular-expression constraints; manages model weights or checkpoints.
**CN:** 这一段引入 `Envs`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 1/9 部分。 在这一范围内，它会读取环境变量驱动的配置；处理语法或正则约束；管理模型权重或检查点。

### Lines 219-278: Class Envs (part 2/9)
```python
    SGLANG_ENABLE_STRICT_MEM_CHECK_DURING_IDLE = EnvBool(True)

    # Scheduler: new token ratio hyperparameters
    SGLANG_INIT_NEW_TOKEN_RATIO = EnvFloat(0.7)
    SGLANG_MIN_NEW_TOKEN_RATIO_FACTOR = EnvFloat(0.14)
    SGLANG_NEW_TOKEN_RATIO_DECAY_STEPS = EnvInt(600)
    SGLANG_RETRACT_DECODE_STEPS = EnvInt(20)
    SGLANG_CLIP_MAX_NEW_TOKENS_ESTIMATION = EnvInt(4096)

    # Scheduler: recv interval
    SGLANG_SCHEDULER_RECV_SKIPPER_WEIGHT_DEFAULT = EnvInt(1000)
    SGLANG_SCHEDULER_RECV_SKIPPER_WEIGHT_DECODE = EnvInt(1)
    SGLANG_SCHEDULER_RECV_SKIPPER_WEIGHT_TARGET_VERIFY = EnvInt(1)
    SGLANG_SCHEDULER_RECV_SKIPPER_WEIGHT_NONE = EnvInt(1)

    # PD Disaggregation (runtime)
    # NOTE: For SGLANG_DISAGGREGATION_THREAD_POOL_SIZE, the effective default is
    # computed dynamically at runtime based on cpu_count; see disaggregation backends.
    SGLANG_DISAGGREGATION_THREAD_POOL_SIZE = EnvInt(None)
    SGLANG_DISAGGREGATION_QUEUE_SIZE = EnvInt(4)
    SGLANG_DISAGGREGATION_BOOTSTRAP_TIMEOUT = EnvInt(300)
    SGLANG_DISAGGREGATION_HEARTBEAT_INTERVAL = EnvFloat(5.0)
    SGLANG_DISAGGREGATION_HEARTBEAT_MAX_FAILURE = EnvInt(2)
    SGLANG_DISAGGREGATION_WAITING_TIMEOUT = EnvInt(300)
    SGLANG_DISAGGREGATION_NIXL_BACKEND = EnvStr("UCX")
    SGLANG_DISAGGREGATION_NIXL_BACKEND_PARAMS = EnvStr("{}")
    SGLANG_DISAGGREGATION_ALL_CP_RANKS_TRANSFER = EnvBool(False)
    SGLANG_DISAGGREGATION_FORCE_QUERY_PREFILL_DP_RANK = EnvBool(False)
    # Extra slots in req_to_token_pool for decode workers (only effective when
    # max_num_reqs > 32). Increases pool capacity so more KV cache transfers
    # can overlap with decode execution without raising max_running_requests.
    SGLANG_DISAGGREGATION_NUM_PRE_ALLOCATE_REQS = EnvInt(0)

    # Scheduler: others:
    SGLANG_EMPTY_CACHE_INTERVAL = EnvFloat(-1)  # in seconds. Set if you observe high memory accumulation over a long serving period.
    SGLANG_DISABLE_CONSECUTIVE_PREFILL_OVERLAP = EnvBool(False)
    SGLANG_SCHEDULER_MAX_RECV_PER_POLL = EnvInt(-1)
    SGLANG_EXPERIMENTAL_CPP_RADIX_TREE = EnvBool(False)
    SGLANG_RADIX_FORCE_MISS = EnvBool(False)
    SGLANG_DYNAMIC_CHUNKING_SMOOTH_FACTOR = EnvFloat(0.75)
    SGLANG_SCHEDULER_SKIP_ALL_GATHER = EnvBool(False)
    SGLANG_SCHEDULER_DECREASE_PREFILL_IDLE = EnvBool(False)
    SGLANG_PREFILL_DELAYER_MAX_DELAY_PASSES = EnvInt(None)
    SGLANG_PREFILL_DELAYER_TOKEN_USAGE_LOW_WATERMARK = EnvFloat(None)
    SGLANG_DATA_PARALLEL_BUDGET_INTERVAL = EnvInt(1)
    SGLANG_REQ_WAITING_TIMEOUT = EnvFloat(-1)  # in seconds
    SGLANG_NCCL_ALL_GATHER_IN_OVERLAP_SCHEDULER_SYNC_BATCH = EnvBool(False)
    SGLANG_REQ_RUNNING_TIMEOUT = EnvFloat(-1)  # in seconds
    SGLANG_DISAGGREGATION_BOOTSTRAP_ENTRY_CLEANUP_INTERVAL = EnvInt(120)
    SGLANG_SWA_EVICTION_INTERVAL_MULTIPLIER = EnvFloat(1.0)
    # For non-streaming requests, the scheduler still flushes intermediate
    # output batches to the tokenizer manager every N decoded tokens so that
    # `first_token_time`/TTFT can be recorded. Lower this (e.g. to 1) to get
    # an accurate TTFT for benchmarking; the upstream default of 50 trades
    # off some TTFT-metric accuracy for less IPC overhead.
    SGLANG_FORCE_STREAM_INTERVAL = EnvInt(50)

    # Test: pd-disaggregation
    SGLANG_TEST_PD_DISAGG_BACKEND = EnvStr("mooncake")
    SGLANG_TEST_PD_DISAGG_DEVICES = EnvStr(None)
```
**EN:** This range introduces `Envs` and defines the structure or metadata that its methods rely on. This chunk is part 2 of 9 for the same logical block. In this range it reads environment-driven configuration; records metrics or tracing signals; manages model weights or checkpoints.
**CN:** 这一段引入 `Envs`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 2/9 部分。 在这一范围内，它会读取环境变量驱动的配置；记录指标或追踪信号；管理模型权重或检查点。

### Lines 279-338: Class Envs (part 3/9)
```python

    # Model Parallel
    SGLANG_USE_MESSAGE_QUEUE_BROADCASTER = EnvBool(True)
    SGLANG_ONE_VISIBLE_DEVICE_PER_PROCESS = EnvBool(False)
    # Override the distributed init method used by torch.distributed.init_process_group.
    # Set to "env://" to use an externally-created TCPStore via MASTER_ADDR/MASTER_PORT.
    SGLANG_DISTRIBUTED_INIT_METHOD_OVERRIDE = EnvStr(None)
    SGLANG_TCP_STORE_PORT = EnvInt(29600)

    # Tool Calling
    SGLANG_FORWARD_UNKNOWN_TOOLS = EnvBool(False)

    # Hi-Cache
    SGLANG_HICACHE_HF3FS_CONFIG_PATH = EnvStr(None)
    SGLANG_HICACHE_DECODE_OFFLOAD_STRIDE = EnvInt(None)
    SGLANG_HICACHE_FILE_BACKEND_STORAGE_DIR = EnvStr(None)
    SGLANG_HICACHE_NIXL_BACKEND_STORAGE_DIR = EnvStr(None)
    # Staging buffer for heterogeneous TP KV transfer
    SGLANG_DISAGG_STAGING_BUFFER = EnvBool(False)
    SGLANG_DISAGG_STAGING_BUFFER_SIZE_MB = EnvInt(64)
    SGLANG_DISAGG_STAGING_POOL_SIZE_MB = EnvInt(4096)
    # TODO(yangminl): remove SGLANG_STAGING_USE_TORCH and the torch fallback in
    # staging_buffer.py once Triton kernels are fully validated in production.
    SGLANG_STAGING_USE_TORCH = EnvBool(False)
    # Mooncake KV Transfer
    SGLANG_MOONCAKE_CUSTOM_MEM_POOL = EnvStr(None)
    ENABLE_ASCEND_TRANSFER_WITH_MOONCAKE = EnvBool(False)
    ASCEND_NPU_PHY_ID = EnvInt(-1)
    SGLANG_MOONCAKE_SEND_AUX_TCP = EnvBool(False)

    # Mooncake Store
    SGLANG_HICACHE_MOONCAKE_CONFIG_PATH = EnvStr(None)
    SGLANG_HICACHE_MOONCAKE_REUSE_TE = EnvBool(True)
    MOONCAKE_MASTER = EnvStr(None)
    MOONCAKE_CLIENT = EnvStr(None)
    MOONCAKE_LOCAL_HOSTNAME = EnvStr("localhost")
    MOONCAKE_TE_META_DATA_SERVER = EnvStr("P2PHANDSHAKE")
    MOONCAKE_GLOBAL_SEGMENT_SIZE = EnvStr("4gb")
    MOONCAKE_PROTOCOL = EnvStr("tcp")
    MOONCAKE_DEVICE = EnvStr("")
    MOONCAKE_MASTER_METRICS_PORT = EnvInt(9003)
    MOONCAKE_CHECK_SERVER = EnvBool(False)
    MOONCAKE_STANDALONE_STORAGE = EnvBool(False)
    MOONCAKE_ENABLE_SSD_OFFLOAD = EnvBool(False)
    MOONCAKE_OFFLOAD_FILE_STORAGE_PATH = EnvStr(None)

    # AMD & ROCm
    SGLANG_USE_AITER = EnvBool(False)
    SGLANG_USE_AITER_UNIFIED_ATTN = EnvBool(False)
    SGLANG_ROCM_FUSED_DECODE_MLA = EnvBool(False)
    SGLANG_ROCM_DISABLE_LINEARQUANT = EnvBool(False)
    SGLANG_MORI_NUM_MAX_DISPATCH_TOKENS_PER_RANK = EnvInt(4096)
    # Enable dual-stream MoE (shared experts vs routed experts) on the
    # ROCm/AITER path. Requires GPU_MAX_HW_QUEUES>=5 to avoid HW-queue serialization.
    SGLANG_ROCM_USE_MULTI_STREAM = EnvBool(False)

    # MPS (Apple Silicon)
    SGLANG_USE_MLX = EnvBool(False)

    # NPU
```
**EN:** This range introduces `Envs` and defines the structure or metadata that its methods rely on. This chunk is part 3 of 9 for the same logical block. In this range it coordinates distributed communication; reads environment-driven configuration; records metrics or tracing signals.
**CN:** 这一段引入 `Envs`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 3/9 部分。 在这一范围内，它会协调分布式通信；读取环境变量驱动的配置；记录指标或追踪信号。

### Lines 339-398: Class Envs (part 4/9)
```python
    SGLANG_NPU_DISABLE_ACL_FORMAT_WEIGHT = EnvBool(False)
    SGLANG_NPU_USE_MULTI_STREAM = EnvBool(False)
    SGLANG_NPU_USE_MLAPO = EnvBool(False)
    # Forward native implementation for activation gelu tanh for model Skywork-Reward-Gemma-2-27B-v0.2
    SGLANG_NPU_FORWARD_NATIVE_GELUTANH = EnvBool(False)
    # Forward native implementation for gemma rms norm for model Skywork-Reward-Gemma-2-27B-v0.2
    SGLANG_NPU_FORWARD_NATIVE_GEMMA_RMS_NORM = EnvBool(False)
    # Delay all-gather after qlora for better performance for Deepseek v3.2
    SGLANG_USE_AG_AFTER_QLORA = EnvBool(False)
    # Quantize x to int8 in the dispatch operator
    DEEP_NORMAL_MODE_USE_INT8_QUANT = EnvBool(False) # This argument is deprecated
    SGLANG_NPU_FUSED_MOE_MODE = EnvInt(1)

    # MTHREADS & MUSA
    SGLANG_MUSA_FA3_FORCE_UPDATE_METADATA = EnvBool(False)

    # Quantization
    SGLANG_INT4_WEIGHT = EnvBool(False)
    SGLANG_CPU_QUANTIZATION = EnvBool(False)
    SGLANG_USE_DYNAMIC_MXFP4_LINEAR = EnvBool(False)
    SGLANG_FORCE_FP8_MARLIN = EnvBool(False)
    SGLANG_MOE_NVFP4_DISPATCH = EnvBool(False)
    SGLANG_NVFP4_CKPT_FP8_GEMM_IN_ATTN = EnvBool(False)
    SGLANG_NVFP4_CKPT_FP8_NEXTN_MOE = EnvBool(False)
    SGLANG_QUANT_ALLOW_DOWNCASTING = EnvBool(False)
    SGLANG_FP8_IGNORED_LAYERS = EnvStr("")

    # Flashinfer
    SGLANG_IS_FLASHINFER_AVAILABLE = EnvBool(True)
    SGLANG_FLASHINFER_USE_PAGED = EnvBool(False)
    # Default to the pick from flashinfer
    SGLANG_FLASHINFER_WORKSPACE_SIZE = EnvInt(384 * 1024 * 1024)
    # Skip-softmax threshold scale factor for TRT-LLM attention (prefill and decode separately).
    # None = standard attention. See https://arxiv.org/abs/2512.12087
    SGLANG_SKIP_SOFTMAX_PREFILL_THRESHOLD_SCALE_FACTOR = EnvFloat(None)
    SGLANG_SKIP_SOFTMAX_DECODE_THRESHOLD_SCALE_FACTOR = EnvFloat(None)
    # TODO(mmangkad): Remove this once the FlashInfer unified allreduce-fusion
    # transport issue on GB200/GB300 platforms is fixed and verified resolved.
    SGLANG_FLASHINFER_FORCE_POSIX_FD_TRANSPORT = EnvBool(None)

    # Triton
    SGLANG_TRITON_DECODE_ATTN_STATIC_KV_SPLITS = EnvBool(False)
    SGLANG_USE_CUSTOM_TRITON_KERNEL_CACHE = EnvBool(False)

    # Torch Compile
    SGLANG_ENABLE_TORCH_COMPILE = EnvBool(False)

    # EPLB
    SGLANG_EXPERT_LOCATION_UPDATER_LOG_INPUT = EnvBool(False)
    SGLANG_EXPERT_LOCATION_UPDATER_CANARY = EnvBool(False)
    SGLANG_EXPERT_LOCATION_UPDATER_LOG_METRICS = EnvBool(False)
    SGLANG_LOG_EXPERT_LOCATION_METADATA = EnvBool(False)
    SGLANG_EXPERT_DISTRIBUTION_RECORDER_DIR = EnvStr("/tmp")
    SGLANG_EPLB_HEATMAP_COLLECTION_INTERVAL = EnvInt(0)
    SGLANG_ENABLE_EPLB_BALANCEDNESS_METRIC = EnvBool(False)

    # TBO
    SGLANG_TBO_DEBUG = EnvBool(False)

    # DeepGemm
```
**EN:** This range introduces `Envs` and defines the structure or metadata that its methods rely on. This chunk is part 4 of 9 for the same logical block. In this range it sets up imports and shared symbols; reads environment-driven configuration; records metrics or tracing signals.
**CN:** 这一段引入 `Envs`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 4/9 部分。 在这一范围内，它会建立导入关系并准备共享符号；读取环境变量驱动的配置；记录指标或追踪信号。

### Lines 399-458: Class Envs (part 5/9)
```python
    SGLANG_ENABLE_JIT_DEEPGEMM = EnvBool(True)
    SGLANG_JIT_DEEPGEMM_PRECOMPILE = EnvBool(True)
    SGLANG_JIT_DEEPGEMM_FAST_WARMUP = EnvBool(False)
    SGLANG_JIT_DEEPGEMM_COMPILE_WORKERS = EnvInt(4)
    SGLANG_IN_DEEPGEMM_PRECOMPILE_STAGE = EnvBool(False)
    SGLANG_DG_CACHE_DIR = EnvStr(os.path.expanduser("~/.cache/deep_gemm"))
    SGLANG_DG_USE_NVRTC = EnvBool(False)
    SGLANG_USE_DEEPGEMM_BMM = EnvBool(False)
    SGLANG_DEEPGEMM_SANITY_CHECK = EnvBool(False)

    # DeepSeek MHA Optimization
    SGLANG_CHUNKED_PREFIX_CACHE_THRESHOLD = EnvInt(8192)
    SGLANG_MAX_KV_CHUNK_CAPACITY = EnvInt(128 * 1024)

    # DeepEP
    SGLANG_DEEPEP_BF16_DISPATCH = EnvBool(False) # This argument is deprecated
    SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK = EnvInt(128)
    SGLANG_DEEPEP_LL_COMBINE_SEND_NUM_SMS = EnvInt(32)
    SGLANG_BLACKWELL_OVERLAP_SHARED_EXPERTS_OUTSIDE_SBO = EnvBool(False)
    # Force dynamic DeepEP Waterfill with runtime EP all-reduce instead of the
    # default static local-batch path.
    SGLANG_DISABLE_STATIC_WATERFILL = EnvBool(False)

    # NIXL-EP
    SGLANG_NIXL_EP_BF16_DISPATCH = EnvBool(False)
    SGLANG_NIXL_EP_NUM_MAX_DISPATCH_TOKENS_PER_RANK = EnvInt(128)

    # NSA Backend
    SGLANG_NSA_FUSE_TOPK = EnvBool(True)
    SGLANG_NSA_ENABLE_MTP_PRECOMPUTE_METADATA = EnvBool(True)
    SGLANG_USE_FUSED_METADATA_COPY = EnvBool(True)
    SGLANG_NSA_PREFILL_DENSE_ATTN_KV_LEN_THRESHOLD = EnvInt(2048)

    # sgl-kernel
    SGLANG_SKIP_SGL_KERNEL_VERSION_CHECK = EnvBool(False)

    # Flash Attention
    SGLANG_USE_SGL_FA3_KERNEL = EnvBool(True)

    # Kernels
    USE_TRITON_W8A8_FP8_KERNEL = EnvBool(False)
    SGLANG_RETURN_ORIGINAL_LOGPROB = EnvBool(False)
    SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN = EnvBool(False)
    SGLANG_MOE_PADDING = EnvBool(False)
    SGLANG_CUTLASS_MOE = EnvBool(False)
    HF_HUB_DISABLE_XET = EnvBool(False)
    DISABLE_OPENAPI_DOC = EnvBool(False)
    SGLANG_ENABLE_TORCH_INFERENCE_MODE = EnvBool(False)
    SGLANG_IS_FIRST_RANK_ON_NODE = EnvBool(True)
    SGLANG_SYNC_TOKEN_IDS_ACROSS_TP = EnvBool(False)
    SGLANG_ENABLE_COLOCATED_BATCH_GEN = EnvBool(False)

    # Deterministic inference
    SGLANG_ENABLE_DETERMINISTIC_INFERENCE = EnvBool(False)
    # Use 1-stage all-reduce kernel on AMD (deterministic, fixed accumulation order)
    # If not set: auto (enabled when --enable-deterministic-inference is on)
    # Set to 1: force enable (even without --enable-deterministic-inference)
    # Set to 0: force disable (use default Aiter AR even with --enable-deterministic-inference)
    SGLANG_USE_1STAGE_ALLREDUCE = EnvBool(False)
    SGLANG_OPT_USE_CUSTOM_ALL_REDUCE_V2 = EnvBool(True)
```
**EN:** This range introduces `Envs` and defines the structure or metadata that its methods rely on. This chunk is part 5 of 9 for the same logical block. In this range it coordinates distributed communication; reads environment-driven configuration; prepares compilation-related behavior.
**CN:** 这一段引入 `Envs`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 5/9 部分。 在这一范围内，它会协调分布式通信；读取环境变量驱动的配置；处理与编译相关的行为。

### Lines 459-518: Class Envs (part 6/9)
```python
    SGLANG_FLASHINFER_PREFILL_SPLIT_TILE_SIZE = EnvInt(4096)
    SGLANG_FLASHINFER_DECODE_SPLIT_TILE_SIZE = EnvInt(2048)
    SGLANG_TRITON_PREFILL_TRUNCATION_ALIGN_SIZE = EnvInt(4096)
    SGLANG_TRITON_DECODE_SPLIT_TILE_SIZE = EnvInt(256)

    # RoPE cache configuration
    SGLANG_SPEC_EXPANSION_SAFETY_FACTOR = EnvInt(2)
    SGLANG_ROPE_CACHE_SAFETY_MARGIN = EnvInt(256)
    SGLANG_ROPE_CACHE_ALIGN = EnvInt(128)

    # Overlap Spec V2
    SGLANG_ENABLE_SPEC_V2 = EnvBool(True)
    SGLANG_ENABLE_OVERLAP_PLAN_STREAM = EnvBool(False)

    # Spec Config
    SGLANG_SPEC_ENABLE_STRICT_FILTER_CHECK = EnvBool(True)
    SGLANG_SPEC_NAN_DETECTION = EnvBool(False)
    SGLANG_SPEC_OOB_DETECTION = EnvBool(False)

    # VLM
    SGLANG_VLM_CACHE_SIZE_MB = EnvInt(100)
    SGLANG_IMAGE_MAX_PIXELS = EnvInt(16384 * 28 * 28)
    SGLANG_RESIZE_RESAMPLE = EnvStr("")
    SGLANG_MM_BUFFER_SIZE_MB = EnvInt(0)
    SGLANG_MM_PRECOMPUTE_HASH = EnvBool(False)
    SGLANG_VIT_ENABLE_CUDA_GRAPH = EnvBool(False)
    SGLANG_MM_SKIP_COMPUTE_HASH = EnvBool(False)


    # VLM Item CUDA IPC Transport
    SGLANG_USE_CUDA_IPC_TRANSPORT = EnvBool(False)
    SGLANG_USE_IPC_POOL_HANDLE_CACHE = EnvBool(False)
    SGLANG_MM_FEATURE_CACHE_MB = EnvInt(1 * 1024)
    SGLANG_MM_ITEM_MEM_POOL_RECYCLE_INTERVAL_SEC = EnvFloat(0.05)

    # Mamba
    SGLANG_MAMBA_CONV_DTYPE = EnvStr("bfloat16")
    SGLANG_MAMBA_SSM_DTYPE = EnvStr(None)

    # Unified Radix Tree
    SGLANG_ENABLE_UNIFIED_RADIX_TREE = EnvBool(False)

    # Breakable CUDA Graph
    SGLANG_USE_BREAKABLE_CUDA_GRAPH = EnvBool(False)

    # Release & Resume Memory
    SGLANG_MEMORY_SAVER_CUDA_GRAPH = EnvBool(False)

    # Sparse Embeddings
    SGLANG_EMBEDDINGS_SPARSE_HEAD = EnvStr(None)

    # Logits processor
    SGLANG_ENABLE_LOGITS_PROCESSER_CHUNK = EnvBool(False)
    SGLANG_LOGITS_PROCESSER_CHUNK_SIZE = EnvInt(2048)

    # Tool-Call behavior
    SGLANG_TOOL_STRICT_LEVEL = EnvInt(ToolStrictLevel.OFF)

    # Think tokens budget: negative means unlimited, >= 0 caps thinking tokens
    SGLANG_MAX_THINK_TOKENS = EnvInt(-1)
```
**EN:** This range introduces `Envs` and defines the structure or metadata that its methods rely on. This chunk is part 6 of 9 for the same logical block. In this range it reads environment-driven configuration; manages graph capture or replay logic.
**CN:** 这一段引入 `Envs`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 6/9 部分。 在这一范围内，它会读取环境变量驱动的配置；管理图捕获或回放逻辑。

### Lines 519-578: Class Envs (part 7/9)
```python

    # Ngram
    SGLANG_NGRAM_FORCE_GREEDY_VERIFY = EnvBool(False)

    # Warmup
    SGLANG_WARMUP_TIMEOUT = EnvFloat(-1) # in seconds. If a warmup forward batch takes longer than this, the server will crash to prevent hanging. Recommend to increase warmup timeout to 1800 to accommodate some kernel JIT precache e.g. deep gemm

    # HTTP Server
    SGLANG_TIMEOUT_KEEP_ALIVE = EnvInt(5)
    # Uvicorn multiprocess supervisor pings each worker on this interval; default 5s is
    # too short when many workers cold-start and load tokenizers in parallel.
    SGLANG_UVICORN_WORKER_HEALTHCHECK_TIMEOUT = EnvInt(10)

    # HTTP/2 Server
    SGLANG_GRANIAN_PARENT_PID = EnvInt(None)

    # Health Check
    SGLANG_ENABLE_HEALTH_ENDPOINT_GENERATION = EnvBool(True)

    # Encoder gRPC
    SGLANG_ENCODER_GRPC_TIMEOUT_SECS = EnvInt(60)
    # Encoder receiver selection: http|grpc (used by EPD paths).
    SGLANG_ENCODER_MM_RECEIVER_MODE = EnvStr("http")

    # Native gRPC server (internal, not yet user-facing)
    SGLANG_GRPC_PORT = EnvInt(None)
    SGLANG_ENABLE_GRPC = EnvBool(False)

    # External models
    SGLANG_EXTERNAL_MODEL_PACKAGE = EnvStr("")
    SGLANG_EXTERNAL_MM_MODEL_ARCH = EnvStr("")
    SGLANG_EXTERNAL_MM_PROCESSOR_PACKAGE = EnvStr("")

    # Numa
    SGLANG_NUMA_BIND_V2 = EnvBool(True)
    SGLANG_AUTO_NUMA_BIND = EnvBool(False)

    # Metrics
    SGLANG_ENABLE_METRICS_DEVICE_TIMER = EnvBool(False)
    SGLANG_ENABLE_METRICS_DP_ATTENTION = EnvBool(False)

    # Tokenizer (Kimi tiktoken: cache all_special_tokens / all_special_ids; the ITL can differ by +10x under high batch size).
    SGLANG_PATCH_TOKENIZER = EnvBool(True)

    # TokenizerManager
    SGLANG_REQUEST_STATE_WAIT_TIMEOUT = EnvInt(4)

    # ZBAL, zero buffer accelerate library, currently worked only in npu
    SGLANG_ZBAL_LOCAL_MEM_SIZE = EnvInt(0)
    SGLANG_ZBAL_BOOTSTRAP_URL = EnvStr("")

    SGLANG_DEFAULT_THINKING = EnvBool(False)

    # ====================================================================
    # DeepSeek V4
    # ====================================================================

    # Set False when using FP4-to-FP8 converted DeepSeek V4 checkpoint.
    SGLANG_DSV4_FP4_EXPERTS = EnvBool(True)
    # Default reasoning_effort for dsv4 chat encoder when request doesn't set it.
```
**EN:** This range introduces `Envs` and defines the structure or metadata that its methods rely on. This chunk is part 7 of 9 for the same logical block. In this range it reads environment-driven configuration; records metrics or tracing signals; manages model weights or checkpoints.
**CN:** 这一段引入 `Envs`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 7/9 部分。 在这一范围内，它会读取环境变量驱动的配置；记录指标或追踪信号；管理模型权重或检查点。

### Lines 579-638: Class Envs (part 8/9)
```python
    # Accepts "", "max", "high" (empty string means unset); other values filtered to None.
    SGLANG_DSV4_REASONING_EFFORT = EnvStr("")

    # CUDA kernels
    SGLANG_OPT_DEEPGEMM_HC_PRENORM = EnvBool(True)
    SGLANG_OPT_USE_TILELANG_MHC_PRE = EnvBool(True)
    SGLANG_OPT_USE_TILELANG_MHC_POST = EnvBool(True)
    SGLANG_OPT_USE_TILELANG_INDEXER = EnvBool(False)
    SGLANG_OPT_USE_JIT_INDEXER_METADATA = EnvBool(True)
    SGLANG_OPT_USE_ONLINE_COMPRESS = EnvBool(False)
    SGLANG_OPT_USE_COMPRESSOR_V2 = EnvBool(True)
    SGLANG_FP8_PAGED_MQA_LOGITS_TORCH = EnvBool(False)
    SGLANG_TOPK_TRANSFORM_512_TORCH = EnvBool(False)

    # SWA radix cache
    SGLANG_OPT_CACHE_SWA_TRANSLATION = EnvBool(True)
    # TODO(DSV4): @ispobock this has bug on main branch when retract
    SGLANG_OPT_SWA_RADIX_CACHE_COMPACT = EnvBool(False)
    SGLANG_OPT_SWA_SPLIT_LEAF_ON_INSERT = EnvBool(False)
    SGLANG_OPT_SWA_RELEASE_LEAF_LOCK_AFTER_WINDOW = EnvBool(False)
    SGLANG_OPT_SWA_EVICT_DROP_PAGE_MARGIN = EnvBool(False)

    # DeepGemm Mega MoE
    SGLANG_OPT_USE_DEEPGEMM_MEGA_MOE = EnvBool(False)
    SGLANG_OPT_DEEPGEMM_MEGA_MOE_NUM_MAX_TOKENS_PER_RANK = EnvInt(1024)

    # When set, the mega-MoE x slot is packed E2M1 (FP4) instead of FP8 E4M3.
    # Halves symm-buffer footprint and unlocks the MXF4 mainloop downstream.
    # Setting this also exports DG_USE_FP4_ACTS=1 so DeepGEMM's symm-buffer
    # sizing + fp8_fp4_mega_moe pick up the FP4 layout.
    SGLANG_OPT_DEEPGEMM_MEGA_MOE_USE_FP4_ACTS = EnvBool(False)
    # Switches the L1+L2 mainloops from kind::mxf8f6f4 (K=32 with-padding) to
    # kind::mxf4 (K=64 dense) inside fp8_fp4_mega_moe. No effect unless
    # SGLANG_OPT_DEEPGEMM_MEGA_MOE_USE_FP4_ACTS is also set; DeepGEMM asserts
    # this combination on the host side.
    SGLANG_OPT_DEEPGEMM_MEGA_MOE_USE_MXF4_KIND = EnvBool(False)
    SGLANG_OPT_FIX_MEGA_MOE_MEMORY = EnvBool(False)

    # TopK
    SGLANG_OPT_USE_FUSED_HASH_TOPK = EnvBool(True)
    SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK = EnvBool(True)
    SGLANG_OPT_USE_TOPK_V2 = EnvBool(True)

    # GEMM / kernel fusion
    SGLANG_OPT_FP8_WO_A_GEMM = EnvBool(True)
    SGLANG_OPT_BF16_FP32_GEMM_ALGO = EnvStr("cublas")
    SGLANG_OPT_USE_JIT_EP_ACTIVATION = EnvBool(True)
    SGLANG_OPT_FUSE_WQA_WKV = EnvBool(True)
    SGLANG_OPT_SWIGLU_CLAMP_FUSION = EnvBool(True)

    # Cache / overlap
    SGLANG_OPT_USE_FUSED_STORE_CACHE = EnvBool(True)
    SGLANG_OPT_USE_MULTI_STREAM_OVERLAP = EnvBool(True)

    # CUDA graph
    SGLANG_PREP_IN_CUDA_GRAPH = EnvBool(True)

    # Distributed
    SGLANG_DSV4_FIX_TP_ATTN_A2A_SCATTER = EnvBool(True)
    SGLANG_SHARED_EXPERT_TP1 = EnvBool(False)
```
**EN:** This range introduces `Envs` and defines the structure or metadata that its methods rely on. This chunk is part 8 of 9 for the same logical block. In this range it sets up imports and shared symbols; reads environment-driven configuration; manages graph capture or replay logic.
**CN:** 这一段引入 `Envs`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 8/9 部分。 在这一范围内，它会建立导入关系并准备共享符号；读取环境变量驱动的配置；管理图捕获或回放逻辑。

### Lines 639-660: Class Envs (part 9/9)
```python
    # Symmetric Memory
    SGLANG_SYMM_MEM_PREALLOC_GB_SIZE = EnvInt(-1)
    SGLANG_DEBUG_SYMM_MEM = EnvBool(False)

    # Aiter
    SGLANG_USE_AITER_FP8_PER_TOKEN = EnvBool(False)
    # fmt: on

    # EPD
    SGLANG_ENCODER_RECV_TIMEOUT = EnvFloat(180.0)
    SGLANG_ENCODER_SEND_TIMEOUT = EnvFloat(180.0)
    SGLANG_ENCODER_DISPATCH_MIN_ITEMS = EnvInt(2)

    # Elastic EP Backup Port
    SGLANG_BACKUP_PORT_BASE = EnvInt(10000)

    # Sglang Cache Dir
    SGLANG_CACHE_DIR = EnvStr(os.path.expanduser("~/.cache/sglang"))

    # Plugin system
    SGLANG_PLATFORM = EnvStr("")
    SGLANG_PLUGINS = EnvStr("")
```
**EN:** This range introduces `Envs` and defines the structure or metadata that its methods rely on. This chunk is part 9 of 9 for the same logical block. In this range it reads environment-driven configuration; records metrics or tracing signals.
**CN:** 这一段引入 `Envs`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 9/9 部分。 在这一范围内，它会读取环境变量驱动的配置；记录指标或追踪信号。

### Lines 661-666: Module-level constants and helpers
```python


envs = Envs()
EnvField._allow_set_name = False


```
**EN:** This range organizes module-level state and shared setup. In this range it reads environment-driven configuration.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 667-675: Function _print_deprecated_env
```python
def _print_deprecated_env(old_name: str, new_name: Optional[str] = None):
    if old_name in os.environ:
        if new_name is None:
            warnings.warn(f"Environment variable {old_name} has been deprecated.")
        else:
            warnings.warn(
                f"Environment variable {old_name} will be deprecated, please use {new_name} instead"
            )
            os.environ[new_name] = os.environ[old_name]
```
**EN:** This callable implements `_print_deprecated_env`. It takes `old_name`, `new_name` and mainly emits or formats diagnostic output. In this range it reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `_print_deprecated_env`。它接收 `old_name`, `new_name`，主要用于输出或格式化诊断信息。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 678-684: Function _warn_deprecated_env_to_cli_flag
```python
def _warn_deprecated_env_to_cli_flag(env_name: str, suggestion: str):
    """Warn when a deprecated environment variable is used.

    This is for env vars that are deprecated in favor of CLI flags.
    """
    if env_name in os.environ:
        warnings.warn(f"Environment variable {env_name} is deprecated. {suggestion}")
```
**EN:** This callable implements `_warn_deprecated_env_to_cli_flag`. It takes `env_name`, `suggestion` and mainly converts data into another representation. The docstring states: "Warn when a deprecated environment variable is used." In this range it reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `_warn_deprecated_env_to_cli_flag`。它接收 `env_name`, `suggestion`，主要用于将数据转换为另一种表示。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 687-721: Function _convert_SGL_to_SGLANG
```python
def _convert_SGL_to_SGLANG():
    _print_deprecated_env("SGLANG_GC_LOG", "SGLANG_LOG_GC")
    _print_deprecated_env(
        "SGLANG_CUTEDSL_MOE_NVFP4_DISPATCH", "SGLANG_MOE_NVFP4_DISPATCH"
    )
    _print_deprecated_env(
        "SGL_DISABLE_TP_MEMORY_INBALANCE_CHECK",
        "SGLANG_ENABLE_TP_MEMORY_INBALANCE_CHECK",
    )
    _print_deprecated_env("SGLANG_PER_TOKEN_GROUP_QUANT_8BIT_V2")
    _print_deprecated_env("SGLANG_ENABLE_THINKING", "SGLANG_DEFAULT_THINKING")
    _print_deprecated_env("SGLANG_REASONING_EFFORT", "SGLANG_DSV4_REASONING_EFFORT")
    _print_deprecated_env(
        "SGLANG_USE_JIT_ALL_REDUCE", "SGLANG_OPT_USE_CUSTOM_ALL_REDUCE_V2"
    )
    _deprecated_ms_to_s = {
        "SGLANG_QUEUED_TIMEOUT_MS": "SGLANG_REQ_WAITING_TIMEOUT",
        "SGLANG_FORWARD_TIMEOUT_MS": "SGLANG_REQ_RUNNING_TIMEOUT",
    }
    for old_name, new_name in _deprecated_ms_to_s.items():
        if old_name in os.environ:
            ms_val = os.environ[old_name]
            warnings.warn(
                f"Environment variable {old_name} (in ms) is deprecated, "
                f"please use {new_name} (in seconds) instead"
            )
            os.environ[new_name] = str(float(ms_val) / 1000.0)

    for key, value in os.environ.items():
        if key.startswith("SGL_"):
            new_key = key.replace("SGL_", "SGLANG_", 1)
            warnings.warn(
                f"Environment variable {key} is deprecated, please use {new_key}"
            )
            os.environ[new_key] = value
```
**EN:** This callable implements `_convert_SGL_to_SGLANG` and mainly converts data into another representation. In this range it coordinates distributed communication; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `_convert_SGL_to_SGLANG`，主要用于将数据转换为另一种表示。 在这一范围内，它会协调分布式通信；读取环境变量驱动的配置。

### Lines 722-743: Module imports, constants, and setup
```python


_convert_SGL_to_SGLANG()
_warn_deprecated_env_to_cli_flag(
    "SGLANG_SCHEDULER_DECREASE_PREFILL_IDLE",
    "Please use '--enable-prefill-delayer' instead.",
)
_warn_deprecated_env_to_cli_flag(
    "SGLANG_PREFILL_DELAYER_MAX_DELAY_PASSES",
    "Please use '--prefill-delayer-max-delay-passes' instead.",
)
_warn_deprecated_env_to_cli_flag(
    "SGLANG_PREFILL_DELAYER_TOKEN_USAGE_LOW_WATERMARK",
    "Please use '--prefill-delayer-token-usage-low-watermark' instead.",
)

# Import cuda_coredump to trigger auto-injection of CUDA env vars
# when SGLANG_CUDA_COREDUMP=1. Best-effort; for strict guarantees,
# set CUDA_* env vars in the shell before launching Python.
import sglang.srt.debug_utils.cuda_coredump  # noqa: F401, E402


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 744-750: Function example_with_exit_stack
```python
def example_with_exit_stack():
    # Use this style of context manager in unit test
    exit_stack = ExitStack()
    exit_stack.enter_context(envs.SGLANG_TEST_RETRACT.override(False))
    assert envs.SGLANG_TEST_RETRACT.get() is False
    exit_stack.close()
    assert envs.SGLANG_TEST_RETRACT.get() is None
```
**EN:** This callable implements `example_with_exit_stack` and mainly implements example with exit stack. In this range it performs defensive checks on invalid state; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `example_with_exit_stack`，主要用于实现 example with exit stack 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；读取环境变量驱动的配置。

### Lines 753-765: Function example_with_subprocess
```python
def example_with_subprocess():
    command = ["python", "-c", "import os; print(os.getenv('SGLANG_TEST_RETRACT'))"]
    with envs.SGLANG_TEST_RETRACT.override(True):
        process = subprocess.Popen(
            command, stdout=subprocess.PIPE, stderr=subprocess.PIPE
        )
        process.wait()
        output = process.stdout.read().decode("utf-8").strip()
        assert output == "True"

    process = subprocess.Popen(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    output = process.stdout.read().decode("utf-8").strip()
    assert output == "None"
```
**EN:** This callable implements `example_with_subprocess` and mainly implements example with subprocess. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `example_with_subprocess`，主要用于实现 example with subprocess 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；读取环境变量驱动的配置。

### Lines 768-789: Function example_with_implicit_bool_avoidance
```python
def example_with_implicit_bool_avoidance():
    @contextmanager
    def assert_throws(message_matcher: str):
        try:
            yield
        except Exception as e:
            assert message_matcher in str(e), f"{e=}"
            print(f"assert_throws find expected error: {e}")
            return
        raise AssertionError(f"assert_throws do not see exceptions")

    with assert_throws("Please use `envs.YOUR_FLAG.get()` instead of `envs.YOUR_FLAG`"):
        if envs.SGLANG_TEST_RETRACT:
            pass

    with assert_throws("Please use `envs.YOUR_FLAG.get()` instead of `envs.YOUR_FLAG`"):
        if (1 != 1) or envs.SGLANG_TEST_RETRACT:
            pass

    with assert_throws("Please use `envs.YOUR_FLAG.get()` instead of `envs.YOUR_FLAG`"):
        if envs.SGLANG_TEST_RETRACT or (1 == 1):
            pass
```
**EN:** This callable implements `example_with_implicit_bool_avoidance` and mainly implements example with implicit bool avoidance. In this range it performs defensive checks on invalid state; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `example_with_implicit_bool_avoidance`，主要用于实现 example with implicit bool avoidance 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；读取环境变量驱动的配置。

### Lines 792-821: Function examples
```python
def examples():
    # Example usage for envs
    envs.SGLANG_TEST_RETRACT.clear()
    assert envs.SGLANG_TEST_RETRACT.get() is False

    envs.SGLANG_TEST_RETRACT.set(None)
    assert envs.SGLANG_TEST_RETRACT.is_set() and envs.SGLANG_TEST_RETRACT.get() is None

    envs.SGLANG_TEST_RETRACT.clear()
    assert not envs.SGLANG_TEST_RETRACT.is_set()

    envs.SGLANG_TEST_RETRACT.set(True)
    assert envs.SGLANG_TEST_RETRACT.get() is True

    with envs.SGLANG_TEST_RETRACT.override(None):
        assert (
            envs.SGLANG_TEST_RETRACT.is_set() and envs.SGLANG_TEST_RETRACT.get() is None
        )

    assert envs.SGLANG_TEST_RETRACT.get() is True

    envs.SGLANG_TEST_RETRACT.set(None)
    with envs.SGLANG_TEST_RETRACT.override(True):
        assert envs.SGLANG_TEST_RETRACT.get() is True

    assert envs.SGLANG_TEST_RETRACT.is_set() and envs.SGLANG_TEST_RETRACT.get() is None

    example_with_exit_stack()
    example_with_subprocess()
    example_with_implicit_bool_avoidance()
```
**EN:** This callable implements `examples` and mainly implements examples. In this range it performs defensive checks on invalid state; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `examples`，主要用于实现 examples 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；读取环境变量驱动的配置。

### Lines 822-825: Module entry point
```python


if __name__ == "__main__":
    examples()
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

## Key Concepts / 关键概念
- `temp_set_env`: applies configuration to mutable state / 将配置写入可变状态
- `EnvField`: core class or state container / 核心类或状态容器
- `EnvTuple`: core class or state container / 核心类或状态容器
- `EnvStr`: core class or state container / 核心类或状态容器
- `EnvBool`: core class or state container / 核心类或状态容器
- `EnvInt`: core class or state container / 核心类或状态容器
- `EnvFloat`: core class or state container / 核心类或状态容器
- `ToolStrictLevel`: core class or state container / 核心类或状态容器
- `Envs`: core class or state container / 核心类或状态容器
- `_print_deprecated_env`: emits or formats diagnostic output / 输出或格式化诊断信息

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `subprocess`, `warnings`, `contextlib`, `enum`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.debug_utils.cuda_coredump`
