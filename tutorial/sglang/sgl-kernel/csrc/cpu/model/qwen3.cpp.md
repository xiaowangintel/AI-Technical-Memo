# qwen3.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cpu/model/qwen3.cpp`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides CPU-side reference implementations, preprocessing, or fallback execution paths. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 CPU 侧参考实现、预处理或回退执行路径。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Namespace and shared declarations
```cpp
#include "common.h"
#include "vec.h"
namespace {

template <typename scalar_t>
inline void copy_stub(scalar_t* __restrict__ out, const scalar_t* __restrict__ src, int64_t size) {
  using bVec = at::vec::Vectorized<scalar_t>;
  constexpr int kVecSize = bVec::size();
  int64_t d = 0;
#pragma GCC unroll 4
  for (; d <= size - kVecSize; d += kVecSize) {
    bVec out_bvec = bVec::loadu(src + d);
    out_bvec.store(out + d);
  }
  for (; d < size; ++d) {
    out[d] = src[d];
  }
}
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 20-41: Runtime integration and dispatch
```cpp
template <typename scalar_t>
void fused_qkvzba_split_reshape_cat_impl(
    const scalar_t* __restrict__ mixed_qkvz,
    const scalar_t* __restrict__ mixed_ba,
    scalar_t* __restrict__ mixed_qkv,
    scalar_t* __restrict__ z,
    scalar_t* __restrict__ b,
    scalar_t* __restrict__ a,
    int64_t batch,
    int64_t num_heads_qk,
    int64_t num_heads_v,
    int64_t head_qk,
    int64_t group,
    int64_t head_v,
    int64_t qkv_strideB,
    int64_t qkvz_strideB,
    int64_t ba_strideB) {
  int64_t qkvz_stride_per_head = head_qk * 2 + head_v * 2 * group;
  at::parallel_for(0, batch * num_heads_qk, 0, [&](int64_t begin, int64_t end) {
    int64_t bi{0}, hi{0};
    data_index_init(begin, bi, batch, hi, num_heads_qk);
    for (int64_t i = begin; i < end; ++i) {
```
**EN:** This section uses `fused_qkvzba_split_reshape_cat_impl`, `at::parallel_for`, `data_index_init` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fused_qkvzba_split_reshape_cat_impl`、`at::parallel_for`、`data_index_init`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 42-63: Local implementation details
```cpp
      scalar_t* __restrict__ q_out_ptr = mixed_qkv + bi * qkv_strideB + hi * head_qk;
      const scalar_t* __restrict__ q_in_ptr = mixed_qkvz + bi * qkvz_strideB + hi * qkvz_stride_per_head;
      scalar_t* __restrict__ k_out_ptr = q_out_ptr + num_heads_qk * head_qk;
      const scalar_t* __restrict__ k_in_ptr = q_in_ptr + head_qk;
      scalar_t* __restrict__ v_out_ptr = k_out_ptr + num_heads_qk * head_qk + hi * head_qk * (group - 1);
      const scalar_t* __restrict__ v_in_ptr = k_in_ptr + head_qk;
      scalar_t* __restrict__ z_out_ptr = z + bi * num_heads_v * head_v + hi * group * head_v;
      const scalar_t* __restrict__ z_in_ptr = v_in_ptr + head_qk * group;
      copy_stub(q_out_ptr, q_in_ptr, head_qk);
      copy_stub(k_out_ptr, k_in_ptr, head_qk);
      copy_stub(v_out_ptr, v_in_ptr, head_qk * group);
      copy_stub(z_out_ptr, z_in_ptr, head_qk * group);
      scalar_t* __restrict__ b_out_ptr = b + bi * num_heads_v + hi * group;
      const scalar_t* __restrict__ b_in_ptr = mixed_ba + bi * ba_strideB + hi * group * 2;
      scalar_t* __restrict__ a_out_ptr = a + bi * num_heads_v + hi * group;
      const scalar_t* __restrict__ a_in_ptr = b_in_ptr + group;
      copy_stub(b_out_ptr, b_in_ptr, group);
      copy_stub(a_out_ptr, a_in_ptr, group);
      data_index_step(bi, batch, hi, num_heads_qk);
    }
  });
}
```
**EN:** This section fills in the local implementation details around `copy_stub`, `data_index_step`, `q_in_ptr`, completing the behavior required by the file.
**CN:** 本段补充了`copy_stub`、`data_index_step`、`q_in_ptr`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 64-85: Runtime integration and dispatch
```cpp

template <typename scalar_t>
void fused_qkvzba_split_reshape_cat_contiguous_impl(
    const scalar_t* __restrict__ mixed_qkvz,
    const scalar_t* __restrict__ mixed_ba,
    scalar_t* __restrict__ mixed_qkv,
    scalar_t* __restrict__ z,
    scalar_t* __restrict__ b,
    scalar_t* __restrict__ a,
    int64_t batch,
    int64_t k_tp,
    int64_t v_tp,
    int64_t num_heads_v,
    int64_t qkv_dim,
    int64_t qkv_strideB,
    int64_t qkvz_strideB,
    int64_t ba_strideB) {
  at::parallel_for(0, batch, 0, [&](int64_t begin, int64_t end) {
    for (int64_t bi = begin; bi < end; ++bi) {
      scalar_t* __restrict__ qkv_out_ptr = mixed_qkv + bi * qkv_strideB;
      const scalar_t* __restrict__ qkv_in_ptr = mixed_qkvz + bi * qkvz_strideB;
      scalar_t* __restrict__ z_out_ptr = z + bi * v_tp;
```
**EN:** This section uses `fused_qkvzba_split_reshape_cat_contiguous_impl`, `at::parallel_for`, `qkv_in_ptr` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fused_qkvzba_split_reshape_cat_contiguous_impl`、`at::parallel_for`、`qkv_in_ptr`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 86-107: Runtime integration and dispatch
```cpp
      const scalar_t* __restrict__ z_in_ptr = qkv_in_ptr + qkv_dim;
      copy_stub(qkv_out_ptr, qkv_in_ptr, qkv_dim);
      copy_stub(z_out_ptr, z_in_ptr, v_tp);
      scalar_t* __restrict__ b_out_ptr = b + bi * num_heads_v;
      const scalar_t* __restrict__ b_in_ptr = mixed_ba + bi * ba_strideB;
      scalar_t* __restrict__ a_out_ptr = a + bi * num_heads_v;
      const scalar_t* __restrict__ a_in_ptr = b_in_ptr + num_heads_v;
      copy_stub(b_out_ptr, b_in_ptr, num_heads_v);
      copy_stub(a_out_ptr, a_in_ptr, num_heads_v);
    }
  });
}

}  // anonymous namespace

// mixed_qkvz: [batch, num_heads_qk * head_qk * 2 + num_heads_v * head_v * 2]
// mixed_ba: [batch, num_heads_v * 2]
std::tuple<at::Tensor, at::Tensor, at::Tensor, at::Tensor> fused_qkvzba_split_reshape_cat_cpu(
    const at::Tensor& mixed_qkvz,
    const at::Tensor& mixed_ba,
    int64_t num_heads_qk,
    int64_t num_heads_v,
```
**EN:** This section uses `copy_stub`, `z_in_ptr`, `b_in_ptr` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`copy_stub`、`z_in_ptr`、`b_in_ptr`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 108-129: Runtime integration and dispatch
```cpp
    int64_t head_qk,
    int64_t head_v) {
  CHECK_DIM(2, mixed_qkvz);
  CHECK_DIM(2, mixed_ba);
  CHECK_INPUT(mixed_qkvz);
  CHECK_INPUT(mixed_ba);
  int64_t batch = mixed_qkvz.size(0);
  int64_t qkv_dim = num_heads_qk * head_qk * 2 + num_heads_v * head_v;
  int64_t ba_dim = num_heads_v * 2;
  int64_t expected_dim = qkv_dim + num_heads_v * head_v;
  CHECK_EQ(mixed_qkvz.size(1), expected_dim);
  CHECK_EQ(mixed_ba.size(0), batch);
  CHECK_EQ(mixed_ba.size(1), ba_dim);
  TORCH_CHECK(mixed_ba.scalar_type() == mixed_qkvz.scalar_type(), "mixed_ba and mixed_qkvz must share same dtype");
  CHECK_EQ(num_heads_v % num_heads_qk, 0);
  at::Tensor mixed_qkv = at::empty({batch, qkv_dim}, mixed_qkvz.options());
  at::Tensor z = at::empty({batch, num_heads_v, head_v}, mixed_qkvz.options());
  at::Tensor b = at::empty({batch, num_heads_v}, mixed_ba.options());
  at::Tensor a = at::empty({batch, num_heads_v}, mixed_ba.options());
  int64_t group = num_heads_v / num_heads_qk;
  int64_t qkvz_strideB = mixed_qkvz.size(1);
  int64_t qkv_strideB = mixed_qkv.size(1);
```
**EN:** This section uses `CHECK_DIM`, `CHECK_INPUT`, `CHECK_EQ` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_DIM`、`CHECK_INPUT`、`CHECK_EQ`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 130-151: Local implementation details
```cpp
  int64_t ba_strideB = mixed_ba.size(1);
  AT_DISPATCH_REDUCED_FLOATING_TYPES(mixed_qkvz.scalar_type(), "fused_qkvzba_split_reshape_cat_impl", [&] {
    fused_qkvzba_split_reshape_cat_impl<scalar_t>(
        mixed_qkvz.data_ptr<scalar_t>(),
        mixed_ba.data_ptr<scalar_t>(),
        mixed_qkv.data_ptr<scalar_t>(),
        z.data_ptr<scalar_t>(),
        b.data_ptr<scalar_t>(),
        a.data_ptr<scalar_t>(),
        batch,
        num_heads_qk,
        num_heads_v,
        head_qk,
        group,
        head_v,
        qkv_strideB,
        qkvz_strideB,
        ba_strideB);
  });
  return std::make_tuple(mixed_qkv, z, b, a);
}
```
**EN:** This section fills in the local implementation details around `make_tuple`, completing the behavior required by the file.
**CN:** 本段补充了`make_tuple`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 152-173: Runtime integration and dispatch
```cpp
// mixed_qkvz: [batch, num_heads_qk * head_qk * 2 + num_heads_v * head_v * 2]
// mixed_ba: [batch, num_heads_v * 2]
std::tuple<at::Tensor, at::Tensor, at::Tensor, at::Tensor> fused_qkvzba_split_reshape_cat_contiguous_cpu(
    const at::Tensor& mixed_qkvz,
    const at::Tensor& mixed_ba,
    int64_t num_heads_qk,
    int64_t num_heads_v,
    int64_t head_qk,
    int64_t head_v) {
  CHECK_DIM(2, mixed_qkvz);
  CHECK_DIM(2, mixed_ba);
  CHECK_INPUT(mixed_qkvz);
  CHECK_INPUT(mixed_ba);
  int64_t batch = mixed_qkvz.size(0);
  int64_t k_tp = num_heads_qk * head_qk;
  int64_t v_tp = num_heads_v * head_v;
  int64_t qkv_dim = k_tp * 2 + v_tp;
  int64_t ba_dim = num_heads_v * 2;
  int64_t expected_dim = qkv_dim + v_tp;
  CHECK_EQ(mixed_qkvz.size(1), expected_dim);
  CHECK_EQ(mixed_ba.size(0), batch);
  CHECK_EQ(mixed_ba.size(1), ba_dim);
```
**EN:** This section uses `fused_qkvzba_split_reshape_cat_contiguous_cpu`, `CHECK_DIM`, `CHECK_INPUT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fused_qkvzba_split_reshape_cat_contiguous_cpu`、`CHECK_DIM`、`CHECK_INPUT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 174-200: Runtime integration and dispatch
```cpp
  TORCH_CHECK(mixed_ba.scalar_type() == mixed_qkvz.scalar_type(), "mixed_ba and mixed_qkvz must share same dtype");
  at::Tensor mixed_qkv = at::empty({batch, qkv_dim}, mixed_qkvz.options());
  at::Tensor z = at::empty({batch, num_heads_v, head_v}, mixed_qkvz.options());
  at::Tensor b = at::empty({batch, num_heads_v}, mixed_ba.options());
  at::Tensor a = at::empty({batch, num_heads_v}, mixed_ba.options());
  int64_t qkvz_strideB = mixed_qkvz.size(1);
  int64_t qkv_strideB = mixed_qkv.size(1);
  int64_t ba_strideB = mixed_ba.size(1);
  AT_DISPATCH_REDUCED_FLOATING_TYPES(mixed_qkvz.scalar_type(), "fused_qkvzba_split_reshape_cat_contiguous_impl", [&] {
    fused_qkvzba_split_reshape_cat_contiguous_impl<scalar_t>(
        mixed_qkvz.data_ptr<scalar_t>(),
        mixed_ba.data_ptr<scalar_t>(),
        mixed_qkv.data_ptr<scalar_t>(),
        z.data_ptr<scalar_t>(),
        b.data_ptr<scalar_t>(),
        a.data_ptr<scalar_t>(),
        batch,
        k_tp,
        v_tp,
        num_heads_v,
        qkv_dim,
        qkv_strideB,
        qkvz_strideB,
        ba_strideB);
  });
  return std::make_tuple(mixed_qkv, z, b, a);
}
```
**EN:** This section uses `TORCH_CHECK`, `options`, `make_tuple` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`、`options`、`make_tuple`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `common.h`, `vec.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cpu / model / qwen3.cpp
