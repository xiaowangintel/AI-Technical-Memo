# lyra.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/lyra/lyra.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements JNI bridge abstractions used by Android-facing native code.
- 用途 (CN): 实现供 Android 原生代码使用的 JNI 桥接抽象。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````cpp
 1: /*
 2:  * Copyright (c) Facebook, Inc. and its affiliates.
 3:  *
 4:  * Licensed under the Apache License, Version 2.0 (the "License");
 5:  * you may not use this file except in compliance with the License.
 6:  * You may obtain a copy of the License at
 7:  *
 8:  *     http://www.apache.org/licenses/LICENSE-2.0
 9:  *
10:  * Unless required by applicable law or agreed to in writing, software
11:  * distributed under the License is distributed on an "AS IS" BASIS,
12:  * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
13:  * See the License for the specific language governing permissions and
14:  * limitations under the License.
15:  */
16: 
17: #include <lyra/lyra.h>
18: 
19: #include <atomic>
20: #include <ios>
21: #include <ostream>
22: #include <iomanip>
23: #include <memory>
24: #include <vector>
25: 
26: #ifndef _WIN32
27: #include <dlfcn.h>
28: #include <unwind.h>
29: #endif
30: 
31: #include <fbjni/detail/Log.h>
32: 
33: using namespace std;
34: 
35: namespace facebook {
36: namespace lyra {
37: 
38: namespace {
39: 
40: class IosFlagsSaver {
````
- EN: Pulls in native headers such as `lyra/lyra.h`, `atomic`, `ios`, `ostream`.
- CN: 引入原生头文件，例如 `lyra/lyra.h`, `atomic`, `ios`, `ostream`。
- EN: Declares or extends types including `IosFlagsSaver`.
- CN: 声明或扩展类型，包括 `IosFlagsSaver`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41:   ios_base& ios_;
42:   ios_base::fmtflags flags_;
43: 
44:  public:
45:   IosFlagsSaver(ios_base& ios)
46:   : ios_(ios),
47:     flags_(ios.flags())
48:   {}
49: 
50:   ~IosFlagsSaver() {
51:     ios_.flags(flags_);
52:   }
53: };
54: 
55: struct BacktraceState {
56:   size_t skip;
57:   vector<InstructionPointer>& stackTrace;
58: };
59: 
60: #ifndef _MSC_VER
61: _Unwind_Reason_Code unwindCallback(struct _Unwind_Context* context, void* arg) {
62:   BacktraceState* state = reinterpret_cast<BacktraceState*>(arg);
63:   auto absoluteProgramCounter =
64:       reinterpret_cast<InstructionPointer>(_Unwind_GetIP(context));
65: 
66:   if (state->skip > 0) {
67:     --state->skip;
68:     return _URC_NO_REASON;
69:   }
70: 
71:   if (state->stackTrace.size() == state->stackTrace.capacity()) {
72:     return _URC_END_OF_STACK;
73:   }
74: 
75:   state->stackTrace.push_back(absoluteProgramCounter);
76: 
77:   return _URC_NO_REASON;
78: }
79: #endif
80: 
````
- EN: Declares or extends types including `BacktraceState`, `_Unwind_Context`.
- CN: 声明或扩展类型，包括 `BacktraceState`, `_Unwind_Context`。
- EN: Implements callable logic such as `IosFlagsSaver`, `~IosFlagsSaver`, `unwindCallback`.
- CN: 实现可调用逻辑，例如 `IosFlagsSaver`, `~IosFlagsSaver`, `unwindCallback`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````cpp
 81: void captureBacktrace(size_t skip, vector<InstructionPointer>& stackTrace) {
 82:   // Beware of a bug on some platforms, which makes the trace loop until the
 83:   // buffer is full when it reaches a noexcept function. It seems to be fixed in
 84:   // newer versions of gcc. https://gcc.gnu.org/bugzilla/show_bug.cgi?id=56846
 85:   // TODO(t10738439): Investigate workaround for the stack trace bug
 86:   BacktraceState state = {skip, stackTrace};
 87: #ifndef _WIN32
 88:   _Unwind_Backtrace(unwindCallback, &state);
 89: #endif
 90: }
 91: 
 92: // this is a pointer to a function
 93: std::atomic<LibraryIdentifierFunctionType> gLibraryIdentifierFunction{nullptr};
 94: 
 95: }
 96: 
 97: void setLibraryIdentifierFunction(LibraryIdentifierFunctionType func) {
 98:   gLibraryIdentifierFunction.store(func, std::memory_order_relaxed);
 99: }
100: 
101: std::string StackTraceElement::buildId() const {
102:   if (!hasBuildId_) {
103:     auto getBuildId = gLibraryIdentifierFunction.load(std::memory_order_relaxed);
104:     if (getBuildId) {
105:       buildId_ = getBuildId(libraryName());
106:     } else {
107:       buildId_ = "<unimplemented>";
108:     }
109:     hasBuildId_ = true;
110:   }
111:   return buildId_;
112: }
113: 
114: void getStackTrace(vector<InstructionPointer>& stackTrace, size_t skip) {
115:   stackTrace.clear();
116:   captureBacktrace(skip + 1, stackTrace);
117: }
118: 
119: // TODO(t10737622): Improve on-device symbolification
120: void getStackTraceSymbols(vector<StackTraceElement>& symbols,
````
- EN: Implements callable logic such as `captureBacktrace`, `setLibraryIdentifierFunction`, `StackTraceElement::buildId`, `getStackTrace`.
- CN: 实现可调用逻辑，例如 `captureBacktrace`, `setLibraryIdentifierFunction`, `StackTraceElement::buildId`, `getStackTrace`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-160
````cpp
121:                           const vector<InstructionPointer>& trace) {
122:   symbols.clear();
123:   symbols.reserve(trace.size());
124: 
125: #ifndef _WIN32
126:   for (size_t i = 0; i < trace.size(); ++i) {
127:     Dl_info info;
128:     if (dladdr(trace[i], &info)) {
129:       symbols.emplace_back(trace[i], info.dli_fbase, info.dli_saddr,
130:                            info.dli_fname ? info.dli_fname : "",
131:                            info.dli_sname ? info.dli_sname : "");
132:     }
133:   }
134: #endif
135: }
136: 
137: ostream& operator<<(ostream& out, const StackTraceElement& elm) {
138:   IosFlagsSaver flags{out};
139: 
140:   out << "{dso=" << elm.libraryName() << " offset=" << hex
141:       << showbase << elm.libraryOffset();
142: 
143:   if (!elm.functionName().empty()) {
144:     out << " func=" << elm.functionName() << "()+" << elm.functionOffset();
145:   }
146: 
147:   out << " build-id=" << hex << setw(8) << elm.buildId()
148:       << "}";
149: 
150:   return out;
151: }
152: 
153: // TODO(t10737667): The implement a tool that parse the stack trace and
154: // symbolicate it
155: ostream& operator<<(ostream& out, const vector<StackTraceElement>& trace) {
156:   IosFlagsSaver flags{out};
157: 
158:   auto i = 0;
159:   out << "Backtrace:\n";
160:   for (auto& elm : trace) {
````
- EN: Implements callable logic such as `TODO`.
- CN: 实现可调用逻辑，例如 `TODO`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 161-190
````cpp
161:     out << "    #" << dec << setfill('0') << setw(2) << i++ << " " << elm << '\n';
162:   }
163: 
164:   return out;
165: }
166: 
167: void logStackTrace(const vector<StackTraceElement>& trace) {
168:   auto i = 0;
169:   FBJNI_LOGE("Backtrace:");
170:   for (auto& elm : trace) {
171:     if (!elm.functionName().empty()) {
172:       FBJNI_LOGE("    #%02d |lyra|{dso=%s offset=%#tx func=%s+%#x build-id=%s}",
173:           i++,
174:           elm.libraryName().c_str(),
175:           elm.libraryOffset(),
176:           elm.functionName().c_str(),
177:           elm.functionOffset(),
178:           elm.buildId().c_str());
179:     } else {
180:       FBJNI_LOGE("    #%02d |lyra|{dso=%s offset=%#tx build-id=%s}",
181:           i++,
182:           elm.libraryName().c_str(),
183:           elm.libraryOffset(),
184:           elm.buildId().c_str());
185:     }
186:   }
187: }
188: 
189: }
190: }
````
- EN: Implements callable logic such as `logStackTrace`.
- CN: 实现可调用逻辑，例如 `logStackTrace`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `IosFlagsSaver` / 符号 `IosFlagsSaver`
- Symbol `BacktraceState` / 符号 `BacktraceState`
- Symbol `_Unwind_Context` / 符号 `_Unwind_Context`
- Symbol `~IosFlagsSaver` / 符号 `~IosFlagsSaver`

## Dependencies / 依赖关系
- C/C++ includes: `lyra/lyra.h`, `atomic`, `ios`, `ostream`, `iomanip`, `memory`, `vector`, `dlfcn.h`, `unwind.h`, `fbjni/detail/Log.h`
- C/C++ 头文件: `lyra/lyra.h`, `atomic`, `ios`, `ostream`, `iomanip`, `memory`, `vector`, `dlfcn.h`, `unwind.h`, `fbjni/detail/Log.h`
