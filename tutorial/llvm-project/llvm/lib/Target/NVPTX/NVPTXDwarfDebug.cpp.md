# NVPTXDwarfDebug.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXDwarfDebug.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements helper functions for NVPTX-specific debug information processing.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXDwarfDebug.cpp - NVPTX DwarfDebug Implementation ------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file implements helper functions for NVPTX-specific debug information
10: // processing.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "NVPTXDwarfDebug.h"
15: #include "NVPTXSubtarget.h"
16: #include "llvm/BinaryFormat/Dwarf.h"
17: #include "llvm/CodeGen/MachineFunction.h"
18: #include "llvm/CodeGen/MachineInstr.h"
19: #include "llvm/IR/DebugInfoMetadata.h"
20: #include "llvm/IR/Function.h"
21: #include "llvm/IR/GlobalVariable.h"
22: #include "llvm/MC/MCAsmInfo.h"
23: #include "llvm/MC/MCContext.h"
24: #include "llvm/MC/MCStreamer.h"
25: #include "llvm/Support/CommandLine.h"
26: #include "llvm/Support/NVPTXAddrSpace.h"
27: #include "llvm/Target/TargetMachine.h"
28:
29: using namespace llvm;
30:
31: // Command line option to control inlined_at enhancement to lineinfo support.
32: // Valid only when debuginfo emissionkind is DebugDirectivesOnly or
33: // LineTablesOnly.
34: static cl::opt<bool> LineInfoWithInlinedAt(
35:     "line-info-inlined-at",
36:     cl::desc("Emit line with inlined_at enhancement for NVPTX"), cl::init(true),
37:     cl::Hidden);
38:
39: NVPTXDwarfDebug::NVPTXDwarfDebug(AsmPrinter *A) : DwarfDebug(A) {
40:   // PTX emits debug strings inline (no .debug_str section), does not support
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 41-80
```cpp
41:   // .debug_ranges, and uses sections as references (no temp symbols inside
42:   // DWARF sections).  DWARF v2 is the default for NVPTX and does not support
43:   // accelerator tables.
44:   setUseInlineStrings(true);
45:   setUseRangesSection(false);
46:   setUseSectionsAsReferences(true);
47:   Asm->OutStreamer->getContext().setDwarfVersion(2);
48:   setTheAccelTableKind(AccelTableKind::None);
49: }
50:
51: MCSymbol *NVPTXDwarfDebug::getOrCreateFuncNameSymbol(StringRef LinkageName) {
52:   return InfoHolder.getStringPool().getEntry(*Asm, LinkageName).getSymbol();
53: }
54:
55: bool NVPTXDwarfDebug::isEnhancedLineinfo(const MachineFunction &MF) const {
56:   const DISubprogram *SP = MF.getFunction().getSubprogram();
57:   const NVPTXSubtarget &STI = MF.getSubtarget<NVPTXSubtarget>();
58:   return LineInfoWithInlinedAt && (STI.getPTXVersion() >= 72) && SP &&
59:          (SP->getUnit()->isDebugDirectivesOnly() ||
60:           SP->getUnit()->getEmissionKind() == DICompileUnit::LineTablesOnly);
61: }
62:
63: /// NVPTX-specific source line recording with inlined_at support.
64: ///
65: /// Why this exists:
66: /// NVPTX supports an "enhanced lineinfo" mode where inlining context is carried
67: /// via line-table directives, rather than full DWARF DIEs. This is conceptually
68: /// similar to proposals[1] for richer DWARF line tables that carry inline call
69: /// context and callee identity in the line table. NVPTX implements this via
70: /// target-specific `.loc` extensions in the PTX ISA[3].
71: ///
72: /// How it impacts PTX assembly generation:
73: /// - When enabled (PTX ISA >= 7.2 + line-tables-only / debug-directives-only),
74: ///   we emit multiple consecutive `.loc` directives for a single inlined
75: ///   instruction: the instruction's own location and its `inlined_at` parent
76: ///   chain.
77: /// - During emission we use `MCStreamer::emitDwarfLocDirectiveWithInlinedAt` to
78: ///   emit an enhanced `.loc` directive[3] that carries the extra
79: ///   `function_name` and `inlined_at` operands in the PTX assembly stream.
80: ///
```
- EN: This range implements operational logic in helpers such as setUseInlineStrings, setUseRangesSection, setUseSectionsAsReferences, getContext, translating backend policy into executable code.
- CN: 这一段实现了 setUseInlineStrings、setUseRangesSection、setUseSectionsAsReferences、getContext 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81: /// Example (conceptual PTX `.loc` sequence for an inlined callsite):
 82: ///   .loc 1 16 3                            // caller location
 83: ///   .loc 1  5 3, function_name $L__info_stringN, inlined_at 1 16 3
 84: ///                                         // inlined callee location
 85: ///   Here, $L__info_stringN is a label (or label+immediate) referring into
 86: ///   `.debug_str`.
 87: ///
 88: /// How this impacts DWARF :
 89: /// DWARF generation tools that consume this PTX(e.g. ptxas assembler) can use
 90: /// the `inlined_at` and `function_name` operands to extend the DWARF v2
 91: /// line table information.
 92: /// This adds:
 93: /// - a `context` column[2]: the `inlined_at <file> <line> <col>` information
 94: ///   populates an inlining "context" (a reference to the parent/callsite row)
 95: ///   enabling reconstruction of inline call chains from the line table.
 96: /// - a `function_name` column[2]: the `.loc ... function_name <sym>` identifies
 97: ///   the inlined callee associated with a non-zero context.
 98: ///
 99: /// References:
100: /// - [1] DWARF line tables / Two-Level Line Tables:
101: ///   https://wiki.dwarfstd.org/TwoLevelLineTables.md
102: /// - [2] DWARF issue tracking for Two-Level Line Tables:
103: ///   https://dwarfstd.org/issues/140906.1.html
104: /// - [3] NVIDIA PTX ISA `.loc` (debugging directives; PTX ISA 7.2+):
105: ///   https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#debugging-directives-loc
106: void NVPTXDwarfDebug::recordTargetSourceLine(const DebugLoc &DL,
107:                                              unsigned Flags) {
108:   // Maintain a work list of .loc to be emitted. If we are emitting the
109:   // inlined_at directive, we might need to emit additional .loc prior
110:   // to it for the location contained in the inlined_at.
111:   SmallVector<const DILocation *, 8> WorkList;
112:   SmallDenseSet<const DILocation *, 8> WorkListSet;
113:   const DILocation *EmitLoc = DL.get();
114:
115:   if (!EmitLoc)
116:     return;
117:
118:   const MachineFunction *MF = Asm->MF;
119:   if (!MF)
120:     return;
```
- EN: This range implements operational logic in helpers such as get, translating backend policy into executable code.
- CN: 这一段实现了 get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:
122:   const bool EnhancedLineinfo = isEnhancedLineinfo(*MF);
123:
124:   while (EmitLoc) {
125:     // Get the scope for the current location.
126:     const DIScope *Scope = EmitLoc->getScope();
127:     if (!Scope)
128:       break; // scope is null, we are done.
129:
130:     // Check if this loc is already in work list, if so, we are done.
131:     if (WorkListSet.contains(EmitLoc))
132:       break;
133:
134:     // Add this location to the work list.
135:     WorkList.push_back(EmitLoc);
136:     WorkListSet.insert(EmitLoc);
137:
138:     if (!EnhancedLineinfo) // No enhanced lineinfo, we are done.
139:       break;
140:
141:     const DILocation *IA = EmitLoc->getInlinedAt();
142:     // Check if this has inlined_at information, and if the parent location
143:     // has not yet been emitted. If already emitted, we don't need to
144:     // re-emit the parent chain.
145:     if (IA && !EmittedInlinedAtLocs.contains(IA))
146:       EmitLoc = IA;
147:     else // We are done.
148:       break;
149:   }
150:
151:   const unsigned CUID = Asm->OutStreamer->getContext().getDwarfCompileUnitID();
152:   // Traverse the work list, and emit .loc.
153:   while (!WorkList.empty()) {
154:     const DILocation *Current = WorkList.pop_back_val();
155:     const DIScope *Scope = Current->getScope();
156:
157:     if (!Scope)
158:       llvm_unreachable("we shouldn't be here for null scope");
159:
160:     const DILocation *InlinedAt = Current->getInlinedAt();
```
- EN: This range implements operational logic in helpers such as isEnhancedLineinfo, getScope, push_back, insert, translating backend policy into executable code.
- CN: 这一段实现了 isEnhancedLineinfo、getScope、push_back、insert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:     StringRef Fn = Scope->getFilename();
162:     const unsigned Line = Current->getLine();
163:     const unsigned Col = Current->getColumn();
164:     unsigned Discriminator = 0;
165:     if (Line != 0 && getDwarfVersion() >= 4)
166:       if (const DILexicalBlockFile *LBF = dyn_cast<DILexicalBlockFile>(Scope))
167:         Discriminator = LBF->getDiscriminator();
168:
169:     const unsigned FileNo = static_cast<DwarfCompileUnit &>(*getUnits()[CUID])
170:                                 .getOrCreateSourceID(Scope->getFile());
171:
172:     if (EnhancedLineinfo && InlinedAt) {
173:       const unsigned FileIA = static_cast<DwarfCompileUnit &>(*getUnits()[CUID])
174:                                   .getOrCreateSourceID(InlinedAt->getFile());
175:       const DISubprogram *SubProgram = getDISubprogram(Current->getScope());
176:       DwarfStringPoolEntryRef Entry = InfoHolder.getStringPool().getEntry(
177:           *Asm, SubProgram->getLinkageName());
178:       Asm->OutStreamer->emitDwarfLocDirectiveWithInlinedAt(
179:           FileNo, Line, Col, FileIA, InlinedAt->getLine(),
180:           InlinedAt->getColumn(), Entry.getSymbol(), Flags, 0, Discriminator,
181:           Fn);
182:     } else {
183:       Asm->OutStreamer->emitDwarfLocDirective(FileNo, Line, Col, Flags, 0,
184:                                               Discriminator, Fn);
185:     }
186:     // Mark this location as emitted so we don't re-emit the parent chain
187:     // for subsequent instructions that share the same inlined_at parent.
188:     if (EnhancedLineinfo)
189:       EmittedInlinedAtLocs.insert(Current);
190:   }
191: }
192:
193: /// NVPTX-specific debug info initialization.
194: void NVPTXDwarfDebug::initializeTargetDebugInfo(const MachineFunction &MF) {
195:   EmittedInlinedAtLocs.clear();
196: }
197:
198: // PTX does not support subtracting labels from the code section in the
199: // debug_loc section.  To work around this, the NVPTX backend needs the
200: // compile unit to have no low_pc in order to have a zero base_address
```
- EN: This range implements operational logic in helpers such as getFilename, getLine, getColumn, getDiscriminator, translating backend policy into executable code.
- CN: 这一段实现了 getFilename、getLine、getColumn、getDiscriminator 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201: // when handling debug_loc in cuda-gdb.
202: bool NVPTXDwarfDebug::shouldAttachCompileUnitRanges() const {
203:   return !tuneForGDB();
204: }
205:
206: // Same label-subtraction limitation as above: cuda-gdb doesn't handle
207: // setting a per-variable base to zero, so we emit labels with no base
208: // while having no compile unit low_pc.
209: bool NVPTXDwarfDebug::shouldResetBaseAddress(const MCSection &Section) const {
210:   return tuneForGDB();
211: }
212:
213: static unsigned translateToNVVMDWARFAddrSpace(unsigned AddrSpace) {
214:   switch (AddrSpace) {
215:   case NVPTXAS::ADDRESS_SPACE_GENERIC:
216:     return NVPTXAS::DWARF_ADDR_generic_space;
217:   case NVPTXAS::ADDRESS_SPACE_GLOBAL:
218:     return NVPTXAS::DWARF_ADDR_global_space;
219:   case NVPTXAS::ADDRESS_SPACE_SHARED:
220:     return NVPTXAS::DWARF_ADDR_shared_space;
221:   case NVPTXAS::ADDRESS_SPACE_CONST:
222:     return NVPTXAS::DWARF_ADDR_const_space;
223:   case NVPTXAS::ADDRESS_SPACE_LOCAL:
224:     return NVPTXAS::DWARF_ADDR_local_space;
225:   default:
226:     llvm_unreachable(
227:         "Cannot translate unknown address space to DWARF address space");
228:     return AddrSpace;
229:   }
230: }
231:
232: // cuda-gdb requires DW_AT_address_class on variable DIEs.  The address space
233: // is encoded in the DIExpression as a DW_OP_constu <DWARF Address Space>
234: // DW_OP_swap DW_OP_xderef sequence.  We strip that sequence from the
235: // expression and return the address space so the caller can emit
236: // DW_AT_address_class separately.
237: const DIExpression *NVPTXDwarfDebug::adjustExpressionForTarget(
238:     const DIExpression *Expr, std::optional<unsigned> &TargetAddrSpace) const {
239:   if (!tuneForGDB())
240:     return Expr;
```
- EN: This range implements operational logic in helpers such as NVPTXDwarfDebug::shouldAttachCompileUnitRanges, tuneForGDB, NVPTXDwarfDebug::shouldResetBaseAddress, translateToNVVMDWARFAddrSpace, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXDwarfDebug::shouldAttachCompileUnitRanges、tuneForGDB、NVPTXDwarfDebug::shouldResetBaseAddress、translateToNVVMDWARFAddrSpace 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-280
```cpp
241:   unsigned LocalAddrSpace;
242:   const DIExpression *NewExpr =
243:       DIExpression::extractAddressClass(Expr, LocalAddrSpace);
244:   if (NewExpr != Expr) {
245:     TargetAddrSpace = LocalAddrSpace;
246:     return NewExpr;
247:   }
248:   return Expr;
249: }
250:
251: // Emit DW_AT_address_class for cuda-gdb.  See NVPTXAS::DWARF_AddressSpace.
252: //
253: // The address class depends on the variable's storage kind:
254: //   Global:     from the expression (if encoded) or the IR address space
255: //   Register:   DWARF_ADDR_reg_space (no expression means register location)
256: //   FrameIndex: from the expression (if encoded) or DWARF_ADDR_local_space
257: void NVPTXDwarfDebug::addTargetVariableAttributes(
258:     DwarfCompileUnit &CU, DIE &Die, std::optional<unsigned> TargetAddrSpace,
259:     VariableLocationKind VarLocKind, const GlobalVariable *GV) const {
260:   if (!tuneForGDB())
261:     return;
262:
263:   unsigned DefaultAddrSpace = NVPTXAS::DWARF_ADDR_global_space;
264:   switch (VarLocKind) {
265:   case VariableLocationKind::Global:
266:     if (!TargetAddrSpace && GV)
267:       TargetAddrSpace =
268:           translateToNVVMDWARFAddrSpace(GV->getType()->getAddressSpace());
269:     DefaultAddrSpace = NVPTXAS::DWARF_ADDR_global_space;
270:     break;
271:   case VariableLocationKind::Register:
272:     DefaultAddrSpace = NVPTXAS::DWARF_ADDR_reg_space;
273:     break;
274:   case VariableLocationKind::FrameIndex:
275:     DefaultAddrSpace = NVPTXAS::DWARF_ADDR_local_space;
276:     break;
277:   }
278:
279:   CU.addUInt(Die, dwarf::DW_AT_address_class, dwarf::DW_FORM_data1,
280:              TargetAddrSpace.value_or(DefaultAddrSpace));
```
- EN: This range implements operational logic in helpers such as DIExpression::extractAddressClass, translateToNVVMDWARFAddrSpace, value_or, translating backend policy into executable code.
- CN: 这一段实现了 DIExpression::extractAddressClass、translateToNVVMDWARFAddrSpace、value_or 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 281-281
```cpp
281: }
```
- EN: This range opens, closes, or reshapes namespaces so the backend implementation lives in the expected LLVM scope.
- CN: 这一段打开、关闭或调整命名空间，使后端实现位于 LLVM 约定的作用域中。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXDwarfDebug::NVPTXDwarfDebug, setUseInlineStrings, setUseRangesSection, setUseSectionsAsReferences, getContext, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXDwarfDebug::NVPTXDwarfDebug, setUseInlineStrings, setUseRangesSection, setUseSectionsAsReferences, getContext，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXDwarfDebug.h`
  - `NVPTXSubtarget.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/BinaryFormat/Dwarf.h`
  - `llvm/CodeGen/MachineFunction.h`
  - `llvm/CodeGen/MachineInstr.h`
  - `llvm/IR/DebugInfoMetadata.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/GlobalVariable.h`
  - `llvm/MC/MCAsmInfo.h`
  - `llvm/MC/MCContext.h`
  - `llvm/MC/MCStreamer.h`
  - `llvm/Support/CommandLine.h`
  - `llvm/Support/NVPTXAddrSpace.h`
  - `llvm/Target/TargetMachine.h`
