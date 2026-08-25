# Jason Gilman — MCP servers, in more detail

**Talk notes · 2026-08-25 · Responsible Gen-AI for NASA Earthdata, Seattle**

Follows Joe Hamman's MCP overview — see
[2026-08-25-joe-hamman-mcp.md](2026-08-25-joe-hamman-mcp.md).

> Live notes — taken during the talk, not reviewed. Attributions are as-heard.

## Questions to listen for
Carried over from the Joe session and our D3 research:

- Is `nasa/earthdata-mcp` the server Aimee's resources point at?
- **The repo declares no license.** Can that be fixed?
- Where should fitness knowledge live — UMM-C/UMM-V fields, a STAC extension, or
  outside CMR entirely?
- `spatial_resolution` is free text with mixed units, and null for GPM_3IMERGHH. Is
  normalising or backfilling it on anyone's roadmap?
- The HLS `cloud_cover` quirk is hard-coded in the server's system prompt. Is there an
  intended structured home for that class of fact?
- Does the roadmap include a `role: example` style access-recipe asset (our D7)?

## Notes

