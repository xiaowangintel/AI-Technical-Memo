# web_demo.py 代码分析

## 1. 文件概述 (File Overview)
`web_demo.py` 是 MiniMind 的 **Streamlit Web UI** 演示脚本。它负责把本地模型包装成可交互网页，支持模型选择、语言切换、thinking 展示、工具选择、多轮对话状态管理与流式输出。在部署流程里，它不是标准 API 服务，而是面向人工体验与 Demo 展示的前端交互层。

## 2. 依赖说明 (Dependencies)
- `random`, `re`, `json`, `os`：随机数、正则解析、JSON 处理、目录扫描。
- `Thread`：把生成放到后台线程。
- `torch`, `numpy`：推理与随机种子同步。
- `streamlit as st`：Web UI 框架。
- `AutoModelForCausalLM`, `AutoTokenizer`, `TextIteratorStreamer`：模型加载、tokenizer、流式迭代输出。

## 3. 逐行代码分析 (Line-by-Line Analysis)
- **第 1-10 行**：导入标准库、线程、Torch、NumPy、Streamlit 与 Transformers 组件。
- **第 12 行**：设置 Streamlit 页面标题与侧边栏默认状态。
- **第 14-68 行**：通过 `st.markdown(..., unsafe_allow_html=True)` 注入 CSS，重写按钮形状、尺寸、悬停效果与页面边距，属于前端样式层。
- **第 70 行**：自动选择 `cuda` 或 `cpu`。
- **第 72-100 行**：定义双语文本字典 `LANG_TEXTS`，让 UI 可以在中文/英文之间切换。
- **第 102-104 行**：`get_text(key)` 根据 `st.session_state.lang` 返回当前语言对应文案。
- **第 106-116 行**：定义 `TOOLS`，采用 function-calling 风格 JSON Schema，用于把工具信息注入模型 prompt。
- **第 118-122 行**：定义 `TOOL_SHORT_NAMES`，把工具长名称映射成更适合 UI 展示的短标签。
- **第 124-146 行**：`execute_tool(tool_name, args)` 实现本地 mock 工具：计算、时间、随机数、字数、单位换算、天气、汇率、翻译等；若异常则返回错误对象。
- **第 149-195 行**：`process_assistant_content(content, is_streaming=False)` 是显示层核心：
  - **第 151-160 行**：把 `<tool_call>...</tool_call>` 转成带背景色的 HTML 卡片；
  - **第 162-172 行**：对于流式且 thinking 开启但 `<think>` 标签未完整出现的情况，尝试提前把前半段内容包进“思考中...”折叠框；
  - **第 173-180 行**：当存在完整 `<think>...</think>` 时，转成“已思考”折叠块；
  - **第 181-185 行**：当只有起始 `<think>`、尚未结束时，显示为“思考中...”状态；
  - **第 187-193 行**：当只有 `</think>` 而无起始标签时，也尽量容错格式化；
  - **第 195 行**：返回格式化后的 HTML 字符串。
- **第 198-209 行**：`load_model_tokenizer(model_path)` 使用 `@st.cache_resource` 缓存模型与 tokenizer，避免每次页面刷新都重复加载。
- **第 212-214 行**：`clear_chat_messages()` 删除会话状态中的消息缓存。
- **第 217-231 行**：`init_chat_messages()`：如果已有历史，则按角色重新渲染；否则初始化 `messages` 与 `chat_messages` 两个列表。
- **第 233-236 行**：`regenerate_answer(index)` 删除最后一轮消息并触发 `st.rerun()`，用于“重生成”。
- **第 239-249 行**：动态扫描脚本所在目录下的子目录，把含模型文件的目录识别为可选模型；若没有找到，则提供占位项 `No models found`。
- **第 251-253 行**：侧边栏模型选择，并根据当前语言生成欢迎语 `slogan`。
- **第 255-266 行**：在侧边栏中切换语言；若语言改变，则写入 `st.session_state.lang` 并 `st.rerun()`。
- **第 268-271 行**：侧边栏参数滑块：历史轮数、最大生成长度、温度。
- **第 273-287 行**：功能开关区：
  - `enable_thinking` 控制思考模式；
  - 工具选择区最多允许选 4 个工具；
  - 通过 `st.session_state.selected_tools` 记录当前勾选结果。
- **第 288-299 行**：渲染顶部 logo、欢迎语与免责声明。
- **第 302-309 行**：`setup_seed(seed)` 同时设置 Python、NumPy、Torch 与 CUDA 的随机种子，并固定 cuDNN 行为。

### 3.1 主推理流程 `main()`
- **第 312-313 行**：加载当前选中模型与 tokenizer。
- **第 315-317 行**：若会话状态中还没有消息列表，则初始化。
- **第 319-327 行**：重新渲染已有消息；assistant 消息走 `process_assistant_content()`，user 消息用自定义右对齐气泡显示。
- **第 329 行**：显示 `st.chat_input()` 输入框。
- **第 331-336 行**：若用户触发了 regenerate，则把上次 user 消息重新塞回 `prompt`，并清理相关状态。
- **第 338-343 行**：如果本轮有新输入：
  - 先把用户消息渲染出来；
  - 再把裁剪后的 prompt 同时写入 `messages` 与 `chat_messages`。
- **第 345 行**：创建 `placeholder`，后续用于动态刷新 assistant 输出。
- **第 347-348 行**：随机生成 seed 并调用 `setup_seed()`。
- **第 350 行**：根据 UI 勾选结果过滤出当前可用工具；若为空则传 `None`。
- **第 351 行**：没有工具时自动加一条 `system prompt`，让回答更稳；有工具时不额外注入 system prompt。
- **第 352 行**：把 `chat_messages` 截断为最近若干轮历史。
- **第 353-358 行**：构造 `template_kwargs`，按需添加 `open_thinking` 与 `tools`，然后调用 `tokenizer.apply_chat_template(...)` 生成 prompt。
- **第 360 行**：tokenization 并转到 device。
- **第 362-374 行**：创建 `TextIteratorStreamer` 与 `generation_kwargs`，设置 `max_length`、`do_sample=True`、`temperature`、`top_p=0.85`、`pad_token_id`、`eos_token_id` 等参数。
- **第 376 行**：在后台线程中启动 `model.generate()`。
- **第 378-381 行**：持续迭代 `streamer`，把新文本拼接到 `answer`，并实时刷新 `placeholder`；显示时会再次经过 `process_assistant_content(..., is_streaming=True)`。

### 3.2 Tool calling 多轮循环
- **第 383-387 行**：把当前回答保存为 `full_answer`，并最多循环 16 次检测是否存在 `<tool_call>` 标签；没有就结束。
- **第 388 行**：把当前 assistant 输出加入 `chat_messages`，让下一轮工具结果有上下文依赖。
- **第 389-397 行**：逐个解析工具调用 JSON，执行 `execute_tool()`，并把工具结果作为 `role=tool` 消息写回历史；同时生成一段 `ToolCalled` HTML 卡片用于页面展示。
- **第 398-399 行**：把工具调用结果拼接进 `full_answer` 并刷新页面。
- **第 400-407 行**：基于“assistant 输出 + tool result”重新构造 prompt、tokenize、更新 `generation_kwargs`，再次启动 `model.generate()`。
- **第 408-412 行**：继续流式接收新的 assistant 文本并刷新页面。
- **第 412-413 行**：把这一轮补充回答拼接回 `full_answer`，最后作为完整答案。
- **第 415-416 行**：把最终 assistant 内容同步写入 `messages` 与 `chat_messages`。
- **第 419-420 行**：标准入口 `main()`。

## 4. 关键算法解析 (Key Algorithm Deep-Dive)
### 4.1 Thinking display
`process_assistant_content()` 并不是简单打印纯文本，而是把 `<think>` 片段变成可折叠的 HTML `details` 区块。这样用户既能看最终答案，也能按需展开 reasoning content。

### 4.2 Tool selection 与 template injection
UI 侧最多允许选择 4 个工具；选中的工具会通过 `tokenizer.apply_chat_template(..., tools=tools)` 注入 prompt。也就是说，页面上的 checkbox 最终会影响模型看到的工具 schema。

### 4.3 多轮 conversation state
脚本同时维护两套状态：
- `messages`：面向 UI 展示；
- `chat_messages`：面向模型推理。
这样可以把显示层格式化与模型输入历史分离，避免 HTML 或展示卡片污染真实 prompt。

### 4.4 Generation parameters
- `temperature`：由侧边栏滑块控制；
- `top_p=0.85`：固定 nucleus sampling；
- `top_k`：此脚本未显式设置；
- `repetition_penalty`：此脚本未显式设置；
- `max_length = input_length + max_new_tokens`：总长度上限按“输入长度 + 生成长度”计算。

### 4.5 Tool call parsing
Tool call 采用文本协议 `<tool_call>{...}</tool_call>`。Web UI 先用正则解析，再本地执行 mock tool，最后把 `role=tool` 结果回填给模型继续生成。这是一个完整的 multi-turn tool-augmented chat loop。

## 5. 调用关系 (Call Graph)
```text
main()
 ├─ load_model_tokenizer(model_path)
 │   ├─ AutoModelForCausalLM.from_pretrained()
 │   └─ AutoTokenizer.from_pretrained()
 ├─ 渲染 st.session_state.messages
 ├─ st.chat_input()
 ├─ tokenizer.apply_chat_template(chat_messages, tools=..., open_thinking=...)
 ├─ tokenizer(...)
 ├─ model.generate(..., streamer=TextIteratorStreamer)
 ├─ process_assistant_content()   [HTML 格式化 thinking/tool_call]
 ├─ execute_tool()                [若检测到 <tool_call>]
 └─ 把 tool result 回填到 chat_messages 后再次 generate()
```
从体系结构看，`web_demo.py` 是“本地模型 + tokenizer + Streamlit UI + mock tools”组合而成的前端演示壳层。
