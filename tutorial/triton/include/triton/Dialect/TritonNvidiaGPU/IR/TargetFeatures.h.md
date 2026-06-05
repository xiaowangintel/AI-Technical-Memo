# TargetFeatures.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonNvidiaGPU/IR/TargetFeatures.h`
- **EN:** Declares APIs centered on `TargetFeatures` inside Triton.
- **CN:** 声明 Triton 中围绕 `TargetFeatures` 的 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_DIALECT_TRITONNVIDIAGPU_IR_TARGETFEATURES_H_
   2: #define TRITON_DIALECT_TRITONNVIDIAGPU_IR_TARGETFEATURES_H_
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-7
```cpp
   4: #include "mlir/IR/BuiltinAttributes.h"
   5: #include "mlir/IR/BuiltinOps.h"
   6: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
   7: #include <cassert>
```
**EN:** This block imports the direct dependencies needed here, including mlir/IR/BuiltinAttributes.h, mlir/IR/BuiltinOps.h, triton/Dialect/TritonGPU/IR/Dialect.h, and <cassert>.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/IR/BuiltinAttributes.h, mlir/IR/BuiltinOps.h, triton/Dialect/TritonGPU/IR/Dialect.h, and <cassert>。

### Lines 9-9
```cpp
   9: namespace mlir::triton::nvidia_gpu {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir::triton::nvidia_gpu.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir::triton::nvidia_gpu 下。

### Lines 11-14
```cpp
  11: class TargetFeatures {
  12: public:
  13:   explicit TargetFeatures(int computeCapability)
  14:       : computeCapability(computeCapability) {}
```
**EN:** This block introduces `TargetFeatures`, the main class/struct defined here. Within the declaration, methods such as computeCapability expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `TargetFeatures`。 其中 computeCapability 等方法构成了它的主要接口。

### Lines 16-19
```cpp
  16:   static TargetFeatures fromModuleOp(ModuleOp moduleOp) {
  17:     auto targetAttr =
  18:         moduleOp->getAttrOfType<StringAttr>(triton::gpu::AttrTargetName);
  19:     assert(targetAttr && "Expected a target attribute on the module operation");
```
**EN:** This block declares or defines callable APIs such as fromModuleOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 fromModuleOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 21-23
```cpp
  21:     StringRef targetName = targetAttr.getValue();
  22:     assert(targetName.starts_with(kTargetPrefix) &&
  23:            "expected target attribute to be prefixed with \"cuda:\"");
```
**EN:** This block declares or defines callable APIs such as getValue and starts_with, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getValue and starts_with 等可调用 API，用来封装这里提供的核心行为。

### Lines 25-29
```cpp
  25:     int computeCapability;
  26:     bool parseError = targetName.drop_front(sizeof(kTargetPrefix) - 1)
  27:                           .getAsInteger(10, computeCapability);
  28:     assert(!parseError &&
  29:            "invalid compute capability string in target attribute");
```
**EN:** This block declares or defines callable APIs such as drop_front and getAsInteger, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 drop_front and getAsInteger 等可调用 API，用来封装这里提供的核心行为。

### Lines 31-32
```cpp
  31:     return TargetFeatures(computeCapability);
  32:   }
```
**EN:** This block declares or defines callable APIs such as TargetFeatures, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 TargetFeatures 等可调用 API，用来封装这里提供的核心行为。

### Lines 34-34
```cpp
  34:   int getComputeCapability() const { return computeCapability; }
```
**EN:** This block declares or defines callable APIs such as getComputeCapability, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getComputeCapability 等可调用 API，用来封装这里提供的核心行为。

### Lines 36-38
```cpp
  36:   bool supportClusterOps() const {
  37:     return computeCapability >= 90 && computeCapability / 10 != 12;
  38:   }
```
**EN:** This block declares or defines callable APIs such as supportClusterOps, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 supportClusterOps 等可调用 API，用来封装这里提供的核心行为。

### Lines 40-40
```cpp
  40:   bool supportMaximumMinimum() const { return computeCapability >= 80; }
```
**EN:** This block declares or defines callable APIs such as supportMaximumMinimum, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 supportMaximumMinimum 等可调用 API，用来封装这里提供的核心行为。

### Lines 42-44
```cpp
  42:   bool supportLdMatrix() const { return computeCapability >= 75; }
  43:   bool supportStMatrix() const { return computeCapability >= 90; }
  44:   bool supportLdStMatrixB8() const { return computeCapability >= 100; }
```
**EN:** This block declares or defines callable APIs such as supportLdMatrix, supportStMatrix, and supportLdStMatrixB8, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 supportLdMatrix, supportStMatrix, and supportLdStMatrixB8 等可调用 API，用来封装这里提供的核心行为。

### Lines 46-49
```cpp
  46:   bool supportBitwidth16Elementwise() const {
  47:     // Hopper (sm90) and newer.
  48:     return computeCapability >= 90;
  49:   }
```
**EN:** This block declares or defines callable APIs such as supportBitwidth16Elementwise, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 supportBitwidth16Elementwise 等可调用 API，用来封装这里提供的核心行为。

### Lines 51-54
```cpp
  51:   bool supportBitwidth32Elementwise() const {
  52:     // Blackwell (sm100) and newer.
  53:     return computeCapability >= 100;
  54:   }
```
**EN:** This block declares or defines callable APIs such as supportBitwidth32Elementwise, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 supportBitwidth32Elementwise 等可调用 API，用来封装这里提供的核心行为。

### Lines 56-57
```cpp
  56: private:
  57:   static constexpr char kTargetPrefix[] = "cuda:";
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 59-60
```cpp
  59:   int computeCapability;
  60: };
```
**EN:** This block stores supporting state such as computeCapability, which other APIs in the file consume.
**CN:** 该代码块声明了 computeCapability 等支撑状态，供本文件中的其他 API 使用。

### Lines 62-62
```cpp
  62: } // namespace mlir::triton::nvidia_gpu
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 64-64
```cpp
  64: #endif // TRITON_DIALECT_TRITONNVIDIAGPU_IR_TARGETFEATURES_H_
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** attributes  
  **CN:** 属性
- **EN:** types  
  **CN:** 类型
- **EN:** NVIDIA backend support  
  **CN:** NVIDIA 后端支持
- **EN:** MLIR integration  
  **CN:** MLIR 集成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/IR/BuiltinAttributes.h`
  - `mlir/IR/BuiltinOps.h`
  - `triton/Dialect/TritonGPU/IR/Dialect.h`
- **System or external includes / 系统或外部依赖:**
  - `<cassert>`
