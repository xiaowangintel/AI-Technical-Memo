# torchscript_functions.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/rpc/torchscript_functions.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for torchscript functions in the distributed RPC layer. Representative routines include `rpcTorchscript`, `remoteTorchscript`.
- 用途 (CN): 该文件在分布式 RPC 层中提供torchscript functions 的实现逻辑。 代表性例程包括 `rpcTorchscript`、`remoteTorchscript`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```cpp
1: #include <ATen/ThreadLocalState.h>
2: #include <fmt/format.h>
3: #include <torch/csrc/autograd/record_function_ops.h>
4: #include <torch/csrc/distributed/autograd/utils.h>
5: #include <torch/csrc/distributed/rpc/message.h>
6: #include <torch/csrc/distributed/rpc/profiler/remote_profiler_manager.h>
7: #include <torch/csrc/distributed/rpc/rpc_agent.h>
8: #include <torch/csrc/distributed/rpc/script_call.h>
9: #include <torch/csrc/distributed/rpc/torchscript_functions.h>
10: #include <torch/csrc/distributed/rpc/utils.h>
11: 
12: namespace torch::distributed::rpc {
```

- EN: Lines 1-12 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 1-12 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 13-24 / 第 13-24 行

```cpp
13: 
14: c10::intrusive_ptr<JitFuture> rpcTorchscript(
15:     const std::string& dstWorkerName,
16:     const c10::QualifiedName& qualifiedName,
17:     const c10::FunctionSchema& functionSchema,
18:     std::vector<c10::IValue> stack,
19:     const float rpcTimeoutSeconds,
20:     const bool isAsyncExecution) {
21:   c10::intrusive_ptr<torch::autograd::profiler::PythonRecordFunction> record;
22:   auto shouldProfile = torch::autograd::profiler::profilerEnabled() &&
23:       !torch::distributed::rpc::RemoteProfilerManager::getInstance()
24:            .isCurrentKeySet();
```

- EN: Lines 13-24 introduces executable logic in routines such as `rpcTorchscript`.
- CN: 第 13-24 行在 `rpcTorchscript` 等例程中引入具体执行逻辑。

### Lines 25-36 / 第 25-36 行

```cpp
25:   if (shouldProfile) {
26:     auto rpcAsyncJitKey = fmt::format(
27:         "rpc_async_jit#{}({} -> {})",
28:         qualifiedName
29:             .qualifiedName(), /* name of torchscript function being run */
30:         RpcAgent::getCurrentRpcAgent()->getWorkerInfo().name_,
31:         dstWorkerName);
32:     record =
33:         torch::autograd::profiler::record_function_enter_new(rpcAsyncJitKey);
34:     auto& remoteProfilerManager =
35:         torch::distributed::rpc::RemoteProfilerManager::getInstance();
36:     remoteProfilerManager.setCurrentKey(rpcAsyncJitKey);
```

- EN: Lines 25-36 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 25-36 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 37-48 / 第 37-48 行

```cpp
37:   }
38:   auto scriptCall = std::make_unique<ScriptCall>(
39:       qualifiedName, std::move(stack), isAsyncExecution);
40:   auto rpcAgentPtr = RpcAgent::getCurrentRpcAgent();
41:   auto jitFuture = autograd::sendMessageWithAutograd(
42:       *rpcAgentPtr,
43:       rpcAgentPtr->getWorkerInfo(dstWorkerName),
44:       std::move(*scriptCall).toMessage(),
45:       true /*forceGradRecording*/,
46:       rpcTimeoutSeconds);
47: 
48:   // Get function return type to construct JitFuture.
```

- EN: Lines 37-48 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 37-48 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 49-60 / 第 49-60 行

```cpp
49:   auto const& returns = functionSchema.returns();
50:   // Script call only allows single IValue returned.
51:   TORCH_INTERNAL_ASSERT(
52:       returns.size() == 1,
53:       "Return value of an annotated torchScript function should be a single "
54:       "IValue.",
55:       returns.size());
56:   auto returnType = returns.at(0).type();
57: 
58:   // Create a JIT future and pass it to futMessage's callback to set state
59:   // of the JIT future.
60:   auto futPtr = jitFuture->createInstance(returnType);
```

- EN: Lines 49-60 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 49-60 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 61-72 / 第 61-72 行

```cpp
61:   jitFuture->addCallback(at::wrapPropagateTLSState([futPtr](JitFuture& future) {
62:     if (future.hasError()) {
63:       futPtr->setError(future.exception_ptr());
64:     } else {
65:       futPtr->markCompleted(
66:           deserializeRespToIValue(
67:               *future.constValue().toCustomClass<Message>()),
68:           future.storages());
69:     }
70:   }));
71:   if (shouldProfile) {
72:     auto profiledFutPtr =
```

- EN: Lines 61-72 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 61-72 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 73-84 / 第 73-84 行

```cpp
73:         torch::autograd::profiler::_call_end_callbacks_on_fut_new(
74:             record, futPtr);
75:     return profiledFutPtr;
76:   }
77:   return futPtr;
78: }
79: 
80: c10::intrusive_ptr<RRef> remoteTorchscript(
81:     const std::string& dstWorkerName,
82:     const c10::QualifiedName& qualifiedName,
83:     const c10::FunctionSchema& functionSchema,
84:     std::vector<c10::IValue>& stack,
```

- EN: Lines 73-84 returns computed state or forwards results to the surrounding caller.
- CN: 第 73-84 行返回计算结果，或将结果继续传递给外围调用方。

### Lines 85-96 / 第 85-96 行

```cpp
85:     const float rpcTimeoutSeconds,
86:     const bool isAsyncExecution) {
87:   auto rpcAgentPtr = RpcAgent::getCurrentRpcAgent();
88:   auto dstWorkerInfo = rpcAgentPtr->getWorkerInfo(dstWorkerName);
89:   auto& ctx = RRefContext::getInstance();
90: 
91:   // Get function return type to construct UserRRef.
92:   auto const& returns = functionSchema.returns();
93:   // Script call only allows single IValue returned.
94:   TORCH_INTERNAL_ASSERT(
95:       returns.size() == 1,
96:       "Return value of an annotated torchScript function should be a single "
```

- EN: Lines 85-96 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 85-96 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 97-108 / 第 97-108 行

```cpp
97:       "IValue.",
98:       returns.size());
99:   auto returnType = returns.at(0).type();
100: 
101:   if (ctx.getWorkerId() != dstWorkerInfo.id_) {
102:     auto userRRefPtr = ctx.createUserRRef(dstWorkerInfo.id_, returnType);
103: 
104:     auto scriptRemoteCall = std::make_unique<ScriptRemoteCall>(
105:         qualifiedName,
106:         std::move(stack),
107:         userRRefPtr->rrefId(),
108:         userRRefPtr->forkId(),
```

- EN: Lines 97-108 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 97-108 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 109-120 / 第 109-120 行

```cpp
109:         isAsyncExecution);
110: 
111:     auto jitFuture = torch::distributed::autograd::sendMessageWithAutograd(
112:         *rpcAgentPtr,
113:         dstWorkerInfo,
114:         std::move(*scriptRemoteCall).toMessage(),
115:         true /*forceGradRecording*/,
116:         rpcTimeoutSeconds /* timeout */);
117: 
118:     userRRefPtr->registerOwnerCreationFuture(jitFuture);
119:     ctx.addPendingUser(userRRefPtr->forkId(), userRRefPtr);
120:     jitFuture->addCallback(at::wrapPropagateTLSState(
```

- EN: Lines 109-120 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 109-120 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 121-132 / 第 121-132 行

```cpp
121:         [forkId{userRRefPtr->forkId()}](JitFuture& future) {
122:           callback::confirmPendingUser(future, forkId);
123:         }));
124: 
125:     return userRRefPtr;
126:   } else {
127:     auto ownerRRefPtr = ctx.createOwnerRRef(returnType);
128:     // prevent this owner RRef from being deleted due to other forks
129:     ctx.addSelfAsFork(ownerRRefPtr);
130: 
131:     auto scriptRemoteCall = std::make_unique<ScriptRemoteCall>(
132:         qualifiedName,
```

- EN: Lines 121-132 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 121-132 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 133-144 / 第 133-144 行

```cpp
133:         std::move(stack),
134:         ownerRRefPtr->rrefId(),
135:         ownerRRefPtr->rrefId(),
136:         isAsyncExecution);
137: 
138:     auto jitFuture = torch::distributed::autograd::sendMessageWithAutograd(
139:         *rpcAgentPtr,
140:         dstWorkerInfo,
141:         std::move(*scriptRemoteCall).toMessage(),
142:         true /*forceGradRecording*/,
143:         rpcTimeoutSeconds /* timeout */);
144: 
```

- EN: Lines 133-144 continues the local implementation details and data flow for this file.
- CN: 第 133-144 行继续展开本文件的局部实现细节与数据流。

### Lines 145-154 / 第 145-154 行

```cpp
145:     ownerRRefPtr->registerOwnerCreationFuture(jitFuture);
146:     jitFuture->addCallback(at::wrapPropagateTLSState(
147:         [ownerRRefId = ownerRRefPtr->rrefId()](JitFuture& future) {
148:           callback::finishCreatingOwnerRRef(future, ownerRRefId);
149:         }));
150:     return ownerRRefPtr;
151:   }
152: }
153: 
154: } // namespace torch::distributed::rpc
```

- EN: Lines 145-154 opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 145-154 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

## Key Concepts / 关键概念

- EN: Subsystem: distributed RPC layer.
- CN: 子系统：分布式 RPC 层。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `rpcTorchscript`, `remoteTorchscript`
- CN: 核心符号：`rpcTorchscript`、`remoteTorchscript`
- EN: Notable themes: RPC/RRef semantics.
- CN: 值得关注的主题：RPC/RRef 语义。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/autograd/utils.h`, `torch/csrc/distributed/rpc/message.h`, `torch/csrc/distributed/rpc/profiler/remote_profiler_manager.h`, `torch/csrc/distributed/rpc/rpc_agent.h`, `torch/csrc/distributed/rpc/script_call.h`, `torch/csrc/distributed/rpc/torchscript_functions.h`, `torch/csrc/distributed/rpc/utils.h`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/ThreadLocalState.h`, `torch/csrc/autograd/record_function_ops.h`
- External or system headers / 外部或系统头文件: `fmt/format.h`
- Local symbols / 本地符号: `rpcTorchscript`, `remoteTorchscript`