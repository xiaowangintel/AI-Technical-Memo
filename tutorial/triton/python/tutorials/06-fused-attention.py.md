# 06-fused-attention.py — Code Analysis / 代码分析

## Source / 来源

- **EN:** Source file: `/root/xw/triton/python/tutorials/06-fused-attention.py`. This tutorial implements Triton Flash Attention v2, including forward, backward, tests, and benchmarking code.
- **CN:** 源文件：`/root/xw/triton/python/tutorials/06-fused-attention.py`。该教程实现了 Triton 版本的 Flash Attention v2，包括前向、反向、测试与基准代码。

## Line-by-Line Analysis / 逐行分析

### Block 1 (Lines 1-44) / 代码块 1（第 1-44 行）

```python
"""
Fused Attention
===============

This is a Triton implementation of the Flash Attention v2 algorithm from Tri Dao (https://tridao.me/publications/flash2/flash2.pdf)

Credits: OpenAI kernel team

Extra Credits:

* Original flash attention paper (https://arxiv.org/abs/2205.14135)
* Rabe and Staats (https://arxiv.org/pdf/2112.05682v2.pdf)

"""

import pytest
import torch
import os

import triton
import triton.language as tl
from triton.tools.tensor_descriptor import TensorDescriptor

DEVICE = triton.runtime.driver.active.get_active_torch_device()


def is_hip():
    return triton.runtime.driver.active.get_current_target().backend == "hip"


def is_cuda():
    return triton.runtime.driver.active.get_current_target().backend == "cuda"


def supports_host_descriptor():
    return is_cuda() and torch.cuda.get_device_capability()[0] >= 9


def is_blackwell():
    return is_cuda() and torch.cuda.get_device_capability()[0] == 10


def is_hopper():
    return is_cuda() and torch.cuda.get_device_capability()[0] == 9
```
**EN:** The file starts with a tutorial header, imports, and backend capability probes. These helpers decide whether the run is on HIP or CUDA, and whether Hopper/Blackwell-only features such as host tensor descriptors should be used. Those predicates later gate autotune choices, FP8 layout handling, and warp-specialized launch behavior.
**CN:** 文件开头给出了教程说明、导入项以及后端能力探测函数。这些辅助函数用于判断当前运行环境是 HIP 还是 CUDA，以及是否可以启用 Hopper/Blackwell 相关特性（如 host tensor descriptor）。后续的自动调优、FP8 布局分支和 warp specialization 启动策略都依赖这些判断。

### Block 2 (Lines 47-110) / 代码块 2（第 47-110 行）

```python
@triton.jit
def _attn_fwd_inner(acc, l_i, m_i, q,  #
                    desc_k, desc_v,  #
                    offset_y, dtype: tl.constexpr, start_m, qk_scale,  #
                    BLOCK_M: tl.constexpr, HEAD_DIM: tl.constexpr, BLOCK_N: tl.constexpr,  #
                    STAGE: tl.constexpr, offs_m: tl.constexpr, offs_n: tl.constexpr,  #
                    N_CTX: tl.constexpr, warp_specialize: tl.constexpr, IS_HOPPER: tl.constexpr):
    # range of values handled by this stage
    if STAGE == 1:
        lo, hi = 0, start_m * BLOCK_M
    elif STAGE == 2:
        lo, hi = start_m * BLOCK_M, (start_m + 1) * BLOCK_M
        lo = tl.multiple_of(lo, BLOCK_M)
    # causal = False
    else:
        lo, hi = 0, N_CTX
    offsetk_y = offset_y + lo
    if dtype == tl.float8e5:
        offsetv_y = offset_y * HEAD_DIM + lo
    else:
        offsetv_y = offset_y + lo
    # loop over k, v and update accumulator
    for start_n in tl.range(lo, hi, BLOCK_N, warp_specialize=warp_specialize):
        start_n = tl.multiple_of(start_n, BLOCK_N)
        # -- compute qk ----
        k = desc_k.load([offsetk_y, 0]).T
        qk = tl.dot(q, k)
        if STAGE == 2:
            mask = offs_m[:, None] >= (start_n + offs_n[None, :])
            qk = qk * qk_scale + tl.where(mask, 0, -1.0e6)
            m_ij = tl.maximum(m_i, tl.max(qk, 1))
            qk -= m_ij[:, None]
        else:
            m_ij = tl.maximum(m_i, tl.max(qk, 1) * qk_scale)
            qk = qk * qk_scale - m_ij[:, None]
        p = tl.math.exp2(qk)
        # -- compute correction factor
        alpha = tl.math.exp2(m_i - m_ij)
        l_ij = tl.sum(p, 1)
        # -- update output accumulator --
        if not IS_HOPPER and warp_specialize and BLOCK_M == 128 and HEAD_DIM == 128:
            BM: tl.constexpr = acc.shape[0]
            BN: tl.constexpr = acc.shape[1]
            acc0, acc1 = acc.reshape([BM, 2, BN // 2]).permute(0, 2, 1).split()
            acc0 = acc0 * alpha[:, None]
            acc1 = acc1 * alpha[:, None]
            acc = tl.join(acc0, acc1).permute(0, 2, 1).reshape([BM, BN])
        else:
            acc = acc * alpha[:, None]
        # prepare p and v for the dot
        if dtype == tl.float8e5:
            v = desc_v.load([0, offsetv_y]).T
        else:
            v = desc_v.load([offsetv_y, 0])
        p = p.to(dtype)
        # note that this non transposed v for FP8 is only supported on Blackwell
        acc = tl.dot(p, v, acc)
        # update m_i and l_i
        # place this at the end of the loop to reduce register pressure
        l_i = l_i * alpha + l_ij
        m_i = m_ij
        offsetk_y += BLOCK_N
        offsetv_y += BLOCK_N
    return acc, l_i, m_i
```
**EN:** _attn_fwd_inner is the Flash Attention v2 forward inner loop. It streams over K/V tiles, computes QK^T, applies the causal mask only on the diagonal stage, and maintains the online softmax state with per-row max m_i and denominator l_i. The qk scale is expressed in base-2 space so Triton can use exp2 efficiently. The accumulator update is numerically stable and includes a special register-pressure workaround for non-Hopper warp-specialized 128x128 cases.
**CN:** _attn_fwd_inner 是 Flash Attention v2 前向过程的核心内循环。它按块遍历 K/V，计算 QK^T，并只在对角块阶段施加因果 mask，同时通过逐行最大值 m_i 和归一化因子 l_i 维护在线 softmax 状态。qk 的缩放被转换到以 2 为底的指数空间，从而高效使用 exp2。累加器更新采用数值稳定形式，并对非 Hopper 平台下 128x128 的 warp-specialized 情况加入了寄存器压力优化分支。

### Block 3 (Lines 113-165) / 代码块 3（第 113-165 行）

```python
def _host_descriptor_pre_hook(nargs):
    BLOCK_M = nargs["BLOCK_M"]
    BLOCK_N = nargs["BLOCK_N"]
    HEAD_DIM = nargs["HEAD_DIM"]
    if not isinstance(nargs["desc_q"], TensorDescriptor):
        return
    nargs["desc_q"].block_shape = [BLOCK_M, HEAD_DIM]
    if nargs["FP8_OUTPUT"]:
        nargs["desc_v"].block_shape = [HEAD_DIM, BLOCK_N]
    else:
        nargs["desc_v"].block_shape = [BLOCK_N, HEAD_DIM]
    nargs["desc_k"].block_shape = [BLOCK_N, HEAD_DIM]
    nargs["desc_o"].block_shape = [BLOCK_M, HEAD_DIM]


if is_hip():
    NUM_STAGES_OPTIONS = [1]
elif supports_host_descriptor():
    NUM_STAGES_OPTIONS = [2, 3, 4]
else:
    NUM_STAGES_OPTIONS = [2, 3, 4]

configs = [
    triton.Config({'BLOCK_M': BM, 'BLOCK_N': BN}, num_stages=s, num_warps=w, pre_hook=_host_descriptor_pre_hook) \
    for BM in [64, 128]\
    for BN in [32, 64, 128]\
    for s in NUM_STAGES_OPTIONS \
    for w in [4, 8]\
]
if "PYTEST_VERSION" in os.environ:
    # Use a single config in testing for reproducibility
    configs = [
        triton.Config(dict(BLOCK_M=128, BLOCK_N=64), num_stages=2, num_warps=4, pre_hook=_host_descriptor_pre_hook),
    ]


def keep(conf):
    BLOCK_M = conf.kwargs["BLOCK_M"]
    BLOCK_N = conf.kwargs["BLOCK_N"]
    return not (is_cuda() and torch.cuda.get_device_capability()[0] == 9 and BLOCK_M * BLOCK_N < 128 * 128
                and conf.num_warps == 8)


def prune_invalid_configs(configs, named_args, **kwargs):
    N_CTX = kwargs["N_CTX"]
    STAGE = kwargs["STAGE"]

    # Filter out configs where BLOCK_M > N_CTX
    # Filter out configs where BLOCK_M < BLOCK_N when causal is True
    return [
        conf for conf in configs if conf.kwargs.get("BLOCK_M", 0) <= N_CTX and (
            conf.kwargs.get("BLOCK_M", 0) >= conf.kwargs.get("BLOCK_N", 0) or STAGE == 1)
    ]
```
**EN:** This block prepares descriptor-aware autotuning. _host_descriptor_pre_hook patches the runtime block shapes on TensorDescriptor objects so the same kernel can be launched with the tile sizes chosen by autotune. The config list sweeps BLOCK_M/BLOCK_N, pipeline stages, and warp counts, then keep() and prune_invalid_configs() remove combinations that are invalid or known to perform poorly on specific hardware, especially Hopper and causal cases.
**CN:** 这一段为支持 descriptor 的自动调优做准备。_host_descriptor_pre_hook 会把 autotune 选出的 tile 大小写回 TensorDescriptor 的 block_shape，使同一个内核能够匹配不同配置启动。随后 configs 遍历 BLOCK_M/BLOCK_N、流水级数和 warp 数；keep() 与 prune_invalid_configs() 再过滤无效或已知在特定硬件上表现较差的组合，尤其是 Hopper 和因果注意力场景。

### Block 4 (Lines 168-247) / 代码块 4（第 168-247 行）

```python
@triton.jit
def _maybe_make_tensor_desc(desc_or_ptr, shape, strides, block_shape):
    if isinstance(desc_or_ptr, tl.tensor_descriptor):
        return desc_or_ptr
    else:
        return tl.make_tensor_descriptor(desc_or_ptr, shape, strides, block_shape)


@triton.autotune(configs=list(filter(keep, configs)), key=["N_CTX", "HEAD_DIM", "FP8_OUTPUT", "warp_specialize"],
                 prune_configs_by={'early_config_prune': prune_invalid_configs})
@triton.jit
def _attn_fwd(sm_scale, M,  #
              Z, H, desc_q, desc_k, desc_v, desc_o, N_CTX,  #
              HEAD_DIM: tl.constexpr,  #
              BLOCK_M: tl.constexpr,  #
              BLOCK_N: tl.constexpr,  #
              FP8_OUTPUT: tl.constexpr,  #
              STAGE: tl.constexpr,  #
              warp_specialize: tl.constexpr,  #
              IS_HOPPER: tl.constexpr,  #
              ):
    dtype = tl.float8e5 if FP8_OUTPUT else tl.float16
    tl.static_assert(BLOCK_N <= HEAD_DIM)
    start_m = tl.program_id(0)
    off_hz = tl.program_id(1)
    off_z = off_hz // H
    off_h = off_hz % H

    y_dim = Z * H * N_CTX
    desc_q = _maybe_make_tensor_desc(desc_q, shape=[y_dim, HEAD_DIM], strides=[HEAD_DIM, 1],
                                     block_shape=[BLOCK_M, HEAD_DIM])
    if FP8_OUTPUT:
        desc_v = _maybe_make_tensor_desc(desc_v, shape=[HEAD_DIM, y_dim], strides=[N_CTX, 1],
                                         block_shape=[HEAD_DIM, BLOCK_N])
    else:
        desc_v = _maybe_make_tensor_desc(desc_v, shape=[y_dim, HEAD_DIM], strides=[HEAD_DIM, 1],
                                         block_shape=[BLOCK_N, HEAD_DIM])
    desc_k = _maybe_make_tensor_desc(desc_k, shape=[y_dim, HEAD_DIM], strides=[HEAD_DIM, 1],
                                     block_shape=[BLOCK_N, HEAD_DIM])
    desc_o = _maybe_make_tensor_desc(desc_o, shape=[y_dim, HEAD_DIM], strides=[HEAD_DIM, 1],
                                     block_shape=[BLOCK_M, HEAD_DIM])

    offset_y = off_z * (N_CTX * H) + off_h * N_CTX
    qo_offset_y = offset_y + start_m * BLOCK_M
    # initialize offsets
    offs_m = start_m * BLOCK_M + tl.arange(0, BLOCK_M)
    offs_n = tl.arange(0, BLOCK_N)
    # initialize pointer to m and l
    m_i = tl.zeros([BLOCK_M], dtype=tl.float32) - float("inf")
    l_i = tl.zeros([BLOCK_M], dtype=tl.float32) + 1.0
    acc = tl.zeros([BLOCK_M, HEAD_DIM], dtype=tl.float32)
    # load scales
    qk_scale = sm_scale
    qk_scale *= 1.44269504  # 1/log(2)
    # load q: it will stay in SRAM throughout
    q = desc_q.load([qo_offset_y, 0])
    # stage 1: off-band
    # For causal = True, STAGE = 3 and _attn_fwd_inner gets 1 as its STAGE
    # For causal = False, STAGE = 1, and _attn_fwd_inner gets 3 as its STAGE
    if STAGE & 1:
        acc, l_i, m_i = _attn_fwd_inner(acc, l_i, m_i, q,  #
                                        desc_k, desc_v,  #
                                        offset_y, dtype, start_m, qk_scale,  #
                                        BLOCK_M, HEAD_DIM, BLOCK_N,  #
                                        4 - STAGE, offs_m, offs_n, N_CTX,  #
                                        warp_specialize, IS_HOPPER)
    # stage 2: on-band
    if STAGE & 2:
        acc, l_i, m_i = _attn_fwd_inner(acc, l_i, m_i, q,  #
                                        desc_k, desc_v,  #
                                        offset_y, dtype, start_m, qk_scale,  #
                                        BLOCK_M, HEAD_DIM, BLOCK_N,  #
                                        2, offs_m, offs_n, N_CTX,  #
                                        warp_specialize, IS_HOPPER)
    # epilogue
    m_i += tl.math.log2(l_i)
    acc = acc / l_i[:, None]
    m_ptrs = M + off_hz * N_CTX + offs_m
    tl.store(m_ptrs, m_i)
    desc_o.store([qo_offset_y, 0], acc.to(dtype))
```
**EN:** _maybe_make_tensor_desc converts raw pointers into Triton tensor descriptors when needed, which lets the forward kernel work with both descriptor and pointer inputs. _attn_fwd flattens (Z, H, N_CTX) into a single y dimension, builds descriptor views for Q/K/V/O, and selects FP16 or FP8 output behavior. For FP8 output, V is treated with a transposed descriptor layout because the dot-product operand format differs. The kernel then runs one or two stages depending on causal mode and finishes by storing the running log-sum-exp statistics M plus the normalized output tile O.
**CN:** _maybe_make_tensor_desc 在需要时把原始指针包装成 Triton tensor descriptor，从而让前向内核同时支持 descriptor 输入和普通指针输入。_attn_fwd 把 (Z, H, N_CTX) 展平成一个 y 维度，为 Q/K/V/O 构造 descriptor 视图，并在 FP16 与 FP8 输出之间选择路径。对于 FP8 输出，V 使用转置后的 descriptor 布局，因为点积操作数格式要求不同。随后内核根据 causal 模式执行一个或两个阶段，最后把在线 log-sum-exp 统计量 M 与归一化输出 O 写回内存。

### Block 5 (Lines 250-264) / 代码块 5（第 250-264 行）

```python
@triton.jit
def _attn_bwd_preprocess(O, DO,  #
                         Delta,  #
                         Z, H, N_CTX,  #
                         BLOCK_M: tl.constexpr, HEAD_DIM: tl.constexpr  #
                         ):
    off_m = tl.program_id(0) * BLOCK_M + tl.arange(0, BLOCK_M)
    off_hz = tl.program_id(1)
    off_n = tl.arange(0, HEAD_DIM)
    # load
    o = tl.load(O + off_hz * HEAD_DIM * N_CTX + off_m[:, None] * HEAD_DIM + off_n[None, :])
    do = tl.load(DO + off_hz * HEAD_DIM * N_CTX + off_m[:, None] * HEAD_DIM + off_n[None, :]).to(tl.float32)
    delta = tl.sum(o * do, axis=1)
    # write-back
    tl.store(Delta + off_hz * N_CTX + off_m, delta)
```
**EN:** _attn_bwd_preprocess computes delta = sum(O * DO, axis=1) for each query row. Flash Attention v2 uses this cached reduction during backward softmax differentiation so the main backward kernel does not need to recompute it repeatedly. The output is stored in Delta and later passed as D.
**CN:** _attn_bwd_preprocess 为每个 query 行计算 delta = sum(O * DO, axis=1)。Flash Attention v2 在反向 softmax 求导时会复用这个中间量，因此主反向内核无需反复重算。结果写入 Delta，并在后续以内核参数 D 的形式传入。

### Block 6 (Lines 267-317) / 代码块 6（第 267-317 行）

```python
# The main inner-loop logic for computing dK and dV.
@triton.jit
def _attn_bwd_dkdv(dk, dv,  #
                   Q, k, v, sm_scale,  #
                   DO,  #
                   M, D,  #
                   # shared by Q/K/V/DO.
                   stride_tok, stride_d,  #
                   H, N_CTX, BLOCK_M1: tl.constexpr,  #
                   BLOCK_N1: tl.constexpr,  #
                   HEAD_DIM: tl.constexpr,  #
                   # Filled in by the wrapper.
                   start_n, start_m, num_steps,  #
                   MASK: tl.constexpr):
    offs_m = start_m + tl.arange(0, BLOCK_M1)
    offs_n = start_n + tl.arange(0, BLOCK_N1)
    offs_k = tl.arange(0, HEAD_DIM)
    qT_ptrs = Q + offs_m[None, :] * stride_tok + offs_k[:, None] * stride_d
    do_ptrs = DO + offs_m[:, None] * stride_tok + offs_k[None, :] * stride_d
    # BLOCK_N1 must be a multiple of BLOCK_M1, otherwise the code wouldn't work.
    tl.static_assert(BLOCK_N1 % BLOCK_M1 == 0)
    curr_m = start_m
    step_m = BLOCK_M1
    for blk_idx in range(num_steps):
        qT = tl.load(qT_ptrs)
        # Load m before computing qk to reduce pipeline stall.
        offs_m = curr_m + tl.arange(0, BLOCK_M1)
        m = tl.load(M + offs_m)
        qkT = tl.dot(k, qT)
        pT = tl.math.exp2(qkT - m[None, :])
        # Autoregressive masking.
        if MASK:
            mask = (offs_m[None, :] >= offs_n[:, None])
            pT = tl.where(mask, pT, 0.0)
        do = tl.load(do_ptrs)
        # Compute dV.
        ppT = pT
        ppT = ppT.to(tl.float16)
        dv += tl.dot(ppT, do)
        # D (= delta) is pre-divided by ds_scale.
        Di = tl.load(D + offs_m)
        # Compute dP and dS.
        dpT = tl.dot(v, tl.trans(do)).to(tl.float32)
        dsT = pT * (dpT - Di[None, :])
        dsT = dsT.to(tl.float16)
        dk += tl.dot(dsT, tl.trans(qT))
        # Increment pointers.
        curr_m += step_m
        qT_ptrs += step_m * stride_tok
        do_ptrs += step_m * stride_tok
    return dk, dv
```
**EN:** _attn_bwd_dkdv is the inner loop for dK and dV. It reloads Q^T and DO blocks, recomputes probabilities p from qk and the saved row maxima M, optionally applies the causal mask, and accumulates dV via P^T @ dO. It then forms dP and dS using the precomputed delta D and accumulates dK via dS^T @ Q. This matches the FlashAttention strategy of recomputation instead of storing the full attention matrix.
**CN:** _attn_bwd_dkdv 是计算 dK 和 dV 的内循环。它重新加载 Q^T 与 DO 分块，结合保存的逐行最大值 M 重新计算概率 p，并在需要时施加因果 mask，然后通过 P^T @ dO 累加 dV。之后利用预计算的 delta D 构造 dP 与 dS，再通过 dS^T @ Q 累加 dK。这正是 FlashAttention 以“重计算代替显式保存完整注意力矩阵”的核心思想。

### Block 7 (Lines 321-365) / 代码块 7（第 321-365 行）

```python
@triton.jit
def _attn_bwd_dq(dq, q, K, V,  #
                 do, m, D,
                 # shared by Q/K/V/DO.
                 stride_tok, stride_d,  #
                 H, N_CTX,  #
                 BLOCK_M2: tl.constexpr,  #
                 BLOCK_N2: tl.constexpr,  #
                 HEAD_DIM: tl.constexpr,
                 # Filled in by the wrapper.
                 start_m, start_n, num_steps,  #
                 MASK: tl.constexpr):
    offs_m = start_m + tl.arange(0, BLOCK_M2)
    offs_n = start_n + tl.arange(0, BLOCK_N2)
    offs_k = tl.arange(0, HEAD_DIM)
    kT_ptrs = K + offs_n[None, :] * stride_tok + offs_k[:, None] * stride_d
    vT_ptrs = V + offs_n[None, :] * stride_tok + offs_k[:, None] * stride_d
    # D (= delta) is pre-divided by ds_scale.
    Di = tl.load(D + offs_m)
    # BLOCK_M2 must be a multiple of BLOCK_N2, otherwise the code wouldn't work.
    tl.static_assert(BLOCK_M2 % BLOCK_N2 == 0)
    curr_n = start_n
    step_n = BLOCK_N2
    for blk_idx in range(num_steps):
        kT = tl.load(kT_ptrs)
        vT = tl.load(vT_ptrs)
        qk = tl.dot(q, kT)
        p = tl.math.exp2(qk - m)
        # Autoregressive masking.
        if MASK:
            offs_n = curr_n + tl.arange(0, BLOCK_N2)
            mask = (offs_m[:, None] >= offs_n[None, :])
            p = tl.where(mask, p, 0.0)
        # Compute dP and dS.
        dp = tl.dot(do, vT).to(tl.float32)
        ds = p * (dp - Di[:, None])
        ds = ds.to(tl.float16)
        # Compute dQ.
        # NOTE: We need to de-scale dq in the end, because kT was pre-scaled.
        dq += tl.dot(ds, tl.trans(kT))
        # Increment pointers.
        curr_n += step_n
        kT_ptrs += step_n * stride_tok
        vT_ptrs += step_n * stride_tok
    return dq
```
**EN:** _attn_bwd_dq mirrors the same idea for dQ. It walks over K/V tiles, reconstructs p, applies the optional diagonal causal mask, forms dS = p * (dP - delta), and accumulates dQ through dS @ K^T. The comment about de-scaling is important: K is pre-scaled before the main backward launch, so the wrapper later multiplies dQ by ln(2) to undo the base-2 scaling convention used in the kernels.
**CN:** _attn_bwd_dq 以相同思路计算 dQ。它遍历 K/V 分块，重建概率 p，按需施加对角因果 mask，构造 dS = p * (dP - delta)，并通过 dS @ K^T 累加 dQ。代码中关于“去缩放”的注释非常关键：主反向启动前 K 已经被预缩放，因此包装层会在最后把 dQ 乘以 ln(2)，以抵消内核内部采用的以 2 为底的缩放约定。

### Block 8 (Lines 368-502) / 代码块 8（第 368-502 行）

```python
@triton.jit
def _attn_bwd(Q, K, V, sm_scale,  #
              DO,  #
              DQ, DK, DV,  #
              M, D,
              # shared by Q/K/V/DO.
              stride_z, stride_h, stride_tok, stride_d,  #
              H, N_CTX,  #
              BLOCK_M1: tl.constexpr,  #
              BLOCK_N1: tl.constexpr,  #
              BLOCK_M2: tl.constexpr,  #
              BLOCK_N2: tl.constexpr,  #
              BLK_SLICE_FACTOR: tl.constexpr,  #
              HEAD_DIM: tl.constexpr,  #
              CAUSAL: tl.constexpr):
    LN2: tl.constexpr = 0.6931471824645996  # = ln(2)

    bhid = tl.program_id(2)
    off_chz = (bhid * N_CTX).to(tl.int64)
    adj = (stride_h * (bhid % H) + stride_z * (bhid // H)).to(tl.int64)
    pid = tl.program_id(0)

    # offset pointers for batch/head
    Q += adj
    K += adj
    V += adj
    DO += adj
    DQ += adj
    DK += adj
    DV += adj
    M += off_chz
    D += off_chz

    # load scales
    offs_k = tl.arange(0, HEAD_DIM)

    start_n = pid * BLOCK_N1
    start_m = 0

    MASK_BLOCK_M1: tl.constexpr = BLOCK_M1 // BLK_SLICE_FACTOR
    offs_n = start_n + tl.arange(0, BLOCK_N1)

    dv = tl.zeros([BLOCK_N1, HEAD_DIM], dtype=tl.float32)
    dk = tl.zeros([BLOCK_N1, HEAD_DIM], dtype=tl.float32)

    # load K and V: they stay in SRAM throughout the inner loop.
    k = tl.load(K + offs_n[:, None] * stride_tok + offs_k[None, :] * stride_d)
    v = tl.load(V + offs_n[:, None] * stride_tok + offs_k[None, :] * stride_d)

    if CAUSAL:
        start_m = start_n
        num_steps = BLOCK_N1 // MASK_BLOCK_M1
        dk, dv = _attn_bwd_dkdv(dk, dv,  #
                                Q, k, v, sm_scale,  #
                                DO,  #
                                M, D,  #
                                stride_tok, stride_d,  #
                                H, N_CTX,  #
                                MASK_BLOCK_M1, BLOCK_N1, HEAD_DIM,  #
                                start_n, start_m, num_steps,  #
                                MASK=True,  #
                                )

        start_m += num_steps * MASK_BLOCK_M1

    # Compute dK and dV for non-masked blocks.
    num_steps = (N_CTX - start_m) // BLOCK_M1
    dk, dv = _attn_bwd_dkdv(  #
        dk, dv,  #
        Q, k, v, sm_scale,  #
        DO,  #
        M, D,  #
        stride_tok, stride_d,  #
        H, N_CTX,  #
        BLOCK_M1, BLOCK_N1, HEAD_DIM,  #
        start_n, start_m, num_steps,  #
        MASK=False,  #
    )

    dv_ptrs = DV + offs_n[:, None] * stride_tok + offs_k[None, :] * stride_d
    tl.store(dv_ptrs, dv)

    # Write back dK.
    dk *= sm_scale
    dk_ptrs = DK + offs_n[:, None] * stride_tok + offs_k[None, :] * stride_d
    tl.store(dk_ptrs, dk)

    # THIS BLOCK DOES DQ:
    start_m = pid * BLOCK_M2
    start_n = 0
    num_steps = N_CTX // BLOCK_N2

    MASK_BLOCK_N2: tl.constexpr = BLOCK_N2 // BLK_SLICE_FACTOR
    offs_m = start_m + tl.arange(0, BLOCK_M2)

    q = tl.load(Q + offs_m[:, None] * stride_tok + offs_k[None, :] * stride_d)
    dq = tl.zeros([BLOCK_M2, HEAD_DIM], dtype=tl.float32)
    do = tl.load(DO + offs_m[:, None] * stride_tok + offs_k[None, :] * stride_d)

    m = tl.load(M + offs_m)
    m = m[:, None]

    if CAUSAL:
        # Compute dQ for masked (diagonal) blocks.
        # NOTE: This code scans each row of QK^T backward (from right to left,
        # but inside each call to _attn_bwd_dq, from left to right), but that's
        # not due to anything important.  I just wanted to reuse the loop
        # structure for dK & dV above as much as possible.
        end_n = start_m + BLOCK_M2
        num_steps = BLOCK_M2 // MASK_BLOCK_N2
        dq = _attn_bwd_dq(dq, q, K, V,  #
                          do, m, D,  #
                          stride_tok, stride_d,  #
                          H, N_CTX,  #
                          BLOCK_M2, MASK_BLOCK_N2, HEAD_DIM,  #
                          start_m, end_n - num_steps * MASK_BLOCK_N2, num_steps,  #
                          MASK=True,  #
                          )
        end_n -= num_steps * MASK_BLOCK_N2
        # stage 2
        num_steps = end_n // BLOCK_N2
        start_n = end_n - num_steps * BLOCK_N2

    dq = _attn_bwd_dq(dq, q, K, V,  #
                      do, m, D,  #
                      stride_tok, stride_d,  #
                      H, N_CTX,  #
                      BLOCK_M2, BLOCK_N2, HEAD_DIM,  #
                      start_m, start_n, num_steps,  #
                      MASK=False,  #
                      )
    # Write back dQ.
    dq_ptrs = DQ + offs_m[:, None] * stride_tok + offs_k[None, :] * stride_d
    dq *= LN2
    tl.store(dq_ptrs, dq)
```
**EN:** _attn_bwd is the main backward kernel that coordinates both gradient paths. Each program instance owns one K/V tile (indexed by pid over BLOCK_N1), keeps that tile in SRAM, and first computes dK/dV. In causal mode it separates the diagonal masked slices from the fully valid region. It then switches to the dQ path, where each program processes one Q tile, again splitting masked and unmasked regions. Finally it writes DV, DK, and DQ back, with dK multiplied by sm_scale and dQ multiplied by ln(2) so the algebra matches the forward kernel's exp2-based scaling.
**CN:** _attn_bwd 是协调两条梯度路径的主反向内核。每个程序实例负责一个 K/V 分块（由 pid 和 BLOCK_N1 决定），先将该分块常驻 SRAM 并计算 dK/dV。在 causal 模式下，它会把对角附近需要 mask 的区域与完全有效区域分开处理。之后切换到 dQ 路径，每个程序再处理一个 Q 分块，同样区分 masked 与 non-masked 区域。最后将 DV、DK、DQ 写回，其中 dK 乘上 sm_scale，dQ 乘上 ln(2)，从而与前向内核基于 exp2 的缩放体系保持一致。

### Block 9 (Lines 505-575) / 代码块 9（第 505-575 行）

```python
class _attention(torch.autograd.Function):

    @staticmethod
    def forward(ctx, q, k, v, causal, sm_scale, warp_specialize=True):
        # shape constraints
        HEAD_DIM_Q, HEAD_DIM_K = q.shape[-1], k.shape[-1]
        # when v is in float8_e5m2 it is transposed.
        HEAD_DIM_V = v.shape[-1]
        assert HEAD_DIM_Q == HEAD_DIM_K and HEAD_DIM_K == HEAD_DIM_V
        assert HEAD_DIM_K in {16, 32, 64, 128, 256}
        o = torch.empty_like(q)
        stage = 3 if causal else 1
        extra_kern_args = {}
        # Tuning for AMD target
        if is_hip():
            waves_per_eu = 3 if HEAD_DIM_K <= 64 else 2
            extra_kern_args = {"waves_per_eu": waves_per_eu, "allow_flush_denorm": True}

        M = torch.empty((q.shape[0], q.shape[1], q.shape[2]), device=q.device, dtype=torch.float32)
        # Use device_descriptor for Hopper + warpspec.
        if supports_host_descriptor() and not (is_hopper() and warp_specialize):
            # Note that on Hopper we cannot perform a FP8 dot with a non-transposed second tensor
            y_dim = q.shape[0] * q.shape[1] * q.shape[2]

            dummy_block = [1, 1]
            desc_q = TensorDescriptor(q, shape=[y_dim, HEAD_DIM_K], strides=[HEAD_DIM_K, 1], block_shape=dummy_block)
            if q.dtype == torch.float8_e5m2:
                desc_v = TensorDescriptor(v, shape=[HEAD_DIM_K, y_dim], strides=[q.shape[2], 1],
                                          block_shape=dummy_block)
            else:
                desc_v = TensorDescriptor(v, shape=[y_dim, HEAD_DIM_K], strides=[HEAD_DIM_K, 1],
                                          block_shape=dummy_block)
            desc_k = TensorDescriptor(k, shape=[y_dim, HEAD_DIM_K], strides=[HEAD_DIM_K, 1], block_shape=dummy_block)
            desc_o = TensorDescriptor(o, shape=[y_dim, HEAD_DIM_K], strides=[HEAD_DIM_K, 1], block_shape=dummy_block)
        else:
            desc_q = q
            desc_v = v
            desc_k = k
            desc_o = o

        def alloc_fn(size: int, align: int, _):
            return torch.empty(size, dtype=torch.int8, device="cuda")

        triton.set_allocator(alloc_fn)

        def grid(META):
            return (triton.cdiv(q.shape[2], META["BLOCK_M"]), q.shape[0] * q.shape[1], 1)

        ctx.grid = grid
        if is_blackwell() and warp_specialize:
            if HEAD_DIM_K == 128 and q.dtype == torch.float16:
                extra_kern_args["maxnreg"] = 168
            else:
                extra_kern_args["maxnreg"] = 80
        _attn_fwd[grid](
            sm_scale, M,  #
            q.shape[0], q.shape[1],  #
            desc_q, desc_k, desc_v, desc_o,  #
            N_CTX=q.shape[2],  #
            HEAD_DIM=HEAD_DIM_K,  #
            FP8_OUTPUT=q.dtype == torch.float8_e5m2,  #
            STAGE=stage,  #
            warp_specialize=warp_specialize,  #
            IS_HOPPER=is_hopper(),  #
            **extra_kern_args)

        ctx.save_for_backward(q, k, v, o, M)
        ctx.sm_scale = sm_scale
        ctx.HEAD_DIM = HEAD_DIM_K
        ctx.causal = causal
        return o
```
**EN:** The autograd forward wrapper validates head dimensions, allocates the output O and the per-row statistics tensor M, and selects launch parameters. On AMD it tunes waves_per_eu; on Hopper/Blackwell it decides whether host descriptors are usable and adjusts register limits for warp specialization. The wrapper constructs TensorDescriptor objects when allowed, installs a CUDA allocator for Triton runtime allocations, launches _attn_fwd, and saves Q/K/V/O/M for backward. Causal attention maps to STAGE=3 so the kernel runs both the off-band and on-band passes.
**CN:** 这个 autograd 前向包装层会先检查 head 维度是否合法，然后分配输出 O 和逐行统计张量 M，并选择启动参数。在 AMD 上它调节 waves_per_eu；在 Hopper/Blackwell 上则判断是否可以使用 host descriptor，并为 warp specialization 调整寄存器上限。包装层在允许时构造 TensorDescriptor，对 Triton 运行时安装 CUDA 分配器，随后启动 _attn_fwd，并把 Q/K/V/O/M 保存到上下文中供反向使用。对于 causal 注意力，它把 STAGE 设为 3，使内核执行 off-band 与 on-band 两个阶段。

### Block 10 (Lines 577-623) / 代码块 10（第 577-623 行）

```python
    @staticmethod
    def backward(ctx, do):
        q, k, v, o, M = ctx.saved_tensors
        assert do.is_contiguous()
        assert q.stride() == k.stride() == v.stride() == o.stride() == do.stride()
        dq = torch.empty_like(q)
        dk = torch.empty_like(k)
        dv = torch.empty_like(v)
        BATCH, N_HEAD, N_CTX = q.shape[:3]
        PRE_BLOCK = 128
        NUM_WARPS, NUM_STAGES = 4, 5
        BLOCK_M1, BLOCK_N1, BLOCK_M2, BLOCK_N2 = 32, 128, 128, 32
        BLK_SLICE_FACTOR = 2
        RCP_LN2 = 1.4426950408889634  # = 1.0 / ln(2)
        arg_k = k
        arg_k = arg_k * (ctx.sm_scale * RCP_LN2)
        PRE_BLOCK = 128
        assert N_CTX % PRE_BLOCK == 0
        pre_grid = (N_CTX // PRE_BLOCK, BATCH * N_HEAD)
        delta = torch.empty_like(M)
        _attn_bwd_preprocess[pre_grid](
            o, do,  #
            delta,  #
            BATCH, N_HEAD, N_CTX,  #
            BLOCK_M=PRE_BLOCK, HEAD_DIM=ctx.HEAD_DIM  #
        )
        grid = (N_CTX // BLOCK_N1, 1, BATCH * N_HEAD)
        _attn_bwd[grid](
            q, arg_k, v, ctx.sm_scale, do, dq, dk, dv,  #
            M, delta,  #
            q.stride(0), q.stride(1), q.stride(2), q.stride(3),  #
            N_HEAD, N_CTX,  #
            BLOCK_M1=BLOCK_M1, BLOCK_N1=BLOCK_N1,  #
            BLOCK_M2=BLOCK_M2, BLOCK_N2=BLOCK_N2,  #
            BLK_SLICE_FACTOR=BLK_SLICE_FACTOR,  #
            HEAD_DIM=ctx.HEAD_DIM,  #
            num_warps=NUM_WARPS,  #
            num_stages=NUM_STAGES,  #
            CAUSAL=ctx.causal,  #
        )

        return dq, dk, dv, None, None, None, None


attention = _attention.apply

TORCH_HAS_FP8 = hasattr(torch, 'float8_e5m2')
```
**EN:** The autograd backward wrapper enforces contiguous layouts and shared strides, allocates dq/dk/dv, and prepares constants for the backward launch. A key detail is arg_k = k * (sm_scale / ln(2)); this lets the kernels stay in exp2 space during recomputation. It first runs _attn_bwd_preprocess to produce delta, then launches _attn_bwd with fixed tile sizes. The public Python API is exposed as attention = _attention.apply, and TORCH_HAS_FP8 records whether PyTorch exposes float8_e5m2.
**CN:** 这个 autograd 反向包装层会强制要求输入布局连续且步长一致，分配 dq/dk/dv，并准备反向启动所需常量。一个关键细节是 arg_k = k * (sm_scale / ln(2))；这样内核在重计算过程中就可以始终停留在 exp2 空间中。它先调用 _attn_bwd_preprocess 生成 delta，再以固定 tile 大小启动 _attn_bwd。对外公开的 Python 接口是 attention = _attention.apply，而 TORCH_HAS_FP8 用于记录当前 PyTorch 是否提供 float8_e5m2 类型。

### Block 11 (Lines 626-690) / 代码块 11（第 626-690 行）

```python
@pytest.mark.parametrize("Z", [1, 4])
@pytest.mark.parametrize("H", [2, 48])
@pytest.mark.parametrize("N_CTX", [128, 1024, (2 if is_hip() else 4) * 1024])
@pytest.mark.parametrize("HEAD_DIM", [64, 128])
@pytest.mark.parametrize("causal", [False, True])
@pytest.mark.parametrize("warp_specialize", [False, True] if is_blackwell() else [False])
@pytest.mark.parametrize("mode", ["fwd", "bwd"])
@pytest.mark.parametrize("provider", ["triton-fp16"] + (["triton-fp8"] if TORCH_HAS_FP8 else []))
def test_op(Z, H, N_CTX, HEAD_DIM, causal, warp_specialize, mode, provider, dtype=torch.float16):
    if mode == "fwd" and "fp16" in provider:
        pytest.skip("Avoid running the forward computation twice.")
    if mode == "bwd" and "fp8" in provider:
        pytest.skip("Backward pass with FP8 is not supported.")
    torch.manual_seed(20)
    q = (torch.empty((Z, H, N_CTX, HEAD_DIM), dtype=dtype, device=DEVICE).normal_(mean=0.0, std=0.5).requires_grad_())
    k = (torch.empty((Z, H, N_CTX, HEAD_DIM), dtype=dtype, device=DEVICE).normal_(mean=0.0, std=0.5).requires_grad_())
    v = (torch.empty((Z, H, N_CTX, HEAD_DIM), dtype=dtype, device=DEVICE).normal_(mean=0.0, std=0.5).requires_grad_())
    sm_scale = 0.5
    # reference implementation
    ref_dtype = dtype
    if mode == "fwd" and "fp8" in provider:
        ref_dtype = torch.float32
    q = q.to(ref_dtype)
    k = k.to(ref_dtype)
    v = v.to(ref_dtype)
    M = torch.tril(torch.ones((N_CTX, N_CTX), device=DEVICE))
    p = torch.matmul(q, k.transpose(2, 3)) * sm_scale
    if causal:
        p[:, :, M == 0] = float("-inf")
    p = torch.softmax(p.float(), dim=-1)
    p = p.to(ref_dtype)
    # p = torch.exp(p)
    ref_out = torch.matmul(p, v).half()
    if mode == "bwd":
        dout = torch.randn_like(q)
        ref_out.backward(dout)
        ref_dv, v.grad = v.grad.clone(), None
        ref_dk, k.grad = k.grad.clone(), None
        ref_dq, q.grad = q.grad.clone(), None
    # triton implementation
    if mode == "fwd" and "fp8" in provider:
        q = q.to(torch.float8_e5m2)
        k = k.to(torch.float8_e5m2)
        v = v.permute(0, 1, 3, 2).contiguous()
        v = v.permute(0, 1, 3, 2)
        v = v.to(torch.float8_e5m2)
    tri_out = attention(q, k, v, causal, sm_scale, warp_specialize).half()
    if mode == "fwd":
        atol = 3 if "fp8" in provider else 1e-2
        torch.testing.assert_close(tri_out, ref_out, atol=atol, rtol=0)
        return
    tri_out.backward(dout)
    tri_dv, v.grad = v.grad.clone(), None
    tri_dk, k.grad = k.grad.clone(), None
    tri_dq, q.grad = q.grad.clone(), None
    # compare
    torch.testing.assert_close(tri_out, ref_out, atol=1e-2, rtol=0)
    rtol = 0.0
    # Relative tolerance workaround for known hardware limitation of CDNA2 GPU.
    # For details see https://pytorch.org/docs/stable/notes/numerical_accuracy.html#reduced-precision-fp16-and-bf16-gemms-and-convolutions-on-amd-instinct-mi200-devices
    if torch.version.hip is not None and triton.runtime.driver.active.get_current_target().arch == "gfx90a":
        rtol = 1e-2
    torch.testing.assert_close(tri_dv, ref_dv, atol=1e-2, rtol=rtol)
    torch.testing.assert_close(tri_dk, ref_dk, atol=1e-2, rtol=rtol)
    torch.testing.assert_close(tri_dq, ref_dq, atol=1e-2, rtol=rtol)
```
**EN:** The pytest block validates both forward and backward correctness across batch sizes, head counts, sequence lengths, head dimensions, causal/non-causal mode, and optional warp specialization on Blackwell. It builds a PyTorch reference using explicit QK^T, masking, softmax, and PV multiplication. For FP8 forward, the reference is computed in higher precision and the Triton path permutes V into the layout expected by the FP8 kernel. Backward correctness compares dq/dk/dv, with a relaxed AMD tolerance for known MI200 reduced-precision behavior.
**CN:** 这一段 pytest 用例会在不同 batch、大量头数、序列长度、head 维度、causal/non-causal 模式，以及 Blackwell 可选 warp specialization 组合下验证前向和反向正确性。它用 PyTorch 显式构造 QK^T、mask、softmax 和 PV 乘法作为参考实现。对于 FP8 前向，参考结果在更高精度下计算，而 Triton 路径会先把 V 变换到 FP8 内核期望的布局。反向验证会比较 dq/dk/dv，并针对 AMD MI200 系列已知的低精度行为放宽相对误差阈值。

### Block 12 (Lines 693-775) / 代码块 12（第 693-775 行）

```python
try:
    from flash_attn.flash_attn_interface import \
        flash_attn_qkvpacked_func as flash_attn_func
    HAS_FLASH = True
except BaseException:
    HAS_FLASH = False

TORCH_HAS_FP8 = hasattr(torch, 'float8_e5m2')
BATCH, N_HEADS = 4, 32
# vary seq length for fixed head and batch=4
configs = []
for HEAD_DIM in [64, 128]:
    for mode in ["fwd", "bwd"]:
        for causal in [True, False]:
            # Enable warpspec for causal fwd on Hopper
            enable_ws = mode == "fwd" and (is_blackwell() or (is_hopper() and not causal))
            for warp_specialize in [False, True] if enable_ws else [False]:
                configs.append(
                    triton.testing.Benchmark(
                        x_names=["N_CTX"],
                        x_vals=[2**i for i in range(10, 15)],
                        line_arg="provider",
                        line_vals=["triton-fp16"] + (["triton-fp8"] if TORCH_HAS_FP8 else []) +
                        (["flash"] if HAS_FLASH else []),
                        line_names=["Triton [FP16]"] + (["Triton [FP8]"] if TORCH_HAS_FP8 else []) +
                        (["Flash-2"] if HAS_FLASH else []),
                        styles=[("red", "-"), ("blue", "-"), ("green", "-")],
                        ylabel="TFLOPS",
                        plot_name=
                        f"fused-attention-batch{BATCH}-head{N_HEADS}-d{HEAD_DIM}-{mode}-causal={causal}-warp_specialize={warp_specialize}",
                        args={
                            "H": N_HEADS,
                            "BATCH": BATCH,
                            "HEAD_DIM": HEAD_DIM,
                            "mode": mode,
                            "causal": causal,
                            "warp_specialize": warp_specialize,
                        },
                    ))


@triton.testing.perf_report(configs)
def bench_flash_attention(BATCH, H, N_CTX, HEAD_DIM, causal, warp_specialize, mode, provider, device=DEVICE):
    assert mode in ["fwd", "bwd"]
    dtype = torch.float16
    if "triton" in provider:
        q = torch.randn((BATCH, H, N_CTX, HEAD_DIM), dtype=dtype, device=device, requires_grad=True)
        k = torch.randn((BATCH, H, N_CTX, HEAD_DIM), dtype=dtype, device=device, requires_grad=True)
        v = torch.randn((BATCH, H, N_CTX, HEAD_DIM), dtype=dtype, device=device, requires_grad=True)
        if mode == "fwd" and "fp8" in provider:
            q = q.to(torch.float8_e5m2)
            k = k.to(torch.float8_e5m2)
            v = v.permute(0, 1, 3, 2).contiguous()
            v = v.permute(0, 1, 3, 2)
            v = v.to(torch.float8_e5m2)
        sm_scale = 1.3
        fn = lambda: attention(q, k, v, causal, sm_scale, warp_specialize)
        if mode == "bwd":
            o = fn()
            do = torch.randn_like(o)
            fn = lambda: o.backward(do, retain_graph=True)
        ms = triton.testing.do_bench(fn)

    if provider == "flash":
        qkv = torch.randn((BATCH, N_CTX, 3, H, HEAD_DIM), dtype=dtype, device=device, requires_grad=True)
        fn = lambda: flash_attn_func(qkv, causal=causal)
        if mode == "bwd":
            o = fn()
            do = torch.randn_like(o)
            fn = lambda: o.backward(do, retain_graph=True)
        ms = triton.testing.do_bench(fn)
    flops_per_matmul = 2.0 * BATCH * H * N_CTX * N_CTX * HEAD_DIM
    total_flops = 2 * flops_per_matmul
    if causal:
        total_flops *= 0.5
    if mode == "bwd":
        total_flops *= 2.5  # 2.0(bwd) + 0.5(recompute)
    return total_flops * 1e-12 / (ms * 1e-3)


if __name__ == "__main__":
    # only works on post-Ampere GPUs right now
    bench_flash_attention.run(save_path=".", print_data=True)
```
**EN:** The final section optionally imports FlashAttention-2 for comparison, builds a benchmark matrix over head dimension, mode, causality, and warp specialization, and registers a perf-report function. The Triton benchmark path measures either forward execution or backward via retain_graph. The FlashAttention path uses qkv-packed input, matching that library's interface. FLOP accounting reflects attention's two matrix multiplications, halves the count for causal mode, and multiplies backward by 2.5 to include recomputation cost. Running the file directly emits performance data and plots.
**CN:** 最后一部分会在可用时导入 FlashAttention-2 作为对照，随后围绕 head 维度、执行模式、是否因果以及 warp specialization 组合构建基准矩阵，并注册 perf-report 函数。Triton 路径可以分别测试前向或通过 retain_graph 测试反向；FlashAttention 路径则使用其库接口要求的 qkv-packed 输入。FLOP 统计考虑了注意力中的两次矩阵乘法，在 causal 模式下减半，并把反向乘以 2.5 以计入重计算成本。直接运行该文件会输出性能数据并生成图表。

## Key Concepts / 关键概念

- **Online softmax / 在线 softmax**  
  **EN:** The forward kernel never materializes the full attention matrix. Instead, it keeps per-row maxima `m_i`, normalizers `l_i`, and a running accumulator `acc`, which is the defining memory-saving idea behind Flash Attention v2.  
  **CN:** 前向内核不会显式构造完整注意力矩阵，而是维护逐行最大值 `m_i`、归一化因子 `l_i` 和运行中累加器 `acc`，这正是 Flash Attention v2 节省显存的核心。
- **Base-2 exponent math / 以 2 为底的指数计算**  
  **EN:** The code converts scaling factors with `1/log(2)` so it can use `tl.math.exp2`, which is efficient on GPUs and keeps forward/backward numerically consistent.  
  **CN:** 代码通过 `1/log(2)` 把缩放因子转换到以 2 为底的指数域，以便使用 `tl.math.exp2`，这在 GPU 上更高效，也让前反向的数值约定保持一致。
- **Causal staging / 因果分阶段计算**  
  **EN:** Causal forward is split into off-band and on-band phases so only the diagonal tile needs masking. Backward similarly separates masked diagonal slices from the unmasked region.  
  **CN:** 因果前向会拆成 off-band 与 on-band 两个阶段，因此只有对角块需要 mask；反向也采用类似策略，把对角 masked 区域与普通区域分开处理。
- **Recomputation in backward / 反向中的重计算**  
  **EN:** Instead of storing attention probabilities, backward recomputes them from `Q`, `K`, and the saved row maxima `M`. This trades extra arithmetic for much lower memory traffic.  
  **CN:** 反向阶段不会保存完整注意力概率，而是利用 `Q`、`K` 与保存下来的逐行最大值 `M` 进行重计算，用更多算力换取更低的内存带宽压力。
- **Descriptor-aware kernels / 感知 descriptor 的内核**  
  **EN:** The tutorial supports both raw pointers and `TensorDescriptor`, allowing newer NVIDIA architectures to use descriptor-based memory access and FP8-friendly layouts.  
  **CN:** 该教程同时支持原始指针与 `TensorDescriptor`，使较新的 NVIDIA 架构能够利用基于 descriptor 的访存方式以及更适合 FP8 的布局。
- **Hardware-specific tuning / 面向硬件的调优**  
  **EN:** HIP uses AMD-specific launch hints, Hopper/Blackwell affect descriptor use and register limits, and autotune prunes tile shapes that are invalid or slow on a given target.  
  **CN:** HIP 路径带有 AMD 特定的启动提示；Hopper/Blackwell 会影响 descriptor 使用与寄存器限制；自动调优还会剔除在特定目标上无效或性能较差的 tile 配置。

## Dependencies / 依赖关系

- **External libraries / 外部库**  
  **EN:** Core dependencies are `torch`, `triton`, `triton.language as tl`, `pytest`, and `TensorDescriptor` from `triton.tools.tensor_descriptor`. Optional benchmarking comparison uses `flash_attn.flash_attn_interface`.  
  **CN:** 核心依赖包括 `torch`、`triton`、`triton.language as tl`、`pytest`，以及 `triton.tools.tensor_descriptor` 中的 `TensorDescriptor`。可选的性能对照依赖 `flash_attn.flash_attn_interface`。
- **Forward call graph / 前向调用图**  
  **EN:** `attention -> _attention.forward -> _attn_fwd -> _attn_fwd_inner`. Autotuning and descriptor pre-hooks influence `_attn_fwd` launch configuration.  
  **CN:** `attention -> _attention.forward -> _attn_fwd -> _attn_fwd_inner`。自动调优与 descriptor pre-hook 会共同影响 `_attn_fwd` 的启动配置。
- **Backward call graph / 反向调用图**  
  **EN:** `autograd backward -> _attention.backward -> _attn_bwd_preprocess -> _attn_bwd -> (_attn_bwd_dkdv, _attn_bwd_dq)`. The saved tensors `q, k, v, o, M` connect forward and backward.  
  **CN:** `autograd backward -> _attention.backward -> _attn_bwd_preprocess -> _attn_bwd -> (_attn_bwd_dkdv, _attn_bwd_dq)`。前向保存的 `q, k, v, o, M` 是连接前后向的关键数据。
- **Data dependencies / 数据依赖**  
  **EN:** Forward stores `M` (row-wise log-sum-exp state) and output `O`; backward uses `M`, `O`, and `DO` to build `Delta`, then recomputes attention probabilities to derive `DQ`, `DK`, and `DV`.  
  **CN:** 前向会保存 `M`（逐行 log-sum-exp 状态）和输出 `O`；反向利用 `M`、`O` 与 `DO` 构造 `Delta`，再通过重计算注意力概率得到 `DQ`、`DK` 和 `DV`。
- **Hardware dependencies / 硬件依赖**  
  **EN:** Host descriptors require CUDA capability >= 9, Blackwell gets additional register tuning, HIP takes a separate tuning path, and FP8 backward is intentionally unsupported in this tutorial.  
  **CN:** Host descriptor 需要 CUDA 计算能力 >= 9；Blackwell 还会做额外寄存器调优；HIP 走单独调优路径；本教程则明确不支持 FP8 反向。
- **Validation and benchmarking dependencies / 验证与基准依赖**  
  **EN:** `test_op` depends on the exported `attention` API and a PyTorch reference implementation, while `bench_flash_attention` depends on Triton's benchmarking helpers and optionally FlashAttention-2 for comparison.  
  **CN:** `test_op` 依赖导出的 `attention` 接口以及 PyTorch 参考实现，而 `bench_flash_attention` 依赖 Triton 的基准辅助工具，并可选地引入 FlashAttention-2 作为对照。

