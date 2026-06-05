# machete_prepack_launcher.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/machete/machete_prepack_launcher.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Validates a PyTorch weight tensor, reconstructs the logical `B` layout expected by Machete, and launches the prepack kernel. / [CN] 对 PyTorch 权重张量进行校验，重建 Machete 期望的逻辑 `B` 布局，并启动预打包内核。

## Line-by-Line Analysis / 逐行分析
### Input descriptor / 输入描述结构
```cpp
struct PrepackBArgs {
  torch::Tensor const& B;
  at::ScalarType a_type;
  vllm::ScalarType b_type;
  std::optional<at::ScalarType> maybe_group_scales_type;
};
```
**EN:** `PrepackBArgs` records not only the tensor itself, but also the activation and weight dtypes that will later determine which prepacked layout is compatible.
**CN:** `PrepackBArgs` 不仅保存张量本身，还记录激活与权重的数据类型；这些信息会决定后续应该选择哪一种预打包布局。

### Device/stream setup and packed-element width / 设备、流与打包元素宽度
```cpp
template <typename PrepackedLayoutB>
torch::Tensor prepack_impl(torch::Tensor const B) {
  const at::cuda::OptionalCUDAGuard device_guard(device_of(B));
  using ElementB = typename PrepackedLayoutB::ElementB;
  ...
  auto B_ptr = static_cast<ElementB const*>(B.const_data_ptr());
  auto eles_per_storage =
      (B.dtype().itemsize() * 8) / cute::sizeof_bits_v<ElementB>;
```
**EN:** The launcher binds to `B`'s CUDA device, then computes how many logical quantized elements are packed into one storage element. This matters because packed `B` uses fewer bits per logical value than its physical storage type.
**CN:** 启动器先绑定到 `B` 所在的 CUDA 设备，再计算一个存储单元里打包了多少个逻辑量化元素。之所以要这样做，是因为打包后的 `B` 在逻辑值层面往往比其物理存储类型使用更少的位数。

### Transposing to match CUTLASS `B` convention / 转置以匹配 CUTLASS 的 `B` 约定
```cpp
// torch B passed in is/should be (packed_K,N), the kernel expects (N,K,L)
auto Bt_packed = B.t();

TORCH_CHECK(
    (B.size(0) * eles_per_storage) % size<1>(PPBlockShape_NK{}) == 0,
    ...);
TORCH_CHECK(B.size(1) % size<0>(PPBlockShape_NK{}) == 0, ...);
```
**EN:** PyTorch provides `B` in `(packed_K, N)` form, but the downstream CUTLASS-based kernel reasons about `B` as `(N, K, L)`. The transpose and divisibility checks ensure the packed tensor can be interpreted as whole prepack tiles.
**CN:** PyTorch 传入的 `B` 是 `(packed_K, N)` 形式，但下游基于 CUTLASS 的内核按 `(N, K, L)` 来理解 `B`。转置和整除检查保证这个打包张量可以被解释为完整的预打包 tile。

### Rebuilding the unpacked logical layout / 重建解包后的逻辑布局
```cpp
auto const l_Bt_packed = make_cute_layout<StrideB>(Bt_packed, "B");
...
auto const layout_Bt = make_layout(
    transform_with_idx(l_Bt_packed.shape(), [&](auto ele, auto idx) {
      return idx == 1 ? ele * eles_per_storage : ele;
    }),
    transform_with_idx(l_Bt_packed.stride(), [&](auto ele, auto idx) {
      return idx != 1 ? ele * eles_per_storage : ele;
    }));
```
**EN:** The interesting part is that the code does not physically unpack `B`. Instead, it builds a new Cute layout whose shape and strides reinterpret the packed storage as if it were the full `(N, K, L)` tensor.
**CN:** 这里最关键的一点是：代码并没有真正把 `B` 物理解包，而是构造了一个新的 Cute 布局，通过修改 shape 和 stride，把打包存储“重新解释”为完整的 `(N, K, L)` 逻辑张量。

### Output allocation and kernel launch / 输出分配与内核启动
```cpp
torch::Tensor D = torch::empty_like(B, {}, at::MemoryFormat::Contiguous);

prepack_B_template<PrepackedLayoutB>(
    stream, B_ptr, layout_Bt, static_cast<ElementB*>(D.mutable_data_ptr()));
```
**EN:** The output tensor keeps the same storage size as the input packed tensor, but its element order is rewritten into Machete's preferred prepacked format.
**CN:** 输出张量与输入打包张量拥有相同的存储大小，但其元素顺序会被改写为 Machete 偏好的预打包格式。

### External dispatcher / 外部分发器
```cpp
torch::Tensor prepack_B_dispatch(PrepackBArgs args);
```
**EN:** Runtime type-based selection is delegated to `prepack_B_dispatch`, while this header keeps the reusable typed implementation.
**CN:** 基于运行时类型的选择逻辑交给 `prepack_B_dispatch`，本头文件则保留可复用的类型化实现。

## Key Concepts / 关键概念
- **Layout reinterpretation instead of unpacking**: the code changes shape/stride metadata rather than expanding bits into a new dense tensor. / **重解释布局而不是解包数据**：代码通过修改 shape/stride 元数据完成转换，而不是把位级数据展开成新的稠密张量。
- **Prepack tile divisibility**: shape checks guarantee every block is full and can be copied with a fixed mapping. / **预打包块整除性**：形状检查保证每个块都是完整块，从而可以用固定映射拷贝。
- **Separation of concerns**: runtime dispatch and typed implementation are separated. / **职责分离**：运行时分发与类型化实现分开。

## Dependencies / 依赖关系
- `machete_prepack_kernel.cuh` provides `prepack_B_template`, the actual launch target. / `machete_prepack_kernel.cuh` 提供实际启动目标 `prepack_B_template`。
- `cutlass_extensions/torch_utils.hpp` supplies `device_of` and tensor-to-layout helpers. / `cutlass_extensions/torch_utils.hpp` 提供 `device_of` 以及张量到布局的辅助工具。
- `core/scalar_type.hpp` is used by the dispatcher to match vLLM scalar types with torch dtypes. / `core/scalar_type.hpp` 会被分发器用来匹配 vLLM 标量类型与 torch dtype。
