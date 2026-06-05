# logger.hpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/logger.hpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides interface and type declarations for logger in the c10d distributed process-group subsystem. Key types include `ProcessGroupStatus`, `TORCH_API`, `C10dLoggingData`.
- 用途 (CN): 该文件在c10d 分布式进程组子系统中提供logger 的接口与类型声明。 关键类型包括 `ProcessGroupStatus`、`TORCH_API`、`C10dLoggingData`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```cpp
1: #pragma once
2: 
3: #include <c10/util/Logging.h>
4: #include <torch/csrc/distributed/c10d/reducer.hpp>
5: 
6: #include <utility>
7: 
8: namespace c10d {
9: 
10: // A struct to hold the latest status of the process group.
11: struct ProcessGroupStatus {
12:   // the sequential number of the last collective enqueued into workMetaList_
```

- EN: Lines 1-12 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component; declares or defines types such as `ProcessGroupStatus`.
- CN: 第 1-12 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中；声明或定义了 `ProcessGroupStatus` 等类型。

### Lines 13-24 / 第 13-24 行

```cpp
13:   // This is useful for identifying a rank that has not join a collective
14:   // initialized to be -1 to indicate no collective has been enqueued
15:   int64_t lastEnqueuedSeq{-1};
16:   // the sequential number of the last collective started as the kernel
17:   int64_t lastStartedSeq{-1};
18:   // the sequential number of the last collective completed marked by
19:   // the watchdog thread
20:   // initialized to be -1 to indicate no collective has been completed
21:   int64_t lastCompletedSeq{-1};
22: 
23:   // the name of the last collective enqueued into workMetaList_
24:   std::string lastEnqueuedWorkName;
```

- EN: Lines 13-24 continues the local implementation details and data flow for this file.
- CN: 第 13-24 行继续展开本文件的局部实现细节与数据流。

### Lines 25-36 / 第 25-36 行

```cpp
25:   // the name of the last collective started as the kernel
26:   std::string lastStartedWorkName;
27:   // the name of the last collective completed
28:   std::string lastCompletedWorkName;
29: 
30:   // the sizes of the last work enqueued
31:   size_t lastEnqueuedNumelIn;
32:   size_t lastEnqueuedNumelOut;
33:   // the sizes of the last work completed
34:   size_t lastCompletedNumelIn;
35:   size_t lastCompletedNumelOut;
36:   // the sizes of the last work started
```

- EN: Lines 25-36 continues the local implementation details and data flow for this file.
- CN: 第 25-36 行继续展开本文件的局部实现细节与数据流。

### Lines 37-48 / 第 37-48 行

```cpp
37:   size_t lastStartedNumelIn;
38:   size_t lastStartedNumelOut;
39: };
40: 
41: class TORCH_API Logger {
42:  public:
43:   explicit Logger(std::shared_ptr<c10d::Reducer> reducer);
44:   // Set logging data that can be got during DistributedDataParallel
45:   // construction time.
46:   void set_construction_data_and_log(
47:       const std::string& module_name,
48:       const std::vector<int>& device_ids,
```

- EN: Lines 37-48 declares or defines types such as `TORCH_API`; introduces executable logic in routines such as `Logger`.
- CN: 第 37-48 行声明或定义了 `TORCH_API` 等类型；在 `Logger` 等例程中引入具体执行逻辑。

### Lines 49-60 / 第 49-60 行

```cpp
49:       int output_device,
50:       bool broadcast_buffers,
51:       bool has_sync_bn,
52:       bool static_graph);
53: 
54:   void set_static_graph();
55: 
56:   // An interface for users to get DDPLoggingData and log them
57:   // in the applications. Explanation of logging fields are in
58:   // "struct DDPLoggingData" of "torch/c10/util/Logging.h".
59:   at::DDPLoggingData get_ddp_logging_data();
60: 
```

- EN: Lines 49-60 introduces executable logic in routines such as `set_static_graph`, `get_ddp_logging_data`.
- CN: 第 49-60 行在 `set_static_graph`、`get_ddp_logging_data` 等例程中引入具体执行逻辑。

### Lines 61-72 / 第 61-72 行

```cpp
61:   // Stream insertion operator for logging data to stream under
62:   // TORCH_DISTRIBUTED_DEBUG.
63:   friend std::ostream& operator<<(std::ostream& output, const Logger& logger);
64: 
65:   ~Logger() noexcept(false) {
66:     // Log if DDP graph is static in Logger dtor instead of Reducer dtor since
67:     // Logger is deleted before Reducer.
68:     log_if_graph_static(reducer_->ddp_graph_static());
69:   }
70: 
71:   // Set environment variables.
72:   void set_env_variables();
```

- EN: Lines 61-72 introduces executable logic in routines such as `operator<<`, `~Logger`, `set_env_variables`.
- CN: 第 61-72 行在 `operator<<`、`~Logger`、`set_env_variables` 等例程中引入具体执行逻辑。

### Lines 73-84 / 第 73-84 行

```cpp
73:   // Set parameters stats.
74:   void set_parameter_stats();
75:   // Get size of each bucket (Bytes).
76:   std::vector<int64_t> get_bucket_sizes();
77:   // Get variable indices for each bucket.
78:   std::vector<std::vector<size_t>> get_per_bucket_variable_indices();
79:   // Set comm. hook, if used
80:   void set_comm_hook(const std::string& hook);
81:   // Set running with uneven input detection (model.join() context manager)
82:   void set_uneven_input_join();
83: 
84:   // Reset performance stats at current iteration
```

- EN: Lines 73-84 introduces executable logic in routines such as `set_parameter_stats`, `get_bucket_sizes`, `get_per_bucket_variable_indices`.
- CN: 第 73-84 行在 `set_parameter_stats`、`get_bucket_sizes`、`get_per_bucket_variable_indices` 等例程中引入具体执行逻辑。

### Lines 85-96 / 第 85-96 行

```cpp
85:   void reset_performance_stats();
86: 
87:   // Calculate avg stats using cpu timer and gpu timer
88:   // that has been recorded in reducer.
89:   void calculate_avg_time(
90:       int64_t& avg_time,
91:       int64_t& time_duration,
92:       Timer& timer,
93:       Timer::Event start_event,
94:       Timer::Event end_event);
95: 
96:   // Set the absolute time of the event that has been recorded in reducer.
```

- EN: Lines 85-96 introduces executable logic in routines such as `reset_performance_stats`, `calculate_avg_time`.
- CN: 第 85-96 行在 `reset_performance_stats`、`calculate_avg_time` 等例程中引入具体执行逻辑。

### Lines 97-108 / 第 97-108 行

```cpp
97:   void set_event_time(int64_t& event_time, Timer& timer, Timer::Event event);
98:   // Set stats that can be collected only during
99:   // training loop. It is called at the beginning of forward call
100:   // to record the run time stats of sampled iterations that previously ran.
101:   // GPU performance stats are collected only for single process
102:   // single device program and single device module right now.
103:   // TODO to support single process multiple devices and multi device modules,
104:   // events need to be created and recorded on multiple devices.
105:   void set_runtime_stats_and_log();
106: 
107:   // Called when DDP/reducer is failing with an error. The
108:   // logging data structure will have two fields filled: "has_error" indicating
```

- EN: Lines 97-108 introduces executable logic in routines such as `set_event_time`, `set_runtime_stats_and_log`.
- CN: 第 97-108 行在 `set_event_time`、`set_runtime_stats_and_log` 等例程中引入具体执行逻辑。

### Lines 109-120 / 第 109-120 行

```cpp
109:   // that this iteration encountered an error and other fields are not valid,
110:   // and "error", a string which contains the error message that DDP failed
111:   // with.
112:   template <typename... Args>
113:   void set_error_and_log(const std::string& ddp_error, const Args&... args) {
114:     ddp_logging_data_->ints_map["has_error"] = 1;
115:     auto err = c10::str(ddp_error, args...);
116:     ddp_logging_data_->strs_map["error"] = err;
117:     // Report the iteration we are erroring at so user knows how many examples
118:     // successfully processed before this error was hit.
119:     ddp_logging_data_->ints_map["iteration"] = reducer_->num_iterations_;
120:     at::LogPyTorchDDPUsage(*ddp_logging_data_);
```

- EN: Lines 109-120 introduces executable logic in routines such as `set_error_and_log`.
- CN: 第 109-120 行在 `set_error_and_log` 等例程中引入具体执行逻辑。

### Lines 121-132 / 第 121-132 行

```cpp
121:   }
122: 
123:   // When running without static graph, called when reducer is destroyed to log
124:   // if graph was actually static and is a candidate for static graph
125:   // optimization.
126:   void log_if_graph_static(bool is_static);
127: 
128:  private:
129:   // ddp_logging_data_ is used to hold all the ddp related logging
130:   // data fields.
131:   std::unique_ptr<at::DDPLoggingData> ddp_logging_data_;
132:   std::shared_ptr<c10d::Reducer> reducer_;
```

- EN: Lines 121-132 introduces executable logic in routines such as `log_if_graph_static`.
- CN: 第 121-132 行在 `log_if_graph_static` 等例程中引入具体执行逻辑。

### Lines 133-144 / 第 133-144 行

```cpp
133:   // track the number of iterations when runtime stats are collected so far.
134:   long num_iterations_stats_recorded_ = 0;
135: };
136: 
137: // a generic logging data struct that holds different types of logging data.
138: // starting with key value pairs of strings and integers,
139: // It can be extended to more types as needed.
140: struct C10dLoggingData {
141:   // logging fields that are string types.
142:   std::map<std::string, std::string> strings;
143:   // logging fields that are int64_t types.
144:   std::map<std::string, int64_t> integers;
```

- EN: Lines 133-144 declares or defines types such as `C10dLoggingData`.
- CN: 第 133-144 行声明或定义了 `C10dLoggingData` 等类型。

### Lines 145-156 / 第 145-156 行

```cpp
145: };
146: 
147: class TORCH_API C10dLogger {
148:  public:
149:   C10dLogger(const C10dLogger&) = default;
150:   C10dLogger(C10dLogger&&) = delete;
151:   C10dLogger& operator=(const C10dLogger&) = default;
152:   C10dLogger& operator=(C10dLogger&&) = delete;
153:   virtual ~C10dLogger() = default;
154:   virtual void log(const C10dLoggingData& data);
155:   static C10dLogger* getLogger();
156:   static void registerLogger(std::unique_ptr<C10dLogger> /*logger*/);
```

- EN: Lines 145-156 declares or defines types such as `TORCH_API`; introduces executable logic in routines such as `log`, `getLogger`, `registerLogger`.
- CN: 第 145-156 行声明或定义了 `TORCH_API` 等类型；在 `log`、`getLogger`、`registerLogger` 等例程中引入具体执行逻辑。

### Lines 157-168 / 第 157-168 行

```cpp
157: 
158:  protected:
159:   // singletion, hide constructor from the public
160:   C10dLogger(std::string logDestination)
161:       : logDestination_(std::move(logDestination)) {}
162: 
163:   // the name of the destination this logger should log to
164:   std::string logDestination_;
165: 
166:  private:
167:   static std::unique_ptr<C10dLogger> logger_;
168:   static std::atomic<bool> registered_;
```

- EN: Lines 157-168 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 157-168 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 169-171 / 第 169-171 行

```cpp
169: };
170: 
171: } // namespace c10d
```

- EN: Lines 169-171 opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 169-171 行打开或关闭命名空间，以便将代码放入正确的分布式组件中。

## Key Concepts / 关键概念

- EN: Subsystem: c10d distributed process-group subsystem.
- CN: 子系统：c10d 分布式进程组子系统。
- EN: File role: interface/definition.
- CN: 文件角色：接口/定义。
- EN: Primary symbols: `ProcessGroupStatus`, `TORCH_API`, `C10dLoggingData`
- CN: 核心符号：`ProcessGroupStatus`、`TORCH_API`、`C10dLoggingData`
- EN: Notable themes: process-group orchestration, collective communication logic.
- CN: 值得关注的主题：进程组编排、集合通信逻辑。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/reducer.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `c10/util/Logging.h`
- External or system headers / 外部或系统头文件: `utility`
- Local symbols / 本地符号: `ProcessGroupStatus`, `TORCH_API`, `C10dLoggingData`