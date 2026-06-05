# triton_flashinfer_cudnn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/decoding_attention_triton/triton_flashinfer_cudnn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels decoding attention triton flashinfer. It primarily measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. / 该 Python 模块聚焦于 kernels decoding attention triton flashinfer 相关流程。它主要用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and setup / 导入与初始化
```python
import itertools
import math

import cudnn
import torch
import torch.utils.benchmark as benchmark
from flashinfer import BatchDecodeWithPagedKVCacheWrapper

from sglang.srt.layers.attention.flashinfer_backend import should_use_tensor_core
from sglang.srt.layers.attention.triton_ops.decode_attention import decode_attention_fwd
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。

### Lines 13-31: Function `benchmark_forward` / 函数 `benchmark_forward`
```python
def benchmark_forward(
    fn,
    *inputs,
    repeats=10,
    amp=False,
    amp_dtype=torch.float16,
    **kwinputs,
):
    def amp_wrapper(*inputs, **kwinputs):
        with torch.autocast(device_type="cuda", dtype=amp_dtype, enabled=amp):
            fn(*inputs, **kwinputs)

    t = benchmark.Timer(
        stmt="fn_amp(*inputs, **kwinputs)",
        globals={"fn_amp": amp_wrapper, "inputs": inputs, "kwinputs": kwinputs},
        num_threads=torch.get_num_threads(),
    )
    m = t.timeit(repeats)
    return t, m
```
**EN:** `benchmark_forward` is a function that measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels. It returns `(t, m)` to the caller. Notable calls include `benchmark.Timer`, `t.timeit`, `torch.autocast`.
**CN:** `benchmark_forward` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。它会向调用方返回 `(t, m)`。其中较关键的调用包括 `benchmark.Timer`, `t.timeit`, `torch.autocast`。

### Lines 34-36: Function `time_fwd` / 函数 `time_fwd`
```python
def time_fwd(func, *args, **kwargs):
    time_f = benchmark_forward(func, *args, **kwargs)
    return time_f[1].mean * 1e6
```
**EN:** `time_fwd` is a function that measures runtime latency, throughput, or other benchmark metrics. It returns `time_f[1].mean * 1000000.0` to the caller. Notable calls include `benchmark_forward`.
**CN:** `time_fwd` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `time_f[1].mean * 1000000.0`。其中较关键的调用包括 `benchmark_forward`。

### Lines 39-95: Function `decode_attention_sglang` / 函数 `decode_attention_sglang`
```python
def decode_attention_sglang(
    q,
    kv_data,
    batch_size,
    kv_len,
    head_num_q,
    head_num_kv,
    head_dim,
    num_kv_splits,
    warmup=10,
):

    k_buffer = kv_data[0].view(-1, head_num_kv, head_dim)
    v_buffer = kv_data[1].view(-1, head_num_kv, head_dim)
    o = torch.empty_like(q)
    total_tokens = batch_size * kv_len
    req_to_token = torch.arange(0, total_tokens).to(0).int().view(batch_size, kv_len)
    b_req_idx = torch.arange(0, batch_size).to(0).int()
    b_seq_len = torch.full((batch_size,), kv_len, dtype=torch.int32, device="cuda")
    max_len_in_batch = kv_len
    sm_scale = 1.0 / (head_dim**0.5)

    attn_logits = torch.empty(
        (batch_size, head_num_q, num_kv_splits, head_dim + 1),
        dtype=torch.float32,
        device="cuda",
    )

    for _ in range(warmup):
        decode_attention_fwd(
            q,
            k_buffer,
            v_buffer,
            o,
            req_to_token,
            b_req_idx,
            b_seq_len,
            attn_logits,
            num_kv_splits,
            sm_scale,
        )

    f = time_fwd(
        decode_attention_fwd,
        q,
        k_buffer,
        v_buffer,
        o,
        req_to_token,
        b_req_idx,
        b_seq_len,
        attn_logits,
        num_kv_splits,
        sm_scale,
    )

    return f, o
```
**EN:** `decode_attention_sglang` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `(f, o)` to the caller. Notable calls include `kv_data[0].view`, `kv_data[1].view`, `torch.empty_like`.
**CN:** `decode_attention_sglang` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `(f, o)`。其中较关键的调用包括 `kv_data[0].view`, `kv_data[1].view`, `torch.empty_like`。

### Lines 98-156: Function `decode_attention_flashinfer` / 函数 `decode_attention_flashinfer`
```python
def decode_attention_flashinfer(dtype, head_num_q, head_num_kv):
    workspace_buffer = torch.empty(128 * 1024 * 1024, dtype=torch.int8, device="cuda")
    use_tensor_cores = should_use_tensor_core(
        kv_cache_dtype=dtype,
        num_attention_heads=head_num_q,
        num_kv_heads=head_num_kv,
    )
    flashinfer_decode_wrapper = BatchDecodeWithPagedKVCacheWrapper(
        workspace_buffer, "NHD", use_tensor_cores=use_tensor_cores
    )

    class FlashinferAttention(torch.autograd.Function):
        @staticmethod
        def forward(
            ctx,
            q,
            kv_data,
            batch_size,
            kv_len,
            head_num_q,
            head_num_kv,
            head_dim,
            dtype,
            warmup=10,
        ):
            total_tokens = batch_size * kv_len
            kv_indptr = torch.arange(0, batch_size + 1).to(0).int() * kv_len
            kv_indices = torch.arange(0, total_tokens).to(0).int()
            kv_last_page_len = torch.full(
                (batch_size,), 1, dtype=torch.int32, device="cuda"
            )

            flashinfer_decode_wrapper.end_forward()
            flashinfer_decode_wrapper.begin_forward(
                kv_indptr,
                kv_indices,
                kv_last_page_len,
                head_num_q,
                head_num_kv,
                head_dim,
                1,
                pos_encoding_mode="NONE",
                data_type=dtype,
            )

            for _ in range(warmup):
                o = flashinfer_decode_wrapper.forward(
                    q.contiguous().view(-1, head_num_q, head_dim), kv_data
                )

            f = time_fwd(
                flashinfer_decode_wrapper.forward,
                q.contiguous().view(-1, head_num_q, head_dim),
                kv_data,
            )

            return f, o

    return FlashinferAttention
```
**EN:** `decode_attention_flashinfer` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `FlashinferAttention` to the caller. Notable calls include `torch.empty`, `should_use_tensor_core`, `BatchDecodeWithPagedKVCacheWrapper`.
**CN:** `decode_attention_flashinfer` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `FlashinferAttention`。其中较关键的调用包括 `torch.empty`, `should_use_tensor_core`, `BatchDecodeWithPagedKVCacheWrapper`。

### Lines 159-171: Function `convert_to_cudnn_type` / 函数 `convert_to_cudnn_type`
```python
def convert_to_cudnn_type(torch_type):
    if torch_type == torch.float16:
        return cudnn.data_type.HALF
    elif torch_type == torch.bfloat16:
        return cudnn.data_type.BFLOAT16
    elif torch_type == torch.float32:
        return cudnn.data_type.FLOAT
    elif torch_type == torch.int32:
        return cudnn.data_type.INT32
    elif torch_type == torch.int64:
        return cudnn.data_type.INT64
    else:
        raise ValueError("Unsupported tensor data type.")
```
**EN:** `convert_to_cudnn_type` is a function that prepares tensors and invokes GPU kernels. It returns `cudnn.data_type.HALF` to the caller. Notable calls include `ValueError`.
**CN:** `convert_to_cudnn_type` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `cudnn.data_type.HALF`。其中较关键的调用包括 `ValueError`。

### Lines 174-284: Function `decode_attention_cudnn` / 函数 `decode_attention_cudnn`
```python
def decode_attention_cudnn(
    q, kv_data, batch_size, kv_len, head_num_q, head_num_kv, head_dim, dtype, warmup=10
):
    # Prepare data: continuous q,k,v
    dims_q = (batch_size, head_num_q, 1, head_dim)
    strides_q = (head_num_q * head_dim, head_dim, head_num_q * head_dim, 1)
    q_gpu = q.as_strided(dims_q, strides_q)
    o_gpu = (
        torch.empty(batch_size * head_num_q * head_dim)
        .half()
        .cuda()
        .as_strided(dims_q, strides_q)
    )

    dims_kv = (batch_size, head_num_kv, kv_len, head_dim)
    strides_kv = (
        kv_len * head_num_kv * head_dim,
        head_dim,
        head_num_kv * head_dim,
        1,
    )
    k_gpu = kv_data[0].as_strided(dims_kv, strides_kv)
    v_gpu = kv_data[1].as_strided(dims_kv, strides_kv)

    seq_len_q_gpu = torch.full((batch_size, 1, 1, 1), 1, device="cuda")
    seq_len_kv_gpu = torch.full((batch_size, 1, 1, 1), kv_len, device="cuda")
    attn_scale = 1.0 / (head_dim**0.5)

    # Prepare data: paged k,v
    block_size = 1
    blocks_per_batch = math.ceil(kv_len / block_size)
    # [num_blocks, head_num_kv, block_size, head_dim], num_blocks = batch_size * blocks_per_batch
    container_k_gpu = torch.cat(k_gpu.chunk(blocks_per_batch, dim=2), dim=0)
    container_v_gpu = torch.cat(v_gpu.chunk(blocks_per_batch, dim=2), dim=0)
    page_table_k_gpu = (
        torch.linspace(
            0,
            batch_size * blocks_per_batch - 1,
            batch_size * blocks_per_batch,
            device="cuda",
            dtype=torch.int32,
        )
        .reshape(blocks_per_batch, 1, batch_size, 1)
        .transpose(0, 2)
    )
    page_table_v_gpu = page_table_k_gpu.clone()

    graph = cudnn.pygraph(
        io_data_type=convert_to_cudnn_type(dtype),
        intermediate_data_type=cudnn.data_type.FLOAT,
        compute_data_type=cudnn.data_type.FLOAT,
    )

    q = graph.tensor_like(q_gpu)
    container_k = graph.tensor_like(container_k_gpu)
    container_v = graph.tensor_like(container_v_gpu)
    page_table_k = graph.tensor_like(page_table_k_gpu)
    page_table_v = graph.tensor_like(page_table_v_gpu)

    seq_len_q = graph.tensor_like(seq_len_q_gpu)
    seq_len_kv = graph.tensor_like(seq_len_kv_gpu)

    o, _ = graph.sdpa(
        name="sdpa",
        q=q,
        k=container_k,  # Container K: non contiguous container with K blocks
        v=container_v,  # Container V: non contiguous container with V blocks
        is_inference=True,
        attn_scale=attn_scale,
        use_causal_mask=False,
        use_padding_mask=True,
        seq_len_q=seq_len_q,
        seq_len_kv=seq_len_kv,
        paged_attention_k_table=page_table_k,  # Page Table K: Tensor containing offsets to the container with K blocks
        paged_attention_v_table=page_table_v,  # Page Table V: Tensor containing offsets to the container with V blocks
        paged_attention_max_seq_len_kv=kv_len,  # The maximum sequence length for K caches (this is optional, but recommended)
    )

    o.set_output(True).set_dim(dims_q).set_stride(strides_q)

    graph.validate()
    graph.build_operation_graph()
    graph.create_execution_plans([cudnn.heur_mode.A])
    graph.check_support()
    graph.build_plans()

    workspace = torch.empty(
        graph.get_workspace_size(), device="cuda", dtype=torch.uint8
    )

    variant_pack = {
        q: q_gpu,
        container_k: container_k_gpu,
        container_v: container_v_gpu,
        page_table_k: page_table_k_gpu,
        page_table_v: page_table_v_gpu,
        seq_len_q: seq_len_q_gpu,
        seq_len_kv: seq_len_kv_gpu,
        o: o_gpu,
    }

    for _ in range(warmup):
        graph.execute(variant_pack, workspace)

    f = time_fwd(
        graph.execute,
        variant_pack,
        workspace,
    )

    return f, o_gpu.squeeze(dim=2)
```
**EN:** `decode_attention_cudnn` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `(f, o_gpu.squeeze(dim=2))` to the caller. Notable calls include `q.as_strided`, `torch.empty(batch_size * head_num_q * head_dim).half().cuda().as_strided`, `kv_data[0].as_strided`.
**CN:** `decode_attention_cudnn` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `(f, o_gpu.squeeze(dim=2))`。其中较关键的调用包括 `q.as_strided`, `torch.empty(batch_size * head_num_q * head_dim).half().cuda().as_strided`, `kv_data[0].as_strided`。

### Lines 287-337: Function `calculate_diff` / 函数 `calculate_diff`
```python
def calculate_diff():

    dtype = torch.float16
    batch_size = 64
    kv_len = 4096
    head_num_q = 64
    head_num_kv = 8
    head_dim = 128

    q = torch.randn(batch_size, head_num_q, head_dim, dtype=dtype, device="cuda")
    kv_data = (
        torch.randn(
            batch_size * kv_len, head_num_kv, head_dim, dtype=dtype, device="cuda"
        ),
        torch.randn(
            batch_size * kv_len, head_num_kv, head_dim, dtype=dtype, device="cuda"
        ),
    )

    _, output_sglang = decode_attention_sglang(
        q,
        kv_data,
        batch_size,
        kv_len,
        head_num_q,
        head_num_kv,
        head_dim,
        num_kv_splits=8,
    )

    attn_flashinfer = decode_attention_flashinfer(dtype, head_num_q, head_num_kv).apply
    _, output_flashinfer = attn_flashinfer(
        q, kv_data, batch_size, kv_len, head_num_q, head_num_kv, head_dim, dtype
    )

    _, output_cudnn = decode_attention_cudnn(
        q, kv_data, batch_size, kv_len, head_num_q, head_num_kv, head_dim, dtype
    )

    print(f"SGLang output={output_sglang}")
    print(f"FlashInfer output={output_flashinfer}")
    print(f"cuDNN output={output_cudnn}")
    if torch.allclose(output_sglang, output_flashinfer, atol=1e-2, rtol=1e-2):
        print("✅ SGLang[Triton] and FlashInfer match")
    else:
        print("❌ SGLang[Triton] and FlashInfer differ")

    if torch.allclose(output_sglang, output_cudnn, atol=1e-2, rtol=1e-2):
        print("✅ SGLang[Triton] and cuDNN match")
    else:
        print("❌ SGLang[Triton] and cuDNN differ")
```
**EN:** `calculate_diff` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. Notable calls include `torch.randn`, `decode_attention_sglang`, `attn_flashinfer`.
**CN:** `calculate_diff` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。其中较关键的调用包括 `torch.randn`, `decode_attention_sglang`, `attn_flashinfer`。

### Lines 340-403: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    calculate_diff()

    head_dim = 128
    dtype = torch.float16
    batch_size_range = [2**i for i in range(0, 8, 2)]
    kv_len_range = [2**i for i in range(6, 13, 1)]
    configs = list(itertools.product(batch_size_range, kv_len_range))

    for head_num_q, head_num_kv in [[32, 32], [64, 8], [40, 8]]:
        attn_flashinfer = decode_attention_flashinfer(
            dtype, head_num_q, head_num_kv
        ).apply
        for batch_size, kv_len in configs:
            q = torch.randn(
                batch_size, head_num_q, head_dim, dtype=dtype, device="cuda"
            )
            kv_data = (
                torch.randn(
                    batch_size * kv_len,
                    head_num_kv,
                    head_dim,
                    dtype=dtype,
                    device="cuda",
                ),
                torch.randn(
                    batch_size * kv_len,
                    head_num_kv,
                    head_dim,
                    dtype=dtype,
                    device="cuda",
                ),
            )
            us_cudnn, output_cudnn = decode_attention_cudnn(
                q, kv_data, batch_size, kv_len, head_num_q, head_num_kv, head_dim, dtype
            )
            us_sglang, output_sglang = decode_attention_sglang(
                q,
                kv_data,
                batch_size,
                kv_len,
                head_num_q,
                head_num_kv,
                head_dim,
                num_kv_splits=8,
            )
            us_flashinfer, _ = attn_flashinfer(
                q, kv_data, batch_size, kv_len, head_num_q, head_num_kv, head_dim, dtype
            )
            print(
                head_num_q,
                "  ",
                head_num_kv,
                "  ",
                batch_size,
                "  ",
                kv_len,
                "  ",
                us_cudnn,
                "  ",
                us_sglang,
                "  ",
                us_flashinfer,
            )
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `math`
- **Third-party / 第三方依赖**: `cudnn`, `torch`, `torch.utils.benchmark`, `flashinfer`
- **Internal / 项目内部依赖**: `sglang.srt.layers.attention.flashinfer_backend`, `sglang.srt.layers.attention.triton_ops.decode_attention`
