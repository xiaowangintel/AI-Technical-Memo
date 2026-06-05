# SemaCUDA.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Sema/SemaCUDA.h`
- Repository: `llvm-project`
- Purpose (EN): This file declares semantic analysis for CUDA constructs.
- 用途（中文）: 该文件为 Sema 子系统中的 Sema CUDA 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27

```cpp
 1: //===----- SemaCUDA.h ----- Semantic Analysis for CUDA constructs ---------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: /// \file
 9: /// This file declares semantic analysis for CUDA constructs.
10: ///
11: //===----------------------------------------------------------------------===//
12: 
13: #ifndef LLVM_CLANG_SEMA_SEMACUDA_H
14: #define LLVM_CLANG_SEMA_SEMACUDA_H
15: 
16: #include "clang/AST/ASTFwd.h"
17: #include "clang/AST/DeclAccessPair.h"
18: #include "clang/AST/Redeclarable.h"
19: #include "clang/Basic/Cuda.h"
20: #include "clang/Basic/LLVM.h"
21: #include "clang/Basic/SourceLocation.h"
22: #include "clang/Sema/Lookup.h"
23: #include "clang/Sema/Ownership.h"
24: #include "clang/Sema/SemaBase.h"
25: #include "llvm/ADT/DenseMap.h"
26: #include "llvm/ADT/DenseMapInfo.h"
27: #include "llvm/ADT/DenseSet.h"
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/AST/ASTFwd.h`, `clang/AST/DeclAccessPair.h`, `clang/AST/Redeclarable.h` and 9 more.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/AST/ASTFwd.h`, `clang/AST/DeclAccessPair.h`, `clang/AST/Redeclarable.h` 以及另外 9 项依赖。

### Lines 28-54

```cpp
28: #include "llvm/ADT/Hashing.h"
29: #include "llvm/ADT/SmallVector.h"
30: #include <string>
31: #include <utility>
32: 
33: namespace clang {
34: namespace sema {
35: class Capture;
36: } // namespace sema
37: 
38: class ASTReader;
39: class ASTWriter;
40: enum class CUDAFunctionTarget;
41: enum class CXXSpecialMemberKind;
42: class ParsedAttributesView;
43: class Scope;
44: 
45: class SemaCUDA : public SemaBase {
46: public:
47:   SemaCUDA(Sema &S);
48: 
49:   /// Increments our count of the number of times we've seen a pragma forcing
50:   /// functions to be __host__ __device__.  So long as this count is greater
51:   /// than zero, all functions encountered will be __host__ __device__.
52:   void PushForceHostDevice();
53: 
54:   /// Decrements our count of the number of times we've seen a pragma forcing
```
- EN: This block imports dependencies such as `llvm/ADT/Hashing.h`, `llvm/ADT/SmallVector.h`, `string` and 1 more. It opens, closes, or documents namespace scope for `clang`, `sema`. Key type declarations here include `Capture`, `ASTReader`, `ASTWriter`, `CUDAFunctionTarget`.
- 中文: 这一块引入了 `llvm/ADT/Hashing.h`, `llvm/ADT/SmallVector.h`, `string` 以及另外 1 项依赖。 它打开、关闭或说明了 `clang`, `sema` 的命名空间作用域。 这里的重要类型声明包括 `Capture`, `ASTReader`, `ASTWriter`, `CUDAFunctionTarget`。

### Lines 55-81

```cpp
55:   /// functions to be __host__ __device__.  Returns false if the count is 0
56:   /// before incrementing, so you can emit an error.
57:   bool PopForceHostDevice();
58: 
59:   ExprResult ActOnExecConfigExpr(Scope *S, SourceLocation LLLLoc,
60:                                  MultiExprArg ExecConfig,
61:                                  SourceLocation GGGLoc);
62: 
63:   /// A pair of a canonical FunctionDecl and a SourceLocation.  When used as the
64:   /// key in a hashtable, both the FD and location are hashed.
65:   struct FunctionDeclAndLoc {
66:     CanonicalDeclPtr<const FunctionDecl> FD;
67:     SourceLocation Loc;
68:   };
69: 
70:   /// FunctionDecls and SourceLocations for which CheckCall has emitted a
71:   /// (maybe deferred) "bad call" diagnostic.  We use this to avoid emitting the
72:   /// same deferred diag twice.
73:   llvm::DenseSet<FunctionDeclAndLoc> LocsWithCUDACallDiags;
74: 
75:   /// An inverse call graph, mapping known-emitted functions to their
76:   /// known-emitted callers (plus the location of the call).
77:   ///
78:   /// Functions that we can tell a priori must be emitted aren't added to this
79:   /// map. A function may have multiple callers that force it into device
80:   /// context, so we store all of them to produce complete diagnostics.
81:   llvm::DenseMap</* Callee = */ CanonicalDeclPtr<const FunctionDecl>,
```
- EN: Key type declarations here include `FunctionDeclAndLoc`. It exposes API surface such as `PopForceHostDevice`.
- 中文: 这里的重要类型声明包括 `FunctionDeclAndLoc`。 它暴露了 `PopForceHostDevice` 等接口。

### Lines 82-108

```cpp
 82:                  /* Callers = */ llvm::SmallVector<FunctionDeclAndLoc, 1>>
 83:       DeviceKnownEmittedFns;
 84: 
 85:   /// Creates a SemaDiagnosticBuilder that emits the diagnostic if the current
 86:   /// context is "used as device code".
 87:   ///
 88:   /// - If CurContext is a __host__ function, does not emit any diagnostics
 89:   ///   unless \p EmitOnBothSides is true.
 90:   /// - If CurContext is a __device__ or __global__ function, emits the
 91:   ///   diagnostics immediately.
 92:   /// - If CurContext is a __host__ __device__ function and we are compiling for
 93:   ///   the device, creates a diagnostic which is emitted if and when we realize
 94:   ///   that the function will be codegen'ed.
 95:   ///
 96:   /// Example usage:
 97:   ///
 98:   ///  // Variable-length arrays are not allowed in CUDA device code.
 99:   ///  if (DiagIfDeviceCode(Loc, diag::err_cuda_vla) << CurrentTarget())
100:   ///    return ExprError();
101:   ///  // Otherwise, continue parsing as normal.
102:   SemaDiagnosticBuilder DiagIfDeviceCode(SourceLocation Loc, unsigned DiagID);
103: 
104:   /// Creates a SemaDiagnosticBuilder that emits the diagnostic if the current
105:   /// context is "used as host code".
106:   ///
107:   /// Same as DiagIfDeviceCode, with "host" and "device" switched.
108:   SemaDiagnosticBuilder DiagIfHostCode(SourceLocation Loc, unsigned DiagID);
```
- EN: It exposes API surface such as `DiagIfDeviceCode`, `DiagIfHostCode`.
- 中文: 它暴露了 `DiagIfDeviceCode`, `DiagIfHostCode` 等接口。

### Lines 109-135

```cpp
109: 
110:   /// Determines whether the given function is a CUDA device/host/kernel/etc.
111:   /// function.
112:   ///
113:   /// Use this rather than examining the function's attributes yourself -- you
114:   /// will get it wrong.  Returns CUDAFunctionTarget::Host if D is null.
115:   CUDAFunctionTarget IdentifyTarget(const FunctionDecl *D,
116:                                     bool IgnoreImplicitHDAttr = false);
117:   CUDAFunctionTarget IdentifyTarget(const ParsedAttributesView &Attrs);
118: 
119:   enum CUDAVariableTarget {
120:     CVT_Device,  /// Emitted on device side with a shadow variable on host side
121:     CVT_Host,    /// Emitted on host side only
122:     CVT_Both,    /// Emitted on both sides with different addresses
123:     CVT_Unified, /// Emitted as a unified address, e.g. managed variables
124:   };
125:   /// Determines whether the given variable is emitted on host or device side.
126:   CUDAVariableTarget IdentifyTarget(const VarDecl *D);
127: 
128:   /// Defines kinds of CUDA global host/device context where a function may be
129:   /// called.
130:   enum CUDATargetContextKind {
131:     CTCK_Unknown,       /// Unknown context
132:     CTCK_InitGlobalVar, /// Function called during global variable
133:                         /// initialization
134:   };
135: 
```
- EN: It introduces enum-based state or option sets such as `CUDAVariableTarget`, `CUDATargetContextKind`. It exposes API surface such as `IdentifyTarget`.
- 中文: 它引入了 `CUDAVariableTarget`, `CUDATargetContextKind` 等基于枚举的状态或选项集合。 它暴露了 `IdentifyTarget` 等接口。

### Lines 136-162

```cpp
136:   /// Define the current global CUDA host/device context where a function may be
137:   /// called. Only used when a function is called outside of any functions.
138:   struct CUDATargetContext {
139:     CUDAFunctionTarget Target = CUDAFunctionTarget::HostDevice;
140:     CUDATargetContextKind Kind = CTCK_Unknown;
141:     Decl *D = nullptr;
142:   } CurCUDATargetCtx;
143: 
144:   struct CUDATargetContextRAII {
145:     SemaCUDA &S;
146:     SemaCUDA::CUDATargetContext SavedCtx;
147:     CUDATargetContextRAII(SemaCUDA &S_, SemaCUDA::CUDATargetContextKind K,
148:                           Decl *D);
149:     ~CUDATargetContextRAII() { S.CurCUDATargetCtx = SavedCtx; }
150:   };
151: 
152:   /// Gets the CUDA target for the current context.
153:   CUDAFunctionTarget CurrentTarget() {
154:     return IdentifyTarget(dyn_cast<FunctionDecl>(SemaRef.CurContext));
155:   }
156: 
157:   static bool isImplicitHostDeviceFunction(const FunctionDecl *D);
158: 
159:   // CUDA function call preference. Must be ordered numerically from
160:   // worst to best.
161:   enum CUDAFunctionPreference {
162:     CFP_Never,      // Invalid caller/callee combination.
```
- EN: Key type declarations here include `CUDATargetContext`, `CUDATargetContextRAII`. It introduces enum-based state or option sets such as `CUDAFunctionPreference`. It exposes API surface such as `~CUDATargetContextRAII`, `CurrentTarget`, `IdentifyTarget`, `isImplicitHostDeviceFunction`.
- 中文: 这里的重要类型声明包括 `CUDATargetContext`, `CUDATargetContextRAII`。 它引入了 `CUDAFunctionPreference` 等基于枚举的状态或选项集合。 它暴露了 `~CUDATargetContextRAII`, `CurrentTarget`, `IdentifyTarget`, `isImplicitHostDeviceFunction` 等接口。

### Lines 163-189

```cpp
163:     CFP_WrongSide,  // Calls from host-device to host or device
164:                     // function that do not match current compilation
165:                     // mode.
166:     CFP_HostDevice, // Any calls to host/device functions.
167:     CFP_SameSide,   // Calls from host-device to host or device
168:                     // function matching current compilation mode.
169:     CFP_Native,     // host-to-host or device-to-device calls.
170:   };
171: 
172:   /// Identifies relative preference of a given Caller/Callee
173:   /// combination, based on their host/device attributes.
174:   /// \param Caller function which needs address of \p Callee.
175:   ///               nullptr in case of global context.
176:   /// \param Callee target function
177:   ///
178:   /// \returns preference value for particular Caller/Callee combination.
179:   CUDAFunctionPreference IdentifyPreference(const FunctionDecl *Caller,
180:                                             const FunctionDecl *Callee);
181: 
182:   /// Determines whether Caller may invoke Callee, based on their CUDA
183:   /// host/device attributes.  Returns false if the call is not allowed.
184:   ///
185:   /// Note: Will return true for CFP_WrongSide calls.  These may appear in
186:   /// semantically correct CUDA programs, but only if they're never codegen'ed.
187:   bool IsAllowedCall(const FunctionDecl *Caller, const FunctionDecl *Callee) {
188:     return IdentifyPreference(Caller, Callee) != CFP_Never;
189:   }
```
- EN: It exposes API surface such as `IsAllowedCall`.
- 中文: 它暴露了 `IsAllowedCall` 等接口。

### Lines 190-216

```cpp
190: 
191:   /// May add implicit CUDAHostAttr and CUDADeviceAttr attributes to FD,
192:   /// depending on FD and the current compilation settings.
193:   void maybeAddHostDeviceAttrs(FunctionDecl *FD, const LookupResult &Previous);
194: 
195:   /// May add implicit CUDAConstantAttr attribute to VD, depending on VD
196:   /// and current compilation settings.
197:   void MaybeAddConstantAttr(VarDecl *VD);
198: 
199:   /// Check whether we're allowed to call Callee from the current context.
200:   ///
201:   /// - If the call is never allowed in a semantically-correct program
202:   ///   (CFP_Never), emits an error and returns false.
203:   ///
204:   /// - If the call is allowed in semantically-correct programs, but only if
205:   ///   it's never codegen'ed (CFP_WrongSide), creates a deferred diagnostic to
206:   ///   be emitted if and when the caller is codegen'ed, and returns true.
207:   ///
208:   ///   Will only create deferred diagnostics for a given SourceLocation once,
209:   ///   so you can safely call this multiple times without generating duplicate
210:   ///   deferred errors.
211:   ///
212:   /// - Otherwise, returns true without emitting any diagnostics.
213:   bool CheckCall(SourceLocation Loc, FunctionDecl *Callee);
214: 
215:   void CheckLambdaCapture(CXXMethodDecl *D, const sema::Capture &Capture);
216: 
```
- EN: It exposes API surface such as `maybeAddHostDeviceAttrs`, `MaybeAddConstantAttr`, `CheckCall`, `CheckLambdaCapture`.
- 中文: 它暴露了 `maybeAddHostDeviceAttrs`, `MaybeAddConstantAttr`, `CheckCall`, `CheckLambdaCapture` 等接口。

### Lines 217-243

```cpp
217:   /// Set __device__ or __host__ __device__ attributes on the given lambda
218:   /// operator() method.
219:   ///
220:   /// CUDA lambdas by default is host device function unless it has explicit
221:   /// host or device attribute.
222:   void SetLambdaAttrs(CXXMethodDecl *Method);
223: 
224:   /// Record \p FD if it is a CUDA/HIP implicit host device function used on
225:   /// device side in device compilation.
226:   void RecordImplicitHostDeviceFuncUsedByDevice(const FunctionDecl *FD);
227: 
228:   /// Finds a function in \p Matches with highest calling priority
229:   /// from \p Caller context and erases all functions with lower
230:   /// calling priority.
231:   void EraseUnwantedMatches(
232:       const FunctionDecl *Caller,
233:       llvm::SmallVectorImpl<std::pair<DeclAccessPair, FunctionDecl *>>
234:           &Matches);
235: 
236:   /// Given a implicit special member, infer its CUDA target from the
237:   /// calls it needs to make to underlying base/field special members.
238:   /// \param ClassDecl the class for which the member is being created.
239:   /// \param CSM the kind of special member.
240:   /// \param MemberDecl the special member itself.
241:   /// \param ConstRHS true if this is a copy operation with a const object on
242:   ///        its RHS.
243:   /// \param Diagnose true if this call should emit diagnostics.
```
- EN: It exposes API surface such as `SetLambdaAttrs`, `RecordImplicitHostDeviceFuncUsedByDevice`.
- 中文: 它暴露了 `SetLambdaAttrs`, `RecordImplicitHostDeviceFuncUsedByDevice` 等接口。

### Lines 244-270

```cpp
244:   /// \return true if there was an error inferring.
245:   /// The result of this call is implicit CUDA target attribute(s) attached to
246:   /// the member declaration.
247:   bool inferTargetForImplicitSpecialMember(CXXRecordDecl *ClassDecl,
248:                                            CXXSpecialMemberKind CSM,
249:                                            CXXMethodDecl *MemberDecl,
250:                                            bool ConstRHS, bool Diagnose);
251: 
252:   /// \return true if \p CD can be considered empty according to CUDA
253:   /// (E.2.3.1 in CUDA 7.5 Programming guide).
254:   bool isEmptyConstructor(SourceLocation Loc, CXXConstructorDecl *CD);
255:   bool isEmptyDestructor(SourceLocation Loc, CXXDestructorDecl *CD);
256: 
257:   // \brief Checks that initializers of \p Var satisfy CUDA restrictions. In
258:   // case of error emits appropriate diagnostic and invalidates \p Var.
259:   //
260:   // \details CUDA allows only empty constructors as initializers for global
261:   // variables (see E.2.3.1, CUDA 7.5). The same restriction also applies to all
262:   // __shared__ variables whether they are local or not (they all are implicitly
263:   // static in CUDA). One exception is that CUDA allows constant initializers
264:   // for __constant__ and __device__ variables.
265:   void checkAllowedInitializer(VarDecl *VD);
266: 
267:   /// Check whether NewFD is a valid overload for CUDA. Emits
268:   /// diagnostics and invalidates NewFD if not.
269:   void checkTargetOverload(FunctionDecl *NewFD, const LookupResult &Previous);
270:   /// Copies target attributes from the template TD to the function FD.
```
- EN: It exposes API surface such as `isEmptyConstructor`, `isEmptyDestructor`, `checkAllowedInitializer`, `checkTargetOverload`.
- 中文: 它暴露了 `isEmptyConstructor`, `isEmptyDestructor`, `checkAllowedInitializer`, `checkTargetOverload` 等接口。

### Lines 271-297

```cpp
271:   void inheritTargetAttrs(FunctionDecl *FD, const FunctionTemplateDecl &TD);
272: 
273:   /// Returns the name of the launch configuration function.  This is the name
274:   /// of the function that will be called to configure kernel call, with the
275:   /// parameters specified via <<<>>>.
276:   std::string getConfigureFuncName() const;
277:   /// Return the name of the parameter buffer allocation function for the
278:   /// device kernel launch.
279:   std::string getGetParameterBufferFuncName() const;
280:   /// Return the name of the device kernel launch function.
281:   std::string getLaunchDeviceFuncName() const;
282: 
283:   /// Record variables that are potentially ODR-used in CUDA/HIP.
284:   void recordPotentialODRUsedVariable(MultiExprArg Args,
285:                                       OverloadCandidateSet &CandidateSet);
286: 
287: private:
288:   unsigned ForceHostDeviceDepth = 0;
289: 
290:   friend class ASTReader;
291:   friend class ASTWriter;
292: };
293: 
294: } // namespace clang
295: 
296: namespace llvm {
297: // Hash a FunctionDeclAndLoc by looking at both its FunctionDecl and its
```
- EN: It opens, closes, or documents namespace scope for `clang`, `llvm`. Key type declarations here include `ASTReader`, `ASTWriter`. It exposes API surface such as `inheritTargetAttrs`, `getConfigureFuncName`, `getGetParameterBufferFuncName`, `getLaunchDeviceFuncName`.
- 中文: 它打开、关闭或说明了 `clang`, `llvm` 的命名空间作用域。 这里的重要类型声明包括 `ASTReader`, `ASTWriter`。 它暴露了 `inheritTargetAttrs`, `getConfigureFuncName`, `getGetParameterBufferFuncName`, `getLaunchDeviceFuncName` 等接口。

### Lines 298-324

```cpp
298: // SourceLocation.
299: template <> struct DenseMapInfo<clang::SemaCUDA::FunctionDeclAndLoc> {
300:   using FunctionDeclAndLoc = clang::SemaCUDA::FunctionDeclAndLoc;
301:   using FDBaseInfo =
302:       DenseMapInfo<clang::CanonicalDeclPtr<const clang::FunctionDecl>>;
303: 
304:   static FunctionDeclAndLoc getEmptyKey() {
305:     return {FDBaseInfo::getEmptyKey(), clang::SourceLocation()};
306:   }
307: 
308:   static FunctionDeclAndLoc getTombstoneKey() {
309:     return {FDBaseInfo::getTombstoneKey(), clang::SourceLocation()};
310:   }
311: 
312:   static unsigned getHashValue(const FunctionDeclAndLoc &FDL) {
313:     return hash_combine(FDBaseInfo::getHashValue(FDL.FD),
314:                         FDL.Loc.getHashValue());
315:   }
316: 
317:   static bool isEqual(const FunctionDeclAndLoc &LHS,
318:                       const FunctionDeclAndLoc &RHS) {
319:     return LHS.FD == RHS.FD && LHS.Loc == RHS.Loc;
320:   }
321: };
322: } // namespace llvm
323: 
324: #endif // LLVM_CLANG_SEMA_SEMACUDA_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `llvm`. Key type declarations here include `DenseMapInfo`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `llvm` 的命名空间作用域。 这里的重要类型声明包括 `DenseMapInfo`。

## Key Concepts / 关键概念

- `Capture`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ASTReader`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ASTWriter`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CUDAFunctionTarget`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CXXSpecialMemberKind`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ParsedAttributesView`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Scope`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `SemaCUDA`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/AST/ASTFwd.h`, `clang/AST/DeclAccessPair.h`, `clang/AST/Redeclarable.h`, `clang/Basic/Cuda.h`, `clang/Basic/LLVM.h`, `clang/Basic/SourceLocation.h`, `clang/Sema/Lookup.h`, `clang/Sema/Ownership.h`, `clang/Sema/SemaBase.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/DenseMapInfo.h`, `llvm/ADT/DenseSet.h`, `llvm/ADT/Hashing.h`, `llvm/ADT/SmallVector.h`, `string`
- Forward declarations / 前向声明: `Capture`, `ASTReader`, `ASTWriter`, `CUDAFunctionTarget`, `CXXSpecialMemberKind`, `ParsedAttributesView`, `Scope`
- Namespace context / 命名空间上下文: `clang`, `sema`, `llvm`
- Macro-style dependencies / 宏式依赖: None / 无
