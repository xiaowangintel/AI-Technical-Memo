# core.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/vlm_utils/core.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for multimodal processing and model-facing behavior. The file exposes 3 helper/class block(s) used by nearby tests. / [CN] 为多模态处理与面向模型的行为提供共享测试工具。该文件暴露了 3 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L16)
```python
from collections.abc import Callable
from typing import Any

import torch
from transformers.models.auto.auto_factory import _BaseAutoModelClass

from vllm.config.model import RunnerOption
from vllm.tokenizers import TokenizerLike

from .....conftest import HfRunner, VllmRunner
from ....registry import HF_EXAMPLE_MODELS
from .types import PromptWithMultiModalInput, RunnerOutput
```
**EN:** Imports standard-library modules such as `collections.abc.Callable`, `typing.Any`, third-party packages like `torch`, `transformers.models.auto.auto_factory._BaseAutoModelClass`, project helpers such as `vllm.config.model.RunnerOption`, `vllm.tokenizers.TokenizerLike`, `.....conftest.HfRunner`.
**CN:** 导入标准库模块（如 `collections.abc.Callable`、`typing.Any`）、第三方包（如 `torch`、`transformers.models.auto.auto_factory._BaseAutoModelClass`）、项目内辅助模块（如 `vllm.config.model.RunnerOption`、`vllm.tokenizers.TokenizerLike`、`.....conftest.HfRunner`）。

### Helper / 辅助函数: run_test (L19-L180)
```python
def run_test(
    *,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    inputs: list[PromptWithMultiModalInput],
    model: str,
    dtype: str,
    max_tokens: int,
    num_logprobs: int,
    enforce_eager: bool,
    max_model_len: int,
    max_num_seqs: int,
    hf_output_post_proc: Callable[[RunnerOutput, str], Any] | None,
    vllm_output_post_proc: Callable[[RunnerOutput, str], Any] | None,
    auto_cls: type[_BaseAutoModelClass],
    use_tokenizer_eos: bool,
    comparator: Callable[..., None],
    get_stop_token_ids: Callable[[TokenizerLike], list[int]] | None,
# ... 136 lines omitted for brevity ...
        # This is usually check_logprobs_close, but it's passed through to
        # allow things like check_outputs_equal where needed
        comparator(
            outputs_0_lst=hf_outputs,
            outputs_1_lst=vllm_outputs,
            name_0="hf",
            name_1="vllm",
        )
```
**EN:** This helper encapsulates reusable logic in `run_test`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_test` 中。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Helper / 辅助函数: process_runner_outputs (L183-L199)
```python
def process_runner_outputs(
    model,
    first_runner_outputs,
    second_runner_outputs,
    first_runner_processor=None,
    second_runner_processor=None,
):
    """Applies the runner processor(s) to the runner outputs, if any."""
    if first_runner_processor is not None:
        first_runner_outputs = process_outputs(
            first_runner_processor, model, first_runner_outputs
        )
    if second_runner_processor is not None:
        second_runner_outputs = process_outputs(
            second_runner_processor, model, second_runner_outputs
        )
    return first_runner_outputs, second_runner_outputs
```
**EN:** This helper encapsulates reusable logic in `process_runner_outputs`. Key inputs are `model`, `first_runner_outputs`, `second_runner_outputs`, `first_runner_processor`, `second_runner_processor`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `process_runner_outputs` 中。 关键输入包括 `model`、`first_runner_outputs`、`second_runner_outputs`、`first_runner_processor`、`second_runner_processor`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: process_outputs (L202-L207)
```python
def process_outputs(output_processor, model, outputs_per_image):
    """Applies a model specific post-processor function to a runner's output"""
    return [
        [output_processor(res, model) for res in outputs]
        for outputs in outputs_per_image
    ]
```
**EN:** This helper encapsulates reusable logic in `process_outputs`. Key inputs are `output_processor`, `model`, `outputs_per_image`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `process_outputs` 中。 关键输入包括 `output_processor`、`model`、`outputs_per_image`。 它把计算得到的状态或辅助对象返回给调用方。

## Key Concepts / 关键概念
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Callable`, `typing.Any`
- **Third-party / 第三方**: `torch`, `transformers.models.auto.auto_factory._BaseAutoModelClass`
- **Project / 项目内**: `vllm.config.model.RunnerOption`, `vllm.tokenizers.TokenizerLike`
- **Local relative imports / 本地相对导入**: `.....conftest.HfRunner`, `.....conftest.VllmRunner`, `....registry.HF_EXAMPLE_MODELS`, `.types.PromptWithMultiModalInput`, `.types.RunnerOutput`
