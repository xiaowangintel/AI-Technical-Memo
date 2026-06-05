# medusa.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/spec_decode/medusa.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `MedusaProposer` for the V1 `spec_decode` subsystem. / 为 V1 的 `spec_decode` 子系统实现 `MedusaProposer`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import torch
import torch.nn as nn

from vllm.config import VllmConfig
from vllm.forward_context import set_forward_context
from vllm.logger import init_logger
from vllm.model_executor.model_loader import get_model
from vllm.model_executor.models.interfaces import is_mixture_of_experts
from vllm.v1.sample.metadata import SamplingMetadata

# Initialize logger
logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `MedusaProposer` class / `MedusaProposer` 类
```python
class MedusaProposer:
    """
    Medusa proposer class for generating token sequences
    """
```
**EN:** Introduces the `MedusaProposer` class. Core methods include `__init__`, `propose`, `load_model`, `dummy_run`. Docstring signal: Medusa proposer class for generating token sequences
**CN:** 这里定义 `MedusaProposer` 类。核心方法包括 `__init__`, `propose`, `load_model`, `dummy_run`。

### `MedusaProposer.__init__` method / `MedusaProposer.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        # Save config parameters
        self.vllm_config = vllm_config
        assert vllm_config.speculative_config is not None, (
            "Speculative config must be set"
        )
        self.spec_config = vllm_config.speculative_config
        self.device = device
        self.max_num_tokens = vllm_config.scheduler_config.max_num_batched_tokens
        self.hidden_size = self.spec_config.draft_model_config.get_hidden_size()
        self.dtype = vllm_config.model_config.dtype
```
**EN:** This method initializes the object state within `MedusaProposer`. Key calls include `get_hidden_size`. It touches state such as `vllm_config`, `spec_config`, `device`, `max_num_tokens`, `hidden_size`, `dtype`.
**CN:** 该方法会初始化对象状态，其作用域位于`MedusaProposer`。 关键调用包括 `get_hidden_size`。 它会读写 `vllm_config`, `spec_config`, `device`, `max_num_tokens`, `hidden_size`, `dtype` 等状态。

### `MedusaProposer.propose` method / `MedusaProposer.propose` 方法
```python
    def propose(
        self,
        target_hidden_states: torch.Tensor,
        sampling_metadata: SamplingMetadata,
        slot_mappings: dict[str, torch.Tensor]
        | list[dict[str, torch.Tensor]]
        | None = None,  # unused
    ) -> torch.Tensor:
        # Generate blocks and compute logits
        blocks = self.model(target_hidden_states)
        logits = self.model.compute_logits(blocks)

        # Compute argmax for each Medusa head and stack into a single tensor
        # Shape: [batch_size, num_heads]
        draft_tokens = torch.stack([logit.argmax(dim=-1) for logit in logits], dim=1)

        return draft_tokens
```
**EN:** This method implements `propose` within `MedusaProposer`. Key calls include `model`, `compute_logits`, `stack`, `argmax`.
**CN:** 该方法会实现 `propose`，其作用域位于`MedusaProposer`。 关键调用包括 `model`, `compute_logits`, `stack`, `argmax`。

### `MedusaProposer.load_model` method / `MedusaProposer.load_model` 方法
```python
    def load_model(self, target_model: nn.Module) -> None:
        from vllm.compilation.backends import set_model_tag

        with set_model_tag("medusa_head"):
            self.model = get_model(
                vllm_config=self.vllm_config,
                model_config=self.spec_config.draft_model_config,
            )
        assert not (
            is_mixture_of_experts(self.model)
            and self.vllm_config.parallel_config.enable_eplb
        ), "EPLB for Medusa is not supported"
```
**EN:** This method loads external or cached state within `MedusaProposer`. Key calls include `set_model_tag`, `get_model`, `is_mixture_of_experts`. It touches state such as `model`.
**CN:** 该方法会加载外部或缓存状态，其作用域位于`MedusaProposer`。 关键调用包括 `set_model_tag`, `get_model`, `is_mixture_of_experts`。 它会读写 `model` 等状态。

### `MedusaProposer.dummy_run` method / `MedusaProposer.dummy_run` 方法
```python
    @torch.inference_mode()
    def dummy_run(self, num_tokens: int) -> None:
        hidden_states = torch.zeros(
            (self.max_num_tokens, self.hidden_size),
            dtype=self.dtype,
            device=self.device,
        )
        with set_forward_context(None, self.vllm_config, num_tokens=num_tokens):
            self.model(hidden_states)
```
**EN:** This method implements `dummy_run` within `MedusaProposer`. Key calls include `inference_mode`, `zeros`, `set_forward_context`, `model`.
**CN:** 该方法会实现 `dummy_run`，其作用域位于`MedusaProposer`。 关键调用包括 `inference_mode`, `zeros`, `set_forward_context`, `model`。

## Key Concepts / 关键概念
- `MedusaProposer`: central class or interface in this module. / `MedusaProposer`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.model_loader`, `vllm.model_executor.models.interfaces`, `vllm.v1.sample.metadata`, `vllm.compilation.backends`
