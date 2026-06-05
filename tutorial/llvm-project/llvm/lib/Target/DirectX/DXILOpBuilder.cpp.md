# DXILOpBuilder.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILOpBuilder.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains class to help build DXIL op functions.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- DXILOpBuilder.cpp - Helper class for build DIXLOp functions --------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: ///
 9: /// \file This file contains class to help build DXIL op functions.
10: //===----------------------------------------------------------------------===//
11:
12: #include "DXILOpBuilder.h"
13: #include "DXILConstants.h"
14: #include "llvm/IR/Module.h"
15: #include "llvm/Support/DXILABI.h"
16: #include "llvm/Support/ErrorHandling.h"
17: #include <optional>
18:
19: using namespace llvm;
20: using namespace llvm::dxil;
21:
22: constexpr StringLiteral DXILOpNamePrefix = "dx.op.";
23:
24: namespace {
25: enum OverloadKind : uint16_t {
26:   UNDEFINED = 0,
27:   VOID = 1,
28:   HALF = 1 << 1,
29:   FLOAT = 1 << 2,
30:   DOUBLE = 1 << 3,
31:   I1 = 1 << 4,
32:   I8 = 1 << 5,
33:   I16 = 1 << 6,
34:   I32 = 1 << 7,
35:   I64 = 1 << 8,
36:   UserDefineType = 1 << 9,
37:   ObjectType = 1 << 10,
38: };
39: struct Version {
40:   unsigned Major = 0;
41:   unsigned Minor = 0;
42: };
43:
44: struct OpOverload {
45:   Version DXILVersion;
46:   uint16_t ValidTys;
47: };
48: } // namespace
49:
50: struct OpStage {
51:   Version DXILVersion;
52:   uint32_t ValidStages;
53: };
54:
55: static const char *getOverloadTypeName(OverloadKind Kind) {
56:   switch (Kind) {
57:   case OverloadKind::HALF:
58:     return "f16";
59:   case OverloadKind::FLOAT:
60:     return "f32";
```
- EN: This range defines or declares important types such as OverloadKind, Version, OpOverload, OpStage, shaping the data model used by DXILOpBuilder.cpp.
- CN: 这一段定义或声明了 OverloadKind、Version、OpOverload、OpStage 等关键类型，构成 DXILOpBuilder.cpp 使用的数据模型。

### Lines 61-120
```cpp
 61:   case OverloadKind::DOUBLE:
 62:     return "f64";
 63:   case OverloadKind::I1:
 64:     return "i1";
 65:   case OverloadKind::I8:
 66:     return "i8";
 67:   case OverloadKind::I16:
 68:     return "i16";
 69:   case OverloadKind::I32:
 70:     return "i32";
 71:   case OverloadKind::I64:
 72:     return "i64";
 73:   case OverloadKind::VOID:
 74:   case OverloadKind::UNDEFINED:
 75:     return "void";
 76:   case OverloadKind::ObjectType:
 77:   case OverloadKind::UserDefineType:
 78:     break;
 79:   }
 80:   llvm_unreachable("invalid overload type for name");
 81: }
 82:
 83: static OverloadKind getOverloadKind(Type *Ty) {
 84:   if (!Ty)
 85:     return OverloadKind::VOID;
 86:
 87:   Type::TypeID T = Ty->getTypeID();
 88:   switch (T) {
 89:   case Type::VoidTyID:
 90:     return OverloadKind::VOID;
 91:   case Type::HalfTyID:
 92:     return OverloadKind::HALF;
 93:   case Type::FloatTyID:
 94:     return OverloadKind::FLOAT;
 95:   case Type::DoubleTyID:
 96:     return OverloadKind::DOUBLE;
 97:   case Type::IntegerTyID: {
 98:     IntegerType *ITy = cast<IntegerType>(Ty);
 99:     unsigned Bits = ITy->getBitWidth();
100:     switch (Bits) {
101:     case 1:
102:       return OverloadKind::I1;
103:     case 8:
104:       return OverloadKind::I8;
105:     case 16:
106:       return OverloadKind::I16;
107:     case 32:
108:       return OverloadKind::I32;
109:     case 64:
110:       return OverloadKind::I64;
111:     default:
112:       llvm_unreachable("invalid overload type");
113:       return OverloadKind::VOID;
114:     }
115:   }
116:   case Type::PointerTyID:
117:     return OverloadKind::UserDefineType;
118:   case Type::StructTyID: {
119:     // TODO: This is a hack. As described in DXILEmitter.cpp, we need to rework
120:     // how we're handling overloads and remove the `OverloadKind` proxy enum.
```
- EN: This range implements operational logic in helpers such as llvm_unreachable, getOverloadKind, getTypeID, getBitWidth, translating backend policy into executable code.
- CN: 这一段实现了 llvm_unreachable、getOverloadKind、getTypeID、getBitWidth 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:     StructType *ST = cast<StructType>(Ty);
122:     return getOverloadKind(ST->getElementType(0));
123:   }
124:   default:
125:     return OverloadKind::UNDEFINED;
126:   }
127: }
128:
129: static std::string getTypeName(OverloadKind Kind, Type *Ty) {
130:   if (Kind < OverloadKind::UserDefineType) {
131:     return getOverloadTypeName(Kind);
132:   } else if (Kind == OverloadKind::UserDefineType) {
133:     StructType *ST = cast<StructType>(Ty);
134:     return ST->getStructName().str();
135:   } else if (Kind == OverloadKind::ObjectType) {
136:     StructType *ST = cast<StructType>(Ty);
137:     return ST->getStructName().str();
138:   } else {
139:     std::string Str;
140:     raw_string_ostream OS(Str);
141:     Ty->print(OS);
142:     return OS.str();
143:   }
144: }
145:
146: // Static properties.
147: struct OpCodeProperty {
148:   dxil::OpCode OpCode;
149:   // Offset in DXILOpCodeNameTable.
150:   unsigned OpCodeNameOffset;
151:   dxil::OpCodeClass OpCodeClass;
152:   // Offset in DXILOpCodeClassNameTable.
153:   unsigned OpCodeClassNameOffset;
154:   llvm::SmallVector<OpOverload> Overloads;
155:   llvm::SmallVector<OpStage> Stages;
156:   int OverloadParamIndex; // parameter index which control the overload.
157:                           // When < 0, should be only 1 overload type.
158: };
159:
160: // Include getOpCodeClassName getOpCodeProperty, getOpCodeName and
161: // getOpCodeParameterKind which generated by tableGen.
162: #define DXIL_OP_OPERATION_TABLE
163: #include "DXILOperation.inc"
164: #undef DXIL_OP_OPERATION_TABLE
165:
166: static std::string constructOverloadName(OverloadKind Kind, Type *Ty,
167:                                          const OpCodeProperty &Prop) {
168:   if (Kind == OverloadKind::VOID) {
169:     return (Twine(DXILOpNamePrefix) + getOpCodeClassName(Prop)).str();
170:   }
171:   return (Twine(DXILOpNamePrefix) + getOpCodeClassName(Prop) + "." +
172:           getTypeName(Kind, Ty))
173:       .str();
174: }
175:
176: static std::string constructOverloadTypeName(OverloadKind Kind,
177:                                              StringRef TypeName) {
178:   if (Kind == OverloadKind::VOID)
179:     return TypeName.str();
180:
```
- EN: This range defines or declares important types such as getOverloadKind, getTypeName, getOverloadTypeName, getStructName, shaping the data model used by DXILOpBuilder.cpp.
- CN: 这一段定义或声明了 getOverloadKind、getTypeName、getOverloadTypeName、getStructName 等关键类型，构成 DXILOpBuilder.cpp 使用的数据模型。

### Lines 181-240
```cpp
181:   assert(Kind < OverloadKind::UserDefineType && "invalid overload kind");
182:   return (Twine(TypeName) + getOverloadTypeName(Kind)).str();
183: }
184:
185: static StructType *getOrCreateStructType(StringRef Name,
186:                                          ArrayRef<Type *> EltTys,
187:                                          LLVMContext &Ctx) {
188:   StructType *ST = StructType::getTypeByName(Ctx, Name);
189:   if (ST)
190:     return ST;
191:
192:   return StructType::create(Ctx, EltTys, Name);
193: }
194:
195: static StructType *getResRetType(Type *ElementTy) {
196:   LLVMContext &Ctx = ElementTy->getContext();
197:   OverloadKind Kind = getOverloadKind(ElementTy);
198:   std::string TypeName = constructOverloadTypeName(Kind, "dx.types.ResRet.");
199:   Type *FieldTypes[5] = {ElementTy, ElementTy, ElementTy, ElementTy,
200:                          Type::getInt32Ty(Ctx)};
201:   return getOrCreateStructType(TypeName, FieldTypes, Ctx);
202: }
203:
204: static StructType *getCBufRetType(Type *ElementTy) {
205:   LLVMContext &Ctx = ElementTy->getContext();
206:   OverloadKind Kind = getOverloadKind(ElementTy);
207:   std::string TypeName = constructOverloadTypeName(Kind, "dx.types.CBufRet.");
208:
209:   // 64-bit types only have two elements
210:   if (ElementTy->isDoubleTy() || ElementTy->isIntegerTy(64))
211:     return getOrCreateStructType(TypeName, {ElementTy, ElementTy}, Ctx);
212:
213:   // 16-bit types pack 8 elements and have .8 in their name to differentiate
214:   // from min-precision types.
215:   if (ElementTy->isHalfTy() || ElementTy->isIntegerTy(16)) {
216:     TypeName += ".8";
217:     return getOrCreateStructType(TypeName,
218:                                  {ElementTy, ElementTy, ElementTy, ElementTy,
219:                                   ElementTy, ElementTy, ElementTy, ElementTy},
220:                                  Ctx);
221:   }
222:
223:   return getOrCreateStructType(
224:       TypeName, {ElementTy, ElementTy, ElementTy, ElementTy}, Ctx);
225: }
226:
227: static StructType *getHandleType(LLVMContext &Ctx) {
228:   return getOrCreateStructType("dx.types.Handle", PointerType::getUnqual(Ctx),
229:                                Ctx);
230: }
231:
232: static StructType *getResBindType(LLVMContext &Context) {
233:   if (auto *ST = StructType::getTypeByName(Context, "dx.types.ResBind"))
234:     return ST;
235:   Type *Int32Ty = Type::getInt32Ty(Context);
236:   Type *Int8Ty = Type::getInt8Ty(Context);
237:   return StructType::create({Int32Ty, Int32Ty, Int32Ty, Int8Ty},
238:                             "dx.types.ResBind");
239: }
240:
```
- EN: This range implements operational logic in helpers such as assert, StructType::getTypeByName, StructType::create, getResRetType, translating backend policy into executable code.
- CN: 这一段实现了 assert、StructType::getTypeByName、StructType::create、getResRetType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241: static StructType *getResPropsType(LLVMContext &Context) {
242:   if (auto *ST =
243:           StructType::getTypeByName(Context, "dx.types.ResourceProperties"))
244:     return ST;
245:   Type *Int32Ty = Type::getInt32Ty(Context);
246:   return StructType::create({Int32Ty, Int32Ty}, "dx.types.ResourceProperties");
247: }
248:
249: static StructType *getSplitDoubleType(LLVMContext &Context) {
250:   if (auto *ST = StructType::getTypeByName(Context, "dx.types.splitdouble"))
251:     return ST;
252:   Type *Int32Ty = Type::getInt32Ty(Context);
253:   return StructType::create({Int32Ty, Int32Ty}, "dx.types.splitdouble");
254: }
255:
256: static StructType *getBinaryWithCarryType(LLVMContext &Context) {
257:   if (auto *ST = StructType::getTypeByName(Context, "dx.types.i32c"))
258:     return ST;
259:   Type *Int32Ty = Type::getInt32Ty(Context);
260:   Type *Int1Ty = Type::getInt1Ty(Context);
261:   return StructType::create({Int32Ty, Int1Ty}, "dx.types.i32c");
262: }
263:
264: static StructType *getDimensionsType(LLVMContext &Context) {
265:   Type *Int32Ty = Type::getInt32Ty(Context);
266:   return getOrCreateStructType("dx.types.Dimensions",
267:                                {Int32Ty, Int32Ty, Int32Ty, Int32Ty}, Context);
268: }
269:
270: static StructType *getFouri32sType(LLVMContext &Context) {
271:   if (auto *ST = StructType::getTypeByName(Context, "dx.types.fouri32"))
272:     return ST;
273:   Type *Int32Ty = Type::getInt32Ty(Context);
274:   return getOrCreateStructType("dx.types.fouri32",
275:                                {Int32Ty, Int32Ty, Int32Ty, Int32Ty}, Context);
276: }
277:
278: static Type *getTypeFromOpParamType(OpParamType Kind, LLVMContext &Ctx,
279:                                     Type *OverloadTy) {
280:   switch (Kind) {
281:   case OpParamType::VoidTy:
282:     return Type::getVoidTy(Ctx);
283:   case OpParamType::HalfTy:
284:     return Type::getHalfTy(Ctx);
285:   case OpParamType::FloatTy:
286:     return Type::getFloatTy(Ctx);
287:   case OpParamType::DoubleTy:
288:     return Type::getDoubleTy(Ctx);
289:   case OpParamType::Int1Ty:
290:     return Type::getInt1Ty(Ctx);
291:   case OpParamType::Int8Ty:
292:     return Type::getInt8Ty(Ctx);
293:   case OpParamType::Int16Ty:
294:     return Type::getInt16Ty(Ctx);
295:   case OpParamType::Int32Ty:
296:     return Type::getInt32Ty(Ctx);
297:   case OpParamType::Int64Ty:
298:     return Type::getInt64Ty(Ctx);
299:   case OpParamType::OverloadTy:
300:     return OverloadTy;
```
- EN: This range implements operational logic in helpers such as getResPropsType, StructType::getTypeByName, Type::getInt32Ty, StructType::create, translating backend policy into executable code.
- CN: 这一段实现了 getResPropsType、StructType::getTypeByName、Type::getInt32Ty、StructType::create 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:   case OpParamType::ResRetHalfTy:
302:     return getResRetType(Type::getHalfTy(Ctx));
303:   case OpParamType::ResRetFloatTy:
304:     return getResRetType(Type::getFloatTy(Ctx));
305:   case OpParamType::ResRetDoubleTy:
306:     return getResRetType(Type::getDoubleTy(Ctx));
307:   case OpParamType::ResRetInt16Ty:
308:     return getResRetType(Type::getInt16Ty(Ctx));
309:   case OpParamType::ResRetInt32Ty:
310:     return getResRetType(Type::getInt32Ty(Ctx));
311:   case OpParamType::ResRetInt64Ty:
312:     return getResRetType(Type::getInt64Ty(Ctx));
313:   case OpParamType::CBufRetHalfTy:
314:     return getCBufRetType(Type::getHalfTy(Ctx));
315:   case OpParamType::CBufRetFloatTy:
316:     return getCBufRetType(Type::getFloatTy(Ctx));
317:   case OpParamType::CBufRetDoubleTy:
318:     return getCBufRetType(Type::getDoubleTy(Ctx));
319:   case OpParamType::CBufRetInt16Ty:
320:     return getCBufRetType(Type::getInt16Ty(Ctx));
321:   case OpParamType::CBufRetInt32Ty:
322:     return getCBufRetType(Type::getInt32Ty(Ctx));
323:   case OpParamType::CBufRetInt64Ty:
324:     return getCBufRetType(Type::getInt64Ty(Ctx));
325:   case OpParamType::HandleTy:
326:     return getHandleType(Ctx);
327:   case OpParamType::ResBindTy:
328:     return getResBindType(Ctx);
329:   case OpParamType::ResPropsTy:
330:     return getResPropsType(Ctx);
331:   case OpParamType::SplitDoubleTy:
332:     return getSplitDoubleType(Ctx);
333:   case OpParamType::BinaryWithCarryTy:
334:     return getBinaryWithCarryType(Ctx);
335:   case OpParamType::DimensionsTy:
336:     return getDimensionsType(Ctx);
337:   case OpParamType::Fouri32s:
338:     return getFouri32sType(Ctx);
339:   }
340:
341:   llvm_unreachable("Invalid parameter kind");
342:   return nullptr;
343: }
344:
345: static ShaderKind getShaderKindEnum(Triple::EnvironmentType EnvType) {
346:   switch (EnvType) {
347:   case Triple::Pixel:
348:     return ShaderKind::pixel;
349:   case Triple::Vertex:
350:     return ShaderKind::vertex;
351:   case Triple::Geometry:
352:     return ShaderKind::geometry;
353:   case Triple::Hull:
354:     return ShaderKind::hull;
355:   case Triple::Domain:
356:     return ShaderKind::domain;
357:   case Triple::Compute:
358:     return ShaderKind::compute;
359:   case Triple::Library:
360:     return ShaderKind::library;
```
- EN: This range implements operational logic in helpers such as getResRetType, getCBufRetType, getHandleType, getResBindType, translating backend policy into executable code.
- CN: 这一段实现了 getResRetType、getCBufRetType、getHandleType、getResBindType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:   case Triple::RayGeneration:
362:     return ShaderKind::raygeneration;
363:   case Triple::Intersection:
364:     return ShaderKind::intersection;
365:   case Triple::AnyHit:
366:     return ShaderKind::anyhit;
367:   case Triple::ClosestHit:
368:     return ShaderKind::closesthit;
369:   case Triple::Miss:
370:     return ShaderKind::miss;
371:   case Triple::Callable:
372:     return ShaderKind::callable;
373:   case Triple::Mesh:
374:     return ShaderKind::mesh;
375:   case Triple::Amplification:
376:     return ShaderKind::amplification;
377:   default:
378:     break;
379:   }
380:   llvm_unreachable(
381:       "Shader Kind Not Found - Invalid DXIL Environment Specified");
382: }
383:
384: static SmallVector<Type *>
385: getArgTypesFromOpParamTypes(ArrayRef<dxil::OpParamType> Types,
386:                             LLVMContext &Context, Type *OverloadTy) {
387:   SmallVector<Type *> ArgTys;
388:   ArgTys.emplace_back(Type::getInt32Ty(Context));
389:   for (dxil::OpParamType Ty : Types)
390:     ArgTys.emplace_back(getTypeFromOpParamType(Ty, Context, OverloadTy));
391:   return ArgTys;
392: }
393:
394: /// Construct DXIL function type. This is the type of a function with
395: /// the following prototype
396: ///     OverloadType dx.op.<opclass>.<return-type>(int opcode, <param types>)
397: /// <param-types> are constructed from types in Prop.
398: static FunctionType *getDXILOpFunctionType(dxil::OpCode OpCode,
399:                                            LLVMContext &Context,
400:                                            Type *OverloadTy) {
401:
402:   switch (OpCode) {
403: #define DXIL_OP_FUNCTION_TYPE(OpCode, RetType, ...)                            \
404:   case OpCode:                                                                 \
405:     return FunctionType::get(                                                  \
406:         getTypeFromOpParamType(RetType, Context, OverloadTy),                  \
407:         getArgTypesFromOpParamTypes({__VA_ARGS__}, Context, OverloadTy),       \
408:         /*isVarArg=*/false);
409: #include "DXILOperation.inc"
410:   }
411:   llvm_unreachable("Invalid OpCode?");
412: }
413:
414: /// Get index of the property from PropList valid for the most recent
415: /// DXIL version not greater than DXILVer.
416: /// PropList is expected to be sorted in ascending order of DXIL version.
417: template <typename T>
418: static std::optional<size_t> getPropIndex(ArrayRef<T> PropList,
419:                                           const VersionTuple DXILVer) {
420:   size_t Index = PropList.size() - 1;
```
- EN: This range implements operational logic in helpers such as emplace_back, llvm_unreachable, translating backend policy into executable code.
- CN: 这一段实现了 emplace_back、llvm_unreachable 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-480
```cpp
421:   for (auto Iter = PropList.rbegin(); Iter != PropList.rend();
422:        Iter++, Index--) {
423:     const T &Prop = *Iter;
424:     if (VersionTuple(Prop.DXILVersion.Major, Prop.DXILVersion.Minor) <=
425:         DXILVer) {
426:       return Index;
427:     }
428:   }
429:   return std::nullopt;
430: }
431:
432: // Helper function to pack an OpCode and VersionTuple into a uint64_t for use
433: // in a switch statement
434: constexpr static uint64_t computeSwitchEnum(dxil::OpCode OpCode,
435:                                             uint16_t VersionMajor,
436:                                             uint16_t VersionMinor) {
437:   uint64_t OpCodePack = (uint64_t)OpCode;
438:   return (OpCodePack << 32) | (VersionMajor << 16) | VersionMinor;
439: }
440:
441: /// Get the set of attributes for a given DXIL OpCode and the DXIL version.
442: static dxil::Attributes getDXILAttributes(dxil::OpCode OpCode,
443:                                           VersionTuple DXILVersion) {
444:   // Instantiate all versions to iterate through
445:   SmallVector<Version> Versions = {
446: #define DXIL_VERSION(MAJOR, MINOR) {MAJOR, MINOR},
447: #include "DXILOperation.inc"
448:   };
449:
450:   dxil::Attributes Attributes;
451:   for (auto Version : Versions) {
452:     if (DXILVersion < VersionTuple(Version.Major, Version.Minor))
453:       continue;
454:
455:     // Switch through and match an OpCode with the specific version and set the
456:     // corresponding flag(s) if available
457:     switch (computeSwitchEnum(OpCode, Version.Major, Version.Minor)) {
458: #define DXIL_OP_ATTRIBUTES(OpCode, VersionMajor, VersionMinor, ...)            \
459:   case computeSwitchEnum(OpCode, VersionMajor, VersionMinor): {                \
460:     auto Other = dxil::Attributes{__VA_ARGS__};                                \
461:     Attributes |= Other;                                                       \
462:     break;                                                                     \
463:   };
464: #include "DXILOperation.inc"
465:     }
466:   }
467:   return Attributes;
468: }
469:
470: /// Get the attributes to apply to the function for the DXIL operation with the
471: /// given OpCode and DXIL version.
472: static AttributeList getDXILFnAttributeList(LLVMContext &Ctx,
473:                                             dxil::OpCode OpCode,
474:                                             VersionTuple DXILVersion) {
475:   dxil::Attributes Attributes = getDXILAttributes(OpCode, DXILVersion);
476:   AttrBuilder FnAttrs(Ctx);
477:
478:   if (Attributes.ReadNone)
479:     FnAttrs.addMemoryAttr(MemoryEffects::none());
480:   if (Attributes.ReadOnly)
```
- EN: This range implements operational logic in helpers such as getDXILAttributes, FnAttrs, addMemoryAttr, translating backend policy into executable code.
- CN: 这一段实现了 getDXILAttributes、FnAttrs、addMemoryAttr 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-540
```cpp
481:     FnAttrs.addMemoryAttr(MemoryEffects::readOnly());
482:   if (Attributes.NoReturn)
483:     FnAttrs.addAttribute(Attribute::NoReturn);
484:   if (Attributes.NoDuplicate)
485:     FnAttrs.addAttribute(Attribute::NoDuplicate);
486:   FnAttrs.addAttribute(Attribute::NoUnwind);
487:
488:   return AttributeList::get(Ctx, AttributeList::FunctionIndex, FnAttrs);
489: }
490:
491: namespace llvm {
492: namespace dxil {
493:
494: // No extra checks on TargetTriple need be performed to verify that the
495: // Triple is well-formed or that the target is supported since these checks
496: // would have been done at the time the module M is constructed in the earlier
497: // stages of compilation.
498: DXILOpBuilder::DXILOpBuilder(Module &M) : M(M), IRB(M.getContext()) {
499:   const Triple &TT = M.getTargetTriple();
500:   DXILVersion = TT.getDXILVersion();
501:   ShaderStage = TT.getEnvironment();
502:   // Ensure Environment type is known
503:   if (ShaderStage == Triple::UnknownEnvironment) {
504:     reportFatalUsageError(
505:         Twine(DXILVersion.getAsString()) +
506:         ": Unknown Compilation Target Shader Stage specified ");
507:   }
508: }
509:
510: static Error makeOpError(dxil::OpCode OpCode, Twine Msg) {
511:   return make_error<StringError>(
512:       Twine("Cannot create ") + getOpCodeName(OpCode) + " operation: " + Msg,
513:       inconvertibleErrorCode());
514: }
515:
516: Expected<CallInst *> DXILOpBuilder::tryCreateOp(dxil::OpCode OpCode,
517:                                                 ArrayRef<Value *> Args,
518:                                                 const Twine &Name,
519:                                                 Type *RetTy) {
520:   const OpCodeProperty *Prop = getOpCodeProperty(OpCode);
521:
522:   Type *OverloadTy = nullptr;
523:   if (Prop->OverloadParamIndex == 0) {
524:     if (!RetTy)
525:       return makeOpError(OpCode, "Op overloaded on unknown return type");
526:     OverloadTy = RetTy;
527:   } else if (Prop->OverloadParamIndex > 0) {
528:     // The index counts including the return type
529:     unsigned ArgIndex = Prop->OverloadParamIndex - 1;
530:     if (static_cast<unsigned>(ArgIndex) >= Args.size())
531:       return makeOpError(OpCode, "Wrong number of arguments");
532:     OverloadTy = Args[ArgIndex]->getType();
533:   }
534:
535:   FunctionType *DXILOpFT =
536:       getDXILOpFunctionType(OpCode, M.getContext(), OverloadTy);
537:
538:   std::optional<size_t> OlIndexOrErr =
539:       getPropIndex(ArrayRef(Prop->Overloads), DXILVersion);
540:   if (!OlIndexOrErr.has_value())
```
- EN: This range implements operational logic in helpers such as addMemoryAttr, addAttribute, AttributeList::get, DXILOpBuilder::DXILOpBuilder, translating backend policy into executable code.
- CN: 这一段实现了 addMemoryAttr、addAttribute、AttributeList::get、DXILOpBuilder::DXILOpBuilder 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-600
```cpp
541:     return makeOpError(OpCode, Twine("No valid overloads for DXIL version ") +
542:                                    DXILVersion.getAsString());
543:
544:   uint16_t ValidTyMask = Prop->Overloads[*OlIndexOrErr].ValidTys;
545:
546:   OverloadKind Kind = getOverloadKind(OverloadTy);
547:
548:   // Check if the operation supports overload types and OverloadTy is valid
549:   // per the specified types for the operation
550:   if ((ValidTyMask != OverloadKind::UNDEFINED) &&
551:       (ValidTyMask & (uint16_t)Kind) == 0)
552:     return makeOpError(OpCode, "Invalid overload type");
553:
554:   // Perform necessary checks to ensure Opcode is valid in the targeted shader
555:   // kind
556:   std::optional<size_t> StIndexOrErr =
557:       getPropIndex(ArrayRef(Prop->Stages), DXILVersion);
558:   if (!StIndexOrErr.has_value())
559:     return makeOpError(OpCode, Twine("No valid stage for DXIL version ") +
560:                                    DXILVersion.getAsString());
561:
562:   uint16_t ValidShaderKindMask = Prop->Stages[*StIndexOrErr].ValidStages;
563:
564:   // Ensure valid shader stage properties are specified
565:   if (ValidShaderKindMask == ShaderKind::removed)
566:     return makeOpError(OpCode, "Operation has been removed");
567:
568:   // Shader stage need not be validated since getShaderKindEnum() fails
569:   // for unknown shader stage.
570:
571:   // Verify the target shader stage is valid for the DXIL operation
572:   ShaderKind ModuleStagekind = getShaderKindEnum(ShaderStage);
573:   if (!(ValidShaderKindMask & ModuleStagekind))
574:     return makeOpError(OpCode, "Invalid stage");
575:
576:   AttributeList DXILFnAttrs =
577:       getDXILFnAttributeList(M.getContext(), OpCode, DXILVersion);
578:   std::string DXILFnName = constructOverloadName(Kind, OverloadTy, *Prop);
579:   FunctionCallee DXILFn =
580:       M.getOrInsertFunction(DXILFnName, DXILOpFT, DXILFnAttrs);
581:
582:   // We need to inject the opcode as the first argument.
583:   SmallVector<Value *> OpArgs;
584:   OpArgs.push_back(IRB.getInt32(llvm::to_underlying(OpCode)));
585:   OpArgs.append(Args.begin(), Args.end());
586:
587:   // Create the function call instruction
588:   CallInst *CI = IRB.CreateCall(DXILFn, OpArgs, Name);
589:
590:   return CI;
591: }
592:
593: CallInst *DXILOpBuilder::createOp(dxil::OpCode OpCode, ArrayRef<Value *> Args,
594:                                   const Twine &Name, Type *RetTy) {
595:   Expected<CallInst *> Result = tryCreateOp(OpCode, Args, Name, RetTy);
596:   if (Error E = Result.takeError())
597:     llvm_unreachable("Invalid arguments for operation");
598:   return *Result;
599: }
600:
```
- EN: This range implements operational logic in helpers such as getAsString, getOverloadKind, makeOpError, getPropIndex, translating backend policy into executable code.
- CN: 这一段实现了 getAsString、getOverloadKind、makeOpError、getPropIndex 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 601-636
```cpp
601: StructType *DXILOpBuilder::getResRetType(Type *ElementTy) {
602:   return ::getResRetType(ElementTy);
603: }
604:
605: StructType *DXILOpBuilder::getCBufRetType(Type *ElementTy) {
606:   return ::getCBufRetType(ElementTy);
607: }
608:
609: StructType *DXILOpBuilder::getHandleType() {
610:   return ::getHandleType(IRB.getContext());
611: }
612:
613: Constant *DXILOpBuilder::getResBind(uint32_t LowerBound, uint32_t UpperBound,
614:                                     uint32_t SpaceID, dxil::ResourceClass RC) {
615:   Type *Int32Ty = IRB.getInt32Ty();
616:   Type *Int8Ty = IRB.getInt8Ty();
617:   return ConstantStruct::get(
618:       getResBindType(IRB.getContext()),
619:       {ConstantInt::get(Int32Ty, LowerBound),
620:        ConstantInt::get(Int32Ty, UpperBound),
621:        ConstantInt::get(Int32Ty, SpaceID),
622:        ConstantInt::get(Int8Ty, llvm::to_underlying(RC))});
623: }
624:
625: Constant *DXILOpBuilder::getResProps(uint32_t Word0, uint32_t Word1) {
626:   Type *Int32Ty = IRB.getInt32Ty();
627:   return ConstantStruct::get(
628:       getResPropsType(IRB.getContext()),
629:       {ConstantInt::get(Int32Ty, Word0), ConstantInt::get(Int32Ty, Word1)});
630: }
631:
632: const char *DXILOpBuilder::getOpCodeName(dxil::OpCode DXILOp) {
633:   return ::getOpCodeName(DXILOp);
634: }
635: } // namespace dxil
636: } // namespace llvm
```
- EN: This range implements operational logic in helpers such as DXILOpBuilder::getResRetType, getResRetType, DXILOpBuilder::getCBufRetType, getCBufRetType, translating backend policy into executable code.
- CN: 这一段实现了 DXILOpBuilder::getResRetType、getResRetType、DXILOpBuilder::getCBufRetType、getCBufRetType 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include OverloadKind, Version, OpOverload, OpStage, getOverloadTypeName, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 OverloadKind, Version, OpOverload, OpStage, getOverloadTypeName，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILOpBuilder.h`
  - `DXILConstants.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/Module.h`
  - `llvm/Support/DXILABI.h`
  - `llvm/Support/ErrorHandling.h`
- System/standard headers / 系统或标准头文件:
  - `optional`
  - `DXILOperation.inc`
  - `DXILOperation.inc`
  - `DXILOperation.inc`
  - `DXILOperation.inc`
