# SPIRVModuleAnalysis.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVModuleAnalysis.h`
- Repository: `llvm-project`
- Purpose (EN): The analysis collects instructions that should be output at the module level and performs the global register numbering.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- SPIRVModuleAnalysis.h - analysis of global instrs & regs -*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // The analysis collects instructions that should be output at the module level
10: // and performs the global register numbering.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #ifndef LLVM_LIB_TARGET_SPIRV_SPIRVMODULEANALYSIS_H
15: #define LLVM_LIB_TARGET_SPIRV_SPIRVMODULEANALYSIS_H
16:
17: #include "MCTargetDesc/SPIRVBaseInfo.h"
18: #include "SPIRVGlobalRegistry.h"
19: #include "SPIRVUtils.h"
20: #include "llvm/ADT/DenseMap.h"
21: #include "llvm/ADT/SmallSet.h"
22: #include "llvm/ADT/SmallVector.h"
23:
24: namespace llvm {
25: class SPIRVSubtarget;
26: class MachineFunction;
27: class MachineModuleInfo;
28:
29: namespace SPIRV {
30: // The enum contains logical module sections for the instruction collection.
31: enum ModuleSectionType {
32:   //  MB_Capabilities, MB_Extensions, MB_ExtInstImports, MB_MemoryModel,
33:   MB_EntryPoints, // All OpEntryPoint instructions (if any).
34:   //  MB_ExecutionModes, MB_DebugSourceAndStrings,
35:   MB_DebugNames,           // All OpName and OpMemberName intrs.
36:   MB_DebugStrings,         // All OpString intrs.
37:   MB_DebugModuleProcessed, // All OpModuleProcessed instructions.
38:   MB_AliasingInsts,        // SPV_INTEL_memory_access_aliasing instructions.
39:   MB_Annotations,          // OpDecorate, OpMemberDecorate etc.
40:   MB_TypeConstVars,        // OpTypeXXX, OpConstantXXX, and global OpVariables.
```
- EN: This range defines or declares important types such as SPIRVSubtarget, MachineFunction, MachineModuleInfo, ModuleSectionType, shaping the data model used by SPIRVModuleAnalysis.h.
- CN: 这一段定义或声明了 SPIRVSubtarget、MachineFunction、MachineModuleInfo、ModuleSectionType 等关键类型，构成 SPIRVModuleAnalysis.h 使用的数据模型。

### Lines 41-80
```cpp
41:   MB_NonSemanticGlobalDI,  // OpExtInst with e.g. DebugSource, DebugTypeBasic.
42:   MB_ExtFuncDecls,         // OpFunction etc. to declare for external funcs.
43:   NUM_MODULE_SECTIONS      // Total number of sections requiring basic blocks.
44: };
45:
46: struct Requirements {
47:   const bool IsSatisfiable;
48:   const std::optional<Capability::Capability> Cap;
49:   const ExtensionList Exts;
50:   const VersionTuple MinVer; // 0 if no min version is required.
51:   const VersionTuple MaxVer; // 0 if no max version is required.
52:
53:   Requirements(bool IsSatisfiable = false,
54:                std::optional<Capability::Capability> Cap = {},
55:                ExtensionList Exts = {}, VersionTuple MinVer = VersionTuple(),
56:                VersionTuple MaxVer = VersionTuple())
57:       : IsSatisfiable(IsSatisfiable), Cap(Cap), Exts(std::move(Exts)),
58:         MinVer(MinVer), MaxVer(MaxVer) {}
59:   Requirements(Capability::Capability Cap) : Requirements(true, {Cap}) {}
60: };
61:
62: struct RequirementHandler {
63: private:
64:   CapabilityList MinimalCaps;
65:
66:   // AllCaps and AvailableCaps are related but different. AllCaps is a subset of
67:   // AvailableCaps. AvailableCaps is the complete set of capabilities that are
68:   // available to the current target. AllCaps is the set of capabilities that
69:   // are required by the current module.
70:   SmallSet<Capability::Capability, 8> AllCaps;
71:   DenseSet<unsigned> AvailableCaps;
72:
73:   SmallSet<Extension::Extension, 4> AllExtensions;
74:   VersionTuple MinVersion; // 0 if no min version is defined.
75:   VersionTuple MaxVersion; // 0 if no max version is defined.
76:   // Add capabilities to AllCaps, recursing through their implicitly declared
77:   // capabilities too.
78:   void recursiveAddCapabilities(const CapabilityList &ToPrune);
79:
80:   void initAvailableCapabilitiesForOpenCL(const SPIRVSubtarget &ST);
```
- EN: This range defines or declares important types such as Requirements, VersionTuple, MinVer, RequirementHandler, shaping the data model used by SPIRVModuleAnalysis.h.
- CN: 这一段定义或声明了 Requirements、VersionTuple、MinVer、RequirementHandler 等关键类型，构成 SPIRVModuleAnalysis.h 使用的数据模型。

### Lines 81-120
```cpp
 81:   void initAvailableCapabilitiesForVulkan(const SPIRVSubtarget &ST);
 82:
 83: public:
 84:   RequirementHandler() = default;
 85:   void clear() {
 86:     MinimalCaps.clear();
 87:     AllCaps.clear();
 88:     AvailableCaps.clear();
 89:     AllExtensions.clear();
 90:     MinVersion = VersionTuple();
 91:     MaxVersion = VersionTuple();
 92:   }
 93:   const CapabilityList &getMinimalCapabilities() const { return MinimalCaps; }
 94:   const SmallSet<Extension::Extension, 4> &getExtensions() const {
 95:     return AllExtensions;
 96:   }
 97:   // Add a list of capabilities, ensuring AllCaps captures all the implicitly
 98:   // declared capabilities, and MinimalCaps has the minimal set of required
 99:   // capabilities (so all implicitly declared ones are removed).
100:   void addCapabilities(const CapabilityList &ToAdd);
101:   void addCapability(Capability::Capability ToAdd) { addCapabilities({ToAdd}); }
102:   void addExtensions(const ExtensionList &ToAdd) {
103:     AllExtensions.insert_range(ToAdd);
104:   }
105:   void addExtension(Extension::Extension ToAdd) { AllExtensions.insert(ToAdd); }
106:   // Add the given requirements to the lists. If constraints conflict, or these
107:   // requirements cannot be satisfied, then abort the compilation.
108:   void addRequirements(const Requirements &Req);
109:   // Get requirement and add it to the list.
110:   void getAndAddRequirements(SPIRV::OperandCategory::OperandCategory Category,
111:                              uint32_t i, const SPIRVSubtarget &ST);
112:   // Check if all the requirements can be satisfied for the given subtarget, and
113:   // if not abort compilation.
114:   void checkSatisfiable(const SPIRVSubtarget &ST) const;
115:   void initAvailableCapabilities(const SPIRVSubtarget &ST);
116:   // Add the given capabilities to available and all their implicitly defined
117:   // capabilities too.
118:   void addAvailableCaps(const CapabilityList &ToAdd);
119:   bool isCapabilityAvailable(Capability::Capability Cap) const {
120:     return AvailableCaps.contains(Cap);
```
- EN: This range declares interfaces or inline helpers such as initAvailableCapabilitiesForVulkan, clear, VersionTuple, getMinimalCapabilities, defining how other backend pieces interact with this header.
- CN: 这一段声明了 initAvailableCapabilitiesForVulkan、clear、VersionTuple、getMinimalCapabilities 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 121-160
```cpp
121:   }
122:
123:   // Remove capability ToRemove, but only if IfPresent is present.
124:   void removeCapabilityIf(const Capability::Capability ToRemove,
125:                           const Capability::Capability IfPresent);
126: };
127:
128: using InstrList = SmallVector<const MachineInstr *>;
129: // Maps a local register to the corresponding global alias.
130: using LocalToGlobalRegTable = std::map<Register, MCRegister>;
131: using RegisterAliasMapTy =
132:     std::map<const MachineFunction *, LocalToGlobalRegTable>;
133:
134: // The struct contains results of the module analysis and methods
135: // to access them.
136: struct ModuleAnalysisInfo {
137:   RequirementHandler Reqs;
138:   MemoryModel::MemoryModel Mem;
139:   AddressingModel::AddressingModel Addr;
140:   SourceLanguage::SourceLanguage SrcLang;
141:   unsigned SrcLangVersion;
142:   StringSet<> SrcExt;
143:   // Maps ExtInstSet to corresponding ID register.
144:   DenseMap<unsigned, MCRegister> ExtInstSetMap;
145:   // Contains the list of all global OpVariables in the module.
146:   SmallVector<const MachineInstr *, 4> GlobalVarList;
147:   // Maps functions and global variables to corresponding ID registers.
148:   DenseMap<const GlobalObject *, MCRegister> GlobalObjMap;
149:   // The set contains machine instructions which are necessary
150:   // for correct MIR but will not be emitted in function bodies.
151:   DenseSet<const MachineInstr *> InstrsToDelete;
152:   // The table contains global aliases of local registers for each machine
153:   // function. The aliases are used to substitute local registers during
154:   // code emission.
155:   RegisterAliasMapTy RegisterAliasTable;
156:   // The counter holds the maximum ID we have in the module.
157:   unsigned MaxID;
158:   // The array contains lists of MIs for each module section.
159:   InstrList MS[NUM_MODULE_SECTIONS];
160:   // The table maps MBB number to SPIR-V unique ID register.
```
- EN: This range defines or declares important types such as ModuleAnalysisInfo, shaping the data model used by SPIRVModuleAnalysis.h.
- CN: 这一段定义或声明了 ModuleAnalysisInfo 等关键类型，构成 SPIRVModuleAnalysis.h 使用的数据模型。

### Lines 161-200
```cpp
161:   DenseMap<std::pair<const MachineFunction *, int>, MCRegister> BBNumToRegMap;
162:   // The table maps function pointers to their default FP fast math info. It can
163:   // be assumed that the SmallVector is sorted by the bit width of the type. The
164:   // first element is the smallest bit width, and the last element is the
165:   // largest bit width, therefore, we will have {half, float, double} in
166:   // the order of their bit widths.
167:   DenseMap<const Function *, SPIRV::FPFastMathDefaultInfoVector>
168:       FPFastMathDefaultInfoMap;
169:
170:   MCRegister getGlobalObjReg(const GlobalObject *GO) {
171:     assert(GO && "GlobalObject is null");
172:     return GlobalObjMap.lookup(GO);
173:   }
174:   MCRegister getExtInstSetReg(unsigned SetNum) { return ExtInstSetMap[SetNum]; }
175:   InstrList &getMSInstrs(unsigned MSType) { return MS[MSType]; }
176:   void setSkipEmission(const MachineInstr *MI) { InstrsToDelete.insert(MI); }
177:   bool getSkipEmission(const MachineInstr *MI) {
178:     return InstrsToDelete.contains(MI);
179:   }
180:   void setRegisterAlias(const MachineFunction *MF, Register Reg,
181:                         MCRegister AliasReg) {
182:     RegisterAliasTable[MF][Reg] = AliasReg;
183:   }
184:   MCRegister getRegisterAlias(const MachineFunction *MF, Register Reg) {
185:     auto &RegTable = RegisterAliasTable[MF];
186:     auto RI = RegTable.find(Reg);
187:     if (RI == RegTable.end()) {
188:       return MCRegister();
189:     }
190:     return RI->second;
191:   }
192:   bool hasRegisterAlias(const MachineFunction *MF, Register Reg) {
193:     auto RI = RegisterAliasTable.find(MF);
194:     if (RI == RegisterAliasTable.end())
195:       return false;
196:     return RI->second.find(Reg) != RI->second.end();
197:   }
198:   unsigned getNextID() { return MaxID++; }
199:   MCRegister getNextIDRegister() {
200:     return MCRegister((1U << 31) | getNextID());
```
- EN: This range declares interfaces or inline helpers such as getGlobalObjReg, assert, lookup, getExtInstSetReg, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getGlobalObjReg、assert、lookup、getExtInstSetReg 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 201-240
```cpp
201:   }
202:   bool hasMBBRegister(const MachineBasicBlock &MBB) {
203:     auto Key = std::make_pair(MBB.getParent(), MBB.getNumber());
204:     return BBNumToRegMap.contains(Key);
205:   }
206:   // Convert MBB's number to corresponding ID register.
207:   MCRegister getOrCreateMBBRegister(const MachineBasicBlock &MBB) {
208:     auto Key = std::make_pair(MBB.getParent(), MBB.getNumber());
209:     auto [It, Inserted] = BBNumToRegMap.try_emplace(Key);
210:     if (Inserted)
211:       It->second = getNextIDRegister();
212:     return It->second;
213:   }
214: };
215: } // namespace SPIRV
216:
217: using InstrSignature = SmallVector<size_t>;
218: using InstrTraces = std::set<InstrSignature>;
219: using InstrGRegsMap = std::map<SmallVector<size_t>, unsigned>;
220:
221: struct SPIRVModuleAnalysis : public ModulePass {
222:   static char ID;
223:
224: public:
225:   SPIRVModuleAnalysis()
226:       : ModulePass(ID), ST(nullptr), GR(nullptr), TII(nullptr), MMI(nullptr) {}
227:
228:   bool runOnModule(Module &M) override;
229:   void getAnalysisUsage(AnalysisUsage &AU) const override;
230:   SPIRV::ModuleAnalysisInfo MAI;
231:
232: private:
233:   void setBaseInfo(const Module &M);
234:   void collectFuncNames(MachineInstr &MI, const Function *F);
235:   void processOtherInstrs(const Module &M);
236:   void numberRegistersGlobally(const Module &M);
237:
238:   // analyze dependencies to collect module scope definitions
239:   void collectDeclarations(const Module &M);
240:   void visitDecl(const MachineRegisterInfo &MRI, InstrGRegsMap &SignatureToGReg,
```
- EN: This range defines or declares important types such as hasMBBRegister, std::make_pair, contains, getOrCreateMBBRegister, shaping the data model used by SPIRVModuleAnalysis.h.
- CN: 这一段定义或声明了 hasMBBRegister、std::make_pair、contains、getOrCreateMBBRegister 等关键类型，构成 SPIRVModuleAnalysis.h 使用的数据模型。

### Lines 241-262
```cpp
241:                  std::map<const Value *, unsigned> &GlobalToGReg,
242:                  const MachineFunction *MF, const MachineInstr &MI);
243:   MCRegister handleVariable(const MachineFunction *MF, const MachineInstr &MI,
244:                             std::map<const Value *, unsigned> &GlobalToGReg);
245:   MCRegister handleTypeDeclOrConstant(const MachineInstr &MI,
246:                                       InstrGRegsMap &SignatureToGReg);
247:   MCRegister
248:   handleFunctionOrParameter(const MachineFunction *MF, const MachineInstr &MI,
249:                             std::map<const Value *, unsigned> &GlobalToGReg,
250:                             bool &IsFunDef);
251:   void visitFunPtrUse(Register OpReg, InstrGRegsMap &SignatureToGReg,
252:                       std::map<const Value *, unsigned> &GlobalToGReg,
253:                       const MachineFunction *MF, const MachineInstr &MI);
254:   bool isDeclSection(const MachineRegisterInfo &MRI, const MachineInstr &MI);
255:
256:   const SPIRVSubtarget *ST;
257:   SPIRVGlobalRegistry *GR;
258:   const SPIRVInstrInfo *TII;
259:   MachineModuleInfo *MMI;
260: };
261: } // namespace llvm
262: #endif // LLVM_LIB_TARGET_SPIRV_SPIRVMODULEANALYSIS_H
```
- EN: This range declares interfaces or inline helpers such as isDeclSection, defining how other backend pieces interact with this header.
- CN: 这一段声明了 isDeclSection 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVSubtarget, MachineFunction, MachineModuleInfo, ModuleSectionType, Requirements, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVSubtarget, MachineFunction, MachineModuleInfo, ModuleSectionType, Requirements，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `MCTargetDesc/SPIRVBaseInfo.h`
  - `SPIRVGlobalRegistry.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/DenseMap.h`
  - `llvm/ADT/SmallSet.h`
  - `llvm/ADT/SmallVector.h`
