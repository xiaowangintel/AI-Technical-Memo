# SparseGPUCodegen.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/SparseTensor/Transforms/SparseGPUCodegen.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This is a prototype GPU codegenerator for the sparsifier. The objective is to eventually use the right combination of direct code generation and libary calls into vendor-specific highly optimized sparse libraries (e.g. cuSparse for CUDA).
- **Purpose (CN)**: 实现稀疏张量优化、lowering 与重写 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===- SparseGPUCodegen.cpp - Generates GPU code --------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is a prototype GPU codegenerator for the sparsifier.
// The objective is to eventually use the right combination of
// direct code generation and libary calls into vendor-specific
// highly optimized sparse libraries (e.g. cuSparse for CUDA).
//
//===----------------------------------------------------------------------===//

#include "Utils/CodegenUtils.h"
#include "Utils/LoopEmitter.h"

#include "mlir/Dialect/Bufferization/IR/Bufferization.h"
#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/Linalg/IR/Linalg.h"
#include "mlir/Dialect/Linalg/Utils/Utils.h"
#include "mlir/Dialect/MemRef/IR/MemRef.h"
#include "mlir/Dialect/SCF/IR/SCF.h"
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This is a prototype GPU codegenerator for the sparsifier.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is a prototype GPU codegenerator for the sparsifier.`。
- **L10 EN**: Comment explains nearby logic, invariants, or intent: `The objective is to eventually use the right combination of`.
  **L10 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The objective is to eventually use the right combination of`。
- **L11 EN**: Comment explains nearby logic, invariants, or intent: `direct code generation and libary calls into vendor-specific`.
  **L11 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`direct code generation and libary calls into vendor-specific`。
- **L12 EN**: Comment explains nearby logic, invariants, or intent: `highly optimized sparse libraries (e.g. cuSparse for CUDA).`.
  **L12 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`highly optimized sparse libraries (e.g. cuSparse for CUDA).`。
- **L13 EN**: Separator comment used for visual grouping.
  **L13 CN**: 用于视觉分组的分隔注释。
- **L14 EN**: Banner comment marking a file or section boundary.
  **L14 CN**: 横幅注释，用于标记文件或章节边界。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L16 EN**: Includes "Utils/CodegenUtils.h" to access local declarations paired with this implementation unit.
  **L16 CN**: 引入 "Utils/CodegenUtils.h" 以使用与该实现单元配套的本地声明。
- **L17 EN**: Includes "Utils/LoopEmitter.h" to access local declarations paired with this implementation unit.
  **L17 CN**: 引入 "Utils/LoopEmitter.h" 以使用与该实现单元配套的本地声明。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Includes "mlir/Dialect/Bufferization/IR/Bufferization.h" to access dialect-specific IR, transforms, or shared utilities.
  **L19 CN**: 引入 "mlir/Dialect/Bufferization/IR/Bufferization.h" 以使用方言专用 IR、变换或共享工具。
- **L20 EN**: Includes "mlir/Dialect/GPU/IR/GPUDialect.h" to access dialect-specific IR, transforms, or shared utilities.
  **L20 CN**: 引入 "mlir/Dialect/GPU/IR/GPUDialect.h" 以使用方言专用 IR、变换或共享工具。
- **L21 EN**: Includes "mlir/Dialect/Linalg/IR/Linalg.h" to access dialect-specific IR, transforms, or shared utilities.
  **L21 CN**: 引入 "mlir/Dialect/Linalg/IR/Linalg.h" 以使用方言专用 IR、变换或共享工具。
- **L22 EN**: Includes "mlir/Dialect/Linalg/Utils/Utils.h" to access dialect-specific IR, transforms, or shared utilities.
  **L22 CN**: 引入 "mlir/Dialect/Linalg/Utils/Utils.h" 以使用方言专用 IR、变换或共享工具。
- **L23 EN**: Includes "mlir/Dialect/MemRef/IR/MemRef.h" to access dialect-specific IR, transforms, or shared utilities.
  **L23 CN**: 引入 "mlir/Dialect/MemRef/IR/MemRef.h" 以使用方言专用 IR、变换或共享工具。
- **L24 EN**: Includes "mlir/Dialect/SCF/IR/SCF.h" to access dialect-specific IR, transforms, or shared utilities.
  **L24 CN**: 引入 "mlir/Dialect/SCF/IR/SCF.h" 以使用方言专用 IR、变换或共享工具。

### Lines 25-48

````cpp
#include "mlir/Dialect/SparseTensor/IR/SparseTensor.h"
#include "mlir/Dialect/SparseTensor/IR/SparseTensorType.h"
#include "mlir/Dialect/SparseTensor/Transforms/Passes.h"
#include "mlir/IR/IRMapping.h"
#include "mlir/IR/Matchers.h"
#include "mlir/Interfaces/SideEffectInterfaces.h"
#include "llvm/Support/Casting.h"

using namespace mlir;
using namespace mlir::sparse_tensor;

namespace {

// Sparse formats supported by cuSparse.
enum class CuSparseFormat {
  kNone,
  kCOO,
  kCSR,
  kCSC,
  kBSR,
};

//===----------------------------------------------------------------------===//
// Helper methods.
````
- **L25 EN**: Includes "mlir/Dialect/SparseTensor/IR/SparseTensor.h" to access dialect-specific IR, transforms, or shared utilities.
  **L25 CN**: 引入 "mlir/Dialect/SparseTensor/IR/SparseTensor.h" 以使用方言专用 IR、变换或共享工具。
- **L26 EN**: Includes "mlir/Dialect/SparseTensor/IR/SparseTensorType.h" to access dialect-specific IR, transforms, or shared utilities.
  **L26 CN**: 引入 "mlir/Dialect/SparseTensor/IR/SparseTensorType.h" 以使用方言专用 IR、变换或共享工具。
- **L27 EN**: Includes "mlir/Dialect/SparseTensor/Transforms/Passes.h" to access dialect-specific IR, transforms, or shared utilities.
  **L27 CN**: 引入 "mlir/Dialect/SparseTensor/Transforms/Passes.h" 以使用方言专用 IR、变换或共享工具。
- **L28 EN**: Includes "mlir/IR/IRMapping.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L28 CN**: 引入 "mlir/IR/IRMapping.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L29 EN**: Includes "mlir/IR/Matchers.h" to access core MLIR IR classes such as operations, attributes, and types.
  **L29 CN**: 引入 "mlir/IR/Matchers.h" 以使用MLIR 核心 IR 类，如操作、属性与类型。
- **L30 EN**: Includes "mlir/Interfaces/SideEffectInterfaces.h" to access generic MLIR interfaces implemented by operations or dialects.
  **L30 CN**: 引入 "mlir/Interfaces/SideEffectInterfaces.h" 以使用由操作或方言实现的通用 MLIR 接口。
- **L31 EN**: Includes "llvm/Support/Casting.h" to access LLVM support-library facilities such as diagnostics and casting.
  **L31 CN**: 引入 "llvm/Support/Casting.h" 以使用LLVM 支持库设施，如诊断与类型转换。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L33 EN**: Brings namespace `mlir` into local scope.
  **L33 CN**: 将命名空间 `mlir` 引入当前作用域。
- **L34 EN**: Brings namespace `mlir::sparse_tensor` into local scope.
  **L34 CN**: 将命名空间 `mlir::sparse_tensor` 引入当前作用域。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L36 EN**: Opens namespace scope ``.
  **L36 CN**: 打开命名空间作用域 ``。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L38 EN**: Comment explains nearby logic, invariants, or intent: `Sparse formats supported by cuSparse.`.
  **L38 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sparse formats supported by cuSparse.`。
- **L39 EN**: Declares enum `class`.
  **L39 CN**: 声明 enum `class`。
- **L40 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `kNone,`.
  **L40 CN**: 继续一个多行参数列表、初始化器或聚合项：`kNone,`。
- **L41 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `kCOO,`.
  **L41 CN**: 继续一个多行参数列表、初始化器或聚合项：`kCOO,`。
- **L42 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `kCSR,`.
  **L42 CN**: 继续一个多行参数列表、初始化器或聚合项：`kCSR,`。
- **L43 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `kCSC,`.
  **L43 CN**: 继续一个多行参数列表、初始化器或聚合项：`kCSC,`。
- **L44 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `kBSR,`.
  **L44 CN**: 继续一个多行参数列表、初始化器或聚合项：`kBSR,`。
- **L45 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L45 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L47 EN**: Banner comment marking a file or section boundary.
  **L47 CN**: 横幅注释，用于标记文件或章节边界。
- **L48 EN**: Comment explains nearby logic, invariants, or intent: `Helper methods.`.
  **L48 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper methods.`。

### Lines 49-72

````cpp
//===----------------------------------------------------------------------===//

/// Marks the given top module as a GPU container module.
static void markAsGPUContainer(ModuleOp topModule) {
  topModule->setAttr(gpu::GPUDialect::getContainerModuleAttrName(),
                     UnitAttr::get(topModule->getContext()));
}

/// Constructs a new GPU module (for GPU kernels) inside the given top module,
/// or returns an existing GPU module if one was built previously.
static gpu::GPUModuleOp genGPUModule(OpBuilder &builder, ModuleOp topModule) {
  for (auto op : topModule.getBodyRegion().getOps<gpu::GPUModuleOp>())
    return op; // existing
  markAsGPUContainer(topModule);
  builder.setInsertionPointToStart(topModule.getBody());
  return gpu::GPUModuleOp::create(builder, topModule->getLoc(),
                                  "sparse_kernels");
}

/// Constructs a new GPU kernel in the given GPU module.
static gpu::GPUFuncOp genGPUFunc(OpBuilder &builder, gpu::GPUModuleOp gpuModule,
                                 SmallVectorImpl<Value> &args) {
  // Get a unique kernel name. Not very creative,
  // but we simply try kernel0, kernel1, etc.
````
- **L49 EN**: Banner comment marking a file or section boundary.
  **L49 CN**: 横幅注释，用于标记文件或章节边界。
- **L50 EN**: Blank line separating nearby declarations or logic blocks.
  **L50 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L51 EN**: Comment explains nearby logic, invariants, or intent: `Marks the given top module as a GPU container module.`.
  **L51 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Marks the given top module as a GPU container module.`。
- **L52 EN**: Starts a function, method, lambda, or structured scope: `static void markAsGPUContainer(ModuleOp topModule) {`.
  **L52 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static void markAsGPUContainer(ModuleOp topModule) {`。
- **L53 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `topModule->setAttr(gpu::GPUDialect::getContainerModuleAttrName(),`.
  **L53 CN**: 继续一个多行参数列表、初始化器或聚合项：`topModule->setAttr(gpu::GPUDialect::getContainerModuleAttrName(),`。
- **L54 EN**: Executes a call or declaration centered on `UnitAttr::get`.
  **L54 CN**: 执行以 `UnitAttr::get` 为核心的调用或声明。
- **L55 EN**: Closes the current lexical scope or compound statement.
  **L55 CN**: 结束当前词法作用域或复合语句块。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L57 EN**: Comment explains nearby logic, invariants, or intent: `Constructs a new GPU module (for GPU kernels) inside the given top module,`.
  **L57 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Constructs a new GPU module (for GPU kernels) inside the given top module,`。
- **L58 EN**: Comment explains nearby logic, invariants, or intent: `or returns an existing GPU module if one was built previously.`.
  **L58 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`or returns an existing GPU module if one was built previously.`。
- **L59 EN**: Starts a function, method, lambda, or structured scope: `static gpu::GPUModuleOp genGPUModule(OpBuilder &builder, ModuleOp topModule) {`.
  **L59 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static gpu::GPUModuleOp genGPUModule(OpBuilder &builder, ModuleOp topModule) {`。
- **L60 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L60 CN**: 开始 `for` 控制流语句并计算其条件。
- **L61 EN**: Returns from the current function with `op; // existing`.
  **L61 CN**: 以 `op; // existing` 从当前函数返回。
- **L62 EN**: Executes a call or declaration centered on `markAsGPUContainer`.
  **L62 CN**: 执行以 `markAsGPUContainer` 为核心的调用或声明。
- **L63 EN**: Executes a call or declaration centered on `builder.setInsertionPointToStart`.
  **L63 CN**: 执行以 `builder.setInsertionPointToStart` 为核心的调用或声明。
- **L64 EN**: Returns from the current function with `gpu::GPUModuleOp::create(builder, topModule->getLoc(),`.
  **L64 CN**: 以 `gpu::GPUModuleOp::create(builder, topModule->getLoc(),` 从当前函数返回。
- **L65 EN**: Executes a standalone statement or declaration: `"sparse_kernels");`.
  **L65 CN**: 执行一条独立语句或声明：`"sparse_kernels");`。
- **L66 EN**: Closes the current lexical scope or compound statement.
  **L66 CN**: 结束当前词法作用域或复合语句块。
- **L67 EN**: Blank line separating nearby declarations or logic blocks.
  **L67 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L68 EN**: Comment explains nearby logic, invariants, or intent: `Constructs a new GPU kernel in the given GPU module.`.
  **L68 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Constructs a new GPU kernel in the given GPU module.`。
- **L69 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static gpu::GPUFuncOp genGPUFunc(OpBuilder &builder, gpu::GPUModuleOp gpuModule,`.
  **L69 CN**: 继续一个多行参数列表、初始化器或聚合项：`static gpu::GPUFuncOp genGPUFunc(OpBuilder &builder, gpu::GPUModuleOp gpuModule,`。
- **L70 EN**: Continues the surrounding expression or declaration: `SmallVectorImpl<Value> &args) {`.
  **L70 CN**: 继续构造周围的表达式或声明：`SmallVectorImpl<Value> &args) {`。
- **L71 EN**: Comment explains nearby logic, invariants, or intent: `Get a unique kernel name. Not very creative,`.
  **L71 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get a unique kernel name. Not very creative,`。
- **L72 EN**: Comment explains nearby logic, invariants, or intent: `but we simply try kernel0, kernel1, etc.`.
  **L72 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`but we simply try kernel0, kernel1, etc.`。

### Lines 73-96

````cpp
  unsigned kernelNumber = 0;
  SmallString<16> kernelName;
  do {
    kernelName.clear();
    ("kernel" + Twine(kernelNumber++)).toStringRef(kernelName);
  } while (gpuModule.lookupSymbol(kernelName));
  // Then we insert a new kernel with given arguments into the module.
  builder.setInsertionPointToStart(gpuModule.getBody());
  SmallVector<Type> argsTp;
  for (auto arg : args)
    argsTp.push_back(arg.getType());
  FunctionType type = FunctionType::get(gpuModule->getContext(), argsTp, {});
  auto gpuFunc =
      gpu::GPUFuncOp::create(builder, gpuModule->getLoc(), kernelName, type);
  gpuFunc.setKernel(true);
  return gpuFunc;
}

/// Constructs code to launch GPU kernel.
static Value genLaunchGPUFunc(OpBuilder &builder, gpu::GPUFuncOp gpuFunc,
                              SmallVectorImpl<Value> &args,
                              SmallVectorImpl<Value> &tokens,
                              unsigned numThreads) {
  Location loc = gpuFunc->getLoc();
````
- **L73 EN**: Initializes variable `kernelNumber` from the right-hand expression.
  **L73 CN**: 使用右侧表达式初始化变量 `kernelNumber`。
- **L74 EN**: Executes a standalone statement or declaration: `SmallString<16> kernelName;`.
  **L74 CN**: 执行一条独立语句或声明：`SmallString<16> kernelName;`。
- **L75 EN**: Continues the surrounding expression or declaration: `do {`.
  **L75 CN**: 继续构造周围的表达式或声明：`do {`。
- **L76 EN**: Executes a call or declaration centered on `kernelName.clear`.
  **L76 CN**: 执行以 `kernelName.clear` 为核心的调用或声明。
- **L77 EN**: Executes a call or declaration centered on `statement`.
  **L77 CN**: 执行以 `statement` 为核心的调用或声明。
- **L78 EN**: Executes a call or declaration centered on `while`.
  **L78 CN**: 执行以 `while` 为核心的调用或声明。
- **L79 EN**: Comment explains nearby logic, invariants, or intent: `Then we insert a new kernel with given arguments into the module.`.
  **L79 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Then we insert a new kernel with given arguments into the module.`。
- **L80 EN**: Executes a call or declaration centered on `builder.setInsertionPointToStart`.
  **L80 CN**: 执行以 `builder.setInsertionPointToStart` 为核心的调用或声明。
- **L81 EN**: Executes a standalone statement or declaration: `SmallVector<Type> argsTp;`.
  **L81 CN**: 执行一条独立语句或声明：`SmallVector<Type> argsTp;`。
- **L82 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L82 CN**: 开始 `for` 控制流语句并计算其条件。
- **L83 EN**: Executes a call or declaration centered on `argsTp.push_back`.
  **L83 CN**: 执行以 `argsTp.push_back` 为核心的调用或声明。
- **L84 EN**: Initializes variable `type` from the right-hand expression.
  **L84 CN**: 使用右侧表达式初始化变量 `type`。
- **L85 EN**: Continues the surrounding expression or declaration: `auto gpuFunc =`.
  **L85 CN**: 继续构造周围的表达式或声明：`auto gpuFunc =`。
- **L86 EN**: Executes a call or declaration centered on `gpu::GPUFuncOp::create`.
  **L86 CN**: 执行以 `gpu::GPUFuncOp::create` 为核心的调用或声明。
- **L87 EN**: Executes a call or declaration centered on `gpuFunc.setKernel`.
  **L87 CN**: 执行以 `gpuFunc.setKernel` 为核心的调用或声明。
- **L88 EN**: Returns from the current function with `gpuFunc`.
  **L88 CN**: 以 `gpuFunc` 从当前函数返回。
- **L89 EN**: Closes the current lexical scope or compound statement.
  **L89 CN**: 结束当前词法作用域或复合语句块。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L91 EN**: Comment explains nearby logic, invariants, or intent: `Constructs code to launch GPU kernel.`.
  **L91 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Constructs code to launch GPU kernel.`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value genLaunchGPUFunc(OpBuilder &builder, gpu::GPUFuncOp gpuFunc,`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value genLaunchGPUFunc(OpBuilder &builder, gpu::GPUFuncOp gpuFunc,`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVectorImpl<Value> &args,`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVectorImpl<Value> &args,`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVectorImpl<Value> &tokens,`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVectorImpl<Value> &tokens,`。
- **L95 EN**: Continues the surrounding expression or declaration: `unsigned numThreads) {`.
  **L95 CN**: 继续构造周围的表达式或声明：`unsigned numThreads) {`。
- **L96 EN**: Initializes variable `loc` from the right-hand expression.
  **L96 CN**: 使用右侧表达式初始化变量 `loc`。

### Lines 97-120

````cpp
  Value none = TypedValue<::mlir::IntegerType>{};
  Value one = constantIndex(builder, loc, 1);
  Value numT = constantIndex(builder, loc, numThreads);
  gpu::KernelDim3 gridSize = {one, one, one};
  gpu::KernelDim3 blckSize = {numT, one, one};
  return gpu::LaunchFuncOp::create(builder, loc, gpuFunc, gridSize, blckSize,
                                   /*dynSharedMemSz*/ none, args,
                                   builder.getType<gpu::AsyncTokenType>(),
                                   tokens)
      .getAsyncToken();
}

/// Maps the provided ranked host buffer into the device address space.
/// Writes from the host are guaranteed to be visible to device kernels
/// that are launched afterwards. Writes from the device are guaranteed
/// to be visible on the host after synchronizing with the device kernel
/// completion. Needs to cast the buffer to a unranked buffer.
static Value genHostRegisterMemref(OpBuilder &builder, Location loc,
                                   Value mem) {
  MemRefType memTp = cast<MemRefType>(mem.getType());
  UnrankedMemRefType resTp =
      UnrankedMemRefType::get(memTp.getElementType(), /*memorySpace=*/0);
  Value cast = memref::CastOp::create(builder, loc, resTp, mem);
  gpu::HostRegisterOp::create(builder, loc, cast);
````
- **L97 EN**: Initializes variable `none` from the right-hand expression.
  **L97 CN**: 使用右侧表达式初始化变量 `none`。
- **L98 EN**: Initializes variable `one` from the right-hand expression.
  **L98 CN**: 使用右侧表达式初始化变量 `one`。
- **L99 EN**: Initializes variable `numT` from the right-hand expression.
  **L99 CN**: 使用右侧表达式初始化变量 `numT`。
- **L100 EN**: Initializes variable `gridSize` from the right-hand expression.
  **L100 CN**: 使用右侧表达式初始化变量 `gridSize`。
- **L101 EN**: Initializes variable `blckSize` from the right-hand expression.
  **L101 CN**: 使用右侧表达式初始化变量 `blckSize`。
- **L102 EN**: Returns from the current function with `gpu::LaunchFuncOp::create(builder, loc, gpuFunc, gridSize, blckSize,`.
  **L102 CN**: 以 `gpu::LaunchFuncOp::create(builder, loc, gpuFunc, gridSize, blckSize,` 从当前函数返回。
- **L103 EN**: Comment explains nearby logic, invariants, or intent: `dynSharedMemSz*/ none, args,`.
  **L103 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`dynSharedMemSz*/ none, args,`。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `builder.getType<gpu::AsyncTokenType>(),`.
  **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`builder.getType<gpu::AsyncTokenType>(),`。
- **L105 EN**: Continues the surrounding expression or declaration: `tokens)`.
  **L105 CN**: 继续构造周围的表达式或声明：`tokens)`。
- **L106 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L106 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L107 EN**: Closes the current lexical scope or compound statement.
  **L107 CN**: 结束当前词法作用域或复合语句块。
- **L108 EN**: Blank line separating nearby declarations or logic blocks.
  **L108 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L109 EN**: Comment explains nearby logic, invariants, or intent: `Maps the provided ranked host buffer into the device address space.`.
  **L109 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Maps the provided ranked host buffer into the device address space.`。
- **L110 EN**: Comment explains nearby logic, invariants, or intent: `Writes from the host are guaranteed to be visible to device kernels`.
  **L110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Writes from the host are guaranteed to be visible to device kernels`。
- **L111 EN**: Comment explains nearby logic, invariants, or intent: `that are launched afterwards. Writes from the device are guaranteed`.
  **L111 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`that are launched afterwards. Writes from the device are guaranteed`。
- **L112 EN**: Comment explains nearby logic, invariants, or intent: `to be visible on the host after synchronizing with the device kernel`.
  **L112 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to be visible on the host after synchronizing with the device kernel`。
- **L113 EN**: Comment explains nearby logic, invariants, or intent: `completion. Needs to cast the buffer to a unranked buffer.`.
  **L113 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`completion. Needs to cast the buffer to a unranked buffer.`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value genHostRegisterMemref(OpBuilder &builder, Location loc,`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value genHostRegisterMemref(OpBuilder &builder, Location loc,`。
- **L115 EN**: Continues the surrounding expression or declaration: `Value mem) {`.
  **L115 CN**: 继续构造周围的表达式或声明：`Value mem) {`。
- **L116 EN**: Initializes variable `memTp` from the right-hand expression.
  **L116 CN**: 使用右侧表达式初始化变量 `memTp`。
- **L117 EN**: Continues the surrounding expression or declaration: `UnrankedMemRefType resTp =`.
  **L117 CN**: 继续构造周围的表达式或声明：`UnrankedMemRefType resTp =`。
- **L118 EN**: Executes a call or declaration centered on `UnrankedMemRefType::get`.
  **L118 CN**: 执行以 `UnrankedMemRefType::get` 为核心的调用或声明。
- **L119 EN**: Initializes variable `cast` from the right-hand expression.
  **L119 CN**: 使用右侧表达式初始化变量 `cast`。
- **L120 EN**: Executes a call or declaration centered on `gpu::HostRegisterOp::create`.
  **L120 CN**: 执行以 `gpu::HostRegisterOp::create` 为核心的调用或声明。

### Lines 121-144

````cpp
  return cast;
}

/// Unmaps the provided buffer, expecting the casted buffer.
static void genHostUnregisterMemref(OpBuilder &builder, Location loc,
                                    Value cast) {
  gpu::HostUnregisterOp::create(builder, loc, cast);
}

/// Generates first wait in an asynchronous chain.
static Value genFirstWait(OpBuilder &builder, Location loc) {
  Type tokenType = builder.getType<gpu::AsyncTokenType>();
  return gpu::WaitOp::create(builder, loc, tokenType, ValueRange())
      .getAsyncToken();
}

/// Generates last, blocking wait in an asynchronous chain.
static void genBlockingWait(OpBuilder &builder, Location loc,
                            ValueRange operands) {
  gpu::WaitOp::create(builder, loc, Type(), operands);
}

/// Allocates memory on the device.
/// TODO: A `host_shared` attribute could be used to indicate that
````
- **L121 EN**: Returns from the current function with `cast`.
  **L121 CN**: 以 `cast` 从当前函数返回。
- **L122 EN**: Closes the current lexical scope or compound statement.
  **L122 CN**: 结束当前词法作用域或复合语句块。
- **L123 EN**: Blank line separating nearby declarations or logic blocks.
  **L123 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L124 EN**: Comment explains nearby logic, invariants, or intent: `Unmaps the provided buffer, expecting the casted buffer.`.
  **L124 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Unmaps the provided buffer, expecting the casted buffer.`。
- **L125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void genHostUnregisterMemref(OpBuilder &builder, Location loc,`.
  **L125 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void genHostUnregisterMemref(OpBuilder &builder, Location loc,`。
- **L126 EN**: Continues the surrounding expression or declaration: `Value cast) {`.
  **L126 CN**: 继续构造周围的表达式或声明：`Value cast) {`。
- **L127 EN**: Executes a call or declaration centered on `gpu::HostUnregisterOp::create`.
  **L127 CN**: 执行以 `gpu::HostUnregisterOp::create` 为核心的调用或声明。
- **L128 EN**: Closes the current lexical scope or compound statement.
  **L128 CN**: 结束当前词法作用域或复合语句块。
- **L129 EN**: Blank line separating nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L130 EN**: Comment explains nearby logic, invariants, or intent: `Generates first wait in an asynchronous chain.`.
  **L130 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generates first wait in an asynchronous chain.`。
- **L131 EN**: Starts a function, method, lambda, or structured scope: `static Value genFirstWait(OpBuilder &builder, Location loc) {`.
  **L131 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static Value genFirstWait(OpBuilder &builder, Location loc) {`。
- **L132 EN**: Initializes variable `tokenType` from the right-hand expression.
  **L132 CN**: 使用右侧表达式初始化变量 `tokenType`。
- **L133 EN**: Returns from the current function with `gpu::WaitOp::create(builder, loc, tokenType, ValueRange())`.
  **L133 CN**: 以 `gpu::WaitOp::create(builder, loc, tokenType, ValueRange())` 从当前函数返回。
- **L134 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L134 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L135 EN**: Closes the current lexical scope or compound statement.
  **L135 CN**: 结束当前词法作用域或复合语句块。
- **L136 EN**: Blank line separating nearby declarations or logic blocks.
  **L136 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L137 EN**: Comment explains nearby logic, invariants, or intent: `Generates last, blocking wait in an asynchronous chain.`.
  **L137 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generates last, blocking wait in an asynchronous chain.`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void genBlockingWait(OpBuilder &builder, Location loc,`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void genBlockingWait(OpBuilder &builder, Location loc,`。
- **L139 EN**: Continues the surrounding expression or declaration: `ValueRange operands) {`.
  **L139 CN**: 继续构造周围的表达式或声明：`ValueRange operands) {`。
- **L140 EN**: Executes a call or declaration centered on `gpu::WaitOp::create`.
  **L140 CN**: 执行以 `gpu::WaitOp::create` 为核心的调用或声明。
- **L141 EN**: Closes the current lexical scope or compound statement.
  **L141 CN**: 结束当前词法作用域或复合语句块。
- **L142 EN**: Blank line separating nearby declarations or logic blocks.
  **L142 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L143 EN**: Comment explains nearby logic, invariants, or intent: `Allocates memory on the device.`.
  **L143 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Allocates memory on the device.`。
- **L144 EN**: Comment records a pending task or caution: `TODO: A `host_shared` attribute could be used to indicate that`.
  **L144 CN**: 注释记录了待办事项或注意点：`TODO: A `host_shared` attribute could be used to indicate that`。

### Lines 145-168

````cpp
///       the buffer is visible by both host and device, but lowering
///       that feature does not seem to be fully supported yet.
static gpu::AllocOp genAllocMemRef(OpBuilder &builder, Location loc, Value mem,
                                   Value token) {
  auto tp = cast<ShapedType>(mem.getType());
  auto elemTp = tp.getElementType();
  auto shape = tp.getShape();
  auto memTp = MemRefType::get(shape, elemTp);
  SmallVector<Value> dynamicSizes;
  for (unsigned r = 0, rank = tp.getRank(); r < rank; r++) {
    if (shape[r] == ShapedType::kDynamic) {
      Value dimOp = linalg::createOrFoldDimOp(builder, loc, mem, r);
      dynamicSizes.push_back(dimOp);
    }
  }
  return gpu::AllocOp::create(builder, loc, TypeRange({memTp, token.getType()}),
                              token, dynamicSizes, ValueRange());
}

// Allocates a typed buffer on the host with given size.
static Value genHostBuffer(OpBuilder &builder, Location loc, Type type,
                           Value size) {
  const auto memTp = MemRefType::get({ShapedType::kDynamic}, type);
  return memref::AllocOp::create(builder, loc, memTp, size).getResult();
````
- **L145 EN**: Comment explains nearby logic, invariants, or intent: `the buffer is visible by both host and device, but lowering`.
  **L145 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the buffer is visible by both host and device, but lowering`。
- **L146 EN**: Comment explains nearby logic, invariants, or intent: `that feature does not seem to be fully supported yet.`.
  **L146 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`that feature does not seem to be fully supported yet.`。
- **L147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static gpu::AllocOp genAllocMemRef(OpBuilder &builder, Location loc, Value mem,`.
  **L147 CN**: 继续一个多行参数列表、初始化器或聚合项：`static gpu::AllocOp genAllocMemRef(OpBuilder &builder, Location loc, Value mem,`。
- **L148 EN**: Continues the surrounding expression or declaration: `Value token) {`.
  **L148 CN**: 继续构造周围的表达式或声明：`Value token) {`。
- **L149 EN**: Initializes variable `tp` from the right-hand expression.
  **L149 CN**: 使用右侧表达式初始化变量 `tp`。
- **L150 EN**: Initializes variable `elemTp` from the right-hand expression.
  **L150 CN**: 使用右侧表达式初始化变量 `elemTp`。
- **L151 EN**: Initializes variable `shape` from the right-hand expression.
  **L151 CN**: 使用右侧表达式初始化变量 `shape`。
- **L152 EN**: Initializes variable `memTp` from the right-hand expression.
  **L152 CN**: 使用右侧表达式初始化变量 `memTp`。
- **L153 EN**: Executes a standalone statement or declaration: `SmallVector<Value> dynamicSizes;`.
  **L153 CN**: 执行一条独立语句或声明：`SmallVector<Value> dynamicSizes;`。
- **L154 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L154 CN**: 开始 `for` 控制流语句并计算其条件。
- **L155 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L155 CN**: 开始 `if` 控制流语句并计算其条件。
- **L156 EN**: Initializes variable `dimOp` from the right-hand expression.
  **L156 CN**: 使用右侧表达式初始化变量 `dimOp`。
- **L157 EN**: Executes a call or declaration centered on `dynamicSizes.push_back`.
  **L157 CN**: 执行以 `dynamicSizes.push_back` 为核心的调用或声明。
- **L158 EN**: Closes the current lexical scope or compound statement.
  **L158 CN**: 结束当前词法作用域或复合语句块。
- **L159 EN**: Closes the current lexical scope or compound statement.
  **L159 CN**: 结束当前词法作用域或复合语句块。
- **L160 EN**: Returns from the current function with `gpu::AllocOp::create(builder, loc, TypeRange({memTp, token.getType()}),`.
  **L160 CN**: 以 `gpu::AllocOp::create(builder, loc, TypeRange({memTp, token.getType()}),` 从当前函数返回。
- **L161 EN**: Executes a call or declaration centered on `ValueRange`.
  **L161 CN**: 执行以 `ValueRange` 为核心的调用或声明。
- **L162 EN**: Closes the current lexical scope or compound statement.
  **L162 CN**: 结束当前词法作用域或复合语句块。
- **L163 EN**: Blank line separating nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L164 EN**: Comment explains nearby logic, invariants, or intent: `Allocates a typed buffer on the host with given size.`.
  **L164 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Allocates a typed buffer on the host with given size.`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value genHostBuffer(OpBuilder &builder, Location loc, Type type,`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value genHostBuffer(OpBuilder &builder, Location loc, Type type,`。
- **L166 EN**: Continues the surrounding expression or declaration: `Value size) {`.
  **L166 CN**: 继续构造周围的表达式或声明：`Value size) {`。
- **L167 EN**: Initializes variable `memTp` from the right-hand expression.
  **L167 CN**: 使用右侧表达式初始化变量 `memTp`。
- **L168 EN**: Returns from the current function with `memref::AllocOp::create(builder, loc, memTp, size).getResult()`.
  **L168 CN**: 以 `memref::AllocOp::create(builder, loc, memTp, size).getResult()` 从当前函数返回。

### Lines 169-192

````cpp
}

// Allocates a typed buffer on the device with given size.
static gpu::AllocOp genAllocBuffer(OpBuilder &builder, Location loc, Type type,
                                   Value size, Value token) {
  const auto memTp = MemRefType::get({ShapedType::kDynamic}, type);
  return gpu::AllocOp::create(builder, loc, TypeRange({memTp, token.getType()}),
                              token, size, ValueRange());
}

// Allocates a void buffer on the device with given size.
static gpu::AllocOp genAllocBuffer(OpBuilder &builder, Location loc, Value size,
                                   Value token) {
  return genAllocBuffer(builder, loc, builder.getI8Type(), size, token);
}

/// Deallocates memory from the device.
static Value genDeallocMemRef(OpBuilder &builder, Location loc, Value mem,
                              Value token) {
  return gpu::DeallocOp::create(builder, loc, token.getType(), token, mem)
      .getAsyncToken();
}

/// Copies memory between host and device (direction is implicit).
````
- **L169 EN**: Closes the current lexical scope or compound statement.
  **L169 CN**: 结束当前词法作用域或复合语句块。
- **L170 EN**: Blank line separating nearby declarations or logic blocks.
  **L170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L171 EN**: Comment explains nearby logic, invariants, or intent: `Allocates a typed buffer on the device with given size.`.
  **L171 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Allocates a typed buffer on the device with given size.`。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static gpu::AllocOp genAllocBuffer(OpBuilder &builder, Location loc, Type type,`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`static gpu::AllocOp genAllocBuffer(OpBuilder &builder, Location loc, Type type,`。
- **L173 EN**: Continues the surrounding expression or declaration: `Value size, Value token) {`.
  **L173 CN**: 继续构造周围的表达式或声明：`Value size, Value token) {`。
- **L174 EN**: Initializes variable `memTp` from the right-hand expression.
  **L174 CN**: 使用右侧表达式初始化变量 `memTp`。
- **L175 EN**: Returns from the current function with `gpu::AllocOp::create(builder, loc, TypeRange({memTp, token.getType()}),`.
  **L175 CN**: 以 `gpu::AllocOp::create(builder, loc, TypeRange({memTp, token.getType()}),` 从当前函数返回。
- **L176 EN**: Executes a call or declaration centered on `ValueRange`.
  **L176 CN**: 执行以 `ValueRange` 为核心的调用或声明。
- **L177 EN**: Closes the current lexical scope or compound statement.
  **L177 CN**: 结束当前词法作用域或复合语句块。
- **L178 EN**: Blank line separating nearby declarations or logic blocks.
  **L178 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L179 EN**: Comment explains nearby logic, invariants, or intent: `Allocates a void buffer on the device with given size.`.
  **L179 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Allocates a void buffer on the device with given size.`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static gpu::AllocOp genAllocBuffer(OpBuilder &builder, Location loc, Value size,`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`static gpu::AllocOp genAllocBuffer(OpBuilder &builder, Location loc, Value size,`。
- **L181 EN**: Continues the surrounding expression or declaration: `Value token) {`.
  **L181 CN**: 继续构造周围的表达式或声明：`Value token) {`。
- **L182 EN**: Returns from the current function with `genAllocBuffer(builder, loc, builder.getI8Type(), size, token)`.
  **L182 CN**: 以 `genAllocBuffer(builder, loc, builder.getI8Type(), size, token)` 从当前函数返回。
- **L183 EN**: Closes the current lexical scope or compound statement.
  **L183 CN**: 结束当前词法作用域或复合语句块。
- **L184 EN**: Blank line separating nearby declarations or logic blocks.
  **L184 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L185 EN**: Comment explains nearby logic, invariants, or intent: `Deallocates memory from the device.`.
  **L185 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Deallocates memory from the device.`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value genDeallocMemRef(OpBuilder &builder, Location loc, Value mem,`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value genDeallocMemRef(OpBuilder &builder, Location loc, Value mem,`。
- **L187 EN**: Continues the surrounding expression or declaration: `Value token) {`.
  **L187 CN**: 继续构造周围的表达式或声明：`Value token) {`。
- **L188 EN**: Returns from the current function with `gpu::DeallocOp::create(builder, loc, token.getType(), token, mem)`.
  **L188 CN**: 以 `gpu::DeallocOp::create(builder, loc, token.getType(), token, mem)` 从当前函数返回。
- **L189 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L189 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L190 EN**: Closes the current lexical scope or compound statement.
  **L190 CN**: 结束当前词法作用域或复合语句块。
- **L191 EN**: Blank line separating nearby declarations or logic blocks.
  **L191 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L192 EN**: Comment explains nearby logic, invariants, or intent: `Copies memory between host and device (direction is implicit).`.
  **L192 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copies memory between host and device (direction is implicit).`。

### Lines 193-216

````cpp
static Value genCopyMemRef(OpBuilder &builder, Location loc, Value dst,
                           Value src, Value token) {
  return gpu::MemcpyOp::create(builder, loc, token.getType(), token, dst, src)
      .getAsyncToken();
}

/// Generates an alloc/copy pair.
static Value genAllocCopy(OpBuilder &builder, Location loc, Value b,
                          SmallVectorImpl<Value> &tokens) {
  Value firstToken = genFirstWait(builder, loc);
  auto alloc = genAllocMemRef(builder, loc, b, firstToken);
  Value devMem = alloc.getResult(0);
  Value depToken = alloc.getAsyncToken(); // copy-after-alloc
  tokens.push_back(genCopyMemRef(builder, loc, devMem, b, depToken));
  return devMem;
}

/// Generates a memref from tensor operation.
static Value genTensorToMemref(PatternRewriter &rewriter, Location loc,
                               Value tensor) {
  auto tensorType = llvm::cast<ShapedType>(tensor.getType());
  auto memrefType =
      MemRefType::get(tensorType.getShape(), tensorType.getElementType());
  return bufferization::ToBufferOp::create(rewriter, loc, memrefType, tensor);
````
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value genCopyMemRef(OpBuilder &builder, Location loc, Value dst,`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value genCopyMemRef(OpBuilder &builder, Location loc, Value dst,`。
- **L194 EN**: Continues the surrounding expression or declaration: `Value src, Value token) {`.
  **L194 CN**: 继续构造周围的表达式或声明：`Value src, Value token) {`。
- **L195 EN**: Returns from the current function with `gpu::MemcpyOp::create(builder, loc, token.getType(), token, dst, src)`.
  **L195 CN**: 以 `gpu::MemcpyOp::create(builder, loc, token.getType(), token, dst, src)` 从当前函数返回。
- **L196 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L196 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L197 EN**: Closes the current lexical scope or compound statement.
  **L197 CN**: 结束当前词法作用域或复合语句块。
- **L198 EN**: Blank line separating nearby declarations or logic blocks.
  **L198 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L199 EN**: Comment explains nearby logic, invariants, or intent: `Generates an alloc/copy pair.`.
  **L199 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generates an alloc/copy pair.`。
- **L200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value genAllocCopy(OpBuilder &builder, Location loc, Value b,`.
  **L200 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value genAllocCopy(OpBuilder &builder, Location loc, Value b,`。
- **L201 EN**: Continues the surrounding expression or declaration: `SmallVectorImpl<Value> &tokens) {`.
  **L201 CN**: 继续构造周围的表达式或声明：`SmallVectorImpl<Value> &tokens) {`。
- **L202 EN**: Initializes variable `firstToken` from the right-hand expression.
  **L202 CN**: 使用右侧表达式初始化变量 `firstToken`。
- **L203 EN**: Initializes variable `alloc` from the right-hand expression.
  **L203 CN**: 使用右侧表达式初始化变量 `alloc`。
- **L204 EN**: Initializes variable `devMem` from the right-hand expression.
  **L204 CN**: 使用右侧表达式初始化变量 `devMem`。
- **L205 EN**: Continues logic associated with callable symbol `getAsyncToken`.
  **L205 CN**: 继续与可调用符号 `getAsyncToken` 相关的逻辑。
- **L206 EN**: Executes a call or declaration centered on `tokens.push_back`.
  **L206 CN**: 执行以 `tokens.push_back` 为核心的调用或声明。
- **L207 EN**: Returns from the current function with `devMem`.
  **L207 CN**: 以 `devMem` 从当前函数返回。
- **L208 EN**: Closes the current lexical scope or compound statement.
  **L208 CN**: 结束当前词法作用域或复合语句块。
- **L209 EN**: Blank line separating nearby declarations or logic blocks.
  **L209 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L210 EN**: Comment explains nearby logic, invariants, or intent: `Generates a memref from tensor operation.`.
  **L210 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generates a memref from tensor operation.`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value genTensorToMemref(PatternRewriter &rewriter, Location loc,`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value genTensorToMemref(PatternRewriter &rewriter, Location loc,`。
- **L212 EN**: Continues the surrounding expression or declaration: `Value tensor) {`.
  **L212 CN**: 继续构造周围的表达式或声明：`Value tensor) {`。
- **L213 EN**: Initializes variable `tensorType` from the right-hand expression.
  **L213 CN**: 使用右侧表达式初始化变量 `tensorType`。
- **L214 EN**: Continues the surrounding expression or declaration: `auto memrefType =`.
  **L214 CN**: 继续构造周围的表达式或声明：`auto memrefType =`。
- **L215 EN**: Executes a call or declaration centered on `MemRefType::get`.
  **L215 CN**: 执行以 `MemRefType::get` 为核心的调用或声明。
- **L216 EN**: Returns from the current function with `bufferization::ToBufferOp::create(rewriter, loc, memrefType, tensor)`.
  **L216 CN**: 以 `bufferization::ToBufferOp::create(rewriter, loc, memrefType, tensor)` 从当前函数返回。

### Lines 217-240

````cpp
}

/// Prepares the outlined arguments, passing scalars and buffers in. Here we
/// assume that the first buffer is the one allocated for output. We create
/// a set of properly chained asynchronous allocation/copy pairs to increase
/// overlap before launching the kernel.
static Value genParametersIn(OpBuilder &builder, Location loc,
                             SmallVectorImpl<Value> &scalars,
                             SmallVectorImpl<Value> &buffers,
                             SmallVectorImpl<Value> &args,
                             SmallVectorImpl<Value> &tokens,
                             bool useHostRegistrationForOut) {
  Value out;
  // Scalars are passed by value.
  for (Value s : scalars)
    args.push_back(s);
  // Buffers are need to be made visible on device.
  for (Value b : buffers) {
    if (useHostRegistrationForOut) {
      out = genHostRegisterMemref(builder, loc, b);
      args.push_back(b);
      useHostRegistrationForOut = false;
      continue;
    }
````
- **L217 EN**: Closes the current lexical scope or compound statement.
  **L217 CN**: 结束当前词法作用域或复合语句块。
- **L218 EN**: Blank line separating nearby declarations or logic blocks.
  **L218 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L219 EN**: Comment explains nearby logic, invariants, or intent: `Prepares the outlined arguments, passing scalars and buffers in. Here we`.
  **L219 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Prepares the outlined arguments, passing scalars and buffers in. Here we`。
- **L220 EN**: Comment explains nearby logic, invariants, or intent: `assume that the first buffer is the one allocated for output. We create`.
  **L220 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`assume that the first buffer is the one allocated for output. We create`。
- **L221 EN**: Comment explains nearby logic, invariants, or intent: `a set of properly chained asynchronous allocation/copy pairs to increase`.
  **L221 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a set of properly chained asynchronous allocation/copy pairs to increase`。
- **L222 EN**: Comment explains nearby logic, invariants, or intent: `overlap before launching the kernel.`.
  **L222 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`overlap before launching the kernel.`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value genParametersIn(OpBuilder &builder, Location loc,`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value genParametersIn(OpBuilder &builder, Location loc,`。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVectorImpl<Value> &scalars,`.
  **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVectorImpl<Value> &scalars,`。
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVectorImpl<Value> &buffers,`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVectorImpl<Value> &buffers,`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVectorImpl<Value> &args,`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVectorImpl<Value> &args,`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVectorImpl<Value> &tokens,`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVectorImpl<Value> &tokens,`。
- **L228 EN**: Continues the surrounding expression or declaration: `bool useHostRegistrationForOut) {`.
  **L228 CN**: 继续构造周围的表达式或声明：`bool useHostRegistrationForOut) {`。
- **L229 EN**: Executes a standalone statement or declaration: `Value out;`.
  **L229 CN**: 执行一条独立语句或声明：`Value out;`。
- **L230 EN**: Comment explains nearby logic, invariants, or intent: `Scalars are passed by value.`.
  **L230 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Scalars are passed by value.`。
- **L231 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L231 CN**: 开始 `for` 控制流语句并计算其条件。
- **L232 EN**: Executes a call or declaration centered on `args.push_back`.
  **L232 CN**: 执行以 `args.push_back` 为核心的调用或声明。
- **L233 EN**: Comment explains nearby logic, invariants, or intent: `Buffers are need to be made visible on device.`.
  **L233 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Buffers are need to be made visible on device.`。
- **L234 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L234 CN**: 开始 `for` 控制流语句并计算其条件。
- **L235 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L235 CN**: 开始 `if` 控制流语句并计算其条件。
- **L236 EN**: Executes a call or declaration centered on `genHostRegisterMemref`.
  **L236 CN**: 执行以 `genHostRegisterMemref` 为核心的调用或声明。
- **L237 EN**: Executes a call or declaration centered on `args.push_back`.
  **L237 CN**: 执行以 `args.push_back` 为核心的调用或声明。
- **L238 EN**: Executes a standalone statement or declaration: `useHostRegistrationForOut = false;`.
  **L238 CN**: 执行一条独立语句或声明：`useHostRegistrationForOut = false;`。
- **L239 EN**: Skips to the next loop iteration.
  **L239 CN**: 跳到下一次循环迭代。
- **L240 EN**: Closes the current lexical scope or compound statement.
  **L240 CN**: 结束当前词法作用域或复合语句块。

### Lines 241-264

````cpp
    args.push_back(genAllocCopy(builder, loc, b, tokens));
  }
  return out;
}

/// Finalizes the outlined arguments. The output buffer is copied depending
/// on the kernel token and then deallocated. All other buffers are simply
/// deallocated. Then we wait for all operations to complete.
///
/// `copyBack` maps 1:1 to the `buffers` array. It tracks which buffers were
/// mutated by the kernel and require a device-to-host copy. An empty
/// `copyBack` array implies no buffers are "copied back".
static void genParametersOut(OpBuilder &builder, Location loc, Value out,
                             Value kernelToken, SmallVectorImpl<Value> &scalars,
                             SmallVectorImpl<Value> &buffers,
                             SmallVectorImpl<Value> &args,
                             SmallVectorImpl<Value> &tokens,
                             ArrayRef<bool> copyBack) {
  unsigned base = scalars.size();

  // `args` stores scalars followed by buffers. `base` is the index of the first
  // buffer. `bufIdx` maps the current buffer to its exact 1:1 counterpart in
  // the `copyBack` mask.
  for (unsigned i = base, e = args.size(); i < e; i++) {
````
- **L241 EN**: Executes a call or declaration centered on `args.push_back`.
  **L241 CN**: 执行以 `args.push_back` 为核心的调用或声明。
- **L242 EN**: Closes the current lexical scope or compound statement.
  **L242 CN**: 结束当前词法作用域或复合语句块。
- **L243 EN**: Returns from the current function with `out`.
  **L243 CN**: 以 `out` 从当前函数返回。
- **L244 EN**: Closes the current lexical scope or compound statement.
  **L244 CN**: 结束当前词法作用域或复合语句块。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L246 EN**: Comment explains nearby logic, invariants, or intent: `Finalizes the outlined arguments. The output buffer is copied depending`.
  **L246 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Finalizes the outlined arguments. The output buffer is copied depending`。
- **L247 EN**: Comment explains nearby logic, invariants, or intent: `on the kernel token and then deallocated. All other buffers are simply`.
  **L247 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`on the kernel token and then deallocated. All other buffers are simply`。
- **L248 EN**: Comment explains nearby logic, invariants, or intent: `deallocated. Then we wait for all operations to complete.`.
  **L248 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`deallocated. Then we wait for all operations to complete.`。
- **L249 EN**: Separator comment used for visual grouping.
  **L249 CN**: 用于视觉分组的分隔注释。
- **L250 EN**: Comment explains nearby logic, invariants, or intent: ``copyBack` maps 1:1 to the `buffers` array. It tracks which buffers were`.
  **L250 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：``copyBack` maps 1:1 to the `buffers` array. It tracks which buffers were`。
- **L251 EN**: Comment explains nearby logic, invariants, or intent: `mutated by the kernel and require a device-to-host copy. An empty`.
  **L251 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mutated by the kernel and require a device-to-host copy. An empty`。
- **L252 EN**: Comment explains nearby logic, invariants, or intent: ``copyBack` array implies no buffers are "copied back".`.
  **L252 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：``copyBack` array implies no buffers are "copied back".`。
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void genParametersOut(OpBuilder &builder, Location loc, Value out,`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void genParametersOut(OpBuilder &builder, Location loc, Value out,`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value kernelToken, SmallVectorImpl<Value> &scalars,`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value kernelToken, SmallVectorImpl<Value> &scalars,`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVectorImpl<Value> &buffers,`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVectorImpl<Value> &buffers,`。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVectorImpl<Value> &args,`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVectorImpl<Value> &args,`。
- **L257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVectorImpl<Value> &tokens,`.
  **L257 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVectorImpl<Value> &tokens,`。
- **L258 EN**: Continues the surrounding expression or declaration: `ArrayRef<bool> copyBack) {`.
  **L258 CN**: 继续构造周围的表达式或声明：`ArrayRef<bool> copyBack) {`。
- **L259 EN**: Initializes variable `base` from the right-hand expression.
  **L259 CN**: 使用右侧表达式初始化变量 `base`。
- **L260 EN**: Blank line separating nearby declarations or logic blocks.
  **L260 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L261 EN**: Comment explains nearby logic, invariants, or intent: ``args` stores scalars followed by buffers. `base` is the index of the first`.
  **L261 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：``args` stores scalars followed by buffers. `base` is the index of the first`。
- **L262 EN**: Comment explains nearby logic, invariants, or intent: `buffer. `bufIdx` maps the current buffer to its exact 1:1 counterpart in`.
  **L262 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`buffer. `bufIdx` maps the current buffer to its exact 1:1 counterpart in`。
- **L263 EN**: Comment explains nearby logic, invariants, or intent: `the `copyBack` mask.`.
  **L263 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the `copyBack` mask.`。
- **L264 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L264 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 265-288

````cpp
    unsigned bufIdx = i - base;
    Value firstToken;

    // Checks if the current buffer needs a device-to-host copy.
    if (copyBack[bufIdx]) {
      if (out && bufIdx == 0) {
        genHostUnregisterMemref(builder, loc, out);
        out = Value();
        continue;
      }
      firstToken =
          genCopyMemRef(builder, loc, buffers[bufIdx], args[i], kernelToken);
    } else {
      firstToken = genFirstWait(builder, loc);
    }
    tokens.push_back(genDeallocMemRef(builder, loc, args[i], firstToken));
  }
}

/// Constructs code for new GPU kernel.
static void genGPUCode(PatternRewriter &rewriter, gpu::GPUFuncOp gpuFunc,
                       scf::ParallelOp forallOp,
                       SmallVectorImpl<Value> &constants,
                       SmallVectorImpl<Value> &scalars,
````
- **L265 EN**: Initializes variable `bufIdx` from the right-hand expression.
  **L265 CN**: 使用右侧表达式初始化变量 `bufIdx`。
- **L266 EN**: Executes a standalone statement or declaration: `Value firstToken;`.
  **L266 CN**: 执行一条独立语句或声明：`Value firstToken;`。
- **L267 EN**: Blank line separating nearby declarations or logic blocks.
  **L267 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L268 EN**: Comment explains nearby logic, invariants, or intent: `Checks if the current buffer needs a device-to-host copy.`.
  **L268 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Checks if the current buffer needs a device-to-host copy.`。
- **L269 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L269 CN**: 开始 `if` 控制流语句并计算其条件。
- **L270 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L270 CN**: 开始 `if` 控制流语句并计算其条件。
- **L271 EN**: Executes a call or declaration centered on `genHostUnregisterMemref`.
  **L271 CN**: 执行以 `genHostUnregisterMemref` 为核心的调用或声明。
- **L272 EN**: Executes a call or declaration centered on `Value`.
  **L272 CN**: 执行以 `Value` 为核心的调用或声明。
- **L273 EN**: Skips to the next loop iteration.
  **L273 CN**: 跳到下一次循环迭代。
- **L274 EN**: Closes the current lexical scope or compound statement.
  **L274 CN**: 结束当前词法作用域或复合语句块。
- **L275 EN**: Continues the surrounding expression or declaration: `firstToken =`.
  **L275 CN**: 继续构造周围的表达式或声明：`firstToken =`。
- **L276 EN**: Executes a call or declaration centered on `genCopyMemRef`.
  **L276 CN**: 执行以 `genCopyMemRef` 为核心的调用或声明。
- **L277 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L277 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L278 EN**: Executes a call or declaration centered on `genFirstWait`.
  **L278 CN**: 执行以 `genFirstWait` 为核心的调用或声明。
- **L279 EN**: Closes the current lexical scope or compound statement.
  **L279 CN**: 结束当前词法作用域或复合语句块。
- **L280 EN**: Executes a call or declaration centered on `tokens.push_back`.
  **L280 CN**: 执行以 `tokens.push_back` 为核心的调用或声明。
- **L281 EN**: Closes the current lexical scope or compound statement.
  **L281 CN**: 结束当前词法作用域或复合语句块。
- **L282 EN**: Closes the current lexical scope or compound statement.
  **L282 CN**: 结束当前词法作用域或复合语句块。
- **L283 EN**: Blank line separating nearby declarations or logic blocks.
  **L283 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L284 EN**: Comment explains nearby logic, invariants, or intent: `Constructs code for new GPU kernel.`.
  **L284 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Constructs code for new GPU kernel.`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static void genGPUCode(PatternRewriter &rewriter, gpu::GPUFuncOp gpuFunc,`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`static void genGPUCode(PatternRewriter &rewriter, gpu::GPUFuncOp gpuFunc,`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `scf::ParallelOp forallOp,`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`scf::ParallelOp forallOp,`。
- **L287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVectorImpl<Value> &constants,`.
  **L287 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVectorImpl<Value> &constants,`。
- **L288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SmallVectorImpl<Value> &scalars,`.
  **L288 CN**: 继续一个多行参数列表、初始化器或聚合项：`SmallVectorImpl<Value> &scalars,`。

### Lines 289-312

````cpp
                       SmallVectorImpl<Value> &buffers) {
  Location loc = gpuFunc->getLoc();
  Block &block = gpuFunc.getBody().front();
  rewriter.setInsertionPointToStart(&block);

  // Re-generate the constants, recapture all arguments.
  unsigned arg = 0;
  IRMapping irMap;
  for (Value c : constants)
    irMap.map(c, rewriter.clone(*c.getDefiningOp())->getResult(0));
  for (Value s : scalars)
    irMap.map(s, block.getArgument(arg++));
  for (Value b : buffers)
    irMap.map(b, block.getArgument(arg++));

  // Assume 1-dimensional grid/block configuration (only x dimension),
  // so that:
  //   row = blockIdx.x * blockDim.x + threadIdx.x
  //   inc = blockDim.x * gridDim.x
  Value bid = gpu::BlockIdOp::create(rewriter, loc, gpu::Dimension::x);
  Value bsz = gpu::BlockDimOp::create(rewriter, loc, gpu::Dimension::x);
  Value tid = gpu::ThreadIdOp::create(rewriter, loc, gpu::Dimension::x);
  Value gsz = gpu::GridDimOp::create(rewriter, loc, gpu::Dimension::x);
  Value mul = arith::MulIOp::create(rewriter, loc, bid, bsz);
````
- **L289 EN**: Continues the surrounding expression or declaration: `SmallVectorImpl<Value> &buffers) {`.
  **L289 CN**: 继续构造周围的表达式或声明：`SmallVectorImpl<Value> &buffers) {`。
- **L290 EN**: Initializes variable `loc` from the right-hand expression.
  **L290 CN**: 使用右侧表达式初始化变量 `loc`。
- **L291 EN**: Executes a call or declaration centered on `gpuFunc.getBody`.
  **L291 CN**: 执行以 `gpuFunc.getBody` 为核心的调用或声明。
- **L292 EN**: Executes a call or declaration centered on `rewriter.setInsertionPointToStart`.
  **L292 CN**: 执行以 `rewriter.setInsertionPointToStart` 为核心的调用或声明。
- **L293 EN**: Blank line separating nearby declarations or logic blocks.
  **L293 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L294 EN**: Comment explains nearby logic, invariants, or intent: `Re-generate the constants, recapture all arguments.`.
  **L294 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Re-generate the constants, recapture all arguments.`。
- **L295 EN**: Initializes variable `arg` from the right-hand expression.
  **L295 CN**: 使用右侧表达式初始化变量 `arg`。
- **L296 EN**: Executes a standalone statement or declaration: `IRMapping irMap;`.
  **L296 CN**: 执行一条独立语句或声明：`IRMapping irMap;`。
- **L297 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L297 CN**: 开始 `for` 控制流语句并计算其条件。
- **L298 EN**: Executes a call or declaration centered on `irMap.map`.
  **L298 CN**: 执行以 `irMap.map` 为核心的调用或声明。
- **L299 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L299 CN**: 开始 `for` 控制流语句并计算其条件。
- **L300 EN**: Executes a call or declaration centered on `irMap.map`.
  **L300 CN**: 执行以 `irMap.map` 为核心的调用或声明。
- **L301 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L301 CN**: 开始 `for` 控制流语句并计算其条件。
- **L302 EN**: Executes a call or declaration centered on `irMap.map`.
  **L302 CN**: 执行以 `irMap.map` 为核心的调用或声明。
- **L303 EN**: Blank line separating nearby declarations or logic blocks.
  **L303 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L304 EN**: Comment explains nearby logic, invariants, or intent: `Assume 1-dimensional grid/block configuration (only x dimension),`.
  **L304 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Assume 1-dimensional grid/block configuration (only x dimension),`。
- **L305 EN**: Comment explains nearby logic, invariants, or intent: `so that:`.
  **L305 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`so that:`。
- **L306 EN**: Comment explains nearby logic, invariants, or intent: `row = blockIdx.x * blockDim.x + threadIdx.x`.
  **L306 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`row = blockIdx.x * blockDim.x + threadIdx.x`。
- **L307 EN**: Comment explains nearby logic, invariants, or intent: `inc = blockDim.x * gridDim.x`.
  **L307 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`inc = blockDim.x * gridDim.x`。
- **L308 EN**: Initializes variable `bid` from the right-hand expression.
  **L308 CN**: 使用右侧表达式初始化变量 `bid`。
- **L309 EN**: Initializes variable `bsz` from the right-hand expression.
  **L309 CN**: 使用右侧表达式初始化变量 `bsz`。
- **L310 EN**: Initializes variable `tid` from the right-hand expression.
  **L310 CN**: 使用右侧表达式初始化变量 `tid`。
- **L311 EN**: Initializes variable `gsz` from the right-hand expression.
  **L311 CN**: 使用右侧表达式初始化变量 `gsz`。
- **L312 EN**: Initializes variable `mul` from the right-hand expression.
  **L312 CN**: 使用右侧表达式初始化变量 `mul`。

### Lines 313-336

````cpp
  Value row = arith::AddIOp::create(rewriter, loc, mul, tid);
  Value inc = arith::MulIOp::create(rewriter, loc, bsz, gsz);

  // Construct the iteration over the computational space that
  // accounts for the fact that the total number of threads and
  // the amount of work to be done usually do not match precisely.
  //   for (r = row; r < N; r += inc) {
  //     <loop-body>
  //   }
  Value upper = irMap.lookup(forallOp.getUpperBound()[0]);
  scf::ForOp forOp = scf::ForOp::create(rewriter, loc, row, upper, inc);
  // The scf.for builder creates an empty block. scf.for does not allow multiple
  // blocks in its region, so delete the block before `cloneRegionBefore` adds
  // an additional block.
  rewriter.eraseBlock(forOp.getBody());
  rewriter.cloneRegionBefore(forallOp.getRegion(), forOp.getRegion(),
                             forOp.getRegion().begin(), irMap);
  // Replace the scf.reduce terminator.
  rewriter.setInsertionPoint(forOp.getBody()->getTerminator());
  rewriter.replaceOpWithNewOp<scf::YieldOp>(forOp.getBody()->getTerminator());

  // Done.
  rewriter.setInsertionPointAfter(forOp);
  gpu::ReturnOp::create(rewriter, gpuFunc->getLoc());
````
- **L313 EN**: Initializes variable `row` from the right-hand expression.
  **L313 CN**: 使用右侧表达式初始化变量 `row`。
- **L314 EN**: Initializes variable `inc` from the right-hand expression.
  **L314 CN**: 使用右侧表达式初始化变量 `inc`。
- **L315 EN**: Blank line separating nearby declarations or logic blocks.
  **L315 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L316 EN**: Comment explains nearby logic, invariants, or intent: `Construct the iteration over the computational space that`.
  **L316 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Construct the iteration over the computational space that`。
- **L317 EN**: Comment explains nearby logic, invariants, or intent: `accounts for the fact that the total number of threads and`.
  **L317 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`accounts for the fact that the total number of threads and`。
- **L318 EN**: Comment explains nearby logic, invariants, or intent: `the amount of work to be done usually do not match precisely.`.
  **L318 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the amount of work to be done usually do not match precisely.`。
- **L319 EN**: Comment explains nearby logic, invariants, or intent: `for (r = row; r < N; r += inc) {`.
  **L319 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for (r = row; r < N; r += inc) {`。
- **L320 EN**: Comment explains nearby logic, invariants, or intent: `<loop-body>`.
  **L320 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<loop-body>`。
- **L321 EN**: Comment explains nearby logic, invariants, or intent: `}`.
  **L321 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`}`。
- **L322 EN**: Initializes variable `upper` from the right-hand expression.
  **L322 CN**: 使用右侧表达式初始化变量 `upper`。
- **L323 EN**: Initializes variable `forOp` from the right-hand expression.
  **L323 CN**: 使用右侧表达式初始化变量 `forOp`。
- **L324 EN**: Comment explains nearby logic, invariants, or intent: `The scf.for builder creates an empty block. scf.for does not allow multiple`.
  **L324 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The scf.for builder creates an empty block. scf.for does not allow multiple`。
- **L325 EN**: Comment explains nearby logic, invariants, or intent: `blocks in its region, so delete the block before `cloneRegionBefore` adds`.
  **L325 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`blocks in its region, so delete the block before `cloneRegionBefore` adds`。
- **L326 EN**: Comment explains nearby logic, invariants, or intent: `an additional block.`.
  **L326 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`an additional block.`。
- **L327 EN**: Executes a call or declaration centered on `rewriter.eraseBlock`.
  **L327 CN**: 执行以 `rewriter.eraseBlock` 为核心的调用或声明。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter.cloneRegionBefore(forallOp.getRegion(), forOp.getRegion(),`.
  **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter.cloneRegionBefore(forallOp.getRegion(), forOp.getRegion(),`。
- **L329 EN**: Executes a call or declaration centered on `forOp.getRegion`.
  **L329 CN**: 执行以 `forOp.getRegion` 为核心的调用或声明。
- **L330 EN**: Comment explains nearby logic, invariants, or intent: `Replace the scf.reduce terminator.`.
  **L330 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Replace the scf.reduce terminator.`。
- **L331 EN**: Executes a call or declaration centered on `rewriter.setInsertionPoint`.
  **L331 CN**: 执行以 `rewriter.setInsertionPoint` 为核心的调用或声明。
- **L332 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithNewOp<scf::YieldOp>`.
  **L332 CN**: 执行以 `rewriter.replaceOpWithNewOp<scf::YieldOp>` 为核心的调用或声明。
- **L333 EN**: Blank line separating nearby declarations or logic blocks.
  **L333 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L334 EN**: Comment explains nearby logic, invariants, or intent: `Done.`.
  **L334 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Done.`。
- **L335 EN**: Executes a call or declaration centered on `rewriter.setInsertionPointAfter`.
  **L335 CN**: 执行以 `rewriter.setInsertionPointAfter` 为核心的调用或声明。
- **L336 EN**: Executes a call or declaration centered on `gpu::ReturnOp::create`.
  **L336 CN**: 执行以 `gpu::ReturnOp::create` 为核心的调用或声明。

### Lines 337-360

````cpp
}

//===----------------------------------------------------------------------===//
// Library helper methods.
//===----------------------------------------------------------------------===//

/// Helper to detect a + b with arguments taken from given block.
static bool matchAddOfArgs(Block *block, Value val) {
  if (auto *def = val.getDefiningOp()) {
    if (isa<arith::AddFOp, arith::AddIOp>(def)) {
      Value a = block->getArguments()[0];
      Value b = block->getArguments()[1];
      return (def->getOperand(0) == a && def->getOperand(1) == b) ||
             (def->getOperand(0) == b && def->getOperand(1) == a);
    }
  }
  return false;
}

/// Helper to detect a * b with arguments taken from given block.
static bool matchMulOfArgs(Block *block, Value val) {
  if (auto *def = val.getDefiningOp()) {
    if (isa<arith::MulFOp, arith::MulIOp>(def)) {
      Value a = block->getArguments()[0];
````
- **L337 EN**: Closes the current lexical scope or compound statement.
  **L337 CN**: 结束当前词法作用域或复合语句块。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L339 EN**: Banner comment marking a file or section boundary.
  **L339 CN**: 横幅注释，用于标记文件或章节边界。
- **L340 EN**: Comment explains nearby logic, invariants, or intent: `Library helper methods.`.
  **L340 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Library helper methods.`。
- **L341 EN**: Banner comment marking a file or section boundary.
  **L341 CN**: 横幅注释，用于标记文件或章节边界。
- **L342 EN**: Blank line separating nearby declarations or logic blocks.
  **L342 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L343 EN**: Comment explains nearby logic, invariants, or intent: `Helper to detect a + b with arguments taken from given block.`.
  **L343 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper to detect a + b with arguments taken from given block.`。
- **L344 EN**: Starts a function, method, lambda, or structured scope: `static bool matchAddOfArgs(Block *block, Value val) {`.
  **L344 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool matchAddOfArgs(Block *block, Value val) {`。
- **L345 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L345 CN**: 开始 `if` 控制流语句并计算其条件。
- **L346 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L346 CN**: 开始 `if` 控制流语句并计算其条件。
- **L347 EN**: Initializes variable `a` from the right-hand expression.
  **L347 CN**: 使用右侧表达式初始化变量 `a`。
- **L348 EN**: Initializes variable `b` from the right-hand expression.
  **L348 CN**: 使用右侧表达式初始化变量 `b`。
- **L349 EN**: Returns from the current function with `(def->getOperand(0) == a && def->getOperand(1) == b) ||`.
  **L349 CN**: 以 `(def->getOperand(0) == a && def->getOperand(1) == b) ||` 从当前函数返回。
- **L350 EN**: Executes a call or declaration centered on `statement`.
  **L350 CN**: 执行以 `statement` 为核心的调用或声明。
- **L351 EN**: Closes the current lexical scope or compound statement.
  **L351 CN**: 结束当前词法作用域或复合语句块。
- **L352 EN**: Closes the current lexical scope or compound statement.
  **L352 CN**: 结束当前词法作用域或复合语句块。
- **L353 EN**: Returns from the current function with `false`.
  **L353 CN**: 以 `false` 从当前函数返回。
- **L354 EN**: Closes the current lexical scope or compound statement.
  **L354 CN**: 结束当前词法作用域或复合语句块。
- **L355 EN**: Blank line separating nearby declarations or logic blocks.
  **L355 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L356 EN**: Comment explains nearby logic, invariants, or intent: `Helper to detect a * b with arguments taken from given block.`.
  **L356 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper to detect a * b with arguments taken from given block.`。
- **L357 EN**: Starts a function, method, lambda, or structured scope: `static bool matchMulOfArgs(Block *block, Value val) {`.
  **L357 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool matchMulOfArgs(Block *block, Value val) {`。
- **L358 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L358 CN**: 开始 `if` 控制流语句并计算其条件。
- **L359 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L359 CN**: 开始 `if` 控制流语句并计算其条件。
- **L360 EN**: Initializes variable `a` from the right-hand expression.
  **L360 CN**: 使用右侧表达式初始化变量 `a`。

### Lines 361-384

````cpp
      Value b = block->getArguments()[1];
      return (def->getOperand(0) == a && def->getOperand(1) == b) ||
             (def->getOperand(0) == b && def->getOperand(1) == a);
    }
  }
  return false;
}

/// Helper to detect x = x + a * b
static bool matchSumOfMultOfArgs(linalg::GenericOp op) {
  auto yieldOp = cast<linalg::YieldOp>(op.getRegion().front().getTerminator());
  if (auto *def = yieldOp.getOperand(0).getDefiningOp()) {
    if (isa<arith::AddFOp, arith::AddIOp>(def)) {
      Value x = op.getBlock()->getArguments()[2];
      return (def->getOperand(0) == x &&
              matchMulOfArgs(op.getBlock(), def->getOperand(1))) ||
             (def->getOperand(1) == x &&
              matchMulOfArgs(op.getBlock(), def->getOperand(0)));
    }
  }
  return false;
}

// Helper to detect c += spy(s) x (a * b)
````
- **L361 EN**: Initializes variable `b` from the right-hand expression.
  **L361 CN**: 使用右侧表达式初始化变量 `b`。
- **L362 EN**: Returns from the current function with `(def->getOperand(0) == a && def->getOperand(1) == b) ||`.
  **L362 CN**: 以 `(def->getOperand(0) == a && def->getOperand(1) == b) ||` 从当前函数返回。
- **L363 EN**: Executes a call or declaration centered on `statement`.
  **L363 CN**: 执行以 `statement` 为核心的调用或声明。
- **L364 EN**: Closes the current lexical scope or compound statement.
  **L364 CN**: 结束当前词法作用域或复合语句块。
- **L365 EN**: Closes the current lexical scope or compound statement.
  **L365 CN**: 结束当前词法作用域或复合语句块。
- **L366 EN**: Returns from the current function with `false`.
  **L366 CN**: 以 `false` 从当前函数返回。
- **L367 EN**: Closes the current lexical scope or compound statement.
  **L367 CN**: 结束当前词法作用域或复合语句块。
- **L368 EN**: Blank line separating nearby declarations or logic blocks.
  **L368 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L369 EN**: Comment explains nearby logic, invariants, or intent: `Helper to detect x = x + a * b`.
  **L369 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper to detect x = x + a * b`。
- **L370 EN**: Starts a function, method, lambda, or structured scope: `static bool matchSumOfMultOfArgs(linalg::GenericOp op) {`.
  **L370 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool matchSumOfMultOfArgs(linalg::GenericOp op) {`。
- **L371 EN**: Initializes variable `yieldOp` from the right-hand expression.
  **L371 CN**: 使用右侧表达式初始化变量 `yieldOp`。
- **L372 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L372 CN**: 开始 `if` 控制流语句并计算其条件。
- **L373 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L373 CN**: 开始 `if` 控制流语句并计算其条件。
- **L374 EN**: Initializes variable `x` from the right-hand expression.
  **L374 CN**: 使用右侧表达式初始化变量 `x`。
- **L375 EN**: Returns from the current function with `(def->getOperand(0) == x &&`.
  **L375 CN**: 以 `(def->getOperand(0) == x &&` 从当前函数返回。
- **L376 EN**: Continues logic associated with callable symbol `matchMulOfArgs`.
  **L376 CN**: 继续与可调用符号 `matchMulOfArgs` 相关的逻辑。
- **L377 EN**: Continues logic associated with callable symbol `getOperand`.
  **L377 CN**: 继续与可调用符号 `getOperand` 相关的逻辑。
- **L378 EN**: Executes a call or declaration centered on `matchMulOfArgs`.
  **L378 CN**: 执行以 `matchMulOfArgs` 为核心的调用或声明。
- **L379 EN**: Closes the current lexical scope or compound statement.
  **L379 CN**: 结束当前词法作用域或复合语句块。
- **L380 EN**: Closes the current lexical scope or compound statement.
  **L380 CN**: 结束当前词法作用域或复合语句块。
- **L381 EN**: Returns from the current function with `false`.
  **L381 CN**: 以 `false` 从当前函数返回。
- **L382 EN**: Closes the current lexical scope or compound statement.
  **L382 CN**: 结束当前词法作用域或复合语句块。
- **L383 EN**: Blank line separating nearby declarations or logic blocks.
  **L383 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L384 EN**: Comment explains nearby logic, invariants, or intent: `Helper to detect c += spy(s) x (a * b)`.
  **L384 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper to detect c += spy(s) x (a * b)`。

### Lines 385-408

````cpp
static bool matchSumReductionOfMulUnary(linalg::GenericOp op) {
  auto yieldOp = cast<linalg::YieldOp>(op.getRegion().front().getTerminator());
  // The linalg yields a custom reduce result.
  Value s_out = op.getBlock()->getArguments()[2];
  if (auto redOp =
          yieldOp.getOperand(0).getDefiningOp<sparse_tensor::ReduceOp>()) {
    // The reduce consumes the output.
    Value other;
    if (s_out == redOp->getOperand(0))
      other = redOp->getOperand(1);
    else if (s_out == redOp->getOperand(1))
      other = redOp->getOperand(0);
    else
      return false;
    // The reduce op also consumes an unary which also consumes the output
    // and does not define an absent value.
    if (auto unOp = other.getDefiningOp<sparse_tensor::UnaryOp>()) {
      if (s_out != unOp->getOperand(0) || !unOp.getAbsentRegion().empty())
        return false;
      // And the bodies are as expected.
      auto yieldUn = cast<sparse_tensor::YieldOp>(
          unOp.getRegion(0).front().getTerminator());
      auto yieldRed = cast<sparse_tensor::YieldOp>(
          redOp.getRegion().front().getTerminator());
````
- **L385 EN**: Starts a function, method, lambda, or structured scope: `static bool matchSumReductionOfMulUnary(linalg::GenericOp op) {`.
  **L385 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool matchSumReductionOfMulUnary(linalg::GenericOp op) {`。
- **L386 EN**: Initializes variable `yieldOp` from the right-hand expression.
  **L386 CN**: 使用右侧表达式初始化变量 `yieldOp`。
- **L387 EN**: Comment explains nearby logic, invariants, or intent: `The linalg yields a custom reduce result.`.
  **L387 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The linalg yields a custom reduce result.`。
- **L388 EN**: Initializes variable `s_out` from the right-hand expression.
  **L388 CN**: 使用右侧表达式初始化变量 `s_out`。
- **L389 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L389 CN**: 开始 `if` 控制流语句并计算其条件。
- **L390 EN**: Starts a function, method, lambda, or structured scope: `yieldOp.getOperand(0).getDefiningOp<sparse_tensor::ReduceOp>()) {`.
  **L390 CN**: 开始一个函数、方法、lambda 或结构化作用域：`yieldOp.getOperand(0).getDefiningOp<sparse_tensor::ReduceOp>()) {`。
- **L391 EN**: Comment explains nearby logic, invariants, or intent: `The reduce consumes the output.`.
  **L391 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The reduce consumes the output.`。
- **L392 EN**: Executes a standalone statement or declaration: `Value other;`.
  **L392 CN**: 执行一条独立语句或声明：`Value other;`。
- **L393 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L393 CN**: 开始 `if` 控制流语句并计算其条件。
- **L394 EN**: Executes a call or declaration centered on `redOp->getOperand`.
  **L394 CN**: 执行以 `redOp->getOperand` 为核心的调用或声明。
- **L395 EN**: Starts the alternative branch of the preceding conditional.
  **L395 CN**: 开始前一个条件语句的备选分支。
- **L396 EN**: Executes a call or declaration centered on `redOp->getOperand`.
  **L396 CN**: 执行以 `redOp->getOperand` 为核心的调用或声明。
- **L397 EN**: Starts the alternative branch of the preceding conditional.
  **L397 CN**: 开始前一个条件语句的备选分支。
- **L398 EN**: Returns from the current function with `false`.
  **L398 CN**: 以 `false` 从当前函数返回。
- **L399 EN**: Comment explains nearby logic, invariants, or intent: `The reduce op also consumes an unary which also consumes the output`.
  **L399 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The reduce op also consumes an unary which also consumes the output`。
- **L400 EN**: Comment explains nearby logic, invariants, or intent: `and does not define an absent value.`.
  **L400 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and does not define an absent value.`。
- **L401 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L401 CN**: 开始 `if` 控制流语句并计算其条件。
- **L402 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L402 CN**: 开始 `if` 控制流语句并计算其条件。
- **L403 EN**: Returns from the current function with `false`.
  **L403 CN**: 以 `false` 从当前函数返回。
- **L404 EN**: Comment explains nearby logic, invariants, or intent: `And the bodies are as expected.`.
  **L404 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`And the bodies are as expected.`。
- **L405 EN**: Continues logic associated with callable symbol `YieldOp>`.
  **L405 CN**: 继续与可调用符号 `YieldOp>` 相关的逻辑。
- **L406 EN**: Executes a call or declaration centered on `unOp.getRegion`.
  **L406 CN**: 执行以 `unOp.getRegion` 为核心的调用或声明。
- **L407 EN**: Continues logic associated with callable symbol `YieldOp>`.
  **L407 CN**: 继续与可调用符号 `YieldOp>` 相关的逻辑。
- **L408 EN**: Executes a call or declaration centered on `redOp.getRegion`.
  **L408 CN**: 执行以 `redOp.getRegion` 为核心的调用或声明。

### Lines 409-432

````cpp
      return matchMulOfArgs(op.getBlock(), yieldUn.getOperand(0)) &&
             matchAddOfArgs(&redOp.getRegion().front(), yieldRed.getOperand(0));
    }
  }
  return false;
}

/// Test for dense tensor.
static bool isDenseTensor(Value v) {
  auto sTp = getSparseTensorType(v);
  return sTp.getDimRank() == sTp.getLvlRank() && sTp.isAllDense();
}

/// Test for suitable positions/coordinates width.
static bool isAdmissibleMetaData(SparseTensorType &aTp) {
  return (aTp.getPosWidth() == 0 || aTp.getPosWidth() >= 16) &&
         (aTp.getCrdWidth() == 0 || aTp.getCrdWidth() >= 16);
}

/// Test for sorted COO matrix with suitable metadata.
static bool isAdmissibleCOO(SparseTensorType &aTp) {
  return aTp.getDimRank() == 2 && aTp.getLvlRank() == 2 && aTp.isIdentity() &&
         aTp.isCompressedLvl(0) && aTp.isOrderedLvl(0) && !aTp.isUniqueLvl(0) &&
         aTp.isSingletonLvl(1) && aTp.isOrderedLvl(1) && aTp.isUniqueLvl(1) &&
````
- **L409 EN**: Returns from the current function with `matchMulOfArgs(op.getBlock(), yieldUn.getOperand(0)) &&`.
  **L409 CN**: 以 `matchMulOfArgs(op.getBlock(), yieldUn.getOperand(0)) &&` 从当前函数返回。
- **L410 EN**: Executes a call or declaration centered on `matchAddOfArgs`.
  **L410 CN**: 执行以 `matchAddOfArgs` 为核心的调用或声明。
- **L411 EN**: Closes the current lexical scope or compound statement.
  **L411 CN**: 结束当前词法作用域或复合语句块。
- **L412 EN**: Closes the current lexical scope or compound statement.
  **L412 CN**: 结束当前词法作用域或复合语句块。
- **L413 EN**: Returns from the current function with `false`.
  **L413 CN**: 以 `false` 从当前函数返回。
- **L414 EN**: Closes the current lexical scope or compound statement.
  **L414 CN**: 结束当前词法作用域或复合语句块。
- **L415 EN**: Blank line separating nearby declarations or logic blocks.
  **L415 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L416 EN**: Comment explains nearby logic, invariants, or intent: `Test for dense tensor.`.
  **L416 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Test for dense tensor.`。
- **L417 EN**: Starts a function, method, lambda, or structured scope: `static bool isDenseTensor(Value v) {`.
  **L417 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isDenseTensor(Value v) {`。
- **L418 EN**: Initializes variable `sTp` from the right-hand expression.
  **L418 CN**: 使用右侧表达式初始化变量 `sTp`。
- **L419 EN**: Returns from the current function with `sTp.getDimRank() == sTp.getLvlRank() && sTp.isAllDense()`.
  **L419 CN**: 以 `sTp.getDimRank() == sTp.getLvlRank() && sTp.isAllDense()` 从当前函数返回。
- **L420 EN**: Closes the current lexical scope or compound statement.
  **L420 CN**: 结束当前词法作用域或复合语句块。
- **L421 EN**: Blank line separating nearby declarations or logic blocks.
  **L421 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L422 EN**: Comment explains nearby logic, invariants, or intent: `Test for suitable positions/coordinates width.`.
  **L422 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Test for suitable positions/coordinates width.`。
- **L423 EN**: Starts a function, method, lambda, or structured scope: `static bool isAdmissibleMetaData(SparseTensorType &aTp) {`.
  **L423 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isAdmissibleMetaData(SparseTensorType &aTp) {`。
- **L424 EN**: Returns from the current function with `(aTp.getPosWidth() == 0 || aTp.getPosWidth() >= 16) &&`.
  **L424 CN**: 以 `(aTp.getPosWidth() == 0 || aTp.getPosWidth() >= 16) &&` 从当前函数返回。
- **L425 EN**: Executes a call or declaration centered on `statement`.
  **L425 CN**: 执行以 `statement` 为核心的调用或声明。
- **L426 EN**: Closes the current lexical scope or compound statement.
  **L426 CN**: 结束当前词法作用域或复合语句块。
- **L427 EN**: Blank line separating nearby declarations or logic blocks.
  **L427 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L428 EN**: Comment explains nearby logic, invariants, or intent: `Test for sorted COO matrix with suitable metadata.`.
  **L428 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Test for sorted COO matrix with suitable metadata.`。
- **L429 EN**: Starts a function, method, lambda, or structured scope: `static bool isAdmissibleCOO(SparseTensorType &aTp) {`.
  **L429 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isAdmissibleCOO(SparseTensorType &aTp) {`。
- **L430 EN**: Returns from the current function with `aTp.getDimRank() == 2 && aTp.getLvlRank() == 2 && aTp.isIdentity() &&`.
  **L430 CN**: 以 `aTp.getDimRank() == 2 && aTp.getLvlRank() == 2 && aTp.isIdentity() &&` 从当前函数返回。
- **L431 EN**: Continues logic associated with callable symbol `isCompressedLvl`.
  **L431 CN**: 继续与可调用符号 `isCompressedLvl` 相关的逻辑。
- **L432 EN**: Continues logic associated with callable symbol `isSingletonLvl`.
  **L432 CN**: 继续与可调用符号 `isSingletonLvl` 相关的逻辑。

### Lines 433-456

````cpp
         isAdmissibleMetaData(aTp);
}

/// Test for CSR matrix with suitable metadata.
static bool isAdmissibleCSR(SparseTensorType &aTp) {
  return aTp.getDimRank() == 2 && aTp.getLvlRank() == 2 && aTp.isIdentity() &&
         aTp.isDenseLvl(0) && aTp.isCompressedLvl(1) && aTp.isOrderedLvl(1) &&
         aTp.isUniqueLvl(1) && isAdmissibleMetaData(aTp);
}

/// Test for CSC matrix with suitable metadata.
static bool isAdmissibleCSC(SparseTensorType &aTp) {
  return aTp.getDimRank() == 2 && aTp.getLvlRank() == 2 && !aTp.isIdentity() &&
         aTp.isPermutation() && aTp.isDenseLvl(0) && aTp.isCompressedLvl(1) &&
         aTp.isOrderedLvl(1) && aTp.isUniqueLvl(1) && isAdmissibleMetaData(aTp);
}

/// Test for BSR matrix with suitable metadata.
static bool isAdmissibleBSR(SparseTensorType &aTp) {
  if (aTp.getDimRank() == 2 && aTp.getLvlRank() == 4 && aTp.isDenseLvl(0) &&
      aTp.isCompressedLvl(1) && aTp.isOrderedLvl(1) && aTp.isUniqueLvl(1) &&
      aTp.isDenseLvl(2) && aTp.isDenseLvl(3) && isAdmissibleMetaData(aTp)) {
    // CuSparse only supports "square" blocks currently.
    SmallVector<unsigned> dims = getBlockSize(aTp.getDimToLvl());
````
- **L433 EN**: Executes a call or declaration centered on `isAdmissibleMetaData`.
  **L433 CN**: 执行以 `isAdmissibleMetaData` 为核心的调用或声明。
- **L434 EN**: Closes the current lexical scope or compound statement.
  **L434 CN**: 结束当前词法作用域或复合语句块。
- **L435 EN**: Blank line separating nearby declarations or logic blocks.
  **L435 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L436 EN**: Comment explains nearby logic, invariants, or intent: `Test for CSR matrix with suitable metadata.`.
  **L436 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Test for CSR matrix with suitable metadata.`。
- **L437 EN**: Starts a function, method, lambda, or structured scope: `static bool isAdmissibleCSR(SparseTensorType &aTp) {`.
  **L437 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isAdmissibleCSR(SparseTensorType &aTp) {`。
- **L438 EN**: Returns from the current function with `aTp.getDimRank() == 2 && aTp.getLvlRank() == 2 && aTp.isIdentity() &&`.
  **L438 CN**: 以 `aTp.getDimRank() == 2 && aTp.getLvlRank() == 2 && aTp.isIdentity() &&` 从当前函数返回。
- **L439 EN**: Continues logic associated with callable symbol `isDenseLvl`.
  **L439 CN**: 继续与可调用符号 `isDenseLvl` 相关的逻辑。
- **L440 EN**: Executes a call or declaration centered on `aTp.isUniqueLvl`.
  **L440 CN**: 执行以 `aTp.isUniqueLvl` 为核心的调用或声明。
- **L441 EN**: Closes the current lexical scope or compound statement.
  **L441 CN**: 结束当前词法作用域或复合语句块。
- **L442 EN**: Blank line separating nearby declarations or logic blocks.
  **L442 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L443 EN**: Comment explains nearby logic, invariants, or intent: `Test for CSC matrix with suitable metadata.`.
  **L443 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Test for CSC matrix with suitable metadata.`。
- **L444 EN**: Starts a function, method, lambda, or structured scope: `static bool isAdmissibleCSC(SparseTensorType &aTp) {`.
  **L444 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isAdmissibleCSC(SparseTensorType &aTp) {`。
- **L445 EN**: Returns from the current function with `aTp.getDimRank() == 2 && aTp.getLvlRank() == 2 && !aTp.isIdentity() &&`.
  **L445 CN**: 以 `aTp.getDimRank() == 2 && aTp.getLvlRank() == 2 && !aTp.isIdentity() &&` 从当前函数返回。
- **L446 EN**: Continues logic associated with callable symbol `isPermutation`.
  **L446 CN**: 继续与可调用符号 `isPermutation` 相关的逻辑。
- **L447 EN**: Executes a call or declaration centered on `aTp.isOrderedLvl`.
  **L447 CN**: 执行以 `aTp.isOrderedLvl` 为核心的调用或声明。
- **L448 EN**: Closes the current lexical scope or compound statement.
  **L448 CN**: 结束当前词法作用域或复合语句块。
- **L449 EN**: Blank line separating nearby declarations or logic blocks.
  **L449 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L450 EN**: Comment explains nearby logic, invariants, or intent: `Test for BSR matrix with suitable metadata.`.
  **L450 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Test for BSR matrix with suitable metadata.`。
- **L451 EN**: Starts a function, method, lambda, or structured scope: `static bool isAdmissibleBSR(SparseTensorType &aTp) {`.
  **L451 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isAdmissibleBSR(SparseTensorType &aTp) {`。
- **L452 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L452 CN**: 开始 `if` 控制流语句并计算其条件。
- **L453 EN**: Continues logic associated with callable symbol `isCompressedLvl`.
  **L453 CN**: 继续与可调用符号 `isCompressedLvl` 相关的逻辑。
- **L454 EN**: Starts a function, method, lambda, or structured scope: `aTp.isDenseLvl(2) && aTp.isDenseLvl(3) && isAdmissibleMetaData(aTp)) {`.
  **L454 CN**: 开始一个函数、方法、lambda 或结构化作用域：`aTp.isDenseLvl(2) && aTp.isDenseLvl(3) && isAdmissibleMetaData(aTp)) {`。
- **L455 EN**: Comment explains nearby logic, invariants, or intent: `CuSparse only supports "square" blocks currently.`.
  **L455 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`CuSparse only supports "square" blocks currently.`。
- **L456 EN**: Initializes variable `dims` from the right-hand expression.
  **L456 CN**: 使用右侧表达式初始化变量 `dims`。

### Lines 457-480

````cpp
    assert(dims.size() == 2);
    return dims[0] == dims[1] && dims[0] > 1;
  }
  return false;
}

/// Test for 2:4 matrix with suitable metadata.
static bool isAdmissible24(SparseTensorType &aTp) {
  return aTp.getDimRank() == 2 && aTp.getLvlRank() == 3 && aTp.isDenseLvl(0) &&
         aTp.isDenseLvl(1) && aTp.isNOutOfMLvl(2) && isAdmissibleMetaData(aTp);
}

/// Test for conversion into 2:4 matrix.
static bool isConversionInto24(Value v) {
  if (auto cnv = v.getDefiningOp<ConvertOp>()) {
    Value a = cnv.getResult();
    Value d = cnv.getSource();
    SparseTensorType aTp = getSparseTensorType(a);
    return isDenseTensor(d) && isAdmissible24(aTp);
  }
  return false;
}

/// Returns a suitable sparse format for the operation and given operand
````
- **L457 EN**: Checks an internal invariant in debug builds.
  **L457 CN**: 在调试构建中检查内部不变式。
- **L458 EN**: Returns from the current function with `dims[0] == dims[1] && dims[0] > 1`.
  **L458 CN**: 以 `dims[0] == dims[1] && dims[0] > 1` 从当前函数返回。
- **L459 EN**: Closes the current lexical scope or compound statement.
  **L459 CN**: 结束当前词法作用域或复合语句块。
- **L460 EN**: Returns from the current function with `false`.
  **L460 CN**: 以 `false` 从当前函数返回。
- **L461 EN**: Closes the current lexical scope or compound statement.
  **L461 CN**: 结束当前词法作用域或复合语句块。
- **L462 EN**: Blank line separating nearby declarations or logic blocks.
  **L462 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L463 EN**: Comment explains nearby logic, invariants, or intent: `Test for 2:4 matrix with suitable metadata.`.
  **L463 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Test for 2:4 matrix with suitable metadata.`。
- **L464 EN**: Starts a function, method, lambda, or structured scope: `static bool isAdmissible24(SparseTensorType &aTp) {`.
  **L464 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isAdmissible24(SparseTensorType &aTp) {`。
- **L465 EN**: Returns from the current function with `aTp.getDimRank() == 2 && aTp.getLvlRank() == 3 && aTp.isDenseLvl(0) &&`.
  **L465 CN**: 以 `aTp.getDimRank() == 2 && aTp.getLvlRank() == 3 && aTp.isDenseLvl(0) &&` 从当前函数返回。
- **L466 EN**: Executes a call or declaration centered on `aTp.isDenseLvl`.
  **L466 CN**: 执行以 `aTp.isDenseLvl` 为核心的调用或声明。
- **L467 EN**: Closes the current lexical scope or compound statement.
  **L467 CN**: 结束当前词法作用域或复合语句块。
- **L468 EN**: Blank line separating nearby declarations or logic blocks.
  **L468 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L469 EN**: Comment explains nearby logic, invariants, or intent: `Test for conversion into 2:4 matrix.`.
  **L469 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Test for conversion into 2:4 matrix.`。
- **L470 EN**: Starts a function, method, lambda, or structured scope: `static bool isConversionInto24(Value v) {`.
  **L470 CN**: 开始一个函数、方法、lambda 或结构化作用域：`static bool isConversionInto24(Value v) {`。
- **L471 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L471 CN**: 开始 `if` 控制流语句并计算其条件。
- **L472 EN**: Initializes variable `a` from the right-hand expression.
  **L472 CN**: 使用右侧表达式初始化变量 `a`。
- **L473 EN**: Initializes variable `d` from the right-hand expression.
  **L473 CN**: 使用右侧表达式初始化变量 `d`。
- **L474 EN**: Initializes variable `aTp` from the right-hand expression.
  **L474 CN**: 使用右侧表达式初始化变量 `aTp`。
- **L475 EN**: Returns from the current function with `isDenseTensor(d) && isAdmissible24(aTp)`.
  **L475 CN**: 以 `isDenseTensor(d) && isAdmissible24(aTp)` 从当前函数返回。
- **L476 EN**: Closes the current lexical scope or compound statement.
  **L476 CN**: 结束当前词法作用域或复合语句块。
- **L477 EN**: Returns from the current function with `false`.
  **L477 CN**: 以 `false` 从当前函数返回。
- **L478 EN**: Closes the current lexical scope or compound statement.
  **L478 CN**: 结束当前词法作用域或复合语句块。
- **L479 EN**: Blank line separating nearby declarations or logic blocks.
  **L479 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L480 EN**: Comment explains nearby logic, invariants, or intent: `Returns a suitable sparse format for the operation and given operand`.
  **L480 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns a suitable sparse format for the operation and given operand`。

### Lines 481-504

````cpp
/// types with cuSparse, or kNone if none is available.
static CuSparseFormat getCuSparseFormat(SparseTensorType aTp,
                                        SparseTensorType bTp,
                                        SparseTensorType cTp, bool enableRT,
                                        bool isMatVec) {
  // The other operands have a dense type.
  if (bTp.hasEncoding() || cTp.hasEncoding())
    return CuSparseFormat::kNone;
  // Now check for suitable operand type for the main operand.
  if (isAdmissibleCOO(aTp))
#ifdef CUSPARSE_COO_AOS
    return isMatVec ? CuSparseFormat::kCOO : CuSparseFormat::kNone;
#else
    return enableRT ? CuSparseFormat::kCOO : CuSparseFormat::kNone;
#endif
  if (isAdmissibleCSR(aTp))
    return CuSparseFormat::kCSR;
  if (isAdmissibleCSC(aTp))
    return CuSparseFormat::kCSC;
  if (isAdmissibleBSR(aTp))
    return CuSparseFormat::kBSR;
  return CuSparseFormat::kNone;
}

````
- **L481 EN**: Comment explains nearby logic, invariants, or intent: `types with cuSparse, or kNone if none is available.`.
  **L481 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`types with cuSparse, or kNone if none is available.`。
- **L482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static CuSparseFormat getCuSparseFormat(SparseTensorType aTp,`.
  **L482 CN**: 继续一个多行参数列表、初始化器或聚合项：`static CuSparseFormat getCuSparseFormat(SparseTensorType aTp,`。
- **L483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SparseTensorType bTp,`.
  **L483 CN**: 继续一个多行参数列表、初始化器或聚合项：`SparseTensorType bTp,`。
- **L484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SparseTensorType cTp, bool enableRT,`.
  **L484 CN**: 继续一个多行参数列表、初始化器或聚合项：`SparseTensorType cTp, bool enableRT,`。
- **L485 EN**: Continues the surrounding expression or declaration: `bool isMatVec) {`.
  **L485 CN**: 继续构造周围的表达式或声明：`bool isMatVec) {`。
- **L486 EN**: Comment explains nearby logic, invariants, or intent: `The other operands have a dense type.`.
  **L486 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The other operands have a dense type.`。
- **L487 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L487 CN**: 开始 `if` 控制流语句并计算其条件。
- **L488 EN**: Returns from the current function with `CuSparseFormat::kNone`.
  **L488 CN**: 以 `CuSparseFormat::kNone` 从当前函数返回。
- **L489 EN**: Comment explains nearby logic, invariants, or intent: `Now check for suitable operand type for the main operand.`.
  **L489 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Now check for suitable operand type for the main operand.`。
- **L490 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L490 CN**: 开始 `if` 控制流语句并计算其条件。
- **L491 EN**: Starts a preprocessor conditional block: `#ifdef CUSPARSE_COO_AOS`.
  **L491 CN**: 开始一个预处理条件块：`#ifdef CUSPARSE_COO_AOS`。
- **L492 EN**: Returns from the current function with `isMatVec ? CuSparseFormat::kCOO : CuSparseFormat::kNone`.
  **L492 CN**: 以 `isMatVec ? CuSparseFormat::kCOO : CuSparseFormat::kNone` 从当前函数返回。
- **L493 EN**: Continues the active preprocessor branch selection.
  **L493 CN**: 继续当前的预处理分支选择。
- **L494 EN**: Returns from the current function with `enableRT ? CuSparseFormat::kCOO : CuSparseFormat::kNone`.
  **L494 CN**: 以 `enableRT ? CuSparseFormat::kCOO : CuSparseFormat::kNone` 从当前函数返回。
- **L495 EN**: Closes the current preprocessor conditional block.
  **L495 CN**: 结束当前预处理条件块。
- **L496 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L496 CN**: 开始 `if` 控制流语句并计算其条件。
- **L497 EN**: Returns from the current function with `CuSparseFormat::kCSR`.
  **L497 CN**: 以 `CuSparseFormat::kCSR` 从当前函数返回。
- **L498 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L498 CN**: 开始 `if` 控制流语句并计算其条件。
- **L499 EN**: Returns from the current function with `CuSparseFormat::kCSC`.
  **L499 CN**: 以 `CuSparseFormat::kCSC` 从当前函数返回。
- **L500 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L500 CN**: 开始 `if` 控制流语句并计算其条件。
- **L501 EN**: Returns from the current function with `CuSparseFormat::kBSR`.
  **L501 CN**: 以 `CuSparseFormat::kBSR` 从当前函数返回。
- **L502 EN**: Returns from the current function with `CuSparseFormat::kNone`.
  **L502 CN**: 以 `CuSparseFormat::kNone` 从当前函数返回。
- **L503 EN**: Closes the current lexical scope or compound statement.
  **L503 CN**: 结束当前词法作用域或复合语句块。
- **L504 EN**: Blank line separating nearby declarations or logic blocks.
  **L504 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 505-528

````cpp
/// Generates the first positions/coordinates of a sparse matrix.
static Value genFirstPosOrCrds(OpBuilder &builder, Location loc, Value a,
                               CuSparseFormat format, bool enableRT) {
  if (format == CuSparseFormat::kCOO) {
    // Library uses SoA COO, direct IR uses AoS COO.
    if (enableRT)
      return ToCoordinatesOp::create(builder, loc, a, 0);
    return ToCoordinatesBufferOp::create(builder, loc, a);
  }
  // Formats CSR/CSC and BSR use positions at 1.
  return ToPositionsOp::create(builder, loc, a, 1);
}

/// Generates the second coordinates of a sparse matrix.
static Value genSecondCrds(OpBuilder &builder, Location loc, Value a,
                           CuSparseFormat format, bool enableRT) {
  bool isCOO = format == CuSparseFormat::kCOO;
  if (isCOO && !enableRT)
    return Value(); // nothing needed
  // Formats CSR/CSC and BSR use coordinates at 1.
  return ToCoordinatesOp::create(builder, loc, a, 1);
}

/// Generates the sparse matrix handle.
````
- **L505 EN**: Comment explains nearby logic, invariants, or intent: `Generates the first positions/coordinates of a sparse matrix.`.
  **L505 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generates the first positions/coordinates of a sparse matrix.`。
- **L506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value genFirstPosOrCrds(OpBuilder &builder, Location loc, Value a,`.
  **L506 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value genFirstPosOrCrds(OpBuilder &builder, Location loc, Value a,`。
- **L507 EN**: Continues the surrounding expression or declaration: `CuSparseFormat format, bool enableRT) {`.
  **L507 CN**: 继续构造周围的表达式或声明：`CuSparseFormat format, bool enableRT) {`。
- **L508 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L508 CN**: 开始 `if` 控制流语句并计算其条件。
- **L509 EN**: Comment explains nearby logic, invariants, or intent: `Library uses SoA COO, direct IR uses AoS COO.`.
  **L509 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Library uses SoA COO, direct IR uses AoS COO.`。
- **L510 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L510 CN**: 开始 `if` 控制流语句并计算其条件。
- **L511 EN**: Returns from the current function with `ToCoordinatesOp::create(builder, loc, a, 0)`.
  **L511 CN**: 以 `ToCoordinatesOp::create(builder, loc, a, 0)` 从当前函数返回。
- **L512 EN**: Returns from the current function with `ToCoordinatesBufferOp::create(builder, loc, a)`.
  **L512 CN**: 以 `ToCoordinatesBufferOp::create(builder, loc, a)` 从当前函数返回。
- **L513 EN**: Closes the current lexical scope or compound statement.
  **L513 CN**: 结束当前词法作用域或复合语句块。
- **L514 EN**: Comment explains nearby logic, invariants, or intent: `Formats CSR/CSC and BSR use positions at 1.`.
  **L514 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Formats CSR/CSC and BSR use positions at 1.`。
- **L515 EN**: Returns from the current function with `ToPositionsOp::create(builder, loc, a, 1)`.
  **L515 CN**: 以 `ToPositionsOp::create(builder, loc, a, 1)` 从当前函数返回。
- **L516 EN**: Closes the current lexical scope or compound statement.
  **L516 CN**: 结束当前词法作用域或复合语句块。
- **L517 EN**: Blank line separating nearby declarations or logic blocks.
  **L517 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L518 EN**: Comment explains nearby logic, invariants, or intent: `Generates the second coordinates of a sparse matrix.`.
  **L518 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generates the second coordinates of a sparse matrix.`。
- **L519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Value genSecondCrds(OpBuilder &builder, Location loc, Value a,`.
  **L519 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Value genSecondCrds(OpBuilder &builder, Location loc, Value a,`。
- **L520 EN**: Continues the surrounding expression or declaration: `CuSparseFormat format, bool enableRT) {`.
  **L520 CN**: 继续构造周围的表达式或声明：`CuSparseFormat format, bool enableRT) {`。
- **L521 EN**: Initializes variable `isCOO` from the right-hand expression.
  **L521 CN**: 使用右侧表达式初始化变量 `isCOO`。
- **L522 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L522 CN**: 开始 `if` 控制流语句并计算其条件。
- **L523 EN**: Returns from the current function with `Value(); // nothing needed`.
  **L523 CN**: 以 `Value(); // nothing needed` 从当前函数返回。
- **L524 EN**: Comment explains nearby logic, invariants, or intent: `Formats CSR/CSC and BSR use coordinates at 1.`.
  **L524 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Formats CSR/CSC and BSR use coordinates at 1.`。
- **L525 EN**: Returns from the current function with `ToCoordinatesOp::create(builder, loc, a, 1)`.
  **L525 CN**: 以 `ToCoordinatesOp::create(builder, loc, a, 1)` 从当前函数返回。
- **L526 EN**: Closes the current lexical scope or compound statement.
  **L526 CN**: 结束当前词法作用域或复合语句块。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L528 EN**: Comment explains nearby logic, invariants, or intent: `Generates the sparse matrix handle.`.
  **L528 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generates the sparse matrix handle.`。

### Lines 529-552

````cpp
static Operation *genSpMat(OpBuilder &builder, Location loc,
                           SparseTensorType &aTp, Type handleTp, Type tokenTp,
                           Value token, Value sz1, Value sz2, Value nseA,
                           Value rowA, Value colA, Value valA,
                           CuSparseFormat format, bool enableRT) {
  if (format == CuSparseFormat::kCOO) {
    // Library uses SoA COO, direct IR uses AoS COO.
    if (enableRT) {
      assert(colA);
      return gpu::CreateCooOp::create(builder, loc, handleTp, tokenTp, token,
                                      sz1, sz2, nseA, rowA, colA, valA);
    }
#ifdef CUSPARSE_COO_AOS
    assert(!colA);
    return gpu::CreateCooAoSOp::create(builder, loc, handleTp, tokenTp, token,
                                       sz1, sz2, nseA, rowA, valA);
#else
    llvm_unreachable("gpu::CreateCooAoSOp is deprecated");
#endif
  }
  assert(colA);
  if (format == CuSparseFormat::kCSR)
    return gpu::CreateCsrOp::create(builder, loc, handleTp, tokenTp, token, sz1,
                                    sz2, nseA, rowA, colA, valA);
````
- **L529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static Operation *genSpMat(OpBuilder &builder, Location loc,`.
  **L529 CN**: 继续一个多行参数列表、初始化器或聚合项：`static Operation *genSpMat(OpBuilder &builder, Location loc,`。
- **L530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SparseTensorType &aTp, Type handleTp, Type tokenTp,`.
  **L530 CN**: 继续一个多行参数列表、初始化器或聚合项：`SparseTensorType &aTp, Type handleTp, Type tokenTp,`。
- **L531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value token, Value sz1, Value sz2, Value nseA,`.
  **L531 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value token, Value sz1, Value sz2, Value nseA,`。
- **L532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value rowA, Value colA, Value valA,`.
  **L532 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value rowA, Value colA, Value valA,`。
- **L533 EN**: Continues the surrounding expression or declaration: `CuSparseFormat format, bool enableRT) {`.
  **L533 CN**: 继续构造周围的表达式或声明：`CuSparseFormat format, bool enableRT) {`。
- **L534 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L534 CN**: 开始 `if` 控制流语句并计算其条件。
- **L535 EN**: Comment explains nearby logic, invariants, or intent: `Library uses SoA COO, direct IR uses AoS COO.`.
  **L535 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Library uses SoA COO, direct IR uses AoS COO.`。
- **L536 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L536 CN**: 开始 `if` 控制流语句并计算其条件。
- **L537 EN**: Checks an internal invariant in debug builds.
  **L537 CN**: 在调试构建中检查内部不变式。
- **L538 EN**: Returns from the current function with `gpu::CreateCooOp::create(builder, loc, handleTp, tokenTp, token,`.
  **L538 CN**: 以 `gpu::CreateCooOp::create(builder, loc, handleTp, tokenTp, token,` 从当前函数返回。
- **L539 EN**: Executes a standalone statement or declaration: `sz1, sz2, nseA, rowA, colA, valA);`.
  **L539 CN**: 执行一条独立语句或声明：`sz1, sz2, nseA, rowA, colA, valA);`。
- **L540 EN**: Closes the current lexical scope or compound statement.
  **L540 CN**: 结束当前词法作用域或复合语句块。
- **L541 EN**: Starts a preprocessor conditional block: `#ifdef CUSPARSE_COO_AOS`.
  **L541 CN**: 开始一个预处理条件块：`#ifdef CUSPARSE_COO_AOS`。
- **L542 EN**: Checks an internal invariant in debug builds.
  **L542 CN**: 在调试构建中检查内部不变式。
- **L543 EN**: Returns from the current function with `gpu::CreateCooAoSOp::create(builder, loc, handleTp, tokenTp, token,`.
  **L543 CN**: 以 `gpu::CreateCooAoSOp::create(builder, loc, handleTp, tokenTp, token,` 从当前函数返回。
- **L544 EN**: Executes a standalone statement or declaration: `sz1, sz2, nseA, rowA, valA);`.
  **L544 CN**: 执行一条独立语句或声明：`sz1, sz2, nseA, rowA, valA);`。
- **L545 EN**: Continues the active preprocessor branch selection.
  **L545 CN**: 继续当前的预处理分支选择。
- **L546 EN**: Marks this control path as unreachable.
  **L546 CN**: 将该控制路径标记为不可达。
- **L547 EN**: Closes the current preprocessor conditional block.
  **L547 CN**: 结束当前预处理条件块。
- **L548 EN**: Closes the current lexical scope or compound statement.
  **L548 CN**: 结束当前词法作用域或复合语句块。
- **L549 EN**: Checks an internal invariant in debug builds.
  **L549 CN**: 在调试构建中检查内部不变式。
- **L550 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L550 CN**: 开始 `if` 控制流语句并计算其条件。
- **L551 EN**: Returns from the current function with `gpu::CreateCsrOp::create(builder, loc, handleTp, tokenTp, token, sz1,`.
  **L551 CN**: 以 `gpu::CreateCsrOp::create(builder, loc, handleTp, tokenTp, token, sz1,` 从当前函数返回。
- **L552 EN**: Executes a standalone statement or declaration: `sz2, nseA, rowA, colA, valA);`.
  **L552 CN**: 执行一条独立语句或声明：`sz2, nseA, rowA, colA, valA);`。

### Lines 553-576

````cpp
  if (format == CuSparseFormat::kCSC)
    return gpu::CreateCscOp::create(builder, loc, handleTp, tokenTp, token, sz1,
                                    sz2, nseA, rowA, colA, valA);
  // BSR requires a bit more work since we need to pass in the block size
  // and all others sizes in terms of blocks (#block-rows, #block-cols,
  // #nonzero-blocks).
  assert(format == CuSparseFormat::kBSR);
  SmallVector<unsigned> dims = getBlockSize(aTp.getDimToLvl());
  assert(dims.size() == 2 && dims[0] == dims[1]);
  uint64_t b = dims[0];
  Value bSz = constantIndex(builder, loc, b);
  Value bRows = arith::DivUIOp::create(builder, loc, sz1, bSz);
  Value bCols = arith::DivUIOp::create(builder, loc, sz2, bSz);
  Value bNum = arith::DivUIOp::create(builder, loc, nseA,
                                      constantIndex(builder, loc, b * b));
  return gpu::CreateBsrOp::create(builder, loc, handleTp, tokenTp, token, bRows,
                                  bCols, bNum, bSz, bSz, rowA, colA, valA);
}

/// Match and rewrite SpMV kernel.
static LogicalResult rewriteSpMV(PatternRewriter &rewriter,
                                 linalg::GenericOp op, bool enableRT) {
  Location loc = op.getLoc();
  Value a = op.getOperand(0);
````
- **L553 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L553 CN**: 开始 `if` 控制流语句并计算其条件。
- **L554 EN**: Returns from the current function with `gpu::CreateCscOp::create(builder, loc, handleTp, tokenTp, token, sz1,`.
  **L554 CN**: 以 `gpu::CreateCscOp::create(builder, loc, handleTp, tokenTp, token, sz1,` 从当前函数返回。
- **L555 EN**: Executes a standalone statement or declaration: `sz2, nseA, rowA, colA, valA);`.
  **L555 CN**: 执行一条独立语句或声明：`sz2, nseA, rowA, colA, valA);`。
- **L556 EN**: Comment explains nearby logic, invariants, or intent: `BSR requires a bit more work since we need to pass in the block size`.
  **L556 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BSR requires a bit more work since we need to pass in the block size`。
- **L557 EN**: Comment explains nearby logic, invariants, or intent: `and all others sizes in terms of blocks (#block-rows, #block-cols,`.
  **L557 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and all others sizes in terms of blocks (#block-rows, #block-cols,`。
- **L558 EN**: Comment explains nearby logic, invariants, or intent: `#nonzero-blocks).`.
  **L558 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`#nonzero-blocks).`。
- **L559 EN**: Checks an internal invariant in debug builds.
  **L559 CN**: 在调试构建中检查内部不变式。
- **L560 EN**: Initializes variable `dims` from the right-hand expression.
  **L560 CN**: 使用右侧表达式初始化变量 `dims`。
- **L561 EN**: Checks an internal invariant in debug builds.
  **L561 CN**: 在调试构建中检查内部不变式。
- **L562 EN**: Initializes variable `b` from the right-hand expression.
  **L562 CN**: 使用右侧表达式初始化变量 `b`。
- **L563 EN**: Initializes variable `bSz` from the right-hand expression.
  **L563 CN**: 使用右侧表达式初始化变量 `bSz`。
- **L564 EN**: Initializes variable `bRows` from the right-hand expression.
  **L564 CN**: 使用右侧表达式初始化变量 `bRows`。
- **L565 EN**: Initializes variable `bCols` from the right-hand expression.
  **L565 CN**: 使用右侧表达式初始化变量 `bCols`。
- **L566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value bNum = arith::DivUIOp::create(builder, loc, nseA,`.
  **L566 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value bNum = arith::DivUIOp::create(builder, loc, nseA,`。
- **L567 EN**: Executes a call or declaration centered on `constantIndex`.
  **L567 CN**: 执行以 `constantIndex` 为核心的调用或声明。
- **L568 EN**: Returns from the current function with `gpu::CreateBsrOp::create(builder, loc, handleTp, tokenTp, token, bRows,`.
  **L568 CN**: 以 `gpu::CreateBsrOp::create(builder, loc, handleTp, tokenTp, token, bRows,` 从当前函数返回。
- **L569 EN**: Executes a standalone statement or declaration: `bCols, bNum, bSz, bSz, rowA, colA, valA);`.
  **L569 CN**: 执行一条独立语句或声明：`bCols, bNum, bSz, bSz, rowA, colA, valA);`。
- **L570 EN**: Closes the current lexical scope or compound statement.
  **L570 CN**: 结束当前词法作用域或复合语句块。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L572 EN**: Comment explains nearby logic, invariants, or intent: `Match and rewrite SpMV kernel.`.
  **L572 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Match and rewrite SpMV kernel.`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static LogicalResult rewriteSpMV(PatternRewriter &rewriter,`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`static LogicalResult rewriteSpMV(PatternRewriter &rewriter,`。
- **L574 EN**: Continues the surrounding expression or declaration: `linalg::GenericOp op, bool enableRT) {`.
  **L574 CN**: 继续构造周围的表达式或声明：`linalg::GenericOp op, bool enableRT) {`。
- **L575 EN**: Initializes variable `loc` from the right-hand expression.
  **L575 CN**: 使用右侧表达式初始化变量 `loc`。
- **L576 EN**: Initializes variable `a` from the right-hand expression.
  **L576 CN**: 使用右侧表达式初始化变量 `a`。

### Lines 577-600

````cpp
  Value x = op.getOperand(1);
  Value y = op.getOperand(2); // we have y = Ax
  SmallVector<Value> tokens;

  // Only admissible sparse matrix format and dense vectors (no BSR).
  SparseTensorType aTp = getSparseTensorType(a);
  SparseTensorType xTp = getSparseTensorType(x);
  SparseTensorType yTp = getSparseTensorType(y);
  auto format = getCuSparseFormat(aTp, xTp, yTp, enableRT, /*isMatVec=*/true);
  if (format == CuSparseFormat::kNone || format == CuSparseFormat::kBSR)
    return failure();

  // Start sparse kernel and copy data from host to device.
  //   a : memR/memC/memV -> rowA,colA,valA
  //   x : memX           -> vecX
  //   y : memY           -> vecY
  Value nseA = NumberOfEntriesOp::create(rewriter, loc, a);
  Value szY = linalg::createOrFoldDimOp(rewriter, loc, a, 0);
  Value szX = linalg::createOrFoldDimOp(rewriter, loc, a, 1);
  Value memR = genFirstPosOrCrds(rewriter, loc, a, format, enableRT);
  Value memC = genSecondCrds(rewriter, loc, a, format, enableRT); // or empty
  Value memV = ToValuesOp::create(rewriter, loc, a);
  Value rowA = genAllocCopy(rewriter, loc, memR, tokens);
  Value colA = memC ? genAllocCopy(rewriter, loc, memC, tokens) : Value();
````
- **L577 EN**: Initializes variable `x` from the right-hand expression.
  **L577 CN**: 使用右侧表达式初始化变量 `x`。
- **L578 EN**: Continues logic associated with callable symbol `getOperand`.
  **L578 CN**: 继续与可调用符号 `getOperand` 相关的逻辑。
- **L579 EN**: Executes a standalone statement or declaration: `SmallVector<Value> tokens;`.
  **L579 CN**: 执行一条独立语句或声明：`SmallVector<Value> tokens;`。
- **L580 EN**: Blank line separating nearby declarations or logic blocks.
  **L580 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L581 EN**: Comment explains nearby logic, invariants, or intent: `Only admissible sparse matrix format and dense vectors (no BSR).`.
  **L581 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only admissible sparse matrix format and dense vectors (no BSR).`。
- **L582 EN**: Initializes variable `aTp` from the right-hand expression.
  **L582 CN**: 使用右侧表达式初始化变量 `aTp`。
- **L583 EN**: Initializes variable `xTp` from the right-hand expression.
  **L583 CN**: 使用右侧表达式初始化变量 `xTp`。
- **L584 EN**: Initializes variable `yTp` from the right-hand expression.
  **L584 CN**: 使用右侧表达式初始化变量 `yTp`。
- **L585 EN**: Initializes variable `format` from the right-hand expression.
  **L585 CN**: 使用右侧表达式初始化变量 `format`。
- **L586 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L586 CN**: 开始 `if` 控制流语句并计算其条件。
- **L587 EN**: Returns from the current function with `failure()`.
  **L587 CN**: 以 `failure()` 从当前函数返回。
- **L588 EN**: Blank line separating nearby declarations or logic blocks.
  **L588 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L589 EN**: Comment explains nearby logic, invariants, or intent: `Start sparse kernel and copy data from host to device.`.
  **L589 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Start sparse kernel and copy data from host to device.`。
- **L590 EN**: Comment explains nearby logic, invariants, or intent: `a : memR/memC/memV -> rowA,colA,valA`.
  **L590 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a : memR/memC/memV -> rowA,colA,valA`。
- **L591 EN**: Comment explains nearby logic, invariants, or intent: `x : memX           -> vecX`.
  **L591 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`x : memX           -> vecX`。
- **L592 EN**: Comment explains nearby logic, invariants, or intent: `y : memY           -> vecY`.
  **L592 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`y : memY           -> vecY`。
- **L593 EN**: Initializes variable `nseA` from the right-hand expression.
  **L593 CN**: 使用右侧表达式初始化变量 `nseA`。
- **L594 EN**: Initializes variable `szY` from the right-hand expression.
  **L594 CN**: 使用右侧表达式初始化变量 `szY`。
- **L595 EN**: Initializes variable `szX` from the right-hand expression.
  **L595 CN**: 使用右侧表达式初始化变量 `szX`。
- **L596 EN**: Initializes variable `memR` from the right-hand expression.
  **L596 CN**: 使用右侧表达式初始化变量 `memR`。
- **L597 EN**: Continues logic associated with callable symbol `genSecondCrds`.
  **L597 CN**: 继续与可调用符号 `genSecondCrds` 相关的逻辑。
- **L598 EN**: Initializes variable `memV` from the right-hand expression.
  **L598 CN**: 使用右侧表达式初始化变量 `memV`。
- **L599 EN**: Initializes variable `rowA` from the right-hand expression.
  **L599 CN**: 使用右侧表达式初始化变量 `rowA`。
- **L600 EN**: Initializes variable `colA` from the right-hand expression.
  **L600 CN**: 使用右侧表达式初始化变量 `colA`。

### Lines 601-624

````cpp
  Value valA = genAllocCopy(rewriter, loc, memV, tokens);
  Value memX = genTensorToMemref(rewriter, loc, x);
  Value vecX = genAllocCopy(rewriter, loc, memX, tokens);
  Value memY = genTensorToMemref(rewriter, loc, y);
  Value vecY = genAllocCopy(rewriter, loc, memY, tokens);
  genBlockingWait(rewriter, loc, tokens);
  tokens.clear();

  // Create sparse environment and sparse matrix/dense vector handles.
  Type indexTp = rewriter.getIndexType();
  Type dnTensorHandleTp = rewriter.getType<gpu::SparseDnTensorHandleType>();
  Type spmatHandleTp = rewriter.getType<gpu::SparseSpMatHandleType>();
  Type tokenTp = rewriter.getType<gpu::AsyncTokenType>();
  Value token = genFirstWait(rewriter, loc);
  Operation *spGenA =
      genSpMat(rewriter, loc, aTp, spmatHandleTp, tokenTp, token, szY, szX,
               nseA, rowA, colA, valA, format, enableRT);
  Value spMatA = spGenA->getResult(0);
  token = spGenA->getResult(1);
  auto dvecX = gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp,
                                             tokenTp, token, vecX, szX);
  Value dnX = dvecX.getResult(0);
  token = dvecX.getAsyncToken();
  auto dvecY = gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp,
````
- **L601 EN**: Initializes variable `valA` from the right-hand expression.
  **L601 CN**: 使用右侧表达式初始化变量 `valA`。
- **L602 EN**: Initializes variable `memX` from the right-hand expression.
  **L602 CN**: 使用右侧表达式初始化变量 `memX`。
- **L603 EN**: Initializes variable `vecX` from the right-hand expression.
  **L603 CN**: 使用右侧表达式初始化变量 `vecX`。
- **L604 EN**: Initializes variable `memY` from the right-hand expression.
  **L604 CN**: 使用右侧表达式初始化变量 `memY`。
- **L605 EN**: Initializes variable `vecY` from the right-hand expression.
  **L605 CN**: 使用右侧表达式初始化变量 `vecY`。
- **L606 EN**: Executes a call or declaration centered on `genBlockingWait`.
  **L606 CN**: 执行以 `genBlockingWait` 为核心的调用或声明。
- **L607 EN**: Executes a call or declaration centered on `tokens.clear`.
  **L607 CN**: 执行以 `tokens.clear` 为核心的调用或声明。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L609 EN**: Comment explains nearby logic, invariants, or intent: `Create sparse environment and sparse matrix/dense vector handles.`.
  **L609 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create sparse environment and sparse matrix/dense vector handles.`。
- **L610 EN**: Initializes variable `indexTp` from the right-hand expression.
  **L610 CN**: 使用右侧表达式初始化变量 `indexTp`。
- **L611 EN**: Initializes variable `dnTensorHandleTp` from the right-hand expression.
  **L611 CN**: 使用右侧表达式初始化变量 `dnTensorHandleTp`。
- **L612 EN**: Initializes variable `spmatHandleTp` from the right-hand expression.
  **L612 CN**: 使用右侧表达式初始化变量 `spmatHandleTp`。
- **L613 EN**: Initializes variable `tokenTp` from the right-hand expression.
  **L613 CN**: 使用右侧表达式初始化变量 `tokenTp`。
- **L614 EN**: Initializes variable `token` from the right-hand expression.
  **L614 CN**: 使用右侧表达式初始化变量 `token`。
- **L615 EN**: Continues the surrounding expression or declaration: `Operation *spGenA =`.
  **L615 CN**: 继续构造周围的表达式或声明：`Operation *spGenA =`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genSpMat(rewriter, loc, aTp, spmatHandleTp, tokenTp, token, szY, szX,`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`genSpMat(rewriter, loc, aTp, spmatHandleTp, tokenTp, token, szY, szX,`。
- **L617 EN**: Executes a standalone statement or declaration: `nseA, rowA, colA, valA, format, enableRT);`.
  **L617 CN**: 执行一条独立语句或声明：`nseA, rowA, colA, valA, format, enableRT);`。
- **L618 EN**: Initializes variable `spMatA` from the right-hand expression.
  **L618 CN**: 使用右侧表达式初始化变量 `spMatA`。
- **L619 EN**: Executes a call or declaration centered on `spGenA->getResult`.
  **L619 CN**: 执行以 `spGenA->getResult` 为核心的调用或声明。
- **L620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto dvecX = gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp,`.
  **L620 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto dvecX = gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp,`。
- **L621 EN**: Executes a standalone statement or declaration: `tokenTp, token, vecX, szX);`.
  **L621 CN**: 执行一条独立语句或声明：`tokenTp, token, vecX, szX);`。
- **L622 EN**: Initializes variable `dnX` from the right-hand expression.
  **L622 CN**: 使用右侧表达式初始化变量 `dnX`。
- **L623 EN**: Executes a call or declaration centered on `dvecX.getAsyncToken`.
  **L623 CN**: 执行以 `dvecX.getAsyncToken` 为核心的调用或声明。
- **L624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto dvecY = gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp,`.
  **L624 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto dvecY = gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp,`。

### Lines 625-648

````cpp
                                             tokenTp, token, vecY, szY);
  Value dnY = dvecY.getResult(0);
  token = dvecY.getAsyncToken();
  auto dnYType = llvm::cast<ShapedType>(y.getType()).getElementType();

  // Precompute buffersize for SpMV.
  auto bufferComp = gpu::SpMVBufferSizeOp::create(
      rewriter, loc, indexTp, tokenTp, token, spMatA, dnX, dnY,
      /*computeType=*/dnYType);
  Value bufferSz = bufferComp.getResult(0);
  token = bufferComp.getAsyncToken();
  auto buf = genAllocBuffer(rewriter, loc, bufferSz, token);
  Value buffer = buf.getResult(0);
  token = buf.getAsyncToken();

  // Perform the SpMV.
  auto spmvComp =
      gpu::SpMVOp::create(rewriter, loc, tokenTp, token, spMatA, dnX, dnY,
                          /*computeType=*/dnYType, buffer);
  token = spmvComp.getAsyncToken();

  // Copy data back to host and free all the resoures.
  token = gpu::DestroySpMatOp::create(rewriter, loc, tokenTp, token, spMatA)
              .getAsyncToken();
````
- **L625 EN**: Executes a standalone statement or declaration: `tokenTp, token, vecY, szY);`.
  **L625 CN**: 执行一条独立语句或声明：`tokenTp, token, vecY, szY);`。
- **L626 EN**: Initializes variable `dnY` from the right-hand expression.
  **L626 CN**: 使用右侧表达式初始化变量 `dnY`。
- **L627 EN**: Executes a call or declaration centered on `dvecY.getAsyncToken`.
  **L627 CN**: 执行以 `dvecY.getAsyncToken` 为核心的调用或声明。
- **L628 EN**: Initializes variable `dnYType` from the right-hand expression.
  **L628 CN**: 使用右侧表达式初始化变量 `dnYType`。
- **L629 EN**: Blank line separating nearby declarations or logic blocks.
  **L629 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L630 EN**: Comment explains nearby logic, invariants, or intent: `Precompute buffersize for SpMV.`.
  **L630 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Precompute buffersize for SpMV.`。
- **L631 EN**: Continues logic associated with callable symbol `create`.
  **L631 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, indexTp, tokenTp, token, spMatA, dnX, dnY,`.
  **L632 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, indexTp, tokenTp, token, spMatA, dnX, dnY,`。
- **L633 EN**: Comment explains nearby logic, invariants, or intent: `computeType=*/dnYType);`.
  **L633 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`computeType=*/dnYType);`。
- **L634 EN**: Initializes variable `bufferSz` from the right-hand expression.
  **L634 CN**: 使用右侧表达式初始化变量 `bufferSz`。
- **L635 EN**: Executes a call or declaration centered on `bufferComp.getAsyncToken`.
  **L635 CN**: 执行以 `bufferComp.getAsyncToken` 为核心的调用或声明。
- **L636 EN**: Initializes variable `buf` from the right-hand expression.
  **L636 CN**: 使用右侧表达式初始化变量 `buf`。
- **L637 EN**: Initializes variable `buffer` from the right-hand expression.
  **L637 CN**: 使用右侧表达式初始化变量 `buffer`。
- **L638 EN**: Executes a call or declaration centered on `buf.getAsyncToken`.
  **L638 CN**: 执行以 `buf.getAsyncToken` 为核心的调用或声明。
- **L639 EN**: Blank line separating nearby declarations or logic blocks.
  **L639 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L640 EN**: Comment explains nearby logic, invariants, or intent: `Perform the SpMV.`.
  **L640 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Perform the SpMV.`。
- **L641 EN**: Continues the surrounding expression or declaration: `auto spmvComp =`.
  **L641 CN**: 继续构造周围的表达式或声明：`auto spmvComp =`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::SpMVOp::create(rewriter, loc, tokenTp, token, spMatA, dnX, dnY,`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::SpMVOp::create(rewriter, loc, tokenTp, token, spMatA, dnX, dnY,`。
- **L643 EN**: Comment explains nearby logic, invariants, or intent: `computeType=*/dnYType, buffer);`.
  **L643 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`computeType=*/dnYType, buffer);`。
- **L644 EN**: Executes a call or declaration centered on `spmvComp.getAsyncToken`.
  **L644 CN**: 执行以 `spmvComp.getAsyncToken` 为核心的调用或声明。
- **L645 EN**: Blank line separating nearby declarations or logic blocks.
  **L645 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L646 EN**: Comment explains nearby logic, invariants, or intent: `Copy data back to host and free all the resoures.`.
  **L646 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copy data back to host and free all the resoures.`。
- **L647 EN**: Continues logic associated with callable symbol `create`.
  **L647 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L648 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L648 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。

### Lines 649-672

````cpp
  token = gpu::DestroyDnTensorOp::create(rewriter, loc, tokenTp, token, dnX)
              .getAsyncToken();
  token = gpu::DestroyDnTensorOp::create(rewriter, loc, tokenTp, token, dnY)
              .getAsyncToken();
  token = genDeallocMemRef(rewriter, loc, rowA, token);
  if (colA)
    token = genDeallocMemRef(rewriter, loc, colA, token);
  token = genDeallocMemRef(rewriter, loc, valA, token);
  token = genDeallocMemRef(rewriter, loc, buffer, token);
  token = genDeallocMemRef(rewriter, loc, vecX, token);
  token = genCopyMemRef(rewriter, loc, memY, vecY, token);
  token = genDeallocMemRef(rewriter, loc, vecY, token);
  tokens.push_back(token);
  genBlockingWait(rewriter, loc, tokens);
  tokens.clear();

  // Done.
  rewriter.replaceOpWithNewOp<bufferization::ToTensorOp>(op, y.getType(), memY);
  return success();
}

/// Match and rewrite SpMM kernel.
static LogicalResult rewriteSpMM(PatternRewriter &rewriter,
                                 linalg::GenericOp op, bool enableRT) {
````
- **L649 EN**: Continues logic associated with callable symbol `create`.
  **L649 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L650 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L650 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L651 EN**: Continues logic associated with callable symbol `create`.
  **L651 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L652 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L652 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L653 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L653 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L654 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L654 CN**: 开始 `if` 控制流语句并计算其条件。
- **L655 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L655 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L656 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L656 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L657 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L657 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L658 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L658 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L659 EN**: Executes a call or declaration centered on `genCopyMemRef`.
  **L659 CN**: 执行以 `genCopyMemRef` 为核心的调用或声明。
- **L660 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L660 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L661 EN**: Executes a call or declaration centered on `tokens.push_back`.
  **L661 CN**: 执行以 `tokens.push_back` 为核心的调用或声明。
- **L662 EN**: Executes a call or declaration centered on `genBlockingWait`.
  **L662 CN**: 执行以 `genBlockingWait` 为核心的调用或声明。
- **L663 EN**: Executes a call or declaration centered on `tokens.clear`.
  **L663 CN**: 执行以 `tokens.clear` 为核心的调用或声明。
- **L664 EN**: Blank line separating nearby declarations or logic blocks.
  **L664 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L665 EN**: Comment explains nearby logic, invariants, or intent: `Done.`.
  **L665 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Done.`。
- **L666 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithNewOp<bufferization::ToTensorOp>`.
  **L666 CN**: 执行以 `rewriter.replaceOpWithNewOp<bufferization::ToTensorOp>` 为核心的调用或声明。
- **L667 EN**: Returns from the current function with `success()`.
  **L667 CN**: 以 `success()` 从当前函数返回。
- **L668 EN**: Closes the current lexical scope or compound statement.
  **L668 CN**: 结束当前词法作用域或复合语句块。
- **L669 EN**: Blank line separating nearby declarations or logic blocks.
  **L669 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L670 EN**: Comment explains nearby logic, invariants, or intent: `Match and rewrite SpMM kernel.`.
  **L670 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Match and rewrite SpMM kernel.`。
- **L671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static LogicalResult rewriteSpMM(PatternRewriter &rewriter,`.
  **L671 CN**: 继续一个多行参数列表、初始化器或聚合项：`static LogicalResult rewriteSpMM(PatternRewriter &rewriter,`。
- **L672 EN**: Continues the surrounding expression or declaration: `linalg::GenericOp op, bool enableRT) {`.
  **L672 CN**: 继续构造周围的表达式或声明：`linalg::GenericOp op, bool enableRT) {`。

### Lines 673-696

````cpp
  Location loc = op.getLoc();
  Value a = op.getOperand(0);
  Value b = op.getOperand(1);
  Value c = op.getOperand(2); // we have C = AB
  SmallVector<Value> tokens;

  // Only admissible sparse matrix format and dense matrices (no BSR).
  SparseTensorType aTp = getSparseTensorType(a);
  SparseTensorType bTp = getSparseTensorType(b);
  SparseTensorType cTp = getSparseTensorType(c);
  auto format = getCuSparseFormat(aTp, bTp, cTp, enableRT, /*isMatVec=*/false);
  if (format == CuSparseFormat::kNone || format == CuSparseFormat::kBSR)
    return failure();

  // Start sparse kernel and copy data from host to device.
  //   a : memR/memC/memV -> rowA,colA,valA
  //   b : bufB           -> matB
  //   c : bufC           -> matC
  Value nseA = NumberOfEntriesOp::create(rewriter, loc, a);
  Value szm = linalg::createOrFoldDimOp(rewriter, loc, a, 0);
  Value szk = linalg::createOrFoldDimOp(rewriter, loc, a, 1);
  Value szn = linalg::createOrFoldDimOp(rewriter, loc, b, 1);
  Value memR = genFirstPosOrCrds(rewriter, loc, a, format, enableRT);
  Value memC = genSecondCrds(rewriter, loc, a, format, enableRT); // or empty
````
- **L673 EN**: Initializes variable `loc` from the right-hand expression.
  **L673 CN**: 使用右侧表达式初始化变量 `loc`。
- **L674 EN**: Initializes variable `a` from the right-hand expression.
  **L674 CN**: 使用右侧表达式初始化变量 `a`。
- **L675 EN**: Initializes variable `b` from the right-hand expression.
  **L675 CN**: 使用右侧表达式初始化变量 `b`。
- **L676 EN**: Continues logic associated with callable symbol `getOperand`.
  **L676 CN**: 继续与可调用符号 `getOperand` 相关的逻辑。
- **L677 EN**: Executes a standalone statement or declaration: `SmallVector<Value> tokens;`.
  **L677 CN**: 执行一条独立语句或声明：`SmallVector<Value> tokens;`。
- **L678 EN**: Blank line separating nearby declarations or logic blocks.
  **L678 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L679 EN**: Comment explains nearby logic, invariants, or intent: `Only admissible sparse matrix format and dense matrices (no BSR).`.
  **L679 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only admissible sparse matrix format and dense matrices (no BSR).`。
- **L680 EN**: Initializes variable `aTp` from the right-hand expression.
  **L680 CN**: 使用右侧表达式初始化变量 `aTp`。
- **L681 EN**: Initializes variable `bTp` from the right-hand expression.
  **L681 CN**: 使用右侧表达式初始化变量 `bTp`。
- **L682 EN**: Initializes variable `cTp` from the right-hand expression.
  **L682 CN**: 使用右侧表达式初始化变量 `cTp`。
- **L683 EN**: Initializes variable `format` from the right-hand expression.
  **L683 CN**: 使用右侧表达式初始化变量 `format`。
- **L684 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L684 CN**: 开始 `if` 控制流语句并计算其条件。
- **L685 EN**: Returns from the current function with `failure()`.
  **L685 CN**: 以 `failure()` 从当前函数返回。
- **L686 EN**: Blank line separating nearby declarations or logic blocks.
  **L686 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L687 EN**: Comment explains nearby logic, invariants, or intent: `Start sparse kernel and copy data from host to device.`.
  **L687 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Start sparse kernel and copy data from host to device.`。
- **L688 EN**: Comment explains nearby logic, invariants, or intent: `a : memR/memC/memV -> rowA,colA,valA`.
  **L688 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a : memR/memC/memV -> rowA,colA,valA`。
- **L689 EN**: Comment explains nearby logic, invariants, or intent: `b : bufB           -> matB`.
  **L689 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`b : bufB           -> matB`。
- **L690 EN**: Comment explains nearby logic, invariants, or intent: `c : bufC           -> matC`.
  **L690 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`c : bufC           -> matC`。
- **L691 EN**: Initializes variable `nseA` from the right-hand expression.
  **L691 CN**: 使用右侧表达式初始化变量 `nseA`。
- **L692 EN**: Initializes variable `szm` from the right-hand expression.
  **L692 CN**: 使用右侧表达式初始化变量 `szm`。
- **L693 EN**: Initializes variable `szk` from the right-hand expression.
  **L693 CN**: 使用右侧表达式初始化变量 `szk`。
- **L694 EN**: Initializes variable `szn` from the right-hand expression.
  **L694 CN**: 使用右侧表达式初始化变量 `szn`。
- **L695 EN**: Initializes variable `memR` from the right-hand expression.
  **L695 CN**: 使用右侧表达式初始化变量 `memR`。
- **L696 EN**: Continues logic associated with callable symbol `genSecondCrds`.
  **L696 CN**: 继续与可调用符号 `genSecondCrds` 相关的逻辑。

### Lines 697-720

````cpp
  Value memV = ToValuesOp::create(rewriter, loc, a);
  Value rowA = genAllocCopy(rewriter, loc, memR, tokens);
  Value colA = memC ? genAllocCopy(rewriter, loc, memC, tokens) : Value();
  Value valA = genAllocCopy(rewriter, loc, memV, tokens);
  Value bufB = genTensorToMemref(rewriter, loc, b);
  Value matB = genAllocCopy(rewriter, loc, bufB, tokens);
  Value bufC = genTensorToMemref(rewriter, loc, c);
  Value matC = genAllocCopy(rewriter, loc, bufC, tokens);
  genBlockingWait(rewriter, loc, tokens);
  tokens.clear();

  // Create sparse environment and sparse matrix/dense matrix handles.
  Type indexTp = rewriter.getIndexType();
  Type dnTensorHandleTp = rewriter.getType<gpu::SparseDnTensorHandleType>();
  Type spMatHandleTp = rewriter.getType<gpu::SparseSpMatHandleType>();
  Type tokenTp = rewriter.getType<gpu::AsyncTokenType>();
  Value token = genFirstWait(rewriter, loc);
  Operation *spGenA =
      genSpMat(rewriter, loc, aTp, spMatHandleTp, tokenTp, token, szm, szk,
               nseA, rowA, colA, valA, format, enableRT);
  Value spMatA = spGenA->getResult(0);
  token = spGenA->getResult(1);
  auto dmatB =
      gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp, tokenTp,
````
- **L697 EN**: Initializes variable `memV` from the right-hand expression.
  **L697 CN**: 使用右侧表达式初始化变量 `memV`。
- **L698 EN**: Initializes variable `rowA` from the right-hand expression.
  **L698 CN**: 使用右侧表达式初始化变量 `rowA`。
- **L699 EN**: Initializes variable `colA` from the right-hand expression.
  **L699 CN**: 使用右侧表达式初始化变量 `colA`。
- **L700 EN**: Initializes variable `valA` from the right-hand expression.
  **L700 CN**: 使用右侧表达式初始化变量 `valA`。
- **L701 EN**: Initializes variable `bufB` from the right-hand expression.
  **L701 CN**: 使用右侧表达式初始化变量 `bufB`。
- **L702 EN**: Initializes variable `matB` from the right-hand expression.
  **L702 CN**: 使用右侧表达式初始化变量 `matB`。
- **L703 EN**: Initializes variable `bufC` from the right-hand expression.
  **L703 CN**: 使用右侧表达式初始化变量 `bufC`。
- **L704 EN**: Initializes variable `matC` from the right-hand expression.
  **L704 CN**: 使用右侧表达式初始化变量 `matC`。
- **L705 EN**: Executes a call or declaration centered on `genBlockingWait`.
  **L705 CN**: 执行以 `genBlockingWait` 为核心的调用或声明。
- **L706 EN**: Executes a call or declaration centered on `tokens.clear`.
  **L706 CN**: 执行以 `tokens.clear` 为核心的调用或声明。
- **L707 EN**: Blank line separating nearby declarations or logic blocks.
  **L707 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L708 EN**: Comment explains nearby logic, invariants, or intent: `Create sparse environment and sparse matrix/dense matrix handles.`.
  **L708 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create sparse environment and sparse matrix/dense matrix handles.`。
- **L709 EN**: Initializes variable `indexTp` from the right-hand expression.
  **L709 CN**: 使用右侧表达式初始化变量 `indexTp`。
- **L710 EN**: Initializes variable `dnTensorHandleTp` from the right-hand expression.
  **L710 CN**: 使用右侧表达式初始化变量 `dnTensorHandleTp`。
- **L711 EN**: Initializes variable `spMatHandleTp` from the right-hand expression.
  **L711 CN**: 使用右侧表达式初始化变量 `spMatHandleTp`。
- **L712 EN**: Initializes variable `tokenTp` from the right-hand expression.
  **L712 CN**: 使用右侧表达式初始化变量 `tokenTp`。
- **L713 EN**: Initializes variable `token` from the right-hand expression.
  **L713 CN**: 使用右侧表达式初始化变量 `token`。
- **L714 EN**: Continues the surrounding expression or declaration: `Operation *spGenA =`.
  **L714 CN**: 继续构造周围的表达式或声明：`Operation *spGenA =`。
- **L715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genSpMat(rewriter, loc, aTp, spMatHandleTp, tokenTp, token, szm, szk,`.
  **L715 CN**: 继续一个多行参数列表、初始化器或聚合项：`genSpMat(rewriter, loc, aTp, spMatHandleTp, tokenTp, token, szm, szk,`。
- **L716 EN**: Executes a standalone statement or declaration: `nseA, rowA, colA, valA, format, enableRT);`.
  **L716 CN**: 执行一条独立语句或声明：`nseA, rowA, colA, valA, format, enableRT);`。
- **L717 EN**: Initializes variable `spMatA` from the right-hand expression.
  **L717 CN**: 使用右侧表达式初始化变量 `spMatA`。
- **L718 EN**: Executes a call or declaration centered on `spGenA->getResult`.
  **L718 CN**: 执行以 `spGenA->getResult` 为核心的调用或声明。
- **L719 EN**: Continues the surrounding expression or declaration: `auto dmatB =`.
  **L719 CN**: 继续构造周围的表达式或声明：`auto dmatB =`。
- **L720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp, tokenTp,`.
  **L720 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp, tokenTp,`。

### Lines 721-744

````cpp
                                    token, matB, SmallVector<Value>{szk, szn});
  Value dnB = dmatB.getResult(0);
  token = dmatB.getAsyncToken();
  auto dmatC =
      gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp, tokenTp,
                                    token, matC, SmallVector<Value>{szm, szn});
  Value dnC = dmatC.getResult(0);
  token = dmatC.getAsyncToken();
  auto dmatCType = llvm::cast<ShapedType>(c.getType()).getElementType();

  // Precompute buffersize for SpMM.
  auto bufferComp = gpu::SpMMBufferSizeOp::create(
      rewriter, loc, indexTp, tokenTp, token, spMatA, dnB, dnC,
      /*computeType=*/dmatCType);
  Value bufferSz = bufferComp.getResult(0);
  token = bufferComp.getAsyncToken();
  auto buf = genAllocBuffer(rewriter, loc, bufferSz, token);
  Value buffer = buf.getResult(0);
  token = buf.getAsyncToken();
  auto dnCType = llvm::cast<ShapedType>(c.getType()).getElementType();

  // Perform the SpMM.
  auto spmmComp =
      gpu::SpMMOp::create(rewriter, loc, tokenTp, token, spMatA, dnB, dnC,
````
- **L721 EN**: Executes a standalone statement or declaration: `token, matB, SmallVector<Value>{szk, szn});`.
  **L721 CN**: 执行一条独立语句或声明：`token, matB, SmallVector<Value>{szk, szn});`。
- **L722 EN**: Initializes variable `dnB` from the right-hand expression.
  **L722 CN**: 使用右侧表达式初始化变量 `dnB`。
- **L723 EN**: Executes a call or declaration centered on `dmatB.getAsyncToken`.
  **L723 CN**: 执行以 `dmatB.getAsyncToken` 为核心的调用或声明。
- **L724 EN**: Continues the surrounding expression or declaration: `auto dmatC =`.
  **L724 CN**: 继续构造周围的表达式或声明：`auto dmatC =`。
- **L725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp, tokenTp,`.
  **L725 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp, tokenTp,`。
- **L726 EN**: Executes a standalone statement or declaration: `token, matC, SmallVector<Value>{szm, szn});`.
  **L726 CN**: 执行一条独立语句或声明：`token, matC, SmallVector<Value>{szm, szn});`。
- **L727 EN**: Initializes variable `dnC` from the right-hand expression.
  **L727 CN**: 使用右侧表达式初始化变量 `dnC`。
- **L728 EN**: Executes a call or declaration centered on `dmatC.getAsyncToken`.
  **L728 CN**: 执行以 `dmatC.getAsyncToken` 为核心的调用或声明。
- **L729 EN**: Initializes variable `dmatCType` from the right-hand expression.
  **L729 CN**: 使用右侧表达式初始化变量 `dmatCType`。
- **L730 EN**: Blank line separating nearby declarations or logic blocks.
  **L730 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L731 EN**: Comment explains nearby logic, invariants, or intent: `Precompute buffersize for SpMM.`.
  **L731 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Precompute buffersize for SpMM.`。
- **L732 EN**: Continues logic associated with callable symbol `create`.
  **L732 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, indexTp, tokenTp, token, spMatA, dnB, dnC,`.
  **L733 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, indexTp, tokenTp, token, spMatA, dnB, dnC,`。
- **L734 EN**: Comment explains nearby logic, invariants, or intent: `computeType=*/dmatCType);`.
  **L734 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`computeType=*/dmatCType);`。
- **L735 EN**: Initializes variable `bufferSz` from the right-hand expression.
  **L735 CN**: 使用右侧表达式初始化变量 `bufferSz`。
- **L736 EN**: Executes a call or declaration centered on `bufferComp.getAsyncToken`.
  **L736 CN**: 执行以 `bufferComp.getAsyncToken` 为核心的调用或声明。
- **L737 EN**: Initializes variable `buf` from the right-hand expression.
  **L737 CN**: 使用右侧表达式初始化变量 `buf`。
- **L738 EN**: Initializes variable `buffer` from the right-hand expression.
  **L738 CN**: 使用右侧表达式初始化变量 `buffer`。
- **L739 EN**: Executes a call or declaration centered on `buf.getAsyncToken`.
  **L739 CN**: 执行以 `buf.getAsyncToken` 为核心的调用或声明。
- **L740 EN**: Initializes variable `dnCType` from the right-hand expression.
  **L740 CN**: 使用右侧表达式初始化变量 `dnCType`。
- **L741 EN**: Blank line separating nearby declarations or logic blocks.
  **L741 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L742 EN**: Comment explains nearby logic, invariants, or intent: `Perform the SpMM.`.
  **L742 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Perform the SpMM.`。
- **L743 EN**: Continues the surrounding expression or declaration: `auto spmmComp =`.
  **L743 CN**: 继续构造周围的表达式或声明：`auto spmmComp =`。
- **L744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::SpMMOp::create(rewriter, loc, tokenTp, token, spMatA, dnB, dnC,`.
  **L744 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::SpMMOp::create(rewriter, loc, tokenTp, token, spMatA, dnB, dnC,`。

### Lines 745-768

````cpp
                          /*computeType=*/dnCType, buffer);
  token = spmmComp.getAsyncToken();

  // Copy data back to host and free all the resoures.
  token = gpu::DestroySpMatOp::create(rewriter, loc, tokenTp, token, spMatA)
              .getAsyncToken();
  token = gpu::DestroyDnTensorOp::create(rewriter, loc, tokenTp, token, dnB)
              .getAsyncToken();
  token = gpu::DestroyDnTensorOp::create(rewriter, loc, tokenTp, token, dnC)
              .getAsyncToken();
  token = genDeallocMemRef(rewriter, loc, rowA, token);
  if (colA)
    token = genDeallocMemRef(rewriter, loc, colA, token);
  token = genDeallocMemRef(rewriter, loc, valA, token);
  token = genDeallocMemRef(rewriter, loc, buffer, token);
  token = genDeallocMemRef(rewriter, loc, matB, token);
  token = genCopyMemRef(rewriter, loc, bufC, matC, token);
  token = genDeallocMemRef(rewriter, loc, matC, token);
  tokens.push_back(token);
  genBlockingWait(rewriter, loc, tokens);
  tokens.clear();

  // Done.
  rewriter.replaceOpWithNewOp<bufferization::ToTensorOp>(op, c.getType(), bufC);
````
- **L745 EN**: Comment explains nearby logic, invariants, or intent: `computeType=*/dnCType, buffer);`.
  **L745 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`computeType=*/dnCType, buffer);`。
- **L746 EN**: Executes a call or declaration centered on `spmmComp.getAsyncToken`.
  **L746 CN**: 执行以 `spmmComp.getAsyncToken` 为核心的调用或声明。
- **L747 EN**: Blank line separating nearby declarations or logic blocks.
  **L747 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L748 EN**: Comment explains nearby logic, invariants, or intent: `Copy data back to host and free all the resoures.`.
  **L748 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copy data back to host and free all the resoures.`。
- **L749 EN**: Continues logic associated with callable symbol `create`.
  **L749 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L750 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L750 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L751 EN**: Continues logic associated with callable symbol `create`.
  **L751 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L752 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L752 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L753 EN**: Continues logic associated with callable symbol `create`.
  **L753 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L754 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L754 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L755 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L755 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L756 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L756 CN**: 开始 `if` 控制流语句并计算其条件。
- **L757 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L757 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L758 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L758 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L759 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L759 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L760 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L760 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L761 EN**: Executes a call or declaration centered on `genCopyMemRef`.
  **L761 CN**: 执行以 `genCopyMemRef` 为核心的调用或声明。
- **L762 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L762 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L763 EN**: Executes a call or declaration centered on `tokens.push_back`.
  **L763 CN**: 执行以 `tokens.push_back` 为核心的调用或声明。
- **L764 EN**: Executes a call or declaration centered on `genBlockingWait`.
  **L764 CN**: 执行以 `genBlockingWait` 为核心的调用或声明。
- **L765 EN**: Executes a call or declaration centered on `tokens.clear`.
  **L765 CN**: 执行以 `tokens.clear` 为核心的调用或声明。
- **L766 EN**: Blank line separating nearby declarations or logic blocks.
  **L766 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L767 EN**: Comment explains nearby logic, invariants, or intent: `Done.`.
  **L767 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Done.`。
- **L768 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithNewOp<bufferization::ToTensorOp>`.
  **L768 CN**: 执行以 `rewriter.replaceOpWithNewOp<bufferization::ToTensorOp>` 为核心的调用或声明。

### Lines 769-792

````cpp
  return success();
}

// Match and rewrite SpGEMM kernel.
static LogicalResult rewriteSpGEMM(PatternRewriter &rewriter,
                                   linalg::GenericOp op, bool enableRT) {
  Location loc = op.getLoc();
  Value a = op.getOperand(0);
  Value b = op.getOperand(1);
  Value c = op.getOperand(2); // we have C = AB
  SmallVector<Value> tokens;

  // Only CSR <- CSR x CSR supported.
  auto format = CuSparseFormat::kCSR;
  SparseTensorType aTp = getSparseTensorType(a);
  SparseTensorType bTp = getSparseTensorType(b);
  SparseTensorType cTp = getSparseTensorType(c);
  if (!isAdmissibleCSR(aTp) || !isAdmissibleCSR(bTp) || !isAdmissibleCSR(cTp))
    return failure();

  // Start sparse kernel and copy data from host to device.
  //   a : amemR/amemC/amemV -> rowA,colA,valA
  //   b : bmemR/bmemC/bmemV -> rowB,colB,valB
  //   c : materializes
````
- **L769 EN**: Returns from the current function with `success()`.
  **L769 CN**: 以 `success()` 从当前函数返回。
- **L770 EN**: Closes the current lexical scope or compound statement.
  **L770 CN**: 结束当前词法作用域或复合语句块。
- **L771 EN**: Blank line separating nearby declarations or logic blocks.
  **L771 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L772 EN**: Comment explains nearby logic, invariants, or intent: `Match and rewrite SpGEMM kernel.`.
  **L772 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Match and rewrite SpGEMM kernel.`。
- **L773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static LogicalResult rewriteSpGEMM(PatternRewriter &rewriter,`.
  **L773 CN**: 继续一个多行参数列表、初始化器或聚合项：`static LogicalResult rewriteSpGEMM(PatternRewriter &rewriter,`。
- **L774 EN**: Continues the surrounding expression or declaration: `linalg::GenericOp op, bool enableRT) {`.
  **L774 CN**: 继续构造周围的表达式或声明：`linalg::GenericOp op, bool enableRT) {`。
- **L775 EN**: Initializes variable `loc` from the right-hand expression.
  **L775 CN**: 使用右侧表达式初始化变量 `loc`。
- **L776 EN**: Initializes variable `a` from the right-hand expression.
  **L776 CN**: 使用右侧表达式初始化变量 `a`。
- **L777 EN**: Initializes variable `b` from the right-hand expression.
  **L777 CN**: 使用右侧表达式初始化变量 `b`。
- **L778 EN**: Continues logic associated with callable symbol `getOperand`.
  **L778 CN**: 继续与可调用符号 `getOperand` 相关的逻辑。
- **L779 EN**: Executes a standalone statement or declaration: `SmallVector<Value> tokens;`.
  **L779 CN**: 执行一条独立语句或声明：`SmallVector<Value> tokens;`。
- **L780 EN**: Blank line separating nearby declarations or logic blocks.
  **L780 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L781 EN**: Comment explains nearby logic, invariants, or intent: `Only CSR <- CSR x CSR supported.`.
  **L781 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only CSR <- CSR x CSR supported.`。
- **L782 EN**: Initializes variable `format` from the right-hand expression.
  **L782 CN**: 使用右侧表达式初始化变量 `format`。
- **L783 EN**: Initializes variable `aTp` from the right-hand expression.
  **L783 CN**: 使用右侧表达式初始化变量 `aTp`。
- **L784 EN**: Initializes variable `bTp` from the right-hand expression.
  **L784 CN**: 使用右侧表达式初始化变量 `bTp`。
- **L785 EN**: Initializes variable `cTp` from the right-hand expression.
  **L785 CN**: 使用右侧表达式初始化变量 `cTp`。
- **L786 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L786 CN**: 开始 `if` 控制流语句并计算其条件。
- **L787 EN**: Returns from the current function with `failure()`.
  **L787 CN**: 以 `failure()` 从当前函数返回。
- **L788 EN**: Blank line separating nearby declarations or logic blocks.
  **L788 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L789 EN**: Comment explains nearby logic, invariants, or intent: `Start sparse kernel and copy data from host to device.`.
  **L789 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Start sparse kernel and copy data from host to device.`。
- **L790 EN**: Comment explains nearby logic, invariants, or intent: `a : amemR/amemC/amemV -> rowA,colA,valA`.
  **L790 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a : amemR/amemC/amemV -> rowA,colA,valA`。
- **L791 EN**: Comment explains nearby logic, invariants, or intent: `b : bmemR/bmemC/bmemV -> rowB,colB,valB`.
  **L791 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`b : bmemR/bmemC/bmemV -> rowB,colB,valB`。
- **L792 EN**: Comment explains nearby logic, invariants, or intent: `c : materializes`.
  **L792 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`c : materializes`。

### Lines 793-816

````cpp
  auto dnCType = cTp.getElementType();
  Value nseA = NumberOfEntriesOp::create(rewriter, loc, a);
  Value nseB = NumberOfEntriesOp::create(rewriter, loc, b);
  Value szm = linalg::createOrFoldDimOp(rewriter, loc, a, 0);
  Value szk = linalg::createOrFoldDimOp(rewriter, loc, a, 1);
  Value szn = linalg::createOrFoldDimOp(rewriter, loc, b, 1);
  Value amemR = genFirstPosOrCrds(rewriter, loc, a, format, enableRT);
  Value amemC = genSecondCrds(rewriter, loc, a, format, enableRT); // not empty
  Value amemV = ToValuesOp::create(rewriter, loc, a);
  Value bmemR = genFirstPosOrCrds(rewriter, loc, b, format, enableRT);
  Value bmemC = genSecondCrds(rewriter, loc, b, format, enableRT); // not empty
  Value bmemV = ToValuesOp::create(rewriter, loc, b);
  Value rowA = genAllocCopy(rewriter, loc, amemR, tokens);
  Value colA = genAllocCopy(rewriter, loc, amemC, tokens);
  Value valA = genAllocCopy(rewriter, loc, amemV, tokens);
  Value rowB = genAllocCopy(rewriter, loc, bmemR, tokens);
  Value colB = genAllocCopy(rewriter, loc, bmemC, tokens);
  Value valB = genAllocCopy(rewriter, loc, bmemV, tokens);
  genBlockingWait(rewriter, loc, tokens);
  tokens.clear();

  // Create sparse environment and sparse matrix/dense vector handles.
  Type indexTp = rewriter.getIndexType();
  Type spmatHandleTp = rewriter.getType<gpu::SparseSpMatHandleType>();
````
- **L793 EN**: Initializes variable `dnCType` from the right-hand expression.
  **L793 CN**: 使用右侧表达式初始化变量 `dnCType`。
- **L794 EN**: Initializes variable `nseA` from the right-hand expression.
  **L794 CN**: 使用右侧表达式初始化变量 `nseA`。
- **L795 EN**: Initializes variable `nseB` from the right-hand expression.
  **L795 CN**: 使用右侧表达式初始化变量 `nseB`。
- **L796 EN**: Initializes variable `szm` from the right-hand expression.
  **L796 CN**: 使用右侧表达式初始化变量 `szm`。
- **L797 EN**: Initializes variable `szk` from the right-hand expression.
  **L797 CN**: 使用右侧表达式初始化变量 `szk`。
- **L798 EN**: Initializes variable `szn` from the right-hand expression.
  **L798 CN**: 使用右侧表达式初始化变量 `szn`。
- **L799 EN**: Initializes variable `amemR` from the right-hand expression.
  **L799 CN**: 使用右侧表达式初始化变量 `amemR`。
- **L800 EN**: Continues logic associated with callable symbol `genSecondCrds`.
  **L800 CN**: 继续与可调用符号 `genSecondCrds` 相关的逻辑。
- **L801 EN**: Initializes variable `amemV` from the right-hand expression.
  **L801 CN**: 使用右侧表达式初始化变量 `amemV`。
- **L802 EN**: Initializes variable `bmemR` from the right-hand expression.
  **L802 CN**: 使用右侧表达式初始化变量 `bmemR`。
- **L803 EN**: Continues logic associated with callable symbol `genSecondCrds`.
  **L803 CN**: 继续与可调用符号 `genSecondCrds` 相关的逻辑。
- **L804 EN**: Initializes variable `bmemV` from the right-hand expression.
  **L804 CN**: 使用右侧表达式初始化变量 `bmemV`。
- **L805 EN**: Initializes variable `rowA` from the right-hand expression.
  **L805 CN**: 使用右侧表达式初始化变量 `rowA`。
- **L806 EN**: Initializes variable `colA` from the right-hand expression.
  **L806 CN**: 使用右侧表达式初始化变量 `colA`。
- **L807 EN**: Initializes variable `valA` from the right-hand expression.
  **L807 CN**: 使用右侧表达式初始化变量 `valA`。
- **L808 EN**: Initializes variable `rowB` from the right-hand expression.
  **L808 CN**: 使用右侧表达式初始化变量 `rowB`。
- **L809 EN**: Initializes variable `colB` from the right-hand expression.
  **L809 CN**: 使用右侧表达式初始化变量 `colB`。
- **L810 EN**: Initializes variable `valB` from the right-hand expression.
  **L810 CN**: 使用右侧表达式初始化变量 `valB`。
- **L811 EN**: Executes a call or declaration centered on `genBlockingWait`.
  **L811 CN**: 执行以 `genBlockingWait` 为核心的调用或声明。
- **L812 EN**: Executes a call or declaration centered on `tokens.clear`.
  **L812 CN**: 执行以 `tokens.clear` 为核心的调用或声明。
- **L813 EN**: Blank line separating nearby declarations or logic blocks.
  **L813 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L814 EN**: Comment explains nearby logic, invariants, or intent: `Create sparse environment and sparse matrix/dense vector handles.`.
  **L814 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create sparse environment and sparse matrix/dense vector handles.`。
- **L815 EN**: Initializes variable `indexTp` from the right-hand expression.
  **L815 CN**: 使用右侧表达式初始化变量 `indexTp`。
- **L816 EN**: Initializes variable `spmatHandleTp` from the right-hand expression.
  **L816 CN**: 使用右侧表达式初始化变量 `spmatHandleTp`。

### Lines 817-840

````cpp
  Type descTp = rewriter.getType<gpu::SparseSpGEMMOpHandleType>();
  Type tokenTp = rewriter.getType<gpu::AsyncTokenType>();
  Value token = genFirstWait(rewriter, loc);
  Operation *spGenA =
      genSpMat(rewriter, loc, aTp, spmatHandleTp, tokenTp, token, szm, szk,
               nseA, rowA, colA, valA, format, enableRT);
  Value spMatA = spGenA->getResult(0);
  token = spGenA->getResult(1);
  Operation *spGenB =
      genSpMat(rewriter, loc, bTp, spmatHandleTp, tokenTp, token, szk, szn,
               nseB, rowB, colB, valB, format, enableRT);
  Value spMatB = spGenB->getResult(0);
  token = spGenB->getResult(1);

  // Sparse matrix C materializes (also assumes beta == 0).
  Value zero = constantIndex(rewriter, loc, 0);
  Value one = constantIndex(rewriter, loc, 1);
  Value mplus1 = arith::AddIOp::create(rewriter, loc, szm, one);
  auto e1 = genAllocBuffer(rewriter, loc, cTp.getPosType(), mplus1, token);
  Value rowC = e1.getResult(0);
  token = e1.getAsyncToken();
  auto e2 = genAllocBuffer(rewriter, loc, cTp.getCrdType(), zero, token);
  Value colC = e2.getResult(0); // no free needed
  token = e2.getAsyncToken();
````
- **L817 EN**: Initializes variable `descTp` from the right-hand expression.
  **L817 CN**: 使用右侧表达式初始化变量 `descTp`。
- **L818 EN**: Initializes variable `tokenTp` from the right-hand expression.
  **L818 CN**: 使用右侧表达式初始化变量 `tokenTp`。
- **L819 EN**: Initializes variable `token` from the right-hand expression.
  **L819 CN**: 使用右侧表达式初始化变量 `token`。
- **L820 EN**: Continues the surrounding expression or declaration: `Operation *spGenA =`.
  **L820 CN**: 继续构造周围的表达式或声明：`Operation *spGenA =`。
- **L821 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genSpMat(rewriter, loc, aTp, spmatHandleTp, tokenTp, token, szm, szk,`.
  **L821 CN**: 继续一个多行参数列表、初始化器或聚合项：`genSpMat(rewriter, loc, aTp, spmatHandleTp, tokenTp, token, szm, szk,`。
- **L822 EN**: Executes a standalone statement or declaration: `nseA, rowA, colA, valA, format, enableRT);`.
  **L822 CN**: 执行一条独立语句或声明：`nseA, rowA, colA, valA, format, enableRT);`。
- **L823 EN**: Initializes variable `spMatA` from the right-hand expression.
  **L823 CN**: 使用右侧表达式初始化变量 `spMatA`。
- **L824 EN**: Executes a call or declaration centered on `spGenA->getResult`.
  **L824 CN**: 执行以 `spGenA->getResult` 为核心的调用或声明。
- **L825 EN**: Continues the surrounding expression or declaration: `Operation *spGenB =`.
  **L825 CN**: 继续构造周围的表达式或声明：`Operation *spGenB =`。
- **L826 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genSpMat(rewriter, loc, bTp, spmatHandleTp, tokenTp, token, szk, szn,`.
  **L826 CN**: 继续一个多行参数列表、初始化器或聚合项：`genSpMat(rewriter, loc, bTp, spmatHandleTp, tokenTp, token, szk, szn,`。
- **L827 EN**: Executes a standalone statement or declaration: `nseB, rowB, colB, valB, format, enableRT);`.
  **L827 CN**: 执行一条独立语句或声明：`nseB, rowB, colB, valB, format, enableRT);`。
- **L828 EN**: Initializes variable `spMatB` from the right-hand expression.
  **L828 CN**: 使用右侧表达式初始化变量 `spMatB`。
- **L829 EN**: Executes a call or declaration centered on `spGenB->getResult`.
  **L829 CN**: 执行以 `spGenB->getResult` 为核心的调用或声明。
- **L830 EN**: Blank line separating nearby declarations or logic blocks.
  **L830 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L831 EN**: Comment explains nearby logic, invariants, or intent: `Sparse matrix C materializes (also assumes beta == 0).`.
  **L831 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sparse matrix C materializes (also assumes beta == 0).`。
- **L832 EN**: Initializes variable `zero` from the right-hand expression.
  **L832 CN**: 使用右侧表达式初始化变量 `zero`。
- **L833 EN**: Initializes variable `one` from the right-hand expression.
  **L833 CN**: 使用右侧表达式初始化变量 `one`。
- **L834 EN**: Initializes variable `mplus1` from the right-hand expression.
  **L834 CN**: 使用右侧表达式初始化变量 `mplus1`。
- **L835 EN**: Initializes variable `e1` from the right-hand expression.
  **L835 CN**: 使用右侧表达式初始化变量 `e1`。
- **L836 EN**: Initializes variable `rowC` from the right-hand expression.
  **L836 CN**: 使用右侧表达式初始化变量 `rowC`。
- **L837 EN**: Executes a call or declaration centered on `e1.getAsyncToken`.
  **L837 CN**: 执行以 `e1.getAsyncToken` 为核心的调用或声明。
- **L838 EN**: Initializes variable `e2` from the right-hand expression.
  **L838 CN**: 使用右侧表达式初始化变量 `e2`。
- **L839 EN**: Continues logic associated with callable symbol `getResult`.
  **L839 CN**: 继续与可调用符号 `getResult` 相关的逻辑。
- **L840 EN**: Executes a call or declaration centered on `e2.getAsyncToken`.
  **L840 CN**: 执行以 `e2.getAsyncToken` 为核心的调用或声明。

### Lines 841-864

````cpp
  auto e3 = genAllocBuffer(rewriter, loc, dnCType, zero, token);
  Value valC = e3.getResult(0); // no free needed
  token = e3.getAsyncToken();
  Operation *spGenC =
      genSpMat(rewriter, loc, cTp, spmatHandleTp, tokenTp, token, szm, szn,
               zero, rowC, colC, valC, format, enableRT);
  Value spMatC = spGenC->getResult(0);
  token = spGenC->getResult(1);

  // Precompute buffersizes for SpGEMM.
  Operation *descOp =
      gpu::SpGEMMCreateDescrOp::create(rewriter, loc, descTp, tokenTp, token);
  Value desc = descOp->getResult(0);
  token = descOp->getResult(1);
  Operation *work1 = gpu::SpGEMMWorkEstimationOrComputeOp::create(
      rewriter, loc, indexTp, tokenTp, token, desc,
      gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,
      spMatA, spMatB, spMatC, dnCType, zero, valC,
      gpu::SpGEMMWorkEstimationOrComputeKind::WORK_ESTIMATION);
  Value bufferSz1 = work1->getResult(0);
  token = work1->getResult(1);
  auto buf1 = genAllocBuffer(rewriter, loc, bufferSz1, token);
  Value buffer1 = buf1.getResult(0);
  token = buf1.getAsyncToken();
````
- **L841 EN**: Initializes variable `e3` from the right-hand expression.
  **L841 CN**: 使用右侧表达式初始化变量 `e3`。
- **L842 EN**: Continues logic associated with callable symbol `getResult`.
  **L842 CN**: 继续与可调用符号 `getResult` 相关的逻辑。
- **L843 EN**: Executes a call or declaration centered on `e3.getAsyncToken`.
  **L843 CN**: 执行以 `e3.getAsyncToken` 为核心的调用或声明。
- **L844 EN**: Continues the surrounding expression or declaration: `Operation *spGenC =`.
  **L844 CN**: 继续构造周围的表达式或声明：`Operation *spGenC =`。
- **L845 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genSpMat(rewriter, loc, cTp, spmatHandleTp, tokenTp, token, szm, szn,`.
  **L845 CN**: 继续一个多行参数列表、初始化器或聚合项：`genSpMat(rewriter, loc, cTp, spmatHandleTp, tokenTp, token, szm, szn,`。
- **L846 EN**: Executes a standalone statement or declaration: `zero, rowC, colC, valC, format, enableRT);`.
  **L846 CN**: 执行一条独立语句或声明：`zero, rowC, colC, valC, format, enableRT);`。
- **L847 EN**: Initializes variable `spMatC` from the right-hand expression.
  **L847 CN**: 使用右侧表达式初始化变量 `spMatC`。
- **L848 EN**: Executes a call or declaration centered on `spGenC->getResult`.
  **L848 CN**: 执行以 `spGenC->getResult` 为核心的调用或声明。
- **L849 EN**: Blank line separating nearby declarations or logic blocks.
  **L849 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L850 EN**: Comment explains nearby logic, invariants, or intent: `Precompute buffersizes for SpGEMM.`.
  **L850 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Precompute buffersizes for SpGEMM.`。
- **L851 EN**: Continues the surrounding expression or declaration: `Operation *descOp =`.
  **L851 CN**: 继续构造周围的表达式或声明：`Operation *descOp =`。
- **L852 EN**: Executes a call or declaration centered on `gpu::SpGEMMCreateDescrOp::create`.
  **L852 CN**: 执行以 `gpu::SpGEMMCreateDescrOp::create` 为核心的调用或声明。
- **L853 EN**: Initializes variable `desc` from the right-hand expression.
  **L853 CN**: 使用右侧表达式初始化变量 `desc`。
- **L854 EN**: Executes a call or declaration centered on `descOp->getResult`.
  **L854 CN**: 执行以 `descOp->getResult` 为核心的调用或声明。
- **L855 EN**: Continues logic associated with callable symbol `create`.
  **L855 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, indexTp, tokenTp, token, desc,`.
  **L856 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, indexTp, tokenTp, token, desc,`。
- **L857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,`.
  **L857 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,`。
- **L858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `spMatA, spMatB, spMatC, dnCType, zero, valC,`.
  **L858 CN**: 继续一个多行参数列表、初始化器或聚合项：`spMatA, spMatB, spMatC, dnCType, zero, valC,`。
- **L859 EN**: Executes a standalone statement or declaration: `gpu::SpGEMMWorkEstimationOrComputeKind::WORK_ESTIMATION);`.
  **L859 CN**: 执行一条独立语句或声明：`gpu::SpGEMMWorkEstimationOrComputeKind::WORK_ESTIMATION);`。
- **L860 EN**: Initializes variable `bufferSz1` from the right-hand expression.
  **L860 CN**: 使用右侧表达式初始化变量 `bufferSz1`。
- **L861 EN**: Executes a call or declaration centered on `work1->getResult`.
  **L861 CN**: 执行以 `work1->getResult` 为核心的调用或声明。
- **L862 EN**: Initializes variable `buf1` from the right-hand expression.
  **L862 CN**: 使用右侧表达式初始化变量 `buf1`。
- **L863 EN**: Initializes variable `buffer1` from the right-hand expression.
  **L863 CN**: 使用右侧表达式初始化变量 `buffer1`。
- **L864 EN**: Executes a call or declaration centered on `buf1.getAsyncToken`.
  **L864 CN**: 执行以 `buf1.getAsyncToken` 为核心的调用或声明。

### Lines 865-888

````cpp
  Operation *work2 = gpu::SpGEMMWorkEstimationOrComputeOp::create(
      rewriter, loc, indexTp, tokenTp, token, desc,
      gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,
      spMatA, spMatB, spMatC, dnCType, bufferSz1, buffer1,
      gpu::SpGEMMWorkEstimationOrComputeKind::WORK_ESTIMATION);
  token = work2->getResult(1);

  // Compute step.
  Operation *compute1 = gpu::SpGEMMWorkEstimationOrComputeOp::create(
      rewriter, loc, indexTp, tokenTp, token, desc,
      gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,
      spMatA, spMatB, spMatC, dnCType, zero, valC,
      gpu::SpGEMMWorkEstimationOrComputeKind::COMPUTE);
  Value bufferSz2 = compute1->getResult(0);
  token = compute1->getResult(1);
  auto buf2 = genAllocBuffer(rewriter, loc, bufferSz2, token);
  Value buffer2 = buf2.getResult(0);
  token = buf2.getAsyncToken();
  Operation *compute2 = gpu::SpGEMMWorkEstimationOrComputeOp::create(
      rewriter, loc, indexTp, tokenTp, token, desc,
      gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,
      spMatA, spMatB, spMatC, dnCType, bufferSz2, buffer2,
      gpu::SpGEMMWorkEstimationOrComputeKind::COMPUTE);
  token = compute2->getResult(1);
````
- **L865 EN**: Continues logic associated with callable symbol `create`.
  **L865 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, indexTp, tokenTp, token, desc,`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, indexTp, tokenTp, token, desc,`。
- **L867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,`.
  **L867 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,`。
- **L868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `spMatA, spMatB, spMatC, dnCType, bufferSz1, buffer1,`.
  **L868 CN**: 继续一个多行参数列表、初始化器或聚合项：`spMatA, spMatB, spMatC, dnCType, bufferSz1, buffer1,`。
- **L869 EN**: Executes a standalone statement or declaration: `gpu::SpGEMMWorkEstimationOrComputeKind::WORK_ESTIMATION);`.
  **L869 CN**: 执行一条独立语句或声明：`gpu::SpGEMMWorkEstimationOrComputeKind::WORK_ESTIMATION);`。
- **L870 EN**: Executes a call or declaration centered on `work2->getResult`.
  **L870 CN**: 执行以 `work2->getResult` 为核心的调用或声明。
- **L871 EN**: Blank line separating nearby declarations or logic blocks.
  **L871 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L872 EN**: Comment explains nearby logic, invariants, or intent: `Compute step.`.
  **L872 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Compute step.`。
- **L873 EN**: Continues logic associated with callable symbol `create`.
  **L873 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, indexTp, tokenTp, token, desc,`.
  **L874 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, indexTp, tokenTp, token, desc,`。
- **L875 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,`.
  **L875 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,`。
- **L876 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `spMatA, spMatB, spMatC, dnCType, zero, valC,`.
  **L876 CN**: 继续一个多行参数列表、初始化器或聚合项：`spMatA, spMatB, spMatC, dnCType, zero, valC,`。
- **L877 EN**: Executes a standalone statement or declaration: `gpu::SpGEMMWorkEstimationOrComputeKind::COMPUTE);`.
  **L877 CN**: 执行一条独立语句或声明：`gpu::SpGEMMWorkEstimationOrComputeKind::COMPUTE);`。
- **L878 EN**: Initializes variable `bufferSz2` from the right-hand expression.
  **L878 CN**: 使用右侧表达式初始化变量 `bufferSz2`。
- **L879 EN**: Executes a call or declaration centered on `compute1->getResult`.
  **L879 CN**: 执行以 `compute1->getResult` 为核心的调用或声明。
- **L880 EN**: Initializes variable `buf2` from the right-hand expression.
  **L880 CN**: 使用右侧表达式初始化变量 `buf2`。
- **L881 EN**: Initializes variable `buffer2` from the right-hand expression.
  **L881 CN**: 使用右侧表达式初始化变量 `buffer2`。
- **L882 EN**: Executes a call or declaration centered on `buf2.getAsyncToken`.
  **L882 CN**: 执行以 `buf2.getAsyncToken` 为核心的调用或声明。
- **L883 EN**: Continues logic associated with callable symbol `create`.
  **L883 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, indexTp, tokenTp, token, desc,`.
  **L884 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, indexTp, tokenTp, token, desc,`。
- **L885 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,`.
  **L885 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,`。
- **L886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `spMatA, spMatB, spMatC, dnCType, bufferSz2, buffer2,`.
  **L886 CN**: 继续一个多行参数列表、初始化器或聚合项：`spMatA, spMatB, spMatC, dnCType, bufferSz2, buffer2,`。
- **L887 EN**: Executes a standalone statement or declaration: `gpu::SpGEMMWorkEstimationOrComputeKind::COMPUTE);`.
  **L887 CN**: 执行一条独立语句或声明：`gpu::SpGEMMWorkEstimationOrComputeKind::COMPUTE);`。
- **L888 EN**: Executes a call or declaration centered on `compute2->getResult`.
  **L888 CN**: 执行以 `compute2->getResult` 为核心的调用或声明。

### Lines 889-912

````cpp

  // Get sizes.
  Operation *sizes = gpu::SpMatGetSizeOp::create(
      rewriter, loc, indexTp, indexTp, indexTp, tokenTp, token, spMatC);
  Value nnz = sizes->getResult(2);
  token = sizes->getResult(3);
  auto a2 = genAllocBuffer(rewriter, loc, cTp.getCrdType(), nnz, token);
  colC = a2.getResult(0);
  token = a2.getAsyncToken();
  auto a3 = genAllocBuffer(rewriter, loc, dnCType, nnz, token);
  valC = a3.getResult(0);
  token = a3.getAsyncToken();

  // Update C with new pointers and copy final product back into C.
  Operation *update = gpu::SetCsrPointersOp::create(
      rewriter, loc, tokenTp, token, spMatC, rowC, colC, valC);
  token = update->getResult(0);
  Operation *copy = gpu::SpGEMMCopyOp::create(
      rewriter, loc, tokenTp, token, desc, gpu::TransposeMode::NON_TRANSPOSE,
      gpu::TransposeMode::NON_TRANSPOSE, spMatA, spMatB, spMatC, dnCType);
  token = copy->getResult(0);

  // Allocate buffers on host.
  Value rowH = genHostBuffer(rewriter, loc, cTp.getPosType(), mplus1);
````
- **L889 EN**: Blank line separating nearby declarations or logic blocks.
  **L889 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L890 EN**: Comment explains nearby logic, invariants, or intent: `Get sizes.`.
  **L890 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Get sizes.`。
- **L891 EN**: Continues logic associated with callable symbol `create`.
  **L891 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L892 EN**: Executes a standalone statement or declaration: `rewriter, loc, indexTp, indexTp, indexTp, tokenTp, token, spMatC);`.
  **L892 CN**: 执行一条独立语句或声明：`rewriter, loc, indexTp, indexTp, indexTp, tokenTp, token, spMatC);`。
- **L893 EN**: Initializes variable `nnz` from the right-hand expression.
  **L893 CN**: 使用右侧表达式初始化变量 `nnz`。
- **L894 EN**: Executes a call or declaration centered on `sizes->getResult`.
  **L894 CN**: 执行以 `sizes->getResult` 为核心的调用或声明。
- **L895 EN**: Initializes variable `a2` from the right-hand expression.
  **L895 CN**: 使用右侧表达式初始化变量 `a2`。
- **L896 EN**: Executes a call or declaration centered on `a2.getResult`.
  **L896 CN**: 执行以 `a2.getResult` 为核心的调用或声明。
- **L897 EN**: Executes a call or declaration centered on `a2.getAsyncToken`.
  **L897 CN**: 执行以 `a2.getAsyncToken` 为核心的调用或声明。
- **L898 EN**: Initializes variable `a3` from the right-hand expression.
  **L898 CN**: 使用右侧表达式初始化变量 `a3`。
- **L899 EN**: Executes a call or declaration centered on `a3.getResult`.
  **L899 CN**: 执行以 `a3.getResult` 为核心的调用或声明。
- **L900 EN**: Executes a call or declaration centered on `a3.getAsyncToken`.
  **L900 CN**: 执行以 `a3.getAsyncToken` 为核心的调用或声明。
- **L901 EN**: Blank line separating nearby declarations or logic blocks.
  **L901 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L902 EN**: Comment explains nearby logic, invariants, or intent: `Update C with new pointers and copy final product back into C.`.
  **L902 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Update C with new pointers and copy final product back into C.`。
- **L903 EN**: Continues logic associated with callable symbol `create`.
  **L903 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L904 EN**: Executes a standalone statement or declaration: `rewriter, loc, tokenTp, token, spMatC, rowC, colC, valC);`.
  **L904 CN**: 执行一条独立语句或声明：`rewriter, loc, tokenTp, token, spMatC, rowC, colC, valC);`。
- **L905 EN**: Executes a call or declaration centered on `update->getResult`.
  **L905 CN**: 执行以 `update->getResult` 为核心的调用或声明。
- **L906 EN**: Continues logic associated with callable symbol `create`.
  **L906 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L907 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, tokenTp, token, desc, gpu::TransposeMode::NON_TRANSPOSE,`.
  **L907 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, tokenTp, token, desc, gpu::TransposeMode::NON_TRANSPOSE,`。
- **L908 EN**: Executes a standalone statement or declaration: `gpu::TransposeMode::NON_TRANSPOSE, spMatA, spMatB, spMatC, dnCType);`.
  **L908 CN**: 执行一条独立语句或声明：`gpu::TransposeMode::NON_TRANSPOSE, spMatA, spMatB, spMatC, dnCType);`。
- **L909 EN**: Executes a call or declaration centered on `copy->getResult`.
  **L909 CN**: 执行以 `copy->getResult` 为核心的调用或声明。
- **L910 EN**: Blank line separating nearby declarations or logic blocks.
  **L910 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L911 EN**: Comment explains nearby logic, invariants, or intent: `Allocate buffers on host.`.
  **L911 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Allocate buffers on host.`。
- **L912 EN**: Initializes variable `rowH` from the right-hand expression.
  **L912 CN**: 使用右侧表达式初始化变量 `rowH`。

### Lines 913-936

````cpp
  Value colH = genHostBuffer(rewriter, loc, cTp.getCrdType(), nnz);
  Value valH = genHostBuffer(rewriter, loc, dnCType, nnz);

  // Copy data back to host and free all the resoures.
  token = gpu::SpGEMMDestroyDescrOp::create(rewriter, loc, tokenTp, token, desc)
              .getAsyncToken();
  token = gpu::DestroySpMatOp::create(rewriter, loc, tokenTp, token, spMatA)
              .getAsyncToken();
  token = gpu::DestroySpMatOp::create(rewriter, loc, tokenTp, token, spMatB)
              .getAsyncToken();
  token = gpu::DestroySpMatOp::create(rewriter, loc, tokenTp, token, spMatC)
              .getAsyncToken();
  token = genCopyMemRef(rewriter, loc, rowH, rowC, token);
  token = genCopyMemRef(rewriter, loc, colH, colC, token);
  token = genCopyMemRef(rewriter, loc, valH, valC, token);
  token = genDeallocMemRef(rewriter, loc, rowA, token);
  token = genDeallocMemRef(rewriter, loc, colA, token);
  token = genDeallocMemRef(rewriter, loc, valA, token);
  token = genDeallocMemRef(rewriter, loc, rowB, token);
  token = genDeallocMemRef(rewriter, loc, colB, token);
  token = genDeallocMemRef(rewriter, loc, valB, token);
  token = genDeallocMemRef(rewriter, loc, rowC, token);
  token = genDeallocMemRef(rewriter, loc, colC, token);
  token = genDeallocMemRef(rewriter, loc, valC, token);
````
- **L913 EN**: Initializes variable `colH` from the right-hand expression.
  **L913 CN**: 使用右侧表达式初始化变量 `colH`。
- **L914 EN**: Initializes variable `valH` from the right-hand expression.
  **L914 CN**: 使用右侧表达式初始化变量 `valH`。
- **L915 EN**: Blank line separating nearby declarations or logic blocks.
  **L915 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L916 EN**: Comment explains nearby logic, invariants, or intent: `Copy data back to host and free all the resoures.`.
  **L916 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copy data back to host and free all the resoures.`。
- **L917 EN**: Continues logic associated with callable symbol `create`.
  **L917 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L918 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L918 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L919 EN**: Continues logic associated with callable symbol `create`.
  **L919 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L920 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L920 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L921 EN**: Continues logic associated with callable symbol `create`.
  **L921 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L922 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L922 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L923 EN**: Continues logic associated with callable symbol `create`.
  **L923 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L924 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L924 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L925 EN**: Executes a call or declaration centered on `genCopyMemRef`.
  **L925 CN**: 执行以 `genCopyMemRef` 为核心的调用或声明。
- **L926 EN**: Executes a call or declaration centered on `genCopyMemRef`.
  **L926 CN**: 执行以 `genCopyMemRef` 为核心的调用或声明。
- **L927 EN**: Executes a call or declaration centered on `genCopyMemRef`.
  **L927 CN**: 执行以 `genCopyMemRef` 为核心的调用或声明。
- **L928 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L928 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L929 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L929 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L930 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L930 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L931 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L931 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L932 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L932 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L933 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L933 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L934 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L934 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L935 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L935 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L936 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L936 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。

### Lines 937-960

````cpp
  token = genDeallocMemRef(rewriter, loc, buffer1, token);
  token = genDeallocMemRef(rewriter, loc, buffer2, token);
  tokens.push_back(token);
  genBlockingWait(rewriter, loc, tokens);
  tokens.clear();

  // Done.
  Value vt = bufferization::ToTensorOp::create(
      rewriter, loc, memref::getTensorTypeFromMemRefType(valH.getType()), valH);
  Value rt = bufferization::ToTensorOp::create(
      rewriter, loc, memref::getTensorTypeFromMemRefType(rowH.getType()), rowH);
  Value ct = bufferization::ToTensorOp::create(
      rewriter, loc, memref::getTensorTypeFromMemRefType(colH.getType()), colH);
  rewriter.replaceOpWithNewOp<AssembleOp>(op, c.getType(), ValueRange{rt, ct},
                                          vt);
  return success();
}

// Match and rewrite 2:4 SpMM kernel.
static LogicalResult rewrite2To4SpMM(PatternRewriter &rewriter,
                                     linalg::GenericOp op) {
  Location loc = op.getLoc();
  Value A = op.getOperand(0);
  Value B = op.getOperand(1);
````
- **L937 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L937 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L938 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L938 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L939 EN**: Executes a call or declaration centered on `tokens.push_back`.
  **L939 CN**: 执行以 `tokens.push_back` 为核心的调用或声明。
- **L940 EN**: Executes a call or declaration centered on `genBlockingWait`.
  **L940 CN**: 执行以 `genBlockingWait` 为核心的调用或声明。
- **L941 EN**: Executes a call or declaration centered on `tokens.clear`.
  **L941 CN**: 执行以 `tokens.clear` 为核心的调用或声明。
- **L942 EN**: Blank line separating nearby declarations or logic blocks.
  **L942 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L943 EN**: Comment explains nearby logic, invariants, or intent: `Done.`.
  **L943 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Done.`。
- **L944 EN**: Continues logic associated with callable symbol `create`.
  **L944 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L945 EN**: Executes a call or declaration centered on `memref::getTensorTypeFromMemRefType`.
  **L945 CN**: 执行以 `memref::getTensorTypeFromMemRefType` 为核心的调用或声明。
- **L946 EN**: Continues logic associated with callable symbol `create`.
  **L946 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L947 EN**: Executes a call or declaration centered on `memref::getTensorTypeFromMemRefType`.
  **L947 CN**: 执行以 `memref::getTensorTypeFromMemRefType` 为核心的调用或声明。
- **L948 EN**: Continues logic associated with callable symbol `create`.
  **L948 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L949 EN**: Executes a call or declaration centered on `memref::getTensorTypeFromMemRefType`.
  **L949 CN**: 执行以 `memref::getTensorTypeFromMemRefType` 为核心的调用或声明。
- **L950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter.replaceOpWithNewOp<AssembleOp>(op, c.getType(), ValueRange{rt, ct},`.
  **L950 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter.replaceOpWithNewOp<AssembleOp>(op, c.getType(), ValueRange{rt, ct},`。
- **L951 EN**: Executes a standalone statement or declaration: `vt);`.
  **L951 CN**: 执行一条独立语句或声明：`vt);`。
- **L952 EN**: Returns from the current function with `success()`.
  **L952 CN**: 以 `success()` 从当前函数返回。
- **L953 EN**: Closes the current lexical scope or compound statement.
  **L953 CN**: 结束当前词法作用域或复合语句块。
- **L954 EN**: Blank line separating nearby declarations or logic blocks.
  **L954 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L955 EN**: Comment explains nearby logic, invariants, or intent: `Match and rewrite 2:4 SpMM kernel.`.
  **L955 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Match and rewrite 2:4 SpMM kernel.`。
- **L956 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static LogicalResult rewrite2To4SpMM(PatternRewriter &rewriter,`.
  **L956 CN**: 继续一个多行参数列表、初始化器或聚合项：`static LogicalResult rewrite2To4SpMM(PatternRewriter &rewriter,`。
- **L957 EN**: Continues the surrounding expression or declaration: `linalg::GenericOp op) {`.
  **L957 CN**: 继续构造周围的表达式或声明：`linalg::GenericOp op) {`。
- **L958 EN**: Initializes variable `loc` from the right-hand expression.
  **L958 CN**: 使用右侧表达式初始化变量 `loc`。
- **L959 EN**: Initializes variable `A` from the right-hand expression.
  **L959 CN**: 使用右侧表达式初始化变量 `A`。
- **L960 EN**: Initializes variable `B` from the right-hand expression.
  **L960 CN**: 使用右侧表达式初始化变量 `B`。

### Lines 961-984

````cpp
  Value C = op.getOperand(2); // we have C = AB
  SmallVector<Value> tokens;

  // The cuSparselt API currently only allows pruning and compression
  // to occur on the device. So we recognize the pattern
  //    A' = convert A  ; dense to 2:4
  //    C  = A'B        ; 2:4 matrix mult
  // and then perform compression and matrix multiplication on device.
  auto cnv = A.getDefiningOp<ConvertOp>();
  assert(cnv);
  A = cnv.getSource();

  // All input should be dense tensors.
  if (!isDenseTensor(A) || !isDenseTensor(B) || !isDenseTensor(C))
    return failure();

  // Start sparse kernel and copy data from host to device.
  //   a : bufA -> matA
  //   b : bufB -> matB
  //   c : bufC -> matC
  Value bufA = genTensorToMemref(rewriter, loc, A);
  Value matA = genAllocCopy(rewriter, loc, bufA, tokens);
  Value bufB = genTensorToMemref(rewriter, loc, B);
  Value matB = genAllocCopy(rewriter, loc, bufB, tokens);
````
- **L961 EN**: Continues logic associated with callable symbol `getOperand`.
  **L961 CN**: 继续与可调用符号 `getOperand` 相关的逻辑。
- **L962 EN**: Executes a standalone statement or declaration: `SmallVector<Value> tokens;`.
  **L962 CN**: 执行一条独立语句或声明：`SmallVector<Value> tokens;`。
- **L963 EN**: Blank line separating nearby declarations or logic blocks.
  **L963 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L964 EN**: Comment explains nearby logic, invariants, or intent: `The cuSparselt API currently only allows pruning and compression`.
  **L964 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The cuSparselt API currently only allows pruning and compression`。
- **L965 EN**: Comment explains nearby logic, invariants, or intent: `to occur on the device. So we recognize the pattern`.
  **L965 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to occur on the device. So we recognize the pattern`。
- **L966 EN**: Comment explains nearby logic, invariants, or intent: `A' = convert A  ; dense to 2:4`.
  **L966 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A' = convert A  ; dense to 2:4`。
- **L967 EN**: Comment explains nearby logic, invariants, or intent: `C  = A'B        ; 2:4 matrix mult`.
  **L967 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`C  = A'B        ; 2:4 matrix mult`。
- **L968 EN**: Comment explains nearby logic, invariants, or intent: `and then perform compression and matrix multiplication on device.`.
  **L968 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and then perform compression and matrix multiplication on device.`。
- **L969 EN**: Initializes variable `cnv` from the right-hand expression.
  **L969 CN**: 使用右侧表达式初始化变量 `cnv`。
- **L970 EN**: Checks an internal invariant in debug builds.
  **L970 CN**: 在调试构建中检查内部不变式。
- **L971 EN**: Executes a call or declaration centered on `cnv.getSource`.
  **L971 CN**: 执行以 `cnv.getSource` 为核心的调用或声明。
- **L972 EN**: Blank line separating nearby declarations or logic blocks.
  **L972 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L973 EN**: Comment explains nearby logic, invariants, or intent: `All input should be dense tensors.`.
  **L973 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`All input should be dense tensors.`。
- **L974 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L974 CN**: 开始 `if` 控制流语句并计算其条件。
- **L975 EN**: Returns from the current function with `failure()`.
  **L975 CN**: 以 `failure()` 从当前函数返回。
- **L976 EN**: Blank line separating nearby declarations or logic blocks.
  **L976 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L977 EN**: Comment explains nearby logic, invariants, or intent: `Start sparse kernel and copy data from host to device.`.
  **L977 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Start sparse kernel and copy data from host to device.`。
- **L978 EN**: Comment explains nearby logic, invariants, or intent: `a : bufA -> matA`.
  **L978 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a : bufA -> matA`。
- **L979 EN**: Comment explains nearby logic, invariants, or intent: `b : bufB -> matB`.
  **L979 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`b : bufB -> matB`。
- **L980 EN**: Comment explains nearby logic, invariants, or intent: `c : bufC -> matC`.
  **L980 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`c : bufC -> matC`。
- **L981 EN**: Initializes variable `bufA` from the right-hand expression.
  **L981 CN**: 使用右侧表达式初始化变量 `bufA`。
- **L982 EN**: Initializes variable `matA` from the right-hand expression.
  **L982 CN**: 使用右侧表达式初始化变量 `matA`。
- **L983 EN**: Initializes variable `bufB` from the right-hand expression.
  **L983 CN**: 使用右侧表达式初始化变量 `bufB`。
- **L984 EN**: Initializes variable `matB` from the right-hand expression.
  **L984 CN**: 使用右侧表达式初始化变量 `matB`。

### Lines 985-1008

````cpp
  Value bufC = genTensorToMemref(rewriter, loc, C);
  Value matC = genAllocCopy(rewriter, loc, bufC, tokens);
  genBlockingWait(rewriter, loc, tokens);
  tokens.clear();

  // Create sparse environment and sparse matrix/dense vector handles.
  Value szm = linalg::createOrFoldDimOp(rewriter, loc, matA, 0);
  Value szk = linalg::createOrFoldDimOp(rewriter, loc, matB, 0);
  Value szn = linalg::createOrFoldDimOp(rewriter, loc, matC, 1);
  Type indexTp = rewriter.getIndexType();
  Type dnTensorHandleTp = rewriter.getType<gpu::SparseDnTensorHandleType>();
  Type spMatHandleTp = rewriter.getType<gpu::SparseSpMatHandleType>();
  Type tokenTp = rewriter.getType<gpu::AsyncTokenType>();
  Value token = genFirstWait(rewriter, loc);
  Operation *spGenA = gpu::Create2To4SpMatOp::create(
      rewriter, loc, spMatHandleTp, tokenTp, token, szm, szk,
      gpu::Prune2To4SpMatFlag::PRUNE_AND_CHECK, matA);
  Value spMatA = spGenA->getResult(0);
  token = spGenA->getResult(1);
  auto dmatB =
      gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp, tokenTp,
                                    token, matB, SmallVector<Value>{szk, szn});
  Value dnB = dmatB.getResult(0);
  token = dmatB.getAsyncToken();
````
- **L985 EN**: Initializes variable `bufC` from the right-hand expression.
  **L985 CN**: 使用右侧表达式初始化变量 `bufC`。
- **L986 EN**: Initializes variable `matC` from the right-hand expression.
  **L986 CN**: 使用右侧表达式初始化变量 `matC`。
- **L987 EN**: Executes a call or declaration centered on `genBlockingWait`.
  **L987 CN**: 执行以 `genBlockingWait` 为核心的调用或声明。
- **L988 EN**: Executes a call or declaration centered on `tokens.clear`.
  **L988 CN**: 执行以 `tokens.clear` 为核心的调用或声明。
- **L989 EN**: Blank line separating nearby declarations or logic blocks.
  **L989 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L990 EN**: Comment explains nearby logic, invariants, or intent: `Create sparse environment and sparse matrix/dense vector handles.`.
  **L990 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create sparse environment and sparse matrix/dense vector handles.`。
- **L991 EN**: Initializes variable `szm` from the right-hand expression.
  **L991 CN**: 使用右侧表达式初始化变量 `szm`。
- **L992 EN**: Initializes variable `szk` from the right-hand expression.
  **L992 CN**: 使用右侧表达式初始化变量 `szk`。
- **L993 EN**: Initializes variable `szn` from the right-hand expression.
  **L993 CN**: 使用右侧表达式初始化变量 `szn`。
- **L994 EN**: Initializes variable `indexTp` from the right-hand expression.
  **L994 CN**: 使用右侧表达式初始化变量 `indexTp`。
- **L995 EN**: Initializes variable `dnTensorHandleTp` from the right-hand expression.
  **L995 CN**: 使用右侧表达式初始化变量 `dnTensorHandleTp`。
- **L996 EN**: Initializes variable `spMatHandleTp` from the right-hand expression.
  **L996 CN**: 使用右侧表达式初始化变量 `spMatHandleTp`。
- **L997 EN**: Initializes variable `tokenTp` from the right-hand expression.
  **L997 CN**: 使用右侧表达式初始化变量 `tokenTp`。
- **L998 EN**: Initializes variable `token` from the right-hand expression.
  **L998 CN**: 使用右侧表达式初始化变量 `token`。
- **L999 EN**: Continues logic associated with callable symbol `create`.
  **L999 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, spMatHandleTp, tokenTp, token, szm, szk,`.
  **L1000 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, spMatHandleTp, tokenTp, token, szm, szk,`。
- **L1001 EN**: Executes a standalone statement or declaration: `gpu::Prune2To4SpMatFlag::PRUNE_AND_CHECK, matA);`.
  **L1001 CN**: 执行一条独立语句或声明：`gpu::Prune2To4SpMatFlag::PRUNE_AND_CHECK, matA);`。
- **L1002 EN**: Initializes variable `spMatA` from the right-hand expression.
  **L1002 CN**: 使用右侧表达式初始化变量 `spMatA`。
- **L1003 EN**: Executes a call or declaration centered on `spGenA->getResult`.
  **L1003 CN**: 执行以 `spGenA->getResult` 为核心的调用或声明。
- **L1004 EN**: Continues the surrounding expression or declaration: `auto dmatB =`.
  **L1004 CN**: 继续构造周围的表达式或声明：`auto dmatB =`。
- **L1005 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp, tokenTp,`.
  **L1005 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp, tokenTp,`。
- **L1006 EN**: Executes a standalone statement or declaration: `token, matB, SmallVector<Value>{szk, szn});`.
  **L1006 CN**: 执行一条独立语句或声明：`token, matB, SmallVector<Value>{szk, szn});`。
- **L1007 EN**: Initializes variable `dnB` from the right-hand expression.
  **L1007 CN**: 使用右侧表达式初始化变量 `dnB`。
- **L1008 EN**: Executes a call or declaration centered on `dmatB.getAsyncToken`.
  **L1008 CN**: 执行以 `dmatB.getAsyncToken` 为核心的调用或声明。

### Lines 1009-1032

````cpp
  auto dmatC =
      gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp, tokenTp,
                                    token, matC, SmallVector<Value>{szm, szn});
  Value dnC = dmatC.getResult(0);
  token = dmatC.getAsyncToken();
  auto dmatCType = llvm::cast<ShapedType>(matC.getType()).getElementType();

  // Precompute buffersize for SpMM.
  SmallVector<Type> bufferTypes_{indexTp, indexTp, indexTp};
  TypeRange bufferTypes(bufferTypes_);
  auto bufferComp = gpu::SpMMBufferSizeOp::create(
      rewriter, loc, bufferTypes, tokenTp, token,
      gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,
      spMatA, dnB, dnC,
      /*computeType=*/dmatCType);
  token = bufferComp.getAsyncToken();

  // Allocate buffers on host.
  Value bufferSz1 = bufferComp.getResult(0);
  auto buf1 = genAllocBuffer(rewriter, loc, bufferSz1, token);
  Value buffer1 = buf1.getResult(0);
  token = buf1.getAsyncToken();
  Value bufferSz2 = bufferComp.getResult(1);
  auto buf2 = genAllocBuffer(rewriter, loc, bufferSz2, token);
````
- **L1009 EN**: Continues the surrounding expression or declaration: `auto dmatC =`.
  **L1009 CN**: 继续构造周围的表达式或声明：`auto dmatC =`。
- **L1010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp, tokenTp,`.
  **L1010 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::CreateDnTensorOp::create(rewriter, loc, dnTensorHandleTp, tokenTp,`。
- **L1011 EN**: Executes a standalone statement or declaration: `token, matC, SmallVector<Value>{szm, szn});`.
  **L1011 CN**: 执行一条独立语句或声明：`token, matC, SmallVector<Value>{szm, szn});`。
- **L1012 EN**: Initializes variable `dnC` from the right-hand expression.
  **L1012 CN**: 使用右侧表达式初始化变量 `dnC`。
- **L1013 EN**: Executes a call or declaration centered on `dmatC.getAsyncToken`.
  **L1013 CN**: 执行以 `dmatC.getAsyncToken` 为核心的调用或声明。
- **L1014 EN**: Initializes variable `dmatCType` from the right-hand expression.
  **L1014 CN**: 使用右侧表达式初始化变量 `dmatCType`。
- **L1015 EN**: Blank line separating nearby declarations or logic blocks.
  **L1015 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1016 EN**: Comment explains nearby logic, invariants, or intent: `Precompute buffersize for SpMM.`.
  **L1016 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Precompute buffersize for SpMM.`。
- **L1017 EN**: Executes a standalone statement or declaration: `SmallVector<Type> bufferTypes_{indexTp, indexTp, indexTp};`.
  **L1017 CN**: 执行一条独立语句或声明：`SmallVector<Type> bufferTypes_{indexTp, indexTp, indexTp};`。
- **L1018 EN**: Executes a call or declaration centered on `bufferTypes`.
  **L1018 CN**: 执行以 `bufferTypes` 为核心的调用或声明。
- **L1019 EN**: Continues logic associated with callable symbol `create`.
  **L1019 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1020 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, bufferTypes, tokenTp, token,`.
  **L1020 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, bufferTypes, tokenTp, token,`。
- **L1021 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,`.
  **L1021 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::TransposeMode::NON_TRANSPOSE, gpu::TransposeMode::NON_TRANSPOSE,`。
- **L1022 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `spMatA, dnB, dnC,`.
  **L1022 CN**: 继续一个多行参数列表、初始化器或聚合项：`spMatA, dnB, dnC,`。
- **L1023 EN**: Comment explains nearby logic, invariants, or intent: `computeType=*/dmatCType);`.
  **L1023 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`computeType=*/dmatCType);`。
- **L1024 EN**: Executes a call or declaration centered on `bufferComp.getAsyncToken`.
  **L1024 CN**: 执行以 `bufferComp.getAsyncToken` 为核心的调用或声明。
- **L1025 EN**: Blank line separating nearby declarations or logic blocks.
  **L1025 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1026 EN**: Comment explains nearby logic, invariants, or intent: `Allocate buffers on host.`.
  **L1026 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Allocate buffers on host.`。
- **L1027 EN**: Initializes variable `bufferSz1` from the right-hand expression.
  **L1027 CN**: 使用右侧表达式初始化变量 `bufferSz1`。
- **L1028 EN**: Initializes variable `buf1` from the right-hand expression.
  **L1028 CN**: 使用右侧表达式初始化变量 `buf1`。
- **L1029 EN**: Initializes variable `buffer1` from the right-hand expression.
  **L1029 CN**: 使用右侧表达式初始化变量 `buffer1`。
- **L1030 EN**: Executes a call or declaration centered on `buf1.getAsyncToken`.
  **L1030 CN**: 执行以 `buf1.getAsyncToken` 为核心的调用或声明。
- **L1031 EN**: Initializes variable `bufferSz2` from the right-hand expression.
  **L1031 CN**: 使用右侧表达式初始化变量 `bufferSz2`。
- **L1032 EN**: Initializes variable `buf2` from the right-hand expression.
  **L1032 CN**: 使用右侧表达式初始化变量 `buf2`。

### Lines 1033-1056

````cpp
  Value buffer2 = buf2.getResult(0);
  token = buf2.getAsyncToken();
  Value bufferSz3 = bufferComp.getResult(2);
  auto buf3 = genAllocBuffer(rewriter, loc, bufferSz3, token);
  Value buffer3 = buf3.getResult(0);
  token = buf3.getAsyncToken();

  // Perform the SpMM.
  auto dnCType = llvm::cast<ShapedType>(matC.getType()).getElementType();
  auto spmmComp = gpu::SpMMOp::create(
      rewriter, loc, tokenTp, token, spMatA, dnB, dnC, /*computeType=*/dnCType,
      SmallVector<Value>{buffer1, buffer2, buffer3});
  token = spmmComp.getAsyncToken();

  // Copy data back to host and free all the resources.
  token = gpu::DestroySpMatOp::create(rewriter, loc, tokenTp, token, spMatA)
              .getAsyncToken();
  token = gpu::DestroyDnTensorOp::create(rewriter, loc, tokenTp, token, dnB)
              .getAsyncToken();
  token = gpu::DestroyDnTensorOp::create(rewriter, loc, tokenTp, token, dnC)
              .getAsyncToken();
  token = genDeallocMemRef(rewriter, loc, buffer1, token);
  token = genDeallocMemRef(rewriter, loc, buffer2, token);
  token = genDeallocMemRef(rewriter, loc, buffer3, token);
````
- **L1033 EN**: Initializes variable `buffer2` from the right-hand expression.
  **L1033 CN**: 使用右侧表达式初始化变量 `buffer2`。
- **L1034 EN**: Executes a call or declaration centered on `buf2.getAsyncToken`.
  **L1034 CN**: 执行以 `buf2.getAsyncToken` 为核心的调用或声明。
- **L1035 EN**: Initializes variable `bufferSz3` from the right-hand expression.
  **L1035 CN**: 使用右侧表达式初始化变量 `bufferSz3`。
- **L1036 EN**: Initializes variable `buf3` from the right-hand expression.
  **L1036 CN**: 使用右侧表达式初始化变量 `buf3`。
- **L1037 EN**: Initializes variable `buffer3` from the right-hand expression.
  **L1037 CN**: 使用右侧表达式初始化变量 `buffer3`。
- **L1038 EN**: Executes a call or declaration centered on `buf3.getAsyncToken`.
  **L1038 CN**: 执行以 `buf3.getAsyncToken` 为核心的调用或声明。
- **L1039 EN**: Blank line separating nearby declarations or logic blocks.
  **L1039 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1040 EN**: Comment explains nearby logic, invariants, or intent: `Perform the SpMM.`.
  **L1040 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Perform the SpMM.`。
- **L1041 EN**: Initializes variable `dnCType` from the right-hand expression.
  **L1041 CN**: 使用右侧表达式初始化变量 `dnCType`。
- **L1042 EN**: Continues logic associated with callable symbol `create`.
  **L1042 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1043 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `rewriter, loc, tokenTp, token, spMatA, dnB, dnC, /*computeType=*/dnCType,`.
  **L1043 CN**: 继续一个多行参数列表、初始化器或聚合项：`rewriter, loc, tokenTp, token, spMatA, dnB, dnC, /*computeType=*/dnCType,`。
- **L1044 EN**: Executes a standalone statement or declaration: `SmallVector<Value>{buffer1, buffer2, buffer3});`.
  **L1044 CN**: 执行一条独立语句或声明：`SmallVector<Value>{buffer1, buffer2, buffer3});`。
- **L1045 EN**: Executes a call or declaration centered on `spmmComp.getAsyncToken`.
  **L1045 CN**: 执行以 `spmmComp.getAsyncToken` 为核心的调用或声明。
- **L1046 EN**: Blank line separating nearby declarations or logic blocks.
  **L1046 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1047 EN**: Comment explains nearby logic, invariants, or intent: `Copy data back to host and free all the resources.`.
  **L1047 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copy data back to host and free all the resources.`。
- **L1048 EN**: Continues logic associated with callable symbol `create`.
  **L1048 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1049 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L1049 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L1050 EN**: Continues logic associated with callable symbol `create`.
  **L1050 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1051 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L1051 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L1052 EN**: Continues logic associated with callable symbol `create`.
  **L1052 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1053 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L1053 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L1054 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L1054 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L1055 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L1055 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L1056 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L1056 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。

### Lines 1057-1080

````cpp
  token = genDeallocMemRef(rewriter, loc, matA, token);
  token = genDeallocMemRef(rewriter, loc, matB, token);
  token = genCopyMemRef(rewriter, loc, bufC, matC, token);
  token = genDeallocMemRef(rewriter, loc, matC, token);
  tokens.push_back(token);
  genBlockingWait(rewriter, loc, tokens);
  tokens.clear();

  // Done.
  rewriter.replaceOpWithNewOp<bufferization::ToTensorOp>(op, C.getType(), bufC);
  return success();
}

/// Match and rewrite SDDMM kernel.
static LogicalResult rewriteSDDMM(PatternRewriter &rewriter,
                                  linalg::GenericOp op, bool enableRT) {
  Location loc = op.getLoc();
  Value a = op.getOperand(0);
  Value b = op.getOperand(1);
  Value c = op.getOperand(2);
  SmallVector<Value> tokens;

  // Only admissible sparse matrix format (no COO/CSC) and dense matrices.
  SparseTensorType aTp = getSparseTensorType(a);
````
- **L1057 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L1057 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L1058 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L1058 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L1059 EN**: Executes a call or declaration centered on `genCopyMemRef`.
  **L1059 CN**: 执行以 `genCopyMemRef` 为核心的调用或声明。
- **L1060 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L1060 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L1061 EN**: Executes a call or declaration centered on `tokens.push_back`.
  **L1061 CN**: 执行以 `tokens.push_back` 为核心的调用或声明。
- **L1062 EN**: Executes a call or declaration centered on `genBlockingWait`.
  **L1062 CN**: 执行以 `genBlockingWait` 为核心的调用或声明。
- **L1063 EN**: Executes a call or declaration centered on `tokens.clear`.
  **L1063 CN**: 执行以 `tokens.clear` 为核心的调用或声明。
- **L1064 EN**: Blank line separating nearby declarations or logic blocks.
  **L1064 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1065 EN**: Comment explains nearby logic, invariants, or intent: `Done.`.
  **L1065 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Done.`。
- **L1066 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithNewOp<bufferization::ToTensorOp>`.
  **L1066 CN**: 执行以 `rewriter.replaceOpWithNewOp<bufferization::ToTensorOp>` 为核心的调用或声明。
- **L1067 EN**: Returns from the current function with `success()`.
  **L1067 CN**: 以 `success()` 从当前函数返回。
- **L1068 EN**: Closes the current lexical scope or compound statement.
  **L1068 CN**: 结束当前词法作用域或复合语句块。
- **L1069 EN**: Blank line separating nearby declarations or logic blocks.
  **L1069 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1070 EN**: Comment explains nearby logic, invariants, or intent: `Match and rewrite SDDMM kernel.`.
  **L1070 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Match and rewrite SDDMM kernel.`。
- **L1071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static LogicalResult rewriteSDDMM(PatternRewriter &rewriter,`.
  **L1071 CN**: 继续一个多行参数列表、初始化器或聚合项：`static LogicalResult rewriteSDDMM(PatternRewriter &rewriter,`。
- **L1072 EN**: Continues the surrounding expression or declaration: `linalg::GenericOp op, bool enableRT) {`.
  **L1072 CN**: 继续构造周围的表达式或声明：`linalg::GenericOp op, bool enableRT) {`。
- **L1073 EN**: Initializes variable `loc` from the right-hand expression.
  **L1073 CN**: 使用右侧表达式初始化变量 `loc`。
- **L1074 EN**: Initializes variable `a` from the right-hand expression.
  **L1074 CN**: 使用右侧表达式初始化变量 `a`。
- **L1075 EN**: Initializes variable `b` from the right-hand expression.
  **L1075 CN**: 使用右侧表达式初始化变量 `b`。
- **L1076 EN**: Initializes variable `c` from the right-hand expression.
  **L1076 CN**: 使用右侧表达式初始化变量 `c`。
- **L1077 EN**: Executes a standalone statement or declaration: `SmallVector<Value> tokens;`.
  **L1077 CN**: 执行一条独立语句或声明：`SmallVector<Value> tokens;`。
- **L1078 EN**: Blank line separating nearby declarations or logic blocks.
  **L1078 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1079 EN**: Comment explains nearby logic, invariants, or intent: `Only admissible sparse matrix format (no COO/CSC) and dense matrices.`.
  **L1079 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Only admissible sparse matrix format (no COO/CSC) and dense matrices.`。
- **L1080 EN**: Initializes variable `aTp` from the right-hand expression.
  **L1080 CN**: 使用右侧表达式初始化变量 `aTp`。

### Lines 1081-1104

````cpp
  SparseTensorType bTp = getSparseTensorType(b);
  SparseTensorType cTp = getSparseTensorType(c);
  auto format = getCuSparseFormat(cTp, bTp, aTp, enableRT, /*isMatVec=*/false);
  if (format == CuSparseFormat::kNone || format == CuSparseFormat::kCOO ||
      format == CuSparseFormat::kCSC)
    return failure();

  // The SDDMM does the in-place operation.
  // Start sparse kernel and copy data from host to device.
  //   a : bufA           -> matA
  //   b : bufB           -> matB
  //   c : memR/memC/memV -> rowC,colC,valC
  Value nseC = NumberOfEntriesOp::create(rewriter, loc, c);
  Value szm = linalg::createOrFoldDimOp(rewriter, loc, a, 0);
  Value szk = linalg::createOrFoldDimOp(rewriter, loc, a, 1);
  Value szn = linalg::createOrFoldDimOp(rewriter, loc, b, 1);
  Value bufA = genTensorToMemref(rewriter, loc, a);
  Value matA = genAllocCopy(rewriter, loc, bufA, tokens);
  Value bufB = genTensorToMemref(rewriter, loc, b);
  Value matB = genAllocCopy(rewriter, loc, bufB, tokens);
  Value memR = genFirstPosOrCrds(rewriter, loc, c, format, enableRT);
  Value memC = genSecondCrds(rewriter, loc, c, format, enableRT); // or empty
  Value memV = ToValuesOp::create(rewriter, loc, c);
  Value rowC = genAllocCopy(rewriter, loc, memR, tokens);
````
- **L1081 EN**: Initializes variable `bTp` from the right-hand expression.
  **L1081 CN**: 使用右侧表达式初始化变量 `bTp`。
- **L1082 EN**: Initializes variable `cTp` from the right-hand expression.
  **L1082 CN**: 使用右侧表达式初始化变量 `cTp`。
- **L1083 EN**: Initializes variable `format` from the right-hand expression.
  **L1083 CN**: 使用右侧表达式初始化变量 `format`。
- **L1084 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1084 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1085 EN**: Continues the surrounding expression or declaration: `format == CuSparseFormat::kCSC)`.
  **L1085 CN**: 继续构造周围的表达式或声明：`format == CuSparseFormat::kCSC)`。
- **L1086 EN**: Returns from the current function with `failure()`.
  **L1086 CN**: 以 `failure()` 从当前函数返回。
- **L1087 EN**: Blank line separating nearby declarations or logic blocks.
  **L1087 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1088 EN**: Comment explains nearby logic, invariants, or intent: `The SDDMM does the in-place operation.`.
  **L1088 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The SDDMM does the in-place operation.`。
- **L1089 EN**: Comment explains nearby logic, invariants, or intent: `Start sparse kernel and copy data from host to device.`.
  **L1089 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Start sparse kernel and copy data from host to device.`。
- **L1090 EN**: Comment explains nearby logic, invariants, or intent: `a : bufA           -> matA`.
  **L1090 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a : bufA           -> matA`。
- **L1091 EN**: Comment explains nearby logic, invariants, or intent: `b : bufB           -> matB`.
  **L1091 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`b : bufB           -> matB`。
- **L1092 EN**: Comment explains nearby logic, invariants, or intent: `c : memR/memC/memV -> rowC,colC,valC`.
  **L1092 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`c : memR/memC/memV -> rowC,colC,valC`。
- **L1093 EN**: Initializes variable `nseC` from the right-hand expression.
  **L1093 CN**: 使用右侧表达式初始化变量 `nseC`。
- **L1094 EN**: Initializes variable `szm` from the right-hand expression.
  **L1094 CN**: 使用右侧表达式初始化变量 `szm`。
- **L1095 EN**: Initializes variable `szk` from the right-hand expression.
  **L1095 CN**: 使用右侧表达式初始化变量 `szk`。
- **L1096 EN**: Initializes variable `szn` from the right-hand expression.
  **L1096 CN**: 使用右侧表达式初始化变量 `szn`。
- **L1097 EN**: Initializes variable `bufA` from the right-hand expression.
  **L1097 CN**: 使用右侧表达式初始化变量 `bufA`。
- **L1098 EN**: Initializes variable `matA` from the right-hand expression.
  **L1098 CN**: 使用右侧表达式初始化变量 `matA`。
- **L1099 EN**: Initializes variable `bufB` from the right-hand expression.
  **L1099 CN**: 使用右侧表达式初始化变量 `bufB`。
- **L1100 EN**: Initializes variable `matB` from the right-hand expression.
  **L1100 CN**: 使用右侧表达式初始化变量 `matB`。
- **L1101 EN**: Initializes variable `memR` from the right-hand expression.
  **L1101 CN**: 使用右侧表达式初始化变量 `memR`。
- **L1102 EN**: Continues logic associated with callable symbol `genSecondCrds`.
  **L1102 CN**: 继续与可调用符号 `genSecondCrds` 相关的逻辑。
- **L1103 EN**: Initializes variable `memV` from the right-hand expression.
  **L1103 CN**: 使用右侧表达式初始化变量 `memV`。
- **L1104 EN**: Initializes variable `rowC` from the right-hand expression.
  **L1104 CN**: 使用右侧表达式初始化变量 `rowC`。

### Lines 1105-1128

````cpp
  Value colC = memC ? genAllocCopy(rewriter, loc, memC, tokens) : Value();
  Value valC = genAllocCopy(rewriter, loc, memV, tokens);
  genBlockingWait(rewriter, loc, tokens);
  tokens.clear();

  // Create sparse environment and sparse matrix/dense matrix handles.
  Type indexTp = rewriter.getIndexType();
  Type dnMatHandleTp = rewriter.getType<gpu::SparseDnTensorHandleType>();
  Type spMatHandleTp = rewriter.getType<gpu::SparseSpMatHandleType>();
  Type tokenTp = rewriter.getType<gpu::AsyncTokenType>();
  Value token = genFirstWait(rewriter, loc);
  auto dmatA =
      gpu::CreateDnTensorOp::create(rewriter, loc, dnMatHandleTp, tokenTp,
                                    token, matA, SmallVector<Value>{szm, szk});
  Value dnA = dmatA.getResult(0);
  token = dmatA.getAsyncToken();
  auto dmatB =
      gpu::CreateDnTensorOp::create(rewriter, loc, dnMatHandleTp, tokenTp,
                                    token, matB, SmallVector<Value>{szk, szn});
  Value dnB = dmatB.getResult(0);
  token = dmatB.getAsyncToken();
  Operation *spGenC =
      genSpMat(rewriter, loc, cTp, spMatHandleTp, tokenTp, token, szm, szn,
               nseC, rowC, colC, valC, format, enableRT);
````
- **L1105 EN**: Initializes variable `colC` from the right-hand expression.
  **L1105 CN**: 使用右侧表达式初始化变量 `colC`。
- **L1106 EN**: Initializes variable `valC` from the right-hand expression.
  **L1106 CN**: 使用右侧表达式初始化变量 `valC`。
- **L1107 EN**: Executes a call or declaration centered on `genBlockingWait`.
  **L1107 CN**: 执行以 `genBlockingWait` 为核心的调用或声明。
- **L1108 EN**: Executes a call or declaration centered on `tokens.clear`.
  **L1108 CN**: 执行以 `tokens.clear` 为核心的调用或声明。
- **L1109 EN**: Blank line separating nearby declarations or logic blocks.
  **L1109 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1110 EN**: Comment explains nearby logic, invariants, or intent: `Create sparse environment and sparse matrix/dense matrix handles.`.
  **L1110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create sparse environment and sparse matrix/dense matrix handles.`。
- **L1111 EN**: Initializes variable `indexTp` from the right-hand expression.
  **L1111 CN**: 使用右侧表达式初始化变量 `indexTp`。
- **L1112 EN**: Initializes variable `dnMatHandleTp` from the right-hand expression.
  **L1112 CN**: 使用右侧表达式初始化变量 `dnMatHandleTp`。
- **L1113 EN**: Initializes variable `spMatHandleTp` from the right-hand expression.
  **L1113 CN**: 使用右侧表达式初始化变量 `spMatHandleTp`。
- **L1114 EN**: Initializes variable `tokenTp` from the right-hand expression.
  **L1114 CN**: 使用右侧表达式初始化变量 `tokenTp`。
- **L1115 EN**: Initializes variable `token` from the right-hand expression.
  **L1115 CN**: 使用右侧表达式初始化变量 `token`。
- **L1116 EN**: Continues the surrounding expression or declaration: `auto dmatA =`.
  **L1116 CN**: 继续构造周围的表达式或声明：`auto dmatA =`。
- **L1117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::CreateDnTensorOp::create(rewriter, loc, dnMatHandleTp, tokenTp,`.
  **L1117 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::CreateDnTensorOp::create(rewriter, loc, dnMatHandleTp, tokenTp,`。
- **L1118 EN**: Executes a standalone statement or declaration: `token, matA, SmallVector<Value>{szm, szk});`.
  **L1118 CN**: 执行一条独立语句或声明：`token, matA, SmallVector<Value>{szm, szk});`。
- **L1119 EN**: Initializes variable `dnA` from the right-hand expression.
  **L1119 CN**: 使用右侧表达式初始化变量 `dnA`。
- **L1120 EN**: Executes a call or declaration centered on `dmatA.getAsyncToken`.
  **L1120 CN**: 执行以 `dmatA.getAsyncToken` 为核心的调用或声明。
- **L1121 EN**: Continues the surrounding expression or declaration: `auto dmatB =`.
  **L1121 CN**: 继续构造周围的表达式或声明：`auto dmatB =`。
- **L1122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `gpu::CreateDnTensorOp::create(rewriter, loc, dnMatHandleTp, tokenTp,`.
  **L1122 CN**: 继续一个多行参数列表、初始化器或聚合项：`gpu::CreateDnTensorOp::create(rewriter, loc, dnMatHandleTp, tokenTp,`。
- **L1123 EN**: Executes a standalone statement or declaration: `token, matB, SmallVector<Value>{szk, szn});`.
  **L1123 CN**: 执行一条独立语句或声明：`token, matB, SmallVector<Value>{szk, szn});`。
- **L1124 EN**: Initializes variable `dnB` from the right-hand expression.
  **L1124 CN**: 使用右侧表达式初始化变量 `dnB`。
- **L1125 EN**: Executes a call or declaration centered on `dmatB.getAsyncToken`.
  **L1125 CN**: 执行以 `dmatB.getAsyncToken` 为核心的调用或声明。
- **L1126 EN**: Continues the surrounding expression or declaration: `Operation *spGenC =`.
  **L1126 CN**: 继续构造周围的表达式或声明：`Operation *spGenC =`。
- **L1127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genSpMat(rewriter, loc, cTp, spMatHandleTp, tokenTp, token, szm, szn,`.
  **L1127 CN**: 继续一个多行参数列表、初始化器或聚合项：`genSpMat(rewriter, loc, cTp, spMatHandleTp, tokenTp, token, szm, szn,`。
- **L1128 EN**: Executes a standalone statement or declaration: `nseC, rowC, colC, valC, format, enableRT);`.
  **L1128 CN**: 执行一条独立语句或声明：`nseC, rowC, colC, valC, format, enableRT);`。

### Lines 1129-1152

````cpp
  Value spMatC = spGenC->getResult(0);
  token = spGenC->getResult(1);
  auto dnCType = llvm::cast<ShapedType>(c.getType()).getElementType();

  // Precompute buffersize for SDDMM.
  auto bufferComp = gpu::SDDMMBufferSizeOp::create(
      rewriter, loc, indexTp, tokenTp, token, dnA, dnB, spMatC, dnCType);
  Value bufferSz = bufferComp.getResult(0);
  token = bufferComp.getAsyncToken();
  auto buf = genAllocBuffer(rewriter, loc, bufferSz, token);
  Value buffer = buf.getResult(0);
  token = buf.getAsyncToken();

  // Perform the SDDMM.
  auto sddmmComp = gpu::SDDMMOp::create(rewriter, loc, tokenTp, token, dnA, dnB,
                                        spMatC, dnCType, buffer);
  token = sddmmComp.getAsyncToken();

  // Copy data back to host and free all the resoures.
  token = gpu::DestroyDnTensorOp::create(rewriter, loc, tokenTp, token, dnA)
              .getAsyncToken();
  token = gpu::DestroyDnTensorOp::create(rewriter, loc, tokenTp, token, dnB)
              .getAsyncToken();
  token = gpu::DestroySpMatOp::create(rewriter, loc, tokenTp, token, spMatC)
````
- **L1129 EN**: Initializes variable `spMatC` from the right-hand expression.
  **L1129 CN**: 使用右侧表达式初始化变量 `spMatC`。
- **L1130 EN**: Executes a call or declaration centered on `spGenC->getResult`.
  **L1130 CN**: 执行以 `spGenC->getResult` 为核心的调用或声明。
- **L1131 EN**: Initializes variable `dnCType` from the right-hand expression.
  **L1131 CN**: 使用右侧表达式初始化变量 `dnCType`。
- **L1132 EN**: Blank line separating nearby declarations or logic blocks.
  **L1132 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1133 EN**: Comment explains nearby logic, invariants, or intent: `Precompute buffersize for SDDMM.`.
  **L1133 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Precompute buffersize for SDDMM.`。
- **L1134 EN**: Continues logic associated with callable symbol `create`.
  **L1134 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1135 EN**: Executes a standalone statement or declaration: `rewriter, loc, indexTp, tokenTp, token, dnA, dnB, spMatC, dnCType);`.
  **L1135 CN**: 执行一条独立语句或声明：`rewriter, loc, indexTp, tokenTp, token, dnA, dnB, spMatC, dnCType);`。
- **L1136 EN**: Initializes variable `bufferSz` from the right-hand expression.
  **L1136 CN**: 使用右侧表达式初始化变量 `bufferSz`。
- **L1137 EN**: Executes a call or declaration centered on `bufferComp.getAsyncToken`.
  **L1137 CN**: 执行以 `bufferComp.getAsyncToken` 为核心的调用或声明。
- **L1138 EN**: Initializes variable `buf` from the right-hand expression.
  **L1138 CN**: 使用右侧表达式初始化变量 `buf`。
- **L1139 EN**: Initializes variable `buffer` from the right-hand expression.
  **L1139 CN**: 使用右侧表达式初始化变量 `buffer`。
- **L1140 EN**: Executes a call or declaration centered on `buf.getAsyncToken`.
  **L1140 CN**: 执行以 `buf.getAsyncToken` 为核心的调用或声明。
- **L1141 EN**: Blank line separating nearby declarations or logic blocks.
  **L1141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1142 EN**: Comment explains nearby logic, invariants, or intent: `Perform the SDDMM.`.
  **L1142 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Perform the SDDMM.`。
- **L1143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `auto sddmmComp = gpu::SDDMMOp::create(rewriter, loc, tokenTp, token, dnA, dnB,`.
  **L1143 CN**: 继续一个多行参数列表、初始化器或聚合项：`auto sddmmComp = gpu::SDDMMOp::create(rewriter, loc, tokenTp, token, dnA, dnB,`。
- **L1144 EN**: Executes a standalone statement or declaration: `spMatC, dnCType, buffer);`.
  **L1144 CN**: 执行一条独立语句或声明：`spMatC, dnCType, buffer);`。
- **L1145 EN**: Executes a call or declaration centered on `sddmmComp.getAsyncToken`.
  **L1145 CN**: 执行以 `sddmmComp.getAsyncToken` 为核心的调用或声明。
- **L1146 EN**: Blank line separating nearby declarations or logic blocks.
  **L1146 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1147 EN**: Comment explains nearby logic, invariants, or intent: `Copy data back to host and free all the resoures.`.
  **L1147 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copy data back to host and free all the resoures.`。
- **L1148 EN**: Continues logic associated with callable symbol `create`.
  **L1148 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1149 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L1149 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L1150 EN**: Continues logic associated with callable symbol `create`.
  **L1150 CN**: 继续与可调用符号 `create` 相关的逻辑。
- **L1151 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L1151 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L1152 EN**: Continues logic associated with callable symbol `create`.
  **L1152 CN**: 继续与可调用符号 `create` 相关的逻辑。

### Lines 1153-1176

````cpp
              .getAsyncToken();
  token = genDeallocMemRef(rewriter, loc, buffer, token);
  token = genDeallocMemRef(rewriter, loc, matA, token);
  token = genDeallocMemRef(rewriter, loc, matB, token);
  token = genDeallocMemRef(rewriter, loc, rowC, token);
  if (colC)
    token = genDeallocMemRef(rewriter, loc, colC, token);
  token = genCopyMemRef(rewriter, loc, memV, valC, token);
  token = genDeallocMemRef(rewriter, loc, valC, token);
  tokens.push_back(token);
  genBlockingWait(rewriter, loc, tokens);
  tokens.clear();

  // Done.
  rewriter.replaceOpWithNewOp<sparse_tensor::LoadOp>(op, c);
  return success();
}

//===----------------------------------------------------------------------===//
// Rewriting rules for direct code generation.
//===----------------------------------------------------------------------===//

/// Proof-of-concept rewriter. This rule generates a GPU implementation
/// for each outermost forall loop generated by the sparsifier.
````
- **L1153 EN**: Executes a call or declaration centered on `.getAsyncToken`.
  **L1153 CN**: 执行以 `.getAsyncToken` 为核心的调用或声明。
- **L1154 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L1154 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L1155 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L1155 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L1156 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L1156 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L1157 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L1157 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L1158 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1158 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1159 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L1159 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L1160 EN**: Executes a call or declaration centered on `genCopyMemRef`.
  **L1160 CN**: 执行以 `genCopyMemRef` 为核心的调用或声明。
- **L1161 EN**: Executes a call or declaration centered on `genDeallocMemRef`.
  **L1161 CN**: 执行以 `genDeallocMemRef` 为核心的调用或声明。
- **L1162 EN**: Executes a call or declaration centered on `tokens.push_back`.
  **L1162 CN**: 执行以 `tokens.push_back` 为核心的调用或声明。
- **L1163 EN**: Executes a call or declaration centered on `genBlockingWait`.
  **L1163 CN**: 执行以 `genBlockingWait` 为核心的调用或声明。
- **L1164 EN**: Executes a call or declaration centered on `tokens.clear`.
  **L1164 CN**: 执行以 `tokens.clear` 为核心的调用或声明。
- **L1165 EN**: Blank line separating nearby declarations or logic blocks.
  **L1165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1166 EN**: Comment explains nearby logic, invariants, or intent: `Done.`.
  **L1166 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Done.`。
- **L1167 EN**: Executes a call or declaration centered on `rewriter.replaceOpWithNewOp<sparse_tensor::LoadOp>`.
  **L1167 CN**: 执行以 `rewriter.replaceOpWithNewOp<sparse_tensor::LoadOp>` 为核心的调用或声明。
- **L1168 EN**: Returns from the current function with `success()`.
  **L1168 CN**: 以 `success()` 从当前函数返回。
- **L1169 EN**: Closes the current lexical scope or compound statement.
  **L1169 CN**: 结束当前词法作用域或复合语句块。
- **L1170 EN**: Blank line separating nearby declarations or logic blocks.
  **L1170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1171 EN**: Banner comment marking a file or section boundary.
  **L1171 CN**: 横幅注释，用于标记文件或章节边界。
- **L1172 EN**: Comment explains nearby logic, invariants, or intent: `Rewriting rules for direct code generation.`.
  **L1172 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Rewriting rules for direct code generation.`。
- **L1173 EN**: Banner comment marking a file or section boundary.
  **L1173 CN**: 横幅注释，用于标记文件或章节边界。
- **L1174 EN**: Blank line separating nearby declarations or logic blocks.
  **L1174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1175 EN**: Comment explains nearby logic, invariants, or intent: `Proof-of-concept rewriter. This rule generates a GPU implementation`.
  **L1175 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Proof-of-concept rewriter. This rule generates a GPU implementation`。
- **L1176 EN**: Comment explains nearby logic, invariants, or intent: `for each outermost forall loop generated by the sparsifier.`.
  **L1176 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for each outermost forall loop generated by the sparsifier.`。

### Lines 1177-1200

````cpp
/// TODO: right now works with parallelization-strategy=dense-outer-loop
///       but give this its own flags in the future
struct ForallRewriter : public OpRewritePattern<scf::ParallelOp> {
  using OpRewritePattern<scf::ParallelOp>::OpRewritePattern;

  ForallRewriter(MLIRContext *context, unsigned nT)
      : OpRewritePattern(context), numThreads(nT) {};

  LogicalResult matchAndRewrite(scf::ParallelOp forallOp,
                                PatternRewriter &rewriter) const override {
    // Reject inadmissible loop form.
    // Essentially only accept a loop, generated by the sparsifier,
    // of the form
    //   forall (i = 0; i < N; i++)
    // so that cyclic scheduling over the threads is easy.
    if (!forallOp->hasAttr(LoopEmitter::getLoopEmitterLoopAttrName()) ||
        forallOp.getNumReductions() != 0 || forallOp.getNumLoops() != 1 ||
        !matchPattern(forallOp.getLowerBound()[0], m_Zero()) ||
        !matchPattern(forallOp.getStep()[0], m_One()))
      return failure();
    // Collect every value that is computed outside the parallel loop.
    SetVector<Value> invariants; // stable iteration!
    forallOp->walk([&](Operation *op) {
      // Collect all values of admissible ops.
````
- **L1177 EN**: Comment records a pending task or caution: `TODO: right now works with parallelization-strategy=dense-outer-loop`.
  **L1177 CN**: 注释记录了待办事项或注意点：`TODO: right now works with parallelization-strategy=dense-outer-loop`。
- **L1178 EN**: Comment explains nearby logic, invariants, or intent: `but give this its own flags in the future`.
  **L1178 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`but give this its own flags in the future`。
- **L1179 EN**: Declares struct `ForallRewriter`.
  **L1179 CN**: 声明 struct `ForallRewriter`。
- **L1180 EN**: Executes a standalone statement or declaration: `using OpRewritePattern<scf::ParallelOp>::OpRewritePattern;`.
  **L1180 CN**: 执行一条独立语句或声明：`using OpRewritePattern<scf::ParallelOp>::OpRewritePattern;`。
- **L1181 EN**: Blank line separating nearby declarations or logic blocks.
  **L1181 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1182 EN**: Continues logic associated with callable symbol `ForallRewriter`.
  **L1182 CN**: 继续与可调用符号 `ForallRewriter` 相关的逻辑。
- **L1183 EN**: Executes a call or declaration centered on `OpRewritePattern`.
  **L1183 CN**: 执行以 `OpRewritePattern` 为核心的调用或声明。
- **L1184 EN**: Blank line separating nearby declarations or logic blocks.
  **L1184 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(scf::ParallelOp forallOp,`.
  **L1185 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(scf::ParallelOp forallOp,`。
- **L1186 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L1186 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L1187 EN**: Comment explains nearby logic, invariants, or intent: `Reject inadmissible loop form.`.
  **L1187 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Reject inadmissible loop form.`。
- **L1188 EN**: Comment explains nearby logic, invariants, or intent: `Essentially only accept a loop, generated by the sparsifier,`.
  **L1188 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Essentially only accept a loop, generated by the sparsifier,`。
- **L1189 EN**: Comment explains nearby logic, invariants, or intent: `of the form`.
  **L1189 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`of the form`。
- **L1190 EN**: Comment explains nearby logic, invariants, or intent: `forall (i = 0; i < N; i++)`.
  **L1190 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`forall (i = 0; i < N; i++)`。
- **L1191 EN**: Comment explains nearby logic, invariants, or intent: `so that cyclic scheduling over the threads is easy.`.
  **L1191 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`so that cyclic scheduling over the threads is easy.`。
- **L1192 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1192 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1193 EN**: Continues logic associated with callable symbol `getNumReductions`.
  **L1193 CN**: 继续与可调用符号 `getNumReductions` 相关的逻辑。
- **L1194 EN**: Continues logic associated with callable symbol `matchPattern`.
  **L1194 CN**: 继续与可调用符号 `matchPattern` 相关的逻辑。
- **L1195 EN**: Continues logic associated with callable symbol `matchPattern`.
  **L1195 CN**: 继续与可调用符号 `matchPattern` 相关的逻辑。
- **L1196 EN**: Returns from the current function with `failure()`.
  **L1196 CN**: 以 `failure()` 从当前函数返回。
- **L1197 EN**: Comment explains nearby logic, invariants, or intent: `Collect every value that is computed outside the parallel loop.`.
  **L1197 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Collect every value that is computed outside the parallel loop.`。
- **L1198 EN**: Continues the surrounding expression or declaration: `SetVector<Value> invariants; // stable iteration!`.
  **L1198 CN**: 继续构造周围的表达式或声明：`SetVector<Value> invariants; // stable iteration!`。
- **L1199 EN**: Starts a function, method, lambda, or structured scope: `forallOp->walk([&](Operation *op) {`.
  **L1199 CN**: 开始一个函数、方法、lambda 或结构化作用域：`forallOp->walk([&](Operation *op) {`。
- **L1200 EN**: Comment explains nearby logic, invariants, or intent: `Collect all values of admissible ops.`.
  **L1200 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Collect all values of admissible ops.`。

### Lines 1201-1224

````cpp
      for (OpOperand &o : op->getOpOperands()) {
        Value val = o.get();
        Block *block;
        if (auto arg = dyn_cast<BlockArgument>(val))
          block = arg.getOwner();
        else
          block = val.getDefiningOp()->getBlock();
        if (!forallOp.getRegion().findAncestorBlockInRegion(*block))
          invariants.insert(val);
      }
    });
    // Outline the outside values as proper parameters. Fail when sharing
    // value between host and device is not straightforward.
    SmallVector<Value> constants;
    SmallVector<Value> scalars;
    SmallVector<Value> buffers;
    // A boolean mask aligned 1:1 with the `buffers` array, tracking which
    // of those buffers were mutated by the loop. If true, the corresponding
    // buffer needs to be "copied back" using a device-to-host copy.
    SmallVector<bool> copyBack;
    for (Value val : invariants) {
      Type tp = val.getType();
      if (val.getDefiningOp<arith::ConstantOp>())
        constants.push_back(val);
````
- **L1201 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1201 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1202 EN**: Initializes variable `val` from the right-hand expression.
  **L1202 CN**: 使用右侧表达式初始化变量 `val`。
- **L1203 EN**: Executes a standalone statement or declaration: `Block *block;`.
  **L1203 CN**: 执行一条独立语句或声明：`Block *block;`。
- **L1204 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1204 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1205 EN**: Executes a call or declaration centered on `arg.getOwner`.
  **L1205 CN**: 执行以 `arg.getOwner` 为核心的调用或声明。
- **L1206 EN**: Starts the alternative branch of the preceding conditional.
  **L1206 CN**: 开始前一个条件语句的备选分支。
- **L1207 EN**: Executes a call or declaration centered on `val.getDefiningOp`.
  **L1207 CN**: 执行以 `val.getDefiningOp` 为核心的调用或声明。
- **L1208 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1208 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1209 EN**: Executes a call or declaration centered on `invariants.insert`.
  **L1209 CN**: 执行以 `invariants.insert` 为核心的调用或声明。
- **L1210 EN**: Closes the current lexical scope or compound statement.
  **L1210 CN**: 结束当前词法作用域或复合语句块。
- **L1211 EN**: Executes a standalone statement or declaration: `});`.
  **L1211 CN**: 执行一条独立语句或声明：`});`。
- **L1212 EN**: Comment explains nearby logic, invariants, or intent: `Outline the outside values as proper parameters. Fail when sharing`.
  **L1212 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Outline the outside values as proper parameters. Fail when sharing`。
- **L1213 EN**: Comment explains nearby logic, invariants, or intent: `value between host and device is not straightforward.`.
  **L1213 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`value between host and device is not straightforward.`。
- **L1214 EN**: Executes a standalone statement or declaration: `SmallVector<Value> constants;`.
  **L1214 CN**: 执行一条独立语句或声明：`SmallVector<Value> constants;`。
- **L1215 EN**: Executes a standalone statement or declaration: `SmallVector<Value> scalars;`.
  **L1215 CN**: 执行一条独立语句或声明：`SmallVector<Value> scalars;`。
- **L1216 EN**: Executes a standalone statement or declaration: `SmallVector<Value> buffers;`.
  **L1216 CN**: 执行一条独立语句或声明：`SmallVector<Value> buffers;`。
- **L1217 EN**: Comment explains nearby logic, invariants, or intent: `A boolean mask aligned 1:1 with the `buffers` array, tracking which`.
  **L1217 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A boolean mask aligned 1:1 with the `buffers` array, tracking which`。
- **L1218 EN**: Comment explains nearby logic, invariants, or intent: `of those buffers were mutated by the loop. If true, the corresponding`.
  **L1218 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`of those buffers were mutated by the loop. If true, the corresponding`。
- **L1219 EN**: Comment explains nearby logic, invariants, or intent: `buffer needs to be "copied back" using a device-to-host copy.`.
  **L1219 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`buffer needs to be "copied back" using a device-to-host copy.`。
- **L1220 EN**: Executes a standalone statement or declaration: `SmallVector<bool> copyBack;`.
  **L1220 CN**: 执行一条独立语句或声明：`SmallVector<bool> copyBack;`。
- **L1221 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1221 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1222 EN**: Initializes variable `tp` from the right-hand expression.
  **L1222 CN**: 使用右侧表达式初始化变量 `tp`。
- **L1223 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1223 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1224 EN**: Executes a call or declaration centered on `constants.push_back`.
  **L1224 CN**: 执行以 `constants.push_back` 为核心的调用或声明。

### Lines 1225-1248

````cpp
      else if (isa<FloatType>(tp) || tp.isIntOrIndex())
        scalars.push_back(val);
      else if (isa<MemRefType>(tp)) {
        buffers.push_back(val);

        // Determine if the buffer needs to be "copied back" from device
        // to host by checking for `memref.store` and the write memory effect.
        bool isWrite = false;
        for (Operation *user : val.getUsers()) {
          if (isa<memref::StoreOp>(user)) {
            isWrite = true;
            break;
          }
          if (auto memInterface = dyn_cast<MemoryEffectOpInterface>(user)) {
            if (memInterface.getEffectOnValue<MemoryEffects::Write>(val)) {
              isWrite = true;
              break;
            }
          }
        }
        copyBack.push_back(isWrite);
      } else
        return failure(); // don't know how to share
    }
````
- **L1225 EN**: Starts the alternative branch of the preceding conditional.
  **L1225 CN**: 开始前一个条件语句的备选分支。
- **L1226 EN**: Executes a call or declaration centered on `scalars.push_back`.
  **L1226 CN**: 执行以 `scalars.push_back` 为核心的调用或声明。
- **L1227 EN**: Starts the alternative branch of the preceding conditional.
  **L1227 CN**: 开始前一个条件语句的备选分支。
- **L1228 EN**: Executes a call or declaration centered on `buffers.push_back`.
  **L1228 CN**: 执行以 `buffers.push_back` 为核心的调用或声明。
- **L1229 EN**: Blank line separating nearby declarations or logic blocks.
  **L1229 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1230 EN**: Comment explains nearby logic, invariants, or intent: `Determine if the buffer needs to be "copied back" from device`.
  **L1230 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Determine if the buffer needs to be "copied back" from device`。
- **L1231 EN**: Comment explains nearby logic, invariants, or intent: `to host by checking for `memref.store` and the write memory effect.`.
  **L1231 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to host by checking for `memref.store` and the write memory effect.`。
- **L1232 EN**: Initializes variable `isWrite` from the right-hand expression.
  **L1232 CN**: 使用右侧表达式初始化变量 `isWrite`。
- **L1233 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L1233 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1234 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1234 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1235 EN**: Executes a standalone statement or declaration: `isWrite = true;`.
  **L1235 CN**: 执行一条独立语句或声明：`isWrite = true;`。
- **L1236 EN**: Exits the nearest loop or switch statement.
  **L1236 CN**: 退出最近的循环或 switch 语句。
- **L1237 EN**: Closes the current lexical scope or compound statement.
  **L1237 CN**: 结束当前词法作用域或复合语句块。
- **L1238 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1238 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1239 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1239 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1240 EN**: Executes a standalone statement or declaration: `isWrite = true;`.
  **L1240 CN**: 执行一条独立语句或声明：`isWrite = true;`。
- **L1241 EN**: Exits the nearest loop or switch statement.
  **L1241 CN**: 退出最近的循环或 switch 语句。
- **L1242 EN**: Closes the current lexical scope or compound statement.
  **L1242 CN**: 结束当前词法作用域或复合语句块。
- **L1243 EN**: Closes the current lexical scope or compound statement.
  **L1243 CN**: 结束当前词法作用域或复合语句块。
- **L1244 EN**: Closes the current lexical scope or compound statement.
  **L1244 CN**: 结束当前词法作用域或复合语句块。
- **L1245 EN**: Executes a call or declaration centered on `copyBack.push_back`.
  **L1245 CN**: 执行以 `copyBack.push_back` 为核心的调用或声明。
- **L1246 EN**: Continues the surrounding expression or declaration: `} else`.
  **L1246 CN**: 继续构造周围的表达式或声明：`} else`。
- **L1247 EN**: Returns from the current function with `failure(); // don't know how to share`.
  **L1247 CN**: 以 `failure(); // don't know how to share` 从当前函数返回。
- **L1248 EN**: Closes the current lexical scope or compound statement.
  **L1248 CN**: 结束当前词法作用域或复合语句块。

### Lines 1249-1272

````cpp
    // Pass outlined non-constant values.
    // TODO: Experiment with `useHostRegistrationForOut` to see if we want to
    //       keep the feature at all (either through a heuristic or compiler
    //       option for gpu codegen).
    Location loc = forallOp->getLoc();
    SmallVector<Value> args;
    SmallVector<Value> tokens;
    Value out = genParametersIn(rewriter, loc, scalars, buffers, args, tokens,
                                /*useHostRegistrationForOut=*/false);
    // Set up GPU module and construct GPU function.
    auto saveIp = rewriter.saveInsertionPoint();
    ModuleOp topModule = forallOp->getParentOfType<ModuleOp>();
    auto gpuModule = genGPUModule(rewriter, topModule);
    auto gpuFunc = genGPUFunc(rewriter, gpuModule, args);
    genGPUCode(rewriter, gpuFunc, forallOp, constants, scalars, buffers);
    // Generate code that launches the kernel asynchronously, blocking on all
    // opens tokens and yielding a new token for the output.
    // TODO: Passing in tokens to launch up does not seem to be properly lowered
    //       by cubin yet, hence the current blocking wait.
    rewriter.restoreInsertionPoint(saveIp);
    genBlockingWait(rewriter, loc, tokens);
    tokens.clear();
    Value kernelToken =
        genLaunchGPUFunc(rewriter, gpuFunc, args, tokens, numThreads);
````
- **L1249 EN**: Comment explains nearby logic, invariants, or intent: `Pass outlined non-constant values.`.
  **L1249 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Pass outlined non-constant values.`。
- **L1250 EN**: Comment records a pending task or caution: `TODO: Experiment with `useHostRegistrationForOut` to see if we want to`.
  **L1250 CN**: 注释记录了待办事项或注意点：`TODO: Experiment with `useHostRegistrationForOut` to see if we want to`。
- **L1251 EN**: Comment explains nearby logic, invariants, or intent: `keep the feature at all (either through a heuristic or compiler`.
  **L1251 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`keep the feature at all (either through a heuristic or compiler`。
- **L1252 EN**: Comment explains nearby logic, invariants, or intent: `option for gpu codegen).`.
  **L1252 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`option for gpu codegen).`。
- **L1253 EN**: Initializes variable `loc` from the right-hand expression.
  **L1253 CN**: 使用右侧表达式初始化变量 `loc`。
- **L1254 EN**: Executes a standalone statement or declaration: `SmallVector<Value> args;`.
  **L1254 CN**: 执行一条独立语句或声明：`SmallVector<Value> args;`。
- **L1255 EN**: Executes a standalone statement or declaration: `SmallVector<Value> tokens;`.
  **L1255 CN**: 执行一条独立语句或声明：`SmallVector<Value> tokens;`。
- **L1256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Value out = genParametersIn(rewriter, loc, scalars, buffers, args, tokens,`.
  **L1256 CN**: 继续一个多行参数列表、初始化器或聚合项：`Value out = genParametersIn(rewriter, loc, scalars, buffers, args, tokens,`。
- **L1257 EN**: Comment explains nearby logic, invariants, or intent: `useHostRegistrationForOut=*/false);`.
  **L1257 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`useHostRegistrationForOut=*/false);`。
- **L1258 EN**: Comment explains nearby logic, invariants, or intent: `Set up GPU module and construct GPU function.`.
  **L1258 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Set up GPU module and construct GPU function.`。
- **L1259 EN**: Initializes variable `saveIp` from the right-hand expression.
  **L1259 CN**: 使用右侧表达式初始化变量 `saveIp`。
- **L1260 EN**: Initializes variable `topModule` from the right-hand expression.
  **L1260 CN**: 使用右侧表达式初始化变量 `topModule`。
- **L1261 EN**: Initializes variable `gpuModule` from the right-hand expression.
  **L1261 CN**: 使用右侧表达式初始化变量 `gpuModule`。
- **L1262 EN**: Initializes variable `gpuFunc` from the right-hand expression.
  **L1262 CN**: 使用右侧表达式初始化变量 `gpuFunc`。
- **L1263 EN**: Executes a call or declaration centered on `genGPUCode`.
  **L1263 CN**: 执行以 `genGPUCode` 为核心的调用或声明。
- **L1264 EN**: Comment explains nearby logic, invariants, or intent: `Generate code that launches the kernel asynchronously, blocking on all`.
  **L1264 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generate code that launches the kernel asynchronously, blocking on all`。
- **L1265 EN**: Comment explains nearby logic, invariants, or intent: `opens tokens and yielding a new token for the output.`.
  **L1265 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`opens tokens and yielding a new token for the output.`。
- **L1266 EN**: Comment records a pending task or caution: `TODO: Passing in tokens to launch up does not seem to be properly lowered`.
  **L1266 CN**: 注释记录了待办事项或注意点：`TODO: Passing in tokens to launch up does not seem to be properly lowered`。
- **L1267 EN**: Comment explains nearby logic, invariants, or intent: `by cubin yet, hence the current blocking wait.`.
  **L1267 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`by cubin yet, hence the current blocking wait.`。
- **L1268 EN**: Executes a call or declaration centered on `rewriter.restoreInsertionPoint`.
  **L1268 CN**: 执行以 `rewriter.restoreInsertionPoint` 为核心的调用或声明。
- **L1269 EN**: Executes a call or declaration centered on `genBlockingWait`.
  **L1269 CN**: 执行以 `genBlockingWait` 为核心的调用或声明。
- **L1270 EN**: Executes a call or declaration centered on `tokens.clear`.
  **L1270 CN**: 执行以 `tokens.clear` 为核心的调用或声明。
- **L1271 EN**: Continues the surrounding expression or declaration: `Value kernelToken =`.
  **L1271 CN**: 继续构造周围的表达式或声明：`Value kernelToken =`。
- **L1272 EN**: Executes a call or declaration centered on `genLaunchGPUFunc`.
  **L1272 CN**: 执行以 `genLaunchGPUFunc` 为核心的调用或声明。

### Lines 1273-1296

````cpp
    // Finalize the outlined arguments.
    genParametersOut(rewriter, loc, out, kernelToken, scalars, buffers, args,
                     tokens, copyBack);
    genBlockingWait(rewriter, loc, tokens);
    rewriter.eraseOp(forallOp);
    return success();
  }

private:
  unsigned numThreads;
};

//===----------------------------------------------------------------------===//
// Rewriting rules for library recognition and code generation.
//===----------------------------------------------------------------------===//

/// Proof-of-concept rewriter. This rule recognizes certain math kernels
/// and replaces these with corresponding calls into a sparse library.
struct LinalgOpRewriter : public OpRewritePattern<linalg::GenericOp> {
  using OpRewritePattern<linalg::GenericOp>::OpRewritePattern;

  LinalgOpRewriter(MLIRContext *context, bool rt)
      : OpRewritePattern(context), enableRT(rt) {}

````
- **L1273 EN**: Comment explains nearby logic, invariants, or intent: `Finalize the outlined arguments.`.
  **L1273 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Finalize the outlined arguments.`。
- **L1274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `genParametersOut(rewriter, loc, out, kernelToken, scalars, buffers, args,`.
  **L1274 CN**: 继续一个多行参数列表、初始化器或聚合项：`genParametersOut(rewriter, loc, out, kernelToken, scalars, buffers, args,`。
- **L1275 EN**: Executes a standalone statement or declaration: `tokens, copyBack);`.
  **L1275 CN**: 执行一条独立语句或声明：`tokens, copyBack);`。
- **L1276 EN**: Executes a call or declaration centered on `genBlockingWait`.
  **L1276 CN**: 执行以 `genBlockingWait` 为核心的调用或声明。
- **L1277 EN**: Executes a call or declaration centered on `rewriter.eraseOp`.
  **L1277 CN**: 执行以 `rewriter.eraseOp` 为核心的调用或声明。
- **L1278 EN**: Returns from the current function with `success()`.
  **L1278 CN**: 以 `success()` 从当前函数返回。
- **L1279 EN**: Closes the current lexical scope or compound statement.
  **L1279 CN**: 结束当前词法作用域或复合语句块。
- **L1280 EN**: Blank line separating nearby declarations or logic blocks.
  **L1280 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1281 EN**: Sets the following members to `private` access.
  **L1281 CN**: 将后续成员的访问级别设为 `private`。
- **L1282 EN**: Executes a standalone statement or declaration: `unsigned numThreads;`.
  **L1282 CN**: 执行一条独立语句或声明：`unsigned numThreads;`。
- **L1283 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1283 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1284 EN**: Blank line separating nearby declarations or logic blocks.
  **L1284 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1285 EN**: Banner comment marking a file or section boundary.
  **L1285 CN**: 横幅注释，用于标记文件或章节边界。
- **L1286 EN**: Comment explains nearby logic, invariants, or intent: `Rewriting rules for library recognition and code generation.`.
  **L1286 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Rewriting rules for library recognition and code generation.`。
- **L1287 EN**: Banner comment marking a file or section boundary.
  **L1287 CN**: 横幅注释，用于标记文件或章节边界。
- **L1288 EN**: Blank line separating nearby declarations or logic blocks.
  **L1288 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1289 EN**: Comment explains nearby logic, invariants, or intent: `Proof-of-concept rewriter. This rule recognizes certain math kernels`.
  **L1289 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Proof-of-concept rewriter. This rule recognizes certain math kernels`。
- **L1290 EN**: Comment explains nearby logic, invariants, or intent: `and replaces these with corresponding calls into a sparse library.`.
  **L1290 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and replaces these with corresponding calls into a sparse library.`。
- **L1291 EN**: Declares struct `LinalgOpRewriter`.
  **L1291 CN**: 声明 struct `LinalgOpRewriter`。
- **L1292 EN**: Executes a standalone statement or declaration: `using OpRewritePattern<linalg::GenericOp>::OpRewritePattern;`.
  **L1292 CN**: 执行一条独立语句或声明：`using OpRewritePattern<linalg::GenericOp>::OpRewritePattern;`。
- **L1293 EN**: Blank line separating nearby declarations or logic blocks.
  **L1293 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1294 EN**: Continues logic associated with callable symbol `LinalgOpRewriter`.
  **L1294 CN**: 继续与可调用符号 `LinalgOpRewriter` 相关的逻辑。
- **L1295 EN**: Continues logic associated with callable symbol `OpRewritePattern`.
  **L1295 CN**: 继续与可调用符号 `OpRewritePattern` 相关的逻辑。
- **L1296 EN**: Blank line separating nearby declarations or logic blocks.
  **L1296 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1297-1320

````cpp
  LogicalResult matchAndRewrite(linalg::GenericOp op,
                                PatternRewriter &rewriter) const override {
    if (op.getNumDpsInits() != 1)
      return failure(); // reject multi-output

    const unsigned numLoops = op.getNumLoops();
    const unsigned numTensors = op->getNumOperands();
    const auto iteratorTypes = op.getIteratorTypesArray();
    SmallVector<AffineMap, 4> maps = op.getIndexingMapsArray();

    using MapList = ArrayRef<ArrayRef<AffineExpr>>;
    auto infer = [&](MapList m) {
      return AffineMap::inferFromExprList(m, op.getContext());
    };
    AffineExpr i, j, k;
    bindDims(getContext(), i, j, k);

    // TODO: more robust patterns, transposed versions, more kernels,
    //       identify alpha and beta and pass them to the CUDA calls.

    // Recognize a SpMV kernel.
    if (numLoops == 2 && numTensors == 3 &&
        linalg::isParallelIterator(iteratorTypes[0]) &&
        linalg::isReductionIterator(iteratorTypes[1]) &&
````
- **L1297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LogicalResult matchAndRewrite(linalg::GenericOp op,`.
  **L1297 CN**: 继续一个多行参数列表、初始化器或聚合项：`LogicalResult matchAndRewrite(linalg::GenericOp op,`。
- **L1298 EN**: Continues the surrounding expression or declaration: `PatternRewriter &rewriter) const override {`.
  **L1298 CN**: 继续构造周围的表达式或声明：`PatternRewriter &rewriter) const override {`。
- **L1299 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1299 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1300 EN**: Returns from the current function with `failure(); // reject multi-output`.
  **L1300 CN**: 以 `failure(); // reject multi-output` 从当前函数返回。
- **L1301 EN**: Blank line separating nearby declarations or logic blocks.
  **L1301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1302 EN**: Initializes variable `numLoops` from the right-hand expression.
  **L1302 CN**: 使用右侧表达式初始化变量 `numLoops`。
- **L1303 EN**: Initializes variable `numTensors` from the right-hand expression.
  **L1303 CN**: 使用右侧表达式初始化变量 `numTensors`。
- **L1304 EN**: Initializes variable `iteratorTypes` from the right-hand expression.
  **L1304 CN**: 使用右侧表达式初始化变量 `iteratorTypes`。
- **L1305 EN**: Initializes variable `maps` from the right-hand expression.
  **L1305 CN**: 使用右侧表达式初始化变量 `maps`。
- **L1306 EN**: Blank line separating nearby declarations or logic blocks.
  **L1306 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1307 EN**: Defines alias `MapList` to simplify later code.
  **L1307 CN**: 定义别名 `MapList` 以简化后续代码。
- **L1308 EN**: Starts a function, method, lambda, or structured scope: `auto infer = [&](MapList m) {`.
  **L1308 CN**: 开始一个函数、方法、lambda 或结构化作用域：`auto infer = [&](MapList m) {`。
- **L1309 EN**: Returns from the current function with `AffineMap::inferFromExprList(m, op.getContext())`.
  **L1309 CN**: 以 `AffineMap::inferFromExprList(m, op.getContext())` 从当前函数返回。
- **L1310 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1310 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1311 EN**: Executes a standalone statement or declaration: `AffineExpr i, j, k;`.
  **L1311 CN**: 执行一条独立语句或声明：`AffineExpr i, j, k;`。
- **L1312 EN**: Executes a call or declaration centered on `bindDims`.
  **L1312 CN**: 执行以 `bindDims` 为核心的调用或声明。
- **L1313 EN**: Blank line separating nearby declarations or logic blocks.
  **L1313 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1314 EN**: Comment records a pending task or caution: `TODO: more robust patterns, transposed versions, more kernels,`.
  **L1314 CN**: 注释记录了待办事项或注意点：`TODO: more robust patterns, transposed versions, more kernels,`。
- **L1315 EN**: Comment explains nearby logic, invariants, or intent: `identify alpha and beta and pass them to the CUDA calls.`.
  **L1315 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`identify alpha and beta and pass them to the CUDA calls.`。
- **L1316 EN**: Blank line separating nearby declarations or logic blocks.
  **L1316 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1317 EN**: Comment explains nearby logic, invariants, or intent: `Recognize a SpMV kernel.`.
  **L1317 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Recognize a SpMV kernel.`。
- **L1318 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1318 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1319 EN**: Continues logic associated with callable symbol `isParallelIterator`.
  **L1319 CN**: 继续与可调用符号 `isParallelIterator` 相关的逻辑。
- **L1320 EN**: Continues logic associated with callable symbol `isReductionIterator`.
  **L1320 CN**: 继续与可调用符号 `isReductionIterator` 相关的逻辑。

### Lines 1321-1344

````cpp
        maps == infer({{i, j}, {j}, {i}}) && matchSumOfMultOfArgs(op)) {
      return rewriteSpMV(rewriter, op, enableRT);
    }

    // Recognize a SpGEMM, 2:4-SpMM, or SpMM kernel.
    if (numLoops == 3 && numTensors == 3 &&
        linalg::isParallelIterator(iteratorTypes[0]) &&
        linalg::isParallelIterator(iteratorTypes[1]) &&
        linalg::isReductionIterator(iteratorTypes[2]) &&
        maps == infer({{i, k}, {k, j}, {i, j}}) && matchSumOfMultOfArgs(op)) {
      if (!isDenseTensor(op.getOperand(0)) && !isDenseTensor(op.getOperand(1)))
        return rewriteSpGEMM(rewriter, op, enableRT);
      if (isConversionInto24(op.getOperand(0)))
        return rewrite2To4SpMM(rewriter, op);
      return rewriteSpMM(rewriter, op, enableRT);
    }

    // Recognize a SDDMM kernel.
    if (numLoops == 3 && numTensors == 3 &&
        linalg::isParallelIterator(iteratorTypes[0]) &&
        linalg::isParallelIterator(iteratorTypes[1]) &&
        linalg::isReductionIterator(iteratorTypes[2]) &&
        maps == infer({{i, k}, {k, j}, {i, j}}) &&
        matchSumReductionOfMulUnary(op)) {
````
- **L1321 EN**: Starts a function, method, lambda, or structured scope: `maps == infer({{i, j}, {j}, {i}}) && matchSumOfMultOfArgs(op)) {`.
  **L1321 CN**: 开始一个函数、方法、lambda 或结构化作用域：`maps == infer({{i, j}, {j}, {i}}) && matchSumOfMultOfArgs(op)) {`。
- **L1322 EN**: Returns from the current function with `rewriteSpMV(rewriter, op, enableRT)`.
  **L1322 CN**: 以 `rewriteSpMV(rewriter, op, enableRT)` 从当前函数返回。
- **L1323 EN**: Closes the current lexical scope or compound statement.
  **L1323 CN**: 结束当前词法作用域或复合语句块。
- **L1324 EN**: Blank line separating nearby declarations or logic blocks.
  **L1324 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1325 EN**: Comment explains nearby logic, invariants, or intent: `Recognize a SpGEMM, 2:4-SpMM, or SpMM kernel.`.
  **L1325 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Recognize a SpGEMM, 2:4-SpMM, or SpMM kernel.`。
- **L1326 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1326 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1327 EN**: Continues logic associated with callable symbol `isParallelIterator`.
  **L1327 CN**: 继续与可调用符号 `isParallelIterator` 相关的逻辑。
- **L1328 EN**: Continues logic associated with callable symbol `isParallelIterator`.
  **L1328 CN**: 继续与可调用符号 `isParallelIterator` 相关的逻辑。
- **L1329 EN**: Continues logic associated with callable symbol `isReductionIterator`.
  **L1329 CN**: 继续与可调用符号 `isReductionIterator` 相关的逻辑。
- **L1330 EN**: Starts a function, method, lambda, or structured scope: `maps == infer({{i, k}, {k, j}, {i, j}}) && matchSumOfMultOfArgs(op)) {`.
  **L1330 CN**: 开始一个函数、方法、lambda 或结构化作用域：`maps == infer({{i, k}, {k, j}, {i, j}}) && matchSumOfMultOfArgs(op)) {`。
- **L1331 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1331 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1332 EN**: Returns from the current function with `rewriteSpGEMM(rewriter, op, enableRT)`.
  **L1332 CN**: 以 `rewriteSpGEMM(rewriter, op, enableRT)` 从当前函数返回。
- **L1333 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1333 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1334 EN**: Returns from the current function with `rewrite2To4SpMM(rewriter, op)`.
  **L1334 CN**: 以 `rewrite2To4SpMM(rewriter, op)` 从当前函数返回。
- **L1335 EN**: Returns from the current function with `rewriteSpMM(rewriter, op, enableRT)`.
  **L1335 CN**: 以 `rewriteSpMM(rewriter, op, enableRT)` 从当前函数返回。
- **L1336 EN**: Closes the current lexical scope or compound statement.
  **L1336 CN**: 结束当前词法作用域或复合语句块。
- **L1337 EN**: Blank line separating nearby declarations or logic blocks.
  **L1337 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1338 EN**: Comment explains nearby logic, invariants, or intent: `Recognize a SDDMM kernel.`.
  **L1338 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Recognize a SDDMM kernel.`。
- **L1339 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1339 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1340 EN**: Continues logic associated with callable symbol `isParallelIterator`.
  **L1340 CN**: 继续与可调用符号 `isParallelIterator` 相关的逻辑。
- **L1341 EN**: Continues logic associated with callable symbol `isParallelIterator`.
  **L1341 CN**: 继续与可调用符号 `isParallelIterator` 相关的逻辑。
- **L1342 EN**: Continues logic associated with callable symbol `isReductionIterator`.
  **L1342 CN**: 继续与可调用符号 `isReductionIterator` 相关的逻辑。
- **L1343 EN**: Continues logic associated with callable symbol `infer`.
  **L1343 CN**: 继续与可调用符号 `infer` 相关的逻辑。
- **L1344 EN**: Starts a function, method, lambda, or structured scope: `matchSumReductionOfMulUnary(op)) {`.
  **L1344 CN**: 开始一个函数、方法、lambda 或结构化作用域：`matchSumReductionOfMulUnary(op)) {`。

### Lines 1345-1368

````cpp
      return rewriteSDDMM(rewriter, op, enableRT);
    }

    return failure();
  }

private:
  bool enableRT;
};

} // namespace

//===----------------------------------------------------------------------===//
// Public method for populating GPU rewriting rules.
//
// Currently two set of rewriting rules are made available. The first set
// implements direct code generation, currently by means of convering the
// outermost paralell loop into GPU threads. The second set implements
// libary recognition of a set of sparse operations. Eventually, the right
// combination of these two approaches has to be found.
//===----------------------------------------------------------------------===//

void mlir::populateSparseGPUCodegenPatterns(RewritePatternSet &patterns,
                                            unsigned numThreads) {
````
- **L1345 EN**: Returns from the current function with `rewriteSDDMM(rewriter, op, enableRT)`.
  **L1345 CN**: 以 `rewriteSDDMM(rewriter, op, enableRT)` 从当前函数返回。
- **L1346 EN**: Closes the current lexical scope or compound statement.
  **L1346 CN**: 结束当前词法作用域或复合语句块。
- **L1347 EN**: Blank line separating nearby declarations or logic blocks.
  **L1347 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1348 EN**: Returns from the current function with `failure()`.
  **L1348 CN**: 以 `failure()` 从当前函数返回。
- **L1349 EN**: Closes the current lexical scope or compound statement.
  **L1349 CN**: 结束当前词法作用域或复合语句块。
- **L1350 EN**: Blank line separating nearby declarations or logic blocks.
  **L1350 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1351 EN**: Sets the following members to `private` access.
  **L1351 CN**: 将后续成员的访问级别设为 `private`。
- **L1352 EN**: Executes a standalone statement or declaration: `bool enableRT;`.
  **L1352 CN**: 执行一条独立语句或声明：`bool enableRT;`。
- **L1353 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L1353 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L1354 EN**: Blank line separating nearby declarations or logic blocks.
  **L1354 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1355 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace`.
  **L1355 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace`。
- **L1356 EN**: Blank line separating nearby declarations or logic blocks.
  **L1356 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1357 EN**: Banner comment marking a file or section boundary.
  **L1357 CN**: 横幅注释，用于标记文件或章节边界。
- **L1358 EN**: Comment explains nearby logic, invariants, or intent: `Public method for populating GPU rewriting rules.`.
  **L1358 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Public method for populating GPU rewriting rules.`。
- **L1359 EN**: Separator comment used for visual grouping.
  **L1359 CN**: 用于视觉分组的分隔注释。
- **L1360 EN**: Comment explains nearby logic, invariants, or intent: `Currently two set of rewriting rules are made available. The first set`.
  **L1360 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Currently two set of rewriting rules are made available. The first set`。
- **L1361 EN**: Comment explains nearby logic, invariants, or intent: `implements direct code generation, currently by means of convering the`.
  **L1361 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`implements direct code generation, currently by means of convering the`。
- **L1362 EN**: Comment explains nearby logic, invariants, or intent: `outermost paralell loop into GPU threads. The second set implements`.
  **L1362 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`outermost paralell loop into GPU threads. The second set implements`。
- **L1363 EN**: Comment explains nearby logic, invariants, or intent: `libary recognition of a set of sparse operations. Eventually, the right`.
  **L1363 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`libary recognition of a set of sparse operations. Eventually, the right`。
- **L1364 EN**: Comment explains nearby logic, invariants, or intent: `combination of these two approaches has to be found.`.
  **L1364 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`combination of these two approaches has to be found.`。
- **L1365 EN**: Banner comment marking a file or section boundary.
  **L1365 CN**: 横幅注释，用于标记文件或章节边界。
- **L1366 EN**: Blank line separating nearby declarations or logic blocks.
  **L1366 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void mlir::populateSparseGPUCodegenPatterns(RewritePatternSet &patterns,`.
  **L1367 CN**: 继续一个多行参数列表、初始化器或聚合项：`void mlir::populateSparseGPUCodegenPatterns(RewritePatternSet &patterns,`。
- **L1368 EN**: Continues the surrounding expression or declaration: `unsigned numThreads) {`.
  **L1368 CN**: 继续构造周围的表达式或声明：`unsigned numThreads) {`。

### Lines 1369-1375

````cpp
  patterns.add<ForallRewriter>(patterns.getContext(), numThreads);
}

void mlir::populateSparseGPULibgenPatterns(RewritePatternSet &patterns,
                                           bool enableRT) {
  patterns.add<LinalgOpRewriter>(patterns.getContext(), enableRT);
}
````
- **L1369 EN**: Executes a call or declaration centered on `patterns.add<ForallRewriter>`.
  **L1369 CN**: 执行以 `patterns.add<ForallRewriter>` 为核心的调用或声明。
- **L1370 EN**: Closes the current lexical scope or compound statement.
  **L1370 CN**: 结束当前词法作用域或复合语句块。
- **L1371 EN**: Blank line separating nearby declarations or logic blocks.
  **L1371 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `void mlir::populateSparseGPULibgenPatterns(RewritePatternSet &patterns,`.
  **L1372 CN**: 继续一个多行参数列表、初始化器或聚合项：`void mlir::populateSparseGPULibgenPatterns(RewritePatternSet &patterns,`。
- **L1373 EN**: Continues the surrounding expression or declaration: `bool enableRT) {`.
  **L1373 CN**: 继续构造周围的表达式或声明：`bool enableRT) {`。
- **L1374 EN**: Executes a call or declaration centered on `patterns.add<LinalgOpRewriter>`.
  **L1374 CN**: 执行以 `patterns.add<LinalgOpRewriter>` 为核心的调用或声明。
- **L1375 EN**: Closes the current lexical scope or compound statement.
  **L1375 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Sparse tensor compilation / 稀疏张量编译**
- **Rewrite-driven lowering / 基于重写的 lowering**
- **Dialect definition and registration / 方言定义与注册**
- **Operation semantics and verification / 操作语义与验证**
- **Pattern-based rewriting / 基于模式的重写**
- **Pattern collection management / 重写模式集合管理**
- **Failure-aware result propagation / 带失败语义的结果传播**
- **IR mutation through rewriter helpers / 通过重写器辅助工具进行 IR 变换**
- **Type-system modeling / 类型系统建模**
- **Region-based control structure / 基于 Region 的控制结构**

## Dependencies / 依赖关系

- `Utils/CodegenUtils.h`: Provides local declarations paired with this implementation unit. / 提供与该实现单元配套的本地声明。
- `Utils/LoopEmitter.h`: Provides local declarations paired with this implementation unit. / 提供与该实现单元配套的本地声明。
- `mlir/Dialect/Bufferization/IR/Bufferization.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/GPU/IR/GPUDialect.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Linalg/IR/Linalg.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/Linalg/Utils/Utils.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/MemRef/IR/MemRef.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/SCF/IR/SCF.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/SparseTensor/IR/SparseTensor.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/SparseTensor/IR/SparseTensorType.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/Dialect/SparseTensor/Transforms/Passes.h`: Provides dialect-specific IR, transforms, or shared utilities. / 提供方言专用 IR、变换或共享工具。
- `mlir/IR/IRMapping.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `mlir/IR/Matchers.h`: Provides core MLIR IR classes such as operations, attributes, and types. / 提供MLIR 核心 IR 类，如操作、属性与类型。
- `mlir/Interfaces/SideEffectInterfaces.h`: Provides generic MLIR interfaces implemented by operations or dialects. / 提供由操作或方言实现的通用 MLIR 接口。
- `llvm/Support/Casting.h`: Provides LLVM support-library facilities such as diagnostics and casting. / 提供LLVM 支持库设施，如诊断与类型转换。
