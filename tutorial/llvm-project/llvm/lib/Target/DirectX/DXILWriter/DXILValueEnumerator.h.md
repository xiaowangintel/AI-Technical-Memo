# DXILValueEnumerator.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILWriter/DXILValueEnumerator.h`
- Repository: `llvm-project`
- Purpose (EN): DirectX DXILWriter ValueEnumerator.h - Number values -----*- C++ -*-===.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- DirectX/DXILWriter/ValueEnumerator.h - Number values -----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This class gives values and types Unique ID's.
10: // Forked from lib/Bitcode/Writer
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #ifndef LLVM_DXILWRITER_VALUEENUMERATOR_H
15: #define LLVM_DXILWRITER_VALUEENUMERATOR_H
16:
17: #include "llvm/ADT/ArrayRef.h"
18: #include "llvm/ADT/DenseMap.h"
19: #include "llvm/ADT/UniqueVector.h"
20: #include "llvm/IR/Attributes.h"
21: #include "llvm/IR/UseListOrder.h"
22: #include <cassert>
23: #include <cstdint>
24: #include <utility>
25: #include <vector>
26:
27: namespace llvm {
28:
29: class BasicBlock;
30: class Comdat;
31: class DIArgList;
32: class Function;
33: class Instruction;
34: class LocalAsMetadata;
35: class MDNode;
36: class Metadata;
37: class Module;
38: class NamedMDNode;
39: class raw_ostream;
40: class Type;
41: class Value;
42: class ValueSymbolTable;
43:
44: namespace dxil {
45:
46: class DXILDebugInfoMap;
47:
48: class ValueEnumerator {
49: public:
50:   using TypeList = std::vector<Type *>;
51:
52:   // For each value, we remember its Value* and occurrence frequency.
53:   using ValueList = std::vector<std::pair<const Value *, unsigned>>;
54:
55:   /// Attribute groups as encoded in bitcode are almost AttributeSets, but they
56:   /// include the AttributeList index, so we have to track that in our map.
57:   using IndexAndAttrSet = std::pair<unsigned, AttributeSet>;
58:
59:   UseListOrderStack UseListOrders;
60:
```
- EN: This range defines or declares important types such as BasicBlock, Comdat, DIArgList, Function, shaping the data model used by DXILValueEnumerator.h.
- CN: 这一段定义或声明了 BasicBlock、Comdat、DIArgList、Function 等关键类型，构成 DXILValueEnumerator.h 使用的数据模型。

### Lines 61-120
```cpp
 61: private:
 62:   using TypeMapType = DenseMap<Type *, unsigned>;
 63:   TypeMapType TypeMap;
 64:   TypeList Types;
 65:
 66:   using ValueMapType = DenseMap<const Value *, unsigned>;
 67:   ValueMapType ValueMap;
 68:   ValueList Values;
 69:
 70:   using ComdatSetType = UniqueVector<const Comdat *>;
 71:   ComdatSetType Comdats;
 72:
 73:   std::vector<const Metadata *> MDs;
 74:   std::vector<const Metadata *> FunctionMDs;
 75:
 76:   /// Index of information about a piece of metadata.
 77:   struct MDIndex {
 78:     unsigned F = 0;  ///< The ID of the function for this metadata, if any.
 79:     unsigned ID = 0; ///< The implicit ID of this metadata in bitcode.
 80:
 81:     MDIndex() = default;
 82:     explicit MDIndex(unsigned F) : F(F) {}
 83:
 84:     /// Check if this has a function tag, and it's different from NewF.
 85:     bool hasDifferentFunction(unsigned NewF) const { return F && F != NewF; }
 86:
 87:     /// Fetch the MD this references out of the given metadata array.
 88:     const Metadata *get(ArrayRef<const Metadata *> MDs) const {
 89:       assert(ID && "Expected non-zero ID");
 90:       assert(ID <= MDs.size() && "Expected valid ID");
 91:       return MDs[ID - 1];
 92:     }
 93:   };
 94:
 95:   using MetadataMapType = DenseMap<const Metadata *, MDIndex>;
 96:   MetadataMapType MetadataMap;
 97:
 98:   /// Range of metadata IDs, as a half-open range.
 99:   struct MDRange {
100:     unsigned First = 0;
101:     unsigned Last = 0;
102:
103:     /// Number of strings in the prefix of the metadata range.
104:     unsigned NumStrings = 0;
105:
106:     MDRange() = default;
107:     explicit MDRange(unsigned First) : First(First) {}
108:   };
109:   SmallDenseMap<unsigned, MDRange, 1> FunctionMDInfo;
110:
111:   using AttributeGroupMapType = DenseMap<IndexAndAttrSet, unsigned>;
112:   AttributeGroupMapType AttributeGroupMap;
113:   std::vector<IndexAndAttrSet> AttributeGroups;
114:
115:   using AttributeListMapType = DenseMap<AttributeList, unsigned>;
116:   AttributeListMapType AttributeListMap;
117:   std::vector<AttributeList> AttributeLists;
118:
119:   /// GlobalBasicBlockIDs - This map memoizes the basic block ID's referenced by
120:   /// the "getGlobalBasicBlockID" method.
```
- EN: This range defines or declares important types such as MDIndex, hasDifferentFunction, get, assert, shaping the data model used by DXILValueEnumerator.h.
- CN: 这一段定义或声明了 MDIndex、hasDifferentFunction、get、assert 等关键类型，构成 DXILValueEnumerator.h 使用的数据模型。

### Lines 121-180
```cpp
121:   mutable DenseMap<const BasicBlock *, unsigned> GlobalBasicBlockIDs;
122:
123:   using InstructionMapType = DenseMap<const Instruction *, unsigned>;
124:   InstructionMapType InstructionMap;
125:   unsigned InstructionCount;
126:
127:   /// BasicBlocks - This contains all the basic blocks for the currently
128:   /// incorporated function.  Their reverse mapping is stored in ValueMap.
129:   std::vector<const BasicBlock *> BasicBlocks;
130:
131:   /// When a function is incorporated, this is the size of the Values list
132:   /// before incorporation.
133:   unsigned NumModuleValues;
134:
135:   /// When a function is incorporated, this is the size of the Metadatas list
136:   /// before incorporation.
137:   unsigned NumModuleMDs = 0;
138:   unsigned NumMDStrings = 0;
139:
140:   unsigned FirstFuncConstantID;
141:   unsigned FirstInstID;
142:
143:   const DXILDebugInfoMap &DebugInfo;
144:
145: public:
146:   ValueEnumerator(const Module &M, Type *PrefixType,
147:                   const DXILDebugInfoMap &DebugInfo);
148:   ValueEnumerator(const ValueEnumerator &) = delete;
149:   ValueEnumerator &operator=(const ValueEnumerator &) = delete;
150:
151:   void dump() const;
152:   void print(raw_ostream &OS, const ValueMapType &Map, const char *Name) const;
153:   void print(raw_ostream &OS, const MetadataMapType &Map,
154:              const char *Name) const;
155:
156:   unsigned getValueID(const Value *V) const;
157:
158:   unsigned getMetadataID(const Metadata *MD) const {
159:     auto ID = getMetadataOrNullID(MD);
160:     assert(ID != 0 && "Metadata not in slotcalculator!");
161:     return ID - 1;
162:   }
163:
164:   unsigned getMetadataOrNullID(const Metadata *MD) const {
165:     return MetadataMap.lookup(getDXILMetadata(MD)).ID;
166:   }
167:
168:   unsigned numMDs() const { return MDs.size(); }
169:
170:   unsigned getTypeID(Type *T) const {
171:     TypeMapType::const_iterator I = TypeMap.find(T);
172:     assert(I != TypeMap.end() && "Type not in ValueEnumerator!");
173:     return I->second - 1;
174:   }
175:
176:   unsigned getInstructionID(const Instruction *I) const;
177:   void setInstructionID(const Instruction *I);
178:
179:   unsigned getAttributeListID(AttributeList PAL) const {
180:     if (PAL.isEmpty())
```
- EN: This range declares interfaces or inline helpers such as dump, print, getValueID, getMetadataID, defining how other backend pieces interact with this header.
- CN: 这一段声明了 dump、print、getValueID、getMetadataID 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 181-240
```cpp
181:       return 0; // Null maps to zero.
182:     AttributeListMapType::const_iterator I = AttributeListMap.find(PAL);
183:     assert(I != AttributeListMap.end() && "Attribute not in ValueEnumerator!");
184:     return I->second;
185:   }
186:
187:   unsigned getAttributeGroupID(IndexAndAttrSet Group) const {
188:     if (!Group.second.hasAttributes())
189:       return 0; // Null maps to zero.
190:     AttributeGroupMapType::const_iterator I = AttributeGroupMap.find(Group);
191:     assert(I != AttributeGroupMap.end() && "Attribute not in ValueEnumerator!");
192:     return I->second;
193:   }
194:
195:   /// getFunctionConstantRange - Return the range of values that corresponds to
196:   /// function-local constants.
197:   void getFunctionConstantRange(unsigned &Start, unsigned &End) const {
198:     Start = FirstFuncConstantID;
199:     End = FirstInstID;
200:   }
201:
202:   const ValueList &getValues() const { return Values; }
203:
204:   /// Check whether the current block has any metadata to emit.
205:   bool hasMDs() const { return NumModuleMDs < MDs.size(); }
206:
207:   /// Get the MDString metadata for this block.
208:   ArrayRef<const Metadata *> getMDStrings() const {
209:     return ArrayRef(MDs).slice(NumModuleMDs, NumMDStrings);
210:   }
211:
212:   /// Get the non-MDString metadata for this block.
213:   ArrayRef<const Metadata *> getNonMDStrings() const {
214:     return ArrayRef(MDs).slice(NumModuleMDs).slice(NumMDStrings);
215:   }
216:
217:   const TypeList &getTypes() const { return Types; }
218:
219:   const std::vector<const BasicBlock *> &getBasicBlocks() const {
220:     return BasicBlocks;
221:   }
222:
223:   const std::vector<AttributeList> &getAttributeLists() const {
224:     return AttributeLists;
225:   }
226:
227:   const std::vector<IndexAndAttrSet> &getAttributeGroups() const {
228:     return AttributeGroups;
229:   }
230:
231:   const ComdatSetType &getComdats() const { return Comdats; }
232:   unsigned getComdatID(const Comdat *C) const;
233:
234:   /// getGlobalBasicBlockID - This returns the function-specific ID for the
235:   /// specified basic block.  This is relatively expensive information, so it
236:   /// should only be used by rare constructs such as address-of-label.
237:   unsigned getGlobalBasicBlockID(const BasicBlock *BB) const;
238:
239:   const Metadata *getDXILMetadata(const Metadata *M) const;
240:
```
- EN: This range declares interfaces or inline helpers such as find, assert, getAttributeGroupID, getFunctionConstantRange, defining how other backend pieces interact with this header.
- CN: 这一段声明了 find、assert、getAttributeGroupID、getFunctionConstantRange 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 241-300
```cpp
241:   /// incorporateFunction/purgeFunction - If you'd like to deal with a function,
242:   /// use these two methods to get its data into the ValueEnumerator!
243:   void incorporateFunction(const Function &F);
244:
245:   void purgeFunction();
246:   uint64_t computeBitsRequiredForTypeIndices() const;
247:
248:   void EnumerateType(Type *T);
249:
250: private:
251:
252:   /// Reorder the reachable metadata.
253:   ///
254:   /// This is not just an optimization, but is mandatory for emitting MDString
255:   /// correctly.
256:   void organizeMetadata();
257:
258:   /// Drop the function tag from the transitive operands of the given node.
259:   void dropFunctionFromMetadata(MetadataMapType::value_type &FirstMD);
260:
261:   /// Incorporate the function metadata.
262:   ///
263:   /// This should be called before enumerating LocalAsMetadata for the
264:   /// function.
265:   void incorporateFunctionMetadata(const Function &F);
266:
267:   /// Enumerate a single instance of metadata with the given function tag.
268:   ///
269:   /// If \c MD has already been enumerated, check that \c F matches its
270:   /// function tag.  If not, call \a dropFunctionFromMetadata().
271:   ///
272:   /// Otherwise, mark \c MD as visited.  Assign it an ID, or just return it if
273:   /// it's an \a MDNode.
274:   const MDNode *enumerateMetadataImpl(unsigned F, const Metadata *MD);
275:
276:   unsigned getMetadataFunctionID(const Function *F) const;
277:
278:   /// Enumerate reachable metadata in (almost) post-order.
279:   ///
280:   /// Enumerate all the metadata reachable from MD.  We want to minimize the
281:   /// cost of reading bitcode records, and so the primary consideration is that
282:   /// operands of uniqued nodes are resolved before the nodes are read.  This
283:   /// avoids re-uniquing them on the context and factors away RAUW support.
284:   ///
285:   /// This algorithm guarantees that subgraphs of uniqued nodes are in
286:   /// post-order.  Distinct subgraphs reachable only from a single uniqued node
287:   /// will be in post-order.
288:   ///
289:   /// \note The relative order of a distinct and uniqued node is irrelevant.
290:   /// \a organizeMetadata() will later partition distinct nodes ahead of
291:   /// uniqued ones.
292:   ///{
293:   void EnumerateMetadata(const Function *F, const Metadata *MD);
294:   void EnumerateMetadata(unsigned F, const Metadata *MD);
295:   ///}
296:
297:   void EnumerateFunctionLocalMetadata(const Function &F,
298:                                       const LocalAsMetadata *Local);
299:   void EnumerateFunctionLocalMetadata(unsigned F, const LocalAsMetadata *Local);
300:   void EnumerateFunctionLocalListMetadata(const Function &F,
```
- EN: This range declares interfaces or inline helpers such as incorporateFunction, purgeFunction, computeBitsRequiredForTypeIndices, EnumerateType, defining how other backend pieces interact with this header.
- CN: 这一段声明了 incorporateFunction、purgeFunction、computeBitsRequiredForTypeIndices、EnumerateType 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 301-315
```cpp
301:                                           const DIArgList *ArgList);
302:   void EnumerateFunctionLocalListMetadata(unsigned F, const DIArgList *Arglist);
303:   void EnumerateNamedMDNode(const NamedMDNode *NMD);
304:   void EnumerateValue(const Value *V);
305:   void EnumerateOperandType(const Value *V);
306:   void EnumerateAttributes(AttributeList PAL);
307:
308:   void EnumerateValueSymbolTable(const ValueSymbolTable &ST);
309:   void EnumerateNamedMetadata(const Module &M);
310: };
311:
312: } // end namespace dxil
313: } // end namespace llvm
314:
315: #endif // LLVM_DXILWRITER_VALUEENUMERATOR_H
```
- EN: This range declares interfaces or inline helpers such as EnumerateFunctionLocalListMetadata, EnumerateNamedMDNode, EnumerateValue, EnumerateOperandType, defining how other backend pieces interact with this header.
- CN: 这一段声明了 EnumerateFunctionLocalListMetadata、EnumerateNamedMDNode、EnumerateValue、EnumerateOperandType 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include BasicBlock, Comdat, DIArgList, Function, Instruction, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 BasicBlock, Comdat, DIArgList, Function, Instruction，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/ArrayRef.h`
  - `llvm/ADT/DenseMap.h`
  - `llvm/ADT/UniqueVector.h`
  - `llvm/IR/Attributes.h`
  - `llvm/IR/UseListOrder.h`
- System/standard headers / 系统或标准头文件:
  - `cassert`
  - `cstdint`
  - `utility`
  - `vector`
