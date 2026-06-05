# ampere_gather_scatter_conv.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/59_ampere_gather_scatter_conv/ampere_gather_scatter_conv.cu`  
**Purpose / 用途**: This source demonstrates how one Ampere CUTLASS/CuTe convolution kernel can operate on both dense tensors and gather/scatter tensors by changing layouts rather than changing math code. The file is an end-to-end driver: it builds logical layouts, launches the SM80 convolution operator, optionally checks the dense path against a host reference, and benchmarks the dense and indexed variants. / 这个源文件展示了：同一个基于 Ampere CUTLASS/CuTe 的卷积内核，如何仅通过更换布局描述就同时支持稠密张量和 gather/scatter 张量，而无需改动数学计算代码。它是一个完整驱动程序：负责构造逻辑布局、启动 SM80 卷积算子、可选地对稠密路径做参考校验，并测试稠密与索引版本的性能。

---

## Line-by-Line Analysis / 逐行分析

### Block 1 (Lines 1-74)
```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
/*! \file
  \brief Example demonstrating CuTe and CUTLASS 3.x based Ampere convolution forward propagation kernel
      capable of operating on both affine and gather/scatter tensors.

  This example demonstartes a few super cool features of CUTLASS and CuTe. It shows off
  1. A dense conv 3D fprop kernel written as a single file ...
  2. ... that leverages off the shelf CUTLASS collectives to show how custom kernels can use collectives ...
  3. ... and uses the exact same templated kernel to also stamp out a gather/scatter 3D fprop conv ...
  4. ... while getting near peak performance of the Ampere class tensor core on Ampere and Ada GPUs ...
  5. ... by using static cute shapes and strides in case problem shapes are known at compile time.

  Full documentation for this example can be found within the README.md file in this directory.

  Example executions:
  ./59_ampere_gather_scatter_conv
  ./59_ampere_gather_scatter_conv --n=108
  ./59_ampere_gather_scatter_conv --n=4096 --i=1
  ./59_ampere_gather_scatter_conv --n=1080 --i=1000
  ./59_ampere_gather_scatter_conv --n=131072 --i=1000 --no-check
*/

#include <thrust/sequence.h>
#include <thrust/universal_vector.h>

#include "ampere_conv_kernel.h"
#include "gather_tensor.hpp"

#include "cutlass/util/command_line.h"

bool check_cuda_result(cudaError_t code, const char* file, int line) {
  if (code == cudaSuccess) {
    return true;
  }

  std::cerr << "CUDA error at  (" << file << "," << line << ")\n\t" << unsigned(code) << " -- " << cudaGetErrorString(code) << "\n";
  return false;
}

#define CHECK_CUDA(code) (check_cuda_result(code, __FILE__, __LINE__))

using namespace cute;
using example::IndexedGather;
using example::CustomStride;

```
**EN**: The prologue documents the example goals, then pulls in `ampere_conv_kernel.h`, `gather_tensor.hpp`, Thrust containers, and the CUTLASS command-line helper. `check_cuda_result()` plus `CHECK_CUDA` standardize runtime error handling, which is especially useful in a single-file demo that mixes tensor construction, kernel attribute changes, and multiple launches.
**CN**: 文件前言先说明示例目标，然后引入 `ampere_conv_kernel.h`、`gather_tensor.hpp`、Thrust 容器以及 CUTLASS 命令行辅助头。`check_cuda_result()` 与 `CHECK_CUDA` 统一了运行时错误处理方式，这对一个同时包含张量构造、kernel attribute 设置和多次 launch 的单文件示例尤其重要。

### Block 2 (Lines 75-83)
```cpp
template<class Operator, class FilterTensor, class ActivationTensor, class OutputTensor>
__global__
__launch_bounds__(Operator::MaxThreadsPerBlock, Operator::MinBlocksPerMultiprocessor)
void kernel_entrypoint(FilterTensor mFlt, ActivationTensor mAct, OutputTensor mOut) {
  extern __shared__ char smem_buf[];
  Operator op;
  op(mFlt, mAct, mOut, smem_buf);
}

```
**EN**: `kernel_entrypoint` is a tiny generic CUDA wrapper around an `Operator` functor. The launch bounds come from the operator type itself, shared memory is passed as a raw buffer, and the operator’s call operator performs the real convolution. This pattern lets the same entry point work for dense and gather/scatter tensors as long as the tensor layouts satisfy the operator’s interface.
**CN**: `kernel_entrypoint` 是一个非常薄的泛型 CUDA 包装器，用来调用 `Operator` 仿函数。launch bounds 直接来自算子类型，共享内存以原始缓冲区形式传入，真正的卷积逻辑在算子的 `operator()` 中完成。只要张量布局满足算子接口，这种模式就能让同一个入口同时服务于稠密和 gather/scatter 张量。

### Block 3 (Lines 84-175)
```cpp
int ampere_dense_conv_fprop(
    int num_images,
    float* activations,
    float* filter,
    float* output,
    float* output_ref,
    int num_iterations = 1,
    bool do_ref_check = true) {
  auto D = typename AmpereUnpredicatedFprop::D{};
  auto H = typename AmpereUnpredicatedFprop::H{};
  auto W = typename AmpereUnpredicatedFprop::W{};
  auto Z = typename AmpereUnpredicatedFprop::Z{};
  auto P = typename AmpereUnpredicatedFprop::P{};
  auto Q = typename AmpereUnpredicatedFprop::Q{};
  auto C = typename AmpereUnpredicatedFprop::C{};
  auto K = typename AmpereUnpredicatedFprop::K{};
  auto S = typename AmpereUnpredicatedFprop::S{};
  auto R = typename AmpereUnpredicatedFprop::R{};
  auto T = typename AmpereUnpredicatedFprop::T{};

  int N = num_images; // dynamic
  if (num_images % int(typename AmpereUnpredicatedFprop::Tiler_N{}) != 0) {
    printf("ERROR: Input image count must be evenly divisible by CTA tiler N.\n");
    return 1;
  }

  // Tensor Activation: (n,d,h,w,c)::(?,6,4,4,64):(6144,1536,384,64,1)
  auto activation_layout = make_layout(
    make_shape (make_shape (      N,     D,   H, W), make_shape ( C,   _1{},_1{},_1{})),
    make_stride(make_stride(D*H*W*C, H*W*C, W*C, C), make_stride(_1{}, _0{},_0{},_0{})));

  auto xformed_act_layout = make_layout(
    make_shape (make_shape(N, Z, P, Q),       make_shape ( C,       T,   R, S)),
    make_stride(stride<0>(activation_layout), make_stride(_1{}, H*W*C, W*C, C)));

  // Tensor Filter    : (k,c,s,r,t)::(128,3,3,3,64):(1728,576,192,64,1)
  auto filter_layout = AmpereUnpredicatedFprop::GmemLayoutFlt{};

  // Tensor Output    : (n,z,p,q,k)::(?,4,2,2,128):(2048,1024,512,128,1)
  auto output_layout = make_ordered_layout(
    make_shape( K,   make_shape( N,   Z,   P,   Q)),
    make_tuple(_0{}, make_tuple(_4{},_3{},_2{},_1{})));

  Tensor mActivation = make_tensor(make_gmem_ptr(activations), activation_layout);
  Tensor mXformedAct = make_tensor(make_gmem_ptr(activations), xformed_act_layout);
  Tensor mFilter     = make_tensor(make_gmem_ptr(filter), filter_layout);
  Tensor mOutput     = make_tensor(make_gmem_ptr(output), output_layout); // (K, (N,Z,P,Q))
  Tensor mOutputRef  = make_tensor(make_gmem_ptr(output_ref), output_layout);

  print("xformed act layout ((N,Z,P,Q), (C,T,R,S)) = "); print(xformed_act_layout); print("\n");

  cudaEvent_t start, stop;
  CHECK_CUDA(cudaEventCreate(&start));
  CHECK_CUDA(cudaEventCreate(&stop));

  constexpr size_t smem_size = sizeof(typename AmpereUnpredicatedFprop::SharedStorage);
  Tensor gOutput_mn = zipped_divide(mOutput, typename AmpereUnpredicatedFprop::TilerOut{}); // ((BLK_M, BLK_N), (m', n'))
  dim3 lauch_grid {static_cast<uint32_t>(size<1,1>(gOutput_mn)), static_cast<uint32_t>(size<1,0>(gOutput_mn)), 1};

  CHECK_CUDA(cudaFuncSetAttribute(
    kernel_entrypoint<AmpereUnpredicatedFprop, decltype(mFilter), decltype(mXformedAct), decltype(mOutput)>,
    cudaFuncAttributeMaxDynamicSharedMemorySize,
    smem_size));

  CHECK_CUDA(cudaEventRecord(start));
  for (int i = 0; i < num_iterations; ++i) {
    kernel_entrypoint<AmpereUnpredicatedFprop, decltype(mFilter), decltype(mXformedAct), decltype(mOutput)>
      <<<lauch_grid, AmpereUnpredicatedFprop::MaxThreadsPerBlock, smem_size>>>(
        mFilter, mXformedAct, mOutput);
  }
  CHECK_CUDA(cudaEventRecord(stop));
  CHECK_CUDA(cudaEventSynchronize(stop));

  float milliseconds = 0;
  cudaEventElapsedTime(&milliseconds, start, stop);
  milliseconds /= float(num_iterations);

  double tflop_count = (2 * double(size<0>(xformed_act_layout)) * double(size(filter_layout))) / double(1e12);
  double tflops = tflop_count / (double(milliseconds) / double(1e3));

  printf("Conv TFLOP count = %f\n", tflop_count);
  printf("Conv dense perf: %fms | TFLOP/s = %f\n", milliseconds, tflops);

  if (do_ref_check) {
    printf("Running host reference check ...\n");
    return fprop_reference(mFilter, mXformedAct, mOutput, mOutputRef);
  }
  else {
    return 0;
  }
}

```
**EN**: `ampere_dense_conv_fprop()` builds dense CuTe layouts for the activation, transformed activation, filter, and output tensors, then launches `AmpereUnpredicatedFprop`. The important Ampere-specific detail is that the operator type comes from `ampere_conv_kernel.h`, where an SM80 TF32 tensor-op mainloop, `cp.async` staging, and static tile shapes are defined. This function mostly turns runtime pointers into the exact logical tensor views expected by that operator.
**CN**: `ampere_dense_conv_fprop()` 为激活、变换后的激活、滤波器和输出构建稠密的 CuTe 布局，然后启动 `AmpereUnpredicatedFprop`。关键的 Ampere 细节在于：该算子来自 `ampere_conv_kernel.h`，其中已经定义了基于 SM80 TF32 Tensor Core、`cp.async` 流水和静态 tile 形状的主循环。这里的主要工作是把运行时指针包装成算子期望的逻辑张量视图。

### Block 4 (Lines 176-283)
```cpp
int ampere_gather_scatter_conv_fprop(
    int num_images,
    float* activations,
    uint32_t *gather_idx_buf,
    float* filter,
    float* output,
    uint32_t *scatter_idx_buf,
    int num_iterations = 1) {
  auto D = typename AmpereUnpredicatedFprop::D{};
  auto H = typename AmpereUnpredicatedFprop::H{};
  auto W = typename AmpereUnpredicatedFprop::W{};
  auto Z = typename AmpereUnpredicatedFprop::Z{};
  auto P = typename AmpereUnpredicatedFprop::P{};
  auto Q = typename AmpereUnpredicatedFprop::Q{};
  auto C = typename AmpereUnpredicatedFprop::C{};
  auto K = typename AmpereUnpredicatedFprop::K{};
  auto S = typename AmpereUnpredicatedFprop::S{};
  auto R = typename AmpereUnpredicatedFprop::R{};
  auto T = typename AmpereUnpredicatedFprop::T{};

  int N = num_images; // dynamic
  if (N % int(typename AmpereUnpredicatedFprop::Tiler_N{}) != 0) {
    printf("ERROR: Input image count must be evenly divisible by CTA tiler N. Got num_images = %d\n", N);
    return 1;
  }

  // Tensor Filter    : (k,c,s,r,t)::(128,3,3,3,64):(1728,576,192,64,1)
  auto filter_layout = AmpereUnpredicatedFprop::GmemLayoutFlt{};

  // Tensor Output    : (n,z,p,q,k)::(?,4,2,2,128):(2048,1024,512,128,1)
  auto output_layout = make_ordered_layout(
    make_shape( K,   make_shape( N,   Z,   P,   Q)),
    make_tuple(_0{}, make_tuple(_4{},_3{},_2{},_1{})));

  // Input gather layout
  // inner_layout(make_coord((nzpq), (csrt))) => (idx_buffer_idx, dense_c_idx)
  auto EG = E<0>{};  // Gather basis     (1,0) (idx_buffer_idx) 
  auto EC = E<1>{};  // Contiguous basis (0,1) (dense_offset)    
  auto xformed_act_logical_inner = make_layout(
    make_shape (make_shape (       N,      Z,    P,  Q), make_shape ( C,      T,    R,  S)),
    make_stride(make_stride(D*H*W*EG, H*W*EG, W*EG, EG), make_stride(EC, H*W*EG, W*EG, EG)));

  // outer_layout(make_coord(idx_buffer_idx, dense_c_idx)) => idx
  // IndexedGather obtains idx by applying (gmem_base_ptr + gather_idx_buf[idx_buffer_idx] + dense_offset)
  auto xformed_act_gather_outer = make_layout(
    make_shape(_1{},_1{}),
    make_stride(CustomStride{IndexedGather{gather_idx_buf}, C}, _1{}));

  // Compose the inner and outer layouts
  // gather_composed(make_coord((nzpq), (csrt))) => idx
  auto xformed_act_composed_layout = composition(
    xformed_act_gather_outer,
    make_arithmetic_tuple(_0{}, _0{}),
    xformed_act_logical_inner);

  // Output scatter layout
  auto out_basis_stride = make_stride(
    E<1>{},
    make_stride(Z*P*Q*E<0>{}, P*Q*E<0>{}, Q*E<0>{}, _1{}*E<0>{})); // -> (crd0, crd1)
  auto out_basis_layout = make_layout(shape(output_layout), out_basis_stride);
  auto out_scatter_layout = make_layout(
    make_shape(_1{},_1{}),
    make_stride(CustomStride{IndexedGather{scatter_idx_buf}, K}, _1{}));
  auto out_composed_layout = composition(
    out_scatter_layout,
    make_arithmetic_tuple(_0{},_0{}),
    out_basis_layout);

  Tensor mXformedActGather = make_tensor(make_gmem_ptr(activations), xformed_act_composed_layout);
  Tensor mFilter = make_tensor(make_gmem_ptr(filter), filter_layout);
  Tensor mOutputScatter = make_tensor(make_gmem_ptr(output), out_composed_layout);  // (K, (N,Z,P,Q))

  Tensor gOutput_mn = zipped_divide(mOutputScatter, typename AmpereUnpredicatedFprop::TilerOut{}); // ((BLK_M, BLK_N), (m', n'))
  dim3 lauch_grid {static_cast<uint32_t>(size<1,1>(gOutput_mn)), static_cast<uint32_t>(size<1,0>(gOutput_mn)), 1};
  constexpr size_t smem_size = sizeof(typename AmpereUnpredicatedFprop::SharedStorage);

  print("xforemed gather layout ((N,Z,P,Q), (C,T,R,S)) = "); print(xformed_act_composed_layout); print("\n");
  print("Output  scatter layout ( K,        (N,Z,P,Q)) = "); print(out_composed_layout);         print("\n");
  print("Filter layout          ( K,        (C,T,R,S)) = "); print(filter_layout);               print("\n");

  CHECK_CUDA(cudaFuncSetAttribute(
    kernel_entrypoint<AmpereUnpredicatedFprop, decltype(mFilter), decltype(mXformedActGather), decltype(mOutputScatter)>,
    cudaFuncAttributeMaxDynamicSharedMemorySize,
    smem_size));

  cudaEvent_t start, stop;
  CHECK_CUDA(cudaEventCreate(&start));
  CHECK_CUDA(cudaEventCreate(&stop));
  CHECK_CUDA(cudaEventRecord(start));
  for (int i = 0; i < num_iterations; ++i) {
    kernel_entrypoint<AmpereUnpredicatedFprop, decltype(mFilter), decltype(mXformedActGather), decltype(mOutputScatter)>
      <<<lauch_grid, AmpereUnpredicatedFprop::MaxThreadsPerBlock, smem_size>>>(
          mFilter, mXformedActGather, mOutputScatter);
  }
  CHECK_CUDA(cudaEventRecord(stop));
  CHECK_CUDA(cudaEventSynchronize(stop));
  float milliseconds = 0;
  cudaEventElapsedTime(&milliseconds, start, stop);
  milliseconds /= float(num_iterations);

  double tflop_count = (2 * double(size<0>(xformed_act_logical_inner)) * double(size(filter_layout))) / double(1e12);
  double tflops = tflop_count / (double(milliseconds) / double(1e3));
  printf("Conv TFLOP count = %f\n", tflop_count);
  printf("Conv gather/scatter perf: %fms | TFLOP/s = %f\n", milliseconds, tflops);

  return 0;
}

```
**EN**: `ampere_gather_scatter_conv_fprop()` reuses the same filter/output tiling logic but changes the activation and output layouts into composed indirection layouts. `IndexedGather` and `CustomStride` turn index buffers into layout objects, `composition()` fuses logical coordinates with those indirections, and the same `AmpereUnpredicatedFprop` kernel now reads activations via gather and writes outputs via scatter. This is the core idea of the example: tensor indirection lives in layout algebra, not in a rewritten kernel.
**CN**: `ampere_gather_scatter_conv_fprop()` 复用了相同的 filter/output 分块逻辑，但把激活和输出布局改造成带间接寻址的组合布局。`IndexedGather` 与 `CustomStride` 把索引缓冲区包装成布局对象，`composition()` 再把逻辑坐标和这些间接关系融合起来，于是同一个 `AmpereUnpredicatedFprop` 内核就能通过 gather 读取激活，并通过 scatter 写回输出。这正是本示例的核心思想：张量间接寻址放在布局代数中表达，而不是重写内核。

### Block 5 (Lines 284-392)
```cpp
int
main(int argc, char const** argv) {
  cutlass::CommandLine cmd(argc, argv);
  std::cout << "Ampere convolution forward propagation kernel supporting both affine and gather/scatter tensors.\n\n";
  if (cmd.check_cmd_line_flag("help")) {
    std::cout
      << "Options:\n"
         "\t--n=<int>    Sets the number of images for the input activation tensor (dataset size). Default = 131072.\n"
         "\t--i=<int>    Sets the benchmarking repetitions. Default = 128.\n"
         "\t--nocheck    If specified, skips the reference check for dense kernel.\n"
         "\t--help       Displays this help message and exits.\n";
    return 0;
  }


  cudaDeviceProp props;
  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
    return -1;
  }
  if (props.major < 8) {
    std::cerr << "This example requires an Ampere GPU or newer.\n";
    return 0;
  }

  int num_images = 4320;
  cmd.get_cmd_line_argument("n", num_images, 4320);
  int num_iterations = 128;
  cmd.get_cmd_line_argument("i", num_iterations, 128);
  bool do_host_ref_check = not cmd.check_cmd_line_flag("no-check");

  auto D = typename AmpereUnpredicatedFprop::D{};
  auto H = typename AmpereUnpredicatedFprop::H{};
  auto W = typename AmpereUnpredicatedFprop::W{};
  auto Z = typename AmpereUnpredicatedFprop::Z{};
  auto P = typename AmpereUnpredicatedFprop::P{};
  auto Q = typename AmpereUnpredicatedFprop::Q{};
  auto C = typename AmpereUnpredicatedFprop::C{};
  auto K = typename AmpereUnpredicatedFprop::K{};

  auto activation_layout = make_layout(
    make_shape (make_shape (num_images,     D,   H, W), make_shape ( C,   _1{},_1{},_1{})),
    make_stride(make_stride(   D*H*W*C, H*W*C, W*C, C), make_stride(_1{}, _0{},_0{},_0{})));

  auto filter_layout = typename AmpereUnpredicatedFprop::GmemLayoutFlt{};

  auto output_layout = make_ordered_layout(
    make_shape( K,   make_shape(num_images,   Z,   P,   Q)),
    make_step (_0{}, make_step (      _4{},_3{},_2{},_1{})));

  print("Filter layout     ( K,        (C,T,R,S)) = "); print(filter_layout);     print("\n");
  print("Activation layout ((N,D,H,W), (C,1,1,1)) = "); print(activation_layout); print("\n");
  print("Output layout     ( K,        (N,Z,P,Q)) = "); print(output_layout);     print("\n");

  // allocate tensors
  std::cout << "Allocating tensors ... ";
  thrust::universal_vector<float> activation_data(size_t(cute::size(activation_layout)), float(0));
  thrust::universal_vector<float> filter_data(size_t(cute::size(filter_layout)), float(0));
  thrust::universal_vector<float> output_data(size_t(cute::size(output_layout)), float(0));
  thrust::universal_vector<float> output_data_ref(size_t(cute::size(output_layout)), float(0));
  std::cout << "done.\n";

  // init tensors
  std::cout << "Initializing data ... " << std::flush;
  std::random_device rd;
  std::mt19937 gen(rd());
  std::uniform_real_distribution<float> uniform_dist(-1.0, 1.0);
  for (std::size_t i = 0; i < size_t(cute::size(activation_layout)); ++i) {
    activation_data[i] = uniform_dist(gen);
  }

  for (std::size_t i = 0; i < size_t(cute::size(filter_layout)); ++i) {
    filter_data[i] = uniform_dist(gen);
  }
  std::cout << "done.\n";

  // set up index buffers for gather/scatter, fill with indireciton indices in reversed order
  std::cout << "Initializing gather/scatter index buffers ... ";
  thrust::universal_vector<uint32_t> gather_idx_buf(size_t(size<0>(activation_layout)));
  thrust::universal_vector<uint32_t> scatter_idx_buf(size_t(size<1>(output_layout)));
  thrust::sequence(gather_idx_buf.rbegin(), gather_idx_buf.rend());
  thrust::sequence(scatter_idx_buf.rbegin(), scatter_idx_buf.rend());
  std::cout << "done.\n";

  // launch dense
  std::cout << "\nRunning dense fprop kernel\n";
  int passed = ampere_dense_conv_fprop(
    num_images,
    activation_data.data().get(),
    filter_data.data().get(),
    output_data.data().get(),
    output_data_ref.data().get(),
    num_iterations,
    do_host_ref_check);

  // launch gather/scatter
  std::cout << "\nRunning gather/scatter fprop kernel\n";
  ampere_gather_scatter_conv_fprop(
    num_images,
    activation_data.data().get(),
    gather_idx_buf.data().get(),
    filter_data.data().get(),
    output_data.data().get(),
    scatter_idx_buf.data().get(),
    num_iterations);

  return passed;
}
```
**EN**: `main()` parses user options, checks for Ampere-or-newer hardware, prints the base tensor layouts, allocates unified-memory vectors, initializes random data, builds reversed gather/scatter index buffers, and then runs both the dense and indexed convolutions. The program therefore acts as a practical harness for comparing the correctness and performance impact of affine versus indexed tensor views on the same underlying operator.
**CN**: `main()` 负责解析参数、检查硬件是否至少为 Ampere、打印基础张量布局、分配统一内存向量、初始化随机数据、构造反向的 gather/scatter 索引缓冲区，然后依次运行稠密卷积和索引卷积。因此，这个程序本质上是一个对照实验框架，用来比较同一算子在仿射布局和索引布局下的正确性与性能差异。

---

## Key Concepts / 关键概念

**EN**:
- The heavy lifting is in `AmpereUnpredicatedFprop` from `ampere_conv_kernel.h`: it uses an SM80 TF32 MMA atom, `cp.async`-based staging, and static CuTe shapes tuned for Ampere-class tensor cores.
- Gather/scatter support is achieved by changing layouts, not the convolution math. This is a strong CuTe pattern: encode address computation in the tensor view so the kernel body stays generic.
- The dense and indexed paths share the same kernel entry point, which makes the example a good demonstration of layout polymorphism.
- Unlike Hopper TMA examples, this file is Ampere-specific: its performance model relies on SM80 tensor-op instructions and async-copy pipelines rather than TMA or warpgroup MMA.

**CN**:
- 真正的计算核心在 `ampere_conv_kernel.h` 中的 `AmpereUnpredicatedFprop`：它使用 SM80 TF32 MMA atom、基于 `cp.async` 的流水，以及为 Ampere Tensor Core 调优的静态 CuTe 形状。
- gather/scatter 支持是通过改变布局而非改变卷积数学实现的，这是典型的 CuTe 思路：把地址计算编码进张量视图，让内核主体保持通用。
- 稠密路径和索引路径共享同一个 kernel entry point，因此这个示例很好地展示了布局多态性。
- 与 Hopper 的 TMA 示例不同，这个文件是 Ampere 风格：它依赖 SM80 Tensor Core 指令与 async-copy 流水，而不是 TMA 或 warpgroup MMA。

## Dependencies / 依赖项

**EN**:
- `ampere_conv_kernel.h` defines the `AmpereUnpredicatedFprop` operator, static problem shape, tiling, shared-memory layout, and the actual SM80 mainloop/epilogue logic.
- `gather_tensor.hpp` defines `IndexedGather` and `CustomStride`, which allow index buffers to participate in CuTe layout composition.
- Thrust unified vectors simplify allocation of activation/filter/output data and index buffers without separate host/device management.
- `cutlass/util/command_line.h` provides the simple benchmark-oriented CLI parser.

**CN**:
- `ampere_conv_kernel.h` 定义了 `AmpereUnpredicatedFprop` 算子、静态问题规模、分块方式、共享内存布局，以及真正的 SM80 主循环/epilogue 逻辑。
- `gather_tensor.hpp` 定义了 `IndexedGather` 与 `CustomStride`，使索引缓冲区可以参与 CuTe 布局组合。
- Thrust 的统一内存向量简化了激活、滤波器、输出和索引缓冲区的分配，不需要分别管理主机/设备副本。
- `cutlass/util/command_line.h` 提供了面向基准测试的轻量 CLI 解析器。
