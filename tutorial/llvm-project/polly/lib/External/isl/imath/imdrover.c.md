# imdrover.c — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/lib/External/isl/imath/imdrover.c` | `polly/lib/External/isl/imath/imdrover.c` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Implements or embeds upstream ISL support code used by Polly. | 实现或内嵌 Polly 使用的上游 ISL 支撑代码。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-26

````c
/*
  Name:     imdrover.c
  Purpose:  Keeper of the hordes of testing code.
  Author:   M. J. Fromberger

  Copyright (C) 2002-2007 Michael J. Fromberger, All Rights Reserved.

  Permission is hereby granted, free of charge, to any person obtaining a copy
  of this software and associated documentation files (the "Software"), to deal
  in the Software without restriction, including without limitation the rights
  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
  copies of the Software, and to permit persons to whom the Software is
  furnished to do so, subject to the following conditions:

  The above copyright notice and this permission notice shall be included in
  all copies or substantial portions of the Software.

  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL THE
  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
  SOFTWARE.
 */

````
- **EN**: This block declares or defines routines around `Copyright`, `files`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `Copyright`, `files` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 27-60

````c
#include <assert.h>
#include <limits.h>
#include <stdlib.h>
#include <string.h>

#include "imath.h"
#include "imdrover.h"
#include "imrat.h"
#include "iprime.h"

/* Globals visible from outside this file */
mp_result imath_errno;
char* imath_errmsg;

/* Set imath_errno and return failure from a test. */
#define FAIL(E) return (imath_errno = (E), false)

/* Check that an expression X yields the expected mp_result value V. */
#define VCHECK(X, V)           \
  do {                         \
    mp_result res_;            \
    if ((res_ = (X)) != (V)) { \
      FAIL(res_);              \
    }                          \
  } while (0)
#define CHECK(X) VCHECK(X, MP_OK)
#define ECHECK(X) VCHECK(X, expect)
#define ACHECK(X)      \
  do {                 \
    if (!(X)) {        \
      FAIL(MP_BADARG); \
    }                  \
  } while (0)

````
- **EN**: This block imports system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; defines macros like `FAIL`, `VCHECK`, `CHECK`, `ECHECK` (+1 more); declares or defines routines around `FAIL`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `FAIL`, `VCHECK`, `CHECK`, `ECHECK` (+1 more); 声明或定义与 `FAIL` 相关的例程；并延续周边实现细节。

### Lines 61-87

````c
#define OUTPUT_LIMIT 2048
#define NUM_REGS 16
#define OTHER_ERROR -1024

static char g_output[OUTPUT_LIMIT];
static mpz_t g_zreg[NUM_REGS];
static mpq_t g_qreg[NUM_REGS];
static unsigned char g_bin1[OUTPUT_LIMIT];
static unsigned char g_bin2[OUTPUT_LIMIT];

extern void trim_line(char* line); /* borrowed from imtest.c */

/* Read in a string with radix tags */
static mp_result read_int_value(mp_int z, char* str);
static mp_result read_rat_value(mp_rat q, char* str);

/* Read in a string with radix tags, as a long (not an mp_int) */
static bool read_long(long* z, char* str);

/* Parse the input and output values and fill in pointers to the
   registers containing them.  Returns true if all is well, false
   in case of error.  Caller allocates in/out to correct sizes. */
static bool parse_int_values(testspec_t* t, mp_int* in, mp_int* out,
                             mp_result* rval);
static bool parse_rat_values(testspec_t* t, mp_rat* in, mp_rat* out,
                             mp_result* rval);

````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; defines macros like `OUTPUT_LIMIT`, `NUM_REGS`, `OTHER_ERROR`; declares or defines routines around `trim_line`, `read_int_value`, `read_rat_value`, `long` (+3 more); preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 定义宏，例如 `OUTPUT_LIMIT`, `NUM_REGS`, `OTHER_ERROR`; 声明或定义与 `trim_line`, `read_int_value`, `read_rat_value`, `long` (+3 more) 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 88-119

````c
/* Parse a result code name and return the corresponding result code */
static bool parse_result_code(char* str, mp_result* code);

/* Read in a dot-delimited binary sequence to the given buffer, and return the
   number of bytes read.  Returns < 0 in case of a syntax error.  Records no
   more than limit bytes. */
static int parse_binary(char* str, unsigned char* buf, int limit);

/* Clean up registers (called from atexit()) */
static void done_testing(void);

/*
 * Utility subroutines for writing tests (explained above)
 */

static mp_result read_int_value(mp_int z, char* str) {
  int radix = 10;

  if (*str == '#') {
    ++str;
    switch (*str) {
      case 'x':
      case 'X':
        radix = 16;
        break;
      case 'd':
      case 'D':
        radix = 10;
        break;
      case 'o':
      case 'O':
        radix = 8;
````
- **EN**: This block declares or defines routines around `parse_result_code`, `parse_binary`, `registers`, `done_testing` (+2 more); contains control flow with 1 conditional check(s), 1 switch dispatch(es); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `parse_result_code`, `parse_binary`, `registers`, `done_testing` (+2 more) 相关的例程; 包含控制流结构：1 处条件判断、1 处分支派发; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 120-151

````c
        break;
      case 'b':
      case 'B':
        radix = 2;
        break;
      default:
        return MP_RANGE;
    }
    ++str;
  }

  return mp_int_read_string(z, radix, str);
}

static mp_result read_rat_value(mp_rat q, char* str) {
  int radix = 10;

  if (*str == '#') {
    ++str;
    switch (*str) {
      case 'x':
      case 'X':
        radix = 16;
        break;
      case 'd':
      case 'D':
        radix = 10;
        break;
      case 'o':
      case 'O':
        radix = 8;
        break;
````
- **EN**: This block declares or defines routines around `read_rat_value`; contains control flow with 1 conditional check(s), 1 switch dispatch(es); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `read_rat_value` 相关的例程; 包含控制流结构：1 处条件判断、1 处分支派发; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 152-183

````c
      case 'b':
      case 'B':
        radix = 2;
        break;
      default:
        return MP_RANGE;
    }
    ++str;
  }

  if (*str == '@')
    return mp_rat_read_decimal(q, radix, str + 1);
  else
    return mp_rat_read_string(q, radix, str);
}

static bool read_long(long* z, char* str) {
  char* end;
  int radix = 10;

  if (*str == '#') {
    ++str;
    switch (*str) {
      case 'x':
      case 'X':
        radix = 16;
        break;
      case 'd':
      case 'D':
        radix = 10;
        break;
      case 'o':
````
- **EN**: This block declares or defines routines around `read_long`; contains control flow with 2 conditional check(s), 1 switch dispatch(es); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `read_long` 相关的例程; 包含控制流结构：2 处条件判断、1 处分支派发; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 184-211

````c
      case 'O':
        radix = 8;
        break;
      case 'b':
      case 'B':
        radix = 2;
        break;
      default:
        return false;
    }
    ++str;
  }

  *z = strtol(str, &end, radix);
  return (end != str && *end == '\0');
}

static bool parse_int_values(testspec_t* t, mp_int* in, mp_int* out,
                             mp_result* rval) {
  int pos = 0;
  char* str;

  if (rval != NULL) *rval = MP_OK; /* default */

  if (in != NULL) {
    for (int i = 0; i < t->num_inputs; ++i) {
      str = t->input[i];

````
- **EN**: This block declares or defines routines around `strtol`, `parse_int_values`; contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `strtol`, `parse_int_values` 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 212-239

````c
      trim_line(str);

      if (*str == '=') {
        int k = abs(atoi(str + 1)) - 1;

        if (k < 0 || k >= i) {
          fprintf(stderr, "Line %d: Invalid input back-reference [%s]\n",
                  t->line, str);
          return false;
        }

        in[i] = in[k];
      } else {
        mp_int reg = g_zreg + pos++; /* grab next free register */

        if (read_int_value(reg, str) != MP_OK) {
          fprintf(stderr, "Line %d: Invalid input value [%s]\n", t->line, str);
          return false;
        }

        in[i] = reg;
      }
    }
  }

  for (int i = 0; i < t->num_outputs; ++i) {
    mp_int reg = g_zreg + pos++;

````
- **EN**: This block declares or defines routines around `trim_line`, `abs`, `fprintf`; contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `trim_line`, `abs`, `fprintf` 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 240-265

````c
    str = t->output[i];

    trim_line(str);

    if (strcmp(str, "?") == 0)
      mp_int_zero(reg);
    else if (*str == '$') {
      mp_result code;

      if (!parse_result_code(str, &code)) {
        fprintf(stderr, "Line %d: Invalid result code [%s]\n", t->line, str);
        return false;
      } else if (rval == NULL) {
        fprintf(stderr, "Line %d: Result code not permitted here [%s]\n",
                t->line, str);
        return false;
      } else
        *rval = code;

      /* Provide a dummy value for the corresponding output */
      mp_int_zero(reg);
    } else if (out != NULL && read_int_value(reg, str) != MP_OK) {
      fprintf(stderr, "Line %d: Invalid output value [%s]\n", t->line, str);
      return false;
    }

````
- **EN**: This block declares or defines routines around `trim_line`, `mp_int_zero`, `fprintf`; contains control flow with 5 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `trim_line`, `mp_int_zero`, `fprintf` 相关的例程; 包含控制流结构：5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 266-293

````c
    if (out != NULL) out[i] = reg;
  }

  return true;
}

static bool parse_rat_values(testspec_t* t, mp_rat* in, mp_rat* out,
                             mp_result* rval) {
  int pos = 0;
  char* str;

  if (rval != NULL) *rval = MP_OK; /* default */

  if (in != NULL) {
    for (int i = 0; i < t->num_inputs; ++i) {
      str = t->input[i];

      trim_line(str);

      if (*str == '=') {
        int k = abs(atoi(str + 1)) - 1;

        if (k < 0 || k >= i) {
          fprintf(stderr, "Line %d: Invalid input back-reference [%s]\n",
                  t->line, str);
          return false;
        }

````
- **EN**: This block declares or defines routines around `parse_rat_values`, `trim_line`, `abs`, `fprintf`; contains control flow with 1 loop construct(s), 5 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `parse_rat_values`, `trim_line`, `abs`, `fprintf` 相关的例程; 包含控制流结构：1 处循环、5 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 294-319

````c
        in[i] = in[k];
      } else {
        mp_rat reg = g_qreg + pos++; /* grab next free register */

        if (read_rat_value(reg, str) != MP_OK) {
          fprintf(stderr, "Line %d: Invalid input value [%s]\n", t->line, str);
          return false;
        }

        in[i] = reg;
      }
    }
  }

  for (int i = 0; i < t->num_outputs; ++i) {
    mp_rat reg = g_qreg + pos++;

    str = t->output[i];

    trim_line(str);

    if (strcmp(str, "?") == 0)
      mp_rat_zero(reg);
    else if (*str == '$') {
      mp_result code;

````
- **EN**: This block declares or defines routines around `fprintf`, `trim_line`, `mp_rat_zero`; contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `fprintf`, `trim_line`, `mp_rat_zero` 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 320-347

````c
      if (!parse_result_code(str, &code)) {
        fprintf(stderr, "Line %d: Invalid result code [%s]\n", t->line, str);
        return false;
      } else if (rval == NULL) {
        fprintf(stderr, "Line %d: Result code not permitted here [%s]\n",
                t->line, str);
        return false;
      } else
        *rval = code;

      /* Provide a dummy value for the corresponding output */
      mp_rat_zero(reg);
    } else if (out != NULL && read_rat_value(reg, str) != MP_OK) {
      fprintf(stderr, "Line %d: Invalid output value [%s]\n", t->line, str);
      return false;
    }

    if (out != NULL) out[i] = reg;
  }

  return true;
}

static bool parse_result_code(char* str, mp_result* code) {
  if (str[0] == '$') {
    if (str[1] == '#') {
      long v;

````
- **EN**: This block declares or defines routines around `fprintf`, `mp_rat_zero`, `parse_result_code`; contains control flow with 6 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `fprintf`, `mp_rat_zero`, `parse_result_code` 相关的例程; 包含控制流结构：6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 348-376

````c
      if (!read_long(&v, str + 2)) return false;

      *code = (mp_result)v;
    } else if (strcmp(str + 1, "MP_OK") == 0 ||
               strcmp(str + 1, "MP_FALSE") == 0) {
      *code = MP_OK;
    } else if (strcmp(str + 1, "MP_TRUE") == 0) {
      *code = MP_TRUE;
    } else if (strcmp(str + 1, "MP_MEMORY") == 0) {
      *code = MP_MEMORY;
    } else if (strcmp(str + 1, "MP_RANGE") == 0) {
      *code = MP_RANGE;
    } else if (strcmp(str + 1, "MP_UNDEF") == 0) {
      *code = MP_UNDEF;
    } else if (strcmp(str + 1, "MP_TRUNC") == 0) {
      *code = MP_TRUNC;
    } else if (strcmp(str + 1, "MP_ROUND_UP") == 0) {
      *code = MP_ROUND_UP;
    } else if (strcmp(str + 1, "MP_ROUND_DOWN") == 0) {
      *code = MP_ROUND_DOWN;
    } else if (strcmp(str + 1, "MP_ROUND_HALF_UP") == 0) {
      *code = MP_ROUND_HALF_UP;
    } else if (strcmp(str + 1, "MP_ROUND_HALF_DOWN") == 0) {
      *code = MP_ROUND_HALF_DOWN;
    } else {
      return false;
    }
  }

````
- **EN**: This block declares or defines routines around `strcmp`; contains control flow with 11 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `strcmp` 相关的例程; 包含控制流结构：11 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 377-406

````c
  return true;
}

static int parse_binary(char* str, unsigned char* buf, int limit) {
  int pos = 0;
  char* tok;

  trim_line(str);

  for (tok = strtok(str, "."); tok != NULL && pos < limit;
       tok = strtok(NULL, ".")) {
    long v;

    if (!read_long(&v, tok) || v > UCHAR_MAX || v < 0) return -1;

    buf[pos++] = (unsigned char)v;
  }

  return pos;
}

static void done_testing(void) {
  int i;

  for (i = 0; i < NUM_REGS; ++i) {
    mp_int_clear(g_zreg + i);
    mp_rat_clear(g_qreg + i);
  }
}

````
- **EN**: This block declares or defines routines around `parse_binary`, `trim_line`, `strtok`, `done_testing` (+2 more); contains control flow with 2 loop construct(s), 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `parse_binary`, `trim_line`, `strtok`, `done_testing` (+2 more) 相关的例程; 包含控制流结构：2 处循环、1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 407-433

````c
/*
 * Global functions visible to callers outside this file.
 */

void init_testing(void) {
  static int is_done = 0;

  if (is_done) return;

  for (int i = 0; i < NUM_REGS; ++i) {
    assert(mp_int_init(g_zreg + i) == MP_OK);
    assert(mp_rat_init(g_qreg + i) == MP_OK);
  }

  imath_errmsg = g_output;

  assert(atexit(done_testing) == 0);
  is_done = 1;
}

void reset_registers(void) {
  for (int i = 0; i < NUM_REGS; ++i) {
    mp_int_zero(g_zreg + i);
    mp_rat_zero(g_qreg + i);
  }
}

````
- **EN**: This block declares or defines routines around `init_testing`, `reset_registers`, `mp_int_zero`, `mp_rat_zero`; contains control flow with 2 loop construct(s), 1 conditional check(s); adds defensive checks for invariants or impossible states; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `init_testing`, `reset_registers`, `mp_int_zero`, `mp_rat_zero` 相关的例程; 包含控制流结构：2 处循环、1 处条件判断; 加入用于保护不变量或异常状态的断言/检查; 保留用于解释意图、用法或算法背景的注释.

### Lines 434-463

````c
bool test_init(testspec_t* t, FILE* ofp) {
  mp_int in[2], out[1];
  mp_small v;
  mp_usmall uv;
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));

  if (strcmp(t->code, "initu") == 0) {
    CHECK(mp_int_to_uint(in[1], &uv));
    ECHECK(mp_int_init_uvalue(in[0], uv));
  } else { /* initv */
    CHECK(mp_int_to_int(in[1], &v));
    ECHECK(mp_int_init_value(in[0], v));
  }

  if (expect == MP_OK && mp_int_compare(in[0], out[0]) != 0) {
    mp_int_to_string(in[0], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }

  return true;
}

bool test_set(testspec_t* t, FILE* ofp) {
  mp_int in[2], out[1];
  mp_small v;
  mp_usmall uv;
  mp_result expect;

````
- **EN**: This block declares or defines routines around `test_init`, `ACHECK`, `CHECK`, `ECHECK` (+3 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_init`, `ACHECK`, `CHECK`, `ECHECK` (+3 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 464-493

````c
  ACHECK(parse_int_values(t, in, out, &expect));

  if (strcmp(t->code, "setu") == 0) {
    CHECK(mp_int_to_uint(in[1], &uv));
    ECHECK(mp_int_set_uvalue(in[0], uv));
  } else { /* setv */
    CHECK(mp_int_to_int(in[1], &v));
    ECHECK(mp_int_set_value(in[0], v));
  }

  if (expect == MP_OK && mp_int_compare(in[0], out[0]) != 0) {
    mp_int_to_string(in[0], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }

  return true;
}

bool test_neg(testspec_t* t, FILE* ofp) {
  mp_int in[2], out[1];
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_neg(in[0], in[1]));

  if (expect == MP_OK && mp_int_compare(in[1], out[0]) != 0) {
    mp_int_to_string(in[1], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }

````
- **EN**: This block declares or defines routines around `ACHECK`, `CHECK`, `ECHECK`, `mp_int_to_string` (+2 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `ACHECK`, `CHECK`, `ECHECK`, `mp_int_to_string` (+2 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 494-525

````c
  return true;
}

bool test_abs(testspec_t* t, FILE* ofp) {
  mp_int in[2], out[1];
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_abs(in[0], in[1]));

  if (expect == MP_OK && mp_int_compare(in[1], out[0]) != 0) {
    mp_int_to_string(in[1], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }

  return true;
}

bool test_add(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_small v;
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));

  if (strcmp(t->code, "addv") == 0) {
    CHECK(mp_int_to_int(in[1], &v));
    ECHECK(mp_int_add_value(in[0], v, in[2]));
  } else {
    ECHECK(mp_int_add(in[0], in[1], in[2]));
  }

````
- **EN**: This block declares or defines routines around `test_abs`, `ACHECK`, `ECHECK`, `mp_int_to_string` (+3 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_abs`, `ACHECK`, `ECHECK`, `mp_int_to_string` (+3 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 526-554

````c
  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }

  return true;
}

bool test_sub(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_small v;
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));

  if (strcmp(t->code, "subv") == 0) {
    CHECK(mp_int_to_int(in[1], &v));
    ECHECK(mp_int_sub_value(in[0], v, in[2]));
  } else {
    ECHECK(mp_int_sub(in[0], in[1], in[2]));
  }

  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `mp_int_to_string`, `FAIL`, `test_sub`, `ACHECK` (+2 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_int_to_string`, `FAIL`, `test_sub`, `ACHECK` (+2 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 555-584

````c
bool test_mul(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_mul(in[0], in[1], in[2]));

  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_mulp2(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_result expect;
  mp_small p2;

  ACHECK(parse_int_values(t, in, out, &expect));
  CHECK(mp_int_to_int(in[1], &p2));
  ECHECK(mp_int_mul_pow2(in[0], p2, in[2]));

  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `test_mul`, `ACHECK`, `ECHECK`, `mp_int_to_string` (+3 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_mul`, `ACHECK`, `ECHECK`, `mp_int_to_string` (+3 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 585-614

````c
bool test_mulv(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_result expect;
  mp_small v;

  ACHECK(parse_int_values(t, in, out, &expect));
  CHECK(mp_int_to_int(in[1], &v));
  ECHECK(mp_int_mul_value(in[0], v, in[2]));

  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_sqr(testspec_t* t, FILE* ofp) {
  mp_int in[2], out[1];
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_sqr(in[0], in[1]));

  if (expect == MP_OK && mp_int_compare(in[1], out[0]) != 0) {
    mp_int_to_string(in[1], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `test_mulv`, `ACHECK`, `CHECK`, `ECHECK` (+3 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_mulv`, `ACHECK`, `CHECK`, `ECHECK` (+3 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 615-640

````c
bool test_div(testspec_t* t, FILE* ofp) {
  mp_int in[4], out[2];
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_div(in[0], in[1], in[2], in[3]));

  if (expect == MP_OK && ((mp_int_compare(in[2], out[0]) != 0) ||
                          (mp_int_compare(in[3], out[1]) != 0))) {
    int len;
    char* str;

    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    str = g_output + (len = strlen(g_output));
    *str++ = ',';
    mp_int_to_string(in[3], 10, str, OUTPUT_LIMIT - (len + 1));
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_divp2(testspec_t* t, FILE* ofp) {
  mp_int in[4], out[2];
  mp_result expect;
  mp_small p2;

````
- **EN**: This block declares or defines routines around `test_div`, `ACHECK`, `ECHECK`, `mp_int_compare` (+4 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_div`, `ACHECK`, `ECHECK`, `mp_int_compare` (+4 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 641-668

````c
  ACHECK(parse_int_values(t, in, out, &expect));
  CHECK(mp_int_to_int(in[1], &p2));
  ECHECK(mp_int_div_pow2(in[0], p2, in[2], in[3]));

  if (expect == MP_OK && ((mp_int_compare(in[2], out[0]) != 0) ||
                          (mp_int_compare(in[3], out[1]) != 0))) {
    int len;
    char* str;

    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    str = g_output + (len = strlen(g_output));
    *str++ = ',';
    mp_int_to_string(in[3], 10, str, OUTPUT_LIMIT - (len + 1));
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_divv(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[2];
  mp_result expect;
  mp_small v, rem, orem;

  ACHECK(parse_int_values(t, in, out, &expect));
  CHECK(mp_int_to_int(in[1], &v));
  CHECK(mp_int_to_int(out[1], &orem));
  ECHECK(mp_int_div_value(in[0], v, in[2], &rem));

````
- **EN**: This block declares or defines routines around `ACHECK`, `CHECK`, `ECHECK`, `mp_int_compare` (+4 more); contains control flow with 1 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `ACHECK`, `CHECK`, `ECHECK`, `mp_int_compare` (+4 more) 相关的例程; 包含控制流结构：1 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 669-697

````c
  if (expect == MP_OK &&
      ((mp_int_compare(in[2], out[0]) != 0) || (rem != orem))) {
    char* str;

    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    str = g_output + strlen(g_output);
    *str++ = ',';
    sprintf(str, "%ld", rem);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_expt(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_result expect;
  mp_small pow;

  ACHECK(parse_int_values(t, in, out, &expect));
  CHECK(mp_int_to_int(in[1], &pow));
  ECHECK(mp_int_expt(in[0], pow, in[2]));

  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `mp_int_compare`, `mp_int_to_string`, `strlen`, `sprintf` (+5 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_int_compare`, `mp_int_to_string`, `strlen`, `sprintf` (+5 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 698-728

````c
bool test_exptv(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_result expect;
  mp_small a, b;

  ACHECK(parse_int_values(t, in, out, &expect));
  CHECK(mp_int_to_int(in[0], &a));
  CHECK(mp_int_to_int(in[1], &b));
  ECHECK(mp_int_expt_value(a, b, in[2]));

  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_exptf(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_expt_full(in[0], in[1], in[2]));

  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `test_exptv`, `ACHECK`, `CHECK`, `ECHECK` (+3 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_exptv`, `ACHECK`, `CHECK`, `ECHECK` (+3 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 729-756

````c
bool test_mod(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_mod(in[0], in[1], in[2]));

  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_gcd(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_gcd(in[0], in[1], in[2]));

  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `test_mod`, `ACHECK`, `ECHECK`, `mp_int_to_string` (+2 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_mod`, `ACHECK`, `ECHECK`, `mp_int_to_string` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 757-792

````c
bool test_egcd(testspec_t* t, FILE* ofp) {
  mp_int in[5], out[3], t1 = g_zreg + 8, t2 = g_zreg + 9;
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_egcd(in[0], in[1], in[2], in[3], in[4]));

  /* If we got an error we expected, return success immediately */
  if (expect != MP_OK) return true;

  if ((mp_int_compare(in[2], out[0]) != 0) ||
      (mp_int_compare(in[3], out[1]) != 0) ||
      (mp_int_compare(in[4], out[2]) != 0)) {
    int len, len2;
    char* str;

    /* Failure might occur because the tester computed x and y in a different
       way than we did.  Verify that the results are correct before reporting
       an error. */
    mp_int_mul(in[3], in[0], t1);
    mp_int_mul(in[4], in[1], t2);
    mp_int_add(t1, t2, t2);
    if (mp_int_compare(t2, in[2]) == 0) return true;

    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    str = g_output + (len = strlen(g_output));
    *str++ = ',';
    mp_int_to_string(in[3], 10, str, OUTPUT_LIMIT - (len + 1));
    str = str + (len2 = strlen(str));
    *str++ = ',';
    mp_int_to_string(in[4], 10, str, OUTPUT_LIMIT - (len + len2 + 2));
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `test_egcd`, `ACHECK`, `ECHECK`, `mp_int_compare` (+5 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `test_egcd`, `ACHECK`, `ECHECK`, `mp_int_compare` (+5 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 793-820

````c
bool test_lcm(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_lcm(in[0], in[1], in[2]));

  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_sqrt(testspec_t* t, FILE* ofp) {
  mp_int in[2], out[1];
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_sqrt(in[0], in[1]));

  if (expect == MP_OK && mp_int_compare(in[1], out[0]) != 0) {
    mp_int_to_string(in[1], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `test_lcm`, `ACHECK`, `ECHECK`, `mp_int_to_string` (+2 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_lcm`, `ACHECK`, `ECHECK`, `mp_int_to_string` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 821-850

````c
bool test_root(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_small v;
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  CHECK(mp_int_to_int(in[1], &v));
  ECHECK(mp_int_root(in[0], v, in[2]));

  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_invmod(testspec_t* t, FILE* ofp) {
  mp_int in[3], out[1];
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_invmod(in[0], in[1], in[2]));

  if (expect == MP_OK && mp_int_compare(in[2], out[0]) != 0) {
    mp_int_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `test_root`, `ACHECK`, `CHECK`, `ECHECK` (+3 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_root`, `ACHECK`, `CHECK`, `ECHECK` (+3 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 851-880

````c
bool test_exptmod(testspec_t* t, FILE* ofp) {
  mp_int in[4], out[1];
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_exptmod(in[0], in[1], in[2], in[3]));

  if (expect == MP_OK && mp_int_compare(in[3], out[0]) != 0) {
    mp_int_to_string(in[3], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_exptmod_ev(testspec_t* t, FILE* ofp) {
  mp_int in[4], out[1];
  mp_result expect;
  mp_small v;

  ACHECK(parse_int_values(t, in, out, &expect));
  CHECK(mp_int_to_int(in[1], &v));
  ECHECK(mp_int_exptmod_evalue(in[0], v, in[2], in[3]));

  if (expect == MP_OK && mp_int_compare(in[3], out[0]) != 0) {
    mp_int_to_string(in[3], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `test_exptmod`, `ACHECK`, `ECHECK`, `mp_int_to_string` (+3 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_exptmod`, `ACHECK`, `ECHECK`, `mp_int_to_string` (+3 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 881-910

````c
bool test_exptmod_bv(testspec_t* t, FILE* ofp) {
  mp_int in[4], out[1];
  mp_result expect;
  mp_small v;

  ACHECK(parse_int_values(t, in, out, &expect));
  CHECK(mp_int_to_int(in[0], &v));
  ECHECK(mp_int_exptmod_bvalue(v, in[1], in[2], in[3]));

  if (expect == MP_OK && mp_int_compare(in[3], out[0]) != 0) {
    mp_int_to_string(in[3], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_comp(testspec_t* t, FILE* ofp) {
  mp_int in[2];
  mp_result res, expect;

  ACHECK(parse_int_values(t, in, NULL, &expect));

  if ((res = mp_int_compare(in[0], in[1])) != expect) {
    sprintf(g_output, "Incorrect comparison result (want %d, got %d)", expect,
            res);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `test_exptmod_bv`, `ACHECK`, `CHECK`, `ECHECK` (+4 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_exptmod_bv`, `ACHECK`, `CHECK`, `ECHECK` (+4 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 911-938

````c
bool test_ucomp(testspec_t* t, FILE* ofp) {
  mp_int in[2];
  mp_result res, expect;

  ACHECK(parse_int_values(t, in, NULL, &expect));

  if ((res = mp_int_compare_unsigned(in[0], in[1])) != expect) {
    sprintf(g_output, "Incorrect comparison result (want %d, got %d)", expect,
            res);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_zcomp(testspec_t* t, FILE* ofp) {
  mp_int in[1];
  mp_result res, expect;

  ACHECK(parse_int_values(t, in, NULL, &expect));

  if ((res = mp_int_compare_zero(in[0])) != expect) {
    sprintf(g_output, "Incorrect comparison result (want %d, got %d)", expect,
            res);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `test_ucomp`, `ACHECK`, `sprintf`, `FAIL` (+1 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_ucomp`, `ACHECK`, `sprintf`, `FAIL` (+1 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 939-970

````c
bool test_vcomp(testspec_t* t, FILE* ofp) {
  mp_int in[2];
  mp_result res, expect;
  mp_small v;

  ACHECK(parse_int_values(t, in, NULL, &expect));

  v = atoi(t->input[1]);
  if ((res = mp_int_compare_value(in[0], v)) != expect) {
    sprintf(g_output, "Incorrect comparison result (want %d, got %d)", expect,
            res);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_uvcomp(testspec_t* t, FILE* ofp) {
  mp_int in[2];
  mp_result res, expect;
  mp_usmall v;

  ACHECK(parse_int_values(t, in, NULL, &expect));

  v = strtoul(t->input[1], NULL, 0);
  if ((res = mp_int_compare_uvalue(in[0], v)) != expect) {
    sprintf(g_output, "Incorrect comparison result (want %d, got %d)", expect,
            res);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `test_vcomp`, `ACHECK`, `atoi`, `sprintf` (+3 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_vcomp`, `ACHECK`, `atoi`, `sprintf` (+3 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 971-996

````c
bool test_tostr(testspec_t* t, FILE* ofp) {
  mp_int in[2];
  mp_small radix;
  mp_result len;

  ACHECK(parse_int_values(t, in, NULL, NULL));
  ACHECK(mp_int_to_int(in[1], &radix) == MP_OK);

  if (radix < MP_MIN_RADIX || radix > MP_MAX_RADIX) FAIL(MP_RANGE);

  trim_line(t->output[0]);
  len = mp_int_string_len(in[0], radix);

  CHECK(mp_int_to_string(in[0], radix, g_output, len));

  if (strcmp(t->output[0], g_output) != 0) FAIL(OTHER_ERROR);

  return true;
}

bool test_tobin(testspec_t* t, FILE* ofp) {
  mp_int in[1];
  int test_len, out_len;

  ACHECK(parse_int_values(t, in, NULL, NULL));

````
- **EN**: This block declares or defines routines around `test_tostr`, `ACHECK`, `trim_line`, `mp_int_string_len` (+2 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_tostr`, `ACHECK`, `trim_line`, `mp_int_string_len` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 997-1025

````c
  trim_line(t->output[0]);
  if ((out_len = parse_binary(t->output[0], g_bin1, sizeof(g_bin1))) < 0)
    FAIL(MP_BADARG);

  if ((test_len = mp_int_binary_len(in[0])) != out_len) {
    sprintf(g_output, "Output lengths do not match (want %d, got %d)", test_len,
            out_len);
    FAIL(OTHER_ERROR);
  }

  CHECK(mp_int_to_binary(in[0], g_bin2, sizeof(g_bin2)));

  if (memcmp(g_bin1, g_bin2, test_len) != 0) {
    int pos = 0, i;

    for (i = 0; i < test_len - 1; ++i)
      pos += sprintf(g_output + pos, "%d.", g_bin2[i]);

    sprintf(g_output + pos, "%d", g_bin2[i]);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_to_int(testspec_t* t, FILE* ofp) {
  mp_int in[1], out[1];
  mp_small v;
  mp_result expect;

````
- **EN**: This block declares or defines routines around `trim_line`, `FAIL`, `sprintf`, `CHECK` (+1 more); contains control flow with 1 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `trim_line`, `FAIL`, `sprintf`, `CHECK` (+1 more) 相关的例程; 包含控制流结构：1 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1026-1055

````c
  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_to_int(in[0], &v));

  if (expect == MP_OK && mp_int_compare_value(out[0], v) != 0) {
    sprintf(g_output, "Incorrect value (got %ld)", v);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_to_uint(testspec_t* t, FILE* ofp) {
  mp_int in[1], out[1];
  mp_usmall v;
  mp_result expect;

  ACHECK(parse_int_values(t, in, out, &expect));
  ECHECK(mp_int_to_uint(in[0], &v));

  if (expect == MP_OK && mp_int_compare_uvalue(out[0], v) != 0) {
    sprintf(g_output, "Incorrect value (got %lu)", v);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_read_binary(testspec_t* t, FILE* ofp) {
  mp_int out[1], in = g_zreg + 1;
  int in_len;
  mp_result expect;

````
- **EN**: This block declares or defines routines around `ACHECK`, `ECHECK`, `sprintf`, `FAIL` (+2 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `ACHECK`, `ECHECK`, `sprintf`, `FAIL` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1056-1086

````c
  ACHECK(parse_int_values(t, NULL, out, &expect));

  trim_line(t->input[0]);
  if ((in_len = parse_binary(t->input[0], g_bin1, sizeof(g_bin1))) < 0)
    FAIL(MP_BADARG);

  ECHECK(mp_int_read_binary(in, g_bin1, in_len));

  if (expect == MP_OK && mp_int_compare(in, out[0]) != 0) {
    mp_int_to_string(in, 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_to_uns(testspec_t* t, FILE* ofp) {
  mp_int in[1];
  int test_len, out_len;

  ACHECK(parse_int_values(t, in, NULL, NULL));

  trim_line(t->output[0]);
  if ((out_len = parse_binary(t->output[0], g_bin1, sizeof(g_bin1))) < 0)
    FAIL(MP_BADARG);

  if ((test_len = mp_int_unsigned_len(in[0])) != out_len) {
    sprintf(g_output, "Output lengths do not match (want %d, got %d)", test_len,
            out_len);
    FAIL(OTHER_ERROR);
  }

````
- **EN**: This block declares or defines routines around `ACHECK`, `trim_line`, `FAIL`, `ECHECK` (+3 more); contains control flow with 4 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `ACHECK`, `trim_line`, `FAIL`, `ECHECK` (+3 more) 相关的例程; 包含控制流结构：4 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1087-1113

````c
  CHECK(mp_int_to_unsigned(in[0], g_bin2, sizeof(g_bin2)));

  if (memcmp(g_bin1, g_bin2, test_len) != 0) {
    int pos = 0, i;

    for (i = 0; i < test_len - 1; ++i)
      pos += sprintf(g_output + pos, "%d.", g_bin2[i]);

    sprintf(g_output + pos, "%d", g_bin2[i]);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_read_uns(testspec_t* t, FILE* ofp) {
  mp_int out[1], in = g_zreg + 1;
  int in_len;
  mp_result expect;

  ACHECK(parse_int_values(t, NULL, out, &expect));

  trim_line(t->input[0]);
  if ((in_len = parse_binary(t->input[0], g_bin1, sizeof(g_bin1))) < 0)
    FAIL(MP_BADARG);

  ECHECK(mp_int_read_unsigned(in, g_bin1, in_len));

````
- **EN**: This block declares or defines routines around `CHECK`, `sprintf`, `FAIL`, `test_read_uns` (+3 more); contains control flow with 1 loop construct(s), 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `CHECK`, `sprintf`, `FAIL`, `test_read_uns` (+3 more) 相关的例程; 包含控制流结构：1 处循环、2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1114-1144

````c
  if (expect == MP_OK && mp_int_compare(in, out[0]) != 0) {
    mp_int_to_string(in, 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_meta(testspec_t* t, FILE* ofp) {
  mp_int *in = NULL, *out = NULL;
  int i, j;
  mp_result expect;

  if (t->num_inputs > 0) {
    in = calloc(t->num_inputs, sizeof(mp_int));
  }
  if (t->num_outputs > 0) {
    out = calloc(t->num_outputs, sizeof(mp_int));
  }

  if (!parse_int_values(t, in, out, &expect)) {
    if (in != NULL) free(in);
    if (out != NULL) free(out);
    FAIL(MP_BADARG);
  }

  fprintf(ofp, "Test '%s' defined at line %d\n", t->code, t->line);
  fprintf(ofp, "Expected result: %d\n", expect);
  fprintf(ofp, "Input values: %d\n", t->num_inputs);
  for (i = 0; i < t->num_inputs; ++i) {
    mp_int_to_string(in[i], 10, g_output, OUTPUT_LIMIT);

````
- **EN**: This block declares or defines routines around `mp_int_to_string`, `FAIL`, `test_meta`, `calloc` (+1 more); contains control flow with 1 loop construct(s), 6 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_int_to_string`, `FAIL`, `test_meta`, `calloc` (+1 more) 相关的例程; 包含控制流结构：1 处循环、6 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1145-1172

````c
    fprintf(ofp, " %2d.) %s", i + 1, g_output);

    for (j = i - 1; j >= 0; --j)
      if (in[j] == in[i]) {
        fprintf(ofp, " (=> %d)", j + 1);
        break;
      }

    fputc('\n', ofp);
  }
  fprintf(ofp, "Output values: %d\n", t->num_outputs);
  for (i = 0; i < t->num_outputs; ++i) {
    mp_int_to_string(out[i], 10, g_output, OUTPUT_LIMIT);

    fprintf(ofp, " %2d.) %s\n", i + 1, g_output);
  }
  if (in != NULL) free(in);
  if (out != NULL) free(out);
  return true;
}

bool test_qneg(testspec_t* t, FILE* ofp) {
  mp_rat in[2], out[1];
  mp_result expect;

  ACHECK(parse_rat_values(t, in, out, &expect));
  ECHECK(mp_rat_neg(in[0], in[1]));

````
- **EN**: This block declares or defines routines around `fprintf`, `fputc`, `mp_int_to_string`, `test_qneg` (+2 more); contains control flow with 2 loop construct(s), 3 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `fprintf`, `fputc`, `mp_int_to_string`, `test_qneg` (+2 more) 相关的例程; 包含控制流结构：2 处循环、3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1173-1200

````c
  if (expect == MP_OK && mp_rat_compare(in[1], out[0]) != 0) {
    mp_rat_to_string(in[1], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_qrecip(testspec_t* t, FILE* ofp) {
  mp_rat in[2], out[1];
  mp_result expect;

  ACHECK(parse_rat_values(t, in, out, &expect));
  ECHECK(mp_rat_recip(in[0], in[1]));

  if (expect == MP_OK && mp_rat_compare(in[1], out[0]) != 0) {
    mp_rat_to_string(in[1], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_qabs(testspec_t* t, FILE* ofp) {
  mp_rat in[2], out[1];
  mp_result expect;

  ACHECK(parse_rat_values(t, in, out, &expect));
  ECHECK(mp_rat_abs(in[0], in[1]));

````
- **EN**: This block declares or defines routines around `mp_rat_to_string`, `FAIL`, `test_qrecip`, `ACHECK` (+2 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_rat_to_string`, `FAIL`, `test_qrecip`, `ACHECK` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1201-1228

````c
  if (expect == MP_OK && mp_rat_compare(in[1], out[0]) != 0) {
    mp_rat_to_string(in[1], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_qadd(testspec_t* t, FILE* ofp) {
  mp_rat in[3], out[1];
  mp_result expect;

  ACHECK(parse_rat_values(t, in, out, &expect));
  ECHECK(mp_rat_add(in[0], in[1], in[2]));

  if (expect == MP_OK && mp_rat_compare(in[2], out[0]) != 0) {
    mp_rat_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_qsub(testspec_t* t, FILE* ofp) {
  mp_rat in[3], out[1];
  mp_result expect;

  ACHECK(parse_rat_values(t, in, out, &expect));
  ECHECK(mp_rat_sub(in[0], in[1], in[2]));

````
- **EN**: This block declares or defines routines around `mp_rat_to_string`, `FAIL`, `test_qadd`, `ACHECK` (+2 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_rat_to_string`, `FAIL`, `test_qadd`, `ACHECK` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1229-1256

````c
  if (expect == MP_OK && mp_rat_compare(in[2], out[0]) != 0) {
    mp_rat_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_qmul(testspec_t* t, FILE* ofp) {
  mp_rat in[3], out[1];
  mp_result expect;

  ACHECK(parse_rat_values(t, in, out, &expect));
  ECHECK(mp_rat_mul(in[0], in[1], in[2]));

  if (expect == MP_OK && mp_rat_compare(in[2], out[0]) != 0) {
    mp_rat_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_qdiv(testspec_t* t, FILE* ofp) {
  mp_rat in[3], out[1];
  mp_result expect;

  ACHECK(parse_rat_values(t, in, out, &expect));
  ECHECK(mp_rat_div(in[0], in[1], in[2]));

````
- **EN**: This block declares or defines routines around `mp_rat_to_string`, `FAIL`, `test_qmul`, `ACHECK` (+2 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_rat_to_string`, `FAIL`, `test_qmul`, `ACHECK` (+2 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1257-1285

````c
  if (expect == MP_OK && mp_rat_compare(in[2], out[0]) != 0) {
    mp_rat_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_qaddz(testspec_t* t, FILE* ofp) {
  mp_rat in[3], out[1];
  mp_result expect;

  ACHECK(parse_rat_values(t, in, out, &expect));

  if (!mp_rat_is_integer(in[1])) {
    fprintf(stderr,
            "Line %d: Second argument must be an integer (test_qaddz)\n",
            t->line);
    FAIL(MP_BADARG);
  }

  ECHECK(mp_rat_add_int(in[0], MP_NUMER_P(in[1]), in[2]));

  if (expect == MP_OK && mp_rat_compare(in[2], out[0]) != 0) {
    mp_rat_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `mp_rat_to_string`, `FAIL`, `test_qaddz`, `ACHECK` (+3 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_rat_to_string`, `FAIL`, `test_qaddz`, `ACHECK` (+3 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1286-1311

````c
bool test_qsubz(testspec_t* t, FILE* ofp) {
  mp_rat in[3], out[1];
  mp_result expect;

  ACHECK(parse_rat_values(t, in, out, &expect));

  if (!mp_rat_is_integer(in[1])) {
    fprintf(stderr,
            "Line %d: Second argument must be an integer (test_qsubz)\n",
            t->line);
    FAIL(MP_BADARG);
  }

  ECHECK(mp_rat_sub_int(in[0], MP_NUMER_P(in[1]), in[2]));

  if (expect == MP_OK && mp_rat_compare(in[2], out[0]) != 0) {
    mp_rat_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_qmulz(testspec_t* t, FILE* ofp) {
  mp_rat in[3], out[1];
  mp_result expect;

````
- **EN**: This block declares or defines routines around `test_qsubz`, `ACHECK`, `fprintf`, `integer` (+4 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_qsubz`, `ACHECK`, `fprintf`, `integer` (+4 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1312-1342

````c
  ACHECK(parse_rat_values(t, in, out, &expect));

  if (!mp_rat_is_integer(in[1])) {
    fprintf(stderr,
            "Line %d: Second argument must be an integer (test_qmulz)\n",
            t->line);
    FAIL(MP_BADARG);
  }

  ECHECK(mp_rat_mul_int(in[0], MP_NUMER_P(in[1]), in[2]));

  if (expect == MP_OK && mp_rat_compare(in[2], out[0]) != 0) {
    mp_rat_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_qdivz(testspec_t* t, FILE* ofp) {
  mp_rat in[3], out[1];
  mp_result expect;

  ACHECK(parse_rat_values(t, in, out, &expect));

  if (!mp_rat_is_integer(in[1])) {
    fprintf(stderr,
            "Line %d: Second argument must be an integer (test_qdivz)\n",
            t->line);
    FAIL(MP_BADARG);
  }

````
- **EN**: This block declares or defines routines around `ACHECK`, `fprintf`, `integer`, `FAIL` (+3 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `ACHECK`, `fprintf`, `integer`, `FAIL` (+3 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1343-1368

````c
  ECHECK(mp_rat_div_int(in[0], MP_NUMER_P(in[1]), in[2]));

  if (expect == MP_OK && mp_rat_compare(in[2], out[0]) != 0) {
    mp_rat_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_qexpt(testspec_t* t, FILE* ofp) {
  mp_rat in[3], out[1];
  mp_result expect;
  mp_small power;

  ACHECK(parse_rat_values(t, in, out, &expect));

  if (!mp_rat_is_integer(in[1])) {
    fprintf(stderr,
            "Line %d: Second argument must be an integer (test_qexpt)\n",
            t->line);
    FAIL(MP_BADARG);
  }

  CHECK(mp_int_to_int(MP_NUMER_P(in[1]), &power));
  ECHECK(mp_rat_expt(in[0], power, in[2]));

````
- **EN**: This block declares or defines routines around `ECHECK`, `mp_rat_to_string`, `FAIL`, `test_qexpt` (+4 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `ECHECK`, `mp_rat_to_string`, `FAIL`, `test_qexpt` (+4 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1369-1394

````c
  if (expect == MP_OK && mp_rat_compare(in[2], out[0]) != 0) {
    mp_rat_to_string(in[2], 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

bool test_qtostr(testspec_t* t, FILE* ofp) {
  mp_rat in[2];
  long radix;
  mp_result len;

  ACHECK(parse_rat_values(t, in, NULL, NULL));
  trim_line(t->input[1]);
  ACHECK(read_long(&radix, t->input[1]));

  if (radix < MP_MIN_RADIX || radix > MP_MAX_RADIX) {
    fprintf(stderr, "Line %d: Radix %ld out of range\n", t->line, radix);
    FAIL(MP_RANGE);
  }

  trim_line(t->output[0]);
  len = mp_rat_string_len(in[0], radix);

  CHECK(mp_rat_to_string(in[0], radix, g_output, len));

````
- **EN**: This block declares or defines routines around `mp_rat_to_string`, `FAIL`, `test_qtostr`, `ACHECK` (+4 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `mp_rat_to_string`, `FAIL`, `test_qtostr`, `ACHECK` (+4 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1395-1421

````c
  if (strcmp(t->output[0], g_output) != 0) FAIL(OTHER_ERROR);

  return true;
}

bool test_qtodec(testspec_t* t, FILE* ofp) {
  mp_rat in[4];
  long radix, prec, m;
  mp_round_mode rmode;
  mp_result res, expect = MP_OK, len;

  ACHECK(parse_rat_values(t, in, NULL, NULL));

  if (t->output[0][0] == '$' && !parse_result_code(t->output[0], &expect)) {
    fprintf(stderr, "Line %d: Invalid result code [%s]\n", t->line,
            t->output[0]);
    FAIL(OTHER_ERROR);
  }

  trim_line(t->input[1]);
  trim_line(t->input[2]);
  trim_line(t->input[3]);
  ACHECK(read_long(&radix, t->input[1]));
  ACHECK(read_long(&prec, t->input[2]));
  ACHECK(read_long(&m, t->input[3]));
  rmode = (mp_round_mode)m;

````
- **EN**: This block declares or defines routines around `test_qtodec`, `ACHECK`, `fprintf`, `FAIL` (+1 more); contains control flow with 2 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `test_qtodec`, `ACHECK`, `fprintf`, `FAIL` (+1 more) 相关的例程; 包含控制流结构：2 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1422-1452

````c
  if (prec < 0) {
    fprintf(stderr, "Line %d: Precision %ld out of range\n", t->line, prec);
    FAIL(MP_RANGE);
  }

  trim_line(t->output[0]);
  len = mp_rat_decimal_len(in[0], radix, prec);
  ECHECK((res = mp_rat_to_decimal(in[0], radix, prec, rmode, g_output, len)));

  if (res == MP_OK && strcmp(t->output[0], g_output) != 0) FAIL(OTHER_ERROR);

  return true;
}

bool test_qrdec(testspec_t* t, FILE* ofp) {
  mp_rat out[1] = {NULL}, reg = g_qreg + 1;
  long radix;
  mp_result expect;

  ACHECK(parse_rat_values(t, NULL, out, &expect));
  trim_line(t->input[1]);
  ACHECK(read_long(&radix, t->input[1]));

  ECHECK(mp_rat_read_decimal(reg, radix, t->input[0]));
  if (expect == MP_OK && mp_rat_compare(reg, out[0]) != 0) {
    mp_rat_to_string(reg, 10, g_output, OUTPUT_LIMIT);
    FAIL(OTHER_ERROR);
  }
  return true;
}

````
- **EN**: This block declares or defines routines around `fprintf`, `FAIL`, `trim_line`, `mp_rat_decimal_len` (+4 more); contains control flow with 3 conditional check(s); emits return paths that hand results or status codes back to callers.
- **CN**: 该代码块 声明或定义与 `fprintf`, `FAIL`, `trim_line`, `mp_rat_decimal_len` (+4 more) 相关的例程; 包含控制流结构：3 处条件判断; 包含返回路径，用于向调用者交回结果或状态码.

### Lines 1453-1462

````c
bool test_is_prime(testspec_t* t, FILE* OFP) {
  mp_int in[1] = {NULL};
  mp_result expect;

  ACHECK(parse_int_values(t, in, NULL, &expect));
  ECHECK(mp_int_is_prime(in[0]));
  return true;
}

/* Here there be dragons */
````
- **EN**: This block declares or defines routines around `test_is_prime`, `ACHECK`, `ECHECK`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `test_is_prime`, `ACHECK`, `ECHECK` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

## Key Concepts / 关键概念

- **Loop transformation**
  - **CN**: 循环变换
- **Debug and diagnostics**
  - **CN**: 调试与诊断

## Dependencies / 依赖关系

- **System/standard headers**: `assert.h`, `limits.h`, `stdlib.h`, `string.h`, `imath.h`, `imdrover.h`, `imrat.h`, `iprime.h` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`assert.h`, `limits.h`, `stdlib.h`, `string.h`, `imath.h`, `imdrover.h`, `imrat.h`, `iprime.h` —— 实现所需的标准库或系统声明。
