# color_maps.hpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `third-party/boost-math/include/boost/math/tools/color_maps.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/` (`llvm-project`)
- **Purpose / 目的**:
  - **EN**: Provides Boost.Math utility templates, numeric traits, policies, and low-level support helpers.
  - **CN**: 提供 Boost.Math 的工具模板、数值 traits、策略与底层支撑辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

````cpp
   1: //  (C) Copyright Nick Thompson 2021.
   2: //  (C) Copyright Matt Borland 2022.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_MATH_COLOR_MAPS_HPP
   8: #define BOOST_MATH_COLOR_MAPS_HPP
   9: #include <algorithm> // for std::clamp
  10: #include <array>     // for table data
  11: #include <cmath>     // for std::floor
  12: #include <cstdint>   // fixed width integer types
  13: #include <boost/math/special_functions/fpclassify.hpp>
  14: 
  15: #if __has_include("lodepng.h")
  16: 
  17: #include "lodepng.h"
  18: #include <iostream>
  19: #include <string>
  20: #include <vector>
  21: 
  22: namespace boost::math::tools {
  23: 
  24: // In lodepng, the vector is expected to be row major, with the top row
  25: // specified first. Note that this is a bit confusing sometimes as it's more
  26: // natural to let y increase moving *up*.
  27: unsigned write_png(const std::string &filename,
  28:                    const std::vector<std::uint8_t> &img, std::size_t width,
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
- **L7 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_COLOR_MAPS_HPP`.
  - **L7 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_COLOR_MAPS_HPP`。
- **L8 EN**: Defines macro `BOOST_MATH_COLOR_MAPS_HPP` for compile-time control, shorthand, or generated boilerplate.
  - **L8 CN**: 定义宏 `BOOST_MATH_COLOR_MAPS_HPP`，用于编译期控制、简写或生成样板代码。
- **L9 EN**: Includes <algorithm> to access C or C++ standard library facilities.
  - **L9 CN**: 引入 <algorithm> 以使用C 或 C++ 标准库设施。
- **L10 EN**: Includes <array> to access C or C++ standard library facilities.
  - **L10 CN**: 引入 <array> 以使用C 或 C++ 标准库设施。
- **L11 EN**: Includes <cmath> to access C or C++ standard library facilities.
  - **L11 CN**: 引入 <cmath> 以使用C 或 C++ 标准库设施。
- **L12 EN**: Includes <cstdint> to access C or C++ standard library facilities.
  - **L12 CN**: 引入 <cstdint> 以使用C 或 C++ 标准库设施。
- **L13 EN**: Includes <boost/math/special_functions/fpclassify.hpp> to access Boost.Math special-function declarations.
  - **L13 CN**: 引入 <boost/math/special_functions/fpclassify.hpp> 以使用Boost.Math 特殊函数声明。
- **L14 EN**: Blank line separating nearby declarations or logic.
  - **L14 CN**: 空行，用于分隔相邻声明或逻辑。
- **L15 EN**: Starts a preprocessor conditional block: `#if __has_include("lodepng.h")`.
  - **L15 CN**: 开始一个预处理条件块：`#if __has_include("lodepng.h")`。
- **L16 EN**: Blank line separating nearby declarations or logic.
  - **L16 CN**: 空行，用于分隔相邻声明或逻辑。
- **L17 EN**: Includes "lodepng.h" to access nearby local declarations.
  - **L17 CN**: 引入 "lodepng.h" 以使用附近的本地声明。
- **L18 EN**: Includes <iostream> to access C or C++ standard library facilities.
  - **L18 CN**: 引入 <iostream> 以使用C 或 C++ 标准库设施。
- **L19 EN**: Includes <string> to access C or C++ standard library facilities.
  - **L19 CN**: 引入 <string> 以使用C 或 C++ 标准库设施。
- **L20 EN**: Includes <vector> to access C or C++ standard library facilities.
  - **L20 CN**: 引入 <vector> 以使用C 或 C++ 标准库设施。
- **L21 EN**: Blank line separating nearby declarations or logic.
  - **L21 CN**: 空行，用于分隔相邻声明或逻辑。
- **L22 EN**: Opens namespace scope `boost::math::tools`.
  - **L22 CN**: 打开命名空间作用域 `boost::math::tools`。
- **L23 EN**: Blank line separating nearby declarations or logic.
  - **L23 CN**: 空行，用于分隔相邻声明或逻辑。
- **L24 EN**: Comment documents nearby intent or usage notes: `In lodepng, the vector is expected to be row major, with the top row`.
  - **L24 CN**: 注释说明附近代码的意图或使用说明：`In lodepng, the vector is expected to be row major, with the top row`。
- **L25 EN**: Comment documents nearby intent or usage notes: `specified first. Note that this is a bit confusing sometimes as it's more`.
  - **L25 CN**: 注释说明附近代码的意图或使用说明：`specified first. Note that this is a bit confusing sometimes as it's more`。
- **L26 EN**: Comment documents nearby intent or usage notes: `natural to let y increase moving *up*.`.
  - **L26 CN**: 注释说明附近代码的意图或使用说明：`natural to let y increase moving *up*.`。
- **L27 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned write_png(const std::string &filename,`.
  - **L27 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned write_png(const std::string &filename,`。
- **L28 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `const std::vector<std::uint8_t> &img, std::size_t width,`.
  - **L28 CN**: 继续一个多行参数列表、初始化器或聚合项：`const std::vector<std::uint8_t> &img, std::size_t width,`。

### Lines 29-56 / 第 29-56 行

````cpp
  29:                    std::size_t height) {
  30:   unsigned error = lodepng::encode(filename, img, width, height,
  31:                                    LodePNGColorType::LCT_RGBA, 8);
  32:   if (error) {
  33:     std::cerr << "Error encoding png: " << lodepng_error_text(error) << "\n";
  34:   }
  35:   return error;
  36: }
  37: 
  38: } // Namespace boost::math::tools
  39: #endif // __has_include("lodepng.h")
  40: 
  41: namespace boost::math::tools {
  42: 
  43: namespace detail {
  44: 
  45: // Data taken from: https://www.kennethmoreland.com/color-advice
  46: template <typename Real>
  47: static constexpr std::array<std::array<Real, 3>, 256> extended_kindlmann_data_ = {{
  48:     {0.0, 0.0, 0.0},
  49:     {0.01780246283347332, 0.0008750907117329381, 0.01626889466306607},
  50:     {0.03532931821571093, 0.001701802855992888, 0.03371323527689844},
  51:     {0.05144940541667612, 0.0024840651967150905, 0.05129428731410766},
  52:     {0.0645621163212177, 0.0032248645766847638, 0.06656239137344208},
  53:     {0.07579594587865912, 0.0038635326859125735, 0.08040589852559868},
  54:     {0.08560873187665013, 0.004465497712210337, 0.0932078727850277},
  55:     {0.0942303824351094, 0.005050242022804749, 0.10546097842833466},
  56:     {0.10126270743754426, 0.005684892741078179, 0.1185281778831713},
````
- **L29 EN**: Continues the surrounding expression or declaration: `std::size_t height) {`.
  - **L29 CN**: 继续构造周围的表达式或声明：`std::size_t height) {`。
- **L30 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned error = lodepng::encode(filename, img, width, height,`.
  - **L30 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned error = lodepng::encode(filename, img, width, height,`。
- **L31 EN**: Executes a standalone statement or declaration: `LodePNGColorType::LCT_RGBA, 8);`.
  - **L31 CN**: 执行一条独立语句或声明：`LodePNGColorType::LCT_RGBA, 8);`。
- **L32 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L32 CN**: 开始 `if` 控制流语句并计算其条件。
- **L33 EN**: Executes a call or declaration centered on `lodepng_error_text`.
  - **L33 CN**: 执行以 `lodepng_error_text` 为核心的调用或声明。
- **L34 EN**: Closes the current lexical scope or compound statement.
  - **L34 CN**: 结束当前词法作用域或复合语句块。
- **L35 EN**: Returns from the current function with `error`.
  - **L35 CN**: 以 `error` 从当前函数返回。
- **L36 EN**: Closes the current lexical scope or compound statement.
  - **L36 CN**: 结束当前词法作用域或复合语句块。
- **L37 EN**: Blank line separating nearby declarations or logic.
  - **L37 CN**: 空行，用于分隔相邻声明或逻辑。
- **L38 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L38 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L39 EN**: Closes the current preprocessor conditional block or header guard.
  - **L39 CN**: 结束当前预处理条件块或头文件保护。
- **L40 EN**: Blank line separating nearby declarations or logic.
  - **L40 CN**: 空行，用于分隔相邻声明或逻辑。
- **L41 EN**: Opens namespace scope `boost::math::tools`.
  - **L41 CN**: 打开命名空间作用域 `boost::math::tools`。
- **L42 EN**: Blank line separating nearby declarations or logic.
  - **L42 CN**: 空行，用于分隔相邻声明或逻辑。
- **L43 EN**: Opens namespace scope `detail`.
  - **L43 CN**: 打开命名空间作用域 `detail`。
- **L44 EN**: Blank line separating nearby declarations or logic.
  - **L44 CN**: 空行，用于分隔相邻声明或逻辑。
- **L45 EN**: Comment documents nearby intent or usage notes: `Data taken from: https://www.kennethmoreland.com/color-advice`.
  - **L45 CN**: 注释说明附近代码的意图或使用说明：`Data taken from: https://www.kennethmoreland.com/color-advice`。
- **L46 EN**: Introduces template parameters or specialization context: `template <typename Real>`.
  - **L46 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real>`。
- **L47 EN**: Continues the surrounding expression or declaration: `static constexpr std::array<std::array<Real, 3>, 256> extended_kindlmann_data_ = {{`.
  - **L47 CN**: 继续构造周围的表达式或声明：`static constexpr std::array<std::array<Real, 3>, 256> extended_kindlmann_data_ = {{`。
- **L48 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0, 0.0, 0.0},`.
  - **L48 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0, 0.0, 0.0},`。
- **L49 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01780246283347332, 0.0008750907117329381, 0.01626889466306607},`.
  - **L49 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01780246283347332, 0.0008750907117329381, 0.01626889466306607},`。
- **L50 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03532931821571093, 0.001701802855992888, 0.03371323527689844},`.
  - **L50 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03532931821571093, 0.001701802855992888, 0.03371323527689844},`。
- **L51 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.05144940541667612, 0.0024840651967150905, 0.05129428731410766},`.
  - **L51 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.05144940541667612, 0.0024840651967150905, 0.05129428731410766},`。
- **L52 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0645621163212177, 0.0032248645766847638, 0.06656239137344208},`.
  - **L52 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0645621163212177, 0.0032248645766847638, 0.06656239137344208},`。
- **L53 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07579594587865912, 0.0038635326859125735, 0.08040589852559868},`.
  - **L53 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07579594587865912, 0.0038635326859125735, 0.08040589852559868},`。
- **L54 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.08560873187665013, 0.004465497712210337, 0.0932078727850277},`.
  - **L54 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.08560873187665013, 0.004465497712210337, 0.0932078727850277},`。
- **L55 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0942303824351094, 0.005050242022804749, 0.10546097842833466},`.
  - **L55 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0942303824351094, 0.005050242022804749, 0.10546097842833466},`。
- **L56 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.10126270743754426, 0.005684892741078179, 0.1185281778831713},`.
  - **L56 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.10126270743754426, 0.005684892741078179, 0.1185281778831713},`。

### Lines 57-84 / 第 57-84 行

````cpp
  57:     {0.10670728559557496, 0.006372379780229372, 0.1327264977880264},
  58:     {0.11057848107867363, 0.00711002225409038, 0.1481009557210271},
  59:     {0.11345334766875653, 0.007801930722079915, 0.16375632363209905},
  60:     {0.11546967754116774, 0.008551159105416175, 0.17919646921236213},
  61:     {0.11670723282215431, 0.009290109551844812, 0.19453816576504604},
  62:     {0.11719317716660224, 0.010053390003759367, 0.2096941755954832},
  63:     {0.11698590922074067, 0.010738982214965748, 0.2248153138735493},
  64:     {0.11610841583017484, 0.011423659103765034, 0.2397105899482299},
  65:     {0.11462189563389044, 0.012116630983625478, 0.2543011097912221},
  66:     {0.11260886490150057, 0.012821489721377753, 0.26850817678211986},
  67:     {0.11014892407081325, 0.013468369062412165, 0.28238564749171374},
  68:     {0.1073826713115642, 0.014119392392571815, 0.29573128232180523},
  69:     {0.10439994774781663, 0.014757514745798027, 0.3086071932061514},
  70:     {0.10102122555390916, 0.015343280218838306, 0.3215039849422961},
  71:     {0.0973378808050489, 0.015957600087593806, 0.33424777423118057},
  72:     {0.09343083659118528, 0.016584962806154634, 0.34680083232754255},
  73:     {0.08930170392567455, 0.01713482653380763, 0.35927335578767955},
  74:     {0.08525050060408929, 0.0177409908749586, 0.3713432638745381},
  75:     {0.08133294699659654, 0.018315663017042745, 0.3831259644824107},
  76:     {0.07770620978910117, 0.018846472161798116, 0.3945981112365364},
  77:     {0.07453843004300356, 0.019323310547852572, 0.4057423733826484},
  78:     {0.07234014248177378, 0.019905434286220342, 0.4162427930814269},
  79:     {0.07070739337749535, 0.020358202603665252, 0.4265710517267966},
  80:     {0.06795561424010746, 0.020880774373512326, 0.4372263626095861},
  81:     {0.05495344243231267, 0.02153229240969933, 0.4511422122889977},
  82:     {0.03272243214743511, 0.022251621841318136, 0.4663539858995716},
  83:     {0.022542079629107543, 0.030087469482649072, 0.47217057965768244},
  84:     {0.022458068304623265, 0.043510017517847034, 0.47110682779300844},
````
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.10670728559557496, 0.006372379780229372, 0.1327264977880264},`.
  - **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.10670728559557496, 0.006372379780229372, 0.1327264977880264},`。
- **L58 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11057848107867363, 0.00711002225409038, 0.1481009557210271},`.
  - **L58 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11057848107867363, 0.00711002225409038, 0.1481009557210271},`。
- **L59 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11345334766875653, 0.007801930722079915, 0.16375632363209905},`.
  - **L59 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11345334766875653, 0.007801930722079915, 0.16375632363209905},`。
- **L60 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11546967754116774, 0.008551159105416175, 0.17919646921236213},`.
  - **L60 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11546967754116774, 0.008551159105416175, 0.17919646921236213},`。
- **L61 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11670723282215431, 0.009290109551844812, 0.19453816576504604},`.
  - **L61 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11670723282215431, 0.009290109551844812, 0.19453816576504604},`。
- **L62 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11719317716660224, 0.010053390003759367, 0.2096941755954832},`.
  - **L62 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11719317716660224, 0.010053390003759367, 0.2096941755954832},`。
- **L63 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11698590922074067, 0.010738982214965748, 0.2248153138735493},`.
  - **L63 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11698590922074067, 0.010738982214965748, 0.2248153138735493},`。
- **L64 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11610841583017484, 0.011423659103765034, 0.2397105899482299},`.
  - **L64 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11610841583017484, 0.011423659103765034, 0.2397105899482299},`。
- **L65 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11462189563389044, 0.012116630983625478, 0.2543011097912221},`.
  - **L65 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11462189563389044, 0.012116630983625478, 0.2543011097912221},`。
- **L66 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11260886490150057, 0.012821489721377753, 0.26850817678211986},`.
  - **L66 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11260886490150057, 0.012821489721377753, 0.26850817678211986},`。
- **L67 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11014892407081325, 0.013468369062412165, 0.28238564749171374},`.
  - **L67 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11014892407081325, 0.013468369062412165, 0.28238564749171374},`。
- **L68 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1073826713115642, 0.014119392392571815, 0.29573128232180523},`.
  - **L68 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1073826713115642, 0.014119392392571815, 0.29573128232180523},`。
- **L69 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.10439994774781663, 0.014757514745798027, 0.3086071932061514},`.
  - **L69 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.10439994774781663, 0.014757514745798027, 0.3086071932061514},`。
- **L70 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.10102122555390916, 0.015343280218838306, 0.3215039849422961},`.
  - **L70 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.10102122555390916, 0.015343280218838306, 0.3215039849422961},`。
- **L71 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0973378808050489, 0.015957600087593806, 0.33424777423118057},`.
  - **L71 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0973378808050489, 0.015957600087593806, 0.33424777423118057},`。
- **L72 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.09343083659118528, 0.016584962806154634, 0.34680083232754255},`.
  - **L72 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.09343083659118528, 0.016584962806154634, 0.34680083232754255},`。
- **L73 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.08930170392567455, 0.01713482653380763, 0.35927335578767955},`.
  - **L73 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.08930170392567455, 0.01713482653380763, 0.35927335578767955},`。
- **L74 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.08525050060408929, 0.0177409908749586, 0.3713432638745381},`.
  - **L74 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.08525050060408929, 0.0177409908749586, 0.3713432638745381},`。
- **L75 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.08133294699659654, 0.018315663017042745, 0.3831259644824107},`.
  - **L75 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.08133294699659654, 0.018315663017042745, 0.3831259644824107},`。
- **L76 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07770620978910117, 0.018846472161798116, 0.3945981112365364},`.
  - **L76 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07770620978910117, 0.018846472161798116, 0.3945981112365364},`。
- **L77 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07453843004300356, 0.019323310547852572, 0.4057423733826484},`.
  - **L77 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07453843004300356, 0.019323310547852572, 0.4057423733826484},`。
- **L78 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07234014248177378, 0.019905434286220342, 0.4162427930814269},`.
  - **L78 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07234014248177378, 0.019905434286220342, 0.4162427930814269},`。
- **L79 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07070739337749535, 0.020358202603665252, 0.4265710517267966},`.
  - **L79 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07070739337749535, 0.020358202603665252, 0.4265710517267966},`。
- **L80 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.06795561424010746, 0.020880774373512326, 0.4372263626095861},`.
  - **L80 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.06795561424010746, 0.020880774373512326, 0.4372263626095861},`。
- **L81 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.05495344243231267, 0.02153229240969933, 0.4511422122889977},`.
  - **L81 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.05495344243231267, 0.02153229240969933, 0.4511422122889977},`。
- **L82 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03272243214743511, 0.022251621841318136, 0.4663539858995716},`.
  - **L82 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03272243214743511, 0.022251621841318136, 0.4663539858995716},`。
- **L83 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.022542079629107543, 0.030087469482649072, 0.47217057965768244},`.
  - **L83 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.022542079629107543, 0.030087469482649072, 0.47217057965768244},`。
- **L84 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.022458068304623265, 0.043510017517847034, 0.47110682779300844},`.
  - **L84 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.022458068304623265, 0.043510017517847034, 0.47110682779300844},`。

### Lines 85-112 / 第 85-112 行

````cpp
  85:     {0.022335095562005403, 0.057612064160193856, 0.4678570071198978},
  86:     {0.022116548807623634, 0.07134877081764332, 0.46268236332653717},
  87:     {0.021827538746008197, 0.08452599623053175, 0.4558745921468776},
  88:     {0.02152986717396973, 0.09703540427752388, 0.44774032508967526},
  89:     {0.02094627101890751, 0.1088025513135693, 0.4387215006543877},
  90:     {0.02045606049512309, 0.11985098869691474, 0.4289709820913745},
  91:     {0.02004833311662032, 0.13019611448976093, 0.418764250948481},
  92:     {0.019665629961975118, 0.13987346436551024, 0.4083506364708954},
  93:     {0.019207577360338738, 0.14893167904386712, 0.39794947927856517},
  94:     {0.018537140772831598, 0.15742727919356447, 0.38774823408835746},
  95:     {0.01809424185998195, 0.16542627229987475, 0.3776911843959934},
  96:     {0.01766141281376398, 0.1729786761988682, 0.3679328135626787},
  97:     {0.01730286273144011, 0.18013597770041306, 0.3585022095429489},
  98:     {0.016765513848818806, 0.18694679199155506, 0.3495097966827364},
  99:     {0.016370126207011747, 0.19345202807035153, 0.3408690453055597},
 100:     {0.016125995919003662, 0.19968741140572166, 0.3325877811713702},
 101:     {0.015743835848192653, 0.20569130878531383, 0.32474294907403245},
 102:     {0.01521780170762751, 0.21149505980303446, 0.31731795633577514},
 103:     {0.014826523924383142, 0.21711739930665275, 0.31022963838193685},
 104:     {0.01455583202575653, 0.22257980137551625, 0.3034714591858722},
 105:     {0.014390286122102107, 0.2279012896050861, 0.29703484457819845},
 106:     {0.014011918932944173, 0.23311039219433838, 0.29095765923101685},
 107:     {0.013696545884500274, 0.23821209907670415, 0.28517017024142427},
 108:     {0.013424574431729107, 0.24321964058367318, 0.2796599788292758},
 109:     {0.013175868771764966, 0.2481448283322712, 0.27441390646690217},
 110:     {0.01292963716411873, 0.2529982361555283, 0.2694181731856298},
 111:     {0.012664284210951764, 0.25778935856659907, 0.2646585394556099},
 112:     {0.012715975678052705, 0.26251129306552407, 0.26008624607226105},
````
- **L85 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.022335095562005403, 0.057612064160193856, 0.4678570071198978},`.
  - **L85 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.022335095562005403, 0.057612064160193856, 0.4678570071198978},`。
- **L86 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.022116548807623634, 0.07134877081764332, 0.46268236332653717},`.
  - **L86 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.022116548807623634, 0.07134877081764332, 0.46268236332653717},`。
- **L87 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.021827538746008197, 0.08452599623053175, 0.4558745921468776},`.
  - **L87 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.021827538746008197, 0.08452599623053175, 0.4558745921468776},`。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02152986717396973, 0.09703540427752388, 0.44774032508967526},`.
  - **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02152986717396973, 0.09703540427752388, 0.44774032508967526},`。
- **L89 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02094627101890751, 0.1088025513135693, 0.4387215006543877},`.
  - **L89 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02094627101890751, 0.1088025513135693, 0.4387215006543877},`。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02045606049512309, 0.11985098869691474, 0.4289709820913745},`.
  - **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02045606049512309, 0.11985098869691474, 0.4289709820913745},`。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02004833311662032, 0.13019611448976093, 0.418764250948481},`.
  - **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02004833311662032, 0.13019611448976093, 0.418764250948481},`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.019665629961975118, 0.13987346436551024, 0.4083506364708954},`.
  - **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.019665629961975118, 0.13987346436551024, 0.4083506364708954},`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.019207577360338738, 0.14893167904386712, 0.39794947927856517},`.
  - **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.019207577360338738, 0.14893167904386712, 0.39794947927856517},`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.018537140772831598, 0.15742727919356447, 0.38774823408835746},`.
  - **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.018537140772831598, 0.15742727919356447, 0.38774823408835746},`。
- **L95 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01809424185998195, 0.16542627229987475, 0.3776911843959934},`.
  - **L95 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01809424185998195, 0.16542627229987475, 0.3776911843959934},`。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01766141281376398, 0.1729786761988682, 0.3679328135626787},`.
  - **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01766141281376398, 0.1729786761988682, 0.3679328135626787},`。
- **L97 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01730286273144011, 0.18013597770041306, 0.3585022095429489},`.
  - **L97 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01730286273144011, 0.18013597770041306, 0.3585022095429489},`。
- **L98 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.016765513848818806, 0.18694679199155506, 0.3495097966827364},`.
  - **L98 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.016765513848818806, 0.18694679199155506, 0.3495097966827364},`。
- **L99 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.016370126207011747, 0.19345202807035153, 0.3408690453055597},`.
  - **L99 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.016370126207011747, 0.19345202807035153, 0.3408690453055597},`。
- **L100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.016125995919003662, 0.19968741140572166, 0.3325877811713702},`.
  - **L100 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.016125995919003662, 0.19968741140572166, 0.3325877811713702},`。
- **L101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.015743835848192653, 0.20569130878531383, 0.32474294907403245},`.
  - **L101 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.015743835848192653, 0.20569130878531383, 0.32474294907403245},`。
- **L102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01521780170762751, 0.21149505980303446, 0.31731795633577514},`.
  - **L102 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01521780170762751, 0.21149505980303446, 0.31731795633577514},`。
- **L103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.014826523924383142, 0.21711739930665275, 0.31022963838193685},`.
  - **L103 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.014826523924383142, 0.21711739930665275, 0.31022963838193685},`。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01455583202575653, 0.22257980137551625, 0.3034714591858722},`.
  - **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01455583202575653, 0.22257980137551625, 0.3034714591858722},`。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.014390286122102107, 0.2279012896050861, 0.29703484457819845},`.
  - **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.014390286122102107, 0.2279012896050861, 0.29703484457819845},`。
- **L106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.014011918932944173, 0.23311039219433838, 0.29095765923101685},`.
  - **L106 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.014011918932944173, 0.23311039219433838, 0.29095765923101685},`。
- **L107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.013696545884500274, 0.23821209907670415, 0.28517017024142427},`.
  - **L107 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.013696545884500274, 0.23821209907670415, 0.28517017024142427},`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.013424574431729107, 0.24321964058367318, 0.2796599788292758},`.
  - **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.013424574431729107, 0.24321964058367318, 0.2796599788292758},`。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.013175868771764966, 0.2481448283322712, 0.27441390646690217},`.
  - **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.013175868771764966, 0.2481448283322712, 0.27441390646690217},`。
- **L110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01292963716411873, 0.2529982361555283, 0.2694181731856298},`.
  - **L110 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01292963716411873, 0.2529982361555283, 0.2694181731856298},`。
- **L111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.012664284210951764, 0.25778935856659907, 0.2646585394556099},`.
  - **L111 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.012664284210951764, 0.25778935856659907, 0.2646585394556099},`。
- **L112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.012715975678052705, 0.26251129306552407, 0.26008624607226105},`.
  - **L112 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.012715975678052705, 0.26251129306552407, 0.26008624607226105},`。

### Lines 113-140 / 第 113-140 行

````cpp
 113:     {0.012935277312292002, 0.26721180456135596, 0.25537236409530106},
 114:     {0.013151828277929324, 0.2719132248379506, 0.25038669592493334},
 115:     {0.013365797957379829, 0.2766141217472141, 0.24512715790174772},
 116:     {0.013577263863962, 0.28131308067549815, 0.23959178395846548},
 117:     {0.01378615567948453, 0.2860087090215386, 0.2337787381698218},
 118:     {0.013992194922389659, 0.290699640854686, 0.22768632911662062},
 119:     {0.014194831158161888, 0.2953845417355913, 0.22131302646406403},
 120:     {0.014393175927633691, 0.3000621136791825, 0.21465748025951026},
 121:     {0.014585935863149577, 0.3047311002377708, 0.20771854359069752},
 122:     {0.014771346788489043, 0.3093902916806014, 0.2004952994228159},
 123:     {0.015316330980203078, 0.314019615719643, 0.19303372895005055},
 124:     {0.015485867479267465, 0.318655599270661, 0.18524675366655605},
 125:     {0.015639979963365892, 0.32327849834847494, 0.17717467982889423},
 126:     {0.015774461623191554, 0.32788734271858383, 0.16881795823876974},
 127:     {0.01588440162820881, 0.33248123546361147, 0.16017753232003507},
 128:     {0.016371319978387793, 0.3370394958269932, 0.15133715480703355},
 129:     {0.016424021691114017, 0.34160094506458616, 0.14214299485979426},
 130:     {0.01686035446719742, 0.3461250509254044, 0.13277234235578353},
 131:     {0.016830214158745957, 0.35065150775079285, 0.12304027310946478},
 132:     {0.017189890203442686, 0.3551393166240582, 0.11316410079747846},
 133:     {0.0175057105195252, 0.35960835629733406, 0.10305638071018988},
 134:     {0.017770106546070377, 0.36405841970084374, 0.09273371145052256},
 135:     {0.01797534767005524, 0.3684894220401705, 0.08221895294189338},
 136:     {0.018113811240203243, 0.37290140979305425, 0.07154410587314339},
 137:     {0.01817830121675857, 0.37729457180004805, 0.060754926838460115},
 138:     {0.018678925823257978, 0.3816479795981337, 0.05016037872247533},
 139:     {0.01858851921615119, 0.38600461104105177, 0.039408863532102954},
 140:     {0.018946886533322625, 0.39032256740422405, 0.0299653011886783},
````
- **L113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.012935277312292002, 0.26721180456135596, 0.25537236409530106},`.
  - **L113 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.012935277312292002, 0.26721180456135596, 0.25537236409530106},`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.013151828277929324, 0.2719132248379506, 0.25038669592493334},`.
  - **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.013151828277929324, 0.2719132248379506, 0.25038669592493334},`。
- **L115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.013365797957379829, 0.2766141217472141, 0.24512715790174772},`.
  - **L115 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.013365797957379829, 0.2766141217472141, 0.24512715790174772},`。
- **L116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.013577263863962, 0.28131308067549815, 0.23959178395846548},`.
  - **L116 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.013577263863962, 0.28131308067549815, 0.23959178395846548},`。
- **L117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01378615567948453, 0.2860087090215386, 0.2337787381698218},`.
  - **L117 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01378615567948453, 0.2860087090215386, 0.2337787381698218},`。
- **L118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.013992194922389659, 0.290699640854686, 0.22768632911662062},`.
  - **L118 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.013992194922389659, 0.290699640854686, 0.22768632911662062},`。
- **L119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.014194831158161888, 0.2953845417355913, 0.22131302646406403},`.
  - **L119 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.014194831158161888, 0.2953845417355913, 0.22131302646406403},`。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.014393175927633691, 0.3000621136791825, 0.21465748025951026},`.
  - **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.014393175927633691, 0.3000621136791825, 0.21465748025951026},`。
- **L121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.014585935863149577, 0.3047311002377708, 0.20771854359069752},`.
  - **L121 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.014585935863149577, 0.3047311002377708, 0.20771854359069752},`。
- **L122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.014771346788489043, 0.3093902916806014, 0.2004952994228159},`.
  - **L122 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.014771346788489043, 0.3093902916806014, 0.2004952994228159},`。
- **L123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.015316330980203078, 0.314019615719643, 0.19303372895005055},`.
  - **L123 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.015316330980203078, 0.314019615719643, 0.19303372895005055},`。
- **L124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.015485867479267465, 0.318655599270661, 0.18524675366655605},`.
  - **L124 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.015485867479267465, 0.318655599270661, 0.18524675366655605},`。
- **L125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.015639979963365892, 0.32327849834847494, 0.17717467982889423},`.
  - **L125 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.015639979963365892, 0.32327849834847494, 0.17717467982889423},`。
- **L126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.015774461623191554, 0.32788734271858383, 0.16881795823876974},`.
  - **L126 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.015774461623191554, 0.32788734271858383, 0.16881795823876974},`。
- **L127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01588440162820881, 0.33248123546361147, 0.16017753232003507},`.
  - **L127 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01588440162820881, 0.33248123546361147, 0.16017753232003507},`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.016371319978387793, 0.3370394958269932, 0.15133715480703355},`.
  - **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.016371319978387793, 0.3370394958269932, 0.15133715480703355},`。
- **L129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.016424021691114017, 0.34160094506458616, 0.14214299485979426},`.
  - **L129 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.016424021691114017, 0.34160094506458616, 0.14214299485979426},`。
- **L130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01686035446719742, 0.3461250509254044, 0.13277234235578353},`.
  - **L130 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01686035446719742, 0.3461250509254044, 0.13277234235578353},`。
- **L131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.016830214158745957, 0.35065150775079285, 0.12304027310946478},`.
  - **L131 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.016830214158745957, 0.35065150775079285, 0.12304027310946478},`。
- **L132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.017189890203442686, 0.3551393166240582, 0.11316410079747846},`.
  - **L132 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.017189890203442686, 0.3551393166240582, 0.11316410079747846},`。
- **L133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0175057105195252, 0.35960835629733406, 0.10305638071018988},`.
  - **L133 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0175057105195252, 0.35960835629733406, 0.10305638071018988},`。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.017770106546070377, 0.36405841970084374, 0.09273371145052256},`.
  - **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.017770106546070377, 0.36405841970084374, 0.09273371145052256},`。
- **L135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01797534767005524, 0.3684894220401705, 0.08221895294189338},`.
  - **L135 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01797534767005524, 0.3684894220401705, 0.08221895294189338},`。
- **L136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.018113811240203243, 0.37290140979305425, 0.07154410587314339},`.
  - **L136 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.018113811240203243, 0.37290140979305425, 0.07154410587314339},`。
- **L137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01817830121675857, 0.37729457180004805, 0.060754926838460115},`.
  - **L137 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01817830121675857, 0.37729457180004805, 0.060754926838460115},`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.018678925823257978, 0.3816479795981337, 0.05016037872247533},`.
  - **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.018678925823257978, 0.3816479795981337, 0.05016037872247533},`。
- **L139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01858851921615119, 0.38600461104105177, 0.039408863532102954},`.
  - **L139 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01858851921615119, 0.38600461104105177, 0.039408863532102954},`。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.018946886533322625, 0.39032256740422405, 0.0299653011886783},`.
  - **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.018946886533322625, 0.39032256740422405, 0.0299653011886783},`。

### Lines 141-168 / 第 141-168 行

````cpp
 141:     {0.019234421105765478, 0.3946241249528643, 0.022430553487672214},
 142:     {0.023922664243633134, 0.39873762922211814, 0.019079643738415406},
 143:     {0.032696329777136775, 0.40268821797442356, 0.01927104403025176},
 144:     {0.04029295620970803, 0.40668863206181144, 0.019627058713507277},
 145:     {0.0445051772107334, 0.4108066755028012, 0.019605221754402217},
 146:     {0.05023033461306361, 0.4148702424589703, 0.019837552743924382},
 147:     {0.05751407234865658, 0.4188712020471279, 0.01998429281648217},
 148:     {0.06662959400279314, 0.42277996280066904, 0.020350820236973332},
 149:     {0.07687976931053314, 0.42661141097115085, 0.02061535207978936},
 150:     {0.08808647489550425, 0.4303590557627002, 0.02076563040362206},
 151:     {0.10009395068503488, 0.4340168594615687, 0.020789592249385695},
 152:     {0.11305049861381905, 0.43755751986676594, 0.021011622748947127},
 153:     {0.1265170163106781, 0.440997674167719, 0.021092771828857385},
 154:     {0.14064355039494827, 0.44431106175406876, 0.021367147770030247},
 155:     {0.15509985051348987, 0.4475149894903065, 0.021487010699090946},
 156:     {0.17002834364548297, 0.4505843077306851, 0.02179659002621885},
 157:     {0.18517425344416824, 0.4535369003239919, 0.021939140218683366},
 158:     {0.2005125857975878, 0.4563695433411364, 0.02190396156431214},
 159:     {0.2161609253409036, 0.459058907077816, 0.022051019673724737},
 160:     {0.23193556411255867, 0.46162347760839634, 0.022009915914684375},
 161:     {0.24793825797637276, 0.4640413181325817, 0.022151516874270333},
 162:     {0.26412488725558203, 0.4663117397956657, 0.022482129570858295},
 163:     {0.2803606712938716, 0.4684536010278855, 0.02261757765237555},
 164:     {0.296637938074376, 0.47046590244093683, 0.0225494975844844},
 165:     {0.31302849974924285, 0.47232989838033596, 0.022671977207001336},
 166:     {0.3294300351704986, 0.4740642435062818, 0.0225864988960952},
 167:     {0.3459012409505257, 0.475652007794974, 0.022699138396783294},
 168:     {0.3624157976623936, 0.4770953196310352, 0.02302034933556348},
````
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.019234421105765478, 0.3946241249528643, 0.022430553487672214},`.
  - **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.019234421105765478, 0.3946241249528643, 0.022430553487672214},`。
- **L142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.023922664243633134, 0.39873762922211814, 0.019079643738415406},`.
  - **L142 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.023922664243633134, 0.39873762922211814, 0.019079643738415406},`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.032696329777136775, 0.40268821797442356, 0.01927104403025176},`.
  - **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.032696329777136775, 0.40268821797442356, 0.01927104403025176},`。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.04029295620970803, 0.40668863206181144, 0.019627058713507277},`.
  - **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.04029295620970803, 0.40668863206181144, 0.019627058713507277},`。
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0445051772107334, 0.4108066755028012, 0.019605221754402217},`.
  - **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0445051772107334, 0.4108066755028012, 0.019605221754402217},`。
- **L146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.05023033461306361, 0.4148702424589703, 0.019837552743924382},`.
  - **L146 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.05023033461306361, 0.4148702424589703, 0.019837552743924382},`。
- **L147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.05751407234865658, 0.4188712020471279, 0.01998429281648217},`.
  - **L147 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.05751407234865658, 0.4188712020471279, 0.01998429281648217},`。
- **L148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.06662959400279314, 0.42277996280066904, 0.020350820236973332},`.
  - **L148 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.06662959400279314, 0.42277996280066904, 0.020350820236973332},`。
- **L149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07687976931053314, 0.42661141097115085, 0.02061535207978936},`.
  - **L149 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07687976931053314, 0.42661141097115085, 0.02061535207978936},`。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.08808647489550425, 0.4303590557627002, 0.02076563040362206},`.
  - **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.08808647489550425, 0.4303590557627002, 0.02076563040362206},`。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.10009395068503488, 0.4340168594615687, 0.020789592249385695},`.
  - **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.10009395068503488, 0.4340168594615687, 0.020789592249385695},`。
- **L152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11305049861381905, 0.43755751986676594, 0.021011622748947127},`.
  - **L152 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11305049861381905, 0.43755751986676594, 0.021011622748947127},`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1265170163106781, 0.440997674167719, 0.021092771828857385},`.
  - **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1265170163106781, 0.440997674167719, 0.021092771828857385},`。
- **L154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14064355039494827, 0.44431106175406876, 0.021367147770030247},`.
  - **L154 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14064355039494827, 0.44431106175406876, 0.021367147770030247},`。
- **L155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15509985051348987, 0.4475149894903065, 0.021487010699090946},`.
  - **L155 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15509985051348987, 0.4475149894903065, 0.021487010699090946},`。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.17002834364548297, 0.4505843077306851, 0.02179659002621885},`.
  - **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.17002834364548297, 0.4505843077306851, 0.02179659002621885},`。
- **L157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.18517425344416824, 0.4535369003239919, 0.021939140218683366},`.
  - **L157 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.18517425344416824, 0.4535369003239919, 0.021939140218683366},`。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2005125857975878, 0.4563695433411364, 0.02190396156431214},`.
  - **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2005125857975878, 0.4563695433411364, 0.02190396156431214},`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2161609253409036, 0.459058907077816, 0.022051019673724737},`.
  - **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2161609253409036, 0.459058907077816, 0.022051019673724737},`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.23193556411255867, 0.46162347760839634, 0.022009915914684375},`.
  - **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.23193556411255867, 0.46162347760839634, 0.022009915914684375},`。
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.24793825797637276, 0.4640413181325817, 0.022151516874270333},`.
  - **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.24793825797637276, 0.4640413181325817, 0.022151516874270333},`。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.26412488725558203, 0.4663117397956657, 0.022482129570858295},`.
  - **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.26412488725558203, 0.4663117397956657, 0.022482129570858295},`。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2803606712938716, 0.4684536010278855, 0.02261757765237555},`.
  - **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2803606712938716, 0.4684536010278855, 0.02261757765237555},`。
- **L164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.296637938074376, 0.47046590244093683, 0.0225494975844844},`.
  - **L164 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.296637938074376, 0.47046590244093683, 0.0225494975844844},`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.31302849974924285, 0.47232989838033596, 0.022671977207001336},`.
  - **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.31302849974924285, 0.47232989838033596, 0.022671977207001336},`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3294300351704986, 0.4740642435062818, 0.0225864988960952},`.
  - **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3294300351704986, 0.4740642435062818, 0.0225864988960952},`。
- **L167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3459012409505257, 0.475652007794974, 0.022699138396783294},`.
  - **L167 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3459012409505257, 0.475652007794974, 0.022699138396783294},`。
- **L168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3624157976623936, 0.4770953196310352, 0.02302034933556348},`.
  - **L168 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3624157976623936, 0.4770953196310352, 0.02302034933556348},`。

### Lines 169-196 / 第 169-196 行

````cpp
 169:     {0.37889809766361104, 0.47841222515883747, 0.02313701355469799},
 170:     {0.39534430178227603, 0.4796035027123265, 0.023043880179838123},
 171:     {0.41179003849439816, 0.48065635396111855, 0.023172253504396497},
 172:     {0.42818080415815335, 0.4815874024816033, 0.02309359581707615},
 173:     {0.4445420271983608, 0.48238619597444804, 0.02325212759648816},
 174:     {0.46083197524945735, 0.48306807922963163, 0.023208795520823236},
 175:     {0.477066527169599, 0.4836251567552959, 0.023421052378504333},
 176:     {0.49330882182932195, 0.48404259768616237, 0.02351307077213582},
 177:     {0.5101341569176142, 0.4841269939874597, 0.02474700862397102},
 178:     {0.5276083798119485, 0.4838477594175364, 0.025493073746431755},
 179:     {0.5457675808274345, 0.4831581006215279, 0.026232895897136985},
 180:     {0.5646323060116551, 0.48201590123588106, 0.02698278973250526},
 181:     {0.5842072626155643, 0.4803756805891137, 0.02819024072200311},
 182:     {0.6045266103850581, 0.4781882736718959, 0.028981783530659853},
 183:     {0.6255919000983664, 0.47539734049324306, 0.02979697562508809},
 184:     {0.6473800379403847, 0.47195081918673226, 0.031044032153032777},
 185:     {0.6699068794902967, 0.4677815168730453, 0.03227150878455887},
 186:     {0.6932001837200186, 0.46280562738356607, 0.03304163325898518},
 187:     {0.7171335868744761, 0.4569883986979126, 0.034530544840017025},
 188:     {0.7417906038430161, 0.4502061037239105, 0.035463650965058914},
 189:     {0.7670484685327643, 0.4424084941387379, 0.03656485419201359},
 190:     {0.7927923527600884, 0.4335431093764269, 0.038110096816360045},
 191:     {0.8190538618349916, 0.4234676649184017, 0.03921665123224114},
 192:     {0.845635547462146, 0.4121565351852396, 0.040504503887090086},
 193:     {0.8724273941843113, 0.399532161591502, 0.04177752341186975},
 194:     {0.8993071734347423, 0.38551601321374424, 0.04288218868653492},
 195:     {0.9259938049276442, 0.3701581656966096, 0.0443028413484576},
 196:     {0.95240006413475, 0.3533630589382493, 0.045512688804357486},
````
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.37889809766361104, 0.47841222515883747, 0.02313701355469799},`.
  - **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.37889809766361104, 0.47841222515883747, 0.02313701355469799},`。
- **L170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.39534430178227603, 0.4796035027123265, 0.023043880179838123},`.
  - **L170 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.39534430178227603, 0.4796035027123265, 0.023043880179838123},`。
- **L171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.41179003849439816, 0.48065635396111855, 0.023172253504396497},`.
  - **L171 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.41179003849439816, 0.48065635396111855, 0.023172253504396497},`。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.42818080415815335, 0.4815874024816033, 0.02309359581707615},`.
  - **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.42818080415815335, 0.4815874024816033, 0.02309359581707615},`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4445420271983608, 0.48238619597444804, 0.02325212759648816},`.
  - **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4445420271983608, 0.48238619597444804, 0.02325212759648816},`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.46083197524945735, 0.48306807922963163, 0.023208795520823236},`.
  - **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.46083197524945735, 0.48306807922963163, 0.023208795520823236},`。
- **L175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.477066527169599, 0.4836251567552959, 0.023421052378504333},`.
  - **L175 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.477066527169599, 0.4836251567552959, 0.023421052378504333},`。
- **L176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.49330882182932195, 0.48404259768616237, 0.02351307077213582},`.
  - **L176 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.49330882182932195, 0.48404259768616237, 0.02351307077213582},`。
- **L177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5101341569176142, 0.4841269939874597, 0.02474700862397102},`.
  - **L177 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5101341569176142, 0.4841269939874597, 0.02474700862397102},`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5276083798119485, 0.4838477594175364, 0.025493073746431755},`.
  - **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5276083798119485, 0.4838477594175364, 0.025493073746431755},`。
- **L179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5457675808274345, 0.4831581006215279, 0.026232895897136985},`.
  - **L179 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5457675808274345, 0.4831581006215279, 0.026232895897136985},`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5646323060116551, 0.48201590123588106, 0.02698278973250526},`.
  - **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5646323060116551, 0.48201590123588106, 0.02698278973250526},`。
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5842072626155643, 0.4803756805891137, 0.02819024072200311},`.
  - **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5842072626155643, 0.4803756805891137, 0.02819024072200311},`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6045266103850581, 0.4781882736718959, 0.028981783530659853},`.
  - **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6045266103850581, 0.4781882736718959, 0.028981783530659853},`。
- **L183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6255919000983664, 0.47539734049324306, 0.02979697562508809},`.
  - **L183 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6255919000983664, 0.47539734049324306, 0.02979697562508809},`。
- **L184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6473800379403847, 0.47195081918673226, 0.031044032153032777},`.
  - **L184 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6473800379403847, 0.47195081918673226, 0.031044032153032777},`。
- **L185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6699068794902967, 0.4677815168730453, 0.03227150878455887},`.
  - **L185 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6699068794902967, 0.4677815168730453, 0.03227150878455887},`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6932001837200186, 0.46280562738356607, 0.03304163325898518},`.
  - **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6932001837200186, 0.46280562738356607, 0.03304163325898518},`。
- **L187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7171335868744761, 0.4569883986979126, 0.034530544840017025},`.
  - **L187 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7171335868744761, 0.4569883986979126, 0.034530544840017025},`。
- **L188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7417906038430161, 0.4502061037239105, 0.035463650965058914},`.
  - **L188 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7417906038430161, 0.4502061037239105, 0.035463650965058914},`。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7670484685327643, 0.4424084941387379, 0.03656485419201359},`.
  - **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7670484685327643, 0.4424084941387379, 0.03656485419201359},`。
- **L190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7927923527600884, 0.4335431093764269, 0.038110096816360045},`.
  - **L190 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7927923527600884, 0.4335431093764269, 0.038110096816360045},`。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8190538618349916, 0.4234676649184017, 0.03921665123224114},`.
  - **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8190538618349916, 0.4234676649184017, 0.03921665123224114},`。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.845635547462146, 0.4121565351852396, 0.040504503887090086},`.
  - **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.845635547462146, 0.4121565351852396, 0.040504503887090086},`。
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8724273941843113, 0.399532161591502, 0.04177752341186975},`.
  - **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8724273941843113, 0.399532161591502, 0.04177752341186975},`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8993071734347423, 0.38551601321374424, 0.04288218868653492},`.
  - **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8993071734347423, 0.38551601321374424, 0.04288218868653492},`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9259938049276442, 0.3701581656966096, 0.0443028413484576},`.
  - **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9259938049276442, 0.3701581656966096, 0.0443028413484576},`。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.95240006413475, 0.3533630589382493, 0.045512688804357486},`.
  - **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.95240006413475, 0.3533630589382493, 0.045512688804357486},`。

### Lines 197-224 / 第 197-224 行

````cpp
 197:     {0.9573302263433631, 0.356573181832402, 0.10485176035519561},
 198:     {0.9593762651267365, 0.3625998458005728, 0.14695621074226434},
 199:     {0.9609003469257634, 0.3691267599755898, 0.17919496865835138},
 200:     {0.9621556849513918, 0.37591169845314204, 0.20542079832986643},
 201:     {0.9631644672895096, 0.3829300179877212, 0.22750885439024698},
 202:     {0.964095792386828, 0.39002737893200634, 0.24632170224175154},
 203:     {0.9648793311938041, 0.3972605885272423, 0.26263686798960245},
 204:     {0.9655077342802952, 0.40462643797868664, 0.2769429329997333},
 205:     {0.9660587860054425, 0.41205105448230417, 0.2895251327064319},
 206:     {0.9665994127853909, 0.4194734509128834, 0.3006180918171856},
 207:     {0.9671516506858119, 0.4268421716026803, 0.31083965567414507},
 208:     {0.9677276657441855, 0.4338269194529401, 0.3245165284653611},
 209:     {0.968504522437997, 0.44042580568421147, 0.33966351784918897},
 210:     {0.9692871734210263, 0.4467701596827656, 0.3565758064426104},
 211:     {0.9702540772004374, 0.45269977328586647, 0.37534236971527246},
 212:     {0.9712179772934669, 0.45837869586824703, 0.39562556319119124},
 213:     {0.9722317464026985, 0.4637976195449472, 0.41693497643767774},
 214:     {0.9732263821032568, 0.46903038930488866, 0.438958844869214},
 215:     {0.9742462614418889, 0.4740678468975709, 0.46139896139324615},
 216:     {0.9750871661398574, 0.4790671203997809, 0.48416548522290065},
 217:     {0.9754147288760454, 0.4842669015553663, 0.5072296427052377},
 218:     {0.9756180439862371, 0.48941330245201103, 0.5302646150938287},
 219:     {0.975842338006588, 0.49442037450446213, 0.5531433026983604},
 220:     {0.9761081874822527, 0.4992854244201668, 0.5758237590340696},
 221:     {0.9763112164847584, 0.5040877038326032, 0.5983033566276463},
 222:     {0.9765888685497933, 0.5087442572698033, 0.6205369331248914},
 223:     {0.9767110237468087, 0.5134164774149236, 0.642535156682777},
 224:     {0.9769336559396905, 0.5179413879764756, 0.6642639445578681},
````
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9573302263433631, 0.356573181832402, 0.10485176035519561},`.
  - **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9573302263433631, 0.356573181832402, 0.10485176035519561},`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9593762651267365, 0.3625998458005728, 0.14695621074226434},`.
  - **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9593762651267365, 0.3625998458005728, 0.14695621074226434},`。
- **L199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9609003469257634, 0.3691267599755898, 0.17919496865835138},`.
  - **L199 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9609003469257634, 0.3691267599755898, 0.17919496865835138},`。
- **L200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9621556849513918, 0.37591169845314204, 0.20542079832986643},`.
  - **L200 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9621556849513918, 0.37591169845314204, 0.20542079832986643},`。
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9631644672895096, 0.3829300179877212, 0.22750885439024698},`.
  - **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9631644672895096, 0.3829300179877212, 0.22750885439024698},`。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.964095792386828, 0.39002737893200634, 0.24632170224175154},`.
  - **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.964095792386828, 0.39002737893200634, 0.24632170224175154},`。
- **L203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9648793311938041, 0.3972605885272423, 0.26263686798960245},`.
  - **L203 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9648793311938041, 0.3972605885272423, 0.26263686798960245},`。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9655077342802952, 0.40462643797868664, 0.2769429329997333},`.
  - **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9655077342802952, 0.40462643797868664, 0.2769429329997333},`。
- **L205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9660587860054425, 0.41205105448230417, 0.2895251327064319},`.
  - **L205 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9660587860054425, 0.41205105448230417, 0.2895251327064319},`。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9665994127853909, 0.4194734509128834, 0.3006180918171856},`.
  - **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9665994127853909, 0.4194734509128834, 0.3006180918171856},`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9671516506858119, 0.4268421716026803, 0.31083965567414507},`.
  - **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9671516506858119, 0.4268421716026803, 0.31083965567414507},`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9677276657441855, 0.4338269194529401, 0.3245165284653611},`.
  - **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9677276657441855, 0.4338269194529401, 0.3245165284653611},`。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.968504522437997, 0.44042580568421147, 0.33966351784918897},`.
  - **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.968504522437997, 0.44042580568421147, 0.33966351784918897},`。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9692871734210263, 0.4467701596827656, 0.3565758064426104},`.
  - **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9692871734210263, 0.4467701596827656, 0.3565758064426104},`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9702540772004374, 0.45269977328586647, 0.37534236971527246},`.
  - **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9702540772004374, 0.45269977328586647, 0.37534236971527246},`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9712179772934669, 0.45837869586824703, 0.39562556319119124},`.
  - **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9712179772934669, 0.45837869586824703, 0.39562556319119124},`。
- **L213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9722317464026985, 0.4637976195449472, 0.41693497643767774},`.
  - **L213 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9722317464026985, 0.4637976195449472, 0.41693497643767774},`。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9732263821032568, 0.46903038930488866, 0.438958844869214},`.
  - **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9732263821032568, 0.46903038930488866, 0.438958844869214},`。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9742462614418889, 0.4740678468975709, 0.46139896139324615},`.
  - **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9742462614418889, 0.4740678468975709, 0.46139896139324615},`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9750871661398574, 0.4790671203997809, 0.48416548522290065},`.
  - **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9750871661398574, 0.4790671203997809, 0.48416548522290065},`。
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9754147288760454, 0.4842669015553663, 0.5072296427052377},`.
  - **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9754147288760454, 0.4842669015553663, 0.5072296427052377},`。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9756180439862371, 0.48941330245201103, 0.5302646150938287},`.
  - **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9756180439862371, 0.48941330245201103, 0.5302646150938287},`。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.975842338006588, 0.49442037450446213, 0.5531433026983604},`.
  - **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.975842338006588, 0.49442037450446213, 0.5531433026983604},`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9761081874822527, 0.4992854244201668, 0.5758237590340696},`.
  - **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9761081874822527, 0.4992854244201668, 0.5758237590340696},`。
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9763112164847584, 0.5040877038326032, 0.5983033566276463},`.
  - **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9763112164847584, 0.5040877038326032, 0.5983033566276463},`。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9765888685497933, 0.5087442572698033, 0.6205369331248914},`.
  - **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9765888685497933, 0.5087442572698033, 0.6205369331248914},`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9767110237468087, 0.5134164774149236, 0.642535156682777},`.
  - **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9767110237468087, 0.5134164774149236, 0.642535156682777},`。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9769336559396905, 0.5179413879764756, 0.6642639445578681},`.
  - **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9769336559396905, 0.5179413879764756, 0.6642639445578681},`。

### Lines 225-252 / 第 225-252 行

````cpp
 225:     {0.97714593184672, 0.5224000733132641, 0.6857254497050739},
 226:     {0.9773578783172437, 0.5267928368807481, 0.7069126158797037},
 227:     {0.9775785758579868, 0.5311203562318053, 0.7278211218920855},
 228:     {0.9778162437300041, 0.5353836351919077, 0.7484486916246914},
 229:     {0.9779596006498089, 0.5396657970725794, 0.7687562310971295},
 230:     {0.9782535179019477, 0.5438049537609145, 0.7888137943375886},
 231:     {0.9784674751508416, 0.5479668227414403, 0.8085390020370261},
 232:     {0.9786094914349676, 0.5521542299695001, 0.8279128914232778},
 233:     {0.978802652274665, 0.5562873882925448, 0.8469838693949315},
 234:     {0.9790515109370453, 0.5603687136966451, 0.8657552952207226},
 235:     {0.9792467908639096, 0.5644843583828043, 0.884150643372716},
 236:     {0.979395255578016, 0.5686380796177348, 0.902153526529961},
 237:     {0.9796145283071062, 0.5727494207545081, 0.9198418221195682},
 238:     {0.9797975302508571, 0.5769060493757885, 0.9371197812002375},
 239:     {0.9799502605574898, 0.581112131921931, 0.9539723226660747},
 240:     {0.9802283260758732, 0.5851928436561017, 0.9708243730936762},
 241:     {0.9738084905670951, 0.5944815591203718, 0.9806690105429076},
 242:     {0.9601472511008894, 0.609371508764095, 0.9813253841505029},
 243:     {0.9478379001570983, 0.6230463820702153, 0.9820290537295789},
 244:     {0.9366516898009005, 0.6357832537301149, 0.9826041912890565},
 245:     {0.926560589527891, 0.647678912888478, 0.9831062604022219},
 246:     {0.917620083682068, 0.6587517109196314, 0.9837253021469462},
 247:     {0.9095356769991322, 0.6692646118840428, 0.9841128740419515},
 248:     {0.9024501226204644, 0.6791460846603548, 0.9845858176159193},
 249:     {0.8962465735427142, 0.6885075377321772, 0.9850533329169425},
 250:     {0.8908928004592865, 0.6973907065167453, 0.9855589168505193},
 251:     {0.8862829519968738, 0.7058867599470733, 0.9859932359170603},
 252:     {0.8823207949642251, 0.7140769556818014, 0.9862329988164894},
````
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.97714593184672, 0.5224000733132641, 0.6857254497050739},`.
  - **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.97714593184672, 0.5224000733132641, 0.6857254497050739},`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9773578783172437, 0.5267928368807481, 0.7069126158797037},`.
  - **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9773578783172437, 0.5267928368807481, 0.7069126158797037},`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9775785758579868, 0.5311203562318053, 0.7278211218920855},`.
  - **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9775785758579868, 0.5311203562318053, 0.7278211218920855},`。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9778162437300041, 0.5353836351919077, 0.7484486916246914},`.
  - **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9778162437300041, 0.5353836351919077, 0.7484486916246914},`。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9779596006498089, 0.5396657970725794, 0.7687562310971295},`.
  - **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9779596006498089, 0.5396657970725794, 0.7687562310971295},`。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9782535179019477, 0.5438049537609145, 0.7888137943375886},`.
  - **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9782535179019477, 0.5438049537609145, 0.7888137943375886},`。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9784674751508416, 0.5479668227414403, 0.8085390020370261},`.
  - **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9784674751508416, 0.5479668227414403, 0.8085390020370261},`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9786094914349676, 0.5521542299695001, 0.8279128914232778},`.
  - **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9786094914349676, 0.5521542299695001, 0.8279128914232778},`。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.978802652274665, 0.5562873882925448, 0.8469838693949315},`.
  - **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.978802652274665, 0.5562873882925448, 0.8469838693949315},`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9790515109370453, 0.5603687136966451, 0.8657552952207226},`.
  - **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9790515109370453, 0.5603687136966451, 0.8657552952207226},`。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9792467908639096, 0.5644843583828043, 0.884150643372716},`.
  - **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9792467908639096, 0.5644843583828043, 0.884150643372716},`。
- **L236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.979395255578016, 0.5686380796177348, 0.902153526529961},`.
  - **L236 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.979395255578016, 0.5686380796177348, 0.902153526529961},`。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9796145283071062, 0.5727494207545081, 0.9198418221195682},`.
  - **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9796145283071062, 0.5727494207545081, 0.9198418221195682},`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9797975302508571, 0.5769060493757885, 0.9371197812002375},`.
  - **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9797975302508571, 0.5769060493757885, 0.9371197812002375},`。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9799502605574898, 0.581112131921931, 0.9539723226660747},`.
  - **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9799502605574898, 0.581112131921931, 0.9539723226660747},`。
- **L240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9802283260758732, 0.5851928436561017, 0.9708243730936762},`.
  - **L240 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9802283260758732, 0.5851928436561017, 0.9708243730936762},`。
- **L241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9738084905670951, 0.5944815591203718, 0.9806690105429076},`.
  - **L241 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9738084905670951, 0.5944815591203718, 0.9806690105429076},`。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9601472511008894, 0.609371508764095, 0.9813253841505029},`.
  - **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9601472511008894, 0.609371508764095, 0.9813253841505029},`。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9478379001570983, 0.6230463820702153, 0.9820290537295789},`.
  - **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9478379001570983, 0.6230463820702153, 0.9820290537295789},`。
- **L244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9366516898009005, 0.6357832537301149, 0.9826041912890565},`.
  - **L244 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9366516898009005, 0.6357832537301149, 0.9826041912890565},`。
- **L245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.926560589527891, 0.647678912888478, 0.9831062604022219},`.
  - **L245 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.926560589527891, 0.647678912888478, 0.9831062604022219},`。
- **L246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.917620083682068, 0.6587517109196314, 0.9837253021469462},`.
  - **L246 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.917620083682068, 0.6587517109196314, 0.9837253021469462},`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9095356769991322, 0.6692646118840428, 0.9841128740419515},`.
  - **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9095356769991322, 0.6692646118840428, 0.9841128740419515},`。
- **L248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9024501226204644, 0.6791460846603548, 0.9845858176159193},`.
  - **L248 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9024501226204644, 0.6791460846603548, 0.9845858176159193},`。
- **L249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8962465735427142, 0.6885075377321772, 0.9850533329169425},`.
  - **L249 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8962465735427142, 0.6885075377321772, 0.9850533329169425},`。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8908928004592865, 0.6973907065167453, 0.9855589168505193},`.
  - **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8908928004592865, 0.6973907065167453, 0.9855589168505193},`。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8862829519968738, 0.7058867599470733, 0.9859932359170603},`.
  - **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8862829519968738, 0.7058867599470733, 0.9859932359170603},`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8823207949642251, 0.7140769556818014, 0.9862329988164894},`.
  - **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8823207949642251, 0.7140769556818014, 0.9862329988164894},`。

### Lines 253-280 / 第 253-280 行

````cpp
 253:     {0.8791184292035925, 0.7218801604700443, 0.9866187467737363},
 254:     {0.876573928783208, 0.729373858466773, 0.9870301106239344},
 255:     {0.8745946211951233, 0.7366283504729199, 0.9873335856528983},
 256:     {0.8732148850190337, 0.7436116593104639, 0.9877197983463958},
 257:     {0.8723449314581001, 0.7503888453166406, 0.9880489690947681},
 258:     {0.871958051266914, 0.7569736347405297, 0.988343697831869},
 259:     {0.8720269380214271, 0.7633787505197638, 0.9886267968388239},
 260:     {0.872524018161346, 0.7696159559904953, 0.9889212756286642},
 261:     {0.8734217880637981, 0.7756960853936921, 0.9892503149320232},
 262:     {0.8746431257299938, 0.7816733363052604, 0.989454685042798},
 263:     {0.8762135041761673, 0.7875116467112275, 0.9897367010924504},
 264:     {0.878107018520635, 0.7932193523390607, 0.9901203210173019},
 265:     {0.880203346150829, 0.798890286193674, 0.9902546300884126},
 266:     {0.8825765528541782, 0.8044436600674107, 0.9905332036399778},
 267:     {0.885156863735442, 0.8099282584935031, 0.9907909519806376},
 268:     {0.8879236719089479, 0.815348848661961, 0.9910495231926625},
 269:     {0.890857185433795, 0.8207097106633178, 0.9913312422092354},
 270:     {0.8939377339865693, 0.8260147864401665, 0.9916599021959583},
 271:     {0.8970530658331168, 0.8313161221171385, 0.9919275427432451},
 272:     {0.8995688302115749, 0.8368229403489976, 0.9920969557419316},
 273:     {0.9019890025620054, 0.8423510813517494, 0.9923144994581347},
 274:     {0.9041189751589628, 0.8479541784289291, 0.992640232246134},
 275:     {0.9058973525369924, 0.8536784698518803, 0.9928660652275899},
 276:     {0.9073992963543378, 0.8594712199063077, 0.9931944239168621},
 277:     {0.9086279741575543, 0.8653520270780298, 0.993459751831716},
 278:     {0.9096655096337553, 0.8712669417189164, 0.9938665873784843},
 279:     {0.9105035986931571, 0.8772607309343242, 0.9940839016190398},
 280:     {0.9112280528710646, 0.8832787634425558, 0.994319531984062},
````
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8791184292035925, 0.7218801604700443, 0.9866187467737363},`.
  - **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8791184292035925, 0.7218801604700443, 0.9866187467737363},`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.876573928783208, 0.729373858466773, 0.9870301106239344},`.
  - **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.876573928783208, 0.729373858466773, 0.9870301106239344},`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8745946211951233, 0.7366283504729199, 0.9873335856528983},`.
  - **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8745946211951233, 0.7366283504729199, 0.9873335856528983},`。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8732148850190337, 0.7436116593104639, 0.9877197983463958},`.
  - **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8732148850190337, 0.7436116593104639, 0.9877197983463958},`。
- **L257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8723449314581001, 0.7503888453166406, 0.9880489690947681},`.
  - **L257 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8723449314581001, 0.7503888453166406, 0.9880489690947681},`。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.871958051266914, 0.7569736347405297, 0.988343697831869},`.
  - **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.871958051266914, 0.7569736347405297, 0.988343697831869},`。
- **L259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8720269380214271, 0.7633787505197638, 0.9886267968388239},`.
  - **L259 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8720269380214271, 0.7633787505197638, 0.9886267968388239},`。
- **L260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.872524018161346, 0.7696159559904953, 0.9889212756286642},`.
  - **L260 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.872524018161346, 0.7696159559904953, 0.9889212756286642},`。
- **L261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8734217880637981, 0.7756960853936921, 0.9892503149320232},`.
  - **L261 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8734217880637981, 0.7756960853936921, 0.9892503149320232},`。
- **L262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8746431257299938, 0.7816733363052604, 0.989454685042798},`.
  - **L262 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8746431257299938, 0.7816733363052604, 0.989454685042798},`。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8762135041761673, 0.7875116467112275, 0.9897367010924504},`.
  - **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8762135041761673, 0.7875116467112275, 0.9897367010924504},`。
- **L264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.878107018520635, 0.7932193523390607, 0.9901203210173019},`.
  - **L264 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.878107018520635, 0.7932193523390607, 0.9901203210173019},`。
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.880203346150829, 0.798890286193674, 0.9902546300884126},`.
  - **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.880203346150829, 0.798890286193674, 0.9902546300884126},`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8825765528541782, 0.8044436600674107, 0.9905332036399778},`.
  - **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8825765528541782, 0.8044436600674107, 0.9905332036399778},`。
- **L267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.885156863735442, 0.8099282584935031, 0.9907909519806376},`.
  - **L267 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.885156863735442, 0.8099282584935031, 0.9907909519806376},`。
- **L268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8879236719089479, 0.815348848661961, 0.9910495231926625},`.
  - **L268 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8879236719089479, 0.815348848661961, 0.9910495231926625},`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.890857185433795, 0.8207097106633178, 0.9913312422092354},`.
  - **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.890857185433795, 0.8207097106633178, 0.9913312422092354},`。
- **L270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8939377339865693, 0.8260147864401665, 0.9916599021959583},`.
  - **L270 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8939377339865693, 0.8260147864401665, 0.9916599021959583},`。
- **L271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8970530658331168, 0.8313161221171385, 0.9919275427432451},`.
  - **L271 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8970530658331168, 0.8313161221171385, 0.9919275427432451},`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8995688302115749, 0.8368229403489976, 0.9920969557419316},`.
  - **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8995688302115749, 0.8368229403489976, 0.9920969557419316},`。
- **L273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9019890025620054, 0.8423510813517494, 0.9923144994581347},`.
  - **L273 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9019890025620054, 0.8423510813517494, 0.9923144994581347},`。
- **L274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9041189751589628, 0.8479541784289291, 0.992640232246134},`.
  - **L274 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9041189751589628, 0.8479541784289291, 0.992640232246134},`。
- **L275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9058973525369924, 0.8536784698518803, 0.9928660652275899},`.
  - **L275 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9058973525369924, 0.8536784698518803, 0.9928660652275899},`。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9073992963543378, 0.8594712199063077, 0.9931944239168621},`.
  - **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9073992963543378, 0.8594712199063077, 0.9931944239168621},`。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9086279741575543, 0.8653520270780298, 0.993459751831716},`.
  - **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9086279741575543, 0.8653520270780298, 0.993459751831716},`。
- **L278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9096655096337553, 0.8712669417189164, 0.9938665873784843},`.
  - **L278 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9096655096337553, 0.8712669417189164, 0.9938665873784843},`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9105035986931571, 0.8772607309343242, 0.9940839016190398},`.
  - **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9105035986931571, 0.8772607309343242, 0.9940839016190398},`。
- **L280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9112280528710646, 0.8832787634425558, 0.994319531984062},`.
  - **L280 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9112280528710646, 0.8832787634425558, 0.994319531984062},`。

### Lines 281-308 / 第 281-308 行

````cpp
 281:     {0.9118856791641718, 0.8893033549283489, 0.9945972975280256},
 282:     {0.9125160989249034, 0.8953200253413901, 0.994937094070283},
 283:     {0.9131521110159186, 0.9013364967475651, 0.9952011954414587},
 284:     {0.9138294807228358, 0.907320474034156, 0.9955652276122916},
 285:     {0.9145934489263029, 0.9132934622711121, 0.9957533433444864},
 286:     {0.9154721624193913, 0.9192261671429544, 0.9959406615249712},
 287:     {0.9165162261152294, 0.9251182443147156, 0.9960169412606291},
 288:     {0.9177621627085478, 0.9309558310395318, 0.9960174468690394},
 289:     {0.9192000090908993, 0.9367250258396427, 0.9960919139576695},
 290:     {0.920883428013182, 0.9424235334496841, 0.9961424557894082},
 291:     {0.922780751860871, 0.9480459375852194, 0.9963020587068238},
 292:     {0.9250092107169314, 0.9535821762261815, 0.9963856518750184},
 293:     {0.9275356684808912, 0.9590268903215627, 0.996529920191361},
 294:     {0.9304408284972104, 0.9643654252860402, 0.996676965947248},
 295:     {0.9338239050383462, 0.9695749605431102, 0.9967943472276378},
 296:     {0.9376406019134518, 0.97465362230454, 0.9970154826822965},
 297:     {0.9421630827040032, 0.9795433335221727, 0.9971921136372807},
 298:     {0.9473578426139436, 0.9842358767121905, 0.9974820554788658},
 299:     {0.9536370282349044, 0.9886296256370345, 0.9977639971129887},
 300:     {0.961277626215249, 0.9926367016857381, 0.9981081638729355},
 301:     {0.9706782225332987, 0.996129580837985, 0.9985982664103797},
 302:     {0.982998706722621, 0.9987706424331216, 0.9991799080785383},
 303:     {1.0, 1.0, 1.0},
 304: }};
 305: 
 306: template <typename Real>
 307: static constexpr std::array<std::array<Real, 3>, 256> kindlmann_data_ = {{
 308:     {0.0, 0.0, 0.0},
````
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9118856791641718, 0.8893033549283489, 0.9945972975280256},`.
  - **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9118856791641718, 0.8893033549283489, 0.9945972975280256},`。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9125160989249034, 0.8953200253413901, 0.994937094070283},`.
  - **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9125160989249034, 0.8953200253413901, 0.994937094070283},`。
- **L283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9131521110159186, 0.9013364967475651, 0.9952011954414587},`.
  - **L283 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9131521110159186, 0.9013364967475651, 0.9952011954414587},`。
- **L284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9138294807228358, 0.907320474034156, 0.9955652276122916},`.
  - **L284 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9138294807228358, 0.907320474034156, 0.9955652276122916},`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9145934489263029, 0.9132934622711121, 0.9957533433444864},`.
  - **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9145934489263029, 0.9132934622711121, 0.9957533433444864},`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9154721624193913, 0.9192261671429544, 0.9959406615249712},`.
  - **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9154721624193913, 0.9192261671429544, 0.9959406615249712},`。
- **L287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9165162261152294, 0.9251182443147156, 0.9960169412606291},`.
  - **L287 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9165162261152294, 0.9251182443147156, 0.9960169412606291},`。
- **L288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9177621627085478, 0.9309558310395318, 0.9960174468690394},`.
  - **L288 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9177621627085478, 0.9309558310395318, 0.9960174468690394},`。
- **L289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9192000090908993, 0.9367250258396427, 0.9960919139576695},`.
  - **L289 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9192000090908993, 0.9367250258396427, 0.9960919139576695},`。
- **L290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.920883428013182, 0.9424235334496841, 0.9961424557894082},`.
  - **L290 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.920883428013182, 0.9424235334496841, 0.9961424557894082},`。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.922780751860871, 0.9480459375852194, 0.9963020587068238},`.
  - **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.922780751860871, 0.9480459375852194, 0.9963020587068238},`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9250092107169314, 0.9535821762261815, 0.9963856518750184},`.
  - **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9250092107169314, 0.9535821762261815, 0.9963856518750184},`。
- **L293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9275356684808912, 0.9590268903215627, 0.996529920191361},`.
  - **L293 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9275356684808912, 0.9590268903215627, 0.996529920191361},`。
- **L294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9304408284972104, 0.9643654252860402, 0.996676965947248},`.
  - **L294 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9304408284972104, 0.9643654252860402, 0.996676965947248},`。
- **L295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9338239050383462, 0.9695749605431102, 0.9967943472276378},`.
  - **L295 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9338239050383462, 0.9695749605431102, 0.9967943472276378},`。
- **L296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9376406019134518, 0.97465362230454, 0.9970154826822965},`.
  - **L296 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9376406019134518, 0.97465362230454, 0.9970154826822965},`。
- **L297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9421630827040032, 0.9795433335221727, 0.9971921136372807},`.
  - **L297 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9421630827040032, 0.9795433335221727, 0.9971921136372807},`。
- **L298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9473578426139436, 0.9842358767121905, 0.9974820554788658},`.
  - **L298 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9473578426139436, 0.9842358767121905, 0.9974820554788658},`。
- **L299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9536370282349044, 0.9886296256370345, 0.9977639971129887},`.
  - **L299 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9536370282349044, 0.9886296256370345, 0.9977639971129887},`。
- **L300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.961277626215249, 0.9926367016857381, 0.9981081638729355},`.
  - **L300 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.961277626215249, 0.9926367016857381, 0.9981081638729355},`。
- **L301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9706782225332987, 0.996129580837985, 0.9985982664103797},`.
  - **L301 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9706782225332987, 0.996129580837985, 0.9985982664103797},`。
- **L302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.982998706722621, 0.9987706424331216, 0.9991799080785383},`.
  - **L302 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.982998706722621, 0.9987706424331216, 0.9991799080785383},`。
- **L303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{1.0, 1.0, 1.0},`.
  - **L303 CN**: 继续一个多行参数列表、初始化器或聚合项：`{1.0, 1.0, 1.0},`。
- **L304 EN**: Executes a standalone statement or declaration: `}};`.
  - **L304 CN**: 执行一条独立语句或声明：`}};`。
- **L305 EN**: Blank line separating nearby declarations or logic.
  - **L305 CN**: 空行，用于分隔相邻声明或逻辑。
- **L306 EN**: Introduces template parameters or specialization context: `template <typename Real>`.
  - **L306 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real>`。
- **L307 EN**: Continues the surrounding expression or declaration: `static constexpr std::array<std::array<Real, 3>, 256> kindlmann_data_ = {{`.
  - **L307 CN**: 继续构造周围的表达式或声明：`static constexpr std::array<std::array<Real, 3>, 256> kindlmann_data_ = {{`。
- **L308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0, 0.0, 0.0},`.
  - **L308 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0, 0.0, 0.0},`。

### Lines 309-336 / 第 309-336 行

````cpp
 309:     {0.017846074066284252, 0.0009158559874893362, 0.016056295374498146},
 310:     {0.03572864786642702, 0.0017291229250328806, 0.03302143519907636},
 311:     {0.05220741890989234, 0.002509857835251149, 0.04992588792612627},
 312:     {0.06579320887318146, 0.0031951554951018895, 0.06457251900523656},
 313:     {0.07760358429899875, 0.003724534058880596, 0.07776920342353211},
 314:     {0.08789358790850602, 0.004294574826048965, 0.0897198393739522},
 315:     {0.09712335630575433, 0.004842625166895046, 0.10086394984030281},
 316:     {0.10521029512901023, 0.005373953338851922, 0.11223547956850252},
 317:     {0.11206559338234914, 0.005946169553289608, 0.12432777397900399},
 318:     {0.11777117178354526, 0.006573754050745371, 0.13716171084354922},
 319:     {0.12274092212849055, 0.007190967398920267, 0.15010297798342745},
 320:     {0.12721746956233745, 0.007765930791621213, 0.1628260628965371},
 321:     {0.13120675065695167, 0.008361505755992952, 0.17526973313157249},
 322:     {0.13476383339950823, 0.008965513206303781, 0.1874768588728739},
 323:     {0.1379416850964659, 0.009551513984339633, 0.1995014270040738},
 324:     {0.14078915018940993, 0.010080340518883863, 0.21140807652685528},
 325:     {0.14328800070233533, 0.010654292784112113, 0.22305086172802313},
 326:     {0.1454925697745392, 0.011223945666441406, 0.23450000042491753},
 327:     {0.14744975665550916, 0.01172962096468942, 0.24583568409411483},
 328:     {0.14915607230438568, 0.012265761398485226, 0.2569125532571454},
 329:     {0.15064370028657081, 0.012825896285586958, 0.2678146545485108},
 330:     {0.15192214754665317, 0.013340464397739584, 0.27899859238410224},
 331:     {0.1529680602403743, 0.01383169406084123, 0.29046429953683084},
 332:     {0.15375355050540976, 0.014469905953793126, 0.3019580272214856},
 333:     {0.1542908116514455, 0.014991396474282258, 0.313852441550019},
 334:     {0.15455584740256964, 0.015566553274758022, 0.3258902207027679},
 335:     {0.15454470889937796, 0.01619279407587136, 0.3380624435826929},
 336:     {0.15424074730776174, 0.01677345343135211, 0.3504951935746775},
````
- **L309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.017846074066284252, 0.0009158559874893362, 0.016056295374498146},`.
  - **L309 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.017846074066284252, 0.0009158559874893362, 0.016056295374498146},`。
- **L310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03572864786642702, 0.0017291229250328806, 0.03302143519907636},`.
  - **L310 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03572864786642702, 0.0017291229250328806, 0.03302143519907636},`。
- **L311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.05220741890989234, 0.002509857835251149, 0.04992588792612627},`.
  - **L311 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.05220741890989234, 0.002509857835251149, 0.04992588792612627},`。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.06579320887318146, 0.0031951554951018895, 0.06457251900523656},`.
  - **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.06579320887318146, 0.0031951554951018895, 0.06457251900523656},`。
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07760358429899875, 0.003724534058880596, 0.07776920342353211},`.
  - **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07760358429899875, 0.003724534058880596, 0.07776920342353211},`。
- **L314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.08789358790850602, 0.004294574826048965, 0.0897198393739522},`.
  - **L314 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.08789358790850602, 0.004294574826048965, 0.0897198393739522},`。
- **L315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.09712335630575433, 0.004842625166895046, 0.10086394984030281},`.
  - **L315 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.09712335630575433, 0.004842625166895046, 0.10086394984030281},`。
- **L316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.10521029512901023, 0.005373953338851922, 0.11223547956850252},`.
  - **L316 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.10521029512901023, 0.005373953338851922, 0.11223547956850252},`。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11206559338234914, 0.005946169553289608, 0.12432777397900399},`.
  - **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11206559338234914, 0.005946169553289608, 0.12432777397900399},`。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11777117178354526, 0.006573754050745371, 0.13716171084354922},`.
  - **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11777117178354526, 0.006573754050745371, 0.13716171084354922},`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12274092212849055, 0.007190967398920267, 0.15010297798342745},`.
  - **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12274092212849055, 0.007190967398920267, 0.15010297798342745},`。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12721746956233745, 0.007765930791621213, 0.1628260628965371},`.
  - **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12721746956233745, 0.007765930791621213, 0.1628260628965371},`。
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13120675065695167, 0.008361505755992952, 0.17526973313157249},`.
  - **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13120675065695167, 0.008361505755992952, 0.17526973313157249},`。
- **L322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13476383339950823, 0.008965513206303781, 0.1874768588728739},`.
  - **L322 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13476383339950823, 0.008965513206303781, 0.1874768588728739},`。
- **L323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1379416850964659, 0.009551513984339633, 0.1995014270040738},`.
  - **L323 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1379416850964659, 0.009551513984339633, 0.1995014270040738},`。
- **L324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14078915018940993, 0.010080340518883863, 0.21140807652685528},`.
  - **L324 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14078915018940993, 0.010080340518883863, 0.21140807652685528},`。
- **L325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14328800070233533, 0.010654292784112113, 0.22305086172802313},`.
  - **L325 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14328800070233533, 0.010654292784112113, 0.22305086172802313},`。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1454925697745392, 0.011223945666441406, 0.23450000042491753},`.
  - **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1454925697745392, 0.011223945666441406, 0.23450000042491753},`。
- **L327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14744975665550916, 0.01172962096468942, 0.24583568409411483},`.
  - **L327 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14744975665550916, 0.01172962096468942, 0.24583568409411483},`。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14915607230438568, 0.012265761398485226, 0.2569125532571454},`.
  - **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14915607230438568, 0.012265761398485226, 0.2569125532571454},`。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15064370028657081, 0.012825896285586958, 0.2678146545485108},`.
  - **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15064370028657081, 0.012825896285586958, 0.2678146545485108},`。
- **L330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15192214754665317, 0.013340464397739584, 0.27899859238410224},`.
  - **L330 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15192214754665317, 0.013340464397739584, 0.27899859238410224},`。
- **L331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1529680602403743, 0.01383169406084123, 0.29046429953683084},`.
  - **L331 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1529680602403743, 0.01383169406084123, 0.29046429953683084},`。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15375355050540976, 0.014469905953793126, 0.3019580272214856},`.
  - **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15375355050540976, 0.014469905953793126, 0.3019580272214856},`。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1542908116514455, 0.014991396474282258, 0.313852441550019},`.
  - **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1542908116514455, 0.014991396474282258, 0.313852441550019},`。
- **L334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15455584740256964, 0.015566553274758022, 0.3258902207027679},`.
  - **L334 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15455584740256964, 0.015566553274758022, 0.3258902207027679},`。
- **L335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15454470889937796, 0.01619279407587136, 0.3380624435826929},`.
  - **L335 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15454470889937796, 0.01619279407587136, 0.3380624435826929},`。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15424074730776174, 0.01677345343135211, 0.3504951935746775},`.
  - **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15424074730776174, 0.01677345343135211, 0.3504951935746775},`。

### Lines 337-364 / 第 337-364 行

````cpp
 337:     {0.15362513362574176, 0.01730005530615292, 0.3631847587056686},
 338:     {0.15272770714157807, 0.017955417060815455, 0.3758470094405716},
 339:     {0.15150259153104584, 0.018542834077193353, 0.3887481108848792},
 340:     {0.14996750216169222, 0.019150121523940637, 0.4017384234617481},
 341:     {0.1481223688773872, 0.019769579646988222, 0.4148055969839783},
 342:     {0.14596849097254724, 0.02039268583437749, 0.42793691410477086},
 343:     {0.14350871584895447, 0.021010135344188375, 0.4411193724446502},
 344:     {0.14081675084553444, 0.021716621813360978, 0.4541885105212106},
 345:     {0.13777021342781093, 0.022293516296525755, 0.4674318891638602},
 346:     {0.13452661891393936, 0.022940593473192553, 0.48053217251642305},
 347:     {0.13103033265010522, 0.023541432666866674, 0.49362776425328886},
 348:     {0.12740778581316103, 0.024195126427194676, 0.5065484866366465},
 349:     {0.12359314328868806, 0.024781571830105195, 0.5194370648731684},
 350:     {0.11974712167592705, 0.025403149292983516, 0.5321219022221657},
 351:     {0.1159404877051333, 0.0260531205128538, 0.5445888017818501},
 352:     {0.11209206787148834, 0.02660656407054731, 0.5569877413493692},
 353:     {0.10842241040440373, 0.02717266184945997, 0.5691469684334989},
 354:     {0.1050253848722921, 0.027746789078543728, 0.5810568608426063},
 355:     {0.101999609017976, 0.02832560680437759, 0.5927097735404816},
 356:     {0.09922826785261996, 0.028779550542447317, 0.6042668353483279},
 357:     {0.09722602817414985, 0.02936178919960733, 0.6153923636579981},
 358:     {0.09528935573037353, 0.029890777438344337, 0.6264955320241609},
 359:     {0.0930823731478326, 0.030438044833523397, 0.6376509097116411},
 360:     {0.0781891239412737, 0.031121586300131683, 0.6525577781586464},
 361:     {0.05857961917089497, 0.031924832133126, 0.6676204841351329},
 362:     {0.03284057845765246, 0.03469657698669593, 0.6813738713631898},
 363:     {0.032679442069734624, 0.04697971049610799, 0.6849034147771731},
 364:     {0.033207560139530704, 0.05995495438213944, 0.6868241583911188},
````
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15362513362574176, 0.01730005530615292, 0.3631847587056686},`.
  - **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15362513362574176, 0.01730005530615292, 0.3631847587056686},`。
- **L338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15272770714157807, 0.017955417060815455, 0.3758470094405716},`.
  - **L338 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15272770714157807, 0.017955417060815455, 0.3758470094405716},`。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15150259153104584, 0.018542834077193353, 0.3887481108848792},`.
  - **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15150259153104584, 0.018542834077193353, 0.3887481108848792},`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14996750216169222, 0.019150121523940637, 0.4017384234617481},`.
  - **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14996750216169222, 0.019150121523940637, 0.4017384234617481},`。
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1481223688773872, 0.019769579646988222, 0.4148055969839783},`.
  - **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1481223688773872, 0.019769579646988222, 0.4148055969839783},`。
- **L342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14596849097254724, 0.02039268583437749, 0.42793691410477086},`.
  - **L342 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14596849097254724, 0.02039268583437749, 0.42793691410477086},`。
- **L343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14350871584895447, 0.021010135344188375, 0.4411193724446502},`.
  - **L343 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14350871584895447, 0.021010135344188375, 0.4411193724446502},`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14081675084553444, 0.021716621813360978, 0.4541885105212106},`.
  - **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14081675084553444, 0.021716621813360978, 0.4541885105212106},`。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13777021342781093, 0.022293516296525755, 0.4674318891638602},`.
  - **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13777021342781093, 0.022293516296525755, 0.4674318891638602},`。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13452661891393936, 0.022940593473192553, 0.48053217251642305},`.
  - **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13452661891393936, 0.022940593473192553, 0.48053217251642305},`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13103033265010522, 0.023541432666866674, 0.49362776425328886},`.
  - **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13103033265010522, 0.023541432666866674, 0.49362776425328886},`。
- **L348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12740778581316103, 0.024195126427194676, 0.5065484866366465},`.
  - **L348 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12740778581316103, 0.024195126427194676, 0.5065484866366465},`。
- **L349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12359314328868806, 0.024781571830105195, 0.5194370648731684},`.
  - **L349 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12359314328868806, 0.024781571830105195, 0.5194370648731684},`。
- **L350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11974712167592705, 0.025403149292983516, 0.5321219022221657},`.
  - **L350 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11974712167592705, 0.025403149292983516, 0.5321219022221657},`。
- **L351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1159404877051333, 0.0260531205128538, 0.5445888017818501},`.
  - **L351 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1159404877051333, 0.0260531205128538, 0.5445888017818501},`。
- **L352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11209206787148834, 0.02660656407054731, 0.5569877413493692},`.
  - **L352 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11209206787148834, 0.02660656407054731, 0.5569877413493692},`。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.10842241040440373, 0.02717266184945997, 0.5691469684334989},`.
  - **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.10842241040440373, 0.02717266184945997, 0.5691469684334989},`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1050253848722921, 0.027746789078543728, 0.5810568608426063},`.
  - **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1050253848722921, 0.027746789078543728, 0.5810568608426063},`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.101999609017976, 0.02832560680437759, 0.5927097735404816},`.
  - **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.101999609017976, 0.02832560680437759, 0.5927097735404816},`。
- **L356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.09922826785261996, 0.028779550542447317, 0.6042668353483279},`.
  - **L356 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.09922826785261996, 0.028779550542447317, 0.6042668353483279},`。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.09722602817414985, 0.02936178919960733, 0.6153923636579981},`.
  - **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.09722602817414985, 0.02936178919960733, 0.6153923636579981},`。
- **L358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.09528935573037353, 0.029890777438344337, 0.6264955320241609},`.
  - **L358 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.09528935573037353, 0.029890777438344337, 0.6264955320241609},`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0930823731478326, 0.030438044833523397, 0.6376509097116411},`.
  - **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0930823731478326, 0.030438044833523397, 0.6376509097116411},`。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0781891239412737, 0.031121586300131683, 0.6525577781586464},`.
  - **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0781891239412737, 0.031121586300131683, 0.6525577781586464},`。
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.05857961917089497, 0.031924832133126, 0.6676204841351329},`.
  - **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.05857961917089497, 0.031924832133126, 0.6676204841351329},`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03284057845765246, 0.03469657698669593, 0.6813738713631898},`.
  - **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03284057845765246, 0.03469657698669593, 0.6813738713631898},`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.032679442069734624, 0.04697971049610799, 0.6849034147771731},`.
  - **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.032679442069734624, 0.04697971049610799, 0.6849034147771731},`。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.033207560139530704, 0.05995495438213944, 0.6868241583911188},`.
  - **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.033207560139530704, 0.05995495438213944, 0.6868241583911188},`。

### Lines 365-392 / 第 365-392 行

````cpp
 365:     {0.033317187200492364, 0.07300667062971217, 0.6874759761236308},
 366:     {0.033087445334472984, 0.08601513051125326, 0.6868754153808352},
 367:     {0.03265381069556877, 0.09888929701373608, 0.6850522082772128},
 368:     {0.03277467744530855, 0.11161079522124029, 0.6818870111240353},
 369:     {0.0325022291000698, 0.1240111603824268, 0.6777553638535389},
 370:     {0.032066045593596146, 0.13606903293962488, 0.6727110790151253},
 371:     {0.03227600851941834, 0.14780096691985975, 0.6666596846794193},
 372:     {0.031649331579416705, 0.15909372791133777, 0.6601264529755937},
 373:     {0.03155801725217711, 0.17001566158899392, 0.6528643797965168},
 374:     {0.0310838330844518, 0.18051935094604918, 0.6452318387821873},
 375:     {0.030432973047734187, 0.19062325974122588, 0.6372784105620277},
 376:     {0.030338619295788537, 0.20036054096647732, 0.6289128168374558},
 377:     {0.029853960367767295, 0.20971568331952556, 0.6204629562646412},
 378:     {0.02966850358552374, 0.2187242415010579, 0.6118301548875958},
 379:     {0.02882374869999708, 0.2273866229818785, 0.603314553849001},
 380:     {0.028500638040224537, 0.2357441994433061, 0.5946825020006182},
 381:     {0.028245919974351025, 0.24380777892174738, 0.5860925629625072},
 382:     {0.02760771294131195, 0.2515940717685919, 0.5776869157187913},
 383:     {0.02717197158834595, 0.25912806556578544, 0.5693601411266609},
 384:     {0.026982221792628543, 0.266427065214078, 0.5611305112918692},
 385:     {0.026559642397133442, 0.27350794101395076, 0.5531226098917067},
 386:     {0.02644787880084816, 0.2803874742418932, 0.5452351099365369},
 387:     {0.02564842515734134, 0.2870852596241667, 0.5376830481507792},
 388:     {0.02570596827933925, 0.29360869815637475, 0.5301627048805485},
 389:     {0.025105902288682952, 0.2999809743840178, 0.5229767137199569},
 390:     {0.024870375536254246, 0.30620709089548, 0.5159307080617925},
 391:     {0.024495137563131944, 0.3123034508268453, 0.5091188274555094},
 392:     {0.02397954323509251, 0.3182816255246543, 0.502536537756261},
````
- **L365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.033317187200492364, 0.07300667062971217, 0.6874759761236308},`.
  - **L365 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.033317187200492364, 0.07300667062971217, 0.6874759761236308},`。
- **L366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.033087445334472984, 0.08601513051125326, 0.6868754153808352},`.
  - **L366 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.033087445334472984, 0.08601513051125326, 0.6868754153808352},`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03265381069556877, 0.09888929701373608, 0.6850522082772128},`.
  - **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03265381069556877, 0.09888929701373608, 0.6850522082772128},`。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03277467744530855, 0.11161079522124029, 0.6818870111240353},`.
  - **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03277467744530855, 0.11161079522124029, 0.6818870111240353},`。
- **L369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0325022291000698, 0.1240111603824268, 0.6777553638535389},`.
  - **L369 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0325022291000698, 0.1240111603824268, 0.6777553638535389},`。
- **L370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.032066045593596146, 0.13606903293962488, 0.6727110790151253},`.
  - **L370 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.032066045593596146, 0.13606903293962488, 0.6727110790151253},`。
- **L371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03227600851941834, 0.14780096691985975, 0.6666596846794193},`.
  - **L371 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03227600851941834, 0.14780096691985975, 0.6666596846794193},`。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.031649331579416705, 0.15909372791133777, 0.6601264529755937},`.
  - **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.031649331579416705, 0.15909372791133777, 0.6601264529755937},`。
- **L373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03155801725217711, 0.17001566158899392, 0.6528643797965168},`.
  - **L373 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03155801725217711, 0.17001566158899392, 0.6528643797965168},`。
- **L374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0310838330844518, 0.18051935094604918, 0.6452318387821873},`.
  - **L374 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0310838330844518, 0.18051935094604918, 0.6452318387821873},`。
- **L375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.030432973047734187, 0.19062325974122588, 0.6372784105620277},`.
  - **L375 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.030432973047734187, 0.19062325974122588, 0.6372784105620277},`。
- **L376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.030338619295788537, 0.20036054096647732, 0.6289128168374558},`.
  - **L376 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.030338619295788537, 0.20036054096647732, 0.6289128168374558},`。
- **L377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.029853960367767295, 0.20971568331952556, 0.6204629562646412},`.
  - **L377 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.029853960367767295, 0.20971568331952556, 0.6204629562646412},`。
- **L378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02966850358552374, 0.2187242415010579, 0.6118301548875958},`.
  - **L378 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02966850358552374, 0.2187242415010579, 0.6118301548875958},`。
- **L379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02882374869999708, 0.2273866229818785, 0.603314553849001},`.
  - **L379 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02882374869999708, 0.2273866229818785, 0.603314553849001},`。
- **L380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.028500638040224537, 0.2357441994433061, 0.5946825020006182},`.
  - **L380 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.028500638040224537, 0.2357441994433061, 0.5946825020006182},`。
- **L381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.028245919974351025, 0.24380777892174738, 0.5860925629625072},`.
  - **L381 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.028245919974351025, 0.24380777892174738, 0.5860925629625072},`。
- **L382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02760771294131195, 0.2515940717685919, 0.5776869157187913},`.
  - **L382 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02760771294131195, 0.2515940717685919, 0.5776869157187913},`。
- **L383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02717197158834595, 0.25912806556578544, 0.5693601411266609},`.
  - **L383 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02717197158834595, 0.25912806556578544, 0.5693601411266609},`。
- **L384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.026982221792628543, 0.266427065214078, 0.5611305112918692},`.
  - **L384 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.026982221792628543, 0.266427065214078, 0.5611305112918692},`。
- **L385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.026559642397133442, 0.27350794101395076, 0.5531226098917067},`.
  - **L385 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.026559642397133442, 0.27350794101395076, 0.5531226098917067},`。
- **L386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02644787880084816, 0.2803874742418932, 0.5452351099365369},`.
  - **L386 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02644787880084816, 0.2803874742418932, 0.5452351099365369},`。
- **L387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02564842515734134, 0.2870852596241667, 0.5376830481507792},`.
  - **L387 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02564842515734134, 0.2870852596241667, 0.5376830481507792},`。
- **L388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02570596827933925, 0.29360869815637475, 0.5301627048805485},`.
  - **L388 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02570596827933925, 0.29360869815637475, 0.5301627048805485},`。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.025105902288682952, 0.2999809743840178, 0.5229767137199569},`.
  - **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.025105902288682952, 0.2999809743840178, 0.5229767137199569},`。
- **L390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.024870375536254246, 0.30620709089548, 0.5159307080617925},`.
  - **L390 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.024870375536254246, 0.30620709089548, 0.5159307080617925},`。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.024495137563131944, 0.3123034508268453, 0.5091188274555094},`.
  - **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.024495137563131944, 0.3123034508268453, 0.5091188274555094},`。
- **L392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02397954323509251, 0.3182816255246543, 0.502536537756261},`.
  - **L392 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02397954323509251, 0.3182816255246543, 0.502536537756261},`。

### Lines 393-420 / 第 393-420 行

````cpp
 393:     {0.023829708770758788, 0.32414426165027227, 0.4960985638991457},
 394:     {0.023532604704116965, 0.32990776857516413, 0.48988582950117815},
 395:     {0.02307993829976581, 0.3355813090448093, 0.4838916171846975},
 396:     {0.022979595210601424, 0.3411632670690572, 0.47803935490382515},
 397:     {0.022707635361622264, 0.3466702701862077, 0.4723976815702628},
 398:     {0.022250414084789737, 0.352109624090965, 0.4669584702823865},
 399:     {0.022124477998067993, 0.3574761812445124, 0.46165444016667906},
 400:     {0.021791069977336153, 0.3627869887174417, 0.45654262929804507},
 401:     {0.021776256319502838, 0.3680348834809229, 0.45156215250307846},
 402:     {0.02152936248024083, 0.3732373044965933, 0.44676258718151424},
 403:     {0.02158822978060292, 0.37838527471660577, 0.44208955676541317},
 404:     {0.021388037427047302, 0.38349668536630105, 0.43758535324043873},
 405:     {0.020906126379826747, 0.3885760631680783, 0.4332402802381849},
 406:     {0.02070114401850818, 0.3936120031480679, 0.4290098739833766},
 407:     {0.020774708005350834, 0.3986070226842801, 0.4248945065220385},
 408:     {0.020526909471275397, 0.4035800658477305, 0.4209222889365068},
 409:     {0.01993018897938756, 0.4085347069464005, 0.4170831873583663},
 410:     {0.02020289046425452, 0.4134392235922892, 0.41332502021802126},
 411:     {0.020207032023267582, 0.41834155873012324, 0.40954621812568487},
 412:     {0.020205716119178502, 0.42324888777657366, 0.4055896035873459},
 413:     {0.020829051716263414, 0.42814253234088623, 0.4014443452785458},
 414:     {0.020817169253550774, 0.4330580290829061, 0.3971329152020192},
 415:     {0.02143446555544063, 0.437958395126164, 0.3926385324661419},
 416:     {0.02140951927285399, 0.44287991873506705, 0.38796855906767097},
 417:     {0.021375101041108045, 0.4478037488631326, 0.3831160408679206},
 418:     {0.021978368208421244, 0.45271034115960945, 0.3780894735135355},
 419:     {0.021925941019470395, 0.45763704847590275, 0.372873157602702},
 420:     {0.022518324495068114, 0.4625451606740822, 0.3674890229173533},
````
- **L393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.023829708770758788, 0.32414426165027227, 0.4960985638991457},`.
  - **L393 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.023829708770758788, 0.32414426165027227, 0.4960985638991457},`。
- **L394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.023532604704116965, 0.32990776857516413, 0.48988582950117815},`.
  - **L394 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.023532604704116965, 0.32990776857516413, 0.48988582950117815},`。
- **L395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02307993829976581, 0.3355813090448093, 0.4838916171846975},`.
  - **L395 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02307993829976581, 0.3355813090448093, 0.4838916171846975},`。
- **L396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.022979595210601424, 0.3411632670690572, 0.47803935490382515},`.
  - **L396 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.022979595210601424, 0.3411632670690572, 0.47803935490382515},`。
- **L397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.022707635361622264, 0.3466702701862077, 0.4723976815702628},`.
  - **L397 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.022707635361622264, 0.3466702701862077, 0.4723976815702628},`。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.022250414084789737, 0.352109624090965, 0.4669584702823865},`.
  - **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.022250414084789737, 0.352109624090965, 0.4669584702823865},`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.022124477998067993, 0.3574761812445124, 0.46165444016667906},`.
  - **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.022124477998067993, 0.3574761812445124, 0.46165444016667906},`。
- **L400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.021791069977336153, 0.3627869887174417, 0.45654262929804507},`.
  - **L400 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.021791069977336153, 0.3627869887174417, 0.45654262929804507},`。
- **L401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.021776256319502838, 0.3680348834809229, 0.45156215250307846},`.
  - **L401 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.021776256319502838, 0.3680348834809229, 0.45156215250307846},`。
- **L402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02152936248024083, 0.3732373044965933, 0.44676258718151424},`.
  - **L402 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02152936248024083, 0.3732373044965933, 0.44676258718151424},`。
- **L403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02158822978060292, 0.37838527471660577, 0.44208955676541317},`.
  - **L403 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02158822978060292, 0.37838527471660577, 0.44208955676541317},`。
- **L404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.021388037427047302, 0.38349668536630105, 0.43758535324043873},`.
  - **L404 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.021388037427047302, 0.38349668536630105, 0.43758535324043873},`。
- **L405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.020906126379826747, 0.3885760631680783, 0.4332402802381849},`.
  - **L405 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.020906126379826747, 0.3885760631680783, 0.4332402802381849},`。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02070114401850818, 0.3936120031480679, 0.4290098739833766},`.
  - **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02070114401850818, 0.3936120031480679, 0.4290098739833766},`。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.020774708005350834, 0.3986070226842801, 0.4248945065220385},`.
  - **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.020774708005350834, 0.3986070226842801, 0.4248945065220385},`。
- **L408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.020526909471275397, 0.4035800658477305, 0.4209222889365068},`.
  - **L408 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.020526909471275397, 0.4035800658477305, 0.4209222889365068},`。
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01993018897938756, 0.4085347069464005, 0.4170831873583663},`.
  - **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01993018897938756, 0.4085347069464005, 0.4170831873583663},`。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02020289046425452, 0.4134392235922892, 0.41332502021802126},`.
  - **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02020289046425452, 0.4134392235922892, 0.41332502021802126},`。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.020207032023267582, 0.41834155873012324, 0.40954621812568487},`.
  - **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.020207032023267582, 0.41834155873012324, 0.40954621812568487},`。
- **L412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.020205716119178502, 0.42324888777657366, 0.4055896035873459},`.
  - **L412 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.020205716119178502, 0.42324888777657366, 0.4055896035873459},`。
- **L413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.020829051716263414, 0.42814253234088623, 0.4014443452785458},`.
  - **L413 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.020829051716263414, 0.42814253234088623, 0.4014443452785458},`。
- **L414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.020817169253550774, 0.4330580290829061, 0.3971329152020192},`.
  - **L414 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.020817169253550774, 0.4330580290829061, 0.3971329152020192},`。
- **L415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02143446555544063, 0.437958395126164, 0.3926385324661419},`.
  - **L415 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02143446555544063, 0.437958395126164, 0.3926385324661419},`。
- **L416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02140951927285399, 0.44287991873506705, 0.38796855906767097},`.
  - **L416 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02140951927285399, 0.44287991873506705, 0.38796855906767097},`。
- **L417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.021375101041108045, 0.4478037488631326, 0.3831160408679206},`.
  - **L417 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.021375101041108045, 0.4478037488631326, 0.3831160408679206},`。
- **L418 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.021978368208421244, 0.45271034115960945, 0.3780894735135355},`.
  - **L418 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.021978368208421244, 0.45271034115960945, 0.3780894735135355},`。
- **L419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.021925941019470395, 0.45763704847590275, 0.372873157602702},`.
  - **L419 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.021925941019470395, 0.45763704847590275, 0.372873157602702},`。
- **L420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.022518324495068114, 0.4625451606740822, 0.3674890229173533},`.
  - **L420 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.022518324495068114, 0.4625451606740822, 0.3674890229173533},`。

### Lines 421-448 / 第 421-448 行

````cpp
 421:     {0.0224439650744986, 0.4674727059721964, 0.3619056954025697},
 422:     {0.02302262600613305, 0.47238034113781563, 0.3561610413615427},
 423:     {0.022921527380028887, 0.4773067430685342, 0.350207729192515},
 424:     {0.023482745772902724, 0.4822119663418752, 0.34409988145088766},
 425:     {0.02334902491772799, 0.48713531181669306, 0.3377738746272117},
 426:     {0.023887996465323004, 0.4920362610259304, 0.33130046883268455},
 427:     {0.023714510501347055, 0.4969547158806737, 0.32459934406811614},
 428:     {0.024225153974701766, 0.5018496124079483, 0.3177583627477745},
 429:     {0.024723266142386315, 0.5067407770624851, 0.31073519425545465},
 430:     {0.02447812509827954, 0.5116485921195848, 0.3034698224412634},
 431:     {0.02493648885622696, 0.5165312197695955, 0.2960768968333586},
 432:     {0.0253764023681478, 0.5214089374064284, 0.28850111378145044},
 433:     {0.025795626075147713, 0.5262813821767296, 0.28074244497417117},
 434:     {0.025421474545393858, 0.5311694950761902, 0.2727216863169472},
 435:     {0.025780309667251394, 0.53603048885089, 0.2645922829614648},
 436:     {0.02610999696637235, 0.5408852158877796, 0.25628033494817076},
 437:     {0.026407437583934296, 0.5457333806719771, 0.2477862504464216},
 438:     {0.02666932808870506, 0.5505747064434545, 0.2391105880601489},
 439:     {0.02689217252655957, 0.5554089359982296, 0.2302540772313253},
 440:     {0.027072298940817408, 0.5602358325123109, 0.22121764273058586},
 441:     {0.027205880727598008, 0.5650551803951163, 0.2120024343570966},
 442:     {0.027288963146258696, 0.5698667861810437, 0.20260986337352},
 443:     {0.028204203304642753, 0.5746482086659772, 0.19318007629151898},
 444:     {0.028188559261174452, 0.5794437480873181, 0.18344751604265283},
 445:     {0.0281103207218881, 0.5842311106814237, 0.17354484850307106},
 446:     {0.028896130485080208, 0.5889876510327495, 0.16364429504261102},
 447:     {0.028695320134244477, 0.5937583183864393, 0.1534244803870241},
 448:     {0.029380675523176917, 0.5984978909930776, 0.14324398649586118},
````
- **L421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0224439650744986, 0.4674727059721964, 0.3619056954025697},`.
  - **L421 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0224439650744986, 0.4674727059721964, 0.3619056954025697},`。
- **L422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02302262600613305, 0.47238034113781563, 0.3561610413615427},`.
  - **L422 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02302262600613305, 0.47238034113781563, 0.3561610413615427},`。
- **L423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.022921527380028887, 0.4773067430685342, 0.350207729192515},`.
  - **L423 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.022921527380028887, 0.4773067430685342, 0.350207729192515},`。
- **L424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.023482745772902724, 0.4822119663418752, 0.34409988145088766},`.
  - **L424 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.023482745772902724, 0.4822119663418752, 0.34409988145088766},`。
- **L425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02334902491772799, 0.48713531181669306, 0.3377738746272117},`.
  - **L425 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02334902491772799, 0.48713531181669306, 0.3377738746272117},`。
- **L426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.023887996465323004, 0.4920362610259304, 0.33130046883268455},`.
  - **L426 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.023887996465323004, 0.4920362610259304, 0.33130046883268455},`。
- **L427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.023714510501347055, 0.4969547158806737, 0.32459934406811614},`.
  - **L427 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.023714510501347055, 0.4969547158806737, 0.32459934406811614},`。
- **L428 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.024225153974701766, 0.5018496124079483, 0.3177583627477745},`.
  - **L428 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.024225153974701766, 0.5018496124079483, 0.3177583627477745},`。
- **L429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.024723266142386315, 0.5067407770624851, 0.31073519425545465},`.
  - **L429 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.024723266142386315, 0.5067407770624851, 0.31073519425545465},`。
- **L430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02447812509827954, 0.5116485921195848, 0.3034698224412634},`.
  - **L430 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02447812509827954, 0.5116485921195848, 0.3034698224412634},`。
- **L431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02493648885622696, 0.5165312197695955, 0.2960768968333586},`.
  - **L431 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02493648885622696, 0.5165312197695955, 0.2960768968333586},`。
- **L432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0253764023681478, 0.5214089374064284, 0.28850111378145044},`.
  - **L432 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0253764023681478, 0.5214089374064284, 0.28850111378145044},`。
- **L433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.025795626075147713, 0.5262813821767296, 0.28074244497417117},`.
  - **L433 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.025795626075147713, 0.5262813821767296, 0.28074244497417117},`。
- **L434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.025421474545393858, 0.5311694950761902, 0.2727216863169472},`.
  - **L434 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.025421474545393858, 0.5311694950761902, 0.2727216863169472},`。
- **L435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.025780309667251394, 0.53603048885089, 0.2645922829614648},`.
  - **L435 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.025780309667251394, 0.53603048885089, 0.2645922829614648},`。
- **L436 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02610999696637235, 0.5408852158877796, 0.25628033494817076},`.
  - **L436 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02610999696637235, 0.5408852158877796, 0.25628033494817076},`。
- **L437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.026407437583934296, 0.5457333806719771, 0.2477862504464216},`.
  - **L437 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.026407437583934296, 0.5457333806719771, 0.2477862504464216},`。
- **L438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02666932808870506, 0.5505747064434545, 0.2391105880601489},`.
  - **L438 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02666932808870506, 0.5505747064434545, 0.2391105880601489},`。
- **L439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02689217252655957, 0.5554089359982296, 0.2302540772313253},`.
  - **L439 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02689217252655957, 0.5554089359982296, 0.2302540772313253},`。
- **L440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.027072298940817408, 0.5602358325123109, 0.22121764273058586},`.
  - **L440 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.027072298940817408, 0.5602358325123109, 0.22121764273058586},`。
- **L441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.027205880727598008, 0.5650551803951163, 0.2120024343570966},`.
  - **L441 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.027205880727598008, 0.5650551803951163, 0.2120024343570966},`。
- **L442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.027288963146258696, 0.5698667861810437, 0.20260986337352},`.
  - **L442 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.027288963146258696, 0.5698667861810437, 0.20260986337352},`。
- **L443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.028204203304642753, 0.5746482086659772, 0.19318007629151898},`.
  - **L443 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.028204203304642753, 0.5746482086659772, 0.19318007629151898},`。
- **L444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.028188559261174452, 0.5794437480873181, 0.18344751604265283},`.
  - **L444 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.028188559261174452, 0.5794437480873181, 0.18344751604265283},`。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0281103207218881, 0.5842311106814237, 0.17354484850307106},`.
  - **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0281103207218881, 0.5842311106814237, 0.17354484850307106},`。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.028896130485080208, 0.5889876510327495, 0.16364429504261102},`.
  - **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.028896130485080208, 0.5889876510327495, 0.16364429504261102},`。
- **L447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.028695320134244477, 0.5937583183864393, 0.1534244803870241},`.
  - **L447 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.028695320134244477, 0.5937583183864393, 0.1534244803870241},`。
- **L448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.029380675523176917, 0.5984978909930776, 0.14324398649586118},`.
  - **L448 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.029380675523176917, 0.5984978909930776, 0.14324398649586118},`。

### Lines 449-476 / 第 449-476 行

````cpp
 449:     {0.029041895792470927, 0.6032517278747106, 0.13273404723048013},
 450:     {0.029612109748158165, 0.6079743365463844, 0.12231515476812047},
 451:     {0.030127173837479853, 0.6126885119214123, 0.1118109733754355},
 452:     {0.029563230689198295, 0.6174173890888045, 0.10096978844729684},
 453:     {0.029944189232757292, 0.6221151405821017, 0.09035027988372307},
 454:     {0.030261789165931138, 0.6268049184840729, 0.07974541336580176},
 455:     {0.030514080739125825, 0.6314869729283614, 0.06921734345076924},
 456:     {0.030699652650745195, 0.6361616116842398, 0.05885329347452114},
 457:     {0.0308177148230897, 0.6408292087724975, 0.04877724313606999},
 458:     {0.03086818674766468, 0.6454902160815501, 0.03915677764354433},
 459:     {0.03197034466002908, 0.6501217563441514, 0.031444380194868886},
 460:     {0.046293996677654235, 0.6544720579771781, 0.031637291074855106},
 461:     {0.058794558570863804, 0.6588271549701028, 0.0317521581310916},
 462:     {0.06503863628200135, 0.6633055777623768, 0.03209873268943441},
 463:     {0.07023149742711651, 0.6678120145394701, 0.03183433002082528},
 464:     {0.0767138128414942, 0.6722807177648771, 0.032132076596637124},
 465:     {0.08432067120858369, 0.6767139870108101, 0.032338563602709626},
 466:     {0.09294277780722546, 0.6811085964562938, 0.032444071987208487},
 467:     {0.10305233592409935, 0.6854378106573311, 0.03307401108271617},
 468:     {0.11331609119127764, 0.6897459817142692, 0.03295480891791886},
 469:     {0.12476614069574826, 0.6939830479703625, 0.033353619722170935},
 470:     {0.13673711440841982, 0.6981699856866227, 0.03362590965526557},
 471:     {0.1491636843492167, 0.7023042408994585, 0.033762275984369046},
 472:     {0.16199129317074465, 0.706383361101949, 0.03375336199700755},
 473:     {0.17553308151438088, 0.7103815097863063, 0.03426353710068628},
 474:     {0.18933948484375165, 0.7143200103636559, 0.03462289684303036},
 475:     {0.20338652270943824, 0.7181967154976012, 0.03482214714504671},
 476:     {0.21765378276012448, 0.7220095660591789, 0.0348520554844732},
````
- **L449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.029041895792470927, 0.6032517278747106, 0.13273404723048013},`.
  - **L449 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.029041895792470927, 0.6032517278747106, 0.13273404723048013},`。
- **L450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.029612109748158165, 0.6079743365463844, 0.12231515476812047},`.
  - **L450 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.029612109748158165, 0.6079743365463844, 0.12231515476812047},`。
- **L451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.030127173837479853, 0.6126885119214123, 0.1118109733754355},`.
  - **L451 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.030127173837479853, 0.6126885119214123, 0.1118109733754355},`。
- **L452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.029563230689198295, 0.6174173890888045, 0.10096978844729684},`.
  - **L452 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.029563230689198295, 0.6174173890888045, 0.10096978844729684},`。
- **L453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.029944189232757292, 0.6221151405821017, 0.09035027988372307},`.
  - **L453 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.029944189232757292, 0.6221151405821017, 0.09035027988372307},`。
- **L454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.030261789165931138, 0.6268049184840729, 0.07974541336580176},`.
  - **L454 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.030261789165931138, 0.6268049184840729, 0.07974541336580176},`。
- **L455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.030514080739125825, 0.6314869729283614, 0.06921734345076924},`.
  - **L455 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.030514080739125825, 0.6314869729283614, 0.06921734345076924},`。
- **L456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.030699652650745195, 0.6361616116842398, 0.05885329347452114},`.
  - **L456 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.030699652650745195, 0.6361616116842398, 0.05885329347452114},`。
- **L457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0308177148230897, 0.6408292087724975, 0.04877724313606999},`.
  - **L457 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0308177148230897, 0.6408292087724975, 0.04877724313606999},`。
- **L458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03086818674766468, 0.6454902160815501, 0.03915677764354433},`.
  - **L458 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03086818674766468, 0.6454902160815501, 0.03915677764354433},`。
- **L459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03197034466002908, 0.6501217563441514, 0.031444380194868886},`.
  - **L459 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03197034466002908, 0.6501217563441514, 0.031444380194868886},`。
- **L460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.046293996677654235, 0.6544720579771781, 0.031637291074855106},`.
  - **L460 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.046293996677654235, 0.6544720579771781, 0.031637291074855106},`。
- **L461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.058794558570863804, 0.6588271549701028, 0.0317521581310916},`.
  - **L461 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.058794558570863804, 0.6588271549701028, 0.0317521581310916},`。
- **L462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.06503863628200135, 0.6633055777623768, 0.03209873268943441},`.
  - **L462 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.06503863628200135, 0.6633055777623768, 0.03209873268943441},`。
- **L463 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07023149742711651, 0.6678120145394701, 0.03183433002082528},`.
  - **L463 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07023149742711651, 0.6678120145394701, 0.03183433002082528},`。
- **L464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0767138128414942, 0.6722807177648771, 0.032132076596637124},`.
  - **L464 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0767138128414942, 0.6722807177648771, 0.032132076596637124},`。
- **L465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.08432067120858369, 0.6767139870108101, 0.032338563602709626},`.
  - **L465 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.08432067120858369, 0.6767139870108101, 0.032338563602709626},`。
- **L466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.09294277780722546, 0.6811085964562938, 0.032444071987208487},`.
  - **L466 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.09294277780722546, 0.6811085964562938, 0.032444071987208487},`。
- **L467 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.10305233592409935, 0.6854378106573311, 0.03307401108271617},`.
  - **L467 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.10305233592409935, 0.6854378106573311, 0.03307401108271617},`。
- **L468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11331609119127764, 0.6897459817142692, 0.03295480891791886},`.
  - **L468 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11331609119127764, 0.6897459817142692, 0.03295480891791886},`。
- **L469 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12476614069574826, 0.6939830479703625, 0.033353619722170935},`.
  - **L469 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12476614069574826, 0.6939830479703625, 0.033353619722170935},`。
- **L470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13673711440841982, 0.6981699856866227, 0.03362590965526557},`.
  - **L470 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13673711440841982, 0.6981699856866227, 0.03362590965526557},`。
- **L471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1491636843492167, 0.7023042408994585, 0.033762275984369046},`.
  - **L471 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1491636843492167, 0.7023042408994585, 0.033762275984369046},`。
- **L472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.16199129317074465, 0.706383361101949, 0.03375336199700755},`.
  - **L472 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.16199129317074465, 0.706383361101949, 0.03375336199700755},`。
- **L473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.17553308151438088, 0.7103815097863063, 0.03426353710068628},`.
  - **L473 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.17553308151438088, 0.7103815097863063, 0.03426353710068628},`。
- **L474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.18933948484375165, 0.7143200103636559, 0.03462289684303036},`.
  - **L474 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.18933948484375165, 0.7143200103636559, 0.03462289684303036},`。
- **L475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.20338652270943824, 0.7181967154976012, 0.03482214714504671},`.
  - **L475 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.20338652270943824, 0.7181967154976012, 0.03482214714504671},`。
- **L476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.21765378276012448, 0.7220095660591789, 0.0348520554844732},`.
  - **L476 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.21765378276012448, 0.7220095660591789, 0.0348520554844732},`。

### Lines 477-504 / 第 477-504 行

````cpp
 477:     {0.23212369587552023, 0.7257565893785501, 0.03470346402383256},
 478:     {0.24702985279407214, 0.7294127796374531, 0.03507444712661609},
 479:     {0.2620775705935113, 0.7329996606625439, 0.035262260320886224},
 480:     {0.2772591134255436, 0.7365155484918555, 0.03525791814557077},
 481:     {0.2927712616124238, 0.7399360689986639, 0.035780536388425606},
 482:     {0.3083781576598969, 0.7432827601590504, 0.03610724156995657},
 483:     {0.3240763123743059, 0.746554228453955, 0.036229314118799705},
 484:     {0.33986207442019495, 0.7497491625691519, 0.03613819725600252},
 485:     {0.3558907708679886, 0.7528442021403842, 0.036581966518590206},
 486:     {0.3718311425153303, 0.7558826557986548, 0.036046421794990055},
 487:     {0.38813047246050614, 0.7587977057098403, 0.03681458696854892},
 488:     {0.4043371221939297, 0.7616541995826684, 0.03658736257615303},
 489:     {0.42072356025646707, 0.7644081013195043, 0.03690635392025313},
 490:     {0.43714739423907745, 0.7670803847862102, 0.03699298731846738},
 491:     {0.45360699836908186, 0.769670380372701, 0.03683987811183992},
 492:     {0.4702051248974935, 0.7721571183110766, 0.03724796925537228},
 493:     {0.4868227556665157, 0.7745610930844775, 0.0374172194555026},
 494:     {0.503458588002791, 0.7768819000929792, 0.03734086718256391},
 495:     {0.5201979969654729, 0.779099877137358, 0.03784362950139135},
 496:     {0.5368601535092656, 0.7812537776842732, 0.03726439057607977},
 497:     {0.5536883905115004, 0.7832866691953926, 0.0381141962950549},
 498:     {0.570436957512631, 0.7852552273306327, 0.03787015508826886},
 499:     {0.5872524511360537, 0.787122879084239, 0.03822846891183527},
 500:     {0.604057585524296, 0.7889082049024678, 0.03833696358570872},
 501:     {0.6208512632987039, 0.7906113236317394, 0.03819060762615894},
 502:     {0.6376323001265665, 0.7922323958682111, 0.03778469202199975},
 503:     {0.654448292617523, 0.7937561555975436, 0.03800948341735075},
 504:     {0.6712409304097, 0.7951995038068883, 0.03798214896623336},
````
- **L477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.23212369587552023, 0.7257565893785501, 0.03470346402383256},`.
  - **L477 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.23212369587552023, 0.7257565893785501, 0.03470346402383256},`。
- **L478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.24702985279407214, 0.7294127796374531, 0.03507444712661609},`.
  - **L478 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.24702985279407214, 0.7294127796374531, 0.03507444712661609},`。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2620775705935113, 0.7329996606625439, 0.035262260320886224},`.
  - **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2620775705935113, 0.7329996606625439, 0.035262260320886224},`。
- **L480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2772591134255436, 0.7365155484918555, 0.03525791814557077},`.
  - **L480 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2772591134255436, 0.7365155484918555, 0.03525791814557077},`。
- **L481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2927712616124238, 0.7399360689986639, 0.035780536388425606},`.
  - **L481 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2927712616124238, 0.7399360689986639, 0.035780536388425606},`。
- **L482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3083781576598969, 0.7432827601590504, 0.03610724156995657},`.
  - **L482 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3083781576598969, 0.7432827601590504, 0.03610724156995657},`。
- **L483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3240763123743059, 0.746554228453955, 0.036229314118799705},`.
  - **L483 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3240763123743059, 0.746554228453955, 0.036229314118799705},`。
- **L484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.33986207442019495, 0.7497491625691519, 0.03613819725600252},`.
  - **L484 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.33986207442019495, 0.7497491625691519, 0.03613819725600252},`。
- **L485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3558907708679886, 0.7528442021403842, 0.036581966518590206},`.
  - **L485 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3558907708679886, 0.7528442021403842, 0.036581966518590206},`。
- **L486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3718311425153303, 0.7558826557986548, 0.036046421794990055},`.
  - **L486 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3718311425153303, 0.7558826557986548, 0.036046421794990055},`。
- **L487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.38813047246050614, 0.7587977057098403, 0.03681458696854892},`.
  - **L487 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.38813047246050614, 0.7587977057098403, 0.03681458696854892},`。
- **L488 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4043371221939297, 0.7616541995826684, 0.03658736257615303},`.
  - **L488 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4043371221939297, 0.7616541995826684, 0.03658736257615303},`。
- **L489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.42072356025646707, 0.7644081013195043, 0.03690635392025313},`.
  - **L489 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.42072356025646707, 0.7644081013195043, 0.03690635392025313},`。
- **L490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.43714739423907745, 0.7670803847862102, 0.03699298731846738},`.
  - **L490 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.43714739423907745, 0.7670803847862102, 0.03699298731846738},`。
- **L491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.45360699836908186, 0.769670380372701, 0.03683987811183992},`.
  - **L491 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.45360699836908186, 0.769670380372701, 0.03683987811183992},`。
- **L492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4702051248974935, 0.7721571183110766, 0.03724796925537228},`.
  - **L492 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4702051248974935, 0.7721571183110766, 0.03724796925537228},`。
- **L493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4868227556665157, 0.7745610930844775, 0.0374172194555026},`.
  - **L493 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4868227556665157, 0.7745610930844775, 0.0374172194555026},`。
- **L494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.503458588002791, 0.7768819000929792, 0.03734086718256391},`.
  - **L494 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.503458588002791, 0.7768819000929792, 0.03734086718256391},`。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5201979969654729, 0.779099877137358, 0.03784362950139135},`.
  - **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5201979969654729, 0.779099877137358, 0.03784362950139135},`。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5368601535092656, 0.7812537776842732, 0.03726439057607977},`.
  - **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5368601535092656, 0.7812537776842732, 0.03726439057607977},`。
- **L497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5536883905115004, 0.7832866691953926, 0.0381141962950549},`.
  - **L497 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5536883905115004, 0.7832866691953926, 0.0381141962950549},`。
- **L498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.570436957512631, 0.7852552273306327, 0.03787015508826886},`.
  - **L498 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.570436957512631, 0.7852552273306327, 0.03787015508826886},`。
- **L499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5872524511360537, 0.787122879084239, 0.03822846891183527},`.
  - **L499 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5872524511360537, 0.787122879084239, 0.03822846891183527},`。
- **L500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.604057585524296, 0.7889082049024678, 0.03833696358570872},`.
  - **L500 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.604057585524296, 0.7889082049024678, 0.03833696358570872},`。
- **L501 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6208512632987039, 0.7906113236317394, 0.03819060762615894},`.
  - **L501 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6208512632987039, 0.7906113236317394, 0.03819060762615894},`。
- **L502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6376323001265665, 0.7922323958682111, 0.03778469202199975},`.
  - **L502 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6376323001265665, 0.7922323958682111, 0.03778469202199975},`。
- **L503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.654448292617523, 0.7937561555975436, 0.03800948341735075},`.
  - **L503 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.654448292617523, 0.7937561555975436, 0.03800948341735075},`。
- **L504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6712409304097, 0.7951995038068883, 0.03798214896623336},`.
  - **L504 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6712409304097, 0.7951995038068883, 0.03798214896623336},`。

### Lines 505-532 / 第 505-532 行

````cpp
 505:     {0.6880499867367655, 0.7965485565944844, 0.038610111469974305},
 506:     {0.7047891048851465, 0.7978328175913231, 0.03807498680487568},
 507:     {0.7215350356575515, 0.7990249426739793, 0.03820456984010566},
 508:     {0.7382756146491691, 0.8001275152547487, 0.03902092612392253},
 509:     {0.7549478024082115, 0.8011661514812047, 0.03865607007534288},
 510:     {0.7716059891201252, 0.8021178857956822, 0.038989380941084874},
 511:     {0.7882203132421456, 0.8029955440735611, 0.03908165535934637},
 512:     {0.8047902362352904, 0.8037997989252117, 0.03893065650816213},
 513:     {0.8215523268463777, 0.8044555316790379, 0.03926557049926157},
 514:     {0.8387398480023723, 0.8048817689898747, 0.04067065212316836},
 515:     {0.856341544941598, 0.8050796719349672, 0.04123085284140562},
 516:     {0.8743768083538662, 0.8050267567672601, 0.0419261583900458},
 517:     {0.8928528884736874, 0.8047076768017004, 0.0427644722591047},
 518:     {0.911776412947498, 0.8041062177363495, 0.04375041022498651},
 519:     {0.9311533128855841, 0.8032052518345966, 0.044885135667359986},
 520:     {0.9510015669358057, 0.8019861521093661, 0.04532603899543026},
 521:     {0.962930103481011, 0.8014439331866959, 0.22169249312004036},
 522:     {0.9683410354743099, 0.8024669332735311, 0.3358446858430373},
 523:     {0.9719380800534977, 0.8044065772211566, 0.4114102805053895},
 524:     {0.9746751561332506, 0.8069447701330453, 0.46819081680773716},
 525:     {0.9768136632655356, 0.8099358552525316, 0.5140546510678895},
 526:     {0.978649037790159, 0.8132574705132256, 0.5522569503355605},
 527:     {0.9802036704451057, 0.8168594951941832, 0.5851451968636262},
 528:     {0.981554091047551, 0.8206924722262127, 0.6139795814385243},
 529:     {0.9827732625825781, 0.8247142837530934, 0.6395746026568098},
 530:     {0.9839157453834834, 0.8288914274285599, 0.6625186328985215},
 531:     {0.9848784092487323, 0.8332318824714189, 0.6834911738162105},
 532:     {0.9858320115191356, 0.8376808147738173, 0.7025706724123436},
````
- **L505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6880499867367655, 0.7965485565944844, 0.038610111469974305},`.
  - **L505 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6880499867367655, 0.7965485565944844, 0.038610111469974305},`。
- **L506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7047891048851465, 0.7978328175913231, 0.03807498680487568},`.
  - **L506 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7047891048851465, 0.7978328175913231, 0.03807498680487568},`。
- **L507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7215350356575515, 0.7990249426739793, 0.03820456984010566},`.
  - **L507 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7215350356575515, 0.7990249426739793, 0.03820456984010566},`。
- **L508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7382756146491691, 0.8001275152547487, 0.03902092612392253},`.
  - **L508 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7382756146491691, 0.8001275152547487, 0.03902092612392253},`。
- **L509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7549478024082115, 0.8011661514812047, 0.03865607007534288},`.
  - **L509 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7549478024082115, 0.8011661514812047, 0.03865607007534288},`。
- **L510 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7716059891201252, 0.8021178857956822, 0.038989380941084874},`.
  - **L510 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7716059891201252, 0.8021178857956822, 0.038989380941084874},`。
- **L511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7882203132421456, 0.8029955440735611, 0.03908165535934637},`.
  - **L511 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7882203132421456, 0.8029955440735611, 0.03908165535934637},`。
- **L512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8047902362352904, 0.8037997989252117, 0.03893065650816213},`.
  - **L512 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8047902362352904, 0.8037997989252117, 0.03893065650816213},`。
- **L513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8215523268463777, 0.8044555316790379, 0.03926557049926157},`.
  - **L513 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8215523268463777, 0.8044555316790379, 0.03926557049926157},`。
- **L514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8387398480023723, 0.8048817689898747, 0.04067065212316836},`.
  - **L514 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8387398480023723, 0.8048817689898747, 0.04067065212316836},`。
- **L515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.856341544941598, 0.8050796719349672, 0.04123085284140562},`.
  - **L515 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.856341544941598, 0.8050796719349672, 0.04123085284140562},`。
- **L516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8743768083538662, 0.8050267567672601, 0.0419261583900458},`.
  - **L516 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8743768083538662, 0.8050267567672601, 0.0419261583900458},`。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8928528884736874, 0.8047076768017004, 0.0427644722591047},`.
  - **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8928528884736874, 0.8047076768017004, 0.0427644722591047},`。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.911776412947498, 0.8041062177363495, 0.04375041022498651},`.
  - **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.911776412947498, 0.8041062177363495, 0.04375041022498651},`。
- **L519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9311533128855841, 0.8032052518345966, 0.044885135667359986},`.
  - **L519 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9311533128855841, 0.8032052518345966, 0.044885135667359986},`。
- **L520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9510015669358057, 0.8019861521093661, 0.04532603899543026},`.
  - **L520 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9510015669358057, 0.8019861521093661, 0.04532603899543026},`。
- **L521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.962930103481011, 0.8014439331866959, 0.22169249312004036},`.
  - **L521 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.962930103481011, 0.8014439331866959, 0.22169249312004036},`。
- **L522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9683410354743099, 0.8024669332735311, 0.3358446858430373},`.
  - **L522 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9683410354743099, 0.8024669332735311, 0.3358446858430373},`。
- **L523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9719380800534977, 0.8044065772211566, 0.4114102805053895},`.
  - **L523 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9719380800534977, 0.8044065772211566, 0.4114102805053895},`。
- **L524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9746751561332506, 0.8069447701330453, 0.46819081680773716},`.
  - **L524 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9746751561332506, 0.8069447701330453, 0.46819081680773716},`。
- **L525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9768136632655356, 0.8099358552525316, 0.5140546510678895},`.
  - **L525 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9768136632655356, 0.8099358552525316, 0.5140546510678895},`。
- **L526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.978649037790159, 0.8132574705132256, 0.5522569503355605},`.
  - **L526 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.978649037790159, 0.8132574705132256, 0.5522569503355605},`。
- **L527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9802036704451057, 0.8168594951941832, 0.5851451968636262},`.
  - **L527 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9802036704451057, 0.8168594951941832, 0.5851451968636262},`。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.981554091047551, 0.8206924722262127, 0.6139795814385243},`.
  - **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.981554091047551, 0.8206924722262127, 0.6139795814385243},`。
- **L529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9827732625825781, 0.8247142837530934, 0.6395746026568098},`.
  - **L529 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9827732625825781, 0.8247142837530934, 0.6395746026568098},`。
- **L530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9839157453834834, 0.8288914274285599, 0.6625186328985215},`.
  - **L530 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9839157453834834, 0.8288914274285599, 0.6625186328985215},`。
- **L531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9848784092487323, 0.8332318824714189, 0.6834911738162105},`.
  - **L531 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9848784092487323, 0.8332318824714189, 0.6834911738162105},`。
- **L532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9858320115191356, 0.8376808147738173, 0.7025706724123436},`.
  - **L532 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9858320115191356, 0.8376808147738173, 0.7025706724123436},`。

### Lines 533-560 / 第 533-560 行

````cpp
 533:     {0.9866558657916994, 0.8422559436774792, 0.7202399867839039},
 534:     {0.9873677995679001, 0.8469433249862409, 0.7366844886107697},
 535:     {0.9881805379141889, 0.8516804941618812, 0.7518089864773918},
 536:     {0.9887611047789485, 0.8565442781873747, 0.7661909960366566},
 537:     {0.9894043443414985, 0.8614528824718757, 0.7795879261784779},
 538:     {0.9900758649057856, 0.8664087680409848, 0.7921604870664645},
 539:     {0.99059092673338, 0.8714545088111433, 0.8041942791071222},
 540:     {0.9911277189421445, 0.8765391108526217, 0.8155756474332213},
 541:     {0.9916061462381213, 0.8816790723671002, 0.8264519900822266},
 542:     {0.9922158985563451, 0.8868207617131697, 0.836709699896381},
 543:     {0.9925679440918894, 0.8920625188742286, 0.8467473851492474},
 544:     {0.9931424279738416, 0.8972751802406753, 0.8561933859696547},
 545:     {0.9935217736048007, 0.9025652348682511, 0.8654402820837326},
 546:     {0.9938879084327678, 0.9078821573987383, 0.874370338308405},
 547:     {0.99427342946785, 0.9132148454645382, 0.8829927351962013},
 548:     {0.9947111506654817, 0.918552416162813, 0.8913160852881722},
 549:     {0.995056938233056, 0.9239302650640105, 0.8994722486725081},
 550:     {0.995512287532556, 0.9292939740928252, 0.9073439419102849},
 551:     {0.9959250022023757, 0.9346812618630085, 0.9150611281410694},
 552:     {0.9961274042213023, 0.9401333891345969, 0.9227518397011023},
 553:     {0.9965143545143408, 0.9455470167979051, 0.9301752047442942},
 554:     {0.996922153842586, 0.9509629844614244, 0.9374587708548032},
 555:     {0.9971731694726661, 0.9564250004713418, 0.9447286698401807},
 556:     {0.9976825719783714, 0.9618265695993463, 0.9517442599956717},
 557:     {0.9978661370132459, 0.967313947090583, 0.958874650466398},
 558:     {0.9983489942212443, 0.972728972574113, 0.9657569772496776},
 559:     {0.9985297770627254, 0.9782208004653724, 0.9727591013871136},
 560:     {0.9988366714262512, 0.9836818403108719, 0.9796396969161855},
````
- **L533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9866558657916994, 0.8422559436774792, 0.7202399867839039},`.
  - **L533 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9866558657916994, 0.8422559436774792, 0.7202399867839039},`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9873677995679001, 0.8469433249862409, 0.7366844886107697},`.
  - **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9873677995679001, 0.8469433249862409, 0.7366844886107697},`。
- **L535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9881805379141889, 0.8516804941618812, 0.7518089864773918},`.
  - **L535 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9881805379141889, 0.8516804941618812, 0.7518089864773918},`。
- **L536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9887611047789485, 0.8565442781873747, 0.7661909960366566},`.
  - **L536 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9887611047789485, 0.8565442781873747, 0.7661909960366566},`。
- **L537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9894043443414985, 0.8614528824718757, 0.7795879261784779},`.
  - **L537 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9894043443414985, 0.8614528824718757, 0.7795879261784779},`。
- **L538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9900758649057856, 0.8664087680409848, 0.7921604870664645},`.
  - **L538 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9900758649057856, 0.8664087680409848, 0.7921604870664645},`。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.99059092673338, 0.8714545088111433, 0.8041942791071222},`.
  - **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.99059092673338, 0.8714545088111433, 0.8041942791071222},`。
- **L540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9911277189421445, 0.8765391108526217, 0.8155756474332213},`.
  - **L540 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9911277189421445, 0.8765391108526217, 0.8155756474332213},`。
- **L541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9916061462381213, 0.8816790723671002, 0.8264519900822266},`.
  - **L541 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9916061462381213, 0.8816790723671002, 0.8264519900822266},`。
- **L542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9922158985563451, 0.8868207617131697, 0.836709699896381},`.
  - **L542 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9922158985563451, 0.8868207617131697, 0.836709699896381},`。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9925679440918894, 0.8920625188742286, 0.8467473851492474},`.
  - **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9925679440918894, 0.8920625188742286, 0.8467473851492474},`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9931424279738416, 0.8972751802406753, 0.8561933859696547},`.
  - **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9931424279738416, 0.8972751802406753, 0.8561933859696547},`。
- **L545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9935217736048007, 0.9025652348682511, 0.8654402820837326},`.
  - **L545 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9935217736048007, 0.9025652348682511, 0.8654402820837326},`。
- **L546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9938879084327678, 0.9078821573987383, 0.874370338308405},`.
  - **L546 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9938879084327678, 0.9078821573987383, 0.874370338308405},`。
- **L547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.99427342946785, 0.9132148454645382, 0.8829927351962013},`.
  - **L547 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.99427342946785, 0.9132148454645382, 0.8829927351962013},`。
- **L548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9947111506654817, 0.918552416162813, 0.8913160852881722},`.
  - **L548 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9947111506654817, 0.918552416162813, 0.8913160852881722},`。
- **L549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.995056938233056, 0.9239302650640105, 0.8994722486725081},`.
  - **L549 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.995056938233056, 0.9239302650640105, 0.8994722486725081},`。
- **L550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.995512287532556, 0.9292939740928252, 0.9073439419102849},`.
  - **L550 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.995512287532556, 0.9292939740928252, 0.9073439419102849},`。
- **L551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9959250022023757, 0.9346812618630085, 0.9150611281410694},`.
  - **L551 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9959250022023757, 0.9346812618630085, 0.9150611281410694},`。
- **L552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9961274042213023, 0.9401333891345969, 0.9227518397011023},`.
  - **L552 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9961274042213023, 0.9401333891345969, 0.9227518397011023},`。
- **L553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9965143545143408, 0.9455470167979051, 0.9301752047442942},`.
  - **L553 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9965143545143408, 0.9455470167979051, 0.9301752047442942},`。
- **L554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.996922153842586, 0.9509629844614244, 0.9374587708548032},`.
  - **L554 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.996922153842586, 0.9509629844614244, 0.9374587708548032},`。
- **L555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9971731694726661, 0.9564250004713418, 0.9447286698401807},`.
  - **L555 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9971731694726661, 0.9564250004713418, 0.9447286698401807},`。
- **L556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9976825719783714, 0.9618265695993463, 0.9517442599956717},`.
  - **L556 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9976825719783714, 0.9618265695993463, 0.9517442599956717},`。
- **L557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9978661370132459, 0.967313947090583, 0.958874650466398},`.
  - **L557 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9978661370132459, 0.967313947090583, 0.958874650466398},`。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9983489942212443, 0.972728972574113, 0.9657569772496776},`.
  - **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9983489942212443, 0.972728972574113, 0.9657569772496776},`。
- **L559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9985297770627254, 0.9782208004653724, 0.9727591013871136},`.
  - **L559 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9985297770627254, 0.9782208004653724, 0.9727591013871136},`。
- **L560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9988366714262512, 0.9836818403108719, 0.9796396969161855},`.
  - **L560 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9988366714262512, 0.9836818403108719, 0.9796396969161855},`。

### Lines 561-588 / 第 561-588 行

````cpp
 561:     {0.999287652981863, 0.989107488582374, 0.9864006969015998},
 562:     {0.9994697671436109, 0.9945977204425903, 0.9932874563643866},
 563:     {1.0, 1.0, 1.0},
 564: }};
 565: 
 566: template <typename Real>
 567: static constexpr std::array<std::array<Real, 3>, 256> inferno_data_ = {{
 568:     {0.0014619955811715805, 0.0004659913919114934, 0.013866005775115809},
 569:     {0.0022669056023600243, 0.001269897101615975, 0.018569490325902337},
 570:     {0.003299036110031063, 0.0022490183451722313, 0.024239243465136288},
 571:     {0.004546896852350439, 0.0033918841632656804, 0.03090851258977682},
 572:     {0.006006105056993791, 0.004692061241092744, 0.038558624443389096},
 573:     {0.007675918804434457, 0.006135891503856028, 0.046835705273408614},
 574:     {0.009561203394731096, 0.0077131160510677, 0.05514393874236257},
 575:     {0.011662968995142865, 0.0094169153553472, 0.06345998081431048},
 576:     {0.013994707785115502, 0.011224701548363442, 0.07186110007643282},
 577:     {0.01656105637139426, 0.01313595377716254, 0.08028228905271992},
 578:     {0.019372742788596516, 0.01513271743178829, 0.08876637381708319},
 579:     {0.02244719411002579, 0.017198996785303594, 0.09732759285010725},
 580:     {0.02579282311826555, 0.01933074318511054, 0.10592963918696552},
 581:     {0.029432387489373633, 0.02150303695278668, 0.11462190156591848},
 582:     {0.03338491092042817, 0.023701776954208033, 0.12339694246694158},
 583:     {0.03766747608151851, 0.025920506649278047, 0.13223096637532075},
 584:     {0.042253067017337005, 0.028138807463796305, 0.1411412582171954},
 585:     {0.04691458399133113, 0.030323540520811973, 0.15016326468244964},
 586:     {0.05164425209311529, 0.03247382729611796, 0.15925458396315},
 587:     {0.05644871419547314, 0.03456857075460381, 0.16841357852522199},
 588:     {0.06134044494312783, 0.03658982977294056, 0.17764292167289156},
````
- **L561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.999287652981863, 0.989107488582374, 0.9864006969015998},`.
  - **L561 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.999287652981863, 0.989107488582374, 0.9864006969015998},`。
- **L562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9994697671436109, 0.9945977204425903, 0.9932874563643866},`.
  - **L562 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9994697671436109, 0.9945977204425903, 0.9932874563643866},`。
- **L563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{1.0, 1.0, 1.0},`.
  - **L563 CN**: 继续一个多行参数列表、初始化器或聚合项：`{1.0, 1.0, 1.0},`。
- **L564 EN**: Executes a standalone statement or declaration: `}};`.
  - **L564 CN**: 执行一条独立语句或声明：`}};`。
- **L565 EN**: Blank line separating nearby declarations or logic.
  - **L565 CN**: 空行，用于分隔相邻声明或逻辑。
- **L566 EN**: Introduces template parameters or specialization context: `template <typename Real>`.
  - **L566 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real>`。
- **L567 EN**: Continues the surrounding expression or declaration: `static constexpr std::array<std::array<Real, 3>, 256> inferno_data_ = {{`.
  - **L567 CN**: 继续构造周围的表达式或声明：`static constexpr std::array<std::array<Real, 3>, 256> inferno_data_ = {{`。
- **L568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0014619955811715805, 0.0004659913919114934, 0.013866005775115809},`.
  - **L568 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0014619955811715805, 0.0004659913919114934, 0.013866005775115809},`。
- **L569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0022669056023600243, 0.001269897101615975, 0.018569490325902337},`.
  - **L569 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0022669056023600243, 0.001269897101615975, 0.018569490325902337},`。
- **L570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.003299036110031063, 0.0022490183451722313, 0.024239243465136288},`.
  - **L570 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.003299036110031063, 0.0022490183451722313, 0.024239243465136288},`。
- **L571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.004546896852350439, 0.0033918841632656804, 0.03090851258977682},`.
  - **L571 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.004546896852350439, 0.0033918841632656804, 0.03090851258977682},`。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.006006105056993791, 0.004692061241092744, 0.038558624443389096},`.
  - **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.006006105056993791, 0.004692061241092744, 0.038558624443389096},`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.007675918804434457, 0.006135891503856028, 0.046835705273408614},`.
  - **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.007675918804434457, 0.006135891503856028, 0.046835705273408614},`。
- **L574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.009561203394731096, 0.0077131160510677, 0.05514393874236257},`.
  - **L574 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.009561203394731096, 0.0077131160510677, 0.05514393874236257},`。
- **L575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.011662968995142865, 0.0094169153553472, 0.06345998081431048},`.
  - **L575 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.011662968995142865, 0.0094169153553472, 0.06345998081431048},`。
- **L576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.013994707785115502, 0.011224701548363442, 0.07186110007643282},`.
  - **L576 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.013994707785115502, 0.011224701548363442, 0.07186110007643282},`。
- **L577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.01656105637139426, 0.01313595377716254, 0.08028228905271992},`.
  - **L577 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.01656105637139426, 0.01313595377716254, 0.08028228905271992},`。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.019372742788596516, 0.01513271743178829, 0.08876637381708319},`.
  - **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.019372742788596516, 0.01513271743178829, 0.08876637381708319},`。
- **L579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02244719411002579, 0.017198996785303594, 0.09732759285010725},`.
  - **L579 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02244719411002579, 0.017198996785303594, 0.09732759285010725},`。
- **L580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02579282311826555, 0.01933074318511054, 0.10592963918696552},`.
  - **L580 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02579282311826555, 0.01933074318511054, 0.10592963918696552},`。
- **L581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.029432387489373633, 0.02150303695278668, 0.11462190156591848},`.
  - **L581 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.029432387489373633, 0.02150303695278668, 0.11462190156591848},`。
- **L582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03338491092042817, 0.023701776954208033, 0.12339694246694158},`.
  - **L582 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03338491092042817, 0.023701776954208033, 0.12339694246694158},`。
- **L583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03766747608151851, 0.025920506649278047, 0.13223096637532075},`.
  - **L583 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03766747608151851, 0.025920506649278047, 0.13223096637532075},`。
- **L584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.042253067017337005, 0.028138807463796305, 0.1411412582171954},`.
  - **L584 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.042253067017337005, 0.028138807463796305, 0.1411412582171954},`。
- **L585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.04691458399133113, 0.030323540520811973, 0.15016326468244964},`.
  - **L585 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.04691458399133113, 0.030323540520811973, 0.15016326468244964},`。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.05164425209311529, 0.03247382729611796, 0.15925458396315},`.
  - **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.05164425209311529, 0.03247382729611796, 0.15925458396315},`。
- **L587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.05644871419547314, 0.03456857075460381, 0.16841357852522199},`.
  - **L587 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.05644871419547314, 0.03456857075460381, 0.16841357852522199},`。
- **L588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.06134044494312783, 0.03658982977294056, 0.17764292167289156},`.
  - **L588 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.06134044494312783, 0.03658982977294056, 0.17764292167289156},`。

### Lines 589-616 / 第 589-616 行

````cpp
 589:     {0.06633085137688166, 0.038503594004715896, 0.18696190141292804},
 590:     {0.07142826458096291, 0.04029339894981024, 0.19635286429314014},
 591:     {0.07663702254956387, 0.04190461115902603, 0.2057992341013456},
 592:     {0.08196142528164385, 0.043327451450508585, 0.2152881892106146},
 593:     {0.087411214696131, 0.04455560411873583, 0.22481357132253754},
 594:     {0.09298959953890884, 0.04558246689176038, 0.23435752319694075},
 595:     {0.0987024276165075, 0.0464015612109181, 0.24390490595057124},
 596:     {0.1045507907963224, 0.04700744862572622, 0.2534298623464375},
 597:     {0.11053667232221573, 0.04739845362551404, 0.26291322780905574},
 598:     {0.11665600122383982, 0.04757339371302694, 0.27232119603015154},
 599:     {0.12290731198945251, 0.04753536748517837, 0.2816231724086292},
 600:     {0.12928523040832837, 0.04729230240096718, 0.2907885083783503},
 601:     {0.13577751583663375, 0.046855310502846824, 0.29977552118778034},
 602:     {0.14237847430795506, 0.04624117675574093, 0.30855378680836465},
 603:     {0.1490727384829781, 0.04546721023082908, 0.3170848511067041},
 604:     {0.1558507266503708, 0.044558026492337664, 0.32533901695859424},
 605:     {0.1626889760379725, 0.0435530737190721, 0.3332771435961689},
 606:     {0.16957421944883505, 0.042488139902889556, 0.3408733080618529},
 607:     {0.17649322158660016, 0.04140091975980204, 0.3481113785987225},
 608:     {0.18342846019226636, 0.04032795978956832, 0.35497062509269495},
 609:     {0.19036746731312684, 0.03930776663057157, 0.36144755914871207},
 610:     {0.19729670678394517, 0.03839881405885271, 0.3675348910966813},
 611:     {0.20420970925948218, 0.03763066869776516, 0.3732386865741054},
 612:     {0.21109495319112997, 0.0370286796745015, 0.3785631007654439},
 613:     {0.21794820252834113, 0.036613674395490264, 0.38352155568816976},
 614:     {0.22476319518924603, 0.03640357374688182, 0.38812924842856483},
 615:     {0.2315374529310898, 0.03640351817286241, 0.39239977934534576},
 616:     {0.2382734320696066, 0.03661949264330557, 0.3963533531157254},
````
- **L589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.06633085137688166, 0.038503594004715896, 0.18696190141292804},`.
  - **L589 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.06633085137688166, 0.038503594004715896, 0.18696190141292804},`。
- **L590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07142826458096291, 0.04029339894981024, 0.19635286429314014},`.
  - **L590 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07142826458096291, 0.04029339894981024, 0.19635286429314014},`。
- **L591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07663702254956387, 0.04190461115902603, 0.2057992341013456},`.
  - **L591 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07663702254956387, 0.04190461115902603, 0.2057992341013456},`。
- **L592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.08196142528164385, 0.043327451450508585, 0.2152881892106146},`.
  - **L592 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.08196142528164385, 0.043327451450508585, 0.2152881892106146},`。
- **L593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.087411214696131, 0.04455560411873583, 0.22481357132253754},`.
  - **L593 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.087411214696131, 0.04455560411873583, 0.22481357132253754},`。
- **L594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.09298959953890884, 0.04558246689176038, 0.23435752319694075},`.
  - **L594 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.09298959953890884, 0.04558246689176038, 0.23435752319694075},`。
- **L595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0987024276165075, 0.0464015612109181, 0.24390490595057124},`.
  - **L595 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0987024276165075, 0.0464015612109181, 0.24390490595057124},`。
- **L596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1045507907963224, 0.04700744862572622, 0.2534298623464375},`.
  - **L596 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1045507907963224, 0.04700744862572622, 0.2534298623464375},`。
- **L597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11053667232221573, 0.04739845362551404, 0.26291322780905574},`.
  - **L597 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11053667232221573, 0.04739845362551404, 0.26291322780905574},`。
- **L598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11665600122383982, 0.04757339371302694, 0.27232119603015154},`.
  - **L598 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11665600122383982, 0.04757339371302694, 0.27232119603015154},`。
- **L599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12290731198945251, 0.04753536748517837, 0.2816231724086292},`.
  - **L599 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12290731198945251, 0.04753536748517837, 0.2816231724086292},`。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12928523040832837, 0.04729230240096718, 0.2907885083783503},`.
  - **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12928523040832837, 0.04729230240096718, 0.2907885083783503},`。
- **L601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13577751583663375, 0.046855310502846824, 0.29977552118778034},`.
  - **L601 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13577751583663375, 0.046855310502846824, 0.29977552118778034},`。
- **L602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14237847430795506, 0.04624117675574093, 0.30855378680836465},`.
  - **L602 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14237847430795506, 0.04624117675574093, 0.30855378680836465},`。
- **L603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1490727384829781, 0.04546721023082908, 0.3170848511067041},`.
  - **L603 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1490727384829781, 0.04546721023082908, 0.3170848511067041},`。
- **L604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1558507266503708, 0.044558026492337664, 0.32533901695859424},`.
  - **L604 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1558507266503708, 0.044558026492337664, 0.32533901695859424},`。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1626889760379725, 0.0435530737190721, 0.3332771435961689},`.
  - **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1626889760379725, 0.0435530737190721, 0.3332771435961689},`。
- **L606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.16957421944883505, 0.042488139902889556, 0.3408733080618529},`.
  - **L606 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.16957421944883505, 0.042488139902889556, 0.3408733080618529},`。
- **L607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.17649322158660016, 0.04140091975980204, 0.3481113785987225},`.
  - **L607 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.17649322158660016, 0.04140091975980204, 0.3481113785987225},`。
- **L608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.18342846019226636, 0.04032795978956832, 0.35497062509269495},`.
  - **L608 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.18342846019226636, 0.04032795978956832, 0.35497062509269495},`。
- **L609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.19036746731312684, 0.03930776663057157, 0.36144755914871207},`.
  - **L609 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.19036746731312684, 0.03930776663057157, 0.36144755914871207},`。
- **L610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.19729670678394517, 0.03839881405885271, 0.3675348910966813},`.
  - **L610 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.19729670678394517, 0.03839881405885271, 0.3675348910966813},`。
- **L611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.20420970925948218, 0.03763066869776516, 0.3732386865741054},`.
  - **L611 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.20420970925948218, 0.03763066869776516, 0.3732386865741054},`。
- **L612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.21109495319112997, 0.0370286796745015, 0.3785631007654439},`.
  - **L612 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.21109495319112997, 0.0370286796745015, 0.3785631007654439},`。
- **L613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.21794820252834113, 0.036613674395490264, 0.38352155568816976},`.
  - **L613 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.21794820252834113, 0.036613674395490264, 0.38352155568816976},`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.22476319518924603, 0.03640357374688182, 0.38812924842856483},`.
  - **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.22476319518924603, 0.03640357374688182, 0.38812924842856483},`。
- **L615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2315374529310898, 0.03640351817286241, 0.39239977934534576},`.
  - **L615 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2315374529310898, 0.03640351817286241, 0.39239977934534576},`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2382734320696066, 0.03661949264330557, 0.3963533531157254},`.
  - **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2382734320696066, 0.03661949264330557, 0.3963533531157254},`。

### Lines 617-644 / 第 617-644 行

````cpp
 617:     {0.2449666980359304, 0.03705338654185691, 0.4000069520159531},
 618:     {0.2516206637792285, 0.037703436097523746, 0.4033784221656792},
 619:     {0.2582339375612672, 0.038569281262784215, 0.4064850812186898},
 620:     {0.2648092188471272, 0.039645098963587894, 0.4093447670552162},
 621:     {0.27134717156688476, 0.040920248896872104, 0.41197616680691795},
 622:     {0.27784945989435444, 0.04235106621674233, 0.41439190152253697},
 623:     {0.2843214014200332, 0.043931272334442044, 0.4166082251959268},
 624:     {0.2907626953161, 0.04564206683863344, 0.41863700244674007},
 625:     {0.2971786279866803, 0.04746832221255668, 0.42049126153840694},
 626:     {0.3035679263767977, 0.04939409900213965, 0.42218207588627776},
 627:     {0.3099342310464275, 0.05140486616668636, 0.4237209077370659},
 628:     {0.3162821544923382, 0.0534881537223529, 0.42511612281358585},
 629:     {0.3226094622720638, 0.05563191003781664, 0.42637698442915734},
 630:     {0.32892138033063256, 0.057825220219243655, 0.4275111514018109},
 631:     {0.33521669037969953, 0.060057968826412046, 0.42852404020939816},
 632:     {0.34150060454122194, 0.06232329248965957, 0.42942516526482805},
 633:     {0.34777091610802874, 0.06461403589398387, 0.43021707827304073},
 634:     {0.35403123226470046, 0.06692277585963113, 0.43090600382653765},
 635:     {0.3602841408353374, 0.06924510666507289, 0.4314970989990494},
 636:     {0.366528457743285, 0.07157684449494817, 0.4319940445656597},
 637:     {0.3727683648145053, 0.0739131776252345, 0.4324001060667602},
 638:     {0.37900068183696356, 0.07625091481761018, 0.432719071099916},
 639:     {0.3852285882170877, 0.07858924726823341, 0.43295510186497294},
 640:     {0.391452904966983, 0.08092498458487726, 0.4331090849213901},
 641:     {0.3976732251941389, 0.08325472094735673, 0.43318307823977154},
 642:     {0.4038941281371945, 0.08557805215845263, 0.4331790858011016},
 643:     {0.4101124478324236, 0.08789378949993748, 0.43309809489281975},
 644:     {0.41633135152767126, 0.09020111672608602, 0.4329430750225925},
````
- **L617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2449666980359304, 0.03705338654185691, 0.4000069520159531},`.
  - **L617 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2449666980359304, 0.03705338654185691, 0.4000069520159531},`。
- **L618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2516206637792285, 0.037703436097523746, 0.4033784221656792},`.
  - **L618 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2516206637792285, 0.037703436097523746, 0.4033784221656792},`。
- **L619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2582339375612672, 0.038569281262784215, 0.4064850812186898},`.
  - **L619 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2582339375612672, 0.038569281262784215, 0.4064850812186898},`。
- **L620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2648092188471272, 0.039645098963587894, 0.4093447670552162},`.
  - **L620 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2648092188471272, 0.039645098963587894, 0.4093447670552162},`。
- **L621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27134717156688476, 0.040920248896872104, 0.41197616680691795},`.
  - **L621 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27134717156688476, 0.040920248896872104, 0.41197616680691795},`。
- **L622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27784945989435444, 0.04235106621674233, 0.41439190152253697},`.
  - **L622 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27784945989435444, 0.04235106621674233, 0.41439190152253697},`。
- **L623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2843214014200332, 0.043931272334442044, 0.4166082251959268},`.
  - **L623 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2843214014200332, 0.043931272334442044, 0.4166082251959268},`。
- **L624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2907626953161, 0.04564206683863344, 0.41863700244674007},`.
  - **L624 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2907626953161, 0.04564206683863344, 0.41863700244674007},`。
- **L625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2971786279866803, 0.04746832221255668, 0.42049126153840694},`.
  - **L625 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2971786279866803, 0.04746832221255668, 0.42049126153840694},`。
- **L626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3035679263767977, 0.04939409900213965, 0.42218207588627776},`.
  - **L626 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3035679263767977, 0.04939409900213965, 0.42218207588627776},`。
- **L627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3099342310464275, 0.05140486616668636, 0.4237209077370659},`.
  - **L627 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3099342310464275, 0.05140486616668636, 0.4237209077370659},`。
- **L628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3162821544923382, 0.0534881537223529, 0.42511612281358585},`.
  - **L628 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3162821544923382, 0.0534881537223529, 0.42511612281358585},`。
- **L629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3226094622720638, 0.05563191003781664, 0.42637698442915734},`.
  - **L629 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3226094622720638, 0.05563191003781664, 0.42637698442915734},`。
- **L630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.32892138033063256, 0.057825220219243655, 0.4275111514018109},`.
  - **L630 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.32892138033063256, 0.057825220219243655, 0.4275111514018109},`。
- **L631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.33521669037969953, 0.060057968826412046, 0.42852404020939816},`.
  - **L631 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.33521669037969953, 0.060057968826412046, 0.42852404020939816},`。
- **L632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.34150060454122194, 0.06232329248965957, 0.42942516526482805},`.
  - **L632 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.34150060454122194, 0.06232329248965957, 0.42942516526482805},`。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.34777091610802874, 0.06461403589398387, 0.43021707827304073},`.
  - **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.34777091610802874, 0.06461403589398387, 0.43021707827304073},`。
- **L634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.35403123226470046, 0.06692277585963113, 0.43090600382653765},`.
  - **L634 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.35403123226470046, 0.06692277585963113, 0.43090600382653765},`。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3602841408353374, 0.06924510666507289, 0.4314970989990494},`.
  - **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3602841408353374, 0.06924510666507289, 0.4314970989990494},`。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.366528457743285, 0.07157684449494817, 0.4319940445656597},`.
  - **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.366528457743285, 0.07157684449494817, 0.4319940445656597},`。
- **L637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3727683648145053, 0.0739131776252345, 0.4324001060667602},`.
  - **L637 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3727683648145053, 0.0739131776252345, 0.4324001060667602},`。
- **L638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.37900068183696356, 0.07625091481761018, 0.432719071099916},`.
  - **L638 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.37900068183696356, 0.07625091481761018, 0.432719071099916},`。
- **L639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3852285882170877, 0.07858924726823341, 0.43295510186497294},`.
  - **L639 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3852285882170877, 0.07858924726823341, 0.43295510186497294},`。
- **L640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.391452904966983, 0.08092498458487726, 0.4331090849213901},`.
  - **L640 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.391452904966983, 0.08092498458487726, 0.4331090849213901},`。
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3976732251941389, 0.08325472094735673, 0.43318307823977154},`.
  - **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3976732251941389, 0.08325472094735673, 0.43318307823977154},`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4038941281371945, 0.08557805215845263, 0.4331790858011016},`.
  - **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4038941281371945, 0.08557805215845263, 0.4331790858011016},`。
- **L643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4101124478324236, 0.08789378949993748, 0.43309809489281975},`.
  - **L643 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4101124478324236, 0.08789378949993748, 0.43309809489281975},`。
- **L644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.41633135152767126, 0.09020111672608602, 0.4329430750225925},`.
  - **L644 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.41633135152767126, 0.09020111672608602, 0.4329430750225925},`。

### Lines 645-672 / 第 645-672 行

````cpp
 645:     {0.4225486702204431, 0.09249885580078815, 0.43271410026220614},
 646:     {0.42876857492222104, 0.09478817843783055, 0.43241205420979634},
 647:     {0.43498689244861954, 0.09706691940470738, 0.43203909514551403},
 648:     {0.4412077984093083, 0.09933623724389631, 0.4315940236596366},
 649:     {0.447428115165217, 0.10159497990744795, 0.43108007898556605},
 650:     {0.453650434334324, 0.10384572179396404, 0.4304981431533577},
 651:     {0.4598753385411611, 0.10608703735300581, 0.4298460517373555},
 652:     {0.4660996564728312, 0.10831978115925261, 0.42912513104437605},
 653:     {0.47232856222023284, 0.11054509253877559, 0.428334014815688},
 654:     {0.47855787863379246, 0.11276183809637635, 0.42747510913916503},
 655:     {0.4847897859200844, 0.11497214601360681, 0.42654796832755787},
 656:     {0.49102210135850827, 0.11717689277568918, 0.4255520769077051},
 657:     {0.4972564191293403, 0.11937663835804257, 0.42448819407856125},
 658:     {0.5034933248743276, 0.12157294554931822, 0.42335603351162365},
 659:     {0.5097296414881407, 0.12376669200308499, 0.4221561654704694},
 660:     {0.5159675484473075, 0.12595799741934913, 0.42088698041253036},
 661:     {0.522205863946251, 0.1281477445657914, 0.4195491273628126},
 662:     {0.5284447718500154, 0.1303390492990618, 0.41814191774726495},
 663:     {0.5346830866012935, 0.1325317964071948, 0.4166670792419288},
 664:     {0.5409194040751939, 0.13472654160063324, 0.4151232492186141},
 665:     {0.5471573097689081, 0.1369268480746088, 0.41351101955274133},
 666:     {0.5533916265944651, 0.13913159264204145, 0.4118292045095448},
 667:     {0.5596245324931008, 0.14134390081316123, 0.41007795018698034},
 668:     {0.565853849035322, 0.14356464417843176, 0.408258150100951},
 669:     {0.5720817545692755, 0.14579495531212058, 0.40636887114816594},
 670:     {0.5783040710826255, 0.1480366969821801, 0.4044110859921461},
 671:     {0.5845203906718561, 0.15029143569692027, 0.4023853090317509},
 672:     {0.5907342930011564, 0.15256075186424045, 0.40029001001388526},
````
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4225486702204431, 0.09249885580078815, 0.43271410026220614},`.
  - **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4225486702204431, 0.09249885580078815, 0.43271410026220614},`。
- **L646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.42876857492222104, 0.09478817843783055, 0.43241205420979634},`.
  - **L646 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.42876857492222104, 0.09478817843783055, 0.43241205420979634},`。
- **L647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.43498689244861954, 0.09706691940470738, 0.43203909514551403},`.
  - **L647 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.43498689244861954, 0.09706691940470738, 0.43203909514551403},`。
- **L648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4412077984093083, 0.09933623724389631, 0.4315940236596366},`.
  - **L648 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4412077984093083, 0.09933623724389631, 0.4315940236596366},`。
- **L649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.447428115165217, 0.10159497990744795, 0.43108007898556605},`.
  - **L649 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.447428115165217, 0.10159497990744795, 0.43108007898556605},`。
- **L650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.453650434334324, 0.10384572179396404, 0.4304981431533577},`.
  - **L650 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.453650434334324, 0.10384572179396404, 0.4304981431533577},`。
- **L651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4598753385411611, 0.10608703735300581, 0.4298460517373555},`.
  - **L651 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4598753385411611, 0.10608703735300581, 0.4298460517373555},`。
- **L652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4660996564728312, 0.10831978115925261, 0.42912513104437605},`.
  - **L652 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4660996564728312, 0.10831978115925261, 0.42912513104437605},`。
- **L653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.47232856222023284, 0.11054509253877559, 0.428334014815688},`.
  - **L653 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.47232856222023284, 0.11054509253877559, 0.428334014815688},`。
- **L654 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.47855787863379246, 0.11276183809637635, 0.42747510913916503},`.
  - **L654 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.47855787863379246, 0.11276183809637635, 0.42747510913916503},`。
- **L655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4847897859200844, 0.11497214601360681, 0.42654796832755787},`.
  - **L655 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4847897859200844, 0.11497214601360681, 0.42654796832755787},`。
- **L656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.49102210135850827, 0.11717689277568918, 0.4255520769077051},`.
  - **L656 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.49102210135850827, 0.11717689277568918, 0.4255520769077051},`。
- **L657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4972564191293403, 0.11937663835804257, 0.42448819407856125},`.
  - **L657 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4972564191293403, 0.11937663835804257, 0.42448819407856125},`。
- **L658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5034933248743276, 0.12157294554931822, 0.42335603351162365},`.
  - **L658 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5034933248743276, 0.12157294554931822, 0.42335603351162365},`。
- **L659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5097296414881407, 0.12376669200308499, 0.4221561654704694},`.
  - **L659 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5097296414881407, 0.12376669200308499, 0.4221561654704694},`。
- **L660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5159675484473075, 0.12595799741934913, 0.42088698041253036},`.
  - **L660 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5159675484473075, 0.12595799741934913, 0.42088698041253036},`。
- **L661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.522205863946251, 0.1281477445657914, 0.4195491273628126},`.
  - **L661 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.522205863946251, 0.1281477445657914, 0.4195491273628126},`。
- **L662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5284447718500154, 0.1303390492990618, 0.41814191774726495},`.
  - **L662 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5284447718500154, 0.1303390492990618, 0.41814191774726495},`。
- **L663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5346830866012935, 0.1325317964071948, 0.4166670792419288},`.
  - **L663 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5346830866012935, 0.1325317964071948, 0.4166670792419288},`。
- **L664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5409194040751939, 0.13472654160063324, 0.4151232492186141},`.
  - **L664 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5409194040751939, 0.13472654160063324, 0.4151232492186141},`。
- **L665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5471573097689081, 0.1369268480746088, 0.41351101955274133},`.
  - **L665 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5471573097689081, 0.1369268480746088, 0.41351101955274133},`。
- **L666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5533916265944651, 0.13913159264204145, 0.4118292045095448},`.
  - **L666 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5533916265944651, 0.13913159264204145, 0.4118292045095448},`。
- **L667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5596245324931008, 0.14134390081316123, 0.41007795018698034},`.
  - **L667 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5596245324931008, 0.14134390081316123, 0.41007795018698034},`。
- **L668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.565853849035322, 0.14356464417843176, 0.408258150100951},`.
  - **L668 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.565853849035322, 0.14356464417843176, 0.408258150100951},`。
- **L669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5720817545692755, 0.14579495531212058, 0.40636887114816594},`.
  - **L669 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5720817545692755, 0.14579495531212058, 0.40636887114816594},`。
- **L670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5783040710826255, 0.1480366969821801, 0.4044110859921461},`.
  - **L670 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5783040710826255, 0.1480366969821801, 0.4044110859921461},`。
- **L671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5845203906718561, 0.15029143569692027, 0.4023853090317509},`.
  - **L671 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5845203906718561, 0.15029143569692027, 0.4023853090317509},`。
- **L672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5907342930011564, 0.15256075186424045, 0.40029001001388526},`.
  - **L672 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5907342930011564, 0.15256075186424045, 0.40029001001388526},`。

### Lines 673-700 / 第 673-700 行

````cpp
 673:     {0.5969396129909775, 0.1548454880249755, 0.3981252481044335},
 674:     {0.6031395137899174, 0.15714880993418692, 0.39589092443931145},
 675:     {0.6093298346523712, 0.15947154274866268, 0.3935891773266438},
 676:     {0.6155137330265519, 0.1618148719991605, 0.3912188293792153},
 677:     {0.6216850552354126, 0.1641816008624372, 0.38878109691187235},
 678:     {0.6278463807173074, 0.1665723257656469, 0.3862763730397491},
 679:     {0.6339982745553374, 0.16898966363736537, 0.3837040052607365},
 680:     {0.6401346020827624, 0.17143538357408133, 0.3810652955393134},
 681:     {0.6462604918955885, 0.17391173185514677, 0.3783589039476966},
 682:     {0.6523688217818822, 0.17641844614159138, 0.37558620861598563},
 683:     {0.6584637067856558, 0.17895980652738866, 0.37274779377320805},
 684:     {0.6645400395062868, 0.1815365142239986, 0.36984611221667457},
 685:     {0.6705983762227489, 0.18415021667573447, 0.36687943961906233},
 686:     {0.6766382547106133, 0.18680458958736995, 0.3638490055304848},
 687:     {0.6826555952415246, 0.1894982847225483, 0.3607573457624624},
 688:     {0.6886534667732338, 0.1922366729397742, 0.3576028897156723},
 689:     {0.6946268114822389, 0.19501835963153946, 0.35438824300823807},
 690:     {0.7005766750861048, 0.19784876490425912, 0.35111276552861453},
 691:     {0.7065000246724039, 0.20072544237887552, 0.34777713155969964},
 692:     {0.7123953793054851, 0.20365311312098916, 0.34438350640978904},
 693:     {0.7182642339188836, 0.20663353472537765, 0.34093101094395056},
 694:     {0.7241025940115757, 0.20966719575096354, 0.3374243970752363},
 695:     {0.7299094384556731, 0.21275663751205837, 0.33386088208499215},
 696:     {0.7356828051012506, 0.21590328739196812, 0.3302452795512392},
 697:     {0.7414236378808644, 0.2191097508283441, 0.32657574613398255},
 698:     {0.7471270116731434, 0.2223753888240187, 0.32285615422841446},
 699:     {0.7527948315414559, 0.22570387532426378, 0.31908460363536006},
 700:     {0.7584222127903781, 0.22909450162198797, 0.31526602130047027},
````
- **L673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5969396129909775, 0.1548454880249755, 0.3981252481044335},`.
  - **L673 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5969396129909775, 0.1548454880249755, 0.3981252481044335},`。
- **L674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6031395137899174, 0.15714880993418692, 0.39589092443931145},`.
  - **L674 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6031395137899174, 0.15714880993418692, 0.39589092443931145},`。
- **L675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6093298346523712, 0.15947154274866268, 0.3935891773266438},`.
  - **L675 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6093298346523712, 0.15947154274866268, 0.3935891773266438},`。
- **L676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6155137330265519, 0.1618148719991605, 0.3912188293792153},`.
  - **L676 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6155137330265519, 0.1618148719991605, 0.3912188293792153},`。
- **L677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6216850552354126, 0.1641816008624372, 0.38878109691187235},`.
  - **L677 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6216850552354126, 0.1641816008624372, 0.38878109691187235},`。
- **L678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6278463807173074, 0.1665723257656469, 0.3862763730397491},`.
  - **L678 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6278463807173074, 0.1665723257656469, 0.3862763730397491},`。
- **L679 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6339982745553374, 0.16898966363736537, 0.3837040052607365},`.
  - **L679 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6339982745553374, 0.16898966363736537, 0.3837040052607365},`。
- **L680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6401346020827624, 0.17143538357408133, 0.3810652955393134},`.
  - **L680 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6401346020827624, 0.17143538357408133, 0.3810652955393134},`。
- **L681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6462604918955885, 0.17391173185514677, 0.3783589039476966},`.
  - **L681 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6462604918955885, 0.17391173185514677, 0.3783589039476966},`。
- **L682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6523688217818822, 0.17641844614159138, 0.37558620861598563},`.
  - **L682 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6523688217818822, 0.17641844614159138, 0.37558620861598563},`。
- **L683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6584637067856558, 0.17895980652738866, 0.37274779377320805},`.
  - **L683 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6584637067856558, 0.17895980652738866, 0.37274779377320805},`。
- **L684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6645400395062868, 0.1815365142239986, 0.36984611221667457},`.
  - **L684 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6645400395062868, 0.1815365142239986, 0.36984611221667457},`。
- **L685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6705983762227489, 0.18415021667573447, 0.36687943961906233},`.
  - **L685 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6705983762227489, 0.18415021667573447, 0.36687943961906233},`。
- **L686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6766382547106133, 0.18680458958736995, 0.3638490055304848},`.
  - **L686 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6766382547106133, 0.18680458958736995, 0.3638490055304848},`。
- **L687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6826555952415246, 0.1894982847225483, 0.3607573457624624},`.
  - **L687 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6826555952415246, 0.1894982847225483, 0.3607573457624624},`。
- **L688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6886534667732338, 0.1922366729397742, 0.3576028897156723},`.
  - **L688 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6886534667732338, 0.1922366729397742, 0.3576028897156723},`。
- **L689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6946268114822389, 0.19501835963153946, 0.35438824300823807},`.
  - **L689 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6946268114822389, 0.19501835963153946, 0.35438824300823807},`。
- **L690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7005766750861048, 0.19784876490425912, 0.35111276552861453},`.
  - **L690 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7005766750861048, 0.19784876490425912, 0.35111276552861453},`。
- **L691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7065000246724039, 0.20072544237887552, 0.34777713155969964},`.
  - **L691 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7065000246724039, 0.20072544237887552, 0.34777713155969964},`。
- **L692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7123953793054851, 0.20365311312098916, 0.34438350640978904},`.
  - **L692 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7123953793054851, 0.20365311312098916, 0.34438350640978904},`。
- **L693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7182642339188836, 0.20663353472537765, 0.34093101094395056},`.
  - **L693 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7182642339188836, 0.20663353472537765, 0.34093101094395056},`。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7241025940115757, 0.20966719575096354, 0.3374243970752363},`.
  - **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7241025940115757, 0.20966719575096354, 0.3374243970752363},`。
- **L695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7299094384556731, 0.21275663751205837, 0.33386088208499215},`.
  - **L695 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7299094384556731, 0.21275663751205837, 0.33386088208499215},`。
- **L696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7356828051012506, 0.21590328739196812, 0.3302452795512392},`.
  - **L696 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7356828051012506, 0.21590328739196812, 0.3302452795512392},`。
- **L697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7414236378808644, 0.2191097508283441, 0.32657574613398255},`.
  - **L697 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7414236378808644, 0.2191097508283441, 0.32657574613398255},`。
- **L698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7471270116731434, 0.2223753888240187, 0.32285615422841446},`.
  - **L698 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7471270116731434, 0.2223753888240187, 0.32285615422841446},`。
- **L699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7527948315414559, 0.22570387532426378, 0.31908460363536006},`.
  - **L699 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7527948315414559, 0.22570387532426378, 0.31908460363536006},`。
- **L700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7584222127903781, 0.22909450162198797, 0.31526602130047027},`.
  - **L700 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7584222127903781, 0.22909450162198797, 0.31526602130047027},`。

### Lines 701-728 / 第 701-728 行

````cpp
 701:     {0.7640096003184447, 0.23255111985362076, 0.3113994474823908},
 702:     {0.769556407505972, 0.236074626914262, 0.30748488142933444},
 703:     {0.7750588038588068, 0.23966423164432504, 0.3035263167932573},
 704:     {0.780517595641067, 0.24332476411029125, 0.29952273568573573},
 705:     {0.7859290015238034, 0.2470533545534328, 0.29547717944438406},
 706:     {0.7912937764432374, 0.25085391354890735, 0.29138958485956357},
 707:     {0.7966071922843984, 0.25472548987486643, 0.28726403596779526},
 708:     {0.8018706152376446, 0.2586710580118296, 0.28309949569184917},
 709:     {0.8070823747046473, 0.26268963921769606, 0.27889788734079896},
 710:     {0.8122388089759692, 0.26678319189183297, 0.2746613536242391},
 711:     {0.8173415491080798, 0.27095180082645115, 0.27038973420543944},
 712:     {0.8223859952913317, 0.27519433772522317, 0.2660852062616442},
 713:     {0.8273727148541918, 0.2795149750247453, 0.26174957724469494},
 714:     {0.8322991733910813, 0.2839104959497832, 0.25738305415913015},
 715:     {0.8371646398804131, 0.2883820094161365, 0.252988540402642},
 716:     {0.8419693412456319, 0.29293066888944563, 0.2485638989415601},
 717:     {0.846708821213788, 0.29755616578042293, 0.2441133893753179},
 718:     {0.8513844997023888, 0.3022578538064719, 0.23963574040066615},
 719:     {0.855991993845198, 0.3070353341236893, 0.23513323409260284},
 720:     {0.8605336484626919, 0.31189005054876767, 0.230605578997568},
 721:     {0.865006157141607, 0.316819514079576, 0.2260550749407627},
 722:     {0.8694086743123614, 0.3218239712798184, 0.2214825826370368},
 723:     {0.873741308408031, 0.32690370860531126, 0.21688591478628577},
 724:     {0.8780008409852356, 0.33205714927467816, 0.212268424608822},
 725:     {0.8821884493306369, 0.33728491405873967, 0.20762775209164724},
 726:     {0.8863019976673894, 0.3425833384271254, 0.20296826303399285},
 727:     {0.8903415794934312, 0.3479551300484793, 0.19828558691030201},
 728:     {0.894305144045262, 0.3533965381592675, 0.19358409819949085},
````
- **L701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7640096003184447, 0.23255111985362076, 0.3113994474823908},`.
  - **L701 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7640096003184447, 0.23255111985362076, 0.3113994474823908},`。
- **L702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.769556407505972, 0.236074626914262, 0.30748488142933444},`.
  - **L702 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.769556407505972, 0.236074626914262, 0.30748488142933444},`。
- **L703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7750588038588068, 0.23966423164432504, 0.3035263167932573},`.
  - **L703 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7750588038588068, 0.23966423164432504, 0.3035263167932573},`。
- **L704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.780517595641067, 0.24332476411029125, 0.29952273568573573},`.
  - **L704 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.780517595641067, 0.24332476411029125, 0.29952273568573573},`。
- **L705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7859290015238034, 0.2470533545534328, 0.29547717944438406},`.
  - **L705 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7859290015238034, 0.2470533545534328, 0.29547717944438406},`。
- **L706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7912937764432374, 0.25085391354890735, 0.29138958485956357},`.
  - **L706 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7912937764432374, 0.25085391354890735, 0.29138958485956357},`。
- **L707 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7966071922843984, 0.25472548987486643, 0.28726403596779526},`.
  - **L707 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7966071922843984, 0.25472548987486643, 0.28726403596779526},`。
- **L708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8018706152376446, 0.2586710580118296, 0.28309949569184917},`.
  - **L708 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8018706152376446, 0.2586710580118296, 0.28309949569184917},`。
- **L709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8070823747046473, 0.26268963921769606, 0.27889788734079896},`.
  - **L709 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8070823747046473, 0.26268963921769606, 0.27889788734079896},`。
- **L710 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8122388089759692, 0.26678319189183297, 0.2746613536242391},`.
  - **L710 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8122388089759692, 0.26678319189183297, 0.2746613536242391},`。
- **L711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8173415491080798, 0.27095180082645115, 0.27038973420543944},`.
  - **L711 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8173415491080798, 0.27095180082645115, 0.27038973420543944},`。
- **L712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8223859952913317, 0.27519433772522317, 0.2660852062616442},`.
  - **L712 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8223859952913317, 0.27519433772522317, 0.2660852062616442},`。
- **L713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8273727148541918, 0.2795149750247453, 0.26174957724469494},`.
  - **L713 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8273727148541918, 0.2795149750247453, 0.26174957724469494},`。
- **L714 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8322991733910813, 0.2839104959497832, 0.25738305415913015},`.
  - **L714 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8322991733910813, 0.2839104959497832, 0.25738305415913015},`。
- **L715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8371646398804131, 0.2883820094161365, 0.252988540402642},`.
  - **L715 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8371646398804131, 0.2883820094161365, 0.252988540402642},`。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8419693412456319, 0.29293066888944563, 0.2485638989415601},`.
  - **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8419693412456319, 0.29293066888944563, 0.2485638989415601},`。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.846708821213788, 0.29755616578042293, 0.2441133893753179},`.
  - **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.846708821213788, 0.29755616578042293, 0.2441133893753179},`。
- **L718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8513844997023888, 0.3022578538064719, 0.23963574040066615},`.
  - **L718 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8513844997023888, 0.3022578538064719, 0.23963574040066615},`。
- **L719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.855991993845198, 0.3070353341236893, 0.23513323409260284},`.
  - **L719 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.855991993845198, 0.3070353341236893, 0.23513323409260284},`。
- **L720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8605336484626919, 0.31189005054876767, 0.230605578997568},`.
  - **L720 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8605336484626919, 0.31189005054876767, 0.230605578997568},`。
- **L721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.865006157141607, 0.316819514079576, 0.2260550749407627},`.
  - **L721 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.865006157141607, 0.316819514079576, 0.2260550749407627},`。
- **L722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8694086743123614, 0.3218239712798184, 0.2214825826370368},`.
  - **L722 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8694086743123614, 0.3218239712798184, 0.2214825826370368},`。
- **L723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.873741308408031, 0.32690370860531126, 0.21688591478628577},`.
  - **L723 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.873741308408031, 0.32690370860531126, 0.21688591478628577},`。
- **L724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8780008409852356, 0.33205714927467816, 0.212268424608822},`.
  - **L724 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8780008409852356, 0.33205714927467816, 0.212268424608822},`。
- **L725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8821884493306369, 0.33728491405873967, 0.20762775209164724},`.
  - **L725 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8821884493306369, 0.33728491405873967, 0.20762775209164724},`。
- **L726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8863019976673894, 0.3425833384271254, 0.20296826303399285},`.
  - **L726 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8863019976673894, 0.3425833384271254, 0.20296826303399285},`。
- **L727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8903415794934312, 0.3479551300484793, 0.19828558691030201},`.
  - **L727 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8903415794934312, 0.3479551300484793, 0.19828558691030201},`。
- **L728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.894305144045262, 0.3533965381592675, 0.19358409819949085},`.
  - **L728 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.894305144045262, 0.3533965381592675, 0.19358409819949085},`。

### Lines 729-756 / 第 729-756 行

````cpp
 729:     {0.8981917181039129, 0.35890794053186803, 0.1888606247963219},
 730:     {0.902003277546344, 0.36448975074745, 0.18411593340897883},
 731:     {0.9057348680145032, 0.37013713800749826, 0.17935046113406727},
 732:     {0.9093903996647411, 0.3758539732830311, 0.17456276670706064},
 733:     {0.9129660070101344, 0.38163334525219944, 0.1697552941505453},
 734:     {0.9164625104530708, 0.3874792047452722, 0.16492359727200515},
 735:     {0.9198791350013626, 0.39338656173222386, 0.16007012382758104},
 736:     {0.9232147698912994, 0.3993559137889013, 0.1551936703489871},
 737:     {0.9264702498113229, 0.40538678902429115, 0.15029195385198924},
 738:     {0.9296439014941755, 0.41147612778815296, 0.14536750158970949},
 739:     {0.9327373523126056, 0.4176250250322833, 0.14041678247519898},
 740:     {0.9357470217364718, 0.42382835018615, 0.13544032915276746},
 741:     {0.9386754432346461, 0.4300892683482416, 0.13043760827101122},
 742:     {0.9415211303729047, 0.4364025802485547, 0.12540915246812306},
 743:     {0.9442848284617543, 0.4427688880476121, 0.12035472282994122},
 744:     {0.9469652258626157, 0.44918881902600133, 0.11527197615350762},
 745:     {0.9495619408937822, 0.4556571156219838, 0.11016454791032279},
 746:     {0.9520753085089858, 0.4621760654003351, 0.10503080049999589},
 747:     {0.9545060414475516, 0.468741350075834, 0.09987436834002841},
 748:     {0.9568523791775677, 0.4753543175690524, 0.09469462335752518},
 749:     {0.9591141303022052, 0.4820115908490303, 0.08949918430671361},
 750:     {0.9612934380573407, 0.4887145750891598, 0.08428844691950368},
 751:     {0.9633872061002898, 0.4954598384326913, 0.07907300229562086},
 752:     {0.9653969856405892, 0.5022460991811288, 0.07385959127645336},
 753:     {0.9673222685246364, 0.5090760925737746, 0.06865882956308128},
 754:     {0.9691630662078553, 0.5159433430954236, 0.06348840124042007},
 755:     {0.970919318954511, 0.5228513513717987, 0.05836666742473027},
 756:     {0.9725901348141913, 0.529795592113789, 0.053324210911405455},
````
- **L729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8981917181039129, 0.35890794053186803, 0.1888606247963219},`.
  - **L729 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8981917181039129, 0.35890794053186803, 0.1888606247963219},`。
- **L730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.902003277546344, 0.36448975074745, 0.18411593340897883},`.
  - **L730 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.902003277546344, 0.36448975074745, 0.18411593340897883},`。
- **L731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9057348680145032, 0.37013713800749826, 0.17935046113406727},`.
  - **L731 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9057348680145032, 0.37013713800749826, 0.17935046113406727},`。
- **L732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9093903996647411, 0.3758539732830311, 0.17456276670706064},`.
  - **L732 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9093903996647411, 0.3758539732830311, 0.17456276670706064},`。
- **L733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9129660070101344, 0.38163334525219944, 0.1697552941505453},`.
  - **L733 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9129660070101344, 0.38163334525219944, 0.1697552941505453},`。
- **L734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9164625104530708, 0.3874792047452722, 0.16492359727200515},`.
  - **L734 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9164625104530708, 0.3874792047452722, 0.16492359727200515},`。
- **L735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9198791350013626, 0.39338656173222386, 0.16007012382758104},`.
  - **L735 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9198791350013626, 0.39338656173222386, 0.16007012382758104},`。
- **L736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9232147698912994, 0.3993559137889013, 0.1551936703489871},`.
  - **L736 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9232147698912994, 0.3993559137889013, 0.1551936703489871},`。
- **L737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9264702498113229, 0.40538678902429115, 0.15029195385198924},`.
  - **L737 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9264702498113229, 0.40538678902429115, 0.15029195385198924},`。
- **L738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9296439014941755, 0.41147612778815296, 0.14536750158970949},`.
  - **L738 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9296439014941755, 0.41147612778815296, 0.14536750158970949},`。
- **L739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9327373523126056, 0.4176250250322833, 0.14041678247519898},`.
  - **L739 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9327373523126056, 0.4176250250322833, 0.14041678247519898},`。
- **L740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9357470217364718, 0.42382835018615, 0.13544032915276746},`.
  - **L740 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9357470217364718, 0.42382835018615, 0.13544032915276746},`。
- **L741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9386754432346461, 0.4300892683482416, 0.13043760827101122},`.
  - **L741 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9386754432346461, 0.4300892683482416, 0.13043760827101122},`。
- **L742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9415211303729047, 0.4364025802485547, 0.12540915246812306},`.
  - **L742 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9415211303729047, 0.4364025802485547, 0.12540915246812306},`。
- **L743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9442848284617543, 0.4427688880476121, 0.12035472282994122},`.
  - **L743 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9442848284617543, 0.4427688880476121, 0.12035472282994122},`。
- **L744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9469652258626157, 0.44918881902600133, 0.11527197615350762},`.
  - **L744 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9469652258626157, 0.44918881902600133, 0.11527197615350762},`。
- **L745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9495619408937822, 0.4556571156219838, 0.11016454791032279},`.
  - **L745 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9495619408937822, 0.4556571156219838, 0.11016454791032279},`。
- **L746 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9520753085089858, 0.4621760654003351, 0.10503080049999589},`.
  - **L746 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9520753085089858, 0.4621760654003351, 0.10503080049999589},`。
- **L747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9545060414475516, 0.468741350075834, 0.09987436834002841},`.
  - **L747 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9545060414475516, 0.468741350075834, 0.09987436834002841},`。
- **L748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9568523791775677, 0.4753543175690524, 0.09469462335752518},`.
  - **L748 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9568523791775677, 0.4753543175690524, 0.09469462335752518},`。
- **L749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9591141303022052, 0.4820115908490303, 0.08949918430671361},`.
  - **L749 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9591141303022052, 0.4820115908490303, 0.08949918430671361},`。
- **L750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9612934380573407, 0.4887145750891598, 0.08428844691950368},`.
  - **L750 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9612934380573407, 0.4887145750891598, 0.08428844691950368},`。
- **L751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9633872061002898, 0.4954598384326913, 0.07907300229562086},`.
  - **L751 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9633872061002898, 0.4954598384326913, 0.07907300229562086},`。
- **L752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9653969856405892, 0.5022460991811288, 0.07385959127645336},`.
  - **L752 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9653969856405892, 0.5022460991811288, 0.07385959127645336},`。
- **L753 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9673222685246364, 0.5090760925737746, 0.06865882956308128},`.
  - **L753 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9673222685246364, 0.5090760925737746, 0.06865882956308128},`。
- **L754 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9691630662078553, 0.5159433430954236, 0.06348840124042007},`.
  - **L754 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9691630662078553, 0.5159433430954236, 0.06348840124042007},`。
- **L755 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.970919318954511, 0.5228513513717987, 0.05836666742473027},`.
  - **L755 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.970919318954511, 0.5228513513717987, 0.05836666742473027},`。
- **L756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9725901348141913, 0.529795592113789, 0.053324210911405455},`.
  - **L756 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9725901348141913, 0.529795592113789, 0.053324210911405455},`。

### Lines 757-784 / 第 757-784 行

````cpp
 757:     {0.9741763574084353, 0.5367786143884726, 0.04839153015610343},
 758:     {0.975677190645675, 0.5437958464199314, 0.043618036090846},
 759:     {0.9770920356520277, 0.5508470769374313, 0.03905055303111867},
 760:     {0.9784222326450961, 0.5579351064433469, 0.034930916546532825},
 761:     {0.979666095876416, 0.5650543281731687, 0.0314092990272324},
 762:     {0.9808242624402922, 0.5722073701062071, 0.028507883553058225},
 763:     {0.9818951440117506, 0.5793895836320366, 0.02625012573179964},
 764:     {0.982881280054488, 0.5866046371076197, 0.024660950197297846},
 765:     {0.9837791795198422, 0.5938468430754209, 0.023770046474572596},
 766:     {0.9845910909683125, 0.6011190480992957, 0.023606059102018802},
 767:     {0.9853152007413414, 0.608420107478453, 0.02420209240341553},
 768:     {0.9859521306674056, 0.6157473052058917, 0.025591934272695092},
 769:     {0.9865022096405148, 0.6231033747802108, 0.027814257478890377},
 770:     {0.9869641579064014, 0.6304825655882533, 0.030907915000313535},
 771:     {0.9873372060008064, 0.6378886447799842, 0.034916559619452406},
 772:     {0.987622172670732, 0.6453178289458518, 0.039886017500422775},
 773:     {0.9878191520033638, 0.6527700129448509, 0.045580403012790684},
 774:     {0.9879261727372323, 0.6602480965501968, 0.05175024605159847},
 775:     {0.9879451706317489, 0.6677452744424257, 0.05832852598609512},
 776:     {0.9878741601998459, 0.675265366380739, 0.06525752407999996},
 777:     {0.9877141765851464, 0.6828045385569567, 0.07248872059669512},
 778:     {0.9874641349558363, 0.690364638021527, 0.07999083430127826},
 779:     {0.9871241697382241, 0.6979418049868507, 0.08773096280857864},
 780:     {0.9866942187047217, 0.7055369718171162, 0.09569310480416554},
 781:     {0.9861751485274827, 0.7131510743985874, 0.103863256330502},
 782:     {0.9855662155788282, 0.7207792367356154, 0.11222834433058854},
 783:     {0.9848651143109276, 0.7284253454782301, 0.12078557949263882},
 784:     {0.9840751996931422, 0.7360845034258754, 0.12952661474987895},
````
- **L757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9741763574084353, 0.5367786143884726, 0.04839153015610343},`.
  - **L757 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9741763574084353, 0.5367786143884726, 0.04839153015610343},`。
- **L758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.975677190645675, 0.5437958464199314, 0.043618036090846},`.
  - **L758 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.975677190645675, 0.5437958464199314, 0.043618036090846},`。
- **L759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9770920356520277, 0.5508470769374313, 0.03905055303111867},`.
  - **L759 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9770920356520277, 0.5508470769374313, 0.03905055303111867},`。
- **L760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9784222326450961, 0.5579351064433469, 0.034930916546532825},`.
  - **L760 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9784222326450961, 0.5579351064433469, 0.034930916546532825},`。
- **L761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.979666095876416, 0.5650543281731687, 0.0314092990272324},`.
  - **L761 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.979666095876416, 0.5650543281731687, 0.0314092990272324},`。
- **L762 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9808242624402922, 0.5722073701062071, 0.028507883553058225},`.
  - **L762 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9808242624402922, 0.5722073701062071, 0.028507883553058225},`。
- **L763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9818951440117506, 0.5793895836320366, 0.02625012573179964},`.
  - **L763 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9818951440117506, 0.5793895836320366, 0.02625012573179964},`。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.982881280054488, 0.5866046371076197, 0.024660950197297846},`.
  - **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.982881280054488, 0.5866046371076197, 0.024660950197297846},`。
- **L765 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9837791795198422, 0.5938468430754209, 0.023770046474572596},`.
  - **L765 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9837791795198422, 0.5938468430754209, 0.023770046474572596},`。
- **L766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9845910909683125, 0.6011190480992957, 0.023606059102018802},`.
  - **L766 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9845910909683125, 0.6011190480992957, 0.023606059102018802},`。
- **L767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9853152007413414, 0.608420107478453, 0.02420209240341553},`.
  - **L767 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9853152007413414, 0.608420107478453, 0.02420209240341553},`。
- **L768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9859521306674056, 0.6157473052058917, 0.025591934272695092},`.
  - **L768 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9859521306674056, 0.6157473052058917, 0.025591934272695092},`。
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9865022096405148, 0.6231033747802108, 0.027814257478890377},`.
  - **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9865022096405148, 0.6231033747802108, 0.027814257478890377},`。
- **L770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9869641579064014, 0.6304825655882533, 0.030907915000313535},`.
  - **L770 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9869641579064014, 0.6304825655882533, 0.030907915000313535},`。
- **L771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9873372060008064, 0.6378886447799842, 0.034916559619452406},`.
  - **L771 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9873372060008064, 0.6378886447799842, 0.034916559619452406},`。
- **L772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.987622172670732, 0.6453178289458518, 0.039886017500422775},`.
  - **L772 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.987622172670732, 0.6453178289458518, 0.039886017500422775},`。
- **L773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9878191520033638, 0.6527700129448509, 0.045580403012790684},`.
  - **L773 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9878191520033638, 0.6527700129448509, 0.045580403012790684},`。
- **L774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9879261727372323, 0.6602480965501968, 0.05175024605159847},`.
  - **L774 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9879261727372323, 0.6602480965501968, 0.05175024605159847},`。
- **L775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9879451706317489, 0.6677452744424257, 0.05832852598609512},`.
  - **L775 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9879451706317489, 0.6677452744424257, 0.05832852598609512},`。
- **L776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9878741601998459, 0.675265366380739, 0.06525752407999996},`.
  - **L776 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9878741601998459, 0.675265366380739, 0.06525752407999996},`。
- **L777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9877141765851464, 0.6828045385569567, 0.07248872059669512},`.
  - **L777 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9877141765851464, 0.6828045385569567, 0.07248872059669512},`。
- **L778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9874641349558363, 0.690364638021527, 0.07999083430127826},`.
  - **L778 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9874641349558363, 0.690364638021527, 0.07999083430127826},`。
- **L779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9871241697382241, 0.6979418049868507, 0.08773096280857864},`.
  - **L779 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9871241697382241, 0.6979418049868507, 0.08773096280857864},`。
- **L780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9866942187047217, 0.7055369718171162, 0.09569310480416554},`.
  - **L780 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9866942187047217, 0.7055369718171162, 0.09569310480416554},`。
- **L781 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9861751485274827, 0.7131510743985874, 0.103863256330502},`.
  - **L781 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9861751485274827, 0.7131510743985874, 0.103863256330502},`。
- **L782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9855662155788282, 0.7207792367356154, 0.11222834433058854},`.
  - **L782 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9855662155788282, 0.7207792367356154, 0.11222834433058854},`。
- **L783 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9848651143109276, 0.7284253454782301, 0.12078557949263882},`.
  - **L783 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9848651143109276, 0.7284253454782301, 0.12078557949263882},`。
- **L784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9840751996931422, 0.7360845034258754, 0.12952661474987895},`.
  - **L784 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9840751996931422, 0.7360845034258754, 0.12952661474987895},`。

### Lines 785-812 / 第 785-812 行

````cpp
 785:     {0.9831960676449997, 0.7437566173190143, 0.13845392837069237},
 786:     {0.9822281707577935, 0.7514397717174452, 0.14756491297187693},
 787:     {0.9811732892122574, 0.7591319265002779, 0.15686190499859645},
 788:     {0.9800321283245658, 0.7668350414250283, 0.16635325033390524},
 789:     {0.9788062634501479, 0.7745421938654863, 0.1760361942373471},
 790:     {0.9774970739304246, 0.7822563112740953, 0.18592362086777084},
 791:     {0.976108224906952, 0.7899714619636461, 0.1960175103988243},
 792:     {0.9746380082272822, 0.7976905802513132, 0.20633302377351131},
 793:     {0.9730881744707035, 0.8054067300567858, 0.21687685576253862},
 794:     {0.9714683557085738, 0.8131188812721618, 0.2276566805995705},
 795:     {0.9697831130812914, 0.8208229971469754, 0.23868624376433029},
 796:     {0.968041305354793, 0.8285121501431812, 0.24997101177190928},
 797:     {0.9662430438023951, 0.836189261303023, 0.2615346777715621},
 798:     {0.9643942451078572, 0.8438454171701422, 0.273390375539181},
 799:     {0.9625169727369577, 0.8514745194008927, 0.2855471549893648},
 800:     {0.9606261754860036, 0.8590666816140542, 0.29800977930778266},
 801:     {0.9587199044763673, 0.86662276892152, 0.310821685193729},
 802:     {0.9568341051627561, 0.8741269393072385, 0.3239742355406551},
 803:     {0.9549973084887696, 0.8815661177531517, 0.3374737603502378},
 804:     {0.9532150458875698, 0.8889401848892272, 0.3513697543417886},
 805:     {0.9515462272506953, 0.8962233797142661, 0.36562618964558136},
 806:     {0.950018012245954, 0.9034074125145412, 0.3802723264284489},
 807:     {0.9486831530694507, 0.9104706308186431, 0.3952886725437556},
 808:     {0.9475940275601823, 0.9173976138382822, 0.4106669431858655},
 809:     {0.9468091087983127, 0.9241658620670481, 0.4263732130148635},
 810:     {0.9463921563722028, 0.9307581300310862, 0.4423654634949841},
 811:     {0.9464031311450462, 0.9371570608055302, 0.458592798889426},
 812:     {0.9469030741916794, 0.9433453721925603, 0.4749690020361225},
````
- **L785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9831960676449997, 0.7437566173190143, 0.13845392837069237},`.
  - **L785 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9831960676449997, 0.7437566173190143, 0.13845392837069237},`。
- **L786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9822281707577935, 0.7514397717174452, 0.14756491297187693},`.
  - **L786 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9822281707577935, 0.7514397717174452, 0.14756491297187693},`。
- **L787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9811732892122574, 0.7591319265002779, 0.15686190499859645},`.
  - **L787 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9811732892122574, 0.7591319265002779, 0.15686190499859645},`。
- **L788 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9800321283245658, 0.7668350414250283, 0.16635325033390524},`.
  - **L788 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9800321283245658, 0.7668350414250283, 0.16635325033390524},`。
- **L789 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9788062634501479, 0.7745421938654863, 0.1760361942373471},`.
  - **L789 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9788062634501479, 0.7745421938654863, 0.1760361942373471},`。
- **L790 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9774970739304246, 0.7822563112740953, 0.18592362086777084},`.
  - **L790 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9774970739304246, 0.7822563112740953, 0.18592362086777084},`。
- **L791 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.976108224906952, 0.7899714619636461, 0.1960175103988243},`.
  - **L791 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.976108224906952, 0.7899714619636461, 0.1960175103988243},`。
- **L792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9746380082272822, 0.7976905802513132, 0.20633302377351131},`.
  - **L792 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9746380082272822, 0.7976905802513132, 0.20633302377351131},`。
- **L793 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9730881744707035, 0.8054067300567858, 0.21687685576253862},`.
  - **L793 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9730881744707035, 0.8054067300567858, 0.21687685576253862},`。
- **L794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9714683557085738, 0.8131188812721618, 0.2276566805995705},`.
  - **L794 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9714683557085738, 0.8131188812721618, 0.2276566805995705},`。
- **L795 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9697831130812914, 0.8208229971469754, 0.23868624376433029},`.
  - **L795 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9697831130812914, 0.8208229971469754, 0.23868624376433029},`。
- **L796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.968041305354793, 0.8285121501431812, 0.24997101177190928},`.
  - **L796 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.968041305354793, 0.8285121501431812, 0.24997101177190928},`。
- **L797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9662430438023951, 0.836189261303023, 0.2615346777715621},`.
  - **L797 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9662430438023951, 0.836189261303023, 0.2615346777715621},`。
- **L798 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9643942451078572, 0.8438454171701422, 0.273390375539181},`.
  - **L798 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9643942451078572, 0.8438454171701422, 0.273390375539181},`。
- **L799 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9625169727369577, 0.8514745194008927, 0.2855471549893648},`.
  - **L799 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9625169727369577, 0.8514745194008927, 0.2855471549893648},`。
- **L800 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9606261754860036, 0.8590666816140542, 0.29800977930778266},`.
  - **L800 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9606261754860036, 0.8590666816140542, 0.29800977930778266},`。
- **L801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9587199044763673, 0.86662276892152, 0.310821685193729},`.
  - **L801 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9587199044763673, 0.86662276892152, 0.310821685193729},`。
- **L802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9568341051627561, 0.8741269393072385, 0.3239742355406551},`.
  - **L802 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9568341051627561, 0.8741269393072385, 0.3239742355406551},`。
- **L803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9549973084887696, 0.8815661177531517, 0.3374737603502378},`.
  - **L803 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9549973084887696, 0.8815661177531517, 0.3374737603502378},`。
- **L804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9532150458875698, 0.8889401848892272, 0.3513697543417886},`.
  - **L804 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9532150458875698, 0.8889401848892272, 0.3513697543417886},`。
- **L805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9515462272506953, 0.8962233797142661, 0.36562618964558136},`.
  - **L805 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9515462272506953, 0.8962233797142661, 0.36562618964558136},`。
- **L806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.950018012245954, 0.9034074125145412, 0.3802723264284489},`.
  - **L806 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.950018012245954, 0.9034074125145412, 0.3802723264284489},`。
- **L807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9486831530694507, 0.9104706308186431, 0.3952886725437556},`.
  - **L807 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9486831530694507, 0.9104706308186431, 0.3952886725437556},`。
- **L808 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9475940275601823, 0.9173976138382822, 0.4106669431858655},`.
  - **L808 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9475940275601823, 0.9173976138382822, 0.4106669431858655},`。
- **L809 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9468091087983127, 0.9241658620670481, 0.4263732130148635},`.
  - **L809 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9468091087983127, 0.9241658620670481, 0.4263732130148635},`。
- **L810 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9463921563722028, 0.9307581300310862, 0.4423654634949841},`.
  - **L810 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9463921563722028, 0.9307581300310862, 0.4423654634949841},`。
- **L811 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9464031311450462, 0.9371570608055302, 0.458592798889426},`.
  - **L811 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9464031311450462, 0.9371570608055302, 0.458592798889426},`。
- **L812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9469030741916794, 0.9433453721925603, 0.4749690020361225},`.
  - **L812 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9469030741916794, 0.9433453721925603, 0.4749690020361225},`。

### Lines 813-840 / 第 813-840 行

````cpp
 813:     {0.9479372327167105, 0.9493162253885609, 0.49142738476056075},
 814:     {0.9495450508917115, 0.9550605846153878, 0.5078595779865485},
 815:     {0.9517404201103928, 0.9605853570748163, 0.5242049355302347},
 816:     {0.9545291096426641, 0.9658937621848641, 0.5403611560472134},
 817:     {0.9578957403425837, 0.971000188760133, 0.5562734123813695},
 818:     {0.9618122666687167, 0.9759219036314865, 0.571925695133198},
 819:     {0.966248777481722, 0.980675368253291, 0.5872050182876477},
 820:     {0.9711624870815522, 0.985280023457573, 0.6021551859229256},
 821:     {0.9765108962599102, 0.9897505184610552, 0.6167595818025277},
 822:     {0.9822577545753769, 0.9941071290312428, 0.6310186261682112},
 823:     {0.9883620799212208, 0.9983616470620554, 0.6449240982803861},
 824: }};
 825: 
 826: template <typename Real>
 827: static constexpr std::array<std::array<Real, 3>, 256> black_body_data_ = {{
 828:     {0.0, 0.0, 0.0},
 829:     {0.013038855104993618, 0.0037537033758315535, 0.002103027943341456},
 830:     {0.02607771020998725, 0.007507406751663157, 0.004206055886683011},
 831:     {0.03911656531498074, 0.01126111012749475, 0.006309083830024466},
 832:     {0.05111281846889674, 0.0150148135033264, 0.008412111773366015},
 833:     {0.06145201887409168, 0.018768516879157954, 0.010515139716707521},
 834:     {0.07064326337459148, 0.02252222025498951, 0.012618167660048977},
 835:     {0.07897806577026972, 0.026275923630821117, 0.014721195603390481},
 836:     {0.08664361913059343, 0.03002962700665271, 0.016824223546731985},
 837:     {0.09376835672821318, 0.03378333038248431, 0.018927251490073488},
 838:     {0.10044479589169861, 0.03753703375831587, 0.021030279433414945},
 839:     {0.10674212293169041, 0.041271634201045536, 0.02313330737675645},
 840:     {0.11271363536038692, 0.04482597891806931, 0.025236335320097954},
````
- **L813 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9479372327167105, 0.9493162253885609, 0.49142738476056075},`.
  - **L813 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9479372327167105, 0.9493162253885609, 0.49142738476056075},`。
- **L814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9495450508917115, 0.9550605846153878, 0.5078595779865485},`.
  - **L814 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9495450508917115, 0.9550605846153878, 0.5078595779865485},`。
- **L815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9517404201103928, 0.9605853570748163, 0.5242049355302347},`.
  - **L815 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9517404201103928, 0.9605853570748163, 0.5242049355302347},`。
- **L816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9545291096426641, 0.9658937621848641, 0.5403611560472134},`.
  - **L816 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9545291096426641, 0.9658937621848641, 0.5403611560472134},`。
- **L817 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9578957403425837, 0.971000188760133, 0.5562734123813695},`.
  - **L817 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9578957403425837, 0.971000188760133, 0.5562734123813695},`。
- **L818 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9618122666687167, 0.9759219036314865, 0.571925695133198},`.
  - **L818 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9618122666687167, 0.9759219036314865, 0.571925695133198},`。
- **L819 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.966248777481722, 0.980675368253291, 0.5872050182876477},`.
  - **L819 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.966248777481722, 0.980675368253291, 0.5872050182876477},`。
- **L820 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9711624870815522, 0.985280023457573, 0.6021551859229256},`.
  - **L820 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9711624870815522, 0.985280023457573, 0.6021551859229256},`。
- **L821 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9765108962599102, 0.9897505184610552, 0.6167595818025277},`.
  - **L821 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9765108962599102, 0.9897505184610552, 0.6167595818025277},`。
- **L822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9822577545753769, 0.9941071290312428, 0.6310186261682112},`.
  - **L822 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9822577545753769, 0.9941071290312428, 0.6310186261682112},`。
- **L823 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9883620799212208, 0.9983616470620554, 0.6449240982803861},`.
  - **L823 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9883620799212208, 0.9983616470620554, 0.6449240982803861},`。
- **L824 EN**: Executes a standalone statement or declaration: `}};`.
  - **L824 CN**: 执行一条独立语句或声明：`}};`。
- **L825 EN**: Blank line separating nearby declarations or logic.
  - **L825 CN**: 空行，用于分隔相邻声明或逻辑。
- **L826 EN**: Introduces template parameters or specialization context: `template <typename Real>`.
  - **L826 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real>`。
- **L827 EN**: Continues the surrounding expression or declaration: `static constexpr std::array<std::array<Real, 3>, 256> black_body_data_ = {{`.
  - **L827 CN**: 继续构造周围的表达式或声明：`static constexpr std::array<std::array<Real, 3>, 256> black_body_data_ = {{`。
- **L828 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.0, 0.0, 0.0},`.
  - **L828 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.0, 0.0, 0.0},`。
- **L829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.013038855104993618, 0.0037537033758315535, 0.002103027943341456},`.
  - **L829 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.013038855104993618, 0.0037537033758315535, 0.002103027943341456},`。
- **L830 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.02607771020998725, 0.007507406751663157, 0.004206055886683011},`.
  - **L830 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.02607771020998725, 0.007507406751663157, 0.004206055886683011},`。
- **L831 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.03911656531498074, 0.01126111012749475, 0.006309083830024466},`.
  - **L831 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.03911656531498074, 0.01126111012749475, 0.006309083830024466},`。
- **L832 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.05111281846889674, 0.0150148135033264, 0.008412111773366015},`.
  - **L832 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.05111281846889674, 0.0150148135033264, 0.008412111773366015},`。
- **L833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.06145201887409168, 0.018768516879157954, 0.010515139716707521},`.
  - **L833 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.06145201887409168, 0.018768516879157954, 0.010515139716707521},`。
- **L834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07064326337459148, 0.02252222025498951, 0.012618167660048977},`.
  - **L834 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07064326337459148, 0.02252222025498951, 0.012618167660048977},`。
- **L835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07897806577026972, 0.026275923630821117, 0.014721195603390481},`.
  - **L835 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07897806577026972, 0.026275923630821117, 0.014721195603390481},`。
- **L836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.08664361913059343, 0.03002962700665271, 0.016824223546731985},`.
  - **L836 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.08664361913059343, 0.03002962700665271, 0.016824223546731985},`。
- **L837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.09376835672821318, 0.03378333038248431, 0.018927251490073488},`.
  - **L837 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.09376835672821318, 0.03378333038248431, 0.018927251490073488},`。
- **L838 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.10044479589169861, 0.03753703375831587, 0.021030279433414945},`.
  - **L838 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.10044479589169861, 0.03753703375831587, 0.021030279433414945},`。
- **L839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.10674212293169041, 0.041271634201045536, 0.02313330737675645},`.
  - **L839 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.10674212293169041, 0.041271634201045536, 0.02313330737675645},`。
- **L840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11271363536038692, 0.04482597891806931, 0.025236335320097954},`.
  - **L840 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11271363536038692, 0.04482597891806931, 0.025236335320097954},`。

### Lines 841-868 / 第 841-868 行

````cpp
 841:     {0.11840139369546077, 0.048211428422088996, 0.027339363263439456},
 842:     {0.12383926177477003, 0.051448139039433315, 0.029442391206781018},
 843:     {0.12905496821437284, 0.05455261530914037, 0.03154541915012247},
 844:     {0.13415881215680708, 0.05748457013282512, 0.03365241563251253},
 845:     {0.1394282635051205, 0.06009480851170646, 0.035776007200922486},
 846:     {0.1448680365076732, 0.06239658846641056, 0.03791741526748158},
 847:     {0.15046463427844672, 0.06440943575351296, 0.04007701382092898},
 848:     {0.15620567729622375, 0.06614841603004337, 0.042202264001466754},
 849:     {0.1620642871793741, 0.06766725618508368, 0.04427088573012868},
 850:     {0.1679681168976293, 0.06914956019046758, 0.04626676146380417},
 851:     {0.17390879488805058, 0.07061411563078038, 0.048192562428916404},
 852:     {0.17988507705788212, 0.07206107502164819, 0.05005305250836759},
 853:     {0.18589582390636886, 0.073490579693259, 0.0518524382574397},
 854:     {0.19193998883434982, 0.07490276049381833, 0.053594454613670746},
 855:     {0.19801660795994802, 0.07629773842679596, 0.05528243440605366},
 856:     {0.20412479122741767, 0.0776756252290656, 0.05691936524751085},
 857:     {0.21026371462828286, 0.07903652389614257, 0.05850793649497276},
 858:     {0.2164326133809747, 0.08038052915995664, 0.060050578312412416},
 859:     {0.2226307759380115, 0.08170772792393452, 0.06154949439758999},
 860:     {0.22885753870905418, 0.08301819965959711, 0.0630066895818179},
 861:     {0.23511228140445872, 0.08431201676837646, 0.06442399324878009},
 862:     {0.2413944229177235, 0.08558924491193284, 0.06580307931909093},
 863:     {0.24770341767689402, 0.08684994331387272, 0.06714548339482998},
 864:     {0.25403875240487506, 0.08809416503544282, 0.06845261754062515},
 865:     {0.2603999432369943, 0.089321957227491, 0.06972578308625077},
 866:     {0.26678653315130063, 0.09053336136073056, 0.07096618176381639},
 867:     {0.2731980896731617, 0.09172841343612223, 0.07217492543577086},
 868:     {0.2796342028209079, 0.09290714417700069, 0.07335304462466913},
````
- **L841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11840139369546077, 0.048211428422088996, 0.027339363263439456},`.
  - **L841 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11840139369546077, 0.048211428422088996, 0.027339363263439456},`。
- **L842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12383926177477003, 0.051448139039433315, 0.029442391206781018},`.
  - **L842 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12383926177477003, 0.051448139039433315, 0.029442391206781018},`。
- **L843 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12905496821437284, 0.05455261530914037, 0.03154541915012247},`.
  - **L843 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12905496821437284, 0.05455261530914037, 0.03154541915012247},`。
- **L844 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13415881215680708, 0.05748457013282512, 0.03365241563251253},`.
  - **L844 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13415881215680708, 0.05748457013282512, 0.03365241563251253},`。
- **L845 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1394282635051205, 0.06009480851170646, 0.035776007200922486},`.
  - **L845 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1394282635051205, 0.06009480851170646, 0.035776007200922486},`。
- **L846 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1448680365076732, 0.06239658846641056, 0.03791741526748158},`.
  - **L846 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1448680365076732, 0.06239658846641056, 0.03791741526748158},`。
- **L847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15046463427844672, 0.06440943575351296, 0.04007701382092898},`.
  - **L847 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15046463427844672, 0.06440943575351296, 0.04007701382092898},`。
- **L848 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15620567729622375, 0.06614841603004337, 0.042202264001466754},`.
  - **L848 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15620567729622375, 0.06614841603004337, 0.042202264001466754},`。
- **L849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1620642871793741, 0.06766725618508368, 0.04427088573012868},`.
  - **L849 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1620642871793741, 0.06766725618508368, 0.04427088573012868},`。
- **L850 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1679681168976293, 0.06914956019046758, 0.04626676146380417},`.
  - **L850 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1679681168976293, 0.06914956019046758, 0.04626676146380417},`。
- **L851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.17390879488805058, 0.07061411563078038, 0.048192562428916404},`.
  - **L851 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.17390879488805058, 0.07061411563078038, 0.048192562428916404},`。
- **L852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.17988507705788212, 0.07206107502164819, 0.05005305250836759},`.
  - **L852 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.17988507705788212, 0.07206107502164819, 0.05005305250836759},`。
- **L853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.18589582390636886, 0.073490579693259, 0.0518524382574397},`.
  - **L853 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.18589582390636886, 0.073490579693259, 0.0518524382574397},`。
- **L854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.19193998883434982, 0.07490276049381833, 0.053594454613670746},`.
  - **L854 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.19193998883434982, 0.07490276049381833, 0.053594454613670746},`。
- **L855 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.19801660795994802, 0.07629773842679596, 0.05528243440605366},`.
  - **L855 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.19801660795994802, 0.07629773842679596, 0.05528243440605366},`。
- **L856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.20412479122741767, 0.0776756252290656, 0.05691936524751085},`.
  - **L856 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.20412479122741767, 0.0776756252290656, 0.05691936524751085},`。
- **L857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.21026371462828286, 0.07903652389614257, 0.05850793649497276},`.
  - **L857 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.21026371462828286, 0.07903652389614257, 0.05850793649497276},`。
- **L858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2164326133809747, 0.08038052915995664, 0.060050578312412416},`.
  - **L858 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2164326133809747, 0.08038052915995664, 0.060050578312412416},`。
- **L859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2226307759380115, 0.08170772792393452, 0.06154949439758999},`.
  - **L859 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2226307759380115, 0.08170772792393452, 0.06154949439758999},`。
- **L860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.22885753870905418, 0.08301819965959711, 0.0630066895818179},`.
  - **L860 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.22885753870905418, 0.08301819965959711, 0.0630066895818179},`。
- **L861 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.23511228140445872, 0.08431201676837646, 0.06442399324878009},`.
  - **L861 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.23511228140445872, 0.08431201676837646, 0.06442399324878009},`。
- **L862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2413944229177235, 0.08558924491193284, 0.06580307931909093},`.
  - **L862 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2413944229177235, 0.08558924491193284, 0.06580307931909093},`。
- **L863 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.24770341767689402, 0.08684994331387272, 0.06714548339482998},`.
  - **L863 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.24770341767689402, 0.08684994331387272, 0.06714548339482998},`。
- **L864 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.25403875240487506, 0.08809416503544282, 0.06845261754062515},`.
  - **L864 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.25403875240487506, 0.08809416503544282, 0.06845261754062515},`。
- **L865 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2603999432369943, 0.089321957227491, 0.06972578308625077},`.
  - **L865 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2603999432369943, 0.089321957227491, 0.06972578308625077},`。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.26678653315130063, 0.09053336136073056, 0.07096618176381639},`.
  - **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.26678653315130063, 0.09053336136073056, 0.07096618176381639},`。
- **L867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2731980896731617, 0.09172841343612223, 0.07217492543577086},`.
  - **L867 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2731980896731617, 0.09172841343612223, 0.07217492543577086},`。
- **L868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2796342028209079, 0.09290714417700069, 0.07335304462466913},`.
  - **L868 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2796342028209079, 0.09290714417700069, 0.07335304462466913},`。

### Lines 869-896 / 第 869-896 行

````cpp
 869:     {0.2860944832637109, 0.09406957920438949, 0.07450149601935155},
 870:     {0.29257856066667054, 0.09521573919680446, 0.07562116910289712},
 871:     {0.29908608220134825, 0.09634564003570609, 0.07671289202394302},
 872:     {0.3056167112027767, 0.09745929293764075, 0.07777743681356081},
 873:     {0.31217012595638344, 0.0985567045740047, 0.07881552403396175},
 874:     {0.31874601860034424, 0.09963787717926836, 0.0798278269321728},
 875:     {0.32534319363742265, 0.10070307071710113, 0.08082304674985791},
 876:     {0.3319596041219896, 0.1017528114823801, 0.08181815186765931},
 877:     {0.3385951481217959, 0.10278707282194993, 0.08281314988265284},
 878:     {0.34524972170116486, 0.10380582517735878, 0.08380805027785368},
 879:     {0.35192321947598326, 0.10480903601879557, 0.08480286230027789},
 880:     {0.35861553489350906, 0.10579666984465097, 0.08579759496848993},
 881:     {0.36532656048179657, 0.10676868817762619, 0.08679225707984467},
 882:     {0.3720561880720667, 0.10772504955741516, 0.08778685721743867},
 883:     {0.3788043089969579, 0.10866570952997373, 0.08878140375678456},
 884:     {0.38557081426725, 0.10959062063337968, 0.08977590487222206},
 885:     {0.392355594729354, 0.11049973238028288, 0.09077036854307868},
 886:     {0.3991585412055988, 0.1113929912369285, 0.09176480255959146},
 887:     {0.40597954461911684, 0.11227034059873503, 0.09275921452860061},
 888:     {0.41281849610493104, 0.1131317207623947, 0.09375361187902712},
 889:     {0.4196752871086668, 0.11397706889445536, 0.09474800186714263},
 890:     {0.4265498094741573, 0.11480631899633567, 0.0957423915816421},
 891:     {0.4334419555210749, 0.11561940186571482, 0.09673678794852814},
 892:     {0.44035161811359996, 0.11641624505422682, 0.09773119773581507},
 893:     {0.44727869072102816, 0.11719677282138305, 0.09872562755806155},
 894:     {0.45422306747112823, 0.11796090608463114, 0.09972008388073797},
 895:     {0.46118464319697156, 0.11870856236545602, 0.10071457302443804},
 896:     {0.4681633134778857, 0.11943965573140711, 0.1017091011689388},
````
- **L869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2860944832637109, 0.09406957920438949, 0.07450149601935155},`.
  - **L869 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2860944832637109, 0.09406957920438949, 0.07450149601935155},`。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.29257856066667054, 0.09521573919680446, 0.07562116910289712},`.
  - **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.29257856066667054, 0.09521573919680446, 0.07562116910289712},`。
- **L871 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.29908608220134825, 0.09634564003570609, 0.07671289202394302},`.
  - **L871 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.29908608220134825, 0.09634564003570609, 0.07671289202394302},`。
- **L872 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3056167112027767, 0.09745929293764075, 0.07777743681356081},`.
  - **L872 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3056167112027767, 0.09745929293764075, 0.07777743681356081},`。
- **L873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.31217012595638344, 0.0985567045740047, 0.07881552403396175},`.
  - **L873 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.31217012595638344, 0.0985567045740047, 0.07881552403396175},`。
- **L874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.31874601860034424, 0.09963787717926836, 0.0798278269321728},`.
  - **L874 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.31874601860034424, 0.09963787717926836, 0.0798278269321728},`。
- **L875 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.32534319363742265, 0.10070307071710113, 0.08082304674985791},`.
  - **L875 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.32534319363742265, 0.10070307071710113, 0.08082304674985791},`。
- **L876 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3319596041219896, 0.1017528114823801, 0.08181815186765931},`.
  - **L876 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3319596041219896, 0.1017528114823801, 0.08181815186765931},`。
- **L877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3385951481217959, 0.10278707282194993, 0.08281314988265284},`.
  - **L877 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3385951481217959, 0.10278707282194993, 0.08281314988265284},`。
- **L878 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.34524972170116486, 0.10380582517735878, 0.08380805027785368},`.
  - **L878 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.34524972170116486, 0.10380582517735878, 0.08380805027785368},`。
- **L879 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.35192321947598326, 0.10480903601879557, 0.08480286230027789},`.
  - **L879 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.35192321947598326, 0.10480903601879557, 0.08480286230027789},`。
- **L880 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.35861553489350906, 0.10579666984465097, 0.08579759496848993},`.
  - **L880 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.35861553489350906, 0.10579666984465097, 0.08579759496848993},`。
- **L881 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.36532656048179657, 0.10676868817762619, 0.08679225707984467},`.
  - **L881 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.36532656048179657, 0.10676868817762619, 0.08679225707984467},`。
- **L882 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3720561880720667, 0.10772504955741516, 0.08778685721743867},`.
  - **L882 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3720561880720667, 0.10772504955741516, 0.08778685721743867},`。
- **L883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3788043089969579, 0.10866570952997373, 0.08878140375678456},`.
  - **L883 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3788043089969579, 0.10866570952997373, 0.08878140375678456},`。
- **L884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.38557081426725, 0.10959062063337968, 0.08977590487222206},`.
  - **L884 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.38557081426725, 0.10959062063337968, 0.08977590487222206},`。
- **L885 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.392355594729354, 0.11049973238028288, 0.09077036854307868},`.
  - **L885 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.392355594729354, 0.11049973238028288, 0.09077036854307868},`。
- **L886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3991585412055988, 0.1113929912369285, 0.09176480255959146},`.
  - **L886 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3991585412055988, 0.1113929912369285, 0.09176480255959146},`。
- **L887 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.40597954461911684, 0.11227034059873503, 0.09275921452860061},`.
  - **L887 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.40597954461911684, 0.11227034059873503, 0.09275921452860061},`。
- **L888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.41281849610493104, 0.1131317207623947, 0.09375361187902712},`.
  - **L888 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.41281849610493104, 0.1131317207623947, 0.09375361187902712},`。
- **L889 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4196752871086668, 0.11397706889445536, 0.09474800186714263},`.
  - **L889 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4196752871086668, 0.11397706889445536, 0.09474800186714263},`。
- **L890 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4265498094741573, 0.11480631899633567, 0.0957423915816421},`.
  - **L890 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4265498094741573, 0.11480631899633567, 0.0957423915816421},`。
- **L891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4334419555210749, 0.11561940186571482, 0.09673678794852814},`.
  - **L891 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4334419555210749, 0.11561940186571482, 0.09673678794852814},`。
- **L892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.44035161811359996, 0.11641624505422682, 0.09773119773581507},`.
  - **L892 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.44035161811359996, 0.11641624505422682, 0.09773119773581507},`。
- **L893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.44727869072102816, 0.11719677282138305, 0.09872562755806155},`.
  - **L893 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.44727869072102816, 0.11719677282138305, 0.09872562755806155},`。
- **L894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.45422306747112823, 0.11796090608463114, 0.09972008388073797},`.
  - **L894 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.45422306747112823, 0.11796090608463114, 0.09972008388073797},`。
- **L895 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.46118464319697156, 0.11870856236545602, 0.10071457302443804},`.
  - **L895 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.46118464319697156, 0.11870856236545602, 0.10071457302443804},`。
- **L896 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4681633134778857, 0.11943965573140711, 0.1017091011689388},`.
  - **L896 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4681633134778857, 0.11943965573140711, 0.1017091011689388},`。

### Lines 897-924 / 第 897-924 行

````cpp
 897:     {0.47515897467511364, 0.12015409673393382, 0.10270367435711814},
 898:     {0.4821715239627041, 0.1208517923418958, 0.10369829849873419},
 899:     {0.4892008593541031, 0.12153264587059787, 0.1046929793740726},
 900:     {0.49624687972487286, 0.12219655690619557, 0.10568772263746884},
 901:     {0.5033094848319175, 0.12284342122529493, 0.10668253382070847},
 902:     {0.5103885753295632, 0.12347313070955779, 0.10767741833631286},
 903:     {0.5174840527828012, 0.12408557325511224, 0.10867238148071282},
 904:     {0.5245958196779755, 0.12468063267654717, 0.10966742843731633},
 905:     {0.5317237794311703, 0.12525818860524965, 0.11066256427947438},
 906:     {0.5388678363945218, 0.12581811638183757, 0.1116577939733483},
 907:     {0.5460278958606667, 0.12636028694240006, 0.11265312238068345},
 908:     {0.5532038640655134, 0.12688456669826084, 0.11364855426149251},
 909:     {0.5603956481895038, 0.12739081740893254, 0.11464409427665198},
 910:     {0.5676031563575221, 0.12787889604792488, 0.11563974699041571},
 911:     {0.5748262976375883, 0.12834865466103332, 0.11663551687284723},
 912:     {0.5820649820384645, 0.12879994021670815, 0.11763140830217725},
 913:     {0.5893191205062877, 0.1292325944480741, 0.11862742556708483},
 914:     {0.5965886249203336, 0.12964645368613567, 0.11962357286890879},
 915:     {0.6038734080880022, 0.13004134868367784, 0.12061985432379002},
 916:     {0.6111733837391209, 0.13041710442930957, 0.12161627396474861},
 917:     {0.618488466519628, 0.1307735399510897, 0.12261283574369633},
 918:     {0.6258185719847208, 0.13111046810909502, 0.12360954353338896},
 919:     {0.6331636165915261, 0.13142769537626633, 0.12460640112931987},
 920:     {0.6405235176913502, 0.13172502160680363, 0.125603412251556},
 921:     {0.6478981935215649, 0.13200223979132508, 0.12660058054652026},
 922:     {0.6552875631971755, 0.13225913579794255, 0.1275979095887203},
 923:     {0.6626915467021134, 0.13249548809833553, 0.1285954028824274},
 924:     {0.6701100648802979, 0.13271106747782782, 0.12959306386330607},
````
- **L897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.47515897467511364, 0.12015409673393382, 0.10270367435711814},`.
  - **L897 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.47515897467511364, 0.12015409673393382, 0.10270367435711814},`。
- **L898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4821715239627041, 0.1208517923418958, 0.10369829849873419},`.
  - **L898 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4821715239627041, 0.1208517923418958, 0.10369829849873419},`。
- **L899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4892008593541031, 0.12153264587059787, 0.1046929793740726},`.
  - **L899 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4892008593541031, 0.12153264587059787, 0.1046929793740726},`。
- **L900 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.49624687972487286, 0.12219655690619557, 0.10568772263746884},`.
  - **L900 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.49624687972487286, 0.12219655690619557, 0.10568772263746884},`。
- **L901 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5033094848319175, 0.12284342122529493, 0.10668253382070847},`.
  - **L901 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5033094848319175, 0.12284342122529493, 0.10668253382070847},`。
- **L902 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5103885753295632, 0.12347313070955779, 0.10767741833631286},`.
  - **L902 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5103885753295632, 0.12347313070955779, 0.10767741833631286},`。
- **L903 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5174840527828012, 0.12408557325511224, 0.10867238148071282},`.
  - **L903 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5174840527828012, 0.12408557325511224, 0.10867238148071282},`。
- **L904 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5245958196779755, 0.12468063267654717, 0.10966742843731633},`.
  - **L904 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5245958196779755, 0.12468063267654717, 0.10966742843731633},`。
- **L905 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5317237794311703, 0.12525818860524965, 0.11066256427947438},`.
  - **L905 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5317237794311703, 0.12525818860524965, 0.11066256427947438},`。
- **L906 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5388678363945218, 0.12581811638183757, 0.1116577939733483},`.
  - **L906 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5388678363945218, 0.12581811638183757, 0.1116577939733483},`。
- **L907 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5460278958606667, 0.12636028694240006, 0.11265312238068345},`.
  - **L907 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5460278958606667, 0.12636028694240006, 0.11265312238068345},`。
- **L908 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5532038640655134, 0.12688456669826084, 0.11364855426149251},`.
  - **L908 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5532038640655134, 0.12688456669826084, 0.11364855426149251},`。
- **L909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5603956481895038, 0.12739081740893254, 0.11464409427665198},`.
  - **L909 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5603956481895038, 0.12739081740893254, 0.11464409427665198},`。
- **L910 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5676031563575221, 0.12787889604792488, 0.11563974699041571},`.
  - **L910 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5676031563575221, 0.12787889604792488, 0.11563974699041571},`。
- **L911 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5748262976375883, 0.12834865466103332, 0.11663551687284723},`.
  - **L911 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5748262976375883, 0.12834865466103332, 0.11663551687284723},`。
- **L912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5820649820384645, 0.12879994021670815, 0.11763140830217725},`.
  - **L912 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5820649820384645, 0.12879994021670815, 0.11763140830217725},`。
- **L913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5893191205062877, 0.1292325944480741, 0.11862742556708483},`.
  - **L913 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5893191205062877, 0.1292325944480741, 0.11862742556708483},`。
- **L914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5965886249203336, 0.12964645368613567, 0.11962357286890879},`.
  - **L914 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5965886249203336, 0.12964645368613567, 0.11962357286890879},`。
- **L915 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6038734080880022, 0.13004134868367784, 0.12061985432379002},`.
  - **L915 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6038734080880022, 0.13004134868367784, 0.12061985432379002},`。
- **L916 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6111733837391209, 0.13041710442930957, 0.12161627396474861},`.
  - **L916 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6111733837391209, 0.13041710442930957, 0.12161627396474861},`。
- **L917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.618488466519628, 0.1307735399510897, 0.12261283574369633},`.
  - **L917 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.618488466519628, 0.1307735399510897, 0.12261283574369633},`。
- **L918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6258185719847208, 0.13111046810909502, 0.12360954353338896},`.
  - **L918 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6258185719847208, 0.13111046810909502, 0.12360954353338896},`。
- **L919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6331636165915261, 0.13142769537626633, 0.12460640112931987},`.
  - **L919 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6331636165915261, 0.13142769537626633, 0.12460640112931987},`。
- **L920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6405235176913502, 0.13172502160680363, 0.125603412251556},`.
  - **L920 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6405235176913502, 0.13172502160680363, 0.125603412251556},`。
- **L921 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6478981935215649, 0.13200223979132508, 0.12660058054652026},`.
  - **L921 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6478981935215649, 0.13200223979132508, 0.12660058054652026},`。
- **L922 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6552875631971755, 0.13225913579794255, 0.1275979095887203},`.
  - **L922 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6552875631971755, 0.13225913579794255, 0.1275979095887203},`。
- **L923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6626915467021134, 0.13249548809833553, 0.1285954028824274},`.
  - **L923 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6626915467021134, 0.13249548809833553, 0.1285954028824274},`。
- **L924 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6701100648802979, 0.13271106747782782, 0.12959306386330607},`.
  - **L924 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6701100648802979, 0.13271106747782782, 0.12959306386330607},`。

### Lines 925-952 / 第 925-952 行

````cpp
 925:     {0.6775430394264935, 0.13290563672840347, 0.13059089589999584},
 926:     {0.6849903928770076, 0.133078950323475, 0.13158890229564854},
 927:     {0.6924520486002461, 0.13323075407316545, 0.13258708628942104},
 928:     {0.6990269444289148, 0.13526285713010114, 0.13312878204163103},
 929:     {0.7029385031056067, 0.14276164252353424, 0.13230087174729266},
 930:     {0.706849160518104, 0.15003367886528432, 0.13144472110159658},
 931:     {0.7107589339135043, 0.1571072636984418, 0.13055959687730143},
 932:     {0.7146678402982333, 0.16400574341929422, 0.12964472942353197},
 933:     {0.7185758964424839, 0.17074862440458652, 0.1286993099989257},
 934:     {0.7224831188845541, 0.1773523851612915, 0.12772248784920406},
 935:     {0.7263895239350789, 0.18383108192179715, 0.12671336699848523},
 936:     {0.7302951276811657, 0.19019680820853718, 0.12567100271919956},
 937:     {0.7341999459904341, 0.19646004903982417, 0.12459439764026739},
 938:     {0.7381039945149617, 0.20262995773846548, 0.12348249744706852},
 939:     {0.742007288695138, 0.20871457496055673, 0.12233418611951721},
 940:     {0.7459098437634323, 0.21472100396010463, 0.1211482806460144},
 941:     {0.7498116747480748, 0.22065555226824157, 0.11992352514088764},
 942:     {0.7537127964766552, 0.22652384728976455, 0.11865858428082418},
 943:     {0.7576132235796376, 0.23233093142236372, 0.11735203596128549},
 944:     {0.7615129704938024, 0.2380813409383808, 0.11600236305643169},
 945:     {0.7654120514656042, 0.24377917187260056, 0.11460794414495701},
 946:     {0.7693104805544623, 0.24942813542336495, 0.11316704303859387},
 947:     {0.7732082716359728, 0.25503160482399895, 0.11167779691868823},
 948:     {0.7771054384050546, 0.260592655225706, 0.11013820284777173},
 949:     {0.7810019943790253, 0.266114097815743, 0.1085461023755086},
 950:     {0.784897952900607, 0.27159850915024936, 0.10689916389931109},
 951:     {0.7887933271408747, 0.2770482564911545, 0.10519486236601067},
 952:     {0.792688130102133, 0.28246551978783785, 0.10343045580787474},
````
- **L925 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6775430394264935, 0.13290563672840347, 0.13059089589999584},`.
  - **L925 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6775430394264935, 0.13290563672840347, 0.13059089589999584},`。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6849903928770076, 0.133078950323475, 0.13158890229564854},`.
  - **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6849903928770076, 0.133078950323475, 0.13158890229564854},`。
- **L927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6924520486002461, 0.13323075407316545, 0.13258708628942104},`.
  - **L927 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6924520486002461, 0.13323075407316545, 0.13258708628942104},`。
- **L928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6990269444289148, 0.13526285713010114, 0.13312878204163103},`.
  - **L928 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6990269444289148, 0.13526285713010114, 0.13312878204163103},`。
- **L929 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7029385031056067, 0.14276164252353424, 0.13230087174729266},`.
  - **L929 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7029385031056067, 0.14276164252353424, 0.13230087174729266},`。
- **L930 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.706849160518104, 0.15003367886528432, 0.13144472110159658},`.
  - **L930 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.706849160518104, 0.15003367886528432, 0.13144472110159658},`。
- **L931 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7107589339135043, 0.1571072636984418, 0.13055959687730143},`.
  - **L931 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7107589339135043, 0.1571072636984418, 0.13055959687730143},`。
- **L932 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7146678402982333, 0.16400574341929422, 0.12964472942353197},`.
  - **L932 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7146678402982333, 0.16400574341929422, 0.12964472942353197},`。
- **L933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7185758964424839, 0.17074862440458652, 0.1286993099989257},`.
  - **L933 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7185758964424839, 0.17074862440458652, 0.1286993099989257},`。
- **L934 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7224831188845541, 0.1773523851612915, 0.12772248784920406},`.
  - **L934 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7224831188845541, 0.1773523851612915, 0.12772248784920406},`。
- **L935 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7263895239350789, 0.18383108192179715, 0.12671336699848523},`.
  - **L935 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7263895239350789, 0.18383108192179715, 0.12671336699848523},`。
- **L936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7302951276811657, 0.19019680820853718, 0.12567100271919956},`.
  - **L936 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7302951276811657, 0.19019680820853718, 0.12567100271919956},`。
- **L937 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7341999459904341, 0.19646004903982417, 0.12459439764026739},`.
  - **L937 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7341999459904341, 0.19646004903982417, 0.12459439764026739},`。
- **L938 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7381039945149617, 0.20262995773846548, 0.12348249744706852},`.
  - **L938 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7381039945149617, 0.20262995773846548, 0.12348249744706852},`。
- **L939 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.742007288695138, 0.20871457496055673, 0.12233418611951721},`.
  - **L939 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.742007288695138, 0.20871457496055673, 0.12233418611951721},`。
- **L940 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7459098437634323, 0.21472100396010463, 0.1211482806460144},`.
  - **L940 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7459098437634323, 0.21472100396010463, 0.1211482806460144},`。
- **L941 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7498116747480748, 0.22065555226824157, 0.11992352514088764},`.
  - **L941 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7498116747480748, 0.22065555226824157, 0.11992352514088764},`。
- **L942 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7537127964766552, 0.22652384728976455, 0.11865858428082418},`.
  - **L942 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7537127964766552, 0.22652384728976455, 0.11865858428082418},`。
- **L943 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7576132235796376, 0.23233093142236372, 0.11735203596128549},`.
  - **L943 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7576132235796376, 0.23233093142236372, 0.11735203596128549},`。
- **L944 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7615129704938024, 0.2380813409383808, 0.11600236305643169},`.
  - **L944 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7615129704938024, 0.2380813409383808, 0.11600236305643169},`。
- **L945 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7654120514656042, 0.24377917187260056, 0.11460794414495701},`.
  - **L945 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7654120514656042, 0.24377917187260056, 0.11460794414495701},`。
- **L946 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7693104805544623, 0.24942813542336495, 0.11316704303859387},`.
  - **L946 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7693104805544623, 0.24942813542336495, 0.11316704303859387},`。
- **L947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7732082716359728, 0.25503160482399895, 0.11167779691868823},`.
  - **L947 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7732082716359728, 0.25503160482399895, 0.11167779691868823},`。
- **L948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7771054384050546, 0.260592655225706, 0.11013820284777173},`.
  - **L948 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7771054384050546, 0.260592655225706, 0.11013820284777173},`。
- **L949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7810019943790253, 0.266114097815743, 0.1085461023755086},`.
  - **L949 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7810019943790253, 0.266114097815743, 0.1085461023755086},`。
- **L950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.784897952900607, 0.27159850915024936, 0.10689916389931109},`.
  - **L950 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.784897952900607, 0.27159850915024936, 0.10689916389931109},`。
- **L951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7887933271408747, 0.2770482564911545, 0.10519486236601067},`.
  - **L951 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7887933271408747, 0.2770482564911545, 0.10519486236601067},`。
- **L952 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.792688130102133, 0.28246551978783785, 0.10343045580787474},`.
  - **L952 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.792688130102133, 0.28246551978783785, 0.10343045580787474},`。

### Lines 953-980 / 第 953-980 行

````cpp
 953:     {0.7965823746207359, 0.28785231082677337, 0.1016029580881492},
 954:     {0.8004760733698467, 0.2932104899790604, 0.09970910708026909},
 955:     {0.8043692388621376, 0.2985417809010483, 0.09774532731012447},
 956:     {0.8082618834524321, 0.30384778348310415, 0.09570768583739653},
 957:     {0.8121540193402956, 0.30912998529284585, 0.09359183981917277},
 958:     {0.8160456585725648, 0.31438977171944593, 0.09139297375735872},
 959:     {0.8199368130458352, 0.31962843499310634, 0.08910572383870971},
 960:     {0.8238274945088833, 0.32484718222701414, 0.0867240859712489},
 961:     {0.8277177145650534, 0.33004714260695417, 0.08424130301309385},
 962:     {0.8316074846745842, 0.33522937383533186, 0.08164972514342614},
 963:     {0.8354968161568933, 0.3403948679210186, 0.07894063513317165},
 964:     {0.8393857201928158, 0.3455445563935293, 0.07610402711123557},
 965:     {0.8432742078267932, 0.350679315009233, 0.07312832277504416},
 966:     {0.8471622899690239, 0.35579996800812036, 0.07000000201860399},
 967:     {0.8510499773975676, 0.3609072919719153, 0.06670311423184544},
 968:     {0.8549372807604048, 0.3660020193276998, 0.06321861961083997},
 969:     {0.8588242105774635, 0.37108484153559657, 0.05952348231757188},
 970:     {0.8627107772425975, 0.37615641199422883, 0.05558939105319657},
 971:     {0.8665969910255304, 0.38121734869353224, 0.0513809015925008},
 972:     {0.870482862073761, 0.3862682366409203, 0.046852647240651586},
 973:     {0.8743684004144292, 0.39130963008373454, 0.04194497518373403},
 974:     {0.8782536159561499, 0.39634205454821875, 0.03662194288964867},
 975:     {0.8821385184908072, 0.40136600871296174, 0.03117410153953512},
 976:     {0.8860231176953159, 0.40638196613271077, 0.025643215104659372},
 977:     {0.8899074231333507, 0.4113903768267081, 0.020028781568071516},
 978:     {0.891065382338744, 0.4184635684621956, 0.021466471088552607},
 979:     {0.8919871230973406, 0.4256470552949243, 0.02354603033215587},
 980:     {0.8928908994946323, 0.4327756220188152, 0.02570095483314324},
````
- **L953 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7965823746207359, 0.28785231082677337, 0.1016029580881492},`.
  - **L953 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7965823746207359, 0.28785231082677337, 0.1016029580881492},`。
- **L954 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8004760733698467, 0.2932104899790604, 0.09970910708026909},`.
  - **L954 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8004760733698467, 0.2932104899790604, 0.09970910708026909},`。
- **L955 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8043692388621376, 0.2985417809010483, 0.09774532731012447},`.
  - **L955 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8043692388621376, 0.2985417809010483, 0.09774532731012447},`。
- **L956 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8082618834524321, 0.30384778348310415, 0.09570768583739653},`.
  - **L956 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8082618834524321, 0.30384778348310415, 0.09570768583739653},`。
- **L957 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8121540193402956, 0.30912998529284585, 0.09359183981917277},`.
  - **L957 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8121540193402956, 0.30912998529284585, 0.09359183981917277},`。
- **L958 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8160456585725648, 0.31438977171944593, 0.09139297375735872},`.
  - **L958 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8160456585725648, 0.31438977171944593, 0.09139297375735872},`。
- **L959 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8199368130458352, 0.31962843499310634, 0.08910572383870971},`.
  - **L959 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8199368130458352, 0.31962843499310634, 0.08910572383870971},`。
- **L960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8238274945088833, 0.32484718222701414, 0.0867240859712489},`.
  - **L960 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8238274945088833, 0.32484718222701414, 0.0867240859712489},`。
- **L961 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8277177145650534, 0.33004714260695417, 0.08424130301309385},`.
  - **L961 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8277177145650534, 0.33004714260695417, 0.08424130301309385},`。
- **L962 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8316074846745842, 0.33522937383533186, 0.08164972514342614},`.
  - **L962 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8316074846745842, 0.33522937383533186, 0.08164972514342614},`。
- **L963 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8354968161568933, 0.3403948679210186, 0.07894063513317165},`.
  - **L963 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8354968161568933, 0.3403948679210186, 0.07894063513317165},`。
- **L964 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8393857201928158, 0.3455445563935293, 0.07610402711123557},`.
  - **L964 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8393857201928158, 0.3455445563935293, 0.07610402711123557},`。
- **L965 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8432742078267932, 0.350679315009233, 0.07312832277504416},`.
  - **L965 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8432742078267932, 0.350679315009233, 0.07312832277504416},`。
- **L966 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8471622899690239, 0.35579996800812036, 0.07000000201860399},`.
  - **L966 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8471622899690239, 0.35579996800812036, 0.07000000201860399},`。
- **L967 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8510499773975676, 0.3609072919719153, 0.06670311423184544},`.
  - **L967 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8510499773975676, 0.3609072919719153, 0.06670311423184544},`。
- **L968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8549372807604048, 0.3660020193276998, 0.06321861961083997},`.
  - **L968 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8549372807604048, 0.3660020193276998, 0.06321861961083997},`。
- **L969 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8588242105774635, 0.37108484153559657, 0.05952348231757188},`.
  - **L969 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8588242105774635, 0.37108484153559657, 0.05952348231757188},`。
- **L970 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8627107772425975, 0.37615641199422883, 0.05558939105319657},`.
  - **L970 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8627107772425975, 0.37615641199422883, 0.05558939105319657},`。
- **L971 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8665969910255304, 0.38121734869353224, 0.0513809015925008},`.
  - **L971 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8665969910255304, 0.38121734869353224, 0.0513809015925008},`。
- **L972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.870482862073761, 0.3862682366409203, 0.046852647240651586},`.
  - **L972 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.870482862073761, 0.3862682366409203, 0.046852647240651586},`。
- **L973 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8743684004144292, 0.39130963008373454, 0.04194497518373403},`.
  - **L973 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8743684004144292, 0.39130963008373454, 0.04194497518373403},`。
- **L974 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8782536159561499, 0.39634205454821875, 0.03662194288964867},`.
  - **L974 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8782536159561499, 0.39634205454821875, 0.03662194288964867},`。
- **L975 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8821385184908072, 0.40136600871296174, 0.03117410153953512},`.
  - **L975 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8821385184908072, 0.40136600871296174, 0.03117410153953512},`。
- **L976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8860231176953159, 0.40638196613271077, 0.025643215104659372},`.
  - **L976 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8860231176953159, 0.40638196613271077, 0.025643215104659372},`。
- **L977 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8899074231333507, 0.4113903768267081, 0.020028781568071516},`.
  - **L977 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8899074231333507, 0.4113903768267081, 0.020028781568071516},`。
- **L978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.891065382338744, 0.4184635684621956, 0.021466471088552607},`.
  - **L978 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.891065382338744, 0.4184635684621956, 0.021466471088552607},`。
- **L979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8919871230973406, 0.4256470552949243, 0.02354603033215587},`.
  - **L979 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8919871230973406, 0.4256470552949243, 0.02354603033215587},`。
- **L980 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8928908994946323, 0.4327756220188152, 0.02570095483314324},`.
  - **L980 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8928908994946323, 0.4327756220188152, 0.02570095483314324},`。

### Lines 981-1008 / 第 981-1008 行

````cpp
 981:     {0.8937766259341077, 0.4398522418561916, 0.027932126084511257},
 982:     {0.8946442158848376, 0.4468796753940895, 0.030240425579256377},
 983:     {0.8954935818622007, 0.4538604904909103, 0.032626734810374966},
 984:     {0.8963246354082741, 0.4607970798827202, 0.03509193527086419},
 985:     {0.8971372870718866, 0.4676916768037213, 0.037636908453718976},
 986:     {0.8979314463883195, 0.4745463688860316, 0.04026253585193785},
 987:     {0.8987070218586405, 0.48136311056322806, 0.042883774659142816},
 988:     {0.899463920928664, 0.4881437341684682, 0.045483032991513316},
 989:     {0.9002020499675142, 0.49488995989000545, 0.048065003458762835},
 990:     {0.9009213142457936, 0.5016034047235632, 0.05063130309097131},
 991:     {0.9016216179133228, 0.5082855905414284, 0.053183371551623206},
 992:     {0.9023028639764596, 0.5149379513816515, 0.055722495847998925},
 993:     {0.902964954274961, 0.5215618400467861, 0.05824983088146562},
 994:     {0.9036077894583927, 0.5281585340897971, 0.060766416652818796},
 995:     {0.9042312689620561, 0.5347292412547051, 0.06327319275680962},
 996:     {0.9048352909824239, 0.5412751044309391, 0.06577101066369245},
 997:     {0.9054197524520664, 0.5477972061730271, 0.0682606441818552},
 998:     {0.9059845490140521, 0.5542965728309169, 0.07074279841591524},
 999:     {0.9065295749958051, 0.5607741783307898, 0.07321811747290291},
1000:     {0.9070547233823978, 0.5672309476415033, 0.07568719112093336},
1001:     {0.9075598857892664, 0.5736677599577373, 0.07815056056678477},
1002:     {0.9080449524343264, 0.5800854516273583, 0.08060872348871567},
1003:     {0.9085098121094662, 0.5864848188474415, 0.083062138436825},
1004:     {0.9089543521514049, 0.5928666201506813, 0.08551122869395478},
1005:     {0.9093784584118849, 0.5992315787015761, 0.08795638567456415},
1006:     {0.909782015227184, 0.6055803844196885, 0.09039797192630794},
1007:     {0.9101649053869216, 0.6119136959454732, 0.09283632378872866},
1008:     {0.9105270101021349, 0.6182321424625531, 0.09527175375494526},
````
- **L981 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8937766259341077, 0.4398522418561916, 0.027932126084511257},`.
  - **L981 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8937766259341077, 0.4398522418561916, 0.027932126084511257},`。
- **L982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8946442158848376, 0.4468796753940895, 0.030240425579256377},`.
  - **L982 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8946442158848376, 0.4468796753940895, 0.030240425579256377},`。
- **L983 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8954935818622007, 0.4538604904909103, 0.032626734810374966},`.
  - **L983 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8954935818622007, 0.4538604904909103, 0.032626734810374966},`。
- **L984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8963246354082741, 0.4607970798827202, 0.03509193527086419},`.
  - **L984 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8963246354082741, 0.4607970798827202, 0.03509193527086419},`。
- **L985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8971372870718866, 0.4676916768037213, 0.037636908453718976},`.
  - **L985 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8971372870718866, 0.4676916768037213, 0.037636908453718976},`。
- **L986 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8979314463883195, 0.4745463688860316, 0.04026253585193785},`.
  - **L986 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8979314463883195, 0.4745463688860316, 0.04026253585193785},`。
- **L987 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8987070218586405, 0.48136311056322806, 0.042883774659142816},`.
  - **L987 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8987070218586405, 0.48136311056322806, 0.042883774659142816},`。
- **L988 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.899463920928664, 0.4881437341684682, 0.045483032991513316},`.
  - **L988 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.899463920928664, 0.4881437341684682, 0.045483032991513316},`。
- **L989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9002020499675142, 0.49488995989000545, 0.048065003458762835},`.
  - **L989 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9002020499675142, 0.49488995989000545, 0.048065003458762835},`。
- **L990 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9009213142457936, 0.5016034047235632, 0.05063130309097131},`.
  - **L990 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9009213142457936, 0.5016034047235632, 0.05063130309097131},`。
- **L991 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9016216179133228, 0.5082855905414284, 0.053183371551623206},`.
  - **L991 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9016216179133228, 0.5082855905414284, 0.053183371551623206},`。
- **L992 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9023028639764596, 0.5149379513816515, 0.055722495847998925},`.
  - **L992 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9023028639764596, 0.5149379513816515, 0.055722495847998925},`。
- **L993 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.902964954274961, 0.5215618400467861, 0.05824983088146562},`.
  - **L993 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.902964954274961, 0.5215618400467861, 0.05824983088146562},`。
- **L994 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9036077894583927, 0.5281585340897971, 0.060766416652818796},`.
  - **L994 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9036077894583927, 0.5281585340897971, 0.060766416652818796},`。
- **L995 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9042312689620561, 0.5347292412547051, 0.06327319275680962},`.
  - **L995 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9042312689620561, 0.5347292412547051, 0.06327319275680962},`。
- **L996 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9048352909824239, 0.5412751044309391, 0.06577101066369245},`.
  - **L996 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9048352909824239, 0.5412751044309391, 0.06577101066369245},`。
- **L997 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9054197524520664, 0.5477972061730271, 0.0682606441818552},`.
  - **L997 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9054197524520664, 0.5477972061730271, 0.0682606441818552},`。
- **L998 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9059845490140521, 0.5542965728309169, 0.07074279841591524},`.
  - **L998 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9059845490140521, 0.5542965728309169, 0.07074279841591524},`。
- **L999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9065295749958051, 0.5607741783307898, 0.07321811747290291},`.
  - **L999 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9065295749958051, 0.5607741783307898, 0.07321811747290291},`。
- **L1000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9070547233823978, 0.5672309476415033, 0.07568719112093336},`.
  - **L1000 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9070547233823978, 0.5672309476415033, 0.07568719112093336},`。
- **L1001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9075598857892664, 0.5736677599577373, 0.07815056056678477},`.
  - **L1001 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9075598857892664, 0.5736677599577373, 0.07815056056678477},`。
- **L1002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9080449524343264, 0.5800854516273583, 0.08060872348871567},`.
  - **L1002 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9080449524343264, 0.5800854516273583, 0.08060872348871567},`。
- **L1003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9085098121094662, 0.5864848188474415, 0.083062138436825},`.
  - **L1003 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9085098121094662, 0.5864848188474415, 0.083062138436825},`。
- **L1004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9089543521514049, 0.5928666201506813, 0.08551122869395478},`.
  - **L1004 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9089543521514049, 0.5928666201506813, 0.08551122869395478},`。
- **L1005 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9093784584118849, 0.5992315787015761, 0.08795638567456415},`.
  - **L1005 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9093784584118849, 0.5992315787015761, 0.08795638567456415},`。
- **L1006 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.909782015227184, 0.6055803844196885, 0.09039797192630794},`.
  - **L1006 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.909782015227184, 0.6055803844196885, 0.09039797192630794},`。
- **L1007 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9101649053869216, 0.6119136959454732, 0.09283632378872866},`.
  - **L1007 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9101649053869216, 0.6119136959454732, 0.09283632378872866},`。
- **L1008 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9105270101021349, 0.6182321424625531, 0.09527175375494526},`.
  - **L1008 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9105270101021349, 0.6182321424625531, 0.09527175375494526},`。

### Lines 1009-1036 / 第 1009-1036 行

````cpp
1009:     {0.9108682089726016, 0.6245363253889187, 0.09770455257523358},
1010:     {0.9111883799533859, 0.630826819948256, 0.10013499113555632},
1011:     {0.9114873993205763, 0.6371041766315243, 0.10256332213928446},
1012:     {0.9117651416361977, 0.6433689225578914, 0.10498978161628147},
1013:     {0.9120214797122554, 0.649621562743278, 0.10741459028014497},
1014:     {0.912256284573901, 0.6558625812839713, 0.1098379547515301},
1015:     {0.91246942542167, 0.6620924424620748, 0.11226006866305718},
1016:     {0.9126607695927773, 0.6683115917789337, 0.11468111365925662},
1017:     {0.9128301825214254, 0.6745204569221216, 0.11710126030325854},
1018:     {0.9129775276981008, 0.6807194486710708, 0.11952066890043198},
1019:     {0.9131026666278179, 0.6869089617459752, 0.12193949024790668},
1020:     {0.9132054587872768, 0.6930893756041974, 0.12435786631781182},
1021:     {0.9132857615808976, 0.6992610551880414, 0.1267759308811086},
1022:     {0.9133434302956924, 0.7054243516274225, 0.12919381007809072},
1023:     {0.9133783180549305, 0.7115796029006738, 0.13161162294090045},
1024:     {0.9133902757705612, 0.7177271344564569, 0.13402948187280572},
1025:     {0.9133791520943453, 0.7238672597994953, 0.13644749308843845},
1026:     {0.913344793367654, 0.7300002810426419, 0.13886575701873277},
1027:     {0.9132870435698786, 0.7361264894275755, 0.1412843686838794},
1028:     {0.9132057442654163, 0.7422461658162536, 0.1437034180372798},
1029:     {0.913100734549166, 0.7483595811550647, 0.1461229902831289},
1030:     {0.9129718509904932, 0.7544669969134981, 0.14854316617002308},
1031:     {0.9128189275755945, 0.7605686654989812, 0.15096402226269828},
1032:     {0.9126417956482176, 0.7666648306494336, 0.15338563119383095},
1033:     {0.9124402838486693, 0.7727557278049575, 0.1558080618976074},
1034:     {0.9122142180510416, 0.778841584459982, 0.15823137982661514},
1035:     {0.9119634212986102, 0.7849226204970876, 0.16065564715346278},
1036:     {0.9116877137373088, 0.7909990485036371, 0.16308092295837506},
````
- **L1009 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9108682089726016, 0.6245363253889187, 0.09770455257523358},`.
  - **L1009 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9108682089726016, 0.6245363253889187, 0.09770455257523358},`。
- **L1010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9111883799533859, 0.630826819948256, 0.10013499113555632},`.
  - **L1010 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9111883799533859, 0.630826819948256, 0.10013499113555632},`。
- **L1011 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9114873993205763, 0.6371041766315243, 0.10256332213928446},`.
  - **L1011 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9114873993205763, 0.6371041766315243, 0.10256332213928446},`。
- **L1012 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9117651416361977, 0.6433689225578914, 0.10498978161628147},`.
  - **L1012 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9117651416361977, 0.6433689225578914, 0.10498978161628147},`。
- **L1013 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9120214797122554, 0.649621562743278, 0.10741459028014497},`.
  - **L1013 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9120214797122554, 0.649621562743278, 0.10741459028014497},`。
- **L1014 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.912256284573901, 0.6558625812839713, 0.1098379547515301},`.
  - **L1014 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.912256284573901, 0.6558625812839713, 0.1098379547515301},`。
- **L1015 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.91246942542167, 0.6620924424620748, 0.11226006866305718},`.
  - **L1015 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.91246942542167, 0.6620924424620748, 0.11226006866305718},`。
- **L1016 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9126607695927773, 0.6683115917789337, 0.11468111365925662},`.
  - **L1016 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9126607695927773, 0.6683115917789337, 0.11468111365925662},`。
- **L1017 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9128301825214254, 0.6745204569221216, 0.11710126030325854},`.
  - **L1017 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9128301825214254, 0.6745204569221216, 0.11710126030325854},`。
- **L1018 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9129775276981008, 0.6807194486710708, 0.11952066890043198},`.
  - **L1018 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9129775276981008, 0.6807194486710708, 0.11952066890043198},`。
- **L1019 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9131026666278179, 0.6869089617459752, 0.12193949024790668},`.
  - **L1019 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9131026666278179, 0.6869089617459752, 0.12193949024790668},`。
- **L1020 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9132054587872768, 0.6930893756041974, 0.12435786631781182},`.
  - **L1020 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9132054587872768, 0.6930893756041974, 0.12435786631781182},`。
- **L1021 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9132857615808976, 0.6992610551880414, 0.1267759308811086},`.
  - **L1021 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9132857615808976, 0.6992610551880414, 0.1267759308811086},`。
- **L1022 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9133434302956924, 0.7054243516274225, 0.12919381007809072},`.
  - **L1022 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9133434302956924, 0.7054243516274225, 0.12919381007809072},`。
- **L1023 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9133783180549305, 0.7115796029006738, 0.13161162294090045},`.
  - **L1023 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9133783180549305, 0.7115796029006738, 0.13161162294090045},`。
- **L1024 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9133902757705612, 0.7177271344564569, 0.13402948187280572},`.
  - **L1024 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9133902757705612, 0.7177271344564569, 0.13402948187280572},`。
- **L1025 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9133791520943453, 0.7238672597994953, 0.13644749308843845},`.
  - **L1025 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9133791520943453, 0.7238672597994953, 0.13644749308843845},`。
- **L1026 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.913344793367654, 0.7300002810426419, 0.13886575701873277},`.
  - **L1026 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.913344793367654, 0.7300002810426419, 0.13886575701873277},`。
- **L1027 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9132870435698786, 0.7361264894275755, 0.1412843686838794},`.
  - **L1027 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9132870435698786, 0.7361264894275755, 0.1412843686838794},`。
- **L1028 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9132057442654163, 0.7422461658162536, 0.1437034180372798},`.
  - **L1028 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9132057442654163, 0.7422461658162536, 0.1437034180372798},`。
- **L1029 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.913100734549166, 0.7483595811550647, 0.1461229902831289},`.
  - **L1029 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.913100734549166, 0.7483595811550647, 0.1461229902831289},`。
- **L1030 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9129718509904932, 0.7544669969134981, 0.14854316617002308},`.
  - **L1030 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9129718509904932, 0.7544669969134981, 0.14854316617002308},`。
- **L1031 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9128189275755945, 0.7605686654989812, 0.15096402226269828},`.
  - **L1031 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9128189275755945, 0.7605686654989812, 0.15096402226269828},`。
- **L1032 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9126417956482176, 0.7666648306494336, 0.15338563119383095},`.
  - **L1032 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9126417956482176, 0.7666648306494336, 0.15338563119383095},`。
- **L1033 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9124402838486693, 0.7727557278049575, 0.1558080618976074},`.
  - **L1033 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9124402838486693, 0.7727557278049575, 0.1558080618976074},`。
- **L1034 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9122142180510416, 0.778841584459982, 0.15823137982661514},`.
  - **L1034 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9122142180510416, 0.778841584459982, 0.15823137982661514},`。
- **L1035 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9119634212986102, 0.7849226204970876, 0.16065564715346278},`.
  - **L1035 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9119634212986102, 0.7849226204970876, 0.16065564715346278},`。
- **L1036 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9116877137373088, 0.7909990485036371, 0.16308092295837506},`.
  - **L1036 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9116877137373088, 0.7909990485036371, 0.16308092295837506},`。

### Lines 1037-1064 / 第 1037-1064 行

````cpp
1037:     {0.9113869125472361, 0.797071074072276, 0.16550726340392827},
1038:     {0.9110608318720959, 0.803138896086265, 0.16793472189794498},
1039:     {0.9107092827465118, 0.8092027069905707, 0.1703633492455004},
1040:     {0.9103320730211178, 0.8152626930495434, 0.17279319379088973},
1041:     {0.909929007285357, 0.8213190345919861, 0.17522430155033844},
1042:     {0.9094998867878827, 0.8273719062443313, 0.17765671633615956},
1043:     {0.9090445093544782, 0.8334214771526294, 0.1800904798730111},
1044:     {0.9085626693033965, 0.8394679111939684, 0.18252563190683574},
1045:     {0.9080541573580172, 0.8455113671779366, 0.18496221030703314},
1046:     {0.9075187605567102, 0.8515519990386777, 0.18740025116234627},
1047:     {0.9069562621598014, 0.8575899560180628, 0.18983978887092715},
1048:     {0.9063664415535179, 0.8636253828404642, 0.19228085622498767},
1049:     {0.9057490741507888, 0.8696584198795938, 0.19472348449042068},
1050:     {0.9051039312887829, 0.875689203317828, 0.197167703481744},
1051:     {0.9044307801230321, 0.8817178652984231, 0.19961354163268363},
1052:     {0.9037293835180177, 0.8877445340709991, 0.2020610260626995},
1053:     {0.9029994999340526, 0.8937693341306414, 0.20451018263972115},
1054:     {0.9022408833103207, 0.8997923863509576, 0.2069610360393544},
1055:     {0.9059888943947992, 0.9040221289848984, 0.2345476432758713},
1056:     {0.9121353668077321, 0.9072538390291374, 0.2723959319742808},
1057:     {0.918103004002936, 0.910496798148879, 0.30707485069273716},
1058:     {0.9238889872583352, 0.913751436869596, 0.3395892760133458},
1059:     {0.9294904738341414, 0.9170181766931385, 0.3705416931634215},
1060:     {0.9349045893736176, 0.9202974302069179, 0.4003237614900562},
1061:     {0.9401284204049521, 0.9235896011902338, 0.4292051255044417},
1062:     {0.9451590068972918, 0.9268950847179593, 0.45737955986332374},
1063:     {0.9499933348250024, 0.9302142672617891, 0.4849910259595919},
1064:     {0.9546283286947478, 0.9335475267892457, 0.5121493527958956},
````
- **L1037 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9113869125472361, 0.797071074072276, 0.16550726340392827},`.
  - **L1037 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9113869125472361, 0.797071074072276, 0.16550726340392827},`。
- **L1038 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9110608318720959, 0.803138896086265, 0.16793472189794498},`.
  - **L1038 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9110608318720959, 0.803138896086265, 0.16793472189794498},`。
- **L1039 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9107092827465118, 0.8092027069905707, 0.1703633492455004},`.
  - **L1039 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9107092827465118, 0.8092027069905707, 0.1703633492455004},`。
- **L1040 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9103320730211178, 0.8152626930495434, 0.17279319379088973},`.
  - **L1040 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9103320730211178, 0.8152626930495434, 0.17279319379088973},`。
- **L1041 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.909929007285357, 0.8213190345919861, 0.17522430155033844},`.
  - **L1041 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.909929007285357, 0.8213190345919861, 0.17522430155033844},`。
- **L1042 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9094998867878827, 0.8273719062443313, 0.17765671633615956},`.
  - **L1042 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9094998867878827, 0.8273719062443313, 0.17765671633615956},`。
- **L1043 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9090445093544782, 0.8334214771526294, 0.1800904798730111},`.
  - **L1043 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9090445093544782, 0.8334214771526294, 0.1800904798730111},`。
- **L1044 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9085626693033965, 0.8394679111939684, 0.18252563190683574},`.
  - **L1044 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9085626693033965, 0.8394679111939684, 0.18252563190683574},`。
- **L1045 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9080541573580172, 0.8455113671779366, 0.18496221030703314},`.
  - **L1045 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9080541573580172, 0.8455113671779366, 0.18496221030703314},`。
- **L1046 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9075187605567102, 0.8515519990386777, 0.18740025116234627},`.
  - **L1046 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9075187605567102, 0.8515519990386777, 0.18740025116234627},`。
- **L1047 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9069562621598014, 0.8575899560180628, 0.18983978887092715},`.
  - **L1047 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9069562621598014, 0.8575899560180628, 0.18983978887092715},`。
- **L1048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9063664415535179, 0.8636253828404642, 0.19228085622498767},`.
  - **L1048 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9063664415535179, 0.8636253828404642, 0.19228085622498767},`。
- **L1049 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9057490741507888, 0.8696584198795938, 0.19472348449042068},`.
  - **L1049 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9057490741507888, 0.8696584198795938, 0.19472348449042068},`。
- **L1050 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9051039312887829, 0.875689203317828, 0.197167703481744},`.
  - **L1050 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9051039312887829, 0.875689203317828, 0.197167703481744},`。
- **L1051 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9044307801230321, 0.8817178652984231, 0.19961354163268363},`.
  - **L1051 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9044307801230321, 0.8817178652984231, 0.19961354163268363},`。
- **L1052 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9037293835180177, 0.8877445340709991, 0.2020610260626995},`.
  - **L1052 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9037293835180177, 0.8877445340709991, 0.2020610260626995},`。
- **L1053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9029994999340526, 0.8937693341306414, 0.20451018263972115},`.
  - **L1053 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9029994999340526, 0.8937693341306414, 0.20451018263972115},`。
- **L1054 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9022408833103207, 0.8997923863509576, 0.2069610360393544},`.
  - **L1054 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9022408833103207, 0.8997923863509576, 0.2069610360393544},`。
- **L1055 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9059888943947992, 0.9040221289848984, 0.2345476432758713},`.
  - **L1055 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9059888943947992, 0.9040221289848984, 0.2345476432758713},`。
- **L1056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9121353668077321, 0.9072538390291374, 0.2723959319742808},`.
  - **L1056 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9121353668077321, 0.9072538390291374, 0.2723959319742808},`。
- **L1057 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.918103004002936, 0.910496798148879, 0.30707485069273716},`.
  - **L1057 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.918103004002936, 0.910496798148879, 0.30707485069273716},`。
- **L1058 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9238889872583352, 0.913751436869596, 0.3395892760133458},`.
  - **L1058 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9238889872583352, 0.913751436869596, 0.3395892760133458},`。
- **L1059 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9294904738341414, 0.9170181766931385, 0.3705416931634215},`.
  - **L1059 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9294904738341414, 0.9170181766931385, 0.3705416931634215},`。
- **L1060 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9349045893736176, 0.9202974302069179, 0.4003237614900562},`.
  - **L1060 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9349045893736176, 0.9202974302069179, 0.4003237614900562},`。
- **L1061 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9401284204049521, 0.9235896011902338, 0.4292051255044417},`.
  - **L1061 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9401284204049521, 0.9235896011902338, 0.4292051255044417},`。
- **L1062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9451590068972918, 0.9268950847179593, 0.45737955986332374},`.
  - **L1062 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9451590068972918, 0.9268950847179593, 0.45737955986332374},`。
- **L1063 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9499933348250024, 0.9302142672617891, 0.4849910259595919},`.
  - **L1063 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9499933348250024, 0.9302142672617891, 0.4849910259595919},`。
- **L1064 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9546283286947478, 0.9335475267892457, 0.5121493527958956},`.
  - **L1064 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9546283286947478, 0.9335475267892457, 0.5121493527958956},`。

### Lines 1065-1092 / 第 1065-1092 行

````cpp
1065:     {0.9590608439900784, 0.936895232860618, 0.5389401619772186},
1066:     {0.9632876594878218, 0.9402577467240134, 0.5654314125128757},
1067:     {0.9673054693997428, 0.9436354214086733, 0.5916778659454154},
1068:     {0.971110875291573, 0.9470286018167143, 0.6177242212867308},
1069:     {0.9747003777296833, 0.9504376248134289, 0.6436073706776709},
1070:     {0.9780703676032384, 0.9538628193162827, 0.6693580571761435},
1071:     {0.9812171170667078, 0.9573045063827349, 0.6950021159198041},
1072:     {0.9841367700439326, 0.9607629992969943, 0.7205614186521087},
1073:     {0.9868253322306101, 0.9642386036558243, 0.7460546029896806},
1074:     {0.9892786605268938, 0.9677316174534963, 0.7714976428174245},
1075:     {0.9914924518257658, 0.9712423311659882, 0.7969042996355301},
1076:     {0.9934622310757725, 0.9747710278345115, 0.8222864834722017},
1077:     {0.9951833385285259, 0.9783179831484546, 0.8476545442427741},
1078:     {0.9966509160718274, 0.9818834655278131, 0.873017509008704},
1079:     {0.9978598925382617, 0.9854677362051741, 0.8983832767211907},
1080:     {0.9988049678662737, 0.9890710493073268, 0.9237587792366369},
1081:     {0.999480595975958, 0.9926936519365445, 0.9491501153416039},
1082:     {0.9998809662045313, 0.9963357842516011, 0.9745626630050933},
1083:     {1.0, 1.0, 1.0},
1084: }};
1085: 
1086: template <typename Real>
1087: static constexpr std::array<std::array<Real, 3>, 256> plasma_data_ = {{
1088:     {0.05038205347059877, 0.029801736499741757, 0.5279751010495176},
1089:     {0.06353382706361996, 0.028424851177690835, 0.5331235351456174},
1090:     {0.07535267397875561, 0.027204618108821313, 0.5380072654878371},
1091:     {0.08622056271327161, 0.02612369628606181, 0.5426577546250408},
1092:     {0.09637909234021627, 0.02516351238474626, 0.5471034027913018},
````
- **L1065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9590608439900784, 0.936895232860618, 0.5389401619772186},`.
  - **L1065 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9590608439900784, 0.936895232860618, 0.5389401619772186},`。
- **L1066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9632876594878218, 0.9402577467240134, 0.5654314125128757},`.
  - **L1066 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9632876594878218, 0.9402577467240134, 0.5654314125128757},`。
- **L1067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9673054693997428, 0.9436354214086733, 0.5916778659454154},`.
  - **L1067 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9673054693997428, 0.9436354214086733, 0.5916778659454154},`。
- **L1068 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.971110875291573, 0.9470286018167143, 0.6177242212867308},`.
  - **L1068 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.971110875291573, 0.9470286018167143, 0.6177242212867308},`。
- **L1069 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9747003777296833, 0.9504376248134289, 0.6436073706776709},`.
  - **L1069 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9747003777296833, 0.9504376248134289, 0.6436073706776709},`。
- **L1070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9780703676032384, 0.9538628193162827, 0.6693580571761435},`.
  - **L1070 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9780703676032384, 0.9538628193162827, 0.6693580571761435},`。
- **L1071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9812171170667078, 0.9573045063827349, 0.6950021159198041},`.
  - **L1071 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9812171170667078, 0.9573045063827349, 0.6950021159198041},`。
- **L1072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9841367700439326, 0.9607629992969943, 0.7205614186521087},`.
  - **L1072 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9841367700439326, 0.9607629992969943, 0.7205614186521087},`。
- **L1073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9868253322306101, 0.9642386036558243, 0.7460546029896806},`.
  - **L1073 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9868253322306101, 0.9642386036558243, 0.7460546029896806},`。
- **L1074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9892786605268938, 0.9677316174534963, 0.7714976428174245},`.
  - **L1074 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9892786605268938, 0.9677316174534963, 0.7714976428174245},`。
- **L1075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9914924518257658, 0.9712423311659882, 0.7969042996355301},`.
  - **L1075 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9914924518257658, 0.9712423311659882, 0.7969042996355301},`。
- **L1076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9934622310757725, 0.9747710278345115, 0.8222864834722017},`.
  - **L1076 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9934622310757725, 0.9747710278345115, 0.8222864834722017},`。
- **L1077 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9951833385285259, 0.9783179831484546, 0.8476545442427741},`.
  - **L1077 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9951833385285259, 0.9783179831484546, 0.8476545442427741},`。
- **L1078 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9966509160718274, 0.9818834655278131, 0.873017509008704},`.
  - **L1078 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9966509160718274, 0.9818834655278131, 0.873017509008704},`。
- **L1079 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9978598925382617, 0.9854677362051741, 0.8983832767211907},`.
  - **L1079 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9978598925382617, 0.9854677362051741, 0.8983832767211907},`。
- **L1080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9988049678662737, 0.9890710493073268, 0.9237587792366369},`.
  - **L1080 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9988049678662737, 0.9890710493073268, 0.9237587792366369},`。
- **L1081 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.999480595975958, 0.9926936519365445, 0.9491501153416039},`.
  - **L1081 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.999480595975958, 0.9926936519365445, 0.9491501153416039},`。
- **L1082 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9998809662045313, 0.9963357842516011, 0.9745626630050933},`.
  - **L1082 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9998809662045313, 0.9963357842516011, 0.9745626630050933},`。
- **L1083 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{1.0, 1.0, 1.0},`.
  - **L1083 CN**: 继续一个多行参数列表、初始化器或聚合项：`{1.0, 1.0, 1.0},`。
- **L1084 EN**: Executes a standalone statement or declaration: `}};`.
  - **L1084 CN**: 执行一条独立语句或声明：`}};`。
- **L1085 EN**: Blank line separating nearby declarations or logic.
  - **L1085 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1086 EN**: Introduces template parameters or specialization context: `template <typename Real>`.
  - **L1086 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real>`。
- **L1087 EN**: Continues the surrounding expression or declaration: `static constexpr std::array<std::array<Real, 3>, 256> plasma_data_ = {{`.
  - **L1087 CN**: 继续构造周围的表达式或声明：`static constexpr std::array<std::array<Real, 3>, 256> plasma_data_ = {{`。
- **L1088 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.05038205347059877, 0.029801736499741757, 0.5279751010495176},`.
  - **L1088 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.05038205347059877, 0.029801736499741757, 0.5279751010495176},`。
- **L1089 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.06353382706361996, 0.028424851177690835, 0.5331235351456174},`.
  - **L1089 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.06353382706361996, 0.028424851177690835, 0.5331235351456174},`。
- **L1090 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.07535267397875561, 0.027204618108821313, 0.5380072654878371},`.
  - **L1090 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.07535267397875561, 0.027204618108821313, 0.5380072654878371},`。
- **L1091 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.08622056271327161, 0.02612369628606181, 0.5426577546250408},`.
  - **L1091 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.08622056271327161, 0.02612369628606181, 0.5426577546250408},`。
- **L1092 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.09637909234021627, 0.02516351238474626, 0.5471034027913018},`.
  - **L1092 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.09637909234021627, 0.02516351238474626, 0.5471034027913018},`。

### Lines 1093-1120 / 第 1093-1120 行

````cpp
1093:     {0.10597907050045527, 0.02430756301203486, 0.5513679345503644},
1094:     {0.11512442337588075, 0.0235544144422598, 0.5554685209574166},
1095:     {0.12390246837508934, 0.022876443394988937, 0.5594230869193363},
1096:     {0.13237958264924907, 0.022256467283011946, 0.5632496670754658},
1097:     {0.14060280627862978, 0.02168533317299063, 0.5669592160880329},
1098:     {0.14860595337659185, 0.021152345584078584, 0.5705618203562642},
1099:     {0.1564211065006705, 0.020649224182780333, 0.5740653310686394},
1100:     {0.16406928001421978, 0.020169228789859318, 0.5774779563374444},
1101:     {0.17157438178522647, 0.019704113165841947, 0.5808064352016079},
1102:     {0.1789495767422052, 0.019250113119147653, 0.584054078695406},
1103:     {0.18621180115469324, 0.018801112566559006, 0.5872277307558299},
1104:     {0.19337385391747602, 0.018351995878840264, 0.5903301881492813},
1105:     {0.20044409185351197, 0.01789999393180025, 0.593363854926767},
1106:     {0.20743511639383128, 0.01743987460850466, 0.5963332877187512},
1107:     {0.21434936531465415, 0.016970872739093673, 0.5992389686439623},
1108:     {0.22119736746683188, 0.016494748620846298, 0.6020833788659872},
1109:     {0.22798262579984302, 0.01600474729153228, 0.6048670732620841},
1110:     {0.23471390034056855, 0.015499747535836161, 0.6075917748304479},
1111:     {0.24139587735519374, 0.014976616724022502, 0.6102591681981376},
1112:     {0.24803115851643903, 0.01443661887416811, 0.6128678816678474},
1113:     {0.25462712204192733, 0.013879480363965033, 0.6154192540143766},
1114:     {0.26118240815987454, 0.013305484446440807, 0.6179109803957406},
1115:     {0.2677033607536392, 0.012713338432089919, 0.6203463317006325},
1116:     {0.27419065115382085, 0.012106344331753975, 0.6227220708698477},
1117:     {0.2806485945135714, 0.011485191704992936, 0.625038400861471},
1118:     {0.2870758894612177, 0.010852198594472949, 0.6272951521878755},
1119:     {0.29347719408360684, 0.010210206090003579, 0.6294899111444117},
1120:     {0.2998551242452729, 0.009558047948332786, 0.6316242235777169},
````
- **L1093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.10597907050045527, 0.02430756301203486, 0.5513679345503644},`.
  - **L1093 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.10597907050045527, 0.02430756301203486, 0.5513679345503644},`。
- **L1094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11512442337588075, 0.0235544144422598, 0.5554685209574166},`.
  - **L1094 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11512442337588075, 0.0235544144422598, 0.5554685209574166},`。
- **L1095 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12390246837508934, 0.022876443394988937, 0.5594230869193363},`.
  - **L1095 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12390246837508934, 0.022876443394988937, 0.5594230869193363},`。
- **L1096 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13237958264924907, 0.022256467283011946, 0.5632496670754658},`.
  - **L1096 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13237958264924907, 0.022256467283011946, 0.5632496670754658},`。
- **L1097 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14060280627862978, 0.02168533317299063, 0.5669592160880329},`.
  - **L1097 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14060280627862978, 0.02168533317299063, 0.5669592160880329},`。
- **L1098 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14860595337659185, 0.021152345584078584, 0.5705618203562642},`.
  - **L1098 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14860595337659185, 0.021152345584078584, 0.5705618203562642},`。
- **L1099 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1564211065006705, 0.020649224182780333, 0.5740653310686394},`.
  - **L1099 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1564211065006705, 0.020649224182780333, 0.5740653310686394},`。
- **L1100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.16406928001421978, 0.020169228789859318, 0.5774779563374444},`.
  - **L1100 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.16406928001421978, 0.020169228789859318, 0.5774779563374444},`。
- **L1101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.17157438178522647, 0.019704113165841947, 0.5808064352016079},`.
  - **L1101 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.17157438178522647, 0.019704113165841947, 0.5808064352016079},`。
- **L1102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1789495767422052, 0.019250113119147653, 0.584054078695406},`.
  - **L1102 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1789495767422052, 0.019250113119147653, 0.584054078695406},`。
- **L1103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.18621180115469324, 0.018801112566559006, 0.5872277307558299},`.
  - **L1103 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.18621180115469324, 0.018801112566559006, 0.5872277307558299},`。
- **L1104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.19337385391747602, 0.018351995878840264, 0.5903301881492813},`.
  - **L1104 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.19337385391747602, 0.018351995878840264, 0.5903301881492813},`。
- **L1105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.20044409185351197, 0.01789999393180025, 0.593363854926767},`.
  - **L1105 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.20044409185351197, 0.01789999393180025, 0.593363854926767},`。
- **L1106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.20743511639383128, 0.01743987460850466, 0.5963332877187512},`.
  - **L1106 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.20743511639383128, 0.01743987460850466, 0.5963332877187512},`。
- **L1107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.21434936531465415, 0.016970872739093673, 0.5992389686439623},`.
  - **L1107 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.21434936531465415, 0.016970872739093673, 0.5992389686439623},`。
- **L1108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.22119736746683188, 0.016494748620846298, 0.6020833788659872},`.
  - **L1108 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.22119736746683188, 0.016494748620846298, 0.6020833788659872},`。
- **L1109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.22798262579984302, 0.01600474729153228, 0.6048670732620841},`.
  - **L1109 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.22798262579984302, 0.01600474729153228, 0.6048670732620841},`。
- **L1110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.23471390034056855, 0.015499747535836161, 0.6075917748304479},`.
  - **L1110 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.23471390034056855, 0.015499747535836161, 0.6075917748304479},`。
- **L1111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.24139587735519374, 0.014976616724022502, 0.6102591681981376},`.
  - **L1111 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.24139587735519374, 0.014976616724022502, 0.6102591681981376},`。
- **L1112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.24803115851643903, 0.01443661887416811, 0.6128678816678474},`.
  - **L1112 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.24803115851643903, 0.01443661887416811, 0.6128678816678474},`。
- **L1113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.25462712204192733, 0.013879480363965033, 0.6154192540143766},`.
  - **L1113 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.25462712204192733, 0.013879480363965033, 0.6154192540143766},`。
- **L1114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.26118240815987454, 0.013305484446440807, 0.6179109803957406},`.
  - **L1114 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.26118240815987454, 0.013305484446440807, 0.6179109803957406},`。
- **L1115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2677033607536392, 0.012713338432089919, 0.6203463317006325},`.
  - **L1115 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2677033607536392, 0.012713338432089919, 0.6203463317006325},`。
- **L1116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27419065115382085, 0.012106344331753975, 0.6227220708698477},`.
  - **L1116 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27419065115382085, 0.012106344331753975, 0.6227220708698477},`。
- **L1117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2806485945135714, 0.011485191704992936, 0.625038400861471},`.
  - **L1117 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2806485945135714, 0.011485191704992936, 0.625038400861471},`。
- **L1118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2870758894612177, 0.010852198594472949, 0.6272951521878755},`.
  - **L1118 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2870758894612177, 0.010852198594472949, 0.6272951521878755},`。
- **L1119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.29347719408360684, 0.010210206090003579, 0.6294899111444117},`.
  - **L1119 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.29347719408360684, 0.010210206090003579, 0.6294899111444117},`。
- **L1120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2998551242452729, 0.009558047948332786, 0.6316242235777169},`.
  - **L1120 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2998551242452729, 0.009558047948332786, 0.6316242235777169},`。

### Lines 1121-1148 / 第 1121-1148 行

````cpp
1121:     {0.30620943129674466, 0.008899055475505046, 0.6336939963015096},
1122:     {0.3125433553743016, 0.008235893717416108, 0.6357002857590853},
1123:     {0.31885566464941756, 0.007572899889250414, 0.6376400727865161},
1124:     {0.3251505835029789, 0.0069117371894770714, 0.6395123380850828},
1125:     {0.3314258950888549, 0.006257740755607049, 0.6413161393477004},
1126:     {0.3376822144299248, 0.005614742031369818, 0.643048949066938},
1127:     {0.343925123616484, 0.00498757999986155, 0.6447101941536691},
1128:     {0.35014944433050965, 0.004378575729205644, 0.6462980198153693},
1129:     {0.3563593495264177, 0.0037944194657679817, 0.6478102381291251},
1130:     {0.3625526716933222, 0.0032394075990651583, 0.6492450805997739},
1131:     {0.3687335729955321, 0.002720261374452697, 0.6506012705463909},
1132:     {0.3748968968242403, 0.002241239865606646, 0.6518761301685362},
1133:     {0.38104622727684795, 0.001810210773066439, 0.6530679993306958},
1134:     {0.3871831206545137, 0.0014300761021609504, 0.6541771655986202},
1135:     {0.3933034524768182, 0.0011100331980040906, 0.6551990534286212},
1136:     {0.39941134218209506, 0.0008549181444151815, 0.6561331884791302},
1137:     {0.40550267547562835, 0.0006738585958816878, 0.656977095844568},
1138:     {0.4115805613352908, 0.0005727689276359682, 0.6577301981873359},
1139:     {0.4176418962835305, 0.000559689848675834, 0.6583901257368883},
1140:     {0.4236882377452387, 0.0006415963809730485, 0.6589560636788017},
1141:     {0.4297191157868382, 0.0008265327400941048, 0.6594251390708742},
1142:     {0.43573345912178957, 0.0011224144971593696, 0.6597970982982151},
1143:     {0.4417323328094361, 0.0015353882778950684, 0.6600691382766763},
1144:     {0.4477136781734527, 0.002075242343276844, 0.6602401195491348},
1145:     {0.45367754705516167, 0.0027502600296914884, 0.6603101230367315},
1146:     {0.45962289480536683, 0.003569082980411755, 0.6602771266757947},
1147:     {0.4655492492356133, 0.004539882193632057, 0.6601391424818946},
1148:     {0.4714571093971736, 0.005672943386802442, 0.6598971161567793},
````
- **L1121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.30620943129674466, 0.008899055475505046, 0.6336939963015096},`.
  - **L1121 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.30620943129674466, 0.008899055475505046, 0.6336939963015096},`。
- **L1122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3125433553743016, 0.008235893717416108, 0.6357002857590853},`.
  - **L1122 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3125433553743016, 0.008235893717416108, 0.6357002857590853},`。
- **L1123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.31885566464941756, 0.007572899889250414, 0.6376400727865161},`.
  - **L1123 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.31885566464941756, 0.007572899889250414, 0.6376400727865161},`。
- **L1124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3251505835029789, 0.0069117371894770714, 0.6395123380850828},`.
  - **L1124 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3251505835029789, 0.0069117371894770714, 0.6395123380850828},`。
- **L1125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3314258950888549, 0.006257740755607049, 0.6413161393477004},`.
  - **L1125 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3314258950888549, 0.006257740755607049, 0.6413161393477004},`。
- **L1126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3376822144299248, 0.005614742031369818, 0.643048949066938},`.
  - **L1126 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3376822144299248, 0.005614742031369818, 0.643048949066938},`。
- **L1127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.343925123616484, 0.00498757999986155, 0.6447101941536691},`.
  - **L1127 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.343925123616484, 0.00498757999986155, 0.6447101941536691},`。
- **L1128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.35014944433050965, 0.004378575729205644, 0.6462980198153693},`.
  - **L1128 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.35014944433050965, 0.004378575729205644, 0.6462980198153693},`。
- **L1129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3563593495264177, 0.0037944194657679817, 0.6478102381291251},`.
  - **L1129 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3563593495264177, 0.0037944194657679817, 0.6478102381291251},`。
- **L1130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3625526716933222, 0.0032394075990651583, 0.6492450805997739},`.
  - **L1130 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3625526716933222, 0.0032394075990651583, 0.6492450805997739},`。
- **L1131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3687335729955321, 0.002720261374452697, 0.6506012705463909},`.
  - **L1131 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3687335729955321, 0.002720261374452697, 0.6506012705463909},`。
- **L1132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3748968968242403, 0.002241239865606646, 0.6518761301685362},`.
  - **L1132 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3748968968242403, 0.002241239865606646, 0.6518761301685362},`。
- **L1133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.38104622727684795, 0.001810210773066439, 0.6530679993306958},`.
  - **L1133 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.38104622727684795, 0.001810210773066439, 0.6530679993306958},`。
- **L1134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3871831206545137, 0.0014300761021609504, 0.6541771655986202},`.
  - **L1134 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3871831206545137, 0.0014300761021609504, 0.6541771655986202},`。
- **L1135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3933034524768182, 0.0011100331980040906, 0.6551990534286212},`.
  - **L1135 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3933034524768182, 0.0011100331980040906, 0.6551990534286212},`。
- **L1136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.39941134218209506, 0.0008549181444151815, 0.6561331884791302},`.
  - **L1136 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.39941134218209506, 0.0008549181444151815, 0.6561331884791302},`。
- **L1137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.40550267547562835, 0.0006738585958816878, 0.656977095844568},`.
  - **L1137 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.40550267547562835, 0.0006738585958816878, 0.656977095844568},`。
- **L1138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4115805613352908, 0.0005727689276359682, 0.6577301981873359},`.
  - **L1138 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4115805613352908, 0.0005727689276359682, 0.6577301981873359},`。
- **L1139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4176418962835305, 0.000559689848675834, 0.6583901257368883},`.
  - **L1139 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4176418962835305, 0.000559689848675834, 0.6583901257368883},`。
- **L1140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4236882377452387, 0.0006415963809730485, 0.6589560636788017},`.
  - **L1140 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4236882377452387, 0.0006415963809730485, 0.6589560636788017},`。
- **L1141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4297191157868382, 0.0008265327400941048, 0.6594251390708742},`.
  - **L1141 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4297191157868382, 0.0008265327400941048, 0.6594251390708742},`。
- **L1142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.43573345912178957, 0.0011224144971593696, 0.6597970982982151},`.
  - **L1142 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.43573345912178957, 0.0011224144971593696, 0.6597970982982151},`。
- **L1143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4417323328094361, 0.0015353882778950684, 0.6600691382766763},`.
  - **L1143 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4417323328094361, 0.0015353882778950684, 0.6600691382766763},`。
- **L1144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4477136781734527, 0.002075242343276844, 0.6602401195491348},`.
  - **L1144 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4477136781734527, 0.002075242343276844, 0.6602401195491348},`。
- **L1145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.45367754705516167, 0.0027502600296914884, 0.6603101230367315},`.
  - **L1145 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.45367754705516167, 0.0027502600296914884, 0.6603101230367315},`。
- **L1146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.45962289480536683, 0.003569082980411755, 0.6602771266757947},`.
  - **L1146 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.45962289480536683, 0.003569082980411755, 0.6602771266757947},`。
- **L1147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4655492492356133, 0.004539882193632057, 0.6601391424818946},`.
  - **L1147 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4655492492356133, 0.004539882193632057, 0.6601391424818946},`。
- **L1148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4714571093971736, 0.005672943386802442, 0.6598971161567793},`.
  - **L1148 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4714571093971736, 0.005672943386802442, 0.6598971161567793},`。

### Lines 1149-1176 / 第 1149-1176 行

````cpp
1149:     {0.4773434667747343, 0.0069747060416045946, 0.6595491544709485},
1150:     {0.48321032105464906, 0.00845482487303384, 0.6590950901585111},
1151:     {0.4890546814926811, 0.010121547209062692, 0.6585341518234636},
1152:     {0.49487752945710545, 0.011984730267283644, 0.6578650488841956},
1153:     {0.5006778931801235, 0.01404940920364397, 0.6570881342241002},
1154:     {0.5064532641572821, 0.016327053582825775, 0.6562022327592171},
1155:     {0.512206101996081, 0.018827299101003524, 0.6552090987935029},
1156:     {0.517932476985869, 0.021556895188755264, 0.6541092197088938},
1157:     {0.5236333071472989, 0.02452621873336916, 0.6529010471797861},
1158:     {0.5293056865203295, 0.02774076220334348, 0.651586191657666},
1159:     {0.5349525083091329, 0.031211170127490386, 0.6501649809244013},
1160:     {0.5405698921735111, 0.034943657938542025, 0.6486401484123572},
1161:     {0.5461562839529602, 0.03894709993862434, 0.6470103291518646},
1162:     {0.5517150940137298, 0.04312993669146801, 0.6452770884507062},
1163:     {0.5572424908996385, 0.04732473166532047, 0.6434432899905641},
1164:     {0.5627382914453162, 0.05153956982770822, 0.6415090129244055},
1165:     {0.5682006939186361, 0.05577231508140689, 0.639477236154527},
1166:     {0.5736324841026456, 0.06002312768941918, 0.6373489241079173},
1167:     {0.5790288922820648, 0.06429083299444488, 0.635126168256215},
1168:     {0.5843916716197954, 0.06857462612193421, 0.6328118231142932},
1169:     {0.5897190859433459, 0.07287330061804262, 0.6304080857996939},
1170:     {0.5950105087389658, 0.07718497188889406, 0.627917359511397},
1171:     {0.600266274548656, 0.08151172798904885, 0.6253419897153364},
1172:     {0.6054847038259513, 0.08584937397306835, 0.6226862814869409},
1173:     {0.6106674577942587, 0.09020012029057023, 0.6199508828839105},
1174:     {0.6158118936935632, 0.09455974518910823, 0.6171401916106442},
1175:     {0.6209196353232503, 0.09893048328312148, 0.6142567668402714},
1176:     {0.625987078238522, 0.10330809138750463, 0.6113050903371116},
````
- **L1149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4773434667747343, 0.0069747060416045946, 0.6595491544709485},`.
  - **L1149 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4773434667747343, 0.0069747060416045946, 0.6595491544709485},`。
- **L1150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.48321032105464906, 0.00845482487303384, 0.6590950901585111},`.
  - **L1150 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.48321032105464906, 0.00845482487303384, 0.6590950901585111},`。
- **L1151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4890546814926811, 0.010121547209062692, 0.6585341518234636},`.
  - **L1151 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4890546814926811, 0.010121547209062692, 0.6585341518234636},`。
- **L1152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.49487752945710545, 0.011984730267283644, 0.6578650488841956},`.
  - **L1152 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.49487752945710545, 0.011984730267283644, 0.6578650488841956},`。
- **L1153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5006778931801235, 0.01404940920364397, 0.6570881342241002},`.
  - **L1153 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5006778931801235, 0.01404940920364397, 0.6570881342241002},`。
- **L1154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5064532641572821, 0.016327053582825775, 0.6562022327592171},`.
  - **L1154 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5064532641572821, 0.016327053582825775, 0.6562022327592171},`。
- **L1155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.512206101996081, 0.018827299101003524, 0.6552090987935029},`.
  - **L1155 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.512206101996081, 0.018827299101003524, 0.6552090987935029},`。
- **L1156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.517932476985869, 0.021556895188755264, 0.6541092197088938},`.
  - **L1156 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.517932476985869, 0.021556895188755264, 0.6541092197088938},`。
- **L1157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5236333071472989, 0.02452621873336916, 0.6529010471797861},`.
  - **L1157 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5236333071472989, 0.02452621873336916, 0.6529010471797861},`。
- **L1158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5293056865203295, 0.02774076220334348, 0.651586191657666},`.
  - **L1158 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5293056865203295, 0.02774076220334348, 0.651586191657666},`。
- **L1159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5349525083091329, 0.031211170127490386, 0.6501649809244013},`.
  - **L1159 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5349525083091329, 0.031211170127490386, 0.6501649809244013},`。
- **L1160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5405698921735111, 0.034943657938542025, 0.6486401484123572},`.
  - **L1160 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5405698921735111, 0.034943657938542025, 0.6486401484123572},`。
- **L1161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5461562839529602, 0.03894709993862434, 0.6470103291518646},`.
  - **L1161 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5461562839529602, 0.03894709993862434, 0.6470103291518646},`。
- **L1162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5517150940137298, 0.04312993669146801, 0.6452770884507062},`.
  - **L1162 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5517150940137298, 0.04312993669146801, 0.6452770884507062},`。
- **L1163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5572424908996385, 0.04732473166532047, 0.6434432899905641},`.
  - **L1163 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5572424908996385, 0.04732473166532047, 0.6434432899905641},`。
- **L1164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5627382914453162, 0.05153956982770822, 0.6415090129244055},`.
  - **L1164 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5627382914453162, 0.05153956982770822, 0.6415090129244055},`。
- **L1165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5682006939186361, 0.05577231508140689, 0.639477236154527},`.
  - **L1165 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5682006939186361, 0.05577231508140689, 0.639477236154527},`。
- **L1166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5736324841026456, 0.06002312768941918, 0.6373489241079173},`.
  - **L1166 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5736324841026456, 0.06002312768941918, 0.6373489241079173},`。
- **L1167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5790288922820648, 0.06429083299444488, 0.635126168256215},`.
  - **L1167 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5790288922820648, 0.06429083299444488, 0.635126168256215},`。
- **L1168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5843916716197954, 0.06857462612193421, 0.6328118231142932},`.
  - **L1168 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5843916716197954, 0.06857462612193421, 0.6328118231142932},`。
- **L1169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5897190859433459, 0.07287330061804262, 0.6304080857996939},`.
  - **L1169 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5897190859433459, 0.07287330061804262, 0.6304080857996939},`。
- **L1170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5950105087389658, 0.07718497188889406, 0.627917359511397},`.
  - **L1170 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5950105087389658, 0.07718497188889406, 0.627917359511397},`。
- **L1171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.600266274548656, 0.08151172798904885, 0.6253419897153364},`.
  - **L1171 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.600266274548656, 0.08151172798904885, 0.6253419897153364},`。
- **L1172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6054847038259513, 0.08584937397306835, 0.6226862814869409},`.
  - **L1172 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6054847038259513, 0.08584937397306835, 0.6226862814869409},`。
- **L1173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6106674577942587, 0.09020012029057023, 0.6199508828839105},`.
  - **L1173 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6106674577942587, 0.09020012029057023, 0.6199508828839105},`。
- **L1174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6158118936935632, 0.09455974518910823, 0.6171401916106442},`.
  - **L1174 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6158118936935632, 0.09455974518910823, 0.6171401916106442},`。
- **L1175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6209196353232503, 0.09893048328312148, 0.6142567668402714},`.
  - **L1175 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6209196353232503, 0.09893048328312148, 0.6142567668402714},`。
- **L1176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.625987078238522, 0.10330809138750463, 0.6113050903371116},`.
  - **L1176 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.625987078238522, 0.10330809138750463, 0.6113050903371116},`。

### Lines 1177-1204 / 第 1177-1204 行

````cpp
1177:     {0.6310165297116345, 0.10769470016602048, 0.6082874219554474},
1178:     {0.6360082571946242, 0.11208841745759027, 0.6052049784133696},
1179:     {0.6409587158624425, 0.11648801193763364, 0.6020653233567386},
1180:     {0.6458724305886356, 0.120894724408147, 0.5988668588802512},
1181:     {0.6507458962880415, 0.1253053066756304, 0.5956172158180971},
1182:     {0.65558059804725, 0.12972201477991677, 0.59231673323355},
1183:     {0.6603740711643852, 0.13414058702631182, 0.5889711004114799},
1184:     {0.6651285526432004, 0.1385621615837031, 0.5855824728255514},
1185:     {0.669845240296135, 0.1429888560980906, 0.582153977816893},
1186:     {0.6745217289587141, 0.14741542201076516, 0.5786883585907729},
1187:     {0.6791604036855876, 0.15184511348395324, 0.5751888508715063},
1188:     {0.6837578997178535, 0.1562746720253843, 0.5716602384468066},
1189:     {0.6883185616652895, 0.16070636061039098, 0.5681027207214548},
1190:     {0.6928400648216108, 0.16513791277492407, 0.5645221138655818},
1191:     {0.6973235757805955, 0.1695694677210192, 0.5609195098635373},
1192:     {0.7017692243107729, 0.1740021467142358, 0.5572959855617685},
1193:     {0.7061777422578236, 0.17843369623403554, 0.5536573853799269},
1194:     {0.7105493783435904, 0.18286537302055275, 0.5500038553740142},
1195:     {0.7148829033672169, 0.1872959178727836, 0.5463382579948199},
1196:     {0.7191815273421805, 0.19172659246702414, 0.5426627244615733},
1197:     {0.7234440590934259, 0.196155133293472, 0.5389811286444972},
1198:     {0.7276695980921538, 0.20058267652430906, 0.5352935340967259},
1199:     {0.7318622095195731, 0.20501034405621088, 0.5316009984849964},
1200:     {0.7360187549600071, 0.20943588386264006, 0.527908404300108},
1201:     {0.7401433549469377, 0.2138615497973006, 0.5242158683817802},
1202:     {0.7442319068498962, 0.2182850864322064, 0.520524273866664},
1203:     {0.7482894956274093, 0.22270875088361386, 0.516833738969623},
1204:     {0.7523120538611476, 0.2271302847089524, 0.5131491434259574},
````
- **L1177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6310165297116345, 0.10769470016602048, 0.6082874219554474},`.
  - **L1177 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6310165297116345, 0.10769470016602048, 0.6082874219554474},`。
- **L1178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6360082571946242, 0.11208841745759027, 0.6052049784133696},`.
  - **L1178 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6360082571946242, 0.11208841745759027, 0.6052049784133696},`。
- **L1179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6409587158624425, 0.11648801193763364, 0.6020653233567386},`.
  - **L1179 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6409587158624425, 0.11648801193763364, 0.6020653233567386},`。
- **L1180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6458724305886356, 0.120894724408147, 0.5988668588802512},`.
  - **L1180 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6458724305886356, 0.120894724408147, 0.5988668588802512},`。
- **L1181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6507458962880415, 0.1253053066756304, 0.5956172158180971},`.
  - **L1181 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6507458962880415, 0.1253053066756304, 0.5956172158180971},`。
- **L1182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.65558059804725, 0.12972201477991677, 0.59231673323355},`.
  - **L1182 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.65558059804725, 0.12972201477991677, 0.59231673323355},`。
- **L1183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6603740711643852, 0.13414058702631182, 0.5889711004114799},`.
  - **L1183 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6603740711643852, 0.13414058702631182, 0.5889711004114799},`。
- **L1184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6651285526432004, 0.1385621615837031, 0.5855824728255514},`.
  - **L1184 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6651285526432004, 0.1385621615837031, 0.5855824728255514},`。
- **L1185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.669845240296135, 0.1429888560980906, 0.582153977816893},`.
  - **L1185 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.669845240296135, 0.1429888560980906, 0.582153977816893},`。
- **L1186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6745217289587141, 0.14741542201076516, 0.5786883585907729},`.
  - **L1186 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6745217289587141, 0.14741542201076516, 0.5786883585907729},`。
- **L1187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6791604036855876, 0.15184511348395324, 0.5751888508715063},`.
  - **L1187 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6791604036855876, 0.15184511348395324, 0.5751888508715063},`。
- **L1188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6837578997178535, 0.1562746720253843, 0.5716602384468066},`.
  - **L1188 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6837578997178535, 0.1562746720253843, 0.5716602384468066},`。
- **L1189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6883185616652895, 0.16070636061039098, 0.5681027207214548},`.
  - **L1189 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6883185616652895, 0.16070636061039098, 0.5681027207214548},`。
- **L1190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6928400648216108, 0.16513791277492407, 0.5645221138655818},`.
  - **L1190 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6928400648216108, 0.16513791277492407, 0.5645221138655818},`。
- **L1191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6973235757805955, 0.1695694677210192, 0.5609195098635373},`.
  - **L1191 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6973235757805955, 0.1695694677210192, 0.5609195098635373},`。
- **L1192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7017692243107729, 0.1740021467142358, 0.5572959855617685},`.
  - **L1192 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7017692243107729, 0.1740021467142358, 0.5572959855617685},`。
- **L1193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7061777422578236, 0.17843369623403554, 0.5536573853799269},`.
  - **L1193 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7061777422578236, 0.17843369623403554, 0.5536573853799269},`。
- **L1194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7105493783435904, 0.18286537302055275, 0.5500038553740142},`.
  - **L1194 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7105493783435904, 0.18286537302055275, 0.5500038553740142},`。
- **L1195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7148829033672169, 0.1872959178727836, 0.5463382579948199},`.
  - **L1195 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7148829033672169, 0.1872959178727836, 0.5463382579948199},`。
- **L1196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7191815273421805, 0.19172659246702414, 0.5426627244615733},`.
  - **L1196 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7191815273421805, 0.19172659246702414, 0.5426627244615733},`。
- **L1197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7234440590934259, 0.196155133293472, 0.5389811286444972},`.
  - **L1197 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7234440590934259, 0.196155133293472, 0.5389811286444972},`。
- **L1198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7276695980921538, 0.20058267652430906, 0.5352935340967259},`.
  - **L1198 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7276695980921538, 0.20058267652430906, 0.5352935340967259},`。
- **L1199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7318622095195731, 0.20501034405621088, 0.5316009984849964},`.
  - **L1199 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7318622095195731, 0.20501034405621088, 0.5316009984849964},`。
- **L1200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7360187549600071, 0.20943588386264006, 0.527908404300108},`.
  - **L1200 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7360187549600071, 0.20943588386264006, 0.527908404300108},`。
- **L1201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7401433549469377, 0.2138615497973006, 0.5242158683817802},`.
  - **L1201 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7401433549469377, 0.2138615497973006, 0.5242158683817802},`。
- **L1202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7442319068498962, 0.2182850864322064, 0.520524273866664},`.
  - **L1202 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7442319068498962, 0.2182850864322064, 0.520524273866664},`。
- **L1203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7482894956274093, 0.22270875088361386, 0.516833738969623},`.
  - **L1203 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7482894956274093, 0.22270875088361386, 0.516833738969623},`。
- **L1204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7523120538611476, 0.2271302847089524, 0.5131491434259574},`.
  - **L1204 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7523120538611476, 0.2271302847089524, 0.5131491434259574},`。

### Lines 1205-1232 / 第 1205-1232 行

````cpp
1205:     {0.7563036182318114, 0.23155182037018107, 0.509468548147597},
1206:     {0.760264196042346, 0.23597347976619776, 0.505794013908438},
1207:     {0.7641927663268164, 0.24039301314981837, 0.5021264170700042},
1208:     {0.7680903336977949, 0.24481467160443043, 0.49846488562080354},
1209:     {0.7719579098239822, 0.24923420250548467, 0.494813286735782},
1210:     {0.7757964670137317, 0.2536558602885356, 0.49117075886112754},
1211:     {0.7796040489503765, 0.2580753889528994, 0.487539157444613},
1212:     {0.7833826364367662, 0.262496918855311, 0.48391855557538205},
1213:     {0.7871331836672137, 0.2669195732674377, 0.4803070299618413},
1214:     {0.7908547764917832, 0.2713421013551332, 0.4767064258861618},
1215:     {0.7945493142314948, 0.2757677557136531, 0.47311690432863956},
1216:     {0.7982159123706819, 0.2801942814938595, 0.4695382974681735},
1217:     {0.801855440805233, 0.28462393619999926, 0.46597078035645983},
1218:     {0.8054670443253337, 0.28905445984517764, 0.46241517054928255},
1219:     {0.8090525637702202, 0.2934891151740604, 0.4588696581516773},
1220:     {0.8126121722311862, 0.29792563692815727, 0.4553380455774297},
1221:     {0.8161437857429269, 0.3023651595826911, 0.45181643298028346},
1222:     {0.8196512962080388, 0.30680981323456863, 0.44830592266123775},
1223:     {0.8231319147062668, 0.31125833345906073, 0.4448063071891859},
1224:     {0.8265884164961489, 0.31571198854333155, 0.4413158011478088},
1225:     {0.8300180400348914, 0.3201695064581668, 0.43783618296069426},
1226:     {0.8334225331012705, 0.3246331633551905, 0.43436568094830924},
1227:     {0.8368011616192005, 0.3291026789528604, 0.4309050603621552},
1228:     {0.8401547945201179, 0.33357719515430856, 0.42745544005358527},
1229:     {0.8434842793132458, 0.33805985158665625, 0.42401293964429126},
1230:     {0.8467879171814312, 0.3425483651134692, 0.42057931701643003},
1231:     {0.8500663933256695, 0.3470460241716122, 0.41715281999997633},
1232:     {0.853319036312829, 0.35155053489832117, 0.4137341950926385},
````
- **L1205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7563036182318114, 0.23155182037018107, 0.509468548147597},`.
  - **L1205 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7563036182318114, 0.23155182037018107, 0.509468548147597},`。
- **L1206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.760264196042346, 0.23597347976619776, 0.505794013908438},`.
  - **L1206 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.760264196042346, 0.23597347976619776, 0.505794013908438},`。
- **L1207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7641927663268164, 0.24039301314981837, 0.5021264170700042},`.
  - **L1207 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7641927663268164, 0.24039301314981837, 0.5021264170700042},`。
- **L1208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7680903336977949, 0.24481467160443043, 0.49846488562080354},`.
  - **L1208 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7680903336977949, 0.24481467160443043, 0.49846488562080354},`。
- **L1209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7719579098239822, 0.24923420250548467, 0.494813286735782},`.
  - **L1209 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7719579098239822, 0.24923420250548467, 0.494813286735782},`。
- **L1210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7757964670137317, 0.2536558602885356, 0.49117075886112754},`.
  - **L1210 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7757964670137317, 0.2536558602885356, 0.49117075886112754},`。
- **L1211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7796040489503765, 0.2580753889528994, 0.487539157444613},`.
  - **L1211 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7796040489503765, 0.2580753889528994, 0.487539157444613},`。
- **L1212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7833826364367662, 0.262496918855311, 0.48391855557538205},`.
  - **L1212 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7833826364367662, 0.262496918855311, 0.48391855557538205},`。
- **L1213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7871331836672137, 0.2669195732674377, 0.4803070299618413},`.
  - **L1213 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7871331836672137, 0.2669195732674377, 0.4803070299618413},`。
- **L1214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7908547764917832, 0.2713421013551332, 0.4767064258861618},`.
  - **L1214 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7908547764917832, 0.2713421013551332, 0.4767064258861618},`。
- **L1215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7945493142314948, 0.2757677557136531, 0.47311690432863956},`.
  - **L1215 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7945493142314948, 0.2757677557136531, 0.47311690432863956},`。
- **L1216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7982159123706819, 0.2801942814938595, 0.4695382974681735},`.
  - **L1216 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7982159123706819, 0.2801942814938595, 0.4695382974681735},`。
- **L1217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.801855440805233, 0.28462393619999926, 0.46597078035645983},`.
  - **L1217 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.801855440805233, 0.28462393619999926, 0.46597078035645983},`。
- **L1218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8054670443253337, 0.28905445984517764, 0.46241517054928255},`.
  - **L1218 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8054670443253337, 0.28905445984517764, 0.46241517054928255},`。
- **L1219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8090525637702202, 0.2934891151740604, 0.4588696581516773},`.
  - **L1219 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8090525637702202, 0.2934891151740604, 0.4588696581516773},`。
- **L1220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8126121722311862, 0.29792563692815727, 0.4553380455774297},`.
  - **L1220 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8126121722311862, 0.29792563692815727, 0.4553380455774297},`。
- **L1221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8161437857429269, 0.3023651595826911, 0.45181643298028346},`.
  - **L1221 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8161437857429269, 0.3023651595826911, 0.45181643298028346},`。
- **L1222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8196512962080388, 0.30680981323456863, 0.44830592266123775},`.
  - **L1222 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8196512962080388, 0.30680981323456863, 0.44830592266123775},`。
- **L1223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8231319147062668, 0.31125833345906073, 0.4448063071891859},`.
  - **L1223 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8231319147062668, 0.31125833345906073, 0.4448063071891859},`。
- **L1224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8265884164961489, 0.31571198854333155, 0.4413158011478088},`.
  - **L1224 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8265884164961489, 0.31571198854333155, 0.4413158011478088},`。
- **L1225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8300180400348914, 0.3201695064581668, 0.43783618296069426},`.
  - **L1225 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8300180400348914, 0.3201695064581668, 0.43783618296069426},`。
- **L1226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8334225331012705, 0.3246331633551905, 0.43436568094830924},`.
  - **L1226 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8334225331012705, 0.3246331633551905, 0.43436568094830924},`。
- **L1227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8368011616192005, 0.3291026789528604, 0.4309050603621552},`.
  - **L1227 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8368011616192005, 0.3291026789528604, 0.4309050603621552},`。
- **L1228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8401547945201179, 0.33357719515430856, 0.42745544005358527},`.
  - **L1228 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8401547945201179, 0.33357719515430856, 0.42745544005358527},`。
- **L1229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8434842793132458, 0.33805985158665625, 0.42401293964429126},`.
  - **L1229 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8434842793132458, 0.33805985158665625, 0.42401293964429126},`。
- **L1230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8467879171814312, 0.3425483651134692, 0.42057931701643003},`.
  - **L1230 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8467879171814312, 0.3425483651134692, 0.42057931701643003},`。
- **L1231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8500663933256695, 0.3470460241716122, 0.41715281999997633},`.
  - **L1231 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8500663933256695, 0.3470460241716122, 0.41715281999997633},`。
- **L1232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.853319036312829, 0.35155053489832117, 0.4137341950926385},`.
  - **L1232 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.853319036312829, 0.35155053489832117, 0.4137341950926385},`。

### Lines 1233-1260 / 第 1233-1260 行

````cpp
1233:     {0.8565475037861691, 0.35606419689875035, 0.4103217013252645},
1234:     {0.8597501517507731, 0.3605857048991911, 0.4069170743064185},
1235:     {0.8629268039532411, 0.36511621303625214, 0.40351944853543975},
1236:     {0.8660782631128492, 0.36965787597771715, 0.4001259551388744},
1237:     {0.8692029205728464, 0.3742093808977564, 0.3967383275867196},
1238:     {0.8723033707464051, 0.37877204764837713, 0.39335483663056486},
1239:     {0.8753760334959819, 0.38334454940566526, 0.38997620732417515},
1240:     {0.8784234745450857, 0.3879302203109463, 0.3865997186779487},
1241:     {0.8814431427224427, 0.3925267186370576, 0.38322908763780417},
1242:     {0.8844358150168464, 0.39713621668929217, 0.3798604589983262},
1243:     {0.8874022475255273, 0.4017598897994324, 0.3764939692996154},
1244:     {0.8903399254603347, 0.40639538423669647, 0.3731303392842935},
1245:     {0.8932503482591801, 0.41104606226907536, 0.3697678512068009},
1246:     {0.8961310320776956, 0.41570955292043654, 0.3664072199362221},
1247:     {0.8989844448210237, 0.42039023641999307, 0.36304673325600695},
1248:     {0.9018071346530614, 0.4250847229961323, 0.35968810073229274},
1249:     {0.9046008288037846, 0.4297942088907302, 0.35632947125103215},
1250:     {0.9073652324299479, 0.4345218955954952, 0.3529699824197478},
1251:     {0.9100979328220618, 0.43926537724749737, 0.3496103521951318},
1252:     {0.9128003257007559, 0.4440270702284773, 0.3462508642176817},
1253:     {0.9154710325568927, 0.4488045475293247, 0.3428902329848131},
1254:     {0.9181094141599885, 0.45360124699433185, 0.3395287458827024},
1255:     {0.9207141279319739, 0.45841471975818476, 0.3361661136226204},
1256:     {0.9232868464124966, 0.463248191094171, 0.3328014851212701},
1257:     {0.9258252179695905, 0.46810089493647095, 0.3294349948733575},
1258:     {0.928328943461609, 0.4729723618415593, 0.32606736582761625},
1259:     {0.9307983028156818, 0.4778650729009944, 0.32269687622031573},
1260:     {0.9332320356689454, 0.48277753475654456, 0.31932524626654357},
````
- **L1233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8565475037861691, 0.35606419689875035, 0.4103217013252645},`.
  - **L1233 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8565475037861691, 0.35606419689875035, 0.4103217013252645},`。
- **L1234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8597501517507731, 0.3605857048991911, 0.4069170743064185},`.
  - **L1234 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8597501517507731, 0.3605857048991911, 0.4069170743064185},`。
- **L1235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8629268039532411, 0.36511621303625214, 0.40351944853543975},`.
  - **L1235 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8629268039532411, 0.36511621303625214, 0.40351944853543975},`。
- **L1236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8660782631128492, 0.36965787597771715, 0.4001259551388744},`.
  - **L1236 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8660782631128492, 0.36965787597771715, 0.4001259551388744},`。
- **L1237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8692029205728464, 0.3742093808977564, 0.3967383275867196},`.
  - **L1237 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8692029205728464, 0.3742093808977564, 0.3967383275867196},`。
- **L1238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8723033707464051, 0.37877204764837713, 0.39335483663056486},`.
  - **L1238 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8723033707464051, 0.37877204764837713, 0.39335483663056486},`。
- **L1239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8753760334959819, 0.38334454940566526, 0.38997620732417515},`.
  - **L1239 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8753760334959819, 0.38334454940566526, 0.38997620732417515},`。
- **L1240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8784234745450857, 0.3879302203109463, 0.3865997186779487},`.
  - **L1240 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8784234745450857, 0.3879302203109463, 0.3865997186779487},`。
- **L1241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8814431427224427, 0.3925267186370576, 0.38322908763780417},`.
  - **L1241 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8814431427224427, 0.3925267186370576, 0.38322908763780417},`。
- **L1242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8844358150168464, 0.39713621668929217, 0.3798604589983262},`.
  - **L1242 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8844358150168464, 0.39713621668929217, 0.3798604589983262},`。
- **L1243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8874022475255273, 0.4017598897994324, 0.3764939692996154},`.
  - **L1243 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8874022475255273, 0.4017598897994324, 0.3764939692996154},`。
- **L1244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8903399254603347, 0.40639538423669647, 0.3731303392842935},`.
  - **L1244 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8903399254603347, 0.40639538423669647, 0.3731303392842935},`。
- **L1245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8932503482591801, 0.41104606226907536, 0.3697678512068009},`.
  - **L1245 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8932503482591801, 0.41104606226907536, 0.3697678512068009},`。
- **L1246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8961310320776956, 0.41570955292043654, 0.3664072199362221},`.
  - **L1246 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8961310320776956, 0.41570955292043654, 0.3664072199362221},`。
- **L1247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8989844448210237, 0.42039023641999307, 0.36304673325600695},`.
  - **L1247 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8989844448210237, 0.42039023641999307, 0.36304673325600695},`。
- **L1248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9018071346530614, 0.4250847229961323, 0.35968810073229274},`.
  - **L1248 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9018071346530614, 0.4250847229961323, 0.35968810073229274},`。
- **L1249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9046008288037846, 0.4297942088907302, 0.35632947125103215},`.
  - **L1249 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9046008288037846, 0.4297942088907302, 0.35632947125103215},`。
- **L1250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9073652324299479, 0.4345218955954952, 0.3529699824197478},`.
  - **L1250 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9073652324299479, 0.4345218955954952, 0.3529699824197478},`。
- **L1251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9100979328220618, 0.43926537724749737, 0.3496103521951318},`.
  - **L1251 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9100979328220618, 0.43926537724749737, 0.3496103521951318},`。
- **L1252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9128003257007559, 0.4440270702284773, 0.3462508642176817},`.
  - **L1252 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9128003257007559, 0.4440270702284773, 0.3462508642176817},`。
- **L1253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9154710325568927, 0.4488045475293247, 0.3428902329848131},`.
  - **L1253 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9154710325568927, 0.4488045475293247, 0.3428902329848131},`。
- **L1254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9181094141599885, 0.45360124699433185, 0.3395287458827024},`.
  - **L1254 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9181094141599885, 0.45360124699433185, 0.3395287458827024},`。
- **L1255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9207141279319739, 0.45841471975818476, 0.3361661136226204},`.
  - **L1255 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9207141279319739, 0.45841471975818476, 0.3361661136226204},`。
- **L1256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9232868464124966, 0.463248191094171, 0.3328014851212701},`.
  - **L1256 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9232868464124966, 0.463248191094171, 0.3328014851212701},`。
- **L1257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9258252179695905, 0.46810089493647095, 0.3294349948733575},`.
  - **L1257 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9258252179695905, 0.46810089493647095, 0.3294349948733575},`。
- **L1258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.928328943461609, 0.4729723618415593, 0.32606736582761625},`.
  - **L1258 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.928328943461609, 0.4729723618415593, 0.32606736582761625},`。
- **L1259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9307983028156818, 0.4778650729009944, 0.32269687622031573},`.
  - **L1259 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9307983028156818, 0.4778650729009944, 0.32269687622031573},`。
- **L1260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9332320356689454, 0.48277753475654456, 0.31932524626654357},`.
  - **L1260 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9332320356689454, 0.48277753475654456, 0.31932524626654357},`。

### Lines 1261-1288 / 第 1261-1288 行

````cpp
1261:     {0.935630382313249, 0.4877102534916354, 0.3159517572279356},
1262:     {0.9379901229720649, 0.4926647102078156, 0.3125751263224191},
1263:     {0.94031286905006, 0.4976391653394365, 0.30919749958566495},
1264:     {0.9425982043994484, 0.5026368889510107, 0.3058160067766645},
1265:     {0.9448439581861301, 0.507655339193432, 0.30243337976158996},
1266:     {0.9470512797767, 0.5126970710073548, 0.2990488875873464},
1267:     {0.9492170420289342, 0.5177605158051324, 0.29566225939396684},
1268:     {0.9513443493064214, 0.5228482560427825, 0.2922747680564989},
1269:     {0.95342812024145, 0.5279576952925963, 0.2888831386174737},
1270:     {0.9554704126542967, 0.5330914441820193, 0.28548964857337034},
1271:     {0.9574691920662094, 0.5382478782970336, 0.2820960181641041},
1272:     {0.9594239777856378, 0.5434283103460965, 0.27870139293719676},
1273:     {0.9613362570153808, 0.5486340651043815, 0.27530489863581714},
1274:     {0.9632030520719662, 0.5538624913974601, 0.2719092715909324},
1275:     {0.9650243153126318, 0.5591162551393607, 0.26851277921609085},
1276:     {0.9667981202444841, 0.5643936755766422, 0.2651181499385089},
1277:     {0.9685263667936735, 0.5696984485228503, 0.26172065963235575},
1278:     {0.9702051814086846, 0.5750258633596333, 0.258325028790098},
1279:     {0.9718350028152128, 0.5803792760776375, 0.25493140399836905},
1280:     {0.973416234677147, 0.5857590553651805, 0.2515399093896209},
1281:     {0.9749470666071361, 0.5911624621207934, 0.24815128177724166},
1282:     {0.9764282806289234, 0.5965932508399702, 0.24476679063210272},
1283:     {0.9778561235005537, 0.6020486514176712, 0.24138715955646184},
1284:     {0.9792333190737449, 0.6075304496592373, 0.23801267291982608},
1285:     {0.980556172868528, 0.6130368443483086, 0.23464603815019064},
1286:     {0.9818260338342485, 0.618569237111758, 0.2312874100772107},
1287:     {0.983041213251059, 0.6241290418881347, 0.2279369200964918},
1288:     {0.984199086129537, 0.6297154282611285, 0.22459528753260782},
````
- **L1261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.935630382313249, 0.4877102534916354, 0.3159517572279356},`.
  - **L1261 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.935630382313249, 0.4877102534916354, 0.3159517572279356},`。
- **L1262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9379901229720649, 0.4926647102078156, 0.3125751263224191},`.
  - **L1262 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9379901229720649, 0.4926647102078156, 0.3125751263224191},`。
- **L1263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.94031286905006, 0.4976391653394365, 0.30919749958566495},`.
  - **L1263 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.94031286905006, 0.4976391653394365, 0.30919749958566495},`。
- **L1264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9425982043994484, 0.5026368889510107, 0.3058160067766645},`.
  - **L1264 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9425982043994484, 0.5026368889510107, 0.3058160067766645},`。
- **L1265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9448439581861301, 0.507655339193432, 0.30243337976158996},`.
  - **L1265 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9448439581861301, 0.507655339193432, 0.30243337976158996},`。
- **L1266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9470512797767, 0.5126970710073548, 0.2990488875873464},`.
  - **L1266 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9470512797767, 0.5126970710073548, 0.2990488875873464},`。
- **L1267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9492170420289342, 0.5177605158051324, 0.29566225939396684},`.
  - **L1267 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9492170420289342, 0.5177605158051324, 0.29566225939396684},`。
- **L1268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9513443493064214, 0.5228482560427825, 0.2922747680564989},`.
  - **L1268 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9513443493064214, 0.5228482560427825, 0.2922747680564989},`。
- **L1269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.95342812024145, 0.5279576952925963, 0.2888831386174737},`.
  - **L1269 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.95342812024145, 0.5279576952925963, 0.2888831386174737},`。
- **L1270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9554704126542967, 0.5330914441820193, 0.28548964857337034},`.
  - **L1270 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9554704126542967, 0.5330914441820193, 0.28548964857337034},`。
- **L1271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9574691920662094, 0.5382478782970336, 0.2820960181641041},`.
  - **L1271 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9574691920662094, 0.5382478782970336, 0.2820960181641041},`。
- **L1272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9594239777856378, 0.5434283103460965, 0.27870139293719676},`.
  - **L1272 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9594239777856378, 0.5434283103460965, 0.27870139293719676},`。
- **L1273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9613362570153808, 0.5486340651043815, 0.27530489863581714},`.
  - **L1273 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9613362570153808, 0.5486340651043815, 0.27530489863581714},`。
- **L1274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9632030520719662, 0.5538624913974601, 0.2719092715909324},`.
  - **L1274 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9632030520719662, 0.5538624913974601, 0.2719092715909324},`。
- **L1275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9650243153126318, 0.5591162551393607, 0.26851277921609085},`.
  - **L1275 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9650243153126318, 0.5591162551393607, 0.26851277921609085},`。
- **L1276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9667981202444841, 0.5643936755766422, 0.2651181499385089},`.
  - **L1276 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9667981202444841, 0.5643936755766422, 0.2651181499385089},`。
- **L1277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9685263667936735, 0.5696984485228503, 0.26172065963235575},`.
  - **L1277 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9685263667936735, 0.5696984485228503, 0.26172065963235575},`。
- **L1278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9702051814086846, 0.5750258633596333, 0.258325028790098},`.
  - **L1278 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9702051814086846, 0.5750258633596333, 0.258325028790098},`。
- **L1279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9718350028152128, 0.5803792760776375, 0.25493140399836905},`.
  - **L1279 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9718350028152128, 0.5803792760776375, 0.25493140399836905},`。
- **L1280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.973416234677147, 0.5857590553651805, 0.2515399093896209},`.
  - **L1280 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.973416234677147, 0.5857590553651805, 0.2515399093896209},`。
- **L1281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9749470666071361, 0.5911624621207934, 0.24815128177724166},`.
  - **L1281 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9749470666071361, 0.5911624621207934, 0.24815128177724166},`。
- **L1282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9764282806289234, 0.5965932508399702, 0.24476679063210272},`.
  - **L1282 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9764282806289234, 0.5965932508399702, 0.24476679063210272},`。
- **L1283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9778561235005537, 0.6020486514176712, 0.24138715955646184},`.
  - **L1283 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9778561235005537, 0.6020486514176712, 0.24138715955646184},`。
- **L1284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9792333190737449, 0.6075304496592373, 0.23801267291982608},`.
  - **L1284 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9792333190737449, 0.6075304496592373, 0.23801267291982608},`。
- **L1285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.980556172868528, 0.6130368443483086, 0.23464603815019064},`.
  - **L1285 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.980556172868528, 0.6130368443483086, 0.23464603815019064},`。
- **L1286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9818260338342485, 0.618569237111758, 0.2312874100772107},`.
  - **L1286 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9818260338342485, 0.618569237111758, 0.2312874100772107},`。
- **L1287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.983041213251059, 0.6241290418881347, 0.2279369200964918},`.
  - **L1287 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.983041213251059, 0.6241290418881347, 0.2279369200964918},`。
- **L1288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.984199086129537, 0.6297154282611285, 0.22459528753260782},`.
  - **L1288 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.984199086129537, 0.6297154282611285, 0.22459528753260782},`。

### Lines 1289-1316 / 第 1289-1316 行

````cpp
1289:     {0.9853012455753073, 0.6353282428938619, 0.22126480373791835},
1290:     {0.9863451305526932, 0.6409666230896872, 0.2179481656192832},
1291:     {0.987332269632511, 0.6466314474784756, 0.21464768988345995},
1292:     {0.988260166889663, 0.6523228213835303, 0.2113640451754461},
1293:     {0.9891280720682684, 0.6580401935630623, 0.20810040712251393},
1294:     {0.989935193134965, 0.6637850245048573, 0.2048589308368297},
1295:     {0.9906811112647872, 0.6695553903969398, 0.20164228467841383},
1296:     {0.9913652104916543, 0.675353231166615, 0.19845282041957812},
1297:     {0.9919851419644327, 0.681176590770927, 0.19529516473738912},
1298:     {0.9925412186450403, 0.6870284413732891, 0.19216971523394297},
1299:     {0.9930321637354537, 0.6929047946873668, 0.18908404856516375},
1300:     {0.9934561180556553, 0.6988071461016482, 0.18604138565745748},
1301:     {0.9938141746253392, 0.7047390032802133, 0.18304294167742508},
1302:     {0.9941031429791255, 0.7106953484789053, 0.18009726563511483},
1303:     {0.9943241754734297, 0.7166792156038955, 0.17720784326567807},
1304:     {0.9944741584952311, 0.7226885545120336, 0.17438115112347216},
1305:     {0.9945531662213939, 0.7287264314705092, 0.17162175513054803},
1306:     {0.994561164165776, 0.7347887640856848, 0.16893804446424532},
1307:     {0.9944951727911301, 0.7408770946176795, 0.16633533149747243},
1308:     {0.9943551582286289, 0.7469929781194636, 0.16382095249343898},
1309:     {0.9941411819033531, 0.7531343027240105, 0.16140421771177754},
1310:     {0.993851140881807, 0.7593021960529095, 0.1590918763664673},
1311:     {0.9934821808433413, 0.7654965143680154, 0.15689111532223957},
1312:     {0.9930331122322037, 0.7717184174350065, 0.15480781781660563},
1313:     {0.9925051688188334, 0.7779647295700995, 0.15285502755098562},
1314:     {0.9918972375664695, 0.7842360395514291, 0.15104222448313137},
1315:     {0.9912091443124736, 0.7905349489016323, 0.1493769620239061},
1316:     {0.9904392291943233, 0.7968562534486284, 0.1478701260432203},
````
- **L1289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9853012455753073, 0.6353282428938619, 0.22126480373791835},`.
  - **L1289 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9853012455753073, 0.6353282428938619, 0.22126480373791835},`。
- **L1290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9863451305526932, 0.6409666230896872, 0.2179481656192832},`.
  - **L1290 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9863451305526932, 0.6409666230896872, 0.2179481656192832},`。
- **L1291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.987332269632511, 0.6466314474784756, 0.21464768988345995},`.
  - **L1291 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.987332269632511, 0.6466314474784756, 0.21464768988345995},`。
- **L1292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.988260166889663, 0.6523228213835303, 0.2113640451754461},`.
  - **L1292 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.988260166889663, 0.6523228213835303, 0.2113640451754461},`。
- **L1293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9891280720682684, 0.6580401935630623, 0.20810040712251393},`.
  - **L1293 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9891280720682684, 0.6580401935630623, 0.20810040712251393},`。
- **L1294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.989935193134965, 0.6637850245048573, 0.2048589308368297},`.
  - **L1294 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.989935193134965, 0.6637850245048573, 0.2048589308368297},`。
- **L1295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9906811112647872, 0.6695553903969398, 0.20164228467841383},`.
  - **L1295 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9906811112647872, 0.6695553903969398, 0.20164228467841383},`。
- **L1296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9913652104916543, 0.675353231166615, 0.19845282041957812},`.
  - **L1296 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9913652104916543, 0.675353231166615, 0.19845282041957812},`。
- **L1297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9919851419644327, 0.681176590770927, 0.19529516473738912},`.
  - **L1297 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9919851419644327, 0.681176590770927, 0.19529516473738912},`。
- **L1298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9925412186450403, 0.6870284413732891, 0.19216971523394297},`.
  - **L1298 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9925412186450403, 0.6870284413732891, 0.19216971523394297},`。
- **L1299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9930321637354537, 0.6929047946873668, 0.18908404856516375},`.
  - **L1299 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9930321637354537, 0.6929047946873668, 0.18908404856516375},`。
- **L1300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9934561180556553, 0.6988071461016482, 0.18604138565745748},`.
  - **L1300 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9934561180556553, 0.6988071461016482, 0.18604138565745748},`。
- **L1301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9938141746253392, 0.7047390032802133, 0.18304294167742508},`.
  - **L1301 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9938141746253392, 0.7047390032802133, 0.18304294167742508},`。
- **L1302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9941031429791255, 0.7106953484789053, 0.18009726563511483},`.
  - **L1302 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9941031429791255, 0.7106953484789053, 0.18009726563511483},`。
- **L1303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9943241754734297, 0.7166792156038955, 0.17720784326567807},`.
  - **L1303 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9943241754734297, 0.7166792156038955, 0.17720784326567807},`。
- **L1304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9944741584952311, 0.7226885545120336, 0.17438115112347216},`.
  - **L1304 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9944741584952311, 0.7226885545120336, 0.17438115112347216},`。
- **L1305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9945531662213939, 0.7287264314705092, 0.17162175513054803},`.
  - **L1305 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9945531662213939, 0.7287264314705092, 0.17162175513054803},`。
- **L1306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.994561164165776, 0.7347887640856848, 0.16893804446424532},`.
  - **L1306 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.994561164165776, 0.7347887640856848, 0.16893804446424532},`。
- **L1307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9944951727911301, 0.7408770946176795, 0.16633533149747243},`.
  - **L1307 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9944951727911301, 0.7408770946176795, 0.16633533149747243},`。
- **L1308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9943551582286289, 0.7469929781194636, 0.16382095249343898},`.
  - **L1308 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9943551582286289, 0.7469929781194636, 0.16382095249343898},`。
- **L1309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9941411819033531, 0.7531343027240105, 0.16140421771177754},`.
  - **L1309 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9941411819033531, 0.7531343027240105, 0.16140421771177754},`。
- **L1310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.993851140881807, 0.7593021960529095, 0.1590918763664673},`.
  - **L1310 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.993851140881807, 0.7593021960529095, 0.1590918763664673},`。
- **L1311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9934821808433413, 0.7654965143680154, 0.15689111532223957},`.
  - **L1311 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9934821808433413, 0.7654965143680154, 0.15689111532223957},`。
- **L1312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9930331122322037, 0.7717184174350065, 0.15480781781660563},`.
  - **L1312 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9930331122322037, 0.7717184174350065, 0.15480781781660563},`。
- **L1313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9925051688188334, 0.7779647295700995, 0.15285502755098562},`.
  - **L1313 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9925051688188334, 0.7779647295700995, 0.15285502755098562},`。
- **L1314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9918972375664695, 0.7842360395514291, 0.15104222448313137},`.
  - **L1314 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9918972375664695, 0.7842360395514291, 0.15104222448313137},`。
- **L1315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9912091443124736, 0.7905349489016323, 0.1493769620239061},`.
  - **L1315 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9912091443124736, 0.7905349489016323, 0.1493769620239061},`。
- **L1316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9904392291943233, 0.7968562534486284, 0.1478701260432203},`.
  - **L1316 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9904392291943233, 0.7968562534486284, 0.1478701260432203},`。

### Lines 1317-1344 / 第 1317-1344 行

````cpp
1317:     {0.9895871071514554, 0.8032031721875696, 0.14652892189857109},
1318:     {0.9886482096752364, 0.8095764705517057, 0.1453570476879498},
1319:     {0.9876210577245915, 0.815976398662919, 0.14436290762709728},
1320:     {0.986509178197983, 0.8223986911764639, 0.14355699253114637},
1321:     {0.9853139963633676, 0.8288446279885695, 0.14294492163758302},
1322:     {0.9840311334696324, 0.8353129155540111, 0.14252796680830504},
1323:     {0.9826532853628255, 0.8418092005650518, 0.14230298881977124},
1324:     {0.9811900755223993, 0.8483271434249618, 0.14227897296255487},
1325:     {0.9796442447131871, 0.854863423552606, 0.1424529510156046},
1326:     {0.9779950043835023, 0.8614303744688768, 0.14280800468858543},
1327:     {0.9762651924699953, 0.8680136489692144, 0.14335094248589855},
1328:     {0.9744429204121595, 0.8746206082101065, 0.14406105658536314},
1329:     {0.972530126666216, 0.8812478776950604, 0.14492296106549138},
1330:     {0.9705333468892211, 0.8878931459431306, 0.14591884881454822},
1331:     {0.968443047163475, 0.8945621097233081, 0.14701399877638566},
1332:     {0.9662712851816472, 0.901246373209881, 0.14817986820470014},
1333:     {0.964020955903533, 0.9079483440427187, 0.1493700336843369},
1334:     {0.9616812107840874, 0.9146696029935507, 0.1505199051295305},
1335:     {0.959275857396244, 0.9214055790441742, 0.15156603392174148},
1336:     {0.9568081246010715, 0.9281498350816461, 0.1524089373934798},
1337:     {0.9542874028057909, 0.934905090703852, 0.15292087802256293},
1338:     {0.9517260321586368, 0.9416690683960072, 0.15292489914850274},
1339:     {0.9491513146628231, 0.9484323225943193, 0.1521779775289114},
1340:     {0.9466019493393495, 0.9551882982848324, 0.15032766414862223},
1341:     {0.9441522165792432, 0.9619135560223769, 0.14686103617160504},
1342:     {0.9418959241387963, 0.9685885113553053, 0.14095488309848805},
1343:     {0.9400151278782742, 0.9751557856205376, 0.131325887773911},
1344: }};
````
- **L1317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9895871071514554, 0.8032031721875696, 0.14652892189857109},`.
  - **L1317 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9895871071514554, 0.8032031721875696, 0.14652892189857109},`。
- **L1318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9886482096752364, 0.8095764705517057, 0.1453570476879498},`.
  - **L1318 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9886482096752364, 0.8095764705517057, 0.1453570476879498},`。
- **L1319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9876210577245915, 0.815976398662919, 0.14436290762709728},`.
  - **L1319 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9876210577245915, 0.815976398662919, 0.14436290762709728},`。
- **L1320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.986509178197983, 0.8223986911764639, 0.14355699253114637},`.
  - **L1320 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.986509178197983, 0.8223986911764639, 0.14355699253114637},`。
- **L1321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9853139963633676, 0.8288446279885695, 0.14294492163758302},`.
  - **L1321 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9853139963633676, 0.8288446279885695, 0.14294492163758302},`。
- **L1322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9840311334696324, 0.8353129155540111, 0.14252796680830504},`.
  - **L1322 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9840311334696324, 0.8353129155540111, 0.14252796680830504},`。
- **L1323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9826532853628255, 0.8418092005650518, 0.14230298881977124},`.
  - **L1323 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9826532853628255, 0.8418092005650518, 0.14230298881977124},`。
- **L1324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9811900755223993, 0.8483271434249618, 0.14227897296255487},`.
  - **L1324 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9811900755223993, 0.8483271434249618, 0.14227897296255487},`。
- **L1325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9796442447131871, 0.854863423552606, 0.1424529510156046},`.
  - **L1325 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9796442447131871, 0.854863423552606, 0.1424529510156046},`。
- **L1326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9779950043835023, 0.8614303744688768, 0.14280800468858543},`.
  - **L1326 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9779950043835023, 0.8614303744688768, 0.14280800468858543},`。
- **L1327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9762651924699953, 0.8680136489692144, 0.14335094248589855},`.
  - **L1327 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9762651924699953, 0.8680136489692144, 0.14335094248589855},`。
- **L1328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9744429204121595, 0.8746206082101065, 0.14406105658536314},`.
  - **L1328 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9744429204121595, 0.8746206082101065, 0.14406105658536314},`。
- **L1329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.972530126666216, 0.8812478776950604, 0.14492296106549138},`.
  - **L1329 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.972530126666216, 0.8812478776950604, 0.14492296106549138},`。
- **L1330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9705333468892211, 0.8878931459431306, 0.14591884881454822},`.
  - **L1330 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9705333468892211, 0.8878931459431306, 0.14591884881454822},`。
- **L1331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.968443047163475, 0.8945621097233081, 0.14701399877638566},`.
  - **L1331 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.968443047163475, 0.8945621097233081, 0.14701399877638566},`。
- **L1332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9662712851816472, 0.901246373209881, 0.14817986820470014},`.
  - **L1332 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9662712851816472, 0.901246373209881, 0.14817986820470014},`。
- **L1333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.964020955903533, 0.9079483440427187, 0.1493700336843369},`.
  - **L1333 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.964020955903533, 0.9079483440427187, 0.1493700336843369},`。
- **L1334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9616812107840874, 0.9146696029935507, 0.1505199051295305},`.
  - **L1334 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9616812107840874, 0.9146696029935507, 0.1505199051295305},`。
- **L1335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.959275857396244, 0.9214055790441742, 0.15156603392174148},`.
  - **L1335 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.959275857396244, 0.9214055790441742, 0.15156603392174148},`。
- **L1336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9568081246010715, 0.9281498350816461, 0.1524089373934798},`.
  - **L1336 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9568081246010715, 0.9281498350816461, 0.1524089373934798},`。
- **L1337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9542874028057909, 0.934905090703852, 0.15292087802256293},`.
  - **L1337 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9542874028057909, 0.934905090703852, 0.15292087802256293},`。
- **L1338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9517260321586368, 0.9416690683960072, 0.15292489914850274},`.
  - **L1338 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9517260321586368, 0.9416690683960072, 0.15292489914850274},`。
- **L1339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9491513146628231, 0.9484323225943193, 0.1521779775289114},`.
  - **L1339 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9491513146628231, 0.9484323225943193, 0.1521779775289114},`。
- **L1340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9466019493393495, 0.9551882982848324, 0.15032766414862223},`.
  - **L1340 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9466019493393495, 0.9551882982848324, 0.15032766414862223},`。
- **L1341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9441522165792432, 0.9619135560223769, 0.14686103617160504},`.
  - **L1341 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9441522165792432, 0.9619135560223769, 0.14686103617160504},`。
- **L1342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9418959241387963, 0.9685885113553053, 0.14095488309848805},`.
  - **L1342 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9418959241387963, 0.9685885113553053, 0.14095488309848805},`。
- **L1343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9400151278782742, 0.9751557856205376, 0.131325887773911},`.
  - **L1343 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9400151278782742, 0.9751557856205376, 0.131325887773911},`。
- **L1344 EN**: Executes a standalone statement or declaration: `}};`.
  - **L1344 CN**: 执行一条独立语句或声明：`}};`。

### Lines 1345-1372 / 第 1345-1372 行

````cpp
1345: 
1346: template <typename Real>
1347: static constexpr std::array<std::array<Real, 3>, 256> smooth_cool_warm_data_ = {
1348:         {{0.22999950386952345, 0.2989989340493756, 0.754000138575591},
1349:          {0.23451750918602265, 0.30586471825124395, 0.760211287847582},
1350:          {0.23905139222321087, 0.31271835359723077, 0.7663613706951183},
1351:          {0.2436017365913441, 0.3195596165920886, 0.7724494083708939},
1352:          {0.24816908043486074, 0.3263882334026712, 0.7784744332927407},
1353:          {0.25275391815771386, 0.33320388462766504, 0.7844354891865984},
1354:          {0.2573567020073366, 0.34000620954660093, 0.7903316312278389},
1355:          {0.2619778435347082, 0.34679480991553485, 0.7961619261808826},
1356:          {0.26661771494573344, 0.3535692533668808, 0.8019254525370478},
1357:          {0.271276650357161, 0.3603290764626063, 0.8076213006505808},
1358:          {0.27595494696856954, 0.3670737874430388, 0.813248572872804},
1359:          {0.28065286616048707, 0.37380286870769464, 0.8188063836843368},
1360:          {0.28537063452740463, 0.3805157790595848, 0.8242938598253321},
1361:          {0.29010844485334814, 0.3872119557402635, 0.829710140423685},
1362:          {0.29486645703670583, 0.3938908162793105, 0.8350543771211604},
1363:          {0.2996447989701334, 0.4005517601788923, 0.8403257341974003},
1364:          {0.30444356738064665, 0.4071941704514421, 0.8455233886917599},
1365:          {0.3092628286343541, 0.41381741502624314, 0.8506465305229377},
1366:          {0.31410261950970153, 0.42042084803879193, 0.8556943626063482},
1367:          {0.3189629479426216, 0.4270038110151302, 0.8606661009692109},
1368:          {0.32384379374653405, 0.43356563396190284, 0.8655609748633052},
1369:          {0.32874510930975565, 0.4401056363716498, 0.8703782268753609},
1370:          {0.33366682027256384, 0.4466231281517378, 0.8751171130350492},
1371:          {0.33860882618583127, 0.45311741048440196, 0.8797769029205332},
1372:          {0.3435710011529169, 0.45958777662452927, 0.8843568797615591},
````
- **L1345 EN**: Blank line separating nearby declarations or logic.
  - **L1345 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1346 EN**: Introduces template parameters or specialization context: `template <typename Real>`.
  - **L1346 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real>`。
- **L1347 EN**: Continues the surrounding expression or declaration: `static constexpr std::array<std::array<Real, 3>, 256> smooth_cool_warm_data_ = {`.
  - **L1347 CN**: 继续构造周围的表达式或声明：`static constexpr std::array<std::array<Real, 3>, 256> smooth_cool_warm_data_ = {`。
- **L1348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{0.22999950386952345, 0.2989989340493756, 0.754000138575591},`.
  - **L1348 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{0.22999950386952345, 0.2989989340493756, 0.754000138575591},`。
- **L1349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.23451750918602265, 0.30586471825124395, 0.760211287847582},`.
  - **L1349 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.23451750918602265, 0.30586471825124395, 0.760211287847582},`。
- **L1350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.23905139222321087, 0.31271835359723077, 0.7663613706951183},`.
  - **L1350 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.23905139222321087, 0.31271835359723077, 0.7663613706951183},`。
- **L1351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2436017365913441, 0.3195596165920886, 0.7724494083708939},`.
  - **L1351 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2436017365913441, 0.3195596165920886, 0.7724494083708939},`。
- **L1352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.24816908043486074, 0.3263882334026712, 0.7784744332927407},`.
  - **L1352 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.24816908043486074, 0.3263882334026712, 0.7784744332927407},`。
- **L1353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.25275391815771386, 0.33320388462766504, 0.7844354891865984},`.
  - **L1353 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.25275391815771386, 0.33320388462766504, 0.7844354891865984},`。
- **L1354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2573567020073366, 0.34000620954660093, 0.7903316312278389},`.
  - **L1354 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2573567020073366, 0.34000620954660093, 0.7903316312278389},`。
- **L1355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2619778435347082, 0.34679480991553485, 0.7961619261808826},`.
  - **L1355 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2619778435347082, 0.34679480991553485, 0.7961619261808826},`。
- **L1356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.26661771494573344, 0.3535692533668808, 0.8019254525370478},`.
  - **L1356 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.26661771494573344, 0.3535692533668808, 0.8019254525370478},`。
- **L1357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.271276650357161, 0.3603290764626063, 0.8076213006505808},`.
  - **L1357 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.271276650357161, 0.3603290764626063, 0.8076213006505808},`。
- **L1358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27595494696856954, 0.3670737874430388, 0.813248572872804},`.
  - **L1358 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27595494696856954, 0.3670737874430388, 0.813248572872804},`。
- **L1359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.28065286616048707, 0.37380286870769464, 0.8188063836843368},`.
  - **L1359 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.28065286616048707, 0.37380286870769464, 0.8188063836843368},`。
- **L1360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.28537063452740463, 0.3805157790595848, 0.8242938598253321},`.
  - **L1360 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.28537063452740463, 0.3805157790595848, 0.8242938598253321},`。
- **L1361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.29010844485334814, 0.3872119557402635, 0.829710140423685},`.
  - **L1361 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.29010844485334814, 0.3872119557402635, 0.829710140423685},`。
- **L1362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.29486645703670583, 0.3938908162793105, 0.8350543771211604},`.
  - **L1362 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.29486645703670583, 0.3938908162793105, 0.8350543771211604},`。
- **L1363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2996447989701334, 0.4005517601788923, 0.8403257341974003},`.
  - **L1363 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2996447989701334, 0.4005517601788923, 0.8403257341974003},`。
- **L1364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.30444356738064665, 0.4071941704514421, 0.8455233886917599},`.
  - **L1364 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.30444356738064665, 0.4071941704514421, 0.8455233886917599},`。
- **L1365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3092628286343541, 0.41381741502624314, 0.8506465305229377},`.
  - **L1365 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3092628286343541, 0.41381741502624314, 0.8506465305229377},`。
- **L1366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.31410261950970153, 0.42042084803879193, 0.8556943626063482},`.
  - **L1366 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.31410261950970153, 0.42042084803879193, 0.8556943626063482},`。
- **L1367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3189629479426216, 0.4270038110151302, 0.8606661009692109},`.
  - **L1367 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3189629479426216, 0.4270038110151302, 0.8606661009692109},`。
- **L1368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.32384379374653405, 0.43356563396190284, 0.8655609748633052},`.
  - **L1368 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.32384379374653405, 0.43356563396190284, 0.8655609748633052},`。
- **L1369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.32874510930975565, 0.4401056363716498, 0.8703782268753609},`.
  - **L1369 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.32874510930975565, 0.4401056363716498, 0.8703782268753609},`。
- **L1370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.33366682027256384, 0.4466231281517378, 0.8751171130350492},`.
  - **L1370 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.33366682027256384, 0.4466231281517378, 0.8751171130350492},`。
- **L1371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.33860882618583127, 0.45311741048440196, 0.8797769029205332},`.
  - **L1371 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.33860882618583127, 0.45311741048440196, 0.8797769029205332},`。
- **L1372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3435710011529169, 0.45958777662452927, 0.8843568797615591},`.
  - **L1372 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3435710011529169, 0.45958777662452927, 0.8843568797615591},`。

### Lines 1373-1400 / 第 1373-1400 行

````cpp
1373:          {0.3485531944562718, 0.46603351264108767, 0.8888563405400349},
1374:          {0.3535552311699886, 0.47245389810747423, 0.8932745960880898},
1375:          {0.3585769127594044, 0.47884820674548473, 0.8976109711835649},
1376:          {0.3636180176686533, 0.4852157070271205, 0.9018648046429221},
1377:          {0.3686783018969878, 0.49155566273800605, 0.9060354494115308},
1378:          {0.37375749956453747, 0.49786733350580864, 0.9101222726513168},
1379:          {0.37885532346808715, 0.5041499752967064, 0.9141246558257436},
1380:          {0.38397146562736917, 0.5104028408826495, 0.918041994782103},
1381:          {0.38910559782229387, 0.5166251802818846, 0.9218736998310882},
1382:          {0.39425737212145806, 0.5228162411749818, 0.9256191958236355},
1383:          {0.39942642140225487, 0.5289752692983762, 0.9292779222250065},
1384:          {0.4046123598628117, 0.5351015088172548, 0.9328493331860931},
1385:          {0.4098147835259849, 0.541194202679442, 0.9363328976119261},
1386:          {0.41503327073558766, 0.5472525929517891, 0.9397280992273693},
1387:          {0.4202673826449946, 0.5532759211404236, 0.943034436639982},
1388:          {0.42551666369825586, 0.5592634284961118, 0.9462514234000324},
1389:          {0.43078064210382544, 0.5652143563058468, 0.9493785880576479},
1390:          {0.43605883030099185, 0.5711279461717078, 0.9524154742170866},
1391:          {0.44135072541910086, 0.5770034402779125, 0.9553616405881157},
1392:          {0.4466558097296202, 0.5828400816469308, 0.9582166610344868},
1393:          {0.4519735510911263, 0.5886371143854353, 0.960980124619491},
1394:          {0.457303403387259, 0.5943937839208078, 0.9636516356485881},
1395:          {0.46264480695769356, 0.6001093372288535, 0.9662308137090968},
1396:          {0.4679971890221843, 0.6057830230533238, 0.968717293706932},
1397:          {0.4733599640977317, 0.6114140921177984, 0.9711107259003909},
1398:          {0.4787325344089037, 0.6170017973304325, 0.9734107759309701},
1399:          {0.48411429029138214, 0.6225453939820256, 0.9756171248512094},
1400:          {0.48950461058876166, 0.6280441399378467, 0.9777294691495587},
````
- **L1373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3485531944562718, 0.46603351264108767, 0.8888563405400349},`.
  - **L1373 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3485531944562718, 0.46603351264108767, 0.8888563405400349},`。
- **L1374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3535552311699886, 0.47245389810747423, 0.8932745960880898},`.
  - **L1374 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3535552311699886, 0.47245389810747423, 0.8932745960880898},`。
- **L1375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3585769127594044, 0.47884820674548473, 0.8976109711835649},`.
  - **L1375 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3585769127594044, 0.47884820674548473, 0.8976109711835649},`。
- **L1376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3636180176686533, 0.4852157070271205, 0.9018648046429221},`.
  - **L1376 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3636180176686533, 0.4852157070271205, 0.9018648046429221},`。
- **L1377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3686783018969878, 0.49155566273800605, 0.9060354494115308},`.
  - **L1377 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3686783018969878, 0.49155566273800605, 0.9060354494115308},`。
- **L1378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.37375749956453747, 0.49786733350580864, 0.9101222726513168},`.
  - **L1378 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.37375749956453747, 0.49786733350580864, 0.9101222726513168},`。
- **L1379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.37885532346808715, 0.5041499752967064, 0.9141246558257436},`.
  - **L1379 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.37885532346808715, 0.5041499752967064, 0.9141246558257436},`。
- **L1380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.38397146562736917, 0.5104028408826495, 0.918041994782103},`.
  - **L1380 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.38397146562736917, 0.5104028408826495, 0.918041994782103},`。
- **L1381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.38910559782229387, 0.5166251802818846, 0.9218736998310882},`.
  - **L1381 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.38910559782229387, 0.5166251802818846, 0.9218736998310882},`。
- **L1382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.39425737212145806, 0.5228162411749818, 0.9256191958236355},`.
  - **L1382 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.39425737212145806, 0.5228162411749818, 0.9256191958236355},`。
- **L1383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.39942642140225487, 0.5289752692983762, 0.9292779222250065},`.
  - **L1383 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.39942642140225487, 0.5289752692983762, 0.9292779222250065},`。
- **L1384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4046123598628117, 0.5351015088172548, 0.9328493331860931},`.
  - **L1384 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4046123598628117, 0.5351015088172548, 0.9328493331860931},`。
- **L1385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4098147835259849, 0.541194202679442, 0.9363328976119261},`.
  - **L1385 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4098147835259849, 0.541194202679442, 0.9363328976119261},`。
- **L1386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.41503327073558766, 0.5472525929517891, 0.9397280992273693},`.
  - **L1386 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.41503327073558766, 0.5472525929517891, 0.9397280992273693},`。
- **L1387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4202673826449946, 0.5532759211404236, 0.943034436639982},`.
  - **L1387 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4202673826449946, 0.5532759211404236, 0.943034436639982},`。
- **L1388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.42551666369825586, 0.5592634284961118, 0.9462514234000324},`.
  - **L1388 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.42551666369825586, 0.5592634284961118, 0.9462514234000324},`。
- **L1389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.43078064210382544, 0.5652143563058468, 0.9493785880576479},`.
  - **L1389 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.43078064210382544, 0.5652143563058468, 0.9493785880576479},`。
- **L1390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.43605883030099185, 0.5711279461717078, 0.9524154742170866},`.
  - **L1390 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.43605883030099185, 0.5711279461717078, 0.9524154742170866},`。
- **L1391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.44135072541910086, 0.5770034402779125, 0.9553616405881157},`.
  - **L1391 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.44135072541910086, 0.5770034402779125, 0.9553616405881157},`。
- **L1392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4466558097296202, 0.5828400816469308, 0.9582166610344868},`.
  - **L1392 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4466558097296202, 0.5828400816469308, 0.9582166610344868},`。
- **L1393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4519735510911263, 0.5886371143854353, 0.960980124619491},`.
  - **L1393 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4519735510911263, 0.5886371143854353, 0.960980124619491},`。
- **L1394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.457303403387259, 0.5943937839208078, 0.9636516356485881},`.
  - **L1394 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.457303403387259, 0.5943937839208078, 0.9636516356485881},`。
- **L1395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.46264480695769356, 0.6001093372288535, 0.9662308137090968},`.
  - **L1395 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.46264480695769356, 0.6001093372288535, 0.9662308137090968},`。
- **L1396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4679971890221843, 0.6057830230533238, 0.968717293706932},`.
  - **L1396 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4679971890221843, 0.6057830230533238, 0.968717293706932},`。
- **L1397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4733599640977317, 0.6114140921177984, 0.9711107259003909},`.
  - **L1397 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4733599640977317, 0.6114140921177984, 0.9711107259003909},`。
- **L1398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4787325344089037, 0.6170017973304325, 0.9734107759309701},`.
  - **L1398 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4787325344089037, 0.6170017973304325, 0.9734107759309701},`。
- **L1399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.48411429029138214, 0.6225453939820256, 0.9756171248512094},`.
  - **L1399 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.48411429029138214, 0.6225453939820256, 0.9756171248512094},`。
- **L1400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.48950461058876166, 0.6280441399378467, 0.9777294691495587},`.
  - **L1400 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.48950461058876166, 0.6280441399378467, 0.9777294691495587},`。

### Lines 1401-1428 / 第 1401-1428 行

````cpp
1401:          {0.49490286304267245, 0.6334972958235978, 0.9797475207722576},
1402:          {0.5003084046762621, 0.6389041252058836, 0.9816710071422271},
1403:          {0.5057205821711058, 0.644263894767512, 0.983499671174965},
1404:          {0.511138732237593, 0.6495758744779339, 0.9852332712914448},
1405:          {0.5165621819788595, 0.654839337759102, 0.9868715814280132},
1406:          {0.5219902492483212, 0.6600535616470082, 0.9884143910432847},
1407:          {0.5274222430008886, 0.6652178269491371, 0.9898615051220324},
1408:          {0.5328574636379119, 0.670331418398056, 0.9912127441760742},
1409:          {0.5382952033459485, 0.6753936248013475, 0.9924679442421503},
1410:          {0.5437347464294153, 0.6804037391880692, 0.9936269568768029},
1411:          {0.5491753696372063, 0.685361058951912, 0.9946896491482432},
1412:          {0.5546163424833603, 0.690264885991224, 0.9956559036252249},
1413:          {0.5600569275618472, 0.695114526846042, 0.9965256183629146},
1414:          {0.5654963808555731, 0.6999092928322662, 0.9972987068857656},
1415:          {0.5709339520396719, 0.7046485001731109, 0.9979750981673984},
1416:          {0.5763688847791901, 0.7093314701279408, 0.9985547366074979},
1417:          {0.5818004170212294, 0.7139575291186055, 0.9990375820057199},
1418:          {0.5872277812816606, 0.7185260088533676, 0.9994236095326229},
1419:          {0.5926502049264796, 0.7230362464485207, 0.9997128096976274},
1420:          {0.5980669104479099, 0.7274875845477786, 0.9999051883140077},
1421:          {0.603477115735333, 0.7318793714395154, 1.0},
1422:          {0.6088800343411463, 0.7362109611719294, 0.9999995804425049},
1423:          {0.6142748757416401, 0.7404817136661965, 0.9999016817439826},
1424:          {0.6196608455929782, 0.7446909948276748, 0.9997071369848932},
1425:          {0.6250371459823868, 0.7488381766552155, 0.9994160278693497},
1426:          {0.6304029756746347, 0.7529226373486367, 0.9990284511333963},
1427:          {0.6357575303538953, 0.7569437614144018, 0.9985445184894549},
1428:          {0.6411000028610946, 0.7609009397695506, 0.9979643565678705},
````
- **L1401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.49490286304267245, 0.6334972958235978, 0.9797475207722576},`.
  - **L1401 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.49490286304267245, 0.6334972958235978, 0.9797475207722576},`。
- **L1402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5003084046762621, 0.6389041252058836, 0.9816710071422271},`.
  - **L1402 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5003084046762621, 0.6389041252058836, 0.9816710071422271},`。
- **L1403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5057205821711058, 0.644263894767512, 0.983499671174965},`.
  - **L1403 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5057205821711058, 0.644263894767512, 0.983499671174965},`。
- **L1404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.511138732237593, 0.6495758744779339, 0.9852332712914448},`.
  - **L1404 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.511138732237593, 0.6495758744779339, 0.9852332712914448},`。
- **L1405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5165621819788595, 0.654839337759102, 0.9868715814280132},`.
  - **L1405 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5165621819788595, 0.654839337759102, 0.9868715814280132},`。
- **L1406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5219902492483212, 0.6600535616470082, 0.9884143910432847},`.
  - **L1406 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5219902492483212, 0.6600535616470082, 0.9884143910432847},`。
- **L1407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5274222430008886, 0.6652178269491371, 0.9898615051220324},`.
  - **L1407 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5274222430008886, 0.6652178269491371, 0.9898615051220324},`。
- **L1408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5328574636379119, 0.670331418398056, 0.9912127441760742},`.
  - **L1408 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5328574636379119, 0.670331418398056, 0.9912127441760742},`。
- **L1409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5382952033459485, 0.6753936248013475, 0.9924679442421503},`.
  - **L1409 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5382952033459485, 0.6753936248013475, 0.9924679442421503},`。
- **L1410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5437347464294153, 0.6804037391880692, 0.9936269568768029},`.
  - **L1410 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5437347464294153, 0.6804037391880692, 0.9936269568768029},`。
- **L1411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5491753696372063, 0.685361058951912, 0.9946896491482432},`.
  - **L1411 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5491753696372063, 0.685361058951912, 0.9946896491482432},`。
- **L1412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5546163424833603, 0.690264885991224, 0.9956559036252249},`.
  - **L1412 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5546163424833603, 0.690264885991224, 0.9956559036252249},`。
- **L1413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5600569275618472, 0.695114526846042, 0.9965256183629146},`.
  - **L1413 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5600569275618472, 0.695114526846042, 0.9965256183629146},`。
- **L1414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5654963808555731, 0.6999092928322662, 0.9972987068857656},`.
  - **L1414 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5654963808555731, 0.6999092928322662, 0.9972987068857656},`。
- **L1415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5709339520396719, 0.7046485001731109, 0.9979750981673984},`.
  - **L1415 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5709339520396719, 0.7046485001731109, 0.9979750981673984},`。
- **L1416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5763688847791901, 0.7093314701279408, 0.9985547366074979},`.
  - **L1416 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5763688847791901, 0.7093314701279408, 0.9985547366074979},`。
- **L1417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5818004170212294, 0.7139575291186055, 0.9990375820057199},`.
  - **L1417 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5818004170212294, 0.7139575291186055, 0.9990375820057199},`。
- **L1418 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5872277812816606, 0.7185260088533676, 0.9994236095326229},`.
  - **L1418 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5872277812816606, 0.7185260088533676, 0.9994236095326229},`。
- **L1419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5926502049264796, 0.7230362464485207, 0.9997128096976274},`.
  - **L1419 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5926502049264796, 0.7230362464485207, 0.9997128096976274},`。
- **L1420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5980669104479099, 0.7274875845477786, 0.9999051883140077},`.
  - **L1420 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5980669104479099, 0.7274875845477786, 0.9999051883140077},`。
- **L1421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.603477115735333, 0.7318793714395154, 1.0},`.
  - **L1421 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.603477115735333, 0.7318793714395154, 1.0},`。
- **L1422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6088800343411463, 0.7362109611719294, 0.9999995804425049},`.
  - **L1422 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6088800343411463, 0.7362109611719294, 0.9999995804425049},`。
- **L1423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6142748757416401, 0.7404817136661965, 0.9999016817439826},`.
  - **L1423 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6142748757416401, 0.7404817136661965, 0.9999016817439826},`。
- **L1424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6196608455929782, 0.7446909948276748, 0.9997071369848932},`.
  - **L1424 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6196608455929782, 0.7446909948276748, 0.9997071369848932},`。
- **L1425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6250371459823868, 0.7488381766552155, 0.9994160278693497},`.
  - **L1425 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6250371459823868, 0.7488381766552155, 0.9994160278693497},`。
- **L1426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6304029756746347, 0.7529226373486367, 0.9990284511333963},`.
  - **L1426 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6304029756746347, 0.7529226373486367, 0.9990284511333963},`。
- **L1427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6357575303538953, 0.7569437614144018, 0.9985445184894549},`.
  - **L1427 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6357575303538953, 0.7569437614144018, 0.9985445184894549},`。
- **L1428 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6411000028610946, 0.7609009397695506, 0.9979643565678705},`.
  - **L1428 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6411000028610946, 0.7609009397695506, 0.9979643565678705},`。

### Lines 1429-1456 / 第 1429-1456 行

````cpp
1429:          {0.6464295834268168, 0.7647935698439208, 0.9972881068555687},
1430:          {0.6517454598998733, 0.7686210556806996, 0.9965159256318358},
1431:          {0.6570468179716188, 0.7723828080353353, 0.9956479839012286},
1432:          {0.6623328413960967, 0.7760782444728409, 0.9946844673236305},
1433:          {0.6676027122061123, 0.7797067894635203, 0.9936255761414589},
1434:          {0.6728556109253114, 0.7832678744771355, 0.9924715251040422},
1435:          {0.6780907167763519, 0.7867609380755451, 0.9912225433891731},
1436:          {0.683307207885253, 0.7901854260038265, 0.9898788745218482},
1437:          {0.6885042614820046, 0.7935407912799094, 0.988440776290216},
1438:          {0.6936810540975126, 0.7968264942827246, 0.9869085206587308},
1439:          {0.6988367617569751, 0.8000420028388932, 0.9852823936785383},
1440:          {0.7039705601697447, 0.8031867923079613, 0.983562695395095},
1441:          {0.7090816249157784, 0.8062603456661956, 0.9817497397530396},
1442:          {0.714169131628733, 0.8092621535889422, 0.9798438544983238},
1443:          {0.7192322561757896, 0.8121917145315617, 0.9778453810776183},
1444:          {0.7242701748342745, 0.8150485348089448, 0.9757546745350029},
1445:          {0.7292820644651583, 0.8178321286736122, 0.9735721034059553},
1446:          {0.7342671026834836, 0.8205420183923986, 0.971298049608644},
1447:          {0.7392244680258144, 0.8231777343217284, 0.9689329083325428},
1448:          {0.7441533401147525, 0.8257388149814784, 0.9664770879243741},
1449:          {0.7490528998206014, 0.8282248071274302, 0.9639310097713906},
1450:          {0.7539223294202332, 0.8306352658223072, 0.9612951081820061},
1451:          {0.7587608127532254, 0.8329697545053927, 0.9585698302637836},
1452:          {0.7635675353753275, 0.8352278450607313, 0.9557556357987903},
1453:          {0.7683416847093205, 0.8374091178838983, 0.9528529971163268},
1454:          {0.7730824501933197, 0.8395131619473374, 0.9498623989630331},
1455:          {0.7777890234265885, 0.8415395748642601, 0.9467843383703861},
1456:          {0.7824605983129111, 0.8434879629510917, 0.9436193245195849},
````
- **L1429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6464295834268168, 0.7647935698439208, 0.9972881068555687},`.
  - **L1429 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6464295834268168, 0.7647935698439208, 0.9972881068555687},`。
- **L1430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6517454598998733, 0.7686210556806996, 0.9965159256318358},`.
  - **L1430 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6517454598998733, 0.7686210556806996, 0.9965159256318358},`。
- **L1431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6570468179716188, 0.7723828080353353, 0.9956479839012286},`.
  - **L1431 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6570468179716188, 0.7723828080353353, 0.9956479839012286},`。
- **L1432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6623328413960967, 0.7760782444728409, 0.9946844673236305},`.
  - **L1432 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6623328413960967, 0.7760782444728409, 0.9946844673236305},`。
- **L1433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6676027122061123, 0.7797067894635203, 0.9936255761414589},`.
  - **L1433 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6676027122061123, 0.7797067894635203, 0.9936255761414589},`。
- **L1434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6728556109253114, 0.7832678744771355, 0.9924715251040422},`.
  - **L1434 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6728556109253114, 0.7832678744771355, 0.9924715251040422},`。
- **L1435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6780907167763519, 0.7867609380755451, 0.9912225433891731},`.
  - **L1435 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6780907167763519, 0.7867609380755451, 0.9912225433891731},`。
- **L1436 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.683307207885253, 0.7901854260038265, 0.9898788745218482},`.
  - **L1436 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.683307207885253, 0.7901854260038265, 0.9898788745218482},`。
- **L1437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6885042614820046, 0.7935407912799094, 0.988440776290216},`.
  - **L1437 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6885042614820046, 0.7935407912799094, 0.988440776290216},`。
- **L1438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6936810540975126, 0.7968264942827246, 0.9869085206587308},`.
  - **L1438 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6936810540975126, 0.7968264942827246, 0.9869085206587308},`。
- **L1439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6988367617569751, 0.8000420028388932, 0.9852823936785383},`.
  - **L1439 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6988367617569751, 0.8000420028388932, 0.9852823936785383},`。
- **L1440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7039705601697447, 0.8031867923079613, 0.983562695395095},`.
  - **L1440 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7039705601697447, 0.8031867923079613, 0.983562695395095},`。
- **L1441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7090816249157784, 0.8062603456661956, 0.9817497397530396},`.
  - **L1441 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7090816249157784, 0.8062603456661956, 0.9817497397530396},`。
- **L1442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.714169131628733, 0.8092621535889422, 0.9798438544983238},`.
  - **L1442 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.714169131628733, 0.8092621535889422, 0.9798438544983238},`。
- **L1443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7192322561757896, 0.8121917145315617, 0.9778453810776183},`.
  - **L1443 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7192322561757896, 0.8121917145315617, 0.9778453810776183},`。
- **L1444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7242701748342745, 0.8150485348089448, 0.9757546745350029},`.
  - **L1444 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7242701748342745, 0.8150485348089448, 0.9757546745350029},`。
- **L1445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7292820644651583, 0.8178321286736122, 0.9735721034059553},`.
  - **L1445 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7292820644651583, 0.8178321286736122, 0.9735721034059553},`。
- **L1446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7342671026834836, 0.8205420183923986, 0.971298049608644},`.
  - **L1446 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7342671026834836, 0.8205420183923986, 0.971298049608644},`。
- **L1447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7392244680258144, 0.8231777343217284, 0.9689329083325428},`.
  - **L1447 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7392244680258144, 0.8231777343217284, 0.9689329083325428},`。
- **L1448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7441533401147525, 0.8257388149814784, 0.9664770879243741},`.
  - **L1448 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7441533401147525, 0.8257388149814784, 0.9664770879243741},`。
- **L1449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7490528998206014, 0.8282248071274302, 0.9639310097713906},`.
  - **L1449 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7490528998206014, 0.8282248071274302, 0.9639310097713906},`。
- **L1450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7539223294202332, 0.8306352658223072, 0.9612951081820061},`.
  - **L1450 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7539223294202332, 0.8306352658223072, 0.9612951081820061},`。
- **L1451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7587608127532254, 0.8329697545053927, 0.9585698302637836},`.
  - **L1451 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7587608127532254, 0.8329697545053927, 0.9585698302637836},`。
- **L1452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7635675353753275, 0.8352278450607313, 0.9557556357987903},`.
  - **L1452 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7635675353753275, 0.8352278450607313, 0.9557556357987903},`。
- **L1453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7683416847093205, 0.8374091178838983, 0.9528529971163268},`.
  - **L1453 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7683416847093205, 0.8374091178838983, 0.9528529971163268},`。
- **L1454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7730824501933197, 0.8395131619473374, 0.9498623989630331},`.
  - **L1454 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7730824501933197, 0.8395131619473374, 0.9498623989630331},`。
- **L1455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7777890234265885, 0.8415395748642601, 0.9467843383703861},`.
  - **L1455 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7777890234265885, 0.8415395748642601, 0.9467843383703861},`。
- **L1456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7824605983129111, 0.8434879629510917, 0.9436193245195849},`.
  - **L1456 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7824605983129111, 0.8434879629510917, 0.9436193245195849},`。

### Lines 1457-1484 / 第 1457-1484 行

````cpp
1457:          {0.7870963712015859, 0.8453579412884608, 0.9403678786038298},
1458:          {0.7916955410260809, 0.8471491337807205, 0.9370305336880028},
1459:          {0.7962573094404225, 0.8488611732139871, 0.933607834565744},
1460:          {0.8007808809533457, 0.8504937013126886, 0.9301003376139276},
1461:          {0.8052654630602748, 0.8520463687946024, 0.9265086106445327},
1462:          {0.8097102663731715, 0.8535188354243787, 0.9228332327539112},
1463:          {0.8141145047483038, 0.854910770065524, 0.9190747941694408},
1464:          {0.8184773954119764, 0.8562218507308311, 0.9152338960935597},
1465:          {0.822798159084272, 0.857451764631246, 0.9113111505451725},
1466:          {0.827076020100851, 0.8586002082231403, 0.9073071801984174},
1467:          {0.8313102065328385, 0.8596668872539794, 0.9032226182187731},
1468:          {0.8354999503048602, 0.8606515168063638, 0.8990581080965018},
1469:          {0.8396444873112505, 0.8615538213404197, 0.8948143034773921},
1470:          {0.8437430575304858, 0.8623735347345196, 0.8904918679907933},
1471:          {0.8477949051378751, 0.8631104003243038, 0.8860914750749006},
1472:          {0.8517992786165454, 0.8637641709399845, 0.8816138077992733},
1473:          {0.8557554308667662, 0.8643346089419015, 0.8770595586845422},
1474:          {0.8596626193136427, 0.8648214862543027, 0.8724294295192715},
1475:          {0.8635201060132168, 0.8652245843973202, 0.8677241311739308},
1476:          {0.8676431881102519, 0.8645140349150363, 0.8626161485985507},
1477:          {0.8719973001889246, 0.8626912788780988, 0.8571344116931553},
1478:          {0.8762551679574422, 0.8607866996599632, 0.8516174449794455},
1479:          {0.8804169678453379, 0.8588005566711197, 0.8460661683343939},
1480:          {0.8844828646854237, 0.8567331182372422, 0.8404815005112565},
1481:          {0.8884530127026752, 0.8545846614985785, 0.834864358985154},
1482:          {0.892327556446231, 0.8523554723065947, 0.829215659800157},
1483:          {0.8961066316684723, 0.8500458451177988, 0.8235363174178995},
1484:          {0.8997903661548088, 0.8476560828846528, 0.8178272445677435},
````
- **L1457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7870963712015859, 0.8453579412884608, 0.9403678786038298},`.
  - **L1457 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7870963712015859, 0.8453579412884608, 0.9403678786038298},`。
- **L1458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7916955410260809, 0.8471491337807205, 0.9370305336880028},`.
  - **L1458 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7916955410260809, 0.8471491337807205, 0.9370305336880028},`。
- **L1459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7962573094404225, 0.8488611732139871, 0.933607834565744},`.
  - **L1459 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7962573094404225, 0.8488611732139871, 0.933607834565744},`。
- **L1460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8007808809533457, 0.8504937013126886, 0.9301003376139276},`.
  - **L1460 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8007808809533457, 0.8504937013126886, 0.9301003376139276},`。
- **L1461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8052654630602748, 0.8520463687946024, 0.9265086106445327},`.
  - **L1461 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8052654630602748, 0.8520463687946024, 0.9265086106445327},`。
- **L1462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8097102663731715, 0.8535188354243787, 0.9228332327539112},`.
  - **L1462 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8097102663731715, 0.8535188354243787, 0.9228332327539112},`。
- **L1463 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8141145047483038, 0.854910770065524, 0.9190747941694408},`.
  - **L1463 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8141145047483038, 0.854910770065524, 0.9190747941694408},`。
- **L1464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8184773954119764, 0.8562218507308311, 0.9152338960935597},`.
  - **L1464 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8184773954119764, 0.8562218507308311, 0.9152338960935597},`。
- **L1465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.822798159084272, 0.857451764631246, 0.9113111505451725},`.
  - **L1465 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.822798159084272, 0.857451764631246, 0.9113111505451725},`。
- **L1466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.827076020100851, 0.8586002082231403, 0.9073071801984174},`.
  - **L1466 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.827076020100851, 0.8586002082231403, 0.9073071801984174},`。
- **L1467 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8313102065328385, 0.8596668872539794, 0.9032226182187731},`.
  - **L1467 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8313102065328385, 0.8596668872539794, 0.9032226182187731},`。
- **L1468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8354999503048602, 0.8606515168063638, 0.8990581080965018},`.
  - **L1468 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8354999503048602, 0.8606515168063638, 0.8990581080965018},`。
- **L1469 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8396444873112505, 0.8615538213404197, 0.8948143034773921},`.
  - **L1469 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8396444873112505, 0.8615538213404197, 0.8948143034773921},`。
- **L1470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8437430575304858, 0.8623735347345196, 0.8904918679907933},`.
  - **L1470 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8437430575304858, 0.8623735347345196, 0.8904918679907933},`。
- **L1471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8477949051378751, 0.8631104003243038, 0.8860914750749006},`.
  - **L1471 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8477949051378751, 0.8631104003243038, 0.8860914750749006},`。
- **L1472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8517992786165454, 0.8637641709399845, 0.8816138077992733},`.
  - **L1472 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8517992786165454, 0.8637641709399845, 0.8816138077992733},`。
- **L1473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8557554308667662, 0.8643346089419015, 0.8770595586845422},`.
  - **L1473 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8557554308667662, 0.8643346089419015, 0.8770595586845422},`。
- **L1474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8596626193136427, 0.8648214862543027, 0.8724294295192715},`.
  - **L1474 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8596626193136427, 0.8648214862543027, 0.8724294295192715},`。
- **L1475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8635201060132168, 0.8652245843973202, 0.8677241311739308},`.
  - **L1475 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8635201060132168, 0.8652245843973202, 0.8677241311739308},`。
- **L1476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8676431881102519, 0.8645140349150363, 0.8626161485985507},`.
  - **L1476 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8676431881102519, 0.8645140349150363, 0.8626161485985507},`。
- **L1477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8719973001889246, 0.8626912788780988, 0.8571344116931553},`.
  - **L1477 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8719973001889246, 0.8626912788780988, 0.8571344116931553},`。
- **L1478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8762551679574422, 0.8607866996599632, 0.8516174449794455},`.
  - **L1478 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8762551679574422, 0.8607866996599632, 0.8516174449794455},`。
- **L1479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8804169678453379, 0.8588005566711197, 0.8460661683343939},`.
  - **L1479 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8804169678453379, 0.8588005566711197, 0.8460661683343939},`。
- **L1480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8844828646854237, 0.8567331182372422, 0.8404815005112565},`.
  - **L1480 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8844828646854237, 0.8567331182372422, 0.8404815005112565},`。
- **L1481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8884530127026752, 0.8545846614985785, 0.834864358985154},`.
  - **L1481 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8884530127026752, 0.8545846614985785, 0.834864358985154},`。
- **L1482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.892327556446231, 0.8523554723065947, 0.829215659800157},`.
  - **L1482 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.892327556446231, 0.8523554723065947, 0.829215659800157},`。
- **L1483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8961066316684723, 0.8500458451177988, 0.8235363174178995},`.
  - **L1483 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8961066316684723, 0.8500458451177988, 0.8235363174178995},`。
- **L1484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8997903661548088, 0.8476560828846528, 0.8178272445677435},`.
  - **L1484 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8997903661548088, 0.8476560828846528, 0.8178272445677435},`。

### Lines 1485-1512 / 第 1485-1512 行

````cpp
1485:          {0.9033788805075311, 0.8451864969434798, 0.812089352098517},
1486:          {0.9068722888867912, 0.8426374068992736, 0.8063235488318476},
1487:          {0.910270699711566, 0.8400091405073019, 0.8005307414171138},
1488:          {0.9135742163232042, 0.8373020335514, 0.7947118341880348},
1489:          {0.9167829376139871, 0.8345164297188294, 0.7888677290209213},
1490:          {0.9198969586229171, 0.8316526804715908, 0.7829993251946056},
1491:          {0.9229163711008167, 0.8287111449140491, 0.7771075192520753},
1492:          {0.9258412640466325, 0.8256921896567365, 0.7711932048638259},
1493:          {0.9286717242167226, 0.822596188676188, 0.7652572726929551},
1494:          {0.9314078366087624, 0.8194235231706464, 0.7593006102620192},
1495:          {0.9340496849217929, 0.8161745814114749, 0.7533241018216675},
1496:          {0.9365973519938281, 0.8128497585900935, 0.7473286282210768},
1497:          {0.9390509202183217, 0.8094494566602553, 0.7413150667802011},
1498:          {0.9414104719407324, 0.8059740841754534, 0.7352842911638605},
1499:          {0.9436760898363059, 0.8024240561212462, 0.7292371712576793},
1500:          {0.9458478572701444, 0.7987997937422701, 0.7231745730458999},
1501:          {0.9479258586405416, 0.7951017243636899, 0.717097358491085},
1502:          {0.9499101797065052, 0.7913302812068249, 0.7110063854157246},
1503:          {0.951800907900325, 0.7874859031986702, 0.7049025073857709},
1504:          {0.953598132625986, 0.7835690347750057, 0.6987865735961123},
1505:          {0.9553019455441706, 0.7795801256767763, 0.6926594287580068},
1506:          {0.9569124408445594, 0.7755196307393911, 0.6865219129884912},
1507:          {0.9584297155060683, 0.7713880096745724, 0.6803748617017831},
1508:          {0.9598538695456499, 0.7671857268443542, 0.6742191055026909},
1509:          {0.9611850062562218, 0.7629132510268043, 0.6680554700820523},
1510:          {0.9624232324342575, 0.758571055173009, 0.6618847761142137},
1511:          {0.9635686585975486, 0.7541596161548229, 0.6557078391565709},
1512:          {0.9646213991936067, 0.7496794145028565, 0.6495254695511868},
````
- **L1485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9033788805075311, 0.8451864969434798, 0.812089352098517},`.
  - **L1485 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9033788805075311, 0.8451864969434798, 0.812089352098517},`。
- **L1486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9068722888867912, 0.8426374068992736, 0.8063235488318476},`.
  - **L1486 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9068722888867912, 0.8426374068992736, 0.8063235488318476},`。
- **L1487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.910270699711566, 0.8400091405073019, 0.8005307414171138},`.
  - **L1487 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.910270699711566, 0.8400091405073019, 0.8005307414171138},`。
- **L1488 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9135742163232042, 0.8373020335514, 0.7947118341880348},`.
  - **L1488 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9135742163232042, 0.8373020335514, 0.7947118341880348},`。
- **L1489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9167829376139871, 0.8345164297188294, 0.7888677290209213},`.
  - **L1489 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9167829376139871, 0.8345164297188294, 0.7888677290209213},`。
- **L1490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9198969586229171, 0.8316526804715908, 0.7829993251946056},`.
  - **L1490 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9198969586229171, 0.8316526804715908, 0.7829993251946056},`。
- **L1491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9229163711008167, 0.8287111449140491, 0.7771075192520753},`.
  - **L1491 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9229163711008167, 0.8287111449140491, 0.7771075192520753},`。
- **L1492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9258412640466325, 0.8256921896567365, 0.7711932048638259},`.
  - **L1492 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9258412640466325, 0.8256921896567365, 0.7711932048638259},`。
- **L1493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9286717242167226, 0.822596188676188, 0.7652572726929551},`.
  - **L1493 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9286717242167226, 0.822596188676188, 0.7652572726929551},`。
- **L1494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9314078366087624, 0.8194235231706464, 0.7593006102620192},`.
  - **L1494 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9314078366087624, 0.8194235231706464, 0.7593006102620192},`。
- **L1495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9340496849217929, 0.8161745814114749, 0.7533241018216675},`.
  - **L1495 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9340496849217929, 0.8161745814114749, 0.7533241018216675},`。
- **L1496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9365973519938281, 0.8128497585900935, 0.7473286282210768},`.
  - **L1496 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9365973519938281, 0.8128497585900935, 0.7473286282210768},`。
- **L1497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9390509202183217, 0.8094494566602553, 0.7413150667802011},`.
  - **L1497 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9390509202183217, 0.8094494566602553, 0.7413150667802011},`。
- **L1498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9414104719407324, 0.8059740841754534, 0.7352842911638605},`.
  - **L1498 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9414104719407324, 0.8059740841754534, 0.7352842911638605},`。
- **L1499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9436760898363059, 0.8024240561212462, 0.7292371712576793},`.
  - **L1499 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9436760898363059, 0.8024240561212462, 0.7292371712576793},`。
- **L1500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9458478572701444, 0.7987997937422701, 0.7231745730458999},`.
  - **L1500 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9458478572701444, 0.7987997937422701, 0.7231745730458999},`。
- **L1501 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9479258586405416, 0.7951017243636899, 0.717097358491085},`.
  - **L1501 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9479258586405416, 0.7951017243636899, 0.717097358491085},`。
- **L1502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9499101797065052, 0.7913302812068249, 0.7110063854157246},`.
  - **L1502 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9499101797065052, 0.7913302812068249, 0.7110063854157246},`。
- **L1503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.951800907900325, 0.7874859031986702, 0.7049025073857709},`.
  - **L1503 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.951800907900325, 0.7874859031986702, 0.7049025073857709},`。
- **L1504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.953598132625986, 0.7835690347750057, 0.6987865735961123},`.
  - **L1504 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.953598132625986, 0.7835690347750057, 0.6987865735961123},`。
- **L1505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9553019455441706, 0.7795801256767763, 0.6926594287580068},`.
  - **L1505 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9553019455441706, 0.7795801256767763, 0.6926594287580068},`。
- **L1506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9569124408445594, 0.7755196307393911, 0.6865219129884912},`.
  - **L1506 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9569124408445594, 0.7755196307393911, 0.6865219129884912},`。
- **L1507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9584297155060683, 0.7713880096745724, 0.6803748617017831},`.
  - **L1507 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9584297155060683, 0.7713880096745724, 0.6803748617017831},`。
- **L1508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9598538695456499, 0.7671857268443542, 0.6742191055026909},`.
  - **L1508 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9598538695456499, 0.7671857268443542, 0.6742191055026909},`。
- **L1509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9611850062562218, 0.7629132510268043, 0.6680554700820523},`.
  - **L1509 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9611850062562218, 0.7629132510268043, 0.6680554700820523},`。
- **L1510 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9624232324342575, 0.758571055173009, 0.6618847761142137},`.
  - **L1510 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9624232324342575, 0.758571055173009, 0.6618847761142137},`。
- **L1511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9635686585975486, 0.7541596161548229, 0.6557078391565709},`.
  - **L1511 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9635686585975486, 0.7541596161548229, 0.6557078391565709},`。
- **L1512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9646213991936067, 0.7496794145028565, 0.6495254695511868},`.
  - **L1512 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9646213991936067, 0.7496794145028565, 0.6495254695511868},`。

### Lines 1513-1540 / 第 1513-1540 行

````cpp
1513:          {0.9655815727991436, 0.7451309341341271, 0.6433384723285004},
1514:          {0.9664493023110522, 0.7405146620687538, 0.6371476471131512},
1515:          {0.9672247151292687, 0.7358310881350345, 0.6309537880319284},
1516:          {0.9679079433318883, 0.7310807046621858, 0.6247576836238656},
1517:          {0.9684991238428787, 0.7262640061599712, 0.6185601167525033},
1518:          {0.9689983985927076, 0.7213814889843815, 0.612361864520328},
1519:          {0.9694059146721955, 0.7164336509884588, 0.6061636981854146},
1520:          {0.9697218244798756, 0.7114209911572899, 0.5999663830802852},
1521:          {0.9699462858631324, 0.7063440092261024, 0.5937706785330095},
1522:          {0.9700794622533694, 0.7012032052803152, 0.5875773377905578},
1523:          {0.9701215227954475, 0.6959990793362977, 0.5813871079444363},
1524:          {0.9700726424716136, 0.6907321309014792, 0.5752007298586196},
1525:          {0.9699330022201408, 0.6854028585123331, 0.5690189380998024},
1526:          {0.9697027890488664, 0.6800117592486405, 0.5628424608699958},
1527:          {0.9693821961438281, 0.6745593282222776, 0.556672019941486},
1528:          {0.9689714229731695, 0.6690460580386323, 0.5505083305941867},
1529:          {0.968470675386481, 0.663472438228566, 0.5443521015554007},
1530:          {0.9678801657097379, 0.65783895464866, 0.5382040349420275},
1531:          {0.9672001128359847, 0.6521460888472544, 0.5320648262052324},
1532:          {0.9664307423119013, 0.6463943173935754, 0.5259351640776159},
1533:          {0.9655722864203883, 0.6405841111669635, 0.5198157305229032},
1534:          {0.964624984259297, 0.6347159346029446, 0.5137072006881956},
1535:          {0.9635890818164178, 0.6287902448925465, 0.507610242858806},
1536:          {0.9624648320408441, 0.6228074911309212, 0.5015255184157178},
1537:          {0.9612524949108144, 0.6167681134109175, 0.4954536817957061},
1538:          {0.9599523374981328, 0.6106725418568123, 0.4893953804541488},
1539:          {0.9585646340292623, 0.6045211955929051, 0.4833512548305818},
1540:          {0.9570896659431806, 0.5983144816411149, 0.4773219383170267},
````
- **L1513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9655815727991436, 0.7451309341341271, 0.6433384723285004},`.
  - **L1513 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9655815727991436, 0.7451309341341271, 0.6433384723285004},`。
- **L1514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9664493023110522, 0.7405146620687538, 0.6371476471131512},`.
  - **L1514 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9664493023110522, 0.7405146620687538, 0.6371476471131512},`。
- **L1515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9672247151292687, 0.7358310881350345, 0.6309537880319284},`.
  - **L1515 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9672247151292687, 0.7358310881350345, 0.6309537880319284},`。
- **L1516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9679079433318883, 0.7310807046621858, 0.6247576836238656},`.
  - **L1516 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9679079433318883, 0.7310807046621858, 0.6247576836238656},`。
- **L1517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9684991238428787, 0.7262640061599712, 0.6185601167525033},`.
  - **L1517 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9684991238428787, 0.7262640061599712, 0.6185601167525033},`。
- **L1518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9689983985927076, 0.7213814889843815, 0.612361864520328},`.
  - **L1518 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9689983985927076, 0.7213814889843815, 0.612361864520328},`。
- **L1519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9694059146721955, 0.7164336509884588, 0.6061636981854146},`.
  - **L1519 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9694059146721955, 0.7164336509884588, 0.6061636981854146},`。
- **L1520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9697218244798756, 0.7114209911572899, 0.5999663830802852},`.
  - **L1520 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9697218244798756, 0.7114209911572899, 0.5999663830802852},`。
- **L1521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9699462858631324, 0.7063440092261024, 0.5937706785330095},`.
  - **L1521 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9699462858631324, 0.7063440092261024, 0.5937706785330095},`。
- **L1522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9700794622533694, 0.7012032052803152, 0.5875773377905578},`.
  - **L1522 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9700794622533694, 0.7012032052803152, 0.5875773377905578},`。
- **L1523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9701215227954475, 0.6959990793362977, 0.5813871079444363},`.
  - **L1523 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9701215227954475, 0.6959990793362977, 0.5813871079444363},`。
- **L1524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9700726424716136, 0.6907321309014792, 0.5752007298586196},`.
  - **L1524 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9700726424716136, 0.6907321309014792, 0.5752007298586196},`。
- **L1525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9699330022201408, 0.6854028585123331, 0.5690189380998024},`.
  - **L1525 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9699330022201408, 0.6854028585123331, 0.5690189380998024},`。
- **L1526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9697027890488664, 0.6800117592486405, 0.5628424608699958},`.
  - **L1526 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9697027890488664, 0.6800117592486405, 0.5628424608699958},`。
- **L1527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9693821961438281, 0.6745593282222776, 0.556672019941486},`.
  - **L1527 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9693821961438281, 0.6745593282222776, 0.556672019941486},`。
- **L1528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9689714229731695, 0.6690460580386323, 0.5505083305941867},`.
  - **L1528 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9689714229731695, 0.6690460580386323, 0.5505083305941867},`。
- **L1529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.968470675386481, 0.663472438228566, 0.5443521015554007},`.
  - **L1529 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.968470675386481, 0.663472438228566, 0.5443521015554007},`。
- **L1530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9678801657097379, 0.65783895464866, 0.5382040349420275},`.
  - **L1530 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9678801657097379, 0.65783895464866, 0.5382040349420275},`。
- **L1531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9672001128359847, 0.6521460888472544, 0.5320648262052324},`.
  - **L1531 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9672001128359847, 0.6521460888472544, 0.5320648262052324},`。
- **L1532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9664307423119013, 0.6463943173935754, 0.5259351640776159},`.
  - **L1532 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9664307423119013, 0.6463943173935754, 0.5259351640776159},`。
- **L1533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9655722864203883, 0.6405841111669635, 0.5198157305229032},`.
  - **L1533 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9655722864203883, 0.6405841111669635, 0.5198157305229032},`。
- **L1534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.964624984259297, 0.6347159346029446, 0.5137072006881956},`.
  - **L1534 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.964624984259297, 0.6347159346029446, 0.5137072006881956},`。
- **L1535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9635890818164178, 0.6287902448925465, 0.507610242858806},`.
  - **L1535 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9635890818164178, 0.6287902448925465, 0.507610242858806},`。
- **L1536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9624648320408441, 0.6228074911309212, 0.5015255184157178},`.
  - **L1536 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9624648320408441, 0.6228074911309212, 0.5015255184157178},`。
- **L1537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9612524949108144, 0.6167681134109175, 0.4954536817957061},`.
  - **L1537 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9612524949108144, 0.6167681134109175, 0.4954536817957061},`。
- **L1538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9599523374981328, 0.6106725418568123, 0.4893953804541488},`.
  - **L1538 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9599523374981328, 0.6106725418568123, 0.4893953804541488},`。
- **L1539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9585646340292623, 0.6045211955929051, 0.4833512548305818},`.
  - **L1539 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9585646340292623, 0.6045211955929051, 0.4833512548305818},`。
- **L1540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9570896659431806, 0.5983144816411149, 0.4773219383170267},`.
  - **L1540 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9570896659431806, 0.5983144816411149, 0.4773219383170267},`。

### Lines 1541-1568 / 第 1541-1568 行

````cpp
1541:          {0.9555277219460815, 0.5920527937410947, 0.47130805722914443},
1542:          {0.9538790980630029, 0.5857365110856726, 0.4653102307802585},
1543:          {0.9521440976864516, 0.5793659969636203, 0.4593290710582975},
1544:          {0.9503230316221086, 0.57294159730086, 0.4533651830057083},
1545:          {0.948416218131665, 0.5664636390902013, 0.4474191644024001},
1546:          {0.9464239829728659, 0.5599324286985411, 0.44149160585177294},
1547:          {0.9443466594368136, 0.5533482500391552, 0.4355830907698924},
1548:          {0.9421845883825966, 0.546711362595224, 0.42969419537788606},
1549:          {0.9399381182692833, 0.5400219992790316, 0.4238254886976179},
1550:          {0.9376076051853501, 0.533280364109342, 0.4179775325507263},
1551:          {0.9351934128755736, 0.5264866296872404, 0.4121508815610997},
1552:          {0.9326959127654463, 0.5196409344481697, 0.40634608316087356},
1553:          {0.9301154839831494, 0.5127433796649743, 0.40056367760003836},
1554:          {0.9274525133791317, 0.5057940261733677, 0.39480419795975347},
1555:          {0.9247073955433249, 0.49879289078734274, 0.38906817016946776},
1556:          {0.9218805328200373, 0.4917399423674949, 0.38335611302795025},
1557:          {0.9189723353205592, 0.4846350974999652, 0.37766853822834884},
1558:          {0.9159832209335118, 0.47747821573754495, 0.37200595038739454},
1559:          {0.9129136153329698, 0.47026909434728564, 0.3663688470788784},
1560:          {0.90976395198439, 0.46300746250050845, 0.36075771887154084},
1561:          {0.9065346721483711, 0.4556929748311352, 0.35517304937151345},
1562:          {0.9032262248822704, 0.44832520427650463, 0.34961531526947287},
1563:          {0.8998390670397057, 0.44090363410085304, 0.34408498639266283},
1564:          {0.896373663267962, 0.43342764898501834, 0.3385825257619646},
1565:          {0.8928304860033279, 0.42589652504602493, 0.33310838965420186},
1566:          {0.8892100154643842, 0.41830941862630455, 0.32766302766986616},
1567:          {0.8855127396432578, 0.4106653536635276, 0.32224688280648867},
1568:          {0.8817391542948727, 0.4029632074170497, 0.31686039153786477},
````
- **L1541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9555277219460815, 0.5920527937410947, 0.47130805722914443},`.
  - **L1541 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9555277219460815, 0.5920527937410947, 0.47130805722914443},`。
- **L1542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9538790980630029, 0.5857365110856726, 0.4653102307802585},`.
  - **L1542 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9538790980630029, 0.5857365110856726, 0.4653102307802585},`。
- **L1543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9521440976864516, 0.5793659969636203, 0.4593290710582975},`.
  - **L1543 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9521440976864516, 0.5793659969636203, 0.4593290710582975},`。
- **L1544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9503230316221086, 0.57294159730086, 0.4533651830057083},`.
  - **L1544 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9503230316221086, 0.57294159730086, 0.4533651830057083},`。
- **L1545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.948416218131665, 0.5664636390902013, 0.4474191644024001},`.
  - **L1545 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.948416218131665, 0.5664636390902013, 0.4474191644024001},`。
- **L1546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9464239829728659, 0.5599324286985411, 0.44149160585177294},`.
  - **L1546 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9464239829728659, 0.5599324286985411, 0.44149160585177294},`。
- **L1547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9443466594368136, 0.5533482500391552, 0.4355830907698924},`.
  - **L1547 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9443466594368136, 0.5533482500391552, 0.4355830907698924},`。
- **L1548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9421845883825966, 0.546711362595224, 0.42969419537788606},`.
  - **L1548 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9421845883825966, 0.546711362595224, 0.42969419537788606},`。
- **L1549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9399381182692833, 0.5400219992790316, 0.4238254886976179},`.
  - **L1549 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9399381182692833, 0.5400219992790316, 0.4238254886976179},`。
- **L1550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9376076051853501, 0.533280364109342, 0.4179775325507263},`.
  - **L1550 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9376076051853501, 0.533280364109342, 0.4179775325507263},`。
- **L1551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9351934128755736, 0.5264866296872404, 0.4121508815610997},`.
  - **L1551 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9351934128755736, 0.5264866296872404, 0.4121508815610997},`。
- **L1552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9326959127654463, 0.5196409344481697, 0.40634608316087356},`.
  - **L1552 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9326959127654463, 0.5196409344481697, 0.40634608316087356},`。
- **L1553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9301154839831494, 0.5127433796649743, 0.40056367760003836},`.
  - **L1553 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9301154839831494, 0.5127433796649743, 0.40056367760003836},`。
- **L1554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9274525133791317, 0.5057940261733677, 0.39480419795975347},`.
  - **L1554 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9274525133791317, 0.5057940261733677, 0.39480419795975347},`。
- **L1555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9247073955433249, 0.49879289078734274, 0.38906817016946776},`.
  - **L1555 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9247073955433249, 0.49879289078734274, 0.38906817016946776},`。
- **L1556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9218805328200373, 0.4917399423674949, 0.38335611302795025},`.
  - **L1556 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9218805328200373, 0.4917399423674949, 0.38335611302795025},`。
- **L1557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9189723353205592, 0.4846350974999652, 0.37766853822834884},`.
  - **L1557 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9189723353205592, 0.4846350974999652, 0.37766853822834884},`。
- **L1558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9159832209335118, 0.47747821573754495, 0.37200595038739454},`.
  - **L1558 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9159832209335118, 0.47747821573754495, 0.37200595038739454},`。
- **L1559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9129136153329698, 0.47026909434728564, 0.3663688470788784},`.
  - **L1559 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9129136153329698, 0.47026909434728564, 0.3663688470788784},`。
- **L1560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.90976395198439, 0.46300746250050845, 0.36075771887154084},`.
  - **L1560 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.90976395198439, 0.46300746250050845, 0.36075771887154084},`。
- **L1561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9065346721483711, 0.4556929748311352, 0.35517304937151345},`.
  - **L1561 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9065346721483711, 0.4556929748311352, 0.35517304937151345},`。
- **L1562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9032262248822704, 0.44832520427650463, 0.34961531526947287},`.
  - **L1562 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9032262248822704, 0.44832520427650463, 0.34961531526947287},`。
- **L1563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8998390670397057, 0.44090363410085304, 0.34408498639266283},`.
  - **L1563 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8998390670397057, 0.44090363410085304, 0.34408498639266283},`。
- **L1564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.896373663267962, 0.43342764898501834, 0.3385825257619646},`.
  - **L1564 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.896373663267962, 0.43342764898501834, 0.3385825257619646},`。
- **L1565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8928304860033279, 0.42589652504602493, 0.33310838965420186},`.
  - **L1565 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8928304860033279, 0.42589652504602493, 0.33310838965420186},`。
- **L1566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8892100154643842, 0.41830941862630455, 0.32766302766986616},`.
  - **L1566 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8892100154643842, 0.41830941862630455, 0.32766302766986616},`。
- **L1567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8855127396432578, 0.4106653536635276, 0.32224688280648867},`.
  - **L1567 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8855127396432578, 0.4106653536635276, 0.32224688280648867},`。
- **L1568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8817391542948727, 0.4029632074170497, 0.31686039153786477},`.
  - **L1568 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8817391542948727, 0.4029632074170497, 0.31686039153786477},`。

### Lines 1569-1596 / 第 1569-1596 行

````cpp
1569:          {0.8778897629242048, 0.3952016942845104, 0.3115039838993836},
1570:          {0.8739650767715597, 0.3873793473900178, 0.30617808357969667},
1571:          {0.8699656147959004, 0.37949449756134346, 0.30088310801901375},
1572:          {0.8658919036562271, 0.37154524923423027, 0.2956194685142945},
1573:          {0.8617444776910301, 0.3635294527231879, 0.29038757033164536},
1574:          {0.8575238788958367, 0.35544467217440223, 0.2851878128262474},
1575:          {0.8532306568988527, 0.3472881483602095, 0.2800205895701496},
1576:          {0.8488653689347272, 0.33905675527607415, 0.27488628848829755},
1577:          {0.844428579816439, 0.33074694924673165, 0.2697852920031808},
1578:          {0.8399208619053304, 0.3223547089197083, 0.26471797718851403},
1579:          {0.835342795079293, 0.3138754640964424, 0.2596847159323806},
1580:          {0.8306949666991194, 0.30530401078823033, 0.25468587511031116},
1581:          {0.825977971573034, 0.2966344091359757, 0.24972181676877747},
1582:          {0.8211924119194148, 0.28785985982733997, 0.2447928983196327},
1583:          {0.8163388973277163, 0.2789725532777859, 0.23989947274604145},
1584:          {0.8114180447176049, 0.2699634839588782, 0.2350418888204895},
1585:          {0.8064304782963232, 0.2608222196264688, 0.230220491335487},
1586:          {0.8013768295142913, 0.25153661146923556, 0.2254356213476219},
1587:          {0.7962577370189533, 0.24209242581295431, 0.22068761643565016},
1588:          {0.7910738466068946, 0.2324728700992567, 0.21597681097335308},
1589:          {0.785825811174225, 0.22265797397565007, 0.2113035364179236},
1590:          {0.7805142906652605, 0.21262376808227257, 0.20666812161469034},
1591:          {0.7751399520194979, 0.20234117434687338, 0.2020708931190197},
1592:          {0.7697034691169152, 0.19177447487857652, 0.19751217553628297},
1593:          {0.7642055227215985, 0.18087914808905534, 0.19299229188080813},
1594:          {0.7586468004237221, 0.16959872367321424, 0.18851156395477772},
1595:          {0.7530279965798936, 0.15786005774647732, 0.18407031274806854},
1596:          {0.7473498122518853, 0.1455659466237694, 0.17966885886006054},
````
- **L1569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8778897629242048, 0.3952016942845104, 0.3115039838993836},`.
  - **L1569 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8778897629242048, 0.3952016942845104, 0.3115039838993836},`。
- **L1570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8739650767715597, 0.3873793473900178, 0.30617808357969667},`.
  - **L1570 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8739650767715597, 0.3873793473900178, 0.30617808357969667},`。
- **L1571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8699656147959004, 0.37949449756134346, 0.30088310801901375},`.
  - **L1571 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8699656147959004, 0.37949449756134346, 0.30088310801901375},`。
- **L1572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8658919036562271, 0.37154524923423027, 0.2956194685142945},`.
  - **L1572 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8658919036562271, 0.37154524923423027, 0.2956194685142945},`。
- **L1573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8617444776910301, 0.3635294527231879, 0.29038757033164536},`.
  - **L1573 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8617444776910301, 0.3635294527231879, 0.29038757033164536},`。
- **L1574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8575238788958367, 0.35544467217440223, 0.2851878128262474},`.
  - **L1574 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8575238788958367, 0.35544467217440223, 0.2851878128262474},`。
- **L1575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8532306568988527, 0.3472881483602095, 0.2800205895701496},`.
  - **L1575 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8532306568988527, 0.3472881483602095, 0.2800205895701496},`。
- **L1576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8488653689347272, 0.33905675527607415, 0.27488628848829755},`.
  - **L1576 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8488653689347272, 0.33905675527607415, 0.27488628848829755},`。
- **L1577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.844428579816439, 0.33074694924673165, 0.2697852920031808},`.
  - **L1577 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.844428579816439, 0.33074694924673165, 0.2697852920031808},`。
- **L1578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8399208619053304, 0.3223547089197083, 0.26471797718851403},`.
  - **L1578 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8399208619053304, 0.3223547089197083, 0.26471797718851403},`。
- **L1579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.835342795079293, 0.3138754640964424, 0.2596847159323806},`.
  - **L1579 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.835342795079293, 0.3138754640964424, 0.2596847159323806},`。
- **L1580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8306949666991194, 0.30530401078823033, 0.25468587511031116},`.
  - **L1580 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8306949666991194, 0.30530401078823033, 0.25468587511031116},`。
- **L1581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.825977971573034, 0.2966344091359757, 0.24972181676877747},`.
  - **L1581 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.825977971573034, 0.2966344091359757, 0.24972181676877747},`。
- **L1582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8211924119194148, 0.28785985982733997, 0.2447928983196327},`.
  - **L1582 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8211924119194148, 0.28785985982733997, 0.2447928983196327},`。
- **L1583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8163388973277163, 0.2789725532777859, 0.23989947274604145},`.
  - **L1583 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8163388973277163, 0.2789725532777859, 0.23989947274604145},`。
- **L1584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8114180447176049, 0.2699634839588782, 0.2350418888204895},`.
  - **L1584 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8114180447176049, 0.2699634839588782, 0.2350418888204895},`。
- **L1585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8064304782963232, 0.2608222196264688, 0.230220491335487},`.
  - **L1585 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8064304782963232, 0.2608222196264688, 0.230220491335487},`。
- **L1586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8013768295142913, 0.25153661146923556, 0.2254356213476219},`.
  - **L1586 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8013768295142913, 0.25153661146923556, 0.2254356213476219},`。
- **L1587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7962577370189533, 0.24209242581295431, 0.22068761643565016},`.
  - **L1587 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7962577370189533, 0.24209242581295431, 0.22068761643565016},`。
- **L1588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7910738466068946, 0.2324728700992567, 0.21597681097335308},`.
  - **L1588 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7910738466068946, 0.2324728700992567, 0.21597681097335308},`。
- **L1589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.785825811174225, 0.22265797397565007, 0.2113035364179236},`.
  - **L1589 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.785825811174225, 0.22265797397565007, 0.2113035364179236},`。
- **L1590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7805142906652605, 0.21262376808227257, 0.20666812161469034},`.
  - **L1590 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7805142906652605, 0.21262376808227257, 0.20666812161469034},`。
- **L1591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7751399520194979, 0.20234117434687338, 0.2020708931190197},`.
  - **L1591 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7751399520194979, 0.20234117434687338, 0.2020708931190197},`。
- **L1592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7697034691169152, 0.19177447487857652, 0.19751217553628297},`.
  - **L1592 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7697034691169152, 0.19177447487857652, 0.19751217553628297},`。
- **L1593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7642055227215985, 0.18087914808905534, 0.19299229188080813},`.
  - **L1593 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7642055227215985, 0.18087914808905534, 0.19299229188080813},`。
- **L1594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7586468004237221, 0.16959872367321424, 0.18851156395477772},`.
  - **L1594 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7586468004237221, 0.16959872367321424, 0.18851156395477772},`。
- **L1595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7530279965798936, 0.15786005774647732, 0.18407031274806854},`.
  - **L1595 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7530279965798936, 0.15786005774647732, 0.18407031274806854},`。
- **L1596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7473498122518853, 0.1455659466237694, 0.17966885886006054},`.
  - **L1596 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7473498122518853, 0.1455659466237694, 0.17966885886006054},`。

### Lines 1597-1624 / 第 1597-1624 行

````cpp
1597:          {0.7416129551437725, 0.13258300379739485, 0.1753075229444704},
1598:          {0.7358181395374971, 0.11872050682883606, 0.17098662617829208},
1599:          {0.729966086226886, 0.1036904371731237, 0.16670649075593325},
1600:          {0.7240575224501452, 0.08702339888581809, 0.1624674404096517},
1601:          {0.7180931818208612, 0.06786174152286231, 0.15826980095738155},
1602:          {0.712073804257538, 0.04430319728532283, 0.15411390087901955},
1603:          {0.7060001359117047, 0.015991824033980695, 0.15000007192220008}}};
1604: 
1605: template <typename Real>
1606: static constexpr std::array<std::array<Real, 3>, 256> viridis_data_ = {
1607:     {{0.2670039853213788, 0.0048725657145795975, 0.32941506855247793},
1608:      {0.26850981914385313, 0.009602990407952114, 0.33542640725404194},
1609:      {0.2699440291511295, 0.014623657659867702, 0.34137927634304566},
1610:      {0.271304877824855, 0.01994002237673082, 0.3472686291318146},
1611:      {0.27259406260318486, 0.0255617891189931, 0.3530934750332732},
1612:      {0.27380892621369024, 0.03149509118386225, 0.3588528426972369},
1613:      {0.27495208578509805, 0.03775096330071057, 0.36454366367333946},
1614:      {0.27602196419599767, 0.04416532238909473, 0.3701640471152313},
1615:      {0.2770178542655486, 0.050341751402647274, 0.37571443702683854},
1616:      {0.2779409905222335, 0.05632269668695442, 0.381191238867294},
1617:      {0.2787908952957126, 0.06214312525024266, 0.386591645366443},
1618:      {0.27956600607518295, 0.06783500475980075, 0.39191741935399227},
1619:      {0.2802669260610985, 0.07341543578584622, 0.3971628433351474},
1620:      {0.2808940110629919, 0.07890627296909186, 0.402329588370124},
1621:      {0.2814459462121492, 0.08431870698834085, 0.4074140304183279},
1622:      {0.2819238934697361, 0.08966415929405766, 0.4124144814203868},
1623:      {0.2823269547340802, 0.09495395414045926, 0.4173312035070467},
1624:      {0.28265591676374746, 0.10019440706631136, 0.42215967285462885},
````
- **L1597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7416129551437725, 0.13258300379739485, 0.1753075229444704},`.
  - **L1597 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7416129551437725, 0.13258300379739485, 0.1753075229444704},`。
- **L1598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7358181395374971, 0.11872050682883606, 0.17098662617829208},`.
  - **L1598 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7358181395374971, 0.11872050682883606, 0.17098662617829208},`。
- **L1599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.729966086226886, 0.1036904371731237, 0.16670649075593325},`.
  - **L1599 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.729966086226886, 0.1036904371731237, 0.16670649075593325},`。
- **L1600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7240575224501452, 0.08702339888581809, 0.1624674404096517},`.
  - **L1600 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7240575224501452, 0.08702339888581809, 0.1624674404096517},`。
- **L1601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7180931818208612, 0.06786174152286231, 0.15826980095738155},`.
  - **L1601 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7180931818208612, 0.06786174152286231, 0.15826980095738155},`。
- **L1602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.712073804257538, 0.04430319728532283, 0.15411390087901955},`.
  - **L1602 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.712073804257538, 0.04430319728532283, 0.15411390087901955},`。
- **L1603 EN**: Executes a standalone statement or declaration: `{0.7060001359117047, 0.015991824033980695, 0.15000007192220008}}};`.
  - **L1603 CN**: 执行一条独立语句或声明：`{0.7060001359117047, 0.015991824033980695, 0.15000007192220008}}};`。
- **L1604 EN**: Blank line separating nearby declarations or logic.
  - **L1604 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1605 EN**: Introduces template parameters or specialization context: `template <typename Real>`.
  - **L1605 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real>`。
- **L1606 EN**: Uses standard-library type traits or utility templates to shape generic behavior.
  - **L1606 CN**: 使用标准库类型 traits 或工具模板来塑造泛型行为。
- **L1607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{{0.2670039853213788, 0.0048725657145795975, 0.32941506855247793},`.
  - **L1607 CN**: 继续一个多行参数列表、初始化器或聚合项：`{{0.2670039853213788, 0.0048725657145795975, 0.32941506855247793},`。
- **L1608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.26850981914385313, 0.009602990407952114, 0.33542640725404194},`.
  - **L1608 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.26850981914385313, 0.009602990407952114, 0.33542640725404194},`。
- **L1609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2699440291511295, 0.014623657659867702, 0.34137927634304566},`.
  - **L1609 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2699440291511295, 0.014623657659867702, 0.34137927634304566},`。
- **L1610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.271304877824855, 0.01994002237673082, 0.3472686291318146},`.
  - **L1610 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.271304877824855, 0.01994002237673082, 0.3472686291318146},`。
- **L1611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27259406260318486, 0.0255617891189931, 0.3530934750332732},`.
  - **L1611 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27259406260318486, 0.0255617891189931, 0.3530934750332732},`。
- **L1612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27380892621369024, 0.03149509118386225, 0.3588528426972369},`.
  - **L1612 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27380892621369024, 0.03149509118386225, 0.3588528426972369},`。
- **L1613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27495208578509805, 0.03775096330071057, 0.36454366367333946},`.
  - **L1613 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27495208578509805, 0.03775096330071057, 0.36454366367333946},`。
- **L1614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27602196419599767, 0.04416532238909473, 0.3701640471152313},`.
  - **L1614 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27602196419599767, 0.04416532238909473, 0.3701640471152313},`。
- **L1615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2770178542655486, 0.050341751402647274, 0.37571443702683854},`.
  - **L1615 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2770178542655486, 0.050341751402647274, 0.37571443702683854},`。
- **L1616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2779409905222335, 0.05632269668695442, 0.381191238867294},`.
  - **L1616 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2779409905222335, 0.05632269668695442, 0.381191238867294},`。
- **L1617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2787908952957126, 0.06214312525024266, 0.386591645366443},`.
  - **L1617 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2787908952957126, 0.06214312525024266, 0.386591645366443},`。
- **L1618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27956600607518295, 0.06783500475980075, 0.39191741935399227},`.
  - **L1618 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27956600607518295, 0.06783500475980075, 0.39191741935399227},`。
- **L1619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2802669260610985, 0.07341543578584622, 0.3971628433351474},`.
  - **L1619 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2802669260610985, 0.07341543578584622, 0.3971628433351474},`。
- **L1620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2808940110629919, 0.07890627296909186, 0.402329588370124},`.
  - **L1620 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2808940110629919, 0.07890627296909186, 0.402329588370124},`。
- **L1621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2814459462121492, 0.08431870698834085, 0.4074140304183279},`.
  - **L1621 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2814459462121492, 0.08431870698834085, 0.4074140304183279},`。
- **L1622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2819238934697361, 0.08966415929405766, 0.4124144814203868},`.
  - **L1622 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2819238934697361, 0.08966415929405766, 0.4124144814203868},`。
- **L1623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2823269547340802, 0.09495395414045926, 0.4173312035070467},`.
  - **L1623 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2823269547340802, 0.09495395414045926, 0.4173312035070467},`。
- **L1624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.28265591676374746, 0.10019440706631136, 0.42215967285462885},`.
  - **L1624 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.28265591676374746, 0.10019440706631136, 0.42215967285462885},`。

### Lines 1625-1652 / 第 1625-1652 行

````cpp
1625:      {0.2829099523467409, 0.10539218382981966, 0.42690236327520725},
1626:      {0.2830909293924702, 0.11055163703080462, 0.43155385240026206},
1627:      {0.2831969393871863, 0.11567940038180258, 0.4361155100499537},
1628:      {0.2832289314889045, 0.1207758539544326, 0.4405840195472734},
1629:      {0.2831869362255922, 0.1258463172454443, 0.4449595399107695},
1630:      {0.28307192228564804, 0.13089406218358995, 0.4492411716459423},
1631:      {0.28288394120635924, 0.1359185257084681, 0.45342671165618736},
1632:      {0.2826229022373855, 0.1409252633679265, 0.4575173087262126},
1633:      {0.28228993571972105, 0.14591072658175047, 0.46150987016675354},
1634:      {0.28188687220607433, 0.1508804582163323, 0.46540543189260436},
1635:      {0.2814119198278991, 0.15583292129399035, 0.4692010151483717},
1636:      {0.28086783263625503, 0.16077064757597467, 0.4728995412212838},
1637:      {0.28025489346354565, 0.16569211148611415, 0.47649814467710355},
1638:      {0.2795739667261627, 0.17059758179728512, 0.47999676019204823},
1639:      {0.2788258570423396, 0.1754892977307652, 0.48339725816565043},
1640:      {0.27801194338413965, 0.1803657682817326, 0.4866968956818299},
1641:      {0.27713381181214125, 0.18522747944269774, 0.4898983578428951},
1642:      {0.27619391034977797, 0.19007295088871226, 0.49300101709660177},
1643:      {0.2751907580349635, 0.19490465707076896, 0.49600544367769356},
1644:      {0.2741278682015853, 0.19972012989770083, 0.4989111236465034},
1645:      {0.2730059901819072, 0.20451860824383267, 0.501720814888259},
1646:      {0.2718278175684958, 0.2093023055918103, 0.5044342141093779},
1647:      {0.270594950010942, 0.21406778548878086, 0.5070519265124999},
1648:      {0.269307759397765, 0.21881747729569145, 0.5095772914792083},
1649:      {0.26796790172934454, 0.22354795925559695, 0.5120080244602125},
1650:      {0.26657969502130613, 0.22826164495517987, 0.5143493563955112},
1651:      {0.2651448458327315, 0.23295512949133207, 0.5165991088272563},
1652:      {0.2636630077044143, 0.23762961892442647, 0.518761871180001},
````
- **L1625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2829099523467409, 0.10539218382981966, 0.42690236327520725},`.
  - **L1625 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2829099523467409, 0.10539218382981966, 0.42690236327520725},`。
- **L1626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2830909293924702, 0.11055163703080462, 0.43155385240026206},`.
  - **L1626 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2830909293924702, 0.11055163703080462, 0.43155385240026206},`。
- **L1627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2831969393871863, 0.11567940038180258, 0.4361155100499537},`.
  - **L1627 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2831969393871863, 0.11567940038180258, 0.4361155100499537},`。
- **L1628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2832289314889045, 0.1207758539544326, 0.4405840195472734},`.
  - **L1628 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2832289314889045, 0.1207758539544326, 0.4405840195472734},`。
- **L1629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2831869362255922, 0.1258463172454443, 0.4449595399107695},`.
  - **L1629 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2831869362255922, 0.1258463172454443, 0.4449595399107695},`。
- **L1630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.28307192228564804, 0.13089406218358995, 0.4492411716459423},`.
  - **L1630 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.28307192228564804, 0.13089406218358995, 0.4492411716459423},`。
- **L1631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.28288394120635924, 0.1359185257084681, 0.45342671165618736},`.
  - **L1631 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.28288394120635924, 0.1359185257084681, 0.45342671165618736},`。
- **L1632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2826229022373855, 0.1409252633679265, 0.4575173087262126},`.
  - **L1632 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2826229022373855, 0.1409252633679265, 0.4575173087262126},`。
- **L1633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.28228993571972105, 0.14591072658175047, 0.46150987016675354},`.
  - **L1633 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.28228993571972105, 0.14591072658175047, 0.46150987016675354},`。
- **L1634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.28188687220607433, 0.1508804582163323, 0.46540543189260436},`.
  - **L1634 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.28188687220607433, 0.1508804582163323, 0.46540543189260436},`。
- **L1635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2814119198278991, 0.15583292129399035, 0.4692010151483717},`.
  - **L1635 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2814119198278991, 0.15583292129399035, 0.4692010151483717},`。
- **L1636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.28086783263625503, 0.16077064757597467, 0.4728995412212838},`.
  - **L1636 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.28086783263625503, 0.16077064757597467, 0.4728995412212838},`。
- **L1637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.28025489346354565, 0.16569211148611415, 0.47649814467710355},`.
  - **L1637 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.28025489346354565, 0.16569211148611415, 0.47649814467710355},`。
- **L1638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2795739667261627, 0.17059758179728512, 0.47999676019204823},`.
  - **L1638 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2795739667261627, 0.17059758179728512, 0.47999676019204823},`。
- **L1639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2788258570423396, 0.1754892977307652, 0.48339725816565043},`.
  - **L1639 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2788258570423396, 0.1754892977307652, 0.48339725816565043},`。
- **L1640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27801194338413965, 0.1803657682817326, 0.4866968956818299},`.
  - **L1640 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27801194338413965, 0.1803657682817326, 0.4866968956818299},`。
- **L1641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27713381181214125, 0.18522747944269774, 0.4898983578428951},`.
  - **L1641 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27713381181214125, 0.18522747944269774, 0.4898983578428951},`。
- **L1642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27619391034977797, 0.19007295088871226, 0.49300101709660177},`.
  - **L1642 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27619391034977797, 0.19007295088871226, 0.49300101709660177},`。
- **L1643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2751907580349635, 0.19490465707076896, 0.49600544367769356},`.
  - **L1643 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2751907580349635, 0.19490465707076896, 0.49600544367769356},`。
- **L1644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2741278682015853, 0.19972012989770083, 0.4989111236465034},`.
  - **L1644 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2741278682015853, 0.19972012989770083, 0.4989111236465034},`。
- **L1645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2730059901819072, 0.20451860824383267, 0.501720814888259},`.
  - **L1645 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2730059901819072, 0.20451860824383267, 0.501720814888259},`。
- **L1646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2718278175684958, 0.2093023055918103, 0.5044342141093779},`.
  - **L1646 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2718278175684958, 0.2093023055918103, 0.5044342141093779},`。
- **L1647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.270594950010942, 0.21406778548878086, 0.5070519265124999},`.
  - **L1647 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.270594950010942, 0.21406778548878086, 0.5070519265124999},`。
- **L1648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.269307759397765, 0.21881747729569145, 0.5095772914792083},`.
  - **L1648 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.269307759397765, 0.21881747729569145, 0.5095772914792083},`。
- **L1649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.26796790172934454, 0.22354795925559695, 0.5120080244602125},`.
  - **L1649 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.26796790172934454, 0.22354795925559695, 0.5120080244602125},`。
- **L1650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.26657969502130613, 0.22826164495517987, 0.5143493563955112},`.
  - **L1650 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.26657969502130613, 0.22826164495517987, 0.5143493563955112},`。
- **L1651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2651448458327315, 0.23295512949133207, 0.5165991088272563},`.
  - **L1651 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2651448458327315, 0.23295512949133207, 0.5165991088272563},`。
- **L1652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2636630077044143, 0.23762961892442647, 0.518761871180001},`.
  - **L1652 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2636630077044143, 0.23762961892442647, 0.518761871180001},`。

### Lines 1653-1680 / 第 1653-1680 行

````cpp
1653:      {0.2621377837546321, 0.2422852962306279, 0.5208371781925083},
1654:      {0.2605709530063988, 0.2469207884353261, 0.5228279594551629},
1655:      {0.2589647163025411, 0.25153645885437026, 0.5247362360674637},
1656:      {0.25732189181335147, 0.2561289544795441, 0.526563035336139},
1657:      {0.2556446444203948, 0.2607026175814763, 0.5283122832934375},
1658:      {0.25393482507335086, 0.26525311670734747, 0.529983099667603},
1659:      {0.2521940157752079, 0.2697816202890161, 0.5315789243388298},
1660:      {0.25042475493562577, 0.27428927505771267, 0.5331031508732055},
1661:      {0.24862894949587117, 0.27877378236392025, 0.5345559912288645},
1662:      {0.2468106816724398, 0.2832364292997213, 0.535941192655048},
1663:      {0.24497187865778625, 0.28767394082010833, 0.5372600476888723},
1664:      {0.24311260591427697, 0.2920915797036902, 0.5385162258554254},
1665:      {0.24123680455606578, 0.29648409536913767, 0.5397090946733651},
1666:      {0.23934601183850307, 0.30085361498147906, 0.5408439702969619},
1667:      {0.23744072960970264, 0.3052012458833173, 0.54192113151443},
1668:      {0.23552593740524197, 0.30952576959302097, 0.5429440191062838},
1669:      {0.23360265375847195, 0.3138273924379346, 0.5439141607822711},
1670:      {0.2316738606254108, 0.31810492063632195, 0.5448340596245334},
1671:      {0.22973857739190007, 0.3223605353157074, 0.545706183707169},
1672:      {0.22780178259701628, 0.3265930678980134, 0.5465320928009385},
1673:      {0.2258629950192772, 0.3308036038834664, 0.547314007363377},
1674:      {0.22392470580707102, 0.33499321123505194, 0.5480531185864527},
1675:      {0.22198891605799553, 0.33915975136273824, 0.5487520417750932},
1676:      {0.22005663012512805, 0.34330635098841344, 0.5494131386548431},
1677:      {0.2181298368119725, 0.34743089526543186, 0.550038069892983},
1678:      {0.21620955549289145, 0.3515344874402382, 0.5506271540883877},
1679:      {0.21429775792403594, 0.3556180358438001, 0.5511840927098516},
1680:      {0.21239496605173308, 0.3596815871313281, 0.5517100354247325},
````
- **L1653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2621377837546321, 0.2422852962306279, 0.5208371781925083},`.
  - **L1653 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2621377837546321, 0.2422852962306279, 0.5208371781925083},`。
- **L1654 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2605709530063988, 0.2469207884353261, 0.5228279594551629},`.
  - **L1654 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2605709530063988, 0.2469207884353261, 0.5228279594551629},`。
- **L1655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2589647163025411, 0.25153645885437026, 0.5247362360674637},`.
  - **L1655 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2589647163025411, 0.25153645885437026, 0.5247362360674637},`。
- **L1656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.25732189181335147, 0.2561289544795441, 0.526563035336139},`.
  - **L1656 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.25732189181335147, 0.2561289544795441, 0.526563035336139},`。
- **L1657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2556446444203948, 0.2607026175814763, 0.5283122832934375},`.
  - **L1657 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2556446444203948, 0.2607026175814763, 0.5283122832934375},`。
- **L1658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.25393482507335086, 0.26525311670734747, 0.529983099667603},`.
  - **L1658 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.25393482507335086, 0.26525311670734747, 0.529983099667603},`。
- **L1659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2521940157752079, 0.2697816202890161, 0.5315789243388298},`.
  - **L1659 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2521940157752079, 0.2697816202890161, 0.5315789243388298},`。
- **L1660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.25042475493562577, 0.27428927505771267, 0.5331031508732055},`.
  - **L1660 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.25042475493562577, 0.27428927505771267, 0.5331031508732055},`。
- **L1661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.24862894949587117, 0.27877378236392025, 0.5345559912288645},`.
  - **L1661 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.24862894949587117, 0.27877378236392025, 0.5345559912288645},`。
- **L1662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2468106816724398, 0.2832364292997213, 0.535941192655048},`.
  - **L1662 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2468106816724398, 0.2832364292997213, 0.535941192655048},`。
- **L1663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.24497187865778625, 0.28767394082010833, 0.5372600476888723},`.
  - **L1663 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.24497187865778625, 0.28767394082010833, 0.5372600476888723},`。
- **L1664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.24311260591427697, 0.2920915797036902, 0.5385162258554254},`.
  - **L1664 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.24311260591427697, 0.2920915797036902, 0.5385162258554254},`。
- **L1665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.24123680455606578, 0.29648409536913767, 0.5397090946733651},`.
  - **L1665 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.24123680455606578, 0.29648409536913767, 0.5397090946733651},`。
- **L1666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.23934601183850307, 0.30085361498147906, 0.5408439702969619},`.
  - **L1666 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.23934601183850307, 0.30085361498147906, 0.5408439702969619},`。
- **L1667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.23744072960970264, 0.3052012458833173, 0.54192113151443},`.
  - **L1667 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.23744072960970264, 0.3052012458833173, 0.54192113151443},`。
- **L1668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.23552593740524197, 0.30952576959302097, 0.5429440191062838},`.
  - **L1668 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.23552593740524197, 0.30952576959302097, 0.5429440191062838},`。
- **L1669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.23360265375847195, 0.3138273924379346, 0.5439141607822711},`.
  - **L1669 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.23360265375847195, 0.3138273924379346, 0.5439141607822711},`。
- **L1670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2316738606254108, 0.31810492063632195, 0.5448340596245334},`.
  - **L1670 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2316738606254108, 0.31810492063632195, 0.5448340596245334},`。
- **L1671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.22973857739190007, 0.3223605353157074, 0.545706183707169},`.
  - **L1671 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.22973857739190007, 0.3223605353157074, 0.545706183707169},`。
- **L1672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.22780178259701628, 0.3265930678980134, 0.5465320928009385},`.
  - **L1672 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.22780178259701628, 0.3265930678980134, 0.5465320928009385},`。
- **L1673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2258629950192772, 0.3308036038834664, 0.547314007363377},`.
  - **L1673 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2258629950192772, 0.3308036038834664, 0.547314007363377},`。
- **L1674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.22392470580707102, 0.33499321123505194, 0.5480531185864527},`.
  - **L1674 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.22392470580707102, 0.33499321123505194, 0.5480531185864527},`。
- **L1675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.22198891605799553, 0.33915975136273824, 0.5487520417750932},`.
  - **L1675 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.22198891605799553, 0.33915975136273824, 0.5487520417750932},`。
- **L1676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.22005663012512805, 0.34330635098841344, 0.5494131386548431},`.
  - **L1676 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.22005663012512805, 0.34330635098841344, 0.5494131386548431},`。
- **L1677 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2181298368119725, 0.34743089526543186, 0.550038069892983},`.
  - **L1677 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2181298368119725, 0.34743089526543186, 0.550038069892983},`。
- **L1678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.21620955549289145, 0.3515344874402382, 0.5506271540883877},`.
  - **L1678 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.21620955549289145, 0.3515344874402382, 0.5506271540883877},`。
- **L1679 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.21429775792403594, 0.3556180358438001, 0.5511840927098516},`.
  - **L1679 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.21429775792403594, 0.3556180358438001, 0.5511840927098516},`。
- **L1680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.21239496605173308, 0.3596815871313281, 0.5517100354247325},`.
  - **L1680 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.21239496605173308, 0.3596815871313281, 0.5517100354247325},`。

### Lines 1681-1708 / 第 1681-1708 行

````cpp
1681:      {0.21050268170035974, 0.36372617292766, 0.5522061103879761},
1682:      {0.20862288591525807, 0.36775072789850616, 0.5526750588505742},
1683:      {0.20675560837319001, 0.3717573067826811, 0.5531171238472018},
1684:      {0.20490280700807212, 0.3757448655138847, 0.5535330780419422},
1685:      {0.203062536964648, 0.3797154379631177, 0.5539251341179698},
1686:      {0.20123872979412472, 0.3836690001875795, 0.554294093380729},
1687:      {0.1994294675547129, 0.3876065665572156, 0.5546421417067892},
1688:      {0.19763565582691228, 0.39152713200801975, 0.5549691052245624},
1689:      {0.1958598486981541, 0.39543169955970453, 0.5552760714931869},
1690:      {0.19409958576782552, 0.3993222611485653, 0.5555651140662646},
1691:      {0.19235677239756216, 0.40319783169922974, 0.5558360841919235},
1692:      {0.19063051802725675, 0.4070603881536437, 0.5560891205440711},
1693:      {0.18892269821185473, 0.41090896152858886, 0.5563260943031525},
1694:      {0.18723045233808386, 0.4147455132535933, 0.5565471249886993},
1695:      {0.185555627191317, 0.4185690891632772, 0.5567531019269415},
1696:      {0.18389780714307888, 0.42238166634427127, 0.5569440809497231},
1697:      {0.18225556044836397, 0.4261832146898191, 0.5571201071543374},
1698:      {0.1806287343914301, 0.4299737941881375, 0.5572820892587651},
1699:      {0.17901849569766765, 0.43375533862493676, 0.5574301104997077},
1700:      {0.1774226636398102, 0.4375259202510252, 0.55756509546256},
1701:      {0.17584043270381128, 0.4412894613100537, 0.5576851118285253},
1702:      {0.17427359528182015, 0.4450430447202011, 0.557792099701029},
1703:      {0.17271876393003963, 0.4487896287788335, 0.557885089501858},
1704:      {0.1711755310919842, 0.45252916766885326, 0.5579651016381476},
1705:      {0.1696456943850158, 0.4562607533360476, 0.5580300944043132},
1706:      {0.1681254681463012, 0.4599872896301423, 0.5580821016030234},
1707:      {0.1666166265008438, 0.46370687659642523, 0.5581190973303202},
1708:      {0.16511640601237285, 0.46742241082026353, 0.5581410993753088},
````
- **L1681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.21050268170035974, 0.36372617292766, 0.5522061103879761},`.
  - **L1681 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.21050268170035974, 0.36372617292766, 0.5522061103879761},`。
- **L1682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.20862288591525807, 0.36775072789850616, 0.5526750588505742},`.
  - **L1682 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.20862288591525807, 0.36775072789850616, 0.5526750588505742},`。
- **L1683 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.20675560837319001, 0.3717573067826811, 0.5531171238472018},`.
  - **L1683 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.20675560837319001, 0.3717573067826811, 0.5531171238472018},`。
- **L1684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.20490280700807212, 0.3757448655138847, 0.5535330780419422},`.
  - **L1684 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.20490280700807212, 0.3757448655138847, 0.5535330780419422},`。
- **L1685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.203062536964648, 0.3797154379631177, 0.5539251341179698},`.
  - **L1685 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.203062536964648, 0.3797154379631177, 0.5539251341179698},`。
- **L1686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.20123872979412472, 0.3836690001875795, 0.554294093380729},`.
  - **L1686 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.20123872979412472, 0.3836690001875795, 0.554294093380729},`。
- **L1687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1994294675547129, 0.3876065665572156, 0.5546421417067892},`.
  - **L1687 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1994294675547129, 0.3876065665572156, 0.5546421417067892},`。
- **L1688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.19763565582691228, 0.39152713200801975, 0.5549691052245624},`.
  - **L1688 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.19763565582691228, 0.39152713200801975, 0.5549691052245624},`。
- **L1689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1958598486981541, 0.39543169955970453, 0.5552760714931869},`.
  - **L1689 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1958598486981541, 0.39543169955970453, 0.5552760714931869},`。
- **L1690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.19409958576782552, 0.3993222611485653, 0.5555651140662646},`.
  - **L1690 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.19409958576782552, 0.3993222611485653, 0.5555651140662646},`。
- **L1691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.19235677239756216, 0.40319783169922974, 0.5558360841919235},`.
  - **L1691 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.19235677239756216, 0.40319783169922974, 0.5558360841919235},`。
- **L1692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.19063051802725675, 0.4070603881536437, 0.5560891205440711},`.
  - **L1692 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.19063051802725675, 0.4070603881536437, 0.5560891205440711},`。
- **L1693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.18892269821185473, 0.41090896152858886, 0.5563260943031525},`.
  - **L1693 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.18892269821185473, 0.41090896152858886, 0.5563260943031525},`。
- **L1694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.18723045233808386, 0.4147455132535933, 0.5565471249886993},`.
  - **L1694 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.18723045233808386, 0.4147455132535933, 0.5565471249886993},`。
- **L1695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.185555627191317, 0.4185690891632772, 0.5567531019269415},`.
  - **L1695 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.185555627191317, 0.4185690891632772, 0.5567531019269415},`。
- **L1696 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.18389780714307888, 0.42238166634427127, 0.5569440809497231},`.
  - **L1696 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.18389780714307888, 0.42238166634427127, 0.5569440809497231},`。
- **L1697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.18225556044836397, 0.4261832146898191, 0.5571201071543374},`.
  - **L1697 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.18225556044836397, 0.4261832146898191, 0.5571201071543374},`。
- **L1698 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1806287343914301, 0.4299737941881375, 0.5572820892587651},`.
  - **L1698 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1806287343914301, 0.4299737941881375, 0.5572820892587651},`。
- **L1699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.17901849569766765, 0.43375533862493676, 0.5574301104997077},`.
  - **L1699 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.17901849569766765, 0.43375533862493676, 0.5574301104997077},`。
- **L1700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1774226636398102, 0.4375259202510252, 0.55756509546256},`.
  - **L1700 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1774226636398102, 0.4375259202510252, 0.55756509546256},`。
- **L1701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.17584043270381128, 0.4412894613100537, 0.5576851118285253},`.
  - **L1701 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.17584043270381128, 0.4412894613100537, 0.5576851118285253},`。
- **L1702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.17427359528182015, 0.4450430447202011, 0.557792099701029},`.
  - **L1702 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.17427359528182015, 0.4450430447202011, 0.557792099701029},`。
- **L1703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.17271876393003963, 0.4487896287788335, 0.557885089501858},`.
  - **L1703 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.17271876393003963, 0.4487896287788335, 0.557885089501858},`。
- **L1704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1711755310919842, 0.45252916766885326, 0.5579651016381476},`.
  - **L1704 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1711755310919842, 0.45252916766885326, 0.5579651016381476},`。
- **L1705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1696456943850158, 0.4562607533360476, 0.5580300944043132},`.
  - **L1705 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1696456943850158, 0.4562607533360476, 0.5580300944043132},`。
- **L1706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1681254681463012, 0.4599872896301423, 0.5580821016030234},`.
  - **L1706 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1681254681463012, 0.4599872896301423, 0.5580821016030234},`。
- **L1707 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1666166265008438, 0.46370687659642523, 0.5581190973303202},`.
  - **L1707 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1666166265008438, 0.46370687659642523, 0.5581190973303202},`。
- **L1708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.16511640601237285, 0.46742241082026353, 0.5581410993753088},`.
  - **L1708 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.16511640601237285, 0.46742241082026353, 0.5581410993753088},`。

### Lines 1709-1736 / 第 1709-1736 行

````cpp
1709:      {0.1636245598287687, 0.47113199888460483, 0.5581480982786068},
1710:      {0.1621417205544945, 0.4748365872638394, 0.5581400991890099},
1711:      {0.16066449675721917, 0.4785391201733818, 0.5581150965608792},
1712:      {0.15919365349458042, 0.48223570943504873, 0.5580731010879704},
1713:      {0.15772843426155927, 0.48593124098759627, 0.5580130923782937},
1714:      {0.1562695869579672, 0.4896228308128307, 0.5579361006261909},
1715:      {0.1548143719173188, 0.49331236133238937, 0.5578400853105705},
1716:      {0.1533635209362011, 0.4969989517366848, 0.5577240977120556},
1717:      {0.1519176771981809, 0.5006835421001327, 0.5575871128989297},
1718:      {0.15047545758720676, 0.504368072136516, 0.557430091404037},
1719:      {0.14903861048813108, 0.5080496628380102, 0.5572501110986087},
1720:      {0.1476063950561745, 0.5117321922659829, 0.5570490817856095},
1721:      {0.14617954401090907, 0.5154117832267491, 0.5568231063967104},
1722:      {0.14475833306803976, 0.519092312358787, 0.5565720684276931},
1723:      {0.14334247788386512, 0.522771903366683, 0.5562950984952888},
1724:      {0.141934629618203, 0.5264514940838813, 0.5559911321954153},
1725:      {0.14053541463564725, 0.5301310233517843, 0.5556590862704173},
1726:      {0.13914656229528236, 0.5338106140906136, 0.555298125858835},
1727:      {0.13776935391441048, 0.5374911432800976, 0.5549060695018647},
1728:      {0.13640749570156585, 0.5411717339632349, 0.5544831157469011},
1729:      {0.13506529616957136, 0.544852263189304, 0.554029048083696},
1730:      {0.13374243075510836, 0.5485338538220298, 0.5535411013024801},
1731:      {0.13244357030675505, 0.5522144442800521, 0.553018159322021},
1732:      {0.13117137121614486, 0.5558979736087772, 0.5524590815222541},
1733:      {0.12993250302214393, 0.5595805638443141, 0.5518641468320762},
1734:      {0.12872831884644725, 0.5632640933331003, 0.5512290558865701},
1735:      {0.12756743939839627, 0.5669476835219266, 0.5505561295873137},
1736:      {0.12645227472350737, 0.5706322130377723, 0.5498410244005342},
````
- **L1709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1636245598287687, 0.47113199888460483, 0.5581480982786068},`.
  - **L1709 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1636245598287687, 0.47113199888460483, 0.5581480982786068},`。
- **L1710 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1621417205544945, 0.4748365872638394, 0.5581400991890099},`.
  - **L1710 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1621417205544945, 0.4748365872638394, 0.5581400991890099},`。
- **L1711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.16066449675721917, 0.4785391201733818, 0.5581150965608792},`.
  - **L1711 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.16066449675721917, 0.4785391201733818, 0.5581150965608792},`。
- **L1712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15919365349458042, 0.48223570943504873, 0.5580731010879704},`.
  - **L1712 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15919365349458042, 0.48223570943504873, 0.5580731010879704},`。
- **L1713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15772843426155927, 0.48593124098759627, 0.5580130923782937},`.
  - **L1713 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15772843426155927, 0.48593124098759627, 0.5580130923782937},`。
- **L1714 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1562695869579672, 0.4896228308128307, 0.5579361006261909},`.
  - **L1714 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1562695869579672, 0.4896228308128307, 0.5579361006261909},`。
- **L1715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1548143719173188, 0.49331236133238937, 0.5578400853105705},`.
  - **L1715 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1548143719173188, 0.49331236133238937, 0.5578400853105705},`。
- **L1716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1533635209362011, 0.4969989517366848, 0.5577240977120556},`.
  - **L1716 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1533635209362011, 0.4969989517366848, 0.5577240977120556},`。
- **L1717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1519176771981809, 0.5006835421001327, 0.5575871128989297},`.
  - **L1717 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1519176771981809, 0.5006835421001327, 0.5575871128989297},`。
- **L1718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15047545758720676, 0.504368072136516, 0.557430091404037},`.
  - **L1718 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15047545758720676, 0.504368072136516, 0.557430091404037},`。
- **L1719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14903861048813108, 0.5080496628380102, 0.5572501110986087},`.
  - **L1719 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14903861048813108, 0.5080496628380102, 0.5572501110986087},`。
- **L1720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1476063950561745, 0.5117321922659829, 0.5570490817856095},`.
  - **L1720 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1476063950561745, 0.5117321922659829, 0.5570490817856095},`。
- **L1721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14617954401090907, 0.5154117832267491, 0.5568231063967104},`.
  - **L1721 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14617954401090907, 0.5154117832267491, 0.5568231063967104},`。
- **L1722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14475833306803976, 0.519092312358787, 0.5565720684276931},`.
  - **L1722 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14475833306803976, 0.519092312358787, 0.5565720684276931},`。
- **L1723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14334247788386512, 0.522771903366683, 0.5562950984952888},`.
  - **L1723 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14334247788386512, 0.522771903366683, 0.5562950984952888},`。
- **L1724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.141934629618203, 0.5264514940838813, 0.5559911321954153},`.
  - **L1724 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.141934629618203, 0.5264514940838813, 0.5559911321954153},`。
- **L1725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14053541463564725, 0.5301310233517843, 0.5556590862704173},`.
  - **L1725 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14053541463564725, 0.5301310233517843, 0.5556590862704173},`。
- **L1726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13914656229528236, 0.5338106140906136, 0.555298125858835},`.
  - **L1726 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13914656229528236, 0.5338106140906136, 0.555298125858835},`。
- **L1727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13776935391441048, 0.5374911432800976, 0.5549060695018647},`.
  - **L1727 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13776935391441048, 0.5374911432800976, 0.5549060695018647},`。
- **L1728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13640749570156585, 0.5411717339632349, 0.5544831157469011},`.
  - **L1728 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13640749570156585, 0.5411717339632349, 0.5544831157469011},`。
- **L1729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13506529616957136, 0.544852263189304, 0.554029048083696},`.
  - **L1729 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13506529616957136, 0.544852263189304, 0.554029048083696},`。
- **L1730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13374243075510836, 0.5485338538220298, 0.5535411013024801},`.
  - **L1730 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13374243075510836, 0.5485338538220298, 0.5535411013024801},`。
- **L1731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13244357030675505, 0.5522144442800521, 0.553018159322021},`.
  - **L1731 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13244357030675505, 0.5522144442800521, 0.553018159322021},`。
- **L1732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13117137121614486, 0.5558979736087772, 0.5524590815222541},`.
  - **L1732 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13117137121614486, 0.5558979736087772, 0.5524590815222541},`。
- **L1733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12993250302214393, 0.5595805638443141, 0.5518641468320762},`.
  - **L1733 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12993250302214393, 0.5595805638443141, 0.5518641468320762},`。
- **L1734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12872831884644725, 0.5632640933331003, 0.5512290558865701},`.
  - **L1734 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12872831884644725, 0.5632640933331003, 0.5512290558865701},`。
- **L1735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12756743939839627, 0.5669476835219266, 0.5505561295873137},`.
  - **L1735 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12756743939839627, 0.5669476835219266, 0.5505561295873137},`。
- **L1736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12645227472350737, 0.5706322130377723, 0.5498410244005342},`.
  - **L1736 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12645227472350737, 0.5706322130377723, 0.5498410244005342},`。

### Lines 1737-1764 / 第 1737-1764 行

````cpp
1737:      {0.12539338154311577, 0.5743168031831126, 0.5490861069146677},
1738:      {0.12439424184336756, 0.5780013325949876, 0.5482869865182912},
1739:      {0.12346233479357416, 0.5816859227477941, 0.5474450777703374},
1740:      {0.122605426766972, 0.5853695126930177, 0.5465571752923358},
1741:      {0.12183030409088419, 0.5890540421334565, 0.5456230413386914},
1742:      {0.1211473750007544, 0.592737632144662, 0.5446411489781364},
1743:      {0.12056429075653713, 0.5964211612480832, 0.5436109978665574},
1744:      {0.12009133699819924, 0.6001027515502533, 0.5425301161500574},
1745:      {0.11973729926029653, 0.6037842799987541, 0.541399947187814},
1746:      {0.11951131929191963, 0.6074628706490315, 0.5402180758379009},
1747:      {0.11942233180714437, 0.6111394611583431, 0.5389822116295199},
1748:      {0.11948233048502255, 0.6148159892453784, 0.537692026866569},
1749:      {0.11969830860262576, 0.6184885802617517, 0.5363471743826108},
1750:      {0.12008036996652452, 0.6221601072840744, 0.5349459699054088},
1751:      {0.12063730989641286, 0.6258266990315783, 0.5334881295553462},
1752:      {0.1213794417235072, 0.6294912247800063, 0.5319729045347256},
1753:      {0.12231134108889466, 0.6331518172152074, 0.5303980764654408},
1754:      {0.1234432274542607, 0.6368074097895444, 0.5287632561537092},
1755:      {0.12477941321977204, 0.6404599344231255, 0.5270680133779303},
1756:      {0.12632525332983902, 0.6441055281645606, 0.5253112061650272},
1757:      {0.1280865204366354, 0.6477480507159987, 0.5234909414076758},
1758:      {0.1300663115014182, 0.6513826457355586, 0.5216081476931036},
1759:      {0.13226766345372454, 0.6550131658699597, 0.5196608602452112},
1760:      {0.1346914034616326, 0.6586347623899736, 0.5176490803131216},
1761:      {0.13733813187439356, 0.6622503590720668, 0.5155713087695435},
1762:      {0.1402095413205375, 0.6658578775169739, 0.5134270017152724},
1763:      {0.14330221816701194, 0.6694574760216492, 0.511215244517751},
1764:      {0.14661571262857995, 0.6730489911445044, 0.5089359136232384},
````
- **L1737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12539338154311577, 0.5743168031831126, 0.5490861069146677},`.
  - **L1737 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12539338154311577, 0.5743168031831126, 0.5490861069146677},`。
- **L1738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12439424184336756, 0.5780013325949876, 0.5482869865182912},`.
  - **L1738 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12439424184336756, 0.5780013325949876, 0.5482869865182912},`。
- **L1739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12346233479357416, 0.5816859227477941, 0.5474450777703374},`.
  - **L1739 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12346233479357416, 0.5816859227477941, 0.5474450777703374},`。
- **L1740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.122605426766972, 0.5853695126930177, 0.5465571752923358},`.
  - **L1740 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.122605426766972, 0.5853695126930177, 0.5465571752923358},`。
- **L1741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12183030409088419, 0.5890540421334565, 0.5456230413386914},`.
  - **L1741 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12183030409088419, 0.5890540421334565, 0.5456230413386914},`。
- **L1742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1211473750007544, 0.592737632144662, 0.5446411489781364},`.
  - **L1742 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1211473750007544, 0.592737632144662, 0.5446411489781364},`。
- **L1743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12056429075653713, 0.5964211612480832, 0.5436109978665574},`.
  - **L1743 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12056429075653713, 0.5964211612480832, 0.5436109978665574},`。
- **L1744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12009133699819924, 0.6001027515502533, 0.5425301161500574},`.
  - **L1744 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12009133699819924, 0.6001027515502533, 0.5425301161500574},`。
- **L1745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11973729926029653, 0.6037842799987541, 0.541399947187814},`.
  - **L1745 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11973729926029653, 0.6037842799987541, 0.541399947187814},`。
- **L1746 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11951131929191963, 0.6074628706490315, 0.5402180758379009},`.
  - **L1746 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11951131929191963, 0.6074628706490315, 0.5402180758379009},`。
- **L1747 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11942233180714437, 0.6111394611583431, 0.5389822116295199},`.
  - **L1747 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11942233180714437, 0.6111394611583431, 0.5389822116295199},`。
- **L1748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11948233048502255, 0.6148159892453784, 0.537692026866569},`.
  - **L1748 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11948233048502255, 0.6148159892453784, 0.537692026866569},`。
- **L1749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.11969830860262576, 0.6184885802617517, 0.5363471743826108},`.
  - **L1749 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.11969830860262576, 0.6184885802617517, 0.5363471743826108},`。
- **L1750 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12008036996652452, 0.6221601072840744, 0.5349459699054088},`.
  - **L1750 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12008036996652452, 0.6221601072840744, 0.5349459699054088},`。
- **L1751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12063730989641286, 0.6258266990315783, 0.5334881295553462},`.
  - **L1751 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12063730989641286, 0.6258266990315783, 0.5334881295553462},`。
- **L1752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1213794417235072, 0.6294912247800063, 0.5319729045347256},`.
  - **L1752 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1213794417235072, 0.6294912247800063, 0.5319729045347256},`。
- **L1753 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12231134108889466, 0.6331518172152074, 0.5303980764654408},`.
  - **L1753 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12231134108889466, 0.6331518172152074, 0.5303980764654408},`。
- **L1754 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1234432274542607, 0.6368074097895444, 0.5287632561537092},`.
  - **L1754 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1234432274542607, 0.6368074097895444, 0.5287632561537092},`。
- **L1755 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12477941321977204, 0.6404599344231255, 0.5270680133779303},`.
  - **L1755 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12477941321977204, 0.6404599344231255, 0.5270680133779303},`。
- **L1756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.12632525332983902, 0.6441055281645606, 0.5253112061650272},`.
  - **L1756 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.12632525332983902, 0.6441055281645606, 0.5253112061650272},`。
- **L1757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1280865204366354, 0.6477480507159987, 0.5234909414076758},`.
  - **L1757 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1280865204366354, 0.6477480507159987, 0.5234909414076758},`。
- **L1758 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1300663115014182, 0.6513826457355586, 0.5216081476931036},`.
  - **L1758 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1300663115014182, 0.6513826457355586, 0.5216081476931036},`。
- **L1759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13226766345372454, 0.6550131658699597, 0.5196608602452112},`.
  - **L1759 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13226766345372454, 0.6550131658699597, 0.5196608602452112},`。
- **L1760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1346914034616326, 0.6586347623899736, 0.5176490803131216},`.
  - **L1760 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1346914034616326, 0.6586347623899736, 0.5176490803131216},`。
- **L1761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.13733813187439356, 0.6622503590720668, 0.5155713087695435},`.
  - **L1761 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.13733813187439356, 0.6622503590720668, 0.5155713087695435},`。
- **L1762 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1402095413205375, 0.6658578775169739, 0.5134270017152724},`.
  - **L1762 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1402095413205375, 0.6658578775169739, 0.5134270017152724},`。
- **L1763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14330221816701194, 0.6694574760216492, 0.511215244517751},`.
  - **L1763 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14330221816701194, 0.6694574760216492, 0.511215244517751},`。
- **L1764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.14661571262857995, 0.6730489911445044, 0.5089359136232384},`.
  - **L1764 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.14661571262857995, 0.6730489911445044, 0.5089359136232384},`。

### Lines 1765-1792 / 第 1765-1792 行

````cpp
1765:      {0.15014733747892414, 0.6766295918037593, 0.5065891708280766},
1766:      {0.1538939151679172, 0.6802021032660456, 0.5041718156197705},
1767:      {0.15785048833835377, 0.6837637060778221, 0.5016860876044946},
1768:      {0.162015053737826, 0.6873143096440719, 0.49912936913057365},
1769:      {0.1663826794355118, 0.6908548182506177, 0.49650199274870527},
1770:      {0.17094719748923695, 0.6943824243247052, 0.4938032891165472},
1771:      {0.1757068993715085, 0.6978989283401605, 0.4910328874274477},
1772:      {0.18065236935056878, 0.7014005373289202, 0.4881891991816308},
1773:      {0.18578314370608157, 0.704890036301888, 0.48527277191686385},
1774:      {0.19108956697744361, 0.7083646483099804, 0.48228409897291785},
1775:      {0.19656998786638302, 0.7118252614850998, 0.4792214366155811},
1776:      {0.20221879696964223, 0.7152707565867668, 0.47608398702197946},
1777:      {0.2080291781284243, 0.7186993731402823, 0.47287333975819085},
1778:      {0.21400005157818647, 0.7221128620832928, 0.4695878639271211},
1779:      {0.22012339076765558, 0.7255074824139881, 0.46622623255251056},
1780:      {0.22639732501345455, 0.7288869647642494, 0.46278873032462237},
1781:      {0.2328146240063111, 0.7322455890831692, 0.45927711461163473},
1782:      {0.2393729244291451, 0.7355862151598742, 0.4556885106741981},
1783:      {0.2460698827335157, 0.7389086924367883, 0.452023984943217},
1784:      {0.25289815079190964, 0.742209322598094, 0.44828439600718756},
1785:      {0.2598571632843528, 0.745490792270686, 0.4444668436999196},
1786:      {0.2669403953805219, 0.7487494270612788, 0.44057327076578856},
1787:      {0.27414945925613216, 0.7519868887012355, 0.43660069156818965},
1788:      {0.28147665665716626, 0.755201528324013, 0.43255213457796365},
1789:      {0.2889217691195198, 0.7583929812556258, 0.42842552870164097},
1790:      {0.2964789382360529, 0.7615596257155389, 0.4242229866985808},
1791:      {0.3041471124530943, 0.7647022727239619, 0.41994345744752104},
1792:      {0.31192524014449496, 0.7678207187194435, 0.41558582710182396},
````
- **L1765 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15014733747892414, 0.6766295918037593, 0.5065891708280766},`.
  - **L1765 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15014733747892414, 0.6766295918037593, 0.5065891708280766},`。
- **L1766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1538939151679172, 0.6802021032660456, 0.5041718156197705},`.
  - **L1766 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1538939151679172, 0.6802021032660456, 0.5041718156197705},`。
- **L1767 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.15785048833835377, 0.6837637060778221, 0.5016860876044946},`.
  - **L1767 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.15785048833835377, 0.6837637060778221, 0.5016860876044946},`。
- **L1768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.162015053737826, 0.6873143096440719, 0.49912936913057365},`.
  - **L1768 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.162015053737826, 0.6873143096440719, 0.49912936913057365},`。
- **L1769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1663826794355118, 0.6908548182506177, 0.49650199274870527},`.
  - **L1769 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1663826794355118, 0.6908548182506177, 0.49650199274870527},`。
- **L1770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.17094719748923695, 0.6943824243247052, 0.4938032891165472},`.
  - **L1770 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.17094719748923695, 0.6943824243247052, 0.4938032891165472},`。
- **L1771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.1757068993715085, 0.6978989283401605, 0.4910328874274477},`.
  - **L1771 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.1757068993715085, 0.6978989283401605, 0.4910328874274477},`。
- **L1772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.18065236935056878, 0.7014005373289202, 0.4881891991816308},`.
  - **L1772 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.18065236935056878, 0.7014005373289202, 0.4881891991816308},`。
- **L1773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.18578314370608157, 0.704890036301888, 0.48527277191686385},`.
  - **L1773 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.18578314370608157, 0.704890036301888, 0.48527277191686385},`。
- **L1774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.19108956697744361, 0.7083646483099804, 0.48228409897291785},`.
  - **L1774 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.19108956697744361, 0.7083646483099804, 0.48228409897291785},`。
- **L1775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.19656998786638302, 0.7118252614850998, 0.4792214366155811},`.
  - **L1775 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.19656998786638302, 0.7118252614850998, 0.4792214366155811},`。
- **L1776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.20221879696964223, 0.7152707565867668, 0.47608398702197946},`.
  - **L1776 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.20221879696964223, 0.7152707565867668, 0.47608398702197946},`。
- **L1777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2080291781284243, 0.7186993731402823, 0.47287333975819085},`.
  - **L1777 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2080291781284243, 0.7186993731402823, 0.47287333975819085},`。
- **L1778 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.21400005157818647, 0.7221128620832928, 0.4695878639271211},`.
  - **L1778 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.21400005157818647, 0.7221128620832928, 0.4695878639271211},`。
- **L1779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.22012339076765558, 0.7255074824139881, 0.46622623255251056},`.
  - **L1779 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.22012339076765558, 0.7255074824139881, 0.46622623255251056},`。
- **L1780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.22639732501345455, 0.7288869647642494, 0.46278873032462237},`.
  - **L1780 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.22639732501345455, 0.7288869647642494, 0.46278873032462237},`。
- **L1781 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2328146240063111, 0.7322455890831692, 0.45927711461163473},`.
  - **L1781 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2328146240063111, 0.7322455890831692, 0.45927711461163473},`。
- **L1782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2393729244291451, 0.7355862151598742, 0.4556885106741981},`.
  - **L1782 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2393729244291451, 0.7355862151598742, 0.4556885106741981},`。
- **L1783 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2460698827335157, 0.7389086924367883, 0.452023984943217},`.
  - **L1783 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2460698827335157, 0.7389086924367883, 0.452023984943217},`。
- **L1784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.25289815079190964, 0.742209322598094, 0.44828439600718756},`.
  - **L1784 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.25289815079190964, 0.742209322598094, 0.44828439600718756},`。
- **L1785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2598571632843528, 0.745490792270686, 0.4444668436999196},`.
  - **L1785 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2598571632843528, 0.745490792270686, 0.4444668436999196},`。
- **L1786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2669403953805219, 0.7487494270612788, 0.44057327076578856},`.
  - **L1786 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2669403953805219, 0.7487494270612788, 0.44057327076578856},`。
- **L1787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.27414945925613216, 0.7519868887012355, 0.43660069156818965},`.
  - **L1787 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.27414945925613216, 0.7519868887012355, 0.43660069156818965},`。
- **L1788 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.28147665665716626, 0.755201528324013, 0.43255213457796365},`.
  - **L1788 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.28147665665716626, 0.755201528324013, 0.43255213457796365},`。
- **L1789 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2889217691195198, 0.7583929812556258, 0.42842552870164097},`.
  - **L1789 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2889217691195198, 0.7583929812556258, 0.42842552870164097},`。
- **L1790 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.2964789382360529, 0.7615596257155389, 0.4242229866985808},`.
  - **L1790 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.2964789382360529, 0.7615596257155389, 0.4242229866985808},`。
- **L1791 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3041471124530943, 0.7647022727239619, 0.41994345744752104},`.
  - **L1791 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3041471124530943, 0.7647022727239619, 0.41994345744752104},`。
- **L1792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.31192524014449496, 0.7678207187194435, 0.41558582710182396},`.
  - **L1792 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.31192524014449496, 0.7678207187194435, 0.41558582710182396},`。

### Lines 1793-1820 / 第 1793-1820 行

````cpp
1793:      {0.31980838304287407, 0.7709123713051562, 0.41115231354707876},
1794:      {0.32779655496333804, 0.7739788075712202, 0.40663965647349865},
1795:      {0.33588466791449195, 0.7770164660401522, 0.4020491587042435},
1796:      {0.34407488193371133, 0.7800278920656099, 0.3973804752308849},
1797:      {0.3523599689997323, 0.7830095562872507, 0.3926359923281875},
1798:      {0.36074006346245197, 0.7859622236060678, 0.3878145232212208},
1799:      {0.36921428939455786, 0.7888866412701451, 0.38291381435701694},
1800:      {0.37777835651210034, 0.7917793150973379, 0.37793936040098913},
1801:      {0.3864336209519174, 0.7946427215203885, 0.3728856256306759},
1802:      {0.3951736619954228, 0.797473402038798, 0.3677571865876491},
1803:      {0.4040019632351159, 0.8002737966998706, 0.3625514261814766},
1804:      {0.4129129804482464, 0.8030394840252982, 0.3572690015831872},
1805:      {0.4219070059826357, 0.805772174900056, 0.3519105924567475},
1806:      {0.4309833172715677, 0.8084715598896897, 0.3464758054878499},
1807:      {0.4401363189373519, 0.8111362580724669, 0.34096741055434293},
1808:      {0.44936866381374757, 0.8137666304213753, 0.3353835991203212},
1809:      {0.458673642833963, 0.8163613360822743, 0.32972721765157326},
1810:      {0.46805401938353625, 0.8189196953983692, 0.3239973827222524},
1811:      {0.47750397699915853, 0.8214424087022711, 0.3181950138984179},
1812:      {0.48702494316955264, 0.8239271261653897, 0.31232166294182256},
1813:      {0.49661532811633474, 0.8263744743966721, 0.306376800619062},
1814:      {0.5062702739767431, 0.8287841998369972, 0.3003624619571904},
1815:      {0.5159926874598648, 0.8311565342261135, 0.2942785778807102},
1816:      {0.5257756141542576, 0.8334892678013092, 0.28812725033290043},
1817:      {0.5356220541188685, 0.8357835879688815, 0.28190734645112897},
1818:      {0.5455239631257167, 0.838037329833802, 0.2756260283833135},
1819:      {0.5554828787549823, 0.8402520764412206, 0.2692817313403921},
1820:      {0.5654983251339641, 0.8424283847332169, 0.2628767994071899},
````
- **L1793 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.31980838304287407, 0.7709123713051562, 0.41115231354707876},`.
  - **L1793 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.31980838304287407, 0.7709123713051562, 0.41115231354707876},`。
- **L1794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.32779655496333804, 0.7739788075712202, 0.40663965647349865},`.
  - **L1794 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.32779655496333804, 0.7739788075712202, 0.40663965647349865},`。
- **L1795 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.33588466791449195, 0.7770164660401522, 0.4020491587042435},`.
  - **L1795 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.33588466791449195, 0.7770164660401522, 0.4020491587042435},`。
- **L1796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.34407488193371133, 0.7800278920656099, 0.3973804752308849},`.
  - **L1796 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.34407488193371133, 0.7800278920656099, 0.3973804752308849},`。
- **L1797 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3523599689997323, 0.7830095562872507, 0.3926359923281875},`.
  - **L1797 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3523599689997323, 0.7830095562872507, 0.3926359923281875},`。
- **L1798 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.36074006346245197, 0.7859622236060678, 0.3878145232212208},`.
  - **L1798 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.36074006346245197, 0.7859622236060678, 0.3878145232212208},`。
- **L1799 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.36921428939455786, 0.7888866412701451, 0.38291381435701694},`.
  - **L1799 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.36921428939455786, 0.7888866412701451, 0.38291381435701694},`。
- **L1800 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.37777835651210034, 0.7917793150973379, 0.37793936040098913},`.
  - **L1800 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.37777835651210034, 0.7917793150973379, 0.37793936040098913},`。
- **L1801 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3864336209519174, 0.7946427215203885, 0.3728856256306759},`.
  - **L1801 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3864336209519174, 0.7946427215203885, 0.3728856256306759},`。
- **L1802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.3951736619954228, 0.797473402038798, 0.3677571865876491},`.
  - **L1802 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.3951736619954228, 0.797473402038798, 0.3677571865876491},`。
- **L1803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4040019632351159, 0.8002737966998706, 0.3625514261814766},`.
  - **L1803 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4040019632351159, 0.8002737966998706, 0.3625514261814766},`。
- **L1804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4129129804482464, 0.8030394840252982, 0.3572690015831872},`.
  - **L1804 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4129129804482464, 0.8030394840252982, 0.3572690015831872},`。
- **L1805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4219070059826357, 0.805772174900056, 0.3519105924567475},`.
  - **L1805 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4219070059826357, 0.805772174900056, 0.3519105924567475},`。
- **L1806 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4309833172715677, 0.8084715598896897, 0.3464758054878499},`.
  - **L1806 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4309833172715677, 0.8084715598896897, 0.3464758054878499},`。
- **L1807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.4401363189373519, 0.8111362580724669, 0.34096741055434293},`.
  - **L1807 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.4401363189373519, 0.8111362580724669, 0.34096741055434293},`。
- **L1808 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.44936866381374757, 0.8137666304213753, 0.3353835991203212},`.
  - **L1808 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.44936866381374757, 0.8137666304213753, 0.3353835991203212},`。
- **L1809 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.458673642833963, 0.8163613360822743, 0.32972721765157326},`.
  - **L1809 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.458673642833963, 0.8163613360822743, 0.32972721765157326},`。
- **L1810 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.46805401938353625, 0.8189196953983692, 0.3239973827222524},`.
  - **L1810 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.46805401938353625, 0.8189196953983692, 0.3239973827222524},`。
- **L1811 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.47750397699915853, 0.8214424087022711, 0.3181950138984179},`.
  - **L1811 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.47750397699915853, 0.8214424087022711, 0.3181950138984179},`。
- **L1812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.48702494316955264, 0.8239271261653897, 0.31232166294182256},`.
  - **L1812 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.48702494316955264, 0.8239271261653897, 0.31232166294182256},`。
- **L1813 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.49661532811633474, 0.8263744743966721, 0.306376800619062},`.
  - **L1813 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.49661532811633474, 0.8263744743966721, 0.306376800619062},`。
- **L1814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5062702739767431, 0.8287841998369972, 0.3003624619571904},`.
  - **L1814 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5062702739767431, 0.8287841998369972, 0.3003624619571904},`。
- **L1815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5159926874598648, 0.8311565342261135, 0.2942785778807102},`.
  - **L1815 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5159926874598648, 0.8311565342261135, 0.2942785778807102},`。
- **L1816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5257756141542576, 0.8334892678013092, 0.28812725033290043},`.
  - **L1816 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5257756141542576, 0.8334892678013092, 0.28812725033290043},`。
- **L1817 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5356220541188685, 0.8357835879688815, 0.28190734645112897},`.
  - **L1817 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5356220541188685, 0.8357835879688815, 0.28190734645112897},`。
- **L1818 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5455239631257167, 0.838037329833802, 0.2756260283833135},`.
  - **L1818 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5455239631257167, 0.838037329833802, 0.2756260283833135},`。
- **L1819 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5554828787549823, 0.8402520764412206, 0.2692817313403921},`.
  - **L1819 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5554828787549823, 0.8402520764412206, 0.2692817313403921},`。
- **L1820 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5654983251339641, 0.8424283847332169, 0.2628767994071899},`.
  - **L1820 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5654983251339641, 0.8424283847332169, 0.2628767994071899},`。

### Lines 1821-1848 / 第 1821-1848 行

````cpp
1821:      {0.5755622254006186, 0.8445641394621258, 0.25641551104548627},
1822:      {0.5856786942007215, 0.8466594333420596, 0.24989656346356887},
1823:      {0.5958385796727564, 0.8487151964088427, 0.2433292811877542},
1824:      {0.6060460684908495, 0.8507314758177834, 0.2367113216835447},
1825:      {0.616292940650758, 0.8527072471488548, 0.23005204277336477},
1826:      {0.6265778183939344, 0.8546430238400886, 0.2233537871857682},
1827:      {0.6369023105271391, 0.8565402909227283, 0.2166198017460239},
1828:      {0.6472561782044223, 0.8583980753432965, 0.2098615478515251},
1829:      {0.6576426853779066, 0.8602173283504794, 0.20308155987152815},
1830:      {0.6680535433907611, 0.8619971207604976, 0.19629330158139313},
1831:      {0.6784900623681449, 0.8637403601361504, 0.18950231977185272},
1832:      {0.6889439127328617, 0.86544616002535, 0.18272505128011118},
1833:      {0.6994137684819729, 0.8671149653844072, 0.17597180349415806},
1834:      {0.7098982863987701, 0.868749193048019, 0.1692568079838667},
1835:      {0.7203901380970731, 0.8703480050687866, 0.16260354455700873},
1836:      {0.7308896616716263, 0.8719142204262896, 0.1560285800997179},
1837:      {0.7413875101130836, 0.873447039008859, 0.1495612865358065},
1838:      {0.7518850354669301, 0.8749492431662675, 0.14322737498076443},
1839:      {0.7623728828501631, 0.8764220675983936, 0.13706403823168214},
1840:      {0.7728534061279669, 0.8778662616531304, 0.13110820913488913},
1841:      {0.7833152551224711, 0.8792830913055214, 0.1254048217206159},
1842:      {0.7937591112086629, 0.880675925144139, 0.12000541978485552},
1843:      {0.8041826246822775, 0.8820441108502998, 0.11496466377053949},
1844:      {0.8145754845039437, 0.8833909489919768, 0.11034716773962891},
1845:      {0.8249409891695173, 0.8847181273467387, 0.10621658195896774},
1846:      {0.8352698545299048, 0.8860269688856748, 0.1026459672214125},
1847:      {0.8455623467619914, 0.8873201416511192, 0.09970160643955417},
1848:      {0.8558102197403156, 0.8885989857373076, 0.09745185636949635},
````
- **L1821 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5755622254006186, 0.8445641394621258, 0.25641551104548627},`.
  - **L1821 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5755622254006186, 0.8445641394621258, 0.25641551104548627},`。
- **L1822 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5856786942007215, 0.8466594333420596, 0.24989656346356887},`.
  - **L1822 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5856786942007215, 0.8466594333420596, 0.24989656346356887},`。
- **L1823 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.5958385796727564, 0.8487151964088427, 0.2433292811877542},`.
  - **L1823 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.5958385796727564, 0.8487151964088427, 0.2433292811877542},`。
- **L1824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6060460684908495, 0.8507314758177834, 0.2367113216835447},`.
  - **L1824 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6060460684908495, 0.8507314758177834, 0.2367113216835447},`。
- **L1825 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.616292940650758, 0.8527072471488548, 0.23005204277336477},`.
  - **L1825 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.616292940650758, 0.8527072471488548, 0.23005204277336477},`。
- **L1826 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6265778183939344, 0.8546430238400886, 0.2233537871857682},`.
  - **L1826 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6265778183939344, 0.8546430238400886, 0.2233537871857682},`。
- **L1827 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6369023105271391, 0.8565402909227283, 0.2166198017460239},`.
  - **L1827 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6369023105271391, 0.8565402909227283, 0.2166198017460239},`。
- **L1828 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6472561782044223, 0.8583980753432965, 0.2098615478515251},`.
  - **L1828 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6472561782044223, 0.8583980753432965, 0.2098615478515251},`。
- **L1829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6576426853779066, 0.8602173283504794, 0.20308155987152815},`.
  - **L1829 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6576426853779066, 0.8602173283504794, 0.20308155987152815},`。
- **L1830 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6680535433907611, 0.8619971207604976, 0.19629330158139313},`.
  - **L1830 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6680535433907611, 0.8619971207604976, 0.19629330158139313},`。
- **L1831 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6784900623681449, 0.8637403601361504, 0.18950231977185272},`.
  - **L1831 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6784900623681449, 0.8637403601361504, 0.18950231977185272},`。
- **L1832 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6889439127328617, 0.86544616002535, 0.18272505128011118},`.
  - **L1832 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6889439127328617, 0.86544616002535, 0.18272505128011118},`。
- **L1833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.6994137684819729, 0.8671149653844072, 0.17597180349415806},`.
  - **L1833 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.6994137684819729, 0.8671149653844072, 0.17597180349415806},`。
- **L1834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7098982863987701, 0.868749193048019, 0.1692568079838667},`.
  - **L1834 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7098982863987701, 0.868749193048019, 0.1692568079838667},`。
- **L1835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7203901380970731, 0.8703480050687866, 0.16260354455700873},`.
  - **L1835 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7203901380970731, 0.8703480050687866, 0.16260354455700873},`。
- **L1836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7308896616716263, 0.8719142204262896, 0.1560285800997179},`.
  - **L1836 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7308896616716263, 0.8719142204262896, 0.1560285800997179},`。
- **L1837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7413875101130836, 0.873447039008859, 0.1495612865358065},`.
  - **L1837 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7413875101130836, 0.873447039008859, 0.1495612865358065},`。
- **L1838 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7518850354669301, 0.8749492431662675, 0.14322737498076443},`.
  - **L1838 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7518850354669301, 0.8749492431662675, 0.14322737498076443},`。
- **L1839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7623728828501631, 0.8764220675983936, 0.13706403823168214},`.
  - **L1839 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7623728828501631, 0.8764220675983936, 0.13706403823168214},`。
- **L1840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7728534061279669, 0.8778662616531304, 0.13110820913488913},`.
  - **L1840 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7728534061279669, 0.8778662616531304, 0.13110820913488913},`。
- **L1841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7833152551224711, 0.8792830913055214, 0.1254048217206159},`.
  - **L1841 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7833152551224711, 0.8792830913055214, 0.1254048217206159},`。
- **L1842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.7937591112086629, 0.880675925144139, 0.12000541978485552},`.
  - **L1842 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.7937591112086629, 0.880675925144139, 0.12000541978485552},`。
- **L1843 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8041826246822775, 0.8820441108502998, 0.11496466377053949},`.
  - **L1843 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8041826246822775, 0.8820441108502998, 0.11496466377053949},`。
- **L1844 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8145754845039437, 0.8833909489919768, 0.11034716773962891},`.
  - **L1844 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8145754845039437, 0.8833909489919768, 0.11034716773962891},`。
- **L1845 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8249409891695173, 0.8847181273467387, 0.10621658195896774},`.
  - **L1845 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8249409891695173, 0.8847181273467387, 0.10621658195896774},`。
- **L1846 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8352698545299048, 0.8860269688856748, 0.1026459672214125},`.
  - **L1846 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8352698545299048, 0.8860269688856748, 0.1026459672214125},`。
- **L1847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8455623467619914, 0.8873201416511192, 0.09970160643955417},`.
  - **L1847 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8455623467619914, 0.8873201416511192, 0.09970160643955417},`。
- **L1848 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8558102197403156, 0.8885989857373076, 0.09745185636949635},`.
  - **L1848 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8558102197403156, 0.8885989857373076, 0.09745185636949635},`。

### Lines 1849-1876 / 第 1849-1876 行

````cpp
1849:      {0.8660121015321746, 0.8898658327014707, 0.09595302142849352},
1850:      {0.876168577861264, 0.8911230006650279, 0.09524987622820644},
1851:      {0.8862704693847823, 0.8923718488516762, 0.09537386433165168},
1852:      {0.8963209276849421, 0.8936140145545759, 0.09633501924437288},
1853:      {0.9063108305457804, 0.8948528630958348, 0.0981248266153556},
1854:      {0.916243267627844, 0.8960890285051739, 0.10071727474383468},
1855:      {0.9261061834911258, 0.8973278764153646, 0.10407091253184719},
1856:      {0.9359031089867296, 0.8985677257631529, 0.10813047743488122},
1857:      {0.9456365256783655, 0.899812890310841, 0.112838121119233},
1858:      {0.9552994651064829, 0.9010627378634819, 0.11812754907477127},
1859:      {0.9648948576822979, 0.9023209051300205, 0.12394140017411451},
1860:      {0.9744168120052771, 0.9035877501583347, 0.13021471316475502},
1861:      {0.9838691793408701, 0.9048649215326077, 0.13689772464133854},
1862:      {0.9932481489335602, 0.9061547634208059, 0.14393594366968385}}};
1863: 
1864: template <typename Real>
1865: inline std::array<Real, 3>
1866: color_map_(Real scalar, std::array<std::array<Real, 3>, 256> const &table) {
1867:   static_assert(std::is_floating_point_v<Real>,
1868:                 "Color tables are only implemented in floating point "
1869:                 "arithmetic. If you require bytes please submit an issue or "
1870:                 "pull request");
1871: 
1872:   using boost::math::isnan;
1873: 
1874:   if ((isnan)(scalar))
1875:   {
1876:       scalar = static_cast<Real>(0);
````
- **L1849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8660121015321746, 0.8898658327014707, 0.09595302142849352},`.
  - **L1849 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8660121015321746, 0.8898658327014707, 0.09595302142849352},`。
- **L1850 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.876168577861264, 0.8911230006650279, 0.09524987622820644},`.
  - **L1850 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.876168577861264, 0.8911230006650279, 0.09524987622820644},`。
- **L1851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8862704693847823, 0.8923718488516762, 0.09537386433165168},`.
  - **L1851 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8862704693847823, 0.8923718488516762, 0.09537386433165168},`。
- **L1852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.8963209276849421, 0.8936140145545759, 0.09633501924437288},`.
  - **L1852 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.8963209276849421, 0.8936140145545759, 0.09633501924437288},`。
- **L1853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9063108305457804, 0.8948528630958348, 0.0981248266153556},`.
  - **L1853 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9063108305457804, 0.8948528630958348, 0.0981248266153556},`。
- **L1854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.916243267627844, 0.8960890285051739, 0.10071727474383468},`.
  - **L1854 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.916243267627844, 0.8960890285051739, 0.10071727474383468},`。
- **L1855 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9261061834911258, 0.8973278764153646, 0.10407091253184719},`.
  - **L1855 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9261061834911258, 0.8973278764153646, 0.10407091253184719},`。
- **L1856 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9359031089867296, 0.8985677257631529, 0.10813047743488122},`.
  - **L1856 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9359031089867296, 0.8985677257631529, 0.10813047743488122},`。
- **L1857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9456365256783655, 0.899812890310841, 0.112838121119233},`.
  - **L1857 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9456365256783655, 0.899812890310841, 0.112838121119233},`。
- **L1858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9552994651064829, 0.9010627378634819, 0.11812754907477127},`.
  - **L1858 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9552994651064829, 0.9010627378634819, 0.11812754907477127},`。
- **L1859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9648948576822979, 0.9023209051300205, 0.12394140017411451},`.
  - **L1859 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9648948576822979, 0.9023209051300205, 0.12394140017411451},`。
- **L1860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9744168120052771, 0.9035877501583347, 0.13021471316475502},`.
  - **L1860 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9744168120052771, 0.9035877501583347, 0.13021471316475502},`。
- **L1861 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `{0.9838691793408701, 0.9048649215326077, 0.13689772464133854},`.
  - **L1861 CN**: 继续一个多行参数列表、初始化器或聚合项：`{0.9838691793408701, 0.9048649215326077, 0.13689772464133854},`。
- **L1862 EN**: Executes a standalone statement or declaration: `{0.9932481489335602, 0.9061547634208059, 0.14393594366968385}}};`.
  - **L1862 CN**: 执行一条独立语句或声明：`{0.9932481489335602, 0.9061547634208059, 0.14393594366968385}}};`。
- **L1863 EN**: Blank line separating nearby declarations or logic.
  - **L1863 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1864 EN**: Introduces template parameters or specialization context: `template <typename Real>`.
  - **L1864 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real>`。
- **L1865 EN**: Continues the surrounding expression or declaration: `inline std::array<Real, 3>`.
  - **L1865 CN**: 继续构造周围的表达式或声明：`inline std::array<Real, 3>`。
- **L1866 EN**: Starts a function, method, lambda, or structured scope: `color_map_(Real scalar, std::array<std::array<Real, 3>, 256> const &table) {`.
  - **L1866 CN**: 开始一个函数、方法、lambda 或结构化作用域：`color_map_(Real scalar, std::array<std::array<Real, 3>, 256> const &table) {`。
- **L1867 EN**: Checks a compile-time invariant and rejects invalid configurations early.
  - **L1867 CN**: 检查编译期不变式，并及早拒绝无效配置。
- **L1868 EN**: Continues the surrounding expression or declaration: `"Color tables are only implemented in floating point "`.
  - **L1868 CN**: 继续构造周围的表达式或声明：`"Color tables are only implemented in floating point "`。
- **L1869 EN**: Continues the surrounding expression or declaration: `"arithmetic. If you require bytes please submit an issue or "`.
  - **L1869 CN**: 继续构造周围的表达式或声明：`"arithmetic. If you require bytes please submit an issue or "`。
- **L1870 EN**: Executes a standalone statement or declaration: `"pull request");`.
  - **L1870 CN**: 执行一条独立语句或声明：`"pull request");`。
- **L1871 EN**: Blank line separating nearby declarations or logic.
  - **L1871 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1872 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1872 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1873 EN**: Blank line separating nearby declarations or logic.
  - **L1873 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1874 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1874 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1875 EN**: Opens a new lexical scope or compound statement.
  - **L1875 CN**: 打开一个新的词法作用域或复合语句块。
- **L1876 EN**: Executes a call or declaration centered on `static_cast<Real>`.
  - **L1876 CN**: 执行以 `static_cast<Real>` 为核心的调用或声明。

### Lines 1877-1904 / 第 1877-1904 行

````cpp
1877:   }
1878:   else
1879:   {
1880:       scalar = std::clamp(scalar, static_cast<Real>(0), static_cast<Real>(1));
1881:   }
1882: 
1883:   if (scalar == static_cast<Real>(1)) {
1884:     return table.back();
1885:   }
1886: 
1887:   Real s = (table.size() - 1) * scalar;
1888:   Real ii = std::floor(s);
1889:   Real t = s - ii;
1890:   auto i = static_cast<std::size_t>(ii);
1891:   auto const &rgb0 = table[i];
1892:   auto const &rgb1 = table[i + 1];
1893:   return {(1 - t) * rgb0[0] + t * rgb1[0], (1 - t) * rgb0[1] + t * rgb1[1],
1894:           (1 - t) * rgb0[2] + t * rgb1[2]};
1895: }
1896: } // namespace detail
1897: 
1898: template <typename Real = float> std::array<Real, 3> viridis(Real x) {
1899:   return detail::color_map_<Real>(x, detail::viridis_data_<Real>);
1900: }
1901: 
1902: template <typename Real = float> std::array<Real, 3> smooth_cool_warm(Real x) {
1903:   return detail::color_map_<Real>(x, detail::smooth_cool_warm_data_<Real>);
1904: }
````
- **L1877 EN**: Closes the current lexical scope or compound statement.
  - **L1877 CN**: 结束当前词法作用域或复合语句块。
- **L1878 EN**: Starts the alternative branch of the preceding conditional.
  - **L1878 CN**: 开始前一个条件语句的备选分支。
- **L1879 EN**: Opens a new lexical scope or compound statement.
  - **L1879 CN**: 打开一个新的词法作用域或复合语句块。
- **L1880 EN**: Executes a call or declaration centered on `std::clamp`.
  - **L1880 CN**: 执行以 `std::clamp` 为核心的调用或声明。
- **L1881 EN**: Closes the current lexical scope or compound statement.
  - **L1881 CN**: 结束当前词法作用域或复合语句块。
- **L1882 EN**: Blank line separating nearby declarations or logic.
  - **L1882 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1883 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L1883 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1884 EN**: Returns from the current function with `table.back()`.
  - **L1884 CN**: 以 `table.back()` 从当前函数返回。
- **L1885 EN**: Closes the current lexical scope or compound statement.
  - **L1885 CN**: 结束当前词法作用域或复合语句块。
- **L1886 EN**: Blank line separating nearby declarations or logic.
  - **L1886 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1887 EN**: Initializes variable `s` from the right-hand expression.
  - **L1887 CN**: 使用右侧表达式初始化变量 `s`。
- **L1888 EN**: Initializes variable `ii` from the right-hand expression.
  - **L1888 CN**: 使用右侧表达式初始化变量 `ii`。
- **L1889 EN**: Initializes variable `t` from the right-hand expression.
  - **L1889 CN**: 使用右侧表达式初始化变量 `t`。
- **L1890 EN**: Initializes variable `i` from the right-hand expression.
  - **L1890 CN**: 使用右侧表达式初始化变量 `i`。
- **L1891 EN**: Executes a standalone statement or declaration: `auto const &rgb0 = table[i];`.
  - **L1891 CN**: 执行一条独立语句或声明：`auto const &rgb0 = table[i];`。
- **L1892 EN**: Executes a standalone statement or declaration: `auto const &rgb1 = table[i + 1];`.
  - **L1892 CN**: 执行一条独立语句或声明：`auto const &rgb1 = table[i + 1];`。
- **L1893 EN**: Returns from the current function with `{(1 - t) * rgb0[0] + t * rgb1[0], (1 - t) * rgb0[1] + t * rgb1[1],`.
  - **L1893 CN**: 以 `{(1 - t) * rgb0[0] + t * rgb1[0], (1 - t) * rgb0[1] + t * rgb1[1],` 从当前函数返回。
- **L1894 EN**: Executes a call or declaration centered on `call site`.
  - **L1894 CN**: 执行以 `call site` 为核心的调用或声明。
- **L1895 EN**: Closes the current lexical scope or compound statement.
  - **L1895 CN**: 结束当前词法作用域或复合语句块。
- **L1896 EN**: Closes a namespace scope while preserving the trailing comment: `} // namespace detail`.
  - **L1896 CN**: 结束一个命名空间作用域，并保留尾部注释：`} // namespace detail`。
- **L1897 EN**: Blank line separating nearby declarations or logic.
  - **L1897 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1898 EN**: Introduces template parameters or specialization context: `template <typename Real = float> std::array<Real, 3> viridis(Real x) {`.
  - **L1898 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real = float> std::array<Real, 3> viridis(Real x) {`。
- **L1899 EN**: Returns from the current function with `detail::color_map_<Real>(x, detail::viridis_data_<Real>)`.
  - **L1899 CN**: 以 `detail::color_map_<Real>(x, detail::viridis_data_<Real>)` 从当前函数返回。
- **L1900 EN**: Closes the current lexical scope or compound statement.
  - **L1900 CN**: 结束当前词法作用域或复合语句块。
- **L1901 EN**: Blank line separating nearby declarations or logic.
  - **L1901 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1902 EN**: Introduces template parameters or specialization context: `template <typename Real = float> std::array<Real, 3> smooth_cool_warm(Real x) {`.
  - **L1902 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real = float> std::array<Real, 3> smooth_cool_warm(Real x) {`。
- **L1903 EN**: Returns from the current function with `detail::color_map_<Real>(x, detail::smooth_cool_warm_data_<Real>)`.
  - **L1903 CN**: 以 `detail::color_map_<Real>(x, detail::smooth_cool_warm_data_<Real>)` 从当前函数返回。
- **L1904 EN**: Closes the current lexical scope or compound statement.
  - **L1904 CN**: 结束当前词法作用域或复合语句块。

### Lines 1905-1932 / 第 1905-1932 行

````cpp
1905: 
1906: template <typename Real = float> std::array<Real, 3> plasma(Real x) {
1907:   return detail::color_map_<Real>(x, detail::plasma_data_<Real>);
1908: }
1909: 
1910: template <typename Real = float> std::array<Real, 3> black_body(Real x) {
1911:   return detail::color_map_<Real>(x, detail::black_body_data_<Real>);
1912: }
1913: 
1914: template <typename Real = float> std::array<Real, 3> inferno(Real x) {
1915:   return detail::color_map_<Real>(x, detail::inferno_data_<Real>);
1916: }
1917: 
1918: template <typename Real = float> std::array<Real, 3> kindlmann(Real x) {
1919:   return detail::color_map_<Real>(x, detail::kindlmann_data_<Real>);
1920: }
1921: 
1922: template <typename Real = float>
1923: std::array<Real, 3> extended_kindlmann(Real x) {
1924:   return detail::color_map_<Real>(x, detail::extended_kindlmann_data_<Real>);
1925: }
1926: 
1927: template <typename Real>
1928: std::array<std::uint8_t, 4> to_8bit_rgba(const std::array<Real, 3> &v) {
1929:   using std::sqrt;
1930:   std::array<std::uint8_t, 4> pixel {};
1931:   for (auto i = 0; i < 3; ++i) {
1932:     // Apply gamma correction here:
````
- **L1905 EN**: Blank line separating nearby declarations or logic.
  - **L1905 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1906 EN**: Introduces template parameters or specialization context: `template <typename Real = float> std::array<Real, 3> plasma(Real x) {`.
  - **L1906 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real = float> std::array<Real, 3> plasma(Real x) {`。
- **L1907 EN**: Returns from the current function with `detail::color_map_<Real>(x, detail::plasma_data_<Real>)`.
  - **L1907 CN**: 以 `detail::color_map_<Real>(x, detail::plasma_data_<Real>)` 从当前函数返回。
- **L1908 EN**: Closes the current lexical scope or compound statement.
  - **L1908 CN**: 结束当前词法作用域或复合语句块。
- **L1909 EN**: Blank line separating nearby declarations or logic.
  - **L1909 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1910 EN**: Introduces template parameters or specialization context: `template <typename Real = float> std::array<Real, 3> black_body(Real x) {`.
  - **L1910 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real = float> std::array<Real, 3> black_body(Real x) {`。
- **L1911 EN**: Returns from the current function with `detail::color_map_<Real>(x, detail::black_body_data_<Real>)`.
  - **L1911 CN**: 以 `detail::color_map_<Real>(x, detail::black_body_data_<Real>)` 从当前函数返回。
- **L1912 EN**: Closes the current lexical scope or compound statement.
  - **L1912 CN**: 结束当前词法作用域或复合语句块。
- **L1913 EN**: Blank line separating nearby declarations or logic.
  - **L1913 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1914 EN**: Introduces template parameters or specialization context: `template <typename Real = float> std::array<Real, 3> inferno(Real x) {`.
  - **L1914 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real = float> std::array<Real, 3> inferno(Real x) {`。
- **L1915 EN**: Returns from the current function with `detail::color_map_<Real>(x, detail::inferno_data_<Real>)`.
  - **L1915 CN**: 以 `detail::color_map_<Real>(x, detail::inferno_data_<Real>)` 从当前函数返回。
- **L1916 EN**: Closes the current lexical scope or compound statement.
  - **L1916 CN**: 结束当前词法作用域或复合语句块。
- **L1917 EN**: Blank line separating nearby declarations or logic.
  - **L1917 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1918 EN**: Introduces template parameters or specialization context: `template <typename Real = float> std::array<Real, 3> kindlmann(Real x) {`.
  - **L1918 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real = float> std::array<Real, 3> kindlmann(Real x) {`。
- **L1919 EN**: Returns from the current function with `detail::color_map_<Real>(x, detail::kindlmann_data_<Real>)`.
  - **L1919 CN**: 以 `detail::color_map_<Real>(x, detail::kindlmann_data_<Real>)` 从当前函数返回。
- **L1920 EN**: Closes the current lexical scope or compound statement.
  - **L1920 CN**: 结束当前词法作用域或复合语句块。
- **L1921 EN**: Blank line separating nearby declarations or logic.
  - **L1921 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1922 EN**: Introduces template parameters or specialization context: `template <typename Real = float>`.
  - **L1922 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real = float>`。
- **L1923 EN**: Starts a function or method definition for `extended_kindlmann`.
  - **L1923 CN**: 开始定义函数或方法 `extended_kindlmann`。
- **L1924 EN**: Returns from the current function with `detail::color_map_<Real>(x, detail::extended_kindlmann_data_<Real>)`.
  - **L1924 CN**: 以 `detail::color_map_<Real>(x, detail::extended_kindlmann_data_<Real>)` 从当前函数返回。
- **L1925 EN**: Closes the current lexical scope or compound statement.
  - **L1925 CN**: 结束当前词法作用域或复合语句块。
- **L1926 EN**: Blank line separating nearby declarations or logic.
  - **L1926 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1927 EN**: Introduces template parameters or specialization context: `template <typename Real>`.
  - **L1927 CN**: 为后续声明引入模板参数或特化上下文：`template <typename Real>`。
- **L1928 EN**: Starts a function or method definition for `to_8bit_rgba`.
  - **L1928 CN**: 开始定义函数或方法 `to_8bit_rgba`。
- **L1929 EN**: Executes a standalone statement or declaration: `using std::sqrt;`.
  - **L1929 CN**: 执行一条独立语句或声明：`using std::sqrt;`。
- **L1930 EN**: Executes a standalone statement or declaration: `std::array<std::uint8_t, 4> pixel {};`.
  - **L1930 CN**: 执行一条独立语句或声明：`std::array<std::uint8_t, 4> pixel {};`。
- **L1931 EN**: Begins a `for` control-flow statement and evaluates its condition.
  - **L1931 CN**: 开始 `for` 控制流语句并计算其条件。
- **L1932 EN**: Comment documents nearby intent or usage notes: `Apply gamma correction here:`.
  - **L1932 CN**: 注释说明附近代码的意图或使用说明：`Apply gamma correction here:`。

### Lines 1933-1943 / 第 1933-1943 行

````cpp
1933:     Real u = sqrt(v[i]);
1934:     pixel[i] = 255 * std::clamp(u, static_cast<Real>(0), static_cast<Real>(1));
1935:   }
1936: 
1937:   pixel[3] = 255;
1938:   return pixel;
1939: }
1940: 
1941: } // Namespace boost::math::tools
1942: 
1943: #endif // BOOST_MATH_COLOR_MAPS_HPP
````
- **L1933 EN**: Initializes variable `u` from the right-hand expression.
  - **L1933 CN**: 使用右侧表达式初始化变量 `u`。
- **L1934 EN**: Executes a call or declaration centered on `std::clamp`.
  - **L1934 CN**: 执行以 `std::clamp` 为核心的调用或声明。
- **L1935 EN**: Closes the current lexical scope or compound statement.
  - **L1935 CN**: 结束当前词法作用域或复合语句块。
- **L1936 EN**: Blank line separating nearby declarations or logic.
  - **L1936 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1937 EN**: Executes a standalone statement or declaration: `pixel[3] = 255;`.
  - **L1937 CN**: 执行一条独立语句或声明：`pixel[3] = 255;`。
- **L1938 EN**: Returns from the current function with `pixel`.
  - **L1938 CN**: 以 `pixel` 从当前函数返回。
- **L1939 EN**: Closes the current lexical scope or compound statement.
  - **L1939 CN**: 结束当前词法作用域或复合语句块。
- **L1940 EN**: Blank line separating nearby declarations or logic.
  - **L1940 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1941 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L1941 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L1942 EN**: Blank line separating nearby declarations or logic.
  - **L1942 CN**: 空行，用于分隔相邻声明或逻辑。
- **L1943 EN**: Closes the current preprocessor conditional block or header guard.
  - **L1943 CN**: 结束当前预处理条件块或头文件保护。

## Key Concepts / 关键概念

- **Numeric support layer / 数值支撑层**:
  - **EN**: Provides low-level traits, constants, policy hooks, and helpers reused across Boost.Math algorithms.
  - **CN**: 提供可在 Boost.Math 算法之间复用的底层 traits、常量、策略钩子与辅助逻辑。
- **Template genericity / 模板泛型**:
  - **EN**: Uses C++ templates to adapt behavior across types, policies, or compile-time parameters.
  - **CN**: 使用 C++ 模板使行为适配不同类型、策略或编译期参数。
- **Type traits and metaprogramming / 类型 traits 与元编程**:
  - **EN**: Uses compile-time traits to select implementations and validate types.
  - **CN**: 使用编译期 traits 来选择实现并校验类型。
- **Assertions and contracts / 断言与契约**:
  - **EN**: Checks preconditions, postconditions, or test outcomes to keep behavior well defined.
  - **CN**: 检查前置条件、后置条件或测试结果，以保持行为有明确定义。
- **Compile-time evaluation / 编译期求值**:
  - **EN**: Marks values or functions so some work can be performed during compilation.
  - **CN**: 标记值或函数，使部分工作可在编译期间完成。
- **Boost integration / Boost 集成**:
  - **EN**: Fits the implementation into Boost naming, configuration, and portability conventions.
  - **CN**: 让实现融入 Boost 的命名、配置与可移植性约定。
- **Dependency layering / 依赖分层**:
  - **EN**: Builds on included headers instead of re-declaring reusable infrastructure.
  - **CN**: 构建在已包含头文件之上，而不是重复声明可复用基础设施。
- **Header contracts / 头文件契约**:
  - **EN**: Provides declarations, inline logic, or macros that other translation units consume.
  - **CN**: 提供供其他编译单元使用的声明、内联逻辑或宏。

## Dependencies / 依赖关系

- **Direct local/internal includes / 直接本地或内部包含**: `algorithm`, `array`, `cmath`, `cstdint`, `boost/math/special_functions/fpclassify.hpp`, `lodepng.h`, `iostream`, `string`, `vector`
- **Dependency categories / 依赖类别**: C or C++ standard library facilities / C 或 C++ 标准库设施 (7), Boost.Math special-function declarations / Boost.Math 特殊函数声明 (1), nearby local declarations / 附近的本地声明 (1)

- **EN**: `algorithm` provides C or C++ standard library facilities.
  - **CN**: `algorithm` 提供C 或 C++ 标准库设施。
- **EN**: `array` provides C or C++ standard library facilities.
  - **CN**: `array` 提供C 或 C++ 标准库设施。
- **EN**: `cmath` provides C or C++ standard library facilities.
  - **CN**: `cmath` 提供C 或 C++ 标准库设施。
- **EN**: `cstdint` provides C or C++ standard library facilities.
  - **CN**: `cstdint` 提供C 或 C++ 标准库设施。
- **EN**: `boost/math/special_functions/fpclassify.hpp` provides Boost.Math special-function declarations.
  - **CN**: `boost/math/special_functions/fpclassify.hpp` 提供Boost.Math 特殊函数声明。
- **EN**: `lodepng.h` provides nearby local declarations.
  - **CN**: `lodepng.h` 提供附近的本地声明。
- **EN**: `iostream` provides C or C++ standard library facilities.
  - **CN**: `iostream` 提供C 或 C++ 标准库设施。
- **EN**: `string` provides C or C++ standard library facilities.
  - **CN**: `string` 提供C 或 C++ 标准库设施。
- **EN**: `vector` provides C or C++ standard library facilities.
  - **CN**: `vector` 提供C 或 C++ 标准库设施。
