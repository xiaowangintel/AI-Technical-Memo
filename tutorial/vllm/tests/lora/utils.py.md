# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/lora/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Utils behavior in the LoRA test area through focused pytest scenarios. It focuses on scenarios such as Dummyloramanager, Close, Punicatensors. / 该文件在 LoRA 测试域中，通过有针对性的 pytest 场景验证 Utils 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-14)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import json
import os
from dataclasses import dataclass

import torch
from safetensors.torch import save_file

from vllm.lora.lora_weights import LoRALayerWeights, PackedLoRALayerWeights
from vllm.platforms import current_platform

DEVICE_TYPE = current_platform.device_type
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `json`, `os`, `torch`, `safetensors.torch`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: DummyLoRAManager (lines 17-95)
```python
class DummyLoRAManager:
    def __init__(self, device: torch.device = f"{DEVICE_TYPE}:0"):
        super().__init__()
        self._loras: dict[str, LoRALayerWeights] = {}
        self._device = device

    def set_module_lora(self, module_name: str, lora: LoRALayerWeights):
        self._loras[module_name] = lora

    def get_module_lora(self, module_name: str) -> LoRALayerWeights:
        return self._loras[module_name]

    def init_random_lora(
        self,
        module_name: str,
        weight: torch.Tensor,
        rank: int = 8,
    ):
        lora = LoRALayerWeights(
# ... omitted for brevity ...
        noop_lora_index_set = set(noop_lora_index or [])

        for i, out_dim in enumerate(output_dims):
            base_lora = self.init_lora(
                module_name + "_000_" + str(i),
                input_dim,
                out_dim,
                rank=rank,
                noop=i in noop_lora_index_set,
            )
            base_loras.append(base_lora)
        packed_lora = PackedLoRALayerWeights.pack(base_loras)
        self.set_module_lora(module_name, packed_lora)
        return packed_lora
```
**EN:** Groups related scenarios for Dummyloramanager.
**CN:** 该类把与 Dummyloramanager 相关的场景组织在一起。

### Helper: assert_close (lines 98-104)
```python
def assert_close(a, b):
    rtol, atol = {
        torch.float16: (6e-2, 6e-2),
        torch.bfloat16: (6e-2, 6e-2),
        torch.float32: (1e-2, 1e-2),
    }[a.dtype]
    torch.testing.assert_close(a, b, rtol=rtol, atol=atol)
```
**EN:** Implements a reusable helper for Close, reducing duplication across related tests. It coordinates operations such as `torch.testing.assert_close`.
**CN:** 该辅助函数为 Close 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `torch.testing.assert_close` 等操作。

### Class: PunicaTensors (lines 107-129)
```python
@dataclass
class PunicaTensors:
    inputs_tensor: torch.Tensor
    lora_weights: torch.Tensor | list[torch.Tensor]
    our_out_tensor: torch.Tensor
    ref_out_tensor: torch.Tensor
    b_seq_start_loc: torch.Tensor
    prompt_lora_mapping: torch.Tensor
    seq_len_tensor: torch.Tensor
    token_lora_mapping: torch.Tensor

    def meta(self) -> tuple[int, int]:
        """
        Infer max_seq_length and token_nums from the tensors
        and return them.
        """
        max_seq_length = self.seq_len_tensor.max()
        token_nums = self.seq_len_tensor.sum().item()
        if isinstance(max_seq_length, tuple):
            max_seq_length = max_seq_length[0].item()
        else:
            max_seq_length = max_seq_length.item()
        return max_seq_length, token_nums
```
**EN:** Groups related scenarios for Punicatensors.
**CN:** 该类把与 Punicatensors 相关的场景组织在一起。

### Helper: generate_data (lines 132-200)
```python
def generate_data(
    batches,
    hidden_size,
    lora_nums,
    max_rank,
    seq_length,
    dtype,
    op_type,
    device,
) -> PunicaTensors:
    seq_len_tensor = torch.randint(seq_length, seq_length + 1, (batches,)).to(device)
    b_seq_start_loc = torch.cumsum(
        torch.tensor([0] + seq_len_tensor[:-1].tolist(), dtype=torch.long),
        dim=0,
    ).to(device)
    total_tokens = seq_len_tensor.sum()
    if op_type == "shrink":
        inputs_tensor = torch.rand((total_tokens, hidden_size), dtype=dtype).to(device)
        lora_weights = torch.rand(
# ... omitted for brevity ...
            lora_index
        )
        current_offset += seq_len_tensor[b_id].item()

    return PunicaTensors(
        inputs_tensor,
        lora_weights,
        our_out_tensor,
        ref_out_tensor,
        b_seq_start_loc,
        lora_indices_tensor,
        seq_len_tensor,
        indices,
    )
```
**EN:** Implements a reusable helper for Generate Data, reducing duplication across related tests. It coordinates operations such as `torch.randint(seq_length, seq_length + 1, (batches,)).to`, `torch.cumsum(torch.tensor([0] + seq_len_tensor[:-1].tolist(), dtype=torch.long), dim=0).to`, `seq_len_tensor.sum`.
**CN:** 该辅助函数为 Generate Data 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `torch.randint(seq_length, seq_length + 1, (batches,)).to`, `torch.cumsum(torch.tensor([0] + seq_len_tensor[:-1].tolist(), dtype=torch.long), dim=0).to`, `seq_len_tensor.sum` 等操作。

### Helper: generate_data_for_expand_nslices (lines 203-260)
```python
def generate_data_for_expand_nslices(
    batches,
    hidden_size,
    lora_nums,
    max_rank,
    seq_length,
    dtype,
    nslices,
    device,
) -> PunicaTensors:
    seq_len_tensor = torch.randint(seq_length, seq_length + 1, (batches,)).to(device)
    b_seq_start_loc = torch.cumsum(
        torch.tensor([0] + seq_len_tensor[:-1].tolist(), dtype=torch.long),
        dim=0,
    ).to(device)
    total_tokens = seq_len_tensor.sum()
    inputs_tensor = torch.rand(
        (total_tokens, max_rank),
        dtype=dtype,
# ... omitted for brevity ...
        )
        current_offset += seq_len_tensor[b_id].item()

    lora_indices_tensor = lora_indices_tensor.to(device)
    return PunicaTensors(
        inputs_tensor,
        lora_weights_lst,
        our_out_tensor,
        ref_out_tensor,
        b_seq_start_loc,
        lora_indices_tensor,
        seq_len_tensor,
        indices,
    )
```
**EN:** Implements a reusable helper for Generate Data For Expand Nslices, reducing duplication across related tests. It coordinates operations such as `torch.randint(seq_length, seq_length + 1, (batches,)).to`, `torch.cumsum(torch.tensor([0] + seq_len_tensor[:-1].tolist(), dtype=torch.long), dim=0).to`, `seq_len_tensor.sum`.
**CN:** 该辅助函数为 Generate Data For Expand Nslices 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `torch.randint(seq_length, seq_length + 1, (batches,)).to`, `torch.cumsum(torch.tensor([0] + seq_len_tensor[:-1].tolist(), dtype=torch.long), dim=0).to`, `seq_len_tensor.sum` 等操作。

### Helper: generate_data_for_nslices (lines 263-341)
```python
def generate_data_for_nslices(
    batches,
    hidden_size,
    lora_nums,
    max_rank,
    seq_length,
    nslices,
    dtype,
    op_type,
    device,
) -> PunicaTensors:
    seq_len_tensor = torch.randint(seq_length, seq_length + 1, (batches,)).to(device)
    b_seq_start_loc = torch.cumsum(
        torch.tensor([0] + seq_len_tensor[:-1].tolist(), dtype=torch.long),
        dim=0,
    ).to(device)
    total_tokens = seq_len_tensor.sum()

    lora_weights_lst = []
# ... omitted for brevity ...
        )
        current_offset += seq_len_tensor[b_id].item()

    lora_indices_tensor = lora_indices_tensor.to(device)
    return PunicaTensors(
        inputs_tensor,
        lora_weights_lst,
        our_out_tensor,
        ref_out_tensor,
        b_seq_start_loc,
        lora_indices_tensor,
        seq_len_tensor,
        indices,
    )
```
**EN:** Implements a reusable helper for Generate Data For Nslices, reducing duplication across related tests. It coordinates operations such as `torch.randint(seq_length, seq_length + 1, (batches,)).to`, `torch.cumsum(torch.tensor([0] + seq_len_tensor[:-1].tolist(), dtype=torch.long), dim=0).to`, `seq_len_tensor.sum`.
**CN:** 该辅助函数为 Generate Data For Nslices 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `torch.randint(seq_length, seq_length + 1, (batches,)).to`, `torch.cumsum(torch.tensor([0] + seq_len_tensor[:-1].tolist(), dtype=torch.long), dim=0).to`, `seq_len_tensor.sum` 等操作。

### Helper: create_peft_lora (lines 344-410)
```python
def create_peft_lora(
    model: torch.nn.Module,
    save_dir: str,
    target_modules: list[str],
    rank: int = 8,
    alpha: int = 16,
    dropout: float = 0.1,
    lora_dtype: torch.dtype = torch.float16,
) -> dict[str, torch.Tensor]:
    lora_weights = {}
    adapter_config = {
        "peft_type": "LORA",
        "auto_mapping": None,
        "base_model_name_or_path": "dummy_model",
        "revision": None,
        "task_type": "CAUSAL_LM",
        "inference_mode": False,
        "r": rank,
        "lora_alpha": alpha,
# ... omitted for brevity ...
        lora_B = torch.zeros(out_features, rank, dtype=lora_dtype)

        # PEFT style
        lora_weights[f"base_model.model.{module_name}.lora_A.weight"] = lora_A
        lora_weights[f"base_model.model.{module_name}.lora_B.weight"] = lora_B

    config_path = os.path.join(save_dir, "adapter_config.json")
    with open(config_path, "w", encoding="utf-8") as f:
        json.dump(adapter_config, f, indent=2, ensure_ascii=False)

    weights_path = os.path.join(save_dir, "adapter_model.safetensors")
    save_file(lora_weights, weights_path)

    return lora_weights
```
**EN:** Implements a reusable helper for Create Peft LoRA, reducing duplication across related tests. It coordinates operations such as `os.path.join`, `save_file`, `module_name.split`.
**CN:** 该辅助函数为 Create Peft LoRA 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `os.path.join`, `save_file`, `module_name.split` 等操作。

## Key Concepts / 关键概念
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **LoRA adapters / LoRA 适配器**
  - **EN:** The scenarios focus on adapter loading, composition, or low-rank update behavior.
  - **CN:** 这些场景关注适配器加载、组合方式或低秩更新行为。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `dataclasses`
- **Third-party / 第三方依赖**: `torch`, `safetensors.torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.lora.lora_weights`, `vllm.platforms`
