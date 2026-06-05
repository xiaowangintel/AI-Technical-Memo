# __init__.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fft/__init__.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Initializes the package namespace, re-exports symbols, and wires lazy imports or feature flags.
- **Purpose (CN)**: 初始化包命名空间，重新导出符号，并连接懒加载导入或特性开关。
## Line-by-Line Analysis / 逐行分析

### Lines 1-26 / 第 1-26 行
````python
import torch
from torch._C import _add_docstr, _fft  # type: ignore[attr-defined]
from torch._torch_docs import common_args, factory_common_args


__all__ = [
    "fft",
    "ifft",
    "fft2",
    "ifft2",
    "fftn",
    "ifftn",
    "rfft",
    "irfft",
    "rfft2",
    "irfft2",
    "rfftn",
    "irfftn",
    "hfft",
    "ihfft",
    "fftfreq",
    "rfftfreq",
    "fftshift",
    "ifftshift",
    "Tensor",
]
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._C, torch._torch_docs. `__all__` defines the public symbols that this module chooses to export.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._C、torch._torch_docs。 `__all__` 定义了本模块选择导出的公共符号。

### Lines 28-50 / 第 28-50 行
````python
Tensor = torch.Tensor

# Note: This not only adds the doc strings for the spectral ops, but
# connects the torch.fft Python namespace to the torch._C._fft builtins.

fft = _add_docstr(
    _fft.fft_fft,
    r"""
fft(input, n=None, dim=-1, norm=None, *, out=None) -> Tensor

Computes the one dimensional discrete Fourier transform of :attr:`input`.

Note:
    The Fourier domain representation of any real signal satisfies the
    Hermitian property: `X[i] = conj(X[-i])`. This function always returns both
    the positive and negative frequency terms even though, for real inputs, the
    negative frequencies are redundant. :func:`~torch.fft.rfft` returns the
    more compact one-sided representation where only the positive frequencies
    are returned.

Note:
    Supports torch.half and torch.chalf on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimension.
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 52-74 / 第 52-74 行
````python
Args:
    input (Tensor): the input tensor
    n (int, optional): Signal length. If given, the input will either be zero-padded
        or trimmed to this length before computing the FFT.
    dim (int, optional): The dimension along which to take the one dimensional FFT.
    norm (str, optional): Normalization mode. For the forward transform
        (:func:`~torch.fft.fft`), these correspond to:

        * ``"forward"`` - normalize by ``1/n``
        * ``"backward"`` - no normalization
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the FFT orthonormal)

        Calling the backward transform (:func:`~torch.fft.ifft`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.ifft`
        the exact inverse.

        Default is ``"backward"`` (no normalization).

Keyword args:
    {out}

Example:
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 76-97 / 第 76-97 行
````python
    >>> t = torch.arange(4)
    >>> t
    tensor([0, 1, 2, 3])
    >>> torch.fft.fft(t)
    tensor([ 6.+0.j, -2.+2.j, -2.+0.j, -2.-2.j])

    >>> t = torch.tensor([0.+1.j, 2.+3.j, 4.+5.j, 6.+7.j])
    >>> torch.fft.fft(t)
    tensor([12.+16.j, -8.+0.j, -4.-4.j,  0.-8.j])
""".format(**common_args),
)

ifft = _add_docstr(
    _fft.fft_ifft,
    r"""
ifft(input, n=None, dim=-1, norm=None, *, out=None) -> Tensor

Computes the one dimensional inverse discrete Fourier transform of :attr:`input`.

Note:
    Supports torch.half and torch.chalf on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimension.
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 99-121 / 第 99-121 行
````python
Args:
    input (Tensor): the input tensor
    n (int, optional): Signal length. If given, the input will either be zero-padded
        or trimmed to this length before computing the IFFT.
    dim (int, optional): The dimension along which to take the one dimensional IFFT.
    norm (str, optional): Normalization mode. For the backward transform
        (:func:`~torch.fft.ifft`), these correspond to:

        * ``"forward"`` - no normalization
        * ``"backward"`` - normalize by ``1/n``
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the IFFT orthonormal)

        Calling the forward transform (:func:`~torch.fft.fft`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.ifft`
        the exact inverse.

        Default is ``"backward"`` (normalize by ``1/n``).

Keyword args:
    {out}

Example:
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 123-147 / 第 123-147 行
````python
    >>> t = torch.tensor([ 6.+0.j, -2.+2.j, -2.+0.j, -2.-2.j])
    >>> torch.fft.ifft(t)
    tensor([0.+0.j, 1.+0.j, 2.+0.j, 3.+0.j])
""".format(**common_args),
)

fft2 = _add_docstr(
    _fft.fft_fft2,
    r"""
fft2(input, s=None, dim=(-2, -1), norm=None, *, out=None) -> Tensor

Computes the 2 dimensional discrete Fourier transform of :attr:`input`.
Equivalent to :func:`~torch.fft.fftn` but FFTs only the last two dimensions by default.

Note:
    The Fourier domain representation of any real signal satisfies the
    Hermitian property: ``X[i, j] = conj(X[-i, -j])``. This
    function always returns all positive and negative frequency terms even
    though, for real inputs, half of these values are redundant.
    :func:`~torch.fft.rfft2` returns the more compact one-sided representation
    where only the positive frequencies of the last dimension are returned.

Note:
    Supports torch.half and torch.chalf on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimensions.
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 149-174 / 第 149-174 行
````python
Args:
    input (Tensor): the input tensor
    s (Tuple[int], optional): Signal size in the transformed dimensions.
        If given, each dimension ``dim[i]`` will either be zero-padded or
        trimmed to the length ``s[i]`` before computing the FFT.
        If a length ``-1`` is specified, no padding is done in that dimension.
        Default: ``s = [input.size(d) for d in dim]``
    dim (Tuple[int], optional): Dimensions to be transformed.
        Default: last two dimensions.
    norm (str, optional): Normalization mode. For the forward transform
        (:func:`~torch.fft.fft2`), these correspond to:

        * ``"forward"`` - normalize by ``1/n``
        * ``"backward"`` - no normalization
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the FFT orthonormal)

        Where ``n = prod(s)`` is the logical FFT size.
        Calling the backward transform (:func:`~torch.fft.ifft2`) with the same
        normalization mode will apply an overall normalization of ``1/n``
        between the two transforms. This is required to make
        :func:`~torch.fft.ifft2` the exact inverse.

        Default is ``"backward"`` (no normalization).

Keyword args:
    {out}
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 176-200 / 第 176-200 行
````python
Example:

    >>> x = torch.rand(10, 10, dtype=torch.complex64)
    >>> fft2 = torch.fft.fft2(x)

    The discrete Fourier transform is separable, so :func:`~torch.fft.fft2`
    here is equivalent to two one-dimensional :func:`~torch.fft.fft` calls:

    >>> two_ffts = torch.fft.fft(torch.fft.fft(x, dim=0), dim=1)
    >>> torch.testing.assert_close(fft2, two_ffts, check_stride=False)

""".format(**common_args),
)

ifft2 = _add_docstr(
    _fft.fft_ifft2,
    r"""
ifft2(input, s=None, dim=(-2, -1), norm=None, *, out=None) -> Tensor

Computes the 2 dimensional inverse discrete Fourier transform of :attr:`input`.
Equivalent to :func:`~torch.fft.ifftn` but IFFTs only the last two dimensions by default.

Note:
    Supports torch.half and torch.chalf on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimensions.
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 202-227 / 第 202-227 行
````python
Args:
    input (Tensor): the input tensor
    s (Tuple[int], optional): Signal size in the transformed dimensions.
        If given, each dimension ``dim[i]`` will either be zero-padded or
        trimmed to the length ``s[i]`` before computing the IFFT.
        If a length ``-1`` is specified, no padding is done in that dimension.
        Default: ``s = [input.size(d) for d in dim]``
    dim (Tuple[int], optional): Dimensions to be transformed.
        Default: last two dimensions.
    norm (str, optional): Normalization mode. For the backward transform
        (:func:`~torch.fft.ifft2`), these correspond to:

        * ``"forward"`` - no normalization
        * ``"backward"`` - normalize by ``1/n``
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the IFFT orthonormal)

        Where ``n = prod(s)`` is the logical IFFT size.
        Calling the forward transform (:func:`~torch.fft.fft2`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.ifft2`
        the exact inverse.

        Default is ``"backward"`` (normalize by ``1/n``).

Keyword args:
    {out}
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 229-248 / 第 229-248 行
````python
Example:

    >>> x = torch.rand(10, 10, dtype=torch.complex64)
    >>> ifft2 = torch.fft.ifft2(x)

    The discrete Fourier transform is separable, so :func:`~torch.fft.ifft2`
    here is equivalent to two one-dimensional :func:`~torch.fft.ifft` calls:

    >>> two_iffts = torch.fft.ifft(torch.fft.ifft(x, dim=0), dim=1)
    >>> torch.testing.assert_close(ifft2, two_iffts, check_stride=False)

""".format(**common_args),
)

fftn = _add_docstr(
    _fft.fft_fftn,
    r"""
fftn(input, s=None, dim=None, norm=None, *, out=None) -> Tensor

Computes the N dimensional discrete Fourier transform of :attr:`input`.
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 250-276 / 第 250-276 行
````python
Note:
    The Fourier domain representation of any real signal satisfies the
    Hermitian property: ``X[i_1, ..., i_n] = conj(X[-i_1, ..., -i_n])``. This
    function always returns all positive and negative frequency terms even
    though, for real inputs, half of these values are redundant.
    :func:`~torch.fft.rfftn` returns the more compact one-sided representation
    where only the positive frequencies of the last dimension are returned.

Note:
    Supports torch.half and torch.chalf on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimensions.

Args:
    input (Tensor): the input tensor
    s (Tuple[int], optional): Signal size in the transformed dimensions.
        If given, each dimension ``dim[i]`` will either be zero-padded or
        trimmed to the length ``s[i]`` before computing the FFT.
        If a length ``-1`` is specified, no padding is done in that dimension.
        Default: ``s = [input.size(d) for d in dim]``
    dim (Tuple[int], optional): Dimensions to be transformed.
        Default: all dimensions, or the last ``len(s)`` dimensions if :attr:`s` is given.
    norm (str, optional): Normalization mode. For the forward transform
        (:func:`~torch.fft.fftn`), these correspond to:

        * ``"forward"`` - normalize by ``1/n``
        * ``"backward"`` - no normalization
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the FFT orthonormal)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 278-301 / 第 278-301 行
````python
        Where ``n = prod(s)`` is the logical FFT size.
        Calling the backward transform (:func:`~torch.fft.ifftn`) with the same
        normalization mode will apply an overall normalization of ``1/n``
        between the two transforms. This is required to make
        :func:`~torch.fft.ifftn` the exact inverse.

        Default is ``"backward"`` (no normalization).

Keyword args:
    {out}

Example:

    >>> x = torch.rand(10, 10, dtype=torch.complex64)
    >>> fftn = torch.fft.fftn(x)

    The discrete Fourier transform is separable, so :func:`~torch.fft.fftn`
    here is equivalent to two one-dimensional :func:`~torch.fft.fft` calls:

    >>> two_ffts = torch.fft.fft(torch.fft.fft(x, dim=0), dim=1)
    >>> torch.testing.assert_close(fftn, two_ffts, check_stride=False)

""".format(**common_args),
)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 303-328 / 第 303-328 行
````python
ifftn = _add_docstr(
    _fft.fft_ifftn,
    r"""
ifftn(input, s=None, dim=None, norm=None, *, out=None) -> Tensor

Computes the N dimensional inverse discrete Fourier transform of :attr:`input`.

Note:
    Supports torch.half and torch.chalf on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimensions.

Args:
    input (Tensor): the input tensor
    s (Tuple[int], optional): Signal size in the transformed dimensions.
        If given, each dimension ``dim[i]`` will either be zero-padded or
        trimmed to the length ``s[i]`` before computing the IFFT.
        If a length ``-1`` is specified, no padding is done in that dimension.
        Default: ``s = [input.size(d) for d in dim]``
    dim (Tuple[int], optional): Dimensions to be transformed.
        Default: all dimensions, or the last ``len(s)`` dimensions if :attr:`s` is given.
    norm (str, optional): Normalization mode. For the backward transform
        (:func:`~torch.fft.ifftn`), these correspond to:

        * ``"forward"`` - no normalization
        * ``"backward"`` - normalize by ``1/n``
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the IFFT orthonormal)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 330-353 / 第 330-353 行
````python
        Where ``n = prod(s)`` is the logical IFFT size.
        Calling the forward transform (:func:`~torch.fft.fftn`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.ifftn`
        the exact inverse.

        Default is ``"backward"`` (normalize by ``1/n``).

Keyword args:
    {out}

Example:

    >>> x = torch.rand(10, 10, dtype=torch.complex64)
    >>> ifftn = torch.fft.ifftn(x)

    The discrete Fourier transform is separable, so :func:`~torch.fft.ifftn`
    here is equivalent to two one-dimensional :func:`~torch.fft.ifft` calls:

    >>> two_iffts = torch.fft.ifft(torch.fft.ifft(x, dim=0), dim=1)
    >>> torch.testing.assert_close(ifftn, two_iffts, check_stride=False)

""".format(**common_args),
)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 355-380 / 第 355-380 行
````python
rfft = _add_docstr(
    _fft.fft_rfft,
    r"""
rfft(input, n=None, dim=-1, norm=None, *, out=None) -> Tensor

Computes the one dimensional Fourier transform of real-valued :attr:`input`.

The FFT of a real signal is Hermitian-symmetric, ``X[i] = conj(X[-i])`` so
the output contains only the positive frequencies below the Nyquist frequency.
To compute the full output, use :func:`~torch.fft.fft`

Note:
    Supports torch.half on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimension.

Args:
    input (Tensor): the real input tensor
    n (int, optional): Signal length. If given, the input will either be zero-padded
        or trimmed to this length before computing the real FFT.
    dim (int, optional): The dimension along which to take the one dimensional real FFT.
    norm (str, optional): Normalization mode. For the forward transform
        (:func:`~torch.fft.rfft`), these correspond to:

        * ``"forward"`` - normalize by ``1/n``
        * ``"backward"`` - no normalization
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the FFT orthonormal)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 382-403 / 第 382-403 行
````python
        Calling the backward transform (:func:`~torch.fft.irfft`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.irfft`
        the exact inverse.

        Default is ``"backward"`` (no normalization).

Keyword args:
    {out}

Example:

    >>> t = torch.arange(4)
    >>> t
    tensor([0, 1, 2, 3])
    >>> torch.fft.rfft(t)
    tensor([ 6.+0.j, -2.+2.j, -2.+0.j])

    Compare against the full output from :func:`~torch.fft.fft`:

    >>> torch.fft.fft(t)
    tensor([ 6.+0.j, -2.+2.j, -2.+0.j, -2.-2.j])
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 405-426 / 第 405-426 行
````python
    Notice that the symmetric element ``T[-1] == T[1].conj()`` is omitted.
    At the Nyquist frequency ``T[-2] == T[2]`` is it's own symmetric pair,
    and therefore must always be real-valued.
""".format(**common_args),
)

irfft = _add_docstr(
    _fft.fft_irfft,
    r"""
irfft(input, n=None, dim=-1, norm=None, *, out=None) -> Tensor

Computes the inverse of :func:`~torch.fft.rfft`.

:attr:`input` is interpreted as a one-sided Hermitian signal in the Fourier
domain, as produced by :func:`~torch.fft.rfft`. By the Hermitian property, the
output will be real-valued.

Note:
    Some input frequencies must be real-valued to satisfy the Hermitian
    property. In these cases the imaginary component will be ignored.
    For example, any imaginary component in the zero-frequency term cannot
    be represented in a real output and so will always be ignored.
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 428-453 / 第 428-453 行
````python
Note:
    The correct interpretation of the Hermitian input depends on the length of
    the original data, as given by :attr:`n`. This is because each input shape
    could correspond to either an odd or even length signal. By default, the
    signal is assumed to be even length and odd signals will not round-trip
    properly. So, it is recommended to always pass the signal length :attr:`n`.

Note:
    Supports torch.half and torch.chalf on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimension.
    With default arguments, size of the transformed dimension should be (2^n + 1) as argument
    `n` defaults to even output size = 2 * (transformed_dim_size - 1)

Args:
    input (Tensor): the input tensor representing a half-Hermitian signal
    n (int, optional): Output signal length. This determines the length of the
        output signal. If given, the input will either be zero-padded or trimmed to this
        length before computing the real IFFT.
        Defaults to even output: ``n=2*(input.size(dim) - 1)``.
    dim (int, optional): The dimension along which to take the one dimensional real IFFT.
    norm (str, optional): Normalization mode. For the backward transform
        (:func:`~torch.fft.irfft`), these correspond to:

        * ``"forward"`` - no normalization
        * ``"backward"`` - normalize by ``1/n``
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the real IFFT orthonormal)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 455-480 / 第 455-480 行
````python
        Calling the forward transform (:func:`~torch.fft.rfft`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.irfft`
        the exact inverse.

        Default is ``"backward"`` (normalize by ``1/n``).

Keyword args:
    {out}

Example:

    >>> t = torch.linspace(0, 1, 5)
    >>> t
    tensor([0.0000, 0.2500, 0.5000, 0.7500, 1.0000])
    >>> T = torch.fft.rfft(t)
    >>> T
    tensor([ 2.5000+0.0000j, -0.6250+0.8602j, -0.6250+0.2031j])

    Without specifying the output length to :func:`~torch.fft.irfft`, the output
    will not round-trip properly because the input is odd-length:

    >>> torch.fft.irfft(T)
    tensor([0.1562, 0.3511, 0.7812, 1.2114])

    So, it is recommended to always pass the signal length :attr:`n`:
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 482-503 / 第 482-503 行
````python
    >>> roundtrip = torch.fft.irfft(T, t.numel())
    >>> torch.testing.assert_close(roundtrip, t, check_stride=False)

""".format(**common_args),
)

rfft2 = _add_docstr(
    _fft.fft_rfft2,
    r"""
rfft2(input, s=None, dim=(-2, -1), norm=None, *, out=None) -> Tensor

Computes the 2-dimensional discrete Fourier transform of real :attr:`input`.
Equivalent to :func:`~torch.fft.rfftn` but FFTs only the last two dimensions by default.

The FFT of a real signal is Hermitian-symmetric, ``X[i, j] = conj(X[-i, -j])``,
so the full :func:`~torch.fft.fft2` output contains redundant information.
:func:`~torch.fft.rfft2` instead omits the negative frequencies in the last
dimension.

Note:
    Supports torch.half on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimensions.
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 505-530 / 第 505-530 行
````python
Args:
    input (Tensor): the input tensor
    s (Tuple[int], optional): Signal size in the transformed dimensions.
        If given, each dimension ``dim[i]`` will either be zero-padded or
        trimmed to the length ``s[i]`` before computing the real FFT.
        If a length ``-1`` is specified, no padding is done in that dimension.
        Default: ``s = [input.size(d) for d in dim]``
    dim (Tuple[int], optional): Dimensions to be transformed.
        Default: last two dimensions.
    norm (str, optional): Normalization mode. For the forward transform
        (:func:`~torch.fft.rfft2`), these correspond to:

        * ``"forward"`` - normalize by ``1/n``
        * ``"backward"`` - no normalization
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the real FFT orthonormal)

        Where ``n = prod(s)`` is the logical FFT size.
        Calling the backward transform (:func:`~torch.fft.irfft2`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.irfft2`
        the exact inverse.

        Default is ``"backward"`` (no normalization).

Keyword args:
    {out}
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 532-558 / 第 532-558 行
````python
Example:

    >>> t = torch.rand(10, 10)
    >>> rfft2 = torch.fft.rfft2(t)
    >>> rfft2.size()
    torch.Size([10, 6])

    Compared against the full output from :func:`~torch.fft.fft2`, we have all
    elements up to the Nyquist frequency.

    >>> fft2 = torch.fft.fft2(t)
    >>> torch.testing.assert_close(fft2[..., :6], rfft2, check_stride=False)

    The discrete Fourier transform is separable, so :func:`~torch.fft.rfft2`
    here is equivalent to a combination of :func:`~torch.fft.fft` and
    :func:`~torch.fft.rfft`:

    >>> two_ffts = torch.fft.fft(torch.fft.rfft(t, dim=1), dim=0)
    >>> torch.testing.assert_close(rfft2, two_ffts, check_stride=False)

""".format(**common_args),
)

irfft2 = _add_docstr(
    _fft.fft_irfft2,
    r"""
irfft2(input, s=None, dim=(-2, -1), norm=None, *, out=None) -> Tensor
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 560-584 / 第 560-584 行
````python
Computes the inverse of :func:`~torch.fft.rfft2`.
Equivalent to :func:`~torch.fft.irfftn` but IFFTs only the last two dimensions by default.

:attr:`input` is interpreted as a one-sided Hermitian signal in the Fourier
domain, as produced by :func:`~torch.fft.rfft2`. By the Hermitian property, the
output will be real-valued.

Note:
    Some input frequencies must be real-valued to satisfy the Hermitian
    property. In these cases the imaginary component will be ignored.
    For example, any imaginary component in the zero-frequency term cannot
    be represented in a real output and so will always be ignored.

Note:
    The correct interpretation of the Hermitian input depends on the length of
    the original data, as given by :attr:`s`. This is because each input shape
    could correspond to either an odd or even length signal. By default, the
    signal is assumed to be even length and odd signals will not round-trip
    properly. So, it is recommended to always pass the signal shape :attr:`s`.

Note:
    Supports torch.half and torch.chalf on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimensions.
    With default arguments, the size of last dimension should be (2^n + 1) as argument
    `s` defaults to even output size = 2 * (last_dim_size - 1)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 586-610 / 第 586-610 行
````python
Args:
    input (Tensor): the input tensor
    s (Tuple[int], optional): Signal size in the transformed dimensions.
        If given, each dimension ``dim[i]`` will either be zero-padded or
        trimmed to the length ``s[i]`` before computing the real FFT.
        If a length ``-1`` is specified, no padding is done in that dimension.
        Defaults to even output in the last dimension:
        ``s[-1] = 2*(input.size(dim[-1]) - 1)``.
    dim (Tuple[int], optional): Dimensions to be transformed.
        The last dimension must be the half-Hermitian compressed dimension.
        Default: last two dimensions.
    norm (str, optional): Normalization mode. For the backward transform
        (:func:`~torch.fft.irfft2`), these correspond to:

        * ``"forward"`` - no normalization
        * ``"backward"`` - normalize by ``1/n``
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the real IFFT orthonormal)

        Where ``n = prod(s)`` is the logical IFFT size.
        Calling the forward transform (:func:`~torch.fft.rfft2`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.irfft2`
        the exact inverse.

        Default is ``"backward"`` (normalize by ``1/n``).
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 612-635 / 第 612-635 行
````python
Keyword args:
    {out}

Example:

    >>> t = torch.rand(10, 9)
    >>> T = torch.fft.rfft2(t)

    Without specifying the output length to :func:`~torch.fft.irfft2`, the output
    will not round-trip properly because the input is odd-length in the last
    dimension:

    >>> torch.fft.irfft2(T).size()
    torch.Size([10, 8])

    So, it is recommended to always pass the signal shape :attr:`s`.

    >>> roundtrip = torch.fft.irfft2(T, t.size())
    >>> roundtrip.size()
    torch.Size([10, 9])
    >>> torch.testing.assert_close(roundtrip, t, check_stride=False)

""".format(**common_args),
)
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 637-664 / 第 637-664 行
````python
rfftn = _add_docstr(
    _fft.fft_rfftn,
    r"""
rfftn(input, s=None, dim=None, norm=None, *, out=None) -> Tensor

Computes the N-dimensional discrete Fourier transform of real :attr:`input`.

The FFT of a real signal is Hermitian-symmetric,
``X[i_1, ..., i_n] = conj(X[-i_1, ..., -i_n])`` so the full
:func:`~torch.fft.fftn` output contains redundant information.
:func:`~torch.fft.rfftn` instead omits the negative frequencies in the
last dimension.

Note:
    Supports torch.half on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimensions.

Args:
    input (Tensor): the input tensor
    s (Tuple[int], optional): Signal size in the transformed dimensions.
        If given, each dimension ``dim[i]`` will either be zero-padded or
        trimmed to the length ``s[i]`` before computing the real FFT.
        If a length ``-1`` is specified, no padding is done in that dimension.
        Default: ``s = [input.size(d) for d in dim]``
    dim (Tuple[int], optional): Dimensions to be transformed.
        Default: all dimensions, or the last ``len(s)`` dimensions if :attr:`s` is given.
    norm (str, optional): Normalization mode. For the forward transform
        (:func:`~torch.fft.rfftn`), these correspond to:
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 666-692 / 第 666-692 行
````python
        * ``"forward"`` - normalize by ``1/n``
        * ``"backward"`` - no normalization
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the real FFT orthonormal)

        Where ``n = prod(s)`` is the logical FFT size.
        Calling the backward transform (:func:`~torch.fft.irfftn`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.irfftn`
        the exact inverse.

        Default is ``"backward"`` (no normalization).

Keyword args:
    {out}

Example:

    >>> t = torch.rand(10, 10)
    >>> rfftn = torch.fft.rfftn(t)
    >>> rfftn.size()
    torch.Size([10, 6])

    Compared against the full output from :func:`~torch.fft.fftn`, we have all
    elements up to the Nyquist frequency.

    >>> fftn = torch.fft.fftn(t)
    >>> torch.testing.assert_close(fftn[..., :6], rfftn, check_stride=False)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 694-719 / 第 694-719 行
````python
    The discrete Fourier transform is separable, so :func:`~torch.fft.rfftn`
    here is equivalent to a combination of :func:`~torch.fft.fft` and
    :func:`~torch.fft.rfft`:

    >>> two_ffts = torch.fft.fft(torch.fft.rfft(t, dim=1), dim=0)
    >>> torch.testing.assert_close(rfftn, two_ffts, check_stride=False)

""".format(**common_args),
)

irfftn = _add_docstr(
    _fft.fft_irfftn,
    r"""
irfftn(input, s=None, dim=None, norm=None, *, out=None) -> Tensor

Computes the inverse of :func:`~torch.fft.rfftn`.

:attr:`input` is interpreted as a one-sided Hermitian signal in the Fourier
domain, as produced by :func:`~torch.fft.rfftn`. By the Hermitian property, the
output will be real-valued.

Note:
    Some input frequencies must be real-valued to satisfy the Hermitian
    property. In these cases the imaginary component will be ignored.
    For example, any imaginary component in the zero-frequency term cannot
    be represented in a real output and so will always be ignored.
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 721-746 / 第 721-746 行
````python
Note:
    The correct interpretation of the Hermitian input depends on the length of
    the original data, as given by :attr:`s`. This is because each input shape
    could correspond to either an odd or even length signal. By default, the
    signal is assumed to be even length and odd signals will not round-trip
    properly. So, it is recommended to always pass the signal shape :attr:`s`.

Note:
    Supports torch.half and torch.chalf on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimensions.
    With default arguments, the size of last dimension should be (2^n + 1) as argument
    `s` defaults to even output size = 2 * (last_dim_size - 1)

Args:
    input (Tensor): the input tensor
    s (Tuple[int], optional): Signal size in the transformed dimensions.
        If given, each dimension ``dim[i]`` will either be zero-padded or
        trimmed to the length ``s[i]`` before computing the real FFT.
        If a length ``-1`` is specified, no padding is done in that dimension.
        Defaults to even output in the last dimension:
        ``s[-1] = 2*(input.size(dim[-1]) - 1)``.
    dim (Tuple[int], optional): Dimensions to be transformed.
        The last dimension must be the half-Hermitian compressed dimension.
        Default: all dimensions, or the last ``len(s)`` dimensions if :attr:`s` is given.
    norm (str, optional): Normalization mode. For the backward transform
        (:func:`~torch.fft.irfftn`), these correspond to:
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 748-773 / 第 748-773 行
````python
        * ``"forward"`` - no normalization
        * ``"backward"`` - normalize by ``1/n``
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the real IFFT orthonormal)

        Where ``n = prod(s)`` is the logical IFFT size.
        Calling the forward transform (:func:`~torch.fft.rfftn`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.irfftn`
        the exact inverse.

        Default is ``"backward"`` (normalize by ``1/n``).

Keyword args:
    {out}

Example:

    >>> t = torch.rand(10, 9)
    >>> T = torch.fft.rfftn(t)

    Without specifying the output length to :func:`~torch.fft.irfft`, the output
    will not round-trip properly because the input is odd-length in the last
    dimension:

    >>> torch.fft.irfftn(T).size()
    torch.Size([10, 8])
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 775-801 / 第 775-801 行
````python
    So, it is recommended to always pass the signal shape :attr:`s`.

    >>> roundtrip = torch.fft.irfftn(T, t.size())
    >>> roundtrip.size()
    torch.Size([10, 9])
    >>> torch.testing.assert_close(roundtrip, t, check_stride=False)

""".format(**common_args),
)

hfft = _add_docstr(
    _fft.fft_hfft,
    r"""
hfft(input, n=None, dim=-1, norm=None, *, out=None) -> Tensor

Computes the one dimensional discrete Fourier transform of a Hermitian
symmetric :attr:`input` signal.

Note:

    :func:`~torch.fft.hfft`/:func:`~torch.fft.ihfft` are analogous to
    :func:`~torch.fft.rfft`/:func:`~torch.fft.irfft`. The real FFT expects
    a real signal in the time-domain and gives a Hermitian symmetry in the
    frequency-domain. The Hermitian FFT is the opposite; Hermitian symmetric in
    the time-domain and real-valued in the frequency-domain. For this reason,
    special care needs to be taken with the length argument :attr:`n`, in the
    same way as with :func:`~torch.fft.irfft`.
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 803-822 / 第 803-822 行
````python
Note:
    Because the signal is Hermitian in the time-domain, the result will be
    real in the frequency domain. Note that some input frequencies must be
    real-valued to satisfy the Hermitian property. In these cases the imaginary
    component will be ignored. For example, any imaginary component in
    ``input[0]`` would result in one or more complex frequency terms which
    cannot be represented in a real output and so will always be ignored.

Note:
    The correct interpretation of the Hermitian input depends on the length of
    the original data, as given by :attr:`n`. This is because each input shape
    could correspond to either an odd or even length signal. By default, the
    signal is assumed to be even length and odd signals will not round-trip
    properly. So, it is recommended to always pass the signal length :attr:`n`.

Note:
    Supports torch.half and torch.chalf on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimension.
    With default arguments, size of the transformed dimension should be (2^n + 1) as argument
    `n` defaults to even output size = 2 * (transformed_dim_size - 1)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 824-848 / 第 824-848 行
````python
Args:
    input (Tensor): the input tensor representing a half-Hermitian signal
    n (int, optional): Output signal length. This determines the length of the
        real output. If given, the input will either be zero-padded or trimmed to this
        length before computing the Hermitian FFT.
        Defaults to even output: ``n=2*(input.size(dim) - 1)``.
    dim (int, optional): The dimension along which to take the one dimensional Hermitian FFT.
    norm (str, optional): Normalization mode. For the forward transform
        (:func:`~torch.fft.hfft`), these correspond to:

        * ``"forward"`` - normalize by ``1/n``
        * ``"backward"`` - no normalization
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the Hermitian FFT orthonormal)

        Calling the backward transform (:func:`~torch.fft.ihfft`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.ihfft`
        the exact inverse.

        Default is ``"backward"`` (no normalization).

Keyword args:
    {out}

Example:
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 850-874 / 第 850-874 行
````python
    Taking a real-valued frequency signal and bringing it into the time domain
    gives Hermitian symmetric output:

    >>> t = torch.linspace(0, 1, 5)
    >>> t
    tensor([0.0000, 0.2500, 0.5000, 0.7500, 1.0000])
    >>> T = torch.fft.ifft(t)
    >>> T
    tensor([ 0.5000-0.0000j, -0.1250-0.1720j, -0.1250-0.0406j, -0.1250+0.0406j,
            -0.1250+0.1720j])

    Note that ``T[1] == T[-1].conj()`` and ``T[2] == T[-2].conj()`` is
    redundant. We can thus compute the forward transform without considering
    negative frequencies:

    >>> torch.fft.hfft(T[:3], n=5)
    tensor([0.0000, 0.2500, 0.5000, 0.7500, 1.0000])

    Like with :func:`~torch.fft.irfft`, the output length must be given in order
    to recover an even length output:

    >>> torch.fft.hfft(T[:3])
    tensor([0.1250, 0.2809, 0.6250, 0.9691])
""".format(**common_args),
)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 876-899 / 第 876-899 行
````python
ihfft = _add_docstr(
    _fft.fft_ihfft,
    r"""
ihfft(input, n=None, dim=-1, norm=None, *, out=None) -> Tensor

Computes the inverse of :func:`~torch.fft.hfft`.

:attr:`input` must be a real-valued signal, interpreted in the Fourier domain.
The IFFT of a real signal is Hermitian-symmetric, ``X[i] = conj(X[-i])``.
:func:`~torch.fft.ihfft` represents this in the one-sided form where only the
positive frequencies below the Nyquist frequency are included. To compute the
full output, use :func:`~torch.fft.ifft`.

Note:
    Supports torch.half on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimension.

Args:
    input (Tensor): the real input tensor
    n (int, optional): Signal length. If given, the input will either be zero-padded
        or trimmed to this length before computing the Hermitian IFFT.
    dim (int, optional): The dimension along which to take the one dimensional Hermitian IFFT.
    norm (str, optional): Normalization mode. For the backward transform
        (:func:`~torch.fft.ihfft`), these correspond to:
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 901-923 / 第 901-923 行
````python
        * ``"forward"`` - no normalization
        * ``"backward"`` - normalize by ``1/n``
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the IFFT orthonormal)

        Calling the forward transform (:func:`~torch.fft.hfft`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.ihfft`
        the exact inverse.

        Default is ``"backward"`` (normalize by ``1/n``).

Keyword args:
    {out}

Example:

    >>> t = torch.arange(5)
    >>> t
    tensor([0, 1, 2, 3, 4])
    >>> torch.fft.ihfft(t)
    tensor([ 2.0000-0.0000j, -0.5000-0.6882j, -0.5000-0.1625j])

    Compare against the full output from :func:`~torch.fft.ifft`:
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 925-947 / 第 925-947 行
````python
    >>> torch.fft.ifft(t)
    tensor([ 2.0000-0.0000j, -0.5000-0.6882j, -0.5000-0.1625j, -0.5000+0.1625j,
            -0.5000+0.6882j])
""".format(**common_args),
)

hfft2 = _add_docstr(
    _fft.fft_hfft2,
    r"""
hfft2(input, s=None, dim=(-2, -1), norm=None, *, out=None) -> Tensor

Computes the 2-dimensional discrete Fourier transform of a Hermitian symmetric
:attr:`input` signal. Equivalent to :func:`~torch.fft.hfftn` but only
transforms the last two dimensions by default.

:attr:`input` is interpreted as a one-sided Hermitian signal in the time
domain. By the Hermitian property, the Fourier transform will be real-valued.

Note:
    Supports torch.half and torch.chalf on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimensions.
    With default arguments, the size of last dimension should be (2^n + 1) as argument
    `s` defaults to even output size = 2 * (last_dim_size - 1)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 949-973 / 第 949-973 行
````python
Args:
    input (Tensor): the input tensor
    s (Tuple[int], optional): Signal size in the transformed dimensions.
        If given, each dimension ``dim[i]`` will either be zero-padded or
        trimmed to the length ``s[i]`` before computing the Hermitian FFT.
        If a length ``-1`` is specified, no padding is done in that dimension.
        Defaults to even output in the last dimension:
        ``s[-1] = 2*(input.size(dim[-1]) - 1)``.
    dim (Tuple[int], optional): Dimensions to be transformed.
        The last dimension must be the half-Hermitian compressed dimension.
        Default: last two dimensions.
    norm (str, optional): Normalization mode. For the forward transform
        (:func:`~torch.fft.hfft2`), these correspond to:

        * ``"forward"`` - normalize by ``1/n``
        * ``"backward"`` - no normalization
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the Hermitian FFT orthonormal)

        Where ``n = prod(s)`` is the logical FFT size.
        Calling the backward transform (:func:`~torch.fft.ihfft2`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.ihfft2`
        the exact inverse.

        Default is ``"backward"`` (no normalization).
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 975-1001 / 第 975-1001 行
````python
Keyword args:
    {out}

Example:

    Starting from a real frequency-space signal, we can generate a
    Hermitian-symmetric time-domain signal:
    >>> T = torch.rand(10, 9)
    >>> t = torch.fft.ihfft2(T)

    Without specifying the output length to :func:`~torch.fft.hfftn`, the
    output will not round-trip properly because the input is odd-length in the
    last dimension:

    >>> torch.fft.hfft2(t).size()
    torch.Size([10, 10])

    So, it is recommended to always pass the signal shape :attr:`s`.

    >>> roundtrip = torch.fft.hfft2(t, T.size())
    >>> roundtrip.size()
    torch.Size([10, 9])
    >>> torch.allclose(roundtrip, T)
    True

""".format(**common_args),
)
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1003-1026 / 第 1003-1026 行
````python
ihfft2 = _add_docstr(
    _fft.fft_ihfft2,
    r"""
ihfft2(input, s=None, dim=(-2, -1), norm=None, *, out=None) -> Tensor

Computes the 2-dimensional inverse discrete Fourier transform of real
:attr:`input`. Equivalent to :func:`~torch.fft.ihfftn` but transforms only the
two last dimensions by default.

Note:
    Supports torch.half on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimensions.

Args:
    input (Tensor): the input tensor
    s (Tuple[int], optional): Signal size in the transformed dimensions.
        If given, each dimension ``dim[i]`` will either be zero-padded or
        trimmed to the length ``s[i]`` before computing the Hermitian IFFT.
        If a length ``-1`` is specified, no padding is done in that dimension.
        Default: ``s = [input.size(d) for d in dim]``
    dim (Tuple[int], optional): Dimensions to be transformed.
        Default: last two dimensions.
    norm (str, optional): Normalization mode. For the backward transform
        (:func:`~torch.fft.ihfft2`), these correspond to:
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 1028-1051 / 第 1028-1051 行
````python
        * ``"forward"`` - no normalization
        * ``"backward"`` - normalize by ``1/n``
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the Hermitian IFFT orthonormal)

        Where ``n = prod(s)`` is the logical IFFT size.
        Calling the forward transform (:func:`~torch.fft.hfft2`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.ihfft2`
        the exact inverse.

        Default is ``"backward"`` (normalize by ``1/n``).

Keyword args:
    {out}

Example:

    >>> T = torch.rand(10, 10)
    >>> t = torch.fft.ihfft2(t)
    >>> t.size()
    torch.Size([10, 6])

    Compared against the full output from :func:`~torch.fft.ifft2`, the
    Hermitian time-space signal takes up only half the space.
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 1053-1077 / 第 1053-1077 行
````python
    >>> fftn = torch.fft.ifft2(t)
    >>> torch.allclose(fftn[..., :6], rfftn)
    True

    The discrete Fourier transform is separable, so :func:`~torch.fft.ihfft2`
    here is equivalent to a combination of :func:`~torch.fft.ifft` and
    :func:`~torch.fft.ihfft`:

    >>> two_ffts = torch.fft.ifft(torch.fft.ihfft(t, dim=1), dim=0)
    >>> torch.allclose(t, two_ffts)
    True

""".format(**common_args),
)

hfftn = _add_docstr(
    _fft.fft_hfftn,
    r"""
hfftn(input, s=None, dim=None, norm=None, *, out=None) -> Tensor

Computes the n-dimensional discrete Fourier transform of a Hermitian symmetric
:attr:`input` signal.

:attr:`input` is interpreted as a one-sided Hermitian signal in the time
domain. By the Hermitian property, the Fourier transform will be real-valued.
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1079-1105 / 第 1079-1105 行
````python
Note:
    :func:`~torch.fft.hfftn`/:func:`~torch.fft.ihfftn` are analogous to
    :func:`~torch.fft.rfftn`/:func:`~torch.fft.irfftn`. The real FFT expects
    a real signal in the time-domain and gives Hermitian symmetry in the
    frequency-domain. The Hermitian FFT is the opposite; Hermitian symmetric in
    the time-domain and real-valued in the frequency-domain. For this reason,
    special care needs to be taken with the shape argument :attr:`s`, in the
    same way as with :func:`~torch.fft.irfftn`.

Note:
    Some input frequencies must be real-valued to satisfy the Hermitian
    property. In these cases the imaginary component will be ignored.
    For example, any imaginary component in the zero-frequency term cannot
    be represented in a real output and so will always be ignored.

Note:
    The correct interpretation of the Hermitian input depends on the length of
    the original data, as given by :attr:`s`. This is because each input shape
    could correspond to either an odd or even length signal. By default, the
    signal is assumed to be even length and odd signals will not round-trip
    properly. It is recommended to always pass the signal shape :attr:`s`.

Note:
    Supports torch.half and torch.chalf on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimensions.
    With default arguments, the size of last dimension should be (2^n + 1) as argument
    `s` defaults to even output size = 2 * (last_dim_size - 1)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 1107-1131 / 第 1107-1131 行
````python
Args:
    input (Tensor): the input tensor
    s (Tuple[int], optional): Signal size in the transformed dimensions.
        If given, each dimension ``dim[i]`` will either be zero-padded or
        trimmed to the length ``s[i]`` before computing the real FFT.
        If a length ``-1`` is specified, no padding is done in that dimension.
        Defaults to even output in the last dimension:
        ``s[-1] = 2*(input.size(dim[-1]) - 1)``.
    dim (Tuple[int], optional): Dimensions to be transformed.
        The last dimension must be the half-Hermitian compressed dimension.
        Default: all dimensions, or the last ``len(s)`` dimensions if :attr:`s` is given.
    norm (str, optional): Normalization mode. For the forward transform
        (:func:`~torch.fft.hfftn`), these correspond to:

        * ``"forward"`` - normalize by ``1/n``
        * ``"backward"`` - no normalization
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the Hermitian FFT orthonormal)

        Where ``n = prod(s)`` is the logical FFT size.
        Calling the backward transform (:func:`~torch.fft.ihfftn`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.ihfftn`
        the exact inverse.

        Default is ``"backward"`` (no normalization).
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 1133-1159 / 第 1133-1159 行
````python
Keyword args:
    {out}

Example:

    Starting from a real frequency-space signal, we can generate a
    Hermitian-symmetric time-domain signal:
    >>> T = torch.rand(10, 9)
    >>> t = torch.fft.ihfftn(T)

    Without specifying the output length to :func:`~torch.fft.hfftn`, the
    output will not round-trip properly because the input is odd-length in the
    last dimension:

    >>> torch.fft.hfftn(t).size()
    torch.Size([10, 10])

    So, it is recommended to always pass the signal shape :attr:`s`.

    >>> roundtrip = torch.fft.hfftn(t, T.size())
    >>> roundtrip.size()
    torch.Size([10, 9])
    >>> torch.allclose(roundtrip, T)
    True

""".format(**common_args),
)
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1161-1188 / 第 1161-1188 行
````python
ihfftn = _add_docstr(
    _fft.fft_ihfftn,
    r"""
ihfftn(input, s=None, dim=None, norm=None, *, out=None) -> Tensor

Computes the N-dimensional inverse discrete Fourier transform of real :attr:`input`.

:attr:`input` must be a real-valued signal, interpreted in the Fourier domain.
The n-dimensional IFFT of a real signal is Hermitian-symmetric,
``X[i, j, ...] = conj(X[-i, -j, ...])``. :func:`~torch.fft.ihfftn` represents
this in the one-sided form where only the positive frequencies below the
Nyquist frequency are included in the last signal dimension. To compute the
full output, use :func:`~torch.fft.ifftn`.

Note:
    Supports torch.half on CUDA with GPU Architecture SM53 or greater.
    However it only supports powers of 2 signal length in every transformed dimensions.

Args:
    input (Tensor): the input tensor
    s (Tuple[int], optional): Signal size in the transformed dimensions.
        If given, each dimension ``dim[i]`` will either be zero-padded or
        trimmed to the length ``s[i]`` before computing the Hermitian IFFT.
        If a length ``-1`` is specified, no padding is done in that dimension.
        Default: ``s = [input.size(d) for d in dim]``
    dim (Tuple[int], optional): Dimensions to be transformed.
        Default: all dimensions, or the last ``len(s)`` dimensions if :attr:`s` is given.
    norm (str, optional): Normalization mode. For the backward transform
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 1189-1214 / 第 1189-1214 行
````python
        (:func:`~torch.fft.ihfftn`), these correspond to:

        * ``"forward"`` - no normalization
        * ``"backward"`` - normalize by ``1/n``
        * ``"ortho"`` - normalize by ``1/sqrt(n)`` (making the Hermitian IFFT orthonormal)

        Where ``n = prod(s)`` is the logical IFFT size.
        Calling the forward transform (:func:`~torch.fft.hfftn`) with the same
        normalization mode will apply an overall normalization of ``1/n`` between
        the two transforms. This is required to make :func:`~torch.fft.ihfftn`
        the exact inverse.

        Default is ``"backward"`` (normalize by ``1/n``).

Keyword args:
    {out}

Example:

    >>> T = torch.rand(10, 10)
    >>> ihfftn = torch.fft.ihfftn(T)
    >>> ihfftn.size()
    torch.Size([10, 6])

    Compared against the full output from :func:`~torch.fft.ifftn`, we have all
    elements up to the Nyquist frequency.
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。

### Lines 1216-1236 / 第 1216-1236 行
````python
    >>> ifftn = torch.fft.ifftn(t)
    >>> torch.allclose(ifftn[..., :6], ihfftn)
    True

    The discrete Fourier transform is separable, so :func:`~torch.fft.ihfftn`
    here is equivalent to a combination of :func:`~torch.fft.ihfft` and
    :func:`~torch.fft.ifft`:

    >>> two_iffts = torch.fft.ifft(torch.fft.ihfft(t, dim=1), dim=0)
    >>> torch.allclose(ihfftn, two_iffts)
    True

""".format(**common_args),
)

fftfreq = _add_docstr(
    _fft.fft_fftfreq,
    r"""
fftfreq(n, d=1.0, *, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False) -> Tensor

Computes the discrete Fourier Transform sample frequencies for a signal of size :attr:`n`.
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 1238-1264 / 第 1238-1264 行
````python
Note:
    By convention, :func:`~torch.fft.fft` returns positive frequency terms
    first, followed by the negative frequencies in reverse order, so that
    ``f[-i]`` for all :math:`0 < i \leq n/2`` in Python gives the negative
    frequency terms. For an FFT of length :attr:`n` and with inputs spaced in
    length unit :attr:`d`, the frequencies are::

        f = [0, 1, ..., (n - 1) // 2, -(n // 2), ..., -1] / (d * n)

Note:
    For even lengths, the Nyquist frequency at ``f[n/2]`` can be thought of as
    either negative or positive. :func:`~torch.fft.fftfreq` follows NumPy's
    convention of taking it to be negative.

Args:
    n (int): the FFT length
    d (float, optional): The sampling length scale.
        The spacing between individual samples of the FFT input.
        The default assumes unit spacing, dividing that result by the actual
        spacing gives the result in physical frequency units.

Keyword Args:
    {out}
    {dtype}
    {layout}
    {device}
    {requires_grad}
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 1266-1292 / 第 1266-1292 行
````python
Example:

    >>> torch.fft.fftfreq(5)
    tensor([ 0.0000,  0.2000,  0.4000, -0.4000, -0.2000])

    For even input, we can see the Nyquist frequency at ``f[2]`` is given as
    negative:

    >>> torch.fft.fftfreq(4)
    tensor([ 0.0000,  0.2500, -0.5000, -0.2500])

""".format(**factory_common_args),
)

rfftfreq = _add_docstr(
    _fft.fft_rfftfreq,
    r"""
rfftfreq(n, d=1.0, *, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False) -> Tensor

Computes the sample frequencies for :func:`~torch.fft.rfft` with a signal of size :attr:`n`.

Note:
    :func:`~torch.fft.rfft` returns Hermitian one-sided output, so only the
    positive frequency terms are returned. For a real FFT of length :attr:`n`
    and with inputs spaced in length unit :attr:`d`, the frequencies are::

        f = torch.arange((n + 1) // 2) / (d * n)
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 1294-1319 / 第 1294-1319 行
````python
Note:
    For even lengths, the Nyquist frequency at ``f[n/2]`` can be thought of as
    either negative or positive. Unlike :func:`~torch.fft.fftfreq`,
    :func:`~torch.fft.rfftfreq` always returns it as positive.

Args:
    n (int): the real FFT length
    d (float, optional): The sampling length scale.
        The spacing between individual samples of the FFT input.
        The default assumes unit spacing, dividing that result by the actual
        spacing gives the result in physical frequency units.

Keyword Args:
    {out}
    {dtype}
    {layout}
    {device}
    {requires_grad}

Example:

    >>> torch.fft.rfftfreq(5)
    tensor([0.0000, 0.2000, 0.4000])

    >>> torch.fft.rfftfreq(4)
    tensor([0.0000, 0.2500, 0.5000])
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1321-1346 / 第 1321-1346 行
````python
    Compared to the output from :func:`~torch.fft.fftfreq`, we see that the
    Nyquist frequency at ``f[2]`` has changed sign:
    >>> torch.fft.fftfreq(4)
    tensor([ 0.0000,  0.2500, -0.5000, -0.2500])

""".format(**factory_common_args),
)

fftshift = _add_docstr(
    _fft.fft_fftshift,
    r"""
fftshift(input, dim=None) -> Tensor

Reorders n-dimensional FFT data, as provided by :func:`~torch.fft.fftn`, to have
negative frequency terms first.

This performs a periodic shift of n-dimensional data such that the origin
``(0, ..., 0)`` is moved to the center of the tensor. Specifically, to
``input.shape[dim] // 2`` in each selected dimension.

Note:
    By convention, the FFT returns positive frequency terms first, followed by
    the negative frequencies in reverse order, so that ``f[-i]`` for all
    :math:`0 < i \leq n/2` in Python gives the negative frequency terms.
    :func:`~torch.fft.fftshift` rearranges all frequencies into ascending order
    from negative to positive with the zero-frequency term in the center.
````
- **EN**: Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 1348-1373 / 第 1348-1373 行
````python
Note:
    For even lengths, the Nyquist frequency at ``f[n/2]`` can be thought of as
    either negative or positive. :func:`~torch.fft.fftshift` always puts the
    Nyquist term at the 0-index. This is the same convention used by
    :func:`~torch.fft.fftfreq`.

Args:
    input (Tensor): the tensor in FFT order
    dim (int, Tuple[int], optional): The dimensions to rearrange.
        Only dimensions specified here will be rearranged, any other dimensions
        will be left in their original order.
        Default: All dimensions of :attr:`input`.

Example:

    >>> f = torch.fft.fftfreq(4)
    >>> f
    tensor([ 0.0000,  0.2500, -0.5000, -0.2500])

    >>> torch.fft.fftshift(f)
    tensor([-0.5000, -0.2500,  0.0000,  0.2500])

    Also notice that the Nyquist frequency term at ``f[2]`` was moved to the
    beginning of the tensor.

    This also works for multi-dimensional transforms:
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 1375-1400 / 第 1375-1400 行
````python
    >>> x = torch.fft.fftfreq(5, d=1/5) + 0.1 * torch.fft.fftfreq(5, d=1/5).unsqueeze(1)
    >>> x
    tensor([[ 0.0000,  1.0000,  2.0000, -2.0000, -1.0000],
            [ 0.1000,  1.1000,  2.1000, -1.9000, -0.9000],
            [ 0.2000,  1.2000,  2.2000, -1.8000, -0.8000],
            [-0.2000,  0.8000,  1.8000, -2.2000, -1.2000],
            [-0.1000,  0.9000,  1.9000, -2.1000, -1.1000]])

    >>> torch.fft.fftshift(x)
    tensor([[-2.2000, -1.2000, -0.2000,  0.8000,  1.8000],
            [-2.1000, -1.1000, -0.1000,  0.9000,  1.9000],
            [-2.0000, -1.0000,  0.0000,  1.0000,  2.0000],
            [-1.9000, -0.9000,  0.1000,  1.1000,  2.1000],
            [-1.8000, -0.8000,  0.2000,  1.2000,  2.2000]])

    :func:`~torch.fft.fftshift` can also be useful for spatial data. If our
    data is defined on a centered grid (``[-(N//2), (N-1)//2]``) then we can
    use the standard FFT defined on an uncentered grid (``[0, N)``) by first
    applying an :func:`~torch.fft.ifftshift`.

    >>> x_centered = torch.arange(-5, 5)
    >>> x_uncentered = torch.fft.ifftshift(x_centered)
    >>> fft_uncentered = torch.fft.fft(x_uncentered)

    Similarly, we can convert the frequency domain components to centered
    convention by applying :func:`~torch.fft.fftshift`.
````
- **EN**: Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 1402-1428 / 第 1402-1428 行
````python
    >>> fft_centered = torch.fft.fftshift(fft_uncentered)

    The inverse transform, from centered Fourier space back to centered spatial
    data, can be performed by applying the inverse shifts in reverse order:

    >>> x_centered_2 = torch.fft.fftshift(torch.fft.ifft(torch.fft.ifftshift(fft_centered)))
    >>> torch.testing.assert_close(x_centered.to(torch.complex64), x_centered_2, check_stride=False)


""",
)

ifftshift = _add_docstr(
    _fft.fft_ifftshift,
    r"""
ifftshift(input, dim=None) -> Tensor

Inverse of :func:`~torch.fft.fftshift`.

Args:
    input (Tensor): the tensor in FFT order
    dim (int, Tuple[int], optional): The dimensions to rearrange.
        Only dimensions specified here will be rearranged, any other dimensions
        will be left in their original order.
        Default: All dimensions of :attr:`input`.

Example:
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

### Lines 1430-1442 / 第 1430-1442 行
````python
    >>> f = torch.fft.fftfreq(5)
    >>> f
    tensor([ 0.0000,  0.2000,  0.4000, -0.4000, -0.2000])

    A round-trip through :func:`~torch.fft.fftshift` and
    :func:`~torch.fft.ifftshift` gives the same result:

    >>> shifted = torch.fft.fftshift(f)
    >>> torch.fft.ifftshift(shifted)
    tensor([ 0.0000,  0.2000,  0.4000, -0.4000, -0.2000])

""",
)
````
- **EN**: This chunk contributes a focused piece of Python-side wiring for the surrounding torch subsystem.
- **CN**: 这一段为周边 torch 子系统补上了一块聚焦的 Python 侧连接逻辑。

## Key Concepts / 关键概念

- **Spectral transforms**
  - EN: Groups FFT-related namespace functions and their Python-facing dispatch surface.
  - CN: 组织 FFT 相关的命名空间函数及其面向 Python 的分发表面。
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
