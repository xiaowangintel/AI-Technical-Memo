# kvfp4_tensor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/kvfp4_tensor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for kvfp4 tensor quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 kvfp4 张量 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module imports and setup
```python
# Copyright 2025 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

# Define a enum class for FP4 formats, including MXFP4, NVFP4 and future formats
from enum import Enum

import torch
```
**EN:** This block imports enum, flashinfer, torch, sglang.srt.utils and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 enum, flashinfer, torch, sglang.srt.utils 等依赖，并为当前量化实现准备模块命名空间。

### Lines 21-21: class FP4KVCacheRecipe: definition
```python
class FP4KVCacheRecipe(Enum):
```
**EN:** This block declares `FP4KVCacheRecipe`, a supporting class for the quantization stack. It organizes behaviors such as class state.
**CN:** 该代码块声明 `FP4KVCacheRecipe`，它是量化栈中的支撑类，组织了 class state 等行为。

### Lines 22-22: FP4KVCacheRecipe member: initialize MXFP4
```python
    MXFP4 = 1  # KVFP4: block-wise scaling
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as MXFP4.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 MXFP4。

### Lines 23-23: FP4KVCacheRecipe member: initialize NVFP4
```python
    NVFP4 = 2  # two-level scaling: global FP32 + block FP8 E4M3
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as NVFP4.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 NVFP4。

### Lines 26-26: initialize E2M1_MAX
```python
E2M1_MAX = 6.0
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as E2M1_MAX.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 E2M1_MAX。

### Lines 27-27: initialize MAX_BLOCK_SCALE_FP8
```python
MAX_BLOCK_SCALE_FP8 = 448.0  # Maximum FP8 E4M3 value
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as MAX_BLOCK_SCALE_FP8.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 MAX_BLOCK_SCALE_FP8。

### Lines 29-29: initialize _device
```python
_device = "cuda" if torch.cuda.is_available() else "cpu"
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _device.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _device。

### Lines 34-55: initialize E2M1_VALUES
```python
E2M1_VALUES = torch.tensor(
    [
        0,
        0.5,
        1,
        1.5,
        2,
        3,
        4,
        6,  # 0x0-0x7: positive values
        -0,
        -0.5,
        -1,
        -1.5,
        -2,
        -3,
        -4,
        -6,
    ],  # 0x8-0xF: negative values
    dtype=torch.float32,
    device=_device,
)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as E2M1_VALUES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 E2M1_VALUES。

### Lines 56-58: initialize E2M1_BOUNDS
```python
E2M1_BOUNDS = torch.tensor(
    [0.25, 0.75, 1.25, 1.75, 2.5, 3.5, 5], dtype=torch.float32, device=_device
)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as E2M1_BOUNDS.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 E2M1_BOUNDS。

### Lines 61-67: class BlockFP4KVQuantizeUtil: definition
```python
class BlockFP4KVQuantizeUtil:
    """Block-wise FP4 (E2M1) quantization for KV cache.

    Similar to MXFP4 but uses block_size=16 (MXFP4 spec defines block_size=32).
    Each block of 16 elements shares one uint8 exponent-only scale factor.
    """
```
**EN:** This block declares `BlockFP4KVQuantizeUtil`, a supporting class for the quantization stack. It organizes behaviors such as batched_quantize, batched_dequantize.
**CN:** 该代码块声明 `BlockFP4KVQuantizeUtil`，它是量化栈中的支撑类，组织了 batched_quantize, batched_dequantize 等行为。

### Lines 68-107: BlockFP4KVQuantizeUtil.batched_quantize()
```python
    @staticmethod
    @torch.compile
    def batched_quantize(tensor: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Quantize tensor to KVFP4 format
        Args:
            tensor: Input tensor of shape [B, M, N]

        Returns:
            quant_tensor: Quantized tensor of shape [B, M, N/2]
            scale_factors: Scale factors of shape [B, M*N/16]
        """
        b, m, n = tensor.shape

        # Reshape to [B, M*N/16, 16] for block-wise quantization
        reshaped = tensor.view(b, m * n // 16, 16)

        # Compute scale factors per block
        block_max = reshaped.abs().max(dim=-1, keepdim=True).values
        scale_exp = torch.ceil(torch.log2(torch.clamp(block_max / E2M1_MAX, min=1e-10)))
        scale_factors = (scale_exp + 127).squeeze(-1).to(torch.uint8)

        # Apply scaling
        scaled = reshaped / torch.exp2(scale_exp)

        # Quantize to FP4
        sign_bits = (scaled < 0).to(torch.uint8) << 3
        abs_vals = scaled.abs()

        # Pure tensor version (CUDA Graph safe)
        magnitude_bits = torch.sum(abs_vals.unsqueeze(-1) >= E2M1_BOUNDS, dim=-1)

        # Combine sign and magnitude
        fp4_vals = sign_bits + magnitude_bits.to(torch.uint8)

        # Pack two FP4 values into one uint8
        fp4_reshaped = fp4_vals.view(b, m, n)
        packed = (fp4_reshaped[..., 1::2] << 4) + fp4_reshaped[..., 0::2]

        return packed, scale_factors
```
**EN:** This block defines `BlockFP4KVQuantizeUtil.batched_quantize()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `BlockFP4KVQuantizeUtil.batched_quantize()`，用于处理张量或权重量化逻辑。

### Lines 109-149: BlockFP4KVQuantizeUtil.batched_dequantize()
```python
    @staticmethod
    @torch.compile
    def batched_dequantize(
        quant_tensor: torch.Tensor,
        scale_factors: torch.Tensor,
        dtype: torch.dtype = torch.bfloat16,
    ) -> torch.Tensor:
        """
        Dequantize KVFP4 tensor
        Args:
            quant_tensor: Quantized tensor of shape [B, M, N/2]
            scale_factors: Scale factors of shape [B, M*N/16]
            dtype: Target dtype for output

        Returns:
            Dequantized tensor of shape [B, M, N]
        """
        b, m, n_half = quant_tensor.shape
        n = n_half * 2

        # More efficient unpacking using bit operations
        fp4_vals = torch.empty(b, m, n, dtype=torch.uint8, device=quant_tensor.device)
        fp4_vals[..., 0::2] = quant_tensor & 0x0F
        fp4_vals[..., 1::2] = (quant_tensor >> 4) & 0x0F

        # Extract sign and magnitude
        sign_mask = (fp4_vals & 0x08) != 0
        magnitude_idx = fp4_vals & 0x07

        # Convert to float values
        float_vals = E2M1_VALUES[magnitude_idx.long()]
        float_vals = torch.where(sign_mask, -float_vals, float_vals)

        # Reshape for block-wise scaling
        reshaped = float_vals.view(b, m * n // 16, 16)

        # Apply scale factors
        scale_exp = scale_factors.float() - 127
        scaled = reshaped * torch.exp2(scale_exp.unsqueeze(-1))

        return scaled.view(b, m, n).to(dtype)
```
**EN:** This block defines `BlockFP4KVQuantizeUtil.batched_dequantize()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `BlockFP4KVQuantizeUtil.batched_dequantize()`，用于为后续计算恢复低精度数据。

### Lines 152-160: class NVFP4KVQuantizeUtil: definition
```python
class NVFP4KVQuantizeUtil:
    """Utility class for NVFP4 quantization and dequantization with two-level scaling
    (global FP32 + block FP8 E4M3).

    Quantize formula:  x_fp4 * block_scale * global_scale = x_bf16
    - Quantize: ``nvfp4_kv_quantize`` (SM100+), fallback ``fp4_quantize`` (SM90)
    - Dequantize: ``nvfp4_kv_dequantize`` (SM100+)
    """
```
**EN:** This block declares `NVFP4KVQuantizeUtil`, a supporting class for the quantization stack. It organizes behaviors such as quantize, dequantize.
**CN:** 该代码块声明 `NVFP4KVQuantizeUtil`，它是量化栈中的支撑类，组织了 quantize, dequantize 等行为。

### Lines 161-218: NVFP4KVQuantizeUtil.quantize()
```python
    @staticmethod
    def quantize(
        tensor: torch.Tensor, global_scale: torch.Tensor
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        """Quantize BF16/FP16 tensor to NVFP4 format.

        Requires SM90+.  Uses ``nvfp4_kv_quantize`` on SM100+ (native PTX),
        falls back to ``fp4_quantize`` on SM90.

        Args:
            tensor: Input tensor of shape [B, M, N]
            global_scale: Global scale factor (float32 scalar or 1-element tensor)

        Returns:
            (fp4_data, block_scales, global_scale):
                fp4_data: shape [B, M, N/2], dtype uint8
                block_scales: shape [B, M, N/16], dtype float8_e4m3fn
                global_scale: passthrough
        """
        from sglang.srt.utils import is_sm90_supported, is_sm100_supported

        assert is_sm90_supported(), "NVFP4 KV cache quantize requires SM90+ GPU"

        b, m, n = tensor.shape
        tensor_2d = tensor.reshape(b * m, n)

        if isinstance(global_scale, (int, float)):
            global_scale = torch.tensor(
                [global_scale], dtype=torch.float32, device=tensor.device
            )
        elif global_scale.dim() == 0:
            global_scale = global_scale.unsqueeze(0)

        if is_sm100_supported():
            from flashinfer import nvfp4_kv_quantize

            # nvfp4_kv_quantize takes global_scale directly (not inverted)
            fp4_2d, scales_2d = nvfp4_kv_quantize(tensor_2d, global_scale)
        else:
            # SM90: fp4_quantize takes inverted global_scale
            from flashinfer import fp4_quantize

            global_scale_inv = 1.0 / global_scale
            fp4_2d, scales_2d = fp4_quantize(
                tensor_2d,
                global_scale_inv,
                sf_vec_size=16,
                sf_use_ue8m0=False,
                is_sf_swizzled_layout=False,
                is_sf_8x4_layout=False,
                enable_pdl=None,
            )

        fp4_data = fp4_2d.view(b, m, fp4_2d.shape[-1])
        block_scales = scales_2d.view(b, m, scales_2d.shape[-1]).view(
            torch.float8_e4m3fn
        )
        return fp4_data, block_scales, global_scale
```
**EN:** This block defines `NVFP4KVQuantizeUtil.quantize()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `NVFP4KVQuantizeUtil.quantize()`，用于处理张量或权重量化逻辑。

### Lines 220-273: NVFP4KVQuantizeUtil.dequantize()
```python
    @staticmethod
    def dequantize(
        quant_tensor: torch.Tensor,
        block_scales: torch.Tensor,
        global_scale: torch.Tensor,
        dtype: torch.dtype = torch.bfloat16,
    ) -> torch.Tensor:
        """Dequantize NVFP4 tensor to BF16/FP16.

        Uses ``nvfp4_kv_dequantize`` on SM100+, falls back to pure PyTorch
        E2M1 LUT on SM90.

        Args:
            quant_tensor: Packed FP4 data of shape [B, M, N/2] (uint8)
            block_scales: Per-block FP8 E4M3 scales of shape [B, M, N/16]
            global_scale: Global scale factor (float32)
            dtype: Output dtype (bfloat16 or float16)

        Returns:
            Dequantized tensor of shape [B, M, N]
        """
        from sglang.srt.utils import is_sm100_supported

        b, m, n_half = quant_tensor.shape

        if isinstance(global_scale, (int, float)):
            global_scale = torch.tensor(
                [global_scale], dtype=torch.float32, device=quant_tensor.device
            )
        elif global_scale.dim() == 0:
            global_scale = global_scale.unsqueeze(0)

        if is_sm100_supported():
            from flashinfer import nvfp4_kv_dequantize

            quant_2d = quant_tensor.view(torch.uint8).reshape(b * m, n_half)
            scales_2d = block_scales.view(torch.uint8).reshape(b * m, -1)
            output_2d = nvfp4_kv_dequantize(
                quant_2d, scales_2d, global_scale, output_dtype=dtype
            )
            return output_2d.reshape(b, m, -1)
        else:
            # Pure PyTorch fallback for SM90
            n = n_half * 2
            fp4_vals = torch.empty(
                b, m, n, dtype=torch.uint8, device=quant_tensor.device
            )
            fp4_vals[..., 0::2] = quant_tensor & 0x0F
            fp4_vals[..., 1::2] = (quant_tensor >> 4) & 0x0F
            float_vals = E2M1_VALUES[fp4_vals.long()]
            reshaped = float_vals.view(b, m * n // 16, 16)
            block_scales_float = block_scales.float().unsqueeze(-1)
            scaled = reshaped * block_scales_float
            return (scaled.view(b, m, n) * global_scale).to(dtype)
```
**EN:** This block defines `NVFP4KVQuantizeUtil.dequantize()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `NVFP4KVQuantizeUtil.dequantize()`，用于为后续计算恢复低精度数据。

## Key Concepts / 关键概念
- `FP4KVCacheRecipe`: A supporting class that structures file-level quantization behavior. / `FP4KVCacheRecipe` 是一个支撑类，用于组织该文件中的量化行为。
- `E2M1_MAX`: A module-level constant or registry. / `E2M1_MAX`：模块级常量或注册表。
- `MAX_BLOCK_SCALE_FP8`: A module-level constant or registry. / `MAX_BLOCK_SCALE_FP8`：模块级常量或注册表。
- `E2M1_VALUES`: A module-level constant or registry. / `E2M1_VALUES`：模块级常量或注册表。
- `E2M1_BOUNDS`: A module-level constant or registry. / `E2M1_BOUNDS`：模块级常量或注册表。
- `BlockFP4KVQuantizeUtil`: A supporting class that structures file-level quantization behavior. / `BlockFP4KVQuantizeUtil` 是一个支撑类，用于组织该文件中的量化行为。
- `NVFP4KVQuantizeUtil`: A supporting class that structures file-level quantization behavior. / `NVFP4KVQuantizeUtil` 是一个支撑类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `enum`, `flashinfer`, `torch`
- **Internal / 内部**: `sglang.srt.utils`
