# fused_multi_head_attention_backward.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/41_fused_multi_head_attention/fused_multi_head_attention_backward.cu`
**Purpose / 用途**: Backward FMHA example that instantiates the fused backward kernel, prepares tensors/workspace, validates gradients, and benchmarks launches / 反向 FMHA 示例，负责实例化融合反向内核、准备张量与工作区、校验梯度并进行性能测试
---
## Line-by-Line Analysis / 逐行分析
```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

/////////////////////////////////////////////////////////////////////////////////////////////////

#include <vector>
#include <iostream>
#include <fstream>

#include "kernel_backward.h"

#include "cutlass/util/device_memory.h"
#include "cutlass/util/host_tensor.h"


using Arch = cutlass::arch::Sm80;
static constexpr int kMaxK = 128;

template <typename ArchTag, typename Element, int kMaxK>
struct DefaultKernel {
    // Some heuristics to select the best kernel (tested on Sm60, Sm70, Sm80)
    // NOTE: Requires quite a lot of shmem for Sm80+,
    // so might require tweaking those manually for Sm86/Sm89

    static constexpr bool kSupports64x128 =
        ArchTag::kMinComputeCapability >= 80 ||
        (ArchTag::kMinComputeCapability >= 70 &&
        cutlass::sizeof_bits<Element>::value <= 16);
    static constexpr int kBlockSizeI = kSupports64x128 && kMaxK > 64 ? 128 : 64;
    static constexpr bool kIsHalf = cutlass::sizeof_bits<Element>::value <= 16;
    static constexpr bool kOutputInRF = kIsHalf && kMaxK <= kBlockSizeI;
    static constexpr bool kPreload = kIsHalf && ArchTag::kMinComputeCapability >= 80 && kOutputInRF;
    static constexpr int kBlockSizeJ = kPreload && kMaxK > 64 ? 128 : 64;

    using Kernel = AttentionBackwardKernel<
        Arch,
        Element,
        true,        // kIsAligned_
        false,       // kApplyDropout_
        kPreload,    // kPreload_
        kBlockSizeI, // kBlockSizeI_,
        kBlockSizeJ, // kBlockSizeJ_,
        kMaxK,       // kMaxK
        false,       // kKeysQueriesAlignedToBlockSize
        true         // kEnableSplitKeys
    >;
};
```
**EN**: Lines 1-75. This file is a compact backward-pass driver around `AttentionBackwardKernel` from `kernel_backward.h`. `DefaultKernel` hard-wires the example to `Sm80`, then uses compile-time heuristics to choose `kBlockSizeI`, `kBlockSizeJ`, register-file output, preload behavior, and split-key support from the architecture capability, element width, and `kMaxK`. That block is the main FMHA launch/config policy for the backward example.
**CN**: 第1-75行：该文件本质上是 `kernel_backward.h` 中 `AttentionBackwardKernel` 的一个精简反向执行驱动。`DefaultKernel` 将示例固定到 `Sm80`，并根据架构能力、元素位宽以及 `kMaxK` 在编译期选择 `kBlockSizeI`、`kBlockSizeJ`、是否将输出累加保存在寄存器、是否预取以及是否启用 split-key。这一段就是反向示例最核心的 FMHA 启动/配置策略。
```cpp

/////////////////////////////////////////////////////////////////////////////////////////////////

namespace {
template <typename T> struct TypeName;
template <> struct TypeName<float> { static constexpr const char* Name = "f32"; };
template <> struct TypeName<cutlass::half_t> { static constexpr const char* Name = "f16"; };
template <> struct TypeName<cutlass::bfloat16_t> { static constexpr const char* Name = "b16"; };

void readExpect(std::string const& expected) {
    std::string read;
    std::cin >> read;
    if (read != expected) {
        std::cerr << "FATAL: Read '" << read << "' but expected '" << expected << "'" << std::endl;
        std::exit(1);
    }
}

/// Helpers to read from stdin
template <typename Element>
cutlass::HostTensor<Element, cutlass::layout::RowMajor> readTensorOnDevice(std::string const& expectedName) {
    readExpect("tensor_begin");
    readExpect(std::string(TypeName<Element>::Name) + ":" + expectedName);
    uint64_t len = 0;
    std::cin >> len;
    readExpect("file");
    std::string filename;
    std::cin >> filename;

    cutlass::HostTensor<Element, cutlass::layout::RowMajor> tensor({int64_t(1), int64_t(len / sizeof(Element))});
    uint8_t* data = (uint8_t*)tensor.host_data();

    std::fstream myFile(filename, std::ios::in | std::ios::binary );
    myFile.read((char*)data, len);
    readExpect("tensor_end");
    tensor.sync_device();
    return tensor;
}

int64_t readInt64(std::string const& expectedName) {
    readExpect(expectedName);
    int64_t s = 0;
    std::cin >> s;
    return s;
}

float readFloat(std::string const& expectedName) {
    readExpect(expectedName);
    float s = 0;
    std::cin >> s;
    return s;
}

// Writing
template <typename Element>
void writeTensor(std::string const& name, cutlass::HostTensor<Element, cutlass::layout::RowMajor>& tensor) {
    tensor.sync_host(); // device->host
    size_t u8len = tensor.size() * sizeof(Element);

    // Python is expected to provide a file name to write to
    readExpect("tmpfile");
    std::string tmpfile;
    std::cin >> tmpfile;

    uint8_t* data = (uint8_t*)tensor.host_data();
    std::fstream myFile(tmpfile, std::ios::out | std::ios::binary );
    myFile.write((char*)data, u8len);
    myFile.close();

    std::cout << "tensor_begin " << TypeName<Element>::Name << ":" << name << " ";
    std::cout << u8len << " file " << tmpfile << " tensor_end" << std::endl;
}

void writeInt64(std::string const& name, int64_t value) {
    std::cout << name << " " << value << std::endl;
}
}
```
**EN**: Lines 76-152. The anonymous namespace builds a tiny stdin/stdout protocol layer. `TypeName` maps C++ element types to textual tokens, `readExpect` validates the serialized stream, `readTensorOnDevice` loads raw tensor bytes from a file into a `HostTensor` and syncs them to device memory, and the write helpers mirror that flow for outputs. The backward kernel is therefore wired as a data-exchange endpoint rather than a standalone benchmark binary.
**CN**: 第76-152行：匿名命名空间实现了一层很小的标准输入/输出协议。`TypeName` 把 C++ 元素类型映射为文本标记，`readExpect` 用于校验序列化输入流，`readTensorOnDevice` 从文件中读取原始张量字节到 `HostTensor` 后再同步到设备内存，而写出辅助函数则执行相反流程。因此这个反向程序更像数据交换端点，而不是独立 benchmark。
```cpp

/////////////////////////////////////////////////////////////////////////////////////////////////

template <typename Element>
int runKernel() {
    using Kernel = typename DefaultKernel<Arch, Element, kMaxK>::Kernel;

#define READ_I64(NAME) p.NAME = (decltype(p.NAME))readInt64(#NAME)
#define READ_TENSOR_AND_STRIDES_BMH(DT, NAME, NAME_XS) \
    auto storage##NAME = readTensorOnDevice<DT>(#NAME); \
    p.NAME##_ptr = storage##NAME.device_data(); \
    READ_I64(NAME_XS##_strideB); \
    READ_I64(NAME_XS##_strideM); \
    READ_I64(NAME_XS##_strideH);

#define CUDA_CHECK(FN) { \
    auto cudaError = FN; \
    if (cudaError != cudaSuccess) { \
        std::cerr << "FATAL: " #FN " failed: " << cudaGetErrorString(cudaError) << std::endl; \
        return -1; \
    } \
}

    typename Kernel::Params p;
    p.scale = readFloat("scale");
    READ_I64(head_dim);
    READ_I64(head_dim_value);
    READ_I64(num_queries);
    READ_I64(num_keys);
    READ_I64(num_heads);
    READ_I64(custom_mask_type);
    READ_I64(num_batches);
    int64_t repeat_count = readInt64("repeat_count");
    READ_I64(num_splits_key);

    READ_TENSOR_AND_STRIDES_BMH(Element, query, q);
    READ_TENSOR_AND_STRIDES_BMH(Element, key, k);
    READ_TENSOR_AND_STRIDES_BMH(Element, value, v);
    auto lse = readTensorOnDevice<typename Kernel::lse_scalar_t>("logsumexp");
    p.logsumexp_ptr = lse.device_data();
    p.lse_strideB = readInt64("lse_strideB");
    p.lse_strideH = readInt64("lse_strideH");

    // output
    auto stOutput = readTensorOnDevice<Element>("output");
    p.output_ptr = stOutput.device_data();
    READ_I64(o_strideB);
    auto o_strideM = readInt64("o_strideM");
    if (o_strideM != p.o_strideM()) {
        std::cerr << "Invalid `o_strideM`: " << o_strideM << " - expected " << p.o_strideM();
        return 2;
    }
    READ_I64(o_strideH);

    READ_TENSOR_AND_STRIDES_BMH(Element, grad_output, gO);

    auto stDelta = readTensorOnDevice<typename Kernel::accum_t>("delta");
    p.delta_ptr = stDelta.device_data();
    READ_I64(delta_strideB);
    READ_I64(delta_strideH);

    // Allocate workspace
```
**EN**: Lines 153-214. `runKernel<Element>()` begins by selecting the concrete backward kernel type and defining macros that read scalar fields plus BM(H) tensor strides. It then fills `Kernel::Params` from the external stream: scale, head sizes, query/key counts, head and batch counts, mask mode, split-key count, Q/K/V pointers, `logsumexp`, the forward output, `grad_output`, and `delta`. This block is the main argument-wiring path from host-side serialized inputs into the template kernel parameter object.
**CN**: 第153-214行：`runKernel<Element>()` 首先选定具体的反向 kernel 类型，并定义用于读取标量字段与 BM(H) 张量步长的宏。随后它从外部输入流中填充 `Kernel::Params`：包括缩放系数、head 维度、query/key 数量、head 数、batch 数、mask 类型、split-key 数、Q/K/V 指针、`logsumexp`、前向输出、`grad_output` 以及 `delta`。这一段就是把宿主侧序列化参数接入模板 kernel 参数对象的主路径。
```cpp
    if (p.workspace_size()) {
        cudaMalloc(&p.workspace, p.workspace_size());
    }

    // Allocate outputs in BMHK format
    p.gQKV_strideM_multiplier = 1;
    p.gQ_strideH = p.head_dim;
    p.gQ_strideB = p.gQ_strideM() * p.num_queries;
    p.gK_strideH = p.head_dim;
    p.gK_strideB = p.gK_strideM() * p.num_keys;
    p.gV_strideH = p.head_dim_value;
    p.gV_strideB = p.gV_strideM() * p.num_keys;

    cutlass::HostTensor<Element, cutlass::layout::RowMajor> gQ({int64_t(1), p.gQ_strideB * p.num_batches});
    cutlass::HostTensor<Element, cutlass::layout::RowMajor> gK({int64_t(1), p.gK_strideB * p.num_batches});
    cutlass::HostTensor<Element, cutlass::layout::RowMajor> gV({int64_t(1), p.gV_strideB * p.num_batches});
    p.grad_query_ptr = gQ.device_data();
    p.grad_key_ptr = gK.device_data();
    p.grad_value_ptr = gV.device_data();

    if (!Kernel::check_supported(p)) {
      std::cerr << "FATAL: Kernel does not support these inputs" << std::endl;
      return 2;
    }

    // Run kernel
    cudaDeviceSynchronize();
    auto kernel_fn = attention_kernel_backward_batched_impl<Kernel>;
    size_t smem_bytes = sizeof(typename Kernel::SharedStorage);
    CUDA_CHECK(cudaFuncSetAttribute(kernel_fn, cudaFuncAttributeMaxDynamicSharedMemorySize, int(smem_bytes)));
    kernel_fn<<<p.getBlocksGrid(), p.getThreadsGrid(), smem_bytes>>>(p);

    // Write outputs
    std::cout << "OK ";
    writeTensor("grad_query", gQ);
    writeInt64("gQ_strideB", p.gQ_strideB);
    writeInt64("gQ_strideM", p.gQ_strideM());
    writeInt64("gQ_strideH", p.gQ_strideH);
    writeTensor("grad_key", gK);
    writeInt64("gK_strideB", p.gK_strideB);
    writeInt64("gK_strideM", p.gK_strideM());
    writeInt64("gK_strideH", p.gK_strideH);
    writeTensor("grad_value", gV);
    writeInt64("gV_strideB", p.gV_strideB);
    writeInt64("gV_strideM", p.gV_strideM());
    writeInt64("gV_strideH", p.gV_strideH);

    // Timing
    cudaEvent_t events[2];
    for (auto & event : events) {
      CUDA_CHECK(cudaEventCreate(&event));
    }
    CUDA_CHECK(cudaEventRecord(events[0]));
    for (int i = 0; i < repeat_count; ++i) {
        kernel_fn<<<p.getBlocksGrid(), p.getThreadsGrid(), smem_bytes>>>(p);
    }
    CUDA_CHECK(cudaEventRecord(events[1]));
    CUDA_CHECK(cudaEventSynchronize(events[1]));
    // Measure elapsed runtime
    float runtime_ms = 0;
    CUDA_CHECK(cudaEventElapsedTime(&runtime_ms, events[0], events[1]));

    std::cout << "runtime_ms " << runtime_ms / float(repeat_count) << std::endl;
    return 0;
}
```
**EN**: Lines 215-279. The second half of `runKernel()` allocates optional workspace for split-key reduction, derives BMHK gradient strides, allocates output tensors `gQ/gK/gV`, and points `Params` at those buffers. After `Kernel::check_supported(p)` gates unsupported shapes, the code sets the dynamic shared-memory attribute, launches `attention_kernel_backward_batched_impl<Kernel>`, writes gradients and their strides back through stdout, then reruns the same kernel in a timed loop with CUDA events. The backward invocation flow is therefore: deserialize inputs → configure params/workspace → launch kernel → serialize gradients → profile repeated launches.
**CN**: 第215-279行：`runKernel()` 的后半段会为 split-key 归约按需分配 workspace，推导 BMHK 梯度步长，分配输出张量 `gQ/gK/gV`，并把这些缓冲区地址写入 `Params`。`Kernel::check_supported(p)` 先筛掉不支持的形状，然后代码设置动态共享内存属性，启动 `attention_kernel_backward_batched_impl<Kernel>`，通过标准输出写回梯度及其步长，最后再用 CUDA event 对同一 kernel 做重复计时。因此其反向调用流为：反序列化输入 → 配置参数/工作区 → 启动 kernel → 序列化梯度 → 重复启动做性能计时。
```cpp

int main() {
    std::ios_base::sync_with_stdio(false);

    std::string dtype;
    std::cin >> dtype;
    std::cerr << "Running kernel with dtype: " << dtype << std::endl;
    if (dtype == "f16") {
        return runKernel<cutlass::half_t>();
    } else if (dtype == "b16") {
        return runKernel<cutlass::bfloat16_t>();
    } else if (dtype == "f32") {
        return runKernel<float>();
    } else {
        std::cerr << "FATAL: Unknown dtype: " << dtype << std::endl;
        return 3;
    }
}
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: Lines 280-298. `main()` only reads a dtype token and dispatches to `runKernel<half_t>`, `runKernel<bfloat16_t>`, or `runKernel<float>`. That minimal runner makes the template instantiation explicit while keeping all launch details inside `runKernel()` and the compile-time `DefaultKernel` policy.
**CN**: 第280-298行：`main()` 只读取一个数据类型标记，然后分发到 `runKernel<half_t>`、`runKernel<bfloat16_t>` 或 `runKernel<float>`。这个极简 runner 把模板实例化入口显式化，同时把所有启动细节都收拢在 `runKernel()` 与编译期 `DefaultKernel` 策略中。
---
## Key Concepts / 关键概念
- The FMHA backward example is not a full benchmark framework; it is a serialization wrapper that converts external tensors into `Kernel::Params` and launches one specialized kernel. / 这个 FMHA 反向示例不是完整 benchmark 框架，而是一个序列化包装层：它把外部张量转换为 `Kernel::Params`，再启动一个特化 kernel。
- Kernel configuration is decided mostly at compile time through `DefaultKernel`, especially block shapes, preload policy, register-file accumulation, and split-key support. / kernel 配置主要通过 `DefaultKernel` 在编译期确定，尤其是 block 形状、预取策略、寄存器累加以及 split-key 支持。
- The runner wires forward-state tensors such as `output`, `logsumexp`, and `delta` together with `grad_output`, which is exactly the information the backward FMHA kernel needs. / 该 runner 会把 `output`、`logsumexp`、`delta` 等前向状态与 `grad_output` 一并接入，这正是 FMHA 反向 kernel 所需的信息。
- Runtime safety comes from `Kernel::check_supported(p)` plus CUDA error checks and explicit shared-memory attribute setup before launch. / 运行时安全性主要来自 `Kernel::check_supported(p)`、CUDA 错误检查，以及启动前显式设置的共享内存属性。
## Dependencies / 依赖项
- `kernel_backward.h` — Defines the fused backward FMHA kernel, traits, and runtime entry points / 定义融合反向 FMHA 内核、traits 与运行时入口
- `cutlass/util/device_memory.h` — Allocates device-side workspace buffers and scratch storage / 分配设备侧工作区缓冲和临时存储
- `cutlass/util/host_tensor.h` — Owns host tensors used for setup, copies, and result checks / 管理用于初始化、拷贝和结果检查的主机张量
- `<iostream>, <fstream>` — Provide logging and optional tensor dump utilities for debugging / 提供日志输出和可选张量导出调试能力
