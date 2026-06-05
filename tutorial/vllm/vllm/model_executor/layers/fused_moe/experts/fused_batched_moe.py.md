# fused_batched_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/fused_batched_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fused batched MoE kernel. / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 5-34 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
)
from vllm.model_executor.layers.fused_moe.fused_moe import try_get_optimal_moe_config
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceDelegate,
)
from vllm.model_executor.layers.fused_moe.utils import (
    _resize_cache,
    moe_kernel_quantize_input,
    normalize_batched_scales_shape,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    group_broadcast,
    kFp8Dynamic128Sym,
    kFp8DynamicTensorSym,
    kFp8DynamicTokenSym,
    kFp8Static128BlockSym,
    kFp8StaticChannelSym,
    kFp8StaticTensorSym,
)
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 37-151 — function `moe_mmk`
```python
@triton.jit
def moe_mmk(
    a_ptrs,
    b_ptrs,
    K,
    expert_id,
    a_scale_ptr,
    b_scale_ptr,
    # The stride variables represent how much to increase the ptr by when
    # moving by 1 element in a particular dimension. E.g. `stride_am` is
    # how much to increase `a_ptr` by to get the element one row down
    # (A has M rows).
    stride_ak: tl.int64,
    stride_bk: tl.int64,
    stride_ase: tl.int64,
    stride_asm: tl.int64,
    stride_ask: tl.int64,
    stride_bse: tl.int64,
    stride_bsk: tl.int64,
    stride_bsn: tl.int64,
    # Offsets and masks
    offs_m,
    offs_n,
    offs_bn,
    mask_m,
    # Block size for block-wise quantization
    group_n: tl.constexpr,
    group_k: tl.constexpr,
    # Meta-parameters
    BLOCK_M: tl.constexpr,
    BLOCK_N: tl.constexpr,
    BLOCK_K: tl.constexpr,
    compute_type: tl.constexpr,
    use_w8a8: tl.constexpr,
    use_w8a16: tl.constexpr,
    per_act_token_quant: tl.constexpr,
):
    offs_k = tl.arange(0, BLOCK_K)

    if use_w8a16:
        b_scale_ptrs = (
            b_scale_ptr + expert_id * stride_bse + offs_n[None, :] * stride_bsn
        )
        b_scale = tl.load(b_scale_ptrs)

    if use_w8a8:
        # block-wise
        if group_k > 0 and group_n > 0:
            a_scale_ptrs = a_scale_ptr + offs_m * stride_asm
            offs_bsn = offs_bn // group_n
            b_scale_ptrs = b_scale_ptr + offs_bsn * stride_bsn

        # per act token
        elif per_act_token_quant:
            # Load per-token scale for activations
            a_scale_ptrs = a_scale_ptr + offs_m * stride_asm
# ... omitted for brevity ...

    return accumulator
```
**EN:** This function defines `moe_mmk`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `a_ptrs`, `b_ptrs`, `K`, `expert_id`, `a_scale_ptr`, `b_scale_ptr`. Key calls include `tl.arange`, `tl.zeros`, `range`, `tl.load`, `tl.cdiv`, `to`. It writes or updates `offs_k`, `accumulator`, `b_scale_ptrs`, `b_scale`, `a`, `b`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `moe_mmk`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `a_ptrs`, `b_ptrs`, `K`, `expert_id`, `a_scale_ptr`, `b_scale_ptr`。 关键调用包括 `tl.arange`, `tl.zeros`, `range`, `tl.load`, `tl.cdiv`, `to`。 它会写入或更新 `offs_k`, `accumulator`, `b_scale_ptrs`, `b_scale`, `a`, `b`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 154-246 — function `expert_triton_kernel`
```python
@triton.jit
def expert_triton_kernel(
    a_ptr,  # [max_tokens, K]
    b_ptr,  # [K, N]
    c_ptr,  # [max_tokens, N]
    expert_id,
    compute_type: tl.constexpr,
    # Dimensions
    M,
    N,
    K,
    # Quantization data
    a_scale_ptr,
    b_scale_ptr,
    b_zp_ptr,
    # strides
    stride_am: tl.int64,
    stride_ak: tl.int64,
    stride_bk: tl.int64,
    stride_bn: tl.int64,
    stride_cm: tl.int64,
    stride_cn: tl.int64,
    stride_ase: tl.int64,
    stride_asm: tl.int64,
    stride_ask: tl.int64,
    stride_bse: tl.int64,
    stride_bsk: tl.int64,
    stride_bsn: tl.int64,
    # offsets
    offs_bn,
    # Blockwise quantization data
    group_n,
    group_k,
    # Quantization schemes
    use_fp8_w8a8: tl.constexpr,
    use_int8_w8a16: tl.constexpr,
    per_act_token_quant: tl.constexpr,
    # Kernel config
    BLOCK_M: tl.constexpr,
    BLOCK_N: tl.constexpr,
    BLOCK_K: tl.constexpr,
):
    offs_m = tl.arange(0, BLOCK_M)
    offs_n = tl.arange(0, BLOCK_N) % N
    offs_k = tl.arange(0, BLOCK_K)
    mask_m = offs_m < M

    # Make grids of a + b pointers
    a_ptrs = a_ptr + offs_m[:, None] * stride_am + offs_k[None, :] * stride_ak
    b_ptrs = b_ptr + offs_k[:, None] * stride_bk + offs_n[None, :] * stride_bn

    accumulator = moe_mmk(
        a_ptrs,
        b_ptrs,
        K,
        expert_id,
# ... omitted for brevity ...
    c_mask = mask_m[:, None] & (offs_cn[None, :] < N)
    tl.store(c_ptrs, accumulator, mask=c_mask)
```
**EN:** This function defines `expert_triton_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `a_ptr`, `b_ptr`, `c_ptr`, `expert_id`, `compute_type`, `M`. Key calls include `tl.arange`, `moe_mmk`, `tl.store`. It writes or updates `offs_m`, `offs_n`, `offs_k`, `mask_m`, `a_ptrs`, `b_ptrs`.
**CN:** 该函数定义 `expert_triton_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `a_ptr`, `b_ptr`, `c_ptr`, `expert_id`, `compute_type`, `M`。 关键调用包括 `tl.arange`, `moe_mmk`, `tl.store`。 它会写入或更新 `offs_m`, `offs_n`, `offs_k`, `mask_m`, `a_ptrs`, `b_ptrs`。

### Lines 249-374 — function `batched_triton_kernel`
```python
@triton.jit
def batched_triton_kernel(
    a_ptr,  # [E, max_num_tokens, K]
    b_ptr,  # [E, K, N]
    c_ptr,  # [E, max_num_tokens, N]
    expert_num_tokens,  # [E]
    compute_type: tl.constexpr,
    # Dimensions
    max_num_tokens,
    K,
    N,
    # Quantization data
    a_scale_ptr,
    b_scale_ptr,
    b_zp_ptr,
    # The stride variables represent how much to increase the ptr by when
    # moving by 1 element in a particular dimension. E.g. `stride_am` is
    # how much to increase `a_ptr` by to get the element one row down
    # (A has M rows).
    stride_ae: tl.int64,
    stride_am: tl.int64,
    stride_ak: tl.int64,
    stride_be: tl.int64,
    stride_bk: tl.int64,
    stride_bn: tl.int64,
    stride_ce: tl.int64,
    stride_cm: tl.int64,
    stride_cn: tl.int64,
    stride_ase: tl.int64,
    stride_asm: tl.int64,
    stride_ask: tl.int64,
    stride_bse: tl.int64,
    stride_bsk: tl.int64,
    stride_bsn: tl.int64,
    # Blockwise quantization data
    group_n: tl.constexpr,
    group_k: tl.constexpr,
    # Quantization schemes
    use_fp8_w8a8: tl.constexpr,
    use_int8_w8a16: tl.constexpr,
    per_act_token_quant: tl.constexpr,
    # Kernel config
    BLOCK_M: tl.constexpr,
    BLOCK_N: tl.constexpr,
    BLOCK_K: tl.constexpr,
):
    expert_id = tl.program_id(axis=0)
    e_num_tokens = tl.load(expert_num_tokens + expert_id)
    if e_num_tokens == 0:
        # Early exit
        return

    # axis 1 is M_blocks * N_blocks
    pid_mn = tl.program_id(axis=1)
    # num_pid_m = tl.cdiv(max_num_tokens, BLOCK_M)
    num_pid_n = tl.cdiv(N, BLOCK_N)
# ... omitted for brevity ...
        BLOCK_K,
    )
```
**EN:** This function defines `batched_triton_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `a_ptr`, `b_ptr`, `c_ptr`, `expert_num_tokens`, `compute_type`, `max_num_tokens`. Key calls include `tl.program_id`, `tl.load`, `tl.cdiv`, `min`, `expert_triton_kernel`, `tl.arange.to`. It writes or updates `expert_id`, `e_num_tokens`, `pid_mn`, `num_pid_n`, `pid_m`, `pid_n`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `batched_triton_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `a_ptr`, `b_ptr`, `c_ptr`, `expert_num_tokens`, `compute_type`, `max_num_tokens`。 关键调用包括 `tl.program_id`, `tl.load`, `tl.cdiv`, `min`, `expert_triton_kernel`, `tl.arange.to`。 它会写入或更新 `expert_id`, `e_num_tokens`, `pid_mn`, `num_pid_n`, `pid_m`, `pid_n`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 377-487 — function `invoke_moe_batched_triton_kernel`
```python
def invoke_moe_batched_triton_kernel(
    A: torch.Tensor,  # [E, max_tokens, K]
    B: torch.Tensor,  # [E, N, K]
    C: torch.Tensor,  # [E, max_tokens, N]
    expert_num_tokens: torch.Tensor,  # [E]
    compute_type: tl.dtype,
    # Quantization data
    A_scale: torch.Tensor | None,
    B_scale: torch.Tensor | None,
    B_zp: torch.Tensor,
    # Quantization schemes
    use_fp8_w8a8: bool,
    use_int8_w8a16: bool,
    use_int4_w4a16: bool,
    config: dict[str, int],
    per_act_token_quant: bool,
    block_shape: list[int] | None = None,
):
    assert not use_int4_w4a16
    max_num_tokens = A.size(1)
    K = A.size(2)
    N = C.size(2)

    BLOCK_M = config["BLOCK_SIZE_M"]
    BLOCK_N = config["BLOCK_SIZE_N"]
    BLOCK_K = config["BLOCK_SIZE_K"]

    grid = (
        expert_num_tokens.size(0),
        triton.cdiv(max_num_tokens, BLOCK_M) * triton.cdiv(B.size(1), BLOCK_N),
    )

    A_scale = normalize_batched_scales_shape(A_scale, expert_num_tokens.shape[0])

    if B_scale is not None and B_scale.ndim == 1:
        assert B_scale.numel() == expert_num_tokens.shape[0]
        B_scale = B_scale.view(-1, 1, 1)

    assert A_scale is None or A_scale.ndim == 3, (
        f"{0 if A_scale is None else A_scale.shape}"
    )
    assert B_scale is None or B_scale.ndim == 1 or B_scale.ndim == 3, (
        f"{0 if B_scale is None else B_scale.shape}"
    )
# ... omitted for brevity ...
        BLOCK_K=BLOCK_K,
    )
```
**EN:** This function defines `invoke_moe_batched_triton_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `A`, `B`, `C`, `expert_num_tokens`, `compute_type`, `A_scale`. Key calls include `A.size`, `C.size`, `normalize_batched_scales_shape`, `batched_triton_kernel`, `expert_num_tokens.size`, `B_scale.view`. It writes or updates `max_num_tokens`, `K`, `N`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `invoke_moe_batched_triton_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `A`, `B`, `C`, `expert_num_tokens`, `compute_type`, `A_scale`。 关键调用包括 `A.size`, `C.size`, `normalize_batched_scales_shape`, `batched_triton_kernel`, `expert_num_tokens.size`, `B_scale.view`。 它会写入或更新 `max_num_tokens`, `K`, `N`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 490-650 — class `NaiveBatchedExperts`
```python
class NaiveBatchedExperts(mk.FusedMoEExpertsModular):
    """
    A reference MoE expert class that operates on expert batched format,
    i.e. E x max_num_tokens x K.  This is the format that the batched
    dispatch/combine kernels use.
    """

    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int,
        num_dispatchers: int,
    ):
# ... omitted for brevity ...

            output[expert, :num, :] = tmp @ w2_dq.transpose(0, 1).to(tmp.dtype)
```
**EN:** This class defines `NaiveBatchedExperts`. It inherits from `mk.FusedMoEExpertsModular`. A reference MoE expert class that operates on expert batched format, i.e. Important methods include `__init__`, `apply`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_shapes`. Key calls include `super.__init__`, `NotImplementedError`, `TopKWeightAndReduceDelegate`, `w1.size`, `range`, `hidden_states.dim`. It writes or updates `num_dp`, `num_experts`, `workspace13`, `workspace2`, `output`, `f32`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该类定义了 `NaiveBatchedExperts`。 它继承自 `mk.FusedMoEExpertsModular`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `apply`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_shapes`。 关键调用包括 `super.__init__`, `NotImplementedError`, `TopKWeightAndReduceDelegate`, `w1.size`, `range`, `hidden_states.dim`。 它会写入或更新 `num_dp`, `num_experts`, `workspace13`, `workspace2`, `output`, `f32`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 497-513 — method `NaiveBatchedExperts.__init__`
```python
    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int,
        num_dispatchers: int,
    ):
        super().__init__(
            moe_config=moe_config,
            quant_config=quant_config,
            max_num_tokens=max_num_tokens,
            num_dispatchers=num_dispatchers,
        )
        assert not self.quant_config.use_int8_w8a8, "NYI"
        assert not self.quant_config.use_int8_w8a16, "NYI"
        assert not self.quant_config.use_int4_w4a16, "NYI"
        assert self.quant_config.ocp_mx_scheme is None, "NYI"
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`. Key calls include `super.__init__`, `super`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `moe_config`, `quant_config`, `max_num_tokens`, `num_dispatchers`。 关键调用包括 `super.__init__`, `super`。

### Lines 592-650 — method `NaiveBatchedExperts.apply`
```python
    def apply(
        self,
        output: torch.Tensor,
        hidden_states: torch.Tensor,
        w1: torch.Tensor,
        w2: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        activation: MoEActivation,
        global_num_experts: int,
        expert_map: torch.Tensor | None,
        a1q_scale: torch.Tensor | None,
        a2_scale: torch.Tensor | None,
        workspace13: torch.Tensor,
        workspace2: torch.Tensor,
        expert_tokens_meta: mk.ExpertTokensMetadata | None,
        apply_router_weight_on_input: bool,
    ):
        assert hidden_states.dim() == 3
        assert expert_tokens_meta is not None
        expert_num_tokens = expert_tokens_meta.expert_num_tokens

        num_local_experts = w1.size(0)
        assert num_local_experts == w1.size(0), f"{num_local_experts} == {w1.size(0)}"

        N = w1.size(1) // 2

        for expert in range(num_local_experts):
            # Indexing expert_num_tokens doesn't work w/cudagraphs or inductor
            if (
                torch.compiler.is_compiling()
                or torch.cuda.is_current_stream_capturing()
            ):
                num = hidden_states.shape[1]
            else:
                num = int(expert_num_tokens[expert].item())

            if num == 0:
                continue

            tmp = _resize_cache(workspace2, (num, N))

            if self.quant_config.is_quantized:
                assert a1q_scale is not None and self.w1_scale is not None
# ... omitted for brevity ...

            output[expert, :num, :] = tmp @ w2_dq.transpose(0, 1).to(tmp.dtype)
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `w1.size`, `range`, `hidden_states.dim`, `_resize_cache`, `self.activation`, `torch.compiler.is_compiling`. It writes or updates `expert_num_tokens`, `num_local_experts`, `N`, `tmp`, `num`, `input`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `w1.size`, `range`, `hidden_states.dim`, `_resize_cache`, `self.activation`, `torch.compiler.is_compiling`。 它会写入或更新 `expert_num_tokens`, `num_local_experts`, `N`, `tmp`, `num`, `input`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 515-517 — method `NaiveBatchedExperts.activation_format`
```python
    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.BatchedExperts
```
**EN:** This method defines `activation_format`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `activation_format`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 557-558 — method `NaiveBatchedExperts.supports_expert_map`
```python
    def supports_expert_map(self) -> bool:
        return False
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 560-562 — method `NaiveBatchedExperts.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        # Let PrepareAndFinalize::finalize() decide the impl.
        return TopKWeightAndReduceDelegate()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceDelegate`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceDelegate`。

### Lines 564-582 — method `NaiveBatchedExperts.workspace_shapes`
```python
    def workspace_shapes(
        self,
        M: int,
        N: int,
        K: int,
        topk: int,
        global_num_experts: int,
        local_num_experts: int,
        expert_tokens_meta: mk.ExpertTokensMetadata | None,
        activation: MoEActivation,
    ) -> tuple[tuple[int, ...], tuple[int, ...], tuple[int, ...]]:
        assert self.num_dispatchers is not None
        assert self.max_num_tokens is not None
        num_dp = self.num_dispatchers
        num_experts = local_num_experts
        workspace13 = (num_experts, self.max_num_tokens * num_dp, K)
        workspace2 = (self.max_num_tokens * num_dp, N)
        output = workspace13
        return (workspace13, workspace2, output)
```
**EN:** This method defines `workspace_shapes`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`. It writes or updates `num_dp`, `num_experts`, `workspace13`, `workspace2`, `output`.
**CN:** 该方法定义 `workspace_shapes`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `M`, `N`, `K`, `topk`, `global_num_experts`, `local_num_experts`。 它会写入或更新 `num_dp`, `num_experts`, `workspace13`, `workspace2`, `output`。

### Lines 653-716 — function `batched_moe_kernel_quantize_input`
```python
def batched_moe_kernel_quantize_input(
    A: torch.Tensor,
    A_scale: torch.Tensor | None,
    num_tokens: int,
    E: int,
    N: int,
    expert_num_tokens: torch.Tensor,
    qtype: torch.dtype | None,
    per_act_token_quant: bool,
    block_shape: list[int] | None = None,
) -> tuple[torch.Tensor, torch.Tensor | None]:
    if torch.compiler.is_compiling() or torch.cuda.is_current_stream_capturing():
        # Note: this does a bunch of extra work because expert_num_tokens is
        # ignored but it does support torch.compile + cudagraphs.
        hidden_dim = A.size(-1)
        assert A_scale is None or A_scale.ndim <= 2, (
            f"{A_scale.shape if A_scale is not None else None}"
        )
        A_q, A_q_scale = moe_kernel_quantize_input(
            A.view(-1, hidden_dim), A_scale, qtype, per_act_token_quant, block_shape
        )
        A_q = A_q.view(E, -1, hidden_dim)
        A_q_scale = normalize_batched_scales_shape(A_q_scale, E)

        return A_q, A_q_scale
    elif qtype is None:
        return A, normalize_batched_scales_shape(A_scale, E)
    else:
        A_q = torch.empty_like(A, dtype=qtype)

        if per_act_token_quant:
            assert block_shape is None
            scale_shape = (E, num_tokens, 1)
        elif block_shape is not None:
            _, block_k = block_shape
            k_tiles = (A.shape[-1] + block_k - 1) // block_k
            scale_shape = (E, num_tokens, k_tiles)
        else:
            scale_shape = (E, 1, 1)

        A_q_scale = torch.zeros(scale_shape, dtype=torch.float32, device=A.device)

        num_experts = expert_num_tokens.numel()

# ... omitted for brevity ...

        return A_q, A_q_scale
```
**EN:** This function defines `batched_moe_kernel_quantize_input`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `A`, `A_scale`, `num_tokens`, `E`, `N`, `expert_num_tokens`. Key calls include `torch.compiler.is_compiling`, `torch.cuda.is_current_stream_capturing`, `A.size`, `moe_kernel_quantize_input`, `A_q.view`, `normalize_batched_scales_shape`. It writes or updates `hidden_dim`, `A_q`, `A_q_scale`, `num_experts`, `A_scale`, `scale_shape`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `batched_moe_kernel_quantize_input`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `A`, `A_scale`, `num_tokens`, `E`, `N`, `expert_num_tokens`。 关键调用包括 `torch.compiler.is_compiling`, `torch.cuda.is_current_stream_capturing`, `A.size`, `moe_kernel_quantize_input`, `A_q.view`, `normalize_batched_scales_shape`。 它会写入或更新 `hidden_dim`, `A_q`, `A_q_scale`, `num_experts`, `A_scale`, `scale_shape`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 719-967 — class `BatchedTritonExperts`
```python
class BatchedTritonExperts(mk.FusedMoEExpertsModular):
    """
    A Triton based MoE expert class that operates on expert batched format,
    i.e. E x max_num_tokens x K.  This is the format that the batched
    dispatch/combine kernels use.
    """

    def __init__(
        self,
        moe_config: FusedMoEConfig,
        quant_config: FusedMoEQuantConfig,
        max_num_tokens: int,
        num_dispatchers: int,
    ):
# ... omitted for brevity ...
            block_shape=self.block_shape,
        )
```
**EN:** This class defines `BatchedTritonExperts`. It inherits from `mk.FusedMoEExpertsModular`. A Triton based MoE expert class that operates on expert batched format, i.e. Important methods include `__init__`, `apply`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_shapes`. Key calls include `super.__init__`, `current_platform.is_cuda_alike`, `p.is_rocm`, `TopKWeightAndReduceDelegate`, `self.adjust_N_for_activation`, `hidden_states.is_contiguous`. It writes or updates `p`, `device_supports_fp8`, `supported`, `num_dp`, `num_experts`, `max_num_tokens`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `BatchedTritonExperts`。 它继承自 `mk.FusedMoEExpertsModular`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `apply`, `activation_format`, `supports_expert_map`, `finalize_weight_and_reduce_impl`, `workspace_shapes`。 关键调用包括 `super.__init__`, `current_platform.is_cuda_alike`, `p.is_rocm`, `TopKWeightAndReduceDelegate`, `self.adjust_N_for_activation`, `hidden_states.is_contiguous`。 它会写入或更新 `p`, `device_supports_fp8`, `supported`, `num_dp`, `num_experts`, `max_num_tokens`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `moe_mmk`, `expert_triton_kernel`, `batched_triton_kernel`, `invoke_moe_batched_triton_kernel` / [CN] 核心符号：`moe_mmk`, `expert_triton_kernel`, `batched_triton_kernel`, `invoke_moe_batched_triton_kernel`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.triton_utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.triton_utils`
- **Runtime traits**: platform-aware dispatch, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, Triton kernels, distributed collectives
