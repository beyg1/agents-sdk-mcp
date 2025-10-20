This is basically an imitation of Anthropic's MCP project course but setup with 
openai agents sdk so that we can use free gemini api.

Create Project with "uv init agents-sdk-mcp".
source .venv/bin/activate to create the virtual environment.
ctrl + shift + p and copy .venv path to python interpretor there.
uv add mcp uvicorn openai-agents prompt-toolkit to add packages and run the cli project.

run the mcp server by "uv run uvicorn mcp_server:mcp_app --reload" and then 
uv run main.py to run cli chat.

                    The Design:
main.py is responsible for the cli chatbot and runs by uv run main.py

mcp_server is the mcp server it must run parralel with chatbot so the agent can access the mcp server
it runs by uv run uvicorn mcp_server:mcp_app --reload
2 tools have been written and learnt in panaversity course 
1. read_docs
2. edit_docs

mcp_client is the client which connects the cli chatbot app with mcp server
you can check if it works by uv run mcp_client.py and it should show the tools of the MCP
2 tools have been written and learnt in panaversity course 
1. list_tools
2. call_tool

                     Since Tools are done now we will shift to Resources ->
mcp_server write a function for listing Resources and a function to read and return content of
a resource. It can have 2 types of content. text & binary (contain raw binary data encoded in base64. 
These are suitable for Images, PDFs, Audio files Video files and Other non-text formats).

mcp_client  write a function that can read a resource and return it's content after parsing.
For testing Postman is recommended. but in cli chat "@" should show the resources instead of using a toolcall


                     After Tools & Resources only Prompts remain to work on any MCP

                    