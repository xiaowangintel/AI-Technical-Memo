# relaxed_categorical.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/distributions/relaxed_categorical.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements the `ExpRelaxedCategorical` distribution or helper type, including parameter handling, sampling, and statistical methods.
- **Purpose (CN)**: 实现 `ExpRelaxedCategorical` 分布或辅助类型，涵盖参数处理、采样与统计方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

````python
0001: # mypy: allow-untyped-defs
0002: 
0003: import torch
0004: from torch import Tensor
0005: from torch.distributions import constraints
0006: from torch.distributions.categorical import Categorical
0007: from torch.distributions.distribution import Distribution
0008: from torch.distributions.transformed_distribution import TransformedDistribution
0009: from torch.distributions.transforms import ExpTransform
0010: from torch.distributions.utils import broadcast_all, clamp_probs
0011: from torch.types import _size
0012: 
0013: 
0014: __all__ = ["ExpRelaxedCategorical", "RelaxedOneHotCategorical"]
0015: 
0016: 
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L3** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L4** EN: Imports `Tensor` from `torch` so later code can reuse those definitions. | CN: 从 `torch` 导入 `Tensor`，供后续代码复用这些定义。
- **L5** EN: Imports `constraints` from `torch.distributions` so later code can reuse those definitions. | CN: 从 `torch.distributions` 导入 `constraints`，供后续代码复用这些定义。
- **L6** EN: Imports `Categorical` from `torch.distributions.categorical` so later code can reuse those definitions. | CN: 从 `torch.distributions.categorical` 导入 `Categorical`，供后续代码复用这些定义。
- **L7** EN: Imports `Distribution` from `torch.distributions.distribution` so later code can reuse those definitions. | CN: 从 `torch.distributions.distribution` 导入 `Distribution`，供后续代码复用这些定义。
- **L8** EN: Imports `TransformedDistribution` from `torch.distributions.transformed_distribution` so later code can reuse those definitions. | CN: 从 `torch.distributions.transformed_distribution` 导入 `TransformedDistribution`，供后续代码复用这些定义。
- **L9** EN: Imports `ExpTransform` from `torch.distributions.transforms` so later code can reuse those definitions. | CN: 从 `torch.distributions.transforms` 导入 `ExpTransform`，供后续代码复用这些定义。
- **L10** EN: Imports `broadcast_all, clamp_probs` from `torch.distributions.utils` so later code can reuse those definitions. | CN: 从 `torch.distributions.utils` 导入 `broadcast_all, clamp_probs`，供后续代码复用这些定义。
- **L11** EN: Imports `_size` from `torch.types` so later code can reuse those definitions. | CN: 从 `torch.types` 导入 `_size`，供后续代码复用这些定义。
- **L12** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L13** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L14** EN: Defines or updates the module’s public export list. | CN: 定义或更新模块的公开导出列表。
- **L15** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L16** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 17-35 / 第 17-35 行

````python
0017: class ExpRelaxedCategorical(Distribution):
0018:     r"""
0019:     Creates a ExpRelaxedCategorical parameterized by
0020:     :attr:`temperature`, and either :attr:`probs` or :attr:`logits` (but not both).
0021:     Returns the log of a point in the simplex. Based on the interface to
0022:     :class:`OneHotCategorical`.
0023: 
0024:     Implementation based on [1].
0025: 
0026:     See also: :func:`torch.distributions.OneHotCategorical`
0027: 
0028:     Args:
0029:         temperature (Tensor): relaxation temperature
0030:         probs (Tensor): event probabilities
0031:         logits (Tensor): unnormalized log probability for each event
0032: 
0033:     [1] The Concrete Distribution: A Continuous Relaxation of Discrete Random Variables
0034:     (Maddison et al., 2017)
0035: 
````

- **L17** EN: Defines class `ExpRelaxedCategorical` with bases `Distribution`, which encapsulates distribution parameters, validation rules, and tensor-valued statistical behavior. | CN: 定义类 `ExpRelaxedCategorical`，其基类为 `Distribution`，作用是封装分布参数、校验规则以及基于张量的统计行为。
- **L18** EN: Starts the docstring for class `ExpRelaxedCategorical`. | CN: 开始为 class `ExpRelaxedCategorical` 编写文档字符串。
- **L19** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L20** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L21** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L22** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L23** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L24** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L25** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L26** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L27** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L28** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L29** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L30** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L31** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L32** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L33** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L34** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L35** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 36-57 / 第 36-57 行

````python
0036:     [2] Categorical Reparametrization with Gumbel-Softmax
0037:     (Jang et al., 2017)
0038:     """
0039: 
0040:     # pyrefly: ignore [bad-override]
0041:     arg_constraints = {"probs": constraints.simplex, "logits": constraints.real_vector}
0042:     support = (
0043:         constraints.real_vector
0044:     )  # The true support is actually a submanifold of this.
0045:     has_rsample = True
0046: 
0047:     def __init__(
0048:         self,
0049:         temperature: Tensor,
0050:         probs: Tensor | None = None,
0051:         logits: Tensor | None = None,
0052:         validate_args: bool | None = None,
0053:     ) -> None:
0054:         self._categorical = Categorical(probs, logits)
0055:         self.temperature = temperature
0056:         batch_shape = self._categorical.batch_shape
0057:         event_shape = self._categorical.param_shape[-1:]
````

- **L36** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L37** EN: Continues the docstring for class `ExpRelaxedCategorical`. | CN: 继续补充 class `ExpRelaxedCategorical` 的文档字符串。
- **L38** EN: Ends the docstring for class `ExpRelaxedCategorical`. | CN: 结束 class `ExpRelaxedCategorical` 的文档字符串。
- **L39** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L40** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L41** EN: Sets structural metadata on `arg_constraints` that other routines consult later. | CN: 为 `arg_constraints` 设置结构化元数据，供后续例程查询。
- **L42** EN: Sets structural metadata on `support` that other routines consult later. | CN: 为 `support` 设置结构化元数据，供后续例程查询。
- **L43** EN: Continues class `ExpRelaxedCategorical`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `ExpRelaxedCategorical` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L44** EN: Continues class `ExpRelaxedCategorical`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `ExpRelaxedCategorical` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L45** EN: Assigns or updates `has_rsample`. | CN: 对 `has_rsample` 进行赋值或更新。
- **L46** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L47** EN: Defines function `__init__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__init__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L48** EN: Continues `ExpRelaxedCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ExpRelaxedCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L49** EN: Continues `ExpRelaxedCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ExpRelaxedCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L50** EN: Continues `ExpRelaxedCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ExpRelaxedCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L51** EN: Continues `ExpRelaxedCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ExpRelaxedCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L52** EN: Continues `ExpRelaxedCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ExpRelaxedCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L53** EN: Continues `ExpRelaxedCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ExpRelaxedCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L54** EN: Updates object state via `self._categorical`. | CN: 通过 `self._categorical` 更新对象状态。
- **L55** EN: Updates object state via `self.temperature`. | CN: 通过 `self.temperature` 更新对象状态。
- **L56** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L57** EN: Assigns or updates `event_shape`. | CN: 对 `event_shape` 进行赋值或更新。

### Lines 58-78 / 第 58-78 行

````python
0058:         # pyrefly: ignore [bad-argument-type]
0059:         super().__init__(batch_shape, event_shape, validate_args=validate_args)
0060: 
0061:     def expand(self, batch_shape, _instance=None):
0062:         new = self._get_checked_instance(ExpRelaxedCategorical, _instance)
0063:         batch_shape = torch.Size(batch_shape)
0064:         new.temperature = self.temperature
0065:         new._categorical = self._categorical.expand(batch_shape)
0066:         super(ExpRelaxedCategorical, new).__init__(
0067:             batch_shape, self.event_shape, validate_args=False
0068:         )
0069:         new._validate_args = self._validate_args
0070:         return new
0071: 
0072:     def _new(self, *args, **kwargs):
0073:         return self._categorical._new(*args, **kwargs)
0074: 
0075:     @property
0076:     def param_shape(self) -> torch.Size:
0077:         return self._categorical.param_shape
0078: 
````

- **L58** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L59** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L60** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L61** EN: Defines function `expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 定义函数 `expand`，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L62** EN: Assigns or updates `new`. | CN: 对 `new` 进行赋值或更新。
- **L63** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L64** EN: Updates object state via `new.temperature`. | CN: 通过 `new.temperature` 更新对象状态。
- **L65** EN: Updates object state via `new._categorical`. | CN: 通过 `new._categorical` 更新对象状态。
- **L66** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L67** EN: Continues `ExpRelaxedCategorical.expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 继续 `ExpRelaxedCategorical.expand` 的实现，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L68** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L69** EN: Updates object state via `new._validate_args`. | CN: 通过 `new._validate_args` 更新对象状态。
- **L70** EN: Returns from `ExpRelaxedCategorical.expand` with the computed result or updated state. | CN: 从 `ExpRelaxedCategorical.expand` 返回计算结果或更新后的状态。
- **L71** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L72** EN: Defines function `_new`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_new`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L73** EN: Returns from `ExpRelaxedCategorical._new` with the computed result or updated state. | CN: 从 `ExpRelaxedCategorical._new` 返回计算结果或更新后的状态。
- **L74** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L75** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L76** EN: Defines function `param_shape`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `param_shape`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L77** EN: Returns from `ExpRelaxedCategorical.param_shape` with the computed result or updated state. | CN: 从 `ExpRelaxedCategorical.param_shape` 返回计算结果或更新后的状态。
- **L78** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 79-95 / 第 79-95 行

````python
0079:     @property
0080:     def logits(self) -> Tensor:
0081:         return self._categorical.logits
0082: 
0083:     @property
0084:     def probs(self) -> Tensor:
0085:         return self._categorical.probs
0086: 
0087:     def rsample(self, sample_shape: _size = torch.Size()) -> Tensor:
0088:         shape = self._extended_shape(sample_shape)
0089:         uniforms = clamp_probs(
0090:             torch.rand(shape, dtype=self.logits.dtype, device=self.logits.device)
0091:         )
0092:         gumbels = -((-(uniforms.log())).log())
0093:         scores = (self.logits + gumbels) / self.temperature
0094:         return scores - scores.logsumexp(dim=-1, keepdim=True)
0095: 
````

- **L79** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L80** EN: Defines function `logits`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `logits`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L81** EN: Returns from `ExpRelaxedCategorical.logits` with the computed result or updated state. | CN: 从 `ExpRelaxedCategorical.logits` 返回计算结果或更新后的状态。
- **L82** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L83** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L84** EN: Defines function `probs`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `probs`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L85** EN: Returns from `ExpRelaxedCategorical.probs` with the computed result or updated state. | CN: 从 `ExpRelaxedCategorical.probs` 返回计算结果或更新后的状态。
- **L86** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L87** EN: Defines function `rsample`, which produces samples according to the modeled stochastic behavior. | CN: 定义函数 `rsample`，其作用是根据建模的随机行为生成样本。
- **L88** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L89** EN: Assigns or updates `uniforms`. | CN: 对 `uniforms` 进行赋值或更新。
- **L90** EN: Invokes `torch.rand` to advance the surrounding implementation. | CN: 调用 `torch.rand` 来推进周围的实现逻辑。
- **L91** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L92** EN: Assigns or updates `gumbels`. | CN: 对 `gumbels` 进行赋值或更新。
- **L93** EN: Assigns or updates `scores`. | CN: 对 `scores` 进行赋值或更新。
- **L94** EN: Returns from `ExpRelaxedCategorical.rsample` with the computed result or updated state. | CN: 从 `ExpRelaxedCategorical.rsample` 返回计算结果或更新后的状态。
- **L95** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 96-117 / 第 96-117 行

````python
0096:     def log_prob(self, value):
0097:         K = self._categorical._num_events
0098:         if self._validate_args:
0099:             self._validate_sample(value)
0100:         logits, value = broadcast_all(self.logits, value)
0101:         log_scale = torch.full_like(
0102:             self.temperature, float(K)
0103:         ).lgamma() - self.temperature.log().mul(-(K - 1))
0104:         score = logits - value.mul(self.temperature)
0105:         score = (score - score.logsumexp(dim=-1, keepdim=True)).sum(-1)
0106:         return score + log_scale
0107: 
0108: 
0109: class RelaxedOneHotCategorical(TransformedDistribution):
0110:     r"""
0111:     Creates a RelaxedOneHotCategorical distribution parametrized by
0112:     :attr:`temperature`, and either :attr:`probs` or :attr:`logits`.
0113:     This is a relaxed version of the :class:`OneHotCategorical` distribution, so
0114:     its samples are on simplex, and are reparametrizable.
0115: 
0116:     Example::
0117: 
````

- **L96** EN: Defines function `log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 定义函数 `log_prob`，其作用是为给定输入计算对数概率或类似密度分数。
- **L97** EN: Assigns module-level configuration or cached state to `K`. | CN: 为 `K` 赋予模块级配置或缓存状态。
- **L98** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L99** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L100** EN: Invokes `broadcast_all` to advance the surrounding implementation. | CN: 调用 `broadcast_all` 来推进周围的实现逻辑。
- **L101** EN: Assigns or updates `log_scale`. | CN: 对 `log_scale` 进行赋值或更新。
- **L102** EN: Invokes `float` to advance the surrounding implementation. | CN: 调用 `float` 来推进周围的实现逻辑。
- **L103** EN: Invokes `lgamma` to advance the surrounding implementation. | CN: 调用 `lgamma` 来推进周围的实现逻辑。
- **L104** EN: Assigns or updates `score`. | CN: 对 `score` 进行赋值或更新。
- **L105** EN: Assigns or updates `score`. | CN: 对 `score` 进行赋值或更新。
- **L106** EN: Returns from `ExpRelaxedCategorical.log_prob` with the computed result or updated state. | CN: 从 `ExpRelaxedCategorical.log_prob` 返回计算结果或更新后的状态。
- **L107** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L108** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L109** EN: Defines class `RelaxedOneHotCategorical` with bases `TransformedDistribution`, which encapsulates distribution parameters, validation rules, and tensor-valued statistical behavior. | CN: 定义类 `RelaxedOneHotCategorical`，其基类为 `TransformedDistribution`，作用是封装分布参数、校验规则以及基于张量的统计行为。
- **L110** EN: Starts the docstring for class `RelaxedOneHotCategorical`. | CN: 开始为 class `RelaxedOneHotCategorical` 编写文档字符串。
- **L111** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L112** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L113** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L114** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L115** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L116** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L117** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 118-136 / 第 118-136 行

````python
0118:         >>> # xdoctest: +IGNORE_WANT("non-deterministic")
0119:         >>> m = RelaxedOneHotCategorical(torch.tensor([2.2]),
0120:         ...                              torch.tensor([0.1, 0.2, 0.3, 0.4]))
0121:         >>> m.sample()
0122:         tensor([ 0.1294,  0.2324,  0.3859,  0.2523])
0123: 
0124:     Args:
0125:         temperature (Tensor): relaxation temperature
0126:         probs (Tensor): event probabilities
0127:         logits (Tensor): unnormalized log probability for each event
0128:     """
0129: 
0130:     arg_constraints = {"probs": constraints.simplex, "logits": constraints.real_vector}
0131:     # pyrefly: ignore [bad-override]
0132:     support = constraints.simplex
0133:     has_rsample = True
0134:     # pyrefly: ignore [bad-override]
0135:     base_dist: ExpRelaxedCategorical
0136: 
````

- **L118** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L119** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L120** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L121** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L122** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L123** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L124** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L125** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L126** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L127** EN: Continues the docstring for class `RelaxedOneHotCategorical`. | CN: 继续补充 class `RelaxedOneHotCategorical` 的文档字符串。
- **L128** EN: Ends the docstring for class `RelaxedOneHotCategorical`. | CN: 结束 class `RelaxedOneHotCategorical` 的文档字符串。
- **L129** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L130** EN: Sets structural metadata on `arg_constraints` that other routines consult later. | CN: 为 `arg_constraints` 设置结构化元数据，供后续例程查询。
- **L131** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L132** EN: Sets structural metadata on `support` that other routines consult later. | CN: 为 `support` 设置结构化元数据，供后续例程查询。
- **L133** EN: Assigns or updates `has_rsample`. | CN: 对 `has_rsample` 进行赋值或更新。
- **L134** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L135** EN: Continues class `RelaxedOneHotCategorical`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `RelaxedOneHotCategorical` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L136** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 137-156 / 第 137-156 行

````python
0137:     def __init__(
0138:         self,
0139:         temperature: Tensor,
0140:         probs: Tensor | None = None,
0141:         logits: Tensor | None = None,
0142:         validate_args: bool | None = None,
0143:     ) -> None:
0144:         base_dist = ExpRelaxedCategorical(
0145:             temperature, probs, logits, validate_args=validate_args
0146:         )
0147:         super().__init__(base_dist, ExpTransform(), validate_args=validate_args)
0148: 
0149:     def expand(self, batch_shape, _instance=None):
0150:         new = self._get_checked_instance(RelaxedOneHotCategorical, _instance)
0151:         return super().expand(batch_shape, _instance=new)
0152: 
0153:     @property
0154:     def temperature(self) -> Tensor:
0155:         return self.base_dist.temperature
0156: 
````

- **L137** EN: Defines function `__init__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__init__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L138** EN: Continues `RelaxedOneHotCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `RelaxedOneHotCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L139** EN: Continues `RelaxedOneHotCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `RelaxedOneHotCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L140** EN: Continues `RelaxedOneHotCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `RelaxedOneHotCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L141** EN: Continues `RelaxedOneHotCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `RelaxedOneHotCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L142** EN: Continues `RelaxedOneHotCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `RelaxedOneHotCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L143** EN: Continues `RelaxedOneHotCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `RelaxedOneHotCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L144** EN: Assigns or updates `base_dist`. | CN: 对 `base_dist` 进行赋值或更新。
- **L145** EN: Continues `RelaxedOneHotCategorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `RelaxedOneHotCategorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L146** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L147** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L148** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L149** EN: Defines function `expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 定义函数 `expand`，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L150** EN: Assigns or updates `new`. | CN: 对 `new` 进行赋值或更新。
- **L151** EN: Returns from `RelaxedOneHotCategorical.expand` with the computed result or updated state. | CN: 从 `RelaxedOneHotCategorical.expand` 返回计算结果或更新后的状态。
- **L152** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L153** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L154** EN: Defines function `temperature`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `temperature`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L155** EN: Returns from `RelaxedOneHotCategorical.temperature` with the computed result or updated state. | CN: 从 `RelaxedOneHotCategorical.temperature` 返回计算结果或更新后的状态。
- **L156** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 157-163 / 第 157-163 行

````python
0157:     @property
0158:     def logits(self) -> Tensor:
0159:         return self.base_dist.logits
0160: 
0161:     @property
0162:     def probs(self) -> Tensor:
0163:         return self.base_dist.probs
````

- **L157** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L158** EN: Defines function `logits`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `logits`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L159** EN: Returns from `RelaxedOneHotCategorical.logits` with the computed result or updated state. | CN: 从 `RelaxedOneHotCategorical.logits` 返回计算结果或更新后的状态。
- **L160** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L161** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L162** EN: Defines function `probs`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `probs`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L163** EN: Returns from `RelaxedOneHotCategorical.probs` with the computed result or updated state. | CN: 从 `RelaxedOneHotCategorical.probs` 返回计算结果或更新后的状态。

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

- **Torch imports / Torch 导入**: `torch`、`torch:Tensor`、`torch.distributions:constraints`、`torch.distributions.categorical:Categorical`、`torch.distributions.distribution:Distribution`、`torch.distributions.transformed_distribution:TransformedDistribution`、`torch.distributions.transforms:ExpTransform`、`torch.distributions.utils:broadcast_all, clamp_probs`、`torch.types:_size`
- **Other imports / 其他导入**: 无
- **Top-level classes / 顶层类**: `ExpRelaxedCategorical`、`RelaxedOneHotCategorical`
- **Top-level functions / 顶层函数**: 无
- **Base classes / 基类**: `Distribution`、`TransformedDistribution`
- **Decorators / 装饰器**: 无
- **Module assignments / 模块级赋值**: `__all__`
