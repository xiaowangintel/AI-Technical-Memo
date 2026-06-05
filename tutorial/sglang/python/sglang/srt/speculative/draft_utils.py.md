# draft_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/draft_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements speculative decoding infrastructure, such as workers, metadata objects, CUDA-graph runners, or shared utilities. It coordinates draft-model execution with target-model verification to improve throughput. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8: Module header, imports, and shared constants
```python
import logging

from sglang.srt.server_args import ServerArgs, get_global_server_args
from sglang.srt.utils.common import is_blackwell, is_musa

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 9-9: DraftBackendFactory class declaration
```python
class DraftBackendFactory:
```
**EN:** This block declares the `DraftBackendFactory` class, which exists to produce draft tokens or draft-side state. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `DraftBackendFactory` 类，其职责是生成草稿 token 或草稿侧状态。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 10-21: DraftBackendFactory initializer
```python
    def __init__(
        self,
        server_args: ServerArgs,
        draft_model_runner,
        topk: int,
        speculative_num_steps: int,
    ):
        self.server_args = server_args
        self.draft_model_runner = draft_model_runner
        self.topk = topk
        self.speculative_num_steps = speculative_num_steps
        self.draft_attn_backend = server_args.speculative_draft_attention_backend
```
**EN:** This block initializes the `DraftBackendFactory` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `DraftBackendFactory` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 23-37: DraftBackendFactory._create_backend method
```python
    def _create_backend(
        self, backend_name: str, backend_map: dict, error_template: str
    ):
        backend_type = (
            self.draft_attn_backend
            if self.draft_attn_backend
            else getattr(self.server_args, backend_name)
        )
        if backend_type is None:
            backend_type = self.server_args.attention_backend

        if backend_type not in backend_map:
            raise ValueError(error_template.format(backend_type=backend_type))

        return backend_map[backend_type]()
```
**EN:** This block uses `DraftBackendFactory._create_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 39-67: DraftBackendFactory.create_decode_backend method
```python
    def create_decode_backend(self):
        if self.speculative_num_steps == 1:
            return None

        backend_map = {
            "flashinfer": self._create_flashinfer_decode_backend,
            "triton": self._create_triton_decode_backend,
            "aiter": self._create_aiter_decode_backend,
            "fa3": self._create_fa3_decode_backend,
            "hybrid_linear_attn": (
                self._create_fa3_decode_backend
                if not is_blackwell()
                else self._create_triton_decode_backend
            ),
            "flashmla": self._create_flashmla_decode_backend,
            "trtllm_mha": self._create_trtllm_mha_decode_backend,
            "trtllm_mla": self._create_trtllm_mla_decode_backend,
            "tokenspeed_mla": self._create_tokenspeed_mla_decode_backend,
            "nsa": self._create_nsa_decode_backend,
            "ascend": self._create_ascend_decode_backend,
            "fa4": self._create_fa4_decode_backend,
            "dsv4": self._create_dsv4_decode_backend,
        }

        return self._create_backend(
            "decode_attention_backend",
            backend_map,
            "EAGLE is not supported in decode attention backend {backend_type}",
        )
```
**EN:** This block uses `DraftBackendFactory.create_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory.create_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 69-98: DraftBackendFactory.create_draft_extend_backend method
```python
    def create_draft_extend_backend(self):
        backend_map = {
            "flashinfer": self._create_flashinfer_prefill_backend,
            "triton": self._create_triton_prefill_backend,
            "aiter": self._create_aiter_prefill_backend,
            "fa3": self._create_fa3_prefill_backend,
            "hybrid_linear_attn": (
                self._create_fa3_prefill_backend
                if not is_blackwell()
                else self._create_triton_prefill_backend
            ),
            "flashmla": self._create_flashmla_prefill_backend,
            "trtllm_mha": self._create_trtllm_mha_prefill_backend,
            "trtllm_mla": self._create_trtllm_mla_prefill_backend,
            "tokenspeed_mla": self._create_tokenspeed_mla_prefill_backend,
            "nsa": self._create_nsa_prefill_backend,
            "ascend": self._create_ascend_prefill_backend,
            "fa4": self._create_fa4_prefill_backend,
            "dsv4": self._create_dsv4_prefill_backend,
        }
        backend_name = (
            "decode_attention_backend"
            if self.server_args.speculative_attention_mode == "decode"
            else "prefill_attention_backend"
        )
        return self._create_backend(
            backend_name,
            backend_map,
            "EAGLE is not supported in attention backend {backend_type}",
        )
```
**EN:** This block uses `DraftBackendFactory.create_draft_extend_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory.create_draft_extend_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 100-107: DraftBackendFactory._create_nsa_decode_backend method
```python
    def _create_nsa_decode_backend(self):
        from sglang.srt.layers.attention.nsa_backend import (
            NativeSparseAttnMultiStepBackend,
        )

        return NativeSparseAttnMultiStepBackend(
            self.draft_model_runner, self.topk, self.speculative_num_steps
        )
```
**EN:** This block uses `DraftBackendFactory._create_nsa_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_nsa_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 109-112: DraftBackendFactory._create_nsa_prefill_backend method
```python
    def _create_nsa_prefill_backend(self):
        from sglang.srt.layers.attention.nsa_backend import NativeSparseAttnBackend

        return NativeSparseAttnBackend(self.draft_model_runner, skip_prefill=False)
```
**EN:** This block uses `DraftBackendFactory._create_nsa_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_nsa_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 114-130: DraftBackendFactory._create_flashinfer_decode_backend method
```python
    def _create_flashinfer_decode_backend(self):
        if not get_global_server_args().use_mla_backend:
            from sglang.srt.layers.attention.flashinfer_backend import (
                FlashInferMultiStepDraftBackend,
            )

            return FlashInferMultiStepDraftBackend(
                self.draft_model_runner, self.topk, self.speculative_num_steps
            )
        else:
            from sglang.srt.layers.attention.flashinfer_mla_backend import (
                FlashInferMLAMultiStepDraftBackend,
            )

            return FlashInferMLAMultiStepDraftBackend(
                self.draft_model_runner, self.topk, self.speculative_num_steps
            )
```
**EN:** This block uses `DraftBackendFactory._create_flashinfer_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_flashinfer_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 132-139: DraftBackendFactory._create_triton_decode_backend method
```python
    def _create_triton_decode_backend(self):
        from sglang.srt.layers.attention.triton_backend import (
            TritonMultiStepDraftBackend,
        )

        return TritonMultiStepDraftBackend(
            self.draft_model_runner, self.topk, self.speculative_num_steps
        )
```
**EN:** This block uses `DraftBackendFactory._create_triton_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_triton_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 141-146: DraftBackendFactory._create_aiter_decode_backend method
```python
    def _create_aiter_decode_backend(self):
        from sglang.srt.layers.attention.aiter_backend import AiterMultiStepDraftBackend

        return AiterMultiStepDraftBackend(
            self.draft_model_runner, self.topk, self.speculative_num_steps
        )
```
**EN:** This block uses `DraftBackendFactory._create_aiter_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_aiter_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 148-163: DraftBackendFactory._create_fa_decode_backend method
```python
    def _create_fa_decode_backend(self, fa_impl_ver: int = 3):
        if not is_musa():
            from sglang.srt.layers.attention.flashattention_backend import (
                FlashAttentionMultiStepBackend,
            )
        else:
            from sglang.srt.hardware_backend.musa.attention.flashattention_backend import (
                MusaFlashAttentionMultiStepBackend as FlashAttentionMultiStepBackend,
            )

        return FlashAttentionMultiStepBackend(
            self.draft_model_runner,
            self.topk,
            self.speculative_num_steps,
            fa_impl_ver=fa_impl_ver,
        )
```
**EN:** This block uses `DraftBackendFactory._create_fa_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_fa_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 165-166: DraftBackendFactory._create_fa3_decode_backend method
```python
    def _create_fa3_decode_backend(self):
        return self._create_fa_decode_backend(fa_impl_ver=3)
```
**EN:** This block uses `DraftBackendFactory._create_fa3_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_fa3_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 168-169: DraftBackendFactory._create_fa4_decode_backend method
```python
    def _create_fa4_decode_backend(self):
        return self._create_fa_decode_backend(fa_impl_ver=4)
```
**EN:** This block uses `DraftBackendFactory._create_fa4_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_fa4_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 171-178: DraftBackendFactory._create_flashmla_decode_backend method
```python
    def _create_flashmla_decode_backend(self):
        from sglang.srt.layers.attention.flashmla_backend import (
            FlashMLAMultiStepDraftBackend,
        )

        return FlashMLAMultiStepDraftBackend(
            self.draft_model_runner, self.topk, self.speculative_num_steps
        )
```
**EN:** This block uses `DraftBackendFactory._create_flashmla_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_flashmla_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 180-187: DraftBackendFactory._create_trtllm_mha_decode_backend method
```python
    def _create_trtllm_mha_decode_backend(self):
        from sglang.srt.layers.attention.trtllm_mha_backend import (
            TRTLLMHAAttnMultiStepDraftBackend,
        )

        return TRTLLMHAAttnMultiStepDraftBackend(
            self.draft_model_runner, self.topk, self.speculative_num_steps
        )
```
**EN:** This block uses `DraftBackendFactory._create_trtllm_mha_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_trtllm_mha_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 189-201: DraftBackendFactory._create_trtllm_mla_decode_backend method
```python
    def _create_trtllm_mla_decode_backend(self):
        if not get_global_server_args().use_mla_backend:
            raise ValueError(
                "trtllm_mla backend requires MLA model (use_mla_backend=True)."
            )

        from sglang.srt.layers.attention.trtllm_mla_backend import (
            TRTLLMMLAMultiStepDraftBackend,
        )

        return TRTLLMMLAMultiStepDraftBackend(
            self.draft_model_runner, self.topk, self.speculative_num_steps
        )
```
**EN:** This block uses `DraftBackendFactory._create_trtllm_mla_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_trtllm_mla_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 203-215: DraftBackendFactory._create_tokenspeed_mla_decode_backend method
```python
    def _create_tokenspeed_mla_decode_backend(self):
        if not get_global_server_args().use_mla_backend:
            raise ValueError(
                "tokenspeed_mla backend requires MLA model (use_mla_backend=True)."
            )

        from sglang.srt.layers.attention.tokenspeed_mla_backend import (
            TokenspeedMLAMultiStepDraftBackend,
        )

        return TokenspeedMLAMultiStepDraftBackend(
            self.draft_model_runner, self.topk, self.speculative_num_steps
        )
```
**EN:** This block uses `DraftBackendFactory._create_tokenspeed_mla_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_tokenspeed_mla_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 217-224: DraftBackendFactory._create_ascend_decode_backend method
```python
    def _create_ascend_decode_backend(self):
        from sglang.srt.hardware_backend.npu.attention.ascend_backend import (
            AscendAttnMultiStepDraftBackend,
        )

        return AscendAttnMultiStepDraftBackend(
            self.draft_model_runner, self.topk, self.speculative_num_steps
        )
```
**EN:** This block uses `DraftBackendFactory._create_ascend_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_ascend_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 226-233: DraftBackendFactory._create_dsv4_decode_backend method
```python
    def _create_dsv4_decode_backend(self):
        from sglang.srt.layers.attention.deepseek_v4_backend import (
            DeepseekV4MultiStepBackend,
        )

        return DeepseekV4MultiStepBackend(
            self.draft_model_runner, self.topk, self.speculative_num_steps
        )
```
**EN:** This block uses `DraftBackendFactory._create_dsv4_decode_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_dsv4_decode_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 235-247: DraftBackendFactory._create_flashinfer_prefill_backend method
```python
    def _create_flashinfer_prefill_backend(self):
        if not get_global_server_args().use_mla_backend:
            from sglang.srt.layers.attention.flashinfer_backend import (
                FlashInferAttnBackend,
            )

            return FlashInferAttnBackend(self.draft_model_runner, skip_prefill=False)
        else:
            from sglang.srt.layers.attention.flashinfer_mla_backend import (
                FlashInferMLAAttnBackend,
            )

            return FlashInferMLAAttnBackend(self.draft_model_runner, skip_prefill=False)
```
**EN:** This block uses `DraftBackendFactory._create_flashinfer_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_flashinfer_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 249-252: DraftBackendFactory._create_triton_prefill_backend method
```python
    def _create_triton_prefill_backend(self):
        from sglang.srt.layers.attention.triton_backend import TritonAttnBackend

        return TritonAttnBackend(self.draft_model_runner, skip_prefill=False)
```
**EN:** This block uses `DraftBackendFactory._create_triton_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_triton_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 254-257: DraftBackendFactory._create_aiter_prefill_backend method
```python
    def _create_aiter_prefill_backend(self):
        from sglang.srt.layers.attention.aiter_backend import AiterAttnBackend

        return AiterAttnBackend(self.draft_model_runner, skip_prefill=False)
```
**EN:** This block uses `DraftBackendFactory._create_aiter_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_aiter_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 259-270: DraftBackendFactory._create_fa_prefill_backend method
```python
    def _create_fa_prefill_backend(self, fa_impl_ver: int = 3):
        if not is_musa():
            from sglang.srt.layers.attention.flashattention_backend import (
                FlashAttentionBackend,
            )
        else:
            from sglang.srt.hardware_backend.musa.attention.flashattention_backend import (
                MusaFlashAttentionBackend as FlashAttentionBackend,
            )
        return FlashAttentionBackend(
            self.draft_model_runner, skip_prefill=False, fa_impl_ver=fa_impl_ver
        )
```
**EN:** This block uses `DraftBackendFactory._create_fa_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_fa_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 272-273: DraftBackendFactory._create_fa3_prefill_backend method
```python
    def _create_fa3_prefill_backend(self):
        return self._create_fa_prefill_backend(fa_impl_ver=3)
```
**EN:** This block uses `DraftBackendFactory._create_fa3_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_fa3_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 275-276: DraftBackendFactory._create_fa4_prefill_backend method
```python
    def _create_fa4_prefill_backend(self):
        return self._create_fa_prefill_backend(fa_impl_ver=4)
```
**EN:** This block uses `DraftBackendFactory._create_fa4_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_fa4_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 278-281: DraftBackendFactory._create_trtllm_mha_prefill_backend method
```python
    def _create_trtllm_mha_prefill_backend(self):
        from sglang.srt.layers.attention.trtllm_mha_backend import TRTLLMHAAttnBackend

        return TRTLLMHAAttnBackend(self.draft_model_runner, skip_prefill=False)
```
**EN:** This block uses `DraftBackendFactory._create_trtllm_mha_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_trtllm_mha_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 283-291: DraftBackendFactory._create_trtllm_mla_prefill_backend method
```python
    def _create_trtllm_mla_prefill_backend(self):
        if not get_global_server_args().use_mla_backend:
            raise ValueError(
                "trtllm_mla backend requires MLA model (use_mla_backend=True)."
            )

        from sglang.srt.layers.attention.trtllm_mla_backend import TRTLLMMLABackend

        return TRTLLMMLABackend(self.draft_model_runner, skip_prefill=False)
```
**EN:** This block uses `DraftBackendFactory._create_trtllm_mla_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_trtllm_mla_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 293-303: DraftBackendFactory._create_tokenspeed_mla_prefill_backend method
```python
    def _create_tokenspeed_mla_prefill_backend(self):
        if not get_global_server_args().use_mla_backend:
            raise ValueError(
                "tokenspeed_mla backend requires MLA model (use_mla_backend=True)."
            )

        from sglang.srt.layers.attention.tokenspeed_mla_backend import (
            TokenspeedMLABackend,
        )

        return TokenspeedMLABackend(self.draft_model_runner, skip_prefill=False)
```
**EN:** This block uses `DraftBackendFactory._create_tokenspeed_mla_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_tokenspeed_mla_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 305-310: DraftBackendFactory._create_ascend_prefill_backend method
```python
    def _create_ascend_prefill_backend(self):
        from sglang.srt.hardware_backend.npu.attention.ascend_backend import (
            AscendAttnBackend,
        )

        return AscendAttnBackend(self.draft_model_runner)
```
**EN:** This block uses `DraftBackendFactory._create_ascend_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_ascend_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 312-316: DraftBackendFactory._create_flashmla_prefill_backend method
```python
    def _create_flashmla_prefill_backend(self):
        logger.warning(
            "flashmla prefill backend is not yet supported for draft extend."
        )
        return None
```
**EN:** This block uses `DraftBackendFactory._create_flashmla_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_flashmla_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 318-323: DraftBackendFactory._create_dsv4_prefill_backend method
```python
    def _create_dsv4_prefill_backend(self):
        from sglang.srt.layers.attention.deepseek_v4_backend import (
            DeepseekV4AttnBackend,
        )

        return DeepseekV4AttnBackend(self.draft_model_runner, skip_prefill=False)
```
**EN:** This block uses `DraftBackendFactory._create_dsv4_prefill_backend` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `DraftBackendFactory._create_dsv4_prefill_backend` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Speculative decoding / 推测解码

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.hardware_backend.musa.attention.flashattention_backend`
- `sglang.srt.hardware_backend.npu.attention.ascend_backend`
- `sglang.srt.layers.attention.aiter_backend`
- `sglang.srt.layers.attention.deepseek_v4_backend`
- `sglang.srt.layers.attention.flashattention_backend`
- `sglang.srt.layers.attention.flashinfer_backend`
- `sglang.srt.layers.attention.flashinfer_mla_backend`
- `sglang.srt.layers.attention.flashmla_backend`
- `sglang.srt.layers.attention.nsa_backend`
- `sglang.srt.layers.attention.tokenspeed_mla_backend`
- `sglang.srt.layers.attention.triton_backend`
- `sglang.srt.layers.attention.trtllm_mha_backend`
- `sglang.srt.layers.attention.trtllm_mla_backend`
- `sglang.srt.server_args`
- `sglang.srt.utils.common`
### External / 外部
- `logging` (stdlib)
