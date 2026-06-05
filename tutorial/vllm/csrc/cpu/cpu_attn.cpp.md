# cpu_attn.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/cpu/cpu_attn.cpp`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements CPU attention kernels, dispatch paths, and ISA-specific specializations. / 实现 CPU 注意力内核、分派路径以及不同 ISA 的专用实现。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-4)
```cpp
#include "cpu_attn_dispatch_generated.h"

// Maps kv_cache_dtype string to Fp8KVCacheDataType enum.
// "auto" -> kAuto(0); "fp8"/"fp8_e4m3" -> kFp8E4M3; "fp8_e5m2" -> kFp8E5M2.
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Function / Kernel: parse_fp8_kv_dtype (lines 5-12)
```cpp
static inline cpu_attention::Fp8KVCacheDataType parse_fp8_kv_dtype(
    const std::string& kv_cache_dtype) {
  if (kv_cache_dtype == "fp8_e5m2")
    return cpu_attention::Fp8KVCacheDataType::kFp8E5M2;
  if (kv_cache_dtype == "fp8_e4m3" || kv_cache_dtype == "fp8")
    return cpu_attention::Fp8KVCacheDataType::kFp8E4M3;
  return cpu_attention::Fp8KVCacheDataType::kAuto;
}
```
**EN:** This function contains the CPU/host orchestration for attention, including data movement, shape handling, and backend dispatch.
**CN:** 该函数承载了注意力计算在 CPU/宿主侧的调度逻辑，包括数据搬运、形状处理和后端分派。

### Function / Kernel: get_scheduler_metadata (lines 14-81)
```cpp
torch::Tensor get_scheduler_metadata(
    const int64_t num_req, const int64_t num_heads_q,
    const int64_t num_heads_kv, const int64_t head_dim,
    const torch::Tensor& seq_lens, at::ScalarType dtype,
    const torch::Tensor& query_start_loc, const bool casual,
    const int64_t window_size, const std::string& isa_hint,
    const bool enable_kv_split) {
  cpu_attention::ISA isa;
  if (isa_hint == "amx") {
    isa = cpu_attention::ISA::AMX;
  } else if (isa_hint == "vec") {
    isa = cpu_attention::ISA::VEC;
  } else if (isa_hint == "vec16") {
    isa = cpu_attention::ISA::VEC16;
// ...
  torch::Tensor metadata = scheduler.schedule(input);
  return metadata;
}
```
**EN:** This function contains the CPU/host orchestration for attention, including data movement, shape handling, and backend dispatch.
**CN:** 该函数承载了注意力计算在 CPU/宿主侧的调度逻辑，包括数据搬运、形状处理和后端分派。

### Function / Kernel: cpu_attn_reshape_and_cache (lines 83-165)
```cpp
void cpu_attn_reshape_and_cache(
    const torch::Tensor& key,    // [token_num, head_num, head_size]
    const torch::Tensor& value,  // [token_num, head_num, head_size]
    torch::Tensor&
        key_cache,  // [num_blocks, num_kv_heads, block_size, head_size]
    torch::Tensor&
        value_cache,  // [num_blocks, num_kv_heads, block_size, head_size]
    const torch::Tensor& slot_mapping, const std::string& isa,
    const double k_scale = 1.0, const double v_scale = 1.0,
    const std::string& kv_cache_dtype = "auto") {
  TORCH_CHECK_EQ(key.dim(), 3);
  TORCH_CHECK_EQ(value.dim(), 3);
  TORCH_CHECK_EQ(key_cache.dim(), 4);
  TORCH_CHECK_EQ(value_cache.dim(), 4);
// ...
        });
      });
}
```
**EN:** This function contains the CPU/host orchestration for attention, including data movement, shape handling, and backend dispatch.
**CN:** 该函数承载了注意力计算在 CPU/宿主侧的调度逻辑，包括数据搬运、形状处理和后端分派。

### Function / Kernel: cpu_attention_with_kv_cache (lines 167-250)
```cpp
void cpu_attention_with_kv_cache(
    const torch::Tensor& query,  // [num_tokens, num_heads, head_size]
    const torch::Tensor&
        key_cache,  // [num_blocks, num_kv_heads, block_size, head_size]
    const torch::Tensor&
        value_cache,        // [num_blocks, num_kv_heads, block_size, head_size]
    torch::Tensor& output,  // [num_tokens, num_heads, head_size]
    const torch::Tensor& query_start_loc,  // [num_tokens + 1]
    const torch::Tensor& seq_lens,         // [num_tokens]
    const double scale, const bool causal,
    const std::optional<torch::Tensor>& alibi_slopes,  // [num_heads]
    const int64_t sliding_window_left, const int64_t sliding_window_right,
    const torch::Tensor& block_table,  // [num_tokens, max_block_num]
    const double softcap, const torch::Tensor& scheduler_metadata,
// ...
            });
      });
}
```
**EN:** This function contains the CPU/host orchestration for attention, including data movement, shape handling, and backend dispatch.
**CN:** 该函数承载了注意力计算在 CPU/宿主侧的调度逻辑，包括数据搬运、形状处理和后端分派。

## Key Concepts / 关键概念
- Attention computation / 注意力计算
- FP8 quantization / compute / FP8 量化与计算
- Integer kernel specialization / 整数内核专用化
- Intel AMX tiling / Intel AMX 分块
- SIMD vectorization / SIMD 向量化
- RISC-V vector support / RISC-V 向量支持
- ARM NEON specialization / ARM NEON 专用化
- Power VSX specialization / Power VSX 专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: C++ standard library or platform support
