# xpu_mla_sparse.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/ops/xpu_mla_sparse.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_bf16_mla_sparse_kernel`, `triton_bf16_mla_sparse_interface` for the V1 `attention/ops` subsystem. / 为 V1 的 `attention/ops` 子系统实现 `_bf16_mla_sparse_kernel`, `triton_bf16_mla_sparse_interface`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import torch

from vllm.triton_utils import LOG2E, LOGE2, tl, triton
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.triton_utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.triton_utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `_bf16_mla_sparse_kernel` function / `_bf16_mla_sparse_kernel` 函数
```python
@triton.jit
def _bf16_mla_sparse_kernel(
    q_buffer,
    k_buffer,
    v_buffer,
    indices_ptr,
    out_ptr,
    softmax_lse_ptr,
    max_logits_ptr,
    seq_q,
    seq_kv,
    h_q,
    dim_qk,
    dim_v,
    stride_q_token,
    stride_q_head,
    stride_k_token,
    stride_k_head,
    stride_v_token,
    stride_v_head,
    stride_out_token,
    stride_out_head,
    stride_lse,
    stride_indices_token,
    stride_indices_head,
    sm_scale,
    kv_group_num: tl.constexpr,
    index_topk: tl.constexpr,
    BLOCK_H: tl.constexpr,  # block size for num heads
    BLOCK_M: tl.constexpr,  # block size for num tokens
    BLOCK_N: tl.constexpr,  # block size for indices
    BLOCK_DV: tl.constexpr,  # block size for dim_v
    BLOCK_DMODEL: tl.constexpr,  # block size for dim_nope
    BLOCK_DPE: tl.constexpr,  # block size for positional embedding
    LOGE2: tl.constexpr,
):
    cur_q = tl.program_id(0)
    cur_head_id = tl.program_id(1)
    cur_kv_head_id = cur_head_id // tl.cdiv(kv_group_num, BLOCK_H)

    VALID_BLOCK_H: tl.constexpr = BLOCK_H if kv_group_num > BLOCK_H else kv_group_num
    cur_head = cur_head_id * VALID_BLOCK_H + tl.arange(0, BLOCK_H)
    mask_h = cur_head < (cur_head_id + 1) * VALID_BLOCK_H
    mask_h = mask_h & (cur_head < h_q)

    offs_d = tl.arange(0, BLOCK_DMODEL)
    offs_dv = tl.arange(0, BLOCK_DV)

    off_q = cur_q * stride_q_token + cur_head[:, None] * stride_q_head + offs_d[None, :]
    mask_dmodel = offs_d < BLOCK_DMODEL
# ... omitted for brevity ...
        e_max = n_e_max

    # rescaling
    acc /= e_sum[:, None]

    max_logits = e_max * LOGE2
    # calculate lse
    lse = max_logits + tl.log2(e_sum) * LOGE2

    # write output
    offs_o = (
        cur_q * stride_out_token
        + cur_head[:, None] * stride_out_head
        + offs_dv[None, :]
    )
    mask_out_d = offs_dv < dim_v
    tl.store(
        out_ptr + offs_o,
        acc.to(tl.bfloat16),
        mask=(mask_h[:, None]) & (mask_out_d[None, :]),
    )

    offs_lse = cur_q * stride_lse + cur_head
    tl.store(softmax_lse_ptr + offs_lse, lse, mask=mask_h)
    tl.store(max_logits_ptr + offs_lse, max_logits, mask=mask_h)
```
**EN:** This function implements `_bf16_mla_sparse_kernel` within the module. Key calls include `program_id`, `arange`, `load`, `zeros`, `range`, `store`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `_bf16_mla_sparse_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `arange`, `load`, `zeros`, `range`, `store`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `triton_bf16_mla_sparse_interface` function / `triton_bf16_mla_sparse_interface` 函数
```python
def triton_bf16_mla_sparse_interface(
    q: torch.Tensor,  # [num_tokens, num_heads_q, dim_qk]
    kv: torch.Tensor,  # [num_tokens, num_heads_kv, dim_qk]
    indices: torch.Tensor,  # [num_tokens, num_heads_kv, topk]
    sm_scale: float,
    d_v: int = 512,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    out : [num_tokens, num_heads_q, d_v]
    max_logits : [num_tokens, num_heads_q]
    lse : logsumexp, [num_tokens, num_heads_q]
    """
    num_tokens, num_heads_q, dim_qk = q.shape
    _, num_heads_kv, _ = kv.shape
    assert dim_qk == kv.shape[2], "q and kv have different head dimensions"

    # for deepseek v3.2, index topk should be 2048
    _, _, index_topk = indices.shape

    BLOCK_H = 16
    BLOCK_DMODEL = 512
    BLOCK_DPE = 64
    BLOCK_M = 32
    BLOCK_N = 16
    BLOCK_DV = 512
    assert d_v == BLOCK_DV, "only support d_v = 512"

    assert dim_qk == BLOCK_DMODEL + BLOCK_DPE, (
        "dim_qk does not match BLOCK_DMODEL + BLOCK_DPE"
    )
    assert num_heads_kv == 1, "only support kv head = 1 for now"
    assert index_topk % BLOCK_N == 0, "index_topk must be multiple of BLOCK_N"

    sm_scale *= LOG2E

    kv_group_num = num_heads_q // num_heads_kv
    grid = (
        num_tokens,
        triton.cdiv(num_heads_q, min(BLOCK_H, kv_group_num)),
    )

    out = torch.zeros((num_tokens, num_heads_q, d_v), dtype=q.dtype, device=q.device)
    softmax_lse = torch.zeros(
        (num_tokens, num_heads_q), dtype=torch.float32, device=q.device
    )
    max_logits = torch.zeros(
        (num_tokens, num_heads_q), dtype=torch.float32, device=q.device
    )

    k = kv
    v = kv[..., :d_v]

    _bf16_mla_sparse_kernel[grid](
        q_buffer=q,
        k_buffer=k,
        v_buffer=v,
        indices_ptr=indices,
        out_ptr=out,
        softmax_lse_ptr=softmax_lse,
        max_logits_ptr=max_logits,
        seq_q=num_tokens,
        seq_kv=kv.shape[0],
        h_q=num_heads_q,
        dim_qk=dim_qk,
        dim_v=d_v,
        stride_q_token=q.stride(0),
        stride_q_head=q.stride(1),
        stride_k_token=k.stride(0),
        stride_k_head=k.stride(1),
        stride_v_token=v.stride(0),
        stride_v_head=v.stride(1),
        stride_out_token=out.stride(0),
        stride_out_head=out.stride(1),
        stride_lse=softmax_lse.stride(0),
        stride_indices_token=indices.stride(0),
        stride_indices_head=indices.stride(1),
        sm_scale=sm_scale,
        kv_group_num=kv_group_num,
        index_topk=index_topk,
        BLOCK_H=BLOCK_H,
        BLOCK_M=BLOCK_M,
        BLOCK_N=BLOCK_N,
        BLOCK_DV=BLOCK_DV,
        BLOCK_DMODEL=BLOCK_DMODEL,
        BLOCK_DPE=BLOCK_DPE,
        LOGE2=LOGE2,
    )

    return out, max_logits, softmax_lse
```
**EN:** This function implements `triton_bf16_mla_sparse_interface` within the module. The docstring frames it as: out : [num_tokens, num_heads_q, d_v] max_logits : [num_tokens, num_heads_q] lse : logsumexp, [num_tokens, num_heads_q] Key calls include `zeros`, `cdiv`, `min`, `stride`.
**CN:** 该函数会实现 `triton_bf16_mla_sparse_interface`，其作用域位于the module。 关键调用包括 `zeros`, `cdiv`, `min`, `stride`。

## Key Concepts / 关键概念
- `_bf16_mla_sparse_kernel`: top-level helper or orchestration entry point. / `_bf16_mla_sparse_kernel`：顶层辅助函数或编排入口。
- `triton_bf16_mla_sparse_interface`: top-level helper or orchestration entry point. / `triton_bf16_mla_sparse_interface`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.triton_utils`
