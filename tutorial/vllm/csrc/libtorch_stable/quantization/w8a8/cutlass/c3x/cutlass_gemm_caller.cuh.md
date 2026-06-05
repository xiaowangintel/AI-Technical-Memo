# cutlass_gemm_caller.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `./csrc/libtorch_stable/quantization/w8a8/cutlass/c3x/cutlass_gemm_caller.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides shared helpers that turn `torch::stable::Tensor` inputs into CUTLASS 3.x GEMM arguments and launch the selected kernel. / 提供共享辅助逻辑，把 `torch::stable::Tensor` 输入转换为 CUTLASS 3.x GEMM 参数并启动选定内核。

## Line-by-Line Analysis / 逐行分析
### Problem shape helper / 问题形状辅助函数
```cpp
static inline cute::Shape<int, int, int, int> get_problem_shape(
    torch::stable::Tensor const& a, torch::stable::Tensor const& b) {
  int32_t m = a.size(0), n = b.size(1), k = a.size(1);
  return {m, n, k, 1};
```
**EN:** This helper derives the `(M, N, K, L)` problem shape directly from tensor dimensions, which keeps later dispatch code independent from manual shape packing.
**CN:** 该辅助函数直接从张量维度推导 `(M, N, K, L)` 问题形状，使后续分发代码无需手工组装形状。

### Low-level kernel launcher / 底层内核启动器
```cpp
template <typename GemmKernel>
void cutlass_gemm_caller(
    torch::stable::Device device, cute::Shape<int, int, int, int> prob_shape,
    typename GemmKernel::MainloopArguments mainloop_args,
    typename GemmKernel::EpilogueArguments epilogue_args,
    typename GemmKernel::TileSchedulerArguments scheduler = {}) {
  cutlass::KernelHardwareInfo hw_info;
  typename GemmKernel::Arguments args{cutlass::gemm::GemmUniversalMode::kGemm,
                                      prob_shape,
                                      mainloop_args,
                                      epilogue_args,
                                      hw_info,
                                      scheduler};

  // Launch the CUTLASS GEMM kernel.
  using GemmOp = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
  GemmOp gemm_op;
  CUTLASS_CHECK(gemm_op.can_implement(args));

  size_t workspace_size = gemm_op.get_workspace_size(args);
  auto workspace =
      torch::stable::empty(workspace_size, torch::headeronly::ScalarType::Byte,
                           std::nullopt, device);

  auto stream = get_current_cuda_stream(device.index());

  cutlass::Status status = gemm_op.run(args, workspace.data_ptr(), stream);
  CUTLASS_CHECK(status);
}
```
**EN:** The first `cutlass_gemm_caller` overload packages CUTLASS universal GEMM arguments, checks whether the kernel can run, allocates workspace, fetches the current CUDA stream, and launches the kernel.
**CN:** 第一个 `cutlass_gemm_caller` 重载负责打包 CUTLASS 通用 GEMM 参数、检查内核是否可执行、分配工作区、获取当前 CUDA stream，并最终启动内核。

### Tensor adapter / 张量适配层
```cpp
template <typename Gemm, typename... EpilogueArgs>
void cutlass_gemm_caller(torch::stable::Tensor& out,
                         torch::stable::Tensor const& a,
                         torch::stable::Tensor const& b,
                         EpilogueArgs&&... epilogue_params) {
  using ElementAB = typename Gemm::ElementAB;
  using ElementC = typename Gemm::ElementC;
  using ElementD = typename Gemm::ElementD;
  using GemmKernel = typename Gemm::GemmKernel;

  using StrideA = typename Gemm::GemmKernel::StrideA;
  using StrideB = typename Gemm::GemmKernel::StrideB;
  using StrideC = typename Gemm::GemmKernel::StrideC;
  using StrideD = StrideC;
  using StrideAux = StrideC;

  typename GemmKernel::ProblemShape prob_shape = get_problem_shape(a, b);
  auto [M, N, K, L] = prob_shape;

  StrideA a_stride =
      cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(M, K, L));
  StrideB b_stride =
      cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(N, K, L));
  StrideC c_stride =
      cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(M, N, L));
  StrideD d_stride =
      cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(M, N, L));
  StrideAux aux_stride = d_stride;

  auto a_ptr = static_cast<ElementAB*>(a.data_ptr());
  auto b_ptr = static_cast<ElementAB*>(b.data_ptr());
  typename GemmKernel::MainloopArguments mainloop_args{a_ptr, a_stride, b_ptr,
                                                       b_stride};

  auto c_ptr = static_cast<ElementD*>(out.data_ptr());
  // auto d_ptr = static_cast<ElementC*>(out.data_ptr());
  typename GemmKernel::EpilogueArguments epilogue_args{
      Gemm::Epilogue::prepare_args(
          std::forward<EpilogueArgs>(epilogue_params)...),
      c_ptr, c_stride, c_ptr, d_stride};

  cutlass_gemm_caller<GemmKernel>(a.device(), prob_shape, mainloop_args,
                                  epilogue_args);
}
```
**EN:** The tensor-based overload computes packed strides, extracts raw pointers, prepares epilogue arguments through the GEMM type, and forwards everything to the low-level launcher.
**CN:** 基于张量的重载会计算紧凑 stride、提取原始指针、通过 GEMM 类型构造 epilogue 参数，然后把所有信息转交给底层启动器。

## Key Concepts / 关键概念
- **Layered launch flow / 分层启动流程**: Separates tensor-to-argument conversion from the actual CUTLASS launch path. / 将“张量到参数”的转换与真正的 CUTLASS 启动流程解耦。
- **Packed strides / 紧凑步幅**: Uses CUTLASS/CUTE packed stride helpers so layouts match kernel expectations. / 使用 CUTLASS/CUTE 的 packed stride 辅助函数，保证布局符合内核预期。

## Dependencies / 依赖关系
- **torch::stable::Tensor / torch::stable::Tensor**: Accepts stable tensor wrappers and reads shapes, dtypes, and raw pointers from them. / 接收 stable tensor 封装，并从中读取形状、数据类型和原始指针。
- **CUTLASS / CUTLASS**: Uses CUTLASS collective builders, kernel adapters, and architecture tags to assemble GEMM kernels. / 使用 CUTLASS 的 collective builder、kernel adapter 与架构标签来组装 GEMM 内核。
- **CUDA stream helper / CUDA stream 辅助函数**: Calls `get_current_cuda_stream` from `libtorch_stable/torch_utils.h` so kernels run on the active stream. / 调用 `libtorch_stable/torch_utils.h` 中的 `get_current_cuda_stream`，让内核运行在当前活动 stream 上。
