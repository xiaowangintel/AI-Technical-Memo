# tensor_dump_forward_hook.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/tensor_dump_forward_hook.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on forward-hook based tensor dumping. It mainly captures forward-pass tensors for later inspection. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于基于前向钩子的张量转储。它主要用于捕获前向传播张量以供后续检查。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Document the module intent / 说明模块意图
```python
"""
This file provides a function `register_forward_hook_for_model` that registers a forward hook on every operator of the model.
After registration, during model inference, all tensors generated throughout the forward pass will be recorded.

Usage:
Specify the output directory for dumping tensors using the argument `--debug-tensor-dump-output-folder`.
A separate directory will be created for each GPU rank, named in the format `f"TP{tp_rank}_PP{pp_rank}_Rank{rank}_pid{pid}"`.
Each complete forward pass of the model generates a `.pt` file named `f"Pass{pass_num}.pt"`, which can be loaded using `torch.load`.
The file contains a series of key-value pairs, where the keys correspond to operator names in the model
(similar to those in model.safetensors.index.json), and the values are the outputs produced by the respective operators.
"""
```
**EN:** The docstring states the module intent, so readers can understand the debugging scenario before reading the implementation details.
**CN:** 文档字符串先说明模块意图，让读者在进入实现细节之前就能理解对应的调试场景。

### Lines 13-21: Import dependencies and shared types / 导入依赖与共享类型
```python
import logging
import os
from pathlib import Path
from typing import List, Optional

import torch

from sglang.srt.layers.logits_processor import LogitsProcessorOutput
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, PPProxyTensors
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 23-23: Declare module-level symbols such as `logger` / 声明模块级符号，例如 `logger`
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 26-26: Define class `TensorDumper` and class context / 定义类 `TensorDumper`及类上下文
```python
class TensorDumper:
```
**EN:** This section introduces `TensorDumper`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `TensorDumper`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 27-44: Implement method `__init__` for `TensorDumper` / 为 `TensorDumper` 实现方法 `__init__`
```python
    def __init__(
        self,
        dump_dir: str,
        dump_layers: Optional[List[int]],
        tp_size: int,
        tp_rank: int,
        pp_rank: int,
    ):
        self._dump_layers = dump_layers
        self._forward_pass_id = 0
        self._pid = os.getpid()
        self._current_tensors = {}
        self._base_dir = Path(dump_dir)
        rank = tp_size * pp_rank + tp_rank
        self._process_dir = (
            self._base_dir / f"TP{tp_rank}_PP{pp_rank}_Rank{rank}_pid{self._pid}"
        )
        self._process_dir.mkdir(parents=True, exist_ok=True)
```
**EN:** Method `__init__` implements behavior on `TensorDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `TensorDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 46-47: Implement method `get_dump_dir` for `TensorDumper` / 为 `TensorDumper` 实现方法 `get_dump_dir`
```python
    def get_dump_dir(self):
        return str(self._process_dir)
```
**EN:** Method `get_dump_dir` implements behavior on `TensorDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `get_dump_dir` 为 `TensorDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 49-76: Implement method `add_tensor` for `TensorDumper` / 为 `TensorDumper` 实现方法 `add_tensor`
```python
    def add_tensor(self, name, tensor_item):
        if isinstance(tensor_item, (tuple, list)):
            tensors = [t.cpu() for t in tensor_item if t is not None]
            if len(tensors) == 1:
                self._current_tensors[name] = tensors[0]
            else:
                self._current_tensors[name] = tensors
        elif isinstance(tensor_item, torch.Tensor):
            self._current_tensors[name] = tensor_item.cpu()
        elif isinstance(tensor_item, LogitsProcessorOutput):
            self._current_tensors[name] = tensor_item.next_token_logits.cpu()
        elif isinstance(tensor_item, ForwardBatch):
            self._current_tensors[name + ".forward_batch_info.input_ids"] = (
                tensor_item.input_ids.cpu()
            )
            self._current_tensors[name + ".forward_batch_info.seq_lens"] = (
                tensor_item.seq_lens.cpu()
            )
            self._current_tensors[name + ".forward_batch_info.positions"] = (
                tensor_item.positions.cpu()
            )
        elif isinstance(tensor_item, PPProxyTensors):
            for tensor_name in tensor_item.tensors.keys():
                self._current_tensors[name + ".pp_proxy_tensors." + tensor_name] = (
                    tensor_item.tensors[tensor_name].cpu()
                )
        else:
            logger.warning(f"Unsupported type: {type(tensor_item)}: {tensor_item}")
```
**EN:** Method `add_tensor` implements behavior on `TensorDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `add_tensor` 为 `TensorDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 78-87: Implement method `dump_current_tensors` for `TensorDumper` / 为 `TensorDumper` 实现方法 `dump_current_tensors`
```python
    def dump_current_tensors(self):
        if len(self._current_tensors) == 0:
            return
        tensor_file_for_pass = self._process_dir / f"Pass{self._forward_pass_id:05d}.pt"
        logger.info(
            f"Dump {self._forward_pass_id:05d}th pass to {tensor_file_for_pass}"
        )
        torch.save(self._current_tensors, str(tensor_file_for_pass))
        self._current_tensors = {}
        self._forward_pass_id += 1
```
**EN:** Method `dump_current_tensors` implements behavior on `TensorDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `dump_current_tensors` 为 `TensorDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 89-125: Implement method `_add_hook_recursive` for `TensorDumper` / 为 `TensorDumper` 实现方法 `_add_hook_recursive`
```python
    def _add_hook_recursive(
        self, model, prefix, top_level_module_name, layers_module_name
    ):
        model_top_level_module_matched = False
        layers_prefix = top_level_module_name + "." + layers_module_name
        for name, module in model._modules.items():
            top_level_model = False
            if len(prefix) == 0:
                cur_name = name
                if cur_name == top_level_module_name:
                    model_top_level_module_matched = True
                    top_level_model = True
            else:
                cur_name = prefix + "." + name
            if (
                self._dump_layers is not None
                and name.isdigit()
                and prefix == layers_prefix
            ):
                # If we only need n layers, skip the reset layers.
                # Most models' layout is like model.layers.0.
                cur_layer = int(name)
                if cur_layer not in self._dump_layers:
                    continue
            if module is not None:
                _, sub_count = self._add_hook_recursive(
                    module, cur_name, top_level_module_name, layers_module_name
                )
                if sub_count == 0 or top_level_model:
                    # Avoid duplicated output hooks, e.g. self_attn may contain:
                    # self_attn.qkv_proj, self_attn.attn & self_attn.o_proj.
                    # Therefore, we do not need to add output hooks for self_attn,
                    # since the output of self_attn should be the same to self_attn.o_proj.
                    module.register_forward_hook(
                        self._dump_hook(cur_name, top_level_model)
                    )
        return model_top_level_module_matched, len(model._modules.items())
```
**EN:** Method `_add_hook_recursive` implements behavior on `TensorDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_add_hook_recursive` 为 `TensorDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 127-138: Implement method `_dump_hook` for `TensorDumper` / 为 `TensorDumper` 实现方法 `_dump_hook`
```python
    def _dump_hook(self, tensor_name, do_dump):
        def inner_dump_hook(module, input, output):
            if do_dump:
                # This is the top-level model, so we will record the input for it.
                for item in input:
                    if isinstance(item, ForwardBatch):
                        self.add_tensor(tensor_name, item)
                self.dump_current_tensors()
            if output is not None:
                self.add_tensor(tensor_name, output)

        return inner_dump_hook
```
**EN:** Method `_dump_hook` implements behavior on `TensorDumper`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_dump_hook` 为 `TensorDumper` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 141-164: Implement function `register_forward_hook_for_model` / 实现函数 `register_forward_hook_for_model`
```python
def register_forward_hook_for_model(
    model,
    dump_dir: str,
    dump_layers: Optional[List[int]],
    tp_size: int,
    tp_rank: int,
    pp_rank: int,
):
    tensor_dumper = TensorDumper(dump_dir, dump_layers, tp_size, tp_rank, pp_rank)
    # Most models have the layerout like:
    # XxxxForCausalLM
    #     (model): XxxxModel
    #         (layers): ModuleList
    # If the model is not constructed with this layout,
    # environment variable can be used to specify the module names.
    top_level_module_name = os.getenv("TENSOR_DUMP_TOP_LEVEL_MODULE_NAME", "model")
    layers_module_name = os.getenv("TENSOR_DUMP_LAYERS_MODULE_NAME", "layers")
    model_top_level_module_matched, _ = tensor_dumper._add_hook_recursive(
        model, "", top_level_module_name, layers_module_name
    )
    assert (
        model_top_level_module_matched
    ), f"model should have a module named {top_level_module_name}"
    return tensor_dumper
```
**EN:** Function `register_forward_hook_for_model` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `register_forward_hook_for_model` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `TensorDumper`, `register_forward_hook_for_model`
- **Module role / 模块角色**: Forward-hook based tensor dumping / 基于前向钩子的张量转储
- **Implementation focus / 实现重点**: Captures forward-pass tensors for later inspection / 捕获前向传播张量以供后续检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `os`, `pathlib`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal / 内部**: `sglang.srt.layers.logits_processor`, `sglang.srt.model_executor.forward_batch_info`
