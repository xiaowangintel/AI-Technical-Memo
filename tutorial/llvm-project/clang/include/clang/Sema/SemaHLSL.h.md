# SemaHLSL.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Sema/SemaHLSL.h`
- Repository: `llvm-project`
- Purpose (EN): This file declares semantic analysis for HLSL constructs.
- 用途（中文）: 该文件为 Sema 子系统中的 Sema HLSL 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
 1: //===----- SemaHLSL.h ----- Semantic Analysis for HLSL constructs ---------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: /// \file
 9: /// This file declares semantic analysis for HLSL constructs.
10: ///
11: //===----------------------------------------------------------------------===//
12: 
13: #ifndef LLVM_CLANG_SEMA_SEMAHLSL_H
14: #define LLVM_CLANG_SEMA_SEMAHLSL_H
15: 
16: #include "clang/AST/ASTFwd.h"
17: #include "clang/AST/Attr.h"
18: #include "clang/AST/Type.h"
19: #include "clang/AST/TypeLoc.h"
20: #include "clang/Basic/DiagnosticSema.h"
21: #include "clang/Basic/SourceLocation.h"
22: #include "clang/Sema/SemaBase.h"
23: #include "llvm/ADT/DenseMap.h"
24: #include "llvm/ADT/SmallVector.h"
25: #include "llvm/ADT/StringSet.h"
26: #include "llvm/TargetParser/Triple.h"
27: #include <initializer_list>
28: 
29: namespace clang {
30: class AttributeCommonInfo;
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/AST/ASTFwd.h`, `clang/AST/Attr.h`, `clang/AST/Type.h` and 9 more. It opens, closes, or documents namespace scope for `clang`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/AST/ASTFwd.h`, `clang/AST/Attr.h`, `clang/AST/Type.h` 以及另外 9 项依赖。 它打开、关闭或说明了 `clang` 的命名空间作用域。

### Lines 31-60

```cpp
31: class IdentifierInfo;
32: class InitializedEntity;
33: class InitializationKind;
34: class ParsedAttr;
35: class Scope;
36: class VarDecl;
37: 
38: namespace hlsl {
39: 
40: // Introduce a wrapper struct around the underlying RootElement. This structure
41: // will retain extra clang diagnostic information that is not available in llvm.
42: struct RootSignatureElement {
43:   RootSignatureElement(SourceLocation Loc,
44:                        llvm::hlsl::rootsig::RootElement Element)
45:       : Loc(Loc), Element(Element) {}
46: 
47:   const llvm::hlsl::rootsig::RootElement &getElement() const { return Element; }
48:   const SourceLocation &getLocation() const { return Loc; }
49: 
50: private:
51:   SourceLocation Loc;
52:   llvm::hlsl::rootsig::RootElement Element;
53: };
54: 
55: } // namespace hlsl
56: 
57: using llvm::dxil::ResourceClass;
58: 
59: // FIXME: This can be hidden (as static function in SemaHLSL.cpp) once we no
60: // longer need to create builtin buffer types in HLSLExternalSemaSource.
```
- EN: It opens, closes, or documents namespace scope for `hlsl`. Key type declarations here include `IdentifierInfo`, `InitializedEntity`, `InitializationKind`, `ParsedAttr`. It exposes API surface such as `Loc`, `getElement`, `getLocation`.
- 中文: 它打开、关闭或说明了 `hlsl` 的命名空间作用域。 这里的重要类型声明包括 `IdentifierInfo`, `InitializedEntity`, `InitializationKind`, `ParsedAttr`。 它暴露了 `Loc`, `getElement`, `getLocation` 等接口。

### Lines 61-90

```cpp
61: bool CreateHLSLAttributedResourceType(
62:     Sema &S, QualType Wrapped, ArrayRef<const Attr *> AttrList,
63:     QualType &ResType, HLSLAttributedResourceLocInfo *LocInfo = nullptr);
64: 
65: enum class BindingType : uint8_t { NotAssigned, Explicit, Implicit };
66: 
67: // DeclBindingInfo struct stores information about required/assigned resource
68: // binding onon a declaration for specific resource class.
69: struct DeclBindingInfo {
70:   const VarDecl *Decl;
71:   ResourceClass ResClass;
72:   const HLSLResourceBindingAttr *Attr;
73:   BindingType BindType;
74: 
75:   DeclBindingInfo(const VarDecl *Decl, ResourceClass ResClass,
76:                   BindingType BindType = BindingType::NotAssigned,
77:                   const HLSLResourceBindingAttr *Attr = nullptr)
78:       : Decl(Decl), ResClass(ResClass), Attr(Attr), BindType(BindType) {}
79: 
80:   void setBindingAttribute(HLSLResourceBindingAttr *A, BindingType BT) {
81:     assert(Attr == nullptr && BindType == BindingType::NotAssigned &&
82:            "binding attribute already assigned");
83:     Attr = A;
84:     BindType = BT;
85:   }
86: };
87: 
88: // ResourceBindings class stores information about all resource bindings
89: // in a shader. It is used for binding diagnostics and implicit binding
90: // assignments.
```
- EN: Key type declarations here include `BindingType`, `DeclBindingInfo`. It introduces enum-based state or option sets such as `BindingType`. It exposes API surface such as `Decl`, `setBindingAttribute`.
- 中文: 这里的重要类型声明包括 `BindingType`, `DeclBindingInfo`。 它引入了 `BindingType` 等基于枚举的状态或选项集合。 它暴露了 `Decl`, `setBindingAttribute` 等接口。

### Lines 91-120

```cpp
 91: class ResourceBindings {
 92: public:
 93:   DeclBindingInfo *addDeclBindingInfo(const VarDecl *VD,
 94:                                       ResourceClass ResClass);
 95:   DeclBindingInfo *getDeclBindingInfo(const VarDecl *VD,
 96:                                       ResourceClass ResClass);
 97:   bool hasBindingInfoForDecl(const VarDecl *VD) const;
 98: 
 99: private:
100:   // List of all resource bindings required by the shader.
101:   // A global declaration can have multiple bindings for different
102:   // resource classes. They are all stored sequentially in this list.
103:   // The DeclToBindingListIndex hashtable maps a declaration to the
104:   // index of the first binding info in the list.
105:   llvm::SmallVector<DeclBindingInfo> BindingsList;
106:   llvm::DenseMap<const VarDecl *, unsigned> DeclToBindingListIndex;
107: };
108: 
109: class SemaHLSL : public SemaBase {
110: public:
111:   SemaHLSL(Sema &S);
112: 
113:   Decl *ActOnStartBuffer(Scope *BufferScope, bool CBuffer, SourceLocation KwLoc,
114:                          IdentifierInfo *Ident, SourceLocation IdentLoc,
115:                          SourceLocation LBrace);
116:   void ActOnFinishBuffer(Decl *Dcl, SourceLocation RBrace);
117:   HLSLNumThreadsAttr *mergeNumThreadsAttr(Decl *D,
118:                                           const AttributeCommonInfo &AL, int X,
119:                                           int Y, int Z);
120:   HLSLWaveSizeAttr *mergeWaveSizeAttr(Decl *D, const AttributeCommonInfo &AL,
```
- EN: Key type declarations here include `ResourceBindings`, `SemaHLSL`. It exposes API surface such as `hasBindingInfoForDecl`, `SemaHLSL`, `ActOnFinishBuffer`.
- 中文: 这里的重要类型声明包括 `ResourceBindings`, `SemaHLSL`。 它暴露了 `hasBindingInfoForDecl`, `SemaHLSL`, `ActOnFinishBuffer` 等接口。

### Lines 121-150

```cpp
121:                                       int Min, int Max, int Preferred,
122:                                       int SpelledArgsCount);
123:   HLSLVkConstantIdAttr *
124:   mergeVkConstantIdAttr(Decl *D, const AttributeCommonInfo &AL, int Id);
125:   HLSLShaderAttr *mergeShaderAttr(Decl *D, const AttributeCommonInfo &AL,
126:                                   llvm::Triple::EnvironmentType ShaderType);
127:   HLSLParamModifierAttr *
128:   mergeParamModifierAttr(Decl *D, const AttributeCommonInfo &AL,
129:                          HLSLParamModifierAttr::Spelling Spelling);
130:   void ActOnTopLevelFunction(FunctionDecl *FD);
131:   void ActOnVariableDeclarator(VarDecl *VD);
132:   bool ActOnUninitializedVarDecl(VarDecl *D);
133:   void ActOnEndOfTranslationUnit(TranslationUnitDecl *TU);
134:   bool ActOnResourceMemberAccessExpr(MemberExpr *ME);
135:   void CheckEntryPoint(FunctionDecl *FD);
136: 
137:   // Return true if everything is ok; returns false if there was an error.
138:   bool CheckResourceBinOp(BinaryOperatorKind Opc, Expr *LHSExpr, Expr *RHSExpr,
139:                           SourceLocation Loc);
140: 
141:   QualType handleVectorBinOpConversion(ExprResult &LHS, ExprResult &RHS,
142:                                        QualType LHSType, QualType RHSType,
143:                                        bool IsCompAssign);
144:   void emitLogicalOperatorFixIt(Expr *LHS, Expr *RHS, BinaryOperatorKind Opc);
145: 
146:   // Returns the result of converting ConstantBuffer<T> to
147:   // `const hlsl_constant T&`. If `BaseExpr`'s type is not ConstantBuffer<T>
148:   // then the return value is `std::nullopt`.
149:   std::optional<ExprResult>
150:   tryPerformConstantBufferConversion(ExprResult &BaseExpr);
```
- EN: It exposes API surface such as `mergeVkConstantIdAttr`, `ActOnTopLevelFunction`, `ActOnVariableDeclarator`, `ActOnUninitializedVarDecl`.
- 中文: 它暴露了 `mergeVkConstantIdAttr`, `ActOnTopLevelFunction`, `ActOnVariableDeclarator`, `ActOnUninitializedVarDecl` 等接口。

### Lines 151-180

```cpp
151: 
152:   // Returns the conversion operator to convert `RD` to `const hlsl_constant
153:   // Type&`. Returns `nullptr` if it could not be found.
154:   NamedDecl *getConstantBufferConversionFunction(QualType Type,
155:                                                  CXXRecordDecl *RD);
156: 
157:   /// Computes the unique Root Signature identifier from the given signature,
158:   /// then lookup if there is a previousy created Root Signature decl.
159:   ///
160:   /// Returns the identifier and if it was found
161:   std::pair<IdentifierInfo *, bool>
162:   ActOnStartRootSignatureDecl(StringRef Signature);
163: 
164:   /// Creates the Root Signature decl of the parsed Root Signature elements
165:   /// onto the AST and push it onto current Scope
166:   void
167:   ActOnFinishRootSignatureDecl(SourceLocation Loc, IdentifierInfo *DeclIdent,
168:                                ArrayRef<hlsl::RootSignatureElement> Elements);
169: 
170:   void SetRootSignatureOverride(IdentifierInfo *DeclIdent) {
171:     RootSigOverrideIdent = DeclIdent;
172:   }
173: 
174:   HLSLRootSignatureDecl *lookupRootSignatureOverrideDecl(DeclContext *DC) const;
175: 
176:   // Returns true if any RootSignatureElement is invalid and a diagnostic was
177:   // produced
178:   bool
179:   handleRootSignatureElements(ArrayRef<hlsl::RootSignatureElement> Elements);
180:   void handleRootSignatureAttr(Decl *D, const ParsedAttr &AL);
```
- EN: It exposes API surface such as `ActOnStartRootSignatureDecl`, `SetRootSignatureOverride`, `lookupRootSignatureOverrideDecl`, `handleRootSignatureElements`.
- 中文: 它暴露了 `ActOnStartRootSignatureDecl`, `SetRootSignatureOverride`, `lookupRootSignatureOverrideDecl`, `handleRootSignatureElements` 等接口。

### Lines 181-210

```cpp
181:   void handleNumThreadsAttr(Decl *D, const ParsedAttr &AL);
182:   void handleWaveSizeAttr(Decl *D, const ParsedAttr &AL);
183:   void handleVkConstantIdAttr(Decl *D, const ParsedAttr &AL);
184:   void handleVkBindingAttr(Decl *D, const ParsedAttr &AL);
185:   void handleVkLocationAttr(Decl *D, const ParsedAttr &AL);
186:   void handlePackOffsetAttr(Decl *D, const ParsedAttr &AL);
187:   void handleShaderAttr(Decl *D, const ParsedAttr &AL);
188:   void handleResourceBindingAttr(Decl *D, const ParsedAttr &AL);
189:   void handleParamModifierAttr(Decl *D, const ParsedAttr &AL);
190:   void handleMatrixLayoutAttr(Decl *D, const ParsedAttr &AL);
191:   bool diagnoseInstantiatedMatrixLayoutAttr(Decl *D,
192:                                             const HLSLMatrixLayoutAttr *Attr);
193:   bool handleResourceTypeAttr(QualType T, const ParsedAttr &AL);
194: 
195:   template <typename T>
196:   T *createSemanticAttr(const AttributeCommonInfo &ACI,
197:                         std::optional<unsigned> Location) {
198:     return ::new (getASTContext())
199:         T(getASTContext(), ACI, ACI.getAttrName()->getName(),
200:           Location.value_or(0));
201:   }
202: 
203:   void diagnoseSystemSemanticAttr(Decl *D, const ParsedAttr &AL,
204:                                   std::optional<unsigned> Index);
205:   void handleSemanticAttr(Decl *D, const ParsedAttr &AL);
206: 
207:   void handleVkExtBuiltinInputAttr(Decl *D, const ParsedAttr &AL);
208:   void handleVkExtBuiltinOutputAttr(Decl *D, const ParsedAttr &AL);
209:   void handleVkPushConstantAttr(Decl *D, const ParsedAttr &AL);
210: 
```
- EN: It exposes API surface such as `handleNumThreadsAttr`, `handleWaveSizeAttr`, `handleVkConstantIdAttr`, `handleVkBindingAttr`.
- 中文: 它暴露了 `handleNumThreadsAttr`, `handleWaveSizeAttr`, `handleVkConstantIdAttr`, `handleVkBindingAttr` 等接口。

### Lines 211-240

```cpp
211:   bool CheckBuiltinFunctionCall(unsigned BuiltinID, CallExpr *TheCall);
212:   QualType ProcessResourceTypeAttributes(QualType Wrapped);
213:   HLSLAttributedResourceLocInfo
214:   TakeLocForHLSLAttribute(const HLSLAttributedResourceType *RT);
215: 
216:   // HLSL Type trait implementations
217:   bool IsScalarizedLayoutCompatible(QualType T1, QualType T2) const;
218:   bool IsTypedResourceElementCompatible(QualType T1);
219:   bool IsConstantBufferElementCompatible(QualType T1);
220: 
221:   bool CheckCompatibleParameterABI(FunctionDecl *New, FunctionDecl *Old);
222: 
223:   QualType ActOnTemplateShorthand(TemplateDecl *Template,
224:                                   SourceLocation NameLoc);
225: 
226:   // Diagnose whether the input ID is uint/unit2/uint3 type.
227:   bool diagnoseInputIDType(QualType T, const ParsedAttr &AL);
228:   bool diagnosePositionType(QualType T, const ParsedAttr &AL);
229: 
230:   bool CanPerformScalarCast(QualType SrcTy, QualType DestTy);
231:   bool CanPerformElementwiseCast(Expr *Src, QualType DestType);
232:   bool CanPerformAggregateSplatCast(Expr *Src, QualType DestType);
233:   ExprResult ActOnOutParamExpr(ParmVarDecl *Param, Expr *Arg);
234: 
235:   QualType getInoutParameterType(QualType Ty);
236: 
237:   bool transformInitList(const InitializedEntity &Entity, InitListExpr *Init);
238:   bool handleInitialization(VarDecl *VDecl, Expr *&Init);
239:   void deduceAddressSpace(VarDecl *Decl);
240:   QualType checkMatrixComponent(Sema &S, QualType baseType, ExprValueKind &VK,
```
- EN: It exposes API surface such as `CheckBuiltinFunctionCall`, `ProcessResourceTypeAttributes`, `TakeLocForHLSLAttribute`, `IsScalarizedLayoutCompatible`.
- 中文: 它暴露了 `CheckBuiltinFunctionCall`, `ProcessResourceTypeAttributes`, `TakeLocForHLSLAttribute`, `IsScalarizedLayoutCompatible` 等接口。

### Lines 241-270

```cpp
241:                                 SourceLocation OpLoc,
242:                                 const IdentifierInfo *CompName,
243:                                 SourceLocation CompLoc);
244: 
245:   uint32_t getNextImplicitBindingOrderID() {
246:     return ImplicitBindingNextOrderID++;
247:   }
248: 
249:   bool initGlobalResourceDecl(VarDecl *VD);
250:   bool initGlobalResourceArrayDecl(VarDecl *VD);
251: 
252: private:
253:   // HLSL resource type attributes need to be processed all at once.
254:   // This is a list to collect them.
255:   llvm::SmallVector<const Attr *> HLSLResourcesTypeAttrs;
256: 
257:   /// TypeLoc data for HLSLAttributedResourceType instances that we
258:   /// have not yet populated.
259:   llvm::DenseMap<const HLSLAttributedResourceType *,
260:                  HLSLAttributedResourceLocInfo>
261:       LocsForHLSLAttributedResources;
262: 
263:   // List of all resource bindings
264:   ResourceBindings Bindings;
265: 
266:   // Map of local resource variables to their assigned global resources.
267:   //
268:   // The binding can be a nullptr, in which case, the variable has yet to be
269:   // initialized or assigned to.
270:   llvm::DenseMap<const VarDecl *, const DeclBindingInfo *> Assigns;
```
- EN: It exposes API surface such as `getNextImplicitBindingOrderID`, `initGlobalResourceDecl`, `initGlobalResourceArrayDecl`.
- 中文: 它暴露了 `getNextImplicitBindingOrderID`, `initGlobalResourceDecl`, `initGlobalResourceArrayDecl` 等接口。

### Lines 271-300

```cpp
271: 
272:   // Global declaration collected for the $Globals default constant
273:   // buffer which will be created at the end of the translation unit.
274:   llvm::SmallVector<Decl *> DefaultCBufferDecls;
275: 
276:   uint32_t ImplicitBindingNextOrderID = 0;
277: 
278:   IdentifierInfo *RootSigOverrideIdent = nullptr;
279: 
280:   bool HasDeclaredAPushConstant = false;
281: 
282:   // Information about the current subtree being flattened.
283:   struct SemanticInfo {
284:     HLSLParsedSemanticAttr *Semantic;
285:     std::optional<uint32_t> Index = std::nullopt;
286:   };
287: 
288:   // Bitmask used to recall if the current semantic subtree is
289:   // input, output or inout.
290:   enum IOType {
291:     In = 0b01,
292:     Out = 0b10,
293:     InOut = 0b11,
294:   };
295: 
296:   // The context shared by all semantics with the same IOType during
297:   // flattening.
298:   struct SemanticContext {
299:     // Present if any semantic sharing the same IO type has an explicit or
300:     // implicit SPIR-V location index assigned.
```
- EN: Key type declarations here include `SemanticInfo`, `SemanticContext`. It introduces enum-based state or option sets such as `IOType`.
- 中文: 这里的重要类型声明包括 `SemanticInfo`, `SemanticContext`。 它引入了 `IOType` 等基于枚举的状态或选项集合。

### Lines 301-330

```cpp
301:     std::optional<bool> UsesExplicitVkLocations = std::nullopt;
302:     // The set of semantics found to be active during flattening. Used to detect
303:     // index collisions.
304:     llvm::StringSet<> ActiveSemantics = {};
305:     // The IOType of this semantic set.
306:     IOType CurrentIOType;
307:   };
308: 
309:   struct SemanticStageInfo {
310:     llvm::Triple::EnvironmentType Stage;
311:     IOType AllowedIOTypesMask;
312:   };
313: 
314: private:
315:   void collectResourceBindingsOnVarDecl(VarDecl *D);
316:   void collectResourceBindingsOnUserRecordDecl(const VarDecl *VD,
317:                                                const RecordType *RT);
318: 
319:   void checkSemanticAnnotation(FunctionDecl *EntryPoint, const Decl *Param,
320:                                const HLSLAppliedSemanticAttr *SemanticAttr,
321:                                const SemanticContext &SC);
322: 
323:   bool determineActiveSemanticOnScalar(FunctionDecl *FD,
324:                                        DeclaratorDecl *OutputDecl,
325:                                        DeclaratorDecl *D,
326:                                        SemanticInfo &ActiveSemantic,
327:                                        SemanticContext &SC);
328: 
329:   bool determineActiveSemantic(FunctionDecl *FD, DeclaratorDecl *OutputDecl,
330:                                DeclaratorDecl *D, SemanticInfo &ActiveSemantic,
```
- EN: Key type declarations here include `SemanticStageInfo`. It exposes API surface such as `collectResourceBindingsOnVarDecl`.
- 中文: 这里的重要类型声明包括 `SemanticStageInfo`。 它暴露了 `collectResourceBindingsOnVarDecl` 等接口。

### Lines 331-359

```cpp
331:                                SemanticContext &SC);
332: 
333:   void processExplicitBindingsOnDecl(VarDecl *D);
334: 
335:   void diagnoseAvailabilityViolations(TranslationUnitDecl *TU);
336: 
337:   void diagnoseAttrStageMismatch(
338:       const Attr *A, llvm::Triple::EnvironmentType Stage,
339:       std::initializer_list<llvm::Triple::EnvironmentType> AllowedStages);
340: 
341:   void diagnoseSemanticStageMismatch(
342:       const Attr *A, llvm::Triple::EnvironmentType Stage, IOType CurrentIOType,
343:       std::initializer_list<SemanticStageInfo> AllowedStages);
344: 
345:   void handleGlobalStructOrArrayOfWithResources(VarDecl *VD);
346: 
347:   // Infer a common global binding info for an Expr
348:   //
349:   // Returns std::nullopt if the expr refers to non-unique global bindings.
350:   // Returns nullptr if it refer to any global binding, otherwise it returns
351:   // a reference to the global binding info.
352:   std::optional<const DeclBindingInfo *> inferGlobalBinding(Expr *E);
353: 
354:   void trackLocalResource(VarDecl *VDecl, Expr *E);
355: };
356: 
357: } // namespace clang
358: 
359: #endif // LLVM_CLANG_SEMA_SEMAHLSL_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `clang`. It exposes API surface such as `processExplicitBindingsOnDecl`, `diagnoseAvailabilityViolations`, `handleGlobalStructOrArrayOfWithResources`, `inferGlobalBinding`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `clang` 的命名空间作用域。 它暴露了 `processExplicitBindingsOnDecl`, `diagnoseAvailabilityViolations`, `handleGlobalStructOrArrayOfWithResources`, `inferGlobalBinding` 等接口。

## Key Concepts / 关键概念

- `AttributeCommonInfo`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `IdentifierInfo`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `InitializedEntity`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `InitializationKind`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ParsedAttr`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Scope`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `VarDecl`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `RootSignatureElement`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/AST/ASTFwd.h`, `clang/AST/Attr.h`, `clang/AST/Type.h`, `clang/AST/TypeLoc.h`, `clang/Basic/DiagnosticSema.h`, `clang/Basic/SourceLocation.h`, `clang/Sema/SemaBase.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringSet.h`, `llvm/TargetParser/Triple.h`, `initializer_list`
- Forward declarations / 前向声明: `AttributeCommonInfo`, `IdentifierInfo`, `InitializedEntity`, `InitializationKind`, `ParsedAttr`, `Scope`, `VarDecl`
- Namespace context / 命名空间上下文: `clang`, `hlsl`
- Macro-style dependencies / 宏式依赖: None / 无
