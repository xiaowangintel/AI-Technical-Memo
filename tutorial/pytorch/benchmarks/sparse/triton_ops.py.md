# triton_ops.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `benchmarks/sparse/triton_ops.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10 / 第 1-10 行

```python
import torch
from torch._inductor.runtime.benchmarking import benchmarker


def create_blocked_tensor(B, M, N, blocksize, sparsity, dtype, device):
    if not (sparsity <= 1.0 and sparsity >= 0.0):
        raise AssertionError(
            f"sparsity should be a value between 0 and 1, but got {sparsity}"
        )
    if M % blocksize[0] != 0:
```

- **EN:** Important local symbols in this block include create_blocked_tensor.
- **CN:** 该代码块中的重要局部符号包括 create_blocked_tensor。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 11-23 / 第 11-23 行

```python
        raise AssertionError(
            f"M ({M}) must be divisible by blocksize[0] ({blocksize[0]})"
        )
    if N % blocksize[1] != 0:
        raise AssertionError(
            f"N ({N}) must be divisible by blocksize[1] ({blocksize[1]})"
        )
    shape = (B, M // blocksize[0], N // blocksize[1])[int(B == 0) :]
    A = torch.bernoulli(torch.full(shape, 1 - sparsity, dtype=dtype, device=device))
    expected_nnz = int((1 - sparsity) * M * N / (blocksize[0] * blocksize[1]))
    nonzero_indices = A.flatten().nonzero()
    actual_nnz = nonzero_indices.shape[0]
    if actual_nnz > expected_nnz:
```

- **EN:** Tensor construction or buffer wrapping happens here, so input layout and data preparation matter to the surrounding logic.
- **CN:** 这里发生了张量构造或缓冲区封装，因此输入布局与数据准备对周边逻辑很重要。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 24-34 / 第 24-34 行

```python
        selected_nonzeros = torch.randperm(actual_nnz)[: actual_nnz - expected_nnz]
        A.flatten()[nonzero_indices[selected_nonzeros]] = 0
    elif actual_nnz < expected_nnz:
        zero_indices = (A == 0).flatten().nonzero()
        selected_zeros = torch.randperm(zero_indices.shape[0])[
            : expected_nnz - actual_nnz
        ]
        A.flatten()[zero_indices[selected_zeros]] = 1
    A = torch.repeat_interleave(A, blocksize[0], dim=-2)
    A = torch.repeat_interleave(A, blocksize[1], dim=-1)
    return A
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 35-42 / 第 35-42 行

```python


def _test_worker(test_func):
    ms, ms_min, ms_max = benchmarker.benchmark_gpu(test_func, warmup=500, rep=100)

    tflops = 2 * m * k * n * 1e-12 / (ms * 1e-3)
    return ms, tflops

```

- **EN:** Important local symbols in this block include _test_worker.
- **CN:** 该代码块中的重要局部符号包括 _test_worker。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 43-50 / 第 43-50 行

```python

def test_dense_dense_mm(x, y, **meta):
    def test_func(x=x.to_dense(), y=y):
        return torch.matmul(x, y)

    return _test_worker(test_func)


```

- **EN:** Important local symbols in this block include test_dense_dense_mm, test_func.
- **CN:** 该代码块中的重要局部符号包括 test_dense_dense_mm、test_func。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 51-58 / 第 51-58 行

```python
def test_torch_matmul(x, y, **meta):
    def test_func(x=x, y=y):
        return torch.matmul(x, y)

    return _test_worker(test_func)


def test_bsr_dense_mm(x, y, **meta):
```

- **EN:** Important local symbols in this block include test_torch_matmul, test_func, test_bsr_dense_mm.
- **CN:** 该代码块中的重要局部符号包括 test_torch_matmul、test_func、test_bsr_dense_mm。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 59-66 / 第 59-66 行

```python
    from torch.sparse._triton_ops import bsr_dense_mm

    def test_func(x=x, y=y):
        return bsr_dense_mm(
            x, y, meta=dict(GROUP_SIZE_ROW=4, num_stages=1, num_warps=4)
        )

    return _test_worker(test_func)
```

- **EN:** Important local symbols in this block include test_func.
- **CN:** 该代码块中的重要局部符号包括 test_func。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 67-74 / 第 67-74 行

```python


def test_bsr_dense_mm_with_meta(x, y, **meta):
    from torch.sparse._triton_ops import bsr_dense_mm

    def test_func(x=x, y=y, meta=meta):
        return bsr_dense_mm(x, y, meta=meta)

```

- **EN:** Important local symbols in this block include test_bsr_dense_mm_with_meta, test_func.
- **CN:** 该代码块中的重要局部符号包括 test_bsr_dense_mm_with_meta、test_func。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 75-84 / 第 75-84 行

```python
    return _test_worker(test_func)


def test_bsr_scatter_mm2(x, y, **meta):
    from torch.sparse._triton_ops import bsr_scatter_mm, bsr_scatter_mm_indices_data

    indices_data = bsr_scatter_mm_indices_data(
        x, y, indices_format="scatter_mm", **meta
    )

```

- **EN:** Important local symbols in this block include test_bsr_scatter_mm2.
- **CN:** 该代码块中的重要局部符号包括 test_bsr_scatter_mm2。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 85-93 / 第 85-93 行

```python
    def test_func(x=x, y=y):
        return bsr_scatter_mm(x, y, indices_data=indices_data)

    return _test_worker(test_func)


def test_bsr_scatter_mm6(x, y, **meta):
    from torch.sparse._triton_ops import bsr_scatter_mm, bsr_scatter_mm_indices_data

```

- **EN:** Important local symbols in this block include test_func, test_bsr_scatter_mm6.
- **CN:** 该代码块中的重要局部符号包括 test_func、test_bsr_scatter_mm6。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 94-101 / 第 94-101 行

```python
    indices_data = bsr_scatter_mm_indices_data(
        x, y, indices_format="bsr_strided_mm_compressed", **meta
    )

    def test_func(x=x, y=y):
        return bsr_scatter_mm(x, y, indices_data=indices_data)

    return _test_worker(test_func)
```

- **EN:** Important local symbols in this block include test_func.
- **CN:** 该代码块中的重要局部符号包括 test_func。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 102-111 / 第 102-111 行

```python


def test_bsr_scatter_mm(x, y, **meta):
    from torch.sparse._triton_ops import bsr_scatter_mm, bsr_scatter_mm_indices_data

    def test_func(x=x, y=y):
        indices_data = bsr_scatter_mm_indices_data(
            x, y, indices_format="bsr_strided_mm_compressed", **meta
        )
        return bsr_scatter_mm(x, y, indices_data=indices_data)
```

- **EN:** Important local symbols in this block include test_bsr_scatter_mm, test_func.
- **CN:** 该代码块中的重要局部符号包括 test_bsr_scatter_mm、test_func。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 112-119 / 第 112-119 行

```python

    return _test_worker(test_func)


def test_linear(x, y, **meta):
    import torch.nn.functional as F

    def test_func(x=x, y=y.transpose(-2, -1)):
```

- **EN:** Important local symbols in this block include test_linear, test_func.
- **CN:** 该代码块中的重要局部符号包括 test_linear、test_func。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 120-130 / 第 120-130 行

```python
        return F.linear(y, x)

    return _test_worker(test_func)


if __name__ == "__main__":
    import argparse
    import atexit
    import itertools
    import sys

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 131-138 / 第 131-138 行

```python
    import triton

    from torch.testing import make_tensor

    torch.manual_seed(0)

    def integer_list(a):
        return list(map(int, a.split(",")))
```

- **EN:** Important local symbols in this block include integer_list.
- **CN:** 该代码块中的重要局部符号包括 integer_list。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 139-146 / 第 139-146 行

```python

    def float_list(a):
        return list(map(float, a.split(",")))

    def integer_or_float_list(a):
        lst = []
        for n in a.split(","):
            if n.count(":") == 1:
```

- **EN:** Important local symbols in this block include float_list, integer_or_float_list.
- **CN:** 该代码块中的重要局部符号包括 float_list、integer_or_float_list。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 147-156 / 第 147-156 行

```python
                start, end = map(int, n.split(":"))
                lst.extend(range(start, end))
            elif n.count(":") == 2:
                start, end, step = map(int, n.split(":"))
                lst.extend(range(start, end, step))
            elif "." in n:
                lst.append(float(n))
            else:
                lst.append(int(n))
        return lst
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 157-166 / 第 157-166 行

```python

    parser = argparse.ArgumentParser(description="SpTritonOps")

    parser.add_argument(
        "--ops",
        default="dense_dense_mm,bsr_dense_mm,bsr_scatter_mm6",
        type=str,
    )
    parser.add_argument("--b", default="0", type=int)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 167-182 / 第 167-182 行

```python
    parser.add_argument("--m", default="1024", type=integer_list)
    parser.add_argument("--k", default=None, type=integer_list)
    parser.add_argument("--n", default=None, type=integer_list)
    parser.add_argument("--bm", default="16", type=integer_list)
    parser.add_argument("--bk", default=None, type=integer_list)
    parser.add_argument("--tile_m", default=None, type=integer_list)
    parser.add_argument("--tile_n", default=None, type=integer_list)
    parser.add_argument("--split_n", default=None, type=integer_list)
    parser.add_argument("--group_size", default=None, type=integer_list)
    parser.add_argument("--num_warps", default=None, type=integer_list)
    parser.add_argument("--num_stages", default=None, type=integer_list)
    parser.add_argument("--sparsity", default="0.5", type=integer_or_float_list)
    parser.add_argument("--dtype", default="float16", type=str)
    parser.add_argument("--device", default="cuda", type=str)
    parser.add_argument("--repeat", default="1", type=int)
    parser.add_argument("--outfile", default="stdout", type=str)
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 183-196 / 第 183-196 行

```python
    parser.add_argument("--star", default=False, action="store_true")

    args = parser.parse_args()

    if args.outfile == "stdout":
        outfile = sys.stdout
        need_close = False
    elif args.outfile == "stderr":
        outfile = sys.stderr
        need_close = False
    else:
        outfile = open(args.outfile, "a")  # noqa: SIM115
        need_close = True

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 197-212 / 第 197-212 行

```python
    ops = args.ops.split(",")

    b = args.b

    m_list = args.m or [1024]
    n_list = args.n or [None]
    k_list = args.k or [None]
    bm_list = args.bm or [16]
    bk_list = args.bk or [None]
    split_n_list = args.split_n or [None]
    tile_m_list = args.tile_m or [None]
    tile_n_list = args.tile_n or [None]
    group_size_list = args.group_size or [None]
    num_warps_list = args.num_warps or [None]
    num_stages_list = args.num_stages or [None]
    sparsity_list = args.sparsity or [0.5]
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 213-227 / 第 213-227 行

```python
    dtype = getattr(torch, args.dtype)

    if args.star > 0:
        import torch.sparse._triton_ops

        if {len(m_list), len(n_list), len(k_list), len(bm_list), len(bk_list)} != {1}:
            raise AssertionError(
                "When --star is set, m, n, k, bm, bk lists must all have length 1"
            )
        m = m_list[0]
        n = n_list[0] or m
        k = k_list[0] or m
        bm = bm_list[0]
        bk = bk_list[0] or bm
        if "bsr_scatter_mm6" in ops:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 228-242 / 第 228-242 行

```python
            meta = torch.sparse._triton_ops.scatter_mm_meta(m, k, n, bm, bk)
        elif "bsr_dense_mm_with_meta" in ops:
            meta = torch.sparse._triton_ops.bsr_dense_mm_meta(m, k, n, bm, bk)
        else:
            raise NotImplementedError(f"--star not implemented for operations in {ops}")
        if "bsr_scatter_mm6" in ops:
            if split_n_list[0] is None:
                split_n_list = [
                    meta["SPLIT_N"] // 2,
                    meta["SPLIT_N"],
                    meta["SPLIT_N"] * 2,
                ][int(meta["SPLIT_N"] == 1) :]
            elif split_n_list[0] == 0:
                split_n_list = [meta["SPLIT_N"]]
            if tile_m_list[0] is None:
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 243-254 / 第 243-254 行

```python
                tile_m_list = [meta["TILE_M"] // 2, meta["TILE_M"], meta["TILE_M"] * 2][
                    int(meta["TILE_M"] == 16) :
                ]
            elif tile_m_list[0] == 0:
                tile_m_list = [meta["TILE_M"]]
            if tile_n_list[0] is None:
                tile_n_list = [meta["TILE_N"] // 2, meta["TILE_N"], meta["TILE_N"] * 2][
                    int(meta["TILE_N"] == 16) :
                ]
            elif tile_n_list[0] == 0:
                tile_n_list = [meta["TILE_N"]]
            if group_size_list[0] is None:
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 255-262 / 第 255-262 行

```python
                group_size_list = [
                    meta["GROUP_SIZE"] - 1,
                    meta["GROUP_SIZE"],
                    meta["GROUP_SIZE"] + 1,
                ][int(meta["GROUP_SIZE"] == 1) :]
            elif group_size_list[0] == 0:
                group_size_list = [meta["GROUP_SIZE"]]
        if "bsr_dense_mm_with_meta" in ops:
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 263-271 / 第 263-271 行

```python
            if group_size_list[0] is None:
                group_size_list = [
                    meta["GROUP_SIZE_ROW"] - 1,
                    meta["GROUP_SIZE_ROW"],
                    meta["GROUP_SIZE_ROW"] + 1,
                ][int(meta["GROUP_SIZE_ROW"] == 1) :]
            elif group_size_list[0] == 0:
                group_size_list = [meta["GROUP_SIZE_ROW"]]
        if num_warps_list[0] is None:
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 272-279 / 第 272-279 行

```python
            num_warps_list = [
                meta["num_warps"] // 2,
                meta["num_warps"],
                meta["num_warps"] * 2,
            ][int(meta["num_warps"] == 1) :]
        elif num_warps_list[0] == 0:
            num_warps_list = [meta["num_warps"]]
        if num_stages_list[0] is None:
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 280-287 / 第 280-287 行

```python
            num_stages_list = [
                meta["num_stages"] - 1,
                meta["num_stages"],
                meta["num_stages"] + 1,
            ][int(meta["num_stages"] == 1) :]
        elif num_stages_list[0] == 0:
            num_stages_list = [meta["num_stages"]]

```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 288-296 / 第 288-296 行

```python
    device = args.device
    dense_dense_mm_sizes = set()
    target_performance = None
    performance_rtol = 1e-2

    best_messages = []

    @atexit.register
    def show_best_messages(best_messages=best_messages):
```

- **EN:** Important local symbols in this block include show_best_messages.
- **CN:** 该代码块中的重要局部符号包括 show_best_messages。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 297-308 / 第 297-308 行

```python
        print("TOP 10:")
        for m in best_messages[-10:]:
            print(m)
        sys.stdout.flush()

    for m, k, n, bm, bk, sparsity in itertools.product(
        m_list, k_list, n_list, bm_list, bk_list, sparsity_list
    ):
        k = k or m
        n = n or m
        bk = bk or bm

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 309-318 / 第 309-318 行

```python
        if bm > m or bk > k:
            # Skip invalid parameter combinations
            continue

        blocksize = (bm, bk)

        if isinstance(sparsity, int):
            # integer sparsity value corresponds to desired nnz value
            sparsity = 1 - bk * bm * sparsity / (m * k)

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 319-328 / 第 319-328 行

```python
        if sparsity > 1 or sparsity < 0:
            continue

        x = create_blocked_tensor(
            b, m, k, blocksize, sparsity, dtype, device
        ).to_sparse_bsr(blocksize)

        # recompute sparsity
        sparsity = 1 - bk * bm * x._nnz() / (m * k)

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 329-340 / 第 329-340 行

```python
        y = make_tensor(k, n, dtype=dtype, device=device)

        bsr_size = f"{b}x{m}x{k}" if b > 0 else f"{k}x{n}"

        for op in ops:
            if op == "dense_dense_mm":
                if (m, k, n) in dense_dense_mm_sizes:
                    # Skip already benchmarked cases
                    continue
                dense_dense_mm_sizes.add((m, k, n))
            best_tflops = 0
            for (
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。
- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。
- **EN:** Looping logic iterates over benchmark dimensions, shape metadata, or container elements.
- **CN:** 循环逻辑会遍历 benchmark 维度、形状元数据或容器元素。

### Lines 341-355 / 第 341-355 行

```python
                split_n,
                num_warps,
                num_stages,
                tile_m,
                tile_n,
                group_size,
            ) in itertools.product(
                split_n_list,
                num_warps_list,
                num_stages_list,
                tile_m_list,
                tile_n_list,
                group_size_list,
            ):
                if (
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 356-371 / 第 356-371 行

```python
                    (tile_m or 0) > bm
                    or (tile_n or 0) > n // (split_n or 1)
                    or n % (split_n or 1) != 0
                    or (split_n or 0) > n
                ):
                    # Skip invalid parameter combinations
                    continue
                test_func = globals()["test_" + op]
                meta = dict(
                    bsr_scatter_mm6=dict(
                        SPLIT_N=split_n,
                        TILE_M=tile_m,
                        TILE_N=tile_n,
                        GROUP_SIZE=group_size,
                        num_stages=num_stages,
                        num_warps=num_warps,
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 372-379 / 第 372-379 行

```python
                    ),
                    bsr_dense_mm_with_meta=dict(
                        GROUP_SIZE_ROW=group_size,
                        num_stages=num_stages,
                        num_warps=num_warps,
                    ),
                ).get(op, {})

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 380-395 / 第 380-395 行

```python
                meta_str = ";".join(
                    f"{k}={v}" for k, v in meta.items() if v is not None
                )
                time_ms_lst = []
                performance_tflops_lst = []
                for r in range(args.repeat):
                    try:
                        time_ms, performance_tflops = test_func(x, y, **meta)
                    except triton.compiler.OutOfResources:
                        print(
                            f"op={op}[{meta_str}]({bsr_size},{k}x{n}) dtype={args.dtype} {sparsity=}(nnz={x._nnz()})"
                            f" blocksize={bm}x{bk} OutOfResources",
                            file=outfile,
                        )
                        continue
                    except AssertionError:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 396-408 / 第 396-408 行

```python
                        raise
                    except Exception as msg:
                        msg = str(msg).split("\n", 1)[0]
                        print(
                            f"op={op}[{meta_str}]({bsr_size},{k}x{n}) dtype={args.dtype} {sparsity=}(nnz={x._nnz()})"
                            f" blocksize={bm}x{bk} {msg}",
                            file=outfile,
                        )
                        continue
                    time_ms_lst.append(time_ms)
                    performance_tflops_lst.append(performance_tflops)
                    mark = ""
                    if op == "dense_dense_mm":
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 409-417 / 第 409-417 行

```python
                        if target_performance is None:
                            target_performance = performance_tflops
                    elif target_performance is not None:
                        if (
                            abs(1 - performance_tflops / target_performance)
                            < performance_rtol
                        ):
                            mark += " @@@"
                    if best_tflops < performance_tflops:
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 418-433 / 第 418-433 行

```python
                        best_tflops = performance_tflops
                        best_message = (
                            f"op={op}[{meta_str}]({bsr_size},x{n}) dtype={args.dtype} {sparsity=:.4f}(nnz={x._nnz()})"
                            f" blocksize={bm}x{bk} time={time_ms:.3f} ms performance={performance_tflops:.3f} TFLOPS"
                        )
                        if best_message not in best_messages:
                            best_messages.append(best_message)
                        mark += " !!!"
                    print(
                        f"op={op}[{meta_str}]({bsr_size},x{n}) dtype={args.dtype} {sparsity=:.4f}(nnz={x._nnz()})"
                        f" blocksize={bm}x{bk}"
                        f" time={time_ms:.3f} ms performance={performance_tflops:.3f} TFLOPS{mark}",
                        file=outfile,
                    )
                    outfile.flush()
                if args.repeat > 1:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 434-445 / 第 434-445 行

```python
                    avg_time_ms = sum(time_ms_lst) / len(time_ms_lst)
                    avg_performance_tflops = sum(performance_tflops_lst) / len(
                        performance_tflops_lst
                    )
                    print(
                        f"op={op}[{meta_str}]({bsr_size},{k}x{n}) dtype={args.dtype} {sparsity=}(nnz={x._nnz()})"
                        f" blocksize={bm}x{bk}"
                        f" time={time_ms:.3f} ms performance={performance_tflops:.3f} TFLOPS [AVERAGE]",
                        file=outfile,
                    )
                    outfile.flush()
                if op not in {"bsr_scatter_mm6", "bsr_dense_mm_with_meta"}:
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 446-449 / 第 446-449 行

```python
                    # Break on operations that do not consume parameters
                    break
    if need_close:
        outfile.close()
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

## Key Concepts / 关键概念

- **Benchmark infrastructure** — 基准设施
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Data type coverage** — 体现文件如何在不同张量 dtype 间覆盖行为差异。
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **Representative symbols: create_blocked_tensor, _test_worker, test_dense_dense_mm, test_func, test_torch_matmul, test_bsr_dense_mm, test_bsr_dense_mm_with_meta, test_bsr_scatter_mm2** — 代表性符号：create_blocked_tensor、_test_worker、test_dense_dense_mm、test_func、test_torch_matmul、test_bsr_dense_mm、test_bsr_dense_mm_with_meta、test_bsr_scatter_mm2

## Dependencies / 依赖关系

- `torch`
- `torch._inductor.runtime.benchmarking`
- `torch.sparse._triton_ops`
- `torch.nn.functional`
- `argparse`
- `atexit`
- `itertools`
- `sys`
- `triton`
- `torch.testing`
