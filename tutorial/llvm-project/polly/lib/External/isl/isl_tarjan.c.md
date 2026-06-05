# isl_tarjan.c — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `polly/lib/External/isl/isl_tarjan.c`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Implements ordering and graph-structure utilities for Polly's bundled Integer Set Library (isl).
- **Purpose (CN)**: 为 Polly 内置的整数集合库（isl）实现排序与图结构工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*
 * Copyright 2010-2011 INRIA Saclay
 * Copyright 2012      Ecole Normale Superieure
 *
 * Use of this software is governed by the MIT license
 *
 * Written by Sven Verdoolaege, INRIA Saclay - Ile-de-France,
 * Parc Club Orsay Universite, ZAC des vignes, 4 rue Jacques Monod,
 * 91893 Orsay, France
 * and Ecole Normale Superieure, 45 rue d’Ulm, 75230 Paris, France
 */

#include <stdlib.h>
#include <isl/ctx.h>
#include <isl_tarjan.h>

struct isl_tarjan_graph *isl_tarjan_graph_free(struct isl_tarjan_graph *g)
{
	if (!g)
		return NULL;
````
- **L1 EN**: Separator comment used for visual grouping.
  **L1 CN**: 用于视觉分组的分隔注释。
- **L2 EN**: Comment explains nearby logic, invariants, or intent: `Copyright 2010-2011 INRIA Saclay`.
  **L2 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copyright 2010-2011 INRIA Saclay`。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Copyright 2012      Ecole Normale Superieure`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Copyright 2012      Ecole Normale Superieure`。
- **L4 EN**: Separator comment used for visual grouping.
  **L4 CN**: 用于视觉分组的分隔注释。
- **L5 EN**: Comment explains nearby logic, invariants, or intent: `Use of this software is governed by the MIT license`.
  **L5 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Use of this software is governed by the MIT license`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Comment explains nearby logic, invariants, or intent: `Written by Sven Verdoolaege, INRIA Saclay - Ile-de-France,`.
  **L7 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Written by Sven Verdoolaege, INRIA Saclay - Ile-de-France,`。
- **L8 EN**: Comment explains nearby logic, invariants, or intent: `Parc Club Orsay Universite, ZAC des vignes, 4 rue Jacques Monod,`.
  **L8 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Parc Club Orsay Universite, ZAC des vignes, 4 rue Jacques Monod,`。
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `91893 Orsay, France`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`91893 Orsay, France`。
- **L10 EN**: Comment explains nearby logic, invariants, or intent: `and Ecole Normale Superieure, 45 rue d’Ulm, 75230 Paris, France`.
  **L10 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and Ecole Normale Superieure, 45 rue d’Ulm, 75230 Paris, France`。
- **L11 EN**: Separator comment used for visual grouping.
  **L11 CN**: 用于视觉分组的分隔注释。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Includes <stdlib.h> to access standard C library facilities.
  **L13 CN**: 引入 <stdlib.h> 以使用标准 C 库功能。
- **L14 EN**: Includes <isl/ctx.h> to access public isl interfaces imported by this file.
  **L14 CN**: 引入 <isl/ctx.h> 以使用该文件使用的公开 isl 接口。
- **L15 EN**: Includes <isl_tarjan.h> to access local isl declarations paired with this implementation file.
  **L15 CN**: 引入 <isl_tarjan.h> 以使用与该实现文件配套的本地 isl 声明。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L17 EN**: Declares struct `isl_tarjan_graph`.
  **L17 CN**: 声明 struct `isl_tarjan_graph`。
- **L18 EN**: Opens a new lexical scope or compound statement.
  **L18 CN**: 打开一个新的词法作用域或复合语句块。
- **L19 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L19 CN**: 开始 `if` 控制流语句并计算其条件。
- **L20 EN**: Returns from the current function with `NULL`.
  **L20 CN**: 以 `NULL` 从当前函数返回。

### Lines 21-40

````c
	free(g->node);
	free(g->stack);
	free(g->order);
	free(g);
	return NULL;
}

static struct isl_tarjan_graph *isl_tarjan_graph_alloc(isl_ctx *ctx, int len)
{
	struct isl_tarjan_graph *g;
	int i;

	g = isl_calloc_type(ctx, struct isl_tarjan_graph);
	if (!g)
		return NULL;
	g->len = len;
	g->node = isl_alloc_array(ctx, struct isl_tarjan_node, len);
	if (len && !g->node)
		goto error;
	for (i = 0; i < len; ++i)
````
- **L21 EN**: Executes a call or declaration centered on `free`.
  **L21 CN**: 执行以 `free` 为核心的调用或声明。
- **L22 EN**: Executes a call or declaration centered on `free`.
  **L22 CN**: 执行以 `free` 为核心的调用或声明。
- **L23 EN**: Executes a call or declaration centered on `free`.
  **L23 CN**: 执行以 `free` 为核心的调用或声明。
- **L24 EN**: Executes a call or declaration centered on `free`.
  **L24 CN**: 执行以 `free` 为核心的调用或声明。
- **L25 EN**: Returns from the current function with `NULL`.
  **L25 CN**: 以 `NULL` 从当前函数返回。
- **L26 EN**: Closes the current lexical scope or compound statement.
  **L26 CN**: 结束当前词法作用域或复合语句块。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L28 EN**: Continues logic associated with callable symbol `isl_tarjan_graph_alloc`.
  **L28 CN**: 继续与可调用符号 `isl_tarjan_graph_alloc` 相关的逻辑。
- **L29 EN**: Opens a new lexical scope or compound statement.
  **L29 CN**: 打开一个新的词法作用域或复合语句块。
- **L30 EN**: Declares struct `isl_tarjan_graph`.
  **L30 CN**: 声明 struct `isl_tarjan_graph`。
- **L31 EN**: Executes a standalone statement or declaration: `int i;`.
  **L31 CN**: 执行一条独立语句或声明：`int i;`。
- **L32 EN**: Blank line separating nearby declarations or logic blocks.
  **L32 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L33 EN**: Executes a call or declaration centered on `isl_calloc_type`.
  **L33 CN**: 执行以 `isl_calloc_type` 为核心的调用或声明。
- **L34 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L34 CN**: 开始 `if` 控制流语句并计算其条件。
- **L35 EN**: Returns from the current function with `NULL`.
  **L35 CN**: 以 `NULL` 从当前函数返回。
- **L36 EN**: Executes a standalone statement or declaration: `g->len = len;`.
  **L36 CN**: 执行一条独立语句或声明：`g->len = len;`。
- **L37 EN**: Executes a call or declaration centered on `isl_alloc_array`.
  **L37 CN**: 执行以 `isl_alloc_array` 为核心的调用或声明。
- **L38 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L38 CN**: 开始 `if` 控制流语句并计算其条件。
- **L39 EN**: Jumps to label `error` for structured cleanup or control transfer.
  **L39 CN**: 跳转到标签 `error`，用于结构化清理或控制转移。
- **L40 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L40 CN**: 开始 `for` 控制流语句并计算其条件。

### Lines 41-60

````c
		g->node[i].index = -1;
	g->stack = isl_alloc_array(ctx, int, len);
	if (len && !g->stack)
		goto error;
	g->order = isl_alloc_array(ctx, int, 2 * len);
	if (len && !g->order)
		goto error;

	g->sp = 0;
	g->index = 0;
	g->op = 0;

	return g;
error:
	isl_tarjan_graph_free(g);
	return NULL;
}

/* Perform Tarjan's algorithm for computing the strongly connected components
 * in the graph with g->len nodes and with edges defined by "follows".
````
- **L41 EN**: Executes a standalone statement or declaration: `g->node[i].index = -1;`.
  **L41 CN**: 执行一条独立语句或声明：`g->node[i].index = -1;`。
- **L42 EN**: Executes a call or declaration centered on `isl_alloc_array`.
  **L42 CN**: 执行以 `isl_alloc_array` 为核心的调用或声明。
- **L43 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L43 CN**: 开始 `if` 控制流语句并计算其条件。
- **L44 EN**: Jumps to label `error` for structured cleanup or control transfer.
  **L44 CN**: 跳转到标签 `error`，用于结构化清理或控制转移。
- **L45 EN**: Executes a call or declaration centered on `isl_alloc_array`.
  **L45 CN**: 执行以 `isl_alloc_array` 为核心的调用或声明。
- **L46 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L46 CN**: 开始 `if` 控制流语句并计算其条件。
- **L47 EN**: Jumps to label `error` for structured cleanup or control transfer.
  **L47 CN**: 跳转到标签 `error`，用于结构化清理或控制转移。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L49 EN**: Executes a standalone statement or declaration: `g->sp = 0;`.
  **L49 CN**: 执行一条独立语句或声明：`g->sp = 0;`。
- **L50 EN**: Executes a standalone statement or declaration: `g->index = 0;`.
  **L50 CN**: 执行一条独立语句或声明：`g->index = 0;`。
- **L51 EN**: Executes a standalone statement or declaration: `g->op = 0;`.
  **L51 CN**: 执行一条独立语句或声明：`g->op = 0;`。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L53 EN**: Returns from the current function with `g`.
  **L53 CN**: 以 `g` 从当前函数返回。
- **L54 EN**: Defines a local jump label `error`.
  **L54 CN**: 定义一个本地跳转标签 `error`。
- **L55 EN**: Executes a call or declaration centered on `isl_tarjan_graph_free`.
  **L55 CN**: 执行以 `isl_tarjan_graph_free` 为核心的调用或声明。
- **L56 EN**: Returns from the current function with `NULL`.
  **L56 CN**: 以 `NULL` 从当前函数返回。
- **L57 EN**: Closes the current lexical scope or compound statement.
  **L57 CN**: 结束当前词法作用域或复合语句块。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L59 EN**: Comment explains nearby logic, invariants, or intent: `Perform Tarjan's algorithm for computing the strongly connected components`.
  **L59 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Perform Tarjan's algorithm for computing the strongly connected components`。
- **L60 EN**: Comment explains nearby logic, invariants, or intent: `in the graph with g->len nodes and with edges defined by "follows".`.
  **L60 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`in the graph with g->len nodes and with edges defined by "follows".`。

### Lines 61-80

````c
 */
static isl_stat isl_tarjan_components(struct isl_tarjan_graph *g, int i,
	isl_bool (*follows)(int i, int j, void *user), void *user)
{
	int j;

	g->node[i].index = g->index;
	g->node[i].min_index = g->index;
	g->node[i].on_stack = 1;
	g->index++;
	g->stack[g->sp++] = i;

	for (j = g->len - 1; j >= 0; --j) {
		isl_bool f;

		if (j == i)
			continue;
		if (g->node[j].index >= 0 &&
			(!g->node[j].on_stack ||
			 g->node[j].index > g->node[i].min_index))
````
- **L61 EN**: Separator comment used for visual grouping.
  **L61 CN**: 用于视觉分组的分隔注释。
- **L62 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static isl_stat isl_tarjan_components(struct isl_tarjan_graph *g, int i,`.
  **L62 CN**: 继续一个多行参数列表、初始化器或聚合项：`static isl_stat isl_tarjan_components(struct isl_tarjan_graph *g, int i,`。
- **L63 EN**: Continues logic associated with callable symbol `isl_bool`.
  **L63 CN**: 继续与可调用符号 `isl_bool` 相关的逻辑。
- **L64 EN**: Opens a new lexical scope or compound statement.
  **L64 CN**: 打开一个新的词法作用域或复合语句块。
- **L65 EN**: Executes a standalone statement or declaration: `int j;`.
  **L65 CN**: 执行一条独立语句或声明：`int j;`。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L67 EN**: Executes a standalone statement or declaration: `g->node[i].index = g->index;`.
  **L67 CN**: 执行一条独立语句或声明：`g->node[i].index = g->index;`。
- **L68 EN**: Executes a standalone statement or declaration: `g->node[i].min_index = g->index;`.
  **L68 CN**: 执行一条独立语句或声明：`g->node[i].min_index = g->index;`。
- **L69 EN**: Executes a standalone statement or declaration: `g->node[i].on_stack = 1;`.
  **L69 CN**: 执行一条独立语句或声明：`g->node[i].on_stack = 1;`。
- **L70 EN**: Executes a standalone statement or declaration: `g->index++;`.
  **L70 CN**: 执行一条独立语句或声明：`g->index++;`。
- **L71 EN**: Executes a standalone statement or declaration: `g->stack[g->sp++] = i;`.
  **L71 CN**: 执行一条独立语句或声明：`g->stack[g->sp++] = i;`。
- **L72 EN**: Blank line separating nearby declarations or logic blocks.
  **L72 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L73 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L73 CN**: 开始 `for` 控制流语句并计算其条件。
- **L74 EN**: Executes a standalone statement or declaration: `isl_bool f;`.
  **L74 CN**: 执行一条独立语句或声明：`isl_bool f;`。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L76 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L76 CN**: 开始 `if` 控制流语句并计算其条件。
- **L77 EN**: Skips to the next loop iteration.
  **L77 CN**: 跳到下一次循环迭代。
- **L78 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L78 CN**: 开始 `if` 控制流语句并计算其条件。
- **L79 EN**: Continues the surrounding expression or declaration: `(!g->node[j].on_stack ||`.
  **L79 CN**: 继续构造周围的表达式或声明：`(!g->node[j].on_stack ||`。
- **L80 EN**: Continues the surrounding expression or declaration: `g->node[j].index > g->node[i].min_index))`.
  **L80 CN**: 继续构造周围的表达式或声明：`g->node[j].index > g->node[i].min_index))`。

### Lines 81-100

````c
			continue;

		f = follows(i, j, user);
		if (f < 0)
			return isl_stat_error;
		if (!f)
			continue;

		if (g->node[j].index < 0) {
			isl_tarjan_components(g, j, follows, user);
			if (g->node[j].min_index < g->node[i].min_index)
				g->node[i].min_index = g->node[j].min_index;
		} else if (g->node[j].index < g->node[i].min_index)
			g->node[i].min_index = g->node[j].index;
	}

	if (g->node[i].index != g->node[i].min_index)
		return isl_stat_ok;

	do {
````
- **L81 EN**: Skips to the next loop iteration.
  **L81 CN**: 跳到下一次循环迭代。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L83 EN**: Executes a call or declaration centered on `follows`.
  **L83 CN**: 执行以 `follows` 为核心的调用或声明。
- **L84 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L84 CN**: 开始 `if` 控制流语句并计算其条件。
- **L85 EN**: Returns from the current function with `isl_stat_error`.
  **L85 CN**: 以 `isl_stat_error` 从当前函数返回。
- **L86 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L86 CN**: 开始 `if` 控制流语句并计算其条件。
- **L87 EN**: Skips to the next loop iteration.
  **L87 CN**: 跳到下一次循环迭代。
- **L88 EN**: Blank line separating nearby declarations or logic blocks.
  **L88 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L89 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L89 CN**: 开始 `if` 控制流语句并计算其条件。
- **L90 EN**: Executes a call or declaration centered on `isl_tarjan_components`.
  **L90 CN**: 执行以 `isl_tarjan_components` 为核心的调用或声明。
- **L91 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L91 CN**: 开始 `if` 控制流语句并计算其条件。
- **L92 EN**: Executes a standalone statement or declaration: `g->node[i].min_index = g->node[j].min_index;`.
  **L92 CN**: 执行一条独立语句或声明：`g->node[i].min_index = g->node[j].min_index;`。
- **L93 EN**: Continues the surrounding expression or declaration: `} else if (g->node[j].index < g->node[i].min_index)`.
  **L93 CN**: 继续构造周围的表达式或声明：`} else if (g->node[j].index < g->node[i].min_index)`。
- **L94 EN**: Executes a standalone statement or declaration: `g->node[i].min_index = g->node[j].index;`.
  **L94 CN**: 执行一条独立语句或声明：`g->node[i].min_index = g->node[j].index;`。
- **L95 EN**: Closes the current lexical scope or compound statement.
  **L95 CN**: 结束当前词法作用域或复合语句块。
- **L96 EN**: Blank line separating nearby declarations or logic blocks.
  **L96 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L97 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L97 CN**: 开始 `if` 控制流语句并计算其条件。
- **L98 EN**: Returns from the current function with `isl_stat_ok`.
  **L98 CN**: 以 `isl_stat_ok` 从当前函数返回。
- **L99 EN**: Blank line separating nearby declarations or logic blocks.
  **L99 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L100 EN**: Continues the surrounding expression or declaration: `do {`.
  **L100 CN**: 继续构造周围的表达式或声明：`do {`。

### Lines 101-120

````c
		j = g->stack[--g->sp];
		g->node[j].on_stack = 0;
		g->order[g->op++] = j;
	} while (j != i);
	g->order[g->op++] = -1;

	return isl_stat_ok;
}

/* Decompose the graph with "len" nodes and edges defined by "follows"
 * into strongly connected components (SCCs).
 * follows(i, j, user) should return 1 if "i" follows "j" and 0 otherwise.
 * It should return -1 on error.
 *
 * If SCC a contains a node i that follows a node j in another SCC b
 * (i.e., follows(i, j, user) returns 1), then SCC a will appear after SCC b
 * in the result.
 */
struct isl_tarjan_graph *isl_tarjan_graph_init(isl_ctx *ctx, int len,
	isl_bool (*follows)(int i, int j, void *user), void *user)
````
- **L101 EN**: Executes a standalone statement or declaration: `j = g->stack[--g->sp];`.
  **L101 CN**: 执行一条独立语句或声明：`j = g->stack[--g->sp];`。
- **L102 EN**: Executes a standalone statement or declaration: `g->node[j].on_stack = 0;`.
  **L102 CN**: 执行一条独立语句或声明：`g->node[j].on_stack = 0;`。
- **L103 EN**: Executes a standalone statement or declaration: `g->order[g->op++] = j;`.
  **L103 CN**: 执行一条独立语句或声明：`g->order[g->op++] = j;`。
- **L104 EN**: Executes a call or declaration centered on `while`.
  **L104 CN**: 执行以 `while` 为核心的调用或声明。
- **L105 EN**: Executes a standalone statement or declaration: `g->order[g->op++] = -1;`.
  **L105 CN**: 执行一条独立语句或声明：`g->order[g->op++] = -1;`。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L107 EN**: Returns from the current function with `isl_stat_ok`.
  **L107 CN**: 以 `isl_stat_ok` 从当前函数返回。
- **L108 EN**: Closes the current lexical scope or compound statement.
  **L108 CN**: 结束当前词法作用域或复合语句块。
- **L109 EN**: Blank line separating nearby declarations or logic blocks.
  **L109 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L110 EN**: Comment explains nearby logic, invariants, or intent: `Decompose the graph with "len" nodes and edges defined by "follows"`.
  **L110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Decompose the graph with "len" nodes and edges defined by "follows"`。
- **L111 EN**: Comment explains nearby logic, invariants, or intent: `into strongly connected components (SCCs).`.
  **L111 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`into strongly connected components (SCCs).`。
- **L112 EN**: Comment explains nearby logic, invariants, or intent: `follows(i, j, user) should return 1 if "i" follows "j" and 0 otherwise.`.
  **L112 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`follows(i, j, user) should return 1 if "i" follows "j" and 0 otherwise.`。
- **L113 EN**: Comment explains nearby logic, invariants, or intent: `It should return -1 on error.`.
  **L113 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`It should return -1 on error.`。
- **L114 EN**: Separator comment used for visual grouping.
  **L114 CN**: 用于视觉分组的分隔注释。
- **L115 EN**: Comment explains nearby logic, invariants, or intent: `If SCC a contains a node i that follows a node j in another SCC b`.
  **L115 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If SCC a contains a node i that follows a node j in another SCC b`。
- **L116 EN**: Comment explains nearby logic, invariants, or intent: `(i.e., follows(i, j, user) returns 1), then SCC a will appear after SCC b`.
  **L116 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`(i.e., follows(i, j, user) returns 1), then SCC a will appear after SCC b`。
- **L117 EN**: Comment explains nearby logic, invariants, or intent: `in the result.`.
  **L117 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`in the result.`。
- **L118 EN**: Separator comment used for visual grouping.
  **L118 CN**: 用于视觉分组的分隔注释。
- **L119 EN**: Declares struct `isl_tarjan_graph`.
  **L119 CN**: 声明 struct `isl_tarjan_graph`。
- **L120 EN**: Continues logic associated with callable symbol `isl_bool`.
  **L120 CN**: 继续与可调用符号 `isl_bool` 相关的逻辑。

### Lines 121-140

````c
{
	int i;
	struct isl_tarjan_graph *g = NULL;

	g = isl_tarjan_graph_alloc(ctx, len);
	if (!g)
		return NULL;
	for (i = len - 1; i >= 0; --i) {
		if (g->node[i].index >= 0)
			continue;
		if (isl_tarjan_components(g, i, follows, user) < 0)
			return isl_tarjan_graph_free(g);
	}

	return g;
}

/* Decompose the graph with "len" nodes and edges defined by "follows"
 * into the strongly connected component (SCC) that contains "node"
 * as well as all SCCs that are followed by this SCC.
````
- **L121 EN**: Opens a new lexical scope or compound statement.
  **L121 CN**: 打开一个新的词法作用域或复合语句块。
- **L122 EN**: Executes a standalone statement or declaration: `int i;`.
  **L122 CN**: 执行一条独立语句或声明：`int i;`。
- **L123 EN**: Declares struct `isl_tarjan_graph`.
  **L123 CN**: 声明 struct `isl_tarjan_graph`。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L125 EN**: Executes a call or declaration centered on `isl_tarjan_graph_alloc`.
  **L125 CN**: 执行以 `isl_tarjan_graph_alloc` 为核心的调用或声明。
- **L126 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L126 CN**: 开始 `if` 控制流语句并计算其条件。
- **L127 EN**: Returns from the current function with `NULL`.
  **L127 CN**: 以 `NULL` 从当前函数返回。
- **L128 EN**: Begins a `for` control-flow statement and evaluates its condition.
  **L128 CN**: 开始 `for` 控制流语句并计算其条件。
- **L129 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L129 CN**: 开始 `if` 控制流语句并计算其条件。
- **L130 EN**: Skips to the next loop iteration.
  **L130 CN**: 跳到下一次循环迭代。
- **L131 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L131 CN**: 开始 `if` 控制流语句并计算其条件。
- **L132 EN**: Returns from the current function with `isl_tarjan_graph_free(g)`.
  **L132 CN**: 以 `isl_tarjan_graph_free(g)` 从当前函数返回。
- **L133 EN**: Closes the current lexical scope or compound statement.
  **L133 CN**: 结束当前词法作用域或复合语句块。
- **L134 EN**: Blank line separating nearby declarations or logic blocks.
  **L134 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L135 EN**: Returns from the current function with `g`.
  **L135 CN**: 以 `g` 从当前函数返回。
- **L136 EN**: Closes the current lexical scope or compound statement.
  **L136 CN**: 结束当前词法作用域或复合语句块。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L138 EN**: Comment explains nearby logic, invariants, or intent: `Decompose the graph with "len" nodes and edges defined by "follows"`.
  **L138 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Decompose the graph with "len" nodes and edges defined by "follows"`。
- **L139 EN**: Comment explains nearby logic, invariants, or intent: `into the strongly connected component (SCC) that contains "node"`.
  **L139 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`into the strongly connected component (SCC) that contains "node"`。
- **L140 EN**: Comment explains nearby logic, invariants, or intent: `as well as all SCCs that are followed by this SCC.`.
  **L140 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`as well as all SCCs that are followed by this SCC.`。

### Lines 141-159

````c
 * follows(i, j, user) should return 1 if "i" follows "j" and 0 otherwise.
 * It should return -1 on error.
 *
 * The SCC containing "node" will appear as the last component
 * in g->order.
 */
struct isl_tarjan_graph *isl_tarjan_graph_component(isl_ctx *ctx, int len,
	int node, isl_bool (*follows)(int i, int j, void *user), void *user)
{
	struct isl_tarjan_graph *g;

	g = isl_tarjan_graph_alloc(ctx, len);
	if (!g)
		return NULL;
	if (isl_tarjan_components(g, node, follows, user) < 0)
		return isl_tarjan_graph_free(g);

	return g;
}
````
- **L141 EN**: Comment explains nearby logic, invariants, or intent: `follows(i, j, user) should return 1 if "i" follows "j" and 0 otherwise.`.
  **L141 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`follows(i, j, user) should return 1 if "i" follows "j" and 0 otherwise.`。
- **L142 EN**: Comment explains nearby logic, invariants, or intent: `It should return -1 on error.`.
  **L142 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`It should return -1 on error.`。
- **L143 EN**: Separator comment used for visual grouping.
  **L143 CN**: 用于视觉分组的分隔注释。
- **L144 EN**: Comment explains nearby logic, invariants, or intent: `The SCC containing "node" will appear as the last component`.
  **L144 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The SCC containing "node" will appear as the last component`。
- **L145 EN**: Comment explains nearby logic, invariants, or intent: `in g->order.`.
  **L145 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`in g->order.`。
- **L146 EN**: Separator comment used for visual grouping.
  **L146 CN**: 用于视觉分组的分隔注释。
- **L147 EN**: Declares struct `isl_tarjan_graph`.
  **L147 CN**: 声明 struct `isl_tarjan_graph`。
- **L148 EN**: Continues logic associated with callable symbol `isl_bool`.
  **L148 CN**: 继续与可调用符号 `isl_bool` 相关的逻辑。
- **L149 EN**: Opens a new lexical scope or compound statement.
  **L149 CN**: 打开一个新的词法作用域或复合语句块。
- **L150 EN**: Declares struct `isl_tarjan_graph`.
  **L150 CN**: 声明 struct `isl_tarjan_graph`。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L152 EN**: Executes a call or declaration centered on `isl_tarjan_graph_alloc`.
  **L152 CN**: 执行以 `isl_tarjan_graph_alloc` 为核心的调用或声明。
- **L153 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L153 CN**: 开始 `if` 控制流语句并计算其条件。
- **L154 EN**: Returns from the current function with `NULL`.
  **L154 CN**: 以 `NULL` 从当前函数返回。
- **L155 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L155 CN**: 开始 `if` 控制流语句并计算其条件。
- **L156 EN**: Returns from the current function with `isl_tarjan_graph_free(g)`.
  **L156 CN**: 以 `isl_tarjan_graph_free(g)` 从当前函数返回。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L158 EN**: Returns from the current function with `g`.
  **L158 CN**: 以 `g` 从当前函数返回。
- **L159 EN**: Closes the current lexical scope or compound statement.
  **L159 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Polyhedral integer-set operations / 多面体整数集合操作**
- **AST-based code generation / 基于 AST 的代码生成**
- **Graph SCC decomposition / 图的强连通分量分解**
- **Identifier attachment and lookup / 标识符附着与查找**

## Dependencies / 依赖关系

- `stdlib.h`: Provides standard C library facilities. / 提供标准 C 库功能。
- `isl/ctx.h`: Provides public isl interfaces imported by this file. / 提供该文件使用的公开 isl 接口。
- `isl_tarjan.h`: Provides local isl declarations paired with this implementation file. / 提供与该实现文件配套的本地 isl 声明。
