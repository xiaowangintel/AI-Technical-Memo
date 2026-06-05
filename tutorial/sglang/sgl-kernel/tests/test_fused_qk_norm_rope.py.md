# test_fused_qk_norm_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_fused_qk_norm_rope.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `fused qk normalization RoPE` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `fused qk normalization RoPE` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and module setup
````python
import pytest
import torch
from sgl_kernel import fused_qk_norm_rope as sgl_fused_qk_norm_rope

from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.rotary_embedding import get_rope
from sglang.srt.server_args import (
    ServerArgs,
    get_global_server_args,
    set_global_server_args_for_scheduler,
)
from sglang.srt.utils import (
    cpu_has_amx_support,
    is_cpu,
    is_cuda,
    is_hip,
    is_npu,
    is_xpu,
)
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 21-28: Constants and configuration
````python
_is_cuda = is_cuda()
_is_hip = is_hip()
_is_cpu = is_cpu()
_is_cpu_amx_available = cpu_has_amx_support()
_is_npu = is_npu()
_is_xpu = is_xpu()

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
````
**EN:** This block defines shared constants or configuration values such as `_is_cuda`, `_is_hip`, `_is_cpu`, `_is_cpu_amx_available`, `_is_npu`, `_is_xpu`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_is_cuda`, `_is_hip`, `_is_cpu`, `_is_cpu_amx_available`, `_is_npu`, `_is_xpu`），供后续函数或控制流程复用。

### Lines 31-123: `torch_ref_rms_norm_rope` definition
````python
@torch.inference_mode()
def torch_ref_rms_norm_rope(
    qkv,
    num_heads_q,
    num_heads_k,
    num_heads_v,
    head_dim,
    eps,
    q_weight,
    k_weight,
    base,
    is_neox,
    position_ids,
    partial_rotary_factor,
):
    """
    PyTorch reference implementation of RMSNorm+RoPE for verification.

    Uses SGLang's own RMSNorm and RotaryEmbedding modules to ensure consistency
    with the expected behavior of the fused kernel.

    Args:
        qkv: Combined QKV tensor of shape [num_tokens, hidden_size]
        num_heads_q: Number of query heads
        num_heads_k: Number of key heads
        num_heads_v: Number of value heads (unused for normalization/RoPE but needed for tensor splitting)
        head_dim: Dimension of each head
        eps: Epsilon value for RMS normalization
        q_weight: RMSNorm weights for query [head_dim]
        k_weight: RMSNorm weights for key [head_dim]
        base: Base value for RoPE calculations
        is_neox: Whether to use NeoX style RoPE
        position_ids: Position IDs for RoPE of shape [num_tokens]
        partial_rotary_factor: Partial rotary factor

    Returns:
        Combined tensor with Q and K parts normalized and RoPE applied
    """
    # Get input shape information
    num_tokens = qkv.shape[0]
    hidden_size = qkv.shape[1]

    # Calculate dimensions for Q, K, V segments
    q_size = num_heads_q * head_dim
    k_size = num_heads_k * head_dim
    v_size = num_heads_v * head_dim

    # Verify dimensions match
    assert (
        hidden_size == q_size + k_size + v_size
    ), f"Hidden size {hidden_size} doesn't match Q+K+V dimensions {q_size + k_size + v_size}"

    # Split the tensor into Q, K, V parts
    q = qkv[:, :q_size]
    k = qkv[:, q_size : q_size + k_size]
    v = qkv[:, q_size + k_size :]

    rotary_emb = get_rope(
        head_dim,
        rotary_dim=head_dim,
        max_position=8192,
        base=10000,
        is_neox_style=is_neox,
        rope_scaling=None,
        dual_chunk_attention_config=None,
        partial_rotary_factor=partial_rotary_factor,
    )
    rotary_emb = rotary_emb.to(qkv.device)

    # Create and apply RMSNorm modules with custom weights
    q_norm = RMSNorm(hidden_size=head_dim, eps=eps).to(qkv.device).to(qkv.dtype)
    q_norm.weight.data.copy_(q_weight)
    k_norm = RMSNorm(hidden_size=head_dim, eps=eps).to(qkv.device).to(qkv.dtype)
    k_norm.weight.data.copy_(k_weight)

    q_by_head = q.reshape(-1, head_dim)
    q_by_head = q_norm(q_by_head)
    k_by_head = k.reshape(-1, head_dim)
    k_by_head = k_norm(k_by_head)
    q = q_by_head.view(q.shape)
    k = k_by_head.view(k.shape)

    [q_rope, k_rope] = rotary_emb(
        position_ids,
        q,
        k,
        fused_set_kv_buffer_arg=None,
    )

    # Combine Q, K, V back together
    result = torch.cat([q_rope, k_rope, v], dim=1)

    return result
````
**EN:** This section defines `torch_ref_rms_norm_rope` and implements the core logic associated with torch ref rms normalization RoPE. Docstring summary: PyTorch reference implementation of RMSNorm+RoPE for verification. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `torch_ref_rms_norm_rope`，并实现与 torch ref rms normalization RoPE 相关的核心逻辑。 文档字符串摘要：PyTorch reference implementation of RMSNorm+RoPE for verification. 它还会在继续执行前进行显式断言或形状检查。

### Lines 126-138: Constants and configuration
````python
head_dims = [64, 128]
# (Q heads, K heads, V heads)
num_heads_groups = [
    (16, 8, 8),  # Qwen3-0.6B, Qwen3-1.7B
    (32, 8, 8),  # Qwen3-4B, Qwen3-8B, Qwen3-30B-A3B
    (40, 8, 8),  # Qwen3-14B
    (64, 8, 8),  # Qwen3-32B, Qwen3-235B-A22B
    (12, 1, 1),  # GLM4.6 TP8
]
num_tokens_list = [1, 3, 8, 32, 256]
is_neox_list = [False, True]
dtypes = [torch.bfloat16]
partial_rotary_factor_list = [1.0, 0.5]
````
**EN:** This block defines shared constants or configuration values such as `head_dims`, `num_heads_groups`, `num_tokens_list`, `is_neox_list`, `dtypes`, `partial_rotary_factor_list`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `head_dims`, `num_heads_groups`, `num_tokens_list`, `is_neox_list`, `dtypes`, `partial_rotary_factor_list`），供后续函数或控制流程复用。

### Lines 141-235: `test_fused_qk_norm_rope` definition
````python
@pytest.mark.skipif(not _is_cuda, reason="Skipping CUDA/ROCm only tests.")
@pytest.mark.parametrize("head_dim", head_dims)
@pytest.mark.parametrize("num_heads_group", num_heads_groups)
@pytest.mark.parametrize("num_tokens", num_tokens_list)
@pytest.mark.parametrize("is_neox", is_neox_list)
@pytest.mark.parametrize("dtype", dtypes)
@pytest.mark.parametrize("partial_rotary_factor", partial_rotary_factor_list)
def test_fused_qk_norm_rope(
    head_dim, num_heads_group, num_tokens, is_neox, dtype, partial_rotary_factor
):
    """
    Test the fused QK RMSNorm + RoPE operation with various configurations.

    This test verifies that the fused kernel correctly applies:
    1. RMSNorm to both query (Q) and key (K) portions of the QKV tensor
    2. Rotary Position Embeddings (RoPE) to the normalized Q and K
    3. Leaves the value (V) portion unchanged

    Args:
        head_dim: Dimension of each attention head
        num_heads_group: Tuple of (num_heads_q, num_heads_k, num_heads_v)
        num_tokens: Number of tokens to process
        dtype: Data type (float16 or bfloat16)
    """
    set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))
    device = "cuda"
    torch_dtype = dtype

    # Unpack head counts
    num_heads_q, num_heads_k, num_heads_v = num_heads_group

    # Calculate total hidden dimension
    hidden_size = (num_heads_q + num_heads_k + num_heads_v) * head_dim

    # Generate random inputs directly as 2D [num_tokens, hidden_size]
    torch.random.manual_seed(0)
    qkv = torch.randn(num_tokens, hidden_size, dtype=torch_dtype, device=device)
    qkv_copy = qkv.clone()

    # Generate position IDs with +100 offset to test decoding scenarios
    position_ids = torch.arange(num_tokens, dtype=torch.int32, device=device) + 100

    # Generate random weights for RMSNorm
    q_weight = torch.randn(head_dim, dtype=torch_dtype, device=device) * 5.0
    k_weight = torch.randn(head_dim, dtype=torch_dtype, device=device) * 5.0

    # Set RMSNorm and RoPE parameters
    eps = 1e-5
    base = 10000.0

    factor, low, high, attention_factor = 1.0, 0, 0, 1.0
    # Run the custom fusedQKNormRope operation
    sgl_fused_qk_norm_rope(
        qkv,
        num_heads_q,
        num_heads_k,
        num_heads_v,
        head_dim,
        eps,
        q_weight,
        k_weight,
        base,
        is_neox,
        position_ids,
        factor,
        low,
        high,
        attention_factor,
        int(head_dim * partial_rotary_factor),
    )
    output = qkv  # This op is inplace

    # Compute reference output using TensorRT-LLM modules
    ref_output = torch_ref_rms_norm_rope(
        qkv_copy,
        num_heads_q,
        num_heads_k,
        num_heads_v,
        head_dim,
        eps,
        q_weight,
        k_weight,
        base,
        is_neox,
        position_ids,
        partial_rotary_factor,
    )

    # Compare outputs from custom kernel vs reference implementation
    torch.testing.assert_close(
        output,
        ref_output,
        rtol=5e-2,
        atol=1e-1,
    )
````
**EN:** This section defines the test `test_fused_qk_norm_rope`. It sets up inputs, runs the target path, and checks the expected result. Docstring summary: Test the fused QK RMSNorm + RoPE operation with various configurations. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_fused_qk_norm_rope`。它会准备输入、执行目标路径，并检查预期结果。 文档字符串摘要：Test the fused QK RMSNorm + RoPE operation with various configurations. 参数化装饰器会把覆盖范围扩展到多组输入场景。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `torch_ref_rms_norm_rope`, `test_fused_qk_norm_rope`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.srt.layers.layernorm`, `sglang.srt.layers.rotary_embedding`, `sglang.srt.server_args`, `sglang.srt.utils`
- **External / 外部**: `pytest`, `torch`
