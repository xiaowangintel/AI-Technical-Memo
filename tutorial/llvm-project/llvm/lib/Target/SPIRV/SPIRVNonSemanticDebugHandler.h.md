# SPIRVNonSemanticDebugHandler.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVNonSemanticDebugHandler.h`
- Repository: `llvm-project`
- Purpose (EN): This file declares SPIRVNonSemanticDebugHandler, a DebugHandlerBase subclass that emits NonSemantic.Shader.DebugInfo.100 instructions in the SPIR-V AsmPrinter.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVNonSemanticDebugHandler.h - NSDI AsmPrinter handler -*- C++
 2: //-*-===//
 3: //
 4: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 5: // See https://llvm.org/LICENSE.txt for license information.
 6: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 7: //
 8: //===----------------------------------------------------------------------===//
 9: //
10: // This file declares SPIRVNonSemanticDebugHandler, a DebugHandlerBase subclass
11: // that emits NonSemantic.Shader.DebugInfo.100 instructions in the SPIR-V
12: // AsmPrinter. It replaces SPIRVEmitNonSemanticDI, which was a
13: // MachineFunctionPass, with a handler that controls instruction placement
14: // directly instead of routing through SPIRVModuleAnalysis.
15: //
16: //===----------------------------------------------------------------------===//
17:
18: #ifndef LLVM_LIB_TARGET_SPIRV_SPIRVNONSEMANTICDEBUGHANDLER_H
19: #define LLVM_LIB_TARGET_SPIRV_SPIRVNONSEMANTICDEBUGHANDLER_H
20:
21: #include "MCTargetDesc/SPIRVBaseInfo.h"
22: #include "SPIRVModuleAnalysis.h"
23: #include "llvm/ADT/DenseMap.h"
24: #include "llvm/ADT/SetVector.h"
25: #include "llvm/ADT/SmallString.h"
26: #include "llvm/ADT/SmallVector.h"
27: #include "llvm/CodeGen/DebugHandlerBase.h"
28: #include "llvm/IR/DebugInfoMetadata.h"
29: #include "llvm/MC/MCInst.h"
30: #include "llvm/MC/MCRegister.h"
31:
32: namespace llvm {
33:
34: class SPIRVSubtarget;
35:
36: /// AsmPrinter handler that emits NonSemantic.Shader.DebugInfo.100 (NSDI)
37: /// instructions for the SPIR-V backend. Registered with SPIRVAsmPrinter when
38: /// the module contains debug info (llvm.dbg.cu).
39: ///
40: /// Call sequence:
```
- EN: This range defines or declares important types such as SPIRVSubtarget, shaping the data model used by SPIRVNonSemanticDebugHandler.h.
- CN: 这一段定义或声明了 SPIRVSubtarget 等关键类型，构成 SPIRVNonSemanticDebugHandler.h 使用的数据模型。

### Lines 41-80
```cpp
41: ///   beginModule()                    -- collect compile-unit metadata.
42: ///   prepareModuleOutput()            -- add extension + ext inst set to MAI.
43: ///   emitNonSemanticGlobalDebugInfo() -- emit DebugSource,
44: ///                                       DebugCompilationUnit, DebugTypeBasic,
45: ///                                       DebugTypePointer.
46: ///   beginFunctionImpl()              -- no-op (no per-function DI yet).
47: ///   endFunctionImpl()                -- no-op.
48: class SPIRVNonSemanticDebugHandler : public DebugHandlerBase {
49:   struct CompileUnitInfo {
50:     SmallString<128> FilePath;
51:     unsigned SpirvSourceLanguage = 0; // NonSemantic.Shader.DebugInfo.100 source
52:                                       // language code (section 4.3)
53:   };
54:   // TODO: When per-function NSDI emission is implemented, augment
55:   // CompileUnitInfo with the originating DICompileUnit pointer so that
56:   // Parent operands on DebugFunction and similar instructions can resolve
57:   // the compile unit's result register.
58:   SmallVector<CompileUnitInfo> CompileUnits;
59:   int64_t DwarfVersion = 0;
60:
61:   // Types referenced by debug variable records, collected in beginModule().
62:   SetVector<const DIBasicType *> BasicTypes;
63:   SetVector<const DIDerivedType *> PointerTypes;
64:
65:   // Cache of already-emitted i32 constants, keyed by value. Prevents
66:   // duplicate OpConstant instructions for the same integer value.
67:   DenseMap<uint32_t, MCRegister> I32ConstantCache;
68:
69:   // OpString registers for NSDI instructions, populated by
70:   // emitNonSemanticDebugStrings() (section 7) and consumed by
71:   // emitNonSemanticGlobalDebugInfo() (section 10). OpString must appear in
72:   // section 7 per the SPIR-V module layout; it cannot be emitted alongside the
73:   // OpExtInst instructions in section 10.
74:   SmallVector<MCRegister> FileStringRegs;    // one per CompileUnits entry
75:   SmallVector<MCRegister> BasicTypeNameRegs; // one per BasicTypes entry
76:
77:   // True once emitNonSemanticGlobalDebugInfo() has run. Both
78:   // SPIRVAsmPrinter::emitFunctionHeader() and emitEndOfAsmFile() may call
79:   // outputModuleSections(), each guarded by ModuleSectionsEmitted, so only
80:   // one fires. This flag provides a secondary guard in case the call sites
```
- EN: This range defines or declares important types such as SPIRVNonSemanticDebugHandler, CompileUnitInfo, shaping the data model used by SPIRVNonSemanticDebugHandler.h.
- CN: 这一段定义或声明了 SPIRVNonSemanticDebugHandler、CompileUnitInfo 等关键类型，构成 SPIRVNonSemanticDebugHandler.h 使用的数据模型。

### Lines 81-120
```cpp
 81:   // change.
 82:   bool GlobalDIEmitted = false;
 83:
 84: public:
 85:   explicit SPIRVNonSemanticDebugHandler(AsmPrinter &AP);
 86:
 87:   /// Collect compile-unit metadata from the module. Called by
 88:   /// AsmPrinter::doInitialization() via the handler list. No emission.
 89:   void beginModule(Module *M) override;
 90:
 91:   /// Emit OpString instructions for all NSDI file paths and basic type names
 92:   /// into the debug section (section 7 of the SPIR-V module layout). Must be
 93:   /// called from SPIRVAsmPrinter::outputDebugSourceAndStrings(), after
 94:   /// prepareModuleOutput() has registered the ext inst set. The resulting
 95:   /// registers are cached in FileStringRegs and BasicTypeNameRegs for use by
 96:   /// emitNonSemanticGlobalDebugInfo().
 97:   void emitNonSemanticDebugStrings(SPIRV::ModuleAnalysisInfo &MAI);
 98:
 99:   /// Add SPV_KHR_non_semantic_info extension and
100:   /// NonSemantic.Shader.DebugInfo.100 ext inst set entry to MAI. Must be called
101:   /// before outputGlobalRequirements() and outputOpExtInstImports() in
102:   /// SPIRVAsmPrinter::outputModuleSections().
103:   void prepareModuleOutput(const SPIRVSubtarget &ST,
104:                            SPIRV::ModuleAnalysisInfo &MAI);
105:
106:   /// Emit module-scope NSDI instructions (DebugSource, DebugCompilationUnit,
107:   /// DebugTypeBasic, DebugTypePointer). Called by SPIRVAsmPrinter::
108:   /// outputModuleSections() at section 10 in place of
109:   /// outputModuleSection(MB_NonSemanticGlobalDI).
110:   void emitNonSemanticGlobalDebugInfo(SPIRV::ModuleAnalysisInfo &MAI);
111:
112: protected:
113:   // All module-level output is driven by emitNonSemanticGlobalDebugInfo(),
114:   // called explicitly from SPIRVAsmPrinter::outputModuleSections(). Nothing
115:   // needs to happen in the AsmPrinterHandler::endModule() callback.
116:   void endModule() override {}
117:
118:   // DebugHandlerBase stores MMI as a pointer copy from Asm->MMI at construction
119:   // time (DebugHandlerBase.cpp: `MMI(Asm->MMI)`). The handler is constructed
120:   // before AsmPrinter::doInitialization() runs, so Asm->MMI is null at that
```
- EN: This range declares interfaces or inline helpers such as SPIRVNonSemanticDebugHandler, beginModule, emitNonSemanticDebugStrings, emitNonSemanticGlobalDebugInfo, defining how other backend pieces interact with this header.
- CN: 这一段声明了 SPIRVNonSemanticDebugHandler、beginModule、emitNonSemanticDebugStrings、emitNonSemanticGlobalDebugInfo 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 121-160
```cpp
121:   // point and MMI remains null for this handler's entire lifetime. The
122:   // base-class beginInstruction/endInstruction dereference MMI to create temp
123:   // symbols for label tracking and would crash. Override them as no-ops.
124:   // When per-function NSDI is implemented, use Asm->OutStreamer->getContext()
125:   // for MCContext access rather than MMI->getContext().
126:   void beginInstruction(const MachineInstr *MI) override {}
127:   void endInstruction() override {}
128:
129:   // TODO: Emit DebugFunction and DebugFunctionDefinition here once per-function
130:   // NSDI emission is implemented. DebugHandlerBase::beginFunction() populates
131:   // LScopes and DbgValues, which are needed for DebugLine emission. Do not
132:   // override beginFunction() until that work is in place.
133:   void beginFunctionImpl(const MachineFunction *MF) override {}
134:   // TODO: Add per-function cleanup when DebugFunction emission is in place.
135:   void endFunctionImpl(const MachineFunction *MF) override {}
136:
137: private:
138:   void emitMCInst(MCInst &Inst);
139:   MCRegister emitOpString(StringRef S, SPIRV::ModuleAnalysisInfo &MAI);
140:   MCRegister emitOpConstantI32(uint32_t Value, MCRegister I32TypeReg,
141:                                SPIRV::ModuleAnalysisInfo &MAI);
142:   MCRegister emitExtInst(SPIRV::NonSemanticExtInst::NonSemanticExtInst Opcode,
143:                          MCRegister VoidTypeReg, MCRegister ExtInstSetReg,
144:                          ArrayRef<MCRegister> Operands,
145:                          SPIRV::ModuleAnalysisInfo &MAI);
146:
147:   /// Find OpTypeVoid in the already-emitted TypeConstVars section, or emit one
148:   /// if the module does not contain it (e.g. no void-returning functions).
149:   MCRegister findOrEmitOpTypeVoid(SPIRV::ModuleAnalysisInfo &MAI);
150:
151:   /// Find OpTypeInt 32 0 in the already-emitted TypeConstVars section, or emit
152:   /// one if the module does not contain it.
153:   MCRegister findOrEmitOpTypeInt32(SPIRV::ModuleAnalysisInfo &MAI);
154:
155:   /// Emit a DebugTypePointer instruction for PT. Skips pointer types that do
156:   /// not carry a DWARF address space. For pointers whose base type is a
157:   /// DIBasicType, looks up the base type's DebugTypeBasic register in
158:   /// BasicTypeRegs. All other pointers (void pointers and pointers whose base
159:   /// type is not a DIBasicType) use DebugInfoNone as the base type operand.
160:   void emitDebugTypePointer(
```
- EN: This range declares interfaces or inline helpers such as beginInstruction, endInstruction, beginFunctionImpl, endFunctionImpl, defining how other backend pieces interact with this header.
- CN: 这一段声明了 beginInstruction、endInstruction、beginFunctionImpl、endFunctionImpl 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 161-173
```cpp
161:       const DIDerivedType *PT, MCRegister VoidTypeReg, MCRegister I32TypeReg,
162:       MCRegister ExtInstSetReg, MCRegister I32ZeroReg,
163:       const DenseMap<const DIBasicType *, MCRegister> &BasicTypeRegs,
164:       SPIRV::ModuleAnalysisInfo &MAI);
165:
166:   /// Map a DWARF source language code to a NonSemantic.Shader.DebugInfo.100
167:   /// source language code.
168:   static unsigned toNSDISrcLang(unsigned DwarfSrcLang);
169: };
170:
171: } // namespace llvm
172:
173: #endif // LLVM_LIB_TARGET_SPIRV_SPIRVNONSEMANTICDEBUGHANDLER_H
```
- EN: This range declares interfaces or inline helpers such as toNSDISrcLang, defining how other backend pieces interact with this header.
- CN: 这一段声明了 toNSDISrcLang 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVSubtarget, SPIRVNonSemanticDebugHandler, CompileUnitInfo, beginModule, emitNonSemanticDebugStrings, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVSubtarget, SPIRVNonSemanticDebugHandler, CompileUnitInfo, beginModule, emitNonSemanticDebugStrings，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `MCTargetDesc/SPIRVBaseInfo.h`
  - `SPIRVModuleAnalysis.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/DenseMap.h`
  - `llvm/ADT/SetVector.h`
  - `llvm/ADT/SmallString.h`
  - `llvm/ADT/SmallVector.h`
  - `llvm/CodeGen/DebugHandlerBase.h`
  - `llvm/IR/DebugInfoMetadata.h`
  - `llvm/MC/MCInst.h`
  - `llvm/MC/MCRegister.h`
