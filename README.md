# Paper Search MCP

A Model Context Protocol (MCP) server for searching and downloading academic papers from multiple sources, including arXiv, PubMed, bioRxiv, and more. Designed for seamless integration with large language models like Claude Desktop. **Now with HTTP/SSE support for n8n, webhooks, and IoT clients!**

![PyPI](https://img.shields.io/pypi/v/paper-search-mcp.svg) ![License](https://img.shields.io/badge/license-MIT-blue.svg) ![Python](https://img.shields.io/badge/python-3.10+-blue.svg)
[![smithery badge](https://smithery.ai/badge/@openags/paper-search-mcp)](https://smithery.ai/server/@openags/paper-search-mcp)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Installation](#installation)
  - [Quick Start](#quick-start)
    - [Install Package](#install-package)
    - [Configure Claude Desktop](#configure-claude-desktop)
  - [HTTP Server Mode](#http-server-mode)
  - [For Development](#for-development)
- [HTTP API](#http-api)
- [Contributing](#contributing)
- [Demo](#demo)
- [License](#license)

---

## Overview

`paper-search-mcp` is a Python-based MCP server that enables users to search and download academic papers from various platforms. It provides tools for searching papers (e.g., `search_arxiv`) and downloading PDFs (e.g., `download_arxiv`), making it ideal for researchers and AI-driven workflows. Built with the MCP Python SDK, it integrates seamlessly with LLM clients like Claude Desktop.

---

## Features

- **Multi-Source Support**: Search and download papers from arXiv, PubMed, bioRxiv, medRxiv, Google Scholar, IACR ePrint Archive, Semantic Scholar, and CrossRef.
- **Dual Transport**: Supports both stdio (for Claude Desktop) and HTTP/SSE (for web apps, n8n, webhooks).
- **REST API**: Full HTTP REST API for searching, downloading, and reading papers.
- **Server-Sent Events (SSE)**: Real-time progress updates for long-running operations.
- **MCP over HTTP**: JSON-RPC 2.0 interface for MCP protocol compatibility.
- **Standardized Output**: Papers are returned in a consistent dictionary format via the `Paper` class.
- **Asynchronous Tools**: Efficiently handles network requests using `httpx`.
- **Extensible Design**: Easily add new academic platforms by extending the `academic_platforms` module.

---

## Installation

`paper-search-mcp` can be installed using `uv` or `pip`. Below are options for immediate use and development.

### Installing via Smithery

To install paper-search-mcp for Claude Desktop automatically via [Smithery](https://smithery.ai/server/@openags/paper-search-mcp):

```bash
npx -y @smithery/cli install @openags/paper-search-mcp --client claude
```

### Quick Start (stdio mode)

For users who want to quickly run the server with Claude Desktop:

1. **Install Package**:

   ```bash
   uv add paper-search-mcp
   ```

2. **Configure Claude Desktop**:
   Add this configuration to `~/Library/Application Support/Claude/claude_desktop_config.json` (Mac) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):
   ```json
   {
     "mcpServers": {
       "paper_search_server": {
         "command": "uv",
         "args": [
           "run",
           "--directory",
           "/path/to/your/paper-search-mcp",
           "-m",
           "paper_search_mcp.server"
         ],
         "env": {
           "SEMANTIC_SCHOLAR_API_KEY": "" 
         }
       }
     }
   }
   ```
   > Note: Replace `/path/to/your/paper-search-mcp` with your actual installation path.

### HTTP Server Mode

For integration with n8n, webhooks, web applications, or IoT clients:

1. **Start the HTTP Server**:

   ```bash
   # Using the start script
   ./start_server.sh http

   # Or using uvicorn directly
   uvicorn paper_search_mcp.server_http:app --host 0.0.0.0 --port 8090

   # Or using Python
   python -m paper_search_mcp.server_http
   ```

2. **Using Docker**:

   ```bash
   docker build -t paper-search-mcp .
   docker run -p 8090:8090 paper-search-mcp
   ```

3. **Environment Variables**:

   | Variable | Default | Description |
   |----------|---------|-------------|
   | `PAPER_SEARCH_HOST` | `0.0.0.0` | Host to bind to |
   | `PAPER_SEARCH_PORT` | `8090` | Port to bind to |
   | `PAPER_SEARCH_DEBUG` | `false` | Enable debug mode |
   | `SEMANTIC_SCHOLAR_API_KEY` | - | Optional API key |

### For Development

For developers who want to modify the code or contribute:

1. **Setup Environment**:

   ```bash
   # Install uv if not installed
   curl -LsSf https://astral.sh/uv/install.sh | sh

   # Clone repository
   git clone https://github.com/openags/paper-search-mcp.git
   cd paper-search-mcp

   # Create and activate virtual environment
   uv venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   ```

2. **Install Dependencies**:

   ```bash
   # Install project in editable mode
   uv pip install -e .

   # Run tests
   pytest tests/
   ```

---

## HTTP API

When running in HTTP mode, the following endpoints are available:

### Quick Reference

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Health check |
| `/api/platforms` | GET | List available platforms |
| `/api/search/{platform}` | POST/GET | Search papers |
| `/api/download/{platform}` | POST | Download PDF |
| `/api/read/{platform}` | POST | Extract text from PDF |
| `/api/paper/{platform}/{id}` | GET | Get paper details |
| `/sse` | GET | Server-Sent Events stream |
| `/mcp` | POST | MCP JSON-RPC endpoint |

### Example: Search Papers

```bash
curl -X POST http://localhost:8090/api/search/arxiv \
  -H "Content-Type: application/json" \
  -d '{"query": "machine learning", "max_results": 5}'
```

### Example: SSE Connection (JavaScript)

```javascript
const eventSource = new EventSource('http://localhost:8090/sse');
eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log(`Event: ${data.type}`, data.data);
};
```

### Example: MCP over HTTP

```bash
curl -X POST http://localhost:8090/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "search_arxiv",
      "arguments": {"query": "neural networks", "max_results": 5}
    }
  }'
```

📚 **Full API Documentation**: [docs/API.md](docs/API.md)

---

## Supported Platforms

| Platform | Search | Download | Read |
|----------|--------|----------|------|
| arXiv | ✅ | ✅ | ✅ |
| PubMed | ✅ | ❌ | ❌ |
| bioRxiv | ✅ | ✅ | ✅ |
| medRxiv | ✅ | ✅ | ✅ |
| Google Scholar | ✅ | ❌ | ❌ |
| IACR ePrint | ✅ | ✅ | ✅ |
| Semantic Scholar | ✅ | ✅ | ✅ |
| CrossRef | ✅ | ❌ | ❌ |

---

## Contributing

We welcome contributions! Here's how to get started:

1. **Fork the Repository**:
   Click "Fork" on GitHub.

2. **Clone and Set Up**:

   ```bash
   git clone https://github.com/yourusername/paper-search-mcp.git
   cd paper-search-mcp
   uv pip install -e .
   ```

3. **Make Changes**:

   - Add new platforms in `academic_platforms/`.
   - Update tests in `tests/`.

4. **Run Tests**:

   ```bash
   pytest tests/
   ```

5. **Submit a Pull Request**:
   Push changes and create a PR on GitHub.

---

## Demo

<img src="docs/images/demo.png" alt="Demo" width="800">

---

## License

This project is licensed under the MIT License. See the LICENSE file for details.

---

Happy researching with `paper-search-mcp`! If you encounter issues, open a GitHub issue.
