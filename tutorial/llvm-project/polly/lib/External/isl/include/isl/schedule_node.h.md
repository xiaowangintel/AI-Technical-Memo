# schedule_node.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/include/isl/schedule_node.h` | `polly/lib/External/isl/include/isl/schedule_node.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Declares upstream ISL interfaces and C++ wrappers that Polly depends on for polyhedral operations. | 声明 Polly 依赖的上游 ISL 接口与 C++ 包装层，用于多面体计算。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

````cpp
#ifndef ISL_SCHEDULE_NODE_H
#define ISL_SCHEDULE_NODE_H

#include <isl/schedule_type.h>
#include <isl/union_set_type.h>
#include <isl/aff_type.h>
#include <isl/ast_type.h>
#include <isl/val_type.h>
#include <isl/space_type.h>
#include <isl/id_type.h>
#include <isl/set_type.h>

````
- **EN**: This block imports ISL headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines macros like `ISL_SCHEDULE_NODE_H`; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 引入周边逻辑所需的 ISL 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `ISL_SCHEDULE_NODE_H`; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 13-27

````cpp
#if defined(__cplusplus)
extern "C" {
#endif

__isl_export
__isl_give isl_schedule_node *isl_schedule_node_from_domain(
	__isl_take isl_union_set *domain);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_from_extension(
	__isl_take isl_union_map *extension);
__isl_give isl_schedule_node *isl_schedule_node_copy(
	__isl_keep isl_schedule_node *node);
__isl_null isl_schedule_node *isl_schedule_node_free(
	__isl_take isl_schedule_node *node);

````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or defines routines around `isl_schedule_node_from_domain`, `isl_schedule_node_from_extension`, `isl_schedule_node_copy`, `isl_schedule_node_free`; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或定义与 `isl_schedule_node_from_domain`, `isl_schedule_node_from_extension`, `isl_schedule_node_copy`, `isl_schedule_node_free` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 28-41

````cpp
__isl_export
isl_bool isl_schedule_node_is_equal(__isl_keep isl_schedule_node *node1,
	__isl_keep isl_schedule_node *node2);

isl_ctx *isl_schedule_node_get_ctx(__isl_keep isl_schedule_node *node);
__isl_subclass(isl_schedule_node)
enum isl_schedule_node_type isl_schedule_node_get_type(
	__isl_keep isl_schedule_node *node);
enum isl_schedule_node_type isl_schedule_node_get_parent_type(
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_schedule *isl_schedule_node_get_schedule(
	__isl_keep isl_schedule_node *node);

````
- **EN**: This block defines enum values such as `isl_schedule_node_type`; declares or defines routines around `isl_schedule_node_is_equal`, `isl_schedule_node_get_ctx`, `__isl_subclass`, `isl_schedule_node_get_type` (+2 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 定义枚举类型，例如 `isl_schedule_node_type`; 声明或定义与 `isl_schedule_node_is_equal`, `isl_schedule_node_get_ctx`, `__isl_subclass`, `isl_schedule_node_get_type` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 42-61

````cpp
__isl_export
isl_stat isl_schedule_node_foreach_descendant_top_down(
	__isl_keep isl_schedule_node *node,
	isl_bool (*fn)(__isl_keep isl_schedule_node *node, void *user),
	void *user);
__isl_export
isl_bool isl_schedule_node_every_descendant(__isl_keep isl_schedule_node *node,
	isl_bool (*test)(__isl_keep isl_schedule_node *node, void *user),
	void *user);
__isl_export
isl_stat isl_schedule_node_foreach_ancestor_top_down(
	__isl_keep isl_schedule_node *node,
	isl_stat (*fn)(__isl_keep isl_schedule_node *node, void *user),
	void *user);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_map_descendant_bottom_up(
	__isl_take isl_schedule_node *node,
	__isl_give isl_schedule_node *(*fn)(__isl_take isl_schedule_node *node,
		void *user), void *user);

````
- **EN**: This block declares or defines routines around `isl_schedule_node_foreach_descendant_top_down`, `isl_bool`, `isl_schedule_node_every_descendant`, `isl_schedule_node_foreach_ancestor_top_down` (+2 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_foreach_descendant_top_down`, `isl_bool`, `isl_schedule_node_every_descendant`, `isl_schedule_node_foreach_ancestor_top_down` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 62-77

````cpp
__isl_export
isl_size isl_schedule_node_get_tree_depth(__isl_keep isl_schedule_node *node);
__isl_export
isl_bool isl_schedule_node_has_parent(__isl_keep isl_schedule_node *node);
__isl_export
isl_bool isl_schedule_node_has_children(__isl_keep isl_schedule_node *node);
__isl_export
isl_bool isl_schedule_node_has_previous_sibling(
	__isl_keep isl_schedule_node *node);
__isl_export
isl_bool isl_schedule_node_has_next_sibling(__isl_keep isl_schedule_node *node);
__isl_export
isl_size isl_schedule_node_n_children(__isl_keep isl_schedule_node *node);
__isl_export
isl_size isl_schedule_node_get_child_position(
	__isl_keep isl_schedule_node *node);
````
- **EN**: This block declares or defines routines around `isl_schedule_node_get_tree_depth`, `isl_schedule_node_has_parent`, `isl_schedule_node_has_children`, `isl_schedule_node_has_previous_sibling` (+3 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_get_tree_depth`, `isl_schedule_node_has_parent`, `isl_schedule_node_has_children`, `isl_schedule_node_has_previous_sibling` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 78-93

````cpp
__isl_export
isl_size isl_schedule_node_get_ancestor_child_position(
	__isl_keep isl_schedule_node *node,
	__isl_keep isl_schedule_node *ancestor);
__isl_give isl_schedule_node *isl_schedule_node_get_child(
	__isl_keep isl_schedule_node *node, int pos);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_get_shared_ancestor(
	__isl_keep isl_schedule_node *node1,
	__isl_keep isl_schedule_node *node2);

__isl_export
__isl_give isl_schedule_node *isl_schedule_node_root(
	__isl_take isl_schedule_node *node);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_parent(
````
- **EN**: This block declares or defines routines around `isl_schedule_node_get_ancestor_child_position`, `isl_schedule_node_get_child`, `isl_schedule_node_get_shared_ancestor`, `isl_schedule_node_root` (+1 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_get_ancestor_child_position`, `isl_schedule_node_get_child`, `isl_schedule_node_get_shared_ancestor`, `isl_schedule_node_root` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 94-109

````cpp
	__isl_take isl_schedule_node *node);
__isl_give isl_schedule_node *isl_schedule_node_grandparent(
	__isl_take isl_schedule_node *node);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_ancestor(
	__isl_take isl_schedule_node *node, int generation);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_child(
	__isl_take isl_schedule_node *node, int pos);
__isl_give isl_schedule_node *isl_schedule_node_grandchild(
	__isl_take isl_schedule_node *node, int pos1, int pos2);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_first_child(
	__isl_take isl_schedule_node *node);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_previous_sibling(
````
- **EN**: This block declares or defines routines around `isl_schedule_node_grandparent`, `isl_schedule_node_ancestor`, `isl_schedule_node_child`, `isl_schedule_node_grandchild` (+2 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_grandparent`, `isl_schedule_node_ancestor`, `isl_schedule_node_child`, `isl_schedule_node_grandchild` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 110-121

````cpp
	__isl_take isl_schedule_node *node);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_next_sibling(
	__isl_take isl_schedule_node *node);

__isl_export
isl_bool isl_schedule_node_is_subtree_anchored(
	__isl_keep isl_schedule_node *node);

__isl_give isl_schedule_node *isl_schedule_node_group(
	__isl_take isl_schedule_node *node, __isl_take isl_id *group_id);

````
- **EN**: This block declares or defines routines around `isl_schedule_node_next_sibling`, `isl_schedule_node_is_subtree_anchored`, `isl_schedule_node_group`; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_next_sibling`, `isl_schedule_node_is_subtree_anchored`, `isl_schedule_node_group` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 122-137

````cpp
__isl_give isl_schedule_node *isl_schedule_node_sequence_splice_child(
	__isl_take isl_schedule_node *node, int pos);
__isl_give isl_schedule_node *isl_schedule_node_sequence_splice_children(
	__isl_take isl_schedule_node *node);

__isl_give isl_space *isl_schedule_node_band_get_space(
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_multi_union_pw_aff *isl_schedule_node_band_get_partial_schedule(
	__isl_keep isl_schedule_node *node);
__isl_give isl_union_map *isl_schedule_node_band_get_partial_schedule_union_map(
	__isl_keep isl_schedule_node *node);
enum isl_ast_loop_type isl_schedule_node_band_member_get_ast_loop_type(
	__isl_keep isl_schedule_node *node, int pos);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_band_member_set_ast_loop_type(
````
- **EN**: This block defines enum values such as `isl_ast_loop_type`; declares or defines routines around `isl_schedule_node_sequence_splice_child`, `isl_schedule_node_sequence_splice_children`, `isl_schedule_node_band_get_space`, `isl_schedule_node_band_get_partial_schedule` (+3 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 定义枚举类型，例如 `isl_ast_loop_type`; 声明或定义与 `isl_schedule_node_sequence_splice_child`, `isl_schedule_node_sequence_splice_children`, `isl_schedule_node_band_get_space`, `isl_schedule_node_band_get_partial_schedule` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 138-153

````cpp
	__isl_take isl_schedule_node *node, int pos,
	enum isl_ast_loop_type type);
enum isl_ast_loop_type isl_schedule_node_band_member_get_isolate_ast_loop_type(
	__isl_keep isl_schedule_node *node, int pos);
__isl_give isl_schedule_node *
isl_schedule_node_band_member_set_isolate_ast_loop_type(
	__isl_take isl_schedule_node *node, int pos,
	enum isl_ast_loop_type type);
__isl_export
__isl_give isl_union_set *isl_schedule_node_band_get_ast_build_options(
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_band_set_ast_build_options(
	__isl_take isl_schedule_node *node, __isl_take isl_union_set *options);
__isl_export
__isl_give isl_set *isl_schedule_node_band_get_ast_isolate_option(
````
- **EN**: This block defines enum values such as `isl_ast_loop_type`; declares or defines routines around `isl_schedule_node_band_member_get_isolate_ast_loop_type`, `isl_schedule_node_band_member_set_isolate_ast_loop_type`, `isl_schedule_node_band_get_ast_build_options`, `isl_schedule_node_band_set_ast_build_options` (+1 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 定义枚举类型，例如 `isl_ast_loop_type`; 声明或定义与 `isl_schedule_node_band_member_get_isolate_ast_loop_type`, `isl_schedule_node_band_member_set_isolate_ast_loop_type`, `isl_schedule_node_band_get_ast_build_options`, `isl_schedule_node_band_set_ast_build_options` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 154-169

````cpp
	__isl_keep isl_schedule_node *node);
__isl_export
isl_size isl_schedule_node_band_n_member(__isl_keep isl_schedule_node *node);
__isl_export
isl_bool isl_schedule_node_band_member_get_coincident(
	__isl_keep isl_schedule_node *node, int pos);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_band_member_set_coincident(
	__isl_take isl_schedule_node *node, int pos, int coincident);
__isl_export
isl_bool isl_schedule_node_band_get_permutable(
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_band_set_permutable(
	__isl_take isl_schedule_node *node, int permutable);

````
- **EN**: This block declares or defines routines around `isl_schedule_node_band_n_member`, `isl_schedule_node_band_member_get_coincident`, `isl_schedule_node_band_member_set_coincident`, `isl_schedule_node_band_get_permutable` (+1 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_band_n_member`, `isl_schedule_node_band_member_get_coincident`, `isl_schedule_node_band_member_set_coincident`, `isl_schedule_node_band_get_permutable` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 170-185

````cpp
isl_stat isl_options_set_tile_scale_tile_loops(isl_ctx *ctx, int val);
int isl_options_get_tile_scale_tile_loops(isl_ctx *ctx);
isl_stat isl_options_set_tile_shift_point_loops(isl_ctx *ctx, int val);
int isl_options_get_tile_shift_point_loops(isl_ctx *ctx);

__isl_export
__isl_give isl_schedule_node *isl_schedule_node_band_scale(
	__isl_take isl_schedule_node *node, __isl_take isl_multi_val *mv);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_band_scale_down(
	__isl_take isl_schedule_node *node, __isl_take isl_multi_val *mv);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_band_mod(
	__isl_take isl_schedule_node *node, __isl_take isl_multi_val *mv);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_band_shift(
````
- **EN**: This block declares or defines routines around `isl_options_set_tile_scale_tile_loops`, `isl_options_get_tile_scale_tile_loops`, `isl_options_set_tile_shift_point_loops`, `isl_options_get_tile_shift_point_loops` (+4 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_options_set_tile_scale_tile_loops`, `isl_options_get_tile_scale_tile_loops`, `isl_options_set_tile_shift_point_loops`, `isl_options_get_tile_shift_point_loops` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 186-201

````cpp
	__isl_take isl_schedule_node *node,
	__isl_take isl_multi_union_pw_aff *shift);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_band_tile(
	__isl_take isl_schedule_node *node, __isl_take isl_multi_val *sizes);
__isl_give isl_schedule_node *isl_schedule_node_band_sink(
	__isl_take isl_schedule_node *node);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_band_split(
	__isl_take isl_schedule_node *node, int pos);

__isl_export
__isl_give isl_set *isl_schedule_node_context_get_context(
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_union_set *isl_schedule_node_domain_get_domain(
````
- **EN**: This block declares or defines routines around `isl_schedule_node_band_tile`, `isl_schedule_node_band_sink`, `isl_schedule_node_band_split`, `isl_schedule_node_context_get_context` (+1 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_band_tile`, `isl_schedule_node_band_sink`, `isl_schedule_node_band_split`, `isl_schedule_node_context_get_context` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 202-220

````cpp
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_union_map *isl_schedule_node_expansion_get_expansion(
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_union_pw_multi_aff *isl_schedule_node_expansion_get_contraction(
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_union_map *isl_schedule_node_extension_get_extension(
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_union_set *isl_schedule_node_filter_get_filter(
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_set *isl_schedule_node_guard_get_guard(
	__isl_keep isl_schedule_node *node);
__isl_give isl_id *isl_schedule_node_mark_get_id(
	__isl_keep isl_schedule_node *node);

````
- **EN**: This block declares or defines routines around `isl_schedule_node_expansion_get_expansion`, `isl_schedule_node_expansion_get_contraction`, `isl_schedule_node_extension_get_extension`, `isl_schedule_node_filter_get_filter` (+2 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_expansion_get_expansion`, `isl_schedule_node_expansion_get_contraction`, `isl_schedule_node_extension_get_extension`, `isl_schedule_node_filter_get_filter` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 221-236

````cpp
isl_size isl_schedule_node_get_schedule_depth(
	__isl_keep isl_schedule_node *node);
__isl_give isl_union_set *isl_schedule_node_get_domain(
	__isl_keep isl_schedule_node *node);
__isl_give isl_union_set *isl_schedule_node_get_universe_domain(
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_multi_union_pw_aff *
isl_schedule_node_get_prefix_schedule_multi_union_pw_aff(
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_union_pw_multi_aff *
isl_schedule_node_get_prefix_schedule_union_pw_multi_aff(
	__isl_keep isl_schedule_node *node);
__isl_export
__isl_give isl_union_map *isl_schedule_node_get_prefix_schedule_union_map(
````
- **EN**: This block declares or defines routines around `isl_schedule_node_get_schedule_depth`, `isl_schedule_node_get_domain`, `isl_schedule_node_get_universe_domain`, `isl_schedule_node_get_prefix_schedule_multi_union_pw_aff` (+2 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_get_schedule_depth`, `isl_schedule_node_get_domain`, `isl_schedule_node_get_universe_domain`, `isl_schedule_node_get_prefix_schedule_multi_union_pw_aff` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 237-252

````cpp
	__isl_keep isl_schedule_node *node);
__isl_give isl_union_map *isl_schedule_node_get_prefix_schedule_relation(
	__isl_keep isl_schedule_node *node);
__isl_give isl_union_map *isl_schedule_node_get_subtree_schedule_union_map(
	__isl_keep isl_schedule_node *node);
__isl_give isl_union_map *isl_schedule_node_get_subtree_expansion(
	__isl_keep isl_schedule_node *node);
__isl_give isl_union_pw_multi_aff *isl_schedule_node_get_subtree_contraction(
	__isl_keep isl_schedule_node *node);

__isl_export
__isl_give isl_schedule_node *isl_schedule_node_insert_context(
	__isl_take isl_schedule_node *node, __isl_take isl_set *context);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_insert_partial_schedule(
	__isl_take isl_schedule_node *node,
````
- **EN**: This block declares or defines routines around `isl_schedule_node_get_prefix_schedule_relation`, `isl_schedule_node_get_subtree_schedule_union_map`, `isl_schedule_node_get_subtree_expansion`, `isl_schedule_node_get_subtree_contraction` (+2 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_get_prefix_schedule_relation`, `isl_schedule_node_get_subtree_schedule_union_map`, `isl_schedule_node_get_subtree_expansion`, `isl_schedule_node_get_subtree_contraction` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 253-271

````cpp
	__isl_take isl_multi_union_pw_aff *schedule);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_insert_filter(
	__isl_take isl_schedule_node *node, __isl_take isl_union_set *filter);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_insert_guard(
	__isl_take isl_schedule_node *node, __isl_take isl_set *context);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_insert_mark(
	__isl_take isl_schedule_node *node, __isl_take isl_id *mark);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_insert_sequence(
	__isl_take isl_schedule_node *node,
	__isl_take isl_union_set_list *filters);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_insert_set(
	__isl_take isl_schedule_node *node,
	__isl_take isl_union_set_list *filters);

````
- **EN**: This block declares or defines routines around `isl_schedule_node_insert_filter`, `isl_schedule_node_insert_guard`, `isl_schedule_node_insert_mark`, `isl_schedule_node_insert_sequence` (+1 more); uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_insert_filter`, `isl_schedule_node_insert_guard`, `isl_schedule_node_insert_mark`, `isl_schedule_node_insert_sequence` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 272-283

````cpp
__isl_give isl_schedule_node *isl_schedule_node_cut(
	__isl_take isl_schedule_node *node);
__isl_give isl_schedule_node *isl_schedule_node_delete(
	__isl_take isl_schedule_node *node);

__isl_export
__isl_give isl_schedule_node *isl_schedule_node_order_before(
	__isl_take isl_schedule_node *node, __isl_take isl_union_set *filter);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_order_after(
	__isl_take isl_schedule_node *node, __isl_take isl_union_set *filter);

````
- **EN**: This block declares or defines routines around `isl_schedule_node_cut`, `isl_schedule_node_delete`, `isl_schedule_node_order_before`, `isl_schedule_node_order_after`; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_cut`, `isl_schedule_node_delete`, `isl_schedule_node_order_before`, `isl_schedule_node_order_after` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 284-297

````cpp
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_graft_before(
	__isl_take isl_schedule_node *node,
	__isl_take isl_schedule_node *graft);
__isl_export
__isl_give isl_schedule_node *isl_schedule_node_graft_after(
	__isl_take isl_schedule_node *node,
	__isl_take isl_schedule_node *graft);

__isl_give isl_schedule_node *isl_schedule_node_reset_user(
	__isl_take isl_schedule_node *node);
__isl_give isl_schedule_node *isl_schedule_node_align_params(
	__isl_take isl_schedule_node *node, __isl_take isl_space *space);

````
- **EN**: This block declares or defines routines around `isl_schedule_node_graft_before`, `isl_schedule_node_graft_after`, `isl_schedule_node_reset_user`, `isl_schedule_node_align_params`; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 声明或定义与 `isl_schedule_node_graft_before`, `isl_schedule_node_graft_after`, `isl_schedule_node_reset_user`, `isl_schedule_node_align_params` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

### Lines 298-307

````cpp
__isl_give isl_printer *isl_printer_print_schedule_node(
	__isl_take isl_printer *p, __isl_keep isl_schedule_node *node);
void isl_schedule_node_dump(__isl_keep isl_schedule_node *node);
__isl_give char *isl_schedule_node_to_str(__isl_keep isl_schedule_node *node);

#if defined(__cplusplus)
}
#endif

#endif
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or defines routines around `isl_printer_print_schedule_node`, `isl_schedule_node_dump`, `isl_schedule_node_to_str`; uses ISL data structures or helpers for polyhedral reasoning; mentions schedule-related state that orders statements or iterations.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或定义与 `isl_printer_print_schedule_node`, `isl_schedule_node_dump`, `isl_schedule_node_to_str` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 涉及调度相关状态，用于安排语句或迭代顺序.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Schedule construction**
  - **CN**: 调度构建
- **Loop transformation**
  - **CN**: 循环变换
- **Public interface declarations**
  - **CN**: 公共接口声明

## Dependencies / 依赖关系

- **ISL headers**: `isl/schedule_type.h`, `isl/union_set_type.h`, `isl/aff_type.h`, `isl/ast_type.h`, `isl/val_type.h`, `isl/space_type.h`, `isl/id_type.h`, `isl/set_type.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/schedule_type.h`, `isl/union_set_type.h`, `isl/aff_type.h`, `isl/ast_type.h`, `isl/val_type.h`, `isl/space_type.h`, `isl/id_type.h`, `isl/set_type.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
