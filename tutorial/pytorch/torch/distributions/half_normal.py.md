# half_normal.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/distributions/half_normal.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements the `HalfNormal` distribution or helper type, including parameter handling, sampling, and statistical methods.
- **Purpose (CN)**: 实现 `HalfNormal` 分布或辅助类型，涵盖参数处理、采样与统计方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21 / 第 1-21 行

````python
0001: # mypy: allow-untyped-defs
0002: import math
0003: 
0004: import torch
0005: from torch import inf, Tensor
0006: from torch.distributions import constraints
0007: from torch.distributions.normal import Normal
0008: from torch.distributions.transformed_distribution import TransformedDistribution
0009: from torch.distributions.transforms import AbsTransform
0010: 
0011: 
0012: __all__ = ["HalfNormal"]
0013: 
0014: 
0015: class HalfNormal(TransformedDistribution):
0016:     r"""
0017:     Creates a half-normal distribution parameterized by `scale` where::
0018: 
0019:         X ~ Normal(0, scale)
0020:         Y = |X| ~ HalfNormal(scale)
0021: 
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Imports module dependencies: `math`. | CN: 导入模块依赖：`math`。
- **L3** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L4** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L5** EN: Imports `inf, Tensor` from `torch` so later code can reuse those definitions. | CN: 从 `torch` 导入 `inf, Tensor`，供后续代码复用这些定义。
- **L6** EN: Imports `constraints` from `torch.distributions` so later code can reuse those definitions. | CN: 从 `torch.distributions` 导入 `constraints`，供后续代码复用这些定义。
- **L7** EN: Imports `Normal` from `torch.distributions.normal` so later code can reuse those definitions. | CN: 从 `torch.distributions.normal` 导入 `Normal`，供后续代码复用这些定义。
- **L8** EN: Imports `TransformedDistribution` from `torch.distributions.transformed_distribution` so later code can reuse those definitions. | CN: 从 `torch.distributions.transformed_distribution` 导入 `TransformedDistribution`，供后续代码复用这些定义。
- **L9** EN: Imports `AbsTransform` from `torch.distributions.transforms` so later code can reuse those definitions. | CN: 从 `torch.distributions.transforms` 导入 `AbsTransform`，供后续代码复用这些定义。
- **L10** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L11** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L12** EN: Defines or updates the module’s public export list. | CN: 定义或更新模块的公开导出列表。
- **L13** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L14** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L15** EN: Defines class `HalfNormal` with bases `TransformedDistribution`, which encapsulates distribution parameters, validation rules, and tensor-valued statistical behavior. | CN: 定义类 `HalfNormal`，其基类为 `TransformedDistribution`，作用是封装分布参数、校验规则以及基于张量的统计行为。
- **L16** EN: Starts the docstring for class `HalfNormal`. | CN: 开始为 class `HalfNormal` 编写文档字符串。
- **L17** EN: Continues the docstring for class `HalfNormal`. | CN: 继续补充 class `HalfNormal` 的文档字符串。
- **L18** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L19** EN: Continues the docstring for class `HalfNormal`. | CN: 继续补充 class `HalfNormal` 的文档字符串。
- **L20** EN: Continues the docstring for class `HalfNormal`. | CN: 继续补充 class `HalfNormal` 的文档字符串。
- **L21** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 22-39 / 第 22-39 行

````python
0022:     Example::
0023: 
0024:         >>> # xdoctest: +IGNORE_WANT("non-deterministic")
0025:         >>> m = HalfNormal(torch.tensor([1.0]))
0026:         >>> m.sample()  # half-normal distributed with scale=1
0027:         tensor([ 0.1046])
0028: 
0029:     Args:
0030:         scale (float or Tensor): scale of the full Normal distribution
0031:     """
0032: 
0033:     arg_constraints = {"scale": constraints.positive}
0034:     # pyrefly: ignore [bad-override]
0035:     support = constraints.nonnegative
0036:     has_rsample = True
0037:     # pyrefly: ignore [bad-override]
0038:     base_dist: Normal
0039: 
````

- **L22** EN: Continues the docstring for class `HalfNormal`. | CN: 继续补充 class `HalfNormal` 的文档字符串。
- **L23** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L24** EN: Continues the docstring for class `HalfNormal`. | CN: 继续补充 class `HalfNormal` 的文档字符串。
- **L25** EN: Continues the docstring for class `HalfNormal`. | CN: 继续补充 class `HalfNormal` 的文档字符串。
- **L26** EN: Continues the docstring for class `HalfNormal`. | CN: 继续补充 class `HalfNormal` 的文档字符串。
- **L27** EN: Continues the docstring for class `HalfNormal`. | CN: 继续补充 class `HalfNormal` 的文档字符串。
- **L28** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L29** EN: Continues the docstring for class `HalfNormal`. | CN: 继续补充 class `HalfNormal` 的文档字符串。
- **L30** EN: Continues the docstring for class `HalfNormal`. | CN: 继续补充 class `HalfNormal` 的文档字符串。
- **L31** EN: Ends the docstring for class `HalfNormal`. | CN: 结束 class `HalfNormal` 的文档字符串。
- **L32** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L33** EN: Sets structural metadata on `arg_constraints` that other routines consult later. | CN: 为 `arg_constraints` 设置结构化元数据，供后续例程查询。
- **L34** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L35** EN: Sets structural metadata on `support` that other routines consult later. | CN: 为 `support` 设置结构化元数据，供后续例程查询。
- **L36** EN: Assigns or updates `has_rsample`. | CN: 对 `has_rsample` 进行赋值或更新。
- **L37** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L38** EN: Continues class `HalfNormal`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `HalfNormal` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L39** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 40-59 / 第 40-59 行

````python
0040:     def __init__(
0041:         self,
0042:         scale: Tensor | float,
0043:         validate_args: bool | None = None,
0044:     ) -> None:
0045:         base_dist = Normal(0, scale, validate_args=False)
0046:         super().__init__(base_dist, AbsTransform(), validate_args=validate_args)
0047: 
0048:     def expand(self, batch_shape, _instance=None):
0049:         new = self._get_checked_instance(HalfNormal, _instance)
0050:         return super().expand(batch_shape, _instance=new)
0051: 
0052:     @property
0053:     def scale(self) -> Tensor:
0054:         return self.base_dist.scale
0055: 
0056:     @property
0057:     def mean(self) -> Tensor:
0058:         return self.scale * math.sqrt(2 / math.pi)
0059: 
````

- **L40** EN: Defines function `__init__`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `__init__`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L41** EN: Continues `HalfNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `HalfNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L42** EN: Continues `HalfNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `HalfNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L43** EN: Continues `HalfNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `HalfNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L44** EN: Continues `HalfNormal.__init__`, which implements part of the distribution API or its numerical helpers. | CN: 继续 `HalfNormal.__init__` 的实现，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L45** EN: Assigns or updates `base_dist`. | CN: 对 `base_dist` 进行赋值或更新。
- **L46** EN: Invokes `super` to advance the surrounding implementation. | CN: 调用 `super` 来推进周围的实现逻辑。
- **L47** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L48** EN: Defines function `expand`, which adapts stored tensors or metadata to a requested batch shape. | CN: 定义函数 `expand`，其作用是把保存的张量或元数据调整到指定的 batch 形状。
- **L49** EN: Assigns or updates `new`. | CN: 对 `new` 进行赋值或更新。
- **L50** EN: Returns from `HalfNormal.expand` with the computed result or updated state. | CN: 从 `HalfNormal.expand` 返回计算结果或更新后的状态。
- **L51** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L52** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L53** EN: Defines function `scale`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `scale`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L54** EN: Returns from `HalfNormal.scale` with the computed result or updated state. | CN: 从 `HalfNormal.scale` 返回计算结果或更新后的状态。
- **L55** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L56** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L57** EN: Defines function `mean`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mean`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L58** EN: Returns from `HalfNormal.mean` with the computed result or updated state. | CN: 从 `HalfNormal.mean` 返回计算结果或更新后的状态。
- **L59** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 60-79 / 第 60-79 行

````python
0060:     @property
0061:     def mode(self) -> Tensor:
0062:         return torch.zeros_like(self.scale)
0063: 
0064:     @property
0065:     def variance(self) -> Tensor:
0066:         return self.scale.pow(2) * (1 - 2 / math.pi)
0067: 
0068:     def log_prob(self, value):
0069:         if self._validate_args:
0070:             self._validate_sample(value)
0071:         log_prob = self.base_dist.log_prob(value) + math.log(2)
0072:         log_prob = torch.where(value >= 0, log_prob, -inf)
0073:         return log_prob
0074: 
0075:     def cdf(self, value):
0076:         if self._validate_args:
0077:             self._validate_sample(value)
0078:         return 2 * self.base_dist.cdf(value) - 1
0079: 
````

- **L60** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L61** EN: Defines function `mode`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `mode`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L62** EN: Returns from `HalfNormal.mode` with the computed result or updated state. | CN: 从 `HalfNormal.mode` 返回计算结果或更新后的状态。
- **L63** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L64** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L65** EN: Defines function `variance`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `variance`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L66** EN: Returns from `HalfNormal.variance` with the computed result or updated state. | CN: 从 `HalfNormal.variance` 返回计算结果或更新后的状态。
- **L67** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L68** EN: Defines function `log_prob`, which computes a log-probability or density-like score for supplied inputs. | CN: 定义函数 `log_prob`，其作用是为给定输入计算对数概率或类似密度分数。
- **L69** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L70** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L71** EN: Assigns or updates `log_prob`. | CN: 对 `log_prob` 进行赋值或更新。
- **L72** EN: Assigns or updates `log_prob`. | CN: 对 `log_prob` 进行赋值或更新。
- **L73** EN: Returns from `HalfNormal.log_prob` with the computed result or updated state. | CN: 从 `HalfNormal.log_prob` 返回计算结果或更新后的状态。
- **L74** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L75** EN: Defines function `cdf`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `cdf`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L76** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L77** EN: Invokes `self._validate_sample` to advance the surrounding implementation. | CN: 调用 `self._validate_sample` 来推进周围的实现逻辑。
- **L78** EN: Returns from `HalfNormal.cdf` with the computed result or updated state. | CN: 从 `HalfNormal.cdf` 返回计算结果或更新后的状态。
- **L79** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 80-84 / 第 80-84 行

````python
0080:     def icdf(self, prob):
0081:         return self.base_dist.icdf((prob + 1) / 2)
0082: 
0083:     def entropy(self):
0084:         return self.base_dist.entropy() - math.log(2)
````

- **L80** EN: Defines function `icdf`, which implements part of the distribution API or its numerical helpers. | CN: 定义函数 `icdf`，其作用是实现分布 API 或其数值辅助逻辑的一部分。
- **L81** EN: Returns from `HalfNormal.icdf` with the computed result or updated state. | CN: 从 `HalfNormal.icdf` 返回计算结果或更新后的状态。
- **L82** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L83** EN: Defines function `entropy`, which computes an entropy-style summary of uncertainty. | CN: 定义函数 `entropy`，其作用是计算描述不确定性的熵类指标。
- **L84** EN: Returns from `HalfNormal.entropy` with the computed result or updated state. | CN: 从 `HalfNormal.entropy` 返回计算结果或更新后的状态。

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

- **Torch imports / Torch 导入**: `torch`、`torch:inf, Tensor`、`torch.distributions:constraints`、`torch.distributions.normal:Normal`、`torch.distributions.transformed_distribution:TransformedDistribution`、`torch.distributions.transforms:AbsTransform`
- **Other imports / 其他导入**: `math`
- **Top-level classes / 顶层类**: `HalfNormal`
- **Top-level functions / 顶层函数**: 无
- **Base classes / 基类**: `TransformedDistribution`
- **Decorators / 装饰器**: 无
- **Module assignments / 模块级赋值**: `__all__`
