# bivariate_statistics.hpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `third-party/boost-math/include/boost/math/statistics/bivariate_statistics.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/` (`llvm-project`)
- **Purpose / 目的**:
  - **EN**: Declares Boost.Math statistical algorithms, estimators, and container- or iterator-based analysis helpers.
  - **CN**: 声明 Boost.Math 的统计算法、估计器以及基于容器或迭代器的分析辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````cpp
   1: //  (C) Copyright Nick Thompson 2018.
   2: //  (C) Copyright Matt Borland 2021.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_MATH_STATISTICS_BIVARIATE_STATISTICS_HPP
   8: #define BOOST_MATH_STATISTICS_BIVARIATE_STATISTICS_HPP
   9: 
  10: #include <iterator>
  11: #include <tuple>
  12: #include <type_traits>
  13: #include <stdexcept>
  14: #include <vector>
  15: #include <algorithm>
  16: #include <cmath>
  17: #include <cstddef>
  18: #include <boost/math/tools/assert.hpp>
  19: #include <boost/math/tools/config.hpp>
  20: 
````
- **L1 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L1 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L2 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L2 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L3 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L3 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L4 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L4 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L5 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L5 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L6 EN**: Blank line separating nearby declarations or logic.
  - **L6 CN**: 空行，用于分隔相邻声明或逻辑。
- **L7 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_STATISTICS_BIVARIATE_STATISTICS_HPP`.
  - **L7 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_STATISTICS_BIVARIATE_STATISTICS_HPP`。
- **L8 EN**: Defines macro `BOOST_MATH_STATISTICS_BIVARIATE_STATISTICS_HPP` for compile-time control, shorthand, or generated boilerplate.
  - **L8 CN**: 定义宏 `BOOST_MATH_STATISTICS_BIVARIATE_STATISTICS_HPP`，用于编译期控制、简写或生成样板代码。
- **L9 EN**: Blank line separating nearby declarations or logic.
  - **L9 CN**: 空行，用于分隔相邻声明或逻辑。
- **L10 EN**: Includes <iterator> to access C or C++ standard library facilities.
  - **L10 CN**: 引入 <iterator> 以使用C 或 C++ 标准库设施。
- **L11 EN**: Includes <tuple> to access C or C++ standard library facilities.
  - **L11 CN**: 引入 <tuple> 以使用C 或 C++ 标准库设施。
- **L12 EN**: Includes <type_traits> to access C or C++ standard library facilities.
  - **L12 CN**: 引入 <type_traits> 以使用C 或 C++ 标准库设施。
- **L13 EN**: Includes <stdexcept> to access C or C++ standard library facilities.
  - **L13 CN**: 引入 <stdexcept> 以使用C 或 C++ 标准库设施。
- **L14 EN**: Includes <vector> to access C or C++ standard library facilities.
  - **L14 CN**: 引入 <vector> 以使用C 或 C++ 标准库设施。
- **L15 EN**: Includes <algorithm> to access C or C++ standard library facilities.
  - **L15 CN**: 引入 <algorithm> 以使用C 或 C++ 标准库设施。
- **L16 EN**: Includes <cmath> to access C or C++ standard library facilities.
  - **L16 CN**: 引入 <cmath> 以使用C 或 C++ 标准库设施。
- **L17 EN**: Includes <cstddef> to access C or C++ standard library facilities.
  - **L17 CN**: 引入 <cstddef> 以使用C 或 C++ 标准库设施。
- **L18 EN**: Includes <boost/math/tools/assert.hpp> to access Boost.Math numeric tool helpers.
  - **L18 CN**: 引入 <boost/math/tools/assert.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L19 EN**: Includes <boost/math/tools/config.hpp> to access Boost.Math numeric tool helpers.
  - **L19 CN**: 引入 <boost/math/tools/config.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L20 EN**: Blank line separating nearby declarations or logic.
  - **L20 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 21-40 / 第 21-40 行

````cpp
  21: #ifdef BOOST_MATH_EXEC_COMPATIBLE
  22: #include <execution>
  23: #include <future>
  24: #include <thread>
  25: #endif
  26: 
  27: namespace boost{ namespace math{ namespace statistics { namespace detail {
  28: 
  29: // See Equation III.9 of "Numerically Stable, Single-Pass, Parallel Statistics Algorithms", Bennet et al.
  30: template<typename ReturnType, typename ForwardIterator>
  31: ReturnType means_and_covariance_seq_impl(ForwardIterator u_begin, ForwardIterator u_end, ForwardIterator v_begin, ForwardIterator v_end)
  32: {
  33:     using Real = typename std::tuple_element<0, ReturnType>::type;
  34: 
  35:     Real cov = 0;
  36:     ForwardIterator u_it = u_begin;
  37:     ForwardIterator v_it = v_begin;
  38:     Real mu_u = *u_it++;
  39:     Real mu_v = *v_it++;
  40:     std::size_t i = 1;
````
- **L21 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_EXEC_COMPATIBLE`.
  - **L21 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_EXEC_COMPATIBLE`。
- **L22 EN**: Includes <execution> to access C or C++ standard library facilities.
  - **L22 CN**: 引入 <execution> 以使用C 或 C++ 标准库设施。
- **L23 EN**: Includes <future> to access C or C++ standard library facilities.
  - **L23 CN**: 引入 <future> 以使用C 或 C++ 标准库设施。
- **L24 EN**: Includes <thread> to access C or C++ standard library facilities.
  - **L24 CN**: 引入 <thread> 以使用C 或 C++ 标准库设施。
- **L25 EN**: Closes the current preprocessor conditional block or header guard.
  - **L25 CN**: 结束当前预处理条件块或头文件保护。
- **L26 EN**: Blank line separating nearby declarations or logic.
  - **L26 CN**: 空行，用于分隔相邻声明或逻辑。
- **L27 EN**: Opens namespace scope `boost{ namespace math{ namespace statistics { namespace detail`.
  - **L27 CN**: 打开命名空间作用域 `boost{ namespace math{ namespace statistics { namespace detail`。
- **L28 EN**: Blank line separating nearby declarations or logic.
  - **L28 CN**: 空行，用于分隔相邻声明或逻辑。
- **L29 EN**: Comment documents nearby intent or usage notes: `See Equation III.9 of "Numerically Stable, Single-Pass, Parallel Statistics Algorithms", Bennet et al.`.
  - **L29 CN**: 注释说明附近代码的意图或使用说明：`See Equation III.9 of "Numerically Stable, Single-Pass, Parallel Statistics Algorithms", Bennet et al.`。
- **L30 EN**: Introduces template parameters or specialization context: `template<typename ReturnType, typename ForwardIterator>`.
  - **L30 CN**: 为后续声明引入模板参数或特化上下文：`template<typename ReturnType, typename ForwardIterator>`。
- **L31 EN**: Continues logic associated with callable symbol `means_and_covariance_seq_impl`.
  - **L31 CN**: 继续与可调用符号 `means_and_covariance_seq_impl` 相关的逻辑。
- **L32 EN**: Opens a new lexical scope or compound statement.
  - **L32 CN**: 打开一个新的词法作用域或复合语句块。
- **L33 EN**: Defines alias `Real` to simplify later code.
  - **L33 CN**: 定义别名 `Real` 以简化后续代码。
- **L34 EN**: Blank line separating nearby declarations or logic.
  - **L34 CN**: 空行，用于分隔相邻声明或逻辑。
- **L35 EN**: Initializes variable `cov` from the right-hand expression.
  - **L35 CN**: 使用右侧表达式初始化变量 `cov`。
- **L36 EN**: Initializes variable `u_it` from the right-hand expression.
  - **L36 CN**: 使用右侧表达式初始化变量 `u_it`。
- **L37 EN**: Initializes variable `v_it` from the right-hand expression.
  - **L37 CN**: 使用右侧表达式初始化变量 `v_it`。
- **L38 EN**: Initializes variable `mu_u` from the right-hand expression.
  - **L38 CN**: 使用右侧表达式初始化变量 `mu_u`。
- **L39 EN**: Initializes variable `mu_v` from the right-hand expression.
  - **L39 CN**: 使用右侧表达式初始化变量 `mu_v`。
- **L40 EN**: Initializes variable `i` from the right-hand expression.
  - **L40 CN**: 使用右侧表达式初始化变量 `i`。

### Lines 41-60 / 第 41-60 行

````cpp
  41: 
  42:     while(u_it != u_end && v_it != v_end)
  43:     {
  44:         Real u_temp = (*u_it++ - mu_u)/(i+1);
  45:         Real v_temp = *v_it++ - mu_v;
  46:         cov += i*u_temp*v_temp;
  47:         mu_u = mu_u + u_temp;
  48:         mu_v = mu_v + v_temp/(i+1);
  49:         i = i + 1;
  50:     }
  51: 
  52:     if(u_it != u_end || v_it != v_end)
  53:     {
  54:         throw std::domain_error("The size of each sample set must be the same to compute covariance");
  55:     }
  56: 
  57:     return std::make_tuple(mu_u, mu_v, cov/i, Real(i));
  58: }
  59: 
  60: #ifdef BOOST_MATH_EXEC_COMPATIBLE
````
- **L41 EN**: Blank line separating nearby declarations or logic.
  - **L41 CN**: 空行，用于分隔相邻声明或逻辑。
- **L42 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L42 CN**: 开始 `while` 控制流语句并计算其条件。
- **L43 EN**: Opens a new lexical scope or compound statement.
  - **L43 CN**: 打开一个新的词法作用域或复合语句块。
- **L44 EN**: Initializes variable `u_temp` from the right-hand expression.
  - **L44 CN**: 使用右侧表达式初始化变量 `u_temp`。
- **L45 EN**: Initializes variable `v_temp` from the right-hand expression.
  - **L45 CN**: 使用右侧表达式初始化变量 `v_temp`。
- **L46 EN**: Executes a standalone statement or declaration: `cov += i*u_temp*v_temp;`.
  - **L46 CN**: 执行一条独立语句或声明：`cov += i*u_temp*v_temp;`。
- **L47 EN**: Executes a standalone statement or declaration: `mu_u = mu_u + u_temp;`.
  - **L47 CN**: 执行一条独立语句或声明：`mu_u = mu_u + u_temp;`。
- **L48 EN**: Executes a call or declaration centered on `v_temp/`.
  - **L48 CN**: 执行以 `v_temp/` 为核心的调用或声明。
- **L49 EN**: Executes a standalone statement or declaration: `i = i + 1;`.
  - **L49 CN**: 执行一条独立语句或声明：`i = i + 1;`。
- **L50 EN**: Closes the current lexical scope or compound statement.
  - **L50 CN**: 结束当前词法作用域或复合语句块。
- **L51 EN**: Blank line separating nearby declarations or logic.
  - **L51 CN**: 空行，用于分隔相邻声明或逻辑。
- **L52 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L52 CN**: 开始 `if` 控制流语句并计算其条件。
- **L53 EN**: Opens a new lexical scope or compound statement.
  - **L53 CN**: 打开一个新的词法作用域或复合语句块。
- **L54 EN**: Throws an exception object or error marker: `throw std::domain_error("The size of each sample set must be the same to compute covariance");`.
  - **L54 CN**: 抛出异常对象或错误标记：`throw std::domain_error("The size of each sample set must be the same to compute covariance");`。
- **L55 EN**: Closes the current lexical scope or compound statement.
  - **L55 CN**: 结束当前词法作用域或复合语句块。
- **L56 EN**: Blank line separating nearby declarations or logic.
  - **L56 CN**: 空行，用于分隔相邻声明或逻辑。
- **L57 EN**: Returns from the current function with `std::make_tuple(mu_u, mu_v, cov/i, Real(i))`.
  - **L57 CN**: 以 `std::make_tuple(mu_u, mu_v, cov/i, Real(i))` 从当前函数返回。
- **L58 EN**: Closes the current lexical scope or compound statement.
  - **L58 CN**: 结束当前词法作用域或复合语句块。
- **L59 EN**: Blank line separating nearby declarations or logic.
  - **L59 CN**: 空行，用于分隔相邻声明或逻辑。
- **L60 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_EXEC_COMPATIBLE`.
  - **L60 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_EXEC_COMPATIBLE`。

### Lines 61-80 / 第 61-80 行

````cpp
  61: 
  62: // Numerically stable parallel computation of (co-)variance
  63: // https://dl.acm.org/doi/10.1145/3221269.3223036
  64: template<typename ReturnType, typename ForwardIterator>
  65: ReturnType means_and_covariance_parallel_impl(ForwardIterator u_begin, ForwardIterator u_end, ForwardIterator v_begin, ForwardIterator v_end)
  66: {
  67:     using Real = typename std::tuple_element<0, ReturnType>::type;
  68: 
  69:     const auto u_elements = std::distance(u_begin, u_end);
  70:     const auto v_elements = std::distance(v_begin, v_end);
  71: 
  72:     if(u_elements != v_elements)
  73:     {
  74:         throw std::domain_error("The size of each sample set must be the same to compute covariance");
  75:     }
  76: 
  77:     const unsigned max_concurrency = std::thread::hardware_concurrency() == 0 ? 2u : std::thread::hardware_concurrency();
  78:     unsigned num_threads = 2u;
  79:     
  80:     // 5.16 comes from benchmarking. See boost/math/reporting/performance/bivariate_statistics_performance.cpp
````
- **L61 EN**: Blank line separating nearby declarations or logic.
  - **L61 CN**: 空行，用于分隔相邻声明或逻辑。
- **L62 EN**: Comment documents nearby intent or usage notes: `Numerically stable parallel computation of (co-)variance`.
  - **L62 CN**: 注释说明附近代码的意图或使用说明：`Numerically stable parallel computation of (co-)variance`。
- **L63 EN**: Comment documents nearby intent or usage notes: `https://dl.acm.org/doi/10.1145/3221269.3223036`.
  - **L63 CN**: 注释说明附近代码的意图或使用说明：`https://dl.acm.org/doi/10.1145/3221269.3223036`。
- **L64 EN**: Introduces template parameters or specialization context: `template<typename ReturnType, typename ForwardIterator>`.
  - **L64 CN**: 为后续声明引入模板参数或特化上下文：`template<typename ReturnType, typename ForwardIterator>`。
- **L65 EN**: Continues logic associated with callable symbol `means_and_covariance_parallel_impl`.
  - **L65 CN**: 继续与可调用符号 `means_and_covariance_parallel_impl` 相关的逻辑。
- **L66 EN**: Opens a new lexical scope or compound statement.
  - **L66 CN**: 打开一个新的词法作用域或复合语句块。
- **L67 EN**: Defines alias `Real` to simplify later code.
  - **L67 CN**: 定义别名 `Real` 以简化后续代码。
- **L68 EN**: Blank line separating nearby declarations or logic.
  - **L68 CN**: 空行，用于分隔相邻声明或逻辑。
- **L69 EN**: Initializes variable `u_elements` from the right-hand expression.
  - **L69 CN**: 使用右侧表达式初始化变量 `u_elements`。
- **L70 EN**: Initializes variable `v_elements` from the right-hand expression.
  - **L70 CN**: 使用右侧表达式初始化变量 `v_elements`。
- **L71 EN**: Blank line separating nearby declarations or logic.
  - **L71 CN**: 空行，用于分隔相邻声明或逻辑。
- **L72 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L72 CN**: 开始 `if` 控制流语句并计算其条件。
- **L73 EN**: Opens a new lexical scope or compound statement.
  - **L73 CN**: 打开一个新的词法作用域或复合语句块。
- **L74 EN**: Throws an exception object or error marker: `throw std::domain_error("The size of each sample set must be the same to compute covariance");`.
  - **L74 CN**: 抛出异常对象或错误标记：`throw std::domain_error("The size of each sample set must be the same to compute covariance");`。
- **L75 EN**: Closes the current lexical scope or compound statement.
  - **L75 CN**: 结束当前词法作用域或复合语句块。
- **L76 EN**: Blank line separating nearby declarations or logic.
  - **L76 CN**: 空行，用于分隔相邻声明或逻辑。
- **L77 EN**: Initializes variable `max_concurrency` from the right-hand expression.
  - **L77 CN**: 使用右侧表达式初始化变量 `max_concurrency`。
- **L78 EN**: Initializes variable `num_threads` from the right-hand expression.
  - **L78 CN**: 使用右侧表达式初始化变量 `num_threads`。
- **L79 EN**: Blank line separating nearby declarations or logic.
  - **L79 CN**: 空行，用于分隔相邻声明或逻辑。
- **L80 EN**: Comment documents nearby intent or usage notes: `5.16 comes from benchmarking. See boost/math/reporting/performance/bivariate_statistics_performance.cpp`.
  - **L80 CN**: 注释说明附近代码的意图或使用说明：`5.16 comes from benchmarking. See boost/math/reporting/performance/bivariate_statistics_performance.cpp`。

### Lines 81-100 / 第 81-100 行

````cpp
  81:     // Threading is faster for: 10 + 5.16e-3 N/j <= 5.16e-3N => N >= 10^4j/5.16(j-1).
  82:     const auto parallel_lower_bound = 10e4*max_concurrency/(5.16*(max_concurrency-1));
  83:     const auto parallel_upper_bound = 10e4*2/5.16; // j = 2
  84: 
  85:     // https://lemire.me/blog/2020/01/30/cost-of-a-thread-in-c-under-linux/
  86:     if(u_elements < parallel_lower_bound)
  87:     {
  88:         return means_and_covariance_seq_impl<ReturnType>(u_begin, u_end, v_begin, v_end);
  89:     }
  90:     else if(u_elements >= parallel_upper_bound)
  91:     {
  92:         num_threads = max_concurrency;
  93:     }
  94:     else
  95:     {
  96:         for(unsigned i = 3; i < max_concurrency; ++i)
  97:         {
  98:             if(parallel_lower_bound < 10e4*i/(5.16*(i-1)))
  99:             {
 100:                 num_threads = i;
````
- **L81 EN**: Comment documents nearby intent or usage notes: `Threading is faster for: 10 + 5.16e-3 N/j <= 5.16e-3N => N >= 10^4j/5.16(j-1).`.
  - **L81 CN**: 注释说明附近代码的意图或使用说明：`Threading is faster for: 10 + 5.16e-3 N/j <= 5.16e-3N => N >= 10^4j/5.16(j-1).`。
- **L82 EN**: Initializes variable `parallel_lower_bound` from the right-hand expression.
  - **L82 CN**: 使用右侧表达式初始化变量 `parallel_lower_bound`。
- **L83 EN**: Continues the surrounding expression or declaration: `const auto parallel_upper_bound = 10e4*2/5.16; // j = 2`.
  - **L83 CN**: 继续构造周围的表达式或声明：`const auto parallel_upper_bound = 10e4*2/5.16; // j = 2`。
- **L84 EN**: Blank line separating nearby declarations or logic.
  - **L84 CN**: 空行，用于分隔相邻声明或逻辑。
- **L85 EN**: Comment documents nearby intent or usage notes: `https://lemire.me/blog/2020/01/30/cost-of-a-thread-in-c-under-linux/`.
  - **L85 CN**: 注释说明附近代码的意图或使用说明：`https://lemire.me/blog/2020/01/30/cost-of-a-thread-in-c-under-linux/`。
- **L86 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L86 CN**: 开始 `if` 控制流语句并计算其条件。
- **L87 EN**: Opens a new lexical scope or compound statement.
  - **L87 CN**: 打开一个新的词法作用域或复合语句块。
- **L88 EN**: Returns from the current function with `means_and_covariance_seq_impl<ReturnType>(u_begin, u_end, v_begin, v_end)`.
  - **L88 CN**: 以 `means_and_covariance_seq_impl<ReturnType>(u_begin, u_end, v_begin, v_end)` 从当前函数返回。
- **L89 EN**: Closes the current lexical scope or compound statement.
  - **L89 CN**: 结束当前词法作用域或复合语句块。
- **L90 EN**: Starts the alternative branch of the preceding conditional.
  - **L90 CN**: 开始前一个条件语句的备选分支。
- **L91 EN**: Opens a new lexical scope or compound statement.
  - **L91 CN**: 打开一个新的词法作用域或复合语句块。
- **L92 EN**: Executes a standalone statement or declaration: `num_threads = max_concurrency;`.
  - **L92 CN**: 执行一条独立语句或声明：`num_threads = max_concurrency;`。
- **L93 EN**: Closes the current lexical scope or compound statement.
  - **L93 CN**: 结束当前词法作用域或复合语句块。
- **L94 EN**: Starts the alternative branch of the preceding conditional.
  - **L94 CN**: 开始前一个条件语句的备选分支。
- **L95 EN**: Opens a new lexical scope or compound statement.
  - **L95 CN**: 打开一个新的词法作用域或复合语句块。
- **L96 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L96 CN**: 开始 `for` 控制流语句并计算其条件。
- **L97 EN**: Opens a new lexical scope or compound statement.
  - **L97 CN**: 打开一个新的词法作用域或复合语句块。
- **L98 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L98 CN**: 开始 `if` 控制流语句并计算其条件。
- **L99 EN**: Opens a new lexical scope or compound statement.
  - **L99 CN**: 打开一个新的词法作用域或复合语句块。
- **L100 EN**: Executes a standalone statement or declaration: `num_threads = i;`.
  - **L100 CN**: 执行一条独立语句或声明：`num_threads = i;`。

### Lines 101-120 / 第 101-120 行

````cpp
 101:                 break;
 102:             }
 103:         }
 104:     }
 105: 
 106:     std::vector<std::future<ReturnType>> future_manager;
 107:     const auto elements_per_thread = std::ceil(static_cast<double>(u_elements)/num_threads);
 108: 
 109:     ForwardIterator u_it = u_begin;
 110:     ForwardIterator v_it = v_begin;
 111: 
 112:     for(std::size_t i = 0; i < num_threads - 1; ++i)
 113:     {
 114:         future_manager.emplace_back(std::async(std::launch::async | std::launch::deferred, [u_it, v_it, elements_per_thread]() -> ReturnType
 115:         {
 116:             return means_and_covariance_seq_impl<ReturnType>(u_it, std::next(u_it, elements_per_thread), v_it, std::next(v_it, elements_per_thread));
 117:         }));
 118:         u_it = std::next(u_it, elements_per_thread);
 119:         v_it = std::next(v_it, elements_per_thread);
 120:     }
````
- **L101 EN**: Exits the nearest loop or switch statement.
  - **L101 CN**: 退出最近的循环或 switch 语句。
- **L102 EN**: Closes the current lexical scope or compound statement.
  - **L102 CN**: 结束当前词法作用域或复合语句块。
- **L103 EN**: Closes the current lexical scope or compound statement.
  - **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Closes the current lexical scope or compound statement.
  - **L104 CN**: 结束当前词法作用域或复合语句块。
- **L105 EN**: Blank line separating nearby declarations or logic.
  - **L105 CN**: 空行，用于分隔相邻声明或逻辑。
- **L106 EN**: Executes a standalone statement or declaration: `std::vector<std::future<ReturnType>> future_manager;`.
  - **L106 CN**: 执行一条独立语句或声明：`std::vector<std::future<ReturnType>> future_manager;`。
- **L107 EN**: Initializes variable `elements_per_thread` from the right-hand expression.
  - **L107 CN**: 使用右侧表达式初始化变量 `elements_per_thread`。
- **L108 EN**: Blank line separating nearby declarations or logic.
  - **L108 CN**: 空行，用于分隔相邻声明或逻辑。
- **L109 EN**: Initializes variable `u_it` from the right-hand expression.
  - **L109 CN**: 使用右侧表达式初始化变量 `u_it`。
- **L110 EN**: Initializes variable `v_it` from the right-hand expression.
  - **L110 CN**: 使用右侧表达式初始化变量 `v_it`。
- **L111 EN**: Blank line separating nearby declarations or logic.
  - **L111 CN**: 空行，用于分隔相邻声明或逻辑。
- **L112 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L112 CN**: 开始 `for` 控制流语句并计算其条件。
- **L113 EN**: Opens a new lexical scope or compound statement.
  - **L113 CN**: 打开一个新的词法作用域或复合语句块。
- **L114 EN**: Continues logic associated with callable symbol `emplace_back`.
  - **L114 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L115 EN**: Opens a new lexical scope or compound statement.
  - **L115 CN**: 打开一个新的词法作用域或复合语句块。
- **L116 EN**: Returns from the current function with `means_and_covariance_seq_impl<ReturnType>(u_it, std::next(u_it, elements_per_thread), v_it, std::next(v_it, elements_per_thread))`.
  - **L116 CN**: 以 `means_and_covariance_seq_impl<ReturnType>(u_it, std::next(u_it, elements_per_thread), v_it, std::next(v_it, elements_per_thread))` 从当前函数返回。
- **L117 EN**: Executes a standalone statement or declaration: `}));`.
  - **L117 CN**: 执行一条独立语句或声明：`}));`。
- **L118 EN**: Executes a call or declaration centered on `std::next`.
  - **L118 CN**: 执行以 `std::next` 为核心的调用或声明。
- **L119 EN**: Executes a call or declaration centered on `std::next`.
  - **L119 CN**: 执行以 `std::next` 为核心的调用或声明。
- **L120 EN**: Closes the current lexical scope or compound statement.
  - **L120 CN**: 结束当前词法作用域或复合语句块。

### Lines 121-140 / 第 121-140 行

````cpp
 121: 
 122:     future_manager.emplace_back(std::async(std::launch::async | std::launch::deferred, [u_it, u_end, v_it, v_end]() -> ReturnType
 123:     {
 124:         return means_and_covariance_seq_impl<ReturnType>(u_it, u_end, v_it, v_end);
 125:     }));
 126: 
 127:     ReturnType temp = future_manager[0].get();
 128:     Real mu_u_a = std::get<0>(temp);
 129:     Real mu_v_a = std::get<1>(temp);
 130:     Real cov_a = std::get<2>(temp);
 131:     Real n_a = std::get<3>(temp);
 132: 
 133:     for(std::size_t i = 1; i < future_manager.size(); ++i)
 134:     {
 135:         temp = future_manager[i].get();
 136:         Real mu_u_b = std::get<0>(temp);
 137:         Real mu_v_b = std::get<1>(temp);
 138:         Real cov_b = std::get<2>(temp);
 139:         Real n_b = std::get<3>(temp);
 140: 
````
- **L121 EN**: Blank line separating nearby declarations or logic.
  - **L121 CN**: 空行，用于分隔相邻声明或逻辑。
- **L122 EN**: Continues logic associated with callable symbol `emplace_back`.
  - **L122 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L123 EN**: Opens a new lexical scope or compound statement.
  - **L123 CN**: 打开一个新的词法作用域或复合语句块。
- **L124 EN**: Returns from the current function with `means_and_covariance_seq_impl<ReturnType>(u_it, u_end, v_it, v_end)`.
  - **L124 CN**: 以 `means_and_covariance_seq_impl<ReturnType>(u_it, u_end, v_it, v_end)` 从当前函数返回。
- **L125 EN**: Executes a standalone statement or declaration: `}));`.
  - **L125 CN**: 执行一条独立语句或声明：`}));`。
- **L126 EN**: Blank line separating nearby declarations or logic.
  - **L126 CN**: 空行，用于分隔相邻声明或逻辑。
- **L127 EN**: Initializes variable `temp` from the right-hand expression.
  - **L127 CN**: 使用右侧表达式初始化变量 `temp`。
- **L128 EN**: Initializes variable `mu_u_a` from the right-hand expression.
  - **L128 CN**: 使用右侧表达式初始化变量 `mu_u_a`。
- **L129 EN**: Initializes variable `mu_v_a` from the right-hand expression.
  - **L129 CN**: 使用右侧表达式初始化变量 `mu_v_a`。
- **L130 EN**: Initializes variable `cov_a` from the right-hand expression.
  - **L130 CN**: 使用右侧表达式初始化变量 `cov_a`。
- **L131 EN**: Initializes variable `n_a` from the right-hand expression.
  - **L131 CN**: 使用右侧表达式初始化变量 `n_a`。
- **L132 EN**: Blank line separating nearby declarations or logic.
  - **L132 CN**: 空行，用于分隔相邻声明或逻辑。
- **L133 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L133 CN**: 开始 `for` 控制流语句并计算其条件。
- **L134 EN**: Opens a new lexical scope or compound statement.
  - **L134 CN**: 打开一个新的词法作用域或复合语句块。
- **L135 EN**: Executes a call or declaration centered on `future_manager[i].get`.
  - **L135 CN**: 执行以 `future_manager[i].get` 为核心的调用或声明。
- **L136 EN**: Initializes variable `mu_u_b` from the right-hand expression.
  - **L136 CN**: 使用右侧表达式初始化变量 `mu_u_b`。
- **L137 EN**: Initializes variable `mu_v_b` from the right-hand expression.
  - **L137 CN**: 使用右侧表达式初始化变量 `mu_v_b`。
- **L138 EN**: Initializes variable `cov_b` from the right-hand expression.
  - **L138 CN**: 使用右侧表达式初始化变量 `cov_b`。
- **L139 EN**: Initializes variable `n_b` from the right-hand expression.
  - **L139 CN**: 使用右侧表达式初始化变量 `n_b`。
- **L140 EN**: Blank line separating nearby declarations or logic.
  - **L140 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 141-160 / 第 141-160 行

````cpp
 141:         const Real n_ab = n_a + n_b;
 142:         const Real delta_u = mu_u_b - mu_u_a;
 143:         const Real delta_v = mu_v_b - mu_v_a;
 144: 
 145:         cov_a = cov_a + cov_b + (-delta_u)*(-delta_v)*((n_a*n_b)/n_ab);
 146:         mu_u_a = mu_u_a + delta_u*(n_b/n_ab);
 147:         mu_v_a = mu_v_a + delta_v*(n_b/n_ab);
 148:         n_a = n_ab;
 149:     }
 150: 
 151:     return std::make_tuple(mu_u_a, mu_v_a, cov_a, n_a);
 152: }
 153: 
 154: #endif // BOOST_MATH_EXEC_COMPATIBLE
 155: 
 156: template<typename ReturnType, typename ForwardIterator>
 157: ReturnType correlation_coefficient_seq_impl(ForwardIterator u_begin, ForwardIterator u_end, ForwardIterator v_begin, ForwardIterator v_end)
 158: {
 159:     using Real = typename std::tuple_element<0, ReturnType>::type;
 160:     using std::sqrt;
````
- **L141 EN**: Initializes variable `n_ab` from the right-hand expression.
  - **L141 CN**: 使用右侧表达式初始化变量 `n_ab`。
- **L142 EN**: Initializes variable `delta_u` from the right-hand expression.
  - **L142 CN**: 使用右侧表达式初始化变量 `delta_u`。
- **L143 EN**: Initializes variable `delta_v` from the right-hand expression.
  - **L143 CN**: 使用右侧表达式初始化变量 `delta_v`。
- **L144 EN**: Blank line separating nearby declarations or logic.
  - **L144 CN**: 空行，用于分隔相邻声明或逻辑。
- **L145 EN**: Executes a call or declaration centered on `+`.
  - **L145 CN**: 执行以 `+` 为核心的调用或声明。
- **L146 EN**: Executes a call or declaration centered on `delta_u*`.
  - **L146 CN**: 执行以 `delta_u*` 为核心的调用或声明。
- **L147 EN**: Executes a call or declaration centered on `delta_v*`.
  - **L147 CN**: 执行以 `delta_v*` 为核心的调用或声明。
- **L148 EN**: Executes a standalone statement or declaration: `n_a = n_ab;`.
  - **L148 CN**: 执行一条独立语句或声明：`n_a = n_ab;`。
- **L149 EN**: Closes the current lexical scope or compound statement.
  - **L149 CN**: 结束当前词法作用域或复合语句块。
- **L150 EN**: Blank line separating nearby declarations or logic.
  - **L150 CN**: 空行，用于分隔相邻声明或逻辑。
- **L151 EN**: Returns from the current function with `std::make_tuple(mu_u_a, mu_v_a, cov_a, n_a)`.
  - **L151 CN**: 以 `std::make_tuple(mu_u_a, mu_v_a, cov_a, n_a)` 从当前函数返回。
- **L152 EN**: Closes the current lexical scope or compound statement.
  - **L152 CN**: 结束当前词法作用域或复合语句块。
- **L153 EN**: Blank line separating nearby declarations or logic.
  - **L153 CN**: 空行，用于分隔相邻声明或逻辑。
- **L154 EN**: Closes the current preprocessor conditional block or header guard.
  - **L154 CN**: 结束当前预处理条件块或头文件保护。
- **L155 EN**: Blank line separating nearby declarations or logic.
  - **L155 CN**: 空行，用于分隔相邻声明或逻辑。
- **L156 EN**: Introduces template parameters or specialization context: `template<typename ReturnType, typename ForwardIterator>`.
  - **L156 CN**: 为后续声明引入模板参数或特化上下文：`template<typename ReturnType, typename ForwardIterator>`。
- **L157 EN**: Continues logic associated with callable symbol `correlation_coefficient_seq_impl`.
  - **L157 CN**: 继续与可调用符号 `correlation_coefficient_seq_impl` 相关的逻辑。
- **L158 EN**: Opens a new lexical scope or compound statement.
  - **L158 CN**: 打开一个新的词法作用域或复合语句块。
- **L159 EN**: Defines alias `Real` to simplify later code.
  - **L159 CN**: 定义别名 `Real` 以简化后续代码。
- **L160 EN**: Executes a standalone statement or declaration: `using std::sqrt;`.
  - **L160 CN**: 执行一条独立语句或声明：`using std::sqrt;`。

### Lines 161-180 / 第 161-180 行

````cpp
 161: 
 162:     Real cov = 0;
 163:     ForwardIterator u_it = u_begin;
 164:     ForwardIterator v_it = v_begin;
 165:     Real mu_u = *u_it++;
 166:     Real mu_v = *v_it++;
 167:     Real Qu = 0;
 168:     Real Qv = 0;
 169:     std::size_t i = 1;
 170: 
 171:     while(u_it != u_end && v_it != v_end)
 172:     {
 173:         Real u_tmp = *u_it++ - mu_u;
 174:         Real v_tmp = *v_it++ - mu_v;
 175:         Qu = Qu + (i*u_tmp*u_tmp)/(i+1);
 176:         Qv = Qv + (i*v_tmp*v_tmp)/(i+1);
 177:         cov += i*u_tmp*v_tmp/(i+1);
 178:         mu_u = mu_u + u_tmp/(i+1);
 179:         mu_v = mu_v + v_tmp/(i+1);
 180:         ++i;
````
- **L161 EN**: Blank line separating nearby declarations or logic.
  - **L161 CN**: 空行，用于分隔相邻声明或逻辑。
- **L162 EN**: Initializes variable `cov` from the right-hand expression.
  - **L162 CN**: 使用右侧表达式初始化变量 `cov`。
- **L163 EN**: Initializes variable `u_it` from the right-hand expression.
  - **L163 CN**: 使用右侧表达式初始化变量 `u_it`。
- **L164 EN**: Initializes variable `v_it` from the right-hand expression.
  - **L164 CN**: 使用右侧表达式初始化变量 `v_it`。
- **L165 EN**: Initializes variable `mu_u` from the right-hand expression.
  - **L165 CN**: 使用右侧表达式初始化变量 `mu_u`。
- **L166 EN**: Initializes variable `mu_v` from the right-hand expression.
  - **L166 CN**: 使用右侧表达式初始化变量 `mu_v`。
- **L167 EN**: Initializes variable `Qu` from the right-hand expression.
  - **L167 CN**: 使用右侧表达式初始化变量 `Qu`。
- **L168 EN**: Initializes variable `Qv` from the right-hand expression.
  - **L168 CN**: 使用右侧表达式初始化变量 `Qv`。
- **L169 EN**: Initializes variable `i` from the right-hand expression.
  - **L169 CN**: 使用右侧表达式初始化变量 `i`。
- **L170 EN**: Blank line separating nearby declarations or logic.
  - **L170 CN**: 空行，用于分隔相邻声明或逻辑。
- **L171 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L171 CN**: 开始 `while` 控制流语句并计算其条件。
- **L172 EN**: Opens a new lexical scope or compound statement.
  - **L172 CN**: 打开一个新的词法作用域或复合语句块。
- **L173 EN**: Initializes variable `u_tmp` from the right-hand expression.
  - **L173 CN**: 使用右侧表达式初始化变量 `u_tmp`。
- **L174 EN**: Initializes variable `v_tmp` from the right-hand expression.
  - **L174 CN**: 使用右侧表达式初始化变量 `v_tmp`。
- **L175 EN**: Executes a call or declaration centered on `+`.
  - **L175 CN**: 执行以 `+` 为核心的调用或声明。
- **L176 EN**: Executes a call or declaration centered on `+`.
  - **L176 CN**: 执行以 `+` 为核心的调用或声明。
- **L177 EN**: Executes a call or declaration centered on `i*u_tmp*v_tmp/`.
  - **L177 CN**: 执行以 `i*u_tmp*v_tmp/` 为核心的调用或声明。
- **L178 EN**: Executes a call or declaration centered on `u_tmp/`.
  - **L178 CN**: 执行以 `u_tmp/` 为核心的调用或声明。
- **L179 EN**: Executes a call or declaration centered on `v_tmp/`.
  - **L179 CN**: 执行以 `v_tmp/` 为核心的调用或声明。
- **L180 EN**: Executes a standalone statement or declaration: `++i;`.
  - **L180 CN**: 执行一条独立语句或声明：`++i;`。

### Lines 181-200 / 第 181-200 行

````cpp
 181:     }
 182: 
 183: 
 184:     // If one dataset is constant, then the correlation coefficient is undefined.
 185:     // See https://stats.stackexchange.com/questions/23676/normalized-correlation-with-a-constant-vector
 186:     // Thanks to zbjornson for pointing this out.
 187:     if (Qu == 0 || Qv == 0)
 188:     {
 189:         return std::make_tuple(mu_u, Qu, mu_v, Qv, cov, std::numeric_limits<Real>::quiet_NaN(), Real(i));
 190:     }
 191: 
 192:     // Make sure rho in [-1, 1], even in the presence of numerical noise.
 193:     Real rho = cov/sqrt(Qu*Qv);
 194:     if (rho > 1) {
 195:         rho = 1;
 196:     }
 197:     if (rho < -1) {
 198:         rho = -1;
 199:     }
 200: 
````
- **L181 EN**: Closes the current lexical scope or compound statement.
  - **L181 CN**: 结束当前词法作用域或复合语句块。
- **L182 EN**: Blank line separating nearby declarations or logic.
  - **L182 CN**: 空行，用于分隔相邻声明或逻辑。
- **L183 EN**: Blank line separating nearby declarations or logic.
  - **L183 CN**: 空行，用于分隔相邻声明或逻辑。
- **L184 EN**: Comment documents nearby intent or usage notes: `If one dataset is constant, then the correlation coefficient is undefined.`.
  - **L184 CN**: 注释说明附近代码的意图或使用说明：`If one dataset is constant, then the correlation coefficient is undefined.`。
- **L185 EN**: Comment documents nearby intent or usage notes: `See https://stats.stackexchange.com/questions/23676/normalized-correlation-with-a-constant-vector`.
  - **L185 CN**: 注释说明附近代码的意图或使用说明：`See https://stats.stackexchange.com/questions/23676/normalized-correlation-with-a-constant-vector`。
- **L186 EN**: Comment documents nearby intent or usage notes: `Thanks to zbjornson for pointing this out.`.
  - **L186 CN**: 注释说明附近代码的意图或使用说明：`Thanks to zbjornson for pointing this out.`。
- **L187 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L187 CN**: 开始 `if` 控制流语句并计算其条件。
- **L188 EN**: Opens a new lexical scope or compound statement.
  - **L188 CN**: 打开一个新的词法作用域或复合语句块。
- **L189 EN**: Returns from the current function with `std::make_tuple(mu_u, Qu, mu_v, Qv, cov, std::numeric_limits<Real>::quiet_NaN(), Real(i))`.
  - **L189 CN**: 以 `std::make_tuple(mu_u, Qu, mu_v, Qv, cov, std::numeric_limits<Real>::quiet_NaN(), Real(i))` 从当前函数返回。
- **L190 EN**: Closes the current lexical scope or compound statement.
  - **L190 CN**: 结束当前词法作用域或复合语句块。
- **L191 EN**: Blank line separating nearby declarations or logic.
  - **L191 CN**: 空行，用于分隔相邻声明或逻辑。
- **L192 EN**: Comment documents nearby intent or usage notes: `Make sure rho in [-1, 1], even in the presence of numerical noise.`.
  - **L192 CN**: 注释说明附近代码的意图或使用说明：`Make sure rho in [-1, 1], even in the presence of numerical noise.`。
- **L193 EN**: Initializes variable `rho` from the right-hand expression.
  - **L193 CN**: 使用右侧表达式初始化变量 `rho`。
- **L194 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L194 CN**: 开始 `if` 控制流语句并计算其条件。
- **L195 EN**: Executes a standalone statement or declaration: `rho = 1;`.
  - **L195 CN**: 执行一条独立语句或声明：`rho = 1;`。
- **L196 EN**: Closes the current lexical scope or compound statement.
  - **L196 CN**: 结束当前词法作用域或复合语句块。
- **L197 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L197 CN**: 开始 `if` 控制流语句并计算其条件。
- **L198 EN**: Executes a standalone statement or declaration: `rho = -1;`.
  - **L198 CN**: 执行一条独立语句或声明：`rho = -1;`。
- **L199 EN**: Closes the current lexical scope or compound statement.
  - **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Blank line separating nearby declarations or logic.
  - **L200 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 201-220 / 第 201-220 行

````cpp
 201:     return std::make_tuple(mu_u, Qu, mu_v, Qv, cov, rho, Real(i));
 202: }
 203: 
 204: #ifdef BOOST_MATH_EXEC_COMPATIBLE
 205: 
 206: // Numerically stable parallel computation of (co-)variance:
 207: // https://dl.acm.org/doi/10.1145/3221269.3223036
 208: //
 209: // Parallel computation of variance:
 210: // http://i.stanford.edu/pub/cstr/reports/cs/tr/79/773/CS-TR-79-773.pdf
 211: template<typename ReturnType, typename ForwardIterator>
 212: ReturnType correlation_coefficient_parallel_impl(ForwardIterator u_begin, ForwardIterator u_end, ForwardIterator v_begin, ForwardIterator v_end)
 213: {
 214:     using Real = typename std::tuple_element<0, ReturnType>::type;
 215: 
 216:     const auto u_elements = std::distance(u_begin, u_end);
 217:     const auto v_elements = std::distance(v_begin, v_end);
 218: 
 219:     if(u_elements != v_elements)
 220:     {
````
- **L201 EN**: Returns from the current function with `std::make_tuple(mu_u, Qu, mu_v, Qv, cov, rho, Real(i))`.
  - **L201 CN**: 以 `std::make_tuple(mu_u, Qu, mu_v, Qv, cov, rho, Real(i))` 从当前函数返回。
- **L202 EN**: Closes the current lexical scope or compound statement.
  - **L202 CN**: 结束当前词法作用域或复合语句块。
- **L203 EN**: Blank line separating nearby declarations or logic.
  - **L203 CN**: 空行，用于分隔相邻声明或逻辑。
- **L204 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_EXEC_COMPATIBLE`.
  - **L204 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_EXEC_COMPATIBLE`。
- **L205 EN**: Blank line separating nearby declarations or logic.
  - **L205 CN**: 空行，用于分隔相邻声明或逻辑。
- **L206 EN**: Comment documents nearby intent or usage notes: `Numerically stable parallel computation of (co-)variance:`.
  - **L206 CN**: 注释说明附近代码的意图或使用说明：`Numerically stable parallel computation of (co-)variance:`。
- **L207 EN**: Comment documents nearby intent or usage notes: `https://dl.acm.org/doi/10.1145/3221269.3223036`.
  - **L207 CN**: 注释说明附近代码的意图或使用说明：`https://dl.acm.org/doi/10.1145/3221269.3223036`。
- **L208 EN**: Separator comment used for visual grouping.
  - **L208 CN**: 分隔注释，用于视觉分组。
- **L209 EN**: Comment documents nearby intent or usage notes: `Parallel computation of variance:`.
  - **L209 CN**: 注释说明附近代码的意图或使用说明：`Parallel computation of variance:`。
- **L210 EN**: Comment documents nearby intent or usage notes: `http://i.stanford.edu/pub/cstr/reports/cs/tr/79/773/CS-TR-79-773.pdf`.
  - **L210 CN**: 注释说明附近代码的意图或使用说明：`http://i.stanford.edu/pub/cstr/reports/cs/tr/79/773/CS-TR-79-773.pdf`。
- **L211 EN**: Introduces template parameters or specialization context: `template<typename ReturnType, typename ForwardIterator>`.
  - **L211 CN**: 为后续声明引入模板参数或特化上下文：`template<typename ReturnType, typename ForwardIterator>`。
- **L212 EN**: Continues logic associated with callable symbol `correlation_coefficient_parallel_impl`.
  - **L212 CN**: 继续与可调用符号 `correlation_coefficient_parallel_impl` 相关的逻辑。
- **L213 EN**: Opens a new lexical scope or compound statement.
  - **L213 CN**: 打开一个新的词法作用域或复合语句块。
- **L214 EN**: Defines alias `Real` to simplify later code.
  - **L214 CN**: 定义别名 `Real` 以简化后续代码。
- **L215 EN**: Blank line separating nearby declarations or logic.
  - **L215 CN**: 空行，用于分隔相邻声明或逻辑。
- **L216 EN**: Initializes variable `u_elements` from the right-hand expression.
  - **L216 CN**: 使用右侧表达式初始化变量 `u_elements`。
- **L217 EN**: Initializes variable `v_elements` from the right-hand expression.
  - **L217 CN**: 使用右侧表达式初始化变量 `v_elements`。
- **L218 EN**: Blank line separating nearby declarations or logic.
  - **L218 CN**: 空行，用于分隔相邻声明或逻辑。
- **L219 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L219 CN**: 开始 `if` 控制流语句并计算其条件。
- **L220 EN**: Opens a new lexical scope or compound statement.
  - **L220 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 221-240 / 第 221-240 行

````cpp
 221:         throw std::domain_error("The size of each sample set must be the same to compute covariance");
 222:     }
 223: 
 224:     const unsigned max_concurrency = std::thread::hardware_concurrency() == 0 ? 2u : std::thread::hardware_concurrency();
 225:     unsigned num_threads = 2u;
 226:     
 227:     // 3.25 comes from benchmarking. See boost/math/reporting/performance/bivariate_statistics_performance.cpp
 228:     // Threading is faster for: 10 + 3.25e-3 N/j <= 3.25e-3N => N >= 10^4j/3.25(j-1).
 229:     const auto parallel_lower_bound = 10e4*max_concurrency/(3.25*(max_concurrency-1));
 230:     const auto parallel_upper_bound = 10e4*2/3.25; // j = 2
 231: 
 232:     // https://lemire.me/blog/2020/01/30/cost-of-a-thread-in-c-under-linux/
 233:     if(u_elements < parallel_lower_bound)
 234:     {
 235:         return correlation_coefficient_seq_impl<ReturnType>(u_begin, u_end, v_begin, v_end);
 236:     }
 237:     else if(u_elements >= parallel_upper_bound)
 238:     {
 239:         num_threads = max_concurrency;
 240:     }
````
- **L221 EN**: Throws an exception object or error marker: `throw std::domain_error("The size of each sample set must be the same to compute covariance");`.
  - **L221 CN**: 抛出异常对象或错误标记：`throw std::domain_error("The size of each sample set must be the same to compute covariance");`。
- **L222 EN**: Closes the current lexical scope or compound statement.
  - **L222 CN**: 结束当前词法作用域或复合语句块。
- **L223 EN**: Blank line separating nearby declarations or logic.
  - **L223 CN**: 空行，用于分隔相邻声明或逻辑。
- **L224 EN**: Initializes variable `max_concurrency` from the right-hand expression.
  - **L224 CN**: 使用右侧表达式初始化变量 `max_concurrency`。
- **L225 EN**: Initializes variable `num_threads` from the right-hand expression.
  - **L225 CN**: 使用右侧表达式初始化变量 `num_threads`。
- **L226 EN**: Blank line separating nearby declarations or logic.
  - **L226 CN**: 空行，用于分隔相邻声明或逻辑。
- **L227 EN**: Comment documents nearby intent or usage notes: `3.25 comes from benchmarking. See boost/math/reporting/performance/bivariate_statistics_performance.cpp`.
  - **L227 CN**: 注释说明附近代码的意图或使用说明：`3.25 comes from benchmarking. See boost/math/reporting/performance/bivariate_statistics_performance.cpp`。
- **L228 EN**: Comment documents nearby intent or usage notes: `Threading is faster for: 10 + 3.25e-3 N/j <= 3.25e-3N => N >= 10^4j/3.25(j-1).`.
  - **L228 CN**: 注释说明附近代码的意图或使用说明：`Threading is faster for: 10 + 3.25e-3 N/j <= 3.25e-3N => N >= 10^4j/3.25(j-1).`。
- **L229 EN**: Initializes variable `parallel_lower_bound` from the right-hand expression.
  - **L229 CN**: 使用右侧表达式初始化变量 `parallel_lower_bound`。
- **L230 EN**: Continues the surrounding expression or declaration: `const auto parallel_upper_bound = 10e4*2/3.25; // j = 2`.
  - **L230 CN**: 继续构造周围的表达式或声明：`const auto parallel_upper_bound = 10e4*2/3.25; // j = 2`。
- **L231 EN**: Blank line separating nearby declarations or logic.
  - **L231 CN**: 空行，用于分隔相邻声明或逻辑。
- **L232 EN**: Comment documents nearby intent or usage notes: `https://lemire.me/blog/2020/01/30/cost-of-a-thread-in-c-under-linux/`.
  - **L232 CN**: 注释说明附近代码的意图或使用说明：`https://lemire.me/blog/2020/01/30/cost-of-a-thread-in-c-under-linux/`。
- **L233 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L233 CN**: 开始 `if` 控制流语句并计算其条件。
- **L234 EN**: Opens a new lexical scope or compound statement.
  - **L234 CN**: 打开一个新的词法作用域或复合语句块。
- **L235 EN**: Returns from the current function with `correlation_coefficient_seq_impl<ReturnType>(u_begin, u_end, v_begin, v_end)`.
  - **L235 CN**: 以 `correlation_coefficient_seq_impl<ReturnType>(u_begin, u_end, v_begin, v_end)` 从当前函数返回。
- **L236 EN**: Closes the current lexical scope or compound statement.
  - **L236 CN**: 结束当前词法作用域或复合语句块。
- **L237 EN**: Starts the alternative branch of the preceding conditional.
  - **L237 CN**: 开始前一个条件语句的备选分支。
- **L238 EN**: Opens a new lexical scope or compound statement.
  - **L238 CN**: 打开一个新的词法作用域或复合语句块。
- **L239 EN**: Executes a standalone statement or declaration: `num_threads = max_concurrency;`.
  - **L239 CN**: 执行一条独立语句或声明：`num_threads = max_concurrency;`。
- **L240 EN**: Closes the current lexical scope or compound statement.
  - **L240 CN**: 结束当前词法作用域或复合语句块。

### Lines 241-260 / 第 241-260 行

````cpp
 241:     else
 242:     {
 243:         for(unsigned i = 3; i < max_concurrency; ++i)
 244:         {
 245:             if(parallel_lower_bound < 10e4*i/(3.25*(i-1)))
 246:             {
 247:                 num_threads = i;
 248:                 break;
 249:             }
 250:         }
 251:     }
 252: 
 253:     std::vector<std::future<ReturnType>> future_manager;
 254:     const auto elements_per_thread = std::ceil(static_cast<double>(u_elements)/num_threads);
 255: 
 256:     ForwardIterator u_it = u_begin;
 257:     ForwardIterator v_it = v_begin;
 258: 
 259:     for(std::size_t i = 0; i < num_threads - 1; ++i)
 260:     {
````
- **L241 EN**: Starts the alternative branch of the preceding conditional.
  - **L241 CN**: 开始前一个条件语句的备选分支。
- **L242 EN**: Opens a new lexical scope or compound statement.
  - **L242 CN**: 打开一个新的词法作用域或复合语句块。
- **L243 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L243 CN**: 开始 `for` 控制流语句并计算其条件。
- **L244 EN**: Opens a new lexical scope or compound statement.
  - **L244 CN**: 打开一个新的词法作用域或复合语句块。
- **L245 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L245 CN**: 开始 `if` 控制流语句并计算其条件。
- **L246 EN**: Opens a new lexical scope or compound statement.
  - **L246 CN**: 打开一个新的词法作用域或复合语句块。
- **L247 EN**: Executes a standalone statement or declaration: `num_threads = i;`.
  - **L247 CN**: 执行一条独立语句或声明：`num_threads = i;`。
- **L248 EN**: Exits the nearest loop or switch statement.
  - **L248 CN**: 退出最近的循环或 switch 语句。
- **L249 EN**: Closes the current lexical scope or compound statement.
  - **L249 CN**: 结束当前词法作用域或复合语句块。
- **L250 EN**: Closes the current lexical scope or compound statement.
  - **L250 CN**: 结束当前词法作用域或复合语句块。
- **L251 EN**: Closes the current lexical scope or compound statement.
  - **L251 CN**: 结束当前词法作用域或复合语句块。
- **L252 EN**: Blank line separating nearby declarations or logic.
  - **L252 CN**: 空行，用于分隔相邻声明或逻辑。
- **L253 EN**: Executes a standalone statement or declaration: `std::vector<std::future<ReturnType>> future_manager;`.
  - **L253 CN**: 执行一条独立语句或声明：`std::vector<std::future<ReturnType>> future_manager;`。
- **L254 EN**: Initializes variable `elements_per_thread` from the right-hand expression.
  - **L254 CN**: 使用右侧表达式初始化变量 `elements_per_thread`。
- **L255 EN**: Blank line separating nearby declarations or logic.
  - **L255 CN**: 空行，用于分隔相邻声明或逻辑。
- **L256 EN**: Initializes variable `u_it` from the right-hand expression.
  - **L256 CN**: 使用右侧表达式初始化变量 `u_it`。
- **L257 EN**: Initializes variable `v_it` from the right-hand expression.
  - **L257 CN**: 使用右侧表达式初始化变量 `v_it`。
- **L258 EN**: Blank line separating nearby declarations or logic.
  - **L258 CN**: 空行，用于分隔相邻声明或逻辑。
- **L259 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L259 CN**: 开始 `for` 控制流语句并计算其条件。
- **L260 EN**: Opens a new lexical scope or compound statement.
  - **L260 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 261-280 / 第 261-280 行

````cpp
 261:         future_manager.emplace_back(std::async(std::launch::async | std::launch::deferred, [u_it, v_it, elements_per_thread]() -> ReturnType
 262:         {
 263:             return correlation_coefficient_seq_impl<ReturnType>(u_it, std::next(u_it, elements_per_thread), v_it, std::next(v_it, elements_per_thread));
 264:         }));
 265:         u_it = std::next(u_it, elements_per_thread);
 266:         v_it = std::next(v_it, elements_per_thread);
 267:     }
 268: 
 269:     future_manager.emplace_back(std::async(std::launch::async | std::launch::deferred, [u_it, u_end, v_it, v_end]() -> ReturnType
 270:     {
 271:         return correlation_coefficient_seq_impl<ReturnType>(u_it, u_end, v_it, v_end);
 272:     }));
 273: 
 274:     ReturnType temp = future_manager[0].get();
 275:     Real mu_u_a = std::get<0>(temp);
 276:     Real Qu_a = std::get<1>(temp);
 277:     Real mu_v_a = std::get<2>(temp);
 278:     Real Qv_a = std::get<3>(temp);
 279:     Real cov_a = std::get<4>(temp);
 280:     Real n_a = std::get<6>(temp);
````
- **L261 EN**: Continues logic associated with callable symbol `emplace_back`.
  - **L261 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L262 EN**: Opens a new lexical scope or compound statement.
  - **L262 CN**: 打开一个新的词法作用域或复合语句块。
- **L263 EN**: Returns from the current function with `correlation_coefficient_seq_impl<ReturnType>(u_it, std::next(u_it, elements_per_thread), v_it, std::next(v_it, elements_per_thread))`.
  - **L263 CN**: 以 `correlation_coefficient_seq_impl<ReturnType>(u_it, std::next(u_it, elements_per_thread), v_it, std::next(v_it, elements_per_thread))` 从当前函数返回。
- **L264 EN**: Executes a standalone statement or declaration: `}));`.
  - **L264 CN**: 执行一条独立语句或声明：`}));`。
- **L265 EN**: Executes a call or declaration centered on `std::next`.
  - **L265 CN**: 执行以 `std::next` 为核心的调用或声明。
- **L266 EN**: Executes a call or declaration centered on `std::next`.
  - **L266 CN**: 执行以 `std::next` 为核心的调用或声明。
- **L267 EN**: Closes the current lexical scope or compound statement.
  - **L267 CN**: 结束当前词法作用域或复合语句块。
- **L268 EN**: Blank line separating nearby declarations or logic.
  - **L268 CN**: 空行，用于分隔相邻声明或逻辑。
- **L269 EN**: Continues logic associated with callable symbol `emplace_back`.
  - **L269 CN**: 继续与可调用符号 `emplace_back` 相关的逻辑。
- **L270 EN**: Opens a new lexical scope or compound statement.
  - **L270 CN**: 打开一个新的词法作用域或复合语句块。
- **L271 EN**: Returns from the current function with `correlation_coefficient_seq_impl<ReturnType>(u_it, u_end, v_it, v_end)`.
  - **L271 CN**: 以 `correlation_coefficient_seq_impl<ReturnType>(u_it, u_end, v_it, v_end)` 从当前函数返回。
- **L272 EN**: Executes a standalone statement or declaration: `}));`.
  - **L272 CN**: 执行一条独立语句或声明：`}));`。
- **L273 EN**: Blank line separating nearby declarations or logic.
  - **L273 CN**: 空行，用于分隔相邻声明或逻辑。
- **L274 EN**: Initializes variable `temp` from the right-hand expression.
  - **L274 CN**: 使用右侧表达式初始化变量 `temp`。
- **L275 EN**: Initializes variable `mu_u_a` from the right-hand expression.
  - **L275 CN**: 使用右侧表达式初始化变量 `mu_u_a`。
- **L276 EN**: Initializes variable `Qu_a` from the right-hand expression.
  - **L276 CN**: 使用右侧表达式初始化变量 `Qu_a`。
- **L277 EN**: Initializes variable `mu_v_a` from the right-hand expression.
  - **L277 CN**: 使用右侧表达式初始化变量 `mu_v_a`。
- **L278 EN**: Initializes variable `Qv_a` from the right-hand expression.
  - **L278 CN**: 使用右侧表达式初始化变量 `Qv_a`。
- **L279 EN**: Initializes variable `cov_a` from the right-hand expression.
  - **L279 CN**: 使用右侧表达式初始化变量 `cov_a`。
- **L280 EN**: Initializes variable `n_a` from the right-hand expression.
  - **L280 CN**: 使用右侧表达式初始化变量 `n_a`。

### Lines 281-300 / 第 281-300 行

````cpp
 281: 
 282:     for(std::size_t i = 1; i < future_manager.size(); ++i)
 283:     {
 284:         temp = future_manager[i].get();
 285:         Real mu_u_b = std::get<0>(temp);
 286:         Real Qu_b = std::get<1>(temp);
 287:         Real mu_v_b = std::get<2>(temp);
 288:         Real Qv_b = std::get<3>(temp);
 289:         Real cov_b = std::get<4>(temp);
 290:         Real n_b = std::get<6>(temp);
 291: 
 292:         const Real n_ab = n_a + n_b;
 293:         const Real delta_u = mu_u_b - mu_u_a;
 294:         const Real delta_v = mu_v_b - mu_v_a;
 295: 
 296:         cov_a = cov_a + cov_b + (-delta_u)*(-delta_v)*((n_a*n_b)/n_ab);
 297:         mu_u_a = mu_u_a + delta_u*(n_b/n_ab);
 298:         mu_v_a = mu_v_a + delta_v*(n_b/n_ab);
 299:         Qu_a = Qu_a + Qu_b + delta_u*delta_u*((n_a*n_b)/n_ab);
 300:         Qv_b = Qv_a + Qv_b + delta_v*delta_v*((n_a*n_b)/n_ab);
````
- **L281 EN**: Blank line separating nearby declarations or logic.
  - **L281 CN**: 空行，用于分隔相邻声明或逻辑。
- **L282 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L282 CN**: 开始 `for` 控制流语句并计算其条件。
- **L283 EN**: Opens a new lexical scope or compound statement.
  - **L283 CN**: 打开一个新的词法作用域或复合语句块。
- **L284 EN**: Executes a call or declaration centered on `future_manager[i].get`.
  - **L284 CN**: 执行以 `future_manager[i].get` 为核心的调用或声明。
- **L285 EN**: Initializes variable `mu_u_b` from the right-hand expression.
  - **L285 CN**: 使用右侧表达式初始化变量 `mu_u_b`。
- **L286 EN**: Initializes variable `Qu_b` from the right-hand expression.
  - **L286 CN**: 使用右侧表达式初始化变量 `Qu_b`。
- **L287 EN**: Initializes variable `mu_v_b` from the right-hand expression.
  - **L287 CN**: 使用右侧表达式初始化变量 `mu_v_b`。
- **L288 EN**: Initializes variable `Qv_b` from the right-hand expression.
  - **L288 CN**: 使用右侧表达式初始化变量 `Qv_b`。
- **L289 EN**: Initializes variable `cov_b` from the right-hand expression.
  - **L289 CN**: 使用右侧表达式初始化变量 `cov_b`。
- **L290 EN**: Initializes variable `n_b` from the right-hand expression.
  - **L290 CN**: 使用右侧表达式初始化变量 `n_b`。
- **L291 EN**: Blank line separating nearby declarations or logic.
  - **L291 CN**: 空行，用于分隔相邻声明或逻辑。
- **L292 EN**: Initializes variable `n_ab` from the right-hand expression.
  - **L292 CN**: 使用右侧表达式初始化变量 `n_ab`。
- **L293 EN**: Initializes variable `delta_u` from the right-hand expression.
  - **L293 CN**: 使用右侧表达式初始化变量 `delta_u`。
- **L294 EN**: Initializes variable `delta_v` from the right-hand expression.
  - **L294 CN**: 使用右侧表达式初始化变量 `delta_v`。
- **L295 EN**: Blank line separating nearby declarations or logic.
  - **L295 CN**: 空行，用于分隔相邻声明或逻辑。
- **L296 EN**: Executes a call or declaration centered on `+`.
  - **L296 CN**: 执行以 `+` 为核心的调用或声明。
- **L297 EN**: Executes a call or declaration centered on `delta_u*`.
  - **L297 CN**: 执行以 `delta_u*` 为核心的调用或声明。
- **L298 EN**: Executes a call or declaration centered on `delta_v*`.
  - **L298 CN**: 执行以 `delta_v*` 为核心的调用或声明。
- **L299 EN**: Executes a call or declaration centered on `delta_u*delta_u*`.
  - **L299 CN**: 执行以 `delta_u*delta_u*` 为核心的调用或声明。
- **L300 EN**: Executes a call or declaration centered on `delta_v*delta_v*`.
  - **L300 CN**: 执行以 `delta_v*delta_v*` 为核心的调用或声明。

### Lines 301-320 / 第 301-320 行

````cpp
 301:         n_a = n_ab;
 302:     }
 303: 
 304:     // If one dataset is constant, then the correlation coefficient is undefined.
 305:     // See https://stats.stackexchange.com/questions/23676/normalized-correlation-with-a-constant-vector
 306:     // Thanks to zbjornson for pointing this out.
 307:     if (Qu_a == 0 || Qv_a == 0)
 308:     {
 309:         return std::make_tuple(mu_u_a, Qu_a, mu_v_a, Qv_a, cov_a, std::numeric_limits<Real>::quiet_NaN(), n_a);
 310:     }
 311: 
 312:     // Make sure rho in [-1, 1], even in the presence of numerical noise.
 313:     Real rho = cov_a/sqrt(Qu_a*Qv_a);
 314:     if (rho > 1) {
 315:         rho = 1;
 316:     }
 317:     if (rho < -1) {
 318:         rho = -1;
 319:     }
 320: 
````
- **L301 EN**: Executes a standalone statement or declaration: `n_a = n_ab;`.
  - **L301 CN**: 执行一条独立语句或声明：`n_a = n_ab;`。
- **L302 EN**: Closes the current lexical scope or compound statement.
  - **L302 CN**: 结束当前词法作用域或复合语句块。
- **L303 EN**: Blank line separating nearby declarations or logic.
  - **L303 CN**: 空行，用于分隔相邻声明或逻辑。
- **L304 EN**: Comment documents nearby intent or usage notes: `If one dataset is constant, then the correlation coefficient is undefined.`.
  - **L304 CN**: 注释说明附近代码的意图或使用说明：`If one dataset is constant, then the correlation coefficient is undefined.`。
- **L305 EN**: Comment documents nearby intent or usage notes: `See https://stats.stackexchange.com/questions/23676/normalized-correlation-with-a-constant-vector`.
  - **L305 CN**: 注释说明附近代码的意图或使用说明：`See https://stats.stackexchange.com/questions/23676/normalized-correlation-with-a-constant-vector`。
- **L306 EN**: Comment documents nearby intent or usage notes: `Thanks to zbjornson for pointing this out.`.
  - **L306 CN**: 注释说明附近代码的意图或使用说明：`Thanks to zbjornson for pointing this out.`。
- **L307 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L307 CN**: 开始 `if` 控制流语句并计算其条件。
- **L308 EN**: Opens a new lexical scope or compound statement.
  - **L308 CN**: 打开一个新的词法作用域或复合语句块。
- **L309 EN**: Returns from the current function with `std::make_tuple(mu_u_a, Qu_a, mu_v_a, Qv_a, cov_a, std::numeric_limits<Real>::quiet_NaN(), n_a)`.
  - **L309 CN**: 以 `std::make_tuple(mu_u_a, Qu_a, mu_v_a, Qv_a, cov_a, std::numeric_limits<Real>::quiet_NaN(), n_a)` 从当前函数返回。
- **L310 EN**: Closes the current lexical scope or compound statement.
  - **L310 CN**: 结束当前词法作用域或复合语句块。
- **L311 EN**: Blank line separating nearby declarations or logic.
  - **L311 CN**: 空行，用于分隔相邻声明或逻辑。
- **L312 EN**: Comment documents nearby intent or usage notes: `Make sure rho in [-1, 1], even in the presence of numerical noise.`.
  - **L312 CN**: 注释说明附近代码的意图或使用说明：`Make sure rho in [-1, 1], even in the presence of numerical noise.`。
- **L313 EN**: Initializes variable `rho` from the right-hand expression.
  - **L313 CN**: 使用右侧表达式初始化变量 `rho`。
- **L314 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L314 CN**: 开始 `if` 控制流语句并计算其条件。
- **L315 EN**: Executes a standalone statement or declaration: `rho = 1;`.
  - **L315 CN**: 执行一条独立语句或声明：`rho = 1;`。
- **L316 EN**: Closes the current lexical scope or compound statement.
  - **L316 CN**: 结束当前词法作用域或复合语句块。
- **L317 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L317 CN**: 开始 `if` 控制流语句并计算其条件。
- **L318 EN**: Executes a standalone statement or declaration: `rho = -1;`.
  - **L318 CN**: 执行一条独立语句或声明：`rho = -1;`。
- **L319 EN**: Closes the current lexical scope or compound statement.
  - **L319 CN**: 结束当前词法作用域或复合语句块。
- **L320 EN**: Blank line separating nearby declarations or logic.
  - **L320 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 321-340 / 第 321-340 行

````cpp
 321:     return std::make_tuple(mu_u_a, Qu_a, mu_v_a, Qv_a, cov_a, rho, n_a);
 322: }
 323: 
 324: #endif // BOOST_MATH_EXEC_COMPATIBLE
 325: 
 326: } // namespace detail
 327: 
 328: #ifdef BOOST_MATH_EXEC_COMPATIBLE
 329: 
 330: template<typename ExecutionPolicy, typename Container, typename Real = typename Container::value_type>
 331: inline auto means_and_covariance(ExecutionPolicy&& exec, Container const & u, Container const & v)
 332: {
 333:     if constexpr (std::is_same_v<std::remove_reference_t<decltype(exec)>, decltype(std::execution::seq)>)
 334:     {
 335:         if constexpr (std::is_integral_v<Real>)
 336:         {
 337:             using ReturnType = std::tuple<double, double, double, double>;
 338:             ReturnType temp = detail::means_and_covariance_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v));
 339:             return std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp));
 340:         }
````
- **L321 EN**: Returns from the current function with `std::make_tuple(mu_u_a, Qu_a, mu_v_a, Qv_a, cov_a, rho, n_a)`.
  - **L321 CN**: 以 `std::make_tuple(mu_u_a, Qu_a, mu_v_a, Qv_a, cov_a, rho, n_a)` 从当前函数返回。
- **L322 EN**: Closes the current lexical scope or compound statement.
  - **L322 CN**: 结束当前词法作用域或复合语句块。
- **L323 EN**: Blank line separating nearby declarations or logic.
  - **L323 CN**: 空行，用于分隔相邻声明或逻辑。
- **L324 EN**: Closes the current preprocessor conditional block or header guard.
  - **L324 CN**: 结束当前预处理条件块或头文件保护。
- **L325 EN**: Blank line separating nearby declarations or logic.
  - **L325 CN**: 空行，用于分隔相邻声明或逻辑。
- **L326 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace detail`.
  - **L326 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace detail`。
- **L327 EN**: Blank line separating nearby declarations or logic.
  - **L327 CN**: 空行，用于分隔相邻声明或逻辑。
- **L328 EN**: Starts a preprocessor conditional block: `#ifdef BOOST_MATH_EXEC_COMPATIBLE`.
  - **L328 CN**: 开始一个预处理条件块：`#ifdef BOOST_MATH_EXEC_COMPATIBLE`。
- **L329 EN**: Blank line separating nearby declarations or logic.
  - **L329 CN**: 空行，用于分隔相邻声明或逻辑。
- **L330 EN**: Introduces template parameters or specialization context: `template<typename ExecutionPolicy, typename Container, typename Real = typename Container::value_type>`.
  - **L330 CN**: 为后续声明引入模板参数或特化上下文：`template<typename ExecutionPolicy, typename Container, typename Real = typename Container::value_type>`。
- **L331 EN**: Continues logic associated with callable symbol `means_and_covariance`.
  - **L331 CN**: 继续与可调用符号 `means_and_covariance` 相关的逻辑。
- **L332 EN**: Opens a new lexical scope or compound statement.
  - **L332 CN**: 打开一个新的词法作用域或复合语句块。
- **L333 EN**: Uses standard-library type traits or utility templates to shape generic behavior.
  - **L333 CN**: 使用标准库类型 traits 或工具模板来塑造泛型行为。
- **L334 EN**: Opens a new lexical scope or compound statement.
  - **L334 CN**: 打开一个新的词法作用域或复合语句块。
- **L335 EN**: Uses standard-library type traits or utility templates to shape generic behavior.
  - **L335 CN**: 使用标准库类型 traits 或工具模板来塑造泛型行为。
- **L336 EN**: Opens a new lexical scope or compound statement.
  - **L336 CN**: 打开一个新的词法作用域或复合语句块。
- **L337 EN**: Defines alias `ReturnType` to simplify later code.
  - **L337 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L338 EN**: Initializes variable `temp` from the right-hand expression.
  - **L338 CN**: 使用右侧表达式初始化变量 `temp`。
- **L339 EN**: Returns from the current function with `std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp))`.
  - **L339 CN**: 以 `std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp))` 从当前函数返回。
- **L340 EN**: Closes the current lexical scope or compound statement.
  - **L340 CN**: 结束当前词法作用域或复合语句块。

### Lines 341-360 / 第 341-360 行

````cpp
 341:         else
 342:         {
 343:             using ReturnType = std::tuple<Real, Real, Real, Real>;
 344:             ReturnType temp = detail::means_and_covariance_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v));
 345:             return std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp));
 346:         }
 347:     }
 348:     else
 349:     {
 350:         if constexpr (std::is_integral_v<Real>)
 351:         {
 352:             using ReturnType = std::tuple<double, double, double, double>;
 353:             ReturnType temp = detail::means_and_covariance_parallel_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v));
 354:             return std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp));
 355:         }
 356:         else
 357:         {
 358:             using ReturnType = std::tuple<Real, Real, Real, Real>;
 359:             ReturnType temp = detail::means_and_covariance_parallel_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v));
 360:             return std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp));
````
- **L341 EN**: Starts the alternative branch of the preceding conditional.
  - **L341 CN**: 开始前一个条件语句的备选分支。
- **L342 EN**: Opens a new lexical scope or compound statement.
  - **L342 CN**: 打开一个新的词法作用域或复合语句块。
- **L343 EN**: Defines alias `ReturnType` to simplify later code.
  - **L343 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L344 EN**: Initializes variable `temp` from the right-hand expression.
  - **L344 CN**: 使用右侧表达式初始化变量 `temp`。
- **L345 EN**: Returns from the current function with `std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp))`.
  - **L345 CN**: 以 `std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp))` 从当前函数返回。
- **L346 EN**: Closes the current lexical scope or compound statement.
  - **L346 CN**: 结束当前词法作用域或复合语句块。
- **L347 EN**: Closes the current lexical scope or compound statement.
  - **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Starts the alternative branch of the preceding conditional.
  - **L348 CN**: 开始前一个条件语句的备选分支。
- **L349 EN**: Opens a new lexical scope or compound statement.
  - **L349 CN**: 打开一个新的词法作用域或复合语句块。
- **L350 EN**: Uses standard-library type traits or utility templates to shape generic behavior.
  - **L350 CN**: 使用标准库类型 traits 或工具模板来塑造泛型行为。
- **L351 EN**: Opens a new lexical scope or compound statement.
  - **L351 CN**: 打开一个新的词法作用域或复合语句块。
- **L352 EN**: Defines alias `ReturnType` to simplify later code.
  - **L352 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L353 EN**: Initializes variable `temp` from the right-hand expression.
  - **L353 CN**: 使用右侧表达式初始化变量 `temp`。
- **L354 EN**: Returns from the current function with `std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp))`.
  - **L354 CN**: 以 `std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp))` 从当前函数返回。
- **L355 EN**: Closes the current lexical scope or compound statement.
  - **L355 CN**: 结束当前词法作用域或复合语句块。
- **L356 EN**: Starts the alternative branch of the preceding conditional.
  - **L356 CN**: 开始前一个条件语句的备选分支。
- **L357 EN**: Opens a new lexical scope or compound statement.
  - **L357 CN**: 打开一个新的词法作用域或复合语句块。
- **L358 EN**: Defines alias `ReturnType` to simplify later code.
  - **L358 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L359 EN**: Initializes variable `temp` from the right-hand expression.
  - **L359 CN**: 使用右侧表达式初始化变量 `temp`。
- **L360 EN**: Returns from the current function with `std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp))`.
  - **L360 CN**: 以 `std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp))` 从当前函数返回。

### Lines 361-380 / 第 361-380 行

````cpp
 361:         }
 362:     }
 363: }
 364: 
 365: template<typename Container>
 366: inline auto means_and_covariance(Container const & u, Container const & v)
 367: {
 368:     return means_and_covariance(std::execution::seq, u, v);
 369: }
 370: 
 371: template<typename ExecutionPolicy, typename Container>
 372: inline auto covariance(ExecutionPolicy&& exec, Container const & u, Container const & v)
 373: {
 374:     return std::get<2>(means_and_covariance(exec, u, v));
 375: }
 376: 
 377: template<typename Container>
 378: inline auto covariance(Container const & u, Container const & v)
 379: {
 380:     return covariance(std::execution::seq, u, v);
````
- **L361 EN**: Closes the current lexical scope or compound statement.
  - **L361 CN**: 结束当前词法作用域或复合语句块。
- **L362 EN**: Closes the current lexical scope or compound statement.
  - **L362 CN**: 结束当前词法作用域或复合语句块。
- **L363 EN**: Closes the current lexical scope or compound statement.
  - **L363 CN**: 结束当前词法作用域或复合语句块。
- **L364 EN**: Blank line separating nearby declarations or logic.
  - **L364 CN**: 空行，用于分隔相邻声明或逻辑。
- **L365 EN**: Introduces template parameters or specialization context: `template<typename Container>`.
  - **L365 CN**: 为后续声明引入模板参数或特化上下文：`template<typename Container>`。
- **L366 EN**: Continues logic associated with callable symbol `means_and_covariance`.
  - **L366 CN**: 继续与可调用符号 `means_and_covariance` 相关的逻辑。
- **L367 EN**: Opens a new lexical scope or compound statement.
  - **L367 CN**: 打开一个新的词法作用域或复合语句块。
- **L368 EN**: Returns from the current function with `means_and_covariance(std::execution::seq, u, v)`.
  - **L368 CN**: 以 `means_and_covariance(std::execution::seq, u, v)` 从当前函数返回。
- **L369 EN**: Closes the current lexical scope or compound statement.
  - **L369 CN**: 结束当前词法作用域或复合语句块。
- **L370 EN**: Blank line separating nearby declarations or logic.
  - **L370 CN**: 空行，用于分隔相邻声明或逻辑。
- **L371 EN**: Introduces template parameters or specialization context: `template<typename ExecutionPolicy, typename Container>`.
  - **L371 CN**: 为后续声明引入模板参数或特化上下文：`template<typename ExecutionPolicy, typename Container>`。
- **L372 EN**: Continues logic associated with callable symbol `covariance`.
  - **L372 CN**: 继续与可调用符号 `covariance` 相关的逻辑。
- **L373 EN**: Opens a new lexical scope or compound statement.
  - **L373 CN**: 打开一个新的词法作用域或复合语句块。
- **L374 EN**: Returns from the current function with `std::get<2>(means_and_covariance(exec, u, v))`.
  - **L374 CN**: 以 `std::get<2>(means_and_covariance(exec, u, v))` 从当前函数返回。
- **L375 EN**: Closes the current lexical scope or compound statement.
  - **L375 CN**: 结束当前词法作用域或复合语句块。
- **L376 EN**: Blank line separating nearby declarations or logic.
  - **L376 CN**: 空行，用于分隔相邻声明或逻辑。
- **L377 EN**: Introduces template parameters or specialization context: `template<typename Container>`.
  - **L377 CN**: 为后续声明引入模板参数或特化上下文：`template<typename Container>`。
- **L378 EN**: Continues logic associated with callable symbol `covariance`.
  - **L378 CN**: 继续与可调用符号 `covariance` 相关的逻辑。
- **L379 EN**: Opens a new lexical scope or compound statement.
  - **L379 CN**: 打开一个新的词法作用域或复合语句块。
- **L380 EN**: Returns from the current function with `covariance(std::execution::seq, u, v)`.
  - **L380 CN**: 以 `covariance(std::execution::seq, u, v)` 从当前函数返回。

### Lines 381-400 / 第 381-400 行

````cpp
 381: }
 382: 
 383: template<typename ExecutionPolicy, typename Container, typename Real = typename Container::value_type>
 384: inline auto correlation_coefficient(ExecutionPolicy&& exec, Container const & u, Container const & v)
 385: {
 386:     if constexpr (std::is_same_v<std::remove_reference_t<decltype(exec)>, decltype(std::execution::seq)>)
 387:     {
 388:         if constexpr (std::is_integral_v<Real>)
 389:         {
 390:             using ReturnType = std::tuple<double, double, double, double, double, double, double>;
 391:             return std::get<5>(detail::correlation_coefficient_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)));
 392:         }
 393:         else
 394:         {
 395:             using ReturnType = std::tuple<Real, Real, Real, Real, Real, Real, Real>;
 396:             return std::get<5>(detail::correlation_coefficient_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)));
 397:         }
 398:     }
 399:     else
 400:     {
````
- **L381 EN**: Closes the current lexical scope or compound statement.
  - **L381 CN**: 结束当前词法作用域或复合语句块。
- **L382 EN**: Blank line separating nearby declarations or logic.
  - **L382 CN**: 空行，用于分隔相邻声明或逻辑。
- **L383 EN**: Introduces template parameters or specialization context: `template<typename ExecutionPolicy, typename Container, typename Real = typename Container::value_type>`.
  - **L383 CN**: 为后续声明引入模板参数或特化上下文：`template<typename ExecutionPolicy, typename Container, typename Real = typename Container::value_type>`。
- **L384 EN**: Continues logic associated with callable symbol `correlation_coefficient`.
  - **L384 CN**: 继续与可调用符号 `correlation_coefficient` 相关的逻辑。
- **L385 EN**: Opens a new lexical scope or compound statement.
  - **L385 CN**: 打开一个新的词法作用域或复合语句块。
- **L386 EN**: Uses standard-library type traits or utility templates to shape generic behavior.
  - **L386 CN**: 使用标准库类型 traits 或工具模板来塑造泛型行为。
- **L387 EN**: Opens a new lexical scope or compound statement.
  - **L387 CN**: 打开一个新的词法作用域或复合语句块。
- **L388 EN**: Uses standard-library type traits or utility templates to shape generic behavior.
  - **L388 CN**: 使用标准库类型 traits 或工具模板来塑造泛型行为。
- **L389 EN**: Opens a new lexical scope or compound statement.
  - **L389 CN**: 打开一个新的词法作用域或复合语句块。
- **L390 EN**: Defines alias `ReturnType` to simplify later code.
  - **L390 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L391 EN**: Returns from the current function with `std::get<5>(detail::correlation_coefficient_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))`.
  - **L391 CN**: 以 `std::get<5>(detail::correlation_coefficient_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))` 从当前函数返回。
- **L392 EN**: Closes the current lexical scope or compound statement.
  - **L392 CN**: 结束当前词法作用域或复合语句块。
- **L393 EN**: Starts the alternative branch of the preceding conditional.
  - **L393 CN**: 开始前一个条件语句的备选分支。
- **L394 EN**: Opens a new lexical scope or compound statement.
  - **L394 CN**: 打开一个新的词法作用域或复合语句块。
- **L395 EN**: Defines alias `ReturnType` to simplify later code.
  - **L395 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L396 EN**: Returns from the current function with `std::get<5>(detail::correlation_coefficient_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))`.
  - **L396 CN**: 以 `std::get<5>(detail::correlation_coefficient_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))` 从当前函数返回。
- **L397 EN**: Closes the current lexical scope or compound statement.
  - **L397 CN**: 结束当前词法作用域或复合语句块。
- **L398 EN**: Closes the current lexical scope or compound statement.
  - **L398 CN**: 结束当前词法作用域或复合语句块。
- **L399 EN**: Starts the alternative branch of the preceding conditional.
  - **L399 CN**: 开始前一个条件语句的备选分支。
- **L400 EN**: Opens a new lexical scope or compound statement.
  - **L400 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 401-420 / 第 401-420 行

````cpp
 401:         if constexpr (std::is_integral_v<Real>)
 402:         {
 403:             using ReturnType = std::tuple<double, double, double, double, double, double, double>;
 404:             return std::get<5>(detail::correlation_coefficient_parallel_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)));
 405:         }
 406:         else
 407:         {
 408:             using ReturnType = std::tuple<Real, Real, Real, Real, Real, Real, Real>;
 409:             return std::get<5>(detail::correlation_coefficient_parallel_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)));
 410:         }
 411:     }
 412: }
 413: 
 414: template<typename Container, typename Real = typename Container::value_type>
 415: inline auto correlation_coefficient(Container const & u, Container const & v)
 416: {
 417:     return correlation_coefficient(std::execution::seq, u, v);
 418: }
 419: 
 420: #else // C++11 and single threaded bindings
````
- **L401 EN**: Uses standard-library type traits or utility templates to shape generic behavior.
  - **L401 CN**: 使用标准库类型 traits 或工具模板来塑造泛型行为。
- **L402 EN**: Opens a new lexical scope or compound statement.
  - **L402 CN**: 打开一个新的词法作用域或复合语句块。
- **L403 EN**: Defines alias `ReturnType` to simplify later code.
  - **L403 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L404 EN**: Returns from the current function with `std::get<5>(detail::correlation_coefficient_parallel_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))`.
  - **L404 CN**: 以 `std::get<5>(detail::correlation_coefficient_parallel_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))` 从当前函数返回。
- **L405 EN**: Closes the current lexical scope or compound statement.
  - **L405 CN**: 结束当前词法作用域或复合语句块。
- **L406 EN**: Starts the alternative branch of the preceding conditional.
  - **L406 CN**: 开始前一个条件语句的备选分支。
- **L407 EN**: Opens a new lexical scope or compound statement.
  - **L407 CN**: 打开一个新的词法作用域或复合语句块。
- **L408 EN**: Defines alias `ReturnType` to simplify later code.
  - **L408 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L409 EN**: Returns from the current function with `std::get<5>(detail::correlation_coefficient_parallel_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))`.
  - **L409 CN**: 以 `std::get<5>(detail::correlation_coefficient_parallel_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))` 从当前函数返回。
- **L410 EN**: Closes the current lexical scope or compound statement.
  - **L410 CN**: 结束当前词法作用域或复合语句块。
- **L411 EN**: Closes the current lexical scope or compound statement.
  - **L411 CN**: 结束当前词法作用域或复合语句块。
- **L412 EN**: Closes the current lexical scope or compound statement.
  - **L412 CN**: 结束当前词法作用域或复合语句块。
- **L413 EN**: Blank line separating nearby declarations or logic.
  - **L413 CN**: 空行，用于分隔相邻声明或逻辑。
- **L414 EN**: Introduces template parameters or specialization context: `template<typename Container, typename Real = typename Container::value_type>`.
  - **L414 CN**: 为后续声明引入模板参数或特化上下文：`template<typename Container, typename Real = typename Container::value_type>`。
- **L415 EN**: Continues logic associated with callable symbol `correlation_coefficient`.
  - **L415 CN**: 继续与可调用符号 `correlation_coefficient` 相关的逻辑。
- **L416 EN**: Opens a new lexical scope or compound statement.
  - **L416 CN**: 打开一个新的词法作用域或复合语句块。
- **L417 EN**: Returns from the current function with `correlation_coefficient(std::execution::seq, u, v)`.
  - **L417 CN**: 以 `correlation_coefficient(std::execution::seq, u, v)` 从当前函数返回。
- **L418 EN**: Closes the current lexical scope or compound statement.
  - **L418 CN**: 结束当前词法作用域或复合语句块。
- **L419 EN**: Blank line separating nearby declarations or logic.
  - **L419 CN**: 空行，用于分隔相邻声明或逻辑。
- **L420 EN**: Continues the current preprocessor branch selection.
  - **L420 CN**: 继续当前的预处理分支选择。

### Lines 421-440 / 第 421-440 行

````cpp
 421: 
 422: template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<std::is_integral<Real>::value, bool>::type = true>
 423: inline auto means_and_covariance(Container const & u, Container const & v) -> std::tuple<double, double, double>
 424: {
 425:     using ReturnType = std::tuple<double, double, double, double>;
 426:     ReturnType temp = detail::means_and_covariance_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v));
 427:     return std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp));
 428: }
 429: 
 430: template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<!std::is_integral<Real>::value, bool>::type = true>
 431: inline auto means_and_covariance(Container const & u, Container const & v) -> std::tuple<Real, Real, Real>
 432: {
 433:     using ReturnType = std::tuple<Real, Real, Real, Real>;
 434:     ReturnType temp = detail::means_and_covariance_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v));
 435:     return std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp));
 436: }
 437: 
 438: template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<std::is_integral<Real>::value, bool>::type = true>
 439: inline double covariance(Container const & u, Container const & v)
 440: {
````
- **L421 EN**: Blank line separating nearby declarations or logic.
  - **L421 CN**: 空行，用于分隔相邻声明或逻辑。
- **L422 EN**: Introduces template parameters or specialization context: `template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<std::is_integral<Real>::value, bool>::type = true>`.
  - **L422 CN**: 为后续声明引入模板参数或特化上下文：`template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<std::is_integral<Real>::value, bool>::type = true>`。
- **L423 EN**: Uses standard-library type traits or utility templates to shape generic behavior.
  - **L423 CN**: 使用标准库类型 traits 或工具模板来塑造泛型行为。
- **L424 EN**: Opens a new lexical scope or compound statement.
  - **L424 CN**: 打开一个新的词法作用域或复合语句块。
- **L425 EN**: Defines alias `ReturnType` to simplify later code.
  - **L425 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L426 EN**: Initializes variable `temp` from the right-hand expression.
  - **L426 CN**: 使用右侧表达式初始化变量 `temp`。
- **L427 EN**: Returns from the current function with `std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp))`.
  - **L427 CN**: 以 `std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp))` 从当前函数返回。
- **L428 EN**: Closes the current lexical scope or compound statement.
  - **L428 CN**: 结束当前词法作用域或复合语句块。
- **L429 EN**: Blank line separating nearby declarations or logic.
  - **L429 CN**: 空行，用于分隔相邻声明或逻辑。
- **L430 EN**: Introduces template parameters or specialization context: `template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<!std::is_integral<Real>::value, bool>::type = true>`.
  - **L430 CN**: 为后续声明引入模板参数或特化上下文：`template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<!std::is_integral<Real>::value, bool>::type = true>`。
- **L431 EN**: Uses standard-library type traits or utility templates to shape generic behavior.
  - **L431 CN**: 使用标准库类型 traits 或工具模板来塑造泛型行为。
- **L432 EN**: Opens a new lexical scope or compound statement.
  - **L432 CN**: 打开一个新的词法作用域或复合语句块。
- **L433 EN**: Defines alias `ReturnType` to simplify later code.
  - **L433 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L434 EN**: Initializes variable `temp` from the right-hand expression.
  - **L434 CN**: 使用右侧表达式初始化变量 `temp`。
- **L435 EN**: Returns from the current function with `std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp))`.
  - **L435 CN**: 以 `std::make_tuple(std::get<0>(temp), std::get<1>(temp), std::get<2>(temp))` 从当前函数返回。
- **L436 EN**: Closes the current lexical scope or compound statement.
  - **L436 CN**: 结束当前词法作用域或复合语句块。
- **L437 EN**: Blank line separating nearby declarations or logic.
  - **L437 CN**: 空行，用于分隔相邻声明或逻辑。
- **L438 EN**: Introduces template parameters or specialization context: `template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<std::is_integral<Real>::value, bool>::type = true>`.
  - **L438 CN**: 为后续声明引入模板参数或特化上下文：`template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<std::is_integral<Real>::value, bool>::type = true>`。
- **L439 EN**: Continues logic associated with callable symbol `covariance`.
  - **L439 CN**: 继续与可调用符号 `covariance` 相关的逻辑。
- **L440 EN**: Opens a new lexical scope or compound statement.
  - **L440 CN**: 打开一个新的词法作用域或复合语句块。

### Lines 441-460 / 第 441-460 行

````cpp
 441:     using ReturnType = std::tuple<double, double, double, double>;
 442:     return std::get<2>(detail::means_and_covariance_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)));
 443: }
 444: 
 445: template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<!std::is_integral<Real>::value, bool>::type = true>
 446: inline Real covariance(Container const & u, Container const & v)
 447: {
 448:     using ReturnType = std::tuple<Real, Real, Real, Real>;
 449:     return std::get<2>(detail::means_and_covariance_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)));
 450: }
 451: 
 452: template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<std::is_integral<Real>::value, bool>::type = true>
 453: inline double correlation_coefficient(Container const & u, Container const & v)
 454: {
 455:     using ReturnType = std::tuple<double, double, double, double, double, double, double>;
 456:     return std::get<5>(detail::correlation_coefficient_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)));
 457: }
 458: 
 459: template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<!std::is_integral<Real>::value, bool>::type = true>
 460: inline Real correlation_coefficient(Container const & u, Container const & v)
````
- **L441 EN**: Defines alias `ReturnType` to simplify later code.
  - **L441 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L442 EN**: Returns from the current function with `std::get<2>(detail::means_and_covariance_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))`.
  - **L442 CN**: 以 `std::get<2>(detail::means_and_covariance_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))` 从当前函数返回。
- **L443 EN**: Closes the current lexical scope or compound statement.
  - **L443 CN**: 结束当前词法作用域或复合语句块。
- **L444 EN**: Blank line separating nearby declarations or logic.
  - **L444 CN**: 空行，用于分隔相邻声明或逻辑。
- **L445 EN**: Introduces template parameters or specialization context: `template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<!std::is_integral<Real>::value, bool>::type = true>`.
  - **L445 CN**: 为后续声明引入模板参数或特化上下文：`template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<!std::is_integral<Real>::value, bool>::type = true>`。
- **L446 EN**: Continues logic associated with callable symbol `covariance`.
  - **L446 CN**: 继续与可调用符号 `covariance` 相关的逻辑。
- **L447 EN**: Opens a new lexical scope or compound statement.
  - **L447 CN**: 打开一个新的词法作用域或复合语句块。
- **L448 EN**: Defines alias `ReturnType` to simplify later code.
  - **L448 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L449 EN**: Returns from the current function with `std::get<2>(detail::means_and_covariance_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))`.
  - **L449 CN**: 以 `std::get<2>(detail::means_and_covariance_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))` 从当前函数返回。
- **L450 EN**: Closes the current lexical scope or compound statement.
  - **L450 CN**: 结束当前词法作用域或复合语句块。
- **L451 EN**: Blank line separating nearby declarations or logic.
  - **L451 CN**: 空行，用于分隔相邻声明或逻辑。
- **L452 EN**: Introduces template parameters or specialization context: `template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<std::is_integral<Real>::value, bool>::type = true>`.
  - **L452 CN**: 为后续声明引入模板参数或特化上下文：`template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<std::is_integral<Real>::value, bool>::type = true>`。
- **L453 EN**: Continues logic associated with callable symbol `correlation_coefficient`.
  - **L453 CN**: 继续与可调用符号 `correlation_coefficient` 相关的逻辑。
- **L454 EN**: Opens a new lexical scope or compound statement.
  - **L454 CN**: 打开一个新的词法作用域或复合语句块。
- **L455 EN**: Defines alias `ReturnType` to simplify later code.
  - **L455 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L456 EN**: Returns from the current function with `std::get<5>(detail::correlation_coefficient_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))`.
  - **L456 CN**: 以 `std::get<5>(detail::correlation_coefficient_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))` 从当前函数返回。
- **L457 EN**: Closes the current lexical scope or compound statement.
  - **L457 CN**: 结束当前词法作用域或复合语句块。
- **L458 EN**: Blank line separating nearby declarations or logic.
  - **L458 CN**: 空行，用于分隔相邻声明或逻辑。
- **L459 EN**: Introduces template parameters or specialization context: `template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<!std::is_integral<Real>::value, bool>::type = true>`.
  - **L459 CN**: 为后续声明引入模板参数或特化上下文：`template<typename Container, typename Real = typename Container::value_type, typename std::enable_if<!std::is_integral<Real>::value, bool>::type = true>`。
- **L460 EN**: Continues logic associated with callable symbol `correlation_coefficient`.
  - **L460 CN**: 继续与可调用符号 `correlation_coefficient` 相关的逻辑。

### Lines 461-470 / 第 461-470 行

````cpp
 461: {
 462:     using ReturnType = std::tuple<Real, Real, Real, Real, Real, Real, Real>;
 463:     return std::get<5>(detail::correlation_coefficient_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)));
 464: }
 465: 
 466: #endif
 467: 
 468: }}} // namespace boost::math::statistics
 469: 
 470: #endif
````
- **L461 EN**: Opens a new lexical scope or compound statement.
  - **L461 CN**: 打开一个新的词法作用域或复合语句块。
- **L462 EN**: Defines alias `ReturnType` to simplify later code.
  - **L462 CN**: 定义别名 `ReturnType` 以简化后续代码。
- **L463 EN**: Returns from the current function with `std::get<5>(detail::correlation_coefficient_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))`.
  - **L463 CN**: 以 `std::get<5>(detail::correlation_coefficient_seq_impl<ReturnType>(std::begin(u), std::end(u), std::begin(v), std::end(v)))` 从当前函数返回。
- **L464 EN**: Closes the current lexical scope or compound statement.
  - **L464 CN**: 结束当前词法作用域或复合语句块。
- **L465 EN**: Blank line separating nearby declarations or logic.
  - **L465 CN**: 空行，用于分隔相邻声明或逻辑。
- **L466 EN**: Closes the current preprocessor conditional block or header guard.
  - **L466 CN**: 结束当前预处理条件块或头文件保护。
- **L467 EN**: Blank line separating nearby declarations or logic.
  - **L467 CN**: 空行，用于分隔相邻声明或逻辑。
- **L468 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L468 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L469 EN**: Blank line separating nearby declarations or logic.
  - **L469 CN**: 空行，用于分隔相邻声明或逻辑。
- **L470 EN**: Closes the current preprocessor conditional block or header guard.
  - **L470 CN**: 结束当前预处理条件块或头文件保护。

## Key Concepts / 关键概念

- **Statistical estimation / 统计估计**:
  - **EN**: Computes descriptive statistics or hypothesis-oriented quantities from iterators or containers.
  - **CN**: 从迭代器或容器计算描述性统计量或面向假设检验的数值。
- **Template genericity / 模板泛型**:
  - **EN**: Uses C++ templates to adapt behavior across types, policies, or compile-time parameters.
  - **CN**: 使用 C++ 模板使行为适配不同类型、策略或编译期参数。
- **Policy customization / 策略定制**:
  - **EN**: Routes behavior through policy objects or compile-time policy choices.
  - **CN**: 通过策略对象或编译期策略选择来路由行为。
- **Type traits and metaprogramming / 类型 traits 与元编程**:
  - **EN**: Uses compile-time traits to select implementations and validate types.
  - **CN**: 使用编译期 traits 来选择实现并校验类型。
- **Assertions and contracts / 断言与契约**:
  - **EN**: Checks preconditions, postconditions, or test outcomes to keep behavior well defined.
  - **CN**: 检查前置条件、后置条件或测试结果，以保持行为有明确定义。
- **Iterator-based algorithms / 基于迭代器的算法**:
  - **EN**: Operates over iterator ranges and generic containers instead of concrete storage types.
  - **CN**: 面向迭代器区间和泛型容器工作，而不是绑定具体存储类型。
- **Compile-time evaluation / 编译期求值**:
  - **EN**: Marks values or functions so some work can be performed during compilation.
  - **CN**: 标记值或函数，使部分工作可在编译期间完成。
- **Boost integration / Boost 集成**:
  - **EN**: Fits the implementation into Boost naming, configuration, and portability conventions.
  - **CN**: 让实现融入 Boost 的命名、配置与可移植性约定。

## Dependencies / 依赖关系

- **Direct local/internal includes / 直接本地或内部包含**: `iterator`, `tuple`, `type_traits`, `stdexcept`, `vector`, `algorithm`, `cmath`, `cstddef`, `boost/math/tools/assert.hpp`, `boost/math/tools/config.hpp`, `execution`, `future` ... (+1 more)
- **Dependency categories / 依赖类别**: C or C++ standard library facilities / C 或 C++ 标准库设施 (11), Boost.Math numeric tool helpers / Boost.Math 数值工具辅助逻辑 (2)

- **EN**: `iterator` provides C or C++ standard library facilities.
  - **CN**: `iterator` 提供C 或 C++ 标准库设施。
- **EN**: `tuple` provides C or C++ standard library facilities.
  - **CN**: `tuple` 提供C 或 C++ 标准库设施。
- **EN**: `type_traits` provides C or C++ standard library facilities.
  - **CN**: `type_traits` 提供C 或 C++ 标准库设施。
- **EN**: `stdexcept` provides C or C++ standard library facilities.
  - **CN**: `stdexcept` 提供C 或 C++ 标准库设施。
- **EN**: `vector` provides C or C++ standard library facilities.
  - **CN**: `vector` 提供C 或 C++ 标准库设施。
- **EN**: `algorithm` provides C or C++ standard library facilities.
  - **CN**: `algorithm` 提供C 或 C++ 标准库设施。
- **EN**: `cmath` provides C or C++ standard library facilities.
  - **CN**: `cmath` 提供C 或 C++ 标准库设施。
- **EN**: `cstddef` provides C or C++ standard library facilities.
  - **CN**: `cstddef` 提供C 或 C++ 标准库设施。
- **EN**: `boost/math/tools/assert.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/assert.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/config.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/config.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `execution` provides C or C++ standard library facilities.
  - **CN**: `execution` 提供C 或 C++ 标准库设施。
- **EN**: `future` provides C or C++ standard library facilities.
  - **CN**: `future` 提供C 或 C++ 标准库设施。
- **EN**: `thread` provides C or C++ standard library facilities.
  - **CN**: `thread` 提供C 或 C++ 标准库设施。
