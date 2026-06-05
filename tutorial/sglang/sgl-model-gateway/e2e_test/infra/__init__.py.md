# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/infra/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This package initializer exposes package-level symbols and metadata for the end-to-end infrastructure layer. It keeps imports convenient for downstream modules. / 该包初始化文件为 端到端测试基础设施 暴露包级符号与元数据，便于下游模块进行导入。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Module docstring
```python
"""Infrastructure for parallel GPU test execution."""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 3-66: Imports and dependencies
```python
from .constants import (  # Enums; Convenience sets; Fixture parameters; Defaults; Environment variables
    CLOUD_RUNTIMES,
    DEFAULT_HOST,
    DEFAULT_MODEL,
    DEFAULT_ROUTER_TIMEOUT,
    DEFAULT_STARTUP_TIMEOUT,
    ENV_BACKENDS,
    ENV_MODEL,
    ENV_MODELS,
    ENV_SHOW_ROUTER_LOGS,
    ENV_SHOW_WORKER_LOGS,
    ENV_SKIP_BACKEND_SETUP,
    ENV_SKIP_MODEL_POOL,
    ENV_STARTUP_TIMEOUT,
    HEALTH_CHECK_INTERVAL,
    LOCAL_MODES,
    LOCAL_RUNTIMES,
    LOG_SEPARATOR_WIDTH,
    MAX_RETRY_ATTEMPTS,
    PARAM_BACKEND_ROUTER,
    PARAM_MODEL,
    PARAM_SETUP_BACKEND,
    ConnectionMode,
    Runtime,
    WorkerType,
)
from .gateway import Gateway, WorkerInfo, launch_cloud_gateway
from .gpu_allocator import (
    GPUAllocator,
    GPUInfo,
    GPUSlot,
    get_gpu_memory_usage,
    get_open_port,
    get_physical_device_indices,
    nvml_context,
    wait_for_gpu_memory_to_clear,
)
from .gpu_monitor import GPUMonitor
from .gpu_monitor import should_monitor as should_monitor_gpu
from .model_pool import ModelInstance, ModelPool, WorkerIdentity
from .model_specs import (  # Default model paths; Model groups
    CHAT_MODELS,
    DEFAULT_EMBEDDING_MODEL_PATH,
    DEFAULT_ENABLE_THINKING_MODEL_PATH,
    DEFAULT_GPT_OSS_MODEL_PATH,
    DEFAULT_MISTRAL_FUNCTION_CALLING_MODEL_PATH,
    DEFAULT_MODEL_PATH,
    DEFAULT_QWEN_FUNCTION_CALLING_MODEL_PATH,
    DEFAULT_REASONING_MODEL_PATH,
    DEFAULT_SMALL_MODEL_PATH,
    EMBEDDING_MODELS,
    FUNCTION_CALLING_MODELS,
    MODEL_SPECS,
    REASONING_MODELS,
    THIRD_PARTY_MODELS,
)
from .process_utils import (
    detect_ib_device,
    kill_process_tree,
    terminate_process,
    wait_for_health,
    wait_for_workers_ready,
)
from .run_eval import run_eval
```
**EN:** This block imports `.constants`, `.gateway`, `.gpu_allocator`, `.gpu_monitor`, and 5 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 68-144: Module constants and configuration
```python
__all__ = [
    # Enums and Identity
    "ConnectionMode",
    "WorkerType",
    "Runtime",
    "WorkerIdentity",
    # Convenience sets
    "LOCAL_MODES",
    "LOCAL_RUNTIMES",
    "CLOUD_RUNTIMES",
    # Fixture params
    "PARAM_SETUP_BACKEND",
    "PARAM_BACKEND_ROUTER",
    "PARAM_MODEL",
    # Defaults
    "DEFAULT_MODEL",
    "DEFAULT_HOST",
    "DEFAULT_STARTUP_TIMEOUT",
    "DEFAULT_ROUTER_TIMEOUT",
    "HEALTH_CHECK_INTERVAL",
    "MAX_RETRY_ATTEMPTS",
    "LOG_SEPARATOR_WIDTH",
    # Env vars
    "ENV_MODELS",
    "ENV_BACKENDS",
    "ENV_MODEL",
    "ENV_STARTUP_TIMEOUT",
    "ENV_SKIP_MODEL_POOL",
    "ENV_SKIP_BACKEND_SETUP",
    "ENV_SHOW_ROUTER_LOGS",
    "ENV_SHOW_WORKER_LOGS",
    # GPU allocation
    "GPUAllocator",
    "GPUInfo",
    "GPUSlot",
    # GPU utilities
    "nvml_context",
    "get_open_port",
    "get_physical_device_indices",
    "get_gpu_memory_usage",
    "wait_for_gpu_memory_to_clear",
    # Process utilities
    "kill_process_tree",
    "terminate_process",
    "wait_for_health",
    "wait_for_workers_ready",
    "detect_ib_device",
    # GPU monitoring
    "GPUMonitor",
    "should_monitor_gpu",
    # Model management
    "ModelInstance",
    "ModelPool",
    "MODEL_SPECS",
    # Gateway
    "Gateway",
    "WorkerInfo",
    "launch_cloud_gateway",
    # Default model paths
    "DEFAULT_MODEL_PATH",
    "DEFAULT_SMALL_MODEL_PATH",
    "DEFAULT_REASONING_MODEL_PATH",
    "DEFAULT_ENABLE_THINKING_MODEL_PATH",
    "DEFAULT_QWEN_FUNCTION_CALLING_MODEL_PATH",
    "DEFAULT_MISTRAL_FUNCTION_CALLING_MODEL_PATH",
    "DEFAULT_GPT_OSS_MODEL_PATH",
    "DEFAULT_EMBEDDING_MODEL_PATH",
    # Model groups
    "CHAT_MODELS",
    "EMBEDDING_MODELS",
    "REASONING_MODELS",
    "FUNCTION_CALLING_MODELS",
    # Third-party models
    "THIRD_PARTY_MODELS",
    # Evaluation
    "run_eval",
]
```
**EN:** This section defines module-level names such as `__all__`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Test infrastructure orchestration / 测试基础设施编排

## Dependencies / 依赖关系
- **Standard library / 标准库**: None explicitly imported / 未显式导入
- **Third-party / 第三方**: None explicitly imported / 未显式导入
- **Internal / 内部模块**: `.constants`, `.gateway`, `.gpu_allocator`, `.gpu_monitor`, `.model_pool`, `.model_specs`, `.process_utils`, `.run_eval`
