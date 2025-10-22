# Agents SDK MCP Project

This project is an implementation of the **Model Context Protocol (MCP)** using the OpenAI Agents SDK, but configured to leverage the free Gemini API from Google AI. It's designed as a learning resource for students interested in AI agents, MCPs, and integrating large language models (LLMs) with external tools and data sources.

## What is MCP?

MCP (Model Context Protocol) is a standard for connecting AI models to external data sources, tools, and resources. This allows agents to interact with the world beyond just text generation, enabling tasks like reading documents, editing files, and accessing structured data.

In this project, we build a simple MCP server that provides tools for document management and a CLI chatbot that uses these tools via an MCP client.

## Prerequisites

Before getting started, ensure you have:

- Python 3.13 or higher
- `uv` package manager (for managing dependencies and virtual environments)
- A Google AI API key (free tier available for Gemini models)

## Setup and Initialization

1. **Create a new project:**
   ```bash
   uv init agents-sdk-mcp
   cd agents-sdk-mcp
   ```

2. **Activate the virtual environment:**
   ```bash
   source .venv/bin/activate
   ```

3. **Set up Python interpreter in VS Code:**
   - Press `Ctrl + Shift + P` (or `Cmd + Shift + P` on Mac)
   - Select "Python: Select Interpreter"
   - Choose the path to your `.venv` folder (e.g., `/path/to/agents-sdk-mcp/.venv/bin/python`)

4. **Install dependencies:**
   ```bash
   uv add mcp uvicorn openai-agents prompt-toolkit
   ```

5. **Set up environment variables:**
   Create a `.env` file in the project root with the following:
   ```env
   LLM_MODEL_API_KEY=your_gemini_api_key_here
   LLM_CHAT_COMPLETION_URL=https://generativelanguage.googleapis.com/v1beta/openai/
   LLM_MODEL=models/gemini-flash-latest
   ```
   - Get your API key from [Google AI Studio](https://makersuite.google.com/app/apikey)
   - Replace `your_gemini_api_key_here` with your actual key

## Project Architecture

The project consists of three main components:

- **CLI Chatbot (`main.py`)**: The user-facing interface for interacting with the AI agent.
- **MCP Server (`mcp_server.py`)**: Provides tools and resources for the agent.
- **MCP Client (`mcp_client.py`)**: Connects the chatbot to the MCP server.

Here's a simple flowchart of the architecture:

```
User Input
    ↓
CLI Chatbot (main.py)
    ↓
MCP Client (mcp_client.py)
    ↓
MCP Server (mcp_server.py)
    ↓
Tools/Resources (e.g., read_docs, edit_docs)
    ↓
AI Model (Gemini via OpenAI SDK)
    ↓
Response back to User
```

The MCP server runs on `http://localhost:8000/mcp/` and must be started before the chatbot for the integration to work.

## Running the Project

1. **Start the MCP Server:**
   ```bash
   uv run uvicorn mcp_server:mcp_app --reload
   ```
   - This runs the server in the background. Keep this terminal open.

2. **Run the CLI Chatbot:**
   ```bash
   uv run main.py
   ```
   - This launches an interactive chat session where you can query the agent.

3. **Test the MCP Client (Optional):**
   ```bash
   uv run mcp_client.py
   ```
   - This will list available tools and demonstrate calling one (e.g., reading a document).

## Design Details

### main.py
This file sets up the CLI chatbot and integrates it with the MCP client. It loads environment variables, connects to the MCP server, and initializes the agent service with the Gemini model. The chatbot allows users to interact with the AI, which can use MCP tools to perform actions like reading or editing documents.

### mcp_server.py
The MCP server defines tools and resources:
- **Tools**: Functions that the AI can call, such as `read_docs` and `edit_docs`.
- **Resources**: Data endpoints, like listing documents or fetching specific content.
- It uses the `FastMCP` library to create a stateless HTTP server.

### mcp_client.py
This handles communication with the MCP server. It provides methods to:
- List available tools
- Call specific tools
- Read resources
- The client connects to the server via HTTP and parses responses.

## Tools Implementation

Two example tools are implemented to demonstrate MCP tool usage:

1. **`read_docs`**:
   - **Description**: Reads the contents of a specified document and returns it as a string.
   - **Parameters**: `doc_id` (e.g., "deposition.md")
   - **Usage**: The AI can call this to retrieve document content for analysis or summarization.

2. **`edit_docs`**:
   - **Description**: Edits a document by replacing a specific string with new text.
   - **Parameters**: `doc_id`, `old_str`, `new_str`
   - **Usage**: Allows the AI to make changes to documents, such as corrections or updates.

These tools are controlled by the LLM (Gemini), meaning the AI decides when and how to use them based on user queries.

## Resources

Resources in MCP provide access to data sources. In this project:

- **Server-side (`mcp_server.py`)**:
  - `list_docs()`: Returns a list of all document IDs (e.g., via `docs://documents`).
  - `get_doc(doc_id)`: Returns the content of a specific document (e.g., via `docs://deposition.md`).

- **Client-side (`mcp_client.py`)**:
  - `read_resource(uri)`: Fetches and parses resource content. Supports text and JSON formats.

**Testing Tips**:
- Use Postman to test resource endpoints directly (import the collection from `postman/` folder).
- In the CLI chat, prefix with "@" to list or access resources (e.g., "@docs://documents").

## Prompts

Prompts are pre-defined templates that help users get better results from the AI without crafting complex prompts themselves.

- **Server-side**: The MCP server can define standard prompts.
- **Client-side**: The client can fetch and use these prompts to guide the AI's responses.

In the CLI chat, prefix with "/" to list or access prompts.

## Contributing and Learning

This project is a starting point for exploring MCP and AI agents. Feel free to extend it by:
- Adding more tools (e.g., file upload, search)
- Implementing prompts for common tasks
- Integrating with other APIs

For questions or improvements, refer to the code comments and documentation in the source files.

---

Happy coding! If you have any issues, check the console output for errors related to API keys or server connections.