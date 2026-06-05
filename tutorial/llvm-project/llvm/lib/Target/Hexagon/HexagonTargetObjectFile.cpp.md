# HexagonTargetObjectFile.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonTargetObjectFile.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file contains the declarations of the HexagonTargetAsmInfo properties.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及汇编/MC 层处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===-- HexagonTargetObjectFile.cpp ---------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file contains the declarations of the HexagonTargetAsmInfo properties.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #include "HexagonTargetObjectFile.h"
    14: #include "llvm/ADT/SmallString.h"
    15: #include "llvm/ADT/StringRef.h"
    16: #include "llvm/ADT/Twine.h"
    17: #include "llvm/BinaryFormat/ELF.h"
    18: #include "llvm/IR/DataLayout.h"
    19: #include "llvm/IR/DerivedTypes.h"
    20: #include "llvm/IR/GlobalObject.h"
    21: #include "llvm/IR/GlobalValue.h"
    22: #include "llvm/IR/GlobalVariable.h"
    23: #include "llvm/IR/Module.h"
    24: #include "llvm/IR/Type.h"
    25: #include "llvm/MC/MCContext.h"
    26: #include "llvm/MC/SectionKind.h"
    27: #include "llvm/Support/Casting.h"
    28: #include "llvm/Support/CommandLine.h"
    29: #include "llvm/Support/Debug.h"
    30: #include "llvm/Support/raw_ostream.h"
    31: #include "llvm/Target/TargetMachine.h"
    32: 
    33: #define DEBUG_TYPE "hexagon-sdata"
    34: 
    35: using namespace llvm;
    36: 
    37: static cl::opt<unsigned> SmallDataThreshold("hexagon-small-data-threshold",
    38:   cl::init(8), cl::Hidden,
    39:   cl::desc("The maximum size of an object in the sdata section"));
    40: 
    41: static cl::opt<bool> NoSmallDataSorting("mno-sort-sda", cl::init(false),
    42:   cl::Hidden, cl::desc("Disable small data sections sorting"));
    43: 
    44: static cl::opt<bool>
    45:     StaticsInSData("hexagon-statics-in-small-data", cl::Hidden,
    46:                    cl::desc("Allow static variables in .sdata"));
    47: 
    48: static cl::opt<bool> TraceGVPlacement("trace-gv-placement",
    49:   cl::Hidden, cl::init(false),
    50:   cl::desc("Trace global value placement"));
```
- EN: It imports headers such as HexagonTargetObjectFile.h, llvm/ADT/SmallString.h, llvm/ADT/StringRef.h, llvm/ADT/Twine.h, ... (19 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as SmallDataThreshold, NoSmallDataSorting, StaticsInSData, TraceGVPlacement, translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonTargetObjectFile.h, llvm/ADT/SmallString.h, llvm/ADT/StringRef.h, llvm/ADT/Twine.h, ... (19 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 SmallDataThreshold, NoSmallDataSorting, StaticsInSData, TraceGVPlacement 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-100 / 第 51-100 行

```cpp
    51: 
    52: static cl::opt<bool>
    53:     EmitJtInText("hexagon-emit-jt-text", cl::Hidden, cl::init(false),
    54:                  cl::desc("Emit hexagon jump tables in function section"));
    55: 
    56: static cl::opt<bool>
    57:     EmitLutInText("hexagon-emit-lut-text", cl::Hidden, cl::init(false),
    58:                  cl::desc("Emit hexagon lookup tables in function section"));
    59: 
    60: // TraceGVPlacement controls messages for all builds. For builds with assertions
    61: // (debug or release), messages are also controlled by the usual debug flags
    62: // (e.g. -debug and -debug-only=globallayout)
    63: #define TRACE_TO(s, X) s << X
    64: #ifdef NDEBUG
    65: #define TRACE(X)                                                               \
    66:   do {                                                                         \
    67:     if (TraceGVPlacement) {                                                    \
    68:       TRACE_TO(errs(), X);                                                     \
    69:     }                                                                          \
    70:   } while (false)
    71: #else
    72: #define TRACE(X)                                                               \
    73:   do {                                                                         \
    74:     if (TraceGVPlacement) {                                                    \
    75:       TRACE_TO(errs(), X);                                                     \
    76:     } else {                                                                   \
    77:       LLVM_DEBUG(TRACE_TO(dbgs(), X));                                         \
    78:     }                                                                          \
    79:   } while (false)
    80: #endif
    81: 
    82: // Returns true if the section name is such that the symbol will be put
    83: // in a small data section.
    84: // For instance, global variables with section attributes such as ".sdata"
    85: // ".sdata.*", ".sbss", and ".sbss.*" will go into small data.
    86: static bool isSmallDataSection(StringRef Sec) {
    87:   // sectionName is either ".sdata" or ".sbss". Looking for an exact match
    88:   // obviates the need for checks for section names such as ".sdatafoo".
    89:   if (Sec == ".sdata" || Sec == ".sbss" || Sec == ".scommon")
    90:     return true;
    91:   // If either ".sdata." or ".sbss." is a substring of the section name
    92:   // then put the symbol in small data.
    93:   return Sec.contains(".sdata.") || Sec.contains(".sbss.") ||
    94:          Sec.contains(".scommon.");
    95: }
    96: 
    97: static const char *getSectionSuffixForSize(unsigned Size) {
    98:   switch (Size) {
    99:   default:
   100:     return "";
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as EmitJtInText, EmitLutInText, TRACE_TO, LLVM_DEBUG, ... (6 total), translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 EmitJtInText, EmitLutInText, TRACE_TO, LLVM_DEBUG, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 101-150 / 第 101-150 行

```cpp
   101:   case 1:
   102:     return ".1";
   103:   case 2:
   104:     return ".2";
   105:   case 4:
   106:     return ".4";
   107:   case 8:
   108:     return ".8";
   109:   }
   110: }
   111: 
   112: void HexagonTargetObjectFile::Initialize(MCContext &Ctx,
   113:       const TargetMachine &TM) {
   114:   TargetLoweringObjectFileELF::Initialize(Ctx, TM);
   115: 
   116:   SmallDataSection =
   117:     getContext().getELFSection(".sdata", ELF::SHT_PROGBITS,
   118:                                ELF::SHF_WRITE | ELF::SHF_ALLOC |
   119:                                ELF::SHF_HEX_GPREL);
   120:   SmallBSSSection =
   121:     getContext().getELFSection(".sbss", ELF::SHT_NOBITS,
   122:                                ELF::SHF_WRITE | ELF::SHF_ALLOC |
   123:                                ELF::SHF_HEX_GPREL);
   124: }
   125: 
   126: MCSection *HexagonTargetObjectFile::SelectSectionForGlobal(
   127:     const GlobalObject *GO, SectionKind Kind, const TargetMachine &TM) const {
   128:   TRACE("[SelectSectionForGlobal] GO(" << GO->getName() << ") ");
   129:   TRACE("input section(" << GO->getSection() << ") ");
   130: 
   131:   TRACE((GO->hasPrivateLinkage() ? "private_linkage " : "")
   132:          << (GO->hasLocalLinkage() ? "local_linkage " : "")
   133:          << (GO->hasInternalLinkage() ? "internal " : "")
   134:          << (GO->hasExternalLinkage() ? "external " : "")
   135:          << (GO->hasCommonLinkage() ? "common_linkage " : "")
   136:          << (GO->hasCommonLinkage() ? "common " : "" )
   137:          << (Kind.isCommon() ? "kind_common " : "" )
   138:          << (Kind.isBSS() ? "kind_bss " : "" )
   139:          << (Kind.isBSSLocal() ? "kind_bss_local " : "" ));
   140: 
   141:   // If the lookup table is used by more than one function, do not place
   142:   // it in text section.
   143:   if (EmitLutInText && GO->getName().starts_with("switch.table")) {
   144:     if (const Function *Fn = getLutUsedFunction(GO))
   145:       return selectSectionForLookupTable(GO, TM, Fn);
   146:   }
   147: 
   148:   if (isGlobalInSmallSection(GO, TM))
   149:     return selectSmallSectionForGlobal(GO, Kind, TM);
   150: 
```
- EN: It declares or implements routines such as HexagonTargetObjectFile::Initialize, TargetLoweringObjectFileELF::Initialize, getContext, HexagonTargetObjectFile::SelectSectionForGlobal, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTargetObjectFile, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonTargetObjectFile::Initialize, TargetLoweringObjectFileELF::Initialize, getContext, HexagonTargetObjectFile::SelectSectionForGlobal, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTargetObjectFile，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   if (Kind.isCommon()) {
   152:     // This is purely for LTO+Linker Script because commons don't really have a
   153:     // section. However, the BitcodeSectionWriter pass will query for the
   154:     // sections of commons (and the linker expects us to know their section) so
   155:     // we'll return one here.
   156:     return BSSSection;
   157:   }
   158: 
   159:   TRACE("default_ELF_section\n");
   160:   // Otherwise, we work the same as ELF.
   161:   return TargetLoweringObjectFileELF::SelectSectionForGlobal(GO, Kind, TM);
   162: }
   163: 
   164: MCSection *HexagonTargetObjectFile::getExplicitSectionGlobal(
   165:     const GlobalObject *GO, SectionKind Kind, const TargetMachine &TM) const {
   166:   TRACE("[getExplicitSectionGlobal] GO(" << GO->getName() << ") from("
   167:         << GO->getSection() << ") ");
   168:   TRACE((GO->hasPrivateLinkage() ? "private_linkage " : "")
   169:          << (GO->hasLocalLinkage() ? "local_linkage " : "")
   170:          << (GO->hasInternalLinkage() ? "internal " : "")
   171:          << (GO->hasExternalLinkage() ? "external " : "")
   172:          << (GO->hasCommonLinkage() ? "common_linkage " : "")
   173:          << (GO->hasCommonLinkage() ? "common " : "" )
   174:          << (Kind.isCommon() ? "kind_common " : "" )
   175:          << (Kind.isBSS() ? "kind_bss " : "" )
   176:          << (Kind.isBSSLocal() ? "kind_bss_local " : "" ));
   177: 
   178:   if (GO->hasSection()) {
   179:     StringRef Section = GO->getSection();
   180:     if (Section.contains(".access.text.group"))
   181:       return getContext().getELFSection(GO->getSection(), ELF::SHT_PROGBITS,
   182:                                         ELF::SHF_ALLOC | ELF::SHF_EXECINSTR);
   183:     if (Section.contains(".access.data.group"))
   184:       return getContext().getELFSection(GO->getSection(), ELF::SHT_PROGBITS,
   185:                                         ELF::SHF_WRITE | ELF::SHF_ALLOC);
   186:   }
   187: 
   188:   if (isGlobalInSmallSection(GO, TM))
   189:     return selectSmallSectionForGlobal(GO, Kind, TM);
   190: 
   191:   // Otherwise, we work the same as ELF.
   192:   TRACE("default_ELF_section\n");
   193:   return TargetLoweringObjectFileELF::getExplicitSectionGlobal(GO, Kind, TM);
   194: }
   195: 
   196: /// Return true if this global value should be placed into small data/bss
   197: /// section.
   198: bool HexagonTargetObjectFile::isGlobalInSmallSection(const GlobalObject *GO,
   199:       const TargetMachine &TM) const {
   200:   bool HaveSData = isSmallDataEnabled(TM);
```
- EN: It declares or implements routines such as TRACE, TargetLoweringObjectFileELF::SelectSectionForGlobal, HexagonTargetObjectFile::getExplicitSectionGlobal, getSection, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTargetObjectFile, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 TRACE, TargetLoweringObjectFileELF::SelectSectionForGlobal, HexagonTargetObjectFile::getExplicitSectionGlobal, getSection, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTargetObjectFile，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   if (!HaveSData)
   202:     LLVM_DEBUG(dbgs() << "Small-data allocation is disabled, but symbols "
   203:                          "may have explicit section assignments...\n");
   204:   // Only global variables, not functions.
   205:   LLVM_DEBUG(dbgs() << "Checking if value is in small-data, -G"
   206:                     << SmallDataThreshold << ": \"" << GO->getName() << "\": ");
   207:   const GlobalVariable *GVar = dyn_cast<GlobalVariable>(GO);
   208:   if (!GVar) {
   209:     LLVM_DEBUG(dbgs() << "no, not a global variable\n");
   210:     return false;
   211:   }
   212: 
   213:   // Globals with external linkage that have an original section set must be
   214:   // emitted to that section, regardless of whether we would put them into
   215:   // small data or not. This is how we can support mixing -G0/-G8 in LTO.
   216:   if (GVar->hasSection()) {
   217:     bool IsSmall = isSmallDataSection(GVar->getSection());
   218:     LLVM_DEBUG(dbgs() << (IsSmall ? "yes" : "no")
   219:                       << ", has section: " << GVar->getSection() << '\n');
   220:     return IsSmall;
   221:   }
   222: 
   223:   // If sdata is disabled, stop the checks here.
   224:   if (!HaveSData) {
   225:     LLVM_DEBUG(dbgs() << "no, small-data allocation is disabled\n");
   226:     return false;
   227:   }
   228: 
   229:   if (GVar->isConstant()) {
   230:     LLVM_DEBUG(dbgs() << "no, is a constant\n");
   231:     return false;
   232:   }
   233: 
   234:   bool IsLocal = GVar->hasLocalLinkage();
   235:   if (!StaticsInSData && IsLocal) {
   236:     LLVM_DEBUG(dbgs() << "no, is static\n");
   237:     return false;
   238:   }
   239: 
   240:   Type *GType = GVar->getValueType();
   241:   if (isa<ArrayType>(GType)) {
   242:     LLVM_DEBUG(dbgs() << "no, is an array\n");
   243:     return false;
   244:   }
   245: 
   246:   // If the type is a struct with no body provided, treat is conservatively.
   247:   // There cannot be actual definitions of object of such a type in this CU
   248:   // (only references), so assuming that they are not in sdata is safe. If
   249:   // these objects end up in the sdata, the references will still be valid.
   250:   if (StructType *ST = dyn_cast<StructType>(GType)) {
```
- EN: It declares or implements routines such as LLVM_DEBUG, dyn_cast<GlobalVariable>, isSmallDataSection, hasLocalLinkage, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 LLVM_DEBUG, dyn_cast<GlobalVariable>, isSmallDataSection, hasLocalLinkage, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 251-300 / 第 251-300 行

```cpp
   251:     if (ST->isOpaque()) {
   252:       LLVM_DEBUG(dbgs() << "no, has opaque type\n");
   253:       return false;
   254:     }
   255:   }
   256: 
   257:   unsigned Size = GVar->getDataLayout().getTypeAllocSize(GType);
   258:   if (Size == 0) {
   259:     LLVM_DEBUG(dbgs() << "no, has size 0\n");
   260:     return false;
   261:   }
   262:   if (Size > SmallDataThreshold) {
   263:     LLVM_DEBUG(dbgs() << "no, size exceeds sdata threshold: " << Size << '\n');
   264:     return false;
   265:   }
   266: 
   267:   LLVM_DEBUG(dbgs() << "yes\n");
   268:   return true;
   269: }
   270: 
   271: bool HexagonTargetObjectFile::isSmallDataEnabled(const TargetMachine &TM)
   272:     const {
   273:   return SmallDataThreshold > 0 && !TM.isPositionIndependent();
   274: }
   275: 
   276: unsigned HexagonTargetObjectFile::getSmallDataSize() const {
   277:   return SmallDataThreshold;
   278: }
   279: 
   280: bool HexagonTargetObjectFile::shouldPutJumpTableInFunctionSection(
   281:     bool UsesLabelDifference, const Function &F) const {
   282:   return EmitJtInText;
   283: }
   284: 
   285: /// Descends any type down to "elementary" components,
   286: /// discovering the smallest addressable one.
   287: /// If zero is returned, declaration will not be modified.
   288: unsigned HexagonTargetObjectFile::getSmallestAddressableSize(const Type *Ty,
   289:       const GlobalValue *GV, const TargetMachine &TM) const {
   290:   // Assign the smallest element access size to the highest
   291:   // value which assembler can handle.
   292:   unsigned SmallestElement = 8;
   293: 
   294:   if (!Ty)
   295:     return 0;
   296:   switch (Ty->getTypeID()) {
   297:   case Type::StructTyID: {
   298:     const StructType *STy = cast<const StructType>(Ty);
   299:     for (auto &E : STy->elements()) {
   300:       unsigned AtomicSize = getSmallestAddressableSize(E, GV, TM);
```
- EN: It declares or implements routines such as LLVM_DEBUG, getDataLayout, HexagonTargetObjectFile::isSmallDataEnabled, isPositionIndependent, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTargetObjectFile, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 LLVM_DEBUG, getDataLayout, HexagonTargetObjectFile::isSmallDataEnabled, isPositionIndependent, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTargetObjectFile，说明了它与同级后端组件的连接关系。

### Lines 301-350 / 第 301-350 行

```cpp
   301:       if (AtomicSize < SmallestElement)
   302:         SmallestElement = AtomicSize;
   303:     }
   304:     return (STy->getNumElements() == 0) ? 0 : SmallestElement;
   305:   }
   306:   case Type::ArrayTyID: {
   307:     const ArrayType *ATy = cast<const ArrayType>(Ty);
   308:     return getSmallestAddressableSize(ATy->getElementType(), GV, TM);
   309:   }
   310:   case Type::FixedVectorTyID:
   311:   case Type::ScalableVectorTyID: {
   312:     const VectorType *PTy = cast<const VectorType>(Ty);
   313:     return getSmallestAddressableSize(PTy->getElementType(), GV, TM);
   314:   }
   315:   case Type::PointerTyID:
   316:   case Type::HalfTyID:
   317:   case Type::FloatTyID:
   318:   case Type::DoubleTyID:
   319:   case Type::ByteTyID:
   320:   case Type::IntegerTyID: {
   321:     const DataLayout &DL = GV->getDataLayout();
   322:     // It is unfortunate that DL's function take non-const Type*.
   323:     return DL.getTypeAllocSize(const_cast<Type*>(Ty));
   324:   }
   325:   case Type::FunctionTyID:
   326:   case Type::VoidTyID:
   327:   case Type::BFloatTyID:
   328:   case Type::X86_FP80TyID:
   329:   case Type::FP128TyID:
   330:   case Type::PPC_FP128TyID:
   331:   case Type::LabelTyID:
   332:   case Type::MetadataTyID:
   333:   case Type::X86_AMXTyID:
   334:   case Type::TokenTyID:
   335:   case Type::TypedPointerTyID:
   336:   case Type::TargetExtTyID:
   337:     return 0;
   338:   }
   339: 
   340:   return 0;
   341: }
   342: 
   343: MCSection *HexagonTargetObjectFile::selectSmallSectionForGlobal(
   344:     const GlobalObject *GO, SectionKind Kind, const TargetMachine &TM) const {
   345:   const Type *GTy = GO->getValueType();
   346:   unsigned Size = getSmallestAddressableSize(GTy, GO, TM);
   347: 
   348:   // If we have -ffunction-section or -fdata-section then we should emit the
   349:   // global value to a unique section specifically for it... even for sdata.
   350:   bool EmitUniquedSection = TM.getDataSections();
```
- EN: It declares or implements routines such as ArrayType>, getSmallestAddressableSize, VectorType>, getDataLayout, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTargetObjectFile, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 ArrayType>, getSmallestAddressableSize, VectorType>, getDataLayout, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTargetObjectFile，说明了它与同级后端组件的连接关系。

### Lines 351-400 / 第 351-400 行

```cpp
   351: 
   352:   TRACE("Small data. Size(" << Size << ")");
   353:   // Handle Small Section classification here.
   354:   if (Kind.isBSS() || Kind.isBSSLocal()) {
   355:     // If -mno-sort-sda is not set, find out smallest accessible entity in
   356:     // declaration and add it to the section name string.
   357:     // Note. It does not track the actual usage of the value, only its de-
   358:     // claration. Also, compiler adds explicit pad fields to some struct
   359:     // declarations - they are currently counted towards smallest addres-
   360:     // sable entity.
   361:     if (NoSmallDataSorting) {
   362:       TRACE(" default sbss\n");
   363:       return SmallBSSSection;
   364:     }
   365: 
   366:     StringRef Prefix(".sbss");
   367:     SmallString<128> Name(Prefix);
   368:     Name.append(getSectionSuffixForSize(Size));
   369: 
   370:     if (EmitUniquedSection) {
   371:       Name.append(".");
   372:       Name.append(GO->getName());
   373:     }
   374:     TRACE(" unique sbss(" << Name << ")\n");
   375:     return getContext().getELFSection(Name.str(), ELF::SHT_NOBITS,
   376:                 ELF::SHF_WRITE | ELF::SHF_ALLOC | ELF::SHF_HEX_GPREL);
   377:   }
   378: 
   379:   if (Kind.isCommon()) {
   380:     // This is purely for LTO+Linker Script because commons don't really have a
   381:     // section. However, the BitcodeSectionWriter pass will query for the
   382:     // sections of commons (and the linker expects us to know their section) so
   383:     // we'll return one here.
   384:     if (NoSmallDataSorting)
   385:       return BSSSection;
   386: 
   387:     Twine Name = Twine(".scommon") + getSectionSuffixForSize(Size);
   388:     TRACE(" small COMMON (" << Name << ")\n");
   389: 
   390:     return getContext().getELFSection(Name.str(), ELF::SHT_NOBITS,
   391:                                       ELF::SHF_WRITE | ELF::SHF_ALLOC |
   392:                                       ELF::SHF_HEX_GPREL);
   393:   }
   394: 
   395:   // We could have changed sdata object to a constant... in this
   396:   // case the Kind could be wrong for it.
   397:   if (Kind.isMergeableConst()) {
   398:     TRACE(" const_object_as_data ");
   399:     const GlobalVariable *GVar = dyn_cast<GlobalVariable>(GO);
   400:     if (GVar->hasSection() && isSmallDataSection(GVar->getSection()))
```
- EN: It declares or implements routines such as TRACE, Prefix, Name, append, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 TRACE, Prefix, Name, append, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 401-450 / 第 401-450 行

```cpp
   401:       Kind = SectionKind::getData();
   402:   }
   403: 
   404:   if (Kind.isData()) {
   405:     if (NoSmallDataSorting) {
   406:       TRACE(" default sdata\n");
   407:       return SmallDataSection;
   408:     }
   409: 
   410:     StringRef Prefix(".sdata");
   411:     SmallString<128> Name(Prefix);
   412:     Name.append(getSectionSuffixForSize(Size));
   413: 
   414:     if (EmitUniquedSection) {
   415:       Name.append(".");
   416:       Name.append(GO->getName());
   417:     }
   418:     TRACE(" unique sdata(" << Name << ")\n");
   419:     return getContext().getELFSection(Name.str(), ELF::SHT_PROGBITS,
   420:                 ELF::SHF_WRITE | ELF::SHF_ALLOC | ELF::SHF_HEX_GPREL);
   421:   }
   422: 
   423:   TRACE("default ELF section\n");
   424:   // Otherwise, we work the same as ELF.
   425:   return TargetLoweringObjectFileELF::SelectSectionForGlobal(GO, Kind, TM);
   426: }
   427: 
   428: // Return the function that uses the lookup table. If there are more
   429: // than one live function that uses this look table, bail out and place
   430: // the lookup table in default section.
   431: const Function *
   432: HexagonTargetObjectFile::getLutUsedFunction(const GlobalObject *GO) const {
   433:   const Function *ReturnFn = nullptr;
   434:   for (const auto *U : GO->users()) {
   435:     // validate each instance of user to be a live function.
   436:     auto *I = dyn_cast<Instruction>(U);
   437:     if (!I)
   438:       continue;
   439:     auto *Bb = I->getParent();
   440:     if (!Bb)
   441:       continue;
   442:     auto *UserFn = Bb->getParent();
   443:     if (!ReturnFn)
   444:       ReturnFn = UserFn;
   445:     else if (ReturnFn != UserFn)
   446:       return nullptr;
   447:   }
   448:   return ReturnFn;
   449: }
   450: 
```
- EN: It declares or implements routines such as SectionKind::getData, TRACE, Prefix, Name, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTargetObjectFile, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 SectionKind::getData, TRACE, Prefix, Name, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTargetObjectFile，说明了它与同级后端组件的连接关系。

### Lines 451-462 / 第 451-462 行

```cpp
   451: MCSection *HexagonTargetObjectFile::selectSectionForLookupTable(
   452:     const GlobalObject *GO, const TargetMachine &TM, const Function *Fn) const {
   453: 
   454:   SectionKind Kind = SectionKind::getText();
   455:   // If the function has explicit section, place the lookup table in this
   456:   // explicit section.
   457:   if (Fn->hasSection())
   458:     return getExplicitSectionGlobal(Fn, Kind, TM);
   459: 
   460:   const auto *FuncObj = dyn_cast<GlobalObject>(Fn);
   461:   return SelectSectionForGlobal(FuncObj, Kind, TM);
   462: }
```
- EN: It declares or implements routines such as HexagonTargetObjectFile::selectSectionForLookupTable, SectionKind::getText, dyn_cast<GlobalObject>, SelectSectionForGlobal, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTargetObjectFile, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonTargetObjectFile::selectSectionForLookupTable, SectionKind::getText, dyn_cast<GlobalObject>, SelectSectionForGlobal 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTargetObjectFile，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- assembly/MC integration / 汇编/MC 集成
- MC-layer target description / MC 层目标描述
- target pipeline configuration / 目标流水线配置

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonTargetObjectFile.h, llvm/ADT/SmallString.h, llvm/ADT/StringRef.h, llvm/ADT/Twine.h, llvm/BinaryFormat/ELF.h, llvm/IR/DataLayout.h, llvm/IR/DerivedTypes.h, llvm/IR/GlobalObject.h, llvm/IR/GlobalValue.h, llvm/IR/GlobalVariable.h, ... (19 total)`
- Hexagon symbols / Hexagon 符号: `HexagonTargetObjectFile, HexagonTargetAsmInfo`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
