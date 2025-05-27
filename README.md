# QV MCP Client

A Python client library for the Model Context Protocol (MCP) that provides clean, async connectivity to MCP servers.

[![Python](https://img.shields.io/badge/python-3.10+-blue.svg)](https://python.org)
[![MCP](https://img.shields.io/badge/MCP-compatible-green.svg)](https://modelcontextprotocol.io)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## What is MCP?

The Model Context Protocol (MCP) is an open standard that enables AI assistants to securely connect to external data sources and tools. It provides a standardized way for AI models to interact with various systems including databases, file systems, APIs, and other services.

## Features

- **Async Support**: Full async/await pattern for non-blocking operations
- **Multiple Transports**: Support for stdio and HTTP-based connections
- **Connection Management**: Built-in manager for handling multiple server connections
- **Cross-Platform**: Windows-specific command handling included
- **Error Handling**: Comprehensive exception hierarchy with detailed error information
- **Type Safety**: Complete type hints throughout the codebase
- **Modular Design**: Clean separation of concerns for easy maintenance

## Installation

```bash
pip install mcp
pip install -e .
```

## Quick Start

### Basic Client Usage

```python
import asyncio
from qv_mcp_client import MCPClient, MCPServerConfig, MCPTransportType

async def main():
    config = MCPServerConfig(
        name="filesystem",
        transport=MCPTransportType.STDIO,
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
    )
    
    client = MCPClient()
    await client.connect(config)
    
    # List available tools
    tools = await client.list_tools()
    print(f"Available tools: {[tool['name'] for tool in tools]}")
    
    # Call a tool
    result = await client.call_tool("read_file", {"path": "/tmp/test.txt"})
    print(f"Result: {result}")
    
    await client.disconnect()

asyncio.run(main())
```

### Managing Multiple Connections

```python
from qv_mcp_client import MCPManager

async def main():
    manager = MCPManager()
    
    # Add multiple servers
    await manager.add_server(MCPServerConfig(
        name="filesystem",
        transport=MCPTransportType.STDIO,
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
    ))
    
    await manager.add_server(MCPServerConfig(
        name="api-server",
        transport=MCPTransportType.STREAMABLE_HTTP,
        url="http://localhost:3000/mcp"
    ))
    
    # Use specific client
    fs_client = manager.get_client("filesystem")
    if fs_client:
        tools = await fs_client.list_tools()
        print(f"Filesystem tools: {len(tools)}")
    
    await manager.disconnect_all()
```

## Transport Types

### STDIO Transport
For local subprocess-based servers (Python, Node.js, etc.)

```python
config = MCPServerConfig(
    name="python-server",
    transport=MCPTransportType.STDIO,
    command="python",
    args=["-m", "my_mcp_server"],
    env={"DEBUG": "1"}
)
```

### HTTP Transport
For remote or web-based MCP servers

```python
config = MCPServerConfig(
    name="web-server",
    transport=MCPTransportType.STREAMABLE_HTTP,
    url="https://api.example.com/mcp"
)
```

## API Overview

### MCPClient Methods

**Connection:**
- `connect(config)` - Connect to MCP server
- `disconnect()` - Disconnect from server

**Tools:**
- `list_tools()` - Get available tools
- `call_tool(name, arguments)` - Execute a tool

**Resources:**
- `list_resources()` - Get available resources
- `read_resource(uri)` - Read resource content

**Prompts:**
- `list_prompts()` - Get available prompts
- `get_prompt(name, arguments)` - Retrieve formatted prompt

### MCPManager Methods

- `add_server(config)` - Add and connect to server
- `remove_server(name)` - Remove server connection
- `get_client(name)` - Get specific client instance
- `list_servers()` - List connected servers
- `disconnect_all()` - Close all connections

## Error Handling

The library provides specific exception types for different error scenarios:

```python
from qv_mcp_client.exceptions import MCPConnectionError, MCPToolError

try:
    await client.connect(config)
    result = await client.call_tool("example_tool", {})
except MCPConnectionError as e:
    print(f"Connection failed: {e}")
except MCPToolError as e:
    print(f"Tool execution failed: {e}")
```

## Platform Support

- **Windows**: Automatic handling of `.cmd` and `.bat` files
- **Linux/macOS**: Direct command execution
- **Python 3.10+**: Full compatibility

## Project Structure

```
src/qv_mcp_client/
├── __init__.py          # Package exports
├── client.py            # Main MCPClient implementation
├── manager.py           # Multi-connection manager
├── config.py            # Configuration classes
├── utils.py             # Platform utilities
└── exceptions.py        # Custom exceptions
```

## Examples

The `examples/` directory contains working examples for different use cases:

- Basic client usage with filesystem server
- Multi-server management
- Error handling patterns

## Requirements

- Python 3.10 or higher
- `mcp` package (Model Context Protocol SDK)
- Platform-specific requirements for server execution

## MCP Server Compatibility

This client works with any MCP-compliant server including:

- Official MCP servers (filesystem, database, etc.)
- Custom Python MCP servers
- Docker-based MCP services
- HTTP/REST MCP endpoints

## Development Status

This is a personal project developed for specific use cases. The code is provided as-is and will be maintained and improved based on personal requirements. While the library is functional and well-tested in its intended environments, it may not cover all edge cases or use scenarios.

## License

MIT License - see LICENSE file for details.

## Related Links

- [Model Context Protocol Specification](https://modelcontextprotocol.io)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)
