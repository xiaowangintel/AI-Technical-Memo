# BasicPtxBuilderInterface.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/LLVMIR/IR/BasicPtxBuilderInterface.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Defines the interface to build PTX (Parallel Thread Execution) from NVVM Ops automatically. It is used by NVVM to LLVM pass.
  - **CN**: 实现 LLVM 方言家族与 LLVM IR 桥接支持 的核心 IR、操作、类型、属性或接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- BasicPtxBuilderInterface.td - PTX builder interface -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Defines the interface to build PTX (Parallel Thread Execution) from NVVM Ops
// automatically. It is used by NVVM to LLVM pass.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件的高层描述。

### Lines 13-27
```cpp

#include "mlir/Dialect/LLVMIR/BasicPtxBuilderInterface.h"
#include "mlir/IR/BuiltinTypes.h"
#include "mlir/IR/Diagnostics.h"
#include "mlir/IR/Location.h"
#include "mlir/IR/MLIRContext.h"

#include "mlir/Support/LLVM.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/ADT/TypeSwitch.h"
#include "llvm/Support/DebugLog.h"
#include "llvm/Support/FormatVariadic.h"
#include "llvm/Support/LogicalResult.h"
#include "llvm/Support/Regex.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/LLVMIR/BasicPtxBuilderInterface.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Diagnostics.h`, `mlir/IR/Location.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/LLVMIR/BasicPtxBuilderInterface.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Diagnostics.h`, `mlir/IR/Location.h`。

### Lines 28-38
```cpp
#define DEBUG_TYPE "ptx-builder"

//===----------------------------------------------------------------------===//
// BasicPtxBuilderInterface
//===----------------------------------------------------------------------===//

#include "mlir/Dialect/LLVMIR/BasicPtxBuilderInterface.cpp.inc"

using namespace mlir;
using namespace NVVM;

```
- **EN**: Bridges to TableGen-generated declarations or definitions that expand MLIR op/type/interface metadata.
- **CN**: 桥接到由 TableGen 生成的声明或定义，用于展开 MLIR 操作/类型/接口元数据。

### Lines 39-58
```cpp
static constexpr int64_t kSharedMemorySpace = 3;

static FailureOr<char> getRegisterType(Type type, Location loc) {
  MLIRContext *ctx = type.getContext();
  auto i16 = IntegerType::get(ctx, 16);
  auto i32 = IntegerType::get(ctx, 32);
  auto f32 = Float32Type::get(ctx);

  auto getRegisterTypeForScalar = [&](Type type) -> FailureOr<char> {
    if (type.isInteger(1))
      return 'b';
    if (type.isInteger(16))
      return 'h';
    if (type.isInteger(32))
      return 'r';
    if (type.isInteger(64))
      return 'l';
    if (type.isF32())
      return 'f';
    if (type.isF64())
```
- **EN**: Implements logic around `getRegisterType`, `getContext`, `get`, `isInteger`, and 2 more symbols.
- **CN**: 围绕 `getRegisterType`, `getContext`, `get`, `isInteger`, and 2 more symbols 实现具体逻辑。

### Lines 59-78
```cpp
      return 'd';
    if (auto ptr = dyn_cast<LLVM::LLVMPointerType>(type)) {
      // Shared address spaces is addressed with 32-bit pointers.
      if (ptr.getAddressSpace() == kSharedMemorySpace) {
        return 'r';
      }
      return 'l';
    }
    // register type for struct is not supported.
    mlir::emitError(
        loc, "The register type could not be deduced from MLIR type. The ")
        << type
        << " is not supported. Supported types are:"
           "i1, i16, i32, i64, f32, f64,"
           "pointers.\nPlease use llvm.bitcast if you have different type. "
           "\nSee the constraints from here: "
           "https://docs.nvidia.com/cuda/inline-ptx-assembly/"
           "index.html#constraints";
    return failure();
  };
```
- **EN**: Implements logic around `getAddressSpace`, `emitError`, `failure`.
- **CN**: 围绕 `getAddressSpace`, `emitError`, `failure` 实现具体逻辑。

### Lines 79-90
```cpp

  // Packed registers
  if (auto v = dyn_cast<VectorType>(type)) {
    assert(v.getNumDynamicDims() == 0 && "Dynamic vectors are not supported");

    int64_t lanes = v.getNumElements();
    Type elem = v.getElementType();

    // Case 1. Single vector
    if (lanes <= 1)
      return getRegisterTypeForScalar(elem);

```
- **EN**: Implements logic around `dyn_cast`, `assert`, `getNumElements`, `getElementType`, and 1 more symbols.
- **CN**: 围绕 `dyn_cast`, `assert`, `getNumElements`, `getElementType`, and 1 more symbols 实现具体逻辑。

### Lines 91-110
```cpp
    // Case 2. Packed registers
    Type widened = elem;
    switch (lanes) {

    case 2:
      if (elem.isF16() || elem.isBF16()) // vector<2xf16>
        widened = f32;
      else if (elem.isFloat(8)) // vector<2xf8>
        widened = i16;
      break;
    case 4:
      if (elem.isInteger(8)) // vector<i8x4>
        widened = i32;
      else if (elem.isFloat(8)) // vector<f8x4>
        widened = f32;
      else if (elem.isFloat(4)) // vector<f4x4>
        widened = i16;
      break;
      // Other packing is not supported
    default:
```
- **EN**: Implements logic around `isF16`, `isFloat`, `isInteger`.
- **CN**: 围绕 `isF16`, `isFloat`, `isInteger` 实现具体逻辑。

### Lines 111-124
```cpp
      break;
    }
    return getRegisterTypeForScalar(widened);
  }

  return getRegisterTypeForScalar(type);
}

static FailureOr<char> getRegisterType(Value v, Location loc) {
  if (v.getDefiningOp<LLVM::ConstantOp>())
    return 'n';
  return getRegisterType(v.getType(), loc);
}

```
- **EN**: Implements logic around `getRegisterTypeForScalar`, `getRegisterType`, `ConstantOp>`.
- **CN**: 围绕 `getRegisterTypeForScalar`, `getRegisterType`, `ConstantOp>` 实现具体逻辑。

### Lines 125-134
```cpp
/// Extract every element of a struct value.
static SmallVector<Value> extractStructElements(PatternRewriter &rewriter,
                                                Location loc, Value structVal) {
  auto structTy = dyn_cast<LLVM::LLVMStructType>(structVal.getType());
  assert(structTy && "expected LLVM struct");

  SmallVector<Value> elems;
  for (unsigned i : llvm::seq<unsigned>(0, structTy.getBody().size()))
    elems.push_back(LLVM::ExtractValueOp::create(rewriter, loc, structVal, i));

```
- **EN**: Implements logic around `extractStructElements`, `getType`, `assert`, `seq`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `extractStructElements`, `getType`, `assert`, `seq`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 135-154
```cpp
  return elems;
}

LogicalResult PtxBuilder::insertValue(Value v, PTXRegisterMod itype) {
  LDBG() << v << "\t Modifier : " << itype << "\n";
  registerModifiers.push_back(itype);

  Location loc = interfaceOp->getLoc();
  auto getModifier = [&]() -> const char * {
    switch (itype) {
    case PTXRegisterMod::Read:
      return "";
    case PTXRegisterMod::Write:
      return "=";
    case PTXRegisterMod::ReadWrite:
      // "Read-Write modifier is not actually supported
      // Interface will change it to "=" later and add integer mapping
      return "+";
    }
    llvm_unreachable("Unknown PTX register modifier");
```
- **EN**: Implements logic around `insertValue`, `LDBG`, `push_back`, `getLoc`, and 1 more symbols; this block defines or attaches interface behavior.
- **CN**: 围绕 `insertValue`, `LDBG`, `push_back`, `getLoc`, and 1 more symbols 实现具体逻辑；该代码块定义或附加接口行为。

### Lines 155-166
```cpp
  };

  auto addValue = [&](Value v) {
    if (itype == PTXRegisterMod::Read) {
      ptxOperands.push_back(v);
      return;
    }
    if (itype == PTXRegisterMod::ReadWrite)
      ptxOperands.push_back(v);
    hasResult = true;
  };

```
- **EN**: Implements logic around `push_back`.
- **CN**: 围绕 `push_back` 实现具体逻辑。

### Lines 167-186
```cpp
  llvm::raw_string_ostream ss(registerConstraints);
  // Handle Structs
  if (auto stype = dyn_cast<LLVM::LLVMStructType>(v.getType())) {
    if (itype == PTXRegisterMod::Write) {
      addValue(v);
    }
    for (auto [idx, t] : llvm::enumerate(stype.getBody())) {
      if (itype != PTXRegisterMod::Write) {
        Value extractValue =
            LLVM::ExtractValueOp::create(rewriter, loc, v, idx);
        addValue(extractValue);
      }
      if (itype == PTXRegisterMod::ReadWrite) {
        ss << idx << ",";
      } else {
        FailureOr<char> regType = getRegisterType(t, loc);
        if (failed(regType))
          return rewriter.notifyMatchFailure(loc,
                                             "failed to get register type");
        ss << getModifier() << regType.value() << ",";
```
- **EN**: Implements logic around `ss`, `getType`, `addValue`, `enumerate`, and 5 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `ss`, `getType`, `addValue`, `enumerate`, and 5 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 187-199
```cpp
      }
    }
    return success();
  }
  // Handle Scalars
  addValue(v);
  FailureOr<char> regType = getRegisterType(v, loc);
  if (failed(regType))
    return rewriter.notifyMatchFailure(loc, "failed to get register type");
  ss << getModifier() << regType.value() << ",";
  return success();
}

```
- **EN**: Implements logic around `success`, `addValue`, `getRegisterType`, `failed`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `success`, `addValue`, `getRegisterType`, `failed`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 200-214
```cpp
/// Check if the operation needs to pack and unpack results.
static bool
needsPackUnpack(BasicPtxBuilderInterface interfaceOp,
                bool needsManualRegisterMapping,
                SmallVectorImpl<PTXRegisterMod> &registerModifiers) {
  if (needsManualRegisterMapping)
    return false;
  const unsigned writeOnlyVals = interfaceOp->getNumResults();
  const unsigned readWriteVals =
      llvm::count_if(registerModifiers, [](PTXRegisterMod m) {
        return m == PTXRegisterMod::ReadWrite;
      });
  return (writeOnlyVals + readWriteVals) > 1;
}

```
- **EN**: Implements logic around `needsPackUnpack`, `getNumResults`, `count_if`.
- **CN**: 围绕 `needsPackUnpack`, `getNumResults`, `count_if` 实现具体逻辑。

### Lines 215-225
```cpp
/// Pack the result types of the interface operation.
/// If the operation has multiple results, it packs them into a struct
/// type. Otherwise, it returns the original result types.
static SmallVector<Type>
packResultTypes(BasicPtxBuilderInterface interfaceOp,
                bool needsManualRegisterMapping,
                SmallVectorImpl<PTXRegisterMod> &registerModifiers,
                SmallVectorImpl<Value> &ptxOperands) {
  MLIRContext *ctx = interfaceOp->getContext();
  TypeRange resultRange = interfaceOp->getResultTypes();

```
- **EN**: Implements logic around `packResultTypes`, `getContext`, `getResultTypes`; this block defines or attaches interface behavior.
- **CN**: 围绕 `packResultTypes`, `getContext`, `getResultTypes` 实现具体逻辑；该代码块定义或附加接口行为。

### Lines 226-237
```cpp
  if (!needsPackUnpack(interfaceOp, needsManualRegisterMapping,
                       registerModifiers)) {
    // Single value path:
    if (interfaceOp->getResults().size() == 1)
      return SmallVector<Type>{resultRange.front()};

    // No declared results: if there is an RW, forward its type.
    for (auto [m, v] : llvm::zip(registerModifiers, ptxOperands))
      if (m == PTXRegisterMod::ReadWrite)
        return SmallVector<Type>{v.getType()};
  }

```
- **EN**: Implements logic around `needsPackUnpack`, `getResults`, `front`, `zip`, and 1 more symbols.
- **CN**: 围绕 `needsPackUnpack`, `getResults`, `front`, `zip`, and 1 more symbols 实现具体逻辑。

### Lines 238-247
```cpp
  SmallVector<Type> packed;
  for (auto [m, v] : llvm::zip(registerModifiers, ptxOperands))
    if (m == PTXRegisterMod::ReadWrite)
      packed.push_back(v.getType());
  for (Type t : resultRange)
    packed.push_back(t);

  if (packed.empty())
    return {};

```
- **EN**: Implements logic around `zip`, `push_back`, `empty`.
- **CN**: 围绕 `zip`, `push_back`, `empty` 实现具体逻辑。

### Lines 248-257
```cpp
  auto sTy = LLVM::LLVMStructType::getLiteral(ctx, packed, /*isPacked=*/false);
  return SmallVector<Type>{sTy};
}

/// Canonicalize the register constraints:
///  - Turn every "+X" into "=X"
///  - Append (at the very end) the 0-based indices of tokens that were "+X"
/// Examples:
///  "+f,+f,+r,=r,=r,r,r" -> "=f,=f,=r,=r,=r,r,r,0,1,2"
///  "+f,+f,+r,=r,=r"     -> "=f,=f,=r,=r,=r,0,1,2"
```
- **EN**: Implements logic around `getLiteral`.
- **CN**: 围绕 `getLiteral` 实现具体逻辑。

### Lines 258-275
```cpp
static std::string canonicalizeRegisterConstraints(llvm::StringRef csv) {
  SmallVector<llvm::StringRef> toks;
  SmallVector<std::string> out;
  SmallVector<unsigned> plusIdx;

  csv.split(toks, ',');
  out.reserve(toks.size() + 8);

  for (unsigned i = 0, e = toks.size(); i < e; ++i) {
    StringRef t = toks[i].trim();
    if (t.consume_front("+")) {
      plusIdx.push_back(i);
      out.push_back(("=" + t).str());
    } else {
      out.push_back(t.str());
    }
  }

```
- **EN**: Implements logic around `canonicalizeRegisterConstraints`, `split`, `reserve`, `size`, and 3 more symbols.
- **CN**: 围绕 `canonicalizeRegisterConstraints`, `split`, `reserve`, `size`, and 3 more symbols 实现具体逻辑。

### Lines 276-291
```cpp
  // Append indices of original "+X" tokens.
  for (unsigned idx : plusIdx)
    out.push_back(std::to_string(idx));

  // Join back to CSV.
  std::string result;
  result.reserve(csv.size() + plusIdx.size() * 2);
  llvm::raw_string_ostream os(result);
  for (size_t i = 0; i < out.size(); ++i) {
    if (i)
      os << ',';
    os << out[i];
  }
  return os.str();
}

```
- **EN**: Implements logic around `push_back`, `reserve`, `os`, `size`, and 1 more symbols.
- **CN**: 围绕 `push_back`, `reserve`, `os`, `size`, and 1 more symbols 实现具体逻辑。

### Lines 292-304
```cpp
constexpr llvm::StringLiteral kReadWritePrefix{"rw"};
constexpr llvm::StringLiteral kWriteOnlyPrefix{"w"};
constexpr llvm::StringLiteral kReadOnlyPrefix{"r"};

/// Returns a regex that matches {$rwN}, {$wN}, {$rN}
static llvm::Regex getPredicateMappingRegex() {
  llvm::Regex rx(llvm::formatv(R"(\{\$({0}|{1}|{2})([0-9]+)\})",
                               kReadWritePrefix, kWriteOnlyPrefix,
                               kReadOnlyPrefix)
                     .str());
  return rx;
}

```
- **EN**: Implements logic around `getPredicateMappingRegex`, `rx`, `str`.
- **CN**: 围绕 `getPredicateMappingRegex`, `rx`, `str` 实现具体逻辑。

### Lines 305-315
```cpp
void mlir::NVVM::countPlaceholderNumbers(
    StringRef ptxCode, llvm::SmallDenseSet<unsigned int> &seenRW,
    llvm::SmallDenseSet<unsigned int> &seenW,
    llvm::SmallDenseSet<unsigned int> &seenR,
    llvm::SmallVectorImpl<unsigned int> &rwNums,
    llvm::SmallVectorImpl<unsigned int> &wNums,
    llvm::SmallVectorImpl<unsigned int> &rNums) {

  llvm::Regex rx = getPredicateMappingRegex();
  StringRef rest = ptxCode;

```
- **EN**: Implements logic around `countPlaceholderNumbers`, `getPredicateMappingRegex`; this block encodes accelerator-specific execution or lowering rules.
- **CN**: 围绕 `countPlaceholderNumbers`, `getPredicateMappingRegex` 实现具体逻辑；该代码块编码加速器专用执行或 lowering 规则。

### Lines 316-331
```cpp
  SmallVector<StringRef, 3> m; // 0: full, 1: kind, 2: number
  while (!rest.empty() && rx.match(rest, &m)) {
    unsigned num = 0;
    (void)m[2].getAsInteger(10, num);
    // Insert it into the vector only the first time we see this number
    if (m[1].equals_insensitive(kReadWritePrefix)) {
      if (seenRW.insert(num).second)
        rwNums.push_back(num);
    } else if (m[1].equals_insensitive(kWriteOnlyPrefix)) {
      if (seenW.insert(num).second)
        wNums.push_back(num);
    } else {
      if (seenR.insert(num).second)
        rNums.push_back(num);
    }

```
- **EN**: Implements logic around `empty`, `getAsInteger`, `equals_insensitive`, `insert`, and 1 more symbols.
- **CN**: 围绕 `empty`, `getAsInteger`, `equals_insensitive`, `insert`, and 1 more symbols 实现具体逻辑。

### Lines 332-341
```cpp
    const size_t advance = (size_t)(m[0].data() - rest.data()) + m[0].size();
    rest = rest.drop_front(advance);
  }
}

/// Rewrites `{$rwN}`, `{$wN}`, and `{$rN}` placeholders in `ptxCode` into
/// compact `$K` indices:
///   - All `rw*` first (sorted by N),
///   - Then `w*`,
///   - Then `r*`.
```
- **EN**: Implements logic around `data`, `drop_front`.
- **CN**: 围绕 `data`, `drop_front` 实现具体逻辑。

### Lines 342-351
```cpp
/// If there a predicate, it comes always in the end.
/// Each number is assigned once; duplicates are ignored.
///
/// Example Input:
/// "{
///       reg .pred p;
///       setp.ge.s32 p,   {$r0}, {$r1};"
///       selp.s32 {$rw0}, {$r0}, {$r1}, p;
///       selp.s32 {$rw1}, {$r0}, {$r1}, p;
///       selp.s32 {$w0},  {$r0}, {$r1}, p;
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 352-361
```cpp
///       selp.s32 {$w1},  {$r0}, {$r1}, p;
/// }\n"
/// Example Output:
/// "{
///       reg .pred p;
///       setp.ge.s32 p, $4, $5;"
///       selp.s32 $0,   $4, $5, p;
///       selp.s32 $1,   $4, $5, p;
///       selp.s32 $2,   $4, $5, p;
///       selp.s32 $3,   $4, $5, p;
```
- **EN**: Documents the next declarations or records design constraints for the surrounding implementation.
- **CN**: 为接下来的声明提供说明，或记录周边实现的设计约束。

### Lines 362-374
```cpp
/// }\n"
static std::string rewriteAsmPlaceholders(llvm::StringRef ptxCode) {
  llvm::SmallDenseSet<unsigned> seenRW, seenW, seenR;
  llvm::SmallVector<unsigned> rwNums, wNums, rNums;

  // Step 1. Count Register Placeholder numbers
  countPlaceholderNumbers(ptxCode, seenRW, seenW, seenR, rwNums, wNums, rNums);

  // Step 2. Sort the Register Placeholder numbers
  llvm::sort(rwNums);
  llvm::sort(wNums);
  llvm::sort(rNums);

```
- **EN**: Implements logic around `rewriteAsmPlaceholders`, `countPlaceholderNumbers`, `sort`.
- **CN**: 围绕 `rewriteAsmPlaceholders`, `countPlaceholderNumbers`, `sort` 实现具体逻辑。

### Lines 375-384
```cpp
  // Step 3. Create mapping from original to new IDs
  llvm::DenseMap<unsigned, unsigned> rwMap, wMap, rMap;
  unsigned nextId = 0;
  for (unsigned n : rwNums)
    rwMap[n] = nextId++;
  for (unsigned n : wNums)
    wMap[n] = nextId++;
  for (unsigned n : rNums)
    rMap[n] = nextId++;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 385-396
```cpp
  // Step 4. Rewrite the PTX code with new IDs
  std::string out;
  out.reserve(ptxCode.size());
  size_t prev = 0;
  StringRef rest = ptxCode;
  SmallVector<StringRef, 3> matches;
  llvm::Regex rx = getPredicateMappingRegex();
  while (!rest.empty() && rx.match(rest, &matches)) {
    // Compute absolute match bounds in the original buffer.
    size_t absStart = (size_t)(matches[0].data() - ptxCode.data());
    size_t absEnd = absStart + matches[0].size();

```
- **EN**: Implements logic around `reserve`, `getPredicateMappingRegex`, `empty`, `data`, and 1 more symbols; this block moves data between tensor-style values and explicit buffers.
- **CN**: 围绕 `reserve`, `getPredicateMappingRegex`, `empty`, `data`, and 1 more symbols 实现具体逻辑；该代码块在张量风格值与显式缓冲区之间移动数据。

### Lines 397-410
```cpp
    // Emit text before the match.
    out.append(ptxCode.data() + prev, ptxCode.data() + absStart);

    // Emit compact $K
    unsigned num = 0;
    (void)matches[2].getAsInteger(10, num);
    unsigned id = 0;
    if (matches[1].equals_insensitive(kReadWritePrefix))
      id = rwMap.lookup(num);
    else if (matches[1].equals_insensitive(kWriteOnlyPrefix))
      id = wMap.lookup(num);
    else
      id = rMap.lookup(num);

```
- **EN**: Implements logic around `append`, `getAsInteger`, `equals_insensitive`, `lookup`.
- **CN**: 围绕 `append`, `getAsInteger`, `equals_insensitive`, `lookup` 实现具体逻辑。

### Lines 411-420
```cpp
    out.push_back('$');
    out += std::to_string(id);

    prev = absEnd;

    const size_t advance =
        (size_t)(matches[0].data() - rest.data()) + matches[0].size();
    rest = rest.drop_front(advance);
  }

```
- **EN**: Implements logic around `push_back`, `to_string`, `data`, `drop_front`.
- **CN**: 围绕 `push_back`, `to_string`, `data`, `drop_front` 实现具体逻辑。

### Lines 421-430
```cpp
  // Step 5. Tail.
  out.append(ptxCode.data() + prev, ptxCode.data() + ptxCode.size());
  return out;
}

/// Return the constraint index of the predicate operand.  The predicate
/// constraint ("b") is always the last non-tied token in the canonicalized
/// constraint string.  Tied constraints (digit-only tokens from read-write
/// canonicalization) are appended at the end, so we walk backwards to skip
/// them.
```
- **EN**: Implements logic around `append`.
- **CN**: 围绕 `append` 实现具体逻辑。

### Lines 431-440
```cpp
static unsigned getPredicateConstraintIndex(StringRef constraints) {
  SmallVector<StringRef> tokens;
  constraints.split(tokens, ',');
  assert(!tokens.empty() && "expected at least a predicate constraint");

  auto isTiedConstraint = [](StringRef tok) {
    unsigned idx;
    return !tok.trim().getAsInteger(10, idx);
  };

```
- **EN**: Implements logic around `getPredicateConstraintIndex`, `split`, `assert`, `trim`.
- **CN**: 围绕 `getPredicateConstraintIndex`, `split`, `assert`, `trim` 实现具体逻辑。

### Lines 441-450
```cpp
  size_t numTied = 0;
  for (StringRef tok : llvm::reverse(tokens)) {
    if (!isTiedConstraint(tok))
      break;
    ++numTied;
  }
  assert(numTied < tokens.size() && "all constraints are tied");
  return tokens.size() - numTied - 1;
}

```
- **EN**: Implements logic around `reverse`, `isTiedConstraint`, `assert`, `size`.
- **CN**: 围绕 `reverse`, `isTiedConstraint`, `assert`, `size` 实现具体逻辑。

### Lines 451-463
```cpp
LLVM::InlineAsmOp PtxBuilder::build() {
  auto asmDialectAttr = LLVM::AsmDialectAttr::get(interfaceOp->getContext(),
                                                  LLVM::AsmDialect::AD_ATT);

  SmallVector<Type> resultTypes = packResultTypes(
      interfaceOp, needsManualRegisterMapping, registerModifiers, ptxOperands);

  // Remove the last comma from the constraints string.
  if (!registerConstraints.empty() &&
      registerConstraints[registerConstraints.size() - 1] == ',')
    registerConstraints.pop_back();
  registerConstraints = canonicalizeRegisterConstraints(registerConstraints);

```
- **EN**: Implements logic around `build`, `get`, `packResultTypes`, `empty`, and 3 more symbols.
- **CN**: 围绕 `build`, `get`, `packResultTypes`, `empty`, and 3 more symbols 实现具体逻辑。

### Lines 464-476
```cpp
  std::string ptxInstruction = interfaceOp.getPtx();
  if (!needsManualRegisterMapping)
    ptxInstruction = rewriteAsmPlaceholders(ptxInstruction);

  // Add the predicate to the asm string.
  if (interfaceOp.getPredicate().has_value() &&
      interfaceOp.getPredicate().value()) {
    unsigned predIdx = getPredicateConstraintIndex(registerConstraints);
    std::string predicateStr = "@%";
    predicateStr += std::to_string(predIdx);
    ptxInstruction = predicateStr + " " + ptxInstruction;
  }

```
- **EN**: Implements logic around `getPtx`, `rewriteAsmPlaceholders`, `getPredicate`, `getPredicateConstraintIndex`, and 1 more symbols.
- **CN**: 围绕 `getPtx`, `rewriteAsmPlaceholders`, `getPredicate`, `getPredicateConstraintIndex`, and 1 more symbols 实现具体逻辑。

### Lines 477-492
```cpp
  // Tablegen doesn't accept $, so we use %, but inline assembly uses $.
  // Replace all % with $
  llvm::replace(ptxInstruction, '%', '$');

  return LLVM::InlineAsmOp::create(
      rewriter, interfaceOp->getLoc(),
      /*result types=*/resultTypes,
      /*operands=*/ptxOperands,
      /*asm_string=*/ptxInstruction,
      /*constraints=*/registerConstraints.data(),
      /*has_side_effects=*/interfaceOp.hasSideEffect(),
      /*is_align_stack=*/false, LLVM::TailCallKind::None,
      /*asm_dialect=*/asmDialectAttr,
      /*operand_attrs=*/ArrayAttr());
}

```
- **EN**: Implements logic around `replace`, `create`, `getLoc`, `data`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `replace`, `create`, `getLoc`, `data`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 493-502
```cpp
void PtxBuilder::buildAndReplaceOp() {
  LLVM::InlineAsmOp inlineAsmOp = build();
  LDBG() << "\n Generated PTX \n\t" << inlineAsmOp;

  // Case 0: no result at all → just erase wrapper op.
  if (!hasResult) {
    rewriter.eraseOp(interfaceOp);
    return;
  }

```
- **EN**: Implements logic around `buildAndReplaceOp`, `build`, `LDBG`, `eraseOp`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `buildAndReplaceOp`, `build`, `LDBG`, `eraseOp` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 503-522
```cpp
  if (needsManualRegisterMapping) {
    rewriter.replaceOp(interfaceOp, inlineAsmOp->getResults());
    return;
  }

  // Case 1: Simple path, single scalar inline asm result.
  if (!needsPackUnpack(interfaceOp, needsManualRegisterMapping,
                       registerModifiers)) {
    // Sub-case 1a: the wrapper op has a declared result -- replace it
    // directly with the inline asm result.
    if (interfaceOp->getNumResults() > 0) {
      rewriter.replaceOp(interfaceOp, inlineAsmOp->getResults());
      return;
    }
    // Sub-case 1b: RW-only, no declared result. The inline asm produces a
    // single value that represents the post-asm value of the read-write
    // operand; forward it to that operand's uses and erase the wrapper.
    if (inlineAsmOp->getNumResults() > 0) {
      Value postAsm = inlineAsmOp->getResult(0);
      for (auto [m, v] : llvm::zip(registerModifiers, ptxOperands)) {
```
- **EN**: Implements logic around `replaceOp`, `needsPackUnpack`, `getNumResults`, `getResult`, and 1 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `replaceOp`, `needsPackUnpack`, `getNumResults`, `getResult`, and 1 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 523-535
```cpp
        if (m != PTXRegisterMod::ReadWrite)
          continue;
        v.replaceUsesWithIf(postAsm, [&](OpOperand &use) {
          Operation *owner = use.getOwner();
          return owner != interfaceOp && owner != inlineAsmOp;
        });
        break;
      }
    }
    rewriter.eraseOp(interfaceOp);
    return;
  }

```
- **EN**: Implements logic around `replaceUsesWithIf`, `getOwner`, `eraseOp`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `replaceUsesWithIf`, `getOwner`, `eraseOp` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 536-546
```cpp
  const bool hasRW = llvm::any_of(registerModifiers, [](PTXRegisterMod m) {
    return m == PTXRegisterMod::ReadWrite;
  });

  // All multi-value paths produce a single struct result we need to unpack.
  assert(LLVM::LLVMStructType::classof(inlineAsmOp.getResultTypes().front()) &&
         "expected struct return for multi-result inline asm");
  Value structVal = inlineAsmOp.getResult(0);
  SmallVector<Value> unpacked =
      extractStructElements(rewriter, interfaceOp->getLoc(), structVal);

```
- **EN**: Implements logic around `any_of`, `assert`, `getResult`, `extractStructElements`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `any_of`, `assert`, `getResult`, `extractStructElements` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 547-566
```cpp
  // Case 2: only declared results (no RW): replace the op with all unpacked.
  if (!hasRW && interfaceOp->getResults().size() > 0) {
    rewriter.replaceOp(interfaceOp, unpacked);
    return;
  }

  // Case 3: RW-only (no declared results): update RW uses and erase wrapper.
  if (hasRW && interfaceOp->getResults().size() == 0) {
    unsigned idx = 0;
    for (auto [m, v] : llvm::zip(registerModifiers, ptxOperands)) {
      if (m != PTXRegisterMod::ReadWrite)
        continue;
      Value repl = unpacked[idx++];
      v.replaceUsesWithIf(repl, [&](OpOperand &use) {
        Operation *owner = use.getOwner();
        return owner != interfaceOp && owner != inlineAsmOp;
      });
    }
    rewriter.eraseOp(interfaceOp);
    return;
```
- **EN**: Implements logic around `getResults`, `replaceOp`, `zip`, `replaceUsesWithIf`, and 2 more symbols; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `getResults`, `replaceOp`, `zip`, `replaceUsesWithIf`, and 2 more symbols 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

### Lines 567-586
```cpp
  }

  // Case 4: mixed (RW + declared results).
  {
    // First rewrite RW operands in place.
    unsigned idx = 0;
    for (auto [m, v] : llvm::zip(registerModifiers, ptxOperands)) {
      if (m != PTXRegisterMod::ReadWrite)
        continue;
      Value repl = unpacked[idx++];
      v.replaceUsesWithIf(repl, [&](OpOperand &use) {
        Operation *owner = use.getOwner();
        return owner != interfaceOp && owner != inlineAsmOp;
      });
    }
    // The remaining unpacked values correspond to the declared results.
    SmallVector<Value> tail;
    tail.reserve(unpacked.size() - idx);
    for (unsigned i = idx, e = unpacked.size(); i < e; ++i)
      tail.push_back(unpacked[i]);
```
- **EN**: Implements logic around `zip`, `replaceUsesWithIf`, `getOwner`, `reserve`, and 2 more symbols.
- **CN**: 围绕 `zip`, `replaceUsesWithIf`, `getOwner`, `reserve`, and 2 more symbols 实现具体逻辑。

### Lines 587-590
```cpp

    rewriter.replaceOp(interfaceOp, tail);
  }
}
```
- **EN**: Implements logic around `replaceOp`; this block applies MLIR pattern-rewrite or conversion infrastructure.
- **CN**: 围绕 `replaceOp` 实现具体逻辑；该代码块应用 MLIR 模式重写或方言转换基础设施。

## Key Concepts / 关键概念

- **Dialect IR modeling / 方言 IR 建模**:
  - **EN**: Defines operations, attributes, types, verifiers, parsers, and printers for a dialect.
  - **CN**: 定义方言的操作、属性、类型、验证器、解析器与打印器。
- **LLVM dialect bridging / LLVM 方言桥接**:
  - **EN**: Maps MLIR constructs to LLVM-compatible types, ops, intrinsics, and metadata.
  - **CN**: 将 MLIR 构造映射到兼容 LLVM 的类型、操作、Intrinsic 与元数据。
- **Interface dispatch / 接口分派**:
  - **EN**: Attaches shared behavior to different ops or types through MLIR interfaces.
  - **CN**: 通过 MLIR 接口把共享行为附着到不同操作或类型上。
- **Type/attribute storage / 类型/属性存储**:
  - **EN**: Uses uniqued storage and parser/printer hooks for extensible compile-time metadata.
  - **CN**: 使用唯一化存储以及解析/打印钩子来表示可扩展的编译期元数据。
- **LLVM interop / LLVM 互操作**:
  - **EN**: Bridges MLIR concepts to LLVM-compatible data structures, intrinsics, or codegen expectations.
  - **CN**: 把 MLIR 概念桥接到 LLVM 兼容的数据结构、intrinsic 或代码生成预期。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/LLVMIR/BasicPtxBuilderInterface.h`, `mlir/IR/BuiltinTypes.h`, `mlir/IR/Diagnostics.h`, `mlir/IR/Location.h`, `mlir/IR/MLIRContext.h`, `mlir/Support/LLVM.h`, `llvm/ADT/StringExtras.h`, `llvm/ADT/TypeSwitch.h`, `llvm/Support/DebugLog.h`, `llvm/Support/FormatVariadic.h` ... (+3 more)
- **Subsystem categories / 子系统类别**: MLIR IR core abstractions / MLIR IR 核心抽象 (4), LLVM support-library helpers / LLVM Support 库辅助功能 (4), other MLIR dialect declarations / 其他 MLIR 方言声明 (2), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (2), MLIR support-library helpers / MLIR Support 库辅助功能 (1)
