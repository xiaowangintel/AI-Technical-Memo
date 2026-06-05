# AMDGPULibFunc.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPULibFunc.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPULibFunc in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPULibFunc 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: File banner, includes, and setup
```cpp
//===-- AMDGPULibFunc.h ----------------------------------------*- C++ -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef _AMDGPU_LIBFUNC_H_
#define _AMDGPU_LIBFUNC_H_

#include "llvm/ADT/StringRef.h"
#include <memory>

namespace llvm {

class FunctionCallee;
class FunctionType;
class Function;
class Module;
class Type;

class AMDGPULibFuncBase {
public:
  enum EFuncId {
    EI_NONE,

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `FunctionCallee`, `FunctionType`, `Function`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`FunctionCallee`, `FunctionType`, `Function`。

### Lines 28-61: Type declarations and aliases
```cpp
    // IMPORTANT: enums below should go in ascending by 1 value order
    // because they are used as indexes in the mangling rules table.
    // don't use explicit value assignment.
    //
    // There are two types of library functions: those with mangled
    // name and those with unmangled name. The enums for the library
    // functions with mangled name are defined before enums for the
    // library functions with unmangled name. The enum for the last
    // library function with mangled name is EI_LAST_MANGLED.
    //
    // Library functions with mangled name.
    EI_ABS,
    EI_ABS_DIFF,
    EI_ACOS,
    EI_ACOSH,
    EI_ACOSPI,
    EI_ADD_SAT,
    EI_ALL,
    EI_ANY,
    EI_ASIN,
    EI_ASINH,
    EI_ASINPI,
    EI_ASYNC_WORK_GROUP_COPY,
    EI_ASYNC_WORK_GROUP_STRIDED_COPY,
    EI_ATAN,
    EI_ATAN2,
    EI_ATAN2PI,
    EI_ATANH,
    EI_ATANPI,
    EI_ATOMIC_ADD,
    EI_ATOMIC_AND,
    EI_ATOMIC_CMPXCHG,
    EI_ATOMIC_DEC,
    EI_ATOMIC_INC,
```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。

### Lines 62-95: Implementation details and local logic
```cpp
    EI_ATOMIC_MAX,
    EI_ATOMIC_MIN,
    EI_ATOMIC_OR,
    EI_ATOMIC_SUB,
    EI_ATOMIC_XCHG,
    EI_ATOMIC_XOR,
    EI_BITSELECT,
    EI_CBRT,
    EI_CEIL,
    EI_CLAMP,
    EI_CLZ,
    EI_COMMIT_READ_PIPE,
    EI_COMMIT_WRITE_PIPE,
    EI_COPYSIGN,
    EI_COS,
    EI_COSH,
    EI_COSPI,
    EI_CROSS,
    EI_CTZ,
    EI_DEGREES,
    EI_DISTANCE,
    EI_DIVIDE,
    EI_DOT,
    EI_ERF,
    EI_ERFC,
    EI_EXP,
    EI_EXP10,
    EI_EXP2,
    EI_EXPM1,
    EI_FABS,
    EI_FAST_DISTANCE,
    EI_FAST_LENGTH,
    EI_FAST_NORMALIZE,
    EI_FDIM,
```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 96-129: Implementation details and local logic
```cpp
    EI_FLOOR,
    EI_FMA,
    EI_FMAX,
    EI_FMIN,
    EI_FMOD,
    EI_FRACT,
    EI_FREXP,
    EI_GET_IMAGE_ARRAY_SIZE,
    EI_GET_IMAGE_CHANNEL_DATA_TYPE,
    EI_GET_IMAGE_CHANNEL_ORDER,
    EI_GET_IMAGE_DIM,
    EI_GET_IMAGE_HEIGHT,
    EI_GET_IMAGE_WIDTH,
    EI_GET_PIPE_MAX_PACKETS,
    EI_GET_PIPE_NUM_PACKETS,
    EI_HADD,
    EI_HYPOT,
    EI_ILOGB,
    EI_ISEQUAL,
    EI_ISFINITE,
    EI_ISGREATER,
    EI_ISGREATEREQUAL,
    EI_ISINF,
    EI_ISLESS,
    EI_ISLESSEQUAL,
    EI_ISLESSGREATER,
    EI_ISNAN,
    EI_ISNORMAL,
    EI_ISNOTEQUAL,
    EI_ISORDERED,
    EI_ISUNORDERED,
    EI_LDEXP,
    EI_LENGTH,
    EI_LGAMMA,
```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 130-163: Implementation details and local logic
```cpp
    EI_LGAMMA_R,
    EI_LOG,
    EI_LOG10,
    EI_LOG1P,
    EI_LOG2,
    EI_LOGB,
    EI_MAD,
    EI_MAD24,
    EI_MAD_HI,
    EI_MAD_SAT,
    EI_MAX,
    EI_MAXMAG,
    EI_MIN,
    EI_MINMAG,
    EI_MIX,
    EI_MODF,
    EI_MUL24,
    EI_MUL_HI,
    EI_NAN,
    EI_NEXTAFTER,
    EI_NORMALIZE,
    EI_POPCOUNT,
    EI_POW,
    EI_POW_FAST,
    EI_POWN,
    EI_POWN_FAST,
    EI_POWR,
    EI_POWR_FAST,
    EI_PREFETCH,
    EI_RADIANS,
    EI_RECIP,
    EI_REMAINDER,
    EI_REMQUO,
    EI_RESERVE_READ_PIPE,
```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 164-197: Implementation details and local logic
```cpp
    EI_RESERVE_WRITE_PIPE,
    EI_RHADD,
    EI_RINT,
    EI_ROOTN,
    EI_ROOTN_FAST,
    EI_ROTATE,
    EI_ROUND,
    EI_RSQRT,
    EI_SELECT,
    EI_SHUFFLE,
    EI_SHUFFLE2,
    EI_SIGN,
    EI_SIGNBIT,
    EI_SIN,
    EI_SINCOS,
    EI_SINH,
    EI_SINPI,
    EI_SMOOTHSTEP,
    EI_SQRT,
    EI_STEP,
    EI_SUB_GROUP_BROADCAST,
    EI_SUB_GROUP_COMMIT_READ_PIPE,
    EI_SUB_GROUP_COMMIT_WRITE_PIPE,
    EI_SUB_GROUP_REDUCE_ADD,
    EI_SUB_GROUP_REDUCE_MAX,
    EI_SUB_GROUP_REDUCE_MIN,
    EI_SUB_GROUP_RESERVE_READ_PIPE,
    EI_SUB_GROUP_RESERVE_WRITE_PIPE,
    EI_SUB_GROUP_SCAN_EXCLUSIVE_ADD,
    EI_SUB_GROUP_SCAN_EXCLUSIVE_MAX,
    EI_SUB_GROUP_SCAN_EXCLUSIVE_MIN,
    EI_SUB_GROUP_SCAN_INCLUSIVE_ADD,
    EI_SUB_GROUP_SCAN_INCLUSIVE_MAX,
    EI_SUB_GROUP_SCAN_INCLUSIVE_MIN,
```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 198-231: Implementation details and local logic
```cpp
    EI_SUB_SAT,
    EI_TAN,
    EI_TANH,
    EI_TANPI,
    EI_TGAMMA,
    EI_TRUNC,
    EI_UPSAMPLE,
    EI_VEC_STEP,
    EI_VSTORE,
    EI_VSTORE16,
    EI_VSTORE2,
    EI_VSTORE3,
    EI_VSTORE4,
    EI_VSTORE8,
    EI_WORK_GROUP_COMMIT_READ_PIPE,
    EI_WORK_GROUP_COMMIT_WRITE_PIPE,
    EI_WORK_GROUP_REDUCE_ADD,
    EI_WORK_GROUP_REDUCE_MAX,
    EI_WORK_GROUP_REDUCE_MIN,
    EI_WORK_GROUP_RESERVE_READ_PIPE,
    EI_WORK_GROUP_RESERVE_WRITE_PIPE,
    EI_WORK_GROUP_SCAN_EXCLUSIVE_ADD,
    EI_WORK_GROUP_SCAN_EXCLUSIVE_MAX,
    EI_WORK_GROUP_SCAN_EXCLUSIVE_MIN,
    EI_WORK_GROUP_SCAN_INCLUSIVE_ADD,
    EI_WORK_GROUP_SCAN_INCLUSIVE_MAX,
    EI_WORK_GROUP_SCAN_INCLUSIVE_MIN,
    EI_WRITE_IMAGEF,
    EI_WRITE_IMAGEI,
    EI_WRITE_IMAGEUI,
    EI_NCOS,
    EI_NEXP2,
    EI_NFMA,
    EI_NLOG2,
```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 232-257: Type declarations and aliases
```cpp
    EI_NRCP,
    EI_NRSQRT,
    EI_NSIN,
    EI_NSQRT,
    EI_FTZ,
    EI_FLDEXP,
    EI_CLASS,
    EI_RCBRT,
    EI_LAST_MANGLED =
        EI_RCBRT, /* The last library function with mangled name */

    // Library functions with unmangled name.
    EI_READ_PIPE_2,
    EI_READ_PIPE_4,
    EI_WRITE_PIPE_2,
    EI_WRITE_PIPE_4,

    EX_INTRINSICS_COUNT
  };

  enum ENamePrefix {
    NOPFX,
    NATIVE,
    HALF
  };

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic. Main symbols: `ENamePrefix`.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。 主要符号：`ENamePrefix`。

### Lines 258-289: Type declarations and aliases
```cpp
  enum EType {
    B8  = 1,
    B16 = 2,
    B32 = 3,
    B64 = 4,
    SIZE_MASK = 7,
    FLOAT = 0x10,
    INT   = 0x20,
    UINT  = 0x30,
    BASE_TYPE_MASK = 0x30,
    U8  =  UINT | B8,
    U16 =  UINT | B16,
    U32 =  UINT | B32,
    U64 =  UINT | B64,
    I8  =   INT | B8,
    I16 =   INT | B16,
    I32 =   INT | B32,
    I64 =   INT | B64,
    F16 = FLOAT | B16,
    F32 = FLOAT | B32,
    F64 = FLOAT | B64,
    IMG1DA = 0x80,
    IMG1DB,
    IMG2DA,
    IMG1D,
    IMG2D,
    IMG3D,
    SAMPLER,
    EVENT,
    DUMMY
  };

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic. Main symbols: `EType`.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。 主要符号：`EType`。

### Lines 290-322: Declares struct Param
```cpp
  enum EPtrKind {
    BYVALUE = 0,
    ADDR_SPACE = 0xF, // Address space takes value 0x1 ~ 0xF.
    CONST      = 0x10,
    VOLATILE   = 0x20
  };

  struct Param {
    unsigned char ArgType = 0;
    unsigned char VectorSize = 1;
    unsigned char PtrKind = 0;

    unsigned char Reserved = 0;

    void reset() {
      ArgType = 0;
      VectorSize = 1;
      PtrKind = 0;
    }

    static Param getIntN(unsigned char NumElts) {
      return Param{I32, NumElts, 0, 0};
    }

    static Param getFromTy(Type *Ty, bool Signed);

    template <typename Stream>
    void mangleItanium(Stream& os);
  };
  static bool isMangled(EFuncId Id) {
    return static_cast<unsigned>(Id) <= static_cast<unsigned>(EI_LAST_MANGLED);
  }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `EPtrKind`, `Param`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`EPtrKind`, `Param`。

### Lines 323-355: Declares class AMDGPULibFuncImpl
```cpp
  static unsigned getEPtrKindFromAddrSpace(unsigned AS) {
    assert(((AS + 1) & ~ADDR_SPACE) == 0);
    return AS + 1;
  }

  static unsigned getAddrSpaceFromEPtrKind(unsigned Kind) {
    Kind = Kind & ADDR_SPACE;
    assert(Kind >= 1);
    return Kind - 1;
  }
};

class AMDGPULibFuncImpl : public AMDGPULibFuncBase {
public:
  AMDGPULibFuncImpl() = default;
  virtual ~AMDGPULibFuncImpl() = default;

  /// Get unmangled name for mangled library function and name for unmangled
  /// library function.
  virtual std::string getName() const = 0;
  virtual unsigned getNumArgs() const = 0;
  EFuncId getId() const { return FuncId; }
  ENamePrefix getPrefix() const { return FKind; }

  bool isMangled() const { return AMDGPULibFuncBase::isMangled(FuncId); }

  void setId(EFuncId id) { FuncId = id; }
  virtual bool parseFuncName(StringRef &mangledName) = 0;

  /// \return The mangled function name for mangled library functions
  /// and unmangled function name for unmangled library functions.
  virtual std::string mangle() const = 0;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPULibFuncImpl`, `AMDGPULibFuncBase::isMangled`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPULibFuncImpl`, `AMDGPULibFuncBase::isMangled`。

### Lines 356-381: Declares class AMDGPULibFunc
```cpp
  void setName(StringRef N) { Name = std::string(N); }
  void setPrefix(ENamePrefix pfx) { FKind = pfx; }

  virtual FunctionType *getFunctionType(const Module &M) const = 0;

protected:
  EFuncId FuncId;
  std::string Name;
  ENamePrefix FKind = NOPFX;
};

/// Wrapper class for AMDGPULIbFuncImpl
class AMDGPULibFunc : public AMDGPULibFuncBase {
public:
  explicit AMDGPULibFunc() : Impl(std::unique_ptr<AMDGPULibFuncImpl>()) {}
  AMDGPULibFunc(const AMDGPULibFunc &F);
  /// Clone a mangled library func with the Id \p Id and argument info from \p
  /// CopyFrom.
  explicit AMDGPULibFunc(EFuncId Id, const AMDGPULibFunc &CopyFrom);
  explicit AMDGPULibFunc(EFuncId Id, FunctionType *FT, bool SignedInts);

  /// Construct an unmangled library function on the fly.
  explicit AMDGPULibFunc(StringRef FName, FunctionType *FT);

  AMDGPULibFunc &operator=(const AMDGPULibFunc &F);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPULibFunc`, `std::string`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPULibFunc`, `std::string`。

### Lines 382-414: Defines getName
```cpp
  /// Get unmangled name for mangled library function and name for unmangled
  /// library function.
  std::string getName() const { return Impl->getName(); }
  unsigned getNumArgs() const { return Impl->getNumArgs(); }
  EFuncId getId() const { return Impl->getId(); }
  ENamePrefix getPrefix() const { return Impl->getPrefix(); }
  /// Get leading parameters for mangled lib functions.
  Param *getLeads();
  const Param *getLeads() const;

  bool isMangled() const { return Impl->isMangled(); }
  void setId(EFuncId Id) { Impl->setId(Id); }
  bool parseFuncName(StringRef &MangledName) {
    return Impl->parseFuncName(MangledName);
  }

  /// Return true if it's legal to splat a scalar value passed in parameter \p
  /// ArgIdx to a vector argument.
  bool allowsImplicitVectorSplat(int ArgIdx) const {
    switch (getId()) {
    case EI_LDEXP:
      return ArgIdx == 1;
    case EI_FMIN:
    case EI_FMAX:
      return true;
    default:
      return false;
    }
  }

  // Validate the call type matches the expected libfunc type.
  bool isCompatibleSignature(const Module &M, const FunctionType *FuncTy) const;

```
**EN:** This section contains concrete logic for getName. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getName 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 415-446: Declares class AMDGPUMangledLibFunc
```cpp
  /// \return The mangled function name for mangled library functions
  /// and unmangled function name for unmangled library functions.
  std::string mangle() const { return Impl->mangle(); }

  void setName(StringRef N) { Impl->setName(N); }
  void setPrefix(ENamePrefix PFX) { Impl->setPrefix(PFX); }

  FunctionType *getFunctionType(const Module &M) const {
    return Impl->getFunctionType(M);
  }
  static Function *getFunction(llvm::Module *M, const AMDGPULibFunc &fInfo);

  static FunctionCallee getOrInsertFunction(llvm::Module *M,
                                            const AMDGPULibFunc &fInfo);
  static bool parse(StringRef MangledName, AMDGPULibFunc &Ptr);

private:
  /// Initialize as a mangled library function.
  void initMangled();
  std::unique_ptr<AMDGPULibFuncImpl> Impl;
};

class AMDGPUMangledLibFunc : public AMDGPULibFuncImpl {
public:
  Param Leads[2];

  explicit AMDGPUMangledLibFunc();
  explicit AMDGPUMangledLibFunc(EFuncId id,
                                const AMDGPUMangledLibFunc &copyFrom);
  explicit AMDGPUMangledLibFunc(EFuncId id, FunctionType *FT,
                                bool SignedInts = true);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUMangledLibFunc`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUMangledLibFunc`。

### Lines 447-470: Declares class AMDGPUUnmangledLibFunc
```cpp
  std::string getName() const override;
  unsigned getNumArgs() const override;
  FunctionType *getFunctionType(const Module &M) const override;
  static StringRef getUnmangledName(StringRef MangledName);

  bool parseFuncName(StringRef &mangledName) override;

  // Methods for support type inquiry through isa, cast, and dyn_cast:
  static bool classof(const AMDGPULibFuncImpl *F) { return F->isMangled(); }

  std::string mangle() const override;

private:
  std::string mangleNameItanium() const;

  std::string mangleName(StringRef Name) const;
  bool parseUnmangledName(StringRef MangledName);

  template <typename Stream> void writeName(Stream &OS) const;
};

class AMDGPUUnmangledLibFunc : public AMDGPULibFuncImpl {
  FunctionType *FuncTy;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUUnmangledLibFunc`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUUnmangledLibFunc`。

### Lines 471-493: Preprocessor guards and macros
```cpp
public:
  explicit AMDGPUUnmangledLibFunc();
  explicit AMDGPUUnmangledLibFunc(StringRef FName, FunctionType *FT) {
    Name = std::string(FName);
    FuncTy = FT;
  }
  std::string getName() const override { return Name; }
  unsigned getNumArgs() const override;
  FunctionType *getFunctionType(const Module &M) const override {
    return FuncTy;
  }

  bool parseFuncName(StringRef &Name) override;

  // Methods for support type inquiry through isa, cast, and dyn_cast:
  static bool classof(const AMDGPULibFuncImpl *F) { return !F->isMangled(); }

  std::string mangle() const override { return Name; }

  void setFunctionType(FunctionType *FT) { FuncTy = FT; }
};
}
#endif // _AMDGPU_LIBFUNC_H_
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `std::string`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`std::string`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `FunctionCallee`, `FunctionType`, `Function`, `Module`, `Type`, `AMDGPULibFuncBase`
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/ADT/StringRef.h"`
- `<memory>`
