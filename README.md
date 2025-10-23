# Agents SDK MCP Project

This project is an implementation of the **Model Context Protocol (MCP)** using the OpenAI Agents SDK, but configured to leverage the free Gemini API from Google AI. It's designed as a learning resource for students interested in AI agents, MCPs, and integrating large language models (LLMs) with external tools and data sources.

## What is MCP?

MCP (Model Context Protocol) is a standard for connecting AI models to external data sources, tools, and resources. This allows agents to interact with the world beyond just text generation, enabling tasks like reading documents, editing files, and accessing structured data.

### Why MCP Matters for Beginners
As a student, think of MCP as the "bridge" that turns your AI from a simple chatbot into a powerful assistant. Without MCP, AI models like Gemini or GPT can only generate text based on what they've been trained on. But with MCP's core primitives—**tools**, **resources**, and **prompts**—your AI can:
- **Perform Actions**: Use tools to read, edit, or manipulate data in real-time.
- **Access Data**: Fetch resources like documents or databases to inform responses.
- **Follow Templates**: Use prompts to ensure consistent, high-quality outputs for common tasks.

This design is crucial because it enables AI to handle real-world scenarios, such as automating document processing or integrating with APIs, making it a foundational skill for building advanced AI applications.

In this project, we build a simple MCP server that provides tools for document management and a CLI chatbot that uses these tools via an MCP client. By the end, you'll understand how to set up and extend your own MCP-based system.

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
   - Get your API key from [Google AI Studio](https://aistudio.google.com/)
   - Replace `your_gemini_api_key_here` with your actual key

## Project Architecture

The project consists of three main components:

- **CLI Chatbot (`main.py`)**: The user-facing interface for interacting with the AI agent.
- **MCP Server (`mcp_server.py`)**: Provides tools and resources for the agent.
- **MCP Client (`mcp_client.py`)**: Connects the chatbot to the MCP server.

### Visual Architecture Overview
Here's a Mermaid diagram illustrating the data flow and how MCP primitives (tools, resources, prompts) integrate:

graph TD
    A[User Input] --> B["CLI Chatbot\n(main.py)"]
    B --> C["MCP Client\n(mcp_client.py)"]
    C --> D["MCP Server\n(mcp_server.py)"]
    D --> E["Tools\n(e.g., read_docs, edit_docs)"]
    D --> F["Resources\n(e.g., docs://documents)"]
    D --> G["Prompts\n(e.g., /summarize)"]
    E --> H["AI Model\n(Gemini via OpenAI SDK)"]
    F --> H
    G --> H
    H --> I["Response back to User"]

    subgraph "MCP Primitives"
        E
        F
        G
    end
    


This diagram shows how user input flows through the system and how the AI leverages MCP primitives to perform actions, access data, and follow templates for enhanced responses.

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

This section breaks down the core components and how they work together, focusing on the MCP primitives. Each part includes examples to help you understand how to implement similar setups.

### main.py: The Entry Point
This file is the "conductor" of the project. It:
1. Loads environment variables from `.env` (e.g., API keys).
2. Connects to the MCP server (running at `http://localhost:8000/mcp/`).
3. Initializes the AI agent with the Gemini model via the OpenAI Agents SDK.
4. Starts the CLI chat interface for user interaction.

**Example**: When you run `uv run main.py`, it checks your `.env` file, sets up the agent, and launches the chat. The AI can then decide to use tools based on your queries, like reading a document if you ask for a summary.

### mcp_server.py: The Tool and Resource Provider
This server provides the MCP primitives:
- **Tools**: Executable functions the AI can invoke (e.g., `read_docs` to read a file).
- **Resources**: Data access points (e.g., `docs://documents` for listing files).
- **Prompts**: Templates for consistent responses (e.g., a summarization prompt).
It uses the `FastMCP` library to run as a stateless HTTP server.

**Example**: If the AI needs to edit a document, it calls the `edit_docs` tool via the server, which performs the action and returns the result.

### mcp_client.py: The Bridge to the Server
This client manages communication:
- Connects to the MCP server via HTTP.
- Lists and calls tools, reads resources, and fetches prompts.
- Handles responses and errors for seamless integration.

**Example**: When you type "@docs://deposition.md" in the chat, the client fetches the document content from the server and injects it into the AI's context.

## Tools Implementation

Tools are the "actions" your AI can take, like functions in a program. The AI decides when to call them based on user input. Below are examples with step-by-step breakdowns.

### Example Tool: `read_docs`
- **Description**: Reads a document and returns its content as text for the AI to process.
- **Parameters**: `doc_id` (e.g., "deposition.md").
- **How It Works** (Step-by-Step):
  1. User asks: "Summarize the deposition.md."
  2. AI decides it needs the document content.
  3. AI calls `read_docs` via the MCP server.
  4. Server fetches and returns the text.
  5. AI uses the content to generate a summary.
- **Usage in Chat**: The AI automatically invokes this if relevant—no need for you to call it directly.
- **Learning Tip**: This tool shows how AI can "read" external data, enabling tasks like summarization or analysis.

### Example Tool: `edit_docs`
- **Description**: Modifies a document by replacing text.
- **Parameters**: `doc_id`, `old_str`, `new_str`.
- **How It Works** (Step-by-Step):
  1. User says: "Change 'old text' to 'new text' in report.md."
  2. AI identifies the need to edit.
  3. AI calls `edit_docs` with the parameters.
  4. Server updates the document and confirms.
  5. AI responds with the updated content.
- **Usage in Chat**: AI handles the editing seamlessly based on your instructions.
- **Learning Tip**: Demonstrates AI-driven modifications, useful for automation like corrections in legal docs.

These tools are powered by the LLM (Gemini), allowing the AI to reason and execute actions dynamically.

## Resources

Resources are like "data endpoints" that give AI access to information, such as files or databases, without hardcoding everything.

### Server-Side Implementation (`mcp_server.py`)
- **list_docs()**: Lists all available documents (accessed via `docs://documents`).
- **get_doc(doc_id)**: Fetches a specific document's content (e.g., `docs://deposition.md`).

**Example**: The server stores sample documents in memory. When AI needs data, it calls these resources to "look up" information.

### Client-Side Usage (`mcp_client.py`)
- **read_resource(uri)**: Retrieves and formats resource data (e.g., text or JSON).

**How It Works** (Step-by-Step):
1. User types: "@docs://deposition.md" in the chat.
2. CLI recognizes the "@" prefix and triggers resource fetch.
3. Client sends a request to the server for that URI.
4. Server returns the document content.
5. Content is added to the AI's context for a more informed response.
- **Usage in Chat**: Prefix messages with "@" to access resources (e.g., "@docs://documents" to list all).

**Learning Tip**: Resources enable AI to "remember" or access external data, making responses more accurate—like referencing a real document for summaries.

**Testing Tips**:
- Use Postman (import from `postman/` folder) to test endpoints directly.
- In CLI, use "@" for quick access during chats.

## Prompts

Prompts are reusable templates that guide the AI to produce consistent, high-quality responses for common tasks, reducing the need for users to write detailed instructions.

### Server-Side Definition (`mcp_server.py`)
- The server defines standard prompts (e.g., for summarization or markdown conversion).

### Client-Side Usage (`mcp_client.py`)
- Fetches and applies prompts to enhance AI interactions.

**How It Works** (Step-by-Step):
1. User types: "/summarize" in the chat.
2. CLI recognizes the "/" prefix and loads the summarization prompt.
3. Prompt is sent to the AI along with the query (e.g., "Summarize the following document using this template.").
4. AI generates a structured response based on the template.
- **Usage in Chat**: Prefix with "/" to list or use prompts (e.g., "/list_prompts" to see available ones).

**Example**: A prompt might instruct the AI to "Start with an overview, then key points, and end with conclusions" for summaries, ensuring reliable output.

**Learning Tip**: Prompts standardize AI behavior, making it easier to get professional results—like using a template for report generation in business apps.

## Learning Outcomes and Setup Guidance

By following this README, you'll gain hands-on experience with MCP, enabling you to set up your own system. Key takeaways:

- **Understanding MCP Primitives**: Learn how tools (actions), resources (data access), and prompts (templates) empower AI to perform real-world tasks.
- **Building AI Agents**: Practice integrating LLMs with external systems for automation.
- **CLI and Networking**: Set up servers, clients, and interactive interfaces.
- **Real-World Applications**: See how this applies to document management, API integrations, and more.

**Next Steps for Your Own MCP**:
1. **Extend the Server**: Add more tools (e.g., web scraping) by modifying `mcp_server.py`.
2. **Customize Resources**: Integrate with databases or APIs in `mcp_server.py`.
3. **Create Prompts**: Define templates for your needs and add them to the server.
4. **Test and Iterate**: Use the CLI or Postman to experiment, then build your own client.

## Contributing and Learning

This project is a starting point for exploring MCP and AI agents. Feel free to extend it by:
- Adding more tools (e.g., file upload, search)
- Implementing prompts for common tasks
- Integrating with other APIs

For questions or improvements, refer to the code comments and documentation in the source files. To set up your own MCP:
1. Start with a simple server like `mcp_server.py`.
2. Define primitives in code.
3. Build a client to connect them.
4. Test with a CLI or web interface.

---

Happy coding! If you have any issues, check the console output for errors related to API keys or server connections.