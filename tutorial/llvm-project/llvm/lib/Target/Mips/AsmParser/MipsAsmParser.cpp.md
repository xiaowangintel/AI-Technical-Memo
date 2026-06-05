# MipsAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/AsmParser/MipsAsmParser.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsAsmParser` for the Mips backend, focusing on assembly parsing and operand/instruction decoding.
- 用途 (CN): 实现 Mips 后端中的 `MipsAsmParser`，重点处理汇编解析以及操作数/指令解码。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MipsAsmParser.cpp - Parse Mips assembly to MCInst instructions ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-23
```cpp
#include "MCTargetDesc/MipsABIFlagsSection.h"
#include "MCTargetDesc/MipsABIInfo.h"
#include "MCTargetDesc/MipsBaseInfo.h"
#include "MCTargetDesc/MipsMCAsmInfo.h"
#include "MCTargetDesc/MipsMCTargetDesc.h"
#include "MCTargetDesc/MipsTargetStreamer.h"
#include "TargetInfo/MipsTargetInfo.h"
#include "llvm/ADT/APFloat.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/ADT/Twine.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 24-38
```cpp
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCObjectFileInfo.h"
#include "llvm/MC/MCParser/AsmLexer.h"
#include "llvm/MC/MCParser/MCAsmParser.h"
#include "llvm/MC/MCParser/MCAsmParserExtension.h"
#include "llvm/MC/MCParser/MCAsmParserUtils.h"
#include "llvm/MC/MCParser/MCParsedAsmOperand.h"
#include "llvm/MC/MCParser/MCTargetAsmParser.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCSymbolELF.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 39-53
```cpp
#include "llvm/MC/MCValue.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Alignment.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Support/SMLoc.h"
#include "llvm/Support/SourceMgr.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/TargetParser/SubtargetFeature.h"
#include "llvm/TargetParser/Triple.h"
#include <algorithm>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 54-58
```cpp
#include <cassert>
#include <cstdint>
#include <memory>
#include <string>
#include <utility>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 60-60
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 62-62
```cpp
#define DEBUG_TYPE "mips-asm-parser"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 64-64
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 66-66
```cpp
class MCInstrInfo;
```
- EN: Declares `MCInstrInfo`, packaging target-specific state and APIs around `MipsAsmParser`.
- CN: 这里声明 `MCInstrInfo`，把与 `MipsAsmParser` 相关的目标特定状态和 API 组织在一起。

### Lines 68-68
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 70-71
```cpp
extern cl::opt<bool> EmitJalrReloc;
extern cl::opt<bool> NoZeroDivCheck;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 73-73
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 75-77
```cpp
class MipsAssemblerOptions {
public:
  MipsAssemblerOptions(const FeatureBitset &Features_) : Features(Features_) {}
```
- EN: Declares `MipsAssemblerOptions`, packaging target-specific state and APIs around `MipsAsmParser`.
- CN: 这里声明 `MipsAssemblerOptions`，把与 `MipsAsmParser` 相关的目标特定状态和 API 组织在一起。

### Lines 79-84
```cpp
  MipsAssemblerOptions(const MipsAssemblerOptions *Opts) {
    ATReg = Opts->getATRegIndex();
    Reorder = Opts->isReorder();
    Macro = Opts->isMacro();
    Features = Opts->getFeatures();
  }
```
- EN: Implements `MipsAssemblerOptions`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAssemblerOptions`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 86-89
```cpp
  unsigned getATRegIndex() const { return ATReg; }
  bool setATRegIndex(unsigned Reg) {
    if (Reg > 31)
      return false;
```
- EN: Implements `getATRegIndex`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getATRegIndex`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 91-93
```cpp
    ATReg = Reg;
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 95-97
```cpp
  bool isReorder() const { return Reorder; }
  void setReorder() { Reorder = true; }
  void setNoReorder() { Reorder = false; }
```
- EN: Implements `isReorder`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isReorder`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 99-101
```cpp
  bool isMacro() const { return Macro; }
  void setMacro() { Macro = true; }
  void setNoMacro() { Macro = false; }
```
- EN: Implements `isMacro`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMacro`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 103-104
```cpp
  const FeatureBitset &getFeatures() const { return Features; }
  void setFeatures(const FeatureBitset &Features_) { Features = Features_; }
```
- EN: Implements `getFeatures`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFeatures`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 106-111
```cpp
  // Set of features that are either architecture features or referenced
  // by them (e.g.: FeatureNaN2008 implied by FeatureMips32r6).
  // The full table can be found in MipsGenSubtargetInfo.inc (MipsFeatureKV[]).
  // The reason we need this mask is explained in the selectArch function.
  // FIXME: Ideally we would like TableGen to generate this information.
  static const FeatureBitset AllArchRelatedMask;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 113-118
```cpp
private:
  unsigned ATReg = 1;
  bool Reorder = true;
  bool Macro = true;
  FeatureBitset Features;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 120-120
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 122-132
```cpp
const FeatureBitset MipsAssemblerOptions::AllArchRelatedMask = {
    Mips::FeatureMips1, Mips::FeatureMips2, Mips::FeatureMips3,
    Mips::FeatureMips3_32, Mips::FeatureMips3_32r2, Mips::FeatureMips4,
    Mips::FeatureMips4_32, Mips::FeatureMips4_32r2, Mips::FeatureMips5,
    Mips::FeatureMips5_32r2, Mips::FeatureMips32, Mips::FeatureMips32r2,
    Mips::FeatureMips32r3, Mips::FeatureMips32r5, Mips::FeatureMips32r6,
    Mips::FeatureMips64, Mips::FeatureMips64r2, Mips::FeatureMips64r3,
    Mips::FeatureMips64r5, Mips::FeatureMips64r6, Mips::FeatureCnMips,
    Mips::FeatureCnMipsP, Mips::FeatureFP64Bit, Mips::FeatureGP64Bit,
    Mips::FeatureNaN2008
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 134-134
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 136-142
```cpp
class MipsAsmParser : public MCTargetAsmParser {
  MipsTargetStreamer &getTargetStreamer() {
    assert(getParser().getStreamer().getTargetStreamer() &&
           "do not have a target streamer");
    MCTargetStreamer &TS = *getParser().getStreamer().getTargetStreamer();
    return static_cast<MipsTargetStreamer &>(TS);
  }
```
- EN: Declares `MipsAsmParser`, packaging target-specific state and APIs around `MipsAsmParser`.
- CN: 这里声明 `MipsAsmParser`，把与 `MipsAsmParser` 相关的目标特定状态和 API 组织在一起。

### Lines 144-158
```cpp
  MipsABIInfo ABI;
  SmallVector<std::unique_ptr<MipsAssemblerOptions>, 2> AssemblerOptions;
  MCSymbol *CurrentFn; // Pointer to the function being parsed. It may be a
                       // nullptr, which indicates that no function is currently
                       // selected. This usually happens after an '.end func'
                       // directive.
  bool IsLittleEndian;
  bool IsPicEnabled;
  bool IsCpRestoreSet;
  bool CurForbiddenSlotAttr;
  int CpRestoreOffset;
  MCRegister GPReg;
  unsigned CpSaveLocation;
  /// If true, then CpSaveLocation is a register, otherwise it's an offset.
  bool     CpSaveLocationIsRegister;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 160-161
```cpp
  // Map of register aliases created via the .set directive.
  StringMap<AsmToken> RegisterSets;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 163-165
```cpp
  // Print a warning along with its fix-it message at the given range.
  void printWarningWithFixIt(const Twine &Msg, const Twine &FixMsg,
                             SMRange Range, bool ShowColors = true);
```
- EN: Declares `printWarningWithFixIt`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `printWarningWithFixIt`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 167-167
```cpp
  void ConvertXWPOperands(MCInst &Inst, const OperandVector &Operands);
```
- EN: Declares `ConvertXWPOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ConvertXWPOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 169-170
```cpp
#define GET_ASSEMBLER_HEADER
#include "MipsGenAsmMatcher.inc"
```
- EN: Connects this file to TableGen-generated declarations so target-specific enums and helper tables become available to C++ code.
- CN: 这里把文件连接到 TableGen 生成的声明，使目标相关枚举和辅助表能够在 C++ 代码中使用。

### Lines 172-175
```cpp
  unsigned
  checkEarlyTargetMatchPredicate(MCInst &Inst,
                                 const OperandVector &Operands) override;
  unsigned checkTargetMatchPredicate(MCInst &Inst) override;
```
- EN: Declares `checkEarlyTargetMatchPredicate`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `checkEarlyTargetMatchPredicate`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 177-180
```cpp
  bool matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
                               OperandVector &Operands, MCStreamer &Out,
                               uint64_t &ErrorInfo,
                               bool MatchingInlineAsm) override;
```
- EN: Declares `matchAndEmitInstruction`, a target-specific routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `matchAndEmitInstruction`，它是一个围绕MC streamer 交互展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 182-185
```cpp
  /// Parse a register as used in CFI directives
  bool parseRegister(MCRegister &Reg, SMLoc &StartLoc, SMLoc &EndLoc) override;
  ParseStatus tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
                               SMLoc &EndLoc) override;
```
- EN: Declares `parseRegister`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `parseRegister`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 187-187
```cpp
  bool parseParenSuffix(StringRef Name, OperandVector &Operands);
```
- EN: Declares `parseParenSuffix`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `parseParenSuffix`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 189-189
```cpp
  bool parseBracketSuffix(StringRef Name, OperandVector &Operands);
```
- EN: Declares `parseBracketSuffix`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `parseBracketSuffix`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 191-191
```cpp
  bool mnemonicIsValid(StringRef Mnemonic, unsigned VariantID);
```
- EN: Declares `mnemonicIsValid`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `mnemonicIsValid`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 193-194
```cpp
  bool parseInstruction(ParseInstructionInfo &Info, StringRef Name,
                        SMLoc NameLoc, OperandVector &Operands) override;
```
- EN: Declares `parseInstruction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `parseInstruction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 196-196
```cpp
  bool ParseDirective(AsmToken DirectiveID) override;
```
- EN: Declares `ParseDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ParseDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 198-209
```cpp
  ParseStatus parseMemOperand(OperandVector &Operands);
  ParseStatus matchAnyRegisterNameWithoutDollar(OperandVector &Operands,
                                                StringRef Identifier, SMLoc S);
  ParseStatus matchAnyRegisterWithoutDollar(OperandVector &Operands,
                                            const AsmToken &Token, SMLoc S);
  ParseStatus matchAnyRegisterWithoutDollar(OperandVector &Operands, SMLoc S);
  ParseStatus parseAnyRegister(OperandVector &Operands);
  ParseStatus parseImm(OperandVector &Operands);
  ParseStatus parseJumpTarget(OperandVector &Operands);
  ParseStatus parseInvNum(OperandVector &Operands);
  ParseStatus parseRegisterList(OperandVector &Operands);
  const MCExpr *parseRelocExpr();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 211-211
```cpp
  bool searchSymbolAlias(OperandVector &Operands);
```
- EN: Declares `searchSymbolAlias`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `searchSymbolAlias`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 213-213
```cpp
  bool parseOperand(OperandVector &, StringRef Mnemonic);
```
- EN: Declares `parseOperand`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `parseOperand`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 215-219
```cpp
  enum MacroExpanderResultTy {
    MER_NotAMacro,
    MER_Success,
    MER_Fail,
  };
```
- EN: Defines enumeration `MacroExpanderResultTy` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `MacroExpanderResultTy`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 221-224
```cpp
  // Expands assembly pseudo instructions.
  MacroExpanderResultTy tryExpandInstruction(MCInst &Inst, SMLoc IDLoc,
                                             MCStreamer &Out,
                                             const MCSubtargetInfo *STI);
```
- EN: Declares `tryExpandInstruction`, a target-specific routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `tryExpandInstruction`，它是一个围绕MC streamer 交互展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 226-227
```cpp
  bool expandJalWithRegs(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                         const MCSubtargetInfo *STI);
```
- EN: Declares `expandJalWithRegs`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandJalWithRegs`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 229-231
```cpp
  bool loadImmediate(int64_t ImmValue, MCRegister DstReg, MCRegister SrcReg,
                     bool Is32BitImm, bool IsAddress, SMLoc IDLoc,
                     MCStreamer &Out, const MCSubtargetInfo *STI);
```
- EN: Declares `loadImmediate`, a target-specific routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `loadImmediate`，它是一个围绕MC streamer 交互展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 233-235
```cpp
  bool loadAndAddSymbolAddress(const MCExpr *SymExpr, MCRegister DstReg,
                               MCRegister SrcReg, bool Is32BitSym, SMLoc IDLoc,
                               MCStreamer &Out, const MCSubtargetInfo *STI);
```
- EN: Declares `loadAndAddSymbolAddress`, a target-specific routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `loadAndAddSymbolAddress`，它是一个围绕MC streamer 交互展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 237-237
```cpp
  bool emitPartialAddress(MipsTargetStreamer &TOut, SMLoc IDLoc, MCSymbol *Sym);
```
- EN: Declares `emitPartialAddress`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitPartialAddress`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 239-240
```cpp
  bool expandLoadImm(MCInst &Inst, bool Is32BitImm, SMLoc IDLoc,
                     MCStreamer &Out, const MCSubtargetInfo *STI);
```
- EN: Declares `expandLoadImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandLoadImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 242-249
```cpp
  bool expandLoadSingleImmToGPR(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                const MCSubtargetInfo *STI);
  bool expandLoadSingleImmToFPR(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                const MCSubtargetInfo *STI);
  bool expandLoadDoubleImmToGPR(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                const MCSubtargetInfo *STI);
  bool expandLoadDoubleImmToFPR(MCInst &Inst, bool Is64FPU, SMLoc IDLoc,
                                MCStreamer &Out, const MCSubtargetInfo *STI);
```
- EN: Declares `expandLoadSingleImmToGPR`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandLoadSingleImmToGPR`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 251-254
```cpp
  bool expandLoadAddress(MCRegister DstReg, MCRegister BaseReg,
                         const MCOperand &Offset, bool Is32BitAddress,
                         SMLoc IDLoc, MCStreamer &Out,
                         const MCSubtargetInfo *STI);
```
- EN: Declares `expandLoadAddress`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandLoadAddress`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 256-257
```cpp
  bool expandUncondBranchMMPseudo(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                  const MCSubtargetInfo *STI);
```
- EN: Declares `expandUncondBranchMMPseudo`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandUncondBranchMMPseudo`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 259-262
```cpp
  void expandMem16Inst(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                       const MCSubtargetInfo *STI, bool IsLoad);
  void expandMem9Inst(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                      const MCSubtargetInfo *STI, bool IsLoad);
```
- EN: Declares `expandMem16Inst`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandMem16Inst`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 264-265
```cpp
  bool expandLoadStoreMultiple(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                               const MCSubtargetInfo *STI);
```
- EN: Declares `expandLoadStoreMultiple`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandLoadStoreMultiple`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 267-268
```cpp
  bool expandAliasImmediate(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                            const MCSubtargetInfo *STI);
```
- EN: Declares `expandAliasImmediate`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandAliasImmediate`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 270-271
```cpp
  bool expandBranchImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                       const MCSubtargetInfo *STI);
```
- EN: Declares `expandBranchImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandBranchImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 273-274
```cpp
  bool expandCondBranches(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                          const MCSubtargetInfo *STI);
```
- EN: Declares `expandCondBranches`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandCondBranches`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 276-278
```cpp
  bool expandDivRem(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                    const MCSubtargetInfo *STI, const bool IsMips64,
                    const bool Signed);
```
- EN: Declares `expandDivRem`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandDivRem`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 280-281
```cpp
  bool expandTrunc(MCInst &Inst, bool IsDouble, bool Is64FPU, SMLoc IDLoc,
                   MCStreamer &Out, const MCSubtargetInfo *STI);
```
- EN: Declares `expandTrunc`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandTrunc`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 283-284
```cpp
  bool expandUlh(MCInst &Inst, bool Signed, SMLoc IDLoc, MCStreamer &Out,
                 const MCSubtargetInfo *STI);
```
- EN: Declares `expandUlh`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandUlh`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 286-287
```cpp
  bool expandUsh(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                 const MCSubtargetInfo *STI);
```
- EN: Declares `expandUsh`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandUsh`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 289-290
```cpp
  bool expandUxw(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                 const MCSubtargetInfo *STI);
```
- EN: Declares `expandUxw`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandUxw`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 292-293
```cpp
  bool expandSge(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                 const MCSubtargetInfo *STI);
```
- EN: Declares `expandSge`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandSge`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 295-296
```cpp
  bool expandSgeImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                    const MCSubtargetInfo *STI);
```
- EN: Declares `expandSgeImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandSgeImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 298-299
```cpp
  bool expandSgtImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                    const MCSubtargetInfo *STI);
```
- EN: Declares `expandSgtImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandSgtImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 301-302
```cpp
  bool expandSle(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                 const MCSubtargetInfo *STI);
```
- EN: Declares `expandSle`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandSle`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 304-305
```cpp
  bool expandSleImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                    const MCSubtargetInfo *STI);
```
- EN: Declares `expandSleImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandSleImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 307-314
```cpp
  bool expandRotation(MCInst &Inst, SMLoc IDLoc,
                      MCStreamer &Out, const MCSubtargetInfo *STI);
  bool expandRotationImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                         const MCSubtargetInfo *STI);
  bool expandDRotation(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                       const MCSubtargetInfo *STI);
  bool expandDRotationImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                          const MCSubtargetInfo *STI);
```
- EN: Declares `expandRotation`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandRotation`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 316-317
```cpp
  bool expandAbs(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                 const MCSubtargetInfo *STI);
```
- EN: Declares `expandAbs`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandAbs`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 319-320
```cpp
  bool expandMulImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                    const MCSubtargetInfo *STI);
```
- EN: Declares `expandMulImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandMulImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 322-323
```cpp
  bool expandMulO(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                  const MCSubtargetInfo *STI);
```
- EN: Declares `expandMulO`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandMulO`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 325-326
```cpp
  bool expandMulOU(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                   const MCSubtargetInfo *STI);
```
- EN: Declares `expandMulOU`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandMulOU`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 328-329
```cpp
  bool expandDMULMacro(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                       const MCSubtargetInfo *STI);
```
- EN: Declares `expandDMULMacro`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandDMULMacro`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 331-332
```cpp
  bool expandLoadStoreDMacro(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                             const MCSubtargetInfo *STI, bool IsLoad);
```
- EN: Declares `expandLoadStoreDMacro`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandLoadStoreDMacro`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 334-335
```cpp
  bool expandStoreDM1Macro(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                           const MCSubtargetInfo *STI);
```
- EN: Declares `expandStoreDM1Macro`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandStoreDM1Macro`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 337-338
```cpp
  bool expandSeq(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                 const MCSubtargetInfo *STI);
```
- EN: Declares `expandSeq`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandSeq`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 340-341
```cpp
  bool expandSeqI(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                  const MCSubtargetInfo *STI);
```
- EN: Declares `expandSeqI`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandSeqI`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 343-344
```cpp
  bool expandSne(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                 const MCSubtargetInfo *STI);
```
- EN: Declares `expandSne`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandSne`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 346-347
```cpp
  bool expandSneI(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                  const MCSubtargetInfo *STI);
```
- EN: Declares `expandSneI`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandSneI`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 349-350
```cpp
  bool expandMXTRAlias(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                       const MCSubtargetInfo *STI);
```
- EN: Declares `expandMXTRAlias`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandMXTRAlias`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 352-353
```cpp
  bool expandSaaAddr(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                     const MCSubtargetInfo *STI);
```
- EN: Declares `expandSaaAddr`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandSaaAddr`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 355-356
```cpp
  bool reportParseError(const Twine &ErrorMsg);
  bool reportParseError(SMLoc Loc, const Twine &ErrorMsg);
```
- EN: Declares `reportParseError`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `reportParseError`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 358-372
```cpp
  bool parseSetMips0Directive();
  bool parseSetArchDirective();
  bool parseSetFeature(uint64_t Feature);
  bool isPicAndNotNxxAbi(); // Used by .cpload, .cprestore, and .cpsetup.
  bool parseDirectiveCpAdd(SMLoc Loc);
  bool parseDirectiveCpLoad(SMLoc Loc);
  bool parseDirectiveCpLocal(SMLoc Loc);
  bool parseDirectiveCpRestore(SMLoc Loc);
  bool parseDirectiveCPSetup();
  bool parseDirectiveCPReturn();
  bool parseDirectiveNaN();
  bool parseDirectiveSet();
  bool parseDirectiveOption();
  bool parseInsnDirective();
  bool parseRSectionDirective(StringRef Section);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 373-373
```cpp
  bool parseSSectionDirective(StringRef Section, unsigned Type);
```
- EN: Declares `parseSSectionDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `parseSSectionDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 375-389
```cpp
  bool parseSetAtDirective();
  bool parseSetNoAtDirective();
  bool parseSetMacroDirective();
  bool parseSetNoMacroDirective();
  bool parseSetMsaDirective();
  bool parseSetNoMsaDirective();
  bool parseSetNoDspDirective();
  bool parseSetNoMips3DDirective();
  bool parseSetReorderDirective();
  bool parseSetNoReorderDirective();
  bool parseSetMips16Directive();
  bool parseSetNoMips16Directive();
  bool parseSetFpDirective();
  bool parseSetOddSPRegDirective();
  bool parseSetNoOddSPRegDirective();
```
- EN: Declares `parseSetAtDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `parseSetAtDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 390-398
```cpp
  bool parseSetPopDirective();
  bool parseSetPushDirective();
  bool parseSetSoftFloatDirective();
  bool parseSetHardFloatDirective();
  bool parseSetMtDirective();
  bool parseSetNoMtDirective();
  bool parseSetNoCRCDirective();
  bool parseSetNoVirtDirective();
  bool parseSetNoGINVDirective();
```
- EN: Declares `parseSetPopDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `parseSetPopDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 400-400
```cpp
  bool parseSetAssignment();
```
- EN: Declares `parseSetAssignment`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `parseSetAssignment`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 402-411
```cpp
  bool parseDirectiveGpWord();
  bool parseDirectiveGpDWord();
  bool parseDirectiveDtpRelWord();
  bool parseDirectiveDtpRelDWord();
  bool parseDirectiveTpRelWord();
  bool parseDirectiveTpRelDWord();
  bool parseDirectiveModule();
  bool parseDirectiveModuleFP();
  bool parseFpABIValue(MipsABIFlagsSection::FpABIKind &FpABI,
                       StringRef Directive);
```
- EN: Declares `parseDirectiveGpWord`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `parseDirectiveGpWord`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 413-413
```cpp
  bool parseInternalDirectiveReallowModule();
```
- EN: Declares `parseInternalDirectiveReallowModule`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `parseInternalDirectiveReallowModule`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 415-415
```cpp
  bool eatComma(StringRef ErrorStr);
```
- EN: Declares `eatComma`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `eatComma`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 417-417
```cpp
  int matchCPURegisterName(StringRef Symbol);
```
- EN: Declares `matchCPURegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `matchCPURegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 419-419
```cpp
  int matchHWRegsRegisterName(StringRef Symbol);
```
- EN: Declares `matchHWRegsRegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `matchHWRegsRegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 421-421
```cpp
  int matchFPURegisterName(StringRef Name);
```
- EN: Declares `matchFPURegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `matchFPURegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 423-423
```cpp
  int matchFCCRegisterName(StringRef Name);
```
- EN: Declares `matchFCCRegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `matchFCCRegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 425-425
```cpp
  int matchACRegisterName(StringRef Name);
```
- EN: Declares `matchACRegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `matchACRegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 427-427
```cpp
  int matchMSA128RegisterName(StringRef Name);
```
- EN: Declares `matchMSA128RegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `matchMSA128RegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 429-429
```cpp
  int matchMSA128CtrlRegisterName(StringRef Name);
```
- EN: Declares `matchMSA128CtrlRegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `matchMSA128CtrlRegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 431-431
```cpp
  MCRegister getReg(int RC, int RegNo);
```
- EN: Declares `getReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 433-436
```cpp
  /// Returns the internal register number for the current AT. Also checks if
  /// the current AT is unavailable (set to $0) and gives an error if it is.
  /// This should be used in pseudo-instruction expansions which need AT.
  MCRegister getATReg(SMLoc Loc);
```
- EN: Declares `getATReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getATReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 438-438
```cpp
  bool canUseATReg();
```
- EN: Declares `canUseATReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `canUseATReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 440-441
```cpp
  bool processInstruction(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                          const MCSubtargetInfo *STI);
```
- EN: Declares `processInstruction`, a target-specific routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `processInstruction`，它是一个围绕MC streamer 交互展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 443-446
```cpp
  // Helper function that checks if the value of a vector index is within the
  // boundaries of accepted values for each RegisterKind
  // Example: INSERT.B $w0[n], $1 => 16 > n >= 0
  bool validateMSAIndex(int Val, int RegKind);
```
- EN: Declares `validateMSAIndex`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `validateMSAIndex`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 448-462
```cpp
  // Selects a new architecture by updating the FeatureBits with the necessary
  // info including implied dependencies.
  // Internally, it clears all the feature bits related to *any* architecture
  // and selects the new one using the ToggleFeature functionality of the
  // MCSubtargetInfo object that handles implied dependencies. The reason we
  // clear all the arch related bits manually is because ToggleFeature only
  // clears the features that imply the feature being cleared and not the
  // features implied by the feature being cleared. This is easier to see
  // with an example:
  //  --------------------------------------------------
  // | Feature         | Implies                        |
  // | -------------------------------------------------|
  // | FeatureMips1    | None                           |
  // | FeatureMips2    | FeatureMips1                   |
  // | FeatureMips3    | FeatureMips2 | FeatureMipsGP64 |
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 463-477
```cpp
  // | FeatureMips4    | FeatureMips3                   |
  // | ...             |                                |
  //  --------------------------------------------------
  //
  // Setting Mips3 is equivalent to set: (FeatureMips3 | FeatureMips2 |
  // FeatureMipsGP64 | FeatureMips1)
  // Clearing Mips3 is equivalent to clear (FeatureMips3 | FeatureMips4).
  void selectArch(StringRef ArchFeature) {
    MCSubtargetInfo &STI = copySTI();
    FeatureBitset FeatureBits = STI.getFeatureBits();
    FeatureBits &= ~MipsAssemblerOptions::AllArchRelatedMask;
    STI.setFeatureBits(FeatureBits);
    setAvailableFeatures(
        ComputeAvailableFeatures(STI.ToggleFeature(ArchFeature)));
    AssemblerOptions.back()->setFeatures(STI.getFeatureBits());
```
- EN: Implements `selectArch`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `selectArch`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 478-478
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 480-487
```cpp
  void setFeatureBits(uint64_t Feature, StringRef FeatureString) {
    if (!(getSTI().hasFeature(Feature))) {
      MCSubtargetInfo &STI = copySTI();
      setAvailableFeatures(
          ComputeAvailableFeatures(STI.ToggleFeature(FeatureString)));
      AssemblerOptions.back()->setFeatures(STI.getFeatureBits());
    }
  }
```
- EN: Implements `setFeatureBits`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setFeatureBits`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 489-496
```cpp
  void clearFeatureBits(uint64_t Feature, StringRef FeatureString) {
    if (getSTI().hasFeature(Feature)) {
      MCSubtargetInfo &STI = copySTI();
      setAvailableFeatures(
          ComputeAvailableFeatures(STI.ToggleFeature(FeatureString)));
      AssemblerOptions.back()->setFeatures(STI.getFeatureBits());
    }
  }
```
- EN: Implements `clearFeatureBits`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `clearFeatureBits`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 498-501
```cpp
  void setModuleFeatureBits(uint64_t Feature, StringRef FeatureString) {
    setFeatureBits(Feature, FeatureString);
    AssemblerOptions.front()->setFeatures(getSTI().getFeatureBits());
  }
```
- EN: Implements `setModuleFeatureBits`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setModuleFeatureBits`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 503-506
```cpp
  void clearModuleFeatureBits(uint64_t Feature, StringRef FeatureString) {
    clearFeatureBits(Feature, FeatureString);
    AssemblerOptions.front()->setFeatures(getSTI().getFeatureBits());
  }
```
- EN: Implements `clearModuleFeatureBits`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `clearModuleFeatureBits`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 508-522
```cpp
public:
  enum MipsMatchResultTy {
    Match_RequiresDifferentSrcAndDst = FIRST_TARGET_MATCH_RESULT_TY,
    Match_RequiresDifferentOperands,
    Match_RequiresNoZeroRegister,
    Match_RequiresSameSrcAndDst,
    Match_NoFCCRegisterForCurrentISA,
    Match_NonZeroOperandForSync,
    Match_NonZeroOperandForMTCX,
    Match_RequiresPosSizeRange0_32,
    Match_RequiresPosSizeRange33_64,
    Match_RequiresPosSizeUImm6,
#define GET_OPERAND_DIAGNOSTIC_TYPES
#include "MipsGenAsmMatcher.inc"
#undef GET_OPERAND_DIAGNOSTIC_TYPES
```
- EN: Defines enumeration `MipsMatchResultTy` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `MipsMatchResultTy`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 523-523
```cpp
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 525-531
```cpp
  MipsAsmParser(const MCSubtargetInfo &sti, MCAsmParser &parser,
                const MCInstrInfo &MII)
      : MCTargetAsmParser(sti, MII),
        ABI(MipsABIInfo::computeTargetABI(
            sti.getTargetTriple(),
            parser.getContext().getTargetOptions().getABIName())) {
    MCAsmParserExtension::Initialize(parser);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 533-536
```cpp
    parser.addAliasForDirective(".asciiz", ".asciz");
    parser.addAliasForDirective(".hword", ".2byte");
    parser.addAliasForDirective(".word", ".4byte");
    parser.addAliasForDirective(".dword", ".8byte");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 538-539
```cpp
    // Initialize the set of available features.
    setAvailableFeatures(ComputeAvailableFeatures(getSTI().getFeatureBits()));
```
- EN: Declares `setAvailableFeatures`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setAvailableFeatures`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 541-543
```cpp
    // Remember the initial assembler options. The user can not modify these.
    AssemblerOptions.push_back(
        std::make_unique<MipsAssemblerOptions>(getSTI().getFeatureBits()));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 545-547
```cpp
    // Create an assembler options environment for the user to modify.
    AssemblerOptions.push_back(
        std::make_unique<MipsAssemblerOptions>(getSTI().getFeatureBits()));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 549-549
```cpp
    getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 551-552
```cpp
    if (!isABI_O32() && !useOddSPReg() != 0)
      report_fatal_error("-mno-odd-spreg requires the O32 ABI");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 554-554
```cpp
    CurrentFn = nullptr;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 556-557
```cpp
    CurForbiddenSlotAttr = false;
    IsPicEnabled = getContext().getObjectFileInfo()->isPositionIndependent();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 559-561
```cpp
    IsCpRestoreSet = false;
    CpRestoreOffset = -1;
    GPReg = ABI.GetGlobalPtr();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 563-564
```cpp
    const Triple &TheTriple = sti.getTargetTriple();
    IsLittleEndian = TheTriple.isLittleEndian();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 566-567
```cpp
    if (getSTI().getCPU() == "mips64r6" && inMicroMipsMode())
      report_fatal_error("microMIPS64R6 is not supported", false);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 569-571
```cpp
    if (!isABI_O32() && inMicroMipsMode())
      report_fatal_error("microMIPS64 is not supported", false);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 573-574
```cpp
  /// True if all of $fcc0 - $fcc7 exist for the current ISA.
  bool hasEightFccRegisters() const { return hasMips4() || hasMips32(); }
```
- EN: Implements `hasEightFccRegisters`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasEightFccRegisters`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 576-578
```cpp
  bool isGP64bit() const {
    return getSTI().hasFeature(Mips::FeatureGP64Bit);
  }
```
- EN: Implements `isGP64bit`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isGP64bit`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 580-582
```cpp
  bool isFP64bit() const {
    return getSTI().hasFeature(Mips::FeatureFP64Bit);
  }
```
- EN: Implements `isFP64bit`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isFP64bit`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 584-595
```cpp
  bool isJalrRelocAvailable(const MCExpr *JalExpr) {
    if (!EmitJalrReloc)
      return false;
    MCValue Res;
    if (!JalExpr->evaluateAsRelocatable(Res, nullptr))
      return false;
    if (Res.getSubSym())
      return false;
    if (Res.getConstant() != 0)
      return ABI.IsN32() || ABI.IsN64();
    return true;
  }
```
- EN: Implements `isJalrRelocAvailable`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isJalrRelocAvailable`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 597-603
```cpp
  const MipsABIInfo &getABI() const { return ABI; }
  bool isABI_N32() const { return ABI.IsN32(); }
  bool isABI_N64() const { return ABI.IsN64(); }
  bool isABI_O32() const { return ABI.IsO32(); }
  bool isABI_FPXX() const {
    return getSTI().hasFeature(Mips::FeatureFPXX);
  }
```
- EN: Implements `getABI`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getABI`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 605-607
```cpp
  bool useOddSPReg() const {
    return !(getSTI().hasFeature(Mips::FeatureNoOddSPReg));
  }
```
- EN: Implements `useOddSPReg`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `useOddSPReg`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 609-611
```cpp
  bool inMicroMipsMode() const {
    return getSTI().hasFeature(Mips::FeatureMicroMips);
  }
```
- EN: Implements `inMicroMipsMode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `inMicroMipsMode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 613-615
```cpp
  bool hasMips1() const {
    return getSTI().hasFeature(Mips::FeatureMips1);
  }
```
- EN: Implements `hasMips1`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips1`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 617-619
```cpp
  bool hasMips2() const {
    return getSTI().hasFeature(Mips::FeatureMips2);
  }
```
- EN: Implements `hasMips2`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips2`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 621-623
```cpp
  bool hasMips3() const {
    return getSTI().hasFeature(Mips::FeatureMips3);
  }
```
- EN: Implements `hasMips3`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips3`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 625-627
```cpp
  bool hasMips4() const {
    return getSTI().hasFeature(Mips::FeatureMips4);
  }
```
- EN: Implements `hasMips4`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips4`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 629-631
```cpp
  bool hasMips5() const {
    return getSTI().hasFeature(Mips::FeatureMips5);
  }
```
- EN: Implements `hasMips5`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips5`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 633-635
```cpp
  bool hasMips32() const {
    return getSTI().hasFeature(Mips::FeatureMips32);
  }
```
- EN: Implements `hasMips32`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips32`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 637-639
```cpp
  bool hasMips64() const {
    return getSTI().hasFeature(Mips::FeatureMips64);
  }
```
- EN: Implements `hasMips64`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips64`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 641-643
```cpp
  bool hasMips32r2() const {
    return getSTI().hasFeature(Mips::FeatureMips32r2);
  }
```
- EN: Implements `hasMips32r2`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips32r2`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 645-647
```cpp
  bool hasMips64r2() const {
    return getSTI().hasFeature(Mips::FeatureMips64r2);
  }
```
- EN: Implements `hasMips64r2`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips64r2`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 649-651
```cpp
  bool hasMips32r3() const {
    return (getSTI().hasFeature(Mips::FeatureMips32r3));
  }
```
- EN: Implements `hasMips32r3`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips32r3`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 653-655
```cpp
  bool hasMips64r3() const {
    return (getSTI().hasFeature(Mips::FeatureMips64r3));
  }
```
- EN: Implements `hasMips64r3`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips64r3`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 657-659
```cpp
  bool hasMips32r5() const {
    return (getSTI().hasFeature(Mips::FeatureMips32r5));
  }
```
- EN: Implements `hasMips32r5`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips32r5`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 661-663
```cpp
  bool hasMips64r5() const {
    return (getSTI().hasFeature(Mips::FeatureMips64r5));
  }
```
- EN: Implements `hasMips64r5`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips64r5`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 665-667
```cpp
  bool hasMips32r6() const {
    return getSTI().hasFeature(Mips::FeatureMips32r6);
  }
```
- EN: Implements `hasMips32r6`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips32r6`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 669-671
```cpp
  bool hasMips64r6() const {
    return getSTI().hasFeature(Mips::FeatureMips64r6);
  }
```
- EN: Implements `hasMips64r6`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips64r6`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 673-675
```cpp
  bool hasDSP() const {
    return getSTI().hasFeature(Mips::FeatureDSP);
  }
```
- EN: Implements `hasDSP`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasDSP`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 677-679
```cpp
  bool hasDSPR2() const {
    return getSTI().hasFeature(Mips::FeatureDSPR2);
  }
```
- EN: Implements `hasDSPR2`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasDSPR2`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 681-683
```cpp
  bool hasDSPR3() const {
    return getSTI().hasFeature(Mips::FeatureDSPR3);
  }
```
- EN: Implements `hasDSPR3`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasDSPR3`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 685-687
```cpp
  bool hasMSA() const {
    return getSTI().hasFeature(Mips::FeatureMSA);
  }
```
- EN: Implements `hasMSA`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMSA`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 689-691
```cpp
  bool hasCnMips() const {
    return (getSTI().hasFeature(Mips::FeatureCnMips));
  }
```
- EN: Implements `hasCnMips`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasCnMips`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 693-695
```cpp
  bool hasCnMipsP() const {
    return (getSTI().hasFeature(Mips::FeatureCnMipsP));
  }
```
- EN: Implements `hasCnMipsP`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasCnMipsP`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 697-697
```cpp
  bool isR5900() const { return (getSTI().hasFeature(Mips::FeatureR5900)); }
```
- EN: Implements `isR5900`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isR5900`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 699-701
```cpp
  bool inPicMode() {
    return IsPicEnabled;
  }
```
- EN: Implements `inPicMode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `inPicMode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 703-705
```cpp
  bool inMips16Mode() const {
    return getSTI().hasFeature(Mips::FeatureMips16);
  }
```
- EN: Implements `inMips16Mode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `inMips16Mode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 707-709
```cpp
  bool useTraps() const {
    return getSTI().hasFeature(Mips::FeatureUseTCCInDIV);
  }
```
- EN: Implements `useTraps`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `useTraps`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 711-713
```cpp
  bool useSoftFloat() const {
    return getSTI().hasFeature(Mips::FeatureSoftFloat);
  }
```
- EN: Implements `useSoftFloat`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `useSoftFloat`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 715-717
```cpp
  bool isSingleFloat() const {
    return getSTI().hasFeature(Mips::FeatureSingleFloat);
  }
```
- EN: Implements `isSingleFloat`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isSingleFloat`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 719-721
```cpp
  bool hasMT() const {
    return getSTI().hasFeature(Mips::FeatureMT);
  }
```
- EN: Implements `hasMT`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMT`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 723-725
```cpp
  bool hasCRC() const {
    return getSTI().hasFeature(Mips::FeatureCRC);
  }
```
- EN: Implements `hasCRC`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasCRC`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 727-729
```cpp
  bool hasVirt() const {
    return getSTI().hasFeature(Mips::FeatureVirt);
  }
```
- EN: Implements `hasVirt`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasVirt`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 731-733
```cpp
  bool hasGINV() const {
    return getSTI().hasFeature(Mips::FeatureGINV);
  }
```
- EN: Implements `hasGINV`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasGINV`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 735-737
```cpp
  bool hasForbiddenSlot(const MCInstrDesc &MCID) const {
    return !inMicroMipsMode() && (MCID.TSFlags & MipsII::HasForbiddenSlot);
  }
```
- EN: Implements `hasForbiddenSlot`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasForbiddenSlot`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 739-741
```cpp
  bool SafeInForbiddenSlot(const MCInstrDesc &MCID) const {
    return !(MCID.TSFlags & MipsII::IsCTI);
  }
```
- EN: Implements `SafeInForbiddenSlot`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `SafeInForbiddenSlot`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 743-743
```cpp
  void onEndOfFile() override;
```
- EN: Declares `onEndOfFile`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `onEndOfFile`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 745-746
```cpp
  /// Warn if RegIndex is the same as the current AT.
  void warnIfRegIndexIsAT(MCRegister RegIndex, SMLoc Loc);
```
- EN: Declares `warnIfRegIndexIsAT`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfRegIndexIsAT`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 748-748
```cpp
  void warnIfNoMacro(SMLoc Loc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 750-750
```cpp
  bool isLittle() const { return IsLittleEndian; }
```
- EN: Implements `isLittle`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isLittle`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 752-754
```cpp
  bool areEqualRegs(const MCParsedAsmOperand &Op1,
                    const MCParsedAsmOperand &Op2) const override;
};
```
- EN: Declares `areEqualRegs`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `areEqualRegs`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 756-770
```cpp
/// MipsOperand - Instances of this class represent a parsed Mips machine
/// instruction.
class MipsOperand : public MCParsedAsmOperand {
public:
  /// Broad categories of register classes
  /// The exact class is finalized by the render method.
  enum RegKind {
    RegKind_GPR = 1,      /// GPR32 and GPR64 (depending on isGP64bit())
    RegKind_FGR = 2,      /// FGR32, FGR64, AFGR64 (depending on context and
                          /// isFP64bit())
    RegKind_FCC = 4,      /// FCC
    RegKind_MSA128 = 8,   /// MSA128[BHWD] (makes no difference which)
    RegKind_MSACtrl = 16, /// MSA control registers
    RegKind_COP2 = 32,    /// COP2
    RegKind_ACC = 64,     /// HI32DSP, LO32DSP, and ACC64DSP (depending on
```
- EN: Declares `represent`, packaging target-specific state and APIs around `MipsAsmParser`.
- CN: 这里声明 `represent`，把与 `MipsAsmParser` 相关的目标特定状态和 API 组织在一起。

### Lines 771-780
```cpp
                          /// context).
    RegKind_CCR = 128,    /// CCR
    RegKind_HWRegs = 256, /// HWRegs
    RegKind_COP3 = 512,   /// COP3
    RegKind_COP0 = 1024,  /// COP0
    /// Potentially any (e.g. $1)
    RegKind_Numeric = RegKind_GPR | RegKind_FGR | RegKind_FCC | RegKind_MSA128 |
                      RegKind_MSACtrl | RegKind_COP2 | RegKind_ACC |
                      RegKind_CCR | RegKind_HWRegs | RegKind_COP3 | RegKind_COP0
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 782-789
```cpp
private:
  enum KindTy {
    k_Immediate,     /// An immediate (possibly involving symbol references)
    k_Memory,        /// Base + Offset Memory Address
    k_RegisterIndex, /// A register index in one or more RegKind.
    k_Token,         /// A simple token
    k_RegList,       /// A physical register list
  } Kind;
```
- EN: Defines enumeration `KindTy` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `KindTy`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 791-792
```cpp
public:
  MipsOperand(KindTy K, MipsAsmParser &Parser) : Kind(K), AsmParser(Parser) {}
```
- EN: Implements `MipsOperand`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsOperand`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 794-807
```cpp
  ~MipsOperand() override {
    switch (Kind) {
    case k_Memory:
      delete Mem.Base;
      break;
    case k_RegList:
      delete RegList.List;
      break;
    case k_Immediate:
    case k_RegisterIndex:
    case k_Token:
      break;
    }
  }
```
- EN: Implements `~MipsOperand`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `~MipsOperand`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 809-811
```cpp
private:
  /// For diagnostics, and checking the assembler temporary
  MipsAsmParser &AsmParser;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 813-816
```cpp
  struct Token {
    const char *Data;
    unsigned Length;
  };
```
- EN: Declares `Token`, packaging target-specific state and APIs around `MipsAsmParser`.
- CN: 这里声明 `Token`，把与 `MipsAsmParser` 相关的目标特定状态和 API 组织在一起。

### Lines 818-823
```cpp
  struct RegIdxOp {
    unsigned Index; /// Index into the register class
    RegKind Kind;   /// Bitfield of the kinds it could possibly be
    struct Token Tok; /// The input token this operand originated from.
    const MCRegisterInfo *RegInfo;
  };
```
- EN: Declares `RegIdxOp`, packaging target-specific state and APIs around `MipsAsmParser`.
- CN: 这里声明 `RegIdxOp`，把与 `MipsAsmParser` 相关的目标特定状态和 API 组织在一起。

### Lines 825-827
```cpp
  struct ImmOp {
    const MCExpr *Val;
  };
```
- EN: Declares `ImmOp`, packaging target-specific state and APIs around `MipsAsmParser`.
- CN: 这里声明 `ImmOp`，把与 `MipsAsmParser` 相关的目标特定状态和 API 组织在一起。

### Lines 829-832
```cpp
  struct MemOp {
    MipsOperand *Base;
    const MCExpr *Off;
  };
```
- EN: Declares `MemOp`, packaging target-specific state and APIs around `MipsAsmParser`.
- CN: 这里声明 `MemOp`，把与 `MipsAsmParser` 相关的目标特定状态和 API 组织在一起。

### Lines 834-836
```cpp
  struct RegListOp {
    SmallVector<MCRegister, 10> *List;
  };
```
- EN: Declares `RegListOp`, packaging target-specific state and APIs around `MipsAsmParser`.
- CN: 这里声明 `RegListOp`，把与 `MipsAsmParser` 相关的目标特定状态和 API 组织在一起。

### Lines 838-844
```cpp
  union {
    struct Token Tok;
    struct RegIdxOp RegIdx;
    struct ImmOp Imm;
    struct MemOp Mem;
    struct RegListOp RegList;
  };
```
- EN: Declares `Token`, packaging target-specific state and APIs around `MipsAsmParser`.
- CN: 这里声明 `Token`，把与 `MipsAsmParser` 相关的目标特定状态和 API 组织在一起。

### Lines 846-846
```cpp
  SMLoc StartLoc, EndLoc;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 848-862
```cpp
  /// Internal constructor for register kinds
  static std::unique_ptr<MipsOperand> CreateReg(unsigned Index, StringRef Str,
                                                RegKind RegKind,
                                                const MCRegisterInfo *RegInfo,
                                                SMLoc S, SMLoc E,
                                                MipsAsmParser &Parser) {
    auto Op = std::make_unique<MipsOperand>(k_RegisterIndex, Parser);
    Op->RegIdx.Index = Index;
    Op->RegIdx.RegInfo = RegInfo;
    Op->RegIdx.Kind = RegKind;
    Op->RegIdx.Tok.Data = Str.data();
    Op->RegIdx.Tok.Length = Str.size();
    Op->StartLoc = S;
    Op->EndLoc = E;
    return Op;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 863-863
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 865-873
```cpp
public:
  /// Coerce the register to GPR32 and return the real register for the current
  /// target.
  MCRegister getGPR32Reg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_GPR) && "Invalid access!");
    AsmParser.warnIfRegIndexIsAT(RegIdx.Index, StartLoc);
    unsigned ClassID = Mips::GPR32RegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getGPR32Reg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getGPR32Reg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 875-881
```cpp
  /// Coerce the register to GPR32 and return the real register for the current
  /// target.
  MCRegister getGPRMM16Reg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_GPR) && "Invalid access!");
    unsigned ClassID = Mips::GPR32RegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getGPRMM16Reg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getGPRMM16Reg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 883-889
```cpp
  /// Coerce the register to GPR64 and return the real register for the current
  /// target.
  MCRegister getGPR64Reg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_GPR) && "Invalid access!");
    unsigned ClassID = Mips::GPR64RegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getGPR64Reg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getGPR64Reg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 891-900
```cpp
private:
  /// Coerce the register to AFGR64 and return the real register for the current
  /// target.
  MCRegister getAFGR64Reg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_FGR) && "Invalid access!");
    if (RegIdx.Index % 2 != 0)
      AsmParser.Warning(StartLoc, "Float register should be even.");
    return RegIdx.RegInfo->getRegClass(Mips::AFGR64RegClassID)
        .getRegister(RegIdx.Index / 2);
  }
```
- EN: Implements `getAFGR64Reg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAFGR64Reg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 902-908
```cpp
  /// Coerce the register to FGR64 and return the real register for the current
  /// target.
  MCRegister getFGR64Reg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_FGR) && "Invalid access!");
    return RegIdx.RegInfo->getRegClass(Mips::FGR64RegClassID)
        .getRegister(RegIdx.Index);
  }
```
- EN: Implements `getFGR64Reg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFGR64Reg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 910-916
```cpp
  /// Coerce the register to FGR32 and return the real register for the current
  /// target.
  MCRegister getFGR32Reg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_FGR) && "Invalid access!");
    return RegIdx.RegInfo->getRegClass(Mips::FGR32RegClassID)
        .getRegister(RegIdx.Index);
  }
```
- EN: Implements `getFGR32Reg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFGR32Reg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 918-924
```cpp
  /// Coerce the register to FCC and return the real register for the current
  /// target.
  MCRegister getFCCReg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_FCC) && "Invalid access!");
    return RegIdx.RegInfo->getRegClass(Mips::FCCRegClassID)
        .getRegister(RegIdx.Index);
  }
```
- EN: Implements `getFCCReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFCCReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 926-934
```cpp
  /// Coerce the register to MSA128 and return the real register for the current
  /// target.
  MCRegister getMSA128Reg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_MSA128) && "Invalid access!");
    // It doesn't matter which of the MSA128[BHWD] classes we use. They are all
    // identical
    unsigned ClassID = Mips::MSA128BRegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getMSA128Reg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMSA128Reg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 936-942
```cpp
  /// Coerce the register to MSACtrl and return the real register for the
  /// current target.
  MCRegister getMSACtrlReg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_MSACtrl) && "Invalid access!");
    unsigned ClassID = Mips::MSACtrlRegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getMSACtrlReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMSACtrlReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 944-950
```cpp
  /// Coerce the register to COP0 and return the real register for the
  /// current target.
  MCRegister getCOP0Reg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_COP0) && "Invalid access!");
    unsigned ClassID = Mips::COP0RegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getCOP0Reg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getCOP0Reg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 952-958
```cpp
  /// Coerce the register to COP2 and return the real register for the
  /// current target.
  MCRegister getCOP2Reg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_COP2) && "Invalid access!");
    unsigned ClassID = Mips::COP2RegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getCOP2Reg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getCOP2Reg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 960-966
```cpp
  /// Coerce the register to COP3 and return the real register for the
  /// current target.
  MCRegister getCOP3Reg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_COP3) && "Invalid access!");
    unsigned ClassID = Mips::COP3RegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getCOP3Reg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getCOP3Reg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 968-974
```cpp
  /// Coerce the register to ACC64DSP and return the real register for the
  /// current target.
  MCRegister getACC64DSPReg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_ACC) && "Invalid access!");
    unsigned ClassID = Mips::ACC64DSPRegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getACC64DSPReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getACC64DSPReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 976-982
```cpp
  /// Coerce the register to HI32DSP and return the real register for the
  /// current target.
  MCRegister getHI32DSPReg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_ACC) && "Invalid access!");
    unsigned ClassID = Mips::HI32DSPRegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getHI32DSPReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getHI32DSPReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 984-990
```cpp
  /// Coerce the register to LO32DSP and return the real register for the
  /// current target.
  MCRegister getLO32DSPReg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_ACC) && "Invalid access!");
    unsigned ClassID = Mips::LO32DSPRegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getLO32DSPReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getLO32DSPReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 992-998
```cpp
  /// Coerce the register to CCR and return the real register for the
  /// current target.
  MCRegister getCCRReg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_CCR) && "Invalid access!");
    unsigned ClassID = Mips::CCRRegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getCCRReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getCCRReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1000-1006
```cpp
  /// Coerce the register to HWRegs and return the real register for the
  /// current target.
  MCRegister getHWRegsReg() const {
    assert(isRegIdx() && (RegIdx.Kind & RegKind_HWRegs) && "Invalid access!");
    unsigned ClassID = Mips::HWRegsRegClassID;
    return RegIdx.RegInfo->getRegClass(ClassID).getRegister(RegIdx.Index);
  }
```
- EN: Implements `getHWRegsReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getHWRegsReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1008-1017
```cpp
public:
  void addExpr(MCInst &Inst, const MCExpr *Expr) const {
    // Add as immediate when possible.  Null MCExpr = 0.
    if (!Expr)
      Inst.addOperand(MCOperand::createImm(0));
    else if (const MCConstantExpr *CE = dyn_cast<MCConstantExpr>(Expr))
      Inst.addOperand(MCOperand::createImm(CE->getValue()));
    else
      Inst.addOperand(MCOperand::createExpr(Expr));
  }
```
- EN: Implements `addExpr`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addExpr`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1019-1021
```cpp
  void addRegOperands(MCInst &Inst, unsigned N) const {
    llvm_unreachable("Use a custom parser instead");
  }
```
- EN: Implements `addRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1023-1029
```cpp
  /// Render the operand to an MCInst as a GPR32
  /// Asserts if the wrong number of operands are requested, or the operand
  /// is not a k_RegisterIndex compatible with RegKind_GPR
  void addGPR32ZeroAsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getGPR32Reg()));
  }
```
- EN: Implements `addGPR32ZeroAsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addGPR32ZeroAsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1031-1034
```cpp
  void addGPR32NonZeroAsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getGPR32Reg()));
  }
```
- EN: Implements `addGPR32NonZeroAsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addGPR32NonZeroAsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1036-1039
```cpp
  void addGPR32AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getGPR32Reg()));
  }
```
- EN: Implements `addGPR32AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addGPR32AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1041-1044
```cpp
  void addGPRMM16AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getGPRMM16Reg()));
  }
```
- EN: Implements `addGPRMM16AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addGPRMM16AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1046-1049
```cpp
  void addGPRMM16AsmRegZeroOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getGPRMM16Reg()));
  }
```
- EN: Implements `addGPRMM16AsmRegZeroOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addGPRMM16AsmRegZeroOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1051-1054
```cpp
  void addGPRMM16AsmRegMovePOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getGPRMM16Reg()));
  }
```
- EN: Implements `addGPRMM16AsmRegMovePOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addGPRMM16AsmRegMovePOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1056-1059
```cpp
  void addGPRMM16AsmRegMovePPairFirstOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getGPRMM16Reg()));
  }
```
- EN: Implements `addGPRMM16AsmRegMovePPairFirstOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addGPRMM16AsmRegMovePPairFirstOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1061-1065
```cpp
  void addGPRMM16AsmRegMovePPairSecondOperands(MCInst &Inst,
                                               unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getGPRMM16Reg()));
  }
```
- EN: Implements `addGPRMM16AsmRegMovePPairSecondOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addGPRMM16AsmRegMovePPairSecondOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1067-1073
```cpp
  /// Render the operand to an MCInst as a GPR64
  /// Asserts if the wrong number of operands are requested, or the operand
  /// is not a k_RegisterIndex compatible with RegKind_GPR
  void addGPR64AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getGPR64Reg()));
  }
```
- EN: Implements `addGPR64AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addGPR64AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1075-1078
```cpp
  void addAFGR64AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getAFGR64Reg()));
  }
```
- EN: Implements `addAFGR64AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addAFGR64AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1080-1083
```cpp
  void addStrictlyAFGR64AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getAFGR64Reg()));
  }
```
- EN: Implements `addStrictlyAFGR64AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addStrictlyAFGR64AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1085-1088
```cpp
  void addStrictlyFGR64AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getFGR64Reg()));
  }
```
- EN: Implements `addStrictlyFGR64AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addStrictlyFGR64AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1090-1093
```cpp
  void addFGR64AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getFGR64Reg()));
  }
```
- EN: Implements `addFGR64AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addFGR64AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1095-1104
```cpp
  void addFGR32AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getFGR32Reg()));
    // FIXME: We ought to do this for -integrated-as without -via-file-asm too.
    // FIXME: This should propagate failure up to parseStatement.
    if (!AsmParser.useOddSPReg() && RegIdx.Index & 1)
      AsmParser.getParser().printError(
          StartLoc, "-mno-odd-spreg prohibits the use of odd FPU "
                    "registers");
  }
```
- EN: Implements `addFGR32AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addFGR32AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1106-1113
```cpp
  void addStrictlyFGR32AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getFGR32Reg()));
    // FIXME: We ought to do this for -integrated-as without -via-file-asm too.
    if (!AsmParser.useOddSPReg() && RegIdx.Index & 1)
      AsmParser.Error(StartLoc, "-mno-odd-spreg prohibits the use of odd FPU "
                                "registers");
  }
```
- EN: Implements `addStrictlyFGR32AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addStrictlyFGR32AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1115-1118
```cpp
  void addFCCAsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getFCCReg()));
  }
```
- EN: Implements `addFCCAsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addFCCAsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1120-1123
```cpp
  void addMSA128AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getMSA128Reg()));
  }
```
- EN: Implements `addMSA128AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addMSA128AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1125-1128
```cpp
  void addMSACtrlAsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getMSACtrlReg()));
  }
```
- EN: Implements `addMSACtrlAsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addMSACtrlAsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1130-1133
```cpp
  void addCOP0AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getCOP0Reg()));
  }
```
- EN: Implements `addCOP0AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addCOP0AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1135-1138
```cpp
  void addCOP2AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getCOP2Reg()));
  }
```
- EN: Implements `addCOP2AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addCOP2AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1140-1143
```cpp
  void addCOP3AsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getCOP3Reg()));
  }
```
- EN: Implements `addCOP3AsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addCOP3AsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1145-1148
```cpp
  void addACC64DSPAsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getACC64DSPReg()));
  }
```
- EN: Implements `addACC64DSPAsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addACC64DSPAsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1150-1153
```cpp
  void addHI32DSPAsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getHI32DSPReg()));
  }
```
- EN: Implements `addHI32DSPAsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addHI32DSPAsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1155-1158
```cpp
  void addLO32DSPAsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getLO32DSPReg()));
  }
```
- EN: Implements `addLO32DSPAsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addLO32DSPAsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1160-1163
```cpp
  void addCCRAsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getCCRReg()));
  }
```
- EN: Implements `addCCRAsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addCCRAsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1165-1168
```cpp
  void addHWRegsAsmRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getHWRegsReg()));
  }
```
- EN: Implements `addHWRegsAsmRegOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addHWRegsAsmRegOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1170-1178
```cpp
  template <unsigned Bits, int Offset = 0, int AdjustOffset = 0>
  void addConstantUImmOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    uint64_t Imm = getConstantImm() - Offset;
    Imm &= (1ULL << Bits) - 1;
    Imm += Offset;
    Imm += AdjustOffset;
    Inst.addOperand(MCOperand::createImm(Imm));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1180-1187
```cpp
  template <unsigned Bits>
  void addSImmOperands(MCInst &Inst, unsigned N) const {
    if (isImm() && !isConstantImm()) {
      addExpr(Inst, getImm());
      return;
    }
    addConstantSImmOperands<Bits, 0, 0>(Inst, N);
  }
```
- EN: Implements `addSImmOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addSImmOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1189-1196
```cpp
  template <unsigned Bits>
  void addUImmOperands(MCInst &Inst, unsigned N) const {
    if (isImm() && !isConstantImm()) {
      addExpr(Inst, getImm());
      return;
    }
    addConstantUImmOperands<Bits, 0, 0>(Inst, N);
  }
```
- EN: Implements `addUImmOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addUImmOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1198-1206
```cpp
  template <unsigned Bits, int Offset = 0, int AdjustOffset = 0>
  void addConstantSImmOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    int64_t Imm = getConstantImm() - Offset;
    Imm = SignExtend64<Bits>(Imm);
    Imm += Offset;
    Imm += AdjustOffset;
    Inst.addOperand(MCOperand::createImm(Imm));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1208-1212
```cpp
  void addImmOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    const MCExpr *Expr = getImm();
    addExpr(Inst, Expr);
  }
```
- EN: Implements `addImmOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addImmOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1214-1215
```cpp
  void addMemOperands(MCInst &Inst, unsigned N) const {
    assert(N == 2 && "Invalid number of operands!");
```
- EN: Implements `addMemOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addMemOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1217-1219
```cpp
    Inst.addOperand(MCOperand::createReg(AsmParser.getABI().ArePtrs64bit()
                                             ? getMemBase()->getGPR64Reg()
                                             : getMemBase()->getGPR32Reg()));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1221-1223
```cpp
    const MCExpr *Expr = getMemOff();
    addExpr(Inst, Expr);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1225-1226
```cpp
  void addMicroMipsMemOperands(MCInst &Inst, unsigned N) const {
    assert(N == 2 && "Invalid number of operands!");
```
- EN: Implements `addMicroMipsMemOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addMicroMipsMemOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1228-1228
```cpp
    Inst.addOperand(MCOperand::createReg(getMemBase()->getGPRMM16Reg()));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1230-1232
```cpp
    const MCExpr *Expr = getMemOff();
    addExpr(Inst, Expr);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1234-1235
```cpp
  void addRegListOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
```
- EN: Implements `addRegListOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addRegListOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1237-1239
```cpp
    for (auto RegNo : getRegList())
      Inst.addOperand(MCOperand::createReg(RegNo));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1241-1245
```cpp
  bool isReg() const override {
    // As a special case until we sort out the definition of div/divu, accept
    // $0/$zero here so that MCK_ZERO works correctly.
    return isGPRAsmReg() && RegIdx.Index == 0;
  }
```
- EN: Implements `isReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1247-1248
```cpp
  bool isRegIdx() const { return Kind == k_RegisterIndex; }
  bool isImm() const override { return Kind == k_Immediate; }
```
- EN: Implements `isRegIdx`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isRegIdx`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1250-1253
```cpp
  bool isConstantImm() const {
    int64_t Res;
    return isImm() && getImm()->evaluateAsAbsolute(Res);
  }
```
- EN: Implements `isConstantImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isConstantImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1255-1257
```cpp
  bool isConstantImmz() const {
    return isConstantImm() && getConstantImm() == 0;
  }
```
- EN: Implements `isConstantImmz`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isConstantImmz`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1259-1261
```cpp
  template <unsigned Bits, int Offset = 0> bool isConstantUImm() const {
    return isConstantImm() && isUInt<Bits>(getConstantImm() - Offset);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1263-1271
```cpp
  template <unsigned Bits> bool isSImm() const {
    if (!isImm())
      return false;
    int64_t Res;
    if (getImm()->evaluateAsAbsolute(Res))
      return isInt<Bits>(Res);
    // Allow conservatively if not a parse-time constant.
    return true;
  }
```
- EN: Implements `isSImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isSImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1273-1281
```cpp
  template <unsigned Bits> bool isUImm() const {
    if (!isImm())
      return false;
    int64_t Res;
    if (getImm()->evaluateAsAbsolute(Res))
      return isUInt<Bits>(Res);
    // Allow conservatively if not a parse-time constant.
    return true;
  }
```
- EN: Implements `isUImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isUImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1283-1287
```cpp
  template <unsigned Bits> bool isAnyImm() const {
    return isConstantImm() ? (isInt<Bits>(getConstantImm()) ||
                              isUInt<Bits>(getConstantImm()))
                           : isImm();
  }
```
- EN: Implements `isAnyImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isAnyImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1289-1291
```cpp
  template <unsigned Bits, int Offset = 0> bool isConstantSImm() const {
    return isConstantImm() && isInt<Bits>(getConstantImm() - Offset);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1293-1296
```cpp
  template <unsigned Bottom, unsigned Top> bool isConstantUImmRange() const {
    return isConstantImm() && getConstantImm() >= Bottom &&
           getConstantImm() <= Top;
  }
```
- EN: Implements `isConstantUImmRange`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isConstantUImmRange`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1298-1302
```cpp
  bool isToken() const override {
    // Note: It's not possible to pretend that other operand kinds are tokens.
    // The matcher emitter checks tokens first.
    return Kind == k_Token;
  }
```
- EN: Implements `isToken`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isToken`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1304-1304
```cpp
  bool isMem() const override { return Kind == k_Memory; }
```
- EN: Implements `isMem`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMem`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1306-1308
```cpp
  bool isConstantMemOff() const {
    return isMem() && isa<MCConstantExpr>(getMemOff());
  }
```
- EN: Implements `isConstantMemOff`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isConstantMemOff`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1310-1324
```cpp
  // Allow relocation operators.
  template <unsigned Bits, unsigned ShiftAmount = 0>
  bool isMemWithSimmOffset() const {
    if (!isMem())
      return false;
    if (!getMemBase()->isGPRAsmReg())
      return false;
    if (isa<MCSpecifierExpr>(getMemOff()) ||
        (isConstantMemOff() &&
         isShiftedInt<Bits, ShiftAmount>(getConstantMemOff())))
      return true;
    MCValue Res;
    bool IsReloc = getMemOff()->evaluateAsRelocatable(Res, nullptr);
    return IsReloc && isShiftedInt<Bits, ShiftAmount>(Res.getConstant());
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1326-1338
```cpp
  bool isMemWithPtrSizeOffset() const {
    if (!isMem())
      return false;
    if (!getMemBase()->isGPRAsmReg())
      return false;
    const unsigned PtrBits = AsmParser.getABI().ArePtrs64bit() ? 64 : 32;
    if (isa<MCSpecifierExpr>(getMemOff()) ||
        (isConstantMemOff() && isIntN(PtrBits, getConstantMemOff())))
      return true;
    MCValue Res;
    bool IsReloc = getMemOff()->evaluateAsRelocatable(Res, nullptr);
    return IsReloc && isIntN(PtrBits, Res.getConstant());
  }
```
- EN: Implements `isMemWithPtrSizeOffset`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMemWithPtrSizeOffset`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1340-1342
```cpp
  bool isMemWithGRPMM16Base() const {
    return isMem() && getMemBase()->isMM16AsmReg();
  }
```
- EN: Implements `isMemWithGRPMM16Base`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMemWithGRPMM16Base`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1344-1347
```cpp
  template <unsigned Bits> bool isMemWithUimmOffsetSP() const {
    return isMem() && isConstantMemOff() && isUInt<Bits>(getConstantMemOff())
      && getMemBase()->isRegIdx() && (getMemBase()->getGPR32Reg() == Mips::SP);
  }
```
- EN: Implements `isMemWithUimmOffsetSP`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMemWithUimmOffsetSP`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1349-1353
```cpp
  template <unsigned Bits> bool isMemWithUimmWordAlignedOffsetSP() const {
    return isMem() && isConstantMemOff() && isUInt<Bits>(getConstantMemOff())
      && (getConstantMemOff() % 4 == 0) && getMemBase()->isRegIdx()
      && (getMemBase()->getGPR32Reg() == Mips::SP);
  }
```
- EN: Implements `isMemWithUimmWordAlignedOffsetSP`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMemWithUimmWordAlignedOffsetSP`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1355-1359
```cpp
  template <unsigned Bits> bool isMemWithSimmWordAlignedOffsetGP() const {
    return isMem() && isConstantMemOff() && isInt<Bits>(getConstantMemOff())
      && (getConstantMemOff() % 4 == 0) && getMemBase()->isRegIdx()
      && (getMemBase()->getGPR32Reg() == Mips::GP);
  }
```
- EN: Implements `isMemWithSimmWordAlignedOffsetGP`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMemWithSimmWordAlignedOffsetGP`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1361-1365
```cpp
  template <unsigned Bits, unsigned ShiftLeftAmount>
  bool isScaledUImm() const {
    return isConstantImm() &&
           isShiftedUInt<Bits, ShiftLeftAmount>(getConstantImm());
  }
```
- EN: Implements `isScaledUImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isScaledUImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1367-1379
```cpp
  template <unsigned Bits, unsigned ShiftLeftAmount>
  bool isScaledSImm() const {
    if (isConstantImm() &&
        isShiftedInt<Bits, ShiftLeftAmount>(getConstantImm()))
      return true;
    // Operand can also be a symbol or symbol plus
    // offset in case of relocations.
    if (Kind != k_Immediate)
      return false;
    MCValue Res;
    bool Success = getImm()->evaluateAsRelocatable(Res, nullptr);
    return Success && isShiftedInt<Bits, ShiftLeftAmount>(Res.getConstant());
  }
```
- EN: Implements `isScaledSImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isScaledSImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1381-1383
```cpp
  bool isRegList16() const {
    if (!isRegList())
      return false;
```
- EN: Implements `isRegList16`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isRegList16`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1385-1387
```cpp
    int Size = RegList.List->size();
    if (Size < 2 || Size > 5)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1389-1393
```cpp
    MCRegister R0 = RegList.List->front();
    MCRegister R1 = RegList.List->back();
    if (!((R0 == Mips::S0 && R1 == Mips::RA) ||
          (R0 == Mips::S0_64 && R1 == Mips::RA_64)))
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1395-1401
```cpp
    MCRegister PrevReg = RegList.List->front();
    for (int i = 1; i < Size - 1; i++) {
      MCRegister Reg = (*(RegList.List))[i];
      if ( Reg != PrevReg + 1)
        return false;
      PrevReg = Reg;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1403-1404
```cpp
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1406-1406
```cpp
  bool isInvNum() const { return Kind == k_Immediate; }
```
- EN: Implements `isInvNum`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isInvNum`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1408-1413
```cpp
  bool isLSAImm() const {
    if (!isConstantImm())
      return false;
    int64_t Val = getConstantImm();
    return 1 <= Val && Val <= 4;
  }
```
- EN: Implements `isLSAImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isLSAImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1415-1415
```cpp
  bool isRegList() const { return Kind == k_RegList; }
```
- EN: Implements `isRegList`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isRegList`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1417-1420
```cpp
  StringRef getToken() const {
    assert(Kind == k_Token && "Invalid access!");
    return StringRef(Tok.Data, Tok.Length);
  }
```
- EN: Implements `getToken`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getToken`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1422-1427
```cpp
  MCRegister getReg() const override {
    // As a special case until we sort out the definition of div/divu, accept
    // $0/$zero here so that MCK_ZERO works correctly.
    if (Kind == k_RegisterIndex && RegIdx.Index == 0 &&
        RegIdx.Kind & RegKind_GPR)
      return getGPR32Reg(); // FIXME: GPR64 too
```
- EN: Implements `getReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1429-1431
```cpp
    llvm_unreachable("Invalid access!");
    return 0;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1433-1436
```cpp
  const MCExpr *getImm() const {
    assert((Kind == k_Immediate) && "Invalid access!");
    return Imm.Val;
  }
```
- EN: Implements `getImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1438-1443
```cpp
  int64_t getConstantImm() const {
    const MCExpr *Val = getImm();
    int64_t Value = 0;
    (void)Val->evaluateAsAbsolute(Value);
    return Value;
  }
```
- EN: Implements `getConstantImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getConstantImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1445-1448
```cpp
  MipsOperand *getMemBase() const {
    assert((Kind == k_Memory) && "Invalid access!");
    return Mem.Base;
  }
```
- EN: Implements `getMemBase`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMemBase`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1450-1453
```cpp
  const MCExpr *getMemOff() const {
    assert((Kind == k_Memory) && "Invalid access!");
    return Mem.Off;
  }
```
- EN: Implements `getMemOff`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMemOff`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1455-1457
```cpp
  int64_t getConstantMemOff() const {
    return static_cast<const MCConstantExpr *>(getMemOff())->getValue();
  }
```
- EN: Implements `getConstantMemOff`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getConstantMemOff`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1459-1462
```cpp
  const SmallVectorImpl<MCRegister> &getRegList() const {
    assert((Kind == k_RegList) && "Invalid access!");
    return *(RegList.List);
  }
```
- EN: Implements `getRegList`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRegList`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1464-1472
```cpp
  static std::unique_ptr<MipsOperand> CreateToken(StringRef Str, SMLoc S,
                                                  MipsAsmParser &Parser) {
    auto Op = std::make_unique<MipsOperand>(k_Token, Parser);
    Op->Tok.Data = Str.data();
    Op->Tok.Length = Str.size();
    Op->StartLoc = S;
    Op->EndLoc = S;
    return Op;
  }
```
- EN: Implements `CreateToken`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CreateToken`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1474-1481
```cpp
  /// Create a numeric register (e.g. $1). The exact register remains
  /// unresolved until an instruction successfully matches
  static std::unique_ptr<MipsOperand>
  createNumericReg(unsigned Index, StringRef Str, const MCRegisterInfo *RegInfo,
                   SMLoc S, SMLoc E, MipsAsmParser &Parser) {
    LLVM_DEBUG(dbgs() << "createNumericReg(" << Index << ", ...)\n");
    return CreateReg(Index, Str, RegKind_Numeric, RegInfo, S, E, Parser);
  }
```
- EN: Implements `createNumericReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createNumericReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1483-1489
```cpp
  /// Create a register that is definitely a GPR.
  /// This is typically only used for named registers such as $gp.
  static std::unique_ptr<MipsOperand>
  createGPRReg(unsigned Index, StringRef Str, const MCRegisterInfo *RegInfo,
               SMLoc S, SMLoc E, MipsAsmParser &Parser) {
    return CreateReg(Index, Str, RegKind_GPR, RegInfo, S, E, Parser);
  }
```
- EN: Implements `createGPRReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createGPRReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1491-1497
```cpp
  /// Create a register that is definitely a FGR.
  /// This is typically only used for named registers such as $f0.
  static std::unique_ptr<MipsOperand>
  createFGRReg(unsigned Index, StringRef Str, const MCRegisterInfo *RegInfo,
               SMLoc S, SMLoc E, MipsAsmParser &Parser) {
    return CreateReg(Index, Str, RegKind_FGR, RegInfo, S, E, Parser);
  }
```
- EN: Implements `createFGRReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createFGRReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1499-1505
```cpp
  /// Create a register that is definitely a HWReg.
  /// This is typically only used for named registers such as $hwr_cpunum.
  static std::unique_ptr<MipsOperand>
  createHWRegsReg(unsigned Index, StringRef Str, const MCRegisterInfo *RegInfo,
                  SMLoc S, SMLoc E, MipsAsmParser &Parser) {
    return CreateReg(Index, Str, RegKind_HWRegs, RegInfo, S, E, Parser);
  }
```
- EN: Implements `createHWRegsReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createHWRegsReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1507-1513
```cpp
  /// Create a register that is definitely an FCC.
  /// This is typically only used for named registers such as $fcc0.
  static std::unique_ptr<MipsOperand>
  createFCCReg(unsigned Index, StringRef Str, const MCRegisterInfo *RegInfo,
               SMLoc S, SMLoc E, MipsAsmParser &Parser) {
    return CreateReg(Index, Str, RegKind_FCC, RegInfo, S, E, Parser);
  }
```
- EN: Implements `createFCCReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createFCCReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1515-1521
```cpp
  /// Create a register that is definitely an ACC.
  /// This is typically only used for named registers such as $ac0.
  static std::unique_ptr<MipsOperand>
  createACCReg(unsigned Index, StringRef Str, const MCRegisterInfo *RegInfo,
               SMLoc S, SMLoc E, MipsAsmParser &Parser) {
    return CreateReg(Index, Str, RegKind_ACC, RegInfo, S, E, Parser);
  }
```
- EN: Implements `createACCReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createACCReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1523-1529
```cpp
  /// Create a register that is definitely an MSA128.
  /// This is typically only used for named registers such as $w0.
  static std::unique_ptr<MipsOperand>
  createMSA128Reg(unsigned Index, StringRef Str, const MCRegisterInfo *RegInfo,
                  SMLoc S, SMLoc E, MipsAsmParser &Parser) {
    return CreateReg(Index, Str, RegKind_MSA128, RegInfo, S, E, Parser);
  }
```
- EN: Implements `createMSA128Reg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createMSA128Reg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1531-1537
```cpp
  /// Create a register that is definitely an MSACtrl.
  /// This is typically only used for named registers such as $msaaccess.
  static std::unique_ptr<MipsOperand>
  createMSACtrlReg(unsigned Index, StringRef Str, const MCRegisterInfo *RegInfo,
                   SMLoc S, SMLoc E, MipsAsmParser &Parser) {
    return CreateReg(Index, Str, RegKind_MSACtrl, RegInfo, S, E, Parser);
  }
```
- EN: Implements `createMSACtrlReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createMSACtrlReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1539-1546
```cpp
  static std::unique_ptr<MipsOperand>
  CreateImm(const MCExpr *Val, SMLoc S, SMLoc E, MipsAsmParser &Parser) {
    auto Op = std::make_unique<MipsOperand>(k_Immediate, Parser);
    Op->Imm.Val = Val;
    Op->StartLoc = S;
    Op->EndLoc = E;
    return Op;
  }
```
- EN: Implements `CreateImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CreateImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1548-1557
```cpp
  static std::unique_ptr<MipsOperand>
  CreateMem(std::unique_ptr<MipsOperand> Base, const MCExpr *Off, SMLoc S,
            SMLoc E, MipsAsmParser &Parser) {
    auto Op = std::make_unique<MipsOperand>(k_Memory, Parser);
    Op->Mem.Base = Base.release();
    Op->Mem.Off = Off;
    Op->StartLoc = S;
    Op->EndLoc = E;
    return Op;
  }
```
- EN: Implements `CreateMem`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CreateMem`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1559-1562
```cpp
  static std::unique_ptr<MipsOperand>
  CreateRegList(SmallVectorImpl<MCRegister> &Regs, SMLoc StartLoc, SMLoc EndLoc,
                MipsAsmParser &Parser) {
    assert(!Regs.empty() && "Empty list not allowed");
```
- EN: Implements `CreateRegList`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CreateRegList`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1564-1570
```cpp
    auto Op = std::make_unique<MipsOperand>(k_RegList, Parser);
    Op->RegList.List =
        new SmallVector<MCRegister, 10>(Regs.begin(), Regs.end());
    Op->StartLoc = StartLoc;
    Op->EndLoc = EndLoc;
    return Op;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1572-1574
```cpp
 bool isGPRZeroAsmReg() const {
    return isRegIdx() && RegIdx.Kind & RegKind_GPR && RegIdx.Index == 0;
  }
```
- EN: Implements `isGPRZeroAsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isGPRZeroAsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1576-1579
```cpp
 bool isGPRNonZeroAsmReg() const {
   return isRegIdx() && RegIdx.Kind & RegKind_GPR && RegIdx.Index > 0 &&
          RegIdx.Index <= 31;
  }
```
- EN: Implements `isGPRNonZeroAsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isGPRNonZeroAsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1581-1583
```cpp
  bool isGPRAsmReg() const {
    return isRegIdx() && RegIdx.Kind & RegKind_GPR && RegIdx.Index <= 31;
  }
```
- EN: Implements `isGPRAsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isGPRAsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1585-1589
```cpp
  bool isMM16AsmReg() const {
    if (!(isRegIdx() && RegIdx.Kind))
      return false;
    return ((RegIdx.Index >= 2 && RegIdx.Index <= 7)
            || RegIdx.Index == 16 || RegIdx.Index == 17);
```
- EN: Implements `isMM16AsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMM16AsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1591-1598
```cpp
  }
  bool isMM16AsmRegZero() const {
    if (!(isRegIdx() && RegIdx.Kind))
      return false;
    return (RegIdx.Index == 0 ||
            (RegIdx.Index >= 2 && RegIdx.Index <= 7) ||
            RegIdx.Index == 17);
  }
```
- EN: Implements `isMM16AsmRegZero`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMM16AsmRegZero`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1600-1605
```cpp
  bool isMM16AsmRegMoveP() const {
    if (!(isRegIdx() && RegIdx.Kind))
      return false;
    return (RegIdx.Index == 0 || (RegIdx.Index >= 2 && RegIdx.Index <= 3) ||
      (RegIdx.Index >= 16 && RegIdx.Index <= 20));
  }
```
- EN: Implements `isMM16AsmRegMoveP`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMM16AsmRegMoveP`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1607-1611
```cpp
  bool isMM16AsmRegMovePPairFirst() const {
    if (!(isRegIdx() && RegIdx.Kind))
      return false;
    return RegIdx.Index >= 4 && RegIdx.Index <= 6;
  }
```
- EN: Implements `isMM16AsmRegMovePPairFirst`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMM16AsmRegMovePPairFirst`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1613-1618
```cpp
  bool isMM16AsmRegMovePPairSecond() const {
    if (!(isRegIdx() && RegIdx.Kind))
      return false;
    return (RegIdx.Index == 21 || RegIdx.Index == 22 ||
      (RegIdx.Index >= 5 && RegIdx.Index <= 7));
  }
```
- EN: Implements `isMM16AsmRegMovePPairSecond`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMM16AsmRegMovePPairSecond`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1620-1623
```cpp
  bool isFGRAsmReg() const {
    // AFGR64 is $0-$15 but we handle this in getAFGR64()
    return isRegIdx() && RegIdx.Kind & RegKind_FGR && RegIdx.Index <= 31;
  }
```
- EN: Implements `isFGRAsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isFGRAsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1625-1628
```cpp
  bool isStrictlyFGRAsmReg() const {
    // AFGR64 is $0-$15 but we handle this in getAFGR64()
    return isRegIdx() && RegIdx.Kind == RegKind_FGR && RegIdx.Index <= 31;
  }
```
- EN: Implements `isStrictlyFGRAsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isStrictlyFGRAsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1630-1632
```cpp
  bool isHWRegsAsmReg() const {
    return isRegIdx() && RegIdx.Kind & RegKind_HWRegs && RegIdx.Index <= 31;
  }
```
- EN: Implements `isHWRegsAsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isHWRegsAsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1634-1636
```cpp
  bool isCCRAsmReg() const {
    return isRegIdx() && RegIdx.Kind & RegKind_CCR && RegIdx.Index <= 31;
  }
```
- EN: Implements `isCCRAsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isCCRAsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1638-1642
```cpp
  bool isFCCAsmReg() const {
    if (!(isRegIdx() && RegIdx.Kind & RegKind_FCC))
      return false;
    return RegIdx.Index <= 7;
  }
```
- EN: Implements `isFCCAsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isFCCAsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1644-1646
```cpp
  bool isACCAsmReg() const {
    return isRegIdx() && RegIdx.Kind & RegKind_ACC && RegIdx.Index <= 3;
  }
```
- EN: Implements `isACCAsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isACCAsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1648-1650
```cpp
  bool isCOP0AsmReg() const {
    return isRegIdx() && RegIdx.Kind & RegKind_COP0 && RegIdx.Index <= 31;
  }
```
- EN: Implements `isCOP0AsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isCOP0AsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1652-1654
```cpp
  bool isCOP2AsmReg() const {
    return isRegIdx() && RegIdx.Kind & RegKind_COP2 && RegIdx.Index <= 31;
  }
```
- EN: Implements `isCOP2AsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isCOP2AsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1656-1658
```cpp
  bool isCOP3AsmReg() const {
    return isRegIdx() && RegIdx.Kind & RegKind_COP3 && RegIdx.Index <= 31;
  }
```
- EN: Implements `isCOP3AsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isCOP3AsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1660-1662
```cpp
  bool isMSA128AsmReg() const {
    return isRegIdx() && RegIdx.Kind & RegKind_MSA128 && RegIdx.Index <= 31;
  }
```
- EN: Implements `isMSA128AsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMSA128AsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1664-1666
```cpp
  bool isMSACtrlAsmReg() const {
    return isRegIdx() && RegIdx.Kind & RegKind_MSACtrl && RegIdx.Index <= 7;
  }
```
- EN: Implements `isMSACtrlAsmReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMSACtrlAsmReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1668-1671
```cpp
  /// getStartLoc - Get the location of the first token of this operand.
  SMLoc getStartLoc() const override { return StartLoc; }
  /// getEndLoc - Get the location of the last token of this operand.
  SMLoc getEndLoc() const override { return EndLoc; }
```
- EN: Implements `getStartLoc`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getStartLoc`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1673-1687
```cpp
  void print(raw_ostream &OS, const MCAsmInfo &MAI) const override {
    switch (Kind) {
    case k_Immediate:
      OS << "Imm<";
      MAI.printExpr(OS, *Imm.Val);
      OS << ">";
      break;
    case k_Memory:
      OS << "Mem<";
      Mem.Base->print(OS, MAI);
      OS << ", ";
      MAI.printExpr(OS, *Mem.Off);
      OS << ">";
      break;
    case k_RegisterIndex:
```
- EN: Implements `print`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `print`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1688-1701
```cpp
      OS << "RegIdx<" << RegIdx.Index << ":" << RegIdx.Kind << ", "
         << StringRef(RegIdx.Tok.Data, RegIdx.Tok.Length) << ">";
      break;
    case k_Token:
      OS << getToken();
      break;
    case k_RegList:
      OS << "RegList< ";
      for (auto Reg : (*RegList.List))
        OS << Reg.id() << " ";
      OS <<  ">";
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1703-1705
```cpp
  bool isValidForTie(const MipsOperand &Other) const {
    if (Kind != Other.Kind)
      return false;
```
- EN: Implements `isValidForTie`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isValidForTie`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1707-1718
```cpp
    switch (Kind) {
    default:
      llvm_unreachable("Unexpected kind");
      return false;
    case k_RegisterIndex: {
      StringRef Token(RegIdx.Tok.Data, RegIdx.Tok.Length);
      StringRef OtherToken(Other.RegIdx.Tok.Data, Other.RegIdx.Tok.Length);
      return Token == OtherToken;
    }
    }
  }
}; // class MipsOperand
```
- EN: Declares `MipsOperand`, packaging target-specific state and APIs around `MipsAsmParser`.
- CN: 这里声明 `MipsOperand`，把与 `MipsAsmParser` 相关的目标特定状态和 API 组织在一起。

### Lines 1720-1720
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1722-1736
```cpp
static bool hasShortDelaySlot(MCInst &Inst) {
  switch (Inst.getOpcode()) {
    case Mips::BEQ_MM:
    case Mips::BNE_MM:
    case Mips::BLTZ_MM:
    case Mips::BGEZ_MM:
    case Mips::BLEZ_MM:
    case Mips::BGTZ_MM:
    case Mips::JRC16_MM:
    case Mips::JALS_MM:
    case Mips::JALRS_MM:
    case Mips::JALRS16_MM:
    case Mips::BGEZALS_MM:
    case Mips::BLTZALS_MM:
      return true;
```
- EN: Implements `hasShortDelaySlot`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasShortDelaySlot`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1737-1742
```cpp
    case Mips::J_MM:
      return !Inst.getOperand(0).isReg();
    default:
      return false;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1744-1747
```cpp
static const MCSymbol *getSingleMCSymbol(const MCExpr *Expr) {
  if (const MCSymbolRefExpr *SRExpr = dyn_cast<MCSymbolRefExpr>(Expr)) {
    return &SRExpr->getSymbol();
  }
```
- EN: Implements `getSingleMCSymbol`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSingleMCSymbol`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1749-1751
```cpp
  if (const MCBinaryExpr *BExpr = dyn_cast<MCBinaryExpr>(Expr)) {
    const MCSymbol *LHSSym = getSingleMCSymbol(BExpr->getLHS());
    const MCSymbol *RHSSym = getSingleMCSymbol(BExpr->getRHS());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1753-1754
```cpp
    if (LHSSym)
      return LHSSym;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1756-1757
```cpp
    if (RHSSym)
      return RHSSym;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1759-1760
```cpp
    return nullptr;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1762-1763
```cpp
  if (const MCUnaryExpr *UExpr = dyn_cast<MCUnaryExpr>(Expr))
    return getSingleMCSymbol(UExpr->getSubExpr());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1765-1766
```cpp
  return nullptr;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1768-1770
```cpp
static unsigned countMCSymbolRefExpr(const MCExpr *Expr) {
  if (isa<MCSymbolRefExpr>(Expr))
    return 1;
```
- EN: Implements `countMCSymbolRefExpr`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `countMCSymbolRefExpr`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1772-1774
```cpp
  if (const MCBinaryExpr *BExpr = dyn_cast<MCBinaryExpr>(Expr))
    return countMCSymbolRefExpr(BExpr->getLHS()) +
           countMCSymbolRefExpr(BExpr->getRHS());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1776-1777
```cpp
  if (const MCUnaryExpr *UExpr = dyn_cast<MCUnaryExpr>(Expr))
    return countMCSymbolRefExpr(UExpr->getSubExpr());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1779-1780
```cpp
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1782-1796
```cpp
static bool isEvaluated(const MCExpr *Expr) {
  switch (Expr->getKind()) {
  case MCExpr::Constant:
    return true;
  case MCExpr::SymbolRef:
    return (cast<MCSymbolRefExpr>(Expr)->getSpecifier());
  case MCExpr::Binary: {
    const MCBinaryExpr *BE = cast<MCBinaryExpr>(Expr);
    if (!isEvaluated(BE->getLHS()))
      return false;
    return isEvaluated(BE->getRHS());
  }
  case MCExpr::Unary:
    return isEvaluated(cast<MCUnaryExpr>(Expr)->getSubExpr());
  case MCExpr::Specifier:
```
- EN: Implements `isEvaluated`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isEvaluated`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1797-1802
```cpp
    return true;
  case MCExpr::Target:
    llvm_unreachable("unused by this backend");
  }
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1804-1807
```cpp
static bool needsExpandMemInst(MCInst &Inst, const MCInstrDesc &MCID) {
  unsigned NumOp = MCID.getNumOperands();
  if (NumOp != 3 && NumOp != 4)
    return false;
```
- EN: Implements `needsExpandMemInst`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `needsExpandMemInst`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1809-1813
```cpp
  const MCOperandInfo &OpInfo = MCID.operands()[NumOp - 1];
  if (OpInfo.OperandType != MCOI::OPERAND_MEMORY &&
      OpInfo.OperandType != MCOI::OPERAND_UNKNOWN &&
      OpInfo.OperandType != MipsII::OPERAND_MEM_SIMM9)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1815-1821
```cpp
  MCOperand &Op = Inst.getOperand(NumOp - 1);
  if (Op.isImm()) {
    if (OpInfo.OperandType == MipsII::OPERAND_MEM_SIMM9)
      return !isInt<9>(Op.getImm());
    // Offset can't exceed 16bit value.
    return !isInt<16>(Op.getImm());
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1823-1826
```cpp
  if (Op.isExpr()) {
    const MCExpr *Expr = Op.getExpr();
    if (Expr->getKind() != MCExpr::SymbolRef)
      return !isEvaluated(Expr);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1828-1831
```cpp
    // Expand symbol.
    const MCSymbolRefExpr *SR = static_cast<const MCSymbolRefExpr *>(Expr);
    return SR->getSpecifier() == 0;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1833-1834
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1836-1842
```cpp
bool MipsAsmParser::processInstruction(MCInst &Inst, SMLoc IDLoc,
                                       MCStreamer &Out,
                                       const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  const unsigned Opcode = Inst.getOpcode();
  const MCInstrDesc &MCID = MII.get(Opcode);
  bool ExpandedJalSym = false;
```
- EN: Implements `MipsAsmParser::processInstruction`, a target-specific routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::processInstruction`，它是一个围绕MC streamer 交互展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1844-1844
```cpp
  Inst.setLoc(IDLoc);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1846-1847
```cpp
  if (MCID.isBranch() || MCID.isCall()) {
    MCOperand Offset;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1849-1857
```cpp
    switch (Opcode) {
    default:
      break;
    case Mips::BBIT0:
    case Mips::BBIT032:
    case Mips::BBIT1:
    case Mips::BBIT132:
      assert(hasCnMips() && "instruction only valid for octeon cpus");
      [[fallthrough]];
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1859-1873
```cpp
    case Mips::BEQ:
    case Mips::BNE:
    case Mips::BEQ_MM:
    case Mips::BNE_MM:
      assert(MCID.getNumOperands() == 3 && "unexpected number of operands");
      Offset = Inst.getOperand(2);
      if (!Offset.isImm())
        break; // We'll deal with this situation later on when applying fixups.
      if (!isIntN(inMicroMipsMode() ? 17 : 18, Offset.getImm()))
        return Error(IDLoc, "branch target out of range");
      if (offsetToAlignment(Offset.getImm(),
                            (inMicroMipsMode() ? Align(2) : Align(4))))
        return Error(IDLoc, "branch to misaligned address");
      break;
    case Mips::BGEZ:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1874-1888
```cpp
    case Mips::BGTZ:
    case Mips::BLEZ:
    case Mips::BLTZ:
    case Mips::BGEZAL:
    case Mips::BLTZAL:
    case Mips::BC1F:
    case Mips::BC1T:
    case Mips::BGEZ_MM:
    case Mips::BGTZ_MM:
    case Mips::BLEZ_MM:
    case Mips::BLTZ_MM:
    case Mips::BGEZAL_MM:
    case Mips::BLTZAL_MM:
    case Mips::BC1F_MM:
    case Mips::BC1T_MM:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1889-1903
```cpp
    case Mips::BC1EQZC_MMR6:
    case Mips::BC1NEZC_MMR6:
    case Mips::BC2EQZC_MMR6:
    case Mips::BC2NEZC_MMR6:
      assert(MCID.getNumOperands() == 2 && "unexpected number of operands");
      Offset = Inst.getOperand(1);
      if (!Offset.isImm())
        break; // We'll deal with this situation later on when applying fixups.
      if (!isIntN(inMicroMipsMode() ? 17 : 18, Offset.getImm()))
        return Error(IDLoc, "branch target out of range");
      if (offsetToAlignment(Offset.getImm(),
                            (inMicroMipsMode() ? Align(2) : Align(4))))
        return Error(IDLoc, "branch to misaligned address");
      break;
    case Mips::BGEC:    case Mips::BGEC_MMR6:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1904-1918
```cpp
    case Mips::BLTC:    case Mips::BLTC_MMR6:
    case Mips::BGEUC:   case Mips::BGEUC_MMR6:
    case Mips::BLTUC:   case Mips::BLTUC_MMR6:
    case Mips::BEQC:    case Mips::BEQC_MMR6:
    case Mips::BNEC:    case Mips::BNEC_MMR6:
      assert(MCID.getNumOperands() == 3 && "unexpected number of operands");
      Offset = Inst.getOperand(2);
      if (!Offset.isImm())
        break; // We'll deal with this situation later on when applying fixups.
      if (!isIntN(18, Offset.getImm()))
        return Error(IDLoc, "branch target out of range");
      if (offsetToAlignment(Offset.getImm(), Align(4)))
        return Error(IDLoc, "branch to misaligned address");
      break;
    case Mips::BLEZC:   case Mips::BLEZC_MMR6:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1919-1933
```cpp
    case Mips::BGEZC:   case Mips::BGEZC_MMR6:
    case Mips::BGTZC:   case Mips::BGTZC_MMR6:
    case Mips::BLTZC:   case Mips::BLTZC_MMR6:
      assert(MCID.getNumOperands() == 2 && "unexpected number of operands");
      Offset = Inst.getOperand(1);
      if (!Offset.isImm())
        break; // We'll deal with this situation later on when applying fixups.
      if (!isIntN(18, Offset.getImm()))
        return Error(IDLoc, "branch target out of range");
      if (offsetToAlignment(Offset.getImm(), Align(4)))
        return Error(IDLoc, "branch to misaligned address");
      break;
    case Mips::BEQZC:   case Mips::BEQZC_MMR6:
    case Mips::BNEZC:   case Mips::BNEZC_MMR6:
      assert(MCID.getNumOperands() == 2 && "unexpected number of operands");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1934-1948
```cpp
      Offset = Inst.getOperand(1);
      if (!Offset.isImm())
        break; // We'll deal with this situation later on when applying fixups.
      if (!isIntN(23, Offset.getImm()))
        return Error(IDLoc, "branch target out of range");
      if (offsetToAlignment(Offset.getImm(), Align(4)))
        return Error(IDLoc, "branch to misaligned address");
      break;
    case Mips::BEQZ16_MM:
    case Mips::BEQZC16_MMR6:
    case Mips::BNEZ16_MM:
    case Mips::BNEZC16_MMR6:
      assert(MCID.getNumOperands() == 2 && "unexpected number of operands");
      Offset = Inst.getOperand(1);
      if (!Offset.isImm())
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1949-1956
```cpp
        break; // We'll deal with this situation later on when applying fixups.
      if (!isInt<8>(Offset.getImm()))
        return Error(IDLoc, "branch target out of range");
      if (offsetToAlignment(Offset.getImm(), Align(2)))
        return Error(IDLoc, "branch to misaligned address");
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1958-1964
```cpp
  // SSNOP is deprecated on MIPS32r6/MIPS64r6
  // We still accept it but it is a normal nop.
  if (hasMips32r6() && Opcode == Mips::SSNOP) {
    std::string ISA = hasMips64r6() ? "MIPS64r6" : "MIPS32r6";
    Warning(IDLoc, "ssnop is deprecated for " + ISA + " and is equivalent to a "
                                                      "nop instruction");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1966-1968
```cpp
  if (hasCnMips()) {
    MCOperand Opnd;
    int Imm;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1970-1972
```cpp
    switch (Opcode) {
      default:
        break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1974-1988
```cpp
      case Mips::BBIT0:
      case Mips::BBIT032:
      case Mips::BBIT1:
      case Mips::BBIT132:
        assert(MCID.getNumOperands() == 3 && "unexpected number of operands");
        // The offset is handled above
        Opnd = Inst.getOperand(1);
        if (!Opnd.isImm())
          return Error(IDLoc, "expected immediate operand kind");
        Imm = Opnd.getImm();
        if (Imm < 0 || Imm > (Opcode == Mips::BBIT0 ||
                              Opcode == Mips::BBIT1 ? 63 : 31))
          return Error(IDLoc, "immediate operand value out of range");
        if (Imm > 31) {
          Inst.setOpcode(Opcode == Mips::BBIT0 ? Mips::BBIT032
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1989-1992
```cpp
                                               : Mips::BBIT132);
          Inst.getOperand(1).setImm(Imm - 32);
        }
        break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1994-2005
```cpp
      case Mips::SEQi:
      case Mips::SNEi:
        assert(MCID.getNumOperands() == 3 && "unexpected number of operands");
        Opnd = Inst.getOperand(2);
        if (!Opnd.isImm())
          return Error(IDLoc, "expected immediate operand kind");
        Imm = Opnd.getImm();
        if (!isInt<10>(Imm))
          return Error(IDLoc, "immediate operand value out of range");
        break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2007-2021
```cpp
  // Warn on division by zero. We're checking here as all instructions get
  // processed here, not just the macros that need expansion.
  //
  // The MIPS backend models most of the divison instructions and macros as
  // three operand instructions. The pre-R6 divide instructions however have
  // two operands and explicitly define HI/LO as part of the instruction,
  // not in the operands.
  unsigned FirstOp = 1;
  unsigned SecondOp = 2;
  switch (Opcode) {
  default:
    break;
  case Mips::SDivIMacro:
  case Mips::UDivIMacro:
  case Mips::DSDivIMacro:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2022-2036
```cpp
  case Mips::DUDivIMacro:
    if (Inst.getOperand(2).getImm() == 0) {
      if (Inst.getOperand(1).getReg() == Mips::ZERO ||
          Inst.getOperand(1).getReg() == Mips::ZERO_64)
        Warning(IDLoc, "dividing zero by zero");
      else
        Warning(IDLoc, "division by zero");
    }
    break;
  case Mips::DSDIV:
  case Mips::SDIV:
  case Mips::UDIV:
  case Mips::DUDIV:
  case Mips::UDIV_MM:
  case Mips::SDIV_MM:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2037-2051
```cpp
    FirstOp = 0;
    SecondOp = 1;
    [[fallthrough]];
  case Mips::SDivMacro:
  case Mips::DSDivMacro:
  case Mips::UDivMacro:
  case Mips::DUDivMacro:
  case Mips::DIV:
  case Mips::DIVU:
  case Mips::DDIV:
  case Mips::DDIVU:
  case Mips::DIVU_MMR6:
  case Mips::DIV_MMR6:
    if (Inst.getOperand(SecondOp).getReg() == Mips::ZERO ||
        Inst.getOperand(SecondOp).getReg() == Mips::ZERO_64) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2052-2059
```cpp
      if (Inst.getOperand(FirstOp).getReg() == Mips::ZERO ||
          Inst.getOperand(FirstOp).getReg() == Mips::ZERO_64)
        Warning(IDLoc, "dividing zero by zero");
      else
        Warning(IDLoc, "division by zero");
    }
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2061-2069
```cpp
  // For PIC code convert unconditional jump to unconditional branch.
  if ((Opcode == Mips::J || Opcode == Mips::J_MM) && inPicMode()) {
    MCInst BInst;
    BInst.setOpcode(inMicroMipsMode() ? Mips::BEQ_MM : Mips::BEQ);
    BInst.addOperand(MCOperand::createReg(Mips::ZERO));
    BInst.addOperand(MCOperand::createReg(Mips::ZERO));
    BInst.addOperand(Inst.getOperand(0));
    Inst = BInst;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2071-2074
```cpp
  // This expansion is not in a function called by tryExpandInstruction()
  // because the pseudo-instruction doesn't have a distinct opcode.
  if ((Opcode == Mips::JAL || Opcode == Mips::JAL_MM) && inPicMode()) {
    warnIfNoMacro(IDLoc);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2076-2078
```cpp
    if (!Inst.getOperand(0).isExpr()) {
      return Error(IDLoc, "unsupported constant in relocation");
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2080-2080
```cpp
    const MCExpr *JalExpr = Inst.getOperand(0).getExpr();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2082-2085
```cpp
    // We can do this expansion if there's only 1 symbol in the argument
    // expression.
    if (countMCSymbolRefExpr(JalExpr) > 1)
      return Error(IDLoc, "jal doesn't support multiple symbols in PIC mode");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2087-2089
```cpp
    // FIXME: This is checking the expression can be handled by the later stages
    //        of the assembler. We ought to leave it to those later stages.
    const MCSymbol *JalSym = getSingleMCSymbol(JalExpr);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2091-2093
```cpp
    if (expandLoadAddress(Mips::T9, MCRegister(), Inst.getOperand(0),
                          !isGP64bit(), IDLoc, Out, STI))
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2095-2101
```cpp
    MCInst JalrInst;
    if (inMicroMipsMode())
      JalrInst.setOpcode(IsCpRestoreSet ? Mips::JALRS_MM : Mips::JALR_MM);
    else
      JalrInst.setOpcode(Mips::JALR);
    JalrInst.addOperand(MCOperand::createReg(Mips::RA));
    JalrInst.addOperand(MCOperand::createReg(Mips::T9));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2103-2110
```cpp
    if (isJalrRelocAvailable(JalExpr)) {
      // As an optimization hint for the linker, before the JALR we add:
      // .reloc tmplabel, R_{MICRO}MIPS_JALR, symbol
      // tmplabel:
      MCSymbol *TmpLabel = getContext().createTempSymbol();
      const MCExpr *TmpExpr = MCSymbolRefExpr::create(TmpLabel, getContext());
      const MCExpr *RelocJalrExpr =
          MCSymbolRefExpr::create(JalSym, getContext(), IDLoc);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2112-2116
```cpp
      TOut.getStreamer().emitRelocDirective(
          *TmpExpr, inMicroMipsMode() ? "R_MICROMIPS_JALR" : "R_MIPS_JALR",
          RelocJalrExpr);
      TOut.getStreamer().emitLabel(TmpLabel);
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2118-2120
```cpp
    Inst = JalrInst;
    ExpandedJalSym = true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2122-2136
```cpp
  if (MCID.mayLoad() || MCID.mayStore()) {
    // Check the offset of memory operand, if it is a symbol
    // reference or immediate we may have to expand instructions.
    if (needsExpandMemInst(Inst, MCID)) {
      switch (MCID.operands()[MCID.getNumOperands() - 1].OperandType) {
      case MipsII::OPERAND_MEM_SIMM9:
        expandMem9Inst(Inst, IDLoc, Out, STI, MCID.mayLoad());
        break;
      default:
        expandMem16Inst(Inst, IDLoc, Out, STI, MCID.mayLoad());
        break;
      }
      return getParser().hasPendingError();
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2138-2152
```cpp
  if (inMicroMipsMode()) {
    if (MCID.mayLoad() && Opcode != Mips::LWP_MM) {
      // Try to create 16-bit GP relative load instruction.
      for (unsigned i = 0; i < MCID.getNumOperands(); i++) {
        const MCOperandInfo &OpInfo = MCID.operands()[i];
        if ((OpInfo.OperandType == MCOI::OPERAND_MEMORY) ||
            (OpInfo.OperandType == MCOI::OPERAND_UNKNOWN)) {
          MCOperand &Op = Inst.getOperand(i);
          if (Op.isImm()) {
            int MemOffset = Op.getImm();
            MCOperand &DstReg = Inst.getOperand(0);
            MCOperand &BaseReg = Inst.getOperand(1);
            if (isInt<9>(MemOffset) && (MemOffset % 4 == 0) &&
                getContext().getRegisterInfo()->getRegClass(
                  Mips::GPRMM16RegClassID).contains(DstReg.getReg()) &&
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2153-2154
```cpp
                (BaseReg.getReg() == Mips::GP ||
                BaseReg.getReg() == Mips::GP_64)) {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2156-2163
```cpp
              TOut.emitRRI(Mips::LWGP_MM, DstReg.getReg(), Mips::GP, MemOffset,
                           IDLoc, STI);
              return false;
            }
          }
        }
      } // for
    }   // if load
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2165-2165
```cpp
    // TODO: Handle this with the AsmOperandClass.PredicateMethod.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2167-2168
```cpp
    MCOperand Opnd;
    int Imm;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2170-2184
```cpp
    switch (Opcode) {
      default:
        break;
      case Mips::ADDIUSP_MM:
        Opnd = Inst.getOperand(0);
        if (!Opnd.isImm())
          return Error(IDLoc, "expected immediate operand kind");
        Imm = Opnd.getImm();
        if (Imm < -1032 || Imm > 1028 || (Imm < 8 && Imm > -12) ||
            Imm % 4 != 0)
          return Error(IDLoc, "immediate operand value out of range");
        break;
      case Mips::SLL16_MM:
      case Mips::SRL16_MM:
        Opnd = Inst.getOperand(2);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2185-2199
```cpp
        if (!Opnd.isImm())
          return Error(IDLoc, "expected immediate operand kind");
        Imm = Opnd.getImm();
        if (Imm < 1 || Imm > 8)
          return Error(IDLoc, "immediate operand value out of range");
        break;
      case Mips::LI16_MM:
        Opnd = Inst.getOperand(1);
        if (!Opnd.isImm())
          return Error(IDLoc, "expected immediate operand kind");
        Imm = Opnd.getImm();
        if (Imm < -1 || Imm > 126)
          return Error(IDLoc, "immediate operand value out of range");
        break;
      case Mips::ADDIUR2_MM:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2200-2214
```cpp
        Opnd = Inst.getOperand(2);
        if (!Opnd.isImm())
          return Error(IDLoc, "expected immediate operand kind");
        Imm = Opnd.getImm();
        if (!(Imm == 1 || Imm == -1 ||
              ((Imm % 4 == 0) && Imm < 28 && Imm > 0)))
          return Error(IDLoc, "immediate operand value out of range");
        break;
      case Mips::ANDI16_MM:
        Opnd = Inst.getOperand(2);
        if (!Opnd.isImm())
          return Error(IDLoc, "expected immediate operand kind");
        Imm = Opnd.getImm();
        if (!(Imm == 128 || (Imm >= 1 && Imm <= 4) || Imm == 7 || Imm == 8 ||
              Imm == 15 || Imm == 16 || Imm == 31 || Imm == 32 || Imm == 63 ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2215-2229
```cpp
              Imm == 64 || Imm == 255 || Imm == 32768 || Imm == 65535))
          return Error(IDLoc, "immediate operand value out of range");
        break;
      case Mips::LBU16_MM:
        Opnd = Inst.getOperand(2);
        if (!Opnd.isImm())
          return Error(IDLoc, "expected immediate operand kind");
        Imm = Opnd.getImm();
        if (Imm < -1 || Imm > 14)
          return Error(IDLoc, "immediate operand value out of range");
        break;
      case Mips::SB16_MM:
      case Mips::SB16_MMR6:
        Opnd = Inst.getOperand(2);
        if (!Opnd.isImm())
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2230-2244
```cpp
          return Error(IDLoc, "expected immediate operand kind");
        Imm = Opnd.getImm();
        if (Imm < 0 || Imm > 15)
          return Error(IDLoc, "immediate operand value out of range");
        break;
      case Mips::LHU16_MM:
      case Mips::SH16_MM:
      case Mips::SH16_MMR6:
        Opnd = Inst.getOperand(2);
        if (!Opnd.isImm())
          return Error(IDLoc, "expected immediate operand kind");
        Imm = Opnd.getImm();
        if (Imm < 0 || Imm > 30 || (Imm % 2 != 0))
          return Error(IDLoc, "immediate operand value out of range");
        break;
```
- EN: Declares `Error`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Error`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2245-2259
```cpp
      case Mips::LW16_MM:
      case Mips::SW16_MM:
      case Mips::SW16_MMR6:
        Opnd = Inst.getOperand(2);
        if (!Opnd.isImm())
          return Error(IDLoc, "expected immediate operand kind");
        Imm = Opnd.getImm();
        if (Imm < 0 || Imm > 60 || (Imm % 4 != 0))
          return Error(IDLoc, "immediate operand value out of range");
        break;
      case Mips::ADDIUPC_MM:
        Opnd = Inst.getOperand(1);
        if (!Opnd.isImm())
          return Error(IDLoc, "expected immediate operand kind");
        Imm = Opnd.getImm();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2260-2274
```cpp
        if ((Imm % 4 != 0) || !isInt<25>(Imm))
          return Error(IDLoc, "immediate operand value out of range");
        break;
      case Mips::LWP_MM:
      case Mips::SWP_MM:
        if (Inst.getOperand(0).getReg() == Mips::RA)
          return Error(IDLoc, "invalid operand for instruction");
        break;
      case Mips::MOVEP_MM:
      case Mips::MOVEP_MMR6: {
        MCRegister R0 = Inst.getOperand(0).getReg();
        MCRegister R1 = Inst.getOperand(1).getReg();
        bool RegPair = ((R0 == Mips::A1 && R1 == Mips::A2) ||
                        (R0 == Mips::A1 && R1 == Mips::A3) ||
                        (R0 == Mips::A2 && R1 == Mips::A3) ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2275-2285
```cpp
                        (R0 == Mips::A0 && R1 == Mips::S5) ||
                        (R0 == Mips::A0 && R1 == Mips::S6) ||
                        (R0 == Mips::A0 && R1 == Mips::A1) ||
                        (R0 == Mips::A0 && R1 == Mips::A2) ||
                        (R0 == Mips::A0 && R1 == Mips::A3));
        if (!RegPair)
          return Error(IDLoc, "invalid operand for instruction");
        break;
      }
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2287-2288
```cpp
  bool FillDelaySlot =
      MCID.hasDelaySlot() && AssemblerOptions.back()->isReorder();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2290-2292
```cpp
  // Get previous instruction`s forbidden slot attribute and
  // whether set reorder.
  bool PrevForbiddenSlotAttr = CurForbiddenSlotAttr;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2294-2295
```cpp
  // Flag represents we set reorder after nop.
  bool SetReorderAfterNop = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2297-2311
```cpp
  // If previous instruction has forbidden slot and .set reorder
  // is active and current instruction is CTI.
  // Then emit a NOP after it.
  if (PrevForbiddenSlotAttr && !SafeInForbiddenSlot(MCID)) {
    TOut.emitEmptyDelaySlot(false, IDLoc, STI);
    // When 'FillDelaySlot' is true, the existing logic will add
    // noreorder before instruction and reorder after it. So there
    // need exclude this case avoiding two '.set reorder'.
    // The format of the first case is:
    // .set noreorder
    // bnezc
    // nop
    // .set reorder
    if (AssemblerOptions.back()->isReorder() && !FillDelaySlot) {
      SetReorderAfterNop = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2312-2314
```cpp
      TOut.emitDirectiveSetReorder();
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2316-2321
```cpp
  // Save current instruction`s forbidden slot and whether set reorder.
  // This is the judgment condition for whether to add nop.
  // We would add a couple of '.set noreorder' and '.set reorder' to
  // wrap the current instruction and the next instruction.
  CurForbiddenSlotAttr =
      hasForbiddenSlot(MCID) && AssemblerOptions.back()->isReorder();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2323-2324
```cpp
  if (FillDelaySlot || CurForbiddenSlotAttr)
    TOut.emitDirectiveSetNoReorder();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2326-2336
```cpp
  MacroExpanderResultTy ExpandResult =
      tryExpandInstruction(Inst, IDLoc, Out, STI);
  switch (ExpandResult) {
  case MER_NotAMacro:
    Out.emitInstruction(Inst, *STI);
    break;
  case MER_Success:
    break;
  case MER_Fail:
    return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2338-2347
```cpp
  // When current instruction was not CTI, recover reorder state.
  // The format of the second case is:
  // .set noreoder
  // bnezc
  // add
  // .set reorder
  if (PrevForbiddenSlotAttr && !SetReorderAfterNop && !FillDelaySlot &&
      AssemblerOptions.back()->isReorder()) {
    TOut.emitDirectiveSetReorder();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2349-2354
```cpp
  // We know we emitted an instruction on the MER_NotAMacro or MER_Success path.
  // If we're in microMIPS mode then we must also set EF_MIPS_MICROMIPS.
  if (inMicroMipsMode()) {
    TOut.setUsesMicroMips();
    TOut.updateABIInfo(*this);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2356-2369
```cpp
  // If this instruction has a delay slot and .set reorder is active,
  // emit a NOP after it.
  // The format of the third case is:
  // .set noreorder
  // bnezc
  // nop
  // .set noreorder
  // j
  // nop
  // .set reorder
  if (FillDelaySlot) {
    TOut.emitEmptyDelaySlot(hasShortDelaySlot(Inst), IDLoc, STI);
    TOut.emitDirectiveSetReorder();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2371-2380
```cpp
  if ((Opcode == Mips::JalOneReg || Opcode == Mips::JalTwoReg ||
       ExpandedJalSym) &&
      isPicAndNotNxxAbi()) {
    if (IsCpRestoreSet) {
      // We need a NOP between the JALR and the LW:
      // If .set reorder has been used, we've already emitted a NOP.
      // If .set noreorder has been used, we need to emit a NOP at this point.
      if (!AssemblerOptions.back()->isReorder())
        TOut.emitEmptyDelaySlot(hasShortDelaySlot(Inst), IDLoc,
                                STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2382-2386
```cpp
      // Load the $gp from the stack.
      TOut.emitGPRestore(CpRestoreOffset, IDLoc, STI);
    } else
      Warning(IDLoc, "no .cprestore used in PIC mode");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2388-2389
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2391-2400
```cpp
void MipsAsmParser::onEndOfFile() {
  MipsTargetStreamer &TOut = getTargetStreamer();
  SMLoc IDLoc = SMLoc();
  // If has pending forbidden slot, fill nop and recover reorder.
  if (CurForbiddenSlotAttr) {
    TOut.emitEmptyDelaySlot(false, IDLoc, STI);
    if (AssemblerOptions.back()->isReorder())
      TOut.emitDirectiveSetReorder();
  }
}
```
- EN: Implements `MipsAsmParser::onEndOfFile`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::onEndOfFile`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2402-2416
```cpp
MipsAsmParser::MacroExpanderResultTy
MipsAsmParser::tryExpandInstruction(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                    const MCSubtargetInfo *STI) {
  switch (Inst.getOpcode()) {
  default:
    return MER_NotAMacro;
  case Mips::LoadImm32:
    return expandLoadImm(Inst, true, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::LoadImm64:
    return expandLoadImm(Inst, false, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::LoadAddrImm32:
  case Mips::LoadAddrImm64:
    assert(Inst.getOperand(0).isReg() && "expected register operand kind");
    assert((Inst.getOperand(1).isImm() || Inst.getOperand(1).isExpr()) &&
           "expected immediate operand kind");
```
- EN: Implements `MipsAsmParser::tryExpandInstruction`, a target-specific routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::tryExpandInstruction`，它是一个围绕MC streamer 交互展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2418-2428
```cpp
    return expandLoadAddress(
               Inst.getOperand(0).getReg(), MCRegister(), Inst.getOperand(1),
               Inst.getOpcode() == Mips::LoadAddrImm32, IDLoc, Out, STI)
               ? MER_Fail
               : MER_Success;
  case Mips::LoadAddrReg32:
  case Mips::LoadAddrReg64:
    assert(Inst.getOperand(0).isReg() && "expected register operand kind");
    assert(Inst.getOperand(1).isReg() && "expected register operand kind");
    assert((Inst.getOperand(2).isImm() || Inst.getOperand(2).isExpr()) &&
           "expected immediate operand kind");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2430-2444
```cpp
    return expandLoadAddress(Inst.getOperand(0).getReg(),
                             Inst.getOperand(1).getReg(), Inst.getOperand(2),
                             Inst.getOpcode() == Mips::LoadAddrReg32, IDLoc,
                             Out, STI)
               ? MER_Fail
               : MER_Success;
  case Mips::B_MM_Pseudo:
  case Mips::B_MMR6_Pseudo:
    return expandUncondBranchMMPseudo(Inst, IDLoc, Out, STI) ? MER_Fail
                                                             : MER_Success;
  case Mips::SWM_MM:
  case Mips::LWM_MM:
    return expandLoadStoreMultiple(Inst, IDLoc, Out, STI) ? MER_Fail
                                                          : MER_Success;
  case Mips::JalOneReg:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2445-2459
```cpp
  case Mips::JalTwoReg:
    return expandJalWithRegs(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::BneImm:
  case Mips::BeqImm:
  case Mips::BEQLImmMacro:
  case Mips::BNELImmMacro:
    return expandBranchImm(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::BLT:
  case Mips::BLE:
  case Mips::BGE:
  case Mips::BGT:
  case Mips::BLTU:
  case Mips::BLEU:
  case Mips::BGEU:
  case Mips::BGTU:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2460-2474
```cpp
  case Mips::BLTL:
  case Mips::BLEL:
  case Mips::BGEL:
  case Mips::BGTL:
  case Mips::BLTUL:
  case Mips::BLEUL:
  case Mips::BGEUL:
  case Mips::BGTUL:
  case Mips::BLTImmMacro:
  case Mips::BLEImmMacro:
  case Mips::BGEImmMacro:
  case Mips::BGTImmMacro:
  case Mips::BLTUImmMacro:
  case Mips::BLEUImmMacro:
  case Mips::BGEUImmMacro:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2475-2489
```cpp
  case Mips::BGTUImmMacro:
  case Mips::BLTLImmMacro:
  case Mips::BLELImmMacro:
  case Mips::BGELImmMacro:
  case Mips::BGTLImmMacro:
  case Mips::BLTULImmMacro:
  case Mips::BLEULImmMacro:
  case Mips::BGEULImmMacro:
  case Mips::BGTULImmMacro:
    return expandCondBranches(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::SDivMacro:
  case Mips::SDivIMacro:
  case Mips::SRemMacro:
  case Mips::SRemIMacro:
    return expandDivRem(Inst, IDLoc, Out, STI, false, true) ? MER_Fail
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2490-2504
```cpp
                                                            : MER_Success;
  case Mips::DSDivMacro:
  case Mips::DSDivIMacro:
  case Mips::DSRemMacro:
  case Mips::DSRemIMacro:
    return expandDivRem(Inst, IDLoc, Out, STI, true, true) ? MER_Fail
                                                           : MER_Success;
  case Mips::UDivMacro:
  case Mips::UDivIMacro:
  case Mips::URemMacro:
  case Mips::URemIMacro:
    return expandDivRem(Inst, IDLoc, Out, STI, false, false) ? MER_Fail
                                                             : MER_Success;
  case Mips::DUDivMacro:
  case Mips::DUDivIMacro:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2505-2517
```cpp
  case Mips::DURemMacro:
  case Mips::DURemIMacro:
    return expandDivRem(Inst, IDLoc, Out, STI, true, false) ? MER_Fail
                                                            : MER_Success;
  case Mips::PseudoTRUNC_W_S:
    return expandTrunc(Inst, false, false, IDLoc, Out, STI) ? MER_Fail
                                                            : MER_Success;
  case Mips::PseudoTRUNC_W_D32:
    return expandTrunc(Inst, true, false, IDLoc, Out, STI) ? MER_Fail
                                                           : MER_Success;
  case Mips::PseudoTRUNC_W_D:
    return expandTrunc(Inst, true, true, IDLoc, Out, STI) ? MER_Fail
                                                          : MER_Success;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2519-2533
```cpp
  case Mips::LoadImmSingleGPR:
    return expandLoadSingleImmToGPR(Inst, IDLoc, Out, STI) ? MER_Fail
                                                           : MER_Success;
  case Mips::LoadImmSingleFGR:
    return expandLoadSingleImmToFPR(Inst, IDLoc, Out, STI) ? MER_Fail
                                                           : MER_Success;
  case Mips::LoadImmDoubleGPR:
    return expandLoadDoubleImmToGPR(Inst, IDLoc, Out, STI) ? MER_Fail
                                                           : MER_Success;
  case Mips::LoadImmDoubleFGR:
    return expandLoadDoubleImmToFPR(Inst, true, IDLoc, Out, STI) ? MER_Fail
                                                                 : MER_Success;
  case Mips::LoadImmDoubleFGR_32:
    return expandLoadDoubleImmToFPR(Inst, false, IDLoc, Out, STI) ? MER_Fail
                                                                  : MER_Success;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2535-2549
```cpp
  case Mips::Ulh:
    return expandUlh(Inst, true, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::Ulhu:
    return expandUlh(Inst, false, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::Ush:
    return expandUsh(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::Ulw:
  case Mips::Usw:
    return expandUxw(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::NORImm:
  case Mips::NORImm64:
    return expandAliasImmediate(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::SGE:
  case Mips::SGEU:
    return expandSge(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2550-2564
```cpp
  case Mips::SGEImm:
  case Mips::SGEUImm:
  case Mips::SGEImm64:
  case Mips::SGEUImm64:
    return expandSgeImm(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::SGTImm:
  case Mips::SGTUImm:
  case Mips::SGTImm64:
  case Mips::SGTUImm64:
    return expandSgtImm(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::SLE:
  case Mips::SLEU:
    return expandSle(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::SLEImm:
  case Mips::SLEUImm:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2565-2579
```cpp
  case Mips::SLEImm64:
  case Mips::SLEUImm64:
    return expandSleImm(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::SLTImm64:
    if (isInt<16>(Inst.getOperand(2).getImm())) {
      Inst.setOpcode(Mips::SLTi64);
      return MER_NotAMacro;
    }
    return expandAliasImmediate(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::SLTUImm64:
    if (isInt<16>(Inst.getOperand(2).getImm())) {
      Inst.setOpcode(Mips::SLTiu64);
      return MER_NotAMacro;
    }
    return expandAliasImmediate(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2580-2594
```cpp
  case Mips::ADDi:   case Mips::ADDi_MM:
  case Mips::ADDiu:  case Mips::ADDiu_MM:
  case Mips::SLTi:   case Mips::SLTi_MM:
  case Mips::SLTiu:  case Mips::SLTiu_MM:
    if ((Inst.getNumOperands() == 3) && Inst.getOperand(0).isReg() &&
        Inst.getOperand(1).isReg() && Inst.getOperand(2).isImm()) {
      int64_t ImmValue = Inst.getOperand(2).getImm();
      if (isInt<16>(ImmValue))
        return MER_NotAMacro;
      return expandAliasImmediate(Inst, IDLoc, Out, STI) ? MER_Fail
                                                         : MER_Success;
    }
    return MER_NotAMacro;
  case Mips::ANDi:  case Mips::ANDi_MM:  case Mips::ANDi64:
  case Mips::ORi:   case Mips::ORi_MM:   case Mips::ORi64:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2595-2609
```cpp
  case Mips::XORi:  case Mips::XORi_MM:  case Mips::XORi64:
    if ((Inst.getNumOperands() == 3) && Inst.getOperand(0).isReg() &&
        Inst.getOperand(1).isReg() && Inst.getOperand(2).isImm()) {
      int64_t ImmValue = Inst.getOperand(2).getImm();
      if (isUInt<16>(ImmValue))
        return MER_NotAMacro;
      return expandAliasImmediate(Inst, IDLoc, Out, STI) ? MER_Fail
                                                         : MER_Success;
    }
    return MER_NotAMacro;
  case Mips::ROL:
  case Mips::ROR:
    return expandRotation(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::ROLImm:
  case Mips::RORImm:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2610-2624
```cpp
    return expandRotationImm(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::DROL:
  case Mips::DROR:
    return expandDRotation(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::DROLImm:
  case Mips::DRORImm:
    return expandDRotationImm(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::ABSMacro:
    return expandAbs(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::MULImmMacro:
  case Mips::DMULImmMacro:
    return expandMulImm(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::MULOMacro:
  case Mips::DMULOMacro:
    return expandMulO(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
```
- EN: Declares `expandRotationImm`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandRotationImm`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2625-2639
```cpp
  case Mips::MULOUMacro:
  case Mips::DMULOUMacro:
    return expandMulOU(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::DMULMacro:
    return expandDMULMacro(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::LDMacro:
  case Mips::SDMacro:
    return expandLoadStoreDMacro(Inst, IDLoc, Out, STI,
                                 Inst.getOpcode() == Mips::LDMacro)
               ? MER_Fail
               : MER_Success;
  case Mips::SDC1_M1:
    return expandStoreDM1Macro(Inst, IDLoc, Out, STI)
               ? MER_Fail
               : MER_Success;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2640-2654
```cpp
  case Mips::SEQMacro:
    return expandSeq(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::SEQIMacro:
    return expandSeqI(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::SNEMacro:
    return expandSne(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::SNEIMacro:
    return expandSneI(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::MFTC0:   case Mips::MTTC0:
  case Mips::MFTGPR:  case Mips::MTTGPR:
  case Mips::MFTLO:   case Mips::MTTLO:
  case Mips::MFTHI:   case Mips::MTTHI:
  case Mips::MFTACX:  case Mips::MTTACX:
  case Mips::MFTDSP:  case Mips::MTTDSP:
  case Mips::MFTC1:   case Mips::MTTC1:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2655-2662
```cpp
  case Mips::MFTHC1:  case Mips::MTTHC1:
  case Mips::CFTC1:   case Mips::CTTC1:
    return expandMXTRAlias(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  case Mips::SaaAddr:
  case Mips::SaadAddr:
    return expandSaaAddr(Inst, IDLoc, Out, STI) ? MER_Fail : MER_Success;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2664-2667
```cpp
bool MipsAsmParser::expandJalWithRegs(MCInst &Inst, SMLoc IDLoc,
                                      MCStreamer &Out,
                                      const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandJalWithRegs`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandJalWithRegs`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2669-2673
```cpp
  // Create a JALR instruction which is going to replace the pseudo-JAL.
  MCInst JalrInst;
  JalrInst.setLoc(IDLoc);
  const MCOperand FirstRegOp = Inst.getOperand(0);
  const unsigned Opcode = Inst.getOpcode();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2675-2689
```cpp
  if (Opcode == Mips::JalOneReg) {
    // jal $rs => jalr $rs
    if (IsCpRestoreSet && inMicroMipsMode()) {
      JalrInst.setOpcode(Mips::JALRS16_MM);
      JalrInst.addOperand(FirstRegOp);
    } else if (inMicroMipsMode()) {
      JalrInst.setOpcode(hasMips32r6() ? Mips::JALRC16_MMR6 : Mips::JALR16_MM);
      JalrInst.addOperand(FirstRegOp);
    } else {
      JalrInst.setOpcode(Mips::JALR);
      JalrInst.addOperand(MCOperand::createReg(Mips::RA));
      JalrInst.addOperand(FirstRegOp);
    }
  } else if (Opcode == Mips::JalTwoReg) {
    // jal $rd, $rs => jalr $rd, $rs
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2690-2698
```cpp
    if (IsCpRestoreSet && inMicroMipsMode())
      JalrInst.setOpcode(Mips::JALRS_MM);
    else
      JalrInst.setOpcode(inMicroMipsMode() ? Mips::JALR_MM : Mips::JALR);
    JalrInst.addOperand(FirstRegOp);
    const MCOperand SecondRegOp = Inst.getOperand(1);
    JalrInst.addOperand(SecondRegOp);
  }
  Out.emitInstruction(JalrInst, *STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2700-2705
```cpp
  // If .set reorder is active and branch instruction has a delay slot,
  // emit a NOP after it.
  const MCInstrDesc &MCID = MII.get(JalrInst.getOpcode());
  if (MCID.hasDelaySlot() && AssemblerOptions.back()->isReorder())
    TOut.emitEmptyDelaySlot(hasShortDelaySlot(JalrInst), IDLoc,
                            STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2707-2708
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2710-2713
```cpp
/// Can the value be represented by a unsigned N-bit value and a shift left?
template <unsigned N> static bool isShiftedUIntAtAnyPosition(uint64_t x) {
  return x && isUInt<N>(x >> llvm::countr_zero(x));
}
```
- EN: Implements `isShiftedUIntAtAnyPosition`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isShiftedUIntAtAnyPosition`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2715-2729
```cpp
/// Load (or add) an immediate into a register.
///
/// @param ImmValue     The immediate to load.
/// @param DstReg       The register that will hold the immediate.
/// @param SrcReg       A register to add to the immediate or MCRegister()
///                     for a simple initialization.
/// @param Is32BitImm   Is ImmValue 32-bit or 64-bit?
/// @param IsAddress    True if the immediate represents an address. False if it
///                     is an integer.
/// @param IDLoc        Location of the immediate in the source file.
bool MipsAsmParser::loadImmediate(int64_t ImmValue, MCRegister DstReg,
                                  MCRegister SrcReg, bool Is32BitImm,
                                  bool IsAddress, SMLoc IDLoc, MCStreamer &Out,
                                  const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::loadImmediate`, a target-specific routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::loadImmediate`，它是一个围绕MC streamer 交互展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2731-2734
```cpp
  if (!Is32BitImm && !isGP64bit()) {
    Error(IDLoc, "instruction requires a 64-bit architecture");
    return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2736-2746
```cpp
  if (Is32BitImm) {
    if (isInt<32>(ImmValue) || isUInt<32>(ImmValue)) {
      // Sign extend up to 64-bit so that the predicates match the hardware
      // behaviour. In particular, isInt<16>(0xffff8000) and similar should be
      // true.
      ImmValue = SignExtend64<32>(ImmValue);
    } else {
      Error(IDLoc, "instruction requires a 32-bit immediate");
      return true;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2748-2749
```cpp
  MCRegister ZeroReg = IsAddress ? ABI.GetNullPtr() : ABI.GetZeroReg();
  unsigned AdduOp = !Is32BitImm ? Mips::DADDu : Mips::ADDu;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2751-2753
```cpp
  bool UseSrcReg = false;
  if (SrcReg)
    UseSrcReg = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2755-2764
```cpp
  MCRegister TmpReg = DstReg;
  if (UseSrcReg &&
      getContext().getRegisterInfo()->isSuperOrSubRegisterEq(DstReg, SrcReg)) {
    // At this point we need AT to perform the expansions and we exit if it is
    // not available.
    MCRegister ATReg = getATReg(IDLoc);
    if (!ATReg)
      return true;
    TmpReg = ATReg;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2766-2768
```cpp
  if (isInt<16>(ImmValue)) {
    if (!UseSrcReg)
      SrcReg = ZeroReg;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2770-2776
```cpp
    // This doesn't quite follow the usual ABI expectations for N32 but matches
    // traditional assembler behaviour. N32 would normally use addiu for both
    // integers and addresses.
    if (IsAddress && !Is32BitImm) {
      TOut.emitRRI(Mips::DADDiu, DstReg, SrcReg, ImmValue, IDLoc, STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2778-2780
```cpp
    TOut.emitRRI(Mips::ADDiu, DstReg, SrcReg, ImmValue, IDLoc, STI);
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2782-2788
```cpp
  if (isUInt<16>(ImmValue)) {
    MCRegister TmpReg = DstReg;
    if (SrcReg == DstReg) {
      TmpReg = getATReg(IDLoc);
      if (!TmpReg)
        return true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2790-2794
```cpp
    TOut.emitRRI(Mips::ORi, TmpReg, ZeroReg, ImmValue, IDLoc, STI);
    if (UseSrcReg)
      TOut.emitRRR(ABI.GetPtrAdduOp(), DstReg, TmpReg, SrcReg, IDLoc, STI);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2796-2797
```cpp
  if (isInt<32>(ImmValue) || isUInt<32>(ImmValue)) {
    warnIfNoMacro(IDLoc);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2799-2810
```cpp
    uint16_t Bits31To16 = (ImmValue >> 16) & 0xffff;
    uint16_t Bits15To0 = ImmValue & 0xffff;
    if (!Is32BitImm && !isInt<32>(ImmValue)) {
      // Traditional behaviour seems to special case this particular value. It's
      // not clear why other masks are handled differently.
      if (ImmValue == 0xffffffff) {
        TOut.emitRI(Mips::LUi, TmpReg, 0xffff, IDLoc, STI);
        TOut.emitRRI(Mips::DSRL32, TmpReg, TmpReg, 0, IDLoc, STI);
        if (UseSrcReg)
          TOut.emitRRR(AdduOp, DstReg, TmpReg, SrcReg, IDLoc, STI);
        return false;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2812-2821
```cpp
      // Expand to an ORi instead of a LUi to avoid sign-extending into the
      // upper 32 bits.
      TOut.emitRRI(Mips::ORi, TmpReg, ZeroReg, Bits31To16, IDLoc, STI);
      TOut.emitRRI(Mips::DSLL, TmpReg, TmpReg, 16, IDLoc, STI);
      if (Bits15To0)
        TOut.emitRRI(Mips::ORi, TmpReg, TmpReg, Bits15To0, IDLoc, STI);
      if (UseSrcReg)
        TOut.emitRRR(AdduOp, DstReg, TmpReg, SrcReg, IDLoc, STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2823-2829
```cpp
    TOut.emitRI(Mips::LUi, TmpReg, Bits31To16, IDLoc, STI);
    if (Bits15To0)
      TOut.emitRRI(Mips::ORi, TmpReg, TmpReg, Bits15To0, IDLoc, STI);
    if (UseSrcReg)
      TOut.emitRRR(AdduOp, DstReg, TmpReg, SrcReg, IDLoc, STI);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2831-2835
```cpp
  if (isShiftedUIntAtAnyPosition<16>(ImmValue)) {
    if (Is32BitImm) {
      Error(IDLoc, "instruction requires a 32-bit immediate");
      return true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2837-2840
```cpp
    // We've processed ImmValue satisfying isUInt<16> above, so ImmValue must be
    // at least 17-bit wide here.
    unsigned BitWidth = llvm::bit_width((uint64_t)ImmValue);
    assert(BitWidth >= 17 && "ImmValue must be at least 17-bit wide");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2842-2847
```cpp
    // Traditionally, these immediates are shifted as little as possible and as
    // such we align the most significant bit to bit 15 of our temporary.
    unsigned ShiftAmount = BitWidth - 16;
    uint16_t Bits = (ImmValue >> ShiftAmount) & 0xffff;
    TOut.emitRRI(Mips::ORi, TmpReg, ZeroReg, Bits, IDLoc, STI);
    TOut.emitRRI(Mips::DSLL, TmpReg, TmpReg, ShiftAmount, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2849-2850
```cpp
    if (UseSrcReg)
      TOut.emitRRR(AdduOp, DstReg, TmpReg, SrcReg, IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2852-2853
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2855-2855
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2857-2859
```cpp
  // The remaining case is packed with a sequence of dsll and ori with zeros
  // being omitted and any neighbouring dsll's being coalesced.
  // The highest 32-bit's are equivalent to a 32-bit immediate load.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2861-2864
```cpp
  // Load bits 32-63 of ImmValue into bits 0-31 of the temporary register.
  if (loadImmediate(ImmValue >> 32, TmpReg, MCRegister(), true, false, IDLoc,
                    Out, STI))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2866-2870
```cpp
  // Shift and accumulate into the register. If a 16-bit chunk is zero, then
  // skip it and defer the shift to the next chunk.
  unsigned ShiftCarriedForwards = 16;
  for (int BitNum = 16; BitNum >= 0; BitNum -= 16) {
    uint16_t ImmChunk = (ImmValue >> BitNum) & 0xffff;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2872-2876
```cpp
    if (ImmChunk != 0) {
      TOut.emitDSLL(TmpReg, TmpReg, ShiftCarriedForwards, IDLoc, STI);
      TOut.emitRRI(Mips::ORi, TmpReg, TmpReg, ImmChunk, IDLoc, STI);
      ShiftCarriedForwards = 0;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2878-2880
```cpp
    ShiftCarriedForwards += 16;
  }
  ShiftCarriedForwards -= 16;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2882-2884
```cpp
  // Finish any remaining shifts left by trailing zeros.
  if (ShiftCarriedForwards)
    TOut.emitDSLL(TmpReg, TmpReg, ShiftCarriedForwards, IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2886-2887
```cpp
  if (UseSrcReg)
    TOut.emitRRR(AdduOp, DstReg, TmpReg, SrcReg, IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2889-2890
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2892-2897
```cpp
bool MipsAsmParser::expandLoadImm(MCInst &Inst, bool Is32BitImm, SMLoc IDLoc,
                                  MCStreamer &Out, const MCSubtargetInfo *STI) {
  const MCOperand &ImmOp = Inst.getOperand(1);
  assert(ImmOp.isImm() && "expected immediate operand kind");
  const MCOperand &DstRegOp = Inst.getOperand(0);
  assert(DstRegOp.isReg() && "expected register operand kind");
```
- EN: Implements `MipsAsmParser::expandLoadImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandLoadImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2899-2901
```cpp
  if (loadImmediate(ImmOp.getImm(), DstRegOp.getReg(), MCRegister(), Is32BitImm,
                    false, IDLoc, Out, STI))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2903-2904
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2906-2916
```cpp
bool MipsAsmParser::expandLoadAddress(MCRegister DstReg, MCRegister BaseReg,
                                      const MCOperand &Offset,
                                      bool Is32BitAddress, SMLoc IDLoc,
                                      MCStreamer &Out,
                                      const MCSubtargetInfo *STI) {
  // la can't produce a usable address when addresses are 64-bit.
  if (Is32BitAddress && ABI.ArePtrs64bit()) {
    Warning(IDLoc, "la used to load 64-bit address");
    // Continue as if we had 'dla' instead.
    Is32BitAddress = false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2918-2922
```cpp
  // dla requires 64-bit addresses.
  if (!Is32BitAddress && !hasMips3()) {
    Error(IDLoc, "instruction requires a 64-bit architecture");
    return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2924-2926
```cpp
  if (!Offset.isImm())
    return loadAndAddSymbolAddress(Offset.getExpr(), DstReg, BaseReg,
                                   Is32BitAddress, IDLoc, Out, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2928-2931
```cpp
  if (!ABI.ArePtrs64bit()) {
    // Continue as if we had 'la' whether we had 'la' or 'dla'.
    Is32BitAddress = true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2933-2935
```cpp
  return loadImmediate(Offset.getImm(), DstReg, BaseReg, Is32BitAddress, true,
                       IDLoc, Out, STI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2937-2945
```cpp
bool MipsAsmParser::loadAndAddSymbolAddress(const MCExpr *SymExpr,
                                            MCRegister DstReg,
                                            MCRegister SrcReg, bool Is32BitSym,
                                            SMLoc IDLoc, MCStreamer &Out,
                                            const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  bool UseSrcReg =
      SrcReg.isValid() && SrcReg != Mips::ZERO && SrcReg != Mips::ZERO_64;
  warnIfNoMacro(IDLoc);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2947-2956
```cpp
  if (inPicMode()) {
    MCValue Res;
    if (!SymExpr->evaluateAsRelocatable(Res, nullptr)) {
      Error(IDLoc, "expected relocatable expression");
      return true;
    }
    if (Res.getSubSym()) {
      Error(IDLoc, "expected relocatable expression with only one symbol");
      return true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2958-2969
```cpp
    bool IsPtr64 = ABI.ArePtrs64bit();
    bool IsLocalSym =
        Res.getAddSym()->isInSection() || Res.getAddSym()->isTemporary() ||
        (getContext().isELF() &&
         static_cast<const MCSymbolELF *>(Res.getAddSym())->getBinding() ==
             ELF::STB_LOCAL);
    // For O32, "$"-prefixed symbols are recognized as temporary while
    // .L-prefixed symbols are not (InternalSymbolPrefix is "$"). Recognize ".L"
    // manually.
    if (ABI.IsO32() && Res.getAddSym()->getName().starts_with(".L"))
      IsLocalSym = true;
    bool UseXGOT = STI->hasFeature(Mips::FeatureXGOT) && !IsLocalSym;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2971-2985
```cpp
    // The case where the result register is $25 is somewhat special. If the
    // symbol in the final relocation is external and not modified with a
    // constant then we must use R_MIPS_CALL16 instead of R_MIPS_GOT16
    // or R_MIPS_CALL16 instead of R_MIPS_GOT_DISP in 64-bit case.
    if ((DstReg == Mips::T9 || DstReg == Mips::T9_64) && !UseSrcReg &&
        Res.getConstant() == 0 && !IsLocalSym) {
      if (UseXGOT) {
        const MCExpr *CallHiExpr =
            MCSpecifierExpr::create(SymExpr, Mips::S_CALL_HI16, getContext());
        const MCExpr *CallLoExpr =
            MCSpecifierExpr::create(SymExpr, Mips::S_CALL_LO16, getContext());
        TOut.emitRX(Mips::LUi, DstReg, MCOperand::createExpr(CallHiExpr), IDLoc,
                    STI);
        TOut.emitRRR(IsPtr64 ? Mips::DADDu : Mips::ADDu, DstReg, DstReg, GPReg,
                     IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2986-2995
```cpp
        TOut.emitRRX(IsPtr64 ? Mips::LD : Mips::LW, DstReg, DstReg,
                     MCOperand::createExpr(CallLoExpr), IDLoc, STI);
      } else {
        const MCExpr *CallExpr =
            MCSpecifierExpr::create(SymExpr, Mips::S_GOT_CALL, getContext());
        TOut.emitRRX(IsPtr64 ? Mips::LD : Mips::LW, DstReg, GPReg,
                     MCOperand::createExpr(CallExpr), IDLoc, STI);
      }
      return false;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2997-3007
```cpp
    MCRegister TmpReg = DstReg;
    if (UseSrcReg &&
        getContext().getRegisterInfo()->isSuperOrSubRegisterEq(DstReg,
                                                               SrcReg)) {
      // If $rs is the same as $rd, we need to use AT.
      // If it is not available we exit.
      MCRegister ATReg = getATReg(IDLoc);
      if (!ATReg)
        return true;
      TmpReg = ATReg;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3009-3023
```cpp
    // FIXME: In case of N32 / N64 ABI and emabled XGOT, local addresses
    // loaded using R_MIPS_GOT_PAGE / R_MIPS_GOT_OFST pair of relocations.
    // FIXME: Implement XGOT for microMIPS.
    if (UseXGOT) {
      // Loading address from XGOT
      //   External GOT: lui $tmp, %got_hi(symbol)($gp)
      //                 addu $tmp, $tmp, $gp
      //                 lw $tmp, %got_lo(symbol)($tmp)
      //                >addiu $tmp, $tmp, offset
      //                >addiu $rd, $tmp, $rs
      // The addiu's marked with a '>' may be omitted if they are redundant. If
      // this happens then the last instruction must use $rd as the result
      // register.
      const MCExpr *CallHiExpr =
          MCSpecifierExpr::create(SymExpr, Mips::S_GOT_HI16, getContext());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3024-3025
```cpp
      const MCExpr *CallLoExpr = MCSpecifierExpr::create(
          Res.getAddSym(), Mips::S_GOT_LO16, getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3027-3032
```cpp
      TOut.emitRX(Mips::LUi, TmpReg, MCOperand::createExpr(CallHiExpr), IDLoc,
                  STI);
      TOut.emitRRR(IsPtr64 ? Mips::DADDu : Mips::ADDu, TmpReg, TmpReg, GPReg,
                   IDLoc, STI);
      TOut.emitRRX(IsPtr64 ? Mips::LD : Mips::LW, TmpReg, TmpReg,
                   MCOperand::createExpr(CallLoExpr), IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3034-3038
```cpp
      if (Res.getConstant() != 0)
        TOut.emitRRX(IsPtr64 ? Mips::DADDiu : Mips::ADDiu, TmpReg, TmpReg,
                     MCOperand::createExpr(MCConstantExpr::create(
                         Res.getConstant(), getContext())),
                     IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3040-3044
```cpp
      if (UseSrcReg)
        TOut.emitRRR(IsPtr64 ? Mips::DADDu : Mips::ADDu, DstReg, TmpReg, SrcReg,
                     IDLoc, STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3046-3060
```cpp
    const MCSpecifierExpr *GotExpr = nullptr;
    const MCExpr *LoExpr = nullptr;
    if (ABI.IsN32() || ABI.IsN64()) {
      // The remaining cases are:
      //   Small offset: ld $tmp, %got_disp(symbol)($gp)
      //                >daddiu $tmp, $tmp, offset
      //                >daddu $rd, $tmp, $rs
      // The daddiu's marked with a '>' may be omitted if they are redundant. If
      // this happens then the last instruction must use $rd as the result
      // register.
      GotExpr = MCSpecifierExpr::create(Res.getAddSym(), Mips::S_GOT_DISP,
                                        getContext());
      if (Res.getConstant() != 0) {
        // Symbols fully resolve with just the %got_disp(symbol) but we
        // must still account for any offset to the symbol for
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3061-3062
```cpp
        // expressions like symbol+8.
        LoExpr = MCConstantExpr::create(Res.getConstant(), getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3064-3078
```cpp
        // FIXME: Offsets greater than 16 bits are not yet implemented.
        // FIXME: The correct range is a 32-bit sign-extended number.
        if (Res.getConstant() < -0x8000 || Res.getConstant() > 0x7fff) {
          Error(IDLoc, "macro instruction uses large offset, which is not "
                       "currently supported");
          return true;
        }
      }
    } else {
      // The remaining cases are:
      //   External GOT: lw $tmp, %got(symbol)($gp)
      //                >addiu $tmp, $tmp, offset
      //                >addiu $rd, $tmp, $rs
      //   Local GOT:    lw $tmp, %got(symbol+offset)($gp)
      //                 addiu $tmp, $tmp, %lo(symbol+offset)($gp)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3079-3093
```cpp
      //                >addiu $rd, $tmp, $rs
      // The addiu's marked with a '>' may be omitted if they are redundant. If
      // this happens then the last instruction must use $rd as the result
      // register.
      if (IsLocalSym) {
        GotExpr = MCSpecifierExpr::create(SymExpr, Mips::S_GOT, getContext());
        LoExpr = MCSpecifierExpr::create(SymExpr, Mips::S_LO, getContext());
      } else {
        // External symbols fully resolve the symbol with just the %got(symbol)
        // but we must still account for any offset to the symbol for
        // expressions like symbol+8.
        GotExpr =
            MCSpecifierExpr::create(Res.getAddSym(), Mips::S_GOT, getContext());
        if (Res.getConstant() != 0)
          LoExpr = MCConstantExpr::create(Res.getConstant(), getContext());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3094-3095
```cpp
      }
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3097-3098
```cpp
    TOut.emitRRX(IsPtr64 ? Mips::LD : Mips::LW, TmpReg, GPReg,
                 MCOperand::createExpr(GotExpr), IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3100-3102
```cpp
    if (LoExpr)
      TOut.emitRRX(IsPtr64 ? Mips::DADDiu : Mips::ADDiu, TmpReg, TmpReg,
                   MCOperand::createExpr(LoExpr), IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3104-3106
```cpp
    if (UseSrcReg)
      TOut.emitRRR(IsPtr64 ? Mips::DADDu : Mips::ADDu, DstReg, TmpReg, SrcReg,
                   IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3108-3109
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3111-3114
```cpp
  const auto *HiExpr =
      MCSpecifierExpr::create(SymExpr, Mips::S_HI, getContext());
  const auto *LoExpr =
      MCSpecifierExpr::create(SymExpr, Mips::S_LO, getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3116-3123
```cpp
  // This is the 64-bit symbol address expansion.
  if (ABI.ArePtrs64bit() && isGP64bit()) {
    // We need AT for the 64-bit expansion in the cases where the optional
    // source register is the destination register and for the superscalar
    // scheduled form.
    //
    // If it is not available we exit if the destination is the same as the
    // source register.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3125-3128
```cpp
    const auto *HighestExpr =
        MCSpecifierExpr::create(SymExpr, Mips::S_HIGHEST, getContext());
    const auto *HigherExpr =
        MCSpecifierExpr::create(SymExpr, Mips::S_HIGHER, getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3130-3132
```cpp
    bool RdRegIsRsReg =
        UseSrcReg &&
        getContext().getRegisterInfo()->isSuperOrSubRegisterEq(DstReg, SrcReg);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3134-3135
```cpp
    if (canUseATReg() && UseSrcReg && RdRegIsRsReg) {
      MCRegister ATReg = getATReg(IDLoc);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3137-3151
```cpp
      // If $rs is the same as $rd:
      // (d)la $rd, sym($rd) => lui    $at, %highest(sym)
      //                        daddiu $at, $at, %higher(sym)
      //                        dsll   $at, $at, 16
      //                        daddiu $at, $at, %hi(sym)
      //                        dsll   $at, $at, 16
      //                        daddiu $at, $at, %lo(sym)
      //                        daddu  $rd, $at, $rd
      TOut.emitRX(Mips::LUi, ATReg, MCOperand::createExpr(HighestExpr), IDLoc,
                  STI);
      TOut.emitRRX(Mips::DADDiu, ATReg, ATReg,
                   MCOperand::createExpr(HigherExpr), IDLoc, STI);
      TOut.emitRRI(Mips::DSLL, ATReg, ATReg, 16, IDLoc, STI);
      TOut.emitRRX(Mips::DADDiu, ATReg, ATReg, MCOperand::createExpr(HiExpr),
                   IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3152-3155
```cpp
      TOut.emitRRI(Mips::DSLL, ATReg, ATReg, 16, IDLoc, STI);
      TOut.emitRRX(Mips::DADDiu, ATReg, ATReg, MCOperand::createExpr(LoExpr),
                   IDLoc, STI);
      TOut.emitRRR(Mips::DADDu, DstReg, ATReg, SrcReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3157-3159
```cpp
      return false;
    } else if (canUseATReg() && !RdRegIsRsReg && DstReg != getATReg(IDLoc)) {
      MCRegister ATReg = getATReg(IDLoc);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3161-3175
```cpp
      // If the $rs is different from $rd or if $rs isn't specified and we
      // have $at available:
      // (d)la $rd, sym/sym($rs) => lui    $rd, %highest(sym)
      //                            lui    $at, %hi(sym)
      //                            daddiu $rd, $rd, %higher(sym)
      //                            daddiu $at, $at, %lo(sym)
      //                            dsll32 $rd, $rd, 0
      //                            daddu  $rd, $rd, $at
      //                            (daddu  $rd, $rd, $rs)
      //
      // Which is preferred for superscalar issue.
      TOut.emitRX(Mips::LUi, DstReg, MCOperand::createExpr(HighestExpr), IDLoc,
                  STI);
      TOut.emitRX(Mips::LUi, ATReg, MCOperand::createExpr(HiExpr), IDLoc, STI);
      TOut.emitRRX(Mips::DADDiu, DstReg, DstReg,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3176-3182
```cpp
                   MCOperand::createExpr(HigherExpr), IDLoc, STI);
      TOut.emitRRX(Mips::DADDiu, ATReg, ATReg, MCOperand::createExpr(LoExpr),
                   IDLoc, STI);
      TOut.emitRRI(Mips::DSLL32, DstReg, DstReg, 0, IDLoc, STI);
      TOut.emitRRR(Mips::DADDu, DstReg, DstReg, ATReg, IDLoc, STI);
      if (UseSrcReg)
        TOut.emitRRR(Mips::DADDu, DstReg, DstReg, SrcReg, IDLoc, STI);
```
- EN: Declares `MCOperand::createExpr`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MCOperand::createExpr`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3184-3198
```cpp
      return false;
    } else if ((!canUseATReg() && !RdRegIsRsReg) ||
               (canUseATReg() && DstReg == getATReg(IDLoc))) {
      // Otherwise, synthesize the address in the destination register
      // serially:
      // (d)la $rd, sym/sym($rs) => lui    $rd, %highest(sym)
      //                            daddiu $rd, $rd, %higher(sym)
      //                            dsll   $rd, $rd, 16
      //                            daddiu $rd, $rd, %hi(sym)
      //                            dsll   $rd, $rd, 16
      //                            daddiu $rd, $rd, %lo(sym)
      TOut.emitRX(Mips::LUi, DstReg, MCOperand::createExpr(HighestExpr), IDLoc,
                  STI);
      TOut.emitRRX(Mips::DADDiu, DstReg, DstReg,
                   MCOperand::createExpr(HigherExpr), IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3199-3206
```cpp
      TOut.emitRRI(Mips::DSLL, DstReg, DstReg, 16, IDLoc, STI);
      TOut.emitRRX(Mips::DADDiu, DstReg, DstReg,
                   MCOperand::createExpr(HiExpr), IDLoc, STI);
      TOut.emitRRI(Mips::DSLL, DstReg, DstReg, 16, IDLoc, STI);
      TOut.emitRRX(Mips::DADDiu, DstReg, DstReg,
                   MCOperand::createExpr(LoExpr), IDLoc, STI);
      if (UseSrcReg)
        TOut.emitRRR(Mips::DADDu, DstReg, DstReg, SrcReg, IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3208-3218
```cpp
      return false;
    } else {
      // We have a case where SrcReg == DstReg and we don't have $at
      // available. We can't expand this case, so error out appropriately.
      assert(SrcReg == DstReg && !canUseATReg() &&
             "Could have expanded dla but didn't?");
      reportParseError(IDLoc,
                     "pseudo-instruction requires $at, which is not available");
      return true;
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3220-3234
```cpp
  // And now, the 32-bit symbol address expansion:
  // If $rs is the same as $rd:
  // (d)la $rd, sym($rd)     => lui   $at, %hi(sym)
  //                            ori   $at, $at, %lo(sym)
  //                            addu  $rd, $at, $rd
  // Otherwise, if the $rs is different from $rd or if $rs isn't specified:
  // (d)la $rd, sym/sym($rs) => lui   $rd, %hi(sym)
  //                            ori   $rd, $rd, %lo(sym)
  //                            (addu $rd, $rd, $rs)
  MCRegister TmpReg = DstReg;
  if (UseSrcReg &&
      getContext().getRegisterInfo()->isSuperOrSubRegisterEq(DstReg, SrcReg)) {
    // If $rs is the same as $rd, we need to use AT.
    // If it is not available we exit.
    MCRegister ATReg = getATReg(IDLoc);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3235-3238
```cpp
    if (!ATReg)
      return true;
    TmpReg = ATReg;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3240-3242
```cpp
  TOut.emitRX(Mips::LUi, TmpReg, MCOperand::createExpr(HiExpr), IDLoc, STI);
  TOut.emitRRX(Mips::ADDiu, TmpReg, TmpReg, MCOperand::createExpr(LoExpr),
               IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3244-3248
```cpp
  if (UseSrcReg)
    TOut.emitRRR(Mips::ADDu, DstReg, TmpReg, SrcReg, IDLoc, STI);
  else
    assert(
        getContext().getRegisterInfo()->isSuperOrSubRegisterEq(DstReg, TmpReg));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3250-3251
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3253-3267
```cpp
// Each double-precision register DO-D15 overlaps with two of the single
// precision registers F0-F31. As an example, all of the following hold true:
// D0 + 1 == F1, F1 + 1 == D1, F1 + 1 == F2, depending on the context.
static MCRegister nextReg(MCRegister Reg) {
  if (MipsMCRegisterClasses[Mips::FGR32RegClassID].contains(Reg))
    return Reg == (unsigned)Mips::F31 ? (unsigned)Mips::F0 : Reg + 1;
  switch (Reg.id()) {
  default: llvm_unreachable("Unknown register in assembly macro expansion!");
  case Mips::ZERO: return Mips::AT;
  case Mips::AT:   return Mips::V0;
  case Mips::V0:   return Mips::V1;
  case Mips::V1:   return Mips::A0;
  case Mips::A0:   return Mips::A1;
  case Mips::A1:   return Mips::A2;
  case Mips::A2:   return Mips::A3;
```
- EN: Implements `nextReg`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `nextReg`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3268-3282
```cpp
  case Mips::A3:   return Mips::T0;
  case Mips::T0:   return Mips::T1;
  case Mips::T1:   return Mips::T2;
  case Mips::T2:   return Mips::T3;
  case Mips::T3:   return Mips::T4;
  case Mips::T4:   return Mips::T5;
  case Mips::T5:   return Mips::T6;
  case Mips::T6:   return Mips::T7;
  case Mips::T7:   return Mips::S0;
  case Mips::S0:   return Mips::S1;
  case Mips::S1:   return Mips::S2;
  case Mips::S2:   return Mips::S3;
  case Mips::S3:   return Mips::S4;
  case Mips::S4:   return Mips::S5;
  case Mips::S5:   return Mips::S6;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3283-3297
```cpp
  case Mips::S6:   return Mips::S7;
  case Mips::S7:   return Mips::T8;
  case Mips::T8:   return Mips::T9;
  case Mips::T9:   return Mips::K0;
  case Mips::K0:   return Mips::K1;
  case Mips::K1:   return Mips::GP;
  case Mips::GP:   return Mips::SP;
  case Mips::SP:   return Mips::FP;
  case Mips::FP:   return Mips::RA;
  case Mips::RA:   return Mips::ZERO;
  case Mips::D0:   return Mips::F1;
  case Mips::D1:   return Mips::F3;
  case Mips::D2:   return Mips::F5;
  case Mips::D3:   return Mips::F7;
  case Mips::D4:   return Mips::F9;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3298-3310
```cpp
  case Mips::D5:   return Mips::F11;
  case Mips::D6:   return Mips::F13;
  case Mips::D7:   return Mips::F15;
  case Mips::D8:   return Mips::F17;
  case Mips::D9:   return Mips::F19;
  case Mips::D10:   return Mips::F21;
  case Mips::D11:   return Mips::F23;
  case Mips::D12:   return Mips::F25;
  case Mips::D13:   return Mips::F27;
  case Mips::D14:   return Mips::F29;
  case Mips::D15:   return Mips::F31;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3312-3322
```cpp
// FIXME: This method is too general. In principle we should compute the number
// of instructions required to synthesize the immediate inline compared to
// synthesizing the address inline and relying on non .text sections.
// For static O32 and N32 this may yield a small benefit, for static N64 this is
// likely to yield a much larger benefit as we have to synthesize a 64bit
// address to load a 64 bit value.
bool MipsAsmParser::emitPartialAddress(MipsTargetStreamer &TOut, SMLoc IDLoc,
                                       MCSymbol *Sym) {
  MCRegister ATReg = getATReg(IDLoc);
  if (!ATReg)
    return true;
```
- EN: Implements `MipsAsmParser::emitPartialAddress`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::emitPartialAddress`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3324-3327
```cpp
  if(IsPicEnabled) {
    const MCExpr *GotSym = MCSymbolRefExpr::create(Sym, getContext());
    const auto *GotExpr =
        MCSpecifierExpr::create(GotSym, Mips::S_GOT, getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3329-3339
```cpp
    if(isABI_O32() || isABI_N32()) {
      TOut.emitRRX(Mips::LW, ATReg, GPReg, MCOperand::createExpr(GotExpr),
                   IDLoc, STI);
    } else { //isABI_N64()
      TOut.emitRRX(Mips::LD, ATReg, GPReg, MCOperand::createExpr(GotExpr),
                   IDLoc, STI);
    }
  } else { //!IsPicEnabled
    const MCExpr *HiSym = MCSymbolRefExpr::create(Sym, getContext());
    const auto *HiExpr =
        MCSpecifierExpr::create(HiSym, Mips::S_HI, getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3341-3355
```cpp
    // FIXME: This is technically correct but gives a different result to gas,
    // but gas is incomplete there (it has a fixme noting it doesn't work with
    // 64-bit addresses).
    // FIXME: With -msym32 option, the address expansion for N64 should probably
    // use the O32 / N32 case. It's safe to use the 64 address expansion as the
    // symbol's value is considered sign extended.
    if(isABI_O32() || isABI_N32()) {
      TOut.emitRX(Mips::LUi, ATReg, MCOperand::createExpr(HiExpr), IDLoc, STI);
    } else { //isABI_N64()
      const MCExpr *HighestSym = MCSymbolRefExpr::create(Sym, getContext());
      const auto *HighestExpr =
          MCSpecifierExpr::create(HighestSym, Mips::S_HIGHEST, getContext());
      const MCExpr *HigherSym = MCSymbolRefExpr::create(Sym, getContext());
      const auto *HigherExpr =
          MCSpecifierExpr::create(HigherSym, Mips::S_HIGHER, getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3357-3368
```cpp
      TOut.emitRX(Mips::LUi, ATReg, MCOperand::createExpr(HighestExpr), IDLoc,
                  STI);
      TOut.emitRRX(Mips::DADDiu, ATReg, ATReg,
                   MCOperand::createExpr(HigherExpr), IDLoc, STI);
      TOut.emitRRI(Mips::DSLL, ATReg, ATReg, 16, IDLoc, STI);
      TOut.emitRRX(Mips::DADDiu, ATReg, ATReg, MCOperand::createExpr(HiExpr),
                   IDLoc, STI);
      TOut.emitRRI(Mips::DSLL, ATReg, ATReg, 16, IDLoc, STI);
    }
  }
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3370-3378
```cpp
static uint64_t convertIntToDoubleImm(uint64_t ImmOp64) {
  // If ImmOp64 is AsmToken::Integer type (all bits set to zero in the
  // exponent field), convert it to double (e.g. 1 to 1.0)
  if ((Hi_32(ImmOp64) & 0x7ff00000) == 0) {
    APFloat RealVal(APFloat::IEEEdouble(), ImmOp64);
    ImmOp64 = RealVal.bitcastToAPInt().getZExtValue();
  }
  return ImmOp64;
}
```
- EN: Implements `convertIntToDoubleImm`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `convertIntToDoubleImm`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3380-3386
```cpp
static uint32_t covertDoubleImmToSingleImm(uint64_t ImmOp64) {
  // Conversion of a double in an uint64_t to a float in a uint32_t,
  // retaining the bit pattern of a float.
  double DoubleImm = llvm::bit_cast<double>(ImmOp64);
  float TmpFloat = static_cast<float>(DoubleImm);
  return llvm::bit_cast<uint32_t>(TmpFloat);
}
```
- EN: Implements `covertDoubleImmToSingleImm`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `covertDoubleImmToSingleImm`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3388-3393
```cpp
bool MipsAsmParser::expandLoadSingleImmToGPR(MCInst &Inst, SMLoc IDLoc,
                                             MCStreamer &Out,
                                             const MCSubtargetInfo *STI) {
  assert(Inst.getNumOperands() == 2 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() && Inst.getOperand(1).isImm() &&
         "Invalid instruction operand.");
```
- EN: Implements `MipsAsmParser::expandLoadSingleImmToGPR`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandLoadSingleImmToGPR`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3395-3396
```cpp
  MCRegister FirstReg = Inst.getOperand(0).getReg();
  uint64_t ImmOp64 = Inst.getOperand(1).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3398-3398
```cpp
  uint32_t ImmOp32 = covertDoubleImmToSingleImm(convertIntToDoubleImm(ImmOp64));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3400-3402
```cpp
  return loadImmediate(ImmOp32, FirstReg, MCRegister(), true, false, IDLoc, Out,
                       STI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3404-3410
```cpp
bool MipsAsmParser::expandLoadSingleImmToFPR(MCInst &Inst, SMLoc IDLoc,
                                             MCStreamer &Out,
                                             const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  assert(Inst.getNumOperands() == 2 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() && Inst.getOperand(1).isImm() &&
         "Invalid instruction operand.");
```
- EN: Implements `MipsAsmParser::expandLoadSingleImmToFPR`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandLoadSingleImmToFPR`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3412-3413
```cpp
  MCRegister FirstReg = Inst.getOperand(0).getReg();
  uint64_t ImmOp64 = Inst.getOperand(1).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3415-3415
```cpp
  ImmOp64 = convertIntToDoubleImm(ImmOp64);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3417-3417
```cpp
  uint32_t ImmOp32 = covertDoubleImmToSingleImm(ImmOp64);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3419-3424
```cpp
  MCRegister TmpReg = Mips::ZERO;
  if (ImmOp32 != 0) {
    TmpReg = getATReg(IDLoc);
    if (!TmpReg)
      return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3426-3432
```cpp
  if (Lo_32(ImmOp64) == 0) {
    if (TmpReg != Mips::ZERO && loadImmediate(ImmOp32, TmpReg, MCRegister(),
                                              true, false, IDLoc, Out, STI))
      return true;
    TOut.emitRR(Mips::MTC1, FirstReg, TmpReg, IDLoc, STI);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3434-3438
```cpp
  MCSection *CS = getStreamer().getCurrentSectionOnly();
  // FIXME: Enhance this expansion to use the .lit4 & .lit8 sections
  // where appropriate.
  MCSection *ReadOnlySection =
      getContext().getELFSection(".rodata", ELF::SHT_PROGBITS, ELF::SHF_ALLOC);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3440-3442
```cpp
  MCSymbol *Sym = getContext().createTempSymbol();
  const MCExpr *LoSym = MCSymbolRefExpr::create(Sym, getContext());
  const auto *LoExpr = MCSpecifierExpr::create(LoSym, Mips::S_LO, getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3444-3447
```cpp
  getStreamer().switchSection(ReadOnlySection);
  getStreamer().emitLabel(Sym, IDLoc);
  getStreamer().emitInt32(ImmOp32);
  getStreamer().switchSection(CS);
```
- EN: Declares `getStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3449-3454
```cpp
  if (emitPartialAddress(TOut, IDLoc, Sym))
    return true;
  TOut.emitRRX(Mips::LWC1, FirstReg, TmpReg, MCOperand::createExpr(LoExpr),
               IDLoc, STI);
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3456-3462
```cpp
bool MipsAsmParser::expandLoadDoubleImmToGPR(MCInst &Inst, SMLoc IDLoc,
                                             MCStreamer &Out,
                                             const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  assert(Inst.getNumOperands() == 2 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() && Inst.getOperand(1).isImm() &&
         "Invalid instruction operand.");
```
- EN: Implements `MipsAsmParser::expandLoadDoubleImmToGPR`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandLoadDoubleImmToGPR`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3464-3465
```cpp
  MCRegister FirstReg = Inst.getOperand(0).getReg();
  uint64_t ImmOp64 = Inst.getOperand(1).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3467-3467
```cpp
  ImmOp64 = convertIntToDoubleImm(ImmOp64);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3469-3477
```cpp
  if (Lo_32(ImmOp64) == 0) {
    if (isGP64bit()) {
      if (loadImmediate(ImmOp64, FirstReg, MCRegister(), false, false, IDLoc,
                        Out, STI))
        return true;
    } else {
      if (loadImmediate(Hi_32(ImmOp64), FirstReg, MCRegister(), true, false,
                        IDLoc, Out, STI))
        return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3479-3484
```cpp
      if (loadImmediate(0, nextReg(FirstReg), MCRegister(), true, false, IDLoc,
                        Out, STI))
        return true;
    }
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3486-3488
```cpp
  MCSection *CS = getStreamer().getCurrentSectionOnly();
  MCSection *ReadOnlySection =
      getContext().getELFSection(".rodata", ELF::SHT_PROGBITS, ELF::SHF_ALLOC);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3490-3492
```cpp
  MCSymbol *Sym = getContext().createTempSymbol();
  const MCExpr *LoSym = MCSymbolRefExpr::create(Sym, getContext());
  const auto *LoExpr = MCSpecifierExpr::create(LoSym, Mips::S_LO, getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3494-3498
```cpp
  getStreamer().switchSection(ReadOnlySection);
  getStreamer().emitLabel(Sym, IDLoc);
  getStreamer().emitValueToAlignment(Align(8));
  getStreamer().emitIntValue(ImmOp64, 8);
  getStreamer().switchSection(CS);
```
- EN: Declares `getStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3500-3502
```cpp
  MCRegister TmpReg = getATReg(IDLoc);
  if (!TmpReg)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3504-3505
```cpp
  if (emitPartialAddress(TOut, IDLoc, Sym))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3507-3508
```cpp
  TOut.emitRRX(isABI_N64() ? Mips::DADDiu : Mips::ADDiu, TmpReg, TmpReg,
               MCOperand::createExpr(LoExpr), IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3510-3517
```cpp
  if (isGP64bit())
    TOut.emitRRI(Mips::LD, FirstReg, TmpReg, 0, IDLoc, STI);
  else {
    TOut.emitRRI(Mips::LW, FirstReg, TmpReg, 0, IDLoc, STI);
    TOut.emitRRI(Mips::LW, nextReg(FirstReg), TmpReg, 4, IDLoc, STI);
  }
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3519-3525
```cpp
bool MipsAsmParser::expandLoadDoubleImmToFPR(MCInst &Inst, bool Is64FPU,
                                             SMLoc IDLoc, MCStreamer &Out,
                                             const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  assert(Inst.getNumOperands() == 2 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() && Inst.getOperand(1).isImm() &&
         "Invalid instruction operand.");
```
- EN: Implements `MipsAsmParser::expandLoadDoubleImmToFPR`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandLoadDoubleImmToFPR`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3527-3528
```cpp
  MCRegister FirstReg = Inst.getOperand(0).getReg();
  uint64_t ImmOp64 = Inst.getOperand(1).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3530-3530
```cpp
  ImmOp64 = convertIntToDoubleImm(ImmOp64);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3532-3537
```cpp
  MCRegister TmpReg = Mips::ZERO;
  if (ImmOp64 != 0) {
    TmpReg = getATReg(IDLoc);
    if (!TmpReg)
      return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3539-3547
```cpp
  if ((Lo_32(ImmOp64) == 0) &&
      !((Hi_32(ImmOp64) & 0xffff0000) && (Hi_32(ImmOp64) & 0x0000ffff))) {
    if (isGP64bit()) {
      if (TmpReg != Mips::ZERO && loadImmediate(ImmOp64, TmpReg, MCRegister(),
                                                false, false, IDLoc, Out, STI))
        return true;
      TOut.emitRR(Mips::DMTC1, FirstReg, TmpReg, IDLoc, STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3549-3552
```cpp
    if (TmpReg != Mips::ZERO &&
        loadImmediate(Hi_32(ImmOp64), TmpReg, MCRegister(), true, false, IDLoc,
                      Out, STI))
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3554-3562
```cpp
    if (hasMips32r2()) {
      TOut.emitRR(Mips::MTC1, FirstReg, Mips::ZERO, IDLoc, STI);
      TOut.emitRRR(Mips::MTHC1_D32, FirstReg, FirstReg, TmpReg, IDLoc, STI);
    } else {
      TOut.emitRR(Mips::MTC1, nextReg(FirstReg), TmpReg, IDLoc, STI);
      TOut.emitRR(Mips::MTC1, FirstReg, Mips::ZERO, IDLoc, STI);
    }
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3564-3568
```cpp
  MCSection *CS = getStreamer().getCurrentSectionOnly();
  // FIXME: Enhance this expansion to use the .lit4 & .lit8 sections
  // where appropriate.
  MCSection *ReadOnlySection =
      getContext().getELFSection(".rodata", ELF::SHT_PROGBITS, ELF::SHF_ALLOC);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3570-3572
```cpp
  MCSymbol *Sym = getContext().createTempSymbol();
  const MCExpr *LoSym = MCSymbolRefExpr::create(Sym, getContext());
  const auto *LoExpr = MCSpecifierExpr::create(LoSym, Mips::S_LO, getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3574-3578
```cpp
  getStreamer().switchSection(ReadOnlySection);
  getStreamer().emitLabel(Sym, IDLoc);
  getStreamer().emitValueToAlignment(Align(8));
  getStreamer().emitIntValue(ImmOp64, 8);
  getStreamer().switchSection(CS);
```
- EN: Declares `getStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3580-3581
```cpp
  if (emitPartialAddress(TOut, IDLoc, Sym))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3583-3584
```cpp
  TOut.emitRRX(Is64FPU ? Mips::LDC164 : Mips::LDC1, FirstReg, TmpReg,
               MCOperand::createExpr(LoExpr), IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3586-3587
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3589-3592
```cpp
bool MipsAsmParser::expandUncondBranchMMPseudo(MCInst &Inst, SMLoc IDLoc,
                                               MCStreamer &Out,
                                               const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandUncondBranchMMPseudo`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandUncondBranchMMPseudo`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3594-3595
```cpp
  assert(MII.get(Inst.getOpcode()).getNumOperands() == 1 &&
         "unexpected number of operands");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3597-3611
```cpp
  MCOperand Offset = Inst.getOperand(0);
  if (Offset.isExpr()) {
    Inst.clear();
    Inst.setOpcode(Mips::BEQ_MM);
    Inst.addOperand(MCOperand::createReg(Mips::ZERO));
    Inst.addOperand(MCOperand::createReg(Mips::ZERO));
    Inst.addOperand(MCOperand::createExpr(Offset.getExpr()));
  } else {
    assert(Offset.isImm() && "expected immediate operand kind");
    if (isInt<11>(Offset.getImm())) {
      // If offset fits into 11 bits then this instruction becomes microMIPS
      // 16-bit unconditional branch instruction.
      if (inMicroMipsMode())
        Inst.setOpcode(hasMips32r6() ? Mips::BC16_MMR6 : Mips::B16_MM);
    } else {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3612-3623
```cpp
      if (!isInt<17>(Offset.getImm()))
        return Error(IDLoc, "branch target out of range");
      if (offsetToAlignment(Offset.getImm(), Align(2)))
        return Error(IDLoc, "branch to misaligned address");
      Inst.clear();
      Inst.setOpcode(Mips::BEQ_MM);
      Inst.addOperand(MCOperand::createReg(Mips::ZERO));
      Inst.addOperand(MCOperand::createReg(Mips::ZERO));
      Inst.addOperand(MCOperand::createImm(Offset.getImm()));
    }
  }
  Out.emitInstruction(Inst, *STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3625-3629
```cpp
  // If .set reorder is active and branch instruction has a delay slot,
  // emit a NOP after it.
  const MCInstrDesc &MCID = MII.get(Inst.getOpcode());
  if (MCID.hasDelaySlot() && AssemblerOptions.back()->isReorder())
    TOut.emitEmptyDelaySlot(true, IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3631-3632
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3634-3638
```cpp
bool MipsAsmParser::expandBranchImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                    const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  const MCOperand &DstRegOp = Inst.getOperand(0);
  assert(DstRegOp.isReg() && "expected register operand kind");
```
- EN: Implements `MipsAsmParser::expandBranchImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandBranchImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3640-3641
```cpp
  const MCOperand &ImmOp = Inst.getOperand(1);
  assert(ImmOp.isImm() && "expected immediate operand kind");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3643-3645
```cpp
  const MCOperand &MemOffsetOp = Inst.getOperand(2);
  assert((MemOffsetOp.isImm() || MemOffsetOp.isExpr()) &&
         "expected immediate or expression operand");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3647-3647
```cpp
  bool IsLikely = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3649-3663
```cpp
  unsigned OpCode = 0;
  switch(Inst.getOpcode()) {
    case Mips::BneImm:
      OpCode = Mips::BNE;
      break;
    case Mips::BeqImm:
      OpCode = Mips::BEQ;
      break;
    case Mips::BEQLImmMacro:
      OpCode = Mips::BEQL;
      IsLikely = true;
      break;
    case Mips::BNELImmMacro:
      OpCode = Mips::BNEL;
      IsLikely = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3664-3668
```cpp
      break;
    default:
      llvm_unreachable("Unknown immediate branch pseudo-instruction.");
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3670-3680
```cpp
  int64_t ImmValue = ImmOp.getImm();
  if (ImmValue == 0) {
    if (IsLikely) {
      TOut.emitRRX(OpCode, DstRegOp.getReg(), Mips::ZERO,
                   MCOperand::createExpr(MemOffsetOp.getExpr()), IDLoc, STI);
      TOut.emitRRI(Mips::SLL, Mips::ZERO, Mips::ZERO, 0, IDLoc, STI);
    } else
      TOut.emitRRX(OpCode, DstRegOp.getReg(), Mips::ZERO, MemOffsetOp, IDLoc,
              STI);
  } else {
    warnIfNoMacro(IDLoc);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3682-3684
```cpp
    MCRegister ATReg = getATReg(IDLoc);
    if (!ATReg)
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3686-3688
```cpp
    if (loadImmediate(ImmValue, ATReg, MCRegister(), !isGP64bit(), true, IDLoc,
                      Out, STI))
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3690-3698
```cpp
    if (IsLikely && MemOffsetOp.isExpr()) {
      TOut.emitRRX(OpCode, DstRegOp.getReg(), ATReg,
              MCOperand::createExpr(MemOffsetOp.getExpr()), IDLoc, STI);
      TOut.emitRRI(Mips::SLL, Mips::ZERO, Mips::ZERO, 0, IDLoc, STI);
    } else
      TOut.emitRRX(OpCode, DstRegOp.getReg(), ATReg, MemOffsetOp, IDLoc, STI);
  }
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3700-3704
```cpp
void MipsAsmParser::expandMem16Inst(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                    const MCSubtargetInfo *STI, bool IsLoad) {
  unsigned NumOp = Inst.getNumOperands();
  assert((NumOp == 3 || NumOp == 4) && "unexpected operands number");
  unsigned StartOp = NumOp == 3 ? 0 : 1;
```
- EN: Implements `MipsAsmParser::expandMem16Inst`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandMem16Inst`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3706-3710
```cpp
  const MCOperand &DstRegOp = Inst.getOperand(StartOp);
  assert(DstRegOp.isReg() && "expected register operand kind");
  const MCOperand &BaseRegOp = Inst.getOperand(StartOp + 1);
  assert(BaseRegOp.isReg() && "expected register operand kind");
  const MCOperand &OffsetOp = Inst.getOperand(StartOp + 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3712-3716
```cpp
  MipsTargetStreamer &TOut = getTargetStreamer();
  unsigned OpCode = Inst.getOpcode();
  MCRegister DstReg = DstRegOp.getReg();
  MCRegister BaseReg = BaseRegOp.getReg();
  MCRegister TmpReg = DstReg;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3718-3725
```cpp
  const MCInstrDesc &Desc = MII.get(OpCode);
  int16_t DstRegClass =
      MII.getOpRegClassID(Desc.operands()[StartOp],
                          STI->getHwMode(MCSubtargetInfo::HwMode_RegInfo));
  unsigned DstRegClassID =
      getContext().getRegisterInfo()->getRegClass(DstRegClass).getID();
  bool IsGPR = (DstRegClassID == Mips::GPR32RegClassID) ||
               (DstRegClassID == Mips::GPR64RegClassID);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3727-3733
```cpp
  if (!IsLoad || !IsGPR || (BaseReg == DstReg)) {
    // At this point we need AT to perform the expansions
    // and we exit if it is not available.
    TmpReg = getATReg(IDLoc);
    if (!TmpReg)
      return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3735-3740
```cpp
  auto emitInstWithOffset = [&](const MCOperand &Off) {
    if (NumOp == 3)
      TOut.emitRRX(OpCode, DstReg, TmpReg, Off, IDLoc, STI);
    else
      TOut.emitRRRX(OpCode, DstReg, DstReg, TmpReg, Off, IDLoc, STI);
  };
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3742-3744
```cpp
  if (OffsetOp.isImm()) {
    int64_t LoOffset = OffsetOp.getImm() & 0xffff;
    int64_t HiOffset = OffsetOp.getImm() & ~0xffff;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3746-3749
```cpp
    // If msb of LoOffset is 1(negative number) we must increment
    // HiOffset to account for the sign-extension of the low part.
    if (LoOffset & 0x8000)
      HiOffset += 0x10000;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3751-3751
```cpp
    bool IsLargeOffset = HiOffset != 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3753-3758
```cpp
    if (IsLargeOffset) {
      bool Is32BitImm = isInt<32>(OffsetOp.getImm());
      if (loadImmediate(HiOffset, TmpReg, MCRegister(), Is32BitImm, true, IDLoc,
                        Out, STI))
        return;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3760-3765
```cpp
    if (BaseReg != Mips::ZERO && BaseReg != Mips::ZERO_64)
      TOut.emitRRR(ABI.ArePtrs64bit() ? Mips::DADDu : Mips::ADDu, TmpReg,
                   TmpReg, BaseReg, IDLoc, STI);
    emitInstWithOffset(MCOperand::createImm(int16_t(LoOffset)));
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3767-3781
```cpp
  if (OffsetOp.isExpr()) {
    if (inPicMode()) {
      // FIXME:
      // c) Check that immediates of R_MIPS_GOT16/R_MIPS_LO16 relocations
      //    do not exceed 16-bit.
      // d) Use R_MIPS_GOT_PAGE/R_MIPS_GOT_OFST relocations instead
      //    of R_MIPS_GOT_DISP in appropriate cases to reduce number
      //    of GOT entries.
      MCValue Res;
      if (!OffsetOp.getExpr()->evaluateAsRelocatable(Res, nullptr)) {
        Error(IDLoc, "expected relocatable expression");
        return;
      }
      if (Res.getSubSym()) {
        Error(IDLoc, "expected relocatable expression with only one symbol");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3782-3783
```cpp
        return;
      }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3785-3799
```cpp
      loadAndAddSymbolAddress(
          MCSymbolRefExpr::create(Res.getAddSym(), getContext()), TmpReg,
          BaseReg, !ABI.ArePtrs64bit(), IDLoc, Out, STI);
      emitInstWithOffset(MCOperand::createImm(int16_t(Res.getConstant())));
    } else {
      // FIXME: Implement 64-bit case.
      // 1) lw $8, sym => lui $8,  %hi(sym)
      //                  lw  $8,  %lo(sym)($8)
      // 2) sw $8, sym => lui $at, %hi(sym)
      //                  sw  $8,  %lo(sym)($at)
      const MCExpr *OffExpr = OffsetOp.getExpr();
      MCOperand LoOperand = MCOperand::createExpr(
          MCSpecifierExpr::create(OffExpr, Mips::S_LO, getContext()));
      MCOperand HiOperand = MCOperand::createExpr(
          MCSpecifierExpr::create(OffExpr, Mips::S_HI, getContext()));
```
- EN: Declares `MCSymbolRefExpr::create`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MCSymbolRefExpr::create`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3801-3805
```cpp
      if (ABI.IsN64()) {
        MCOperand HighestOperand = MCOperand::createExpr(
            MCSpecifierExpr::create(OffExpr, Mips::S_HIGHEST, getContext()));
        MCOperand HigherOperand = MCOperand::createExpr(
            MCSpecifierExpr::create(OffExpr, Mips::S_HIGHER, getContext()));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3807-3821
```cpp
        TOut.emitRX(Mips::LUi, TmpReg, HighestOperand, IDLoc, STI);
        TOut.emitRRX(Mips::DADDiu, TmpReg, TmpReg, HigherOperand, IDLoc, STI);
        TOut.emitRRI(Mips::DSLL, TmpReg, TmpReg, 16, IDLoc, STI);
        TOut.emitRRX(Mips::DADDiu, TmpReg, TmpReg, HiOperand, IDLoc, STI);
        TOut.emitRRI(Mips::DSLL, TmpReg, TmpReg, 16, IDLoc, STI);
        if (BaseReg != Mips::ZERO && BaseReg != Mips::ZERO_64)
          TOut.emitRRR(Mips::DADDu, TmpReg, TmpReg, BaseReg, IDLoc, STI);
        emitInstWithOffset(LoOperand);
      } else {
        // Generate the base address in TmpReg.
        TOut.emitRX(Mips::LUi, TmpReg, HiOperand, IDLoc, STI);
        if (BaseReg != Mips::ZERO)
          TOut.emitRRR(Mips::ADDu, TmpReg, TmpReg, BaseReg, IDLoc, STI);
        // Emit the load or store with the adjusted base and offset.
        emitInstWithOffset(LoOperand);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3822-3825
```cpp
      }
    }
    return;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3827-3828
```cpp
  llvm_unreachable("unexpected operand type");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3830-3834
```cpp
void MipsAsmParser::expandMem9Inst(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                   const MCSubtargetInfo *STI, bool IsLoad) {
  unsigned NumOp = Inst.getNumOperands();
  assert((NumOp == 3 || NumOp == 4) && "unexpected operands number");
  unsigned StartOp = NumOp == 3 ? 0 : 1;
```
- EN: Implements `MipsAsmParser::expandMem9Inst`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandMem9Inst`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3836-3840
```cpp
  const MCOperand &DstRegOp = Inst.getOperand(StartOp);
  assert(DstRegOp.isReg() && "expected register operand kind");
  const MCOperand &BaseRegOp = Inst.getOperand(StartOp + 1);
  assert(BaseRegOp.isReg() && "expected register operand kind");
  const MCOperand &OffsetOp = Inst.getOperand(StartOp + 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3842-3846
```cpp
  MipsTargetStreamer &TOut = getTargetStreamer();
  unsigned OpCode = Inst.getOpcode();
  MCRegister DstReg = DstRegOp.getReg();
  MCRegister BaseReg = BaseRegOp.getReg();
  MCRegister TmpReg = DstReg;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3848-3851
```cpp
  const MCInstrDesc &Desc = MII.get(OpCode);
  int16_t DstRegClass =
      MII.getOpRegClassID(Desc.operands()[StartOp],
                          STI->getHwMode(MCSubtargetInfo::HwMode_RegInfo));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3853-3856
```cpp
  unsigned DstRegClassID =
      getContext().getRegisterInfo()->getRegClass(DstRegClass).getID();
  bool IsGPR = (DstRegClassID == Mips::GPR32RegClassID) ||
               (DstRegClassID == Mips::GPR64RegClassID);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3858-3864
```cpp
  if (!IsLoad || !IsGPR || (BaseReg == DstReg)) {
    // At this point we need AT to perform the expansions
    // and we exit if it is not available.
    TmpReg = getATReg(IDLoc);
    if (!TmpReg)
      return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3866-3872
```cpp
  auto emitInst = [&]() {
    if (NumOp == 3)
      TOut.emitRRX(OpCode, DstReg, TmpReg, MCOperand::createImm(0), IDLoc, STI);
    else
      TOut.emitRRRX(OpCode, DstReg, DstReg, TmpReg, MCOperand::createImm(0),
                    IDLoc, STI);
  };
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3874-3879
```cpp
  if (OffsetOp.isImm()) {
    loadImmediate(OffsetOp.getImm(), TmpReg, BaseReg, !ABI.ArePtrs64bit(), true,
                  IDLoc, Out, STI);
    emitInst();
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3881-3886
```cpp
  if (OffsetOp.isExpr()) {
    loadAndAddSymbolAddress(OffsetOp.getExpr(), TmpReg, BaseReg,
                            !ABI.ArePtrs64bit(), IDLoc, Out, STI);
    emitInst();
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3888-3889
```cpp
  llvm_unreachable("unexpected operand type");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3891-3896
```cpp
bool MipsAsmParser::expandLoadStoreMultiple(MCInst &Inst, SMLoc IDLoc,
                                            MCStreamer &Out,
                                            const MCSubtargetInfo *STI) {
  unsigned OpNum = Inst.getNumOperands();
  unsigned Opcode = Inst.getOpcode();
  unsigned NewOpcode = Opcode == Mips::SWM_MM ? Mips::SWM32_MM : Mips::LWM32_MM;
```
- EN: Implements `MipsAsmParser::expandLoadStoreMultiple`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandLoadStoreMultiple`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3898-3900
```cpp
  assert(Inst.getOperand(OpNum - 1).isImm() &&
         Inst.getOperand(OpNum - 2).isReg() &&
         Inst.getOperand(OpNum - 3).isReg() && "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3902-3913
```cpp
  if (OpNum < 8 && Inst.getOperand(OpNum - 1).getImm() <= 60 &&
      Inst.getOperand(OpNum - 1).getImm() >= 0 &&
      (Inst.getOperand(OpNum - 2).getReg() == Mips::SP ||
       Inst.getOperand(OpNum - 2).getReg() == Mips::SP_64) &&
      (Inst.getOperand(OpNum - 3).getReg() == Mips::RA ||
       Inst.getOperand(OpNum - 3).getReg() == Mips::RA_64)) {
    // It can be implemented as SWM16 or LWM16 instruction.
    if (inMicroMipsMode() && hasMips32r6())
      NewOpcode = Opcode == Mips::SWM_MM ? Mips::SWM16_MMR6 : Mips::LWM16_MMR6;
    else
      NewOpcode = Opcode == Mips::SWM_MM ? Mips::SWM16_MM : Mips::LWM16_MM;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3915-3918
```cpp
  Inst.setOpcode(NewOpcode);
  Out.emitInstruction(Inst, *STI);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3920-3928
```cpp
bool MipsAsmParser::expandCondBranches(MCInst &Inst, SMLoc IDLoc,
                                       MCStreamer &Out,
                                       const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  bool EmittedNoMacroWarning = false;
  unsigned PseudoOpcode = Inst.getOpcode();
  MCRegister SrcReg = Inst.getOperand(0).getReg();
  const MCOperand &TrgOp = Inst.getOperand(1);
  const MCExpr *OffsetExpr = Inst.getOperand(2).getExpr();
```
- EN: Implements `MipsAsmParser::expandCondBranches`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandCondBranches`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3930-3931
```cpp
  unsigned ZeroSrcOpcode, ZeroTrgOpcode;
  bool ReverseOrderSLT, IsUnsigned, IsLikely, AcceptsEquality;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3933-3938
```cpp
  MCRegister TrgReg;
  if (TrgOp.isReg())
    TrgReg = TrgOp.getReg();
  else if (TrgOp.isImm()) {
    warnIfNoMacro(IDLoc);
    EmittedNoMacroWarning = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3940-3942
```cpp
    TrgReg = getATReg(IDLoc);
    if (!TrgReg)
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3944-3958
```cpp
    switch(PseudoOpcode) {
    default:
      llvm_unreachable("unknown opcode for branch pseudo-instruction");
    case Mips::BLTImmMacro:
      PseudoOpcode = Mips::BLT;
      break;
    case Mips::BLEImmMacro:
      PseudoOpcode = Mips::BLE;
      break;
    case Mips::BGEImmMacro:
      PseudoOpcode = Mips::BGE;
      break;
    case Mips::BGTImmMacro:
      PseudoOpcode = Mips::BGT;
      break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3959-3973
```cpp
    case Mips::BLTUImmMacro:
      PseudoOpcode = Mips::BLTU;
      break;
    case Mips::BLEUImmMacro:
      PseudoOpcode = Mips::BLEU;
      break;
    case Mips::BGEUImmMacro:
      PseudoOpcode = Mips::BGEU;
      break;
    case Mips::BGTUImmMacro:
      PseudoOpcode = Mips::BGTU;
      break;
    case Mips::BLTLImmMacro:
      PseudoOpcode = Mips::BLTL;
      break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3974-3988
```cpp
    case Mips::BLELImmMacro:
      PseudoOpcode = Mips::BLEL;
      break;
    case Mips::BGELImmMacro:
      PseudoOpcode = Mips::BGEL;
      break;
    case Mips::BGTLImmMacro:
      PseudoOpcode = Mips::BGTL;
      break;
    case Mips::BLTULImmMacro:
      PseudoOpcode = Mips::BLTUL;
      break;
    case Mips::BLEULImmMacro:
      PseudoOpcode = Mips::BLEUL;
      break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3989-3995
```cpp
    case Mips::BGEULImmMacro:
      PseudoOpcode = Mips::BGEUL;
      break;
    case Mips::BGTULImmMacro:
      PseudoOpcode = Mips::BGTUL;
      break;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3997-4000
```cpp
    if (loadImmediate(TrgOp.getImm(), TrgReg, MCRegister(), !isGP64bit(), false,
                      IDLoc, Out, STI))
      return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4002-4016
```cpp
  switch (PseudoOpcode) {
  case Mips::BLT:
  case Mips::BLTU:
  case Mips::BLTL:
  case Mips::BLTUL:
    AcceptsEquality = false;
    ReverseOrderSLT = false;
    IsUnsigned =
        ((PseudoOpcode == Mips::BLTU) || (PseudoOpcode == Mips::BLTUL));
    IsLikely = ((PseudoOpcode == Mips::BLTL) || (PseudoOpcode == Mips::BLTUL));
    ZeroSrcOpcode = Mips::BGTZ;
    ZeroTrgOpcode = Mips::BLTZ;
    break;
  case Mips::BLE:
  case Mips::BLEU:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4017-4031
```cpp
  case Mips::BLEL:
  case Mips::BLEUL:
    AcceptsEquality = true;
    ReverseOrderSLT = true;
    IsUnsigned =
        ((PseudoOpcode == Mips::BLEU) || (PseudoOpcode == Mips::BLEUL));
    IsLikely = ((PseudoOpcode == Mips::BLEL) || (PseudoOpcode == Mips::BLEUL));
    ZeroSrcOpcode = Mips::BGEZ;
    ZeroTrgOpcode = Mips::BLEZ;
    break;
  case Mips::BGE:
  case Mips::BGEU:
  case Mips::BGEL:
  case Mips::BGEUL:
    AcceptsEquality = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4032-4046
```cpp
    ReverseOrderSLT = false;
    IsUnsigned =
        ((PseudoOpcode == Mips::BGEU) || (PseudoOpcode == Mips::BGEUL));
    IsLikely = ((PseudoOpcode == Mips::BGEL) || (PseudoOpcode == Mips::BGEUL));
    ZeroSrcOpcode = Mips::BLEZ;
    ZeroTrgOpcode = Mips::BGEZ;
    break;
  case Mips::BGT:
  case Mips::BGTU:
  case Mips::BGTL:
  case Mips::BGTUL:
    AcceptsEquality = false;
    ReverseOrderSLT = true;
    IsUnsigned =
        ((PseudoOpcode == Mips::BGTU) || (PseudoOpcode == Mips::BGTUL));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4047-4053
```cpp
    IsLikely = ((PseudoOpcode == Mips::BGTL) || (PseudoOpcode == Mips::BGTUL));
    ZeroSrcOpcode = Mips::BLTZ;
    ZeroTrgOpcode = Mips::BGTZ;
    break;
  default:
    llvm_unreachable("unknown opcode for branch pseudo-instruction");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4055-4069
```cpp
  bool IsTrgRegZero = (TrgReg == Mips::ZERO);
  bool IsSrcRegZero = (SrcReg == Mips::ZERO);
  if (IsSrcRegZero && IsTrgRegZero) {
    // FIXME: All of these Opcode-specific if's are needed for compatibility
    // with GAS' behaviour. However, they may not generate the most efficient
    // code in some circumstances.
    if (PseudoOpcode == Mips::BLT) {
      TOut.emitRX(Mips::BLTZ, Mips::ZERO, MCOperand::createExpr(OffsetExpr),
                  IDLoc, STI);
      return false;
    }
    if (PseudoOpcode == Mips::BLE) {
      TOut.emitRX(Mips::BLEZ, Mips::ZERO, MCOperand::createExpr(OffsetExpr),
                  IDLoc, STI);
      Warning(IDLoc, "branch is always taken");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4070-4084
```cpp
      return false;
    }
    if (PseudoOpcode == Mips::BGE) {
      TOut.emitRX(Mips::BGEZ, Mips::ZERO, MCOperand::createExpr(OffsetExpr),
                  IDLoc, STI);
      Warning(IDLoc, "branch is always taken");
      return false;
    }
    if (PseudoOpcode == Mips::BGT) {
      TOut.emitRX(Mips::BGTZ, Mips::ZERO, MCOperand::createExpr(OffsetExpr),
                  IDLoc, STI);
      return false;
    }
    if (PseudoOpcode == Mips::BGTU) {
      TOut.emitRRX(Mips::BNE, Mips::ZERO, Mips::ZERO,
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4085-4099
```cpp
                   MCOperand::createExpr(OffsetExpr), IDLoc, STI);
      return false;
    }
    if (AcceptsEquality) {
      // If both registers are $0 and the pseudo-branch accepts equality, it
      // will always be taken, so we emit an unconditional branch.
      TOut.emitRRX(Mips::BEQ, Mips::ZERO, Mips::ZERO,
                   MCOperand::createExpr(OffsetExpr), IDLoc, STI);
      Warning(IDLoc, "branch is always taken");
      return false;
    }
    // If both registers are $0 and the pseudo-branch does not accept
    // equality, it will never be taken, so we don't have to emit anything.
    return false;
  }
```
- EN: Implements `MCOperand::createExpr`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MCOperand::createExpr`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4100-4114
```cpp
  if (IsSrcRegZero || IsTrgRegZero) {
    if ((IsSrcRegZero && PseudoOpcode == Mips::BGTU) ||
        (IsTrgRegZero && PseudoOpcode == Mips::BLTU)) {
      // If the $rs is $0 and the pseudo-branch is BGTU (0 > x) or
      // if the $rt is $0 and the pseudo-branch is BLTU (x < 0),
      // the pseudo-branch will never be taken, so we don't emit anything.
      // This only applies to unsigned pseudo-branches.
      return false;
    }
    if ((IsSrcRegZero && PseudoOpcode == Mips::BLEU) ||
        (IsTrgRegZero && PseudoOpcode == Mips::BGEU)) {
      // If the $rs is $0 and the pseudo-branch is BLEU (0 <= x) or
      // if the $rt is $0 and the pseudo-branch is BGEU (x >= 0),
      // the pseudo-branch will always be taken, so we emit an unconditional
      // branch.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4115-4129
```cpp
      // This only applies to unsigned pseudo-branches.
      TOut.emitRRX(Mips::BEQ, Mips::ZERO, Mips::ZERO,
                   MCOperand::createExpr(OffsetExpr), IDLoc, STI);
      Warning(IDLoc, "branch is always taken");
      return false;
    }
    if (IsUnsigned) {
      // If the $rs is $0 and the pseudo-branch is BLTU (0 < x) or
      // if the $rt is $0 and the pseudo-branch is BGTU (x > 0),
      // the pseudo-branch will be taken only when the non-zero register is
      // different from 0, so we emit a BNEZ.
      //
      // If the $rs is $0 and the pseudo-branch is BGEU (0 >= x) or
      // if the $rt is $0 and the pseudo-branch is BLEU (x <= 0),
      // the pseudo-branch will be taken only when the non-zero register is
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4130-4144
```cpp
      // equal to 0, so we emit a BEQZ.
      //
      // Because only BLEU and BGEU branch on equality, we can use the
      // AcceptsEquality variable to decide when to emit the BEQZ.
      TOut.emitRRX(AcceptsEquality ? Mips::BEQ : Mips::BNE,
                   IsSrcRegZero ? TrgReg : SrcReg, Mips::ZERO,
                   MCOperand::createExpr(OffsetExpr), IDLoc, STI);
      return false;
    }
    // If we have a signed pseudo-branch and one of the registers is $0,
    // we can use an appropriate compare-to-zero branch. We select which one
    // to use in the switch statement above.
    TOut.emitRX(IsSrcRegZero ? ZeroSrcOpcode : ZeroTrgOpcode,
                IsSrcRegZero ? TrgReg : SrcReg,
                MCOperand::createExpr(OffsetExpr), IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4145-4146
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4148-4152
```cpp
  // If neither the SrcReg nor the TrgReg are $0, we need AT to perform the
  // expansions. If it is not available, we return.
  MCRegister ATRegNum = getATReg(IDLoc);
  if (!ATRegNum)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4154-4155
```cpp
  if (!EmittedNoMacroWarning)
    warnIfNoMacro(IDLoc);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4157-4171
```cpp
  // SLT fits well with 2 of our 4 pseudo-branches:
  //   BLT, where $rs < $rt, translates into "slt $at, $rs, $rt" and
  //   BGT, where $rs > $rt, translates into "slt $at, $rt, $rs".
  // If the result of the SLT is 1, we branch, and if it's 0, we don't.
  // This is accomplished by using a BNEZ with the result of the SLT.
  //
  // The other 2 pseudo-branches are opposites of the above 2 (BGE with BLT
  // and BLE with BGT), so we change the BNEZ into a BEQZ.
  // Because only BGE and BLE branch on equality, we can use the
  // AcceptsEquality variable to decide when to emit the BEQZ.
  // Note that the order of the SLT arguments doesn't change between
  // opposites.
  //
  // The same applies to the unsigned variants, except that SLTu is used
  // instead of SLT.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 4172-4174
```cpp
  TOut.emitRRR(IsUnsigned ? Mips::SLTu : Mips::SLT, ATRegNum,
               ReverseOrderSLT ? TrgReg : SrcReg,
               ReverseOrderSLT ? SrcReg : TrgReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4176-4181
```cpp
  TOut.emitRRX(IsLikely ? (AcceptsEquality ? Mips::BEQL : Mips::BNEL)
                        : (AcceptsEquality ? Mips::BEQ : Mips::BNE),
               ATRegNum, Mips::ZERO, MCOperand::createExpr(OffsetExpr), IDLoc,
               STI);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4183-4189
```cpp
// Expand a integer division macro.
//
// Notably we don't have to emit a warning when encountering $rt as the $zero
// register, or 0 as an immediate. processInstruction() has already done that.
//
// The destination register can only be $zero when expanding (S)DivIMacro or
// D(S)DivMacro.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 4191-4194
```cpp
bool MipsAsmParser::expandDivRem(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                 const MCSubtargetInfo *STI,
                                 const bool IsMips64, const bool Signed) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandDivRem`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandDivRem`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4196-4196
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4198-4200
```cpp
  const MCOperand &RdRegOp = Inst.getOperand(0);
  assert(RdRegOp.isReg() && "expected register operand kind");
  MCRegister RdReg = RdRegOp.getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4202-4204
```cpp
  const MCOperand &RsRegOp = Inst.getOperand(1);
  assert(RsRegOp.isReg() && "expected register operand kind");
  MCRegister RsReg = RsRegOp.getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4206-4207
```cpp
  MCRegister RtReg;
  int64_t ImmValue;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4209-4215
```cpp
  const MCOperand &RtOp = Inst.getOperand(2);
  assert((RtOp.isReg() || RtOp.isImm()) &&
         "expected register or immediate operand kind");
  if (RtOp.isReg())
    RtReg = RtOp.getReg();
  else
    ImmValue = RtOp.getImm();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4217-4219
```cpp
  unsigned DivOp;
  unsigned ZeroReg;
  unsigned SubOp;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4221-4229
```cpp
  if (IsMips64) {
    DivOp = Signed ? Mips::DSDIV : Mips::DUDIV;
    ZeroReg = Mips::ZERO_64;
    SubOp = Mips::DSUB;
  } else {
    DivOp = Signed ? Mips::SDIV : Mips::UDIV;
    ZeroReg = Mips::ZERO;
    SubOp = Mips::SUB;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4231-4231
```cpp
  bool UseTraps = useTraps();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4233-4237
```cpp
  unsigned Opcode = Inst.getOpcode();
  bool isDiv = Opcode == Mips::SDivMacro || Opcode == Mips::SDivIMacro ||
               Opcode == Mips::UDivMacro || Opcode == Mips::UDivIMacro ||
               Opcode == Mips::DSDivMacro || Opcode == Mips::DSDivIMacro ||
               Opcode == Mips::DUDivMacro || Opcode == Mips::DUDivIMacro;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4239-4242
```cpp
  bool isRem = Opcode == Mips::SRemMacro || Opcode == Mips::SRemIMacro ||
               Opcode == Mips::URemMacro || Opcode == Mips::URemIMacro ||
               Opcode == Mips::DSRemMacro || Opcode == Mips::DSRemIMacro ||
               Opcode == Mips::DURemMacro || Opcode == Mips::DURemIMacro;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4244-4247
```cpp
  if (RtOp.isImm()) {
    MCRegister ATReg = getATReg(IDLoc);
    if (!ATReg)
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4249-4255
```cpp
    if (!NoZeroDivCheck && ImmValue == 0) {
      if (UseTraps)
        TOut.emitRRI(Mips::TEQ, ZeroReg, ZeroReg, 0x7, IDLoc, STI);
      else
        TOut.emitII(Mips::BREAK, 0x7, 0, IDLoc, STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4257-4271
```cpp
    if (isRem && (ImmValue == 1 || (Signed && (ImmValue == -1)))) {
      TOut.emitRRR(Mips::OR, RdReg, ZeroReg, ZeroReg, IDLoc, STI);
      return false;
    } else if (isDiv && ImmValue == 1) {
      TOut.emitRRR(Mips::OR, RdReg, RsReg, Mips::ZERO, IDLoc, STI);
      return false;
    } else if (isDiv && Signed && ImmValue == -1) {
      TOut.emitRRR(SubOp, RdReg, ZeroReg, RsReg, IDLoc, STI);
      return false;
    } else {
      if (loadImmediate(ImmValue, ATReg, MCRegister(), isInt<32>(ImmValue),
                        false, Inst.getLoc(), Out, STI))
        return true;
      TOut.emitRR(DivOp, RsReg, ATReg, IDLoc, STI);
      TOut.emitR(isDiv ? Mips::MFLO : Mips::MFHI, RdReg, IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4272-4275
```cpp
      return false;
    }
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4277-4288
```cpp
  // If the macro expansion of (d)div(u) or (d)rem(u) would always trap or
  // break, insert the trap/break and exit. This gives a different result to
  // GAS. GAS has an inconsistency/missed optimization in that not all cases
  // are handled equivalently. As the observed behaviour is the same, we're ok.
  if (!NoZeroDivCheck && (RtReg == Mips::ZERO || RtReg == Mips::ZERO_64)) {
    if (UseTraps) {
      TOut.emitRRI(Mips::TEQ, ZeroReg, ZeroReg, 0x7, IDLoc, STI);
      return false;
    }
    TOut.emitII(Mips::BREAK, 0x7, 0, IDLoc, STI);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4290-4295
```cpp
  // (d)rem(u) $0, $X, $Y is a special case. Like div $zero, $X, $Y, it does
  // not expand to macro sequence.
  if (isRem && (RdReg == Mips::ZERO || RdReg == Mips::ZERO_64)) {
    TOut.emitRR(DivOp, RsReg, RtReg, IDLoc, STI);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4297-4300
```cpp
  // Temporary label for first branch traget
  MCContext &Context = TOut.getContext();
  MCSymbol *BrTarget;
  MCOperand LabelOp;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4302-4313
```cpp
  TOut.emitRR(DivOp, RsReg, RtReg, IDLoc, STI);
  if (!NoZeroDivCheck) {
    if (UseTraps) {
      TOut.emitRRI(Mips::TEQ, RtReg, ZeroReg, 0x7, IDLoc, STI);
    } else {
      // Branch to the li instruction.
      BrTarget = Context.createTempSymbol();
      LabelOp =
          MCOperand::createExpr(MCSymbolRefExpr::create(BrTarget, Context));
      TOut.emitRRX(Mips::BNE, RtReg, ZeroReg, LabelOp, IDLoc, STI);
      TOut.emitNop(IDLoc, STI);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4315-4316
```cpp
    if (!UseTraps)
      TOut.emitII(Mips::BREAK, 0x7, 0, IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4318-4320
```cpp
    if (!UseTraps)
      TOut.getStreamer().emitLabel(BrTarget);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4322-4324
```cpp
  TOut.emitR(isDiv ? Mips::MFLO : Mips::MFHI, RdReg, IDLoc, STI);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4326-4329
```cpp
bool MipsAsmParser::expandTrunc(MCInst &Inst, bool IsDouble, bool Is64FPU,
                                SMLoc IDLoc, MCStreamer &Out,
                                const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandTrunc`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandTrunc`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4331-4333
```cpp
  assert(Inst.getNumOperands() == 3 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() && Inst.getOperand(1).isReg() &&
         Inst.getOperand(2).isReg() && "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4335-4337
```cpp
  MCRegister FirstReg = Inst.getOperand(0).getReg();
  MCRegister SecondReg = Inst.getOperand(1).getReg();
  MCRegister ThirdReg = Inst.getOperand(2).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4339-4353
```cpp
  if (hasMips1() && !hasMips2()) {
    MCRegister ATReg = getATReg(IDLoc);
    if (!ATReg)
      return true;
    TOut.emitRR(Mips::CFC1, ThirdReg, Mips::RA, IDLoc, STI);
    TOut.emitRR(Mips::CFC1, ThirdReg, Mips::RA, IDLoc, STI);
    TOut.emitNop(IDLoc, STI);
    TOut.emitRRI(Mips::ORi, ATReg, ThirdReg, 0x3, IDLoc, STI);
    TOut.emitRRI(Mips::XORi, ATReg, ATReg, 0x2, IDLoc, STI);
    TOut.emitRR(Mips::CTC1, Mips::RA, ATReg, IDLoc, STI);
    TOut.emitNop(IDLoc, STI);
    TOut.emitRR(IsDouble ? (Is64FPU ? Mips::CVT_W_D64 : Mips::CVT_W_D32)
                         : Mips::CVT_W_S,
                FirstReg, SecondReg, IDLoc, STI);
    TOut.emitRR(Mips::CTC1, Mips::RA, ThirdReg, IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4354-4356
```cpp
    TOut.emitNop(IDLoc, STI);
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4358-4360
```cpp
  TOut.emitRR(IsDouble ? (Is64FPU ? Mips::TRUNC_W_D64 : Mips::TRUNC_W_D32)
                       : Mips::TRUNC_W_S,
              FirstReg, SecondReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4362-4363
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4365-4369
```cpp
bool MipsAsmParser::expandUlh(MCInst &Inst, bool Signed, SMLoc IDLoc,
                              MCStreamer &Out, const MCSubtargetInfo *STI) {
  if (hasMips32r6() || hasMips64r6()) {
    return Error(IDLoc, "instruction not supported on mips32r6 or mips64r6");
  }
```
- EN: Implements `MipsAsmParser::expandUlh`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandUlh`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4371-4376
```cpp
  const MCOperand &DstRegOp = Inst.getOperand(0);
  assert(DstRegOp.isReg() && "expected register operand kind");
  const MCOperand &SrcRegOp = Inst.getOperand(1);
  assert(SrcRegOp.isReg() && "expected register operand kind");
  const MCOperand &OffsetImmOp = Inst.getOperand(2);
  assert(OffsetImmOp.isImm() && "expected immediate operand kind");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4378-4381
```cpp
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister DstReg = DstRegOp.getReg();
  MCRegister SrcReg = SrcRegOp.getReg();
  int64_t OffsetValue = OffsetImmOp.getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4383-4388
```cpp
  // NOTE: We always need AT for ULHU, as it is always used as the source
  // register for one of the LBu's.
  warnIfNoMacro(IDLoc);
  MCRegister ATReg = getATReg(IDLoc);
  if (!ATReg)
    return true;
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4390-4395
```cpp
  bool IsLargeOffset = !(isInt<16>(OffsetValue + 1) && isInt<16>(OffsetValue));
  if (IsLargeOffset) {
    if (loadImmediate(OffsetValue, ATReg, SrcReg, !ABI.ArePtrs64bit(), true,
                      IDLoc, Out, STI))
      return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4397-4400
```cpp
  int64_t FirstOffset = IsLargeOffset ? 0 : OffsetValue;
  int64_t SecondOffset = IsLargeOffset ? 1 : (OffsetValue + 1);
  if (isLittle())
    std::swap(FirstOffset, SecondOffset);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4402-4403
```cpp
  MCRegister FirstLbuDstReg = IsLargeOffset ? DstReg : ATReg;
  MCRegister SecondLbuDstReg = IsLargeOffset ? ATReg : DstReg;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4405-4406
```cpp
  MCRegister LbuSrcReg = IsLargeOffset ? ATReg : SrcReg;
  MCRegister SllReg = IsLargeOffset ? DstReg : ATReg;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4408-4412
```cpp
  TOut.emitRRI(Signed ? Mips::LB : Mips::LBu, FirstLbuDstReg, LbuSrcReg,
               FirstOffset, IDLoc, STI);
  TOut.emitRRI(Mips::LBu, SecondLbuDstReg, LbuSrcReg, SecondOffset, IDLoc, STI);
  TOut.emitRRI(Mips::SLL, SllReg, SllReg, 8, IDLoc, STI);
  TOut.emitRRR(Mips::OR, DstReg, DstReg, ATReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4414-4415
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4417-4421
```cpp
bool MipsAsmParser::expandUsh(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                              const MCSubtargetInfo *STI) {
  if (hasMips32r6() || hasMips64r6()) {
    return Error(IDLoc, "instruction not supported on mips32r6 or mips64r6");
  }
```
- EN: Implements `MipsAsmParser::expandUsh`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandUsh`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4423-4428
```cpp
  const MCOperand &DstRegOp = Inst.getOperand(0);
  assert(DstRegOp.isReg() && "expected register operand kind");
  const MCOperand &SrcRegOp = Inst.getOperand(1);
  assert(SrcRegOp.isReg() && "expected register operand kind");
  const MCOperand &OffsetImmOp = Inst.getOperand(2);
  assert(OffsetImmOp.isImm() && "expected immediate operand kind");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4430-4433
```cpp
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister DstReg = DstRegOp.getReg();
  MCRegister SrcReg = SrcRegOp.getReg();
  int64_t OffsetValue = OffsetImmOp.getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4435-4438
```cpp
  warnIfNoMacro(IDLoc);
  MCRegister ATReg = getATReg(IDLoc);
  if (!ATReg)
    return true;
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4440-4445
```cpp
  bool IsLargeOffset = !(isInt<16>(OffsetValue + 1) && isInt<16>(OffsetValue));
  if (IsLargeOffset) {
    if (loadImmediate(OffsetValue, ATReg, SrcReg, !ABI.ArePtrs64bit(), true,
                      IDLoc, Out, STI))
      return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4447-4450
```cpp
  int64_t FirstOffset = IsLargeOffset ? 1 : (OffsetValue + 1);
  int64_t SecondOffset = IsLargeOffset ? 0 : OffsetValue;
  if (isLittle())
    std::swap(FirstOffset, SecondOffset);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4452-4463
```cpp
  if (IsLargeOffset) {
    TOut.emitRRI(Mips::SB, DstReg, ATReg, FirstOffset, IDLoc, STI);
    TOut.emitRRI(Mips::SRL, DstReg, DstReg, 8, IDLoc, STI);
    TOut.emitRRI(Mips::SB, DstReg, ATReg, SecondOffset, IDLoc, STI);
    TOut.emitRRI(Mips::LBu, ATReg, ATReg, 0, IDLoc, STI);
    TOut.emitRRI(Mips::SLL, DstReg, DstReg, 8, IDLoc, STI);
    TOut.emitRRR(Mips::OR, DstReg, DstReg, ATReg, IDLoc, STI);
  } else {
    TOut.emitRRI(Mips::SB, DstReg, SrcReg, FirstOffset, IDLoc, STI);
    TOut.emitRRI(Mips::SRL, ATReg, DstReg, 8, IDLoc, STI);
    TOut.emitRRI(Mips::SB, ATReg, SrcReg, SecondOffset, IDLoc, STI);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4465-4466
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4468-4472
```cpp
bool MipsAsmParser::expandUxw(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                              const MCSubtargetInfo *STI) {
  if (hasMips32r6() || hasMips64r6()) {
    return Error(IDLoc, "instruction not supported on mips32r6 or mips64r6");
  }
```
- EN: Implements `MipsAsmParser::expandUxw`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandUxw`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4474-4479
```cpp
  const MCOperand &DstRegOp = Inst.getOperand(0);
  assert(DstRegOp.isReg() && "expected register operand kind");
  const MCOperand &SrcRegOp = Inst.getOperand(1);
  assert(SrcRegOp.isReg() && "expected register operand kind");
  const MCOperand &OffsetImmOp = Inst.getOperand(2);
  assert(OffsetImmOp.isImm() && "expected immediate operand kind");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4481-4484
```cpp
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister DstReg = DstRegOp.getReg();
  MCRegister SrcReg = SrcRegOp.getReg();
  int64_t OffsetValue = OffsetImmOp.getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4486-4491
```cpp
  // Compute left/right load/store offsets.
  bool IsLargeOffset = !(isInt<16>(OffsetValue + 3) && isInt<16>(OffsetValue));
  int64_t LxlOffset = IsLargeOffset ? 0 : OffsetValue;
  int64_t LxrOffset = IsLargeOffset ? 3 : (OffsetValue + 3);
  if (isLittle())
    std::swap(LxlOffset, LxrOffset);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4493-4501
```cpp
  bool IsLoadInst = (Inst.getOpcode() == Mips::Ulw);
  bool DoMove = IsLoadInst && (SrcReg == DstReg) && !IsLargeOffset;
  MCRegister TmpReg = SrcReg;
  if (IsLargeOffset || DoMove) {
    warnIfNoMacro(IDLoc);
    TmpReg = getATReg(IDLoc);
    if (!TmpReg)
      return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4503-4507
```cpp
  if (IsLargeOffset) {
    if (loadImmediate(OffsetValue, TmpReg, SrcReg, !ABI.ArePtrs64bit(), true,
                      IDLoc, Out, STI))
      return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4509-4510
```cpp
  if (DoMove)
    std::swap(DstReg, TmpReg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4512-4515
```cpp
  unsigned XWL = IsLoadInst ? Mips::LWL : Mips::SWL;
  unsigned XWR = IsLoadInst ? Mips::LWR : Mips::SWR;
  TOut.emitRRI(XWL, DstReg, TmpReg, LxlOffset, IDLoc, STI);
  TOut.emitRRI(XWR, DstReg, TmpReg, LxrOffset, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4517-4518
```cpp
  if (DoMove)
    TOut.emitRRR(Mips::OR, TmpReg, DstReg, Mips::ZERO, IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4520-4521
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4523-4525
```cpp
bool MipsAsmParser::expandSge(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                              const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandSge`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandSge`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4527-4530
```cpp
  assert(Inst.getNumOperands() == 3 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() &&
         Inst.getOperand(1).isReg() &&
         Inst.getOperand(2).isReg() && "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4532-4535
```cpp
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  MCRegister OpReg = Inst.getOperand(2).getReg();
  unsigned OpCode;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4537-4537
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4539-4548
```cpp
  switch (Inst.getOpcode()) {
  case Mips::SGE:
    OpCode = Mips::SLT;
    break;
  case Mips::SGEU:
    OpCode = Mips::SLTu;
    break;
  default:
    llvm_unreachable("unexpected 'sge' opcode");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4550-4552
```cpp
  // $SrcReg >= $OpReg is equal to (not ($SrcReg < $OpReg))
  TOut.emitRRR(OpCode, DstReg, SrcReg, OpReg, IDLoc, STI);
  TOut.emitRRI(Mips::XORi, DstReg, DstReg, 1, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4554-4555
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4557-4559
```cpp
bool MipsAsmParser::expandSgeImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                 const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandSgeImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandSgeImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4561-4564
```cpp
  assert(Inst.getNumOperands() == 3 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() &&
         Inst.getOperand(1).isReg() &&
         Inst.getOperand(2).isImm() && "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4566-4569
```cpp
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  int64_t ImmValue = Inst.getOperand(2).getImm();
  unsigned OpRegCode, OpImmCode;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4571-4571
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4573-4586
```cpp
  switch (Inst.getOpcode()) {
  case Mips::SGEImm:
  case Mips::SGEImm64:
    OpRegCode = Mips::SLT;
    OpImmCode = Mips::SLTi;
    break;
  case Mips::SGEUImm:
  case Mips::SGEUImm64:
    OpRegCode = Mips::SLTu;
    OpImmCode = Mips::SLTiu;
    break;
  default:
    llvm_unreachable("unexpected 'sge' opcode with immediate");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4588-4600
```cpp
  // $SrcReg >= Imm is equal to (not ($SrcReg < Imm))
  if (isInt<16>(ImmValue)) {
    // Use immediate version of STL.
    TOut.emitRRI(OpImmCode, DstReg, SrcReg, ImmValue, IDLoc, STI);
    TOut.emitRRI(Mips::XORi, DstReg, DstReg, 1, IDLoc, STI);
  } else {
    MCRegister ImmReg = DstReg;
    if (DstReg == SrcReg) {
      MCRegister ATReg = getATReg(Inst.getLoc());
      if (!ATReg)
        return true;
      ImmReg = ATReg;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4602-4604
```cpp
    if (loadImmediate(ImmValue, ImmReg, MCRegister(), isInt<32>(ImmValue),
                      false, IDLoc, Out, STI))
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4606-4608
```cpp
    TOut.emitRRR(OpRegCode, DstReg, SrcReg, ImmReg, IDLoc, STI);
    TOut.emitRRI(Mips::XORi, DstReg, DstReg, 1, IDLoc, STI);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4610-4611
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4613-4615
```cpp
bool MipsAsmParser::expandSgtImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                 const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandSgtImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandSgtImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4617-4620
```cpp
  assert(Inst.getNumOperands() == 3 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() &&
         Inst.getOperand(1).isReg() &&
         Inst.getOperand(2).isImm() && "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4622-4626
```cpp
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  MCRegister ImmReg = DstReg;
  int64_t ImmValue = Inst.getOperand(2).getImm();
  unsigned OpCode;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4628-4628
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4630-4641
```cpp
  switch (Inst.getOpcode()) {
  case Mips::SGTImm:
  case Mips::SGTImm64:
    OpCode = Mips::SLT;
    break;
  case Mips::SGTUImm:
  case Mips::SGTUImm64:
    OpCode = Mips::SLTu;
    break;
  default:
    llvm_unreachable("unexpected 'sgt' opcode with immediate");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4643-4648
```cpp
  if (DstReg == SrcReg) {
    MCRegister ATReg = getATReg(Inst.getLoc());
    if (!ATReg)
      return true;
    ImmReg = ATReg;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4650-4652
```cpp
  if (loadImmediate(ImmValue, ImmReg, MCRegister(), isInt<32>(ImmValue), false,
                    IDLoc, Out, STI))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4654-4655
```cpp
  // $SrcReg > $ImmReg is equal to $ImmReg < $SrcReg
  TOut.emitRRR(OpCode, DstReg, ImmReg, SrcReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4657-4658
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4660-4662
```cpp
bool MipsAsmParser::expandSle(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                              const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandSle`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandSle`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4664-4667
```cpp
  assert(Inst.getNumOperands() == 3 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() &&
         Inst.getOperand(1).isReg() &&
         Inst.getOperand(2).isReg() && "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4669-4672
```cpp
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  MCRegister OpReg = Inst.getOperand(2).getReg();
  unsigned OpCode;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4674-4674
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4676-4685
```cpp
  switch (Inst.getOpcode()) {
  case Mips::SLE:
    OpCode = Mips::SLT;
    break;
  case Mips::SLEU:
    OpCode = Mips::SLTu;
    break;
  default:
    llvm_unreachable("unexpected 'sge' opcode");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4687-4689
```cpp
  // $SrcReg <= $OpReg is equal to (not ($OpReg < $SrcReg))
  TOut.emitRRR(OpCode, DstReg, OpReg, SrcReg, IDLoc, STI);
  TOut.emitRRI(Mips::XORi, DstReg, DstReg, 1, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4691-4692
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4694-4696
```cpp
bool MipsAsmParser::expandSleImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                 const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandSleImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandSleImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4698-4701
```cpp
  assert(Inst.getNumOperands() == 3 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() &&
         Inst.getOperand(1).isReg() &&
         Inst.getOperand(2).isImm() && "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4703-4706
```cpp
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  int64_t ImmValue = Inst.getOperand(2).getImm();
  unsigned OpRegCode;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4708-4708
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4710-4721
```cpp
  switch (Inst.getOpcode()) {
  case Mips::SLEImm:
  case Mips::SLEImm64:
    OpRegCode = Mips::SLT;
    break;
  case Mips::SLEUImm:
  case Mips::SLEUImm64:
    OpRegCode = Mips::SLTu;
    break;
  default:
    llvm_unreachable("unexpected 'sge' opcode with immediate");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4723-4730
```cpp
  // $SrcReg <= Imm is equal to (not (Imm < $SrcReg))
  MCRegister ImmReg = DstReg;
  if (DstReg == SrcReg) {
    MCRegister ATReg = getATReg(Inst.getLoc());
    if (!ATReg)
      return true;
    ImmReg = ATReg;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4732-4734
```cpp
  if (loadImmediate(ImmValue, ImmReg, MCRegister(), isInt<32>(ImmValue), false,
                    IDLoc, Out, STI))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4736-4737
```cpp
  TOut.emitRRR(OpRegCode, DstReg, ImmReg, SrcReg, IDLoc, STI);
  TOut.emitRRI(Mips::XORi, DstReg, DstReg, 1, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4739-4740
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4742-4745
```cpp
bool MipsAsmParser::expandAliasImmediate(MCInst &Inst, SMLoc IDLoc,
                                         MCStreamer &Out,
                                         const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandAliasImmediate`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandAliasImmediate`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4747-4750
```cpp
  assert(Inst.getNumOperands() == 3 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() &&
         Inst.getOperand(1).isReg() &&
         Inst.getOperand(2).isImm() && "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4752-4756
```cpp
  MCRegister ATReg;
  MCRegister FinalDstReg;
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  int64_t ImmValue = Inst.getOperand(2).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4758-4758
```cpp
  bool Is32Bit = isInt<32>(ImmValue) || (!isGP64bit() && isUInt<32>(ImmValue));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4760-4760
```cpp
  unsigned FinalOpcode = Inst.getOpcode();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4762-4768
```cpp
  if (DstReg == SrcReg) {
    ATReg = getATReg(Inst.getLoc());
    if (!ATReg)
      return true;
    FinalDstReg = DstReg;
    DstReg = ATReg;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4770-4784
```cpp
  if (!loadImmediate(ImmValue, DstReg, MCRegister(), Is32Bit, false,
                     Inst.getLoc(), Out, STI)) {
    switch (FinalOpcode) {
    default:
      llvm_unreachable("unimplemented expansion");
    case Mips::ADDi:
      FinalOpcode = Mips::ADD;
      break;
    case Mips::ADDiu:
      FinalOpcode = Mips::ADDu;
      break;
    case Mips::ANDi:
      FinalOpcode = Mips::AND;
      break;
    case Mips::NORImm:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4785-4799
```cpp
      FinalOpcode = Mips::NOR;
      break;
    case Mips::ORi:
      FinalOpcode = Mips::OR;
      break;
    case Mips::SLTi:
      FinalOpcode = Mips::SLT;
      break;
    case Mips::SLTiu:
      FinalOpcode = Mips::SLTu;
      break;
    case Mips::XORi:
      FinalOpcode = Mips::XOR;
      break;
    case Mips::ADDi_MM:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4800-4814
```cpp
      FinalOpcode = Mips::ADD_MM;
      break;
    case Mips::ADDiu_MM:
      FinalOpcode = Mips::ADDu_MM;
      break;
    case Mips::ANDi_MM:
      FinalOpcode = Mips::AND_MM;
      break;
    case Mips::ORi_MM:
      FinalOpcode = Mips::OR_MM;
      break;
    case Mips::SLTi_MM:
      FinalOpcode = Mips::SLT_MM;
      break;
    case Mips::SLTiu_MM:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4815-4829
```cpp
      FinalOpcode = Mips::SLTu_MM;
      break;
    case Mips::XORi_MM:
      FinalOpcode = Mips::XOR_MM;
      break;
    case Mips::ANDi64:
      FinalOpcode = Mips::AND64;
      break;
    case Mips::NORImm64:
      FinalOpcode = Mips::NOR64;
      break;
    case Mips::ORi64:
      FinalOpcode = Mips::OR64;
      break;
    case Mips::SLTImm64:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4830-4838
```cpp
      FinalOpcode = Mips::SLT64;
      break;
    case Mips::SLTUImm64:
      FinalOpcode = Mips::SLTu64;
      break;
    case Mips::XORi64:
      FinalOpcode = Mips::XOR64;
      break;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4840-4847
```cpp
    if (!FinalDstReg)
      TOut.emitRRR(FinalOpcode, DstReg, DstReg, SrcReg, IDLoc, STI);
    else
      TOut.emitRRR(FinalOpcode, FinalDstReg, FinalDstReg, DstReg, IDLoc, STI);
    return false;
  }
  return true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4849-4856
```cpp
bool MipsAsmParser::expandRotation(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                   const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister ATReg;
  MCRegister DReg = Inst.getOperand(0).getReg();
  MCRegister SReg = Inst.getOperand(1).getReg();
  MCRegister TReg = Inst.getOperand(2).getReg();
  MCRegister TmpReg = DReg;
```
- EN: Implements `MipsAsmParser::expandRotation`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandRotation`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4858-4859
```cpp
  unsigned FirstShift = Mips::NOP;
  unsigned SecondShift = Mips::NOP;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4861-4866
```cpp
  if (hasMips32r2()) {
    if (DReg == SReg) {
      TmpReg = getATReg(Inst.getLoc());
      if (!TmpReg)
        return true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4868-4872
```cpp
    if (Inst.getOpcode() == Mips::ROL) {
      TOut.emitRRR(Mips::SUBu, TmpReg, Mips::ZERO, TReg, Inst.getLoc(), STI);
      TOut.emitRRR(Mips::ROTRV, DReg, SReg, TmpReg, Inst.getLoc(), STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4874-4877
```cpp
    if (Inst.getOpcode() == Mips::ROR) {
      TOut.emitRRR(Mips::ROTRV, DReg, SReg, TReg, Inst.getLoc(), STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4879-4880
```cpp
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4882-4894
```cpp
  if (hasMips32()) {
    switch (Inst.getOpcode()) {
    default:
      llvm_unreachable("unexpected instruction opcode");
    case Mips::ROL:
      FirstShift = Mips::SRLV;
      SecondShift = Mips::SLLV;
      break;
    case Mips::ROR:
      FirstShift = Mips::SLLV;
      SecondShift = Mips::SRLV;
      break;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4896-4898
```cpp
    ATReg = getATReg(Inst.getLoc());
    if (!ATReg)
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4900-4903
```cpp
    TOut.emitRRR(Mips::SUBu, ATReg, Mips::ZERO, TReg, Inst.getLoc(), STI);
    TOut.emitRRR(FirstShift, ATReg, SReg, ATReg, Inst.getLoc(), STI);
    TOut.emitRRR(SecondShift, DReg, SReg, TReg, Inst.getLoc(), STI);
    TOut.emitRRR(Mips::OR, DReg, DReg, ATReg, Inst.getLoc(), STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4905-4906
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4908-4909
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4911-4918
```cpp
bool MipsAsmParser::expandRotationImm(MCInst &Inst, SMLoc IDLoc,
                                      MCStreamer &Out,
                                      const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister ATReg;
  MCRegister DReg = Inst.getOperand(0).getReg();
  MCRegister SReg = Inst.getOperand(1).getReg();
  int64_t ImmValue = Inst.getOperand(2).getImm();
```
- EN: Implements `MipsAsmParser::expandRotationImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandRotationImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4920-4921
```cpp
  unsigned FirstShift = Mips::NOP;
  unsigned SecondShift = Mips::NOP;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4923-4931
```cpp
  if (hasMips32r2()) {
    if (Inst.getOpcode() == Mips::ROLImm) {
      uint64_t MaxShift = 32;
      uint64_t ShiftValue = ImmValue;
      if (ImmValue != 0)
        ShiftValue = MaxShift - ImmValue;
      TOut.emitRRI(Mips::ROTR, DReg, SReg, ShiftValue, Inst.getLoc(), STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4933-4936
```cpp
    if (Inst.getOpcode() == Mips::RORImm) {
      TOut.emitRRI(Mips::ROTR, DReg, SReg, ImmValue, Inst.getLoc(), STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4938-4939
```cpp
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4941-4945
```cpp
  if (hasMips32()) {
    if (ImmValue == 0) {
      TOut.emitRRI(Mips::SRL, DReg, SReg, 0, Inst.getLoc(), STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4947-4958
```cpp
    switch (Inst.getOpcode()) {
    default:
      llvm_unreachable("unexpected instruction opcode");
    case Mips::ROLImm:
      FirstShift = Mips::SLL;
      SecondShift = Mips::SRL;
      break;
    case Mips::RORImm:
      FirstShift = Mips::SRL;
      SecondShift = Mips::SLL;
      break;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4960-4962
```cpp
    ATReg = getATReg(Inst.getLoc());
    if (!ATReg)
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4964-4966
```cpp
    TOut.emitRRI(FirstShift, ATReg, SReg, ImmValue, Inst.getLoc(), STI);
    TOut.emitRRI(SecondShift, DReg, SReg, 32 - ImmValue, Inst.getLoc(), STI);
    TOut.emitRRR(Mips::OR, DReg, DReg, ATReg, Inst.getLoc(), STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4968-4969
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4971-4972
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4974-4981
```cpp
bool MipsAsmParser::expandDRotation(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                    const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister ATReg;
  MCRegister DReg = Inst.getOperand(0).getReg();
  MCRegister SReg = Inst.getOperand(1).getReg();
  MCRegister TReg = Inst.getOperand(2).getReg();
  MCRegister TmpReg = DReg;
```
- EN: Implements `MipsAsmParser::expandDRotation`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandDRotation`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4983-4984
```cpp
  unsigned FirstShift = Mips::NOP;
  unsigned SecondShift = Mips::NOP;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4986-4991
```cpp
  if (hasMips64r2()) {
    if (TmpReg == SReg) {
      TmpReg = getATReg(Inst.getLoc());
      if (!TmpReg)
        return true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4993-4997
```cpp
    if (Inst.getOpcode() == Mips::DROL) {
      TOut.emitRRR(Mips::DSUBu, TmpReg, Mips::ZERO, TReg, Inst.getLoc(), STI);
      TOut.emitRRR(Mips::DROTRV, DReg, SReg, TmpReg, Inst.getLoc(), STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4999-5002
```cpp
    if (Inst.getOpcode() == Mips::DROR) {
      TOut.emitRRR(Mips::DROTRV, DReg, SReg, TReg, Inst.getLoc(), STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5004-5005
```cpp
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5007-5019
```cpp
  if (hasMips64()) {
    switch (Inst.getOpcode()) {
    default:
      llvm_unreachable("unexpected instruction opcode");
    case Mips::DROL:
      FirstShift = Mips::DSRLV;
      SecondShift = Mips::DSLLV;
      break;
    case Mips::DROR:
      FirstShift = Mips::DSLLV;
      SecondShift = Mips::DSRLV;
      break;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5021-5023
```cpp
    ATReg = getATReg(Inst.getLoc());
    if (!ATReg)
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5025-5028
```cpp
    TOut.emitRRR(Mips::DSUBu, ATReg, Mips::ZERO, TReg, Inst.getLoc(), STI);
    TOut.emitRRR(FirstShift, ATReg, SReg, ATReg, Inst.getLoc(), STI);
    TOut.emitRRR(SecondShift, DReg, SReg, TReg, Inst.getLoc(), STI);
    TOut.emitRRR(Mips::OR, DReg, DReg, ATReg, Inst.getLoc(), STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5030-5031
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5033-5034
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5036-5043
```cpp
bool MipsAsmParser::expandDRotationImm(MCInst &Inst, SMLoc IDLoc,
                                       MCStreamer &Out,
                                       const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister ATReg;
  MCRegister DReg = Inst.getOperand(0).getReg();
  MCRegister SReg = Inst.getOperand(1).getReg();
  int64_t ImmValue = Inst.getOperand(2).getImm() % 64;
```
- EN: Implements `MipsAsmParser::expandDRotationImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandDRotationImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5045-5046
```cpp
  unsigned FirstShift = Mips::NOP;
  unsigned SecondShift = Mips::NOP;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5048-5048
```cpp
  MCInst TmpInst;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5050-5064
```cpp
  if (hasMips64r2()) {
    unsigned FinalOpcode = Mips::NOP;
    if (ImmValue == 0)
      FinalOpcode = Mips::DROTR;
    else if (ImmValue % 32 == 0)
      FinalOpcode = Mips::DROTR32;
    else if ((ImmValue >= 1) && (ImmValue <= 32)) {
      if (Inst.getOpcode() == Mips::DROLImm)
        FinalOpcode = Mips::DROTR32;
      else
        FinalOpcode = Mips::DROTR;
    } else if (ImmValue >= 33) {
      if (Inst.getOpcode() == Mips::DROLImm)
        FinalOpcode = Mips::DROTR;
      else
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5065-5066
```cpp
        FinalOpcode = Mips::DROTR32;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5068-5070
```cpp
    uint64_t ShiftValue = ImmValue % 32;
    if (Inst.getOpcode() == Mips::DROLImm)
      ShiftValue = (32 - ImmValue % 32) % 32;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5072-5072
```cpp
    TOut.emitRRI(FinalOpcode, DReg, SReg, ShiftValue, Inst.getLoc(), STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5074-5075
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5077-5081
```cpp
  if (hasMips64()) {
    if (ImmValue == 0) {
      TOut.emitRRI(Mips::DSRL, DReg, SReg, 0, Inst.getLoc(), STI);
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5083-5097
```cpp
    switch (Inst.getOpcode()) {
    default:
      llvm_unreachable("unexpected instruction opcode");
    case Mips::DROLImm:
      if ((ImmValue >= 1) && (ImmValue <= 31)) {
        FirstShift = Mips::DSLL;
        SecondShift = Mips::DSRL32;
      }
      if (ImmValue == 32) {
        FirstShift = Mips::DSLL32;
        SecondShift = Mips::DSRL32;
      }
      if ((ImmValue >= 33) && (ImmValue <= 63)) {
        FirstShift = Mips::DSLL32;
        SecondShift = Mips::DSRL;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5098-5112
```cpp
      }
      break;
    case Mips::DRORImm:
      if ((ImmValue >= 1) && (ImmValue <= 31)) {
        FirstShift = Mips::DSRL;
        SecondShift = Mips::DSLL32;
      }
      if (ImmValue == 32) {
        FirstShift = Mips::DSRL32;
        SecondShift = Mips::DSLL32;
      }
      if ((ImmValue >= 33) && (ImmValue <= 63)) {
        FirstShift = Mips::DSRL32;
        SecondShift = Mips::DSLL;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5113-5114
```cpp
      break;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5116-5118
```cpp
    ATReg = getATReg(Inst.getLoc());
    if (!ATReg)
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5120-5123
```cpp
    TOut.emitRRI(FirstShift, ATReg, SReg, ImmValue % 32, Inst.getLoc(), STI);
    TOut.emitRRI(SecondShift, DReg, SReg, (32 - ImmValue % 32) % 32,
                 Inst.getLoc(), STI);
    TOut.emitRRR(Mips::OR, DReg, DReg, ATReg, Inst.getLoc(), STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5125-5126
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5128-5129
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5131-5135
```cpp
bool MipsAsmParser::expandAbs(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                              const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister FirstRegOp = Inst.getOperand(0).getReg();
  MCRegister SecondRegOp = Inst.getOperand(1).getReg();
```
- EN: Implements `MipsAsmParser::expandAbs`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandAbs`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5137-5142
```cpp
  TOut.emitRI(Mips::BGEZ, SecondRegOp, 8, IDLoc, STI);
  if (FirstRegOp != SecondRegOp)
    TOut.emitRRR(Mips::ADDu, FirstRegOp, SecondRegOp, Mips::ZERO, IDLoc, STI);
  else
    TOut.emitEmptyDelaySlot(false, IDLoc, STI);
  TOut.emitRRR(Mips::SUB, FirstRegOp, Mips::ZERO, SecondRegOp, IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5144-5145
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5147-5153
```cpp
bool MipsAsmParser::expandMulImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                 const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister ATReg;
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  int32_t ImmValue = Inst.getOperand(2).getImm();
```
- EN: Implements `MipsAsmParser::expandMulImm`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandMulImm`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5155-5157
```cpp
  ATReg = getATReg(IDLoc);
  if (!ATReg)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5159-5159
```cpp
  loadImmediate(ImmValue, ATReg, MCRegister(), true, false, IDLoc, Out, STI);
```
- EN: Declares `loadImmediate`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `loadImmediate`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5161-5162
```cpp
  TOut.emitRR(Inst.getOpcode() == Mips::MULImmMacro ? Mips::MULT : Mips::DMULT,
              SrcReg, ATReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5164-5164
```cpp
  TOut.emitR(Mips::MFLO, DstReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5166-5167
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5169-5175
```cpp
bool MipsAsmParser::expandMulO(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                               const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister ATReg;
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  MCRegister TmpReg = Inst.getOperand(2).getReg();
```
- EN: Implements `MipsAsmParser::expandMulO`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandMulO`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5177-5179
```cpp
  ATReg = getATReg(Inst.getLoc());
  if (!ATReg)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5181-5182
```cpp
  TOut.emitRR(Inst.getOpcode() == Mips::MULOMacro ? Mips::MULT : Mips::DMULT,
              SrcReg, TmpReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5184-5184
```cpp
  TOut.emitR(Mips::MFLO, DstReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5186-5187
```cpp
  TOut.emitRRI(Inst.getOpcode() == Mips::MULOMacro ? Mips::SRA : Mips::DSRA32,
               DstReg, DstReg, 0x1F, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5189-5189
```cpp
  TOut.emitR(Mips::MFHI, ATReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5191-5197
```cpp
  if (useTraps()) {
    TOut.emitRRI(Mips::TNE, DstReg, ATReg, 6, IDLoc, STI);
  } else {
    MCContext &Context = TOut.getContext();
    MCSymbol * BrTarget = Context.createTempSymbol();
    MCOperand LabelOp =
        MCOperand::createExpr(MCSymbolRefExpr::create(BrTarget, Context));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5199-5202
```cpp
    TOut.emitRRX(Mips::BEQ, DstReg, ATReg, LabelOp, IDLoc, STI);
    if (AssemblerOptions.back()->isReorder())
      TOut.emitNop(IDLoc, STI);
    TOut.emitII(Mips::BREAK, 6, 0, IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5204-5206
```cpp
    TOut.getStreamer().emitLabel(BrTarget);
  }
  TOut.emitR(Mips::MFLO, DstReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5208-5209
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5211-5217
```cpp
bool MipsAsmParser::expandMulOU(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister ATReg;
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  MCRegister TmpReg = Inst.getOperand(2).getReg();
```
- EN: Implements `MipsAsmParser::expandMulOU`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandMulOU`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5219-5221
```cpp
  ATReg = getATReg(IDLoc);
  if (!ATReg)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5223-5224
```cpp
  TOut.emitRR(Inst.getOpcode() == Mips::MULOUMacro ? Mips::MULTu : Mips::DMULTu,
              SrcReg, TmpReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5226-5234
```cpp
  TOut.emitR(Mips::MFHI, ATReg, IDLoc, STI);
  TOut.emitR(Mips::MFLO, DstReg, IDLoc, STI);
  if (useTraps()) {
    TOut.emitRRI(Mips::TNE, ATReg, Mips::ZERO, 6, IDLoc, STI);
  } else {
    MCContext &Context = TOut.getContext();
    MCSymbol * BrTarget = Context.createTempSymbol();
    MCOperand LabelOp =
        MCOperand::createExpr(MCSymbolRefExpr::create(BrTarget, Context));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5236-5239
```cpp
    TOut.emitRRX(Mips::BEQ, ATReg, Mips::ZERO, LabelOp, IDLoc, STI);
    if (AssemblerOptions.back()->isReorder())
      TOut.emitNop(IDLoc, STI);
    TOut.emitII(Mips::BREAK, 6, 0, IDLoc, STI);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5241-5242
```cpp
    TOut.getStreamer().emitLabel(BrTarget);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5244-5245
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5247-5252
```cpp
bool MipsAsmParser::expandDMULMacro(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                    const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  MCRegister TmpReg = Inst.getOperand(2).getReg();
```
- EN: Implements `MipsAsmParser::expandDMULMacro`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandDMULMacro`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5254-5255
```cpp
  TOut.emitRR(Mips::DMULTu, SrcReg, TmpReg, IDLoc, STI);
  TOut.emitR(Mips::MFLO, DstReg, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5257-5258
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5260-5270
```cpp
// Expand 'ld $<reg> offset($reg2)' to 'lw $<reg>, offset($reg2);
//                                      lw $<reg+1>>, offset+4($reg2)'
// or expand 'sd $<reg> offset($reg2)' to 'sw $<reg>, offset($reg2);
//                                         sw $<reg+1>>, offset+4($reg2)'
// for O32.
bool MipsAsmParser::expandLoadStoreDMacro(MCInst &Inst, SMLoc IDLoc,
                                          MCStreamer &Out,
                                          const MCSubtargetInfo *STI,
                                          bool IsLoad) {
  if (!isABI_O32())
    return true;
```
- EN: Implements `MipsAsmParser::expandLoadStoreDMacro`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandLoadStoreDMacro`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5272-5272
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5274-5280
```cpp
  MipsTargetStreamer &TOut = getTargetStreamer();
  unsigned Opcode = IsLoad ? Mips::LW : Mips::SW;
  MCRegister FirstReg = Inst.getOperand(0).getReg();
  MCRegister SecondReg = nextReg(FirstReg);
  MCRegister BaseReg = Inst.getOperand(1).getReg();
  if (!SecondReg)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5282-5282
```cpp
  warnIfRegIndexIsAT(FirstReg, IDLoc);
```
- EN: Declares `warnIfRegIndexIsAT`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfRegIndexIsAT`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5284-5285
```cpp
  assert(Inst.getOperand(2).isImm() &&
         "Offset for load macro is not immediate!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5287-5289
```cpp
  MCOperand &FirstOffset = Inst.getOperand(2);
  signed NextOffset = FirstOffset.getImm() + 4;
  MCOperand SecondOffset = MCOperand::createImm(NextOffset);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5291-5292
```cpp
  if (!isInt<16>(FirstOffset.getImm()) || !isInt<16>(NextOffset))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5294-5302
```cpp
  // For loads, clobber the base register with the second load instead of the
  // first if the BaseReg == FirstReg.
  if (FirstReg != BaseReg || !IsLoad) {
    TOut.emitRRX(Opcode, FirstReg, BaseReg, FirstOffset, IDLoc, STI);
    TOut.emitRRX(Opcode, SecondReg, BaseReg, SecondOffset, IDLoc, STI);
  } else {
    TOut.emitRRX(Opcode, SecondReg, BaseReg, SecondOffset, IDLoc, STI);
    TOut.emitRRX(Opcode, FirstReg, BaseReg, FirstOffset, IDLoc, STI);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5304-5305
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5308-5317
```cpp
// Expand 's.d $<reg> offset($reg2)' to 'swc1 $<reg+1>, offset($reg2);
//                                       swc1 $<reg>, offset+4($reg2)'
// or if little endian to 'swc1 $<reg>, offset($reg2);
//                         swc1 $<reg+1>, offset+4($reg2)'
// for Mips1.
bool MipsAsmParser::expandStoreDM1Macro(MCInst &Inst, SMLoc IDLoc,
                                        MCStreamer &Out,
                                        const MCSubtargetInfo *STI) {
  if (!isABI_O32())
    return true;
```
- EN: Implements `MipsAsmParser::expandStoreDM1Macro`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandStoreDM1Macro`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5319-5319
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5321-5327
```cpp
  MipsTargetStreamer &TOut = getTargetStreamer();
  unsigned Opcode = Mips::SWC1;
  MCRegister FirstReg = Inst.getOperand(0).getReg();
  MCRegister SecondReg = nextReg(FirstReg);
  MCRegister BaseReg = Inst.getOperand(1).getReg();
  if (!SecondReg)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5329-5329
```cpp
  warnIfRegIndexIsAT(FirstReg, IDLoc);
```
- EN: Declares `warnIfRegIndexIsAT`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfRegIndexIsAT`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5331-5332
```cpp
  assert(Inst.getOperand(2).isImm() &&
         "Offset for macro is not immediate!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5334-5336
```cpp
  MCOperand &FirstOffset = Inst.getOperand(2);
  signed NextOffset = FirstOffset.getImm() + 4;
  MCOperand SecondOffset = MCOperand::createImm(NextOffset);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5338-5339
```cpp
  if (!isInt<16>(FirstOffset.getImm()) || !isInt<16>(NextOffset))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5341-5342
```cpp
  if (!IsLittleEndian)
    std::swap(FirstReg, SecondReg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5344-5345
```cpp
  TOut.emitRRX(Opcode, FirstReg, BaseReg, FirstOffset, IDLoc, STI);
  TOut.emitRRX(Opcode, SecondReg, BaseReg, SecondOffset, IDLoc, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5347-5348
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5350-5352
```cpp
bool MipsAsmParser::expandSeq(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                              const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandSeq`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandSeq`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5354-5357
```cpp
  assert(Inst.getNumOperands() == 3 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() &&
         Inst.getOperand(1).isReg() &&
         Inst.getOperand(2).isReg() && "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5359-5361
```cpp
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  MCRegister OpReg = Inst.getOperand(2).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5363-5363
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5365-5369
```cpp
  if (SrcReg != Mips::ZERO && OpReg != Mips::ZERO) {
    TOut.emitRRR(Mips::XOR, DstReg, SrcReg, OpReg, IDLoc, STI);
    TOut.emitRRI(Mips::SLTiu, DstReg, DstReg, 1, IDLoc, STI);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5371-5374
```cpp
  MCRegister Reg = SrcReg == Mips::ZERO ? OpReg : SrcReg;
  TOut.emitRRI(Mips::SLTiu, DstReg, Reg, 1, IDLoc, STI);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5376-5378
```cpp
bool MipsAsmParser::expandSeqI(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                               const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandSeqI`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandSeqI`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5380-5383
```cpp
  assert(Inst.getNumOperands() == 3 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() &&
         Inst.getOperand(1).isReg() &&
         Inst.getOperand(2).isImm() && "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5385-5387
```cpp
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  int64_t Imm = Inst.getOperand(2).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5389-5389
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5391-5394
```cpp
  if (Imm == 0) {
    TOut.emitRRI(Mips::SLTiu, DstReg, SrcReg, 1, IDLoc, STI);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5396-5401
```cpp
  if (SrcReg == Mips::ZERO) {
    Warning(IDLoc, "comparison is always false");
    TOut.emitRRR(isGP64bit() ? Mips::DADDu : Mips::ADDu,
                 DstReg, SrcReg, SrcReg, IDLoc, STI);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5403-5409
```cpp
  unsigned Opc;
  if (Imm > -0x8000 && Imm < 0) {
    Imm = -Imm;
    Opc = isGP64bit() ? Mips::DADDiu : Mips::ADDiu;
  } else {
    Opc = Mips::XORi;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5411-5414
```cpp
  if (!isUInt<16>(Imm)) {
    MCRegister ATReg = getATReg(IDLoc);
    if (!ATReg)
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5416-5418
```cpp
    if (loadImmediate(Imm, ATReg, MCRegister(), true, isGP64bit(), IDLoc, Out,
                      STI))
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5420-5423
```cpp
    TOut.emitRRR(Mips::XOR, DstReg, SrcReg, ATReg, IDLoc, STI);
    TOut.emitRRI(Mips::SLTiu, DstReg, DstReg, 1, IDLoc, STI);
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5425-5428
```cpp
  TOut.emitRRI(Opc, DstReg, SrcReg, Imm, IDLoc, STI);
  TOut.emitRRI(Mips::SLTiu, DstReg, DstReg, 1, IDLoc, STI);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5430-5431
```cpp
bool MipsAsmParser::expandSne(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                              const MCSubtargetInfo *STI) {
```
- EN: Implements `MipsAsmParser::expandSne`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandSne`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5433-5433
```cpp
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5435-5438
```cpp
  assert(Inst.getNumOperands() == 3 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() &&
         Inst.getOperand(1).isReg() &&
         Inst.getOperand(2).isReg() && "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5440-5442
```cpp
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  MCRegister OpReg = Inst.getOperand(2).getReg();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5444-5444
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5446-5450
```cpp
  if (SrcReg != Mips::ZERO && OpReg != Mips::ZERO) {
    TOut.emitRRR(Mips::XOR, DstReg, SrcReg, OpReg, IDLoc, STI);
    TOut.emitRRR(Mips::SLTu, DstReg, Mips::ZERO, DstReg, IDLoc, STI);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5452-5455
```cpp
  MCRegister Reg = SrcReg == Mips::ZERO ? OpReg : SrcReg;
  TOut.emitRRR(Mips::SLTu, DstReg, Mips::ZERO, Reg, IDLoc, STI);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5457-5459
```cpp
bool MipsAsmParser::expandSneI(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                               const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
```
- EN: Implements `MipsAsmParser::expandSneI`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandSneI`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5461-5464
```cpp
  assert(Inst.getNumOperands() == 3 && "Invalid operand count");
  assert(Inst.getOperand(0).isReg() &&
         Inst.getOperand(1).isReg() &&
         Inst.getOperand(2).isImm() && "Invalid instruction operand.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5466-5468
```cpp
  MCRegister DstReg = Inst.getOperand(0).getReg();
  MCRegister SrcReg = Inst.getOperand(1).getReg();
  int64_t ImmValue = Inst.getOperand(2).getImm();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5470-5470
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5472-5475
```cpp
  if (ImmValue == 0) {
    TOut.emitRRR(Mips::SLTu, DstReg, Mips::ZERO, SrcReg, IDLoc, STI);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5477-5482
```cpp
  if (SrcReg == Mips::ZERO) {
    Warning(IDLoc, "comparison is always true");
    if (loadImmediate(1, DstReg, MCRegister(), true, false, IDLoc, Out, STI))
      return true;
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5484-5490
```cpp
  unsigned Opc;
  if (ImmValue > -0x8000 && ImmValue < 0) {
    ImmValue = -ImmValue;
    Opc = isGP64bit() ? Mips::DADDiu : Mips::ADDiu;
  } else {
    Opc = Mips::XORi;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5492-5496
```cpp
  if (isUInt<16>(ImmValue)) {
    TOut.emitRRI(Opc, DstReg, SrcReg, ImmValue, IDLoc, STI);
    TOut.emitRRR(Mips::SLTu, DstReg, Mips::ZERO, DstReg, IDLoc, STI);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5498-5500
```cpp
  MCRegister ATReg = getATReg(IDLoc);
  if (!ATReg)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5502-5504
```cpp
  if (loadImmediate(ImmValue, ATReg, MCRegister(), isInt<32>(ImmValue), false,
                    IDLoc, Out, STI))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5506-5509
```cpp
  TOut.emitRRR(Mips::XOR, DstReg, SrcReg, ATReg, IDLoc, STI);
  TOut.emitRRR(Mips::SLTu, DstReg, Mips::ZERO, DstReg, IDLoc, STI);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5511-5525
```cpp
// Map the DSP accumulator and control register to the corresponding gpr
// operand. Unlike the other alias, the m(f|t)t(lo|hi|acx) instructions
// do not map the DSP registers contigously to gpr registers.
static unsigned getRegisterForMxtrDSP(MCInst &Inst, bool IsMFDSP) {
  switch (Inst.getOpcode()) {
    case Mips::MFTLO:
    case Mips::MTTLO:
      switch (Inst.getOperand(IsMFDSP ? 1 : 0).getReg().id()) {
        case Mips::AC0:
          return Mips::ZERO;
        case Mips::AC1:
          return Mips::A0;
        case Mips::AC2:
          return Mips::T0;
        case Mips::AC3:
```
- EN: Implements `getRegisterForMxtrDSP`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRegisterForMxtrDSP`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5526-5540
```cpp
          return Mips::T4;
        default:
          llvm_unreachable("Unknown register for 'mttr' alias!");
    }
    case Mips::MFTHI:
    case Mips::MTTHI:
      switch (Inst.getOperand(IsMFDSP ? 1 : 0).getReg().id()) {
        case Mips::AC0:
          return Mips::AT;
        case Mips::AC1:
          return Mips::A1;
        case Mips::AC2:
          return Mips::T1;
        case Mips::AC3:
          return Mips::T5;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5541-5555
```cpp
        default:
          llvm_unreachable("Unknown register for 'mttr' alias!");
    }
    case Mips::MFTACX:
    case Mips::MTTACX:
      switch (Inst.getOperand(IsMFDSP ? 1 : 0).getReg().id()) {
        case Mips::AC0:
          return Mips::V0;
        case Mips::AC1:
          return Mips::A2;
        case Mips::AC2:
          return Mips::T2;
        case Mips::AC3:
          return Mips::T6;
        default:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5556-5564
```cpp
          llvm_unreachable("Unknown register for 'mttr' alias!");
    }
    case Mips::MFTDSP:
    case Mips::MTTDSP:
      return Mips::S0;
    default:
      llvm_unreachable("Unknown instruction for 'mttr' dsp alias!");
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5566-5580
```cpp
// Map the floating point register operand to the corresponding register
// operand.
static unsigned getRegisterForMxtrFP(MCInst &Inst, bool IsMFTC1) {
  switch (Inst.getOperand(IsMFTC1 ? 1 : 0).getReg().id()) {
    case Mips::F0:  return Mips::ZERO;
    case Mips::F1:  return Mips::AT;
    case Mips::F2:  return Mips::V0;
    case Mips::F3:  return Mips::V1;
    case Mips::F4:  return Mips::A0;
    case Mips::F5:  return Mips::A1;
    case Mips::F6:  return Mips::A2;
    case Mips::F7:  return Mips::A3;
    case Mips::F8:  return Mips::T0;
    case Mips::F9:  return Mips::T1;
    case Mips::F10: return Mips::T2;
```
- EN: Implements `getRegisterForMxtrFP`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRegisterForMxtrFP`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5581-5595
```cpp
    case Mips::F11: return Mips::T3;
    case Mips::F12: return Mips::T4;
    case Mips::F13: return Mips::T5;
    case Mips::F14: return Mips::T6;
    case Mips::F15: return Mips::T7;
    case Mips::F16: return Mips::S0;
    case Mips::F17: return Mips::S1;
    case Mips::F18: return Mips::S2;
    case Mips::F19: return Mips::S3;
    case Mips::F20: return Mips::S4;
    case Mips::F21: return Mips::S5;
    case Mips::F22: return Mips::S6;
    case Mips::F23: return Mips::S7;
    case Mips::F24: return Mips::T8;
    case Mips::F25: return Mips::T9;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5596-5604
```cpp
    case Mips::F26: return Mips::K0;
    case Mips::F27: return Mips::K1;
    case Mips::F28: return Mips::GP;
    case Mips::F29: return Mips::SP;
    case Mips::F30: return Mips::FP;
    case Mips::F31: return Mips::RA;
    default: llvm_unreachable("Unknown register for mttc1 alias!");
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5606-5620
```cpp
// Map the coprocessor operand the corresponding gpr register operand.
static unsigned getRegisterForMxtrC0(MCInst &Inst, bool IsMFTC0) {
  switch (Inst.getOperand(IsMFTC0 ? 1 : 0).getReg().id()) {
    case Mips::COP00:  return Mips::ZERO;
    case Mips::COP01:  return Mips::AT;
    case Mips::COP02:  return Mips::V0;
    case Mips::COP03:  return Mips::V1;
    case Mips::COP04:  return Mips::A0;
    case Mips::COP05:  return Mips::A1;
    case Mips::COP06:  return Mips::A2;
    case Mips::COP07:  return Mips::A3;
    case Mips::COP08:  return Mips::T0;
    case Mips::COP09:  return Mips::T1;
    case Mips::COP010: return Mips::T2;
    case Mips::COP011: return Mips::T3;
```
- EN: Implements `getRegisterForMxtrC0`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRegisterForMxtrC0`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5621-5635
```cpp
    case Mips::COP012: return Mips::T4;
    case Mips::COP013: return Mips::T5;
    case Mips::COP014: return Mips::T6;
    case Mips::COP015: return Mips::T7;
    case Mips::COP016: return Mips::S0;
    case Mips::COP017: return Mips::S1;
    case Mips::COP018: return Mips::S2;
    case Mips::COP019: return Mips::S3;
    case Mips::COP020: return Mips::S4;
    case Mips::COP021: return Mips::S5;
    case Mips::COP022: return Mips::S6;
    case Mips::COP023: return Mips::S7;
    case Mips::COP024: return Mips::T8;
    case Mips::COP025: return Mips::T9;
    case Mips::COP026: return Mips::K0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5636-5643
```cpp
    case Mips::COP027: return Mips::K1;
    case Mips::COP028: return Mips::GP;
    case Mips::COP029: return Mips::SP;
    case Mips::COP030: return Mips::FP;
    case Mips::COP031: return Mips::RA;
    default: llvm_unreachable("Unknown register for mttc0 alias!");
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5645-5659
```cpp
/// Expand an alias of 'mftr' or 'mttr' into the full instruction, by producing
/// an mftr or mttr with the correctly mapped gpr register, u, sel and h bits.
bool MipsAsmParser::expandMXTRAlias(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                    const MCSubtargetInfo *STI) {
  MipsTargetStreamer &TOut = getTargetStreamer();
  MCRegister rd;
  unsigned u = 1;
  unsigned sel = 0;
  unsigned h = 0;
  bool IsMFTR = false;
  switch (Inst.getOpcode()) {
    case Mips::MFTC0:
      IsMFTR = true;
      [[fallthrough]];
    case Mips::MTTC0:
```
- EN: Implements `MipsAsmParser::expandMXTRAlias`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandMXTRAlias`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5660-5674
```cpp
      u = 0;
      rd = getRegisterForMxtrC0(Inst, IsMFTR);
      sel = Inst.getOperand(2).getImm();
      break;
    case Mips::MFTGPR:
      IsMFTR = true;
      [[fallthrough]];
    case Mips::MTTGPR:
      rd = Inst.getOperand(IsMFTR ? 1 : 0).getReg();
      break;
    case Mips::MFTLO:
    case Mips::MFTHI:
    case Mips::MFTACX:
    case Mips::MFTDSP:
      IsMFTR = true;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5675-5689
```cpp
      [[fallthrough]];
    case Mips::MTTLO:
    case Mips::MTTHI:
    case Mips::MTTACX:
    case Mips::MTTDSP:
      rd = getRegisterForMxtrDSP(Inst, IsMFTR);
      sel = 1;
      break;
    case Mips::MFTHC1:
      h = 1;
      [[fallthrough]];
    case Mips::MFTC1:
      IsMFTR = true;
      rd = getRegisterForMxtrFP(Inst, IsMFTR);
      sel = 2;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5690-5704
```cpp
      break;
    case Mips::MTTHC1:
      h = 1;
      [[fallthrough]];
    case Mips::MTTC1:
      rd = getRegisterForMxtrFP(Inst, IsMFTR);
      sel = 2;
      break;
    case Mips::CFTC1:
      IsMFTR = true;
      [[fallthrough]];
    case Mips::CTTC1:
      rd = getRegisterForMxtrFP(Inst, IsMFTR);
      sel = 3;
      break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5705-5710
```cpp
  }
  MCRegister Op0 = IsMFTR ? Inst.getOperand(0).getReg() : MCRegister(rd);
  MCRegister Op1 =
      IsMFTR ? MCRegister(rd)
             : (Inst.getOpcode() != Mips::MTTDSP ? Inst.getOperand(1).getReg()
                                                 : Inst.getOperand(0).getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5712-5715
```cpp
  TOut.emitRRIII(IsMFTR ? Mips::MFTR : Mips::MTTR, Op0, Op1, u, sel, h, IDLoc,
                 STI);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5717-5721
```cpp
bool MipsAsmParser::expandSaaAddr(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                                  const MCSubtargetInfo *STI) {
  assert(Inst.getNumOperands() == 3 && "expected three operands");
  assert(Inst.getOperand(0).isReg() && "expected register operand kind");
  assert(Inst.getOperand(1).isReg() && "expected register operand kind");
```
- EN: Implements `MipsAsmParser::expandSaaAddr`, a mutation/build routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::expandSaaAddr`，它是一个围绕MC streamer 交互展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5723-5723
```cpp
  warnIfNoMacro(IDLoc);
```
- EN: Declares `warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5725-5729
```cpp
  MipsTargetStreamer &TOut = getTargetStreamer();
  unsigned Opcode = Inst.getOpcode() == Mips::SaaAddr ? Mips::SAA : Mips::SAAD;
  MCRegister RtReg = Inst.getOperand(0).getReg();
  MCRegister BaseReg = Inst.getOperand(1).getReg();
  const MCOperand &BaseOp = Inst.getOperand(2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5731-5737
```cpp
  if (BaseOp.isImm()) {
    int64_t ImmValue = BaseOp.getImm();
    if (ImmValue == 0) {
      TOut.emitRR(Opcode, RtReg, BaseReg, IDLoc, STI);
      return false;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5739-5741
```cpp
  MCRegister ATReg = getATReg(IDLoc);
  if (!ATReg)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5743-5744
```cpp
  if (expandLoadAddress(ATReg, BaseReg, BaseOp, !isGP64bit(), IDLoc, Out, STI))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5746-5748
```cpp
  TOut.emitRR(Opcode, RtReg, ATReg, IDLoc, STI);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5750-5763
```cpp
unsigned
MipsAsmParser::checkEarlyTargetMatchPredicate(MCInst &Inst,
                                              const OperandVector &Operands) {
  switch (Inst.getOpcode()) {
  default:
    return Match_Success;
  case Mips::DATI:
  case Mips::DAHI:
    if (static_cast<MipsOperand &>(*Operands[1])
            .isValidForTie(static_cast<MipsOperand &>(*Operands[2])))
      return Match_Success;
    return Match_RequiresSameSrcAndDst;
  }
}
```
- EN: Implements `MipsAsmParser::checkEarlyTargetMatchPredicate`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::checkEarlyTargetMatchPredicate`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5765-5779
```cpp
unsigned MipsAsmParser::checkTargetMatchPredicate(MCInst &Inst) {
  switch (Inst.getOpcode()) {
  // As described by the MIPSR6 spec, daui must not use the zero operand for
  // its source operand.
  case Mips::DAUI:
    if (Inst.getOperand(1).getReg() == Mips::ZERO ||
        Inst.getOperand(1).getReg() == Mips::ZERO_64)
      return Match_RequiresNoZeroRegister;
    return Match_Success;
  // As described by the Mips32r2 spec, the registers Rd and Rs for
  // jalr.hb must be different.
  // It also applies for registers Rt and Rs of microMIPSr6 jalrc.hb instruction
  // and registers Rd and Base for microMIPS lwp instruction
  case Mips::JALR_HB:
  case Mips::JALR_HB64:
```
- EN: Implements `MipsAsmParser::checkTargetMatchPredicate`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::checkTargetMatchPredicate`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5780-5794
```cpp
  case Mips::JALRC_HB_MMR6:
  case Mips::JALRC_MMR6:
    if (Inst.getOperand(0).getReg() == Inst.getOperand(1).getReg())
      return Match_RequiresDifferentSrcAndDst;
    return Match_Success;
  case Mips::LWP_MM:
    if (Inst.getOperand(0).getReg() == Inst.getOperand(2).getReg())
      return Match_RequiresDifferentSrcAndDst;
    return Match_Success;
  case Mips::SYNC:
    if (Inst.getOperand(0).getImm() != 0 && !hasMips32())
      return Match_NonZeroOperandForSync;
    return Match_Success;
  case Mips::MFC0:
  case Mips::MTC0:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5795-5809
```cpp
  case Mips::MTC2:
  case Mips::MFC2:
    if (Inst.getOperand(2).getImm() != 0 && !hasMips32())
      return Match_NonZeroOperandForMTCX;
    return Match_Success;
  // As described the MIPSR6 spec, the compact branches that compare registers
  // must:
  // a) Not use the zero register.
  // b) Not use the same register twice.
  // c) rs < rt for bnec, beqc.
  //    NB: For this case, the encoding will swap the operands as their
  //    ordering doesn't matter. GAS performs this transformation  too.
  //    Hence, that constraint does not have to be enforced.
  //
  // The compact branches that branch iff the signed addition of two registers
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5810-5824
```cpp
  // would overflow must have rs >= rt. That can be handled like beqc/bnec with
  // operand swapping. They do not have restriction of using the zero register.
  case Mips::BLEZC:   case Mips::BLEZC_MMR6:
  case Mips::BGEZC:   case Mips::BGEZC_MMR6:
  case Mips::BGTZC:   case Mips::BGTZC_MMR6:
  case Mips::BLTZC:   case Mips::BLTZC_MMR6:
  case Mips::BEQZC:   case Mips::BEQZC_MMR6:
  case Mips::BNEZC:   case Mips::BNEZC_MMR6:
  case Mips::BLEZC64:
  case Mips::BGEZC64:
  case Mips::BGTZC64:
  case Mips::BLTZC64:
  case Mips::BEQZC64:
  case Mips::BNEZC64:
    if (Inst.getOperand(0).getReg() == Mips::ZERO ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5825-5839
```cpp
        Inst.getOperand(0).getReg() == Mips::ZERO_64)
      return Match_RequiresNoZeroRegister;
    return Match_Success;
  case Mips::BGEC:    case Mips::BGEC_MMR6:
  case Mips::BLTC:    case Mips::BLTC_MMR6:
  case Mips::BGEUC:   case Mips::BGEUC_MMR6:
  case Mips::BLTUC:   case Mips::BLTUC_MMR6:
  case Mips::BEQC:    case Mips::BEQC_MMR6:
  case Mips::BNEC:    case Mips::BNEC_MMR6:
  case Mips::BGEC64:
  case Mips::BLTC64:
  case Mips::BGEUC64:
  case Mips::BLTUC64:
  case Mips::BEQC64:
  case Mips::BNEC64:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5840-5854
```cpp
    if (Inst.getOperand(0).getReg() == Mips::ZERO ||
        Inst.getOperand(0).getReg() == Mips::ZERO_64)
      return Match_RequiresNoZeroRegister;
    if (Inst.getOperand(1).getReg() == Mips::ZERO ||
        Inst.getOperand(1).getReg() == Mips::ZERO_64)
      return Match_RequiresNoZeroRegister;
    if (Inst.getOperand(0).getReg() == Inst.getOperand(1).getReg())
      return Match_RequiresDifferentOperands;
    return Match_Success;
  case Mips::DINS: {
    assert(Inst.getOperand(2).isImm() && Inst.getOperand(3).isImm() &&
           "Operands must be immediates for dins!");
    const signed Pos = Inst.getOperand(2).getImm();
    const signed Size = Inst.getOperand(3).getImm();
    if ((0 > (Pos + Size)) || ((Pos + Size) > 32))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5855-5869
```cpp
      return Match_RequiresPosSizeRange0_32;
    return Match_Success;
  }
  case Mips::DINSM:
  case Mips::DINSU: {
    assert(Inst.getOperand(2).isImm() && Inst.getOperand(3).isImm() &&
           "Operands must be immediates for dinsm/dinsu!");
    const signed Pos = Inst.getOperand(2).getImm();
    const signed Size = Inst.getOperand(3).getImm();
    if ((32 >= (Pos + Size)) || ((Pos + Size) > 64))
      return Match_RequiresPosSizeRange33_64;
    return Match_Success;
  }
  case Mips::DEXT: {
    assert(Inst.getOperand(2).isImm() && Inst.getOperand(3).isImm() &&
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5870-5884
```cpp
           "Operands must be immediates for DEXTM!");
    const signed Pos = Inst.getOperand(2).getImm();
    const signed Size = Inst.getOperand(3).getImm();
    if ((1 > (Pos + Size)) || ((Pos + Size) > 63))
      return Match_RequiresPosSizeUImm6;
    return Match_Success;
  }
  case Mips::DEXTM:
  case Mips::DEXTU: {
    assert(Inst.getOperand(2).isImm() && Inst.getOperand(3).isImm() &&
           "Operands must be immediates for dextm/dextu!");
    const signed Pos = Inst.getOperand(2).getImm();
    const signed Size = Inst.getOperand(3).getImm();
    if ((32 > (Pos + Size)) || ((Pos + Size) > 64))
      return Match_RequiresPosSizeRange33_64;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5885-5894
```cpp
    return Match_Success;
  }
  case Mips::CRC32B: case Mips::CRC32CB:
  case Mips::CRC32H: case Mips::CRC32CH:
  case Mips::CRC32W: case Mips::CRC32CW:
  case Mips::CRC32D: case Mips::CRC32CD:
    if (Inst.getOperand(0).getReg() != Inst.getOperand(2).getReg())
      return Match_RequiresSameSrcAndDst;
    return Match_Success;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5896-5899
```cpp
  uint64_t TSFlags = MII.get(Inst.getOpcode()).TSFlags;
  if ((TSFlags & MipsII::HasFCCRegOperand) &&
      (Inst.getOperand(0).getReg() != Mips::FCC0) && !hasEightFccRegisters())
    return Match_NoFCCRegisterForCurrentISA;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5901-5901
```cpp
  return Match_Success;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5903-5903
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5905-5914
```cpp
static SMLoc RefineErrorLoc(const SMLoc Loc, const OperandVector &Operands,
                            uint64_t ErrorInfo) {
  if (ErrorInfo != ~0ULL && ErrorInfo < Operands.size()) {
    SMLoc ErrorLoc = Operands[ErrorInfo]->getStartLoc();
    if (ErrorLoc == SMLoc())
      return Loc;
    return ErrorLoc;
  }
  return Loc;
}
```
- EN: Implements `RefineErrorLoc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `RefineErrorLoc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5916-5923
```cpp
bool MipsAsmParser::matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
                                            OperandVector &Operands,
                                            MCStreamer &Out,
                                            uint64_t &ErrorInfo,
                                            bool MatchingInlineAsm) {
  MCInst Inst;
  unsigned MatchResult =
      MatchInstructionImpl(Operands, Inst, ErrorInfo, MatchingInlineAsm);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5925-5939
```cpp
  switch (MatchResult) {
  case Match_Success:
    if (processInstruction(Inst, IDLoc, Out, STI))
      return true;
    return false;
  case Match_MissingFeature:
    Error(IDLoc, "instruction requires a CPU feature not currently enabled");
    return true;
  case Match_InvalidTiedOperand:
    Error(IDLoc, "operand must match destination register");
    return true;
  case Match_InvalidOperand: {
    SMLoc ErrorLoc = IDLoc;
    if (ErrorInfo != ~0ULL) {
      if (ErrorInfo >= Operands.size())
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5940-5940
```cpp
        return Error(IDLoc, "too few operands for instruction");
```
- EN: Declares `Error`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Error`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5942-5945
```cpp
      ErrorLoc = Operands[ErrorInfo]->getStartLoc();
      if (ErrorLoc == SMLoc())
        ErrorLoc = IDLoc;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5947-5961
```cpp
    return Error(ErrorLoc, "invalid operand for instruction");
  }
  case Match_NonZeroOperandForSync:
    return Error(IDLoc,
                 "s-type must be zero or unspecified for pre-MIPS32 ISAs");
  case Match_NonZeroOperandForMTCX:
    return Error(IDLoc, "selector must be zero for pre-MIPS32 ISAs");
  case Match_MnemonicFail:
    return Error(IDLoc, "invalid instruction");
  case Match_RequiresDifferentSrcAndDst:
    return Error(IDLoc, "source and destination must be different");
  case Match_RequiresDifferentOperands:
    return Error(IDLoc, "registers must be different");
  case Match_RequiresNoZeroRegister:
    return Error(IDLoc, "invalid operand ($zero) for instruction");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5962-5976
```cpp
  case Match_RequiresSameSrcAndDst:
    return Error(IDLoc, "source and destination must match");
  case Match_NoFCCRegisterForCurrentISA:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "non-zero fcc register doesn't exist in current ISA level");
  case Match_Immz:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo), "expected '0'");
  case Match_UImm1_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 1-bit unsigned immediate");
  case Match_UImm2_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 2-bit unsigned immediate");
  case Match_UImm2_1:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5977-5991
```cpp
                 "expected immediate in range 1 .. 4");
  case Match_UImm3_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 3-bit unsigned immediate");
  case Match_UImm4_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 4-bit unsigned immediate");
  case Match_SImm4_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 4-bit signed immediate");
  case Match_UImm5_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 5-bit unsigned immediate");
  case Match_SImm5_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5992-6006
```cpp
                 "expected 5-bit signed immediate");
  case Match_UImm5_1:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range 1 .. 32");
  case Match_UImm5_32:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range 32 .. 63");
  case Match_UImm5_33:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range 33 .. 64");
  case Match_UImm5_0_Report_UImm6:
    // This is used on UImm5 operands that have a corresponding UImm5_32
    // operand to avoid confusing the user.
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 6-bit unsigned immediate");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6007-6021
```cpp
  case Match_UImm5_Lsl2:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected both 7-bit unsigned immediate and multiple of 4");
  case Match_UImmRange2_64:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range 2 .. 64");
  case Match_UImm6_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 6-bit unsigned immediate");
  case Match_UImm6_Lsl2:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected both 8-bit unsigned immediate and multiple of 4");
  case Match_SImm6_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 6-bit signed immediate");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6022-6036
```cpp
  case Match_UImm7_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 7-bit unsigned immediate");
  case Match_UImm7_N1:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range -1 .. 126");
  case Match_SImm7_Lsl2:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected both 9-bit signed immediate and multiple of 4");
  case Match_UImm8_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 8-bit unsigned immediate");
  case Match_UImm10_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 10-bit unsigned immediate");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6037-6051
```cpp
  case Match_SImm10_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 10-bit signed immediate");
  case Match_SImm11_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 11-bit signed immediate");
  case Match_UImm16:
  case Match_UImm16_Relaxed:
  case Match_UImm16_AltRelaxed:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 16-bit unsigned immediate");
  case Match_SImm16:
  case Match_SImm16_Relaxed:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 16-bit signed immediate");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6052-6066
```cpp
  case Match_SImm19_Lsl2:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected both 19-bit signed immediate and multiple of 4");
  case Match_UImm20_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 20-bit unsigned immediate");
  case Match_UImm26_0:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 26-bit unsigned immediate");
  case Match_SImm32:
  case Match_SImm32_Relaxed:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected 32-bit signed immediate");
  case Match_UImm32_Coerced:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6067-6081
```cpp
                 "expected 32-bit immediate");
  case Match_MemSImm9:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected memory with 9-bit signed offset");
  case Match_MemSImm10:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected memory with 10-bit signed offset");
  case Match_MemSImm10Lsl1:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected memory with 11-bit signed offset and multiple of 2");
  case Match_MemSImm10Lsl2:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected memory with 12-bit signed offset and multiple of 4");
  case Match_MemSImm10Lsl3:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6082-6096
```cpp
                 "expected memory with 13-bit signed offset and multiple of 8");
  case Match_MemSImm11:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected memory with 11-bit signed offset");
  case Match_MemSImm12:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected memory with 12-bit signed offset");
  case Match_MemSImm16:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected memory with 16-bit signed offset");
  case Match_MemSImmPtr:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected memory with 32-bit signed offset");
  case Match_RequiresPosSizeRange0_32: {
    SMLoc ErrorStart = Operands[3]->getStartLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6097-6111
```cpp
    SMLoc ErrorEnd = Operands[4]->getEndLoc();
    return Error(ErrorStart, "size plus position are not in the range 0 .. 32",
                 SMRange(ErrorStart, ErrorEnd));
    }
  case Match_RequiresPosSizeUImm6: {
    SMLoc ErrorStart = Operands[3]->getStartLoc();
    SMLoc ErrorEnd = Operands[4]->getEndLoc();
    return Error(ErrorStart, "size plus position are not in the range 1 .. 63",
                 SMRange(ErrorStart, ErrorEnd));
    }
  case Match_RequiresPosSizeRange33_64: {
    SMLoc ErrorStart = Operands[3]->getStartLoc();
    SMLoc ErrorEnd = Operands[4]->getEndLoc();
    return Error(ErrorStart, "size plus position are not in the range 33 .. 64",
                 SMRange(ErrorStart, ErrorEnd));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6112-6113
```cpp
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6115-6116
```cpp
  llvm_unreachable("Implement any new match types added!");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6118-6122
```cpp
void MipsAsmParser::warnIfRegIndexIsAT(MCRegister RegIndex, SMLoc Loc) {
  if (RegIndex && AssemblerOptions.back()->getATRegIndex() == RegIndex)
    Warning(Loc, "used $at (currently $" + Twine(RegIndex.id()) +
                     ") without \".set noat\"");
}
```
- EN: Implements `MipsAsmParser::warnIfRegIndexIsAT`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::warnIfRegIndexIsAT`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6124-6127
```cpp
void MipsAsmParser::warnIfNoMacro(SMLoc Loc) {
  if (!AssemblerOptions.back()->isMacro())
    Warning(Loc, "macro instruction expanded into multiple instructions");
}
```
- EN: Implements `MipsAsmParser::warnIfNoMacro`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::warnIfNoMacro`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6129-6138
```cpp
void MipsAsmParser::ConvertXWPOperands(MCInst &Inst,
                                       const OperandVector &Operands) {
  assert(
      (Inst.getOpcode() == Mips::LWP_MM || Inst.getOpcode() == Mips::SWP_MM) &&
      "Unexpected instruction!");
  ((MipsOperand &)*Operands[1]).addGPR32ZeroAsmRegOperands(Inst, 1);
  MCRegister NextReg = nextReg(((MipsOperand &)*Operands[1]).getGPR32Reg());
  Inst.addOperand(MCOperand::createReg(NextReg));
  ((MipsOperand &)*Operands[2]).addMemOperands(Inst, 2);
}
```
- EN: Implements `MipsAsmParser::ConvertXWPOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::ConvertXWPOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6140-6146
```cpp
void
MipsAsmParser::printWarningWithFixIt(const Twine &Msg, const Twine &FixMsg,
                                     SMRange Range, bool ShowColors) {
  getSourceManager().PrintMessage(Range.Start, SourceMgr::DK_Warning, Msg,
                                  Range, SMFixIt(Range, FixMsg),
                                  ShowColors);
}
```
- EN: Implements `MipsAsmParser::printWarningWithFixIt`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::printWarningWithFixIt`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6148-6149
```cpp
int MipsAsmParser::matchCPURegisterName(StringRef Name) {
  int CC;
```
- EN: Implements `MipsAsmParser::matchCPURegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::matchCPURegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6151-6165
```cpp
  CC = StringSwitch<unsigned>(Name)
           .Case("zero", 0)
           .Cases({"at", "AT"}, 1)
           .Case("a0", 4)
           .Case("a1", 5)
           .Case("a2", 6)
           .Case("a3", 7)
           .Case("v0", 2)
           .Case("v1", 3)
           .Case("s0", 16)
           .Case("s1", 17)
           .Case("s2", 18)
           .Case("s3", 19)
           .Case("s4", 20)
           .Case("s5", 21)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6166-6180
```cpp
           .Case("s6", 22)
           .Case("s7", 23)
           .Case("k0", 26)
           .Case("k1", 27)
           .Case("gp", 28)
           .Case("sp", 29)
           .Case("fp", 30)
           .Case("s8", 30)
           .Case("ra", 31)
           .Case("t0", 8)
           .Case("t1", 9)
           .Case("t2", 10)
           .Case("t3", 11)
           .Case("t4", 12)
           .Case("t5", 13)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6181-6185
```cpp
           .Case("t6", 14)
           .Case("t7", 15)
           .Case("t8", 24)
           .Case("t9", 25)
           .Default(-1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6187-6188
```cpp
  if (!(isABI_N32() || isABI_N64()))
    return CC;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6190-6193
```cpp
  if (12 <= CC && CC <= 15) {
    // Name is one of t4-t7
    AsmToken RegTok = getLexer().peekTok();
    SMRange RegRange = RegTok.getLocRange();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6195-6201
```cpp
    StringRef FixedName = StringSwitch<StringRef>(Name)
                              .Case("t4", "t0")
                              .Case("t5", "t1")
                              .Case("t6", "t2")
                              .Case("t7", "t3")
                              .Default("");
    assert(FixedName != "" &&  "Register name is not one of t4-t7.");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6203-6205
```cpp
    printWarningWithFixIt("register names $t4-$t7 are only available in O32.",
                          "Did you mean $" + FixedName + "?", RegRange);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6207-6211
```cpp
  // Although SGI documentation just cuts out t0-t3 for n32/n64,
  // GNU pushes the values of t0-t3 to override the o32/o64 values for t4-t7
  // We are supporting both cases, so for t0-t3 we'll just push them to t4-t7.
  if (8 <= CC && CC <= 11)
    CC += 4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6213-6221
```cpp
  if (CC == -1)
    CC = StringSwitch<unsigned>(Name)
             .Case("a4", 8)
             .Case("a5", 9)
             .Case("a6", 10)
             .Case("a7", 11)
             .Case("kt0", 26)
             .Case("kt1", 27)
             .Default(-1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6223-6224
```cpp
  return CC;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6226-6227
```cpp
int MipsAsmParser::matchHWRegsRegisterName(StringRef Name) {
  int CC;
```
- EN: Implements `MipsAsmParser::matchHWRegsRegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::matchHWRegsRegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6229-6235
```cpp
  CC = StringSwitch<unsigned>(Name)
            .Case("hwr_cpunum", 0)
            .Case("hwr_synci_step", 1)
            .Case("hwr_cc", 2)
            .Case("hwr_ccres", 3)
            .Case("hwr_ulr", 29)
            .Default(-1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6237-6238
```cpp
  return CC;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6240-6251
```cpp
int MipsAsmParser::matchFPURegisterName(StringRef Name) {
  if (Name[0] == 'f') {
    StringRef NumString = Name.substr(1);
    unsigned IntVal;
    if (NumString.getAsInteger(10, IntVal))
      return -1;     // This is not an integer.
    if (IntVal > 31) // Maximum index for fpu register.
      return -1;
    return IntVal;
  }
  return -1;
}
```
- EN: Implements `MipsAsmParser::matchFPURegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::matchFPURegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6253-6264
```cpp
int MipsAsmParser::matchFCCRegisterName(StringRef Name) {
  if (Name.starts_with("fcc")) {
    StringRef NumString = Name.substr(3);
    unsigned IntVal;
    if (NumString.getAsInteger(10, IntVal))
      return -1;    // This is not an integer.
    if (IntVal > 7) // There are only 8 fcc registers.
      return -1;
    return IntVal;
  }
  return -1;
}
```
- EN: Implements `MipsAsmParser::matchFCCRegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::matchFCCRegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6266-6277
```cpp
int MipsAsmParser::matchACRegisterName(StringRef Name) {
  if (Name.starts_with("ac")) {
    StringRef NumString = Name.substr(2);
    unsigned IntVal;
    if (NumString.getAsInteger(10, IntVal))
      return -1;    // This is not an integer.
    if (IntVal > 3) // There are only 3 acc registers.
      return -1;
    return IntVal;
  }
  return -1;
}
```
- EN: Implements `MipsAsmParser::matchACRegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::matchACRegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6279-6280
```cpp
int MipsAsmParser::matchMSA128RegisterName(StringRef Name) {
  unsigned IntVal;
```
- EN: Implements `MipsAsmParser::matchMSA128RegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::matchMSA128RegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6282-6283
```cpp
  if (Name.front() != 'w' || Name.drop_front(1).getAsInteger(10, IntVal))
    return -1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6285-6286
```cpp
  if (IntVal > 31)
    return -1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6288-6289
```cpp
  return IntVal;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6291-6292
```cpp
int MipsAsmParser::matchMSA128CtrlRegisterName(StringRef Name) {
  int CC;
```
- EN: Implements `MipsAsmParser::matchMSA128CtrlRegisterName`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::matchMSA128CtrlRegisterName`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6294-6303
```cpp
  CC = StringSwitch<unsigned>(Name)
           .Case("msair", 0)
           .Case("msacsr", 1)
           .Case("msaaccess", 2)
           .Case("msasave", 3)
           .Case("msamodify", 4)
           .Case("msarequest", 5)
           .Case("msamap", 6)
           .Case("msaunmap", 7)
           .Default(-1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6305-6306
```cpp
  return CC;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6308-6310
```cpp
bool MipsAsmParser::canUseATReg() {
  return AssemblerOptions.back()->getATRegIndex() != 0;
}
```
- EN: Implements `MipsAsmParser::canUseATReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::canUseATReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6312-6322
```cpp
MCRegister MipsAsmParser::getATReg(SMLoc Loc) {
  unsigned ATIndex = AssemblerOptions.back()->getATRegIndex();
  if (ATIndex == 0) {
    reportParseError(Loc,
                     "pseudo-instruction requires $at, which is not available");
    return 0;
  }
  MCRegister AT = getReg(
      (isGP64bit()) ? Mips::GPR64RegClassID : Mips::GPR32RegClassID, ATIndex);
  return AT;
}
```
- EN: Implements `MipsAsmParser::getATReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::getATReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6324-6326
```cpp
MCRegister MipsAsmParser::getReg(int RC, int RegNo) {
  return getContext().getRegisterInfo()->getRegClass(RC).getRegister(RegNo);
}
```
- EN: Implements `MipsAsmParser::getReg`, a query/helper routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::getReg`，它是一个围绕寄存器管理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6328-6342
```cpp
// Parse an expression with optional relocation operator prefixes (e.g. %lo).
// Some weird expressions allowed by gas are not supported for simplicity,
// e.g. "%lo foo", "(%lo(foo))", "%lo(foo)+1".
const MCExpr *MipsAsmParser::parseRelocExpr() {
  auto getOp = [](StringRef Op) {
    return StringSwitch<Mips::Specifier>(Op)
        .Case("call16", Mips::S_GOT_CALL)
        .Case("call_hi", Mips::S_CALL_HI16)
        .Case("call_lo", Mips::S_CALL_LO16)
        .Case("dtprel_hi", Mips::S_DTPREL_HI)
        .Case("dtprel_lo", Mips::S_DTPREL_LO)
        .Case("got", Mips::S_GOT)
        .Case("got_disp", Mips::S_GOT_DISP)
        .Case("got_hi", Mips::S_GOT_HI16)
        .Case("got_lo", Mips::S_GOT_LO16)
```
- EN: Implements `MipsAsmParser::parseRelocExpr`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseRelocExpr`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6343-6357
```cpp
        .Case("got_ofst", Mips::S_GOT_OFST)
        .Case("got_page", Mips::S_GOT_PAGE)
        .Case("gottprel", Mips::S_GOTTPREL)
        .Case("gp_rel", Mips::S_GPREL)
        .Case("hi", Mips::S_HI)
        .Case("higher", Mips::S_HIGHER)
        .Case("highest", Mips::S_HIGHEST)
        .Case("lo", Mips::S_LO)
        .Case("neg", Mips::S_NEG)
        .Case("pcrel_hi", Mips::S_PCREL_HI16)
        .Case("pcrel_lo", Mips::S_PCREL_LO16)
        .Case("tlsgd", Mips::S_TLSGD)
        .Case("tlsldm", Mips::S_TLSLDM)
        .Case("tprel_hi", Mips::S_TPREL_HI)
        .Case("tprel_lo", Mips::S_TPREL_LO)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6358-6359
```cpp
        .Default(Mips::S_None);
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6361-6375
```cpp
  MCAsmParser &Parser = getParser();
  StringRef Name;
  const MCExpr *Res = nullptr;
  SmallVector<Mips::Specifier, 0> Ops;
  while (parseOptionalToken(AsmToken::Percent)) {
    if (Parser.parseIdentifier(Name) ||
        Parser.parseToken(AsmToken::LParen, "expected '('"))
      return nullptr;
    auto Op = getOp(Name);
    if (Op == Mips::S_None) {
      Error(Parser.getTok().getLoc(), "invalid relocation operator");
      return nullptr;
    }
    Ops.push_back(Op);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6376-6384
```cpp
  if (Parser.parseExpression(Res))
    return nullptr;
  while (Ops.size()) {
    if (Parser.parseToken(AsmToken::RParen, "expected ')'"))
      return nullptr;
    Res = MCSpecifierExpr::create(Res, Ops.pop_back_val(), getContext());
  }
  return Res;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6386-6388
```cpp
bool MipsAsmParser::parseOperand(OperandVector &Operands, StringRef Mnemonic) {
  MCAsmParser &Parser = getParser();
  LLVM_DEBUG(dbgs() << "parseOperand\n");
```
- EN: Implements `MipsAsmParser::parseOperand`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseOperand`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6390-6404
```cpp
  // Check if the current operand has a custom associated parser, if so, try to
  // custom parse the operand, or fallback to the general approach.
  // Setting the third parameter to true tells the parser to keep parsing even
  // if the operands are not supported with the current feature set. In this
  // case, the instruction matcher will output a "instruction requires a CPU
  // feature not currently enabled" error. If this were false, the parser would
  // stop here and output a less useful "invalid operand" error.
  ParseStatus Res = MatchOperandParserImpl(Operands, Mnemonic, true);
  if (Res.isSuccess())
    return false;
  // If there wasn't a custom match, try the generic matcher below. Otherwise,
  // there was a match, but an error occurred, in which case, just return that
  // the operand parsing failed.
  if (Res.isFailure())
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6406-6406
```cpp
  LLVM_DEBUG(dbgs() << ".. Generic Parser\n");
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6408-6411
```cpp
  switch (getLexer().getKind()) {
  case AsmToken::Dollar: {
    // Parse the register.
    SMLoc S = Parser.getTok().getLoc();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6413-6419
```cpp
    // Almost all registers have been parsed by custom parsers. There is only
    // one exception to this. $zero (and it's alias $0) will reach this point
    // for div, divu, and similar instructions because it is not an operand
    // to the instruction definition but an explicit register. Special case
    // this situation for now.
    if (!parseAnyRegister(Operands).isNoMatch())
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6421-6424
```cpp
    // Maybe it is a symbol reference.
    StringRef Identifier;
    if (Parser.parseIdentifier(Identifier))
      return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6426-6429
```cpp
    SMLoc E = SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
    MCSymbol *Sym = getContext().getOrCreateSymbol(Identifier);
    // Otherwise create a symbol reference.
    const MCExpr *SymRef = MCSymbolRefExpr::create(Sym, getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6431-6445
```cpp
    Operands.push_back(MipsOperand::CreateImm(SymRef, S, E, *this));
    return false;
  }
  default: {
    SMLoc S = Parser.getTok().getLoc(); // Start location of the operand.
    const MCExpr *Expr = parseRelocExpr();
    if (!Expr)
      return true;
    SMLoc E = SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
    Operands.push_back(MipsOperand::CreateImm(Expr, S, E, *this));
    return false;
  }
  } // switch(getLexer().getKind())
  return true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6447-6450
```cpp
bool MipsAsmParser::parseRegister(MCRegister &Reg, SMLoc &StartLoc,
                                  SMLoc &EndLoc) {
  return !tryParseRegister(Reg, StartLoc, EndLoc).isSuccess();
}
```
- EN: Implements `MipsAsmParser::parseRegister`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseRegister`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6452-6460
```cpp
ParseStatus MipsAsmParser::tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
                                            SMLoc &EndLoc) {
  SmallVector<std::unique_ptr<MCParsedAsmOperand>, 1> Operands;
  ParseStatus Res = parseAnyRegister(Operands);
  if (Res.isSuccess()) {
    assert(Operands.size() == 1);
    MipsOperand &Operand = static_cast<MipsOperand &>(*Operands.front());
    StartLoc = Operand.getStartLoc();
    EndLoc = Operand.getEndLoc();
```
- EN: Implements `MipsAsmParser::tryParseRegister`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::tryParseRegister`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6462-6469
```cpp
    // AFAIK, we only support numeric registers and named GPR's in CFI
    // directives.
    // Don't worry about eating tokens before failing. Using an unrecognised
    // register is a parse error.
    if (Operand.isGPRAsmReg()) {
      // Resolve to GPR32 or GPR64 appropriately.
      Reg = isGP64bit() ? Operand.getGPR64Reg() : Operand.getGPR32Reg();
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6471-6472
```cpp
    return (Reg == (unsigned)-1) ? ParseStatus::NoMatch : ParseStatus::Success;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6474-6476
```cpp
  assert(Operands.size() == 0);
  return (Reg == (unsigned)-1) ? ParseStatus::NoMatch : ParseStatus::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6478-6486
```cpp
ParseStatus MipsAsmParser::parseMemOperand(OperandVector &Operands) {
  MCAsmParser &Parser = getParser();
  LLVM_DEBUG(dbgs() << "parseMemOperand\n");
  const MCExpr *IdVal = nullptr;
  SMLoc S;
  bool isParenExpr = false;
  ParseStatus Res = ParseStatus::NoMatch;
  // First operand is the offset.
  S = Parser.getTok().getLoc();
```
- EN: Implements `MipsAsmParser::parseMemOperand`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseMemOperand`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6488-6491
```cpp
  if (getLexer().getKind() == AsmToken::LParen) {
    Parser.Lex();
    isParenExpr = true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6493-6498
```cpp
  if (getLexer().getKind() != AsmToken::Dollar) {
    IdVal = parseRelocExpr();
    if (!IdVal)
      return ParseStatus::Failure;
    if (isParenExpr && Parser.parseRParen())
      return ParseStatus::Failure;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6500-6511
```cpp
    const AsmToken &Tok = Parser.getTok(); // Get the next token.
    if (Tok.isNot(AsmToken::LParen)) {
      MipsOperand &Mnemonic = static_cast<MipsOperand &>(*Operands[0]);
      if (Mnemonic.getToken() == "la" || Mnemonic.getToken() == "dla") {
        SMLoc E =
            SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
        Operands.push_back(MipsOperand::CreateImm(IdVal, S, E, *this));
        return ParseStatus::Success;
      }
      if (Tok.is(AsmToken::EndOfStatement)) {
        SMLoc E =
            SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6513-6527
```cpp
        // Zero register assumed, add a memory operand with ZERO as its base.
        // "Base" will be managed by k_Memory.
        auto Base = MipsOperand::createGPRReg(
            0, "0", getContext().getRegisterInfo(), S, E, *this);
        Operands.push_back(
            MipsOperand::CreateMem(std::move(Base), IdVal, S, E, *this));
        return ParseStatus::Success;
      }
      MCBinaryExpr::Opcode Opcode;
      // GAS and LLVM treat comparison operators different. GAS will generate -1
      // or 0, while LLVM will generate 0 or 1. Since a comparsion operator is
      // highly unlikely to be found in a memory offset expression, we don't
      // handle them.
      switch (Tok.getKind()) {
      case AsmToken::Plus:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6528-6542
```cpp
        Opcode = MCBinaryExpr::Add;
        Parser.Lex();
        break;
      case AsmToken::Minus:
        Opcode = MCBinaryExpr::Sub;
        Parser.Lex();
        break;
      case AsmToken::Star:
        Opcode = MCBinaryExpr::Mul;
        Parser.Lex();
        break;
      case AsmToken::Pipe:
        Opcode = MCBinaryExpr::Or;
        Parser.Lex();
        break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6543-6557
```cpp
      case AsmToken::Amp:
        Opcode = MCBinaryExpr::And;
        Parser.Lex();
        break;
      case AsmToken::LessLess:
        Opcode = MCBinaryExpr::Shl;
        Parser.Lex();
        break;
      case AsmToken::GreaterGreater:
        Opcode = MCBinaryExpr::LShr;
        Parser.Lex();
        break;
      case AsmToken::Caret:
        Opcode = MCBinaryExpr::Xor;
        Parser.Lex();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6558-6572
```cpp
        break;
      case AsmToken::Slash:
        Opcode = MCBinaryExpr::Div;
        Parser.Lex();
        break;
      case AsmToken::Percent:
        Opcode = MCBinaryExpr::Mod;
        Parser.Lex();
        break;
      default:
        return Error(Parser.getTok().getLoc(), "'(' or expression expected");
      }
      const MCExpr * NextExpr;
      if (getParser().parseExpression(NextExpr))
        return ParseStatus::Failure;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6573-6574
```cpp
      IdVal = MCBinaryExpr::create(Opcode, IdVal, NextExpr, getContext());
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6576-6577
```cpp
    Parser.Lex(); // Eat the '(' token.
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6579-6581
```cpp
  Res = parseAnyRegister(Operands);
  if (!Res.isSuccess())
    return Res;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6583-6584
```cpp
  if (Parser.getTok().isNot(AsmToken::RParen))
    return Error(Parser.getTok().getLoc(), "')' expected");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6586-6586
```cpp
  SMLoc E = SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6588-6588
```cpp
  Parser.Lex(); // Eat the ')' token.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6590-6591
```cpp
  if (!IdVal)
    IdVal = MCConstantExpr::create(0, getContext());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6593-6607
```cpp
  // Replace the register operand with the memory operand.
  std::unique_ptr<MipsOperand> op(
      static_cast<MipsOperand *>(Operands.back().release()));
  // Remove the register from the operands.
  // "op" will be managed by k_Memory.
  Operands.pop_back();
  // Add the memory operand.
  if (const MCBinaryExpr *BE = dyn_cast<MCBinaryExpr>(IdVal)) {
    int64_t Imm;
    if (IdVal->evaluateAsAbsolute(Imm))
      IdVal = MCConstantExpr::create(Imm, getContext());
    else if (BE->getLHS()->getKind() != MCExpr::SymbolRef)
      IdVal = MCBinaryExpr::create(BE->getOpcode(), BE->getRHS(), BE->getLHS(),
                                   getContext());
  }
```
- EN: Implements `op`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `op`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6609-6611
```cpp
  Operands.push_back(MipsOperand::CreateMem(std::move(op), IdVal, S, E, *this));
  return ParseStatus::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6613-6617
```cpp
bool MipsAsmParser::searchSymbolAlias(OperandVector &Operands) {
  MCAsmParser &Parser = getParser();
  MCSymbol *Sym = getContext().lookupSymbol(Parser.getTok().getIdentifier());
  if (!Sym)
    return false;
```
- EN: Implements `MipsAsmParser::searchSymbolAlias`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::searchSymbolAlias`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6619-6633
```cpp
  SMLoc S = Parser.getTok().getLoc();
  if (Sym->isVariable()) {
    const MCExpr *Expr = Sym->getVariableValue();
    if (Expr->getKind() == MCExpr::SymbolRef) {
      const MCSymbolRefExpr *Ref = static_cast<const MCSymbolRefExpr *>(Expr);
      StringRef DefSymbol = Ref->getSymbol().getName();
      if (DefSymbol.starts_with("$")) {
        ParseStatus Res =
            matchAnyRegisterNameWithoutDollar(Operands, DefSymbol.substr(1), S);
        if (Res.isSuccess()) {
          Parser.Lex();
          return true;
        }
        if (Res.isFailure())
          llvm_unreachable("Should never fail");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6634-6648
```cpp
      }
    }
  } else if (Sym->isUndefined()) {
    // If symbol is unset, it might be created in the `parseSetAssignment`
    // routine as an alias for a numeric register name.
    // Lookup in the aliases list.
    auto Entry = RegisterSets.find(Sym->getName());
    if (Entry != RegisterSets.end()) {
      ParseStatus Res =
          matchAnyRegisterWithoutDollar(Operands, Entry->getValue(), S);
      if (Res.isSuccess()) {
        Parser.Lex();
        return true;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6649-6649
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6651-6652
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6654-6662
```cpp
ParseStatus MipsAsmParser::matchAnyRegisterNameWithoutDollar(
    OperandVector &Operands, StringRef Identifier, SMLoc S) {
  int Index = matchCPURegisterName(Identifier);
  if (Index != -1) {
    Operands.push_back(MipsOperand::createGPRReg(
        Index, Identifier, getContext().getRegisterInfo(), S,
        getLexer().getLoc(), *this));
    return ParseStatus::Success;
  }
```
- EN: Implements `MipsAsmParser::matchAnyRegisterNameWithoutDollar`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::matchAnyRegisterNameWithoutDollar`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6664-6670
```cpp
  Index = matchHWRegsRegisterName(Identifier);
  if (Index != -1) {
    Operands.push_back(MipsOperand::createHWRegsReg(
        Index, Identifier, getContext().getRegisterInfo(), S,
        getLexer().getLoc(), *this));
    return ParseStatus::Success;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6672-6678
```cpp
  Index = matchFPURegisterName(Identifier);
  if (Index != -1) {
    Operands.push_back(MipsOperand::createFGRReg(
        Index, Identifier, getContext().getRegisterInfo(), S,
        getLexer().getLoc(), *this));
    return ParseStatus::Success;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6680-6686
```cpp
  Index = matchFCCRegisterName(Identifier);
  if (Index != -1) {
    Operands.push_back(MipsOperand::createFCCReg(
        Index, Identifier, getContext().getRegisterInfo(), S,
        getLexer().getLoc(), *this));
    return ParseStatus::Success;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6688-6694
```cpp
  Index = matchACRegisterName(Identifier);
  if (Index != -1) {
    Operands.push_back(MipsOperand::createACCReg(
        Index, Identifier, getContext().getRegisterInfo(), S,
        getLexer().getLoc(), *this));
    return ParseStatus::Success;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6696-6702
```cpp
  Index = matchMSA128RegisterName(Identifier);
  if (Index != -1) {
    Operands.push_back(MipsOperand::createMSA128Reg(
        Index, Identifier, getContext().getRegisterInfo(), S,
        getLexer().getLoc(), *this));
    return ParseStatus::Success;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6704-6710
```cpp
  Index = matchMSA128CtrlRegisterName(Identifier);
  if (Index != -1) {
    Operands.push_back(MipsOperand::createMSACtrlReg(
        Index, Identifier, getContext().getRegisterInfo(), S,
        getLexer().getLoc(), *this));
    return ParseStatus::Success;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6712-6713
```cpp
  return ParseStatus::NoMatch;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6715-6729
```cpp
ParseStatus
MipsAsmParser::matchAnyRegisterWithoutDollar(OperandVector &Operands,
                                             const AsmToken &Token, SMLoc S) {
  if (Token.is(AsmToken::Identifier)) {
    LLVM_DEBUG(dbgs() << ".. identifier\n");
    StringRef Identifier = Token.getIdentifier();
    return matchAnyRegisterNameWithoutDollar(Operands, Identifier, S);
  }
  if (Token.is(AsmToken::Integer)) {
    LLVM_DEBUG(dbgs() << ".. integer\n");
    int64_t RegNum = Token.getIntVal();
    if (RegNum < 0 || RegNum > 31) {
      // Show the error, but treat invalid register
      // number as a normal one to continue parsing
      // and catch other possible errors.
```
- EN: Implements `MipsAsmParser::matchAnyRegisterWithoutDollar`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::matchAnyRegisterWithoutDollar`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6730-6736
```cpp
      Error(getLexer().getLoc(), "invalid register number");
    }
    Operands.push_back(MipsOperand::createNumericReg(
        RegNum, Token.getString(), getContext().getRegisterInfo(), S,
        Token.getLoc(), *this));
    return ParseStatus::Success;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6738-6738
```cpp
  LLVM_DEBUG(dbgs() << Token.getKind() << "\n");
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6740-6741
```cpp
  return ParseStatus::NoMatch;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6743-6747
```cpp
ParseStatus
MipsAsmParser::matchAnyRegisterWithoutDollar(OperandVector &Operands, SMLoc S) {
  auto Token = getLexer().peekTok(false);
  return matchAnyRegisterWithoutDollar(Operands, Token, S);
}
```
- EN: Implements `MipsAsmParser::matchAnyRegisterWithoutDollar`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::matchAnyRegisterWithoutDollar`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6749-6751
```cpp
ParseStatus MipsAsmParser::parseAnyRegister(OperandVector &Operands) {
  MCAsmParser &Parser = getParser();
  LLVM_DEBUG(dbgs() << "parseAnyRegister\n");
```
- EN: Implements `MipsAsmParser::parseAnyRegister`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseAnyRegister`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6753-6753
```cpp
  auto Token = Parser.getTok();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6755-6755
```cpp
  SMLoc S = Token.getLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6757-6766
```cpp
  if (Token.isNot(AsmToken::Dollar)) {
    LLVM_DEBUG(dbgs() << ".. !$ -> try sym aliasing\n");
    if (Token.is(AsmToken::Identifier)) {
      if (searchSymbolAlias(Operands))
        return ParseStatus::Success;
    }
    LLVM_DEBUG(dbgs() << ".. !symalias -> NoMatch\n");
    return ParseStatus::NoMatch;
  }
  LLVM_DEBUG(dbgs() << ".. $\n");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6768-6774
```cpp
  ParseStatus Res = matchAnyRegisterWithoutDollar(Operands, S);
  if (Res.isSuccess()) {
    Parser.Lex(); // $
    Parser.Lex(); // identifier
  }
  return Res;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6776-6778
```cpp
ParseStatus MipsAsmParser::parseJumpTarget(OperandVector &Operands) {
  MCAsmParser &Parser = getParser();
  LLVM_DEBUG(dbgs() << "parseJumpTarget\n");
```
- EN: Implements `MipsAsmParser::parseJumpTarget`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseJumpTarget`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6780-6780
```cpp
  SMLoc S = getLexer().getLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6782-6785
```cpp
  // Registers are a valid target and have priority over symbols.
  ParseStatus Res = parseAnyRegister(Operands);
  if (!Res.isNoMatch())
    return Res;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6787-6796
```cpp
  // Integers and expressions are acceptable
  const MCExpr *Expr = nullptr;
  if (Parser.parseExpression(Expr)) {
    // We have no way of knowing if a symbol was consumed so we must ParseFail
    return ParseStatus::Failure;
  }
  Operands.push_back(
      MipsOperand::CreateImm(Expr, S, getLexer().getLoc(), *this));
  return ParseStatus::Success;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6798-6812
```cpp
ParseStatus MipsAsmParser::parseInvNum(OperandVector &Operands) {
  MCAsmParser &Parser = getParser();
  const MCExpr *IdVal;
  // If the first token is '$' we may have register operand. We have to reject
  // cases where it is not a register. Complicating the matter is that
  // register names are not reserved across all ABIs.
  // Peek past the dollar to see if it's a register name for this ABI.
  SMLoc S = Parser.getTok().getLoc();
  if (Parser.getTok().is(AsmToken::Dollar)) {
    return matchCPURegisterName(Parser.getLexer().peekTok().getString()) == -1
               ? ParseStatus::Failure
               : ParseStatus::NoMatch;
  }
  if (getParser().parseExpression(IdVal))
    return ParseStatus::Failure;
```
- EN: Implements `MipsAsmParser::parseInvNum`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseInvNum`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6813-6821
```cpp
  const MCConstantExpr *MCE = dyn_cast<MCConstantExpr>(IdVal);
  if (!MCE)
    return ParseStatus::NoMatch;
  int64_t Val = MCE->getValue();
  SMLoc E = SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
  Operands.push_back(MipsOperand::CreateImm(
      MCConstantExpr::create(0 - Val, getContext()), S, E, *this));
  return ParseStatus::Success;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6823-6829
```cpp
ParseStatus MipsAsmParser::parseRegisterList(OperandVector &Operands) {
  MCAsmParser &Parser = getParser();
  SmallVector<MCRegister, 10> Regs;
  MCRegister Reg;
  MCRegister PrevReg;
  bool RegRange = false;
  SmallVector<std::unique_ptr<MCParsedAsmOperand>, 8> TmpOperands;
```
- EN: Implements `MipsAsmParser::parseRegisterList`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseRegisterList`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6831-6832
```cpp
  if (Parser.getTok().isNot(AsmToken::Dollar))
    return ParseStatus::Failure;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6834-6848
```cpp
  SMLoc S = Parser.getTok().getLoc();
  while (parseAnyRegister(TmpOperands).isSuccess()) {
    SMLoc E = getLexer().getLoc();
    MipsOperand &RegOpnd = static_cast<MipsOperand &>(*TmpOperands.back());
    Reg = isGP64bit() ? RegOpnd.getGPR64Reg() : RegOpnd.getGPR32Reg();
    if (RegRange) {
      // Remove last register operand because registers from register range
      // should be inserted first.
      if ((isGP64bit() && Reg == Mips::RA_64) ||
          (!isGP64bit() && Reg == Mips::RA)) {
        Regs.push_back(Reg);
      } else {
        MCRegister TmpReg = PrevReg + 1;
        while (TmpReg <= Reg) {
          if ((((TmpReg < Mips::S0) || (TmpReg > Mips::S7)) && !isGP64bit()) ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6849-6851
```cpp
              (((TmpReg < Mips::S0_64) || (TmpReg > Mips::S7_64)) &&
               isGP64bit()))
            return Error(E, "invalid register operand");
```
- EN: Declares `isGP64bit`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isGP64bit`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6853-6857
```cpp
          PrevReg = TmpReg;
          Regs.push_back(TmpReg);
          TmpReg = TmpReg.id() + 1;
        }
      }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6859-6873
```cpp
      RegRange = false;
    } else {
      if (!PrevReg.isValid() &&
          ((isGP64bit() && (Reg != Mips::S0_64) && (Reg != Mips::RA_64)) ||
           (!isGP64bit() && (Reg != Mips::S0) && (Reg != Mips::RA))))
        return Error(E, "$16 or $31 expected");
      if (!(((Reg == Mips::FP || Reg == Mips::RA ||
              (Reg >= Mips::S0 && Reg <= Mips::S7)) &&
             !isGP64bit()) ||
            ((Reg == Mips::FP_64 || Reg == Mips::RA_64 ||
              (Reg >= Mips::S0_64 && Reg <= Mips::S7_64)) &&
             isGP64bit())))
        return Error(E, "invalid register operand");
      if (PrevReg.isValid() && (Reg != PrevReg + 1) &&
          ((Reg != Mips::FP && Reg != Mips::RA && !isGP64bit()) ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6874-6875
```cpp
           (Reg != Mips::FP_64 && Reg != Mips::RA_64 && isGP64bit())))
        return Error(E, "consecutive register numbers expected");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6877-6878
```cpp
      Regs.push_back(Reg);
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6880-6881
```cpp
    if (Parser.getTok().is(AsmToken::Minus))
      RegRange = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6883-6885
```cpp
    if (!Parser.getTok().isNot(AsmToken::Minus) &&
        !Parser.getTok().isNot(AsmToken::Comma))
      return Error(E, "',' or '-' expected");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6887-6889
```cpp
    Lex(); // Consume comma or minus
    if (Parser.getTok().isNot(AsmToken::Dollar))
      break;
```
- EN: Declares `Lex`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Lex`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6891-6892
```cpp
    PrevReg = Reg;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6894-6898
```cpp
  SMLoc E = Parser.getTok().getLoc();
  Operands.push_back(MipsOperand::CreateRegList(Regs, S, E, *this));
  parseMemOperand(Operands);
  return ParseStatus::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6900-6914
```cpp
/// Sometimes (i.e. load/stores) the operand may be followed immediately by
/// either this.
/// ::= '(', register, ')'
/// handle it before we iterate so we don't get tripped up by the lack of
/// a comma.
bool MipsAsmParser::parseParenSuffix(StringRef Name, OperandVector &Operands) {
  MCAsmParser &Parser = getParser();
  if (getLexer().is(AsmToken::LParen)) {
    Operands.push_back(
        MipsOperand::CreateToken("(", getLexer().getLoc(), *this));
    Parser.Lex();
    if (parseOperand(Operands, Name)) {
      SMLoc Loc = getLexer().getLoc();
      return Error(Loc, "unexpected token in argument list");
    }
```
- EN: Implements `MipsAsmParser::parseParenSuffix`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseParenSuffix`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6915-6924
```cpp
    if (Parser.getTok().isNot(AsmToken::RParen)) {
      SMLoc Loc = getLexer().getLoc();
      return Error(Loc, "unexpected token, expected ')'");
    }
    Operands.push_back(
        MipsOperand::CreateToken(")", getLexer().getLoc(), *this));
    Parser.Lex();
  }
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6926-6940
```cpp
/// Sometimes (i.e. in MSA) the operand may be followed immediately by
/// either one of these.
/// ::= '[', register, ']'
/// ::= '[', integer, ']'
/// handle it before we iterate so we don't get tripped up by the lack of
/// a comma.
bool MipsAsmParser::parseBracketSuffix(StringRef Name,
                                       OperandVector &Operands) {
  MCAsmParser &Parser = getParser();
  if (getLexer().is(AsmToken::LBrac)) {
    Operands.push_back(
        MipsOperand::CreateToken("[", getLexer().getLoc(), *this));
    Parser.Lex();
    if (parseOperand(Operands, Name)) {
      SMLoc Loc = getLexer().getLoc();
```
- EN: Implements `MipsAsmParser::parseBracketSuffix`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseBracketSuffix`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6941-6952
```cpp
      return Error(Loc, "unexpected token in argument list");
    }
    if (Parser.getTok().isNot(AsmToken::RBrac)) {
      SMLoc Loc = getLexer().getLoc();
      return Error(Loc, "unexpected token, expected ']'");
    }
    Operands.push_back(
        MipsOperand::CreateToken("]", getLexer().getLoc(), *this));
    Parser.Lex();
  }
  return false;
}
```
- EN: Implements `Error`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Error`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6954-6955
```cpp
static std::string MipsMnemonicSpellCheck(StringRef S, const FeatureBitset &FBS,
                                          unsigned VariantID = 0);
```
- EN: Declares `MipsMnemonicSpellCheck`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MipsMnemonicSpellCheck`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6957-6968
```cpp
bool MipsAsmParser::areEqualRegs(const MCParsedAsmOperand &Op1,
                                 const MCParsedAsmOperand &Op2) const {
  // This target-overriden function exists to maintain current behaviour for
  // e.g.
  //   dahi    $3, $3, 0x5678
  // as tested in test/MC/Mips/mips64r6/valid.s.
  // FIXME: Should this test actually fail with an error? If so, then remove
  // this overloaded method.
  if (!Op1.isReg() || !Op2.isReg())
    return true;
  return Op1.getReg() == Op2.getReg();
}
```
- EN: Implements `MipsAsmParser::areEqualRegs`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::areEqualRegs`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6970-6973
```cpp
bool MipsAsmParser::parseInstruction(ParseInstructionInfo &Info, StringRef Name,
                                     SMLoc NameLoc, OperandVector &Operands) {
  MCAsmParser &Parser = getParser();
  LLVM_DEBUG(dbgs() << "parseInstruction\n");
```
- EN: Implements `MipsAsmParser::parseInstruction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseInstruction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6975-6976
```cpp
  // We have reached first instruction, module directive are now forbidden.
  getTargetStreamer().forbidModuleDirective();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6978-6985
```cpp
  // Check if we have valid mnemonic
  if (!mnemonicIsValid(Name, 0)) {
    FeatureBitset FBS = ComputeAvailableFeatures(getSTI().getFeatureBits());
    std::string Suggestion = MipsMnemonicSpellCheck(Name, FBS);
    return Error(NameLoc, "unknown instruction" + Suggestion);
  }
  // First operand in MCInst is instruction mnemonic.
  Operands.push_back(MipsOperand::CreateToken(Name, NameLoc, *this));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6987-6996
```cpp
  // Read the remaining operands.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    // Read the first operand.
    if (parseOperand(Operands, Name)) {
      SMLoc Loc = getLexer().getLoc();
      return Error(Loc, "unexpected token in argument list");
    }
    if (getLexer().is(AsmToken::LBrac) && parseBracketSuffix(Name, Operands))
      return true;
    // AFAIK, parenthesis suffixes are never on the first operand
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6998-7012
```cpp
    while (getLexer().is(AsmToken::Comma)) {
      Parser.Lex(); // Eat the comma.
      // Parse and remember the operand.
      if (parseOperand(Operands, Name)) {
        SMLoc Loc = getLexer().getLoc();
        return Error(Loc, "unexpected token in argument list");
      }
      // Parse bracket and parenthesis suffixes before we iterate
      if (getLexer().is(AsmToken::LBrac)) {
        if (parseBracketSuffix(Name, Operands))
          return true;
      } else if (getLexer().is(AsmToken::LParen) &&
                 parseParenSuffix(Name, Operands))
        return true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7013-7020
```cpp
  }
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    SMLoc Loc = getLexer().getLoc();
    return Error(Loc, "unexpected token in argument list");
  }
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Implements `getLexer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getLexer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7022-7027
```cpp
// FIXME: Given that these have the same name, these should both be
// consistent on affecting the Parser.
bool MipsAsmParser::reportParseError(const Twine &ErrorMsg) {
  SMLoc Loc = getLexer().getLoc();
  return Error(Loc, ErrorMsg);
}
```
- EN: Implements `MipsAsmParser::reportParseError`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::reportParseError`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7029-7031
```cpp
bool MipsAsmParser::reportParseError(SMLoc Loc, const Twine &ErrorMsg) {
  return Error(Loc, ErrorMsg);
}
```
- EN: Implements `MipsAsmParser::reportParseError`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::reportParseError`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7033-7035
```cpp
bool MipsAsmParser::parseSetNoAtDirective() {
  MCAsmParser &Parser = getParser();
  // Line should look like: ".set noat".
```
- EN: Implements `MipsAsmParser::parseSetNoAtDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetNoAtDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7037-7038
```cpp
  // Set the $at register to $0.
  AssemblerOptions.back()->setATRegIndex(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7040-7040
```cpp
  Parser.Lex(); // Eat "noat".
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7042-7046
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7048-7051
```cpp
  getTargetStreamer().emitDirectiveSetNoAt();
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7053-7057
```cpp
bool MipsAsmParser::parseSetAtDirective() {
  // Line can be: ".set at", which sets $at to $1
  //          or  ".set at=$reg", which sets $at to $reg.
  MCAsmParser &Parser = getParser();
  Parser.Lex(); // Eat "at".
```
- EN: Implements `MipsAsmParser::parseSetAtDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetAtDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7059-7061
```cpp
  if (getLexer().is(AsmToken::EndOfStatement)) {
    // No register was specified, so we set $at to $1.
    AssemblerOptions.back()->setATRegIndex(1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7063-7066
```cpp
    getTargetStreamer().emitDirectiveSetAt();
    Parser.Lex(); // Consume the EndOfStatement.
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7068-7072
```cpp
  if (getLexer().isNot(AsmToken::Equal)) {
    reportParseError("unexpected token, expected equals sign");
    return false;
  }
  Parser.Lex(); // Eat "=".
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7074-7083
```cpp
  if (getLexer().isNot(AsmToken::Dollar)) {
    if (getLexer().is(AsmToken::EndOfStatement)) {
      reportParseError("no register specified");
      return false;
    } else {
      reportParseError("unexpected token, expected dollar sign '$'");
      return false;
    }
  }
  Parser.Lex(); // Eat "$".
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7085-7095
```cpp
  // Find out what "reg" is.
  unsigned AtRegNo;
  const AsmToken &Reg = Parser.getTok();
  if (Reg.is(AsmToken::Identifier)) {
    AtRegNo = matchCPURegisterName(Reg.getIdentifier());
  } else if (Reg.is(AsmToken::Integer)) {
    AtRegNo = Reg.getIntVal();
  } else {
    reportParseError("unexpected token, expected identifier or integer");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7097-7102
```cpp
  // Check if $reg is a valid register. If it is, set $at to $reg.
  if (!AssemblerOptions.back()->setATRegIndex(AtRegNo)) {
    reportParseError("invalid register");
    return false;
  }
  Parser.Lex(); // Eat "reg".
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7104-7108
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7110-7110
```cpp
  getTargetStreamer().emitDirectiveSetAtWithArg(AtRegNo);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7112-7114
```cpp
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7116-7128
```cpp
bool MipsAsmParser::parseSetReorderDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex();
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
  AssemblerOptions.back()->setReorder();
  getTargetStreamer().emitDirectiveSetReorder();
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Implements `MipsAsmParser::parseSetReorderDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetReorderDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7130-7142
```cpp
bool MipsAsmParser::parseSetNoReorderDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex();
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
  AssemblerOptions.back()->setNoReorder();
  getTargetStreamer().emitDirectiveSetNoReorder();
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Implements `MipsAsmParser::parseSetNoReorderDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetNoReorderDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7144-7156
```cpp
bool MipsAsmParser::parseSetMacroDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex();
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
  AssemblerOptions.back()->setMacro();
  getTargetStreamer().emitDirectiveSetMacro();
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Implements `MipsAsmParser::parseSetMacroDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetMacroDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7158-7172
```cpp
bool MipsAsmParser::parseSetNoMacroDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex();
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
  if (AssemblerOptions.back()->isReorder()) {
    reportParseError("`noreorder' must be set before `nomacro'");
    return false;
  }
  AssemblerOptions.back()->setNoMacro();
  getTargetStreamer().emitDirectiveSetNoMacro();
  Parser.Lex(); // Consume the EndOfStatement.
```
- EN: Implements `MipsAsmParser::parseSetNoMacroDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetNoMacroDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7173-7174
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7176-7178
```cpp
bool MipsAsmParser::parseSetMsaDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex();
```
- EN: Implements `MipsAsmParser::parseSetMsaDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetMsaDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7180-7182
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return reportParseError("unexpected token, expected end of statement");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7184-7187
```cpp
  setFeatureBits(Mips::FeatureMSA, "msa");
  getTargetStreamer().emitDirectiveSetMsa();
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7189-7191
```cpp
bool MipsAsmParser::parseSetNoMsaDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex();
```
- EN: Implements `MipsAsmParser::parseSetNoMsaDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetNoMsaDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7193-7195
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return reportParseError("unexpected token, expected end of statement");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7197-7200
```cpp
  clearFeatureBits(Mips::FeatureMSA, "msa");
  getTargetStreamer().emitDirectiveSetNoMsa();
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7202-7204
```cpp
bool MipsAsmParser::parseSetNoDspDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex(); // Eat "nodsp".
```
- EN: Implements `MipsAsmParser::parseSetNoDspDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetNoDspDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7206-7210
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7212-7215
```cpp
  clearFeatureBits(Mips::FeatureDSP, "dsp");
  getTargetStreamer().emitDirectiveSetNoDsp();
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7217-7219
```cpp
bool MipsAsmParser::parseSetNoMips3DDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex(); // Eat "nomips3d".
```
- EN: Implements `MipsAsmParser::parseSetNoMips3DDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetNoMips3DDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7221-7225
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7227-7230
```cpp
  clearFeatureBits(Mips::FeatureMips3D, "mips3d");
  getTargetStreamer().emitDirectiveSetNoMips3D();
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7232-7234
```cpp
bool MipsAsmParser::parseSetMips16Directive() {
  MCAsmParser &Parser = getParser();
  Parser.Lex(); // Eat "mips16".
```
- EN: Implements `MipsAsmParser::parseSetMips16Directive`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetMips16Directive`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7236-7240
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7242-7246
```cpp
  setFeatureBits(Mips::FeatureMips16, "mips16");
  getTargetStreamer().emitDirectiveSetMips16();
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Declares `setFeatureBits`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setFeatureBits`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7248-7250
```cpp
bool MipsAsmParser::parseSetNoMips16Directive() {
  MCAsmParser &Parser = getParser();
  Parser.Lex(); // Eat "nomips16".
```
- EN: Implements `MipsAsmParser::parseSetNoMips16Directive`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetNoMips16Directive`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7252-7256
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7258-7262
```cpp
  clearFeatureBits(Mips::FeatureMips16, "mips16");
  getTargetStreamer().emitDirectiveSetNoMips16();
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Declares `clearFeatureBits`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `clearFeatureBits`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7264-7277
```cpp
bool MipsAsmParser::parseSetFpDirective() {
  MCAsmParser &Parser = getParser();
  MipsABIFlagsSection::FpABIKind FpAbiVal;
  // Line can be: .set fp=32
  //              .set fp=xx
  //              .set fp=64
  Parser.Lex(); // Eat fp token
  AsmToken Tok = Parser.getTok();
  if (Tok.isNot(AsmToken::Equal)) {
    reportParseError("unexpected token, expected equals sign '='");
    return false;
  }
  Parser.Lex(); // Eat '=' token.
  Tok = Parser.getTok();
```
- EN: Implements `MipsAsmParser::parseSetFpDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetFpDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7279-7280
```cpp
  if (!parseFpABIValue(FpAbiVal, ".set"))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7282-7289
```cpp
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
  getTargetStreamer().emitDirectiveSetFp(FpAbiVal);
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7291-7292
```cpp
bool MipsAsmParser::parseSetOddSPRegDirective() {
  MCAsmParser &Parser = getParser();
```
- EN: Implements `MipsAsmParser::parseSetOddSPRegDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetOddSPRegDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7294-7298
```cpp
  Parser.Lex(); // Eat "oddspreg".
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7300-7303
```cpp
  clearFeatureBits(Mips::FeatureNoOddSPReg, "nooddspreg");
  getTargetStreamer().emitDirectiveSetOddSPReg();
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7305-7306
```cpp
bool MipsAsmParser::parseSetNoOddSPRegDirective() {
  MCAsmParser &Parser = getParser();
```
- EN: Implements `MipsAsmParser::parseSetNoOddSPRegDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetNoOddSPRegDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7308-7312
```cpp
  Parser.Lex(); // Eat "nooddspreg".
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7314-7317
```cpp
  setFeatureBits(Mips::FeatureNoOddSPReg, "nooddspreg");
  getTargetStreamer().emitDirectiveSetNoOddSPReg();
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7319-7321
```cpp
bool MipsAsmParser::parseSetMtDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex(); // Eat "mt".
```
- EN: Implements `MipsAsmParser::parseSetMtDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetMtDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7323-7327
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7329-7333
```cpp
  setFeatureBits(Mips::FeatureMT, "mt");
  getTargetStreamer().emitDirectiveSetMt();
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Declares `setFeatureBits`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setFeatureBits`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7335-7337
```cpp
bool MipsAsmParser::parseSetNoMtDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex(); // Eat "nomt".
```
- EN: Implements `MipsAsmParser::parseSetNoMtDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetNoMtDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7339-7343
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7345-7345
```cpp
  clearFeatureBits(Mips::FeatureMT, "mt");
```
- EN: Declares `clearFeatureBits`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `clearFeatureBits`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7347-7350
```cpp
  getTargetStreamer().emitDirectiveSetNoMt();
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7352-7354
```cpp
bool MipsAsmParser::parseSetNoCRCDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex(); // Eat "nocrc".
```
- EN: Implements `MipsAsmParser::parseSetNoCRCDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetNoCRCDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7356-7360
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7362-7362
```cpp
  clearFeatureBits(Mips::FeatureCRC, "crc");
```
- EN: Declares `clearFeatureBits`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `clearFeatureBits`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7364-7367
```cpp
  getTargetStreamer().emitDirectiveSetNoCRC();
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7369-7371
```cpp
bool MipsAsmParser::parseSetNoVirtDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex(); // Eat "novirt".
```
- EN: Implements `MipsAsmParser::parseSetNoVirtDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetNoVirtDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7373-7377
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7379-7379
```cpp
  clearFeatureBits(Mips::FeatureVirt, "virt");
```
- EN: Declares `clearFeatureBits`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `clearFeatureBits`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7381-7384
```cpp
  getTargetStreamer().emitDirectiveSetNoVirt();
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7386-7388
```cpp
bool MipsAsmParser::parseSetNoGINVDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex(); // Eat "noginv".
```
- EN: Implements `MipsAsmParser::parseSetNoGINVDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetNoGINVDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7390-7394
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7396-7396
```cpp
  clearFeatureBits(Mips::FeatureGINV, "ginv");
```
- EN: Declares `clearFeatureBits`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `clearFeatureBits`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7398-7401
```cpp
  getTargetStreamer().emitDirectiveSetNoGINV();
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7403-7405
```cpp
bool MipsAsmParser::parseSetPopDirective() {
  MCAsmParser &Parser = getParser();
  SMLoc Loc = getLexer().getLoc();
```
- EN: Implements `MipsAsmParser::parseSetPopDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetPopDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7407-7409
```cpp
  Parser.Lex();
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return reportParseError("unexpected token, expected end of statement");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7411-7414
```cpp
  // Always keep an element on the options "stack" to prevent the user
  // from changing the initial options. This is how we remember them.
  if (AssemblerOptions.size() == 2)
    return reportParseError(Loc, ".set pop with no .set push");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7416-7420
```cpp
  MCSubtargetInfo &STI = copySTI();
  AssemblerOptions.pop_back();
  setAvailableFeatures(
      ComputeAvailableFeatures(AssemblerOptions.back()->getFeatures()));
  STI.setFeatureBits(AssemblerOptions.back()->getFeatures());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7422-7424
```cpp
  getTargetStreamer().emitDirectiveSetPop();
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7426-7430
```cpp
bool MipsAsmParser::parseSetPushDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex();
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return reportParseError("unexpected token, expected end of statement");
```
- EN: Implements `MipsAsmParser::parseSetPushDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetPushDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7432-7434
```cpp
  // Create a copy of the current assembler options environment and push it.
  AssemblerOptions.push_back(
        std::make_unique<MipsAssemblerOptions>(AssemblerOptions.back().get()));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7436-7438
```cpp
  getTargetStreamer().emitDirectiveSetPush();
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7440-7444
```cpp
bool MipsAsmParser::parseSetSoftFloatDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex();
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return reportParseError("unexpected token, expected end of statement");
```
- EN: Implements `MipsAsmParser::parseSetSoftFloatDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetSoftFloatDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7446-7449
```cpp
  setFeatureBits(Mips::FeatureSoftFloat, "soft-float");
  getTargetStreamer().emitDirectiveSetSoftFloat();
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7451-7455
```cpp
bool MipsAsmParser::parseSetHardFloatDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex();
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return reportParseError("unexpected token, expected end of statement");
```
- EN: Implements `MipsAsmParser::parseSetHardFloatDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetHardFloatDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7457-7460
```cpp
  clearFeatureBits(Mips::FeatureSoftFloat, "soft-float");
  getTargetStreamer().emitDirectiveSetHardFloat();
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7462-7464
```cpp
bool MipsAsmParser::parseSetAssignment() {
  StringRef Name;
  MCAsmParser &Parser = getParser();
```
- EN: Implements `MipsAsmParser::parseSetAssignment`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetAssignment`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7466-7467
```cpp
  if (Parser.parseIdentifier(Name))
    return reportParseError("expected identifier after .set");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7469-7471
```cpp
  if (getLexer().isNot(AsmToken::Comma))
    return reportParseError("unexpected token, expected comma");
  Lex(); // Eat comma
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7473-7482
```cpp
  if (getLexer().is(AsmToken::Dollar) &&
      getLexer().peekTok().is(AsmToken::Integer)) {
    // Parse assignment of a numeric register:
    //   .set r1,$1
    Parser.Lex(); // Eat $.
    RegisterSets[Name] = Parser.getTok();
    Parser.Lex(); // Eat identifier.
    getContext().getOrCreateSymbol(Name);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7484-7489
```cpp
  MCSymbol *Sym;
  const MCExpr *Value;
  if (MCParserUtils::parseAssignmentExpression(Name, /* allow_redef */ true,
                                               Parser, Sym, Value))
    return true;
  getStreamer().emitAssignment(Sym, Value);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7491-7492
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7494-7498
```cpp
bool MipsAsmParser::parseSetMips0Directive() {
  MCAsmParser &Parser = getParser();
  Parser.Lex();
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return reportParseError("unexpected token, expected end of statement");
```
- EN: Implements `MipsAsmParser::parseSetMips0Directive`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetMips0Directive`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7500-7505
```cpp
  // Reset assembler options to their initial values.
  MCSubtargetInfo &STI = copySTI();
  setAvailableFeatures(
      ComputeAvailableFeatures(AssemblerOptions.front()->getFeatures()));
  STI.setFeatureBits(AssemblerOptions.front()->getFeatures());
  AssemblerOptions.back()->setFeatures(AssemblerOptions.front()->getFeatures());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7507-7509
```cpp
  getTargetStreamer().emitDirectiveSetMips0();
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7511-7515
```cpp
bool MipsAsmParser::parseSetArchDirective() {
  MCAsmParser &Parser = getParser();
  Parser.Lex();
  if (getLexer().isNot(AsmToken::Equal))
    return reportParseError("unexpected token, expected equals sign");
```
- EN: Implements `MipsAsmParser::parseSetArchDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetArchDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7517-7520
```cpp
  Parser.Lex();
  StringRef Arch = getParser().parseStringToEndOfStatement().trim();
  if (Arch.empty())
    return reportParseError("expected arch identifier");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7522-7536
```cpp
  StringRef ArchFeatureName =
      StringSwitch<StringRef>(Arch)
          .Case("mips1", "mips1")
          .Case("mips2", "mips2")
          .Case("mips3", "mips3")
          .Case("mips4", "mips4")
          .Case("mips5", "mips5")
          .Case("mips32", "mips32")
          .Case("mips32r2", "mips32r2")
          .Case("mips32r3", "mips32r3")
          .Case("mips32r5", "mips32r5")
          .Case("mips32r6", "mips32r6")
          .Case("mips64", "mips64")
          .Case("mips64r2", "mips64r2")
          .Case("mips64r3", "mips64r3")
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7537-7542
```cpp
          .Case("mips64r5", "mips64r5")
          .Case("mips64r6", "mips64r6")
          .Case("octeon", "cnmips")
          .Case("octeon+", "cnmipsp")
          .Case("r4000", "mips3") // This is an implementation of Mips3.
          .Default("");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7544-7545
```cpp
  if (ArchFeatureName.empty())
    return reportParseError("unsupported architecture");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7547-7548
```cpp
  if (ArchFeatureName == "mips64r6" && inMicroMipsMode())
    return reportParseError("mips64r6 does not support microMIPS");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7550-7553
```cpp
  selectArch(ArchFeatureName);
  getTargetStreamer().emitDirectiveSetArch(Arch);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7555-7559
```cpp
bool MipsAsmParser::parseSetFeature(uint64_t Feature) {
  MCAsmParser &Parser = getParser();
  Parser.Lex();
  if (getLexer().isNot(AsmToken::EndOfStatement))
    return reportParseError("unexpected token, expected end of statement");
```
- EN: Implements `MipsAsmParser::parseSetFeature`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSetFeature`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7561-7575
```cpp
  switch (Feature) {
  default:
    llvm_unreachable("Unimplemented feature");
  case Mips::FeatureMips3D:
    setFeatureBits(Mips::FeatureMips3D, "mips3d");
    getTargetStreamer().emitDirectiveSetMips3D();
    break;
  case Mips::FeatureDSP:
    setFeatureBits(Mips::FeatureDSP, "dsp");
    getTargetStreamer().emitDirectiveSetDsp();
    break;
  case Mips::FeatureDSPR2:
    setFeatureBits(Mips::FeatureDSPR2, "dspr2");
    getTargetStreamer().emitDirectiveSetDspr2();
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7576-7590
```cpp
  case Mips::FeatureMicroMips:
    setFeatureBits(Mips::FeatureMicroMips, "micromips");
    getTargetStreamer().emitDirectiveSetMicroMips();
    break;
  case Mips::FeatureMips1:
    selectArch("mips1");
    getTargetStreamer().emitDirectiveSetMips1();
    break;
  case Mips::FeatureMips2:
    selectArch("mips2");
    getTargetStreamer().emitDirectiveSetMips2();
    break;
  case Mips::FeatureMips3:
    selectArch("mips3");
    getTargetStreamer().emitDirectiveSetMips3();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7591-7605
```cpp
    break;
  case Mips::FeatureMips4:
    selectArch("mips4");
    getTargetStreamer().emitDirectiveSetMips4();
    break;
  case Mips::FeatureMips5:
    selectArch("mips5");
    getTargetStreamer().emitDirectiveSetMips5();
    break;
  case Mips::FeatureMips32:
    selectArch("mips32");
    getTargetStreamer().emitDirectiveSetMips32();
    break;
  case Mips::FeatureMips32r2:
    selectArch("mips32r2");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7606-7620
```cpp
    getTargetStreamer().emitDirectiveSetMips32R2();
    break;
  case Mips::FeatureMips32r3:
    selectArch("mips32r3");
    getTargetStreamer().emitDirectiveSetMips32R3();
    break;
  case Mips::FeatureMips32r5:
    selectArch("mips32r5");
    getTargetStreamer().emitDirectiveSetMips32R5();
    break;
  case Mips::FeatureMips32r6:
    selectArch("mips32r6");
    getTargetStreamer().emitDirectiveSetMips32R6();
    break;
  case Mips::FeatureMips64:
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7621-7635
```cpp
    selectArch("mips64");
    getTargetStreamer().emitDirectiveSetMips64();
    break;
  case Mips::FeatureMips64r2:
    selectArch("mips64r2");
    getTargetStreamer().emitDirectiveSetMips64R2();
    break;
  case Mips::FeatureMips64r3:
    selectArch("mips64r3");
    getTargetStreamer().emitDirectiveSetMips64R3();
    break;
  case Mips::FeatureMips64r5:
    selectArch("mips64r5");
    getTargetStreamer().emitDirectiveSetMips64R5();
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7636-7650
```cpp
  case Mips::FeatureMips64r6:
    selectArch("mips64r6");
    getTargetStreamer().emitDirectiveSetMips64R6();
    break;
  case Mips::FeatureCRC:
    setFeatureBits(Mips::FeatureCRC, "crc");
    getTargetStreamer().emitDirectiveSetCRC();
    break;
  case Mips::FeatureVirt:
    setFeatureBits(Mips::FeatureVirt, "virt");
    getTargetStreamer().emitDirectiveSetVirt();
    break;
  case Mips::FeatureGINV:
    setFeatureBits(Mips::FeatureGINV, "ginv");
    getTargetStreamer().emitDirectiveSetGINV();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7651-7654
```cpp
    break;
  }
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7656-7661
```cpp
bool MipsAsmParser::eatComma(StringRef ErrorStr) {
  MCAsmParser &Parser = getParser();
  if (getLexer().isNot(AsmToken::Comma)) {
    SMLoc Loc = getLexer().getLoc();
    return Error(Loc, ErrorStr);
  }
```
- EN: Implements `MipsAsmParser::eatComma`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::eatComma`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7663-7665
```cpp
  Parser.Lex(); // Eat the comma.
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7667-7673
```cpp
// Used to determine if .cpload, .cprestore, and .cpsetup have any effect.
// In this class, it is only used for .cprestore.
// FIXME: Only keep track of IsPicEnabled in one place, instead of in both
// MipsTargetELFStreamer and MipsAsmParser.
bool MipsAsmParser::isPicAndNotNxxAbi() {
  return inPicMode() && !(isABI_N32() || isABI_N64());
}
```
- EN: Implements `MipsAsmParser::isPicAndNotNxxAbi`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::isPicAndNotNxxAbi`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7675-7681
```cpp
bool MipsAsmParser::parseDirectiveCpAdd(SMLoc Loc) {
  SmallVector<std::unique_ptr<MCParsedAsmOperand>, 1> Reg;
  ParseStatus Res = parseAnyRegister(Reg);
  if (Res.isNoMatch() || Res.isFailure()) {
    reportParseError("expected register");
    return false;
  }
```
- EN: Implements `MipsAsmParser::parseDirectiveCpAdd`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveCpAdd`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7683-7687
```cpp
  MipsOperand &RegOpnd = static_cast<MipsOperand &>(*Reg[0]);
  if (!RegOpnd.isGPRAsmReg()) {
    reportParseError(RegOpnd.getStartLoc(), "invalid register");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7689-7694
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
  getParser().Lex(); // Consume the EndOfStatement.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7696-7698
```cpp
  getTargetStreamer().emitDirectiveCpAdd(RegOpnd.getGPR32Reg());
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7700-7702
```cpp
bool MipsAsmParser::parseDirectiveCpLoad(SMLoc Loc) {
  if (AssemblerOptions.back()->isReorder())
    Warning(Loc, ".cpload should be inside a noreorder section");
```
- EN: Implements `MipsAsmParser::parseDirectiveCpLoad`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveCpLoad`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7704-7707
```cpp
  if (inMips16Mode()) {
    reportParseError(".cpload is not supported in Mips16 mode");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7709-7714
```cpp
  SmallVector<std::unique_ptr<MCParsedAsmOperand>, 1> Reg;
  ParseStatus Res = parseAnyRegister(Reg);
  if (Res.isNoMatch() || Res.isFailure()) {
    reportParseError("expected register containing function address");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7716-7720
```cpp
  MipsOperand &RegOpnd = static_cast<MipsOperand &>(*Reg[0]);
  if (!RegOpnd.isGPRAsmReg()) {
    reportParseError(RegOpnd.getStartLoc(), "invalid register");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7722-7726
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7728-7730
```cpp
  getTargetStreamer().emitDirectiveCpLoad(RegOpnd.getGPR32Reg());
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7732-7736
```cpp
bool MipsAsmParser::parseDirectiveCpLocal(SMLoc Loc) {
  if (!isABI_N32() && !isABI_N64()) {
    reportParseError(".cplocal is allowed only in N32 or N64 mode");
    return false;
  }
```
- EN: Implements `MipsAsmParser::parseDirectiveCpLocal`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveCpLocal`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7738-7743
```cpp
  SmallVector<std::unique_ptr<MCParsedAsmOperand>, 1> Reg;
  ParseStatus Res = parseAnyRegister(Reg);
  if (Res.isNoMatch() || Res.isFailure()) {
    reportParseError("expected register containing global pointer");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7745-7749
```cpp
  MipsOperand &RegOpnd = static_cast<MipsOperand &>(*Reg[0]);
  if (!RegOpnd.isGPRAsmReg()) {
    reportParseError(RegOpnd.getStartLoc(), "invalid register");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7751-7756
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
  getParser().Lex(); // Consume the EndOfStatement.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7758-7760
```cpp
  MCRegister NewReg = RegOpnd.getGPR32Reg();
  if (IsPicEnabled)
    GPReg = NewReg;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7762-7764
```cpp
  getTargetStreamer().emitDirectiveCpLocal(NewReg);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7766-7767
```cpp
bool MipsAsmParser::parseDirectiveCpRestore(SMLoc Loc) {
  MCAsmParser &Parser = getParser();
```
- EN: Implements `MipsAsmParser::parseDirectiveCpRestore`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveCpRestore`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7769-7770
```cpp
  // Note that .cprestore is ignored if used with the N32 and N64 ABIs or if it
  // is used in non-PIC mode.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 7772-7775
```cpp
  if (inMips16Mode()) {
    reportParseError(".cprestore is not supported in Mips16 mode");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7777-7783
```cpp
  // Get the stack offset value.
  const MCExpr *StackOffset;
  int64_t StackOffsetVal;
  if (Parser.parseExpression(StackOffset)) {
    reportParseError("expected stack offset value");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7785-7788
```cpp
  if (!StackOffset->evaluateAsAbsolute(StackOffsetVal)) {
    reportParseError("stack offset is not an absolute expression");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7790-7796
```cpp
  if (StackOffsetVal < 0) {
    Warning(Loc, ".cprestore with negative stack offset has no effect");
    IsCpRestoreSet = false;
  } else {
    IsCpRestoreSet = true;
    CpRestoreOffset = StackOffsetVal;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7798-7802
```cpp
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7804-7809
```cpp
  if (!getTargetStreamer().emitDirectiveCpRestore(
          CpRestoreOffset, [&]() { return getATReg(Loc); }, Loc, STI))
    return true;
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7811-7814
```cpp
bool MipsAsmParser::parseDirectiveCPSetup() {
  MCAsmParser &Parser = getParser();
  unsigned Save;
  bool SaveIsReg = true;
```
- EN: Implements `MipsAsmParser::parseDirectiveCPSetup`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveCPSetup`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7816-7821
```cpp
  SmallVector<std::unique_ptr<MCParsedAsmOperand>, 1> TmpReg;
  ParseStatus Res = parseAnyRegister(TmpReg);
  if (Res.isNoMatch()) {
    reportParseError("expected register containing function address");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7823-7827
```cpp
  MipsOperand &FuncRegOpnd = static_cast<MipsOperand &>(*TmpReg[0]);
  if (!FuncRegOpnd.isGPRAsmReg()) {
    reportParseError(FuncRegOpnd.getStartLoc(), "invalid register");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7829-7830
```cpp
  MCRegister FuncReg = FuncRegOpnd.getGPR32Reg();
  TmpReg.clear();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7832-7833
```cpp
  if (!eatComma("unexpected token, expected comma"))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7835-7839
```cpp
  Res = parseAnyRegister(TmpReg);
  if (Res.isNoMatch()) {
    const MCExpr *OffsetExpr;
    int64_t OffsetVal;
    SMLoc ExprLoc = getLexer().getLoc();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7841-7845
```cpp
    if (Parser.parseExpression(OffsetExpr) ||
        !OffsetExpr->evaluateAsAbsolute(OffsetVal)) {
      reportParseError(ExprLoc, "expected save register or stack offset");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7847-7856
```cpp
    Save = OffsetVal;
    SaveIsReg = false;
  } else {
    MipsOperand &SaveOpnd = static_cast<MipsOperand &>(*TmpReg[0]);
    if (!SaveOpnd.isGPRAsmReg()) {
      reportParseError(SaveOpnd.getStartLoc(), "invalid register");
      return false;
    }
    Save = SaveOpnd.getGPR32Reg().id();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7858-7859
```cpp
  if (!eatComma("unexpected token, expected comma"))
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7861-7865
```cpp
  const MCExpr *Expr;
  if (Parser.parseExpression(Expr)) {
    reportParseError("expected expression");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7867-7871
```cpp
  if (Expr->getKind() != MCExpr::SymbolRef) {
    reportParseError("expected symbol");
    return false;
  }
  const MCSymbolRefExpr *Ref = static_cast<const MCSymbolRefExpr *>(Expr);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7873-7874
```cpp
  CpSaveLocation = Save;
  CpSaveLocationIsRegister = SaveIsReg;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7876-7879
```cpp
  getTargetStreamer().emitDirectiveCpsetup(FuncReg, Save, Ref->getSymbol(),
                                           SaveIsReg);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 7881-7885
```cpp
bool MipsAsmParser::parseDirectiveCPReturn() {
  getTargetStreamer().emitDirectiveCpreturn(CpSaveLocation,
                                            CpSaveLocationIsRegister);
  return false;
}
```
- EN: Implements `MipsAsmParser::parseDirectiveCPReturn`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveCPReturn`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7887-7890
```cpp
bool MipsAsmParser::parseDirectiveNaN() {
  MCAsmParser &Parser = getParser();
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    const AsmToken &Tok = Parser.getTok();
```
- EN: Implements `MipsAsmParser::parseDirectiveNaN`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveNaN`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7892-7906
```cpp
    if (Tok.getString() == "2008") {
      Parser.Lex();
      getTargetStreamer().emitDirectiveNaN2008();
      return false;
    } else if (Tok.getString() == "legacy") {
      Parser.Lex();
      getTargetStreamer().emitDirectiveNaNLegacy();
      return false;
    }
  }
  // If we don't recognize the option passed to the .nan
  // directive (e.g. no option or unknown option), emit an error.
  reportParseError("invalid option in .nan directive");
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7908-7911
```cpp
bool MipsAsmParser::parseDirectiveSet() {
  const AsmToken &Tok = getParser().getTok();
  StringRef IdVal = Tok.getString();
  SMLoc Loc = Tok.getLoc();
```
- EN: Implements `MipsAsmParser::parseDirectiveSet`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveSet`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 7913-7927
```cpp
  if (IdVal == "noat")
    return parseSetNoAtDirective();
  if (IdVal == "at")
    return parseSetAtDirective();
  if (IdVal == "arch")
    return parseSetArchDirective();
  if (IdVal == "bopt") {
    Warning(Loc, "'bopt' feature is unsupported");
    getParser().Lex();
    return false;
  }
  if (IdVal == "nobopt") {
    // We're already running in nobopt mode, so nothing to do.
    getParser().Lex();
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7928-7942
```cpp
  }
  if (IdVal == "fp")
    return parseSetFpDirective();
  if (IdVal == "oddspreg")
    return parseSetOddSPRegDirective();
  if (IdVal == "nooddspreg")
    return parseSetNoOddSPRegDirective();
  if (IdVal == "pop")
    return parseSetPopDirective();
  if (IdVal == "push")
    return parseSetPushDirective();
  if (IdVal == "reorder")
    return parseSetReorderDirective();
  if (IdVal == "noreorder")
    return parseSetNoReorderDirective();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7943-7957
```cpp
  if (IdVal == "macro")
    return parseSetMacroDirective();
  if (IdVal == "nomacro")
    return parseSetNoMacroDirective();
  if (IdVal == "mips16")
    return parseSetMips16Directive();
  if (IdVal == "nomips16")
    return parseSetNoMips16Directive();
  if (IdVal == "nomicromips") {
    clearFeatureBits(Mips::FeatureMicroMips, "micromips");
    getTargetStreamer().emitDirectiveSetNoMicroMips();
    getParser().eatToEndOfStatement();
    return false;
  }
  if (IdVal == "micromips") {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7958-7972
```cpp
    if (hasMips64r6()) {
      Error(Loc, ".set micromips directive is not supported with MIPS64R6");
      return false;
    }
    return parseSetFeature(Mips::FeatureMicroMips);
  }
  if (IdVal == "mips0")
    return parseSetMips0Directive();
  if (IdVal == "mips1")
    return parseSetFeature(Mips::FeatureMips1);
  if (IdVal == "mips2")
    return parseSetFeature(Mips::FeatureMips2);
  if (IdVal == "mips3")
    return parseSetFeature(Mips::FeatureMips3);
  if (IdVal == "mips4")
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7973-7987
```cpp
    return parseSetFeature(Mips::FeatureMips4);
  if (IdVal == "mips5")
    return parseSetFeature(Mips::FeatureMips5);
  if (IdVal == "mips32")
    return parseSetFeature(Mips::FeatureMips32);
  if (IdVal == "mips32r2")
    return parseSetFeature(Mips::FeatureMips32r2);
  if (IdVal == "mips32r3")
    return parseSetFeature(Mips::FeatureMips32r3);
  if (IdVal == "mips32r5")
    return parseSetFeature(Mips::FeatureMips32r5);
  if (IdVal == "mips32r6")
    return parseSetFeature(Mips::FeatureMips32r6);
  if (IdVal == "mips64")
    return parseSetFeature(Mips::FeatureMips64);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 7988-8002
```cpp
  if (IdVal == "mips64r2")
    return parseSetFeature(Mips::FeatureMips64r2);
  if (IdVal == "mips64r3")
    return parseSetFeature(Mips::FeatureMips64r3);
  if (IdVal == "mips64r5")
    return parseSetFeature(Mips::FeatureMips64r5);
  if (IdVal == "mips64r6") {
    if (inMicroMipsMode()) {
      Error(Loc, "MIPS64R6 is not supported with microMIPS");
      return false;
    }
    return parseSetFeature(Mips::FeatureMips64r6);
  }
  if (IdVal == "dsp")
    return parseSetFeature(Mips::FeatureDSP);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8003-8017
```cpp
  if (IdVal == "dspr2")
    return parseSetFeature(Mips::FeatureDSPR2);
  if (IdVal == "nodsp")
    return parseSetNoDspDirective();
  if (IdVal == "mips3d")
    return parseSetFeature(Mips::FeatureMips3D);
  if (IdVal == "nomips3d")
    return parseSetNoMips3DDirective();
  if (IdVal == "msa")
    return parseSetMsaDirective();
  if (IdVal == "nomsa")
    return parseSetNoMsaDirective();
  if (IdVal == "mt")
    return parseSetMtDirective();
  if (IdVal == "nomt")
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8018-8032
```cpp
    return parseSetNoMtDirective();
  if (IdVal == "softfloat")
    return parseSetSoftFloatDirective();
  if (IdVal == "hardfloat")
    return parseSetHardFloatDirective();
  if (IdVal == "crc")
    return parseSetFeature(Mips::FeatureCRC);
  if (IdVal == "nocrc")
    return parseSetNoCRCDirective();
  if (IdVal == "virt")
    return parseSetFeature(Mips::FeatureVirt);
  if (IdVal == "novirt")
    return parseSetNoVirtDirective();
  if (IdVal == "ginv")
    return parseSetFeature(Mips::FeatureGINV);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8033-8034
```cpp
  if (IdVal == "noginv")
    return parseSetNoGINVDirective();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8036-8038
```cpp
  // It is just an identifier, look for an assignment.
  return parseSetAssignment();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8040-8048
```cpp
/// parseDirectiveGpWord
///  ::= .gpword local_sym
bool MipsAsmParser::parseDirectiveGpWord() {
  const MCExpr *Value;
  if (getParser().parseExpression(Value))
    return true;
  getTargetStreamer().emitGPRel32Value(Value);
  return parseEOL();
}
```
- EN: Implements `MipsAsmParser::parseDirectiveGpWord`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveGpWord`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8050-8058
```cpp
/// parseDirectiveGpDWord
///  ::= .gpdword local_sym
bool MipsAsmParser::parseDirectiveGpDWord() {
  const MCExpr *Value;
  if (getParser().parseExpression(Value))
    return true;
  getTargetStreamer().emitGPRel64Value(Value);
  return parseEOL();
}
```
- EN: Implements `MipsAsmParser::parseDirectiveGpDWord`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveGpDWord`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8060-8068
```cpp
/// parseDirectiveDtpRelWord
///  ::= .dtprelword tls_sym
bool MipsAsmParser::parseDirectiveDtpRelWord() {
  const MCExpr *Value;
  if (getParser().parseExpression(Value))
    return true;
  getTargetStreamer().emitDTPRel32Value(Value);
  return parseEOL();
}
```
- EN: Implements `MipsAsmParser::parseDirectiveDtpRelWord`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveDtpRelWord`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8070-8078
```cpp
/// parseDirectiveDtpRelDWord
///  ::= .dtpreldword tls_sym
bool MipsAsmParser::parseDirectiveDtpRelDWord() {
  const MCExpr *Value;
  if (getParser().parseExpression(Value))
    return true;
  getTargetStreamer().emitDTPRel64Value(Value);
  return parseEOL();
}
```
- EN: Implements `MipsAsmParser::parseDirectiveDtpRelDWord`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveDtpRelDWord`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8080-8088
```cpp
/// parseDirectiveTpRelWord
///  ::= .tprelword tls_sym
bool MipsAsmParser::parseDirectiveTpRelWord() {
  const MCExpr *Value;
  if (getParser().parseExpression(Value))
    return true;
  getTargetStreamer().emitTPRel32Value(Value);
  return parseEOL();
}
```
- EN: Implements `MipsAsmParser::parseDirectiveTpRelWord`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveTpRelWord`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8090-8098
```cpp
/// parseDirectiveTpRelDWord
///  ::= .tpreldword tls_sym
bool MipsAsmParser::parseDirectiveTpRelDWord() {
  const MCExpr *Value;
  if (getParser().parseExpression(Value))
    return true;
  getTargetStreamer().emitTPRel64Value(Value);
  return parseEOL();
}
```
- EN: Implements `MipsAsmParser::parseDirectiveTpRelDWord`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveTpRelDWord`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8100-8108
```cpp
bool MipsAsmParser::parseDirectiveOption() {
  MCAsmParser &Parser = getParser();
  // Get the option token.
  AsmToken Tok = Parser.getTok();
  // At the moment only identifiers are supported.
  if (Tok.isNot(AsmToken::Identifier)) {
    return Error(Parser.getTok().getLoc(),
                 "unexpected token, expected identifier");
  }
```
- EN: Implements `MipsAsmParser::parseDirectiveOption`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveOption`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8110-8110
```cpp
  StringRef Option = Tok.getIdentifier();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8112-8114
```cpp
  if (Option == "pic0") {
    // MipsAsmParser needs to know if the current PIC mode changes.
    IsPicEnabled = false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8116-8123
```cpp
    getTargetStreamer().emitDirectiveOptionPic0();
    Parser.Lex();
    if (Parser.getTok().isNot(AsmToken::EndOfStatement)) {
      return Error(Parser.getTok().getLoc(),
                   "unexpected token, expected end of statement");
    }
    return false;
  }
```
- EN: Implements `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8125-8127
```cpp
  if (Option == "pic2") {
    // MipsAsmParser needs to know if the current PIC mode changes.
    IsPicEnabled = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8129-8136
```cpp
    getTargetStreamer().emitDirectiveOptionPic2();
    Parser.Lex();
    if (Parser.getTok().isNot(AsmToken::EndOfStatement)) {
      return Error(Parser.getTok().getLoc(),
                   "unexpected token, expected end of statement");
    }
    return false;
  }
```
- EN: Implements `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8138-8143
```cpp
  // Unknown option.
  Warning(Parser.getTok().getLoc(),
          "unknown option, expected 'pic0' or 'pic2'");
  Parser.eatToEndOfStatement();
  return false;
}
```
- EN: Declares `Warning`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Warning`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8145-8152
```cpp
/// parseInsnDirective
///  ::= .insn
bool MipsAsmParser::parseInsnDirective() {
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Implements `MipsAsmParser::parseInsnDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseInsnDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8154-8156
```cpp
  // The actual label marking happens in
  // MipsELFStreamer::createPendingLabelRelocs().
  getTargetStreamer().emitDirectiveInsn();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8158-8160
```cpp
  getParser().Lex(); // Eat EndOfStatement token.
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8162-8169
```cpp
/// parseRSectionDirective
///  ::= .rdata
bool MipsAsmParser::parseRSectionDirective(StringRef Section) {
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Implements `MipsAsmParser::parseRSectionDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseRSectionDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8171-8173
```cpp
  MCSection *ELFSection = getContext().getELFSection(
      Section, ELF::SHT_PROGBITS, ELF::SHF_ALLOC);
  getParser().getStreamer().switchSection(ELFSection);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8175-8177
```cpp
  getParser().Lex(); // Eat EndOfStatement token.
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8179-8187
```cpp
/// parseSSectionDirective
///  ::= .sbss
///  ::= .sdata
bool MipsAsmParser::parseSSectionDirective(StringRef Section, unsigned Type) {
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Implements `MipsAsmParser::parseSSectionDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseSSectionDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8189-8191
```cpp
  MCSection *ELFSection = getContext().getELFSection(
      Section, Type, ELF::SHF_WRITE | ELF::SHF_ALLOC | ELF::SHF_MIPS_GPREL);
  getParser().getStreamer().switchSection(ELFSection);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8193-8195
```cpp
  getParser().Lex(); // Eat EndOfStatement token.
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8197-8211
```cpp
/// parseDirectiveModule
///  ::= .module oddspreg
///  ::= .module nooddspreg
///  ::= .module fp=value
///  ::= .module softfloat
///  ::= .module hardfloat
///  ::= .module mt
///  ::= .module crc
///  ::= .module nocrc
///  ::= .module virt
///  ::= .module novirt
///  ::= .module ginv
///  ::= .module noginv
bool MipsAsmParser::parseDirectiveModule() {
  MCAsmParser &Parser = getParser();
```
- EN: Implements `MipsAsmParser::parseDirectiveModule`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveModule`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8212-8213
```cpp
  AsmLexer &Lexer = getLexer();
  SMLoc L = Lexer.getLoc();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8215-8219
```cpp
  if (!getTargetStreamer().isModuleDirectiveAllowed()) {
    // TODO : get a better message.
    reportParseError(".module directive must appear before any code");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8221-8225
```cpp
  StringRef Option;
  if (Parser.parseIdentifier(Option)) {
    reportParseError("expected .module option identifier");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8227-8228
```cpp
  if (Option == "oddspreg") {
    clearModuleFeatureBits(Mips::FeatureNoOddSPReg, "nooddspreg");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8230-8232
```cpp
    // Synchronize the abiflags information with the FeatureBits information we
    // changed above.
    getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8234-8237
```cpp
    // If printing assembly, use the recently updated abiflags information.
    // If generating ELF, don't do anything (the .MIPS.abiflags section gets
    // emitted at the end).
    getTargetStreamer().emitDirectiveModuleOddSPReg();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8239-8243
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8245-8249
```cpp
    return false; // parseDirectiveModule has finished successfully.
  } else if (Option == "nooddspreg") {
    if (!isABI_O32()) {
      return Error(L, "'.module nooddspreg' requires the O32 ABI");
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8251-8251
```cpp
    setModuleFeatureBits(Mips::FeatureNoOddSPReg, "nooddspreg");
```
- EN: Declares `setModuleFeatureBits`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setModuleFeatureBits`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8253-8255
```cpp
    // Synchronize the abiflags information with the FeatureBits information we
    // changed above.
    getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8257-8260
```cpp
    // If printing assembly, use the recently updated abiflags information.
    // If generating ELF, don't do anything (the .MIPS.abiflags section gets
    // emitted at the end).
    getTargetStreamer().emitDirectiveModuleOddSPReg();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8262-8266
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8268-8272
```cpp
    return false; // parseDirectiveModule has finished successfully.
  } else if (Option == "fp") {
    return parseDirectiveModuleFP();
  } else if (Option == "softfloat") {
    setModuleFeatureBits(Mips::FeatureSoftFloat, "soft-float");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8274-8276
```cpp
    // Synchronize the ABI Flags information with the FeatureBits information we
    // updated above.
    getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8278-8281
```cpp
    // If printing assembly, use the recently updated ABI Flags information.
    // If generating ELF, don't do anything (the .MIPS.abiflags section gets
    // emitted later).
    getTargetStreamer().emitDirectiveModuleSoftFloat();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8283-8287
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8289-8291
```cpp
    return false; // parseDirectiveModule has finished successfully.
  } else if (Option == "hardfloat") {
    clearModuleFeatureBits(Mips::FeatureSoftFloat, "soft-float");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8293-8295
```cpp
    // Synchronize the ABI Flags information with the FeatureBits information we
    // updated above.
    getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8297-8300
```cpp
    // If printing assembly, use the recently updated ABI Flags information.
    // If generating ELF, don't do anything (the .MIPS.abiflags section gets
    // emitted later).
    getTargetStreamer().emitDirectiveModuleHardFloat();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8302-8306
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8308-8310
```cpp
    return false; // parseDirectiveModule has finished successfully.
  } else if (Option == "mt") {
    setModuleFeatureBits(Mips::FeatureMT, "mt");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8312-8314
```cpp
    // Synchronize the ABI Flags information with the FeatureBits information we
    // updated above.
    getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8316-8319
```cpp
    // If printing assembly, use the recently updated ABI Flags information.
    // If generating ELF, don't do anything (the .MIPS.abiflags section gets
    // emitted later).
    getTargetStreamer().emitDirectiveModuleMT();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8321-8325
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8327-8329
```cpp
    return false; // parseDirectiveModule has finished successfully.
  } else if (Option == "crc") {
    setModuleFeatureBits(Mips::FeatureCRC, "crc");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8331-8333
```cpp
    // Synchronize the ABI Flags information with the FeatureBits information we
    // updated above.
    getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8335-8338
```cpp
    // If printing assembly, use the recently updated ABI Flags information.
    // If generating ELF, don't do anything (the .MIPS.abiflags section gets
    // emitted later).
    getTargetStreamer().emitDirectiveModuleCRC();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8340-8344
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8346-8348
```cpp
    return false; // parseDirectiveModule has finished successfully.
  } else if (Option == "nocrc") {
    clearModuleFeatureBits(Mips::FeatureCRC, "crc");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8350-8352
```cpp
    // Synchronize the ABI Flags information with the FeatureBits information we
    // updated above.
    getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8354-8357
```cpp
    // If printing assembly, use the recently updated ABI Flags information.
    // If generating ELF, don't do anything (the .MIPS.abiflags section gets
    // emitted later).
    getTargetStreamer().emitDirectiveModuleNoCRC();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8359-8363
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8365-8367
```cpp
    return false; // parseDirectiveModule has finished successfully.
  } else if (Option == "virt") {
    setModuleFeatureBits(Mips::FeatureVirt, "virt");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8369-8371
```cpp
    // Synchronize the ABI Flags information with the FeatureBits information we
    // updated above.
    getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8373-8376
```cpp
    // If printing assembly, use the recently updated ABI Flags information.
    // If generating ELF, don't do anything (the .MIPS.abiflags section gets
    // emitted later).
    getTargetStreamer().emitDirectiveModuleVirt();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8378-8382
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8384-8386
```cpp
    return false; // parseDirectiveModule has finished successfully.
  } else if (Option == "novirt") {
    clearModuleFeatureBits(Mips::FeatureVirt, "virt");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8388-8390
```cpp
    // Synchronize the ABI Flags information with the FeatureBits information we
    // updated above.
    getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8392-8395
```cpp
    // If printing assembly, use the recently updated ABI Flags information.
    // If generating ELF, don't do anything (the .MIPS.abiflags section gets
    // emitted later).
    getTargetStreamer().emitDirectiveModuleNoVirt();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8397-8401
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8403-8405
```cpp
    return false; // parseDirectiveModule has finished successfully.
  } else if (Option == "ginv") {
    setModuleFeatureBits(Mips::FeatureGINV, "ginv");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8407-8409
```cpp
    // Synchronize the ABI Flags information with the FeatureBits information we
    // updated above.
    getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8411-8414
```cpp
    // If printing assembly, use the recently updated ABI Flags information.
    // If generating ELF, don't do anything (the .MIPS.abiflags section gets
    // emitted later).
    getTargetStreamer().emitDirectiveModuleGINV();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8416-8420
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8422-8424
```cpp
    return false; // parseDirectiveModule has finished successfully.
  } else if (Option == "noginv") {
    clearModuleFeatureBits(Mips::FeatureGINV, "ginv");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8426-8428
```cpp
    // Synchronize the ABI Flags information with the FeatureBits information we
    // updated above.
    getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8430-8433
```cpp
    // If printing assembly, use the recently updated ABI Flags information.
    // If generating ELF, don't do anything (the .MIPS.abiflags section gets
    // emitted later).
    getTargetStreamer().emitDirectiveModuleNoGINV();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8435-8439
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8441-8445
```cpp
    return false; // parseDirectiveModule has finished successfully.
  } else {
    return Error(L, "'" + Twine(Option) + "' is not a valid .module option.");
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8447-8453
```cpp
/// parseDirectiveModuleFP
///  ::= =32
///  ::= =xx
///  ::= =64
bool MipsAsmParser::parseDirectiveModuleFP() {
  MCAsmParser &Parser = getParser();
  AsmLexer &Lexer = getLexer();
```
- EN: Implements `MipsAsmParser::parseDirectiveModuleFP`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseDirectiveModuleFP`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8455-8459
```cpp
  if (Lexer.isNot(AsmToken::Equal)) {
    reportParseError("unexpected token, expected equals sign '='");
    return false;
  }
  Parser.Lex(); // Eat '=' token.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8461-8463
```cpp
  MipsABIFlagsSection::FpABIKind FpABI;
  if (!parseFpABIValue(FpABI, ".module"))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8465-8468
```cpp
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8470-8472
```cpp
  // Synchronize the abiflags information with the FeatureBits information we
  // changed above.
  getTargetStreamer().updateABIInfo(*this);
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8474-8477
```cpp
  // If printing assembly, use the recently updated abiflags information.
  // If generating ELF, don't do anything (the .MIPS.abiflags section gets
  // emitted at the end).
  getTargetStreamer().emitDirectiveModuleFP();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8479-8481
```cpp
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8483-8487
```cpp
bool MipsAsmParser::parseFpABIValue(MipsABIFlagsSection::FpABIKind &FpABI,
                                    StringRef Directive) {
  MCAsmParser &Parser = getParser();
  AsmLexer &Lexer = getLexer();
  bool ModuleLevelOptions = Directive == ".module";
```
- EN: Implements `MipsAsmParser::parseFpABIValue`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseFpABIValue`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8489-8491
```cpp
  if (Lexer.is(AsmToken::Identifier)) {
    StringRef Value = Parser.getTok().getString();
    Parser.Lex();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8493-8496
```cpp
    if (Value != "xx") {
      reportParseError("unsupported value, expected 'xx', '32' or '64'");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8498-8501
```cpp
    if (!isABI_O32()) {
      reportParseError("'" + Directive + " fp=xx' requires the O32 ABI");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8503-8512
```cpp
    FpABI = MipsABIFlagsSection::FpABIKind::XX;
    if (ModuleLevelOptions) {
      setModuleFeatureBits(Mips::FeatureFPXX, "fpxx");
      clearModuleFeatureBits(Mips::FeatureFP64Bit, "fp64");
    } else {
      setFeatureBits(Mips::FeatureFPXX, "fpxx");
      clearFeatureBits(Mips::FeatureFP64Bit, "fp64");
    }
    return true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8514-8516
```cpp
  if (Lexer.is(AsmToken::Integer)) {
    unsigned Value = Parser.getTok().getIntVal();
    Parser.Lex();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8518-8521
```cpp
    if (Value != 32 && Value != 64) {
      reportParseError("unsupported value, expected 'xx', '32' or '64'");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8523-8527
```cpp
    if (Value == 32) {
      if (!isABI_O32()) {
        reportParseError("'" + Directive + " fp=32' requires the O32 ABI");
        return false;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8529-8543
```cpp
      FpABI = MipsABIFlagsSection::FpABIKind::S32;
      if (ModuleLevelOptions) {
        clearModuleFeatureBits(Mips::FeatureFPXX, "fpxx");
        clearModuleFeatureBits(Mips::FeatureFP64Bit, "fp64");
      } else {
        clearFeatureBits(Mips::FeatureFPXX, "fpxx");
        clearFeatureBits(Mips::FeatureFP64Bit, "fp64");
      }
    } else {
      FpABI = MipsABIFlagsSection::FpABIKind::S64;
      if (ModuleLevelOptions) {
        clearModuleFeatureBits(Mips::FeatureFPXX, "fpxx");
        setModuleFeatureBits(Mips::FeatureFP64Bit, "fp64");
      } else {
        clearFeatureBits(Mips::FeatureFPXX, "fpxx");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8544-8546
```cpp
        setFeatureBits(Mips::FeatureFP64Bit, "fp64");
      }
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8548-8549
```cpp
    return true;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8551-8552
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8554-8558
```cpp
bool MipsAsmParser::ParseDirective(AsmToken DirectiveID) {
  // This returns false if this function recognizes the directive
  // regardless of whether it is successfully handles or reports an
  // error. Otherwise it returns true to give the generic parser a
  // chance at recognizing it.
```
- EN: Implements `MipsAsmParser::ParseDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::ParseDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8560-8561
```cpp
  MCAsmParser &Parser = getParser();
  StringRef IDVal = DirectiveID.getString();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8563-8577
```cpp
  if (IDVal == ".cpadd") {
    parseDirectiveCpAdd(DirectiveID.getLoc());
    return false;
  }
  if (IDVal == ".cpload") {
    parseDirectiveCpLoad(DirectiveID.getLoc());
    return false;
  }
  if (IDVal == ".cprestore") {
    parseDirectiveCpRestore(DirectiveID.getLoc());
    return false;
  }
  if (IDVal == ".cplocal") {
    parseDirectiveCpLocal(DirectiveID.getLoc());
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8578-8580
```cpp
  }
  if (IDVal == ".ent") {
    StringRef SymbolName;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8582-8585
```cpp
    if (Parser.parseIdentifier(SymbolName)) {
      reportParseError("expected identifier after .ent");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8587-8601
```cpp
    // There's an undocumented extension that allows an integer to
    // follow the name of the procedure which AFAICS is ignored by GAS.
    // Example: .ent foo,2
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      if (getLexer().isNot(AsmToken::Comma)) {
        // Even though we accept this undocumented extension for compatibility
        // reasons, the additional integer argument does not actually change
        // the behaviour of the '.ent' directive, so we would like to discourage
        // its use. We do this by not referring to the extended version in
        // error messages which are not directly related to its use.
        reportParseError("unexpected token, expected end of statement");
        return false;
      }
      Parser.Lex(); // Eat the comma.
      const MCExpr *DummyNumber;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8602-8613
```cpp
      int64_t DummyNumberVal;
      // If the user was explicitly trying to use the extended version,
      // we still give helpful extension-related error messages.
      if (Parser.parseExpression(DummyNumber)) {
        reportParseError("expected number after comma");
        return false;
      }
      if (!DummyNumber->evaluateAsAbsolute(DummyNumberVal)) {
        reportParseError("expected an absolute expression after comma");
        return false;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8615-8619
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8621-8621
```cpp
    MCSymbol *Sym = getContext().getOrCreateSymbol(SymbolName);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8623-8627
```cpp
    getTargetStreamer().emitDirectiveEnt(*Sym);
    CurrentFn = Sym;
    IsCpRestoreSet = false;
    return false;
  }
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8629-8630
```cpp
  if (IDVal == ".end") {
    StringRef SymbolName;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8632-8635
```cpp
    if (Parser.parseIdentifier(SymbolName)) {
      reportParseError("expected identifier after .end");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8637-8640
```cpp
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8642-8645
```cpp
    if (CurrentFn == nullptr) {
      reportParseError(".end used without .ent");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8647-8650
```cpp
    if ((SymbolName != CurrentFn->getName())) {
      reportParseError(".end symbol does not match .ent symbol");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8652-8656
```cpp
    getTargetStreamer().emitDirectiveEnd(SymbolName);
    CurrentFn = nullptr;
    IsCpRestoreSet = false;
    return false;
  }
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8658-8665
```cpp
  if (IDVal == ".frame") {
    // .frame $stack_reg, frame_size_in_bytes, $return_reg
    SmallVector<std::unique_ptr<MCParsedAsmOperand>, 1> TmpReg;
    ParseStatus Res = parseAnyRegister(TmpReg);
    if (Res.isNoMatch() || Res.isFailure()) {
      reportParseError("expected stack register");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8667-8673
```cpp
    MipsOperand &StackRegOpnd = static_cast<MipsOperand &>(*TmpReg[0]);
    if (!StackRegOpnd.isGPRAsmReg()) {
      reportParseError(StackRegOpnd.getStartLoc(),
                       "expected general purpose register");
      return false;
    }
    MCRegister StackReg = StackRegOpnd.getGPR32Reg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8675-8680
```cpp
    if (Parser.getTok().is(AsmToken::Comma))
      Parser.Lex();
    else {
      reportParseError("unexpected token, expected comma");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8682-8684
```cpp
    // Parse the frame size.
    const MCExpr *FrameSize;
    int64_t FrameSizeVal;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8686-8689
```cpp
    if (Parser.parseExpression(FrameSize)) {
      reportParseError("expected frame size value");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8691-8694
```cpp
    if (!FrameSize->evaluateAsAbsolute(FrameSizeVal)) {
      reportParseError("frame size not an absolute expression");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8696-8701
```cpp
    if (Parser.getTok().is(AsmToken::Comma))
      Parser.Lex();
    else {
      reportParseError("unexpected token, expected comma");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8703-8709
```cpp
    // Parse the return register.
    TmpReg.clear();
    Res = parseAnyRegister(TmpReg);
    if (Res.isNoMatch() || Res.isFailure()) {
      reportParseError("expected return register");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8711-8716
```cpp
    MipsOperand &ReturnRegOpnd = static_cast<MipsOperand &>(*TmpReg[0]);
    if (!ReturnRegOpnd.isGPRAsmReg()) {
      reportParseError(ReturnRegOpnd.getStartLoc(),
                       "expected general purpose register");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8718-8722
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8724-8728
```cpp
    getTargetStreamer().emitFrame(StackReg, FrameSizeVal,
                                  ReturnRegOpnd.getGPR32Reg());
    IsCpRestoreSet = false;
    return false;
  }
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8730-8733
```cpp
  if (IDVal == ".set") {
    parseDirectiveSet();
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8735-8743
```cpp
  if (IDVal == ".mask" || IDVal == ".fmask") {
    // .mask bitmask, frame_offset
    // bitmask: One bit for each register used.
    // frame_offset: Offset from Canonical Frame Address ($sp on entry) where
    //               first register is expected to be saved.
    // Examples:
    //   .mask 0x80000000, -4
    //   .fmask 0x80000000, -4
    //
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8745-8747
```cpp
    // Parse the bitmask
    const MCExpr *BitMask;
    int64_t BitMaskVal;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8749-8752
```cpp
    if (Parser.parseExpression(BitMask)) {
      reportParseError("expected bitmask value");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8754-8757
```cpp
    if (!BitMask->evaluateAsAbsolute(BitMaskVal)) {
      reportParseError("bitmask not an absolute expression");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8759-8764
```cpp
    if (Parser.getTok().is(AsmToken::Comma))
      Parser.Lex();
    else {
      reportParseError("unexpected token, expected comma");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8766-8768
```cpp
    // Parse the frame_offset
    const MCExpr *FrameOffset;
    int64_t FrameOffsetVal;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8770-8773
```cpp
    if (Parser.parseExpression(FrameOffset)) {
      reportParseError("expected frame offset value");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8775-8778
```cpp
    if (!FrameOffset->evaluateAsAbsolute(FrameOffsetVal)) {
      reportParseError("frame offset not an absolute expression");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8780-8784
```cpp
    // If this is not the end of the statement, report an error.
    if (getLexer().isNot(AsmToken::EndOfStatement)) {
      reportParseError("unexpected token, expected end of statement");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8786-8791
```cpp
    if (IDVal == ".mask")
      getTargetStreamer().emitMask(BitMaskVal, FrameOffsetVal);
    else
      getTargetStreamer().emitFMask(BitMaskVal, FrameOffsetVal);
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8793-8794
```cpp
  if (IDVal == ".nan")
    return parseDirectiveNaN();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8796-8799
```cpp
  if (IDVal == ".gpword") {
    parseDirectiveGpWord();
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8801-8804
```cpp
  if (IDVal == ".gpdword") {
    parseDirectiveGpDWord();
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8806-8809
```cpp
  if (IDVal == ".dtprelword") {
    parseDirectiveDtpRelWord();
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8811-8814
```cpp
  if (IDVal == ".dtpreldword") {
    parseDirectiveDtpRelDWord();
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8816-8819
```cpp
  if (IDVal == ".tprelword") {
    parseDirectiveTpRelWord();
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8821-8824
```cpp
  if (IDVal == ".tpreldword") {
    parseDirectiveTpRelDWord();
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8826-8829
```cpp
  if (IDVal == ".option") {
    parseDirectiveOption();
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8831-8838
```cpp
  if (IDVal == ".abicalls") {
    getTargetStreamer().emitDirectiveAbiCalls();
    if (Parser.getTok().isNot(AsmToken::EndOfStatement)) {
      Error(Parser.getTok().getLoc(),
            "unexpected token, expected end of statement");
    }
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8840-8854
```cpp
  if (IDVal == ".cpsetup") {
    parseDirectiveCPSetup();
    return false;
  }
  if (IDVal == ".cpreturn") {
    parseDirectiveCPReturn();
    return false;
  }
  if (IDVal == ".module") {
    parseDirectiveModule();
    return false;
  }
  if (IDVal == ".llvm_internal_mips_reallow_module_directive") {
    parseInternalDirectiveReallowModule();
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8855-8869
```cpp
  }
  if (IDVal == ".insn") {
    parseInsnDirective();
    return false;
  }
  if (IDVal == ".rdata") {
    parseRSectionDirective(".rodata");
    return false;
  }
  if (IDVal == ".sbss") {
    parseSSectionDirective(IDVal, ELF::SHT_NOBITS);
    return false;
  }
  if (IDVal == ".sdata") {
    parseSSectionDirective(IDVal, ELF::SHT_PROGBITS);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 8870-8871
```cpp
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8873-8874
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8876-8881
```cpp
bool MipsAsmParser::parseInternalDirectiveReallowModule() {
  // If this is not the end of the statement, report an error.
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    reportParseError("unexpected token, expected end of statement");
    return false;
  }
```
- EN: Implements `MipsAsmParser::parseInternalDirectiveReallowModule`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::parseInternalDirectiveReallowModule`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8883-8883
```cpp
  getTargetStreamer().reallowModuleDirective();
```
- EN: Declares `getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8885-8887
```cpp
  getParser().Lex(); // Eat EndOfStatement token.
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 8889-8895
```cpp
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeMipsAsmParser() {
  RegisterMCAsmParser<MipsAsmParser> X(getTheMipsTarget());
  RegisterMCAsmParser<MipsAsmParser> Y(getTheMipselTarget());
  RegisterMCAsmParser<MipsAsmParser> A(getTheMips64Target());
  RegisterMCAsmParser<MipsAsmParser> B(getTheMips64elTarget());
}
```
- EN: Implements `LLVMInitializeMipsAsmParser`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LLVMInitializeMipsAsmParser`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 8897-8900
```cpp
#define GET_REGISTER_MATCHER
#define GET_MATCHER_IMPLEMENTATION
#define GET_MNEMONIC_SPELL_CHECKER
#include "MipsGenAsmMatcher.inc"
```
- EN: Connects this file to TableGen-generated declarations so target-specific enums and helper tables become available to C++ code.
- CN: 这里把文件连接到 TableGen 生成的声明，使目标相关枚举和辅助表能够在 C++ 代码中使用。

### Lines 8902-8912
```cpp
bool MipsAsmParser::mnemonicIsValid(StringRef Mnemonic, unsigned VariantID) {
  // Find the appropriate table for this asm variant.
  const MatchEntry *Start, *End;
  switch (VariantID) {
  default: llvm_unreachable("invalid variant!");
  case 0: Start = std::begin(MatchTable0); End = std::end(MatchTable0); break;
  }
  // Search the table.
  auto MnemonicRange = std::equal_range(Start, End, Mnemonic, LessOpcode());
  return MnemonicRange.first != MnemonicRange.second;
}
```
- EN: Implements `MipsAsmParser::mnemonicIsValid`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmParser::mnemonicIsValid`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: assembly parsing and operand/instruction decoding.
  - CN: 核心职责：汇编解析以及操作数/指令解码。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/MipsABIFlagsSection.h`, `MCTargetDesc/MipsABIInfo.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsMCAsmInfo.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `MCTargetDesc/MipsTargetStreamer.h`, `TargetInfo/MipsTargetInfo.h`, `MipsGenAsmMatcher.inc` ... (+2 more).
  - CN: 后端本地头文件：`MCTargetDesc/MipsABIFlagsSection.h`, `MCTargetDesc/MipsABIInfo.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsMCAsmInfo.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `MCTargetDesc/MipsTargetStreamer.h`, `TargetInfo/MipsTargetInfo.h`, `MipsGenAsmMatcher.inc` ... (+2 more)。
- EN: LLVM infrastructure headers: `llvm/ADT/APFloat.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h` ... (+29 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/APFloat.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/ELF.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h` ... (+29 more)。
- EN: Standard/system headers: `algorithm`, `cassert`, `cstdint`, `memory`, `string`, `utility`.
  - CN: 标准库/系统头文件：`algorithm`, `cassert`, `cstdint`, `memory`, `string`, `utility`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
