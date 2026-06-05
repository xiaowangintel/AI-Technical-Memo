# util.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/scoring/util.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for pooling or embedding behavior and scoring or reranking behavior. The file exposes 9 helper/class block(s) used by nearby tests. / [CN] 为池化或嵌入行为与打分或重排行为提供共享测试工具。该文件暴露了 9 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L20)
```python
from io import BytesIO

import pybase64 as base64
import torch
import torch.nn.functional as F
from huggingface_hub import hf_hub_download
from PIL import Image
from safetensors.torch import load_file
from transformers import AutoModel, AutoTokenizer

from tests.conftest import HfRunner
from vllm.entrypoints.chat_utils import (
    ChatCompletionContentPartImageParam,
    ChatCompletionContentPartTextParam,
)
from vllm.entrypoints.pooling.scoring.typing import ScoreMultiModalParam
from vllm.entrypoints.pooling.scoring.utils import compute_maxsim_score
```
**EN:** Imports standard-library modules such as `io.BytesIO`, third-party packages like `PIL.Image`, `huggingface_hub.hf_hub_download`, `pybase64`, project helpers such as `tests.conftest.HfRunner`, `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageParam`, `vllm.entrypoints.chat_utils.ChatCompletionContentPartTextParam`.
**CN:** 导入标准库模块（如 `io.BytesIO`）、第三方包（如 `PIL.Image`、`huggingface_hub.hf_hub_download`、`pybase64`）、项目内辅助模块（如 `tests.conftest.HfRunner`、`vllm.entrypoints.chat_utils.ChatCompletionContentPartImageParam`、`vllm.entrypoints.chat_utils.ChatCompletionContentPartTextParam`）。

### Class / 类: ColBERTScoringHfRunner (L23-L64)
```python
class ColBERTScoringHfRunner(torch.nn.Module):
    def __init__(self, model_name, linear_weights_key):
        super().__init__()

        self.tokenizer = AutoTokenizer.from_pretrained(model_name)

        self.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

        extra = {}
        if self.device.type == "cpu":
            extra["attn_implementation"] = "eager"

        self.model = AutoModel.from_pretrained(
            model_name,
# ... 20 lines omitted for brevity ...
    @torch.inference_mode()
    def predict(self, prompts: list[list[str]], *args, **kwargs):
        hf_embeddings = [self(prompt) for prompt in prompts]
        hf_outputs = [
            compute_maxsim_score(*map(torch.tensor, pair)).item()
            for pair in hf_embeddings
        ]
        return torch.as_tensor(hf_outputs)
```
**EN:** This class groups related scenarios in `ColBERTScoringHfRunner`. It contains 0 test method(s) and 3 supporting method(s). Representative methods include `__init__`, `forward`.
**CN:** 该类将与 `ColBERTScoringHfRunner` 相关的场景组织在一起。 它包含 0 个测试方法和 3 个辅助方法。 代表性方法包括 `__init__`、`forward`。

### Helper method / 辅助方法: ColBERTScoringHfRunner.__init__ (L24-L44)
```python
    def __init__(self, model_name, linear_weights_key):
        super().__init__()

        self.tokenizer = AutoTokenizer.from_pretrained(model_name)

        self.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

        extra = {}
        if self.device.type == "cpu":
            extra["attn_implementation"] = "eager"

        self.model = AutoModel.from_pretrained(
            model_name,
            **extra,
        ).to(self.device)
        self.model.eval()

        path = hf_hub_download(model_name, filename="model.safetensors")
        weights = load_file(path)

        self.linear_weight = weights[linear_weights_key].to(self.device).float()
```
**EN:** This helper encapsulates reusable logic in `ColBERTScoringHfRunner.__init__`. Key inputs are `model_name`, `linear_weights_key`.
**CN:** 这个辅助函数将可复用逻辑封装在 `ColBERTScoringHfRunner.__init__` 中。 关键输入包括 `model_name`、`linear_weights_key`。

### Helper method / 辅助方法: ColBERTScoringHfRunner.forward (L46-L55)
```python
    @torch.inference_mode()
    def forward(self, texts):
        embeddings = []
        for text in texts:
            inputs = self.tokenizer(text, return_tensors="pt").to(self.device)
            hidden = self.model(**inputs).last_hidden_state.float()
            projected = F.linear(hidden, self.linear_weight.float())
            normalised = F.normalize(projected, p=2, dim=-1)
            embeddings.append(normalised.squeeze(0).cpu())
        return embeddings
```
**EN:** This helper encapsulates reusable logic in `ColBERTScoringHfRunner.forward`. Key inputs are `texts`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `ColBERTScoringHfRunner.forward` 中。 关键输入包括 `texts`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: ColBERTScoringHfRunner.predict (L57-L64)
```python
    @torch.inference_mode()
    def predict(self, prompts: list[list[str]], *args, **kwargs):
        hf_embeddings = [self(prompt) for prompt in prompts]
        hf_outputs = [
            compute_maxsim_score(*map(torch.tensor, pair)).item()
            for pair in hf_embeddings
        ]
        return torch.as_tensor(hf_outputs)
```
**EN:** This helper encapsulates reusable logic in `ColBERTScoringHfRunner.predict`. Key inputs are `prompts`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `ColBERTScoringHfRunner.predict` 中。 关键输入包括 `prompts`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: EncoderScoringHfRunner (L67-L78)
```python
class EncoderScoringHfRunner(HfRunner):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs, is_sentence_transformer=True)

    @torch.inference_mode()
    def predict(self, prompts: list[list[str]], *args, **kwargs):
        hf_embeddings = [self.encode(prompt) for prompt in prompts]
        hf_outputs = [
            F.cosine_similarity(*map(torch.tensor, pair), dim=0)
            for pair in hf_embeddings
        ]
        return torch.as_tensor(hf_outputs)
```
**EN:** This class groups related scenarios in `EncoderScoringHfRunner`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `predict`.
**CN:** 该类将与 `EncoderScoringHfRunner` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`predict`。

### Helper method / 辅助方法: EncoderScoringHfRunner.__init__ (L68-L69)
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs, is_sentence_transformer=True)
```
**EN:** This helper encapsulates reusable logic in `EncoderScoringHfRunner.__init__`.
**CN:** 这个辅助函数将可复用逻辑封装在 `EncoderScoringHfRunner.__init__` 中。

### Helper method / 辅助方法: EncoderScoringHfRunner.predict (L71-L78)
```python
    @torch.inference_mode()
    def predict(self, prompts: list[list[str]], *args, **kwargs):
        hf_embeddings = [self.encode(prompt) for prompt in prompts]
        hf_outputs = [
            F.cosine_similarity(*map(torch.tensor, pair), dim=0)
            for pair in hf_embeddings
        ]
        return torch.as_tensor(hf_outputs)
```
**EN:** This helper encapsulates reusable logic in `EncoderScoringHfRunner.predict`. Key inputs are `prompts`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `EncoderScoringHfRunner.predict` 中。 关键输入包括 `prompts`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: make_base64_image (L81-L89)
```python
def make_base64_image(
    width: int = 64, height: int = 64, color: tuple[int, int, int] = (255, 0, 0)
) -> str:
    """Create a small solid-color PNG image and return its base64 data URI."""
    img = Image.new("RGB", (width, height), color)
    buf = BytesIO()
    img.save(buf, format="PNG")
    b64 = base64.b64encode(buf.getvalue()).decode()
    return f"data:image/png;base64,{b64}"
```
**EN:** This helper encapsulates reusable logic in `make_base64_image`. Key inputs are `width`, `height`, `color`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `make_base64_image` 中。 关键输入包括 `width`、`height`、`color`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: make_image_mm_param (L92-L107)
```python
def make_image_mm_param(
    image_uri: str,
    text: str | None = None,
) -> ScoreMultiModalParam:
    """Build a ScoreMultiModalParam containing an image (and optional text)."""
    content: list = [
        ChatCompletionContentPartImageParam(
            type="image_url",
            image_url={"url": image_uri},
        ),
    ]
    if text is not None:
        content.append(
            ChatCompletionContentPartTextParam(type="text", text=text),
        )
    return ScoreMultiModalParam(content=content)
```
**EN:** This helper encapsulates reusable logic in `make_image_mm_param`. Key inputs are `image_uri`, `text`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `make_image_mm_param` 中。 关键输入包括 `image_uri`、`text`。 它把计算得到的状态或辅助对象返回给调用方。

## Key Concepts / 关键概念
- **EN:** The file documents setup, execution, and validation logic for its test area.
  **CN:** 该文件记录了对应测试领域的初始化、执行与校验逻辑。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `io.BytesIO`
- **Third-party / 第三方**: `PIL.Image`, `huggingface_hub.hf_hub_download`, `pybase64`, `safetensors.torch.load_file`, `torch`, `torch.nn.functional`, `transformers.AutoModel`, `transformers.AutoTokenizer`
- **Project / 项目内**: `tests.conftest.HfRunner`, `vllm.entrypoints.chat_utils.ChatCompletionContentPartImageParam`, `vllm.entrypoints.chat_utils.ChatCompletionContentPartTextParam`, `vllm.entrypoints.pooling.scoring.typing.ScoreMultiModalParam`, `vllm.entrypoints.pooling.scoring.utils.compute_maxsim_score`
