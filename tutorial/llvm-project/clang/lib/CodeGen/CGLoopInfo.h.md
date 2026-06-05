# CGLoopInfo.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/CGLoopInfo.h`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Declares the CGLoopInfo interfaces, data structures, and helper APIs used by Clang CodeGen.
- **Purpose (CN) / 目的（中文）**: 声明 Clang CodeGen 使用的 CGLoopInfo 接口、数据结构与辅助 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: //===---- CGLoopInfo.h - LLVM CodeGen for loop metadata -*- C++ -*---------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This is the internal state used for llvm translation for loop statement
10: // metadata.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #ifndef LLVM_CLANG_LIB_CODEGEN_CGLOOPINFO_H
15: #define LLVM_CLANG_LIB_CODEGEN_CGLOOPINFO_H
16: 
```
- **EN**: This block contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块包含影响本编译单元构建方式的预处理结构。

### Lines 17-32
```cpp
17: #include "llvm/ADT/ArrayRef.h"
18: #include "llvm/ADT/SmallVector.h"
19: #include "llvm/IR/DebugLoc.h"
20: #include "llvm/IR/Value.h"
21: #include "llvm/Support/Compiler.h"
22: 
23: namespace llvm {
24: class BasicBlock;
25: class Instruction;
26: class MDNode;
27: } // end namespace llvm
28: 
29: namespace clang {
30: class Attr;
31: class ASTContext;
32: class CodeGenOptions;
```
- **EN**: This block imports LLVM headers `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`, `llvm/IR/DebugLoc.h`, and 2 more; opens or references namespaces `llvm`, `clang`; introduces declarations such as `BasicBlock`, `Instruction`, `MDNode`, `Attr`, `ASTContext`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 LLVM 头文件 `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`, `llvm/IR/DebugLoc.h`, and 2 more；打开或引用命名空间 `llvm`, `clang`；给出诸如 `BasicBlock`, `Instruction`, `MDNode`, `Attr`, `ASTContext` 的声明；包含影响本编译单元构建方式的预处理结构。

### Lines 33-48
```cpp
33: namespace CodeGen {
34: 
35: /// Attributes that may be specified on loops.
36: struct LoopAttributes {
37:   explicit LoopAttributes(bool IsParallel = false);
38:   void clear();
39: 
40:   /// Generate llvm.loop.parallel metadata for loads and stores.
41:   bool IsParallel;
42: 
43:   /// State of loop vectorization or unrolling.
44:   enum LVEnableState { Unspecified, Enable, Disable, Full };
45: 
46:   /// Value for llvm.loop.vectorize.enable metadata.
47:   LVEnableState VectorizeEnable;
48: 
```
- **EN**: This block opens or references namespaces `CodeGen`; introduces declarations such as `LoopAttributes`, `LVEnableState`; defines callable entry points like `LoopAttributes`, `clear`.
- **CN**: 该代码块打开或引用命名空间 `CodeGen`；给出诸如 `LoopAttributes`, `LVEnableState` 的声明；定义可调用入口，例如 `LoopAttributes`, `clear`。

### Lines 49-64
```cpp
49:   /// Value for llvm.loop.unroll.* metadata (enable, disable, or full).
50:   LVEnableState UnrollEnable;
51: 
52:   /// Value for llvm.loop.unroll_and_jam.* metadata (enable, disable, or full).
53:   LVEnableState UnrollAndJamEnable;
54: 
55:   /// Value for llvm.loop.vectorize.predicate metadata
56:   LVEnableState VectorizePredicateEnable;
57: 
58:   /// Value for llvm.loop.vectorize.width metadata.
59:   unsigned VectorizeWidth;
60: 
61:   // Value for llvm.loop.vectorize.scalable.enable
62:   LVEnableState VectorizeScalable;
63: 
64:   /// Value for llvm.loop.interleave.count metadata.
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 65-80
```cpp
65:   unsigned InterleaveCount;
66: 
67:   /// llvm.unroll.
68:   unsigned UnrollCount;
69: 
70:   /// llvm.unroll.
71:   unsigned UnrollAndJamCount;
72: 
73:   /// Value for llvm.loop.distribute.enable metadata.
74:   LVEnableState DistributeEnable;
75: 
76:   /// Value for llvm.loop.pipeline.disable metadata.
77:   bool PipelineDisabled;
78: 
79:   /// Value for llvm.licm.disable metadata.
80:   bool LICMDisabled;
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 81-96
```cpp
81: 
82:   /// Value for llvm.loop.pipeline.iicount metadata.
83:   unsigned PipelineInitiationInterval;
84: 
85:   /// Value for 'llvm.loop.align' metadata.
86:   unsigned CodeAlign;
87: 
88:   /// Value for whether the loop is required to make progress.
89:   bool MustProgress;
90: };
91: 
92: /// Information used when generating a structured loop.
93: class LoopInfo {
94: public:
95:   /// Construct a new LoopInfo for the loop with entry Header.
96:   LoopInfo(llvm::BasicBlock *Header, const LoopAttributes &Attrs,
```
- **EN**: This block introduces declarations such as `LoopInfo`.
- **CN**: 该代码块给出诸如 `LoopInfo` 的声明。

### Lines 97-112
```cpp
 97:            const llvm::DebugLoc &StartLoc, const llvm::DebugLoc &EndLoc,
 98:            LoopInfo *Parent);
 99: 
100:   /// Get the loop id metadata for this loop.
101:   llvm::MDNode *getLoopID() const { return TempLoopID.get(); }
102: 
103:   /// Get the header block of this loop.
104:   llvm::BasicBlock *getHeader() const { return Header; }
105: 
106:   /// Get the set of attributes active for this loop.
107:   const LoopAttributes &getAttributes() const { return Attrs; }
108: 
109:   /// Return this loop's access group or nullptr if it does not have one.
110:   llvm::MDNode *getAccessGroup() const { return AccGroup; }
111: 
112:   /// Create the loop's metadata. Must be called after its nested loops have
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 113-128
```cpp
113:   /// been processed.
114:   void finish();
115: 
116:   /// Returns the first outer loop containing this loop if any, nullptr
117:   /// otherwise.
118:   const LoopInfo *getParent() const { return Parent; }
119: 
120: private:
121:   /// Loop ID metadata.
122:   llvm::TempMDTuple TempLoopID;
123:   /// Header block of this loop.
124:   llvm::BasicBlock *Header;
125:   /// The attributes for this loop.
126:   LoopAttributes Attrs;
127:   /// The access group for memory accesses parallel to this loop.
128:   llvm::MDNode *AccGroup = nullptr;
```
- **EN**: This block defines callable entry points like `finish`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `finish`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 129-144
```cpp
129:   /// Start location of this loop.
130:   llvm::DebugLoc StartLoc;
131:   /// End location of this loop.
132:   llvm::DebugLoc EndLoc;
133:   /// The next outer loop, or nullptr if this is the outermost loop.
134:   LoopInfo *Parent;
135:   /// If this loop has unroll-and-jam metadata, this can be set by the inner
136:   /// loop's LoopInfo to set the llvm.loop.unroll_and_jam.followup_inner
137:   /// metadata.
138:   std::optional<llvm::SmallVector<llvm::Metadata *, 4>>
139:       UnrollAndJamInnerFollowup;
140: 
141:   /// Create a followup MDNode that has @p LoopProperties as its attributes.
142:   llvm::MDNode *
143:   createFollowupMetadata(const char *FollowupName,
144:                          llvm::ArrayRef<llvm::Metadata *> LoopProperties);
```
- **EN**: This block spells out callable entry points like `createFollowupMetadata`.
- **CN**: 该代码块给出可调用入口的声明，例如 `createFollowupMetadata`。

### Lines 145-160
```cpp
145: 
146:   /// Create a metadata list for transformations.
147:   ///
148:   /// The methods call each other in case multiple transformations are applied
149:   /// to a loop. The transformation first to be applied will use metadata list
150:   /// of the next transformation in its followup attribute.
151:   ///
152:   /// @param Attrs             The loop's transformations.
153:   /// @param LoopProperties    Non-transformation properties such as debug
154:   ///                          location, parallel accesses and disabled
155:   ///                          transformations. These are added to the returned
156:   ///                          LoopID.
157:   /// @param HasUserTransforms [out] Set to true if the returned MDNode encodes
158:   ///                          at least one transformation.
159:   ///
160:   /// @return A metadata list that can be used for the llvm.loop annotation or
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 161-176
```cpp
161:   ///         followup-attribute.
162:   /// @{
163:   llvm::SmallVector<llvm::Metadata *, 4>
164:   createPipeliningMetadata(const LoopAttributes &Attrs,
165:                            llvm::ArrayRef<llvm::Metadata *> LoopProperties,
166:                            bool &HasUserTransforms);
167:   llvm::SmallVector<llvm::Metadata *, 4>
168:   createPartialUnrollMetadata(const LoopAttributes &Attrs,
169:                               llvm::ArrayRef<llvm::Metadata *> LoopProperties,
170:                               bool &HasUserTransforms);
171:   llvm::SmallVector<llvm::Metadata *, 4>
172:   createUnrollAndJamMetadata(const LoopAttributes &Attrs,
173:                              llvm::ArrayRef<llvm::Metadata *> LoopProperties,
174:                              bool &HasUserTransforms);
175:   llvm::SmallVector<llvm::Metadata *, 4>
176:   createLoopVectorizeMetadata(const LoopAttributes &Attrs,
```
- **EN**: This block spells out callable entry points like `createPipeliningMetadata`, `createPartialUnrollMetadata`, `createUnrollAndJamMetadata`.
- **CN**: 该代码块给出可调用入口的声明，例如 `createPipeliningMetadata`, `createPartialUnrollMetadata`, `createUnrollAndJamMetadata`。

### Lines 177-192
```cpp
177:                               llvm::ArrayRef<llvm::Metadata *> LoopProperties,
178:                               bool &HasUserTransforms);
179:   llvm::SmallVector<llvm::Metadata *, 4>
180:   createLoopDistributeMetadata(const LoopAttributes &Attrs,
181:                                llvm::ArrayRef<llvm::Metadata *> LoopProperties,
182:                                bool &HasUserTransforms);
183:   llvm::SmallVector<llvm::Metadata *, 4>
184:   createFullUnrollMetadata(const LoopAttributes &Attrs,
185:                            llvm::ArrayRef<llvm::Metadata *> LoopProperties,
186:                            bool &HasUserTransforms);
187: 
188:   /// @}
189: 
190:   /// Create a metadata list for this loop, including transformation-unspecific
191:   /// metadata such as debug location.
192:   ///
```
- **EN**: This block spells out callable entry points like `createLoopDistributeMetadata`, `createFullUnrollMetadata`.
- **CN**: 该代码块给出可调用入口的声明，例如 `createLoopDistributeMetadata`, `createFullUnrollMetadata`。

### Lines 193-208
```cpp
193:   /// @param Attrs             This loop's attributes and transformations.
194:   /// @param LoopProperties    Additional non-transformation properties to add
195:   ///                          to the LoopID, such as transformation-specific
196:   ///                          metadata that are not covered by @p Attrs.
197:   /// @param HasUserTransforms [out] Set to true if the returned MDNode encodes
198:   ///                          at least one transformation.
199:   ///
200:   /// @return A metadata list that can be used for the llvm.loop annotation.
201:   llvm::SmallVector<llvm::Metadata *, 4>
202:   createMetadata(const LoopAttributes &Attrs,
203:                  llvm::ArrayRef<llvm::Metadata *> LoopProperties,
204:                  bool &HasUserTransforms);
205: };
206: 
207: /// A stack of loop information corresponding to loop nesting levels.
208: /// This stack can be used to prepare attributes which are applied when a loop
```
- **EN**: This block spells out callable entry points like `createMetadata`.
- **CN**: 该代码块给出可调用入口的声明，例如 `createMetadata`。

### Lines 209-224
```cpp
209: /// is emitted.
210: class LoopInfoStack {
211:   LoopInfoStack(const LoopInfoStack &) = delete;
212:   void operator=(const LoopInfoStack &) = delete;
213: 
214: public:
215:   LoopInfoStack() {}
216: 
217:   /// Begin a new structured loop. The set of staged attributes will be
218:   /// applied to the loop and then cleared.
219:   void push(llvm::BasicBlock *Header, const llvm::DebugLoc &StartLoc,
220:             const llvm::DebugLoc &EndLoc);
221: 
222:   /// Begin a new structured loop. Stage attributes from the Attrs list.
223:   /// The staged attributes are applied to the loop and then cleared.
224:   void push(llvm::BasicBlock *Header, clang::ASTContext &Ctx,
```
- **EN**: This block introduces declarations such as `LoopInfoStack`; defines callable entry points like `LoopInfoStack`, `push`.
- **CN**: 该代码块给出诸如 `LoopInfoStack` 的声明；定义可调用入口，例如 `LoopInfoStack`, `push`。

### Lines 225-240
```cpp
225:             const clang::CodeGenOptions &CGOpts,
226:             llvm::ArrayRef<const Attr *> Attrs, const llvm::DebugLoc &StartLoc,
227:             const llvm::DebugLoc &EndLoc, bool MustProgress = false);
228: 
229:   /// End the current loop.
230:   void pop();
231: 
232:   /// Return the top loop id metadata.
233:   llvm::MDNode *getCurLoopID() const { return getInfo().getLoopID(); }
234: 
235:   /// Return true if the top loop is parallel.
236:   bool getCurLoopParallel() const {
237:     return hasInfo() ? getInfo().getAttributes().IsParallel : false;
238:   }
239: 
240:   /// Function called by the CodeGenFunction when an instruction is
```
- **EN**: This block defines callable entry points like `pop`, `getCurLoopParallel`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `pop`, `getCurLoopParallel`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 241-256
```cpp
241:   /// created.
242:   void InsertHelper(llvm::Instruction *I) const;
243: 
244:   /// Set the next pushed loop as parallel.
245:   void setParallel(bool Enable = true) { StagedAttrs.IsParallel = Enable; }
246: 
247:   /// Set the next pushed loop 'vectorize.enable'
248:   void setVectorizeEnable(bool Enable = true) {
249:     StagedAttrs.VectorizeEnable =
250:         Enable ? LoopAttributes::Enable : LoopAttributes::Disable;
251:   }
252: 
253:   /// Set the next pushed loop as a distribution candidate.
254:   void setDistributeState(bool Enable = true) {
255:     StagedAttrs.DistributeEnable =
256:         Enable ? LoopAttributes::Enable : LoopAttributes::Disable;
```
- **EN**: This block defines callable entry points like `InsertHelper`, `setParallel`, `setVectorizeEnable`, `setDistributeState`.
- **CN**: 该代码块定义可调用入口，例如 `InsertHelper`, `setParallel`, `setVectorizeEnable`, `setDistributeState`。

### Lines 257-272
```cpp
257:   }
258: 
259:   /// Set the next pushed loop LICM disable state.
260:   void setLICMDisabled(bool Disabled = true) {
261:     StagedAttrs.LICMDisabled = Disabled;
262:   }
263: 
264:   /// Set the next pushed loop unroll state.
265:   void setUnrollState(const LoopAttributes::LVEnableState &State) {
266:     StagedAttrs.UnrollEnable = State;
267:   }
268: 
269:   /// Set the next pushed vectorize predicate state.
270:   void setVectorizePredicateState(const LoopAttributes::LVEnableState &State) {
271:     StagedAttrs.VectorizePredicateEnable = State;
272:   }
```
- **EN**: This block defines callable entry points like `setLICMDisabled`, `setUnrollState`, `setVectorizePredicateState`.
- **CN**: 该代码块定义可调用入口，例如 `setLICMDisabled`, `setUnrollState`, `setVectorizePredicateState`。

### Lines 273-288
```cpp
273: 
274:   /// Set the next pushed loop unroll_and_jam state.
275:   void setUnrollAndJamState(const LoopAttributes::LVEnableState &State) {
276:     StagedAttrs.UnrollAndJamEnable = State;
277:   }
278: 
279:   /// Set the vectorize width for the next loop pushed.
280:   void setVectorizeWidth(unsigned W) { StagedAttrs.VectorizeWidth = W; }
281: 
282:   void setVectorizeScalable(const LoopAttributes::LVEnableState &State) {
283:     StagedAttrs.VectorizeScalable = State;
284:   }
285: 
286:   /// Set the interleave count for the next loop pushed.
287:   void setInterleaveCount(unsigned C) { StagedAttrs.InterleaveCount = C; }
288: 
```
- **EN**: This block defines callable entry points like `setUnrollAndJamState`, `setVectorizeWidth`, `setVectorizeScalable`, `setInterleaveCount`.
- **CN**: 该代码块定义可调用入口，例如 `setUnrollAndJamState`, `setVectorizeWidth`, `setVectorizeScalable`, `setInterleaveCount`。

### Lines 289-304
```cpp
289:   /// Set the unroll count for the next loop pushed.
290:   void setUnrollCount(unsigned C) { StagedAttrs.UnrollCount = C; }
291: 
292:   /// \brief Set the unroll count for the next loop pushed.
293:   void setUnrollAndJamCount(unsigned C) { StagedAttrs.UnrollAndJamCount = C; }
294: 
295:   /// Set the pipeline disabled state.
296:   void setPipelineDisabled(bool S) { StagedAttrs.PipelineDisabled = S; }
297: 
298:   /// Set the pipeline initiation interval.
299:   void setPipelineInitiationInterval(unsigned C) {
300:     StagedAttrs.PipelineInitiationInterval = C;
301:   }
302: 
303:   /// Set value of code align for the next loop pushed.
304:   void setCodeAlign(unsigned C) { StagedAttrs.CodeAlign = C; }
```
- **EN**: This block defines callable entry points like `setUnrollCount`, `setUnrollAndJamCount`, `setPipelineDisabled`, `setPipelineInitiationInterval`, `setCodeAlign`.
- **CN**: 该代码块定义可调用入口，例如 `setUnrollCount`, `setUnrollAndJamCount`, `setPipelineDisabled`, `setPipelineInitiationInterval`, `setCodeAlign`。

### Lines 305-320
```cpp
305: 
306:   /// Set no progress for the next loop pushed.
307:   void setMustProgress(bool P) { StagedAttrs.MustProgress = P; }
308: 
309:   /// Returns true if there is LoopInfo on the stack.
310:   bool hasInfo() const { return !Active.empty(); }
311:   /// Return the LoopInfo for the current loop. HasInfo should be called
312:   /// first to ensure LoopInfo is present.
313:   const LoopInfo &getInfo() const { return *Active.back(); }
314: 
315: private:
316:   /// The set of attributes that will be applied to the next pushed loop.
317:   LoopAttributes StagedAttrs;
318:   /// Stack of active loops.
319:   llvm::SmallVector<std::unique_ptr<LoopInfo>, 4> Active;
320: };
```
- **EN**: This block defines callable entry points like `setMustProgress`, `hasInfo`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `setMustProgress`, `hasInfo`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 321-325
```cpp
321: 
322: } // end namespace CodeGen
323: } // end namespace clang
324: 
325: #endif
```
- **EN**: This block opens or references namespaces `CodeGen`, `clang`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块打开或引用命名空间 `CodeGen`, `clang`；包含影响本编译单元构建方式的预处理结构。

## Key Concepts / 关键概念

- **LoopAttributes**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **StagedAttrs**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Metadata**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **LVEnableState**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Attrs**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **ArrayRef**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **SmallVector**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **DebugLoc**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **LLVM libraries / LLVM 库**: `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`, `llvm/IR/DebugLoc.h`, `llvm/IR/Value.h`, `llvm/Support/Compiler.h`
