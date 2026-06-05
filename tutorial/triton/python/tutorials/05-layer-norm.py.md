# 05-layer-norm.py — Code Analysis / 代码分析

## Source / 来源

- **Source file / 源文件:** `/root/xw/triton/python/tutorials/05-layer-norm.py`

- **EN:** Implements a fused Triton LayerNorm with custom forward and backward kernels, including parallel reduction for weight/bias gradients and a benchmark against PyTorch/Apex.

- **CN:** 实现一个融合式 Triton LayerNorm，包括自定义前向与反向内核、用于权重/偏置梯度的并行归约，以及与 PyTorch/Apex 的性能对比。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13 — Tutorial overview / 教程概述

```python
"""
Layer Normalization
====================
In this tutorial, you will write a high-performance layer normalization
kernel that runs faster than the PyTorch implementation.

In doing so, you will learn about:

* Implementing backward pass in Triton.

* Implementing parallel reduction in Triton.

"""
```

**EN:** The docstring emphasizes two advanced Triton topics beyond simple elementwise kernels: writing backward passes and organizing parallel reductions efficiently.

**CN:** 文档字符串强调了两个比简单逐元素内核更高级的 Triton 主题：自己实现反向传播，以及高效组织并行归约。

### Lines 16-30 — LayerNorm motivation and formula / LayerNorm 动机与公式

```python
# Motivations
# -----------
#
# The *LayerNorm* operator was first introduced in [BA2016]_ as a way to improve the performance
# of sequential models (e.g., Transformers) or neural networks with small batch size.
# It takes a vector :math:`x` as input and produces a vector :math:`y` of the same shape as output.
# The normalization is performed by subtracting the mean and dividing by the standard deviation of :math:`x`.
# After the normalization, a learnable linear transformation with weights :math:`w` and biases :math:`b` is applied.
# The forward pass can be expressed as follows:
#
# .. math::
#    y = \frac{ x - \text{E}[x] }{ \sqrt{\text{Var}(x) + \epsilon} } * w + b
#
# where :math:`\epsilon` is a small constant added to the denominator for numerical stability.
# Let’s first take a look at the forward pass implementation.
```

**EN:** These comments define LayerNorm mathematically: normalize each feature vector using its mean and variance, then apply learned scale and bias. The section also explains the role of `eps` in numerical stability.

**CN:** 这一段从数学上定义了 LayerNorm：先用均值和方差对每个特征向量做归一化，再施加可学习的缩放与偏置。同时说明了 `eps` 在数值稳定性中的作用。

### Lines 32-45 — Imports and optional Apex support / 导入与可选 Apex 支持

```python
import torch

import triton
import triton.language as tl

try:
    # This is https://github.com/NVIDIA/apex, NOT the apex on PyPi, so it
    # should not be added to extras_require in setup.py.
    import apex
    HAS_APEX = True
except ModuleNotFoundError:
    HAS_APEX = False

DEVICE = triton.runtime.driver.active.get_active_torch_device()
```

**EN:** Besides the standard Triton imports, the script conditionally imports NVIDIA Apex for benchmarking. Failing to import Apex is acceptable because it is only an optional comparison target, not a functional dependency of the Triton kernels.

**CN:** 除了标准的 Triton 导入外，脚本还会按需导入 NVIDIA Apex 作为基准对比对象。即便导入失败也没关系，因为 Apex 只是可选对照，并不是 Triton 内核的功能依赖。

### Lines 48-95 — Fused LayerNorm forward kernel / 融合 LayerNorm 前向内核

```python
@triton.jit
def _layer_norm_fwd_fused(
    X,  # pointer to the input
    Y,  # pointer to the output
    W,  # pointer to the weights
    B,  # pointer to the biases
    Mean,  # pointer to the mean
    Rstd,  # pointer to the 1/std
    stride,  # how much to increase the pointer when moving by 1 row
    N,  # number of columns in X
    eps,  # epsilon to avoid division by zero
    BLOCK_SIZE: tl.constexpr,
):
    # Map the program id to the row of X and Y it should compute.
    row = tl.program_id(0)
    Y += row * stride
    X += row * stride
    # Compute mean
    mean = 0
    _mean = tl.zeros([BLOCK_SIZE], dtype=tl.float32)
    for off in range(0, N, BLOCK_SIZE):
        cols = off + tl.arange(0, BLOCK_SIZE)
        a = tl.load(X + cols, mask=cols < N, other=0.).to(tl.float32)
        _mean += a
    mean = tl.sum(_mean, axis=0) / N
    # Compute variance
    _var = tl.zeros([BLOCK_SIZE], dtype=tl.float32)
    for off in range(0, N, BLOCK_SIZE):
        cols = off + tl.arange(0, BLOCK_SIZE)
        x = tl.load(X + cols, mask=cols < N, other=0.).to(tl.float32)
        x = tl.where(cols < N, x - mean, 0.)
        _var += x * x
    var = tl.sum(_var, axis=0) / N
    rstd = 1 / tl.sqrt(var + eps)
    # Write mean / rstd
    tl.store(Mean + row, mean)
    tl.store(Rstd + row, rstd)
    # Normalize and apply linear transformation
    for off in range(0, N, BLOCK_SIZE):
        cols = off + tl.arange(0, BLOCK_SIZE)
        mask = cols < N
        w = tl.load(W + cols, mask=mask)
        b = tl.load(B + cols, mask=mask)
        x = tl.load(X + cols, mask=mask, other=0.).to(tl.float32)
        x_hat = (x - mean) * rstd
        y = x_hat * w + b
        # Write output
        tl.store(Y + cols, y, mask=mask)
```

**EN:** The forward kernel maps one program to one row, computes the row mean and variance in FP32 across one or more `BLOCK_SIZE` chunks, stores `mean` and `rstd` for backward use, then reloads the row to normalize and apply affine parameters `W` and `B`.

**CN:** 前向内核把一个 program 映射到输入的一行，在一个或多个 `BLOCK_SIZE` 分块上以 FP32 计算该行的均值与方差，把 `mean` 和 `rstd` 保存下来供反向传播使用，然后重新读取该行完成归一化，并应用仿射参数 `W` 与 `B`。

### Lines 98-130 — Backward derivation and reduction plan / 反向公式与归约策略

```python
# %%
# Backward pass
# -------------
#
# The backward pass for the layer normalization operator is a bit more involved than the forward pass.
# Let :math:`\hat{x}` be the normalized inputs :math:`\frac{ x - \text{E}[x] }{ \sqrt{\text{Var}(x) + \epsilon} }` before the linear transformation,
# the Vector-Jacobian Products (VJP) :math:`\nabla_{x}` of :math:`x` are given by:
#
# .. math::
#    \nabla_{x} = \frac{1}{\sigma}\Big( \nabla_{y} \odot w - \underbrace{ \big( \frac{1}{N} \hat{x} \cdot (\nabla_{y} \odot w) \big) }_{c_1} \odot \hat{x} - \underbrace{ \frac{1}{N} \nabla_{y} \cdot w }_{c_2} \Big)
#
# where :math:`\odot` denotes the element-wise multiplication, :math:`\cdot` denotes the dot product, and :math:`\sigma` is the standard deviation.
# :math:`c_1` and :math:`c_2` are intermediate constants that improve the readability of the following implementation.
#
# For the weights :math:`w` and biases :math:`b`, the VJPs :math:`\nabla_{w}` and :math:`\nabla_{b}` are more straightforward:
#
# .. math::
#    \nabla_{w} = \nabla_{y} \odot \hat{x} \quad \text{and} \quad \nabla_{b} = \nabla_{y}
#
# Since the same weights :math:`w` and biases :math:`b` are used for all rows in the same batch, their gradients need to sum up.
# To perform this step efficiently, we use a parallel reduction strategy: each kernel instance accumulates
# partial :math:`\nabla_{w}` and :math:`\nabla_{b}` across certain rows into one of :math:`\text{GROUP_SIZE_M}` independent buffers.
# These buffers stay in the L2 cache and then are further reduced by another function to compute the actual :math:`\nabla_{w}` and :math:`\nabla_{b}`.
#
# Let the number of input rows :math:`M = 4` and :math:`\text{GROUP_SIZE_M} = 2`,
# here's a diagram of the parallel reduction strategy for :math:`\nabla_{w}` (:math:`\nabla_{b}` is omitted for brevity):
#
#   .. image:: parallel_reduction.png
#
# In Stage 1, the rows of X that have the same color share the same buffer and thus a lock is used to ensure that only one kernel instance writes to the buffer at a time.
# In Stage 2, the buffers are further reduced to compute the final :math:`\nabla_{w}` and :math:`\nabla_{b}`.
# In the following implementation, Stage 1 is implemented by the function :code:`_layer_norm_bwd_dx_fused` and Stage 2 is implemented by the function :code:`_layer_norm_bwd_dwdb`.
```

**EN:** This comment block derives the gradients and explains why `dw`/`db` need a two-stage reduction: every input row contributes to the same parameter vectors. The tutorial therefore splits backward work into a fused `dx` kernel plus a separate reduction kernel for final parameter gradients.

**CN:** 这一大段注释推导了梯度公式，并解释为什么 `dw`/`db` 需要两阶段归约：每一行输入都会对同一组参数向量产生贡献。因此教程把反向传播拆成两个部分：一个同时计算 `dx` 和局部参数梯度的融合内核，以及一个专门做最终参数归约的内核。

### Lines 132-195 — Backward fused dx/partial-dw/db kernel / 反向融合 dx/局部 dw db 内核

```python
@triton.jit
def _layer_norm_bwd_dx_fused(DX,  # pointer to the input gradient
                             DY,  # pointer to the output gradient
                             DW,  # pointer to the partial sum of weights gradient
                             DB,  # pointer to the partial sum of biases gradient
                             X,  # pointer to the input
                             W,  # pointer to the weights
                             Mean,  # pointer to the mean
                             Rstd,  # pointer to the 1/std
                             Lock,  # pointer to the lock
                             stride,  # how much to increase the pointer when moving by 1 row
                             N,  # number of columns in X
                             GROUP_SIZE_M: tl.constexpr, BLOCK_SIZE_N: tl.constexpr):
    # Map the program id to the elements of X, DX, and DY it should compute.
    row = tl.program_id(0)
    cols = tl.arange(0, BLOCK_SIZE_N)
    mask = cols < N
    X += row * stride
    DY += row * stride
    DX += row * stride
    # Offset locks and weights/biases gradient pointer for parallel reduction
    lock_id = row % GROUP_SIZE_M
    Lock += lock_id
    Count = Lock + GROUP_SIZE_M
    DW = DW + lock_id * N + cols
    DB = DB + lock_id * N + cols
    # Load data to SRAM
    x = tl.load(X + cols, mask=mask, other=0).to(tl.float32)
    dy = tl.load(DY + cols, mask=mask, other=0).to(tl.float32)
    w = tl.load(W + cols, mask=mask).to(tl.float32)
    mean = tl.load(Mean + row)
    rstd = tl.load(Rstd + row)
    # Compute dx
    xhat = (x - mean) * rstd
    wdy = w * dy
    xhat = tl.where(mask, xhat, 0.)
    wdy = tl.where(mask, wdy, 0.)
    c1 = tl.sum(xhat * wdy, axis=0) / N
    c2 = tl.sum(wdy, axis=0) / N
    dx = (wdy - (xhat * c1 + c2)) * rstd
    # Write dx
    tl.store(DX + cols, dx, mask=mask)
    # Accumulate partial sums for dw/db
    partial_dw = (dy * xhat).to(w.dtype)
    partial_db = (dy).to(w.dtype)
    while tl.atomic_cas(Lock, 0, 1) == 1:
        pass
    count = tl.load(Count)
    # First store doesn't accumulate
    if count == 0:
        tl.atomic_xchg(Count, 1)
    else:
        partial_dw += tl.load(DW, mask=mask)
        partial_db += tl.load(DB, mask=mask)
    tl.store(DW, partial_dw, mask=mask)
    tl.store(DB, partial_db, mask=mask)

    # need a barrier to ensure all threads finished before
    # releasing the lock
    tl.debug_barrier()

    # Release the lock
    tl.atomic_xchg(Lock, 0)
```

**EN:** `_layer_norm_bwd_dx_fused` loads one row of `X`, `DY`, and `W`, reconstructs normalized activations from saved `Mean` and `Rstd`, computes `dx`, and stores it. It also accumulates partial `dw`/`db` into one of `GROUP_SIZE_M` shared buffers guarded by a spin lock built from `tl.atomic_cas` and `tl.atomic_xchg`.

**CN:** `_layer_norm_bwd_dx_fused` 会加载 `X`、`DY` 和 `W` 的一行，利用保存下来的 `Mean` 与 `Rstd` 重建归一化激活，计算 `dx` 并写回。同时，它还会把局部 `dw`/`db` 累加到 `GROUP_SIZE_M` 个共享缓冲区之一，并通过 `tl.atomic_cas` 与 `tl.atomic_xchg` 组成的自旋锁来保护并发更新。

### Lines 197-221 — Final dw/db reduction kernel / 最终 dw/db 归约内核

```python
@triton.jit
def _layer_norm_bwd_dwdb(DW,  # pointer to the partial sum of weights gradient
                         DB,  # pointer to the partial sum of biases gradient
                         FINAL_DW,  # pointer to the weights gradient
                         FINAL_DB,  # pointer to the biases gradient
                         M,  # GROUP_SIZE_M
                         N,  # number of columns
                         BLOCK_SIZE_M: tl.constexpr, BLOCK_SIZE_N: tl.constexpr):
    # Map the program id to the elements of DW and DB it should compute.
    pid = tl.program_id(0)
    cols = pid * BLOCK_SIZE_N + tl.arange(0, BLOCK_SIZE_N)
    dw = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
    db = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
    # Iterate through the rows of DW and DB to sum the partial sums.
    for i in range(0, M, BLOCK_SIZE_M):
        rows = i + tl.arange(0, BLOCK_SIZE_M)
        mask = (rows[:, None] < M) & (cols[None, :] < N)
        offs = rows[:, None] * N + cols[None, :]
        dw += tl.load(DW + offs, mask=mask, other=0.)
        db += tl.load(DB + offs, mask=mask, other=0.)
    # Write the final sum to the output.
    sum_dw = tl.sum(dw, axis=0)
    sum_db = tl.sum(db, axis=0)
    tl.store(FINAL_DW + cols, sum_dw, mask=cols < N)
    tl.store(FINAL_DB + cols, sum_db, mask=cols < N)
```

**EN:** `_layer_norm_bwd_dwdb` performs Stage 2 of the reduction. Each program visits a strip of columns, sums partial gradients across the temporary buffers, and writes the final `dw` and `db` vectors.

**CN:** `_layer_norm_bwd_dwdb` 完成归约的第二阶段。每个 program 负责一段列区间，把临时缓冲区中的局部梯度沿行方向求和，得到最终的 `dw` 和 `db` 向量。

### Lines 233-260 — Autograd forward bridge / 连接到 autograd 的前向桥接

```python
class LayerNorm(torch.autograd.Function):

    @staticmethod
    def forward(ctx, x, normalized_shape, weight, bias, eps):
        # allocate output
        y = torch.empty_like(x)
        # reshape input data into 2D tensor
        x_arg = x.reshape(-1, x.shape[-1])
        M, N = x_arg.shape
        mean = torch.empty((M, ), dtype=torch.float32, device=x.device)
        rstd = torch.empty((M, ), dtype=torch.float32, device=x.device)
        # Less than 64KB per feature: enqueue fused kernel
        MAX_FUSED_SIZE = 65536 // x.element_size()
        BLOCK_SIZE = min(MAX_FUSED_SIZE, triton.next_power_of_2(N))
        if N > BLOCK_SIZE:
            raise RuntimeError("This layer norm doesn't support feature dim >= 64KB.")
        # heuristics for number of warps
        num_warps = min(max(BLOCK_SIZE // 256, 1), 8)
        # enqueue kernel
        _layer_norm_fwd_fused[(M, )](  #
            x_arg, y, weight, bias, mean, rstd,  #
            x_arg.stride(0), N, eps,  #
            BLOCK_SIZE=BLOCK_SIZE, num_warps=num_warps, num_ctas=1)
        ctx.save_for_backward(x, weight, bias, mean, rstd)
        ctx.BLOCK_SIZE = BLOCK_SIZE
        ctx.num_warps = num_warps
        ctx.eps = eps
        return y
```

**EN:** The custom `LayerNorm` autograd function reshapes the input to 2D, allocates outputs plus saved statistics, enforces the tutorial's “feature dimension < 64KB” assumption, picks `BLOCK_SIZE`/`num_warps`, and launches the fused forward kernel.

**CN:** 自定义 `LayerNorm` autograd 函数会先把输入重排为二维，分配输出和需要保存的统计量，强制满足教程中的“特征维度小于 64KB”假设，再选择 `BLOCK_SIZE`/`num_warps` 并启动融合前向内核。

### Lines 262-294 — Autograd backward bridge / 连接到 autograd 的反向桥接

```python
    @staticmethod
    def backward(ctx, dy):
        x, w, b, m, v = ctx.saved_tensors
        # heuristics for amount of parallel reduction stream for DW/DB
        N = w.shape[0]
        GROUP_SIZE_M = 64
        if N <= 8192: GROUP_SIZE_M = 96
        if N <= 4096: GROUP_SIZE_M = 128
        if N <= 1024: GROUP_SIZE_M = 256
        # allocate output
        locks = torch.zeros(2 * GROUP_SIZE_M, dtype=torch.int32, device=w.device)
        _dw = torch.zeros((GROUP_SIZE_M, N), dtype=x.dtype, device=w.device)
        _db = torch.zeros((GROUP_SIZE_M, N), dtype=x.dtype, device=w.device)
        dw = torch.empty((N, ), dtype=w.dtype, device=w.device)
        db = torch.empty((N, ), dtype=w.dtype, device=w.device)
        dx = torch.empty_like(dy)
        # enqueue kernel using forward pass heuristics
        # also compute partial sums for DW and DB
        x_arg = x.reshape(-1, x.shape[-1])
        M, N = x_arg.shape
        _layer_norm_bwd_dx_fused[(M, )](  #
            dx, dy, _dw, _db, x, w, m, v, locks,  #
            x_arg.stride(0), N,  #
            BLOCK_SIZE_N=ctx.BLOCK_SIZE,  #
            GROUP_SIZE_M=GROUP_SIZE_M,  #
            num_warps=ctx.num_warps)
        grid = lambda meta: (triton.cdiv(N, meta['BLOCK_SIZE_N']), )
        # accumulate partial sums in separate kernel
        _layer_norm_bwd_dwdb[grid](
            _dw, _db, dw, db, min(GROUP_SIZE_M, M), N,  #
            BLOCK_SIZE_M=32,  #
            BLOCK_SIZE_N=128, num_ctas=1)
        return dx, None, dw, db, None
```

**EN:** The `backward` method chooses `GROUP_SIZE_M` heuristically from the feature size, allocates lock and temporary buffers, launches the fused `dx` kernel, then launches the second reduction kernel to finalize `dw` and `db`.

**CN:** `backward` 方法会根据特征维度启发式选择 `GROUP_SIZE_M`，分配锁和临时缓冲区，先启动融合 `dx` 内核，再启动第二个归约内核完成最终的 `dw` 和 `db`。

### Lines 297-323 — Alias and correctness test / 别名与正确性测试

```python
layer_norm = LayerNorm.apply


def test_layer_norm(M, N, dtype, eps=1e-5, device=DEVICE):
    # create data
    x_shape = (M, N)
    w_shape = (x_shape[-1], )
    weight = torch.rand(w_shape, dtype=dtype, device=device, requires_grad=True)
    bias = torch.rand(w_shape, dtype=dtype, device=device, requires_grad=True)
    x = -2.3 + 0.5 * torch.randn(x_shape, dtype=dtype, device=device)
    dy = .1 * torch.randn_like(x)
    x.requires_grad_(True)
    # forward pass
    y_tri = layer_norm(x, w_shape, weight, bias, eps)
    y_ref = torch.nn.functional.layer_norm(x, w_shape, weight, bias, eps).to(dtype)
    # backward pass (triton)
    y_tri.backward(dy, retain_graph=True)
    dx_tri, dw_tri, db_tri = [_.grad.clone() for _ in [x, weight, bias]]
    x.grad, weight.grad, bias.grad = None, None, None
    # backward pass (torch)
    y_ref.backward(dy, retain_graph=True)
    dx_ref, dw_ref, db_ref = [_.grad.clone() for _ in [x, weight, bias]]
    # compare
    assert torch.allclose(y_tri, y_ref, atol=1e-2, rtol=0)
    assert torch.allclose(dx_tri, dx_ref, atol=1e-2, rtol=0)
    assert torch.allclose(db_tri, db_ref, atol=1e-2, rtol=0)
    assert torch.allclose(dw_tri, dw_ref, atol=1e-2, rtol=0)
```

**EN:** `layer_norm = LayerNorm.apply` exposes the custom op in a PyTorch-friendly form. `test_layer_norm` then checks both forward outputs and backward gradients against `torch.nn.functional.layer_norm`, using tolerances suitable for FP16.

**CN:** `layer_norm = LayerNorm.apply` 以符合 PyTorch 使用习惯的方式暴露这个自定义算子。随后 `test_layer_norm` 会把前向输出和反向梯度都与 `torch.nn.functional.layer_norm` 比较，并采用适合 FP16 的容差。

### Lines 326-371 — Benchmark setup and timing logic / 基准配置与计时逻辑

```python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=['N'],
        x_vals=[512 * i for i in range(2, 32)],
        line_arg='provider',
        line_vals=['triton', 'torch'] + (['apex'] if HAS_APEX else []),
        line_names=['Triton', 'Torch'] + (['Apex'] if HAS_APEX else []),
        styles=[('blue', '-'), ('green', '-'), ('orange', '-')],
        ylabel='GB/s',
        plot_name='layer-norm-backward',
        args={'M': 4096, 'dtype': torch.float16, 'mode': 'backward'},
    ))
def bench_layer_norm(M, N, dtype, provider, mode='backward', eps=1e-5, device=DEVICE):
    # create data
    x_shape = (M, N)
    w_shape = (x_shape[-1], )
    weight = torch.rand(w_shape, dtype=dtype, device=device, requires_grad=True)
    bias = torch.rand(w_shape, dtype=dtype, device=device, requires_grad=True)
    x = -2.3 + 0.5 * torch.randn(x_shape, dtype=dtype, device=device)
    dy = .1 * torch.randn_like(x)
    x.requires_grad_(True)
    quantiles = [0.5, 0.2, 0.8]

    def y_fwd():

        if provider == "triton":
            return layer_norm(x, w_shape, weight, bias, eps)  # noqa: F811, E704

        if provider == "torch":
            return torch.nn.functional.layer_norm(x, w_shape, weight, bias, eps)  # noqa: F811, E704

        if provider == "apex":
            apex_layer_norm = (apex.normalization.FusedLayerNorm(w_shape).to(x.device).to(x.dtype))
            return apex_layer_norm(x)  # noqa: F811, E704

    # forward pass
    if mode == 'forward':
        gbps = lambda ms: 2 * x.numel() * x.element_size() * 1e-9 / (ms * 1e-3)
        ms, min_ms, max_ms = triton.testing.do_bench(y_fwd, quantiles=quantiles, rep=500)
    # backward pass
    if mode == 'backward':
        y = y_fwd()
        gbps = lambda ms: 3 * x.numel() * x.element_size() * 1e-9 / (ms * 1e-3)  # noqa: F811, E704
        ms, min_ms, max_ms = triton.testing.do_bench(lambda: y.backward(dy, retain_graph=True), quantiles=quantiles,
                                                     grad_to_none=[x], rep=500)
    return gbps(ms), gbps(max_ms), gbps(min_ms)
```

**EN:** The benchmark decorator sweeps feature size `N` and compares Triton against Torch and, if available, Apex. `bench_layer_norm` can measure either forward or backward mode; in backward mode it times `y.backward(...)` and converts runtime into effective GB/s.

**CN:** 基准装饰器会扫描特征维度 `N`，并将 Triton 与 Torch 以及（如果可用）Apex 进行对比。`bench_layer_norm` 既可以测前向，也可以测反向；在反向模式下，它实际计时的是 `y.backward(...)`，并把耗时换算成有效 GB/s。

### Lines 374-381 — Run benchmark and references / 运行基准并给出参考文献

```python
test_layer_norm(1151, 8192, torch.float16)
bench_layer_norm.run(save_path='.', print_data=True)

# %%
# References
# ----------
#
# .. [BA2016] Jimmy Lei Ba and Jamie Ryan Kiros and Geoffrey E. Hinton, "Layer Normalization", Arxiv 2016
```

**EN:** The script ends by running one correctness test, launching the benchmark, and citing the original LayerNorm paper. This makes the tutorial executable end-to-end from theory to measurement.

**CN:** 脚本最后先执行一次正确性测试，再运行基准测试，并引用 LayerNorm 原始论文。这样整个教程就形成了从理论到测量的可执行闭环。

## Key Concepts / 关键概念

- **EN:** LayerNorm forward is row-wise, but parameter gradients couple all rows, so backward needs both per-row work and cross-row reduction.
  **CN:** LayerNorm 的前向是按行独立的，但参数梯度会汇聚所有行，因此反向传播既需要逐行计算，也需要跨行归约。

- **EN:** Saving `mean` and `rstd` from forward avoids recomputing statistics during backward.
  **CN:** 在前向阶段保存 `mean` 和 `rstd`，可以避免反向传播时重新计算统计量。

- **EN:** The backward implementation uses a two-stage reduction to keep partial sums cache-friendly and parallelizable.
  **CN:** 反向实现使用两阶段归约，使局部累加既更利于缓存，也更容易并行化。

- **EN:** Atomic locks are used only for the temporary partial-gradient buffers; final parameter gradients are produced by a separate clean reduction kernel.
  **CN:** 原子锁只用于保护临时局部梯度缓冲区；最终参数梯度由单独的归约内核生成。

- **EN:** Wrapping Triton kernels in `torch.autograd.Function` integrates custom GPU code with PyTorch's differentiation API.
  **CN:** 把 Triton 内核封装进 `torch.autograd.Function`，即可把自定义 GPU 代码接入 PyTorch 的自动求导接口。

## Dependencies / 依赖关系

- **EN:** Forward depends on `_layer_norm_fwd_fused`; backward depends on both `_layer_norm_bwd_dx_fused` and `_layer_norm_bwd_dwdb`.
  **CN:** 前向依赖 `_layer_norm_fwd_fused`；反向同时依赖 `_layer_norm_bwd_dx_fused` 与 `_layer_norm_bwd_dwdb`。

- **EN:** `torch.autograd.Function` provides the integration point so Triton kernels participate in PyTorch training loops.
  **CN:** `torch.autograd.Function` 提供了集成点，使 Triton 内核能够参与 PyTorch 的训练流程。

- **EN:** `torch.nn.functional.layer_norm` is the numerical reference used for validation.
  **CN:** `torch.nn.functional.layer_norm` 是用于正确性验证的数值参考实现。

- **EN:** Apex is optional and used only as an extra benchmark provider when installed.
  **CN:** Apex 是可选依赖，仅在安装后作为额外的性能对比对象出现。
