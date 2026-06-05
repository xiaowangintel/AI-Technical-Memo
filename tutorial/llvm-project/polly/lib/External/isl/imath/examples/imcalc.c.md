# imcalc.c — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/imath/examples/imcalc.c` | `polly/lib/External/isl/imath/examples/imcalc.c` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements or embeds upstream ISL support code used by Polly. | 实现或内嵌 Polly 使用的上游 ISL 支撑代码。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*
  Name:     imcalc.c
  Purpose:  Simple RPN calculator based on IMath library.
  Author:   M. J. Fromberger

  This is a very simplistic RPN calculator that will let you test the features
  of the IMath built-in functions.

  Copyright (C) 2002-2008 Michael J. Fromberger, All Rights Reserved.

  Permission is hereby granted, free of charge, to any person obtaining a copy
  of this software and associated documentation files (the "Software"), to deal
  in the Software without restriction, including without limitation the rights
  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
  copies of the Software, and to permit persons to whom the Software is
  furnished to do so, subject to the following conditions:

  The above copyright notice and this permission notice shall be included in
  all copies or substantial portions of the Software.

````
- **EN**: This block declares or defines routines around `Copyright`, `files`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `Copyright`, `files` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 21-41

````c
  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL THE
  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
  SOFTWARE.
 */

#include <assert.h>
#include <ctype.h>
#include <errno.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <strings.h> /* for strcasecmp */

#include <getopt.h>
#include <unistd.h>

````
- **EN**: This block imports system/standard headers needed by the surrounding code.
- **CN**: 该代码块 引入周边逻辑所需的 system/standard 头文件.

### Lines 42-66

````c
#include "imath.h"
#include "imrat.h"
#include "iprime.h"

/* A cstate_t represents a stack of operands; numeric operands are pushed on
   the stack, and commands cause them to be consumed in various ways.
 */
typedef struct {
  /* Operand stack    */
  mp_int *elts;
  mp_size alloc; /* number of slots available */
  mp_size used;  /* number of slots free      */

  /* Named variables  */
  mp_int *mem;    /* named memory slots        */
  char **names;   /* names of memory slots     */
  mp_size mslots; /* number of memory slots    */
  mp_size mused;  /* number of used memories   */

  /* I/O components   */
  FILE *ifp;  /* input file handle         */
  char *ibuf; /* input scratch buffer      */
  int buflen; /* size of scratch buffer    */
} cstate_t;

````
- **EN**: This block imports system/standard headers needed by the surrounding code; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 引入周边逻辑所需的 system/standard 头文件; 保留用于解释意图、用法或算法背景的注释.

### Lines 67-90

````c
static mp_result state_init(cstate_t *sp, mp_size n_elts);
static void state_clear(cstate_t *sp);
static void stack_flush(cstate_t *sp);
static mp_result stack_push(cstate_t *sp, mp_int elt);
static mp_result stack_pop(cstate_t *sp);
static mp_result mem_insert(cstate_t *sp, const char *name, mp_int value);
static mp_result mem_recall(cstate_t *sp, const char *name, mp_int value);
static mp_result mem_clear(cstate_t *sp);

typedef mp_result (*op_func)(cstate_t *);

static mp_result cf_abs(cstate_t *sp);
static mp_result cf_neg(cstate_t *sp);
static mp_result cf_add(cstate_t *sp);
static mp_result cf_sub(cstate_t *sp);
static mp_result cf_mul(cstate_t *sp);
static mp_result cf_divmod(cstate_t *sp);
static mp_result cf_div(cstate_t *sp);
static mp_result cf_mod(cstate_t *sp);
static mp_result cf_expt(cstate_t *sp);
static mp_result cf_exptmod(cstate_t *sp);
static mp_result cf_square(cstate_t *sp);
static mp_result cf_invmod(cstate_t *sp);
static mp_result cf_gcd(cstate_t *sp);
````
- **EN**: This block declares or defines routines around `state_init`, `state_clear`, `stack_flush`, `stack_push` (+18 more).
- **CN**: 该代码块 声明或定义与 `state_init`, `state_clear`, `stack_flush`, `stack_push` (+18 more) 相关的例程.

### Lines 91-114

````c
static mp_result cf_xgcd(cstate_t *sp);
static mp_result cf_sqrt(cstate_t *sp);
static mp_result cf_root(cstate_t *sp);
static mp_result cf_cmplt(cstate_t *sp);
static mp_result cf_cmpgt(cstate_t *sp);
static mp_result cf_cmple(cstate_t *sp);
static mp_result cf_cmpge(cstate_t *sp);
static mp_result cf_cmpeq(cstate_t *sp);
static mp_result cf_cmpne(cstate_t *sp);
static mp_result cf_inc(cstate_t *sp);
static mp_result cf_dec(cstate_t *sp);
static mp_result cf_fact(cstate_t *sp);
static mp_result cf_pprint(cstate_t *sp);
static mp_result cf_print(cstate_t *sp);
static mp_result cf_pstack(cstate_t *sp);
static mp_result cf_clstk(cstate_t *sp);
static mp_result cf_pop(cstate_t *sp);
static mp_result cf_dup(cstate_t *sp);
static mp_result cf_copy(cstate_t *sp);
static mp_result cf_swap(cstate_t *sp);
static mp_result cf_rot(cstate_t *sp);
static mp_result cf_pick(cstate_t *sp);
static mp_result cf_setr(cstate_t *sp);
static mp_result cf_setbin(cstate_t *sp);
````
- **EN**: This block declares or defines routines around `cf_xgcd`, `cf_sqrt`, `cf_root`, `cf_cmplt` (+20 more).
- **CN**: 该代码块 声明或定义与 `cf_xgcd`, `cf_sqrt`, `cf_root`, `cf_cmplt` (+20 more) 相关的例程.

### Lines 115-138

````c
static mp_result cf_help(cstate_t *sp);
static mp_result cf_store(cstate_t *sp);
static mp_result cf_recall(cstate_t *sp);
static mp_result cf_cmem(cstate_t *sp);
static mp_result cf_pmem(cstate_t *sp);
static mp_result cf_qrecall(cstate_t *sp);

typedef struct {
  char *name;      /* The name of the operator.           */
  int stack_size;  /* Number of stack arguments required. */
  op_func handler; /* Function implementing operation.    */
  char *descript;  /* Human-readable description.         */
} calcop_t;

static calcop_t g_ops[] = {
    {"abs", 1, cf_abs, "x -- |x|"},
    {"neg", 1, cf_neg, "x -- (-x)"},
    {"+", 2, cf_add, "x y -- (x+y)"},
    {"add", 2, cf_add, "x y -- (x+y)"},
    {"-", 2, cf_sub, "x y -- (x-y)"},
    {"sub", 2, cf_sub, "x y -- (x-y)"},
    {"*", 2, cf_mul, "x y -- (x*y)"},
    {"mul", 2, cf_mul, "x y -- (x*y)"},
    {"/", 2, cf_divmod, "x y -- q r ; x = yq + r, 0 <= r < y"},
````
- **EN**: This block declares or defines routines around `cf_help`, `cf_store`, `cf_recall`, `cf_cmem` (+2 more).
- **CN**: 该代码块 声明或定义与 `cf_help`, `cf_store`, `cf_recall`, `cf_cmem` (+2 more) 相关的例程.

### Lines 139-163

````c
    {"//", 2, cf_div, "x y -- (x div y)"},
    {"div", 2, cf_div, "x y -- (x div y)"},
    {"%", 2, cf_mod, "x y -- (x mod y)"},
    {"mod", 2, cf_mod, "x y -- (x mod y)"},
    {"^", 2, cf_expt, "x y -- (x^y)"},
    {"expt", 2, cf_expt, "x y -- (x^y)"},
    {"^^", 3, cf_exptmod, "x y m -- (x^y mod m)"},
    {"emod", 3, cf_exptmod, "x y m -- (x^y mod m)"},
    {"sqr", 1, cf_square, "x -- (x*x)"},
    {"inv", 2, cf_invmod, "x m -- (1/x mod m)"},
    {"gcd", 2, cf_gcd, "x y -- gcd(x, y)"},
    {"xgcd", 2, cf_xgcd, "x y -- g u v ; g = ux + vy"},
    {"sqrt", 1, cf_sqrt, "x -- floor(sqrt(x))"},
    {"root", 2, cf_root, "x y -- floor(x^{1/y}) ; y > 0"},
    {"<", 2, cf_cmplt, "x y -- (x<y)"},
    {">", 2, cf_cmpgt, "x y -- (x>y)"},
    {"<=", 2, cf_cmple, "x y -- (x<=y)"},
    {">=", 2, cf_cmpge, "x y -- (x>=y)"},
    {"=", 2, cf_cmpeq, "x y -- (x=y)"},
    {"<>", 2, cf_cmpne, "x y -- (x<>y)"},
    {"inc", 1, cf_inc, "x -- (x+1)"},
    {"dec", 1, cf_dec, "x -- (x-1)"},
    {"!", 1, cf_fact, "x -- x!"},
    {"fact", 1, cf_fact, "x -- x!"},

````
- **EN**: This block declares or defines routines around `gcd`, `floor`.
- **CN**: 该代码块 声明或定义与 `gcd`, `floor` 相关的例程.

### Lines 164-184

````c
    {".", 1, cf_pprint, "x -- ; print x in current output mode"},
    {";", 1, cf_print, "x -- x ; print x in current output mode"},
    {"?", 0, cf_pstack, "-- ; print stack"},
    {"cls", 0, cf_clstk, "... -- ; clear stack"},
    {"$", 1, cf_pop, "x --"},
    {"drop", 1, cf_pop, "x --"},
    {"dup", 1, cf_dup, "x -- x x"},
    {"copy", 2, cf_copy, "vn ... v1 v0 n -- vn ... v0 vn ... v0"},
    {"swap", 2, cf_swap, "x y -- y x"},
    {"rot", 3, cf_rot, "a b c -- b c a"},
    {"pick", 2, cf_pick, "... v2 v1 v0 n -- ... v2 v1 v0 vn"},

    {">>", 1, cf_store, "x -- ; save in named variable"},
    {"<<", 0, cf_recall, "-- x ; recall from named variable"},
    {"clm", 0, cf_cmem, "-- ; clear memory"},
    {"??", 0, cf_pmem, "-- ; print memory"},

    {"out", 1, cf_setr, "r -- ; set output radix to r"},
    {"bin", 0, cf_setbin, "-- ; set output format to binary"},
    {"help", 0, cf_help, "-- ; print help message"},

````
- **EN**: This block contains straightforward declarations or statements that continue the file's implementation.
- **CN**: 该代码块 包含延续本文件实现的直接声明或语句.

### Lines 185-204

````c
    /* This is the end-marker, but it is also used to catch implicit
       variable lookups from memory.
     */
    {NULL, 0, cf_qrecall, "-- x ; recall from named variable"},
};

#define BUFFER_SIZE 16384 /* max. length of input values, in chars */

/* Token types from the primitive lexical analyzer */
typedef enum { t_eof, t_symbol, t_number, t_error } token_t;

static token_t next_token(FILE *ifp, char *buf, int size);
static mp_result read_number(char *buf, mp_int *out);
static int find_command(cstate_t *ops);
static void print_value(mp_int v);
static mp_result run_file(FILE *ifp, cstate_t *op_state);

/* Error code used internally to signal input problems. */
static mp_result MP_INPUT;

````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; defines macros like `BUFFER_SIZE`; declares or defines routines around `next_token`, `read_number`, `find_command`, `print_value` (+1 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `BUFFER_SIZE`; 声明或定义与 `next_token`, `read_number`, `find_command`, `print_value` (+1 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 205-230

````c
static int g_output_radix = 10; /* output radix */
static FILE *g_output_file = NULL;

int main(int argc, char *argv[]) {
  extern char *optarg;
  extern int optind;

  int opt, errs = 0;
  FILE *ifp;

  cstate_t op_state;
  mp_result res;

  MP_INPUT = MP_MINERR - 1;

  g_output_file = stdout;
  while ((opt = getopt(argc, argv, "ho:")) != EOF) {
    switch (opt) {
      case 'h':
        fprintf(
            stderr,
            "Usage: imcalc [-h] [-o <output>] input*\n\n"
            "Options:\n"
            "  -h          : display this help message.\n"
            "  -o <output> : send output to file.\n\n"

````
- **EN**: This block declares or defines routines around `main`, `fprintf`; contains control flow with 1 loop construct(s), 1 switch dispatch(es).
- **CN**: 该代码块 声明或定义与 `main`, `fprintf` 相关的例程; 包含控制流结构：1 处循环、1 处分支派发.

### Lines 231-252

````c
            "If no input files are given, the standard input is read.  The\n"
            "special file name \"-\" is interpreted to mean the standard "
            "input.\n"
            "Output goes to standard output unless \"-o\" is used.\n\n");
        return 0;

      case 'o':
        if ((g_output_file = fopen(optarg, "wt")) == NULL) {
          fprintf(stderr, "Unable to open \"%s\" for writing: %s\n", optarg,
                  strerror(errno));
          return 1;
        }
        break;

      default:
        fprintf(stderr,
                "Usage: imcalc [-h] [-o <output>] input*\n"
                "       [use \"imcalc -h\" to get help]\n\n");
        return 1;
    }
  }

````
- **EN**: This block declares or defines routines around `fprintf`, `strerror`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `fprintf`, `strerror` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 253-272

````c
  if ((res = state_init(&op_state, 1)) != MP_OK) {
    fprintf(stderr, "Error: state_init: %s\n", mp_error_string(res));
    return 1;
  }

  if (optind < argc) {
    int ix;

    for (ix = optind; ix < argc; ++ix) {
      if (strcmp(argv[ix], "-") == 0)
        ifp = stdin;
      else if ((ifp = fopen(argv[optind], "rt")) == NULL) {
        fprintf(stderr, "Unable to open \"%s\" for reading: %s\n", argv[optind],
                strerror(errno));
        return 1;
      }

      if (run_file(ifp, &op_state) != MP_OK) ++errs;
    }

````
- **EN**: This block declares or defines routines around `fprintf`, `strerror`; contains control flow with 1 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `fprintf`, `strerror` 相关的例程; 包含控制流结构：1 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 273-295

````c
    state_clear(&op_state);
    return errs > 0;
  } else {
    int rv = 1 - (run_file(stdin, &op_state) == MP_OK);
    state_clear(&op_state);
    return rv;
  }
}

static token_t next_token(FILE *ifp, char *buf, int size) {
  int ch, pos = 0;
  token_t res;

  assert(buf != NULL && size > 0);

  while ((ch = fgetc(ifp)) != EOF && isspace(ch)) /* empty */
    ;

  if (ch == EOF) {
    buf[0] = '\0';
    return t_eof;
  }

````
- **EN**: This block declares or defines routines around `state_clear`, `run_file`, `next_token`; contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states.
- **CN**: 该代码块 声明或定义与 `state_clear`, `run_file`, `next_token` 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查.

### Lines 296-320

````c
  if (ch == '-') {
    int next = fgetc(ifp);
    if (next == EOF || !isdigit(next))
      res = t_symbol;
    else
      res = t_number;
    ungetc(next, ifp);
  } else if (isdigit(ch) || ch == '#')
    res = t_number;
  else
    res = t_symbol;

  buf[pos++] = ch;
  while ((ch = fgetc(ifp)) != EOF) {
    if ((res == t_number && ispunct(ch) && ch != '-') ||
        (res == t_symbol && isdigit(ch)) || isspace(ch)) {
      ungetc(ch, ifp);
      break;
    } else if (pos + 1 >= size) {
      res = t_error;
      break;
    }
    buf[pos++] = ch;
  }

````
- **EN**: This block declares or defines routines around `fgetc`, `ungetc`, `isdigit`; contains control flow with 1 loop construct(s), 5 conditional check(s).
- **CN**: 该代码块 声明或定义与 `fgetc`, `ungetc`, `isdigit` 相关的例程; 包含控制流结构：1 处循环、5 处条件判断.

### Lines 321-344

````c
  buf[pos] = '\0';
  return res;
}

static mp_result read_number(char *buf, mp_int *out) {
  int radix = 10, pos = 0;
  mp_result res;
  mp_int value;

  assert(buf != NULL && out != NULL);

  if (buf[pos] == '#') {
    switch (buf[1]) {
      case 'b':
      case 'B':
        radix = 2;
        break;
      case 'd':
      case 'D':
        radix = 10;
        break;
      case 'o':
      case 'O':
        radix = 8;
````
- **EN**: This block declares or defines routines around `read_number`; contains control flow with 1 conditional check(s), 1 switch dispatch(es); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states.
- **CN**: 该代码块 声明或定义与 `read_number` 相关的例程; 包含控制流结构：1 处条件判断、1 处分支派发; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查.

### Lines 345-367

````c
        break;
      case 'x':
      case 'X':
        radix = 16;
        break;
      default:
        return MP_BADARG;
    }

    pos += 2;
  }

  if ((value = mp_int_alloc()) == NULL) {
    *out = NULL;
    return MP_MEMORY;
  }

  if ((res = mp_int_read_string(value, radix, buf + pos)) != MP_OK) {
    mp_int_free(value);
    *out = NULL;
    return res;
  }

````
- **EN**: This block declares or defines routines around `mp_int_free`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_int_free` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 368-389

````c
  *out = value;
  return res;
}

static int find_command(cstate_t *op) {
  int ix, jx;
  char *buf = op->ibuf;

  /* First, try to find the command by name */
  for (ix = 0; g_ops[ix].name != NULL; ++ix) {
    if (strcasecmp(buf, g_ops[ix].name) == 0) return ix;
  }

  /* If we don't find the command, try a variable lookup */
  for (jx = 0; (mp_size)jx < op->mused; ++jx) {
    if (strcmp(buf, op->names[jx]) == 0) return ix; /* sentinel */
  }

  /* If variable lookup fails, report command not found */
  return -1;
}

````
- **EN**: This block declares or defines routines around `find_command`; contains control flow with 2 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `find_command` 相关的例程; 包含控制流结构：2 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 390-409

````c
static void print_value(mp_int v) {
  if (g_output_radix == 0) {
    mp_result len = mp_int_binary_len(v);
    unsigned char *buf = malloc(len);
    int ix;

    if (buf != NULL) {
      mp_int_to_binary(v, buf, len);
      for (ix = 0; ix < len - 1; ++ix) {
        fprintf(g_output_file, "%02x.", buf[ix]);
      }
      fprintf(g_output_file, "%02x\n", buf[ix]);
      free(buf);
    } else {
      fprintf(g_output_file, "<insufficient memory to print>\n");
    }
  } else {
    mp_result len = mp_int_string_len(v, g_output_radix);
    char *buf = malloc(len);

````
- **EN**: This block declares or defines routines around `print_value`, `mp_int_binary_len`, `malloc`, `mp_int_to_binary` (+3 more); contains control flow with 1 loop construct(s), 2 conditional check(s).
- **CN**: 该代码块 声明或定义与 `print_value`, `mp_int_binary_len`, `malloc`, `mp_int_to_binary` (+3 more) 相关的例程; 包含控制流结构：1 处循环、2 处条件判断.

### Lines 410-429

````c
    if (buf != NULL) {
      mp_int_to_string(v, g_output_radix, buf, len);
      fputs(buf, g_output_file);
      fputc('\n', g_output_file);
      free(buf);
    } else {
      fprintf(g_output_file, "<insufficient memory to print>\n");
    }
  }
}

static mp_result run_file(FILE *ifp, cstate_t *op_state) {
  mp_result res = MP_OK;
  token_t next;

  op_state->ifp = ifp;
  while ((next = next_token(ifp, op_state->ibuf, op_state->buflen)) != t_eof) {
    mp_int value = NULL;
    int cpos;

````
- **EN**: This block declares or defines routines around `mp_int_to_string`, `fputs`, `fputc`, `free` (+2 more); contains control flow with 1 loop construct(s), 1 conditional check(s).
- **CN**: 该代码块 声明或定义与 `mp_int_to_string`, `fputs`, `fputc`, `free` (+2 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断.

### Lines 430-453

````c
    switch (next) {
      case t_number:
        if ((res = read_number(op_state->ibuf, &value)) != MP_OK)
          fprintf(stderr, "error: invalid number syntax: %s\n", op_state->ibuf);
        else if ((res = stack_push(op_state, value)) != MP_OK)
          goto EXIT;
        break;
      case t_symbol:
        if ((cpos = find_command(op_state)) < 0) {
          fprintf(stderr, "error: command not understood: %s\n",
                  op_state->ibuf);
        } else if (op_state->used < (mp_size)g_ops[cpos].stack_size) {
          fprintf(stderr, "error: not enough arguments (have %d, want %d)\n",
                  op_state->used, g_ops[cpos].stack_size);
        } else if ((res = (g_ops[cpos].handler)(op_state)) != MP_OK) {
          if (res == MP_INPUT) {
            fprintf(stderr, "error: incorrect input format\n");
          } else {
            fprintf(stderr, "error: %s\n", mp_error_string(res));
          }
        }
        break;
      default:
        fprintf(stderr, "error: invalid input token: %s\n", op_state->ibuf);
````
- **EN**: This block declares or defines routines around `fprintf`; contains control flow with 6 conditional check(s), 1 switch dispatch(es).
- **CN**: 该代码块 声明或定义与 `fprintf` 相关的例程; 包含控制流结构：6 处条件判断、1 处分支派发.

### Lines 454-477

````c
        res = MP_BADARG;
        goto EXIT;
    }
  }

EXIT:
  return res;
}

static mp_result state_init(cstate_t *sp, mp_size n_elts) {
  int ix;

  assert(sp != NULL && n_elts > 0);

  if ((sp->elts = malloc(n_elts * sizeof(*(sp->elts)))) == NULL)
    return MP_MEMORY;
  if ((sp->mem = malloc(n_elts * sizeof(*(sp->mem)))) == NULL) {
    free(sp->elts);
    return MP_MEMORY;
  }
  if ((sp->names = malloc(n_elts * sizeof(*(sp->names)))) == NULL) {
    free(sp->mem);
    free(sp->elts);
    return MP_MEMORY;
````
- **EN**: This block declares or defines routines around `state_init`, `free`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states.
- **CN**: 该代码块 声明或定义与 `state_init`, `free` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查.

### Lines 478-497

````c
  }
  if ((sp->ibuf = malloc(BUFFER_SIZE * sizeof(char))) == NULL) {
    free(sp->names);
    free(sp->mem);
    free(sp->elts);
    return MP_MEMORY;
  }

  for (ix = 0; (mp_size)ix < n_elts; ++ix) {
    sp->elts[ix] = NULL;
    sp->mem[ix] = NULL;
    sp->names[ix] = NULL;
  }

  sp->alloc = n_elts;
  sp->used = 0;
  sp->mslots = n_elts;
  sp->mused = 0;
  sp->buflen = BUFFER_SIZE;

````
- **EN**: This block declares or defines routines around `free`; contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `free` 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 498-519

````c
  return MP_OK;
}

static void state_clear(cstate_t *sp) {
  assert(sp != NULL);

  if (sp->elts != NULL) {
    int ix;

    for (ix = 0; (mp_size)ix < sp->used; ++ix) {
      mp_int_clear(sp->elts[ix]);
      sp->elts[ix] = NULL;
    }

    free(sp->elts);
    sp->elts = NULL;
    sp->alloc = 0;
    sp->used = 0;
  }
  if (sp->mem != NULL) {
    int ix;

````
- **EN**: This block declares or defines routines around `state_clear`, `mp_int_clear`, `free`; contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states.
- **CN**: 该代码块 声明或定义与 `state_clear`, `mp_int_clear`, `free` 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查.

### Lines 520-544

````c
    for (ix = 0; (mp_size)ix < sp->mused; ++ix) {
      mp_int_free(sp->mem[ix]);
      sp->mem[ix] = NULL;
      free(sp->names[ix]);
      sp->names[ix] = NULL;
    }

    free(sp->mem);
    sp->mem = NULL;
    free(sp->names);
    sp->names = NULL;

    sp->mslots = 0;
    sp->mused = 0;
  }
  if (sp->ibuf != NULL) {
    free(sp->ibuf);
    sp->buflen = 0;
  }
  if (sp->ifp != NULL) {
    fclose(sp->ifp);
    sp->ifp = NULL;
  }
}

````
- **EN**: This block declares or defines routines around `mp_int_free`, `free`, `fclose`; contains control flow with 1 loop construct(s), 2 conditional check(s).
- **CN**: 该代码块 声明或定义与 `mp_int_free`, `free`, `fclose` 相关的例程; 包含控制流结构：1 处循环、2 处条件判断.

### Lines 545-565

````c
static void stack_flush(cstate_t *sp) {
  int ix;

  assert(sp != NULL && sp->elts != NULL);

  for (ix = 0; (mp_size)ix < sp->used; ++ix) {
    mp_int_clear(sp->elts[ix]);
    sp->elts[ix] = NULL;
  }

  sp->used = 0;
}

static mp_result stack_push(cstate_t *sp, mp_int elt) {
  if (sp->used >= sp->alloc) {
    mp_size nsize = 2 * sp->alloc;
    mp_int *tmp;
    int ix;

    if ((tmp = malloc(nsize * sizeof(*(sp->elts)))) == NULL) return MP_MEMORY;

````
- **EN**: This block declares or defines routines around `stack_flush`, `mp_int_clear`, `stack_push`; contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states.
- **CN**: 该代码块 声明或定义与 `stack_flush`, `mp_int_clear`, `stack_push` 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查.

### Lines 566-587

````c
    for (ix = 0; (mp_size)ix < sp->used; ++ix) {
      tmp[ix] = sp->elts[ix];
    }

    free(sp->elts);
    sp->elts = tmp;
    sp->alloc = nsize;
  }

  sp->elts[sp->used++] = elt;
  return MP_OK;
}

static mp_result stack_pop(cstate_t *sp) {
  assert(sp != NULL && sp->elts != NULL);

  if (sp->used == 0) return MP_UNDEF;

  sp->used -= 1;
  mp_int_clear(sp->elts[sp->used]);
  sp->elts[sp->used] = NULL;

````
- **EN**: This block declares or defines routines around `free`, `stack_pop`, `mp_int_clear`; contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers; adds defensive checks for invariants or impossible states.
- **CN**: 该代码块 声明或定义与 `free`, `stack_pop`, `mp_int_clear` 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 加入用于保护不变量或异常状态的断言/检查.

### Lines 588-610

````c
  return MP_OK;
}

static mp_result mem_insert(cstate_t *sp, const char *name, mp_int value) {
  int ix;

  for (ix = 0; (mp_size)ix < sp->mused; ++ix) {
    if (strcmp(name, sp->names[ix]) == 0) break;
  }

  /* Two cases:
     ix < sp->mused   ==> replacing existing entry.
     otherwise        ==> adding new entry, may need to grow dictionary.
   */
  if ((mp_size)ix < sp->mused) {
    mp_int_free(sp->mem[ix]); /* fall through to the end */
  } else {
    if (sp->mused >= sp->mslots) {
      mp_size nsize = 2 * sp->mslots;
      mp_int *tz;
      char **tc;
      int jx;

````
- **EN**: This block declares or defines routines around `mem_insert`, `mp_int_free`; contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `mem_insert`, `mp_int_free` 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 611-634

````c
      if ((tz = malloc(nsize * sizeof(*(sp->mem)))) == NULL) return MP_MEMORY;
      if ((tc = malloc(nsize * sizeof(*(sp->names)))) == NULL) {
        free(tz);
        return MP_MEMORY;
      }

      for (jx = 0; (mp_size)jx < sp->mused; ++jx) {
        tz[jx] = sp->mem[jx];
        tc[jx] = sp->names[jx];
      }

      free(sp->mem);
      sp->mem = tz;
      free(sp->names);
      sp->names = tc;

      sp->mslots = nsize;
    }

    sp->mused += 1;
    sp->names[ix] = malloc(1 + strlen(name));
    strcpy(sp->names[ix], name);
  }

````
- **EN**: This block declares or defines routines around `free`, `malloc`, `strcpy`; contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `free`, `malloc`, `strcpy` 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 635-659

````c
  sp->mem[ix] = mp_int_alloc();
  return mp_int_copy(value, sp->mem[ix]);
}

static mp_result mem_recall(cstate_t *sp, const char *name, mp_int value) {
  int ix;

  for (ix = 0; (mp_size)ix < sp->mused; ++ix) {
    if (strcmp(name, sp->names[ix]) == 0) {
      return mp_int_copy(sp->mem[ix], value);
    }
  }

  return MP_UNDEF; /* not found */
}

static mp_result mem_clear(cstate_t *sp) {
  int ix;

  for (ix = 0; (mp_size)ix < sp->mused; ++ix) {
    mp_int_free(sp->mem[ix]);
    free(sp->names[ix]);
  }
  sp->mused = 0;

````
- **EN**: This block declares or defines routines around `mp_int_alloc`, `mem_recall`, `mem_clear`, `mp_int_free` (+1 more); contains control flow with 2 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_int_alloc`, `mem_recall`, `mem_clear`, `mp_int_free` (+1 more) 相关的例程; 包含控制流结构：2 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 660-679

````c
  return MP_OK;
}

static mp_result cf_abs(cstate_t *sp) {
  mp_int a = sp->elts[sp->used - 1];

  return mp_int_abs(a, a);
}

static mp_result cf_neg(cstate_t *sp) {
  mp_int a = sp->elts[sp->used - 1];

  return mp_int_neg(a, a);
}

static mp_result cf_add(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res = mp_int_add(a, b, a);

````
- **EN**: This block declares or defines routines around `cf_abs`, `cf_neg`, `cf_add`, `mp_int_add`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_abs`, `cf_neg`, `cf_add`, `mp_int_add` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 680-699

````c
  if (res == MP_OK) stack_pop(sp);

  return res;
}

static mp_result cf_sub(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res = mp_int_sub(a, b, a);

  if (res == MP_OK) stack_pop(sp);

  return res;
}

static mp_result cf_mul(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res = mp_int_mul(a, b, a);

````
- **EN**: This block declares or defines routines around `cf_sub`, `mp_int_sub`, `cf_mul`, `mp_int_mul`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_sub`, `mp_int_sub`, `cf_mul`, `mp_int_mul` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 700-721

````c
  if (res == MP_OK) stack_pop(sp);

  return res;
}

static mp_result cf_divmod(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];

  return mp_int_div(a, b, a, b);
}

static mp_result cf_div(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res = mp_int_div(a, b, a, NULL);

  if (res == MP_OK) stack_pop(sp);

  return res;
}

````
- **EN**: This block declares or defines routines around `cf_divmod`, `cf_div`, `mp_int_div`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_divmod`, `cf_div`, `mp_int_div` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 722-743

````c
static mp_result cf_mod(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res = mp_int_mod(a, b, a);

  if (res == MP_OK) stack_pop(sp);

  return res;
}

static mp_result cf_expt(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res;
  mp_small bval;

  if ((res = mp_int_to_int(b, &bval)) != MP_OK) return res;

  stack_pop(sp);
  return mp_int_expt(a, bval, a);
}

````
- **EN**: This block declares or defines routines around `cf_mod`, `mp_int_mod`, `cf_expt`, `stack_pop`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_mod`, `mp_int_mod`, `cf_expt`, `stack_pop` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 744-763

````c
static mp_result cf_exptmod(cstate_t *sp) {
  mp_int m = sp->elts[sp->used - 1];
  mp_int b = sp->elts[sp->used - 2];
  mp_int a = sp->elts[sp->used - 3];
  mp_result res = mp_int_exptmod(a, b, m, a);

  if (res == MP_OK) {
    stack_pop(sp);
    stack_pop(sp);
  }

  return res;
}

static mp_result cf_square(cstate_t *sp) {
  mp_int a = sp->elts[sp->used - 1];

  return mp_int_sqr(a, a);
}

````
- **EN**: This block declares or defines routines around `cf_exptmod`, `mp_int_exptmod`, `stack_pop`, `cf_square`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_exptmod`, `mp_int_exptmod`, `stack_pop`, `cf_square` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 764-783

````c
static mp_result cf_invmod(cstate_t *sp) {
  mp_int m = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res = mp_int_invmod(a, m, a);

  stack_pop(sp);

  return res;
}

static mp_result cf_gcd(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res = mp_int_gcd(a, b, a);

  if (res == MP_OK) stack_pop(sp);

  return res;
}

````
- **EN**: This block declares or defines routines around `cf_invmod`, `mp_int_invmod`, `stack_pop`, `cf_gcd` (+1 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_invmod`, `mp_int_invmod`, `stack_pop`, `cf_gcd` (+1 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 784-803

````c
static mp_result cf_xgcd(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_int t;
  mp_result res;

  if ((t = mp_int_alloc()) == NULL) return MP_MEMORY;
  if ((res = mp_int_egcd(a, b, a, b, t)) != MP_OK) {
    mp_int_free(t);
    return res;
  }

  if ((res = stack_push(sp, t)) != MP_OK) mp_int_free(t);

  return res;
}

static mp_result cf_sqrt(cstate_t *sp) {
  mp_int a = sp->elts[sp->used - 1];

````
- **EN**: This block declares or defines routines around `cf_xgcd`, `mp_int_free`, `cf_sqrt`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_xgcd`, `mp_int_free`, `cf_sqrt` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 804-823

````c
  return mp_int_sqrt(a, a);
}

static mp_result cf_root(cstate_t *sp) {
  mp_int a = sp->elts[sp->used - 2];
  mp_int bp = sp->elts[sp->used - 1];
  mp_small b;
  mp_result res;

  if ((res = mp_int_to_int(bp, &b)) != MP_OK) return res;

  stack_pop(sp);
  return mp_int_root(a, b, a);
}

static mp_result cf_cmplt(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res;

````
- **EN**: This block declares or defines routines around `cf_root`, `stack_pop`, `cf_cmplt`; contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_root`, `stack_pop`, `cf_cmplt` 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 824-843

````c
  res = mp_int_set_value(a, (mp_int_compare(a, b) < 0));
  stack_pop(sp);
  return res;
}

static mp_result cf_cmpgt(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res;

  res = mp_int_set_value(a, (mp_int_compare(a, b) > 0));
  stack_pop(sp);
  return res;
}

static mp_result cf_cmple(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res;

````
- **EN**: This block declares or defines routines around `mp_int_set_value`, `stack_pop`, `cf_cmpgt`, `cf_cmple`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_int_set_value`, `stack_pop`, `cf_cmpgt`, `cf_cmple` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 844-863

````c
  res = mp_int_set_value(a, (mp_int_compare(a, b) <= 0));
  stack_pop(sp);
  return res;
}

static mp_result cf_cmpge(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res;

  res = mp_int_set_value(a, (mp_int_compare(a, b) >= 0));
  stack_pop(sp);
  return res;
}

static mp_result cf_cmpeq(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res;

````
- **EN**: This block declares or defines routines around `mp_int_set_value`, `stack_pop`, `cf_cmpge`, `cf_cmpeq`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_int_set_value`, `stack_pop`, `cf_cmpge`, `cf_cmpeq` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 864-884

````c
  res = mp_int_set_value(a, (mp_int_compare(a, b) == 0));
  stack_pop(sp);
  return res;
}

static mp_result cf_cmpne(cstate_t *sp) {
  mp_int b = sp->elts[sp->used - 1];
  mp_int a = sp->elts[sp->used - 2];
  mp_result res;

  res = mp_int_set_value(a, (mp_int_compare(a, b) != 0));
  stack_pop(sp);
  return res;
}

static mp_result cf_inc(cstate_t *sp) {
  mp_int a = sp->elts[sp->used - 1];

  return mp_int_add_value(a, 1, a);
}

````
- **EN**: This block declares or defines routines around `mp_int_set_value`, `stack_pop`, `cf_cmpne`, `cf_inc`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_int_set_value`, `stack_pop`, `cf_cmpne`, `cf_inc` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 885-904

````c
static mp_result cf_dec(cstate_t *sp) {
  mp_int a = sp->elts[sp->used - 1];

  return mp_int_sub_value(a, 1, a);
}

static mp_result cf_fact(cstate_t *sp) {
  mpz_t tmp;
  mp_int x = sp->elts[sp->used - 1];
  mp_result res = MP_OK;

  if (mp_int_compare_zero(x) < 0) return MP_UNDEF;

  (void)mp_int_init_value(&tmp, 1);

  while (mp_int_compare_value(x, 1) > 0) {
    if ((res = mp_int_mul(&tmp, x, &tmp)) != MP_OK) goto CLEANUP;
    if ((res = mp_int_sub_value(x, 1, x)) != MP_OK) goto CLEANUP;
  }

````
- **EN**: This block declares or defines routines around `cf_dec`, `cf_fact`, `mp_int_init_value`; contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_dec`, `cf_fact`, `mp_int_init_value` 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 905-925

````c
  res = mp_int_copy(&tmp, x);

CLEANUP:
  mp_int_clear(&tmp);
  return res;
}

static mp_result cf_pprint(cstate_t *sp) {
  print_value(sp->elts[sp->used - 1]);
  stack_pop(sp);
  return MP_OK;
}

static mp_result cf_print(cstate_t *sp) {
  print_value(sp->elts[sp->used - 1]);
  return MP_OK;
}

static mp_result cf_pstack(cstate_t *sp) {
  int ix;

````
- **EN**: This block declares or defines routines around `mp_int_copy`, `mp_int_clear`, `cf_pprint`, `print_value` (+3 more); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_int_copy`, `mp_int_clear`, `cf_pprint`, `print_value` (+3 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 926-945

````c
  if (sp->used == 0) {
    fprintf(g_output_file, "<stack empty>\n");
  } else {
    for (ix = 0; (mp_size)ix < sp->used; ++ix) {
      fprintf(g_output_file, "%2d: ", ix);
      print_value(sp->elts[sp->used - 1 - ix]);
    }
  }

  return MP_OK;
}

static mp_result cf_clstk(cstate_t *sp) {
  stack_flush(sp);

  return MP_OK;
}

static mp_result cf_pop(cstate_t *sp) { return stack_pop(sp); }

````
- **EN**: This block declares or defines routines around `fprintf`, `print_value`, `cf_clstk`, `stack_flush` (+1 more); contains control flow with 1 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `fprintf`, `print_value`, `cf_clstk`, `stack_flush` (+1 more) 相关的例程; 包含控制流结构：1 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 946-967

````c
static mp_result cf_dup(cstate_t *sp) {
  mp_int cp = mp_int_alloc();
  mp_result res;

  if (cp == NULL) return MP_MEMORY;

  if ((res = mp_int_copy(sp->elts[sp->used - 1], cp)) != MP_OK) {
    mp_int_free(cp);
    return res;
  }

  if ((res = stack_push(sp, cp)) != MP_OK) mp_int_free(cp);

  return res;
}

static mp_result cf_copy(cstate_t *sp) {
  mp_int n = sp->elts[sp->used - 1];
  mp_result res;
  mp_small ncopy;
  int ix;

````
- **EN**: This block declares or defines routines around `cf_dup`, `mp_int_alloc`, `mp_int_free`, `cf_copy`; contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_dup`, `mp_int_alloc`, `mp_int_free`, `cf_copy` 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 968-989

````c
  if ((res = mp_int_to_int(n, &ncopy)) != MP_OK) return res;

  if (ncopy < 1 || ncopy >= sp->used) return MP_RANGE;

  stack_pop(sp);

  for (ix = 0; ix < ncopy; ++ix) {
    mp_int old = sp->elts[sp->used - ncopy];
    mp_int new = mp_int_alloc();

    if (new == NULL) return MP_MEMORY;

    if ((res = mp_int_copy(old, new)) != MP_OK) {
      mp_int_free(new);
      return res;
    }
    if ((res = stack_push(sp, new)) != MP_OK) return res;
  }

  return MP_OK;
}

````
- **EN**: This block declares or defines routines around `stack_pop`, `mp_int_alloc`, `mp_int_free`; contains control flow with 1 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `stack_pop`, `mp_int_alloc`, `mp_int_free` 相关的例程; 包含控制流结构：1 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 990-1013

````c
static mp_result cf_swap(cstate_t *sp) {
  mp_int t = sp->elts[sp->used - 1];

  sp->elts[sp->used - 1] = sp->elts[sp->used - 2];
  sp->elts[sp->used - 2] = t;

  return MP_OK;
}

static mp_result cf_rot(cstate_t *sp) {
  mp_int t = sp->elts[sp->used - 3];

  sp->elts[sp->used - 3] = sp->elts[sp->used - 2];
  sp->elts[sp->used - 2] = sp->elts[sp->used - 1];
  sp->elts[sp->used - 1] = t;

  return MP_OK;
}

static mp_result cf_pick(cstate_t *sp) {
  mp_int n = sp->elts[sp->used - 1];
  mp_result res;
  mp_small pos = 0;

````
- **EN**: This block declares or defines routines around `cf_swap`, `cf_rot`, `cf_pick`; emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_swap`, `cf_rot`, `cf_pick` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1014-1034

````c
  if ((res = mp_int_to_int(n, &pos)) != MP_OK) return res;

  if (pos < 0 || pos >= sp->used - 1) return MP_RANGE;

  return mp_int_copy(sp->elts[sp->used - 2 - pos], n);
}

static mp_result cf_setr(cstate_t *sp) {
  mp_int a = sp->elts[sp->used - 1];
  mp_result res;
  mp_small rdx = 0;

  if ((res = mp_int_to_int(a, &rdx)) != MP_OK) return res;

  if (rdx < MP_MIN_RADIX || rdx > MP_MAX_RADIX) return MP_RANGE;

  g_output_radix = rdx;
  stack_pop(sp);
  return MP_OK;
}

````
- **EN**: This block declares or defines routines around `cf_setr`, `stack_pop`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_setr`, `stack_pop` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1035-1055

````c
static mp_result cf_setbin(cstate_t *sp) {
  g_output_radix = 0;
  return MP_OK;
}

static mp_result cf_help(cstate_t *sp) {
  int ix, maxlen = 10; /* minimum width */

  for (ix = 0; g_ops[ix].name != NULL; ++ix) {
    int len = strlen(g_ops[ix].name);

    if (len > maxlen) maxlen = len;
  }

  fprintf(stderr, "Operators understood:\n");
  for (ix = 0; g_ops[ix].name != NULL; ++ix) {
    int len = strlen(g_ops[ix].name);

    fputs(g_ops[ix].name, stderr);
    while (len++ <= maxlen) fputc(' ', stderr);

````
- **EN**: This block declares or defines routines around `cf_setbin`, `cf_help`, `strlen`, `fprintf` (+1 more); contains control flow with 3 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `cf_setbin`, `cf_help`, `strlen`, `fprintf` (+1 more) 相关的例程; 包含控制流结构：3 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1056-1077

````c
    fprintf(stderr, "%s\n", g_ops[ix].descript);
  }
  fputc('\n', stderr);

  return MP_OK;
}

static mp_result cf_store(cstate_t *sp) {
  mp_result res;

  if (next_token(sp->ifp, sp->ibuf, sp->buflen) != t_symbol) return MP_INPUT;

  if ((res = mem_insert(sp, sp->ibuf, sp->elts[sp->used - 1])) != MP_OK)
    return res;

  return stack_pop(sp);
}

static mp_result cf_recall(cstate_t *sp) {
  mp_result res;
  mp_int val;

````
- **EN**: This block declares or defines routines around `fprintf`, `fputc`, `cf_store`, `cf_recall`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `fprintf`, `fputc`, `cf_store`, `cf_recall` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1078-1098

````c
  if (next_token(sp->ifp, sp->ibuf, sp->buflen) != t_symbol) return MP_INPUT;

  if ((val = mp_int_alloc()) == NULL) return MP_MEMORY;
  if ((res = mem_recall(sp, sp->ibuf, val)) != MP_OK) {
    mp_int_free(val);
    return res;
  }

  return stack_push(sp, val);
}

static mp_result cf_cmem(cstate_t *sp) { return mem_clear(sp); }

static mp_result cf_pmem(cstate_t *sp) {
  int ix, max_len = 0;

  if (sp->mused == 0) {
    fprintf(g_output_file, "<memory empty>\n");
    return MP_OK;
  }

````
- **EN**: This block declares or defines routines around `mp_int_free`, `cf_cmem`, `cf_pmem`, `fprintf`; contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_int_free`, `cf_cmem`, `cf_pmem`, `fprintf` 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1099-1119

````c
  for (ix = 0; (mp_size)ix < sp->mused; ++ix) {
    int ln = strlen(sp->names[ix]);

    if (ln > max_len) max_len = ln;
  }

  max_len += 1; /* allow for a padding space */

  for (ix = 0; (mp_size)ix < sp->mused; ++ix) {
    int ln = strlen(sp->names[ix]);

    fprintf(g_output_file, "%s:", sp->names[ix]);

    while (ln++ < max_len) fputc(' ', g_output_file);

    print_value(sp->mem[ix]);
  }

  return MP_OK;
}

````
- **EN**: This block declares or defines routines around `strlen`, `fprintf`, `print_value`; contains control flow with 3 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `strlen`, `fprintf`, `print_value` 相关的例程; 包含控制流结构：3 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1120-1134

````c
static mp_result cf_qrecall(cstate_t *sp) {
  mp_result res;
  mp_int val;

  if ((val = mp_int_alloc()) == NULL) return MP_MEMORY;

  if ((res = mem_recall(sp, sp->ibuf, val)) != MP_OK) {
    mp_int_free(val);
    return res;
  }

  return stack_push(sp, val);
}

/* Here there be dragons */
````
- **EN**: This block declares or defines routines around `cf_qrecall`, `mp_int_free`; contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `cf_qrecall`, `mp_int_free` 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

## Key Concepts / 关键概念

- **Loop transformation**
  - **CN**: 循环变换
- **Debug and diagnostics**
  - **CN**: 调试与诊断
- **Template/type wrappers**
  - **CN**: 模板与类型包装

## Dependencies / 依赖关系

- **System/standard headers**: `assert.h`, `ctype.h`, `errno.h`, `limits.h`, `stdio.h`, `stdlib.h`, `string.h`, `strings.h` (+5 more) — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`assert.h`, `ctype.h`, `errno.h`, `limits.h`, `stdio.h`, `stdlib.h`, `string.h`, `strings.h` (+5 more) —— 实现所需的标准库或系统声明。
