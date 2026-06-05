# kl.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/distributions/kl.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Registers or implements KL-divergence rules between compatible distribution pairs.
- **Purpose (CN)**: 注册或实现兼容分布对之间的 KL 散度规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-34 / 第 1-34 行

````python
0001: # mypy: allow-untyped-defs
0002: import math
0003: import warnings
0004: from collections.abc import Callable
0005: from functools import total_ordering
0006: 
0007: import torch
0008: from torch import inf, Tensor
0009: 
0010: from .bernoulli import Bernoulli
0011: from .beta import Beta
0012: from .binomial import Binomial
0013: from .categorical import Categorical
0014: from .cauchy import Cauchy
0015: from .continuous_bernoulli import ContinuousBernoulli
0016: from .dirichlet import Dirichlet
0017: from .distribution import Distribution
0018: from .exp_family import ExponentialFamily
0019: from .exponential import Exponential
0020: from .gamma import Gamma
0021: from .geometric import Geometric
0022: from .gumbel import Gumbel
0023: from .half_normal import HalfNormal
0024: from .independent import Independent
0025: from .laplace import Laplace
0026: from .lowrank_multivariate_normal import (
0027:     _batch_lowrank_logdet,
0028:     _batch_lowrank_mahalanobis,
0029:     LowRankMultivariateNormal,
0030: )
0031: from .multivariate_normal import _batch_mahalanobis, MultivariateNormal
0032: from .normal import Normal
0033: from .one_hot_categorical import OneHotCategorical
0034: from .pareto import Pareto
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Imports module dependencies: `math`. | CN: 导入模块依赖：`math`。
- **L3** EN: Imports module dependencies: `warnings`. | CN: 导入模块依赖：`warnings`。
- **L4** EN: Imports `Callable` from `collections.abc` so later code can reuse those definitions. | CN: 从 `collections.abc` 导入 `Callable`，供后续代码复用这些定义。
- **L5** EN: Imports `total_ordering` from `functools` so later code can reuse those definitions. | CN: 从 `functools` 导入 `total_ordering`，供后续代码复用这些定义。
- **L6** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L7** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L8** EN: Imports `inf, Tensor` from `torch` so later code can reuse those definitions. | CN: 从 `torch` 导入 `inf, Tensor`，供后续代码复用这些定义。
- **L9** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L10** EN: Imports `Bernoulli` from `.bernoulli` so later code can reuse those definitions. | CN: 从 `.bernoulli` 导入 `Bernoulli`，供后续代码复用这些定义。
- **L11** EN: Imports `Beta` from `.beta` so later code can reuse those definitions. | CN: 从 `.beta` 导入 `Beta`，供后续代码复用这些定义。
- **L12** EN: Imports `Binomial` from `.binomial` so later code can reuse those definitions. | CN: 从 `.binomial` 导入 `Binomial`，供后续代码复用这些定义。
- **L13** EN: Imports `Categorical` from `.categorical` so later code can reuse those definitions. | CN: 从 `.categorical` 导入 `Categorical`，供后续代码复用这些定义。
- **L14** EN: Imports `Cauchy` from `.cauchy` so later code can reuse those definitions. | CN: 从 `.cauchy` 导入 `Cauchy`，供后续代码复用这些定义。
- **L15** EN: Imports `ContinuousBernoulli` from `.continuous_bernoulli` so later code can reuse those definitions. | CN: 从 `.continuous_bernoulli` 导入 `ContinuousBernoulli`，供后续代码复用这些定义。
- **L16** EN: Imports `Dirichlet` from `.dirichlet` so later code can reuse those definitions. | CN: 从 `.dirichlet` 导入 `Dirichlet`，供后续代码复用这些定义。
- **L17** EN: Imports `Distribution` from `.distribution` so later code can reuse those definitions. | CN: 从 `.distribution` 导入 `Distribution`，供后续代码复用这些定义。
- **L18** EN: Imports `ExponentialFamily` from `.exp_family` so later code can reuse those definitions. | CN: 从 `.exp_family` 导入 `ExponentialFamily`，供后续代码复用这些定义。
- **L19** EN: Imports `Exponential` from `.exponential` so later code can reuse those definitions. | CN: 从 `.exponential` 导入 `Exponential`，供后续代码复用这些定义。
- **L20** EN: Imports `Gamma` from `.gamma` so later code can reuse those definitions. | CN: 从 `.gamma` 导入 `Gamma`，供后续代码复用这些定义。
- **L21** EN: Imports `Geometric` from `.geometric` so later code can reuse those definitions. | CN: 从 `.geometric` 导入 `Geometric`，供后续代码复用这些定义。
- **L22** EN: Imports `Gumbel` from `.gumbel` so later code can reuse those definitions. | CN: 从 `.gumbel` 导入 `Gumbel`，供后续代码复用这些定义。
- **L23** EN: Imports `HalfNormal` from `.half_normal` so later code can reuse those definitions. | CN: 从 `.half_normal` 导入 `HalfNormal`，供后续代码复用这些定义。
- **L24** EN: Imports `Independent` from `.independent` so later code can reuse those definitions. | CN: 从 `.independent` 导入 `Independent`，供后续代码复用这些定义。
- **L25** EN: Imports `Laplace` from `.laplace` so later code can reuse those definitions. | CN: 从 `.laplace` 导入 `Laplace`，供后续代码复用这些定义。
- **L26** EN: Starts a multi-line import from `.lowrank_multivariate_normal` so several helpers can be listed clearly. | CN: 开始一个来自 `.lowrank_multivariate_normal` 的多行导入，以便清晰列出多个辅助符号。
- **L27** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L28** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L29** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L30** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L31** EN: Imports `_batch_mahalanobis, MultivariateNormal` from `.multivariate_normal` so later code can reuse those definitions. | CN: 从 `.multivariate_normal` 导入 `_batch_mahalanobis, MultivariateNormal`，供后续代码复用这些定义。
- **L32** EN: Imports `Normal` from `.normal` so later code can reuse those definitions. | CN: 从 `.normal` 导入 `Normal`，供后续代码复用这些定义。
- **L33** EN: Imports `OneHotCategorical` from `.one_hot_categorical` so later code can reuse those definitions. | CN: 从 `.one_hot_categorical` 导入 `OneHotCategorical`，供后续代码复用这些定义。
- **L34** EN: Imports `Pareto` from `.pareto` so later code can reuse those definitions. | CN: 从 `.pareto` 导入 `Pareto`，供后续代码复用这些定义。

### Lines 35-68 / 第 35-68 行

````python
0035: from .poisson import Poisson
0036: from .transformed_distribution import TransformedDistribution
0037: from .uniform import Uniform
0038: from .utils import _sum_rightmost, euler_constant as _euler_gamma
0039: 
0040: 
0041: _KL_REGISTRY: dict[
0042:     tuple[type, type], Callable
0043: ] = {}  # Source of truth mapping a few general (type, type) pairs to functions.
0044: _KL_MEMOIZE: dict[
0045:     tuple[type, type], Callable
0046: ] = {}  # Memoized version mapping many specific (type, type) pairs to functions.
0047: 
0048: __all__ = ["register_kl", "kl_divergence"]
0049: 
0050: 
0051: def register_kl(type_p, type_q):
0052:     """
0053:     Decorator to register a pairwise function with :meth:`kl_divergence`.
0054:     Usage::
0055: 
0056:         @register_kl(Normal, Normal)
0057:         def kl_normal_normal(p, q):
0058:             # insert implementation here
0059: 
0060:     Lookup returns the most specific (type,type) match ordered by subclass. If
0061:     the match is ambiguous, a `RuntimeWarning` is raised. For example to
0062:     resolve the ambiguous situation::
0063: 
0064:         @register_kl(BaseP, DerivedQ)
0065:         def kl_version1(p, q): ...
0066:         @register_kl(DerivedP, BaseQ)
0067:         def kl_version2(p, q): ...
0068: 
````

- **L35** EN: Imports `Poisson` from `.poisson` so later code can reuse those definitions. | CN: 从 `.poisson` 导入 `Poisson`，供后续代码复用这些定义。
- **L36** EN: Imports `TransformedDistribution` from `.transformed_distribution` so later code can reuse those definitions. | CN: 从 `.transformed_distribution` 导入 `TransformedDistribution`，供后续代码复用这些定义。
- **L37** EN: Imports `Uniform` from `.uniform` so later code can reuse those definitions. | CN: 从 `.uniform` 导入 `Uniform`，供后续代码复用这些定义。
- **L38** EN: Imports `_sum_rightmost, euler_constant as _euler_gamma` from `.utils` so later code can reuse those definitions. | CN: 从 `.utils` 导入 `_sum_rightmost, euler_constant as _euler_gamma`，供后续代码复用这些定义。
- **L39** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L40** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L41** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L42** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L43** EN: Invokes `general` to advance the surrounding implementation. | CN: 调用 `general` 来推进周围的实现逻辑。
- **L44** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L45** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L46** EN: Invokes `specific` to advance the surrounding implementation. | CN: 调用 `specific` 来推进周围的实现逻辑。
- **L47** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L48** EN: Defines or updates the module’s public export list. | CN: 定义或更新模块的公开导出列表。
- **L49** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L50** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L51** EN: Defines function `register_kl`, which registers behavior, patterns, or handlers with a surrounding subsystem. | CN: 定义函数 `register_kl`，其作用是向周边子系统注册行为、模式或处理器。
- **L52** EN: Starts the docstring for function `register_kl`. | CN: 开始为 function `register_kl` 编写文档字符串。
- **L53** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L54** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L55** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L56** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L57** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L58** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L59** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L60** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L61** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L62** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L63** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L64** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L65** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L66** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L67** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L68** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 69-100 / 第 69-100 行

````python
0069:     you should register a third most-specific implementation, e.g.::
0070: 
0071:         register_kl(DerivedP, DerivedQ)(kl_version1)  # Break the tie.
0072: 
0073:     Args:
0074:         type_p (type): A subclass of :class:`~torch.distributions.Distribution`.
0075:         type_q (type): A subclass of :class:`~torch.distributions.Distribution`.
0076:     """
0077:     if not isinstance(type_p, type) and issubclass(type_p, Distribution):
0078:         raise TypeError(
0079:             f"Expected type_p to be a Distribution subclass but got {type_p}"
0080:         )
0081:     if not isinstance(type_q, type) and issubclass(type_q, Distribution):
0082:         raise TypeError(
0083:             f"Expected type_q to be a Distribution subclass but got {type_q}"
0084:         )
0085: 
0086:     def decorator(fun):
0087:         _KL_REGISTRY[type_p, type_q] = fun
0088:         _KL_MEMOIZE.clear()  # reset since lookup order may have changed
0089:         return fun
0090: 
0091:     return decorator
0092: 
0093: 
0094: @total_ordering
0095: class _Match:
0096:     __slots__ = ["types"]
0097: 
0098:     def __init__(self, *types):
0099:         self.types = types
0100: 
````

- **L69** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L70** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L71** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L72** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L73** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L74** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L75** EN: Continues the docstring for function `register_kl`. | CN: 继续补充 function `register_kl` 的文档字符串。
- **L76** EN: Ends the docstring for function `register_kl`. | CN: 结束 function `register_kl` 的文档字符串。
- **L77** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L78** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L79** EN: Continues `register_kl`, which registers behavior, patterns, or handlers with a surrounding subsystem. | CN: 继续 `register_kl` 的实现，其作用是向周边子系统注册行为、模式或处理器。
- **L80** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L81** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L82** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L83** EN: Continues `register_kl`, which registers behavior, patterns, or handlers with a surrounding subsystem. | CN: 继续 `register_kl` 的实现，其作用是向周边子系统注册行为、模式或处理器。
- **L84** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L85** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L86** EN: Defines function `decorator`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `decorator`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L87** EN: Continues `register_kl.decorator`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `register_kl.decorator` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L88** EN: Invokes `_KL_MEMOIZE.clear` to advance the surrounding implementation. | CN: 调用 `_KL_MEMOIZE.clear` 来推进周围的实现逻辑。
- **L89** EN: Returns from `register_kl.decorator` with the computed result or updated state. | CN: 从 `register_kl.decorator` 返回计算结果或更新后的状态。
- **L90** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L91** EN: Returns from `register_kl` with the computed result or updated state. | CN: 从 `register_kl` 返回计算结果或更新后的状态。
- **L92** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L93** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L94** EN: Applies decorator `total_ordering`, which modifies the behavior of the following definition. | CN: 应用装饰器 `total_ordering`，其作用是修改后续定义的行为。
- **L95** EN: Defines class `_Match`, which encapsulates distribution parameters, validation rules, and tensor-valued statistical behavior. | CN: 定义类 `_Match`，其作用是封装分布参数、校验规则以及基于张量的统计行为。
- **L96** EN: Assigns module-level configuration or cached state to `__slots__`. | CN: 为 `__slots__` 赋予模块级配置或缓存状态。
- **L97** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L98** EN: Defines function `__init__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__init__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L99** EN: Updates object state via `self.types`. | CN: 通过 `self.types` 更新对象状态。
- **L100** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 101-134 / 第 101-134 行

````python
0101:     def __eq__(self, other):
0102:         return self.types == other.types
0103: 
0104:     def __le__(self, other):
0105:         for x, y in zip(self.types, other.types):
0106:             if not issubclass(x, y):
0107:                 return False
0108:             if x is not y:
0109:                 break
0110:         return True
0111: 
0112: 
0113: def _dispatch_kl(type_p, type_q):
0114:     """
0115:     Find the most specific approximate match, assuming single inheritance.
0116:     """
0117:     matches = [
0118:         (super_p, super_q)
0119:         for super_p, super_q in _KL_REGISTRY
0120:         if issubclass(type_p, super_p) and issubclass(type_q, super_q)
0121:     ]
0122:     if not matches:
0123:         return NotImplemented
0124:     # Check that the left- and right- lexicographic orders agree.
0125:     # mypy isn't smart enough to know that _Match implements __lt__
0126:     # see: https://github.com/python/typing/issues/760#issuecomment-710670503
0127:     left_p, left_q = min(_Match(*m) for m in matches).types  # type: ignore[type-var]
0128:     right_q, right_p = min(_Match(*reversed(m)) for m in matches).types  # type: ignore[type-var]
0129:     left_fun = _KL_REGISTRY[left_p, left_q]
0130:     right_fun = _KL_REGISTRY[right_p, right_q]
0131:     if left_fun is not right_fun:
0132:         warnings.warn(
0133:             f"Ambiguous kl_divergence({type_p.__name__}, {type_q.__name__}). "
0134:             f"Please register_kl({left_p.__name__}, {right_q.__name__})",
````

- **L101** EN: Defines function `__eq__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__eq__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L102** EN: Returns from `_Match.__eq__` with the computed result or updated state. | CN: 从 `_Match.__eq__` 返回计算结果或更新后的状态。
- **L103** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L104** EN: Defines function `__le__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__le__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L105** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L106** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L107** EN: Returns from `_Match.__le__` with the computed result or updated state. | CN: 从 `_Match.__le__` 返回计算结果或更新后的状态。
- **L108** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L109** EN: Continues `_Match.__le__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_Match.__le__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L110** EN: Returns from `_Match.__le__` with the computed result or updated state. | CN: 从 `_Match.__le__` 返回计算结果或更新后的状态。
- **L111** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L112** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L113** EN: Defines function `_dispatch_kl`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_dispatch_kl`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L114** EN: Starts the docstring for function `_dispatch_kl`. | CN: 开始为 function `_dispatch_kl` 编写文档字符串。
- **L115** EN: Continues the docstring for function `_dispatch_kl`. | CN: 继续补充 function `_dispatch_kl` 的文档字符串。
- **L116** EN: Ends the docstring for function `_dispatch_kl`. | CN: 结束 function `_dispatch_kl` 的文档字符串。
- **L117** EN: Assigns or updates `matches`. | CN: 对 `matches` 进行赋值或更新。
- **L118** EN: Continues `_dispatch_kl`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_dispatch_kl` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L119** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L120** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L121** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L122** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L123** EN: Returns from `_dispatch_kl` with the computed result or updated state. | CN: 从 `_dispatch_kl` 返回计算结果或更新后的状态。
- **L124** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L125** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L126** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L127** EN: Invokes `min` to advance the surrounding implementation. | CN: 调用 `min` 来推进周围的实现逻辑。
- **L128** EN: Invokes `min` to advance the surrounding implementation. | CN: 调用 `min` 来推进周围的实现逻辑。
- **L129** EN: Assigns or updates `left_fun`. | CN: 对 `left_fun` 进行赋值或更新。
- **L130** EN: Assigns or updates `right_fun`. | CN: 对 `right_fun` 进行赋值或更新。
- **L131** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L132** EN: Invokes `warnings.warn` to advance the surrounding implementation. | CN: 调用 `warnings.warn` 来推进周围的实现逻辑。
- **L133** EN: Invokes `kl_divergence` to advance the surrounding implementation. | CN: 调用 `kl_divergence` 来推进周围的实现逻辑。
- **L134** EN: Invokes `register_kl` to advance the surrounding implementation. | CN: 调用 `register_kl` 来推进周围的实现逻辑。

### Lines 135-168 / 第 135-168 行

````python
0135:             RuntimeWarning,
0136:             stacklevel=2,
0137:         )
0138:     return left_fun
0139: 
0140: 
0141: def _infinite_like(tensor):
0142:     """
0143:     Helper function for obtaining infinite KL Divergence throughout
0144:     """
0145:     return torch.full_like(tensor, inf)
0146: 
0147: 
0148: def _x_log_x(tensor):
0149:     """
0150:     Utility function for calculating x log x
0151:     """
0152:     return torch.special.xlogy(tensor, tensor)  # produces correct result for x=0
0153: 
0154: 
0155: def _batch_trace_XXT(bmat):
0156:     """
0157:     Utility function for calculating the trace of XX^{T} with X having arbitrary trailing batch dimensions
0158:     """
0159:     n = bmat.size(-1)
0160:     m = bmat.size(-2)
0161:     flat_trace = bmat.reshape(-1, m * n).pow(2).sum(-1)
0162:     return flat_trace.reshape(bmat.shape[:-2])
0163: 
0164: 
0165: def kl_divergence(p: Distribution, q: Distribution) -> Tensor:
0166:     r"""
0167:     Compute Kullback-Leibler divergence :math:`KL(p \| q)` between two distributions.
0168: 
````

- **L135** EN: Continues `_dispatch_kl`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_dispatch_kl` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L136** EN: Assigns or updates `stacklevel`. | CN: 对 `stacklevel` 进行赋值或更新。
- **L137** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L138** EN: Returns from `_dispatch_kl` with the computed result or updated state. | CN: 从 `_dispatch_kl` 返回计算结果或更新后的状态。
- **L139** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L140** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L141** EN: Defines function `_infinite_like`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_infinite_like`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L142** EN: Starts the docstring for function `_infinite_like`. | CN: 开始为 function `_infinite_like` 编写文档字符串。
- **L143** EN: Continues the docstring for function `_infinite_like`. | CN: 继续补充 function `_infinite_like` 的文档字符串。
- **L144** EN: Ends the docstring for function `_infinite_like`. | CN: 结束 function `_infinite_like` 的文档字符串。
- **L145** EN: Returns from `_infinite_like` with the computed result or updated state. | CN: 从 `_infinite_like` 返回计算结果或更新后的状态。
- **L146** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L147** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L148** EN: Defines function `_x_log_x`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_x_log_x`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L149** EN: Starts the docstring for function `_x_log_x`. | CN: 开始为 function `_x_log_x` 编写文档字符串。
- **L150** EN: Continues the docstring for function `_x_log_x`. | CN: 继续补充 function `_x_log_x` 的文档字符串。
- **L151** EN: Ends the docstring for function `_x_log_x`. | CN: 结束 function `_x_log_x` 的文档字符串。
- **L152** EN: Returns from `_x_log_x` with the computed result or updated state. | CN: 从 `_x_log_x` 返回计算结果或更新后的状态。
- **L153** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L154** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L155** EN: Defines function `_batch_trace_XXT`, which records or analyzes execution structure for later compilation. | CN: 定义函数 `_batch_trace_XXT`，其作用是记录或分析执行结构，以便后续编译。
- **L156** EN: Starts the docstring for function `_batch_trace_XXT`. | CN: 开始为 function `_batch_trace_XXT` 编写文档字符串。
- **L157** EN: Continues the docstring for function `_batch_trace_XXT`. | CN: 继续补充 function `_batch_trace_XXT` 的文档字符串。
- **L158** EN: Ends the docstring for function `_batch_trace_XXT`. | CN: 结束 function `_batch_trace_XXT` 的文档字符串。
- **L159** EN: Assigns or updates `n`. | CN: 对 `n` 进行赋值或更新。
- **L160** EN: Assigns or updates `m`. | CN: 对 `m` 进行赋值或更新。
- **L161** EN: Assigns or updates `flat_trace`. | CN: 对 `flat_trace` 进行赋值或更新。
- **L162** EN: Returns from `_batch_trace_XXT` with the computed result or updated state. | CN: 从 `_batch_trace_XXT` 返回计算结果或更新后的状态。
- **L163** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L164** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L165** EN: Defines function `kl_divergence`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `kl_divergence`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L166** EN: Starts the docstring for function `kl_divergence`. | CN: 开始为 function `kl_divergence` 编写文档字符串。
- **L167** EN: Continues the docstring for function `kl_divergence`. | CN: 继续补充 function `kl_divergence` 的文档字符串。
- **L168** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 169-202 / 第 169-202 行

````python
0169:     .. math::
0170: 
0171:         KL(p \| q) = \int p(x) \log\frac {p(x)} {q(x)} \,dx
0172: 
0173:     Args:
0174:         p (Distribution): A :class:`~torch.distributions.Distribution` object.
0175:         q (Distribution): A :class:`~torch.distributions.Distribution` object.
0176: 
0177:     Returns:
0178:         Tensor: A batch of KL divergences of shape `batch_shape`.
0179: 
0180:     Raises:
0181:         NotImplementedError: If the distribution types have not been registered via
0182:             :meth:`register_kl`.
0183:     """
0184:     try:
0185:         fun = _KL_MEMOIZE[type(p), type(q)]
0186:     except KeyError:
0187:         fun = _dispatch_kl(type(p), type(q))
0188:         _KL_MEMOIZE[type(p), type(q)] = fun
0189:     if fun is NotImplemented:
0190:         raise NotImplementedError(
0191:             f"No KL(p || q) is implemented for p type {p.__class__.__name__} and q type {q.__class__.__name__}"
0192:         )
0193:     return fun(p, q)
0194: 
0195: 
0196: ################################################################################
0197: # KL Divergence Implementations
0198: ################################################################################
0199: 
0200: # Same distributions
0201: 
0202: 
````

- **L169** EN: Continues the docstring for function `kl_divergence`. | CN: 继续补充 function `kl_divergence` 的文档字符串。
- **L170** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L171** EN: Continues the docstring for function `kl_divergence`. | CN: 继续补充 function `kl_divergence` 的文档字符串。
- **L172** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L173** EN: Continues the docstring for function `kl_divergence`. | CN: 继续补充 function `kl_divergence` 的文档字符串。
- **L174** EN: Continues the docstring for function `kl_divergence`. | CN: 继续补充 function `kl_divergence` 的文档字符串。
- **L175** EN: Continues the docstring for function `kl_divergence`. | CN: 继续补充 function `kl_divergence` 的文档字符串。
- **L176** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L177** EN: Continues the docstring for function `kl_divergence`. | CN: 继续补充 function `kl_divergence` 的文档字符串。
- **L178** EN: Continues the docstring for function `kl_divergence`. | CN: 继续补充 function `kl_divergence` 的文档字符串。
- **L179** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L180** EN: Continues the docstring for function `kl_divergence`. | CN: 继续补充 function `kl_divergence` 的文档字符串。
- **L181** EN: Continues the docstring for function `kl_divergence`. | CN: 继续补充 function `kl_divergence` 的文档字符串。
- **L182** EN: Continues the docstring for function `kl_divergence`. | CN: 继续补充 function `kl_divergence` 的文档字符串。
- **L183** EN: Ends the docstring for function `kl_divergence`. | CN: 结束 function `kl_divergence` 的文档字符串。
- **L184** EN: Begins a guarded region that can recover from runtime failures. | CN: 开始一个可从运行时失败中恢复的受保护区域。
- **L185** EN: Assigns or updates `fun`. | CN: 对 `fun` 进行赋值或更新。
- **L186** EN: Handles an exception path for a specific failure mode. | CN: 处理某种特定失败模式对应的异常路径。
- **L187** EN: Assigns or updates `fun`. | CN: 对 `fun` 进行赋值或更新。
- **L188** EN: Invokes `type` to advance the surrounding implementation. | CN: 调用 `type` 来推进周围的实现逻辑。
- **L189** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L190** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L191** EN: Invokes `KL` to advance the surrounding implementation. | CN: 调用 `KL` 来推进周围的实现逻辑。
- **L192** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L193** EN: Returns from `kl_divergence` with the computed result or updated state. | CN: 从 `kl_divergence` 返回计算结果或更新后的状态。
- **L194** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L195** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L196** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L197** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L198** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L199** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L200** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L201** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L202** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 203-230 / 第 203-230 行

````python
0203: @register_kl(Bernoulli, Bernoulli)
0204: def _kl_bernoulli_bernoulli(p, q):
0205:     t1 = p.probs * (
0206:         torch.nn.functional.softplus(-q.logits)
0207:         - torch.nn.functional.softplus(-p.logits)
0208:     )
0209:     t1[q.probs == 0] = inf
0210:     t1[p.probs == 0] = 0
0211:     t2 = (1 - p.probs) * (
0212:         torch.nn.functional.softplus(q.logits) - torch.nn.functional.softplus(p.logits)
0213:     )
0214:     t2[q.probs == 1] = inf
0215:     t2[p.probs == 1] = 0
0216:     return t1 + t2
0217: 
0218: 
0219: @register_kl(Beta, Beta)
0220: def _kl_beta_beta(p, q):
0221:     sum_params_p = p.concentration1 + p.concentration0
0222:     sum_params_q = q.concentration1 + q.concentration0
0223:     t1 = q.concentration1.lgamma() + q.concentration0.lgamma() + (sum_params_p).lgamma()
0224:     t2 = p.concentration1.lgamma() + p.concentration0.lgamma() + (sum_params_q).lgamma()
0225:     t3 = (p.concentration1 - q.concentration1) * torch.digamma(p.concentration1)
0226:     t4 = (p.concentration0 - q.concentration0) * torch.digamma(p.concentration0)
0227:     t5 = (sum_params_q - sum_params_p) * torch.digamma(sum_params_p)
0228:     return t1 - t2 + t3 + t4 + t5
0229: 
0230: 
````

- **L203** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L204** EN: Defines function `_kl_bernoulli_bernoulli`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_bernoulli_bernoulli`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L205** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L206** EN: Invokes `torch.nn.functional.softplus` to advance the surrounding implementation. | CN: 调用 `torch.nn.functional.softplus` 来推进周围的实现逻辑。
- **L207** EN: Invokes `torch.nn.functional.softplus` to advance the surrounding implementation. | CN: 调用 `torch.nn.functional.softplus` 来推进周围的实现逻辑。
- **L208** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L209** EN: Continues `_kl_bernoulli_bernoulli`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_bernoulli_bernoulli` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L210** EN: Continues `_kl_bernoulli_bernoulli`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_bernoulli_bernoulli` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L211** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L212** EN: Invokes `torch.nn.functional.softplus` to advance the surrounding implementation. | CN: 调用 `torch.nn.functional.softplus` 来推进周围的实现逻辑。
- **L213** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L214** EN: Continues `_kl_bernoulli_bernoulli`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_bernoulli_bernoulli` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L215** EN: Continues `_kl_bernoulli_bernoulli`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_bernoulli_bernoulli` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L216** EN: Returns from `_kl_bernoulli_bernoulli` with the computed result or updated state. | CN: 从 `_kl_bernoulli_bernoulli` 返回计算结果或更新后的状态。
- **L217** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L218** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L219** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L220** EN: Defines function `_kl_beta_beta`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_beta_beta`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L221** EN: Assigns or updates `sum_params_p`. | CN: 对 `sum_params_p` 进行赋值或更新。
- **L222** EN: Assigns or updates `sum_params_q`. | CN: 对 `sum_params_q` 进行赋值或更新。
- **L223** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L224** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L225** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L226** EN: Assigns or updates `t4`. | CN: 对 `t4` 进行赋值或更新。
- **L227** EN: Assigns or updates `t5`. | CN: 对 `t5` 进行赋值或更新。
- **L228** EN: Returns from `_kl_beta_beta` with the computed result or updated state. | CN: 从 `_kl_beta_beta` 返回计算结果或更新后的状态。
- **L229** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L230** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 231-262 / 第 231-262 行

````python
0231: @register_kl(Binomial, Binomial)
0232: def _kl_binomial_binomial(p, q):
0233:     # from https://math.stackexchange.com/questions/2214993/
0234:     # kullback-leibler-divergence-for-binomial-distributions-p-and-q
0235:     if (p.total_count < q.total_count).any():
0236:         raise NotImplementedError(
0237:             "KL between Binomials where q.total_count > p.total_count is not implemented"
0238:         )
0239:     kl = p.total_count * (
0240:         p.probs * (p.logits - q.logits) + (-p.probs).log1p() - (-q.probs).log1p()
0241:     )
0242:     inf_idxs = p.total_count > q.total_count
0243:     kl[inf_idxs] = _infinite_like(kl[inf_idxs])
0244:     return kl
0245: 
0246: 
0247: @register_kl(Categorical, Categorical)
0248: def _kl_categorical_categorical(p, q):
0249:     t = p.probs * (p.logits - q.logits)
0250:     t[(q.probs == 0).expand_as(t)] = inf
0251:     t[(p.probs == 0).expand_as(t)] = 0
0252:     return t.sum(-1)
0253: 
0254: 
0255: @register_kl(ContinuousBernoulli, ContinuousBernoulli)
0256: def _kl_continuous_bernoulli_continuous_bernoulli(p, q):
0257:     t1 = p.mean * (p.logits - q.logits)
0258:     t2 = p._cont_bern_log_norm() + torch.log1p(-p.probs)
0259:     t3 = -q._cont_bern_log_norm() - torch.log1p(-q.probs)
0260:     return t1 + t2 + t3
0261: 
0262: 
````

- **L231** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L232** EN: Defines function `_kl_binomial_binomial`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_binomial_binomial`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L233** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L234** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L235** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L236** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L237** EN: Continues `_kl_binomial_binomial`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_binomial_binomial` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L238** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L239** EN: Assigns or updates `kl`. | CN: 对 `kl` 进行赋值或更新。
- **L240** EN: Invokes `log1p` to advance the surrounding implementation. | CN: 调用 `log1p` 来推进周围的实现逻辑。
- **L241** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L242** EN: Assigns or updates `inf_idxs`. | CN: 对 `inf_idxs` 进行赋值或更新。
- **L243** EN: Invokes `_infinite_like` to advance the surrounding implementation. | CN: 调用 `_infinite_like` 来推进周围的实现逻辑。
- **L244** EN: Returns from `_kl_binomial_binomial` with the computed result or updated state. | CN: 从 `_kl_binomial_binomial` 返回计算结果或更新后的状态。
- **L245** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L246** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L247** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L248** EN: Defines function `_kl_categorical_categorical`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_categorical_categorical`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L249** EN: Assigns or updates `t`. | CN: 对 `t` 进行赋值或更新。
- **L250** EN: Invokes `expand_as` to advance the surrounding implementation. | CN: 调用 `expand_as` 来推进周围的实现逻辑。
- **L251** EN: Invokes `expand_as` to advance the surrounding implementation. | CN: 调用 `expand_as` 来推进周围的实现逻辑。
- **L252** EN: Returns from `_kl_categorical_categorical` with the computed result or updated state. | CN: 从 `_kl_categorical_categorical` 返回计算结果或更新后的状态。
- **L253** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L254** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L255** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L256** EN: Defines function `_kl_continuous_bernoulli_continuous_bernoulli`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_continuous_bernoulli_continuous_bernoulli`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L257** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L258** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L259** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L260** EN: Returns from `_kl_continuous_bernoulli_continuous_bernoulli` with the computed result or updated state. | CN: 从 `_kl_continuous_bernoulli_continuous_bernoulli` 返回计算结果或更新后的状态。
- **L261** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L262** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 263-296 / 第 263-296 行

````python
0263: @register_kl(Dirichlet, Dirichlet)
0264: def _kl_dirichlet_dirichlet(p, q):
0265:     # From http://bariskurt.com/kullback-leibler-divergence-between-two-dirichlet-and-beta-distributions/
0266:     sum_p_concentration = p.concentration.sum(-1)
0267:     sum_q_concentration = q.concentration.sum(-1)
0268:     t1 = sum_p_concentration.lgamma() - sum_q_concentration.lgamma()
0269:     t2 = (p.concentration.lgamma() - q.concentration.lgamma()).sum(-1)
0270:     t3 = p.concentration - q.concentration
0271:     t4 = p.concentration.digamma() - sum_p_concentration.digamma().unsqueeze(-1)
0272:     return t1 - t2 + (t3 * t4).sum(-1)
0273: 
0274: 
0275: @register_kl(Exponential, Exponential)
0276: def _kl_exponential_exponential(p, q):
0277:     rate_ratio = q.rate / p.rate
0278:     t1 = -rate_ratio.log()
0279:     return t1 + rate_ratio - 1
0280: 
0281: 
0282: @register_kl(ExponentialFamily, ExponentialFamily)
0283: def _kl_expfamily_expfamily(p, q):
0284:     if type(p) is not type(q):
0285:         raise NotImplementedError(
0286:             "The cross KL-divergence between different exponential families cannot \
0287:                             be computed using Bregman divergences"
0288:         )
0289:     p_nparams = [np.detach().requires_grad_() for np in p._natural_params]
0290:     q_nparams = q._natural_params
0291:     lg_normal = p._log_normalizer(*p_nparams)
0292:     gradients = torch.autograd.grad(lg_normal.sum(), p_nparams, create_graph=True)
0293:     result = q._log_normalizer(*q_nparams) - lg_normal
0294:     for pnp, qnp, g in zip(p_nparams, q_nparams, gradients):
0295:         term = (qnp - pnp) * g
0296:         result -= _sum_rightmost(term, len(q.event_shape))
````

- **L263** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L264** EN: Defines function `_kl_dirichlet_dirichlet`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_dirichlet_dirichlet`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L265** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L266** EN: Assigns or updates `sum_p_concentration`. | CN: 对 `sum_p_concentration` 进行赋值或更新。
- **L267** EN: Assigns or updates `sum_q_concentration`. | CN: 对 `sum_q_concentration` 进行赋值或更新。
- **L268** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L269** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L270** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L271** EN: Assigns or updates `t4`. | CN: 对 `t4` 进行赋值或更新。
- **L272** EN: Returns from `_kl_dirichlet_dirichlet` with the computed result or updated state. | CN: 从 `_kl_dirichlet_dirichlet` 返回计算结果或更新后的状态。
- **L273** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L274** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L275** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L276** EN: Defines function `_kl_exponential_exponential`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_exponential_exponential`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L277** EN: Assigns or updates `rate_ratio`. | CN: 对 `rate_ratio` 进行赋值或更新。
- **L278** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L279** EN: Returns from `_kl_exponential_exponential` with the computed result or updated state. | CN: 从 `_kl_exponential_exponential` 返回计算结果或更新后的状态。
- **L280** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L281** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L282** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L283** EN: Defines function `_kl_expfamily_expfamily`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_expfamily_expfamily`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L284** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L285** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L286** EN: Continues `_kl_expfamily_expfamily`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_expfamily_expfamily` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L287** EN: Continues `_kl_expfamily_expfamily`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_expfamily_expfamily` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L288** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L289** EN: Assigns or updates `p_nparams`. | CN: 对 `p_nparams` 进行赋值或更新。
- **L290** EN: Assigns or updates `q_nparams`. | CN: 对 `q_nparams` 进行赋值或更新。
- **L291** EN: Assigns or updates `lg_normal`. | CN: 对 `lg_normal` 进行赋值或更新。
- **L292** EN: Assigns or updates `gradients`. | CN: 对 `gradients` 进行赋值或更新。
- **L293** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L294** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L295** EN: Assigns or updates `term`. | CN: 对 `term` 进行赋值或更新。
- **L296** EN: Invokes `_sum_rightmost` to advance the surrounding implementation. | CN: 调用 `_sum_rightmost` 来推进周围的实现逻辑。

### Lines 297-329 / 第 297-329 行

````python
0297:     return result
0298: 
0299: 
0300: @register_kl(Gamma, Gamma)
0301: def _kl_gamma_gamma(p, q):
0302:     t1 = q.concentration * (p.rate / q.rate).log()
0303:     t2 = torch.lgamma(q.concentration) - torch.lgamma(p.concentration)
0304:     t3 = (p.concentration - q.concentration) * torch.digamma(p.concentration)
0305:     t4 = (q.rate - p.rate) * (p.concentration / p.rate)
0306:     return t1 + t2 + t3 + t4
0307: 
0308: 
0309: @register_kl(Gumbel, Gumbel)
0310: def _kl_gumbel_gumbel(p, q):
0311:     ct1 = p.scale / q.scale
0312:     ct2 = q.loc / q.scale
0313:     ct3 = p.loc / q.scale
0314:     t1 = -ct1.log() - ct2 + ct3
0315:     t2 = ct1 * _euler_gamma
0316:     t3 = torch.exp(ct2 + (1 + ct1).lgamma() - ct3)
0317:     return t1 + t2 + t3 - (1 + _euler_gamma)
0318: 
0319: 
0320: @register_kl(Geometric, Geometric)
0321: def _kl_geometric_geometric(p, q):
0322:     return -p.entropy() - torch.log1p(-q.probs) / p.probs - q.logits
0323: 
0324: 
0325: @register_kl(HalfNormal, HalfNormal)
0326: def _kl_halfnormal_halfnormal(p, q):
0327:     return _kl_normal_normal(p.base_dist, q.base_dist)
0328: 
0329: 
````

- **L297** EN: Returns from `_kl_expfamily_expfamily` with the computed result or updated state. | CN: 从 `_kl_expfamily_expfamily` 返回计算结果或更新后的状态。
- **L298** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L299** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L300** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L301** EN: Defines function `_kl_gamma_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_gamma_gamma`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L302** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L303** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L304** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L305** EN: Assigns or updates `t4`. | CN: 对 `t4` 进行赋值或更新。
- **L306** EN: Returns from `_kl_gamma_gamma` with the computed result or updated state. | CN: 从 `_kl_gamma_gamma` 返回计算结果或更新后的状态。
- **L307** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L308** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L309** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L310** EN: Defines function `_kl_gumbel_gumbel`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_gumbel_gumbel`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L311** EN: Assigns or updates `ct1`. | CN: 对 `ct1` 进行赋值或更新。
- **L312** EN: Assigns or updates `ct2`. | CN: 对 `ct2` 进行赋值或更新。
- **L313** EN: Assigns or updates `ct3`. | CN: 对 `ct3` 进行赋值或更新。
- **L314** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L315** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L316** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L317** EN: Returns from `_kl_gumbel_gumbel` with the computed result or updated state. | CN: 从 `_kl_gumbel_gumbel` 返回计算结果或更新后的状态。
- **L318** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L319** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L320** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L321** EN: Defines function `_kl_geometric_geometric`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_geometric_geometric`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L322** EN: Returns from `_kl_geometric_geometric` with the computed result or updated state. | CN: 从 `_kl_geometric_geometric` 返回计算结果或更新后的状态。
- **L323** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L324** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L325** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L326** EN: Defines function `_kl_halfnormal_halfnormal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_halfnormal_halfnormal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L327** EN: Returns from `_kl_halfnormal_halfnormal` with the computed result or updated state. | CN: 从 `_kl_halfnormal_halfnormal` 返回计算结果或更新后的状态。
- **L328** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L329** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 330-363 / 第 330-363 行

````python
0330: @register_kl(Laplace, Laplace)
0331: def _kl_laplace_laplace(p, q):
0332:     # From http://www.mast.queensu.ca/~communications/Papers/gil-msc11.pdf
0333:     scale_ratio = p.scale / q.scale
0334:     loc_abs_diff = (p.loc - q.loc).abs()
0335:     t1 = -scale_ratio.log()
0336:     t2 = loc_abs_diff / q.scale
0337:     t3 = scale_ratio * torch.exp(-loc_abs_diff / p.scale)
0338:     return t1 + t2 + t3 - 1
0339: 
0340: 
0341: @register_kl(LowRankMultivariateNormal, LowRankMultivariateNormal)
0342: def _kl_lowrankmultivariatenormal_lowrankmultivariatenormal(p, q):
0343:     if p.event_shape != q.event_shape:
0344:         raise ValueError(
0345:             "KL-divergence between two Low Rank Multivariate Normals with\
0346:                           different event shapes cannot be computed"
0347:         )
0348: 
0349:     term1 = _batch_lowrank_logdet(
0350:         q._unbroadcasted_cov_factor, q._unbroadcasted_cov_diag, q._capacitance_tril
0351:     ) - _batch_lowrank_logdet(
0352:         p._unbroadcasted_cov_factor, p._unbroadcasted_cov_diag, p._capacitance_tril
0353:     )
0354:     term3 = _batch_lowrank_mahalanobis(
0355:         q._unbroadcasted_cov_factor,
0356:         q._unbroadcasted_cov_diag,
0357:         q.loc - p.loc,
0358:         q._capacitance_tril,
0359:     )
0360:     # Expands term2 according to
0361:     # inv(qcov) @ pcov = [inv(qD) - inv(qD) @ qW @ inv(qC) @ qW.T @ inv(qD)] @ (pW @ pW.T + pD)
0362:     #                  = [inv(qD) - A.T @ A] @ (pD + pW @ pW.T)
0363:     qWt_qDinv = q._unbroadcasted_cov_factor.mT / q._unbroadcasted_cov_diag.unsqueeze(-2)
````

- **L330** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L331** EN: Defines function `_kl_laplace_laplace`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_laplace_laplace`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L332** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L333** EN: Assigns or updates `scale_ratio`. | CN: 对 `scale_ratio` 进行赋值或更新。
- **L334** EN: Assigns or updates `loc_abs_diff`. | CN: 对 `loc_abs_diff` 进行赋值或更新。
- **L335** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L336** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L337** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L338** EN: Returns from `_kl_laplace_laplace` with the computed result or updated state. | CN: 从 `_kl_laplace_laplace` 返回计算结果或更新后的状态。
- **L339** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L340** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L341** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L342** EN: Defines function `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L343** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L344** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L345** EN: Continues `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L346** EN: Continues `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L347** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L348** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L349** EN: Assigns or updates `term1`. | CN: 对 `term1` 进行赋值或更新。
- **L350** EN: Continues `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L351** EN: Invokes `_batch_lowrank_logdet` to advance the surrounding implementation. | CN: 调用 `_batch_lowrank_logdet` 来推进周围的实现逻辑。
- **L352** EN: Continues `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L353** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L354** EN: Assigns or updates `term3`. | CN: 对 `term3` 进行赋值或更新。
- **L355** EN: Continues `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L356** EN: Continues `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L357** EN: Continues `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L358** EN: Continues `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L359** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L360** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L361** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L362** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L363** EN: Assigns or updates `qWt_qDinv`. | CN: 对 `qWt_qDinv` 进行赋值或更新。

### Lines 364-397 / 第 364-397 行

````python
0364:     A = torch.linalg.solve_triangular(q._capacitance_tril, qWt_qDinv, upper=False)
0365:     term21 = (p._unbroadcasted_cov_diag / q._unbroadcasted_cov_diag).sum(-1)
0366:     term22 = _batch_trace_XXT(
0367:         p._unbroadcasted_cov_factor * q._unbroadcasted_cov_diag.rsqrt().unsqueeze(-1)
0368:     )
0369:     term23 = _batch_trace_XXT(A * p._unbroadcasted_cov_diag.sqrt().unsqueeze(-2))
0370:     term24 = _batch_trace_XXT(A.matmul(p._unbroadcasted_cov_factor))
0371:     term2 = term21 + term22 - term23 - term24
0372:     return 0.5 * (term1 + term2 + term3 - p.event_shape[0])
0373: 
0374: 
0375: @register_kl(MultivariateNormal, LowRankMultivariateNormal)
0376: def _kl_multivariatenormal_lowrankmultivariatenormal(p, q):
0377:     if p.event_shape != q.event_shape:
0378:         raise ValueError(
0379:             "KL-divergence between two (Low Rank) Multivariate Normals with\
0380:                           different event shapes cannot be computed"
0381:         )
0382: 
0383:     term1 = _batch_lowrank_logdet(
0384:         q._unbroadcasted_cov_factor, q._unbroadcasted_cov_diag, q._capacitance_tril
0385:     ) - 2 * p._unbroadcasted_scale_tril.diagonal(dim1=-2, dim2=-1).log().sum(-1)
0386:     term3 = _batch_lowrank_mahalanobis(
0387:         q._unbroadcasted_cov_factor,
0388:         q._unbroadcasted_cov_diag,
0389:         q.loc - p.loc,
0390:         q._capacitance_tril,
0391:     )
0392:     # Expands term2 according to
0393:     # inv(qcov) @ pcov = [inv(qD) - inv(qD) @ qW @ inv(qC) @ qW.T @ inv(qD)] @ p_tril @ p_tril.T
0394:     #                  = [inv(qD) - A.T @ A] @ p_tril @ p_tril.T
0395:     qWt_qDinv = q._unbroadcasted_cov_factor.mT / q._unbroadcasted_cov_diag.unsqueeze(-2)
0396:     A = torch.linalg.solve_triangular(q._capacitance_tril, qWt_qDinv, upper=False)
0397:     term21 = _batch_trace_XXT(
````

- **L364** EN: Assigns module-level configuration or cached state to `A`. | CN: 为 `A` 赋予模块级配置或缓存状态。
- **L365** EN: Assigns or updates `term21`. | CN: 对 `term21` 进行赋值或更新。
- **L366** EN: Assigns or updates `term22`. | CN: 对 `term22` 进行赋值或更新。
- **L367** EN: Invokes `q._unbroadcasted_cov_diag.rsqrt` to advance the surrounding implementation. | CN: 调用 `q._unbroadcasted_cov_diag.rsqrt` 来推进周围的实现逻辑。
- **L368** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L369** EN: Assigns or updates `term23`. | CN: 对 `term23` 进行赋值或更新。
- **L370** EN: Assigns or updates `term24`. | CN: 对 `term24` 进行赋值或更新。
- **L371** EN: Assigns or updates `term2`. | CN: 对 `term2` 进行赋值或更新。
- **L372** EN: Returns from `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal` with the computed result or updated state. | CN: 从 `_kl_lowrankmultivariatenormal_lowrankmultivariatenormal` 返回计算结果或更新后的状态。
- **L373** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L374** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L375** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L376** EN: Defines function `_kl_multivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_multivariatenormal_lowrankmultivariatenormal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L377** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L378** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L379** EN: Invokes `two` to advance the surrounding implementation. | CN: 调用 `two` 来推进周围的实现逻辑。
- **L380** EN: Continues `_kl_multivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_multivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L381** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L382** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L383** EN: Assigns or updates `term1`. | CN: 对 `term1` 进行赋值或更新。
- **L384** EN: Continues `_kl_multivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_multivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L385** EN: Invokes `p._unbroadcasted_scale_tril.diagonal` to advance the surrounding implementation. | CN: 调用 `p._unbroadcasted_scale_tril.diagonal` 来推进周围的实现逻辑。
- **L386** EN: Assigns or updates `term3`. | CN: 对 `term3` 进行赋值或更新。
- **L387** EN: Continues `_kl_multivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_multivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L388** EN: Continues `_kl_multivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_multivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L389** EN: Continues `_kl_multivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_multivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L390** EN: Continues `_kl_multivariatenormal_lowrankmultivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_multivariatenormal_lowrankmultivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L391** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L392** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L393** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L394** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L395** EN: Assigns or updates `qWt_qDinv`. | CN: 对 `qWt_qDinv` 进行赋值或更新。
- **L396** EN: Assigns module-level configuration or cached state to `A`. | CN: 为 `A` 赋予模块级配置或缓存状态。
- **L397** EN: Assigns or updates `term21`. | CN: 对 `term21` 进行赋值或更新。

### Lines 398-431 / 第 398-431 行

````python
0398:         p._unbroadcasted_scale_tril * q._unbroadcasted_cov_diag.rsqrt().unsqueeze(-1)
0399:     )
0400:     term22 = _batch_trace_XXT(A.matmul(p._unbroadcasted_scale_tril))
0401:     term2 = term21 - term22
0402:     return 0.5 * (term1 + term2 + term3 - p.event_shape[0])
0403: 
0404: 
0405: @register_kl(LowRankMultivariateNormal, MultivariateNormal)
0406: def _kl_lowrankmultivariatenormal_multivariatenormal(p, q):
0407:     if p.event_shape != q.event_shape:
0408:         raise ValueError(
0409:             "KL-divergence between two (Low Rank) Multivariate Normals with\
0410:                           different event shapes cannot be computed"
0411:         )
0412: 
0413:     term1 = 2 * q._unbroadcasted_scale_tril.diagonal(dim1=-2, dim2=-1).log().sum(
0414:         -1
0415:     ) - _batch_lowrank_logdet(
0416:         p._unbroadcasted_cov_factor, p._unbroadcasted_cov_diag, p._capacitance_tril
0417:     )
0418:     term3 = _batch_mahalanobis(q._unbroadcasted_scale_tril, (q.loc - p.loc))
0419:     # Expands term2 according to
0420:     # inv(qcov) @ pcov = inv(q_tril @ q_tril.T) @ (pW @ pW.T + pD)
0421:     combined_batch_shape = torch._C._infer_size(
0422:         q._unbroadcasted_scale_tril.shape[:-2], p._unbroadcasted_cov_factor.shape[:-2]
0423:     )
0424:     n = p.event_shape[0]
0425:     q_scale_tril = q._unbroadcasted_scale_tril.expand(combined_batch_shape + (n, n))
0426:     p_cov_factor = p._unbroadcasted_cov_factor.expand(
0427:         combined_batch_shape + (n, p.cov_factor.size(-1))
0428:     )
0429:     p_cov_diag = torch.diag_embed(p._unbroadcasted_cov_diag.sqrt()).expand(
0430:         combined_batch_shape + (n, n)
0431:     )
````

- **L398** EN: Invokes `q._unbroadcasted_cov_diag.rsqrt` to advance the surrounding implementation. | CN: 调用 `q._unbroadcasted_cov_diag.rsqrt` 来推进周围的实现逻辑。
- **L399** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L400** EN: Assigns or updates `term22`. | CN: 对 `term22` 进行赋值或更新。
- **L401** EN: Assigns or updates `term2`. | CN: 对 `term2` 进行赋值或更新。
- **L402** EN: Returns from `_kl_multivariatenormal_lowrankmultivariatenormal` with the computed result or updated state. | CN: 从 `_kl_multivariatenormal_lowrankmultivariatenormal` 返回计算结果或更新后的状态。
- **L403** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L404** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L405** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L406** EN: Defines function `_kl_lowrankmultivariatenormal_multivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_lowrankmultivariatenormal_multivariatenormal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L407** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L408** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L409** EN: Invokes `two` to advance the surrounding implementation. | CN: 调用 `two` 来推进周围的实现逻辑。
- **L410** EN: Continues `_kl_lowrankmultivariatenormal_multivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_multivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L411** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L412** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L413** EN: Assigns or updates `term1`. | CN: 对 `term1` 进行赋值或更新。
- **L414** EN: Continues `_kl_lowrankmultivariatenormal_multivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_multivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L415** EN: Invokes `_batch_lowrank_logdet` to advance the surrounding implementation. | CN: 调用 `_batch_lowrank_logdet` 来推进周围的实现逻辑。
- **L416** EN: Continues `_kl_lowrankmultivariatenormal_multivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_multivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L417** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L418** EN: Assigns or updates `term3`. | CN: 对 `term3` 进行赋值或更新。
- **L419** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L420** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L421** EN: Assigns or updates `combined_batch_shape`. | CN: 对 `combined_batch_shape` 进行赋值或更新。
- **L422** EN: Continues `_kl_lowrankmultivariatenormal_multivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_multivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L423** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L424** EN: Assigns or updates `n`. | CN: 对 `n` 进行赋值或更新。
- **L425** EN: Assigns or updates `q_scale_tril`. | CN: 对 `q_scale_tril` 进行赋值或更新。
- **L426** EN: Assigns or updates `p_cov_factor`. | CN: 对 `p_cov_factor` 进行赋值或更新。
- **L427** EN: Invokes `p.cov_factor.size` to advance the surrounding implementation. | CN: 调用 `p.cov_factor.size` 来推进周围的实现逻辑。
- **L428** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L429** EN: Assigns or updates `p_cov_diag`. | CN: 对 `p_cov_diag` 进行赋值或更新。
- **L430** EN: Continues `_kl_lowrankmultivariatenormal_multivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_lowrankmultivariatenormal_multivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L431** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。

### Lines 432-465 / 第 432-465 行

````python
0432:     term21 = _batch_trace_XXT(
0433:         torch.linalg.solve_triangular(q_scale_tril, p_cov_factor, upper=False)
0434:     )
0435:     term22 = _batch_trace_XXT(
0436:         torch.linalg.solve_triangular(q_scale_tril, p_cov_diag, upper=False)
0437:     )
0438:     term2 = term21 + term22
0439:     return 0.5 * (term1 + term2 + term3 - p.event_shape[0])
0440: 
0441: 
0442: @register_kl(MultivariateNormal, MultivariateNormal)
0443: def _kl_multivariatenormal_multivariatenormal(p, q):
0444:     # From https://en.wikipedia.org/wiki/Multivariate_normal_distribution#Kullback%E2%80%93Leibler_divergence
0445:     if p.event_shape != q.event_shape:
0446:         raise ValueError(
0447:             "KL-divergence between two Multivariate Normals with\
0448:                           different event shapes cannot be computed"
0449:         )
0450: 
0451:     half_term1 = q._unbroadcasted_scale_tril.diagonal(dim1=-2, dim2=-1).log().sum(
0452:         -1
0453:     ) - p._unbroadcasted_scale_tril.diagonal(dim1=-2, dim2=-1).log().sum(-1)
0454:     combined_batch_shape = torch._C._infer_size(
0455:         q._unbroadcasted_scale_tril.shape[:-2], p._unbroadcasted_scale_tril.shape[:-2]
0456:     )
0457:     n = p.event_shape[0]
0458:     q_scale_tril = q._unbroadcasted_scale_tril.expand(combined_batch_shape + (n, n))
0459:     p_scale_tril = p._unbroadcasted_scale_tril.expand(combined_batch_shape + (n, n))
0460:     term2 = _batch_trace_XXT(
0461:         torch.linalg.solve_triangular(q_scale_tril, p_scale_tril, upper=False)
0462:     )
0463:     term3 = _batch_mahalanobis(q._unbroadcasted_scale_tril, (q.loc - p.loc))
0464:     return half_term1 + 0.5 * (term2 + term3 - n)
0465: 
````

- **L432** EN: Assigns or updates `term21`. | CN: 对 `term21` 进行赋值或更新。
- **L433** EN: Invokes `torch.linalg.solve_triangular` to advance the surrounding implementation. | CN: 调用 `torch.linalg.solve_triangular` 来推进周围的实现逻辑。
- **L434** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L435** EN: Assigns or updates `term22`. | CN: 对 `term22` 进行赋值或更新。
- **L436** EN: Invokes `torch.linalg.solve_triangular` to advance the surrounding implementation. | CN: 调用 `torch.linalg.solve_triangular` 来推进周围的实现逻辑。
- **L437** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L438** EN: Assigns or updates `term2`. | CN: 对 `term2` 进行赋值或更新。
- **L439** EN: Returns from `_kl_lowrankmultivariatenormal_multivariatenormal` with the computed result or updated state. | CN: 从 `_kl_lowrankmultivariatenormal_multivariatenormal` 返回计算结果或更新后的状态。
- **L440** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L441** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L442** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L443** EN: Defines function `_kl_multivariatenormal_multivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_multivariatenormal_multivariatenormal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L444** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L445** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L446** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L447** EN: Continues `_kl_multivariatenormal_multivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_multivariatenormal_multivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L448** EN: Continues `_kl_multivariatenormal_multivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_multivariatenormal_multivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L449** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L450** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L451** EN: Assigns or updates `half_term1`. | CN: 对 `half_term1` 进行赋值或更新。
- **L452** EN: Continues `_kl_multivariatenormal_multivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_multivariatenormal_multivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L453** EN: Invokes `p._unbroadcasted_scale_tril.diagonal` to advance the surrounding implementation. | CN: 调用 `p._unbroadcasted_scale_tril.diagonal` 来推进周围的实现逻辑。
- **L454** EN: Assigns or updates `combined_batch_shape`. | CN: 对 `combined_batch_shape` 进行赋值或更新。
- **L455** EN: Continues `_kl_multivariatenormal_multivariatenormal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_multivariatenormal_multivariatenormal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L456** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L457** EN: Assigns or updates `n`. | CN: 对 `n` 进行赋值或更新。
- **L458** EN: Assigns or updates `q_scale_tril`. | CN: 对 `q_scale_tril` 进行赋值或更新。
- **L459** EN: Assigns or updates `p_scale_tril`. | CN: 对 `p_scale_tril` 进行赋值或更新。
- **L460** EN: Assigns or updates `term2`. | CN: 对 `term2` 进行赋值或更新。
- **L461** EN: Invokes `torch.linalg.solve_triangular` to advance the surrounding implementation. | CN: 调用 `torch.linalg.solve_triangular` 来推进周围的实现逻辑。
- **L462** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L463** EN: Assigns or updates `term3`. | CN: 对 `term3` 进行赋值或更新。
- **L464** EN: Returns from `_kl_multivariatenormal_multivariatenormal` with the computed result or updated state. | CN: 从 `_kl_multivariatenormal_multivariatenormal` 返回计算结果或更新后的状态。
- **L465** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 466-495 / 第 466-495 行

````python
0466: 
0467: @register_kl(Normal, Normal)
0468: def _kl_normal_normal(p, q):
0469:     var_ratio = (p.scale / q.scale).pow(2)
0470:     t1 = ((p.loc - q.loc) / q.scale).pow(2)
0471:     return 0.5 * (var_ratio + t1 - 1 - var_ratio.log())
0472: 
0473: 
0474: @register_kl(OneHotCategorical, OneHotCategorical)
0475: def _kl_onehotcategorical_onehotcategorical(p, q):
0476:     return _kl_categorical_categorical(p._categorical, q._categorical)
0477: 
0478: 
0479: @register_kl(Pareto, Pareto)
0480: def _kl_pareto_pareto(p, q):
0481:     # From http://www.mast.queensu.ca/~communications/Papers/gil-msc11.pdf
0482:     scale_ratio = p.scale / q.scale
0483:     alpha_ratio = q.alpha / p.alpha
0484:     t1 = q.alpha * scale_ratio.log()
0485:     t2 = -alpha_ratio.log()
0486:     result = t1 + t2 + alpha_ratio - 1
0487:     result[p.support.lower_bound < q.support.lower_bound] = inf
0488:     return result
0489: 
0490: 
0491: @register_kl(Poisson, Poisson)
0492: def _kl_poisson_poisson(p, q):
0493:     return p.rate * (p.rate.log() - q.rate.log()) - (p.rate - q.rate)
0494: 
0495: 
````

- **L466** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L467** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L468** EN: Defines function `_kl_normal_normal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_normal_normal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L469** EN: Assigns or updates `var_ratio`. | CN: 对 `var_ratio` 进行赋值或更新。
- **L470** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L471** EN: Returns from `_kl_normal_normal` with the computed result or updated state. | CN: 从 `_kl_normal_normal` 返回计算结果或更新后的状态。
- **L472** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L473** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L474** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L475** EN: Defines function `_kl_onehotcategorical_onehotcategorical`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_onehotcategorical_onehotcategorical`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L476** EN: Returns from `_kl_onehotcategorical_onehotcategorical` with the computed result or updated state. | CN: 从 `_kl_onehotcategorical_onehotcategorical` 返回计算结果或更新后的状态。
- **L477** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L478** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L479** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L480** EN: Defines function `_kl_pareto_pareto`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_pareto_pareto`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L481** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L482** EN: Assigns or updates `scale_ratio`. | CN: 对 `scale_ratio` 进行赋值或更新。
- **L483** EN: Assigns or updates `alpha_ratio`. | CN: 对 `alpha_ratio` 进行赋值或更新。
- **L484** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L485** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L486** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L487** EN: Continues `_kl_pareto_pareto`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_pareto_pareto` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L488** EN: Returns from `_kl_pareto_pareto` with the computed result or updated state. | CN: 从 `_kl_pareto_pareto` 返回计算结果或更新后的状态。
- **L489** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L490** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L491** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L492** EN: Defines function `_kl_poisson_poisson`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_poisson_poisson`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L493** EN: Returns from `_kl_poisson_poisson` with the computed result or updated state. | CN: 从 `_kl_poisson_poisson` 返回计算结果或更新后的状态。
- **L494** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L495** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 496-527 / 第 496-527 行

````python
0496: @register_kl(TransformedDistribution, TransformedDistribution)
0497: def _kl_transformed_transformed(p, q):
0498:     if p.transforms != q.transforms:
0499:         raise NotImplementedError
0500:     if p.event_shape != q.event_shape:
0501:         raise NotImplementedError
0502:     return kl_divergence(p.base_dist, q.base_dist)
0503: 
0504: 
0505: @register_kl(Uniform, Uniform)
0506: def _kl_uniform_uniform(p, q):
0507:     result = ((q.high - q.low) / (p.high - p.low)).log()
0508:     result[(q.low > p.low) | (q.high < p.high)] = inf
0509:     return result
0510: 
0511: 
0512: # Different distributions
0513: @register_kl(Bernoulli, Poisson)
0514: def _kl_bernoulli_poisson(p, q):
0515:     return -p.entropy() - (p.probs * q.rate.log() - q.rate)
0516: 
0517: 
0518: @register_kl(Beta, ContinuousBernoulli)
0519: def _kl_beta_continuous_bernoulli(p, q):
0520:     return (
0521:         -p.entropy()
0522:         - p.mean * q.logits
0523:         - torch.log1p(-q.probs)
0524:         - q._cont_bern_log_norm()
0525:     )
0526: 
0527: 
````

- **L496** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L497** EN: Defines function `_kl_transformed_transformed`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_transformed_transformed`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L498** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L499** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L500** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L501** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L502** EN: Returns from `_kl_transformed_transformed` with the computed result or updated state. | CN: 从 `_kl_transformed_transformed` 返回计算结果或更新后的状态。
- **L503** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L504** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L505** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L506** EN: Defines function `_kl_uniform_uniform`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_uniform_uniform`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L507** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L508** EN: Continues `_kl_uniform_uniform`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_uniform` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L509** EN: Returns from `_kl_uniform_uniform` with the computed result or updated state. | CN: 从 `_kl_uniform_uniform` 返回计算结果或更新后的状态。
- **L510** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L511** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L512** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L513** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L514** EN: Defines function `_kl_bernoulli_poisson`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_bernoulli_poisson`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L515** EN: Returns from `_kl_bernoulli_poisson` with the computed result or updated state. | CN: 从 `_kl_bernoulli_poisson` 返回计算结果或更新后的状态。
- **L516** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L517** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L518** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L519** EN: Defines function `_kl_beta_continuous_bernoulli`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_beta_continuous_bernoulli`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L520** EN: Returns from `_kl_beta_continuous_bernoulli` with the computed result or updated state. | CN: 从 `_kl_beta_continuous_bernoulli` 返回计算结果或更新后的状态。
- **L521** EN: Invokes `p.entropy` to advance the surrounding implementation. | CN: 调用 `p.entropy` 来推进周围的实现逻辑。
- **L522** EN: Continues `_kl_beta_continuous_bernoulli`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_beta_continuous_bernoulli` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L523** EN: Invokes `torch.log1p` to advance the surrounding implementation. | CN: 调用 `torch.log1p` 来推进周围的实现逻辑。
- **L524** EN: Invokes `q._cont_bern_log_norm` to advance the surrounding implementation. | CN: 调用 `q._cont_bern_log_norm` 来推进周围的实现逻辑。
- **L525** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L526** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L527** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 528-555 / 第 528-555 行

````python
0528: @register_kl(Beta, Pareto)
0529: def _kl_beta_infinity(p, q):
0530:     return _infinite_like(p.concentration1)
0531: 
0532: 
0533: @register_kl(Beta, Exponential)
0534: def _kl_beta_exponential(p, q):
0535:     return (
0536:         -p.entropy()
0537:         - q.rate.log()
0538:         + q.rate * (p.concentration1 / (p.concentration1 + p.concentration0))
0539:     )
0540: 
0541: 
0542: @register_kl(Beta, Gamma)
0543: def _kl_beta_gamma(p, q):
0544:     t1 = -p.entropy()
0545:     t2 = q.concentration.lgamma() - q.concentration * q.rate.log()
0546:     t3 = (q.concentration - 1) * (
0547:         p.concentration1.digamma() - (p.concentration1 + p.concentration0).digamma()
0548:     )
0549:     t4 = q.rate * p.concentration1 / (p.concentration1 + p.concentration0)
0550:     return t1 + t2 - t3 + t4
0551: 
0552: 
0553: # TODO: Add Beta-Laplace KL Divergence
0554: 
0555: 
````

- **L528** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L529** EN: Defines function `_kl_beta_infinity`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_beta_infinity`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L530** EN: Returns from `_kl_beta_infinity` with the computed result or updated state. | CN: 从 `_kl_beta_infinity` 返回计算结果或更新后的状态。
- **L531** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L532** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L533** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L534** EN: Defines function `_kl_beta_exponential`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_beta_exponential`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L535** EN: Returns from `_kl_beta_exponential` with the computed result or updated state. | CN: 从 `_kl_beta_exponential` 返回计算结果或更新后的状态。
- **L536** EN: Invokes `p.entropy` to advance the surrounding implementation. | CN: 调用 `p.entropy` 来推进周围的实现逻辑。
- **L537** EN: Invokes `q.rate.log` to advance the surrounding implementation. | CN: 调用 `q.rate.log` 来推进周围的实现逻辑。
- **L538** EN: Continues `_kl_beta_exponential`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_beta_exponential` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L539** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L540** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L541** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L542** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L543** EN: Defines function `_kl_beta_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_beta_gamma`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L544** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L545** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L546** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L547** EN: Invokes `p.concentration1.digamma` to advance the surrounding implementation. | CN: 调用 `p.concentration1.digamma` 来推进周围的实现逻辑。
- **L548** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L549** EN: Assigns or updates `t4`. | CN: 对 `t4` 进行赋值或更新。
- **L550** EN: Returns from `_kl_beta_gamma` with the computed result or updated state. | CN: 从 `_kl_beta_gamma` 返回计算结果或更新后的状态。
- **L551** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L552** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L553** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L554** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L555** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 556-589 / 第 556-589 行

````python
0556: @register_kl(Beta, Normal)
0557: def _kl_beta_normal(p, q):
0558:     E_beta = p.concentration1 / (p.concentration1 + p.concentration0)
0559:     var_normal = q.scale.pow(2)
0560:     t1 = -p.entropy()
0561:     t2 = 0.5 * (var_normal * 2 * math.pi).log()
0562:     t3 = (
0563:         E_beta * (1 - E_beta) / (p.concentration1 + p.concentration0 + 1)
0564:         + E_beta.pow(2)
0565:     ) * 0.5
0566:     t4 = q.loc * E_beta
0567:     t5 = q.loc.pow(2) * 0.5
0568:     return t1 + t2 + (t3 - t4 + t5) / var_normal
0569: 
0570: 
0571: @register_kl(Beta, Uniform)
0572: def _kl_beta_uniform(p, q):
0573:     result = -p.entropy() + (q.high - q.low).log()
0574:     result[(q.low > p.support.lower_bound) | (q.high < p.support.upper_bound)] = inf
0575:     return result
0576: 
0577: 
0578: # Note that the KL between a ContinuousBernoulli and Beta has no closed form
0579: 
0580: 
0581: @register_kl(ContinuousBernoulli, Pareto)
0582: def _kl_continuous_bernoulli_infinity(p, q):
0583:     return _infinite_like(p.probs)
0584: 
0585: 
0586: @register_kl(ContinuousBernoulli, Exponential)
0587: def _kl_continuous_bernoulli_exponential(p, q):
0588:     return -p.entropy() - torch.log(q.rate) + q.rate * p.mean
0589: 
````

- **L556** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L557** EN: Defines function `_kl_beta_normal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_beta_normal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L558** EN: Assigns or updates `E_beta`. | CN: 对 `E_beta` 进行赋值或更新。
- **L559** EN: Assigns or updates `var_normal`. | CN: 对 `var_normal` 进行赋值或更新。
- **L560** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L561** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L562** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L563** EN: Continues `_kl_beta_normal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_beta_normal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L564** EN: Invokes `E_beta.pow` to advance the surrounding implementation. | CN: 调用 `E_beta.pow` 来推进周围的实现逻辑。
- **L565** EN: Continues `_kl_beta_normal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_beta_normal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L566** EN: Assigns or updates `t4`. | CN: 对 `t4` 进行赋值或更新。
- **L567** EN: Assigns or updates `t5`. | CN: 对 `t5` 进行赋值或更新。
- **L568** EN: Returns from `_kl_beta_normal` with the computed result or updated state. | CN: 从 `_kl_beta_normal` 返回计算结果或更新后的状态。
- **L569** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L570** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L571** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L572** EN: Defines function `_kl_beta_uniform`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_beta_uniform`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L573** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L574** EN: Continues `_kl_beta_uniform`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_beta_uniform` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L575** EN: Returns from `_kl_beta_uniform` with the computed result or updated state. | CN: 从 `_kl_beta_uniform` 返回计算结果或更新后的状态。
- **L576** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L577** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L578** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L579** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L580** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L581** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L582** EN: Defines function `_kl_continuous_bernoulli_infinity`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_continuous_bernoulli_infinity`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L583** EN: Returns from `_kl_continuous_bernoulli_infinity` with the computed result or updated state. | CN: 从 `_kl_continuous_bernoulli_infinity` 返回计算结果或更新后的状态。
- **L584** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L585** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L586** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L587** EN: Defines function `_kl_continuous_bernoulli_exponential`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_continuous_bernoulli_exponential`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L588** EN: Returns from `_kl_continuous_bernoulli_exponential` with the computed result or updated state. | CN: 从 `_kl_continuous_bernoulli_exponential` 返回计算结果或更新后的状态。
- **L589** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 590-619 / 第 590-619 行

````python
0590: 
0591: # Note that the KL between a ContinuousBernoulli and Gamma has no closed form
0592: # TODO: Add ContinuousBernoulli-Laplace KL Divergence
0593: 
0594: 
0595: @register_kl(ContinuousBernoulli, Normal)
0596: def _kl_continuous_bernoulli_normal(p, q):
0597:     t1 = -p.entropy()
0598:     t2 = 0.5 * (math.log(2.0 * math.pi) + torch.square(q.loc / q.scale)) + torch.log(
0599:         q.scale
0600:     )
0601:     t3 = (p.variance + torch.square(p.mean) - 2.0 * q.loc * p.mean) / (
0602:         2.0 * torch.square(q.scale)
0603:     )
0604:     return t1 + t2 + t3
0605: 
0606: 
0607: @register_kl(ContinuousBernoulli, Uniform)
0608: def _kl_continuous_bernoulli_uniform(p, q):
0609:     result = -p.entropy() + (q.high - q.low).log()
0610:     return torch.where(
0611:         torch.max(
0612:             torch.ge(q.low, p.support.lower_bound),
0613:             torch.le(q.high, p.support.upper_bound),
0614:         ),
0615:         torch.ones_like(result) * inf,
0616:         result,
0617:     )
0618: 
0619: 
````

- **L590** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L591** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L592** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L593** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L594** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L595** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L596** EN: Defines function `_kl_continuous_bernoulli_normal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_continuous_bernoulli_normal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L597** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L598** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L599** EN: Continues `_kl_continuous_bernoulli_normal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_continuous_bernoulli_normal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L600** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L601** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L602** EN: Invokes `torch.square` to advance the surrounding implementation. | CN: 调用 `torch.square` 来推进周围的实现逻辑。
- **L603** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L604** EN: Returns from `_kl_continuous_bernoulli_normal` with the computed result or updated state. | CN: 从 `_kl_continuous_bernoulli_normal` 返回计算结果或更新后的状态。
- **L605** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L606** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L607** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L608** EN: Defines function `_kl_continuous_bernoulli_uniform`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_continuous_bernoulli_uniform`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L609** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L610** EN: Returns from `_kl_continuous_bernoulli_uniform` with the computed result or updated state. | CN: 从 `_kl_continuous_bernoulli_uniform` 返回计算结果或更新后的状态。
- **L611** EN: Invokes `torch.max` to advance the surrounding implementation. | CN: 调用 `torch.max` 来推进周围的实现逻辑。
- **L612** EN: Invokes `torch.ge` to advance the surrounding implementation. | CN: 调用 `torch.ge` 来推进周围的实现逻辑。
- **L613** EN: Invokes `torch.le` to advance the surrounding implementation. | CN: 调用 `torch.le` 来推进周围的实现逻辑。
- **L614** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L615** EN: Invokes `torch.ones_like` to advance the surrounding implementation. | CN: 调用 `torch.ones_like` 来推进周围的实现逻辑。
- **L616** EN: Continues `_kl_continuous_bernoulli_uniform`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_continuous_bernoulli_uniform` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L617** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L618** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L619** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 620-653 / 第 620-653 行

````python
0620: @register_kl(Exponential, Beta)
0621: @register_kl(Exponential, ContinuousBernoulli)
0622: @register_kl(Exponential, Pareto)
0623: @register_kl(Exponential, Uniform)
0624: def _kl_exponential_infinity(p, q):
0625:     return _infinite_like(p.rate)
0626: 
0627: 
0628: @register_kl(Exponential, Gamma)
0629: def _kl_exponential_gamma(p, q):
0630:     ratio = q.rate / p.rate
0631:     t1 = -q.concentration * torch.log(ratio)
0632:     return (
0633:         t1
0634:         + ratio
0635:         + q.concentration.lgamma()
0636:         + q.concentration * _euler_gamma
0637:         - (1 + _euler_gamma)
0638:     )
0639: 
0640: 
0641: @register_kl(Exponential, Gumbel)
0642: def _kl_exponential_gumbel(p, q):
0643:     scale_rate_prod = p.rate * q.scale
0644:     loc_scale_ratio = q.loc / q.scale
0645:     t1 = scale_rate_prod.log() - 1
0646:     t2 = torch.exp(loc_scale_ratio) * scale_rate_prod / (scale_rate_prod + 1)
0647:     t3 = scale_rate_prod.reciprocal()
0648:     return t1 - loc_scale_ratio + t2 + t3
0649: 
0650: 
0651: # TODO: Add Exponential-Laplace KL Divergence
0652: 
0653: 
````

- **L620** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L621** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L622** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L623** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L624** EN: Defines function `_kl_exponential_infinity`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_exponential_infinity`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L625** EN: Returns from `_kl_exponential_infinity` with the computed result or updated state. | CN: 从 `_kl_exponential_infinity` 返回计算结果或更新后的状态。
- **L626** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L627** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L628** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L629** EN: Defines function `_kl_exponential_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_exponential_gamma`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L630** EN: Assigns or updates `ratio`. | CN: 对 `ratio` 进行赋值或更新。
- **L631** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L632** EN: Returns from `_kl_exponential_gamma` with the computed result or updated state. | CN: 从 `_kl_exponential_gamma` 返回计算结果或更新后的状态。
- **L633** EN: Continues `_kl_exponential_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_exponential_gamma` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L634** EN: Continues `_kl_exponential_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_exponential_gamma` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L635** EN: Invokes `q.concentration.lgamma` to advance the surrounding implementation. | CN: 调用 `q.concentration.lgamma` 来推进周围的实现逻辑。
- **L636** EN: Continues `_kl_exponential_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_exponential_gamma` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L637** EN: Continues `_kl_exponential_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_exponential_gamma` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L638** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L639** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L640** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L641** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L642** EN: Defines function `_kl_exponential_gumbel`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_exponential_gumbel`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L643** EN: Assigns or updates `scale_rate_prod`. | CN: 对 `scale_rate_prod` 进行赋值或更新。
- **L644** EN: Assigns or updates `loc_scale_ratio`. | CN: 对 `loc_scale_ratio` 进行赋值或更新。
- **L645** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L646** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L647** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L648** EN: Returns from `_kl_exponential_gumbel` with the computed result or updated state. | CN: 从 `_kl_exponential_gumbel` 返回计算结果或更新后的状态。
- **L649** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L650** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L651** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L652** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L653** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 654-677 / 第 654-677 行

````python
0654: @register_kl(Exponential, Normal)
0655: def _kl_exponential_normal(p, q):
0656:     var_normal = q.scale.pow(2)
0657:     rate_sqr = p.rate.pow(2)
0658:     t1 = 0.5 * torch.log(rate_sqr * var_normal * 2 * math.pi)
0659:     t2 = rate_sqr.reciprocal()
0660:     t3 = q.loc / p.rate
0661:     t4 = q.loc.pow(2) * 0.5
0662:     return t1 - 1 + (t2 - t3 + t4) / var_normal
0663: 
0664: 
0665: @register_kl(Gamma, Beta)
0666: @register_kl(Gamma, ContinuousBernoulli)
0667: @register_kl(Gamma, Pareto)
0668: @register_kl(Gamma, Uniform)
0669: def _kl_gamma_infinity(p, q):
0670:     return _infinite_like(p.concentration)
0671: 
0672: 
0673: @register_kl(Gamma, Exponential)
0674: def _kl_gamma_exponential(p, q):
0675:     return -p.entropy() - q.rate.log() + q.rate * p.concentration / p.rate
0676: 
0677: 
````

- **L654** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L655** EN: Defines function `_kl_exponential_normal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_exponential_normal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L656** EN: Assigns or updates `var_normal`. | CN: 对 `var_normal` 进行赋值或更新。
- **L657** EN: Assigns or updates `rate_sqr`. | CN: 对 `rate_sqr` 进行赋值或更新。
- **L658** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L659** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L660** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L661** EN: Assigns or updates `t4`. | CN: 对 `t4` 进行赋值或更新。
- **L662** EN: Returns from `_kl_exponential_normal` with the computed result or updated state. | CN: 从 `_kl_exponential_normal` 返回计算结果或更新后的状态。
- **L663** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L664** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L665** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L666** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L667** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L668** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L669** EN: Defines function `_kl_gamma_infinity`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_gamma_infinity`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L670** EN: Returns from `_kl_gamma_infinity` with the computed result or updated state. | CN: 从 `_kl_gamma_infinity` 返回计算结果或更新后的状态。
- **L671** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L672** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L673** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L674** EN: Defines function `_kl_gamma_exponential`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_gamma_exponential`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L675** EN: Returns from `_kl_gamma_exponential` with the computed result or updated state. | CN: 从 `_kl_gamma_exponential` 返回计算结果或更新后的状态。
- **L676** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L677** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 678-711 / 第 678-711 行

````python
0678: @register_kl(Gamma, Gumbel)
0679: def _kl_gamma_gumbel(p, q):
0680:     beta_scale_prod = p.rate * q.scale
0681:     loc_scale_ratio = q.loc / q.scale
0682:     t1 = (
0683:         (p.concentration - 1) * p.concentration.digamma()
0684:         - p.concentration.lgamma()
0685:         - p.concentration
0686:     )
0687:     t2 = beta_scale_prod.log() + p.concentration / beta_scale_prod
0688:     t3 = (
0689:         torch.exp(loc_scale_ratio)
0690:         * (1 + beta_scale_prod.reciprocal()).pow(-p.concentration)
0691:         - loc_scale_ratio
0692:     )
0693:     return t1 + t2 + t3
0694: 
0695: 
0696: # TODO: Add Gamma-Laplace KL Divergence
0697: 
0698: 
0699: @register_kl(Gamma, Normal)
0700: def _kl_gamma_normal(p, q):
0701:     var_normal = q.scale.pow(2)
0702:     beta_sqr = p.rate.pow(2)
0703:     t1 = (
0704:         0.5 * torch.log(beta_sqr * var_normal * 2 * math.pi)
0705:         - p.concentration
0706:         - p.concentration.lgamma()
0707:     )
0708:     t2 = 0.5 * (p.concentration.pow(2) + p.concentration) / beta_sqr
0709:     t3 = q.loc * p.concentration / p.rate
0710:     t4 = 0.5 * q.loc.pow(2)
0711:     return (
````

- **L678** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L679** EN: Defines function `_kl_gamma_gumbel`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_gamma_gumbel`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L680** EN: Assigns or updates `beta_scale_prod`. | CN: 对 `beta_scale_prod` 进行赋值或更新。
- **L681** EN: Assigns or updates `loc_scale_ratio`. | CN: 对 `loc_scale_ratio` 进行赋值或更新。
- **L682** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L683** EN: Invokes `p.concentration.digamma` to advance the surrounding implementation. | CN: 调用 `p.concentration.digamma` 来推进周围的实现逻辑。
- **L684** EN: Invokes `p.concentration.lgamma` to advance the surrounding implementation. | CN: 调用 `p.concentration.lgamma` 来推进周围的实现逻辑。
- **L685** EN: Continues `_kl_gamma_gumbel`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_gamma_gumbel` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L686** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L687** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L688** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L689** EN: Invokes `torch.exp` to advance the surrounding implementation. | CN: 调用 `torch.exp` 来推进周围的实现逻辑。
- **L690** EN: Invokes `beta_scale_prod.reciprocal` to advance the surrounding implementation. | CN: 调用 `beta_scale_prod.reciprocal` 来推进周围的实现逻辑。
- **L691** EN: Continues `_kl_gamma_gumbel`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_gamma_gumbel` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L692** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L693** EN: Returns from `_kl_gamma_gumbel` with the computed result or updated state. | CN: 从 `_kl_gamma_gumbel` 返回计算结果或更新后的状态。
- **L694** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L695** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L696** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L697** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L698** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L699** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L700** EN: Defines function `_kl_gamma_normal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_gamma_normal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L701** EN: Assigns or updates `var_normal`. | CN: 对 `var_normal` 进行赋值或更新。
- **L702** EN: Assigns or updates `beta_sqr`. | CN: 对 `beta_sqr` 进行赋值或更新。
- **L703** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L704** EN: Invokes `torch.log` to advance the surrounding implementation. | CN: 调用 `torch.log` 来推进周围的实现逻辑。
- **L705** EN: Continues `_kl_gamma_normal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_gamma_normal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L706** EN: Invokes `p.concentration.lgamma` to advance the surrounding implementation. | CN: 调用 `p.concentration.lgamma` 来推进周围的实现逻辑。
- **L707** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L708** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L709** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L710** EN: Assigns or updates `t4`. | CN: 对 `t4` 进行赋值或更新。
- **L711** EN: Returns from `_kl_gamma_normal` with the computed result or updated state. | CN: 从 `_kl_gamma_normal` 返回计算结果或更新后的状态。

### Lines 712-739 / 第 712-739 行

````python
0712:         t1
0713:         + (p.concentration - 1) * p.concentration.digamma()
0714:         + (t2 - t3 + t4) / var_normal
0715:     )
0716: 
0717: 
0718: @register_kl(Gumbel, Beta)
0719: @register_kl(Gumbel, ContinuousBernoulli)
0720: @register_kl(Gumbel, Exponential)
0721: @register_kl(Gumbel, Gamma)
0722: @register_kl(Gumbel, Pareto)
0723: @register_kl(Gumbel, Uniform)
0724: def _kl_gumbel_infinity(p, q):
0725:     return _infinite_like(p.loc)
0726: 
0727: 
0728: # TODO: Add Gumbel-Laplace KL Divergence
0729: 
0730: 
0731: @register_kl(Gumbel, Normal)
0732: def _kl_gumbel_normal(p, q):
0733:     param_ratio = p.scale / q.scale
0734:     t1 = (param_ratio / math.sqrt(2 * math.pi)).log()
0735:     t2 = (math.pi * param_ratio * 0.5).pow(2) / 3
0736:     t3 = ((p.loc + p.scale * _euler_gamma - q.loc) / q.scale).pow(2) * 0.5
0737:     return -t1 + t2 + t3 - (_euler_gamma + 1)
0738: 
0739: 
````

- **L712** EN: Continues `_kl_gamma_normal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_gamma_normal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L713** EN: Invokes `p.concentration.digamma` to advance the surrounding implementation. | CN: 调用 `p.concentration.digamma` 来推进周围的实现逻辑。
- **L714** EN: Continues `_kl_gamma_normal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_gamma_normal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L715** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L716** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L717** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L718** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L719** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L720** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L721** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L722** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L723** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L724** EN: Defines function `_kl_gumbel_infinity`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_gumbel_infinity`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L725** EN: Returns from `_kl_gumbel_infinity` with the computed result or updated state. | CN: 从 `_kl_gumbel_infinity` 返回计算结果或更新后的状态。
- **L726** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L727** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L728** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L729** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L730** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L731** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L732** EN: Defines function `_kl_gumbel_normal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_gumbel_normal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L733** EN: Assigns or updates `param_ratio`. | CN: 对 `param_ratio` 进行赋值或更新。
- **L734** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L735** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L736** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L737** EN: Returns from `_kl_gumbel_normal` with the computed result or updated state. | CN: 从 `_kl_gumbel_normal` 返回计算结果或更新后的状态。
- **L738** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L739** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 740-770 / 第 740-770 行

````python
0740: @register_kl(Laplace, Beta)
0741: @register_kl(Laplace, ContinuousBernoulli)
0742: @register_kl(Laplace, Exponential)
0743: @register_kl(Laplace, Gamma)
0744: @register_kl(Laplace, Pareto)
0745: @register_kl(Laplace, Uniform)
0746: def _kl_laplace_infinity(p, q):
0747:     return _infinite_like(p.loc)
0748: 
0749: 
0750: @register_kl(Laplace, Normal)
0751: def _kl_laplace_normal(p, q):
0752:     var_normal = q.scale.pow(2)
0753:     scale_sqr_var_ratio = p.scale.pow(2) / var_normal
0754:     t1 = 0.5 * torch.log(2 * scale_sqr_var_ratio / math.pi)
0755:     t2 = 0.5 * p.loc.pow(2)
0756:     t3 = p.loc * q.loc
0757:     t4 = 0.5 * q.loc.pow(2)
0758:     return -t1 + scale_sqr_var_ratio + (t2 - t3 + t4) / var_normal - 1
0759: 
0760: 
0761: @register_kl(Normal, Beta)
0762: @register_kl(Normal, ContinuousBernoulli)
0763: @register_kl(Normal, Exponential)
0764: @register_kl(Normal, Gamma)
0765: @register_kl(Normal, Pareto)
0766: @register_kl(Normal, Uniform)
0767: def _kl_normal_infinity(p, q):
0768:     return _infinite_like(p.loc)
0769: 
0770: 
````

- **L740** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L741** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L742** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L743** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L744** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L745** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L746** EN: Defines function `_kl_laplace_infinity`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_laplace_infinity`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L747** EN: Returns from `_kl_laplace_infinity` with the computed result or updated state. | CN: 从 `_kl_laplace_infinity` 返回计算结果或更新后的状态。
- **L748** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L749** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L750** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L751** EN: Defines function `_kl_laplace_normal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_laplace_normal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L752** EN: Assigns or updates `var_normal`. | CN: 对 `var_normal` 进行赋值或更新。
- **L753** EN: Assigns or updates `scale_sqr_var_ratio`. | CN: 对 `scale_sqr_var_ratio` 进行赋值或更新。
- **L754** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L755** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L756** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L757** EN: Assigns or updates `t4`. | CN: 对 `t4` 进行赋值或更新。
- **L758** EN: Returns from `_kl_laplace_normal` with the computed result or updated state. | CN: 从 `_kl_laplace_normal` 返回计算结果或更新后的状态。
- **L759** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L760** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L761** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L762** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L763** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L764** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L765** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L766** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L767** EN: Defines function `_kl_normal_infinity`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_normal_infinity`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L768** EN: Returns from `_kl_normal_infinity` with the computed result or updated state. | CN: 从 `_kl_normal_infinity` 返回计算结果或更新后的状态。
- **L769** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L770** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 771-801 / 第 771-801 行

````python
0771: @register_kl(Normal, Gumbel)
0772: def _kl_normal_gumbel(p, q):
0773:     mean_scale_ratio = p.loc / q.scale
0774:     var_scale_sqr_ratio = (p.scale / q.scale).pow(2)
0775:     loc_scale_ratio = q.loc / q.scale
0776:     t1 = var_scale_sqr_ratio.log() * 0.5
0777:     t2 = mean_scale_ratio - loc_scale_ratio
0778:     t3 = torch.exp(-mean_scale_ratio + 0.5 * var_scale_sqr_ratio + loc_scale_ratio)
0779:     return -t1 + t2 + t3 - (0.5 * (1 + math.log(2 * math.pi)))
0780: 
0781: 
0782: @register_kl(Normal, Laplace)
0783: def _kl_normal_laplace(p, q):
0784:     loc_diff = p.loc - q.loc
0785:     scale_ratio = p.scale / q.scale
0786:     loc_diff_scale_ratio = loc_diff / p.scale
0787:     t1 = torch.log(scale_ratio)
0788:     t2 = (
0789:         math.sqrt(2 / math.pi) * p.scale * torch.exp(-0.5 * loc_diff_scale_ratio.pow(2))
0790:     )
0791:     t3 = loc_diff * torch.erf(math.sqrt(0.5) * loc_diff_scale_ratio)
0792:     return -t1 + (t2 + t3) / q.scale - (0.5 * (1 + math.log(0.5 * math.pi)))
0793: 
0794: 
0795: @register_kl(Pareto, Beta)
0796: @register_kl(Pareto, ContinuousBernoulli)
0797: @register_kl(Pareto, Uniform)
0798: def _kl_pareto_infinity(p, q):
0799:     return _infinite_like(p.scale)
0800: 
0801: 
````

- **L771** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L772** EN: Defines function `_kl_normal_gumbel`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_normal_gumbel`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L773** EN: Assigns or updates `mean_scale_ratio`. | CN: 对 `mean_scale_ratio` 进行赋值或更新。
- **L774** EN: Assigns or updates `var_scale_sqr_ratio`. | CN: 对 `var_scale_sqr_ratio` 进行赋值或更新。
- **L775** EN: Assigns or updates `loc_scale_ratio`. | CN: 对 `loc_scale_ratio` 进行赋值或更新。
- **L776** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L777** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L778** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L779** EN: Returns from `_kl_normal_gumbel` with the computed result or updated state. | CN: 从 `_kl_normal_gumbel` 返回计算结果或更新后的状态。
- **L780** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L781** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L782** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L783** EN: Defines function `_kl_normal_laplace`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_normal_laplace`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L784** EN: Assigns or updates `loc_diff`. | CN: 对 `loc_diff` 进行赋值或更新。
- **L785** EN: Assigns or updates `scale_ratio`. | CN: 对 `scale_ratio` 进行赋值或更新。
- **L786** EN: Assigns or updates `loc_diff_scale_ratio`. | CN: 对 `loc_diff_scale_ratio` 进行赋值或更新。
- **L787** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L788** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L789** EN: Invokes `math.sqrt` to advance the surrounding implementation. | CN: 调用 `math.sqrt` 来推进周围的实现逻辑。
- **L790** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L791** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L792** EN: Returns from `_kl_normal_laplace` with the computed result or updated state. | CN: 从 `_kl_normal_laplace` 返回计算结果或更新后的状态。
- **L793** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L794** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L795** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L796** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L797** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L798** EN: Defines function `_kl_pareto_infinity`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_pareto_infinity`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L799** EN: Returns from `_kl_pareto_infinity` with the computed result or updated state. | CN: 从 `_kl_pareto_infinity` 返回计算结果或更新后的状态。
- **L800** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L801** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 802-827 / 第 802-827 行

````python
0802: @register_kl(Pareto, Exponential)
0803: def _kl_pareto_exponential(p, q):
0804:     scale_rate_prod = p.scale * q.rate
0805:     t1 = (p.alpha / scale_rate_prod).log()
0806:     t2 = p.alpha.reciprocal()
0807:     t3 = p.alpha * scale_rate_prod / (p.alpha - 1)
0808:     result = t1 - t2 + t3 - 1
0809:     result[p.alpha <= 1] = inf
0810:     return result
0811: 
0812: 
0813: @register_kl(Pareto, Gamma)
0814: def _kl_pareto_gamma(p, q):
0815:     common_term = p.scale.log() + p.alpha.reciprocal()
0816:     t1 = p.alpha.log() - common_term
0817:     t2 = q.concentration.lgamma() - q.concentration * q.rate.log()
0818:     t3 = (1 - q.concentration) * common_term
0819:     t4 = q.rate * p.alpha * p.scale / (p.alpha - 1)
0820:     result = t1 + t2 + t3 + t4 - 1
0821:     result[p.alpha <= 1] = inf
0822:     return result
0823: 
0824: 
0825: # TODO: Add Pareto-Laplace KL Divergence
0826: 
0827: 
````

- **L802** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L803** EN: Defines function `_kl_pareto_exponential`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_pareto_exponential`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L804** EN: Assigns or updates `scale_rate_prod`. | CN: 对 `scale_rate_prod` 进行赋值或更新。
- **L805** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L806** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L807** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L808** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L809** EN: Continues `_kl_pareto_exponential`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_pareto_exponential` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L810** EN: Returns from `_kl_pareto_exponential` with the computed result or updated state. | CN: 从 `_kl_pareto_exponential` 返回计算结果或更新后的状态。
- **L811** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L812** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L813** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L814** EN: Defines function `_kl_pareto_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_pareto_gamma`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L815** EN: Assigns or updates `common_term`. | CN: 对 `common_term` 进行赋值或更新。
- **L816** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L817** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L818** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L819** EN: Assigns or updates `t4`. | CN: 对 `t4` 进行赋值或更新。
- **L820** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L821** EN: Continues `_kl_pareto_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_pareto_gamma` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L822** EN: Returns from `_kl_pareto_gamma` with the computed result or updated state. | CN: 从 `_kl_pareto_gamma` 返回计算结果或更新后的状态。
- **L823** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L824** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L825** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L826** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L827** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 828-861 / 第 828-861 行

````python
0828: @register_kl(Pareto, Normal)
0829: def _kl_pareto_normal(p, q):
0830:     var_normal = 2 * q.scale.pow(2)
0831:     common_term = p.scale / (p.alpha - 1)
0832:     t1 = (math.sqrt(2 * math.pi) * q.scale * p.alpha / p.scale).log()
0833:     t2 = p.alpha.reciprocal()
0834:     t3 = p.alpha * common_term.pow(2) / (p.alpha - 2)
0835:     t4 = (p.alpha * common_term - q.loc).pow(2)
0836:     result = t1 - t2 + (t3 + t4) / var_normal - 1
0837:     result[p.alpha <= 2] = inf
0838:     return result
0839: 
0840: 
0841: @register_kl(Poisson, Bernoulli)
0842: @register_kl(Poisson, Binomial)
0843: def _kl_poisson_infinity(p, q):
0844:     return _infinite_like(p.rate)
0845: 
0846: 
0847: @register_kl(Uniform, Beta)
0848: def _kl_uniform_beta(p, q):
0849:     common_term = p.high - p.low
0850:     t1 = torch.log(common_term)
0851:     t2 = (
0852:         (q.concentration1 - 1)
0853:         * (_x_log_x(p.high) - _x_log_x(p.low) - common_term)
0854:         / common_term
0855:     )
0856:     t3 = (
0857:         (q.concentration0 - 1)
0858:         * (_x_log_x(1 - p.high) - _x_log_x(1 - p.low) + common_term)
0859:         / common_term
0860:     )
0861:     t4 = (
````

- **L828** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L829** EN: Defines function `_kl_pareto_normal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_pareto_normal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L830** EN: Assigns or updates `var_normal`. | CN: 对 `var_normal` 进行赋值或更新。
- **L831** EN: Assigns or updates `common_term`. | CN: 对 `common_term` 进行赋值或更新。
- **L832** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L833** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L834** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L835** EN: Assigns or updates `t4`. | CN: 对 `t4` 进行赋值或更新。
- **L836** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L837** EN: Continues `_kl_pareto_normal`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_pareto_normal` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L838** EN: Returns from `_kl_pareto_normal` with the computed result or updated state. | CN: 从 `_kl_pareto_normal` 返回计算结果或更新后的状态。
- **L839** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L840** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L841** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L842** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L843** EN: Defines function `_kl_poisson_infinity`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_poisson_infinity`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L844** EN: Returns from `_kl_poisson_infinity` with the computed result or updated state. | CN: 从 `_kl_poisson_infinity` 返回计算结果或更新后的状态。
- **L845** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L846** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L847** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L848** EN: Defines function `_kl_uniform_beta`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_uniform_beta`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L849** EN: Assigns or updates `common_term`. | CN: 对 `common_term` 进行赋值或更新。
- **L850** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L851** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L852** EN: Continues `_kl_uniform_beta`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_beta` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L853** EN: Invokes `_x_log_x` to advance the surrounding implementation. | CN: 调用 `_x_log_x` 来推进周围的实现逻辑。
- **L854** EN: Continues `_kl_uniform_beta`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_beta` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L855** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L856** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L857** EN: Continues `_kl_uniform_beta`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_beta` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L858** EN: Invokes `_x_log_x` to advance the surrounding implementation. | CN: 调用 `_x_log_x` 来推进周围的实现逻辑。
- **L859** EN: Continues `_kl_uniform_beta`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_beta` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L860** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L861** EN: Assigns or updates `t4`. | CN: 对 `t4` 进行赋值或更新。

### Lines 862-895 / 第 862-895 行

````python
0862:         q.concentration1.lgamma()
0863:         + q.concentration0.lgamma()
0864:         - (q.concentration1 + q.concentration0).lgamma()
0865:     )
0866:     result = t3 + t4 - t1 - t2
0867:     result[(p.high > q.support.upper_bound) | (p.low < q.support.lower_bound)] = inf
0868:     return result
0869: 
0870: 
0871: @register_kl(Uniform, ContinuousBernoulli)
0872: def _kl_uniform_continuous_bernoulli(p, q):
0873:     result = (
0874:         -p.entropy()
0875:         - p.mean * q.logits
0876:         - torch.log1p(-q.probs)
0877:         - q._cont_bern_log_norm()
0878:     )
0879:     return torch.where(
0880:         torch.max(
0881:             torch.ge(p.high, q.support.upper_bound),
0882:             torch.le(p.low, q.support.lower_bound),
0883:         ),
0884:         torch.ones_like(result) * inf,
0885:         result,
0886:     )
0887: 
0888: 
0889: @register_kl(Uniform, Exponential)
0890: def _kl_uniform_exponetial(p, q):
0891:     result = q.rate * (p.high + p.low) / 2 - ((p.high - p.low) * q.rate).log()
0892:     result[p.low < q.support.lower_bound] = inf
0893:     return result
0894: 
0895: 
````

- **L862** EN: Invokes `q.concentration1.lgamma` to advance the surrounding implementation. | CN: 调用 `q.concentration1.lgamma` 来推进周围的实现逻辑。
- **L863** EN: Invokes `q.concentration0.lgamma` to advance the surrounding implementation. | CN: 调用 `q.concentration0.lgamma` 来推进周围的实现逻辑。
- **L864** EN: Invokes `lgamma` to advance the surrounding implementation. | CN: 调用 `lgamma` 来推进周围的实现逻辑。
- **L865** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L866** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L867** EN: Continues `_kl_uniform_beta`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_beta` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L868** EN: Returns from `_kl_uniform_beta` with the computed result or updated state. | CN: 从 `_kl_uniform_beta` 返回计算结果或更新后的状态。
- **L869** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L870** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L871** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L872** EN: Defines function `_kl_uniform_continuous_bernoulli`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_uniform_continuous_bernoulli`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L873** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L874** EN: Invokes `p.entropy` to advance the surrounding implementation. | CN: 调用 `p.entropy` 来推进周围的实现逻辑。
- **L875** EN: Continues `_kl_uniform_continuous_bernoulli`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_continuous_bernoulli` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L876** EN: Invokes `torch.log1p` to advance the surrounding implementation. | CN: 调用 `torch.log1p` 来推进周围的实现逻辑。
- **L877** EN: Invokes `q._cont_bern_log_norm` to advance the surrounding implementation. | CN: 调用 `q._cont_bern_log_norm` 来推进周围的实现逻辑。
- **L878** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L879** EN: Returns from `_kl_uniform_continuous_bernoulli` with the computed result or updated state. | CN: 从 `_kl_uniform_continuous_bernoulli` 返回计算结果或更新后的状态。
- **L880** EN: Invokes `torch.max` to advance the surrounding implementation. | CN: 调用 `torch.max` 来推进周围的实现逻辑。
- **L881** EN: Invokes `torch.ge` to advance the surrounding implementation. | CN: 调用 `torch.ge` 来推进周围的实现逻辑。
- **L882** EN: Invokes `torch.le` to advance the surrounding implementation. | CN: 调用 `torch.le` 来推进周围的实现逻辑。
- **L883** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L884** EN: Invokes `torch.ones_like` to advance the surrounding implementation. | CN: 调用 `torch.ones_like` 来推进周围的实现逻辑。
- **L885** EN: Continues `_kl_uniform_continuous_bernoulli`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_continuous_bernoulli` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L886** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L887** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L888** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L889** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L890** EN: Defines function `_kl_uniform_exponetial`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_uniform_exponetial`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L891** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L892** EN: Continues `_kl_uniform_exponetial`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_exponetial` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L893** EN: Returns from `_kl_uniform_exponetial` with the computed result or updated state. | CN: 从 `_kl_uniform_exponetial` 返回计算结果或更新后的状态。
- **L894** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L895** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 896-924 / 第 896-924 行

````python
0896: @register_kl(Uniform, Gamma)
0897: def _kl_uniform_gamma(p, q):
0898:     common_term = p.high - p.low
0899:     t1 = common_term.log()
0900:     t2 = q.concentration.lgamma() - q.concentration * q.rate.log()
0901:     t3 = (
0902:         (1 - q.concentration)
0903:         * (_x_log_x(p.high) - _x_log_x(p.low) - common_term)
0904:         / common_term
0905:     )
0906:     t4 = q.rate * (p.high + p.low) / 2
0907:     result = -t1 + t2 + t3 + t4
0908:     result[p.low < q.support.lower_bound] = inf
0909:     return result
0910: 
0911: 
0912: @register_kl(Uniform, Gumbel)
0913: def _kl_uniform_gumbel(p, q):
0914:     common_term = q.scale / (p.high - p.low)
0915:     high_loc_diff = (p.high - q.loc) / q.scale
0916:     low_loc_diff = (p.low - q.loc) / q.scale
0917:     t1 = common_term.log() + 0.5 * (high_loc_diff + low_loc_diff)
0918:     t2 = common_term * (torch.exp(-high_loc_diff) - torch.exp(-low_loc_diff))
0919:     return t1 - t2
0920: 
0921: 
0922: # TODO: Uniform-Laplace KL Divergence
0923: 
0924: 
````

- **L896** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L897** EN: Defines function `_kl_uniform_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_uniform_gamma`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L898** EN: Assigns or updates `common_term`. | CN: 对 `common_term` 进行赋值或更新。
- **L899** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L900** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L901** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L902** EN: Continues `_kl_uniform_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_gamma` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L903** EN: Invokes `_x_log_x` to advance the surrounding implementation. | CN: 调用 `_x_log_x` 来推进周围的实现逻辑。
- **L904** EN: Continues `_kl_uniform_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_gamma` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L905** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L906** EN: Assigns or updates `t4`. | CN: 对 `t4` 进行赋值或更新。
- **L907** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L908** EN: Continues `_kl_uniform_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_gamma` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L909** EN: Returns from `_kl_uniform_gamma` with the computed result or updated state. | CN: 从 `_kl_uniform_gamma` 返回计算结果或更新后的状态。
- **L910** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L911** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L912** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L913** EN: Defines function `_kl_uniform_gumbel`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_uniform_gumbel`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L914** EN: Assigns or updates `common_term`. | CN: 对 `common_term` 进行赋值或更新。
- **L915** EN: Assigns or updates `high_loc_diff`. | CN: 对 `high_loc_diff` 进行赋值或更新。
- **L916** EN: Assigns or updates `low_loc_diff`. | CN: 对 `low_loc_diff` 进行赋值或更新。
- **L917** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L918** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L919** EN: Returns from `_kl_uniform_gumbel` with the computed result or updated state. | CN: 从 `_kl_uniform_gumbel` 返回计算结果或更新后的状态。
- **L920** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L921** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L922** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L923** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L924** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 925-958 / 第 925-958 行

````python
0925: @register_kl(Uniform, Normal)
0926: def _kl_uniform_normal(p, q):
0927:     common_term = p.high - p.low
0928:     t1 = (math.sqrt(math.pi * 2) * q.scale / common_term).log()
0929:     t2 = (common_term).pow(2) / 12
0930:     t3 = ((p.high + p.low - 2 * q.loc) / 2).pow(2)
0931:     return t1 + 0.5 * (t2 + t3) / q.scale.pow(2)
0932: 
0933: 
0934: @register_kl(Uniform, Pareto)
0935: def _kl_uniform_pareto(p, q):
0936:     support_uniform = p.high - p.low
0937:     t1 = (q.alpha * q.scale.pow(q.alpha) * (support_uniform)).log()
0938:     t2 = (_x_log_x(p.high) - _x_log_x(p.low) - support_uniform) / support_uniform
0939:     result = t2 * (q.alpha + 1) - t1
0940:     result[p.low < q.support.lower_bound] = inf
0941:     return result
0942: 
0943: 
0944: @register_kl(Independent, Independent)
0945: def _kl_independent_independent(p, q):
0946:     if p.reinterpreted_batch_ndims != q.reinterpreted_batch_ndims:
0947:         raise NotImplementedError
0948:     result = kl_divergence(p.base_dist, q.base_dist)
0949:     return _sum_rightmost(result, p.reinterpreted_batch_ndims)
0950: 
0951: 
0952: @register_kl(Cauchy, Cauchy)
0953: def _kl_cauchy_cauchy(p, q):
0954:     # From https://arxiv.org/abs/1905.10965
0955:     t1 = ((p.scale + q.scale).pow(2) + (p.loc - q.loc).pow(2)).log()
0956:     t2 = (4 * p.scale * q.scale).log()
0957:     return t1 - t2
0958: 
````

- **L925** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L926** EN: Defines function `_kl_uniform_normal`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_uniform_normal`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L927** EN: Assigns or updates `common_term`. | CN: 对 `common_term` 进行赋值或更新。
- **L928** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L929** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L930** EN: Assigns or updates `t3`. | CN: 对 `t3` 进行赋值或更新。
- **L931** EN: Returns from `_kl_uniform_normal` with the computed result or updated state. | CN: 从 `_kl_uniform_normal` 返回计算结果或更新后的状态。
- **L932** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L933** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L934** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L935** EN: Defines function `_kl_uniform_pareto`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_uniform_pareto`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L936** EN: Assigns or updates `support_uniform`. | CN: 对 `support_uniform` 进行赋值或更新。
- **L937** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L938** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L939** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L940** EN: Continues `_kl_uniform_pareto`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_kl_uniform_pareto` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L941** EN: Returns from `_kl_uniform_pareto` with the computed result or updated state. | CN: 从 `_kl_uniform_pareto` 返回计算结果或更新后的状态。
- **L942** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L943** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L944** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L945** EN: Defines function `_kl_independent_independent`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_independent_independent`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L946** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L947** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L948** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L949** EN: Returns from `_kl_independent_independent` with the computed result or updated state. | CN: 从 `_kl_independent_independent` 返回计算结果或更新后的状态。
- **L950** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L951** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L952** EN: Applies decorator `register_kl`, which modifies the behavior of the following definition. | CN: 应用装饰器 `register_kl`，其作用是修改后续定义的行为。
- **L953** EN: Defines function `_kl_cauchy_cauchy`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_kl_cauchy_cauchy`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L954** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L955** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L956** EN: Assigns or updates `t2`. | CN: 对 `t2` 进行赋值或更新。
- **L957** EN: Returns from `_kl_cauchy_cauchy` with the computed result or updated state. | CN: 从 `_kl_cauchy_cauchy` 返回计算结果或更新后的状态。
- **L958** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 959-974 / 第 959-974 行

````python
0959: 
0960: def _add_kl_info():
0961:     """Appends a list of implemented KL functions to the doc for kl_divergence."""
0962:     rows = [
0963:         "KL divergence is currently implemented for the following distribution pairs:"
0964:     ]
0965:     for p, q in sorted(
0966:         _KL_REGISTRY, key=lambda p_q: (p_q[0].__name__, p_q[1].__name__)
0967:     ):
0968:         rows.append(
0969:             f"* :class:`~torch.distributions.{p.__name__}` and :class:`~torch.distributions.{q.__name__}`"
0970:         )
0971:     kl_info = "\n\t".join(rows)
0972:     if kl_divergence.__doc__:
0973:         # pyrefly: ignore [missing-attribute]
0974:         kl_divergence.__doc__ += kl_info
````

- **L959** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L960** EN: Defines function `_add_kl_info`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_add_kl_info`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L961** EN: Provides a one-line docstring for function `_add_kl_info`. | CN: 为 function `_add_kl_info` 提供单行文档字符串。
- **L962** EN: Assigns or updates `rows`. | CN: 对 `rows` 进行赋值或更新。
- **L963** EN: Continues `_add_kl_info`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_add_kl_info` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L964** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L965** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L966** EN: Continues `_add_kl_info`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_add_kl_info` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L967** EN: Continues `_add_kl_info`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_add_kl_info` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L968** EN: Invokes `rows.append` to advance the surrounding implementation. | CN: 调用 `rows.append` 来推进周围的实现逻辑。
- **L969** EN: Continues `_add_kl_info`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_add_kl_info` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L970** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L971** EN: Assigns or updates `kl_info`. | CN: 对 `kl_info` 进行赋值或更新。
- **L972** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L973** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L974** EN: Continues `_add_kl_info`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `_add_kl_info` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。

## Key Concepts / 关键概念

- **EN**: Probability parameterization — The code manages parameters such as probabilities, logits, concentration values, or base measures.
  **CN**: Probability parameterization——代码管理概率、logits、浓度参数或基测度等分布参数。
- **EN**: Sampling and statistics — The implementation usually defines sampling, moments, support, or likelihood-related routines.
  **CN**: Sampling and statistics——实现通常会定义采样、矩、支持集或似然相关例程。
- **EN**: Shape and support rules — Broadcasting rules and support constraints keep tensor-valued distributions mathematically valid.
  **CN**: Shape and support rules——广播规则与支持集约束用于保证张量分布在数学上有效。
- **EN**: Structured mapping — The code applies a region or callable across structured inputs in a traceable way.
  **CN**: Structured mapping——代码以可追踪方式把区域或可调用对象应用到结构化输入上。
- **EN**: Distribution base classes — The file participates in the common abstractions shared by many probability distributions.
  **CN**: Distribution base classes——该文件参与多个概率分布共享的公共抽象。
- **EN**: Transforms — The file applies mathematical or graph-level transforms to values or programs.
  **CN**: Transforms——该文件对数值或程序施加数学变换或图级变换。

## Dependencies / 依赖关系

- **Torch imports / Torch 导入**: `torch`、`torch:inf, Tensor`
- **Other imports / 其他导入**: `math`、`warnings`、`collections.abc:Callable`、`functools:total_ordering`、`.bernoulli:Bernoulli`、`.beta:Beta`、`.binomial:Binomial`、`.categorical:Categorical`、`.cauchy:Cauchy`、`.continuous_bernoulli:ContinuousBernoulli` 等共 29 项
- **Top-level classes / 顶层类**: `_Match`
- **Top-level functions / 顶层函数**: `register_kl`、`_dispatch_kl`、`_infinite_like`、`_x_log_x`、`_batch_trace_XXT`、`kl_divergence`、`_kl_bernoulli_bernoulli`、`_kl_beta_beta`、`_kl_binomial_binomial`、`_kl_categorical_categorical` 等共 70 项
- **Base classes / 基类**: 无
- **Decorators / 装饰器**: `total_ordering`、`register_kl`
- **Module assignments / 模块级赋值**: `_KL_REGISTRY`、`_KL_MEMOIZE`、`__all__`
