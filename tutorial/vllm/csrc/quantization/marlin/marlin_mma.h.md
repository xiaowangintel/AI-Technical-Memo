# marlin_mma.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/marlin/marlin_mma.h`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Wrap architecture-specific Tensor Core MMA instructions for Marlin fragment types. / [CN] 为 Marlin 片段类型封装架构相关的 Tensor Core MMA 指令。

## Line-by-Line Analysis / 逐行分析
### Generic MMA wrapper and type dispatch
```cpp
template <vllm::ScalarTypeId type_id, bool use_fp16_accum, int k_size = 16>
__device__ inline void mma(
    const typename MarlinScalarType<type_id>::FragA& a_frag,
    const typename MarlinScalarType<type_id>::FragB& frag_b,
    typename MarlinScalarType<type_id>::FragC& frag_c, int idx = 0) {
  const uint32_t* a = reinterpret_cast<const uint32_t*>(&a_frag);
  const uint32_t* b = reinterpret_cast<const uint32_t*>(&frag_b);
```
**EN:** `mma` is the single entry point used by the main kernel. It reinterprets typed fragments as raw registers and then selects the correct PTX instruction sequence based on scalar type, K tile size, and whether FP16 accumulation is allowed.
**CN:** `mma` 是主内核统一调用的入口。它先把类型化片段重解释为寄存器数组，再根据标量类型、K tile 大小以及是否允许 FP16 累加来选择正确的 PTX 指令序列。

### FP16/BF16/FP8/INT8 `m16n8k16` paths
```cpp
asm volatile(
    "mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32 "
    "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9}, {%10,%11,%12,%13};
"
    : "=f"(c[0]), "=f"(c[1]), "=f"(c[2]), "=f"(c[3])
    : "r"(a[0]), "r"(a[1]), "r"(a[2]), "r"(a[3]), "r"(b[0]), "r"(b[1]),
      "f"(c[0]), "f"(c[1]), "f"(c[2]), "f"(c[3]));
```
**EN:** The header hardcodes PTX MMA forms for each supported data path: FP16→FP32, FP16→FP16, BF16→FP32, FP8→FP32, and INT8→INT32/FP32-style accumulation storage. Turing (`sm_75`) uses split `k8` sequences where necessary.
**CN:** 这个头文件为每条受支持的数据通路硬编码了 PTX MMA 形式：FP16→FP32、FP16→FP16、BF16→FP32、FP8→FP32，以及 INT8→INT32/FP32 风格的累加存储。对于 Turing (`sm_75`)，必要时会拆成两个 `k8` 序列。

### Wider `k_size == 32` path for 8-bit data
```cpp
} else if (k_size == 32) {
  if constexpr (std::is_same<scalar_t, __nv_fp8_e4m3>::value) {
    asm volatile(
        "mma.sync.aligned.m16n8k32.row.col.f32.e4m3.e4m3.f32 "
```
**EN:** FP8 and INT8 use `k32` MMA variants because 8-bit data packs twice as many values into the same register footprint. Marlin exposes this through the same wrapper instead of creating a separate API.
**CN:** FP8 和 INT8 采用 `k32` 版本的 MMA，因为 8-bit 数据能在相同寄存器占用下打包两倍元素。Marlin 通过同一个封装接口暴露这件事，而不是另起一套 API。

### `mma_trans` for the 8-row special case
```cpp
template <vllm::ScalarTypeId type_id, bool use_fp16_accum, int k_size = 16>
__device__ inline void mma_trans(
    const typename MarlinScalarType<type_id>::FragA& a_frag,
    const typename MarlinScalarType<type_id>::FragB& frag_b,
    const typename MarlinScalarType<type_id>::FragB& frag_b2,
    typename MarlinScalarType<type_id>::FragC& frag_c) {
```
**EN:** `mma_trans` swaps the operand interpretation so that Marlin can handle the `m_block_size_8` layout used for very small `M`. The code emits the same low-level instructions, but wires registers differently to match the shared-memory arrangement.
**CN:** `mma_trans` 交换了操作数的解释方式，使 Marlin 能处理极小 `M` 时使用的 `m_block_size_8` 布局。底层仍然发出相同类型的指令，只是按共享内存布局重新接线寄存器。

## Key Concepts / 关键概念
- PTX `mma.sync` is wrapped once and reused everywhere else. / PTX `mma.sync` 被统一封装，然后在其他地方复用。
- `k16` vs `k32` is driven by data packing width. / `k16` 和 `k32` 的区别由数据打包宽度决定。
- Turing gets compatibility fallbacks without changing call sites. / Turing 通过兼容分支获得支持，而调用点无需改变。

## Dependencies / 依赖关系
- Depends on `marlin_dtypes.cuh` for fragment definitions. / 依赖 `marlin_dtypes.cuh` 提供片段定义。
- Used directly by `marlin_template.h` during the GEMM inner loop. / 在 `marlin_template.h` 的 GEMM 内循环中被直接调用。
