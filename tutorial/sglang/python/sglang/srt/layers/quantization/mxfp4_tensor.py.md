# mxfp4_tensor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/mxfp4_tensor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for mxfp4 tensor quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 MXFP4 张量 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module imports and setup
```python
# SPDX-FileCopyrightText: Copyright (c) 2024 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: Apache-2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

from typing import Optional

import torch
```
**EN:** This block imports torch, typing and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 torch, typing 等依赖，并为当前量化实现准备模块命名空间。

### Lines 22-22: class MXFP4QuantizeUtil: definition
```python
class MXFP4QuantizeUtil:
```
**EN:** This block declares `MXFP4QuantizeUtil`, a supporting class for the quantization stack. It organizes behaviors such as quantize, dequantize.
**CN:** 该代码块声明 `MXFP4QuantizeUtil`，它是量化栈中的支撑类，组织了 quantize, dequantize 等行为。

### Lines 23-23: MXFP4QuantizeUtil member: initialize E2M1_max
```python
    E2M1_max = 6.0
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as E2M1_max.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 E2M1_max。

### Lines 25-25: MXFP4QuantizeUtil member: initialize E2M1_values
```python
    E2M1_values = [0, 0.5, 1, 1.5, 2, 3, 4, 6]
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as E2M1_values.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 E2M1_values。

### Lines 26-26: MXFP4QuantizeUtil member: initialize E2M1_bounds
```python
    E2M1_bounds = torch.tensor([0.25, 0.75, 1.25, 1.75, 2.5, 3.5, 5])
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as E2M1_bounds.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 E2M1_bounds。

### Lines 28-74: MXFP4QuantizeUtil.quantize()
```python
    @classmethod
    def quantize(cls, input: torch.Tensor, block_size: Optional[int]) -> tuple:
        """Converting a tensor to a quantized format based on MXFP4 quantization. Only E4M3 is supported.
        Args:
            input (torch.Tensor): The input tensor to be quantized.
            block_sizes (dict | None): The block sizes for quantization.
        """

        def cast_fp4(x):
            sign = torch.sign(x)
            sign_bit = (2 - sign) // 2
            ord_ = torch.sum(
                (x.abs().unsqueeze(-1) - cls.E2M1_bounds.to(x.device)) > 0, dim=-1
            )
            fp4_val = (sign_bit * 0b1000 + ord_).to(torch.uint8)
            return fp4_val

        def fuse_uint4_to_uint8(x):
            # If the last dimension is odd, pad with zeros
            # If this behavior is not desired, please modify the code accordingly
            left_side = x[..., 0::2]  # Even indices (0, 2, 4...)
            right_side = x[..., 1::2]  # Odd indices (1, 3, 5...)
            new_data = (
                right_side.clone() << 4
            )  # Put odd indices (higher addresses) in high bits
            new_data[
                ..., : left_side.shape[-1]
            ] += left_side  # Put even indices in low bits
            return new_data

        if block_size is None:
            block_size = 32

        original_shape = input.shape
        original_dtype = input.dtype
        input = input.view(-1, block_size)
        # get scales
        input_amax = input.abs().max(dim=-1, keepdim=True).values
        descale = input_amax / cls.E2M1_max
        min_value = torch.tensor(-127.0, device=descale.device)
        e8m0_scale = torch.ceil(torch.maximum(torch.log2(descale), min_value))

        input = (input / torch.exp2(e8m0_scale)).view(original_shape)
        input_q = cast_fp4(input)
        input_q = fuse_uint4_to_uint8(input_q)
        e8m0_scale = (e8m0_scale + 127).to(torch.uint8)
        return cls(original_shape, original_dtype, input_q), e8m0_scale
```
**EN:** This block defines `MXFP4QuantizeUtil.quantize()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `MXFP4QuantizeUtil.quantize()`，用于处理张量或权重量化逻辑。

### Lines 76-135: MXFP4QuantizeUtil.dequantize()
```python
    @classmethod
    def dequantize(cls, quantized_data, dtype: torch.dtype, scale, block_sizes):
        """Dequantze MXFP4 packed tensor to a target dtype."""

        def unfuse_uint8_to_uint4(x):
            """Unfuse uint8 values back to uint4 values.
            This is the inverse operation of fuse_uint4_to_uint8.
            """
            # Extract the lower 4 bits (even indices)
            left_side = x & 0x0F

            # Extract the upper 4 bits (odd indices)
            right_side = (x >> 4) & 0x0F

            # Create a new tensor with alternating values
            shape = list(x.shape)
            shape[-1] = shape[-1] * 2
            result = torch.zeros(shape, dtype=torch.uint8, device=x.device)

            # Fill in the values - even indices get low bits, odd indices get high bits
            result[..., 0::2] = left_side  # Even indices from low bits
            result[..., 1::2] = right_side  # Odd indices from high bits

            return result

        e8m0_scale = scale
        block_size = block_sizes[-1]

        # Unfuse the uint8 values back to uint4
        x_unfused = unfuse_uint8_to_uint4(quantized_data)
        # Extract sign and magnitude
        sign = 1 - 2 * ((x_unfused & 0b1000) >> 3).to(
            torch.float32
        )  # Extract sign bit and convert to +1/-1
        magnitude = x_unfused & 0b0111  # Extract magnitude bits
        magnitude = magnitude.to(torch.long)

        # Create a tensor with the E2M1 values
        values = torch.tensor(cls.E2M1_values, device=quantized_data.device)

        # Use gather to index the values tensor properly
        # We need to reshape magnitude to match the dimensions we want to gather along
        original_shape = magnitude.shape
        x_float = values[magnitude.reshape(-1)].reshape(original_shape)

        # Apply sign and scale
        x_float = sign.float() * x_float

        # Reshape to apply block-wise scaling
        x_float = x_float.reshape(-1, block_size)

        # Apply the E8M0 scale
        scale_factor = torch.exp2(e8m0_scale.float() - 127)
        scale_factor = scale_factor.reshape(-1, 1)  # Reshape for proper broadcasting

        # Apply scaling and reshape back to original shape
        x_float = x_float * scale_factor

        # Reshape back to the original shape
        return x_float.reshape(original_shape).to(dtype)
```
**EN:** This block defines `MXFP4QuantizeUtil.dequantize()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `MXFP4QuantizeUtil.dequantize()`，用于为后续计算恢复低精度数据。

## Key Concepts / 关键概念
- `MXFP4QuantizeUtil`: A supporting class that structures file-level quantization behavior. / `MXFP4QuantizeUtil` 是一个支撑类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `torch`, `typing`
- **Internal / 内部**: Minimal package-local imports. / 包内直接导入较少。
