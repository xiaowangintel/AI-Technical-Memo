# VirtualInstruction.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/include/polly/Support/VirtualInstruction.h` | `polly/include/polly/Support/VirtualInstruction.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Declares public Polly interfaces, passes, analyses, and data structures. | 声明 Polly 的公共接口、Pass、分析能力与数据结构。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

````cpp
//===------ VirtualInstruction.cpp ------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Tools for determining which instructions are within a statement and the
// nature of their operands.
//
//===----------------------------------------------------------------------===//

````
- **EN**: This block records the standard LLVM file banner and license metadata; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 保留用于解释意图、用法或算法背景的注释.

### Lines 14-29

````cpp
#ifndef POLLY_SUPPORT_VIRTUALINSTRUCTION_H
#define POLLY_SUPPORT_VIRTUALINSTRUCTION_H

#include "polly/ScopInfo.h"

namespace polly {
using llvm::User;

/// Determine the nature of a value's use within a statement.
///
/// These are not always representable by llvm::Use. For instance, scalar write
/// MemoryAccesses do use a value, but are not associated with an instruction's
/// argument.
///
/// Despite its name it is not tied to virtual instructions (although it works
/// fine with them), but to promote consistent handling of values used in
````
- **EN**: This block imports Polly headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; opens or organizes declarations inside a C++ namespace; defines macros like `POLLY_SUPPORT_VIRTUALINSTRUCTION_H`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 Polly 头文件; 使用预处理指令控制编译、宏或头文件保护; 在 C++ 命名空间中组织声明或实现; 定义宏，例如 `POLLY_SUPPORT_VIRTUALINSTRUCTION_H`；并延续周边实现细节。

### Lines 30-42

````cpp
/// statements.
class VirtualUse final {
public:
  /// The different types of uses. Handling usually differentiates a lot between
  /// these; one can use a switch to handle each case (and get warned by the
  /// compiler if one is not handled).
  enum UseKind {
    // An llvm::Constant.
    Constant,

    // An llvm::BasicBlock.
    Block,

````
- **EN**: This block declares or references types such as `VirtualUse`; defines enum values such as `UseKind`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或引用类型，例如 `VirtualUse`; 定义枚举类型，例如 `UseKind`; 保留用于解释意图、用法或算法背景的注释.

### Lines 43-56

````cpp
    // A value that can be generated using ScopExpander.
    Synthesizable,

    // A load that always reads the same value throughout the SCoP (address and
    // the value located there a SCoP-invariant) and has been hoisted in front
    // of the SCoP.
    Hoisted,

    // Definition before the SCoP and not synthesizable. Can be an instruction
    // outside the SCoP, a function argument or a global value. Whether there is
    // a scalar MemoryAccess in this statement for reading it depends on the
    // -polly-analyze-read-only-scalars switch.
    ReadOnly,

````
- **EN**: This block touches Polly SCoP abstractions that model analyzable regions; works with memory-access metadata or access relations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 处理内存访问元数据或访问关系; 保留用于解释意图、用法或算法背景的注释.

### Lines 57-69

````cpp
    // A definition within the same statement. No MemoryAccess between
    // definition and use are necessary.
    Intra,

    // Definition in another statement. There is a scalar MemoryAccess that
    // makes it available in this statement.
    Inter
  };

private:
  /// The statement where a value is used.
  ScopStmt *User;

````
- **EN**: This block touches Polly SCoP abstractions that model analyzable regions; works with memory-access metadata or access relations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 处理内存访问元数据或访问关系; 保留用于解释意图、用法或算法背景的注释.

### Lines 70-84

````cpp
  /// The value that is used.
  Value *Val;

  /// The type of value use.
  UseKind Kind;

  /// The value represented as llvm::SCEV expression.
  const SCEV *ScevExpr;

  /// If this is an inter-statement (or read-only) use, contains the
  /// MemoryAccess that makes the value available in this statement. In case of
  /// intra-statement uses, can contain a MemoryKind::Array access. In all other
  /// cases, it is a nullptr.
  MemoryAccess *InputMA;

````
- **EN**: This block works with memory-access metadata or access relations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 处理内存访问元数据或访问关系; 保留用于解释意图、用法或算法背景的注释.

### Lines 85-101

````cpp
  VirtualUse(ScopStmt *User, Value *Val, UseKind Kind, const SCEV *ScevExpr,
             MemoryAccess *InputMA)
      : User(User), Val(Val), Kind(Kind), ScevExpr(ScevExpr), InputMA(InputMA) {
  }

public:
  /// Get a VirtualUse for an llvm::Use.
  ///
  /// @param S       The Scop object.
  /// @param U       The llvm::Use the get information for.
  /// @param LI      The LoopInfo analysis. Needed to determine whether the
  ///                value is synthesizable.
  /// @param Virtual Whether to ignore existing MemoryAccess.
  ///
  /// @return The VirtualUse representing the same use as @p U.
  static VirtualUse create(Scop *S, const Use &U, LoopInfo *LI, bool Virtual);

````
- **EN**: This block declares or defines routines around `VirtualUse`, `User`, `create`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; works with memory-access metadata or access relations; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `VirtualUse`, `User`, `create` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 处理内存访问元数据或访问关系；并延续周边实现细节。

### Lines 102-117

````cpp
  /// Get a VirtualUse for uses within statements.
  ///
  /// It is assumed that the user is not a PHINode. Such uses are always
  /// VirtualUse::Inter unless in a regions statement.
  ///
  /// @param S         The Scop object.
  /// @param UserStmt  The statement in which @p Val is used. Can be nullptr, in
  ///                  which case it assumed that the statement has been
  ///                  removed, which is only possible if no instruction in it
  ///                  had side-effects or computes a value used by another
  ///                  statement.
  /// @param UserScope Loop scope in which the value is used. Needed to
  ///                  determine whether the value is synthesizable.
  /// @param Val       The value being used.
  /// @param Virtual   Whether to use (and prioritize over instruction location)
  ///                  information about MemoryAccesses.
````
- **EN**: This block touches Polly SCoP abstractions that model analyzable regions; works with memory-access metadata or access relations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 处理内存访问元数据或访问关系; 保留用于解释意图、用法或算法背景的注释.

### Lines 118-136

````cpp
  ///
  /// @return A VirtualUse object that gives information about @p Val's use in
  ///         @p UserStmt.
  static VirtualUse create(Scop *S, ScopStmt *UserStmt, Loop *UserScope,
                           Value *Val, bool Virtual);

  static VirtualUse create(ScopStmt *UserStmt, Loop *UserScope, Value *Val,
                           bool Virtual) {
    return create(UserStmt->getParent(), UserStmt, UserScope, Val, Virtual);
  }

  bool isConstant() const { return Kind == Constant; }
  bool isBlock() const { return Kind == Block; }
  bool isSynthesizable() const { return Kind == Synthesizable; }
  bool isHoisted() const { return Kind == Hoisted; }
  bool isReadOnly() const { return Kind == ReadOnly; }
  bool isIntra() const { return Kind == Intra; }
  bool isInter() const { return Kind == Inter; }

````
- **EN**: This block declares or defines routines around `create`, `isConstant`, `isBlock`, `isSynthesizable` (+4 more); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `create`, `isConstant`, `isBlock`, `isSynthesizable` (+4 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 137-148

````cpp
  /// Return user statement.
  ScopStmt *getUser() const { return User; }

  /// Return the used value.
  llvm::Value *getValue() const { return Val; }

  /// Return the type of use.
  UseKind getKind() const { return Kind; }

  /// Return the ScalarEvolution representation of @p Val.
  const SCEV *getScevExpr() const { return ScevExpr; }

````
- **EN**: This block declares or defines routines around `getUser`, `getValue`, `getKind`, `getScevExpr`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getUser`, `getValue`, `getKind`, `getScevExpr` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 149-163

````cpp
  /// Return the MemoryAccess that makes the value available in this statement,
  /// if any.
  MemoryAccess *getMemoryAccess() const { return InputMA; }

  /// Print a description of this object.
  ///
  /// @param OS           Stream to print to.
  /// @param Reproducible If true, ensures that the output is stable between
  ///                     runs and is suitable to check in regression tests.
  ///                     This excludes printing e.g. pointer values. If false,
  ///                     the output should not be used for regression tests,
  ///                     but may contain more information useful in debugger
  ///                     sessions.
  void print(raw_ostream &OS, bool Reproducible = true) const;

````
- **EN**: This block declares or defines routines around `getMemoryAccess`, `print`; emits return paths that hand results or status codes back to callers; works with memory-access metadata or access relations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getMemoryAccess`, `print` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 处理内存访问元数据或访问关系; 保留用于解释意图、用法或算法背景的注释.

### Lines 164-175

````cpp
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  void dump() const;
#endif
};

/// An iterator for virtual operands.
class VirtualOperandIterator final {
  friend class VirtualInstruction;
  friend class VirtualUse;

  using Self = VirtualOperandIterator;

````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or references types such as `VirtualOperandIterator`; declares or defines routines around `dump`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或引用类型，例如 `VirtualOperandIterator`; 声明或定义与 `dump` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 176-188

````cpp
  ScopStmt *User;
  User::op_iterator U;

  VirtualOperandIterator(ScopStmt *User, User::op_iterator U)
      : User(User), U(U) {}

public:
  using iterator_category = std::forward_iterator_tag;
  using value_type = VirtualUse;
  using difference_type = std::ptrdiff_t;
  using pointer = value_type *;
  using reference = value_type &;

````
- **EN**: This block declares or defines routines around `VirtualOperandIterator`, `User`; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `VirtualOperandIterator`, `User` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 189-202

````cpp
  inline bool operator==(const Self &that) const {
    assert(this->User == that.User);
    return this->U == that.U;
  }

  inline bool operator!=(const Self &that) const {
    assert(this->User == that.User);
    return this->U != that.U;
  }

  VirtualUse operator*() const {
    return VirtualUse::create(User, User->getSurroundingLoop(), U->get(), true);
  }

````
- **EN**: This block emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states.
- **CN**: 该代码块 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查.

### Lines 203-216

````cpp
  Use *operator->() const { return U; }

  Self &operator++() {
    U++;
    return *this;
  }

  Self operator++(int) {
    Self tmp = *this;
    ++*this;
    return tmp;
  }
};

````
- **EN**: This block emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 包含返回路径，用于向调用者交回结果或状态码.

### Lines 217-234

````cpp
/// This class represents a "virtual instruction", an instruction in a ScopStmt,
/// effectively a ScopStmt/Instruction-pair.
///
/// An instructions can be moved between statements (e.g. to avoid a scalar
/// dependency) and even can be contained in multiple statements (for instance,
/// to recompute a value instead of transferring it), hence 'virtual'. This
/// class is required to represent such instructions that are not in their
/// 'physical' location anymore.
///
/// A statement can currently not contain the same instructions multiple times
/// (that is, from different loop iterations). Therefore, a
/// ScopStmt/Instruction-pair uniquely identifies a virtual instructions.
/// ScopStmt::getInstruction() can contain the same instruction multiple times,
/// but they necessarily compute the same value.
class VirtualInstruction final {
  friend class VirtualOperandIterator;
  friend struct llvm::DenseMapInfo<VirtualInstruction>;

````
- **EN**: This block declares or references types such as `VirtualInstruction`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或引用类型，例如 `VirtualInstruction`; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 235-250

````cpp
private:
  /// The statement this virtual instruction is in.
  ScopStmt *Stmt = nullptr;

  /// The instruction of a statement.
  Instruction *Inst = nullptr;

public:
  VirtualInstruction() {}

  /// Create a new virtual instruction of an instruction @p Inst in @p Stmt.
  VirtualInstruction(ScopStmt *Stmt, Instruction *Inst)
      : Stmt(Stmt), Inst(Inst) {
    assert(Stmt && Inst);
  }

````
- **EN**: This block declares or defines routines around `VirtualInstruction`, `Stmt`; adds defensive checks for invariants or impossible states; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `VirtualInstruction`, `Stmt` 相关的例程; 加入用于保护不变量或异常状态的断言/检查; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 251-266

````cpp
  VirtualOperandIterator operand_begin() const {
    return VirtualOperandIterator(Stmt, Inst->op_begin());
  }

  VirtualOperandIterator operand_end() const {
    return VirtualOperandIterator(Stmt, Inst->op_end());
  }

  /// Returns a list of virtual operands.
  ///
  /// Virtual operands, like virtual instructions, need to encode the ScopStmt
  /// they are in.
  llvm::iterator_range<VirtualOperandIterator> operands() const {
    return {operand_begin(), operand_end()};
  }

````
- **EN**: This block declares or defines routines around `operand_begin`, `operand_end`, `operands`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `operand_begin`, `operand_end`, `operands` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 267-286

````cpp
  /// Return the SCoP everything is contained in.
  Scop *getScop() const { return Stmt->getParent(); }

  /// Return the ScopStmt this virtual instruction is in.
  ScopStmt *getStmt() const { return Stmt; }

  /// Return the instruction in the statement.
  Instruction *getInstruction() const { return Inst; }

  /// Print a description of this object.
  ///
  /// @param OS           Stream to print to.
  /// @param Reproducible If true, ensures that the output is stable between
  ///                     runs and is suitable for checks in regression tests.
  ///                     This excludes printing e.g., pointer values. If false,
  ///                     the output should not be used for regression tests,
  ///                     but may contain more information useful in debugger
  ///                     sessions.
  void print(raw_ostream &OS, bool Reproducible = true) const;

````
- **EN**: This block declares or defines routines around `getScop`, `getStmt`, `getInstruction`, `print`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getScop`, `getStmt`, `getInstruction`, `print` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 287-302

````cpp
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  void dump() const;
#endif
};

static inline bool operator==(VirtualInstruction LHS, VirtualInstruction RHS) {
  return LHS.getStmt() == RHS.getStmt() &&
         LHS.getInstruction() == RHS.getInstruction();
}

/// Find all reachable instructions and accesses.
///
/// @param S              The SCoP to find everything reachable in.
/// @param LI             LoopInfo required for analysis.
/// @param UsedInsts[out] Receives all reachable instructions.
/// @param UsedAccs[out]  Receives all reachable accesses.
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or defines routines around `dump`, `getInstruction`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或定义与 `dump`, `getInstruction` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 303-318

````cpp
/// @param OnlyLocal      If non-nullptr, activates local mode: The SCoP is
///                       assumed to consist only of this statement and is
///                       conservatively correct. Does not require walking the
///                       whole SCoP.
void markReachable(Scop *S, LoopInfo *LI,
                   DenseSet<VirtualInstruction> &UsedInsts,
                   DenseSet<MemoryAccess *> &UsedAccs,
                   ScopStmt *OnlyLocal = nullptr);
} // namespace polly

namespace llvm {
/// Support VirtualInstructions in llvm::DenseMaps.
template <> struct DenseMapInfo<polly::VirtualInstruction> {
public:
  static bool isEqual(polly::VirtualInstruction LHS,
                      polly::VirtualInstruction RHS) {
````
- **EN**: This block opens or organizes declarations inside a C++ namespace; declares or defines routines around `markReachable`, `isEqual`; touches Polly SCoP abstractions that model analyzable regions; works with memory-access metadata or access relations; and continues the surrounding implementation details.
- **CN**: 该代码块 在 C++ 命名空间中组织声明或实现; 声明或定义与 `markReachable`, `isEqual` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 处理内存访问元数据或访问关系；并延续周边实现细节。

### Lines 319-331

````cpp
    return DenseMapInfo<polly::ScopStmt *>::isEqual(LHS.getStmt(),
                                                    RHS.getStmt()) &&
           DenseMapInfo<Instruction *>::isEqual(LHS.getInstruction(),
                                                RHS.getInstruction());
  }

  static polly::VirtualInstruction getTombstoneKey() {
    polly::VirtualInstruction TombstoneKey;
    TombstoneKey.Stmt = DenseMapInfo<polly::ScopStmt *>::getTombstoneKey();
    TombstoneKey.Inst = DenseMapInfo<Instruction *>::getTombstoneKey();
    return TombstoneKey;
  }

````
- **EN**: This block declares or defines routines around `getStmt`, `isEqual`, `getInstruction`, `getTombstoneKey`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 声明或定义与 `getStmt`, `isEqual`, `getInstruction`, `getTombstoneKey` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

### Lines 332-346

````cpp
  static polly::VirtualInstruction getEmptyKey() {
    polly::VirtualInstruction EmptyKey;
    EmptyKey.Stmt = DenseMapInfo<polly::ScopStmt *>::getEmptyKey();
    EmptyKey.Inst = DenseMapInfo<Instruction *>::getEmptyKey();
    return EmptyKey;
  }

  static unsigned getHashValue(polly::VirtualInstruction Val) {
    return DenseMapInfo<std::pair<polly::ScopStmt *, Instruction *>>::
        getHashValue(std::make_pair(Val.getStmt(), Val.getInstruction()));
  }
};
} // namespace llvm

#endif /* POLLY_SUPPORT_VIRTUALINSTRUCTION_H */
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or defines routines around `getEmptyKey`, `getHashValue`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或定义与 `getEmptyKey`, `getHashValue` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

## Key Concepts / 关键概念

- **SCoP modeling**
  - **CN**: SCoP 建模
- **Memory access tracking**
  - **CN**: 内存访问跟踪
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **Loop transformation**
  - **CN**: 循环变换
- **Debug and diagnostics**
  - **CN**: 调试与诊断
- **Public interface declarations**
  - **CN**: 公共接口声明

## Dependencies / 依赖关系

- **Polly headers**: `polly/ScopInfo.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/ScopInfo.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
