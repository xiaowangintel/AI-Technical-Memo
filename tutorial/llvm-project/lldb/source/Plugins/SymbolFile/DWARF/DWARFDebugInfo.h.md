# DWARFDebugInfo.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/SymbolFile/DWARF/DWARFDebugInfo.h`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: All parsing needs to be done partially any managed by this class as accessors are called.
- **Purpose (CN)**: 该文件在 LLDB 的 `SymbolFile` 子系统中声明与 `DWARFDebugInfo` 相关的接口，重点覆盖符号文件解析、调试信息加载、编译单元发现与类型提取。对应英文说明：All parsing needs to be done partially any managed by this class as accessors are called。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行

````cpp
//===-- DWARFDebugInfo.h ----------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLDB_SOURCE_PLUGINS_SYMBOLFILE_DWARF_DWARFDEBUGINFO_H
#define LLDB_SOURCE_PLUGINS_SYMBOLFILE_DWARF_DWARFDEBUGINFO_H

#include <map>
#include <vector>

#include "DWARFDIE.h"
#include "DWARFTypeUnit.h"
#include "DWARFUnit.h"
#include "SymbolFileDWARF.h"
````
- **L1 EN**: Banner comment marks a file or section boundary.
  **L1 CN**: 横幅注释用于标记文件或章节边界。
- **L2 EN**: Separator comment visually groups nearby code.
  **L2 CN**: 分隔注释用于在视觉上分组附近代码。
- **L3 EN**: Comment explains surrounding design intent or invariants: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明周边设计意图或不变式：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains surrounding design intent or invariants: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明周边设计意图或不变式：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains surrounding design intent or invariants: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明周边设计意图或不变式：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment visually groups nearby code.
  **L6 CN**: 分隔注释用于在视觉上分组附近代码。
- **L7 EN**: Banner comment marks a file or section boundary.
  **L7 CN**: 横幅注释用于标记文件或章节边界。
- **L8 EN**: Blank line separates nearby declarations or logic blocks.
  **L8 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L9 EN**: Starts header-guard macro `LLDB_SOURCE_PLUGINS_SYMBOLFILE_DWARF_DWARFDEBUGINFO_H`.
  **L9 CN**: 开始头文件保护宏 `LLDB_SOURCE_PLUGINS_SYMBOLFILE_DWARF_DWARFDEBUGINFO_H`。
- **L10 EN**: Defines macro `LLDB_SOURCE_PLUGINS_SYMBOLFILE_DWARF_DWARFDEBUGINFO_H` for include-guarding, feature control, or helper reuse.
  **L10 CN**: 定义宏 `LLDB_SOURCE_PLUGINS_SYMBOLFILE_DWARF_DWARFDEBUGINFO_H`，用于头文件保护、特性控制或辅助复用。
- **L11 EN**: Blank line separates nearby declarations or logic blocks.
  **L11 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L12 EN**: Includes `map` so this header can use standard-library or system facilities.
  **L12 CN**: 引入 `map`，使该头文件能够使用标准库或系统设施。
- **L13 EN**: Includes `vector` so this header can use standard-library or system facilities.
  **L13 CN**: 引入 `vector`，使该头文件能够使用标准库或系统设施。
- **L14 EN**: Blank line separates nearby declarations or logic blocks.
  **L14 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L15 EN**: Includes `DWARFDIE.h` so this header can use supporting declarations from another header.
  **L15 CN**: 引入 `DWARFDIE.h`，使该头文件能够使用来自其他头文件的辅助声明。
- **L16 EN**: Includes `DWARFTypeUnit.h` so this header can use supporting declarations from another header.
  **L16 CN**: 引入 `DWARFTypeUnit.h`，使该头文件能够使用来自其他头文件的辅助声明。
- **L17 EN**: Includes `DWARFUnit.h` so this header can use supporting declarations from another header.
  **L17 CN**: 引入 `DWARFUnit.h`，使该头文件能够使用来自其他头文件的辅助声明。
- **L18 EN**: Includes `SymbolFileDWARF.h` so this header can use supporting declarations from another header.
  **L18 CN**: 引入 `SymbolFileDWARF.h`，使该头文件能够使用来自其他头文件的辅助声明。

### Lines 19-36 / 第 19-36 行

````cpp
#include "lldb/lldb-private.h"
#include "llvm/Support/Error.h"

namespace lldb_private::plugin {
namespace dwarf {
class DWARFContext;

class DWARFDebugInfo {
public:
  typedef dw_offset_t (*Callback)(SymbolFileDWARF *dwarf2Data, DWARFUnit *cu,
                                  DWARFDebugInfoEntry *die,
                                  const dw_offset_t next_offset,
                                  const uint32_t depth, void *userData);

  explicit DWARFDebugInfo(SymbolFileDWARF &dwarf, DWARFContext &context);

  size_t GetNumUnits();
  DWARFUnit *GetUnitAtIndex(size_t idx);
````
- **L19 EN**: Includes `lldb/lldb-private.h` so this header can use umbrella LLDB declarations, enums, and type aliases.
  **L19 CN**: 引入 `lldb/lldb-private.h`，使该头文件能够使用LLDB 总括声明、枚举与类型别名。
- **L20 EN**: Includes `llvm/Support/Error.h` so this header can use LLVM support-library services.
  **L20 CN**: 引入 `llvm/Support/Error.h`，使该头文件能够使用LLVM 支持库服务。
- **L21 EN**: Blank line separates nearby declarations or logic blocks.
  **L21 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L22 EN**: Opens namespace `lldb_private::plugin` to group related LLDB declarations.
  **L22 CN**: 打开命名空间 `lldb_private::plugin`，以组织相关的 LLDB 声明。
- **L23 EN**: Opens namespace `dwarf` to group related LLDB declarations.
  **L23 CN**: 打开命名空间 `dwarf`，以组织相关的 LLDB 声明。
- **L24 EN**: Declares class `DWARFContext`.
  **L24 CN**: 声明 class `DWARFContext`。
- **L25 EN**: Blank line separates nearby declarations or logic blocks.
  **L25 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L26 EN**: Declares class `DWARFDebugInfo`.
  **L26 CN**: 声明 class `DWARFDebugInfo`。
- **L27 EN**: Switches the following class members to `public` access.
  **L27 CN**: 将后续类成员切换为 `public` 访问级别。
- **L28 EN**: Adds an auxiliary declaration or friend relationship: `typedef dw_offset_t (*Callback)(SymbolFileDWARF *dwarf2Data, DWARFUnit *cu,`.
  **L28 CN**: 添加辅助声明或友元关系：`typedef dw_offset_t (*Callback)(SymbolFileDWARF *dwarf2Data, DWARFUnit *cu,`。
- **L29 EN**: Continues a multi-line list, initializer, or aggregate entry: `DWARFDebugInfoEntry *die,`.
  **L29 CN**: 继续一个多行列表、初始化器或聚合项：`DWARFDebugInfoEntry *die,`。
- **L30 EN**: Continues a multi-line list, initializer, or aggregate entry: `const dw_offset_t next_offset,`.
  **L30 CN**: 继续一个多行列表、初始化器或聚合项：`const dw_offset_t next_offset,`。
- **L31 EN**: Completes a standalone declaration or statement: `const uint32_t depth, void *userData);`.
  **L31 CN**: 完成一条独立声明或语句：`const uint32_t depth, void *userData);`。
- **L32 EN**: Blank line separates nearby declarations or logic blocks.
  **L32 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L33 EN**: Declares or invokes callable logic centered on `DWARFDebugInfo`.
  **L33 CN**: 声明或调用以 `DWARFDebugInfo` 为核心的可调用逻辑。
- **L34 EN**: Blank line separates nearby declarations or logic blocks.
  **L34 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L35 EN**: Declares or invokes callable logic centered on `GetNumUnits`.
  **L35 CN**: 声明或调用以 `GetNumUnits` 为核心的可调用逻辑。
- **L36 EN**: Declares or invokes callable logic centered on `*GetUnitAtIndex`.
  **L36 CN**: 声明或调用以 `*GetUnitAtIndex` 为核心的可调用逻辑。

### Lines 37-54 / 第 37-54 行

````cpp
  DWARFUnit *GetUnitAtOffset(DIERef::Section section, dw_offset_t cu_offset,
                             uint32_t *idx_ptr = nullptr);
  DWARFUnit *GetUnitContainingDIEOffset(DIERef::Section section,
                                        dw_offset_t die_offset);
  DWARFUnit *GetSkeletonUnit(DWARFUnit *dwo_unit);
  DWARFTypeUnit *GetTypeUnitForHash(uint64_t hash);
  bool ContainsTypeUnits();
  DWARFDIE GetDIE(DIERef::Section section, dw_offset_t die_offset);

  enum {
    eDumpFlag_Verbose = (1 << 0),  // Verbose dumping
    eDumpFlag_ShowForm = (1 << 1), // Show the DW_form type
    eDumpFlag_ShowAncestors =
        (1 << 2) // Show all parent DIEs when dumping single DIEs
  };

  const DWARFDebugAranges &GetCompileUnitAranges();

````
- **L37 EN**: Continues a multi-line list, initializer, or aggregate entry: `DWARFUnit *GetUnitAtOffset(DIERef::Section section, dw_offset_t cu_offset,`.
  **L37 CN**: 继续一个多行列表、初始化器或聚合项：`DWARFUnit *GetUnitAtOffset(DIERef::Section section, dw_offset_t cu_offset,`。
- **L38 EN**: Completes a standalone declaration or statement: `uint32_t *idx_ptr = nullptr);`.
  **L38 CN**: 完成一条独立声明或语句：`uint32_t *idx_ptr = nullptr);`。
- **L39 EN**: Continues a multi-line list, initializer, or aggregate entry: `DWARFUnit *GetUnitContainingDIEOffset(DIERef::Section section,`.
  **L39 CN**: 继续一个多行列表、初始化器或聚合项：`DWARFUnit *GetUnitContainingDIEOffset(DIERef::Section section,`。
- **L40 EN**: Completes a standalone declaration or statement: `dw_offset_t die_offset);`.
  **L40 CN**: 完成一条独立声明或语句：`dw_offset_t die_offset);`。
- **L41 EN**: Declares or invokes callable logic centered on `*GetSkeletonUnit`.
  **L41 CN**: 声明或调用以 `*GetSkeletonUnit` 为核心的可调用逻辑。
- **L42 EN**: Declares or invokes callable logic centered on `*GetTypeUnitForHash`.
  **L42 CN**: 声明或调用以 `*GetTypeUnitForHash` 为核心的可调用逻辑。
- **L43 EN**: Declares or invokes callable logic centered on `ContainsTypeUnits`.
  **L43 CN**: 声明或调用以 `ContainsTypeUnits` 为核心的可调用逻辑。
- **L44 EN**: Declares or invokes callable logic centered on `GetDIE`.
  **L44 CN**: 声明或调用以 `GetDIE` 为核心的可调用逻辑。
- **L45 EN**: Blank line separates nearby declarations or logic blocks.
  **L45 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L46 EN**: Declares enum `enum`.
  **L46 CN**: 声明 enum `enum`。
- **L47 EN**: Continues the surrounding declaration or expression: `eDumpFlag_Verbose = (1 << 0),  // Verbose dumping`.
  **L47 CN**: 继续构造周围的声明或表达式：`eDumpFlag_Verbose = (1 << 0),  // Verbose dumping`。
- **L48 EN**: Continues the surrounding declaration or expression: `eDumpFlag_ShowForm = (1 << 1), // Show the DW_form type`.
  **L48 CN**: 继续构造周围的声明或表达式：`eDumpFlag_ShowForm = (1 << 1), // Show the DW_form type`。
- **L49 EN**: Continues the surrounding declaration or expression: `eDumpFlag_ShowAncestors =`.
  **L49 CN**: 继续构造周围的声明或表达式：`eDumpFlag_ShowAncestors =`。
- **L50 EN**: Continues the surrounding declaration or expression: `(1 << 2) // Show all parent DIEs when dumping single DIEs`.
  **L50 CN**: 继续构造周围的声明或表达式：`(1 << 2) // Show all parent DIEs when dumping single DIEs`。
- **L51 EN**: Closes the current declaration scope such as a class or struct.
  **L51 CN**: 结束当前声明作用域，例如类或结构体。
- **L52 EN**: Blank line separates nearby declarations or logic blocks.
  **L52 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L53 EN**: Declares or invokes callable logic centered on `&GetCompileUnitAranges`.
  **L53 CN**: 声明或调用以 `&GetCompileUnitAranges` 为核心的可调用逻辑。
- **L54 EN**: Blank line separates nearby declarations or logic blocks.
  **L54 CN**: 空行用于分隔相邻的声明或逻辑块。

### Lines 55-72 / 第 55-72 行

````cpp
  const std::shared_ptr<SymbolFileDWARFDwo> &GetDwpSymbolFile();

protected:
  typedef std::vector<DWARFUnitSP> UnitColl;

  SymbolFileDWARF &m_dwarf;
  DWARFContext &m_context;

  llvm::once_flag m_units_once_flag;
  UnitColl m_units;

  std::unique_ptr<DWARFDebugAranges>
      m_cu_aranges_up; // A quick address to compile unit table

  std::vector<std::pair<uint64_t, uint32_t>> m_type_hash_to_unit_index;
  llvm::DenseMap<uint64_t, DWARFUnit *> m_dwarf5_dwo_id_to_skeleton_unit;
  llvm::DenseMap<uint64_t, DWARFUnit *> m_dwarf4_dwo_id_to_skeleton_unit;
  llvm::once_flag m_dwarf4_dwo_id_to_skeleton_unit_once_flag;
````
- **L55 EN**: Declares or invokes callable logic centered on `&GetDwpSymbolFile`.
  **L55 CN**: 声明或调用以 `&GetDwpSymbolFile` 为核心的可调用逻辑。
- **L56 EN**: Blank line separates nearby declarations or logic blocks.
  **L56 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L57 EN**: Switches the following class members to `protected` access.
  **L57 CN**: 将后续类成员切换为 `protected` 访问级别。
- **L58 EN**: Adds an auxiliary declaration or friend relationship: `typedef std::vector<DWARFUnitSP> UnitColl;`.
  **L58 CN**: 添加辅助声明或友元关系：`typedef std::vector<DWARFUnitSP> UnitColl;`。
- **L59 EN**: Blank line separates nearby declarations or logic blocks.
  **L59 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L60 EN**: Completes a standalone declaration or statement: `SymbolFileDWARF &m_dwarf;`.
  **L60 CN**: 完成一条独立声明或语句：`SymbolFileDWARF &m_dwarf;`。
- **L61 EN**: Completes a standalone declaration or statement: `DWARFContext &m_context;`.
  **L61 CN**: 完成一条独立声明或语句：`DWARFContext &m_context;`。
- **L62 EN**: Blank line separates nearby declarations or logic blocks.
  **L62 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L63 EN**: Completes a standalone declaration or statement: `llvm::once_flag m_units_once_flag;`.
  **L63 CN**: 完成一条独立声明或语句：`llvm::once_flag m_units_once_flag;`。
- **L64 EN**: Completes a standalone declaration or statement: `UnitColl m_units;`.
  **L64 CN**: 完成一条独立声明或语句：`UnitColl m_units;`。
- **L65 EN**: Blank line separates nearby declarations or logic blocks.
  **L65 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L66 EN**: Continues the surrounding declaration or expression: `std::unique_ptr<DWARFDebugAranges>`.
  **L66 CN**: 继续构造周围的声明或表达式：`std::unique_ptr<DWARFDebugAranges>`。
- **L67 EN**: Continues the surrounding declaration or expression: `m_cu_aranges_up; // A quick address to compile unit table`.
  **L67 CN**: 继续构造周围的声明或表达式：`m_cu_aranges_up; // A quick address to compile unit table`。
- **L68 EN**: Blank line separates nearby declarations or logic blocks.
  **L68 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L69 EN**: Completes a standalone declaration or statement: `std::vector<std::pair<uint64_t, uint32_t>> m_type_hash_to_unit_index;`.
  **L69 CN**: 完成一条独立声明或语句：`std::vector<std::pair<uint64_t, uint32_t>> m_type_hash_to_unit_index;`。
- **L70 EN**: Completes a standalone declaration or statement: `llvm::DenseMap<uint64_t, DWARFUnit *> m_dwarf5_dwo_id_to_skeleton_unit;`.
  **L70 CN**: 完成一条独立声明或语句：`llvm::DenseMap<uint64_t, DWARFUnit *> m_dwarf5_dwo_id_to_skeleton_unit;`。
- **L71 EN**: Completes a standalone declaration or statement: `llvm::DenseMap<uint64_t, DWARFUnit *> m_dwarf4_dwo_id_to_skeleton_unit;`.
  **L71 CN**: 完成一条独立声明或语句：`llvm::DenseMap<uint64_t, DWARFUnit *> m_dwarf4_dwo_id_to_skeleton_unit;`。
- **L72 EN**: Completes a standalone declaration or statement: `llvm::once_flag m_dwarf4_dwo_id_to_skeleton_unit_once_flag;`.
  **L72 CN**: 完成一条独立声明或语句：`llvm::once_flag m_dwarf4_dwo_id_to_skeleton_unit_once_flag;`。

### Lines 73-89 / 第 73-89 行

````cpp

private:
  // All parsing needs to be done partially any managed by this class as
  // accessors are called.
  void ParseUnitHeadersIfNeeded();

  void ParseUnitsFor(DIERef::Section section);

  uint32_t FindUnitIndex(DIERef::Section section, dw_offset_t offset);

  DWARFDebugInfo(const DWARFDebugInfo &) = delete;
  const DWARFDebugInfo &operator=(const DWARFDebugInfo &) = delete;
};
} // namespace dwarf
} // namespace lldb_private::plugin

#endif // LLDB_SOURCE_PLUGINS_SYMBOLFILE_DWARF_DWARFDEBUGINFO_H
````
- **L73 EN**: Blank line separates nearby declarations or logic blocks.
  **L73 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L74 EN**: Switches the following class members to `private` access.
  **L74 CN**: 将后续类成员切换为 `private` 访问级别。
- **L75 EN**: Comment explains surrounding design intent or invariants: `All parsing needs to be done partially any managed by this class as`.
  **L75 CN**: 注释说明周边设计意图或不变式：`All parsing needs to be done partially any managed by this class as`。
- **L76 EN**: Comment explains surrounding design intent or invariants: `accessors are called.`.
  **L76 CN**: 注释说明周边设计意图或不变式：`accessors are called.`。
- **L77 EN**: Declares or invokes callable logic centered on `ParseUnitHeadersIfNeeded`.
  **L77 CN**: 声明或调用以 `ParseUnitHeadersIfNeeded` 为核心的可调用逻辑。
- **L78 EN**: Blank line separates nearby declarations or logic blocks.
  **L78 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L79 EN**: Declares or invokes callable logic centered on `ParseUnitsFor`.
  **L79 CN**: 声明或调用以 `ParseUnitsFor` 为核心的可调用逻辑。
- **L80 EN**: Blank line separates nearby declarations or logic blocks.
  **L80 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L81 EN**: Declares or invokes callable logic centered on `FindUnitIndex`.
  **L81 CN**: 声明或调用以 `FindUnitIndex` 为核心的可调用逻辑。
- **L82 EN**: Blank line separates nearby declarations or logic blocks.
  **L82 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L83 EN**: Declares or invokes callable logic centered on `DWARFDebugInfo`.
  **L83 CN**: 声明或调用以 `DWARFDebugInfo` 为核心的可调用逻辑。
- **L84 EN**: Declares or invokes callable logic centered on `&operator=`.
  **L84 CN**: 声明或调用以 `&operator=` 为核心的可调用逻辑。
- **L85 EN**: Closes the current declaration scope such as a class or struct.
  **L85 CN**: 结束当前声明作用域，例如类或结构体。
- **L86 EN**: Closes a namespace scope and preserves the trailing comment: `} // namespace dwarf`.
  **L86 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace dwarf`。
- **L87 EN**: Closes a namespace scope and preserves the trailing comment: `} // namespace lldb_private::plugin`.
  **L87 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace lldb_private::plugin`。
- **L88 EN**: Blank line separates nearby declarations or logic blocks.
  **L88 CN**: 空行用于分隔相邻的声明或逻辑块。
- **L89 EN**: Ends the current preprocessor-conditional region.
  **L89 CN**: 结束当前预处理条件区域。

## Key Concepts / 关键概念

- **Role / 角色**: This file is a declaration header inside LLDB's **SymbolFile** area. / 该文件是 LLDB **SymbolFile** 范围内的声明头文件。
- **Scale / 规模**: 89 lines with 8 direct includes. / 共 89 行，直接包含 8 个头文件。
- **Subsystem focus / 子系统关注点**: DWARF/PDB parsing, compile-unit indexing, symbol and type reconstruction. / DWARF/PDB 解析、编译单元索引、符号与类型重建。
- **Primary types / 主要类型**: `DWARFContext`, `DWARFDebugInfo`, `as`. / 主要类型包括 `DWARFContext`, `DWARFDebugInfo`, `as`。
- **Visible entry points / 关键入口**: `DWARFDebugInfo`, `GetNumUnits`, `GetUnitAtIndex`, `GetSkeletonUnit`, `GetTypeUnitForHash`, `ContainsTypeUnits`, `GetDIE`, `GetCompileUnitAranges`, `GetDwpSymbolFile`, `ParseUnitHeadersIfNeeded`. / 可见的关键入口包括 `DWARFDebugInfo`, `GetNumUnits`, `GetUnitAtIndex`, `GetSkeletonUnit`, `GetTypeUnitForHash`, `ContainsTypeUnits`, `GetDIE`, `GetCompileUnitAranges`, `GetDwpSymbolFile`, `ParseUnitHeadersIfNeeded`。
- **Namespaces / 命名空间**: `lldb_private::plugin`, `dwarf`. / 涉及的命名空间包括 `lldb_private::plugin`, `dwarf`。
- **Macros / 宏**: `LLDB_SOURCE_PLUGINS_SYMBOLFILE_DWARF_DWARFDEBUGINFO_H`. / 关键宏包括 `LLDB_SOURCE_PLUGINS_SYMBOLFILE_DWARF_DWARFDEBUGINFO_H`。
- **Concept / 概念**: LLVM error propagation. / LLVM 错误传播。

## Dependencies / 依赖关系

- **LLDB headers / LLDB 头文件**: `lldb/lldb-private.h`.
- **LLVM headers / LLVM 头文件**: `llvm/Support/Error.h`.
- **System/other headers / 系统或其他头文件**: `map`, `vector`, `DWARFDIE.h`, `DWARFTypeUnit.h`, `DWARFUnit.h`, `SymbolFileDWARF.h`.
- **Declared types / 声明类型**: `DWARFContext`, `DWARFDebugInfo`, `as`.
- **Callable interfaces / 可调用接口**: `DWARFDebugInfo`, `GetNumUnits`, `GetUnitAtIndex`, `GetSkeletonUnit`, `GetTypeUnitForHash`, `ContainsTypeUnits`, `GetDIE`, `GetCompileUnitAranges`, `GetDwpSymbolFile`, `ParseUnitHeadersIfNeeded`.
