# pe_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/loader/component_loaders/pe_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the component loading layer. It centers on `PEModelWrapper`, `PELoader`, and `_read_model_max_length`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于组件加载层。它围绕 `PEModelWrapper`、`PELoader` 和 `_read_model_max_length` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-13: module setup and imports / 模块初始化与导入
```python
import json
import os

import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.loader.component_loaders.component_loader import (
    ComponentLoader,
)
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `json`, `os`, `torch`, `transformers`, `sglang.multimodal_gen.runtime.distributed`, and `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `json`、`os`、`torch`、`transformers`、`sglang.multimodal_gen.runtime.distributed` 和 `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`。这些依赖为后续实现提供所需符号。

### Lines 15-15: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 18-32: `_read_model_max_length` implementation / `_read_model_max_length` 实现
```python
def _read_model_max_length(model_path: str) -> int | None:
    """Read model_max_length from tokenizer_config.json in the given directory."""
    config_path = os.path.join(model_path, "tokenizer_config.json")
    if os.path.exists(config_path):
        try:
            with open(config_path, encoding="utf-8") as f:
                config = json.load(f)
            val = config.get("model_max_length")
            if val is not None:
                return int(val)
        except Exception as e:
            logger.warning(
                "Failed to read tokenizer_config.json from %s: %s", model_path, e
            )
    return None
```
**EN:** This block defines function `_read_model_max_length`. Read model_max_length from tokenizer_config.json in the given directory. Key calls include `os.path.join`, `os.path.exists`, `config.get`, `open`, and `json.load`. The implementation branches on conditions, handles exceptional paths, uses context-managed resources. Parameters such as `model_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_read_model_max_length`。 它用于处理 read model max length 相关逻辑。 关键调用包括 `os.path.join`、`os.path.exists`、`config.get`、`open` 和 `json.load`。 实现中包含条件分支，处理异常路径，使用上下文管理资源。 本段逻辑主要由 `model_path` 等参数驱动。

### Lines 35-36: `PEModelWrapper` class overview / `PEModelWrapper` 类概览
```python
class PEModelWrapper:
```
**EN:** This block defines class `PEModelWrapper`. It encapsulates pemodel wrapper behavior.
**CN:** 该代码块定义了类 `PEModelWrapper`。 它用于封装 pemodel wrapper 相关行为。

### Lines 37-41: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, model, tokenizer, device, model_max_length: int):
        self.model = model
        self.pe_tokenizer = tokenizer
        self.device = device
        self.model_max_length = model_max_length
```
**EN:** This block defines method `__init__` on `PEModelWrapper`. It initializes the instance state. Parameters such as `model`, `tokenizer`, `device`, and `model_max_length` drive the behavior in this section.
**CN:** 该代码块定义了 `PEModelWrapper` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `model`、`tokenizer`、`device` 和 `model_max_length` 等参数驱动。

### Lines 43-70: `generate` implementation / `generate` 实现
```python
    def generate(self, prompt: str, sampling_params: dict) -> dict:
        inputs = self.pe_tokenizer(
            prompt,
            return_tensors="pt",
            truncation=True,
            max_length=self.model_max_length,
        ).to(self.device)

        input_len = inputs["input_ids"].shape[1]

        generate_kwargs = dict(
            **inputs,
            max_new_tokens=sampling_params.get("max_new_tokens", self.model_max_length),
            do_sample=True,
        )
        temperature = sampling_params.get("temperature")
        top_p = sampling_params.get("top_p")
        if temperature is not None:
            generate_kwargs["temperature"] = temperature
        if top_p is not None:
            generate_kwargs["top_p"] = top_p

        with torch.no_grad():
            output_ids = self.model.generate(**generate_kwargs)

        new_tokens = output_ids[0, input_len:]
        text = self.pe_tokenizer.decode(new_tokens, skip_special_tokens=True)
        return {"text": text}
```
**EN:** This block defines method `generate` on `PEModelWrapper`. It generates function. Key calls include `self.pe_tokenizer.to`, `dict`, `sampling_params.get`, `self.pe_tokenizer.decode`, and `torch.no_grad`. The implementation branches on conditions, uses context-managed resources. Parameters such as `prompt`, and `sampling_params` drive the behavior in this section.
**CN:** 该代码块定义了 `PEModelWrapper` 的方法 `generate`。 它用于生成函数。 关键调用包括 `self.pe_tokenizer.to`、`dict`、`sampling_params.get`、`self.pe_tokenizer.decode` 和 `torch.no_grad`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `prompt` 和 `sampling_params` 等参数驱动。

### Lines 72-79: `to` implementation / `to` 实现
```python
    def to(self, *args, **kwargs):
        """Move underlying model to device."""
        self.model = self.model.to(*args, **kwargs)
        if args:
            device = args[0]
            if isinstance(device, (str, torch.device)):
                self.device = torch.device(device)
        return self
```
**EN:** This block defines method `to` on `PEModelWrapper`. Move underlying model to device. Key calls include `self.model.to`, `isinstance`, and `torch.device`. The implementation branches on conditions.
**CN:** 该代码块定义了 `PEModelWrapper` 的方法 `to`。 它用于转换为函数。 关键调用包括 `self.model.to`、`isinstance` 和 `torch.device`。 实现中包含条件分支。

### Lines 82-84: `PELoader` class overview / `PELoader` 类概览
```python
class PELoader(ComponentLoader):
    """Loader for prompt-enhancement causal LM (Ministral-3 based)."""
```
**EN:** This block defines class `PELoader`. Loader for prompt-enhancement causal LM (Ministral-3 based). It inherits from `ComponentLoader`.
**CN:** 该代码块定义了类 `PELoader`。 它用于封装 peloader 相关行为。 它继承自 `ComponentLoader`。

### Lines 85-86: supporting statements / 辅助语句
```python
    component_names = ["pe"]
    expected_library = "transformers"
```
**EN:** This block gathers supporting statements inside `PELoader`. It updates names such as `component_names`, and `expected_library`.
**CN:** 该代码块汇集了位于 `PELoader` 内部的辅助语句。 它会更新 `component_names` 和 `expected_library` 等名称。

### Lines 88-162: `load_customized` implementation / `load_customized` 实现
```python
    def load_customized(
        self, component_model_path: str, server_args: ServerArgs, component_name: str
    ):
        logger.info("Loading PE model from %s ...", component_model_path)

        pe_tokenizer_dir = os.path.join(
            os.path.dirname(component_model_path), "pe_tokenizer"
        )
        if not os.path.exists(
            os.path.join(component_model_path, "tokenizer_config.json")
        ) and os.path.exists(os.path.join(pe_tokenizer_dir, "tokenizer_config.json")):
            tokenizer_path = pe_tokenizer_dir
            logger.info(
                "PE tokenizer files not found in %s, using %s",
                component_model_path,
                tokenizer_path,
            )
        else:
            tokenizer_path = component_model_path

        model_max_length = _read_model_max_length(tokenizer_path)
        if model_max_length is None:
            raise RuntimeError(
                f"Cannot load PE model: 'model_max_length' not found in "
                f"{os.path.join(tokenizer_path, 'tokenizer_config.json')}. "
                "Please ensure the PE component directory (or its sibling "
                "pe_tokenizer/ directory) contains a valid tokenizer_config.json "
                "with a 'model_max_length' field."
            )
        logger.info(
            "PE model_max_length=%d (from tokenizer_config.json)", model_max_length
        )

        tokenizer = AutoTokenizer.from_pretrained(
            tokenizer_path,
            trust_remote_code=server_args.trust_remote_code,
        )
        if tokenizer.pad_token_id is None:
            tokenizer.pad_token_id = tokenizer.eos_token_id

        attn_impl = "flash_attention_2"
        try:
            model = AutoModelForCausalLM.from_pretrained(
                component_model_path,
                torch_dtype=torch.bfloat16,
                trust_remote_code=server_args.trust_remote_code,
                attn_implementation=attn_impl,
            )
            logger.info("PE model: using Flash Attention 2")
        except (ValueError, ImportError):
            logger.warning("Flash Attention 2 not available, falling back to SDPA")
            attn_impl = "sdpa"
            model = AutoModelForCausalLM.from_pretrained(
                component_model_path,
                torch_dtype=torch.bfloat16,
                trust_remote_code=server_args.trust_remote_code,
                attn_implementation=attn_impl,
            )

        device = get_local_torch_device()
        model = model.to(device).eval()

        logger.info(
            "PE model loaded on %s: %s (attn=%s)",
            device,
            model.__class__.__name__,
            attn_impl,
        )

        return PEModelWrapper(
            model=model,
            tokenizer=tokenizer,
            device=device,
            model_max_length=model_max_length,
        )
```
**EN:** This block defines method `load_customized` on `PELoader`. It loads customized. Key calls include `logger.info`, `os.path.join`, `_read_model_max_length`, `AutoTokenizer.from_pretrained`, and `get_local_torch_device`. The implementation branches on conditions, handles exceptional paths. Parameters such as `component_model_path`, `server_args`, and `component_name` drive the behavior in this section.
**CN:** 该代码块定义了 `PELoader` 的方法 `load_customized`。 它用于加载customized。 关键调用包括 `logger.info`、`os.path.join`、`_read_model_max_length`、`AutoTokenizer.from_pretrained` 和 `get_local_torch_device`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `component_model_path`、`server_args` 和 `component_name` 等参数驱动。

## Key Concepts / 关键概念
- `_read_model_max_length`: Read model_max_length from tokenizer_config.json in the given directory. / 顶层函数，用于处理 read model max length 相关逻辑。
- `PEModelWrapper`: Primary class that encapsulates pemodel wrapper behavior. / 核心类，用于封装 pemodel wrapper 相关行为。
- `PELoader`: Loader for prompt-enhancement causal LM (Ministral-3 based). / 核心类，用于封装 peloader 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`
- **Third-party / 第三方依赖**: `torch`, `transformers`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 162
