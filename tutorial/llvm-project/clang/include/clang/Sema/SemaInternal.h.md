# SemaInternal.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Sema/SemaInternal.h`
- Repository: `llvm-project`
- Purpose (EN): Internal Sema Interfaces.
- 用途（中文）: 该文件为 Sema 子系统中的 Sema Internal 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

```cpp
 1: //===--- SemaInternal.h - Internal Sema Interfaces --------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file provides common API and #includes for the internal
10: // implementation of Sema.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #ifndef LLVM_CLANG_SEMA_SEMAINTERNAL_H
15: #define LLVM_CLANG_SEMA_SEMAINTERNAL_H
16: 
17: #include "clang/AST/ASTContext.h"
18: #include "clang/AST/DeclTemplate.h"
19: #include "clang/Sema/Lookup.h"
20: #include "clang/Sema/Sema.h"
21: #include "clang/Sema/SemaDiagnostic.h"
22: 
23: namespace clang {
24: 
25: inline bool
26: FTIHasSingleVoidParameter(const DeclaratorChunk::FunctionTypeInfo &FTI) {
27:   return FTI.NumParams == 1 && !FTI.isVariadic &&
28:          FTI.Params[0].Ident == nullptr && FTI.Params[0].Param &&
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/AST/ASTContext.h`, `clang/AST/DeclTemplate.h`, `clang/Sema/Lookup.h` and 2 more. It opens, closes, or documents namespace scope for `clang`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/AST/ASTContext.h`, `clang/AST/DeclTemplate.h`, `clang/Sema/Lookup.h` 以及另外 2 项依赖。 它打开、关闭或说明了 `clang` 的命名空间作用域。

### Lines 29-56

```cpp
29:          cast<ParmVarDecl>(FTI.Params[0].Param)->getType()->isVoidType();
30: }
31: 
32: inline bool
33: FTIHasNonVoidParameters(const DeclaratorChunk::FunctionTypeInfo &FTI) {
34:   // Assume FTI is well-formed.
35:   return FTI.NumParams && !FTIHasSingleVoidParameter(FTI);
36: }
37: 
38: // Helper function to check whether D's attributes match current CUDA mode.
39: // Decls with mismatched attributes and related diagnostics may have to be
40: // ignored during this CUDA compilation pass.
41: inline bool DeclAttrsMatchCUDAMode(const LangOptions &LangOpts, Decl *D) {
42:   if (!LangOpts.CUDA || !D)
43:     return true;
44:   bool isDeviceSideDecl = D->hasAttr<CUDADeviceAttr>() ||
45:                           D->hasAttr<CUDASharedAttr>() ||
46:                           D->hasAttr<CUDAGlobalAttr>();
47:   return isDeviceSideDecl == LangOpts.CUDAIsDevice;
48: }
49: 
50: /// Return a DLL attribute from the declaration.
51: inline InheritableAttr *getDLLAttr(Decl *D) {
52:   assert(!(D->hasAttr<DLLImportAttr>() && D->hasAttr<DLLExportAttr>()) &&
53:          "A declaration cannot be both dllimport and dllexport.");
54:   if (auto *Import = D->getAttr<DLLImportAttr>())
55:     return Import;
56:   if (auto *Export = D->getAttr<DLLExportAttr>())
```
- EN: It exposes API surface such as `cast`, `FTIHasNonVoidParameters`, `FTIHasSingleVoidParameter`, `DeclAttrsMatchCUDAMode`.
- 中文: 它暴露了 `cast`, `FTIHasNonVoidParameters`, `FTIHasSingleVoidParameter`, `DeclAttrsMatchCUDAMode` 等接口。

### Lines 57-84

```cpp
57:     return Export;
58:   return nullptr;
59: }
60: 
61: /// Retrieve the depth and index of a template parameter.
62: inline std::pair<unsigned, unsigned> getDepthAndIndex(const NamedDecl *ND) {
63:   if (const auto *TTP = dyn_cast<TemplateTypeParmDecl>(ND))
64:     return std::make_pair(TTP->getDepth(), TTP->getIndex());
65: 
66:   if (const auto *NTTP = dyn_cast<NonTypeTemplateParmDecl>(ND))
67:     return std::make_pair(NTTP->getDepth(), NTTP->getIndex());
68: 
69:   const auto *TTP = cast<TemplateTemplateParmDecl>(ND);
70:   return std::make_pair(TTP->getDepth(), TTP->getIndex());
71: }
72: 
73: /// Retrieve the depth and index of an unexpanded parameter pack.
74: /// Returns nullopt when the unexpanded packs do not correspond to template
75: /// parameters, e.g. __builtin_dedup_types.
76: inline std::optional<std::pair<unsigned, unsigned>>
77: getDepthAndIndex(UnexpandedParameterPack UPP) {
78:   if (const auto *TTP = dyn_cast<const TemplateTypeParmType *>(UPP.first))
79:     return std::make_pair(TTP->getDepth(), TTP->getIndex());
80:   if (isa<NamedDecl *>(UPP.first))
81:     return getDepthAndIndex(cast<NamedDecl *>(UPP.first));
82:   assert((isa<const TemplateSpecializationType *,
83:               const SubstBuiltinTemplatePackType *>(UPP.first)));
84:   return std::nullopt;
```
- EN: It exposes API surface such as `getDepthAndIndex`, `make_pair`, `cast`.
- 中文: 它暴露了 `getDepthAndIndex`, `make_pair`, `cast` 等接口。

### Lines 85-112

```cpp
 85: }
 86: 
 87: class TypoCorrectionConsumer : public VisibleDeclConsumer {
 88:   typedef SmallVector<TypoCorrection, 1> TypoResultList;
 89:   typedef llvm::StringMap<TypoResultList> TypoResultsMap;
 90:   typedef std::map<unsigned, TypoResultsMap> TypoEditDistanceMap;
 91: 
 92: public:
 93:   TypoCorrectionConsumer(Sema &SemaRef,
 94:                          const DeclarationNameInfo &TypoName,
 95:                          Sema::LookupNameKind LookupKind,
 96:                          Scope *S, CXXScopeSpec *SS,
 97:                          std::unique_ptr<CorrectionCandidateCallback> CCC,
 98:                          DeclContext *MemberContext,
 99:                          bool EnteringContext)
100:       : Typo(TypoName.getName().getAsIdentifierInfo()), CurrentTCIndex(0),
101:         SavedTCIndex(0), SemaRef(SemaRef), S(S),
102:         SS(SS ? std::make_unique<CXXScopeSpec>(*SS) : nullptr),
103:         CorrectionValidator(std::move(CCC)), MemberContext(MemberContext),
104:         Result(SemaRef, TypoName, LookupKind),
105:         Namespaces(SemaRef.Context, SemaRef.CurContext, SS),
106:         EnteringContext(EnteringContext), SearchNamespaces(false) {
107:     Result.suppressDiagnostics();
108:     // Arrange for ValidatedCorrections[0] to always be an empty correction.
109:     ValidatedCorrections.push_back(TypoCorrection());
110:   }
111: 
112:   bool includeHiddenDecls() const override { return true; }
```
- EN: Key type declarations here include `TypoCorrectionConsumer`. It exposes API surface such as `EnteringContext`, `suppressDiagnostics`, `push_back`.
- 中文: 这里的重要类型声明包括 `TypoCorrectionConsumer`。 它暴露了 `EnteringContext`, `suppressDiagnostics`, `push_back` 等接口。

### Lines 113-140

```cpp
113: 
114:   // Methods for adding potential corrections to the consumer.
115:   void FoundDecl(NamedDecl *ND, NamedDecl *Hiding, DeclContext *Ctx,
116:                  bool InBaseClass) override;
117:   void FoundName(StringRef Name);
118:   void addKeywordResult(StringRef Keyword);
119:   void addCorrection(TypoCorrection Correction);
120: 
121:   bool empty() const {
122:     return CorrectionResults.empty() && ValidatedCorrections.size() == 1;
123:   }
124: 
125:   /// Return the list of TypoCorrections for the given identifier from
126:   /// the set of corrections that have the closest edit distance, if any.
127:   TypoResultList &operator[](StringRef Name) {
128:     return CorrectionResults.begin()->second[Name];
129:   }
130: 
131:   /// Return the edit distance of the corrections that have the
132:   /// closest/best edit distance from the original typop.
133:   unsigned getBestEditDistance(bool Normalized) {
134:     if (CorrectionResults.empty())
135:       return (std::numeric_limits<unsigned>::max)();
136: 
137:     unsigned BestED = CorrectionResults.begin()->first;
138:     return Normalized ? TypoCorrection::NormalizeEditDistance(BestED) : BestED;
139:   }
140: 
```
- EN: It exposes API surface such as `FoundName`, `addKeywordResult`, `addCorrection`, `empty`.
- 中文: 它暴露了 `FoundName`, `addKeywordResult`, `addCorrection`, `empty` 等接口。

### Lines 141-168

```cpp
141:   /// Set-up method to add to the consumer the set of namespaces to use
142:   /// in performing corrections to nested name specifiers. This method also
143:   /// implicitly adds all of the known classes in the current AST context to the
144:   /// to the consumer for correcting nested name specifiers.
145:   void
146:   addNamespaces(const llvm::MapVector<NamespaceDecl *, bool> &KnownNamespaces);
147: 
148:   /// Return the next typo correction that passes all internal filters
149:   /// and is deemed valid by the consumer's CorrectionCandidateCallback,
150:   /// starting with the corrections that have the closest edit distance. An
151:   /// empty TypoCorrection is returned once no more viable corrections remain
152:   /// in the consumer.
153:   const TypoCorrection &getNextCorrection();
154: 
155:   /// Get the last correction returned by getNextCorrection().
156:   const TypoCorrection &getCurrentCorrection() {
157:     return CurrentTCIndex < ValidatedCorrections.size()
158:                ? ValidatedCorrections[CurrentTCIndex]
159:                : ValidatedCorrections[0];  // The empty correction.
160:   }
161: 
162:   /// Return the next typo correction like getNextCorrection, but keep
163:   /// the internal state pointed to the current correction (i.e. the next time
164:   /// getNextCorrection is called, it will return the same correction returned
165:   /// by peekNextcorrection).
166:   const TypoCorrection &peekNextCorrection() {
167:     auto Current = CurrentTCIndex;
168:     const TypoCorrection &TC = getNextCorrection();
```
- EN: It exposes API surface such as `addNamespaces`, `getNextCorrection`, `getCurrentCorrection`, `peekNextCorrection`.
- 中文: 它暴露了 `addNamespaces`, `getNextCorrection`, `getCurrentCorrection`, `peekNextCorrection` 等接口。

### Lines 169-196

```cpp
169:     CurrentTCIndex = Current;
170:     return TC;
171:   }
172: 
173:   /// In the case of deeply invalid expressions, `getNextCorrection()` will
174:   /// never be called since the transform never makes progress. If we don't
175:   /// detect this we risk trying to correct typos forever.
176:   bool hasMadeAnyCorrectionProgress() const { return CurrentTCIndex != 0; }
177: 
178:   /// Reset the consumer's position in the stream of viable corrections
179:   /// (i.e. getNextCorrection() will return each of the previously returned
180:   /// corrections in order before returning any new corrections).
181:   void resetCorrectionStream() {
182:     CurrentTCIndex = 0;
183:   }
184: 
185:   /// Return whether the end of the stream of corrections has been
186:   /// reached.
187:   bool finished() {
188:     return CorrectionResults.empty() &&
189:            CurrentTCIndex >= ValidatedCorrections.size();
190:   }
191: 
192:   /// Save the current position in the correction stream (overwriting any
193:   /// previously saved position).
194:   void saveCurrentPosition() {
195:     SavedTCIndex = CurrentTCIndex;
196:   }
```
- EN: It exposes API surface such as `hasMadeAnyCorrectionProgress`, `resetCorrectionStream`, `finished`, `size`.
- 中文: 它暴露了 `hasMadeAnyCorrectionProgress`, `resetCorrectionStream`, `finished`, `size` 等接口。

### Lines 197-224

```cpp
197: 
198:   /// Restore the saved position in the correction stream.
199:   void restoreSavedPosition() {
200:     CurrentTCIndex = SavedTCIndex;
201:   }
202: 
203:   ASTContext &getContext() const { return SemaRef.Context; }
204:   const LookupResult &getLookupResult() const { return Result; }
205: 
206:   bool isAddressOfOperand() const { return CorrectionValidator->IsAddressOfOperand; }
207:   const CXXScopeSpec *getSS() const { return SS.get(); }
208:   Scope *getScope() const { return S; }
209:   CorrectionCandidateCallback *getCorrectionValidator() const {
210:     return CorrectionValidator.get();
211:   }
212: 
213: private:
214:   class NamespaceSpecifierSet {
215:     struct SpecifierInfo {
216:       DeclContext* DeclCtx;
217:       NestedNameSpecifier NameSpecifier;
218:       unsigned EditDistance;
219:     };
220: 
221:     typedef SmallVector<DeclContext*, 4> DeclContextList;
222:     typedef SmallVector<SpecifierInfo, 16> SpecifierInfoList;
223: 
224:     ASTContext &Context;
```
- EN: Key type declarations here include `NamespaceSpecifierSet`, `SpecifierInfo`. It exposes API surface such as `restoreSavedPosition`, `getContext`, `getLookupResult`, `isAddressOfOperand`.
- 中文: 这里的重要类型声明包括 `NamespaceSpecifierSet`, `SpecifierInfo`。 它暴露了 `restoreSavedPosition`, `getContext`, `getLookupResult`, `isAddressOfOperand` 等接口。

### Lines 225-252

```cpp
225:     DeclContextList CurContextChain;
226:     std::string CurNameSpecifier;
227:     SmallVector<const IdentifierInfo*, 4> CurContextIdentifiers;
228:     SmallVector<const IdentifierInfo*, 4> CurNameSpecifierIdentifiers;
229: 
230:     std::map<unsigned, SpecifierInfoList> DistanceMap;
231: 
232:     /// Helper for building the list of DeclContexts between the current
233:     /// context and the top of the translation unit
234:     static DeclContextList buildContextChain(DeclContext *Start);
235: 
236:     unsigned buildNestedNameSpecifier(DeclContextList &DeclChain,
237:                                       NestedNameSpecifier &NNS);
238: 
239:   public:
240:     NamespaceSpecifierSet(ASTContext &Context, DeclContext *CurContext,
241:                           CXXScopeSpec *CurScopeSpec);
242: 
243:     /// Add the DeclContext (a namespace or record) to the set, computing
244:     /// the corresponding NestedNameSpecifier and its distance in the process.
245:     void addNameSpecifier(DeclContext *Ctx);
246: 
247:     /// Provides flat iteration over specifiers, sorted by distance.
248:     class iterator
249:         : public llvm::iterator_facade_base<iterator, std::forward_iterator_tag,
250:                                             SpecifierInfo> {
251:       /// Always points to the last element in the distance map.
252:       const std::map<unsigned, SpecifierInfoList>::iterator OuterBack;
```
- EN: Key type declarations here include `iterator`. It exposes API surface such as `buildContextChain`, `addNameSpecifier`.
- 中文: 这里的重要类型声明包括 `iterator`。 它暴露了 `buildContextChain`, `addNameSpecifier` 等接口。

### Lines 253-280

```cpp
253:       /// Iterator on the distance map.
254:       std::map<unsigned, SpecifierInfoList>::iterator Outer;
255:       /// Iterator on an element in the distance map.
256:       SpecifierInfoList::iterator Inner;
257: 
258:     public:
259:       iterator(NamespaceSpecifierSet &Set, bool IsAtEnd)
260:           : OuterBack(std::prev(Set.DistanceMap.end())),
261:             Outer(Set.DistanceMap.begin()),
262:             Inner(!IsAtEnd ? Outer->second.begin() : OuterBack->second.end()) {
263:         assert(!Set.DistanceMap.empty());
264:       }
265: 
266:       iterator &operator++() {
267:         ++Inner;
268:         if (Inner == Outer->second.end() && Outer != OuterBack) {
269:           ++Outer;
270:           Inner = Outer->second.begin();
271:         }
272:         return *this;
273:       }
274: 
275:       SpecifierInfo &operator*() { return *Inner; }
276:       bool operator==(const iterator &RHS) const { return Inner == RHS.Inner; }
277:     };
278: 
279:     iterator begin() { return iterator(*this, /*IsAtEnd=*/false); }
280:     iterator end() { return iterator(*this, /*IsAtEnd=*/true); }
```
- EN: It exposes API surface such as `Inner`, `assert`, `begin`, `end`.
- 中文: 它暴露了 `Inner`, `assert`, `begin`, `end` 等接口。

### Lines 281-308

```cpp
281:   };
282: 
283:   void addName(StringRef Name, NamedDecl *ND,
284:                NestedNameSpecifier NNS = std::nullopt, bool isKeyword = false);
285: 
286:   /// Find any visible decls for the given typo correction candidate.
287:   /// If none are found, it to the set of candidates for which qualified lookups
288:   /// will be performed to find possible nested name specifier changes.
289:   bool resolveCorrection(TypoCorrection &Candidate);
290: 
291:   /// Perform qualified lookups on the queued set of typo correction
292:   /// candidates and add the nested name specifier changes to each candidate if
293:   /// a lookup succeeds (at which point the candidate will be returned to the
294:   /// main pool of potential corrections).
295:   void performQualifiedLookups();
296: 
297:   /// The name written that is a typo in the source.
298:   IdentifierInfo *Typo;
299: 
300:   /// The results found that have the smallest edit distance
301:   /// found (so far) with the typo name.
302:   ///
303:   /// The pointer value being set to the current DeclContext indicates
304:   /// whether there is a keyword with this name.
305:   TypoEditDistanceMap CorrectionResults;
306: 
307:   SmallVector<TypoCorrection, 4> ValidatedCorrections;
308:   size_t CurrentTCIndex;
```
- EN: It exposes API surface such as `resolveCorrection`, `performQualifiedLookups`.
- 中文: 它暴露了 `resolveCorrection`, `performQualifiedLookups` 等接口。

### Lines 309-325

```cpp
309:   size_t SavedTCIndex;
310: 
311:   Sema &SemaRef;
312:   Scope *S;
313:   std::unique_ptr<CXXScopeSpec> SS;
314:   std::unique_ptr<CorrectionCandidateCallback> CorrectionValidator;
315:   DeclContext *MemberContext;
316:   LookupResult Result;
317:   NamespaceSpecifierSet Namespaces;
318:   SmallVector<TypoCorrection, 2> QualifiedResults;
319:   bool EnteringContext;
320:   bool SearchNamespaces;
321: };
322: 
323: } // end namespace clang
324: 
325: #endif
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `clang`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `clang` 的命名空间作用域。

## Key Concepts / 关键概念

- `TypoCorrectionConsumer`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `NamespaceSpecifierSet`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `SpecifierInfo`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `iterator`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `FTIHasSingleVoidParameter`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `cast`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `FTIHasNonVoidParameters`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `DeclAttrsMatchCUDAMode`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/AST/ASTContext.h`, `clang/AST/DeclTemplate.h`, `clang/Sema/Lookup.h`, `clang/Sema/Sema.h`, `clang/Sema/SemaDiagnostic.h`
- Forward declarations / 前向声明: None / 无
- Namespace context / 命名空间上下文: `clang`
- Macro-style dependencies / 宏式依赖: `SS`
