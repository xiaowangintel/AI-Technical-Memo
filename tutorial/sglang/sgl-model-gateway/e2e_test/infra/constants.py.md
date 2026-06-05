# constants.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/infra/constants.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises constants behavior in the end-to-end infrastructure layer. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 端到端测试基础设施 中与 constants 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Module docstring
```python
"""Constants and enums for E2E test infrastructure."""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 3-3: Imports and dependencies
```python
from enum import Enum
```
**EN:** This block imports `enum`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 6-10: Class `ConnectionMode`
```python
class ConnectionMode(str, Enum):
    """Worker connection protocol."""

    HTTP = "http"
    GRPC = "grpc"
```
**EN:** Class `ConnectionMode` groups related state and behavior. It exposes 0 method(s) that implement the module's primary abstraction.
**CN:** 类 `ConnectionMode` 将相关状态与行为封装在一起，提供 0 个方法来实现本模块的核心抽象。

### Lines 13-18: Class `WorkerType`
```python
class WorkerType(str, Enum):
    """Worker specialization type."""

    REGULAR = "regular"
    PREFILL = "prefill"
    DECODE = "decode"
```
**EN:** Class `WorkerType` groups related state and behavior. It exposes 0 method(s) that implement the module's primary abstraction.
**CN:** 类 `WorkerType` 将相关状态与行为封装在一起，提供 0 个方法来实现本模块的核心抽象。

### Lines 21-28: Class `Runtime`
```python
class Runtime(str, Enum):
    """Inference runtime/backend."""

    SGLANG = "sglang"
    VLLM = "vllm"
    OPENAI = "openai"
    XAI = "xai"
    GEMINI = "gemini"
```
**EN:** Class `Runtime` groups related state and behavior. It exposes 0 method(s) that implement the module's primary abstraction.
**CN:** 类 `Runtime` 将相关状态与行为封装在一起，提供 0 个方法来实现本模块的核心抽象。

### Lines 29-74: Module constants and configuration
```python


# Convenience sets
LOCAL_MODES = frozenset({ConnectionMode.HTTP, ConnectionMode.GRPC})
LOCAL_RUNTIMES = frozenset({Runtime.SGLANG, Runtime.VLLM})
CLOUD_RUNTIMES = frozenset({Runtime.OPENAI, Runtime.XAI, Runtime.GEMINI})

# Fixture parameter names (used in @pytest.mark.parametrize)
PARAM_SETUP_BACKEND = "setup_backend"
PARAM_BACKEND_ROUTER = "backend_router"
PARAM_MODEL = "model"

# Default model
DEFAULT_MODEL = "llama-8b"

# Environment variable names
ENV_MODELS = "E2E_MODELS"
ENV_BACKENDS = "E2E_BACKENDS"
ENV_MODEL = "E2E_MODEL"
ENV_STARTUP_TIMEOUT = "E2E_STARTUP_TIMEOUT"
ENV_SKIP_MODEL_POOL = "SKIP_MODEL_POOL"
ENV_SKIP_BACKEND_SETUP = "SKIP_BACKEND_SETUP"
ENV_SHOW_ROUTER_LOGS = "SHOW_ROUTER_LOGS"
ENV_SHOW_WORKER_LOGS = "SHOW_WORKER_LOGS"

# Network
DEFAULT_HOST = "127.0.0.1"

# Timeouts (seconds)
DEFAULT_STARTUP_TIMEOUT = 300
DEFAULT_ROUTER_TIMEOUT = 60
HEALTH_CHECK_INTERVAL = 2  # Check every 2s (was 5s)

# Model loading configuration
INITIAL_GRACE_PERIOD = 30  # Wait before first health check (model loading time)
LAUNCH_STAGGER_DELAY = (
    10  # Delay between launching multiple workers (avoid I/O contention)
)

# Retry configuration
MAX_RETRY_ATTEMPTS = (
    6  # Max retries with exponential backoff (total ~63s: 1+2+4+8+16+32)
)

# Display formatting
LOG_SEPARATOR_WIDTH = 60  # Width for log separator lines (e.g., "="*60)
```
**EN:** This section defines module-level names such as `LOCAL_MODES`, `LOCAL_RUNTIMES`, `CLOUD_RUNTIMES`, `PARAM_SETUP_BACKEND`, and 19 more, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Test infrastructure orchestration / 测试基础设施编排
- Primary classes: `ConnectionMode`, `WorkerType`, `Runtime` / 主要类：`ConnectionMode`, `WorkerType`, `Runtime`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `enum`
- **Third-party / 第三方**: None explicitly imported / 未显式导入
- **Internal / 内部模块**: None explicitly imported / 未显式导入
