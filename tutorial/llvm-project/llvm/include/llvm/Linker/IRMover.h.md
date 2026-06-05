# IRMover.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/Linker/IRMover.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Declares LLVM public interfaces, data structures, and helper APIs for `IRMover`.
- **Purpose (CN)**: 该头文件位于 `llvm/include/llvm/Linker`，主要声明与 `IRMover` 相关的 LLVM 公共接口、数据结构和辅助 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

````cpp
//===- IRMover.h ------------------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LINKER_IRMOVER_H
#define LLVM_LINKER_IRMOVER_H

#include "llvm/ADT/ArrayRef.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/DenseSet.h"
#include "llvm/ADT/FunctionExtras.h"
#include "llvm/ADT/SmallPtrSet.h"
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L9 EN**: Starts a preprocessor conditional block: `#ifndef LLVM_LINKER_IRMOVER_H`.
  **L9 CN**: 开始一个预处理条件块：`#ifndef LLVM_LINKER_IRMOVER_H`。
- **L10 EN**: Defines macro `LLVM_LINKER_IRMOVER_H` for conditional compilation, local shorthand, or diagnostics.
  **L10 CN**: 定义宏 `LLVM_LINKER_IRMOVER_H`，供条件编译、本地简写或诊断使用。
- **L11 EN**: Blank line separating nearby declarations or logic blocks.
  **L11 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L12 EN**: Includes "llvm/ADT/ArrayRef.h" to access LLVM ADT containers and low-level utilities.
  **L12 CN**: 引入 "llvm/ADT/ArrayRef.h" 以使用LLVM ADT 容器与底层工具。
- **L13 EN**: Includes "llvm/ADT/DenseMap.h" to access LLVM ADT containers and low-level utilities.
  **L13 CN**: 引入 "llvm/ADT/DenseMap.h" 以使用LLVM ADT 容器与底层工具。
- **L14 EN**: Includes "llvm/ADT/DenseSet.h" to access LLVM ADT containers and low-level utilities.
  **L14 CN**: 引入 "llvm/ADT/DenseSet.h" 以使用LLVM ADT 容器与底层工具。
- **L15 EN**: Includes "llvm/ADT/FunctionExtras.h" to access LLVM ADT containers and low-level utilities.
  **L15 CN**: 引入 "llvm/ADT/FunctionExtras.h" 以使用LLVM ADT 容器与底层工具。
- **L16 EN**: Includes "llvm/ADT/SmallPtrSet.h" to access LLVM ADT containers and low-level utilities.
  **L16 CN**: 引入 "llvm/ADT/SmallPtrSet.h" 以使用LLVM ADT 容器与底层工具。

### Lines 17-32

````cpp
#include "llvm/Support/Compiler.h"
#include <functional>

namespace llvm {
class Error;
class GlobalValue;
class Metadata;
class MDNode;
class NamedMDNode;
class Module;
class StructType;
class TrackingMDRef;
class Type;

class IRMover {
  struct StructTypeKeyInfo {
````
- **L17 EN**: Includes "llvm/Support/Compiler.h" to access support-library facilities such as diagnostics, casting, or allocation helpers.
  **L17 CN**: 引入 "llvm/Support/Compiler.h" 以使用Support 库设施，例如诊断、类型转换或分配辅助功能。
- **L18 EN**: Includes <functional> to access standard-library facilities used by this interface.
  **L18 CN**: 引入 <functional> 以使用该接口使用的标准库设施。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L20 EN**: Opens namespace scope `llvm`.
  **L20 CN**: 打开命名空间作用域 `llvm`。
- **L21 EN**: Declares class `Error`.
  **L21 CN**: 声明 class `Error`。
- **L22 EN**: Declares class `GlobalValue`.
  **L22 CN**: 声明 class `GlobalValue`。
- **L23 EN**: Declares class `Metadata`.
  **L23 CN**: 声明 class `Metadata`。
- **L24 EN**: Declares class `MDNode`.
  **L24 CN**: 声明 class `MDNode`。
- **L25 EN**: Declares class `NamedMDNode`.
  **L25 CN**: 声明 class `NamedMDNode`。
- **L26 EN**: Declares class `Module`.
  **L26 CN**: 声明 class `Module`。
- **L27 EN**: Declares class `StructType`.
  **L27 CN**: 声明 class `StructType`。
- **L28 EN**: Declares class `TrackingMDRef`.
  **L28 CN**: 声明 class `TrackingMDRef`。
- **L29 EN**: Declares class `Type`.
  **L29 CN**: 声明 class `Type`。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L31 EN**: Declares class `IRMover`.
  **L31 CN**: 声明 class `IRMover`。
- **L32 EN**: Declares struct `StructTypeKeyInfo`.
  **L32 CN**: 声明 struct `StructTypeKeyInfo`。

### Lines 33-48

````cpp
    struct KeyTy {
      ArrayRef<Type *> ETypes;
      bool IsPacked;
      LLVM_ABI KeyTy(ArrayRef<Type *> E, bool P);
      LLVM_ABI KeyTy(const StructType *ST);
      LLVM_ABI bool operator==(const KeyTy &that) const;
      LLVM_ABI bool operator!=(const KeyTy &that) const;
    };
    LLVM_ABI static StructType *getEmptyKey();
    LLVM_ABI static StructType *getTombstoneKey();
    LLVM_ABI static unsigned getHashValue(const KeyTy &Key);
    LLVM_ABI static unsigned getHashValue(const StructType *ST);
    LLVM_ABI static bool isEqual(const KeyTy &LHS, const StructType *RHS);
    LLVM_ABI static bool isEqual(const StructType *LHS, const StructType *RHS);
  };

````
- **L33 EN**: Declares struct `KeyTy`.
  **L33 CN**: 声明 struct `KeyTy`。
- **L34 EN**: Executes a standalone statement or declaration: `ArrayRef<Type *> ETypes;`.
  **L34 CN**: 执行一条独立语句或声明：`ArrayRef<Type *> ETypes;`。
- **L35 EN**: Executes a standalone statement or declaration: `bool IsPacked;`.
  **L35 CN**: 执行一条独立语句或声明：`bool IsPacked;`。
- **L36 EN**: Executes a call or declaration centered on `KeyTy`.
  **L36 CN**: 执行以 `KeyTy` 为核心的调用或声明。
- **L37 EN**: Executes a call or declaration centered on `KeyTy`.
  **L37 CN**: 执行以 `KeyTy` 为核心的调用或声明。
- **L38 EN**: Initializes variable `operator` from the right-hand expression.
  **L38 CN**: 使用右侧表达式初始化变量 `operator`。
- **L39 EN**: Executes a call or declaration centered on `operator!=`.
  **L39 CN**: 执行以 `operator!=` 为核心的调用或声明。
- **L40 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L40 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L41 EN**: Executes a call or declaration centered on `*getEmptyKey`.
  **L41 CN**: 执行以 `*getEmptyKey` 为核心的调用或声明。
- **L42 EN**: Executes a call or declaration centered on `*getTombstoneKey`.
  **L42 CN**: 执行以 `*getTombstoneKey` 为核心的调用或声明。
- **L43 EN**: Executes a call or declaration centered on `getHashValue`.
  **L43 CN**: 执行以 `getHashValue` 为核心的调用或声明。
- **L44 EN**: Executes a call or declaration centered on `getHashValue`.
  **L44 CN**: 执行以 `getHashValue` 为核心的调用或声明。
- **L45 EN**: Executes a call or declaration centered on `isEqual`.
  **L45 CN**: 执行以 `isEqual` 为核心的调用或声明。
- **L46 EN**: Executes a call or declaration centered on `isEqual`.
  **L46 CN**: 执行以 `isEqual` 为核心的调用或声明。
- **L47 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L47 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 49-64

````cpp
  /// Type of the Metadata map in \a ValueToValueMapTy.
  typedef DenseMap<const Metadata *, TrackingMDRef> MDMapT;

public:
  class IdentifiedStructTypeSet {
    // The set of opaque types is the composite module.
    DenseSet<StructType *> OpaqueStructTypes;

    // The set of identified but non opaque structures in the composite module.
    DenseSet<StructType *, StructTypeKeyInfo> NonOpaqueStructTypes;

  public:
    LLVM_ABI void addNonOpaque(StructType *Ty);
    LLVM_ABI void switchToNonOpaque(StructType *Ty);
    LLVM_ABI void addOpaque(StructType *Ty);
    LLVM_ABI StructType *findNonOpaque(ArrayRef<Type *> ETypes, bool IsPacked);
````
- **L49 EN**: Comment explains nearby logic, invariants, or intent: `Type of the Metadata map in \a ValueToValueMapTy.`.
  **L49 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Type of the Metadata map in \a ValueToValueMapTy.`。
- **L50 EN**: Adds an auxiliary declaration: `typedef DenseMap<const Metadata *, TrackingMDRef> MDMapT;`.
  **L50 CN**: 添加一条辅助声明：`typedef DenseMap<const Metadata *, TrackingMDRef> MDMapT;`。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L52 EN**: Sets the following members to `public` access.
  **L52 CN**: 将后续成员的访问级别设为 `public`。
- **L53 EN**: Declares class `IdentifiedStructTypeSet`.
  **L53 CN**: 声明 class `IdentifiedStructTypeSet`。
- **L54 EN**: Comment explains nearby logic, invariants, or intent: `The set of opaque types is the composite module.`.
  **L54 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The set of opaque types is the composite module.`。
- **L55 EN**: Executes a standalone statement or declaration: `DenseSet<StructType *> OpaqueStructTypes;`.
  **L55 CN**: 执行一条独立语句或声明：`DenseSet<StructType *> OpaqueStructTypes;`。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L57 EN**: Comment explains nearby logic, invariants, or intent: `The set of identified but non opaque structures in the composite module.`.
  **L57 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The set of identified but non opaque structures in the composite module.`。
- **L58 EN**: Executes a standalone statement or declaration: `DenseSet<StructType *, StructTypeKeyInfo> NonOpaqueStructTypes;`.
  **L58 CN**: 执行一条独立语句或声明：`DenseSet<StructType *, StructTypeKeyInfo> NonOpaqueStructTypes;`。
- **L59 EN**: Blank line separating nearby declarations or logic blocks.
  **L59 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L60 EN**: Sets the following members to `public` access.
  **L60 CN**: 将后续成员的访问级别设为 `public`。
- **L61 EN**: Executes a call or declaration centered on `addNonOpaque`.
  **L61 CN**: 执行以 `addNonOpaque` 为核心的调用或声明。
- **L62 EN**: Executes a call or declaration centered on `switchToNonOpaque`.
  **L62 CN**: 执行以 `switchToNonOpaque` 为核心的调用或声明。
- **L63 EN**: Executes a call or declaration centered on `addOpaque`.
  **L63 CN**: 执行以 `addOpaque` 为核心的调用或声明。
- **L64 EN**: Executes a call or declaration centered on `*findNonOpaque`.
  **L64 CN**: 执行以 `*findNonOpaque` 为核心的调用或声明。

### Lines 65-80

````cpp
    LLVM_ABI bool hasType(StructType *Ty);
  };

  LLVM_ABI IRMover(Module &M);

  typedef std::function<void(GlobalValue &)> ValueAdder;
  using LazyCallback =
      llvm::unique_function<void(GlobalValue &GV, ValueAdder Add)>;

  using NamedMDNodesT =
      DenseMap<const NamedMDNode *, SmallPtrSet<const MDNode *, 8>>;

  /// Move in the provide values in \p ValuesToLink from \p Src.
  ///
  /// - \p AddLazyFor is a call back that the IRMover will call when a global
  ///   value is referenced by one of the ValuesToLink (transitively) but was
````
- **L65 EN**: Executes a call or declaration centered on `hasType`.
  **L65 CN**: 执行以 `hasType` 为核心的调用或声明。
- **L66 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L66 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L67 EN**: Blank line separating nearby declarations or logic blocks.
  **L67 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L68 EN**: Executes a call or declaration centered on `IRMover`.
  **L68 CN**: 执行以 `IRMover` 为核心的调用或声明。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L70 EN**: Adds an auxiliary declaration: `typedef std::function<void(GlobalValue &)> ValueAdder;`.
  **L70 CN**: 添加一条辅助声明：`typedef std::function<void(GlobalValue &)> ValueAdder;`。
- **L71 EN**: Defines alias `LazyCallback` to simplify later code.
  **L71 CN**: 定义别名 `LazyCallback` 以简化后续代码。
- **L72 EN**: Executes a call or declaration centered on `llvm::unique_function<void`.
  **L72 CN**: 执行以 `llvm::unique_function<void` 为核心的调用或声明。
- **L73 EN**: Blank line separating nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L74 EN**: Defines alias `NamedMDNodesT` to simplify later code.
  **L74 CN**: 定义别名 `NamedMDNodesT` 以简化后续代码。
- **L75 EN**: Executes a standalone statement or declaration: `DenseMap<const NamedMDNode *, SmallPtrSet<const MDNode *, 8>>;`.
  **L75 CN**: 执行一条独立语句或声明：`DenseMap<const NamedMDNode *, SmallPtrSet<const MDNode *, 8>>;`。
- **L76 EN**: Blank line separating nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L77 EN**: Comment explains nearby logic, invariants, or intent: `Move in the provide values in \p ValuesToLink from \p Src.`.
  **L77 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Move in the provide values in \p ValuesToLink from \p Src.`。
- **L78 EN**: Separator comment used for visual grouping.
  **L78 CN**: 用于视觉分组的分隔注释。
- **L79 EN**: Comment explains nearby logic, invariants, or intent: `- \p AddLazyFor is a call back that the IRMover will call when a global`.
  **L79 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`- \p AddLazyFor is a call back that the IRMover will call when a global`。
- **L80 EN**: Comment explains nearby logic, invariants, or intent: `value is referenced by one of the ValuesToLink (transitively) but was`.
  **L80 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`value is referenced by one of the ValuesToLink (transitively) but was`。

### Lines 81-96

````cpp
  ///   not present in ValuesToLink. The GlobalValue and a ValueAdder callback
  ///   are passed as an argument, and the callback is expected to be called
  ///   if the GlobalValue needs to be added to the \p ValuesToLink and linked.
  ///   Pass nullptr if there's no work to be done in such cases.
  /// - \p IsPerformingImport is true when this IR link is to perform ThinLTO
  ///   function importing from Src.
  LLVM_ABI Error move(std::unique_ptr<Module> Src,
                      ArrayRef<GlobalValue *> ValuesToLink,
                      LazyCallback AddLazyFor, bool IsPerformingImport);
  Module &getModule() { return Composite; }

private:
  Module &Composite;
  IdentifiedStructTypeSet IdentifiedStructTypes;
  MDMapT SharedMDs; ///< A Metadata map to use for all calls to \a move().
  NamedMDNodesT NamedMDNodes; ///< Cache for IRMover::linkNamedMDNodes().
````
- **L81 EN**: Comment explains nearby logic, invariants, or intent: `not present in ValuesToLink. The GlobalValue and a ValueAdder callback`.
  **L81 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`not present in ValuesToLink. The GlobalValue and a ValueAdder callback`。
- **L82 EN**: Comment explains nearby logic, invariants, or intent: `are passed as an argument, and the callback is expected to be called`.
  **L82 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`are passed as an argument, and the callback is expected to be called`。
- **L83 EN**: Comment explains nearby logic, invariants, or intent: `if the GlobalValue needs to be added to the \p ValuesToLink and linked.`.
  **L83 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`if the GlobalValue needs to be added to the \p ValuesToLink and linked.`。
- **L84 EN**: Comment explains nearby logic, invariants, or intent: `Pass nullptr if there's no work to be done in such cases.`.
  **L84 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Pass nullptr if there's no work to be done in such cases.`。
- **L85 EN**: Comment explains nearby logic, invariants, or intent: `- \p IsPerformingImport is true when this IR link is to perform ThinLTO`.
  **L85 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`- \p IsPerformingImport is true when this IR link is to perform ThinLTO`。
- **L86 EN**: Comment explains nearby logic, invariants, or intent: `function importing from Src.`.
  **L86 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`function importing from Src.`。
- **L87 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVM_ABI Error move(std::unique_ptr<Module> Src,`.
  **L87 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVM_ABI Error move(std::unique_ptr<Module> Src,`。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArrayRef<GlobalValue *> ValuesToLink,`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArrayRef<GlobalValue *> ValuesToLink,`。
- **L89 EN**: Executes a standalone statement or declaration: `LazyCallback AddLazyFor, bool IsPerformingImport);`.
  **L89 CN**: 执行一条独立语句或声明：`LazyCallback AddLazyFor, bool IsPerformingImport);`。
- **L90 EN**: Continues logic associated with callable symbol `getModule`.
  **L90 CN**: 继续与可调用符号 `getModule` 相关的逻辑。
- **L91 EN**: Blank line separating nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L92 EN**: Sets the following members to `private` access.
  **L92 CN**: 将后续成员的访问级别设为 `private`。
- **L93 EN**: Executes a standalone statement or declaration: `Module &Composite;`.
  **L93 CN**: 执行一条独立语句或声明：`Module &Composite;`。
- **L94 EN**: Executes a standalone statement or declaration: `IdentifiedStructTypeSet IdentifiedStructTypes;`.
  **L94 CN**: 执行一条独立语句或声明：`IdentifiedStructTypeSet IdentifiedStructTypes;`。
- **L95 EN**: Continues logic associated with callable symbol `move`.
  **L95 CN**: 继续与可调用符号 `move` 相关的逻辑。
- **L96 EN**: Continues logic associated with callable symbol `linkNamedMDNodes`.
  **L96 CN**: 继续与可调用符号 `linkNamedMDNodes` 相关的逻辑。

### Lines 97-101

````cpp
};

} // End llvm namespace

#endif
````
- **L97 EN**: Closes the current declaration scope such as a class, struct, or enum.
  **L97 CN**: 结束当前声明作用域，例如类、结构体或枚举。
- **L98 EN**: Blank line separating nearby declarations or logic blocks.
  **L98 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L99 EN**: Continues the surrounding expression or declaration: `} // End llvm namespace`.
  **L99 CN**: 继续构造周围的表达式或声明：`} // End llvm namespace`。
- **L100 EN**: Blank line separating nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L101 EN**: Closes the current preprocessor conditional block.
  **L101 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **LLVM public API surface / LLVM 公共 API 表面**
- **Function-level IR management / 函数级 IR 管理**
- **Metadata representation / 元数据表示**
- **Dense hash-based mapping / DenseMap 哈希映射**
- **Typed error propagation / 类型化错误传播**
- **Value-or-error transport / 值或错误的传递**

## Dependencies / 依赖关系

- `llvm/ADT/ArrayRef.h`: Provides LLVM ADT containers and low-level utilities. / 提供LLVM ADT 容器与底层工具。
- `llvm/ADT/DenseMap.h`: Provides LLVM ADT containers and low-level utilities. / 提供LLVM ADT 容器与底层工具。
- `llvm/ADT/DenseSet.h`: Provides LLVM ADT containers and low-level utilities. / 提供LLVM ADT 容器与底层工具。
- `llvm/ADT/FunctionExtras.h`: Provides LLVM ADT containers and low-level utilities. / 提供LLVM ADT 容器与底层工具。
- `llvm/ADT/SmallPtrSet.h`: Provides LLVM ADT containers and low-level utilities. / 提供LLVM ADT 容器与底层工具。
- `llvm/Support/Compiler.h`: Provides support-library facilities such as diagnostics, casting, or allocation helpers. / 提供Support 库设施，例如诊断、类型转换或分配辅助功能。
- `functional`: Provides supporting declarations used by the current file. / 提供当前文件使用的辅助声明。
