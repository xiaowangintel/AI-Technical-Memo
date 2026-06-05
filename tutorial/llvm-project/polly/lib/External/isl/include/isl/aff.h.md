# aff.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/include/isl/aff.h` | `polly/lib/External/isl/include/isl/aff.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Declares upstream ISL interfaces and C++ wrappers that Polly depends on for polyhedral operations. | 声明 Polly 依赖的上游 ISL 接口与 C++ 包装层，用于多面体计算。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-38

````cpp
#ifndef ISL_AFF_H
#define ISL_AFF_H

#include <isl/stdint.h>
#include <isl/local_space.h>
#include <isl/printer.h>
#include <isl/id_type.h>
#include <isl/set_type.h>
#include <isl/aff_type.h>
#include <isl/list.h>
#include <isl/multi.h>
#include <isl/union_set_type.h>
#include <isl/val_type.h>
#include <isl/point.h>

#if defined(__cplusplus)
extern "C" {
#endif

__isl_overload
__isl_give isl_aff *isl_aff_zero_on_domain_space(__isl_take isl_space *space);
__isl_export
__isl_give isl_aff *isl_space_zero_aff_on_domain(__isl_take isl_space *space);
__isl_give isl_aff *isl_aff_zero_on_domain(__isl_take isl_local_space *ls);
__isl_give isl_aff *isl_aff_val_on_domain_space(__isl_take isl_space *space,
	__isl_take isl_val *val);
__isl_give isl_aff *isl_aff_val_on_domain(__isl_take isl_local_space *ls,
	__isl_take isl_val *val);
__isl_give isl_aff *isl_aff_var_on_domain(__isl_take isl_local_space *ls,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_aff *isl_aff_nan_on_domain_space(__isl_take isl_space *space);
__isl_give isl_aff *isl_aff_nan_on_domain(__isl_take isl_local_space *ls);
__isl_give isl_aff *isl_aff_param_on_domain_space_id(
	__isl_take isl_space *space, __isl_take isl_id *id);
__isl_overload
__isl_give isl_aff *isl_space_param_aff_on_domain_id(
	__isl_take isl_space *space, __isl_take isl_id *id);

````
- **EN**: This block imports ISL headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines enum values such as `isl_dim_type`; defines macros like `ISL_AFF_H`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 ISL 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义枚举类型，例如 `isl_dim_type`; 定义宏，例如 `ISL_AFF_H`；并延续周边实现细节。

### Lines 39-70

````cpp
__isl_give isl_aff *isl_aff_copy(__isl_keep isl_aff *aff);
__isl_null isl_aff *isl_aff_free(__isl_take isl_aff *aff);

isl_ctx *isl_aff_get_ctx(__isl_keep isl_aff *aff);
uint32_t isl_aff_get_hash(__isl_keep isl_aff *aff);

isl_bool isl_aff_involves_locals(__isl_keep isl_aff *aff);

isl_size isl_aff_dim(__isl_keep isl_aff *aff, enum isl_dim_type type);
isl_bool isl_aff_involves_dims(__isl_keep isl_aff *aff,
	enum isl_dim_type type, unsigned first, unsigned n);

__isl_give isl_space *isl_aff_get_domain_space(__isl_keep isl_aff *aff);
__isl_give isl_space *isl_aff_get_space(__isl_keep isl_aff *aff);
__isl_give isl_local_space *isl_aff_get_domain_local_space(
	__isl_keep isl_aff *aff);
__isl_give isl_local_space *isl_aff_get_local_space(__isl_keep isl_aff *aff);

const char *isl_aff_get_dim_name(__isl_keep isl_aff *aff,
	enum isl_dim_type type, unsigned pos);
__isl_export
__isl_give isl_val *isl_aff_get_constant_val(__isl_keep isl_aff *aff);
__isl_give isl_val *isl_aff_get_coefficient_val(__isl_keep isl_aff *aff,
	enum isl_dim_type type, int pos);
int isl_aff_coefficient_sgn(__isl_keep isl_aff *aff,
	enum isl_dim_type type, int pos);
__isl_give isl_val *isl_aff_get_denominator_val(__isl_keep isl_aff *aff);
__isl_give isl_aff *isl_aff_set_constant_si(__isl_take isl_aff *aff, int v);
__isl_give isl_aff *isl_aff_set_constant_val(__isl_take isl_aff *aff,
	__isl_take isl_val *v);
__isl_give isl_aff *isl_aff_set_coefficient_si(__isl_take isl_aff *aff,
	enum isl_dim_type type, int pos, int v);
````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_aff_copy`, `isl_aff_free`, `isl_aff_get_ctx`, `isl_aff_get_hash` (+15 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_aff_copy`, `isl_aff_free`, `isl_aff_get_ctx`, `isl_aff_get_hash` (+15 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 71-101

````cpp
__isl_give isl_aff *isl_aff_set_coefficient_val(__isl_take isl_aff *aff,
	enum isl_dim_type type, int pos, __isl_take isl_val *v);
__isl_give isl_aff *isl_aff_add_constant_si(__isl_take isl_aff *aff, int v);
__isl_overload
__isl_give isl_aff *isl_aff_add_constant_val(__isl_take isl_aff *aff,
	__isl_take isl_val *v);
__isl_give isl_aff *isl_aff_add_constant_num_si(__isl_take isl_aff *aff, int v);
__isl_give isl_aff *isl_aff_add_coefficient_si(__isl_take isl_aff *aff,
	enum isl_dim_type type, int pos, int v);
__isl_give isl_aff *isl_aff_add_coefficient_val(__isl_take isl_aff *aff,
	enum isl_dim_type type, int pos, __isl_take isl_val *v);

__isl_export
isl_bool isl_aff_is_cst(__isl_keep isl_aff *aff);

__isl_give isl_aff *isl_aff_set_tuple_id(__isl_take isl_aff *aff,
	enum isl_dim_type type, __isl_take isl_id *id);
__isl_give isl_aff *isl_aff_set_dim_name(__isl_take isl_aff *aff,
	enum isl_dim_type type, unsigned pos, const char *s);
__isl_give isl_aff *isl_aff_set_dim_id(__isl_take isl_aff *aff,
	enum isl_dim_type type, unsigned pos, __isl_take isl_id *id);

int isl_aff_find_dim_by_name(__isl_keep isl_aff *aff, enum isl_dim_type type,
	const char *name);

__isl_export
isl_bool isl_aff_plain_is_equal(__isl_keep isl_aff *aff1,
	__isl_keep isl_aff *aff2);
isl_bool isl_aff_plain_is_zero(__isl_keep isl_aff *aff);
isl_bool isl_aff_is_nan(__isl_keep isl_aff *aff);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_aff_set_coefficient_val`, `isl_aff_add_constant_si`, `isl_aff_add_constant_val`, `isl_aff_add_constant_num_si` (+10 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_aff_set_coefficient_val`, `isl_aff_add_constant_si`, `isl_aff_add_constant_val`, `isl_aff_add_constant_num_si` (+10 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 102-128

````cpp
__isl_give isl_aff *isl_aff_get_div(__isl_keep isl_aff *aff, int pos);

__isl_give isl_aff *isl_aff_from_range(__isl_take isl_aff *aff);

__isl_export
__isl_give isl_aff *isl_aff_neg(__isl_take isl_aff *aff);
__isl_export
__isl_give isl_aff *isl_aff_ceil(__isl_take isl_aff *aff);
__isl_export
__isl_give isl_aff *isl_aff_floor(__isl_take isl_aff *aff);
__isl_overload
__isl_give isl_aff *isl_aff_mod_val(__isl_take isl_aff *aff,
	__isl_take isl_val *mod);

__isl_export
__isl_give isl_aff *isl_aff_mul(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_export
__isl_give isl_aff *isl_aff_div(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_export
__isl_give isl_aff *isl_aff_add(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_export
__isl_give isl_aff *isl_aff_sub(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);

````
- **EN**: This block declares or defines routines around `isl_aff_get_div`, `isl_aff_from_range`, `isl_aff_neg`, `isl_aff_ceil` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_aff_get_div`, `isl_aff_from_range`, `isl_aff_neg`, `isl_aff_ceil` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 129-155

````cpp
__isl_overload
__isl_give isl_aff *isl_aff_scale_val(__isl_take isl_aff *aff,
	__isl_take isl_val *v);
__isl_give isl_aff *isl_aff_scale_down_ui(__isl_take isl_aff *aff, unsigned f);
__isl_overload
__isl_give isl_aff *isl_aff_scale_down_val(__isl_take isl_aff *aff,
	__isl_take isl_val *v);

__isl_export
__isl_give isl_aff *isl_aff_domain_reverse(__isl_take isl_aff *aff);
__isl_give isl_aff *isl_aff_insert_dims(__isl_take isl_aff *aff,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_aff *isl_aff_add_dims(__isl_take isl_aff *aff,
	enum isl_dim_type type, unsigned n);
__isl_give isl_aff *isl_aff_move_dims(__isl_take isl_aff *aff,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n);
__isl_give isl_aff *isl_aff_drop_dims(__isl_take isl_aff *aff,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_aff *isl_aff_project_domain_on_params(__isl_take isl_aff *aff);
__isl_export
__isl_give isl_aff *isl_aff_unbind_params_insert_domain(
	__isl_take isl_aff *aff, __isl_take isl_multi_id *domain);

__isl_give isl_aff *isl_aff_align_params(__isl_take isl_aff *aff,
	__isl_take isl_space *model);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_aff_scale_val`, `isl_aff_scale_down_ui`, `isl_aff_scale_down_val`, `isl_aff_domain_reverse` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_aff_scale_val`, `isl_aff_scale_down_ui`, `isl_aff_scale_down_val`, `isl_aff_domain_reverse` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 156-187

````cpp
__isl_export
__isl_give isl_aff *isl_aff_gist(__isl_take isl_aff *aff,
	__isl_take isl_set *context);
__isl_export
__isl_give isl_aff *isl_aff_gist_params(__isl_take isl_aff *aff,
	__isl_take isl_set *context);

__isl_export
__isl_give isl_val *isl_aff_eval(__isl_take isl_aff *aff,
	__isl_take isl_point *pnt);

__isl_give isl_aff *isl_aff_pullback_aff(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_overload
__isl_give isl_aff *isl_aff_pullback_multi_aff(__isl_take isl_aff *aff,
	__isl_take isl_multi_aff *ma);

__isl_give isl_basic_set *isl_aff_zero_basic_set(__isl_take isl_aff *aff);
__isl_give isl_basic_set *isl_aff_neg_basic_set(__isl_take isl_aff *aff);

__isl_give isl_basic_set *isl_aff_eq_basic_set(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_export
__isl_give isl_set *isl_aff_eq_set(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_export
__isl_give isl_set *isl_aff_ne_set(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_give isl_basic_set *isl_aff_le_basic_set(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_export
__isl_give isl_set *isl_aff_le_set(__isl_take isl_aff *aff1,
````
- **EN**: This block declares or defines routines around `isl_aff_gist`, `isl_aff_gist_params`, `isl_aff_eval`, `isl_aff_pullback_aff` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_aff_gist`, `isl_aff_gist_params`, `isl_aff_eval`, `isl_aff_pullback_aff` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 188-215

````cpp
	__isl_take isl_aff *aff2);
__isl_give isl_basic_set *isl_aff_lt_basic_set(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_export
__isl_give isl_set *isl_aff_lt_set(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_give isl_basic_set *isl_aff_ge_basic_set(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_export
__isl_give isl_set *isl_aff_ge_set(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_give isl_basic_set *isl_aff_gt_basic_set(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);
__isl_export
__isl_give isl_set *isl_aff_gt_set(__isl_take isl_aff *aff1,
	__isl_take isl_aff *aff2);

__isl_overload
__isl_give isl_basic_set *isl_aff_bind_id(__isl_take isl_aff *aff,
	__isl_take isl_id *id);

__isl_constructor
__isl_give isl_aff *isl_aff_read_from_str(isl_ctx *ctx, const char *str);
__isl_give char *isl_aff_to_str(__isl_keep isl_aff *aff);
__isl_give isl_printer *isl_printer_print_aff(__isl_take isl_printer *p,
	__isl_keep isl_aff *aff);
void isl_aff_dump(__isl_keep isl_aff *aff);

````
- **EN**: This block declares or defines routines around `isl_aff_lt_basic_set`, `isl_aff_lt_set`, `isl_aff_ge_basic_set`, `isl_aff_ge_set` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_aff_lt_basic_set`, `isl_aff_lt_set`, `isl_aff_ge_basic_set`, `isl_aff_ge_set` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 216-245

````cpp
isl_ctx *isl_pw_aff_get_ctx(__isl_keep isl_pw_aff *pwaff);
uint32_t isl_pw_aff_get_hash(__isl_keep isl_pw_aff *pa);
__isl_give isl_space *isl_pw_aff_get_domain_space(__isl_keep isl_pw_aff *pwaff);
__isl_export
__isl_give isl_space *isl_pw_aff_get_space(__isl_keep isl_pw_aff *pwaff);

__isl_constructor
__isl_give isl_pw_aff *isl_pw_aff_from_aff(__isl_take isl_aff *aff);
__isl_give isl_pw_aff *isl_pw_aff_empty(__isl_take isl_space *space);
__isl_give isl_pw_aff *isl_pw_aff_alloc(__isl_take isl_set *set,
	__isl_take isl_aff *aff);
__isl_give isl_pw_aff *isl_pw_aff_zero_on_domain(
	__isl_take isl_local_space *ls);
__isl_give isl_pw_aff *isl_pw_aff_var_on_domain(__isl_take isl_local_space *ls,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_pw_aff *isl_pw_aff_nan_on_domain_space(
	__isl_take isl_space *space);
__isl_give isl_pw_aff *isl_pw_aff_nan_on_domain(__isl_take isl_local_space *ls);
__isl_give isl_pw_aff *isl_pw_aff_val_on_domain(__isl_take isl_set *domain,
	__isl_take isl_val *v);
__isl_overload
__isl_give isl_pw_aff *isl_set_pw_aff_on_domain_val(__isl_take isl_set *domain,
	__isl_take isl_val *v);
__isl_overload
__isl_give isl_pw_aff *isl_pw_aff_param_on_domain_id(
	__isl_take isl_set *domain, __isl_take isl_id *id);
__isl_overload
__isl_give isl_pw_aff *isl_set_param_pw_aff_on_domain_id(
	__isl_take isl_set *domain, __isl_take isl_id *id);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_aff_get_ctx`, `isl_pw_aff_get_hash`, `isl_pw_aff_get_domain_space`, `isl_pw_aff_get_space` (+11 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_aff_get_ctx`, `isl_pw_aff_get_hash`, `isl_pw_aff_get_domain_space`, `isl_pw_aff_get_space` (+11 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 246-278

````cpp
__isl_export
__isl_give isl_pw_aff *isl_set_indicator_function(__isl_take isl_set *set);

const char *isl_pw_aff_get_dim_name(__isl_keep isl_pw_aff *pa,
	enum isl_dim_type type, unsigned pos);
isl_bool isl_pw_aff_has_dim_id(__isl_keep isl_pw_aff *pa,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_id *isl_pw_aff_get_dim_id(__isl_keep isl_pw_aff *pa,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_pw_aff *isl_pw_aff_set_dim_id(__isl_take isl_pw_aff *pma,
	enum isl_dim_type type, unsigned pos, __isl_take isl_id *id);

int isl_pw_aff_find_dim_by_name(__isl_keep isl_pw_aff *pa,
	enum isl_dim_type type, const char *name);

isl_bool isl_pw_aff_is_empty(__isl_keep isl_pw_aff *pwaff);
isl_bool isl_pw_aff_involves_nan(__isl_keep isl_pw_aff *pa);
int isl_pw_aff_plain_cmp(__isl_keep isl_pw_aff *pa1,
	__isl_keep isl_pw_aff *pa2);
__isl_export
isl_bool isl_pw_aff_plain_is_equal(__isl_keep isl_pw_aff *pwaff1,
	__isl_keep isl_pw_aff *pwaff2);
isl_bool isl_pw_aff_is_equal(__isl_keep isl_pw_aff *pa1,
	__isl_keep isl_pw_aff *pa2);

__isl_give isl_pw_aff *isl_pw_aff_union_min(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);
__isl_give isl_pw_aff *isl_pw_aff_union_max(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_union_add(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_set_indicator_function`, `isl_pw_aff_get_dim_name`, `isl_pw_aff_has_dim_id`, `isl_pw_aff_get_dim_id` (+10 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_set_indicator_function`, `isl_pw_aff_get_dim_name`, `isl_pw_aff_has_dim_id`, `isl_pw_aff_get_dim_id` (+10 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 279-311

````cpp
__isl_give isl_pw_aff *isl_pw_aff_copy(__isl_keep isl_pw_aff *pwaff);
__isl_null isl_pw_aff *isl_pw_aff_free(__isl_take isl_pw_aff *pwaff);

isl_size isl_pw_aff_dim(__isl_keep isl_pw_aff *pwaff, enum isl_dim_type type);
isl_bool isl_pw_aff_involves_param_id(__isl_keep isl_pw_aff *pa,
	__isl_keep isl_id *id);
isl_bool isl_pw_aff_involves_dims(__isl_keep isl_pw_aff *pwaff,
	enum isl_dim_type type, unsigned first, unsigned n);

isl_bool isl_pw_aff_is_cst(__isl_keep isl_pw_aff *pwaff);

__isl_export
__isl_give isl_pw_aff *isl_pw_aff_insert_domain(__isl_take isl_pw_aff *pa,
	__isl_take isl_space *domain);
__isl_give isl_pw_aff *isl_pw_aff_project_domain_on_params(
	__isl_take isl_pw_aff *pa);

__isl_give isl_pw_aff *isl_pw_aff_align_params(__isl_take isl_pw_aff *pwaff,
	__isl_take isl_space *model);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_drop_unused_params(
	__isl_take isl_pw_aff *pa);

isl_bool isl_pw_aff_has_tuple_id(__isl_keep isl_pw_aff *pa,
	enum isl_dim_type type);
__isl_give isl_id *isl_pw_aff_get_tuple_id(__isl_keep isl_pw_aff *pa,
	enum isl_dim_type type);
__isl_give isl_pw_aff *isl_pw_aff_set_tuple_id(__isl_take isl_pw_aff *pwaff,
	enum isl_dim_type type, __isl_take isl_id *id);
__isl_give isl_pw_aff *isl_pw_aff_reset_tuple_id(__isl_take isl_pw_aff *pa,
	enum isl_dim_type type);
__isl_give isl_pw_aff *isl_pw_aff_reset_user(__isl_take isl_pw_aff *pa);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_aff_copy`, `isl_pw_aff_free`, `isl_pw_aff_dim`, `isl_pw_aff_involves_param_id` (+11 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_aff_copy`, `isl_pw_aff_free`, `isl_pw_aff_dim`, `isl_pw_aff_involves_param_id` (+11 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 312-343

````cpp
__isl_export
__isl_give isl_set *isl_pw_aff_params(__isl_take isl_pw_aff *pwa);
__isl_export
__isl_give isl_set *isl_pw_aff_domain(__isl_take isl_pw_aff *pwaff);
__isl_give isl_pw_aff *isl_pw_aff_from_range(__isl_take isl_pw_aff *pwa);

__isl_export
__isl_give isl_pw_aff *isl_pw_aff_min(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_max(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_mul(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_div(__isl_take isl_pw_aff *pa1,
	__isl_take isl_pw_aff *pa2);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_add(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_sub(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_neg(__isl_take isl_pw_aff *pwaff);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_ceil(__isl_take isl_pw_aff *pwaff);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_floor(__isl_take isl_pw_aff *pwaff);
__isl_overload
__isl_give isl_pw_aff *isl_pw_aff_mod_val(__isl_take isl_pw_aff *pa,
````
- **EN**: This block declares or defines routines around `isl_pw_aff_params`, `isl_pw_aff_domain`, `isl_pw_aff_from_range`, `isl_pw_aff_min` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_aff_params`, `isl_pw_aff_domain`, `isl_pw_aff_from_range`, `isl_pw_aff_min` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 344-369

````cpp
	__isl_take isl_val *mod);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_tdiv_q(__isl_take isl_pw_aff *pa1,
	__isl_take isl_pw_aff *pa2);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_tdiv_r(__isl_take isl_pw_aff *pa1,
	__isl_take isl_pw_aff *pa2);

__isl_export
__isl_give isl_pw_aff *isl_pw_aff_intersect_params(__isl_take isl_pw_aff *pa,
	__isl_take isl_set *set);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_intersect_domain(__isl_take isl_pw_aff *pa,
	__isl_take isl_set *set);
__isl_give isl_pw_aff *isl_pw_aff_intersect_domain_wrapped_domain(
	__isl_take isl_pw_aff *pa, __isl_take isl_set *set);
__isl_give isl_pw_aff *isl_pw_aff_intersect_domain_wrapped_range(
	__isl_take isl_pw_aff *pa, __isl_take isl_set *set);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_subtract_domain(__isl_take isl_pw_aff *pa,
	__isl_take isl_set *set);

__isl_export
__isl_give isl_pw_aff *isl_pw_aff_cond(__isl_take isl_pw_aff *cond,
	__isl_take isl_pw_aff *pwaff_true, __isl_take isl_pw_aff *pwaff_false);

````
- **EN**: This block declares or defines routines around `isl_pw_aff_tdiv_q`, `isl_pw_aff_tdiv_r`, `isl_pw_aff_intersect_params`, `isl_pw_aff_intersect_domain` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_aff_tdiv_q`, `isl_pw_aff_tdiv_r`, `isl_pw_aff_intersect_params`, `isl_pw_aff_intersect_domain` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 370-400

````cpp
__isl_overload
__isl_give isl_pw_aff *isl_pw_aff_add_constant_val(__isl_take isl_pw_aff *pa,
	__isl_take isl_val *v);
__isl_overload
__isl_give isl_pw_aff *isl_pw_aff_scale_val(__isl_take isl_pw_aff *pa,
	__isl_take isl_val *v);
__isl_overload
__isl_give isl_pw_aff *isl_pw_aff_scale_down_val(__isl_take isl_pw_aff *pa,
	__isl_take isl_val *f);

__isl_export
__isl_give isl_pw_aff *isl_pw_aff_domain_reverse(__isl_take isl_pw_aff *pa);
__isl_give isl_pw_aff *isl_pw_aff_insert_dims(__isl_take isl_pw_aff *pwaff,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_pw_aff *isl_pw_aff_add_dims(__isl_take isl_pw_aff *pwaff,
	enum isl_dim_type type, unsigned n);
__isl_give isl_pw_aff *isl_pw_aff_move_dims(__isl_take isl_pw_aff *pa,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n);
__isl_give isl_pw_aff *isl_pw_aff_drop_dims(__isl_take isl_pw_aff *pwaff,
	enum isl_dim_type type, unsigned first, unsigned n);

__isl_export
__isl_give isl_pw_aff *isl_pw_aff_coalesce(__isl_take isl_pw_aff *pa);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_gist(__isl_take isl_pw_aff *pwaff,
	__isl_take isl_set *context);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_gist_params(__isl_take isl_pw_aff *pwaff,
	__isl_take isl_set *context);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_aff_add_constant_val`, `isl_pw_aff_scale_val`, `isl_pw_aff_scale_down_val`, `isl_pw_aff_domain_reverse` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_aff_add_constant_val`, `isl_pw_aff_scale_val`, `isl_pw_aff_scale_down_val`, `isl_pw_aff_domain_reverse` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 401-426

````cpp
__isl_export
__isl_give isl_val *isl_pw_aff_eval(__isl_take isl_pw_aff *pa,
	__isl_take isl_point *pnt);

__isl_overload
__isl_give isl_pw_aff *isl_pw_aff_pullback_multi_aff(
	__isl_take isl_pw_aff *pa, __isl_take isl_multi_aff *ma);
__isl_overload
__isl_give isl_pw_aff *isl_pw_aff_pullback_pw_multi_aff(
	__isl_take isl_pw_aff *pa, __isl_take isl_pw_multi_aff *pma);
__isl_overload
__isl_give isl_pw_aff *isl_pw_aff_pullback_multi_pw_aff(
	__isl_take isl_pw_aff *pa, __isl_take isl_multi_pw_aff *mpa);

isl_size isl_pw_aff_n_piece(__isl_keep isl_pw_aff *pwaff);
isl_stat isl_pw_aff_foreach_piece(__isl_keep isl_pw_aff *pwaff,
	isl_stat (*fn)(__isl_take isl_set *set, __isl_take isl_aff *aff,
		    void *user), void *user);
isl_bool isl_pw_aff_every_piece(__isl_keep isl_pw_aff *pa,
	isl_bool (*test)(__isl_keep isl_set *set, __isl_keep isl_aff *aff,
		void *user), void *user);
__isl_export
isl_bool isl_pw_aff_isa_aff(__isl_keep isl_pw_aff *pa);
__isl_export
__isl_give isl_aff *isl_pw_aff_as_aff(__isl_take isl_pw_aff *pa);

````
- **EN**: This block declares or defines routines around `isl_pw_aff_eval`, `isl_pw_aff_pullback_multi_aff`, `isl_pw_aff_pullback_pw_multi_aff`, `isl_pw_aff_pullback_multi_pw_aff` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_aff_eval`, `isl_pw_aff_pullback_multi_aff`, `isl_pw_aff_pullback_pw_multi_aff`, `isl_pw_aff_pullback_multi_pw_aff` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 427-455

````cpp
__isl_export
__isl_give isl_map *isl_pw_aff_as_map(__isl_take isl_pw_aff *pa);
__isl_give isl_set *isl_set_from_pw_aff(__isl_take isl_pw_aff *pwaff);
__isl_give isl_map *isl_map_from_pw_aff(__isl_take isl_pw_aff *pwaff);

__isl_give isl_set *isl_pw_aff_pos_set(__isl_take isl_pw_aff *pa);
__isl_give isl_set *isl_pw_aff_nonneg_set(__isl_take isl_pw_aff *pwaff);
__isl_give isl_set *isl_pw_aff_zero_set(__isl_take isl_pw_aff *pwaff);
__isl_give isl_set *isl_pw_aff_non_zero_set(__isl_take isl_pw_aff *pwaff);

__isl_export
__isl_give isl_set *isl_pw_aff_eq_set(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);
__isl_export
__isl_give isl_set *isl_pw_aff_ne_set(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);
__isl_export
__isl_give isl_set *isl_pw_aff_le_set(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);
__isl_export
__isl_give isl_set *isl_pw_aff_lt_set(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);
__isl_export
__isl_give isl_set *isl_pw_aff_ge_set(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);
__isl_export
__isl_give isl_set *isl_pw_aff_gt_set(__isl_take isl_pw_aff *pwaff1,
	__isl_take isl_pw_aff *pwaff2);

````
- **EN**: This block declares or defines routines around `isl_pw_aff_as_map`, `isl_set_from_pw_aff`, `isl_map_from_pw_aff`, `isl_pw_aff_pos_set` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_aff_as_map`, `isl_set_from_pw_aff`, `isl_map_from_pw_aff`, `isl_pw_aff_pos_set` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 456-483

````cpp
__isl_give isl_map *isl_pw_aff_eq_map(__isl_take isl_pw_aff *pa1,
	__isl_take isl_pw_aff *pa2);
__isl_give isl_map *isl_pw_aff_le_map(__isl_take isl_pw_aff *pa1,
	__isl_take isl_pw_aff *pa2);
__isl_give isl_map *isl_pw_aff_lt_map(__isl_take isl_pw_aff *pa1,
	__isl_take isl_pw_aff *pa2);
__isl_give isl_map *isl_pw_aff_ge_map(__isl_take isl_pw_aff *pa1,
	__isl_take isl_pw_aff *pa2);
__isl_give isl_map *isl_pw_aff_gt_map(__isl_take isl_pw_aff *pa1,
	__isl_take isl_pw_aff *pa2);

__isl_export
__isl_give isl_pw_aff *isl_pw_aff_bind_domain(__isl_take isl_pw_aff *pa,
	__isl_take isl_multi_id *tuple);
__isl_export
__isl_give isl_pw_aff *isl_pw_aff_bind_domain_wrapped_domain(
	__isl_take isl_pw_aff *pa, __isl_take isl_multi_id *tuple);
__isl_overload
__isl_give isl_set *isl_pw_aff_bind_id(__isl_take isl_pw_aff *pa,
	__isl_take isl_id *id);

__isl_constructor
__isl_give isl_pw_aff *isl_pw_aff_read_from_str(isl_ctx *ctx, const char *str);
__isl_give char *isl_pw_aff_to_str(__isl_keep isl_pw_aff *pa);
__isl_give isl_printer *isl_printer_print_pw_aff(__isl_take isl_printer *p,
	__isl_keep isl_pw_aff *pwaff);
void isl_pw_aff_dump(__isl_keep isl_pw_aff *pwaff);

````
- **EN**: This block declares or defines routines around `isl_pw_aff_eq_map`, `isl_pw_aff_le_map`, `isl_pw_aff_lt_map`, `isl_pw_aff_ge_map` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_aff_eq_map`, `isl_pw_aff_le_map`, `isl_pw_aff_lt_map`, `isl_pw_aff_ge_map` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 484-516

````cpp
__isl_give isl_pw_aff *isl_pw_aff_list_min(__isl_take isl_pw_aff_list *list);
__isl_give isl_pw_aff *isl_pw_aff_list_max(__isl_take isl_pw_aff_list *list);

__isl_give isl_set *isl_pw_aff_list_eq_set(__isl_take isl_pw_aff_list *list1,
	__isl_take isl_pw_aff_list *list2);
__isl_give isl_set *isl_pw_aff_list_ne_set(__isl_take isl_pw_aff_list *list1,
	__isl_take isl_pw_aff_list *list2);
__isl_give isl_set *isl_pw_aff_list_le_set(__isl_take isl_pw_aff_list *list1,
	__isl_take isl_pw_aff_list *list2);
__isl_give isl_set *isl_pw_aff_list_lt_set(__isl_take isl_pw_aff_list *list1,
	__isl_take isl_pw_aff_list *list2);
__isl_give isl_set *isl_pw_aff_list_ge_set(__isl_take isl_pw_aff_list *list1,
	__isl_take isl_pw_aff_list *list2);
__isl_give isl_set *isl_pw_aff_list_gt_set(__isl_take isl_pw_aff_list *list1,
	__isl_take isl_pw_aff_list *list2);

ISL_DECLARE_MULTI(aff)
ISL_DECLARE_MULTI_IDENTITY(aff)
ISL_DECLARE_MULTI_CMP(aff)
ISL_DECLARE_MULTI_ARITH(aff)
ISL_DECLARE_MULTI_ADD_CONSTANT(aff)
ISL_DECLARE_MULTI_ZERO(aff)
ISL_DECLARE_MULTI_NAN(aff)
ISL_DECLARE_MULTI_DIMS(aff)
ISL_DECLARE_MULTI_INSERT_DOMAIN(aff)
ISL_DECLARE_MULTI_LOCALS(aff)
ISL_DECLARE_MULTI_DIM_ID(aff)
ISL_DECLARE_MULTI_TUPLE_ID(aff)
ISL_DECLARE_MULTI_WITH_DOMAIN(aff)
ISL_DECLARE_MULTI_BIND_DOMAIN(aff)
ISL_DECLARE_MULTI_UNBIND_PARAMS(aff)
ISL_DECLARE_MULTI_DOMAIN_REVERSE(aff)

````
- **EN**: This block declares or defines routines around `isl_pw_aff_list_min`, `isl_pw_aff_list_max`, `isl_pw_aff_list_eq_set`, `isl_pw_aff_list_ne_set` (+20 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_aff_list_min`, `isl_pw_aff_list_max`, `isl_pw_aff_list_eq_set`, `isl_pw_aff_list_ne_set` (+20 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 517-545

````cpp
__isl_constructor
__isl_give isl_multi_aff *isl_multi_aff_from_aff(__isl_take isl_aff *aff);
__isl_export
__isl_give isl_multi_aff *isl_multi_aff_domain_map(__isl_take isl_space *space);
__isl_export
__isl_give isl_multi_aff *isl_space_domain_map_multi_aff(
	__isl_take isl_space *space);
__isl_export
__isl_give isl_multi_aff *isl_multi_aff_range_map(__isl_take isl_space *space);
__isl_export
__isl_give isl_multi_aff *isl_space_range_map_multi_aff(
	__isl_take isl_space *space);
__isl_give isl_multi_aff *isl_multi_aff_project_out_map(
	__isl_take isl_space *space, enum isl_dim_type type,
	unsigned first, unsigned n);

__isl_overload
__isl_give isl_multi_aff *isl_multi_aff_multi_val_on_domain_space(
	__isl_take isl_space *space, __isl_take isl_multi_val *mv);
__isl_overload
__isl_give isl_multi_aff *isl_space_multi_aff_on_domain_multi_val(
	__isl_take isl_space *space, __isl_take isl_multi_val *mv);
__isl_give isl_multi_aff *isl_multi_aff_multi_val_on_space(
	__isl_take isl_space *space, __isl_take isl_multi_val *mv);

__isl_export
__isl_give isl_multi_val *isl_multi_aff_get_constant_multi_val(
	__isl_keep isl_multi_aff *ma);

````
- **EN**: This block declares or defines routines around `isl_multi_aff_from_aff`, `isl_multi_aff_domain_map`, `isl_space_domain_map_multi_aff`, `isl_multi_aff_range_map` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_multi_aff_from_aff`, `isl_multi_aff_domain_map`, `isl_space_domain_map_multi_aff`, `isl_multi_aff_range_map` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 546-575

````cpp
__isl_export
__isl_give isl_multi_aff *isl_multi_aff_floor(__isl_take isl_multi_aff *ma);

__isl_export
__isl_give isl_multi_aff *isl_multi_aff_gist_params(
	__isl_take isl_multi_aff *maff, __isl_take isl_set *context);
__isl_export
__isl_give isl_multi_aff *isl_multi_aff_gist(__isl_take isl_multi_aff *maff,
	__isl_take isl_set *context);

__isl_give isl_multi_aff *isl_multi_aff_lift(__isl_take isl_multi_aff *maff,
	__isl_give isl_local_space **ls);

__isl_overload
__isl_give isl_multi_aff *isl_multi_aff_pullback_multi_aff(
	__isl_take isl_multi_aff *ma1, __isl_take isl_multi_aff *ma2);

__isl_give isl_multi_aff *isl_multi_aff_move_dims(__isl_take isl_multi_aff *ma,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n);

__isl_give isl_set *isl_multi_aff_lex_lt_set(__isl_take isl_multi_aff *ma1,
	__isl_take isl_multi_aff *ma2);
__isl_give isl_set *isl_multi_aff_lex_le_set(__isl_take isl_multi_aff *ma1,
	__isl_take isl_multi_aff *ma2);
__isl_give isl_set *isl_multi_aff_lex_gt_set(__isl_take isl_multi_aff *ma1,
	__isl_take isl_multi_aff *ma2);
__isl_give isl_set *isl_multi_aff_lex_ge_set(__isl_take isl_multi_aff *ma1,
	__isl_take isl_multi_aff *ma2);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_multi_aff_floor`, `isl_multi_aff_gist_params`, `isl_multi_aff_gist`, `isl_multi_aff_lift` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_multi_aff_floor`, `isl_multi_aff_gist_params`, `isl_multi_aff_gist`, `isl_multi_aff_lift` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 576-605

````cpp
__isl_export
__isl_give isl_basic_set *isl_multi_aff_bind(__isl_take isl_multi_aff *ma,
	__isl_take isl_multi_id *tuple);

__isl_give char *isl_multi_aff_to_str(__isl_keep isl_multi_aff *ma);
__isl_give isl_printer *isl_printer_print_multi_aff(__isl_take isl_printer *p,
	__isl_keep isl_multi_aff *maff);

__isl_constructor
__isl_give isl_multi_aff *isl_multi_aff_read_from_str(isl_ctx *ctx,
		const char *str);
void isl_multi_aff_dump(__isl_keep isl_multi_aff *maff);

ISL_DECLARE_MULTI(pw_aff)
ISL_DECLARE_MULTI_IDENTITY(pw_aff)
ISL_DECLARE_MULTI_ARITH(pw_aff)
ISL_DECLARE_MULTI_MIN_MAX(pw_aff)
ISL_DECLARE_MULTI_ADD_CONSTANT(pw_aff)
ISL_DECLARE_MULTI_ZERO(pw_aff)
ISL_DECLARE_MULTI_NAN(pw_aff)
ISL_DECLARE_MULTI_DIMS(pw_aff)
ISL_DECLARE_MULTI_DIM_ID(pw_aff)
ISL_DECLARE_MULTI_INSERT_DOMAIN(pw_aff)
ISL_DECLARE_MULTI_TUPLE_ID(pw_aff)
ISL_DECLARE_MULTI_WITH_DOMAIN(pw_aff)
ISL_DECLARE_MULTI_BIND_DOMAIN(pw_aff)
ISL_DECLARE_MULTI_PARAM(pw_aff)
ISL_DECLARE_MULTI_UNBIND_PARAMS(pw_aff)
ISL_DECLARE_MULTI_DOMAIN_REVERSE(pw_aff)

````
- **EN**: This block declares or defines routines around `isl_multi_aff_bind`, `isl_multi_aff_to_str`, `isl_printer_print_multi_aff`, `isl_multi_aff_read_from_str` (+17 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_multi_aff_bind`, `isl_multi_aff_to_str`, `isl_printer_print_multi_aff`, `isl_multi_aff_read_from_str` (+17 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 606-637

````cpp
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_zero(__isl_take isl_space *space);
__isl_overload
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_identity_on_domain_space(
	__isl_take isl_space *space);
__isl_export
__isl_give isl_pw_multi_aff *isl_space_identity_pw_multi_aff_on_domain(
	__isl_take isl_space *space);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_identity(
	__isl_take isl_space *space);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_domain_map(
	__isl_take isl_space *space);
__isl_export
__isl_give isl_pw_multi_aff *isl_space_domain_map_pw_multi_aff(
	__isl_take isl_space *space);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_range_map(
	__isl_take isl_space *space);
__isl_export
__isl_give isl_pw_multi_aff *isl_space_range_map_pw_multi_aff(
	__isl_take isl_space *space);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_project_out_map(
	__isl_take isl_space *space, enum isl_dim_type type,
	unsigned first, unsigned n);
__isl_export
__isl_give isl_pw_multi_aff *isl_multi_aff_to_pw_multi_aff(
	__isl_take isl_multi_aff *ma);
__isl_constructor
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_from_multi_aff(
	__isl_take isl_multi_aff *ma);
__isl_constructor
````
- **EN**: This block declares or defines routines around `isl_pw_multi_aff_zero`, `isl_pw_multi_aff_identity_on_domain_space`, `isl_space_identity_pw_multi_aff_on_domain`, `isl_pw_multi_aff_identity` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_multi_aff_zero`, `isl_pw_multi_aff_identity_on_domain_space`, `isl_space_identity_pw_multi_aff_on_domain`, `isl_pw_multi_aff_identity` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 638-663

````cpp
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_from_pw_aff(
	__isl_take isl_pw_aff *pa);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_alloc(__isl_take isl_set *set,
	__isl_take isl_multi_aff *maff);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_copy(
	__isl_keep isl_pw_multi_aff *pma);
__isl_null isl_pw_multi_aff *isl_pw_multi_aff_free(
	__isl_take isl_pw_multi_aff *pma);

isl_size isl_pw_multi_aff_dim(__isl_keep isl_pw_multi_aff *pma,
	enum isl_dim_type type);
__isl_export
isl_bool isl_pw_multi_aff_involves_locals(__isl_keep isl_pw_multi_aff *pma);
isl_bool isl_pw_multi_aff_involves_param_id(__isl_keep isl_pw_multi_aff *pma,
	__isl_keep isl_id *id);
isl_bool isl_pw_multi_aff_involves_dims(__isl_keep isl_pw_multi_aff *pma,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_export
__isl_give isl_pw_aff *isl_pw_multi_aff_get_at(
	__isl_keep isl_pw_multi_aff *pma, int pos);
__isl_give isl_pw_aff *isl_pw_multi_aff_get_pw_aff(
	__isl_keep isl_pw_multi_aff *pma, int pos);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_set_pw_aff(
	__isl_take isl_pw_multi_aff *pma, unsigned pos,
	__isl_take isl_pw_aff *pa);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_multi_aff_from_pw_aff`, `isl_pw_multi_aff_alloc`, `isl_pw_multi_aff_copy`, `isl_pw_multi_aff_free` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_multi_aff_from_pw_aff`, `isl_pw_multi_aff_alloc`, `isl_pw_multi_aff_copy`, `isl_pw_multi_aff_free` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 664-693

````cpp
isl_ctx *isl_pw_multi_aff_get_ctx(__isl_keep isl_pw_multi_aff *pma);
__isl_give isl_space *isl_pw_multi_aff_get_domain_space(
	__isl_keep isl_pw_multi_aff *pma);
__isl_export
__isl_give isl_space *isl_pw_multi_aff_get_space(
	__isl_keep isl_pw_multi_aff *pma);
isl_bool isl_pw_multi_aff_has_tuple_name(__isl_keep isl_pw_multi_aff *pma,
	enum isl_dim_type type);
const char *isl_pw_multi_aff_get_tuple_name(__isl_keep isl_pw_multi_aff *pma,
	enum isl_dim_type type);
__isl_export
__isl_give isl_id *isl_pw_multi_aff_get_range_tuple_id(
	__isl_keep isl_pw_multi_aff *pma);
__isl_give isl_id *isl_pw_multi_aff_get_tuple_id(
	__isl_keep isl_pw_multi_aff *pma, enum isl_dim_type type);
__isl_export
isl_bool isl_pw_multi_aff_has_range_tuple_id(__isl_keep isl_pw_multi_aff *pma);
isl_bool isl_pw_multi_aff_has_tuple_id(__isl_keep isl_pw_multi_aff *pma,
	enum isl_dim_type type);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_set_tuple_id(
	__isl_take isl_pw_multi_aff *pma,
	enum isl_dim_type type, __isl_take isl_id *id);
__isl_overload
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_set_range_tuple_id(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_id *id);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_reset_tuple_id(
	__isl_take isl_pw_multi_aff *pma, enum isl_dim_type type);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_reset_user(
	__isl_take isl_pw_multi_aff *pma);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_multi_aff_get_ctx`, `isl_pw_multi_aff_get_domain_space`, `isl_pw_multi_aff_get_space`, `isl_pw_multi_aff_has_tuple_name` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_multi_aff_get_ctx`, `isl_pw_multi_aff_get_domain_space`, `isl_pw_multi_aff_get_space`, `isl_pw_multi_aff_has_tuple_name` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 694-723

````cpp
int isl_pw_multi_aff_find_dim_by_name(__isl_keep isl_pw_multi_aff *pma,
	enum isl_dim_type type, const char *name);

__isl_give isl_pw_multi_aff *isl_pw_multi_aff_drop_dims(
	__isl_take isl_pw_multi_aff *pma,
	enum isl_dim_type type, unsigned first, unsigned n);

__isl_export
__isl_give isl_set *isl_pw_multi_aff_domain(__isl_take isl_pw_multi_aff *pma);

__isl_give isl_pw_multi_aff *isl_pw_multi_aff_empty(__isl_take isl_space *space);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_from_domain(
	__isl_take isl_set *set);

__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_multi_val_on_domain(
	__isl_take isl_set *domain, __isl_take isl_multi_val *mv);
__isl_overload
__isl_give isl_pw_multi_aff *isl_set_pw_multi_aff_on_domain_multi_val(
	__isl_take isl_set *domain, __isl_take isl_multi_val *mv);

const char *isl_pw_multi_aff_get_dim_name(__isl_keep isl_pw_multi_aff *pma,
	enum isl_dim_type type, unsigned pos);
__isl_give isl_id *isl_pw_multi_aff_get_dim_id(
	__isl_keep isl_pw_multi_aff *pma, enum isl_dim_type type,
	unsigned pos);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_set_dim_id(
	__isl_take isl_pw_multi_aff *pma,
	enum isl_dim_type type, unsigned pos, __isl_take isl_id *id);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_pw_multi_aff_find_dim_by_name`, `isl_pw_multi_aff_drop_dims`, `isl_pw_multi_aff_domain`, `isl_pw_multi_aff_empty` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_pw_multi_aff_find_dim_by_name`, `isl_pw_multi_aff_drop_dims`, `isl_pw_multi_aff_domain`, `isl_pw_multi_aff_empty` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 724-752

````cpp
isl_bool isl_pw_multi_aff_involves_nan(__isl_keep isl_pw_multi_aff *pma);
__isl_export
isl_bool isl_pw_multi_aff_plain_is_equal(__isl_keep isl_pw_multi_aff *pma1,
	__isl_keep isl_pw_multi_aff *pma2);
isl_bool isl_pw_multi_aff_is_equal(__isl_keep isl_pw_multi_aff *pma1,
	__isl_keep isl_pw_multi_aff *pma2);

__isl_give isl_pw_multi_aff *isl_pw_multi_aff_fix_si(
	__isl_take isl_pw_multi_aff *pma, enum isl_dim_type type,
	unsigned pos, int value);

__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_union_add(
	__isl_take isl_pw_multi_aff *pma1, __isl_take isl_pw_multi_aff *pma2);

__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_domain_reverse(
	__isl_take isl_pw_multi_aff *pma);

__isl_give isl_pw_multi_aff *isl_pw_multi_aff_neg(
	__isl_take isl_pw_multi_aff *pma);

__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_add(
	__isl_take isl_pw_multi_aff *pma1, __isl_take isl_pw_multi_aff *pma2);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_sub(
	__isl_take isl_pw_multi_aff *pma1, __isl_take isl_pw_multi_aff *pma2);

````
- **EN**: This block declares or defines routines around `isl_pw_multi_aff_involves_nan`, `isl_pw_multi_aff_plain_is_equal`, `isl_pw_multi_aff_is_equal`, `isl_pw_multi_aff_fix_si` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_multi_aff_involves_nan`, `isl_pw_multi_aff_plain_is_equal`, `isl_pw_multi_aff_is_equal`, `isl_pw_multi_aff_fix_si` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 753-778

````cpp
__isl_overload
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_add_constant_val(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_val *v);
__isl_overload
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_add_constant_multi_val(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_multi_val *mv);
__isl_overload
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_scale_val(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_val *v);
__isl_overload
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_scale_down_val(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_val *v);
__isl_overload
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_scale_multi_val(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_multi_val *mv);
__isl_overload
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_scale_down_multi_val(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_multi_val *mv);

__isl_give isl_pw_multi_aff *isl_pw_multi_aff_union_lexmin(
	__isl_take isl_pw_multi_aff *pma1,
	__isl_take isl_pw_multi_aff *pma2);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_union_lexmax(
	__isl_take isl_pw_multi_aff *pma1,
	__isl_take isl_pw_multi_aff *pma2);

````
- **EN**: This block declares or defines routines around `isl_pw_multi_aff_add_constant_val`, `isl_pw_multi_aff_add_constant_multi_val`, `isl_pw_multi_aff_scale_val`, `isl_pw_multi_aff_scale_down_val` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_multi_aff_add_constant_val`, `isl_pw_multi_aff_add_constant_multi_val`, `isl_pw_multi_aff_scale_val`, `isl_pw_multi_aff_scale_down_val` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 779-811

````cpp
__isl_give isl_multi_aff *isl_multi_aff_flatten_domain(
	__isl_take isl_multi_aff *ma);

__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_range_product(
	__isl_take isl_pw_multi_aff *pma1, __isl_take isl_pw_multi_aff *pma2);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_flat_range_product(
	__isl_take isl_pw_multi_aff *pma1, __isl_take isl_pw_multi_aff *pma2);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_product(
	__isl_take isl_pw_multi_aff *pma1, __isl_take isl_pw_multi_aff *pma2);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_range_factor_domain(
	__isl_take isl_pw_multi_aff *pma);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_range_factor_range(
	__isl_take isl_pw_multi_aff *pma);

__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_intersect_params(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_set *set);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_intersect_domain(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_set *set);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_intersect_domain_wrapped_domain(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_set *set);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_intersect_domain_wrapped_range(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_set *set);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_subtract_domain(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_set *set);

````
- **EN**: This block declares or defines routines around `isl_multi_aff_flatten_domain`, `isl_pw_multi_aff_range_product`, `isl_pw_multi_aff_flat_range_product`, `isl_pw_multi_aff_product` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_multi_aff_flatten_domain`, `isl_pw_multi_aff_range_product`, `isl_pw_multi_aff_flat_range_product`, `isl_pw_multi_aff_product` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 812-844

````cpp
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_insert_domain(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_space *domain);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_project_domain_on_params(
	__isl_take isl_pw_multi_aff *pma);

__isl_give isl_pw_multi_aff *isl_pw_multi_aff_align_params(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_space *model);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_drop_unused_params(
	__isl_take isl_pw_multi_aff *pma);

__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_coalesce(
	__isl_take isl_pw_multi_aff *pma);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_gist_params(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_set *set);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_gist(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_set *set);

__isl_overload
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_pullback_multi_aff(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_multi_aff *ma);
__isl_overload
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_pullback_pw_multi_aff(
	__isl_take isl_pw_multi_aff *pma1, __isl_take isl_pw_multi_aff *pma2);
__isl_overload
__isl_give isl_pw_multi_aff *
isl_pw_multi_aff_preimage_domain_wrapped_domain_pw_multi_aff(
	__isl_take isl_pw_multi_aff *pma1, __isl_take isl_pw_multi_aff *pma2);

````
- **EN**: This block declares or defines routines around `isl_pw_multi_aff_insert_domain`, `isl_pw_multi_aff_project_domain_on_params`, `isl_pw_multi_aff_align_params`, `isl_pw_multi_aff_drop_unused_params` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_multi_aff_insert_domain`, `isl_pw_multi_aff_project_domain_on_params`, `isl_pw_multi_aff_align_params`, `isl_pw_multi_aff_drop_unused_params` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 845-870

````cpp
__isl_export
isl_size isl_pw_multi_aff_n_piece(__isl_keep isl_pw_multi_aff *pma);
__isl_export
isl_stat isl_pw_multi_aff_foreach_piece(__isl_keep isl_pw_multi_aff *pma,
	isl_stat (*fn)(__isl_take isl_set *set, __isl_take isl_multi_aff *maff,
		    void *user), void *user);
isl_bool isl_pw_multi_aff_every_piece(__isl_keep isl_pw_multi_aff *pma,
	isl_bool (*test)(__isl_keep isl_set *set, __isl_keep isl_multi_aff *ma,
		void *user), void *user);
__isl_export
isl_bool isl_pw_multi_aff_isa_multi_aff(__isl_keep isl_pw_multi_aff *pma);
__isl_export
__isl_give isl_multi_aff *isl_pw_multi_aff_as_multi_aff(
	__isl_take isl_pw_multi_aff *pma);

__isl_export
__isl_give isl_map *isl_pw_multi_aff_as_map(__isl_take isl_pw_multi_aff *pma);
__isl_give isl_map *isl_map_from_pw_multi_aff(__isl_take isl_pw_multi_aff *pma);
__isl_export
__isl_give isl_set *isl_pw_multi_aff_as_set(__isl_take isl_pw_multi_aff *pma);
__isl_give isl_set *isl_set_from_pw_multi_aff(__isl_take isl_pw_multi_aff *pma);

__isl_give char *isl_pw_multi_aff_to_str(__isl_keep isl_pw_multi_aff *pma);
__isl_give isl_printer *isl_printer_print_pw_multi_aff(__isl_take isl_printer *p,
	__isl_keep isl_pw_multi_aff *pma);

````
- **EN**: This block declares or defines routines around `isl_pw_multi_aff_n_piece`, `isl_pw_multi_aff_foreach_piece`, `isl_stat`, `isl_pw_multi_aff_every_piece` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_multi_aff_n_piece`, `isl_pw_multi_aff_foreach_piece`, `isl_stat`, `isl_pw_multi_aff_every_piece` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 871-902

````cpp
__isl_export
__isl_give isl_pw_multi_aff *isl_set_as_pw_multi_aff(__isl_take isl_set *set);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_from_set(__isl_take isl_set *set);
__isl_export
__isl_give isl_pw_multi_aff *isl_map_as_pw_multi_aff(__isl_take isl_map *map);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_from_map(__isl_take isl_map *map);

__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_bind_domain(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_multi_id *tuple);
__isl_export
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_bind_domain_wrapped_domain(
	__isl_take isl_pw_multi_aff *pma, __isl_take isl_multi_id *tuple);

__isl_constructor
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_read_from_str(isl_ctx *ctx,
	const char *str);
void isl_pw_multi_aff_dump(__isl_keep isl_pw_multi_aff *pma);


__isl_overload
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_empty_ctx(
	isl_ctx *ctx);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_empty_space(
	__isl_take isl_space *space);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_empty(
	__isl_take isl_space *space);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_from_aff(
	__isl_take isl_aff *aff);
__isl_constructor
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_from_multi_aff(
	__isl_take isl_multi_aff *ma);
````
- **EN**: This block declares or defines routines around `isl_set_as_pw_multi_aff`, `isl_pw_multi_aff_from_set`, `isl_map_as_pw_multi_aff`, `isl_pw_multi_aff_from_map` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_set_as_pw_multi_aff`, `isl_pw_multi_aff_from_set`, `isl_map_as_pw_multi_aff`, `isl_pw_multi_aff_from_map` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 903-929

````cpp
__isl_export
__isl_give isl_union_pw_multi_aff *isl_pw_multi_aff_to_union_pw_multi_aff(
	__isl_take isl_pw_multi_aff *pma);
__isl_constructor
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_from_pw_multi_aff(
	__isl_take isl_pw_multi_aff *pma);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_from_domain(
	__isl_take isl_union_set *uset);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_multi_val_on_domain(
	__isl_take isl_union_set *domain, __isl_take isl_multi_val *mv);
__isl_give isl_union_pw_aff *isl_union_pw_aff_param_on_domain_id(
	__isl_take isl_union_set *domain, __isl_take isl_id *id);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_copy(
	__isl_keep isl_union_pw_multi_aff *upma);
__isl_null isl_union_pw_multi_aff *isl_union_pw_multi_aff_free(
	__isl_take isl_union_pw_multi_aff *upma);

__isl_give isl_union_pw_multi_aff *isl_union_set_identity_union_pw_multi_aff(
	__isl_take isl_union_set *uset);

__isl_give isl_union_pw_aff *isl_union_pw_multi_aff_get_union_pw_aff(
	__isl_keep isl_union_pw_multi_aff *upma, int pos);

__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_add_pw_multi_aff(
	__isl_take isl_union_pw_multi_aff *upma,
	__isl_take isl_pw_multi_aff *pma);

````
- **EN**: This block declares or defines routines around `isl_pw_multi_aff_to_union_pw_multi_aff`, `isl_union_pw_multi_aff_from_pw_multi_aff`, `isl_union_pw_multi_aff_from_domain`, `isl_union_pw_multi_aff_multi_val_on_domain` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_multi_aff_to_union_pw_multi_aff`, `isl_union_pw_multi_aff_from_pw_multi_aff`, `isl_union_pw_multi_aff_from_domain`, `isl_union_pw_multi_aff_multi_val_on_domain` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 930-964

````cpp
isl_ctx *isl_union_pw_multi_aff_get_ctx(
	__isl_keep isl_union_pw_multi_aff *upma);
__isl_export
__isl_give isl_space *isl_union_pw_multi_aff_get_space(
	__isl_keep isl_union_pw_multi_aff *upma);
__isl_export
__isl_give isl_pw_multi_aff_list *isl_union_pw_multi_aff_get_pw_multi_aff_list(
	__isl_keep isl_union_pw_multi_aff *upma);

isl_size isl_union_pw_multi_aff_dim(__isl_keep isl_union_pw_multi_aff *upma,
	enum isl_dim_type type);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_set_dim_name(
	__isl_take isl_union_pw_multi_aff *upma,
	enum isl_dim_type type, unsigned pos, const char *s);

int isl_union_pw_multi_aff_find_dim_by_name(
	__isl_keep isl_union_pw_multi_aff *upma, enum isl_dim_type type,
	const char *name);

__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_drop_dims(
	__isl_take isl_union_pw_multi_aff *upma,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_reset_user(
	__isl_take isl_union_pw_multi_aff *upma);

__isl_export
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_coalesce(
	__isl_take isl_union_pw_multi_aff *upma);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_gist_params(
	__isl_take isl_union_pw_multi_aff *upma, __isl_take isl_set *context);
__isl_export
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_gist(
	__isl_take isl_union_pw_multi_aff *upma,
	__isl_take isl_union_set *context);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_union_pw_multi_aff_get_ctx`, `isl_union_pw_multi_aff_get_space`, `isl_union_pw_multi_aff_get_pw_multi_aff_list`, `isl_union_pw_multi_aff_dim` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_union_pw_multi_aff_get_ctx`, `isl_union_pw_multi_aff_get_space`, `isl_union_pw_multi_aff_get_pw_multi_aff_list`, `isl_union_pw_multi_aff_dim` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 965-996

````cpp
__isl_overload
__isl_give isl_union_pw_multi_aff *
isl_union_pw_multi_aff_pullback_union_pw_multi_aff(
	__isl_take isl_union_pw_multi_aff *upma1,
	__isl_take isl_union_pw_multi_aff *upma2);
__isl_overload
__isl_give isl_union_pw_multi_aff *
isl_union_pw_multi_aff_apply_union_pw_multi_aff(
	__isl_take isl_union_pw_multi_aff *upma1,
	__isl_take isl_union_pw_multi_aff *upma2);
__isl_overload
__isl_give isl_union_pw_multi_aff *
isl_union_pw_multi_aff_preimage_domain_wrapped_domain_union_pw_multi_aff(
	__isl_take isl_union_pw_multi_aff *upma1,
	__isl_take isl_union_pw_multi_aff *upma2);

__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_align_params(
	__isl_take isl_union_pw_multi_aff *upma, __isl_take isl_space *model);
__isl_export
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_drop_unused_params(
	__isl_take isl_union_pw_multi_aff *upma);

isl_size isl_union_pw_multi_aff_n_pw_multi_aff(
	__isl_keep isl_union_pw_multi_aff *upma);

isl_stat isl_union_pw_multi_aff_foreach_pw_multi_aff(
	__isl_keep isl_union_pw_multi_aff *upma,
	isl_stat (*fn)(__isl_take isl_pw_multi_aff *pma, void *user),
	void *user);
isl_bool isl_union_pw_multi_aff_every_pw_multi_aff(
	__isl_keep isl_union_pw_multi_aff *upma,
	isl_bool (*test)(__isl_keep isl_pw_multi_aff *pma, void *user),
````
- **EN**: This block declares or defines routines around `isl_union_pw_multi_aff_pullback_union_pw_multi_aff`, `isl_union_pw_multi_aff_apply_union_pw_multi_aff`, `isl_union_pw_multi_aff_preimage_domain_wrapped_domain_union_pw_multi_aff`, `isl_union_pw_multi_aff_align_params` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_multi_aff_pullback_union_pw_multi_aff`, `isl_union_pw_multi_aff_apply_union_pw_multi_aff`, `isl_union_pw_multi_aff_preimage_domain_wrapped_domain_union_pw_multi_aff`, `isl_union_pw_multi_aff_align_params` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 997-1024

````cpp
	void *user);
__isl_export
__isl_give isl_pw_multi_aff *isl_union_pw_multi_aff_extract_pw_multi_aff(
	__isl_keep isl_union_pw_multi_aff *upma, __isl_take isl_space *space);
__isl_export
isl_bool isl_union_pw_multi_aff_isa_pw_multi_aff(
	__isl_keep isl_union_pw_multi_aff *upma);
__isl_export
__isl_give isl_pw_multi_aff *isl_union_pw_multi_aff_as_pw_multi_aff(
	__isl_take isl_union_pw_multi_aff *upma);

__isl_export
isl_bool isl_union_pw_multi_aff_plain_is_empty(
	__isl_keep isl_union_pw_multi_aff *upma);
__isl_export
isl_bool isl_union_pw_multi_aff_involves_locals(
	__isl_keep isl_union_pw_multi_aff *upma);
isl_bool isl_union_pw_multi_aff_involves_nan(
	__isl_keep isl_union_pw_multi_aff *upma);
__isl_export
isl_bool isl_union_pw_multi_aff_plain_is_equal(
	__isl_keep isl_union_pw_multi_aff *upma1,
	__isl_keep isl_union_pw_multi_aff *upma2);

__isl_export
__isl_give isl_union_set *isl_union_pw_multi_aff_domain(
	__isl_take isl_union_pw_multi_aff *upma);

````
- **EN**: This block declares or defines routines around `isl_union_pw_multi_aff_extract_pw_multi_aff`, `isl_union_pw_multi_aff_isa_pw_multi_aff`, `isl_union_pw_multi_aff_as_pw_multi_aff`, `isl_union_pw_multi_aff_plain_is_empty` (+4 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_multi_aff_extract_pw_multi_aff`, `isl_union_pw_multi_aff_isa_pw_multi_aff`, `isl_union_pw_multi_aff_as_pw_multi_aff`, `isl_union_pw_multi_aff_plain_is_empty` (+4 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1025-1062

````cpp
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_neg(
	__isl_take isl_union_pw_multi_aff *upma);

__isl_export
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_add(
	__isl_take isl_union_pw_multi_aff *upma1,
	__isl_take isl_union_pw_multi_aff *upma2);
__isl_export
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_union_add(
	__isl_take isl_union_pw_multi_aff *upma1,
	__isl_take isl_union_pw_multi_aff *upma2);
__isl_export
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_sub(
	__isl_take isl_union_pw_multi_aff *upma1,
	__isl_take isl_union_pw_multi_aff *upma2);

__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_scale_val(
	__isl_take isl_union_pw_multi_aff *upma, __isl_take isl_val *val);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_scale_down_val(
	__isl_take isl_union_pw_multi_aff *upma, __isl_take isl_val *val);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_scale_multi_val(
	__isl_take isl_union_pw_multi_aff *upma, __isl_take isl_multi_val *mv);

__isl_export
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_range_product(
	__isl_take isl_union_pw_multi_aff *upma1,
	__isl_take isl_union_pw_multi_aff *upma2);
__isl_export
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_flat_range_product(
	__isl_take isl_union_pw_multi_aff *upma1,
	__isl_take isl_union_pw_multi_aff *upma2);
__isl_export
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_range_factor_domain(
	__isl_take isl_union_pw_multi_aff *upma);
__isl_export
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_range_factor_range(
	__isl_take isl_union_pw_multi_aff *upma);

````
- **EN**: This block declares or defines routines around `isl_union_pw_multi_aff_neg`, `isl_union_pw_multi_aff_add`, `isl_union_pw_multi_aff_union_add`, `isl_union_pw_multi_aff_sub` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_multi_aff_neg`, `isl_union_pw_multi_aff_add`, `isl_union_pw_multi_aff_union_add`, `isl_union_pw_multi_aff_sub` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1063-1099

````cpp
__isl_export
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_intersect_params(
	__isl_take isl_union_pw_multi_aff *upma, __isl_take isl_set *set);
__isl_overload
__isl_give isl_union_pw_multi_aff *
isl_union_pw_multi_aff_intersect_domain_union_set(
	__isl_take isl_union_pw_multi_aff *upma,
	__isl_take isl_union_set *uset);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_intersect_domain(
	__isl_take isl_union_pw_multi_aff *upma,
	__isl_take isl_union_set *uset);
__isl_overload
__isl_give isl_union_pw_multi_aff *
isl_union_pw_multi_aff_intersect_domain_space(
	__isl_take isl_union_pw_multi_aff *upma, __isl_take isl_space *space);
__isl_export
__isl_give isl_union_pw_multi_aff *
isl_union_pw_multi_aff_intersect_domain_wrapped_domain(
	__isl_take isl_union_pw_multi_aff *upma,
	__isl_take isl_union_set *uset);
__isl_export
__isl_give isl_union_pw_multi_aff *
isl_union_pw_multi_aff_intersect_domain_wrapped_range(
	__isl_take isl_union_pw_multi_aff *upma,
	__isl_take isl_union_set *uset);
__isl_overload
__isl_give isl_union_pw_multi_aff *
isl_union_pw_multi_aff_subtract_domain_union_set(
	__isl_take isl_union_pw_multi_aff *upma,
	__isl_take isl_union_set *uset);
__isl_overload
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_subtract_domain_space(
	__isl_take isl_union_pw_multi_aff *upma, __isl_take isl_space *space);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_subtract_domain(
	__isl_take isl_union_pw_multi_aff *upma,
	__isl_take isl_union_set *uset);

````
- **EN**: This block declares or defines routines around `isl_union_pw_multi_aff_intersect_params`, `isl_union_pw_multi_aff_intersect_domain_union_set`, `isl_union_pw_multi_aff_intersect_domain`, `isl_union_pw_multi_aff_intersect_domain_space` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_multi_aff_intersect_params`, `isl_union_pw_multi_aff_intersect_domain_union_set`, `isl_union_pw_multi_aff_intersect_domain`, `isl_union_pw_multi_aff_intersect_domain_space` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1100-1126

````cpp
__isl_export
__isl_give isl_union_map *isl_union_pw_multi_aff_as_union_map(
	__isl_take isl_union_pw_multi_aff *upma);
__isl_overload
__isl_give isl_union_map *isl_union_map_from_union_pw_multi_aff(
	__isl_take isl_union_pw_multi_aff *upma);

__isl_give isl_printer *isl_printer_print_union_pw_multi_aff(
	__isl_take isl_printer *p, __isl_keep isl_union_pw_multi_aff *upma);

__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_from_union_set(
	__isl_take isl_union_set *uset);
__isl_export
__isl_give isl_union_pw_multi_aff *isl_union_map_as_union_pw_multi_aff(
	__isl_take isl_union_map *umap);
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_from_union_map(
	__isl_take isl_union_map *umap);

__isl_constructor
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_read_from_str(
	isl_ctx *ctx, const char *str);
void isl_union_pw_multi_aff_dump(__isl_keep isl_union_pw_multi_aff *upma);
__isl_give char *isl_union_pw_multi_aff_to_str(
	__isl_keep isl_union_pw_multi_aff *upma);

uint32_t isl_multi_pw_aff_get_hash(__isl_keep isl_multi_pw_aff *mpa);

````
- **EN**: This block declares or defines routines around `isl_union_pw_multi_aff_as_union_map`, `isl_union_map_from_union_pw_multi_aff`, `isl_printer_print_union_pw_multi_aff`, `isl_union_pw_multi_aff_from_union_set` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_multi_aff_as_union_map`, `isl_union_map_from_union_pw_multi_aff`, `isl_printer_print_union_pw_multi_aff`, `isl_union_pw_multi_aff_from_union_set` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1127-1156

````cpp
__isl_constructor
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_from_aff(__isl_take isl_aff *aff);
__isl_export
__isl_give isl_multi_pw_aff *isl_multi_aff_to_multi_pw_aff(
	__isl_take isl_multi_aff *ma);
__isl_constructor
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_from_multi_aff(
	__isl_take isl_multi_aff *ma);
__isl_constructor
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_from_pw_aff(
	__isl_take isl_pw_aff *pa);
__isl_export
__isl_give isl_set *isl_multi_pw_aff_domain(__isl_take isl_multi_pw_aff *mpa);
__isl_export
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_intersect_params(
	__isl_take isl_multi_pw_aff *mpa, __isl_take isl_set *set);
__isl_export
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_intersect_domain(
	__isl_take isl_multi_pw_aff *mpa, __isl_take isl_set *domain);

__isl_export
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_coalesce(
	__isl_take isl_multi_pw_aff *mpa);
__isl_export
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_gist(
	__isl_take isl_multi_pw_aff *mpa, __isl_take isl_set *set);
__isl_export
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_gist_params(
	__isl_take isl_multi_pw_aff *mpa, __isl_take isl_set *set);

````
- **EN**: This block declares or defines routines around `isl_multi_pw_aff_from_aff`, `isl_multi_aff_to_multi_pw_aff`, `isl_multi_pw_aff_from_multi_aff`, `isl_multi_pw_aff_from_pw_aff` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_multi_pw_aff_from_aff`, `isl_multi_aff_to_multi_pw_aff`, `isl_multi_pw_aff_from_multi_aff`, `isl_multi_pw_aff_from_pw_aff` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1157-1185

````cpp
isl_bool isl_multi_pw_aff_is_cst(__isl_keep isl_multi_pw_aff *mpa);
isl_bool isl_multi_pw_aff_is_equal(__isl_keep isl_multi_pw_aff *mpa1,
	__isl_keep isl_multi_pw_aff *mpa2);

__isl_overload
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_pullback_multi_aff(
	__isl_take isl_multi_pw_aff *mpa, __isl_take isl_multi_aff *ma);
__isl_overload
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_pullback_pw_multi_aff(
	__isl_take isl_multi_pw_aff *mpa, __isl_take isl_pw_multi_aff *pma);
__isl_overload
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_pullback_multi_pw_aff(
	__isl_take isl_multi_pw_aff *mpa1, __isl_take isl_multi_pw_aff *mpa2);

__isl_export
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_union_add(
	__isl_take isl_multi_pw_aff *mpa1, __isl_take isl_multi_pw_aff *mpa2);

__isl_give isl_multi_pw_aff *isl_multi_pw_aff_move_dims(
	__isl_take isl_multi_pw_aff *pma,
	enum isl_dim_type dst_type, unsigned dst_pos,
	enum isl_dim_type src_type, unsigned src_pos, unsigned n);

__isl_export
isl_bool isl_multi_pw_aff_isa_multi_aff(__isl_keep isl_multi_pw_aff *mpa);
__isl_export
__isl_give isl_multi_aff *isl_multi_pw_aff_as_multi_aff(
	__isl_take isl_multi_pw_aff *mpa);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_multi_pw_aff_is_cst`, `isl_multi_pw_aff_is_equal`, `isl_multi_pw_aff_pullback_multi_aff`, `isl_multi_pw_aff_pullback_pw_multi_aff` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_multi_pw_aff_is_cst`, `isl_multi_pw_aff_is_equal`, `isl_multi_pw_aff_pullback_multi_aff`, `isl_multi_pw_aff_pullback_pw_multi_aff` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1186-1211

````cpp
__isl_export
__isl_give isl_set *isl_multi_pw_aff_as_set(__isl_take isl_multi_pw_aff *mpa);
__isl_give isl_set *isl_set_from_multi_pw_aff(__isl_take isl_multi_pw_aff *mpa);
__isl_export
__isl_give isl_map *isl_multi_pw_aff_as_map(__isl_take isl_multi_pw_aff *mpa);
__isl_give isl_map *isl_map_from_multi_pw_aff(__isl_take isl_multi_pw_aff *mpa);
__isl_give isl_pw_multi_aff *isl_pw_multi_aff_from_multi_pw_aff(
	__isl_take isl_multi_pw_aff *mpa);
__isl_export
__isl_give isl_multi_pw_aff *isl_pw_multi_aff_to_multi_pw_aff(
	__isl_take isl_pw_multi_aff *pma);
__isl_constructor
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_from_pw_multi_aff(
	__isl_take isl_pw_multi_aff *pma);

__isl_give isl_map *isl_multi_pw_aff_eq_map(__isl_take isl_multi_pw_aff *mpa1,
	__isl_take isl_multi_pw_aff *mpa2);
__isl_give isl_map *isl_multi_pw_aff_lex_le_map(
	__isl_take isl_multi_pw_aff *mpa1, __isl_take isl_multi_pw_aff *mpa2);
__isl_give isl_map *isl_multi_pw_aff_lex_lt_map(
	__isl_take isl_multi_pw_aff *mpa1, __isl_take isl_multi_pw_aff *mpa2);
__isl_give isl_map *isl_multi_pw_aff_lex_ge_map(
	__isl_take isl_multi_pw_aff *mpa1, __isl_take isl_multi_pw_aff *mpa2);
__isl_give isl_map *isl_multi_pw_aff_lex_gt_map(
	__isl_take isl_multi_pw_aff *mpa1, __isl_take isl_multi_pw_aff *mpa2);

````
- **EN**: This block declares or defines routines around `isl_multi_pw_aff_as_set`, `isl_set_from_multi_pw_aff`, `isl_multi_pw_aff_as_map`, `isl_map_from_multi_pw_aff` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_multi_pw_aff_as_set`, `isl_set_from_multi_pw_aff`, `isl_multi_pw_aff_as_map`, `isl_map_from_multi_pw_aff` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1212-1241

````cpp
__isl_export
__isl_give isl_set *isl_multi_pw_aff_bind(__isl_take isl_multi_pw_aff *mpa,
	__isl_take isl_multi_id *tuple);

__isl_constructor
__isl_give isl_multi_pw_aff *isl_multi_pw_aff_read_from_str(isl_ctx *ctx,
	const char *str);
__isl_give char *isl_multi_pw_aff_to_str(__isl_keep isl_multi_pw_aff *mpa);
__isl_give isl_printer *isl_printer_print_multi_pw_aff(
	__isl_take isl_printer *p, __isl_keep isl_multi_pw_aff *mpa);
void isl_multi_pw_aff_dump(__isl_keep isl_multi_pw_aff *mpa);

__isl_give isl_union_pw_aff *isl_union_pw_aff_copy(
	__isl_keep isl_union_pw_aff *upa);
__isl_null isl_union_pw_aff *isl_union_pw_aff_free(
	__isl_take isl_union_pw_aff *upa);

isl_ctx *isl_union_pw_aff_get_ctx(__isl_keep isl_union_pw_aff *upa);
__isl_export
__isl_give isl_space *isl_union_pw_aff_get_space(
	__isl_keep isl_union_pw_aff *upa);
__isl_give isl_pw_aff_list *isl_union_pw_aff_get_pw_aff_list(
	__isl_keep isl_union_pw_aff *upa);

isl_size isl_union_pw_aff_dim(__isl_keep isl_union_pw_aff *upa,
	enum isl_dim_type type);
__isl_give isl_union_pw_aff *isl_union_pw_aff_set_dim_name(
	__isl_take isl_union_pw_aff *upa, enum isl_dim_type type,
	unsigned pos, const char *s);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_multi_pw_aff_bind`, `isl_multi_pw_aff_read_from_str`, `isl_multi_pw_aff_to_str`, `isl_printer_print_multi_pw_aff` (+8 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_multi_pw_aff_bind`, `isl_multi_pw_aff_read_from_str`, `isl_multi_pw_aff_to_str`, `isl_printer_print_multi_pw_aff` (+8 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1242-1272

````cpp
int isl_union_pw_aff_find_dim_by_name(__isl_keep isl_union_pw_aff *upa,
	enum isl_dim_type type, const char *name);

__isl_give isl_union_pw_aff *isl_union_pw_aff_drop_dims(
	__isl_take isl_union_pw_aff *upa,
	enum isl_dim_type type, unsigned first, unsigned n);
__isl_give isl_union_pw_aff *isl_union_pw_aff_reset_user(
	__isl_take isl_union_pw_aff *upa);

__isl_give isl_union_pw_aff *isl_union_pw_aff_empty_ctx(isl_ctx *ctx);
__isl_give isl_union_pw_aff *isl_union_pw_aff_empty_space(
	__isl_take isl_space *space);
__isl_give isl_union_pw_aff *isl_union_pw_aff_empty(
	__isl_take isl_space *space);
__isl_constructor
__isl_give isl_union_pw_aff *isl_union_pw_aff_from_aff(__isl_take isl_aff *aff);
__isl_export
__isl_give isl_union_pw_aff *isl_pw_aff_to_union_pw_aff(
	__isl_take isl_pw_aff *pa);
__isl_constructor
__isl_give isl_union_pw_aff *isl_union_pw_aff_from_pw_aff(
	__isl_take isl_pw_aff *pa);
__isl_give isl_union_pw_aff *isl_union_pw_aff_val_on_domain(
	__isl_take isl_union_set *domain, __isl_take isl_val *v);
__isl_give isl_union_pw_aff *isl_union_pw_aff_aff_on_domain(
	__isl_take isl_union_set *domain, __isl_take isl_aff *aff);
__isl_give isl_union_pw_aff *isl_union_pw_aff_pw_aff_on_domain(
	__isl_take isl_union_set *domain, __isl_take isl_pw_aff *pa);
__isl_give isl_union_pw_aff *isl_union_pw_aff_add_pw_aff(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_pw_aff *pa);

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_union_pw_aff_find_dim_by_name`, `isl_union_pw_aff_drop_dims`, `isl_union_pw_aff_reset_user`, `isl_union_pw_aff_empty_ctx` (+9 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_union_pw_aff_find_dim_by_name`, `isl_union_pw_aff_drop_dims`, `isl_union_pw_aff_reset_user`, `isl_union_pw_aff_empty_ctx` (+9 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1273-1307

````cpp
__isl_constructor
__isl_give isl_union_pw_multi_aff *isl_union_pw_multi_aff_from_union_pw_aff(
	__isl_take isl_union_pw_aff *upa);

isl_size isl_union_pw_aff_n_pw_aff(__isl_keep isl_union_pw_aff *upa);

isl_stat isl_union_pw_aff_foreach_pw_aff(__isl_keep isl_union_pw_aff *upa,
	isl_stat (*fn)(__isl_take isl_pw_aff *pa, void *user), void *user);
isl_bool isl_union_pw_aff_every_pw_aff(__isl_keep isl_union_pw_aff *upa,
	isl_bool (*test)(__isl_keep isl_pw_aff *pa, void *user), void *user);
__isl_give isl_pw_aff *isl_union_pw_aff_extract_pw_aff(
	__isl_keep isl_union_pw_aff *upa, __isl_take isl_space *space);

isl_bool isl_union_pw_aff_involves_nan(__isl_keep isl_union_pw_aff *upa);
__isl_export
isl_bool isl_union_pw_aff_plain_is_equal(__isl_keep isl_union_pw_aff *upa1,
	__isl_keep isl_union_pw_aff *upa2);

__isl_export
__isl_give isl_union_set *isl_union_pw_aff_domain(
	__isl_take isl_union_pw_aff *upa);

__isl_give isl_union_pw_aff *isl_union_pw_aff_neg(
	__isl_take isl_union_pw_aff *upa);

__isl_export
__isl_give isl_union_pw_aff *isl_union_pw_aff_add(
	__isl_take isl_union_pw_aff *upa1, __isl_take isl_union_pw_aff *upa2);
__isl_export
__isl_give isl_union_pw_aff *isl_union_pw_aff_union_add(
	__isl_take isl_union_pw_aff *upa1, __isl_take isl_union_pw_aff *upa2);
__isl_export
__isl_give isl_union_pw_aff *isl_union_pw_aff_sub(
	__isl_take isl_union_pw_aff *upa1, __isl_take isl_union_pw_aff *upa2);

````
- **EN**: This block declares or defines routines around `isl_union_pw_multi_aff_from_union_pw_aff`, `isl_union_pw_aff_n_pw_aff`, `isl_union_pw_aff_foreach_pw_aff`, `isl_stat` (+10 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_multi_aff_from_union_pw_aff`, `isl_union_pw_aff_n_pw_aff`, `isl_union_pw_aff_foreach_pw_aff`, `isl_stat` (+10 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1308-1337

````cpp
__isl_export
__isl_give isl_union_pw_aff *isl_union_pw_aff_coalesce(
	__isl_take isl_union_pw_aff *upa);
__isl_export
__isl_give isl_union_pw_aff *isl_union_pw_aff_gist(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_union_set *context);
__isl_give isl_union_pw_aff *isl_union_pw_aff_gist_params(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_set *context);

__isl_overload
__isl_give isl_union_pw_aff *isl_union_pw_aff_pullback_union_pw_multi_aff(
	__isl_take isl_union_pw_aff *upa,
	__isl_take isl_union_pw_multi_aff *upma);

__isl_give isl_union_pw_aff *isl_union_pw_aff_floor(
	__isl_take isl_union_pw_aff *upa);

__isl_give isl_union_pw_aff *isl_union_pw_aff_scale_val(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_val *v);
__isl_give isl_union_pw_aff *isl_union_pw_aff_scale_down_val(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_val *v);
__isl_give isl_union_pw_aff *isl_union_pw_aff_mod_val(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_val *f);

__isl_give isl_union_pw_aff *isl_union_pw_aff_align_params(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_space *model);
__isl_export
__isl_give isl_union_pw_aff *isl_union_pw_aff_drop_unused_params(
	__isl_take isl_union_pw_aff *upa);

````
- **EN**: This block declares or defines routines around `isl_union_pw_aff_coalesce`, `isl_union_pw_aff_gist`, `isl_union_pw_aff_gist_params`, `isl_union_pw_aff_pullback_union_pw_multi_aff` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_aff_coalesce`, `isl_union_pw_aff_gist`, `isl_union_pw_aff_gist_params`, `isl_union_pw_aff_pullback_union_pw_multi_aff` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1338-1363

````cpp
__isl_export
__isl_give isl_union_pw_aff *isl_union_pw_aff_intersect_params(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_set *set);
__isl_overload
__isl_give isl_union_pw_aff *isl_union_pw_aff_intersect_domain_space(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_space *space);
__isl_overload
__isl_give isl_union_pw_aff *isl_union_pw_aff_intersect_domain_union_set(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_union_set *uset);
__isl_give isl_union_pw_aff *isl_union_pw_aff_intersect_domain(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_union_set *uset);
__isl_export
__isl_give isl_union_pw_aff *isl_union_pw_aff_intersect_domain_wrapped_domain(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_union_set *uset);
__isl_export
__isl_give isl_union_pw_aff *isl_union_pw_aff_intersect_domain_wrapped_range(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_union_set *uset);
__isl_overload
__isl_give isl_union_pw_aff *isl_union_pw_aff_subtract_domain_union_set(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_union_set *uset);
__isl_overload
__isl_give isl_union_pw_aff *isl_union_pw_aff_subtract_domain_space(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_space *space);
__isl_give isl_union_pw_aff *isl_union_pw_aff_subtract_domain(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_union_set *uset);

````
- **EN**: This block declares or defines routines around `isl_union_pw_aff_intersect_params`, `isl_union_pw_aff_intersect_domain_space`, `isl_union_pw_aff_intersect_domain_union_set`, `isl_union_pw_aff_intersect_domain` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_aff_intersect_params`, `isl_union_pw_aff_intersect_domain_space`, `isl_union_pw_aff_intersect_domain_union_set`, `isl_union_pw_aff_intersect_domain` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1364-1393

````cpp
__isl_give isl_union_pw_aff *isl_union_pw_aff_set_dim_name(
	__isl_take isl_union_pw_aff *upa,
	enum isl_dim_type type, unsigned pos, const char *s);

__isl_give isl_union_set *isl_union_pw_aff_zero_union_set(
	__isl_take isl_union_pw_aff *upa);

__isl_give isl_union_map *isl_union_map_from_union_pw_aff(
	__isl_take isl_union_pw_aff *upa);

__isl_overload
__isl_give isl_union_set *isl_union_pw_aff_bind_id(
	__isl_take isl_union_pw_aff *upa, __isl_take isl_id *id);

__isl_constructor
__isl_give isl_union_pw_aff *isl_union_pw_aff_read_from_str(isl_ctx *ctx,
	const char *str);
__isl_give char *isl_union_pw_aff_to_str(__isl_keep isl_union_pw_aff *upa);
__isl_give isl_printer *isl_printer_print_union_pw_aff(
	__isl_take isl_printer *p, __isl_keep isl_union_pw_aff *upa);
void isl_union_pw_aff_dump(__isl_keep isl_union_pw_aff *upa);

ISL_DECLARE_MULTI(union_pw_aff)
ISL_DECLARE_MULTI_ARITH(union_pw_aff)
ISL_DECLARE_MULTI_ZERO(union_pw_aff)
ISL_DECLARE_MULTI_NAN(union_pw_aff)
ISL_DECLARE_MULTI_DROP_DIMS(union_pw_aff)
ISL_DECLARE_MULTI_DIM_ID(union_pw_aff)
ISL_DECLARE_MULTI_TUPLE_ID(union_pw_aff)

````
- **EN**: This block defines enum values such as `isl_dim_type`; declares or defines routines around `isl_union_pw_aff_set_dim_name`, `isl_union_pw_aff_zero_union_set`, `isl_union_map_from_union_pw_aff`, `isl_union_pw_aff_bind_id` (+11 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 定义枚举类型，例如 `isl_dim_type`; 声明或定义与 `isl_union_pw_aff_set_dim_name`, `isl_union_pw_aff_zero_union_set`, `isl_union_map_from_union_pw_aff`, `isl_union_pw_aff_bind_id` (+11 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1394-1425

````cpp
__isl_export
__isl_give isl_multi_union_pw_aff *isl_multi_aff_to_multi_union_pw_aff(
        __isl_take isl_multi_aff *ma);
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_from_multi_aff(
	__isl_take isl_multi_aff *ma);
__isl_constructor
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_from_union_pw_aff(
	__isl_take isl_union_pw_aff *upa);
__isl_constructor
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_from_multi_pw_aff(
	__isl_take isl_multi_pw_aff *mpa);
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_multi_val_on_domain(
	__isl_take isl_union_set *domain, __isl_take isl_multi_val *mv);
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_multi_aff_on_domain(
	__isl_take isl_union_set *domain, __isl_take isl_multi_aff *ma);
__isl_give isl_multi_union_pw_aff *
isl_multi_union_pw_aff_pw_multi_aff_on_domain(__isl_take isl_union_set *domain,
	__isl_take isl_pw_multi_aff *pma);

__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_floor(
	__isl_take isl_multi_union_pw_aff *mupa);

__isl_export
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_intersect_domain(
	__isl_take isl_multi_union_pw_aff *mupa,
	__isl_take isl_union_set *uset);
__isl_export
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_intersect_params(
	__isl_take isl_multi_union_pw_aff *mupa, __isl_take isl_set *params);
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_intersect_range(
	__isl_take isl_multi_union_pw_aff *mupa, __isl_take isl_set *set);

````
- **EN**: This block declares or defines routines around `isl_multi_aff_to_multi_union_pw_aff`, `isl_multi_union_pw_aff_from_multi_aff`, `isl_multi_union_pw_aff_from_union_pw_aff`, `isl_multi_union_pw_aff_from_multi_pw_aff` (+7 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_multi_aff_to_multi_union_pw_aff`, `isl_multi_union_pw_aff_from_multi_aff`, `isl_multi_union_pw_aff_from_union_pw_aff`, `isl_multi_union_pw_aff_from_multi_pw_aff` (+7 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1426-1456

````cpp
__isl_export
__isl_give isl_union_set *isl_multi_union_pw_aff_domain(
	__isl_take isl_multi_union_pw_aff *mupa);

__isl_export
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_coalesce(
	__isl_take isl_multi_union_pw_aff *mupa);
__isl_export
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_gist(
	__isl_take isl_multi_union_pw_aff *mupa,
	__isl_take isl_union_set *context);
__isl_export
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_gist_params(
	__isl_take isl_multi_union_pw_aff *mupa, __isl_take isl_set *context);

__isl_give isl_union_pw_aff *isl_multi_union_pw_aff_apply_aff(
	__isl_take isl_multi_union_pw_aff *mupa, __isl_take isl_aff *aff);
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_apply_multi_aff(
	__isl_take isl_multi_union_pw_aff *mupa, __isl_take isl_multi_aff *ma);
__isl_give isl_union_pw_aff *isl_multi_union_pw_aff_apply_pw_aff(
	__isl_take isl_multi_union_pw_aff *mupa, __isl_take isl_pw_aff *pa);
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_apply_pw_multi_aff(
	__isl_take isl_multi_union_pw_aff *mupa,
	__isl_take isl_pw_multi_aff *pma);

__isl_overload
__isl_give isl_multi_union_pw_aff *
isl_multi_union_pw_aff_pullback_union_pw_multi_aff(
	__isl_take isl_multi_union_pw_aff *mupa,
	__isl_take isl_union_pw_multi_aff *upma);

````
- **EN**: This block declares or defines routines around `isl_multi_union_pw_aff_domain`, `isl_multi_union_pw_aff_coalesce`, `isl_multi_union_pw_aff_gist`, `isl_multi_union_pw_aff_gist_params` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_multi_union_pw_aff_domain`, `isl_multi_union_pw_aff_coalesce`, `isl_multi_union_pw_aff_gist`, `isl_multi_union_pw_aff_gist_params` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1457-1482

````cpp
__isl_give isl_union_pw_multi_aff *
isl_union_pw_multi_aff_from_multi_union_pw_aff(
	__isl_take isl_multi_union_pw_aff *mupa);

__isl_export
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_union_add(
	__isl_take isl_multi_union_pw_aff *mupa1,
	__isl_take isl_multi_union_pw_aff *mupa2);

__isl_export
__isl_give isl_multi_union_pw_aff *
isl_union_pw_multi_aff_as_multi_union_pw_aff(
	__isl_take isl_union_pw_multi_aff *upma);
__isl_give isl_multi_union_pw_aff *
isl_multi_union_pw_aff_from_union_pw_multi_aff(
	__isl_take isl_union_pw_multi_aff *upma);

__isl_export
__isl_give isl_multi_union_pw_aff *isl_union_map_as_multi_union_pw_aff(
	__isl_take isl_union_map *umap);
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_from_union_map(
	__isl_take isl_union_map *umap);
__isl_overload
__isl_give isl_union_map *isl_union_map_from_multi_union_pw_aff(
	__isl_take isl_multi_union_pw_aff *mupa);

````
- **EN**: This block declares or defines routines around `isl_union_pw_multi_aff_from_multi_union_pw_aff`, `isl_multi_union_pw_aff_union_add`, `isl_union_pw_multi_aff_as_multi_union_pw_aff`, `isl_multi_union_pw_aff_from_union_pw_multi_aff` (+3 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_multi_aff_from_multi_union_pw_aff`, `isl_multi_union_pw_aff_union_add`, `isl_union_pw_multi_aff_as_multi_union_pw_aff`, `isl_multi_union_pw_aff_from_union_pw_multi_aff` (+3 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1483-1511

````cpp
__isl_give isl_union_set *isl_multi_union_pw_aff_zero_union_set(
	__isl_take isl_multi_union_pw_aff *mupa);
__isl_export
__isl_give isl_union_set *isl_multi_union_pw_aff_bind(
	__isl_take isl_multi_union_pw_aff *mupa,
	__isl_take isl_multi_id *tuple);

__isl_give isl_multi_pw_aff *isl_multi_union_pw_aff_extract_multi_pw_aff(
	__isl_keep isl_multi_union_pw_aff *mupa, __isl_take isl_space *space);

__isl_constructor
__isl_give isl_multi_union_pw_aff *isl_multi_union_pw_aff_read_from_str(
	isl_ctx *ctx, const char *str);
__isl_give char *isl_multi_union_pw_aff_to_str(
	__isl_keep isl_multi_union_pw_aff *mupa);
__isl_give isl_printer *isl_printer_print_multi_union_pw_aff(
	__isl_take isl_printer *p, __isl_keep isl_multi_union_pw_aff *mupa);
void isl_multi_union_pw_aff_dump(__isl_keep isl_multi_union_pw_aff *mupa);

ISL_DECLARE_EXPORTED_LIST_FN(aff)
ISL_DECLARE_EXPORTED_LIST_FN_READ(aff)
ISL_DECLARE_EXPORTED_LIST_FN(pw_aff)
ISL_DECLARE_EXPORTED_LIST_FN_READ(pw_aff)
ISL_DECLARE_EXPORTED_LIST_FN(pw_multi_aff)
ISL_DECLARE_EXPORTED_LIST_FN_READ(pw_multi_aff)
ISL_DECLARE_EXPORTED_LIST_FN(union_pw_aff)
ISL_DECLARE_EXPORTED_LIST_FN_READ(union_pw_aff)
ISL_DECLARE_LIST_FN(union_pw_multi_aff)

````
- **EN**: This block declares or defines routines around `isl_multi_union_pw_aff_zero_union_set`, `isl_multi_union_pw_aff_bind`, `isl_multi_union_pw_aff_extract_multi_pw_aff`, `isl_multi_union_pw_aff_read_from_str` (+6 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_multi_union_pw_aff_zero_union_set`, `isl_multi_union_pw_aff_bind`, `isl_multi_union_pw_aff_extract_multi_pw_aff`, `isl_multi_union_pw_aff_read_from_str` (+6 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1512-1516

````cpp
#if defined(__cplusplus)
}
#endif

#endif
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Public interface declarations**
  - **CN**: 公共接口声明

## Dependencies / 依赖关系

- **ISL headers**: `isl/stdint.h`, `isl/local_space.h`, `isl/printer.h`, `isl/id_type.h`, `isl/set_type.h`, `isl/aff_type.h`, `isl/list.h`, `isl/multi.h` (+3 more) — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/stdint.h`, `isl/local_space.h`, `isl/printer.h`, `isl/id_type.h`, `isl/set_type.h`, `isl/aff_type.h`, `isl/list.h`, `isl/multi.h` (+3 more) —— 支撑多面体分析与变换的 ISL 关系/集合原语。
