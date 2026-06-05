# CUDASymmetricMemoryUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryUtils.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): This file provides utility helpers in the c10d symmetric-memory support. Key types include `sockaddr_un`, `iovec`, `msghdr`.
- 用途 (CN): 该文件在c10d 对称内存支持中提供工具辅助逻辑。 关键类型包括 `sockaddr_un`、`iovec`、`msghdr`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行

```cpp
1: #include <sys/socket.h>
2: #include <sys/un.h>
3: #include <unistd.h>
4: 
5: #include <c10/util/error.h>
6: 
7: #if !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
8: #include <c10/cuda/driver_api.h>
9: #elif defined(USE_ROCM)
10: #include <c10/hip/HIPException.h>
11: #include <hip/hip_runtime_api.h>
12: #endif
13: 
14: #include <torch/csrc/distributed/c10d/cuda/utils.hpp>
15: #include <torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryUtils.hpp>
16: 
```

- EN: Lines 1-16 pulls in the headers required by this translation unit or interface; uses conditional compilation to adapt to feature flags, platforms, or optional backends.
- CN: 第 1-16 行引入该实现单元或接口所需的头文件；使用条件编译来适配特性开关、平台或可选后端。

### Lines 17-32 / 第 17-32 行

```cpp
17: namespace c10d::symmetric_memory {
18: 
19: bool device_has_multicast_support(int device_idx) {
20:   if (c10::utils::check_env("TORCH_SYMM_MEM_DISABLE_MULTICAST") == true) {
21:     return false;
22:   }
23:   return c10d::cuda::deviceSupportsMulticast(device_idx);
24: }
25: 
26: bool allow_overlapping_devices() {
27:   return c10::utils::check_env("TORCH_SYMM_MEM_ALLOW_OVERLAPPING_DEVICES") ==
28:       true;
29: }
30: 
31: // Query environment variable to get the backend used for CUDA Symmetric Memory.
32: std::string getSymmMemBackendCUDA() {
```

- EN: Lines 17-32 opens or closes namespaces to place the code in the correct distributed component; introduces executable logic in routines such as `device_has_multicast_support`, `allow_overlapping_devices`, `getSymmMemBackendCUDA`.
- CN: 第 17-32 行打开或关闭命名空间，以便将代码放入正确的分布式组件中；在 `device_has_multicast_support`、`allow_overlapping_devices`、`getSymmMemBackendCUDA` 等例程中引入具体执行逻辑。

### Lines 33-48 / 第 33-48 行

```cpp
33:   // TORCH_SYMMMEM environment variable can be used to indicate the preferred
34:   // backend.
35:   static auto val = c10::utils::get_env("TORCH_SYMMMEM");
36:   if (val.has_value()) {
37:     TORCH_CHECK(
38:         val.value() == "CUDA" || val.value() == "NVSHMEM" ||
39:             val.value() == "NCCL",
40:         "TORCH_SYMMMEM environment variable must be one of 'CUDA', 'NVSHMEM', 'NCCL'.")
41:     return val.value();
42:   }
43:   // If TORCH_SYMMMEM is not set, check if NVSHMEM is available (for broader
44:   // support).
45:   // TODO: uncomment this once all single-node tests work with NVSHMEM
46:   // if (is_nvshmem_available()) {
47:   //   return "NVSHMEM";
48:   // }
```

- EN: Lines 33-48 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 33-48 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 49-64 / 第 49-64 行

```cpp
49:   return "CUDA";
50: }
51: 
52: IpcChannel::IpcChannel()
53:     : socket_name_(get_socket_name(getpid())),
54:       socket_(socket(AF_UNIX, SOCK_DGRAM, 0)) {
55:   // On success, a file descriptor for the new socket is returned.
56:   //  On error, -1 is returned, and errno is set to indicate the error.
57:   TORCH_CHECK(
58:       socket_ != -1, "Failed to create socket: ", c10::utils::str_error(errno));
59: 
60:   struct sockaddr_un addr = {.sun_family = AF_UNIX};
61:   std::copy(socket_name_.begin(), socket_name_.end(), addr.sun_path);
62: 
63:   TORCH_CHECK(
64:       bind(socket_, (struct sockaddr*)&addr, SUN_LEN(&addr)) == 0,
```

- EN: Lines 49-64 declares or defines types such as `sockaddr_un`; adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 49-64 行声明或定义了 `sockaddr_un` 等类型；为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 65-80 / 第 65-80 行

```cpp
65:       "Failed to bind socket: ",
66:       c10::utils::str_error(errno));
67: }
68: 
69: IpcChannel::~IpcChannel() {
70:   close(socket_);
71:   unlink(socket_name_.c_str());
72: }
73: 
74: void IpcChannel::send_fd(int dst_pid, int fd) {
75:   // Because file descriptors are process-local kernel objects, and we can’t
76:   // pass them via normal socket payloads (like write() or send()).  Unix domain
77:   // sockets provide a mechanism to pass actual FDs via sendmsg()/recvmsg().
78:   // Define destination socket address
79:   struct sockaddr_un addr = {.sun_family = AF_UNIX};
80:   auto socket_name = get_socket_name(dst_pid);
```

- EN: Lines 65-80 declares or defines types such as `sockaddr_un`; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 65-80 行声明或定义了 `sockaddr_un` 等类型；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 81-96 / 第 81-96 行

```cpp
81:   std::copy(socket_name.begin(), socket_name.end(), addr.sun_path);
82: 
83:   // Prepare data to send
84:   // Data being sent is "fd", the value of fd will be sent as auxiliary data
85:   // (control message)
86:   struct iovec io = {.iov_base = (void*)"fd", .iov_len = 2};
87: 
88:   // Prepare control message data buffer and zero it out
89:   // NOLINTNEXTLINE(*array*)
90:   char cbuf[CMSG_SPACE(sizeof(int))];
91:   memset(cbuf, 0, sizeof(cbuf));
92: 
93:   // Create message header
94:   struct msghdr msg{
95:       // destination socket address and size of it
96:       // message content in msg_iov and number of such structs (1 in our case)
```

- EN: Lines 81-96 declares or defines types such as `iovec`, `msghdr`; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 81-96 行声明或定义了 `iovec`、`msghdr` 等类型；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 97-112 / 第 97-112 行

```cpp
97:       // auxiliary data with the value of fd and size of it
98:       .msg_name = (void*)&addr,
99:       .msg_namelen = sizeof(struct sockaddr_un),
100:       .msg_iov = &io,
101:       .msg_iovlen = 1,
102:       .msg_control = cbuf,
103:       .msg_controllen = sizeof(cbuf)};
104: 
105:   // This points to the first control message header
106:   // With SCM_RIGHTS we let the kernel know that we are passing file
107:   // descriptors.
108:   auto cmsg = CMSG_FIRSTHDR(&msg);
109:   cmsg->cmsg_len = CMSG_LEN(sizeof(int));
110:   // Specify socket level message
111:   cmsg->cmsg_level = SOL_SOCKET;
112:   // SCM_RIGHTS is the type used to pass file descriptors
```

- EN: Lines 97-112 adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 97-112 行为本段代码补充可调用 API、操作符或辅助例程。

### Lines 113-128 / 第 113-128 行

```cpp
113:   cmsg->cmsg_type = SCM_RIGHTS;
114: 
115:   if (fd != -1) {
116:     std::copy(
117:         reinterpret_cast<const char*>(&fd),
118:         reinterpret_cast<const char*>(&fd) + sizeof(fd),
119:         reinterpret_cast<char*>(CMSG_DATA(cmsg)));
120:   } else {
121:     msg.msg_controllen = 0;
122:   }
123: 
124:   // Finally send the message
125:   TORCH_CHECK(
126:       sendmsg(socket_, &msg, 0) > 0,
127:       "Failed to send fd: ",
128:       c10::utils::str_error(errno));
```

- EN: Lines 113-128 adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 113-128 行为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 129-144 / 第 129-144 行

```cpp
129: }
130: 
131: int IpcChannel::recv_fd() {
132:   // Prepare buffer for regular message "fd"
133:   // NOLINTNEXTLINE(*array*)
134:   char buf[2];
135:   memset(&buf, 0, sizeof(buf));
136:   struct iovec io = {.iov_base = (void*)buf, .iov_len = sizeof(buf)};
137: 
138:   // Prepare buffer for control message and zero it out
139:   // NOLINTNEXTLINE(*array*)
140:   char cbuf[CMSG_SPACE(sizeof(int))];
141:   memset(cbuf, 0, sizeof(cbuf));
142: 
143:   // Define socket address to receive on: family AF_UNIX means unix domain
144:   // socket
```

- EN: Lines 129-144 declares or defines types such as `iovec`; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 129-144 行声明或定义了 `iovec` 等类型；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 145-160 / 第 145-160 行

```cpp
145:   struct sockaddr_un addr = {.sun_family = AF_UNIX};
146:   std::copy(socket_name_.begin(), socket_name_.end(), addr.sun_path);
147: 
148:   // Prepare message header
149:   struct msghdr msg = {
150:       .msg_name = (void*)&addr,
151:       .msg_namelen = sizeof(struct sockaddr_un),
152:       .msg_iov = &io,
153:       .msg_iovlen = 1,
154:       .msg_control = cbuf,
155:       .msg_controllen = sizeof(cbuf)};
156: 
157:   // Receive message on socket_
158:   TORCH_CHECK(
159:       recvmsg(socket_, &msg, 0) > 0,
160:       "Failed to receive fd: ",
```

- EN: Lines 145-160 declares or defines types such as `sockaddr_un`, `msghdr`; adds callable APIs, operators, or helper routines for this part of the file; performs validation and error handling to keep distributed state consistent.
- CN: 第 145-160 行声明或定义了 `sockaddr_un`、`msghdr` 等类型；为本段代码补充可调用 API、操作符或辅助例程；执行校验与错误处理，以保持分布式状态一致。

### Lines 161-176 / 第 161-176 行

```cpp
161:       c10::utils::str_error(errno));
162: 
163:   if (msg.msg_controllen == 0) {
164:     return -1;
165:   }
166: 
167:   // Extract control message and validate its content
168:   auto cmsg = CMSG_FIRSTHDR(&msg);
169:   TORCH_CHECK(cmsg != nullptr);
170:   TORCH_CHECK(cmsg->cmsg_len == CMSG_LEN(sizeof(int)));
171:   TORCH_CHECK(cmsg->cmsg_level == SOL_SOCKET && cmsg->cmsg_type == SCM_RIGHTS);
172:   return *reinterpret_cast<int*>(CMSG_DATA(cmsg));
173: }
174: 
175: std::vector<int> IpcChannel::all_gather_fds(
176:     int rank,
```

- EN: Lines 161-176 introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 161-176 行在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 177-192 / 第 177-192 行

```cpp
177:     const std::vector<int>& pids,
178:     int fd) {
179:   int world_size = static_cast<int>(pids.size());
180:   std::vector<int> fds(pids.size());
181:   fds[rank] = fd;
182: 
183:   int dst_rank = (rank + 1) % world_size;
184:   for (int step = 1; step < world_size; ++step) {
185:     int src_rank = (rank + world_size - step) % world_size;
186:     send_fd(pids[dst_rank], fd);
187:     fd = recv_fd();
188:     fds[src_rank] = fd;
189:   }
190:   return fds;
191: }
192: 
```

- EN: Lines 177-192 introduces executable logic in routines such as `send_fd`; returns computed state or forwards results to the surrounding caller.
- CN: 第 177-192 行在 `send_fd` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 193-208 / 第 193-208 行

```cpp
193: int IpcChannel::broadcast_fds(
194:     int rank,
195:     int src_rank,
196:     const std::vector<int>& pids,
197:     int fd) {
198:   int world_size = static_cast<int>(pids.size());
199: 
200:   if (rank == src_rank) {
201:     for (int dst_rank = 0; dst_rank < world_size; ++dst_rank) {
202:       if (dst_rank == rank) {
203:         continue;
204:       }
205:       send_fd(pids[dst_rank], fd);
206:     }
207:     return fd;
208:   }
```

- EN: Lines 193-208 introduces executable logic in routines such as `send_fd`; returns computed state or forwards results to the surrounding caller.
- CN: 第 193-208 行在 `send_fd` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 209-224 / 第 209-224 行

```cpp
209:   return recv_fd();
210: }
211: 
212: std::string IpcChannel::get_socket_name(int pid) {
213:   const char* tmp_dir = "/tmp";
214:   for (const char* env_var : {"TMPDIR", "TMP", "TEMP", "TEMPDIR"}) {
215:     if (const char* path = getenv(env_var)) {
216:       tmp_dir = path;
217:       break;
218:     }
219:   }
220:   std::ostringstream oss;
221:   oss << tmp_dir << "/symm_mem-" << pid;
222:   return oss.str();
223: }
224: 
```

- EN: Lines 209-224 introduces executable logic in routines such as `recv_fd`; returns computed state or forwards results to the surrounding caller.
- CN: 第 209-224 行在 `recv_fd` 等例程中引入具体执行逻辑；返回计算结果，或将结果继续传递给外围调用方。

### Lines 225-240 / 第 225-240 行

```cpp
225: void map_block(
226:     void** ptr,
227:     c10d::symmetric_memory::HandleType handle,
228:     size_t size,
229:     int device_idx) {
230: #if !defined(USE_ROCM) && defined(PYTORCH_C10_DRIVER_API_SUPPORTED)
231:   auto driver_api = c10::cuda::DriverAPI::get();
232:   auto dev_ptr = reinterpret_cast<CUdeviceptr*>(ptr);
233:   // Allocate virtual address space
234:   C10_CUDA_DRIVER_CHECK(
235:       driver_api->cuMemAddressReserve_(dev_ptr, size, 0ULL, 0, 0ULL));
236:   // Map the physical memory to the virtual address
237:   C10_CUDA_DRIVER_CHECK(driver_api->cuMemMap_(*dev_ptr, size, 0, handle, 0ULL));
238: 
239:   // Set access permissions
240:   CUmemAccessDesc desc;
```

- EN: Lines 225-240 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `map_block`.
- CN: 第 225-240 行使用条件编译来适配特性开关、平台或可选后端；在 `map_block` 等例程中引入具体执行逻辑。

### Lines 241-256 / 第 241-256 行

```cpp
241:   desc.location.type = CU_MEM_LOCATION_TYPE_DEVICE;
242:   // NOLINTNEXTLINE(bugprone-signed-char-misuse)
243:   desc.location.id = device_idx;
244:   desc.flags = CU_MEM_ACCESS_FLAGS_PROT_READWRITE;
245:   C10_CUDA_DRIVER_CHECK(driver_api->cuMemSetAccess_(*dev_ptr, size, &desc, 1));
246: #elif defined(USE_ROCM)
247:   C10_CUDA_CHECK(hipMemAddressReserve(ptr, size, 0ULL, 0, 0ULL));
248:   C10_CUDA_CHECK(hipMemMap(
249:       *ptr,
250:       size,
251:       0,
252:       reinterpret_cast<hipMemGenericAllocationHandle_t>(handle),
253:       0ULL));
254:   C10_CUDA_CHECK(hipMemMap(
255:       *ptr,
256:       size,
```

- EN: Lines 241-256 uses conditional compilation to adapt to feature flags, platforms, or optional backends; adds callable APIs, operators, or helper routines for this part of the file.
- CN: 第 241-256 行使用条件编译来适配特性开关、平台或可选后端；为本段代码补充可调用 API、操作符或辅助例程。

### Lines 257-272 / 第 257-272 行

```cpp
257:       0,
258:       reinterpret_cast<hipMemGenericAllocationHandle_t>(handle),
259:       0ULL));
260: 
261:   hipMemAccessDesc desc;
262:   desc.location.type = hipMemLocationTypeDevice;
263:   // NOLINTNEXTLINE(bugprone-signed-char-misuse)
264:   desc.location.id = static_cast<int>(device_idx);
265:   desc.flags = hipMemAccessFlagsProtReadWrite;
266:   C10_CUDA_CHECK(hipMemSetAccess(*ptr, size, &desc, 1));
267: #else
268:   TORCH_CHECK(
269:       false, "CUDASymmetricMemory requires PYTORCH_C10_DRIVER_API_SUPPORTED");
270: #endif
271: }
272: 
```

- EN: Lines 257-272 uses conditional compilation to adapt to feature flags, platforms, or optional backends; introduces executable logic in routines such as `TORCH_CHECK`; performs validation and error handling to keep distributed state consistent.
- CN: 第 257-272 行使用条件编译来适配特性开关、平台或可选后端；在 `TORCH_CHECK` 等例程中引入具体执行逻辑；执行校验与错误处理，以保持分布式状态一致。

### Lines 273-273 / 第 273-273 行

```cpp
273: } // namespace c10d::symmetric_memory
```

- EN: Lines 273-273 opens or closes namespaces to place the code in the correct distributed component.
- CN: 第 273-273 行打开或关闭命名空间，以便将代码放入正确的分布式组件中。

## Key Concepts / 关键概念

- EN: Subsystem: c10d symmetric-memory support.
- CN: 子系统：c10d 对称内存支持。
- EN: File role: implementation/registration.
- CN: 文件角色：实现/注册。
- EN: Primary symbols: `sockaddr_un`, `iovec`, `msghdr`
- CN: 核心符号：`sockaddr_un`、`iovec`、`msghdr`
- EN: Notable themes: distributed infrastructure.
- CN: 值得关注的主题：分布式基础设施。

## Dependencies / 依赖关系

- Internal distributed headers / 内部分布式头文件: `torch/csrc/distributed/c10d/cuda/utils.hpp`, `torch/csrc/distributed/c10d/symm_mem/CUDASymmetricMemoryUtils.hpp`
- PyTorch/ATen/C10 headers / PyTorch/ATen/C10 头文件: `c10/util/error.h`, `c10/cuda/driver_api.h`, `c10/hip/HIPException.h`
- External or system headers / 外部或系统头文件: `sys/socket.h`, `sys/un.h`, `unistd.h`, `hip/hip_runtime_api.h`
- Local symbols / 本地符号: `sockaddr_un`, `iovec`, `msghdr`