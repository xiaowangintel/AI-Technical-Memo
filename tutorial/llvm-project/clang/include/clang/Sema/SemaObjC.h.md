# SemaObjC.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Sema/SemaObjC.h`
- Repository: `llvm-project`
- Purpose (EN): This file declares semantic analysis for Objective-C.
- 用途（中文）: 该文件为 Sema 子系统中的 Sema Obj C 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-48

```cpp
 1: //===----- SemaObjC.h ------ Semantic Analysis for Objective-C ------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: /// \file
 9: /// This file declares semantic analysis for Objective-C.
10: ///
11: //===----------------------------------------------------------------------===//
12: 
13: #ifndef LLVM_CLANG_SEMA_SEMAOBJC_H
14: #define LLVM_CLANG_SEMA_SEMAOBJC_H
15: 
16: #include "clang/AST/ASTFwd.h"
17: #include "clang/AST/DeclObjC.h"
18: #include "clang/AST/NSAPI.h"
19: #include "clang/AST/OperationKinds.h"
20: #include "clang/AST/Type.h"
21: #include "clang/Basic/IdentifierTable.h"
22: #include "clang/Basic/LLVM.h"
23: #include "clang/Basic/SourceLocation.h"
24: #include "clang/Basic/Specifiers.h"
25: #include "clang/Basic/TokenKinds.h"
26: #include "clang/Sema/DeclSpec.h"
27: #include "clang/Sema/ObjCMethodList.h"
28: #include "clang/Sema/Ownership.h"
29: #include "clang/Sema/Redeclaration.h"
30: #include "clang/Sema/Sema.h"
31: #include "clang/Sema/SemaBase.h"
32: #include "llvm/ADT/DenseMap.h"
33: #include "llvm/ADT/MapVector.h"
34: #include "llvm/ADT/SmallPtrSet.h"
35: #include <memory>
36: #include <optional>
37: #include <utility>
38: 
39: namespace clang {
40: 
41: class AttributeCommonInfo;
42: class AvailabilitySpec;
43: enum class CheckedConversionKind;
44: class DeclGroupRef;
45: class LookupResult;
46: struct ObjCDictionaryElement;
47: class ParsedAttr;
48: class ParsedAttributesView;
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/AST/ASTFwd.h`, `clang/AST/DeclObjC.h`, `clang/AST/NSAPI.h` and 19 more. It opens, closes, or documents namespace scope for `clang`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/AST/ASTFwd.h`, `clang/AST/DeclObjC.h`, `clang/AST/NSAPI.h` 以及另外 19 项依赖。 它打开、关闭或说明了 `clang` 的命名空间作用域。

### Lines 49-96

```cpp
49: class Scope;
50: struct SkipBodyInfo;
51: 
52: class SemaObjC : public SemaBase {
53: public:
54:   SemaObjC(Sema &S);
55: 
56:   ExprResult CheckObjCForCollectionOperand(SourceLocation forLoc,
57:                                            Expr *collection);
58:   StmtResult ActOnObjCForCollectionStmt(SourceLocation ForColLoc, Stmt *First,
59:                                         Expr *collection,
60:                                         SourceLocation RParenLoc);
61:   /// FinishObjCForCollectionStmt - Attach the body to a objective-C foreach
62:   /// statement.
63:   StmtResult FinishObjCForCollectionStmt(Stmt *ForCollection, Stmt *Body);
64: 
65:   StmtResult ActOnObjCAtCatchStmt(SourceLocation AtLoc, SourceLocation RParen,
66:                                   Decl *Parm, Stmt *Body);
67: 
68:   StmtResult ActOnObjCAtFinallyStmt(SourceLocation AtLoc, Stmt *Body);
69: 
70:   StmtResult ActOnObjCAtTryStmt(SourceLocation AtLoc, Stmt *Try,
71:                                 MultiStmtArg Catch, Stmt *Finally);
72: 
73:   StmtResult BuildObjCAtThrowStmt(SourceLocation AtLoc, Expr *Throw);
74:   StmtResult ActOnObjCAtThrowStmt(SourceLocation AtLoc, Expr *Throw,
75:                                   Scope *CurScope);
76:   ExprResult ActOnObjCAtSynchronizedOperand(SourceLocation atLoc,
77:                                             Expr *operand);
78:   StmtResult ActOnObjCAtSynchronizedStmt(SourceLocation AtLoc, Expr *SynchExpr,
79:                                          Stmt *SynchBody);
80: 
81:   StmtResult ActOnObjCAutoreleasePoolStmt(SourceLocation AtLoc, Stmt *Body);
82: 
83:   /// Build a an Objective-C protocol-qualified 'id' type where no
84:   /// base type was specified.
85:   TypeResult actOnObjCProtocolQualifierType(
86:       SourceLocation lAngleLoc, ArrayRef<Decl *> protocols,
87:       ArrayRef<SourceLocation> protocolLocs, SourceLocation rAngleLoc);
88: 
89:   /// Build a specialized and/or protocol-qualified Objective-C type.
90:   TypeResult actOnObjCTypeArgsAndProtocolQualifiers(
91:       Scope *S, SourceLocation Loc, ParsedType BaseType,
92:       SourceLocation TypeArgsLAngleLoc, ArrayRef<ParsedType> TypeArgs,
93:       SourceLocation TypeArgsRAngleLoc, SourceLocation ProtocolLAngleLoc,
94:       ArrayRef<Decl *> Protocols, ArrayRef<SourceLocation> ProtocolLocs,
95:       SourceLocation ProtocolRAngleLoc);
96: 
```
- EN: Key type declarations here include `Scope`, `SkipBodyInfo`, `SemaObjC`. It exposes API surface such as `SemaObjC`, `FinishObjCForCollectionStmt`, `ActOnObjCAtFinallyStmt`, `BuildObjCAtThrowStmt`.
- 中文: 这里的重要类型声明包括 `Scope`, `SkipBodyInfo`, `SemaObjC`。 它暴露了 `SemaObjC`, `FinishObjCForCollectionStmt`, `ActOnObjCAtFinallyStmt`, `BuildObjCAtThrowStmt` 等接口。

### Lines 97-144

```cpp
 97:   /// Build an Objective-C type parameter type.
 98:   QualType BuildObjCTypeParamType(const ObjCTypeParamDecl *Decl,
 99:                                   SourceLocation ProtocolLAngleLoc,
100:                                   ArrayRef<ObjCProtocolDecl *> Protocols,
101:                                   ArrayRef<SourceLocation> ProtocolLocs,
102:                                   SourceLocation ProtocolRAngleLoc,
103:                                   bool FailOnError = false);
104: 
105:   /// Build an Objective-C object pointer type.
106:   QualType BuildObjCObjectType(
107:       QualType BaseType, SourceLocation Loc, SourceLocation TypeArgsLAngleLoc,
108:       ArrayRef<TypeSourceInfo *> TypeArgs, SourceLocation TypeArgsRAngleLoc,
109:       SourceLocation ProtocolLAngleLoc, ArrayRef<ObjCProtocolDecl *> Protocols,
110:       ArrayRef<SourceLocation> ProtocolLocs, SourceLocation ProtocolRAngleLoc,
111:       bool FailOnError, bool Rebuilding);
112: 
113:   /// The parser has parsed the context-sensitive type 'instancetype'
114:   /// in an Objective-C message declaration. Return the appropriate type.
115:   ParsedType ActOnObjCInstanceType(SourceLocation Loc);
116: 
117:   /// checkRetainCycles - Check whether an Objective-C message send
118:   /// might create an obvious retain cycle.
119:   void checkRetainCycles(ObjCMessageExpr *msg);
120:   void checkRetainCycles(Expr *receiver, Expr *argument);
121:   void checkRetainCycles(VarDecl *Var, Expr *Init);
122: 
123:   bool CheckObjCString(Expr *Arg);
124:   bool CheckObjCMethodCall(ObjCMethodDecl *Method, SourceLocation loc,
125:                            ArrayRef<const Expr *> Args);
126:   /// Check whether receiver is mutable ObjC container which
127:   /// attempts to add itself into the container
128:   void CheckObjCCircularContainer(ObjCMessageExpr *Message);
129: 
130:   void ActOnObjCContainerStartDefinition(ObjCContainerDecl *IDecl);
131:   void ActOnObjCContainerFinishDefinition();
132: 
133:   /// Invoked when we must temporarily exit the objective-c container
134:   /// scope for parsing/looking-up C constructs.
135:   ///
136:   /// Must be followed by a call to \see ActOnObjCReenterContainerContext
137:   void ActOnObjCTemporaryExitContainerContext(ObjCContainerDecl *ObjCCtx);
138:   void ActOnObjCReenterContainerContext(ObjCContainerDecl *ObjCCtx);
139: 
140:   const DeclContext *getCurObjCLexicalContext() const;
141: 
142:   ObjCProtocolDecl *LookupProtocol(
143:       IdentifierInfo *II, SourceLocation IdLoc,
144:       RedeclarationKind Redecl = RedeclarationKind::NotForRedeclaration);
```
- EN: It exposes API surface such as `ActOnObjCInstanceType`, `checkRetainCycles`, `CheckObjCString`, `CheckObjCCircularContainer`.
- 中文: 它暴露了 `ActOnObjCInstanceType`, `checkRetainCycles`, `CheckObjCString`, `CheckObjCCircularContainer` 等接口。

### Lines 145-192

```cpp
145: 
146:   bool isObjCWritebackConversion(QualType FromType, QualType ToType,
147:                                  QualType &ConvertedType);
148: 
149:   enum ObjCSubscriptKind { OS_Array, OS_Dictionary, OS_Error };
150:   ObjCSubscriptKind CheckSubscriptingKind(Expr *FromE);
151: 
152:   /// AddCFAuditedAttribute - Check whether we're currently within
153:   /// '\#pragma clang arc_cf_code_audited' and, if so, consider adding
154:   /// the appropriate attribute.
155:   void AddCFAuditedAttribute(Decl *D);
156: 
157:   /// The struct behind the CFErrorRef pointer.
158:   RecordDecl *CFError = nullptr;
159:   bool isCFError(RecordDecl *D);
160: 
161:   IdentifierInfo *getNSErrorIdent();
162: 
163:   bool GetFormatNSStringIdx(const FormatAttr *Format, unsigned &Idx);
164: 
165:   /// Diagnose use of %s directive in an NSString which is being passed
166:   /// as formatting string to formatting method.
167:   void DiagnoseCStringFormatDirectiveInCFAPI(const NamedDecl *FDecl,
168:                                              Expr **Args, unsigned NumArgs);
169: 
170:   bool isSignedCharBool(QualType Ty);
171: 
172:   void adornBoolConversionDiagWithTernaryFixit(
173:       const Expr *SourceExpr, const Sema::SemaDiagnosticBuilder &Builder);
174: 
175:   /// Check an Objective-C dictionary literal being converted to the given
176:   /// target type.
177:   void checkDictionaryLiteral(QualType TargetType,
178:                               ObjCDictionaryLiteral *DictionaryLiteral);
179: 
180:   /// Check an Objective-C array literal being converted to the given
181:   /// target type.
182:   void checkArrayLiteral(QualType TargetType, ObjCArrayLiteral *ArrayLiteral);
183: 
184: private:
185:   IdentifierInfo *Ident_NSError = nullptr;
186: 
187:   //
188:   //
189:   // -------------------------------------------------------------------------
190:   //
191:   //
192: 
```
- EN: It introduces enum-based state or option sets such as `ObjCSubscriptKind`. It exposes API surface such as `CheckSubscriptingKind`, `AddCFAuditedAttribute`, `isCFError`, `getNSErrorIdent`.
- 中文: 它引入了 `ObjCSubscriptKind` 等基于枚举的状态或选项集合。 它暴露了 `CheckSubscriptingKind`, `AddCFAuditedAttribute`, `isCFError`, `getNSErrorIdent` 等接口。

### Lines 193-240

```cpp
193:   /// \name ObjC Declarations
194:   /// Implementations are in SemaDeclObjC.cpp
195:   ///@{
196: 
197: public:
198:   enum ObjCSpecialMethodKind {
199:     OSMK_None,
200:     OSMK_Alloc,
201:     OSMK_New,
202:     OSMK_Copy,
203:     OSMK_RetainingInit,
204:     OSMK_NonRetainingInit
205:   };
206: 
207:   /// Method selectors used in a \@selector expression. Used for implementation
208:   /// of -Wselector.
209:   llvm::MapVector<Selector, SourceLocation> ReferencedSelectors;
210: 
211:   using GlobalMethodPool =
212:       llvm::DenseMap<Selector, std::pair<ObjCMethodList, ObjCMethodList>>;
213: 
214:   /// Method Pool - allows efficient lookup when typechecking messages to "id".
215:   /// We need to maintain a list, since selectors can have differing signatures
216:   /// across classes. In Cocoa, this happens to be extremely uncommon (only 1%
217:   /// of selectors are "overloaded").
218:   /// At the head of the list it is recorded whether there were 0, 1, or >= 2
219:   /// methods inside categories with a particular selector.
220:   GlobalMethodPool MethodPool;
221: 
222:   typedef llvm::SmallPtrSet<Selector, 8> SelectorSet;
223: 
224:   enum MethodMatchStrategy { MMS_loose, MMS_strict };
225: 
226:   enum ObjCContainerKind {
227:     OCK_None = -1,
228:     OCK_Interface = 0,
229:     OCK_Protocol,
230:     OCK_Category,
231:     OCK_ClassExtension,
232:     OCK_Implementation,
233:     OCK_CategoryImplementation
234:   };
235:   ObjCContainerKind getObjCContainerKind() const;
236: 
237:   DeclResult actOnObjCTypeParam(Scope *S, ObjCTypeParamVariance variance,
238:                                 SourceLocation varianceLoc, unsigned index,
239:                                 IdentifierInfo *paramName,
240:                                 SourceLocation paramLoc,
```
- EN: It introduces enum-based state or option sets such as `ObjCSpecialMethodKind`, `MethodMatchStrategy`, `ObjCContainerKind`. It defines convenient aliases such as `GlobalMethodPool`. It exposes API surface such as `getObjCContainerKind`.
- 中文: 它引入了 `ObjCSpecialMethodKind`, `MethodMatchStrategy`, `ObjCContainerKind` 等基于枚举的状态或选项集合。 它定义了 `GlobalMethodPool` 等便捷别名。 它暴露了 `getObjCContainerKind` 等接口。

### Lines 241-288

```cpp
241:                                 SourceLocation colonLoc, ParsedType typeBound);
242: 
243:   ObjCTypeParamList *actOnObjCTypeParamList(Scope *S, SourceLocation lAngleLoc,
244:                                             ArrayRef<Decl *> typeParams,
245:                                             SourceLocation rAngleLoc);
246:   void popObjCTypeParamList(Scope *S, ObjCTypeParamList *typeParamList);
247: 
248:   ObjCInterfaceDecl *ActOnStartClassInterface(
249:       Scope *S, SourceLocation AtInterfaceLoc, IdentifierInfo *ClassName,
250:       SourceLocation ClassLoc, ObjCTypeParamList *typeParamList,
251:       IdentifierInfo *SuperName, SourceLocation SuperLoc,
252:       ArrayRef<ParsedType> SuperTypeArgs, SourceRange SuperTypeArgsRange,
253:       Decl *const *ProtoRefs, unsigned NumProtoRefs,
254:       const SourceLocation *ProtoLocs, SourceLocation EndProtoLoc,
255:       const ParsedAttributesView &AttrList, SkipBodyInfo *SkipBody);
256: 
257:   void ActOnSuperClassOfClassInterface(
258:       Scope *S, SourceLocation AtInterfaceLoc, ObjCInterfaceDecl *IDecl,
259:       IdentifierInfo *ClassName, SourceLocation ClassLoc,
260:       IdentifierInfo *SuperName, SourceLocation SuperLoc,
261:       ArrayRef<ParsedType> SuperTypeArgs, SourceRange SuperTypeArgsRange);
262: 
263:   void ActOnTypedefedProtocols(SmallVectorImpl<Decl *> &ProtocolRefs,
264:                                SmallVectorImpl<SourceLocation> &ProtocolLocs,
265:                                IdentifierInfo *SuperName,
266:                                SourceLocation SuperLoc);
267: 
268:   Decl *ActOnCompatibilityAlias(SourceLocation AtCompatibilityAliasLoc,
269:                                 IdentifierInfo *AliasName,
270:                                 SourceLocation AliasLocation,
271:                                 IdentifierInfo *ClassName,
272:                                 SourceLocation ClassLocation);
273: 
274:   bool CheckForwardProtocolDeclarationForCircularDependency(
275:       IdentifierInfo *PName, SourceLocation &PLoc, SourceLocation PrevLoc,
276:       const ObjCList<ObjCProtocolDecl> &PList);
277: 
278:   ObjCProtocolDecl *ActOnStartProtocolInterface(
279:       SourceLocation AtProtoInterfaceLoc, IdentifierInfo *ProtocolName,
280:       SourceLocation ProtocolLoc, Decl *const *ProtoRefNames,
281:       unsigned NumProtoRefs, const SourceLocation *ProtoLocs,
282:       SourceLocation EndProtoLoc, const ParsedAttributesView &AttrList,
283:       SkipBodyInfo *SkipBody);
284: 
285:   ObjCCategoryDecl *ActOnStartCategoryInterface(
286:       SourceLocation AtInterfaceLoc, const IdentifierInfo *ClassName,
287:       SourceLocation ClassLoc, ObjCTypeParamList *typeParamList,
288:       const IdentifierInfo *CategoryName, SourceLocation CategoryLoc,
```
- EN: It exposes API surface such as `popObjCTypeParamList`.
- 中文: 它暴露了 `popObjCTypeParamList` 等接口。

### Lines 289-336

```cpp
289:       Decl *const *ProtoRefs, unsigned NumProtoRefs,
290:       const SourceLocation *ProtoLocs, SourceLocation EndProtoLoc,
291:       const ParsedAttributesView &AttrList);
292: 
293:   ObjCImplementationDecl *ActOnStartClassImplementation(
294:       SourceLocation AtClassImplLoc, const IdentifierInfo *ClassName,
295:       SourceLocation ClassLoc, const IdentifierInfo *SuperClassname,
296:       SourceLocation SuperClassLoc, const ParsedAttributesView &AttrList);
297: 
298:   ObjCCategoryImplDecl *ActOnStartCategoryImplementation(
299:       SourceLocation AtCatImplLoc, const IdentifierInfo *ClassName,
300:       SourceLocation ClassLoc, const IdentifierInfo *CatName,
301:       SourceLocation CatLoc, const ParsedAttributesView &AttrList);
302: 
303:   using DeclGroupPtrTy = OpaquePtr<DeclGroupRef>;
304: 
305:   DeclGroupPtrTy ActOnFinishObjCImplementation(Decl *ObjCImpDecl,
306:                                                ArrayRef<Decl *> Decls);
307: 
308:   DeclGroupPtrTy
309:   ActOnForwardProtocolDeclaration(SourceLocation AtProtoclLoc,
310:                                   ArrayRef<IdentifierLoc> IdentList,
311:                                   const ParsedAttributesView &attrList);
312: 
313:   void FindProtocolDeclaration(bool WarnOnDeclarations, bool ForObjCContainer,
314:                                ArrayRef<IdentifierLoc> ProtocolId,
315:                                SmallVectorImpl<Decl *> &Protocols);
316: 
317:   void DiagnoseTypeArgsAndProtocols(IdentifierInfo *ProtocolId,
318:                                     SourceLocation ProtocolLoc,
319:                                     IdentifierInfo *TypeArgId,
320:                                     SourceLocation TypeArgLoc,
321:                                     bool SelectProtocolFirst = false);
322: 
323:   /// Given a list of identifiers (and their locations), resolve the
324:   /// names to either Objective-C protocol qualifiers or type
325:   /// arguments, as appropriate.
326:   void actOnObjCTypeArgsOrProtocolQualifiers(
327:       Scope *S, ParsedType baseType, SourceLocation lAngleLoc,
328:       ArrayRef<IdentifierInfo *> identifiers,
329:       ArrayRef<SourceLocation> identifierLocs, SourceLocation rAngleLoc,
330:       SourceLocation &typeArgsLAngleLoc, SmallVectorImpl<ParsedType> &typeArgs,
331:       SourceLocation &typeArgsRAngleLoc, SourceLocation &protocolLAngleLoc,
332:       SmallVectorImpl<Decl *> &protocols, SourceLocation &protocolRAngleLoc,
333:       bool warnOnIncompleteProtocols);
334: 
335:   void DiagnoseClassExtensionDupMethods(ObjCCategoryDecl *CAT,
336:                                         ObjCInterfaceDecl *ID);
```
- EN: It defines convenient aliases such as `DeclGroupPtrTy`.
- 中文: 它定义了 `DeclGroupPtrTy` 等便捷别名。

### Lines 337-384

```cpp
337: 
338:   Decl *ActOnAtEnd(Scope *S, SourceRange AtEnd,
339:                    ArrayRef<Decl *> allMethods = {},
340:                    ArrayRef<DeclGroupPtrTy> allTUVars = {});
341: 
342:   struct ObjCArgInfo {
343:     IdentifierInfo *Name;
344:     SourceLocation NameLoc;
345:     // The Type is null if no type was specified, and the DeclSpec is invalid
346:     // in this case.
347:     ParsedType Type;
348:     ObjCDeclSpec DeclSpec;
349: 
350:     /// ArgAttrs - Attribute list for this argument.
351:     ParsedAttributesView ArgAttrs;
352:   };
353: 
354:   ParmVarDecl *ActOnMethodParmDeclaration(Scope *S, ObjCArgInfo &ArgInfo,
355:                                           int ParamIndex,
356:                                           bool MethodDefinition);
357: 
358:   Decl *ActOnMethodDeclaration(
359:       Scope *S,
360:       SourceLocation BeginLoc, // location of the + or -.
361:       SourceLocation EndLoc,   // location of the ; or {.
362:       tok::TokenKind MethodType, ObjCDeclSpec &ReturnQT, ParsedType ReturnType,
363:       ArrayRef<SourceLocation> SelectorLocs, Selector Sel,
364:       // optional arguments. The number of types/arguments is obtained
365:       // from the Sel.getNumArgs().
366:       ParmVarDecl **ArgInfo, DeclaratorChunk::ParamInfo *CParamInfo,
367:       unsigned CNumArgs, // c-style args
368:       const ParsedAttributesView &AttrList, tok::ObjCKeywordKind MethodImplKind,
369:       bool isVariadic, bool MethodDefinition);
370: 
371:   bool CheckARCMethodDecl(ObjCMethodDecl *method);
372: 
373:   bool checkInitMethod(ObjCMethodDecl *method, QualType receiverTypeIfCall);
374: 
375:   /// Check whether the given new method is a valid override of the
376:   /// given overridden method, and set any properties that should be inherited.
377:   void CheckObjCMethodOverride(ObjCMethodDecl *NewMethod,
378:                                const ObjCMethodDecl *Overridden);
379: 
380:   /// Describes the compatibility of a result type with its method.
381:   enum ResultTypeCompatibilityKind {
382:     RTC_Compatible,
383:     RTC_Incompatible,
384:     RTC_Unknown
```
- EN: Key type declarations here include `ObjCArgInfo`. It introduces enum-based state or option sets such as `ResultTypeCompatibilityKind`. It exposes API surface such as `CheckARCMethodDecl`, `checkInitMethod`.
- 中文: 这里的重要类型声明包括 `ObjCArgInfo`。 它引入了 `ResultTypeCompatibilityKind` 等基于枚举的状态或选项集合。 它暴露了 `CheckARCMethodDecl`, `checkInitMethod` 等接口。

### Lines 385-432

```cpp
385:   };
386: 
387:   void CheckObjCMethodDirectOverrides(ObjCMethodDecl *method,
388:                                       ObjCMethodDecl *overridden);
389: 
390:   void CheckObjCMethodOverrides(ObjCMethodDecl *ObjCMethod,
391:                                 ObjCInterfaceDecl *CurrentClass,
392:                                 ResultTypeCompatibilityKind RTC);
393: 
394:   /// AddAnyMethodToGlobalPool - Add any method, instance or factory to global
395:   /// pool.
396:   void AddAnyMethodToGlobalPool(Decl *D);
397: 
398:   void ActOnStartOfObjCMethodDef(Scope *S, Decl *D);
399:   bool isObjCMethodDecl(Decl *D) { return isa_and_nonnull<ObjCMethodDecl>(D); }
400: 
401:   /// CheckImplementationIvars - This routine checks if the instance variables
402:   /// listed in the implelementation match those listed in the interface.
403:   void CheckImplementationIvars(ObjCImplementationDecl *ImpDecl,
404:                                 ObjCIvarDecl **Fields, unsigned nIvars,
405:                                 SourceLocation Loc);
406: 
407:   void WarnConflictingTypedMethods(ObjCMethodDecl *Method,
408:                                    ObjCMethodDecl *MethodDecl,
409:                                    bool IsProtocolMethodDecl);
410: 
411:   void CheckConflictingOverridingMethod(ObjCMethodDecl *Method,
412:                                         ObjCMethodDecl *Overridden,
413:                                         bool IsProtocolMethodDecl);
414: 
415:   /// WarnExactTypedMethods - This routine issues a warning if method
416:   /// implementation declaration matches exactly that of its declaration.
417:   void WarnExactTypedMethods(ObjCMethodDecl *Method, ObjCMethodDecl *MethodDecl,
418:                              bool IsProtocolMethodDecl);
419: 
420:   /// MatchAllMethodDeclarations - Check methods declaraed in interface or
421:   /// or protocol against those declared in their implementations.
422:   void MatchAllMethodDeclarations(
423:       const SelectorSet &InsMap, const SelectorSet &ClsMap,
424:       SelectorSet &InsMapSeen, SelectorSet &ClsMapSeen, ObjCImplDecl *IMPDecl,
425:       ObjCContainerDecl *IDecl, bool &IncompleteImpl, bool ImmediateClass,
426:       bool WarnCategoryMethodImpl = false);
427: 
428:   /// CheckCategoryVsClassMethodMatches - Checks that methods implemented in
429:   /// category matches with those implemented in its primary class and
430:   /// warns each time an exact match is found.
431:   void CheckCategoryVsClassMethodMatches(ObjCCategoryImplDecl *CatIMP);
432: 
```
- EN: It exposes API surface such as `AddAnyMethodToGlobalPool`, `ActOnStartOfObjCMethodDef`, `isObjCMethodDecl`, `CheckCategoryVsClassMethodMatches`.
- 中文: 它暴露了 `AddAnyMethodToGlobalPool`, `ActOnStartOfObjCMethodDef`, `isObjCMethodDecl`, `CheckCategoryVsClassMethodMatches` 等接口。

### Lines 433-480

```cpp
433:   /// ImplMethodsVsClassMethods - This is main routine to warn if any method
434:   /// remains unimplemented in the class or category \@implementation.
435:   void ImplMethodsVsClassMethods(Scope *S, ObjCImplDecl *IMPDecl,
436:                                  ObjCContainerDecl *IDecl,
437:                                  bool IncompleteImpl = false);
438: 
439:   DeclGroupPtrTy ActOnForwardClassDeclaration(
440:       SourceLocation Loc, IdentifierInfo **IdentList, SourceLocation *IdentLocs,
441:       ArrayRef<ObjCTypeParamList *> TypeParamLists, unsigned NumElts);
442: 
443:   /// MatchTwoMethodDeclarations - Checks if two methods' type match and returns
444:   /// true, or false, accordingly.
445:   bool MatchTwoMethodDeclarations(const ObjCMethodDecl *Method,
446:                                   const ObjCMethodDecl *PrevMethod,
447:                                   MethodMatchStrategy strategy = MMS_strict);
448: 
449:   /// Add the given method to the list of globally-known methods.
450:   void addMethodToGlobalList(ObjCMethodList *List, ObjCMethodDecl *Method);
451: 
452:   void ReadMethodPool(Selector Sel);
453:   void updateOutOfDateSelector(Selector Sel);
454: 
455:   /// - Returns instance or factory methods in global method pool for
456:   /// given selector. It checks the desired kind first, if none is found, and
457:   /// parameter checkTheOther is set, it then checks the other kind. If no such
458:   /// method or only one method is found, function returns false; otherwise, it
459:   /// returns true.
460:   bool
461:   CollectMultipleMethodsInGlobalPool(Selector Sel,
462:                                      SmallVectorImpl<ObjCMethodDecl *> &Methods,
463:                                      bool InstanceFirst, bool CheckTheOther,
464:                                      const ObjCObjectType *TypeBound = nullptr);
465: 
466:   bool
467:   AreMultipleMethodsInGlobalPool(Selector Sel, ObjCMethodDecl *BestMethod,
468:                                  SourceRange R, bool receiverIdOrClass,
469:                                  SmallVectorImpl<ObjCMethodDecl *> &Methods);
470: 
471:   void
472:   DiagnoseMultipleMethodInGlobalPool(SmallVectorImpl<ObjCMethodDecl *> &Methods,
473:                                      Selector Sel, SourceRange R,
474:                                      bool receiverIdOrClass);
475: 
476:   const ObjCMethodDecl *
477:   SelectorsForTypoCorrection(Selector Sel, QualType ObjectType = QualType());
478:   /// LookupImplementedMethodInGlobalPool - Returns the method which has an
479:   /// implementation.
480:   ObjCMethodDecl *LookupImplementedMethodInGlobalPool(Selector Sel);
```
- EN: It exposes API surface such as `addMethodToGlobalList`, `ReadMethodPool`, `updateOutOfDateSelector`, `SelectorsForTypoCorrection`.
- 中文: 它暴露了 `addMethodToGlobalList`, `ReadMethodPool`, `updateOutOfDateSelector`, `SelectorsForTypoCorrection` 等接口。

### Lines 481-528

```cpp
481: 
482:   void DiagnoseDuplicateIvars(ObjCInterfaceDecl *ID, ObjCInterfaceDecl *SID);
483: 
484:   /// Checks that the Objective-C declaration is declared in the global scope.
485:   /// Emits an error and marks the declaration as invalid if it's not declared
486:   /// in the global scope.
487:   bool CheckObjCDeclScope(Decl *D);
488: 
489:   void ActOnDefs(Scope *S, Decl *TagD, SourceLocation DeclStart,
490:                  const IdentifierInfo *ClassName,
491:                  SmallVectorImpl<Decl *> &Decls);
492: 
493:   VarDecl *BuildObjCExceptionDecl(TypeSourceInfo *TInfo, QualType ExceptionType,
494:                                   SourceLocation StartLoc, SourceLocation IdLoc,
495:                                   const IdentifierInfo *Id,
496:                                   bool Invalid = false);
497: 
498:   Decl *ActOnObjCExceptionDecl(Scope *S, Declarator &D);
499: 
500:   /// CollectIvarsToConstructOrDestruct - Collect those ivars which require
501:   /// initialization.
502:   void
503:   CollectIvarsToConstructOrDestruct(ObjCInterfaceDecl *OI,
504:                                     SmallVectorImpl<ObjCIvarDecl *> &Ivars);
505: 
506:   void DiagnoseUseOfUnimplementedSelectors();
507: 
508:   /// DiagnoseUnusedBackingIvarInAccessor - Issue an 'unused' warning if ivar
509:   /// which backs the property is not used in the property's accessor.
510:   void DiagnoseUnusedBackingIvarInAccessor(Scope *S,
511:                                            const ObjCImplementationDecl *ImplD);
512: 
513:   /// GetIvarBackingPropertyAccessor - If method is a property setter/getter and
514:   /// it property has a backing ivar, returns this ivar; otherwise, returns
515:   /// NULL. It also returns ivar's property on success.
516:   ObjCIvarDecl *
517:   GetIvarBackingPropertyAccessor(const ObjCMethodDecl *Method,
518:                                  const ObjCPropertyDecl *&PDecl) const;
519: 
520:   /// AddInstanceMethodToGlobalPool - All instance methods in a translation
521:   /// unit are added to a global pool. This allows us to efficiently associate
522:   /// a selector with a method declaraation for purposes of typechecking
523:   /// messages sent to "id" (where the class of the object is unknown).
524:   void AddInstanceMethodToGlobalPool(ObjCMethodDecl *Method,
525:                                      bool impl = false) {
526:     AddMethodToGlobalPool(Method, impl, /*instance*/ true);
527:   }
528: 
```
- EN: It exposes API surface such as `DiagnoseDuplicateIvars`, `CheckObjCDeclScope`, `ActOnObjCExceptionDecl`, `DiagnoseUseOfUnimplementedSelectors`.
- 中文: 它暴露了 `DiagnoseDuplicateIvars`, `CheckObjCDeclScope`, `ActOnObjCExceptionDecl`, `DiagnoseUseOfUnimplementedSelectors` 等接口。

### Lines 529-576

```cpp
529:   /// AddFactoryMethodToGlobalPool - Same as above, but for factory methods.
530:   void AddFactoryMethodToGlobalPool(ObjCMethodDecl *Method, bool impl = false) {
531:     AddMethodToGlobalPool(Method, impl, /*instance*/ false);
532:   }
533: 
534:   QualType AdjustParameterTypeForObjCAutoRefCount(QualType T,
535:                                                   SourceLocation NameLoc,
536:                                                   TypeSourceInfo *TSInfo);
537: 
538:   /// Look for an Objective-C class in the translation unit.
539:   ///
540:   /// \param Id The name of the Objective-C class we're looking for. If
541:   /// typo-correction fixes this name, the Id will be updated
542:   /// to the fixed name.
543:   ///
544:   /// \param IdLoc The location of the name in the translation unit.
545:   ///
546:   /// \param DoTypoCorrection If true, this routine will attempt typo correction
547:   /// if there is no class with the given name.
548:   ///
549:   /// \returns The declaration of the named Objective-C class, or NULL if the
550:   /// class could not be found.
551:   ObjCInterfaceDecl *getObjCInterfaceDecl(const IdentifierInfo *&Id,
552:                                           SourceLocation IdLoc,
553:                                           bool TypoCorrection = false);
554: 
555:   bool inferObjCARCLifetime(ValueDecl *decl);
556: 
557:   /// SetIvarInitializers - This routine builds initialization ASTs for the
558:   /// Objective-C implementation whose ivars need be initialized.
559:   void SetIvarInitializers(ObjCImplementationDecl *ObjCImplementation);
560: 
561:   Decl *ActOnIvar(Scope *S, SourceLocation DeclStart, Declarator &D,
562:                   Expr *BitWidth, tok::ObjCKeywordKind visibility);
563: 
564:   ObjCContainerDecl *getObjCDeclContext() const;
565: 
566: private:
567:   /// AddMethodToGlobalPool - Add an instance or factory method to the global
568:   /// pool. See descriptoin of AddInstanceMethodToGlobalPool.
569:   void AddMethodToGlobalPool(ObjCMethodDecl *Method, bool impl, bool instance);
570: 
571:   /// LookupMethodInGlobalPool - Returns the instance or factory method and
572:   /// optionally warns if there are multiple signatures.
573:   ObjCMethodDecl *LookupMethodInGlobalPool(Selector Sel, SourceRange R,
574:                                            bool receiverIdOrClass,
575:                                            bool instance);
576: 
```
- EN: It exposes API surface such as `AddFactoryMethodToGlobalPool`, `AddMethodToGlobalPool`, `inferObjCARCLifetime`, `SetIvarInitializers`.
- 中文: 它暴露了 `AddFactoryMethodToGlobalPool`, `AddMethodToGlobalPool`, `inferObjCARCLifetime`, `SetIvarInitializers` 等接口。

### Lines 577-624

```cpp
577:   ///@}
578: 
579:   //
580:   //
581:   // -------------------------------------------------------------------------
582:   //
583:   //
584: 
585:   /// \name ObjC Expressions
586:   /// Implementations are in SemaExprObjC.cpp
587:   ///@{
588: 
589: public:
590:   /// Caches identifiers/selectors for NSFoundation APIs.
591:   std::unique_ptr<NSAPI> NSAPIObj;
592: 
593:   /// The declaration of the Objective-C NSNumber class.
594:   ObjCInterfaceDecl *NSNumberDecl;
595: 
596:   /// The declaration of the Objective-C NSValue class.
597:   ObjCInterfaceDecl *NSValueDecl;
598: 
599:   /// Pointer to NSNumber type (NSNumber *).
600:   QualType NSNumberPointer;
601: 
602:   /// Pointer to NSValue type (NSValue *).
603:   QualType NSValuePointer;
604: 
605:   /// The Objective-C NSNumber methods used to create NSNumber literals.
606:   ObjCMethodDecl *NSNumberLiteralMethods[NSAPI::NumNSNumberLiteralMethods];
607: 
608:   /// The declaration of the Objective-C NSString class.
609:   ObjCInterfaceDecl *NSStringDecl;
610: 
611:   /// Pointer to NSString type (NSString *).
612:   QualType NSStringPointer;
613: 
614:   /// The declaration of the stringWithUTF8String: method.
615:   ObjCMethodDecl *StringWithUTF8StringMethod;
616: 
617:   /// The declaration of the valueWithBytes:objCType: method.
618:   ObjCMethodDecl *ValueWithBytesObjCTypeMethod;
619: 
620:   /// The declaration of the Objective-C NSArray class.
621:   ObjCInterfaceDecl *NSArrayDecl;
622: 
623:   /// The declaration of the arrayWithObjects:count: method.
624:   ObjCMethodDecl *ArrayWithObjectsMethod;
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 625-672

```cpp
625: 
626:   /// The declaration of the Objective-C NSDictionary class.
627:   ObjCInterfaceDecl *NSDictionaryDecl;
628: 
629:   /// The declaration of the dictionaryWithObjects:forKeys:count: method.
630:   ObjCMethodDecl *DictionaryWithObjectsMethod;
631: 
632:   /// id<NSCopying> type.
633:   QualType QIDNSCopying;
634: 
635:   /// will hold 'respondsToSelector:'
636:   Selector RespondsToSelectorSel;
637: 
638:   ExprResult HandleExprPropertyRefExpr(const ObjCObjectPointerType *OPT,
639:                                        Expr *BaseExpr, SourceLocation OpLoc,
640:                                        DeclarationName MemberName,
641:                                        SourceLocation MemberLoc,
642:                                        SourceLocation SuperLoc,
643:                                        QualType SuperType, bool Super);
644: 
645:   ExprResult ActOnClassPropertyRefExpr(const IdentifierInfo &receiverName,
646:                                        const IdentifierInfo &propertyName,
647:                                        SourceLocation receiverNameLoc,
648:                                        SourceLocation propertyNameLoc);
649: 
650:   // ParseObjCStringLiteral - Parse Objective-C string literals.
651:   ExprResult ParseObjCStringLiteral(SourceLocation *AtLocs,
652:                                     ArrayRef<Expr *> Strings);
653: 
654:   ExprResult BuildObjCStringLiteral(SourceLocation AtLoc, StringLiteral *S);
655: 
656:   /// BuildObjCNumericLiteral - builds an ObjCBoxedExpr AST node for the
657:   /// numeric literal expression. Type of the expression will be "NSNumber *"
658:   /// or "id" if NSNumber is unavailable.
659:   ExprResult BuildObjCNumericLiteral(SourceLocation AtLoc, Expr *Number);
660:   ExprResult ActOnObjCBoolLiteral(SourceLocation AtLoc, SourceLocation ValueLoc,
661:                                   bool Value);
662:   ExprResult BuildObjCArrayLiteral(SourceRange SR, MultiExprArg Elements);
663: 
664:   /// BuildObjCBoxedExpr - builds an ObjCBoxedExpr AST node for the
665:   /// '@' prefixed parenthesized expression. The type of the expression will
666:   /// either be "NSNumber *", "NSString *" or "NSValue *" depending on the type
667:   /// of ValueType, which is allowed to be a built-in numeric type, "char *",
668:   /// "const char *" or C structure with attribute 'objc_boxable'.
669:   ExprResult BuildObjCBoxedExpr(SourceRange SR, Expr *ValueExpr);
670: 
671:   ExprResult BuildObjCSubscriptExpression(SourceLocation RB, Expr *BaseExpr,
672:                                           Expr *IndexExpr,
```
- EN: It exposes API surface such as `BuildObjCStringLiteral`, `BuildObjCNumericLiteral`, `BuildObjCArrayLiteral`, `BuildObjCBoxedExpr`.
- 中文: 它暴露了 `BuildObjCStringLiteral`, `BuildObjCNumericLiteral`, `BuildObjCArrayLiteral`, `BuildObjCBoxedExpr` 等接口。

### Lines 673-720

```cpp
673:                                           ObjCMethodDecl *getterMethod,
674:                                           ObjCMethodDecl *setterMethod);
675: 
676:   ExprResult
677:   BuildObjCDictionaryLiteral(SourceRange SR,
678:                              MutableArrayRef<ObjCDictionaryElement> Elements);
679: 
680:   ExprResult BuildObjCEncodeExpression(SourceLocation AtLoc,
681:                                        TypeSourceInfo *EncodedTypeInfo,
682:                                        SourceLocation RParenLoc);
683: 
684:   ExprResult ParseObjCEncodeExpression(SourceLocation AtLoc,
685:                                        SourceLocation EncodeLoc,
686:                                        SourceLocation LParenLoc, ParsedType Ty,
687:                                        SourceLocation RParenLoc);
688: 
689:   /// ParseObjCSelectorExpression - Build selector expression for \@selector
690:   ExprResult ParseObjCSelectorExpression(Selector Sel, SourceLocation AtLoc,
691:                                          SourceLocation SelLoc,
692:                                          SourceLocation LParenLoc,
693:                                          SourceLocation RParenLoc,
694:                                          bool WarnMultipleSelectors);
695: 
696:   /// ParseObjCProtocolExpression - Build protocol expression for \@protocol
697:   ExprResult ParseObjCProtocolExpression(IdentifierInfo *ProtocolName,
698:                                          SourceLocation AtLoc,
699:                                          SourceLocation ProtoLoc,
700:                                          SourceLocation LParenLoc,
701:                                          SourceLocation ProtoIdLoc,
702:                                          SourceLocation RParenLoc);
703: 
704:   ObjCMethodDecl *tryCaptureObjCSelf(SourceLocation Loc);
705: 
706:   /// Describes the kind of message expression indicated by a message
707:   /// send that starts with an identifier.
708:   enum ObjCMessageKind {
709:     /// The message is sent to 'super'.
710:     ObjCSuperMessage,
711:     /// The message is an instance message.
712:     ObjCInstanceMessage,
713:     /// The message is a class message, and the identifier is a type
714:     /// name.
715:     ObjCClassMessage
716:   };
717: 
718:   ObjCMessageKind getObjCMessageKind(Scope *S, IdentifierInfo *Name,
719:                                      SourceLocation NameLoc, bool IsSuper,
720:                                      bool HasTrailingDot,
```
- EN: It introduces enum-based state or option sets such as `ObjCMessageKind`. It exposes API surface such as `tryCaptureObjCSelf`.
- 中文: 它引入了 `ObjCMessageKind` 等基于枚举的状态或选项集合。 它暴露了 `tryCaptureObjCSelf` 等接口。

### Lines 721-768

```cpp
721:                                      ParsedType &ReceiverType);
722: 
723:   ExprResult ActOnSuperMessage(Scope *S, SourceLocation SuperLoc, Selector Sel,
724:                                SourceLocation LBracLoc,
725:                                ArrayRef<SourceLocation> SelectorLocs,
726:                                SourceLocation RBracLoc, MultiExprArg Args);
727: 
728:   ExprResult BuildClassMessage(TypeSourceInfo *ReceiverTypeInfo,
729:                                QualType ReceiverType, SourceLocation SuperLoc,
730:                                Selector Sel, ObjCMethodDecl *Method,
731:                                SourceLocation LBracLoc,
732:                                ArrayRef<SourceLocation> SelectorLocs,
733:                                SourceLocation RBracLoc, MultiExprArg Args,
734:                                bool isImplicit = false);
735: 
736:   ExprResult BuildClassMessageImplicit(QualType ReceiverType,
737:                                        bool isSuperReceiver, SourceLocation Loc,
738:                                        Selector Sel, ObjCMethodDecl *Method,
739:                                        MultiExprArg Args);
740: 
741:   ExprResult ActOnClassMessage(Scope *S, ParsedType Receiver, Selector Sel,
742:                                SourceLocation LBracLoc,
743:                                ArrayRef<SourceLocation> SelectorLocs,
744:                                SourceLocation RBracLoc, MultiExprArg Args);
745: 
746:   ExprResult BuildInstanceMessage(Expr *Receiver, QualType ReceiverType,
747:                                   SourceLocation SuperLoc, Selector Sel,
748:                                   ObjCMethodDecl *Method,
749:                                   SourceLocation LBracLoc,
750:                                   ArrayRef<SourceLocation> SelectorLocs,
751:                                   SourceLocation RBracLoc, MultiExprArg Args,
752:                                   bool isImplicit = false);
753: 
754:   ExprResult BuildInstanceMessageImplicit(Expr *Receiver, QualType ReceiverType,
755:                                           SourceLocation Loc, Selector Sel,
756:                                           ObjCMethodDecl *Method,
757:                                           MultiExprArg Args);
758: 
759:   ExprResult ActOnInstanceMessage(Scope *S, Expr *Receiver, Selector Sel,
760:                                   SourceLocation LBracLoc,
761:                                   ArrayRef<SourceLocation> SelectorLocs,
762:                                   SourceLocation RBracLoc, MultiExprArg Args);
763: 
764:   ExprResult BuildObjCBridgedCast(SourceLocation LParenLoc,
765:                                   ObjCBridgeCastKind Kind,
766:                                   SourceLocation BridgeKeywordLoc,
767:                                   TypeSourceInfo *TSInfo, Expr *SubExpr);
768: 
```
- EN: The lines continue connective syntax and structural scaffolding for the surrounding definition.
- 中文: 这些行延续了周围定义所需的连接语法和结构骨架。

### Lines 769-816

```cpp
769:   ExprResult ActOnObjCBridgedCast(Scope *S, SourceLocation LParenLoc,
770:                                   ObjCBridgeCastKind Kind,
771:                                   SourceLocation BridgeKeywordLoc,
772:                                   ParsedType Type, SourceLocation RParenLoc,
773:                                   Expr *SubExpr);
774: 
775:   void CheckTollFreeBridgeCast(QualType castType, Expr *castExpr);
776: 
777:   void CheckObjCBridgeRelatedCast(QualType castType, Expr *castExpr);
778: 
779:   bool CheckTollFreeBridgeStaticCast(QualType castType, Expr *castExpr,
780:                                      CastKind &Kind);
781: 
782:   bool checkObjCBridgeRelatedComponents(SourceLocation Loc, QualType DestType,
783:                                         QualType SrcType,
784:                                         ObjCInterfaceDecl *&RelatedClass,
785:                                         ObjCMethodDecl *&ClassMethod,
786:                                         ObjCMethodDecl *&InstanceMethod,
787:                                         TypedefNameDecl *&TDNDecl, bool CfToNs,
788:                                         bool Diagnose = true);
789: 
790:   bool CheckObjCBridgeRelatedConversions(SourceLocation Loc, QualType DestType,
791:                                          QualType SrcType, Expr *&SrcExpr,
792:                                          bool Diagnose = true);
793: 
794:   /// Private Helper predicate to check for 'self'.
795:   bool isSelfExpr(Expr *RExpr);
796:   bool isSelfExpr(Expr *RExpr, const ObjCMethodDecl *Method);
797: 
798:   ObjCMethodDecl *LookupMethodInQualifiedType(Selector Sel,
799:                                               const ObjCObjectPointerType *OPT,
800:                                               bool IsInstance);
801:   ObjCMethodDecl *LookupMethodInObjectType(Selector Sel, QualType Ty,
802:                                            bool IsInstance);
803: 
804:   bool isKnownName(StringRef name);
805: 
806:   enum ARCConversionResult { ACR_okay, ACR_unbridged, ACR_error };
807: 
808:   /// Checks for invalid conversions and casts between
809:   /// retainable pointers and other pointer kinds for ARC and Weak.
810:   ARCConversionResult CheckObjCConversion(SourceRange castRange,
811:                                           QualType castType, Expr *&op,
812:                                           CheckedConversionKind CCK,
813:                                           bool Diagnose = true,
814:                                           bool DiagnoseCFAudited = false,
815:                                           BinaryOperatorKind Opc = BO_PtrMemD,
816:                                           bool IsReinterpretCast = false);
```
- EN: It introduces enum-based state or option sets such as `ARCConversionResult`. It exposes API surface such as `CheckTollFreeBridgeCast`, `CheckObjCBridgeRelatedCast`, `isSelfExpr`, `isKnownName`.
- 中文: 它引入了 `ARCConversionResult` 等基于枚举的状态或选项集合。 它暴露了 `CheckTollFreeBridgeCast`, `CheckObjCBridgeRelatedCast`, `isSelfExpr`, `isKnownName` 等接口。

### Lines 817-864

```cpp
817: 
818:   Expr *stripARCUnbridgedCast(Expr *e);
819:   void diagnoseARCUnbridgedCast(Expr *e);
820: 
821:   bool CheckObjCARCUnavailableWeakConversion(QualType castType,
822:                                              QualType ExprType);
823: 
824:   /// CheckMessageArgumentTypes - Check types in an Obj-C message send.
825:   /// \param Method - May be null.
826:   /// \param [out] ReturnType - The return type of the send.
827:   /// \return true iff there were any incompatible types.
828:   bool CheckMessageArgumentTypes(const Expr *Receiver, QualType ReceiverType,
829:                                  MultiExprArg Args, Selector Sel,
830:                                  ArrayRef<SourceLocation> SelectorLocs,
831:                                  ObjCMethodDecl *Method, bool isClassMessage,
832:                                  bool isSuperMessage, SourceLocation lbrac,
833:                                  SourceLocation rbrac, SourceRange RecRange,
834:                                  QualType &ReturnType, ExprValueKind &VK);
835: 
836:   /// Determine the result of a message send expression based on
837:   /// the type of the receiver, the method expected to receive the message,
838:   /// and the form of the message send.
839:   QualType getMessageSendResultType(const Expr *Receiver, QualType ReceiverType,
840:                                     ObjCMethodDecl *Method, bool isClassMessage,
841:                                     bool isSuperMessage);
842: 
843:   /// If the given expression involves a message send to a method
844:   /// with a related result type, emit a note describing what happened.
845:   void EmitRelatedResultTypeNote(const Expr *E);
846: 
847:   /// Given that we had incompatible pointer types in a return
848:   /// statement, check whether we're in a method with a related result
849:   /// type, and if so, emit a note describing what happened.
850:   void EmitRelatedResultTypeNoteForReturn(QualType destType);
851: 
852:   /// LookupInstanceMethodInGlobalPool - Returns the method and warns if
853:   /// there are multiple signatures.
854:   ObjCMethodDecl *
855:   LookupInstanceMethodInGlobalPool(Selector Sel, SourceRange R,
856:                                    bool receiverIdOrClass = false) {
857:     return LookupMethodInGlobalPool(Sel, R, receiverIdOrClass,
858:                                     /*instance*/ true);
859:   }
860: 
861:   /// LookupFactoryMethodInGlobalPool - Returns the method and warns if
862:   /// there are multiple signatures.
863:   ObjCMethodDecl *
864:   LookupFactoryMethodInGlobalPool(Selector Sel, SourceRange R,
```
- EN: It exposes API surface such as `stripARCUnbridgedCast`, `diagnoseARCUnbridgedCast`, `EmitRelatedResultTypeNote`, `EmitRelatedResultTypeNoteForReturn`.
- 中文: 它暴露了 `stripARCUnbridgedCast`, `diagnoseARCUnbridgedCast`, `EmitRelatedResultTypeNote`, `EmitRelatedResultTypeNoteForReturn` 等接口。

### Lines 865-912

```cpp
865:                                   bool receiverIdOrClass = false) {
866:     return LookupMethodInGlobalPool(Sel, R, receiverIdOrClass,
867:                                     /*instance*/ false);
868:   }
869: 
870:   /// The parser has read a name in, and Sema has detected that we're currently
871:   /// inside an ObjC method. Perform some additional checks and determine if we
872:   /// should form a reference to an ivar.
873:   ///
874:   /// Ideally, most of this would be done by lookup, but there's
875:   /// actually quite a lot of extra work involved.
876:   DeclResult LookupIvarInObjCMethod(LookupResult &Lookup, Scope *S,
877:                                     IdentifierInfo *II);
878: 
879:   /// The parser has read a name in, and Sema has detected that we're currently
880:   /// inside an ObjC method. Perform some additional checks and determine if we
881:   /// should form a reference to an ivar. If so, build an expression referencing
882:   /// that ivar.
883:   ExprResult LookupInObjCMethod(LookupResult &LookUp, Scope *S,
884:                                 IdentifierInfo *II,
885:                                 bool AllowBuiltinCreation = false);
886: 
887:   ExprResult BuildIvarRefExpr(Scope *S, SourceLocation Loc, ObjCIvarDecl *IV);
888: 
889:   /// FindCompositeObjCPointerType - Helper method to find composite type of
890:   /// two objective-c pointer types of the two input expressions.
891:   QualType FindCompositeObjCPointerType(ExprResult &LHS, ExprResult &RHS,
892:                                         SourceLocation QuestionLoc);
893: 
894:   bool CheckConversionToObjCLiteral(QualType DstType, Expr *&SrcExpr,
895:                                     bool Diagnose = true);
896: 
897:   /// ActOnObjCBoolLiteral - Parse {__objc_yes,__objc_no} literals.
898:   ExprResult ActOnObjCBoolLiteral(SourceLocation OpLoc, tok::TokenKind Kind);
899: 
900:   ExprResult
901:   ActOnObjCAvailabilityCheckExpr(llvm::ArrayRef<AvailabilitySpec> AvailSpecs,
902:                                  SourceLocation AtLoc, SourceLocation RParen);
903: 
904:   /// Prepare a conversion of the given expression to an ObjC object
905:   /// pointer type.
906:   CastKind PrepareCastToObjCObjectPointer(ExprResult &E);
907: 
908:   // Note that LK_String is intentionally after the other literals, as
909:   // this is used for diagnostics logic.
910:   enum ObjCLiteralKind {
911:     LK_Array,
912:     LK_Dictionary,
```
- EN: It introduces enum-based state or option sets such as `ObjCLiteralKind`. It exposes API surface such as `BuildIvarRefExpr`, `ActOnObjCBoolLiteral`, `PrepareCastToObjCObjectPointer`.
- 中文: 它引入了 `ObjCLiteralKind` 等基于枚举的状态或选项集合。 它暴露了 `BuildIvarRefExpr`, `ActOnObjCBoolLiteral`, `PrepareCastToObjCObjectPointer` 等接口。

### Lines 913-960

```cpp
913:     LK_Numeric,
914:     LK_Boxed,
915:     LK_String,
916:     LK_Block,
917:     LK_None
918:   };
919:   ObjCLiteralKind CheckLiteralKind(Expr *FromE);
920: 
921:   ///@}
922: 
923:   //
924:   //
925:   // -------------------------------------------------------------------------
926:   //
927:   //
928: 
929:   /// \name ObjC @property and @synthesize
930:   /// Implementations are in SemaObjCProperty.cpp
931:   ///@{
932: 
933: public:
934:   /// Ensure attributes are consistent with type.
935:   /// \param [in, out] Attributes The attributes to check; they will
936:   /// be modified to be consistent with \p PropertyTy.
937:   void CheckObjCPropertyAttributes(Decl *PropertyPtrTy, SourceLocation Loc,
938:                                    unsigned &Attributes,
939:                                    bool propertyInPrimaryClass);
940: 
941:   /// Process the specified property declaration and create decls for the
942:   /// setters and getters as needed.
943:   /// \param property The property declaration being processed
944:   void ProcessPropertyDecl(ObjCPropertyDecl *property);
945: 
946:   Decl *ActOnProperty(Scope *S, SourceLocation AtLoc, SourceLocation LParenLoc,
947:                       FieldDeclarator &FD, ObjCDeclSpec &ODS,
948:                       Selector GetterSel, Selector SetterSel,
949:                       tok::ObjCKeywordKind MethodImplKind,
950:                       DeclContext *lexicalDC = nullptr);
951: 
952:   Decl *ActOnPropertyImplDecl(Scope *S, SourceLocation AtLoc,
953:                               SourceLocation PropertyLoc, bool ImplKind,
954:                               IdentifierInfo *PropertyId,
955:                               IdentifierInfo *PropertyIvar,
956:                               SourceLocation PropertyIvarLoc,
957:                               ObjCPropertyQueryKind QueryKind);
958: 
959:   /// Called by ActOnProperty to handle \@property declarations in
960:   /// class extensions.
```
- EN: It exposes API surface such as `CheckLiteralKind`, `ProcessPropertyDecl`.
- 中文: 它暴露了 `CheckLiteralKind`, `ProcessPropertyDecl` 等接口。

### Lines 961-1008

```cpp
 961:   ObjCPropertyDecl *HandlePropertyInClassExtension(
 962:       Scope *S, SourceLocation AtLoc, SourceLocation LParenLoc,
 963:       FieldDeclarator &FD, Selector GetterSel, SourceLocation GetterNameLoc,
 964:       Selector SetterSel, SourceLocation SetterNameLoc, const bool isReadWrite,
 965:       unsigned &Attributes, const unsigned AttributesAsWritten, QualType T,
 966:       TypeSourceInfo *TSI, tok::ObjCKeywordKind MethodImplKind);
 967: 
 968:   /// Called by ActOnProperty and HandlePropertyInClassExtension to
 969:   /// handle creating the ObjcPropertyDecl for a category or \@interface.
 970:   ObjCPropertyDecl *
 971:   CreatePropertyDecl(Scope *S, ObjCContainerDecl *CDecl, SourceLocation AtLoc,
 972:                      SourceLocation LParenLoc, FieldDeclarator &FD,
 973:                      Selector GetterSel, SourceLocation GetterNameLoc,
 974:                      Selector SetterSel, SourceLocation SetterNameLoc,
 975:                      const bool isReadWrite, const unsigned Attributes,
 976:                      const unsigned AttributesAsWritten, QualType T,
 977:                      TypeSourceInfo *TSI, tok::ObjCKeywordKind MethodImplKind,
 978:                      DeclContext *lexicalDC = nullptr);
 979: 
 980:   void DiagnosePropertyMismatch(ObjCPropertyDecl *Property,
 981:                                 ObjCPropertyDecl *SuperProperty,
 982:                                 const IdentifierInfo *Name,
 983:                                 bool OverridingProtocolProperty);
 984: 
 985:   bool DiagnosePropertyAccessorMismatch(ObjCPropertyDecl *PD,
 986:                                         ObjCMethodDecl *Getter,
 987:                                         SourceLocation Loc);
 988: 
 989:   /// DiagnoseUnimplementedProperties - This routine warns on those properties
 990:   /// which must be implemented by this implementation.
 991:   void DiagnoseUnimplementedProperties(Scope *S, ObjCImplDecl *IMPDecl,
 992:                                        ObjCContainerDecl *CDecl,
 993:                                        bool SynthesizeProperties);
 994: 
 995:   /// Diagnose any null-resettable synthesized setters.
 996:   void diagnoseNullResettableSynthesizedSetters(const ObjCImplDecl *impDecl);
 997: 
 998:   /// DefaultSynthesizeProperties - This routine default synthesizes all
 999:   /// properties which must be synthesized in the class's \@implementation.
1000:   void DefaultSynthesizeProperties(Scope *S, ObjCImplDecl *IMPDecl,
1001:                                    ObjCInterfaceDecl *IDecl,
1002:                                    SourceLocation AtEnd);
1003:   void DefaultSynthesizeProperties(Scope *S, Decl *D, SourceLocation AtEnd);
1004: 
1005:   /// IvarBacksCurrentMethodAccessor - This routine returns 'true' if 'IV' is
1006:   /// an ivar synthesized for 'Method' and 'Method' is a property accessor
1007:   /// declared in class 'IFace'.
1008:   bool IvarBacksCurrentMethodAccessor(ObjCInterfaceDecl *IFace,
```
- EN: It exposes API surface such as `diagnoseNullResettableSynthesizedSetters`, `DefaultSynthesizeProperties`.
- 中文: 它暴露了 `diagnoseNullResettableSynthesizedSetters`, `DefaultSynthesizeProperties` 等接口。

### Lines 1009-1056

```cpp
1009:                                       ObjCMethodDecl *Method, ObjCIvarDecl *IV);
1010: 
1011:   void DiagnoseOwningPropertyGetterSynthesis(const ObjCImplementationDecl *D);
1012: 
1013:   void
1014:   DiagnoseMissingDesignatedInitOverrides(const ObjCImplementationDecl *ImplD,
1015:                                          const ObjCInterfaceDecl *IFD);
1016: 
1017:   /// AtomicPropertySetterGetterRules - This routine enforces the rule (via
1018:   /// warning) when atomic property has one but not the other user-declared
1019:   /// setter or getter.
1020:   void AtomicPropertySetterGetterRules(ObjCImplDecl *IMPDecl,
1021:                                        ObjCInterfaceDecl *IDecl);
1022: 
1023:   ///@}
1024: 
1025:   //
1026:   //
1027:   // -------------------------------------------------------------------------
1028:   //
1029:   //
1030: 
1031:   /// \name ObjC Attributes
1032:   /// Implementations are in SemaObjC.cpp
1033:   ///@{
1034: 
1035:   bool isNSStringType(QualType T, bool AllowNSAttributedString = false);
1036:   bool isCFStringType(QualType T);
1037: 
1038:   void handleIBOutlet(Decl *D, const ParsedAttr &AL);
1039:   void handleIBOutletCollection(Decl *D, const ParsedAttr &AL);
1040: 
1041:   void handleSuppresProtocolAttr(Decl *D, const ParsedAttr &AL);
1042:   void handleDirectAttr(Decl *D, const ParsedAttr &AL);
1043:   void handleDirectMembersAttr(Decl *D, const ParsedAttr &AL);
1044:   void handleMethodFamilyAttr(Decl *D, const ParsedAttr &AL);
1045:   void handleNSObject(Decl *D, const ParsedAttr &AL);
1046:   void handleIndependentClass(Decl *D, const ParsedAttr &AL);
1047:   void handleBlocksAttr(Decl *D, const ParsedAttr &AL);
1048:   void handleReturnsInnerPointerAttr(Decl *D, const ParsedAttr &Attrs);
1049:   void handleXReturnsXRetainedAttr(Decl *D, const ParsedAttr &AL);
1050:   void handleRequiresSuperAttr(Decl *D, const ParsedAttr &Attrs);
1051:   void handleNSErrorDomain(Decl *D, const ParsedAttr &Attr);
1052:   void handleBridgeAttr(Decl *D, const ParsedAttr &AL);
1053:   void handleBridgeMutableAttr(Decl *D, const ParsedAttr &AL);
1054:   void handleBridgeRelatedAttr(Decl *D, const ParsedAttr &AL);
1055:   void handleDesignatedInitializer(Decl *D, const ParsedAttr &AL);
1056:   void handleRuntimeName(Decl *D, const ParsedAttr &AL);
```
- EN: It exposes API surface such as `DiagnoseOwningPropertyGetterSynthesis`, `isNSStringType`, `isCFStringType`, `handleIBOutlet`.
- 中文: 它暴露了 `DiagnoseOwningPropertyGetterSynthesis`, `isNSStringType`, `isCFStringType`, `handleIBOutlet` 等接口。

### Lines 1057-1078

```cpp
1057:   void handleBoxable(Decl *D, const ParsedAttr &AL);
1058:   void handleOwnershipAttr(Decl *D, const ParsedAttr &AL);
1059:   void handlePreciseLifetimeAttr(Decl *D, const ParsedAttr &AL);
1060:   void handleExternallyRetainedAttr(Decl *D, const ParsedAttr &AL);
1061: 
1062:   void AddXConsumedAttr(Decl *D, const AttributeCommonInfo &CI,
1063:                         Sema::RetainOwnershipKind K,
1064:                         bool IsTemplateInstantiation);
1065: 
1066:   /// \return whether the parameter is a pointer to OSObject pointer.
1067:   bool isValidOSObjectOutParameter(const Decl *D);
1068:   bool checkNSReturnsRetainedReturnType(SourceLocation loc, QualType type);
1069: 
1070:   Sema::RetainOwnershipKind
1071:   parsedAttrToRetainOwnershipKind(const ParsedAttr &AL);
1072: 
1073:   ///@}
1074: };
1075: 
1076: } // namespace clang
1077: 
1078: #endif // LLVM_CLANG_SEMA_SEMAOBJC_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `clang`. It exposes API surface such as `handleBoxable`, `handleOwnershipAttr`, `handlePreciseLifetimeAttr`, `handleExternallyRetainedAttr`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `clang` 的命名空间作用域。 它暴露了 `handleBoxable`, `handleOwnershipAttr`, `handlePreciseLifetimeAttr`, `handleExternallyRetainedAttr` 等接口。

## Key Concepts / 关键概念

- `AttributeCommonInfo`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `AvailabilitySpec`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CheckedConversionKind`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `DeclGroupRef`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `LookupResult`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ObjCDictionaryElement`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ParsedAttr`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ParsedAttributesView`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/AST/ASTFwd.h`, `clang/AST/DeclObjC.h`, `clang/AST/NSAPI.h`, `clang/AST/OperationKinds.h`, `clang/AST/Type.h`, `clang/Basic/IdentifierTable.h`, `clang/Basic/LLVM.h`, `clang/Basic/SourceLocation.h`, `clang/Basic/Specifiers.h`, `clang/Basic/TokenKinds.h`, `clang/Sema/DeclSpec.h`, `clang/Sema/ObjCMethodList.h`, `clang/Sema/Ownership.h`, `clang/Sema/Redeclaration.h`, `clang/Sema/Sema.h`
- Forward declarations / 前向声明: `AttributeCommonInfo`, `AvailabilitySpec`, `CheckedConversionKind`, `DeclGroupRef`, `LookupResult`, `ObjCDictionaryElement`, `ParsedAttr`, `ParsedAttributesView`, `Scope`, `SkipBodyInfo`
- Namespace context / 命名空间上下文: `clang`
- Macro-style dependencies / 宏式依赖: None / 无
