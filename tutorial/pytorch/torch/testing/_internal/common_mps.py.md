# common_mps.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/common_mps.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for common mps, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 common mps 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9
```python
import unittest
from collections.abc import Sequence

import torch

from .common_utils import MACOS_VERSION
from .opinfo.core import DecorateInfo, OpInfo


```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `.common_utils`, `.opinfo.core`; external imports: `unittest`, `collections.abc`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `.common_utils`, `.opinfo.core`；外部导入：`unittest`, `collections.abc`。

### Lines 10-18
```python
if torch.backends.mps.is_available():

    def mps_ops_modifier(
        ops: Sequence[OpInfo],
        device_type: str = "mps",
        xfail_exclusion: list[str] | None = None,
        sparse: bool = False,
    ) -> Sequence[OpInfo]:
        if xfail_exclusion is None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `mps_ops_modifier`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`mps_ops_modifier`。

### Lines 19-36
```python
            xfail_exclusion = []

        # Supported complex OPS
        SUPPORTED_COMPLEX_OPS = {
            "__radd__",
            "__rmul__",
            "__rsub__",
            "__getitem__",
            "_unsafe_masked_index",
            "_unsafe_masked_index_put_accumulate",
            "abs",
            "add",
            "addbmm",
            "alias_copy",
            "argwhere",
            "atleast_1d",
            "atleast_2d",
            "atleast_3d",
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 37-54
```python
            "as_strided",
            "as_strided_copy",
            "as_strided_scatter",
            "asin",
            "asinh",
            "acos",
            "atan",
            "baddbmm",
            "block_diag",
            "broadcast_tensors",
            "broadcast_to",
            "chalf",
            "cfloat",
            "chunk",
            "clone",
            "conj",
            "conj_physical",
            "contiguous",
```
- EN: This block handles tensor metadata or sample values; protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 55-72
```python
            "cos",
            "cosh",
            "cross",
            "cumsum",
            "cumprod",
            "cumulative_trapezoid",
            "diag",
            "diag_embed",
            "diagflat",
            "diagonal",
            "diagonal_copy",
            "diagonal_scatter",
            "dist",
            "divno_rounding_mode",
            "dsplit",
            "empty",
            "empty_permuted",
            "empty_strided",
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 73-90
```python
            "exp",
            "expm1",
            "exp2",
            "expand",
            "expand_as",
            "expand_copy",
            "gather",
            "flatten",
            "fill",
            "full",
            "full_like",
            "H",
            "hsplit",
            "imag",
            "index_add",
            "index_copy",
            "index_fill",
            "index_select",
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 91-108
```python
            "index_put",
            "isfinite",
            "isinf",
            "isreal",
            "istft",
            "item",
            "kron",
            "linalg.cross",
            "linalg.diagonal",
            "linalg.householder_product",
            "linalg.svd",
            "linalg.vander",
            "linalg.vecdot",
            "linalg.vector_norm",
            "log10",
            "log1p",
            "log2",
            "log",
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 109-126
```python
            "logaddexp",
            "logaddexp2",
            "logcumsumexp",
            "mH",
            "mT",
            "masked_fill",
            "masked_scatter",
            "masked_select",
            "meshgridlist_of_tensors",
            "meshgridvariadic_tensors",
            "movedim",
            "mul",
            "narrow",
            "narrow_copy",
            "neg",
            "new_full",
            "new_ones",
            "new_zeros",
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 127-144
```python
            "nn.functional.conv1d",
            "nn.functional.conv2d",
            "nn.functional.conv_transpose1d",
            "nn.functional.conv_transpose2d",
            "nn.functional.conv_transpose3d",
            "nn.functional.feature_alpha_dropoutwithout_train",
            "nn.functional.l1_loss",
            "nn.functional.linear",
            "nn.functional.normalize",
            "nn.functional.padcircular",
            "nn.functional.pairwise_distance",
            "nn.functional.softminwith_dtype",
            "nn.functional.softsign",
            "nn.functional.tanhshrink",
            "nn.functional.triplet_margin_loss",
            "nn.functional.triplet_margin_with_distance_loss",
            "nn.functional.unfold",
            "nonzero",
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 145-162
```python
            "nonzero_static",
            "norm",
            "normfro",
            "norminf",
            "ones",
            "ones_like",
            "outer",
            "permute",
            "permute_copy",
            "positive",
            "randn",
            "ravel",
            "real",
            "repeat",
            "repeat_interleave",
            "reshape_as",
            "reshape",
            "resolve_conj",
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 163-180
```python
            "resolve_neg",
            "rsqrt",
            "rsub",
            "scalar_tensor",
            "scatter",
            "scatter_add",
            "select",
            "sgn",
            "sigmoid",
            "sin",
            "sinc",
            "sinh",
            "slice",
            "softmaxwith_dtype",
            "special.spherical_bessel_j0",
            "special.entr",
            "special.xlog1py",
            "special.zeta",
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 181-198
```python
            "split",
            "split_with_sizes",
            "split_with_sizes_copy",
            "splitlist_args",
            "sqrt",
            "squeeze",
            "squeeze_copy",
            "squeezemultiple",
            "sub",
            "svd",
            "t",
            "t_copy",
            "take_along_dim",
            "tanh",
            "tan",
            "tensor_split",
            "tile",
            "transpose",
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 199-216
```python
            "transpose_copy",
            "tril",
            "triu",
            "true_divide",
            "T",
            "unbind",
            "unbind_copy",
            "unflatten",
            "unfold",
            "unfold_copy",
            "unsafe_chunk",
            "unsafe_split",
            "unsqueeze",
            "unsqueeze_copy",
            "view_as",
            "view_as_real",
            "view",
            "view_copy",
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 217-234
```python
            "vsplit",
            "zero_",
            "zeros",
            "zeros_like",
            "__rdiv__",
            "__rmatmul__",
            "_chunk_cat",
            "acosh",
            "all",
            "allclose",
            "angle",
            "any",
            "addcdiv",
            "addcmul",
            "addmmdecomposed",
            "addmv",
            "atanh",
            "bfloat16",
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 235-252
```python
            "bmm",
            "bool",
            "cartesian_prod",
            "cat",
            "char",
            "column_stack",
            "combinations",
            "corrcoef",
            "constant_pad_nd",
            "cov",
            "count_nonzero",
            "diff",
            "div",
            "dot",
            "dstack",
            "einsum",
            "eq",
            "equal",
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 253-270
```python
            "eye",
            "fft.fft",
            "fft.fft2",
            "fft.fftn",
            "fft.fftshift",
            "fft.ifft",
            "fft.ifft2",
            "fft.ifftn",
            "fft.ifftshift",
            "fft.irfftn",
            "fft.irfft2",
            "fft.irfft",
            "fft.hfftn",
            "fft.hfft2",
            "fft.hfft",
            "flip",
            "fliplr",
            "flipud",
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 271-288
```python
            "float",
            "gradient",
            "half",
            "hstack",
            "inner",
            "int",
            "isclose",
            "isnan",
            "ldexp",
            "lerp",
            "linalg.multi_dot",
            "linalg.pinv",
            "linspace",
            "linspacetensor_overload",
            "logical_and",
            "logical_not",
            "logical_or",
            "logical_xor",
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 289-306
```python
            "logsumexp",
            "long",
            "masked.cumsum",
            "masked.cumprod",
            "masked.mean",
            "masked.normalize",
            "masked.prod",
            "masked.std",
            "masked.sum",
            "masked.var",
            "masked.logsumexp",
            "matmul",
            "mean",
            "mm",
            "mv",
            "ne",
            "nn.functional.padconstant",
            "nn.functional.padreflect",
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 307-324
```python
            "nn.functional.padreplicate",
            "nn.functional.pixel_shuffle",
            "nn.functional.pixel_unshuffle",
            "nn.functional.rms_norm",
            "pinverse",
            "prod",
            "reciprocal",
            "roll",
            "rot90",
            "short",
            "square",
            "stack",
            "stft",
            "sum",
            "sum_to_size",
            "tensordot",
            "trace",
            "trapz",
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 325-336
```python
            "trapezoid",
            "vdot",
            "vstack",
            "where",
            "byte",
        }

        MACOS_BEFORE_14_4_XFAILLIST = {
            # These ops work fine in 14.4 but fail in 14.2 or 13.x
            "fft.hfft2": [torch.complex64],
        }

```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 337-354
```python
        # Those ops are not expected to work
        UNIMPLEMENTED_XFAILLIST: dict[str, list | None] = {
            # Failures due to lack of op implementation on MPS backend
            "logspace": None,
            "logspacetensor_overload": None,
            "linalg.eig": None,
            "linalg.eigvals": None,
            "put": None,
            "frexp": None,
            "geqrf": None,
            "hash_tensor": None,
            "heaviside": None,
            # "kthvalue": None,
            "lcm": None,
            "linalg.cond": None,
            "linalg.eigh": None,
            "linalg.eigvalsh": None,
            "linalg.ldl_factor": None,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物。关键符号：无明显局部符号。

### Lines 355-372
```python
            "linalg.ldl_factor_ex": None,
            "linalg.ldl_solve": None,
            "linalg.lstsq": None,
            "linalg.lstsqgrad_oriented": None,
            "linalg.matrix_norm": [torch.float32],
            "linalg.norm": [torch.float32],
            "linalg.normsubgradients_at_zero": [torch.float32],
            "linalg.svdvals": None,
            "masked.median": None,
            "matrix_exp": None,
            "max_pool2d_with_indices_backward": [
                torch.int8,
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
            ],
            "median": [torch.bool],
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 373-390
```python
            "mode": None,
            "nanmedian": [torch.bool],
            "native_batch_norm": [
                torch.uint8,
                torch.bool,
                torch.int8,
                torch.int16,
                torch.int32,
            ],
            "normnuc": None,
            "nn.functional.avg_pool1d": [
                torch.int16,
                torch.int32,
                torch.uint8,
                torch.bool,
                torch.int8,
            ],
            "nn.functional.avg_pool2d": [
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 391-408
```python
                torch.int16,
                torch.int32,
                torch.uint8,
                torch.bool,
                torch.int8,
            ],
            "nn.functional.avg_pool3d": [
                torch.int16,
                torch.int32,
                torch.uint8,
                torch.int8,
            ],
            "nn.functional.batch_norm": [
                torch.uint8,
                torch.bool,
                torch.int8,
                torch.int16,
                torch.int32,
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 409-426
```python
            ],
            "nn.functional.fractional_max_pool2d": None,
            "nn.functional.fractional_max_pool3d": None,
            "nn.functional.group_norm": [torch.int16, torch.int32],
            "nn.functional.glu": [
                torch.int32,
                torch.uint8,
                torch.bool,
                torch.int8,
                torch.int16,
            ],
            "nn.functional.huber_loss": [
                torch.uint8,
                torch.bool,
                torch.int8,
                torch.int16,
                torch.int32,
            ],
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 427-444
```python
            "nn.functional.adaptive_avg_pool3d": None,
            "nn.functional.adaptive_max_pool1d": [
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
                torch.bool,
                torch.int8,
            ],
            "nn.functional.adaptive_max_pool2d": [
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
                torch.bool,
                torch.int8,
            ],
            "nn.functional.adaptive_max_pool3d": None,
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 445-462
```python
            "nn.functional.interpolatearea": None,
            "nn.functional.interpolatebicubic": [torch.uint8],
            "nn.functional.ctc_loss": None,
            "nn.functional.local_response_norm": [
                torch.int8,
                torch.int16,
                torch.int32,
                torch.uint8,
                torch.bool,
            ],
            "nn.functional.logsigmoid": [
                torch.int16,
                torch.int32,
                torch.uint8,
                torch.bool,
                torch.int8,
            ],
            "nn.functional.max_pool1d": [
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 463-480
```python
                torch.uint8,
                torch.bool,
                torch.int8,
                torch.int16,
                torch.int32,
                torch.int64,
            ],
            "nn.functional.max_pool2d": [torch.bool],
            "nn.functional.max_pool3d": [torch.bool],
            "nn.functional.max_unpool1d": [
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
                torch.bool,
                torch.int8,
            ],
            "nn.functional.max_unpool1dgrad": [
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 481-498
```python
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
                torch.bool,
                torch.int8,
            ],
            "nn.functional.max_unpool2d": [
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
                torch.bool,
                torch.int8,
            ],
            "nn.functional.max_unpool2dgrad": [
                torch.int16,
                torch.int32,
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 499-516
```python
                torch.int64,
                torch.uint8,
                torch.bool,
                torch.int8,
            ],
            "nn.functional.max_unpool3d": [
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
                torch.bool,
                torch.int8,
            ],
            "nn.functional.max_unpool3dgrad": [
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 517-534
```python
                torch.bool,
                torch.int8,
            ],
            "nn.functional.mish": [
                torch.int32,
                torch.uint8,
                torch.bool,
                torch.int8,
                torch.int16,
            ],
            "nn.functional.multi_margin_loss": None,
            "nn.functional.multilabel_margin_loss": [
                torch.int8,
                torch.uint8,
                torch.int32,
                torch.int16,
                torch.float32,
            ],
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 535-552
```python
            "nn.functional.multilabel_soft_margin_loss": [
                torch.int8,
                torch.uint8,
                torch.int32,
                torch.int16,
            ],
            "nn.functional.nll_loss": [
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
                torch.bool,
                torch.int8,
            ],
            "nn.functional.padreplicate_negative": [torch.bool],
            "nn.functional.pdist": None,
            "nn.functional.relu": [torch.bool],
            "nn.functional.rrelu": None,
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 553-570
```python
            "nn.functional.silu": [
                torch.int16,
                torch.int32,
                torch.uint8,
                torch.int8,
            ],
            "nn.functional.softplus": [
                torch.int32,
                torch.uint8,
                torch.bool,
                torch.int8,
                torch.int16,
            ],
            "nn.functional.norm": None,
            "ormqr": None,
            "rounddecimals_0": [
                torch.uint8,
                torch.int8,
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 571-579
```python
                torch.int64,
                torch.int32,
                torch.int16,
            ],
            "scatter_reduceamax": [torch.int32, torch.int64]
            if MACOS_VERSION < 15.0
            else [torch.int64],
            "scatter_reduceamin": [torch.int32, torch.int64]
            if MACOS_VERSION < 15.0
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 580-597
```python
            else [torch.int64],
            "scatter_reducemean": [torch.bool],
            "segment_reduce": None,
            "_segment.reduce": None,
            "segment.reduce": None,
            "segment_reduce_offsets": None,
            "_segment_reduce_offsets": None,
            "_segment_reduce_lengths": None,
            "_segment_reducelengths": None,
            "_segment_reduceoffsets": None,
            "sparse.mm": None,
            "sparse.sampled_addmm": None,
            "sparse.mmreduce": None,
            "special.airy_ai": None,
            "special.laguerre_polynomial_l": None,
            "special.legendre_polynomial_p": None,
            "special.log_ndtr": None,
            "special.ndtri": None,
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 598-615
```python
            "stft": [torch.float16, torch.bfloat16],
            "svd_lowrank": None,
            "symeig": None,
            "take": None,
            "to": None,
            "var_meanunbiased": [
                torch.uint8,
                torch.int8,
                torch.int32,
                torch.int16,
                torch.bool,
            ],
            "var_mean": [torch.uint8, torch.int8, torch.int32, torch.int16, torch.bool],
            "std_mean": [torch.uint8, torch.int8, torch.int32, torch.int16, torch.bool],
            "std_meanunbiased": [
                torch.uint8,
                torch.int8,
                torch.int32,
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 616-633
```python
                torch.int16,
                torch.bool,
            ],
            "segment_reduce_": None,
            "_upsample_bilinear2d_aa": [torch.uint8],  # uint8 is for CPU only
            "_upsample_bicubic2d_aa": [torch.uint8],  # uint8 is for CPU only
            "cdouble": None,
            "double": None,
            "log_softmaxwith_dtype": [
                torch.uint8,
                torch.int8,
                torch.int32,
                torch.int16,
                torch.int64,
                torch.float32,
            ],
            "float_power": None,
            "linalg.matrix_rankhermitian": None,
```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 634-651
```python
            "linalg.pinvhermitian": None,
            # MPS: input sizes must be divisible by output sizes
            "nn.functional.adaptive_avg_pool1d": None,
            "nn.functional.adaptive_avg_pool2d": None,
            # Convolution for integral types is not supported on MPS
            "nn.functional.conv1d": [torch.int64],
            "nn.functional.conv2d": [torch.int64],
            "nn.functional.conv3d": [torch.int64],
            "nn.functional.conv_transpose1d": [torch.int64],
            "nn.functional.conv_transpose2d": [torch.int64, torch.bfloat16],
            "nn.functional.conv_transpose3d": [
                torch.int64,
                torch.bfloat16,
                torch.float16,
            ],
            # Unsupported dtypes
            # GEMM on MPS is not supported for integral types
            "nn.functional.linear": [
```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 652-669
```python
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
                torch.int8,
            ],
            "mat": [torch.int16, torch.int32, torch.int64, torch.uint8, torch.int8],
            # returned output on CPU is float64
            "bincount": [
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
                torch.int8,
            ],
        }
        UNIMPLEMENTED_XFAILLIST_SPARSE: dict[str, list | None] = {
            "logspace": None,
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 670-683
```python
            "logspacetensor_overload": None,
            "linalg.eig": None,
            "linalg.eigvals": None,
            "put": None,
        }

        if MACOS_VERSION < 15.0:
            UNIMPLEMENTED_XFAILLIST.update(
                {
                    "quantile": None,
                    "nanquantile": None,
                }
            )
        if sparse:
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 684-701
```python
            UNIMPLEMENTED_XFAILLIST.update(UNIMPLEMENTED_XFAILLIST_SPARSE)

        UNDEFINED_XFAILLIST: dict[str, list | None] = {
            # Top 60 operators
            # topk fails with duplicate indices
            "topk": [
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
                torch.int8,
            ],
            # Failures due to random output that they generate using
            # Philox engine causing mismatch with CPU results
            "multinomial": [
                torch.float16,
                torch.float32,
                torch.bfloat16,
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 702-719
```python
            ],  # random results
            "uniform": [torch.float16, torch.float32, torch.bfloat16],
            "rand_like": [torch.float16, torch.float32, torch.bfloat16],
            "randint": None,
            "randint_like": None,
            "randn": None,
            "randn_like": None,
            "bernoulli": [torch.float16, torch.float32, torch.bfloat16],
            "exponential": [torch.float16, torch.float32, torch.bfloat16],
            "log_normal": [torch.float16, torch.float32, torch.bfloat16],
            "cauchy": [torch.float16, torch.float32, torch.bfloat16],
            "geometric": [
                torch.float16,
                torch.float32,
                torch.bfloat16,
                torch.int32,
                torch.int16,
                torch.int64,
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 720-737
```python
                torch.int8,
                torch.uint8,
            ],
            "nn.functional.feature_alpha_dropoutwith_train": [
                torch.float16,
                torch.float32,
                torch.bfloat16,
            ],
            "normal": [torch.float16, torch.float32, torch.bfloat16],
            "normalin_place": [torch.float16, torch.float32, torch.bfloat16],
            "normalnumber_mean": [torch.float16, torch.float32, torch.bfloat16],
            "nn.functional.alpha_dropout": [
                torch.float16,
                torch.float32,
                torch.bfloat16,
            ],
            "nn.functional.dropout": [
                torch.float16,
```
- EN: This block implements local helper logic for common mps. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common mps 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 738-755
```python
                torch.float32,
                torch.bfloat16,
                torch.complex64,
            ],
            "nn.functional.dropout2d": [torch.float16, torch.float32, torch.bfloat16],
            "nn.functional.dropout3d": [torch.float16, torch.float32, torch.bfloat16],
            # See https://github.com/pytorch/pytorch/issues/111479
            "nn.functional.multi_head_attention_forward": [
                torch.float32,
                torch.float16,
                torch.bfloat16,
            ],
            # zero to negative integer powers are undefined
            "__rpow__": [torch.int8, torch.int16, torch.int32, torch.int64],
            "resize_": [torch.float16, torch.float32, torch.bfloat16],
            "resize_as_": [torch.float16, torch.float32, torch.bfloat16],
            # CPU Errors:
            "addr": [
```
- EN: This block reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 756-773
```python
                torch.bool,
                torch.int16,
                torch.int32,
                torch.int64,
                torch.uint8,
                torch.int8,
            ],  # "addmv_impl_cpu" not implemented for 'Half'
            "as_stridedpartial_views": None,  # cpu result off, showing random values
            # random results
            # mps vs cpu:
            # Mismatched elements: 40 / 96 (41.7%)
            # Greatest absolute difference: 17.892311096191406 at index (1, 0, 2) (up to 1e-05 allowed)
            # Greatest relative difference: inf at index (1, 0, 0) (up to 1.3e-06 allowed)
            # cuda(2.0.0.dev20230301+cu117) vs cpu:
            # Mismatched elements: 56 / 96 (58.3%)
            # Greatest absolute difference: 17.892311096191406 at index (1, 0, 2) (up to 1e-05 allowed)
            # Greatest relative difference: inf at index (1, 0, 0) (up to 1.3e-06 allowed)
            "nn.functional.scaled_dot_product_attention": [
```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 774-791
```python
                torch.float32,
                torch.float16,
                torch.bfloat16,
            ],
        }

        ON_MPS_XFAILLIST: dict[str, list | None] = {
            # Failures due to lack of implementation of downstream functions on MPS backend
            # TODO: remove these once downstream function 'aten::_linalg_svd.U' have been implemented
            "linalg.matrix_rank": None,
            # Exception: Caused by `torch.arange(-8.001, -4.0, dtype=torch.uint8, device="mps")`
            "arange": [torch.uint8],
            # Failure due to precision issue for fp16
            # on both cpu and mps there are test cases that might produce inf result
            # 'nn.functional.pairwise_distance': [torch.float16],
            # test blow pass on macOS 12 as it falls back to cpu
            # Argsort case using duplicate indices (undefined behaviour):
            #  - CPU output: tensor([2546, 6917, 3181,  ..., 7128, 5133,   30], device='cpu')
```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 792-809
```python
            #  - MPS output: tensor([2546, 6917, 3181,  ..., 7128,   30, 5133], device='mps:0')
            # Elements from index 30 and 5133 are both equal.
            # Since CPU is not using argsort with stable=True, these cases result in undefined behaviour.
            "argsort": [
                torch.float16,
                torch.int8,
                torch.uint8,
                torch.bool,
                torch.bfloat16,
            ],
            # Same issue as `argsort` with duplicate indices. This test checks both the sorted values and the indices.
            # The values of the sorted tensor match the CPU,
            # but in case of the returned indices this results in undefined behaviour.
            "sort": [
                torch.int8,
                torch.uint8,
                torch.bool,
                torch.float16,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 810-827
```python
                torch.bfloat16,
            ],
        }

        EMPTY_OPS_SKIPLIST = {
            # Fill tensors with uninitialized data, causing mismatch with CPU.
            # They occasionally match, thus skipping them.
            # See https://github.com/pytorch/pytorch/issues/100175
            "new_empty": None,
            "new_empty_strided": None,
            "empty_strided": None,
            # CPU: empty is returning all 0's and there is a mismatch with MPS
            # allocation (MacOS 13). According to
            # https://pytorch.org/docs/2.0/generated/torch.empty.html
            "empty": None,
            "empty_like": None,
            "empty_permuted": None,
        }
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 828-838
```python

        SKIPLIST = {
            # Unsupported
            # This doesn't work on M1, but is partially working on M2 with the exception of torch.float16
            "nn.functional.conv3d": None,
            # MPS uses float32 intermediates (opmath_t) while CPU uses native
            # half/bfloat16 precision, causing unbounded divergence.
            # Half precision is covered by test_grid_sampler_3d_half_precision.
            "nn.functional.grid_sample": [torch.float16, torch.bfloat16],
        }

```
- EN: This block reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 839-856
```python
        def addDecorator(op: OpInfo, d: DecorateInfo) -> None:
            if device_type is not None:
                d.device_type = device_type

            op.decorators = op.decorators + (d,)

        for op in ops:
            key = op.name + op.variant_test_name
            addDecorator(
                op,
                DecorateInfo(
                    unittest.expectedFailure,
                    dtypes=[
                        torch.double,
                        torch.cdouble,
                    ],
                ),
            )
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `addDecorator`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`addDecorator`。

### Lines 857-871
```python
            if sparse:
                # Skipped due to test_sparse_zero_dims test in test_sparse.py which allocates empty tensor
                # which leads to unexpected success with it
                addDecorator(
                    op,
                    DecorateInfo(
                        unittest.skip(
                            "Skipped due to MPS not supporting complex128 tensors"
                        ),
                        dtypes=[
                            torch.complex128,
                        ],
                    ),
                )
            if key in EMPTY_OPS_SKIPLIST:
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 872-883
```python
                addDecorator(
                    op,
                    DecorateInfo(
                        unittest.skip("Skipping empty ops."),
                        dtypes=EMPTY_OPS_SKIPLIST[key],
                    ),
                )
            if key in SKIPLIST:
                addDecorator(
                    op, DecorateInfo(unittest.skip("Skipped!"), dtypes=SKIPLIST[key])
                )
            for xfaillist in [
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 884-893
```python
                UNIMPLEMENTED_XFAILLIST,
                UNDEFINED_XFAILLIST,
                ON_MPS_XFAILLIST,
            ]:
                if key in xfaillist and key not in xfail_exclusion:
                    addDecorator(
                        op,
                        DecorateInfo(unittest.expectedFailure, dtypes=xfaillist[key]),
                    )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 894-906
```python
            if (
                key in MACOS_BEFORE_14_4_XFAILLIST
                and key not in xfail_exclusion
                and (MACOS_VERSION < 14.4)
            ):
                addDecorator(
                    op,
                    DecorateInfo(
                        unittest.expectedFailure,
                        dtypes=MACOS_BEFORE_14_4_XFAILLIST[key],
                    ),
                )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 907-916
```python
            # If ops is not supported for complex types, expect it to fail
            if key not in SUPPORTED_COMPLEX_OPS:
                addDecorator(
                    op,
                    DecorateInfo(
                        unittest.expectedFailure,
                        dtypes=[torch.complex32, torch.complex64],
                    ),
                )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 917-934
```python
        return ops

    def mps_ops_grad_modifier(ops: Sequence[OpInfo]) -> Sequence[OpInfo]:
        XFAILLIST_GRAD = {
            # Unimplemented ops
            "_segment_reduce": [torch.float16, torch.float32],
            "_chunk_cat": [torch.float16, torch.float32],
            "_upsample_bilinear2d_aa": None,  # `_upsample_bilinear2d_aa_backward_out` not implemented for MPS
            "_upsample_bicubic2d_aa": None,  # `_upsample_bilinear2d_aa_backward_out` not implemented for MPS
            "sparse.mmreduce": [torch.float32],  # csr not supported
            "linalg.householder_product": None,
            "unique_consecutive": [torch.float16, torch.float32],
            "scalar_tensor": [torch.float16, torch.float32],
            "cdist": None,
            "masked.scatter": [torch.float16, torch.float32],
            "igamma": None,  # currently not supported for any device
            "igammac": None,  # currently not supported for any device
            "special.i1": [torch.float16],  # "i1_backward" not implemented for 'Half'
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `mps_ops_grad_modifier`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`mps_ops_grad_modifier`。

### Lines 935-952
```python
            "special.i1e": [torch.float16],  # "i1e_backward" not implemented for 'Half'
            # Correctness issues
            # Same issue as `argsort` and `sort` with duplicate elements (undefined behaviour).
            # Forward pass is passing since `msort` doesn't return the indices, just the values, which match the CPU.
            # On the backward pass for `sort` both are used (values and indices), thus resulting in a issmatch between CPU and MPS.
            # Running `msort` with stable `sort` passes.
            "msort": [torch.float16],
            # Random output
            "exponential": [torch.float16, torch.float32],
            "log_normal": [torch.float16, torch.float32],
            "cauchy": [torch.float16, torch.float32],
            "geometric": [torch.float16, torch.float32],
            # CPU errors
            # derivative for zeta is not implemented
            "special.zeta": None,
            # derivative for aten::floor_divide is not implemented on CPU
            "floor_divide": [torch.float16, torch.float32],
            # derivative for aten::_histogramdd_from_bin_cts is not implemented on CPU
```
- EN: This block reports or normalizes error conditions; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 953-970
```python
            "histogramdd": [torch.float16, torch.float32],
            # derivative for aten::histogram is not implemented
            "histogram": [torch.float16, torch.float32],
            # 'bool' object is not iterable
            "allclose": [torch.float16, torch.float32],
            "equal": [torch.float16, torch.float32],
            # 'float' object is not iterable
            "item": [torch.float16, torch.float32],
            # cpu error: grad requires non-empty inputs
            "randn": [torch.float16, torch.float32],
            "signal.windows.bartlett": [torch.float32],
            "signal.windows.blackman": [torch.float32],
            "signal.windows.cosine": [torch.float32],
            "signal.windows.exponential": [torch.float32],
            "signal.windows.gaussian": [torch.float32],
            "signal.windows.general_cosine": [torch.float32],
            "signal.windows.general_hamming": [torch.float32],
            "signal.windows.hamming": [torch.float32],
```
- EN: This block reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 971-980
```python
            "signal.windows.hann": [torch.float32],
            "signal.windows.kaiser": [torch.float32],
            "signal.windows.nuttall": [torch.float32],
            "eye": [torch.float16, torch.float32],
            # Could not run 'aten::uniform_' with arguments from the 'SparseCPU' backend
            "to_sparse": None,
            # Exception: the derivative for '_unique2' is not implemented.
            "unique": None,
        }

```
- EN: This block bridges to backend-specific execution artifacts; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块桥接到特定后端的执行产物；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 981-991
```python
        SKIPLIST_GRAD = {
            "nn.functional.pairwise_distance": [torch.float16],
            # failed assertion `destination datatype must be fp32'
            "nn.functional.conv1d": [torch.float16],
            "nn.functional.conv2d": [torch.float16],
            "nn.functional.conv3d": [torch.float16],
            "nn.functional.conv_transpose1d": [torch.float16],
            "nn.functional.conv_transpose2d": [torch.float16],
            "nn.functional.conv_transpose3d": [torch.float16],
        }

```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 992-1000
```python
        ON_MPS_XFAILLIST = {
            # Failures due to lack of implementation of downstream functions on MPS backend
            # TODO: remove these once downstream function 'aten::_linalg_svd.U' have been implemented
            "linalg.matrix_rank": None,
            # Exception: Caused by sample input at index 3 on MPS
            "nn.functional.conv3d": [torch.float32],
        }

        def addDecorator(op: OpInfo, d: DecorateInfo) -> None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `addDecorator`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`addDecorator`。

### Lines 1001-1010
```python
            op.decorators = op.decorators + (d,)

        for op in ops:
            key = op.name + op.variant_test_name
            if key in XFAILLIST_GRAD:
                addDecorator(
                    op,
                    DecorateInfo(unittest.expectedFailure, dtypes=XFAILLIST_GRAD[key]),
                )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1011-1021
```python
            if key in SKIPLIST_GRAD:
                addDecorator(op, DecorateInfo(unittest.skip, dtypes=SKIPLIST_GRAD[key]))

            if key in ON_MPS_XFAILLIST:
                addDecorator(
                    op,
                    DecorateInfo(
                        unittest.expectedFailure, dtypes=ON_MPS_XFAILLIST[key]
                    ),
                )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 1022-1039
```python
        return ops

    def mps_ops_error_inputs_modifier(ops: Sequence[OpInfo]) -> Sequence[OpInfo]:
        # Error input samples do not take a dtype argument.
        XFAILLIST = {
            # Exceptions are not raised
            "__rmod__",
            "__rsub__",
            "__rpow__",
            "clamp_max",
            "clamp_min",
            "masked_scatter",
            # MPS does not support tensor dimensions > 16
            "amax",
            "amin",
            "aminmax",
        }

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `mps_ops_error_inputs_modifier`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`mps_ops_error_inputs_modifier`。

### Lines 1040-1048
```python
        def addDecorator(op: OpInfo, d: DecorateInfo) -> None:
            op.decorators = op.decorators + (d,)

        for op in ops:
            key = op.name + op.variant_test_name
            if key in XFAILLIST:
                addDecorator(op, DecorateInfo(unittest.expectedFailure))

        return ops
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `addDecorator`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`addDecorator`。

### Lines 1049-1057
```python
else:

    def mps_ops_modifier(
        ops: Sequence[OpInfo],
        device_type: str = "mps",
        xfail_exclusion: list[str] | None = None,
        sparse: bool = False,
    ) -> Sequence[OpInfo]:
        return ops
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `mps_ops_modifier`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`mps_ops_modifier`。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: Backend lowering — the code bridges higher-level programs to executable backend artifacts.
  CN: 后端降级——代码把更高层程序桥接到可执行的后端产物。
- EN: Concurrency support — the implementation manages coordination, ownership, or thread safety.
  CN: 并发支持——实现会管理协作、所有权或线程安全。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `.common_utils`, `.opinfo.core`
- External imports / 外部导入: `unittest`, `collections.abc`
- Representative symbols / 代表性符号: 无
