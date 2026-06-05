# rref_impl.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/rpc/rref_impl.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides implementation logic for rref impl in the distributed RPC layer. Representative routines include `getTypeStr`, `tryDel`, `TORCH_INTERNAL_ASSERT`, `TORCH_CHECK`, `operator<<`.
- 用途 (CN): 该文件在分布式 RPC 层中提供rref impl 的实现逻辑。 代表性例程包括 `getTypeStr`、`tryDel`、`TORCH_INTERNAL_ASSERT`、`TORCH_CHECK`、`operator<<`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #include <torch/csrc/distributed/rpc/rref_impl.h>
2: 
3: #include <ATen/record_function.h>
4: #include <fmt/format.h>
5: #include <torch/csrc/distributed/autograd/utils.h>
6: #include <torch/csrc/distributed/rpc/rref_context.h>
7: #include <torch/csrc/distributed/rpc/rref_proto.h>
8: #include <torch/csrc/distributed/rpc/utils.h>
9: 
10: #include <utility>
11: 
12: namespace {
13: // If the type is subtype of named type, return its qualifiedname, otherwise
14: // return its type str.
15: // NOLINTBEGIN(bugprone-unchecked-optional-access)
16: std::string getTypeStr(const c10::TypePtr& type) {
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `getTypeStr`.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `getTypeStr` 等例程中引入具体执行逻辑。

### Lines 17-32 / 第 17-32 行

```cpp
17:   switch (type->kind()) {
18:     case c10::TypeKind::FunctionType:
19:       return type->castRaw<c10::FunctionType>()->name()->qualifiedName();
20:     case c10::TypeKind::TupleType:
21:       return type->castRaw<c10::TupleType>()->name()->qualifiedName();
22:     case c10::TypeKind::ClassType:
23:       return type->castRaw<c10::ClassType>()->name()->qualifiedName();
24:     case c10::TypeKind::InterfaceType:
25:       return type->castRaw<c10::InterfaceType>()->name()->qualifiedName();
26:     default:
27:       return type->annotation_str();
28:   }
29: }
30: // NOLINTEND(bugprone-unchecked-optional-access)
31: 
32: } // namespace
```

- EN: Lines 17-32 opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 17-32 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 33-48 / 第 33-48 行

```cpp
33: 
34: namespace torch::distributed::rpc {
35: 
36: std::atomic<local_id_t> RRefContext::nextLocalId_{0};
37: 
38: //////////////////////////  RRefForkData  /////////////////////////////////
39: 
40: RRefForkData::RRefForkData(
41:     worker_id_t ownerId,
42:     const RRefId& rrefId,
43:     const ForkId& forkId,
44:     worker_id_t parent,
45:     std::string typeStr)
46:     : ownerId_(ownerId),
47:       rrefId_(rrefId),
48:       forkId_(forkId),
```

- EN: Lines 33-48 opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 33-48 行打开或关闭命名空间，以便将代码放入正确的分布式组件中。

### Lines 49-64 / 第 49-64 行

```cpp
49:       parent_(parent),
50:       typeStr_(std::move(typeStr)) {}
51: 
52: //////////////////////////////  RRef  /////////////////////////////////////
53: 
54: RRef::RRef(worker_id_t ownerId, const RRefId& rrefId, TypePtr type)
55:     : ownerId_(ownerId), rrefId_(rrefId), type_(std::move(type)) {}
56: 
57: RRefForkData RRef::fork() const {
58:   auto& ctx = RRefContext::getInstance();
59:   return RRefForkData(
60:       ownerId_,
61:       rrefId_,
62:       ctx.genGloballyUniqueId(),
63:       ctx.getWorkerId(),
64:       getTypeStr(type_));
```

- EN: Lines 49-64 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 49-64 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 65-80 / 第 65-80 行

```cpp
65: }
66: 
67: void RRef::handleError(RPCErrorType errorType, const JitFuture& jitFuture) {
68:   static std::unordered_map<
69:       RPCErrorType,
70:       std::function<void(const JitFuture& jitFuture)>,
71:       std::hash<int>>
72:       errorHandlers = {
73:           {RPCErrorType::TIMEOUT,
74:            [this](const JitFuture& /* unused */) { setTimedOut(); }},
75:           {RPCErrorType::INTENTIONAL_FAILURE,
76:            [this](const JitFuture& /* unused */) { setTimedOut(); }},
77:           {RPCErrorType::UNKNOWN_ERROR, [](const JitFuture& jitFuture) {
78:              // Default error handler
79:              RRefContext::handleException(jitFuture);
80:            }}};
```

- EN: Lines 65-80 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 65-80 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 81-96 / 第 81-96 行

```cpp
81:   errorHandlers.find(errorType)->second(jitFuture);
82: }
83: 
84: //////////////////////////  UserRRef  /////////////////////////////////////
85: 
86: UserRRef::UserRRef(
87:     worker_id_t ownerId,
88:     const RRefId& rrefId,
89:     const ForkId& forkId,
90:     TypePtr type)
91:     : RRef(ownerId, rrefId, std::move(type)),
92:       forkId_(forkId),
93:       confirmedByOwner_(false) {
94:   // Do nothing,
95:   // (1) If this UserRRef is a fork of an existing RRef, RRefContext will send
96:   //     a RREF_FORK_REQUEST message to the owner.
```

- EN: Lines 81-96 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 81-96 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 97-112 / 第 97-112 行

```cpp
97:   // (2) If this the creator UserRRef, ScriptRemoteCall or PythonRemoteCall will
98:   //     properly notify the owner.
99: }
100: 
101: void UserRRef::tryDel() {
102:   std::lock_guard<std::mutex> lockGuard(deletedOnOwnerMutex_);
103:   if (!deletedOnOwner_) {
104:     try {
105:       RRefContext::getInstance().delUser(ownerId_, rrefId_, forkId_);
106:       deletedOnOwner_ = true;
107:     } catch (const std::exception& ex) {
108:       LOG(ERROR) << "Error occurred when deleting" << *this << " : "
109:                  << ex.what();
110:     } catch (...) {
111:       LOG(ERROR) << "Error occurred when deleting" << *this << " : "
112:                  << "unknown error";
```

- EN: Lines 97-112 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 97-112 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 113-128 / 第 113-128 行

```cpp
113:     }
114:   }
115: }
116: 
117: UserRRef::~UserRRef() {
118:   tryDel();
119: }
120: 
121: void UserRRef::release_resources() {
122:   tryDel();
123: }
124: 
125: const ForkId& UserRRef::forkId() const {
126:   return forkId_;
127: }
128: 
```

- EN: Lines 113-128 introduces executable logic in routines such as `tryDel`; returns computed state or forwards results to the surrounding caller.
- CN: 第 113-128 行在 `tryDel` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 129-144 / 第 129-144 行

```cpp
129: IValue UserRRef::toHere(const float timeoutSeconds) const {
130:   TORCH_CHECK(
131:       !getTimedOut(),
132:       "RRef creation via rpc.remote() timed out, and it "
133:       "is possible that the RRef on the owner node does not exist.");
134:   // see Note [Best-Effort Check on Deleted UserRRefs]
135:   TORCH_CHECK(
136:       !deletedOnOwner_,
137:       *this,
138:       " has been deleted. Cannot call to_here() on it after deletion.");
139:   auto toHereKey = std::string("");
140:   if (torch::autograd::profiler::profilerEnabled()) {
141:     toHereKey = fmt::format(
142:         "to_here#({})->({})",
143:         RpcAgent::getCurrentRpcAgent()->getWorkerInfo().name_,
144:         RpcAgent::getCurrentRpcAgent()->getWorkerInfo(ownerId_).name_);
```

- EN: Lines 129-144 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 129-144 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 145-160 / 第 145-160 行

```cpp
145:   }
146:   RECORD_USER_SCOPE(toHereKey);
147:   TORCH_CHECK(
148:       !type_->is_module(),
149:       *this,
150:       " is an RRef to a ScriptModule. "
151:       "It can't be sent through RPC "
152:       "from owner, ",
153:       ownerWorkerInfo(),
154:       ", to user, ",
155:       RpcAgent::getCurrentRpcAgent()->getWorkerInfo(),
156:       ".");
157: 
158:   auto agent = RpcAgent::getCurrentRpcAgent();
159: 
160:   // ScriptRRefFetchCall message always carries autograd context id even if
```

- EN: Lines 145-160 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 145-160 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 161-176 / 第 161-176 行

```cpp
161:   // the message itself does not contain any tensor, because the response would
162:   // potentially contain tensors.
163:   c10::intrusive_ptr<Message> msgToSend;
164: 
165:   if (isPyObj()) {
166:     msgToSend = PythonRRefFetchCall(ownerId_, rrefId()).toMessage();
167:   } else {
168:     msgToSend = ScriptRRefFetchCall(ownerId_, rrefId()).toMessage();
169:   }
170: 
171:   // toHere is profiled as a blocking call, and does not execute operations on
172:   // the remote node. Hence, don't wrap it with a profiling message since we
173:   // don't need the profiler to be enabled remotely.
174:   auto jitFuture = autograd::sendMessageWithAutograd(
175:       *agent,
176:       agent->getWorkerInfo(ownerId_),
```

- EN: Lines 161-176 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 161-176 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 177-192 / 第 177-192 行

```cpp
177:       std::move(msgToSend),
178:       true /* forceGradRecording */,
179:       timeoutSeconds,
180:       true /* forceDisableProfiling */);
181: 
182:   // TODO: we should ideally be able to interrupt this blocking wait if we check
183:   // getTimedOut() and it is true
184:   // (https://github.com/pytorch/pytorch/issues/39411).
185:   jitFuture->waitAndThrow();
186:   auto messagePtr = jitFuture->constValue().toCustomClass<Message>();
187:   MessageType msgType = messagePtr->type();
188:   auto response = deserializeResponse(*messagePtr, msgType);
189:   TORCH_INTERNAL_ASSERT(
190:       msgType == MessageType::SCRIPT_RREF_FETCH_RET ||
191:           msgType == MessageType::PYTHON_RREF_FETCH_RET,
192:       "Message type should either be SCRIPT_RREF_FETCH_RET "
```

- EN: Lines 177-192 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 177-192 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 193-208 / 第 193-208 行

```cpp
193:       "or PYTHON_RREF_FETCH_RET");
194:   RpcCommandBase& rpc = *response;
195:   auto& rrefFetchRet = static_cast<RRefFetchRet&>(rpc);
196:   if (isPyObj()) {
197:     // wrap python serialized vector of ivalues into tuple, this
198:     // made the C++ toHere interface to return single IValue
199:     return ivalue::Tuple::create(rrefFetchRet.values());
200:   } else {
201:     return rrefFetchRet.values().front();
202:   }
203: }
204: 
205: RRefForkData UserRRef::fork() const {
206:   // Note [Best-Effort Check on Deleted UserRRefs]
207:   // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
208:   // This check does not guarantee correctness, as there could be another thread
```

- EN: Lines 193-208 adds callable APIs, operators, or helper routines for this part of the file; returns computed state or forwards results to the surrounding caller.
- CN: 第 193-208 行为本段代码补充可调用 API、操作符或辅助例程；返回计算结果，或将结果继续传递给外围调用方。

### Lines 209-224 / 第 209-224 行

```cpp
209:   // trying to delete this UserRRef concurrently. Passing this check does not
210:   // mean this RRef will be alive throughout this function. This is just our
211:   // best-effort attempt to raise proper error messages. The behavior of using
212:   // deleted UserRRefs is undefined.
213:   //
214:   // The reason for not implementing strict checks are:
215:   // 1. This would need to acquire lock on deletedOnOwnerMutex_, which would
216:   //    introduce unnecessary overhead for most normal use cases.
217:   // 2. This would introduce a lot of complexities to get the behavior correct.
218:   //    Assume we acquired the lock here, and there is another thread X block
219:   //    waiting in tryDel() on the lock. Exiting this fork function would
220:   //    unblock thread X. However, while X proceeds with deleting this UserRRef,
221:   //    the call site of fork() might have added the UserRRef to
222:   //    pendingChildren_ map, but up to this point, nothing prevents X from
223:   //    deleting this RRef even if it shouldn't do so due to the state change
224:   //    in pendingChildren_. We might be able to get it right for now by locking
```

- EN: Lines 209-224 continues the local implementation details and data flow for this file.
- CN: 第 209-224 行继续展开本文件的局部实现细节与数据流。

### Lines 225-240 / 第 225-240 行

```cpp
225:   //    and checking pendingChildren_ in X, but the gain does not seem to
226:   //    worth the complexity.
227:   TORCH_CHECK(
228:       !deletedOnOwner_,
229:       *this,
230:       " has been deleted. Cannot call fork an UserRRef after deletion.");
231:   return RRef::fork();
232: }
233: 
234: //////////////////////////  OwnerRRef  /////////////////////////////////////
235: 
236: OwnerRRef::OwnerRRef(
237:     worker_id_t ownerId,
238:     const RRefId& rrefId,
239:     TypePtr type,
240:     std::vector<c10::Device> devices)
```

- EN: Lines 225-240 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 225-240 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 241-256 / 第 241-256 行

```cpp
241:     : OwnerRRef(
242:           ownerId,
243:           rrefId,
244:           std::move(type),
245:           /* value */ {},
246:           std::move(devices)) {}
247: 
248: OwnerRRef::OwnerRRef(
249:     worker_id_t ownerId,
250:     const RRefId& rrefId,
251:     TypePtr type,
252:     std::optional<IValue> value,
253:     std::vector<c10::Device> devices)
254:     : RRef(ownerId, rrefId, std::move(type)) {
255:   future_ = c10::make_intrusive<JitFuture>(type_, std::move(devices));
256: 
```

- EN: Lines 241-256 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 241-256 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 257-272 / 第 257-272 行

```cpp
257:   if (value.has_value()) {
258:     future_->markCompleted(value.value());
259:   }
260: }
261: 
262: const IValue& OwnerRRef::getValue() const {
263:   TORCH_CHECK(
264:       !getTimedOut(),
265:       "RRef creation via rpc.remote() timed out, and it "
266:       "is possible that the RRef on the owner node does not exist.");
267:   future_->waitAndThrow();
268:   return future_->constValue();
269: }
270: 
271: bool OwnerRRef::hasValue() const {
272:   return future_->completed();
```

- EN: Lines 257-272 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 257-272 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 273-288 / 第 273-288 行

```cpp
273: }
274: 
275: c10::intrusive_ptr<JitFuture> OwnerRRef::getFuture() {
276:   return future_;
277: }
278: 
279: void OwnerRRef::setValue(IValue&& value) {
280:   future_->markCompleted(std::move(value));
281: }
282: 
283: void OwnerRRef::setError(std::exception_ptr eptr) {
284:   future_->setErrorIfNeeded(std::move(eptr));
285: }
286: 
287: std::ostream& operator<<(std::ostream& os, const RRef& rref) {
288:   if (rref.isOwner()) {
```

- EN: Lines 273-288 introduces executable logic in routines such as `operator<<`; returns computed state or forwards results to the surrounding caller.
- CN: 第 273-288 行在 `operator<<` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 289-299 / 第 289-299 行

```cpp
289:     return os << "OwnerRRef("
290:               << "rref_id=" << rref.rrefId() << ')';
291:   } else {
292:     return os << "UserRRef("
293:               << "rref_id=" << rref.rrefId()
294:               << ", fork_id=" << static_cast<const UserRRef*>(&rref)->forkId()
295:               << ')';
296:   }
297: }
298: 
299: } // namespace torch::distributed::rpc
```

- EN: Lines 289-299 opens or closes namespaces to place the code in the correct distributed component; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 289-299 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；为本段代码补充可调用 API、操作符或辅助例程。

## Key Concepts / 关键概念

- EN: Subsystem: distributed RPC layer.
- CN: 子系统：分布式 RPC 层。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `getTypeStr`, `tryDel`, `TORCH_INTERNAL_ASSERT`, `TORCH_CHECK`, `operator<<`
- CN: 核心符号：`getTypeStr`、`tryDel`、`TORCH_INTERNAL_ASSERT`、`TORCH_CHECK`、`operator<<`
- EN: Notable themes: RPC/RRef semantics.
- CN: 值得关注的主题：RPC/RRef 语义。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/rpc/rref_impl.h`, `torch/csrc/distributed/autograd/utils.h`, `torch/csrc/distributed/rpc/rref_context.h`, `torch/csrc/distributed/rpc/rref_proto.h`, `torch/csrc/distributed/rpc/utils.h`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `ATen/record_function.h`
- External or system headers / 外部或系统头文件: `fmt/format.h`, `utility`
- Local symbols / 本地符号: `getTypeStr`, `tryDel`, `TORCH_INTERNAL_ASSERT`, `TORCH_CHECK`, `operator<<`