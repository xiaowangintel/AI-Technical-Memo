# conftest.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/conftest.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared pytest fixtures and hooks for multimodal processing and model-facing behavior. / [CN] 为多模态处理与面向模型的行为提供共享的 pytest fixture 与钩子。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L10)
```python
import os
import warnings

import torch

from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `os`, `warnings`, third-party packages like `torch`, project helpers such as `vllm.platforms.current_platform`.
**CN:** 导入标准库模块（如 `os`、`warnings`）、第三方包（如 `torch`）、项目内辅助模块（如 `vllm.platforms.current_platform`）。

### Helper / 辅助函数: pytest_configure (L13-L27)
```python
def pytest_configure(config):
    """Early ROCm configuration that must happen before test collection."""
    if not current_platform.is_rocm():
        return

    # Disable skinny GEMM on ROCm to avoid non-deterministic results
    # from atomic reductions in wvSplitKrc kernel.
    # See: https://github.com/vllm-project/vllm/pull/33493#issuecomment-3906083975
    os.environ["VLLM_ROCM_USE_SKINNY_GEMM"] = "0"
    warnings.warn(
        "ROCm: Set VLLM_ROCM_USE_SKINNY_GEMM=0 to avoid non-deterministic "
        "results from skinny GEMM atomic reductions",
        UserWarning,
        stacklevel=1,
    )
```
**EN:** This helper encapsulates reusable logic in `pytest_configure`. Key inputs are `config`.
**CN:** 这个辅助函数将可复用逻辑封装在 `pytest_configure` 中。 关键输入包括 `config`。

### Helper / 辅助函数: pytest_collection_modifyitems (L30-L50)
```python
def pytest_collection_modifyitems(config, items):
    """Configure ROCm-specific settings based on collected tests."""
    if not current_platform.is_rocm():
        return

    skip_patterns = ["test_granite_speech.py"]
    if any(pattern in str(arg) for arg in config.args for pattern in skip_patterns):
        return

    # Disable Flash/MemEfficient SDP on ROCm to avoid HF Transformers
    # accuracy issues: https://github.com/vllm-project/vllm/issues/30167
    # TODO: Remove once ROCm SDP accuracy issues are resolved on HuggingFace
    torch.backends.cuda.enable_flash_sdp(False)
    torch.backends.cuda.enable_mem_efficient_sdp(False)
    torch.backends.cuda.enable_math_sdp(True)
    warnings.warn(
        "ROCm: Disabled flash_sdp and mem_efficient_sdp, enabled math_sdp "
        "to avoid HuggingFace Transformers accuracy issues",
        UserWarning,
        stacklevel=1,
    )
```
**EN:** This helper encapsulates reusable logic in `pytest_collection_modifyitems`. Key inputs are `config`, `items`.
**CN:** 这个辅助函数将可复用逻辑封装在 `pytest_collection_modifyitems` 中。 关键输入包括 `config`、`items`。

### Helper / 辅助函数: patch_hf_vision_attn_for_rocm (L53-L69)
```python
def patch_hf_vision_attn_for_rocm(model):
    """Force SDPA for HF vision encoders on ROCm.

    HF's flash_attention_2 has accuracy issues on ROCm that bypass
    torch.backends.cuda settings. This forces SDPA which then uses
    math_sdp via the pytest_collection_modifyitems settings.
    """
    if not current_platform.is_rocm():
        return

    inner = getattr(model, "model", model)

    if hasattr(inner, "vision_embedding"):
        vit = inner.vision_embedding[0]
        for layer in vit.encoder.layers:
            if hasattr(layer, "self_attn"):
                layer.self_attn.vision_config._attn_implementation = "sdpa"
```
**EN:** This helper encapsulates reusable logic in `patch_hf_vision_attn_for_rocm`. Key inputs are `model`.
**CN:** 这个辅助函数将可复用逻辑封装在 `patch_hf_vision_attn_for_rocm` 中。 关键输入包括 `model`。

## Key Concepts / 关键概念
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `os`, `warnings`
- **Third-party / 第三方**: `torch`
- **Project / 项目内**: `vllm.platforms.current_platform`
