# isl_pw_union_opt.c — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `polly/lib/External/isl/isl_pw_union_opt.c`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements core integer-set-library utilities centered on `isl_pw_union_opt` for Polly's bundled Integer Set Library (isl).
- **Purpose (CN)**: 为 Polly 内置的整数集合库（isl）实现围绕 `isl_pw_union_opt` 的整数集合库核心工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*
 * Copyright 2011      INRIA Saclay
 * Copyright 2012      Ecole Normale Superieure
 * Copyright 2020      Cerebras Systems
 *
 * Use of this software is governed by the MIT license
 *
 * Written by Sven Verdoolaege, INRIA Saclay - Ile-de-France,
 * Parc Club Orsay Universite, ZAC des vignes, 4 rue Jacques Monod,
 * 91893 Orsay, France
 * and Ecole Normale Superieure, 45 rue d'Ulm, 75230 Paris, France
 * and Cerebras Systems, 175 S San Antonio Rd, Los Altos, CA, USA
 */

#include <isl_pw_macro.h>

/* Given a function "cmp" that returns the set of elements where
 * "el1" is "better" than "el2", return this set.
 */
static __isl_give isl_set *FN(PW,better)(__isl_keep EL *el1, __isl_keep EL *el2,
````
- **L1 EN**: Separator comment used for visual grouping.
  **L1 CN**: 用于视觉分组的分隔注释。
- **L2 EN**: Comment explains nearby logic, invariants, or intent: `Copyright 2011      INRIA Saclay`.
  **L2 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copyright 2011      INRIA Saclay`。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Copyright 2012      Ecole Normale Superieure`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copyright 2012      Ecole Normale Superieure`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `Copyright 2020      Cerebras Systems`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copyright 2020      Cerebras Systems`。
- **L5 EN**: Separator comment used for visual grouping.
  **L5 CN**: 用于视觉分组的分隔注释。
- **L6 EN**: Comment explains nearby logic, invariants, or intent: `Use of this software is governed by the MIT license`.
  **L6 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Use of this software is governed by the MIT license`。
- **L7 EN**: Separator comment used for visual grouping.
  **L7 CN**: 用于视觉分组的分隔注释。
- **L8 EN**: Comment explains nearby logic, invariants, or intent: `Written by Sven Verdoolaege, INRIA Saclay - Ile-de-France,`.
  **L8 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Written by Sven Verdoolaege, INRIA Saclay - Ile-de-France,`。
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `Parc Club Orsay Universite, ZAC des vignes, 4 rue Jacques Monod,`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Parc Club Orsay Universite, ZAC des vignes, 4 rue Jacques Monod,`。
- **L10 EN**: Comment explains nearby logic, invariants, or intent: `91893 Orsay, France`.
  **L10 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`91893 Orsay, France`。
- **L11 EN**: Comment explains nearby logic, invariants, or intent: `and Ecole Normale Superieure, 45 rue d'Ulm, 75230 Paris, France`.
  **L11 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and Ecole Normale Superieure, 45 rue d'Ulm, 75230 Paris, France`。
- **L12 EN**: Comment explains nearby logic, invariants, or intent: `and Cerebras Systems, 175 S San Antonio Rd, Los Altos, CA, USA`.
  **L12 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and Cerebras Systems, 175 S San Antonio Rd, Los Altos, CA, USA`。
- **L13 EN**: Separator comment used for visual grouping.
  **L13 CN**: 用于视觉分组的分隔注释。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Includes <isl_pw_macro.h> to access local isl declarations paired with this implementation file.
  **L15 CN**: 引入 <isl_pw_macro.h> 以使用与该实现文件配套的本地 isl 声明。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L17 EN**: Comment explains nearby logic, invariants, or intent: `Given a function "cmp" that returns the set of elements where`.
  **L17 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Given a function "cmp" that returns the set of elements where`。
- **L18 EN**: Comment explains nearby logic, invariants, or intent: `"el1" is "better" than "el2", return this set.`.
  **L18 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`"el1" is "better" than "el2", return this set.`。
- **L19 EN**: Separator comment used for visual grouping.
  **L19 CN**: 用于视觉分组的分隔注释。
- **L20 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __isl_give isl_set *FN(PW,better)(__isl_keep EL *el1, __isl_keep EL *el2,`.
  **L20 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __isl_give isl_set *FN(PW,better)(__isl_keep EL *el1, __isl_keep EL *el2,`。

### Lines 21-40

````c
	__isl_give isl_set *(*cmp)(__isl_take EL *el1, __isl_take EL *el2))
{
	return cmp(FN(EL,copy)(el1), FN(EL,copy)(el2));
}

/* Return a list containing the domains of the pieces of "pw".
 */
static __isl_give isl_set_list *FN(PW,extract_domains)(__isl_keep PW *pw)
{
	int i;
	isl_ctx *ctx;
	isl_set_list *list;

	if (!pw)
		return NULL;
	ctx = FN(PW,get_ctx)(pw);
	list = isl_set_list_alloc(ctx, pw->n);
	for (i = 0; i < pw->n; ++i)
		list = isl_set_list_add(list, isl_set_copy(pw->p[i].set));

````
- **L21 EN**: Continues the surrounding expression or declaration: `__isl_give isl_set *(*cmp)(__isl_take EL *el1, __isl_take EL *el2))`.
  **L21 CN**: 继续构造周围的表达式或声明：`__isl_give isl_set *(*cmp)(__isl_take EL *el1, __isl_take EL *el2))`。
- **L22 EN**: Opens a new lexical scope or compound statement.
  **L22 CN**: 打开一个新的词法作用域或复合语句块。
- **L23 EN**: Returns from the current function with `cmp(FN(EL,copy)(el1), FN(EL,copy)(el2))`.
  **L23 CN**: 以 `cmp(FN(EL,copy)(el1), FN(EL,copy)(el2))` 从当前函数返回。
- **L24 EN**: Closes the current lexical scope or compound statement.
  **L24 CN**: 结束当前词法作用域或复合语句块。
- **L25 EN**: Blank line separating nearby declarations or logic blocks.
  **L25 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L26 EN**: Comment explains nearby logic, invariants, or intent: `Return a list containing the domains of the pieces of "pw".`.
  **L26 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Return a list containing the domains of the pieces of "pw".`。
- **L27 EN**: Separator comment used for visual grouping.
  **L27 CN**: 用于视觉分组的分隔注释。
- **L28 EN**: Continues logic associated with callable symbol `FN`.
  **L28 CN**: 继续与可调用符号 `FN` 相关的逻辑。
- **L29 EN**: Opens a new lexical scope or compound statement.
  **L29 CN**: 打开一个新的词法作用域或复合语句块。
- **L30 EN**: Executes a standalone statement or declaration: `int i;`.
  **L30 CN**: 执行一条独立语句或声明：`int i;`。
- **L31 EN**: Executes a standalone statement or declaration: `isl_ctx *ctx;`.
  **L31 CN**: 执行一条独立语句或声明：`isl_ctx *ctx;`。
- **L32 EN**: Executes a standalone statement or declaration: `isl_set_list *list;`.
  **L32 CN**: 执行一条独立语句或声明：`isl_set_list *list;`。
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L34 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L34 CN**: 开始 `if` 控制流语句并计算其条件。
- **L35 EN**: Returns from the current function with `NULL`.
  **L35 CN**: 以 `NULL` 从当前函数返回。
- **L36 EN**: Executes a call or declaration centered on `FN`.
  **L36 CN**: 执行以 `FN` 为核心的调用或声明。
- **L37 EN**: Executes a call or declaration centered on `isl_set_list_alloc`.
  **L37 CN**: 执行以 `isl_set_list_alloc` 为核心的调用或声明。
- **L38 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L38 CN**: 开始 `for` 控制流语句并计算其条件。
- **L39 EN**: Executes a call or declaration centered on `isl_set_list_add`.
  **L39 CN**: 执行以 `isl_set_list_add` 为核心的调用或声明。
- **L40 EN**: Blank line separating nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 41-60

````c
	return list;
}

/* Given sets B ("set"), C ("better") and A' ("out"), return
 *
 *	(B \cap C) \cup ((B \setminus C) \setminus A')
 */
static __isl_give isl_set *FN(PW,better_or_out)(__isl_take isl_set *set,
	__isl_take isl_set *better, __isl_take isl_set *out)
{
	isl_set *set_better, *set_out;

	set_better = isl_set_intersect(isl_set_copy(set), isl_set_copy(better));
	set_out = isl_set_subtract(isl_set_subtract(set, better), out);

	return isl_set_union(set_better, set_out);
}

/* Given sets A ("set"), C ("better") and B' ("out"), return
 *
````
- **L41 EN**: Returns from the current function with `list`.
  **L41 CN**: 以 `list` 从当前函数返回。
- **L42 EN**: Closes the current lexical scope or compound statement.
  **L42 CN**: 结束当前词法作用域或复合语句块。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L44 EN**: Comment explains nearby logic, invariants, or intent: `Given sets B ("set"), C ("better") and A' ("out"), return`.
  **L44 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Given sets B ("set"), C ("better") and A' ("out"), return`。
- **L45 EN**: Separator comment used for visual grouping.
  **L45 CN**: 用于视觉分组的分隔注释。
- **L46 EN**: Comment explains nearby logic, invariants, or intent: `(B \cap C) \cup ((B \setminus C) \setminus A')`.
  **L46 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(B \cap C) \cup ((B \setminus C) \setminus A')`。
- **L47 EN**: Separator comment used for visual grouping.
  **L47 CN**: 用于视觉分组的分隔注释。
- **L48 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __isl_give isl_set *FN(PW,better_or_out)(__isl_take isl_set *set,`.
  **L48 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __isl_give isl_set *FN(PW,better_or_out)(__isl_take isl_set *set,`。
- **L49 EN**: Continues the surrounding expression or declaration: `__isl_take isl_set *better, __isl_take isl_set *out)`.
  **L49 CN**: 继续构造周围的表达式或声明：`__isl_take isl_set *better, __isl_take isl_set *out)`。
- **L50 EN**: Opens a new lexical scope or compound statement.
  **L50 CN**: 打开一个新的词法作用域或复合语句块。
- **L51 EN**: Executes a standalone statement or declaration: `isl_set *set_better, *set_out;`.
  **L51 CN**: 执行一条独立语句或声明：`isl_set *set_better, *set_out;`。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L53 EN**: Executes a call or declaration centered on `isl_set_intersect`.
  **L53 CN**: 执行以 `isl_set_intersect` 为核心的调用或声明。
- **L54 EN**: Executes a call or declaration centered on `isl_set_subtract`.
  **L54 CN**: 执行以 `isl_set_subtract` 为核心的调用或声明。
- **L55 EN**: Blank line separating nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L56 EN**: Returns from the current function with `isl_set_union(set_better, set_out)`.
  **L56 CN**: 以 `isl_set_union(set_better, set_out)` 从当前函数返回。
- **L57 EN**: Closes the current lexical scope or compound statement.
  **L57 CN**: 结束当前词法作用域或复合语句块。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L59 EN**: Comment explains nearby logic, invariants, or intent: `Given sets A ("set"), C ("better") and B' ("out"), return`.
  **L59 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Given sets A ("set"), C ("better") and B' ("out"), return`。
- **L60 EN**: Separator comment used for visual grouping.
  **L60 CN**: 用于视觉分组的分隔注释。

### Lines 61-80

````c
 *	(A \setminus C) \cup ((A \cap C) \setminus B')
 */
static __isl_give isl_set *FN(PW,worse_or_out)(__isl_take isl_set *set,
	__isl_take isl_set *better, __isl_take isl_set *out)
{
	isl_set *set_worse, *set_out;

	set_worse = isl_set_subtract(isl_set_copy(set), isl_set_copy(better));
	set_out = isl_set_subtract(isl_set_intersect(set, better), out);

	return isl_set_union(set_worse, set_out);
}

/* Internal data structure used by isl_pw_*_union_opt_cmp
 * that keeps track of a piecewise expression with updated cells.
 * "pw" holds the original piecewise expression.
 * "list" holds the updated cells.
 */
S(PW,union_opt_cmp_data) {
	PW *pw;
````
- **L61 EN**: Comment explains nearby logic, invariants, or intent: `(A \setminus C) \cup ((A \cap C) \setminus B')`.
  **L61 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(A \setminus C) \cup ((A \cap C) \setminus B')`。
- **L62 EN**: Separator comment used for visual grouping.
  **L62 CN**: 用于视觉分组的分隔注释。
- **L63 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __isl_give isl_set *FN(PW,worse_or_out)(__isl_take isl_set *set,`.
  **L63 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __isl_give isl_set *FN(PW,worse_or_out)(__isl_take isl_set *set,`。
- **L64 EN**: Continues the surrounding expression or declaration: `__isl_take isl_set *better, __isl_take isl_set *out)`.
  **L64 CN**: 继续构造周围的表达式或声明：`__isl_take isl_set *better, __isl_take isl_set *out)`。
- **L65 EN**: Opens a new lexical scope or compound statement.
  **L65 CN**: 打开一个新的词法作用域或复合语句块。
- **L66 EN**: Executes a standalone statement or declaration: `isl_set *set_worse, *set_out;`.
  **L66 CN**: 执行一条独立语句或声明：`isl_set *set_worse, *set_out;`。
- **L67 EN**: Blank line separating nearby declarations or logic blocks.
  **L67 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L68 EN**: Executes a call or declaration centered on `isl_set_subtract`.
  **L68 CN**: 执行以 `isl_set_subtract` 为核心的调用或声明。
- **L69 EN**: Executes a call or declaration centered on `isl_set_subtract`.
  **L69 CN**: 执行以 `isl_set_subtract` 为核心的调用或声明。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L71 EN**: Returns from the current function with `isl_set_union(set_worse, set_out)`.
  **L71 CN**: 以 `isl_set_union(set_worse, set_out)` 从当前函数返回。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。
- **L73 EN**: Blank line separating nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L74 EN**: Comment explains nearby logic, invariants, or intent: `Internal data structure used by isl_pw_*_union_opt_cmp`.
  **L74 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Internal data structure used by isl_pw_*_union_opt_cmp`。
- **L75 EN**: Comment explains nearby logic, invariants, or intent: `that keeps track of a piecewise expression with updated cells.`.
  **L75 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`that keeps track of a piecewise expression with updated cells.`。
- **L76 EN**: Comment explains nearby logic, invariants, or intent: `"pw" holds the original piecewise expression.`.
  **L76 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`"pw" holds the original piecewise expression.`。
- **L77 EN**: Comment explains nearby logic, invariants, or intent: `"list" holds the updated cells.`.
  **L77 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`"list" holds the updated cells.`。
- **L78 EN**: Separator comment used for visual grouping.
  **L78 CN**: 用于视觉分组的分隔注释。
- **L79 EN**: Starts a function, helper, or structured scope: `S(PW,union_opt_cmp_data) {`.
  **L79 CN**: 开始一个函数、辅助例程或结构化作用域：`S(PW,union_opt_cmp_data) {`。
- **L80 EN**: Executes a standalone statement or declaration: `PW *pw;`.
  **L80 CN**: 执行一条独立语句或声明：`PW *pw;`。

### Lines 81-100

````c
	isl_set_list *cell;
};

/* Free all memory allocated for "data".
 */
static void FN(PW,union_opt_cmp_data_clear)(S(PW,union_opt_cmp_data) *data)
{
	isl_set_list_free(data->cell);
	FN(PW,free)(data->pw);
}

/* Given (potentially) updated cells "i" of data_i->pw and "j" of data_j->pw and
 * a set "better" where the piece from data_j->pw is better
 * than the piece from data_i->pw,
 * (further) update the specified cells such that only the better elements
 * remain on the (non-empty) intersection.
 *
 * Let C be the set "better".
 * Let A be the cell data_i->cell[i] and B the cell data_j->cell[j].
 *
````
- **L81 EN**: Executes a standalone statement or declaration: `isl_set_list *cell;`.
  **L81 CN**: 执行一条独立语句或声明：`isl_set_list *cell;`。
- **L82 EN**: Closes the current declaration scope such as a struct, enum, or aggregate.
  **L82 CN**: 结束当前声明作用域，例如结构体、枚举或聚合体。
- **L83 EN**: Blank line separating nearby declarations or logic blocks.
  **L83 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L84 EN**: Comment explains nearby logic, invariants, or intent: `Free all memory allocated for "data".`.
  **L84 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Free all memory allocated for "data".`。
- **L85 EN**: Separator comment used for visual grouping.
  **L85 CN**: 用于视觉分组的分隔注释。
- **L86 EN**: Continues logic associated with callable symbol `FN`.
  **L86 CN**: 继续与可调用符号 `FN` 相关的逻辑。
- **L87 EN**: Opens a new lexical scope or compound statement.
  **L87 CN**: 打开一个新的词法作用域或复合语句块。
- **L88 EN**: Executes a call or declaration centered on `isl_set_list_free`.
  **L88 CN**: 执行以 `isl_set_list_free` 为核心的调用或声明。
- **L89 EN**: Executes a call or declaration centered on `FN`.
  **L89 CN**: 执行以 `FN` 为核心的调用或声明。
- **L90 EN**: Closes the current lexical scope or compound statement.
  **L90 CN**: 结束当前词法作用域或复合语句块。
- **L91 EN**: Blank line separating nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L92 EN**: Comment explains nearby logic, invariants, or intent: `Given (potentially) updated cells "i" of data_i->pw and "j" of data_j->pw and`.
  **L92 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Given (potentially) updated cells "i" of data_i->pw and "j" of data_j->pw and`。
- **L93 EN**: Comment explains nearby logic, invariants, or intent: `a set "better" where the piece from data_j->pw is better`.
  **L93 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a set "better" where the piece from data_j->pw is better`。
- **L94 EN**: Comment explains nearby logic, invariants, or intent: `than the piece from data_i->pw,`.
  **L94 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`than the piece from data_i->pw,`。
- **L95 EN**: Comment explains nearby logic, invariants, or intent: `(further) update the specified cells such that only the better elements`.
  **L95 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(further) update the specified cells such that only the better elements`。
- **L96 EN**: Comment explains nearby logic, invariants, or intent: `remain on the (non-empty) intersection.`.
  **L96 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`remain on the (non-empty) intersection.`。
- **L97 EN**: Separator comment used for visual grouping.
  **L97 CN**: 用于视觉分组的分隔注释。
- **L98 EN**: Comment explains nearby logic, invariants, or intent: `Let C be the set "better".`.
  **L98 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Let C be the set "better".`。
- **L99 EN**: Comment explains nearby logic, invariants, or intent: `Let A be the cell data_i->cell[i] and B the cell data_j->cell[j].`.
  **L99 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Let A be the cell data_i->cell[i] and B the cell data_j->cell[j].`。
- **L100 EN**: Separator comment used for visual grouping.
  **L100 CN**: 用于视觉分组的分隔注释。

### Lines 101-120

````c
 * The elements in C need to be removed from A, except for those parts
 * that lie outside of B.  That is,
 *
 *	A <- (A \setminus C) \cup ((A \cap C) \setminus B')
 *
 * Conversely, the elements in B need to be restricted to C, except
 * for those parts that lie outside of A.  That is
 *
 *	B <- (B \cap C) \cup ((B \setminus C) \setminus A')
 *
 * Since all pairs of pieces are considered, the domains are updated
 * several times.  A and B refer to these updated domains
 * (kept track of in data_i->cell[i] and data_j->cell[j]), while A' and B' refer
 * to the original domains of the pieces.  It is safe to use these
 * original domains because the difference between, say, A' and A is
 * the domains of pw2-pieces that have been removed before and
 * those domains are disjoint from B.  A' is used instead of A
 * because the continued updating of A may result in this domain
 * getting broken up into more disjuncts.
 */
````
- **L101 EN**: Comment explains nearby logic, invariants, or intent: `The elements in C need to be removed from A, except for those parts`.
  **L101 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The elements in C need to be removed from A, except for those parts`。
- **L102 EN**: Comment explains nearby logic, invariants, or intent: `that lie outside of B.  That is,`.
  **L102 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`that lie outside of B.  That is,`。
- **L103 EN**: Separator comment used for visual grouping.
  **L103 CN**: 用于视觉分组的分隔注释。
- **L104 EN**: Comment explains nearby logic, invariants, or intent: `A <- (A \setminus C) \cup ((A \cap C) \setminus B')`.
  **L104 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A <- (A \setminus C) \cup ((A \cap C) \setminus B')`。
- **L105 EN**: Separator comment used for visual grouping.
  **L105 CN**: 用于视觉分组的分隔注释。
- **L106 EN**: Comment explains nearby logic, invariants, or intent: `Conversely, the elements in B need to be restricted to C, except`.
  **L106 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Conversely, the elements in B need to be restricted to C, except`。
- **L107 EN**: Comment explains nearby logic, invariants, or intent: `for those parts that lie outside of A.  That is`.
  **L107 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for those parts that lie outside of A.  That is`。
- **L108 EN**: Separator comment used for visual grouping.
  **L108 CN**: 用于视觉分组的分隔注释。
- **L109 EN**: Comment explains nearby logic, invariants, or intent: `B <- (B \cap C) \cup ((B \setminus C) \setminus A')`.
  **L109 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`B <- (B \cap C) \cup ((B \setminus C) \setminus A')`。
- **L110 EN**: Separator comment used for visual grouping.
  **L110 CN**: 用于视觉分组的分隔注释。
- **L111 EN**: Comment explains nearby logic, invariants, or intent: `Since all pairs of pieces are considered, the domains are updated`.
  **L111 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Since all pairs of pieces are considered, the domains are updated`。
- **L112 EN**: Comment explains nearby logic, invariants, or intent: `several times.  A and B refer to these updated domains`.
  **L112 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`several times.  A and B refer to these updated domains`。
- **L113 EN**: Comment explains nearby logic, invariants, or intent: `(kept track of in data_i->cell[i] and data_j->cell[j]), while A' and B' refer`.
  **L113 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(kept track of in data_i->cell[i] and data_j->cell[j]), while A' and B' refer`。
- **L114 EN**: Comment explains nearby logic, invariants, or intent: `to the original domains of the pieces.  It is safe to use these`.
  **L114 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to the original domains of the pieces.  It is safe to use these`。
- **L115 EN**: Comment explains nearby logic, invariants, or intent: `original domains because the difference between, say, A' and A is`.
  **L115 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`original domains because the difference between, say, A' and A is`。
- **L116 EN**: Comment explains nearby logic, invariants, or intent: `the domains of pw2-pieces that have been removed before and`.
  **L116 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the domains of pw2-pieces that have been removed before and`。
- **L117 EN**: Comment explains nearby logic, invariants, or intent: `those domains are disjoint from B.  A' is used instead of A`.
  **L117 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`those domains are disjoint from B.  A' is used instead of A`。
- **L118 EN**: Comment explains nearby logic, invariants, or intent: `because the continued updating of A may result in this domain`.
  **L118 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`because the continued updating of A may result in this domain`。
- **L119 EN**: Comment explains nearby logic, invariants, or intent: `getting broken up into more disjuncts.`.
  **L119 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`getting broken up into more disjuncts.`。
- **L120 EN**: Separator comment used for visual grouping.
  **L120 CN**: 用于视觉分组的分隔注释。

### Lines 121-140

````c
static isl_stat FN(PW,union_opt_cmp_split)(S(PW,union_opt_cmp_data) *data_i,
	int i, S(PW,union_opt_cmp_data) *data_j, int j,
	__isl_take isl_set *better)
{
	isl_set *set_i, *set_j;

	set_i = isl_set_list_get_set(data_i->cell, i);
	set_j = FN(PW,get_domain_at)(data_j->pw, j);
	set_i = FN(PW,worse_or_out)(set_i, isl_set_copy(better), set_j);
	data_i->cell = isl_set_list_set_set(data_i->cell, i, set_i);
	set_i = FN(PW,get_domain_at)(data_i->pw, i);
	set_j = isl_set_list_get_set(data_j->cell, j);
	set_j = FN(PW,better_or_out)(set_j, better, set_i);
	data_j->cell = isl_set_list_set_set(data_j->cell, j, set_j);

	return isl_stat_ok;
}

/* Given (potentially) updated cells "i" of data_i->pw and "j" of data_j->pw and
 * a function "cmp" that returns the set of elements where
````
- **L121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static isl_stat FN(PW,union_opt_cmp_split)(S(PW,union_opt_cmp_data) *data_i,`.
  **L121 CN**: 继续一个多行参数列表、初始化器或聚合项：`static isl_stat FN(PW,union_opt_cmp_split)(S(PW,union_opt_cmp_data) *data_i,`。
- **L122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int i, S(PW,union_opt_cmp_data) *data_j, int j,`.
  **L122 CN**: 继续一个多行参数列表、初始化器或聚合项：`int i, S(PW,union_opt_cmp_data) *data_j, int j,`。
- **L123 EN**: Continues the surrounding expression or declaration: `__isl_take isl_set *better)`.
  **L123 CN**: 继续构造周围的表达式或声明：`__isl_take isl_set *better)`。
- **L124 EN**: Opens a new lexical scope or compound statement.
  **L124 CN**: 打开一个新的词法作用域或复合语句块。
- **L125 EN**: Executes a standalone statement or declaration: `isl_set *set_i, *set_j;`.
  **L125 CN**: 执行一条独立语句或声明：`isl_set *set_i, *set_j;`。
- **L126 EN**: Blank line separating nearby declarations or logic blocks.
  **L126 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L127 EN**: Executes a call or declaration centered on `isl_set_list_get_set`.
  **L127 CN**: 执行以 `isl_set_list_get_set` 为核心的调用或声明。
- **L128 EN**: Executes a call or declaration centered on `FN`.
  **L128 CN**: 执行以 `FN` 为核心的调用或声明。
- **L129 EN**: Executes a call or declaration centered on `FN`.
  **L129 CN**: 执行以 `FN` 为核心的调用或声明。
- **L130 EN**: Executes a call or declaration centered on `isl_set_list_set_set`.
  **L130 CN**: 执行以 `isl_set_list_set_set` 为核心的调用或声明。
- **L131 EN**: Executes a call or declaration centered on `FN`.
  **L131 CN**: 执行以 `FN` 为核心的调用或声明。
- **L132 EN**: Executes a call or declaration centered on `isl_set_list_get_set`.
  **L132 CN**: 执行以 `isl_set_list_get_set` 为核心的调用或声明。
- **L133 EN**: Executes a call or declaration centered on `FN`.
  **L133 CN**: 执行以 `FN` 为核心的调用或声明。
- **L134 EN**: Executes a call or declaration centered on `isl_set_list_set_set`.
  **L134 CN**: 执行以 `isl_set_list_set_set` 为核心的调用或声明。
- **L135 EN**: Blank line separating nearby declarations or logic blocks.
  **L135 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L136 EN**: Returns from the current function with `isl_stat_ok`.
  **L136 CN**: 以 `isl_stat_ok` 从当前函数返回。
- **L137 EN**: Closes the current lexical scope or compound statement.
  **L137 CN**: 结束当前词法作用域或复合语句块。
- **L138 EN**: Blank line separating nearby declarations or logic blocks.
  **L138 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L139 EN**: Comment explains nearby logic, invariants, or intent: `Given (potentially) updated cells "i" of data_i->pw and "j" of data_j->pw and`.
  **L139 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Given (potentially) updated cells "i" of data_i->pw and "j" of data_j->pw and`。
- **L140 EN**: Comment explains nearby logic, invariants, or intent: `a function "cmp" that returns the set of elements where`.
  **L140 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a function "cmp" that returns the set of elements where`。

### Lines 141-160

````c
 * "el1" is "better" than "el2",
 * (further) update the specified cells such that only the "better" elements
 * remain on the (non-empty) intersection.
 */
static isl_stat FN(PW,union_opt_cmp_pair)(S(PW,union_opt_cmp_data) *data_i,
	int i, S(PW,union_opt_cmp_data) *data_j, int j,
	__isl_give isl_set *(*cmp)(__isl_take EL *el1, __isl_take EL *el2))
{
	isl_set *better;
	EL *el_i, *el_j;

	el_i = FN(PW,peek_base_at)(data_i->pw, i);
	el_j = FN(PW,peek_base_at)(data_j->pw, j);
	better = FN(PW,better)(el_j, el_i, cmp);
	return FN(PW,union_opt_cmp_split)(data_i, i, data_j, j, better);
}

/* Given (potentially) updated cells "i" of data_i->pw and "j" of data_j->pw and
 * a function "cmp" that returns the set of elements where
 * "el1" is "better" than "el2",
````
- **L141 EN**: Comment explains nearby logic, invariants, or intent: `"el1" is "better" than "el2",`.
  **L141 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`"el1" is "better" than "el2",`。
- **L142 EN**: Comment explains nearby logic, invariants, or intent: `(further) update the specified cells such that only the "better" elements`.
  **L142 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(further) update the specified cells such that only the "better" elements`。
- **L143 EN**: Comment explains nearby logic, invariants, or intent: `remain on the (non-empty) intersection.`.
  **L143 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`remain on the (non-empty) intersection.`。
- **L144 EN**: Separator comment used for visual grouping.
  **L144 CN**: 用于视觉分组的分隔注释。
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static isl_stat FN(PW,union_opt_cmp_pair)(S(PW,union_opt_cmp_data) *data_i,`.
  **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`static isl_stat FN(PW,union_opt_cmp_pair)(S(PW,union_opt_cmp_data) *data_i,`。
- **L146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int i, S(PW,union_opt_cmp_data) *data_j, int j,`.
  **L146 CN**: 继续一个多行参数列表、初始化器或聚合项：`int i, S(PW,union_opt_cmp_data) *data_j, int j,`。
- **L147 EN**: Continues the surrounding expression or declaration: `__isl_give isl_set *(*cmp)(__isl_take EL *el1, __isl_take EL *el2))`.
  **L147 CN**: 继续构造周围的表达式或声明：`__isl_give isl_set *(*cmp)(__isl_take EL *el1, __isl_take EL *el2))`。
- **L148 EN**: Opens a new lexical scope or compound statement.
  **L148 CN**: 打开一个新的词法作用域或复合语句块。
- **L149 EN**: Executes a standalone statement or declaration: `isl_set *better;`.
  **L149 CN**: 执行一条独立语句或声明：`isl_set *better;`。
- **L150 EN**: Executes a standalone statement or declaration: `EL *el_i, *el_j;`.
  **L150 CN**: 执行一条独立语句或声明：`EL *el_i, *el_j;`。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L152 EN**: Executes a call or declaration centered on `FN`.
  **L152 CN**: 执行以 `FN` 为核心的调用或声明。
- **L153 EN**: Executes a call or declaration centered on `FN`.
  **L153 CN**: 执行以 `FN` 为核心的调用或声明。
- **L154 EN**: Executes a call or declaration centered on `FN`.
  **L154 CN**: 执行以 `FN` 为核心的调用或声明。
- **L155 EN**: Returns from the current function with `FN(PW,union_opt_cmp_split)(data_i, i, data_j, j, better)`.
  **L155 CN**: 以 `FN(PW,union_opt_cmp_split)(data_i, i, data_j, j, better)` 从当前函数返回。
- **L156 EN**: Closes the current lexical scope or compound statement.
  **L156 CN**: 结束当前词法作用域或复合语句块。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L158 EN**: Comment explains nearby logic, invariants, or intent: `Given (potentially) updated cells "i" of data_i->pw and "j" of data_j->pw and`.
  **L158 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Given (potentially) updated cells "i" of data_i->pw and "j" of data_j->pw and`。
- **L159 EN**: Comment explains nearby logic, invariants, or intent: `a function "cmp" that returns the set of elements where`.
  **L159 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a function "cmp" that returns the set of elements where`。
- **L160 EN**: Comment explains nearby logic, invariants, or intent: `"el1" is "better" than "el2",`.
  **L160 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`"el1" is "better" than "el2",`。

### Lines 161-180

````c
 * (further) update the specified cells such that only the "better" elements
 * remain on the (non-empty) intersection.
 *
 * The base computation is performed by isl_pw_*_union_opt_cmp_pair,
 * which splits the cells according to the set of elements
 * where the piece from data_j->pw is better than the piece from data_i->pw.
 *
 * In some cases, there may be a subset of the intersection
 * where both pieces have the same value and can therefore
 * both be considered to be "better" than the other.
 * This can result in unnecessary splitting on this subset.
 * Avoid some of these cases by checking whether
 * data_i->pw is always better than data_j->pw on the intersection.
 * In particular, do this for the special case where this intersection
 * is equal to the cell "j" and data_i->pw is better on its entire cell.
 *
 * Similarly, if data_i->pw is never better than data_j->pw,
 * then no splitting will occur and there is no need to check
 * where data_j->pw is better than data_i->pw.
 */
````
- **L161 EN**: Comment explains nearby logic, invariants, or intent: `(further) update the specified cells such that only the "better" elements`.
  **L161 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(further) update the specified cells such that only the "better" elements`。
- **L162 EN**: Comment explains nearby logic, invariants, or intent: `remain on the (non-empty) intersection.`.
  **L162 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`remain on the (non-empty) intersection.`。
- **L163 EN**: Separator comment used for visual grouping.
  **L163 CN**: 用于视觉分组的分隔注释。
- **L164 EN**: Comment explains nearby logic, invariants, or intent: `The base computation is performed by isl_pw_*_union_opt_cmp_pair,`.
  **L164 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The base computation is performed by isl_pw_*_union_opt_cmp_pair,`。
- **L165 EN**: Comment explains nearby logic, invariants, or intent: `which splits the cells according to the set of elements`.
  **L165 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`which splits the cells according to the set of elements`。
- **L166 EN**: Comment explains nearby logic, invariants, or intent: `where the piece from data_j->pw is better than the piece from data_i->pw.`.
  **L166 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`where the piece from data_j->pw is better than the piece from data_i->pw.`。
- **L167 EN**: Separator comment used for visual grouping.
  **L167 CN**: 用于视觉分组的分隔注释。
- **L168 EN**: Comment explains nearby logic, invariants, or intent: `In some cases, there may be a subset of the intersection`.
  **L168 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`In some cases, there may be a subset of the intersection`。
- **L169 EN**: Comment explains nearby logic, invariants, or intent: `where both pieces have the same value and can therefore`.
  **L169 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`where both pieces have the same value and can therefore`。
- **L170 EN**: Comment explains nearby logic, invariants, or intent: `both be considered to be "better" than the other.`.
  **L170 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`both be considered to be "better" than the other.`。
- **L171 EN**: Comment explains nearby logic, invariants, or intent: `This can result in unnecessary splitting on this subset.`.
  **L171 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This can result in unnecessary splitting on this subset.`。
- **L172 EN**: Comment explains nearby logic, invariants, or intent: `Avoid some of these cases by checking whether`.
  **L172 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Avoid some of these cases by checking whether`。
- **L173 EN**: Comment explains nearby logic, invariants, or intent: `data_i->pw is always better than data_j->pw on the intersection.`.
  **L173 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`data_i->pw is always better than data_j->pw on the intersection.`。
- **L174 EN**: Comment explains nearby logic, invariants, or intent: `In particular, do this for the special case where this intersection`.
  **L174 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`In particular, do this for the special case where this intersection`。
- **L175 EN**: Comment explains nearby logic, invariants, or intent: `is equal to the cell "j" and data_i->pw is better on its entire cell.`.
  **L175 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`is equal to the cell "j" and data_i->pw is better on its entire cell.`。
- **L176 EN**: Separator comment used for visual grouping.
  **L176 CN**: 用于视觉分组的分隔注释。
- **L177 EN**: Comment explains nearby logic, invariants, or intent: `Similarly, if data_i->pw is never better than data_j->pw,`.
  **L177 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Similarly, if data_i->pw is never better than data_j->pw,`。
- **L178 EN**: Comment explains nearby logic, invariants, or intent: `then no splitting will occur and there is no need to check`.
  **L178 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`then no splitting will occur and there is no need to check`。
- **L179 EN**: Comment explains nearby logic, invariants, or intent: `where data_j->pw is better than data_i->pw.`.
  **L179 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`where data_j->pw is better than data_i->pw.`。
- **L180 EN**: Separator comment used for visual grouping.
  **L180 CN**: 用于视觉分组的分隔注释。

### Lines 181-200

````c
static isl_stat FN(PW,union_opt_cmp_two)(S(PW,union_opt_cmp_data) *data_i,
	int i, S(PW,union_opt_cmp_data) *data_j, int j,
	__isl_give isl_set *(*cmp)(__isl_take EL *el1, __isl_take EL *el2))
{
	isl_bool is_subset, is_empty;
	isl_set *better, *set_i, *set_j;
	EL *el_i, *el_j;

	set_i = FN(PW,peek_domain_at)(data_i->pw, i);
	set_j = FN(PW,peek_domain_at)(data_j->pw, j);
	is_subset = isl_set_is_subset(set_j, set_i);
	if (is_subset < 0)
		return isl_stat_error;
	if (!is_subset)
		return FN(PW,union_opt_cmp_pair)(data_i, i, data_j, j, cmp);

	el_i = FN(PW,peek_base_at)(data_i->pw, i);
	el_j = FN(PW,peek_base_at)(data_j->pw, j);
	better = FN(PW,better)(el_i, el_j, cmp);
	is_empty = isl_set_is_empty(better);
````
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static isl_stat FN(PW,union_opt_cmp_two)(S(PW,union_opt_cmp_data) *data_i,`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`static isl_stat FN(PW,union_opt_cmp_two)(S(PW,union_opt_cmp_data) *data_i,`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int i, S(PW,union_opt_cmp_data) *data_j, int j,`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`int i, S(PW,union_opt_cmp_data) *data_j, int j,`。
- **L183 EN**: Continues the surrounding expression or declaration: `__isl_give isl_set *(*cmp)(__isl_take EL *el1, __isl_take EL *el2))`.
  **L183 CN**: 继续构造周围的表达式或声明：`__isl_give isl_set *(*cmp)(__isl_take EL *el1, __isl_take EL *el2))`。
- **L184 EN**: Opens a new lexical scope or compound statement.
  **L184 CN**: 打开一个新的词法作用域或复合语句块。
- **L185 EN**: Executes a standalone statement or declaration: `isl_bool is_subset, is_empty;`.
  **L185 CN**: 执行一条独立语句或声明：`isl_bool is_subset, is_empty;`。
- **L186 EN**: Executes a standalone statement or declaration: `isl_set *better, *set_i, *set_j;`.
  **L186 CN**: 执行一条独立语句或声明：`isl_set *better, *set_i, *set_j;`。
- **L187 EN**: Executes a standalone statement or declaration: `EL *el_i, *el_j;`.
  **L187 CN**: 执行一条独立语句或声明：`EL *el_i, *el_j;`。
- **L188 EN**: Blank line separating nearby declarations or logic blocks.
  **L188 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L189 EN**: Executes a call or declaration centered on `FN`.
  **L189 CN**: 执行以 `FN` 为核心的调用或声明。
- **L190 EN**: Executes a call or declaration centered on `FN`.
  **L190 CN**: 执行以 `FN` 为核心的调用或声明。
- **L191 EN**: Executes a call or declaration centered on `isl_set_is_subset`.
  **L191 CN**: 执行以 `isl_set_is_subset` 为核心的调用或声明。
- **L192 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L192 CN**: 开始 `if` 控制流语句并计算其条件。
- **L193 EN**: Returns from the current function with `isl_stat_error`.
  **L193 CN**: 以 `isl_stat_error` 从当前函数返回。
- **L194 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L194 CN**: 开始 `if` 控制流语句并计算其条件。
- **L195 EN**: Returns from the current function with `FN(PW,union_opt_cmp_pair)(data_i, i, data_j, j, cmp)`.
  **L195 CN**: 以 `FN(PW,union_opt_cmp_pair)(data_i, i, data_j, j, cmp)` 从当前函数返回。
- **L196 EN**: Blank line separating nearby declarations or logic blocks.
  **L196 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L197 EN**: Executes a call or declaration centered on `FN`.
  **L197 CN**: 执行以 `FN` 为核心的调用或声明。
- **L198 EN**: Executes a call or declaration centered on `FN`.
  **L198 CN**: 执行以 `FN` 为核心的调用或声明。
- **L199 EN**: Executes a call or declaration centered on `FN`.
  **L199 CN**: 执行以 `FN` 为核心的调用或声明。
- **L200 EN**: Executes a call or declaration centered on `isl_set_is_empty`.
  **L200 CN**: 执行以 `isl_set_is_empty` 为核心的调用或声明。

### Lines 201-220

````c
	if (is_empty >= 0 && is_empty)
		return FN(PW,union_opt_cmp_split)(data_j, j, data_i, i, better);
	is_subset = isl_set_is_subset(set_i, better);
	if (is_subset >= 0 && is_subset)
		return FN(PW,union_opt_cmp_split)(data_j, j, data_i, i, better);
	isl_set_free(better);
	if (is_empty < 0 || is_subset < 0)
		return isl_stat_error;

	return FN(PW,union_opt_cmp_pair)(data_i, i, data_j, j, cmp);
}

/* Given two piecewise expressions data1->pw and data2->pw, replace
 * their domains
 * by the sets in data1->cell and data2->cell and combine the results into
 * a single piecewise expression.
 * The pieces of data1->pw and data2->pw are assumed to have been sorted
 * according to the function value expressions.
 * The pieces of the result are also sorted in this way.
 *
````
- **L201 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L201 CN**: 开始 `if` 控制流语句并计算其条件。
- **L202 EN**: Returns from the current function with `FN(PW,union_opt_cmp_split)(data_j, j, data_i, i, better)`.
  **L202 CN**: 以 `FN(PW,union_opt_cmp_split)(data_j, j, data_i, i, better)` 从当前函数返回。
- **L203 EN**: Executes a call or declaration centered on `isl_set_is_subset`.
  **L203 CN**: 执行以 `isl_set_is_subset` 为核心的调用或声明。
- **L204 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L204 CN**: 开始 `if` 控制流语句并计算其条件。
- **L205 EN**: Returns from the current function with `FN(PW,union_opt_cmp_split)(data_j, j, data_i, i, better)`.
  **L205 CN**: 以 `FN(PW,union_opt_cmp_split)(data_j, j, data_i, i, better)` 从当前函数返回。
- **L206 EN**: Executes a call or declaration centered on `isl_set_free`.
  **L206 CN**: 执行以 `isl_set_free` 为核心的调用或声明。
- **L207 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L207 CN**: 开始 `if` 控制流语句并计算其条件。
- **L208 EN**: Returns from the current function with `isl_stat_error`.
  **L208 CN**: 以 `isl_stat_error` 从当前函数返回。
- **L209 EN**: Blank line separating nearby declarations or logic blocks.
  **L209 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L210 EN**: Returns from the current function with `FN(PW,union_opt_cmp_pair)(data_i, i, data_j, j, cmp)`.
  **L210 CN**: 以 `FN(PW,union_opt_cmp_pair)(data_i, i, data_j, j, cmp)` 从当前函数返回。
- **L211 EN**: Closes the current lexical scope or compound statement.
  **L211 CN**: 结束当前词法作用域或复合语句块。
- **L212 EN**: Blank line separating nearby declarations or logic blocks.
  **L212 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L213 EN**: Comment explains nearby logic, invariants, or intent: `Given two piecewise expressions data1->pw and data2->pw, replace`.
  **L213 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Given two piecewise expressions data1->pw and data2->pw, replace`。
- **L214 EN**: Comment explains nearby logic, invariants, or intent: `their domains`.
  **L214 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`their domains`。
- **L215 EN**: Comment explains nearby logic, invariants, or intent: `by the sets in data1->cell and data2->cell and combine the results into`.
  **L215 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`by the sets in data1->cell and data2->cell and combine the results into`。
- **L216 EN**: Comment explains nearby logic, invariants, or intent: `a single piecewise expression.`.
  **L216 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a single piecewise expression.`。
- **L217 EN**: Comment explains nearby logic, invariants, or intent: `The pieces of data1->pw and data2->pw are assumed to have been sorted`.
  **L217 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The pieces of data1->pw and data2->pw are assumed to have been sorted`。
- **L218 EN**: Comment explains nearby logic, invariants, or intent: `according to the function value expressions.`.
  **L218 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`according to the function value expressions.`。
- **L219 EN**: Comment explains nearby logic, invariants, or intent: `The pieces of the result are also sorted in this way.`.
  **L219 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The pieces of the result are also sorted in this way.`。
- **L220 EN**: Separator comment used for visual grouping.
  **L220 CN**: 用于视觉分组的分隔注释。

### Lines 221-240

````c
 * Run through the pieces of data1->pw and data2->pw in order until they
 * have both been exhausted, picking the piece from data1->pw or data2->pw
 * depending on which should come first, together with the corresponding
 * domain from data1->cell or data2->cell.  In cases where the next pieces
 * in both data1->pw and data2->pw have the same function value expression,
 * construct only a single piece in the result with as domain
 * the union of the domains in data1->cell and data2->cell.
 */
static __isl_give PW *FN(PW,merge)(S(PW,union_opt_cmp_data) *data1,
	S(PW,union_opt_cmp_data) *data2)
{
	int i, j;
	PW *res;
	PW *pw1 = data1->pw;
	PW *pw2 = data2->pw;
	isl_set_list *list1 = data1->cell;
	isl_set_list *list2 = data2->cell;

	if (!pw1 || !pw2)
		return NULL;
````
- **L221 EN**: Comment explains nearby logic, invariants, or intent: `Run through the pieces of data1->pw and data2->pw in order until they`.
  **L221 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Run through the pieces of data1->pw and data2->pw in order until they`。
- **L222 EN**: Comment explains nearby logic, invariants, or intent: `have both been exhausted, picking the piece from data1->pw or data2->pw`.
  **L222 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`have both been exhausted, picking the piece from data1->pw or data2->pw`。
- **L223 EN**: Comment explains nearby logic, invariants, or intent: `depending on which should come first, together with the corresponding`.
  **L223 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`depending on which should come first, together with the corresponding`。
- **L224 EN**: Comment explains nearby logic, invariants, or intent: `domain from data1->cell or data2->cell.  In cases where the next pieces`.
  **L224 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`domain from data1->cell or data2->cell.  In cases where the next pieces`。
- **L225 EN**: Comment explains nearby logic, invariants, or intent: `in both data1->pw and data2->pw have the same function value expression,`.
  **L225 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`in both data1->pw and data2->pw have the same function value expression,`。
- **L226 EN**: Comment explains nearby logic, invariants, or intent: `construct only a single piece in the result with as domain`.
  **L226 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`construct only a single piece in the result with as domain`。
- **L227 EN**: Comment explains nearby logic, invariants, or intent: `the union of the domains in data1->cell and data2->cell.`.
  **L227 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the union of the domains in data1->cell and data2->cell.`。
- **L228 EN**: Separator comment used for visual grouping.
  **L228 CN**: 用于视觉分组的分隔注释。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __isl_give PW *FN(PW,merge)(S(PW,union_opt_cmp_data) *data1,`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __isl_give PW *FN(PW,merge)(S(PW,union_opt_cmp_data) *data1,`。
- **L230 EN**: Continues logic associated with callable symbol `S`.
  **L230 CN**: 继续与可调用符号 `S` 相关的逻辑。
- **L231 EN**: Opens a new lexical scope or compound statement.
  **L231 CN**: 打开一个新的词法作用域或复合语句块。
- **L232 EN**: Executes a standalone statement or declaration: `int i, j;`.
  **L232 CN**: 执行一条独立语句或声明：`int i, j;`。
- **L233 EN**: Executes a standalone statement or declaration: `PW *res;`.
  **L233 CN**: 执行一条独立语句或声明：`PW *res;`。
- **L234 EN**: Executes a standalone statement or declaration: `PW *pw1 = data1->pw;`.
  **L234 CN**: 执行一条独立语句或声明：`PW *pw1 = data1->pw;`。
- **L235 EN**: Executes a standalone statement or declaration: `PW *pw2 = data2->pw;`.
  **L235 CN**: 执行一条独立语句或声明：`PW *pw2 = data2->pw;`。
- **L236 EN**: Executes a standalone statement or declaration: `isl_set_list *list1 = data1->cell;`.
  **L236 CN**: 执行一条独立语句或声明：`isl_set_list *list1 = data1->cell;`。
- **L237 EN**: Executes a standalone statement or declaration: `isl_set_list *list2 = data2->cell;`.
  **L237 CN**: 执行一条独立语句或声明：`isl_set_list *list2 = data2->cell;`。
- **L238 EN**: Blank line separating nearby declarations or logic blocks.
  **L238 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L239 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L239 CN**: 开始 `if` 控制流语句并计算其条件。
- **L240 EN**: Returns from the current function with `NULL`.
  **L240 CN**: 以 `NULL` 从当前函数返回。

### Lines 241-260

````c

	res = FN(PW,alloc_size)(isl_space_copy(pw1->dim), pw1->n + pw2->n);

	i = 0; j = 0;
	while (i < pw1->n || j < pw2->n) {
		int cmp;
		isl_set *set;
		EL *el;

		if (i < pw1->n && j < pw2->n)
			cmp = FN(EL,plain_cmp)(pw1->p[i].FIELD,
						pw2->p[j].FIELD);
		else
			cmp = i < pw1->n ? -1 : 1;

		if (cmp < 0) {
			set = isl_set_list_get_set(list1, i);
			el = FN(EL,copy)(pw1->p[i].FIELD);
			++i;
		} else if (cmp > 0) {
````
- **L241 EN**: Blank line separating nearby declarations or logic blocks.
  **L241 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L242 EN**: Executes a call or declaration centered on `FN`.
  **L242 CN**: 执行以 `FN` 为核心的调用或声明。
- **L243 EN**: Blank line separating nearby declarations or logic blocks.
  **L243 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L244 EN**: Executes a standalone statement or declaration: `i = 0; j = 0;`.
  **L244 CN**: 执行一条独立语句或声明：`i = 0; j = 0;`。
- **L245 EN**: Begins a `while` control-flow statement and evaluates its condition.
  **L245 CN**: 开始 `while` 控制流语句并计算其条件。
- **L246 EN**: Executes a standalone statement or declaration: `int cmp;`.
  **L246 CN**: 执行一条独立语句或声明：`int cmp;`。
- **L247 EN**: Executes a standalone statement or declaration: `isl_set *set;`.
  **L247 CN**: 执行一条独立语句或声明：`isl_set *set;`。
- **L248 EN**: Executes a standalone statement or declaration: `EL *el;`.
  **L248 CN**: 执行一条独立语句或声明：`EL *el;`。
- **L249 EN**: Blank line separating nearby declarations or logic blocks.
  **L249 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L250 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L250 CN**: 开始 `if` 控制流语句并计算其条件。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `cmp = FN(EL,plain_cmp)(pw1->p[i].FIELD,`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`cmp = FN(EL,plain_cmp)(pw1->p[i].FIELD,`。
- **L252 EN**: Executes a standalone statement or declaration: `pw2->p[j].FIELD);`.
  **L252 CN**: 执行一条独立语句或声明：`pw2->p[j].FIELD);`。
- **L253 EN**: Starts the alternative branch of the preceding conditional.
  **L253 CN**: 开始前一个条件语句的备选分支。
- **L254 EN**: Executes a standalone statement or declaration: `cmp = i < pw1->n ? -1 : 1;`.
  **L254 CN**: 执行一条独立语句或声明：`cmp = i < pw1->n ? -1 : 1;`。
- **L255 EN**: Blank line separating nearby declarations or logic blocks.
  **L255 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L256 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L256 CN**: 开始 `if` 控制流语句并计算其条件。
- **L257 EN**: Executes a call or declaration centered on `isl_set_list_get_set`.
  **L257 CN**: 执行以 `isl_set_list_get_set` 为核心的调用或声明。
- **L258 EN**: Executes a call or declaration centered on `FN`.
  **L258 CN**: 执行以 `FN` 为核心的调用或声明。
- **L259 EN**: Executes a standalone statement or declaration: `++i;`.
  **L259 CN**: 执行一条独立语句或声明：`++i;`。
- **L260 EN**: Starts a function, helper, or structured scope: `} else if (cmp > 0) {`.
  **L260 CN**: 开始一个函数、辅助例程或结构化作用域：`} else if (cmp > 0) {`。

### Lines 261-280

````c
			set = isl_set_list_get_set(list2, j);
			el = FN(EL,copy)(pw2->p[j].FIELD);
			++j;
		} else {
			set = isl_set_union(isl_set_list_get_set(list1, i),
					    isl_set_list_get_set(list2, j));
			el = FN(EL,copy)(pw1->p[i].FIELD);
			++i;
			++j;
		}
		res = FN(PW,add_piece)(res, set, el);
	}

	return res;
}

/* Given a function "cmp" that returns the set of elements where
 * "el1" is "better" than "el2", return a piecewise
 * expression defined on the union of the definition domains
 * of "pw1" and "pw2" that maps to the "best" of "pw1" and
````
- **L261 EN**: Executes a call or declaration centered on `isl_set_list_get_set`.
  **L261 CN**: 执行以 `isl_set_list_get_set` 为核心的调用或声明。
- **L262 EN**: Executes a call or declaration centered on `FN`.
  **L262 CN**: 执行以 `FN` 为核心的调用或声明。
- **L263 EN**: Executes a standalone statement or declaration: `++j;`.
  **L263 CN**: 执行一条独立语句或声明：`++j;`。
- **L264 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L264 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `set = isl_set_union(isl_set_list_get_set(list1, i),`.
  **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`set = isl_set_union(isl_set_list_get_set(list1, i),`。
- **L266 EN**: Executes a call or declaration centered on `isl_set_list_get_set`.
  **L266 CN**: 执行以 `isl_set_list_get_set` 为核心的调用或声明。
- **L267 EN**: Executes a call or declaration centered on `FN`.
  **L267 CN**: 执行以 `FN` 为核心的调用或声明。
- **L268 EN**: Executes a standalone statement or declaration: `++i;`.
  **L268 CN**: 执行一条独立语句或声明：`++i;`。
- **L269 EN**: Executes a standalone statement or declaration: `++j;`.
  **L269 CN**: 执行一条独立语句或声明：`++j;`。
- **L270 EN**: Closes the current lexical scope or compound statement.
  **L270 CN**: 结束当前词法作用域或复合语句块。
- **L271 EN**: Executes a call or declaration centered on `FN`.
  **L271 CN**: 执行以 `FN` 为核心的调用或声明。
- **L272 EN**: Closes the current lexical scope or compound statement.
  **L272 CN**: 结束当前词法作用域或复合语句块。
- **L273 EN**: Blank line separating nearby declarations or logic blocks.
  **L273 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L274 EN**: Returns from the current function with `res`.
  **L274 CN**: 以 `res` 从当前函数返回。
- **L275 EN**: Closes the current lexical scope or compound statement.
  **L275 CN**: 结束当前词法作用域或复合语句块。
- **L276 EN**: Blank line separating nearby declarations or logic blocks.
  **L276 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L277 EN**: Comment explains nearby logic, invariants, or intent: `Given a function "cmp" that returns the set of elements where`.
  **L277 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Given a function "cmp" that returns the set of elements where`。
- **L278 EN**: Comment explains nearby logic, invariants, or intent: `"el1" is "better" than "el2", return a piecewise`.
  **L278 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`"el1" is "better" than "el2", return a piecewise`。
- **L279 EN**: Comment explains nearby logic, invariants, or intent: `expression defined on the union of the definition domains`.
  **L279 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`expression defined on the union of the definition domains`。
- **L280 EN**: Comment explains nearby logic, invariants, or intent: `of "pw1" and "pw2" that maps to the "best" of "pw1" and`.
  **L280 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`of "pw1" and "pw2" that maps to the "best" of "pw1" and`。

### Lines 281-300

````c
 * "pw2" on each cell.  If only one of the two input functions
 * is defined on a given cell, then it is considered the best.
 *
 * Run through all pairs of pieces in "pw1" and "pw2".
 * If the domains of these pieces intersect, then the intersection
 * needs to be distributed over the two pieces based on "cmp".
 *
 * After the updated domains have been computed, the result is constructed
 * from "pw1", "pw2", data[0].cell and data[1].cell.  If there are any pieces
 * in "pw1" and "pw2" with the same function value expression, then
 * they are combined into a single piece in the result.
 * In order to be able to do this efficiently, the pieces of "pw1" and
 * "pw2" are first sorted according to their function value expressions.
 */
static __isl_give PW *FN(PW,union_opt_cmp)(
	__isl_take PW *pw1, __isl_take PW *pw2,
	__isl_give isl_set *(*cmp)(__isl_take EL *el1, __isl_take EL *el2))
{
	S(PW,union_opt_cmp_data) data[2] = { { pw1, NULL }, { pw2, NULL } };
	int i, j;
````
- **L281 EN**: Comment explains nearby logic, invariants, or intent: `"pw2" on each cell.  If only one of the two input functions`.
  **L281 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`"pw2" on each cell.  If only one of the two input functions`。
- **L282 EN**: Comment explains nearby logic, invariants, or intent: `is defined on a given cell, then it is considered the best.`.
  **L282 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`is defined on a given cell, then it is considered the best.`。
- **L283 EN**: Separator comment used for visual grouping.
  **L283 CN**: 用于视觉分组的分隔注释。
- **L284 EN**: Comment explains nearby logic, invariants, or intent: `Run through all pairs of pieces in "pw1" and "pw2".`.
  **L284 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Run through all pairs of pieces in "pw1" and "pw2".`。
- **L285 EN**: Comment explains nearby logic, invariants, or intent: `If the domains of these pieces intersect, then the intersection`.
  **L285 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If the domains of these pieces intersect, then the intersection`。
- **L286 EN**: Comment explains nearby logic, invariants, or intent: `needs to be distributed over the two pieces based on "cmp".`.
  **L286 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`needs to be distributed over the two pieces based on "cmp".`。
- **L287 EN**: Separator comment used for visual grouping.
  **L287 CN**: 用于视觉分组的分隔注释。
- **L288 EN**: Comment explains nearby logic, invariants, or intent: `After the updated domains have been computed, the result is constructed`.
  **L288 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`After the updated domains have been computed, the result is constructed`。
- **L289 EN**: Comment explains nearby logic, invariants, or intent: `from "pw1", "pw2", data[0].cell and data[1].cell.  If there are any pieces`.
  **L289 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`from "pw1", "pw2", data[0].cell and data[1].cell.  If there are any pieces`。
- **L290 EN**: Comment explains nearby logic, invariants, or intent: `in "pw1" and "pw2" with the same function value expression, then`.
  **L290 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`in "pw1" and "pw2" with the same function value expression, then`。
- **L291 EN**: Comment explains nearby logic, invariants, or intent: `they are combined into a single piece in the result.`.
  **L291 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`they are combined into a single piece in the result.`。
- **L292 EN**: Comment explains nearby logic, invariants, or intent: `In order to be able to do this efficiently, the pieces of "pw1" and`.
  **L292 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`In order to be able to do this efficiently, the pieces of "pw1" and`。
- **L293 EN**: Comment explains nearby logic, invariants, or intent: `"pw2" are first sorted according to their function value expressions.`.
  **L293 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`"pw2" are first sorted according to their function value expressions.`。
- **L294 EN**: Separator comment used for visual grouping.
  **L294 CN**: 用于视觉分组的分隔注释。
- **L295 EN**: Continues logic associated with callable symbol `FN`.
  **L295 CN**: 继续与可调用符号 `FN` 相关的逻辑。
- **L296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__isl_take PW *pw1, __isl_take PW *pw2,`.
  **L296 CN**: 继续一个多行参数列表、初始化器或聚合项：`__isl_take PW *pw1, __isl_take PW *pw2,`。
- **L297 EN**: Continues the surrounding expression or declaration: `__isl_give isl_set *(*cmp)(__isl_take EL *el1, __isl_take EL *el2))`.
  **L297 CN**: 继续构造周围的表达式或声明：`__isl_give isl_set *(*cmp)(__isl_take EL *el1, __isl_take EL *el2))`。
- **L298 EN**: Opens a new lexical scope or compound statement.
  **L298 CN**: 打开一个新的词法作用域或复合语句块。
- **L299 EN**: Executes a call or declaration centered on `S`.
  **L299 CN**: 执行以 `S` 为核心的调用或声明。
- **L300 EN**: Executes a standalone statement or declaration: `int i, j;`.
  **L300 CN**: 执行一条独立语句或声明：`int i, j;`。

### Lines 301-320

````c
	isl_size n1, n2;
	PW *res = NULL;
	isl_ctx *ctx;

	if (!pw1 || !pw2)
		goto error;

	ctx = isl_space_get_ctx(pw1->dim);
	if (!isl_space_is_equal(pw1->dim, pw2->dim))
		isl_die(ctx, isl_error_invalid,
			"arguments should live in the same space", goto error);

	if (FN(PW,is_empty)(pw1)) {
		FN(PW,free)(pw1);
		return pw2;
	}

	if (FN(PW,is_empty)(pw2)) {
		FN(PW,free)(pw2);
		return pw1;
````
- **L301 EN**: Executes a standalone statement or declaration: `isl_size n1, n2;`.
  **L301 CN**: 执行一条独立语句或声明：`isl_size n1, n2;`。
- **L302 EN**: Executes a standalone statement or declaration: `PW *res = NULL;`.
  **L302 CN**: 执行一条独立语句或声明：`PW *res = NULL;`。
- **L303 EN**: Executes a standalone statement or declaration: `isl_ctx *ctx;`.
  **L303 CN**: 执行一条独立语句或声明：`isl_ctx *ctx;`。
- **L304 EN**: Blank line separating nearby declarations or logic blocks.
  **L304 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L305 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L305 CN**: 开始 `if` 控制流语句并计算其条件。
- **L306 EN**: Jumps to label `error` for structured cleanup or control transfer.
  **L306 CN**: 跳转到标签 `error`，用于结构化清理或控制转移。
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L308 EN**: Executes a call or declaration centered on `isl_space_get_ctx`.
  **L308 CN**: 执行以 `isl_space_get_ctx` 为核心的调用或声明。
- **L309 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L309 CN**: 开始 `if` 控制流语句并计算其条件。
- **L310 EN**: Reports an isl error and typically aborts the current operation.
  **L310 CN**: 报告一个 isl 错误，并通常终止当前操作。
- **L311 EN**: Executes a standalone statement or declaration: `"arguments should live in the same space", goto error);`.
  **L311 CN**: 执行一条独立语句或声明：`"arguments should live in the same space", goto error);`。
- **L312 EN**: Blank line separating nearby declarations or logic blocks.
  **L312 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L313 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L313 CN**: 开始 `if` 控制流语句并计算其条件。
- **L314 EN**: Executes a call or declaration centered on `FN`.
  **L314 CN**: 执行以 `FN` 为核心的调用或声明。
- **L315 EN**: Returns from the current function with `pw2`.
  **L315 CN**: 以 `pw2` 从当前函数返回。
- **L316 EN**: Closes the current lexical scope or compound statement.
  **L316 CN**: 结束当前词法作用域或复合语句块。
- **L317 EN**: Blank line separating nearby declarations or logic blocks.
  **L317 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L318 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L318 CN**: 开始 `if` 控制流语句并计算其条件。
- **L319 EN**: Executes a call or declaration centered on `FN`.
  **L319 CN**: 执行以 `FN` 为核心的调用或声明。
- **L320 EN**: Returns from the current function with `pw1`.
  **L320 CN**: 以 `pw1` 从当前函数返回。

### Lines 321-340

````c
	}

	for (i = 0; i < 2; ++i) {
		data[i].pw = FN(PW,sort_unique)(data[i].pw);
		data[i].cell = FN(PW,extract_domains)(data[i].pw);
	}

	n1 = FN(PW,n_piece)(data[0].pw);
	n2 = FN(PW,n_piece)(data[1].pw);
	if (n1 < 0 || n2 < 0)
		goto error;
	for (i = 0; i < n1; ++i) {
		for (j = 0; j < n2; ++j) {
			isl_bool disjoint;
			isl_set *set_i, *set_j;

			set_i = FN(PW,peek_domain_at)(data[0].pw, i);
			set_j = FN(PW,peek_domain_at)(data[1].pw, j);
			disjoint = isl_set_is_disjoint(set_i, set_j);
			if (disjoint < 0)
````
- **L321 EN**: Closes the current lexical scope or compound statement.
  **L321 CN**: 结束当前词法作用域或复合语句块。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L323 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L323 CN**: 开始 `for` 控制流语句并计算其条件。
- **L324 EN**: Executes a call or declaration centered on `FN`.
  **L324 CN**: 执行以 `FN` 为核心的调用或声明。
- **L325 EN**: Executes a call or declaration centered on `FN`.
  **L325 CN**: 执行以 `FN` 为核心的调用或声明。
- **L326 EN**: Closes the current lexical scope or compound statement.
  **L326 CN**: 结束当前词法作用域或复合语句块。
- **L327 EN**: Blank line separating nearby declarations or logic blocks.
  **L327 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L328 EN**: Executes a call or declaration centered on `FN`.
  **L328 CN**: 执行以 `FN` 为核心的调用或声明。
- **L329 EN**: Executes a call or declaration centered on `FN`.
  **L329 CN**: 执行以 `FN` 为核心的调用或声明。
- **L330 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L330 CN**: 开始 `if` 控制流语句并计算其条件。
- **L331 EN**: Jumps to label `error` for structured cleanup or control transfer.
  **L331 CN**: 跳转到标签 `error`，用于结构化清理或控制转移。
- **L332 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L332 CN**: 开始 `for` 控制流语句并计算其条件。
- **L333 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L333 CN**: 开始 `for` 控制流语句并计算其条件。
- **L334 EN**: Executes a standalone statement or declaration: `isl_bool disjoint;`.
  **L334 CN**: 执行一条独立语句或声明：`isl_bool disjoint;`。
- **L335 EN**: Executes a standalone statement or declaration: `isl_set *set_i, *set_j;`.
  **L335 CN**: 执行一条独立语句或声明：`isl_set *set_i, *set_j;`。
- **L336 EN**: Blank line separating nearby declarations or logic blocks.
  **L336 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L337 EN**: Executes a call or declaration centered on `FN`.
  **L337 CN**: 执行以 `FN` 为核心的调用或声明。
- **L338 EN**: Executes a call or declaration centered on `FN`.
  **L338 CN**: 执行以 `FN` 为核心的调用或声明。
- **L339 EN**: Executes a call or declaration centered on `isl_set_is_disjoint`.
  **L339 CN**: 执行以 `isl_set_is_disjoint` 为核心的调用或声明。
- **L340 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L340 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 341-359

````c
				goto error;
			if (disjoint)
				continue;
			if (FN(PW,union_opt_cmp_two)(&data[0], i,
							&data[1], j, cmp) < 0)
				goto error;
		}
	}

	res = FN(PW,merge)(&data[0], &data[1]);
	for (i = 0; i < 2; ++i)
		FN(PW,union_opt_cmp_data_clear)(&data[i]);

	return res;
error:
	for (i = 0; i < 2; ++i)
		FN(PW,union_opt_cmp_data_clear)(&data[i]);
	return FN(PW,free)(res);
}
````
- **L341 EN**: Jumps to label `error` for structured cleanup or control transfer.
  **L341 CN**: 跳转到标签 `error`，用于结构化清理或控制转移。
- **L342 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L342 CN**: 开始 `if` 控制流语句并计算其条件。
- **L343 EN**: Skips to the next loop iteration.
  **L343 CN**: 跳到下一次循环迭代。
- **L344 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L344 CN**: 开始 `if` 控制流语句并计算其条件。
- **L345 EN**: Continues the surrounding expression or declaration: `&data[1], j, cmp) < 0)`.
  **L345 CN**: 继续构造周围的表达式或声明：`&data[1], j, cmp) < 0)`。
- **L346 EN**: Jumps to label `error` for structured cleanup or control transfer.
  **L346 CN**: 跳转到标签 `error`，用于结构化清理或控制转移。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Closes the current lexical scope or compound statement.
  **L348 CN**: 结束当前词法作用域或复合语句块。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L350 EN**: Executes a call or declaration centered on `FN`.
  **L350 CN**: 执行以 `FN` 为核心的调用或声明。
- **L351 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L351 CN**: 开始 `for` 控制流语句并计算其条件。
- **L352 EN**: Executes a call or declaration centered on `FN`.
  **L352 CN**: 执行以 `FN` 为核心的调用或声明。
- **L353 EN**: Blank line separating nearby declarations or logic blocks.
  **L353 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L354 EN**: Returns from the current function with `res`.
  **L354 CN**: 以 `res` 从当前函数返回。
- **L355 EN**: Defines a local jump label `error`.
  **L355 CN**: 定义一个本地跳转标签 `error`。
- **L356 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L356 CN**: 开始 `for` 控制流语句并计算其条件。
- **L357 EN**: Executes a call or declaration centered on `FN`.
  **L357 CN**: 执行以 `FN` 为核心的调用或声明。
- **L358 EN**: Returns from the current function with `FN(PW,free)(res)`.
  **L358 CN**: 以 `FN(PW,free)(res)` 从当前函数返回。
- **L359 EN**: Closes the current lexical scope or compound statement.
  **L359 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Polyhedral integer-set operations / 多面体整数集合操作**
- **Set and basic-set reasoning / 集合与基本集合推理**
- **Dimension and space metadata / 维度与空间元数据**
- **Identifier attachment and lookup / 标识符附着与查找**
- **Arbitrary-precision numeric values / 任意精度数值**

## Dependencies / 依赖关系

- `isl_pw_macro.h`: Provides local isl declarations paired with this implementation file. / 提供与该实现文件配套的本地 isl 声明。
