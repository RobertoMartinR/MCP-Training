# MCP-Training

Practice project for the MCP (Model Context Protocol) course using:
- Gradio as an MCP server (SSE)
- A Hugging Face "Tiny Agents" client
- A local model (e.g., gpt-oss:20b) served by Ollama (OpenAI-compatible API)

## Structure

```
.
├─ 1_Simple_MCP.py
├─ package.json
├─ MCP_with_Gradio/
│  ├─ app.py                 # Gradio app exposing a tool (sentiment_analysis) and enabling MCP (SSE)
│  ├─ agent.json             # Tiny Agents config (Ollama model + Gradio SSE server)
│  ├─ requirements.txt
└─ README.md
```

## Requirements

- Windows with PowerShell
- Git and Python 3.10+ (recommended 3.11/3.12/3.13)
- Ollama installed and running (https://ollama.com/)
- Node.js only if you want to use `mcp-remote` (not needed with SSE)

## Set up the Python environment

From the repo root:

```powershell
# Create and activate venv
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# Upgrade pip
python -m pip install --upgrade pip

# Install Gradio subproject dependencies
pip install -r .\MCP_with_Gradio\requirements.txt

# Tiny Agents (CLI)

```

## Set up Ollama

1) Start Ollama and verify the API is available at `http://localhost:11434/v1`.
2) Pull or verify the model you will use (replace with yours if different):

```powershell
ollama pull llama3.1:8b
ollama list
```

If your model is not `llama3.1:8b`, edit `MCP_with_Gradio/agent.json` and update the `model` field.

## Run the MCP server with Gradio

From the subproject folder:

```powershell
cd .\MCP_with_Gradio
python .\app.py
```

You should see in the console something like:
- HTTP URL: http://127.0.0.1:7860/
- SSE URL:  http://127.0.0.1:7860/gradio_api/mcp/sse (or `/mcp/sse` on older versions)

Keep this process running.

## Configure the agent (Tiny Agents)

File: `MCP_with_Gradio/agent.json`

```json
{
  "model": "llama3.1:8b",
  "endpointUrl": "http://localhost:11434/v1",
  "servers": [
    {
      "type": "sse",
      "url": "http://127.0.0.1:7860/gradio_api/mcp/sse"
    }
  ]
}
```

Notes:
- If your Gradio shows `/mcp/sse` instead of `/gradio_api/mcp/sse`, change it here.
- If you prefer `stdio` with `mcp-remote`, use:

```json
{
  "type": "stdio",
  "command": "npx",
  "args": ["mcp-remote", "http://127.0.0.1:7860/gradio_api/mcp/sse"]
}
```

## Run the agent

In another terminal (keep Gradio running):

```powershell
cd C:\Proyectos\MCP\MCP_with_Gradio
# Use an absolute path so it doesn't look in the public tiny-agents dataset
tiny-agents run C:\Proyectos\MCP\MCP_with_Gradio\agent.json
```

When the CLI loads, it should list the `sentiment_analysis` tool.

## Test the tool via the agent

Ask something like:

> Analyze the sentiment of: "I love this product, it's fantastic".

The response will be a JSON with `polarity`, `subjectivity`, and `assessment`.

## SSE or stdio?

- SSE (recommended with Gradio): simpler, no extra Node processes, robust on Windows, and works the same if you move the server to another machine.
- stdio: useful when the server is a binary/CLI that already speaks MCP over stdin/stdout and you want to avoid open ports.
