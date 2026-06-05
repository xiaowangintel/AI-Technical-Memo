# __init__.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/special/__init__.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Initializes the package namespace, re-exports symbols, and wires lazy imports or feature flags.
- **Purpose (CN)**: 初始化包命名空间，重新导出符号，并连接懒加载导入或特性开关。
## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行
````python
import torch
from torch._C import _add_docstr, _special  # type: ignore[attr-defined]
from torch._torch_docs import common_args, multi_dim_common


__all__ = [
    "airy_ai",
    "bessel_j0",
    "bessel_j1",
    "bessel_y0",
    "bessel_y1",
    "chebyshev_polynomial_t",
    "chebyshev_polynomial_u",
    "chebyshev_polynomial_v",
    "chebyshev_polynomial_w",
    "digamma",
    "entr",
    "erf",
    "erfc",
    "erfcx",
    "erfinv",
    "exp2",
    "expit",
    "expm1",
    "gammainc",
    "gammaincc",
    "gammaln",
    "hermite_polynomial_h",
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._C, torch._torch_docs. `__all__` defines the public symbols that this module chooses to export.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._C、torch._torch_docs。 `__all__` 定义了本模块选择导出的公共符号。

### Lines 29-56 / 第 29-56 行
````python
    "hermite_polynomial_he",
    "i0",
    "i0e",
    "i1",
    "i1e",
    "laguerre_polynomial_l",
    "legendre_polynomial_p",
    "log1p",
    "log_ndtr",
    "log_softmax",
    "logit",
    "logsumexp",
    "modified_bessel_i0",
    "modified_bessel_i1",
    "modified_bessel_k0",
    "modified_bessel_k1",
    "multigammaln",
    "ndtr",
    "ndtri",
    "polygamma",
    "psi",
    "round",
    "shifted_chebyshev_polynomial_t",
    "shifted_chebyshev_polynomial_u",
    "shifted_chebyshev_polynomial_v",
    "shifted_chebyshev_polynomial_w",
    "scaled_modified_bessel_k0",
    "scaled_modified_bessel_k1",
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 57-82 / 第 57-82 行
````python
    "sinc",
    "softmax",
    "spherical_bessel_j0",
    "xlog1py",
    "xlogy",
    "zeta",
]

Tensor = torch.Tensor

entr = _add_docstr(
    _special.special_entr,
    r"""
entr(input, *, out=None) -> Tensor
Computes the entropy on :attr:`input` (as defined below), elementwise.

.. math::
    \begin{align}
    \text{entr(x)} = \begin{cases}
        -x * \ln(x)  & x > 0 \\
        0 &  x = 0.0 \\
        -\infty & x < 0
    \end{cases}
    \end{align}
"""
    + """
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 84-107 / 第 84-107 行
````python
Args:
   input (Tensor): the input tensor.

Keyword args:
    out (Tensor, optional): the output tensor.

Example::

    >>> a = torch.arange(-0.5, 1, 0.5)
    >>> a
    tensor([-0.5000,  0.0000,  0.5000])
    >>> torch.special.entr(a)
    tensor([  -inf, 0.0000, 0.3466])
""",
)

psi = _add_docstr(
    _special.special_psi,
    r"""
psi(input, *, out=None) -> Tensor

Alias for :func:`torch.special.digamma`.
""",
)
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 109-135 / 第 109-135 行
````python
digamma = _add_docstr(
    _special.special_digamma,
    r"""
digamma(input, *, out=None) -> Tensor

Computes the logarithmic derivative of the gamma function on `input`.

.. math::
    \digamma(x) = \frac{d}{dx} \ln\left(\Gamma\left(x\right)\right) = \frac{\Gamma'(x)}{\Gamma(x)}
"""
    + r"""
Args:
    input (Tensor): the tensor to compute the digamma function on

Keyword args:
    {out}

.. note::  This function is similar to SciPy's `scipy.special.digamma`.

.. note::  From PyTorch 1.8 onwards, the digamma function returns `-Inf` for `0`.
           Previously it returned `NaN` for `0`.

Example::

    >>> a = torch.tensor([1, 0.5])
    >>> torch.special.digamma(a)
    tensor([-0.5772, -1.9635])
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 137-161 / 第 137-161 行
````python
""".format(**common_args),
)

gammaln = _add_docstr(
    _special.special_gammaln,
    r"""
gammaln(input, *, out=None) -> Tensor

Computes the natural logarithm of the absolute value of the gamma function on :attr:`input`.

.. math::
    \text{out}_{i} = \ln \Gamma(|\text{input}_{i}|)
"""
    + """
Args:
    {input}

Keyword args:
    {out}

Example::

    >>> a = torch.arange(0.5, 2, 0.5)
    >>> torch.special.gammaln(a)
    tensor([ 0.5724,  0.0000, -0.1208])
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 163-188 / 第 163-188 行
````python
""".format(**common_args),
)

polygamma = _add_docstr(
    _special.special_polygamma,
    r"""
polygamma(n, input, *, out=None) -> Tensor

Computes the :math:`n^{th}` derivative of the digamma function on :attr:`input`.
:math:`n \geq 0` is called the order of the polygamma function.

.. math::
    \psi^{(n)}(x) = \frac{d^{(n)}}{dx^{(n)}} \psi(x)

.. note::
    This function is implemented only for nonnegative integers :math:`n \geq 0`.
"""
    + """
Args:
    n (int): the order of the polygamma function
    {input}

Keyword args:
    {out}

Example::
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 190-214 / 第 190-214 行
````python
    >>> a = torch.tensor([1, 0.5])
    >>> torch.special.polygamma(1, a)
    tensor([1.64493, 4.9348])
    >>> torch.special.polygamma(2, a)
    tensor([ -2.4041, -16.8288])
    >>> torch.special.polygamma(3, a)
    tensor([ 6.4939, 97.4091])
    >>> torch.special.polygamma(4, a)
    tensor([ -24.8863, -771.4742])
""".format(**common_args),
)

erf = _add_docstr(
    _special.special_erf,
    r"""
erf(input, *, out=None) -> Tensor

Computes the error function of :attr:`input`. The error function is defined as follows:

.. math::
    \mathrm{erf}(x) = \frac{2}{\sqrt{\pi}} \int_{0}^{x} e^{-t^2} dt
"""
    + r"""
Args:
    {input}
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 216-242 / 第 216-242 行
````python
Keyword args:
    {out}

Example::

    >>> torch.special.erf(torch.tensor([0, -1., 10.]))
    tensor([ 0.0000, -0.8427,  1.0000])
""".format(**common_args),
)

erfc = _add_docstr(
    _special.special_erfc,
    r"""
erfc(input, *, out=None) -> Tensor

Computes the complementary error function of :attr:`input`.
The complementary error function is defined as follows:

.. math::
    \mathrm{erfc}(x) = 1 - \frac{2}{\sqrt{\pi}} \int_{0}^{x} e^{-t^2} dt
"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 244-270 / 第 244-270 行
````python
Example::

    >>> torch.special.erfc(torch.tensor([0, -1., 10.]))
    tensor([ 1.0000, 1.8427,  0.0000])
""".format(**common_args),
)

erfcx = _add_docstr(
    _special.special_erfcx,
    r"""
erfcx(input, *, out=None) -> Tensor

Computes the scaled complementary error function for each element of :attr:`input`.
The scaled complementary error function is defined as follows:

.. math::
    \mathrm{erfcx}(x) = e^{x^2} \mathrm{erfc}(x)
"""
    + r"""

"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 272-298 / 第 272-298 行
````python
Example::

    >>> torch.special.erfcx(torch.tensor([0, -1., 10.]))
    tensor([ 1.0000, 5.0090, 0.0561])
""".format(**common_args),
)

erfinv = _add_docstr(
    _special.special_erfinv,
    r"""
erfinv(input, *, out=None) -> Tensor

Computes the inverse error function of :attr:`input`.
The inverse error function is defined in the range :math:`(-1, 1)` as:

.. math::
    \mathrm{erfinv}(\mathrm{erf}(x)) = x
"""
    + r"""

Args:
    {input}

Keyword args:
    {out}

Example::
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 300-327 / 第 300-327 行
````python
    >>> torch.special.erfinv(torch.tensor([0, 0.5, -1.]))
    tensor([ 0.0000,  0.4769,    -inf])
""".format(**common_args),
)

logit = _add_docstr(
    _special.special_logit,
    r"""
logit(input, eps=None, *, out=None) -> Tensor

Returns a new tensor with the logit of the elements of :attr:`input`.
:attr:`input` is clamped to [eps, 1 - eps] when eps is not None.
When eps is None and :attr:`input` < 0 or :attr:`input` > 1, the function will yields NaN.

.. math::
    \begin{align}
    y_{i} &= \ln(\frac{z_{i}}{1 - z_{i}}) \\
    z_{i} &= \begin{cases}
        x_{i} & \text{if eps is None} \\
        \text{eps} & \text{if } x_{i} < \text{eps} \\
        x_{i} & \text{if } \text{eps} \leq x_{i} \leq 1 - \text{eps} \\
        1 - \text{eps} & \text{if } x_{i} > 1 - \text{eps}
    \end{cases}
    \end{align}
"""
    + r"""
Args:
    {input}
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 328-350 / 第 328-350 行
````python
    eps (float, optional): the epsilon for input clamp bound. Default: ``None``

Keyword args:
    {out}

Example::

    >>> a = torch.rand(5)
    >>> a
    tensor([0.2796, 0.9331, 0.6486, 0.1523, 0.6516])
    >>> torch.special.logit(a, eps=1e-6)
    tensor([-0.9466,  2.6352,  0.6131, -1.7169,  0.6261])
""".format(**common_args),
)

logsumexp = _add_docstr(
    _special.special_logsumexp,
    r"""
logsumexp(input, dim, keepdim=False, *, out=None)

Alias for :func:`torch.logsumexp`.
""".format(**multi_dim_common),
)
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 352-377 / 第 352-377 行
````python
expit = _add_docstr(
    _special.special_expit,
    r"""
expit(input, *, out=None) -> Tensor

Computes the expit (also known as the logistic sigmoid function) of the elements of :attr:`input`.

.. math::
    \text{out}_{i} = \frac{1}{1 + e^{-\text{input}_{i}}}
"""
    + r"""
Args:
    {input}

Keyword args:
    {out}

Example::

    >>> t = torch.randn(4)
    >>> t
    tensor([ 0.9213,  1.0887, -0.8858, -1.7683])
    >>> torch.special.expit(t)
    tensor([ 0.7153,  0.7481,  0.2920,  0.1458])
""".format(**common_args),
)
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 379-402 / 第 379-402 行
````python
exp2 = _add_docstr(
    _special.special_exp2,
    r"""
exp2(input, *, out=None) -> Tensor

Computes the base two exponential function of :attr:`input`.

.. math::
    y_{i} = 2^{x_{i}}

"""
    + r"""
Args:
    {input}

Keyword args:
    {out}

Example::

    >>> torch.special.exp2(torch.tensor([0, math.log2(2.), 3, 4]))
    tensor([ 1.,  2.,  8., 16.])
""".format(**common_args),
)
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 404-430 / 第 404-430 行
````python
expm1 = _add_docstr(
    _special.special_expm1,
    r"""
expm1(input, *, out=None) -> Tensor

Computes the exponential of the elements minus 1
of :attr:`input`.

.. math::
    y_{i} = e^{x_{i}} - 1

.. note:: This function provides greater precision than exp(x) - 1 for small values of x.

"""
    + r"""
Args:
    {input}

Keyword args:
    {out}

Example::

    >>> torch.special.expm1(torch.tensor([0, math.log(2.)]))
    tensor([ 0.,  1.])
""".format(**common_args),
)
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 432-458 / 第 432-458 行
````python
xlog1py = _add_docstr(
    _special.special_xlog1py,
    r"""
xlog1py(input, other, *, out=None) -> Tensor

Computes ``input * log1p(other)`` with the following cases.

.. math::
    \text{out}_{i} = \begin{cases}
        \text{NaN} & \text{if } \text{other}_{i} = \text{NaN} \\
        0 & \text{if } \text{input}_{i} = 0.0 \text{ and } \text{other}_{i} != \text{NaN} \\
        \text{input}_{i} * \text{log1p}(\text{other}_{i})& \text{otherwise}
    \end{cases}

Similar to SciPy's `scipy.special.xlog1py`.

"""
    + r"""

Args:
    input (Number or Tensor) : Multiplier
    other (Number or Tensor) : Argument

.. note:: At least one of :attr:`input` or :attr:`other` must be a tensor.

Keyword args:
    {out}
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 460-482 / 第 460-482 行
````python
Example::

    >>> x = torch.zeros(5,)
    >>> y = torch.tensor([-1, 0, 1, float('inf'), float('nan')])
    >>> torch.special.xlog1py(x, y)
    tensor([0., 0., 0., 0., nan])
    >>> x = torch.tensor([1, 2, 3])
    >>> y = torch.tensor([3, 2, 1])
    >>> torch.special.xlog1py(x, y)
    tensor([1.3863, 2.1972, 2.0794])
    >>> torch.special.xlog1py(x, 4)
    tensor([1.6094, 3.2189, 4.8283])
    >>> torch.special.xlog1py(2, y)
    tensor([2.7726, 2.1972, 1.3863])
""".format(**common_args),
)

xlogy = _add_docstr(
    _special.special_xlogy,
    r"""
xlogy(input, other, *, out=None) -> Tensor

Computes ``input * log(other)`` with the following cases.
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 484-505 / 第 484-505 行
````python
.. math::
    \text{out}_{i} = \begin{cases}
        \text{NaN} & \text{if } \text{other}_{i} = \text{NaN} \\
        0 & \text{if } \text{input}_{i} = 0.0 \\
        \text{input}_{i} * \log{(\text{other}_{i})} & \text{otherwise}
    \end{cases}

Similar to SciPy's `scipy.special.xlogy`.

"""
    + r"""

Args:
    input (Number or Tensor) : Multiplier
    other (Number or Tensor) : Argument

.. note:: At least one of :attr:`input` or :attr:`other` must be a tensor.

Keyword args:
    {out}

Example::
````
- **EN**: Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 507-530 / 第 507-530 行
````python
    >>> x = torch.zeros(5,)
    >>> y = torch.tensor([-1, 0, 1, float('inf'), float('nan')])
    >>> torch.special.xlogy(x, y)
    tensor([0., 0., 0., 0., nan])
    >>> x = torch.tensor([1, 2, 3])
    >>> y = torch.tensor([3, 2, 1])
    >>> torch.special.xlogy(x, y)
    tensor([1.0986, 1.3863, 0.0000])
    >>> torch.special.xlogy(x, 4)
    tensor([1.3863, 2.7726, 4.1589])
    >>> torch.special.xlogy(2, y)
    tensor([2.1972, 1.3863, 0.0000])
""".format(**common_args),
)

i0 = _add_docstr(
    _special.special_i0,
    r"""
i0(input, *, out=None) -> Tensor

Computes the zeroth order modified Bessel function of the first kind for each element of :attr:`input`.

.. math::
    \text{out}_{i} = I_0(\text{input}_{i}) = \sum_{k=0}^{\infty} \frac{(\text{input}_{i}^2/4)^k}{(k!)^2}
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 532-556 / 第 532-556 行
````python
"""
    + r"""
Args:
    input (Tensor): the input tensor

Keyword args:
    {out}

Example::

    >>> torch.i0(torch.arange(5, dtype=torch.float32))
    tensor([ 1.0000,  1.2661,  2.2796,  4.8808, 11.3019])

""".format(**common_args),
)

i0e = _add_docstr(
    _special.special_i0e,
    r"""
i0e(input, *, out=None) -> Tensor
Computes the exponentially scaled zeroth order modified Bessel function of the first kind (as defined below)
for each element of :attr:`input`.

.. math::
    \text{out}_{i} = \exp(-|x|) * i0(x) = \exp(-|x|) * \sum_{k=0}^{\infty} \frac{(\text{input}_{i}^2/4)^k}{(k!)^2}
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 558-581 / 第 558-581 行
````python
"""
    + r"""
Args:
    {input}

Keyword args:
    {out}

Example::

    >>> torch.special.i0e(torch.arange(5, dtype=torch.float32))
    tensor([1.0000, 0.4658, 0.3085, 0.2430, 0.2070])
""".format(**common_args),
)

i1 = _add_docstr(
    _special.special_i1,
    r"""
i1(input, *, out=None) -> Tensor
Computes the first order modified Bessel function of the first kind (as defined below)
for each element of :attr:`input`.

.. math::
    \text{out}_{i} = \frac{(\text{input}_{i})}{2} * \sum_{k=0}^{\infty} \frac{(\text{input}_{i}^2/4)^k}{(k!) * (k+1)!}
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 583-607 / 第 583-607 行
````python
"""
    + r"""
Args:
    {input}

Keyword args:
    {out}

Example::

    >>> torch.special.i1(torch.arange(5, dtype=torch.float32))
    tensor([0.0000, 0.5652, 1.5906, 3.9534, 9.7595])
""".format(**common_args),
)

i1e = _add_docstr(
    _special.special_i1e,
    r"""
i1e(input, *, out=None) -> Tensor
Computes the exponentially scaled first order modified Bessel function of the first kind (as defined below)
for each element of :attr:`input`.

.. math::
    \text{out}_{i} = \exp(-|x|) * i1(x) =
        \exp(-|x|) * \frac{(\text{input}_{i})}{2} * \sum_{k=0}^{\infty} \frac{(\text{input}_{i}^2/4)^k}{(k!) * (k+1)!}
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 609-632 / 第 609-632 行
````python
"""
    + r"""
Args:
    {input}

Keyword args:
    {out}

Example::

    >>> torch.special.i1e(torch.arange(5, dtype=torch.float32))
    tensor([0.0000, 0.2079, 0.2153, 0.1968, 0.1788])
""".format(**common_args),
)

ndtr = _add_docstr(
    _special.special_ndtr,
    r"""
ndtr(input, *, out=None) -> Tensor
Computes the area under the standard Gaussian probability density function,
integrated from minus infinity to :attr:`input`, elementwise.

.. math::
    \text{ndtr}(x) = \frac{1}{\sqrt{2 \pi}}\int_{-\infty}^{x} e^{-\frac{1}{2}t^2} dt
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 634-660 / 第 634-660 行
````python
"""
    + r"""
Args:
    {input}

Keyword args:
    {out}

Example::

    >>> torch.special.ndtr(torch.tensor([-3., -2, -1, 0, 1, 2, 3]))
    tensor([0.0013, 0.0228, 0.1587, 0.5000, 0.8413, 0.9772, 0.9987])
""".format(**common_args),
)

ndtri = _add_docstr(
    _special.special_ndtri,
    r"""
ndtri(input, *, out=None) -> Tensor
Computes the argument, x, for which the area under the Gaussian probability density function
(integrated from minus infinity to x) is equal to :attr:`input`, elementwise.

.. math::
    \text{ndtri}(p) = \sqrt{2}\text{erf}^{-1}(2p - 1)

.. note::
    Also known as quantile function for Normal Distribution.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 662-685 / 第 662-685 行
````python
"""
    + r"""
Args:
    {input}

Keyword args:
    {out}

Example::

    >>> torch.special.ndtri(torch.tensor([0, 0.25, 0.5, 0.75, 1]))
    tensor([   -inf, -0.6745,  0.0000,  0.6745,     inf])
""".format(**common_args),
)

log_ndtr = _add_docstr(
    _special.special_log_ndtr,
    r"""
log_ndtr(input, *, out=None) -> Tensor
Computes the log of the area under the standard Gaussian probability density function,
integrated from minus infinity to :attr:`input`, elementwise.

.. math::
    \text{log\_ndtr}(x) = \log\left(\frac{1}{\sqrt{2 \pi}}\int_{-\infty}^{x} e^{-\frac{1}{2}t^2} dt \right)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 687-709 / 第 687-709 行
````python
"""
    + r"""
Args:
    {input}

Keyword args:
    {out}

Example::

    >>> torch.special.log_ndtr(torch.tensor([-3., -2, -1, 0, 1, 2, 3]))
    tensor([-6.6077 -3.7832 -1.841  -0.6931 -0.1728 -0.023  -0.0014])
""".format(**common_args),
)

log1p = _add_docstr(
    _special.special_log1p,
    r"""
log1p(input, *, out=None) -> Tensor

Alias for :func:`torch.log1p`.
""",
)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 711-733 / 第 711-733 行
````python
sinc = _add_docstr(
    _special.special_sinc,
    r"""
sinc(input, *, out=None) -> Tensor

Computes the normalized sinc of :attr:`input.`

.. math::
    \text{out}_{i} =
    \begin{cases}
      1, & \text{if}\ \text{input}_{i}=0 \\
      \sin(\pi \text{input}_{i}) / (\pi \text{input}_{i}), & \text{otherwise}
    \end{cases}
"""
    + r"""

Args:
    {input}

Keyword args:
    {out}

Example::
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 735-761 / 第 735-761 行
````python
    >>> t = torch.randn(4)
    >>> t
    tensor([ 0.2252, -0.2948,  1.0267, -1.1566])
    >>> torch.special.sinc(t)
    tensor([ 0.9186,  0.8631, -0.0259, -0.1300])
""".format(**common_args),
)

round = _add_docstr(
    _special.special_round,
    r"""
round(input, *, out=None) -> Tensor

Alias for :func:`torch.round`.
""",
)

softmax = _add_docstr(
    _special.special_softmax,
    r"""
softmax(input, dim, *, dtype=None) -> Tensor

Computes the softmax function.

Softmax is defined as:

:math:`\text{Softmax}(x_{i}) = \frac{\exp(x_i)}{\sum_j \exp(x_j)}`
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 763-787 / 第 763-787 行
````python
It is applied to all slices along dim, and will re-scale them so that the elements
lie in the range `[0, 1]` and sum to 1.

Args:
    input (Tensor): input
    dim (int): A dimension along which softmax will be computed.
    dtype (:class:`torch.dtype`, optional): the desired data type of returned tensor.
        If specified, the input tensor is cast to :attr:`dtype` before the operation
        is performed. This is useful for preventing data type overflows. Default: None.

Examples::
    >>> t = torch.ones(2, 2)
    >>> torch.special.softmax(t, 0)
    tensor([[0.5000, 0.5000],
            [0.5000, 0.5000]])

""",
)

log_softmax = _add_docstr(
    _special.special_log_softmax,
    r"""
log_softmax(input, dim, *, dtype=None) -> Tensor

Computes softmax followed by a logarithm.
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 789-812 / 第 789-812 行
````python
While mathematically equivalent to log(softmax(x)), doing these two
operations separately is slower and numerically unstable. This function
is computed as:

.. math::
    \text{log\_softmax}(x_{i}) = \log\left(\frac{\exp(x_i) }{ \sum_j \exp(x_j)} \right)
"""
    + r"""

Args:
    input (Tensor): input
    dim (int): A dimension along which log_softmax will be computed.
    dtype (:class:`torch.dtype`, optional): the desired data type of returned tensor.
        If specified, the input tensor is cast to :attr:`dtype` before the operation
        is performed. This is useful for preventing data type overflows. Default: None.

Example::

    >>> t = torch.ones(2, 2)
    >>> torch.special.log_softmax(t, 0)
    tensor([[-0.6931, -0.6931],
            [-0.6931, -0.6931]])
""",
)
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 814-836 / 第 814-836 行
````python
zeta = _add_docstr(
    _special.special_zeta,
    r"""
zeta(input, other, *, out=None) -> Tensor

Computes the Hurwitz zeta function, elementwise.

.. math::
    \zeta(x, q) = \sum_{k=0}^{\infty} \frac{1}{(k + q)^x}

"""
    + r"""
Args:
    input (Tensor): the input tensor corresponding to `x`.
    other (Tensor): the input tensor corresponding to `q`.

.. note::
    The Riemann zeta function corresponds to the case when `q = 1`

Keyword args:
    {out}

Example::
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 838-864 / 第 838-864 行
````python
    >>> x = torch.tensor([2., 4.])
    >>> torch.special.zeta(x, 1)
    tensor([1.6449, 1.0823])
    >>> torch.special.zeta(x, torch.tensor([1., 2.]))
    tensor([1.6449, 0.0823])
    >>> torch.special.zeta(2, torch.tensor([1., 2.]))
    tensor([1.6449, 0.6449])
""".format(**common_args),
)

multigammaln = _add_docstr(
    _special.special_multigammaln,
    r"""
multigammaln(input, p, *, out=None) -> Tensor

Computes the `multivariate log-gamma function
<https://en.wikipedia.org/wiki/Multivariate_gamma_function>`_ with dimension
:math:`p` element-wise, given by

.. math::
    \log(\Gamma_{p}(a)) = C + \displaystyle \sum_{i=1}^{p} \log\left(\Gamma\left(a - \frac{i - 1}{2}\right)\right)

where :math:`C = \log(\pi) \cdot \frac{p (p - 1)}{4}` and :math:`\Gamma(-)` is the Gamma function.

All elements must be greater than :math:`\frac{p - 1}{2}`, otherwise the behavior is undefined.
"""
    + """
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 866-890 / 第 866-890 行
````python
Args:
    input (Tensor): the tensor to compute the multivariate log-gamma function
    p (int): the number of dimensions

Keyword args:
    {out}

Example::

    >>> a = torch.empty(2, 3).uniform_(1, 2)
    >>> a
    tensor([[1.6835, 1.8474, 1.1929],
            [1.0475, 1.7162, 1.4180]])
    >>> torch.special.multigammaln(a, 2)
    tensor([[0.3928, 0.4007, 0.7586],
            [1.0311, 0.3901, 0.5049]])
""".format(**common_args),
)

gammainc = _add_docstr(
    _special.special_gammainc,
    r"""
gammainc(input, other, *, out=None) -> Tensor

Computes the regularized lower incomplete gamma function:
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 892-916 / 第 892-916 行
````python
.. math::
    \text{out}_{i} = \frac{1}{\Gamma(\text{input}_i)} \int_0^{\text{other}_i} t^{\text{input}_i-1} e^{-t} dt

where both :math:`\text{input}_i` and :math:`\text{other}_i` are weakly positive
and at least one is strictly positive.
If both are zero or either is negative then :math:`\text{out}_i=\text{nan}`.
:math:`\Gamma(\cdot)` in the equation above is the gamma function,

.. math::
    \Gamma(\text{input}_i) = \int_0^\infty t^{(\text{input}_i-1)} e^{-t} dt.

See :func:`torch.special.gammaincc` and :func:`torch.special.gammaln` for related functions.

Supports :ref:`broadcasting to a common shape <broadcasting-semantics>`
and float inputs.

.. note::
    The backward pass with respect to :attr:`input` is not yet supported.
    Please open an issue on PyTorch's Github to request it.

"""
    + r"""
Args:
    input (Tensor): the first non-negative input tensor
    other (Tensor): the second non-negative input tensor
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 918-942 / 第 918-942 行
````python
Keyword args:
    {out}

Example::

    >>> a1 = torch.tensor([4.0])
    >>> a2 = torch.tensor([3.0, 4.0, 5.0])
    >>> a = torch.special.gammaincc(a1, a2)
    tensor([0.3528, 0.5665, 0.7350])
    tensor([0.3528, 0.5665, 0.7350])
    >>> b = torch.special.gammainc(a1, a2) + torch.special.gammaincc(a1, a2)
    tensor([1., 1., 1.])

""".format(**common_args),
)

gammaincc = _add_docstr(
    _special.special_gammaincc,
    r"""
gammaincc(input, other, *, out=None) -> Tensor

Computes the regularized upper incomplete gamma function:

.. math::
    \text{out}_{i} = \frac{1}{\Gamma(\text{input}_i)} \int_{\text{other}_i}^{\infty} t^{\text{input}_i-1} e^{-t} dt
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 944-970 / 第 944-970 行
````python
where both :math:`\text{input}_i` and :math:`\text{other}_i` are weakly positive
and at least one is strictly positive.
If both are zero or either is negative then :math:`\text{out}_i=\text{nan}`.
:math:`\Gamma(\cdot)` in the equation above is the gamma function,

.. math::
    \Gamma(\text{input}_i) = \int_0^\infty t^{(\text{input}_i-1)} e^{-t} dt.

See :func:`torch.special.gammainc` and :func:`torch.special.gammaln` for related functions.

Supports :ref:`broadcasting to a common shape <broadcasting-semantics>`
and float inputs.

.. note::
    The backward pass with respect to :attr:`input` is not yet supported.
    Please open an issue on PyTorch's Github to request it.

"""
    + r"""
Args:
    input (Tensor): the first non-negative input tensor
    other (Tensor): the second non-negative input tensor

Keyword args:
    {out}

Example::
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 972-997 / 第 972-997 行
````python
    >>> a1 = torch.tensor([4.0])
    >>> a2 = torch.tensor([3.0, 4.0, 5.0])
    >>> a = torch.special.gammaincc(a1, a2)
    tensor([0.6472, 0.4335, 0.2650])
    >>> b = torch.special.gammainc(a1, a2) + torch.special.gammaincc(a1, a2)
    tensor([1., 1., 1.])

""".format(**common_args),
)

airy_ai = _add_docstr(
    _special.special_airy_ai,
    r"""
airy_ai(input, *, out=None) -> Tensor

Airy function :math:`\text{Ai}\left(\text{input}\right)`.

"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
""".format(**common_args),
)
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 999-1021 / 第 999-1021 行
````python
bessel_j0 = _add_docstr(
    _special.special_bessel_j0,
    r"""
bessel_j0(input, *, out=None) -> Tensor

Bessel function of the first kind of order :math:`0`.

"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
""".format(**common_args),
)

bessel_j1 = _add_docstr(
    _special.special_bessel_j1,
    r"""
bessel_j1(input, *, out=None) -> Tensor

Bessel function of the first kind of order :math:`1`.
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1023-1048 / 第 1023-1048 行
````python
"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
""".format(**common_args),
)

bessel_y0 = _add_docstr(
    _special.special_bessel_y0,
    r"""
bessel_y0(input, *, out=None) -> Tensor

Bessel function of the second kind of order :math:`0`.

"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
""".format(**common_args),
)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 1050-1075 / 第 1050-1075 行
````python
bessel_y1 = _add_docstr(
    _special.special_bessel_y1,
    r"""
bessel_y1(input, *, out=None) -> Tensor

Bessel function of the second kind of order :math:`1`.

"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
""".format(**common_args),
)

chebyshev_polynomial_t = _add_docstr(
    _special.special_chebyshev_polynomial_t,
    r"""
chebyshev_polynomial_t(input, n, *, out=None) -> Tensor

Chebyshev polynomial of the first kind :math:`T_{n}(\text{input})`.

If :math:`n = 0`, :math:`1` is returned. If :math:`n = 1`, :math:`\text{input}`
is returned. If :math:`n < 6` or :math:`|\text{input}| > 1` the recursion:
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1077-1103 / 第 1077-1103 行
````python
.. math::
    T_{n + 1}(\text{input}) = 2 \times \text{input} \times T_{n}(\text{input}) - T_{n - 1}(\text{input})

is evaluated. Otherwise, the explicit trigonometric formula:

.. math::
    T_{n}(\text{input}) = \text{cos}(n \times \text{arccos}(x))

is evaluated.

"""
    + r"""
Args:
    {input}
    n (Tensor): Degree of the polynomial.

Keyword args:
    {out}
""".format(**common_args),
)

chebyshev_polynomial_u = _add_docstr(
    _special.special_chebyshev_polynomial_u,
    r"""
chebyshev_polynomial_u(input, n, *, out=None) -> Tensor

Chebyshev polynomial of the second kind :math:`U_{n}(\text{input})`.
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1105-1128 / 第 1105-1128 行
````python
If :math:`n = 0`, :math:`1` is returned. If :math:`n = 1`,
:math:`2 \times \text{input}` is returned. If :math:`n < 6` or
:math:`|\text{input}| > 1`, the recursion:

.. math::
    U_{n + 1}(\text{input}) = 2 \times \text{input} \times U_{n}(\text{input}) - U_{n - 1}(\text{input})

is evaluated. Otherwise, the explicit trigonometric formula:

.. math::
    \frac{\text{sin}((n + 1) \times \text{arccos}(\text{input}))}{\text{sin}(\text{arccos}(\text{input}))}

is evaluated.

"""
    + r"""
Args:
    {input}
    n (Tensor): Degree of the polynomial.

Keyword args:
    {out}
""".format(**common_args),
)
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1130-1153 / 第 1130-1153 行
````python
chebyshev_polynomial_v = _add_docstr(
    _special.special_chebyshev_polynomial_v,
    r"""
chebyshev_polynomial_v(input, n, *, out=None) -> Tensor

Chebyshev polynomial of the third kind :math:`V_{n}^{\ast}(\text{input})`.

"""
    + r"""
Args:
    {input}
    n (Tensor): Degree of the polynomial.

Keyword args:
    {out}
""".format(**common_args),
)

chebyshev_polynomial_w = _add_docstr(
    _special.special_chebyshev_polynomial_w,
    r"""
chebyshev_polynomial_w(input, n, *, out=None) -> Tensor

Chebyshev polynomial of the fourth kind :math:`W_{n}^{\ast}(\text{input})`.
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1155-1179 / 第 1155-1179 行
````python
"""
    + r"""
Args:
    {input}
    n (Tensor): Degree of the polynomial.

Keyword args:
    {out}
""".format(**common_args),
)

hermite_polynomial_h = _add_docstr(
    _special.special_hermite_polynomial_h,
    r"""
hermite_polynomial_h(input, n, *, out=None) -> Tensor

Physicist's Hermite polynomial :math:`H_{n}(\text{input})`.

If :math:`n = 0`, :math:`1` is returned. If :math:`n = 1`, :math:`\text{input}`
is returned. Otherwise, the recursion:

.. math::
    H_{n + 1}(\text{input}) = 2 \times \text{input} \times H_{n}(\text{input}) - H_{n - 1}(\text{input})

is evaluated.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 1181-1205 / 第 1181-1205 行
````python
"""
    + r"""
Args:
    {input}
    n (Tensor): Degree of the polynomial.

Keyword args:
    {out}
""".format(**common_args),
)

hermite_polynomial_he = _add_docstr(
    _special.special_hermite_polynomial_he,
    r"""
hermite_polynomial_he(input, n, *, out=None) -> Tensor

Probabilist's Hermite polynomial :math:`He_{n}(\text{input})`.

If :math:`n = 0`, :math:`1` is returned. If :math:`n = 1`, :math:`\text{input}`
is returned. Otherwise, the recursion:

.. math::
    He_{n + 1}(\text{input}) = 2 \times \text{input} \times He_{n}(\text{input}) - He_{n - 1}(\text{input})

is evaluated.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 1207-1231 / 第 1207-1231 行
````python
"""
    + r"""
Args:
    {input}
    n (Tensor): Degree of the polynomial.

Keyword args:
    {out}
""".format(**common_args),
)

laguerre_polynomial_l = _add_docstr(
    _special.special_laguerre_polynomial_l,
    r"""
laguerre_polynomial_l(input, n, *, out=None) -> Tensor

Laguerre polynomial :math:`L_{n}(\text{input})`.

If :math:`n = 0`, :math:`1` is returned. If :math:`n = 1`, :math:`\text{input}`
is returned. Otherwise, the recursion:

.. math::
    L_{n + 1}(\text{input}) = 2 \times \text{input} \times L_{n}(\text{input}) - L_{n - 1}(\text{input})

is evaluated.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 1233-1257 / 第 1233-1257 行
````python
"""
    + r"""
Args:
    {input}
    n (Tensor): Degree of the polynomial.

Keyword args:
    {out}
""".format(**common_args),
)

legendre_polynomial_p = _add_docstr(
    _special.special_legendre_polynomial_p,
    r"""
legendre_polynomial_p(input, n, *, out=None) -> Tensor

Legendre polynomial :math:`P_{n}(\text{input})`.

If :math:`n = 0`, :math:`1` is returned. If :math:`n = 1`, :math:`\text{input}`
is returned. Otherwise, the recursion:

.. math::
    P_{n + 1}(\text{input}) = 2 \times \text{input} \times P_{n}(\text{input}) - P_{n - 1}(\text{input})

is evaluated.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 1259-1285 / 第 1259-1285 行
````python
"""
    + r"""
Args:
    {input}
    n (Tensor): Degree of the polynomial.

Keyword args:
    {out}
""".format(**common_args),
)

modified_bessel_i0 = _add_docstr(
    _special.special_modified_bessel_i0,
    r"""
modified_bessel_i0(input, *, out=None) -> Tensor

Modified Bessel function of the first kind of order :math:`0`.

"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
""".format(**common_args),
)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 1287-1309 / 第 1287-1309 行
````python
modified_bessel_i1 = _add_docstr(
    _special.special_modified_bessel_i1,
    r"""
modified_bessel_i1(input, *, out=None) -> Tensor

Modified Bessel function of the first kind of order :math:`1`.

"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
""".format(**common_args),
)

modified_bessel_k0 = _add_docstr(
    _special.special_modified_bessel_k0,
    r"""
modified_bessel_k0(input, *, out=None) -> Tensor

Modified Bessel function of the second kind of order :math:`0`.
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1311-1336 / 第 1311-1336 行
````python
"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
""".format(**common_args),
)

modified_bessel_k1 = _add_docstr(
    _special.special_modified_bessel_k1,
    r"""
modified_bessel_k1(input, *, out=None) -> Tensor

Modified Bessel function of the second kind of order :math:`1`.

"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
""".format(**common_args),
)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 1338-1360 / 第 1338-1360 行
````python
scaled_modified_bessel_k0 = _add_docstr(
    _special.special_scaled_modified_bessel_k0,
    r"""
scaled_modified_bessel_k0(input, *, out=None) -> Tensor

Scaled modified Bessel function of the second kind of order :math:`0`.

"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
""".format(**common_args),
)

scaled_modified_bessel_k1 = _add_docstr(
    _special.special_scaled_modified_bessel_k1,
    r"""
scaled_modified_bessel_k1(input, *, out=None) -> Tensor

Scaled modified Bessel function of the second kind of order :math:`1`.
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1362-1388 / 第 1362-1388 行
````python
"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
""".format(**common_args),
)

shifted_chebyshev_polynomial_t = _add_docstr(
    _special.special_shifted_chebyshev_polynomial_t,
    r"""
shifted_chebyshev_polynomial_t(input, n, *, out=None) -> Tensor

Chebyshev polynomial of the first kind :math:`T_{n}^{\ast}(\text{input})`.

"""
    + r"""
Args:
    {input}
    n (Tensor): Degree of the polynomial.

Keyword args:
    {out}
""".format(**common_args),
)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 1390-1413 / 第 1390-1413 行
````python
shifted_chebyshev_polynomial_u = _add_docstr(
    _special.special_shifted_chebyshev_polynomial_u,
    r"""
shifted_chebyshev_polynomial_u(input, n, *, out=None) -> Tensor

Chebyshev polynomial of the second kind :math:`U_{n}^{\ast}(\text{input})`.

"""
    + r"""
Args:
    {input}
    n (Tensor): Degree of the polynomial.

Keyword args:
    {out}
""".format(**common_args),
)

shifted_chebyshev_polynomial_v = _add_docstr(
    _special.special_shifted_chebyshev_polynomial_v,
    r"""
shifted_chebyshev_polynomial_v(input, n, *, out=None) -> Tensor

Chebyshev polynomial of the third kind :math:`V_{n}^{\ast}(\text{input})`.
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1415-1437 / 第 1415-1437 行
````python
"""
    + r"""
Args:
    {input}
    n (Tensor): Degree of the polynomial.

Keyword args:
    {out}
""".format(**common_args),
)

shifted_chebyshev_polynomial_w = _add_docstr(
    _special.special_shifted_chebyshev_polynomial_w,
    r"""
shifted_chebyshev_polynomial_w(input, n, *, out=None) -> Tensor

Chebyshev polynomial of the fourth kind :math:`W_{n}^{\ast}(\text{input})`.

"""
    + r"""
Args:
    {input}
    n (Tensor): Degree of the polynomial.
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 1439-1459 / 第 1439-1459 行
````python
Keyword args:
    {out}
""".format(**common_args),
)

spherical_bessel_j0 = _add_docstr(
    _special.special_spherical_bessel_j0,
    r"""
spherical_bessel_j0(input, *, out=None) -> Tensor

Spherical Bessel function of the first kind of order :math:`0`.

"""
    + r"""
Args:
    {input}

Keyword args:
    {out}
""".format(**common_args),
)
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

## Key Concepts / 关键概念

- **Special functions**
  - EN: Exposes numerical special functions through Python namespace wrappers.
  - CN: 通过 Python 命名空间包装层暴露数值特殊函数。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
- **Package wiring**
  - EN: The file shapes the public namespace by re-exporting symbols and applying package-level initialization.
  - CN: 该文件通过重新导出符号并执行包级初始化来塑造公共命名空间。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._C`, `torch._torch_docs`
- **Primary symbols in this file / 本文件核心符号**: `__all__`
