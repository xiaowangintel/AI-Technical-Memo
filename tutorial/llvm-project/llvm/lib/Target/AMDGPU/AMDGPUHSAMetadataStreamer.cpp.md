# AMDGPUHSAMetadataStreamer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUHSAMetadataStreamer.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUHSAMetadataStreamer for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUHSAMetadataStreamer 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: File banner, includes, and setup
```cpp
//===--- AMDGPUHSAMetadataStreamer.cpp --------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// AMDGPU HSA Metadata Streamer.
///
//
//===----------------------------------------------------------------------===//

#include "AMDGPUHSAMetadataStreamer.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUTargetStreamer.h"
#include "SIMachineFunctionInfo.h"
#include "SIProgramInfo.h"
#include "llvm/IR/Module.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/Target/TargetLoweringObjectFile.h"

using namespace llvm;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 28-57: Defines getArgumentTypeAlign
```cpp
static std::pair<Type *, Align> getArgumentTypeAlign(const Argument &Arg,
                                                     const DataLayout &DL) {
  Type *Ty = Arg.getType();
  MaybeAlign ArgAlign;
  if (Arg.hasByRefAttr()) {
    Ty = Arg.getParamByRefType();
    ArgAlign = Arg.getParamAlign();
  }

  if (!ArgAlign)
    ArgAlign = DL.getABITypeAlign(Ty);

  return std::pair(Ty, *ArgAlign);
}

/// Find the mangled symbol name for the runtime handle for \p EnqueuedBlock
static std::string getEnqueuedBlockSymbolName(const AMDGPUTargetMachine &TM,
                                              const Function &EnqueuedBlock) {
  const MDNode *Associated =
      EnqueuedBlock.getMetadata(LLVMContext::MD_associated);
  if (!Associated)
    return "";

  auto *VM = cast<ValueAsMetadata>(Associated->getOperand(0));
  auto *RuntimeHandle =
      dyn_cast<GlobalVariable>(VM->getValue()->stripPointerCasts());
  if (!RuntimeHandle ||
      RuntimeHandle->getSection() != ".amdgpu.kernel.runtime.handle")
    return "";

```
**EN:** This section contains concrete logic for getArgumentTypeAlign. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::pair`.
**CN:** 本节包含与 getArgumentTypeAlign 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::pair`。

### Lines 58-87: Defines getNameWithPrefix
```cpp
  SmallString<128> Name;
  TM.getNameWithPrefix(Name, RuntimeHandle,
                       TM.getObjFileLowering()->getMangler());
  return Name.str().str();
}

namespace llvm {

static cl::opt<bool> DumpHSAMetadata(
    "amdgpu-dump-hsa-metadata",
    cl::desc("Dump AMDGPU HSA Metadata"));
static cl::opt<bool> VerifyHSAMetadata(
    "amdgpu-verify-hsa-metadata",
    cl::desc("Verify AMDGPU HSA Metadata"));

namespace AMDGPU::HSAMD {

//===----------------------------------------------------------------------===//
// HSAMetadataStreamerV4
//===----------------------------------------------------------------------===//

void MetadataStreamerMsgPackV4::dump(StringRef HSAMetadataString) const {
  errs() << "AMDGPU HSA Metadata:\n" << HSAMetadataString << '\n';
}

void MetadataStreamerMsgPackV4::verify(StringRef HSAMetadataString) const {
  errs() << "AMDGPU HSA Metadata Parser Test: ";

  msgpack::Document FromHSAMetadataString;

```
**EN:** This section contains concrete logic for getNameWithPrefix. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `cl::desc`, `MetadataStreamerMsgPackV4::dump`, `MetadataStreamerMsgPackV4::verify`.
**CN:** 本节包含与 getNameWithPrefix 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`cl::desc`, `MetadataStreamerMsgPackV4::dump`, `MetadataStreamerMsgPackV4::verify`。

### Lines 88-112: Declares StrOS
```cpp
  if (!FromHSAMetadataString.fromYAML(HSAMetadataString)) {
    errs() << "FAIL\n";
    return;
  }

  std::string ToHSAMetadataString;
  raw_string_ostream StrOS(ToHSAMetadataString);
  FromHSAMetadataString.toYAML(StrOS);

  errs() << (HSAMetadataString == StrOS.str() ? "PASS" : "FAIL") << '\n';
  if (HSAMetadataString != ToHSAMetadataString) {
    errs() << "Original input: " << HSAMetadataString << '\n'
           << "Produced output: " << StrOS.str() << '\n';
  }
}

std::optional<StringRef>
MetadataStreamerMsgPackV4::getAccessQualifier(StringRef AccQual) const {
  return StringSwitch<std::optional<StringRef>>(AccQual)
      .Case("read_only", StringRef("read_only"))
      .Case("write_only", StringRef("write_only"))
      .Case("read_write", StringRef("read_write"))
      .Default(std::nullopt);
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `MetadataStreamerMsgPackV4::getAccessQualifier`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`MetadataStreamerMsgPackV4::getAccessQualifier`。

### Lines 113-138: Implements MetadataStreamerMsgPackV4::getAddressSpaceQualifier
```cpp
std::optional<StringRef> MetadataStreamerMsgPackV4::getAddressSpaceQualifier(
    unsigned AddressSpace) const {
  switch (AddressSpace) {
  case AMDGPUAS::PRIVATE_ADDRESS:
    return StringRef("private");
  case AMDGPUAS::GLOBAL_ADDRESS:
    return StringRef("global");
  case AMDGPUAS::CONSTANT_ADDRESS:
    return StringRef("constant");
  case AMDGPUAS::LOCAL_ADDRESS:
    return StringRef("local");
  case AMDGPUAS::FLAT_ADDRESS:
    return StringRef("generic");
  case AMDGPUAS::REGION_ADDRESS:
    return StringRef("region");
  default:
    return std::nullopt;
  }
}

StringRef
MetadataStreamerMsgPackV4::getValueKind(Type *Ty, StringRef TypeQual,
                                        StringRef BaseTypeName) const {
  if (TypeQual.contains("pipe"))
    return "pipe";

```
**EN:** This section contains concrete logic for MetadataStreamerMsgPackV4::getAddressSpaceQualifier. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MetadataStreamerMsgPackV4::getAddressSpaceQualifier`, `MetadataStreamerMsgPackV4::getValueKind`.
**CN:** 本节包含与 MetadataStreamerMsgPackV4::getAddressSpaceQualifier 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MetadataStreamerMsgPackV4::getAddressSpaceQualifier`, `MetadataStreamerMsgPackV4::getValueKind`。

### Lines 139-167: Switch-based control flow
```cpp
  return StringSwitch<StringRef>(BaseTypeName)
      .Case("image1d_t", "image")
      .Case("image1d_array_t", "image")
      .Case("image1d_buffer_t", "image")
      .Case("image2d_t", "image")
      .Case("image2d_array_t", "image")
      .Case("image2d_array_depth_t", "image")
      .Case("image2d_array_msaa_t", "image")
      .Case("image2d_array_msaa_depth_t", "image")
      .Case("image2d_depth_t", "image")
      .Case("image2d_msaa_t", "image")
      .Case("image2d_msaa_depth_t", "image")
      .Case("image3d_t", "image")
      .Case("sampler_t", "sampler")
      .Case("queue_t", "queue")
      .Default(isa<PointerType>(Ty)
                   ? (Ty->getPointerAddressSpace() == AMDGPUAS::LOCAL_ADDRESS
                          ? "dynamic_shared_pointer"
                          : "global_buffer")
                   : "by_value");
}

std::string MetadataStreamerMsgPackV4::getTypeName(Type *Ty,
                                                   bool Signed) const {
  switch (Ty->getTypeID()) {
  case Type::IntegerTyID: {
    if (!Signed)
      return (Twine('u') + getTypeName(Ty, true)).str();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MetadataStreamerMsgPackV4::getTypeName`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MetadataStreamerMsgPackV4::getTypeName`。

### Lines 168-198: Switch-based control flow
```cpp
    auto BitWidth = Ty->getIntegerBitWidth();
    switch (BitWidth) {
    case 8:
      return "char";
    case 16:
      return "short";
    case 32:
      return "int";
    case 64:
      return "long";
    default:
      return (Twine('i') + Twine(BitWidth)).str();
    }
  }
  case Type::HalfTyID:
    return "half";
  case Type::FloatTyID:
    return "float";
  case Type::DoubleTyID:
    return "double";
  case Type::FixedVectorTyID: {
    auto *VecTy = cast<FixedVectorType>(Ty);
    auto *ElTy = VecTy->getElementType();
    auto NumElements = VecTy->getNumElements();
    return (Twine(getTypeName(ElTy, Signed)) + Twine(NumElements)).str();
  }
  default:
    return "unknown";
  }
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 199-228: Implements MetadataStreamerMsgPackV4::getWorkGroupDimensions
```cpp
msgpack::ArrayDocNode
MetadataStreamerMsgPackV4::getWorkGroupDimensions(MDNode *Node) const {
  auto Dims = HSAMetadataDoc->getArrayNode();
  if (Node->getNumOperands() != 3)
    return Dims;

  for (auto &Op : Node->operands())
    Dims.push_back(Dims.getDocument()->getNode(
        mdconst::extract<ConstantInt>(Op)->getZExtValue()));
  return Dims;
}

void MetadataStreamerMsgPackV4::emitVersion() {
  auto Version = HSAMetadataDoc->getArrayNode();
  Version.push_back(Version.getDocument()->getNode(VersionMajorV4));
  Version.push_back(Version.getDocument()->getNode(VersionMinorV4));
  getRootMetadata("amdhsa.version") = Version;
}

void MetadataStreamerMsgPackV4::emitTargetID(
    const IsaInfo::AMDGPUTargetID &TargetID) {
  getRootMetadata("amdhsa.target") =
      HSAMetadataDoc->getNode(TargetID.toString(), /*Copy=*/true);
}

void MetadataStreamerMsgPackV4::emitPrintf(const Module &Mod) {
  auto *Node = Mod.getNamedMetadata("llvm.printf.fmts");
  if (!Node)
    return;

```
**EN:** This section contains concrete logic for MetadataStreamerMsgPackV4::getWorkGroupDimensions. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MetadataStreamerMsgPackV4::getWorkGroupDimensions`, `MetadataStreamerMsgPackV4::emitVersion`, `MetadataStreamerMsgPackV4::emitTargetID`.
**CN:** 本节包含与 MetadataStreamerMsgPackV4::getWorkGroupDimensions 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MetadataStreamerMsgPackV4::getWorkGroupDimensions`, `MetadataStreamerMsgPackV4::emitVersion`, `MetadataStreamerMsgPackV4::emitTargetID`。

### Lines 229-255: Declares getArrayNode
```cpp
  auto Printf = HSAMetadataDoc->getArrayNode();
  for (auto *Op : Node->operands())
    if (Op->getNumOperands())
      Printf.push_back(Printf.getDocument()->getNode(
          cast<MDString>(Op->getOperand(0))->getString(), /*Copy=*/true));
  getRootMetadata("amdhsa.printf") = Printf;
}

void MetadataStreamerMsgPackV4::emitKernelLanguage(const Function &Func,
                                                   msgpack::MapDocNode Kern) {
  // TODO: What about other languages?
  auto *Node = Func.getParent()->getNamedMetadata("opencl.ocl.version");
  if (!Node || !Node->getNumOperands())
    return;
  auto *Op0 = Node->getOperand(0);
  if (Op0->getNumOperands() <= 1)
    return;

  Kern[".language"] = Kern.getDocument()->getNode("OpenCL C");
  auto LanguageVersion = Kern.getDocument()->getArrayNode();
  LanguageVersion.push_back(Kern.getDocument()->getNode(
      mdconst::extract<ConstantInt>(Op0->getOperand(0))->getZExtValue()));
  LanguageVersion.push_back(Kern.getDocument()->getNode(
      mdconst::extract<ConstantInt>(Op0->getOperand(1))->getZExtValue()));
  Kern[".language_version"] = LanguageVersion;
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `MetadataStreamerMsgPackV4::emitKernelLanguage`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`MetadataStreamerMsgPackV4::emitKernelLanguage`。

### Lines 256-283: Implements MetadataStreamerMsgPackV4::emitKernelAttrs
```cpp
void MetadataStreamerMsgPackV4::emitKernelAttrs(const AMDGPUTargetMachine &TM,
                                                const MachineFunction &MF,
                                                msgpack::MapDocNode Kern) {
  const Function &Func = MF.getFunction();
  if (auto *Node = Func.getMetadata("reqd_work_group_size"))
    Kern[".reqd_workgroup_size"] = getWorkGroupDimensions(Node);
  if (auto *Node = Func.getMetadata("work_group_size_hint"))
    Kern[".workgroup_size_hint"] = getWorkGroupDimensions(Node);
  if (auto *Node = Func.getMetadata("vec_type_hint")) {
    Kern[".vec_type_hint"] = Kern.getDocument()->getNode(
        getTypeName(
            cast<ValueAsMetadata>(Node->getOperand(0))->getType(),
            mdconst::extract<ConstantInt>(Node->getOperand(1))->getZExtValue()),
        /*Copy=*/true);
  }

  std::string HandleName = getEnqueuedBlockSymbolName(TM, Func);
  if (!HandleName.empty()) {
    Kern[".device_enqueue_symbol"] =
        Kern.getDocument()->getNode(std::move(HandleName), /*Copy=*/true);
  }

  if (Func.hasFnAttribute("device-init"))
    Kern[".kind"] = Kern.getDocument()->getNode("init");
  else if (Func.hasFnAttribute("device-fini"))
    Kern[".kind"] = Kern.getDocument()->getNode("fini");
}

```
**EN:** This section contains concrete logic for MetadataStreamerMsgPackV4::emitKernelAttrs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MetadataStreamerMsgPackV4::emitKernelAttrs`, `std::move`.
**CN:** 本节包含与 MetadataStreamerMsgPackV4::emitKernelAttrs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MetadataStreamerMsgPackV4::emitKernelAttrs`, `std::move`。

### Lines 284-314: Implements MetadataStreamerMsgPackV4::emitKernelArgs
```cpp
void MetadataStreamerMsgPackV4::emitKernelArgs(const MachineFunction &MF,
                                               msgpack::MapDocNode Kern) {
  auto &Func = MF.getFunction();
  unsigned Offset = 0;
  auto Args = HSAMetadataDoc->getArrayNode();
  for (auto &Arg : Func.args()) {
    if (Arg.hasAttribute("amdgpu-hidden-argument"))
      continue;

    emitKernelArg(Arg, Offset, Args);
  }

  emitHiddenKernelArgs(MF, Offset, Args);

  Kern[".args"] = Args;
}

void MetadataStreamerMsgPackV4::emitKernelArg(const Argument &Arg,
                                              unsigned &Offset,
                                              msgpack::ArrayDocNode Args) {
  const auto *Func = Arg.getParent();
  auto ArgNo = Arg.getArgNo();
  const MDNode *Node;

  StringRef Name;
  Node = Func->getMetadata("kernel_arg_name");
  if (Node && ArgNo < Node->getNumOperands())
    Name = cast<MDString>(Node->getOperand(ArgNo))->getString();
  else if (Arg.hasName())
    Name = Arg.getName();

```
**EN:** This section contains concrete logic for MetadataStreamerMsgPackV4::emitKernelArgs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MetadataStreamerMsgPackV4::emitKernelArgs`, `MetadataStreamerMsgPackV4::emitKernelArg`.
**CN:** 本节包含与 MetadataStreamerMsgPackV4::emitKernelArgs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MetadataStreamerMsgPackV4::emitKernelArgs`, `MetadataStreamerMsgPackV4::emitKernelArg`。

### Lines 315-348: Declares getMetadata
```cpp
  StringRef TypeName;
  Node = Func->getMetadata("kernel_arg_type");
  if (Node && ArgNo < Node->getNumOperands())
    TypeName = cast<MDString>(Node->getOperand(ArgNo))->getString();

  StringRef BaseTypeName;
  Node = Func->getMetadata("kernel_arg_base_type");
  if (Node && ArgNo < Node->getNumOperands())
    BaseTypeName = cast<MDString>(Node->getOperand(ArgNo))->getString();

  StringRef ActAccQual;
  // Do we really need NoAlias check here?
  if (Arg.getType()->isPointerTy() && Arg.hasNoAliasAttr()) {
    if (Arg.onlyReadsMemory())
      ActAccQual = "read_only";
    else if (Arg.hasAttribute(Attribute::WriteOnly))
      ActAccQual = "write_only";
  }

  StringRef AccQual;
  Node = Func->getMetadata("kernel_arg_access_qual");
  if (Node && ArgNo < Node->getNumOperands())
    AccQual = cast<MDString>(Node->getOperand(ArgNo))->getString();

  StringRef TypeQual;
  Node = Func->getMetadata("kernel_arg_type_qual");
  if (Node && ArgNo < Node->getNumOperands())
    TypeQual = cast<MDString>(Node->getOperand(ArgNo))->getString();

  const DataLayout &DL = Func->getDataLayout();

  MaybeAlign PointeeAlign;
  Type *Ty = Arg.hasByRefAttr() ? Arg.getParamByRefType() : Arg.getType();

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 349-372: Implements std::tie
```cpp
  // FIXME: Need to distinguish in memory alignment from pointer alignment.
  if (auto *PtrTy = dyn_cast<PointerType>(Ty)) {
    if (PtrTy->getAddressSpace() == AMDGPUAS::LOCAL_ADDRESS)
      PointeeAlign = Arg.getParamAlign().valueOrOne();
  }

  // There's no distinction between byval aggregates and raw aggregates.
  Type *ArgTy;
  Align ArgAlign;
  std::tie(ArgTy, ArgAlign) = getArgumentTypeAlign(Arg, DL);

  emitKernelArg(DL, ArgTy, ArgAlign,
                getValueKind(ArgTy, TypeQual, BaseTypeName), Offset, Args,
                PointeeAlign, Name, TypeName, BaseTypeName, ActAccQual,
                AccQual, TypeQual);
}

void MetadataStreamerMsgPackV4::emitKernelArg(
    const DataLayout &DL, Type *Ty, Align Alignment, StringRef ValueKind,
    unsigned &Offset, msgpack::ArrayDocNode Args, MaybeAlign PointeeAlign,
    StringRef Name, StringRef TypeName, StringRef BaseTypeName,
    StringRef ActAccQual, StringRef AccQual, StringRef TypeQual) {
  auto Arg = Args.getDocument()->getMapNode();

```
**EN:** This section contains concrete logic for std::tie. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::tie`, `MetadataStreamerMsgPackV4::emitKernelArg`.
**CN:** 本节包含与 std::tie 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::tie`, `MetadataStreamerMsgPackV4::emitKernelArg`。

### Lines 373-398: Declares getTypeAllocSize
```cpp
  if (!Name.empty())
    Arg[".name"] = Arg.getDocument()->getNode(Name, /*Copy=*/true);
  if (!TypeName.empty())
    Arg[".type_name"] = Arg.getDocument()->getNode(TypeName, /*Copy=*/true);
  auto Size = DL.getTypeAllocSize(Ty);
  Arg[".size"] = Arg.getDocument()->getNode(Size);
  Offset = alignTo(Offset, Alignment);
  Arg[".offset"] = Arg.getDocument()->getNode(Offset);
  Offset += Size;
  Arg[".value_kind"] = Arg.getDocument()->getNode(ValueKind, /*Copy=*/true);
  if (PointeeAlign)
    Arg[".pointee_align"] = Arg.getDocument()->getNode(PointeeAlign->value());

  if (auto *PtrTy = dyn_cast<PointerType>(Ty))
    if (auto Qualifier = getAddressSpaceQualifier(PtrTy->getAddressSpace()))
      // Limiting address space to emit only for a certain ValueKind.
      if (ValueKind == "global_buffer" || ValueKind == "dynamic_shared_pointer")
        Arg[".address_space"] = Arg.getDocument()->getNode(*Qualifier,
                                                           /*Copy=*/true);

  if (auto AQ = getAccessQualifier(AccQual))
    Arg[".access"] = Arg.getDocument()->getNode(*AQ, /*Copy=*/true);

  if (auto AAQ = getAccessQualifier(ActAccQual))
    Arg[".actual_access"] = Arg.getDocument()->getNode(*AAQ, /*Copy=*/true);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 399-429: Declares split
```cpp
  SmallVector<StringRef, 1> SplitTypeQuals;
  TypeQual.split(SplitTypeQuals, " ", -1, false);
  for (StringRef Key : SplitTypeQuals) {
    if (Key == "const")
      Arg[".is_const"] = Arg.getDocument()->getNode(true);
    else if (Key == "restrict")
      Arg[".is_restrict"] = Arg.getDocument()->getNode(true);
    else if (Key == "volatile")
      Arg[".is_volatile"] = Arg.getDocument()->getNode(true);
    else if (Key == "pipe")
      Arg[".is_pipe"] = Arg.getDocument()->getNode(true);
  }

  Args.push_back(Arg);
}

void MetadataStreamerMsgPackV4::emitHiddenKernelArgs(
    const MachineFunction &MF, unsigned &Offset, msgpack::ArrayDocNode Args) {
  auto &Func = MF.getFunction();
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();

  unsigned HiddenArgNumBytes = ST.getImplicitArgNumBytes(Func);
  if (!HiddenArgNumBytes)
    return;

  const Module *M = Func.getParent();
  auto &DL = M->getDataLayout();
  auto *Int64Ty = Type::getInt64Ty(Func.getContext());

  Offset = alignTo(Offset, ST.getAlignmentForImplicitArgPtr());

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `MetadataStreamerMsgPackV4::emitHiddenKernelArgs`, `Type::getInt64Ty`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`MetadataStreamerMsgPackV4::emitHiddenKernelArgs`, `Type::getInt64Ty`。

### Lines 430-456: Conditional logic and checks
```cpp
  if (HiddenArgNumBytes >= 8)
    emitKernelArg(DL, Int64Ty, Align(8), "hidden_global_offset_x", Offset,
                  Args);
  if (HiddenArgNumBytes >= 16)
    emitKernelArg(DL, Int64Ty, Align(8), "hidden_global_offset_y", Offset,
                  Args);
  if (HiddenArgNumBytes >= 24)
    emitKernelArg(DL, Int64Ty, Align(8), "hidden_global_offset_z", Offset,
                  Args);

  auto *Int8PtrTy =
      PointerType::get(Func.getContext(), AMDGPUAS::GLOBAL_ADDRESS);

  if (HiddenArgNumBytes >= 32) {
    // We forbid the use of features requiring hostcall when compiling OpenCL
    // before code object V5, which makes the mutual exclusion between the
    // "printf buffer" and "hostcall buffer" here sound.
    if (M->getNamedMetadata("llvm.printf.fmts"))
      emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_printf_buffer", Offset,
                    Args);
    else if (!Func.hasFnAttribute("amdgpu-no-hostcall-ptr"))
      emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_hostcall_buffer", Offset,
                    Args);
    else
      emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_none", Offset, Args);
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `PointerType::get`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`PointerType::get`。

### Lines 457-487: Conditional logic and checks
```cpp
  // Emit "default queue" and "completion action" arguments if enqueue kernel is
  // used, otherwise emit dummy "none" arguments.
  if (HiddenArgNumBytes >= 40) {
    if (!Func.hasFnAttribute("amdgpu-no-default-queue")) {
      emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_default_queue", Offset,
                    Args);
    } else {
      emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_none", Offset, Args);
    }
  }

  if (HiddenArgNumBytes >= 48) {
    if (!Func.hasFnAttribute("amdgpu-no-completion-action")) {
      emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_completion_action", Offset,
                    Args);
    } else {
      emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_none", Offset, Args);
    }
  }

  // Emit the pointer argument for multi-grid object.
  if (HiddenArgNumBytes >= 56) {
    if (!Func.hasFnAttribute("amdgpu-no-multigrid-sync-arg")) {
      emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_multigrid_sync_arg", Offset,
                    Args);
    } else {
      emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_none", Offset, Args);
    }
  }
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 488-514: Implements MetadataStreamerMsgPackV4::getHSAKernelProps
```cpp
msgpack::MapDocNode
MetadataStreamerMsgPackV4::getHSAKernelProps(const MachineFunction &MF,
                                             const SIProgramInfo &ProgramInfo,
                                             unsigned CodeObjectVersion) const {
  const GCNSubtarget &STM = MF.getSubtarget<GCNSubtarget>();
  const SIMachineFunctionInfo &MFI = *MF.getInfo<SIMachineFunctionInfo>();
  const Function &F = MF.getFunction();

  auto Kern = HSAMetadataDoc->getMapNode();

  Align MaxKernArgAlign;
  Kern[".kernarg_segment_size"] = Kern.getDocument()->getNode(
      STM.getKernArgSegmentSize(F, MaxKernArgAlign));
  Kern[".group_segment_fixed_size"] =
      Kern.getDocument()->getNode(ProgramInfo.LDSSize);
  DelayedExprs->assignDocNode(Kern[".private_segment_fixed_size"],
                              msgpack::Type::UInt, ProgramInfo.ScratchSize);
  if (CodeObjectVersion >= AMDGPU::AMDHSA_COV5) {
    DelayedExprs->assignDocNode(Kern[".uses_dynamic_stack"],
                                msgpack::Type::Boolean,
                                ProgramInfo.DynamicCallStack);
  }

  if (CodeObjectVersion >= AMDGPU::AMDHSA_COV5 && STM.supportsWGP())
    Kern[".workgroup_processor_mode"] =
        Kern.getDocument()->getNode(ProgramInfo.WgpMode);

```
**EN:** This section contains concrete logic for MetadataStreamerMsgPackV4::getHSAKernelProps. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MetadataStreamerMsgPackV4::getHSAKernelProps`.
**CN:** 本节包含与 MetadataStreamerMsgPackV4::getHSAKernelProps 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MetadataStreamerMsgPackV4::getHSAKernelProps`。

### Lines 515-547: Defines assignDocNode
```cpp
  // FIXME: The metadata treats the minimum as 16?
  Kern[".kernarg_segment_align"] =
      Kern.getDocument()->getNode(std::max(Align(4), MaxKernArgAlign).value());
  Kern[".wavefront_size"] =
      Kern.getDocument()->getNode(STM.getWavefrontSize());
  DelayedExprs->assignDocNode(Kern[".sgpr_count"], msgpack::Type::UInt,
                              ProgramInfo.NumSGPR);
  DelayedExprs->assignDocNode(Kern[".vgpr_count"], msgpack::Type::UInt,
                              ProgramInfo.NumVGPR);

  // Only add AGPR count to metadata for supported devices
  if (STM.hasMAIInsts()) {
    DelayedExprs->assignDocNode(Kern[".agpr_count"], msgpack::Type::UInt,
                                ProgramInfo.NumAccVGPR);
  }

  Kern[".max_flat_workgroup_size"] =
      Kern.getDocument()->getNode(MFI.getMaxFlatWorkGroupSize());

  uint32_t NumWGY = MFI.getMaxNumWorkGroupsY();
  uint32_t NumWGZ = MFI.getMaxNumWorkGroupsZ();
  uint32_t NumWGX = MFI.getMaxNumWorkGroupsX();

  // TODO: Should consider 0 invalid and reject in IR verifier.
  if (NumWGX != std::numeric_limits<uint32_t>::max() && NumWGX != 0)
    Kern[".max_num_workgroups_x"] = Kern.getDocument()->getNode(NumWGX);

  if (NumWGY != std::numeric_limits<uint32_t>::max() && NumWGY != 0)
    Kern[".max_num_workgroups_y"] = Kern.getDocument()->getNode(NumWGY);

  if (NumWGZ != std::numeric_limits<uint32_t>::max() && NumWGZ != 0)
    Kern[".max_num_workgroups_z"] = Kern.getDocument()->getNode(NumWGZ);

```
**EN:** This section contains concrete logic for assignDocNode. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::max`.
**CN:** 本节包含与 assignDocNode 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::max`。

### Lines 548-581: Implements MetadataStreamerMsgPackV4::emitTo
```cpp
  Kern[".sgpr_spill_count"] =
      Kern.getDocument()->getNode(MFI.getNumSpilledSGPRs());
  Kern[".vgpr_spill_count"] =
      Kern.getDocument()->getNode(MFI.getNumSpilledVGPRs());

  return Kern;
}

bool MetadataStreamerMsgPackV4::emitTo(AMDGPUTargetStreamer &TargetStreamer) {
  DelayedExprs->resolveDelayedExpressions();
  return TargetStreamer.EmitHSAMetadata(*HSAMetadataDoc, true);
}

void MetadataStreamerMsgPackV4::begin(const Module &Mod,
                                      const IsaInfo::AMDGPUTargetID &TargetID) {
  emitVersion();
  emitTargetID(TargetID);
  emitPrintf(Mod);
  getRootMetadata("amdhsa.kernels") = HSAMetadataDoc->getArrayNode();
  DelayedExprs->clear();
}

void MetadataStreamerMsgPackV4::end() {
  DelayedExprs->resolveDelayedExpressions();
  std::string HSAMetadataString;
  raw_string_ostream StrOS(HSAMetadataString);
  HSAMetadataDoc->toYAML(StrOS);

  if (DumpHSAMetadata)
    dump(StrOS.str());
  if (VerifyHSAMetadata)
    verify(StrOS.str());
}

```
**EN:** This section contains concrete logic for MetadataStreamerMsgPackV4::emitTo. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MetadataStreamerMsgPackV4::emitTo`, `MetadataStreamerMsgPackV4::begin`, `MetadataStreamerMsgPackV4::end`.
**CN:** 本节包含与 MetadataStreamerMsgPackV4::emitTo 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MetadataStreamerMsgPackV4::emitTo`, `MetadataStreamerMsgPackV4::begin`, `MetadataStreamerMsgPackV4::end`。

### Lines 582-612: Implements MetadataStreamerMsgPackV4::emitKernel
```cpp
void MetadataStreamerMsgPackV4::emitKernel(const MachineFunction &MF,
                                           const SIProgramInfo &ProgramInfo) {
  auto &Func = MF.getFunction();
  if (Func.getCallingConv() != CallingConv::AMDGPU_KERNEL &&
      Func.getCallingConv() != CallingConv::SPIR_KERNEL)
    return;

  auto CodeObjectVersion =
      AMDGPU::getAMDHSACodeObjectVersion(*Func.getParent());
  auto Kern = getHSAKernelProps(MF, ProgramInfo, CodeObjectVersion);

  auto Kernels =
      getRootMetadata("amdhsa.kernels").getArray(/*Convert=*/true);

  auto &TM = static_cast<const AMDGPUTargetMachine &>(MF.getTarget());
  {
    Kern[".name"] = Kern.getDocument()->getNode(Func.getName());
    Kern[".symbol"] = Kern.getDocument()->getNode(
        (Twine(Func.getName()) + Twine(".kd")).str(), /*Copy=*/true);
    emitKernelLanguage(Func, Kern);
    emitKernelAttrs(TM, MF, Kern);
    emitKernelArgs(MF, Kern);
  }

  Kernels.push_back(Kern);
}

//===----------------------------------------------------------------------===//
// HSAMetadataStreamerV5
//===----------------------------------------------------------------------===//

```
**EN:** This section contains concrete logic for MetadataStreamerMsgPackV4::emitKernel. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MetadataStreamerMsgPackV4::emitKernel`, `AMDGPU::getAMDHSACodeObjectVersion`.
**CN:** 本节包含与 MetadataStreamerMsgPackV4::emitKernel 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MetadataStreamerMsgPackV4::emitKernel`, `AMDGPU::getAMDHSACodeObjectVersion`。

### Lines 613-645: Implements MetadataStreamerMsgPackV5::emitVersion
```cpp
void MetadataStreamerMsgPackV5::emitVersion() {
  auto Version = HSAMetadataDoc->getArrayNode();
  Version.push_back(Version.getDocument()->getNode(VersionMajorV5));
  Version.push_back(Version.getDocument()->getNode(VersionMinorV5));
  getRootMetadata("amdhsa.version") = Version;
}

void MetadataStreamerMsgPackV5::emitHiddenKernelArgs(
    const MachineFunction &MF, unsigned &Offset, msgpack::ArrayDocNode Args) {
  auto &Func = MF.getFunction();
  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();

  // No implicit kernel argument is used.
  if (ST.getImplicitArgNumBytes(Func) == 0)
    return;

  const Module *M = Func.getParent();
  auto &DL = M->getDataLayout();
  const SIMachineFunctionInfo &MFI = *MF.getInfo<SIMachineFunctionInfo>();

  auto *Int64Ty = Type::getInt64Ty(Func.getContext());
  auto *Int32Ty = Type::getInt32Ty(Func.getContext());
  auto *Int16Ty = Type::getInt16Ty(Func.getContext());

  Offset = alignTo(Offset, ST.getAlignmentForImplicitArgPtr());
  emitKernelArg(DL, Int32Ty, Align(4), "hidden_block_count_x", Offset, Args);
  emitKernelArg(DL, Int32Ty, Align(4), "hidden_block_count_y", Offset, Args);
  emitKernelArg(DL, Int32Ty, Align(4), "hidden_block_count_z", Offset, Args);

  emitKernelArg(DL, Int16Ty, Align(2), "hidden_group_size_x", Offset, Args);
  emitKernelArg(DL, Int16Ty, Align(2), "hidden_group_size_y", Offset, Args);
  emitKernelArg(DL, Int16Ty, Align(2), "hidden_group_size_z", Offset, Args);

```
**EN:** This section contains concrete logic for MetadataStreamerMsgPackV5::emitVersion. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MetadataStreamerMsgPackV5::emitVersion`, `MetadataStreamerMsgPackV5::emitHiddenKernelArgs`, `Type::getInt64Ty`.
**CN:** 本节包含与 MetadataStreamerMsgPackV5::emitVersion 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MetadataStreamerMsgPackV5::emitVersion`, `MetadataStreamerMsgPackV5::emitHiddenKernelArgs`, `Type::getInt64Ty`。

### Lines 646-678: Declares emitKernelArg
```cpp
  emitKernelArg(DL, Int16Ty, Align(2), "hidden_remainder_x", Offset, Args);
  emitKernelArg(DL, Int16Ty, Align(2), "hidden_remainder_y", Offset, Args);
  emitKernelArg(DL, Int16Ty, Align(2), "hidden_remainder_z", Offset, Args);

  // Reserved for hidden_tool_correlation_id.
  Offset += 8;

  Offset += 8; // Reserved.

  emitKernelArg(DL, Int64Ty, Align(8), "hidden_global_offset_x", Offset, Args);
  emitKernelArg(DL, Int64Ty, Align(8), "hidden_global_offset_y", Offset, Args);
  emitKernelArg(DL, Int64Ty, Align(8), "hidden_global_offset_z", Offset, Args);

  emitKernelArg(DL, Int16Ty, Align(2), "hidden_grid_dims", Offset, Args);

  Offset += 6; // Reserved.
  auto *Int8PtrTy =
      PointerType::get(Func.getContext(), AMDGPUAS::GLOBAL_ADDRESS);

  if (M->getNamedMetadata("llvm.printf.fmts")) {
    emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_printf_buffer", Offset,
                  Args);
  } else {
    Offset += 8; // Skipped.
  }

  if (!Func.hasFnAttribute("amdgpu-no-hostcall-ptr")) {
    emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_hostcall_buffer", Offset,
                  Args);
  } else {
    Offset += 8; // Skipped.
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `PointerType::get`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`PointerType::get`。

### Lines 679-712: Conditional logic and checks
```cpp
  if (!Func.hasFnAttribute("amdgpu-no-multigrid-sync-arg")) {
    emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_multigrid_sync_arg", Offset,
                Args);
  } else {
    Offset += 8; // Skipped.
  }

  if (!Func.hasFnAttribute("amdgpu-no-heap-ptr"))
    emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_heap_v1", Offset, Args);
  else
    Offset += 8; // Skipped.

  if (!Func.hasFnAttribute("amdgpu-no-default-queue")) {
    emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_default_queue", Offset,
                  Args);
  } else {
    Offset += 8; // Skipped.
  }

  if (!Func.hasFnAttribute("amdgpu-no-completion-action")) {
    emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_completion_action", Offset,
                  Args);
  } else {
    Offset += 8; // Skipped.
  }

  // Emit argument for hidden dynamic lds size
  if (MFI.isDynamicLDSUsed()) {
    emitKernelArg(DL, Int32Ty, Align(4), "hidden_dynamic_lds_size", Offset,
                  Args);
  } else {
    Offset += 4; // skipped
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 713-741: Conditional logic and checks
```cpp
  Offset += 68; // Reserved.

  // hidden_private_base and hidden_shared_base are only when the subtarget has
  // ApertureRegs.
  if (!ST.hasApertureRegs()) {
    emitKernelArg(DL, Int32Ty, Align(4), "hidden_private_base", Offset, Args);
    emitKernelArg(DL, Int32Ty, Align(4), "hidden_shared_base", Offset, Args);
  } else {
    Offset += 8; // Skipped.
  }

  if (MFI.getUserSGPRInfo().hasQueuePtr())
    emitKernelArg(DL, Int8PtrTy, Align(8), "hidden_queue_ptr", Offset, Args);
}

void MetadataStreamerMsgPackV5::emitKernelAttrs(const AMDGPUTargetMachine &TM,
                                                const MachineFunction &MF,
                                                msgpack::MapDocNode Kern) {
  MetadataStreamerMsgPackV4::emitKernelAttrs(TM, MF, Kern);

  const Function &Func = MF.getFunction();
  if (Func.hasFnAttribute("uniform-work-group-size"))
    Kern[".uniform_work_group_size"] = Kern.getDocument()->getNode(1);
}

//===----------------------------------------------------------------------===//
// HSAMetadataStreamerV6
//===----------------------------------------------------------------------===//

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MetadataStreamerMsgPackV5::emitKernelAttrs`, `MetadataStreamerMsgPackV4::emitKernelAttrs`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MetadataStreamerMsgPackV5::emitKernelAttrs`, `MetadataStreamerMsgPackV4::emitKernelAttrs`。

### Lines 742-766: Implements MetadataStreamerMsgPackV6::emitVersion
```cpp
void MetadataStreamerMsgPackV6::emitVersion() {
  auto Version = HSAMetadataDoc->getArrayNode();
  Version.push_back(Version.getDocument()->getNode(VersionMajorV6));
  Version.push_back(Version.getDocument()->getNode(VersionMinorV6));
  getRootMetadata("amdhsa.version") = Version;
}

void MetadataStreamerMsgPackV6::emitKernelAttrs(const AMDGPUTargetMachine &TM,
                                                const MachineFunction &MF,
                                                msgpack::MapDocNode Kern) {
  MetadataStreamerMsgPackV5::emitKernelAttrs(TM, MF, Kern);

  const SIMachineFunctionInfo &MFI = *MF.getInfo<SIMachineFunctionInfo>();
  ClusterDimsAttr Attr = MFI.getClusterDims();
  if (Attr.isFixedDims()) {
    msgpack::ArrayDocNode ClusterDimsNode = HSAMetadataDoc->getArrayNode();
    ClusterDimsNode.push_back(HSAMetadataDoc->getNode(Attr.getDims()[0]));
    ClusterDimsNode.push_back(HSAMetadataDoc->getNode(Attr.getDims()[1]));
    ClusterDimsNode.push_back(HSAMetadataDoc->getNode(Attr.getDims()[2]));
    Kern[".cluster_dims"] = ClusterDimsNode;
  }
}

} // end namespace AMDGPU::HSAMD
} // end namespace llvm
```
**EN:** This section contains concrete logic for MetadataStreamerMsgPackV6::emitVersion. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MetadataStreamerMsgPackV6::emitVersion`, `MetadataStreamerMsgPackV6::emitKernelAttrs`, `MetadataStreamerMsgPackV5::emitKernelAttrs`.
**CN:** 本节包含与 MetadataStreamerMsgPackV6::emitVersion 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MetadataStreamerMsgPackV6::emitVersion`, `MetadataStreamerMsgPackV6::emitKernelAttrs`, `MetadataStreamerMsgPackV5::emitKernelAttrs`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `std::pair`, `cl::desc`, `MetadataStreamerMsgPackV4::dump`, `MetadataStreamerMsgPackV4::verify`, `MetadataStreamerMsgPackV4::getAccessQualifier`, `MetadataStreamerMsgPackV4::getAddressSpaceQualifier`
- **Main themes / 核心主题**: subtarget modeling / 子目标建模; lowering / 降低; assembly handling / 汇编处理; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUHSAMetadataStreamer.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUTargetStreamer.h"`
- `"SIMachineFunctionInfo.h"`
- `"SIProgramInfo.h"`
- `"llvm/IR/Module.h"`
- `"llvm/MC/MCContext.h"`
- `"llvm/MC/MCExpr.h"`
- `"llvm/Target/TargetLoweringObjectFile.h"`
