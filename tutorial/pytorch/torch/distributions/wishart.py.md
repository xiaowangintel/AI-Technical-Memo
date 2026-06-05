# wishart.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/distributions/wishart.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements the `Wishart` distribution or helper type, including parameter handling, sampling, and statistical methods.
- **Purpose (CN)**: 实现 `Wishart` 分布或辅助类型，涵盖参数处理、采样与统计方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27 / 第 1-27 行

````python
0001: # mypy: allow-untyped-defs
0002: import math
0003: import warnings
0004: 
0005: import torch
0006: from torch import nan, Tensor
0007: from torch.distributions import constraints
0008: from torch.distributions.exp_family import ExponentialFamily
0009: from torch.distributions.multivariate_normal import _precision_to_scale_tril
0010: from torch.distributions.utils import lazy_property
0011: from torch.types import _Number, _size, Number
0012: 
0013: 
0014: __all__ = ["Wishart"]
0015: 
0016: _log_2 = math.log(2)
0017: 
0018: 
0019: def _mvdigamma(x: Tensor, p: int) -> Tensor:
0020:     if not x.gt((p - 1) / 2).all():
0021:         raise AssertionError("Wrong domain for multivariate digamma function.")
0022:     return torch.digamma(
0023:         x.unsqueeze(-1)
0024:         - torch.arange(p, dtype=x.dtype, device=x.device).div(2).expand(x.shape + (-1,))
0025:     ).sum(-1)
0026: 
0027: 
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Imports module dependencies: `math`. | CN: 导入模块依赖：`math`。
- **L3** EN: Imports module dependencies: `warnings`. | CN: 导入模块依赖：`warnings`。
- **L4** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L5** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L6** EN: Imports `nan, Tensor` from `torch` so later code can reuse those definitions. | CN: 从 `torch` 导入 `nan, Tensor`，供后续代码复用这些定义。
- **L7** EN: Imports `constraints` from `torch.distributions` so later code can reuse those definitions. | CN: 从 `torch.distributions` 导入 `constraints`，供后续代码复用这些定义。
- **L8** EN: Imports `ExponentialFamily` from `torch.distributions.exp_family` so later code can reuse those definitions. | CN: 从 `torch.distributions.exp_family` 导入 `ExponentialFamily`，供后续代码复用这些定义。
- **L9** EN: Imports `_precision_to_scale_tril` from `torch.distributions.multivariate_normal` so later code can reuse those definitions. | CN: 从 `torch.distributions.multivariate_normal` 导入 `_precision_to_scale_tril`，供后续代码复用这些定义。
- **L10** EN: Imports `lazy_property` from `torch.distributions.utils` so later code can reuse those definitions. | CN: 从 `torch.distributions.utils` 导入 `lazy_property`，供后续代码复用这些定义。
- **L11** EN: Imports `_Number, _size, Number` from `torch.types` so later code can reuse those definitions. | CN: 从 `torch.types` 导入 `_Number, _size, Number`，供后续代码复用这些定义。
- **L12** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L13** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L14** EN: Defines or updates the module’s public export list. | CN: 定义或更新模块的公开导出列表。
- **L15** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L16** EN: Assigns module-level configuration or cached state to `_log_2`. | CN: 为 `_log_2` 赋予模块级配置或缓存状态。
- **L17** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L18** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L19** EN: Defines function `_mvdigamma`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_mvdigamma`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L20** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L21** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L22** EN: Returns from `_mvdigamma` with the computed result or updated state. | CN: 从 `_mvdigamma` 返回计算结果或更新后的状态。
- **L23** EN: Invokes `x.unsqueeze` to advance the surrounding implementation. | CN: 调用 `x.unsqueeze` 来推进周围的实现逻辑。
- **L24** EN: Invokes `torch.arange` to advance the surrounding implementation. | CN: 调用 `torch.arange` 来推进周围的实现逻辑。
- **L25** EN: Invokes `sum` to advance the surrounding implementation. | CN: 调用 `sum` 来推进周围的实现逻辑。
- **L26** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L27** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 28-55 / 第 28-55 行

````python
0028: def _clamp_above_eps(x: Tensor) -> Tensor:
0029:     # We assume positive input for this function
0030:     return x.clamp(min=torch.finfo(x.dtype).eps)
0031: 
0032: 
0033: class Wishart(ExponentialFamily):
0034:     r"""
0035:     Creates a Wishart distribution parameterized by a symmetric positive definite matrix :math:`\Sigma`,
0036:     or its Cholesky decomposition :math:`\mathbf{\Sigma} = \mathbf{L}\mathbf{L}^\top`
0037: 
0038:     Example:
0039:         >>> # xdoctest: +SKIP("FIXME: scale_tril must be at least two-dimensional")
0040:         >>> m = Wishart(torch.Tensor([2]), covariance_matrix=torch.eye(2))
0041:         >>> m.sample()  # Wishart distributed with mean=`df * I` and
0042:         >>> # variance(x_ij)=`df` for i != j and variance(x_ij)=`2 * df` for i == j
0043: 
0044:     Args:
0045:         df (float or Tensor): real-valued parameter larger than the (dimension of Square matrix) - 1
0046:         covariance_matrix (Tensor): positive-definite covariance matrix
0047:         precision_matrix (Tensor): positive-definite precision matrix
0048:         scale_tril (Tensor): lower-triangular factor of covariance, with positive-valued diagonal
0049:     Note:
0050:         Only one of :attr:`covariance_matrix` or :attr:`precision_matrix` or
0051:         :attr:`scale_tril` can be specified.
0052:         Using :attr:`scale_tril` will be more efficient: all computations internally
0053:         are based on :attr:`scale_tril`. If :attr:`covariance_matrix` or
0054:         :attr:`precision_matrix` is passed instead, it is only used to compute
0055:         the corresponding lower triangular matrices using a Cholesky decomposition.
````

- **L28** EN: Defines function `_clamp_above_eps`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_clamp_above_eps`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L29** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L30** EN: Returns from `_clamp_above_eps` with the computed result or updated state. | CN: 从 `_clamp_above_eps` 返回计算结果或更新后的状态。
- **L31** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L32** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L33** EN: Defines class `Wishart` with bases `ExponentialFamily`, which encapsulates distribution parameters, validation rules, and tensor-valued statistical behavior. | CN: 定义类 `Wishart`，其基类为 `ExponentialFamily`，作用是封装分布参数、校验规则以及基于张量的统计行为。
- **L34** EN: Starts the docstring for class `Wishart`. | CN: 开始为 class `Wishart` 编写文档字符串。
- **L35** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L36** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L37** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L38** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L39** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L40** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L41** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L42** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L43** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L44** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L45** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L46** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L47** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L48** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L49** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L50** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L51** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L52** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L53** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L54** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L55** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。

### Lines 56-79 / 第 56-79 行

````python
0056:         'torch.distributions.LKJCholesky' is a restricted Wishart distribution.[1]
0057: 
0058:     **References**
0059: 
0060:     [1] Wang, Z., Wu, Y. and Chu, H., 2018. `On equivalence of the LKJ distribution and the restricted Wishart distribution`.
0061:     [2] Sawyer, S., 2007. `Wishart Distributions and Inverse-Wishart Sampling`.
0062:     [3] Anderson, T. W., 2003. `An Introduction to Multivariate Statistical Analysis (3rd ed.)`.
0063:     [4] Odell, P. L. & Feiveson, A. H., 1966. `A Numerical Procedure to Generate a SampleCovariance Matrix`. JASA, 61(313):199-203.
0064:     [5] Ku, Y.-C. & Bloomfield, P., 2010. `Generating Random Wishart Matrices with Fractional Degrees of Freedom in OX`.
0065:     """
0066: 
0067:     support = constraints.positive_definite
0068:     has_rsample = True
0069:     _mean_carrier_measure = 0
0070: 
0071:     @property
0072:     def arg_constraints(self):
0073:         return {
0074:             "covariance_matrix": constraints.positive_definite,
0075:             "precision_matrix": constraints.positive_definite,
0076:             "scale_tril": constraints.lower_cholesky,
0077:             "df": constraints.greater_than(self.event_shape[-1] - 1),
0078:         }
0079: 
````

- **L56** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L57** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L58** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L59** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L60** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L61** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L62** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L63** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L64** EN: Continues the docstring for class `Wishart`. | CN: 继续补充 class `Wishart` 的文档字符串。
- **L65** EN: Ends the docstring for class `Wishart`. | CN: 结束 class `Wishart` 的文档字符串。
- **L66** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L67** EN: Sets structural metadata on `support` that other routines consult later. | CN: 为 `support` 设置结构化元数据，供后续例程查询。
- **L68** EN: Assigns or updates `has_rsample`. | CN: 对 `has_rsample` 进行赋值或更新。
- **L69** EN: Assigns module-level configuration or cached state to `_mean_carrier_measure`. | CN: 为 `_mean_carrier_measure` 赋予模块级配置或缓存状态。
- **L70** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L71** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L72** EN: Defines function `arg_constraints`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `arg_constraints`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L73** EN: Returns from `Wishart.arg_constraints` with the computed result or updated state. | CN: 从 `Wishart.arg_constraints` 返回计算结果或更新后的状态。
- **L74** EN: Continues `Wishart.arg_constraints`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.arg_constraints` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L75** EN: Continues `Wishart.arg_constraints`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.arg_constraints` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L76** EN: Continues `Wishart.arg_constraints`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.arg_constraints` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L77** EN: Invokes `constraints.greater_than` to advance the surrounding implementation. | CN: 调用 `constraints.greater_than` 来推进周围的实现逻辑。
- **L78** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L79** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 80-107 / 第 80-107 行

````python
0080:     def __init__(
0081:         self,
0082:         df: Tensor | Number,
0083:         covariance_matrix: Tensor | None = None,
0084:         precision_matrix: Tensor | None = None,
0085:         scale_tril: Tensor | None = None,
0086:         validate_args: bool | None = None,
0087:     ) -> None:
0088:         if (
0089:             (covariance_matrix is not None)
0090:             + (scale_tril is not None)
0091:             + (precision_matrix is not None)
0092:         ) != 1:
0093:             raise AssertionError(
0094:                 "Exactly one of covariance_matrix or precision_matrix or scale_tril may be specified."
0095:             )
0096: 
0097:         param = next(
0098:             p
0099:             for p in (covariance_matrix, precision_matrix, scale_tril)
0100:             if p is not None
0101:         )
0102: 
0103:         if param.dim() < 2:
0104:             raise ValueError(
0105:                 "scale_tril must be at least two-dimensional, with optional leading batch dimensions"
0106:             )
0107: 
````

- **L80** EN: Defines function `__init__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__init__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L81** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L82** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L83** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L84** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L85** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L86** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L87** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L88** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L89** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L90** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L91** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L92** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L93** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L94** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L95** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L96** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L97** EN: Assigns or updates `param`. | CN: 对 `param` 进行赋值或更新。
- **L98** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L99** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L100** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L101** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L102** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L103** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L104** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L105** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L106** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L107** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 108-130 / 第 108-130 行

````python
0108:         if isinstance(df, _Number):
0109:             batch_shape = torch.Size(param.shape[:-2])
0110:             self.df = torch.tensor(df, dtype=param.dtype, device=param.device)
0111:         else:
0112:             batch_shape = torch.broadcast_shapes(param.shape[:-2], df.shape)
0113:             self.df = df.expand(batch_shape)
0114:         event_shape = param.shape[-2:]
0115: 
0116:         if self.df.le(event_shape[-1] - 1).any():
0117:             raise ValueError(
0118:                 f"Value of df={df} expected to be greater than ndim - 1 = {event_shape[-1] - 1}."
0119:             )
0120: 
0121:         if scale_tril is not None:
0122:             # pyrefly: ignore [read-only]
0123:             self.scale_tril = param.expand(batch_shape + (-1, -1))
0124:         elif covariance_matrix is not None:
0125:             # pyrefly: ignore [read-only]
0126:             self.covariance_matrix = param.expand(batch_shape + (-1, -1))
0127:         elif precision_matrix is not None:
0128:             # pyrefly: ignore [read-only]
0129:             self.precision_matrix = param.expand(batch_shape + (-1, -1))
0130: 
````

- **L108** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L109** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L110** EN: Updates object state via `self.df`. | CN: 通过 `self.df` 更新对象状态。
- **L111** EN: Introduces the fallback branch when earlier conditions are not met. | CN: 引入前置条件不满足时的后备分支。
- **L112** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L113** EN: Updates object state via `self.df`. | CN: 通过 `self.df` 更新对象状态。
- **L114** EN: Assigns or updates `event_shape`. | CN: 对 `event_shape` 进行赋值或更新。
- **L115** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L116** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L117** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L118** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L119** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L120** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L121** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L122** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L123** EN: Updates object state via `self.scale_tril`. | CN: 通过 `self.scale_tril` 更新对象状态。
- **L124** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L125** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L126** EN: Updates object state via `self.covariance_matrix`. | CN: 通过 `self.covariance_matrix` 更新对象状态。
- **L127** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L128** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L129** EN: Updates object state via `self.precision_matrix`. | CN: 通过 `self.precision_matrix` 更新对象状态。
- **L130** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 131-158 / 第 131-158 行

````python
0131:         if self.df.lt(event_shape[-1]).any():
0132:             warnings.warn(
0133:                 "Low df values detected. Singular samples are highly likely to occur for ndim - 1 < df < ndim.",
0134:                 stacklevel=2,
0135:             )
0136: 
0137:         # pyrefly: ignore [bad-argument-type]
0138:         super().__init__(batch_shape, event_shape, validate_args=validate_args)
0139:         self._batch_dims = [-(x + 1) for x in range(len(self._batch_shape))]
0140: 
0141:         if scale_tril is not None:
0142:             self._unbroadcasted_scale_tril = scale_tril
0143:         elif covariance_matrix is not None:
0144:             self._unbroadcasted_scale_tril = torch.linalg.cholesky(covariance_matrix)
0145:         else:  # precision_matrix is not None
0146:             self._unbroadcasted_scale_tril = _precision_to_scale_tril(precision_matrix)
0147: 
0148:         # Chi2 distribution is needed for Bartlett decomposition sampling
0149:         self._dist_chi2 = torch.distributions.chi2.Chi2(
0150:             df=(
0151:                 self.df.unsqueeze(-1)
0152:                 - torch.arange(
0153:                     self._event_shape[-1],
0154:                     dtype=self._unbroadcasted_scale_tril.dtype,
0155:                     device=self._unbroadcasted_scale_tril.device,
0156:                 ).expand(batch_shape + (-1,))
0157:             )
0158:         )
````

- **L131** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L132** EN: Invokes `warnings.warn` to advance the surrounding implementation. | CN: 调用 `warnings.warn` 来推进周围的实现逻辑。
- **L133** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L134** EN: Assigns or updates `stacklevel`. | CN: 对 `stacklevel` 进行赋值或更新。
- **L135** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L136** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L137** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L138** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L139** EN: Updates object state via `self._batch_dims`. | CN: 通过 `self._batch_dims` 更新对象状态。
- **L140** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L141** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L142** EN: Updates object state via `self._unbroadcasted_scale_tril`. | CN: 通过 `self._unbroadcasted_scale_tril` 更新对象状态。
- **L143** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L144** EN: Updates object state via `self._unbroadcasted_scale_tril`. | CN: 通过 `self._unbroadcasted_scale_tril` 更新对象状态。
- **L145** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L146** EN: Updates object state via `self._unbroadcasted_scale_tril`. | CN: 通过 `self._unbroadcasted_scale_tril` 更新对象状态。
- **L147** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L148** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L149** EN: Updates object state via `self._dist_chi2`. | CN: 通过 `self._dist_chi2` 更新对象状态。
- **L150** EN: Assigns or updates `df`. | CN: 对 `df` 进行赋值或更新。
- **L151** EN: Invokes `self.df.unsqueeze` to advance the surrounding implementation. | CN: 调用 `self.df.unsqueeze` 来推进周围的实现逻辑。
- **L152** EN: Invokes `torch.arange` to advance the surrounding implementation. | CN: 调用 `torch.arange` 来推进周围的实现逻辑。
- **L153** EN: Continues `Wishart.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L154** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L155** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L156** EN: Invokes `expand` to advance the surrounding implementation. | CN: 调用 `expand` 来推进周围的实现逻辑。
- **L157** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L158** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。

### Lines 159-186 / 第 159-186 行

````python
0159: 
0160:     def expand(self, batch_shape, _instance=None):
0161:         new = self._get_checked_instance(Wishart, _instance)
0162:         batch_shape = torch.Size(batch_shape)
0163:         cov_shape = batch_shape + self.event_shape
0164:         new._unbroadcasted_scale_tril = self._unbroadcasted_scale_tril.expand(cov_shape)
0165:         new.df = self.df.expand(batch_shape)
0166: 
0167:         new._batch_dims = [-(x + 1) for x in range(len(batch_shape))]
0168: 
0169:         if "covariance_matrix" in self.__dict__:
0170:             new.covariance_matrix = self.covariance_matrix.expand(cov_shape)
0171:         if "scale_tril" in self.__dict__:
0172:             new.scale_tril = self.scale_tril.expand(cov_shape)
0173:         if "precision_matrix" in self.__dict__:
0174:             new.precision_matrix = self.precision_matrix.expand(cov_shape)
0175: 
0176:         # Chi2 distribution is needed for Bartlett decomposition sampling
0177:         new._dist_chi2 = torch.distributions.chi2.Chi2(
0178:             df=(
0179:                 new.df.unsqueeze(-1)
0180:                 - torch.arange(
0181:                     self.event_shape[-1],
0182:                     dtype=new._unbroadcasted_scale_tril.dtype,
0183:                     device=new._unbroadcasted_scale_tril.device,
0184:                 ).expand(batch_shape + (-1,))
0185:             )
0186:         )
````

- **L159** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L160** EN: Defines function `expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 定义函数 `expand`，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L161** EN: Assigns or updates `new`. | CN: 对 `new` 进行赋值或更新。
- **L162** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L163** EN: Assigns or updates `cov_shape`. | CN: 对 `cov_shape` 进行赋值或更新。
- **L164** EN: Updates object state via `new._unbroadcasted_scale_tril`. | CN: 通过 `new._unbroadcasted_scale_tril` 更新对象状态。
- **L165** EN: Updates object state via `new.df`. | CN: 通过 `new.df` 更新对象状态。
- **L166** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L167** EN: Updates object state via `new._batch_dims`. | CN: 通过 `new._batch_dims` 更新对象状态。
- **L168** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L169** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L170** EN: Updates object state via `new.covariance_matrix`. | CN: 通过 `new.covariance_matrix` 更新对象状态。
- **L171** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L172** EN: Updates object state via `new.scale_tril`. | CN: 通过 `new.scale_tril` 更新对象状态。
- **L173** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L174** EN: Updates object state via `new.precision_matrix`. | CN: 通过 `new.precision_matrix` 更新对象状态。
- **L175** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L176** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L177** EN: Updates object state via `new._dist_chi2`. | CN: 通过 `new._dist_chi2` 更新对象状态。
- **L178** EN: Assigns or updates `df`. | CN: 对 `df` 进行赋值或更新。
- **L179** EN: Invokes `new.df.unsqueeze` to advance the surrounding implementation. | CN: 调用 `new.df.unsqueeze` 来推进周围的实现逻辑。
- **L180** EN: Invokes `torch.arange` to advance the surrounding implementation. | CN: 调用 `torch.arange` 来推进周围的实现逻辑。
- **L181** EN: Continues `Wishart.expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 继续 `Wishart.expand` 的实现，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L182** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L183** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L184** EN: Invokes `expand` to advance the surrounding implementation. | CN: 调用 `expand` 来推进周围的实现逻辑。
- **L185** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L186** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。

### Lines 187-214 / 第 187-214 行

````python
0187: 
0188:         super(Wishart, new).__init__(batch_shape, self.event_shape, validate_args=False)
0189:         new._validate_args = self._validate_args
0190:         return new
0191: 
0192:     @lazy_property
0193:     def scale_tril(self) -> Tensor:
0194:         return self._unbroadcasted_scale_tril.expand(
0195:             self._batch_shape + self._event_shape
0196:         )
0197: 
0198:     @lazy_property
0199:     def covariance_matrix(self) -> Tensor:
0200:         return (
0201:             self._unbroadcasted_scale_tril
0202:             @ self._unbroadcasted_scale_tril.transpose(-2, -1)
0203:         ).expand(self._batch_shape + self._event_shape)
0204: 
0205:     @lazy_property
0206:     def precision_matrix(self) -> Tensor:
0207:         identity = torch.eye(
0208:             self._event_shape[-1],
0209:             device=self._unbroadcasted_scale_tril.device,
0210:             dtype=self._unbroadcasted_scale_tril.dtype,
0211:         )
0212:         return torch.cholesky_solve(identity, self._unbroadcasted_scale_tril).expand(
0213:             self._batch_shape + self._event_shape
0214:         )
````

- **L187** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L188** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L189** EN: Updates object state via `new._validate_args`. | CN: 通过 `new._validate_args` 更新对象状态。
- **L190** EN: Returns from `Wishart.expand` with the computed result or updated state. | CN: 从 `Wishart.expand` 返回计算结果或更新后的状态。
- **L191** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L192** EN: Applies decorator `lazy_property`, which turns a derived value into a lazily cached attribute. | CN: 应用装饰器 `lazy_property`，其作用是把派生值变成惰性缓存属性。
- **L193** EN: Defines function `scale_tril`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `scale_tril`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L194** EN: Returns from `Wishart.scale_tril` with the computed result or updated state. | CN: 从 `Wishart.scale_tril` 返回计算结果或更新后的状态。
- **L195** EN: Continues `Wishart.scale_tril`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.scale_tril` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L196** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L197** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L198** EN: Applies decorator `lazy_property`, which turns a derived value into a lazily cached attribute. | CN: 应用装饰器 `lazy_property`，其作用是把派生值变成惰性缓存属性。
- **L199** EN: Defines function `covariance_matrix`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `covariance_matrix`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L200** EN: Returns from `Wishart.covariance_matrix` with the computed result or updated state. | CN: 从 `Wishart.covariance_matrix` 返回计算结果或更新后的状态。
- **L201** EN: Continues `Wishart.covariance_matrix`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.covariance_matrix` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L202** EN: Applies decorator `self._unbroadcasted_scale_tril.transpose`, which modifies the behavior of the following definition. | CN: 应用装饰器 `self._unbroadcasted_scale_tril.transpose`，其作用是修改后续定义的行为。
- **L203** EN: Invokes `expand` to advance the surrounding implementation. | CN: 调用 `expand` 来推进周围的实现逻辑。
- **L204** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L205** EN: Applies decorator `lazy_property`, which turns a derived value into a lazily cached attribute. | CN: 应用装饰器 `lazy_property`，其作用是把派生值变成惰性缓存属性。
- **L206** EN: Defines function `precision_matrix`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `precision_matrix`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L207** EN: Assigns or updates `identity`. | CN: 对 `identity` 进行赋值或更新。
- **L208** EN: Continues `Wishart.precision_matrix`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.precision_matrix` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L209** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L210** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L211** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L212** EN: Returns from `Wishart.precision_matrix` with the computed result or updated state. | CN: 从 `Wishart.precision_matrix` 返回计算结果或更新后的状态。
- **L213** EN: Continues `Wishart.precision_matrix`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.precision_matrix` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L214** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。

### Lines 215-241 / 第 215-241 行

````python
0215: 
0216:     @property
0217:     def mean(self) -> Tensor:
0218:         return self.df.view(self._batch_shape + (1, 1)) * self.covariance_matrix
0219: 
0220:     @property
0221:     def mode(self) -> Tensor:
0222:         factor = self.df - self.covariance_matrix.shape[-1] - 1
0223:         factor[factor <= 0] = nan
0224:         return factor.view(self._batch_shape + (1, 1)) * self.covariance_matrix
0225: 
0226:     @property
0227:     def variance(self) -> Tensor:
0228:         V = self.covariance_matrix  # has shape (batch_shape x event_shape)
0229:         diag_V = V.diagonal(dim1=-2, dim2=-1)
0230:         return self.df.view(self._batch_shape + (1, 1)) * (
0231:             V.pow(2) + torch.einsum("...i,...j->...ij", diag_V, diag_V)
0232:         )
0233: 
0234:     def _bartlett_sampling(self, sample_shape=torch.Size()):
0235:         p = self._event_shape[-1]  # has singleton shape
0236: 
0237:         # Implemented Sampling using Bartlett decomposition
0238:         noise = _clamp_above_eps(
0239:             self._dist_chi2.rsample(sample_shape).sqrt()
0240:         ).diag_embed(dim1=-2, dim2=-1)
0241: 
````

- **L215** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L216** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L217** EN: Defines function `mean`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mean`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L218** EN: Returns from `Wishart.mean` with the computed result or updated state. | CN: 从 `Wishart.mean` 返回计算结果或更新后的状态。
- **L219** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L220** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L221** EN: Defines function `mode`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mode`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L222** EN: Assigns or updates `factor`. | CN: 对 `factor` 进行赋值或更新。
- **L223** EN: Continues `Wishart.mode`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Wishart.mode` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L224** EN: Returns from `Wishart.mode` with the computed result or updated state. | CN: 从 `Wishart.mode` 返回计算结果或更新后的状态。
- **L225** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L226** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L227** EN: Defines function `variance`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `variance`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L228** EN: Assigns module-level configuration or cached state to `V`. | CN: 为 `V` 赋予模块级配置或缓存状态。
- **L229** EN: Assigns or updates `diag_V`. | CN: 对 `diag_V` 进行赋值或更新。
- **L230** EN: Returns from `Wishart.variance` with the computed result or updated state. | CN: 从 `Wishart.variance` 返回计算结果或更新后的状态。
- **L231** EN: Invokes `V.pow` to advance the surrounding implementation. | CN: 调用 `V.pow` 来推进周围的实现逻辑。
- **L232** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L233** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L234** EN: Defines function `_bartlett_sampling`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_bartlett_sampling`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L235** EN: Assigns or updates `p`. | CN: 对 `p` 进行赋值或更新。
- **L236** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L237** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L238** EN: Assigns or updates `noise`. | CN: 对 `noise` 进行赋值或更新。
- **L239** EN: Invokes `self._dist_chi2.rsample` to advance the surrounding implementation. | CN: 调用 `self._dist_chi2.rsample` 来推进周围的实现逻辑。
- **L240** EN: Invokes `diag_embed` to advance the surrounding implementation. | CN: 调用 `diag_embed` 来推进周围的实现逻辑。
- **L241** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 242-268 / 第 242-268 行

````python
0242:         i, j = torch.tril_indices(p, p, offset=-1)
0243:         noise[..., i, j] = torch.randn(
0244:             torch.Size(sample_shape) + self._batch_shape + (int(p * (p - 1) / 2),),
0245:             dtype=noise.dtype,
0246:             device=noise.device,
0247:         )
0248:         chol = self._unbroadcasted_scale_tril @ noise
0249:         return chol @ chol.transpose(-2, -1)
0250: 
0251:     def rsample(
0252:         self, sample_shape: _size = torch.Size(), max_try_correction=None
0253:     ) -> Tensor:
0254:         r"""
0255:         .. warning::
0256:             In some cases, sampling algorithm based on Bartlett decomposition may return singular matrix samples.
0257:             Several tries to correct singular samples are performed by default, but it may end up returning
0258:             singular matrix samples. Singular samples may return `-inf` values in `.log_prob()`.
0259:             In those cases, the user should validate the samples and either fix the value of `df`
0260:             or adjust `max_try_correction` value for argument in `.rsample` accordingly.
0261:         """
0262: 
0263:         if max_try_correction is None:
0264:             max_try_correction = 3 if torch._C._get_tracing_state() else 10
0265: 
0266:         sample_shape = torch.Size(sample_shape)
0267:         sample = self._bartlett_sampling(sample_shape)
0268: 
````

- **L242** EN: Invokes `torch.tril_indices` to advance the surrounding implementation. | CN: 调用 `torch.tril_indices` 来推进周围的实现逻辑。
- **L243** EN: Invokes `torch.randn` to advance the surrounding implementation. | CN: 调用 `torch.randn` 来推进周围的实现逻辑。
- **L244** EN: Invokes `torch.Size` to advance the surrounding implementation. | CN: 调用 `torch.Size` 来推进周围的实现逻辑。
- **L245** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L246** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L247** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L248** EN: Assigns or updates `chol`. | CN: 对 `chol` 进行赋值或更新。
- **L249** EN: Returns from `Wishart._bartlett_sampling` with the computed result or updated state. | CN: 从 `Wishart._bartlett_sampling` 返回计算结果或更新后的状态。
- **L250** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L251** EN: Defines function `rsample`, which produces samples according to the modeled stochastic behavior. | CN: 定义函数 `rsample`，其作用是根据建模的随机行为生成样本。
- **L252** EN: Invokes `torch.Size` to advance the surrounding implementation. | CN: 调用 `torch.Size` 来推进周围的实现逻辑。
- **L253** EN: Continues `Wishart.rsample`, which produces samples according to the modeled stochastic behavior. | CN: 继续 `Wishart.rsample` 的实现，其作用是根据建模的随机行为生成样本。
- **L254** EN: Starts the docstring for function `Wishart.rsample`. | CN: 开始为 function `Wishart.rsample` 编写文档字符串。
- **L255** EN: Continues the docstring for function `Wishart.rsample`. | CN: 继续补充 function `Wishart.rsample` 的文档字符串。
- **L256** EN: Continues the docstring for function `Wishart.rsample`. | CN: 继续补充 function `Wishart.rsample` 的文档字符串。
- **L257** EN: Continues the docstring for function `Wishart.rsample`. | CN: 继续补充 function `Wishart.rsample` 的文档字符串。
- **L258** EN: Continues the docstring for function `Wishart.rsample`. | CN: 继续补充 function `Wishart.rsample` 的文档字符串。
- **L259** EN: Continues the docstring for function `Wishart.rsample`. | CN: 继续补充 function `Wishart.rsample` 的文档字符串。
- **L260** EN: Continues the docstring for function `Wishart.rsample`. | CN: 继续补充 function `Wishart.rsample` 的文档字符串。
- **L261** EN: Ends the docstring for function `Wishart.rsample`. | CN: 结束 function `Wishart.rsample` 的文档字符串。
- **L262** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L263** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L264** EN: Assigns or updates `max_try_correction`. | CN: 对 `max_try_correction` 进行赋值或更新。
- **L265** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L266** EN: Assigns or updates `sample_shape`. | CN: 对 `sample_shape` 进行赋值或更新。
- **L267** EN: Assigns or updates `sample`. | CN: 对 `sample` 进行赋值或更新。
- **L268** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 269-292 / 第 269-292 行

````python
0269:         # Below part is to improve numerical stability temporally and should be removed in the future
0270:         is_singular = self.support.check(sample)
0271:         if self._batch_shape:
0272:             is_singular = is_singular.amax(self._batch_dims)
0273: 
0274:         if torch._C._get_tracing_state():
0275:             # Less optimized version for JIT
0276:             for _ in range(max_try_correction):
0277:                 sample_new = self._bartlett_sampling(sample_shape)
0278:                 sample = torch.where(is_singular, sample_new, sample)
0279: 
0280:                 is_singular = ~self.support.check(sample)
0281:                 if self._batch_shape:
0282:                     is_singular = is_singular.amax(self._batch_dims)
0283: 
0284:         else:
0285:             # More optimized version with data-dependent control flow.
0286:             if is_singular.any():
0287:                 warnings.warn("Singular sample detected.", stacklevel=2)
0288: 
0289:                 for _ in range(max_try_correction):
0290:                     sample_new = self._bartlett_sampling(is_singular[is_singular].shape)
0291:                     sample[is_singular] = sample_new
0292: 
````

- **L269** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L270** EN: Assigns or updates `is_singular`. | CN: 对 `is_singular` 进行赋值或更新。
- **L271** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L272** EN: Assigns or updates `is_singular`. | CN: 对 `is_singular` 进行赋值或更新。
- **L273** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L274** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L275** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L276** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L277** EN: Assigns or updates `sample_new`. | CN: 对 `sample_new` 进行赋值或更新。
- **L278** EN: Assigns or updates `sample`. | CN: 对 `sample` 进行赋值或更新。
- **L279** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L280** EN: Assigns or updates `is_singular`. | CN: 对 `is_singular` 进行赋值或更新。
- **L281** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L282** EN: Assigns or updates `is_singular`. | CN: 对 `is_singular` 进行赋值或更新。
- **L283** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L284** EN: Introduces the fallback branch when earlier conditions are not met. | CN: 引入前置条件不满足时的后备分支。
- **L285** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L286** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L287** EN: Invokes `warnings.warn` to advance the surrounding implementation. | CN: 调用 `warnings.warn` 来推进周围的实现逻辑。
- **L288** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L289** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L290** EN: Assigns or updates `sample_new`. | CN: 对 `sample_new` 进行赋值或更新。
- **L291** EN: Continues `Wishart.rsample`, which produces samples according to the modeled stochastic behavior. | CN: 继续 `Wishart.rsample` 的实现，其作用是根据建模的随机行为生成样本。
- **L292** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 293-320 / 第 293-320 行

````python
0293:                     is_singular_new = ~self.support.check(sample_new)
0294:                     if self._batch_shape:
0295:                         is_singular_new = is_singular_new.amax(self._batch_dims)
0296:                     is_singular[is_singular.clone()] = is_singular_new
0297: 
0298:                     if not is_singular.any():
0299:                         break
0300: 
0301:         return sample
0302: 
0303:     def log_prob(self, value):
0304:         if self._validate_args:
0305:             self._validate_sample(value)
0306:         nu = self.df  # has shape (batch_shape)
0307:         p = self._event_shape[-1]  # has singleton shape
0308:         return (
0309:             -nu
0310:             * (
0311:                 p * _log_2 / 2
0312:                 + self._unbroadcasted_scale_tril.diagonal(dim1=-2, dim2=-1)
0313:                 .log()
0314:                 .sum(-1)
0315:             )
0316:             - torch.mvlgamma(nu / 2, p=p)
0317:             + (nu - p - 1) / 2 * torch.linalg.slogdet(value).logabsdet
0318:             - torch.cholesky_solve(value, self._unbroadcasted_scale_tril)
0319:             .diagonal(dim1=-2, dim2=-1)
0320:             .sum(dim=-1)
````

- **L293** EN: Assigns or updates `is_singular_new`. | CN: 对 `is_singular_new` 进行赋值或更新。
- **L294** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L295** EN: Assigns or updates `is_singular_new`. | CN: 对 `is_singular_new` 进行赋值或更新。
- **L296** EN: Invokes `is_singular.clone` to advance the surrounding implementation. | CN: 调用 `is_singular.clone` 来推进周围的实现逻辑。
- **L297** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L298** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L299** EN: Continues `Wishart.rsample`, which produces samples according to the modeled stochastic behavior. | CN: 继续 `Wishart.rsample` 的实现，其作用是根据建模的随机行为生成样本。
- **L300** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L301** EN: Returns from `Wishart.rsample` with the computed result or updated state. | CN: 从 `Wishart.rsample` 返回计算结果或更新后的状态。
- **L302** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L303** EN: Defines function `log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 定义函数 `log_prob`，其作用是为给定输入计算对数概率或类似密度分数。
- **L304** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L305** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L306** EN: Assigns or updates `nu`. | CN: 对 `nu` 进行赋值或更新。
- **L307** EN: Assigns or updates `p`. | CN: 对 `p` 进行赋值或更新。
- **L308** EN: Returns from `Wishart.log_prob` with the computed result or updated state. | CN: 从 `Wishart.log_prob` 返回计算结果或更新后的状态。
- **L309** EN: Continues `Wishart.log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 继续 `Wishart.log_prob` 的实现，其作用是为给定输入计算对数概率或类似密度分数。
- **L310** EN: Continues `Wishart.log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 继续 `Wishart.log_prob` 的实现，其作用是为给定输入计算对数概率或类似密度分数。
- **L311** EN: Continues `Wishart.log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 继续 `Wishart.log_prob` 的实现，其作用是为给定输入计算对数概率或类似密度分数。
- **L312** EN: Invokes `self._unbroadcasted_scale_tril.diagonal` to advance the surrounding implementation. | CN: 调用 `self._unbroadcasted_scale_tril.diagonal` 来推进周围的实现逻辑。
- **L313** EN: Invokes `log` to advance the surrounding implementation. | CN: 调用 `log` 来推进周围的实现逻辑。
- **L314** EN: Invokes `sum` to advance the surrounding implementation. | CN: 调用 `sum` 来推进周围的实现逻辑。
- **L315** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L316** EN: Invokes `torch.mvlgamma` to advance the surrounding implementation. | CN: 调用 `torch.mvlgamma` 来推进周围的实现逻辑。
- **L317** EN: Invokes `torch.linalg.slogdet` to advance the surrounding implementation. | CN: 调用 `torch.linalg.slogdet` 来推进周围的实现逻辑。
- **L318** EN: Invokes `torch.cholesky_solve` to advance the surrounding implementation. | CN: 调用 `torch.cholesky_solve` 来推进周围的实现逻辑。
- **L319** EN: Invokes `diagonal` to advance the surrounding implementation. | CN: 调用 `diagonal` 来推进周围的实现逻辑。
- **L320** EN: Invokes `sum` to advance the surrounding implementation. | CN: 调用 `sum` 来推进周围的实现逻辑。

### Lines 321-345 / 第 321-345 行

````python
0321:             / 2
0322:         )
0323: 
0324:     def entropy(self):
0325:         nu = self.df  # has shape (batch_shape)
0326:         p = self._event_shape[-1]  # has singleton shape
0327:         return (
0328:             (p + 1)
0329:             * (
0330:                 p * _log_2 / 2
0331:                 + self._unbroadcasted_scale_tril.diagonal(dim1=-2, dim2=-1)
0332:                 .log()
0333:                 .sum(-1)
0334:             )
0335:             + torch.mvlgamma(nu / 2, p=p)
0336:             - (nu - p - 1) / 2 * _mvdigamma(nu / 2, p=p)
0337:             + nu * p / 2
0338:         )
0339: 
0340:     @property
0341:     def _natural_params(self) -> tuple[Tensor, Tensor]:
0342:         nu = self.df  # has shape (batch_shape)
0343:         p = self._event_shape[-1]  # has singleton shape
0344:         return -self.precision_matrix / 2, (nu - p - 1) / 2
0345: 
````

- **L321** EN: Continues `Wishart.log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 继续 `Wishart.log_prob` 的实现，其作用是为给定输入计算对数概率或类似密度分数。
- **L322** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L323** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L324** EN: Defines function `entropy`, which computes an entropy-style summary of uncertainty. | CN: 定义函数 `entropy`，其作用是计算描述不确定性的熵类指标。
- **L325** EN: Assigns or updates `nu`. | CN: 对 `nu` 进行赋值或更新。
- **L326** EN: Assigns or updates `p`. | CN: 对 `p` 进行赋值或更新。
- **L327** EN: Returns from `Wishart.entropy` with the computed result or updated state. | CN: 从 `Wishart.entropy` 返回计算结果或更新后的状态。
- **L328** EN: Continues `Wishart.entropy`, which computes an entropy-style summary of uncertainty. | CN: 继续 `Wishart.entropy` 的实现，其作用是计算描述不确定性的熵类指标。
- **L329** EN: Continues `Wishart.entropy`, which computes an entropy-style summary of uncertainty. | CN: 继续 `Wishart.entropy` 的实现，其作用是计算描述不确定性的熵类指标。
- **L330** EN: Continues `Wishart.entropy`, which computes an entropy-style summary of uncertainty. | CN: 继续 `Wishart.entropy` 的实现，其作用是计算描述不确定性的熵类指标。
- **L331** EN: Invokes `self._unbroadcasted_scale_tril.diagonal` to advance the surrounding implementation. | CN: 调用 `self._unbroadcasted_scale_tril.diagonal` 来推进周围的实现逻辑。
- **L332** EN: Invokes `log` to advance the surrounding implementation. | CN: 调用 `log` 来推进周围的实现逻辑。
- **L333** EN: Invokes `sum` to advance the surrounding implementation. | CN: 调用 `sum` 来推进周围的实现逻辑。
- **L334** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L335** EN: Invokes `torch.mvlgamma` to advance the surrounding implementation. | CN: 调用 `torch.mvlgamma` 来推进周围的实现逻辑。
- **L336** EN: Invokes `_mvdigamma` to advance the surrounding implementation. | CN: 调用 `_mvdigamma` 来推进周围的实现逻辑。
- **L337** EN: Continues `Wishart.entropy`, which computes an entropy-style summary of uncertainty. | CN: 继续 `Wishart.entropy` 的实现，其作用是计算描述不确定性的熵类指标。
- **L338** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L339** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L340** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L341** EN: Defines function `_natural_params`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_natural_params`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L342** EN: Assigns or updates `nu`. | CN: 对 `nu` 进行赋值或更新。
- **L343** EN: Assigns or updates `p`. | CN: 对 `p` 进行赋值或更新。
- **L344** EN: Returns from `Wishart._natural_params` with the computed result or updated state. | CN: 从 `Wishart._natural_params` 返回计算结果或更新后的状态。
- **L345** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 346-351 / 第 346-351 行

````python
0346:     # pyrefly: ignore [bad-override]
0347:     def _log_normalizer(self, x, y):
0348:         p = self._event_shape[-1]
0349:         return (y + (p + 1) / 2) * (
0350:             -torch.linalg.slogdet(-2 * x).logabsdet + _log_2 * p
0351:         ) + torch.mvlgamma(y + (p + 1) / 2, p=p)
````

- **L346** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L347** EN: Defines function `_log_normalizer`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_log_normalizer`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L348** EN: Assigns or updates `p`. | CN: 对 `p` 进行赋值或更新。
- **L349** EN: Returns from `Wishart._log_normalizer` with the computed result or updated state. | CN: 从 `Wishart._log_normalizer` 返回计算结果或更新后的状态。
- **L350** EN: Invokes `torch.linalg.slogdet` to advance the surrounding implementation. | CN: 调用 `torch.linalg.slogdet` 来推进周围的实现逻辑。
- **L351** EN: Invokes `torch.mvlgamma` to advance the surrounding implementation. | CN: 调用 `torch.mvlgamma` 来推进周围的实现逻辑。

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

- **Torch imports / Torch 导入**: `torch`、`torch:nan, Tensor`、`torch.distributions:constraints`、`torch.distributions.exp_family:ExponentialFamily`、`torch.distributions.multivariate_normal:_precision_to_scale_tril`、`torch.distributions.utils:lazy_property`、`torch.types:_Number, _size, Number`
- **Other imports / 其他导入**: `math`、`warnings`
- **Top-level classes / 顶层类**: `Wishart`
- **Top-level functions / 顶层函数**: `_mvdigamma`、`_clamp_above_eps`
- **Base classes / 基类**: `ExponentialFamily`
- **Decorators / 装饰器**: 无
- **Module assignments / 模块级赋值**: `__all__`、`_log_2`
