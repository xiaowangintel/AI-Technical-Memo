# fp4_kv_cache_quant_method.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/fp4_kv_cache_quant_method.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements runtime method classes that apply fp4 kv cache quant method quantized weights during inference. / 该模块实现了推理期运行方法类，用于应用 FP4 KV 缓存 quant 方法 量化权重。

## Line-by-Line Analysis / 逐行分析
### Lines 14-19: module documentation
```python
"""
KV cache quantization strategy pattern.

Three-player design:
  quant_method (pure compute)  ►  Pool (buffer + batch dequant)  ►  Backend (view adaptation)
"""
```
**EN:** This block stores descriptive documentation for the surrounding module or class.
**CN:** 该代码块为周围模块或类保存说明性文档。

### Lines 21-27: module imports and setup
```python
from abc import ABC, abstractmethod
from typing import Optional

import torch
from torch import Tensor

from sglang.srt.layers.quantization.kvfp4_tensor import E2M1_MAX
```
**EN:** This block imports abc, torch, typing, sglang.srt.layers.quantization.kvfp4_tensor, sglang.srt.model_executor.model_runner and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 abc, torch, typing, sglang.srt.layers.quantization.kvfp4_tensor, sglang.srt.model_executor.model_runner 等依赖，并为当前量化实现准备模块命名空间。

### Lines 30-39: class FP4KVCacheQuantMethod: definition
```python
class FP4KVCacheQuantMethod(ABC):
    """Abstract base for FP4 KV cache quantization strategies.

    Owns the quantize/dequantize computation.  The Pool owns the buffers and
    orchestrates the batch dequant loop.  Backends only do view/reshape.

    All operations (quantize_and_store, dequantize_prev_kv) use FlashInfer
    kernels or pure tensor ops, so they are CUDA-graph compatible.
    """
```
**EN:** This block declares `FP4KVCacheQuantMethod`, a runtime method class for the quantization stack. It organizes behaviors such as needs_dequant_workspace, needs_global_scale, create_buffers, quantize_and_store.
**CN:** 该代码块声明 `FP4KVCacheQuantMethod`，它是量化栈中的运行方法类，组织了 needs_dequant_workspace, needs_global_scale, create_buffers, quantize_and_store 等行为。

### Lines 40-40: FP4KVCacheQuantMethod member: annotate name
```python
    name: str
```
**EN:** This block declares and initializes the annotated symbol `name`.
**CN:** 该代码块声明并初始化带类型注解的符号 `name`。

### Lines 41-41: FP4KVCacheQuantMethod member: annotate SCALE_BLOCK_SIZE
```python
    SCALE_BLOCK_SIZE: int = 1
```
**EN:** This block declares and initializes the annotated symbol `SCALE_BLOCK_SIZE`.
**CN:** 该代码块声明并初始化带类型注解的符号 `SCALE_BLOCK_SIZE`。

### Lines 43-45: FP4KVCacheQuantMethod.needs_dequant_workspace()
```python
    def needs_dequant_workspace(self) -> bool:
        """Whether the pool should allocate dq_k_buffer / dq_v_buffer for prefill."""
        return False
```
**EN:** This block defines `FP4KVCacheQuantMethod.needs_dequant_workspace()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `FP4KVCacheQuantMethod.needs_dequant_workspace()`，用于为后续计算恢复低精度数据。

### Lines 47-49: FP4KVCacheQuantMethod.needs_global_scale()
```python
    def needs_global_scale(self) -> bool:
        """Whether this method uses a per-layer global FP32 scale."""
        return False
```
**EN:** This block defines `FP4KVCacheQuantMethod.needs_global_scale()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `FP4KVCacheQuantMethod.needs_global_scale()`，用于实现量化栈中的可复用模块逻辑。

### Lines 51-65: FP4KVCacheQuantMethod.create_buffers()
```python
    @abstractmethod
    def create_buffers(
        self, size: int, head_num: int, head_dim: int, layer_num: int, device: str
    ) -> dict:
        """Allocate and return a buffer dict:
        {
            "k_buffer": list[Tensor],       # per-layer, shape (size, head_num, head_dim//2)
            "v_buffer": list[Tensor],
            "k_scale_buffer": list[Tensor] | None,
            "v_scale_buffer": list[Tensor] | None,
            "dq_k_buffer": Tensor | None,   # shared across layers (FP8 E4M3)
            "dq_v_buffer": Tensor | None,
            "store_dtype": torch.dtype,
        }
        """
```
**EN:** This block defines `FP4KVCacheQuantMethod.create_buffers()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `FP4KVCacheQuantMethod.create_buffers()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 67-80: FP4KVCacheQuantMethod.quantize_and_store()
```python
    @abstractmethod
    def quantize_and_store(
        self,
        k_buffer: Tensor,
        v_buffer: Tensor,
        k_scale_buffer: Optional[Tensor],
        v_scale_buffer: Optional[Tensor],
        loc: Tensor,
        cache_k: Tensor,
        cache_v: Tensor,
        k_scale=None,
        v_scale=None,
    ) -> None:
        """Quantize cache_k / cache_v and write into buffers at loc."""
```
**EN:** This block defines `FP4KVCacheQuantMethod.quantize_and_store()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `FP4KVCacheQuantMethod.quantize_and_store()`，用于处理张量或权重量化逻辑。

### Lines 82-97: FP4KVCacheQuantMethod.dequantize_prev_kv()
```python
    @abstractmethod
    def dequantize_prev_kv(
        self,
        k_fp4: Tensor,
        k_scales: Tensor,
        v_fp4: Tensor,
        v_scales: Tensor,
        layer_id: int,
    ) -> tuple[Tensor, Tensor]:
        """Dequantize stored FP4 KV (selected token indices already applied).

        Returns:
            (k_fp8, v_fp8): Both in torch.float8_e4m3fn dtype with shape
            matching the input (after unpacking). These are written into the
            shared dequant workspace buffer for the FlashInfer FP8 prefill kernel.
        """
```
**EN:** This block defines `FP4KVCacheQuantMethod.dequantize_prev_kv()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `FP4KVCacheQuantMethod.dequantize_prev_kv()`，用于为后续计算恢复低精度数据。

### Lines 99-103: FP4KVCacheQuantMethod.compute_cell_size()
```python
    @abstractmethod
    def compute_cell_size(
        self, head_num: int, head_dim: int, num_layers: int, kv_size: int
    ) -> int:
        """Per-token memory footprint in bytes (for capacity estimation)."""
```
**EN:** This block defines `FP4KVCacheQuantMethod.compute_cell_size()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `FP4KVCacheQuantMethod.compute_cell_size()`，用于实现量化栈中的可复用模块逻辑。

### Lines 105-107: FP4KVCacheQuantMethod.load_scales_from_model()
```python
    def load_scales_from_model(self, model_runner, sm_version: int = None) -> None:
        """Load per-layer global scales from model weights (no-op by default)."""
        pass
```
**EN:** This block defines `FP4KVCacheQuantMethod.load_scales_from_model()`, which handles checkpoint loading or layout conversion.
**CN:** 该代码块定义了 `FP4KVCacheQuantMethod.load_scales_from_model()`，用于处理检查点加载或布局转换。

### Lines 110-115: class NVFP4KVMethod: definition
```python
class NVFP4KVMethod(FP4KVCacheQuantMethod):
    """NVFP4 two-level scaling: global FP32 + per-block FP8 E4M3.

    Supported on SM100 and SM120.
    """
```
**EN:** This block declares `NVFP4KVMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, needs_dequant_workspace, needs_global_scale, load_scales_from_model.
**CN:** 该代码块声明 `NVFP4KVMethod`，它是量化栈中的运行方法类，组织了 __init__, needs_dequant_workspace, needs_global_scale, load_scales_from_model 等行为。

### Lines 116-116: NVFP4KVMethod member: initialize name
```python
    name = "nvfp4"
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as name.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 name。

### Lines 117-117: NVFP4KVMethod member: initialize SCALE_BLOCK_SIZE
```python
    SCALE_BLOCK_SIZE = 16
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as SCALE_BLOCK_SIZE.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 SCALE_BLOCK_SIZE。

### Lines 119-125: NVFP4KVMethod.__init__()
```python
    def __init__(self, num_layers: int, device: str, sm_version: int = 120):
        self.num_layers = num_layers
        self.device = device
        self.sm_version = sm_version
        # Per-layer global FP32 scales; filled by load_scales_from_model()
        self.k_scales_gpu = torch.ones(num_layers, dtype=torch.float32, device=device)
        self.v_scales_gpu = torch.ones(num_layers, dtype=torch.float32, device=device)
```
**EN:** This block defines `NVFP4KVMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `NVFP4KVMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 127-130: NVFP4KVMethod.needs_dequant_workspace()
```python
    def needs_dequant_workspace(self) -> bool:
        return (
            True  # prefill uses FP8 dequant workspace; future native FP4 kernel → False
        )
```
**EN:** This block defines `NVFP4KVMethod.needs_dequant_workspace()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `NVFP4KVMethod.needs_dequant_workspace()`，用于为后续计算恢复低精度数据。

### Lines 132-133: NVFP4KVMethod.needs_global_scale()
```python
    def needs_global_scale(self) -> bool:
        return True
```
**EN:** This block defines `NVFP4KVMethod.needs_global_scale()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `NVFP4KVMethod.needs_global_scale()`，用于实现量化栈中的可复用模块逻辑。

### Lines 135-202: NVFP4KVMethod.load_scales_from_model()
```python
    def load_scales_from_model(self, model_runner, sm_version: int = None) -> None:
        if sm_version is not None:
            self.sm_version = sm_version

        from sglang.srt.model_executor.model_runner import resolve_language_model

        language_model = resolve_language_model(model_runner.model)

        attention_layers = []
        for layer in language_model.layers:
            if hasattr(layer, "self_attn"):
                if hasattr(layer.self_attn, "attn"):
                    attention_layers.append(layer.self_attn.attn)
                elif hasattr(layer.self_attn, "attn_mqa"):
                    attention_layers.append(layer.self_attn.attn_mqa)
            elif hasattr(layer, "attn"):
                attention_layers.append(layer.attn)
            elif hasattr(layer, "attention"):
                if hasattr(layer.attention, "attn"):
                    attention_layers.append(layer.attention.attn)

        if not attention_layers:
            return

        # k_scales_gpu is indexed by global (absolute) layer_id.  Resize if the model
        # has layers with global IDs larger than what was pre-allocated.
        # This happens in hybrid models (e.g., GDN) where only a subset of layers
        # are full-attention, but their layer_ids are non-contiguous.
        max_global_id = max(layer.layer_id for layer in attention_layers)
        required_size = max_global_id + 1
        if required_size > len(self.k_scales_gpu):
            self.k_scales_gpu = torch.ones(
                required_size, dtype=torch.float32, device=self.device
            )
            self.v_scales_gpu = torch.ones(
                required_size, dtype=torch.float32, device=self.device
            )

        k_scales_cpu = self.k_scales_gpu.cpu().clone()
        v_scales_cpu = self.v_scales_gpu.cpu().clone()

        for layer in attention_layers:
            layer_id = layer.layer_id  # global id
            k_scale = (
                float(layer.k_scale)
                if hasattr(layer, "k_scale") and layer.k_scale is not None
                else 1.0
            )
            v_scale = (
                float(layer.v_scale)
                if hasattr(layer, "v_scale") and layer.v_scale is not None
                else 1.0
            )
            # SM100 uses TRT-LLM XQA kernels that expect KV scales as
            # amax / 448, but the calibrated checkpoint stores amax / (6 * 448).
            # We multiply by E2M1_MAX (6.0) to bridge the gap.  SM120 uses a
            # different kernel path where scales already include this factor.
            # The FP4 data type itself is identical on both architectures.
            # Reference: TRT-LLM FP8QDQLinearMethod.process_weights_after_loading_fused_qkv_linear
            # https://github.com/NVIDIA/TensorRT-LLM/blob/main/tensorrt_llm/_torch/modules/linear.py
            if self.sm_version == 100:
                k_scale *= E2M1_MAX
                v_scale *= E2M1_MAX
            k_scales_cpu[layer_id] = k_scale
            v_scales_cpu[layer_id] = v_scale

        self.k_scales_gpu.copy_(k_scales_cpu, non_blocking=True)
        self.v_scales_gpu.copy_(v_scales_cpu, non_blocking=True)
```
**EN:** This block defines `NVFP4KVMethod.load_scales_from_model()`, which handles checkpoint loading or layout conversion.
**CN:** 该代码块定义了 `NVFP4KVMethod.load_scales_from_model()`，用于处理检查点加载或布局转换。

### Lines 204-245: NVFP4KVMethod.create_buffers()
```python
    def create_buffers(
        self, size: int, head_num: int, head_dim: int, layer_num: int, device: str
    ) -> dict:
        m = size
        n = head_num
        k = head_dim
        store_dtype = torch.uint8
        dq_dtype = torch.float8_e4m3fn

        k_buffer = [
            torch.zeros((m, n, k // 2), dtype=store_dtype, device=device)
            for _ in range(layer_num)
        ]
        v_buffer = [
            torch.zeros((m, n, k // 2), dtype=store_dtype, device=device)
            for _ in range(layer_num)
        ]
        k_scale_buffer = [
            torch.zeros(
                (m, n, k // self.SCALE_BLOCK_SIZE), dtype=store_dtype, device=device
            )
            for _ in range(layer_num)
        ]
        v_scale_buffer = [
            torch.zeros(
                (m, n, k // self.SCALE_BLOCK_SIZE), dtype=store_dtype, device=device
            )
            for _ in range(layer_num)
        ]
        # Shared dequant workspace — one copy, reused per layer during prefill
        dq_k_buffer = torch.zeros((m, n, k), dtype=dq_dtype, device=device)
        dq_v_buffer = torch.zeros((m, n, k), dtype=dq_dtype, device=device)

        return {
            "k_buffer": k_buffer,
            "v_buffer": v_buffer,
            "k_scale_buffer": k_scale_buffer,
            "v_scale_buffer": v_scale_buffer,
            "dq_k_buffer": dq_k_buffer,
            "dq_v_buffer": dq_v_buffer,
            "store_dtype": store_dtype,
        }
```
**EN:** This block defines `NVFP4KVMethod.create_buffers()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `NVFP4KVMethod.create_buffers()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 247-271: NVFP4KVMethod.quantize_and_store()
```python
    def quantize_and_store(
        self,
        k_buffer: Tensor,
        v_buffer: Tensor,
        k_scale_buffer: Optional[Tensor],
        v_scale_buffer: Optional[Tensor],
        loc: Tensor,
        cache_k: Tensor,
        cache_v: Tensor,
        k_scale=None,
        v_scale=None,
    ) -> None:
        from sglang.srt.layers.quantization.kvfp4_tensor import NVFP4KVQuantizeUtil

        cache_k, cache_k_fp4_sf, _ = NVFP4KVQuantizeUtil.quantize(
            cache_k.contiguous(), k_scale
        )
        cache_v, cache_v_fp4_sf, _ = NVFP4KVQuantizeUtil.quantize(
            cache_v.contiguous(), v_scale
        )

        k_buffer[loc] = cache_k.view(torch.uint8)
        v_buffer[loc] = cache_v.view(torch.uint8)
        k_scale_buffer[loc] = cache_k_fp4_sf.view(torch.uint8)
        v_scale_buffer[loc] = cache_v_fp4_sf.view(torch.uint8)
```
**EN:** This block defines `NVFP4KVMethod.quantize_and_store()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `NVFP4KVMethod.quantize_and_store()`，用于处理张量或权重量化逻辑。

### Lines 273-292: NVFP4KVMethod.dequantize_prev_kv()
```python
    def dequantize_prev_kv(
        self,
        k_fp4: Tensor,
        k_scales: Tensor,
        v_fp4: Tensor,
        v_scales: Tensor,
        layer_id: int,
    ) -> tuple[Tensor, Tensor]:
        """Dequantize FP4 KV (indexed tokens) → FP8 E4M3."""
        from sglang.srt.layers.quantization.kvfp4_tensor import NVFP4KVQuantizeUtil

        cur_k_scale = self.k_scales_gpu[layer_id : layer_id + 1]
        cur_v_scale = self.v_scales_gpu[layer_id : layer_id + 1]
        k_bf16 = NVFP4KVQuantizeUtil.dequantize(
            k_fp4.view(torch.uint8), k_scales, cur_k_scale
        )
        v_bf16 = NVFP4KVQuantizeUtil.dequantize(
            v_fp4.view(torch.uint8), v_scales, cur_v_scale
        )
        return k_bf16.to(torch.float8_e4m3fn), v_bf16.to(torch.float8_e4m3fn)
```
**EN:** This block defines `NVFP4KVMethod.dequantize_prev_kv()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `NVFP4KVMethod.dequantize_prev_kv()`，用于为后续计算恢复低精度数据。

### Lines 294-305: NVFP4KVMethod.compute_cell_size()
```python
    def compute_cell_size(
        self, head_num: int, head_dim: int, num_layers: int, kv_size: int
    ) -> int:
        # FP4 data: per-layer, K+V
        fp4_size = head_num * (head_dim // 2) * num_layers * 2 * kv_size
        # Block scales: per-layer, K+V (uint8)
        scale_size = (
            head_num * (head_dim // self.SCALE_BLOCK_SIZE) * num_layers * 2 * kv_size
        )
        # Dequant workspace: shared across layers (not multiplied by num_layers), FP8
        dq_size = head_num * head_dim * 2 * kv_size
        return fp4_size + scale_size + dq_size
```
**EN:** This block defines `NVFP4KVMethod.compute_cell_size()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `NVFP4KVMethod.compute_cell_size()`，用于实现量化栈中的可复用模块逻辑。

### Lines 308-310: class BlockFP4KVMethod: definition
```python
class BlockFP4KVMethod(FP4KVCacheQuantMethod):
    """Block-wise FP4 single-level scaling (similar to MXFP4 but block_size=16)."""
```
**EN:** This block declares `BlockFP4KVMethod`, a runtime method class for the quantization stack. It organizes behaviors such as needs_dequant_workspace, create_buffers, quantize_and_store, dequantize_prev_kv.
**CN:** 该代码块声明 `BlockFP4KVMethod`，它是量化栈中的运行方法类，组织了 needs_dequant_workspace, create_buffers, quantize_and_store, dequantize_prev_kv 等行为。

### Lines 311-311: BlockFP4KVMethod member: initialize name
```python
    name = "blockfp4"
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as name.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 name。

### Lines 312-312: BlockFP4KVMethod member: initialize SCALE_BLOCK_SIZE
```python
    SCALE_BLOCK_SIZE = 16
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as SCALE_BLOCK_SIZE.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 SCALE_BLOCK_SIZE。

### Lines 314-315: BlockFP4KVMethod.needs_dequant_workspace()
```python
    def needs_dequant_workspace(self) -> bool:
        return True
```
**EN:** This block defines `BlockFP4KVMethod.needs_dequant_workspace()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `BlockFP4KVMethod.needs_dequant_workspace()`，用于为后续计算恢复低精度数据。

### Lines 317-364: BlockFP4KVMethod.create_buffers()
```python
    def create_buffers(
        self, size: int, head_num: int, head_dim: int, layer_num: int, device: str
    ) -> dict:
        m = size
        store_dtype = torch.uint8
        dq_dtype = torch.float8_e4m3fn

        k_buffer = [
            torch.zeros((m, head_num, head_dim // 2), dtype=store_dtype, device=device)
            for _ in range(layer_num)
        ]
        v_buffer = [
            torch.zeros((m, head_num, head_dim // 2), dtype=store_dtype, device=device)
            for _ in range(layer_num)
        ]
        # MXFP4 flattens head dimensions for scale storage
        k_scale_buffer = [
            torch.zeros(
                (m, (head_num * head_dim) // self.SCALE_BLOCK_SIZE),
                dtype=store_dtype,
                device=device,
            )
            for _ in range(layer_num)
        ]
        v_scale_buffer = [
            torch.zeros(
                (m, (head_num * head_dim) // self.SCALE_BLOCK_SIZE),
                dtype=store_dtype,
                device=device,
            )
            for _ in range(layer_num)
        ]
        dq_k_buffer = torch.zeros(
            (m, head_num, head_dim), dtype=dq_dtype, device=device
        )
        dq_v_buffer = torch.zeros(
            (m, head_num, head_dim), dtype=dq_dtype, device=device
        )

        return {
            "k_buffer": k_buffer,
            "v_buffer": v_buffer,
            "k_scale_buffer": k_scale_buffer,
            "v_scale_buffer": v_scale_buffer,
            "dq_k_buffer": dq_k_buffer,
            "dq_v_buffer": dq_v_buffer,
            "store_dtype": store_dtype,
        }
```
**EN:** This block defines `BlockFP4KVMethod.create_buffers()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `BlockFP4KVMethod.create_buffers()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 366-385: BlockFP4KVMethod.quantize_and_store()
```python
    def quantize_and_store(
        self,
        k_buffer,
        v_buffer,
        k_scale_buffer,
        v_scale_buffer,
        loc,
        cache_k,
        cache_v,
        k_scale=None,
        v_scale=None,
    ) -> None:
        from sglang.srt.layers.quantization.kvfp4_tensor import BlockFP4KVQuantizeUtil

        cache_k_fp4, cache_k_sf = BlockFP4KVQuantizeUtil.batched_quantize(cache_k)
        cache_v_fp4, cache_v_sf = BlockFP4KVQuantizeUtil.batched_quantize(cache_v)
        k_buffer[loc] = cache_k_fp4
        v_buffer[loc] = cache_v_fp4
        k_scale_buffer[loc] = cache_k_sf
        v_scale_buffer[loc] = cache_v_sf
```
**EN:** This block defines `BlockFP4KVMethod.quantize_and_store()`, which handles tensor or weight quantization work.
**CN:** 该代码块定义了 `BlockFP4KVMethod.quantize_and_store()`，用于处理张量或权重量化逻辑。

### Lines 387-399: BlockFP4KVMethod.dequantize_prev_kv()
```python
    def dequantize_prev_kv(
        self,
        k_fp4: Tensor,
        k_scales: Tensor,
        v_fp4: Tensor,
        v_scales: Tensor,
        layer_id: int,
    ) -> tuple[Tensor, Tensor]:
        from sglang.srt.layers.quantization.kvfp4_tensor import BlockFP4KVQuantizeUtil

        k_bf16 = BlockFP4KVQuantizeUtil.batched_dequantize(k_fp4, k_scales)
        v_bf16 = BlockFP4KVQuantizeUtil.batched_dequantize(v_fp4, v_scales)
        return k_bf16.to(torch.float8_e4m3fn), v_bf16.to(torch.float8_e4m3fn)
```
**EN:** This block defines `BlockFP4KVMethod.dequantize_prev_kv()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `BlockFP4KVMethod.dequantize_prev_kv()`，用于为后续计算恢复低精度数据。

### Lines 401-409: BlockFP4KVMethod.compute_cell_size()
```python
    def compute_cell_size(
        self, head_num: int, head_dim: int, num_layers: int, kv_size: int
    ) -> int:
        fp4_size = head_num * (head_dim // 2) * num_layers * 2 * kv_size
        scale_size = (
            (head_num * head_dim // self.SCALE_BLOCK_SIZE) * num_layers * 2 * kv_size
        )
        dq_size = head_num * head_dim * 2 * kv_size
        return fp4_size + scale_size + dq_size
```
**EN:** This block defines `BlockFP4KVMethod.compute_cell_size()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `BlockFP4KVMethod.compute_cell_size()`，用于实现量化栈中的可复用模块逻辑。

### Lines 413-416: annotate FP4_KV_CACHE_QUANT_REGISTRY
```python
FP4_KV_CACHE_QUANT_REGISTRY: dict[str, type[FP4KVCacheQuantMethod]] = {
    "nvfp4": NVFP4KVMethod,
    "blockfp4": BlockFP4KVMethod,
}
```
**EN:** This block declares and initializes the annotated symbol `FP4_KV_CACHE_QUANT_REGISTRY`.
**CN:** 该代码块声明并初始化带类型注解的符号 `FP4_KV_CACHE_QUANT_REGISTRY`。

### Lines 419-426: get_fp4_kv_cache_quant_method()
```python
def get_fp4_kv_cache_quant_method(name: str, **kwargs) -> FP4KVCacheQuantMethod:
    """Instantiate a FP4KVCacheQuantMethod by recipe name."""
    if name not in FP4_KV_CACHE_QUANT_REGISTRY:
        raise ValueError(
            f"Unknown fp4_kv_cache_recipe: '{name}'. "
            f"Available: {list(FP4_KV_CACHE_QUANT_REGISTRY)}"
        )
    return FP4_KV_CACHE_QUANT_REGISTRY[name](**kwargs)
```
**EN:** This block defines `get_fp4_kv_cache_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `get_fp4_kv_cache_quant_method()`，用于为调用方获取或计算派生值。

## Key Concepts / 关键概念
- `FP4KVCacheQuantMethod`: A runtime method class that structures file-level quantization behavior. / `FP4KVCacheQuantMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `NVFP4KVMethod`: A runtime method class that structures file-level quantization behavior. / `NVFP4KVMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `BlockFP4KVMethod`: A runtime method class that structures file-level quantization behavior. / `BlockFP4KVMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `get_fp4_kv_cache_quant_method()` : A public function that retrieves or computes a derived value for callers. / `get_fp4_kv_cache_quant_method()`：一个公开函数，用于为调用方获取或计算派生值。

## Dependencies / 依赖关系
- **External / 外部**: `abc`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.layers.quantization.kvfp4_tensor`, `sglang.srt.model_executor.model_runner`
