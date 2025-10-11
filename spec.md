# Agents SDK MCP Project Specification

## Project Overview

This project is an educational implementation that demonstrates how to integrate the **OpenAI Agents SDK** with the **Model Context Protocol (MCP)**. It's designed as a learning tool for students to understand how Large Language Models (LLMs) can interact with external tools and resources through structured protocols.

The project serves as a practical imitation of Anthropic's MCP course, adapted to use the OpenAI Agents SDK with free Gemini API access instead of Claude. This makes it more accessible for students who want to experiment with agent-tool interactions without API costs.

## Architecture Overview

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   CLI Interface │◄──►│  Agent Service   │◄──►│   MCP Clients   │
│     (cli.py)    │    │ (agent_service.py)│    │ (mcp_client.py) │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         ▲                        ▲                        │
         │                        │                        ▼
         │                        │                ┌─────────────────┐
         └────────────────────────┘                │   MCP Server    │
                                                  │ (mcp_server.py) │
                                                  └─────────────────┘
```

## Component Specifications

### 1. Main Entry Point (`main.py`)

**Purpose**: Orchestrates the entire application lifecycle and manages MCP client connections.

**Key Responsibilities**:
- **Environment Configuration**: Loads and validates LLM configuration from environment variables
- **Client Management**: Creates and manages multiple MCP client connections
- **Service Initialization**: Sets up the AgentService with LLM credentials and connected clients
- **CLI Integration**: Initializes and runs the command-line chat interface

**Configuration Requirements**:
```bash
LLM_MODEL=gemini-pro                    # Model identifier
LLM_MODEL_API_KEY=your_api_key         # API authentication
LLM_CHAT_COMPLETION_URL=gemini_endpoint # Base URL for completions
```

**Execution Flow**:
1. Validates environment configuration
2. Creates document client (HTTP-based MCP server)
3. Creates additional clients for command-line MCP servers
4. Initializes AgentService with all clients
5. Starts CLI chat interface

**Educational Value**: Demonstrates proper application initialization patterns and dependency management.

### 2. MCP Server (`mcp_server.py`)

**Purpose**: Implements an MCP-compliant server that provides document management capabilities.

**Current Implementation**:
- **Document Storage**: Simple in-memory dictionary storing sample documents
- **Server Framework**: Uses FastMCP for HTTP-based MCP protocol handling
- **Stateless Operation**: Configured for HTTP streaming without server state

**Sample Documents**:
- `deposition.md`: Legal testimony document
- `report.pdf`: Technical condenser tower report
- `financials.docx`: Project budget information
- `outlook.pdf`: System performance projections
- `plan.md`: Project implementation steps
- `spec.txt`: Technical equipment specifications

**Planned Features** (TODO Items):
- Tool to read document contents
- Tool to edit/modify documents
- Resource to list all document IDs
- Resource to retrieve specific document contents
- Prompt templates for document rewriting (markdown format)
- Prompt templates for document summarization

**Educational Value**: Shows how to structure MCP servers and plan tool/resource development.

### 3. MCP Client (`mcp_client.py`)

**Purpose**: Provides a wrapper interface for connecting to and interacting with MCP servers.

**Connection Modes**:
- **HTTP Mode**: Connect to servers via HTTP endpoints (`http://localhost:8000/mcp/`)
- **Command Mode**: Launch and connect to servers as subprocesses

**Core Interface Methods**:
- `connect()`: Establish connection to MCP server
- `list_tools()`: Retrieve available tools from server
- `call_tool()`: Execute specific tool with parameters
- `list_prompts()`: Get available prompt templates
- `get_prompt()`: Retrieve specific prompt with arguments
- `read_resource()`: Access server resources by URI

**Context Management**:
- Implements async context manager protocol (`__aenter__`, `__aexit__`)
- Proper cleanup of connections and resources
- Session state management

**Current State**: All methods contain placeholder implementations (TODO items)

**Educational Value**: Demonstrates MCP client architecture and async resource management patterns.

### 4. Core Module Components

#### Agent Service (`core/agent_service.py`)

**Purpose**: Bridges OpenAI Agents SDK with MCP client ecosystem.

**Key Features**:
- **Tool Conversion**: Converts MCP tools to OpenAI Agents SDK format
- **Client Integration**: Manages multiple MCP clients as a unified tool source
- **Agent Lifecycle**: Handles conversation state and message history
- **Dynamic Tool Loading**: Discovers and integrates tools at runtime

**Tool Integration Process**:
1. Scans all connected MCP clients for available tools
2. Converts MCP tool schemas to SDK-compatible format
3. Creates dynamic tool wrappers with proper error handling
4. Integrates tools into agent for runtime execution

**Educational Value**: Shows advanced patterns for tool abstraction and SDK integration.

#### Chat Interface (`core/chat.py`)

**Purpose**: Provides basic conversational interface using the agent service.

**Responsibilities**:
- Message handling and routing
- Response generation and formatting
- Client state management
- Integration with MCP tool ecosystem

**Educational Value**: Demonstrates clean abstraction between UI and business logic.

#### CLI Chat (`core/cli_chat.py`)

**Purpose**: Extends basic chat with command-line specific functionality.

**Advanced Features**:
- **Command Processing**: Handles slash commands (e.g., `/summarize document.pdf`)
- **Document Referencing**: Processes `@document` mentions in queries
- **Context Injection**: Automatically includes referenced document content
- **Prompt Templates**: Integrates MCP prompt templates into conversations

**Document Processing**:
- Extracts `@document` references from user queries
- Retrieves document contents via MCP client
- Injects content as XML-tagged context
- Maintains separation between user intent and system context

**Educational Value**: Shows sophisticated text processing and context management techniques.

#### CLI Application (`core/cli.py`)

**Purpose**: Provides rich command-line interface with advanced features.

**Advanced Features**:
- **Auto-completion**: Intelligent command and resource completion
- **Command Suggestions**: Context-aware command suggestions
- **Resource Management**: Dynamic resource discovery and caching
- **History Management**: Persistent command history
- **Key Bindings**: Custom keyboard shortcuts for commands

**Completion System**:
- `/` prefix for commands with argument suggestions
- `@` prefix for resource references with ID completion
- Dynamic refresh of available commands and resources
- Threaded completion for responsive UI

**Educational Value**: Demonstrates advanced CLI design patterns and user experience considerations.

#### Tool Manager (`core/tools.py`)

**Purpose**: Manages tool discovery and execution across MCP clients.

**Key Functions**:
- **Tool Discovery**: Aggregates tools from multiple MCP clients
- **Client Resolution**: Finds appropriate client for specific tools
- **Dynamic Execution**: Creates runtime tool wrappers for SDK integration
- **Error Handling**: Manages tool execution failures gracefully

**Tool Execution Pattern**:
1. Parse JSON arguments from SDK
2. Route to appropriate MCP client
3. Execute tool via MCP protocol
4. Return structured results

**Educational Value**: Shows tool abstraction patterns and dynamic function generation.

## Data Flow Architecture

### Message Processing Flow
```
User Input → CLI Parsing → Command Processing → Context Injection → Agent Execution → Tool Calls → Response Generation
```

### Tool Execution Flow
```
Agent → Tool Manager → MCP Client → MCP Server → Tool Execution → Result Processing → Response
```

### Resource Access Flow
```
Document Reference → Resource URI → MCP Client → Server Resource → Content Retrieval → Context Injection
```

## Setup and Configuration

### Prerequisites
- Python virtual environment (uv)
- OpenAI Agents SDK
- MCP libraries
- LLM API access (Gemini)

### Installation Steps
1. Create project: `uv init agents-sdk-mcp`
2. Activate environment: `source .venv/bin/activate`
3. Configure Python interpreter in VS Code
4. Install dependencies: `uv add mcp uvicorn openai-agents prompt-toolkit`

### Environment Configuration
- Copy `.env.example` to `.env` (if provided)
- Configure LLM credentials and endpoints
- Set up MCP server URLs or command paths

## Learning Points for Students

### 1. MCP Protocol Understanding
- How servers expose tools and resources
- Client-server communication patterns
- Protocol versioning and compatibility

### 2. Agents SDK Integration
- Tool abstraction and conversion
- Agent lifecycle management
- Message state handling

### 3. CLI Application Design
- Rich terminal interfaces
- Auto-completion systems
- User experience considerations

### 4. Async Programming Patterns
- Context manager usage
- Resource cleanup strategies
- Concurrent client management

### 5. Software Architecture
- Separation of concerns
- Dependency injection patterns
- Extensible design principles

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

1. **Complete MCP Server**: Implement the TODO items for full document management
2. **Additional Tools**: Add more MCP servers with different capabilities
3. **Web Interface**: Build a web-based UI using the same backend
4. **Persistent Storage**: Replace in-memory storage with database
5. **Authentication**: Add user management and access control
6. **Monitoring**: Implement logging and metrics collection

This specification serves as both documentation and a learning guide for students exploring agent-tool integrations and MCP protocol implementations.