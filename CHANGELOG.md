# Changelog

All notable changes in this fork are documented here. This project is a fork of
[QGIS2OllamaMCP](https://github.com/anitagraser/qgis_mcp) by Anita Graser, which is
itself based on [jjsantos01/qgis_mcp](https://github.com/jjsantos01/qgis_mcp) and
[BlenderMCP](https://github.com/ahujasid/blender-mcp). Distributed under GPL v3.

## [1.1.0] — Hardened fork

### Security
- **Shared-secret token auth on the plugin socket (port 9876).** The QGIS plugin
  now generates a random token on first run, stored in `~/.qgis_mcp/token`, and
  rejects any command that does not present the matching token. The MCP server
  reads the same file and attaches the token to every command. This raises the
  bar against *other* local users and sandboxed processes that can reach the
  socket but cannot read your home directory; a process running as the same
  user can still read `~/.qgis_mcp/token` and present it, so it is not a full
  barrier. The token file is never committed (see `.gitignore`).
  - *Note:* the token only protects direct connections to port 9876. The MCP
    server is a trusted forwarder — anything it accepts on 9877 is forwarded with
    a valid token. The protection for the 9877 surface is the loopback bind below.
- **MCP server bound to `127.0.0.1` only (port 9877).** `mcp.run(...)` now passes
  `host="127.0.0.1"` explicitly, instead of relying on the FastMCP/uvicorn default
  (which could bind to `0.0.0.0` and expose code execution to the local network).

### Housekeeping
- Removed the unused `requests` and `lupa` from the documented install command.
- Removed the upstream author's personal email from `metadata.txt` (a fork should
  not list the original author as its contact); replaced with the maintainer's
  GitHub no-reply address. Original authorship is preserved in the attribution text.
- Updated `pyproject.toml`: version, description, explicit `GPL-3.0-or-later`
  license, and the runtime dependencies the code actually imports.

### Known limitations
- The hardening changes have **not been tested against a live QGIS install**;
  verify locally before relying on them.
- The `execute_code` tool runs arbitrary Python inside QGIS by design. The token
  and loopback bind reduce *who can reach* the tool; they do not sandbox *what it
  can do* once reached. Only run on a trusted machine, only while in use.
