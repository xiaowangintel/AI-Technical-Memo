# test_nemotron_parse.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/test_nemotron_parse.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 1 test(s), 0 fixture(s), and 5 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 1 个测试、0 个 fixture，以及 5 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L14)
```python
from collections.abc import Iterable, Sequence

import pytest
import regex as re
from transformers import AutoModel

from tests.models.utils import check_logprobs_close
from vllm.assets.image import ImageAsset
from vllm.logprobs import Logprob, SampleLogprobs
from vllm.tokenizers import TokenizerLike

from ....conftest import HfRunner, PromptImageInput, VllmRunner
```
**EN:** Imports standard-library modules such as `collections.abc.Iterable`, `collections.abc.Sequence`, third-party packages like `pytest`, `regex`, `transformers.AutoModel`, project helpers such as `tests.models.utils.check_logprobs_close`, `vllm.assets.image.ImageAsset`, `vllm.logprobs.Logprob`.
**CN:** 导入标准库模块（如 `collections.abc.Iterable`、`collections.abc.Sequence`）、第三方包（如 `pytest`、`regex`、`transformers.AutoModel`）、项目内辅助模块（如 `tests.models.utils.check_logprobs_close`、`vllm.assets.image.ImageAsset`、`vllm.logprobs.Logprob`）。

### Module setup / 模块级配置: IMAGE, PROMPT (L16-L19)
```python
IMAGE = ImageAsset("paper-11").pil_image_ext(ext="png").convert("RGB")
PROMPT = (
    "</s><s><predict_bbox><predict_classes><output_markdown><predict_no_text_in_pic>"
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `IMAGE`, `PROMPT`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `IMAGE`、`PROMPT`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: DummyLogprobs (L22-L27)
```python
class DummyLogprobs(dict[int, Logprob]):
    def __init__(self, vocab_ids: Iterable[int]):
        super().__init__(dict.fromkeys(vocab_ids, Logprob(0.0)))

    def __repr__(self):
        return "DummyLogprobs()"
```
**EN:** This class groups related scenarios in `DummyLogprobs`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `__repr__`.
**CN:** 该类将与 `DummyLogprobs` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`__repr__`。

### Helper method / 辅助方法: DummyLogprobs.__init__ (L23-L24)
```python
    def __init__(self, vocab_ids: Iterable[int]):
        super().__init__(dict.fromkeys(vocab_ids, Logprob(0.0)))
```
**EN:** This helper encapsulates reusable logic in `DummyLogprobs.__init__`. Key inputs are `vocab_ids`.
**CN:** 这个辅助函数将可复用逻辑封装在 `DummyLogprobs.__init__` 中。 关键输入包括 `vocab_ids`。

### Helper method / 辅助方法: DummyLogprobs.__repr__ (L26-L27)
```python
    def __repr__(self):
        return "DummyLogprobs()"
```
**EN:** This helper encapsulates reusable logic in `DummyLogprobs.__repr__`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `DummyLogprobs.__repr__` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: mask_bbox_tokens (L30-L50)
```python
def mask_bbox_tokens(
    output: tuple[list[int], str, SampleLogprobs],
    tokenizer: TokenizerLike,
) -> tuple[list[int], str, SampleLogprobs]:
    """
    Always pass check_logprobs_close check for bounding box tokens
    because it is reasonable for them to differ slightly.
    """
    ignore_pattern = r"<[xy]_[\d.]+>"
    vocab = tokenizer.get_vocab()

    output_ids, output_str, out_logprobs = output

    masked_logprobs = list[dict[int, Logprob]]()
    for token, logprobs in zip(output_ids, out_logprobs):
        if re.match(ignore_pattern, tokenizer.decode(token)):
            masked_logprobs.append(DummyLogprobs(vocab.values()))
        else:
            masked_logprobs.append(logprobs)

    return output_ids, output_str, masked_logprobs
```
**EN:** This helper encapsulates reusable logic in `mask_bbox_tokens`. Key inputs are `output`, `tokenizer`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `mask_bbox_tokens` 中。 关键输入包括 `output`、`tokenizer`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: run_test (L53-L105)
```python
def run_test(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    inputs: Sequence[tuple[list[str], PromptImageInput]],
    model: str,
    *,
    dtype: str,
    max_tokens: int,
    num_logprobs: int,
) -> None:
    """Verify that the inference result is the same between hf and vllm."""
    with vllm_runner(
        model,
        dtype=dtype,
        max_num_seqs=64,
        limit_mm_per_prompt={"image": 1},
        trust_remote_code=True,
    ) as vllm_model:
# ... 27 lines omitted for brevity ...
                mask_bbox_tokens(output, tokenizer) for output in hf_outputs
            ],
            outputs_1_lst=[
                mask_bbox_tokens(output, tokenizer) for output in vllm_outputs
            ],
            name_0="hf",
            name_1="vllm",
        )
```
**EN:** This helper encapsulates reusable logic in `run_test`. Key inputs are `hf_runner`, `vllm_runner`, `inputs`, `model`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_test` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`inputs`、`model`。

### Test / 测试: test_models (L108-L124)
```python
@pytest.mark.parametrize("model", ["nvidia/NVIDIA-Nemotron-Parse-v1.2"])
@pytest.mark.parametrize("dtype", ["bfloat16"])
@pytest.mark.parametrize("num_logprobs", [5])
def test_models(
    hf_runner, vllm_runner, model: str, dtype: str, num_logprobs: int
) -> None:
    run_test(
        hf_runner,
        vllm_runner,
        inputs=[
            ([PROMPT] * 10, [IMAGE] * 10),
        ],
        model=model,
        dtype=dtype,
        max_tokens=100,
        num_logprobs=num_logprobs,
    )
```
**EN:** This test validates `test_models`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `dtype`, `num_logprobs`.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`dtype`、`num_logprobs`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Iterable`, `collections.abc.Sequence`
- **Third-party / 第三方**: `pytest`, `regex`, `transformers.AutoModel`
- **Project / 项目内**: `tests.models.utils.check_logprobs_close`, `vllm.assets.image.ImageAsset`, `vllm.logprobs.Logprob`, `vllm.logprobs.SampleLogprobs`, `vllm.tokenizers.TokenizerLike`
- **Local relative imports / 本地相对导入**: `....conftest.HfRunner`, `....conftest.PromptImageInput`, `....conftest.VllmRunner`
