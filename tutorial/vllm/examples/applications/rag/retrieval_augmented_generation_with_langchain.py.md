# retrieval_augmented_generation_with_langchain.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/applications/rag/retrieval_augmented_generation_with_langchain.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Retrieval Augmented Generation (RAG) Implementation with Langchain ================================================================== This script demonstrates a RAG implementation using LangChain, Milvus and vLLM / 演示基于 vLLM 的检索增强生成流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Retrieval Augmented Generation (RAG) Implementation with Langchain
==================================================================

This script demonstrates a RAG implementation using LangChain, Milvus
and vLLM. RAG enhances LLM responses by retrieving relevant context
from a document collection.

Features:
- Web content loading and chunking
- Vector storage with Milvus
- Embedding generation with vLLM
# ... key logic omitted for brevity ...
Usage:
    python retrieval_augmented_generation_with_langchain.py

Notes:
    - Ensure both vLLM services are running before executing
    - Default ports: 8000 (embedding), 8001 (chat)
    - First run may take time to download models
"""
```
**EN:** Retrieval Augmented Generation (RAG) Implementation with Langchain ================================================================== This script demonstrates a RAG implementation using LangChain, Milvus and vLLM.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
from argparse import Namespace
from typing import Any

from langchain_community.document_loaders import WebBaseLoader
from langchain_core.documents import Document
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_milvus import Milvus
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
```
**EN:** This block loads helper libraries such as argparse, typing, langchain_community.document_loaders, langchain_core.documents, and langchain_core.output_parsers.
**CN:** 这一部分加载 argparse、typing、langchain_community.document_loaders、langchain_core.documents，以及 langchain_core.output_parsers 等辅助库。

### Function: load_and_split_documents
```python
def load_and_split_documents(config: dict[str, Any]):
    """
    Load and split documents from web URL
    """
    try:
        loader = WebBaseLoader(web_paths=(config["url"],))
        docs = loader.load()

        text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=config["chunk_size"],
            chunk_overlap=config["chunk_overlap"],
        )
        return text_splitter.split_documents(docs)
    except Exception as e:
        print(f"Error loading document from {config['url']}: {str(e)}")
        raise
```
**EN:** Load and split documents from web URL. It works with parameters such as config. Key operations include WebBaseLoader, loader.load, RecursiveCharacterTextSplitter, text_splitter.split_documents, and print. The return value feeds the next stage of the example pipeline.
**CN:** 该函数加载输入数据并为后续阶段做准备。它会处理 config 等参数。关键操作包括 WebBaseLoader、loader.load、RecursiveCharacterTextSplitter、text_splitter.split_documents，以及 print。其返回值会继续传给示例管线的下一阶段。

### Function: init_vectorstore
```python
def init_vectorstore(config: dict[str, Any], documents: list[Document]):
    """
    Initialize vector store with documents
    """
    return Milvus.from_documents(
        documents=documents,
        embedding=OpenAIEmbeddings(
            model=config["embedding_model"],
            openai_api_key=config["vllm_api_key"],
            openai_api_base=config["vllm_embedding_endpoint"],
        ),
        connection_args={"uri": config["uri"]},
        drop_old=True,
    )
```
**EN:** Initialize vector store with documents. It works with parameters such as config and documents. Key operations include Milvus.from_documents and OpenAIEmbeddings. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。它会处理 config 和 documents 等参数。关键操作包括 Milvus.from_documents 和 OpenAIEmbeddings。其返回值会继续传给示例管线的下一阶段。

### Function: init_llm
```python
def init_llm(config: dict[str, Any]):
    """
    Initialize llm
    """
    return ChatOpenAI(
        model=config["chat_model"],
        openai_api_key=config["vllm_api_key"],
        openai_api_base=config["vllm_chat_endpoint"],
    )
```
**EN:** Initialize llm. It works with parameters such as config. Key operations include ChatOpenAI. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。它会处理 config 等参数。关键操作包括 ChatOpenAI。其返回值会继续传给示例管线的下一阶段。

### Function: get_qa_prompt
```python
def get_qa_prompt():
    """
    Get question answering prompt template
    """
    template = """You are an assistant for question-answering tasks.
Use the following pieces of retrieved context to answer the question.
If you don't know the answer, just say that you don't know.
Use three sentences maximum and keep the answer concise.
Question: {question}
Context: {context}
Answer:
"""
    return PromptTemplate.from_template(template)
```
**EN:** Get question answering prompt template. Key operations include PromptTemplate.from_template. The return value feeds the next stage of the example pipeline.
**CN:** 该函数封装示例中的可复用步骤。关键操作包括 PromptTemplate.from_template。其返回值会继续传给示例管线的下一阶段。

### Function: format_docs
```python
def format_docs(docs: list[Document]):
    """
    Format documents for prompt
    """
    return "\n\n".join(doc.page_content for doc in docs)
```
**EN:** Format documents for prompt. It works with parameters such as docs. Key operations include join. The return value feeds the next stage of the example pipeline.
**CN:** 该函数把中间结果转换成所需格式。它会处理 docs 等参数。关键操作包括 join。其返回值会继续传给示例管线的下一阶段。

### Function: create_qa_chain
```python
def create_qa_chain(retriever: Any, llm: ChatOpenAI, prompt: PromptTemplate):
    """
    Set up question answering chain
    """
    return (
        {
            "context": retriever | format_docs,
            "question": RunnablePassthrough(),
        }
        | prompt
        | llm
        | StrOutputParser()
    )
```
**EN:** Set up question answering chain. It works with parameters such as retriever, llm, and prompt. Key operations include StrOutputParser and RunnablePassthrough. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。它会处理 retriever、llm，以及 prompt 等参数。关键操作包括 StrOutputParser 和 RunnablePassthrough。其返回值会继续传给示例管线的下一阶段。

### Function: get_parser
```python
def get_parser() -> argparse.ArgumentParser:
    """
    Parse command line arguments
    """
    parser = argparse.ArgumentParser(description="RAG with vLLM and langchain")

    # Add command line arguments
    parser.add_argument(
        "--vllm-api-key", default="EMPTY", help="API key for vLLM compatible services"
    )
    parser.add_argument(
        "--vllm-embedding-endpoint",
        default="http://localhost:8000/v1",
        help="Base URL for embedding service",
    )
    parser.add_argument(
        "--vllm-chat-endpoint",
        default="http://localhost:8001/v1",
        help="Base URL for chat service",
    )
    # ... key logic omitted for brevity ...
        "--chunk-size",
        type=int,
        default=1000,
        help="Chunk size for document splitting",
    )
    parser.add_argument(
        "-o",
        "--chunk-overlap",
        type=int,
        default=200,
        help="Chunk overlap for document splitting",
    )

    return parser
```
**EN:** Parse command line arguments. Key operations include parser.add_argument and argparse.ArgumentParser. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument 和 argparse.ArgumentParser。其返回值会继续传给示例管线的下一阶段。

### Function: init_config
```python
def init_config(args: Namespace):
    """
    Initialize configuration settings from command line arguments
    """

    return {
        "vllm_api_key": args.vllm_api_key,
        "vllm_embedding_endpoint": args.vllm_embedding_endpoint,
        "vllm_chat_endpoint": args.vllm_chat_endpoint,
        "uri": args.uri,
        "embedding_model": args.embedding_model,
        "chat_model": args.chat_model,
        "url": args.url,
        "chunk_size": args.chunk_size,
        "chunk_overlap": args.chunk_overlap,
        "top_k": args.top_k,
    }
```
**EN:** Initialize configuration settings from command line arguments. It works with parameters such as args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。它会处理 args 等参数。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main():
    # Parse command line arguments
    args = get_parser().parse_args()

    # Initialize configuration
    config = init_config(args)

    # Load and split documents
    documents = load_and_split_documents(config)

    # Initialize vector store and retriever
    vectorstore = init_vectorstore(config, documents)
    retriever = vectorstore.as_retriever(search_kwargs={"k": config["top_k"]})

    # Initialize llm and prompt
    llm = init_llm(config)
    prompt = get_qa_prompt()

    # Set up QA chain
    qa_chain = create_qa_chain(retriever, llm, prompt)

    # Interactive mode
    if args.interactive:
        print("\nWelcome to Interactive Q&A System!")
        print("Enter 'q' or 'quit' to exit.")

        while True:
            question = input("\nPlease enter your question: ")
            if question.lower() in ["q", "quit"]:
                print("\nThank you for using! Goodbye!")
                break

            output = qa_chain.invoke(question)
            print(output)
    else:
        # Default single question mode
        question = "How to install vLLM?"
        output = qa_chain.invoke(question)
        print("-" * 50)
        print(output)
        print("-" * 50)
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include print, qa_chain.invoke, parse_args, init_config, and load_and_split_documents.
**CN:** 该函数编排端到端工作流。关键操作包括 print、qa_chain.invoke、parse_args、init_config，以及 load_and_split_documents。

### Entry point
```python
if __name__ == "__main__":
    main()
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **RAG orchestration / RAG 编排**: Documents are loaded, indexed, retrieved, and injected into prompts. / 文档会被加载、索引、检索，并注入到提示词中。
- **Tool calling / 工具调用**: Model outputs are mapped to callable tools or structured action requests. / 模型输出会映射为可调用工具或结构化动作请求。

## Dependencies / 依赖关系
- **External libraries / 外部库**: `argparse`, `typing`, `langchain_community.document_loaders`, `langchain_core.documents`, `langchain_core.output_parsers`, `langchain_core.prompts`, `langchain_core.runnables`, `langchain_milvus` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `load_and_split_documents`, `init_vectorstore`, `init_llm`, `get_qa_prompt`, `format_docs`, `create_qa_chain`, `get_parser`, `init_config` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `WebBaseLoader`, `loader.load`, `RecursiveCharacterTextSplitter`, `text_splitter.split_documents`, `print`, `str`, `Milvus.from_documents`, `OpenAIEmbeddings` reveal the main execution path / 这些调用体现了主要执行链路。
