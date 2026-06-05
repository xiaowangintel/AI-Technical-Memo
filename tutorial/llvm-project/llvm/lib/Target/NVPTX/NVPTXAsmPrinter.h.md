# NVPTXAsmPrinter.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXAsmPrinter.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains a printer that converts from our internal representation of machine-dependent LLVM code to NVPTX assembly language.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXAsmPrinter.h - NVPTX LLVM assembly writer ----------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains a printer that converts from our internal representation
10: // of machine-dependent LLVM code to NVPTX assembly language.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #ifndef LLVM_LIB_TARGET_NVPTX_NVPTXASMPRINTER_H
15: #define LLVM_LIB_TARGET_NVPTX_NVPTXASMPRINTER_H
16:
17: #include "NVPTX.h"
18: #include "NVPTXSubtarget.h"
19: #include "NVPTXTargetMachine.h"
20: #include "llvm/ADT/DenseMap.h"
21: #include "llvm/ADT/SmallVector.h"
22: #include "llvm/ADT/StringRef.h"
23: #include "llvm/CodeGen/AsmPrinter.h"
24: #include "llvm/CodeGen/MachineFunction.h"
25: #include "llvm/CodeGen/MachineLoopInfo.h"
26: #include "llvm/IR/Constants.h"
27: #include "llvm/IR/DebugLoc.h"
28: #include "llvm/IR/DerivedTypes.h"
29: #include "llvm/IR/Function.h"
30: #include "llvm/IR/GlobalAlias.h"
31: #include "llvm/IR/GlobalValue.h"
32: #include "llvm/IR/Value.h"
33: #include "llvm/MC/MCExpr.h"
34: #include "llvm/MC/MCStreamer.h"
35: #include "llvm/MC/MCSymbol.h"
36: #include "llvm/Pass.h"
37: #include "llvm/Support/Casting.h"
38: #include "llvm/Support/Compiler.h"
39: #include "llvm/Support/ErrorHandling.h"
40: #include "llvm/Support/raw_ostream.h"
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 41-80
```cpp
41: #include "llvm/Target/TargetMachine.h"
42: #include <algorithm>
43: #include <cassert>
44: #include <map>
45: #include <memory>
46: #include <string>
47: #include <vector>
48:
49: // The ptx syntax and format is very different from that usually seem in a .s
50: // file,
51: // therefore we are not able to use the MCAsmStreamer interface here.
52: //
53: // We are handcrafting the output method here.
54: //
55: // A better approach is to clone the MCAsmStreamer to a MCPTXAsmStreamer
56: // (subclass of MCStreamer).
57:
58: namespace llvm {
59:
60: class MCOperand;
61: class NVPTXTargetStreamer;
62:
63: class LLVM_LIBRARY_VISIBILITY NVPTXAsmPrinter : public AsmPrinter {
64:
65:   class AggBuffer {
66:     // Used to buffer the emitted string for initializing global aggregates.
67:     //
68:     // Normally an aggregate (array, vector, or structure) is emitted as a u8[].
69:     // However, if either element/field of the aggregate is a non-NULL address,
70:     // and all such addresses are properly aligned, then the aggregate is
71:     // emitted as u32[] or u64[]. In the case of unaligned addresses, the
72:     // aggregate is emitted as u8[], and the mask() operator is used for all
73:     // pointers.
74:     //
75:     // We first layout the aggregate in 'buffer' in bytes, except for those
76:     // symbol addresses. For the i-th symbol address in the aggregate, its
77:     // corresponding 4-byte or 8-byte elements in 'buffer' are filled with 0s.
78:     // symbolPosInBuffer[i-1] records its position in 'buffer', and Symbols[i-1]
79:     // records the Value*.
80:     //
```
- EN: This range defines or declares important types such as MCOperand, NVPTXTargetStreamer, LLVM_LIBRARY_VISIBILITY, AggBuffer, shaping the data model used by NVPTXAsmPrinter.h.
- CN: 这一段定义或声明了 MCOperand、NVPTXTargetStreamer、LLVM_LIBRARY_VISIBILITY、AggBuffer 等关键类型，构成 NVPTXAsmPrinter.h 使用的数据模型。

### Lines 81-120
```cpp
 81:     // Once we have this AggBuffer setup, we can choose how to print it out.
 82:   public:
 83:     // number of symbol addresses
 84:     unsigned numSymbols() const { return Symbols.size(); }
 85:
 86:     bool allSymbolsAligned(unsigned ptrSize) const {
 87:       return llvm::all_of(symbolPosInBuffer,
 88:                           [=](unsigned pos) { return pos % ptrSize == 0; });
 89:     }
 90:
 91:   private:
 92:     const unsigned Size;               // size of the buffer in bytes
 93:     std::vector<unsigned char> buffer; // the buffer
 94:     SmallVector<unsigned, 4> symbolPosInBuffer;
 95:     SmallVector<const Value *, 4> Symbols;
 96:     // SymbolsBeforeStripping[i] is the original form of Symbols[i] before
 97:     // stripping pointer casts, i.e.,
 98:     // Symbols[i] == SymbolsBeforeStripping[i]->stripPointerCasts().
 99:     //
100:     // We need to keep these values because AggBuffer::print decides whether to
101:     // emit a "generic()" cast for Symbols[i] depending on the address space of
102:     // SymbolsBeforeStripping[i].
103:     SmallVector<const Value *, 4> SymbolsBeforeStripping;
104:     unsigned curpos;
105:     const NVPTXAsmPrinter &AP;
106:     const bool EmitGeneric;
107:
108:   public:
109:     AggBuffer(unsigned Size, const NVPTXAsmPrinter &AP)
110:         : Size(Size), buffer(Size), curpos(0), AP(AP),
111:           EmitGeneric(AP.EmitGeneric) {}
112:
113:     unsigned getBufferSize() const { return Size; }
114:
115:     // Copy Num bytes from Ptr.
116:     // if Bytes > Num, zero fill up to Bytes.
117:     void addBytes(const unsigned char *Ptr, unsigned Num, unsigned Bytes) {
118:       for (unsigned I : llvm::seq(Num))
119:         addByte(Ptr[I]);
120:       if (Bytes > Num)
```
- EN: This range declares interfaces or inline helpers such as numSymbols, allSymbolsAligned, AggBuffer, EmitGeneric, defining how other backend pieces interact with this header.
- CN: 这一段声明了 numSymbols、allSymbolsAligned、AggBuffer、EmitGeneric 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 121-160
```cpp
121:         addZeros(Bytes - Num);
122:     }
123:
124:     void addByte(uint8_t Byte) {
125:       assert(curpos < Size);
126:       buffer[curpos] = Byte;
127:       curpos++;
128:     }
129:
130:     void addZeros(unsigned Num) {
131:       for ([[maybe_unused]] unsigned _ : llvm::seq(Num)) {
132:         addByte(0);
133:       }
134:     }
135:
136:     void addSymbol(const Value *GVar, const Value *GVarBeforeStripping) {
137:       symbolPosInBuffer.push_back(curpos);
138:       Symbols.push_back(GVar);
139:       SymbolsBeforeStripping.push_back(GVarBeforeStripping);
140:     }
141:
142:     void printBytes(raw_ostream &os);
143:     void printWords(raw_ostream &os);
144:
145:   private:
146:     void printSymbol(unsigned nSym, raw_ostream &os);
147:   };
148:
149:   friend class AggBuffer;
150:
151: public:
152:   static char ID;
153:
154:   StringRef getPassName() const override { return "NVPTX Assembly Printer"; }
155:
156: private:
157:   const Function *F;
158:
159:   NVPTXTargetStreamer *getTargetStreamer() const;
160:
```
- EN: This range defines or declares important types such as addZeros, addByte, assert, addSymbol, shaping the data model used by NVPTXAsmPrinter.h.
- CN: 这一段定义或声明了 addZeros、addByte、assert、addSymbol 等关键类型，构成 NVPTXAsmPrinter.h 使用的数据模型。

### Lines 161-200
```cpp
161:   void emitStartOfAsmFile(Module &M) override;
162:   void emitBasicBlockStart(const MachineBasicBlock &MBB) override;
163:   void emitFunctionEntryLabel() override;
164:   void emitFunctionBodyStart() override;
165:   void emitFunctionBodyEnd() override;
166:   void emitImplicitDef(const MachineInstr *MI) const override;
167:
168:   void emitInstruction(const MachineInstr *) override;
169:   void lowerToMCInst(const MachineInstr *MI, MCInst &OutMI);
170:   MCOperand lowerOperand(const MachineOperand &MO);
171:   MCOperand GetSymbolRef(const MCSymbol *Symbol);
172:   unsigned encodeVirtualRegister(unsigned Reg);
173:
174:   void printMemOperand(const MachineInstr *MI, unsigned OpNum, raw_ostream &O,
175:                        const char *Modifier = nullptr);
176:   void printModuleLevelGV(const GlobalVariable *GVar, raw_ostream &O,
177:                           bool processDemoted, const NVPTXSubtarget &STI);
178:   void emitGlobals(const Module &M);
179:   void emitGlobalAlias(const Module &M, const GlobalAlias &GA) override;
180:   void emitHeader(Module &M, const NVPTXSubtarget &STI);
181:   void emitKernelFunctionDirectives(const Function &F, raw_ostream &O) const;
182:   void emitVirtualRegister(unsigned int vr, raw_ostream &);
183:   void emitFunctionParamList(const Function *, raw_ostream &O);
184:   void setAndEmitFunctionVirtualRegisters(const MachineFunction &MF);
185:   void encodeDebugInfoRegisterNumbers(const MachineFunction &MF);
186:   void printReturnValStr(const Function *, raw_ostream &O);
187:   void printReturnValStr(const MachineFunction &MF, raw_ostream &O);
188:   bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
189:                        const char *ExtraCode, raw_ostream &) override;
190:   void printOperand(const MachineInstr *MI, unsigned OpNum, raw_ostream &O);
191:   bool PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNo,
192:                              const char *ExtraCode, raw_ostream &) override;
193:
194:   const MCExpr *lowerConstantForGV(const Constant *CV,
195:                                    bool ProcessingGeneric) const;
196:   void printMCExpr(const MCExpr &Expr, raw_ostream &OS) const;
197:   /// Emit a blob of inline asm to the output streamer.
198:   void emitInlineAsm(StringRef Str, const MCSubtargetInfo &STI,
199:                      const MCTargetOptions &MCOptions, const MDNode *LocMDNode,
200:                      InlineAsm::AsmDialect Dialect,
```
- EN: This range declares interfaces or inline helpers such as emitStartOfAsmFile, emitBasicBlockStart, emitFunctionEntryLabel, emitFunctionBodyStart, defining how other backend pieces interact with this header.
- CN: 这一段声明了 emitStartOfAsmFile、emitBasicBlockStart、emitFunctionEntryLabel、emitFunctionBodyStart 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 201-240
```cpp
201:                      const MachineInstr *MI) override;
202:
203: protected:
204:   bool doInitialization(Module &M) override;
205:   bool doFinalization(Module &M) override;
206:
207:   /// Create NVPTX-specific DwarfDebug handler.
208:   DwarfDebug *createDwarfDebug() override;
209:
210: private:
211:   bool GlobalsEmitted;
212:
213:   // This is specific per MachineFunction.
214:   const MachineRegisterInfo *MRI;
215:   // The contents are specific for each
216:   // MachineFunction. But the size of the
217:   // array is not.
218:   typedef DenseMap<unsigned, unsigned> VRegMap;
219:   typedef DenseMap<const TargetRegisterClass *, VRegMap> VRegRCMap;
220:   VRegRCMap VRegMapping;
221:
222:   // List of variables demoted to a function scope.
223:   std::map<const Function *, std::vector<const GlobalVariable *>> localDecls;
224:
225:   void emitPTXGlobalVariable(const GlobalVariable *GVar, raw_ostream &O,
226:                              const NVPTXSubtarget &STI);
227:   void emitPTXAddressSpace(unsigned int AddressSpace, raw_ostream &O) const;
228:   std::string getPTXFundamentalTypeStr(Type *Ty, bool = true) const;
229:   void printScalarConstant(const Constant *CPV, raw_ostream &O);
230:   void printFPConstant(const ConstantFP *Fp, raw_ostream &O) const;
231:   void bufferLEByte(const Constant *CPV, int Bytes, AggBuffer *aggBuffer);
232:   void bufferAggregateConstant(const Constant *CV, AggBuffer *aggBuffer);
233:   void bufferAggregateConstVec(const ConstantVector *CV, AggBuffer *aggBuffer);
234:
235:   void emitLinkageDirective(const GlobalValue *V, raw_ostream &O);
236:   void emitDeclarations(const Module &, raw_ostream &O);
237:   void emitDeclaration(const Function *, raw_ostream &O);
238:   void emitAliasDeclaration(const GlobalAlias *, raw_ostream &O);
239:   void emitDeclarationWithName(const Function *, MCSymbol *, raw_ostream &O);
240:   void emitDemotedVars(const Function *, raw_ostream &);
```
- EN: This range declares interfaces or inline helpers such as doInitialization, doFinalization, createDwarfDebug, emitPTXAddressSpace, defining how other backend pieces interact with this header.
- CN: 这一段声明了 doInitialization、doFinalization、createDwarfDebug、emitPTXAddressSpace 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 241-280
```cpp
241:
242:   bool isLoopHeaderOfNoUnroll(const MachineBasicBlock &MBB) const;
243:
244:   // Used to control the need to emit .generic() in the initializer of
245:   // module scope variables.
246:   // Although ptx supports the hybrid mode like the following,
247:   //    .global .u32 a;
248:   //    .global .u32 b;
249:   //    .global .u32 addr[] = {a, generic(b)}
250:   // we have difficulty representing the difference in the NVVM IR.
251:   //
252:   // Since the address value should always be generic in CUDA C and always
253:   // be specific in OpenCL, we use this simple control here.
254:   //
255:   const bool EmitGeneric;
256:
257: public:
258:   NVPTXAsmPrinter(TargetMachine &TM, std::unique_ptr<MCStreamer> Streamer)
259:       : AsmPrinter(TM, std::move(Streamer), ID),
260:         EmitGeneric(static_cast<NVPTXTargetMachine &>(TM).getDrvInterface() ==
261:                     NVPTX::CUDA) {}
262:
263:   bool runOnMachineFunction(MachineFunction &F) override;
264:
265:   void getAnalysisUsage(AnalysisUsage &AU) const override {
266:     AU.addRequired<MachineLoopInfoWrapperPass>();
267:     AsmPrinter::getAnalysisUsage(AU);
268:   }
269:
270:   std::string getVirtualRegisterName(unsigned) const;
271:
272:   const MCSymbol *getFunctionFrameSymbol() const override;
273:
274:   // Make emitGlobalVariable() no-op for NVPTX.
275:   // Global variables have been already emitted by the time the base AsmPrinter
276:   // attempts to do so in doFinalization() (see NVPTXAsmPrinter::emitGlobals()).
277:   void emitGlobalVariable(const GlobalVariable *GV) override {}
278: };
279:
280: } // end namespace llvm
```
- EN: This range declares interfaces or inline helpers such as isLoopHeaderOfNoUnroll, NVPTXAsmPrinter, runOnMachineFunction, getAnalysisUsage, defining how other backend pieces interact with this header.
- CN: 这一段声明了 isLoopHeaderOfNoUnroll、NVPTXAsmPrinter、runOnMachineFunction、getAnalysisUsage 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 281-282
```cpp
281:
282: #endif // LLVM_LIB_TARGET_NVPTX_NVPTXASMPRINTER_H
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

## Key Concepts / 关键概念

- EN: The asm printer layer serializes machine-level state into target assembly or object-oriented output.
  - CN: 汇编打印层负责把机器级状态序列化为目标汇编或面向目标的输出。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include MCOperand, NVPTXTargetStreamer, LLVM_LIBRARY_VISIBILITY, AggBuffer, numSymbols, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 MCOperand, NVPTXTargetStreamer, LLVM_LIBRARY_VISIBILITY, AggBuffer, numSymbols，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
  - `NVPTXSubtarget.h`
  - `NVPTXTargetMachine.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/DenseMap.h`
  - `llvm/ADT/SmallVector.h`
  - `llvm/ADT/StringRef.h`
  - `llvm/CodeGen/AsmPrinter.h`
  - `llvm/CodeGen/MachineFunction.h`
  - `llvm/CodeGen/MachineLoopInfo.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/DebugLoc.h`
  - `llvm/IR/DerivedTypes.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/GlobalAlias.h`
  - `llvm/IR/GlobalValue.h`
  - `llvm/IR/Value.h`
  - `llvm/MC/MCExpr.h`
  - `llvm/MC/MCStreamer.h`
  - `llvm/MC/MCSymbol.h`
- System/standard headers / 系统或标准头文件:
  - `algorithm`
  - `cassert`
  - `map`
  - `memory`
  - `string`
  - `vector`
