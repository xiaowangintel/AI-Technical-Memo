# GPUToXeVMPipeline.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/GPU/Pipelines/GPUToXeVMPipeline.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This file implements a pass for testing the lowering to XeVM as a generally usable sink pass. If XeGPU ops are used, it expects the MLIR code to have XeGPU ops already embedded in gpu code.
  - **CN**: 实现 GPU 方言与异构加速器支持 中与 `GPUToXeVMPipeline` 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- GPUToXeVMPipeline.cpp - Lowering pipeline to XeVM/LLVM -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 8-14
```cpp
//
// This file implements a pass for testing the lowering to XeVM as a generally
// usable sink pass. If XeGPU ops are used, it expects the MLIR code to have
// XeGPU ops already embedded in gpu code.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 15-28
```cpp
#include "mlir/Conversion/AffineToStandard/AffineToStandard.h"
#include "mlir/Conversion/GPUCommon/GPUCommonPass.h"
#include "mlir/Conversion/MathToXeVM/MathToXeVM.h"
#include "mlir/Conversion/Passes.h"
#include "mlir/Conversion/SCFToControlFlow/SCFToControlFlow.h"
#include "mlir/Conversion/VectorToSCF/VectorToSCF.h"
#include "mlir/Conversion/XeGPUToXeVM/XeGPUToXeVM.h"
#include "mlir/Conversion/XeVMToLLVM/XeVMToLLVM.h"
#include "mlir/Dialect/Func/IR/FuncOps.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/GPU/Pipelines/Passes.h"
#include "mlir/Dialect/GPU/Transforms/Passes.h"
#include "mlir/Dialect/LLVMIR/Transforms/RequestCWrappers.h"
#include "mlir/Dialect/MemRef/Transforms/Passes.h"
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Conversion/AffineToStandard/AffineToStandard.h`, `mlir/Conversion/GPUCommon/GPUCommonPass.h`, `mlir/Conversion/MathToXeVM/MathToXeVM.h`, `mlir/Conversion/Passes.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Conversion/AffineToStandard/AffineToStandard.h`, `mlir/Conversion/GPUCommon/GPUCommonPass.h`, `mlir/Conversion/MathToXeVM/MathToXeVM.h`, `mlir/Conversion/Passes.h`。

### Lines 29-36
```cpp
#include "mlir/Dialect/XeGPU/Transforms/Passes.h"
#include "mlir/Pass/PassManager.h"
#include "mlir/Pass/PassOptions.h"
#include "mlir/Target/LLVM/XeVM/Target.h"
#include "mlir/Transforms/Passes.h"

using namespace mlir;

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/XeGPU/Transforms/Passes.h`, `mlir/Pass/PassManager.h`, `mlir/Pass/PassOptions.h`, `mlir/Target/LLVM/XeVM/Target.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/XeGPU/Transforms/Passes.h`, `mlir/Pass/PassManager.h`, `mlir/Pass/PassOptions.h`, `mlir/Target/LLVM/XeVM/Target.h`。

### Lines 37-50
```cpp
namespace {
//===----------------------------------------------------------------------===//
// Pre-GPU common pipeline for both Host and GPU.
//===----------------------------------------------------------------------===//
void buildPreGPUCommonPassPipeline(
    OpPassManager &pm, const mlir::gpu::GPUToXeVMPipelineOptions &options) {
  // builtin.module scope passes.
  pm.addPass(createCSEPass());
  pm.addPass(createConvertVectorToSCFPass());
  {
    GpuXeVMAttachTargetOptions xevmTargetOptions;
    xevmTargetOptions.moduleMatcher = options.xevmModuleMatcher;
    xevmTargetOptions.triple = options.zebinTriple;
    xevmTargetOptions.chip = options.zebinChip;
```
- **EN**: Implements logic around `buildPreGPUCommonPassPipeline`, `addPass`; this block participates in pass execution or pass construction; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `buildPreGPUCommonPassPipeline`, `addPass` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并编码加速器专用执行或 lowering 规则。

### Lines 51-58
```cpp
    xevmTargetOptions.optLevel = options.optLevel;
    xevmTargetOptions.cmdOptions = options.cmdOptions;
    pm.addPass(createGpuXeVMAttachTarget(xevmTargetOptions));
  }
  pm.addPass(createLowerAffinePass());
  pm.addNestedPass<func::FuncOp>(createGpuAsyncRegionPass());
}

```
- **EN**: Implements logic around `addPass`, `FuncOp>`; this block participates in pass execution or pass construction; works with symbol tables or function-like operations.
- **CN**: 围绕 `addPass`, `FuncOp>` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并处理符号表或类函数操作。

### Lines 59-72
```cpp
//===----------------------------------------------------------------------===//
// GPUModule-specific stuff.
//===----------------------------------------------------------------------===//
void buildGPUPassPipeline(OpPassManager &pm,
                          const mlir::gpu::GPUToXeVMPipelineOptions &options) {
  xegpu::XeGPUPropagateLayoutOptions laneLayoutOptions;
  laneLayoutOptions.indexBitWidth = options.use64bitIndex ? 64 : 32;
  laneLayoutOptions.layoutKind = "lane";
  pm.addNestedPass<ModuleOp>(createCSEPass());
  if (options.xegpuOpLevel == "workgroup") {
    xegpu::XeGPUPropagateLayoutOptions sgLayoutOptions;
    sgLayoutOptions.layoutKind = "subgroup";
    pm.addNestedPass<gpu::GPUModuleOp>(
        xegpu::createXeGPUPropagateLayout(sgLayoutOptions));
```
- **EN**: Implements logic around `buildGPUPassPipeline`, `addNestedPass`, `GPUModuleOp>`, `createXeGPUPropagateLayout`; this block participates in pass execution or pass construction; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `buildGPUPassPipeline`, `addNestedPass`, `GPUModuleOp>`, `createXeGPUPropagateLayout` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并编码加速器专用执行或 lowering 规则。

### Lines 73-86
```cpp
    pm.addNestedPass<gpu::GPUModuleOp>(xegpu::createXeGPUWgToSgDistribute());
    pm.addNestedPass<gpu::GPUModuleOp>(createCSEPass());
    pm.addNestedPass<gpu::GPUModuleOp>(createLowerAffinePass());
    pm.addNestedPass<gpu::GPUModuleOp>(createCSEPass());
    xegpu::XeGPUPropagateLayoutOptions instDataOptions;
    instDataOptions.layoutKind = "inst";
    pm.addNestedPass<gpu::GPUModuleOp>(
        xegpu::createXeGPUPropagateLayout(instDataOptions));
    pm.addNestedPass<gpu::GPUModuleOp>(xegpu::createXeGPUBlocking());
    pm.addNestedPass<gpu::GPUModuleOp>(createCSEPass());
  }
  if (options.xegpuOpLevel == "subgroup" ||
      options.xegpuOpLevel == "workgroup") {
    pm.addNestedPass<gpu::GPUModuleOp>(
```
- **EN**: Implements logic around `GPUModuleOp>`, `createXeGPUPropagateLayout`; this block participates in pass execution or pass construction; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `GPUModuleOp>`, `createXeGPUPropagateLayout` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并编码加速器专用执行或 lowering 规则。

### Lines 87-100
```cpp
        xegpu::createXeGPUPropagateLayout(laneLayoutOptions));
    pm.addNestedPass<gpu::GPUModuleOp>(xegpu::createXeGPUPeepHoleOptimizer());
    pm.addNestedPass<gpu::GPUModuleOp>(createCSEPass());
    pm.addNestedPass<gpu::GPUModuleOp>(
        xegpu::createXeGPUPropagateLayout(laneLayoutOptions));
    pm.addNestedPass<gpu::GPUModuleOp>(
        xegpu::createXeGPUSgToWiDistributeExperimental());
    pm.addNestedPass<gpu::GPUModuleOp>(createCanonicalizerPass());
    pm.addNestedPass<gpu::GPUModuleOp>(createCSEPass());
    pm.addNestedPass<gpu::GPUModuleOp>(createLoopInvariantCodeMotionPass());
    pm.addNestedPass<gpu::GPUModuleOp>(createCSEPass());
    pm.addNestedPass<gpu::GPUModuleOp>(xegpu::createXeGPUVectorLinearize());
  }
  pm.addNestedPass<gpu::GPUModuleOp>(createConvertMathToXeVM());
```
- **EN**: Implements logic around `createXeGPUPropagateLayout`, `GPUModuleOp>`, `createXeGPUSgToWiDistributeExperimental`; this block participates in pass execution or pass construction; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `createXeGPUPropagateLayout`, `GPUModuleOp>`, `createXeGPUSgToWiDistributeExperimental` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并编码加速器专用执行或 lowering 规则。

### Lines 101-114
```cpp
  ConvertXeGPUToXeVMPassOptions xegpuToXeVMOptions;
  xegpuToXeVMOptions.use64bitIndex = options.use64bitIndex;
  pm.addNestedPass<gpu::GPUModuleOp>(
      createConvertXeGPUToXeVMPass(xegpuToXeVMOptions));
  {
    ConvertGpuOpsToLLVMSPVOpsOptions gpuToLLVMSPVOptions;
    gpuToLLVMSPVOptions.use64bitIndex = options.use64bitIndex;
    pm.addNestedPass<gpu::GPUModuleOp>(
        createConvertGpuOpsToLLVMSPVOps(gpuToLLVMSPVOptions));
  }
  pm.addNestedPass<gpu::GPUModuleOp>(createCSEPass());
  pm.addNestedPass<gpu::GPUModuleOp>(createReconcileUnrealizedCastsPass());
}

```
- **EN**: Implements logic around `GPUModuleOp>`, `createConvertXeGPUToXeVMPass`, `createConvertGpuOpsToLLVMSPVOps`; this block participates in pass execution or pass construction; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `GPUModuleOp>`, `createConvertXeGPUToXeVMPass`, `createConvertGpuOpsToLLVMSPVOps` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并编码加速器专用执行或 lowering 规则。

### Lines 115-128
```cpp
//===----------------------------------------------------------------------===//
// Post-GPU pipeline for both Host and GPU.
//===----------------------------------------------------------------------===//
void buildPostGPUCommonPassPipeline(
    OpPassManager &pm, const mlir::gpu::GPUToXeVMPipelineOptions &options) {
  // builtin.module scope passes.
  pm.addPass(createSCFToControlFlowPass());
  pm.addPass(memref::createExpandStridedMetadataPass());
  {
    GpuToLLVMConversionPassOptions gpuToLLVMOptions;
    gpuToLLVMOptions.hostBarePtrCallConv = options.hostBarePtrCallConv;
    gpuToLLVMOptions.kernelBarePtrCallConv = options.kernelBarePtrCallConv;
    pm.addPass(createGpuToLLVMConversionPass(gpuToLLVMOptions));
  }
```
- **EN**: Implements logic around `buildPostGPUCommonPassPipeline`, `addPass`; this block participates in pass execution or pass construction; moves data between tensor-style values and explicit buffers; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `buildPostGPUCommonPassPipeline`, `addPass` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并在张量风格值与显式缓冲区之间移动数据，并编码加速器专用执行或 lowering 规则。

### Lines 129-142
```cpp
  pm.addPass(createLowerAffinePass());
  pm.addPass(createConvertVectorToLLVMPass());
  pm.addPass(createConvertToLLVMPass());
  pm.addPass(createReconcileUnrealizedCastsPass());
  pm.addNestedPass<gpu::GPUModuleOp>(createCanonicalizerPass());
  pm.addNestedPass<gpu::GPUModuleOp>(createCSEPass());
  // XeVM-to-LLVM must be the last pass before gpu-module-to-binary.
  pm.addNestedPass<gpu::GPUModuleOp>(createConvertXeVMToLLVMPass());
  // gpu-module-to-binary
  {
    GpuModuleToBinaryPassOptions gpuToModuleBinOptions;
    gpuToModuleBinOptions.compilationTarget = options.binaryFormat;
    gpuToModuleBinOptions.cmdOptions = options.cmdOptions;
    pm.addPass(createGpuModuleToBinaryPass(gpuToModuleBinOptions));
```
- **EN**: Implements logic around `addPass`, `GPUModuleOp>`; this block participates in pass execution or pass construction; encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `addPass`, `GPUModuleOp>` 实现具体逻辑；该代码块参与 pass 执行或 pass 构造，并编码加速器专用执行或 lowering 规则。

### Lines 143-151
```cpp
  }
}
} // namespace

void mlir::gpu::buildLowerToXeVMPassPipeline(
    OpPassManager &pm, const GPUToXeVMPipelineOptions &options) {
  // Pre-GPU common pipelines.
  buildPreGPUCommonPassPipeline(pm, options);

```
- **EN**: Implements logic around `buildLowerToXeVMPassPipeline`, `buildPreGPUCommonPassPipeline`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `buildLowerToXeVMPassPipeline`, `buildPreGPUCommonPassPipeline` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 152-158
```cpp
  // GPUModule-specific stuff.
  buildGPUPassPipeline(pm, options);

  // Post-GPU pipeline for both Host and GPU.
  buildPostGPUCommonPassPipeline(pm, options);
}

```
- **EN**: Implements logic around `buildGPUPassPipeline`, `buildPostGPUCommonPassPipeline`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `buildGPUPassPipeline`, `buildPostGPUCommonPassPipeline` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 159-167
```cpp
void mlir::gpu::registerGPUToXeVMPipeline() {
  PassPipelineRegistration<GPUToXeVMPipelineOptions>(
      "gpu-lower-to-xevm-pipeline",
      "The default GPU to XeVM lowering pipeline. It starts by lowering GPU "
      "code to the "
      "specified compilation target (default is fatbin) then lowers the host "
      "code.",
      buildLowerToXeVMPassPipeline);
}
```
- **EN**: Implements logic around `registerGPUToXeVMPipeline`, `PassPipelineRegistration`, `target`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `registerGPUToXeVMPipeline`, `PassPipelineRegistration`, `target` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

## Key Concepts / 关键概念

- **GPU execution model / GPU 执行模型**:
  - **EN**: Models kernels, memory spaces, launch geometry, and accelerator-specific rewrites.
  - **CN**: 建模 kernel、内存空间、启动几何以及加速器专用重写。
- **Region-based control flow / 基于 Region 的控制流**:
  - **EN**: Represents nested blocks and successors as first-class IR structure.
  - **CN**: 把嵌套 block 与后继边表示为一等 IR 结构。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Conversion/AffineToStandard/AffineToStandard.h`, `mlir/Conversion/GPUCommon/GPUCommonPass.h`, `mlir/Conversion/MathToXeVM/MathToXeVM.h`, `mlir/Conversion/Passes.h`, `mlir/Conversion/SCFToControlFlow/SCFToControlFlow.h`, `mlir/Conversion/VectorToSCF/VectorToSCF.h`, `mlir/Conversion/XeGPUToXeVM/XeGPUToXeVM.h`, `mlir/Conversion/XeVMToLLVM/XeVMToLLVM.h`, `mlir/Dialect/Func/IR/FuncOps.h`, `mlir/Dialect/GPU/IR/GPUDialect.h` ... (+9 more)
- **Subsystem categories / 子系统类别**: dialect conversion infrastructure / 方言转换基础设施 (8), other MLIR dialect declarations / 其他 MLIR 方言声明 (7), pass manager and pass registration support / Pass 管理器与 pass 注册支持 (2), common transformation and pattern-rewrite helpers / 通用变换与模式重写辅助工具 (1)
