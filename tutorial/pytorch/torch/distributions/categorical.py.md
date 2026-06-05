# categorical.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/distributions/categorical.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements the `Categorical` distribution or helper type, including parameter handling, sampling, and statistical methods.
- **Purpose (CN)**: 实现 `Categorical` 分布或辅助类型，涵盖参数处理、采样与统计方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21 / 第 1-21 行

````python
0001: # mypy: allow-untyped-defs
0002: 
0003: import torch
0004: from torch import nan, Tensor
0005: from torch.distributions import constraints
0006: from torch.distributions.distribution import Distribution
0007: from torch.distributions.utils import lazy_property, logits_to_probs, probs_to_logits
0008: 
0009: 
0010: __all__ = ["Categorical"]
0011: 
0012: 
0013: class Categorical(Distribution):
0014:     r"""
0015:     Creates a categorical distribution parameterized by either :attr:`probs` or
0016:     :attr:`logits` (but not both).
0017: 
0018:     .. note::
0019:         It is equivalent to the distribution that :func:`torch.multinomial`
0020:         samples from.
0021: 
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L3** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L4** EN: Imports `nan, Tensor` from `torch` so later code can reuse those definitions. | CN: 从 `torch` 导入 `nan, Tensor`，供后续代码复用这些定义。
- **L5** EN: Imports `constraints` from `torch.distributions` so later code can reuse those definitions. | CN: 从 `torch.distributions` 导入 `constraints`，供后续代码复用这些定义。
- **L6** EN: Imports `Distribution` from `torch.distributions.distribution` so later code can reuse those definitions. | CN: 从 `torch.distributions.distribution` 导入 `Distribution`，供后续代码复用这些定义。
- **L7** EN: Imports `lazy_property, logits_to_probs, probs_to_logits` from `torch.distributions.utils` so later code can reuse those definitions. | CN: 从 `torch.distributions.utils` 导入 `lazy_property, logits_to_probs, probs_to_logits`，供后续代码复用这些定义。
- **L8** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L9** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L10** EN: Defines or updates the module’s public export list. | CN: 定义或更新模块的公开导出列表。
- **L11** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L12** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L13** EN: Defines class `Categorical` with bases `Distribution`, which encapsulates distribution parameters, validation rules, and tensor-valued statistical behavior. | CN: 定义类 `Categorical`，其基类为 `Distribution`，作用是封装分布参数、校验规则以及基于张量的统计行为。
- **L14** EN: Starts the docstring for class `Categorical`. | CN: 开始为 class `Categorical` 编写文档字符串。
- **L15** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L16** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L17** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L18** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L19** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L20** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L21** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 22-41 / 第 22-41 行

````python
0022:     Samples are integers from :math:`\{0, \ldots, K-1\}` where `K` is ``probs.size(-1)``.
0023: 
0024:     If `probs` is 1-dimensional with length-`K`, each element is the relative probability
0025:     of sampling the class at that index.
0026: 
0027:     If `probs` is N-dimensional, the first N-1 dimensions are treated as a batch of
0028:     relative probability vectors.
0029: 
0030:     .. note:: The `probs` argument must be non-negative, finite and have a non-zero sum,
0031:               and it will be normalized to sum to 1 along the last dimension. :attr:`probs`
0032:               will return this normalized value.
0033:               The `logits` argument will be interpreted as unnormalized log probabilities
0034:               and can therefore be any real number. It will likewise be normalized so that
0035:               the resulting probabilities sum to 1 along the last dimension. :attr:`logits`
0036:               will return this normalized value.
0037: 
0038:     See also: :func:`torch.multinomial`
0039: 
0040:     Example::
0041: 
````

- **L22** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L23** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L24** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L25** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L26** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L27** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L28** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L29** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L30** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L31** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L32** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L33** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L34** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L35** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L36** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L37** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L38** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L39** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L40** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L41** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 42-63 / 第 42-63 行

````python
0042:         >>> # xdoctest: +IGNORE_WANT("non-deterministic")
0043:         >>> m = Categorical(torch.tensor([ 0.25, 0.25, 0.25, 0.25 ]))
0044:         >>> m.sample()  # equal probability of 0, 1, 2, 3
0045:         tensor(3)
0046: 
0047:     Args:
0048:         probs (Tensor): event probabilities
0049:         logits (Tensor): event log probabilities (unnormalized)
0050:     """
0051: 
0052:     # pyrefly: ignore [bad-override]
0053:     arg_constraints = {"probs": constraints.simplex, "logits": constraints.real_vector}
0054:     has_enumerate_support = True
0055: 
0056:     def __init__(
0057:         self,
0058:         probs: Tensor | None = None,
0059:         logits: Tensor | None = None,
0060:         validate_args: bool | None = None,
0061:     ) -> None:
0062:         if (probs is None) == (logits is None):
0063:             raise ValueError(
````

- **L42** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L43** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L44** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L45** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L46** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L47** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L48** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L49** EN: Continues the docstring for class `Categorical`. | CN: 继续补充 class `Categorical` 的文档字符串。
- **L50** EN: Ends the docstring for class `Categorical`. | CN: 结束 class `Categorical` 的文档字符串。
- **L51** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L52** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L53** EN: Sets structural metadata on `arg_constraints` that other routines consult later. | CN: 为 `arg_constraints` 设置结构化元数据，供后续例程查询。
- **L54** EN: Sets structural metadata on `has_enumerate_support` that other routines consult later. | CN: 为 `has_enumerate_support` 设置结构化元数据，供后续例程查询。
- **L55** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L56** EN: Defines function `__init__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__init__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L57** EN: Continues `Categorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Categorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L58** EN: Continues `Categorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Categorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L59** EN: Continues `Categorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Categorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L60** EN: Continues `Categorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Categorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L61** EN: Continues `Categorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Categorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L62** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L63** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。

### Lines 64-85 / 第 64-85 行

````python
0064:                 "Either `probs` or `logits` must be specified, but not both."
0065:             )
0066:         if probs is not None:
0067:             if probs.dim() < 1:
0068:                 raise ValueError("`probs` parameter must be at least one-dimensional.")
0069:             # pyrefly: ignore [read-only]
0070:             self.probs = probs / probs.sum(-1, keepdim=True)
0071:         else:
0072:             if logits is None:
0073:                 raise AssertionError("logits is unexpectedly None")
0074:             if logits.dim() < 1:
0075:                 raise ValueError("`logits` parameter must be at least one-dimensional.")
0076:             # Normalize
0077:             # pyrefly: ignore [read-only]
0078:             self.logits = logits - logits.logsumexp(dim=-1, keepdim=True)
0079:         self._param = self.probs if probs is not None else self.logits
0080:         self._num_events = self._param.size()[-1]
0081:         batch_shape = (
0082:             self._param.size()[:-1] if self._param.ndimension() > 1 else torch.Size()
0083:         )
0084:         # pyrefly: ignore [bad-argument-type]
0085:         super().__init__(batch_shape, validate_args=validate_args)
````

- **L64** EN: Continues `Categorical.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Categorical.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L65** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L66** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L67** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L68** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L69** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L70** EN: Updates object state via `self.probs`. | CN: 通过 `self.probs` 更新对象状态。
- **L71** EN: Introduces the fallback branch when earlier conditions are not met. | CN: 引入前置条件不满足时的后备分支。
- **L72** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L73** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L74** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L75** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L76** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L77** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L78** EN: Updates object state via `self.logits`. | CN: 通过 `self.logits` 更新对象状态。
- **L79** EN: Updates object state via `self._param`. | CN: 通过 `self._param` 更新对象状态。
- **L80** EN: Updates object state via `self._num_events`. | CN: 通过 `self._num_events` 更新对象状态。
- **L81** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L82** EN: Invokes `self._param.size` to advance the surrounding implementation. | CN: 调用 `self._param.size` 来推进周围的实现逻辑。
- **L83** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L84** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L85** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。

### Lines 86-104 / 第 86-104 行

````python
0086: 
0087:     def expand(self, batch_shape, _instance=None):
0088:         new = self._get_checked_instance(Categorical, _instance)
0089:         batch_shape = torch.Size(batch_shape)
0090:         param_shape = batch_shape + torch.Size((self._num_events,))
0091:         if "probs" in self.__dict__:
0092:             new.probs = self.probs.expand(param_shape)
0093:             new._param = new.probs
0094:         if "logits" in self.__dict__:
0095:             new.logits = self.logits.expand(param_shape)
0096:             new._param = new.logits
0097:         new._num_events = self._num_events
0098:         super(Categorical, new).__init__(batch_shape, validate_args=False)
0099:         new._validate_args = self._validate_args
0100:         return new
0101: 
0102:     def _new(self, *args, **kwargs):
0103:         return self._param.new(*args, **kwargs)
0104: 
````

- **L86** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L87** EN: Defines function `expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 定义函数 `expand`，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L88** EN: Assigns or updates `new`. | CN: 对 `new` 进行赋值或更新。
- **L89** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L90** EN: Assigns or updates `param_shape`. | CN: 对 `param_shape` 进行赋值或更新。
- **L91** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L92** EN: Updates object state via `new.probs`. | CN: 通过 `new.probs` 更新对象状态。
- **L93** EN: Updates object state via `new._param`. | CN: 通过 `new._param` 更新对象状态。
- **L94** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L95** EN: Updates object state via `new.logits`. | CN: 通过 `new.logits` 更新对象状态。
- **L96** EN: Updates object state via `new._param`. | CN: 通过 `new._param` 更新对象状态。
- **L97** EN: Updates object state via `new._num_events`. | CN: 通过 `new._num_events` 更新对象状态。
- **L98** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L99** EN: Updates object state via `new._validate_args`. | CN: 通过 `new._validate_args` 更新对象状态。
- **L100** EN: Returns from `Categorical.expand` with the computed result or updated state. | CN: 从 `Categorical.expand` 返回计算结果或更新后的状态。
- **L101** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L102** EN: Defines function `_new`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_new`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L103** EN: Returns from `Categorical._new` with the computed result or updated state. | CN: 从 `Categorical._new` 返回计算结果或更新后的状态。
- **L104** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 105-121 / 第 105-121 行

````python
0105:     @constraints.dependent_property(is_discrete=True, event_dim=0)
0106:     # pyrefly: ignore [bad-override]
0107:     def support(self):
0108:         return constraints.integer_interval(0, self._num_events - 1)
0109: 
0110:     @lazy_property
0111:     def logits(self) -> Tensor:
0112:         return probs_to_logits(self.probs)
0113: 
0114:     @lazy_property
0115:     def probs(self) -> Tensor:
0116:         return logits_to_probs(self.logits)
0117: 
0118:     @property
0119:     def param_shape(self) -> torch.Size:
0120:         return self._param.size()
0121: 
````

- **L105** EN: Applies decorator `constraints.dependent_property`, which modifies the behavior of the following definition. | CN: 应用装饰器 `constraints.dependent_property`，其作用是修改后续定义的行为。
- **L106** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L107** EN: Defines function `support`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `support`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L108** EN: Returns from `Categorical.support` with the computed result or updated state. | CN: 从 `Categorical.support` 返回计算结果或更新后的状态。
- **L109** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L110** EN: Applies decorator `lazy_property`, which turns a derived value into a lazily cached attribute. | CN: 应用装饰器 `lazy_property`，其作用是把派生值变成惰性缓存属性。
- **L111** EN: Defines function `logits`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `logits`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L112** EN: Returns from `Categorical.logits` with the computed result or updated state. | CN: 从 `Categorical.logits` 返回计算结果或更新后的状态。
- **L113** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L114** EN: Applies decorator `lazy_property`, which turns a derived value into a lazily cached attribute. | CN: 应用装饰器 `lazy_property`，其作用是把派生值变成惰性缓存属性。
- **L115** EN: Defines function `probs`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `probs`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L116** EN: Returns from `Categorical.probs` with the computed result or updated state. | CN: 从 `Categorical.probs` 返回计算结果或更新后的状态。
- **L117** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L118** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L119** EN: Defines function `param_shape`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `param_shape`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L120** EN: Returns from `Categorical.param_shape` with the computed result or updated state. | CN: 从 `Categorical.param_shape` 返回计算结果或更新后的状态。
- **L121** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 122-143 / 第 122-143 行

````python
0122:     @property
0123:     def mean(self) -> Tensor:
0124:         return torch.full(
0125:             self._extended_shape(),
0126:             nan,
0127:             dtype=self.probs.dtype,
0128:             device=self.probs.device,
0129:         )
0130: 
0131:     @property
0132:     def mode(self) -> Tensor:
0133:         return self.probs.argmax(dim=-1)
0134: 
0135:     @property
0136:     def variance(self) -> Tensor:
0137:         return torch.full(
0138:             self._extended_shape(),
0139:             nan,
0140:             dtype=self.probs.dtype,
0141:             device=self.probs.device,
0142:         )
0143: 
````

- **L122** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L123** EN: Defines function `mean`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mean`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L124** EN: Returns from `Categorical.mean` with the computed result or updated state. | CN: 从 `Categorical.mean` 返回计算结果或更新后的状态。
- **L125** EN: Invokes `self._extended_shape` to advance the surrounding implementation. | CN: 调用 `self._extended_shape` 来推进周围的实现逻辑。
- **L126** EN: Continues `Categorical.mean`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Categorical.mean` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L127** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L128** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L129** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L130** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L131** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L132** EN: Defines function `mode`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mode`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L133** EN: Returns from `Categorical.mode` with the computed result or updated state. | CN: 从 `Categorical.mode` 返回计算结果或更新后的状态。
- **L134** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L135** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L136** EN: Defines function `variance`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `variance`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L137** EN: Returns from `Categorical.variance` with the computed result or updated state. | CN: 从 `Categorical.variance` 返回计算结果或更新后的状态。
- **L138** EN: Invokes `self._extended_shape` to advance the surrounding implementation. | CN: 调用 `self._extended_shape` 来推进周围的实现逻辑。
- **L139** EN: Continues `Categorical.variance`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `Categorical.variance` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L140** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L141** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L142** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L143** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 144-164 / 第 144-164 行

````python
0144:     def sample(self, sample_shape=torch.Size()):
0145:         if not isinstance(sample_shape, torch.Size):
0146:             sample_shape = torch.Size(sample_shape)
0147:         probs_2d = self.probs.reshape(-1, self._num_events)
0148:         samples_2d = torch.multinomial(probs_2d, sample_shape.numel(), True).T
0149:         return samples_2d.reshape(self._extended_shape(sample_shape))
0150: 
0151:     def log_prob(self, value):
0152:         if self._validate_args:
0153:             self._validate_sample(value)
0154:         value = value.long().unsqueeze(-1)
0155:         value, log_pmf = torch.broadcast_tensors(value, self.logits)
0156:         value = value[..., :1]
0157:         return log_pmf.gather(-1, value).squeeze(-1)
0158: 
0159:     def entropy(self):
0160:         min_real = torch.finfo(self.logits.dtype).min
0161:         logits = torch.clamp(self.logits, min=min_real)
0162:         p_log_p = logits * self.probs
0163:         return -p_log_p.sum(-1)
0164: 
````

- **L144** EN: Defines function `sample`, which produces samples according to the modeled stochastic behavior. | CN: 定义函数 `sample`，其作用是根据建模的随机行为生成样本。
- **L145** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L146** EN: Assigns or updates `sample_shape`. | CN: 对 `sample_shape` 进行赋值或更新。
- **L147** EN: Assigns or updates `probs_2d`. | CN: 对 `probs_2d` 进行赋值或更新。
- **L148** EN: Assigns or updates `samples_2d`. | CN: 对 `samples_2d` 进行赋值或更新。
- **L149** EN: Returns from `Categorical.sample` with the computed result or updated state. | CN: 从 `Categorical.sample` 返回计算结果或更新后的状态。
- **L150** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L151** EN: Defines function `log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 定义函数 `log_prob`，其作用是为给定输入计算对数概率或类似密度分数。
- **L152** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L153** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L154** EN: Assigns or updates `value`. | CN: 对 `value` 进行赋值或更新。
- **L155** EN: Invokes `torch.broadcast_tensors` to advance the surrounding implementation. | CN: 调用 `torch.broadcast_tensors` 来推进周围的实现逻辑。
- **L156** EN: Assigns or updates `value`. | CN: 对 `value` 进行赋值或更新。
- **L157** EN: Returns from `Categorical.log_prob` with the computed result or updated state. | CN: 从 `Categorical.log_prob` 返回计算结果或更新后的状态。
- **L158** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L159** EN: Defines function `entropy`, which computes an entropy-style summary of uncertainty. | CN: 定义函数 `entropy`，其作用是计算描述不确定性的熵类指标。
- **L160** EN: Assigns or updates `min_real`. | CN: 对 `min_real` 进行赋值或更新。
- **L161** EN: Assigns or updates `logits`. | CN: 对 `logits` 进行赋值或更新。
- **L162** EN: Assigns or updates `p_log_p`. | CN: 对 `p_log_p` 进行赋值或更新。
- **L163** EN: Returns from `Categorical.entropy` with the computed result or updated state. | CN: 从 `Categorical.entropy` 返回计算结果或更新后的状态。
- **L164** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 165-171 / 第 165-171 行

````python
0165:     def enumerate_support(self, expand=True):
0166:         num_events = self._num_events
0167:         values = torch.arange(num_events, dtype=torch.long, device=self._param.device)
0168:         values = values.view((-1,) + (1,) * len(self._batch_shape))
0169:         if expand:
0170:             values = values.expand((-1,) + self._batch_shape)
0171:         return values
````

- **L165** EN: Defines function `enumerate_support`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `enumerate_support`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L166** EN: Assigns or updates `num_events`. | CN: 对 `num_events` 进行赋值或更新。
- **L167** EN: Assigns or updates `values`. | CN: 对 `values` 进行赋值或更新。
- **L168** EN: Assigns or updates `values`. | CN: 对 `values` 进行赋值或更新。
- **L169** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L170** EN: Assigns or updates `values`. | CN: 对 `values` 进行赋值或更新。
- **L171** EN: Returns from `Categorical.enumerate_support` with the computed result or updated state. | CN: 从 `Categorical.enumerate_support` 返回计算结果或更新后的状态。

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

- **Torch imports / Torch 导入**: `torch`、`torch:nan, Tensor`、`torch.distributions:constraints`、`torch.distributions.distribution:Distribution`、`torch.distributions.utils:lazy_property, logits_to_probs, probs_to_logits`
- **Other imports / 其他导入**: 无
- **Top-level classes / 顶层类**: `Categorical`
- **Top-level functions / 顶层函数**: 无
- **Base classes / 基类**: `Distribution`
- **Decorators / 装饰器**: 无
- **Module assignments / 模块级赋值**: `__all__`
