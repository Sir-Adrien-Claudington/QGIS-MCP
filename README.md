# QGIS MCP (hardened fork)

QGIS MCP connects [QGIS](https://qgis.org/) to an LLM through the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/docs/getting-started/intro), allowing agentic interaction with and control of QGIS. This enables prompt-assisted project creation, layer loading, code execution and more.

## Attribution & lineage

This is a **hardened fork**, distributed under **GPL v3** (see `LICENSE`). Credit to the chain it builds on:

- [BlenderMCP](https://github.com/ahujasid/blender-mcp/tree/main) by [Siddharth Ahuja](https://x.com/sidahuj) — the original concept.
- [QGIS MCP](https://github.com/jjsantos01/qgis_mcp) by [Juan Santos Ochoa (jjsantos01)](https://github.com/jjsantos01) — the QGIS adaptation this is based on.
- [QGIS2OllamaMCP](https://github.com/anitagraser/qgis_mcp) by [Anita Graser](https://github.com/anitagraser) — the immediate upstream of this fork.

### What this fork changes

See `CHANGELOG.md` for details. In short, two security hardening changes (both **untested against a live QGIS** — verify locally before relying on them):

1. **Shared-secret token** on the plugin socket (port 9876). The plugin generates a random token in `~/.qgis_mcp/token` on first run; the MCP server reads the same file and must present the token, so other local processes can't drive QGIS just by reaching the socket.
2. **Loopback-only MCP server.** The MCP server (port 9877) now binds explicitly to `127.0.0.1`, so it is never exposed to your network regardless of library defaults.

> ⚠️ **Security note:** by design this tool lets the LLM run **arbitrary Python inside QGIS** (the `execute_code` tool). Only run the plugin/server while you are actively using it, and only on a machine you trust. After starting, confirm port 9877 is on `127.0.0.1` with `netstat -ano | findstr 9877`.
>
> The token check is **fail-open**: if the plugin cannot read or write `~/.qgis_mcp/token`, it logs a warning and runs without authentication rather than refusing to start. Watch the QGIS log to confirm the token loaded.


## Components

The system consists of two main components:

1. **[QGIS plugin](/qgis_mcp_hardened/)**: A QGIS plugin that creates a socket server within QGIS to receive and execute commands.
2. **[MCP Server](/src/qgis_mcp/qgis_mcp_server.py)**: A Python server that implements the Model Context Protocol and connects to the QGIS plugin.

## Installation

### Prerequisites

- QGIS 3.X (tested with 3.38)
- Ollama and model with support for agents (tested with ministral-3:latest)
- For the MCP server Python environment: `pip install fastmcp ollama` (the upstream `requests` and `lupa` are not used by this code and are omitted)

### QGIS plugin

To install the plugin, you need to copy the folder [qgis_mcp_hardened](/qgis_mcp_hardened/) and its content into your QGIS plugins folder. The folder is named `qgis_mcp_hardened` (not `qgis_mcp_plugin`) on purpose, so it does not collide with the official `qgis_mcp_plugin` on plugins.qgis.org.


## Usage

### Starting the connection in QGIS

1. In QGIS, go to `plugins` > `QGIS2OllamaMCP` > `QGIS2OllamaMCP`
    ![plugins menu](/assets/imgs/qgis-plugins-menu.png)
2. Click "Start Server"
    ![start server](/assets/imgs/qgis-mcp-start-server.png)
    Will launch on localhost:9876 by default.
3. Check the logs to see the connection status 
    ![alt text](/assets/imgs/qgis-log.png)

### Starting the MCP server

`python src/qgis_mcp/qgis_mcp_server.py `

Will launch on localhost:9877/sse by default (bound to `127.0.0.1`).

#### Tools

- `ping` - Simple ping command to check server connectivity
- `get_qgis_info` - Get QGIS information about the current installation
- `load_project` - Load a QGIS project from the specified path
- `create_new_project` - Create a new project and save it
- `get_project_info` - Get current project information
- `add_vector_layer` - Add a vector layer to the project
- `add_raster_layer` - Add a raster layer to the project
- `get_layers` - Retrieve all layers in the current project
- `remove_layer` - Remove a layer from the project by its ID
- `zoom_to_layer` - Zoom to the extent of a specified layer
- `get_layer_features` - Retrieve features from a vector layer with an optional limit
- `execute_processing` - Execute a processing algorithm with the given parameters
- `save_project` - Save the current project to the given path
- `render_map` - Render the current map view to an image file
- `execute_code` - Execute arbitrary PyQGIS code provided as a string

### Starting the conversation

You can launch an example conversation using: 

`python src/qgis_mcp/qgis_mcp_client.py`

Note: you need to update the project file path to point to a project file on your machine. 

Then launch the interactive client to start experimenting: 

`python src/qgis_mcp/qgis_mcp_client_interactive.py`
