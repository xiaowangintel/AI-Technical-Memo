# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates utils behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 utils 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module imports and dependencies / 模块导入与依赖
```python
import itertools
import math

import torch
import torch.nn.functional as F
```
**EN:** This block imports the modules needed by the rest of the file, including `itertools`, `math`, `torch`, `torch.nn.functional`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `itertools`, `math`, `torch`, `torch.nn.functional`。

### Lines 7-16: module-level constants and configuration / 模块级常量与配置
```python
precision = {
    torch.bfloat16: 1e-2,
    torch.float16: 1e-3,
    torch.float32: 1e-5,
}


BLOCK_N, BLOCK_K = 64, 128
factor_for_scale = 1e-3
fp8_max, fp8_min = 400, -400
```
**EN:** This block defines shared names such as `precision`, `BLOCK_N`, `BLOCK_K`, `factor_for_scale`, `fp8_max`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `precision`, `BLOCK_N`, `BLOCK_K`, `factor_for_scale`, `fp8_max` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 19-29: function parametrize / 函数 parametrize
```python
def parametrize(**params):
    def decorator(func):
        def wrapper(self):
            for combo in itertools.product(*params.values()):
                kwargs = dict(zip(params.keys(), combo))
                with self.subTest(**kwargs):
                    func(self, **kwargs)

        return wrapper

    return decorator
```
**EN:** This block implements `parametrize` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `parametrize`，承担模块行为中的一个聚焦逻辑片段。

### Lines 32-34: function Silu And Mul / 函数 Silu And Mul
```python
def SiluAndMul(x: torch.Tensor) -> torch.Tensor:
    d = x.shape[-1] // 2
    return F.silu(x[..., :d]) * x[..., d:]
```
**EN:** This block implements `SiluAndMul` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `SiluAndMul`，承担模块行为中的一个聚焦逻辑片段。

### Lines 37-39: function Gelu And Mul / 函数 Gelu And Mul
```python
def GeluAndMul(x: torch.Tensor, approximate="tanh") -> torch.Tensor:
    d = x.shape[-1] // 2
    return F.gelu(x[..., :d], approximate=approximate) * x[..., d:]
```
**EN:** This block implements `GeluAndMul` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `GeluAndMul`，承担模块行为中的一个聚焦逻辑片段。

### Lines 42-50: function per token quant int8 / 函数 per token quant int8
```python
def per_token_quant_int8(x):
    x = x.float()
    absmax = x.abs().max(dim=-1).values
    absmax = absmax.clamp_min(1e-10).unsqueeze(-1)
    scale_x = absmax / 127
    x_q = x.mul(127 / absmax)
    x_q = torch.round(x_q).to(torch.int8)

    return x_q, scale_x
```
**EN:** This block implements `per_token_quant_int8` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `per_token_quant_int8`，承担模块行为中的一个聚焦逻辑片段。

### Lines 53-101: function convert weight / 函数 convert weight
```python
def convert_weight(weight, scale_block_size, A_dtype):
    N, K = weight.size()
    fp8_max = 448.0
    scale_block_size_N, scale_block_size_K = scale_block_size  # (128, 128)

    pad_N = (scale_block_size_N - (N % scale_block_size_N)) % scale_block_size_N
    pad_K = (scale_block_size_K - (K % scale_block_size_K)) % scale_block_size_K

    if pad_N > 0 or pad_K > 0:
        weight = torch.nn.functional.pad(weight, (0, pad_K, 0, pad_N))

    weight_blocks = weight.view(
        math.ceil(N / scale_block_size_N),
        scale_block_size_N,
        math.ceil(K / scale_block_size_K),
        scale_block_size_K,
    )  # (8, 128, 8, 128)
    weight_blocks = weight_blocks.permute(0, 2, 1, 3).contiguous()  # (8, 8, 128, 128)

    # Step 2: compute per-block max abs values → scale
    abs_max = weight_blocks.abs().amax(dim=(-2, -1), keepdim=True)  # (8, 8, 1, 1)
    scales = abs_max / fp8_max
    scales = torch.where(
        scales == 0, torch.ones_like(scales), scales
    )  # avoid division by zero

    q_fp8 = (weight_blocks / scales).to(torch.float8_e4m3fn)
    q_fp8_reshape = q_fp8.permute(0, 2, 1, 3).contiguous()

    if pad_N > 0 or pad_K > 0:
        q_fp8_reshape = q_fp8_reshape.view(N + pad_N, K + pad_K)
        q_fp8_reshape = q_fp8_reshape[:N, :K].contiguous()
    else:
        q_fp8_reshape = q_fp8_reshape.view(N, K)

    dq_weight = q_fp8.float() * scales
    dq_weight = dq_weight.permute(0, 2, 1, 3).contiguous()  # (8, 128, 8, 128)

    if pad_N > 0 or pad_K > 0:
        w_dq = dq_weight.view(N + pad_N, K + pad_K).to(A_dtype)
        w_dq = w_dq[:N, :K].contiguous()
    else:
        w_dq = dq_weight.view(N, K).to(A_dtype)

    scales = scales.view(
        math.ceil(N / scale_block_size_N), math.ceil(K / scale_block_size_K)
    )

    return q_fp8_reshape, scales, w_dq
```
**EN:** This block implements `convert_weight` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `convert_weight`，承担模块行为中的一个聚焦逻辑片段。

### Lines 104-126: function native w8a8 per token matmul / 函数 native w8a8 per token matmul
```python
def native_w8a8_per_token_matmul(A, B, As, Bs, bias, output_dtype=torch.bfloat16):
    """Matrix multiplication function that supports per-token input quantization and per-column weight quantization"""
    A = A.to(torch.float32)
    B = B.to(torch.float32)

    assert A.shape[-1] == B.shape[-1], "Dimension mismatch"
    assert B.ndim == 2 and B.is_contiguous(), "B must be a 2D contiguous tensor"

    # Reshape input
    M = A.numel() // A.shape[-1]
    B = B.t()  # Transpose weight matrix
    N, K = B.shape
    origin_C_shape = A.shape[:-1] + (K,)
    A = A.reshape(M, N)

    # As is per-token [M, 1], Bs is per-column [1, K]
    C = torch.matmul(A, B)  # [M, K]
    C = As * C * Bs.view(1, -1)  # Broadcast per-column scale

    if bias is not None:
        C.add_(bias.view(1, -1))

    return C.reshape(origin_C_shape).to(output_dtype)
```
**EN:** Matrix multiplication function that supports per-token input quantization and per-column weight quantization This block implements `native_w8a8_per_token_matmul` and captures one focused piece of the module's behavior.
**CN:** Matrix multiplication function that supports per-token input quantization and per-column weight quantization 该代码块实现 `native_w8a8_per_token_matmul`，承担模块行为中的一个聚焦逻辑片段。

### Lines 129-142: function torch naive moe / 函数 torch naive moe
```python
def torch_naive_moe(a, w1, w2, b, routed_scaling_factor, output_dtype=torch.bfloat16):

    a = a.to(torch.float32)
    w1 = w1.to(torch.float32)
    w2 = w2.to(torch.float32)
    b = b.to(torch.float32) if b is not None else None

    ic1 = torch.matmul(a, w1.transpose(0, 1))
    ic2 = SiluAndMul(ic1)
    ic3 = torch.matmul(ic2, w2.transpose(0, 1))

    out = ic3 if b is None else ic3 + b * routed_scaling_factor

    return out.to(output_dtype)
```
**EN:** This block implements `torch_naive_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `torch_naive_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 145-167: function torch w8a8 per column moe / 函数 torch w8a8 per column moe
```python
def torch_w8a8_per_column_moe(
    a, w1_q, w2_q, w1_s, w2_s, b, routed_scaling_factor, output_dtype=torch.bfloat16
):

    a = a.to(torch.float32)
    b = b.to(torch.float32) if b is not None else None

    # Perform per-token quantization
    a_q, a_s = per_token_quant_int8(a)

    ic1 = native_w8a8_per_token_matmul(
        a_q, w1_q, a_s, w1_s, bias=None, output_dtype=torch.float32
    )
    ic2 = SiluAndMul(ic1)

    a1_q, a1_s = per_token_quant_int8(ic2)
    ic3 = native_w8a8_per_token_matmul(
        a1_q, w2_q, a1_s, w2_s, bias=None, output_dtype=torch.float32
    )

    out = ic3 if b is None else ic3 + b * routed_scaling_factor

    return out.to(output_dtype)
```
**EN:** This block implements `torch_w8a8_per_column_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `torch_w8a8_per_column_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 170-198: function scaled weight / 函数 scaled weight
```python
def scaled_weight(weight, scales):
    E, N, K = weight.shape
    pad_N = (BLOCK_N - (N % BLOCK_N)) % BLOCK_N
    pad_K = (BLOCK_K - (K % BLOCK_K)) % BLOCK_K

    if pad_N > 0 or pad_K > 0:
        weight = torch.nn.functional.pad(weight, (0, pad_K, 0, pad_N))

    weight_block = (
        weight.view(E, math.ceil(N / BLOCK_N), BLOCK_N, math.ceil(K / BLOCK_K), BLOCK_K)
        .permute(0, 1, 3, 2, 4)
        .float()
        .contiguous()
    )

    weight_scaled = (
        (
            weight_block
            * scales.view(E, math.ceil(N / BLOCK_N), math.ceil(K / BLOCK_K), 1, 1)
        )
        .permute(0, 1, 3, 2, 4)
        .contiguous()
    )
    if pad_N > 0 or pad_K > 0:
        weight_scaled = weight_scaled.view(E, N + pad_N, K + pad_K)
        weight_scaled = weight_scaled[..., :N, :K].contiguous()
    else:
        weight_scaled = weight_scaled.view(E, N, K)
    return weight_scaled
```
**EN:** This block implements `scaled_weight` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `scaled_weight`，承担模块行为中的一个聚焦逻辑片段。

### Lines 201-221: function torch naive fused moe / 函数 torch naive fused moe
```python
def torch_naive_fused_moe(a, w1, w2, score, topk, renormalize):
    B, D = a.shape
    a = a.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
    out = torch.zeros(B * topk, w2.shape[1], dtype=a.dtype, device=a.device)
    score = torch.softmax(score, dim=-1, dtype=torch.float32)
    topk_weight, topk_ids = torch.topk(score, topk)

    if renormalize:
        topk_weight = topk_weight / topk_weight.sum(dim=-1, keepdim=True)

    topk_weight = topk_weight.view(-1)
    topk_ids = topk_ids.view(-1)
    for i in range(w1.shape[0]):
        mask = topk_ids == i
        if mask.sum():
            out[mask] = SiluAndMul(a[mask] @ w1[i].transpose(0, 1)) @ w2[i].transpose(
                0, 1
            )
    return (
        out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype)
    ).sum(dim=1)
```
**EN:** This block implements `torch_naive_fused_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `torch_naive_fused_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 224-271: function torch w8a8 per column fused moe / 函数 torch w8a8 per column fused moe
```python
def torch_w8a8_per_column_fused_moe(a, w1, w2, w1_s, w2_s, topk_weight, topk_ids, topk):
    """This function performs fused moe with per-column int8 quantization using native torch."""

    B, D = a.shape
    # Perform per-token quantization
    a_q, a_s = per_token_quant_int8(a)
    # Repeat tokens to match topk
    a_q = a_q.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
    # Also repeat the scale
    a_s = a_s.view(B, -1, 1).repeat(1, topk, 1).reshape(-1, 1)  # [B*topk, 1]

    out = torch.zeros(B * topk, w2.shape[1], dtype=torch.float32, device=a.device)

    # Calculate routing
    topk_weight = topk_weight.view(-1)
    topk_ids = topk_ids.view(-1)
    # Process each expert
    for i in range(w1.shape[0]):
        mask = topk_ids == i
        if mask.sum():
            # First MLP layer: note that a_s is now per-token
            inter_out = native_w8a8_per_token_matmul(
                a_q[mask],
                w1[i],
                a_s[mask],
                w1_s[i],
                bias=None,
                output_dtype=torch.float32,
            )
            # Activation function
            act_out = SiluAndMul(inter_out)
            # Quantize activation output with per-token
            act_out_q, act_out_s = per_token_quant_int8(act_out)
            # Second MLP layer
            out[mask] = native_w8a8_per_token_matmul(
                act_out_q,
                w2[i],
                act_out_s,
                w2_s[i],
                bias=None,
                output_dtype=torch.float32,
            )
    # Apply routing weights and sum
    return (
        (out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype))
        .sum(dim=1)
        .to(a.dtype)
    )
```
**EN:** This function performs fused moe with per-column int8 quantization using native torch. This block implements `torch_w8a8_per_column_fused_moe` and captures one focused piece of the module's behavior.
**CN:** This function performs fused moe with per-column int8 quantization using native torch. 该代码块实现 `torch_w8a8_per_column_fused_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 274-294: function native fp8 fused moe / 函数 native fp8 fused moe
```python
def native_fp8_fused_moe(a, w1, w2, topk_weight, topk_ids, topk):
    B, D = a.shape
    a = a.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D).float()
    out = torch.zeros(B * topk, w2.shape[1], dtype=torch.float32, device=a.device)

    # Calculate routing
    topk_weight = topk_weight.view(-1)
    topk_ids = topk_ids.view(-1)

    for i in range(w1.shape[0]):
        mask = topk_ids == i
        if mask.sum():
            ic0 = torch.matmul(a[mask], w1[i].transpose(0, 1))
            ic1 = SiluAndMul(ic0)
            out[mask] = torch.matmul(ic1, w2[i].transpose(0, 1))

    return (
        (out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype))
        .sum(dim=1)
        .to(a.dtype)
    )
```
**EN:** This block implements `native_fp8_fused_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `native_fp8_fused_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 297-302: helper routine make non contiguous / 辅助流程 make non contiguous
```python
def make_non_contiguous(x: torch.Tensor) -> torch.Tensor:
    """
    Make a tensor non-contiguous by slicing it via last dimension.
    """
    last_dim = x.shape[-1]
    return x[..., : last_dim // 2] if x.is_contiguous() else x
```
**EN:** Make a tensor non-contiguous by slicing it via last dimension. This helper encapsulates `make_non_contiguous` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Make a tensor non-contiguous by slicing it via last dimension. 该辅助函数封装了 `make_non_contiguous`，以便周围测试复用准备、执行或校验逻辑。

### Lines 305-329: function awq reverse reorder int tensor / 函数 awq reverse reorder int tensor
```python
def awq_reverse_reorder_int_tensor(int_tensor, bits: int):
    assert bits == 4

    int_tensor = int_tensor.T.contiguous()
    compress_ratio = 32 // bits
    assert int_tensor.shape[-1] % compress_ratio == 0

    order_map = [0, 2, 4, 6, 1, 3, 5, 7]
    order_tensor = torch.tensor(
        order_map, dtype=torch.int32, device=int_tensor.device
    ).reshape(1, -1)
    order_tensor = order_tensor.repeat(int_tensor.shape[1] // compress_ratio, 1)
    order_tensor = order_tensor + torch.arange(
        0,
        int_tensor.shape[1],
        compress_ratio,
        dtype=torch.int32,
        device=int_tensor.device,
    ).reshape(-1, 1)
    order_tensor = order_tensor.reshape(-1)

    reverse_order_tensor = torch.arange(order_tensor.shape[0])[order_tensor]
    reverse_order_tensor = reverse_order_tensor[order_tensor]
    int_tensor = int_tensor[:, reverse_order_tensor]
    return int_tensor
```
**EN:** This block implements `awq_reverse_reorder_int_tensor` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `awq_reverse_reorder_int_tensor`，承担模块行为中的一个聚焦逻辑片段。

### Lines 332-404: function unpack and dequant awq / 函数 unpack and dequant awq
```python
def unpack_and_dequant_awq(
    awq_qweight: torch.Tensor,
    awq_qzeros: torch.Tensor,
    awq_scales: torch.Tensor,
    bits: int,
    group_size: int,
):
    """
    Args:
        awq_qweight (`torch.LongTensor`):
            Expected shape: (in_features, out_features // (32 // bits))
        awq_qzeros (`torch.LongTensor`):
            Expected shape: (in_features // group_size, out_features // (32 // bits))
        awq_scales (`torch.LongTensor`):
            Expected shape: (in_features // group_size, out_features)

    Returns:
        fp16_weight (`torch.LongTensor`):
            With shape (in_features, out_features).
        zeros (`torch.LongTensor`):
            With shape (in_features // group_size, out_features).
    """
    assert bits == 4

    qzeros = awq_qzeros
    qweight = awq_qweight
    qweight = qweight.T.contiguous()

    scales = awq_scales
    scales = scales.reshape(-1, 1, scales.shape[-1])

    infeatures = awq_qweight.shape[0]

    wf = torch.tensor(
        list(range(0, 32, bits)), dtype=torch.int32, device=qzeros.device
    ).unsqueeze(0)
    zeros = torch.bitwise_right_shift(torch.unsqueeze(qzeros, 2), wf.unsqueeze(0)).to(
        torch.int16 if bits == 8 else torch.int8
    )

    torch.bitwise_and(zeros, (2**bits) - 1, out=zeros)

    zeros = zeros.reshape(-1, 1, zeros.shape[1] * zeros.shape[2])

    weight = torch.bitwise_right_shift(
        torch.unsqueeze(qweight, 1), wf.unsqueeze(-1)
    ).to(torch.int16 if bits == 8 else torch.int8)
    torch.bitwise_and(weight, (2**bits) - 1, out=weight)
    weight = weight.reshape(-1, group_size, weight.shape[2])

    weight = weight.view(-1, weight.shape[-1])
    zeros = zeros.view(-1, zeros.shape[-1])

    zeros = zeros.T.contiguous()
    zeros = awq_reverse_reorder_int_tensor(zeros, bits)
    weight = awq_reverse_reorder_int_tensor(weight, bits)

    # Dequantize weights.
    scales = awq_scales
    zeros = zeros.contiguous()
    scale_zeros = zeros * scales

    g_idx = torch.tensor(
        [i // group_size for i in range(infeatures)], dtype=torch.int32
    )
    scale_mat = scales[g_idx]
    scale_zeros_mat = scale_zeros[g_idx].to(torch.bfloat16)

    qdq_weight_T = weight * scale_mat - scale_zeros_mat.to(torch.bfloat16)

    fp16_weight = qdq_weight_T.T

    return fp16_weight, zeros
```
**EN:** Args: awq_qweight (`torch.LongTensor`): Expected shape: (in_features, out_features // (32 // bits)) awq_qzeros (`torch.LongTensor`): Expected shape: (in_features // group_size, out_features // (32 // bits)) awq_scales (`torch.LongTensor`): Expected shape: (in_features // group_size, out_features) Returns: fp16_weight (`torch.LongTensor`): With shape (in_features, out_features). This block implements `unpack_and_dequant_awq` and captures one focused piece of the module's behavior.
**CN:** Args: awq_qweight (`torch.LongTensor`): Expected shape: (in_features, out_features // (32 // bits)) awq_qzeros (`torch.LongTensor`): Expected shape: (in_features // group_size, out_features // (32 // bits)) awq_scales (`torch.LongTensor`): Expected shape: (in_features // group_size, out_features) Returns: fp16_weight (`torch.LongTensor`): With shape (in_features, out_features). 该代码块实现 `unpack_and_dequant_awq`，承担模块行为中的一个聚焦逻辑片段。

### Lines 407-430: function unpack 4bit to 32bit signed / 函数 unpack 4bit to 32bit signed
```python
def unpack_4bit_to_32bit_signed(qweight, qzeros):
    # Unpack 4-bit values and interpret them as signed integers
    unpacked_weights = torch.zeros(
        (qweight.shape[0] * 8, qweight.shape[1]),
        dtype=torch.int8,
        device=qweight.device,
        requires_grad=False,
    )
    unpacked_zeros = torch.zeros(
        (qzeros.shape[0], qzeros.shape[1] * 8),
        dtype=torch.int8,
        device=qzeros.device,
        requires_grad=False,
    )

    for row in range(unpacked_weights.shape[0]):
        i = row % 8
        unpacked_weights[row, :] = (qweight[row // 8, :] >> (4 * i)) & 0xF

    for col in range(unpacked_zeros.shape[1]):
        i = col % 8
        unpacked_zeros[:, col] = (qzeros[:, col // 8] >> (4 * i)) & 0xF

    return unpacked_weights, unpacked_zeros + 1
```
**EN:** This block implements `unpack_4bit_to_32bit_signed` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `unpack_4bit_to_32bit_signed`，承担模块行为中的一个聚焦逻辑片段。

### Lines 433-440: function unpack and dequant gptq / 函数 unpack and dequant gptq
```python
def unpack_and_dequant_gptq(qweight, qzeros, scales):
    unpacked_qweight, unpacked_qzeros = unpack_4bit_to_32bit_signed(qweight, qzeros)
    group_size = unpacked_qweight.shape[0] // scales.shape[0]
    scales = scales.repeat_interleave(group_size, dim=0)
    unpacked_qzeros = unpacked_qzeros.repeat_interleave(group_size, dim=0)
    unpacked_qweight = (unpacked_qweight - unpacked_qzeros) * scales

    return unpacked_qweight.T
```
**EN:** This block implements `unpack_and_dequant_gptq` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `unpack_and_dequant_gptq`，承担模块行为中的一个聚焦逻辑片段。

## Key Concepts / 关键概念
- `parametrize`: This block implements `parametrize` and captures one focused piece of the module's behavior. / 该代码块实现 `parametrize`，承担模块行为中的一个聚焦逻辑片段。
- `SiluAndMul`: This block implements `SiluAndMul` and captures one focused piece of the module's behavior. / 该代码块实现 `SiluAndMul`，承担模块行为中的一个聚焦逻辑片段。
- `GeluAndMul`: This block implements `GeluAndMul` and captures one focused piece of the module's behavior. / 该代码块实现 `GeluAndMul`，承担模块行为中的一个聚焦逻辑片段。
- `per_token_quant_int8`: This block implements `per_token_quant_int8` and captures one focused piece of the module's behavior. / 该代码块实现 `per_token_quant_int8`，承担模块行为中的一个聚焦逻辑片段。
- `convert_weight`: This block implements `convert_weight` and captures one focused piece of the module's behavior. / 该代码块实现 `convert_weight`，承担模块行为中的一个聚焦逻辑片段。
- `native_w8a8_per_token_matmul`: Matrix multiplication function that supports per-token input quantization and per-column weight quantization / 该代码块实现 `native_w8a8_per_token_matmul`，承担模块行为中的一个聚焦逻辑片段。
- `torch_naive_moe`: This block implements `torch_naive_moe` and captures one focused piece of the module's behavior. / 该代码块实现 `torch_naive_moe`，承担模块行为中的一个聚焦逻辑片段。
- `torch_w8a8_per_column_moe`: This block implements `torch_w8a8_per_column_moe` and captures one focused piece of the module's behavior. / 该代码块实现 `torch_w8a8_per_column_moe`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `math`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn.functional`

- **Total lines / 总行数**: 440
