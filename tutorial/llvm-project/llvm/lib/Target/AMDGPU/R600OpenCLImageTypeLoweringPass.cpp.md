# R600OpenCLImageTypeLoweringPass.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600OpenCLImageTypeLoweringPass.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements R600OpenCLImageTypeLoweringPass for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 R600OpenCLImageTypeLoweringPass 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: File banner, license, and overview
```cpp
//===- R600OpenCLImageTypeLoweringPass.cpp ------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This pass resolves calls to OpenCL image attribute, image resource ID and
/// sampler resource ID getter functions.
///
/// Image attributes (size and format) are expected to be passed to the kernel
/// as kernel arguments immediately following the image argument itself,
/// therefore this pass adds image size and format arguments to the kernel
/// functions in the module. The kernel functions with image arguments are
/// re-created using the new signature. The new arguments are added to the
/// kernel metadata with kernel_arg_type set to "image_size" or "image_format".
/// Note: this pass may invalidate pointers to functions.
///
/// Resource IDs of read-only images, write-only images and samplers are
/// defined to be their index among the kernel arguments of the same
/// type and access qualifier.
//
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 27-59: Header dependencies and setup
```cpp
#include "R600.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Metadata.h"
#include "llvm/IR/Module.h"
#include "llvm/Pass.h"
#include "llvm/Transforms/Utils/Cloning.h"

using namespace llvm;

static StringRef GetImageSizeFunc =         "llvm.OpenCL.image.get.size";
static StringRef GetImageFormatFunc =       "llvm.OpenCL.image.get.format";
static StringRef GetImageResourceIDFunc =   "llvm.OpenCL.image.get.resource.id";
static StringRef GetSamplerResourceIDFunc =
    "llvm.OpenCL.sampler.get.resource.id";

static StringRef ImageSizeArgMDType =   "__llvm_image_size";
static StringRef ImageFormatArgMDType = "__llvm_image_format";

static StringRef KernelsMDNodeName = "opencl.kernels";
static StringRef KernelArgMDNodeNames[] = {
  "kernel_arg_addr_space",
  "kernel_arg_access_qual",
  "kernel_arg_type",
  "kernel_arg_base_type",
  "kernel_arg_type_qual"};
static const unsigned NumKernelArgMDNodes = 5;

namespace {

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。

### Lines 60-89: Declares struct KernelArgMD
```cpp
using MDVector = SmallVector<Metadata *, 8>;
struct KernelArgMD {
  MDVector ArgVector[NumKernelArgMDNodes];
};

} // end anonymous namespace

static inline bool
IsImageType(StringRef TypeString) {
  return TypeString == "image2d_t" || TypeString == "image3d_t";
}

static inline bool
IsSamplerType(StringRef TypeString) {
  return TypeString == "sampler_t";
}

static Function *
GetFunctionFromMDNode(MDNode *Node) {
  if (!Node)
    return nullptr;

  size_t NumOps = Node->getNumOperands();
  if (NumOps != NumKernelArgMDNodes + 1)
    return nullptr;

  auto *F = mdconst::dyn_extract<Function>(Node->getOperand(0));
  if (!F)
    return nullptr;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `KernelArgMD`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`KernelArgMD`。

### Lines 90-120: Declares arg_size
```cpp
  // Validation checks.
  size_t ExpectNumArgNodeOps = F->arg_size() + 1;
  for (size_t i = 0; i < NumKernelArgMDNodes; ++i) {
    MDNode *ArgNode = dyn_cast_or_null<MDNode>(Node->getOperand(i + 1));
    if (ArgNode->getNumOperands() != ExpectNumArgNodeOps)
      return nullptr;
    if (!ArgNode->getOperand(0))
      return nullptr;

    // FIXME: It should be possible to do image lowering when some metadata
    // args missing or not in the expected order.
    MDString *StringNode = dyn_cast<MDString>(ArgNode->getOperand(0));
    if (!StringNode || StringNode->getString() != KernelArgMDNodeNames[i])
      return nullptr;
  }

  return F;
}

static StringRef
AccessQualFromMD(MDNode *KernelMDNode, unsigned ArgIdx) {
  MDNode *ArgAQNode = cast<MDNode>(KernelMDNode->getOperand(2));
  return cast<MDString>(ArgAQNode->getOperand(ArgIdx + 1))->getString();
}

static StringRef
ArgTypeFromMD(MDNode *KernelMDNode, unsigned ArgIdx) {
  MDNode *ArgTypeNode = cast<MDNode>(KernelMDNode->getOperand(3));
  return cast<MDString>(ArgTypeNode->getOperand(ArgIdx + 1))->getString();
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 121-154: Declares class R600OpenCLImageTypeLoweringPass
```cpp
static MDVector
GetArgMD(MDNode *KernelMDNode, unsigned OpIdx) {
  MDVector Res;
  for (unsigned i = 0; i < NumKernelArgMDNodes; ++i) {
    MDNode *Node = cast<MDNode>(KernelMDNode->getOperand(i + 1));
    Res.push_back(Node->getOperand(OpIdx));
  }
  return Res;
}

static void
PushArgMD(KernelArgMD &MD, const MDVector &V) {
  assert(V.size() == NumKernelArgMDNodes);
  for (unsigned i = 0; i < NumKernelArgMDNodes; ++i) {
    MD.ArgVector[i].push_back(V[i]);
  }
}

namespace {

class R600OpenCLImageTypeLoweringPass : public ModulePass {
  static char ID;

  LLVMContext *Context;
  Type *Int32Type;
  Type *ImageSizeType;
  Type *ImageFormatType;
  SmallVector<Instruction *, 4> InstsToErase;

  bool replaceImageUses(Argument &ImageArg, uint32_t ResourceID,
                        Argument &ImageSizeArg,
                        Argument &ImageFormatArg) {
    bool Modified = false;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `R600OpenCLImageTypeLoweringPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`R600OpenCLImageTypeLoweringPass`。

### Lines 155-187: Conditional logic and checks
```cpp
    for (auto &Use : ImageArg.uses()) {
      auto *Inst = dyn_cast<CallInst>(Use.getUser());
      if (!Inst) {
        continue;
      }

      Function *F = Inst->getCalledFunction();
      if (!F)
        continue;

      Value *Replacement = nullptr;
      StringRef Name = F->getName();
      if (Name.starts_with(GetImageResourceIDFunc)) {
        Replacement = ConstantInt::get(Int32Type, ResourceID);
      } else if (Name.starts_with(GetImageSizeFunc)) {
        Replacement = &ImageSizeArg;
      } else if (Name.starts_with(GetImageFormatFunc)) {
        Replacement = &ImageFormatArg;
      } else {
        continue;
      }

      Inst->replaceAllUsesWith(Replacement);
      InstsToErase.push_back(Inst);
      Modified = true;
    }

    return Modified;
  }

  bool replaceSamplerUses(Argument &SamplerArg, uint32_t ResourceID) {
    bool Modified = false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `ConstantInt::get`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`ConstantInt::get`。

### Lines 188-218: Conditional logic and checks
```cpp
    for (const auto &Use : SamplerArg.uses()) {
      auto *Inst = dyn_cast<CallInst>(Use.getUser());
      if (!Inst) {
        continue;
      }

      Function *F = Inst->getCalledFunction();
      if (!F)
        continue;

      Value *Replacement = nullptr;
      StringRef Name = F->getName();
      if (Name == GetSamplerResourceIDFunc) {
        Replacement = ConstantInt::get(Int32Type, ResourceID);
      } else {
        continue;
      }

      Inst->replaceAllUsesWith(Replacement);
      InstsToErase.push_back(Inst);
      Modified = true;
    }

    return Modified;
  }

  bool replaceImageAndSamplerUses(Function *F, MDNode *KernelMDNode) {
    uint32_t NumReadOnlyImageArgs = 0;
    uint32_t NumWriteOnlyImageArgs = 0;
    uint32_t NumSamplerArgs = 0;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `ConstantInt::get`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`ConstantInt::get`。

### Lines 219-252: Conditional logic and checks
```cpp
    bool Modified = false;
    InstsToErase.clear();
    for (auto *ArgI = F->arg_begin(); ArgI != F->arg_end(); ++ArgI) {
      Argument &Arg = *ArgI;
      StringRef Type = ArgTypeFromMD(KernelMDNode, Arg.getArgNo());

      // Handle image types.
      if (IsImageType(Type)) {
        StringRef AccessQual = AccessQualFromMD(KernelMDNode, Arg.getArgNo());
        uint32_t ResourceID;
        if (AccessQual == "read_only") {
          ResourceID = NumReadOnlyImageArgs++;
        } else if (AccessQual == "write_only") {
          ResourceID = NumWriteOnlyImageArgs++;
        } else {
          llvm_unreachable("Wrong image access qualifier.");
        }

        Argument &SizeArg = *(++ArgI);
        Argument &FormatArg = *(++ArgI);
        Modified |= replaceImageUses(Arg, ResourceID, SizeArg, FormatArg);

      // Handle sampler type.
      } else if (IsSamplerType(Type)) {
        uint32_t ResourceID = NumSamplerArgs++;
        Modified |= replaceSamplerUses(Arg, ResourceID);
      }
    }
    for (auto *Inst : InstsToErase)
      Inst->eraseFromParent();

    return Modified;
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 253-282: Defines addImplicitArgs
```cpp
  std::tuple<Function *, MDNode *>
  addImplicitArgs(Function *F, MDNode *KernelMDNode) {
    bool Modified = false;

    FunctionType *FT = F->getFunctionType();
    SmallVector<Type *, 8> ArgTypes;

    // Metadata operands for new MDNode.
    KernelArgMD NewArgMDs;
    PushArgMD(NewArgMDs, GetArgMD(KernelMDNode, 0));

    // Add implicit arguments to the signature.
    for (unsigned i = 0; i < FT->getNumParams(); ++i) {
      ArgTypes.push_back(FT->getParamType(i));
      MDVector ArgMD = GetArgMD(KernelMDNode, i + 1);
      PushArgMD(NewArgMDs, ArgMD);

      if (!IsImageType(ArgTypeFromMD(KernelMDNode, i)))
        continue;

      // Add size implicit argument.
      ArgTypes.push_back(ImageSizeType);
      ArgMD[2] = ArgMD[3] = MDString::get(*Context, ImageSizeArgMDType);
      PushArgMD(NewArgMDs, ArgMD);

      // Add format implicit argument.
      ArgTypes.push_back(ImageFormatType);
      ArgMD[2] = ArgMD[3] = MDString::get(*Context, ImageFormatArgMDType);
      PushArgMD(NewArgMDs, ArgMD);

```
**EN:** This section contains concrete logic for addImplicitArgs. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `MDString::get`.
**CN:** 本节包含与 addImplicitArgs 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`MDString::get`。

### Lines 283-316: Conditional logic and checks
```cpp
      Modified = true;
    }
    if (!Modified) {
      return std::tuple(nullptr, nullptr);
    }

    // Create function with new signature and clone the old body into it.
    auto *NewFT = FunctionType::get(FT->getReturnType(), ArgTypes, false);
    auto *NewF = Function::Create(NewFT, F->getLinkage(), F->getName());
    ValueToValueMapTy VMap;
    auto *NewFArgIt = NewF->arg_begin();
    for (auto &Arg: F->args()) {
      auto ArgName = Arg.getName();
      NewFArgIt->setName(ArgName);
      VMap[&Arg] = &(*NewFArgIt++);
      if (IsImageType(ArgTypeFromMD(KernelMDNode, Arg.getArgNo()))) {
        (NewFArgIt++)->setName(Twine("__size_") + ArgName);
        (NewFArgIt++)->setName(Twine("__format_") + ArgName);
      }
    }
    SmallVector<ReturnInst*, 8> Returns;
    CloneFunctionInto(NewF, F, VMap, CloneFunctionChangeType::LocalChangesOnly,
                      Returns);

    // Build new MDNode.
    SmallVector<Metadata *, 6> KernelMDArgs;
    KernelMDArgs.push_back(ConstantAsMetadata::get(NewF));
    for (const MDVector &MDV : NewArgMDs.ArgVector)
      KernelMDArgs.push_back(MDNode::get(*Context, MDV));
    MDNode *NewMDNode = MDNode::get(*Context, KernelMDArgs);

    return std::tuple(NewF, NewMDNode);
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::tuple`, `FunctionType::get`, `Function::Create`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::tuple`, `FunctionType::get`, `Function::Create`。

### Lines 317-350: Defines transformKernels
```cpp
  bool transformKernels(Module &M) {
    NamedMDNode *KernelsMDNode = M.getNamedMetadata(KernelsMDNodeName);
    if (!KernelsMDNode)
      return false;

    bool Modified = false;
    for (unsigned i = 0; i < KernelsMDNode->getNumOperands(); ++i) {
      MDNode *KernelMDNode = KernelsMDNode->getOperand(i);
      Function *F = GetFunctionFromMDNode(KernelMDNode);
      if (!F)
        continue;

      Function *NewF;
      MDNode *NewMDNode;
      std::tie(NewF, NewMDNode) = addImplicitArgs(F, KernelMDNode);
      if (NewF) {
        // Replace old function and metadata with new ones.
        F->eraseFromParent();
        M.getFunctionList().push_back(NewF);
        M.getOrInsertFunction(NewF->getName(), NewF->getFunctionType(),
                              NewF->getAttributes());
        KernelsMDNode->setOperand(i, NewMDNode);

        F = NewF;
        KernelMDNode = NewMDNode;
        Modified = true;
      }

      Modified |= replaceImageAndSamplerUses(F, KernelMDNode);
    }

    return Modified;
  }

```
**EN:** This section contains concrete logic for transformKernels. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::tie`.
**CN:** 本节包含与 transformKernels 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::tie`。

### Lines 351-374: Defines R600OpenCLImageTypeLoweringPass
```cpp
public:
  R600OpenCLImageTypeLoweringPass() : ModulePass(ID) {}

  bool runOnModule(Module &M) override {
    Context = &M.getContext();
    Int32Type = Type::getInt32Ty(M.getContext());
    ImageSizeType = ArrayType::get(Int32Type, 3);
    ImageFormatType = ArrayType::get(Int32Type, 2);

    return transformKernels(M);
  }

  StringRef getPassName() const override {
    return "R600 OpenCL Image Type Pass";
  }
};

} // end anonymous namespace

char R600OpenCLImageTypeLoweringPass::ID = 0;

ModulePass *llvm::createR600OpenCLImageTypeLoweringPass() {
  return new R600OpenCLImageTypeLoweringPass();
}
```
**EN:** This section contains concrete logic for R600OpenCLImageTypeLoweringPass. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `Type::getInt32Ty`, `ArrayType::get`, `llvm::createR600OpenCLImageTypeLoweringPass`.
**CN:** 本节包含与 R600OpenCLImageTypeLoweringPass 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`Type::getInt32Ty`, `ArrayType::get`, `llvm::createR600OpenCLImageTypeLoweringPass`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `KernelArgMD`, `R600OpenCLImageTypeLoweringPass`, `ConstantInt::get`, `MDString::get`, `std::tuple`, `FunctionType::get`
- **Main themes / 核心主题**: instruction semantics / 指令语义; lowering / 降低; LLVM pass integration / LLVM Pass 集成; hardware resource modeling / 硬件资源建模
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"R600.h"`
- `"llvm/ADT/SmallVector.h"`
- `"llvm/ADT/StringRef.h"`
- `"llvm/IR/Constants.h"`
- `"llvm/IR/Function.h"`
- `"llvm/IR/Instructions.h"`
- `"llvm/IR/Metadata.h"`
- `"llvm/IR/Module.h"`
- `"llvm/Pass.h"`
- `"llvm/Transforms/Utils/Cloning.h"`
