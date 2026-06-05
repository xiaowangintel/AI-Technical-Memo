# multinomial.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/distributions/multinomial.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements the `Multinomial` distribution or helper type, including parameter handling, sampling, and statistical methods.
- **Purpose (CN)**: 实现 `Multinomial` 分布或辅助类型，涵盖参数处理、采样与统计方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22 / 第 1-22 行

````python
0001: # mypy: allow-untyped-defs
0002: 
0003: import torch
0004: from torch import inf, Tensor
0005: from torch.distributions import Categorical, constraints
0006: from torch.distributions.binomial import Binomial
0007: from torch.distributions.distribution import Distribution
0008: from torch.distributions.utils import broadcast_all
0009: 
0010: 
0011: __all__ = ["Multinomial"]
0012: 
0013: 
0014: class Multinomial(Distribution):
0015:     r"""
0016:     Creates a Multinomial distribution parameterized by :attr:`total_count` and
0017:     either :attr:`probs` or :attr:`logits` (but not both). The innermost dimension of
0018:     :attr:`probs` indexes over categories. All other dimensions index over batches.
0019: 
0020:     Note that :attr:`total_count` need not be specified if only :meth:`log_prob` is
0021:     called (see example below)
0022: 
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L3** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L4** EN: Imports `inf, Tensor` from `torch` so later code can reuse those definitions. | CN: 从 `torch` 导入 `inf, Tensor`，供后续代码复用这些定义。
- **L5** EN: Imports `Categorical, constraints` from `torch.distributions` so later code can reuse those definitions. | CN: 从 `torch.distributions` 导入 `Categorical, constraints`，供后续代码复用这些定义。
- **L6** EN: Imports `Binomial` from `torch.distributions.binomial` so later code can reuse those definitions. | CN: 从 `torch.distributions.binomial` 导入 `Binomial`，供后续代码复用这些定义。
- **L7** EN: Imports `Distribution` from `torch.distributions.distribution` so later code can reuse those definitions. | CN: 从 `torch.distributions.distribution` 导入 `Distribution`，供后续代码复用这些定义。
- **L8** EN: Imports `broadcast_all` from `torch.distributions.utils` so later code can reuse those definitions. | CN: 从 `torch.distributions.utils` 导入 `broadcast_all`，供后续代码复用这些定义。
- **L9** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L10** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L11** EN: Defines or updates the module’s public export list. | CN: 定义或更新模块的公开导出列表。
- **L12** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L13** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L14** EN: Defines class `Multinomial` with bases `Distribution`, which encapsulates distribution parameters, validation rules, and tensor-valued statistical behavior. | CN: 定义类 `Multinomial`，其基类为 `Distribution`，作用是封装分布参数、校验规则以及基于张量的统计行为。
- **L15** EN: Starts the docstring for class `Multinomial`. | CN: 开始为 class `Multinomial` 编写文档字符串。
- **L16** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L17** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L18** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L19** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L20** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L21** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L22** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 23-42 / 第 23-42 行

````python
0023:     .. note:: The `probs` argument must be non-negative, finite and have a non-zero sum,
0024:               and it will be normalized to sum to 1 along the last dimension. :attr:`probs`
0025:               will return this normalized value.
0026:               The `logits` argument will be interpreted as unnormalized log probabilities
0027:               and can therefore be any real number. It will likewise be normalized so that
0028:               the resulting probabilities sum to 1 along the last dimension. :attr:`logits`
0029:               will return this normalized value.
0030: 
0031:     -   :meth:`sample` requires a single shared `total_count` for all
0032:         parameters and samples.
0033:     -   :meth:`log_prob` allows different `total_count` for each parameter and
0034:         sample.
0035: 
0036:     Example::
0037: 
0038:         >>> # xdoctest: +SKIP("FIXME: found invalid values")
0039:         >>> m = Multinomial(100, torch.tensor([ 1., 1., 1., 1.]))
0040:         >>> x = m.sample()  # equal probability of 0, 1, 2, 3
0041:         tensor([ 21.,  24.,  30.,  25.])
0042: 
````

- **L23** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L24** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L25** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L26** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L27** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L28** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L29** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L30** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L31** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L32** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L33** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L34** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L35** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L36** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L37** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L38** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L39** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L40** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L41** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L42** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 43-63 / 第 43-63 行

````python
0043:         >>> Multinomial(probs=torch.tensor([1., 1., 1., 1.])).log_prob(x)
0044:         tensor([-4.1338])
0045: 
0046:     Args:
0047:         total_count (int): number of trials
0048:         probs (Tensor): event probabilities
0049:         logits (Tensor): event log probabilities (unnormalized)
0050:     """
0051: 
0052:     # pyrefly: ignore [bad-override]
0053:     arg_constraints = {"probs": constraints.simplex, "logits": constraints.real_vector}
0054:     total_count: int
0055: 
0056:     @property
0057:     def mean(self) -> Tensor:
0058:         return self.probs * self.total_count
0059: 
0060:     @property
0061:     def variance(self) -> Tensor:
0062:         return self.total_count * self.probs * (1 - self.probs)
0063: 
````

- **L43** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L44** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L45** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L46** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L47** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L48** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L49** EN: Continues the docstring for class `Multinomial`. | CN: 继续补充 class `Multinomial` 的文档字符串。
- **L50** EN: Ends the docstring for class `Multinomial`. | CN: 结束 class `Multinomial` 的文档字符串。
- **L51** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L52** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L53** EN: Sets structural metadata on `arg_constraints` that other routines consult later. | CN: 为 `arg_constraints` 设置结构化元数据，供后续例程查询。
- **L54** EN: Continues class `Multinomial`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `Multinomial` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L55** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L56** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L57** EN: Defines function `mean`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mean`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L58** EN: Returns from `Multinomial.mean` with the computed result or updated state. | CN: 从 `Multinomial.mean` 返回计算结果或更新后的状态。
- **L59** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L60** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L61** EN: Defines function `variance`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `variance`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L62** EN: Returns from `Multinomial.variance` with the computed result or updated state. | CN: 从 `Multinomial.variance` 返回计算结果或更新后的状态。
- **L63** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 64-80 / 第 64-80 行

````python
0064:     def __init__(
0065:         self,
0066:         total_count: int = 1,
0067:         probs: Tensor | None = None,
0068:         logits: Tensor | None = None,
0069:         validate_args: bool | None = None,
0070:     ) -> None:
0071:         if not isinstance(total_count, int):
0072:             raise NotImplementedError("inhomogeneous total_count is not supported")
0073:         self.total_count = total_count
0074:         self._categorical = Categorical(probs=probs, logits=logits)
0075:         self._binomial = Binomial(total_count=total_count, probs=self.probs)
0076:         batch_shape = self._categorical.batch_shape
0077:         event_shape = self._categorical.param_shape[-1:]
0078:         # pyrefly: ignore [bad-argument-type]
0079:         super().__init__(batch_shape, event_shape, validate_args=validate_args)
0080: 
````

- **L64** EN: Defines function `__init__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__init__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L65** EN: Continues `Multinomial.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Multinomial.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L66** EN: Continues `Multinomial.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Multinomial.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L67** EN: Continues `Multinomial.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Multinomial.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L68** EN: Continues `Multinomial.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Multinomial.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L69** EN: Continues `Multinomial.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Multinomial.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L70** EN: Continues `Multinomial.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Multinomial.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L71** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L72** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L73** EN: Updates object state via `self.total_count`. | CN: 通过 `self.total_count` 更新对象状态。
- **L74** EN: Updates object state via `self._categorical`. | CN: 通过 `self._categorical` 更新对象状态。
- **L75** EN: Updates object state via `self._binomial`. | CN: 通过 `self._binomial` 更新对象状态。
- **L76** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L77** EN: Assigns or updates `event_shape`. | CN: 对 `event_shape` 进行赋值或更新。
- **L78** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L79** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L80** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 81-99 / 第 81-99 行

````python
0081:     def expand(self, batch_shape, _instance=None):
0082:         new = self._get_checked_instance(Multinomial, _instance)
0083:         batch_shape = torch.Size(batch_shape)
0084:         new.total_count = self.total_count
0085:         new._categorical = self._categorical.expand(batch_shape)
0086:         super(Multinomial, new).__init__(
0087:             batch_shape, self.event_shape, validate_args=False
0088:         )
0089:         new._validate_args = self._validate_args
0090:         return new
0091: 
0092:     def _new(self, *args, **kwargs):
0093:         return self._categorical._new(*args, **kwargs)
0094: 
0095:     @constraints.dependent_property(is_discrete=True, event_dim=1)
0096:     # pyrefly: ignore [bad-override]
0097:     def support(self):
0098:         return constraints.multinomial(self.total_count)
0099: 
````

- **L81** EN: Defines function `expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 定义函数 `expand`，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L82** EN: Assigns or updates `new`. | CN: 对 `new` 进行赋值或更新。
- **L83** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L84** EN: Updates object state via `new.total_count`. | CN: 通过 `new.total_count` 更新对象状态。
- **L85** EN: Updates object state via `new._categorical`. | CN: 通过 `new._categorical` 更新对象状态。
- **L86** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L87** EN: Continues `Multinomial.expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 继续 `Multinomial.expand` 的实现，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L88** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L89** EN: Updates object state via `new._validate_args`. | CN: 通过 `new._validate_args` 更新对象状态。
- **L90** EN: Returns from `Multinomial.expand` with the computed result or updated state. | CN: 从 `Multinomial.expand` 返回计算结果或更新后的状态。
- **L91** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L92** EN: Defines function `_new`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_new`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L93** EN: Returns from `Multinomial._new` with the computed result or updated state. | CN: 从 `Multinomial._new` 返回计算结果或更新后的状态。
- **L94** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L95** EN: Applies decorator `constraints.dependent_property`, which modifies the behavior of the following definition. | CN: 应用装饰器 `constraints.dependent_property`，其作用是修改后续定义的行为。
- **L96** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L97** EN: Defines function `support`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `support`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L98** EN: Returns from `Multinomial.support` with the computed result or updated state. | CN: 从 `Multinomial.support` 返回计算结果或更新后的状态。
- **L99** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 100-121 / 第 100-121 行

````python
0100:     @property
0101:     def logits(self) -> Tensor:
0102:         return self._categorical.logits
0103: 
0104:     @property
0105:     def probs(self) -> Tensor:
0106:         return self._categorical.probs
0107: 
0108:     @property
0109:     def param_shape(self) -> torch.Size:
0110:         return self._categorical.param_shape
0111: 
0112:     def sample(self, sample_shape=torch.Size()):
0113:         sample_shape = torch.Size(sample_shape)
0114:         samples = self._categorical.sample(
0115:             torch.Size((self.total_count,)) + sample_shape
0116:         )
0117:         # samples.shape is (total_count, sample_shape, batch_shape), need to change it to
0118:         # (sample_shape, batch_shape, total_count)
0119:         shifted_idx = list(range(samples.dim()))
0120:         shifted_idx.append(shifted_idx.pop(0))
0121:         samples = samples.permute(*shifted_idx)
````

- **L100** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L101** EN: Defines function `logits`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `logits`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L102** EN: Returns from `Multinomial.logits` with the computed result or updated state. | CN: 从 `Multinomial.logits` 返回计算结果或更新后的状态。
- **L103** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L104** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L105** EN: Defines function `probs`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `probs`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L106** EN: Returns from `Multinomial.probs` with the computed result or updated state. | CN: 从 `Multinomial.probs` 返回计算结果或更新后的状态。
- **L107** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L108** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L109** EN: Defines function `param_shape`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `param_shape`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L110** EN: Returns from `Multinomial.param_shape` with the computed result or updated state. | CN: 从 `Multinomial.param_shape` 返回计算结果或更新后的状态。
- **L111** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L112** EN: Defines function `sample`, which produces samples according to the modeled stochastic behavior. | CN: 定义函数 `sample`，其作用是根据建模的随机行为生成样本。
- **L113** EN: Assigns or updates `sample_shape`. | CN: 对 `sample_shape` 进行赋值或更新。
- **L114** EN: Assigns or updates `samples`. | CN: 对 `samples` 进行赋值或更新。
- **L115** EN: Invokes `torch.Size` to advance the surrounding implementation. | CN: 调用 `torch.Size` 来推进周围的实现逻辑。
- **L116** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L117** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L118** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L119** EN: Assigns or updates `shifted_idx`. | CN: 对 `shifted_idx` 进行赋值或更新。
- **L120** EN: Invokes `shifted_idx.append` to advance the surrounding implementation. | CN: 调用 `shifted_idx.append` 来推进周围的实现逻辑。
- **L121** EN: Assigns or updates `samples`. | CN: 对 `samples` 进行赋值或更新。

### Lines 122-138 / 第 122-138 行

````python
0122:         counts = samples.new(self._extended_shape(sample_shape)).zero_()
0123:         counts.scatter_add_(-1, samples, torch.ones_like(samples))
0124:         return counts.type_as(self.probs)
0125: 
0126:     def entropy(self):
0127:         n = torch.tensor(self.total_count)
0128: 
0129:         cat_entropy = self._categorical.entropy()
0130:         term1 = n * cat_entropy - torch.lgamma(n + 1)
0131: 
0132:         support = self._binomial.enumerate_support(expand=False)[1:]
0133:         binomial_probs = torch.exp(self._binomial.log_prob(support))
0134:         weights = torch.lgamma(support + 1)
0135:         term2 = (binomial_probs * weights).sum([0, -1])
0136: 
0137:         return term1 + term2
0138: 
````

- **L122** EN: Assigns or updates `counts`. | CN: 对 `counts` 进行赋值或更新。
- **L123** EN: Invokes `counts.scatter_add_` to advance the surrounding implementation. | CN: 调用 `counts.scatter_add_` 来推进周围的实现逻辑。
- **L124** EN: Returns from `Multinomial.sample` with the computed result or updated state. | CN: 从 `Multinomial.sample` 返回计算结果或更新后的状态。
- **L125** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L126** EN: Defines function `entropy`, which computes an entropy-style summary of uncertainty. | CN: 定义函数 `entropy`，其作用是计算描述不确定性的熵类指标。
- **L127** EN: Assigns or updates `n`. | CN: 对 `n` 进行赋值或更新。
- **L128** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L129** EN: Assigns or updates `cat_entropy`. | CN: 对 `cat_entropy` 进行赋值或更新。
- **L130** EN: Assigns or updates `term1`. | CN: 对 `term1` 进行赋值或更新。
- **L131** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L132** EN: Sets structural metadata on `support` that other routines consult later. | CN: 为 `support` 设置结构化元数据，供后续例程查询。
- **L133** EN: Assigns or updates `binomial_probs`. | CN: 对 `binomial_probs` 进行赋值或更新。
- **L134** EN: Assigns or updates `weights`. | CN: 对 `weights` 进行赋值或更新。
- **L135** EN: Assigns or updates `term2`. | CN: 对 `term2` 进行赋值或更新。
- **L136** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L137** EN: Returns from `Multinomial.entropy` with the computed result or updated state. | CN: 从 `Multinomial.entropy` 返回计算结果或更新后的状态。
- **L138** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 139-148 / 第 139-148 行

````python
0139:     def log_prob(self, value):
0140:         if self._validate_args:
0141:             self._validate_sample(value)
0142:         logits, value = broadcast_all(self.logits, value)
0143:         logits = logits.clone(memory_format=torch.contiguous_format)
0144:         log_factorial_n = torch.lgamma(value.sum(-1) + 1)
0145:         log_factorial_xs = torch.lgamma(value + 1).sum(-1)
0146:         logits[(value == 0) & (logits == -inf)] = 0
0147:         log_powers = (logits * value).sum(-1)
0148:         return log_factorial_n - log_factorial_xs + log_powers
````

- **L139** EN: Defines function `log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 定义函数 `log_prob`，其作用是为给定输入计算对数概率或类似密度分数。
- **L140** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L141** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L142** EN: Invokes `broadcast_all` to advance the surrounding implementation. | CN: 调用 `broadcast_all` 来推进周围的实现逻辑。
- **L143** EN: Assigns or updates `logits`. | CN: 对 `logits` 进行赋值或更新。
- **L144** EN: Assigns or updates `log_factorial_n`. | CN: 对 `log_factorial_n` 进行赋值或更新。
- **L145** EN: Assigns or updates `log_factorial_xs`. | CN: 对 `log_factorial_xs` 进行赋值或更新。
- **L146** EN: Continues `Multinomial.log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 继续 `Multinomial.log_prob` 的实现，其作用是为给定输入计算对数概率或类似密度分数。
- **L147** EN: Assigns or updates `log_powers`. | CN: 对 `log_powers` 进行赋值或更新。
- **L148** EN: Returns from `Multinomial.log_prob` with the computed result or updated state. | CN: 从 `Multinomial.log_prob` 返回计算结果或更新后的状态。

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

- **Torch imports / Torch 导入**: `torch`、`torch:inf, Tensor`、`torch.distributions:Categorical, constraints`、`torch.distributions.binomial:Binomial`、`torch.distributions.distribution:Distribution`、`torch.distributions.utils:broadcast_all`
- **Other imports / 其他导入**: 无
- **Top-level classes / 顶层类**: `Multinomial`
- **Top-level functions / 顶层函数**: 无
- **Base classes / 基类**: `Distribution`
- **Decorators / 装饰器**: 无
- **Module assignments / 模块级赋值**: `__all__`
