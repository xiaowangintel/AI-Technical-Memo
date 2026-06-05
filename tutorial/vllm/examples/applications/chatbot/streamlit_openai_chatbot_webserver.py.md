# streamlit_openai_chatbot_webserver.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/applications/chatbot/streamlit_openai_chatbot_webserver.py`
- **Repository**: vllm-project/vllm
- **Purpose**: vLLM Chat Assistant - A Streamlit Web Interface A streamlined chat interface that quickly integrates with vLLM API server / 演示围绕 vLLM 服务的聊天机器人客户端与 Web 界面集成。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
vLLM Chat Assistant - A Streamlit Web Interface

A streamlined chat interface that quickly integrates
with vLLM API server.

Features:
- Multiple chat sessions management
- Streaming response display
- Configurable API endpoint
- Real-time chat history
- Reasoning Display: Optional thinking process visualization 

Requirements:
    pip install streamlit openai

Usage:
    # Start the app with default settings
    streamlit run streamlit_openai_chatbot_webserver.py

    # Start with custom vLLM API endpoint
    VLLM_API_BASE="http://your-server:8000/v1" \
        streamlit run streamlit_openai_chatbot_webserver.py

    # Enable debug mode
    streamlit run streamlit_openai_chatbot_webserver.py \
        --logger.level=debug
"""
```
**EN:** vLLM Chat Assistant - A Streamlit Web Interface A streamlined chat interface that quickly integrates with vLLM API server.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import os
from datetime import datetime

import streamlit as st
from openai import OpenAI
```
**EN:** This block loads helper libraries such as os, datetime, streamlit, and openai.
**CN:** 这一部分加载 os、datetime、streamlit，以及 openai 等辅助库。

### Top-level setup
```python
openai_api_key = os.getenv("VLLM_API_KEY", "EMPTY")
openai_api_base = os.getenv("VLLM_API_BASE", "http://localhost:8000/v1")

# Initialize session states for managing chat sessions
if "sessions" not in st.session_state:
    st.session_state.sessions = {}

if "current_session" not in st.session_state:
    st.session_state.current_session = None

if "messages" not in st.session_state:
    st.session_state.messages = []

if "active_session" not in st.session_state:
    st.session_state.active_session = None

# Add new session state for reasoning
if "show_reasoning" not in st.session_state:
    st.session_state.show_reasoning = {}

# Initialize session state for API base URL
if "api_base_url" not in st.session_state:
    st.session_state.api_base_url = openai_api_base
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as openai_api_key and openai_api_base. It also performs early helper calls such as os.getenv.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 openai_api_key 和 openai_api_base 等变量。它还会提前执行 os.getenv 等辅助调用。

### Function: create_new_chat_session
```python
def create_new_chat_session():
    """Create a new chat session with timestamp as unique identifier.

    This function initializes a new chat session by:
    1. Generating a timestamp-based session ID
    2. Creating an empty message list for the new session
    3. Setting the new session as both current and active session
    4. Resetting the messages list for the new session

    Returns:
        None

    Session State Updates:
        - sessions: Adds new empty message list with timestamp key
        - current_session: Sets to new session ID
        - active_session: Sets to new session ID
        - messages: Resets to empty list
    """
    session_id = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    st.session_state.sessions[session_id] = []
    st.session_state.current_session = session_id
    st.session_state.active_session = session_id
    st.session_state.messages = []
```
**EN:** Create a new chat session with timestamp as unique identifier.. Key operations include strftime and datetime.now.
**CN:** 该函数构建核心运行时组件。关键操作包括 strftime 和 datetime.now。

### Function: switch_to_chat_session
```python
def switch_to_chat_session(session_id):
    """Switch the active chat context to a different session.

    Args:
        session_id (str): The timestamp ID of the session to switch to

    This function handles chat session switching by:
    1. Setting the specified session as current
    2. Updating the active session marker
    3. Loading the messages history from the specified session

    Session State Updates:
        - current_session: Updated to specified session_id
        - active_session: Updated to specified session_id
        - messages: Loaded from sessions[session_id]
    """
    st.session_state.current_session = session_id
    st.session_state.active_session = session_id
    st.session_state.messages = st.session_state.sessions[session_id]
```
**EN:** Switch the active chat context to a different session.. It works with parameters such as session_id.
**CN:** 该函数封装示例中的可复用步骤。它会处理 session_id 等参数。

### Function: get_llm_response
```python
def get_llm_response(messages, model, reason, content_ph=None, reasoning_ph=None):
    """Generate and stream LLM response with optional reasoning process.

    Args:
        messages (list): List of conversation message dicts with 'role' and 'content'
        model (str): The model identifier to use for generation
        reason (bool): Whether to enable and display reasoning process
        content_ph (streamlit.empty): Placeholder for streaming response content
        reasoning_ph (streamlit.empty): Placeholder for streaming reasoning process

    Returns:
        tuple: (str, str)
            - First string contains the complete response text
            - Second string contains the complete reasoning text (if enabled)

    Features:
        - Streams both reasoning and response text in real-time
        - Handles model API errors gracefully
        - Supports live updating of thinking process
        - Maintains separate content and reasoning displays
    # ... key logic omitted for brevity ...
            if hasattr(delta, "content") and delta.content and content_ph:
                full_text += delta.content
                content_ph.markdown(full_text + "▌")

        # Finalize displays: reasoning remains above, content below
        if reason and live_think:
            live_think.markdown(think_text)
        if content_ph:
            content_ph.markdown(full_text)

        return full_text, think_text
    except Exception as e:
        st.error(f"Error details: {str(e)}")
        return f"Error: {str(e)}", ""
```
**EN:** Generate and stream LLM response with optional reasoning process.. It works with parameters such as messages, model, reason, content_ph, and reasoning_ph. Key operations include content_ph.markdown, live_think.markdown, hasattr, str, and client.chat.completions.create. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。它会处理 messages、model、reason、content_ph，以及 reasoning_ph 等参数。关键操作包括 content_ph.markdown、live_think.markdown、hasattr、str，以及 client.chat.completions.create。其返回值会继续传给示例管线的下一阶段。

### Top-level setup
```python
st.sidebar.title("API Settings")
new_api_base = st.sidebar.text_input(
    "API Base URL:", value=st.session_state.api_base_url
)
if new_api_base != st.session_state.api_base_url:
    st.session_state.api_base_url = new_api_base
    st.rerun()

st.sidebar.divider()

# Sidebar - Session Management
st.sidebar.title("Chat Sessions")
# ... key logic omitted for brevity ...
    # Render assistant messages with reasoning above
    else:
        # If reasoning exists for this assistant message, show it above the content
        if idx in st.session_state.show_reasoning:
            with st.expander("💭 Thinking Process", expanded=False):
                st.markdown(st.session_state.show_reasoning[idx])
        with st.chat_message("assistant"):
            st.write(msg["content"])
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as new_api_base, client, models, and model. It also performs early helper calls such as st.sidebar.title, st.sidebar.text_input, st.rerun, st.sidebar.divider, and st.sidebar.button.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 new_api_base、client、models，以及 model 等变量。它还会提前执行 st.sidebar.title、st.sidebar.text_input、st.rerun、st.sidebar.divider，以及 st.sidebar.button 等辅助调用。

### Function: server_supports_reasoning
```python
def server_supports_reasoning():
    """Check if the current model supports reasoning capability.

    Returns:
        bool: True if the model supports reasoning, False otherwise
    """
    resp = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": "Hi"}],
        stream=False,
    )
    return hasattr(resp.choices[0].message, "reasoning") and bool(
        resp.choices[0].message.reasoning
    )
```
**EN:** Check if the current model supports reasoning capability.. Key operations include st.cache_data, client.chat.completions.create, hasattr, and bool. The return value feeds the next stage of the example pipeline.
**CN:** 该函数处理推理相关内容或元数据。关键操作包括 st.cache_data、client.chat.completions.create、hasattr，以及 bool。其返回值会继续传给示例管线的下一阶段。

### Top-level setup
```python
supports_reasoning = server_supports_reasoning()

# Add reasoning toggle in sidebar if supported
reason = False  # Default to False
if supports_reasoning:
    reason = st.sidebar.checkbox("Enable Reasoning", value=False)
else:
    st.sidebar.markdown(
        "<span style='color:gray;'>Reasoning unavailable for this model.</span>",
        unsafe_allow_html=True,
    )
    # reason remains False
# ... key logic omitted for brevity ...
        full, think = get_llm_response(msgs, model, reason, content_ph, reason_ph)
        # Determine index for this new assistant message
        message_index = len(st.session_state.messages)
        # Save assistant reply
        st.session_state.messages.append({"role": "assistant", "content": full})
        # Persist reasoning in session state if any
        if reason and think:
            st.session_state.show_reasoning[message_index] = think
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as supports_reasoning and reason. It also performs early helper calls such as server_supports_reasoning, st.sidebar.checkbox, st.sidebar.markdown, st.session_state.messages.append, and st.chat_message.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 supports_reasoning 和 reason 等变量。它还会提前执行 server_supports_reasoning、st.sidebar.checkbox、st.sidebar.markdown、st.session_state.messages.append，以及 st.chat_message 等辅助调用。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Interactive application wrappers / 交互式应用封装**: UI or service layers turn model calls into an interactive user experience. / 界面或服务层把模型调用包装成交互式用户体验。
- **Reasoning traces / 推理轨迹**: The example exposes or consumes explicit reasoning content alongside final answers. / 该示例会在最终答案之外展示或消费显式推理内容。

## Dependencies / 依赖关系
- **External libraries / 外部库**: `os`, `datetime`, `streamlit`, `openai` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `create_new_chat_session`, `switch_to_chat_session`, `get_llm_response`, `server_supports_reasoning` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `os.getenv`, `strftime`, `datetime.now`, `content_ph.markdown`, `live_think.markdown`, `hasattr`, `str`, `client.chat.completions.create` reveal the main execution path / 这些调用体现了主要执行链路。
