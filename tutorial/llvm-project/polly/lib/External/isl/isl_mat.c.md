# isl_mat.c — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/isl_mat.c` | `polly/lib/External/isl/isl_mat.c` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements or embeds upstream ISL support code used by Polly. | 实现或内嵌 Polly 使用的上游 ISL 支撑代码。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-29

````c
/*
 * Copyright 2008-2009 Katholieke Universiteit Leuven
 * Copyright 2010      INRIA Saclay
 * Copyright 2014      Ecole Normale Superieure
 * Copyright 2017      Sven Verdoolaege
 *
 * Use of this software is governed by the MIT license
 *
 * Written by Sven Verdoolaege, K.U.Leuven, Departement
 * Computerwetenschappen, Celestijnenlaan 200A, B-3001 Leuven, Belgium
 * and INRIA Saclay - Ile-de-France, Parc Club Orsay Universite,
 * ZAC des vignes, 4 rue Jacques Monod, 91893 Orsay, France
 * and Ecole Normale Superieure, 45 rue d'Ulm, 75230 Paris, France
 */

#include <isl_ctx_private.h>
#include <isl_map_private.h>
#include <isl/space.h>
#include <isl_seq.h>
#include <isl_mat_private.h>
#include <isl_vec_private.h>
#include <isl_space_private.h>
#include <isl_val_private.h>

isl_ctx *isl_mat_get_ctx(__isl_keep isl_mat *mat)
{
	return mat ? mat->ctx : NULL;
}

````
- **EN**: This block imports ISL, system/standard headers needed by the surrounding code; declares or defines routines around `isl_mat_get_ctx`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 ISL、system/standard 头文件; 声明或定义与 `isl_mat_get_ctx` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 30-58

````c
/* Return a hash value that digests "mat".
 */
uint32_t isl_mat_get_hash(__isl_keep isl_mat *mat)
{
	int i;
	uint32_t hash;

	if (!mat)
		return 0;

	hash = isl_hash_init();
	isl_hash_byte(hash, mat->n_row & 0xFF);
	isl_hash_byte(hash, mat->n_col & 0xFF);
	for (i = 0; i < mat->n_row; ++i) {
		uint32_t row_hash;

		row_hash = isl_seq_get_hash(mat->row[i], mat->n_col);
		isl_hash_hash(hash, row_hash);
	}

	return hash;
}

__isl_give isl_mat *isl_mat_alloc(isl_ctx *ctx,
	unsigned n_row, unsigned n_col)
{
	int i;
	struct isl_mat *mat;

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_mat_get_hash`, `isl_hash_init`, `isl_hash_byte`, `isl_seq_get_hash` (+2 more); contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_mat_get_hash`, `isl_hash_init`, `isl_hash_byte`, `isl_seq_get_hash` (+2 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 59-90

````c
	mat = isl_alloc_type(ctx, struct isl_mat);
	if (!mat)
		return NULL;

	mat->row = NULL;
	mat->block = isl_blk_alloc(ctx, n_row * n_col);
	if (isl_blk_is_error(mat->block))
		goto error;
	mat->row = isl_calloc_array(ctx, isl_int *, n_row);
	if (n_row && !mat->row)
		goto error;

	if (n_col != 0) {
		for (i = 0; i < n_row; ++i)
			mat->row[i] = mat->block.data + i * n_col;
	}

	mat->ctx = ctx;
	isl_ctx_ref(ctx);
	mat->ref = 1;
	mat->n_row = n_row;
	mat->n_col = n_col;
	mat->max_col = n_col;
	mat->flags = 0;

	return mat;
error:
	isl_blk_free(ctx, mat->block);
	free(mat);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_alloc_type`, `isl_blk_alloc`, `isl_calloc_array`, `isl_ctx_ref` (+2 more); contains control flow with 1 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_alloc_type`, `isl_blk_alloc`, `isl_calloc_array`, `isl_ctx_ref` (+2 more) 相关的例程; 包含控制流结构：1 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 91-120

````c
__isl_give isl_mat *isl_mat_extend(__isl_take isl_mat *mat,
	unsigned n_row, unsigned n_col)
{
	int i;
	isl_int *old;
	isl_int **row;

	if (!mat)
		return NULL;

	if (mat->max_col >= n_col && mat->n_row >= n_row) {
		if (mat->n_col < n_col)
			mat->n_col = n_col;
		return mat;
	}

	if (mat->max_col < n_col) {
		struct isl_mat *new_mat;

		if (n_row < mat->n_row)
			n_row = mat->n_row;
		new_mat = isl_mat_alloc(mat->ctx, n_row, n_col);
		if (!new_mat)
			goto error;
		for (i = 0; i < mat->n_row; ++i)
			isl_seq_cpy(new_mat->row[i], mat->row[i], mat->n_col);
		isl_mat_free(mat);
		return new_mat;
	}

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_mat_extend`, `isl_mat_alloc`, `isl_seq_cpy`, `isl_mat_free`; contains control flow with 1 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_mat_extend`, `isl_mat_alloc`, `isl_seq_cpy`, `isl_mat_free` 相关的例程; 包含控制流结构：1 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 121-147

````c
	mat = isl_mat_cow(mat);
	if (!mat)
		goto error;

	old = mat->block.data;
	mat->block = isl_blk_extend(mat->ctx, mat->block, n_row * mat->max_col);
	if (isl_blk_is_error(mat->block))
		goto error;
	row = isl_realloc_array(mat->ctx, mat->row, isl_int *, n_row);
	if (n_row && !row)
		goto error;
	mat->row = row;

	for (i = 0; i < mat->n_row; ++i)
		mat->row[i] = mat->block.data + (mat->row[i] - old);
	for (i = mat->n_row; i < n_row; ++i)
		mat->row[i] = mat->block.data + i * mat->max_col;
	mat->n_row = n_row;
	if (mat->n_col < n_col)
		mat->n_col = n_col;

	return mat;
error:
	isl_mat_free(mat);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_mat_cow`, `isl_blk_extend`, `isl_realloc_array`, `isl_mat_free`; contains control flow with 2 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_cow`, `isl_blk_extend`, `isl_realloc_array`, `isl_mat_free` 相关的例程; 包含控制流结构：2 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 148-174

````c
__isl_give isl_mat *isl_mat_sub_alloc6(isl_ctx *ctx, isl_int **row,
	unsigned first_row, unsigned n_row, unsigned first_col, unsigned n_col)
{
	int i;
	struct isl_mat *mat;

	mat = isl_alloc_type(ctx, struct isl_mat);
	if (!mat)
		return NULL;
	mat->row = isl_alloc_array(ctx, isl_int *, n_row);
	if (n_row && !mat->row)
		goto error;
	for (i = 0; i < n_row; ++i)
		mat->row[i] = row[first_row+i] + first_col;
	mat->ctx = ctx;
	isl_ctx_ref(ctx);
	mat->ref = 1;
	mat->n_row = n_row;
	mat->n_col = n_col;
	mat->block = isl_blk_empty();
	mat->flags = ISL_MAT_BORROWED;
	return mat;
error:
	free(mat);
	return NULL;
}

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_mat_sub_alloc6`, `isl_alloc_type`, `isl_alloc_array`, `isl_ctx_ref` (+2 more); contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_mat_sub_alloc6`, `isl_alloc_type`, `isl_alloc_array`, `isl_ctx_ref` (+2 more) 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 175-201

````c
__isl_give isl_mat *isl_mat_sub_alloc(__isl_keep isl_mat *mat,
	unsigned first_row, unsigned n_row, unsigned first_col, unsigned n_col)
{
	if (!mat)
		return NULL;
	return isl_mat_sub_alloc6(mat->ctx, mat->row, first_row, n_row,
				  first_col, n_col);
}

void isl_mat_sub_copy(struct isl_ctx *ctx, isl_int **dst, isl_int **src,
	unsigned n_row, unsigned dst_col, unsigned src_col, unsigned n_col)
{
	int i;

	for (i = 0; i < n_row; ++i)
		isl_seq_cpy(dst[i]+dst_col, src[i]+src_col, n_col);
}

void isl_mat_sub_neg(struct isl_ctx *ctx, isl_int **dst, isl_int **src,
	unsigned n_row, unsigned dst_col, unsigned src_col, unsigned n_col)
{
	int i;

	for (i = 0; i < n_row; ++i)
		isl_seq_neg(dst[i]+dst_col, src[i]+src_col, n_col);
}

````
- **EN**: This block declares or defines routines around `isl_mat_sub_alloc`, `isl_mat_sub_copy`, `isl_seq_cpy`, `isl_mat_sub_neg` (+1 more); contains control flow with 2 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_sub_alloc`, `isl_mat_sub_copy`, `isl_seq_cpy`, `isl_mat_sub_neg` (+1 more) 相关的例程; 包含控制流结构：2 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 202-231

````c
__isl_give isl_mat *isl_mat_copy(__isl_keep isl_mat *mat)
{
	if (!mat)
		return NULL;

	mat->ref++;
	return mat;
}

__isl_give isl_mat *isl_mat_dup(__isl_keep isl_mat *mat)
{
	int i;
	struct isl_mat *mat2;

	if (!mat)
		return NULL;
	mat2 = isl_mat_alloc(mat->ctx, mat->n_row, mat->n_col);
	if (!mat2)
		return NULL;
	for (i = 0; i < mat->n_row; ++i)
		isl_seq_cpy(mat2->row[i], mat->row[i], mat->n_col);
	return mat2;
}

__isl_give isl_mat *isl_mat_cow(__isl_take isl_mat *mat)
{
	struct isl_mat *mat2;
	if (!mat)
		return NULL;

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_mat_copy`, `isl_mat_dup`, `isl_mat_alloc`, `isl_seq_cpy` (+1 more); contains control flow with 1 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_mat_copy`, `isl_mat_dup`, `isl_mat_alloc`, `isl_seq_cpy` (+1 more) 相关的例程; 包含控制流结构：1 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 232-261

````c
	if (mat->ref == 1 && !ISL_F_ISSET(mat, ISL_MAT_BORROWED))
		return mat;

	mat2 = isl_mat_dup(mat);
	isl_mat_free(mat);
	return mat2;
}

__isl_null isl_mat *isl_mat_free(__isl_take isl_mat *mat)
{
	if (!mat)
		return NULL;

	if (--mat->ref > 0)
		return NULL;

	if (!ISL_F_ISSET(mat, ISL_MAT_BORROWED))
		isl_blk_free(mat->ctx, mat->block);
	isl_ctx_deref(mat->ctx);
	free(mat->row);
	free(mat);

	return NULL;
}

isl_size isl_mat_rows(__isl_keep isl_mat *mat)
{
	return mat ? mat->n_row : isl_size_error;
}

````
- **EN**: This block declares or defines routines around `isl_mat_dup`, `isl_mat_free`, `isl_blk_free`, `isl_ctx_deref` (+2 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_dup`, `isl_mat_free`, `isl_blk_free`, `isl_ctx_deref` (+2 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 262-290

````c
isl_size isl_mat_cols(__isl_keep isl_mat *mat)
{
	return mat ? mat->n_col : isl_size_error;
}

/* Check that "col" is a valid column position for "mat".
 */
static isl_stat check_col(__isl_keep isl_mat *mat, int col)
{
	if (!mat)
		return isl_stat_error;
	if (col < 0 || col >= mat->n_col)
		isl_die(isl_mat_get_ctx(mat), isl_error_invalid,
			"column out of range", return isl_stat_error);
	return isl_stat_ok;
}

/* Check that "row" is a valid row position for "mat".
 */
static isl_stat check_row(__isl_keep isl_mat *mat, int row)
{
	if (!mat)
		return isl_stat_error;
	if (row < 0 || row >= mat->n_row)
		isl_die(isl_mat_get_ctx(mat), isl_error_invalid,
			"row out of range", return isl_stat_error);
	return isl_stat_ok;
}

````
- **EN**: This block declares or defines routines around `isl_mat_cols`, `check_col`, `isl_die`, `check_row`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_mat_cols`, `check_col`, `isl_die`, `check_row` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 291-318

````c
/* Check that there are "n" columns starting at position "first" in "mat".
 */
static isl_stat check_col_range(__isl_keep isl_mat *mat, unsigned first,
	unsigned n)
{
	if (!mat)
		return isl_stat_error;
	if (first + n > mat->n_col || first + n < first)
		isl_die(isl_mat_get_ctx(mat), isl_error_invalid,
			"column position or range out of bounds",
			return isl_stat_error);
	return isl_stat_ok;
}

/* Check that there are "n" rows starting at position "first" in "mat".
 */
static isl_stat check_row_range(__isl_keep isl_mat *mat, unsigned first,
	unsigned n)
{
	if (!mat)
		return isl_stat_error;
	if (first + n > mat->n_row || first + n < first)
		isl_die(isl_mat_get_ctx(mat), isl_error_invalid,
			"row position or range out of bounds",
			return isl_stat_error);
	return isl_stat_ok;
}

````
- **EN**: This block declares or defines routines around `check_col_range`, `isl_die`, `check_row_range`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `check_col_range`, `isl_die`, `check_row_range` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 319-355

````c
int isl_mat_get_element(__isl_keep isl_mat *mat, int row, int col, isl_int *v)
{
	if (check_row(mat, row) < 0)
		return -1;
	if (check_col(mat, col) < 0)
		return -1;
	isl_int_set(*v, mat->row[row][col]);
	return 0;
}

/* Extract the element at row "row", oolumn "col" of "mat".
 */
__isl_give isl_val *isl_mat_get_element_val(__isl_keep isl_mat *mat,
	int row, int col)
{
	isl_ctx *ctx;

	if (check_row(mat, row) < 0)
		return NULL;
	if (check_col(mat, col) < 0)
		return NULL;
	ctx = isl_mat_get_ctx(mat);
	return isl_val_int_from_isl_int(ctx, mat->row[row][col]);
}

__isl_give isl_mat *isl_mat_set_element(__isl_take isl_mat *mat,
	int row, int col, isl_int v)
{
	mat = isl_mat_cow(mat);
	if (check_row(mat, row) < 0)
		return isl_mat_free(mat);
	if (check_col(mat, col) < 0)
		return isl_mat_free(mat);
	isl_int_set(mat->row[row][col], v);
	return mat;
}

````
- **EN**: This block declares or defines routines around `isl_mat_get_element`, `isl_int_set`, `isl_mat_get_element_val`, `isl_mat_get_ctx` (+2 more); contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_mat_get_element`, `isl_int_set`, `isl_mat_get_element_val`, `isl_mat_get_ctx` (+2 more) 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 356-385

````c
__isl_give isl_mat *isl_mat_set_element_si(__isl_take isl_mat *mat,
	int row, int col, int v)
{
	mat = isl_mat_cow(mat);
	if (check_row(mat, row) < 0)
		return isl_mat_free(mat);
	if (check_col(mat, col) < 0)
		return isl_mat_free(mat);
	isl_int_set_si(mat->row[row][col], v);
	return mat;
}

/* Replace the element at row "row", column "col" of "mat" by "v".
 */
__isl_give isl_mat *isl_mat_set_element_val(__isl_take isl_mat *mat,
	int row, int col, __isl_take isl_val *v)
{
	if (!v)
		return isl_mat_free(mat);
	if (!isl_val_is_int(v))
		isl_die(isl_val_get_ctx(v), isl_error_invalid,
			"expecting integer value", goto error);
	mat = isl_mat_set_element(mat, row, col, v->n);
	isl_val_free(v);
	return mat;
error:
	isl_val_free(v);
	return isl_mat_free(mat);
}

````
- **EN**: This block declares or defines routines around `isl_mat_set_element_si`, `isl_mat_cow`, `isl_int_set_si`, `isl_mat_set_element_val` (+3 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_mat_set_element_si`, `isl_mat_cow`, `isl_int_set_si`, `isl_mat_set_element_val` (+3 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 386-415

````c
__isl_give isl_mat *isl_mat_diag(isl_ctx *ctx, unsigned n_row, isl_int d)
{
	int i;
	struct isl_mat *mat;

	mat = isl_mat_alloc(ctx, n_row, n_row);
	if (!mat)
		return NULL;
	for (i = 0; i < n_row; ++i) {
		isl_seq_clr(mat->row[i], i);
		isl_int_set(mat->row[i][i], d);
		isl_seq_clr(mat->row[i]+i+1, n_row-(i+1));
	}

	return mat;
}

/* Create an "n_row" by "n_col" matrix with zero elements.
 */
__isl_give isl_mat *isl_mat_zero(isl_ctx *ctx, unsigned n_row, unsigned n_col)
{
	int i;
	isl_mat *mat;

	mat = isl_mat_alloc(ctx, n_row, n_col);
	if (!mat)
		return NULL;
	for (i = 0; i < n_row; ++i)
		isl_seq_clr(mat->row[i], n_col);

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_mat_diag`, `isl_mat_alloc`, `isl_seq_clr`, `isl_int_set` (+1 more); contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_mat_diag`, `isl_mat_alloc`, `isl_seq_clr`, `isl_int_set` (+1 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 416-446

````c
	return mat;
}

__isl_give isl_mat *isl_mat_identity(isl_ctx *ctx, unsigned n_row)
{
	if (!ctx)
		return NULL;
	return isl_mat_diag(ctx, n_row, ctx->one);
}

/* Is "mat" a (possibly scaled) identity matrix?
 */
isl_bool isl_mat_is_scaled_identity(__isl_keep isl_mat *mat)
{
	int i;

	if (!mat)
		return isl_bool_error;
	if (mat->n_row != mat->n_col)
		return isl_bool_false;

	for (i = 0; i < mat->n_row; ++i) {
		if (isl_seq_any_non_zero(mat->row[i], i))
			return isl_bool_false;
		if (isl_int_ne(mat->row[0][0], mat->row[i][i]))
			return isl_bool_false;
		if (isl_seq_any_non_zero(mat->row[i] + i + 1,
					    mat->n_col - (i + 1)))
			return isl_bool_false;
	}

````
- **EN**: This block declares or defines routines around `isl_mat_identity`, `a`, `isl_mat_is_scaled_identity`; contains control flow with 1 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_mat_identity`, `a`, `isl_mat_is_scaled_identity` 相关的例程; 包含控制流结构：1 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 447-476

````c
	return isl_bool_true;
}

__isl_give isl_vec *isl_mat_vec_product(__isl_take isl_mat *mat,
	__isl_take isl_vec *vec)
{
	int i;
	struct isl_vec *prod;

	if (!mat || !vec)
		goto error;

	isl_assert(mat->ctx, mat->n_col == vec->size, goto error);

	prod = isl_vec_alloc(mat->ctx, mat->n_row);
	if (!prod)
		goto error;

	for (i = 0; i < prod->size; ++i)
		isl_seq_inner_product(mat->row[i], vec->el, vec->size,
					&prod->block.data[i]);
	isl_mat_free(mat);
	isl_vec_free(vec);
	return prod;
error:
	isl_mat_free(mat);
	isl_vec_free(vec);
	return NULL;
}

````
- **EN**: This block declares or references types such as `isl_vec`; declares or defines routines around `isl_mat_vec_product`, `isl_assert`, `isl_vec_alloc`, `isl_seq_inner_product` (+2 more); contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_vec`; 声明或定义与 `isl_mat_vec_product`, `isl_assert`, `isl_vec_alloc`, `isl_seq_inner_product` (+2 more) 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 477-505

````c
__isl_give isl_vec *isl_mat_vec_inverse_product(__isl_take isl_mat *mat,
	__isl_take isl_vec *vec)
{
	struct isl_mat *vec_mat;
	int i;

	if (!mat || !vec)
		goto error;
	vec_mat = isl_mat_alloc(vec->ctx, vec->size, 1);
	if (!vec_mat)
		goto error;
	for (i = 0; i < vec->size; ++i)
		isl_int_set(vec_mat->row[i][0], vec->el[i]);
	vec_mat = isl_mat_inverse_product(mat, vec_mat);
	isl_vec_free(vec);
	if (!vec_mat)
		return NULL;
	vec = isl_vec_alloc(vec_mat->ctx, vec_mat->n_row);
	if (vec)
		for (i = 0; i < vec->size; ++i)
			isl_int_set(vec->el[i], vec_mat->row[i][0]);
	isl_mat_free(vec_mat);
	return vec;
error:
	isl_mat_free(mat);
	isl_vec_free(vec);
	return NULL;
}

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_mat_vec_inverse_product`, `isl_mat_alloc`, `isl_int_set`, `isl_mat_inverse_product` (+3 more); contains control flow with 2 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_mat_vec_inverse_product`, `isl_mat_alloc`, `isl_int_set`, `isl_mat_inverse_product` (+3 more) 相关的例程; 包含控制流结构：2 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 506-534

````c
__isl_give isl_vec *isl_vec_mat_product(__isl_take isl_vec *vec,
	__isl_take isl_mat *mat)
{
	int i, j;
	struct isl_vec *prod;

	if (!mat || !vec)
		goto error;

	isl_assert(mat->ctx, mat->n_row == vec->size, goto error);

	prod = isl_vec_alloc(mat->ctx, mat->n_col);
	if (!prod)
		goto error;

	for (i = 0; i < prod->size; ++i) {
		isl_int_set_si(prod->el[i], 0);
		for (j = 0; j < vec->size; ++j)
			isl_int_addmul(prod->el[i], vec->el[j], mat->row[j][i]);
	}
	isl_mat_free(mat);
	isl_vec_free(vec);
	return prod;
error:
	isl_mat_free(mat);
	isl_vec_free(vec);
	return NULL;
}

````
- **EN**: This block declares or references types such as `isl_vec`; declares or defines routines around `isl_vec_mat_product`, `isl_assert`, `isl_vec_alloc`, `isl_int_set_si` (+3 more); contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_vec`; 声明或定义与 `isl_vec_mat_product`, `isl_assert`, `isl_vec_alloc`, `isl_int_set_si` (+3 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 535-561

````c
__isl_give isl_mat *isl_mat_aff_direct_sum(__isl_take isl_mat *left,
	__isl_take isl_mat *right)
{
	int i;
	struct isl_mat *sum;

	if (!left || !right)
		goto error;

	isl_assert(left->ctx, left->n_row == right->n_row, goto error);
	isl_assert(left->ctx, left->n_row >= 1, goto error);
	isl_assert(left->ctx, left->n_col >= 1, goto error);
	isl_assert(left->ctx, right->n_col >= 1, goto error);
	isl_assert(left->ctx,
	    !isl_seq_any_non_zero(left->row[0]+1, left->n_col-1),
	    goto error);
	isl_assert(left->ctx,
	    !isl_seq_any_non_zero(right->row[0]+1, right->n_col-1),
	    goto error);

	sum = isl_mat_alloc(left->ctx, left->n_row, left->n_col + right->n_col - 1);
	if (!sum)
		goto error;
	isl_int_lcm(sum->row[0][0], left->row[0][0], right->row[0][0]);
	isl_int_divexact(left->row[0][0], sum->row[0][0], left->row[0][0]);
	isl_int_divexact(right->row[0][0], sum->row[0][0], right->row[0][0]);

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_mat_aff_direct_sum`, `isl_assert`, `isl_seq_any_non_zero`, `isl_mat_alloc` (+2 more); contains control flow with 2 conditional check(s); adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_mat_aff_direct_sum`, `isl_assert`, `isl_seq_any_non_zero`, `isl_mat_alloc` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 562-598

````c
	isl_seq_clr(sum->row[0]+1, sum->n_col-1);
	for (i = 1; i < sum->n_row; ++i) {
		isl_int_mul(sum->row[i][0], left->row[0][0], left->row[i][0]);
		isl_int_addmul(sum->row[i][0],
				right->row[0][0], right->row[i][0]);
		isl_seq_scale(sum->row[i]+1, left->row[i]+1, left->row[0][0],
				left->n_col-1);
		isl_seq_scale(sum->row[i]+left->n_col,
				right->row[i]+1, right->row[0][0],
				right->n_col-1);
	}

	isl_int_divexact(left->row[0][0], sum->row[0][0], left->row[0][0]);
	isl_int_divexact(right->row[0][0], sum->row[0][0], right->row[0][0]);
	isl_mat_free(left);
	isl_mat_free(right);
	return sum;
error:
	isl_mat_free(left);
	isl_mat_free(right);
	return NULL;
}

static void exchange(__isl_keep isl_mat *M, __isl_keep isl_mat **U,
	__isl_keep isl_mat **Q, unsigned row, unsigned i, unsigned j)
{
	int r;
	for (r = row; r < M->n_row; ++r)
		isl_int_swap(M->row[r][i], M->row[r][j]);
	if (U) {
		for (r = 0; r < (*U)->n_row; ++r)
			isl_int_swap((*U)->row[r][i], (*U)->row[r][j]);
	}
	if (Q)
		isl_mat_swap_rows(*Q, i, j);
}

````
- **EN**: This block declares or defines routines around `isl_seq_clr`, `isl_int_mul`, `isl_int_addmul`, `isl_seq_scale` (+5 more); contains control flow with 3 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_seq_clr`, `isl_int_mul`, `isl_int_addmul`, `isl_seq_scale` (+5 more) 相关的例程; 包含控制流结构：3 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 599-628

````c
static void subtract(__isl_keep isl_mat *M, __isl_keep isl_mat **U,
	__isl_keep isl_mat **Q, unsigned row, unsigned i, unsigned j, isl_int m)
{
	int r;
	for (r = row; r < M->n_row; ++r)
		isl_int_submul(M->row[r][j], m, M->row[r][i]);
	if (U) {
		for (r = 0; r < (*U)->n_row; ++r)
			isl_int_submul((*U)->row[r][j], m, (*U)->row[r][i]);
	}
	if (Q) {
		for (r = 0; r < (*Q)->n_col; ++r)
			isl_int_addmul((*Q)->row[i][r], m, (*Q)->row[j][r]);
	}
}

static void oppose(__isl_keep isl_mat *M, __isl_keep isl_mat **U,
	__isl_keep isl_mat **Q, unsigned row, unsigned col)
{
	int r;
	for (r = row; r < M->n_row; ++r)
		isl_int_neg(M->row[r][col], M->row[r][col]);
	if (U) {
		for (r = 0; r < (*U)->n_row; ++r)
			isl_int_neg((*U)->row[r][col], (*U)->row[r][col]);
	}
	if (Q)
		isl_seq_neg((*Q)->row[col], (*Q)->row[col], (*Q)->n_col);
}

````
- **EN**: This block declares or defines routines around `subtract`, `isl_int_submul`, `isl_int_addmul`, `oppose` (+2 more); contains control flow with 5 loop construct(s), 4 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `subtract`, `isl_int_submul`, `isl_int_addmul`, `oppose` (+2 more) 相关的例程; 包含控制流结构：5 处循环、4 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 629-663

````c
/* Given matrix M, compute
 *
 *		M U = H
 *		M   = H Q
 *
 * with U and Q unimodular matrices and H a matrix in column echelon form
 * such that on each echelon row the entries in the non-echelon column
 * are non-negative (if neg == 0) or non-positive (if neg == 1)
 * and strictly smaller (in absolute value) than the entries in the echelon
 * column.
 * If U or Q are NULL, then these matrices are not computed.
 */
__isl_give isl_mat *isl_mat_left_hermite(__isl_take isl_mat *M, int neg,
	__isl_give isl_mat **U, __isl_give isl_mat **Q)
{
	isl_int c;
	int row, col;

	if (U)
		*U = NULL;
	if (Q)
		*Q = NULL;
	if (!M)
		goto error;
	if (U) {
		*U = isl_mat_identity(M->ctx, M->n_col);
		if (!*U)
			goto error;
	}
	if (Q) {
		*Q = isl_mat_identity(M->ctx, M->n_col);
		if (!*Q)
			goto error;
	}

````
- **EN**: This block declares or defines routines around `negative`, `smaller`, `isl_mat_left_hermite`, `isl_mat_identity`; contains control flow with 7 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `negative`, `smaller`, `isl_mat_left_hermite`, `isl_mat_identity` 相关的例程; 包含控制流结构：7 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 664-695

````c
	if (M->n_col == 0)
		return M;

	M = isl_mat_cow(M);
	if (!M)
		goto error;

	col = 0;
	isl_int_init(c);
	for (row = 0; row < M->n_row; ++row) {
		int first, i, off;
		first = isl_seq_abs_min_non_zero(M->row[row]+col, M->n_col-col);
		if (first == -1)
			continue;
		first += col;
		if (first != col)
			exchange(M, U, Q, row, first, col);
		if (isl_int_is_neg(M->row[row][col]))
			oppose(M, U, Q, row, col);
		first = col+1;
		while ((off = isl_seq_first_non_zero(M->row[row]+first,
						       M->n_col-first)) != -1) {
			first += off;
			isl_int_fdiv_q(c, M->row[row][first], M->row[row][col]);
			subtract(M, U, Q, row, col, first, c);
			if (!isl_int_is_zero(M->row[row][first]))
				exchange(M, U, Q, row, first, col);
			else
				++first;
		}
		for (i = 0; i < col; ++i) {
			if (isl_int_is_zero(M->row[row][i]))
````
- **EN**: This block declares or defines routines around `isl_mat_cow`, `isl_int_init`, `isl_seq_abs_min_non_zero`, `exchange` (+3 more); contains control flow with 3 loop construct(s), 7 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_cow`, `isl_int_init`, `isl_seq_abs_min_non_zero`, `exchange` (+3 more) 相关的例程; 包含控制流结构：3 处循环、7 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 696-722

````c
				continue;
			if (neg)
				isl_int_cdiv_q(c, M->row[row][i], M->row[row][col]);
			else
				isl_int_fdiv_q(c, M->row[row][i], M->row[row][col]);
			if (isl_int_is_zero(c))
				continue;
			subtract(M, U, Q, row, col, i, c);
		}
		++col;
	}
	isl_int_clear(c);

	return M;
error:
	if (Q) {
		isl_mat_free(*Q);
		*Q = NULL;
	}
	if (U) {
		isl_mat_free(*U);
		*U = NULL;
	}
	isl_mat_free(M);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_int_cdiv_q`, `isl_int_fdiv_q`, `subtract`, `isl_int_clear` (+1 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_int_cdiv_q`, `isl_int_fdiv_q`, `subtract`, `isl_int_clear` (+1 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 723-749

````c
/* Use row "row" of "mat" to eliminate column "col" from all other rows.
 */
static __isl_give isl_mat *eliminate(__isl_take isl_mat *mat, int row, int col)
{
	int k;
	isl_size nr, nc;
	isl_ctx *ctx;

	nr = isl_mat_rows(mat);
	nc = isl_mat_cols(mat);
	if (nr < 0 || nc < 0)
		return isl_mat_free(mat);

	ctx = isl_mat_get_ctx(mat);

	for (k = 0; k < nr; ++k) {
		if (k == row)
			continue;
		if (isl_int_is_zero(mat->row[k][col]))
			continue;
		mat = isl_mat_cow(mat);
		if (!mat)
			return NULL;
		isl_seq_elim(mat->row[k], mat->row[row], col, nc, NULL);
		isl_seq_normalize(ctx, mat->row[k], nc);
	}

````
- **EN**: This block declares or defines routines around `eliminate`, `isl_mat_rows`, `isl_mat_cols`, `isl_mat_get_ctx` (+3 more); contains control flow with 1 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `eliminate`, `isl_mat_rows`, `isl_mat_cols`, `isl_mat_get_ctx` (+3 more) 相关的例程; 包含控制流结构：1 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 750-781

````c
	return mat;
}

/* Perform Gaussian elimination on the rows of "mat", but start
 * from the final row and the final column.
 * Any zero rows that result from the elimination are removed.
 *
 * In particular, for each column from last to first,
 * look for the last row with a non-zero coefficient in that column,
 * move it last (but before other rows moved last in previous steps) and
 * use it to eliminate the column from the other rows.
 */
__isl_give isl_mat *isl_mat_reverse_gauss(__isl_take isl_mat *mat)
{
	int k, row, last;
	isl_size nr, nc;

	nr = isl_mat_rows(mat);
	nc = isl_mat_cols(mat);
	if (nr < 0 || nc < 0)
		return isl_mat_free(mat);

	last = nc - 1;
	for (row = nr - 1; row >= 0; --row) {
		for (; last >= 0; --last) {
			for (k = row; k >= 0; --k)
				if (!isl_int_is_zero(mat->row[k][last]))
					break;
			if (k >= 0)
				break;
		}
		if (last < 0)
````
- **EN**: This block declares or defines routines around `last`, `isl_mat_reverse_gauss`, `isl_mat_rows`, `isl_mat_cols`; contains control flow with 3 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `last`, `isl_mat_reverse_gauss`, `isl_mat_rows`, `isl_mat_cols` 相关的例程; 包含控制流结构：3 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 782-810

````c
			break;
		if (k != row)
			mat = isl_mat_swap_rows(mat, k, row);
		if (!mat)
			return NULL;
		if (isl_int_is_neg(mat->row[row][last]))
			mat = isl_mat_row_neg(mat, row);
		mat = eliminate(mat, row, last);
		if (!mat)
			return NULL;
	}
	mat = isl_mat_drop_rows(mat, 0, row + 1);

	return mat;
}

/* Negate the lexicographically negative rows of "mat" such that
 * all rows in the result are lexicographically non-negative.
 */
__isl_give isl_mat *isl_mat_lexnonneg_rows(__isl_take isl_mat *mat)
{
	int i;
	isl_size nr, nc;

	nr = isl_mat_rows(mat);
	nc = isl_mat_cols(mat);
	if (nr < 0 || nc < 0)
		return isl_mat_free(mat);

````
- **EN**: This block declares or defines routines around `isl_mat_swap_rows`, `isl_mat_row_neg`, `eliminate`, `isl_mat_drop_rows` (+3 more); contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_mat_swap_rows`, `isl_mat_row_neg`, `eliminate`, `isl_mat_drop_rows` (+3 more) 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 811-840

````c
	for (i = 0; i < nr; ++i) {
		int pos;

		pos = isl_seq_first_non_zero(mat->row[i], nc);
		if (pos < 0)
			continue;
		if (isl_int_is_nonneg(mat->row[i][pos]))
			continue;
		mat = isl_mat_row_neg(mat, i);
		if (!mat)
			return NULL;
	}

	return mat;
}

/* Given a matrix "H" is column echelon form, what is the first
 * zero column?  That is how many initial columns are non-zero?
 * Start looking at column "first_col" and only consider
 * the columns to be of size "n_row".
 * "H" is assumed to be non-NULL.
 *
 * Since "H" is in column echelon form, the first non-zero entry
 * in a column is always in a later position compared to the previous column.
 */
static int hermite_first_zero_col(__isl_keep isl_mat *H, int first_col,
	int n_row)
{
	int row, col;

````
- **EN**: This block declares or defines routines around `isl_seq_first_non_zero`, `isl_mat_row_neg`, `hermite_first_zero_col`; contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_seq_first_non_zero`, `isl_mat_row_neg`, `hermite_first_zero_col` 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 841-868

````c
	for (col = first_col, row = 0; col < H->n_col; ++col) {
		for (; row < n_row; ++row)
			if (!isl_int_is_zero(H->row[row][col]))
				break;
		if (row == n_row)
			return col;
	}

	return H->n_col;
}

/* Return the rank of "mat", or isl_size_error in case of error.
 */
isl_size isl_mat_rank(__isl_keep isl_mat *mat)
{
	int rank;
	isl_mat *H;

	H = isl_mat_left_hermite(isl_mat_copy(mat), 0, NULL, NULL);
	if (!H)
		return isl_size_error;

	rank = hermite_first_zero_col(H, 0, H->n_row);
	isl_mat_free(H);

	return rank;
}

````
- **EN**: This block declares or defines routines around `isl_mat_rank`, `isl_mat_left_hermite`, `hermite_first_zero_col`, `isl_mat_free`; contains control flow with 2 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_mat_rank`, `isl_mat_left_hermite`, `hermite_first_zero_col`, `isl_mat_free` 相关的例程; 包含控制流结构：2 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 869-897

````c
__isl_give isl_mat *isl_mat_right_kernel(__isl_take isl_mat *mat)
{
	int rank;
	struct isl_mat *U = NULL;
	struct isl_mat *K;

	mat = isl_mat_left_hermite(mat, 0, &U, NULL);
	if (!mat || !U)
		goto error;

	rank = hermite_first_zero_col(mat, 0, mat->n_row);
	K = isl_mat_alloc(U->ctx, U->n_row, U->n_col - rank);
	if (!K)
		goto error;
	isl_mat_sub_copy(K->ctx, K->row, U->row, U->n_row, 0, rank, U->n_col-rank);
	isl_mat_free(mat);
	isl_mat_free(U);
	return K;
error:
	isl_mat_free(mat);
	isl_mat_free(U);
	return NULL;
}

__isl_give isl_mat *isl_mat_lin_to_aff(__isl_take isl_mat *mat)
{
	int i;
	struct isl_mat *mat2;

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_mat_right_kernel`, `isl_mat_left_hermite`, `hermite_first_zero_col`, `isl_mat_alloc` (+3 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_mat_right_kernel`, `isl_mat_left_hermite`, `hermite_first_zero_col`, `isl_mat_alloc` (+3 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 898-926

````c
	if (!mat)
		return NULL;
	mat2 = isl_mat_alloc(mat->ctx, 1+mat->n_row, 1+mat->n_col);
	if (!mat2)
		goto error;
	isl_int_set_si(mat2->row[0][0], 1);
	isl_seq_clr(mat2->row[0]+1, mat->n_col);
	for (i = 0; i < mat->n_row; ++i) {
		isl_int_set_si(mat2->row[1+i][0], 0);
		isl_seq_cpy(mat2->row[1+i]+1, mat->row[i], mat->n_col);
	}
	isl_mat_free(mat);
	return mat2;
error:
	isl_mat_free(mat);
	return NULL;
}

/* Given two matrices M1 and M2, return the block matrix
 *
 *	[ M1  0  ]
 *	[ 0   M2 ]
 */
__isl_give isl_mat *isl_mat_diagonal(__isl_take isl_mat *mat1,
	__isl_take isl_mat *mat2)
{
	int i;
	isl_mat *mat;

````
- **EN**: This block declares or defines routines around `isl_mat_alloc`, `isl_int_set_si`, `isl_seq_clr`, `isl_seq_cpy` (+2 more); contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_mat_alloc`, `isl_int_set_si`, `isl_seq_clr`, `isl_seq_cpy` (+2 more) 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 927-955

````c
	if (!mat1 || !mat2)
		goto error;

	mat = isl_mat_alloc(mat1->ctx, mat1->n_row + mat2->n_row,
				       mat1->n_col + mat2->n_col);
	if (!mat)
		goto error;
	for (i = 0; i < mat1->n_row; ++i) {
		isl_seq_cpy(mat->row[i], mat1->row[i], mat1->n_col);
		isl_seq_clr(mat->row[i] + mat1->n_col, mat2->n_col);
	}
	for (i = 0; i < mat2->n_row; ++i) {
		isl_seq_clr(mat->row[mat1->n_row + i], mat1->n_col);
		isl_seq_cpy(mat->row[mat1->n_row + i] + mat1->n_col,
						    mat2->row[i], mat2->n_col);
	}
	isl_mat_free(mat1);
	isl_mat_free(mat2);
	return mat;
error:
	isl_mat_free(mat1);
	isl_mat_free(mat2);
	return NULL;
}

static int row_first_non_zero(isl_int **row, unsigned n_row, unsigned col)
{
	int i;

````
- **EN**: This block declares or defines routines around `isl_mat_alloc`, `isl_seq_cpy`, `isl_seq_clr`, `isl_mat_free` (+1 more); contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_alloc`, `isl_seq_cpy`, `isl_seq_clr`, `isl_mat_free` (+1 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 956-981

````c
	for (i = 0; i < n_row; ++i)
		if (!isl_int_is_zero(row[i][col]))
			return i;
	return -1;
}

static int row_abs_min_non_zero(isl_int **row, unsigned n_row, unsigned col)
{
	int i, min = row_first_non_zero(row, n_row, col);
	if (min < 0)
		return -1;
	for (i = min + 1; i < n_row; ++i) {
		if (isl_int_is_zero(row[i][col]))
			continue;
		if (isl_int_abs_lt(row[i][col], row[min][col]))
			min = i;
	}
	return min;
}

static isl_stat inv_exchange(__isl_keep isl_mat **left,
	__isl_keep isl_mat **right, unsigned i, unsigned j)
{
	*left = isl_mat_swap_rows(*left, i, j);
	*right = isl_mat_swap_rows(*right, i, j);

````
- **EN**: This block declares or defines routines around `row_abs_min_non_zero`, `row_first_non_zero`, `inv_exchange`, `isl_mat_swap_rows`; contains control flow with 2 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `row_abs_min_non_zero`, `row_first_non_zero`, `inv_exchange`, `isl_mat_swap_rows` 相关的例程; 包含控制流结构：2 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 982-1013

````c
	if (!*left || !*right)
		return isl_stat_error;
	return isl_stat_ok;
}

static void inv_oppose(
	__isl_keep isl_mat *left, __isl_keep isl_mat *right, unsigned row)
{
	isl_seq_neg(left->row[row]+row, left->row[row]+row, left->n_col-row);
	isl_seq_neg(right->row[row], right->row[row], right->n_col);
}

static void inv_subtract(__isl_keep isl_mat *left, __isl_keep isl_mat *right,
	unsigned row, unsigned i, isl_int m)
{
	isl_int_neg(m, m);
	isl_seq_combine(left->row[i]+row,
			left->ctx->one, left->row[i]+row,
			m, left->row[row]+row,
			left->n_col-row);
	isl_seq_combine(right->row[i], right->ctx->one, right->row[i],
			m, right->row[row], right->n_col);
}

/* Compute inv(left)*right
 */
__isl_give isl_mat *isl_mat_inverse_product(__isl_take isl_mat *left,
	__isl_take isl_mat *right)
{
	int row;
	isl_int a, b;

````
- **EN**: This block declares or defines routines around `inv_oppose`, `isl_seq_neg`, `inv_subtract`, `isl_int_neg` (+3 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `inv_oppose`, `isl_seq_neg`, `inv_subtract`, `isl_int_neg` (+3 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1014-1045

````c
	if (!left || !right)
		goto error;

	isl_assert(left->ctx, left->n_row == left->n_col, goto error);
	isl_assert(left->ctx, left->n_row == right->n_row, goto error);

	if (left->n_row == 0) {
		isl_mat_free(left);
		return right;
	}

	left = isl_mat_cow(left);
	right = isl_mat_cow(right);
	if (!left || !right)
		goto error;

	isl_int_init(a);
	isl_int_init(b);
	for (row = 0; row < left->n_row; ++row) {
		int pivot, first, i, off;
		pivot = row_abs_min_non_zero(left->row+row, left->n_row-row, row);
		if (pivot < 0) {
			isl_int_clear(a);
			isl_int_clear(b);
			isl_assert(left->ctx, pivot >= 0, goto error);
		}
		pivot += row;
		if (pivot != row)
			if (inv_exchange(&left, &right, pivot, row) < 0)
				goto error;
		if (isl_int_is_neg(left->row[row][row]))
			inv_oppose(left, right, row);
````
- **EN**: This block declares or defines routines around `isl_assert`, `isl_mat_free`, `isl_mat_cow`, `isl_int_init` (+3 more); contains control flow with 1 loop construct(s), 7 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_assert`, `isl_mat_free`, `isl_mat_cow`, `isl_int_init` (+3 more) 相关的例程; 包含控制流结构：1 处循环、7 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 1046-1076

````c
		first = row+1;
		while ((off = row_first_non_zero(left->row+first,
					left->n_row-first, row)) != -1) {
			first += off;
			isl_int_fdiv_q(a, left->row[first][row],
					left->row[row][row]);
			inv_subtract(left, right, row, first, a);
			if (!isl_int_is_zero(left->row[first][row])) {
				if (inv_exchange(&left, &right, row, first) < 0)
					goto error;
			} else {
				++first;
			}
		}
		for (i = 0; i < row; ++i) {
			if (isl_int_is_zero(left->row[i][row]))
				continue;
			isl_int_gcd(a, left->row[row][row], left->row[i][row]);
			isl_int_divexact(b, left->row[i][row], a);
			isl_int_divexact(a, left->row[row][row], a);
			isl_int_neg(b, b);
			isl_seq_combine(left->row[i] + i,
					a, left->row[i] + i,
					b, left->row[row] + i,
					left->n_col - i);
			isl_seq_combine(right->row[i], a, right->row[i],
					b, right->row[row], right->n_col);
		}
	}
	isl_int_clear(b);

````
- **EN**: This block declares or defines routines around `isl_int_fdiv_q`, `inv_subtract`, `isl_int_gcd`, `isl_int_divexact` (+3 more); contains control flow with 2 loop construct(s), 3 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_int_fdiv_q`, `inv_subtract`, `isl_int_gcd`, `isl_int_divexact` (+3 more) 相关的例程; 包含控制流结构：2 处循环、3 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1077-1104

````c
	isl_int_set(a, left->row[0][0]);
	for (row = 1; row < left->n_row; ++row)
		isl_int_lcm(a, a, left->row[row][row]);
	if (isl_int_is_zero(a)){
		isl_int_clear(a);
		isl_assert(left->ctx, 0, goto error);
	}
	for (row = 0; row < left->n_row; ++row) {
		isl_int_divexact(left->row[row][row], a, left->row[row][row]);
		if (isl_int_is_one(left->row[row][row]))
			continue;
		isl_seq_scale(right->row[row], right->row[row],
				left->row[row][row], right->n_col);
	}
	isl_int_clear(a);

	isl_mat_free(left);
	return right;
error:
	isl_mat_free(left);
	isl_mat_free(right);
	return NULL;
}

void isl_mat_col_scale(__isl_keep isl_mat *mat, unsigned col, isl_int m)
{
	int i;

````
- **EN**: This block declares or defines routines around `isl_int_set`, `isl_int_lcm`, `isl_int_clear`, `isl_assert` (+4 more); contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_int_set`, `isl_int_lcm`, `isl_int_clear`, `isl_assert` (+4 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 1105-1133

````c
	for (i = 0; i < mat->n_row; ++i)
		isl_int_mul(mat->row[i][col], mat->row[i][col], m);
}

void isl_mat_col_combine(__isl_keep isl_mat *mat, unsigned dst,
	isl_int m1, unsigned src1, isl_int m2, unsigned src2)
{
	int i;
	isl_int tmp;

	isl_int_init(tmp);
	for (i = 0; i < mat->n_row; ++i) {
		isl_int_mul(tmp, m1, mat->row[i][src1]);
		isl_int_addmul(tmp, m2, mat->row[i][src2]);
		isl_int_set(mat->row[i][dst], tmp);
	}
	isl_int_clear(tmp);
}

__isl_give isl_mat *isl_mat_right_inverse(__isl_take isl_mat *mat)
{
	struct isl_mat *inv;
	int row;
	isl_int a, b;

	mat = isl_mat_cow(mat);
	if (!mat)
		return NULL;

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_int_mul`, `isl_mat_col_combine`, `isl_int_init`, `isl_int_addmul` (+4 more); contains control flow with 2 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_int_mul`, `isl_mat_col_combine`, `isl_int_init`, `isl_int_addmul` (+4 more) 相关的例程; 包含控制流结构：2 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1134-1165

````c
	inv = isl_mat_identity(mat->ctx, mat->n_col);
	inv = isl_mat_cow(inv);
	if (!inv)
		goto error;

	isl_int_init(a);
	isl_int_init(b);
	for (row = 0; row < mat->n_row; ++row) {
		int pivot, first, i, off;
		pivot = isl_seq_abs_min_non_zero(mat->row[row]+row, mat->n_col-row);
		if (pivot < 0) {
			isl_int_clear(a);
			isl_int_clear(b);
			isl_assert(mat->ctx, pivot >= 0, goto error);
		}
		pivot += row;
		if (pivot != row)
			exchange(mat, &inv, NULL, row, pivot, row);
		if (isl_int_is_neg(mat->row[row][row]))
			oppose(mat, &inv, NULL, row, row);
		first = row+1;
		while ((off = isl_seq_first_non_zero(mat->row[row]+first,
						    mat->n_col-first)) != -1) {
			first += off;
			isl_int_fdiv_q(a, mat->row[row][first],
						    mat->row[row][row]);
			subtract(mat, &inv, NULL, row, row, first, a);
			if (!isl_int_is_zero(mat->row[row][first]))
				exchange(mat, &inv, NULL, row, row, first);
			else
				++first;
		}
````
- **EN**: This block declares or defines routines around `isl_mat_identity`, `isl_mat_cow`, `isl_int_init`, `isl_seq_abs_min_non_zero` (+6 more); contains control flow with 2 loop construct(s), 5 conditional check(s); adds defensive checks for invariants or impossible states; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_identity`, `isl_mat_cow`, `isl_int_init`, `isl_seq_abs_min_non_zero` (+6 more) 相关的例程; 包含控制流结构：2 处循环、5 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1166-1193

````c
		for (i = 0; i < row; ++i) {
			if (isl_int_is_zero(mat->row[row][i]))
				continue;
			isl_int_gcd(a, mat->row[row][row], mat->row[row][i]);
			isl_int_divexact(b, mat->row[row][i], a);
			isl_int_divexact(a, mat->row[row][row], a);
			isl_int_neg(a, a);
			isl_mat_col_combine(mat, i, a, i, b, row);
			isl_mat_col_combine(inv, i, a, i, b, row);
		}
	}
	isl_int_clear(b);

	isl_int_set(a, mat->row[0][0]);
	for (row = 1; row < mat->n_row; ++row)
		isl_int_lcm(a, a, mat->row[row][row]);
	if (isl_int_is_zero(a)){
		isl_int_clear(a);
		goto error;
	}
	for (row = 0; row < mat->n_row; ++row) {
		isl_int_divexact(mat->row[row][row], a, mat->row[row][row]);
		if (isl_int_is_one(mat->row[row][row]))
			continue;
		isl_mat_col_scale(inv, row, mat->row[row][row]);
	}
	isl_int_clear(a);

````
- **EN**: This block declares or defines routines around `isl_int_gcd`, `isl_int_divexact`, `isl_int_neg`, `isl_mat_col_combine` (+4 more); contains control flow with 3 loop construct(s), 3 conditional check(s); uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_int_gcd`, `isl_int_divexact`, `isl_int_neg`, `isl_mat_col_combine` (+4 more) 相关的例程; 包含控制流结构：3 处循环、3 处条件判断; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1194-1225

````c
	isl_mat_free(mat);

	return inv;
error:
	isl_mat_free(mat);
	isl_mat_free(inv);
	return NULL;
}

__isl_give isl_mat *isl_mat_transpose(__isl_take isl_mat *mat)
{
	struct isl_mat *transpose = NULL;
	int i, j;

	if (!mat)
		return NULL;

	if (mat->n_col == mat->n_row) {
		mat = isl_mat_cow(mat);
		if (!mat)
			return NULL;
		for (i = 0; i < mat->n_row; ++i)
			for (j = i + 1; j < mat->n_col; ++j)
				isl_int_swap(mat->row[i][j], mat->row[j][i]);
		return mat;
	}
	transpose = isl_mat_alloc(mat->ctx, mat->n_col, mat->n_row);
	if (!transpose)
		goto error;
	for (i = 0; i < mat->n_row; ++i)
		for (j = 0; j < mat->n_col; ++j)
			isl_int_set(transpose->row[j][i], mat->row[i][j]);
````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_mat_free`, `isl_mat_transpose`, `isl_mat_cow`, `isl_int_swap` (+2 more); contains control flow with 4 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_mat_free`, `isl_mat_transpose`, `isl_mat_cow`, `isl_int_swap` (+2 more) 相关的例程; 包含控制流结构：4 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1226-1252

````c
	isl_mat_free(mat);
	return transpose;
error:
	isl_mat_free(mat);
	return NULL;
}

__isl_give isl_mat *isl_mat_swap_cols(__isl_take isl_mat *mat,
	unsigned i, unsigned j)
{
	int r;

	mat = isl_mat_cow(mat);
	if (check_col_range(mat, i, 1) < 0 ||
	    check_col_range(mat, j, 1) < 0)
		return isl_mat_free(mat);

	for (r = 0; r < mat->n_row; ++r)
		isl_int_swap(mat->row[r][i], mat->row[r][j]);
	return mat;
}

__isl_give isl_mat *isl_mat_swap_rows(__isl_take isl_mat *mat,
	unsigned i, unsigned j)
{
	isl_int *t;

````
- **EN**: This block declares or defines routines around `isl_mat_free`, `isl_mat_swap_cols`, `isl_mat_cow`, `check_col_range` (+2 more); contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_free`, `isl_mat_swap_cols`, `isl_mat_cow`, `check_col_range` (+2 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1253-1284

````c
	if (!mat)
		return NULL;
	mat = isl_mat_cow(mat);
	if (check_row_range(mat, i, 1) < 0 ||
	    check_row_range(mat, j, 1) < 0)
		return isl_mat_free(mat);

	t = mat->row[i];
	mat->row[i] = mat->row[j];
	mat->row[j] = t;
	return mat;
}

/* Calculate the product of two matrices.
 *
 * This function is optimized for operand matrices that contain many zeros and
 * skips multiplications where we know one of the operands is zero.
 */
__isl_give isl_mat *isl_mat_product(__isl_take isl_mat *left,
	__isl_take isl_mat *right)
{
	int i, j, k;
	struct isl_mat *prod;

	if (!left || !right)
		goto error;
	isl_assert(left->ctx, left->n_col == right->n_row, goto error);
	prod = isl_mat_alloc(left->ctx, left->n_row, right->n_col);
	if (!prod)
		goto error;
	if (left->n_col == 0) {
		for (i = 0; i < prod->n_row; ++i)
````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_mat_cow`, `check_row_range`, `isl_mat_product`, `isl_assert` (+1 more); contains control flow with 1 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_mat_cow`, `check_row_range`, `isl_mat_product`, `isl_assert` (+1 more) 相关的例程; 包含控制流结构：1 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1285-1310

````c
			isl_seq_clr(prod->row[i], prod->n_col);
		isl_mat_free(left);
		isl_mat_free(right);
		return prod;
	}
	for (i = 0; i < prod->n_row; ++i) {
		for (j = 0; j < prod->n_col; ++j)
			isl_int_mul(prod->row[i][j],
				    left->row[i][0], right->row[0][j]);
		for (k = 1; k < left->n_col; ++k) {
			if (isl_int_is_zero(left->row[i][k]))
				continue;
			for (j = 0; j < prod->n_col; ++j)
				isl_int_addmul(prod->row[i][j],
					    left->row[i][k], right->row[k][j]);
		}
	}
	isl_mat_free(left);
	isl_mat_free(right);
	return prod;
error:
	isl_mat_free(left);
	isl_mat_free(right);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_seq_clr`, `isl_mat_free`, `isl_int_mul`, `isl_int_addmul`; contains control flow with 4 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_seq_clr`, `isl_mat_free`, `isl_int_mul`, `isl_int_addmul` 相关的例程; 包含控制流结构：4 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1311-1342

````c
/* Replace the variables x in the rows q by x' given by x = M x',
 * with M the matrix mat.
 *
 * If the number of new variables is greater than the original
 * number of variables, then the rows q have already been
 * preextended.  If the new number is smaller, then the coefficients
 * of the divs, which are not changed, need to be shifted down.
 * The row q may be the equalities, the inequalities or the
 * div expressions.  In the latter case, has_div is true and
 * we need to take into account the extra denominator column.
 */
static int preimage(struct isl_ctx *ctx, isl_int **q, unsigned n,
	unsigned n_div, int has_div, struct isl_mat *mat)
{
	int i;
	struct isl_mat *t;
	int e;

	if (mat->n_col >= mat->n_row)
		e = 0;
	else
		e = mat->n_row - mat->n_col;
	if (has_div)
		for (i = 0; i < n; ++i)
			isl_int_mul(q[i][0], q[i][0], mat->row[0][0]);
	t = isl_mat_sub_alloc6(mat->ctx, q, 0, n, has_div, mat->n_row);
	t = isl_mat_product(t, mat);
	if (!t)
		return -1;
	for (i = 0; i < n; ++i) {
		isl_seq_swp_or_cpy(q[i] + has_div, t->row[i], t->n_col);
		isl_seq_cpy(q[i] + has_div + t->n_col,
````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `preimage`, `isl_int_mul`, `isl_mat_sub_alloc6`, `isl_mat_product` (+2 more); contains control flow with 2 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `preimage`, `isl_int_mul`, `isl_mat_sub_alloc6`, `isl_mat_product` (+2 more) 相关的例程; 包含控制流结构：2 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1343-1372

````c
			    q[i] + has_div + t->n_col + e, n_div);
		isl_seq_clr(q[i] + has_div + t->n_col + n_div, e);
	}
	isl_mat_free(t);
	return 0;
}

/* Replace the variables x in bset by x' given by x = M x', with
 * M the matrix mat.
 *
 * If there are fewer variables x' then there are x, then we perform
 * the transformation in place, which means that, in principle,
 * this frees up some extra variables as the number
 * of columns remains constant, but we would have to extend
 * the div array too as the number of rows in this array is assumed
 * to be equal to extra.
 */
__isl_give isl_basic_set *isl_basic_set_preimage(
	__isl_take isl_basic_set *bset, __isl_take isl_mat *mat)
{
	struct isl_ctx *ctx;

	if (!bset || !mat)
		goto error;

	ctx = bset->ctx;
	bset = isl_basic_set_cow(bset);
	if (isl_basic_set_check_no_params(bset) < 0)
		goto error;

````
- **EN**: This block declares or references types such as `isl_ctx`; declares or defines routines around `isl_seq_clr`, `isl_mat_free`, `isl_basic_set_preimage`, `isl_basic_set_cow`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_ctx`; 声明或定义与 `isl_seq_clr`, `isl_mat_free`, `isl_basic_set_preimage`, `isl_basic_set_cow` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1373-1398

````c
	isl_assert(ctx, 1+bset->dim->n_out == mat->n_row, goto error);
	isl_assert(ctx, mat->n_col > 0, goto error);

	if (mat->n_col > mat->n_row) {
		bset = isl_basic_set_add_dims(bset, isl_dim_set,
						mat->n_col - mat->n_row);
		if (!bset)
			goto error;
	} else if (mat->n_col < mat->n_row) {
		bset->dim = isl_space_cow(bset->dim);
		if (!bset->dim)
			goto error;
		bset->dim->n_out -= mat->n_row - mat->n_col;
	}

	if (preimage(ctx, bset->eq, bset->n_eq, bset->n_div, 0,
			isl_mat_copy(mat)) < 0)
		goto error;

	if (preimage(ctx, bset->ineq, bset->n_ineq, bset->n_div, 0,
			isl_mat_copy(mat)) < 0)
		goto error;

	if (preimage(ctx, bset->div, bset->n_div, bset->n_div, 1, mat) < 0)
		goto error2;

````
- **EN**: This block declares or defines routines around `isl_assert`, `isl_basic_set_add_dims`, `isl_space_cow`, `isl_mat_copy`; contains control flow with 7 conditional check(s); adds defensive checks for invariants or impossible states; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_assert`, `isl_basic_set_add_dims`, `isl_space_cow`, `isl_mat_copy` 相关的例程; 包含控制流结构：7 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1399-1425

````c
	ISL_F_CLR(bset, ISL_BASIC_SET_NO_IMPLICIT);
	ISL_F_CLR(bset, ISL_BASIC_SET_NO_REDUNDANT);
	ISL_F_CLR(bset, ISL_BASIC_SET_SORTED);
	ISL_F_CLR(bset, ISL_BASIC_SET_NORMALIZED_DIVS);
	ISL_F_CLR(bset, ISL_BASIC_SET_ALL_EQUALITIES);
	ISL_F_CLR(bset, ISL_BASIC_SET_REDUCED_COEFFICIENTS);

	bset = isl_basic_set_simplify(bset);
	bset = isl_basic_set_finalize(bset);

	return bset;
error:
	isl_mat_free(mat);
error2:
	isl_basic_set_free(bset);
	return NULL;
}

__isl_give isl_set *isl_set_preimage(
	__isl_take isl_set *set, __isl_take isl_mat *mat)
{
	int i;

	set = isl_set_cow(set);
	if (!set)
		goto error;

````
- **EN**: This block declares or defines routines around `ISL_F_CLR`, `isl_basic_set_simplify`, `isl_basic_set_finalize`, `isl_mat_free` (+3 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `ISL_F_CLR`, `isl_basic_set_simplify`, `isl_basic_set_finalize`, `isl_mat_free` (+3 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1426-1458

````c
	for (i = 0; i < set->n; ++i) {
		set->p[i] = isl_basic_set_preimage(set->p[i],
						    isl_mat_copy(mat));
		if (!set->p[i])
			goto error;
	}
	if (mat->n_col != mat->n_row) {
		set->dim = isl_space_cow(set->dim);
		if (!set->dim)
			goto error;
		set->dim->n_out += mat->n_col;
		set->dim->n_out -= mat->n_row;
	}
	isl_mat_free(mat);
	ISL_F_CLR(set, ISL_SET_NORMALIZED);
	return set;
error:
	isl_set_free(set);
	isl_mat_free(mat);
	return NULL;
}

/* Replace the variables x starting at "first_col" in the rows "rows"
 * of some coefficient matrix by x' with x = M x' with M the matrix mat.
 * That is, replace the corresponding coefficients c by c M.
 */
isl_stat isl_mat_sub_transform(isl_int **row, unsigned n_row,
	unsigned first_col, __isl_take isl_mat *mat)
{
	int i;
	isl_ctx *ctx;
	isl_mat *t;

````
- **EN**: This block declares or defines routines around `isl_basic_set_preimage`, `isl_mat_copy`, `isl_space_cow`, `isl_mat_free` (+3 more); contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_basic_set_preimage`, `isl_mat_copy`, `isl_space_cow`, `isl_mat_free` (+3 more) 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1459-1490

````c
	if (!mat)
		return isl_stat_error;
	ctx = isl_mat_get_ctx(mat);
	t = isl_mat_sub_alloc6(ctx, row, 0, n_row, first_col, mat->n_row);
	t = isl_mat_product(t, mat);
	if (!t)
		return isl_stat_error;
	for (i = 0; i < n_row; ++i)
		isl_seq_swp_or_cpy(row[i] + first_col, t->row[i], t->n_col);
	isl_mat_free(t);
	return isl_stat_ok;
}

void isl_mat_print_internal(__isl_keep isl_mat *mat, FILE *out, int indent)
{
	int i, j;

	if (!mat) {
		fprintf(out, "%*snull mat\n", indent, "");
		return;
	}

	if (mat->n_row == 0)
		fprintf(out, "%*s[]\n", indent, "");

	for (i = 0; i < mat->n_row; ++i) {
		if (!i)
			fprintf(out, "%*s[[", indent, "");
		else
			fprintf(out, "%*s[", indent+1, "");
		for (j = 0; j < mat->n_col; ++j) {
			if (j)
````
- **EN**: This block declares or defines routines around `isl_mat_get_ctx`, `isl_mat_sub_alloc6`, `isl_mat_product`, `isl_seq_swp_or_cpy` (+3 more); contains control flow with 3 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_get_ctx`, `isl_mat_sub_alloc6`, `isl_mat_product`, `isl_seq_swp_or_cpy` (+3 more) 相关的例程; 包含控制流结构：3 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1491-1517

````c
			    fprintf(out, ",");
			isl_int_print(out, mat->row[i][j], 0);
		}
		if (i == mat->n_row-1)
			fprintf(out, "]]\n");
		else
			fprintf(out, "]\n");
	}
}

void isl_mat_dump(__isl_keep isl_mat *mat)
{
	isl_mat_print_internal(mat, stderr, 0);
}

__isl_give isl_mat *isl_mat_drop_cols(__isl_take isl_mat *mat,
	unsigned col, unsigned n)
{
	int r;

	if (n == 0)
		return mat;

	mat = isl_mat_cow(mat);
	if (check_col_range(mat, col, n) < 0)
		return isl_mat_free(mat);

````
- **EN**: This block declares or defines routines around `fprintf`, `isl_int_print`, `isl_mat_dump`, `isl_mat_print_internal` (+2 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `fprintf`, `isl_int_print`, `isl_mat_dump`, `isl_mat_print_internal` (+2 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1518-1547

````c
	if (col != mat->n_col-n) {
		for (r = 0; r < mat->n_row; ++r)
			isl_seq_cpy(mat->row[r]+col, mat->row[r]+col+n,
					mat->n_col - col - n);
	}
	mat->n_col -= n;
	return mat;
}

__isl_give isl_mat *isl_mat_drop_rows(__isl_take isl_mat *mat,
	unsigned row, unsigned n)
{
	int r;

	mat = isl_mat_cow(mat);
	if (check_row_range(mat, row, n) < 0)
		return isl_mat_free(mat);

	for (r = row; r+n < mat->n_row; ++r)
		mat->row[r] = mat->row[r+n];

	mat->n_row -= n;
	return mat;
}

__isl_give isl_mat *isl_mat_insert_cols(__isl_take isl_mat *mat,
				unsigned col, unsigned n)
{
	isl_mat *ext;

````
- **EN**: This block declares or defines routines around `isl_seq_cpy`, `isl_mat_drop_rows`, `isl_mat_cow`, `isl_mat_insert_cols`; contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_seq_cpy`, `isl_mat_drop_rows`, `isl_mat_cow`, `isl_mat_insert_cols` 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1548-1578

````c
	if (check_col_range(mat, col, 0) < 0)
		return isl_mat_free(mat);
	if (n == 0)
		return mat;

	ext = isl_mat_alloc(mat->ctx, mat->n_row, mat->n_col + n);
	if (!ext)
		goto error;

	isl_mat_sub_copy(mat->ctx, ext->row, mat->row, mat->n_row, 0, 0, col);
	isl_mat_sub_copy(mat->ctx, ext->row, mat->row, mat->n_row,
				col + n, col, mat->n_col - col);

	isl_mat_free(mat);
	return ext;
error:
	isl_mat_free(mat);
	return NULL;
}

__isl_give isl_mat *isl_mat_insert_zero_cols(__isl_take isl_mat *mat,
	unsigned first, unsigned n)
{
	int i;

	if (!mat)
		return NULL;
	mat = isl_mat_insert_cols(mat, first, n);
	if (!mat)
		return NULL;

````
- **EN**: This block declares or defines routines around `isl_mat_alloc`, `isl_mat_sub_copy`, `isl_mat_free`, `isl_mat_insert_zero_cols` (+1 more); contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_alloc`, `isl_mat_sub_copy`, `isl_mat_free`, `isl_mat_insert_zero_cols` (+1 more) 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1579-1606

````c
	for (i = 0; i < mat->n_row; ++i)
		isl_seq_clr(mat->row[i] + first, n);

	return mat;
}

__isl_give isl_mat *isl_mat_add_zero_cols(__isl_take isl_mat *mat, unsigned n)
{
	if (!mat)
		return NULL;

	return isl_mat_insert_zero_cols(mat, mat->n_col, n);
}

__isl_give isl_mat *isl_mat_insert_rows(__isl_take isl_mat *mat,
				unsigned row, unsigned n)
{
	isl_mat *ext;

	if (check_row_range(mat, row, 0) < 0)
		return isl_mat_free(mat);
	if (n == 0)
		return mat;

	ext = isl_mat_alloc(mat->ctx, mat->n_row + n, mat->n_col);
	if (!ext)
		goto error;

````
- **EN**: This block declares or defines routines around `isl_seq_clr`, `isl_mat_add_zero_cols`, `isl_mat_insert_rows`, `isl_mat_alloc`; contains control flow with 1 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_seq_clr`, `isl_mat_add_zero_cols`, `isl_mat_insert_rows`, `isl_mat_alloc` 相关的例程; 包含控制流结构：1 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1607-1634

````c
	isl_mat_sub_copy(mat->ctx, ext->row, mat->row, row, 0, 0, mat->n_col);
	isl_mat_sub_copy(mat->ctx, ext->row + row + n, mat->row + row,
				mat->n_row - row, 0, 0, mat->n_col);

	isl_mat_free(mat);
	return ext;
error:
	isl_mat_free(mat);
	return NULL;
}

__isl_give isl_mat *isl_mat_add_rows(__isl_take isl_mat *mat, unsigned n)
{
	if (!mat)
		return NULL;

	return isl_mat_insert_rows(mat, mat->n_row, n);
}

__isl_give isl_mat *isl_mat_insert_zero_rows(__isl_take isl_mat *mat,
	unsigned row, unsigned n)
{
	int i;

	mat = isl_mat_insert_rows(mat, row, n);
	if (!mat)
		return NULL;
	
````
- **EN**: This block declares or defines routines around `isl_mat_sub_copy`, `isl_mat_free`, `isl_mat_add_rows`, `isl_mat_insert_zero_rows` (+1 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_sub_copy`, `isl_mat_free`, `isl_mat_add_rows`, `isl_mat_insert_zero_rows` (+1 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1635-1661

````c
	for (i = 0; i < n; ++i)
		isl_seq_clr(mat->row[row + i], mat->n_col);

	return mat;
}

__isl_give isl_mat *isl_mat_add_zero_rows(__isl_take isl_mat *mat, unsigned n)
{
	if (!mat)
		return NULL;

	return isl_mat_insert_zero_rows(mat, mat->n_row, n);
}

void isl_mat_col_submul(__isl_keep isl_mat *mat,
			int dst_col, isl_int f, int src_col)
{
	int i;

	for (i = 0; i < mat->n_row; ++i)
		isl_int_submul(mat->row[i][dst_col], f, mat->row[i][src_col]);
}

void isl_mat_col_add(__isl_keep isl_mat *mat, int dst_col, int src_col)
{
	int i;

````
- **EN**: This block declares or defines routines around `isl_seq_clr`, `isl_mat_add_zero_rows`, `isl_mat_col_submul`, `isl_int_submul` (+1 more); contains control flow with 2 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_seq_clr`, `isl_mat_add_zero_rows`, `isl_mat_col_submul`, `isl_int_submul` (+1 more) 相关的例程; 包含控制流结构：2 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1662-1689

````c
	if (!mat)
		return;

	for (i = 0; i < mat->n_row; ++i)
		isl_int_add(mat->row[i][dst_col],
			    mat->row[i][dst_col], mat->row[i][src_col]);
}

void isl_mat_col_mul(__isl_keep isl_mat *mat, int dst_col, isl_int f,
	int src_col)
{
	int i;

	for (i = 0; i < mat->n_row; ++i)
		isl_int_mul(mat->row[i][dst_col], f, mat->row[i][src_col]);
}

/* Add "f" times column "src_col" to column "dst_col" of "mat" and
 * return the result.
 */
__isl_give isl_mat *isl_mat_col_addmul(__isl_take isl_mat *mat, int dst_col,
	isl_int f, int src_col)
{
	int i;

	if (check_col(mat, dst_col) < 0 || check_col(mat, src_col) < 0)
		return isl_mat_free(mat);

````
- **EN**: This block declares or defines routines around `isl_int_add`, `isl_mat_col_mul`, `isl_int_mul`, `isl_mat_col_addmul`; contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_int_add`, `isl_mat_col_mul`, `isl_int_mul`, `isl_mat_col_addmul` 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1690-1719

````c
	for (i = 0; i < mat->n_row; ++i) {
		if (isl_int_is_zero(mat->row[i][src_col]))
			continue;
		mat = isl_mat_cow(mat);
		if (!mat)
			return NULL;
		isl_int_addmul(mat->row[i][dst_col], f, mat->row[i][src_col]);
	}

	return mat;
}

/* Negate column "col" of "mat" and return the result.
 */
__isl_give isl_mat *isl_mat_col_neg(__isl_take isl_mat *mat, int col)
{
	int i;

	if (check_col(mat, col) < 0)
		return isl_mat_free(mat);

	for (i = 0; i < mat->n_row; ++i) {
		if (isl_int_is_zero(mat->row[i][col]))
			continue;
		mat = isl_mat_cow(mat);
		if (!mat)
			return NULL;
		isl_int_neg(mat->row[i][col], mat->row[i][col]);
	}

````
- **EN**: This block declares or defines routines around `isl_mat_cow`, `isl_int_addmul`, `isl_mat_col_neg`, `isl_int_neg`; contains control flow with 2 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_mat_cow`, `isl_int_addmul`, `isl_mat_col_neg`, `isl_int_neg` 相关的例程; 包含控制流结构：2 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1720-1745

````c
	return mat;
}

/* Negate row "row" of "mat" and return the result.
 */
__isl_give isl_mat *isl_mat_row_neg(__isl_take isl_mat *mat, int row)
{
	if (check_row(mat, row) < 0)
		return isl_mat_free(mat);
	if (!isl_seq_any_non_zero(mat->row[row], mat->n_col))
		return mat;
	mat = isl_mat_cow(mat);
	if (!mat)
		return NULL;
	isl_seq_neg(mat->row[row], mat->row[row], mat->n_col);
	return mat;
}

__isl_give isl_mat *isl_mat_unimodular_complete(__isl_take isl_mat *M, int row)
{
	int r;
	struct isl_mat *H = NULL, *Q = NULL;

	if (!M)
		return NULL;

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_mat_row_neg`, `isl_mat_cow`, `isl_seq_neg`, `isl_mat_unimodular_complete`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_mat_row_neg`, `isl_mat_cow`, `isl_seq_neg`, `isl_mat_unimodular_complete` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1746-1773

````c
	isl_assert(M->ctx, M->n_row == M->n_col, goto error);
	M->n_row = row;
	H = isl_mat_left_hermite(isl_mat_copy(M), 0, NULL, &Q);
	M->n_row = M->n_col;
	if (!H)
		goto error;
	for (r = 0; r < row; ++r)
		isl_assert(M->ctx, isl_int_is_one(H->row[r][r]), goto error);
	for (r = row; r < M->n_row; ++r)
		isl_seq_cpy(M->row[r], Q->row[r], M->n_col);
	isl_mat_free(H);
	isl_mat_free(Q);
	return M;
error:
	isl_mat_free(H);
	isl_mat_free(Q);
	isl_mat_free(M);
	return NULL;
}

__isl_give isl_mat *isl_mat_concat(__isl_take isl_mat *top,
	__isl_take isl_mat *bot)
{
	struct isl_mat *mat;

	if (!top || !bot)
		goto error;

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_assert`, `isl_mat_left_hermite`, `isl_seq_cpy`, `isl_mat_free` (+1 more); contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_assert`, `isl_mat_left_hermite`, `isl_seq_cpy`, `isl_mat_free` (+1 more) 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1774-1799

````c
	isl_assert(top->ctx, top->n_col == bot->n_col, goto error);
	if (top->n_row == 0) {
		isl_mat_free(top);
		return bot;
	}
	if (bot->n_row == 0) {
		isl_mat_free(bot);
		return top;
	}

	mat = isl_mat_alloc(top->ctx, top->n_row + bot->n_row, top->n_col);
	if (!mat)
		goto error;
	isl_mat_sub_copy(mat->ctx, mat->row, top->row, top->n_row,
			 0, 0, mat->n_col);
	isl_mat_sub_copy(mat->ctx, mat->row + top->n_row, bot->row, bot->n_row,
			 0, 0, mat->n_col);
	isl_mat_free(top);
	isl_mat_free(bot);
	return mat;
error:
	isl_mat_free(top);
	isl_mat_free(bot);
	return NULL;
}

````
- **EN**: This block declares or defines routines around `isl_assert`, `isl_mat_free`, `isl_mat_alloc`, `isl_mat_sub_copy`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_assert`, `isl_mat_free`, `isl_mat_alloc`, `isl_mat_sub_copy` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查；并延续周边实现细节。

### Lines 1800-1829

````c
isl_bool isl_mat_is_equal(__isl_keep isl_mat *mat1, __isl_keep isl_mat *mat2)
{
	int i;

	if (!mat1 || !mat2)
		return isl_bool_error;

	if (mat1->n_row != mat2->n_row)
		return isl_bool_false;

	if (mat1->n_col != mat2->n_col)
		return isl_bool_false;

	for (i = 0; i < mat1->n_row; ++i)
		if (!isl_seq_eq(mat1->row[i], mat2->row[i], mat1->n_col))
			return isl_bool_false;

	return isl_bool_true;
}

__isl_give isl_mat *isl_mat_from_row_vec(__isl_take isl_vec *vec)
{
	struct isl_mat *mat;

	if (!vec)
		return NULL;
	mat = isl_mat_alloc(vec->ctx, 1, vec->size);
	if (!mat)
		goto error;

````
- **EN**: This block declares or references types such as `isl_mat`; declares or defines routines around `isl_mat_is_equal`, `isl_mat_from_row_vec`, `isl_mat_alloc`; contains control flow with 1 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `isl_mat`; 声明或定义与 `isl_mat_is_equal`, `isl_mat_from_row_vec`, `isl_mat_alloc` 相关的例程; 包含控制流结构：1 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码；并延续周边实现细节。

### Lines 1830-1855

````c
	isl_seq_cpy(mat->row[0], vec->el, vec->size);

	isl_vec_free(vec);
	return mat;
error:
	isl_vec_free(vec);
	return NULL;
}

/* Return a copy of row "row" of "mat" as an isl_vec.
 */
__isl_give isl_vec *isl_mat_get_row(__isl_keep isl_mat *mat, unsigned row)
{
	isl_vec *v;

	if (!mat)
		return NULL;
	if (row >= mat->n_row)
		isl_die(mat->ctx, isl_error_invalid, "row out of range",
			return NULL);

	v = isl_vec_alloc(isl_mat_get_ctx(mat), mat->n_col);
	if (!v)
		return NULL;
	isl_seq_cpy(v->el, mat->row[row], mat->n_col);

````
- **EN**: This block declares or defines routines around `isl_seq_cpy`, `isl_vec_free`, `isl_mat_get_row`, `isl_die` (+1 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_seq_cpy`, `isl_vec_free`, `isl_mat_get_row`, `isl_die` (+1 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1856-1887

````c
	return v;
}

__isl_give isl_mat *isl_mat_vec_concat(__isl_take isl_mat *top,
	__isl_take isl_vec *bot)
{
	return isl_mat_concat(top, isl_mat_from_row_vec(bot));
}

__isl_give isl_mat *isl_mat_move_cols(__isl_take isl_mat *mat,
	unsigned dst_col, unsigned src_col, unsigned n)
{
	isl_mat *res;

	if (!mat)
		return NULL;
	if (n == 0 || dst_col == src_col)
		return mat;

	res = isl_mat_alloc(mat->ctx, mat->n_row, mat->n_col);
	if (!res)
		goto error;

	if (dst_col < src_col) {
		isl_mat_sub_copy(res->ctx, res->row, mat->row, mat->n_row,
				 0, 0, dst_col);
		isl_mat_sub_copy(res->ctx, res->row, mat->row, mat->n_row,
				 dst_col, src_col, n);
		isl_mat_sub_copy(res->ctx, res->row, mat->row, mat->n_row,
				 dst_col + n, dst_col, src_col - dst_col);
		isl_mat_sub_copy(res->ctx, res->row, mat->row, mat->n_row,
				 src_col + n, src_col + n,
````
- **EN**: This block declares or defines routines around `isl_mat_vec_concat`, `isl_mat_move_cols`, `isl_mat_alloc`, `isl_mat_sub_copy`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_vec_concat`, `isl_mat_move_cols`, `isl_mat_alloc`, `isl_mat_sub_copy` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1888-1915

````c
				 res->n_col - src_col - n);
	} else {
		isl_mat_sub_copy(res->ctx, res->row, mat->row, mat->n_row,
				 0, 0, src_col);
		isl_mat_sub_copy(res->ctx, res->row, mat->row, mat->n_row,
				 src_col, src_col + n, dst_col - src_col);
		isl_mat_sub_copy(res->ctx, res->row, mat->row, mat->n_row,
				 dst_col, src_col, n);
		isl_mat_sub_copy(res->ctx, res->row, mat->row, mat->n_row,
				 dst_col + n, dst_col + n,
				 res->n_col - dst_col - n);
	}
	isl_mat_free(mat);

	return res;
error:
	isl_mat_free(mat);
	return NULL;
}

/* Return the gcd of the elements in row "row" of "mat" in *gcd.
 * Return isl_stat_ok on success and isl_stat_error on failure.
 */
isl_stat isl_mat_row_gcd(__isl_keep isl_mat *mat, int row, isl_int *gcd)
{
	if (check_row(mat, row) < 0)
		return isl_stat_error;

````
- **EN**: This block declares or defines routines around `isl_mat_sub_copy`, `isl_mat_free`, `isl_mat_row_gcd`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_mat_sub_copy`, `isl_mat_free`, `isl_mat_row_gcd` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1916-1943

````c
	isl_seq_gcd(mat->row[row], mat->n_col, gcd);

	return isl_stat_ok;
}

void isl_mat_gcd(__isl_keep isl_mat *mat, isl_int *gcd)
{
	int i;
	isl_int g;

	isl_int_set_si(*gcd, 0);
	if (!mat)
		return;

	isl_int_init(g);
	for (i = 0; i < mat->n_row; ++i) {
		isl_seq_gcd(mat->row[i], mat->n_col, &g);
		isl_int_gcd(*gcd, *gcd, g);
	}
	isl_int_clear(g);
}

/* Return the result of scaling "mat" by a factor of "m".
 */
__isl_give isl_mat *isl_mat_scale(__isl_take isl_mat *mat, isl_int m)
{
	int i;

````
- **EN**: This block declares or defines routines around `isl_seq_gcd`, `isl_mat_gcd`, `isl_int_set_si`, `isl_int_init` (+3 more); contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_seq_gcd`, `isl_mat_gcd`, `isl_int_set_si`, `isl_int_init` (+3 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 1944-1970

````c
	if (isl_int_is_one(m))
		return mat;

	mat = isl_mat_cow(mat);
	if (!mat)
		return NULL;

	for (i = 0; i < mat->n_row; ++i)
		isl_seq_scale(mat->row[i], mat->row[i], m, mat->n_col);

	return mat;
}

__isl_give isl_mat *isl_mat_scale_down(__isl_take isl_mat *mat, isl_int m)
{
	int i;

	if (isl_int_is_one(m))
		return mat;

	mat = isl_mat_cow(mat);
	if (!mat)
		return NULL;

	for (i = 0; i < mat->n_row; ++i)
		isl_seq_scale_down(mat->row[i], mat->row[i], m, mat->n_col);

````
- **EN**: This block declares or defines routines around `isl_mat_cow`, `isl_seq_scale`, `isl_mat_scale_down`, `isl_seq_scale_down`; contains control flow with 2 loop construct(s), 4 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_cow`, `isl_seq_scale`, `isl_mat_scale_down`, `isl_seq_scale_down` 相关的例程; 包含控制流结构：2 处循环、4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 1971-2000

````c
	return mat;
}

__isl_give isl_mat *isl_mat_scale_down_row(__isl_take isl_mat *mat, int row,
	isl_int m)
{
	if (isl_int_is_one(m))
		return mat;

	mat = isl_mat_cow(mat);
	if (!mat)
		return NULL;

	isl_seq_scale_down(mat->row[row], mat->row[row], m, mat->n_col);

	return mat;
}

__isl_give isl_mat *isl_mat_normalize(__isl_take isl_mat *mat)
{
	isl_int gcd;

	if (!mat)
		return NULL;

	isl_int_init(gcd);
	isl_mat_gcd(mat, &gcd);
	mat = isl_mat_scale_down(mat, gcd);
	isl_int_clear(gcd);

````
- **EN**: This block declares or defines routines around `isl_mat_scale_down_row`, `isl_mat_cow`, `isl_seq_scale_down`, `isl_mat_normalize` (+4 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_scale_down_row`, `isl_mat_cow`, `isl_seq_scale_down`, `isl_mat_normalize` (+4 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

### Lines 2001-2027

````c
	return mat;
}

__isl_give isl_mat *isl_mat_normalize_row(__isl_take isl_mat *mat, int row)
{
	mat = isl_mat_cow(mat);
	if (!mat)
		return NULL;

	isl_seq_normalize(mat->ctx, mat->row[row], mat->n_col);

	return mat;
}

/* Number of initial non-zero columns.
 */
int isl_mat_initial_non_zero_cols(__isl_keep isl_mat *mat)
{
	int i;

	if (!mat)
		return -1;

	for (i = 0; i < mat->n_col; ++i)
		if (row_first_non_zero(mat->row, mat->n_row, i) < 0)
			break;

````
- **EN**: This block declares or defines routines around `isl_mat_normalize_row`, `isl_mat_cow`, `isl_seq_normalize`, `isl_mat_initial_non_zero_cols`; contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_mat_normalize_row`, `isl_mat_cow`, `isl_seq_normalize`, `isl_mat_initial_non_zero_cols` 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2028-2059

````c
	return i;
}

/* Return a basis for the space spanned by the rows of "mat".
 * Any basis will do, so simply perform Gaussian elimination and
 * remove the empty rows.
 */
__isl_give isl_mat *isl_mat_row_basis(__isl_take isl_mat *mat)
{
	return isl_mat_reverse_gauss(mat);
}

/* Return rows that extend a basis of "mat1" to one
 * that covers both "mat1" and "mat2".
 * The Hermite normal form of the concatenation of the two matrices is
 *
 *	                     [ Q1 ]
 *	[ M1 ] = [ H1 0  0 ] [ Q2 ]
 *	[ M2 ] = [ H2 H3 0 ] [ Q3 ]
 *
 * The number of columns in H1 and H3 determine the number of rows
 * in Q1 and Q2.  Q1 is a basis for M1, while Q2 extends this basis
 * to also cover M2.
 */
__isl_give isl_mat *isl_mat_row_basis_extension(
	__isl_take isl_mat *mat1, __isl_take isl_mat *mat2)
{
	isl_size n_row;
	int r1, r;
	isl_size n1;
	isl_mat *H, *Q;

````
- **EN**: This block declares or defines routines around `isl_mat_row_basis`, `isl_mat_row_basis_extension`; emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isl_mat_row_basis`, `isl_mat_row_basis_extension` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理; 保留用于解释意图、用法或算法背景的注释.

### Lines 2060-2093

````c
	n1 = isl_mat_rows(mat1);
	H = isl_mat_concat(mat1, mat2);
	H = isl_mat_left_hermite(H, 0, NULL, &Q);
	if (n1 < 0 || !H || !Q)
		goto error;

	r1 = hermite_first_zero_col(H, 0, n1);
	r = hermite_first_zero_col(H, r1, H->n_row);
	n_row = isl_mat_rows(Q);
	if (n_row < 0)
		goto error;
	Q = isl_mat_drop_rows(Q, r, n_row - r);
	Q = isl_mat_drop_rows(Q, 0, r1);

	isl_mat_free(H);
	return Q;
error:
	isl_mat_free(H);
	isl_mat_free(Q);
	return NULL;
}

/* Are the rows of "mat1" linearly independent of those of "mat2"?
 * That is, is there no linear dependence among the combined rows
 * that is not already present in either "mat1" or "mat2"?
 * In other words, is the rank of "mat1" and "mat2" combined equal
 * to the sum of the ranks of "mat1" and "mat2"?
 */
isl_bool isl_mat_has_linearly_independent_rows(__isl_keep isl_mat *mat1,
	__isl_keep isl_mat *mat2)
{
	isl_size r1, r2, r;
	isl_mat *mat;

````
- **EN**: This block declares or defines routines around `isl_mat_rows`, `isl_mat_concat`, `isl_mat_left_hermite`, `hermite_first_zero_col` (+3 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `isl_mat_rows`, `isl_mat_concat`, `isl_mat_left_hermite`, `hermite_first_zero_col` (+3 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理；并延续周边实现细节。

### Lines 2094-2111

````c
	r1 = isl_mat_rank(mat1);
	if (r1 < 0)
		return isl_bool_error;
	if (r1 == 0)
		return isl_bool_true;
	r2 = isl_mat_rank(mat2);
	if (r2 < 0)
		return isl_bool_error;
	if (r2 == 0)
		return isl_bool_true;

	mat = isl_mat_concat(isl_mat_copy(mat1), isl_mat_copy(mat2));
	r = isl_mat_rank(mat);
	isl_mat_free(mat);
	if (r < 0)
		return isl_bool_error;
	return isl_bool_ok(r == r1 + r2);
}
````
- **EN**: This block declares or defines routines around `isl_mat_rank`, `isl_mat_concat`, `isl_mat_free`; contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; uses ISL data structures or helpers for polyhedral reasoning.
- **CN**: 该代码块 声明或定义与 `isl_mat_rank`, `isl_mat_concat`, `isl_mat_free` 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 使用 ISL 数据结构或辅助函数进行多面体推理.

## Key Concepts / 关键概念

- **ISL relations and sets**
  - **CN**: ISL 关系与集合
- **Loop transformation**
  - **CN**: 循环变换
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **Debug and diagnostics**
  - **CN**: 调试与诊断
- **Memory access tracking**
  - **CN**: 内存访问跟踪
- **Dependence analysis**
  - **CN**: 依赖分析

## Dependencies / 依赖关系

- **ISL headers**: `isl/space.h` — ISL relation/set primitives that support polyhedral analysis and transformations.
  **ISL headers（CN）**：`isl/space.h` —— 支撑多面体分析与变换的 ISL 关系/集合原语。
- **System/standard headers**: `isl_ctx_private.h`, `isl_map_private.h`, `isl_seq.h`, `isl_mat_private.h`, `isl_vec_private.h`, `isl_space_private.h`, `isl_val_private.h` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`isl_ctx_private.h`, `isl_map_private.h`, `isl_seq.h`, `isl_mat_private.h`, `isl_vec_private.h`, `isl_space_private.h`, `isl_val_private.h` —— 实现所需的标准库或系统声明。
