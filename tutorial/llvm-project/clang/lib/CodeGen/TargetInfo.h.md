# TargetInfo.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/CodeGen/TargetInfo.h`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose (EN) / 目的（英文）**: Provides target-specific CodeGen interfaces, factories, and ABI hooks.
- **Purpose (CN) / 目的（中文）**: 提供目标相关的 CodeGen 接口、工厂函数与 ABI 钩子。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
 1: //===---- TargetInfo.h - Encapsulate target details -------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // These classes wrap the information about a call or function
10: // definition used to handle ABI compliancy.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #ifndef LLVM_CLANG_LIB_CODEGEN_TARGETINFO_H
15: #define LLVM_CLANG_LIB_CODEGEN_TARGETINFO_H
16: 
```
- **EN**: This block contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块包含影响本编译单元构建方式的预处理结构。

### Lines 17-32
```cpp
17: #include "CGBuilder.h"
18: #include "CGValue.h"
19: #include "CodeGenModule.h"
20: #include "clang/AST/Type.h"
21: #include "clang/Basic/LLVM.h"
22: #include "clang/Basic/SyncScope.h"
23: #include "clang/Basic/TargetInfo.h"
24: #include "llvm/ADT/SmallString.h"
25: #include "llvm/ADT/StringRef.h"
26: 
27: namespace llvm {
28: class Constant;
29: class GlobalValue;
30: class Type;
31: class Value;
32: }
```
- **EN**: This block imports local CodeGen headers `CGBuilder.h`, `CGValue.h`, `CodeGenModule.h`; Clang headers `clang/AST/Type.h`, `clang/Basic/LLVM.h`, `clang/Basic/SyncScope.h`, and 1 more; LLVM headers `llvm/ADT/SmallString.h`, `llvm/ADT/StringRef.h`; opens or references namespaces `llvm`; introduces declarations such as `Constant`, `GlobalValue`, `Type`, `Value`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块引入 本地 CodeGen 头文件 `CGBuilder.h`, `CGValue.h`, `CodeGenModule.h`；Clang 头文件 `clang/AST/Type.h`, `clang/Basic/LLVM.h`, `clang/Basic/SyncScope.h`, and 1 more；LLVM 头文件 `llvm/ADT/SmallString.h`, `llvm/ADT/StringRef.h`；打开或引用命名空间 `llvm`；给出诸如 `Constant`, `GlobalValue`, `Type`, `Value` 的声明；包含影响本编译单元构建方式的预处理结构。

### Lines 33-48
```cpp
33: 
34: namespace clang {
35: class CXXRecordDecl;
36: class Decl;
37: 
38: namespace CodeGen {
39: class ABIInfo;
40: class CallArgList;
41: class CodeGenFunction;
42: class CGHLSLOffsetInfo;
43: class CGBlockInfo;
44: class CGHLSLOffsetInfo;
45: class SwiftABIInfo;
46: 
47: /// TargetCodeGenInfo - This class organizes various target-specific
48: /// codegeneration issues, like target-specific attributes, builtins and so
```
- **EN**: This block opens or references namespaces `clang`, `CodeGen`; introduces declarations such as `CXXRecordDecl`, `Decl`, `ABIInfo`, `CallArgList`, `CodeGenFunction`.
- **CN**: 该代码块打开或引用命名空间 `clang`, `CodeGen`；给出诸如 `CXXRecordDecl`, `Decl`, `ABIInfo`, `CallArgList`, `CodeGenFunction` 的声明。

### Lines 49-64
```cpp
49: /// on.
50: class TargetCodeGenInfo {
51:   std::unique_ptr<ABIInfo> Info;
52: 
53: protected:
54:   // Target hooks supporting Swift calling conventions. The target must
55:   // initialize this field if it claims to support these calling conventions
56:   // by returning true from TargetInfo::checkCallingConvention for them.
57:   std::unique_ptr<SwiftABIInfo> SwiftInfo;
58: 
59:   // Returns ABI info helper for the target. This is for use by derived classes.
60:   template <typename T> const T &getABIInfo() const {
61:     return static_cast<const T &>(*Info);
62:   }
63: 
64: public:
```
- **EN**: This block introduces declarations such as `TargetCodeGenInfo`.
- **CN**: 该代码块给出诸如 `TargetCodeGenInfo` 的声明。

### Lines 65-80
```cpp
65:   TargetCodeGenInfo(std::unique_ptr<ABIInfo> Info);
66:   virtual ~TargetCodeGenInfo();
67: 
68:   /// getABIInfo() - Returns ABI info helper for the target.
69:   const ABIInfo &getABIInfo() const { return *Info; }
70: 
71:   /// Returns Swift ABI info helper for the target.
72:   const SwiftABIInfo &getSwiftABIInfo() const {
73:     assert(SwiftInfo && "Swift ABI info has not been initialized");
74:     return *SwiftInfo;
75:   }
76: 
77:   /// supportsLibCall - Query to whether or not target supports all
78:   /// lib calls.
79:   virtual bool supportsLibCall() const { return true; }
80: 
```
- **EN**: This block defines callable entry points like `TargetCodeGenInfo`, `~TargetCodeGenInfo`, `supportsLibCall`; returns or forwards computed values for the surrounding Clang CodeGen support logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `TargetCodeGenInfo`, `~TargetCodeGenInfo`, `supportsLibCall`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 81-96
```cpp
81:   /// setTargetAttributes - Provides a convenient hook to handle extra
82:   /// target-specific attributes for the given global.
83:   virtual void setTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
84:                                    CodeGen::CodeGenModule &M) const {}
85: 
86:   /// emitTargetMetadata - Provides a convenient hook to handle extra
87:   /// target-specific metadata for the given globals.
88:   virtual void emitTargetMetadata(
89:       CodeGen::CodeGenModule &CGM,
90:       const llvm::MapVector<GlobalDecl, StringRef> &MangledDeclNames) const {}
91: 
92:   /// Provides a convenient hook to handle extra target-specific globals.
93:   virtual void emitTargetGlobals(CodeGen::CodeGenModule &CGM) const {}
94: 
95:   /// Any further codegen related checks that need to be done on a function
96:   /// signature in a target specific manner.
```
- **EN**: This block defines callable entry points like `setTargetAttributes`, `emitTargetMetadata`, `emitTargetGlobals`.
- **CN**: 该代码块定义可调用入口，例如 `setTargetAttributes`, `emitTargetMetadata`, `emitTargetGlobals`。

### Lines 97-112
```cpp
 97:   virtual void checkFunctionABI(CodeGenModule &CGM,
 98:                                 const FunctionDecl *Decl) const {}
 99: 
100:   /// Any further codegen related checks that need to be done on a function call
101:   /// in a target specific manner.
102:   virtual void checkFunctionCallABI(CodeGenModule &CGM, SourceLocation CallLoc,
103:                                     const FunctionDecl *Caller,
104:                                     const FunctionDecl *Callee,
105:                                     const CallArgList &Args,
106:                                     QualType ReturnType) const {}
107: 
108:   /// Returns true if inlining the function call would produce incorrect code
109:   /// for the current target and should be ignored (even with the always_inline
110:   /// or flatten attributes).
111:   ///
112:   /// Note: This probably should be handled in LLVM. However, the LLVM
```
- **EN**: This block defines callable entry points like `checkFunctionABI`, `checkFunctionCallABI`.
- **CN**: 该代码块定义可调用入口，例如 `checkFunctionABI`, `checkFunctionCallABI`。

### Lines 113-128
```cpp
113:   /// `alwaysinline` attribute currently means the inliner will ignore
114:   /// mismatched attributes (which sometimes can generate invalid code). So,
115:   /// this hook allows targets to avoid adding the LLVM `alwaysinline` attribute
116:   /// based on C/C++ attributes or other target-specific reasons.
117:   ///
118:   /// See previous discussion here:
119:   /// https://discourse.llvm.org/t/rfc-avoid-inlining-alwaysinline-functions-when-they-cannot-be-inlined/79528
120:   virtual bool
121:   wouldInliningViolateFunctionCallABI(const FunctionDecl *Caller,
122:                                       const FunctionDecl *Callee) const {
123:     return false;
124:   }
125: 
126:   /// Determines the size of struct _Unwind_Exception on this platform,
127:   /// in 8-bit units.  The Itanium ABI defines this as:
128:   ///   struct _Unwind_Exception {
```
- **EN**: This block defines callable entry points like `wouldInliningViolateFunctionCallABI`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `wouldInliningViolateFunctionCallABI`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 129-144
```cpp
129:   ///     uint64 exception_class;
130:   ///     _Unwind_Exception_Cleanup_Fn exception_cleanup;
131:   ///     uint64 private_1;
132:   ///     uint64 private_2;
133:   ///   };
134:   virtual unsigned getSizeOfUnwindException() const;
135: 
136:   /// Controls whether __builtin_extend_pointer should sign-extend
137:   /// pointers to uint64_t or zero-extend them (the default).  Has
138:   /// no effect for targets:
139:   ///   - that have 64-bit pointers, or
140:   ///   - that cannot address through registers larger than pointers, or
141:   ///   - that implicitly ignore/truncate the top bits when addressing
142:   ///     through such registers.
143:   virtual bool extendPointerWithSExt() const { return false; }
144: 
```
- **EN**: This block defines callable entry points like `getSizeOfUnwindException`, `extendPointerWithSExt`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `getSizeOfUnwindException`, `extendPointerWithSExt`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 145-160
```cpp
145:   /// Determines the DWARF register number for the stack pointer, for
146:   /// exception-handling purposes.  Implements __builtin_dwarf_sp_column.
147:   ///
148:   /// Returns -1 if the operation is unsupported by this target.
149:   virtual int getDwarfEHStackPointer(CodeGen::CodeGenModule &M) const {
150:     return -1;
151:   }
152: 
153:   /// Initializes the given DWARF EH register-size table, a char*.
154:   /// Implements __builtin_init_dwarf_reg_size_table.
155:   ///
156:   /// Returns true if the operation is unsupported by this target.
157:   virtual bool initDwarfEHRegSizeTable(CodeGen::CodeGenFunction &CGF,
158:                                        llvm::Value *Address) const {
159:     return true;
160:   }
```
- **EN**: This block defines callable entry points like `getDwarfEHStackPointer`, `initDwarfEHRegSizeTable`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `getDwarfEHStackPointer`, `initDwarfEHRegSizeTable`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 161-176
```cpp
161: 
162:   /// Performs the code-generation required to convert a return
163:   /// address as stored by the system into the actual address of the
164:   /// next instruction that will be executed.
165:   ///
166:   /// Used by __builtin_extract_return_addr().
167:   virtual llvm::Value *decodeReturnAddress(CodeGen::CodeGenFunction &CGF,
168:                                            llvm::Value *Address) const {
169:     return Address;
170:   }
171: 
172:   /// Performs the code-generation required to convert the address
173:   /// of an instruction into a return address suitable for storage
174:   /// by the system in a return slot.
175:   ///
176:   /// Used by __builtin_frob_return_addr().
```
- **EN**: This block documents intent or context for the surrounding Clang CodeGen support code.
- **CN**: 该代码块说明周围 Clang CodeGen 支撑逻辑 代码的意图或上下文。

### Lines 177-192
```cpp
177:   virtual llvm::Value *encodeReturnAddress(CodeGen::CodeGenFunction &CGF,
178:                                            llvm::Value *Address) const {
179:     return Address;
180:   }
181: 
182:   /// Performs a target specific test of a floating point value for things
183:   /// like IsNaN, Infinity, ... Nullptr is returned if no implementation
184:   /// exists.
185:   virtual llvm::Value *
186:   testFPKind(llvm::Value *V, unsigned BuiltinID, CGBuilderTy &Builder,
187:              CodeGenModule &CGM) const {
188:     assert(V->getType()->isFloatingPointTy() && "V should have an FP type.");
189:     return nullptr;
190:   }
191: 
192:   /// Corrects the low-level LLVM type for a given constraint and "usual"
```
- **EN**: This block defines callable entry points like `testFPKind`; returns or forwards computed values for the surrounding Clang CodeGen support logic; guards important invariants with assertions or unreachable markers.
- **CN**: 该代码块定义可调用入口，例如 `testFPKind`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果；使用断言或不可达标记保护关键不变量。

### Lines 193-208
```cpp
193:   /// type.
194:   ///
195:   /// \returns A pointer to a new LLVM type, possibly the same as the original
196:   /// on success; 0 on failure.
197:   virtual llvm::Type *adjustInlineAsmType(CodeGen::CodeGenFunction &CGF,
198:                                           StringRef Constraint,
199:                                           llvm::Type *Ty) const {
200:     return Ty;
201:   }
202: 
203:   /// Target hook to decide whether an inline asm operand can be passed
204:   /// by value.
205:   virtual bool isScalarizableAsmOperand(CodeGen::CodeGenFunction &CGF,
206:                                         llvm::Type *Ty) const {
207:     return false;
208:   }
```
- **EN**: This block defines callable entry points like `isScalarizableAsmOperand`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `isScalarizableAsmOperand`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 209-224
```cpp
209: 
210:   /// Adds constraints and types for result registers.
211:   virtual void addReturnRegisterOutputs(
212:       CodeGen::CodeGenFunction &CGF, CodeGen::LValue ReturnValue,
213:       std::string &Constraints, std::vector<llvm::Type *> &ResultRegTypes,
214:       std::vector<llvm::Type *> &ResultTruncRegTypes,
215:       std::vector<CodeGen::LValue> &ResultRegDests, std::string &AsmString,
216:       unsigned NumOutputs) const {}
217: 
218:   /// doesReturnSlotInterfereWithArgs - Return true if the target uses an
219:   /// argument slot for an 'sret' type.
220:   virtual bool doesReturnSlotInterfereWithArgs() const { return true; }
221: 
222:   /// Retrieve the address of a function to call immediately before
223:   /// calling objc_retainAutoreleasedReturnValue.  The
224:   /// implementation of objc_autoreleaseReturnValue sniffs the
```
- **EN**: This block defines callable entry points like `addReturnRegisterOutputs`, `doesReturnSlotInterfereWithArgs`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `addReturnRegisterOutputs`, `doesReturnSlotInterfereWithArgs`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 225-240
```cpp
225:   /// instruction stream following its return address to decide
226:   /// whether it's a call to objc_retainAutoreleasedReturnValue.
227:   /// This can be prohibitively expensive, depending on the
228:   /// relocation model, and so on some targets it instead sniffs for
229:   /// a particular instruction sequence.  This functions returns
230:   /// that instruction sequence in inline assembly, which will be
231:   /// empty if none is required.
232:   virtual StringRef getARCRetainAutoreleasedReturnValueMarker() const {
233:     return "";
234:   }
235: 
236:   /// Determine whether a call to objc_retainAutoreleasedReturnValue or
237:   /// objc_unsafeClaimAutoreleasedReturnValue should be marked as 'notail'.
238:   virtual bool markARCOptimizedReturnCallsAsNoTail() const { return false; }
239: 
240:   /// Return a constant used by UBSan as a signature to identify functions
```
- **EN**: This block defines callable entry points like `getARCRetainAutoreleasedReturnValueMarker`, `markARCOptimizedReturnCallsAsNoTail`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `getARCRetainAutoreleasedReturnValueMarker`, `markARCOptimizedReturnCallsAsNoTail`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 241-256
```cpp
241:   /// possessing type information, or 0 if the platform is unsupported.
242:   /// This magic number is invalid instruction encoding in many targets.
243:   virtual llvm::Constant *
244:   getUBSanFunctionSignature(CodeGen::CodeGenModule &CGM) const {
245:     return llvm::ConstantInt::get(CGM.Int32Ty, 0xc105cafe);
246:   }
247: 
248:   /// Determine whether a call to an unprototyped functions under
249:   /// the given calling convention should use the variadic
250:   /// convention or the non-variadic convention.
251:   ///
252:   /// There's a good reason to make a platform's variadic calling
253:   /// convention be different from its non-variadic calling
254:   /// convention: the non-variadic arguments can be passed in
255:   /// registers (better for performance), and the variadic arguments
256:   /// can be passed on the stack (also better for performance).  If
```
- **EN**: This block defines callable entry points like `getUBSanFunctionSignature`, `get`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `getUBSanFunctionSignature`, `get`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 257-272
```cpp
257:   /// this is done, however, unprototyped functions *must* use the
258:   /// non-variadic convention, because C99 states that a call
259:   /// through an unprototyped function type must succeed if the
260:   /// function was defined with a non-variadic prototype with
261:   /// compatible parameters.  Therefore, splitting the conventions
262:   /// makes it impossible to call a variadic function through an
263:   /// unprototyped type.  Since function prototypes came out in the
264:   /// late 1970s, this is probably an acceptable trade-off.
265:   /// Nonetheless, not all platforms are willing to make it, and in
266:   /// particularly x86-64 bends over backwards to make the
267:   /// conventions compatible.
268:   ///
269:   /// The default is false.  This is correct whenever:
270:   ///   - the conventions are exactly the same, because it does not
271:   ///     matter and the resulting IR will be somewhat prettier in
272:   ///     certain cases; or
```
- **EN**: This block documents intent or context for the surrounding Clang CodeGen support code.
- **CN**: 该代码块说明周围 Clang CodeGen 支撑逻辑 代码的意图或上下文。

### Lines 273-288
```cpp
273:   ///   - the conventions are substantively different in how they pass
274:   ///     arguments, because in this case using the variadic convention
275:   ///     will lead to C99 violations.
276:   ///
277:   /// However, some platforms make the conventions identical except
278:   /// for passing additional out-of-band information to a variadic
279:   /// function: for example, x86-64 passes the number of SSE
280:   /// arguments in %al.  On these platforms, it is desirable to
281:   /// call unprototyped functions using the variadic convention so
282:   /// that unprototyped calls to varargs functions still succeed.
283:   ///
284:   /// Relatedly, platforms which pass the fixed arguments to this:
285:   ///   A foo(B, C, D);
286:   /// differently than they would pass them to this:
287:   ///   A foo(B, C, D, ...);
288:   /// may need to adjust the debugger-support code in Sema to do the
```
- **EN**: This block documents intent or context for the surrounding Clang CodeGen support code.
- **CN**: 该代码块说明周围 Clang CodeGen 支撑逻辑 代码的意图或上下文。

### Lines 289-304
```cpp
289:   /// right thing when calling a function with no know signature.
290:   virtual bool isNoProtoCallVariadic(const CodeGen::CallArgList &args,
291:                                      const FunctionNoProtoType *fnType) const;
292: 
293:   /// Gets the linker options necessary to link a dependent library on this
294:   /// platform.
295:   virtual void getDependentLibraryOption(llvm::StringRef Lib,
296:                                          llvm::SmallString<24> &Opt) const;
297: 
298:   /// Gets the linker options necessary to detect object file mismatches on
299:   /// this platform.
300:   virtual void getDetectMismatchOption(llvm::StringRef Name,
301:                                        llvm::StringRef Value,
302:                                        llvm::SmallString<32> &Opt) const {}
303: 
304:   /// Get LLVM calling convention for device kernels.
```
- **EN**: This block defines callable entry points like `isNoProtoCallVariadic`, `getDependentLibraryOption`, `getDetectMismatchOption`.
- **CN**: 该代码块定义可调用入口，例如 `isNoProtoCallVariadic`, `getDependentLibraryOption`, `getDetectMismatchOption`。

### Lines 305-320
```cpp
305:   virtual unsigned getDeviceKernelCallingConv() const;
306: 
307:   /// Get target specific null pointer.
308:   /// \param T is the LLVM type of the null pointer.
309:   /// \param QT is the clang QualType of the null pointer.
310:   /// \return ConstantPointerNull with the given type \p T.
311:   /// Each target can override it to return its own desired constant value.
312:   virtual llvm::Constant *getNullPointer(const CodeGen::CodeGenModule &CGM,
313:       llvm::PointerType *T, QualType QT) const;
314: 
315:   /// Get target favored AST address space of a global variable for languages
316:   /// other than OpenCL and CUDA.
317:   /// If \p D is nullptr, returns the default target favored address space
318:   /// for global variable.
319:   virtual LangAS getGlobalVarAddressSpace(CodeGenModule &CGM,
320:                                           const VarDecl *D) const;
```
- **EN**: This block spells out callable entry points like `getDeviceKernelCallingConv`, `getGlobalVarAddressSpace`.
- **CN**: 该代码块给出可调用入口的声明，例如 `getDeviceKernelCallingConv`, `getGlobalVarAddressSpace`。

### Lines 321-336
```cpp
321: 
322:   /// Get the address space for an indirect (sret) return of the given type.
323:   /// The default falls back to the alloca AS.
324:   virtual LangAS getSRetAddrSpace(const CXXRecordDecl *RD) const {
325:     return LangAS::Default;
326:   }
327: 
328:   /// Get address space of pointer parameter for __cxa_atexit.
329:   virtual LangAS getAddrSpaceOfCxaAtexitPtrParam() const {
330:     return LangAS::Default;
331:   }
332: 
333:   /// Get the syncscope used in LLVM IR as a string
334:   virtual StringRef getLLVMSyncScopeStr(const LangOptions &LangOpts,
335:                                         SyncScope Scope,
336:                                         llvm::AtomicOrdering Ordering) const;
```
- **EN**: This block defines callable entry points like `getSRetAddrSpace`, `getAddrSpaceOfCxaAtexitPtrParam`, `getLLVMSyncScopeStr`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `getSRetAddrSpace`, `getAddrSpaceOfCxaAtexitPtrParam`, `getLLVMSyncScopeStr`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 337-352
```cpp
337: 
338:   /// Get the syncscope used in LLVM IR as a SyncScope ID.
339:   llvm::SyncScope::ID getLLVMSyncScopeID(const LangOptions &LangOpts,
340:                                          SyncScope Scope,
341:                                          llvm::AtomicOrdering Ordering,
342:                                          llvm::LLVMContext &Ctx) const;
343: 
344:   /// Allow the target to apply other metadata to an atomic instruction
345:   virtual void setTargetAtomicMetadata(CodeGenFunction &CGF,
346:                                        llvm::Instruction &AtomicInst,
347:                                        const AtomicExpr *Expr = nullptr) const {
348:   }
349: 
350:   /// Interface class for filling custom fields of a block literal for OpenCL.
351:   class TargetOpenCLBlockHelper {
352:   public:
```
- **EN**: This block introduces declarations such as `TargetOpenCLBlockHelper`; defines callable entry points like `getLLVMSyncScopeID`, `setTargetAtomicMetadata`.
- **CN**: 该代码块给出诸如 `TargetOpenCLBlockHelper` 的声明；定义可调用入口，例如 `getLLVMSyncScopeID`, `setTargetAtomicMetadata`。

### Lines 353-368
```cpp
353:     typedef std::pair<llvm::Value *, StringRef> ValueTy;
354:     TargetOpenCLBlockHelper() {}
355:     virtual ~TargetOpenCLBlockHelper() {}
356:     /// Get the custom field types for OpenCL blocks.
357:     virtual llvm::SmallVector<llvm::Type *, 1> getCustomFieldTypes() = 0;
358:     /// Get the custom field values for OpenCL blocks.
359:     virtual llvm::SmallVector<ValueTy, 1>
360:     getCustomFieldValues(CodeGenFunction &CGF, const CGBlockInfo &Info) = 0;
361:     virtual bool areAllCustomFieldValuesConstant(const CGBlockInfo &Info) = 0;
362:     /// Get the custom field values for OpenCL blocks if all values are LLVM
363:     /// constants.
364:     virtual llvm::SmallVector<llvm::Constant *, 1>
365:     getCustomFieldValues(CodeGenModule &CGM, const CGBlockInfo &Info) = 0;
366:   };
367:   virtual TargetOpenCLBlockHelper *getTargetOpenCLBlockHelper() const {
368:     return nullptr;
```
- **EN**: This block defines callable entry points like `TargetOpenCLBlockHelper`, `~TargetOpenCLBlockHelper`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `TargetOpenCLBlockHelper`, `~TargetOpenCLBlockHelper`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 369-384
```cpp
369:   }
370: 
371:   /// Create an OpenCL kernel for an enqueued block. The kernel function is
372:   /// a wrapper for the block invoke function with target-specific calling
373:   /// convention and ABI as an OpenCL kernel. The wrapper function accepts
374:   /// block context and block arguments in target-specific way and calls
375:   /// the original block invoke function.
376:   virtual llvm::Value *
377:   createEnqueuedBlockKernel(CodeGenFunction &CGF,
378:                             llvm::Function *BlockInvokeFunc,
379:                             llvm::Type *BlockTy) const;
380: 
381:   /// \return true if the target supports alias from the unmangled name to the
382:   /// mangled name of functions declared within an extern "C" region and marked
383:   /// as 'used', and having internal linkage.
384:   virtual bool shouldEmitStaticExternCAliases() const { return true; }
```
- **EN**: This block defines callable entry points like `createEnqueuedBlockKernel`, `shouldEmitStaticExternCAliases`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `createEnqueuedBlockKernel`, `shouldEmitStaticExternCAliases`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 385-400
```cpp
385: 
386:   /// \return true if annonymous zero-sized bitfields should be emitted to
387:   /// correctly distinguish between struct types whose memory layout is the
388:   /// same, but whose layout may differ when used as argument passed by value
389:   virtual bool shouldEmitDWARFBitFieldSeparators() const { return false; }
390: 
391:   virtual void setCUDAKernelCallingConvention(const FunctionType *&FT) const {}
392:   virtual void setOCLKernelStubCallingConvention(const FunctionType *&FT) const;
393:   /// Return the device-side type for the CUDA device builtin surface type.
394:   virtual llvm::Type *getCUDADeviceBuiltinSurfaceDeviceType() const {
395:     // By default, no change from the original one.
396:     return nullptr;
397:   }
398:   /// Return the device-side type for the CUDA device builtin texture type.
399:   virtual llvm::Type *getCUDADeviceBuiltinTextureDeviceType() const {
400:     // By default, no change from the original one.
```
- **EN**: This block defines callable entry points like `shouldEmitDWARFBitFieldSeparators`, `setCUDAKernelCallingConvention`, `setOCLKernelStubCallingConvention`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `shouldEmitDWARFBitFieldSeparators`, `setCUDAKernelCallingConvention`, `setOCLKernelStubCallingConvention`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 401-416
```cpp
401:     return nullptr;
402:   }
403: 
404:   /// Return the WebAssembly externref reference type.
405:   virtual llvm::Type *getWasmExternrefReferenceType() const { return nullptr; }
406: 
407:   /// Return the WebAssembly funcref reference type.
408:   virtual llvm::Type *getWasmFuncrefReferenceType() const { return nullptr; }
409: 
410:   /// Emit the device-side copy of the builtin surface type.
411:   virtual bool emitCUDADeviceBuiltinSurfaceDeviceCopy(CodeGenFunction &CGF,
412:                                                       LValue Dst,
413:                                                       LValue Src) const {
414:     // DO NOTHING by default.
415:     return false;
416:   }
```
- **EN**: This block defines callable entry points like `emitCUDADeviceBuiltinSurfaceDeviceCopy`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `emitCUDADeviceBuiltinSurfaceDeviceCopy`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 417-432
```cpp
417:   /// Emit the device-side copy of the builtin texture type.
418:   virtual bool emitCUDADeviceBuiltinTextureDeviceCopy(CodeGenFunction &CGF,
419:                                                       LValue Dst,
420:                                                       LValue Src) const {
421:     // DO NOTHING by default.
422:     return false;
423:   }
424: 
425:   /// Return an LLVM type that corresponds to an OpenCL type.
426:   virtual llvm::Type *getOpenCLType(CodeGenModule &CGM, const Type *T) const {
427:     return nullptr;
428:   }
429: 
430:   /// Return an LLVM type that corresponds to a HLSL type
431:   virtual llvm::Type *getHLSLType(CodeGenModule &CGM, const Type *T,
432:                                   const CGHLSLOffsetInfo &OffsetInfo) const {
```
- **EN**: This block defines callable entry points like `emitCUDADeviceBuiltinTextureDeviceCopy`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `emitCUDADeviceBuiltinTextureDeviceCopy`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 433-448
```cpp
433:     return nullptr;
434:   }
435: 
436:   /// Return an LLVM type that corresponds to padding in HLSL types
437:   virtual llvm::Type *getHLSLPadding(CodeGenModule &CGM,
438:                                      CharUnits NumBytes) const {
439:     return nullptr;
440:   }
441: 
442:   /// Return true if this is an HLSL padding type.
443:   virtual bool isHLSLPadding(llvm::Type *Ty) const { return false; }
444: 
445:   // Set the Branch Protection Attributes of the Function accordingly to the
446:   // BPI. Remove attributes that contradict with current BPI.
447:   static void
448:   setBranchProtectionFnAttributes(const TargetInfo::BranchProtectionInfo &BPI,
```
- **EN**: This block defines callable entry points like `isHLSLPadding`; returns or forwards computed values for the surrounding Clang CodeGen support logic.
- **CN**: 该代码块定义可调用入口，例如 `isHLSLPadding`；为周围的 Clang CodeGen 支撑逻辑 逻辑返回或转发计算结果。

### Lines 449-464
```cpp
449:                                   llvm::Function &F);
450: 
451:   // Add the Branch Protection Attributes of the FuncAttrs.
452:   static void
453:   initBranchProtectionFnAttributes(const TargetInfo::BranchProtectionInfo &BPI,
454:                                    llvm::AttrBuilder &FuncAttrs);
455: 
456:   // Set the ptrauth-* attributes of the Function accordingly to the Opts.
457:   // Remove attributes that contradict with current Opts.
458:   static void setPointerAuthFnAttributes(const PointerAuthOptions &Opts,
459:                                          llvm::Function &F);
460: 
461:   // Add the ptrauth-* Attributes to the FuncAttrs.
462:   static void initPointerAuthFnAttributes(const PointerAuthOptions &Opts,
463:                                           llvm::AttrBuilder &FuncAttrs);
464: 
```
- **EN**: This block spells out callable entry points like `initBranchProtectionFnAttributes`, `setPointerAuthFnAttributes`, `initPointerAuthFnAttributes`.
- **CN**: 该代码块给出可调用入口的声明，例如 `initBranchProtectionFnAttributes`, `setPointerAuthFnAttributes`, `initPointerAuthFnAttributes`。

### Lines 465-480
```cpp
465: protected:
466:   static std::string qualifyWindowsLibrary(StringRef Lib);
467: 
468:   void addStackProbeTargetAttributes(const Decl *D, llvm::GlobalValue *GV,
469:                                      CodeGen::CodeGenModule &CGM) const;
470: };
471: 
472: std::unique_ptr<TargetCodeGenInfo>
473: createDefaultTargetCodeGenInfo(CodeGenModule &CGM);
474: 
475: enum class AArch64ABIKind {
476:   AAPCS = 0,
477:   DarwinPCS,
478:   Win64,
479:   AAPCSSoft,
480: };
```
- **EN**: This block introduces declarations such as `AArch64ABIKind`; defines callable entry points like `qualifyWindowsLibrary`, `addStackProbeTargetAttributes`, `createDefaultTargetCodeGenInfo`.
- **CN**: 该代码块给出诸如 `AArch64ABIKind` 的声明；定义可调用入口，例如 `qualifyWindowsLibrary`, `addStackProbeTargetAttributes`, `createDefaultTargetCodeGenInfo`。

### Lines 481-496
```cpp
481: 
482: std::unique_ptr<TargetCodeGenInfo>
483: createAArch64TargetCodeGenInfo(CodeGenModule &CGM, AArch64ABIKind Kind);
484: 
485: std::unique_ptr<TargetCodeGenInfo>
486: createWindowsAArch64TargetCodeGenInfo(CodeGenModule &CGM, AArch64ABIKind K);
487: 
488: std::unique_ptr<TargetCodeGenInfo>
489: createAMDGPUTargetCodeGenInfo(CodeGenModule &CGM);
490: 
491: std::unique_ptr<TargetCodeGenInfo>
492: createARCTargetCodeGenInfo(CodeGenModule &CGM);
493: 
494: enum class ARMABIKind {
495:   APCS = 0,
496:   AAPCS = 1,
```
- **EN**: This block introduces declarations such as `ARMABIKind`; defines callable entry points like `createAArch64TargetCodeGenInfo`, `createWindowsAArch64TargetCodeGenInfo`, `createAMDGPUTargetCodeGenInfo`, `createARCTargetCodeGenInfo`.
- **CN**: 该代码块给出诸如 `ARMABIKind` 的声明；定义可调用入口，例如 `createAArch64TargetCodeGenInfo`, `createWindowsAArch64TargetCodeGenInfo`, `createAMDGPUTargetCodeGenInfo`, `createARCTargetCodeGenInfo`。

### Lines 497-512
```cpp
497:   AAPCS_VFP = 2,
498:   AAPCS16_VFP = 3,
499: };
500: 
501: std::unique_ptr<TargetCodeGenInfo>
502: createARMTargetCodeGenInfo(CodeGenModule &CGM, ARMABIKind Kind);
503: 
504: std::unique_ptr<TargetCodeGenInfo>
505: createWindowsARMTargetCodeGenInfo(CodeGenModule &CGM, ARMABIKind K);
506: 
507: std::unique_ptr<TargetCodeGenInfo>
508: createAVRTargetCodeGenInfo(CodeGenModule &CGM, unsigned NPR, unsigned NRR);
509: 
510: std::unique_ptr<TargetCodeGenInfo>
511: createBPFTargetCodeGenInfo(CodeGenModule &CGM);
512: 
```
- **EN**: This block spells out callable entry points like `createARMTargetCodeGenInfo`, `createWindowsARMTargetCodeGenInfo`, `createAVRTargetCodeGenInfo`, `createBPFTargetCodeGenInfo`.
- **CN**: 该代码块给出可调用入口的声明，例如 `createARMTargetCodeGenInfo`, `createWindowsARMTargetCodeGenInfo`, `createAVRTargetCodeGenInfo`, `createBPFTargetCodeGenInfo`。

### Lines 513-528
```cpp
513: std::unique_ptr<TargetCodeGenInfo>
514: createCSKYTargetCodeGenInfo(CodeGenModule &CGM, unsigned FLen);
515: 
516: std::unique_ptr<TargetCodeGenInfo>
517: createHexagonTargetCodeGenInfo(CodeGenModule &CGM);
518: 
519: std::unique_ptr<TargetCodeGenInfo>
520: createLanaiTargetCodeGenInfo(CodeGenModule &CGM);
521: 
522: std::unique_ptr<TargetCodeGenInfo>
523: createLoongArchTargetCodeGenInfo(CodeGenModule &CGM, unsigned GRLen,
524:                                  unsigned FLen);
525: 
526: std::unique_ptr<TargetCodeGenInfo>
527: createM68kTargetCodeGenInfo(CodeGenModule &CGM);
528: 
```
- **EN**: This block spells out callable entry points like `createCSKYTargetCodeGenInfo`, `createHexagonTargetCodeGenInfo`, `createLanaiTargetCodeGenInfo`, `createLoongArchTargetCodeGenInfo`, `createM68kTargetCodeGenInfo`.
- **CN**: 该代码块给出可调用入口的声明，例如 `createCSKYTargetCodeGenInfo`, `createHexagonTargetCodeGenInfo`, `createLanaiTargetCodeGenInfo`, `createLoongArchTargetCodeGenInfo`, `createM68kTargetCodeGenInfo`。

### Lines 529-544
```cpp
529: std::unique_ptr<TargetCodeGenInfo>
530: createMIPSTargetCodeGenInfo(CodeGenModule &CGM, bool IsOS32);
531: 
532: std::unique_ptr<TargetCodeGenInfo>
533: createWindowsMIPSTargetCodeGenInfo(CodeGenModule &CGM, bool IsOS32);
534: 
535: std::unique_ptr<TargetCodeGenInfo>
536: createMSP430TargetCodeGenInfo(CodeGenModule &CGM);
537: 
538: std::unique_ptr<TargetCodeGenInfo>
539: createNVPTXTargetCodeGenInfo(CodeGenModule &CGM);
540: 
541: enum class PPC64_SVR4_ABIKind {
542:   ELFv1 = 0,
543:   ELFv2,
544: };
```
- **EN**: This block introduces declarations such as `PPC64_SVR4_ABIKind`; defines callable entry points like `createMIPSTargetCodeGenInfo`, `createWindowsMIPSTargetCodeGenInfo`, `createMSP430TargetCodeGenInfo`, `createNVPTXTargetCodeGenInfo`.
- **CN**: 该代码块给出诸如 `PPC64_SVR4_ABIKind` 的声明；定义可调用入口，例如 `createMIPSTargetCodeGenInfo`, `createWindowsMIPSTargetCodeGenInfo`, `createMSP430TargetCodeGenInfo`, `createNVPTXTargetCodeGenInfo`。

### Lines 545-560
```cpp
545: 
546: std::unique_ptr<TargetCodeGenInfo>
547: createAIXTargetCodeGenInfo(CodeGenModule &CGM, bool Is64Bit);
548: 
549: std::unique_ptr<TargetCodeGenInfo>
550: createPPC32TargetCodeGenInfo(CodeGenModule &CGM, bool SoftFloatABI);
551: 
552: std::unique_ptr<TargetCodeGenInfo>
553: createPPC64TargetCodeGenInfo(CodeGenModule &CGM);
554: 
555: std::unique_ptr<TargetCodeGenInfo>
556: createPPC64_SVR4_TargetCodeGenInfo(CodeGenModule &CGM, PPC64_SVR4_ABIKind Kind,
557:                                    bool SoftFloatABI);
558: 
559: std::unique_ptr<TargetCodeGenInfo>
560: createRISCVTargetCodeGenInfo(CodeGenModule &CGM, unsigned XLen, unsigned FLen,
```
- **EN**: This block spells out callable entry points like `createAIXTargetCodeGenInfo`, `createPPC32TargetCodeGenInfo`, `createPPC64TargetCodeGenInfo`, `createPPC64_SVR4_TargetCodeGenInfo`.
- **CN**: 该代码块给出可调用入口的声明，例如 `createAIXTargetCodeGenInfo`, `createPPC32TargetCodeGenInfo`, `createPPC64TargetCodeGenInfo`, `createPPC64_SVR4_TargetCodeGenInfo`。

### Lines 561-576
```cpp
561:                              bool EABI);
562: 
563: std::unique_ptr<TargetCodeGenInfo>
564: createCommonSPIRTargetCodeGenInfo(CodeGenModule &CGM);
565: 
566: std::unique_ptr<TargetCodeGenInfo>
567: createSPIRVTargetCodeGenInfo(CodeGenModule &CGM);
568: 
569: std::unique_ptr<TargetCodeGenInfo>
570: createSparcV8TargetCodeGenInfo(CodeGenModule &CGM);
571: 
572: std::unique_ptr<TargetCodeGenInfo>
573: createSparcV9TargetCodeGenInfo(CodeGenModule &CGM);
574: 
575: std::unique_ptr<TargetCodeGenInfo>
576: createSystemZTargetCodeGenInfo(CodeGenModule &CGM, bool HasVector,
```
- **EN**: This block spells out callable entry points like `createCommonSPIRTargetCodeGenInfo`, `createSPIRVTargetCodeGenInfo`, `createSparcV8TargetCodeGenInfo`, `createSparcV9TargetCodeGenInfo`.
- **CN**: 该代码块给出可调用入口的声明，例如 `createCommonSPIRTargetCodeGenInfo`, `createSPIRVTargetCodeGenInfo`, `createSparcV8TargetCodeGenInfo`, `createSparcV9TargetCodeGenInfo`。

### Lines 577-592
```cpp
577:                                bool SoftFloatABI);
578: 
579: std::unique_ptr<TargetCodeGenInfo>
580: createTCETargetCodeGenInfo(CodeGenModule &CGM);
581: 
582: std::unique_ptr<TargetCodeGenInfo>
583: createVETargetCodeGenInfo(CodeGenModule &CGM);
584: 
585: std::unique_ptr<TargetCodeGenInfo>
586: createDirectXTargetCodeGenInfo(CodeGenModule &CGM);
587: 
588: enum class WebAssemblyABIKind {
589:   MVP = 0,
590:   ExperimentalMV = 1,
591: };
592: 
```
- **EN**: This block introduces declarations such as `WebAssemblyABIKind`; defines callable entry points like `createTCETargetCodeGenInfo`, `createVETargetCodeGenInfo`, `createDirectXTargetCodeGenInfo`.
- **CN**: 该代码块给出诸如 `WebAssemblyABIKind` 的声明；定义可调用入口，例如 `createTCETargetCodeGenInfo`, `createVETargetCodeGenInfo`, `createDirectXTargetCodeGenInfo`。

### Lines 593-608
```cpp
593: std::unique_ptr<TargetCodeGenInfo>
594: createWebAssemblyTargetCodeGenInfo(CodeGenModule &CGM, WebAssemblyABIKind K);
595: 
596: /// The AVX ABI level for X86 targets.
597: enum class X86AVXABILevel {
598:   None,
599:   AVX,
600:   AVX512,
601: };
602: 
603: std::unique_ptr<TargetCodeGenInfo> createX86_32TargetCodeGenInfo(
604:     CodeGenModule &CGM, bool DarwinVectorABI, bool Win32StructABI,
605:     unsigned NumRegisterParameters, bool SoftFloatABI);
606: 
607: std::unique_ptr<TargetCodeGenInfo>
608: createWinX86_32TargetCodeGenInfo(CodeGenModule &CGM, bool DarwinVectorABI,
```
- **EN**: This block introduces declarations such as `X86AVXABILevel`; defines callable entry points like `createWebAssemblyTargetCodeGenInfo`, `createX86_32TargetCodeGenInfo`.
- **CN**: 该代码块给出诸如 `X86AVXABILevel` 的声明；定义可调用入口，例如 `createWebAssemblyTargetCodeGenInfo`, `createX86_32TargetCodeGenInfo`。

### Lines 609-624
```cpp
609:                                  bool Win32StructABI,
610:                                  unsigned NumRegisterParameters);
611: 
612: std::unique_ptr<TargetCodeGenInfo>
613: createX86_64TargetCodeGenInfo(CodeGenModule &CGM, X86AVXABILevel AVXLevel);
614: 
615: std::unique_ptr<TargetCodeGenInfo>
616: createWinX86_64TargetCodeGenInfo(CodeGenModule &CGM, X86AVXABILevel AVXLevel);
617: 
618: std::unique_ptr<TargetCodeGenInfo>
619: createXCoreTargetCodeGenInfo(CodeGenModule &CGM);
620: 
621: } // namespace CodeGen
622: } // namespace clang
623: 
624: #endif // LLVM_CLANG_LIB_CODEGEN_TARGETINFO_H
```
- **EN**: This block opens or references namespaces `CodeGen`, `clang`; spells out callable entry points like `createX86_64TargetCodeGenInfo`, `createWinX86_64TargetCodeGenInfo`, `createXCoreTargetCodeGenInfo`; contains preprocessor structure that shapes compilation of this unit.
- **CN**: 该代码块打开或引用命名空间 `CodeGen`, `clang`；给出可调用入口的声明，例如 `createX86_64TargetCodeGenInfo`, `createWinX86_64TargetCodeGenInfo`, `createXCoreTargetCodeGenInfo`；包含影响本编译单元构建方式的预处理结构。

## Key Concepts / 关键概念

- **CodeGenModule**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGM**: Central symbol in this file's implementation of Clang CodeGen support. / 是该文件实现 Clang CodeGen 支撑逻辑 时的核心符号。
- **TargetCodeGenInfo**: Likely stores or computes descriptive metadata that drives Clang CodeGen support. / 很可能用于保存或计算驱动 Clang CodeGen 支撑逻辑 的描述性元数据。
- **Type**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CodeGenFunction**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **Value**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。
- **CGF**: Central symbol in this file's implementation of Clang CodeGen support. / 是该文件实现 Clang CodeGen 支撑逻辑 时的核心符号。
- **StringRef**: Frequently referenced symbol that likely anchors a key abstraction in this file. / 高频出现的符号，通常对应此文件中的关键抽象。

## Dependencies / 依赖关系

- **Local CodeGen / 本地 CodeGen**: `CGBuilder.h`, `CGValue.h`, `CodeGenModule.h`
- **Clang libraries / Clang 库**: `clang/AST/Type.h`, `clang/Basic/LLVM.h`, `clang/Basic/SyncScope.h`, `clang/Basic/TargetInfo.h`
- **LLVM libraries / LLVM 库**: `llvm/ADT/SmallString.h`, `llvm/ADT/StringRef.h`
