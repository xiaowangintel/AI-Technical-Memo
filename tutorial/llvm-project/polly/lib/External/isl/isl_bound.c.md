# isl_bound.c — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/isl_bound.c` | `polly/lib/External/isl/isl_bound.c` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements or embeds upstream ISL support code used by Polly. | 实现或内嵌 Polly 使用的上游 ISL 支撑代码。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*
 * Copyright 2010      INRIA Saclay
 *
 * Use of this software is governed by the MIT license
 *
 * Written by Sven Verdoolaege, INRIA Saclay - Ile-de-France,
 * Parc Club Orsay Universite, ZAC des vignes, 4 rue Jacques Monod,
 * 91893 Orsay, France 
 */

#include <isl/aff.h>
#include <isl/val.h>
#include <isl_ctx_private.h>
#include <isl_map_private.h>
#include <isl_bound.h>
#include <isl_bernstein.h>
#include <isl_range.h>
#include <isl_polynomial_private.h>
#include <isl_options_private.h>

````
- **EN**: This block imports ISL, system/standard headers needed by the surrounding code; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 引入周边逻辑所需的 ISL、system/standard 头文件; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 21-41

````c
/* Given a polynomial "poly" that is constant in terms
 * of the domain variables, construct a polynomial reduction
 * of type "type" that is equal to "poly" on "bset",
 * with the domain projected onto the parameters.
 */
__isl_give isl_pw_qpolynomial_fold *isl_qpolynomial_cst_bound(
	__isl_take isl_basic_set *bset, __isl_take isl_qpolynomial *poly,
	enum isl_fold type, isl_bool *tight)
{
	isl_set *dom;
	isl_qpolynomial_fold *fold;
	isl_pw_qpolynomial_fold *pwf;

	fold = isl_qpolynomial_fold_alloc(type, poly);
	dom = isl_set_from_basic_set(bset);
	if (tight)
		*tight = isl_bool_true;
	pwf = isl_pw_qpolynomial_fold_alloc(type, dom, fold);
	return isl_pw_qpolynomial_fold_project_domain_on_params(pwf);
}

````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_qpolynomial_cst_bound`, `isl_qpolynomial_fold_alloc`, `isl_set_from_basic_set`, `isl_pw_qpolynomial_fold_alloc`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_qpolynomial_cst_bound`, `isl_qpolynomial_fold_alloc`, `isl_set_from_basic_set`, `isl_pw_qpolynomial_fold_alloc` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 42-61

````c
/* Add the bound "pwf", which is not known to be tight,
 * to the output of "bound".
 */
isl_stat isl_bound_add(struct isl_bound *bound,
	__isl_take isl_pw_qpolynomial_fold *pwf)
{
	bound->pwf = isl_pw_qpolynomial_fold_fold(bound->pwf, pwf);
	return isl_stat_non_null(bound->pwf);
}

/* Add the bound "pwf", which is known to be tight,
 * to the output of "bound".
 */
isl_stat isl_bound_add_tight(struct isl_bound *bound,
	__isl_take isl_pw_qpolynomial_fold *pwf)
{
	bound->pwf_tight = isl_pw_qpolynomial_fold_fold(bound->pwf_tight, pwf);
	return isl_stat_non_null(bound->pwf);
}

````
- **EN**: This block declares or defines routines around `isl_bound_add`, `isl_pw_qpolynomial_fold_fold`, `isl_bound_add_tight`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_bound_add`, `isl_pw_qpolynomial_fold_fold`, `isl_bound_add_tight` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 62-85

````c
/* Given a polynomial "poly" that is constant in terms
 * of the domain variables and the domain "bset",
 * construct the corresponding polynomial reduction and
 * add it to the tight bounds of "bound".
 */
static isl_stat add_constant_poly(__isl_take isl_basic_set *bset,
	__isl_take isl_qpolynomial *poly, struct isl_bound *bound)
{
	isl_pw_qpolynomial_fold *pwf;

	pwf = isl_qpolynomial_cst_bound(bset, poly, bound->type, NULL);
	return isl_bound_add_tight(bound, pwf);
}

/* Compute a bound on the polynomial defined over the parametric polytope
 * using either range propagation or bernstein expansion and
 * store the result in bound->pwf and bound->pwf_tight.
 * Since bernstein expansion requires bounded domains, we apply
 * range propagation on unbounded domains.  Otherwise, we respect the choice
 * of the user.
 *
 * If the polynomial does not depend on the set variables
 * then the bound is equal to the polynomial and
 * it can be added to "bound" directly.
````
- **EN**: This block declares or defines routines around `add_constant_poly`, `isl_qpolynomial_cst_bound`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; handles reduction-specific behavior or metadata; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `add_constant_poly`, `isl_qpolynomial_cst_bound` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 处理归约相关行为或元数据；并延续周边实现细节。

### Lines 86-106

````c
 */
static isl_stat compressed_guarded_poly_bound(__isl_take isl_basic_set *bset,
	__isl_take isl_qpolynomial *poly, struct isl_bound *bound)
{
	isl_ctx *ctx;
	int bounded;
	int degree;

	if (!bset || !poly)
		goto error;

	degree = isl_qpolynomial_degree(poly);
	if (degree < -1)
		goto error;
	if (degree <= 0)
		return add_constant_poly(bset, poly, bound);

	ctx = isl_basic_set_get_ctx(bset);
	if (ctx->opt->bound == ISL_BOUND_RANGE)
		return isl_qpolynomial_bound_on_domain_range(bset, poly, bound);

````
- **EN**: This block declares or defines routines around `compressed_guarded_poly_bound`, `isl_qpolynomial_degree`, `isl_basic_set_get_ctx`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `compressed_guarded_poly_bound`, `isl_qpolynomial_degree`, `isl_basic_set_get_ctx` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 107-128

````c
	bounded = isl_basic_set_is_bounded(bset);
	if (bounded < 0)
		goto error;
	if (bounded)
		return isl_qpolynomial_bound_on_domain_bernstein(bset, poly, bound);
	else
		return isl_qpolynomial_bound_on_domain_range(bset, poly, bound);
error:
	isl_basic_set_free(bset);
	isl_qpolynomial_free(poly);
	return isl_stat_error;
}

static isl_stat unwrapped_guarded_poly_bound(__isl_take isl_basic_set *bset,
	__isl_take isl_qpolynomial *poly, struct isl_bound *bound)
{
	isl_pw_qpolynomial_fold *top_pwf;
	isl_pw_qpolynomial_fold *top_pwf_tight;
	isl_space *space;
	isl_morph *morph;
	isl_stat r;

````
- **EN**: This block declares or defines routines around `isl_basic_set_is_bounded`, `isl_basic_set_free`, `isl_qpolynomial_free`, `unwrapped_guarded_poly_bound`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_is_bounded`, `isl_basic_set_free`, `isl_qpolynomial_free`, `unwrapped_guarded_poly_bound` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 129-153

````c
	bset = isl_basic_set_detect_equalities(bset);

	if (!bset)
		goto error;

	if (bset->n_eq == 0)
		return compressed_guarded_poly_bound(bset, poly, bound);

	morph = isl_basic_set_full_compression(bset);

	bset = isl_morph_basic_set(isl_morph_copy(morph), bset);
	poly = isl_qpolynomial_morph_domain(poly, isl_morph_copy(morph));

	space = isl_morph_get_ran_space(morph);
	space = isl_space_params(space);

	top_pwf = bound->pwf;
	top_pwf_tight = bound->pwf_tight;

	space = isl_space_from_domain(space);
	space = isl_space_add_dims(space, isl_dim_out, 1);
	bound->pwf = isl_pw_qpolynomial_fold_zero(isl_space_copy(space),
						  bound->type);
	bound->pwf_tight = isl_pw_qpolynomial_fold_zero(space, bound->type);

````
- **EN**: This block declares or defines routines around `isl_basic_set_detect_equalities`, `isl_basic_set_full_compression`, `isl_morph_basic_set`, `isl_qpolynomial_morph_domain` (+5 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_detect_equalities`, `isl_basic_set_full_compression`, `isl_morph_basic_set`, `isl_qpolynomial_morph_domain` (+5 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 154-174

````c
	r = compressed_guarded_poly_bound(bset, poly, bound);

	morph = isl_morph_dom_params(morph);
	morph = isl_morph_ran_params(morph);
	morph = isl_morph_inverse(morph);

	bound->pwf = isl_pw_qpolynomial_fold_morph_domain(bound->pwf,
							isl_morph_copy(morph));
	bound->pwf_tight = isl_pw_qpolynomial_fold_morph_domain(
						bound->pwf_tight, morph);

	isl_bound_add(bound, top_pwf);
	isl_bound_add_tight(bound, top_pwf_tight);

	return r;
error:
	isl_basic_set_free(bset);
	isl_qpolynomial_free(poly);
	return isl_stat_error;
}

````
- **EN**: This block declares or defines routines around `compressed_guarded_poly_bound`, `isl_morph_dom_params`, `isl_morph_ran_params`, `isl_morph_inverse` (+6 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `compressed_guarded_poly_bound`, `isl_morph_dom_params`, `isl_morph_ran_params`, `isl_morph_inverse` (+6 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 175-198

````c
/* Update bound->pwf and bound->pwf_tight with a bound
 * of type bound->type on the (quasi-)polynomial "qp" over the domain "bset",
 * by calling "unwrapped" on unwrapped versions of "bset and "qp".
 * If "qp" is a polynomial, then "unwrapped" will also be called
 * on a polynomial.
 *
 * If the original problem did not have a wrapped relation in the domain,
 * then call "unwrapped" directly.
 *
 * Otherwise, the bound should be computed over the range
 * of the wrapped relation.  Temporarily treat the domain dimensions
 * of this wrapped relation as parameters, compute a bound using "unwrapped"
 * in terms of these and the original parameters,
 * turn the parameters back into set dimensions and
 * add the results to bound->pwf and bound->pwf_tight.
 *
 * Note that even though "bset" is known to live in the same space
 * as the domain of "qp", the names of the set dimensions
 * may be different (or missing).  Make sure the naming is exactly
 * the same before turning these dimensions into parameters
 * to ensure that the spaces are still the same after
 * this operation.
 */
static isl_stat unwrap(__isl_take isl_basic_set *bset,
````
- **EN**: This block declares or defines routines around `the`, `different`, `unwrap`; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `the`, `different`, `unwrap` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 199-218

````c
	__isl_take isl_qpolynomial *qp,
	isl_stat (*unwrapped)(__isl_take isl_basic_set *bset,
		__isl_take isl_qpolynomial *qp, struct isl_bound *bound),
	struct isl_bound *bound)
{
	isl_space *space;
	isl_pw_qpolynomial_fold *top_pwf;
	isl_pw_qpolynomial_fold *top_pwf_tight;
	isl_size nparam;
	isl_size n_in;
	isl_stat r;

	if (!bound->wrapping)
		return unwrapped(bset, qp, bound);

	nparam = isl_space_dim(bound->dim, isl_dim_param);
	n_in = isl_space_dim(bound->dim, isl_dim_in);
	if (nparam < 0 || n_in < 0)
		goto error;

````
- **EN**: This block declares or references types such as `isl_bound`; declares or defines routines around `isl_stat`, `isl_space_dim`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_bound`; 声明或定义与 `isl_stat`, `isl_space_dim` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 219-238

````c
	space = isl_qpolynomial_get_domain_space(qp);
	bset = isl_basic_set_reset_space(bset, space);

	bset = isl_basic_set_move_dims(bset, isl_dim_param, nparam,
					isl_dim_set, 0, n_in);
	qp = isl_qpolynomial_move_dims(qp, isl_dim_param, nparam,
					isl_dim_in, 0, n_in);

	space = isl_basic_set_get_space(bset);
	space = isl_space_params(space);

	top_pwf = bound->pwf;
	top_pwf_tight = bound->pwf_tight;

	space = isl_space_from_domain(space);
	space = isl_space_add_dims(space, isl_dim_out, 1);
	bound->pwf = isl_pw_qpolynomial_fold_zero(isl_space_copy(space),
						  bound->type);
	bound->pwf_tight = isl_pw_qpolynomial_fold_zero(space, bound->type);

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_get_domain_space`, `isl_basic_set_reset_space`, `isl_basic_set_move_dims`, `isl_qpolynomial_move_dims` (+5 more); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_get_domain_space`, `isl_basic_set_reset_space`, `isl_basic_set_move_dims`, `isl_qpolynomial_move_dims` (+5 more) 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 239-264

````c
	r = unwrapped(bset, qp, bound);

	bound->pwf = isl_pw_qpolynomial_fold_reset_space(bound->pwf,
						    isl_space_copy(bound->dim));
	bound->pwf_tight = isl_pw_qpolynomial_fold_reset_space(bound->pwf_tight,
						    isl_space_copy(bound->dim));

	isl_bound_add(bound, top_pwf);
	isl_bound_add_tight(bound, top_pwf_tight);

	return r;
error:
	isl_basic_set_free(bset);
	isl_qpolynomial_free(qp);
	return isl_stat_error;
}

/* Update bound->pwf and bound->pwf_tight with a bound
 * of type bound->type on the polynomial "poly" over the domain "bset",
 * handling any wrapping in the domain.
 */
static isl_stat guarded_poly_bound(__isl_take isl_basic_set *bset,
	__isl_take isl_qpolynomial *poly, void *user)
{
	struct isl_bound *bound = (struct isl_bound *)user;

````
- **EN**: This block declares or references types such as `isl_bound`; declares or defines routines around `unwrapped`, `isl_pw_qpolynomial_fold_reset_space`, `isl_space_copy`, `isl_bound_add` (+4 more); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_bound`; 声明或定义与 `unwrapped`, `isl_pw_qpolynomial_fold_reset_space`, `isl_space_copy`, `isl_bound_add` (+4 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 265-288

````c
	return unwrap(bset, poly, &unwrapped_guarded_poly_bound, bound);
}

/* Is "bset" bounded and is "qp" a quasi-affine expression?
 */
static isl_bool is_bounded_affine(__isl_keep isl_basic_set *bset,
	__isl_keep isl_qpolynomial *qp)
{
	isl_bool affine;

	affine = isl_qpolynomial_isa_aff(qp);
	if (affine < 0 || !affine)
		return affine;
	return isl_basic_set_is_bounded(bset);
}

/* Update bound->pwf and bound->pwf_tight with a bound
 * of type bound->type on the quasi-polynomial "qp" over the domain "bset",
 * for the case where "bset" is bounded and
 * "qp" is a quasi-affine expression and
 * they have both been unwrapped already if needed.
 *
 * Consider the set of possible function values of "qp" over "bset" and
 * take the minimum or maximum value in this set, depending
````
- **EN**: This block declares or defines routines around `is_bounded_affine`, `isl_qpolynomial_isa_aff`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `is_bounded_affine`, `isl_qpolynomial_isa_aff` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 289-311

````c
 * on whether a lower or an upper bound is being computed.
 * Do this by calling isl_set_lexmin_pw_multi_aff or
 * isl_set_lexmax_pw_multi_aff, which compute a regular minimum or maximum
 * since the set is one-dimensional.
 * Since this computation is exact, the bound is always tight.
 *
 * Note that the minimum or maximum integer value is being computed,
 * so if "qp" has some non-trivial denominator, then it needs
 * to be multiplied out first and then taken into account again
 * after computing the minimum or maximum.
 */
static isl_stat unwrapped_affine_qp(__isl_take isl_basic_set *bset,
	__isl_take isl_qpolynomial *qp, struct isl_bound *bound)
{
	isl_val *d;
	isl_aff *aff;
	isl_basic_map *bmap;
	isl_set *range;
	isl_pw_multi_aff *opt;
	isl_pw_aff *pa;
	isl_pw_qpolynomial *pwqp;
	isl_pw_qpolynomial_fold *pwf;

````
- **EN**: This block declares or defines routines around `unwrapped_affine_qp`; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `unwrapped_affine_qp` 相关的例程; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 312-332

````c
	aff = isl_qpolynomial_as_aff(qp);
	d = isl_aff_get_denominator_val(aff);
	aff = isl_aff_scale_val(aff, isl_val_copy(d));
	bmap = isl_basic_map_from_aff(aff);
	bmap = isl_basic_map_intersect_domain(bmap, bset);
	range = isl_set_from_basic_set(isl_basic_map_range(bmap));
	if (bound->type == isl_fold_min)
		opt = isl_set_lexmin_pw_multi_aff(range);
	else
		opt = isl_set_lexmax_pw_multi_aff(range);
	pa = isl_pw_multi_aff_get_at(opt, 0);
	isl_pw_multi_aff_free(opt);
	pa = isl_pw_aff_scale_down_val(pa, d);
	pwqp = isl_pw_qpolynomial_from_pw_aff(pa);
	pwf = isl_pw_qpolynomial_fold_from_pw_qpolynomial(bound->type, pwqp);

	bound->pwf_tight = isl_pw_qpolynomial_fold_fold(bound->pwf_tight, pwf);

	return isl_stat_non_null(bound->pwf_tight);
}

````
- **EN**: This block declares or defines routines around `isl_qpolynomial_as_aff`, `isl_aff_get_denominator_val`, `isl_aff_scale_val`, `isl_basic_map_from_aff` (+10 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_qpolynomial_as_aff`, `isl_aff_get_denominator_val`, `isl_aff_scale_val`, `isl_basic_map_from_aff` (+10 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 333-356

````c
/* Update bound->pwf and bound->pwf_tight with a bound
 * of type bound->type on the quasi-polynomial "qp" over the domain bound->bset,
 * for the case where bound->bset is bounded and
 * "qp" is a quasi-affine expression,
 * handling any wrapping in the domain.
 */
static isl_stat affine_qp(__isl_take isl_qpolynomial *qp,
	struct isl_bound *bound)
{
	isl_basic_set *bset;

	bset = isl_basic_set_copy(bound->bset);
	return unwrap(bset, qp, &unwrapped_affine_qp, bound);
}

/* Update bound->pwf and bound->pwf_tight with a bound
 * of type bound->type on the quasi-polynomial "qp" over the domain bound->bset.
 *
 * If bound->bset is bounded and if "qp" is a quasi-affine expression,
 * then use a specialized version.
 *
 * Otherwise, treat the integer divisions as extra variables and
 * compute a bound over the polynomial in terms of the original and
 * the extra variables.
````
- **EN**: This block declares or references types such as `isl_bound`; declares or defines routines around `affine_qp`, `isl_basic_set_copy`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_bound`; 声明或定义与 `affine_qp`, `isl_basic_set_copy` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 357-380

````c
 */
static isl_stat guarded_qp(__isl_take isl_qpolynomial *qp, void *user)
{
	struct isl_bound *bound = (struct isl_bound *)user;
	isl_stat r;
	isl_bool bounded_affine;

	bounded_affine = is_bounded_affine(bound->bset, qp);
	if (bounded_affine < 0)
		qp = isl_qpolynomial_free(qp);
	else if (bounded_affine)
		return affine_qp(qp, bound);

	r = isl_qpolynomial_as_polynomial_on_domain(qp, bound->bset,
						    &guarded_poly_bound, user);
	isl_qpolynomial_free(qp);
	return r;
}

static isl_stat basic_guarded_fold(__isl_take isl_basic_set *bset, void *user)
{
	struct isl_bound *bound = (struct isl_bound *)user;
	isl_stat r;

````
- **EN**: This block declares or references types such as `isl_bound`; declares or defines routines around `guarded_qp`, `is_bounded_affine`, `isl_qpolynomial_free`, `isl_qpolynomial_as_polynomial_on_domain` (+1 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_bound`; 声明或定义与 `guarded_qp`, `is_bounded_affine`, `isl_qpolynomial_free`, `isl_qpolynomial_as_polynomial_on_domain` (+1 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 381-400

````c
	bound->bset = bset;
	r = isl_qpolynomial_fold_foreach_qpolynomial(bound->fold,
							&guarded_qp, user);
	isl_basic_set_free(bset);
	return r;
}

static isl_stat guarded_fold(__isl_take isl_set *set,
	__isl_take isl_qpolynomial_fold *fold, void *user)
{
	struct isl_bound *bound = (struct isl_bound *)user;

	if (!set || !fold)
		goto error;

	set = isl_set_make_disjoint(set);

	bound->fold = fold;
	bound->type = isl_qpolynomial_fold_get_type(fold);

````
- **EN**: This block declares or references types such as `isl_bound`; declares or defines routines around `isl_qpolynomial_fold_foreach_qpolynomial`, `isl_basic_set_free`, `guarded_fold`, `isl_set_make_disjoint` (+1 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_bound`; 声明或定义与 `isl_qpolynomial_fold_foreach_qpolynomial`, `isl_basic_set_free`, `guarded_fold`, `isl_set_make_disjoint` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 401-420

````c
	if (isl_set_foreach_basic_set(set, &basic_guarded_fold, bound) < 0)
		goto error;

	isl_set_free(set);
	isl_qpolynomial_fold_free(fold);

	return isl_stat_ok;
error:
	isl_set_free(set);
	isl_qpolynomial_fold_free(fold);
	return isl_stat_error;
}

__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_fold_bound(
	__isl_take isl_pw_qpolynomial_fold *pwf, isl_bool *tight)
{
	isl_size nvar;
	struct isl_bound bound;
	isl_bool covers;

````
- **EN**: This block declares or references types such as `isl_bound`; declares or defines routines around `isl_set_free`, `isl_qpolynomial_fold_free`, `isl_pw_qpolynomial_fold_bound`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_bound`; 声明或定义与 `isl_set_free`, `isl_qpolynomial_fold_free`, `isl_pw_qpolynomial_fold_bound` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 421-441

````c
	if (!pwf)
		return NULL;

	bound.dim = isl_pw_qpolynomial_fold_get_domain_space(pwf);

	bound.wrapping = isl_space_is_wrapping(bound.dim);
	if (bound.wrapping)
		bound.dim = isl_space_unwrap(bound.dim);
	nvar = isl_space_dim(bound.dim, isl_dim_out);
	if (nvar < 0)
		bound.dim = isl_space_free(bound.dim);
	bound.dim = isl_space_domain(bound.dim);
	bound.dim = isl_space_from_domain(bound.dim);
	bound.dim = isl_space_add_dims(bound.dim, isl_dim_out, 1);

	if (nvar == 0) {
		if (tight)
			*tight = isl_bool_true;
		return isl_pw_qpolynomial_fold_reset_space(pwf, bound.dim);
	}

````
- **EN**: This block declares or defines routines around `isl_pw_qpolynomial_fold_get_domain_space`, `isl_space_is_wrapping`, `isl_space_unwrap`, `isl_space_dim` (+4 more); contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_pw_qpolynomial_fold_get_domain_space`, `isl_space_is_wrapping`, `isl_space_unwrap`, `isl_space_dim` (+4 more) 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 442-463

````c
	if (isl_pw_qpolynomial_fold_is_zero(pwf)) {
		enum isl_fold type = pwf->type;
		isl_pw_qpolynomial_fold_free(pwf);
		if (tight)
			*tight = isl_bool_true;
		return isl_pw_qpolynomial_fold_zero(bound.dim, type);
	}

	bound.pwf = isl_pw_qpolynomial_fold_zero(isl_space_copy(bound.dim),
							pwf->type);
	bound.pwf_tight = isl_pw_qpolynomial_fold_zero(isl_space_copy(bound.dim),
							pwf->type);
	bound.check_tight = !!tight;

	if (isl_pw_qpolynomial_fold_foreach_lifted_piece(pwf,
							guarded_fold, &bound) < 0)
		goto error;

	covers = isl_pw_qpolynomial_fold_covers(bound.pwf_tight, bound.pwf);
	if (covers < 0)
		goto error;

````
- **EN**: This block defines enum values such as `isl_fold`; declares or defines routines around `isl_pw_qpolynomial_fold_free`, `isl_pw_qpolynomial_fold_zero`, `isl_pw_qpolynomial_fold_covers`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_pw_qpolynomial_fold_free`, `isl_pw_qpolynomial_fold_zero`, `isl_pw_qpolynomial_fold_covers` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 464-485

````c
	if (tight)
		*tight = covers;

	isl_space_free(bound.dim);
	isl_pw_qpolynomial_fold_free(pwf);

	if (covers) {
		isl_pw_qpolynomial_fold_free(bound.pwf);
		return bound.pwf_tight;
	}

	bound.pwf = isl_pw_qpolynomial_fold_fold(bound.pwf, bound.pwf_tight);

	return bound.pwf;
error:
	isl_pw_qpolynomial_fold_free(bound.pwf_tight);
	isl_pw_qpolynomial_fold_free(bound.pwf);
	isl_pw_qpolynomial_fold_free(pwf);
	isl_space_free(bound.dim);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_space_free`, `isl_pw_qpolynomial_fold_free`, `isl_pw_qpolynomial_fold_fold`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_space_free`, `isl_pw_qpolynomial_fold_free`, `isl_pw_qpolynomial_fold_fold` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 486-506

````c
__isl_give isl_pw_qpolynomial_fold *isl_pw_qpolynomial_bound(
	__isl_take isl_pw_qpolynomial *pwqp, enum isl_fold type,
	isl_bool *tight)
{
	isl_pw_qpolynomial_fold *pwf;

	pwf = isl_pw_qpolynomial_fold_from_pw_qpolynomial(type, pwqp);
	return isl_pw_qpolynomial_fold_bound(pwf, tight);
}

struct isl_union_bound_data {
	enum isl_fold type;
	isl_bool tight;
	isl_union_pw_qpolynomial_fold *res;
};

static isl_stat bound_pw(__isl_take isl_pw_qpolynomial *pwqp, void *user)
{
	struct isl_union_bound_data *data = user;
	isl_pw_qpolynomial_fold *pwf;

````
- **EN**: This block declares or references types such as `isl_union_bound_data`; defines enum values such as `isl_fold`; declares or defines routines around `isl_pw_qpolynomial_bound`, `isl_pw_qpolynomial_fold_from_pw_qpolynomial`, `bound_pw`; emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_union_bound_data`; 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_pw_qpolynomial_bound`, `isl_pw_qpolynomial_fold_from_pw_qpolynomial`, `bound_pw` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 507-527

````c
	pwf = isl_pw_qpolynomial_bound(pwqp, data->type,
					data->tight ? &data->tight : NULL);
	data->res = isl_union_pw_qpolynomial_fold_fold_pw_qpolynomial_fold(
								data->res, pwf);

	return isl_stat_ok;
}

__isl_give isl_union_pw_qpolynomial_fold *isl_union_pw_qpolynomial_bound(
	__isl_take isl_union_pw_qpolynomial *upwqp,
	enum isl_fold type, isl_bool *tight)
{
	isl_space *space;
	struct isl_union_bound_data data = { type, 1, NULL };

	if (!upwqp)
		return NULL;

	if (!tight)
		data.tight = isl_bool_false;

````
- **EN**: This block declares or references types such as `isl_union_bound_data`; defines enum values such as `isl_fold`; declares or defines routines around `isl_pw_qpolynomial_bound`, `isl_union_pw_qpolynomial_fold_fold_pw_qpolynomial_fold`, `isl_union_pw_qpolynomial_bound`; contains control flow with 2 conditional check(s); and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_union_bound_data`; 定义枚举类型，例如 `isl_fold`; 声明或定义与 `isl_pw_qpolynomial_bound`, `isl_union_pw_qpolynomial_fold_fold_pw_qpolynomial_fold`, `isl_union_pw_qpolynomial_bound` 相关的例程; 包含控制流结构：2 处条件判断；并延续周边实现细节。

### Lines 528-543

````c
	space = isl_union_pw_qpolynomial_get_space(upwqp);
	data.res = isl_union_pw_qpolynomial_fold_zero(space, type);
	if (isl_union_pw_qpolynomial_foreach_pw_qpolynomial(upwqp,
						    &bound_pw, &data) < 0)
		goto error;

	isl_union_pw_qpolynomial_free(upwqp);
	if (tight)
		*tight = data.tight;

	return data.res;
error:
	isl_union_pw_qpolynomial_free(upwqp);
	isl_union_pw_qpolynomial_fold_free(data.res);
	return NULL;
}
````
- **EN**: This block declares or defines routines around `isl_union_pw_qpolynomial_get_space`, `isl_union_pw_qpolynomial_fold_zero`, `isl_union_pw_qpolynomial_free`, `isl_union_pw_qpolynomial_fold_free`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_union_pw_qpolynomial_get_space`, `isl_union_pw_qpolynomial_fold_zero`, `isl_union_pw_qpolynomial_free`, `isl_union_pw_qpolynomial_fold_free` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **Reduction handling**
  - **CN**: 归约处理

## Dependencies / 依赖关系

- **ISL headers**: `isl/aff.h`, `isl/val.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/aff.h`, `isl/val.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
- **System/standard headers**: `isl_ctx_private.h`, `isl_map_private.h`, `isl_bound.h`, `isl_bernstein.h`, `isl_range.h`, `isl_polynomial_private.h`, `isl_options_private.h` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`isl_ctx_private.h`, `isl_map_private.h`, `isl_bound.h`, `isl_bernstein.h`, `isl_range.h`, `isl_polynomial_private.h`, `isl_options_private.h` —— 实现所需的标准库或系统声明。
