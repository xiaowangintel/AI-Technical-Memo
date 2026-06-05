# common_state_dict.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/distributed/common_state_dict.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for common state dict, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 common state dict 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```python
# mypy: allow-untyped-defs

# Owner(s): ["oncall: distributed"]

import copy
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `copy`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`copy`。

### Lines 6-10
```python
from itertools import chain
from typing import Any

import torch
import torch.nn as nn
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.nn`; external imports: `itertools`, `typing`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.nn`；外部导入：`itertools`, `typing`。

### Lines 11-19
```python
from torch.distributed._sharded_tensor import ShardedTensor
from torch.distributed._state_dict_utils import _gather_state_dict
from torch.distributed.checkpoint.state_dict import (
    _PG,
    _STATE,
    set_state_dict,
    StateDictOptions,
)
from torch.distributed.tensor import DTensor
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 20-24
```python


class VerifyStateDictMixin:
    def _compare_tensor(self, orig_tensor, dist_tensor, offload_to_cpu=False):
        if isinstance(dist_tensor, (DTensor, ShardedTensor)):
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `VerifyStateDictMixin`, `_compare_tensor`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`VerifyStateDictMixin`, `_compare_tensor`。

### Lines 25-32
```python
            dist_tensor = _gather_state_dict({"mykey": dist_tensor}).pop("mykey")

        if offload_to_cpu:
            orig_tensor = orig_tensor.cpu()
            dist_tensor = dist_tensor.cpu()
        self.assertTrue(isinstance(dist_tensor, torch.Tensor))
        self.assertTrue(torch.allclose(orig_tensor, dist_tensor))

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 33-40
```python
    def _verify_msd(
        self,
        msd: dict[str, Any],
        dist_msd: dict[str, Any],
        options: StateDictOptions = StateDictOptions(),
        offload_to_cpu=False,
    ) -> None:
        if not options.ignore_frozen_params:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_verify_msd`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_verify_msd`。

### Lines 41-50
```python
            self.assertEqual(len(msd), len(dist_msd))
        for fqn, param in msd.items():
            dist_param = dist_msd.get(fqn)
            if not options.ignore_frozen_params:
                self.assertIsNotNone(dist_param, f"{fqn=}")
                try:
                    self._compare_tensor(param, dist_param, offload_to_cpu)
                except AssertionError as e:
                    raise AssertionError(
                        f"{fqn} has mismatched value {param} {dist_param}"
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 51-55
```python
                    ) from e
            elif dist_param is None:
                self.assertFalse(param.requires_grad, f"{fqn=}")

    def _verify_osd(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_verify_osd`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_verify_osd`。

### Lines 56-65
```python
        self,
        model: nn.Module,
        optim: torch.optim.Optimizer,
        osd: dict[str, Any],
        dist_osd: dict[str, Any],
    ) -> None:
        params = list(chain.from_iterable(g["params"] for g in optim.param_groups))
        param_pid_mapping = dict(zip(params, range(len(params)), strict=True))
        fqn_pid_mapping = {}
        for fqn, param in model.named_parameters():
```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 66-70
```python
            pid = param_pid_mapping[param]
            fqn_pid_mapping[fqn] = pid
            fqn_pid_mapping[pid] = fqn
        # Check optimizer_state_dict state

```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 71-77
```python
        self.assertEqual(len(osd[_STATE]), len(dist_osd[_STATE]))
        for pid, states in osd[_STATE].items():
            fqn = fqn_pid_mapping[pid]
            dist_states = dist_osd[_STATE].get(fqn, None)
            self.assertIsNotNone(dist_states, fqn)
            self.assertEqual(len(states), len(dist_states))
            for key, state in states.items():
```
- EN: This block checks invariants or expected outcomes; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 78-84
```python
                dist_state = states.get(key, None)
                self.assertIsNotNone(dist_state)
                self._compare_tensor(state, dist_state)

        # Check optimizer_state_dict param_group
        old_dist_osd_pg = dist_osd[_PG]
        if len(osd[_PG]) != len(dist_osd[_PG]):
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 85-91
```python
            self.assertTrue(len(dist_osd[_PG]) > len(osd[_PG]))
            new_pg = copy.deepcopy(dist_osd[_PG][0])
            new_pg["params"] = []
            for dist_group in dist_osd[_PG]:
                new_pg["params"].extend(dist_group["params"])
            dist_osd[_PG] = [new_pg]

```
- EN: This block checks invariants or expected outcomes; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 92-101
```python
        self.assertEqual(len(osd[_PG]), len(dist_osd[_PG]))
        for group, dist_group in zip(osd[_PG], dist_osd[_PG], strict=True):
            self.assertEqual(len(group), len(dist_group))
            for key, value in group.items():
                # Below doesn't work because param_groups can have None
                # values.
                # dist_value = dist_group.get(key, None)
                # self.assertIsNotNone(dist_value, (dist_group, group))
                dist_value = dist_group[key]
                if key == "params":
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 102-107
```python
                    fqns = [fqn_pid_mapping[pid] for pid in value]
                    self.assertEqual(sorted(fqns), sorted(dist_value))
                else:
                    self.assertEqual(value, dist_value)
        dist_osd[_PG] = old_dist_osd_pg

```
- EN: This block checks invariants or expected outcomes; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 108-117
```python
    def _verify_osd_by_load(
        self,
        model: nn.Module,
        optim: torch.optim.Optimizer,
        new_optim: torch.optim.Optimizer,
        dist_osd: dict[str, Any],
    ) -> None:
        new_dist_osd = _gather_state_dict(dist_osd)
        set_state_dict(
            model,
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_verify_osd_by_load`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_verify_osd_by_load`。

### Lines 118-123
```python
            optimizers=new_optim,
            model_state_dict={},
            optim_state_dict=new_dist_osd,
        )
        self.assertEqual(optim.state_dict(), new_optim.state_dict())

```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 124-130
```python

class FusionEmbedding(nn.Module):
    def __init__(self, vocab_size: int, fusion_vocab_size: int, embed_dim: int) -> None:
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.fusion_embedding = nn.Embedding(fusion_vocab_size, embed_dim)

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `FusionEmbedding`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`FusionEmbedding`。

### Lines 131-140
```python

class FusionEmbeddingWithHook(nn.Module):
    def __init__(self, vocab_size: int, fusion_vocab_size: int, embed_dim: int) -> None:
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.fusion_embedding = nn.Embedding(fusion_vocab_size, embed_dim)
        self._register_state_dict_hook(FusionEmbeddingWithHook._state_dict_hook)
        self._register_load_state_dict_pre_hook(
            FusionEmbeddingWithHook._load_state_dict_hook, with_module=True
        )
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `FusionEmbeddingWithHook`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`FusionEmbeddingWithHook`。

### Lines 141-145
```python

    def _state_dict_hook(self, destination, prefix, keep_vars):
        """Remove "embedding" from the original embedding in the state_dict
        name. This keeps the original state dict name for the embedding
        from before fusing with the FusionEmbedding.
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_state_dict_hook`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_state_dict_hook`。

### Lines 146-151
```python
        """
        key = prefix + "embedding.weight"
        new_key = prefix + "weight"
        destination[new_key] = destination[key]
        del destination[key]

```
- EN: This block implements local helper logic for common state dict. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common state dict 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 152-156
```python
    def _load_state_dict_hook(self, state_dict, prefix, *args, **kwargs):
        """Apply extra "embedding" prefix to the state_dict key to
        account for the FusionEmbedding wrapping.
        """
        if state_dict:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_load_state_dict_hook`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_load_state_dict_hook`。

### Lines 157-161
```python
            key = prefix + "weight"
            new_key = prefix + "embedding.weight"
            state_dict[new_key] = state_dict[key]
            del state_dict[key]

```
- EN: This block implements local helper logic for common state dict. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common state dict 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 162-167
```python

class FusionEmbeddingWithModifier(FusionEmbeddingWithHook):
    # _fqn_modifiers is a private function as a contract between DSD. When users change the state_dict
    # keys, they need to provide a mapping from the new key to the original key. This is used to ensure
    # consistency between the state_dict keys and fqn.
    def _fqn_modifiers(self) -> dict[str, str]:
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `FusionEmbeddingWithModifier`, `_fqn_modifiers`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`FusionEmbeddingWithModifier`, `_fqn_modifiers`。

### Lines 168-170
```python
        return {
            "weight": "embedding",
        }
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Distributed testing — the implementation coordinates multi-process or sharded training checks.
  CN: 分布式测试——该实现会协调多进程或分片训练相关的检查逻辑。
- EN: Test maintainability — the file helps standardize expectations, fixtures, or generated coverage across PyTorch tests.
  CN: 测试可维护性——该文件有助于在 PyTorch 测试中统一期望、夹具或生成式覆盖。
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.nn`, `torch.distributed._sharded_tensor`, `torch.distributed._state_dict_utils`, `torch.distributed.checkpoint.state_dict`, `torch.distributed.tensor`
- External imports / 外部导入: `copy`, `itertools`, `typing`
- Representative symbols / 代表性符号: `VerifyStateDictMixin`, `FusionEmbedding`, `FusionEmbeddingWithHook`, `FusionEmbeddingWithModifier`
