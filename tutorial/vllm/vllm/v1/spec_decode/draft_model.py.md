# draft_model.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/spec_decode/draft_model.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `DraftModelProposer` for the V1 `spec_decode` subsystem. / 为 V1 的 `spec_decode` 子系统实现 `DraftModelProposer`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import torch
import torch.nn as nn
from typing_extensions import override

from vllm.config import VllmConfig
from vllm.config.utils import replace
from vllm.logger import init_logger
from vllm.model_executor.model_loader import get_model
from vllm.v1.spec_decode.llm_base_proposer import SpecDecodeBaseProposer

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `DraftModelProposer` class / `DraftModelProposer` 类
```python
class DraftModelProposer(SpecDecodeBaseProposer):
```
**EN:** Introduces the `DraftModelProposer` class on top of `SpecDecodeBaseProposer`. Core methods include `__init__`, `_raise_if_vocab_size_mismatch`, `_raise_if_draft_tp_mismatch`, `_create_draft_vllm_config`, `_get_model`, `_maybe_share_embeddings`.
**CN:** 这里定义 `DraftModelProposer` 类，其基类包括 `SpecDecodeBaseProposer`。核心方法包括 `__init__`, `_raise_if_vocab_size_mismatch`, `_raise_if_draft_tp_mismatch`, `_create_draft_vllm_config`, `_get_model`, `_maybe_share_embeddings`。

### `DraftModelProposer.__init__` method / `DraftModelProposer.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        device: torch.device,
        runner=None,
    ):
        super().__init__(
            vllm_config=vllm_config,
            device=device,
            pass_hidden_states_to_model=False,
            runner=runner,
        )
        self._raise_if_vocab_size_mismatch()
        self._raise_if_draft_tp_mismatch()
```
**EN:** This method initializes the object state within `DraftModelProposer`. Key calls include `__init__`, `_raise_if_vocab_size_mismatch`, `_raise_if_draft_tp_mismatch`, `super`.
**CN:** 该方法会初始化对象状态，其作用域位于`DraftModelProposer`。 关键调用包括 `__init__`, `_raise_if_vocab_size_mismatch`, `_raise_if_draft_tp_mismatch`, `super`。

### `DraftModelProposer._raise_if_vocab_size_mismatch` method / `DraftModelProposer._raise_if_vocab_size_mismatch` 方法
```python
    def _raise_if_vocab_size_mismatch(self):
        self.speculative_config.verify_equal_vocab_size_if_draft_model()
```
**EN:** This method implements `_raise_if_vocab_size_mismatch` within `DraftModelProposer`. Key calls include `verify_equal_vocab_size_if_draft_model`.
**CN:** 该方法会实现 `_raise_if_vocab_size_mismatch`，其作用域位于`DraftModelProposer`。 关键调用包括 `verify_equal_vocab_size_if_draft_model`。

### `DraftModelProposer._raise_if_draft_tp_mismatch` method / `DraftModelProposer._raise_if_draft_tp_mismatch` 方法
```python
    def _raise_if_draft_tp_mismatch(self):
        # Note(Tomas Ruiz) If we run the target model with TP > 1 and
        # the draft model with TP = 1, then the different TP ranks collide.
        # Specifically when all ranks compile the draft model on rank 0
        # (because TP=1), then the torch compile cache is overwritten and corrupted.
        # We need a mechanism like this: https://github.com/vllm-project/vllm/pull/5414
        # To prevent this error, we assert that both TP sizes must be the same.
        spec_cfg = self.speculative_config
        tgt_tp = spec_cfg.target_parallel_config.tensor_parallel_size
        draft_tp = spec_cfg.draft_parallel_config.tensor_parallel_size
        if draft_tp != tgt_tp:
            raise ValueError(
                f"Currently, 'draft_tensor_parallel_size' and 'tensor_parallel_size' "
                f"must be the same. Got {draft_tp} and {tgt_tp}. "
                "Please pass 'draft_tensor_parallel_size' in the speculative_config."
            )
```
**EN:** This method implements `_raise_if_draft_tp_mismatch` within `DraftModelProposer`. Key calls include `ValueError`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_raise_if_draft_tp_mismatch`，其作用域位于`DraftModelProposer`。 关键调用包括 `ValueError`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DraftModelProposer._create_draft_vllm_config` method / `DraftModelProposer._create_draft_vllm_config` 方法
```python
    @override
    def _create_draft_vllm_config(self) -> VllmConfig:
        base = super()._create_draft_vllm_config()
        spec = self.speculative_config

        return replace(
            base,
            quant_config=None,
            parallel_config=replace(
                spec.draft_parallel_config,
                rank=self.vllm_config.parallel_config.rank,
            ),
            model_config=spec.draft_model_config,
        )
```
**EN:** This method implements `_create_draft_vllm_config` within `DraftModelProposer`. Key calls include `_create_draft_vllm_config`, `replace`, `super`.
**CN:** 该方法会实现 `_create_draft_vllm_config`，其作用域位于`DraftModelProposer`。 关键调用包括 `_create_draft_vllm_config`, `replace`, `super`。

### `DraftModelProposer._get_model` method / `DraftModelProposer._get_model` 方法
```python
    @override
    def _get_model(self) -> nn.Module:
        from vllm.compilation.backends import set_model_tag

        draft_vllm_config = self._create_draft_vllm_config()
        with set_model_tag("draft_model"):
            model = get_model(
                vllm_config=draft_vllm_config,
                prefix="draft_model",
            )
        return model
```
**EN:** This method implements `_get_model` within `DraftModelProposer`. Key calls include `_create_draft_vllm_config`, `set_model_tag`, `get_model`.
**CN:** 该方法会实现 `_get_model`，其作用域位于`DraftModelProposer`。 关键调用包括 `_create_draft_vllm_config`, `set_model_tag`, `get_model`。

### `DraftModelProposer._maybe_share_embeddings` method / `DraftModelProposer._maybe_share_embeddings` 方法
```python
    @override
    def _maybe_share_embeddings(self, target_language_model: nn.Module) -> None:
        # Draft models don't share embeddings with the target model
        pass
```
**EN:** This method implements `_maybe_share_embeddings` within `DraftModelProposer`.
**CN:** 该方法会实现 `_maybe_share_embeddings`，其作用域位于`DraftModelProposer`。

### `DraftModelProposer._maybe_share_lm_head` method / `DraftModelProposer._maybe_share_lm_head` 方法
```python
    @override
    def _maybe_share_lm_head(self, target_language_model: nn.Module) -> None:
        # Draft models don't share lm_head with the target model
        pass
```
**EN:** This method implements `_maybe_share_lm_head` within `DraftModelProposer`.
**CN:** 该方法会实现 `_maybe_share_lm_head`，其作用域位于`DraftModelProposer`。

## Key Concepts / 关键概念
- `DraftModelProposer`: central class or interface in this module. / `DraftModelProposer`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`, `typing_extensions`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.config.utils`, `vllm.logger`, `vllm.model_executor.model_loader`, `vllm.v1.spec_decode.llm_base_proposer`, `vllm.compilation.backends`
