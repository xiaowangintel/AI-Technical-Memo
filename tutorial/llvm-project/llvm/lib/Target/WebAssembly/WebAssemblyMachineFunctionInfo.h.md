# WebAssemblyMachineFunctionInfo.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyMachineFunctionInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file declares WebAssembly-specific per-machine-function information.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyMachineFunctionInfo.h`，主要负责 WebAssembly 后端的后端的公共或内部声明。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
// WebAssemblyMachineFunctionInfo.h-WebAssembly machine function info-*- C++ -*-
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file declares WebAssembly-specific per-machine-function
/// information.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYMACHINEFUNCTIONINFO_H
#define LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYMACHINEFUNCTIONINFO_H

#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "llvm/CodeGen/MIRYamlMapping.h"
```
- **EN**: Pulls in direct dependencies required by this public or internal declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端的公共或内部声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 20-26

```cpp
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/MC/MCSymbolWasm.h"

namespace llvm {
class WebAssemblyTargetLowering;

namespace yaml {
```
- **EN**: Pulls in direct dependencies required by this public or internal declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Register assignment and register-class constraints matter here.
- **CN**: 这一段引入该后端的公共或内部声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 27-52

```cpp
struct WebAssemblyFunctionInfo;
}

/// This class is derived from MachineFunctionInfo and contains private
/// WebAssembly-specific information for each MachineFunction.
class WebAssemblyFunctionInfo final : public MachineFunctionInfo {
  std::vector<MVT> Params;
  std::vector<MVT> Results;
  std::vector<MVT> Locals;

  /// A mapping from CodeGen vreg index to WebAssembly register number.
  std::vector<unsigned> WARegs;

  /// A mapping from CodeGen vreg index to a boolean value indicating whether
  /// the given register is considered to be "stackified", meaning it has been
  /// determined or made to meet the stack requirements:
  ///   - single use (per path)
  ///   - single def (per path)
  ///   - defined and used in LIFO order with other stack registers
  BitVector VRegStackified;

  // A virtual register holding the pointer to the vararg buffer for vararg
  // functions. It is created and set in TLI::LowerFormalArguments and read by
  // TLI::LowerVASTART
  unsigned VarargVreg = -1U;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This class is derived from MachineFunctionInfo and contains private". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This class is derived from MachineFunctionInfo and contains private”。 该区间与栈帧布局或栈访问相关。

### Lines 53-78

```cpp
  // A virtual register holding the base pointer for functions that have
  // overaligned values on the user stack.
  unsigned BasePtrVreg = -1U;
  // A virtual register holding the frame base. This is either FP or SP
  // after it has been replaced by a vreg
  unsigned FrameBaseVreg = -1U;
  // The local holding the frame base. This is either FP or SP
  // after WebAssemblyExplicitLocals
  unsigned FrameBaseLocal = -1U;

  // Function properties.
  bool CFGStackified = false;

public:
  explicit WebAssemblyFunctionInfo(const Function &F,
                                   const TargetSubtargetInfo *STI) {}
  ~WebAssemblyFunctionInfo() override;

  MachineFunctionInfo *
  clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
        const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
      const override;

  void initializeBaseYamlFields(MachineFunction &MF,
                                const yaml::WebAssemblyFunctionInfo &YamlMFI);
```
- **EN**: Implements helper routine(s) `WebAssemblyFunctionInfo`, `clone`, `initializeBaseYamlFields` for this portion of the WebAssembly backend public or internal declarations for the backend. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分后端的公共或内部声明所需的辅助例程 `WebAssemblyFunctionInfo`, `clone`, `initializeBaseYamlFields`。 该区间与栈帧布局或栈访问相关。

### Lines 79-104

```cpp
  void addParam(MVT VT) { Params.push_back(VT); }
  const std::vector<MVT> &getParams() const { return Params; }

  void addResult(MVT VT) { Results.push_back(VT); }
  const std::vector<MVT> &getResults() const { return Results; }

  void clearParamsAndResults() {
    Params.clear();
    Results.clear();
  }

  void setNumLocals(size_t NumLocals) { Locals.resize(NumLocals, MVT::i32); }
  void setLocal(size_t i, MVT VT) { Locals[i] = VT; }
  void addLocal(MVT VT) { Locals.push_back(VT); }
  const std::vector<MVT> &getLocals() const { return Locals; }

  unsigned getVarargBufferVreg() const {
    assert(VarargVreg != -1U && "Vararg vreg hasn't been set");
    return VarargVreg;
  }
  void setVarargBufferVreg(unsigned Reg) { VarargVreg = Reg; }

  unsigned getBasePointerVreg() const {
    assert(BasePtrVreg != -1U && "Base ptr vreg hasn't been set");
    return BasePtrVreg;
  }
```
- **EN**: Implements helper routine(s) `addParam`, `push_back`, `getParams` for this portion of the WebAssembly backend public or internal declarations for the backend.
- **CN**: 这里实现了 WebAssembly 后端该部分后端的公共或内部声明所需的辅助例程 `addParam`, `push_back`, `getParams`。

### Lines 105-130

```cpp
  void setFrameBaseVreg(unsigned Reg) { FrameBaseVreg = Reg; }
  unsigned getFrameBaseVreg() const {
    assert(FrameBaseVreg != -1U && "Frame base vreg hasn't been set");
    return FrameBaseVreg;
  }
  void clearFrameBaseVreg() { FrameBaseVreg = -1U; }
  // Return true if the frame base physreg has been replaced by a virtual reg.
  bool isFrameBaseVirtual() const { return FrameBaseVreg != -1U; }
  void setFrameBaseLocal(unsigned Local) { FrameBaseLocal = Local; }
  unsigned getFrameBaseLocal() const {
    assert(FrameBaseLocal != -1U && "Frame base local hasn't been set");
    return FrameBaseLocal;
  }
  void setBasePointerVreg(unsigned Reg) { BasePtrVreg = Reg; }

  void stackifyVReg(MachineRegisterInfo &MRI, Register VReg) {
    assert(MRI.getUniqueVRegDef(VReg));
    auto I = VReg.virtRegIndex();
    if (I >= VRegStackified.size())
      VRegStackified.resize(I + 1);
    VRegStackified.set(I);
  }
  void unstackifyVReg(Register VReg) {
    auto I = VReg.virtRegIndex();
    if (I < VRegStackified.size())
      VRegStackified.reset(I);
```
- **EN**: Implements helper routine(s) `setFrameBaseVreg`, `getFrameBaseVreg`, `clearFrameBaseVreg` for this portion of the WebAssembly backend public or internal declarations for the backend. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端的公共或内部声明所需的辅助例程 `setFrameBaseVreg`, `getFrameBaseVreg`, `clearFrameBaseVreg`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 131-156

```cpp
  }
  bool isVRegStackified(Register VReg) const {
    auto I = VReg.virtRegIndex();
    if (I >= VRegStackified.size())
      return false;
    return VRegStackified.test(I);
  }

  void initWARegs(MachineRegisterInfo &MRI);
  void setWAReg(Register VReg, unsigned WAReg) {
    assert(WAReg != WebAssembly::UnusedReg);
    auto I = VReg.virtRegIndex();
    assert(I < WARegs.size());
    WARegs[I] = WAReg;
  }
  unsigned getWAReg(Register VReg) const {
    auto I = VReg.virtRegIndex();
    assert(I < WARegs.size());
    return WARegs[I];
  }

  bool isCFGStackified() const { return CFGStackified; }
  void setCFGStackified(bool Value = true) { CFGStackified = Value; }
};

void computeLegalValueVTs(const WebAssemblyTargetLowering &TLI,
```
- **EN**: Implements helper routine(s) `isVRegStackified`, `virtRegIndex`, `size` for this portion of the WebAssembly backend public or internal declarations for the backend. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端的公共或内部声明所需的辅助例程 `isVRegStackified`, `virtRegIndex`, `size`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 157-175

```cpp
                          LLVMContext &Ctx, const DataLayout &DL, Type *Ty,
                          SmallVectorImpl<MVT> &ValueVTs);

void computeLegalValueVTs(const Function &F, const TargetMachine &TM, Type *Ty,
                          SmallVectorImpl<MVT> &ValueVTs);

// Compute the signature for a given FunctionType (Ty). Note that it's not the
// signature for ContextFunc (ContextFunc is just used to get varous context)
void computeSignatureVTs(const FunctionType *Ty, const Function *TargetFunc,
                         const Function &ContextFunc, const TargetMachine &TM,
                         SmallVectorImpl<MVT> &Params,
                         SmallVectorImpl<MVT> &Results);

void valTypesFromMVTs(ArrayRef<MVT> In, SmallVectorImpl<wasm::ValType> &Out);

wasm::WasmSignature *signatureFromMVTs(MCContext &Ctx,
                                       const SmallVectorImpl<MVT> &Results,
                                       const SmallVectorImpl<MVT> &Params);
```
- **EN**: Declares function entry points including `computeLegalValueVTs`, `FunctionType`, `ContextFunc` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `computeLegalValueVTs`, `FunctionType`, `ContextFunc`。

### Lines 176-201

```cpp
namespace yaml {

using BBNumberMap = DenseMap<int, int>;

struct WebAssemblyFunctionInfo final : public yaml::MachineFunctionInfo {
  std::vector<FlowStringValue> Params;
  std::vector<FlowStringValue> Results;
  bool CFGStackified = false;

  WebAssemblyFunctionInfo() = default;
  WebAssemblyFunctionInfo(const llvm::MachineFunction &MF,
                          const llvm::WebAssemblyFunctionInfo &MFI);

  void mappingImpl(yaml::IO &YamlIO) override;
  ~WebAssemblyFunctionInfo() override = default;
};

template <> struct MappingTraits<WebAssemblyFunctionInfo> {
  static void mapping(IO &YamlIO, WebAssemblyFunctionInfo &MFI) {
    YamlIO.mapOptional("params", MFI.Params, std::vector<FlowStringValue>());
    YamlIO.mapOptional("results", MFI.Results, std::vector<FlowStringValue>());
    YamlIO.mapOptional("isCFGStackified", MFI.CFGStackified, false);
  }
};

} // end namespace yaml
```
- **EN**: Opens or re-enters namespace scope so backend symbols stay grouped under LLVM naming conventions. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里进入或重新进入命名空间作用域，使后端符号按照 LLVM 的命名约定组织在一起。 该区间与栈帧布局或栈访问相关。

### Lines 202-205

```cpp

} // end namespace llvm

#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Public or internal declarations for the backend / 后端的公共或内部声明
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Target machine configuration / 目标机器配置
- Object format integration / 目标文件格式集成
- Declarative TableGen records / 声明式 TableGen 记录

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `llvm/CodeGen/MIRYamlMapping.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/MC/MCSymbolWasm.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
