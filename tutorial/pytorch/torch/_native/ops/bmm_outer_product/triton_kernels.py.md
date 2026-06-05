# triton_kernels.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_native/ops/bmm_outer_product/triton_kernels.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Exposes helpers around native bindings or thin wrappers over native runtime entry points.
- **Purpose (CN)**: 暴露与 native 绑定相关的辅助逻辑，或对原生运行时入口的轻量封装。
## Line-by-Line Analysis / 逐行分析

### Lines 1-14 / 第 1-14 行
````python
import triton
import triton.language as tl

import torch


@triton.jit
def _bmm_outer_product_kernel(
    A_ptr,
    B_ptr,
    OUT_ptr,
    B_dim,
    M,
    N,
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch; other helper packages such as triton, triton.language. This chunk defines `_bmm_outer_product_kernel`, which implements a focused helper used by the surrounding module. Decorators such as `triton.jit` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch；其他辅助包，如 triton、triton.language。 这一段定义了 `_bmm_outer_product_kernel`，其作用是实现周边模块使用的关键辅助逻辑。 像 `triton.jit` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

### Lines 15-25 / 第 15-25 行
````python
    stride_ab,
    stride_am,
    stride_bb,
    stride_bn,
    stride_ob,
    stride_om,
    stride_on,
    BLOCK_M: tl.constexpr,
    BLOCK_N: tl.constexpr,
):
    pid = tl.program_id(0)
````
- **EN**: This chunk continues `_bmm_outer_product_kernel` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `_bmm_outer_product_kernel`，进一步展开其内部控制流或状态更新。

### Lines 27-37 / 第 27-37 行
````python
    grid_m = tl.cdiv(M, BLOCK_M)
    grid_n = tl.cdiv(N, BLOCK_N)
    tiles_per_batch = grid_m * grid_n

    pid_b = pid // tiles_per_batch
    pid_mn = pid % tiles_per_batch
    pid_m = pid_mn // grid_n
    pid_n = pid_mn % grid_n

    rm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
    rn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
````
- **EN**: This chunk continues `_bmm_outer_product_kernel` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `_bmm_outer_product_kernel`，进一步展开其内部控制流或状态更新。

### Lines 39-52 / 第 39-52 行
````python
    mask_m = rm < M
    mask_n = rn < N

    a = tl.load(A_ptr + pid_b * stride_ab + rm * stride_am, mask=mask_m, other=0.0)
    b = tl.load(B_ptr + pid_b * stride_bb + rn * stride_bn, mask=mask_n, other=0.0)

    out = a[:, None] * b[None, :]

    mask = mask_m[:, None] & mask_n[None, :]  # pyrefly: ignore[bad-index]
    tl.store(
        OUT_ptr + pid_b * stride_ob + rm[:, None] * stride_om + rn[None, :] * stride_on,
        out,
        mask=mask,
    )
````
- **EN**: This chunk continues `_bmm_outer_product_kernel` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `_bmm_outer_product_kernel`，进一步展开其内部控制流或状态更新。

### Lines 55-65 / 第 55-65 行
````python
def _pick_block_sizes(m: int, n: int) -> tuple[int, int]:
    """I swept over some shapes and in the future we should figure out @autotune story"""
    if m <= 32:
        block_m = triton.next_power_of_2(m)
    elif m <= 96:
        block_m = 32
    elif m <= 192:
        block_m = 64
    else:
        block_m = 128
    return block_m, min(triton.next_power_of_2(n), 128)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_pick_block_sizes`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_pick_block_sizes`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 68-81 / 第 68-81 行
````python
def bmm_outer_product(a: torch.Tensor, b: torch.Tensor) -> torch.Tensor:
    B, M, _ = a.shape
    N = b.shape[2]

    out = torch.empty(B, M, N, dtype=a.dtype, device=a.device)

    BLOCK_M, BLOCK_N = _pick_block_sizes(M, N)

    _bmm_outer_product_kernel[(B * triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N),)](
        a,
        b,
        out,
        B,
        M,
````
- **EN**: This chunk defines `bmm_outer_product`, which implements a focused helper used by the surrounding module.
- **CN**: 这一段定义了 `bmm_outer_product`，其作用是实现周边模块使用的关键辅助逻辑。

### Lines 82-93 / 第 82-93 行
````python
        N,
        a.stride(0),
        a.stride(1),
        b.stride(0),
        b.stride(2),
        out.stride(0),
        out.stride(1),
        out.stride(2),
        BLOCK_M=BLOCK_M,
        BLOCK_N=BLOCK_N,
    )
    return out
````
- **EN**: This chunk continues `bmm_outer_product` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `bmm_outer_product`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Native bindings**
  - EN: Connects Python helpers to native entry points and thin binding layers.
  - CN: 把 Python 辅助逻辑连接到原生入口点和轻量绑定层。
- **_bmm_outer_product_kernel**
  - EN: `_bmm_outer_product_kernel` is one of the main symbols declared or implemented in this file.
  - CN: `_bmm_outer_product_kernel` 是本文件声明或实现的主要符号之一。
- **_pick_block_sizes**
  - EN: `_pick_block_sizes` is one of the main symbols declared or implemented in this file.
  - CN: `_pick_block_sizes` 是本文件声明或实现的主要符号之一。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`
- **Other helper packages / 其他辅助包**: `triton`, `triton.language`
- **Primary symbols in this file / 本文件核心符号**: `_bmm_outer_product_kernel`, `_pick_block_sizes`, `bmm_outer_product`
