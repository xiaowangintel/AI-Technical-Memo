# studentT.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/distributions/studentT.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements the `StudentT` distribution or helper type, including parameter handling, sampling, and statistical methods.
- **Purpose (CN)**: 实现 `StudentT` 分布或辅助类型，涵盖参数处理、采样与统计方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21 / 第 1-21 行

````python
0001: # mypy: allow-untyped-defs
0002: import math
0003: 
0004: import torch
0005: from torch import inf, nan, Tensor
0006: from torch.distributions import Chi2, constraints
0007: from torch.distributions.distribution import Distribution
0008: from torch.distributions.utils import _standard_normal, broadcast_all
0009: from torch.types import _size
0010: 
0011: 
0012: __all__ = ["StudentT"]
0013: 
0014: 
0015: class StudentT(Distribution):
0016:     r"""
0017:     Creates a Student's t-distribution parameterized by degree of
0018:     freedom :attr:`df`, mean :attr:`loc` and scale :attr:`scale`.
0019: 
0020:     Example::
0021: 
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Imports module dependencies: `math`. | CN: 导入模块依赖：`math`。
- **L3** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L4** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L5** EN: Imports `inf, nan, Tensor` from `torch` so later code can reuse those definitions. | CN: 从 `torch` 导入 `inf, nan, Tensor`，供后续代码复用这些定义。
- **L6** EN: Imports `Chi2, constraints` from `torch.distributions` so later code can reuse those definitions. | CN: 从 `torch.distributions` 导入 `Chi2, constraints`，供后续代码复用这些定义。
- **L7** EN: Imports `Distribution` from `torch.distributions.distribution` so later code can reuse those definitions. | CN: 从 `torch.distributions.distribution` 导入 `Distribution`，供后续代码复用这些定义。
- **L8** EN: Imports `_standard_normal, broadcast_all` from `torch.distributions.utils` so later code can reuse those definitions. | CN: 从 `torch.distributions.utils` 导入 `_standard_normal, broadcast_all`，供后续代码复用这些定义。
- **L9** EN: Imports `_size` from `torch.types` so later code can reuse those definitions. | CN: 从 `torch.types` 导入 `_size`，供后续代码复用这些定义。
- **L10** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L11** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L12** EN: Defines or updates the module’s public export list. | CN: 定义或更新模块的公开导出列表。
- **L13** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L14** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L15** EN: Defines class `StudentT` with bases `Distribution`, which encapsulates distribution parameters, validation rules, and tensor-valued statistical behavior. | CN: 定义类 `StudentT`，其基类为 `Distribution`，作用是封装分布参数、校验规则以及基于张量的统计行为。
- **L16** EN: Starts the docstring for class `StudentT`. | CN: 开始为 class `StudentT` 编写文档字符串。
- **L17** EN: Continues the docstring for class `StudentT`. | CN: 继续补充 class `StudentT` 的文档字符串。
- **L18** EN: Continues the docstring for class `StudentT`. | CN: 继续补充 class `StudentT` 的文档字符串。
- **L19** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L20** EN: Continues the docstring for class `StudentT`. | CN: 继续补充 class `StudentT` 的文档字符串。
- **L21** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 22-41 / 第 22-41 行

````python
0022:         >>> # xdoctest: +IGNORE_WANT("non-deterministic")
0023:         >>> m = StudentT(torch.tensor([2.0]))
0024:         >>> m.sample()  # Student's t-distributed with degrees of freedom=2
0025:         tensor([ 0.1046])
0026: 
0027:     Args:
0028:         df (float or Tensor): degrees of freedom
0029:         loc (float or Tensor): mean of the distribution
0030:         scale (float or Tensor): scale of the distribution
0031:     """
0032: 
0033:     # pyrefly: ignore [bad-override]
0034:     arg_constraints = {
0035:         "df": constraints.positive,
0036:         "loc": constraints.real,
0037:         "scale": constraints.positive,
0038:     }
0039:     support = constraints.real
0040:     has_rsample = True
0041: 
````

- **L22** EN: Continues the docstring for class `StudentT`. | CN: 继续补充 class `StudentT` 的文档字符串。
- **L23** EN: Continues the docstring for class `StudentT`. | CN: 继续补充 class `StudentT` 的文档字符串。
- **L24** EN: Continues the docstring for class `StudentT`. | CN: 继续补充 class `StudentT` 的文档字符串。
- **L25** EN: Continues the docstring for class `StudentT`. | CN: 继续补充 class `StudentT` 的文档字符串。
- **L26** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L27** EN: Continues the docstring for class `StudentT`. | CN: 继续补充 class `StudentT` 的文档字符串。
- **L28** EN: Continues the docstring for class `StudentT`. | CN: 继续补充 class `StudentT` 的文档字符串。
- **L29** EN: Continues the docstring for class `StudentT`. | CN: 继续补充 class `StudentT` 的文档字符串。
- **L30** EN: Continues the docstring for class `StudentT`. | CN: 继续补充 class `StudentT` 的文档字符串。
- **L31** EN: Ends the docstring for class `StudentT`. | CN: 结束 class `StudentT` 的文档字符串。
- **L32** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L33** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L34** EN: Sets structural metadata on `arg_constraints` that other routines consult later. | CN: 为 `arg_constraints` 设置结构化元数据，供后续例程查询。
- **L35** EN: Continues class `StudentT`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `StudentT` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L36** EN: Continues class `StudentT`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `StudentT` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L37** EN: Continues class `StudentT`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `StudentT` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L38** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L39** EN: Sets structural metadata on `support` that other routines consult later. | CN: 为 `support` 设置结构化元数据，供后续例程查询。
- **L40** EN: Assigns or updates `has_rsample`. | CN: 对 `has_rsample` 进行赋值或更新。
- **L41** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 42-63 / 第 42-63 行

````python
0042:     @property
0043:     def mean(self) -> Tensor:
0044:         m = self.loc.clone(memory_format=torch.contiguous_format)
0045:         m[self.df <= 1] = nan
0046:         return m
0047: 
0048:     @property
0049:     def mode(self) -> Tensor:
0050:         return self.loc
0051: 
0052:     @property
0053:     def variance(self) -> Tensor:
0054:         m = self.df.clone(memory_format=torch.contiguous_format)
0055:         m[self.df > 2] = (
0056:             self.scale[self.df > 2].pow(2)
0057:             * self.df[self.df > 2]
0058:             / (self.df[self.df > 2] - 2)
0059:         )
0060:         m[(self.df <= 2) & (self.df > 1)] = inf
0061:         m[self.df <= 1] = nan
0062:         return m
0063: 
````

- **L42** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L43** EN: Defines function `mean`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mean`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L44** EN: Assigns or updates `m`. | CN: 对 `m` 进行赋值或更新。
- **L45** EN: Continues `StudentT.mean`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `StudentT.mean` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L46** EN: Returns from `StudentT.mean` with the computed result or updated state. | CN: 从 `StudentT.mean` 返回计算结果或更新后的状态。
- **L47** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L48** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L49** EN: Defines function `mode`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mode`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L50** EN: Returns from `StudentT.mode` with the computed result or updated state. | CN: 从 `StudentT.mode` 返回计算结果或更新后的状态。
- **L51** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L52** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L53** EN: Defines function `variance`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `variance`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L54** EN: Assigns or updates `m`. | CN: 对 `m` 进行赋值或更新。
- **L55** EN: Continues `StudentT.variance`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `StudentT.variance` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L56** EN: Invokes `pow` to advance the surrounding implementation. | CN: 调用 `pow` 来推进周围的实现逻辑。
- **L57** EN: Continues `StudentT.variance`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `StudentT.variance` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L58** EN: Continues `StudentT.variance`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `StudentT.variance` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L59** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L60** EN: Continues `StudentT.variance`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `StudentT.variance` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L61** EN: Continues `StudentT.variance`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `StudentT.variance` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L62** EN: Returns from `StudentT.variance` with the computed result or updated state. | CN: 从 `StudentT.variance` 返回计算结果或更新后的状态。
- **L63** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 64-85 / 第 64-85 行

````python
0064:     def __init__(
0065:         self,
0066:         df: Tensor | float,
0067:         loc: Tensor | float = 0.0,
0068:         scale: Tensor | float = 1.0,
0069:         validate_args: bool | None = None,
0070:     ) -> None:
0071:         self.df, self.loc, self.scale = broadcast_all(df, loc, scale)
0072:         self._chi2 = Chi2(self.df)
0073:         batch_shape = self.df.size()
0074:         super().__init__(batch_shape, validate_args=validate_args)
0075: 
0076:     def expand(self, batch_shape, _instance=None):
0077:         new = self._get_checked_instance(StudentT, _instance)
0078:         batch_shape = torch.Size(batch_shape)
0079:         new.df = self.df.expand(batch_shape)
0080:         new.loc = self.loc.expand(batch_shape)
0081:         new.scale = self.scale.expand(batch_shape)
0082:         new._chi2 = self._chi2.expand(batch_shape)
0083:         super(StudentT, new).__init__(batch_shape, validate_args=False)
0084:         new._validate_args = self._validate_args
0085:         return new
````

- **L64** EN: Defines function `__init__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__init__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L65** EN: Continues `StudentT.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `StudentT.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L66** EN: Continues `StudentT.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `StudentT.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L67** EN: Continues `StudentT.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `StudentT.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L68** EN: Continues `StudentT.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `StudentT.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L69** EN: Continues `StudentT.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `StudentT.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L70** EN: Continues `StudentT.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `StudentT.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L71** EN: Invokes `broadcast_all` to advance the surrounding implementation. | CN: 调用 `broadcast_all` 来推进周围的实现逻辑。
- **L72** EN: Updates object state via `self._chi2`. | CN: 通过 `self._chi2` 更新对象状态。
- **L73** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L74** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L75** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L76** EN: Defines function `expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 定义函数 `expand`，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L77** EN: Assigns or updates `new`. | CN: 对 `new` 进行赋值或更新。
- **L78** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L79** EN: Updates object state via `new.df`. | CN: 通过 `new.df` 更新对象状态。
- **L80** EN: Updates object state via `new.loc`. | CN: 通过 `new.loc` 更新对象状态。
- **L81** EN: Updates object state via `new.scale`. | CN: 通过 `new.scale` 更新对象状态。
- **L82** EN: Updates object state via `new._chi2`. | CN: 通过 `new._chi2` 更新对象状态。
- **L83** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L84** EN: Updates object state via `new._validate_args`. | CN: 通过 `new._validate_args` 更新对象状态。
- **L85** EN: Returns from `StudentT.expand` with the computed result or updated state. | CN: 从 `StudentT.expand` 返回计算结果或更新后的状态。

### Lines 86-100 / 第 86-100 行

````python
0086: 
0087:     def rsample(self, sample_shape: _size = torch.Size()) -> Tensor:
0088:         # NOTE: This does not agree with scipy implementation as much as other distributions.
0089:         # (see https://github.com/fritzo/notebooks/blob/master/debug-student-t.ipynb). Using DoubleTensor
0090:         # parameters seems to help.
0091: 
0092:         #   X ~ Normal(0, 1)
0093:         #   Z ~ Chi2(df)
0094:         #   Y = X / sqrt(Z / df) ~ StudentT(df)
0095:         shape = self._extended_shape(sample_shape)
0096:         X = _standard_normal(shape, dtype=self.df.dtype, device=self.df.device)
0097:         Z = self._chi2.rsample(sample_shape)
0098:         Y = X * torch.rsqrt(Z / self.df)
0099:         return self.loc + self.scale * Y
0100: 
````

- **L86** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L87** EN: Defines function `rsample`, which produces samples according to the modeled stochastic behavior. | CN: 定义函数 `rsample`，其作用是根据建模的随机行为生成样本。
- **L88** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L89** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L90** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L91** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L92** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L93** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L94** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L95** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L96** EN: Assigns module-level configuration or cached state to `X`. | CN: 为 `X` 赋予模块级配置或缓存状态。
- **L97** EN: Assigns module-level configuration or cached state to `Z`. | CN: 为 `Z` 赋予模块级配置或缓存状态。
- **L98** EN: Assigns module-level configuration or cached state to `Y`. | CN: 为 `Y` 赋予模块级配置或缓存状态。
- **L99** EN: Returns from `StudentT.rsample` with the computed result or updated state. | CN: 从 `StudentT.rsample` 返回计算结果或更新后的状态。
- **L100** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 101-122 / 第 101-122 行

````python
0101:     def log_prob(self, value):
0102:         if self._validate_args:
0103:             self._validate_sample(value)
0104:         y = (value - self.loc) / self.scale
0105:         Z = (
0106:             self.scale.log()
0107:             + 0.5 * self.df.log()
0108:             + 0.5 * math.log(math.pi)
0109:             + torch.lgamma(0.5 * self.df)
0110:             - torch.lgamma(0.5 * (self.df + 1.0))
0111:         )
0112:         return -0.5 * (self.df + 1.0) * torch.log1p(y**2.0 / self.df) - Z
0113: 
0114:     def entropy(self):
0115:         lbeta = (
0116:             torch.lgamma(0.5 * self.df)
0117:             + math.lgamma(0.5)
0118:             - torch.lgamma(0.5 * (self.df + 1))
0119:         )
0120:         return (
0121:             self.scale.log()
0122:             + 0.5
````

- **L101** EN: Defines function `log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 定义函数 `log_prob`，其作用是为给定输入计算对数概率或类似密度分数。
- **L102** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L103** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L104** EN: Assigns or updates `y`. | CN: 对 `y` 进行赋值或更新。
- **L105** EN: Assigns module-level configuration or cached state to `Z`. | CN: 为 `Z` 赋予模块级配置或缓存状态。
- **L106** EN: Invokes `self.scale.log` to advance the surrounding implementation. | CN: 调用 `self.scale.log` 来推进周围的实现逻辑。
- **L107** EN: Invokes `self.df.log` to advance the surrounding implementation. | CN: 调用 `self.df.log` 来推进周围的实现逻辑。
- **L108** EN: Invokes `math.log` to advance the surrounding implementation. | CN: 调用 `math.log` 来推进周围的实现逻辑。
- **L109** EN: Invokes `torch.lgamma` to advance the surrounding implementation. | CN: 调用 `torch.lgamma` 来推进周围的实现逻辑。
- **L110** EN: Invokes `torch.lgamma` to advance the surrounding implementation. | CN: 调用 `torch.lgamma` 来推进周围的实现逻辑。
- **L111** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L112** EN: Returns from `StudentT.log_prob` with the computed result or updated state. | CN: 从 `StudentT.log_prob` 返回计算结果或更新后的状态。
- **L113** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L114** EN: Defines function `entropy`, which computes an entropy-style summary of uncertainty. | CN: 定义函数 `entropy`，其作用是计算描述不确定性的熵类指标。
- **L115** EN: Assigns or updates `lbeta`. | CN: 对 `lbeta` 进行赋值或更新。
- **L116** EN: Invokes `torch.lgamma` to advance the surrounding implementation. | CN: 调用 `torch.lgamma` 来推进周围的实现逻辑。
- **L117** EN: Invokes `math.lgamma` to advance the surrounding implementation. | CN: 调用 `math.lgamma` 来推进周围的实现逻辑。
- **L118** EN: Invokes `torch.lgamma` to advance the surrounding implementation. | CN: 调用 `torch.lgamma` 来推进周围的实现逻辑。
- **L119** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L120** EN: Returns from `StudentT.entropy` with the computed result or updated state. | CN: 从 `StudentT.entropy` 返回计算结果或更新后的状态。
- **L121** EN: Invokes `self.scale.log` to advance the surrounding implementation. | CN: 调用 `self.scale.log` 来推进周围的实现逻辑。
- **L122** EN: Continues `StudentT.entropy`, which computes an entropy-style summary of uncertainty. | CN: 继续 `StudentT.entropy` 的实现，其作用是计算描述不确定性的熵类指标。

### Lines 123-127 / 第 123-127 行

````python
0123:             * (self.df + 1)
0124:             * (torch.digamma(0.5 * (self.df + 1)) - torch.digamma(0.5 * self.df))
0125:             + 0.5 * self.df.log()
0126:             + lbeta
0127:         )
````

- **L123** EN: Continues `StudentT.entropy`, which computes an entropy-style summary of uncertainty. | CN: 继续 `StudentT.entropy` 的实现，其作用是计算描述不确定性的熵类指标。
- **L124** EN: Invokes `torch.digamma` to advance the surrounding implementation. | CN: 调用 `torch.digamma` 来推进周围的实现逻辑。
- **L125** EN: Invokes `self.df.log` to advance the surrounding implementation. | CN: 调用 `self.df.log` 来推进周围的实现逻辑。
- **L126** EN: Continues `StudentT.entropy`, which computes an entropy-style summary of uncertainty. | CN: 继续 `StudentT.entropy` 的实现，其作用是计算描述不确定性的熵类指标。
- **L127** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。

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
- **EN**: Likelihood evaluation — Log-probability or density evaluation is a key behavior here.
  **CN**: Likelihood evaluation——对数概率或密度计算是这里的关键行为。

## Dependencies / 依赖关系

- **Torch imports / Torch 导入**: `torch`、`torch:inf, nan, Tensor`、`torch.distributions:Chi2, constraints`、`torch.distributions.distribution:Distribution`、`torch.distributions.utils:_standard_normal, broadcast_all`、`torch.types:_size`
- **Other imports / 其他导入**: `math`
- **Top-level classes / 顶层类**: `StudentT`
- **Top-level functions / 顶层函数**: 无
- **Base classes / 基类**: `Distribution`
- **Decorators / 装饰器**: 无
- **Module assignments / 模块级赋值**: `__all__`
