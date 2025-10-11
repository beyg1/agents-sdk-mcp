# Agents SDK MCP Project Specification

## Project Overview

This project is a **student-friendly learning tool** that shows how to connect **AI agents** (using OpenAI's Agents SDK) with **external tools and resources** (using the Model Context Protocol or MCP).

Think of it like this: You have an AI assistant (like ChatGPT) that you want to connect to external tools (like document readers, calculators, or other services). This project shows you exactly how to do that!

The project is based on Anthropic's MCP course but uses **free Gemini API** instead of Claude, making it cheaper for students to experiment with.

## Architecture Overview

Here's how all the pieces fit together:

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  Command Line   │◄──►│   AI Agent       │◄──►│   Tool         │
│  Interface      │    │   Service        │    │   Clients       │
│  (Lets you chat)│    │ (Main AI Brain)  │    │ (Tool Manager)  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         ▲                        ▲                        │
         │                        │                        ▼
         │                        │                ┌─────────────────┐
         └────────────────────────┘                │   Tool         │
                                                  │   Server        │
                                                  │ (Document Tools)│
                                                  └─────────────────┘
```

## Components Overview

**Main Entry Point** (`main.py`): The main application launcher that configures AI settings, connects to MCP servers, and starts the chat interface.

**MCP Client** (`mcp_client.py`): Remote control system that connects AI agents to MCP servers and enables tool usage over network connections.

**MCP Server** (`mcp_server.py`): Document management server that provides tools for reading, editing, and managing documents to AI agents.

**Agent Service** (`core/agent_service.py`): Bridge component that translates MCP tools into OpenAI Agents SDK format for seamless integration.

**Chat Interface** (`core/chat.py`): Basic chat system that handles message processing and response generation between users and AI agents.

**CLI Chat** (`core/cli_chat.py`): Advanced chat processor that handles special commands, document mentions, and context injection.

**CLI Application** (`core/cli.py`): Rich terminal interface with auto-completion, command hints, and intelligent user experience features.

**Tool Manager** (`core/tools.py`): Central coordinator that manages tools from multiple MCP clients and routes requests appropriately.


## Components Specifications

### 1. Main Entry Point (`main.py`)

**What it does**: This is like the "conductor" of the whole orchestra. It starts everything up and makes sure all parts work together.

**What are its jobs**:
- **Setup AI Configuration**: Reads your AI settings (like which AI model to use, API keys, etc.) from environment variables stored in a `.env` file
- **Connect to Tools**: Creates connections to various MCP servers (tools)
- **Start AI Service**: Sets up the main AI agent with all the tool connections
- **Launch Chat Interface**: Starts the command-line chat where you can talk to the AI

**Configuration Requirements** (these go in your `.env` file):
```bash
# Put these in your .env file:
LLM_MODEL=gemini-pro                    # Which AI model to use
LLM_MODEL_API_KEY=your_actual_api_key   # Your real API key from Google
LLM_CHAT_COMPLETION_URL=your_gemini_endpoint  # Gemini API URL
```

**Step-by-Step Execution Flow**:
1. **Check Settings**: First, it checks if all required settings are present in your `.env` file. If any are missing, it shows an error and stops.
2. **Create Document Client**: Sets up a connection to the document server (running at http://localhost:8000/mcp/)
3. **Create More Tool Clients**: For any additional MCP servers you want to use (like servers started with `uv run mcp_server.py`)
4. **Setup AI Service**: Creates the main AI agent service and gives it access to all the tool clients
5. **Start Chat Interface**: Launches the command-line chat where you can interact with the AI and use tools

**What Students Learn**: How to properly start an application and manage multiple tool connections like a professional developer.

### 2. MCP Server (`mcp_server.py`)

**What it does**: This is a **tool server** that provides document-related tools to AI agents. Think of it as a librarian that helps the AI read and manage documents.

**How it works**:
- **Document Library**: Stores sample documents in memory (like a simple dictionary)
- **Web Server**: Uses FastMCP to create a web server that AI agents can connect to
- **No Memory**: Runs as a stateless server (doesn't remember things between requests)

**Sample Documents Available**:
- `deposition.md`: A legal statement from someone named Angela Smith
- `report.pdf`: A technical report about a 20m condenser tower
- `financials.docx`: Budget and money information for the project
- `outlook.pdf`: Predictions about how the system will perform in the future
- `plan.md`: Step-by-step plan for implementing the project
- `spec.txt`: Technical requirements for equipment

**What Still Needs to Be Built** (TODO Items):
- **Document Reader Tool**: A tool that can read any document's content
- **Document Editor Tool**: A tool that can modify/edit documents
- **Document List Resource**: A way to get a list of all available document names
- **Document Content Resource**: A way to get the full content of a specific document
- **Markdown Converter Prompt**: A template to rewrite documents in markdown format
- **Document Summarizer Prompt**: A template to create summaries of documents

**What Students Learn**: How to create servers that provide useful tools to AI agents, and how to plan out features systematically.

### 3. MCP Client (`mcp_client.py`)

**What it does**: This is like a **remote control** that lets your AI agent connect to and use tools from MCP servers.

**How to Connect**:
- **Web Connection**: Connect to servers running on the web (like `http://localhost:8000/mcp/`)
- **Program Connection**: Start and connect to tool servers as separate programs

**Main Functions** (what it can do):
- `connect()`: Make the initial connection to a tool server
- `list_tools()`: Ask the server "What tools do you have?"
- `call_tool()`: Use a specific tool with your own settings
- `list_prompts()`: Get a list of prompt templates from the server
- `get_prompt()`: Get a specific prompt template with your arguments
- `read_resource()`: Access resources (like documents) using web addresses (URIs)

**Smart Connection Management**:
- **Auto-Cleanup**: Automatically cleans up connections when done (using async context managers)
- **Session Handling**: Manages connection sessions properly
- **Error Prevention**: Prevents connection leaks and manages resources safely

**Current State**: All the functions are planned out but not fully implemented yet (they just have placeholder code)

**What Students Learn**: How to create reliable connections to external tools and manage those connections safely.

### 4. Core Module Components

These are the **helper modules** in the `core/` folder that do the actual work of making everything function together.

#### Agent Service (`core/agent_service.py`)

**What it does**: This is the **bridge** that connects the OpenAI Agents SDK (the AI part) with MCP clients (the tool part).

**Main Features**:
- **Tool Translation**: Converts MCP tools into a format that OpenAI Agents can understand
- **Client Manager**: Handles multiple MCP clients as if they were one big toolbox
- **Memory Manager**: Keeps track of conversation history and context
- **Dynamic Tool Discovery**: Finds and adds new tools while the program is running

**How Tool Integration Works** (Step-by-Step):
1. **Tool Hunt**: Looks through all connected MCP clients to find available tools
2. **Format Conversion**: Changes MCP tool descriptions into OpenAI Agent format
3. **Wrapper Creation**: Makes smart wrapper functions that handle errors properly
4. **Agent Connection**: Adds these tools to the AI agent so it can use them during conversations

**What Students Learn**: How to make different systems (AI agents and external tools) work together smoothly.

#### Chat Interface (`core/chat.py`)

**What it does**: This is the **basic chat system** that handles conversations between you and the AI agent.

**Main Jobs**:
- **Message Handler**: Takes your messages and sends them to the right place
- **Response Generator**: Gets responses from the AI and formats them nicely
- **State Manager**: Keeps track of the current conversation state
- **Tool Connector**: Makes sure the AI can use MCP tools when needed

**What Students Learn**: How to separate the user interface (chat) from the business logic (AI processing).

#### CLI Chat (`core/cli_chat.py`)

**What it does**: This is an **advanced chat system** that adds special command-line features to the basic chat.

**Cool Features**:
- **Command Processing**: Handles special commands like `/summarize document.pdf`
- **Document Mentioning**: Understands when you write `@document` in your message
- **Auto Context**: Automatically adds document content to help the AI understand
- **Prompt Templates**: Uses pre-made prompt templates from MCP servers

**How Document Processing Works**:
- **Find Mentions**: Looks for `@document` references in your messages
- **Get Content**: Asks the MCP client to fetch the document content
- **Add Context**: Inserts document content as XML-tagged information
- **Keep It Separate**: Makes sure your original question stays separate from the document context

**What Students Learn**: How to do advanced text processing and automatically provide relevant context to AI.

#### CLI Application (`core/cli.py`)

**What it does**: This creates the **fancy command-line interface** that you actually interact with when running the program.

**Awesome Features**:
- **Smart Auto-complete**: Guesses what you're typing and suggests completions
- **Command Hints**: Shows helpful suggestions based on what you're doing
- **Resource Finder**: Automatically discovers and remembers available tools/documents
- **History Keeper**: Remembers your previous commands
- **Keyboard Shortcuts**: Special key combinations for quick actions

**How the Completion System Works**:
- **Command Mode**: Type `/` to get command suggestions with argument hints
- **Resource Mode**: Type `@` to get document/resource name completions
- **Auto Refresh**: Updates available commands and resources as they change
- **Background Processing**: Does completions in the background so the interface stays responsive

**What Students Learn**: How to create professional-quality command-line interfaces with great user experience.

#### Tool Manager (`core/tools.py`)

**What it does**: This is the **tool coordinator** that manages all the different tools from various MCP clients.

**Main Functions**:
- **Tool Collector**: Gathers tools from all connected MCP clients into one place
- **Client Finder**: Figures out which MCP client has the specific tool you need
- **Dynamic Wrapper**: Creates on-the-fly wrapper functions for the OpenAI Agent SDK
- **Error Manager**: Handles problems that might happen when using tools

**How Tool Execution Works** (Step-by-Step)**:
1. **Parse Request**: Takes the tool request and arguments from the AI agent (in JSON format)
2. **Find Right Client**: Routes the request to the correct MCP client that has this tool
3. **Execute Tool**: Runs the tool using the MCP protocol
4. **Format Results**: Returns the results in a format the AI agent can understand

**What Students Learn**: How to organize and manage complex tool ecosystems and create flexible wrapper systems.

## Data Flow Architecture

Here's how information flows through the system - think of this as **following a message from your keyboard all the way to getting an answer back**!

### Message Processing Flow (What happens when you send a message)

```
Your Message → CLI Understanding → Command Check → Context Addition → AI Processing → Tool Usage → Final Answer
     ↓              ↓                    ↓                ↓              ↓              ↓             ↓
1. You type       2. CLI app         3. Check if      4. Add         5. AI agent    6. If needed,  7. Format and
   something        parses your         it's a          relevant       processes      use tools     show response
                    input               command         documents      your message   to help answer
```

**Detailed Steps**:
1. **Your Input**: You type a message in the command line
2. **CLI Parsing**: The CLI app reads and understands what you typed
3. **Command Check**: Checks if you're using special commands (like `/summarize`)
4. **Context Addition**: If you mentioned documents (@document), it automatically adds their content
5. **AI Processing**: Sends your message to the AI agent along with any context
6. **Tool Usage**: If the AI needs more info, it uses tools from MCP servers
7. **Response**: Final answer comes back and is displayed

### Tool Execution Flow (What happens when AI uses a tool)

```
AI Agent → Tool Manager → MCP Client → MCP Server → Tool Execution → Result Processing → Response Back
    ↓            ↓             ↓            ↓              ↓                ↓              ↓
1. AI says     2. Tool        3. MCP      4. MCP        5. Server       6. Format      7. Send back
   "I need"      manager       client      server         runs the        results        to AI agent
   tool help     finds right   connects   receives       actual tool     for AI
                 tool/client    to server   request
```

**Detailed Steps**:
1. **AI Request**: AI agent decides it needs to use a tool to help answer
2. **Tool Manager**: Finds which tool and which MCP client has it
3. **Client Connection**: MCP client connects to the right MCP server
4. **Server Request**: Server receives the tool request
5. **Tool Execution**: Server runs the actual tool (like reading a document)
6. **Result Processing**: Formats the results so AI can understand them
7. **Response**: Sends the tool results back to the AI agent

### Resource Access Flow (How documents get loaded)

```
Document Mention → Web Address → MCP Client → Server Storage → Content Loading → Context Addition
       ↓                ↓             ↓             ↓               ↓               ↓
1. You mention      2. Convert to   3. MCP       4. Server      5. Get the      6. Add to
   @document         web address     client      finds in       actual         your message
                     (URI)          connects    storage        content        as context
```

**Detailed Steps**:
1. **Document Mention**: You type something like "@report.pdf" in your message
2. **Address Creation**: System converts this to a proper web address (URI) like "docs://documents/report.pdf"
3. **Client Connection**: MCP client connects to the document server
4. **Server Lookup**: Server looks up the document in its storage (currently just a dictionary)
5. **Content Loading**: Retrieves the actual document content
6. **Context Addition**: Adds the document content to your message as helpful context for the AI


### Configuration (The Important Part!)

**Create a `.env` file** in your project root with these settings:

```bash
# Your AI Model Settings - Put these in .env file:
LLM_MODEL="models/gemini-flash-latest"
LLM_MODEL_API_KEY=your_actual_gemini_api_key_here
LLM_CHAT_COMPLETION_URL=https://generativelanguage.googleapis.com/v1beta/openai/
```

**Getting Your Gemini API Key**:
1. Go to [Google AI Studio](https://aistudio.google.com/)
2. Sign in with your Google account
3. Create a new API key
4. Copy the key and paste it in your `.env` file

**Important Notes**:
- **Never share your `.env` file** - it contains your API key!
- The `.env` file should be in the same folder as `main.py`
- Make sure you have some Gemini API credits (free tier is usually enough for learning)

## Learning Points for Students

Here's what you'll learn by studying this project - these are **real-world skills** that will help you in your programming career!

### 1. Understanding MCP Protocol (Model Context Protocol)
- **Tool Exposure**: How servers can offer tools and resources to AI agents
- **Client-Server Communication**: How programs talk to each other over networks
- **Protocol Standards**: How to make sure different systems can work together

### 2. OpenAI Agents SDK Integration
- **Tool Translation**: How to convert tools from one format to another
- **Agent Lifecycle**: How to manage an AI agent's "life" from start to finish
- **Message Management**: How to keep track of conversation history and context

### 3. Command-Line Interface (CLI) Design
- **Rich Terminal Apps**: How to build powerful command-line programs
- **Smart Completion**: How to add auto-complete that actually helps users
- **User Experience**: How to make command-line tools pleasant to use

### 4. Async Programming Patterns
- **Context Managers**: How to properly manage resources that need cleanup
- **Resource Management**: How to avoid memory leaks and connection problems
- **Concurrent Operations**: How to handle multiple connections at the same time

### 5. Software Architecture Best Practices
- **Separation of Concerns**: How to organize code into logical, manageable pieces
- **Dependency Injection**: How to make components that depend on each other
- **Extensible Design**: How to build software that can grow and change easily

**Key Takeaway**: This project shows you how to build **professional-grade software** that connects AI with external tools - exactly the kind of system used in real companies!

## Current Development State

**Implemented**:
- Project structure and architecture
- MCP client/server framework
- OpenAI Agents SDK integration
- Rich CLI interface
- Document storage foundation

**In Development** (TODO Items):
- Tool implementations in MCP server
- Resource access methods
- Prompt template system
- Document management features

**Educational Ready**: The codebase provides excellent learning examples for all implemented components and demonstrates industry best practices for structuring complex agent-tool integrations.

## Extension Opportunities

Here are some **cool ideas** for extending this project - great for student projects or further learning:

1. **Complete the MCP Server**: Finish implementing all the TODO items to make a fully functional document management system

2. **Add More Tool Types**: Create additional MCP servers for different capabilities like:
   - Calculator tools for math operations
   - Web scraping tools for fetching online data
   - File system tools for reading/writing files
   - Database tools for data queries

3. **Build a Web Interface**: Create a web-based chat UI using the same backend - learn web development and API design

4. **Add Persistent Storage**: Replace the simple in-memory storage with a real database like SQLite or PostgreSQL

5. **User Authentication**: Add user login and access control - learn about security and user management

6. **Performance Monitoring**: Add logging and metrics to track how well the system performs

7. **Docker Container**: Package everything in Docker containers for easy deployment

8. **API Rate Limiting**: Add controls to manage API usage and costs

9. **Tool Chaining**: Allow tools to call other tools in sequence for complex tasks

10. **Plugin System**: Make it easy to add new tools without changing core code

**Pro Tip**: Start with #1 (completing the MCP server) - it's the foundation for everything else!

---

## Final Summary

This **Agents SDK MCP Project** is your **complete learning playground** for understanding how AI agents can use external tools! It shows you:

- ✅ **How AI connects to tools** (MCP protocol)
- ✅ **How to build smart chat interfaces** (CLI with auto-complete)
- ✅ **How to manage complex software systems** (multiple components working together)
- ✅ **How professionals structure code** (separation of concerns, error handling)