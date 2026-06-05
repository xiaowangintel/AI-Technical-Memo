# gamma.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/distributions/gamma.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements the `Gamma` distribution or helper type, including parameter handling, sampling, and statistical methods.
- **Purpose (CN)**: 实现 `Gamma` 分布或辅助类型，涵盖参数处理、采样与统计方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21 / 第 1-21 行

````python
0001: # mypy: allow-untyped-defs
0002: 
0003: import torch
0004: from torch import Tensor
0005: from torch.distributions import constraints
0006: from torch.distributions.exp_family import ExponentialFamily
0007: from torch.distributions.utils import broadcast_all
0008: from torch.types import _Number, _size
0009: 
0010: 
0011: __all__ = ["Gamma"]
0012: 
0013: 
0014: def _standard_gamma(concentration):
0015:     return torch._standard_gamma(concentration)
0016: 
0017: 
0018: class Gamma(ExponentialFamily):
0019:     r"""
0020:     Creates a Gamma distribution parameterized by shape :attr:`concentration` and :attr:`rate`.
0021: 
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L3** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L4** EN: Imports `Tensor` from `torch` so later code can reuse those definitions. | CN: 从 `torch` 导入 `Tensor`，供后续代码复用这些定义。
- **L5** EN: Imports `constraints` from `torch.distributions` so later code can reuse those definitions. | CN: 从 `torch.distributions` 导入 `constraints`，供后续代码复用这些定义。
- **L6** EN: Imports `ExponentialFamily` from `torch.distributions.exp_family` so later code can reuse those definitions. | CN: 从 `torch.distributions.exp_family` 导入 `ExponentialFamily`，供后续代码复用这些定义。
- **L7** EN: Imports `broadcast_all` from `torch.distributions.utils` so later code can reuse those definitions. | CN: 从 `torch.distributions.utils` 导入 `broadcast_all`，供后续代码复用这些定义。
- **L8** EN: Imports `_Number, _size` from `torch.types` so later code can reuse those definitions. | CN: 从 `torch.types` 导入 `_Number, _size`，供后续代码复用这些定义。
- **L9** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L10** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L11** EN: Defines or updates the module’s public export list. | CN: 定义或更新模块的公开导出列表。
- **L12** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L13** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L14** EN: Defines function `_standard_gamma`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_standard_gamma`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L15** EN: Returns from `_standard_gamma` with the computed result or updated state. | CN: 从 `_standard_gamma` 返回计算结果或更新后的状态。
- **L16** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L17** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L18** EN: Defines class `Gamma` with bases `ExponentialFamily`, which encapsulates distribution parameters, validation rules, and tensor-valued statistical behavior. | CN: 定义类 `Gamma`，其基类为 `ExponentialFamily`，作用是封装分布参数、校验规则以及基于张量的统计行为。
- **L19** EN: Starts the docstring for class `Gamma`. | CN: 开始为 class `Gamma` 编写文档字符串。
- **L20** EN: Continues the docstring for class `Gamma`. | CN: 继续补充 class `Gamma` 的文档字符串。
- **L21** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 22-43 / 第 22-43 行

````python
0022:     Example::
0023: 
0024:         >>> # xdoctest: +IGNORE_WANT("non-deterministic")
0025:         >>> m = Gamma(torch.tensor([1.0]), torch.tensor([1.0]))
0026:         >>> m.sample()  # Gamma distributed with concentration=1 and rate=1
0027:         tensor([ 0.1046])
0028: 
0029:     Args:
0030:         concentration (float or Tensor): shape parameter of the distribution
0031:             (often referred to as alpha)
0032:         rate (float or Tensor): rate parameter of the distribution
0033:             (often referred to as beta), rate = 1 / scale
0034:     """
0035: 
0036:     # pyrefly: ignore [bad-override]
0037:     arg_constraints = {
0038:         "concentration": constraints.positive,
0039:         "rate": constraints.positive,
0040:     }
0041:     support = constraints.nonnegative
0042:     has_rsample = True
0043:     _mean_carrier_measure = 0
````

- **L22** EN: Continues the docstring for class `Gamma`. | CN: 继续补充 class `Gamma` 的文档字符串。
- **L23** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L24** EN: Continues the docstring for class `Gamma`. | CN: 继续补充 class `Gamma` 的文档字符串。
- **L25** EN: Continues the docstring for class `Gamma`. | CN: 继续补充 class `Gamma` 的文档字符串。
- **L26** EN: Continues the docstring for class `Gamma`. | CN: 继续补充 class `Gamma` 的文档字符串。
- **L27** EN: Continues the docstring for class `Gamma`. | CN: 继续补充 class `Gamma` 的文档字符串。
- **L28** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L29** EN: Continues the docstring for class `Gamma`. | CN: 继续补充 class `Gamma` 的文档字符串。
- **L30** EN: Continues the docstring for class `Gamma`. | CN: 继续补充 class `Gamma` 的文档字符串。
- **L31** EN: Continues the docstring for class `Gamma`. | CN: 继续补充 class `Gamma` 的文档字符串。
- **L32** EN: Continues the docstring for class `Gamma`. | CN: 继续补充 class `Gamma` 的文档字符串。
- **L33** EN: Continues the docstring for class `Gamma`. | CN: 继续补充 class `Gamma` 的文档字符串。
- **L34** EN: Ends the docstring for class `Gamma`. | CN: 结束 class `Gamma` 的文档字符串。
- **L35** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L36** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L37** EN: Sets structural metadata on `arg_constraints` that other routines consult later. | CN: 为 `arg_constraints` 设置结构化元数据，供后续例程查询。
- **L38** EN: Continues class `Gamma`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `Gamma` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L39** EN: Continues class `Gamma`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `Gamma` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L40** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L41** EN: Sets structural metadata on `support` that other routines consult later. | CN: 为 `support` 设置结构化元数据，供后续例程查询。
- **L42** EN: Assigns or updates `has_rsample`. | CN: 对 `has_rsample` 进行赋值或更新。
- **L43** EN: Assigns module-level configuration or cached state to `_mean_carrier_measure`. | CN: 为 `_mean_carrier_measure` 赋予模块级配置或缓存状态。

### Lines 44-65 / 第 44-65 行

````python
0044: 
0045:     @property
0046:     def mean(self) -> Tensor:
0047:         return self.concentration / self.rate
0048: 
0049:     @property
0050:     def mode(self) -> Tensor:
0051:         return ((self.concentration - 1) / self.rate).clamp(min=0)
0052: 
0053:     @property
0054:     def variance(self) -> Tensor:
0055:         return self.concentration / self.rate.pow(2)
0056: 
0057:     def __init__(
0058:         self,
0059:         concentration: Tensor | float,
0060:         rate: Tensor | float,
0061:         validate_args: bool | None = None,
0062:     ) -> None:
0063:         self.concentration, self.rate = broadcast_all(concentration, rate)
0064:         if isinstance(concentration, _Number) and isinstance(rate, _Number):
0065:             batch_shape = torch.Size()
````

- **L44** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L45** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L46** EN: Defines function `mean`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mean`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L47** EN: Returns from `Gamma.mean` with the computed result or updated state. | CN: 从 `Gamma.mean` 返回计算结果或更新后的状态。
- **L48** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L49** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L50** EN: Defines function `mode`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mode`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L51** EN: Returns from `Gamma.mode` with the computed result or updated state. | CN: 从 `Gamma.mode` 返回计算结果或更新后的状态。
- **L52** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L53** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L54** EN: Defines function `variance`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `variance`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L55** EN: Returns from `Gamma.variance` with the computed result or updated state. | CN: 从 `Gamma.variance` 返回计算结果或更新后的状态。
- **L56** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L57** EN: Defines function `__init__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__init__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L58** EN: Continues `Gamma.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Gamma.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L59** EN: Continues `Gamma.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Gamma.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L60** EN: Continues `Gamma.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Gamma.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L61** EN: Continues `Gamma.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Gamma.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L62** EN: Continues `Gamma.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Gamma.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L63** EN: Invokes `broadcast_all` to advance the surrounding implementation. | CN: 调用 `broadcast_all` 来推进周围的实现逻辑。
- **L64** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L65** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。

### Lines 66-87 / 第 66-87 行

````python
0066:         else:
0067:             batch_shape = self.concentration.size()
0068:         super().__init__(batch_shape, validate_args=validate_args)
0069: 
0070:     def expand(self, batch_shape, _instance=None):
0071:         new = self._get_checked_instance(Gamma, _instance)
0072:         batch_shape = torch.Size(batch_shape)
0073:         new.concentration = self.concentration.expand(batch_shape)
0074:         new.rate = self.rate.expand(batch_shape)
0075:         super(Gamma, new).__init__(batch_shape, validate_args=False)
0076:         new._validate_args = self._validate_args
0077:         return new
0078: 
0079:     def rsample(self, sample_shape: _size = torch.Size()) -> Tensor:
0080:         shape = self._extended_shape(sample_shape)
0081:         value = _standard_gamma(self.concentration.expand(shape)) / self.rate.expand(
0082:             shape
0083:         )
0084:         value.detach().clamp_(
0085:             min=torch.finfo(value.dtype).tiny
0086:         )  # do not record in autograd graph
0087:         return value
````

- **L66** EN: Introduces the fallback branch when earlier conditions are not met. | CN: 引入前置条件不满足时的后备分支。
- **L67** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L68** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L69** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L70** EN: Defines function `expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 定义函数 `expand`，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L71** EN: Assigns or updates `new`. | CN: 对 `new` 进行赋值或更新。
- **L72** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L73** EN: Updates object state via `new.concentration`. | CN: 通过 `new.concentration` 更新对象状态。
- **L74** EN: Updates object state via `new.rate`. | CN: 通过 `new.rate` 更新对象状态。
- **L75** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L76** EN: Updates object state via `new._validate_args`. | CN: 通过 `new._validate_args` 更新对象状态。
- **L77** EN: Returns from `Gamma.expand` with the computed result or updated state. | CN: 从 `Gamma.expand` 返回计算结果或更新后的状态。
- **L78** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L79** EN: Defines function `rsample`, which produces samples according to the modeled stochastic behavior. | CN: 定义函数 `rsample`，其作用是根据建模的随机行为生成样本。
- **L80** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L81** EN: Assigns or updates `value`. | CN: 对 `value` 进行赋值或更新。
- **L82** EN: Continues `Gamma.rsample`, which produces samples according to the modeled stochastic behavior. | CN: 继续 `Gamma.rsample` 的实现，其作用是根据建模的随机行为生成样本。
- **L83** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L84** EN: Invokes `value.detach` to advance the surrounding implementation. | CN: 调用 `value.detach` 来推进周围的实现逻辑。
- **L85** EN: Assigns or updates `min`. | CN: 对 `min` 进行赋值或更新。
- **L86** EN: Continues `Gamma.rsample`, which produces samples according to the modeled stochastic behavior. | CN: 继续 `Gamma.rsample` 的实现，其作用是根据建模的随机行为生成样本。
- **L87** EN: Returns from `Gamma.rsample` with the computed result or updated state. | CN: 从 `Gamma.rsample` 返回计算结果或更新后的状态。

### Lines 88-107 / 第 88-107 行

````python
0088: 
0089:     def log_prob(self, value):
0090:         value = torch.as_tensor(value, dtype=self.rate.dtype, device=self.rate.device)
0091:         if self._validate_args:
0092:             self._validate_sample(value)
0093:         return (
0094:             torch.xlogy(self.concentration, self.rate)
0095:             + torch.xlogy(self.concentration - 1, value)
0096:             - self.rate * value
0097:             - torch.lgamma(self.concentration)
0098:         )
0099: 
0100:     def entropy(self):
0101:         return (
0102:             self.concentration
0103:             - torch.log(self.rate)
0104:             + torch.lgamma(self.concentration)
0105:             + (1.0 - self.concentration) * torch.digamma(self.concentration)
0106:         )
0107: 
````

- **L88** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L89** EN: Defines function `log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 定义函数 `log_prob`，其作用是为给定输入计算对数概率或类似密度分数。
- **L90** EN: Assigns or updates `value`. | CN: 对 `value` 进行赋值或更新。
- **L91** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L92** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L93** EN: Returns from `Gamma.log_prob` with the computed result or updated state. | CN: 从 `Gamma.log_prob` 返回计算结果或更新后的状态。
- **L94** EN: Invokes `torch.xlogy` to advance the surrounding implementation. | CN: 调用 `torch.xlogy` 来推进周围的实现逻辑。
- **L95** EN: Invokes `torch.xlogy` to advance the surrounding implementation. | CN: 调用 `torch.xlogy` 来推进周围的实现逻辑。
- **L96** EN: Continues `Gamma.log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 继续 `Gamma.log_prob` 的实现，其作用是为给定输入计算对数概率或类似密度分数。
- **L97** EN: Invokes `torch.lgamma` to advance the surrounding implementation. | CN: 调用 `torch.lgamma` 来推进周围的实现逻辑。
- **L98** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L99** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L100** EN: Defines function `entropy`, which computes an entropy-style summary of uncertainty. | CN: 定义函数 `entropy`，其作用是计算描述不确定性的熵类指标。
- **L101** EN: Returns from `Gamma.entropy` with the computed result or updated state. | CN: 从 `Gamma.entropy` 返回计算结果或更新后的状态。
- **L102** EN: Continues `Gamma.entropy`, which computes an entropy-style summary of uncertainty. | CN: 继续 `Gamma.entropy` 的实现，其作用是计算描述不确定性的熵类指标。
- **L103** EN: Invokes `torch.log` to advance the surrounding implementation. | CN: 调用 `torch.log` 来推进周围的实现逻辑。
- **L104** EN: Invokes `torch.lgamma` to advance the surrounding implementation. | CN: 调用 `torch.lgamma` 来推进周围的实现逻辑。
- **L105** EN: Invokes `torch.digamma` to advance the surrounding implementation. | CN: 调用 `torch.digamma` 来推进周围的实现逻辑。
- **L106** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L107** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 108-119 / 第 108-119 行

````python
0108:     @property
0109:     def _natural_params(self) -> tuple[Tensor, Tensor]:
0110:         return (self.concentration - 1, -self.rate)
0111: 
0112:     # pyrefly: ignore [bad-override]
0113:     def _log_normalizer(self, x, y):
0114:         return torch.lgamma(x + 1) + (x + 1) * torch.log(-y.reciprocal())
0115: 
0116:     def cdf(self, value):
0117:         if self._validate_args:
0118:             self._validate_sample(value)
0119:         return torch.special.gammainc(self.concentration, self.rate * value)
````

- **L108** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L109** EN: Defines function `_natural_params`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_natural_params`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L110** EN: Returns from `Gamma._natural_params` with the computed result or updated state. | CN: 从 `Gamma._natural_params` 返回计算结果或更新后的状态。
- **L111** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L112** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L113** EN: Defines function `_log_normalizer`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_log_normalizer`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L114** EN: Returns from `Gamma._log_normalizer` with the computed result or updated state. | CN: 从 `Gamma._log_normalizer` 返回计算结果或更新后的状态。
- **L115** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L116** EN: Defines function `cdf`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `cdf`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L117** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L118** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L119** EN: Returns from `Gamma.cdf` with the computed result or updated state. | CN: 从 `Gamma.cdf` 返回计算结果或更新后的状态。

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

- **Torch imports / Torch 导入**: `torch`、`torch:Tensor`、`torch.distributions:constraints`、`torch.distributions.exp_family:ExponentialFamily`、`torch.distributions.utils:broadcast_all`、`torch.types:_Number, _size`
- **Other imports / 其他导入**: 无
- **Top-level classes / 顶层类**: `Gamma`
- **Top-level functions / 顶层函数**: `_standard_gamma`
- **Base classes / 基类**: `ExponentialFamily`
- **Decorators / 装饰器**: 无
- **Module assignments / 模块级赋值**: `__all__`
