# normal.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/distributions/normal.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements the `Normal` distribution or helper type, including parameter handling, sampling, and statistical methods.
- **Purpose (CN)**: 实现 `Normal` 分布或辅助类型，涵盖参数处理、采样与统计方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21 / 第 1-21 行

````python
0001: # mypy: allow-untyped-defs
0002: import math
0003: 
0004: import torch
0005: from torch import Tensor
0006: from torch.distributions import constraints
0007: from torch.distributions.exp_family import ExponentialFamily
0008: from torch.distributions.utils import _standard_normal, broadcast_all
0009: from torch.types import _Number, _size
0010: 
0011: 
0012: __all__ = ["Normal"]
0013: 
0014: 
0015: class Normal(ExponentialFamily):
0016:     r"""
0017:     Creates a normal (also called Gaussian) distribution parameterized by
0018:     :attr:`loc` and :attr:`scale`.
0019: 
0020:     Example::
0021: 
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Imports module dependencies: `math`. | CN: 导入模块依赖：`math`。
- **L3** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L4** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L5** EN: Imports `Tensor` from `torch` so later code can reuse those definitions. | CN: 从 `torch` 导入 `Tensor`，供后续代码复用这些定义。
- **L6** EN: Imports `constraints` from `torch.distributions` so later code can reuse those definitions. | CN: 从 `torch.distributions` 导入 `constraints`，供后续代码复用这些定义。
- **L7** EN: Imports `ExponentialFamily` from `torch.distributions.exp_family` so later code can reuse those definitions. | CN: 从 `torch.distributions.exp_family` 导入 `ExponentialFamily`，供后续代码复用这些定义。
- **L8** EN: Imports `_standard_normal, broadcast_all` from `torch.distributions.utils` so later code can reuse those definitions. | CN: 从 `torch.distributions.utils` 导入 `_standard_normal, broadcast_all`，供后续代码复用这些定义。
- **L9** EN: Imports `_Number, _size` from `torch.types` so later code can reuse those definitions. | CN: 从 `torch.types` 导入 `_Number, _size`，供后续代码复用这些定义。
- **L10** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L11** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L12** EN: Defines or updates the module’s public export list. | CN: 定义或更新模块的公开导出列表。
- **L13** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L14** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L15** EN: Defines class `Normal` with bases `ExponentialFamily`, which encapsulates distribution parameters, validation rules, and tensor-valued statistical behavior. | CN: 定义类 `Normal`，其基类为 `ExponentialFamily`，作用是封装分布参数、校验规则以及基于张量的统计行为。
- **L16** EN: Starts the docstring for class `Normal`. | CN: 开始为 class `Normal` 编写文档字符串。
- **L17** EN: Continues the docstring for class `Normal`. | CN: 继续补充 class `Normal` 的文档字符串。
- **L18** EN: Continues the docstring for class `Normal`. | CN: 继续补充 class `Normal` 的文档字符串。
- **L19** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L20** EN: Continues the docstring for class `Normal`. | CN: 继续补充 class `Normal` 的文档字符串。
- **L21** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 22-42 / 第 22-42 行

````python
0022:         >>> # xdoctest: +IGNORE_WANT("non-deterministic")
0023:         >>> m = Normal(torch.tensor([0.0]), torch.tensor([1.0]))
0024:         >>> m.sample()  # normally distributed with loc=0 and scale=1
0025:         tensor([ 0.1046])
0026: 
0027:     Args:
0028:         loc (float or Tensor): mean of the distribution (often referred to as mu)
0029:         scale (float or Tensor): standard deviation of the distribution
0030:             (often referred to as sigma)
0031:     """
0032: 
0033:     # pyrefly: ignore [bad-override]
0034:     arg_constraints = {"loc": constraints.real, "scale": constraints.positive}
0035:     support = constraints.real
0036:     has_rsample = True
0037:     _mean_carrier_measure = 0
0038: 
0039:     @property
0040:     def mean(self) -> Tensor:
0041:         return self.loc
0042: 
````

- **L22** EN: Continues the docstring for class `Normal`. | CN: 继续补充 class `Normal` 的文档字符串。
- **L23** EN: Continues the docstring for class `Normal`. | CN: 继续补充 class `Normal` 的文档字符串。
- **L24** EN: Continues the docstring for class `Normal`. | CN: 继续补充 class `Normal` 的文档字符串。
- **L25** EN: Continues the docstring for class `Normal`. | CN: 继续补充 class `Normal` 的文档字符串。
- **L26** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L27** EN: Continues the docstring for class `Normal`. | CN: 继续补充 class `Normal` 的文档字符串。
- **L28** EN: Continues the docstring for class `Normal`. | CN: 继续补充 class `Normal` 的文档字符串。
- **L29** EN: Continues the docstring for class `Normal`. | CN: 继续补充 class `Normal` 的文档字符串。
- **L30** EN: Continues the docstring for class `Normal`. | CN: 继续补充 class `Normal` 的文档字符串。
- **L31** EN: Ends the docstring for class `Normal`. | CN: 结束 class `Normal` 的文档字符串。
- **L32** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L33** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L34** EN: Sets structural metadata on `arg_constraints` that other routines consult later. | CN: 为 `arg_constraints` 设置结构化元数据，供后续例程查询。
- **L35** EN: Sets structural metadata on `support` that other routines consult later. | CN: 为 `support` 设置结构化元数据，供后续例程查询。
- **L36** EN: Assigns or updates `has_rsample`. | CN: 对 `has_rsample` 进行赋值或更新。
- **L37** EN: Assigns module-level configuration or cached state to `_mean_carrier_measure`. | CN: 为 `_mean_carrier_measure` 赋予模块级配置或缓存状态。
- **L38** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L39** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L40** EN: Defines function `mean`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mean`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L41** EN: Returns from `Normal.mean` with the computed result or updated state. | CN: 从 `Normal.mean` 返回计算结果或更新后的状态。
- **L42** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 43-64 / 第 43-64 行

````python
0043:     @property
0044:     def mode(self) -> Tensor:
0045:         return self.loc
0046: 
0047:     @property
0048:     def stddev(self) -> Tensor:
0049:         return self.scale
0050: 
0051:     @property
0052:     def variance(self) -> Tensor:
0053:         return self.stddev.pow(2)
0054: 
0055:     def __init__(
0056:         self,
0057:         loc: Tensor | float,
0058:         scale: Tensor | float,
0059:         validate_args: bool | None = None,
0060:     ) -> None:
0061:         self.loc, self.scale = broadcast_all(loc, scale)
0062:         if isinstance(loc, _Number) and isinstance(scale, _Number):
0063:             batch_shape = torch.Size()
0064:         else:
````

- **L43** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L44** EN: Defines function `mode`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mode`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L45** EN: Returns from `Normal.mode` with the computed result or updated state. | CN: 从 `Normal.mode` 返回计算结果或更新后的状态。
- **L46** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L47** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L48** EN: Defines function `stddev`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `stddev`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L49** EN: Returns from `Normal.stddev` with the computed result or updated state. | CN: 从 `Normal.stddev` 返回计算结果或更新后的状态。
- **L50** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L51** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L52** EN: Defines function `variance`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `variance`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L53** EN: Returns from `Normal.variance` with the computed result or updated state. | CN: 从 `Normal.variance` 返回计算结果或更新后的状态。
- **L54** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L55** EN: Defines function `__init__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__init__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L56** EN: Continues `Normal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Normal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L57** EN: Continues `Normal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Normal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L58** EN: Continues `Normal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Normal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L59** EN: Continues `Normal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Normal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L60** EN: Continues `Normal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Normal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L61** EN: Invokes `broadcast_all` to advance the surrounding implementation. | CN: 调用 `broadcast_all` 来推进周围的实现逻辑。
- **L62** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L63** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L64** EN: Introduces the fallback branch when earlier conditions are not met. | CN: 引入前置条件不满足时的后备分支。

### Lines 65-86 / 第 65-86 行

````python
0065:             batch_shape = self.loc.size()
0066:         super().__init__(batch_shape, validate_args=validate_args)
0067: 
0068:     def expand(self, batch_shape, _instance=None):
0069:         new = self._get_checked_instance(Normal, _instance)
0070:         batch_shape = torch.Size(batch_shape)
0071:         new.loc = self.loc.expand(batch_shape)
0072:         new.scale = self.scale.expand(batch_shape)
0073:         super(Normal, new).__init__(batch_shape, validate_args=False)
0074:         new._validate_args = self._validate_args
0075:         return new
0076: 
0077:     def sample(self, sample_shape=torch.Size()):
0078:         shape = self._extended_shape(sample_shape)
0079:         with torch.no_grad():
0080:             return torch.normal(self.loc.expand(shape), self.scale.expand(shape))
0081: 
0082:     def rsample(self, sample_shape: _size = torch.Size()) -> Tensor:
0083:         shape = self._extended_shape(sample_shape)
0084:         eps = _standard_normal(shape, dtype=self.loc.dtype, device=self.loc.device)
0085:         return self.loc + eps * self.scale
0086: 
````

- **L65** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L66** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L67** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L68** EN: Defines function `expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 定义函数 `expand`，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L69** EN: Assigns or updates `new`. | CN: 对 `new` 进行赋值或更新。
- **L70** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L71** EN: Updates object state via `new.loc`. | CN: 通过 `new.loc` 更新对象状态。
- **L72** EN: Updates object state via `new.scale`. | CN: 通过 `new.scale` 更新对象状态。
- **L73** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L74** EN: Updates object state via `new._validate_args`. | CN: 通过 `new._validate_args` 更新对象状态。
- **L75** EN: Returns from `Normal.expand` with the computed result or updated state. | CN: 从 `Normal.expand` 返回计算结果或更新后的状态。
- **L76** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L77** EN: Defines function `sample`, which produces samples according to the modeled stochastic behavior. | CN: 定义函数 `sample`，其作用是根据建模的随机行为生成样本。
- **L78** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L79** EN: Enters a managed context so resources or flags are scoped safely. | CN: 进入受管理的上下文，使资源或标志拥有安全的作用域。
- **L80** EN: Returns from `Normal.sample` with the computed result or updated state. | CN: 从 `Normal.sample` 返回计算结果或更新后的状态。
- **L81** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L82** EN: Defines function `rsample`, which produces samples according to the modeled stochastic behavior. | CN: 定义函数 `rsample`，其作用是根据建模的随机行为生成样本。
- **L83** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L84** EN: Assigns or updates `eps`. | CN: 对 `eps` 进行赋值或更新。
- **L85** EN: Returns from `Normal.rsample` with the computed result or updated state. | CN: 从 `Normal.rsample` 返回计算结果或更新后的状态。
- **L86** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 87-102 / 第 87-102 行

````python
0087:     def log_prob(self, value):
0088:         if self._validate_args:
0089:             self._validate_sample(value)
0090:         # compute the variance
0091:         var = self.scale**2
0092:         log_scale = (
0093:             math.log(self.scale)
0094:             if isinstance(self.scale, _Number)
0095:             else self.scale.log()
0096:         )
0097:         return (
0098:             -((value - self.loc) ** 2) / (2 * var)
0099:             - log_scale
0100:             - math.log(math.sqrt(2 * math.pi))
0101:         )
0102: 
````

- **L87** EN: Defines function `log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 定义函数 `log_prob`，其作用是为给定输入计算对数概率或类似密度分数。
- **L88** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L89** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L90** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L91** EN: Assigns or updates `var`. | CN: 对 `var` 进行赋值或更新。
- **L92** EN: Assigns or updates `log_scale`. | CN: 对 `log_scale` 进行赋值或更新。
- **L93** EN: Invokes `math.log` to advance the surrounding implementation. | CN: 调用 `math.log` 来推进周围的实现逻辑。
- **L94** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L95** EN: Invokes `self.scale.log` to advance the surrounding implementation. | CN: 调用 `self.scale.log` 来推进周围的实现逻辑。
- **L96** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L97** EN: Returns from `Normal.log_prob` with the computed result or updated state. | CN: 从 `Normal.log_prob` 返回计算结果或更新后的状态。
- **L98** EN: Continues `Normal.log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 继续 `Normal.log_prob` 的实现，其作用是为给定输入计算对数概率或类似密度分数。
- **L99** EN: Continues `Normal.log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 继续 `Normal.log_prob` 的实现，其作用是为给定输入计算对数概率或类似密度分数。
- **L100** EN: Invokes `math.log` to advance the surrounding implementation. | CN: 调用 `math.log` 来推进周围的实现逻辑。
- **L101** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L102** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 103-122 / 第 103-122 行

````python
0103:     def cdf(self, value):
0104:         if self._validate_args:
0105:             self._validate_sample(value)
0106:         return 0.5 * (
0107:             1 + torch.erf((value - self.loc) * self.scale.reciprocal() / math.sqrt(2))
0108:         )
0109: 
0110:     def icdf(self, value):
0111:         return self.loc + self.scale * torch.erfinv(2 * value - 1) * math.sqrt(2)
0112: 
0113:     def entropy(self):
0114:         return 0.5 + 0.5 * math.log(2 * math.pi) + torch.log(self.scale)
0115: 
0116:     @property
0117:     def _natural_params(self) -> tuple[Tensor, Tensor]:
0118:         return (self.loc / self.scale.pow(2), -0.5 * self.scale.pow(2).reciprocal())
0119: 
0120:     # pyrefly: ignore [bad-override]
0121:     def _log_normalizer(self, x, y):
0122:         return -0.25 * x.pow(2) / y + 0.5 * torch.log(-math.pi / y)
````

- **L103** EN: Defines function `cdf`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `cdf`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L104** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L105** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L106** EN: Returns from `Normal.cdf` with the computed result or updated state. | CN: 从 `Normal.cdf` 返回计算结果或更新后的状态。
- **L107** EN: Invokes `torch.erf` to advance the surrounding implementation. | CN: 调用 `torch.erf` 来推进周围的实现逻辑。
- **L108** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L109** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L110** EN: Defines function `icdf`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `icdf`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L111** EN: Returns from `Normal.icdf` with the computed result or updated state. | CN: 从 `Normal.icdf` 返回计算结果或更新后的状态。
- **L112** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L113** EN: Defines function `entropy`, which computes an entropy-style summary of uncertainty. | CN: 定义函数 `entropy`，其作用是计算描述不确定性的熵类指标。
- **L114** EN: Returns from `Normal.entropy` with the computed result or updated state. | CN: 从 `Normal.entropy` 返回计算结果或更新后的状态。
- **L115** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L116** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L117** EN: Defines function `_natural_params`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_natural_params`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L118** EN: Returns from `Normal._natural_params` with the computed result or updated state. | CN: 从 `Normal._natural_params` 返回计算结果或更新后的状态。
- **L119** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L120** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L121** EN: Defines function `_log_normalizer`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_log_normalizer`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L122** EN: Returns from `Normal._log_normalizer` with the computed result or updated state. | CN: 从 `Normal._log_normalizer` 返回计算结果或更新后的状态。

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

- **Torch imports / Torch 导入**: `torch`、`torch:Tensor`、`torch.distributions:constraints`、`torch.distributions.exp_family:ExponentialFamily`、`torch.distributions.utils:_standard_normal, broadcast_all`、`torch.types:_Number, _size`
- **Other imports / 其他导入**: `math`
- **Top-level classes / 顶层类**: `Normal`
- **Top-level functions / 顶层函数**: 无
- **Base classes / 基类**: `ExponentialFamily`
- **Decorators / 装饰器**: 无
- **Module assignments / 模块级赋值**: `__all__`
