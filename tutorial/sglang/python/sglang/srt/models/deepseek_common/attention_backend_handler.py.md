# attention_backend_handler.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/deepseek_common/attention_backend_handler.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module selects the concrete attention execution path for DeepSeek-family models based on backend capabilities, runtime mode, and hardware. / 该模块会根据后端能力、运行模式和硬件条件，为 DeepSeek 系列模型选择具体的注意力执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module header and imports / 模块头与导入
```python
from sglang.srt.compilation.piecewise_context_manager import is_in_piecewise_cuda_graph
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 10-10: Assignment: MHA_ONE_SHOT_SUPPORTED_BACKENDS / 赋值：MHA_ONE_SHOT_SUPPORTED_BACKENDS
```python
MHA_ONE_SHOT_SUPPORTED_BACKENDS = ["fa3", "flashinfer", "flashmla"]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 13-22: Class: AttentionBackendRegistry / 类：AttentionBackendRegistry
```python
class AttentionBackendRegistry:
    _handlers = {}

    @classmethod
    def register(cls, backend_name, handler_func):
        cls._handlers[backend_name] = handler_func

    @classmethod
    def get_handler(cls, backend_name):
        return cls._handlers.get(backend_name, cls._handlers.get("triton"))
```
**EN:** This class defines Attention Backend Registry, grouping state and methods for this model component.
**CN:** 该类定义了 Attention Backend Registry，用于封装该模型组件的状态与方法。

### Lines 25-35: Function: _dispatch_mla_subtype() / 函数：_dispatch_mla_subtype()
```python
def _dispatch_mla_subtype(attn, forward_batch):
    if _is_hip:
        if attn.rocm_fused_decode_mla and forward_batch.forward_mode.is_decode():
            return AttnForwardMethod.MLA_FUSED_ROPE_ROCM
        else:
            return AttnForwardMethod.MLA
    else:
        if hasattr(attn, "fused_qkv_a_proj_with_mqa") and use_intel_amx_backend(attn):
            return AttnForwardMethod.MLA_FUSED_ROPE_CPU
        else:
            return AttnForwardMethod.MLA
```
**EN:** This function prepares inputs or selects an execution path before the main compute step. Key parameters include attn, forward_batch.
**CN:** 该函数在主计算之前准备输入，或根据条件选择合适的执行路径。

### Lines 38-53: Function: handle_attention_ascend() / 函数：handle_attention_ascend()
```python
def handle_attention_ascend(attn, forward_batch):
    if (
        forward_batch.forward_mode.is_extend()
        and not forward_batch.forward_mode.is_target_verify()
        and not forward_batch.forward_mode.is_draft_extend()
        and not forward_batch.forward_mode.is_draft_extend_v2()
    ):
        if hasattr(attn, "indexer"):
            return AttnForwardMethod.DSA_NPU
        else:
            return AttnForwardMethod.MHA_NPU
    else:
        if hasattr(attn, "indexer"):
            return AttnForwardMethod.DSA_NPU
        else:
            return AttnForwardMethod.MLA_NPU
```
**EN:** This function implements handle attention ascend for the surrounding model/runtime logic. Key parameters include attn, forward_batch.
**CN:** 该函数实现了 handle attention ascend 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 56-61: Function: _get_sum_extend_prefix_lens() / 函数：_get_sum_extend_prefix_lens()
```python
def _get_sum_extend_prefix_lens(forward_batch):
    return (
        sum(forward_batch.extend_prefix_lens_cpu)
        if forward_batch.extend_prefix_lens_cpu is not None
        else 0
    )
```
**EN:** This function implements get sum extend prefix lens for the surrounding model/runtime logic. Key parameters include forward_batch.
**CN:** 该函数实现了 get sum extend prefix lens 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 64-69: Function: _support_mha_one_shot() / 函数：_support_mha_one_shot()
```python
def _support_mha_one_shot(attn, forward_batch, backend_name):
    attn_supported = backend_name in MHA_ONE_SHOT_SUPPORTED_BACKENDS
    sum_seq_lens = (
        sum(forward_batch.seq_lens_cpu) if forward_batch.seq_lens_cpu is not None else 0
    )
    return attn_supported and sum_seq_lens <= forward_batch.get_max_chunk_capacity()
```
**EN:** This function implements support mha one shot for the surrounding model/runtime logic. Key parameters include attn, forward_batch, backend_name.
**CN:** 该函数实现了 support mha one shot 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 72-96: Function: _handle_attention_backend() / 函数：_handle_attention_backend()
```python
def _handle_attention_backend(attn, forward_batch, backend_name):
    if is_in_piecewise_cuda_graph():
        return AttnForwardMethod.MLA

    sum_extend_prefix_lens = _get_sum_extend_prefix_lens(forward_batch)
    disable_ragged = (
        backend_name in ["flashinfer", "flashmla"]
    ) and attn.flashinfer_mla_disable_ragged

    if (
        not disable_ragged
        and forward_batch.forward_mode.is_extend_without_speculative()
        and (
            (
                sum_extend_prefix_lens >= attn.chunked_prefix_cache_threshold
                and not attn.disable_chunked_prefix_cache
            )
            or sum_extend_prefix_lens == 0
        )
    ):
        if _support_mha_one_shot(attn, forward_batch, backend_name):
            return AttnForwardMethod.MHA_ONE_SHOT
        return AttnForwardMethod.MHA_CHUNKED_KV
    else:
        return _dispatch_mla_subtype(attn, forward_batch)
```
**EN:** This function implements handle attention backend for the surrounding model/runtime logic. Key parameters include attn, forward_batch, backend_name.
**CN:** 该函数实现了 handle attention backend 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 99-100: Function: handle_attention_flashinfer() / 函数：handle_attention_flashinfer()
```python
def handle_attention_flashinfer(attn, forward_batch):
    return _handle_attention_backend(attn, forward_batch, "flashinfer")
```
**EN:** This function implements handle attention flashinfer for the surrounding model/runtime logic. Key parameters include attn, forward_batch.
**CN:** 该函数实现了 handle attention flashinfer 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 103-108: Function: handle_attention_fa3() / 函数：handle_attention_fa3()
```python
def handle_attention_fa3(attn, forward_batch):
    # when deterministic inference is enabled, use MLA
    if get_global_server_args().enable_deterministic_inference:
        return _dispatch_mla_subtype(attn, forward_batch)
    else:
        return _handle_attention_backend(attn, forward_batch, "fa3")
```
**EN:** This function implements handle attention fa3 for the surrounding model/runtime logic. Key parameters include attn, forward_batch.
**CN:** 该函数实现了 handle attention fa3 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 111-112: Function: handle_attention_flashmla() / 函数：handle_attention_flashmla()
```python
def handle_attention_flashmla(attn, forward_batch):
    return _handle_attention_backend(attn, forward_batch, "flashmla")
```
**EN:** This function implements handle attention flashmla for the surrounding model/runtime logic. Key parameters include attn, forward_batch.
**CN:** 该函数实现了 handle attention flashmla 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 115-116: Function: handle_attention_cutlass_mla() / 函数：handle_attention_cutlass_mla()
```python
def handle_attention_cutlass_mla(attn, forward_batch):
    return _handle_attention_backend(attn, forward_batch, "cutlass_mla")
```
**EN:** This function implements handle attention cutlass mla for the surrounding model/runtime logic. Key parameters include attn, forward_batch.
**CN:** 该函数实现了 handle attention cutlass mla 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 119-121: Function: handle_attention_fa4() / 函数：handle_attention_fa4()
```python
def handle_attention_fa4(attn, forward_batch):
    # TODO(cicirori): use FA4 MHA for DeepSeekV3 for now
    return AttnForwardMethod.MHA_CHUNKED_KV
```
**EN:** This function implements handle attention fa4 for the surrounding model/runtime logic. Key parameters include attn, forward_batch.
**CN:** 该函数实现了 handle attention fa4 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 124-134: Function: handle_attention_trtllm_mla() / 函数：handle_attention_trtllm_mla()
```python
def handle_attention_trtllm_mla(attn, forward_batch):
    if is_in_piecewise_cuda_graph():
        return AttnForwardMethod.MLA

    sum_extend_prefix_lens = _get_sum_extend_prefix_lens(forward_batch)
    if forward_batch.forward_mode.is_extend_without_speculative() and (
        not attn.disable_chunked_prefix_cache or sum_extend_prefix_lens == 0
    ):
        return AttnForwardMethod.MHA_CHUNKED_KV
    else:
        return _dispatch_mla_subtype(attn, forward_batch)
```
**EN:** This function implements handle attention trtllm mla for the surrounding model/runtime logic. Key parameters include attn, forward_batch.
**CN:** 该函数实现了 handle attention trtllm mla 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 137-140: Function: handle_attention_tokenspeed_mla() / 函数：handle_attention_tokenspeed_mla()
```python
def handle_attention_tokenspeed_mla(attn, forward_batch):
    # tokenspeed_mla shares the trtllm_mla dispatch pattern: pure prefill goes
    # via MHA chunked KV (TRT-LLM ragged), spec decode / decode goes via MLA.
    return handle_attention_trtllm_mla(attn, forward_batch)
```
**EN:** This function implements handle attention tokenspeed mla for the surrounding model/runtime logic. Key parameters include attn, forward_batch.
**CN:** 该函数实现了 handle attention tokenspeed mla 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 143-147: Function: handle_attention_aiter() / 函数：handle_attention_aiter()
```python
def handle_attention_aiter(attn, forward_batch):
    if forward_batch.forward_mode.is_extend_without_speculative():
        return AttnForwardMethod.MHA
    else:
        return AttnForwardMethod.MLA
```
**EN:** This function implements handle attention aiter for the surrounding model/runtime logic. Key parameters include attn, forward_batch.
**CN:** 该函数实现了 handle attention aiter 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 150-161: Function: handle_attention_nsa() / 函数：handle_attention_nsa()
```python
def handle_attention_nsa(attn, forward_batch):
    """
    Dispatch logic is centralized in NativeSparseAttnBackend.set_nsa_prefill_impl and executed
    in init_forward_metadata. Read the decision from backend.use_mha.
    """

    backend = forward_batch.attn_backend
    if isinstance(backend, TboAttnBackend):  # if enable tbo, get primary backend
        backend = backend.primary
    if hasattr(backend, "use_mha") and backend.use_mha:
        return AttnForwardMethod.MHA_ONE_SHOT
    return AttnForwardMethod.MLA
```
**EN:** This function implements handle attention nsa for the surrounding model/runtime logic. Key parameters include attn, forward_batch.
**CN:** 该函数实现了 handle attention nsa 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 164-178: Function: handle_attention_triton() / 函数：handle_attention_triton()
```python
def handle_attention_triton(attn, forward_batch):
    if is_in_piecewise_cuda_graph():
        return AttnForwardMethod.MLA

    # when deterministic inference is enabled, use MLA
    if get_global_server_args().enable_deterministic_inference:
        return _dispatch_mla_subtype(attn, forward_batch)

    if (
        forward_batch.forward_mode.is_extend_without_speculative()
        and sum(forward_batch.extend_prefix_lens_cpu) == 0
    ):
        return AttnForwardMethod.MHA
    else:
        return _dispatch_mla_subtype(attn, forward_batch)
```
**EN:** This function implements handle attention triton for the surrounding model/runtime logic. Key parameters include attn, forward_batch.
**CN:** 该函数实现了 handle attention triton 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 181-182: Function: handle_attention_intel_xpu() / 函数：handle_attention_intel_xpu()
```python
def handle_attention_intel_xpu(attn, forward_batch):
    return _handle_attention_backend(attn, forward_batch, "intel_xpu")
```
**EN:** This function implements handle attention intel xpu for the surrounding model/runtime logic. Key parameters include attn, forward_batch.
**CN:** 该函数实现了 handle attention intel xpu 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 185-185: Code block / 代码块
```python
AttentionBackendRegistry.register("ascend", handle_attention_ascend)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

### Lines 186-186: Code block / 代码块
```python
AttentionBackendRegistry.register("flashinfer", handle_attention_flashinfer)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

### Lines 187-187: Code block / 代码块
```python
AttentionBackendRegistry.register("fa3", handle_attention_fa3)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

### Lines 188-188: Code block / 代码块
```python
AttentionBackendRegistry.register("flashmla", handle_attention_flashmla)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

### Lines 189-189: Code block / 代码块
```python
AttentionBackendRegistry.register("cutlass_mla", handle_attention_cutlass_mla)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

### Lines 190-190: Code block / 代码块
```python
AttentionBackendRegistry.register("fa4", handle_attention_fa4)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

### Lines 191-191: Code block / 代码块
```python
AttentionBackendRegistry.register("trtllm_mla", handle_attention_trtllm_mla)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

### Lines 192-192: Code block / 代码块
```python
AttentionBackendRegistry.register("tokenspeed_mla", handle_attention_tokenspeed_mla)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

### Lines 193-193: Code block / 代码块
```python
AttentionBackendRegistry.register("aiter", handle_attention_aiter)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

### Lines 194-194: Code block / 代码块
```python
AttentionBackendRegistry.register("nsa", handle_attention_nsa)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

### Lines 195-195: Code block / 代码块
```python
AttentionBackendRegistry.register("triton", handle_attention_triton)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

### Lines 196-196: Code block / 代码块
```python
AttentionBackendRegistry.register("intel_xpu", handle_attention_intel_xpu)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

## Key Concepts / 关键概念
- **EN:** Backend-aware dispatch that selects specialized attention implementations for different hardware and execution modes. / **CN:** 具备后端感知能力的分派机制，可针对不同硬件与执行模式选择特化注意力实现。
- **EN:** Shared utilities that reduce duplication across DeepSeek-family model integrations. / **CN:** 在 DeepSeek 系列模型集成中复用的共享工具，以减少重复实现。

## Dependencies / 依赖关系
- `sglang.srt.compilation.piecewise_context_manager: is_in_piecewise_cuda_graph`
- `sglang.srt.layers.attention.tbo_backend: TboAttnBackend`
- `sglang.srt.models.deepseek_common.attention_forward_methods.forward_methods: AttnForwardMethod`
- `sglang.srt.models.deepseek_common.utils: _is_hip`
- `sglang.srt.server_args: get_global_server_args`
- `sglang.srt.utils: use_intel_amx_backend`
