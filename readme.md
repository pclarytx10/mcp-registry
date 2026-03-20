# MCP Registry & FastMCP-HTTP
This repository is a combination of two complementary components.

MCP Registry is a server solution that manages and coordinates multiple MCP (Model Context Protocol) servers. It provides:
- Central registration for MCP servers
- Dynamic port allocation
- Health monitoring of registered servers
- Unified access to tools across all registered servers
- MCP Registry-compatible API (`GET /v0.1/servers`) with pagination and filtering

FastMCP-HTTP is a Python package that provides an HTTP REST client-server solution for MCP. It offers a unified interface for accessing tools, prompts and resources through HTTP endpoints.

# Components

## HTTP Server
The FastMCPHttpServer provides an HTTP server solution for MCP.

## HTTP Client
The FastMCPHttpClient offers both synchronous and asynchronous interfaces to interact with FastMCP servers.
It is extended to also function as a client to the MCP registry server.

## Registry Server
The MCP Registry Server acts as a central coordinator for multiple MCP servers. It handles server registration, health monitoring, and provides a unified interface to access tools across all connected servers.

### API Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /servers` | List registered servers (legacy format) |
| `GET /v0.1/servers` | MCP Registry-compatible paginated list with `cursor`, `limit`, `search`, `updated_since`, `version`, `include_deleted` |
| `POST /register_server` | Register a new MCP server |

### Known Servers

Add predefined servers via `known_servers.json` in the project root. The file is an array of [ServerResponse](https://registry.modelcontextprotocol.io/docs#/operations/list-servers-v0.1) objects. Example:

```json
[
  {
    "_meta": {
      "io.modelcontextprotocol.registry/official": {
        "status": "active",
        "isLatest": true,
        "publishedAt": "2025-01-01T00:00:00Z",
        "updatedAt": "2025-01-01T00:00:00Z",
        "statusChangedAt": "2025-01-01T00:00:00Z"
      }
    },
    "server": {
      "name": "io.example/my-server",
      "description": "Example MCP server",
      "version": "1.0.0",
      "title": "My Server"
    }
  }
]
```

Known servers are merged with dynamically registered servers when serving `GET /v0.1/servers`. If the file is missing, an empty list is used.

## MCP Explorer
The MCP Explorer provides a graphical user interface for interacting with MCP servers and their tools.

# Installation

1. Clone the repository
2. Install the dependencies:
```bash
pip install -r requirements.txt
```

# Examples

## Using the registry server

### FastMCPHttpServer

```python
from fastmcp_http.server import FastMCPHttpServer

mcp = FastMCPHttpServer("MyServer", description="My MCP Server")

@mcp.tool()
def my_tool(text: str) -> str:
    return f"Processed: {text}"

if __name__ == "__main__":
    mcp.run_http()
```

### FastMCPHttpClient

```python
from fastmcp_http.client import FastMCPHttpClient


def main():
    # Connect to the registry server
    client = FastMCPHttpClient("http://127.0.0.1:31337")

    servers = client.list_servers()
    print(servers)

    tools = client.list_tools()
    print(tools)

    result = client.call_tool("my_tool", {"text": "Hello, World!"})
    print(result)


if __name__ == "__main__":
    main()
```

## Standalone

### FastMCPHttpServer

```python
from fastmcp_http.server import FastMCPHttpServer

mcp = FastMCPHttpServer("MyServer", description="My MCP Server")

@mcp.tool()
def my_tool(text: str) -> str:
    return f"Processed: {text}"

if __name__ == "__main__":
    mcp.run_http(register_server=False, port=15151)
```

### FastMCPHttpClient

```python
from fastmcp_http.client import FastMCPHttpClient


def main():
    client = FastMCPHttpClient("http://127.0.0.1:15151")

    tools = client.list_tools()
    print(tools)

    result = client.call_tool("my_tool", {"text": "Hello, World!"})
    print(result)


if __name__ == "__main__":
    main()
```

## Usage

1. (Optional) Add known servers to `known_servers.json`
2. Start the MCP Registry (`start_registry_server.py`)
3. Start an MCP server (and verify that it is properly registered in the registry)
4. Start a client and connect to the registry url


# License
MIT License