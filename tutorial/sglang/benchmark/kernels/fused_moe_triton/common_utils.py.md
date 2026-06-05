# common_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/fused_moe_triton/common_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels fused moe triton common. It primarily measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets. / 该 Python 模块聚焦于 kernels fused moe triton common 相关流程。它主要用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and setup / 导入与初始化
```python
import json
from typing import Dict, List, TypedDict

import torch

from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import get_config_dtype_str
from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe_triton_config import (
    get_config_file_name,
)
from sglang.srt.utils import is_hip
from sglang.srt.utils.hf_transformers_utils import get_config
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it prepares tensors and invokes GPU kernels, loads, filters, or serializes benchmark datasets, and processes tokenized prompts or decoded outputs.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。

### Lines 14-14: Class `BenchmarkConfig` declaration / 类 `BenchmarkConfig` 声明
```python
class BenchmarkConfig(TypedDict):
```
**EN:** This block introduces class `BenchmarkConfig`, which measures runtime latency, throughput, or other benchmark metrics.
**CN:** 该代码块引入类 `BenchmarkConfig`，用于测量运行时延迟、吞吐或其他基准指标。

### Lines 15-20: Class-level state / 类级状态
```python
    BLOCK_SIZE_M: int
    BLOCK_SIZE_N: int
    BLOCK_SIZE_K: int
    GROUP_SIZE_M: int
    num_warps: int
    num_stages: int
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。

### Lines 23-29: Function `calculate_shard_intermediate_size` / 函数 `calculate_shard_intermediate_size`
```python
def calculate_shard_intermediate_size(
    intermediate_size: int, tp_size: int, ep_size: int = 1
) -> int:
    assert tp_size % ep_size == 0
    moe_tp_size = tp_size // ep_size
    assert intermediate_size % moe_tp_size == 0
    return 2 * intermediate_size // moe_tp_size
```
**EN:** `calculate_shard_intermediate_size` is a function that implements the core logic for this scope. It returns `2 * intermediate_size // moe_tp_size` to the caller.
**CN:** `calculate_shard_intermediate_size` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `2 * intermediate_size // moe_tp_size`。

### Lines 32-169: Function `get_model_config` / 函数 `get_model_config`
```python
def get_model_config(
    model_name: str,
    tp_size: int,
    ep_size: int = 1,
    disable_shared_experts_fusion: bool = False,
    topk_ids_dir: str = None,
) -> Dict:
    config = get_config(model_name, trust_remote_code=True)
    architecture = config.architectures[0]
    block_shape = None
    if (
        hasattr(config, "quantization_config")
        and "weight_block_size" in config.quantization_config
    ):
        block_shape = config.quantization_config["weight_block_size"]
        assert len(block_shape) == 2

    if (
        hasattr(config, "quantization_config")
        and "config_groups" in config.quantization_config
    ):
        config_groups = config.quantization_config["config_groups"]
        # Get group_size from the first group's weights config
        first_group = next(iter(config_groups.values()), {})
        weights_config = first_group.get("weights", {})
        group_size = weights_config.get("group_size")
        block_shape = [0, group_size]
        assert len(block_shape) == 2
    # Replace config with text_config for encoder-decoder models after getting block_shape and architecture
    if hasattr(config, "text_config"):
        config = config.get_text_config()

    hidden_size = config.hidden_size
    if architecture == "DbrxForCausalLM":
        E = config.ffn_config.moe_num_experts // ep_size
        topk = config.ffn_config.moe_top_k
        intermediate_size = config.ffn_config.ffn_hidden_size
    elif architecture == "JambaForCausalLM":
        E = config.num_experts // ep_size
        topk = config.num_experts_per_tok
        intermediate_size = config.intermediate_size
    elif architecture in [
        "Qwen2MoeForCausalLM",
        "Qwen3MoeForCausalLM",
        "Qwen3NextForCausalLM",
        "Qwen3VLMoeForConditionalGeneration",
        "Qwen3_5MoeForConditionalGeneration",
        "InternS2PreviewForConditionalGeneration",
    ]:
        E = config.num_experts // ep_size
        topk = config.num_experts_per_tok
        intermediate_size = config.moe_intermediate_size
    elif architecture in [
        "DeepseekV2ForCausalLM",
        "DeepseekV3ForCausalLM",
        "DeepseekV32ForCausalLM",
        "DeepseekV4ForCausalLM",
        "Glm4MoeForCausalLM",
        "GlmMoeDsaForCausalLM",
        "MistralLarge3ForCausalLM",
    ]:
        E = (config.n_routed_experts // ep_size) + (
            0
            if disable_shared_experts_fusion
            or architecture
            not in [
                "DeepseekV3ForCausalLM",
                "DeepseekV32ForCausalLM",
                "Glm4MoeForCausalLM",
                "GlmMoeDsaForCausalLM",
                "MistralLarge3ForCausalLM",
            ]
            else 1
        )
        topk = config.num_experts_per_tok + (
            0 if disable_shared_experts_fusion or topk_ids_dir is None else 1
        )
        intermediate_size = config.moe_intermediate_size
    elif architecture == "Llama4ForConditionalGeneration":
        E = config.num_local_experts // ep_size + (
            0 if disable_shared_experts_fusion else 1
        )
        topk = config.num_experts_per_tok + (
            0 if disable_shared_experts_fusion or topk_ids_dir is None else 1
        )
        intermediate_size = config.intermediate_size
    elif architecture in [
        "Grok1ForCausalLM",
        "Grok1ImgGen",
        "Grok1AForCausalLM",
    ]:
        E = config.num_local_experts // ep_size
        topk = config.num_experts_per_tok
        intermediate_size = config.moe_intermediate_size
    elif architecture in [
        "BailingMoEForCausalLM",
        "BailingMoeForCausalLM",
        "BailingMoeV2ForCausalLM",
    ]:
        E = config.num_experts // ep_size
        topk = config.num_experts_per_tok
        intermediate_size = config.moe_intermediate_size
    elif architecture == "HYV3ForCausalLM":
        E = config.num_experts // ep_size
        topk = config.num_experts_per_tok
        intermediate_size = config.expert_hidden_dim
    elif architecture == "NemotronHForCausalLM":
        E = config.n_routed_experts // ep_size
        topk = config.num_experts_per_tok
        intermediate_size = config.moe_intermediate_size
        hidden_size = getattr(config, "moe_latent_size", None) or hidden_size
    elif architecture == "Gemma4ForConditionalGeneration":
        E = config.num_experts // ep_size
        topk = config.top_k_experts
        intermediate_size = config.moe_intermediate_size
    elif architecture == "Lfm2MoeForCausalLM":
        E = config.num_experts // ep_size
        topk = config.num_experts_per_tok
        intermediate_size = config.moe_intermediate_size
    else:
        # Default: Mixtral
        E = config.num_local_experts // ep_size
        topk = config.num_experts_per_tok
        intermediate_size = config.intermediate_size

    shard_intermediate_size = calculate_shard_intermediate_size(
        intermediate_size, tp_size, ep_size
    )

    return {
        "num_experts": E,
        "topk": topk,
        "hidden_size": hidden_size,
        "shard_intermediate_size": shard_intermediate_size,
        "dtype": config.torch_dtype,
        "block_shape": block_shape,
        "architecture": architecture,
    }
```
**EN:** `get_model_config` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `{'num_experts': E, 'topk': topk, 'hidden_size': hidden_size, 'shard_intermediate_size': shard_int...` to the caller. Notable calls include `get_config`, `hasattr`, `calculate_shard_intermediate_size`.
**CN:** `get_model_config` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `{'num_experts': E, 'topk': topk, 'hidden_size': hidden_size, 'shard_intermediate_size': shard_int...`。其中较关键的调用包括 `get_config`, `hasattr`, `calculate_shard_intermediate_size`。

### Lines 172-192: Function `get_rocm_configs_compute_bound` / 函数 `get_rocm_configs_compute_bound`
```python
def get_rocm_configs_compute_bound() -> List[Dict[str, int]]:
    configs: List[BenchmarkConfig] = []
    waves_per_eu_range = 0
    for num_stages in [2]:
        for block_m in [32, 64, 128, 256]:
            for block_k in [32, 64, 128, 256]:
                for block_n in [16, 32, 64, 128, 256]:
                    for num_warps in [1, 2, 4, 8]:
                        for group_size in [1, 4, 8, 16, 32]:
                            configs.append(
                                {
                                    "BLOCK_SIZE_M": block_m,
                                    "BLOCK_SIZE_N": block_n,
                                    "BLOCK_SIZE_K": block_k,
                                    "GROUP_SIZE_M": group_size,
                                    "num_warps": num_warps,
                                    "num_stages": num_stages,
                                    "waves_per_eu": waves_per_eu_range,
                                }
                            )
    return configs
```
**EN:** `get_rocm_configs_compute_bound` is a function that measures runtime latency, throughput, or other benchmark metrics and handles audio loading, conversion, or packaging. It returns `configs` to the caller. Notable calls include `configs.append`.
**CN:** `get_rocm_configs_compute_bound` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、处理音频加载、转换或打包。它会向调用方返回 `configs`。其中较关键的调用包括 `configs.append`。

### Lines 195-216: Function `get_configs_compute_bound` / 函数 `get_configs_compute_bound`
```python
def get_configs_compute_bound() -> List[Dict[str, int]]:
    configs: List[BenchmarkConfig] = []
    if is_hip():
        configs = get_rocm_configs_compute_bound()
    else:
        for num_stages in [2, 3, 4, 5]:
            for block_m in [16, 32, 64, 128, 256]:
                for block_k in [64, 128, 256]:
                    for block_n in [32, 64, 128, 256]:
                        for num_warps in [4, 8]:
                            for group_size in [1, 16, 32, 64]:
                                configs.append(
                                    {
                                        "BLOCK_SIZE_M": block_m,
                                        "BLOCK_SIZE_N": block_n,
                                        "BLOCK_SIZE_K": block_k,
                                        "GROUP_SIZE_M": group_size,
                                        "num_warps": num_warps,
                                        "num_stages": num_stages,
                                    }
                                )
    return configs
```
**EN:** `get_configs_compute_bound` is a function that measures runtime latency, throughput, or other benchmark metrics. It returns `configs` to the caller. Notable calls include `is_hip`, `get_rocm_configs_compute_bound`, `configs.append`.
**CN:** `get_configs_compute_bound` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `configs`。其中较关键的调用包括 `is_hip`, `get_rocm_configs_compute_bound`, `configs.append`。

### Lines 219-231: Function `sort_config` / 函数 `sort_config`
```python
def sort_config(config: BenchmarkConfig) -> BenchmarkConfig:
    return {
        "BLOCK_SIZE_M": config["BLOCK_SIZE_M"],
        "BLOCK_SIZE_N": config["BLOCK_SIZE_N"],
        "BLOCK_SIZE_K": config["BLOCK_SIZE_K"],
        "GROUP_SIZE_M": config["GROUP_SIZE_M"],
        "num_warps": config["num_warps"],
        "num_stages": config["num_stages"],
        **(
            {"waves_per_eu": config["waves_per_eu"]} if "waves_per_eu" in config else {}
        ),
        **({"USE_TMA": config["USE_TMA"]} if "USE_TMA" in config else {}),
    }
```
**EN:** `sort_config` is a function that measures runtime latency, throughput, or other benchmark metrics and handles audio loading, conversion, or packaging. It returns `{'BLOCK_SIZE_M': config['BLOCK_SIZE_M'], 'BLOCK_SIZE_N': config['BLOCK_SIZE_N'], 'BLOCK_SIZE_K':...` to the caller.
**CN:** `sort_config` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、处理音频加载、转换或打包。它会向调用方返回 `{'BLOCK_SIZE_M': config['BLOCK_SIZE_M'], 'BLOCK_SIZE_N': config['BLOCK_SIZE_N'], 'BLOCK_SIZE_K':...`。

### Lines 234-241: Function `save_configs` / 函数 `save_configs`
```python
def save_configs(
    configs: Dict[int, BenchmarkConfig],
    filename: str,
) -> None:
    print(f"Writing best config to {filename}...")
    with open(filename, "w") as f:
        json.dump(configs, f, indent=4)
        f.write("\n")
```
**EN:** `save_configs` is a function that measures runtime latency, throughput, or other benchmark metrics, loads, filters, or serializes benchmark datasets, and reads or writes local files and intermediate artifacts. Notable calls include `print`, `open`, `json.dump`.
**CN:** `save_configs` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。其中较关键的调用包括 `print`, `open`, `json.dump`。

### Lines 244-279: Function `get_config_filename` / 函数 `get_config_filename`
```python
def get_config_filename(
    num_experts: int,
    shard_intermediate_size: int,
    hidden_size: int,
    topk: int,
    dtype: torch.dtype,
    use_fp8_w8a8: bool,
    use_int8_w8a8: bool,
    use_int8_w8a16: bool,
    use_int4_w4a16: bool,
    per_channel_quant: bool,
    block_shape: List[int],
) -> str:
    dtype_str = get_config_dtype_str(
        dtype,
        use_int8_w8a16=use_int8_w8a16,
        use_fp8_w8a8=use_fp8_w8a8,
        use_int8_w8a8=use_int8_w8a8,
        use_int4_w4a16=use_int4_w4a16,
    )

    # NOTE(woosuk): The current naming convention uses w2.shape[2], which
    # is the intermediate size after silu_and_mul.
    N = shard_intermediate_size // 2
    if use_int4_w4a16:
        N = N // 2

    filename = get_config_file_name(
        num_experts,
        N,
        dtype_str,
        block_shape,
        per_channel_quant,
    )

    return filename
```
**EN:** `get_config_filename` is a function that prepares tensors and invokes GPU kernels. It returns `filename` to the caller. Notable calls include `get_config_dtype_str`, `get_config_file_name`.
**CN:** `get_config_filename` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `filename`。其中较关键的调用包括 `get_config_dtype_str`, `get_config_file_name`。

### Lines 282-302: Function `get_default_batch_sizes` / 函数 `get_default_batch_sizes`
```python
def get_default_batch_sizes() -> List[int]:
    return [
        1,
        2,
        4,
        8,
        16,
        24,
        32,
        48,
        64,
        96,
        128,
        256,
        512,
        1024,
        1536,
        2048,
        3072,
        4096,
    ]
```
**EN:** `get_default_batch_sizes` is a function that implements the core logic for this scope. It returns `[1, 2, 4, 8, 16, 24, 32, 48, 64, 96, 128, 256, 512, 1024, 1536, 2048, 3072, 4096]` to the caller.
**CN:** `get_default_batch_sizes` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `[1, 2, 4, 8, 16, 24, 32, 48, 64, 96, 128, 256, 512, 1024, 1536, 2048, 3072, 4096]`。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Audio processing / 音频处理**: Prepares waveform data for speech or streaming benchmarks. / 为语音或流式基准测试准备波形数据。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal / 项目内部依赖**: `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe`, `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe_triton_config`, `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`
