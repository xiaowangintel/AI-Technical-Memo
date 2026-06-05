# multivariate_normal.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/distributions/multivariate_normal.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements the `MultivariateNormal` distribution or helper type, including parameter handling, sampling, and statistical methods.
- **Purpose (CN)**: 实现 `MultivariateNormal` 分布或辅助类型，涵盖参数处理、采样与统计方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

````python
0001: # mypy: allow-untyped-defs
0002: import math
0003: 
0004: import torch
0005: from torch import Tensor
0006: from torch.distributions import constraints
0007: from torch.distributions.distribution import Distribution
0008: from torch.distributions.utils import _standard_normal, lazy_property
0009: from torch.types import _size
0010: 
0011: 
0012: __all__ = ["MultivariateNormal"]
0013: 
0014: 
0015: def _batch_mv(bmat, bvec):
0016:     r"""
0017:     Performs a batched matrix-vector product, with compatible but different batch shapes.
0018: 
0019:     This function takes as input `bmat`, containing :math:`n \times n` matrices, and
0020:     `bvec`, containing length :math:`n` vectors.
0021: 
0022:     Both `bmat` and `bvec` may have any number of leading dimensions, which correspond
0023:     to a batch shape. They are not necessarily assumed to have the same batch shape,
0024:     just ones which can be broadcasted.
0025:     """
0026:     return torch.matmul(bmat, bvec.unsqueeze(-1)).squeeze(-1)
0027: 
0028: 
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Imports module dependencies: `math`. | CN: 导入模块依赖：`math`。
- **L3** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L4** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L5** EN: Imports `Tensor` from `torch` so later code can reuse those definitions. | CN: 从 `torch` 导入 `Tensor`，供后续代码复用这些定义。
- **L6** EN: Imports `constraints` from `torch.distributions` so later code can reuse those definitions. | CN: 从 `torch.distributions` 导入 `constraints`，供后续代码复用这些定义。
- **L7** EN: Imports `Distribution` from `torch.distributions.distribution` so later code can reuse those definitions. | CN: 从 `torch.distributions.distribution` 导入 `Distribution`，供后续代码复用这些定义。
- **L8** EN: Imports `_standard_normal, lazy_property` from `torch.distributions.utils` so later code can reuse those definitions. | CN: 从 `torch.distributions.utils` 导入 `_standard_normal, lazy_property`，供后续代码复用这些定义。
- **L9** EN: Imports `_size` from `torch.types` so later code can reuse those definitions. | CN: 从 `torch.types` 导入 `_size`，供后续代码复用这些定义。
- **L10** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L11** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L12** EN: Defines or updates the module’s public export list. | CN: 定义或更新模块的公开导出列表。
- **L13** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L14** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L15** EN: Defines function `_batch_mv`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_batch_mv`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L16** EN: Starts the docstring for function `_batch_mv`. | CN: 开始为 function `_batch_mv` 编写文档字符串。
- **L17** EN: Continues the docstring for function `_batch_mv`. | CN: 继续补充 function `_batch_mv` 的文档字符串。
- **L18** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L19** EN: Continues the docstring for function `_batch_mv`. | CN: 继续补充 function `_batch_mv` 的文档字符串。
- **L20** EN: Continues the docstring for function `_batch_mv`. | CN: 继续补充 function `_batch_mv` 的文档字符串。
- **L21** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L22** EN: Continues the docstring for function `_batch_mv`. | CN: 继续补充 function `_batch_mv` 的文档字符串。
- **L23** EN: Continues the docstring for function `_batch_mv`. | CN: 继续补充 function `_batch_mv` 的文档字符串。
- **L24** EN: Continues the docstring for function `_batch_mv`. | CN: 继续补充 function `_batch_mv` 的文档字符串。
- **L25** EN: Ends the docstring for function `_batch_mv`. | CN: 结束 function `_batch_mv` 的文档字符串。
- **L26** EN: Returns from `_batch_mv` with the computed result or updated state. | CN: 从 `_batch_mv` 返回计算结果或更新后的状态。
- **L27** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L28** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 29-56 / 第 29-56 行

````python
0029: def _batch_mahalanobis(bL, bx):
0030:     r"""
0031:     Computes the squared Mahalanobis distance :math:`\mathbf{x}^\top\mathbf{M}^{-1}\mathbf{x}`
0032:     for a factored :math:`\mathbf{M} = \mathbf{L}\mathbf{L}^\top`.
0033: 
0034:     Accepts batches for both bL and bx. They are not necessarily assumed to have the same batch
0035:     shape, but `bL` one should be able to broadcasted to `bx` one.
0036:     """
0037:     n = bx.size(-1)
0038:     bx_batch_shape = bx.shape[:-1]
0039: 
0040:     # Assume that bL.shape = (i, 1, n, n), bx.shape = (..., i, j, n),
0041:     # we are going to make bx have shape (..., 1, j,  i, 1, n) to apply batched tri.solve
0042:     bx_batch_dims = len(bx_batch_shape)
0043:     bL_batch_dims = bL.dim() - 2
0044:     outer_batch_dims = bx_batch_dims - bL_batch_dims
0045:     old_batch_dims = outer_batch_dims + bL_batch_dims
0046:     new_batch_dims = outer_batch_dims + 2 * bL_batch_dims
0047:     # Reshape bx with the shape (..., 1, i, j, 1, n)
0048:     bx_new_shape = bx.shape[:outer_batch_dims]
0049:     for sL, sx in zip(bL.shape[:-2], bx.shape[outer_batch_dims:-1]):
0050:         bx_new_shape += (sx // sL, sL)
0051:     bx_new_shape += (n,)
0052:     bx = bx.reshape(bx_new_shape)
0053:     # Permute bx to make it have shape (..., 1, j, i, 1, n)
0054:     permute_dims = (
0055:         list(range(outer_batch_dims))
0056:         + list(range(outer_batch_dims, new_batch_dims, 2))
````

- **L29** EN: Defines function `_batch_mahalanobis`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_batch_mahalanobis`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L30** EN: Starts the docstring for function `_batch_mahalanobis`. | CN: 开始为 function `_batch_mahalanobis` 编写文档字符串。
- **L31** EN: Continues the docstring for function `_batch_mahalanobis`. | CN: 继续补充 function `_batch_mahalanobis` 的文档字符串。
- **L32** EN: Continues the docstring for function `_batch_mahalanobis`. | CN: 继续补充 function `_batch_mahalanobis` 的文档字符串。
- **L33** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L34** EN: Continues the docstring for function `_batch_mahalanobis`. | CN: 继续补充 function `_batch_mahalanobis` 的文档字符串。
- **L35** EN: Continues the docstring for function `_batch_mahalanobis`. | CN: 继续补充 function `_batch_mahalanobis` 的文档字符串。
- **L36** EN: Ends the docstring for function `_batch_mahalanobis`. | CN: 结束 function `_batch_mahalanobis` 的文档字符串。
- **L37** EN: Assigns or updates `n`. | CN: 对 `n` 进行赋值或更新。
- **L38** EN: Assigns or updates `bx_batch_shape`. | CN: 对 `bx_batch_shape` 进行赋值或更新。
- **L39** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L40** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L41** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L42** EN: Assigns or updates `bx_batch_dims`. | CN: 对 `bx_batch_dims` 进行赋值或更新。
- **L43** EN: Assigns or updates `bL_batch_dims`. | CN: 对 `bL_batch_dims` 进行赋值或更新。
- **L44** EN: Assigns or updates `outer_batch_dims`. | CN: 对 `outer_batch_dims` 进行赋值或更新。
- **L45** EN: Assigns or updates `old_batch_dims`. | CN: 对 `old_batch_dims` 进行赋值或更新。
- **L46** EN: Assigns or updates `new_batch_dims`. | CN: 对 `new_batch_dims` 进行赋值或更新。
- **L47** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L48** EN: Assigns or updates `bx_new_shape`. | CN: 对 `bx_new_shape` 进行赋值或更新。
- **L49** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L50** EN: Continues `_batch_mahalanobis`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_batch_mahalanobis` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L51** EN: Continues `_batch_mahalanobis`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_batch_mahalanobis` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L52** EN: Assigns or updates `bx`. | CN: 对 `bx` 进行赋值或更新。
- **L53** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L54** EN: Assigns or updates `permute_dims`. | CN: 对 `permute_dims` 进行赋值或更新。
- **L55** EN: Invokes `list` to advance the surrounding implementation. | CN: 调用 `list` 来推进周围的实现逻辑。
- **L56** EN: Invokes `list` to advance the surrounding implementation. | CN: 调用 `list` 来推进周围的实现逻辑。

### Lines 57-78 / 第 57-78 行

````python
0057:         + list(range(outer_batch_dims + 1, new_batch_dims, 2))
0058:         + [new_batch_dims]
0059:     )
0060:     bx = bx.permute(permute_dims)
0061: 
0062:     flat_L = bL.reshape(-1, n, n)  # shape = b x n x n
0063:     flat_x = bx.reshape(-1, flat_L.size(0), n)  # shape = c x b x n
0064:     flat_x_swap = flat_x.permute(1, 2, 0)  # shape = b x n x c
0065:     M_swap = (
0066:         torch.linalg.solve_triangular(flat_L, flat_x_swap, upper=False).pow(2).sum(-2)
0067:     )  # shape = b x c
0068:     M = M_swap.t()  # shape = c x b
0069: 
0070:     # Now we revert the above reshape and permute operators.
0071:     permuted_M = M.reshape(bx.shape[:-1])  # shape = (..., 1, j, i, 1)
0072:     permute_inv_dims = list(range(outer_batch_dims))
0073:     for i in range(bL_batch_dims):
0074:         permute_inv_dims += [outer_batch_dims + i, old_batch_dims + i]
0075:     reshaped_M = permuted_M.permute(permute_inv_dims)  # shape = (..., 1, i, j, 1)
0076:     return reshaped_M.reshape(bx_batch_shape)
0077: 
0078: 
````

- **L57** EN: Invokes `list` to advance the surrounding implementation. | CN: 调用 `list` 来推进周围的实现逻辑。
- **L58** EN: Continues `_batch_mahalanobis`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_batch_mahalanobis` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L59** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L60** EN: Assigns or updates `bx`. | CN: 对 `bx` 进行赋值或更新。
- **L61** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L62** EN: Assigns or updates `flat_L`. | CN: 对 `flat_L` 进行赋值或更新。
- **L63** EN: Assigns or updates `flat_x`. | CN: 对 `flat_x` 进行赋值或更新。
- **L64** EN: Assigns or updates `flat_x_swap`. | CN: 对 `flat_x_swap` 进行赋值或更新。
- **L65** EN: Assigns or updates `M_swap`. | CN: 对 `M_swap` 进行赋值或更新。
- **L66** EN: Invokes `torch.linalg.solve_triangular` to advance the surrounding implementation. | CN: 调用 `torch.linalg.solve_triangular` 来推进周围的实现逻辑。
- **L67** EN: Continues `_batch_mahalanobis`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_batch_mahalanobis` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L68** EN: Assigns module-level configuration or cached state to `M`. | CN: 为 `M` 赋予模块级配置或缓存状态。
- **L69** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L70** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L71** EN: Assigns or updates `permuted_M`. | CN: 对 `permuted_M` 进行赋值或更新。
- **L72** EN: Assigns or updates `permute_inv_dims`. | CN: 对 `permute_inv_dims` 进行赋值或更新。
- **L73** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L74** EN: Continues `_batch_mahalanobis`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_batch_mahalanobis` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L75** EN: Assigns or updates `reshaped_M`. | CN: 对 `reshaped_M` 进行赋值或更新。
- **L76** EN: Returns from `_batch_mahalanobis` with the computed result or updated state. | CN: 从 `_batch_mahalanobis` 返回计算结果或更新后的状态。
- **L77** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L78** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 79-102 / 第 79-102 行

````python
0079: def _precision_to_scale_tril(P):
0080:     # Ref: https://nbviewer.jupyter.org/gist/fehiepsi/5ef8e09e61604f10607380467eb82006#Precision-to-scale_tril
0081:     Lf = torch.linalg.cholesky(torch.flip(P, (-2, -1)))
0082:     L_inv = torch.transpose(torch.flip(Lf, (-2, -1)), -2, -1)
0083:     Id = torch.eye(P.shape[-1], dtype=P.dtype, device=P.device)
0084:     L = torch.linalg.solve_triangular(L_inv, Id, upper=False)
0085:     return L
0086: 
0087: 
0088: class MultivariateNormal(Distribution):
0089:     r"""
0090:     Creates a multivariate normal (also called Gaussian) distribution
0091:     parameterized by a mean vector and a covariance matrix.
0092: 
0093:     The multivariate normal distribution can be parameterized either
0094:     in terms of a positive definite covariance matrix :math:`\mathbf{\Sigma}`
0095:     or a positive definite precision matrix :math:`\mathbf{\Sigma}^{-1}`
0096:     or a lower-triangular matrix :math:`\mathbf{L}` with positive-valued
0097:     diagonal entries, such that
0098:     :math:`\mathbf{\Sigma} = \mathbf{L}\mathbf{L}^\top`. This triangular matrix
0099:     can be obtained via e.g. Cholesky decomposition of the covariance.
0100: 
0101:     Example:
0102: 
````

- **L79** EN: Defines function `_precision_to_scale_tril`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_precision_to_scale_tril`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L80** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L81** EN: Assigns or updates `Lf`. | CN: 对 `Lf` 进行赋值或更新。
- **L82** EN: Assigns or updates `L_inv`. | CN: 对 `L_inv` 进行赋值或更新。
- **L83** EN: Assigns or updates `Id`. | CN: 对 `Id` 进行赋值或更新。
- **L84** EN: Assigns module-level configuration or cached state to `L`. | CN: 为 `L` 赋予模块级配置或缓存状态。
- **L85** EN: Returns from `_precision_to_scale_tril` with the computed result or updated state. | CN: 从 `_precision_to_scale_tril` 返回计算结果或更新后的状态。
- **L86** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L87** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L88** EN: Defines class `MultivariateNormal` with bases `Distribution`, which encapsulates distribution parameters, validation rules, and tensor-valued statistical behavior. | CN: 定义类 `MultivariateNormal`，其基类为 `Distribution`，作用是封装分布参数、校验规则以及基于张量的统计行为。
- **L89** EN: Starts the docstring for class `MultivariateNormal`. | CN: 开始为 class `MultivariateNormal` 编写文档字符串。
- **L90** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L91** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L92** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L93** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L94** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L95** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L96** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L97** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L98** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L99** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L100** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L101** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L102** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 103-124 / 第 103-124 行

````python
0103:         >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_LAPACK)
0104:         >>> # xdoctest: +IGNORE_WANT("non-deterministic")
0105:         >>> m = MultivariateNormal(torch.zeros(2), torch.eye(2))
0106:         >>> m.sample()  # normally distributed with mean=`[0,0]` and covariance_matrix=`I`
0107:         tensor([-0.2102, -0.5429])
0108: 
0109:     Args:
0110:         loc (Tensor): mean of the distribution
0111:         covariance_matrix (Tensor): positive-definite covariance matrix
0112:         precision_matrix (Tensor): positive-definite precision matrix
0113:         scale_tril (Tensor): lower-triangular factor of covariance, with positive-valued diagonal
0114: 
0115:     Note:
0116:         Only one of :attr:`covariance_matrix` or :attr:`precision_matrix` or
0117:         :attr:`scale_tril` can be specified.
0118: 
0119:         Using :attr:`scale_tril` will be more efficient: all computations internally
0120:         are based on :attr:`scale_tril`. If :attr:`covariance_matrix` or
0121:         :attr:`precision_matrix` is passed instead, it is only used to compute
0122:         the corresponding lower triangular matrices using a Cholesky decomposition.
0123:     """
0124: 
````

- **L103** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L104** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L105** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L106** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L107** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L108** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L109** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L110** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L111** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L112** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L113** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L114** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L115** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L116** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L117** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L118** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L119** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L120** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L121** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L122** EN: Continues the docstring for class `MultivariateNormal`. | CN: 继续补充 class `MultivariateNormal` 的文档字符串。
- **L123** EN: Ends the docstring for class `MultivariateNormal`. | CN: 结束 class `MultivariateNormal` 的文档字符串。
- **L124** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 125-151 / 第 125-151 行

````python
0125:     # pyrefly: ignore [bad-override]
0126:     arg_constraints = {
0127:         "loc": constraints.real_vector,
0128:         "covariance_matrix": constraints.positive_definite,
0129:         "precision_matrix": constraints.positive_definite,
0130:         "scale_tril": constraints.lower_cholesky,
0131:     }
0132:     support = constraints.real_vector
0133:     has_rsample = True
0134: 
0135:     def __init__(
0136:         self,
0137:         loc: Tensor,
0138:         covariance_matrix: Tensor | None = None,
0139:         precision_matrix: Tensor | None = None,
0140:         scale_tril: Tensor | None = None,
0141:         validate_args: bool | None = None,
0142:     ) -> None:
0143:         if loc.dim() < 1:
0144:             raise ValueError("loc must be at least one-dimensional.")
0145:         if (covariance_matrix is not None) + (scale_tril is not None) + (
0146:             precision_matrix is not None
0147:         ) != 1:
0148:             raise ValueError(
0149:                 "Exactly one of covariance_matrix or precision_matrix or scale_tril may be specified."
0150:             )
0151: 
````

- **L125** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L126** EN: Sets structural metadata on `arg_constraints` that other routines consult later. | CN: 为 `arg_constraints` 设置结构化元数据，供后续例程查询。
- **L127** EN: Continues class `MultivariateNormal`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `MultivariateNormal` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L128** EN: Continues class `MultivariateNormal`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `MultivariateNormal` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L129** EN: Continues class `MultivariateNormal`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `MultivariateNormal` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L130** EN: Continues class `MultivariateNormal`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `MultivariateNormal` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L131** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L132** EN: Sets structural metadata on `support` that other routines consult later. | CN: 为 `support` 设置结构化元数据，供后续例程查询。
- **L133** EN: Assigns or updates `has_rsample`. | CN: 对 `has_rsample` 进行赋值或更新。
- **L134** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L135** EN: Defines function `__init__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__init__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L136** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L137** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L138** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L139** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L140** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L141** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L142** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L143** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L144** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L145** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L146** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L147** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L148** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L149** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L150** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L151** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 152-179 / 第 152-179 行

````python
0152:         if scale_tril is not None:
0153:             if scale_tril.dim() < 2:
0154:                 raise ValueError(
0155:                     "scale_tril matrix must be at least two-dimensional, "
0156:                     "with optional leading batch dimensions"
0157:                 )
0158:             batch_shape = torch.broadcast_shapes(scale_tril.shape[:-2], loc.shape[:-1])
0159:             # pyrefly: ignore [read-only]
0160:             self.scale_tril = scale_tril.expand(batch_shape + (-1, -1))
0161:         elif covariance_matrix is not None:
0162:             if covariance_matrix.dim() < 2:
0163:                 raise ValueError(
0164:                     "covariance_matrix must be at least two-dimensional, "
0165:                     "with optional leading batch dimensions"
0166:                 )
0167:             batch_shape = torch.broadcast_shapes(
0168:                 covariance_matrix.shape[:-2], loc.shape[:-1]
0169:             )
0170:             # pyrefly: ignore [read-only]
0171:             self.covariance_matrix = covariance_matrix.expand(batch_shape + (-1, -1))
0172:         else:
0173:             if precision_matrix is None:
0174:                 raise AssertionError("precision_matrix is unexpectedly None")
0175:             if precision_matrix.dim() < 2:
0176:                 raise ValueError(
0177:                     "precision_matrix must be at least two-dimensional, "
0178:                     "with optional leading batch dimensions"
0179:                 )
````

- **L152** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L153** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L154** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L155** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L156** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L157** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L158** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L159** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L160** EN: Updates object state via `self.scale_tril`. | CN: 通过 `self.scale_tril` 更新对象状态。
- **L161** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L162** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L163** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L164** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L165** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L166** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L167** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L168** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L169** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L170** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L171** EN: Updates object state via `self.covariance_matrix`. | CN: 通过 `self.covariance_matrix` 更新对象状态。
- **L172** EN: Introduces the fallback branch when earlier conditions are not met. | CN: 引入前置条件不满足时的后备分支。
- **L173** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L174** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L175** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L176** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L177** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L178** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L179** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。

### Lines 180-207 / 第 180-207 行

````python
0180:             batch_shape = torch.broadcast_shapes(
0181:                 precision_matrix.shape[:-2], loc.shape[:-1]
0182:             )
0183:             # pyrefly: ignore [read-only]
0184:             self.precision_matrix = precision_matrix.expand(batch_shape + (-1, -1))
0185:         self.loc = loc.expand(batch_shape + (-1,))
0186: 
0187:         event_shape = self.loc.shape[-1:]
0188:         # pyrefly: ignore [bad-argument-type]
0189:         super().__init__(batch_shape, event_shape, validate_args=validate_args)
0190: 
0191:         if scale_tril is not None:
0192:             self._unbroadcasted_scale_tril = scale_tril
0193:         elif covariance_matrix is not None:
0194:             self._unbroadcasted_scale_tril = torch.linalg.cholesky(covariance_matrix)
0195:         else:  # precision_matrix is not None
0196:             self._unbroadcasted_scale_tril = _precision_to_scale_tril(precision_matrix)
0197: 
0198:     def expand(self, batch_shape, _instance=None):
0199:         new = self._get_checked_instance(MultivariateNormal, _instance)
0200:         batch_shape = torch.Size(batch_shape)
0201:         loc_shape = batch_shape + self.event_shape
0202:         cov_shape = batch_shape + self.event_shape + self.event_shape
0203:         new.loc = self.loc.expand(loc_shape)
0204:         new._unbroadcasted_scale_tril = self._unbroadcasted_scale_tril
0205:         if "covariance_matrix" in self.__dict__:
0206:             new.covariance_matrix = self.covariance_matrix.expand(cov_shape)
0207:         if "scale_tril" in self.__dict__:
````

- **L180** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L181** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L182** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L183** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L184** EN: Updates object state via `self.precision_matrix`. | CN: 通过 `self.precision_matrix` 更新对象状态。
- **L185** EN: Updates object state via `self.loc`. | CN: 通过 `self.loc` 更新对象状态。
- **L186** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L187** EN: Assigns or updates `event_shape`. | CN: 对 `event_shape` 进行赋值或更新。
- **L188** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L189** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L190** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L191** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L192** EN: Updates object state via `self._unbroadcasted_scale_tril`. | CN: 通过 `self._unbroadcasted_scale_tril` 更新对象状态。
- **L193** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L194** EN: Updates object state via `self._unbroadcasted_scale_tril`. | CN: 通过 `self._unbroadcasted_scale_tril` 更新对象状态。
- **L195** EN: Continues `MultivariateNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L196** EN: Updates object state via `self._unbroadcasted_scale_tril`. | CN: 通过 `self._unbroadcasted_scale_tril` 更新对象状态。
- **L197** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L198** EN: Defines function `expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 定义函数 `expand`，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L199** EN: Assigns or updates `new`. | CN: 对 `new` 进行赋值或更新。
- **L200** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L201** EN: Assigns or updates `loc_shape`. | CN: 对 `loc_shape` 进行赋值或更新。
- **L202** EN: Assigns or updates `cov_shape`. | CN: 对 `cov_shape` 进行赋值或更新。
- **L203** EN: Updates object state via `new.loc`. | CN: 通过 `new.loc` 更新对象状态。
- **L204** EN: Updates object state via `new._unbroadcasted_scale_tril`. | CN: 通过 `new._unbroadcasted_scale_tril` 更新对象状态。
- **L205** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L206** EN: Updates object state via `new.covariance_matrix`. | CN: 通过 `new.covariance_matrix` 更新对象状态。
- **L207** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。

### Lines 208-234 / 第 208-234 行

````python
0208:             new.scale_tril = self.scale_tril.expand(cov_shape)
0209:         if "precision_matrix" in self.__dict__:
0210:             new.precision_matrix = self.precision_matrix.expand(cov_shape)
0211:         super(MultivariateNormal, new).__init__(
0212:             batch_shape, self.event_shape, validate_args=False
0213:         )
0214:         new._validate_args = self._validate_args
0215:         return new
0216: 
0217:     @lazy_property
0218:     def scale_tril(self) -> Tensor:
0219:         return self._unbroadcasted_scale_tril.expand(
0220:             self._batch_shape + self._event_shape + self._event_shape
0221:         )
0222: 
0223:     @lazy_property
0224:     def covariance_matrix(self) -> Tensor:
0225:         return torch.matmul(
0226:             self._unbroadcasted_scale_tril, self._unbroadcasted_scale_tril.mT
0227:         ).expand(self._batch_shape + self._event_shape + self._event_shape)
0228: 
0229:     @lazy_property
0230:     def precision_matrix(self) -> Tensor:
0231:         return torch.cholesky_inverse(self._unbroadcasted_scale_tril).expand(
0232:             self._batch_shape + self._event_shape + self._event_shape
0233:         )
0234: 
````

- **L208** EN: Updates object state via `new.scale_tril`. | CN: 通过 `new.scale_tril` 更新对象状态。
- **L209** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L210** EN: Updates object state via `new.precision_matrix`. | CN: 通过 `new.precision_matrix` 更新对象状态。
- **L211** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L212** EN: Continues `MultivariateNormal.expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 继续 `MultivariateNormal.expand` 的实现，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L213** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L214** EN: Updates object state via `new._validate_args`. | CN: 通过 `new._validate_args` 更新对象状态。
- **L215** EN: Returns from `MultivariateNormal.expand` with the computed result or updated state. | CN: 从 `MultivariateNormal.expand` 返回计算结果或更新后的状态。
- **L216** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L217** EN: Applies decorator `lazy_property`, which turns a derived value into a lazily cached attribute. | CN: 应用装饰器 `lazy_property`，其作用是把派生值变成惰性缓存属性。
- **L218** EN: Defines function `scale_tril`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `scale_tril`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L219** EN: Returns from `MultivariateNormal.scale_tril` with the computed result or updated state. | CN: 从 `MultivariateNormal.scale_tril` 返回计算结果或更新后的状态。
- **L220** EN: Continues `MultivariateNormal.scale_tril`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.scale_tril` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L221** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L222** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L223** EN: Applies decorator `lazy_property`, which turns a derived value into a lazily cached attribute. | CN: 应用装饰器 `lazy_property`，其作用是把派生值变成惰性缓存属性。
- **L224** EN: Defines function `covariance_matrix`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `covariance_matrix`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L225** EN: Returns from `MultivariateNormal.covariance_matrix` with the computed result or updated state. | CN: 从 `MultivariateNormal.covariance_matrix` 返回计算结果或更新后的状态。
- **L226** EN: Continues `MultivariateNormal.covariance_matrix`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.covariance_matrix` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L227** EN: Invokes `expand` to advance the surrounding implementation. | CN: 调用 `expand` 来推进周围的实现逻辑。
- **L228** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L229** EN: Applies decorator `lazy_property`, which turns a derived value into a lazily cached attribute. | CN: 应用装饰器 `lazy_property`，其作用是把派生值变成惰性缓存属性。
- **L230** EN: Defines function `precision_matrix`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `precision_matrix`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L231** EN: Returns from `MultivariateNormal.precision_matrix` with the computed result or updated state. | CN: 从 `MultivariateNormal.precision_matrix` 返回计算结果或更新后的状态。
- **L232** EN: Continues `MultivariateNormal.precision_matrix`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `MultivariateNormal.precision_matrix` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L233** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L234** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 235-255 / 第 235-255 行

````python
0235:     @property
0236:     def mean(self) -> Tensor:
0237:         return self.loc
0238: 
0239:     @property
0240:     def mode(self) -> Tensor:
0241:         return self.loc
0242: 
0243:     @property
0244:     def variance(self) -> Tensor:
0245:         return (
0246:             self._unbroadcasted_scale_tril.pow(2)
0247:             .sum(-1)
0248:             .expand(self._batch_shape + self._event_shape)
0249:         )
0250: 
0251:     def rsample(self, sample_shape: _size = torch.Size()) -> Tensor:
0252:         shape = self._extended_shape(sample_shape)
0253:         eps = _standard_normal(shape, dtype=self.loc.dtype, device=self.loc.device)
0254:         return self.loc + _batch_mv(self._unbroadcasted_scale_tril, eps)
0255: 
````

- **L235** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L236** EN: Defines function `mean`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mean`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L237** EN: Returns from `MultivariateNormal.mean` with the computed result or updated state. | CN: 从 `MultivariateNormal.mean` 返回计算结果或更新后的状态。
- **L238** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L239** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L240** EN: Defines function `mode`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mode`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L241** EN: Returns from `MultivariateNormal.mode` with the computed result or updated state. | CN: 从 `MultivariateNormal.mode` 返回计算结果或更新后的状态。
- **L242** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L243** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L244** EN: Defines function `variance`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `variance`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L245** EN: Returns from `MultivariateNormal.variance` with the computed result or updated state. | CN: 从 `MultivariateNormal.variance` 返回计算结果或更新后的状态。
- **L246** EN: Invokes `self._unbroadcasted_scale_tril.pow` to advance the surrounding implementation. | CN: 调用 `self._unbroadcasted_scale_tril.pow` 来推进周围的实现逻辑。
- **L247** EN: Invokes `sum` to advance the surrounding implementation. | CN: 调用 `sum` 来推进周围的实现逻辑。
- **L248** EN: Invokes `expand` to advance the surrounding implementation. | CN: 调用 `expand` 来推进周围的实现逻辑。
- **L249** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L250** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L251** EN: Defines function `rsample`, which produces samples according to the modeled stochastic behavior. | CN: 定义函数 `rsample`，其作用是根据建模的随机行为生成样本。
- **L252** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L253** EN: Assigns or updates `eps`. | CN: 对 `eps` 进行赋值或更新。
- **L254** EN: Returns from `MultivariateNormal.rsample` with the computed result or updated state. | CN: 从 `MultivariateNormal.rsample` 返回计算结果或更新后的状态。
- **L255** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 256-274 / 第 256-274 行

````python
0256:     def log_prob(self, value):
0257:         if self._validate_args:
0258:             self._validate_sample(value)
0259:         diff = value - self.loc
0260:         M = _batch_mahalanobis(self._unbroadcasted_scale_tril, diff)
0261:         half_log_det = (
0262:             self._unbroadcasted_scale_tril.diagonal(dim1=-2, dim2=-1).log().sum(-1)
0263:         )
0264:         return -0.5 * (self._event_shape[0] * math.log(2 * math.pi) + M) - half_log_det
0265: 
0266:     def entropy(self):
0267:         half_log_det = (
0268:             self._unbroadcasted_scale_tril.diagonal(dim1=-2, dim2=-1).log().sum(-1)
0269:         )
0270:         H = 0.5 * self._event_shape[0] * (1.0 + math.log(2 * math.pi)) + half_log_det
0271:         if len(self._batch_shape) == 0:
0272:             return H
0273:         else:
0274:             return H.expand(self._batch_shape)
````

- **L256** EN: Defines function `log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 定义函数 `log_prob`，其作用是为给定输入计算对数概率或类似密度分数。
- **L257** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L258** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L259** EN: Assigns or updates `diff`. | CN: 对 `diff` 进行赋值或更新。
- **L260** EN: Assigns module-level configuration or cached state to `M`. | CN: 为 `M` 赋予模块级配置或缓存状态。
- **L261** EN: Assigns or updates `half_log_det`. | CN: 对 `half_log_det` 进行赋值或更新。
- **L262** EN: Invokes `self._unbroadcasted_scale_tril.diagonal` to advance the surrounding implementation. | CN: 调用 `self._unbroadcasted_scale_tril.diagonal` 来推进周围的实现逻辑。
- **L263** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L264** EN: Returns from `MultivariateNormal.log_prob` with the computed result or updated state. | CN: 从 `MultivariateNormal.log_prob` 返回计算结果或更新后的状态。
- **L265** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L266** EN: Defines function `entropy`, which computes an entropy-style summary of uncertainty. | CN: 定义函数 `entropy`，其作用是计算描述不确定性的熵类指标。
- **L267** EN: Assigns or updates `half_log_det`. | CN: 对 `half_log_det` 进行赋值或更新。
- **L268** EN: Invokes `self._unbroadcasted_scale_tril.diagonal` to advance the surrounding implementation. | CN: 调用 `self._unbroadcasted_scale_tril.diagonal` 来推进周围的实现逻辑。
- **L269** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L270** EN: Assigns module-level configuration or cached state to `H`. | CN: 为 `H` 赋予模块级配置或缓存状态。
- **L271** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L272** EN: Returns from `MultivariateNormal.entropy` with the computed result or updated state. | CN: 从 `MultivariateNormal.entropy` 返回计算结果或更新后的状态。
- **L273** EN: Introduces the fallback branch when earlier conditions are not met. | CN: 引入前置条件不满足时的后备分支。
- **L274** EN: Returns from `MultivariateNormal.entropy` with the computed result or updated state. | CN: 从 `MultivariateNormal.entropy` 返回计算结果或更新后的状态。

## Key Concepts / 关键概念

- **EN**: Probability parameterization — The code manages parameters such as probabilities, logits, concentration values, or base measures.
  **CN**: Probability parameterization——代码管理概率、logits、浓度参数或基测度等分布参数。
- **EN**: Sampling and statistics — The implementation usually defines sampling, moments, support, or likelihood-related routines.
  **CN**: Sampling and statistics——实现通常会定义采样、矩、支持集或似然相关例程。
- **EN**: Shape and support rules — Broadcasting rules and support constraints keep tensor-valued distributions mathematically valid.
  **CN**: Shape and support rules——广播规则与支持集约束用于保证张量分布在数学上有效。
- **EN**: Constraints — Constraint objects or registries encode validity rules for shapes, values, or supports.
  **CN**: Constraints——约束对象或注册表编码了针对形状、取值或支持集的有效性规则。
- **EN**: Distribution base classes — The file participates in the common abstractions shared by many probability distributions.
  **CN**: Distribution base classes——该文件参与多个概率分布共享的公共抽象。
- **EN**: Sampling — Sampling routines generate representative values from modeled behavior.
  **CN**: Sampling——采样例程会根据建模行为生成代表性值。

## Dependencies / 依赖关系

- **Torch imports / Torch 导入**: `torch`、`torch:Tensor`、`torch.distributions:constraints`、`torch.distributions.distribution:Distribution`、`torch.distributions.utils:_standard_normal, lazy_property`、`torch.types:_size`
- **Other imports / 其他导入**: `math`
- **Top-level classes / 顶层类**: `MultivariateNormal`
- **Top-level functions / 顶层函数**: `_batch_mv`、`_batch_mahalanobis`、`_precision_to_scale_tril`
- **Base classes / 基类**: `Distribution`
- **Decorators / 装饰器**: 无
- **Module assignments / 模块级赋值**: `__all__`
