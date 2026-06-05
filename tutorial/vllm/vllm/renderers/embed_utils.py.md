# embed_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/renderers/embed_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements rendering adapters used to turn structured inputs into prompts or outputs. / 实现把结构化输入转换为提示词或输出内容的渲染适配层。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-13)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from io import BytesIO
from typing import TYPE_CHECKING

import pybase64
import torch

from vllm.exceptions import VLLMValidationError
from vllm.utils.async_utils import make_async

if TYPE_CHECKING:
    from vllm.config import ModelConfig
```
**EN:** Sets up the module with standard-library support such as `io`, `typing`, external packages such as `pybase64`, `torch`, vLLM modules such as `vllm.exceptions`, `vllm.utils.async_utils`, `vllm.config`. It prepares the symbols later used by `safe_load_prompt_embeds`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.exceptions`, `vllm.utils.async_utils`, `vllm.config` 等 vLLM 内部依赖。 这些准备工作为后续的 `safe_load_prompt_embeds` 提供上下文。

### safe_load_prompt_embeds (lines 16-77)
```python
def safe_load_prompt_embeds(
    model_config: "ModelConfig",
    embed: bytes,
) -> torch.Tensor:
    if not model_config.enable_prompt_embeds:
        raise VLLMValidationError(
            "You must set `--enable-prompt-embeds` to input `prompt_embeds`.",
            parameter="prompt_embeds",
        )

    # Enable sparse tensor integrity checks to prevent out-of-bounds
    # writes from maliciously crafted tensors
    with torch.sparse.check_sparse_tensor_invariants():
        tensor = torch.load(
            BytesIO(pybase64.b64decode(embed, validate=True)),
            weights_only=True,
            map_location=torch.device("cpu"),
        )
        if not isinstance(tensor, torch.Tensor):
            raise VLLMValidationError(
                "`prompt_embeds` payload did not deserialize to a torch.Tensor.",
                parameter="prompt_embeds",
            )
        tensor = tensor.to_dense()
    # ...
                f"type, cannot safely cast to the model's dtype {expected_dtype}.",
                parameter="prompt_embeds",
            )
        tensor = tensor.to(expected_dtype)

    return tensor
```
**EN:** `safe_load_prompt_embeds` implements helper logic used by this module. It mainly works with `model_config`, `embed`. Inside the body, it relies on `model_config.get_hidden_size`, `VLLMValidationError`, `torch.sparse.check_sparse_tensor_invariants` to complete the main steps.
**CN:** `safe_load_prompt_embeds` 负责实现本模块使用的辅助逻辑。 它主要处理 `model_config`, `embed` 等参数。 实现过程中会调用 `model_config.get_hidden_size`, `VLLMValidationError`, `torch.sparse.check_sparse_tensor_invariants` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`safe_load_prompt_embeds`**: Key helper or entry point in this file. / **`safe_load_prompt_embeds`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: io, typing
- **Third-party / 第三方**: pybase64, torch
- **Internal vLLM / vLLM 内部依赖**: vllm.exceptions, vllm.utils.async_utils, vllm.config
