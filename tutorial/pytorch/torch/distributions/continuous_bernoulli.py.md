# continuous_bernoulli.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/distributions/continuous_bernoulli.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements the `ContinuousBernoulli` distribution or helper type, including parameter handling, sampling, and statistical methods.
- **Purpose (CN)**: 实现 `ContinuousBernoulli` 分布或辅助类型，涵盖参数处理、采样与统计方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26 / 第 1-26 行

````python
0001: # mypy: allow-untyped-defs
0002: import math
0003: 
0004: import torch
0005: from torch import Tensor
0006: from torch.distributions import constraints
0007: from torch.distributions.exp_family import ExponentialFamily
0008: from torch.distributions.utils import (
0009:     broadcast_all,
0010:     clamp_probs,
0011:     lazy_property,
0012:     logits_to_probs,
0013:     probs_to_logits,
0014: )
0015: from torch.nn.functional import binary_cross_entropy_with_logits
0016: from torch.types import _Number, _size, Number
0017: 
0018: 
0019: __all__ = ["ContinuousBernoulli"]
0020: 
0021: 
0022: class ContinuousBernoulli(ExponentialFamily):
0023:     r"""
0024:     Creates a continuous Bernoulli distribution parameterized by :attr:`probs`
0025:     or :attr:`logits` (but not both).
0026: 
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Imports module dependencies: `math`. | CN: 导入模块依赖：`math`。
- **L3** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L4** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L5** EN: Imports `Tensor` from `torch` so later code can reuse those definitions. | CN: 从 `torch` 导入 `Tensor`，供后续代码复用这些定义。
- **L6** EN: Imports `constraints` from `torch.distributions` so later code can reuse those definitions. | CN: 从 `torch.distributions` 导入 `constraints`，供后续代码复用这些定义。
- **L7** EN: Imports `ExponentialFamily` from `torch.distributions.exp_family` so later code can reuse those definitions. | CN: 从 `torch.distributions.exp_family` 导入 `ExponentialFamily`，供后续代码复用这些定义。
- **L8** EN: Starts a multi-line import from `torch.distributions.utils` so several helpers can be listed clearly. | CN: 开始一个来自 `torch.distributions.utils` 的多行导入，以便清晰列出多个辅助符号。
- **L9** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L10** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L11** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L12** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L13** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L14** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L15** EN: Imports `binary_cross_entropy_with_logits` from `torch.nn.functional` so later code can reuse those definitions. | CN: 从 `torch.nn.functional` 导入 `binary_cross_entropy_with_logits`，供后续代码复用这些定义。
- **L16** EN: Imports `_Number, _size, Number` from `torch.types` so later code can reuse those definitions. | CN: 从 `torch.types` 导入 `_Number, _size, Number`，供后续代码复用这些定义。
- **L17** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L18** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L19** EN: Defines or updates the module’s public export list. | CN: 定义或更新模块的公开导出列表。
- **L20** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L21** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L22** EN: Defines class `ContinuousBernoulli` with bases `ExponentialFamily`, which encapsulates distribution parameters, validation rules, and tensor-valued statistical behavior. | CN: 定义类 `ContinuousBernoulli`，其基类为 `ExponentialFamily`，作用是封装分布参数、校验规则以及基于张量的统计行为。
- **L23** EN: Starts the docstring for class `ContinuousBernoulli`. | CN: 开始为 class `ContinuousBernoulli` 编写文档字符串。
- **L24** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L25** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L26** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 27-54 / 第 27-54 行

````python
0027:     The distribution is supported in [0, 1] and parameterized by 'probs' (in
0028:     (0,1)) or 'logits' (real-valued). Note that, unlike the Bernoulli, 'probs'
0029:     does not correspond to a probability and 'logits' does not correspond to
0030:     log-odds, but the same names are used due to the similarity with the
0031:     Bernoulli. See [1] for more details.
0032: 
0033:     Example::
0034: 
0035:         >>> # xdoctest: +IGNORE_WANT("non-deterministic")
0036:         >>> m = ContinuousBernoulli(torch.tensor([0.3]))
0037:         >>> m.sample()
0038:         tensor([ 0.2538])
0039: 
0040:     Args:
0041:         probs (Number, Tensor): (0,1) valued parameters
0042:         logits (Number, Tensor): real valued parameters whose sigmoid matches 'probs'
0043: 
0044:     [1] The continuous Bernoulli: fixing a pervasive error in variational
0045:     autoencoders, Loaiza-Ganem G and Cunningham JP, NeurIPS 2019.
0046:     https://arxiv.org/abs/1907.06845
0047:     """
0048: 
0049:     # pyrefly: ignore [bad-override]
0050:     arg_constraints = {"probs": constraints.unit_interval, "logits": constraints.real}
0051:     support = constraints.unit_interval
0052:     _mean_carrier_measure = 0
0053:     has_rsample = True
0054: 
````

- **L27** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L28** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L29** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L30** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L31** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L32** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L33** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L34** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L35** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L36** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L37** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L38** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L39** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L40** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L41** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L42** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L43** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L44** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L45** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L46** EN: Continues the docstring for class `ContinuousBernoulli`. | CN: 继续补充 class `ContinuousBernoulli` 的文档字符串。
- **L47** EN: Ends the docstring for class `ContinuousBernoulli`. | CN: 结束 class `ContinuousBernoulli` 的文档字符串。
- **L48** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L49** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L50** EN: Sets structural metadata on `arg_constraints` that other routines consult later. | CN: 为 `arg_constraints` 设置结构化元数据，供后续例程查询。
- **L51** EN: Sets structural metadata on `support` that other routines consult later. | CN: 为 `support` 设置结构化元数据，供后续例程查询。
- **L52** EN: Assigns module-level configuration or cached state to `_mean_carrier_measure`. | CN: 为 `_mean_carrier_measure` 赋予模块级配置或缓存状态。
- **L53** EN: Assigns or updates `has_rsample`. | CN: 对 `has_rsample` 进行赋值或更新。
- **L54** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 55-82 / 第 55-82 行

````python
0055:     def __init__(
0056:         self,
0057:         probs: Tensor | Number | None = None,
0058:         logits: Tensor | Number | None = None,
0059:         lims: tuple[float, float] = (0.499, 0.501),
0060:         validate_args: bool | None = None,
0061:     ) -> None:
0062:         if (probs is None) == (logits is None):
0063:             raise ValueError(
0064:                 "Either `probs` or `logits` must be specified, but not both."
0065:             )
0066:         if probs is not None:
0067:             is_scalar = isinstance(probs, _Number)
0068:             # pyrefly: ignore [read-only]
0069:             (self.probs,) = broadcast_all(probs)
0070:             # validate 'probs' here if necessary as it is later clamped for numerical stability
0071:             # close to 0 and 1, later on; otherwise the clamped 'probs' would always pass
0072:             if validate_args is not None:
0073:                 if not self.arg_constraints["probs"].check(self.probs).all():
0074:                     raise ValueError("The parameter probs has invalid values")
0075:             # pyrefly: ignore [read-only]
0076:             self.probs = clamp_probs(self.probs)
0077:         else:
0078:             if logits is None:
0079:                 raise AssertionError("logits is unexpectedly None")
0080:             is_scalar = isinstance(logits, _Number)
0081:             # pyrefly: ignore [read-only]
0082:             (self.logits,) = broadcast_all(logits)
````

- **L55** EN: Defines function `__init__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__init__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L56** EN: Continues `ContinuousBernoulli.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L57** EN: Continues `ContinuousBernoulli.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L58** EN: Continues `ContinuousBernoulli.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L59** EN: Continues `ContinuousBernoulli.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L60** EN: Continues `ContinuousBernoulli.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L61** EN: Continues `ContinuousBernoulli.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L62** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L63** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L64** EN: Continues `ContinuousBernoulli.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L65** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L66** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L67** EN: Assigns or updates `is_scalar`. | CN: 对 `is_scalar` 进行赋值或更新。
- **L68** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L69** EN: Invokes `broadcast_all` to advance the surrounding implementation. | CN: 调用 `broadcast_all` 来推进周围的实现逻辑。
- **L70** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L71** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L72** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L73** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L74** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L75** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L76** EN: Updates object state via `self.probs`. | CN: 通过 `self.probs` 更新对象状态。
- **L77** EN: Introduces the fallback branch when earlier conditions are not met. | CN: 引入前置条件不满足时的后备分支。
- **L78** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L79** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L80** EN: Assigns or updates `is_scalar`. | CN: 对 `is_scalar` 进行赋值或更新。
- **L81** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L82** EN: Invokes `broadcast_all` to advance the surrounding implementation. | CN: 调用 `broadcast_all` 来推进周围的实现逻辑。

### Lines 83-107 / 第 83-107 行

````python
0083:         self._param = self.probs if probs is not None else self.logits
0084:         if is_scalar:
0085:             batch_shape = torch.Size()
0086:         else:
0087:             batch_shape = self._param.size()
0088:         self._lims = lims
0089:         super().__init__(batch_shape, validate_args=validate_args)
0090: 
0091:     def expand(self, batch_shape, _instance=None):
0092:         new = self._get_checked_instance(ContinuousBernoulli, _instance)
0093:         new._lims = self._lims
0094:         batch_shape = torch.Size(batch_shape)
0095:         if "probs" in self.__dict__:
0096:             new.probs = self.probs.expand(batch_shape)
0097:             new._param = new.probs
0098:         if "logits" in self.__dict__:
0099:             new.logits = self.logits.expand(batch_shape)
0100:             new._param = new.logits
0101:         super(ContinuousBernoulli, new).__init__(batch_shape, validate_args=False)
0102:         new._validate_args = self._validate_args
0103:         return new
0104: 
0105:     def _new(self, *args, **kwargs):
0106:         return self._param.new(*args, **kwargs)
0107: 
````

- **L83** EN: Updates object state via `self._param`. | CN: 通过 `self._param` 更新对象状态。
- **L84** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L85** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L86** EN: Introduces the fallback branch when earlier conditions are not met. | CN: 引入前置条件不满足时的后备分支。
- **L87** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L88** EN: Updates object state via `self._lims`. | CN: 通过 `self._lims` 更新对象状态。
- **L89** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L90** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L91** EN: Defines function `expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 定义函数 `expand`，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L92** EN: Assigns or updates `new`. | CN: 对 `new` 进行赋值或更新。
- **L93** EN: Updates object state via `new._lims`. | CN: 通过 `new._lims` 更新对象状态。
- **L94** EN: Assigns or updates `batch_shape`. | CN: 对 `batch_shape` 进行赋值或更新。
- **L95** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L96** EN: Updates object state via `new.probs`. | CN: 通过 `new.probs` 更新对象状态。
- **L97** EN: Updates object state via `new._param`. | CN: 通过 `new._param` 更新对象状态。
- **L98** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L99** EN: Updates object state via `new.logits`. | CN: 通过 `new.logits` 更新对象状态。
- **L100** EN: Updates object state via `new._param`. | CN: 通过 `new._param` 更新对象状态。
- **L101** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L102** EN: Updates object state via `new._validate_args`. | CN: 通过 `new._validate_args` 更新对象状态。
- **L103** EN: Returns from `ContinuousBernoulli.expand` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.expand` 返回计算结果或更新后的状态。
- **L104** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L105** EN: Defines function `_new`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_new`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L106** EN: Returns from `ContinuousBernoulli._new` with the computed result or updated state. | CN: 从 `ContinuousBernoulli._new` 返回计算结果或更新后的状态。
- **L107** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 108-135 / 第 108-135 行

````python
0108:     def _outside_unstable_region(self):
0109:         return torch.max(
0110:             torch.le(self.probs, self._lims[0]), torch.gt(self.probs, self._lims[1])
0111:         )
0112: 
0113:     def _cut_probs(self):
0114:         return torch.where(
0115:             self._outside_unstable_region(),
0116:             self.probs,
0117:             self._lims[0] * torch.ones_like(self.probs),
0118:         )
0119: 
0120:     def _cont_bern_log_norm(self):
0121:         """computes the log normalizing constant as a function of the 'probs' parameter"""
0122:         cut_probs = self._cut_probs()
0123:         cut_probs_below_half = torch.where(
0124:             torch.le(cut_probs, 0.5), cut_probs, torch.zeros_like(cut_probs)
0125:         )
0126:         cut_probs_above_half = torch.where(
0127:             torch.ge(cut_probs, 0.5), cut_probs, torch.ones_like(cut_probs)
0128:         )
0129:         log_norm = torch.log(
0130:             torch.abs(torch.log1p(-cut_probs) - torch.log(cut_probs))
0131:         ) - torch.where(
0132:             torch.le(cut_probs, 0.5),
0133:             torch.log1p(-2.0 * cut_probs_below_half),
0134:             torch.log(2.0 * cut_probs_above_half - 1.0),
0135:         )
````

- **L108** EN: Defines function `_outside_unstable_region`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_outside_unstable_region`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L109** EN: Returns from `ContinuousBernoulli._outside_unstable_region` with the computed result or updated state. | CN: 从 `ContinuousBernoulli._outside_unstable_region` 返回计算结果或更新后的状态。
- **L110** EN: Invokes `torch.le` to advance the surrounding implementation. | CN: 调用 `torch.le` 来推进周围的实现逻辑。
- **L111** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L112** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L113** EN: Defines function `_cut_probs`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_cut_probs`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L114** EN: Returns from `ContinuousBernoulli._cut_probs` with the computed result or updated state. | CN: 从 `ContinuousBernoulli._cut_probs` 返回计算结果或更新后的状态。
- **L115** EN: Invokes `self._outside_unstable_region` to advance the surrounding implementation. | CN: 调用 `self._outside_unstable_region` 来推进周围的实现逻辑。
- **L116** EN: Continues `ContinuousBernoulli._cut_probs`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli._cut_probs` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L117** EN: Invokes `torch.ones_like` to advance the surrounding implementation. | CN: 调用 `torch.ones_like` 来推进周围的实现逻辑。
- **L118** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L119** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L120** EN: Defines function `_cont_bern_log_norm`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_cont_bern_log_norm`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L121** EN: Provides a one-line docstring for function `ContinuousBernoulli._cont_bern_log_norm`. | CN: 为 function `ContinuousBernoulli._cont_bern_log_norm` 提供单行文档字符串。
- **L122** EN: Assigns or updates `cut_probs`. | CN: 对 `cut_probs` 进行赋值或更新。
- **L123** EN: Assigns or updates `cut_probs_below_half`. | CN: 对 `cut_probs_below_half` 进行赋值或更新。
- **L124** EN: Invokes `torch.le` to advance the surrounding implementation. | CN: 调用 `torch.le` 来推进周围的实现逻辑。
- **L125** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L126** EN: Assigns or updates `cut_probs_above_half`. | CN: 对 `cut_probs_above_half` 进行赋值或更新。
- **L127** EN: Invokes `torch.ge` to advance the surrounding implementation. | CN: 调用 `torch.ge` 来推进周围的实现逻辑。
- **L128** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L129** EN: Assigns or updates `log_norm`. | CN: 对 `log_norm` 进行赋值或更新。
- **L130** EN: Invokes `torch.abs` to advance the surrounding implementation. | CN: 调用 `torch.abs` 来推进周围的实现逻辑。
- **L131** EN: Invokes `torch.where` to advance the surrounding implementation. | CN: 调用 `torch.where` 来推进周围的实现逻辑。
- **L132** EN: Invokes `torch.le` to advance the surrounding implementation. | CN: 调用 `torch.le` 来推进周围的实现逻辑。
- **L133** EN: Invokes `torch.log1p` to advance the surrounding implementation. | CN: 调用 `torch.log1p` 来推进周围的实现逻辑。
- **L134** EN: Invokes `torch.log` to advance the surrounding implementation. | CN: 调用 `torch.log` 来推进周围的实现逻辑。
- **L135** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。

### Lines 136-163 / 第 136-163 行

````python
0136:         x = torch.pow(self.probs - 0.5, 2)
0137:         taylor = math.log(2.0) + (4.0 / 3.0 + 104.0 / 45.0 * x) * x
0138:         return torch.where(self._outside_unstable_region(), log_norm, taylor)
0139: 
0140:     @property
0141:     def mean(self) -> Tensor:
0142:         cut_probs = self._cut_probs()
0143:         mus = cut_probs / (2.0 * cut_probs - 1.0) + 1.0 / (
0144:             torch.log1p(-cut_probs) - torch.log(cut_probs)
0145:         )
0146:         x = self.probs - 0.5
0147:         taylor = 0.5 + (1.0 / 3.0 + 16.0 / 45.0 * torch.pow(x, 2)) * x
0148:         return torch.where(self._outside_unstable_region(), mus, taylor)
0149: 
0150:     @property
0151:     def stddev(self) -> Tensor:
0152:         return torch.sqrt(self.variance)
0153: 
0154:     @property
0155:     def variance(self) -> Tensor:
0156:         cut_probs = self._cut_probs()
0157:         vars = cut_probs * (cut_probs - 1.0) / torch.pow(
0158:             1.0 - 2.0 * cut_probs, 2
0159:         ) + 1.0 / torch.pow(torch.log1p(-cut_probs) - torch.log(cut_probs), 2)
0160:         x = torch.pow(self.probs - 0.5, 2)
0161:         taylor = 1.0 / 12.0 - (1.0 / 15.0 - 128.0 / 945.0 * x) * x
0162:         return torch.where(self._outside_unstable_region(), vars, taylor)
0163: 
````

- **L136** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L137** EN: Assigns or updates `taylor`. | CN: 对 `taylor` 进行赋值或更新。
- **L138** EN: Returns from `ContinuousBernoulli._cont_bern_log_norm` with the computed result or updated state. | CN: 从 `ContinuousBernoulli._cont_bern_log_norm` 返回计算结果或更新后的状态。
- **L139** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L140** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L141** EN: Defines function `mean`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mean`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L142** EN: Assigns or updates `cut_probs`. | CN: 对 `cut_probs` 进行赋值或更新。
- **L143** EN: Assigns or updates `mus`. | CN: 对 `mus` 进行赋值或更新。
- **L144** EN: Invokes `torch.log1p` to advance the surrounding implementation. | CN: 调用 `torch.log1p` 来推进周围的实现逻辑。
- **L145** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L146** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L147** EN: Assigns or updates `taylor`. | CN: 对 `taylor` 进行赋值或更新。
- **L148** EN: Returns from `ContinuousBernoulli.mean` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.mean` 返回计算结果或更新后的状态。
- **L149** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L150** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L151** EN: Defines function `stddev`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `stddev`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L152** EN: Returns from `ContinuousBernoulli.stddev` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.stddev` 返回计算结果或更新后的状态。
- **L153** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L154** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L155** EN: Defines function `variance`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `variance`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L156** EN: Assigns or updates `cut_probs`. | CN: 对 `cut_probs` 进行赋值或更新。
- **L157** EN: Assigns or updates `vars`. | CN: 对 `vars` 进行赋值或更新。
- **L158** EN: Continues `ContinuousBernoulli.variance`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.variance` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L159** EN: Invokes `torch.pow` to advance the surrounding implementation. | CN: 调用 `torch.pow` 来推进周围的实现逻辑。
- **L160** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L161** EN: Assigns or updates `taylor`. | CN: 对 `taylor` 进行赋值或更新。
- **L162** EN: Returns from `ContinuousBernoulli.variance` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.variance` 返回计算结果或更新后的状态。
- **L163** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 164-186 / 第 164-186 行

````python
0164:     @lazy_property
0165:     def logits(self) -> Tensor:
0166:         return probs_to_logits(self.probs, is_binary=True)
0167: 
0168:     @lazy_property
0169:     def probs(self) -> Tensor:
0170:         return clamp_probs(logits_to_probs(self.logits, is_binary=True))
0171: 
0172:     @property
0173:     def param_shape(self) -> torch.Size:
0174:         return self._param.size()
0175: 
0176:     def sample(self, sample_shape=torch.Size()):
0177:         shape = self._extended_shape(sample_shape)
0178:         u = torch.rand(shape, dtype=self.probs.dtype, device=self.probs.device)
0179:         with torch.no_grad():
0180:             return self.icdf(u)
0181: 
0182:     def rsample(self, sample_shape: _size = torch.Size()) -> Tensor:
0183:         shape = self._extended_shape(sample_shape)
0184:         u = torch.rand(shape, dtype=self.probs.dtype, device=self.probs.device)
0185:         return self.icdf(u)
0186: 
````

- **L164** EN: Applies decorator `lazy_property`, which turns a derived value into a lazily cached attribute. | CN: 应用装饰器 `lazy_property`，其作用是把派生值变成惰性缓存属性。
- **L165** EN: Defines function `logits`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `logits`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L166** EN: Returns from `ContinuousBernoulli.logits` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.logits` 返回计算结果或更新后的状态。
- **L167** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L168** EN: Applies decorator `lazy_property`, which turns a derived value into a lazily cached attribute. | CN: 应用装饰器 `lazy_property`，其作用是把派生值变成惰性缓存属性。
- **L169** EN: Defines function `probs`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `probs`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L170** EN: Returns from `ContinuousBernoulli.probs` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.probs` 返回计算结果或更新后的状态。
- **L171** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L172** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L173** EN: Defines function `param_shape`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `param_shape`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L174** EN: Returns from `ContinuousBernoulli.param_shape` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.param_shape` 返回计算结果或更新后的状态。
- **L175** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L176** EN: Defines function `sample`, which produces samples according to the modeled stochastic behavior. | CN: 定义函数 `sample`，其作用是根据建模的随机行为生成样本。
- **L177** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L178** EN: Assigns or updates `u`. | CN: 对 `u` 进行赋值或更新。
- **L179** EN: Enters a managed context so resources or flags are scoped safely. | CN: 进入受管理的上下文，使资源或标志拥有安全的作用域。
- **L180** EN: Returns from `ContinuousBernoulli.sample` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.sample` 返回计算结果或更新后的状态。
- **L181** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L182** EN: Defines function `rsample`, which produces samples according to the modeled stochastic behavior. | CN: 定义函数 `rsample`，其作用是根据建模的随机行为生成样本。
- **L183** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L184** EN: Assigns or updates `u`. | CN: 对 `u` 进行赋值或更新。
- **L185** EN: Returns from `ContinuousBernoulli.rsample` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.rsample` 返回计算结果或更新后的状态。
- **L186** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 187-211 / 第 187-211 行

````python
0187:     def log_prob(self, value):
0188:         if self._validate_args:
0189:             self._validate_sample(value)
0190:         logits, value = broadcast_all(self.logits, value)
0191:         return (
0192:             -binary_cross_entropy_with_logits(logits, value, reduction="none")
0193:             + self._cont_bern_log_norm()
0194:         )
0195: 
0196:     def cdf(self, value):
0197:         if self._validate_args:
0198:             self._validate_sample(value)
0199:         cut_probs = self._cut_probs()
0200:         cdfs = (
0201:             torch.pow(cut_probs, value) * torch.pow(1.0 - cut_probs, 1.0 - value)
0202:             + cut_probs
0203:             - 1.0
0204:         ) / (2.0 * cut_probs - 1.0)
0205:         unbounded_cdfs = torch.where(self._outside_unstable_region(), cdfs, value)
0206:         return torch.where(
0207:             torch.le(value, 0.0),
0208:             torch.zeros_like(value),
0209:             torch.where(torch.ge(value, 1.0), torch.ones_like(value), unbounded_cdfs),
0210:         )
0211: 
````

- **L187** EN: Defines function `log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 定义函数 `log_prob`，其作用是为给定输入计算对数概率或类似密度分数。
- **L188** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L189** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L190** EN: Invokes `broadcast_all` to advance the surrounding implementation. | CN: 调用 `broadcast_all` 来推进周围的实现逻辑。
- **L191** EN: Returns from `ContinuousBernoulli.log_prob` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.log_prob` 返回计算结果或更新后的状态。
- **L192** EN: Invokes `binary_cross_entropy_with_logits` to advance the surrounding implementation. | CN: 调用 `binary_cross_entropy_with_logits` 来推进周围的实现逻辑。
- **L193** EN: Invokes `self._cont_bern_log_norm` to advance the surrounding implementation. | CN: 调用 `self._cont_bern_log_norm` 来推进周围的实现逻辑。
- **L194** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L195** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L196** EN: Defines function `cdf`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `cdf`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L197** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L198** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L199** EN: Assigns or updates `cut_probs`. | CN: 对 `cut_probs` 进行赋值或更新。
- **L200** EN: Assigns or updates `cdfs`. | CN: 对 `cdfs` 进行赋值或更新。
- **L201** EN: Invokes `torch.pow` to advance the surrounding implementation. | CN: 调用 `torch.pow` 来推进周围的实现逻辑。
- **L202** EN: Continues `ContinuousBernoulli.cdf`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.cdf` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L203** EN: Continues `ContinuousBernoulli.cdf`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.cdf` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L204** EN: Continues `ContinuousBernoulli.cdf`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.cdf` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L205** EN: Assigns or updates `unbounded_cdfs`. | CN: 对 `unbounded_cdfs` 进行赋值或更新。
- **L206** EN: Returns from `ContinuousBernoulli.cdf` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.cdf` 返回计算结果或更新后的状态。
- **L207** EN: Invokes `torch.le` to advance the surrounding implementation. | CN: 调用 `torch.le` 来推进周围的实现逻辑。
- **L208** EN: Invokes `torch.zeros_like` to advance the surrounding implementation. | CN: 调用 `torch.zeros_like` 来推进周围的实现逻辑。
- **L209** EN: Invokes `torch.where` to advance the surrounding implementation. | CN: 调用 `torch.where` 来推进周围的实现逻辑。
- **L210** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L211** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 212-236 / 第 212-236 行

````python
0212:     def icdf(self, value):
0213:         cut_probs = self._cut_probs()
0214:         return torch.where(
0215:             self._outside_unstable_region(),
0216:             (
0217:                 torch.log1p(-cut_probs + value * (2.0 * cut_probs - 1.0))
0218:                 - torch.log1p(-cut_probs)
0219:             )
0220:             / (torch.log(cut_probs) - torch.log1p(-cut_probs)),
0221:             value,
0222:         )
0223: 
0224:     def entropy(self):
0225:         log_probs0 = torch.log1p(-self.probs)
0226:         log_probs1 = torch.log(self.probs)
0227:         return (
0228:             self.mean * (log_probs0 - log_probs1)
0229:             - self._cont_bern_log_norm()
0230:             - log_probs0
0231:         )
0232: 
0233:     @property
0234:     def _natural_params(self) -> tuple[Tensor]:
0235:         return (self.logits,)
0236: 
````

- **L212** EN: Defines function `icdf`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `icdf`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L213** EN: Assigns or updates `cut_probs`. | CN: 对 `cut_probs` 进行赋值或更新。
- **L214** EN: Returns from `ContinuousBernoulli.icdf` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.icdf` 返回计算结果或更新后的状态。
- **L215** EN: Invokes `self._outside_unstable_region` to advance the surrounding implementation. | CN: 调用 `self._outside_unstable_region` 来推进周围的实现逻辑。
- **L216** EN: Continues `ContinuousBernoulli.icdf`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.icdf` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L217** EN: Invokes `torch.log1p` to advance the surrounding implementation. | CN: 调用 `torch.log1p` 来推进周围的实现逻辑。
- **L218** EN: Invokes `torch.log1p` to advance the surrounding implementation. | CN: 调用 `torch.log1p` 来推进周围的实现逻辑。
- **L219** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L220** EN: Invokes `torch.log` to advance the surrounding implementation. | CN: 调用 `torch.log` 来推进周围的实现逻辑。
- **L221** EN: Continues `ContinuousBernoulli.icdf`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `ContinuousBernoulli.icdf` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L222** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L223** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L224** EN: Defines function `entropy`, which computes an entropy-style summary of uncertainty. | CN: 定义函数 `entropy`，其作用是计算描述不确定性的熵类指标。
- **L225** EN: Assigns or updates `log_probs0`. | CN: 对 `log_probs0` 进行赋值或更新。
- **L226** EN: Assigns or updates `log_probs1`. | CN: 对 `log_probs1` 进行赋值或更新。
- **L227** EN: Returns from `ContinuousBernoulli.entropy` with the computed result or updated state. | CN: 从 `ContinuousBernoulli.entropy` 返回计算结果或更新后的状态。
- **L228** EN: Continues `ContinuousBernoulli.entropy`, which computes an entropy-style summary of uncertainty. | CN: 继续 `ContinuousBernoulli.entropy` 的实现，其作用是计算描述不确定性的熵类指标。
- **L229** EN: Invokes `self._cont_bern_log_norm` to advance the surrounding implementation. | CN: 调用 `self._cont_bern_log_norm` 来推进周围的实现逻辑。
- **L230** EN: Continues `ContinuousBernoulli.entropy`, which computes an entropy-style summary of uncertainty. | CN: 继续 `ContinuousBernoulli.entropy` 的实现，其作用是计算描述不确定性的熵类指标。
- **L231** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L232** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L233** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L234** EN: Defines function `_natural_params`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_natural_params`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L235** EN: Returns from `ContinuousBernoulli._natural_params` with the computed result or updated state. | CN: 从 `ContinuousBernoulli._natural_params` 返回计算结果或更新后的状态。
- **L236** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 237-250 / 第 237-250 行

````python
0237:     # pyrefly: ignore [bad-override]
0238:     def _log_normalizer(self, x):
0239:         """computes the log normalizing constant as a function of the natural parameter"""
0240:         out_unst_reg = torch.max(
0241:             torch.le(x, self._lims[0] - 0.5), torch.gt(x, self._lims[1] - 0.5)
0242:         )
0243:         cut_nat_params = torch.where(
0244:             out_unst_reg, x, (self._lims[0] - 0.5) * torch.ones_like(x)
0245:         )
0246:         log_norm = torch.log(
0247:             torch.abs(torch.special.expm1(cut_nat_params))
0248:         ) - torch.log(torch.abs(cut_nat_params))
0249:         taylor = 0.5 * x + torch.pow(x, 2) / 24.0 - torch.pow(x, 4) / 2880.0
0250:         return torch.where(out_unst_reg, log_norm, taylor)
````

- **L237** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L238** EN: Defines function `_log_normalizer`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `_log_normalizer`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L239** EN: Provides a one-line docstring for function `ContinuousBernoulli._log_normalizer`. | CN: 为 function `ContinuousBernoulli._log_normalizer` 提供单行文档字符串。
- **L240** EN: Assigns or updates `out_unst_reg`. | CN: 对 `out_unst_reg` 进行赋值或更新。
- **L241** EN: Invokes `torch.le` to advance the surrounding implementation. | CN: 调用 `torch.le` 来推进周围的实现逻辑。
- **L242** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L243** EN: Assigns or updates `cut_nat_params`. | CN: 对 `cut_nat_params` 进行赋值或更新。
- **L244** EN: Invokes `torch.ones_like` to advance the surrounding implementation. | CN: 调用 `torch.ones_like` 来推进周围的实现逻辑。
- **L245** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L246** EN: Assigns or updates `log_norm`. | CN: 对 `log_norm` 进行赋值或更新。
- **L247** EN: Invokes `torch.abs` to advance the surrounding implementation. | CN: 调用 `torch.abs` 来推进周围的实现逻辑。
- **L248** EN: Invokes `torch.log` to advance the surrounding implementation. | CN: 调用 `torch.log` 来推进周围的实现逻辑。
- **L249** EN: Assigns or updates `taylor`. | CN: 对 `taylor` 进行赋值或更新。
- **L250** EN: Returns from `ContinuousBernoulli._log_normalizer` with the computed result or updated state. | CN: 从 `ContinuousBernoulli._log_normalizer` 返回计算结果或更新后的状态。

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

- **Torch imports / Torch 导入**: `torch`、`torch:Tensor`、`torch.distributions:constraints`、`torch.distributions.exp_family:ExponentialFamily`、`torch.distributions.utils:broadcast_all, clamp_probs, lazy_property, logits_to_probs, probs_to_logits`、`torch.nn.functional:binary_cross_entropy_with_logits`、`torch.types:_Number, _size, Number`
- **Other imports / 其他导入**: `math`
- **Top-level classes / 顶层类**: `ContinuousBernoulli`
- **Top-level functions / 顶层函数**: 无
- **Base classes / 基类**: `ExponentialFamily`
- **Decorators / 装饰器**: 无
- **Module assignments / 模块级赋值**: `__all__`
