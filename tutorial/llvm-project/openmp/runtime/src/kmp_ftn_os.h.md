# kmp_ftn_os.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `openmp/runtime/src/kmp_ftn_os.h`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN) / 用途（英文）**: kmp_ftn_os.h -- KPTS Fortran defines header file.
- **Purpose (CN) / 用途（中文）**: 声明 OpenMP 运行时：线程/团队管理、调度、同步、环境处理以及 OMPT 钩子。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```cpp
   1: /*
   2:  * kmp_ftn_os.h -- KPTS Fortran defines header file.
   3:  */
   4: 
   5: //===----------------------------------------------------------------------===//
   6: //
   7: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   8: // See https://llvm.org/LICENSE.txt for license information.
   9: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
```

- **L1**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L2**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L3**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L4**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L5**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L6**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L7**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L8**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L9**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L10**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L11**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L12**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 13-36 / 第 13-36 行

```cpp
  13: #ifndef KMP_FTN_OS_H
  14: #define KMP_FTN_OS_H
  15: 
  16: // KMP_FNT_ENTRIES may be one of: KMP_FTN_PLAIN, KMP_FTN_UPPER, KMP_FTN_APPEND,
  17: // KMP_FTN_UAPPEND.
  18: 
  19: /* -------------------------- External definitions ------------------------ */
  20: 
  21: #if KMP_FTN_ENTRIES == KMP_FTN_PLAIN
  22: 
  23: #define FTN_SET_STACKSIZE kmp_set_stacksize
  24: #define FTN_SET_STACKSIZE_S kmp_set_stacksize_s
  25: #define FTN_GET_STACKSIZE kmp_get_stacksize
  26: #define FTN_GET_STACKSIZE_S kmp_get_stacksize_s
  27: #define FTN_SET_BLOCKTIME kmp_set_blocktime
  28: #define FTN_GET_BLOCKTIME kmp_get_blocktime
  29: #define FTN_SET_LIBRARY_SERIAL kmp_set_library_serial
  30: #define FTN_SET_LIBRARY_TURNAROUND kmp_set_library_turnaround
  31: #define FTN_SET_LIBRARY_THROUGHPUT kmp_set_library_throughput
  32: #define FTN_SET_LIBRARY kmp_set_library
  33: #define FTN_GET_LIBRARY kmp_get_library
  34: #define FTN_SET_DEFAULTS kmp_set_defaults
  35: #define FTN_SET_DISP_NUM_BUFFERS kmp_set_disp_num_buffers
  36: #define FTN_SET_AFFINITY kmp_set_affinity
```

- **L13**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。
- **L14**: Defines macro \`KMP_FTN_OS_H\` for conditional compilation or textual reuse. / 定义宏 \`KMP_FTN_OS_H\`，供条件编译或文本复用使用。
- **L15**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L16**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L17**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L18**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L19**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L20**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L21**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。
- **L22**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L23**: Defines macro \`FTN_SET_STACKSIZE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_STACKSIZE\`，供条件编译或文本复用使用。
- **L24**: Defines macro \`FTN_SET_STACKSIZE_S\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_STACKSIZE_S\`，供条件编译或文本复用使用。
- **L25**: Defines macro \`FTN_GET_STACKSIZE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_STACKSIZE\`，供条件编译或文本复用使用。
- **L26**: Defines macro \`FTN_GET_STACKSIZE_S\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_STACKSIZE_S\`，供条件编译或文本复用使用。
- **L27**: Defines macro \`FTN_SET_BLOCKTIME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_BLOCKTIME\`，供条件编译或文本复用使用。
- **L28**: Defines macro \`FTN_GET_BLOCKTIME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_BLOCKTIME\`，供条件编译或文本复用使用。
- **L29**: Defines macro \`FTN_SET_LIBRARY_SERIAL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY_SERIAL\`，供条件编译或文本复用使用。
- **L30**: Defines macro \`FTN_SET_LIBRARY_TURNAROUND\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY_TURNAROUND\`，供条件编译或文本复用使用。
- **L31**: Defines macro \`FTN_SET_LIBRARY_THROUGHPUT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY_THROUGHPUT\`，供条件编译或文本复用使用。
- **L32**: Defines macro \`FTN_SET_LIBRARY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY\`，供条件编译或文本复用使用。
- **L33**: Defines macro \`FTN_GET_LIBRARY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_LIBRARY\`，供条件编译或文本复用使用。
- **L34**: Defines macro \`FTN_SET_DEFAULTS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DEFAULTS\`，供条件编译或文本复用使用。
- **L35**: Defines macro \`FTN_SET_DISP_NUM_BUFFERS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DISP_NUM_BUFFERS\`，供条件编译或文本复用使用。
- **L36**: Defines macro \`FTN_SET_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_AFFINITY\`，供条件编译或文本复用使用。

### Lines 37-50 / 第 37-50 行

```cpp
  37: #define FTN_GET_AFFINITY kmp_get_affinity
  38: #define FTN_GET_AFFINITY_MAX_PROC kmp_get_affinity_max_proc
  39: #define FTN_CREATE_AFFINITY_MASK kmp_create_affinity_mask
  40: #define FTN_DESTROY_AFFINITY_MASK kmp_destroy_affinity_mask
  41: #define FTN_SET_AFFINITY_MASK_PROC kmp_set_affinity_mask_proc
  42: #define FTN_UNSET_AFFINITY_MASK_PROC kmp_unset_affinity_mask_proc
  43: #define FTN_GET_AFFINITY_MASK_PROC kmp_get_affinity_mask_proc
  44: 
  45: #define FTN_MALLOC kmp_malloc
  46: #define FTN_ALIGNED_MALLOC kmp_aligned_malloc
  47: #define FTN_CALLOC kmp_calloc
  48: #define FTN_REALLOC kmp_realloc
  49: #define FTN_KFREE kmp_free
  50: 
```

- **L37**: Defines macro \`FTN_GET_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY\`，供条件编译或文本复用使用。
- **L38**: Defines macro \`FTN_GET_AFFINITY_MAX_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY_MAX_PROC\`，供条件编译或文本复用使用。
- **L39**: Defines macro \`FTN_CREATE_AFFINITY_MASK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CREATE_AFFINITY_MASK\`，供条件编译或文本复用使用。
- **L40**: Defines macro \`FTN_DESTROY_AFFINITY_MASK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_AFFINITY_MASK\`，供条件编译或文本复用使用。
- **L41**: Defines macro \`FTN_SET_AFFINITY_MASK_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_AFFINITY_MASK_PROC\`，供条件编译或文本复用使用。
- **L42**: Defines macro \`FTN_UNSET_AFFINITY_MASK_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_UNSET_AFFINITY_MASK_PROC\`，供条件编译或文本复用使用。
- **L43**: Defines macro \`FTN_GET_AFFINITY_MASK_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY_MASK_PROC\`，供条件编译或文本复用使用。
- **L44**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L45**: Defines macro \`FTN_MALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_MALLOC\`，供条件编译或文本复用使用。
- **L46**: Defines macro \`FTN_ALIGNED_MALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_ALIGNED_MALLOC\`，供条件编译或文本复用使用。
- **L47**: Defines macro \`FTN_CALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CALLOC\`，供条件编译或文本复用使用。
- **L48**: Defines macro \`FTN_REALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_REALLOC\`，供条件编译或文本复用使用。
- **L49**: Defines macro \`FTN_KFREE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_KFREE\`，供条件编译或文本复用使用。
- **L50**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 51-74 / 第 51-74 行

```cpp
  51: #define FTN_GET_NUM_KNOWN_THREADS kmp_get_num_known_threads
  52: 
  53: #define FTN_SET_NUM_THREADS omp_set_num_threads
  54: #define FTN_GET_NUM_THREADS omp_get_num_threads
  55: #define FTN_GET_MAX_THREADS omp_get_max_threads
  56: #define FTN_GET_THREAD_NUM omp_get_thread_num
  57: #define FTN_GET_NUM_PROCS omp_get_num_procs
  58: #define FTN_SET_DYNAMIC omp_set_dynamic
  59: #define FTN_GET_DYNAMIC omp_get_dynamic
  60: #define FTN_SET_NESTED omp_set_nested
  61: #define FTN_GET_NESTED omp_get_nested
  62: #define FTN_IN_PARALLEL omp_in_parallel
  63: #define FTN_GET_THREAD_LIMIT omp_get_thread_limit
  64: #define FTN_SET_SCHEDULE omp_set_schedule
  65: #define FTN_GET_SCHEDULE omp_get_schedule
  66: #define FTN_SET_MAX_ACTIVE_LEVELS omp_set_max_active_levels
  67: #define FTN_GET_MAX_ACTIVE_LEVELS omp_get_max_active_levels
  68: #define FTN_GET_ACTIVE_LEVEL omp_get_active_level
  69: #define FTN_GET_LEVEL omp_get_level
  70: #define FTN_GET_ANCESTOR_THREAD_NUM omp_get_ancestor_thread_num
  71: #define FTN_GET_TEAM_SIZE omp_get_team_size
  72: #define FTN_IN_FINAL omp_in_final
  73: #define FTN_GET_PROC_BIND omp_get_proc_bind
  74: #define FTN_GET_NUM_TEAMS omp_get_num_teams
```

- **L51**: Defines macro \`FTN_GET_NUM_KNOWN_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_KNOWN_THREADS\`，供条件编译或文本复用使用。
- **L52**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L53**: Defines macro \`FTN_SET_NUM_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NUM_THREADS\`，供条件编译或文本复用使用。
- **L54**: Defines macro \`FTN_GET_NUM_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_THREADS\`，供条件编译或文本复用使用。
- **L55**: Defines macro \`FTN_GET_MAX_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_THREADS\`，供条件编译或文本复用使用。
- **L56**: Defines macro \`FTN_GET_THREAD_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_THREAD_NUM\`，供条件编译或文本复用使用。
- **L57**: Defines macro \`FTN_GET_NUM_PROCS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_PROCS\`，供条件编译或文本复用使用。
- **L58**: Defines macro \`FTN_SET_DYNAMIC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DYNAMIC\`，供条件编译或文本复用使用。
- **L59**: Defines macro \`FTN_GET_DYNAMIC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DYNAMIC\`，供条件编译或文本复用使用。
- **L60**: Defines macro \`FTN_SET_NESTED\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NESTED\`，供条件编译或文本复用使用。
- **L61**: Defines macro \`FTN_GET_NESTED\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NESTED\`，供条件编译或文本复用使用。
- **L62**: Defines macro \`FTN_IN_PARALLEL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IN_PARALLEL\`，供条件编译或文本复用使用。
- **L63**: Defines macro \`FTN_GET_THREAD_LIMIT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_THREAD_LIMIT\`，供条件编译或文本复用使用。
- **L64**: Defines macro \`FTN_SET_SCHEDULE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_SCHEDULE\`，供条件编译或文本复用使用。
- **L65**: Defines macro \`FTN_GET_SCHEDULE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_SCHEDULE\`，供条件编译或文本复用使用。
- **L66**: Defines macro \`FTN_SET_MAX_ACTIVE_LEVELS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_MAX_ACTIVE_LEVELS\`，供条件编译或文本复用使用。
- **L67**: Defines macro \`FTN_GET_MAX_ACTIVE_LEVELS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_ACTIVE_LEVELS\`，供条件编译或文本复用使用。
- **L68**: Defines macro \`FTN_GET_ACTIVE_LEVEL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_ACTIVE_LEVEL\`，供条件编译或文本复用使用。
- **L69**: Defines macro \`FTN_GET_LEVEL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_LEVEL\`，供条件编译或文本复用使用。
- **L70**: Defines macro \`FTN_GET_ANCESTOR_THREAD_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_ANCESTOR_THREAD_NUM\`，供条件编译或文本复用使用。
- **L71**: Defines macro \`FTN_GET_TEAM_SIZE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_TEAM_SIZE\`，供条件编译或文本复用使用。
- **L72**: Defines macro \`FTN_IN_FINAL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IN_FINAL\`，供条件编译或文本复用使用。
- **L73**: Defines macro \`FTN_GET_PROC_BIND\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PROC_BIND\`，供条件编译或文本复用使用。
- **L74**: Defines macro \`FTN_GET_NUM_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_TEAMS\`，供条件编译或文本复用使用。

### Lines 75-90 / 第 75-90 行

```cpp
  75: #define FTN_GET_TEAM_NUM omp_get_team_num
  76: #define FTN_INIT_LOCK omp_init_lock
  77: #if KMP_USE_DYNAMIC_LOCK
  78: #define FTN_INIT_LOCK_WITH_HINT omp_init_lock_with_hint
  79: #define FTN_INIT_NEST_LOCK_WITH_HINT omp_init_nest_lock_with_hint
  80: #endif
  81: #define FTN_DESTROY_LOCK omp_destroy_lock
  82: #define FTN_SET_LOCK omp_set_lock
  83: #define FTN_UNSET_LOCK omp_unset_lock
  84: #define FTN_TEST_LOCK omp_test_lock
  85: #define FTN_INIT_NEST_LOCK omp_init_nest_lock
  86: #define FTN_DESTROY_NEST_LOCK omp_destroy_nest_lock
  87: #define FTN_SET_NEST_LOCK omp_set_nest_lock
  88: #define FTN_UNSET_NEST_LOCK omp_unset_nest_lock
  89: #define FTN_TEST_NEST_LOCK omp_test_nest_lock
  90: 
```

- **L75**: Defines macro \`FTN_GET_TEAM_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_TEAM_NUM\`，供条件编译或文本复用使用。
- **L76**: Defines macro \`FTN_INIT_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_LOCK\`，供条件编译或文本复用使用。
- **L77**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。
- **L78**: Defines macro \`FTN_INIT_LOCK_WITH_HINT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_LOCK_WITH_HINT\`，供条件编译或文本复用使用。
- **L79**: Defines macro \`FTN_INIT_NEST_LOCK_WITH_HINT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_NEST_LOCK_WITH_HINT\`，供条件编译或文本复用使用。
- **L80**: Ends the active conditional-compilation region. / 结束当前条件编译区域。
- **L81**: Defines macro \`FTN_DESTROY_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_LOCK\`，供条件编译或文本复用使用。
- **L82**: Defines macro \`FTN_SET_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LOCK\`，供条件编译或文本复用使用。
- **L83**: Defines macro \`FTN_UNSET_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_UNSET_LOCK\`，供条件编译或文本复用使用。
- **L84**: Defines macro \`FTN_TEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TEST_LOCK\`，供条件编译或文本复用使用。
- **L85**: Defines macro \`FTN_INIT_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_NEST_LOCK\`，供条件编译或文本复用使用。
- **L86**: Defines macro \`FTN_DESTROY_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_NEST_LOCK\`，供条件编译或文本复用使用。
- **L87**: Defines macro \`FTN_SET_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NEST_LOCK\`，供条件编译或文本复用使用。
- **L88**: Defines macro \`FTN_UNSET_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_UNSET_NEST_LOCK\`，供条件编译或文本复用使用。
- **L89**: Defines macro \`FTN_TEST_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TEST_NEST_LOCK\`，供条件编译或文本复用使用。
- **L90**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 91-104 / 第 91-104 行

```cpp
  91: #define FTN_SET_WARNINGS_ON kmp_set_warnings_on
  92: #define FTN_SET_WARNINGS_OFF kmp_set_warnings_off
  93: 
  94: #define FTN_GET_WTIME omp_get_wtime
  95: #define FTN_GET_WTICK omp_get_wtick
  96: 
  97: #define FTN_GET_NUM_DEVICES omp_get_num_devices
  98: #define FTN_GET_DEFAULT_DEVICE omp_get_default_device
  99: #define FTN_SET_DEFAULT_DEVICE omp_set_default_device
 100: #define FTN_IS_INITIAL_DEVICE omp_is_initial_device
 101: 
 102: #define FTN_GET_CANCELLATION omp_get_cancellation
 103: #define FTN_GET_CANCELLATION_STATUS kmp_get_cancellation_status
 104: 
```

- **L91**: Defines macro \`FTN_SET_WARNINGS_ON\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_WARNINGS_ON\`，供条件编译或文本复用使用。
- **L92**: Defines macro \`FTN_SET_WARNINGS_OFF\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_WARNINGS_OFF\`，供条件编译或文本复用使用。
- **L93**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L94**: Defines macro \`FTN_GET_WTIME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_WTIME\`，供条件编译或文本复用使用。
- **L95**: Defines macro \`FTN_GET_WTICK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_WTICK\`，供条件编译或文本复用使用。
- **L96**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L97**: Defines macro \`FTN_GET_NUM_DEVICES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_DEVICES\`，供条件编译或文本复用使用。
- **L98**: Defines macro \`FTN_GET_DEFAULT_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEFAULT_DEVICE\`，供条件编译或文本复用使用。
- **L99**: Defines macro \`FTN_SET_DEFAULT_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DEFAULT_DEVICE\`，供条件编译或文本复用使用。
- **L100**: Defines macro \`FTN_IS_INITIAL_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IS_INITIAL_DEVICE\`，供条件编译或文本复用使用。
- **L101**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L102**: Defines macro \`FTN_GET_CANCELLATION\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_CANCELLATION\`，供条件编译或文本复用使用。
- **L103**: Defines macro \`FTN_GET_CANCELLATION_STATUS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_CANCELLATION_STATUS\`，供条件编译或文本复用使用。
- **L104**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 105-123 / 第 105-123 行

```cpp
 105: #define FTN_GET_MAX_TASK_PRIORITY omp_get_max_task_priority
 106: #define FTN_GET_NUM_PLACES omp_get_num_places
 107: #define FTN_GET_PLACE_NUM_PROCS omp_get_place_num_procs
 108: #define FTN_GET_PLACE_PROC_IDS omp_get_place_proc_ids
 109: #define FTN_GET_PLACE_NUM omp_get_place_num
 110: #define FTN_GET_PARTITION_NUM_PLACES omp_get_partition_num_places
 111: #define FTN_GET_PARTITION_PLACE_NUMS omp_get_partition_place_nums
 112: #define FTN_GET_INITIAL_DEVICE omp_get_initial_device
 113: #ifdef KMP_STUB
 114: #define FTN_TARGET_ALLOC omp_target_alloc
 115: #define FTN_TARGET_FREE omp_target_free
 116: #define FTN_TARGET_IS_PRESENT omp_target_is_present
 117: #define FTN_TARGET_MEMCPY omp_target_memcpy
 118: #define FTN_TARGET_MEMCPY_RECT omp_target_memcpy_rect
 119: #define FTN_TARGET_MEMSET omp_target_memset
 120: #define FTN_TARGET_MEMSET_ASYNC omp_target_memset_async
 121: #define FTN_TARGET_ASSOCIATE_PTR omp_target_associate_ptr
 122: #define FTN_TARGET_DISASSOCIATE_PTR omp_target_disassociate_ptr
 123: #endif
```

- **L105**: Defines macro \`FTN_GET_MAX_TASK_PRIORITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_TASK_PRIORITY\`，供条件编译或文本复用使用。
- **L106**: Defines macro \`FTN_GET_NUM_PLACES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_PLACES\`，供条件编译或文本复用使用。
- **L107**: Defines macro \`FTN_GET_PLACE_NUM_PROCS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PLACE_NUM_PROCS\`，供条件编译或文本复用使用。
- **L108**: Defines macro \`FTN_GET_PLACE_PROC_IDS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PLACE_PROC_IDS\`，供条件编译或文本复用使用。
- **L109**: Defines macro \`FTN_GET_PLACE_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PLACE_NUM\`，供条件编译或文本复用使用。
- **L110**: Defines macro \`FTN_GET_PARTITION_NUM_PLACES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PARTITION_NUM_PLACES\`，供条件编译或文本复用使用。
- **L111**: Defines macro \`FTN_GET_PARTITION_PLACE_NUMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PARTITION_PLACE_NUMS\`，供条件编译或文本复用使用。
- **L112**: Defines macro \`FTN_GET_INITIAL_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INITIAL_DEVICE\`，供条件编译或文本复用使用。
- **L113**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。
- **L114**: Defines macro \`FTN_TARGET_ALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_ALLOC\`，供条件编译或文本复用使用。
- **L115**: Defines macro \`FTN_TARGET_FREE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_FREE\`，供条件编译或文本复用使用。
- **L116**: Defines macro \`FTN_TARGET_IS_PRESENT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_IS_PRESENT\`，供条件编译或文本复用使用。
- **L117**: Defines macro \`FTN_TARGET_MEMCPY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_MEMCPY\`，供条件编译或文本复用使用。
- **L118**: Defines macro \`FTN_TARGET_MEMCPY_RECT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_MEMCPY_RECT\`，供条件编译或文本复用使用。
- **L119**: Defines macro \`FTN_TARGET_MEMSET\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_MEMSET\`，供条件编译或文本复用使用。
- **L120**: Defines macro \`FTN_TARGET_MEMSET_ASYNC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_MEMSET_ASYNC\`，供条件编译或文本复用使用。
- **L121**: Defines macro \`FTN_TARGET_ASSOCIATE_PTR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_ASSOCIATE_PTR\`，供条件编译或文本复用使用。
- **L122**: Defines macro \`FTN_TARGET_DISASSOCIATE_PTR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_DISASSOCIATE_PTR\`，供条件编译或文本复用使用。
- **L123**: Ends the active conditional-compilation region. / 结束当前条件编译区域。

### Lines 124-147 / 第 124-147 行

```cpp
 124: 
 125: #define FTN_CONTROL_TOOL omp_control_tool
 126: #define FTN_INIT_ALLOCATOR omp_init_allocator
 127: #define FTN_DESTROY_ALLOCATOR omp_destroy_allocator
 128: #define FTN_SET_DEFAULT_ALLOCATOR omp_set_default_allocator
 129: #define FTN_GET_DEFAULT_ALLOCATOR omp_get_default_allocator
 130: #define FTN_GET_DEVICES_MEMSPACE omp_get_devices_memspace
 131: #define FTN_GET_DEVICE_MEMSPACE omp_get_device_memspace
 132: #define FTN_GET_DEVICES_AND_HOST_MEMSPACE omp_get_devices_and_host_memspace
 133: #define FTN_GET_DEVICE_AND_HOST_MEMSPACE omp_get_device_and_host_memspace
 134: #define FTN_GET_DEVICES_ALL_MEMSPACE omp_get_devices_all_memspace
 135: #define FTN_GET_DEVICES_ALLOCATOR omp_get_devices_allocator
 136: #define FTN_GET_DEVICE_ALLOCATOR omp_get_device_allocator
 137: #define FTN_GET_DEVICES_AND_HOST_ALLOCATOR omp_get_devices_and_host_allocator
 138: #define FTN_GET_DEVICE_AND_HOST_ALLOCATOR omp_get_device_and_host_allocator
 139: #define FTN_GET_DEVICES_ALL_ALLOCATOR omp_get_devices_all_allocator
 140: #define FTN_GET_MEMSPACE_NUM_RESOURCES omp_get_memspace_num_resources
 141: #define FTN_GET_SUBMEMSPACE omp_get_submemspace
 142: #define FTN_GET_DEVICE_NUM omp_get_device_num
 143: #define FTN_GET_UID_FROM_DEVICE omp_get_uid_from_device
 144: #define FTN_GET_DEVICE_FROM_UID omp_get_device_from_uid
 145: #define FTN_SET_AFFINITY_FORMAT omp_set_affinity_format
 146: #define FTN_GET_AFFINITY_FORMAT omp_get_affinity_format
 147: #define FTN_DISPLAY_AFFINITY omp_display_affinity
```

- **L124**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L125**: Defines macro \`FTN_CONTROL_TOOL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CONTROL_TOOL\`，供条件编译或文本复用使用。
- **L126**: Defines macro \`FTN_INIT_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_ALLOCATOR\`，供条件编译或文本复用使用。
- **L127**: Defines macro \`FTN_DESTROY_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_ALLOCATOR\`，供条件编译或文本复用使用。
- **L128**: Defines macro \`FTN_SET_DEFAULT_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DEFAULT_ALLOCATOR\`，供条件编译或文本复用使用。
- **L129**: Defines macro \`FTN_GET_DEFAULT_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEFAULT_ALLOCATOR\`，供条件编译或文本复用使用。
- **L130**: Defines macro \`FTN_GET_DEVICES_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_MEMSPACE\`，供条件编译或文本复用使用。
- **L131**: Defines macro \`FTN_GET_DEVICE_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_MEMSPACE\`，供条件编译或文本复用使用。
- **L132**: Defines macro \`FTN_GET_DEVICES_AND_HOST_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_AND_HOST_MEMSPACE\`，供条件编译或文本复用使用。
- **L133**: Defines macro \`FTN_GET_DEVICE_AND_HOST_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_AND_HOST_MEMSPACE\`，供条件编译或文本复用使用。
- **L134**: Defines macro \`FTN_GET_DEVICES_ALL_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_ALL_MEMSPACE\`，供条件编译或文本复用使用。
- **L135**: Defines macro \`FTN_GET_DEVICES_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_ALLOCATOR\`，供条件编译或文本复用使用。
- **L136**: Defines macro \`FTN_GET_DEVICE_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_ALLOCATOR\`，供条件编译或文本复用使用。
- **L137**: Defines macro \`FTN_GET_DEVICES_AND_HOST_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_AND_HOST_ALLOCATOR\`，供条件编译或文本复用使用。
- **L138**: Defines macro \`FTN_GET_DEVICE_AND_HOST_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_AND_HOST_ALLOCATOR\`，供条件编译或文本复用使用。
- **L139**: Defines macro \`FTN_GET_DEVICES_ALL_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_ALL_ALLOCATOR\`，供条件编译或文本复用使用。
- **L140**: Defines macro \`FTN_GET_MEMSPACE_NUM_RESOURCES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MEMSPACE_NUM_RESOURCES\`，供条件编译或文本复用使用。
- **L141**: Defines macro \`FTN_GET_SUBMEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_SUBMEMSPACE\`，供条件编译或文本复用使用。
- **L142**: Defines macro \`FTN_GET_DEVICE_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_NUM\`，供条件编译或文本复用使用。
- **L143**: Defines macro \`FTN_GET_UID_FROM_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_UID_FROM_DEVICE\`，供条件编译或文本复用使用。
- **L144**: Defines macro \`FTN_GET_DEVICE_FROM_UID\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_FROM_UID\`，供条件编译或文本复用使用。
- **L145**: Defines macro \`FTN_SET_AFFINITY_FORMAT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_AFFINITY_FORMAT\`，供条件编译或文本复用使用。
- **L146**: Defines macro \`FTN_GET_AFFINITY_FORMAT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY_FORMAT\`，供条件编译或文本复用使用。
- **L147**: Defines macro \`FTN_DISPLAY_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DISPLAY_AFFINITY\`，供条件编译或文本复用使用。

### Lines 148-159 / 第 148-159 行

```cpp
 148: #define FTN_CAPTURE_AFFINITY omp_capture_affinity
 149: #define FTN_PAUSE_RESOURCE omp_pause_resource
 150: #define FTN_PAUSE_RESOURCE_ALL omp_pause_resource_all
 151: #define FTN_GET_SUPPORTED_ACTIVE_LEVELS omp_get_supported_active_levels
 152: #define FTN_DISPLAY_ENV omp_display_env
 153: #define FTN_IN_EXPLICIT_TASK omp_in_explicit_task
 154: #define FTN_FULFILL_EVENT omp_fulfill_event
 155: #define FTN_SET_NUM_TEAMS omp_set_num_teams
 156: #define FTN_GET_MAX_TEAMS omp_get_max_teams
 157: #define FTN_SET_TEAMS_THREAD_LIMIT omp_set_teams_thread_limit
 158: #define FTN_GET_TEAMS_THREAD_LIMIT omp_get_teams_thread_limit
 159: 
```

- **L148**: Defines macro \`FTN_CAPTURE_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CAPTURE_AFFINITY\`，供条件编译或文本复用使用。
- **L149**: Defines macro \`FTN_PAUSE_RESOURCE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_PAUSE_RESOURCE\`，供条件编译或文本复用使用。
- **L150**: Defines macro \`FTN_PAUSE_RESOURCE_ALL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_PAUSE_RESOURCE_ALL\`，供条件编译或文本复用使用。
- **L151**: Defines macro \`FTN_GET_SUPPORTED_ACTIVE_LEVELS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_SUPPORTED_ACTIVE_LEVELS\`，供条件编译或文本复用使用。
- **L152**: Defines macro \`FTN_DISPLAY_ENV\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DISPLAY_ENV\`，供条件编译或文本复用使用。
- **L153**: Defines macro \`FTN_IN_EXPLICIT_TASK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IN_EXPLICIT_TASK\`，供条件编译或文本复用使用。
- **L154**: Defines macro \`FTN_FULFILL_EVENT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_FULFILL_EVENT\`，供条件编译或文本复用使用。
- **L155**: Defines macro \`FTN_SET_NUM_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NUM_TEAMS\`，供条件编译或文本复用使用。
- **L156**: Defines macro \`FTN_GET_MAX_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_TEAMS\`，供条件编译或文本复用使用。
- **L157**: Defines macro \`FTN_SET_TEAMS_THREAD_LIMIT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_TEAMS_THREAD_LIMIT\`，供条件编译或文本复用使用。
- **L158**: Defines macro \`FTN_GET_TEAMS_THREAD_LIMIT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_TEAMS_THREAD_LIMIT\`，供条件编译或文本复用使用。
- **L159**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 160-171 / 第 160-171 行

```cpp
 160: #define FTN_GET_NUM_INTEROP_PROPERTIES omp_get_num_interop_properties
 161: #define FTN_GET_INTEROP_INT omp_get_interop_int
 162: #define FTN_GET_INTEROP_PTR omp_get_interop_ptr
 163: #define FTN_GET_INTEROP_STR omp_get_interop_str
 164: #define FTN_GET_INTEROP_NAME omp_get_interop_name
 165: #define FTN_GET_INTEROP_TYPE_DESC omp_get_interop_type_desc
 166: #define FTN_GET_INTEROP_RC_DESC omp_get_interop_rc_desc
 167: 
 168: #endif /* KMP_FTN_PLAIN */
 169: 
 170: /* ------------------------------------------------------------------------ */
 171: 
```

- **L160**: Defines macro \`FTN_GET_NUM_INTEROP_PROPERTIES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_INTEROP_PROPERTIES\`，供条件编译或文本复用使用。
- **L161**: Defines macro \`FTN_GET_INTEROP_INT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_INT\`，供条件编译或文本复用使用。
- **L162**: Defines macro \`FTN_GET_INTEROP_PTR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_PTR\`，供条件编译或文本复用使用。
- **L163**: Defines macro \`FTN_GET_INTEROP_STR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_STR\`，供条件编译或文本复用使用。
- **L164**: Defines macro \`FTN_GET_INTEROP_NAME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_NAME\`，供条件编译或文本复用使用。
- **L165**: Defines macro \`FTN_GET_INTEROP_TYPE_DESC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_TYPE_DESC\`，供条件编译或文本复用使用。
- **L166**: Defines macro \`FTN_GET_INTEROP_RC_DESC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_RC_DESC\`，供条件编译或文本复用使用。
- **L167**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L168**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L169**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L170**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L171**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 172-195 / 第 172-195 行

```cpp
 172: #if KMP_FTN_ENTRIES == KMP_FTN_APPEND
 173: 
 174: #define FTN_SET_STACKSIZE kmp_set_stacksize_
 175: #define FTN_SET_STACKSIZE_S kmp_set_stacksize_s_
 176: #define FTN_GET_STACKSIZE kmp_get_stacksize_
 177: #define FTN_GET_STACKSIZE_S kmp_get_stacksize_s_
 178: #define FTN_SET_BLOCKTIME kmp_set_blocktime_
 179: #define FTN_GET_BLOCKTIME kmp_get_blocktime_
 180: #define FTN_SET_LIBRARY_SERIAL kmp_set_library_serial_
 181: #define FTN_SET_LIBRARY_TURNAROUND kmp_set_library_turnaround_
 182: #define FTN_SET_LIBRARY_THROUGHPUT kmp_set_library_throughput_
 183: #define FTN_SET_LIBRARY kmp_set_library_
 184: #define FTN_GET_LIBRARY kmp_get_library_
 185: #define FTN_SET_DEFAULTS kmp_set_defaults_
 186: #define FTN_SET_DISP_NUM_BUFFERS kmp_set_disp_num_buffers_
 187: #define FTN_SET_AFFINITY kmp_set_affinity_
 188: #define FTN_GET_AFFINITY kmp_get_affinity_
 189: #define FTN_GET_AFFINITY_MAX_PROC kmp_get_affinity_max_proc_
 190: #define FTN_CREATE_AFFINITY_MASK kmp_create_affinity_mask_
 191: #define FTN_DESTROY_AFFINITY_MASK kmp_destroy_affinity_mask_
 192: #define FTN_SET_AFFINITY_MASK_PROC kmp_set_affinity_mask_proc_
 193: #define FTN_UNSET_AFFINITY_MASK_PROC kmp_unset_affinity_mask_proc_
 194: #define FTN_GET_AFFINITY_MASK_PROC kmp_get_affinity_mask_proc_
 195: 
```

- **L172**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。
- **L173**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L174**: Defines macro \`FTN_SET_STACKSIZE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_STACKSIZE\`，供条件编译或文本复用使用。
- **L175**: Defines macro \`FTN_SET_STACKSIZE_S\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_STACKSIZE_S\`，供条件编译或文本复用使用。
- **L176**: Defines macro \`FTN_GET_STACKSIZE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_STACKSIZE\`，供条件编译或文本复用使用。
- **L177**: Defines macro \`FTN_GET_STACKSIZE_S\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_STACKSIZE_S\`，供条件编译或文本复用使用。
- **L178**: Defines macro \`FTN_SET_BLOCKTIME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_BLOCKTIME\`，供条件编译或文本复用使用。
- **L179**: Defines macro \`FTN_GET_BLOCKTIME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_BLOCKTIME\`，供条件编译或文本复用使用。
- **L180**: Defines macro \`FTN_SET_LIBRARY_SERIAL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY_SERIAL\`，供条件编译或文本复用使用。
- **L181**: Defines macro \`FTN_SET_LIBRARY_TURNAROUND\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY_TURNAROUND\`，供条件编译或文本复用使用。
- **L182**: Defines macro \`FTN_SET_LIBRARY_THROUGHPUT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY_THROUGHPUT\`，供条件编译或文本复用使用。
- **L183**: Defines macro \`FTN_SET_LIBRARY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY\`，供条件编译或文本复用使用。
- **L184**: Defines macro \`FTN_GET_LIBRARY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_LIBRARY\`，供条件编译或文本复用使用。
- **L185**: Defines macro \`FTN_SET_DEFAULTS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DEFAULTS\`，供条件编译或文本复用使用。
- **L186**: Defines macro \`FTN_SET_DISP_NUM_BUFFERS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DISP_NUM_BUFFERS\`，供条件编译或文本复用使用。
- **L187**: Defines macro \`FTN_SET_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_AFFINITY\`，供条件编译或文本复用使用。
- **L188**: Defines macro \`FTN_GET_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY\`，供条件编译或文本复用使用。
- **L189**: Defines macro \`FTN_GET_AFFINITY_MAX_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY_MAX_PROC\`，供条件编译或文本复用使用。
- **L190**: Defines macro \`FTN_CREATE_AFFINITY_MASK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CREATE_AFFINITY_MASK\`，供条件编译或文本复用使用。
- **L191**: Defines macro \`FTN_DESTROY_AFFINITY_MASK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_AFFINITY_MASK\`，供条件编译或文本复用使用。
- **L192**: Defines macro \`FTN_SET_AFFINITY_MASK_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_AFFINITY_MASK_PROC\`，供条件编译或文本复用使用。
- **L193**: Defines macro \`FTN_UNSET_AFFINITY_MASK_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_UNSET_AFFINITY_MASK_PROC\`，供条件编译或文本复用使用。
- **L194**: Defines macro \`FTN_GET_AFFINITY_MASK_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY_MASK_PROC\`，供条件编译或文本复用使用。
- **L195**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 196-219 / 第 196-219 行

```cpp
 196: #define FTN_MALLOC kmp_malloc_
 197: #define FTN_ALIGNED_MALLOC kmp_aligned_malloc_
 198: #define FTN_CALLOC kmp_calloc_
 199: #define FTN_REALLOC kmp_realloc_
 200: #define FTN_KFREE kmp_free_
 201: 
 202: #define FTN_GET_NUM_KNOWN_THREADS kmp_get_num_known_threads_
 203: 
 204: #define FTN_SET_NUM_THREADS omp_set_num_threads_
 205: #define FTN_GET_NUM_THREADS omp_get_num_threads_
 206: #define FTN_GET_MAX_THREADS omp_get_max_threads_
 207: #define FTN_GET_THREAD_NUM omp_get_thread_num_
 208: #define FTN_GET_NUM_PROCS omp_get_num_procs_
 209: #define FTN_SET_DYNAMIC omp_set_dynamic_
 210: #define FTN_GET_DYNAMIC omp_get_dynamic_
 211: #define FTN_SET_NESTED omp_set_nested_
 212: #define FTN_GET_NESTED omp_get_nested_
 213: #define FTN_IN_PARALLEL omp_in_parallel_
 214: #define FTN_GET_THREAD_LIMIT omp_get_thread_limit_
 215: #define FTN_SET_SCHEDULE omp_set_schedule_
 216: #define FTN_GET_SCHEDULE omp_get_schedule_
 217: #define FTN_SET_MAX_ACTIVE_LEVELS omp_set_max_active_levels_
 218: #define FTN_GET_MAX_ACTIVE_LEVELS omp_get_max_active_levels_
 219: #define FTN_GET_ACTIVE_LEVEL omp_get_active_level_
```

- **L196**: Defines macro \`FTN_MALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_MALLOC\`，供条件编译或文本复用使用。
- **L197**: Defines macro \`FTN_ALIGNED_MALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_ALIGNED_MALLOC\`，供条件编译或文本复用使用。
- **L198**: Defines macro \`FTN_CALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CALLOC\`，供条件编译或文本复用使用。
- **L199**: Defines macro \`FTN_REALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_REALLOC\`，供条件编译或文本复用使用。
- **L200**: Defines macro \`FTN_KFREE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_KFREE\`，供条件编译或文本复用使用。
- **L201**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L202**: Defines macro \`FTN_GET_NUM_KNOWN_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_KNOWN_THREADS\`，供条件编译或文本复用使用。
- **L203**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L204**: Defines macro \`FTN_SET_NUM_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NUM_THREADS\`，供条件编译或文本复用使用。
- **L205**: Defines macro \`FTN_GET_NUM_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_THREADS\`，供条件编译或文本复用使用。
- **L206**: Defines macro \`FTN_GET_MAX_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_THREADS\`，供条件编译或文本复用使用。
- **L207**: Defines macro \`FTN_GET_THREAD_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_THREAD_NUM\`，供条件编译或文本复用使用。
- **L208**: Defines macro \`FTN_GET_NUM_PROCS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_PROCS\`，供条件编译或文本复用使用。
- **L209**: Defines macro \`FTN_SET_DYNAMIC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DYNAMIC\`，供条件编译或文本复用使用。
- **L210**: Defines macro \`FTN_GET_DYNAMIC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DYNAMIC\`，供条件编译或文本复用使用。
- **L211**: Defines macro \`FTN_SET_NESTED\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NESTED\`，供条件编译或文本复用使用。
- **L212**: Defines macro \`FTN_GET_NESTED\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NESTED\`，供条件编译或文本复用使用。
- **L213**: Defines macro \`FTN_IN_PARALLEL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IN_PARALLEL\`，供条件编译或文本复用使用。
- **L214**: Defines macro \`FTN_GET_THREAD_LIMIT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_THREAD_LIMIT\`，供条件编译或文本复用使用。
- **L215**: Defines macro \`FTN_SET_SCHEDULE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_SCHEDULE\`，供条件编译或文本复用使用。
- **L216**: Defines macro \`FTN_GET_SCHEDULE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_SCHEDULE\`，供条件编译或文本复用使用。
- **L217**: Defines macro \`FTN_SET_MAX_ACTIVE_LEVELS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_MAX_ACTIVE_LEVELS\`，供条件编译或文本复用使用。
- **L218**: Defines macro \`FTN_GET_MAX_ACTIVE_LEVELS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_ACTIVE_LEVELS\`，供条件编译或文本复用使用。
- **L219**: Defines macro \`FTN_GET_ACTIVE_LEVEL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_ACTIVE_LEVEL\`，供条件编译或文本复用使用。

### Lines 220-231 / 第 220-231 行

```cpp
 220: #define FTN_GET_LEVEL omp_get_level_
 221: #define FTN_GET_ANCESTOR_THREAD_NUM omp_get_ancestor_thread_num_
 222: #define FTN_GET_TEAM_SIZE omp_get_team_size_
 223: #define FTN_IN_FINAL omp_in_final_
 224: #define FTN_GET_PROC_BIND omp_get_proc_bind_
 225: #define FTN_GET_NUM_TEAMS omp_get_num_teams_
 226: #define FTN_GET_TEAM_NUM omp_get_team_num_
 227: #define FTN_INIT_LOCK omp_init_lock_
 228: #if KMP_USE_DYNAMIC_LOCK
 229: #define FTN_INIT_LOCK_WITH_HINT omp_init_lock_with_hint_
 230: #define FTN_INIT_NEST_LOCK_WITH_HINT omp_init_nest_lock_with_hint_
 231: #endif
```

- **L220**: Defines macro \`FTN_GET_LEVEL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_LEVEL\`，供条件编译或文本复用使用。
- **L221**: Defines macro \`FTN_GET_ANCESTOR_THREAD_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_ANCESTOR_THREAD_NUM\`，供条件编译或文本复用使用。
- **L222**: Defines macro \`FTN_GET_TEAM_SIZE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_TEAM_SIZE\`，供条件编译或文本复用使用。
- **L223**: Defines macro \`FTN_IN_FINAL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IN_FINAL\`，供条件编译或文本复用使用。
- **L224**: Defines macro \`FTN_GET_PROC_BIND\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PROC_BIND\`，供条件编译或文本复用使用。
- **L225**: Defines macro \`FTN_GET_NUM_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_TEAMS\`，供条件编译或文本复用使用。
- **L226**: Defines macro \`FTN_GET_TEAM_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_TEAM_NUM\`，供条件编译或文本复用使用。
- **L227**: Defines macro \`FTN_INIT_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_LOCK\`，供条件编译或文本复用使用。
- **L228**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。
- **L229**: Defines macro \`FTN_INIT_LOCK_WITH_HINT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_LOCK_WITH_HINT\`，供条件编译或文本复用使用。
- **L230**: Defines macro \`FTN_INIT_NEST_LOCK_WITH_HINT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_NEST_LOCK_WITH_HINT\`，供条件编译或文本复用使用。
- **L231**: Ends the active conditional-compilation region. / 结束当前条件编译区域。

### Lines 232-244 / 第 232-244 行

```cpp
 232: #define FTN_DESTROY_LOCK omp_destroy_lock_
 233: #define FTN_SET_LOCK omp_set_lock_
 234: #define FTN_UNSET_LOCK omp_unset_lock_
 235: #define FTN_TEST_LOCK omp_test_lock_
 236: #define FTN_INIT_NEST_LOCK omp_init_nest_lock_
 237: #define FTN_DESTROY_NEST_LOCK omp_destroy_nest_lock_
 238: #define FTN_SET_NEST_LOCK omp_set_nest_lock_
 239: #define FTN_UNSET_NEST_LOCK omp_unset_nest_lock_
 240: #define FTN_TEST_NEST_LOCK omp_test_nest_lock_
 241: 
 242: #define FTN_SET_WARNINGS_ON kmp_set_warnings_on_
 243: #define FTN_SET_WARNINGS_OFF kmp_set_warnings_off_
 244: 
```

- **L232**: Defines macro \`FTN_DESTROY_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_LOCK\`，供条件编译或文本复用使用。
- **L233**: Defines macro \`FTN_SET_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LOCK\`，供条件编译或文本复用使用。
- **L234**: Defines macro \`FTN_UNSET_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_UNSET_LOCK\`，供条件编译或文本复用使用。
- **L235**: Defines macro \`FTN_TEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TEST_LOCK\`，供条件编译或文本复用使用。
- **L236**: Defines macro \`FTN_INIT_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_NEST_LOCK\`，供条件编译或文本复用使用。
- **L237**: Defines macro \`FTN_DESTROY_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_NEST_LOCK\`，供条件编译或文本复用使用。
- **L238**: Defines macro \`FTN_SET_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NEST_LOCK\`，供条件编译或文本复用使用。
- **L239**: Defines macro \`FTN_UNSET_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_UNSET_NEST_LOCK\`，供条件编译或文本复用使用。
- **L240**: Defines macro \`FTN_TEST_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TEST_NEST_LOCK\`，供条件编译或文本复用使用。
- **L241**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L242**: Defines macro \`FTN_SET_WARNINGS_ON\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_WARNINGS_ON\`，供条件编译或文本复用使用。
- **L243**: Defines macro \`FTN_SET_WARNINGS_OFF\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_WARNINGS_OFF\`，供条件编译或文本复用使用。
- **L244**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 245-264 / 第 245-264 行

```cpp
 245: #define FTN_GET_WTIME omp_get_wtime_
 246: #define FTN_GET_WTICK omp_get_wtick_
 247: 
 248: #define FTN_GET_NUM_DEVICES omp_get_num_devices_
 249: #define FTN_GET_DEFAULT_DEVICE omp_get_default_device_
 250: #define FTN_SET_DEFAULT_DEVICE omp_set_default_device_
 251: #define FTN_IS_INITIAL_DEVICE omp_is_initial_device_
 252: 
 253: #define FTN_GET_CANCELLATION omp_get_cancellation_
 254: #define FTN_GET_CANCELLATION_STATUS kmp_get_cancellation_status_
 255: 
 256: #define FTN_GET_MAX_TASK_PRIORITY omp_get_max_task_priority_
 257: #define FTN_GET_NUM_PLACES omp_get_num_places_
 258: #define FTN_GET_PLACE_NUM_PROCS omp_get_place_num_procs_
 259: #define FTN_GET_PLACE_PROC_IDS omp_get_place_proc_ids_
 260: #define FTN_GET_PLACE_NUM omp_get_place_num_
 261: #define FTN_GET_PARTITION_NUM_PLACES omp_get_partition_num_places_
 262: #define FTN_GET_PARTITION_PLACE_NUMS omp_get_partition_place_nums_
 263: #define FTN_GET_INITIAL_DEVICE omp_get_initial_device_
 264: #ifdef KMP_STUB
```

- **L245**: Defines macro \`FTN_GET_WTIME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_WTIME\`，供条件编译或文本复用使用。
- **L246**: Defines macro \`FTN_GET_WTICK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_WTICK\`，供条件编译或文本复用使用。
- **L247**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L248**: Defines macro \`FTN_GET_NUM_DEVICES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_DEVICES\`，供条件编译或文本复用使用。
- **L249**: Defines macro \`FTN_GET_DEFAULT_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEFAULT_DEVICE\`，供条件编译或文本复用使用。
- **L250**: Defines macro \`FTN_SET_DEFAULT_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DEFAULT_DEVICE\`，供条件编译或文本复用使用。
- **L251**: Defines macro \`FTN_IS_INITIAL_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IS_INITIAL_DEVICE\`，供条件编译或文本复用使用。
- **L252**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L253**: Defines macro \`FTN_GET_CANCELLATION\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_CANCELLATION\`，供条件编译或文本复用使用。
- **L254**: Defines macro \`FTN_GET_CANCELLATION_STATUS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_CANCELLATION_STATUS\`，供条件编译或文本复用使用。
- **L255**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L256**: Defines macro \`FTN_GET_MAX_TASK_PRIORITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_TASK_PRIORITY\`，供条件编译或文本复用使用。
- **L257**: Defines macro \`FTN_GET_NUM_PLACES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_PLACES\`，供条件编译或文本复用使用。
- **L258**: Defines macro \`FTN_GET_PLACE_NUM_PROCS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PLACE_NUM_PROCS\`，供条件编译或文本复用使用。
- **L259**: Defines macro \`FTN_GET_PLACE_PROC_IDS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PLACE_PROC_IDS\`，供条件编译或文本复用使用。
- **L260**: Defines macro \`FTN_GET_PLACE_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PLACE_NUM\`，供条件编译或文本复用使用。
- **L261**: Defines macro \`FTN_GET_PARTITION_NUM_PLACES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PARTITION_NUM_PLACES\`，供条件编译或文本复用使用。
- **L262**: Defines macro \`FTN_GET_PARTITION_PLACE_NUMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PARTITION_PLACE_NUMS\`，供条件编译或文本复用使用。
- **L263**: Defines macro \`FTN_GET_INITIAL_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INITIAL_DEVICE\`，供条件编译或文本复用使用。
- **L264**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。

### Lines 265-288 / 第 265-288 行

```cpp
 265: #define FTN_TARGET_ALLOC omp_target_alloc_
 266: #define FTN_TARGET_FREE omp_target_free_
 267: #define FTN_TARGET_IS_PRESENT omp_target_is_present_
 268: #define FTN_TARGET_MEMCPY omp_target_memcpy_
 269: #define FTN_TARGET_MEMCPY_RECT omp_target_memcpy_rect_
 270: #define FTN_TARGET_ASSOCIATE_PTR omp_target_associate_ptr_
 271: #define FTN_TARGET_DISASSOCIATE_PTR omp_target_disassociate_ptr_
 272: #endif
 273: 
 274: #define FTN_CONTROL_TOOL omp_control_tool_
 275: #define FTN_INIT_ALLOCATOR omp_init_allocator_
 276: #define FTN_DESTROY_ALLOCATOR omp_destroy_allocator_
 277: #define FTN_SET_DEFAULT_ALLOCATOR omp_set_default_allocator_
 278: #define FTN_GET_DEFAULT_ALLOCATOR omp_get_default_allocator_
 279: #define FTN_GET_DEVICES_MEMSPACE omp_get_devices_memspace_
 280: #define FTN_GET_DEVICE_MEMSPACE omp_get_device_memspace_
 281: #define FTN_GET_DEVICES_AND_HOST_MEMSPACE omp_get_devices_and_host_memspace_
 282: #define FTN_GET_DEVICE_AND_HOST_MEMSPACE omp_get_device_and_host_memspace_
 283: #define FTN_GET_DEVICES_ALL_MEMSPACE omp_get_devices_all_memspace_
 284: #define FTN_GET_DEVICES_ALLOCATOR omp_get_devices_allocator_
 285: #define FTN_GET_DEVICE_ALLOCATOR omp_get_device_allocator_
 286: #define FTN_GET_DEVICES_AND_HOST_ALLOCATOR omp_get_devices_and_host_allocator_
 287: #define FTN_GET_DEVICE_AND_HOST_ALLOCATOR omp_get_device_and_host_allocator_
 288: #define FTN_GET_DEVICES_ALL_ALLOCATOR omp_get_devices_all_allocator_
```

- **L265**: Defines macro \`FTN_TARGET_ALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_ALLOC\`，供条件编译或文本复用使用。
- **L266**: Defines macro \`FTN_TARGET_FREE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_FREE\`，供条件编译或文本复用使用。
- **L267**: Defines macro \`FTN_TARGET_IS_PRESENT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_IS_PRESENT\`，供条件编译或文本复用使用。
- **L268**: Defines macro \`FTN_TARGET_MEMCPY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_MEMCPY\`，供条件编译或文本复用使用。
- **L269**: Defines macro \`FTN_TARGET_MEMCPY_RECT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_MEMCPY_RECT\`，供条件编译或文本复用使用。
- **L270**: Defines macro \`FTN_TARGET_ASSOCIATE_PTR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_ASSOCIATE_PTR\`，供条件编译或文本复用使用。
- **L271**: Defines macro \`FTN_TARGET_DISASSOCIATE_PTR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_DISASSOCIATE_PTR\`，供条件编译或文本复用使用。
- **L272**: Ends the active conditional-compilation region. / 结束当前条件编译区域。
- **L273**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L274**: Defines macro \`FTN_CONTROL_TOOL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CONTROL_TOOL\`，供条件编译或文本复用使用。
- **L275**: Defines macro \`FTN_INIT_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_ALLOCATOR\`，供条件编译或文本复用使用。
- **L276**: Defines macro \`FTN_DESTROY_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_ALLOCATOR\`，供条件编译或文本复用使用。
- **L277**: Defines macro \`FTN_SET_DEFAULT_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DEFAULT_ALLOCATOR\`，供条件编译或文本复用使用。
- **L278**: Defines macro \`FTN_GET_DEFAULT_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEFAULT_ALLOCATOR\`，供条件编译或文本复用使用。
- **L279**: Defines macro \`FTN_GET_DEVICES_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_MEMSPACE\`，供条件编译或文本复用使用。
- **L280**: Defines macro \`FTN_GET_DEVICE_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_MEMSPACE\`，供条件编译或文本复用使用。
- **L281**: Defines macro \`FTN_GET_DEVICES_AND_HOST_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_AND_HOST_MEMSPACE\`，供条件编译或文本复用使用。
- **L282**: Defines macro \`FTN_GET_DEVICE_AND_HOST_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_AND_HOST_MEMSPACE\`，供条件编译或文本复用使用。
- **L283**: Defines macro \`FTN_GET_DEVICES_ALL_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_ALL_MEMSPACE\`，供条件编译或文本复用使用。
- **L284**: Defines macro \`FTN_GET_DEVICES_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_ALLOCATOR\`，供条件编译或文本复用使用。
- **L285**: Defines macro \`FTN_GET_DEVICE_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_ALLOCATOR\`，供条件编译或文本复用使用。
- **L286**: Defines macro \`FTN_GET_DEVICES_AND_HOST_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_AND_HOST_ALLOCATOR\`，供条件编译或文本复用使用。
- **L287**: Defines macro \`FTN_GET_DEVICE_AND_HOST_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_AND_HOST_ALLOCATOR\`，供条件编译或文本复用使用。
- **L288**: Defines macro \`FTN_GET_DEVICES_ALL_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_ALL_ALLOCATOR\`，供条件编译或文本复用使用。

### Lines 289-310 / 第 289-310 行

```cpp
 289: #define FTN_GET_MEMSPACE_NUM_RESOURCES omp_get_memspace_num_resources_
 290: #define FTN_GET_SUBMEMSPACE omp_get_submemspace_
 291: #define FTN_ALLOC omp_alloc_
 292: #define FTN_FREE omp_free_
 293: #define FTN_GET_DEVICE_NUM omp_get_device_num_
 294: #define FTN_GET_UID_FROM_DEVICE omp_get_uid_from_device_
 295: #define FTN_GET_DEVICE_FROM_UID omp_get_device_from_uid_
 296: #define FTN_SET_AFFINITY_FORMAT omp_set_affinity_format_
 297: #define FTN_GET_AFFINITY_FORMAT omp_get_affinity_format_
 298: #define FTN_DISPLAY_AFFINITY omp_display_affinity_
 299: #define FTN_CAPTURE_AFFINITY omp_capture_affinity_
 300: #define FTN_PAUSE_RESOURCE omp_pause_resource_
 301: #define FTN_PAUSE_RESOURCE_ALL omp_pause_resource_all_
 302: #define FTN_GET_SUPPORTED_ACTIVE_LEVELS omp_get_supported_active_levels_
 303: #define FTN_DISPLAY_ENV omp_display_env_
 304: #define FTN_IN_EXPLICIT_TASK omp_in_explicit_task_
 305: #define FTN_FULFILL_EVENT omp_fulfill_event_
 306: #define FTN_SET_NUM_TEAMS omp_set_num_teams_
 307: #define FTN_GET_MAX_TEAMS omp_get_max_teams_
 308: #define FTN_SET_TEAMS_THREAD_LIMIT omp_set_teams_thread_limit_
 309: #define FTN_GET_TEAMS_THREAD_LIMIT omp_get_teams_thread_limit_
 310: 
```

- **L289**: Defines macro \`FTN_GET_MEMSPACE_NUM_RESOURCES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MEMSPACE_NUM_RESOURCES\`，供条件编译或文本复用使用。
- **L290**: Defines macro \`FTN_GET_SUBMEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_SUBMEMSPACE\`，供条件编译或文本复用使用。
- **L291**: Defines macro \`FTN_ALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_ALLOC\`，供条件编译或文本复用使用。
- **L292**: Defines macro \`FTN_FREE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_FREE\`，供条件编译或文本复用使用。
- **L293**: Defines macro \`FTN_GET_DEVICE_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_NUM\`，供条件编译或文本复用使用。
- **L294**: Defines macro \`FTN_GET_UID_FROM_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_UID_FROM_DEVICE\`，供条件编译或文本复用使用。
- **L295**: Defines macro \`FTN_GET_DEVICE_FROM_UID\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_FROM_UID\`，供条件编译或文本复用使用。
- **L296**: Defines macro \`FTN_SET_AFFINITY_FORMAT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_AFFINITY_FORMAT\`，供条件编译或文本复用使用。
- **L297**: Defines macro \`FTN_GET_AFFINITY_FORMAT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY_FORMAT\`，供条件编译或文本复用使用。
- **L298**: Defines macro \`FTN_DISPLAY_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DISPLAY_AFFINITY\`，供条件编译或文本复用使用。
- **L299**: Defines macro \`FTN_CAPTURE_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CAPTURE_AFFINITY\`，供条件编译或文本复用使用。
- **L300**: Defines macro \`FTN_PAUSE_RESOURCE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_PAUSE_RESOURCE\`，供条件编译或文本复用使用。
- **L301**: Defines macro \`FTN_PAUSE_RESOURCE_ALL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_PAUSE_RESOURCE_ALL\`，供条件编译或文本复用使用。
- **L302**: Defines macro \`FTN_GET_SUPPORTED_ACTIVE_LEVELS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_SUPPORTED_ACTIVE_LEVELS\`，供条件编译或文本复用使用。
- **L303**: Defines macro \`FTN_DISPLAY_ENV\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DISPLAY_ENV\`，供条件编译或文本复用使用。
- **L304**: Defines macro \`FTN_IN_EXPLICIT_TASK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IN_EXPLICIT_TASK\`，供条件编译或文本复用使用。
- **L305**: Defines macro \`FTN_FULFILL_EVENT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_FULFILL_EVENT\`，供条件编译或文本复用使用。
- **L306**: Defines macro \`FTN_SET_NUM_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NUM_TEAMS\`，供条件编译或文本复用使用。
- **L307**: Defines macro \`FTN_GET_MAX_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_TEAMS\`，供条件编译或文本复用使用。
- **L308**: Defines macro \`FTN_SET_TEAMS_THREAD_LIMIT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_TEAMS_THREAD_LIMIT\`，供条件编译或文本复用使用。
- **L309**: Defines macro \`FTN_GET_TEAMS_THREAD_LIMIT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_TEAMS_THREAD_LIMIT\`，供条件编译或文本复用使用。
- **L310**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 311-322 / 第 311-322 行

```cpp
 311: #define FTN_GET_NUM_INTEROP_PROPERTIES omp_get_num_interop_properties_
 312: #define FTN_GET_INTEROP_INT omp_get_interop_int_
 313: #define FTN_GET_INTEROP_PTR omp_get_interop_ptr_
 314: #define FTN_GET_INTEROP_STR omp_get_interop_str_
 315: #define FTN_GET_INTEROP_NAME omp_get_interop_name_
 316: #define FTN_GET_INTEROP_TYPE_DESC omp_get_interop_type_desc_
 317: #define FTN_GET_INTEROP_RC_DESC omp_get_interop_rc_desc_
 318: 
 319: #endif /* KMP_FTN_APPEND */
 320: 
 321: /* ------------------------------------------------------------------------ */
 322: 
```

- **L311**: Defines macro \`FTN_GET_NUM_INTEROP_PROPERTIES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_INTEROP_PROPERTIES\`，供条件编译或文本复用使用。
- **L312**: Defines macro \`FTN_GET_INTEROP_INT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_INT\`，供条件编译或文本复用使用。
- **L313**: Defines macro \`FTN_GET_INTEROP_PTR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_PTR\`，供条件编译或文本复用使用。
- **L314**: Defines macro \`FTN_GET_INTEROP_STR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_STR\`，供条件编译或文本复用使用。
- **L315**: Defines macro \`FTN_GET_INTEROP_NAME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_NAME\`，供条件编译或文本复用使用。
- **L316**: Defines macro \`FTN_GET_INTEROP_TYPE_DESC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_TYPE_DESC\`，供条件编译或文本复用使用。
- **L317**: Defines macro \`FTN_GET_INTEROP_RC_DESC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_RC_DESC\`，供条件编译或文本复用使用。
- **L318**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L319**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L320**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L321**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L322**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 323-346 / 第 323-346 行

```cpp
 323: #if KMP_FTN_ENTRIES == KMP_FTN_UPPER
 324: 
 325: #define FTN_SET_STACKSIZE KMP_SET_STACKSIZE
 326: #define FTN_SET_STACKSIZE_S KMP_SET_STACKSIZE_S
 327: #define FTN_GET_STACKSIZE KMP_GET_STACKSIZE
 328: #define FTN_GET_STACKSIZE_S KMP_GET_STACKSIZE_S
 329: #define FTN_SET_BLOCKTIME KMP_SET_BLOCKTIME
 330: #define FTN_GET_BLOCKTIME KMP_GET_BLOCKTIME
 331: #define FTN_SET_LIBRARY_SERIAL KMP_SET_LIBRARY_SERIAL
 332: #define FTN_SET_LIBRARY_TURNAROUND KMP_SET_LIBRARY_TURNAROUND
 333: #define FTN_SET_LIBRARY_THROUGHPUT KMP_SET_LIBRARY_THROUGHPUT
 334: #define FTN_SET_LIBRARY KMP_SET_LIBRARY
 335: #define FTN_GET_LIBRARY KMP_GET_LIBRARY
 336: #define FTN_SET_DEFAULTS KMP_SET_DEFAULTS
 337: #define FTN_SET_DISP_NUM_BUFFERS KMP_SET_DISP_NUM_BUFFERS
 338: #define FTN_SET_AFFINITY KMP_SET_AFFINITY
 339: #define FTN_GET_AFFINITY KMP_GET_AFFINITY
 340: #define FTN_GET_AFFINITY_MAX_PROC KMP_GET_AFFINITY_MAX_PROC
 341: #define FTN_CREATE_AFFINITY_MASK KMP_CREATE_AFFINITY_MASK
 342: #define FTN_DESTROY_AFFINITY_MASK KMP_DESTROY_AFFINITY_MASK
 343: #define FTN_SET_AFFINITY_MASK_PROC KMP_SET_AFFINITY_MASK_PROC
 344: #define FTN_UNSET_AFFINITY_MASK_PROC KMP_UNSET_AFFINITY_MASK_PROC
 345: #define FTN_GET_AFFINITY_MASK_PROC KMP_GET_AFFINITY_MASK_PROC
 346: 
```

- **L323**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。
- **L324**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L325**: Defines macro \`FTN_SET_STACKSIZE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_STACKSIZE\`，供条件编译或文本复用使用。
- **L326**: Defines macro \`FTN_SET_STACKSIZE_S\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_STACKSIZE_S\`，供条件编译或文本复用使用。
- **L327**: Defines macro \`FTN_GET_STACKSIZE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_STACKSIZE\`，供条件编译或文本复用使用。
- **L328**: Defines macro \`FTN_GET_STACKSIZE_S\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_STACKSIZE_S\`，供条件编译或文本复用使用。
- **L329**: Defines macro \`FTN_SET_BLOCKTIME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_BLOCKTIME\`，供条件编译或文本复用使用。
- **L330**: Defines macro \`FTN_GET_BLOCKTIME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_BLOCKTIME\`，供条件编译或文本复用使用。
- **L331**: Defines macro \`FTN_SET_LIBRARY_SERIAL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY_SERIAL\`，供条件编译或文本复用使用。
- **L332**: Defines macro \`FTN_SET_LIBRARY_TURNAROUND\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY_TURNAROUND\`，供条件编译或文本复用使用。
- **L333**: Defines macro \`FTN_SET_LIBRARY_THROUGHPUT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY_THROUGHPUT\`，供条件编译或文本复用使用。
- **L334**: Defines macro \`FTN_SET_LIBRARY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY\`，供条件编译或文本复用使用。
- **L335**: Defines macro \`FTN_GET_LIBRARY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_LIBRARY\`，供条件编译或文本复用使用。
- **L336**: Defines macro \`FTN_SET_DEFAULTS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DEFAULTS\`，供条件编译或文本复用使用。
- **L337**: Defines macro \`FTN_SET_DISP_NUM_BUFFERS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DISP_NUM_BUFFERS\`，供条件编译或文本复用使用。
- **L338**: Defines macro \`FTN_SET_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_AFFINITY\`，供条件编译或文本复用使用。
- **L339**: Defines macro \`FTN_GET_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY\`，供条件编译或文本复用使用。
- **L340**: Defines macro \`FTN_GET_AFFINITY_MAX_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY_MAX_PROC\`，供条件编译或文本复用使用。
- **L341**: Defines macro \`FTN_CREATE_AFFINITY_MASK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CREATE_AFFINITY_MASK\`，供条件编译或文本复用使用。
- **L342**: Defines macro \`FTN_DESTROY_AFFINITY_MASK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_AFFINITY_MASK\`，供条件编译或文本复用使用。
- **L343**: Defines macro \`FTN_SET_AFFINITY_MASK_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_AFFINITY_MASK_PROC\`，供条件编译或文本复用使用。
- **L344**: Defines macro \`FTN_UNSET_AFFINITY_MASK_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_UNSET_AFFINITY_MASK_PROC\`，供条件编译或文本复用使用。
- **L345**: Defines macro \`FTN_GET_AFFINITY_MASK_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY_MASK_PROC\`，供条件编译或文本复用使用。
- **L346**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 347-370 / 第 347-370 行

```cpp
 347: #define FTN_MALLOC KMP_MALLOC
 348: #define FTN_ALIGNED_MALLOC KMP_ALIGNED_MALLOC
 349: #define FTN_CALLOC KMP_CALLOC
 350: #define FTN_REALLOC KMP_REALLOC
 351: #define FTN_KFREE KMP_FREE
 352: 
 353: #define FTN_GET_NUM_KNOWN_THREADS KMP_GET_NUM_KNOWN_THREADS
 354: 
 355: #define FTN_SET_NUM_THREADS OMP_SET_NUM_THREADS
 356: #define FTN_GET_NUM_THREADS OMP_GET_NUM_THREADS
 357: #define FTN_GET_MAX_THREADS OMP_GET_MAX_THREADS
 358: #define FTN_GET_THREAD_NUM OMP_GET_THREAD_NUM
 359: #define FTN_GET_NUM_PROCS OMP_GET_NUM_PROCS
 360: #define FTN_SET_DYNAMIC OMP_SET_DYNAMIC
 361: #define FTN_GET_DYNAMIC OMP_GET_DYNAMIC
 362: #define FTN_SET_NESTED OMP_SET_NESTED
 363: #define FTN_GET_NESTED OMP_GET_NESTED
 364: #define FTN_IN_PARALLEL OMP_IN_PARALLEL
 365: #define FTN_GET_THREAD_LIMIT OMP_GET_THREAD_LIMIT
 366: #define FTN_SET_SCHEDULE OMP_SET_SCHEDULE
 367: #define FTN_GET_SCHEDULE OMP_GET_SCHEDULE
 368: #define FTN_SET_MAX_ACTIVE_LEVELS OMP_SET_MAX_ACTIVE_LEVELS
 369: #define FTN_GET_MAX_ACTIVE_LEVELS OMP_GET_MAX_ACTIVE_LEVELS
 370: #define FTN_GET_ACTIVE_LEVEL OMP_GET_ACTIVE_LEVEL
```

- **L347**: Defines macro \`FTN_MALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_MALLOC\`，供条件编译或文本复用使用。
- **L348**: Defines macro \`FTN_ALIGNED_MALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_ALIGNED_MALLOC\`，供条件编译或文本复用使用。
- **L349**: Defines macro \`FTN_CALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CALLOC\`，供条件编译或文本复用使用。
- **L350**: Defines macro \`FTN_REALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_REALLOC\`，供条件编译或文本复用使用。
- **L351**: Defines macro \`FTN_KFREE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_KFREE\`，供条件编译或文本复用使用。
- **L352**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L353**: Defines macro \`FTN_GET_NUM_KNOWN_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_KNOWN_THREADS\`，供条件编译或文本复用使用。
- **L354**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L355**: Defines macro \`FTN_SET_NUM_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NUM_THREADS\`，供条件编译或文本复用使用。
- **L356**: Defines macro \`FTN_GET_NUM_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_THREADS\`，供条件编译或文本复用使用。
- **L357**: Defines macro \`FTN_GET_MAX_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_THREADS\`，供条件编译或文本复用使用。
- **L358**: Defines macro \`FTN_GET_THREAD_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_THREAD_NUM\`，供条件编译或文本复用使用。
- **L359**: Defines macro \`FTN_GET_NUM_PROCS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_PROCS\`，供条件编译或文本复用使用。
- **L360**: Defines macro \`FTN_SET_DYNAMIC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DYNAMIC\`，供条件编译或文本复用使用。
- **L361**: Defines macro \`FTN_GET_DYNAMIC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DYNAMIC\`，供条件编译或文本复用使用。
- **L362**: Defines macro \`FTN_SET_NESTED\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NESTED\`，供条件编译或文本复用使用。
- **L363**: Defines macro \`FTN_GET_NESTED\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NESTED\`，供条件编译或文本复用使用。
- **L364**: Defines macro \`FTN_IN_PARALLEL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IN_PARALLEL\`，供条件编译或文本复用使用。
- **L365**: Defines macro \`FTN_GET_THREAD_LIMIT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_THREAD_LIMIT\`，供条件编译或文本复用使用。
- **L366**: Defines macro \`FTN_SET_SCHEDULE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_SCHEDULE\`，供条件编译或文本复用使用。
- **L367**: Defines macro \`FTN_GET_SCHEDULE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_SCHEDULE\`，供条件编译或文本复用使用。
- **L368**: Defines macro \`FTN_SET_MAX_ACTIVE_LEVELS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_MAX_ACTIVE_LEVELS\`，供条件编译或文本复用使用。
- **L369**: Defines macro \`FTN_GET_MAX_ACTIVE_LEVELS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_ACTIVE_LEVELS\`，供条件编译或文本复用使用。
- **L370**: Defines macro \`FTN_GET_ACTIVE_LEVEL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_ACTIVE_LEVEL\`，供条件编译或文本复用使用。

### Lines 371-382 / 第 371-382 行

```cpp
 371: #define FTN_GET_LEVEL OMP_GET_LEVEL
 372: #define FTN_GET_ANCESTOR_THREAD_NUM OMP_GET_ANCESTOR_THREAD_NUM
 373: #define FTN_GET_TEAM_SIZE OMP_GET_TEAM_SIZE
 374: #define FTN_IN_FINAL OMP_IN_FINAL
 375: #define FTN_GET_PROC_BIND OMP_GET_PROC_BIND
 376: #define FTN_GET_NUM_TEAMS OMP_GET_NUM_TEAMS
 377: #define FTN_GET_TEAM_NUM OMP_GET_TEAM_NUM
 378: #define FTN_INIT_LOCK OMP_INIT_LOCK
 379: #if KMP_USE_DYNAMIC_LOCK
 380: #define FTN_INIT_LOCK_WITH_HINT OMP_INIT_LOCK_WITH_HINT
 381: #define FTN_INIT_NEST_LOCK_WITH_HINT OMP_INIT_NEST_LOCK_WITH_HINT
 382: #endif
```

- **L371**: Defines macro \`FTN_GET_LEVEL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_LEVEL\`，供条件编译或文本复用使用。
- **L372**: Defines macro \`FTN_GET_ANCESTOR_THREAD_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_ANCESTOR_THREAD_NUM\`，供条件编译或文本复用使用。
- **L373**: Defines macro \`FTN_GET_TEAM_SIZE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_TEAM_SIZE\`，供条件编译或文本复用使用。
- **L374**: Defines macro \`FTN_IN_FINAL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IN_FINAL\`，供条件编译或文本复用使用。
- **L375**: Defines macro \`FTN_GET_PROC_BIND\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PROC_BIND\`，供条件编译或文本复用使用。
- **L376**: Defines macro \`FTN_GET_NUM_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_TEAMS\`，供条件编译或文本复用使用。
- **L377**: Defines macro \`FTN_GET_TEAM_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_TEAM_NUM\`，供条件编译或文本复用使用。
- **L378**: Defines macro \`FTN_INIT_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_LOCK\`，供条件编译或文本复用使用。
- **L379**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。
- **L380**: Defines macro \`FTN_INIT_LOCK_WITH_HINT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_LOCK_WITH_HINT\`，供条件编译或文本复用使用。
- **L381**: Defines macro \`FTN_INIT_NEST_LOCK_WITH_HINT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_NEST_LOCK_WITH_HINT\`，供条件编译或文本复用使用。
- **L382**: Ends the active conditional-compilation region. / 结束当前条件编译区域。

### Lines 383-395 / 第 383-395 行

```cpp
 383: #define FTN_DESTROY_LOCK OMP_DESTROY_LOCK
 384: #define FTN_SET_LOCK OMP_SET_LOCK
 385: #define FTN_UNSET_LOCK OMP_UNSET_LOCK
 386: #define FTN_TEST_LOCK OMP_TEST_LOCK
 387: #define FTN_INIT_NEST_LOCK OMP_INIT_NEST_LOCK
 388: #define FTN_DESTROY_NEST_LOCK OMP_DESTROY_NEST_LOCK
 389: #define FTN_SET_NEST_LOCK OMP_SET_NEST_LOCK
 390: #define FTN_UNSET_NEST_LOCK OMP_UNSET_NEST_LOCK
 391: #define FTN_TEST_NEST_LOCK OMP_TEST_NEST_LOCK
 392: 
 393: #define FTN_SET_WARNINGS_ON KMP_SET_WARNINGS_ON
 394: #define FTN_SET_WARNINGS_OFF KMP_SET_WARNINGS_OFF
 395: 
```

- **L383**: Defines macro \`FTN_DESTROY_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_LOCK\`，供条件编译或文本复用使用。
- **L384**: Defines macro \`FTN_SET_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LOCK\`，供条件编译或文本复用使用。
- **L385**: Defines macro \`FTN_UNSET_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_UNSET_LOCK\`，供条件编译或文本复用使用。
- **L386**: Defines macro \`FTN_TEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TEST_LOCK\`，供条件编译或文本复用使用。
- **L387**: Defines macro \`FTN_INIT_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_NEST_LOCK\`，供条件编译或文本复用使用。
- **L388**: Defines macro \`FTN_DESTROY_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_NEST_LOCK\`，供条件编译或文本复用使用。
- **L389**: Defines macro \`FTN_SET_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NEST_LOCK\`，供条件编译或文本复用使用。
- **L390**: Defines macro \`FTN_UNSET_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_UNSET_NEST_LOCK\`，供条件编译或文本复用使用。
- **L391**: Defines macro \`FTN_TEST_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TEST_NEST_LOCK\`，供条件编译或文本复用使用。
- **L392**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L393**: Defines macro \`FTN_SET_WARNINGS_ON\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_WARNINGS_ON\`，供条件编译或文本复用使用。
- **L394**: Defines macro \`FTN_SET_WARNINGS_OFF\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_WARNINGS_OFF\`，供条件编译或文本复用使用。
- **L395**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 396-415 / 第 396-415 行

```cpp
 396: #define FTN_GET_WTIME OMP_GET_WTIME
 397: #define FTN_GET_WTICK OMP_GET_WTICK
 398: 
 399: #define FTN_GET_NUM_DEVICES OMP_GET_NUM_DEVICES
 400: #define FTN_GET_DEFAULT_DEVICE OMP_GET_DEFAULT_DEVICE
 401: #define FTN_SET_DEFAULT_DEVICE OMP_SET_DEFAULT_DEVICE
 402: #define FTN_IS_INITIAL_DEVICE OMP_IS_INITIAL_DEVICE
 403: 
 404: #define FTN_GET_CANCELLATION OMP_GET_CANCELLATION
 405: #define FTN_GET_CANCELLATION_STATUS KMP_GET_CANCELLATION_STATUS
 406: 
 407: #define FTN_GET_MAX_TASK_PRIORITY OMP_GET_MAX_TASK_PRIORITY
 408: #define FTN_GET_NUM_PLACES OMP_GET_NUM_PLACES
 409: #define FTN_GET_PLACE_NUM_PROCS OMP_GET_PLACE_NUM_PROCS
 410: #define FTN_GET_PLACE_PROC_IDS OMP_GET_PLACE_PROC_IDS
 411: #define FTN_GET_PLACE_NUM OMP_GET_PLACE_NUM
 412: #define FTN_GET_PARTITION_NUM_PLACES OMP_GET_PARTITION_NUM_PLACES
 413: #define FTN_GET_PARTITION_PLACE_NUMS OMP_GET_PARTITION_PLACE_NUMS
 414: #define FTN_GET_INITIAL_DEVICE OMP_GET_INITIAL_DEVICE
 415: #ifdef KMP_STUB
```

- **L396**: Defines macro \`FTN_GET_WTIME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_WTIME\`，供条件编译或文本复用使用。
- **L397**: Defines macro \`FTN_GET_WTICK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_WTICK\`，供条件编译或文本复用使用。
- **L398**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L399**: Defines macro \`FTN_GET_NUM_DEVICES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_DEVICES\`，供条件编译或文本复用使用。
- **L400**: Defines macro \`FTN_GET_DEFAULT_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEFAULT_DEVICE\`，供条件编译或文本复用使用。
- **L401**: Defines macro \`FTN_SET_DEFAULT_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DEFAULT_DEVICE\`，供条件编译或文本复用使用。
- **L402**: Defines macro \`FTN_IS_INITIAL_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IS_INITIAL_DEVICE\`，供条件编译或文本复用使用。
- **L403**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L404**: Defines macro \`FTN_GET_CANCELLATION\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_CANCELLATION\`，供条件编译或文本复用使用。
- **L405**: Defines macro \`FTN_GET_CANCELLATION_STATUS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_CANCELLATION_STATUS\`，供条件编译或文本复用使用。
- **L406**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L407**: Defines macro \`FTN_GET_MAX_TASK_PRIORITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_TASK_PRIORITY\`，供条件编译或文本复用使用。
- **L408**: Defines macro \`FTN_GET_NUM_PLACES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_PLACES\`，供条件编译或文本复用使用。
- **L409**: Defines macro \`FTN_GET_PLACE_NUM_PROCS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PLACE_NUM_PROCS\`，供条件编译或文本复用使用。
- **L410**: Defines macro \`FTN_GET_PLACE_PROC_IDS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PLACE_PROC_IDS\`，供条件编译或文本复用使用。
- **L411**: Defines macro \`FTN_GET_PLACE_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PLACE_NUM\`，供条件编译或文本复用使用。
- **L412**: Defines macro \`FTN_GET_PARTITION_NUM_PLACES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PARTITION_NUM_PLACES\`，供条件编译或文本复用使用。
- **L413**: Defines macro \`FTN_GET_PARTITION_PLACE_NUMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PARTITION_PLACE_NUMS\`，供条件编译或文本复用使用。
- **L414**: Defines macro \`FTN_GET_INITIAL_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INITIAL_DEVICE\`，供条件编译或文本复用使用。
- **L415**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。

### Lines 416-439 / 第 416-439 行

```cpp
 416: #define FTN_TARGET_ALLOC OMP_TARGET_ALLOC
 417: #define FTN_TARGET_FREE OMP_TARGET_FREE
 418: #define FTN_TARGET_IS_PRESENT OMP_TARGET_IS_PRESENT
 419: #define FTN_TARGET_MEMCPY OMP_TARGET_MEMCPY
 420: #define FTN_TARGET_MEMCPY_RECT OMP_TARGET_MEMCPY_RECT
 421: #define FTN_TARGET_ASSOCIATE_PTR OMP_TARGET_ASSOCIATE_PTR
 422: #define FTN_TARGET_DISASSOCIATE_PTR OMP_TARGET_DISASSOCIATE_PTR
 423: #endif
 424: 
 425: #define FTN_CONTROL_TOOL OMP_CONTROL_TOOL
 426: #define FTN_INIT_ALLOCATOR OMP_INIT_ALLOCATOR
 427: #define FTN_DESTROY_ALLOCATOR OMP_DESTROY_ALLOCATOR
 428: #define FTN_SET_DEFAULT_ALLOCATOR OMP_SET_DEFAULT_ALLOCATOR
 429: #define FTN_GET_DEFAULT_ALLOCATOR OMP_GET_DEFAULT_ALLOCATOR
 430: #define FTN_GET_DEVICES_MEMSPACE OMP_GET_DEVICES_MEMSPACE
 431: #define FTN_GET_DEVICE_MEMSPACE OMP_GET_DEVICE_MEMSPACE
 432: #define FTN_GET_DEVICES_AND_HOST_MEMSPACE OMP_GET_DEVICES_AND_HOST_MEMSPACE
 433: #define FTN_GET_DEVICE_AND_HOST_MEMSPACE OMP_GET_DEVICE_AND_HOST_MEMSPACE
 434: #define FTN_GET_DEVICES_ALL_MEMSPACE OMP_GET_DEVICES_ALL_MEMSPACE
 435: #define FTN_GET_DEVICES_ALLOCATOR OMP_GET_DEVICES_ALLOCATOR
 436: #define FTN_GET_DEVICE_ALLOCATOR OMP_GET_DEVICE_ALLOCATOR
 437: #define FTN_GET_DEVICES_AND_HOST_ALLOCATOR OMP_GET_DEVICES_AND_HOST_ALLOCATOR
 438: #define FTN_GET_DEVICE_AND_HOST_ALLOCATOR OMP_GET_DEVICE_AND_HOST_ALLOCATOR
 439: #define FTN_GET_DEVICES_ALL_ALLOCATOR OMP_GET_DEVICES_ALL_ALLOCATOR
```

- **L416**: Defines macro \`FTN_TARGET_ALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_ALLOC\`，供条件编译或文本复用使用。
- **L417**: Defines macro \`FTN_TARGET_FREE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_FREE\`，供条件编译或文本复用使用。
- **L418**: Defines macro \`FTN_TARGET_IS_PRESENT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_IS_PRESENT\`，供条件编译或文本复用使用。
- **L419**: Defines macro \`FTN_TARGET_MEMCPY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_MEMCPY\`，供条件编译或文本复用使用。
- **L420**: Defines macro \`FTN_TARGET_MEMCPY_RECT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_MEMCPY_RECT\`，供条件编译或文本复用使用。
- **L421**: Defines macro \`FTN_TARGET_ASSOCIATE_PTR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_ASSOCIATE_PTR\`，供条件编译或文本复用使用。
- **L422**: Defines macro \`FTN_TARGET_DISASSOCIATE_PTR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_DISASSOCIATE_PTR\`，供条件编译或文本复用使用。
- **L423**: Ends the active conditional-compilation region. / 结束当前条件编译区域。
- **L424**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L425**: Defines macro \`FTN_CONTROL_TOOL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CONTROL_TOOL\`，供条件编译或文本复用使用。
- **L426**: Defines macro \`FTN_INIT_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_ALLOCATOR\`，供条件编译或文本复用使用。
- **L427**: Defines macro \`FTN_DESTROY_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_ALLOCATOR\`，供条件编译或文本复用使用。
- **L428**: Defines macro \`FTN_SET_DEFAULT_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DEFAULT_ALLOCATOR\`，供条件编译或文本复用使用。
- **L429**: Defines macro \`FTN_GET_DEFAULT_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEFAULT_ALLOCATOR\`，供条件编译或文本复用使用。
- **L430**: Defines macro \`FTN_GET_DEVICES_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_MEMSPACE\`，供条件编译或文本复用使用。
- **L431**: Defines macro \`FTN_GET_DEVICE_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_MEMSPACE\`，供条件编译或文本复用使用。
- **L432**: Defines macro \`FTN_GET_DEVICES_AND_HOST_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_AND_HOST_MEMSPACE\`，供条件编译或文本复用使用。
- **L433**: Defines macro \`FTN_GET_DEVICE_AND_HOST_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_AND_HOST_MEMSPACE\`，供条件编译或文本复用使用。
- **L434**: Defines macro \`FTN_GET_DEVICES_ALL_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_ALL_MEMSPACE\`，供条件编译或文本复用使用。
- **L435**: Defines macro \`FTN_GET_DEVICES_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_ALLOCATOR\`，供条件编译或文本复用使用。
- **L436**: Defines macro \`FTN_GET_DEVICE_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_ALLOCATOR\`，供条件编译或文本复用使用。
- **L437**: Defines macro \`FTN_GET_DEVICES_AND_HOST_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_AND_HOST_ALLOCATOR\`，供条件编译或文本复用使用。
- **L438**: Defines macro \`FTN_GET_DEVICE_AND_HOST_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_AND_HOST_ALLOCATOR\`，供条件编译或文本复用使用。
- **L439**: Defines macro \`FTN_GET_DEVICES_ALL_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_ALL_ALLOCATOR\`，供条件编译或文本复用使用。

### Lines 440-459 / 第 440-459 行

```cpp
 440: #define FTN_GET_MEMSPACE_NUM_RESOURCES OMP_GET_MEMSPACE_NUM_RESOURCES
 441: #define FTN_GET_SUBMEMSPACE OMP_GET_SUBMEMSPACE
 442: #define FTN_GET_DEVICE_NUM OMP_GET_DEVICE_NUM
 443: #define FTN_GET_UID_FROM_DEVICE OMP_GET_UID_FROM_DEVICE
 444: #define FTN_GET_DEVICE_FROM_UID OMP_GET_DEVICE_FROM_UID
 445: #define FTN_SET_AFFINITY_FORMAT OMP_SET_AFFINITY_FORMAT
 446: #define FTN_GET_AFFINITY_FORMAT OMP_GET_AFFINITY_FORMAT
 447: #define FTN_DISPLAY_AFFINITY OMP_DISPLAY_AFFINITY
 448: #define FTN_CAPTURE_AFFINITY OMP_CAPTURE_AFFINITY
 449: #define FTN_PAUSE_RESOURCE OMP_PAUSE_RESOURCE
 450: #define FTN_PAUSE_RESOURCE_ALL OMP_PAUSE_RESOURCE_ALL
 451: #define FTN_GET_SUPPORTED_ACTIVE_LEVELS OMP_GET_SUPPORTED_ACTIVE_LEVELS
 452: #define FTN_DISPLAY_ENV OMP_DISPLAY_ENV
 453: #define FTN_IN_EXPLICIT_TASK OMP_IN_EXPLICIT_TASK
 454: #define FTN_FULFILL_EVENT OMP_FULFILL_EVENT
 455: #define FTN_SET_NUM_TEAMS OMP_SET_NUM_TEAMS
 456: #define FTN_GET_MAX_TEAMS OMP_GET_MAX_TEAMS
 457: #define FTN_SET_TEAMS_THREAD_LIMIT OMP_SET_TEAMS_THREAD_LIMIT
 458: #define FTN_GET_TEAMS_THREAD_LIMIT OMP_GET_TEAMS_THREAD_LIMIT
 459: 
```

- **L440**: Defines macro \`FTN_GET_MEMSPACE_NUM_RESOURCES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MEMSPACE_NUM_RESOURCES\`，供条件编译或文本复用使用。
- **L441**: Defines macro \`FTN_GET_SUBMEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_SUBMEMSPACE\`，供条件编译或文本复用使用。
- **L442**: Defines macro \`FTN_GET_DEVICE_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_NUM\`，供条件编译或文本复用使用。
- **L443**: Defines macro \`FTN_GET_UID_FROM_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_UID_FROM_DEVICE\`，供条件编译或文本复用使用。
- **L444**: Defines macro \`FTN_GET_DEVICE_FROM_UID\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_FROM_UID\`，供条件编译或文本复用使用。
- **L445**: Defines macro \`FTN_SET_AFFINITY_FORMAT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_AFFINITY_FORMAT\`，供条件编译或文本复用使用。
- **L446**: Defines macro \`FTN_GET_AFFINITY_FORMAT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY_FORMAT\`，供条件编译或文本复用使用。
- **L447**: Defines macro \`FTN_DISPLAY_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DISPLAY_AFFINITY\`，供条件编译或文本复用使用。
- **L448**: Defines macro \`FTN_CAPTURE_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CAPTURE_AFFINITY\`，供条件编译或文本复用使用。
- **L449**: Defines macro \`FTN_PAUSE_RESOURCE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_PAUSE_RESOURCE\`，供条件编译或文本复用使用。
- **L450**: Defines macro \`FTN_PAUSE_RESOURCE_ALL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_PAUSE_RESOURCE_ALL\`，供条件编译或文本复用使用。
- **L451**: Defines macro \`FTN_GET_SUPPORTED_ACTIVE_LEVELS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_SUPPORTED_ACTIVE_LEVELS\`，供条件编译或文本复用使用。
- **L452**: Defines macro \`FTN_DISPLAY_ENV\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DISPLAY_ENV\`，供条件编译或文本复用使用。
- **L453**: Defines macro \`FTN_IN_EXPLICIT_TASK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IN_EXPLICIT_TASK\`，供条件编译或文本复用使用。
- **L454**: Defines macro \`FTN_FULFILL_EVENT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_FULFILL_EVENT\`，供条件编译或文本复用使用。
- **L455**: Defines macro \`FTN_SET_NUM_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NUM_TEAMS\`，供条件编译或文本复用使用。
- **L456**: Defines macro \`FTN_GET_MAX_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_TEAMS\`，供条件编译或文本复用使用。
- **L457**: Defines macro \`FTN_SET_TEAMS_THREAD_LIMIT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_TEAMS_THREAD_LIMIT\`，供条件编译或文本复用使用。
- **L458**: Defines macro \`FTN_GET_TEAMS_THREAD_LIMIT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_TEAMS_THREAD_LIMIT\`，供条件编译或文本复用使用。
- **L459**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 460-471 / 第 460-471 行

```cpp
 460: #define FTN_GET_NUM_INTEROP_PROPERTIES OMP_GET_NUM_INTEROP_PROPERTIES
 461: #define FTN_GET_INTEROP_INT OMP_GET_INTEROP_INT
 462: #define FTN_GET_INTEROP_PTR OMP_GET_INTEROP_PTR
 463: #define FTN_GET_INTEROP_STR OMP_GET_INTEROP_STR
 464: #define FTN_GET_INTEROP_NAME OMP_GET_INTEROP_NAME
 465: #define FTN_GET_INTEROP_TYPE_DESC OMP_GET_INTEROP_TYPE_DESC
 466: #define FTN_GET_INTEROP_RC_DESC OMP_GET_INTEROP_RC_DESC
 467: 
 468: #endif /* KMP_FTN_UPPER */
 469: 
 470: /* ------------------------------------------------------------------------ */
 471: 
```

- **L460**: Defines macro \`FTN_GET_NUM_INTEROP_PROPERTIES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_INTEROP_PROPERTIES\`，供条件编译或文本复用使用。
- **L461**: Defines macro \`FTN_GET_INTEROP_INT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_INT\`，供条件编译或文本复用使用。
- **L462**: Defines macro \`FTN_GET_INTEROP_PTR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_PTR\`，供条件编译或文本复用使用。
- **L463**: Defines macro \`FTN_GET_INTEROP_STR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_STR\`，供条件编译或文本复用使用。
- **L464**: Defines macro \`FTN_GET_INTEROP_NAME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_NAME\`，供条件编译或文本复用使用。
- **L465**: Defines macro \`FTN_GET_INTEROP_TYPE_DESC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_TYPE_DESC\`，供条件编译或文本复用使用。
- **L466**: Defines macro \`FTN_GET_INTEROP_RC_DESC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_RC_DESC\`，供条件编译或文本复用使用。
- **L467**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L468**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L469**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L470**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L471**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 472-495 / 第 472-495 行

```cpp
 472: #if KMP_FTN_ENTRIES == KMP_FTN_UAPPEND
 473: 
 474: #define FTN_SET_STACKSIZE KMP_SET_STACKSIZE_
 475: #define FTN_SET_STACKSIZE_S KMP_SET_STACKSIZE_S_
 476: #define FTN_GET_STACKSIZE KMP_GET_STACKSIZE_
 477: #define FTN_GET_STACKSIZE_S KMP_GET_STACKSIZE_S_
 478: #define FTN_SET_BLOCKTIME KMP_SET_BLOCKTIME_
 479: #define FTN_GET_BLOCKTIME KMP_GET_BLOCKTIME_
 480: #define FTN_SET_LIBRARY_SERIAL KMP_SET_LIBRARY_SERIAL_
 481: #define FTN_SET_LIBRARY_TURNAROUND KMP_SET_LIBRARY_TURNAROUND_
 482: #define FTN_SET_LIBRARY_THROUGHPUT KMP_SET_LIBRARY_THROUGHPUT_
 483: #define FTN_SET_LIBRARY KMP_SET_LIBRARY_
 484: #define FTN_GET_LIBRARY KMP_GET_LIBRARY_
 485: #define FTN_SET_DEFAULTS KMP_SET_DEFAULTS_
 486: #define FTN_SET_DISP_NUM_BUFFERS KMP_SET_DISP_NUM_BUFFERS_
 487: #define FTN_SET_AFFINITY KMP_SET_AFFINITY_
 488: #define FTN_GET_AFFINITY KMP_GET_AFFINITY_
 489: #define FTN_GET_AFFINITY_MAX_PROC KMP_GET_AFFINITY_MAX_PROC_
 490: #define FTN_CREATE_AFFINITY_MASK KMP_CREATE_AFFINITY_MASK_
 491: #define FTN_DESTROY_AFFINITY_MASK KMP_DESTROY_AFFINITY_MASK_
 492: #define FTN_SET_AFFINITY_MASK_PROC KMP_SET_AFFINITY_MASK_PROC_
 493: #define FTN_UNSET_AFFINITY_MASK_PROC KMP_UNSET_AFFINITY_MASK_PROC_
 494: #define FTN_GET_AFFINITY_MASK_PROC KMP_GET_AFFINITY_MASK_PROC_
 495: 
```

- **L472**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。
- **L473**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L474**: Defines macro \`FTN_SET_STACKSIZE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_STACKSIZE\`，供条件编译或文本复用使用。
- **L475**: Defines macro \`FTN_SET_STACKSIZE_S\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_STACKSIZE_S\`，供条件编译或文本复用使用。
- **L476**: Defines macro \`FTN_GET_STACKSIZE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_STACKSIZE\`，供条件编译或文本复用使用。
- **L477**: Defines macro \`FTN_GET_STACKSIZE_S\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_STACKSIZE_S\`，供条件编译或文本复用使用。
- **L478**: Defines macro \`FTN_SET_BLOCKTIME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_BLOCKTIME\`，供条件编译或文本复用使用。
- **L479**: Defines macro \`FTN_GET_BLOCKTIME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_BLOCKTIME\`，供条件编译或文本复用使用。
- **L480**: Defines macro \`FTN_SET_LIBRARY_SERIAL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY_SERIAL\`，供条件编译或文本复用使用。
- **L481**: Defines macro \`FTN_SET_LIBRARY_TURNAROUND\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY_TURNAROUND\`，供条件编译或文本复用使用。
- **L482**: Defines macro \`FTN_SET_LIBRARY_THROUGHPUT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY_THROUGHPUT\`，供条件编译或文本复用使用。
- **L483**: Defines macro \`FTN_SET_LIBRARY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LIBRARY\`，供条件编译或文本复用使用。
- **L484**: Defines macro \`FTN_GET_LIBRARY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_LIBRARY\`，供条件编译或文本复用使用。
- **L485**: Defines macro \`FTN_SET_DEFAULTS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DEFAULTS\`，供条件编译或文本复用使用。
- **L486**: Defines macro \`FTN_SET_DISP_NUM_BUFFERS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DISP_NUM_BUFFERS\`，供条件编译或文本复用使用。
- **L487**: Defines macro \`FTN_SET_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_AFFINITY\`，供条件编译或文本复用使用。
- **L488**: Defines macro \`FTN_GET_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY\`，供条件编译或文本复用使用。
- **L489**: Defines macro \`FTN_GET_AFFINITY_MAX_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY_MAX_PROC\`，供条件编译或文本复用使用。
- **L490**: Defines macro \`FTN_CREATE_AFFINITY_MASK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CREATE_AFFINITY_MASK\`，供条件编译或文本复用使用。
- **L491**: Defines macro \`FTN_DESTROY_AFFINITY_MASK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_AFFINITY_MASK\`，供条件编译或文本复用使用。
- **L492**: Defines macro \`FTN_SET_AFFINITY_MASK_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_AFFINITY_MASK_PROC\`，供条件编译或文本复用使用。
- **L493**: Defines macro \`FTN_UNSET_AFFINITY_MASK_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_UNSET_AFFINITY_MASK_PROC\`，供条件编译或文本复用使用。
- **L494**: Defines macro \`FTN_GET_AFFINITY_MASK_PROC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY_MASK_PROC\`，供条件编译或文本复用使用。
- **L495**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 496-519 / 第 496-519 行

```cpp
 496: #define FTN_MALLOC KMP_MALLOC_
 497: #define FTN_ALIGNED_MALLOC KMP_ALIGNED_MALLOC_
 498: #define FTN_CALLOC KMP_CALLOC_
 499: #define FTN_REALLOC KMP_REALLOC_
 500: #define FTN_KFREE KMP_FREE_
 501: 
 502: #define FTN_GET_NUM_KNOWN_THREADS KMP_GET_NUM_KNOWN_THREADS_
 503: 
 504: #define FTN_SET_NUM_THREADS OMP_SET_NUM_THREADS_
 505: #define FTN_GET_NUM_THREADS OMP_GET_NUM_THREADS_
 506: #define FTN_GET_MAX_THREADS OMP_GET_MAX_THREADS_
 507: #define FTN_GET_THREAD_NUM OMP_GET_THREAD_NUM_
 508: #define FTN_GET_NUM_PROCS OMP_GET_NUM_PROCS_
 509: #define FTN_SET_DYNAMIC OMP_SET_DYNAMIC_
 510: #define FTN_GET_DYNAMIC OMP_GET_DYNAMIC_
 511: #define FTN_SET_NESTED OMP_SET_NESTED_
 512: #define FTN_GET_NESTED OMP_GET_NESTED_
 513: #define FTN_IN_PARALLEL OMP_IN_PARALLEL_
 514: #define FTN_GET_THREAD_LIMIT OMP_GET_THREAD_LIMIT_
 515: #define FTN_SET_SCHEDULE OMP_SET_SCHEDULE_
 516: #define FTN_GET_SCHEDULE OMP_GET_SCHEDULE_
 517: #define FTN_SET_MAX_ACTIVE_LEVELS OMP_SET_MAX_ACTIVE_LEVELS_
 518: #define FTN_GET_MAX_ACTIVE_LEVELS OMP_GET_MAX_ACTIVE_LEVELS_
 519: #define FTN_GET_ACTIVE_LEVEL OMP_GET_ACTIVE_LEVEL_
```

- **L496**: Defines macro \`FTN_MALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_MALLOC\`，供条件编译或文本复用使用。
- **L497**: Defines macro \`FTN_ALIGNED_MALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_ALIGNED_MALLOC\`，供条件编译或文本复用使用。
- **L498**: Defines macro \`FTN_CALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CALLOC\`，供条件编译或文本复用使用。
- **L499**: Defines macro \`FTN_REALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_REALLOC\`，供条件编译或文本复用使用。
- **L500**: Defines macro \`FTN_KFREE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_KFREE\`，供条件编译或文本复用使用。
- **L501**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L502**: Defines macro \`FTN_GET_NUM_KNOWN_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_KNOWN_THREADS\`，供条件编译或文本复用使用。
- **L503**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L504**: Defines macro \`FTN_SET_NUM_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NUM_THREADS\`，供条件编译或文本复用使用。
- **L505**: Defines macro \`FTN_GET_NUM_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_THREADS\`，供条件编译或文本复用使用。
- **L506**: Defines macro \`FTN_GET_MAX_THREADS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_THREADS\`，供条件编译或文本复用使用。
- **L507**: Defines macro \`FTN_GET_THREAD_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_THREAD_NUM\`，供条件编译或文本复用使用。
- **L508**: Defines macro \`FTN_GET_NUM_PROCS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_PROCS\`，供条件编译或文本复用使用。
- **L509**: Defines macro \`FTN_SET_DYNAMIC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DYNAMIC\`，供条件编译或文本复用使用。
- **L510**: Defines macro \`FTN_GET_DYNAMIC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DYNAMIC\`，供条件编译或文本复用使用。
- **L511**: Defines macro \`FTN_SET_NESTED\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NESTED\`，供条件编译或文本复用使用。
- **L512**: Defines macro \`FTN_GET_NESTED\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NESTED\`，供条件编译或文本复用使用。
- **L513**: Defines macro \`FTN_IN_PARALLEL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IN_PARALLEL\`，供条件编译或文本复用使用。
- **L514**: Defines macro \`FTN_GET_THREAD_LIMIT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_THREAD_LIMIT\`，供条件编译或文本复用使用。
- **L515**: Defines macro \`FTN_SET_SCHEDULE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_SCHEDULE\`，供条件编译或文本复用使用。
- **L516**: Defines macro \`FTN_GET_SCHEDULE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_SCHEDULE\`，供条件编译或文本复用使用。
- **L517**: Defines macro \`FTN_SET_MAX_ACTIVE_LEVELS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_MAX_ACTIVE_LEVELS\`，供条件编译或文本复用使用。
- **L518**: Defines macro \`FTN_GET_MAX_ACTIVE_LEVELS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_ACTIVE_LEVELS\`，供条件编译或文本复用使用。
- **L519**: Defines macro \`FTN_GET_ACTIVE_LEVEL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_ACTIVE_LEVEL\`，供条件编译或文本复用使用。

### Lines 520-531 / 第 520-531 行

```cpp
 520: #define FTN_GET_LEVEL OMP_GET_LEVEL_
 521: #define FTN_GET_ANCESTOR_THREAD_NUM OMP_GET_ANCESTOR_THREAD_NUM_
 522: #define FTN_GET_TEAM_SIZE OMP_GET_TEAM_SIZE_
 523: #define FTN_IN_FINAL OMP_IN_FINAL_
 524: #define FTN_GET_PROC_BIND OMP_GET_PROC_BIND_
 525: #define FTN_GET_NUM_TEAMS OMP_GET_NUM_TEAMS_
 526: #define FTN_GET_TEAM_NUM OMP_GET_TEAM_NUM_
 527: #define FTN_INIT_LOCK OMP_INIT_LOCK_
 528: #if KMP_USE_DYNAMIC_LOCK
 529: #define FTN_INIT_LOCK_WITH_HINT OMP_INIT_LOCK_WITH_HINT_
 530: #define FTN_INIT_NEST_LOCK_WITH_HINT OMP_INIT_NEST_LOCK_WITH_HINT_
 531: #endif
```

- **L520**: Defines macro \`FTN_GET_LEVEL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_LEVEL\`，供条件编译或文本复用使用。
- **L521**: Defines macro \`FTN_GET_ANCESTOR_THREAD_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_ANCESTOR_THREAD_NUM\`，供条件编译或文本复用使用。
- **L522**: Defines macro \`FTN_GET_TEAM_SIZE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_TEAM_SIZE\`，供条件编译或文本复用使用。
- **L523**: Defines macro \`FTN_IN_FINAL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IN_FINAL\`，供条件编译或文本复用使用。
- **L524**: Defines macro \`FTN_GET_PROC_BIND\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PROC_BIND\`，供条件编译或文本复用使用。
- **L525**: Defines macro \`FTN_GET_NUM_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_TEAMS\`，供条件编译或文本复用使用。
- **L526**: Defines macro \`FTN_GET_TEAM_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_TEAM_NUM\`，供条件编译或文本复用使用。
- **L527**: Defines macro \`FTN_INIT_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_LOCK\`，供条件编译或文本复用使用。
- **L528**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。
- **L529**: Defines macro \`FTN_INIT_LOCK_WITH_HINT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_LOCK_WITH_HINT\`，供条件编译或文本复用使用。
- **L530**: Defines macro \`FTN_INIT_NEST_LOCK_WITH_HINT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_NEST_LOCK_WITH_HINT\`，供条件编译或文本复用使用。
- **L531**: Ends the active conditional-compilation region. / 结束当前条件编译区域。

### Lines 532-544 / 第 532-544 行

```cpp
 532: #define FTN_DESTROY_LOCK OMP_DESTROY_LOCK_
 533: #define FTN_SET_LOCK OMP_SET_LOCK_
 534: #define FTN_UNSET_LOCK OMP_UNSET_LOCK_
 535: #define FTN_TEST_LOCK OMP_TEST_LOCK_
 536: #define FTN_INIT_NEST_LOCK OMP_INIT_NEST_LOCK_
 537: #define FTN_DESTROY_NEST_LOCK OMP_DESTROY_NEST_LOCK_
 538: #define FTN_SET_NEST_LOCK OMP_SET_NEST_LOCK_
 539: #define FTN_UNSET_NEST_LOCK OMP_UNSET_NEST_LOCK_
 540: #define FTN_TEST_NEST_LOCK OMP_TEST_NEST_LOCK_
 541: 
 542: #define FTN_SET_WARNINGS_ON KMP_SET_WARNINGS_ON_
 543: #define FTN_SET_WARNINGS_OFF KMP_SET_WARNINGS_OFF_
 544: 
```

- **L532**: Defines macro \`FTN_DESTROY_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_LOCK\`，供条件编译或文本复用使用。
- **L533**: Defines macro \`FTN_SET_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_LOCK\`，供条件编译或文本复用使用。
- **L534**: Defines macro \`FTN_UNSET_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_UNSET_LOCK\`，供条件编译或文本复用使用。
- **L535**: Defines macro \`FTN_TEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TEST_LOCK\`，供条件编译或文本复用使用。
- **L536**: Defines macro \`FTN_INIT_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_NEST_LOCK\`，供条件编译或文本复用使用。
- **L537**: Defines macro \`FTN_DESTROY_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_NEST_LOCK\`，供条件编译或文本复用使用。
- **L538**: Defines macro \`FTN_SET_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NEST_LOCK\`，供条件编译或文本复用使用。
- **L539**: Defines macro \`FTN_UNSET_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_UNSET_NEST_LOCK\`，供条件编译或文本复用使用。
- **L540**: Defines macro \`FTN_TEST_NEST_LOCK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TEST_NEST_LOCK\`，供条件编译或文本复用使用。
- **L541**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L542**: Defines macro \`FTN_SET_WARNINGS_ON\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_WARNINGS_ON\`，供条件编译或文本复用使用。
- **L543**: Defines macro \`FTN_SET_WARNINGS_OFF\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_WARNINGS_OFF\`，供条件编译或文本复用使用。
- **L544**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 545-564 / 第 545-564 行

```cpp
 545: #define FTN_GET_WTIME OMP_GET_WTIME_
 546: #define FTN_GET_WTICK OMP_GET_WTICK_
 547: 
 548: #define FTN_GET_NUM_DEVICES OMP_GET_NUM_DEVICES_
 549: #define FTN_GET_DEFAULT_DEVICE OMP_GET_DEFAULT_DEVICE_
 550: #define FTN_SET_DEFAULT_DEVICE OMP_SET_DEFAULT_DEVICE_
 551: #define FTN_IS_INITIAL_DEVICE OMP_IS_INITIAL_DEVICE_
 552: 
 553: #define FTN_GET_CANCELLATION OMP_GET_CANCELLATION_
 554: #define FTN_GET_CANCELLATION_STATUS KMP_GET_CANCELLATION_STATUS_
 555: 
 556: #define FTN_GET_MAX_TASK_PRIORITY OMP_GET_MAX_TASK_PRIORITY_
 557: #define FTN_GET_NUM_PLACES OMP_GET_NUM_PLACES_
 558: #define FTN_GET_PLACE_NUM_PROCS OMP_GET_PLACE_NUM_PROCS_
 559: #define FTN_GET_PLACE_PROC_IDS OMP_GET_PLACE_PROC_IDS_
 560: #define FTN_GET_PLACE_NUM OMP_GET_PLACE_NUM_
 561: #define FTN_GET_PARTITION_NUM_PLACES OMP_GET_PARTITION_NUM_PLACES_
 562: #define FTN_GET_PARTITION_PLACE_NUMS OMP_GET_PARTITION_PLACE_NUMS_
 563: #define FTN_GET_INITIAL_DEVICE OMP_GET_INITIAL_DEVICE_
 564: #ifdef KMP_STUB
```

- **L545**: Defines macro \`FTN_GET_WTIME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_WTIME\`，供条件编译或文本复用使用。
- **L546**: Defines macro \`FTN_GET_WTICK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_WTICK\`，供条件编译或文本复用使用。
- **L547**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L548**: Defines macro \`FTN_GET_NUM_DEVICES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_DEVICES\`，供条件编译或文本复用使用。
- **L549**: Defines macro \`FTN_GET_DEFAULT_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEFAULT_DEVICE\`，供条件编译或文本复用使用。
- **L550**: Defines macro \`FTN_SET_DEFAULT_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DEFAULT_DEVICE\`，供条件编译或文本复用使用。
- **L551**: Defines macro \`FTN_IS_INITIAL_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IS_INITIAL_DEVICE\`，供条件编译或文本复用使用。
- **L552**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L553**: Defines macro \`FTN_GET_CANCELLATION\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_CANCELLATION\`，供条件编译或文本复用使用。
- **L554**: Defines macro \`FTN_GET_CANCELLATION_STATUS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_CANCELLATION_STATUS\`，供条件编译或文本复用使用。
- **L555**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L556**: Defines macro \`FTN_GET_MAX_TASK_PRIORITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_TASK_PRIORITY\`，供条件编译或文本复用使用。
- **L557**: Defines macro \`FTN_GET_NUM_PLACES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_PLACES\`，供条件编译或文本复用使用。
- **L558**: Defines macro \`FTN_GET_PLACE_NUM_PROCS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PLACE_NUM_PROCS\`，供条件编译或文本复用使用。
- **L559**: Defines macro \`FTN_GET_PLACE_PROC_IDS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PLACE_PROC_IDS\`，供条件编译或文本复用使用。
- **L560**: Defines macro \`FTN_GET_PLACE_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PLACE_NUM\`，供条件编译或文本复用使用。
- **L561**: Defines macro \`FTN_GET_PARTITION_NUM_PLACES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PARTITION_NUM_PLACES\`，供条件编译或文本复用使用。
- **L562**: Defines macro \`FTN_GET_PARTITION_PLACE_NUMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_PARTITION_PLACE_NUMS\`，供条件编译或文本复用使用。
- **L563**: Defines macro \`FTN_GET_INITIAL_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INITIAL_DEVICE\`，供条件编译或文本复用使用。
- **L564**: Starts a conditional-compilation region controlled by macros or feature tests. / 开始一个由宏或特性测试控制的条件编译区域。

### Lines 565-588 / 第 565-588 行

```cpp
 565: #define FTN_TARGET_ALLOC OMP_TARGET_ALLOC_
 566: #define FTN_TARGET_FREE OMP_TARGET_FREE_
 567: #define FTN_TARGET_IS_PRESENT OMP_TARGET_IS_PRESENT_
 568: #define FTN_TARGET_MEMCPY OMP_TARGET_MEMCPY_
 569: #define FTN_TARGET_MEMCPY_RECT OMP_TARGET_MEMCPY_RECT_
 570: #define FTN_TARGET_ASSOCIATE_PTR OMP_TARGET_ASSOCIATE_PTR_
 571: #define FTN_TARGET_DISASSOCIATE_PTR OMP_TARGET_DISASSOCIATE_PTR_
 572: #endif
 573: 
 574: #define FTN_CONTROL_TOOL OMP_CONTROL_TOOL_
 575: #define FTN_INIT_ALLOCATOR OMP_INIT_ALLOCATOR_
 576: #define FTN_DESTROY_ALLOCATOR OMP_DESTROY_ALLOCATOR_
 577: #define FTN_SET_DEFAULT_ALLOCATOR OMP_SET_DEFAULT_ALLOCATOR_
 578: #define FTN_GET_DEFAULT_ALLOCATOR OMP_GET_DEFAULT_ALLOCATOR_
 579: #define FTN_GET_DEVICES_MEMSPACE OMP_GET_DEVICES_MEMSPACE_
 580: #define FTN_GET_DEVICE_MEMSPACE OMP_GET_DEVICE_MEMSPACE_
 581: #define FTN_GET_DEVICES_AND_HOST_MEMSPACE OMP_GET_DEVICES_AND_HOST_MEMSPACE_
 582: #define FTN_GET_DEVICE_AND_HOST_MEMSPACE OMP_GET_DEVICE_AND_HOST_MEMSPACE_
 583: #define FTN_GET_DEVICES_ALL_MEMSPACE OMP_GET_DEVICES_ALL_MEMSPACE_
 584: #define FTN_GET_DEVICES_ALLOCATOR OMP_GET_DEVICES_ALLOCATOR_
 585: #define FTN_GET_DEVICE_ALLOCATOR OMP_GET_DEVICE_ALLOCATOR_
 586: #define FTN_GET_DEVICES_AND_HOST_ALLOCATOR OMP_GET_DEVICES_AND_HOST_ALLOCATOR_
 587: #define FTN_GET_DEVICE_AND_HOST_ALLOCATOR OMP_GET_DEVICE_AND_HOST_ALLOCATOR_
 588: #define FTN_GET_DEVICES_ALL_ALLOCATOR OMP_GET_DEVICES_ALL_ALLOCATOR_
```

- **L565**: Defines macro \`FTN_TARGET_ALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_ALLOC\`，供条件编译或文本复用使用。
- **L566**: Defines macro \`FTN_TARGET_FREE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_FREE\`，供条件编译或文本复用使用。
- **L567**: Defines macro \`FTN_TARGET_IS_PRESENT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_IS_PRESENT\`，供条件编译或文本复用使用。
- **L568**: Defines macro \`FTN_TARGET_MEMCPY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_MEMCPY\`，供条件编译或文本复用使用。
- **L569**: Defines macro \`FTN_TARGET_MEMCPY_RECT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_MEMCPY_RECT\`，供条件编译或文本复用使用。
- **L570**: Defines macro \`FTN_TARGET_ASSOCIATE_PTR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_ASSOCIATE_PTR\`，供条件编译或文本复用使用。
- **L571**: Defines macro \`FTN_TARGET_DISASSOCIATE_PTR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_TARGET_DISASSOCIATE_PTR\`，供条件编译或文本复用使用。
- **L572**: Ends the active conditional-compilation region. / 结束当前条件编译区域。
- **L573**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L574**: Defines macro \`FTN_CONTROL_TOOL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CONTROL_TOOL\`，供条件编译或文本复用使用。
- **L575**: Defines macro \`FTN_INIT_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_INIT_ALLOCATOR\`，供条件编译或文本复用使用。
- **L576**: Defines macro \`FTN_DESTROY_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DESTROY_ALLOCATOR\`，供条件编译或文本复用使用。
- **L577**: Defines macro \`FTN_SET_DEFAULT_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_DEFAULT_ALLOCATOR\`，供条件编译或文本复用使用。
- **L578**: Defines macro \`FTN_GET_DEFAULT_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEFAULT_ALLOCATOR\`，供条件编译或文本复用使用。
- **L579**: Defines macro \`FTN_GET_DEVICES_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_MEMSPACE\`，供条件编译或文本复用使用。
- **L580**: Defines macro \`FTN_GET_DEVICE_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_MEMSPACE\`，供条件编译或文本复用使用。
- **L581**: Defines macro \`FTN_GET_DEVICES_AND_HOST_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_AND_HOST_MEMSPACE\`，供条件编译或文本复用使用。
- **L582**: Defines macro \`FTN_GET_DEVICE_AND_HOST_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_AND_HOST_MEMSPACE\`，供条件编译或文本复用使用。
- **L583**: Defines macro \`FTN_GET_DEVICES_ALL_MEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_ALL_MEMSPACE\`，供条件编译或文本复用使用。
- **L584**: Defines macro \`FTN_GET_DEVICES_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_ALLOCATOR\`，供条件编译或文本复用使用。
- **L585**: Defines macro \`FTN_GET_DEVICE_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_ALLOCATOR\`，供条件编译或文本复用使用。
- **L586**: Defines macro \`FTN_GET_DEVICES_AND_HOST_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_AND_HOST_ALLOCATOR\`，供条件编译或文本复用使用。
- **L587**: Defines macro \`FTN_GET_DEVICE_AND_HOST_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_AND_HOST_ALLOCATOR\`，供条件编译或文本复用使用。
- **L588**: Defines macro \`FTN_GET_DEVICES_ALL_ALLOCATOR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICES_ALL_ALLOCATOR\`，供条件编译或文本复用使用。

### Lines 589-610 / 第 589-610 行

```cpp
 589: #define FTN_GET_MEMSPACE_NUM_RESOURCES OMP_GET_MEMSPACE_NUM_RESOURCES_
 590: #define FTN_GET_SUBMEMSPACE OMP_GET_SUBMEMSPACE_
 591: #define FTN_ALLOC OMP_ALLOC_
 592: #define FTN_FREE OMP_FREE_
 593: #define FTN_GET_DEVICE_NUM OMP_GET_DEVICE_NUM_
 594: #define FTN_GET_UID_FROM_DEVICE OMP_GET_UID_FROM_DEVICE_
 595: #define FTN_GET_DEVICE_FROM_UID OMP_GET_DEVICE_FROM_UID_
 596: #define FTN_SET_AFFINITY_FORMAT OMP_SET_AFFINITY_FORMAT_
 597: #define FTN_GET_AFFINITY_FORMAT OMP_GET_AFFINITY_FORMAT_
 598: #define FTN_DISPLAY_AFFINITY OMP_DISPLAY_AFFINITY_
 599: #define FTN_CAPTURE_AFFINITY OMP_CAPTURE_AFFINITY_
 600: #define FTN_PAUSE_RESOURCE OMP_PAUSE_RESOURCE_
 601: #define FTN_PAUSE_RESOURCE_ALL OMP_PAUSE_RESOURCE_ALL_
 602: #define FTN_GET_SUPPORTED_ACTIVE_LEVELS OMP_GET_SUPPORTED_ACTIVE_LEVELS_
 603: #define FTN_DISPLAY_ENV OMP_DISPLAY_ENV_
 604: #define FTN_IN_EXPLICIT_TASK OMP_IN_EXPLICIT_TASK_
 605: #define FTN_FULFILL_EVENT OMP_FULFILL_EVENT_
 606: #define FTN_SET_NUM_TEAMS OMP_SET_NUM_TEAMS_
 607: #define FTN_GET_MAX_TEAMS OMP_GET_MAX_TEAMS_
 608: #define FTN_SET_TEAMS_THREAD_LIMIT OMP_SET_TEAMS_THREAD_LIMIT_
 609: #define FTN_GET_TEAMS_THREAD_LIMIT OMP_GET_TEAMS_THREAD_LIMIT_
 610: 
```

- **L589**: Defines macro \`FTN_GET_MEMSPACE_NUM_RESOURCES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MEMSPACE_NUM_RESOURCES\`，供条件编译或文本复用使用。
- **L590**: Defines macro \`FTN_GET_SUBMEMSPACE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_SUBMEMSPACE\`，供条件编译或文本复用使用。
- **L591**: Defines macro \`FTN_ALLOC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_ALLOC\`，供条件编译或文本复用使用。
- **L592**: Defines macro \`FTN_FREE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_FREE\`，供条件编译或文本复用使用。
- **L593**: Defines macro \`FTN_GET_DEVICE_NUM\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_NUM\`，供条件编译或文本复用使用。
- **L594**: Defines macro \`FTN_GET_UID_FROM_DEVICE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_UID_FROM_DEVICE\`，供条件编译或文本复用使用。
- **L595**: Defines macro \`FTN_GET_DEVICE_FROM_UID\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_DEVICE_FROM_UID\`，供条件编译或文本复用使用。
- **L596**: Defines macro \`FTN_SET_AFFINITY_FORMAT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_AFFINITY_FORMAT\`，供条件编译或文本复用使用。
- **L597**: Defines macro \`FTN_GET_AFFINITY_FORMAT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_AFFINITY_FORMAT\`，供条件编译或文本复用使用。
- **L598**: Defines macro \`FTN_DISPLAY_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DISPLAY_AFFINITY\`，供条件编译或文本复用使用。
- **L599**: Defines macro \`FTN_CAPTURE_AFFINITY\` for conditional compilation or textual reuse. / 定义宏 \`FTN_CAPTURE_AFFINITY\`，供条件编译或文本复用使用。
- **L600**: Defines macro \`FTN_PAUSE_RESOURCE\` for conditional compilation or textual reuse. / 定义宏 \`FTN_PAUSE_RESOURCE\`，供条件编译或文本复用使用。
- **L601**: Defines macro \`FTN_PAUSE_RESOURCE_ALL\` for conditional compilation or textual reuse. / 定义宏 \`FTN_PAUSE_RESOURCE_ALL\`，供条件编译或文本复用使用。
- **L602**: Defines macro \`FTN_GET_SUPPORTED_ACTIVE_LEVELS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_SUPPORTED_ACTIVE_LEVELS\`，供条件编译或文本复用使用。
- **L603**: Defines macro \`FTN_DISPLAY_ENV\` for conditional compilation or textual reuse. / 定义宏 \`FTN_DISPLAY_ENV\`，供条件编译或文本复用使用。
- **L604**: Defines macro \`FTN_IN_EXPLICIT_TASK\` for conditional compilation or textual reuse. / 定义宏 \`FTN_IN_EXPLICIT_TASK\`，供条件编译或文本复用使用。
- **L605**: Defines macro \`FTN_FULFILL_EVENT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_FULFILL_EVENT\`，供条件编译或文本复用使用。
- **L606**: Defines macro \`FTN_SET_NUM_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_NUM_TEAMS\`，供条件编译或文本复用使用。
- **L607**: Defines macro \`FTN_GET_MAX_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_MAX_TEAMS\`，供条件编译或文本复用使用。
- **L608**: Defines macro \`FTN_SET_TEAMS_THREAD_LIMIT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_SET_TEAMS_THREAD_LIMIT\`，供条件编译或文本复用使用。
- **L609**: Defines macro \`FTN_GET_TEAMS_THREAD_LIMIT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_TEAMS_THREAD_LIMIT\`，供条件编译或文本复用使用。
- **L610**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 611-634 / 第 611-634 行

```cpp
 611: #define FTN_GET_NUM_INTEROP_PROPERTIES OMP_GET_NUM_INTEROP_PROPERTIES_
 612: #define FTN_GET_INTEROP_INT OMP_GET_INTEROP_INT_
 613: #define FTN_GET_INTEROP_PTR OMP_GET_INTEROP_PTR_
 614: #define FTN_GET_INTEROP_STR OMP_GET_INTEROP_STR_
 615: #define FTN_GET_INTEROP_NAME OMP_GET_INTEROP_NAME_
 616: #define FTN_GET_INTEROP_TYPE_DESC OMP_GET_INTEROP_TYPE_DESC_
 617: #define FTN_GET_INTEROP_RC_DESC OMP_GET_INTEROP_RC_DESC_
 618: 
 619: #endif /* KMP_FTN_UAPPEND */
 620: 
 621: /* -------------------------- GOMP API NAMES ------------------------ */
 622: // All GOMP_1.0 symbols
 623: #define KMP_API_NAME_GOMP_ATOMIC_END GOMP_atomic_end
 624: #define KMP_API_NAME_GOMP_ATOMIC_START GOMP_atomic_start
 625: #define KMP_API_NAME_GOMP_BARRIER GOMP_barrier
 626: #define KMP_API_NAME_GOMP_CRITICAL_END GOMP_critical_end
 627: #define KMP_API_NAME_GOMP_CRITICAL_NAME_END GOMP_critical_name_end
 628: #define KMP_API_NAME_GOMP_CRITICAL_NAME_START GOMP_critical_name_start
 629: #define KMP_API_NAME_GOMP_CRITICAL_START GOMP_critical_start
 630: #define KMP_API_NAME_GOMP_LOOP_DYNAMIC_NEXT GOMP_loop_dynamic_next
 631: #define KMP_API_NAME_GOMP_LOOP_DYNAMIC_START GOMP_loop_dynamic_start
 632: #define KMP_API_NAME_GOMP_LOOP_END GOMP_loop_end
 633: #define KMP_API_NAME_GOMP_LOOP_END_NOWAIT GOMP_loop_end_nowait
 634: #define KMP_API_NAME_GOMP_LOOP_GUIDED_NEXT GOMP_loop_guided_next
```

- **L611**: Defines macro \`FTN_GET_NUM_INTEROP_PROPERTIES\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_NUM_INTEROP_PROPERTIES\`，供条件编译或文本复用使用。
- **L612**: Defines macro \`FTN_GET_INTEROP_INT\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_INT\`，供条件编译或文本复用使用。
- **L613**: Defines macro \`FTN_GET_INTEROP_PTR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_PTR\`，供条件编译或文本复用使用。
- **L614**: Defines macro \`FTN_GET_INTEROP_STR\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_STR\`，供条件编译或文本复用使用。
- **L615**: Defines macro \`FTN_GET_INTEROP_NAME\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_NAME\`，供条件编译或文本复用使用。
- **L616**: Defines macro \`FTN_GET_INTEROP_TYPE_DESC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_TYPE_DESC\`，供条件编译或文本复用使用。
- **L617**: Defines macro \`FTN_GET_INTEROP_RC_DESC\` for conditional compilation or textual reuse. / 定义宏 \`FTN_GET_INTEROP_RC_DESC\`，供条件编译或文本复用使用。
- **L618**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L619**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L620**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L621**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L622**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L623**: Defines macro \`KMP_API_NAME_GOMP_ATOMIC_END\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_ATOMIC_END\`，供条件编译或文本复用使用。
- **L624**: Defines macro \`KMP_API_NAME_GOMP_ATOMIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_ATOMIC_START\`，供条件编译或文本复用使用。
- **L625**: Defines macro \`KMP_API_NAME_GOMP_BARRIER\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_BARRIER\`，供条件编译或文本复用使用。
- **L626**: Defines macro \`KMP_API_NAME_GOMP_CRITICAL_END\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_CRITICAL_END\`，供条件编译或文本复用使用。
- **L627**: Defines macro \`KMP_API_NAME_GOMP_CRITICAL_NAME_END\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_CRITICAL_NAME_END\`，供条件编译或文本复用使用。
- **L628**: Defines macro \`KMP_API_NAME_GOMP_CRITICAL_NAME_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_CRITICAL_NAME_START\`，供条件编译或文本复用使用。
- **L629**: Defines macro \`KMP_API_NAME_GOMP_CRITICAL_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_CRITICAL_START\`，供条件编译或文本复用使用。
- **L630**: Defines macro \`KMP_API_NAME_GOMP_LOOP_DYNAMIC_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_DYNAMIC_NEXT\`，供条件编译或文本复用使用。
- **L631**: Defines macro \`KMP_API_NAME_GOMP_LOOP_DYNAMIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_DYNAMIC_START\`，供条件编译或文本复用使用。
- **L632**: Defines macro \`KMP_API_NAME_GOMP_LOOP_END\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_END\`，供条件编译或文本复用使用。
- **L633**: Defines macro \`KMP_API_NAME_GOMP_LOOP_END_NOWAIT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_END_NOWAIT\`，供条件编译或文本复用使用。
- **L634**: Defines macro \`KMP_API_NAME_GOMP_LOOP_GUIDED_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_GUIDED_NEXT\`，供条件编译或文本复用使用。

### Lines 635-658 / 第 635-658 行

```cpp
 635: #define KMP_API_NAME_GOMP_LOOP_GUIDED_START GOMP_loop_guided_start
 636: #define KMP_API_NAME_GOMP_LOOP_ORDERED_DYNAMIC_NEXT                            \
 637:   GOMP_loop_ordered_dynamic_next
 638: #define KMP_API_NAME_GOMP_LOOP_ORDERED_DYNAMIC_START                           \
 639:   GOMP_loop_ordered_dynamic_start
 640: #define KMP_API_NAME_GOMP_LOOP_ORDERED_GUIDED_NEXT GOMP_loop_ordered_guided_next
 641: #define KMP_API_NAME_GOMP_LOOP_ORDERED_GUIDED_START                            \
 642:   GOMP_loop_ordered_guided_start
 643: #define KMP_API_NAME_GOMP_LOOP_ORDERED_RUNTIME_NEXT                            \
 644:   GOMP_loop_ordered_runtime_next
 645: #define KMP_API_NAME_GOMP_LOOP_ORDERED_RUNTIME_START                           \
 646:   GOMP_loop_ordered_runtime_start
 647: #define KMP_API_NAME_GOMP_LOOP_ORDERED_STATIC_NEXT GOMP_loop_ordered_static_next
 648: #define KMP_API_NAME_GOMP_LOOP_ORDERED_STATIC_START                            \
 649:   GOMP_loop_ordered_static_start
 650: #define KMP_API_NAME_GOMP_LOOP_RUNTIME_NEXT GOMP_loop_runtime_next
 651: #define KMP_API_NAME_GOMP_LOOP_RUNTIME_START GOMP_loop_runtime_start
 652: #define KMP_API_NAME_GOMP_LOOP_STATIC_NEXT GOMP_loop_static_next
 653: #define KMP_API_NAME_GOMP_LOOP_STATIC_START GOMP_loop_static_start
 654: #define KMP_API_NAME_GOMP_ORDERED_END GOMP_ordered_end
 655: #define KMP_API_NAME_GOMP_ORDERED_START GOMP_ordered_start
 656: #define KMP_API_NAME_GOMP_PARALLEL_END GOMP_parallel_end
 657: #define KMP_API_NAME_GOMP_PARALLEL_LOOP_DYNAMIC_START                          \
 658:   GOMP_parallel_loop_dynamic_start
```

- **L635**: Defines macro \`KMP_API_NAME_GOMP_LOOP_GUIDED_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_GUIDED_START\`，供条件编译或文本复用使用。
- **L636**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ORDERED_DYNAMIC_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ORDERED_DYNAMIC_NEXT\`，供条件编译或文本复用使用。
- **L637**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L638**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ORDERED_DYNAMIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ORDERED_DYNAMIC_START\`，供条件编译或文本复用使用。
- **L639**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L640**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ORDERED_GUIDED_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ORDERED_GUIDED_NEXT\`，供条件编译或文本复用使用。
- **L641**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ORDERED_GUIDED_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ORDERED_GUIDED_START\`，供条件编译或文本复用使用。
- **L642**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L643**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ORDERED_RUNTIME_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ORDERED_RUNTIME_NEXT\`，供条件编译或文本复用使用。
- **L644**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L645**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ORDERED_RUNTIME_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ORDERED_RUNTIME_START\`，供条件编译或文本复用使用。
- **L646**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L647**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ORDERED_STATIC_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ORDERED_STATIC_NEXT\`，供条件编译或文本复用使用。
- **L648**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ORDERED_STATIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ORDERED_STATIC_START\`，供条件编译或文本复用使用。
- **L649**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L650**: Defines macro \`KMP_API_NAME_GOMP_LOOP_RUNTIME_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_RUNTIME_NEXT\`，供条件编译或文本复用使用。
- **L651**: Defines macro \`KMP_API_NAME_GOMP_LOOP_RUNTIME_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_RUNTIME_START\`，供条件编译或文本复用使用。
- **L652**: Defines macro \`KMP_API_NAME_GOMP_LOOP_STATIC_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_STATIC_NEXT\`，供条件编译或文本复用使用。
- **L653**: Defines macro \`KMP_API_NAME_GOMP_LOOP_STATIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_STATIC_START\`，供条件编译或文本复用使用。
- **L654**: Defines macro \`KMP_API_NAME_GOMP_ORDERED_END\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_ORDERED_END\`，供条件编译或文本复用使用。
- **L655**: Defines macro \`KMP_API_NAME_GOMP_ORDERED_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_ORDERED_START\`，供条件编译或文本复用使用。
- **L656**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_END\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_END\`，供条件编译或文本复用使用。
- **L657**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_LOOP_DYNAMIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_LOOP_DYNAMIC_START\`，供条件编译或文本复用使用。
- **L658**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。

### Lines 659-674 / 第 659-674 行

```cpp
 659: #define KMP_API_NAME_GOMP_PARALLEL_LOOP_GUIDED_START                           \
 660:   GOMP_parallel_loop_guided_start
 661: #define KMP_API_NAME_GOMP_PARALLEL_LOOP_RUNTIME_START                          \
 662:   GOMP_parallel_loop_runtime_start
 663: #define KMP_API_NAME_GOMP_PARALLEL_LOOP_STATIC_START                           \
 664:   GOMP_parallel_loop_static_start
 665: #define KMP_API_NAME_GOMP_PARALLEL_SECTIONS_START GOMP_parallel_sections_start
 666: #define KMP_API_NAME_GOMP_PARALLEL_START GOMP_parallel_start
 667: #define KMP_API_NAME_GOMP_SECTIONS_END GOMP_sections_end
 668: #define KMP_API_NAME_GOMP_SECTIONS_END_NOWAIT GOMP_sections_end_nowait
 669: #define KMP_API_NAME_GOMP_SECTIONS_NEXT GOMP_sections_next
 670: #define KMP_API_NAME_GOMP_SECTIONS_START GOMP_sections_start
 671: #define KMP_API_NAME_GOMP_SINGLE_COPY_END GOMP_single_copy_end
 672: #define KMP_API_NAME_GOMP_SINGLE_COPY_START GOMP_single_copy_start
 673: #define KMP_API_NAME_GOMP_SINGLE_START GOMP_single_start
 674: 
```

- **L659**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_LOOP_GUIDED_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_LOOP_GUIDED_START\`，供条件编译或文本复用使用。
- **L660**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L661**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_LOOP_RUNTIME_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_LOOP_RUNTIME_START\`，供条件编译或文本复用使用。
- **L662**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L663**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_LOOP_STATIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_LOOP_STATIC_START\`，供条件编译或文本复用使用。
- **L664**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L665**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_SECTIONS_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_SECTIONS_START\`，供条件编译或文本复用使用。
- **L666**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_START\`，供条件编译或文本复用使用。
- **L667**: Defines macro \`KMP_API_NAME_GOMP_SECTIONS_END\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_SECTIONS_END\`，供条件编译或文本复用使用。
- **L668**: Defines macro \`KMP_API_NAME_GOMP_SECTIONS_END_NOWAIT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_SECTIONS_END_NOWAIT\`，供条件编译或文本复用使用。
- **L669**: Defines macro \`KMP_API_NAME_GOMP_SECTIONS_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_SECTIONS_NEXT\`，供条件编译或文本复用使用。
- **L670**: Defines macro \`KMP_API_NAME_GOMP_SECTIONS_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_SECTIONS_START\`，供条件编译或文本复用使用。
- **L671**: Defines macro \`KMP_API_NAME_GOMP_SINGLE_COPY_END\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_SINGLE_COPY_END\`，供条件编译或文本复用使用。
- **L672**: Defines macro \`KMP_API_NAME_GOMP_SINGLE_COPY_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_SINGLE_COPY_START\`，供条件编译或文本复用使用。
- **L673**: Defines macro \`KMP_API_NAME_GOMP_SINGLE_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_SINGLE_START\`，供条件编译或文本复用使用。
- **L674**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 675-698 / 第 675-698 行

```cpp
 675: // All GOMP_2.0 symbols
 676: #define KMP_API_NAME_GOMP_TASK GOMP_task
 677: #define KMP_API_NAME_GOMP_TASKWAIT GOMP_taskwait
 678: #define KMP_API_NAME_GOMP_LOOP_ULL_DYNAMIC_NEXT GOMP_loop_ull_dynamic_next
 679: #define KMP_API_NAME_GOMP_LOOP_ULL_DYNAMIC_START GOMP_loop_ull_dynamic_start
 680: #define KMP_API_NAME_GOMP_LOOP_ULL_GUIDED_NEXT GOMP_loop_ull_guided_next
 681: #define KMP_API_NAME_GOMP_LOOP_ULL_GUIDED_START GOMP_loop_ull_guided_start
 682: #define KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_DYNAMIC_NEXT                        \
 683:   GOMP_loop_ull_ordered_dynamic_next
 684: #define KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_DYNAMIC_START                       \
 685:   GOMP_loop_ull_ordered_dynamic_start
 686: #define KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_GUIDED_NEXT                         \
 687:   GOMP_loop_ull_ordered_guided_next
 688: #define KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_GUIDED_START                        \
 689:   GOMP_loop_ull_ordered_guided_start
 690: #define KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_RUNTIME_NEXT                        \
 691:   GOMP_loop_ull_ordered_runtime_next
 692: #define KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_RUNTIME_START                       \
 693:   GOMP_loop_ull_ordered_runtime_start
 694: #define KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_STATIC_NEXT                         \
 695:   GOMP_loop_ull_ordered_static_next
 696: #define KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_STATIC_START                        \
 697:   GOMP_loop_ull_ordered_static_start
 698: #define KMP_API_NAME_GOMP_LOOP_ULL_RUNTIME_NEXT GOMP_loop_ull_runtime_next
```

- **L675**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L676**: Defines macro \`KMP_API_NAME_GOMP_TASK\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TASK\`，供条件编译或文本复用使用。
- **L677**: Defines macro \`KMP_API_NAME_GOMP_TASKWAIT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TASKWAIT\`，供条件编译或文本复用使用。
- **L678**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_DYNAMIC_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_DYNAMIC_NEXT\`，供条件编译或文本复用使用。
- **L679**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_DYNAMIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_DYNAMIC_START\`，供条件编译或文本复用使用。
- **L680**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_GUIDED_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_GUIDED_NEXT\`，供条件编译或文本复用使用。
- **L681**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_GUIDED_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_GUIDED_START\`，供条件编译或文本复用使用。
- **L682**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_DYNAMIC_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_DYNAMIC_NEXT\`，供条件编译或文本复用使用。
- **L683**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L684**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_DYNAMIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_DYNAMIC_START\`，供条件编译或文本复用使用。
- **L685**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L686**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_GUIDED_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_GUIDED_NEXT\`，供条件编译或文本复用使用。
- **L687**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L688**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_GUIDED_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_GUIDED_START\`，供条件编译或文本复用使用。
- **L689**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L690**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_RUNTIME_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_RUNTIME_NEXT\`，供条件编译或文本复用使用。
- **L691**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L692**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_RUNTIME_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_RUNTIME_START\`，供条件编译或文本复用使用。
- **L693**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L694**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_STATIC_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_STATIC_NEXT\`，供条件编译或文本复用使用。
- **L695**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L696**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_STATIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_STATIC_START\`，供条件编译或文本复用使用。
- **L697**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L698**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_RUNTIME_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_RUNTIME_NEXT\`，供条件编译或文本复用使用。

### Lines 699-722 / 第 699-722 行

```cpp
 699: #define KMP_API_NAME_GOMP_LOOP_ULL_RUNTIME_START GOMP_loop_ull_runtime_start
 700: #define KMP_API_NAME_GOMP_LOOP_ULL_STATIC_NEXT GOMP_loop_ull_static_next
 701: #define KMP_API_NAME_GOMP_LOOP_ULL_STATIC_START GOMP_loop_ull_static_start
 702: 
 703: // All GOMP_3.0 symbols
 704: #define KMP_API_NAME_GOMP_TASKYIELD GOMP_taskyield
 705: 
 706: // All GOMP_4.0 symbols
 707: #define KMP_API_NAME_GOMP_BARRIER_CANCEL GOMP_barrier_cancel
 708: #define KMP_API_NAME_GOMP_CANCEL GOMP_cancel
 709: #define KMP_API_NAME_GOMP_CANCELLATION_POINT GOMP_cancellation_point
 710: #define KMP_API_NAME_GOMP_LOOP_END_CANCEL GOMP_loop_end_cancel
 711: #define KMP_API_NAME_GOMP_PARALLEL_LOOP_DYNAMIC GOMP_parallel_loop_dynamic
 712: #define KMP_API_NAME_GOMP_PARALLEL_LOOP_GUIDED GOMP_parallel_loop_guided
 713: #define KMP_API_NAME_GOMP_PARALLEL_LOOP_RUNTIME GOMP_parallel_loop_runtime
 714: #define KMP_API_NAME_GOMP_PARALLEL_LOOP_STATIC GOMP_parallel_loop_static
 715: #define KMP_API_NAME_GOMP_PARALLEL_SECTIONS GOMP_parallel_sections
 716: #define KMP_API_NAME_GOMP_PARALLEL GOMP_parallel
 717: #define KMP_API_NAME_GOMP_SECTIONS_END_CANCEL GOMP_sections_end_cancel
 718: #define KMP_API_NAME_GOMP_TASKGROUP_START GOMP_taskgroup_start
 719: #define KMP_API_NAME_GOMP_TASKGROUP_END GOMP_taskgroup_end
 720: /* Target functions should be taken care of by liboffload */
 721: #define KMP_API_NAME_GOMP_TARGET GOMP_target
 722: #define KMP_API_NAME_GOMP_TARGET_DATA GOMP_target_data
```

- **L699**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_RUNTIME_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_RUNTIME_START\`，供条件编译或文本复用使用。
- **L700**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_STATIC_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_STATIC_NEXT\`，供条件编译或文本复用使用。
- **L701**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_STATIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_STATIC_START\`，供条件编译或文本复用使用。
- **L702**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L703**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L704**: Defines macro \`KMP_API_NAME_GOMP_TASKYIELD\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TASKYIELD\`，供条件编译或文本复用使用。
- **L705**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L706**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L707**: Defines macro \`KMP_API_NAME_GOMP_BARRIER_CANCEL\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_BARRIER_CANCEL\`，供条件编译或文本复用使用。
- **L708**: Defines macro \`KMP_API_NAME_GOMP_CANCEL\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_CANCEL\`，供条件编译或文本复用使用。
- **L709**: Defines macro \`KMP_API_NAME_GOMP_CANCELLATION_POINT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_CANCELLATION_POINT\`，供条件编译或文本复用使用。
- **L710**: Defines macro \`KMP_API_NAME_GOMP_LOOP_END_CANCEL\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_END_CANCEL\`，供条件编译或文本复用使用。
- **L711**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_LOOP_DYNAMIC\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_LOOP_DYNAMIC\`，供条件编译或文本复用使用。
- **L712**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_LOOP_GUIDED\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_LOOP_GUIDED\`，供条件编译或文本复用使用。
- **L713**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_LOOP_RUNTIME\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_LOOP_RUNTIME\`，供条件编译或文本复用使用。
- **L714**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_LOOP_STATIC\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_LOOP_STATIC\`，供条件编译或文本复用使用。
- **L715**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_SECTIONS\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_SECTIONS\`，供条件编译或文本复用使用。
- **L716**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL\`，供条件编译或文本复用使用。
- **L717**: Defines macro \`KMP_API_NAME_GOMP_SECTIONS_END_CANCEL\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_SECTIONS_END_CANCEL\`，供条件编译或文本复用使用。
- **L718**: Defines macro \`KMP_API_NAME_GOMP_TASKGROUP_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TASKGROUP_START\`，供条件编译或文本复用使用。
- **L719**: Defines macro \`KMP_API_NAME_GOMP_TASKGROUP_END\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TASKGROUP_END\`，供条件编译或文本复用使用。
- **L720**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L721**: Defines macro \`KMP_API_NAME_GOMP_TARGET\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TARGET\`，供条件编译或文本复用使用。
- **L722**: Defines macro \`KMP_API_NAME_GOMP_TARGET_DATA\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TARGET_DATA\`，供条件编译或文本复用使用。

### Lines 723-746 / 第 723-746 行

```cpp
 723: #define KMP_API_NAME_GOMP_TARGET_END_DATA GOMP_target_end_data
 724: #define KMP_API_NAME_GOMP_TARGET_UPDATE GOMP_target_update
 725: #define KMP_API_NAME_GOMP_TEAMS GOMP_teams
 726: 
 727: // All GOMP_4.5 symbols
 728: #define KMP_API_NAME_GOMP_TASKLOOP GOMP_taskloop
 729: #define KMP_API_NAME_GOMP_TASKLOOP_ULL GOMP_taskloop_ull
 730: #define KMP_API_NAME_GOMP_DOACROSS_POST GOMP_doacross_post
 731: #define KMP_API_NAME_GOMP_DOACROSS_WAIT GOMP_doacross_wait
 732: #define KMP_API_NAME_GOMP_LOOP_DOACROSS_STATIC_START                           \
 733:   GOMP_loop_doacross_static_start
 734: #define KMP_API_NAME_GOMP_LOOP_DOACROSS_DYNAMIC_START                          \
 735:   GOMP_loop_doacross_dynamic_start
 736: #define KMP_API_NAME_GOMP_LOOP_DOACROSS_GUIDED_START                           \
 737:   GOMP_loop_doacross_guided_start
 738: #define KMP_API_NAME_GOMP_LOOP_DOACROSS_RUNTIME_START                          \
 739:   GOMP_loop_doacross_runtime_start
 740: #define KMP_API_NAME_GOMP_DOACROSS_ULL_POST GOMP_doacross_ull_post
 741: #define KMP_API_NAME_GOMP_DOACROSS_ULL_WAIT GOMP_doacross_ull_wait
 742: #define KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_STATIC_START                       \
 743:   GOMP_loop_ull_doacross_static_start
 744: #define KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_DYNAMIC_START                      \
 745:   GOMP_loop_ull_doacross_dynamic_start
 746: #define KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_GUIDED_START                       \
```

- **L723**: Defines macro \`KMP_API_NAME_GOMP_TARGET_END_DATA\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TARGET_END_DATA\`，供条件编译或文本复用使用。
- **L724**: Defines macro \`KMP_API_NAME_GOMP_TARGET_UPDATE\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TARGET_UPDATE\`，供条件编译或文本复用使用。
- **L725**: Defines macro \`KMP_API_NAME_GOMP_TEAMS\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TEAMS\`，供条件编译或文本复用使用。
- **L726**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。
- **L727**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L728**: Defines macro \`KMP_API_NAME_GOMP_TASKLOOP\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TASKLOOP\`，供条件编译或文本复用使用。
- **L729**: Defines macro \`KMP_API_NAME_GOMP_TASKLOOP_ULL\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TASKLOOP_ULL\`，供条件编译或文本复用使用。
- **L730**: Defines macro \`KMP_API_NAME_GOMP_DOACROSS_POST\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_DOACROSS_POST\`，供条件编译或文本复用使用。
- **L731**: Defines macro \`KMP_API_NAME_GOMP_DOACROSS_WAIT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_DOACROSS_WAIT\`，供条件编译或文本复用使用。
- **L732**: Defines macro \`KMP_API_NAME_GOMP_LOOP_DOACROSS_STATIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_DOACROSS_STATIC_START\`，供条件编译或文本复用使用。
- **L733**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L734**: Defines macro \`KMP_API_NAME_GOMP_LOOP_DOACROSS_DYNAMIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_DOACROSS_DYNAMIC_START\`，供条件编译或文本复用使用。
- **L735**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L736**: Defines macro \`KMP_API_NAME_GOMP_LOOP_DOACROSS_GUIDED_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_DOACROSS_GUIDED_START\`，供条件编译或文本复用使用。
- **L737**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L738**: Defines macro \`KMP_API_NAME_GOMP_LOOP_DOACROSS_RUNTIME_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_DOACROSS_RUNTIME_START\`，供条件编译或文本复用使用。
- **L739**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L740**: Defines macro \`KMP_API_NAME_GOMP_DOACROSS_ULL_POST\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_DOACROSS_ULL_POST\`，供条件编译或文本复用使用。
- **L741**: Defines macro \`KMP_API_NAME_GOMP_DOACROSS_ULL_WAIT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_DOACROSS_ULL_WAIT\`，供条件编译或文本复用使用。
- **L742**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_STATIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_STATIC_START\`，供条件编译或文本复用使用。
- **L743**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L744**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_DYNAMIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_DYNAMIC_START\`，供条件编译或文本复用使用。
- **L745**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L746**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_GUIDED_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_GUIDED_START\`，供条件编译或文本复用使用。

### Lines 747-770 / 第 747-770 行

```cpp
 747:   GOMP_loop_ull_doacross_guided_start
 748: #define KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_RUNTIME_START                      \
 749:   GOMP_loop_ull_doacross_runtime_start
 750: #define KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_DYNAMIC_NEXT                       \
 751:   GOMP_loop_nonmonotonic_dynamic_next
 752: #define KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_DYNAMIC_START                      \
 753:   GOMP_loop_nonmonotonic_dynamic_start
 754: #define KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_GUIDED_NEXT                        \
 755:   GOMP_loop_nonmonotonic_guided_next
 756: #define KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_GUIDED_START                       \
 757:   GOMP_loop_nonmonotonic_guided_start
 758: #define KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_DYNAMIC_NEXT                   \
 759:   GOMP_loop_ull_nonmonotonic_dynamic_next
 760: #define KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_DYNAMIC_START                  \
 761:   GOMP_loop_ull_nonmonotonic_dynamic_start
 762: #define KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_GUIDED_NEXT                    \
 763:   GOMP_loop_ull_nonmonotonic_guided_next
 764: #define KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_GUIDED_START                   \
 765:   GOMP_loop_ull_nonmonotonic_guided_start
 766: #define KMP_API_NAME_GOMP_PARALLEL_LOOP_NONMONOTONIC_DYNAMIC                   \
 767:   GOMP_parallel_loop_nonmonotonic_dynamic
 768: #define KMP_API_NAME_GOMP_PARALLEL_LOOP_NONMONOTONIC_GUIDED                    \
 769:   GOMP_parallel_loop_nonmonotonic_guided
 770: 
```

- **L747**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L748**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_RUNTIME_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_RUNTIME_START\`，供条件编译或文本复用使用。
- **L749**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L750**: Defines macro \`KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_DYNAMIC_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_DYNAMIC_NEXT\`，供条件编译或文本复用使用。
- **L751**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L752**: Defines macro \`KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_DYNAMIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_DYNAMIC_START\`，供条件编译或文本复用使用。
- **L753**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L754**: Defines macro \`KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_GUIDED_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_GUIDED_NEXT\`，供条件编译或文本复用使用。
- **L755**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L756**: Defines macro \`KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_GUIDED_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_GUIDED_START\`，供条件编译或文本复用使用。
- **L757**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L758**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_DYNAMIC_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_DYNAMIC_NEXT\`，供条件编译或文本复用使用。
- **L759**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L760**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_DYNAMIC_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_DYNAMIC_START\`，供条件编译或文本复用使用。
- **L761**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L762**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_GUIDED_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_GUIDED_NEXT\`，供条件编译或文本复用使用。
- **L763**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L764**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_GUIDED_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_GUIDED_START\`，供条件编译或文本复用使用。
- **L765**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L766**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_LOOP_NONMONOTONIC_DYNAMIC\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_LOOP_NONMONOTONIC_DYNAMIC\`，供条件编译或文本复用使用。
- **L767**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L768**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_LOOP_NONMONOTONIC_GUIDED\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_LOOP_NONMONOTONIC_GUIDED\`，供条件编译或文本复用使用。
- **L769**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L770**: Blank line separates nearby logical blocks. / 空行用于分隔相邻的逻辑块。

### Lines 771-794 / 第 771-794 行

```cpp
 771: // All GOMP_5.0 symbols
 772: #define KMP_API_NAME_GOMP_LOOP_MAYBE_NONMONOTONIC_RUNTIME_NEXT                 \
 773:   GOMP_loop_maybe_nonmonotonic_runtime_next
 774: #define KMP_API_NAME_GOMP_LOOP_MAYBE_NONMONOTONIC_RUNTIME_START                \
 775:   GOMP_loop_maybe_nonmonotonic_runtime_start
 776: #define KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_RUNTIME_NEXT                       \
 777:   GOMP_loop_nonmonotonic_runtime_next
 778: #define KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_RUNTIME_START                      \
 779:   GOMP_loop_nonmonotonic_runtime_start
 780: #define KMP_API_NAME_GOMP_LOOP_ULL_MAYBE_NONMONOTONIC_RUNTIME_NEXT             \
 781:   GOMP_loop_ull_maybe_nonmonotonic_runtime_next
 782: #define KMP_API_NAME_GOMP_LOOP_ULL_MAYBE_NONMONOTONIC_RUNTIME_START            \
 783:   GOMP_loop_ull_maybe_nonmonotonic_runtime_start
 784: #define KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_RUNTIME_NEXT                   \
 785:   GOMP_loop_ull_nonmonotonic_runtime_next
 786: #define KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_RUNTIME_START                  \
 787:   GOMP_loop_ull_nonmonotonic_runtime_start
 788: #define KMP_API_NAME_GOMP_PARALLEL_LOOP_NONMONOTONIC_RUNTIME                   \
 789:   GOMP_parallel_loop_nonmonotonic_runtime
 790: #define KMP_API_NAME_GOMP_PARALLEL_LOOP_MAYBE_NONMONOTONIC_RUNTIME             \
 791:   GOMP_parallel_loop_maybe_nonmonotonic_runtime
 792: #define KMP_API_NAME_GOMP_TEAMS_REG GOMP_teams_reg
 793: #define KMP_API_NAME_GOMP_TASKWAIT_DEPEND GOMP_taskwait_depend
 794: #define KMP_API_NAME_GOMP_TASKGROUP_REDUCTION_REGISTER                         \
```

- **L771**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。
- **L772**: Defines macro \`KMP_API_NAME_GOMP_LOOP_MAYBE_NONMONOTONIC_RUNTIME_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_MAYBE_NONMONOTONIC_RUNTIME_NEXT\`，供条件编译或文本复用使用。
- **L773**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L774**: Defines macro \`KMP_API_NAME_GOMP_LOOP_MAYBE_NONMONOTONIC_RUNTIME_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_MAYBE_NONMONOTONIC_RUNTIME_START\`，供条件编译或文本复用使用。
- **L775**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L776**: Defines macro \`KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_RUNTIME_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_RUNTIME_NEXT\`，供条件编译或文本复用使用。
- **L777**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L778**: Defines macro \`KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_RUNTIME_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_NONMONOTONIC_RUNTIME_START\`，供条件编译或文本复用使用。
- **L779**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L780**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_MAYBE_NONMONOTONIC_RUNTIME_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_MAYBE_NONMONOTONIC_RUNTIME_NEXT\`，供条件编译或文本复用使用。
- **L781**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L782**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_MAYBE_NONMONOTONIC_RUNTIME_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_MAYBE_NONMONOTONIC_RUNTIME_START\`，供条件编译或文本复用使用。
- **L783**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L784**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_RUNTIME_NEXT\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_RUNTIME_NEXT\`，供条件编译或文本复用使用。
- **L785**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L786**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_RUNTIME_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_NONMONOTONIC_RUNTIME_START\`，供条件编译或文本复用使用。
- **L787**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L788**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_LOOP_NONMONOTONIC_RUNTIME\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_LOOP_NONMONOTONIC_RUNTIME\`，供条件编译或文本复用使用。
- **L789**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L790**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_LOOP_MAYBE_NONMONOTONIC_RUNTIME\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_LOOP_MAYBE_NONMONOTONIC_RUNTIME\`，供条件编译或文本复用使用。
- **L791**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L792**: Defines macro \`KMP_API_NAME_GOMP_TEAMS_REG\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TEAMS_REG\`，供条件编译或文本复用使用。
- **L793**: Defines macro \`KMP_API_NAME_GOMP_TASKWAIT_DEPEND\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TASKWAIT_DEPEND\`，供条件编译或文本复用使用。
- **L794**: Defines macro \`KMP_API_NAME_GOMP_TASKGROUP_REDUCTION_REGISTER\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TASKGROUP_REDUCTION_REGISTER\`，供条件编译或文本复用使用。

### Lines 795-811 / 第 795-811 行

```cpp
 795:   GOMP_taskgroup_reduction_register
 796: #define KMP_API_NAME_GOMP_TASKGROUP_REDUCTION_UNREGISTER                       \
 797:   GOMP_taskgroup_reduction_unregister
 798: #define KMP_API_NAME_GOMP_TASK_REDUCTION_REMAP GOMP_task_reduction_remap
 799: #define KMP_API_NAME_GOMP_PARALLEL_REDUCTIONS GOMP_parallel_reductions
 800: #define KMP_API_NAME_GOMP_LOOP_START GOMP_loop_start
 801: #define KMP_API_NAME_GOMP_LOOP_ULL_START GOMP_loop_ull_start
 802: #define KMP_API_NAME_GOMP_LOOP_DOACROSS_START GOMP_loop_doacross_start
 803: #define KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_START GOMP_loop_ull_doacross_start
 804: #define KMP_API_NAME_GOMP_LOOP_ORDERED_START GOMP_loop_ordered_start
 805: #define KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_START GOMP_loop_ull_ordered_start
 806: #define KMP_API_NAME_GOMP_SECTIONS2_START GOMP_sections2_start
 807: #define KMP_API_NAME_GOMP_WORKSHARE_TASK_REDUCTION_UNREGISTER                  \
 808:   GOMP_workshare_task_reduction_unregister
 809: #define KMP_API_NAME_GOMP_ALLOC GOMP_alloc
 810: #define KMP_API_NAME_GOMP_FREE GOMP_free
 811: #endif /* KMP_FTN_OS_H */
```

- **L795**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L796**: Defines macro \`KMP_API_NAME_GOMP_TASKGROUP_REDUCTION_UNREGISTER\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TASKGROUP_REDUCTION_UNREGISTER\`，供条件编译或文本复用使用。
- **L797**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L798**: Defines macro \`KMP_API_NAME_GOMP_TASK_REDUCTION_REMAP\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_TASK_REDUCTION_REMAP\`，供条件编译或文本复用使用。
- **L799**: Defines macro \`KMP_API_NAME_GOMP_PARALLEL_REDUCTIONS\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_PARALLEL_REDUCTIONS\`，供条件编译或文本复用使用。
- **L800**: Defines macro \`KMP_API_NAME_GOMP_LOOP_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_START\`，供条件编译或文本复用使用。
- **L801**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_START\`，供条件编译或文本复用使用。
- **L802**: Defines macro \`KMP_API_NAME_GOMP_LOOP_DOACROSS_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_DOACROSS_START\`，供条件编译或文本复用使用。
- **L803**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_DOACROSS_START\`，供条件编译或文本复用使用。
- **L804**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ORDERED_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ORDERED_START\`，供条件编译或文本复用使用。
- **L805**: Defines macro \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_LOOP_ULL_ORDERED_START\`，供条件编译或文本复用使用。
- **L806**: Defines macro \`KMP_API_NAME_GOMP_SECTIONS2_START\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_SECTIONS2_START\`，供条件编译或文本复用使用。
- **L807**: Defines macro \`KMP_API_NAME_GOMP_WORKSHARE_TASK_REDUCTION_UNREGISTER\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_WORKSHARE_TASK_REDUCTION_UNREGISTER\`，供条件编译或文本复用使用。
- **L808**: Continues the current declaration, expression, or control-flow construct. / 继续当前的声明、表达式或控制流结构。
- **L809**: Defines macro \`KMP_API_NAME_GOMP_ALLOC\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_ALLOC\`，供条件编译或文本复用使用。
- **L810**: Defines macro \`KMP_API_NAME_GOMP_FREE\` for conditional compilation or textual reuse. / 定义宏 \`KMP_API_NAME_GOMP_FREE\`，供条件编译或文本复用使用。
- **L811**: Comment documents intent, constraints, or context for the surrounding code. / 注释用于说明周边代码的意图、约束或上下文。

## Key Concepts / 关键概念

- **Role / 角色**: kmp_ftn_os.h -- KPTS Fortran defines header file. / 声明 OpenMP 运行时：线程/团队管理、调度、同步、环境处理以及 OMPT 钩子。
- **Scale / 规模**: 811 lines, 0 direct includes, 0 named types, and 0 detected routines. / 共 811 行，含 0 个直接包含、0 个具名类型、0 个检测到的例程。
- **OpenMP runtime execution / OpenMP 运行时执行**: The file participates in scheduling, synchronization, team/thread state, or environment processing. / 该文件参与调度、同步、团队/线程状态或环境处理。
- **Symbol management / 符号管理**: The code tracks declarations, references, visibility, or resolution of symbols. / 该代码跟踪符号的声明、引用、可见性或解析过程。
- **Section/segment layout / 节/段布局**: The file manipulates input or output sections, segments, or related layout state. / 该文件操作输入/输出节、段或相关布局状态。
- **OpenMP runtime APIs / OpenMP 运行时 API**: The code interacts with OpenMP runtime entry points, callbacks, or synchronization primitives. / 该代码与 OpenMP 运行时入口、回调或同步原语交互。
- **Device/plugin integration / 设备/插件集成**: The implementation coordinates backend plugins, device state, or host-device interaction. / 该实现协调后端插件、设备状态或主机-设备交互。
- **Concurrency control / 并发控制**: The implementation manages synchronization, thread state, or atomic updates. / 该实现管理同步、线程状态或原子更新。

## Dependencies / 依赖关系

- **Dependencies / 依赖关系**: No direct include or symbol dependency was detected automatically. / 未自动检测到直接的头文件或符号依赖。
