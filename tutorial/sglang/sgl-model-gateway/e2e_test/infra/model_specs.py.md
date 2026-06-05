# model_specs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/infra/model_specs.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises model specs behavior in the end-to-end infrastructure layer. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 端到端测试基础设施 中与 model specs 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Module docstring
```python
"""Model specifications for E2E tests.

Each model spec defines:
- model: HuggingFace model path or local path
- memory_gb: Estimated GPU memory required
- tp: Tensor parallelism size (number of GPUs needed)
- features: List of features this model supports (for test filtering)
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 10-12: Imports and dependencies
```python
from __future__ import annotations

import os
```
**EN:** This block imports `__future__`, `os`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 13-15: Module constants and configuration
```python

# Environment variable for local model paths (CI uses local copies for speed)
ROUTER_LOCAL_MODEL_PATH = os.environ.get("ROUTER_LOCAL_MODEL_PATH", "")
```
**EN:** This section defines module-level names such as `ROUTER_LOCAL_MODEL_PATH`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 18-24: Helper function `_resolve_model_path`
```python
def _resolve_model_path(hf_path: str) -> str:
    """Resolve model path, preferring local path if available."""
    if ROUTER_LOCAL_MODEL_PATH:
        local_path = os.path.join(ROUTER_LOCAL_MODEL_PATH, hf_path)
        if os.path.exists(local_path):
            return local_path
    return hf_path
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 27-94: Module constants and configuration
```python
MODEL_SPECS: dict[str, dict] = {
    # Primary chat model - used for most tests
    "llama-8b": {
        "model": _resolve_model_path("meta-llama/Llama-3.1-8B-Instruct"),
        "memory_gb": 16,
        "tp": 1,
        "features": ["chat", "streaming", "function_calling"],
    },
    # Small model for quick tests
    "llama-1b": {
        "model": _resolve_model_path("meta-llama/Llama-3.2-1B-Instruct"),
        "memory_gb": 4,
        "tp": 1,
        "features": ["chat", "streaming", "tool_choice"],
    },
    # Function calling specialist
    "qwen-7b": {
        "model": _resolve_model_path("Qwen/Qwen2.5-7B-Instruct"),
        "memory_gb": 14,
        "tp": 1,
        "features": ["chat", "streaming", "function_calling", "pythonic_tools"],
    },
    # Function calling specialist (larger, for Response API tests)
    "qwen-14b": {
        "model": _resolve_model_path("Qwen/Qwen2.5-14B-Instruct"),
        "memory_gb": 28,
        "tp": 2,
        "features": ["chat", "streaming", "function_calling", "pythonic_tools"],
        "worker_args": [
            "--context-length=1000"
        ],  # Faster startup, prevents memory issues
    },
    # Reasoning model
    "deepseek-7b": {
        "model": _resolve_model_path("deepseek-ai/DeepSeek-R1-Distill-Qwen-7B"),
        "memory_gb": 14,
        "tp": 1,
        "features": ["chat", "streaming", "reasoning"],
    },
    # Thinking/reasoning model (larger)
    "qwen-30b": {
        "model": _resolve_model_path("Qwen/Qwen3-30B-A3B"),
        "memory_gb": 60,
        "tp": 4,
        "features": ["chat", "streaming", "thinking", "reasoning"],
    },
    # Mistral for function calling
    "mistral-7b": {
        "model": _resolve_model_path("mistralai/Mistral-7B-Instruct-v0.3"),
        "memory_gb": 14,
        "tp": 1,
        "features": ["chat", "streaming", "function_calling"],
    },
    # Embedding model
    "embedding": {
        "model": _resolve_model_path("intfloat/e5-mistral-7b-instruct"),
        "memory_gb": 14,
        "tp": 1,
        "features": ["embedding"],
    },
    # GPT-OSS model (Harmony)
    "gpt-oss": {
        "model": _resolve_model_path("openai/gpt-oss-20b"),
        "memory_gb": 40,
        "tp": 2,
        "features": ["chat", "streaming", "reasoning", "harmony"],
    },
}
```
**EN:** This section defines module-level names such as `MODEL_SPECS`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 97-103: Helper function `get_models_with_feature`
```python
def get_models_with_feature(feature: str) -> list[str]:
    """Get list of model IDs that support a specific feature."""
    return [
        model_id
        for model_id, spec in MODEL_SPECS.items()
        if feature in spec.get("features", [])
    ]
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 106-112: Helper function `get_model_spec`
```python
def get_model_spec(model_id: str) -> dict:
    """Get spec for a specific model, raising KeyError if not found."""
    if model_id not in MODEL_SPECS:
        raise KeyError(
            f"Unknown model: {model_id}. Available: {list(MODEL_SPECS.keys())}"
        )
    return MODEL_SPECS[model_id]
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 113-151: Module constants and configuration
```python


# Convenience groupings for test parametrization
CHAT_MODELS = get_models_with_feature("chat")
EMBEDDING_MODELS = get_models_with_feature("embedding")
REASONING_MODELS = get_models_with_feature("reasoning")
FUNCTION_CALLING_MODELS = get_models_with_feature("function_calling")


# =============================================================================
# Default model path constants (for backward compatibility with existing tests)
# =============================================================================

DEFAULT_MODEL_PATH = MODEL_SPECS["llama-8b"]["model"]
DEFAULT_SMALL_MODEL_PATH = MODEL_SPECS["llama-1b"]["model"]
DEFAULT_REASONING_MODEL_PATH = MODEL_SPECS["deepseek-7b"]["model"]
DEFAULT_ENABLE_THINKING_MODEL_PATH = MODEL_SPECS["qwen-30b"]["model"]
DEFAULT_QWEN_FUNCTION_CALLING_MODEL_PATH = MODEL_SPECS["qwen-7b"]["model"]
DEFAULT_MISTRAL_FUNCTION_CALLING_MODEL_PATH = MODEL_SPECS["mistral-7b"]["model"]
DEFAULT_GPT_OSS_MODEL_PATH = MODEL_SPECS["gpt-oss"]["model"]
DEFAULT_EMBEDDING_MODEL_PATH = MODEL_SPECS["embedding"]["model"]


# =============================================================================
# Third-party model configurations (cloud APIs)
# =============================================================================

THIRD_PARTY_MODELS: dict[str, dict] = {
    "openai": {
        "description": "OpenAI API",
        "model": "gpt-5-nano",
        "api_key_env": "OPENAI_API_KEY",
    },
    "xai": {
        "description": "xAI API",
        "model": "grok-4-fast",
        "api_key_env": "XAI_API_KEY",
    },
}
```
**EN:** This section defines module-level names such as `CHAT_MODELS`, `EMBEDDING_MODELS`, `REASONING_MODELS`, `FUNCTION_CALLING_MODELS`, and 9 more, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Test infrastructure orchestration / 测试基础设施编排
- Reusable functions: `_resolve_model_path`, `get_models_with_feature`, `get_model_spec` / 可复用函数：`_resolve_model_path`, `get_models_with_feature`, `get_model_spec`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `os`
- **Third-party / 第三方**: None explicitly imported / 未显式导入
- **Internal / 内部模块**: None explicitly imported / 未显式导入
