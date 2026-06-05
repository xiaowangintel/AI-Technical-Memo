# _lowrank.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_lowrank.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides general-purpose torch Python helpers, package wiring, or small runtime-facing utilities.
- **Purpose (CN)**: 提供通用的 torch Python 辅助逻辑、包级连接代码或小型运行时工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行
````python
"""Implement various linear algebra algorithms for low rank matrices."""

__all__ = ["svd_lowrank", "pca_lowrank"]


import torch
from torch import _linalg_utils as _utils, Tensor
from torch.overrides import handle_torch_function, has_torch_function


def get_approximate_basis(
    A: Tensor,
    q: int,
    niter: int | None = 2,
    M: Tensor | None = None,
) -> Tensor:
    """Return tensor :math:`Q` with :math:`q` orthonormal columns such
    that :math:`Q Q^H A` approximates :math:`A`. If :math:`M` is
    specified, then :math:`Q` is such that :math:`Q Q^H (A - M)`
    approximates :math:`A - M`. without instantiating any tensors
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.overrides. The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. `__all__` defines the public symbols that this module chooses to export. This chunk defines `get_approximate_basis`, which retrieves runtime state and exposes it through a Python-friendly accessor. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.overrides。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 `__all__` 定义了本模块选择导出的公共符号。 这一段定义了 `get_approximate_basis`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 21-39 / 第 21-39 行
````python
    of the size of :math:`A` or :math:`M`.

    .. note:: The implementation is based on the Algorithm 4.4 from
              Halko et al., 2009.

    .. note:: For an adequate approximation of a k-rank matrix
              :math:`A`, where k is not known in advance but could be
              estimated, the number of :math:`Q` columns, q, can be
              chosen according to the following criteria: in general,
              :math:`k <= q <= min(2*k, m, n)`. For large low-rank
              matrices, take :math:`q = k + 5..10`.  If k is
              relatively small compared to :math:`min(m, n)`, choosing
              :math:`q = k + 0..2` may be sufficient.

    .. note:: To obtain repeatable results, reset the seed for the
              pseudorandom number generator

    Args::
        A (Tensor): the input tensor of size :math:`(*, m, n)`
````
- **EN**: This chunk continues `get_approximate_basis` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `get_approximate_basis`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 41-58 / 第 41-58 行
````python
        q (int): the dimension of subspace spanned by :math:`Q`
                 columns.

        niter (int, optional): the number of subspace iterations to
                               conduct; ``niter`` must be a
                               nonnegative integer. In most cases, the
                               default value 2 is more than enough.

        M (Tensor, optional): the input tensor's mean of size
                              :math:`(*, m, n)`.

    References::
        - Nathan Halko, Per-Gunnar Martinsson, and Joel Tropp, Finding
          structure with randomness: probabilistic algorithms for
          constructing approximate matrix decompositions,
          arXiv:0909.4061 [math.NA; math.PR], 2009 (available at
          `arXiv <http://arxiv.org/abs/0909.4061>`_).
    """
````
- **EN**: This chunk continues `get_approximate_basis` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `get_approximate_basis`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 60-79 / 第 60-79 行
````python
    niter = 2 if niter is None else niter
    dtype = _utils.get_floating_dtype(A) if not A.is_complex() else A.dtype
    matmul = _utils.matmul

    R = torch.randn(A.shape[-1], q, dtype=dtype, device=A.device)

    # The following code could be made faster using torch.geqrf + torch.ormqr
    # but geqrf is not differentiable

    X = matmul(A, R)
    if M is not None:
        X = X - matmul(M, R)
    Q = torch.linalg.qr(X).Q
    for _ in range(niter):
        X = matmul(A.mH, Q)
        if M is not None:
            X = X - matmul(M.mH, Q)
        Q = torch.linalg.qr(X).Q
        X = matmul(A, Q)
        if M is not None:
````
- **EN**: This chunk continues `get_approximate_basis` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `get_approximate_basis`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 80-97 / 第 80-97 行
````python
            X = X - matmul(M, Q)
        Q = torch.linalg.qr(X).Q
    return Q


def svd_lowrank(
    A: Tensor,
    q: int | None = 6,
    niter: int | None = 2,
    M: Tensor | None = None,
) -> tuple[Tensor, Tensor, Tensor]:
    r"""Return the singular value decomposition ``(U, S, V)`` of a matrix,
    batches of matrices, or a sparse matrix :math:`A` such that
    :math:`A \approx U \operatorname{diag}(S) V^{\text{H}}`. In case :math:`M` is given, then
    SVD is computed for the matrix :math:`A - M`.

    .. note:: The implementation is based on the Algorithm 5.1 from
              Halko et al., 2009.
````
- **EN**: This chunk defines `svd_lowrank`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `svd_lowrank`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 99-115 / 第 99-115 行
````python
    .. note:: For an adequate approximation of a k-rank matrix
              :math:`A`, where k is not known in advance but could be
              estimated, the number of :math:`Q` columns, q, can be
              chosen according to the following criteria: in general,
              :math:`k <= q <= min(2*k, m, n)`. For large low-rank
              matrices, take :math:`q = k + 5..10`.  If k is
              relatively small compared to :math:`min(m, n)`, choosing
              :math:`q = k + 0..2` may be sufficient.

    .. note:: This is a randomized method. To obtain repeatable results,
              set the seed for the pseudorandom number generator

    .. note:: In general, use the full-rank SVD implementation
              :func:`torch.linalg.svd` for dense matrices due to its 10x
              higher performance characteristics. The low-rank SVD
              will be useful for huge sparse matrices that
              :func:`torch.linalg.svd` cannot handle.
````
- **EN**: This chunk continues `svd_lowrank` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `svd_lowrank`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 117-135 / 第 117-135 行
````python
    Args::
        A (Tensor): the input tensor of size :math:`(*, m, n)`

        q (int, optional): a slightly overestimated rank of A.

        niter (int, optional): the number of subspace iterations to
                               conduct; niter must be a nonnegative
                               integer, and defaults to 2

        M (Tensor, optional): the input tensor's mean of size
                              :math:`(*, m, n)`, which will be broadcasted
                              to the size of A in this function.

    References::
        - Nathan Halko, Per-Gunnar Martinsson, and Joel Tropp, Finding
          structure with randomness: probabilistic algorithms for
          constructing approximate matrix decompositions,
          arXiv:0909.4061 [math.NA; math.PR], 2009 (available at
          `arXiv <https://arxiv.org/abs/0909.4061>`_).
````
- **EN**: This chunk continues `svd_lowrank` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `svd_lowrank`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 137-155 / 第 137-155 行
````python
    """
    if not torch.jit.is_scripting():
        tensor_ops = (A, M)
        if not set(map(type, tensor_ops)).issubset(
            (torch.Tensor, type(None))
        ) and has_torch_function(tensor_ops):
            return handle_torch_function(
                svd_lowrank, tensor_ops, A, q=q, niter=niter, M=M
            )
    return _svd_lowrank(A, q=q, niter=niter, M=M)


def _svd_lowrank(
    A: Tensor,
    q: int | None = 6,
    niter: int | None = 2,
    M: Tensor | None = None,
) -> tuple[Tensor, Tensor, Tensor]:
    # Algorithm 5.1 in Halko et al., 2009
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_svd_lowrank`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_svd_lowrank`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 157-175 / 第 157-175 行
````python
    q = 6 if q is None else q
    m, n = A.shape[-2:]
    matmul = _utils.matmul
    if M is not None:
        M = M.broadcast_to(A.size())

    # Assume that A is tall
    if m < n:
        A = A.mH
        if M is not None:
            M = M.mH

    Q = get_approximate_basis(A, q, niter=niter, M=M)
    B = matmul(Q.mH, A)
    if M is not None:
        B = B - matmul(Q.mH, M)
    U, S, Vh = torch.linalg.svd(B, full_matrices=False)
    V = Vh.mH
    U = Q.matmul(U)
````
- **EN**: This chunk continues `_svd_lowrank` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `_svd_lowrank`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 177-194 / 第 177-194 行
````python
    if m < n:
        U, V = V, U

    return U, S, V


def pca_lowrank(
    A: Tensor,
    q: int | None = None,
    center: bool = True,
    niter: int = 2,
) -> tuple[Tensor, Tensor, Tensor]:
    r"""Performs linear Principal Component Analysis (PCA) on a low-rank
    matrix, batches of such matrices, or sparse matrix.

    This function returns a namedtuple ``(U, S, V)`` which is the
    nearly optimal approximation of a singular value decomposition of
    a centered matrix :math:`A` such that :math:`A \approx U \operatorname{diag}(S) V^{\text{H}}`
````
- **EN**: This chunk defines `pca_lowrank`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `pca_lowrank`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 196-214 / 第 196-214 行
````python
    .. note:: The relation of ``(U, S, V)`` to PCA is as follows:

                - :math:`A` is a data matrix with ``m`` samples and
                  ``n`` features

                - the :math:`V` columns represent the principal directions

                - :math:`S ** 2 / (m - 1)` contains the eigenvalues of
                  :math:`A^T A / (m - 1)` which is the covariance of
                  ``A`` when ``center=True`` is provided.

                - ``matmul(A, V[:, :k])`` projects data to the first k
                  principal components

    .. note:: Different from the standard SVD, the size of returned
              matrices depend on the specified rank and q
              values as follows:

                - :math:`U` is m x q matrix
````
- **EN**: This chunk continues `pca_lowrank` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 这一段延续了 `pca_lowrank`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 216-233 / 第 216-233 行
````python
                - :math:`S` is q-vector

                - :math:`V` is n x q matrix

    .. note:: To obtain repeatable results, reset the seed for the
              pseudorandom number generator

    Args:

        A (Tensor): the input tensor of size :math:`(*, m, n)`

        q (int, optional): a slightly overestimated rank of
                           :math:`A`. By default, ``q = min(6, m,
                           n)``.

        center (bool, optional): if True, center the input tensor,
                                 otherwise, assume that the input is
                                 centered.
````
- **EN**: This chunk continues `pca_lowrank` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `pca_lowrank`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 235-253 / 第 235-253 行
````python
        niter (int, optional): the number of subspace iterations to
                               conduct; niter must be a nonnegative
                               integer, and defaults to 2.

    References::

        - Nathan Halko, Per-Gunnar Martinsson, and Joel Tropp, Finding
          structure with randomness: probabilistic algorithms for
          constructing approximate matrix decompositions,
          arXiv:0909.4061 [math.NA; math.PR], 2009 (available at
          `arXiv <http://arxiv.org/abs/0909.4061>`_).

    """

    if not torch.jit.is_scripting():
        if type(A) is not torch.Tensor and has_torch_function((A,)):
            return handle_torch_function(
                pca_lowrank, (A,), A, q=q, center=center, niter=niter
            )
````
- **EN**: This chunk continues `pca_lowrank` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `pca_lowrank`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 255-269 / 第 255-269 行
````python
    (m, n) = A.shape[-2:]

    if q is None:
        q = min(6, m, n)
    elif not (q >= 0 and q <= min(m, n)):
        raise ValueError(
            f"q(={q}) must be non-negative integer and not greater than min(m, n)={min(m, n)}"
        )
    if not (niter >= 0):
        raise ValueError(f"niter(={niter}) must be non-negative integer")

    dtype = _utils.get_floating_dtype(A)

    if not center:
        return _svd_lowrank(A, q, niter=niter, M=None)
````
- **EN**: This chunk continues `pca_lowrank` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `pca_lowrank`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 271-286 / 第 271-286 行
````python
    if _utils.is_sparse(A):
        if len(A.shape) != 2:
            raise ValueError("pca_lowrank input is expected to be 2-dimensional tensor")
        c = torch.sparse.sum(A, dim=(-2,)) / m
        # reshape c
        column_indices = c.indices()[0]
        indices = torch.zeros(
            2,
            len(column_indices),
            dtype=column_indices.dtype,
            device=column_indices.device,
        )
        indices[0] = column_indices
        C_t = torch.sparse_coo_tensor(
            indices, c.values(), (n, 1), dtype=dtype, device=A.device
        )
````
- **EN**: This chunk continues `pca_lowrank` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results.
- **CN**: 这一段延续了 `pca_lowrank`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。

### Lines 288-293 / 第 288-293 行
````python
        ones_m1_t = torch.ones(A.shape[:-2] + (1, m), dtype=dtype, device=A.device)
        M = torch.sparse.mm(C_t, ones_m1_t).mT
        return _svd_lowrank(A, q, niter=niter, M=M)
    else:
        C = A.mean(dim=(-2,), keepdim=True)
        return _svd_lowrank(A - C, q, niter=niter, M=None)
````
- **EN**: This chunk continues `pca_lowrank` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `pca_lowrank`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **get_approximate_basis**
  - EN: `get_approximate_basis` is one of the main symbols declared or implemented in this file.
  - CN: `get_approximate_basis` 是本文件声明或实现的主要符号之一。
- **Sparse semantics**
  - EN: The logic preserves sparse layouts and sparse-specific invariants across helpers and wrappers.
  - CN: 逻辑会在辅助函数和包装层中保持稀疏布局及其不变量。
- **Decomposition**
  - EN: The implementation rewrites larger operators into smaller primitives for reuse and lowering.
  - CN: 实现把较大的算子重写为更小的 primitive，以便复用和降级。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
- **Vectorization**
  - EN: The code exposes SIMD or packed-value helpers for CPU-side performance.
  - CN: 代码暴露 SIMD 或打包数值辅助逻辑，以提升 CPU 侧性能。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.overrides`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `get_approximate_basis`, `svd_lowrank`, `_svd_lowrank`, `pca_lowrank`
