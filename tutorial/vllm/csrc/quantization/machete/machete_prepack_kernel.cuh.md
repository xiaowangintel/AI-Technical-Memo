# machete_prepack_kernel.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/machete/machete_prepack_kernel.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements the CUDA kernel and host helper that rearrange matrix `B` into Machete's prepacked interleaved layout before GEMM. / [CN] 实现 CUDA 内核与主机辅助函数，在 GEMM 前把矩阵 `B` 重排为 Machete 所需的预打包交错布局。

## Line-by-Line Analysis / 逐行分析
### Includes and kernel template / 头文件与内核模板
```cpp
template <int threads, typename PrepackedLayoutB, typename BInTensor,
          typename ElementB>
static __global__ void prepack_B_kernel(BInTensor B_in, ElementB* B_out_ptr) {
  auto constexpr block_size =
      Int<size(typename PrepackedLayoutB::PPBlockShape_NK{})>{};
  auto constexpr eles_per_thread = Int<block_size / threads>{};
```
**EN:** The kernel is fully parameterized by the destination layout. `PPBlockShape_NK` defines how many logical `N x K` elements belong to one prepacked block, and the compile-time `eles_per_thread` splits that block evenly across threads.
**CN:** 这个内核完全由目标布局参数化。`PPBlockShape_NK` 定义一个预打包块包含多少逻辑上的 `N x K` 元素，而编译期常量 `eles_per_thread` 负责把该块均匀分给各线程。

### Selecting the source tile / 选择输入子块
```cpp
auto blk_coord = make_coord(blockIdx.x, blockIdx.y, blockIdx.z);
auto tB_in = local_tile(
    B_in, append(typename PrepackedLayoutB::PPBlockShape_NK{}, _1{}),
    blk_coord);
```
**EN:** Each thread block is responsible for one `(N_tile, K_tile, L)` slice of the original tensor. `local_tile` extracts exactly the tile that must be repacked.
**CN:** 每个线程块负责原始张量中的一个 `(N_tile, K_tile, L)` 切片。`local_tile` 精确取出当前需要重排的子块。

### Mapping the tile into prepacked linear storage / 映射到预打包线性存储
```cpp
auto bNbKL_to_offset = PrepackedLayoutB::bNbKL_to_offset(shape(B_in));
auto tB_out_linear =
    make_tensor(get_logical_ptr(B_out_ptr) + bNbKL_to_offset(blk_coord),
                make_layout(make_shape(block_size)));
auto tB_in_linear = make_tensor(
    tB_in.data(),
    tB_in.layout()
        .compose(right_inverse(PrepackedLayoutB::ppblock_ilvd_NK_to_offset()))
        .with_shape(make_shape(block_size)));
```
**EN:** The destination is treated as a 1D block in the exact physical order required by Machete. The source tile is also flattened, but through the inverse interleaving map, so reading the source linearly yields values in the final packed order.
**CN:** 目标输出被视为一个一维块，其顺序正是 Machete 需要的物理存储顺序。输入子块也被展平成一维，但先经过交错映射的逆变换，因此按线性顺序读取输入就会得到最终打包顺序。

### Thread-local copy through registers / 通过寄存器执行线程局部拷贝
```cpp
auto thr_tB_in_linear =
    local_tile(tB_in_linear, make_shape(eles_per_thread), threadIdx.x);
auto thr_tB_out_linear =
    local_tile(tB_out_linear, make_shape(eles_per_thread), threadIdx.x);
auto fragment = make_tensor<ElementB>(shape(thr_tB_in_linear));

copy(thr_tB_in_linear, fragment);
copy(Copy_Atom<DefaultCopy, uint8_t>{}, fragment, thr_tB_out_linear);
```
**EN:** Every thread copies a contiguous slice of the logical block into a register-backed fragment and then stores it to the linear output tile. The kernel favors simplicity over a highly specialized tiled-copy primitive.
**CN:** 每个线程先把逻辑块中的一段连续数据读入寄存器片段，再写入线性输出块。这里优先选择实现简单，而不是更复杂的专用 tiled-copy 原语。

### Host launcher and grid construction / 主机侧启动逻辑与网格构造
```cpp
template <typename PrepackedLayoutB, typename InLayout>
static void prepack_B_template(
    cudaStream_t stream, typename PrepackedLayoutB::ElementB const* B_in_ptr,
    InLayout B_layout, typename PrepackedLayoutB::ElementB* B_out_ptr) {
  using TileShapeNKL =
      decltype(append(typename PrepackedLayoutB::PPBlockShape_NK{}, _1{}));
  ...
  prepack_B_kernel<128, PrepackedLayoutB>
      <<<dim3(N_tiles, K_tiles, L_tiles), 128, 0, stream>>>(B_in, B_out_ptr);
}
```
**EN:** The host helper validates that the input layout is divisible by the packing tile, computes the tile grid in `N`, `K`, and `L`, wraps the raw pointer as a Cute tensor, and launches the kernel with 128 threads per block.
**CN:** 主机辅助函数先检查输入布局能否被打包块整除，再计算 `N`、`K`、`L` 三个维度上的 tile 数量，把裸指针包装成 Cute tensor，并以每块 128 线程启动内核。

## Key Concepts / 关键概念
- **Prepacked block layout**: turns a logical `B` tile into the exact memory order expected by the later Machete GEMM kernel. / **预打包块布局**：把逻辑上的 `B` 子块变成后续 Machete GEMM 内核所需的内存顺序。
- **Inverse interleaving map**: `right_inverse(...)` is the core trick that converts from logical coordinates to packed offsets. / **逆交错映射**：`right_inverse(...)` 是把逻辑坐标转换到打包偏移的关键。
- **Cute tensor algebra**: the implementation relies on Cute layouts and tensor views instead of manual index arithmetic. / **Cute 张量代数**：实现依赖 Cute 的布局与张量视图，而非手写索引运算。

## Dependencies / 依赖关系
- `machete_mm_kernel.cuh` is not included here, but the produced layout is consumed by Machete matrix-multiply kernels. / 这里虽然未直接包含 `machete_mm_kernel.cuh`，但生成的布局会被 Machete 矩阵乘内核消费。
- `cutlass_extensions/cute_utils.cuh` and Cute primitives provide tiling, layouts, and pointer wrappers. / `cutlass_extensions/cute_utils.cuh` 与 Cute 原语提供切块、布局和指针封装。
- `cutlass_extensions/torch_utils.hpp` supplies `TORCH_CHECK` integration used by the launcher. / `cutlass_extensions/torch_utils.hpp` 提供启动器中使用的 `TORCH_CHECK` 集成。
