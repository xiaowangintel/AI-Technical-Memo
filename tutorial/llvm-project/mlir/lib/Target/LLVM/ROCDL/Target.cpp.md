# Target.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Target/LLVM/ROCDL/Target.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This files defines ROCDL target related functions including registration calls for the `#rocdl.target` compilation attribute.
  - **CN**: 实现 MLIR 目标导入/导出、翻译或序列化支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- Target.cpp - MLIR LLVM ROCDL target compilation ----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This files defines ROCDL target related functions including registration
// calls for the `#rocdl.target` compilation attribute.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及高层摘要。

### Lines 13-32
```cpp

#include "mlir/Target/LLVM/ROCDL/Target.h"

#include "mlir/Dialect/GPU/IR/GPUDialect.h"
#include "mlir/Dialect/LLVMIR/ROCDLDialect.h"
#include "mlir/Support/FileUtilities.h"
#include "mlir/Target/LLVM/ROCDL/Utils.h"
#include "mlir/Target/LLVMIR/Export.h"

#include "llvm/Config/Targets.h"
#include "llvm/IR/Constants.h"
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCObjectFileInfo.h"
#include "llvm/MC/MCObjectWriter.h"
#include "llvm/MC/MCParser/MCTargetAsmParser.h"
#include "llvm/MC/MCRegisterInfo.h"
```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Target/LLVM/ROCDL/Target.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/LLVMIR/ROCDLDialect.h`, `mlir/Support/FileUtilities.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Target/LLVM/ROCDL/Target.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/LLVMIR/ROCDLDialect.h`, `mlir/Support/FileUtilities.h`。

### Lines 33-43
```cpp
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/FileSystem.h"
#include "llvm/Support/FileUtilities.h"
#include "llvm/Support/Path.h"
#include "llvm/Support/Program.h"
#include "llvm/Support/SourceMgr.h"
#include "llvm/Support/TargetSelect.h"
#include "llvm/TargetParser/TargetParser.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `llvm/MC/MCStreamer.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/FileSystem.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `llvm/MC/MCStreamer.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/FileSystem.h`。

### Lines 44-53
```cpp
#include <cstdlib>
#include <optional>

using namespace mlir;
using namespace mlir::ROCDL;

#ifndef __DEFAULT_ROCM_PATH__
#define __DEFAULT_ROCM_PATH__ ""
#endif

```
- **EN**: Pulls in the declarations needed by this translation unit, including `cstdlib`, `optional`.
- **CN**: 引入该编译单元所需的声明，其中包括 `cstdlib`, `optional`。

### Lines 54-68
```cpp
namespace {
// Implementation of the `TargetAttrInterface` model.
class ROCDLTargetAttrImpl
    : public gpu::TargetAttrInterface::FallbackModel<ROCDLTargetAttrImpl> {
public:
  std::optional<mlir::gpu::SerializedObject>
  serializeToObject(Attribute attribute, Operation *module,
                    const gpu::TargetOptions &options) const;

  Attribute createObject(Attribute attribute, Operation *module,
                         const mlir::gpu::SerializedObject &object,
                         const gpu::TargetOptions &options) const;
};
} // namespace

```
- **EN**: Introduces declarations for `ROCDLTargetAttrImpl`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `ROCDLTargetAttrImpl` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 69-83
```cpp
// Register the ROCDL dialect, the ROCDL translation and the target interface.
void mlir::ROCDL::registerROCDLTargetInterfaceExternalModels(
    DialectRegistry &registry) {
  registry.addExtension(+[](MLIRContext *ctx, ROCDL::ROCDLDialect *dialect) {
    ROCDLTargetAttr::attachInterface<ROCDLTargetAttrImpl>(*ctx);
  });
}

void mlir::ROCDL::registerROCDLTargetInterfaceExternalModels(
    MLIRContext &context) {
  DialectRegistry registry;
  registerROCDLTargetInterfaceExternalModels(registry);
  context.appendDialectRegistry(registry);
}

```
- **EN**: Implements logic around `registerROCDLTargetInterfaceExternalModels`, `addExtension`, `attachInterface`, `appendDialectRegistry`; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `registerROCDLTargetInterfaceExternalModels`、`addExtension`、`attachInterface`、`appendDialectRegistry` 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 84-94
```cpp
// Search for the ROCM path.
StringRef mlir::ROCDL::getROCMPath() {
  if (const char *var = std::getenv("ROCM_PATH"))
    return var;
  if (const char *var = std::getenv("ROCM_ROOT"))
    return var;
  if (const char *var = std::getenv("ROCM_HOME"))
    return var;
  return __DEFAULT_ROCM_PATH__;
}

```
- **EN**: Implements logic around `getROCMPath`, `getenv`; this block bridges MLIR-produced code with runtime entry points; connects IR to external target or serialization formats.
- **CN**: 围绕 `getROCMPath`、`getenv` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接，并把 IR 连接到外部目标或序列化格式。

### Lines 95-106
```cpp
SerializeGPUModuleBase::SerializeGPUModuleBase(
    Operation &module, ROCDLTargetAttr target,
    const gpu::TargetOptions &targetOptions)
    : ModuleToObject(module, target.getTriple(), target.getChip(),
                     target.getFeatures(), target.getO()),
      target(target), toolkitPath(targetOptions.getToolkitPath()),
      librariesToLink(targetOptions.getLibrariesToLink()) {

  // If `targetOptions` has an empty toolkitPath use `getROCMPath`
  if (toolkitPath.empty())
    toolkitPath = getROCMPath();

```
- **EN**: Implements logic around `SerializeGPUModuleBase`, `ModuleToObject`, `getFeatures`, `target`, and 3 more symbols; this block manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `SerializeGPUModuleBase`、`ModuleToObject`、`getFeatures`、`target` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 107-125
```cpp
  // Append the files in the target attribute.
  if (target.getLink())
    librariesToLink.append(target.getLink().begin(), target.getLink().end());
}

void SerializeGPUModuleBase::init() {
  static llvm::once_flag initializeBackendOnce;
  llvm::call_once(initializeBackendOnce, []() {
  // If the `AMDGPU` LLVM target was built, initialize it.
#if LLVM_HAS_AMDGPU_TARGET
    LLVMInitializeAMDGPUTarget();
    LLVMInitializeAMDGPUTargetInfo();
    LLVMInitializeAMDGPUTargetMC();
    LLVMInitializeAMDGPUAsmParser();
    LLVMInitializeAMDGPUAsmPrinter();
#endif
  });
}

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 126-138
```cpp
ROCDLTargetAttr SerializeGPUModuleBase::getTarget() const { return target; }

StringRef SerializeGPUModuleBase::getToolkitPath() const { return toolkitPath; }

ArrayRef<Attribute> SerializeGPUModuleBase::getLibrariesToLink() const {
  return librariesToLink;
}

LogicalResult SerializeGPUModuleBase::appendStandardLibs(AMDGCNLibraries libs) {
  if (libs == AMDGCNLibraries::None)
    return success();
  StringRef pathRef = getToolkitPath();

```
- **EN**: Implements logic around `getTarget`, `getToolkitPath`, `getLibrariesToLink`, `appendStandardLibs`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `getTarget`、`getToolkitPath`、`getLibrariesToLink`、`appendStandardLibs` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 139-151
```cpp
  // Get the path for the device libraries
  SmallString<256> path;
  path.insert(path.begin(), pathRef.begin(), pathRef.end());
  llvm::sys::path::append(path, "amdgcn", "bitcode");
  pathRef = StringRef(path.data(), path.size());

  // Fail if the path is invalid.
  if (!llvm::sys::fs::is_directory(pathRef)) {
    getOperation().emitError() << "ROCm amdgcn bitcode path: " << pathRef
                               << " does not exist or is not a directory";
    return failure();
  }

```
- **EN**: Implements logic around `insert`, `append`, `StringRef`, `is_directory`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; bridges MLIR-produced code with runtime entry points; connects IR to external target or serialization formats.
- **CN**: 围绕 `insert`、`append`、`StringRef`、`is_directory` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并在 MLIR 生成代码与运行时入口点之间建立桥接，并把 IR 连接到外部目标或序列化格式。

### Lines 152-166
```cpp
  // Helper function for adding a library.
  auto addLib = [&](const Twine &lib) -> bool {
    auto baseSize = path.size();
    llvm::sys::path::append(path, lib);
    StringRef pathRef(path.data(), path.size());
    if (!llvm::sys::fs::is_regular_file(pathRef)) {
      getOperation().emitRemark() << "bitcode library path: " << pathRef
                                  << " does not exist or is not a file";
      return true;
    }
    librariesToLink.push_back(StringAttr::get(target.getContext(), pathRef));
    path.truncate(baseSize);
    return false;
  };

```
- **EN**: Implements logic around `size`, `append`, `pathRef`, `is_regular_file`, and 3 more symbols; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `size`、`append`、`pathRef`、`is_regular_file` 等另外 3 个符号 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 167-176
```cpp
  // Add ROCm device libraries. Fail if any of the libraries is not found, ie.
  // if any of the `addLib` failed.
  if ((any(libs & AMDGCNLibraries::Ocml) && addLib("ocml.bc")) ||
      (any(libs & AMDGCNLibraries::Ockl) && addLib("ockl.bc")) ||
      (any(libs & AMDGCNLibraries::Hip) && addLib("hip.bc")) ||
      (any(libs & AMDGCNLibraries::OpenCL) && addLib("opencl.bc")))
    return failure();
  return success();
}

```
- **EN**: Implements logic around `any`, `failure`, `success`; this block bridges MLIR-produced code with runtime entry points; connects IR to external target or serialization formats.
- **CN**: 围绕 `any`、`failure`、`success` 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接，并把 IR 连接到外部目标或序列化格式。

### Lines 177-190
```cpp
std::optional<SmallVector<std::unique_ptr<llvm::Module>>>
SerializeGPUModuleBase::loadBitcodeFiles(llvm::Module &module) {
  // Return if there are no libs to load.
  if (deviceLibs == AMDGCNLibraries::None && librariesToLink.empty())
    return SmallVector<std::unique_ptr<llvm::Module>>();
  if (failed(appendStandardLibs(deviceLibs)))
    return std::nullopt;
  SmallVector<std::unique_ptr<llvm::Module>> bcFiles;
  if (failed(loadBitcodeFilesFromList(module.getContext(), librariesToLink,
                                      bcFiles, true)))
    return std::nullopt;
  return std::move(bcFiles);
}

```
- **EN**: Implements logic around `loadBitcodeFiles`, `empty`, `Module>>`, `failed`, and 1 more symbols; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `loadBitcodeFiles`、`empty`、`Module>>`、`failed` 等另外 1 个符号 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 191-203
```cpp
LogicalResult SerializeGPUModuleBase::handleBitcodeFile(llvm::Module &module) {
  // Some ROCM builds don't strip this like they should
  if (auto *openclVersion = module.getNamedMetadata("opencl.ocl.version"))
    module.eraseNamedMetadata(openclVersion);
  // Stop spamming us with clang version numbers
  if (auto *ident = module.getNamedMetadata("llvm.ident"))
    module.eraseNamedMetadata(ident);
  // Override the libModules datalayout and target triple with the compiler's
  // data layout should there be a discrepency.
  setDataLayoutAndTriple(module);
  return success();
}

```
- **EN**: Implements logic around `handleBitcodeFile`, `getNamedMetadata`, `eraseNamedMetadata`, `setDataLayoutAndTriple`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; bridges MLIR-produced code with runtime entry points; connects IR to external target or serialization formats.
- **CN**: 围绕 `handleBitcodeFile`、`getNamedMetadata`、`eraseNamedMetadata`、`setDataLayoutAndTriple` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并在 MLIR 生成代码与运行时入口点之间建立桥接，并把 IR 连接到外部目标或序列化格式。

### Lines 204-223
```cpp
void SerializeGPUModuleBase::handleModulePreLink(llvm::Module &module) {
  // If all libraries are not set, traverse the module to determine which
  // libraries are required.
  if (deviceLibs != AMDGCNLibraries::All) {
    for (llvm::Function &f : module.functions()) {
      if (f.hasExternalLinkage() && f.hasName() && !f.hasExactDefinition()) {
        StringRef funcName = f.getName();
        if ("printf" == funcName)
          deviceLibs |= AMDGCNLibraries::OpenCL | AMDGCNLibraries::Ockl |
                        AMDGCNLibraries::Ocml;
        if (funcName.starts_with("__ockl_"))
          deviceLibs |= AMDGCNLibraries::Ockl;
        if (funcName.starts_with("__ocml_"))
          deviceLibs |= AMDGCNLibraries::Ocml;
        if (funcName == "__atomic_work_item_fence")
          deviceLibs |= AMDGCNLibraries::Hip;
      }
    }
  }
  addControlVariables(module, deviceLibs, target.hasWave64(), target.hasDaz(),
```
- **EN**: Implements logic around `handleModulePreLink`, `functions`, `hasExternalLinkage`, `getName`, and 2 more symbols; this block parses or prints textual MLIR representations; connects IR to external target or serialization formats.
- **CN**: 围绕 `handleModulePreLink`、`functions`、`hasExternalLinkage`、`getName` 等另外 2 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并把 IR 连接到外部目标或序列化格式。

### Lines 224-243
```cpp
                      target.hasFiniteOnly(), target.hasUnsafeMath(),
                      target.hasFastMath(), target.hasCorrectSqrt(),
                      target.getAbi());
}

void SerializeGPUModuleBase::addControlVariables(
    llvm::Module &module, AMDGCNLibraries libs, bool wave64, bool daz,
    bool finiteOnly, bool unsafeMath, bool fastMath, bool correctSqrt,
    StringRef abiVer) {
  // Helper function for adding control variables.
  auto addControlVariable = [&module](StringRef name, uint32_t value,
                                      uint32_t bitwidth) {
    if (module.getNamedGlobal(name))
      return;
    llvm::IntegerType *type =
        llvm::IntegerType::getIntNTy(module.getContext(), bitwidth);
    llvm::GlobalVariable *controlVariable = new llvm::GlobalVariable(
        module, /*isConstant=*/type, true,
        llvm::GlobalValue::LinkageTypes::LinkOnceODRLinkage,
        llvm::ConstantInt::get(type, value), name, /*before=*/nullptr,
```
- **EN**: Implements logic around `hasFiniteOnly`, `hasFastMath`, `getAbi`, `addControlVariables`, and 4 more symbols; this block manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `hasFiniteOnly`、`hasFastMath`、`getAbi`、`addControlVariables` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 244-263
```cpp
        /*threadLocalMode=*/llvm::GlobalValue::ThreadLocalMode::NotThreadLocal,
        /*addressSpace=*/4);
    controlVariable->setVisibility(
        llvm::GlobalValue::VisibilityTypes::ProtectedVisibility);
    controlVariable->setAlignment(llvm::MaybeAlign(bitwidth / 8));
    controlVariable->setUnnamedAddr(llvm::GlobalValue::UnnamedAddr::Local);
  };

  // Note that COV6 requires ROCm 6.3+.
  int abi = 600;
  abiVer.getAsInteger(0, abi);
  module.addModuleFlag(llvm::Module::Error, "amdhsa_code_object_version", abi);
  // Return if no device libraries are required.
  if (libs == AMDGCNLibraries::None)
    return;
  // Add ocml related control variables.
  if (any(libs & AMDGCNLibraries::Ocml)) {
    addControlVariable("__oclc_finite_only_opt", finiteOnly || fastMath, 8);
    addControlVariable("__oclc_daz_opt", daz || fastMath, 8);
    addControlVariable("__oclc_correctly_rounded_sqrt32",
```
- **EN**: Implements logic around `setVisibility`, `setAlignment`, `setUnnamedAddr`, `getAsInteger`, and 3 more symbols; this block bridges MLIR-produced code with runtime entry points; connects IR to external target or serialization formats.
- **CN**: 围绕 `setVisibility`、`setAlignment`、`setUnnamedAddr`、`getAsInteger` 等另外 3 个符号 实现具体逻辑；该代码块在 MLIR 生成代码与运行时入口点之间建立桥接，并把 IR 连接到外部目标或序列化格式。

### Lines 264-280
```cpp
                       correctSqrt && !fastMath, 8);
    addControlVariable("__oclc_unsafe_math_opt", unsafeMath || fastMath, 8);
  }
  // Add ocml or ockl related control variables.
  if (any(libs & (AMDGCNLibraries::Ocml | AMDGCNLibraries::Ockl))) {
    addControlVariable("__oclc_wavefrontsize64", wave64, 8);
    // Get the ISA version.
    llvm::AMDGPU::IsaVersion isaVersion = llvm::AMDGPU::getIsaVersion(chip);
    // Add the ISA control variable.
    addControlVariable("__oclc_ISA_version",
                       isaVersion.Minor + 100 * isaVersion.Stepping +
                           1000 * isaVersion.Major,
                       32);
    addControlVariable("__oclc_ABI_version", abi, 32);
  }
}

```
- **EN**: Implements logic around `addControlVariable`, `any`, `getIsaVersion`; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `addControlVariable`、`any`、`getIsaVersion` 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 281-294
```cpp
FailureOr<SmallVector<char, 0>>
mlir::ROCDL::assembleIsa(StringRef isa, StringRef targetTriple, StringRef chip,
                         StringRef features,
                         function_ref<InFlightDiagnostic()> emitError) {
  SmallVector<char, 0> result;
  llvm::raw_svector_ostream os(result);

  llvm::Triple triple(llvm::Triple::normalize(targetTriple));
  std::string error;
  const llvm::Target *target =
      llvm::TargetRegistry::lookupTarget(triple, error);
  if (!target)
    return emitError() << "failed to lookup target: " << error;

```
- **EN**: Implements logic around `assembleIsa`, `function_ref`, `os`, `triple`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `assembleIsa`、`function_ref`、`os`、`triple` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 295-305
```cpp
  llvm::SourceMgr srcMgr;
  // Copy buffer to ensure it's null terminated.
  srcMgr.AddNewSourceBuffer(llvm::MemoryBuffer::getMemBufferCopy(isa), SMLoc());

  const llvm::MCTargetOptions mcOptions;
  std::unique_ptr<llvm::MCRegisterInfo> mri(target->createMCRegInfo(triple));
  std::unique_ptr<llvm::MCAsmInfo> mai(
      target->createMCAsmInfo(*mri, triple, mcOptions));
  std::unique_ptr<llvm::MCSubtargetInfo> sti(
      target->createMCSubtargetInfo(triple, chip, features));

```
- **EN**: Implements logic around `AddNewSourceBuffer`, `mri`, `mai`, `createMCAsmInfo`, and 2 more symbols; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `AddNewSourceBuffer`、`mri`、`mai`、`createMCAsmInfo` 等另外 2 个符号 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 306-317
```cpp
  llvm::MCContext ctx(triple, *mai, *mri, *sti, &srcMgr);
  std::unique_ptr<llvm::MCObjectFileInfo> mofi(target->createMCObjectFileInfo(
      ctx, /*PIC=*/false, /*LargeCodeModel=*/false));
  ctx.setObjectFileInfo(mofi.get());

  SmallString<128> cwd;
  if (!llvm::sys::fs::current_path(cwd))
    ctx.setCompilationDir(cwd);

  std::unique_ptr<llvm::MCStreamer> mcStreamer;
  std::unique_ptr<llvm::MCInstrInfo> mcii(target->createMCInstrInfo());

```
- **EN**: Implements logic around `ctx`, `mofi`, `setObjectFileInfo`, `current_path`, and 2 more symbols; this block connects IR to external target or serialization formats.
- **CN**: 围绕 `ctx`、`mofi`、`setObjectFileInfo`、`current_path` 等另外 2 个符号 实现具体逻辑；该代码块把 IR 连接到外部目标或序列化格式。

### Lines 318-329
```cpp
  llvm::MCCodeEmitter *ce = target->createMCCodeEmitter(*mcii, ctx);
  llvm::MCAsmBackend *mab = target->createMCAsmBackend(*sti, *mri, mcOptions);
  mcStreamer.reset(target->createMCObjectStreamer(
      triple, ctx, std::unique_ptr<llvm::MCAsmBackend>(mab),
      mab->createObjectWriter(os), std::unique_ptr<llvm::MCCodeEmitter>(ce),
      *sti));

  std::unique_ptr<llvm::MCAsmParser> parser(
      createMCAsmParser(srcMgr, ctx, *mcStreamer, *mai));
  std::unique_ptr<llvm::MCTargetAsmParser> tap(
      target->createMCAsmParser(*sti, *parser, *mcii));

```
- **EN**: Implements logic around `createMCCodeEmitter`, `createMCAsmBackend`, `reset`, `MCAsmBackend>`, and 4 more symbols; this block parses or prints textual MLIR representations; connects IR to external target or serialization formats.
- **CN**: 围绕 `createMCCodeEmitter`、`createMCAsmBackend`、`reset`、`MCAsmBackend>` 等另外 4 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示，并把 IR 连接到外部目标或序列化格式。

### Lines 330-348
```cpp
  if (!tap)
    return emitError() << "assembler initialization error";

  parser->setTargetParser(*tap);
  parser->Run(false);
  return std::move(result);
}

FailureOr<SmallVector<char, 0>>
mlir::ROCDL::linkObjectCode(ArrayRef<char> objectCode, StringRef lldPath,
                            function_ref<InFlightDiagnostic()> emitError) {
  // Save the ISA binary to a temp file.
  int tempIsaBinaryFd = -1;
  SmallString<128> tempIsaBinaryFilename;
  if (llvm::sys::fs::createTemporaryFile("kernel%%", "o", tempIsaBinaryFd,
                                         tempIsaBinaryFilename))
    return emitError()
           << "failed to create a temporary file for dumping the ISA binary";

```
- **EN**: Implements logic around `emitError`, `setTargetParser`, `Run`, `move`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; parses or prints textual MLIR representations; connects IR to external target or serialization formats.
- **CN**: 围绕 `emitError`、`setTargetParser`、`Run`、`move` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并解析或打印文本形式的 MLIR 表示，并把 IR 连接到外部目标或序列化格式。

### Lines 349-361
```cpp
  llvm::FileRemover cleanupIsaBinary(tempIsaBinaryFilename);
  {
    llvm::raw_fd_ostream tempIsaBinaryOs(tempIsaBinaryFd, true);
    tempIsaBinaryOs << StringRef(objectCode.data(), objectCode.size());
    tempIsaBinaryOs.flush();
  }

  // Create a temp file for HSA code object.
  SmallString<128> tempHsacoFilename;
  if (llvm::sys::fs::createTemporaryFile("kernel", "hsaco", tempHsacoFilename))
    return emitError()
           << "failed to create a temporary file for the HSA code object";

```
- **EN**: Implements logic around `cleanupIsaBinary`, `tempIsaBinaryOs`, `StringRef`, `flush`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `cleanupIsaBinary`、`tempIsaBinaryOs`、`StringRef`、`flush` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 362-376
```cpp
  llvm::FileRemover cleanupHsaco(tempHsacoFilename);

  int lldResult = llvm::sys::ExecuteAndWait(
      lldPath,
      {"ld.lld", "-shared", tempIsaBinaryFilename, "-o", tempHsacoFilename});
  if (lldResult != 0)
    return emitError() << "lld invocation failed";

  // Load the HSA code object.
  auto hsacoFile =
      llvm::MemoryBuffer::getFile(tempHsacoFilename, /*IsText=*/false);
  if (!hsacoFile)
    return emitError()
           << "failed to read the HSA code object from the temp file";

```
- **EN**: Implements logic around `cleanupHsaco`, `ExecuteAndWait`, `emitError`, `getFile`; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `cleanupHsaco`、`ExecuteAndWait`、`emitError`、`getFile` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 377-388
```cpp
  StringRef buffer = (*hsacoFile)->getBuffer();

  return SmallVector<char, 0>(buffer.begin(), buffer.end());
}

FailureOr<SmallVector<char, 0>>
SerializeGPUModuleBase::compileToBinary(StringRef serializedISA) {
  auto errCallback = [&]() { return getOperation().emitError(); };
  // Assemble the ISA.
  FailureOr<SmallVector<char, 0>> isaBinary = ROCDL::assembleIsa(
      serializedISA, this->triple, this->chip, this->features, errCallback);

```
- **EN**: Implements logic around `getBuffer`, `begin`, `compileToBinary`, `getOperation`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `getBuffer`、`begin`、`compileToBinary`、`getOperation` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 389-399
```cpp
  if (failed(isaBinary))
    return failure();

  // Link the object code.
  llvm::SmallString<128> lldPath(toolkitPath);
  llvm::sys::path::append(lldPath, "llvm", "bin", "ld.lld");
  FailureOr<SmallVector<char, 0>> linkedCode =
      ROCDL::linkObjectCode(*isaBinary, lldPath, errCallback);
  if (failed(linkedCode))
    return failure();

```
- **EN**: Implements logic around `failed`, `failure`, `lldPath`, `append`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `failed`、`failure`、`lldPath`、`append` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 400-415
```cpp
  return linkedCode;
}

FailureOr<SmallVector<char, 0>> SerializeGPUModuleBase::moduleToObjectImpl(
    const gpu::TargetOptions &targetOptions, llvm::Module &llvmModule) {
  // Return LLVM IR if the compilation target is offload.
#define DEBUG_TYPE "serialize-to-llvm"
  LLVM_DEBUG({
    llvm::dbgs() << "LLVM IR for module: "
                 << cast<gpu::GPUModuleOp>(getOperation()).getNameAttr() << "\n"
                 << llvmModule << "\n";
  });
#undef DEBUG_TYPE
  if (targetOptions.getCompilationTarget() == gpu::CompilationTarget::Offload)
    return SerializeGPUModuleBase::moduleToObject(llvmModule);

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 416-428
```cpp
  FailureOr<llvm::TargetMachine *> targetMachine = getOrCreateTargetMachine();
  if (failed(targetMachine))
    return getOperation().emitError()
           << "target Machine unavailable for triple " << triple
           << ", can't compile with LLVM";

  // Translate the Module to ISA.
  FailureOr<SmallString<0>> serializedISA =
      translateModuleToISA(llvmModule, **targetMachine,
                           [&]() { return getOperation().emitError(); });
  if (failed(serializedISA))
    return getOperation().emitError() << "failed translating the module to ISA";

```
- **EN**: Implements logic around `getOrCreateTargetMachine`, `failed`, `getOperation`, `translateModuleToISA`; this block makes success/failure or diagnostics explicit through MLIR result utilities; connects IR to external target or serialization formats.
- **CN**: 围绕 `getOrCreateTargetMachine`、`failed`、`getOperation`、`translateModuleToISA` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并把 IR 连接到外部目标或序列化格式。

### Lines 429-439
```cpp
#define DEBUG_TYPE "serialize-to-isa"
  LLVM_DEBUG({
    llvm::dbgs() << "ISA for module: "
                 << cast<gpu::GPUModuleOp>(getOperation()).getNameAttr() << "\n"
                 << *serializedISA << "\n";
  });
#undef DEBUG_TYPE
  // Return ISA assembly code if the compilation target is assembly.
  if (targetOptions.getCompilationTarget() == gpu::CompilationTarget::Assembly)
    return SmallVector<char, 0>(serializedISA->begin(), serializedISA->end());

```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 440-450
```cpp
  // Compiling to binary requires a valid ROCm path, fail if it's not found.
  if (getToolkitPath().empty())
    return getOperation().emitError()
           << "invalid ROCm path, please set a valid path";

  // Compile to binary.
  return compileToBinary(*serializedISA);
}

#if LLVM_HAS_AMDGPU_TARGET
namespace {
```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 451-464
```cpp
class AMDGPUSerializer : public SerializeGPUModuleBase {
public:
  AMDGPUSerializer(Operation &module, ROCDLTargetAttr target,
                   const gpu::TargetOptions &targetOptions);

  FailureOr<SmallVector<char, 0>>
  moduleToObject(llvm::Module &llvmModule) override;

private:
  // Target options.
  gpu::TargetOptions targetOptions;
};
} // namespace

```
- **EN**: Introduces declarations for `AMDGPUSerializer`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `AMDGPUSerializer` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 465-475
```cpp
AMDGPUSerializer::AMDGPUSerializer(Operation &module, ROCDLTargetAttr target,
                                   const gpu::TargetOptions &targetOptions)
    : SerializeGPUModuleBase(module, target, targetOptions),
      targetOptions(targetOptions) {}

FailureOr<SmallVector<char, 0>>
AMDGPUSerializer::moduleToObject(llvm::Module &llvmModule) {
  return moduleToObjectImpl(targetOptions, llvmModule);
}
#endif // LLVM_HAS_AMDGPU_TARGET

```
- **EN**: Implements logic around `AMDGPUSerializer`, `SerializeGPUModuleBase`, `targetOptions`, `moduleToObject`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `AMDGPUSerializer`、`SerializeGPUModuleBase`、`targetOptions`、`moduleToObject` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 476-495
```cpp
std::optional<mlir::gpu::SerializedObject>
ROCDLTargetAttrImpl::serializeToObject(
    Attribute attribute, Operation *module,
    const gpu::TargetOptions &options) const {
  assert(module && "The module must be non null.");
  if (!module)
    return std::nullopt;
  if (!mlir::isa<gpu::GPUModuleOp>(module)) {
    module->emitError("module must be a GPU module");
    return std::nullopt;
  }
#if LLVM_HAS_AMDGPU_TARGET
  AMDGPUSerializer serializer(*module, cast<ROCDLTargetAttr>(attribute),
                              options);
  serializer.init();
  std::optional<SmallVector<char, 0>> binary = serializer.run();
  if (!binary)
    return std::nullopt;
  return gpu::SerializedObject{std::move(*binary)};
#else
```
- **EN**: Defines preprocessor-controlled structure, generated include points, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、生成代码插入点或编译期常量。

### Lines 496-515
```cpp
  module->emitError("the `AMDGPU` target was not built. Please enable it when "
                    "building LLVM");
  return std::nullopt;
#endif // LLVM_HAS_AMDGPU_TARGET
}

Attribute
ROCDLTargetAttrImpl::createObject(Attribute attribute, Operation *module,
                                  const mlir::gpu::SerializedObject &object,
                                  const gpu::TargetOptions &options) const {
  gpu::CompilationTarget format = options.getCompilationTarget();
  // If format is `fatbin` transform it to binary as `fatbin` is not yet
  // supported.
  gpu::KernelTableAttr kernels;
  if (format > gpu::CompilationTarget::Binary) {
    format = gpu::CompilationTarget::Binary;
    kernels = ROCDL::getKernelMetadata(module, object.getObject());
  }
  DictionaryAttr properties{};
  Builder builder(attribute.getContext());
```
- **EN**: Implements logic around `emitError`, `createObject`, `getCompilationTarget`, `getKernelMetadata`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `emitError`、`createObject`、`getCompilationTarget`、`getKernelMetadata` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

### Lines 516-520
```cpp
  StringAttr objectStr = builder.getStringAttr(
      StringRef(object.getObject().data(), object.getObject().size()));
  return builder.getAttr<gpu::ObjectAttr>(attribute, format, objectStr,
                                          properties, kernels);
}
```
- **EN**: Implements logic around `getStringAttr`, `StringRef`, `ObjectAttr>`; this block manipulates core MLIR IR objects; connects IR to external target or serialization formats.
- **CN**: 围绕 `getStringAttr`、`StringRef`、`ObjectAttr>` 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并把 IR 连接到外部目标或序列化格式。

## Key Concepts / 关键概念

- **Target translation / 目标翻译**:
  - **EN**: Connects MLIR programs to external target formats such as LLVM IR or runtime ABIs.
  - **CN**: 将 MLIR 程序连接到 LLVM IR 或运行时 ABI 等外部目标格式。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Target or format bridging / 目标或格式桥接**:
  - **EN**: IR is translated to, from, or alongside external target representations.
  - **CN**: 该文件把 IR 翻译到外部目标表示、从外部目标表示恢复，或与之协同工作。
- **LLVM ADT containers / LLVM ADT 容器**:
  - **EN**: The implementation uses LLVM containers for performance-conscious storage and traversal.
  - **CN**: 该实现使用 LLVM 容器来进行兼顾性能的存储与遍历。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Target/LLVM/ROCDL/Target.h`, `mlir/Dialect/GPU/IR/GPUDialect.h`, `mlir/Dialect/LLVMIR/ROCDLDialect.h`, `mlir/Support/FileUtilities.h`, `mlir/Target/LLVM/ROCDL/Utils.h`, `mlir/Target/LLVMIR/Export.h`, `llvm/Config/Targets.h`, `llvm/IR/Constants.h`, `llvm/MC/MCAsmBackend.h`, `llvm/MC/MCAsmInfo.h` ... (+17 more)
- **Standard-library headers / 标准库头文件**: `<cstdlib>`, `<optional>`
- **Subsystem categories / 子系统类别**: generic LLVM subsystem support / 通用 LLVM 子系统支持 (14), LLVM support-library helpers / LLVM Support 库辅助工具 (6), target translation support / 目标翻译支持 (3), shared MLIR support helpers / 共享的 MLIR 支持工具 (1), LLVM IR support APIs / LLVM IR 支持 API (1)
