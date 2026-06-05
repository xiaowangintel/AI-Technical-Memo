# AMDGPU.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `clang/lib/CIR/CodeGen/Targets/AMDGPU.cpp`
- **Repository**: `/root/xw/llvm-project` (`llvm-project`)
- **Purpose (EN)**: This provides AMDGPU-specific CIR CodeGen logic for function attributes.
- **Purpose (CN)**: 实现与 `AMDGPU` 相关的 CIR 代码生成支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
   1: //===---- AMDGPU.cpp - AMDGPU-specific CIR CodeGen ------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This provides AMDGPU-specific CIR CodeGen logic for function attributes.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "../CIRGenModule.h"
  14: #include "../TargetInfo.h"
  15: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `CIRGenModule.h`, `TargetInfo.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `CIRGenModule.h`, `TargetInfo.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 16-25
```cpp
  16: #include "clang/AST/Attr.h"
  17: #include "clang/AST/Decl.h"
  18: #include "clang/Basic/TargetInfo.h"
  19: #include "clang/CIR/Dialect/IR/CIRDialect.h"
  20: #include "llvm/ADT/StringExtras.h"
  21: #include "llvm/Support/raw_ostream.h"
  22: 
  23: using namespace clang;
  24: using namespace clang::CIRGen;
  25: 
```
- **EN**: This block establishes the file-level dependency set and any header-guard structure. Included headers like `Attr.h`, `Decl.h`, `TargetInfo.h`, `CIRDialect.h` reveal the main APIs consumed by this region.
- **CN**: 该代码块建立文件级依赖集合，并在需要时给出头文件保护结构。 像 `Attr.h`, `Decl.h`, `TargetInfo.h`, `CIRDialect.h` 这样的头文件说明了该区域依赖的主要 API。

### Lines 26-30
```cpp
  26: bool clang::CIRGen::requiresAMDGPUProtectedVisibility(
  27:     const Decl *d, cir::VisibilityKind visibility) {
  28:   if (visibility != cir::VisibilityKind::Hidden)
  29:     return false;
  30: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `clang::CIRGen::requiresAMDGPUProtectedVisibility`. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `clang::CIRGen::requiresAMDGPUProtectedVisibility`。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 31-41
```cpp
  31:   return !d->hasAttr<OMPDeclareTargetDeclAttr>() &&
  32:          (d->hasAttr<DeviceKernelAttr>() ||
  33:           (isa<FunctionDecl>(d) && d->hasAttr<CUDAGlobalAttr>()) ||
  34:           (isa<VarDecl>(d) &&
  35:            (d->hasAttr<CUDADeviceAttr>() || d->hasAttr<CUDAConstantAttr>() ||
  36:             cast<VarDecl>(d)->getType()->isCUDADeviceBuiltinSurfaceType() ||
  37:             cast<VarDecl>(d)->getType()->isCUDADeviceBuiltinTextureType())));
  38: }
  39: 
  40: namespace {
  41: 
```
- **EN**: This block organizes symbols with namespaces or using-declarations for shorter references. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块通过命名空间或 using 声明组织符号并简化后续引用。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 42-50
```cpp
  42: /// Handle amdgpu-flat-work-group-size attribute.
  43: static void
  44: handleAMDGPUFlatWorkGroupSizeAttr(const FunctionDecl *fd, cir::FuncOp func,
  45:                                   CIRGenModule &cgm, CIRGenBuilderTy &builder,
  46:                                   bool isOpenCLKernel, bool isHIPKernel) {
  47:   const auto *flatWGS = fd->getAttr<AMDGPUFlatWorkGroupSizeAttr>();
  48:   const auto *reqdWGS =
  49:       cgm.getLangOpts().OpenCL ? fd->getAttr<ReqdWorkGroupSizeAttr>() : nullptr;
  50: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `handleAMDGPUFlatWorkGroupSizeAttr`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `handleAMDGPUFlatWorkGroupSizeAttr`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 51-68
```cpp
  51:   if (flatWGS || reqdWGS) {
  52:     unsigned min = 0, max = 0;
  53:     if (flatWGS) {
  54:       min = flatWGS->getMin()
  55:                 ->EvaluateKnownConstInt(cgm.getASTContext())
  56:                 .getExtValue();
  57:       max = flatWGS->getMax()
  58:                 ->EvaluateKnownConstInt(cgm.getASTContext())
  59:                 .getExtValue();
  60:     }
  61:     if (reqdWGS && min == 0 && max == 0) {
  62:       min = max = reqdWGS->getXDim()
  63:                       ->EvaluateKnownConstInt(cgm.getASTContext())
  64:                       .getExtValue() *
  65:                   reqdWGS->getYDim()
  66:                       ->EvaluateKnownConstInt(cgm.getASTContext())
  67:                       .getExtValue() *
  68:                   reqdWGS->getZDim()
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 69-86
```cpp
  69:                       ->EvaluateKnownConstInt(cgm.getASTContext())
  70:                       .getExtValue();
  71:     }
  72:     if (min != 0) {
  73:       assert(min <= max && "Min must be less than or equal Max");
  74:       std::string attrVal = llvm::utostr(min) + "," + llvm::utostr(max);
  75:       func->setAttr("cir.amdgpu-flat-work-group-size",
  76:                     builder.getStringAttr(attrVal));
  77:     } else {
  78:       assert(max == 0 && "Max must be zero");
  79:     }
  80:   } else if (isOpenCLKernel || isHIPKernel) {
  81:     // By default, restrict the maximum size to a value specified by
  82:     // --gpu-max-threads-per-block=n or its default value for HIP.
  83:     const unsigned openCLDefaultMaxWorkGroupSize = 256;
  84:     const unsigned defaultMaxWorkGroupSize =
  85:         isOpenCLKernel ? openCLDefaultMaxWorkGroupSize
  86:                        : cgm.getLangOpts().GPUMaxThreadsPerBlock;
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。

### Lines 87-93
```cpp
  87:     std::string attrVal =
  88:         std::string("1,") + llvm::utostr(defaultMaxWorkGroupSize);
  89:     func->setAttr("cir.amdgpu-flat-work-group-size",
  90:                   builder.getStringAttr(attrVal));
  91:   }
  92: }
  93: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `std::string`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `std::string`。

### Lines 94-108
```cpp
  94: /// Handle amdgpu-waves-per-eu attribute.
  95: static void handleAMDGPUWavesPerEUAttr(const FunctionDecl *fd, cir::FuncOp func,
  96:                                        CIRGenModule &cgm,
  97:                                        CIRGenBuilderTy &builder) {
  98:   const auto *attr = fd->getAttr<AMDGPUWavesPerEUAttr>();
  99:   if (!attr)
 100:     return;
 101:   unsigned min =
 102:       attr->getMin()->EvaluateKnownConstInt(cgm.getASTContext()).getExtValue();
 103:   unsigned max = attr->getMax()
 104:                      ? attr->getMax()
 105:                            ->EvaluateKnownConstInt(cgm.getASTContext())
 106:                            .getExtValue()
 107:                      : 0;
 108: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `handleAMDGPUWavesPerEUAttr`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `handleAMDGPUWavesPerEUAttr`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 109-119
```cpp
 109:   if (min != 0) {
 110:     assert((max == 0 || min <= max) && "Min must be less than or equal Max");
 111:     std::string attrVal = llvm::utostr(min);
 112:     if (max != 0)
 113:       attrVal = attrVal + "," + llvm::utostr(max);
 114:     func->setAttr("cir.amdgpu-waves-per-eu", builder.getStringAttr(attrVal));
 115:   } else {
 116:     assert(max == 0 && "Max must be zero");
 117:   }
 118: }
 119: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `assert`. Conditional branches guard special cases, feature checks, or error paths. Assertions document invariants that the implementation expects to hold.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `assert`。 条件分支用于保护特殊情况、特性检查或错误路径。 断言用于说明实现期望始终成立的不变量。

### Lines 120-127
```cpp
 120: /// Handle amdgpu-num-sgpr attribute.
 121: static void handleAMDGPUNumSGPRAttr(const FunctionDecl *fd, cir::FuncOp func,
 122:                                     CIRGenModule &cgm,
 123:                                     CIRGenBuilderTy &builder) {
 124:   const auto *attr = fd->getAttr<AMDGPUNumSGPRAttr>();
 125:   if (!attr)
 126:     return;
 127: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `handleAMDGPUNumSGPRAttr`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `handleAMDGPUNumSGPRAttr`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 128-134
```cpp
 128:   uint32_t numSGPR = attr->getNumSGPR();
 129:   if (numSGPR != 0) {
 130:     func->setAttr("cir.amdgpu-num-sgpr",
 131:                   builder.getStringAttr(llvm::utostr(numSGPR)));
 132:   }
 133: }
 134: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 135-142
```cpp
 135: /// Handle amdgpu-num-vgpr attribute.
 136: static void handleAMDGPUNumVGPRAttr(const FunctionDecl *fd, cir::FuncOp func,
 137:                                     CIRGenModule &cgm,
 138:                                     CIRGenBuilderTy &builder) {
 139:   const auto *attr = fd->getAttr<AMDGPUNumVGPRAttr>();
 140:   if (!attr)
 141:     return;
 142: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `handleAMDGPUNumVGPRAttr`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `handleAMDGPUNumVGPRAttr`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 143-149
```cpp
 143:   uint32_t numVGPR = attr->getNumVGPR();
 144:   if (numVGPR != 0) {
 145:     func->setAttr("cir.amdgpu-num-vgpr",
 146:                   builder.getStringAttr(llvm::utostr(numVGPR)));
 147:   }
 148: }
 149: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 150-167
```cpp
 150: /// Handle amdgpu-max-num-workgroups attribute.
 151: static void handleAMDGPUMaxNumWorkGroupsAttr(const FunctionDecl *fd,
 152:                                              cir::FuncOp func,
 153:                                              CIRGenModule &cgm,
 154:                                              CIRGenBuilderTy &builder) {
 155:   const auto *attr = fd->getAttr<AMDGPUMaxNumWorkGroupsAttr>();
 156:   if (!attr)
 157:     return;
 158:   uint32_t x = attr->getMaxNumWorkGroupsX()
 159:                    ->EvaluateKnownConstInt(cgm.getASTContext())
 160:                    .getExtValue();
 161:   uint32_t y = attr->getMaxNumWorkGroupsY()
 162:                    ? attr->getMaxNumWorkGroupsY()
 163:                          ->EvaluateKnownConstInt(cgm.getASTContext())
 164:                          .getExtValue()
 165:                    : 1;
 166:   uint32_t z = attr->getMaxNumWorkGroupsZ()
 167:                    ? attr->getMaxNumWorkGroupsZ()
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `handleAMDGPUMaxNumWorkGroupsAttr`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `handleAMDGPUMaxNumWorkGroupsAttr`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 168-171
```cpp
 168:                          ->EvaluateKnownConstInt(cgm.getASTContext())
 169:                          .getExtValue()
 170:                    : 1;
 171: 
```
- **EN**: This block initializes data, constants, or helper objects used by surrounding logic.
- **CN**: 该代码块初始化被周边逻辑使用的数据、常量或辅助对象。

### Lines 172-178
```cpp
 172:   llvm::SmallString<32> attrVal;
 173:   llvm::raw_svector_ostream os(attrVal);
 174:   os << x << ',' << y << ',' << z;
 175:   func->setAttr("cir.amdgpu-max-num-workgroups",
 176:                 builder.getStringAttr(attrVal.str()));
 177: }
 178: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `os`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `os`。

### Lines 179-184
```cpp
 179: /// Handle amdgpu-cluster-dims attribute.
 180: static void handleAMDGPUClusterDimsAttr(const FunctionDecl *fd,
 181:                                         cir::FuncOp func, CIRGenModule &cgm,
 182:                                         CIRGenBuilderTy &builder,
 183:                                         bool isOpenCLKernel) {
 184: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `handleAMDGPUClusterDimsAttr`. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `handleAMDGPUClusterDimsAttr`。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 185-193
```cpp
 185:   if (const auto *attr = fd->getAttr<CUDAClusterDimsAttr>()) {
 186:     auto getExprVal = [&](const Expr *e) {
 187:       return e ? e->EvaluateKnownConstInt(cgm.getASTContext()).getExtValue()
 188:                : 1;
 189:     };
 190:     unsigned x = getExprVal(attr->getX());
 191:     unsigned y = getExprVal(attr->getY());
 192:     unsigned z = getExprVal(attr->getZ());
 193: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. Return statements hand back computed state, helper objects, or status values.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 返回语句将计算结果、辅助对象或状态值交回调用方。

### Lines 194-200
```cpp
 194:     llvm::SmallString<32> attrVal;
 195:     llvm::raw_svector_ostream os(attrVal);
 196:     os << x << ',' << y << ',' << z;
 197:     func->setAttr("cir.amdgpu-cluster-dims",
 198:                   builder.getStringAttr(attrVal.str()));
 199:   }
 200: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `os`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `os`。

### Lines 201-209
```cpp
 201:   const TargetInfo &targetInfo = cgm.getASTContext().getTargetInfo();
 202:   if ((isOpenCLKernel &&
 203:        targetInfo.hasFeatureEnabled(targetInfo.getTargetOpts().FeatureMap,
 204:                                     "clusters")) ||
 205:       fd->hasAttr<CUDANoClusterAttr>()) {
 206:     func->setAttr("cir.amdgpu-cluster-dims", builder.getStringAttr("0,0,0"));
 207:   }
 208: }
 209: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。

### Lines 210-216
```cpp
 210: /// Handle amdgpu-ieee attribute.
 211: static void handleAMDGPUIEEEAttr(cir::FuncOp func, CIRGenModule &cgm,
 212:                                  CIRGenBuilderTy &builder) {
 213:   if (!cgm.getCodeGenOpts().EmitIEEENaNCompliantInsts)
 214:     func->setAttr("cir.amdgpu-ieee", builder.getStringAttr("false"));
 215: }
 216: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `handleAMDGPUIEEEAttr`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `handleAMDGPUIEEEAttr`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 217-227
```cpp
 217: /// Handle amdgpu-expand-waitcnt-profiling attribute.
 218: static void handleAMDGPUExpandWaitcntProfilingAttr(cir::FuncOp func,
 219:                                                    CIRGenModule &cgm,
 220:                                                    CIRGenBuilderTy &builder) {
 221:   if (cgm.getCodeGenOpts().AMDGPUExpandWaitcntProfiling)
 222:     func->setAttr("cir.amdgpu-expand-waitcnt-profiling",
 223:                   builder.getStringAttr(""));
 224: }
 225: 
 226: } // namespace
 227: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `handleAMDGPUExpandWaitcntProfilingAttr`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `handleAMDGPUExpandWaitcntProfilingAttr`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 228-235
```cpp
 228: void clang::CIRGen::setAMDGPUTargetFunctionAttributes(const Decl *decl,
 229:                                                       cir::FuncOp func,
 230:                                                       CIRGenModule &cgm) {
 231:   if (func.isDeclaration())
 232:     return;
 233: 
 234:   CIRGenBuilderTy &builder = cgm.getBuilder();
 235: 
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `clang::CIRGen::setAMDGPUTargetFunctionAttributes`. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `clang::CIRGen::setAMDGPUTargetFunctionAttributes`。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 236-245
```cpp
 236:   const auto *fd = dyn_cast_or_null<FunctionDecl>(decl);
 237:   if (fd) {
 238:     const bool isOpenCLKernel =
 239:         cgm.getLangOpts().OpenCL && fd->hasAttr<DeviceKernelAttr>();
 240:     const bool isHIPKernel =
 241:         cgm.getLangOpts().HIP && fd->hasAttr<CUDAGlobalAttr>();
 242: 
 243:     if (isHIPKernel)
 244:       func.setCallingConv(cir::CallingConv::AMDGPUKernel);
 245: 
```
- **EN**: This block contains local control-flow decisions or iterative processing. Conditional branches guard special cases, feature checks, or error paths. MLIR/CIR symbols show that the logic is tied to CIR IR construction or transformation.
- **CN**: 该代码块包含局部控制流决策或迭代处理逻辑。 条件分支用于保护特殊情况、特性检查或错误路径。 出现 `mlir::`/`cir::` 符号，说明该逻辑与 CIR IR 的构建或变换直接相关。

### Lines 246-256
```cpp
 246:     handleAMDGPUFlatWorkGroupSizeAttr(fd, func, cgm, builder, isOpenCLKernel,
 247:                                       isHIPKernel);
 248:     handleAMDGPUWavesPerEUAttr(fd, func, cgm, builder);
 249:     handleAMDGPUNumSGPRAttr(fd, func, cgm, builder);
 250:     handleAMDGPUNumVGPRAttr(fd, func, cgm, builder);
 251:     handleAMDGPUMaxNumWorkGroupsAttr(fd, func, cgm, builder);
 252:     handleAMDGPUClusterDimsAttr(fd, func, cgm, builder, isOpenCLKernel);
 253:   }
 254:   handleAMDGPUIEEEAttr(func, cgm, builder);
 255:   handleAMDGPUExpandWaitcntProfilingAttr(func, cgm, builder);
 256: }
```
- **EN**: This block defines or declares executable logic for the file. Notable callable symbols here include `handleAMDGPUFlatWorkGroupSizeAttr`, `handleAMDGPUWavesPerEUAttr`, `handleAMDGPUNumSGPRAttr`, `handleAMDGPUNumVGPRAttr`.
- **CN**: 该代码块定义或声明了该文件中的可执行逻辑。 其中值得关注的可调用符号包括 `handleAMDGPUFlatWorkGroupSizeAttr`、`handleAMDGPUWavesPerEUAttr`、`handleAMDGPUNumSGPRAttr`、`handleAMDGPUNumVGPRAttr`。

## Key Concepts / 关键概念

- **Clang AST integration / Clang AST 集成**: Reads semantic information from Clang AST data structures. 从 Clang AST 数据结构读取语义信息。
- **Declaration handling / 声明处理**: Works with declaration nodes that describe source-level entities. 处理描述源级实体的声明节点。
- **CIR dialect usage / CIR 方言使用**: Manipulates CIR-specific types, attributes, and operations. 操作 CIR 专用的类型、属性和操作。
- **CIR generation pipeline / CIR 生成流水线**: Participates in the lowering pipeline from Clang semantics to CIR constructs. 参与从 Clang 语义到 CIR 构造的 lowering 流水线。
- **CUDA support / CUDA 支持**: Contains logic related to CUDA-specific code generation or runtime handling. 包含与 CUDA 专用代码生成或运行时处理相关的逻辑。
- **`clang::CIRGen::requiresAMDGPUProtectedVisibility` / `clang::CIRGen::requiresAMDGPUProtectedVisibility`**: `clang::CIRGen::requiresAMDGPUProtectedVisibility` is a prominent symbol in this file and helps define its structure or behavior. `clang::CIRGen::requiresAMDGPUProtectedVisibility` 是该文件中的关键符号之一，用来界定其结构或行为。

## Dependencies / 依赖关系

- **Local/Internal / 本地/内部**: `../CIRGenModule.h`, `../TargetInfo.h`
- **Clang / Clang**: `clang/AST/Attr.h`, `clang/AST/Decl.h`, `clang/Basic/TargetInfo.h`, `clang/CIR/Dialect/IR/CIRDialect.h`
- **LLVM / LLVM**: `llvm/ADT/StringExtras.h`, `llvm/Support/raw_ostream.h`
