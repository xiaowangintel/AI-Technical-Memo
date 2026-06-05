# nested.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/opinfo/definitions/nested.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for nested, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 nested 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17
```python
# mypy: ignore-errors

import math
from copy import copy
from dataclasses import dataclass
from functools import partial

import torch
from torch.fx.experimental.symbolic_shapes import is_nested_int
from torch.testing._internal.common_methods_invocations import op_db
from torch.testing._internal.opinfo.core import (
    BinaryUfuncInfo,
    ReductionOpInfo,
    SampleInput,
    UnaryUfuncInfo,
)
from torch.utils._pytree import tree_flatten, tree_map
```
- EN: This block advances ONNX export translation; manipulates graph-like program structures; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块推进 ONNX 导出翻译流程；操作图状程序结构；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 18-31
```python


@dataclass
class ExtraOpData:
    """
    Contains info on top of the typical OpInfo data that is useful for NJT test generation.

    The process that converts the standard op_db -> an NJT-compatible op_db will attach this
    data onto each associated OpInfo entry.
    """

    # Indicates whether the associated op is a view op
    is_view: bool = False

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `ExtraOpData`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`ExtraOpData`。

### Lines 32-44
```python
    # Specifies the names of any dim-related args that the op takes in. This is useful
    # for NJT tests because there is often asymmetry across the supported set of dims for
    # an op; it may make sense to operate over the batch dim but not the ragged dim, for
    # example. The length of this list should match the number of relevant overloads.
    # Each list item of the outer list should specify dim argnames. Ellipses should be used
    # to indicate multi-dim support for a given overload.
    #
    # For example, squeeze() has both a dim and multi-dim overload, where the argname for
    # each is simply "dim". Its entry should be: [["dim"], ["dim..."]].
    #
    # If no overload of the op accepts dim-related args, this should be None.
    dim_args: list[list[str]] = None

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 45-57
```python
    # Helper function to extract names of dim-related args.
    # Returns: tuple of (single dim argname if available, dim list argname if available)
    # If the op doesn't support dim-related args at all OR this op only has overloads
    # with multiple dim args (e.g. transpose()), then this returns (None, None).
    def get_dim_argnames(self) -> tuple[str | None, str | None]:
        if self.dim_args is None:
            return (None, None)

        # name for the dim arg that supports a single dim
        single_dim_argname = None
        # name for the dim arg that supports a list of dims
        dimlist_argname = None
        for overload in self.dim_args:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_dim_argnames`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_dim_argnames`。

### Lines 58-69
```python
            # only consider overloads with a single dim-related arg
            if len(overload) != 1:
                continue
            if overload[0].endswith("..."):
                dimlist_argname = overload[0].replace("...", "")
                if single_dim_argname is None:
                    single_dim_argname = dimlist_argname
            else:
                single_dim_argname = overload[0]
        return (single_dim_argname, dimlist_argname)


```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 70-93
```python
# Mapping of OpInfo full names -> extra data to tack onto the OpInfo entry for use
# in test generation.
extra_op_data = {
    "_segment_reduce.lengths": ExtraOpData(dim_args=[["axis0"]]),
    "_segment_reduce.offsets": ExtraOpData(dim_args=[["axis0"]]),
    "all": ExtraOpData(dim_args=[["dim"], ["dim..."]]),
    "argmax": ExtraOpData(dim_args=[["dim"]]),
    "argmin": ExtraOpData(dim_args=[["dim"]]),
    "amax": ExtraOpData(dim_args=[["dim..."]]),
    "amin": ExtraOpData(dim_args=[["dim..."]]),
    "any": ExtraOpData(dim_args=[["dim"], ["dim..."]]),
    "argsort": ExtraOpData(dim_args=[["dim"]]),
    "broadcast_to": ExtraOpData(is_view=True),
    "cat": ExtraOpData(dim_args=[["dim"]]),
    "chunk": ExtraOpData(is_view=True, dim_args=[["dim"]]),
    "conj": ExtraOpData(is_view=True),
    "contiguous": ExtraOpData(is_view=True),
    "count_nonzero": ExtraOpData(dim_args=[["dim"], ["dim..."]]),
    "cummax": ExtraOpData(dim_args=[["dim"]]),
    "cummin": ExtraOpData(dim_args=[["dim"]]),
    "cumprod": ExtraOpData(dim_args=[["dim"]]),
    "cumsum": ExtraOpData(dim_args=[["dim"]]),
    "cumulative_trapezoid": ExtraOpData(dim_args=[["dim"]]),
    "diag_embed": ExtraOpData(dim_args=[["dim1", "dim2"]]),
```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 94-117
```python
    "diagonal": ExtraOpData(is_view=True, dim_args=[["dim1", "dim2"]]),
    "diagonal_copy": ExtraOpData(dim_args=[["dim1", "dim2"]]),
    "diagonal_scatter": ExtraOpData(dim_args=[["dim1", "dim2"]]),
    "diff": ExtraOpData(dim_args=[["dim"]]),
    "expand": ExtraOpData(is_view=True),
    "expand_as": ExtraOpData(is_view=True),
    "fft.fft": ExtraOpData(dim_args=[["dim"]]),
    "fft.hfft": ExtraOpData(dim_args=[["dim"]]),
    "fft.ifft": ExtraOpData(dim_args=[["dim"]]),
    "fft.ihfft": ExtraOpData(dim_args=[["dim"]]),
    "fft.irfft": ExtraOpData(dim_args=[["dim"]]),
    "fft.rfft": ExtraOpData(dim_args=[["dim"]]),
    "flatten": ExtraOpData(is_view=True, dim_args=[["start_dim", "end_dim"]]),
    "flip": ExtraOpData(dim_args=[["dims..."]]),
    "gather": ExtraOpData(dim_args=[["dim"]]),
    "hash_tensor": ExtraOpData(dim_args=[["dim..."]]),
    "imag": ExtraOpData(is_view=True),
    "index_add": ExtraOpData(dim_args=[["dim"]]),
    "index_copy": ExtraOpData(dim_args=[["dim"]]),
    "index_fill": ExtraOpData(dim_args=[["dim"]]),
    "index_reduce.amax": ExtraOpData(dim_args=[["dim"]]),
    "index_reduce.amin": ExtraOpData(dim_args=[["dim"]]),
    "index_reduce.mean": ExtraOpData(dim_args=[["dim"]]),
    "index_reduce.prod": ExtraOpData(dim_args=[["dim"]]),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 118-141
```python
    "index_select": ExtraOpData(dim_args=[["dim"]]),
    "kthvalue": ExtraOpData(dim_args=[["dim"]]),
    "linalg.cross": ExtraOpData(dim_args=[["dim"]]),
    "linalg.diagonal": ExtraOpData(is_view=True, dim_args=[["dim1", "dim2"]]),
    "linalg.tensorsolve": ExtraOpData(dim_args=[["dims..."]]),
    "linalg.vecdot": ExtraOpData(dim_args=[["dim"]]),
    "linalg.vector_norm": ExtraOpData(dim_args=[["dim..."]]),
    "log_softmax": ExtraOpData(dim_args=[["dim"]]),
    "logcumsumexp": ExtraOpData(dim_args=[["dim"]]),
    "masked.amax": ExtraOpData(dim_args=[["dim"]]),
    "masked.amin": ExtraOpData(dim_args=[["dim"]]),
    "masked.argmax": ExtraOpData(dim_args=[["dim"]]),
    "masked.argmin": ExtraOpData(dim_args=[["dim"]]),
    "masked.logsumexp": ExtraOpData(dim_args=[["dim"]]),
    "masked.mean": ExtraOpData(dim_args=[["dim"]]),
    "masked.norm": ExtraOpData(dim_args=[["dim"]]),
    "masked.prod": ExtraOpData(dim_args=[["dim"]]),
    "masked.std": ExtraOpData(dim_args=[["dim"]]),
    "masked.sum": ExtraOpData(dim_args=[["dim"]]),
    "masked.var": ExtraOpData(dim_args=[["dim"]]),
    "max.reduction_with_dim": ExtraOpData(dim_args=[["dim"]]),
    "median": ExtraOpData(dim_args=[["dim"]]),
    "mean": ExtraOpData(dim_args=[["dim..."]]),
    "min.reduction_with_dim": ExtraOpData(dim_args=[["dim"]]),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 142-165
```python
    "mode": ExtraOpData(dim_args=[["dim"]]),
    "movedim": ExtraOpData(
        dim_args=[["source", "destination"], ["source...", "destination..."]]
    ),
    "nanmean": ExtraOpData(dim_args=[["dim..."]]),
    "nanmedian": ExtraOpData(dim_args=[["dim"]]),
    "nansum": ExtraOpData(dim_args=[["dim..."]]),
    "narrow": ExtraOpData(is_view=True, dim_args=[["dim"]]),
    "narrow_copy": ExtraOpData(dim_args=[["dim"]]),
    "nn.functional.cosine_similarity": ExtraOpData(dim_args=[["dim"]]),
    "nn.functional.glu": ExtraOpData(dim_args=[["dim"]]),
    "permute": ExtraOpData(is_view=True, dim_args=[["dims..."]]),
    "positive": ExtraOpData(is_view=True),
    "prod": ExtraOpData(dim_args=[["dim"]]),
    "ravel": ExtraOpData(is_view=True),
    "real": ExtraOpData(is_view=True),
    "renorm": ExtraOpData(dim_args=[["dim"]]),
    "reshape": ExtraOpData(is_view=True),
    "reshape_as": ExtraOpData(is_view=True),
    "roll": ExtraOpData(dim_args=[["dims..."]]),
    "rot90": ExtraOpData(dim_args=[["dims..."]]),
    "scatter": ExtraOpData(dim_args=[["dim"]]),
    "scatter_add": ExtraOpData(dim_args=[["dim"]]),
    "scatter_reduce.amax": ExtraOpData(dim_args=[["dim"]]),
```
- EN: This block implements local helper logic for nested. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 nested 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 166-189
```python
    "scatter_reduce.amin": ExtraOpData(dim_args=[["dim"]]),
    "scatter_reduce.mean": ExtraOpData(dim_args=[["dim"]]),
    "scatter_reduce.prod": ExtraOpData(dim_args=[["dim"]]),
    "scatter_reduce.sum": ExtraOpData(dim_args=[["dim"]]),
    "select": ExtraOpData(is_view=True, dim_args=[["dim"]]),
    "select_scatter": ExtraOpData(dim_args=[["dim"]]),
    "slice": ExtraOpData(is_view=True, dim_args=[["dim"]]),
    "slice_scatter": ExtraOpData(dim_args=[["dim"]]),
    "softmax": ExtraOpData(dim_args=[["dim"]]),
    "sort": ExtraOpData(dim_args=[["dim"]]),
    "split": ExtraOpData(is_view=True, dim_args=[["dim"]]),
    "split_with_sizes": ExtraOpData(is_view=True, dim_args=[["dim"]]),
    "split_with_sizes_copy": ExtraOpData(dim_args=[["dim"]]),
    "squeeze": ExtraOpData(is_view=True, dim_args=[["dim"], ["dim..."]]),
    "squeeze_copy": ExtraOpData(dim_args=[["dim"], ["dim..."]]),
    "stack": ExtraOpData(dim_args=[["dim"]]),
    "std": ExtraOpData(dim_args=[["dim..."]]),
    "std.unbiased": ExtraOpData(dim_args=[["dim..."]]),
    "sum": ExtraOpData(dim_args=[["dim..."]]),
    "t": ExtraOpData(is_view=True),
    "tensor_split": ExtraOpData(is_view=True, dim_args=[["dim"]]),
    "tensordot": ExtraOpData(dim_args=[["dims..."]]),
    "tile": ExtraOpData(dim_args=[["dims..."]]),
    "topk": ExtraOpData(dim_args=[["dim"]]),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 190-209
```python
    "transpose": ExtraOpData(is_view=True, dim_args=[["dim0", "dim1"]]),
    "transpose_copy": ExtraOpData(dim_args=[["dim0", "dim1"]]),
    "trapezoid": ExtraOpData(dim_args=[["dim"]]),
    "trapz": ExtraOpData(dim_args=[["dim"]]),
    "unbind": ExtraOpData(is_view=True, dim_args=[["dim"]]),
    "unflatten": ExtraOpData(is_view=True, dim_args=[["dim"]]),
    "unfold": ExtraOpData(is_view=True, dim_args=[["dimension"]]),
    "unfold_copy": ExtraOpData(dim_args=[["dimension"]]),
    "unsafe_chunk": ExtraOpData(dim_args=[["dim"]]),
    "unsafe_split": ExtraOpData(dim_args=[["dim"]]),
    "unsqueeze": ExtraOpData(is_view=True, dim_args=[["dim"]]),
    "unsqueeze_copy": ExtraOpData(dim_args=[["dim"]]),
    "var": ExtraOpData(dim_args=[["dim..."]]),
    "var.unbiased": ExtraOpData(dim_args=[["dim..."]]),
    "view": ExtraOpData(is_view=True),
    "view_as": ExtraOpData(is_view=True),
    "view_as_complex": ExtraOpData(is_view=True),
    "view_as_real": ExtraOpData(is_view=True),
}

```
- EN: This block implements local helper logic for nested. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 nested 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 210-223
```python

# random integer used for sizes
def _rnd():
    return torch.randint(3, 8, ()).item()


def _raggedness_matches(nt1, nt2):
    return (
        nt1.is_nested
        and nt2.is_nested
        and nt1._ragged_idx == nt2._ragged_idx
        and nt1.shape[nt1._ragged_idx] == nt2.shape[nt2._ragged_idx]
    )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_rnd`, `_raggedness_matches`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_rnd`, `_raggedness_matches`。

### Lines 224-237
```python

# Helper function to avoid reusing the exact same tensor / NJT across SampleInputs,
# as this causes autograd problems.
def _clone(t):
    requires_grad = t.requires_grad
    return t.detach().clone().requires_grad_(requires_grad)


# Helper function to update a sample with new kwargs / name
def _update_sample(sample, new_kwargs):
    all_kwargs = dict(sample.kwargs)
    all_kwargs.update(new_kwargs)
    full_name = ", ".join([sample.name, *(f"{k}={v}" for (k, v) in new_kwargs.items())])
    return SampleInput(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_clone`, `_update_sample`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_clone`, `_update_sample`。

### Lines 238-252
```python
        _clone(sample.input),
        args=sample.args,
        kwargs=all_kwargs,
        name=full_name,
    )


# Generates a random NT.
# dims should be something like [5, None, 10], with None indicating that a
# random ragged structure should be used
def random_nt_from_dims(
    dims, device=None, dtype=None, layout=torch.strided, requires_grad=False
):
    sizes = [[d if d is not None else _rnd() for d in dims[1:]] for d in range(dims[0])]
    return torch.nested.nested_tensor(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `random_nt_from_dims`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`random_nt_from_dims`。

### Lines 253-265
```python
        [torch.randn(*size) for size in sizes],
        device=device,
        dtype=dtype,
        layout=layout,
        requires_grad=requires_grad,
    )


# Helper function to get a reasonable string representation of an NJT for use in
# SampleInput names.
def _describe_njt(njt) -> str:
    contig_type = "_contig" if njt.is_contiguous() else "_noncontig"
    if njt._lengths is not None and njt._offsets is not None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_describe_njt`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_describe_njt`。

### Lines 266-278
```python
        contig_type += "_holes"
    elif njt._ragged_idx != 1:
        contig_type += "_transposed"

    cached_data = "_without_seqlen_cache"
    if njt._max_seqlen_tensor is not None:
        cached_data = "_with_seqlen_cache"

    return f"{njt.dim()}D{contig_type}{cached_data}"


# Helper function to get a reasonable string representation of a given dim wrt an NJT.
def _describe_dim(njt, dim):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_describe_dim`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_describe_dim`。

### Lines 279-290
```python
    if dim == 0:
        return "batch_dim"
    elif dim == njt._ragged_idx:
        return "ragged_dim"
    return "normal_dim"


# Helper function for generating a comprehensive set of NJT sample inputs.
def _sample_njts(device, dtype, requires_grad=False, dims=None):
    if dims is None:
        dims = [2, 3, 4]
    if not isinstance(dims, (list, tuple)):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_sample_njts`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_sample_njts`。

### Lines 291-305
```python
        dims = [dims]

    # contiguous NJTs
    for dim in dims:
        # with min / max seqlen cached
        shape = (_rnd(), None, *[_rnd() for _ in range(dim - 2)])
        nt = random_nt_from_dims(
            shape,
            device=device,
            dtype=dtype,
            requires_grad=requires_grad,
            layout=torch.jagged,
        )
        yield nt

```
- EN: This block handles tensor metadata or sample values; reuses computed state to reduce repeated work; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；复用已计算状态以减少重复工作；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 306-327
```python
        # without min / max seqlen cached
        values = _clone(nt.values())
        offsets = _clone(nt.offsets())
        yield torch.nested.nested_tensor_from_jagged(values, offsets).requires_grad_(
            requires_grad
        )

        # non-contiguous transposed NJT (not possible for 2D)
        if dim > 2:
            yield nt.transpose(-1, nt._ragged_idx)

        # non-contiguous with holes NJT
        values = _clone(nt.values())
        offsets = _clone(nt.offsets())
        # subtract 1 to cause holes
        lengths = _clone(offsets.diff() - 1)
        yield torch.nested.nested_tensor_from_jagged(
            values=values,
            offsets=offsets,
            lengths=lengths,
        ).requires_grad_(requires_grad)

```
- EN: This block handles tensor metadata or sample values; reuses computed state to reduce repeated work; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；复用已计算状态以减少重复工作；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 328-340
```python

# Computes an unbind-based reference for a given OpInfo on a given SampleInput.
# This reference unbinds the input NJT and invokes the op on each of the components,
# optionally wrapping the result in an NJT.
def unbind_reference(op, sample, wrap_output_as_njt=True):
    # first NJT in the arglist determines expected ragged structure
    nt_inp = (
        sample.input
        if sample.input.is_nested
        # TODO: look in kwargs too?
        else next(a for a in sample.args if a.is_nested)
    )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unbind_reference`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unbind_reference`。

### Lines 341-352
```python
    out_ref_components = []
    for i in range(nt_inp.shape[0]):

        def _slice_input(t, i=i, inp=nt_inp):
            # any NJT with the same ragged structure as the input should
            # be sliced to pass to the reference
            if isinstance(t, torch.Tensor) and _raggedness_matches(t, inp):
                return t[i]
            # allow the SampleInput to tell us how to slice it for ref calculation
            elif isinstance(t, torch.Tensor) and hasattr(t, "_batch_dim"):
                bdim = t._batch_dim  # type: ignore[attr]
                if t.shape[bdim] == 1:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_slice_input`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_slice_input`。

### Lines 353-364
```python
                    return t[0]
                else:
                    return t.select(bdim, i)
            else:
                return t

        inp = _slice_input(sample.input)
        args = tree_map(_slice_input, sample.args)
        kwargs = tree_map(_slice_input, sample.kwargs)

        # Handle indices in index_put
        if "index_put" in op.full_name and "indices" in kwargs:
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 365-385
```python
            if len(kwargs["indices"]) > 1:
                # If after unrolling we still have indices left, use them
                kwargs["indices"] = [t[i] for t in kwargs["indices"][1:]]
            else:
                # If no indices are left, create them so they match the NJT implementation
                sequence_put = kwargs["indices"][0].tolist()
                if i in sequence_put:
                    kwargs["indices"] = [
                        torch.tensor(
                            list(range(inp.shape[0])),
                            dtype=torch.int32,
                            device=kwargs["indices"][0].device,
                        )
                    ]
                else:
                    kwargs["indices"] = [
                        torch.tensor(
                            [], dtype=torch.int32, device=kwargs["indices"][0].device
                        )
                    ]

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 386-402
```python
        from torch.nested._internal.ops import _outer_to_inner_dim

        # Need to adjust dims to apply on NJT component
        if op._extra_op_data.dim_args is not None:
            # get all possible dim-related argnames that could be encountered for this op
            argnames = tree_map(
                lambda a: a.replace("...", ""),
                tree_flatten(op._extra_op_data.dim_args)[0],
            )
            # for all dim-related args present, convert from outer -> inner dim space
            for argname in {a for a in argnames if a in kwargs}:
                # allow the SampleInput to tell us how to canonicalize the dim kwargs
                ndim = nt_inp._ndim if hasattr(nt_inp, "_ndim") else nt_inp.dim()
                kwargs[argname] = _outer_to_inner_dim(
                    ndim, kwargs[argname], nt_inp._ragged_idx, canonicalize=True
                )

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 403-422
```python
        out_ref_component = op.op(inp, *args, **kwargs)
        out_ref_components.append(out_ref_component)

    if wrap_output_as_njt:
        # handle list / tuple of outputs
        if len(out_ref_components) > 0 and isinstance(
            out_ref_components[0], (list, tuple)
        ):
            num_returns = len(out_ref_components[0])
            # ensure we get the same number of returns for each invocation
            if not all(len(o) == num_returns for o in out_ref_components):
                raise AssertionError(
                    f"Expected all outputs to have {num_returns} returns"
                )
            # construct NJTs from same index returns from each invocation
            njt_returns = [
                torch.nested.as_nested_tensor(
                    [o[r] for o in out_ref_components], layout=torch.jagged
                )
                for r in range(num_returns)
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 423-437
```python
            ]
            return type(out_ref_components[0])(njt_returns)
        return torch.nested.as_nested_tensor(out_ref_components, layout=torch.jagged)

    return out_ref_components


# Computes the reference value for a non-reduction unary op with dim-wise application.
def unary_dimwise_reference(op, sample, batchwise_reference=None):
    # extract info about the dim args this op supports
    if op._extra_op_data.dim_args is None:
        raise AssertionError("Expected op._extra_op_data.dim_args to not be None")
    single_dim_argname, dimlist_argname = op._extra_op_data.get_dim_argnames()
    # only support a single non-list dim arg for now
    if dimlist_argname is not None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unary_dimwise_reference`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unary_dimwise_reference`。

### Lines 438-450
```python
        raise AssertionError("Expected dimlist_argname to be None")
    if single_dim_argname is None:
        raise AssertionError("Expected single_dim_argname to not be None")
    if sample.kwargs[single_dim_argname] == 0:
        # unbind reference won't work for batch-wise operation; handle this case here
        if batchwise_reference is None:
            raise AssertionError("Expected batchwise_reference to not be None")
        return batchwise_reference(op, sample)
    return unbind_reference(op, sample)


# Computes the reference value for a reduction op.
def reduction_reference(op, sample):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reduction_reference`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reduction_reference`。

### Lines 451-465
```python
    if not sample.input.is_nested:
        raise AssertionError("Expected sample.input.is_nested to be True")

    # extract info about the dim args this op supports
    if op._extra_op_data.dim_args is None:
        raise AssertionError("Expected op._extra_op_data.dim_args to not be None")
    single_dim_argname, dimlist_argname = op._extra_op_data.get_dim_argnames()
    if single_dim_argname is None:
        raise AssertionError("Expected single_dim_argname to not be None")

    dim = sample.kwargs.get(
        dimlist_argname, sample.kwargs.get(single_dim_argname, None)
    )
    keepdim = sample.kwargs.get("keepdim", False)
    if dim == 0:
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 466-477
```python
        raise AssertionError("reductions over just the batch dim are not supported")
    if isinstance(dim, (tuple, list)):
        reduce_on_ragged = sample.input._ragged_idx in dim
        reduce_on_batch = 0 in dim
    else:
        reduce_on_ragged = sample.input._ragged_idx == dim
        reduce_on_batch = dim == 0

    if dim is None:
        # calculate reference value by running reduction on values buffer
        return op.op(sample.input.values(), *sample.args, **sample.kwargs)

```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 478-489
```python
    if reduce_on_ragged and reduce_on_batch:
        # run reference directly on buffer with dims converted to inner space
        from torch.nested._internal.ops import _outer_to_inner_dim

        ref_kwargs = dict(sample.kwargs)
        if dimlist_argname is None:
            raise AssertionError("Expected dimlist_argname to not be None")
        ref_kwargs[dimlist_argname] = _outer_to_inner_dim(
            sample.input.dim(), dim, sample.input._ragged_idx, canonicalize=True
        )
        out = op.op(sample.input.values(), *sample.args, **ref_kwargs)
        if keepdim:
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 490-506
```python
            if isinstance(out, (tuple, list)):
                # some ops return multiple things; unsqueeze all of them
                out = type(out)(o.unsqueeze(0) for o in out)
            else:
                out = out.unsqueeze(0)
        return out

    if reduce_on_ragged and not reduce_on_batch:
        # calculate reference value by running an unbind reference and stacking
        out_ref_components = unbind_reference(op, sample, wrap_output_as_njt=False)
        if len(out_ref_components) > 0 and isinstance(
            out_ref_components[0], (tuple, list)
        ):
            # some ops return multiple things; stack all of them
            num_returns = len(out_ref_components[0])
            # ensure we get the same number of returns for each invocation
            if not all(len(o) == num_returns for o in out_ref_components):
```
- EN: This block handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 507-519
```python
                raise AssertionError(
                    f"Expected all outputs to have {num_returns} returns"
                )
            # stack same index returns from each invocation
            stacked_returns = [
                torch.stack([o[r] for o in out_ref_components], dim=0)
                for r in range(num_returns)
            ]
            return type(out_ref_components[0])(stacked_returns)
        return torch.stack(out_ref_components, dim=0)

    # unbind reference works for other reductions
    return unbind_reference(op, sample)
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 520-532
```python


def sample_inputs_elementwise_njt_unary(
    op_info, device, dtype, requires_grad, op_kwargs=None, **kwargs
):
    if not op_kwargs:
        op_kwargs = {}

    for njt in _sample_njts(
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[2, 3, 4]
    ):
        yield SampleInput(njt, kwargs=dict(op_kwargs), name=_describe_njt(njt))

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_elementwise_njt_unary`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_elementwise_njt_unary`。

### Lines 533-551
```python

def sample_inputs_elementwise_njt_binary(
    op_info, device, dtype, requires_grad, op_kwargs=None, **kwargs
):
    if not op_kwargs:
        op_kwargs = {}

    for njt1 in _sample_njts(
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[2, 3, 4]
    ):
        njt_desc = _describe_njt(njt1)
        njt2 = torch.randn_like(njt1)
        yield SampleInput(
            _clone(njt1),
            args=(njt2,),
            kwargs=dict(op_kwargs),
            name=f"{njt_desc}: (NT, NT)",
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_elementwise_njt_binary`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_elementwise_njt_binary`。

### Lines 552-575
```python
        # broadcasting case: (B, j0, ...) with (B, 1, ...)
        dense_shape = list(njt1.shape)
        dense_shape[njt1._ragged_idx] = 1
        t = torch.randn(
            dense_shape,
            device=device,
            dtype=dtype,
            requires_grad=requires_grad,
        )
        t2 = _clone(t)
        # used for slicing in unbind_reference()
        t._batch_dim = 0
        t2._batch_dim = 0
        # (NT, T)
        yield SampleInput(
            _clone(njt1),
            args=(t,),
            kwargs=dict(op_kwargs),
            name=f"{njt_desc}: (NT, T) broadcasting 1 over ragged",
        )
        # (T, NT)
        yield SampleInput(
            t2,
            args=(_clone(njt1),),
```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 576-599
```python
            kwargs=dict(op_kwargs),
            name=f"{njt_desc}: (T, NT) broadcasting 1 over ragged",
        )

        # broadcasting case: (B, j0, ...) with (1, 1...)
        t = torch.randn(
            [1 for _ in range(njt1.dim())],
            device=device,
            dtype=dtype,
            requires_grad=requires_grad,
        )
        t2 = _clone(t)
        # used for slicing in unbind_reference()
        t._batch_dim = 0
        t2._batch_dim = 0
        # (NT, T)
        yield SampleInput(
            _clone(njt1),
            args=(t,),
            kwargs=dict(op_kwargs),
            name=f"{njt_desc}: (NT, T) broadcasting all 1s",
        )
        # (T, NT)
        yield SampleInput(
```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 600-623
```python
            t2,
            args=(_clone(njt1),),
            kwargs=dict(op_kwargs),
            name=f"{njt_desc}: (T, NT) broadcasting all 1s",
        )

        # broadcasting case: (B, j0, ...) with (...)
        if njt1.dim() > njt1._ragged_idx + 1:
            t = torch.randn(
                njt1.shape[njt1._ragged_idx + 1 :],
                device=device,
                dtype=dtype,
                requires_grad=requires_grad,
            )
            # (NT, T)
            yield SampleInput(
                _clone(njt1),
                args=(_clone(t),),
                kwargs=dict(op_kwargs),
                name=f"{njt_desc}: (NT, T) broadcasting normal dims",
            )
            # (T, NT)
            yield SampleInput(
                _clone(t),
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 624-645
```python
                args=(_clone(njt1),),
                kwargs=dict(op_kwargs),
                name=f"{njt_desc}: (T, NT) broadcasting normal dims",
            )

        # broadcasting case: (B, j0, ...) with scalar
        t = torch.randn((), device=device, dtype=dtype, requires_grad=requires_grad)
        # (NT, T)
        yield SampleInput(
            _clone(njt1),
            args=(_clone(t),),
            kwargs=dict(op_kwargs),
            name=f"{njt_desc}: (NT, T) broadcasting with scalar",
        )
        # (T, NT)
        yield SampleInput(
            _clone(t),
            args=(_clone(njt1),),
            kwargs=dict(op_kwargs),
            name=f"{njt_desc}: (T, NT) broadcasting with scalar",
        )

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 646-662
```python
    # mixed broadcasting case: (B, j0, 1) with (B, 1, D)
    B = 4
    D = 16
    njt = random_nt_from_dims(
        (B, None, 1),
        device=device,
        dtype=dtype,
        requires_grad=requires_grad,
        layout=torch.jagged,
    )
    njt_desc = _describe_njt(njt)
    t = torch.randn(B, 1, D, device=device, dtype=dtype, requires_grad=requires_grad)
    t2 = _clone(t)
    # used for slicing in unbind_reference()
    t._batch_dim = 0
    t2._batch_dim = 0

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 663-677
```python
    # (NT, T)
    yield SampleInput(
        _clone(njt),
        args=(t,),
        kwargs=dict(op_kwargs),
        name=f"{njt_desc}: (NT, T) mixed broadcasting",
    )
    # (T, NT)
    yield SampleInput(
        t2,
        args=(_clone(njt),),
        kwargs=dict(op_kwargs),
        name=f"{njt_desc}: (T, NT) mixed broadcasting",
    )

```
- EN: This block implements local helper logic for nested. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 nested 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 678-690
```python

def sample_inputs_njt_reduction(
    op_info,
    device,
    dtype,
    requires_grad,
    supports_keepdim=True,
    op_kwargs=None,
    **kwargs,
):
    if not op_kwargs:
        op_kwargs = {}

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_njt_reduction`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_njt_reduction`。

### Lines 691-702
```python
    # extract info about the dim args this op supports
    if op_info._extra_op_data.dim_args is None:
        raise AssertionError("Expected op_info._extra_op_data.dim_args to not be None")
    (
        single_dim_argname,
        dimlist_argname,
    ) = op_info._extra_op_data.get_dim_argnames()
    if single_dim_argname is None:
        raise AssertionError("Expected single_dim_argname to not be None")
    supports_dimlist = dimlist_argname is not None

    for njt in _sample_njts(
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 703-723
```python
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[2, 3, 4]
    ):
        njt_desc = _describe_njt(njt)
        keepdim_values = [False, True] if supports_keepdim else [None]
        for keepdim in keepdim_values:
            keepdim_suffix = f" with keepdim={keepdim}" if supports_keepdim else ""
            # single dim-wise reduction; includes reduction over the ragged dim
            # NB: reduction over the batch dim is not supported!
            # TODO: Cover this in the set of error inputs
            for dim in range(1, njt.dim()):
                dim_desc = "normal" if dim != njt._ragged_idx else "ragged"
                yield SampleInput(
                    _clone(njt),
                    kwargs={
                        **op_kwargs,
                        single_dim_argname: dim,
                        **({"keepdim": keepdim} if supports_keepdim else {}),
                    },
                    name=f"{njt_desc}: {dim_desc} dim reduction{keepdim_suffix}",
                )

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 724-735
```python
            if supports_dimlist:
                # reduce on both batch and ragged dims
                yield SampleInput(
                    _clone(njt),
                    kwargs={
                        **op_kwargs,
                        dimlist_argname: [0, njt._ragged_idx],
                        **({"keepdim": keepdim} if supports_keepdim else {}),
                    },
                    name=f"{njt_desc}: batch+ragged reduction{keepdim_suffix}",
                )

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 736-750
```python
                # reduce on batch, ragged, and other dims
                for other_dim in range(njt._ragged_idx + 1, njt.dim()):
                    yield SampleInput(
                        _clone(njt),
                        kwargs={
                            **op_kwargs,
                            dimlist_argname: [0, njt._ragged_idx, other_dim],
                            **({"keepdim": keepdim} if supports_keepdim else {}),
                        },
                        name=(
                            f"{njt_desc}: batch+ragged+dim={other_dim} "
                            f"reduction{keepdim_suffix}"
                        ),
                    )

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 751-762
```python
                # reduce on two non-ragged, non-batch dims
                if njt.dim() > 3 and njt._ragged_idx == 1:
                    yield SampleInput(
                        _clone(njt),
                        kwargs={
                            **op_kwargs,
                            dimlist_argname: [njt.dim() - 2, njt.dim() - 1],
                            **({"keepdim": keepdim} if supports_keepdim else {}),
                        },
                        name=f"{njt_desc}: two normal dim reduction{keepdim_suffix}",
                    )

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 763-776
```python
                # full reduction by specifying all dims
                yield SampleInput(
                    _clone(njt),
                    kwargs={
                        **op_kwargs,
                        dimlist_argname: list(range(njt.dim())),
                        **({"keepdim": keepdim} if supports_keepdim else {}),
                    },
                    name=f"{njt_desc}: all dim reduction{keepdim_suffix}",
                )

                # TODO: Reducing on ragged dim and non-batch dim is not supported;
                # cover this in the set of error inputs.

```
- EN: This block reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 777-791
```python
        # full reduction
        yield SampleInput(
            _clone(njt),
            kwargs=dict(op_kwargs),
            name=f"{njt_desc}: full reduction with keepdim={keepdim}",
        )


def unsupported_sample_inputs_func(op_name):
    def _f(op_info, device, dtype, requires_grad, op_name=op_name, **kwargs):
        raise RuntimeError(
            f"OpInfo for {op_name} does not support NJT. Support can be added by modifying "
            "torch/testing/_internal/opinfo/definitions/nested.py."
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unsupported_sample_inputs_func`, `_f`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unsupported_sample_inputs_func`, `_f`。

### Lines 792-803
```python
    return _f


def unsupported_reference(op_name):
    def _f(op, sample):
        raise RuntimeError(
            f"OpInfo for {op_name} does not define a ref() function. Support can be added by "
            "modifying torch/testing/_internal/opinfo/definitions/nested.py."
        )

    return _f

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unsupported_reference`, `_f`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unsupported_reference`, `_f`。

### Lines 804-816
```python

# === BEGIN OP-SPECIFIC SAMPLE INPUTS FUNCS / REFERENCES ===
def sample_inputs_unary_dimwise(
    op_info, device, dtype, requires_grad, op_kwargs=None, **kwargs
):
    if op_kwargs is None:
        op_kwargs = {}

    # only support a single non-list dim arg for now
    if op_info._extra_op_data is None:
        raise AssertionError("Expected op_info._extra_op_data to not be None")
    single_dim_argname, dimlist_argname = op_info._extra_op_data.get_dim_argnames()
    if single_dim_argname is None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_unary_dimwise`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_unary_dimwise`。

### Lines 817-832
```python
        raise AssertionError("Expected single_dim_argname to not be None")
    if dimlist_argname is not None:
        raise AssertionError("Expected dimlist_argname to be None")

    for njt in _sample_njts(
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[2, 3, 4]
    ):
        for dim in range(njt.dim()):
            kwargs = {single_dim_argname: dim}
            kwargs.update(op_kwargs)
            yield SampleInput(
                _clone(njt),
                kwargs=kwargs,
                name=f"{_describe_njt(njt)}: {_describe_dim(njt, dim)}",
            )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 833-844
```python

def batchwise_reference_chunk(op, sample):
    # reference for chunk() over dim=0
    B = sample.input.size(0)
    num_chunks = sample.kwargs["chunks"]
    chunk_size = math.ceil(B / num_chunks)
    num_full_chunks = B // chunk_size
    chunk_sizes = [chunk_size for _ in range(num_full_chunks)]
    if B % chunk_size != 0:
        # final chunk contains the leftovers
        chunk_sizes.append(B % chunk_size)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `batchwise_reference_chunk`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`batchwise_reference_chunk`。

### Lines 845-856
```python
    # split unbound components into chunks according to calculated sizes
    components = list(sample.input.unbind())
    start = 0
    chunks = []
    for chunk_size in chunk_sizes:
        chunks.append(components[start : start + chunk_size])
        start += chunk_size

    # rejoin into NJT outputs
    return [torch.nested.as_nested_tensor(lst, layout=torch.jagged) for lst in chunks]


```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 857-870
```python
def batchwise_reference_narrow(op, sample):
    # TODO: write this!
    raise NotImplementedError


def batchwise_reference_select(op, sample):
    # reference for select() over dim=0
    return sample.input.unbind()[sample.kwargs["index"]]


def batchwise_reference_split(op, sample):
    # TODO: write this!
    raise NotImplementedError

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `batchwise_reference_narrow`, `batchwise_reference_select`, `batchwise_reference_split`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`batchwise_reference_narrow`, `batchwise_reference_select`, `batchwise_reference_split`。

### Lines 871-882
```python

def batchwise_reference_split_with_sizes(op, sample):
    # TODO: write this!
    raise NotImplementedError


def batchwise_reference_unflatten(op, sample):
    # TODO: write this!
    raise NotImplementedError


def batchwise_reference_unsqueeze(op, sample):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `batchwise_reference_split_with_sizes`, `batchwise_reference_unflatten`, `batchwise_reference_unsqueeze`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`batchwise_reference_split_with_sizes`, `batchwise_reference_unflatten`, `batchwise_reference_unsqueeze`。

### Lines 883-903
```python
    raise ValueError("unsqueeze() is not intended to operate on the batch dim")


def sample_inputs_clone(op_info, device, dtype, requires_grad, **kwargs):
    # non-contiguous NJTs
    for njt in _sample_njts(
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[2, 3, 4]
    ):
        yield SampleInput(njt, name=_describe_njt(njt))

    for memory_format in (torch.contiguous_format, torch.preserve_format):
        # construct a "non-contiguous with holes" NJT
        values = torch.randn(
            10, 5, device=device, dtype=dtype, requires_grad=requires_grad
        )
        offsets = torch.tensor([0, 2, 4, 10], device=device, dtype=torch.int64)
        lengths = torch.tensor([2, 1, 3], device=device, dtype=torch.int64)
        njt = torch.nested.nested_tensor_from_jagged(
            values, offsets=offsets, lengths=lengths
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_clone`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_clone`。

### Lines 904-916
```python
        njt_desc = _describe_njt(njt)
        yield SampleInput(
            njt,
            kwargs={"memory_format": memory_format},
            name=f"{njt_desc}: {memory_format})",
        )


def sample_inputs_fill(op_info, device, dtype, requires_grad, **kwargs):
    # scalar case
    unary_func = partial(sample_inputs_elementwise_njt_unary, op_kwargs={"value": 42.0})
    yield from unary_func(op_info, device, dtype, requires_grad)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_fill`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_fill`。

### Lines 917-928
```python
    # TODO: add Tensor case


def sample_inputs_mvl_gamma(p):
    return partial(sample_inputs_elementwise_njt_unary, op_kwargs={"p": p})


def sample_inputs_polygamma_n(n):
    return partial(sample_inputs_elementwise_njt_unary, op_kwargs={"n": n})


def sample_inputs_special_polygamma_n(n):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_mvl_gamma`, `sample_inputs_polygamma_n`, `sample_inputs_special_polygamma_n`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_mvl_gamma`, `sample_inputs_polygamma_n`, `sample_inputs_special_polygamma_n`。

### Lines 929-942
```python
    return partial(sample_inputs_elementwise_njt_unary, op_kwargs={"n": n})


def sample_inputs_to(op_info, device, dtype, requires_grad, op_kwargs=None, **kwargs):
    for njt in _sample_njts(
        device=device,
        dtype=dtype,
        requires_grad=requires_grad,
        dims=[2, 3, 4],
    ):
        other_dtypes = (
            d for d in (torch.float32, torch.half, torch.double) if d is not dtype
        )
        for other_dtype in other_dtypes:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_to`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_to`。

### Lines 943-954
```python
            sample_name = f"{njt.dim()}D: {dtype} -> {other_dtype}"
            yield SampleInput(_clone(njt), kwargs={"dtype": dtype}, name=sample_name)

        # only include device transfer for CUDA inputs
        if "cuda" in device:
            other_device = "cpu"
            sample_name = f"{_describe_njt(njt)}: {device} -> {other_device}"
            yield SampleInput(
                _clone(njt), kwargs={"device": other_device}, name=sample_name
            )


```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 955-972
```python
def sample_inputs_bmm(op_info, device, dtype, requires_grad, op_kwargs=None, **kwargs):
    for njt_3d in _sample_njts(
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[3]
    ):
        # (B, j1, D) x (B, D, E) => (B, j1, E)
        if njt_3d._ragged_idx == 1:
            B, D = njt_3d.shape[0], njt_3d.shape[-1]
            E = D + 2
            other = torch.randn(B, D, E, device=device, dtype=dtype)
            # used for slicing in unbind_reference()
            other._batch_dim = 0
            njt_desc = _describe_njt(njt_3d)
            yield SampleInput(
                _clone(njt_3d),
                kwargs={"mat2": other},
                name=f"{njt_desc}: (B, j, D) x (B, D, E)",
            )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_bmm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_bmm`。

### Lines 973-986
```python
        # TODO (need factory functions):
        # (B, D, j1) x (B, j1, E) => (B, D, E)


def reference_bmm(op, sample):
    # unbind reduces a dim and bmm requires 3D, so use matmul as the reference
    matmul_op = copy(op)
    matmul_op.op = torch.matmul
    # change arg name from mat2 -> other
    modified_sample = copy(sample)
    other = modified_sample.kwargs["mat2"]
    del modified_sample.kwargs["mat2"]
    modified_sample.kwargs["other"] = other
    return unbind_reference(matmul_op, modified_sample)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_bmm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_bmm`。

### Lines 987-999
```python


def sample_inputs_chunk(op_info, device, dtype, requires_grad, **kwargs):
    for sample_input in sample_inputs_unary_dimwise(
        op_info, device, dtype, requires_grad, **kwargs
    ):
        # ragged dim chunking: test a single chunks value
        if sample_input.kwargs["dim"] == sample_input.input._ragged_idx:
            yield _update_sample(sample_input, {"chunks": 3})
        # other dim chunking: test different chunks values
        else:
            D = sample_input.input.size(sample_input.kwargs["dim"])
            for chunks in [1, D // 2, D - 1, D]:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_chunk`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_chunk`。

### Lines 1000-1013
```python
                yield _update_sample(sample_input, {"chunks": chunks})


def sample_inputs_matmul(
    op_info, device, dtype, requires_grad, op_kwargs=None, **kwargs
):
    # also run bmm samples through
    for sample_input in sample_inputs_bmm(op_info, device, dtype, requires_grad):
        # change arg name from mat2 -> other
        other = sample_input.kwargs["mat2"]
        del sample_input.kwargs["mat2"]
        sample_input.kwargs["other"] = other
        yield sample_input

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_matmul`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_matmul`。

### Lines 1014-1028
```python
    # 3D cases not covered by bmm
    for njt_3d in _sample_njts(
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[3]
    ):
        # (B, j1, D) x (D, E) => (B, j1, E)
        if njt_3d._ragged_idx == 1:
            D = njt_3d.shape[-1]
            E = D + 2
            njt_desc = _describe_njt(njt_3d)
            yield SampleInput(
                _clone(njt_3d),
                kwargs={"other": torch.randn(D, E, device=device, dtype=dtype)},
                name=f"{njt_desc}: (B, j, D) x (D, E)",
            )

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1029-1043
```python
    # 4D cases
    for njt_4d in _sample_njts(
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[4]
    ):
        # (B, j1, D, E) x (E, F) => (B, j1, D, F)
        if njt_4d._ragged_idx == 1:
            E = njt_4d.shape[-1]
            F = E + 2
            njt_desc = _describe_njt(njt_4d)
            yield SampleInput(
                _clone(njt_4d),
                kwargs={"other": torch.randn(E, F, device=device, dtype=dtype)},
                name=f"{njt_desc}: (B, j, D, E) x (E, F)",
            )

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1044-1066
```python
    # Dense x NJT cases
    for njt_3d in _sample_njts(
        device=device,
        dtype=dtype,
        requires_grad=requires_grad,
        dims=[3],
    ):
        # (B, F, E) x (B, E, j1) => (B, F, j1)
        if njt_3d._ragged_idx == 2:
            B = njt_3d.shape[0]
            E = njt_3d.shape[1]
            F = E + 2
            njt_desc = _describe_njt(njt_3d)
            dense_t = torch.randn(
                B, F, E, device=device, dtype=dtype, requires_grad=requires_grad
            )
            dense_t._batch_dim = 0  # for unbind_reference()
            yield SampleInput(
                dense_t,
                args=(_clone(njt_3d),),
                name=f"{njt_desc}: (B, F, E) x (B, E, j1)",
            )

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1067-1090
```python
    # NJT x NJT => Dense case
    for njt_3d in _sample_njts(
        device=device,
        dtype=dtype,
        requires_grad=requires_grad,
        dims=[3],
    ):
        # (B, E, j1) x (B, j1, F) => (B, E, F)
        if njt_3d._ragged_idx == 2 and njt_3d.is_contiguous():
            B, E, _ = njt_3d.shape
            sum_j1 = len(njt_3d.values())
            other_cont = torch.randn(
                sum_j1, E + 2, device=device, dtype=dtype, requires_grad=requires_grad
            )
            other_njt = torch.nested.nested_tensor_from_jagged(
                other_cont, njt_3d.offsets(), lengths=njt_3d._lengths
            )
            njt_desc = _describe_njt(njt_3d)
            yield SampleInput(
                _clone(njt_3d),
                kwargs={"other": _clone(other_njt)},
                name=f"{njt_desc}: (B, E, j1) x (B, j1, F)",
            )

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1091-1106
```python
        # TODO (need factory functions):
        # (B, j1, D, E) x (B, j1, E, F) => (B, j1, D, F)


def sample_inputs_masked_select(
    op_info, device, dtype, requires_grad, op_kwargs=None, **kwargs
):
    for njt in _sample_njts(
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[2]
    ):
        yield SampleInput(
            njt,
            kwargs={"mask": (torch.randn_like(njt, requires_grad=False) < 0.0)},
            name=_describe_njt(njt),
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_masked_select`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_masked_select`。

### Lines 1107-1118
```python

def sample_inputs_narrow(op_info, device, dtype, requires_grad, **kwargs):
    for sample_input in sample_inputs_unary_dimwise(
        op_info, device, dtype, requires_grad, **kwargs
    ):
        # ragged dim narrowing: test a single start, length value
        if sample_input.kwargs["dim"] == sample_input.input._ragged_idx:
            yield _update_sample(sample_input, {"start": 1, "length": 2})
        # other dim narrowing: test different start, length values
        else:
            D = sample_input.input.size(sample_input.kwargs["dim"])
            for start, length in [(0, D), (0, D - 1), (1, D - 1), (D - 1, 1)]:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_narrow`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_narrow`。

### Lines 1119-1135
```python
                yield _update_sample(sample_input, {"start": start, "length": length})


def sample_inputs_nn_functional_embedding(
    op_info, device, dtype, requires_grad, **kwargs
):
    indices = torch.nested.nested_tensor(
        [
            torch.tensor([0, 2, 1, 3]),
            torch.tensor([4, 2, 1]),
            torch.tensor([6, 7, 5, 2, 4]),
        ],
        layout=torch.jagged,
        dtype=torch.int64,
        device=device,
    )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_nn_functional_embedding`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_nn_functional_embedding`。

### Lines 1136-1152
```python
    NUM_EMBEDDINGS = 20
    EMBEDDING_DIM = 32
    weight = torch.randn(NUM_EMBEDDINGS, EMBEDDING_DIM, device=device, dtype=dtype)

    # NB: the OpInfo entry for embedding_bag expects weight first so the gradients
    # can be checked
    yield SampleInput(
        _clone(weight).requires_grad_(),
        args=(indices,),
    )

    yield SampleInput(
        _clone(weight).requires_grad_(),
        args=(indices,),
        kwargs={"padding_idx": 1},
    )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1153-1165
```python

def sample_inputs_index_put(
    op_info, device, dtype, requires_grad, op_kwargs=None, **kwargs
):
    for njt in _sample_njts(
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[2, 3, 4]
    ):
        for dim in range(njt.dim()):
            indices = [
                torch.tensor(list(range(njt.size(0))), device=njt.device),
                *[
                    torch.tensor([0] * njt.size(0), device=njt.device)
                    for _ in range(dim - 1)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_index_put`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_index_put`。

### Lines 1166-1177
```python
                ],
            ]
            njt_desc = _describe_njt(njt)
            yield SampleInput(
                _clone(njt),
                kwargs={
                    "indices": indices,
                    "values": torch.tensor(1.0, device=njt.device),
                },
                name=f"{njt_desc}: up to dim {dim - 1}",
            )

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1178-1189
```python
    # Non-cont NJT for completeness
    offsets = torch.tensor([0, 2, 5, 7], device=device)
    lengths = torch.tensor([2, 2, 2], device=device)
    indices = [
        torch.tensor([0, 1, 2], device=device),
        torch.tensor([0, 1, 1], device=device),
        torch.tensor([0, 0, 0], device=device),
    ]
    a = torch.nested.nested_tensor_from_jagged(
        torch.zeros(7, 3, device=device), offsets, lengths
    ).requires_grad_(requires_grad)

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1190-1201
```python
    njt_desc = _describe_njt(a)
    yield SampleInput(
        _clone(a),
        kwargs={"indices": indices, "values": torch.tensor(1.0, device=a.device)},
        name=f"{njt_desc}: all dims",
    )


def sample_inputs_nn_functional_embedding_bag(
    op_info, device, dtype, requires_grad, **kwargs
):
    for generate_per_sample_weight in (True, False):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_nn_functional_embedding_bag`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_nn_functional_embedding_bag`。

### Lines 1202-1223
```python
        for mode in ("sum", "mean", "max"):
            # per_sample_weights is only supported for mode='sum'
            if mode != "sum" and generate_per_sample_weight:
                continue

            NUM_EMBEDDINGS = 10
            EMBEDDING_DIM = 32
            weight = torch.randn(
                NUM_EMBEDDINGS, EMBEDDING_DIM, dtype=dtype, device=device
            )

            njt = torch.nested.nested_tensor(
                [
                    torch.randint(0, NUM_EMBEDDINGS, size=(2,)),
                    torch.randint(0, NUM_EMBEDDINGS, size=(3,)),
                    torch.randint(0, NUM_EMBEDDINGS, size=(4,)),
                ],
                layout=torch.jagged,
                dtype=torch.int64,
                device=device,
            )

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1224-1238
```python
            per_sample_weights = None
            if generate_per_sample_weight:
                per_sample_weights = torch.randn_like(njt, dtype=dtype)

            # NB: the OpInfo entry for embedding_bag expects weight first so the gradients
            # can be checked
            yield SampleInput(
                weight,
                args=(njt,),
                kwargs={
                    "mode": mode,
                    "per_sample_weights": per_sample_weights,
                },
            )

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1239-1253
```python

def reference_nn_functional_embedding_bag(op, sample):
    # run reference on a single bag at a time
    new_kwargs = dict(sample.kwargs)
    new_kwargs.update(
        {"offsets": torch.tensor([0], dtype=torch.int64, device=sample.input.device)}
    )
    # flip input / weight back to what unbind_reference() expects
    sample = SampleInput(sample.args[0], args=(sample.input,), kwargs=new_kwargs)
    old_op = op.op
    op.op = torch.nn.functional.embedding_bag
    output = unbind_reference(op, sample, wrap_output_as_njt=False)
    op.op = old_op
    # concat bag outputs to get final output
    return torch.cat(output, dim=0)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_nn_functional_embedding_bag`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_nn_functional_embedding_bag`。

### Lines 1254-1277
```python


def sample_inputs_nn_functional_linear(op_info, device, dtype, requires_grad, **kwargs):
    for njt in _sample_njts(
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[3, 4, 5]
    ):
        # projection over a ragged dim is not currently supported
        if is_nested_int(njt.size(-1)):
            continue

        # with bias
        NUM_OUTPUT = 10
        weight = torch.randn(
            NUM_OUTPUT,
            njt.size(-1),
            device=device,
            dtype=dtype,
            requires_grad=requires_grad,
        )
        bias = torch.randn(
            NUM_OUTPUT, device=device, dtype=dtype, requires_grad=requires_grad
        )
        yield SampleInput(
            _clone(njt),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_nn_functional_linear`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_nn_functional_linear`。

### Lines 1278-1293
```python
            kwargs={
                "weight": _clone(weight),
                "bias": _clone(bias),
            },
            name=f"{_describe_njt(njt)}: with bias",
        )

        # without bias
        yield SampleInput(
            _clone(njt),
            kwargs={
                "weight": _clone(weight),
            },
            name=f"{_describe_njt(njt)}: without bias",
        )

```
- EN: This block implements local helper logic for nested. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 nested 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1294-1311
```python

def sample_inputs_nn_functional_prelu(op_info, device, dtype, requires_grad, **kwargs):
    for njt in _sample_njts(
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[3, 4]
    ):
        # Second dim is interpreted as number of channels; this should be non-ragged for now
        num_channels = njt.size(1)
        if is_nested_int(num_channels):
            continue

        # 1D weight
        weight = torch.randn(
            num_channels,
            device=device,
            dtype=dtype,
            requires_grad=requires_grad,
        )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_nn_functional_prelu`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_nn_functional_prelu`。

### Lines 1312-1328
```python
        yield SampleInput(
            _clone(njt),
            kwargs={
                "weight": _clone(weight),
            },
            name=f"{_describe_njt(njt)}: 1D weight",
        )

        # scalar tensor weight
        yield SampleInput(
            _clone(njt),
            kwargs={
                "weight": torch.tensor(4.2, device=device, dtype=dtype),
            },
            name=f"{_describe_njt(njt)}: scalar tensor weight",
        )

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1329-1340
```python

def sample_inputs_nn_functional_rms_norm(
    op_info, device, dtype, requires_grad, **kwargs
):
    for njt in _sample_njts(
        device=device, dtype=dtype, requires_grad=requires_grad, dims=[3, 4]
    ):
        # normalize over non-ragged dims
        for start_dim in range(njt.dim()):
            if start_dim <= njt._ragged_idx:
                continue

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_nn_functional_rms_norm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_nn_functional_rms_norm`。

### Lines 1341-1357
```python
            normalized_shape = njt.shape[start_dim:]
            weight = torch.randn(
                normalized_shape,
                device=device,
                dtype=dtype,
                requires_grad=requires_grad,
            )

            yield SampleInput(
                _clone(njt),
                kwargs={
                    "normalized_shape": normalized_shape,
                    "weight": weight,
                },
                name=f"{_describe_njt(njt)}",
            )

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1358-1370
```python

sample_inputs_nn_functional_threshold = partial(
    sample_inputs_elementwise_njt_unary,
    op_kwargs={"threshold": float.fromhex("0x1.3ap-3"), "value": -9},
)


def sample_inputs_select(op_info, device, dtype, requires_grad, **kwargs):
    for sample_input in sample_inputs_unary_dimwise(
        op_info, device, dtype, requires_grad, **kwargs
    ):
        # ragged dim chunking: test a single index
        if sample_input.kwargs["dim"] == sample_input.input._ragged_idx:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_select`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_select`。

### Lines 1371-1384
```python
            yield _update_sample(sample_input, {"index": 0})
        # other dim chunking: test different indices
        else:
            D = sample_input.input.size(sample_input.kwargs["dim"])
            for index in [0, D // 2, D - 1]:
                yield _update_sample(sample_input, {"index": index})


def sample_inputs_split(op_info, device, dtype, requires_grad, **kwargs):
    for sample_input in sample_inputs_unary_dimwise(
        op_info, device, dtype, requires_grad, **kwargs
    ):
        # ragged dim chunking: test a single split size
        if sample_input.kwargs["dim"] == sample_input.input._ragged_idx:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_split`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_split`。

### Lines 1385-1396
```python
            yield _update_sample(sample_input, {"split_size_or_sections": 3})
        # other dim chunking: test different split sizes
        else:
            D = sample_input.input.size(sample_input.kwargs["dim"])
            for split_size in [1, D // 2, D - 1, D]:
                yield _update_sample(
                    sample_input, {"split_size_or_sections": split_size}
                )


def sample_inputs_split_with_sizes(op_info, device, dtype, requires_grad, **kwargs):
    for sample_input in sample_inputs_unary_dimwise(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_split_with_sizes`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_split_with_sizes`。

### Lines 1397-1409
```python
        op_info, device, dtype, requires_grad, **kwargs
    ):
        # It will never make sense to operate on the ragged dim.
        # TODO: Handle this with error_inputs
        if sample_input.kwargs["dim"] == sample_input.input._ragged_idx:
            continue

        D = sample_input.input.size(sample_input.kwargs["dim"])
        # splits should add up to D
        split1 = torch.randint(0, D - 1, size=()).item()
        split2 = D - split1
        yield _update_sample(sample_input, {"split_sizes": [split1, split2]})

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 1410-1433
```python

def sample_inputs_squeeze(op_info, device, dtype, requires_grad, **kwargs):
    # squeeze-specific NJT generator (need to ensure there are some 1s in the shape)
    def _get_njts():
        njt = random_nt_from_dims(
            (4, None, 1, 3, 1),
            device=device,
            dtype=dtype,
            requires_grad=requires_grad,
            layout=torch.jagged,
        )
        yield njt
        # without min / max seqlen cached
        values = njt.values().detach().clone()
        offsets = njt.offsets().detach().clone()
        yield torch.nested.nested_tensor_from_jagged(values, offsets)
        # non-contiguous transposed
        yield njt.transpose(1, 3)
        # non-contiguous with holes
        values = njt.values().detach().clone()
        offsets = njt.offsets().detach().clone()
        # subtract 1 to cause holes
        lengths = (offsets.diff() - 1).detach().clone()
        yield torch.nested.nested_tensor_from_jagged(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_squeeze`, `_get_njts`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_squeeze`, `_get_njts`。

### Lines 1434-1446
```python
            values=values,
            offsets=offsets,
            lengths=lengths,
        )

    for njt in _get_njts():
        # single dim operation
        for dim in range(njt.dim()):
            # Operation on batch / ragged dim is never expected to work.
            # TODO: Handle these via error_inputs.
            if dim == 0 or dim == njt._ragged_idx:
                continue

```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1447-1459
```python
            yield SampleInput(
                _clone(njt),
                kwargs={"dim": dim},
                name=f"{_describe_njt(njt)}: {_describe_dim(njt, dim)}",
            )

        # multiple dim operation (pass no args)
        yield SampleInput(
            _clone(njt),
            kwargs={"dim": dim},
            name=f"{_describe_njt(njt)}: multiple dims",
        )

```
- EN: This block implements local helper logic for nested. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 nested 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1460-1474
```python

def sample_inputs_unflatten(op_info, device, dtype, requires_grad, **kwargs):
    for sample_input in sample_inputs_unary_dimwise(
        op_info, device, dtype, requires_grad, **kwargs
    ):
        # It will never make sense to operate on the ragged dim.
        # TODO: Handle this with error_inputs
        if sample_input.kwargs["dim"] == sample_input.input._ragged_idx:
            continue

        D = sample_input.input.size(sample_input.kwargs["dim"])
        # sizes should multiply to be D
        yield _update_sample(sample_input, {"sizes": [D, 1]})
        yield _update_sample(sample_input, {"sizes": [1, D]})
        if D % 2 == 0:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_unflatten`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_unflatten`。

### Lines 1475-1494
```python
            yield _update_sample(sample_input, {"sizes": [D // 2, 2]})
            yield _update_sample(sample_input, {"sizes": [2, D // 2]})


def sample_inputs_unsqueeze(op_info, device, dtype, requires_grad, **kwargs):
    for sample_input in sample_inputs_unary_dimwise(
        op_info, device, dtype, requires_grad, **kwargs
    ):
        yield sample_input

        last_dim_sample = _update_sample(sample_input, {"dim": -1})
        last_dim_sample.name = (
            f"{_describe_njt(last_dim_sample.input)}: add dim to the end"
        )
        # Tell the unbind reference how to canonicalize the dim kwargs
        # This is necessary because unsqueeze() allows for a dim after
        # the last dim to indicate an unsqueeze at the end.
        last_dim_sample.input._ndim = last_dim_sample.input.dim() + 1
        yield last_dim_sample

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_unsqueeze`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_unsqueeze`。

### Lines 1495-1506
```python

def sample_inputs_where(op_info, device, dtype, requires_grad, **kwargs):
    for sample in sample_inputs_elementwise_njt_binary(
        op_info, device, dtype, requires_grad, **kwargs
    ):
        other = sample.args[0]
        sample.args = ()
        sample.kwargs["other"] = other
        sample.kwargs["condition"] = sample.input > 0.0
        sample.name = sample.name.replace("(", "(NT, ")
        yield sample

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_where`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_where`。

### Lines 1507-1530
```python

# === END OP-SPECIFIC SAMPLE INPUTS FUNCS / REFERENCES ===


# Mapping of OpInfo full names -> sample_inputs_funcs, which define the set of sample inputs
# (involving NJTs) to pass to the op. Full name consists of the OpInfo's name and variant name
# separated by a period (e.g. special.polygamma.special_polygamma_n_0). These are necessary
# to specify if they cannot be auto-generated for some reason. Try to keep these sorted
# in alphabetical order!
njt_sample_inputs = {
    "bmm": sample_inputs_bmm,
    "chunk": sample_inputs_chunk,
    "clone": sample_inputs_clone,
    "count_nonzero": partial(sample_inputs_njt_reduction, supports_keepdim=False),
    "fill": sample_inputs_fill,
    **{f"mvlgamma.mvlgamma_p_{p}": sample_inputs_mvl_gamma(p=1) for p in (1, 3, 5)},
    "nn.functional.embedding": sample_inputs_nn_functional_embedding,
    "nn.functional.embedding_bag": sample_inputs_nn_functional_embedding_bag,
    "nn.functional.linear": sample_inputs_nn_functional_linear,
    "nn.functional.prelu": sample_inputs_nn_functional_prelu,
    "nn.functional.rms_norm": sample_inputs_nn_functional_rms_norm,
    "nn.functional.threshold": sample_inputs_nn_functional_threshold,
    **{f"polygamma.polygamma_n_{n}": sample_inputs_polygamma_n(n=n) for n in range(5)},
    "special.polygamma.special_polygamma_n_0": sample_inputs_special_polygamma_n(n=0),
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1531-1547
```python
    "to": sample_inputs_to,
    "matmul": sample_inputs_matmul,
    "masked_select": sample_inputs_masked_select,
    "narrow": sample_inputs_narrow,
    "index_put": sample_inputs_index_put,
    # these two don't have ReductionOpInfo entries
    "max.reduction_with_dim": sample_inputs_njt_reduction,
    "min.reduction_with_dim": sample_inputs_njt_reduction,
    "select": sample_inputs_select,
    "split": sample_inputs_split,
    "split_with_sizes": sample_inputs_split_with_sizes,
    "squeeze": sample_inputs_squeeze,
    "unflatten": sample_inputs_unflatten,
    "unsqueeze": sample_inputs_unsqueeze,
    "where": sample_inputs_where,
}

```
- EN: This block implements local helper logic for nested. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 nested 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1548-1571
```python
njt_references = {
    "bmm": reference_bmm,
    "chunk": partial(
        unary_dimwise_reference, batchwise_reference=batchwise_reference_chunk
    ),
    "count_nonzero": reduction_reference,
    # these two don't have ReductionOpInfo entries
    "max.reduction_with_dim": reduction_reference,
    "min.reduction_with_dim": reduction_reference,
    "narrow": partial(
        unary_dimwise_reference, batchwise_reference=batchwise_reference_narrow
    ),
    "select": partial(
        unary_dimwise_reference, batchwise_reference=batchwise_reference_select
    ),
    "split": partial(
        unary_dimwise_reference, batchwise_reference=batchwise_reference_split
    ),
    "split_with_sizes": partial(
        unary_dimwise_reference,
        batchwise_reference=batchwise_reference_split_with_sizes,
    ),
    "squeeze": unbind_reference,
    "nn.functional.embedding_bag": reference_nn_functional_embedding_bag,
```
- EN: This block implements local helper logic for nested. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 nested 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1572-1587
```python
    "unflatten": partial(
        unary_dimwise_reference, batchwise_reference=batchwise_reference_unflatten
    ),
    "unsqueeze": partial(
        unary_dimwise_reference, batchwise_reference=batchwise_reference_unsqueeze
    ),
}


# Translates an OpInfo entry to one that operates on NJTs.
def translate_opinfo(op):
    new_op = copy(op)
    new_op.supports_njt = True
    # add some extra info for use in generating tests on the right subset of ops
    new_op._extra_op_data = extra_op_data.get(op.full_name, ExtraOpData())

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `translate_opinfo`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`translate_opinfo`。

### Lines 1588-1609
```python
    if op.full_name in njt_sample_inputs:
        new_op.sample_inputs_func = njt_sample_inputs[op.full_name]
        new_op.ref = njt_references.get(op.full_name, unbind_reference)
    elif isinstance(op, UnaryUfuncInfo):
        new_op.sample_inputs_func = partial(
            sample_inputs_elementwise_njt_unary, op_kwargs=None
        )
        new_op.ref = unbind_reference
    elif isinstance(op, BinaryUfuncInfo):
        new_op.sample_inputs_func = partial(
            sample_inputs_elementwise_njt_binary, op_kwargs=None
        )
        new_op.ref = unbind_reference
    elif isinstance(op, ReductionOpInfo):
        new_op.sample_inputs_func = partial(sample_inputs_njt_reduction, op_kwargs=None)
        new_op.ref = reduction_reference
    # TODO: Translate the rest of the OpInfos
    else:
        new_op.sample_inputs_func = unsupported_sample_inputs_func(op.full_name)
        new_op.ref = unsupported_reference(op.full_name)
        new_op.supports_njt = False

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 1610-1613
```python
    return new_op


njt_op_db = [translate_opinfo(op) for op in op_db]
```
- EN: This block iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.fx.experimental.symbolic_shapes`, `torch.testing._internal.common_methods_invocations`, `torch.testing._internal.opinfo.core`, `torch.utils._pytree`, `torch.nested._internal.ops`
- External imports / 外部导入: `math`, `copy`, `dataclasses`, `functools`
- Representative symbols / 代表性符号: `ExtraOpData`, `_rnd`, `_raggedness_matches`, `_clone`, `_update_sample`, `random_nt_from_dims`, `_describe_njt`, `_describe_dim`, `_sample_njts`, `unbind_reference`, `...`
