# DelayedDiagnostic.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Sema/DelayedDiagnostic.h`
- Repository: `llvm-project`
- Purpose (EN): Delayed declarator diagnostics.
- 用途（中文）: 该文件为 Sema 子系统中的 Delayed Diagnostic 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

```cpp
 1: //===- DelayedDiagnostic.h - Delayed declarator diagnostics -----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: /// \file
10: /// Defines the classes clang::DelayedDiagnostic and
11: /// clang::AccessedEntity.
12: ///
13: /// DelayedDiangostic is used to record diagnostics that are being
14: /// conditionally produced during declarator parsing.  Certain kinds of
15: /// diagnostics -- notably deprecation and access control -- are suppressed
16: /// based on semantic properties of the parsed declaration that aren't known
17: /// until it is fully parsed.
18: //
19: //===----------------------------------------------------------------------===//
20: 
21: #ifndef LLVM_CLANG_SEMA_DELAYEDDIAGNOSTIC_H
22: #define LLVM_CLANG_SEMA_DELAYEDDIAGNOSTIC_H
23: 
24: #include "clang/AST/DeclAccessPair.h"
25: #include "clang/AST/DeclBase.h"
26: #include "clang/AST/DeclCXX.h"
27: #include "clang/AST/Type.h"
28: #include "clang/Basic/LLVM.h"
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/AST/DeclAccessPair.h`, `clang/AST/DeclBase.h`, `clang/AST/DeclCXX.h` and 2 more.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/AST/DeclAccessPair.h`, `clang/AST/DeclBase.h`, `clang/AST/DeclCXX.h` 以及另外 2 项依赖。

### Lines 29-56

```cpp
29: #include "clang/Basic/PartialDiagnostic.h"
30: #include "clang/Basic/SourceLocation.h"
31: #include "clang/Basic/Specifiers.h"
32: #include "clang/Sema/Sema.h"
33: #include "llvm/ADT/ArrayRef.h"
34: #include "llvm/ADT/SmallVector.h"
35: #include "llvm/ADT/StringRef.h"
36: #include "llvm/Support/Casting.h"
37: #include <cassert>
38: #include <cstddef>
39: #include <utility>
40: 
41: namespace clang {
42: 
43: class ObjCInterfaceDecl;
44: class ObjCPropertyDecl;
45: 
46: namespace sema {
47: 
48: /// A declaration being accessed, together with information about how
49: /// it was accessed.
50: class AccessedEntity {
51: public:
52:   /// A member declaration found through lookup.  The target is the
53:   /// member.
54:   enum MemberNonce { Member };
55: 
56:   /// A hierarchy (base-to-derived or derived-to-base) conversion.
```
- EN: This block imports dependencies such as `clang/Basic/PartialDiagnostic.h`, `clang/Basic/SourceLocation.h`, `clang/Basic/Specifiers.h` and 8 more. It opens, closes, or documents namespace scope for `clang`, `sema`. Key type declarations here include `ObjCInterfaceDecl`, `ObjCPropertyDecl`, `AccessedEntity`.
- 中文: 这一块引入了 `clang/Basic/PartialDiagnostic.h`, `clang/Basic/SourceLocation.h`, `clang/Basic/Specifiers.h` 以及另外 8 项依赖。 它打开、关闭或说明了 `clang`, `sema` 的命名空间作用域。 这里的重要类型声明包括 `ObjCInterfaceDecl`, `ObjCPropertyDecl`, `AccessedEntity`。

### Lines 57-84

```cpp
57:   /// The target is the base class.
58:   enum BaseNonce { Base };
59: 
60:   AccessedEntity(PartialDiagnostic::DiagStorageAllocator &Allocator,
61:                  MemberNonce _, CXXRecordDecl *NamingClass,
62:                  DeclAccessPair FoundDecl, QualType BaseObjectType)
63:       : Access(FoundDecl.getAccess()), IsMember(true),
64:         Target(FoundDecl.getDecl()), NamingClass(NamingClass),
65:         BaseObjectType(BaseObjectType), Diag(0, Allocator) {}
66: 
67:   AccessedEntity(PartialDiagnostic::DiagStorageAllocator &Allocator,
68:                  BaseNonce _, CXXRecordDecl *BaseClass,
69:                  CXXRecordDecl *DerivedClass, AccessSpecifier Access)
70:       : Access(Access), IsMember(false), Target(BaseClass),
71:         NamingClass(DerivedClass), Diag(0, Allocator) {}
72: 
73:   bool isMemberAccess() const { return IsMember; }
74: 
75:   bool isQuiet() const { return Diag.getDiagID() == 0; }
76: 
77:   AccessSpecifier getAccess() const { return AccessSpecifier(Access); }
78: 
79:   // These apply to member decls...
80:   NamedDecl *getTargetDecl() const { return Target; }
81:   CXXRecordDecl *getNamingClass() const { return NamingClass; }
82: 
83:   // ...and these apply to hierarchy conversions.
84:   CXXRecordDecl *getBaseClass() const {
```
- EN: It introduces enum-based state or option sets such as `BaseNonce`. It exposes API surface such as `BaseObjectType`, `NamingClass`, `isMemberAccess`, `isQuiet`.
- 中文: 它引入了 `BaseNonce` 等基于枚举的状态或选项集合。 它暴露了 `BaseObjectType`, `NamingClass`, `isMemberAccess`, `isQuiet` 等接口。

### Lines 85-112

```cpp
 85:     assert(!IsMember); return cast<CXXRecordDecl>(Target);
 86:   }
 87:   CXXRecordDecl *getDerivedClass() const { return NamingClass; }
 88: 
 89:   /// Retrieves the base object type, important when accessing
 90:   /// an instance member.
 91:   QualType getBaseObjectType() const { return BaseObjectType; }
 92: 
 93:   /// Sets a diagnostic to be performed.  The diagnostic is given
 94:   /// four (additional) arguments:
 95:   ///   %0 - 0 if the entity was private, 1 if protected
 96:   ///   %1 - the DeclarationName of the entity
 97:   ///   %2 - the TypeDecl type of the naming class
 98:   ///   %3 - the TypeDecl type of the declaring class
 99:   void setDiag(const PartialDiagnostic &PDiag) {
100:     assert(isQuiet() && "partial diagnostic already defined");
101:     Diag = PDiag;
102:   }
103:   PartialDiagnostic &setDiag(unsigned DiagID) {
104:     assert(isQuiet() && "partial diagnostic already defined");
105:     assert(DiagID && "creating null diagnostic");
106:     Diag.Reset(DiagID);
107:     return Diag;
108:   }
109:   const PartialDiagnostic &getDiag() const {
110:     return Diag;
111:   }
112: 
```
- EN: It exposes API surface such as `assert`, `getDerivedClass`, `getBaseObjectType`, `setDiag`.
- 中文: 它暴露了 `assert`, `getDerivedClass`, `getBaseObjectType`, `setDiag` 等接口。

### Lines 113-140

```cpp
113: private:
114:   LLVM_PREFERRED_TYPE(AccessSpecifier)
115:   unsigned Access : 2;
116:   LLVM_PREFERRED_TYPE(bool)
117:   unsigned IsMember : 1;
118:   NamedDecl *Target;
119:   CXXRecordDecl *NamingClass;
120:   QualType BaseObjectType;
121:   PartialDiagnostic Diag;
122: };
123: 
124: /// A diagnostic message which has been conditionally emitted pending
125: /// the complete parsing of the current declaration.
126: class DelayedDiagnostic {
127: public:
128:   enum DDKind : unsigned char { Availability, Access, ForbiddenType };
129: 
130:   DDKind Kind;
131:   bool Triggered;
132: 
133:   SourceLocation Loc;
134: 
135:   void Destroy();
136: 
137:   static DelayedDiagnostic makeAvailability(AvailabilityResult AR,
138:                                             ArrayRef<SourceLocation> Locs,
139:                                             const NamedDecl *ReferringDecl,
140:                                             const NamedDecl *OffendingDecl,
```
- EN: Key type declarations here include `DelayedDiagnostic`. It introduces enum-based state or option sets such as `DDKind`. It exposes API surface such as `Destroy`.
- 中文: 这里的重要类型声明包括 `DelayedDiagnostic`。 它引入了 `DDKind` 等基于枚举的状态或选项集合。 它暴露了 `Destroy` 等接口。

### Lines 141-168

```cpp
141:                                             const ObjCInterfaceDecl *UnknownObjCClass,
142:                                             const ObjCPropertyDecl  *ObjCProperty,
143:                                             StringRef Msg,
144:                                             bool ObjCPropertyAccess);
145: 
146:   static DelayedDiagnostic makeAccess(SourceLocation Loc,
147:                                       const AccessedEntity &Entity) {
148:     DelayedDiagnostic DD;
149:     DD.Kind = Access;
150:     DD.Triggered = false;
151:     DD.Loc = Loc;
152:     new (&DD.getAccessData()) AccessedEntity(Entity);
153:     return DD;
154:   }
155: 
156:   static DelayedDiagnostic makeForbiddenType(SourceLocation loc,
157:                                              unsigned diagnostic,
158:                                              QualType type,
159:                                              unsigned argument) {
160:     DelayedDiagnostic DD;
161:     DD.Kind = ForbiddenType;
162:     DD.Triggered = false;
163:     DD.Loc = loc;
164:     DD.ForbiddenTypeData.Diagnostic = diagnostic;
165:     DD.ForbiddenTypeData.OperandType = type.getAsOpaquePtr();
166:     DD.ForbiddenTypeData.Argument = argument;
167:     return DD;
168:   }
```
- EN: It exposes API surface such as `new`, `getAsOpaquePtr`.
- 中文: 它暴露了 `new`, `getAsOpaquePtr` 等接口。

### Lines 169-196

```cpp
169: 
170:   AccessedEntity &getAccessData() {
171:     assert(Kind == Access && "Not an access diagnostic.");
172:     return *reinterpret_cast<AccessedEntity*>(AccessData);
173:   }
174:   const AccessedEntity &getAccessData() const {
175:     assert(Kind == Access && "Not an access diagnostic.");
176:     return *reinterpret_cast<const AccessedEntity*>(AccessData);
177:   }
178: 
179:   const NamedDecl *getAvailabilityReferringDecl() const {
180:     assert(Kind == Availability && "Not an availability diagnostic.");
181:     return AvailabilityData.ReferringDecl;
182:   }
183: 
184:   const NamedDecl *getAvailabilityOffendingDecl() const {
185:     return AvailabilityData.OffendingDecl;
186:   }
187: 
188:   StringRef getAvailabilityMessage() const {
189:     assert(Kind == Availability && "Not an availability diagnostic.");
190:     return StringRef(AvailabilityData.Message, AvailabilityData.MessageLen);
191:   }
192: 
193:   ArrayRef<SourceLocation> getAvailabilitySelectorLocs() const {
194:     assert(Kind == Availability && "Not an availability diagnostic.");
195:     return llvm::ArrayRef(AvailabilityData.SelectorLocs,
196:                           AvailabilityData.NumSelectorLocs);
```
- EN: It exposes API surface such as `getAccessData`, `assert`, `getAvailabilityReferringDecl`, `getAvailabilityOffendingDecl`.
- 中文: 它暴露了 `getAccessData`, `assert`, `getAvailabilityReferringDecl`, `getAvailabilityOffendingDecl` 等接口。

### Lines 197-224

```cpp
197:   }
198: 
199:   AvailabilityResult getAvailabilityResult() const {
200:     assert(Kind == Availability && "Not an availability diagnostic.");
201:     return AvailabilityData.AR;
202:   }
203: 
204:   /// The diagnostic ID to emit.  Used like so:
205:   ///   Diag(diag.Loc, diag.getForbiddenTypeDiagnostic())
206:   ///     << diag.getForbiddenTypeOperand()
207:   ///     << diag.getForbiddenTypeArgument();
208:   unsigned getForbiddenTypeDiagnostic() const {
209:     assert(Kind == ForbiddenType && "not a forbidden-type diagnostic");
210:     return ForbiddenTypeData.Diagnostic;
211:   }
212: 
213:   unsigned getForbiddenTypeArgument() const {
214:     assert(Kind == ForbiddenType && "not a forbidden-type diagnostic");
215:     return ForbiddenTypeData.Argument;
216:   }
217: 
218:   QualType getForbiddenTypeOperand() const {
219:     assert(Kind == ForbiddenType && "not a forbidden-type diagnostic");
220:     return QualType::getFromOpaquePtr(ForbiddenTypeData.OperandType);
221:   }
222: 
223:   const ObjCInterfaceDecl *getUnknownObjCClass() const {
224:     return AvailabilityData.UnknownObjCClass;
```
- EN: It exposes API surface such as `getAvailabilityResult`, `assert`, `getForbiddenTypeDiagnostic`, `getForbiddenTypeArgument`.
- 中文: 它暴露了 `getAvailabilityResult`, `assert`, `getForbiddenTypeDiagnostic`, `getForbiddenTypeArgument` 等接口。

### Lines 225-252

```cpp
225:   }
226: 
227:   const ObjCPropertyDecl *getObjCProperty() const {
228:     return AvailabilityData.ObjCProperty;
229:   }
230: 
231:   bool getObjCPropertyAccess() const {
232:     return AvailabilityData.ObjCPropertyAccess;
233:   }
234: 
235: private:
236:   struct AD {
237:     const NamedDecl *ReferringDecl;
238:     const NamedDecl *OffendingDecl;
239:     const ObjCInterfaceDecl *UnknownObjCClass;
240:     const ObjCPropertyDecl  *ObjCProperty;
241:     const char *Message;
242:     size_t MessageLen;
243:     SourceLocation *SelectorLocs;
244:     size_t NumSelectorLocs;
245:     AvailabilityResult AR;
246:     bool ObjCPropertyAccess;
247:   };
248: 
249:   struct FTD {
250:     unsigned Diagnostic;
251:     unsigned Argument;
252:     void *OperandType;
```
- EN: Key type declarations here include `AD`, `FTD`. It exposes API surface such as `getObjCProperty`, `getObjCPropertyAccess`.
- 中文: 这里的重要类型声明包括 `AD`, `FTD`。 它暴露了 `getObjCProperty`, `getObjCPropertyAccess` 等接口。

### Lines 253-280

```cpp
253:   };
254: 
255:   union {
256:     struct AD AvailabilityData;
257:     struct FTD ForbiddenTypeData;
258: 
259:     /// Access control.
260:     char AccessData[sizeof(AccessedEntity)];
261:   };
262: };
263: 
264: /// A collection of diagnostics which were delayed.
265: class DelayedDiagnosticPool {
266:   const DelayedDiagnosticPool *Parent;
267:   SmallVector<DelayedDiagnostic, 4> Diagnostics;
268: 
269: public:
270:   DelayedDiagnosticPool(const DelayedDiagnosticPool *parent) : Parent(parent) {}
271: 
272:   DelayedDiagnosticPool(const DelayedDiagnosticPool &) = delete;
273:   DelayedDiagnosticPool &operator=(const DelayedDiagnosticPool &) = delete;
274: 
275:   DelayedDiagnosticPool(DelayedDiagnosticPool &&Other)
276:       : Parent(Other.Parent), Diagnostics(std::move(Other.Diagnostics)) {
277:     Other.Diagnostics.clear();
278:   }
279: 
280:   DelayedDiagnosticPool &operator=(DelayedDiagnosticPool &&Other) {
```
- EN: Key type declarations here include `AD`, `FTD`, `DelayedDiagnosticPool`. It exposes API surface such as `DelayedDiagnosticPool`, `Parent`, `clear`.
- 中文: 这里的重要类型声明包括 `AD`, `FTD`, `DelayedDiagnosticPool`。 它暴露了 `DelayedDiagnosticPool`, `Parent`, `clear` 等接口。

### Lines 281-308

```cpp
281:     Parent = Other.Parent;
282:     Diagnostics = std::move(Other.Diagnostics);
283:     Other.Diagnostics.clear();
284:     return *this;
285:   }
286: 
287:   ~DelayedDiagnosticPool() {
288:     for (SmallVectorImpl<DelayedDiagnostic>::iterator
289:            i = Diagnostics.begin(), e = Diagnostics.end(); i != e; ++i)
290:       i->Destroy();
291:   }
292: 
293:   const DelayedDiagnosticPool *getParent() const { return Parent; }
294: 
295:   /// Does this pool, or any of its ancestors, contain any diagnostics?
296:   bool empty() const {
297:     return (Diagnostics.empty() && (!Parent || Parent->empty()));
298:   }
299: 
300:   /// Add a diagnostic to this pool.
301:   void add(const DelayedDiagnostic &diag) {
302:     Diagnostics.push_back(diag);
303:   }
304: 
305:   /// Steal the diagnostics from the given pool.
306:   void steal(DelayedDiagnosticPool &pool) {
307:     if (pool.Diagnostics.empty()) return;
308: 
```
- EN: It exposes API surface such as `move`, `clear`, `~DelayedDiagnosticPool`, `begin`.
- 中文: 它暴露了 `move`, `clear`, `~DelayedDiagnosticPool`, `begin` 等接口。

### Lines 309-334

```cpp
309:     if (Diagnostics.empty()) {
310:       Diagnostics = std::move(pool.Diagnostics);
311:     } else {
312:       Diagnostics.append(pool.pool_begin(), pool.pool_end());
313:     }
314:     pool.Diagnostics.clear();
315:   }
316: 
317:   using pool_iterator = SmallVectorImpl<DelayedDiagnostic>::const_iterator;
318: 
319:   pool_iterator pool_begin() const { return Diagnostics.begin(); }
320:   pool_iterator pool_end() const { return Diagnostics.end(); }
321:   bool pool_empty() const { return Diagnostics.empty(); }
322: };
323: 
324: } // namespace clang
325: 
326: /// Add a diagnostic to the current delay pool.
327: inline void Sema::DelayedDiagnostics::add(const sema::DelayedDiagnostic &diag) {
328:   assert(shouldDelayDiagnostics() && "trying to delay without pool");
329:   CurPool->add(diag);
330: }
331: 
332: } // namespace clang
333: 
334: #endif // LLVM_CLANG_SEMA_DELAYEDDIAGNOSTIC_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `clang`. It defines convenient aliases such as `pool_iterator`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `clang` 的命名空间作用域。 它定义了 `pool_iterator` 等便捷别名。

## Key Concepts / 关键概念

- `ObjCInterfaceDecl`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ObjCPropertyDecl`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `AccessedEntity`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `MemberNonce`: An enumeration that captures a bounded set of modes, states, or categories. / 用于表达有限模式、状态或类别的枚举。
- `BaseNonce`: An enumeration that captures a bounded set of modes, states, or categories. / 用于表达有限模式、状态或类别的枚举。
- `DelayedDiagnostic`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `DDKind`: An enumeration that captures a bounded set of modes, states, or categories. / 用于表达有限模式、状态或类别的枚举。
- `AD`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/AST/DeclAccessPair.h`, `clang/AST/DeclBase.h`, `clang/AST/DeclCXX.h`, `clang/AST/Type.h`, `clang/Basic/LLVM.h`, `clang/Basic/PartialDiagnostic.h`, `clang/Basic/SourceLocation.h`, `clang/Basic/Specifiers.h`, `clang/Sema/Sema.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Casting.h`, `cassert`, `cstddef`
- Forward declarations / 前向声明: `ObjCInterfaceDecl`, `ObjCPropertyDecl`
- Namespace context / 命名空间上下文: `clang`, `sema`
- Macro-style dependencies / 宏式依赖: `LLVM_PREFERRED_TYPE`
