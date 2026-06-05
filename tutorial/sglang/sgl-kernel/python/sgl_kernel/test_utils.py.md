# test_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/test_utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `utilities` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `utilities` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Imports and module setup
````python
import torch
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 4-62: `create_per_token_group_quant_test_data` definition
````python
def create_per_token_group_quant_test_data(num_tokens, hidden_dim, num_ranks, flags):
    device = torch.device("cuda")
    dtype = torch.bfloat16

    seed = num_tokens * 10000 + hidden_dim
    gen_cpu = torch.Generator(device="cpu")
    gen_cpu.manual_seed(seed)
    gen_cuda = torch.Generator(device="cuda")
    gen_cuda.manual_seed(seed)

    if flags["fuse_silu_and_mul"]:
        effective_hidden_dim = hidden_dim * 2
    else:
        effective_hidden_dim = hidden_dim
    del hidden_dim

    if (masked_layout_mode := flags["masked_layout_mode"]) is not None:
        num_max_dispatch_tokens_per_rank = 768
        num_global_experts = 288
        num_local_experts, remainder = divmod(num_global_experts, num_ranks)
        assert remainder == 0

        # mimic DeepEP low_latency_dispatch output
        x = torch.randn(
            num_local_experts,
            num_max_dispatch_tokens_per_rank * num_ranks,
            effective_hidden_dim,
            device=device,
            dtype=dtype,
            generator=gen_cuda,
        )

        if masked_layout_mode == "balanced":
            masked_m = _compute_balanced_split(num_tokens, num_local_experts)
        elif masked_layout_mode == "imbalanced":
            masked_m = _compute_imbalanced_split(
                num_tokens, num_local_experts, gen_cpu=gen_cpu
            )
        elif masked_layout_mode == "extreme":
            masked_m = torch.tensor(
                [num_tokens] + [0] * (num_local_experts - 1), dtype=torch.int
            )
        else:
            raise NotImplementedError
        print(f"{masked_layout_mode=} {masked_m=} {x.shape=}")

        masked_m = masked_m.to(device)

        return x, masked_m
    else:
        x = torch.randn(
            num_tokens,
            effective_hidden_dim,
            device=device,
            dtype=dtype,
            generator=gen_cuda,
        )
        x[torch.randn(x.shape, device=device, generator=gen_cuda) < 0.001] *= 10
        return x, None
````
**EN:** This section defines `create_per_token_group_quant_test_data` and implements the core logic associated with create per token group quantization test data. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `create_per_token_group_quant_test_data`，并实现与 create per token group quantization test data 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 65-70: `_compute_balanced_split` definition
````python
def _compute_balanced_split(total: int, arr_len: int):
    base = total // arr_len
    remainder = total % arr_len
    ans = [base + 1 if i < remainder else base for i in range(arr_len)]
    assert sum(ans) == total
    return torch.tensor(ans, dtype=torch.int)
````
**EN:** This section defines `_compute_balanced_split` and implements the core logic associated with compute balanced split. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `_compute_balanced_split`，并实现与 compute balanced split 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 73-93: `_compute_imbalanced_split` definition
````python
def _compute_imbalanced_split(
    total: int, arr_len: int, gen_cpu, dtype=torch.int
) -> list[int]:
    # can use `rand ** 2`, `rand ** 3`, etc, to change how imbalanced it is
    noise_raw = torch.rand(arr_len, generator=gen_cpu) ** 3

    noise = noise_raw / noise_raw.sum()
    ans = (noise * total).round().to(dtype)

    diff = total - ans.sum().item()
    while diff != 0:
        idx = torch.randint(0, arr_len, (1,), generator=gen_cpu).item()
        if diff > 0:
            ans[idx] += 1
            diff -= 1
        elif diff < 0 and ans[idx] > 0:
            ans[idx] -= 1
            diff += 1

    assert sum(ans) == total
    return ans
````
**EN:** This section defines `_compute_imbalanced_split` and implements the core logic associated with compute imbalanced split. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `_compute_imbalanced_split`，并实现与 compute imbalanced split 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 96-125: `assert_all_close_or_tiny_diff` definition
````python
def assert_all_close_or_tiny_diff(a: torch.Tensor, b: torch.Tensor):
    assert (a.shape == b.shape) and (
        a.dtype == b.dtype
    ), f"{a.shape=} {b.shape=} {a.dtype=} {b.dtype=}"
    numel = a.numel()

    if a.dtype == torch.float8_e4m3fn:
        a_u8 = a.view(torch.uint8)
        b_u8 = b.view(torch.uint8)
        diff_u8 = (a_u8.to(torch.int16) - b_u8.to(torch.int16)).abs()

        count_diff_sign = ((a_u8 >= 0) & (b_u8 < 0)).sum().item()
        count_tiny_diff = (diff_u8 == 1).sum().item()
        count_large_diff = (diff_u8 >= 2).sum().item()
    elif a.dtype == torch.int8:
        diff = (a.to(torch.int16) - a.to(torch.int16)).abs()
        count_diff_sign = ((a >= 0) & (b < 0)).sum().item()
        count_tiny_diff = (diff == 1).sum().item()
        count_large_diff = (diff >= 2).sum().item()
    else:
        raise NotImplementedError

    assert (
        (count_diff_sign == 0)
        and (count_large_diff == 0)
        and (
            (count_tiny_diff / numel < 0.005)
            or ((count_tiny_diff / numel < 0.04) and (numel <= 4096))
        )
    ), f"{count_diff_sign=} {count_tiny_diff=} {count_large_diff=} {numel=} {a=} {b=}"
````
**EN:** This section defines `assert_all_close_or_tiny_diff` and implements the core logic associated with assert all close or tiny diff. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `assert_all_close_or_tiny_diff`，并实现与 assert all close or tiny diff 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `create_per_token_group_quant_test_data`, `_compute_balanced_split`, `_compute_imbalanced_split`, `assert_all_close_or_tiny_diff`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `torch`
