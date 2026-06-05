# test_nvfp4_emulation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_nvfp4_emulation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_nvfp4_emulation, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_nvfp4_emulation 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-16)
```python
import huggingface_hub
import pytest
import torch
from safetensors import safe_open

from vllm.model_executor.layers.quantization.utils import (
    nvfp4_emulation_utils,
)
from vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils import (
    dequantize_to_dtype,
    ref_nvfp4_quant_dequant,
)
from vllm.platforms import current_platform
from vllm.triton_utils import triton
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as huggingface_hub, pytest, torch, safetensors; and vLLM components like vllm.model_executor.layers.quantization.utils, vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils, vllm.platforms, vllm.triton_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 huggingface_hub、pytest、torch、safetensors；vLLM 内部组件，例如 vllm.model_executor.layers.quantization.utils、vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils、vllm.platforms、vllm.triton_utils。

### Function `test_triton_dequantize_nvfp4` (lines 19-206)
```python
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(),
    reason="Triton NVFP4 kernel requires CUDA.",
)
def test_triton_dequantize_nvfp4(monkeypatch) -> None:
    """Test the Triton dequantization kernel against the CPU reference
    using real NVFP4 weights from a checkpoint.

    Tests both 2D (attention projection) and 3D (stacked MoE experts).
    """
    checkpoint_path = huggingface_hub.snapshot_download(
        "nvidia/Qwen3-30B-A3B-NVFP4",
        allow_patterns=["model-00001-of-00004.safetensors"],
    )
    shard_path = f"{checkpoint_path}/model-00001-of-00004.safetensors"
    block_size = 16

    with safe_open(shard_path, framework="pt", device="cpu") as f:
        all_keys = list(f.keys())

        # 2D case: attention projection
        tensor_fp4_2d = f.get_tensor("model.layers.9.self_attn.k_proj.weight")
        tensor_sf_2d = f.get_tensor("model.layers.9.self_attn.k_proj.weight_scale")
        global_scale_2d = f.get_tensor("model.layers.9.self_attn.k_proj.weight_scale_2")

        # 3D case: stack ALL experts for layer 9 up_proj
        expert_prefix = "model.layers.9.mlp.experts."
        expert_indices = sorted(
            int(key.split(".")[5])
            for key in all_keys
            if key.startswith(expert_prefix) and key.endswith(".up_proj.weight")
        )
        assert len(expert_indices) > 0

        all_fp4 = []
        all_sf = []
        all_global_scale = []
        for index in expert_indices:
            name = f"{expert_prefix}{index}.up_proj"
            all_fp4.append(f.get_tensor(f"{name}.weight"))
# ... excerpt ...
            _triton_bench, quantiles=quantiles
        )

        def _reference_bench(
            fp4_cuda=fp4_cuda,
            scale_cuda=sf_cuda,
            global_scale_cuda=gs_cuda,
            block_size=block_size,
        ):
            with monkeypatch.context() as m2:
                m2.setattr(
                    nvfp4_emulation_utils.current_platform,
                    "is_cuda_alike",
                    lambda: False,
                )
                dequantize_to_dtype(
                    fp4_cuda,
                    scale_cuda,
                    global_scale_cuda,
                    torch.bfloat16,
                    block_size,
                    swizzle=False,
                )

        ref_ms, ref_min, ref_max = triton.testing.do_bench(
            _reference_bench, quantiles=quantiles
        )

        speedup = ref_ms / triton_ms if triton_ms > 0 else float("inf")
        print(f"  dequantize {label} {shape}:")
        print(
            f"    triton:    median={triton_ms:.3f}ms, "
            f"min={triton_min:.3f}ms, max={triton_max:.3f}ms"
        )
        print(
            f"    reference: median={ref_ms:.3f}ms, "
            f"min={ref_min:.3f}ms, max={ref_max:.3f}ms"
        )
        print(f"    speedup:   {speedup:.2f}x")
```
**EN:** This pytest case verifies triton dequantize nvfp4. it consumes fixtures or inputs such as monkeypatch. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 triton dequantize nvfp4 的行为。 它会使用诸如 monkeypatch 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

### Function `test_triton_nvfp4_quant_dequant` (lines 209-308)
```python
@pytest.mark.skipif(
    not current_platform.is_cuda_alike(),
    reason="Triton NVFP4 kernel requires CUDA.",
)
@pytest.mark.parametrize(
    "m, k",
    [
        (1, 16),
        (1, 4096),
        (2, 4096),
        (4, 4096),
        (8, 4096),
        (16, 4096),
        (24, 4096),
        (32, 4096),
        (1, 8192),
        (2, 8192),
        (4, 8192),
        (8, 8192),
        (16, 8192),
        (24, 8192),
        (32, 8192),
        (1, 32),
        (2, 48),
        (7, 64),
        (16, 128),
        (33, 160),
        (128, 256),
        (256, 512),
        (1024, 1024),
        (5120, 2048),
        (2048, 4096),
        (4096, 7168),
        (8192, 8192),
        (128, 16384),
    ],
)
@pytest.mark.parametrize("global_scale_value", [0.5, 1.0, 0.001])
def test_triton_nvfp4_quant_dequant(
    monkeypatch, m: int, k: int, global_scale_value: float
# ... excerpt ...
    quantiles = [0.5, 0.001, 0.999]

    def _triton_bench(
        input_tensor=x, input_global_scale=global_scale, input_block_size=block_size
    ):
        return ref_nvfp4_quant_dequant(
            input_tensor, input_global_scale, input_block_size
        )

    triton_ms, triton_min, triton_max = triton.testing.do_bench(
        _triton_bench, quantiles=quantiles
    )

    def _reference_bench(
        input_tensor=x, input_global_scale=global_scale, input_block_size=block_size
    ):
        with monkeypatch.context() as mp2:
            mp2.setattr(
                nvfp4_emulation_utils.current_platform,
                "is_cuda_alike",
                lambda: False,
            )
            ref_nvfp4_quant_dequant(input_tensor, input_global_scale, input_block_size)

    ref_ms, ref_min, ref_max = triton.testing.do_bench(
        _reference_bench, quantiles=quantiles
    )

    speedup = ref_ms / triton_ms if triton_ms > 0 else float("inf")
    print(f"  quant_dequant [{m}x{k}] gs={global_scale_value}:")
    print(
        f"    triton:    median={triton_ms:.3f}ms, "
        f"min={triton_min:.3f}ms, max={triton_max:.3f}ms"
    )
    print(
        f"    reference: median={ref_ms:.3f}ms, "
        f"min={ref_min:.3f}ms, max={ref_max:.3f}ms"
    )
    print(f"    speedup:   {speedup:.2f}x")
```
**EN:** This pytest case verifies triton nvfp4 quant dequant. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as monkeypatch, m, k, global_scale_value. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 triton nvfp4 quant dequant 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 monkeypatch、m、k、global_scale_value 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `huggingface_hub`
- `pytest`
- `torch`
- `safetensors -> safe_open`
- `vllm.model_executor.layers.quantization.utils -> nvfp4_emulation_utils`
- `vllm.model_executor.layers.quantization.utils.nvfp4_emulation_utils -> dequantize_to_dtype, ref_nvfp4_quant_dequant`
- `vllm.platforms -> current_platform`
- `vllm.triton_utils -> triton`
