# TestAllocation.cpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/lib/Analysis/TestAllocation.cpp`
- **Purpose / 目的:** **EN:** This c++ support file exercises allocation-analysis test-pass implementation in Triton's analysis coverage. **CN:** 该C++ 支撑文件用于覆盖 Triton 分析中的分配分析测试 pass 实现相关场景。
- **Validation / 验证内容:** **EN:** The file documents and validates allocation-analysis test-pass implementation in the surrounding test infrastructure. **CN:** 该文件在周边测试基础设施中记录并验证 分配分析测试 pass 实现。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
#include "mlir/Pass/Pass.h"
#include "triton/Analysis/Allocation.h"
```
**EN:** This block declares compilation dependencies via `mlir/Pass/Pass.h`, `triton/Analysis/Allocation.h`.
**CN:** 这一块通过 `mlir/Pass/Pass.h`, `triton/Analysis/Allocation.h` 声明编译依赖。

### Line 4
```cpp
using namespace mlir;
```
**EN:** These `using` directives shorten later references to MLIR and Triton symbols so the pass code stays readable.
**CN:** 这些 `using` 指令缩短了后续对 MLIR 和 Triton 符号的引用，让 pass 代码更易读。

### Line 6
```cpp
namespace {
```
**EN:** This C++ block provides support code around `namespace {` for the test infrastructure.
**CN:** 这一 C++ 代码块围绕 `namespace {` 提供测试基础设施所需的支撑逻辑。

### Line 8
```cpp
unsigned getScratchSize128(Operation *) { return 128; }
```
**EN:** This C++ block provides support code around `unsigned getScratchSize128(Operation *) { return 128; }` for the test infrastructure.
**CN:** 这一 C++ 代码块围绕 `unsigned getScratchSize128(Operation *) { return 128; }` 提供测试基础设施所需的支撑逻辑。

### Lines 10-13
```cpp
enum class GetScratchSizeFunction {
  None,
  ValidConstant,
};
```
**EN:** This block declares the test pass type `GetScratchSizeFunction`. It ties the pass to MLIR's `PassWrapper` infrastructure and scopes the helper methods that implement the test behavior.
**CN:** 这一块声明了测试 pass 类型 `GetScratchSizeFunction`。它把该 pass 接到 MLIR 的 `PassWrapper` 基础设施上，并收纳实现测试行为的辅助方法。

### Lines 15-16
```cpp
struct TestAllocationPass
    : public PassWrapper<TestAllocationPass, OperationPass<ModuleOp>> {
```
**EN:** This block declares the test pass type `TestAllocationPass`. It ties the pass to MLIR's `PassWrapper` infrastructure and scopes the helper methods that implement the test behavior.
**CN:** 这一块声明了测试 pass 类型 `TestAllocationPass`。它把该 pass 接到 MLIR 的 `PassWrapper` 基础设施上，并收纳实现测试行为的辅助方法。

### Line 18
```cpp
  MLIR_DEFINE_EXPLICIT_INTERNAL_INLINE_TYPE_ID(TestAllocationPass);
```
**EN:** This macro gives the pass a stable internal type identifier, which MLIR uses for pass registration and RTTI-like lookup.
**CN:** 这个宏为 pass 提供稳定的内部类型标识，供 MLIR 在 pass 注册和类似 RTTI 的查询中使用。

### Lines 20-22
```cpp
  TestAllocationPass() = default;
  TestAllocationPass(const TestAllocationPass &other)
      : PassWrapper<TestAllocationPass, OperationPass<ModuleOp>>(other) {}
```
**EN:** This block declares the test pass type . It ties the pass to MLIR's `PassWrapper` infrastructure and scopes the helper methods that implement the test behavior.
**CN:** 这一块声明了测试 pass 类型 。它把该 pass 接到 MLIR 的 `PassWrapper` 基础设施上，并收纳实现测试行为的辅助方法。

### Lines 24-27
```cpp
  StringRef getArgument() const final { return "test-print-allocation"; }
  StringRef getDescription() const final {
    return "print the result of the allocation pass";
  }
```
**EN:** This block exposes the command-line pass name and the human-readable description used by MLIR pass registration.
**CN:** 这一块暴露了命令行 pass 名称，以及 MLIR pass 注册时使用的人类可读描述。

### Lines 29-38
```cpp
  ModuleAllocation getModuleAllocation() {
    switch (getScratchSizeFunction) {
    case GetScratchSizeFunction::None:
      return {getOperation(), triton::defaultAllocationAnalysisScratchSizeFn,
              partitionSize};
    case GetScratchSizeFunction::ValidConstant:
      return {getOperation(), getScratchSize128, partitionSize};
    }
    llvm_unreachable("Unhandled case");
  }
```
**EN:** This C++ block provides support code around `ModuleAllocation getModuleAllocation() {` for the test infrastructure.
**CN:** 这一 C++ 代码块围绕 `ModuleAllocation getModuleAllocation() {` 提供测试基础设施所需的支撑逻辑。

### Lines 40-73
```cpp
  void runOnOperation() override {
    ModuleOp moduleOp = getOperation();
    // Convert to std::string can remove quotes from opName
    ModuleAllocation moduleAllocation = getModuleAllocation();
    moduleOp.walk([&](triton::FuncOp funcOp) {
      auto opName = SymbolTable::getSymbolName(funcOp).getValue().str();
      mlir::emitRemark(funcOp.getLoc(), opName);
      auto *allocation = moduleAllocation.getFuncData(funcOp);
      funcOp.walk([&](Operation *op) {
        auto scratchBufferId = allocation->getBufferId(op);
        if (scratchBufferId != Allocation::InvalidBufferId) {
          size_t offset = allocation->getOffset(scratchBufferId);
          size_t size = allocation->getAllocatedSize(scratchBufferId);
          mlir::emitRemark(op->getLoc())
              << (allocation->isVirtualBuffer(scratchBufferId) ? "virtual"
                                                               : "scratch")
              << " offset = " << offset << ", size = " << size;
        }
        if (op->getNumResults() < 1)
          return;
        for (Value result : op->getResults()) {
          auto bufferIds = allocation->getBufferIds(result);
          for (auto bufferId : bufferIds) {
            size_t offset = allocation->getOffset(bufferId);
            size_t size = allocation->getAllocatedSize(bufferId);
            mlir::emitRemark(op->getLoc())
                << "offset = " << offset << ", size = " << size;
          }
        }
      });
      mlir::emitRemark(funcOp.getLoc())
          << "size = " << allocation->getSharedMemorySize();
    });
  }
```
**EN:** `runOnOperation` is the core execution hook. It builds/queries analysis state and walks the IR to emit test output, using helpers such as `runOnOperation`, `getOperation`, `getModuleAllocation`, `walk`, `SymbolTable::getSymbolName`, `getValue`, `str`, `mlir::emitRemark`, ... (+12).
**CN:** `runOnOperation` 是核心执行入口。它构建/查询分析状态并遍历 IR，借助 `runOnOperation`, `getOperation`, `getModuleAllocation`, `walk`, `SymbolTable::getSymbolName`, `getValue`, `str`, `mlir::emitRemark`, ... (+12) 等辅助调用输出测试结果。

### Lines 75-82
```cpp
  Option<GetScratchSizeFunction> getScratchSizeFunction{
      *this, "get-scratch-size-function",
      llvm::cl::desc("Custom scratch size function to use"),
      llvm::cl::init(GetScratchSizeFunction::None),
      llvm::cl::values(
          clEnumValN(GetScratchSizeFunction::None, "None", "None (default)"),
          clEnumValN(GetScratchSizeFunction::ValidConstant, "ValidConstant",
                     "ValidConstant"))};
```
**EN:** This C++ block provides support code around `Option<GetScratchSizeFunction> getScratchSizeFunction{` for the test infrastructure.
**CN:** 这一 C++ 代码块围绕 `Option<GetScratchSizeFunction> getScratchSizeFunction{` 提供测试基础设施所需的支撑逻辑。

### Lines 84-89
```cpp
  Option<size_t> partitionSize{
      *this, "partition-size",
      llvm::cl::desc(
          "Shared memory partition size in bytes (0 = no partitioning)"),
      llvm::cl::init(0)};
};
```
**EN:** This C++ block provides support code around `Option<size_t> partitionSize{` for the test infrastructure.
**CN:** 这一 C++ 代码块围绕 `Option<size_t> partitionSize{` 提供测试基础设施所需的支撑逻辑。

### Line 91
```cpp
} // namespace
```
**EN:** This C++ block provides support code around `} // namespace` for the test infrastructure.
**CN:** 这一 C++ 代码块围绕 `} // namespace` 提供测试基础设施所需的支撑逻辑。

### Lines 93-97
```cpp
namespace mlir {
namespace test {
void registerTestAllocationPass() { PassRegistration<TestAllocationPass>(); }
} // namespace test
} // namespace mlir
```
**EN:** This block registers the test pass with MLIR so lit tests can invoke it by name through `triton-opt`.
**CN:** 这一块把测试 pass 注册到 MLIR 中，这样 lit 测试就能通过 `triton-opt` 按名称调用它。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's analysis coverage and focuses on allocation-analysis test-pass implementation.  
  **CN:** 该文件属于 Triton 的 分析 测试覆盖，关注点是 分配分析测试 pass 实现。
- **EN:** The code is built around MLIR pass registration, IR walking, and diagnostic emission.  
  **CN:** 这份代码围绕 MLIR pass 注册、IR 遍历和诊断输出展开。

## Dependencies / 依赖关系

- **EN:** Header dependencies: `mlir/Pass/Pass.h`, `triton/Analysis/Allocation.h`.  
  **CN:** 头文件依赖：`mlir/Pass/Pass.h`、`triton/Analysis/Allocation.h`。
- **EN:** MLIR pass infrastructure (`PassWrapper`, pass registration, diagnostics) is required for this file.  
  **CN:** 该文件依赖 MLIR pass 基础设施（`PassWrapper`、pass 注册、诊断系统）。
