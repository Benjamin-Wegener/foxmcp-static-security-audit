# FoxMCP Findings Verification (Static)

Date: 2026-05-07
Target: `/Users/user/dev/audits/foxmcp-static-security-audit`
Method: Source-only verification

## Verification Matrix

| Finding | Status | Evidence |
|---|---|---|
| Overbroad extension scope (`<all_urls>`, `webRequest`, content script on all pages) | CONFIRMED | `extension/manifest.json:16-17,25` |
| Missing connection authentication between extension and local WebSocket server | CONFIRMED | `server/server.py:109-127` (connection accepted without auth challenge) |
| Script execution chain (`subprocess.run` -> JS execution in tab context) | CONFIRMED | `server/mcp_tools.py:1285-1398` |
| Localhost binding hardening is present | CONFIRMED (mitigation) | `server/server.py:648-651` |
| Path traversal protections for predefined scripts | CONFIRMED (mitigation) | `server/mcp_tools.py:1304-1318` |

## Conclusion

FoxMCP findings are reproducible and current for this repository snapshot. The report `audit_report_foxmcp_2026-05-07.md` is suitable for upload.
