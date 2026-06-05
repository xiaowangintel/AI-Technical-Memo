# test_all_pooling_plus_chunked_prefill.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling/test_all_pooling_plus_chunked_prefill.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 1 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 1 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L9)
```python
import pytest
import torch
from transformers import AutoModel

from tests.models.utils import check_embeddings_close
from vllm import TokensPrompt
from vllm.config import PoolerConfig
```
**EN:** Imports third-party packages like `pytest`, `torch`, `transformers.AutoModel`, project helpers such as `tests.models.utils.check_embeddings_close`, `vllm.TokensPrompt`, `vllm.config.PoolerConfig`.
**CN:** 导入第三方包（如 `pytest`、`torch`、`transformers.AutoModel`）、项目内辅助模块（如 `tests.models.utils.check_embeddings_close`、`vllm.TokensPrompt`、`vllm.config.PoolerConfig`）。

### Test / 测试: test_embed_models (L12-L55)
```python
@pytest.mark.parametrize(
    "model",
    ["Qwen/Qwen3-Embedding-0.6B"],
)
@torch.inference_mode
def test_embed_models(hf_runner, vllm_runner, model: str):
    chunk_size = 10
    n_prompt_tokens = [55, 56, 57]
    token_prompts = [[1024 + i for i in range(n)] for n in n_prompt_tokens]

    with vllm_runner(
        model,
        runner="pooling",
        pooler_config=PoolerConfig(task="token_embed"),
        max_model_len=128,
        max_num_batched_tokens=chunk_size,
        enforce_eager=True,
        # `enable_chunked_prefill`: Set to `False` instead of `None` in VllmRunner
# ... 18 lines omitted for brevity ...
    for hf_output, vllm_output in zip(hf_outputs, vllm_outputs):
        check_embeddings_close(
            embeddings_0_lst=hf_output,
            embeddings_1_lst=vllm_output,
            name_0="hf",
            name_1="vllm",
            tol=1e-2,
        )
```
**EN:** This test validates `test_embed_models`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `model`.
**CN:** 这个测试验证 `test_embed_models`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`model`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`, `torch`, `transformers.AutoModel`
- **Project / 项目内**: `tests.models.utils.check_embeddings_close`, `vllm.TokensPrompt`, `vllm.config.PoolerConfig`
