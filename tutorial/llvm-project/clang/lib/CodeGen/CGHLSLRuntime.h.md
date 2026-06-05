# CGHLSLRuntime.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/CGHLSLRuntime.h`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Declares the CGHLSLRuntime interfaces, data structures, and helper APIs used by Clang CodeGen.
- **Purpose (CN) / 目的（中文）**: 声明 Clang CodeGen 使用的 CGHLSLRuntime 接口、数据结构与辅助 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: //===----- CGHLSLRuntime.h - Interface to HLSL Runtimes -----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This provides an abstract class for HLSL code generation.  Concrete
10: // subclasses of this implement code generation for specific HLSL
11: // runtime libraries.
12: //
13: //===----------------------------------------------------------------------===//
14: 
15: #ifndef LLVM_CLANG_LIB_CODEGEN_CGHLSLRUNTIME_H
16: #define LLVM_CLANG_LIB_CODEGEN_CGHLSLRUNTIME_H
```
- **EN**: This block contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块包含影响本编译单元构建方式的预处理结构。

### Lines 17-32
```cpp
17: 
18: #include "Address.h"
19: #include "clang/AST/Attr.h"
20: #include "clang/AST/Decl.h"
21: #include "clang/Basic/Builtins.h"
22: #include "clang/Basic/HLSLRuntime.h"
23: #include "llvm/ADT/DenseMap.h"
24: #include "llvm/ADT/SmallVector.h"
25: #include "llvm/ADT/StringRef.h"
26: #include "llvm/Frontend/HLSL/HLSLResource.h"
27: #include "llvm/IR/IRBuilder.h"
28: #include "llvm/IR/Intrinsics.h"
29: #include "llvm/IR/IntrinsicsDirectX.h"
30: #include "llvm/IR/IntrinsicsSPIRV.h"
31: 
32: #include <optional>
```
- **EN**: This block imports local CodeGen headers `Address.h`; Clang headers `clang/AST/Attr.h`, `clang/AST/Decl.h`, `clang/Basic/Builtins.h`, and 1 more; LLVM headers `llvm/ADT/DenseMap.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, and 5 more; other headers `optional`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `Address.h`；Clang 头文件 `clang/AST/Attr.h`, `clang/AST/Decl.h`, `clang/Basic/Builtins.h`, and 1 more；LLVM 头文件 `llvm/ADT/DenseMap.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, and 5 more；其他头文件 `optional`；包含影响本编译单元构建方式的预处理结构。

### Lines 33-48
```cpp
33: #include <vector>
34: 
35: // A function generator macro for picking the right intrinsic
36: // for the target backend
37: #define GENERATE_HLSL_INTRINSIC_FUNCTION(FunctionName, IntrinsicPostfix)       \
38:   llvm::Intrinsic::ID get##FunctionName##Intrinsic() {                         \
39:     llvm::Triple::ArchType Arch = getArch();                                   \
40:     switch (Arch) {                                                            \
41:     case llvm::Triple::dxil:                                                   \
42:       return llvm::Intrinsic::dx_##IntrinsicPostfix;                           \
43:     case llvm::Triple::spirv:                                                  \
44:       return llvm::Intrinsic::spv_##IntrinsicPostfix;                          \
45:     default:                                                                   \
46:       llvm_unreachable("Intrinsic " #IntrinsicPostfix                          \
47:                        " not supported by target architecture");               \
48:     }                                                                          \
```
- **EN**: This block imports other headers `vector`; uses control flow (switch, case) to specialize LLVM IR emission; guards important invariants with assertions or unreachable markers; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 其他头文件 `vector`；通过控制流（switch, case）细化 LLVM IR 生成 行为；使用断言或不可达标记保护关键不变量；包含影响本编译单元构建方式的预处理结构。

### Lines 49-64
```cpp
49:   }
50: 
51: using ResourceClass = llvm::dxil::ResourceClass;
52: 
53: namespace llvm {
54: class GlobalVariable;
55: class Function;
56: class StructType;
57: class Metadata;
58: } // namespace llvm
59: 
60: namespace clang {
61: class NamedDecl;
62: class VarDecl;
63: class ParmVarDecl;
64: class InitListExpr;
```
- **EN**: This block opens or references namespaces `llvm`, `clang`; introduces declarations such as `GlobalVariable`, `Function`, `StructType`, `Metadata`, `NamedDecl`.
- **CN**: 该代码块打开或引用命名空间 `llvm`, `clang`；给出诸如 `GlobalVariable`, `Function`, `StructType`, `Metadata`, `NamedDecl` 的声明。

### Lines 65-80
```cpp
65: class HLSLBufferDecl;
66: class HLSLRootSignatureDecl;
67: class HLSLVkBindingAttr;
68: class HLSLResourceBindingAttr;
69: class Type;
70: class RecordType;
71: class DeclContext;
72: class HLSLPackOffsetAttr;
73: class ArraySubscriptExpr;
74: 
75: class FunctionDecl;
76: 
77: namespace CodeGen {
78: 
79: class CodeGenModule;
80: class CodeGenFunction;
```
- **EN**: This block opens or references namespaces `CodeGen`; introduces declarations such as `HLSLBufferDecl`, `HLSLRootSignatureDecl`, `HLSLVkBindingAttr`, `HLSLResourceBindingAttr`, `Type`.
- **CN**: 该代码块打开或引用命名空间 `CodeGen`；给出诸如 `HLSLBufferDecl`, `HLSLRootSignatureDecl`, `HLSLVkBindingAttr`, `HLSLResourceBindingAttr`, `Type` 的声明。

### Lines 81-96
```cpp
81: class LValue;
82: 
83: class CGHLSLOffsetInfo {
84:   SmallVector<uint32_t> Offsets;
85: 
86: public:
87:   static const uint32_t Unspecified = ~0U;
88: 
89:   /// Iterates over all declarations in the HLSL buffer and based on the
90:   /// packoffset or register(c#) annotations it fills outs the Offsets vector
91:   /// with the user-specified layout offsets. The buffer offsets can be
92:   /// specified 2 ways: 1. declarations in cbuffer {} block can have a
93:   /// packoffset annotation (translates to HLSLPackOffsetAttr) 2. default
94:   /// constant buffer declarations at global scope can have register(c#)
95:   /// annotations (translates to HLSLResourceBindingAttr with RegisterType::C)
96:   /// It is not guaranteed that all declarations in a buffer have an annotation.
```
- **EN**: This block introduces declarations such as `LValue`, `CGHLSLOffsetInfo`.
- **CN**: 该代码块给出诸如 `LValue`, `CGHLSLOffsetInfo` 的声明。

### Lines 97-112
```cpp
 97:   /// For those where it is not specified a `~0U` value is added to the Offsets
 98:   /// vector. In the final layout these declarations will be placed at the end
 99:   /// of the HLSL buffer after all of the elements with specified offset.
100:   static CGHLSLOffsetInfo fromDecl(const HLSLBufferDecl &BufDecl);
101: 
102:   /// Comparison function for offsets received from `operator[]` suitable for
103:   /// use in a `stable_sort`. This will order implicit bindings after explicit
104:   /// offsets.
105:   static bool compareOffsets(uint32_t LHS, uint32_t RHS) { return LHS < RHS; }
106: 
107:   /// Get the given offset, or `~0U` if there is no offset for the member.
108:   uint32_t operator[](size_t I) const {
109:     if (Offsets.empty())
110:       return Unspecified;
111:     return Offsets[I];
112:   }
```
- **EN**: This block defines callable entry points like `fromDecl`, `compareOffsets`; uses control flow (if) to specialize LLVM IR emission.
- **CN**: 该代码块定义可调用入口，例如 `fromDecl`, `compareOffsets`；通过控制流（if）细化 LLVM IR 生成 行为。

### Lines 113-128
```cpp
113: 
114:   bool empty() const { return Offsets.empty(); }
115: };
116: 
117: class CGHLSLRuntime {
118: public:
119:   //===----------------------------------------------------------------------===//
120:   // Start of reserved area for HLSL intrinsic getters.
121:   //===----------------------------------------------------------------------===//
122: 
123:   GENERATE_HLSL_INTRINSIC_FUNCTION(All, all)
124:   GENERATE_HLSL_INTRINSIC_FUNCTION(Any, any)
125:   GENERATE_HLSL_INTRINSIC_FUNCTION(Cross, cross)
126:   GENERATE_HLSL_INTRINSIC_FUNCTION(Degrees, degrees)
127:   GENERATE_HLSL_INTRINSIC_FUNCTION(Frac, frac)
128:   GENERATE_HLSL_INTRINSIC_FUNCTION(FlattenedThreadIdInGroup,
```
- **EN**: This block introduces declarations such as `CGHLSLRuntime`; defines callable entry points like `empty`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块给出诸如 `CGHLSLRuntime` 的声明；定义可调用入口，例如 `empty`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 129-144
```cpp
129:                                    flattened_thread_id_in_group)
130:   GENERATE_HLSL_INTRINSIC_FUNCTION(IsInf, isinf)
131:   GENERATE_HLSL_INTRINSIC_FUNCTION(IsNaN, isnan)
132:   GENERATE_HLSL_INTRINSIC_FUNCTION(Lerp, lerp)
133:   GENERATE_HLSL_INTRINSIC_FUNCTION(Normalize, normalize)
134:   GENERATE_HLSL_INTRINSIC_FUNCTION(Rsqrt, rsqrt)
135:   GENERATE_HLSL_INTRINSIC_FUNCTION(Saturate, saturate)
136:   GENERATE_HLSL_INTRINSIC_FUNCTION(Sign, sign)
137:   GENERATE_HLSL_INTRINSIC_FUNCTION(Step, step)
138:   GENERATE_HLSL_INTRINSIC_FUNCTION(Radians, radians)
139:   GENERATE_HLSL_INTRINSIC_FUNCTION(ThreadId, thread_id)
140:   GENERATE_HLSL_INTRINSIC_FUNCTION(GroupThreadId, thread_id_in_group)
141:   GENERATE_HLSL_INTRINSIC_FUNCTION(GroupId, group_id)
142:   GENERATE_HLSL_INTRINSIC_FUNCTION(FDot, fdot)
143:   GENERATE_HLSL_INTRINSIC_FUNCTION(SDot, sdot)
144:   GENERATE_HLSL_INTRINSIC_FUNCTION(UDot, udot)
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding LLVM IR emission implementation.
- **CN**: 该代码块为周围的 LLVM IR 生成 实现提供必要的胶水代码、布局或分隔结构。

### Lines 145-160
```cpp
145:   GENERATE_HLSL_INTRINSIC_FUNCTION(Dot4AddI8Packed, dot4add_i8packed)
146:   GENERATE_HLSL_INTRINSIC_FUNCTION(Dot4AddU8Packed, dot4add_u8packed)
147:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveActiveAllEqual, wave_all_equal)
148:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveActiveAllTrue, wave_all)
149:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveActiveAnyTrue, wave_any)
150:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveActiveBitOr, wave_reduce_or)
151:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveActiveBitXor, wave_reduce_xor)
152:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveActiveBitAnd, wave_reduce_and)
153:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveActiveMax, wave_reduce_max)
154:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveActiveUMax, wave_reduce_umax)
155:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveActiveMin, wave_reduce_min)
156:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveActiveUMin, wave_reduce_umin)
157:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveActiveCountBits, wave_active_countbits)
158:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveIsFirstLane, wave_is_first_lane)
159:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveGetLaneCount, wave_get_lane_count)
160:   GENERATE_HLSL_INTRINSIC_FUNCTION(WaveReadLaneAt, wave_readlane)
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding LLVM IR emission implementation.
- **CN**: 该代码块为周围的 LLVM IR 生成 实现提供必要的胶水代码、布局或分隔结构。

### Lines 161-176
```cpp
161:   GENERATE_HLSL_INTRINSIC_FUNCTION(QuadReadAcrossX, quad_read_across_x)
162:   GENERATE_HLSL_INTRINSIC_FUNCTION(QuadReadAcrossY, quad_read_across_y)
163:   GENERATE_HLSL_INTRINSIC_FUNCTION(FirstBitUHigh, firstbituhigh)
164:   GENERATE_HLSL_INTRINSIC_FUNCTION(FirstBitSHigh, firstbitshigh)
165:   GENERATE_HLSL_INTRINSIC_FUNCTION(FirstBitLow, firstbitlow)
166:   GENERATE_HLSL_INTRINSIC_FUNCTION(NClamp, nclamp)
167:   GENERATE_HLSL_INTRINSIC_FUNCTION(SClamp, sclamp)
168:   GENERATE_HLSL_INTRINSIC_FUNCTION(UClamp, uclamp)
169: 
170:   GENERATE_HLSL_INTRINSIC_FUNCTION(CreateResourceGetBasePointer,
171:                                    resource_getbasepointer)
172:   GENERATE_HLSL_INTRINSIC_FUNCTION(CreateResourceGetPointer,
173:                                    resource_getpointer)
174:   GENERATE_HLSL_INTRINSIC_FUNCTION(Sample, resource_sample)
175:   GENERATE_HLSL_INTRINSIC_FUNCTION(SampleClamp, resource_sample_clamp)
176:   GENERATE_HLSL_INTRINSIC_FUNCTION(SampleBias, resource_samplebias)
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding LLVM IR emission implementation.
- **CN**: 该代码块为周围的 LLVM IR 生成 实现提供必要的胶水代码、布局或分隔结构。

### Lines 177-192
```cpp
177:   GENERATE_HLSL_INTRINSIC_FUNCTION(SampleBiasClamp, resource_samplebias_clamp)
178:   GENERATE_HLSL_INTRINSIC_FUNCTION(SampleGrad, resource_samplegrad)
179:   GENERATE_HLSL_INTRINSIC_FUNCTION(SampleGradClamp, resource_samplegrad_clamp)
180:   GENERATE_HLSL_INTRINSIC_FUNCTION(SampleLevel, resource_samplelevel)
181:   GENERATE_HLSL_INTRINSIC_FUNCTION(SampleCmp, resource_samplecmp)
182:   GENERATE_HLSL_INTRINSIC_FUNCTION(SampleCmpClamp, resource_samplecmp_clamp)
183:   GENERATE_HLSL_INTRINSIC_FUNCTION(SampleCmpLevelZero,
184:                                    resource_samplecmplevelzero)
185:   GENERATE_HLSL_INTRINSIC_FUNCTION(Gather, resource_gather)
186:   GENERATE_HLSL_INTRINSIC_FUNCTION(GatherCmp, resource_gather_cmp)
187:   GENERATE_HLSL_INTRINSIC_FUNCTION(CreateHandleFromBinding,
188:                                    resource_handlefrombinding)
189:   GENERATE_HLSL_INTRINSIC_FUNCTION(CreateHandleFromImplicitBinding,
190:                                    resource_handlefromimplicitbinding)
191:   GENERATE_HLSL_INTRINSIC_FUNCTION(NonUniformResourceIndex,
192:                                    resource_nonuniformindex)
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding LLVM IR emission implementation.
- **CN**: 该代码块为周围的 LLVM IR 生成 实现提供必要的胶水代码、布局或分隔结构。

### Lines 193-208
```cpp
193:   GENERATE_HLSL_INTRINSIC_FUNCTION(BufferUpdateCounter, resource_updatecounter)
194:   GENERATE_HLSL_INTRINSIC_FUNCTION(AllMemoryBarrier, all_memory_barrier)
195:   GENERATE_HLSL_INTRINSIC_FUNCTION(AllMemoryBarrierWithGroupSync,
196:                                    all_memory_barrier_with_group_sync)
197:   GENERATE_HLSL_INTRINSIC_FUNCTION(DeviceMemoryBarrier, device_memory_barrier)
198:   GENERATE_HLSL_INTRINSIC_FUNCTION(DeviceMemoryBarrierWithGroupSync,
199:                                    device_memory_barrier_with_group_sync)
200:   GENERATE_HLSL_INTRINSIC_FUNCTION(GroupMemoryBarrier, group_memory_barrier)
201:   GENERATE_HLSL_INTRINSIC_FUNCTION(GroupMemoryBarrierWithGroupSync,
202:                                    group_memory_barrier_with_group_sync)
203:   GENERATE_HLSL_INTRINSIC_FUNCTION(GetDimensionsX, resource_getdimensions_x)
204:   GENERATE_HLSL_INTRINSIC_FUNCTION(GetDimensionsXY, resource_getdimensions_xy)
205:   GENERATE_HLSL_INTRINSIC_FUNCTION(GetDimensionsLevelsXY,
206:                                    resource_getdimensions_levels_xy)
207:   GENERATE_HLSL_INTRINSIC_FUNCTION(LoadLevel, resource_load_level)
208:   GENERATE_HLSL_INTRINSIC_FUNCTION(CalculateLod, resource_calculate_lod)
```
- **EN**: This block provides glue code, layout, or delimiters needed by the surrounding LLVM IR emission implementation.
- **CN**: 该代码块为周围的 LLVM IR 生成 实现提供必要的胶水代码、布局或分隔结构。

### Lines 209-224
```cpp
209:   GENERATE_HLSL_INTRINSIC_FUNCTION(CalculateLodUnclamped,
210:                                    resource_calculate_lod_unclamped)
211:   GENERATE_HLSL_INTRINSIC_FUNCTION(DdxCoarse, ddx_coarse)
212:   GENERATE_HLSL_INTRINSIC_FUNCTION(DdyCoarse, ddy_coarse)
213:   GENERATE_HLSL_INTRINSIC_FUNCTION(DdxFine, ddx_fine)
214:   GENERATE_HLSL_INTRINSIC_FUNCTION(DdyFine, ddy_fine)
215: 
216:   //===----------------------------------------------------------------------===//
217:   // End of reserved area for HLSL intrinsic getters.
218:   //===----------------------------------------------------------------------===//
219: 
220: protected:
221:   CodeGenModule &CGM;
222: 
223:   llvm::Value *emitSystemSemanticLoad(llvm::IRBuilder<> &B,
224:                                       const FunctionDecl *FD, llvm::Type *Type,
```
- **EN**: This block documents intent or context for the surrounding LLVM IR emission code.
- **CN**: 该代码块说明周围 LLVM IR 生成 代码的意图或上下文。

### Lines 225-240
```cpp
225:                                       const clang::DeclaratorDecl *Decl,
226:                                       HLSLAppliedSemanticAttr *Semantic,
227:                                       std::optional<unsigned> Index);
228: 
229:   void emitSystemSemanticStore(llvm::IRBuilder<> &B, llvm::Value *Source,
230:                                const clang::DeclaratorDecl *Decl,
231:                                HLSLAppliedSemanticAttr *Semantic,
232:                                std::optional<unsigned> Index);
233: 
234:   llvm::Value *handleScalarSemanticLoad(llvm::IRBuilder<> &B,
235:                                         const FunctionDecl *FD,
236:                                         llvm::Type *Type,
237:                                         const clang::DeclaratorDecl *Decl,
238:                                         HLSLAppliedSemanticAttr *Semantic);
239: 
240:   void handleScalarSemanticStore(llvm::IRBuilder<> &B, const FunctionDecl *FD,
```
- **EN**: This block spells out callable entry points like `emitSystemSemanticStore`.
- **CN**: 该代码块给出可调用入口的声明，例如 `emitSystemSemanticStore`。

### Lines 241-256
```cpp
241:                                  llvm::Value *Source,
242:                                  const clang::DeclaratorDecl *Decl,
243:                                  HLSLAppliedSemanticAttr *Semantic);
244: 
245:   std::pair<llvm::Value *, specific_attr_iterator<HLSLAppliedSemanticAttr>>
246:   handleStructSemanticLoad(
247:       llvm::IRBuilder<> &B, const FunctionDecl *FD, llvm::Type *Type,
248:       const clang::DeclaratorDecl *Decl,
249:       specific_attr_iterator<HLSLAppliedSemanticAttr> begin,
250:       specific_attr_iterator<HLSLAppliedSemanticAttr> end);
251: 
252:   specific_attr_iterator<HLSLAppliedSemanticAttr> handleStructSemanticStore(
253:       llvm::IRBuilder<> &B, const FunctionDecl *FD, llvm::Value *Source,
254:       const clang::DeclaratorDecl *Decl,
255:       specific_attr_iterator<HLSLAppliedSemanticAttr> AttrBegin,
256:       specific_attr_iterator<HLSLAppliedSemanticAttr> AttrEnd);
```
- **EN**: This block spells out callable entry points like `handleStructSemanticLoad`, `handleStructSemanticStore`.
- **CN**: 该代码块给出可调用入口的声明，例如 `handleStructSemanticLoad`, `handleStructSemanticStore`。

### Lines 257-272
```cpp
257: 
258:   std::pair<llvm::Value *, specific_attr_iterator<HLSLAppliedSemanticAttr>>
259:   handleSemanticLoad(llvm::IRBuilder<> &B, const FunctionDecl *FD,
260:                      llvm::Type *Type, const clang::DeclaratorDecl *Decl,
261:                      specific_attr_iterator<HLSLAppliedSemanticAttr> begin,
262:                      specific_attr_iterator<HLSLAppliedSemanticAttr> end);
263: 
264:   specific_attr_iterator<HLSLAppliedSemanticAttr>
265:   handleSemanticStore(llvm::IRBuilder<> &B, const FunctionDecl *FD,
266:                       llvm::Value *Source, const clang::DeclaratorDecl *Decl,
267:                       specific_attr_iterator<HLSLAppliedSemanticAttr> AttrBegin,
268:                       specific_attr_iterator<HLSLAppliedSemanticAttr> AttrEnd);
269: 
270: public:
271:   CGHLSLRuntime(CodeGenModule &CGM) : CGM(CGM) {}
272:   virtual ~CGHLSLRuntime() {}
```
- **EN**: This block defines callable entry points like `handleSemanticLoad`, `handleSemanticStore`, `CGHLSLRuntime`, `~CGHLSLRuntime`.
- **CN**: 该代码块定义可调用入口，例如 `handleSemanticLoad`, `handleSemanticStore`, `CGHLSLRuntime`, `~CGHLSLRuntime`。

### Lines 273-288
```cpp
273: 
274:   llvm::Type *convertHLSLSpecificType(const Type *T,
275:                                       const CGHLSLOffsetInfo &OffsetInfo);
276:   llvm::Type *convertHLSLSpecificType(const Type *T) {
277:     return convertHLSLSpecificType(T, CGHLSLOffsetInfo());
278:   }
279: 
280:   void generateGlobalCtorDtorCalls();
281: 
282:   void addBuffer(const HLSLBufferDecl *D);
283:   void addRootSignature(const HLSLRootSignatureDecl *D);
284:   void finishCodeGen();
285: 
286:   void setHLSLEntryAttributes(const FunctionDecl *FD, llvm::Function *Fn);
287: 
288:   void emitEntryFunction(const FunctionDecl *FD, llvm::Function *Fn);
```
- **EN**: This block defines callable entry points like `convertHLSLSpecificType`, `generateGlobalCtorDtorCalls`, `addBuffer`, `addRootSignature`, `finishCodeGen`; returns or forwards computed values for the surrounding LLVM IR emission logic.
- **CN**: 该代码块定义可调用入口，例如 `convertHLSLSpecificType`, `generateGlobalCtorDtorCalls`, `addBuffer`, `addRootSignature`, `finishCodeGen`；为周围的 LLVM IR 生成 逻辑返回或转发计算结果。

### Lines 289-304
```cpp
289:   void setHLSLFunctionAttributes(const FunctionDecl *FD, llvm::Function *Fn);
290:   void handleGlobalVarDefinition(const VarDecl *VD, llvm::GlobalVariable *Var);
291: 
292:   llvm::Instruction *getConvergenceToken(llvm::BasicBlock &BB);
293: 
294:   llvm::StructType *getHLSLBufferLayoutType(const RecordType *LayoutStructTy);
295:   void addHLSLBufferLayoutType(const RecordType *LayoutStructTy,
296:                                llvm::StructType *LayoutTy);
297:   void emitInitListOpaqueValues(CodeGenFunction &CGF, InitListExpr *E);
298: 
299:   std::optional<LValue>
300:   emitResourceArraySubscriptExpr(const ArraySubscriptExpr *E,
301:                                  CodeGenFunction &CGF);
302:   bool emitResourceArrayCopy(LValue &LHS, Expr *RHSExpr, CodeGenFunction &CGF);
303: 
304:   std::optional<LValue> emitBufferArraySubscriptExpr(
```
- **EN**: This block spells out callable entry points like `setHLSLFunctionAttributes`, `handleGlobalVarDefinition`, `addHLSLBufferLayoutType`, `emitInitListOpaqueValues`, `emitResourceArraySubscriptExpr`.
- **CN**: 该代码块给出可调用入口的声明，例如 `setHLSLFunctionAttributes`, `handleGlobalVarDefinition`, `addHLSLBufferLayoutType`, `emitInitListOpaqueValues`, `emitResourceArraySubscriptExpr`。

### Lines 305-320
```cpp
305:       const ArraySubscriptExpr *E, CodeGenFunction &CGF,
306:       llvm::function_ref<llvm::Value *(bool Promote)> EmitIdxAfterBase);
307: 
308:   RawAddress createBufferMatrixTempAddress(const LValue &LV, SourceLocation Loc,
309:                                            CodeGenFunction &CGF);
310: 
311:   bool emitBufferCopy(CodeGenFunction &CGF, Address DestPtr, Address SrcPtr,
312:                       QualType CType);
313: 
314:   LValue emitBufferMemberExpr(CodeGenFunction &CGF, const MemberExpr *E);
315:   std::optional<LValue> emitResourceMemberExpr(CodeGenFunction &CGF,
316:                                                const MemberExpr *E);
317: 
318: private:
319:   void emitBufferGlobalsAndMetadata(const HLSLBufferDecl *BufDecl,
320:                                     llvm::GlobalVariable *BufGV,
```
- **EN**: This block spells out callable entry points like `createBufferMatrixTempAddress`, `emitBufferCopy`, `emitBufferMemberExpr`, `emitResourceMemberExpr`.
- **CN**: 该代码块给出可调用入口的声明，例如 `createBufferMatrixTempAddress`, `emitBufferCopy`, `emitBufferMemberExpr`, `emitResourceMemberExpr`。

### Lines 321-336
```cpp
321:                                     const CGHLSLOffsetInfo &OffsetInfo);
322:   void initializeBufferFromBinding(const HLSLBufferDecl *BufDecl,
323:                                    llvm::GlobalVariable *GV);
324:   void initializeBufferFromBinding(const HLSLBufferDecl *BufDecl,
325:                                    llvm::GlobalVariable *GV,
326:                                    HLSLResourceBindingAttr *RBA);
327: 
328:   llvm::Value *emitSPIRVUserSemanticLoad(llvm::IRBuilder<> &B, llvm::Type *Type,
329:                                          const clang::DeclaratorDecl *Decl,
330:                                          HLSLAppliedSemanticAttr *Semantic,
331:                                          std::optional<unsigned> Index);
332:   llvm::Value *emitDXILUserSemanticLoad(llvm::IRBuilder<> &B, llvm::Type *Type,
333:                                         HLSLAppliedSemanticAttr *Semantic,
334:                                         std::optional<unsigned> Index);
335:   llvm::Value *emitUserSemanticLoad(llvm::IRBuilder<> &B, llvm::Type *Type,
336:                                     const clang::DeclaratorDecl *Decl,
```
- **EN**: This block spells out callable entry points like `initializeBufferFromBinding`.
- **CN**: 该代码块给出可调用入口的声明，例如 `initializeBufferFromBinding`。

### Lines 337-352
```cpp
337:                                     HLSLAppliedSemanticAttr *Semantic,
338:                                     std::optional<unsigned> Index);
339: 
340:   void emitSPIRVUserSemanticStore(llvm::IRBuilder<> &B, llvm::Value *Source,
341:                                   const clang::DeclaratorDecl *Decl,
342:                                   HLSLAppliedSemanticAttr *Semantic,
343:                                   std::optional<unsigned> Index);
344:   void emitDXILUserSemanticStore(llvm::IRBuilder<> &B, llvm::Value *Source,
345:                                  HLSLAppliedSemanticAttr *Semantic,
346:                                  std::optional<unsigned> Index);
347:   void emitUserSemanticStore(llvm::IRBuilder<> &B, llvm::Value *Source,
348:                              const clang::DeclaratorDecl *Decl,
349:                              HLSLAppliedSemanticAttr *Semantic,
350:                              std::optional<unsigned> Index);
351: 
352:   llvm::Triple::ArchType getArch();
```
- **EN**: This block spells out callable entry points like `emitSPIRVUserSemanticStore`, `emitDXILUserSemanticStore`, `emitUserSemanticStore`, `getArch`.
- **CN**: 该代码块给出可调用入口的声明，例如 `emitSPIRVUserSemanticStore`, `emitDXILUserSemanticStore`, `emitUserSemanticStore`, `getArch`。

### Lines 353-362
```cpp
353: 
354:   llvm::DenseMap<const clang::RecordType *, llvm::StructType *> LayoutTypes;
355:   unsigned SPIRVLastAssignedInputSemanticLocation = 0;
356:   unsigned SPIRVLastAssignedOutputSemanticLocation = 0;
357: };
358: 
359: } // namespace CodeGen
360: } // namespace clang
361: 
362: #endif
```
- **EN**: This block opens or references namespaces `CodeGen`, `clang`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块打开或引用命名空间 `CodeGen`, `clang`；包含影响本编译单元构建方式的预处理结构。

## Key Concepts / 关键概念

- **GENERATE_HLSL_INTRINSIC_FUNCTION**: Central symbol in this file's implementation of LLVM IR emission. / 是该文件实现 LLVM IR 生成 时的核心符号。
- **HLSLAppliedSemanticAttr**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **IRBuilder**: Acts as a construction helper that incrementally assembles LLVM IR emission state. / 充当构建辅助器，逐步组装 LLVM IR 生成 状态。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Decl**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **DeclaratorDecl**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **FunctionDecl**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `Address.h`
- **Clang libraries / Clang 库**: `clang/AST/Attr.h`, `clang/AST/Decl.h`, `clang/Basic/Builtins.h`, `clang/Basic/HLSLRuntime.h`
- **LLVM libraries / LLVM 库**: `llvm/ADT/DenseMap.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/Frontend/HLSL/HLSLResource.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/IntrinsicsDirectX.h`, `llvm/IR/IntrinsicsSPIRV.h`
- **Other headers / 其他头文件**: `optional`, `vector`
