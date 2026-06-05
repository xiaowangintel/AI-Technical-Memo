# Passes.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/include/mlir/Dialect/GPU/Pipelines/Passes.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares public interfaces for the GPU dialect, focused on pipeline construction and pass orchestration helpers and `Passes`.
  - **CN**: 声明 GPU 方言中聚焦 `Passes` 的公共接口，覆盖流水线构建与 Pass 编排辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- Passes.h - GPU pipeline entry points--------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-13
```cpp

#ifndef MLIR_DIALECT_GPU_PIPELINES_PASSES_H_
#define MLIR_DIALECT_GPU_PIPELINES_PASSES_H_

#include "mlir/Pass/PassOptions.h"

```
- **EN**: Pulls in the headers needed by this declaration unit, including `mlir/Pass/PassOptions.h`.
- **CN**: 引入该声明单元所需的头文件，其中包括 `mlir/Pass/PassOptions.h`。

### Lines 14-25
```cpp
namespace mlir {
namespace gpu {

/// Options for the gpu to nvvm pipeline.
struct GPUToNVVMPipelineOptions
    : public PassPipelineOptions<GPUToNVVMPipelineOptions> {
  PassOptions::Option<int64_t> indexBitWidth{
      *this, "index-bitwidth",
      llvm::cl::desc("Bitwidth of the index type for the host (warning this "
                     "should be 64 until the GPU layering is fixed)"),
      llvm::cl::init(64)};
  PassOptions::Option<std::string> cubinTriple{
```
- **EN**: Introduces declarations for `mlir`, `gpu`, `GPUToNVVMPipelineOptions`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `mlir`, `gpu`, `GPUToNVVMPipelineOptions` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 26-37
```cpp
      *this, "cubin-triple",
      llvm::cl::desc("Triple to use to serialize to cubin."),
      llvm::cl::init("nvptx64-nvidia-cuda")};
  PassOptions::Option<std::string> cubinChip{
      *this, "cubin-chip", llvm::cl::desc("Chip to use to serialize to cubin."),
      llvm::cl::init("sm_75")};
  PassOptions::Option<std::string> cubinFeatures{
      *this, "cubin-features",
      llvm::cl::desc("Features to use to serialize to cubin."),
      llvm::cl::init("")};
  PassOptions::Option<std::string> cubinFormat{
      *this, "cubin-format",
```
- **EN**: Implements logic around `desc`, `init`.
- **CN**: 围绕 `desc`, `init` 实现具体逻辑。

### Lines 38-49
```cpp
      llvm::cl::desc("Compilation format to use to serialize to cubin."),
      llvm::cl::init("fatbin")};
  PassOptions::Option<std::string> cmdOptions{
      *this, "ptxas-cmd-options",
      llvm::cl::desc(
          "Command line options to pass to the downstream compiler."),
      llvm::cl::init("")};
  PassOptions::Option<int> optLevel{
      *this, "opt-level",
      llvm::cl::desc("Optimization level for NVVM compilation"),
      llvm::cl::init(2)};
  PassOptions::Option<bool> kernelUseBarePtrCallConv{
```
- **EN**: Implements logic around `desc`, `init`; this block packages logic as an MLIR pass or pass-related API; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `desc`, `init` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或相关 API，并涉及目标平台或加速器专用语义。

### Lines 50-61
```cpp
      *this, "kernel-bare-ptr-calling-convention",
      llvm::cl::desc(
          "Whether to use the bareptr calling convention on the kernel "
          "(warning this should be false until the GPU layering is fixed)"),
      llvm::cl::init(false)};
  PassOptions::Option<bool> hostUseBarePtrCallConv{
      *this, "host-bare-ptr-calling-convention",
      llvm::cl::desc(
          "Whether to use the bareptr calling convention on the host (warning "
          "this should be false until the GPU layering is fixed)"),
      llvm::cl::init(false)};
  PassOptions::Option<bool> allowPatternRollback{
```
- **EN**: Implements logic around `desc`, `init`, `host`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `desc`, `init`, `host` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 62-67
```cpp
      *this, "allow-pattern-rollback",
      llvm::cl::desc("Allow pattern rollback during dialect conversion"),
      llvm::cl::init(true)};
};

/// Options for the gpu to rocdl pipeline.
```
- **EN**: Declares APIs or declarative rules around `desc`, `init`; this block touches target- or accelerator-specific semantics.
- **CN**: 声明与 `desc`, `init` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义。

### Lines 68-79
```cpp
struct GPUToROCDLPipelineOptions
    : public PassPipelineOptions<GPUToROCDLPipelineOptions> {
  PassOptions::Option<int64_t> indexBitWidth{
      *this, "index-bitwidth",
      llvm::cl::desc("Bitwidth of the index type for the host (warning this "
                     "should be 64 until the GPU layering is fixed)"),
      llvm::cl::init(64)};
  PassOptions::Option<std::string> triple{
      *this, "triple",
      llvm::cl::desc("AMDGPU target triple (e.g. amdgcn-amd-amdhsa)."),
      llvm::cl::init("amdgcn-amd-amdhsa")};
  PassOptions::Option<std::string> chip{
```
- **EN**: Introduces declarations for `GPUToROCDLPipelineOptions`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPUToROCDLPipelineOptions` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 80-91
```cpp
      *this, "chip",
      llvm::cl::desc(
          "AMDGPU target chip (e.g. gfx90a, gfx942, gfx1100). Required: "
          "AMDGCN binaries are not forward-compatible across chip families.")};
  PassOptions::Option<std::string> features{
      *this, "features", llvm::cl::desc("AMDGPU target features."),
      llvm::cl::init("")};
  PassOptions::Option<std::string> binaryFormat{
      *this, "binary-format",
      llvm::cl::desc("Final GPU binary emission format (e.g. fatbin, binary, "
                     "isa, llvm, offloading)."),
      llvm::cl::init("fatbin")};
```
- **EN**: Implements logic around `desc`, `chip`, `init`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `desc`, `chip`, `init` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 92-103
```cpp
  PassOptions::Option<std::string> abiVersion{
      *this, "abi",
      llvm::cl::desc("AMDHSA ABI version (e.g. \"500\", \"600\")."),
      llvm::cl::init("600")};
  PassOptions::Option<bool> wave64{
      *this, "wave64",
      llvm::cl::desc("Use Wave64 mode (default true; wave32 if false, "
                     "appropriate for RDNA / gfx10+ where supported)."),
      llvm::cl::init(true)};
  PassOptions::Option<int> optLevel{
      *this, "opt-level",
      llvm::cl::desc("Optimization level for ROCDL/AMDGPU compilation."),
```
- **EN**: Implements logic around `desc`, `init`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `desc`, `init` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 104-115
```cpp
      llvm::cl::init(2)};
  PassOptions::Option<std::string> cmdOptions{
      *this, "rocdl-cmd-options",
      llvm::cl::desc(
          "Command line options to pass to the downstream AMDGPU compiler."),
      llvm::cl::init("")};
  PassOptions::Option<bool> kernelUseBarePtrCallConv{
      *this, "kernel-bare-ptr-calling-convention",
      llvm::cl::desc("Use bareptr calling convention for device kernels."),
      llvm::cl::init(false)};
  PassOptions::Option<bool> hostUseBarePtrCallConv{
      *this, "host-bare-ptr-calling-convention",
```
- **EN**: Implements logic around `init`, `desc`; this block packages logic as an MLIR pass or pass-related API; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `init`, `desc` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或相关 API，并涉及目标平台或加速器专用语义。

### Lines 116-121
```cpp
      llvm::cl::desc("Use bareptr calling convention for the host."),
      llvm::cl::init(false)};
};

// Options for the gpu to xevm pipeline.
struct GPUToXeVMPipelineOptions
```
- **EN**: Introduces declarations for `GPUToXeVMPipelineOptions`, establishing the public symbols or declarative records used later.
- **CN**: 引入 `GPUToXeVMPipelineOptions` 等声明，建立后续使用的公共符号或声明式记录。

### Lines 122-133
```cpp
    : public PassPipelineOptions<GPUToXeVMPipelineOptions> {
  PassOptions::Option<std::string> xegpuOpLevel{
      *this, "xegpu-op-level",
      llvm::cl::desc("Granularity of XeGPU operations to target: workgroup | "
                     "subgroup | lane"),
      llvm::cl::init("workgroup")};
  // General lowering controls.
  PassOptions::Option<bool> use64bitIndex{
      *this, "use-64bit-index",
      llvm::cl::desc("Bitwidth of the index type (host & device)"),
      llvm::cl::init(true)};
  PassOptions::Option<bool> kernelBarePtrCallConv{
```
- **EN**: Implements logic around `desc`, `init`; this block coordinates dialect conversion or lowering decisions; works with dialect IR entities such as ops, types, or attributes; touches target- or accelerator-specific semantics.
- **CN**: 围绕 `desc`, `init` 实现具体逻辑；该代码块协调方言转换或 lowering 决策，并处理方言 IR 实体，如操作、类型或属性，并涉及目标平台或加速器专用语义。

### Lines 134-145
```cpp
      *this, "kernel-bare-ptr-calling-convention",
      llvm::cl::desc("Use bare pointer calling convention for device kernels"),
      llvm::cl::init(false)};
  PassOptions::Option<bool> hostBarePtrCallConv{
      *this, "host-bare-ptr-calling-convention",
      llvm::cl::desc("Use bare pointer calling convention for host launches"),
      llvm::cl::init(false)};
  PassOptions::Option<std::string> binaryFormat{
      *this, "binary-format",
      llvm::cl::desc("Final GPU binary emission format (e.g. fatbin)"),
      llvm::cl::init("fatbin")};
  // Options mirroring xevm-attach-target (GpuXeVMAttachTarget).
```
- **EN**: Implements logic around `desc`, `init`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `desc`, `init` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 146-157
```cpp
  PassOptions::Option<std::string> xevmModuleMatcher{
      *this, "xevm-module-matcher",
      llvm::cl::desc("Regex to match gpu.module names for XeVM target attach"),
      llvm::cl::init("")};
  PassOptions::Option<std::string> zebinTriple{
      *this, "zebin-triple", llvm::cl::desc("Target triple for XeVM codegen"),
      llvm::cl::init("spirv64-unknown-unknown")};
  PassOptions::Option<std::string> zebinChip{
      *this, "zebin-chip", llvm::cl::desc("Target chip (e.g. pvc, bmg)"),
      llvm::cl::init("bmg")};
  PassOptions::Option<unsigned> optLevel{
      *this, "opt-level",
```
- **EN**: Implements logic around `desc`, `init`; this block touches target- or accelerator-specific semantics.
- **CN**: 围绕 `desc`, `init` 实现具体逻辑；该代码块涉及目标平台或加速器专用语义。

### Lines 158-165
```cpp
      llvm::cl::desc("Optimization level for attached target/codegen"),
      llvm::cl::init(2)};
  PassOptions::Option<std::string> cmdOptions{
      *this, "igc-cmd-options",
      llvm::cl::desc("Additional downstream compiler command line options"),
      llvm::cl::init("")};
};

```
- **EN**: Implements logic around `desc`, `init`.
- **CN**: 围绕 `desc`, `init` 实现具体逻辑。

### Lines 166-171
```cpp
//===----------------------------------------------------------------------===//
// Building and Registering.
//===----------------------------------------------------------------------===//

/// Adds the GPU to NVVM pipeline to the given pass manager. Transforms main
/// dialects into NVVM targets. Begins with GPU code regions, then handles host
```
- **EN**: Documents the next declarations, design intent, or usage constraints for this file.
- **CN**: 为后续声明、设计意图或使用约束提供说明。

### Lines 172-177
```cpp
/// code.
void buildLowerToNVVMPassPipeline(OpPassManager &pm,
                                  const GPUToNVVMPipelineOptions &options);

/// Adds the GPU to ROCDL pipeline to the given pass manager. Transforms main
/// dialects (arith, memref, scf, vector, gpu) into ROCDL/AMDGPU. Begins with
```
- **EN**: Declares APIs or declarative rules around `buildLowerToNVVMPassPipeline`; this block packages logic as an MLIR pass or pass-related API; touches target- or accelerator-specific semantics; coordinates behavior across core structured MLIR dialects.
- **CN**: 声明与 `buildLowerToNVVMPassPipeline` 相关的 API 或声明式规则；该代码块将逻辑组织为 MLIR Pass 或相关 API，并涉及目标平台或加速器专用语义，并协调核心结构化 MLIR 方言之间的行为。

### Lines 178-183
```cpp
/// GPU code regions, then handles host code.
void buildLowerToROCDLPassPipeline(OpPassManager &pm,
                                   const GPUToROCDLPipelineOptions &options);

/// Adds the GPU to XeVM pipeline to the given pass manager. Transforms main
/// dialects into XeVM targets. Begins with GPU code regions, then handles host
```
- **EN**: Declares APIs or declarative rules around `buildLowerToROCDLPassPipeline`; this block packages logic as an MLIR pass or pass-related API; touches target- or accelerator-specific semantics.
- **CN**: 声明与 `buildLowerToROCDLPassPipeline` 相关的 API 或声明式规则；该代码块将逻辑组织为 MLIR Pass 或相关 API，并涉及目标平台或加速器专用语义。

### Lines 184-192
```cpp
/// code.
void buildLowerToXeVMPassPipeline(OpPassManager &pm,
                                  const GPUToXeVMPipelineOptions &options);

/// Register all pipelines for the `gpu` dialect.
void registerGPUToNVVMPipeline();
void registerGPUToROCDLPipeline();
void registerGPUToXeVMPipeline();

```
- **EN**: Declares APIs or declarative rules around `buildLowerToXeVMPassPipeline`, `registerGPUToNVVMPipeline`, `registerGPUToROCDLPipeline`, `registerGPUToXeVMPipeline`; this block touches target- or accelerator-specific semantics.
- **CN**: 声明与 `buildLowerToXeVMPassPipeline`, `registerGPUToNVVMPipeline`, `registerGPUToROCDLPipeline`, `registerGPUToXeVMPipeline` 相关的 API 或声明式规则；该代码块涉及目标平台或加速器专用语义。

### Lines 193-196
```cpp
} // namespace gpu
} // namespace mlir

#endif
```
- **EN**: Defines preprocessor-controlled structure, include guards, generated hook points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、头文件保护、生成式钩子或编译期常量。

## Key Concepts / 关键概念

- **Dialect declarations / 方言声明**:
  - **EN**: Defines the public include-surface for a dialect, exposing operations, attributes, types, or interfaces.
  - **CN**: 定义方言的公共头文件表面，暴露操作、属性、类型或接口。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。
- **Target-specific semantics / 目标相关语义**:
  - **EN**: Encodes rules tied to accelerator, GPU, or binary target environments.
  - **CN**: 编码与加速器、GPU 或二进制目标环境绑定的规则。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Pass/PassOptions.h`
- **Subsystem categories / 子系统类别**: pass infrastructure and registration support / Pass 基础设施与注册支持 (1)
