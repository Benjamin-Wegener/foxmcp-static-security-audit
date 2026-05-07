# FoxMCP Static Security Audit Report

Date: 2026-05-07
Scope: Static review of Firefox extension + Python MCP bridge in `/Users/user/dev/audits/foxmcp`
Method: Source inspection only (no runtime execution)

## Executive Summary
FoxMCP is functionally powerful but currently has a high-privilege trust model. The largest risks are command/script execution capability exposed via MCP, permissive extension scopes (`<all_urls>` + `webRequest`), and lack of connection-level authentication between local components.

Overall verdict: **CONDITIONAL / NEEDS HARDENING**.

## Findings

### 1) Overbroad extension permissions and all-site content injection
- Severity: High
- Evidence:
  - `extension/manifest.json` grants `webRequest` and `"<all_urls>"` permissions ([manifest.json](/Users/user/dev/audits/foxmcp/extension/manifest.json:9), [manifest.json](/Users/user/dev/audits/foxmcp/extension/manifest.json:17)).
  - Content script is injected on all URLs ([manifest.json](/Users/user/dev/audits/foxmcp/extension/manifest.json:25)).
- Risk:
  - Compromise of extension logic, local server trust boundary, or message path can impact browsing/session data across all sites.
- Recommendation:
  - Replace `"<all_urls>"` with least-privileged host allowlists where feasible.
  - Restrict content script matches to only required domains/features.

### 2) No explicit authentication of extension WebSocket peer
- Severity: High
- Evidence:
  - Any inbound WebSocket connection is accepted as the extension connection; there is no shared secret, token challenge, or origin validation in `handle_extension_connection` ([server.py](/Users/user/dev/audits/foxmcp/server/server.py:109), [server.py](/Users/user/dev/audits/foxmcp/server/server.py:127)).
- Risk:
  - A local adversarial process could attempt to impersonate the extension, inject responses, or manipulate request/response flow.
- Recommendation:
  - Add mutual authentication (startup nonce/token in env or file with strict perms).
  - Require handshake proof before promoting socket to `self.extension_connection`.

### 3) MCP tool can run external executables and execute generated JS in tab context
- Severity: High
- Evidence:
  - `content_execute_predefined` executes arbitrary executable files from `FOXMCP_EXT_SCRIPTS` via `subprocess.run(...)` ([mcp_tools.py](/Users/user/dev/audits/foxmcp/server/mcp_tools.py:1347)).
  - Script stdout is treated as JavaScript and executed in browser tabs ([mcp_tools.py](/Users/user/dev/audits/foxmcp/server/mcp_tools.py:1366)).
- Risk:
  - Any compromised/untrusted MCP client or weak local boundary can escalate to local command execution + browser script execution.
- Recommendation:
  - Disable by default behind explicit `--allow-predefined-scripts` gate.
  - Add allowlisted script manifest with hashes and per-script argument schema.
  - Add strong audit logging for every invocation.

## Positive Controls Observed
- Localhost-only host enforcement exists in startup path ([server.py](/Users/user/dev/audits/foxmcp/server/server.py:648)).
- Path traversal defenses exist for predefined script name/path resolution ([mcp_tools.py](/Users/user/dev/audits/foxmcp/server/mcp_tools.py:1304), [mcp_tools.py](/Users/user/dev/audits/foxmcp/server/mcp_tools.py:1317)).
- Script execution timeout present (`30s`) ([mcp_tools.py](/Users/user/dev/audits/foxmcp/server/mcp_tools.py:1351)).

## Priority Hardening Order
1. Add authenticated handshake between extension and server.
2. Gate/disable script execution tools by default.
3. Reduce extension permission scope and domain reach.
4. Add security logging and operator-visible warnings for privileged actions.
