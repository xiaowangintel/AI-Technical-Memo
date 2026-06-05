# union_map.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/include/isl/union_map.h` | `polly/lib/External/isl/include/isl/union_map.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Declares upstream ISL interfaces and C++ wrappers that Polly depends on for polyhedral operations. | 声明 Polly 依赖的上游 ISL 接口与 C++ 包装层，用于多面体计算。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-15

````cpp
#ifndef ISL_UNION_MAP_H
#define ISL_UNION_MAP_H

#include <isl/stdint.h>
#include <isl/space_type.h>
#include <isl/aff_type.h>
#include <isl/map_type.h>
#include <isl/union_map_type.h>
#include <isl/printer.h>
#include <isl/val_type.h>

#if defined(__cplusplus)
extern "C" {
#endif

````
- **EN**: This block imports ISL headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines macros like `ISL_UNION_MAP_H`.
- **CN**: 该代码块 引入周边逻辑所需的 ISL 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `ISL_UNION_MAP_H`.

### Lines 16-31

````cpp
isl_size isl_union_map_dim(__isl_keep isl_union_map *umap,
	enum isl_dim_type type);
isl_bool isl_union_map_involves_dims(__isl_keep isl_union_map *umap,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_id *isl_union_map_get_dim_id(__isl_keep isl_union_map *umap,
	enum isl_dim_type type, unsigned pos);

__isl_constructor
__isl_give isl_union_map *isl_union_map_from_basic_map(
	__isl_take isl_basic_map *bmap);
__isl_export
__isl_give isl_union_map *isl_map_to_union_map(__isl_take isl_map *map);
__isl_constructor
__isl_give isl_union_map *isl_union_map_from_map(__isl_take isl_map *map);
__isl_overload
__isl_give isl_union_map *isl_union_map_empty_ctx(isl_ctx *ctx);
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_union_map_dim`, `isl_union_map_involves_dims`, `isl_union_map_get_dim_id`, `isl_union_map_from_basic_map` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_union_map_dim`, `isl_union_map_involves_dims`, `isl_union_map_get_dim_id`, `isl_union_map_from_basic_map` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 32-44

````cpp
__isl_give isl_union_map *isl_union_map_empty_space(
	__isl_take isl_space *space);
__isl_give isl_union_map *isl_union_map_empty(__isl_take isl_space *space);
__isl_give isl_union_map *isl_union_map_copy(__isl_keep isl_union_map *umap);
__isl_null isl_union_map *isl_union_map_free(__isl_take isl_union_map *umap);

isl_ctx *isl_union_map_get_ctx(__isl_keep isl_union_map *umap);
__isl_export
__isl_give isl_space *isl_union_map_get_space(__isl_keep isl_union_map *umap);

__isl_give isl_union_map *isl_union_map_reset_user(
	__isl_take isl_union_map *umap);

````
- **EN**: This block declares or defines routines around `isl_union_map_empty_space`, `isl_union_map_empty`, `isl_union_map_copy`, `isl_union_map_free` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_empty_space`, `isl_union_map_empty`, `isl_union_map_copy`, `isl_union_map_free` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 45-60

````cpp
int isl_union_map_find_dim_by_name(__isl_keep isl_union_map *umap,
	enum isl_dim_type type, const char *name);

__isl_export
__isl_give isl_union_map *isl_union_map_universe(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_set *isl_union_map_params(__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_set *isl_union_map_domain(__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_set *isl_union_map_range(__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_domain_map(
	__isl_take isl_union_map *umap);
__isl_export
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_union_map_find_dim_by_name`, `isl_union_map_universe`, `isl_union_map_params`, `isl_union_map_domain` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_union_map_find_dim_by_name`, `isl_union_map_universe`, `isl_union_map_params`, `isl_union_map_domain` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 61-74

````cpp
__isl_give isl_union_pw_multi_aff *isl_union_map_domain_map_union_pw_multi_aff(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_range_map(
	__isl_take isl_union_map *umap);
__isl_give isl_union_map *isl_union_set_wrapped_domain_map(
	__isl_take isl_union_set *uset);
__isl_export
__isl_give isl_union_map *isl_union_map_from_domain(
	__isl_take isl_union_set *uset);
__isl_export
__isl_give isl_union_map *isl_union_map_from_range(
	__isl_take isl_union_set *uset);

````
- **EN**: This block declares or defines routines around `isl_union_map_domain_map_union_pw_multi_aff`, `isl_union_map_range_map`, `isl_union_set_wrapped_domain_map`, `isl_union_map_from_domain` (+1 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_domain_map_union_pw_multi_aff`, `isl_union_map_range_map`, `isl_union_set_wrapped_domain_map`, `isl_union_map_from_domain` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 75-90

````cpp
__isl_export
__isl_give isl_union_map *isl_union_map_affine_hull(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_polyhedral_hull(
	__isl_take isl_union_map *umap);
__isl_give isl_union_map *isl_union_map_remove_redundancies(
	__isl_take isl_union_map *umap);
__isl_give isl_union_map *isl_union_map_simple_hull(
	__isl_take isl_union_map *umap);
__isl_give isl_union_map *isl_union_map_plain_unshifted_simple_hull(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_coalesce(
	__isl_take isl_union_map *umap);
__isl_export
````
- **EN**: This block declares or defines routines around `isl_union_map_affine_hull`, `isl_union_map_polyhedral_hull`, `isl_union_map_remove_redundancies`, `isl_union_map_simple_hull` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_affine_hull`, `isl_union_map_polyhedral_hull`, `isl_union_map_remove_redundancies`, `isl_union_map_simple_hull` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 91-106

````cpp
__isl_give isl_union_map *isl_union_map_compute_divs(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_lexmin(__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_lexmax(__isl_take isl_union_map *umap);

__isl_give isl_union_map *isl_union_map_add_map(__isl_take isl_union_map *umap,
	__isl_take isl_map *map);
__isl_export
__isl_give isl_union_map *isl_union_map_union(__isl_take isl_union_map *umap1,
	__isl_take isl_union_map *umap2);
__isl_export
__isl_give isl_union_map *isl_union_map_subtract(
	__isl_take isl_union_map *umap1, __isl_take isl_union_map *umap2);
__isl_export
````
- **EN**: This block declares or defines routines around `isl_union_map_compute_divs`, `isl_union_map_lexmin`, `isl_union_map_lexmax`, `isl_union_map_add_map` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_compute_divs`, `isl_union_map_lexmin`, `isl_union_map_lexmax`, `isl_union_map_add_map` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 107-122

````cpp
__isl_give isl_union_map *isl_union_map_intersect(
	__isl_take isl_union_map *umap1, __isl_take isl_union_map *umap2);
__isl_export
__isl_give isl_union_map *isl_union_map_intersect_params(
	__isl_take isl_union_map *umap, __isl_take isl_set *set);
__isl_export
__isl_give isl_union_map *isl_union_map_product(__isl_take isl_union_map *umap1,
	__isl_take isl_union_map *umap2);
__isl_export
__isl_give isl_union_map *isl_union_map_domain_product(
	__isl_take isl_union_map *umap1, __isl_take isl_union_map *umap2);
__isl_give isl_union_map *isl_union_map_flat_domain_product(
	__isl_take isl_union_map *umap1, __isl_take isl_union_map *umap2);
__isl_export
__isl_give isl_union_map *isl_union_map_range_product(
	__isl_take isl_union_map *umap1, __isl_take isl_union_map *umap2);
````
- **EN**: This block declares or defines routines around `isl_union_map_intersect`, `isl_union_map_intersect_params`, `isl_union_map_product`, `isl_union_map_domain_product` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_intersect`, `isl_union_map_intersect_params`, `isl_union_map_product`, `isl_union_map_domain_product` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 123-138

````cpp
__isl_give isl_union_map *isl_union_map_flat_range_product(
	__isl_take isl_union_map *umap1, __isl_take isl_union_map *umap2);
__isl_export
__isl_give isl_union_map *isl_union_map_domain_factor_domain(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_domain_factor_range(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_range_factor_domain(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_range_factor_range(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_factor_domain(
````
- **EN**: This block declares or defines routines around `isl_union_map_flat_range_product`, `isl_union_map_domain_factor_domain`, `isl_union_map_domain_factor_range`, `isl_union_map_range_factor_domain` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_flat_range_product`, `isl_union_map_domain_factor_domain`, `isl_union_map_domain_factor_range`, `isl_union_map_range_factor_domain` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 139-155

````cpp
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_factor_range(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_gist(__isl_take isl_union_map *umap,
	__isl_take isl_union_map *context);
__isl_export
__isl_give isl_union_map *isl_union_map_gist_params(
	__isl_take isl_union_map *umap, __isl_take isl_set *set);
__isl_export
__isl_give isl_union_map *isl_union_map_gist_domain(
	__isl_take isl_union_map *umap, __isl_take isl_union_set *uset);
__isl_export
__isl_give isl_union_map *isl_union_map_gist_range(
	__isl_take isl_union_map *umap, __isl_take isl_union_set *uset);

````
- **EN**: This block declares or defines routines around `isl_union_map_factor_range`, `isl_union_map_gist`, `isl_union_map_gist_params`, `isl_union_map_gist_domain` (+1 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_factor_range`, `isl_union_map_gist`, `isl_union_map_gist_params`, `isl_union_map_gist_domain` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 156-171

````cpp
__isl_overload
__isl_give isl_union_map *isl_union_map_intersect_domain_union_set(
	__isl_take isl_union_map *umap, __isl_take isl_union_set *uset);
__isl_overload
__isl_give isl_union_map *isl_union_map_intersect_domain_space(
	__isl_take isl_union_map *umap, __isl_take isl_space *space);
__isl_give isl_union_map *isl_union_map_intersect_domain(
	__isl_take isl_union_map *umap, __isl_take isl_union_set *uset);
__isl_overload
__isl_give isl_union_map *isl_union_map_intersect_range_union_set(
	__isl_take isl_union_map *umap, __isl_take isl_union_set *uset);
__isl_overload
__isl_give isl_union_map *isl_union_map_intersect_range_space(
	__isl_take isl_union_map *umap, __isl_take isl_space *space);
__isl_give isl_union_map *isl_union_map_intersect_range(
	__isl_take isl_union_map *umap, __isl_take isl_union_set *uset);
````
- **EN**: This block declares or defines routines around `isl_union_map_intersect_domain_union_set`, `isl_union_map_intersect_domain_space`, `isl_union_map_intersect_domain`, `isl_union_map_intersect_range_union_set` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_intersect_domain_union_set`, `isl_union_map_intersect_domain_space`, `isl_union_map_intersect_domain`, `isl_union_map_intersect_range_union_set` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 172-187

````cpp
__isl_export
__isl_give isl_union_map *isl_union_map_intersect_domain_factor_domain(
	__isl_take isl_union_map *umap, __isl_take isl_union_map *factor);
__isl_export
__isl_give isl_union_map *isl_union_map_intersect_domain_factor_range(
	__isl_take isl_union_map *umap, __isl_take isl_union_map *factor);
__isl_export
__isl_give isl_union_map *isl_union_map_intersect_range_factor_domain(
	__isl_take isl_union_map *umap, __isl_take isl_union_map *factor);
__isl_export
__isl_give isl_union_map *isl_union_map_intersect_range_factor_range(
	__isl_take isl_union_map *umap, __isl_take isl_union_map *factor);
__isl_overload
__isl_give isl_union_map *
isl_union_map_intersect_domain_wrapped_domain_union_set(
	__isl_take isl_union_map *umap, __isl_take isl_union_set *domain);
````
- **EN**: This block declares or defines routines around `isl_union_map_intersect_domain_factor_domain`, `isl_union_map_intersect_domain_factor_range`, `isl_union_map_intersect_range_factor_domain`, `isl_union_map_intersect_range_factor_range` (+1 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_intersect_domain_factor_domain`, `isl_union_map_intersect_domain_factor_range`, `isl_union_map_intersect_range_factor_domain`, `isl_union_map_intersect_range_factor_range` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 188-199

````cpp
__isl_overload
__isl_give isl_union_map *
isl_union_map_intersect_range_wrapped_domain_union_set(
	__isl_take isl_union_map *umap, __isl_take isl_union_set *domain);

__isl_export
__isl_give isl_union_map *isl_union_map_subtract_domain(
	__isl_take isl_union_map *umap, __isl_take isl_union_set *dom);
__isl_export
__isl_give isl_union_map *isl_union_map_subtract_range(
	__isl_take isl_union_map *umap, __isl_take isl_union_set *dom);

````
- **EN**: This block declares or defines routines around `isl_union_map_intersect_range_wrapped_domain_union_set`, `isl_union_map_subtract_domain`, `isl_union_map_subtract_range`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_intersect_range_wrapped_domain_union_set`, `isl_union_map_subtract_domain`, `isl_union_map_subtract_range` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 200-215

````cpp
__isl_export
__isl_give isl_union_map *isl_union_map_apply_domain(
	__isl_take isl_union_map *umap1, __isl_take isl_union_map *umap2);
__isl_export
__isl_give isl_union_map *isl_union_map_apply_range(
	__isl_take isl_union_map *umap1, __isl_take isl_union_map *umap2);
__isl_overload
__isl_give isl_union_map *isl_union_map_preimage_domain_multi_aff(
	__isl_take isl_union_map *umap, __isl_take isl_multi_aff *ma);
__isl_overload
__isl_give isl_union_map *isl_union_map_preimage_range_multi_aff(
	__isl_take isl_union_map *umap, __isl_take isl_multi_aff *ma);
__isl_overload
__isl_give isl_union_map *isl_union_map_preimage_domain_pw_multi_aff(
	__isl_take isl_union_map *umap, __isl_take isl_pw_multi_aff *pma);
__isl_overload
````
- **EN**: This block declares or defines routines around `isl_union_map_apply_domain`, `isl_union_map_apply_range`, `isl_union_map_preimage_domain_multi_aff`, `isl_union_map_preimage_range_multi_aff` (+1 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_apply_domain`, `isl_union_map_apply_range`, `isl_union_map_preimage_domain_multi_aff`, `isl_union_map_preimage_range_multi_aff` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 216-231

````cpp
__isl_give isl_union_map *isl_union_map_preimage_range_pw_multi_aff(
	__isl_take isl_union_map *umap, __isl_take isl_pw_multi_aff *pma);
__isl_overload
__isl_give isl_union_map *isl_union_map_preimage_domain_multi_pw_aff(
	__isl_take isl_union_map *umap, __isl_take isl_multi_pw_aff *mpa);
__isl_overload
__isl_give isl_union_map *isl_union_map_preimage_domain_union_pw_multi_aff(
	__isl_take isl_union_map *umap,
	__isl_take isl_union_pw_multi_aff *upma);
__isl_overload
__isl_give isl_union_map *isl_union_map_preimage_range_union_pw_multi_aff(
	__isl_take isl_union_map *umap,
	__isl_take isl_union_pw_multi_aff *upma);
__isl_export
__isl_give isl_union_map *isl_union_map_reverse(__isl_take isl_union_map *umap);
__isl_export
````
- **EN**: This block declares or defines routines around `isl_union_map_preimage_range_pw_multi_aff`, `isl_union_map_preimage_domain_multi_pw_aff`, `isl_union_map_preimage_domain_union_pw_multi_aff`, `isl_union_map_preimage_range_union_pw_multi_aff` (+1 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_preimage_range_pw_multi_aff`, `isl_union_map_preimage_domain_multi_pw_aff`, `isl_union_map_preimage_domain_union_pw_multi_aff`, `isl_union_map_preimage_range_union_pw_multi_aff` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 232-250

````cpp
__isl_give isl_union_map *isl_union_map_domain_reverse(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_range_reverse(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_from_domain_and_range(
	__isl_take isl_union_set *domain, __isl_take isl_union_set *range);

__isl_export
__isl_give isl_union_map *isl_union_map_detect_equalities(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_set *isl_union_map_deltas(__isl_take isl_union_map *umap);
__isl_give isl_union_map *isl_union_map_deltas_map(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_set_identity(__isl_take isl_union_set *uset);

````
- **EN**: This block declares or defines routines around `isl_union_map_domain_reverse`, `isl_union_map_range_reverse`, `isl_union_map_from_domain_and_range`, `isl_union_map_detect_equalities` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_domain_reverse`, `isl_union_map_range_reverse`, `isl_union_map_from_domain_and_range`, `isl_union_map_detect_equalities` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 251-268

````cpp
__isl_overload
__isl_give isl_union_map *isl_union_map_project_out_param_id(
	__isl_take isl_union_map *umap, __isl_take isl_id *id);
__isl_overload
__isl_give isl_union_map *isl_union_map_project_out_param_id_list(
	__isl_take isl_union_map *umap, __isl_take isl_id_list *list);
__isl_give isl_union_map *isl_union_map_project_out(
	__isl_take isl_union_map *umap,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_export
__isl_give isl_union_map *isl_union_map_project_out_all_params(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_drop_unused_params(
	__isl_take isl_union_map *umap);
__isl_give isl_union_map *isl_union_map_remove_divs(
	__isl_take isl_union_map *bmap);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_union_map_project_out_param_id`, `isl_union_map_project_out_param_id_list`, `isl_union_map_project_out`, `isl_union_map_project_out_all_params` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_union_map_project_out_param_id`, `isl_union_map_project_out_param_id_list`, `isl_union_map_project_out`, `isl_union_map_project_out_all_params` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 269-284

````cpp
__isl_export
__isl_give isl_union_set *isl_union_map_bind_range(
	__isl_take isl_union_map *umap, __isl_take isl_multi_id *tuple);

isl_bool isl_union_map_plain_is_empty(__isl_keep isl_union_map *umap);
__isl_export
isl_bool isl_union_map_is_empty(__isl_keep isl_union_map *umap);
__isl_export
isl_bool isl_union_map_is_single_valued(__isl_keep isl_union_map *umap);
isl_bool isl_union_map_plain_is_injective(__isl_keep isl_union_map *umap);
__isl_export
isl_bool isl_union_map_is_injective(__isl_keep isl_union_map *umap);
__isl_export
isl_bool isl_union_map_is_bijective(__isl_keep isl_union_map *umap);
isl_bool isl_union_map_is_identity(__isl_keep isl_union_map *umap);

````
- **EN**: This block declares or defines routines around `isl_union_map_bind_range`, `isl_union_map_plain_is_empty`, `isl_union_map_is_empty`, `isl_union_map_is_single_valued` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_bind_range`, `isl_union_map_plain_is_empty`, `isl_union_map_is_empty`, `isl_union_map_is_single_valued` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 285-297

````cpp
__isl_export
isl_bool isl_union_map_is_subset(__isl_keep isl_union_map *umap1,
	__isl_keep isl_union_map *umap2);
__isl_export
isl_bool isl_union_map_is_equal(__isl_keep isl_union_map *umap1,
	__isl_keep isl_union_map *umap2);
__isl_export
isl_bool isl_union_map_is_disjoint(__isl_keep isl_union_map *umap1,
	__isl_keep isl_union_map *umap2);
__isl_export
isl_bool isl_union_map_is_strict_subset(__isl_keep isl_union_map *umap1,
	__isl_keep isl_union_map *umap2);

````
- **EN**: This block declares or defines routines around `isl_union_map_is_subset`, `isl_union_map_is_equal`, `isl_union_map_is_disjoint`, `isl_union_map_is_strict_subset`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_is_subset`, `isl_union_map_is_equal`, `isl_union_map_is_disjoint`, `isl_union_map_is_strict_subset` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 298-313

````cpp
uint32_t isl_union_map_get_hash(__isl_keep isl_union_map *umap);

isl_size isl_union_map_n_map(__isl_keep isl_union_map *umap);
__isl_export
isl_stat isl_union_map_foreach_map(__isl_keep isl_union_map *umap,
	isl_stat (*fn)(__isl_take isl_map *map, void *user), void *user);
__isl_export
__isl_give isl_map_list *isl_union_map_get_map_list(
	__isl_keep isl_union_map *umap);
__isl_export
isl_bool isl_union_map_every_map(__isl_keep isl_union_map *umap,
	isl_bool (*test)(__isl_keep isl_map *map, void *user), void *user);
__isl_give isl_union_map *isl_union_map_remove_map_if(
	__isl_take isl_union_map *umap,
	isl_bool (*fn)(__isl_keep isl_map *map, void *user), void *user);
isl_bool isl_union_map_contains(__isl_keep isl_union_map *umap,
````
- **EN**: This block declares or defines routines around `isl_union_map_get_hash`, `isl_union_map_n_map`, `isl_union_map_foreach_map`, `isl_stat` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_get_hash`, `isl_union_map_n_map`, `isl_union_map_foreach_map`, `isl_stat` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 314-325

````cpp
	__isl_keep isl_space *space);
__isl_export
__isl_give isl_map *isl_union_map_extract_map(__isl_keep isl_union_map *umap,
	__isl_take isl_space *space);
__isl_export
isl_bool isl_union_map_isa_map(__isl_keep isl_union_map *umap);
__isl_export
__isl_give isl_map *isl_union_map_as_map(__isl_take isl_union_map *umap);
__isl_give isl_map *isl_map_from_union_map(__isl_take isl_union_map *umap);

__isl_give isl_basic_map *isl_union_map_sample(__isl_take isl_union_map *umap);

````
- **EN**: This block declares or defines routines around `isl_union_map_extract_map`, `isl_union_map_isa_map`, `isl_union_map_as_map`, `isl_map_from_union_map` (+1 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_extract_map`, `isl_union_map_isa_map`, `isl_union_map_as_map`, `isl_map_from_union_map` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 326-342

````cpp
__isl_overload
__isl_give isl_union_map *isl_union_map_fixed_power_val(
	__isl_take isl_union_map *umap, __isl_take isl_val *exp);
__isl_give isl_union_map *isl_union_map_power(__isl_take isl_union_map *umap,
	isl_bool *exact);
__isl_give isl_union_map *isl_union_map_transitive_closure(
	__isl_take isl_union_map *umap, isl_bool *exact);

__isl_give isl_union_map *isl_union_map_lex_lt_union_map(
	__isl_take isl_union_map *umap1, __isl_take isl_union_map *umap2);
__isl_give isl_union_map *isl_union_map_lex_le_union_map(
	__isl_take isl_union_map *umap1, __isl_take isl_union_map *umap2);
__isl_give isl_union_map *isl_union_map_lex_gt_union_map(
	__isl_take isl_union_map *umap1, __isl_take isl_union_map *umap2);
__isl_give isl_union_map *isl_union_map_lex_ge_union_map(
	__isl_take isl_union_map *umap1, __isl_take isl_union_map *umap2);

````
- **EN**: This block declares or defines routines around `isl_union_map_fixed_power_val`, `isl_union_map_power`, `isl_union_map_transitive_closure`, `isl_union_map_lex_lt_union_map` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_fixed_power_val`, `isl_union_map_power`, `isl_union_map_transitive_closure`, `isl_union_map_lex_lt_union_map` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 343-359

````cpp
__isl_overload
__isl_give isl_union_map *isl_union_map_eq_at_multi_union_pw_aff(
	__isl_take isl_union_map *umap,
	__isl_take isl_multi_union_pw_aff *mupa);
__isl_give isl_union_map *isl_union_map_lex_le_at_multi_union_pw_aff(
	__isl_take isl_union_map *umap,
	__isl_take isl_multi_union_pw_aff *mupa);
__isl_give isl_union_map *isl_union_map_lex_lt_at_multi_union_pw_aff(
	__isl_take isl_union_map *umap,
	__isl_take isl_multi_union_pw_aff *mupa);
__isl_give isl_union_map *isl_union_map_lex_ge_at_multi_union_pw_aff(
	__isl_take isl_union_map *umap,
	__isl_take isl_multi_union_pw_aff *mupa);
__isl_give isl_union_map *isl_union_map_lex_gt_at_multi_union_pw_aff(
	__isl_take isl_union_map *umap,
	__isl_take isl_multi_union_pw_aff *mupa);

````
- **EN**: This block declares or defines routines around `isl_union_map_eq_at_multi_union_pw_aff`, `isl_union_map_lex_le_at_multi_union_pw_aff`, `isl_union_map_lex_lt_at_multi_union_pw_aff`, `isl_union_map_lex_ge_at_multi_union_pw_aff` (+1 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_eq_at_multi_union_pw_aff`, `isl_union_map_lex_le_at_multi_union_pw_aff`, `isl_union_map_lex_lt_at_multi_union_pw_aff`, `isl_union_map_lex_ge_at_multi_union_pw_aff` (+1 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 360-374

````cpp
__isl_give isl_union_map *isl_union_map_read_from_file(isl_ctx *ctx,
	FILE *input);
__isl_constructor
__isl_give isl_union_map *isl_union_map_read_from_str(isl_ctx *ctx,
	const char *str);
__isl_give char *isl_union_map_to_str(__isl_keep isl_union_map *umap);
__isl_give isl_printer *isl_printer_print_union_map(__isl_take isl_printer *p,
	__isl_keep isl_union_map *umap);
void isl_union_map_dump(__isl_keep isl_union_map *umap);

__isl_export
__isl_give isl_union_set *isl_union_map_wrap(__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_set_unwrap(__isl_take isl_union_set *uset);

````
- **EN**: This block declares or defines routines around `isl_union_map_read_from_file`, `isl_union_map_read_from_str`, `isl_union_map_to_str`, `isl_printer_print_union_map` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_read_from_file`, `isl_union_map_read_from_str`, `isl_union_map_to_str`, `isl_printer_print_union_map` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 375-388

````cpp
__isl_export
__isl_give isl_union_map *isl_union_map_zip(__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_curry(__isl_take isl_union_map *umap);
__isl_give isl_union_map *isl_union_map_range_curry(
	__isl_take isl_union_map *umap);
__isl_export
__isl_give isl_union_map *isl_union_map_uncurry(__isl_take isl_union_map *umap);

__isl_give isl_union_map *isl_union_map_align_params(
	__isl_take isl_union_map *umap, __isl_take isl_space *model);
__isl_give isl_union_set *isl_union_set_align_params(
	__isl_take isl_union_set *uset, __isl_take isl_space *model);

````
- **EN**: This block declares or defines routines around `isl_union_map_zip`, `isl_union_map_curry`, `isl_union_map_range_curry`, `isl_union_map_uncurry` (+2 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_map_zip`, `isl_union_map_curry`, `isl_union_map_range_curry`, `isl_union_map_uncurry` (+2 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 389-395

````cpp
ISL_DECLARE_LIST_FN(union_map)

#if defined(__cplusplus)
}
#endif

#endif
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or defines routines around `ISL_DECLARE_LIST_FN`.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或定义与 `ISL_DECLARE_LIST_FN` 相关的例程.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Public interface declarations**
  - **CN**: 公共接口声明

## Dependencies / 依赖关系

- **ISL headers**: `isl/stdint.h`, `isl/space_type.h`, `isl/aff_type.h`, `isl/map_type.h`, `isl/union_map_type.h`, `isl/printer.h`, `isl/val_type.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/stdint.h`, `isl/space_type.h`, `isl/aff_type.h`, `isl/map_type.h`, `isl/union_map_type.h`, `isl/printer.h`, `isl/val_type.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
