# pi-zai-mcp-cn

Fork of [pi-zai-mcp](https://github.com/fitchmultz/pi-zai-mcp) with **Chinese domestic endpoint support** (open.bigmodel.cn) for GLM Coding Plan users, and unified API key with the zhipu-coding model provider.

Give pi agents Z.ai-powered web search, URL reading, repository reading, and vision tools through MCP without leaving a pi session. This is an unofficial community package, not an official Z.ai package.

**Default: overseas (api.z.ai).** Set `Z_AI_MCP_REGION=cn` to switch to domestic endpoints (open.bigmodel.cn).

## What you get

`pi-zai-mcp` registers pi tools that bridge Z.ai MCP servers:

- **Search the web** with Z.ai Web Search MCP.
- **Read URLs** and convert pages to model-friendly Markdown/text with Z.ai Web Reader MCP.
- **Inspect GitHub repositories** through Zread search, file reading, and directory-structure tools.
- **Analyze images and videos** through Z.ai vision tools for UI screenshots, OCR, error screenshots, diagrams, charts, image understanding, and video understanding.

Generic MCP tools are available immediately. Server-specific `z_ai_*` wrapper tools are registered after tool discovery, either by calling `z_ai_mcp_list_tools` or by setting `Z_AI_MCP_AUTO_DISCOVER=1`.

## Proof from a real discovery run

A live `z_ai_mcp_list_tools` run on 2026-05-11 reported these Z.ai MCP tools:

```text
search: web_search_prime
reader: webReader
zread: search_doc, read_file, get_repo_structure
vision: ui_to_artifact, extract_text_from_screenshot, diagnose_error_screenshot,
        understand_technical_diagram, analyze_data_visualization, ui_diff_check,
        analyze_image, analyze_video
```

The extension turns discovered tools into pi tool names with this pattern:

```text
z_ai_<server>_<mcp_tool_name>
```

Examples:

```text
z_ai_search_web_search_prime
z_ai_reader_webReader
z_ai_zread_search_doc
z_ai_zread_get_repo_structure
z_ai_zread_read_file
z_ai_vision_extract_text_from_screenshot
```

## Install

Install from npm:

```bash
pi install npm:pi-zai-mcp
```

Install from GitHub:

```bash
pi install https://github.com/fitchmultz/pi-zai-mcp
```

Compatibility note: this package is tested against the current pi release during each package update, and pi-bundled runtime packages are declared as optional wildcard peers. That keeps installs forward-open for future pi releases: npm peer ranges should not block users from trying a newer pi, though runtime behavior is only verified against the tested baseline until a follow-up package release confirms it.

Try it without installing permanently:

```bash
export Z_AI_API_KEY="your_z_ai_api_key"
pi -e npm:pi-zai-mcp
```

Run from a local clone:

```bash
git clone https://github.com/fitchmultz/pi-zai-mcp.git
cd pi-zai-mcp
npm install
export Z_AI_API_KEY="your_z_ai_api_key"
pi -e .
```

## Configure

| Variable | Required | Default | Purpose |
| --- | --- | --- | --- |
| `Z_AI_API_KEY` / `ZAI_API_KEY` / `ZHIPU_API_KEY` | Yes | none | API key. `ZHIPU_API_KEY` is checked first to reuse the zhipu-coding model provider key. |
| `Z_AI_MCP_REGION` | No | overseas | Set to `cn`, `china`, `zhipu`, or `domestic` to use domestic endpoints (open.bigmodel.cn + ZHIPU vision mode). |
| `Z_AI_MCP_BASE_URL` | No | auto | Override MCP HTTP base URL. Defaults to `https://api.z.ai/api` (overseas) or `https://open.bigmodel.cn/api` (when `Z_AI_MCP_REGION=cn`). |
| `Z_AI_MCP_SERVERS` | No | `all` | Comma-separated subset of `search,reader,zread,vision`. |
| `Z_AI_MCP_AUTO_DISCOVER` | No | off | Set to `1`, `true`, `yes`, or `on` to discover and register server-specific wrappers at extension startup. |
| `Z_AI_MCP_TIMEOUT_MS` | No | `30000` | Per-connection/tool-call timeout in milliseconds. |
| `Z_AI_MODE` | No | auto | Override vision MCP server mode. Defaults to `ZAI` (overseas) or `ZHIPU` (when `Z_AI_MCP_REGION=cn`). |

Example: disable vision server access for a lighter setup.

```bash
export Z_AI_MCP_SERVERS=search,reader,zread
```

## Use

Built-in generic tools:

- `z_ai_mcp_list_tools` — list configured Z.ai MCP tools and schemas; successful discovery also registers server-specific wrappers.
- `z_ai_mcp_call_tool` — call an exact MCP tool by server id and raw MCP tool name.

Typical flow:

1. Ask pi to call `z_ai_mcp_list_tools` for `search`, `reader`, `zread`, or `vision`.
2. Use a discovered wrapper such as `z_ai_search_web_search_prime`, or call the exact MCP tool through `z_ai_mcp_call_tool`.
3. Run `/zai-mcp-status` in interactive pi to inspect server connection, discovery, and wrapper registration status.

Large MCP outputs are truncated to pi's standard 50 KB / 2000 line limit. When truncation happens, the full output is saved to a temp file and the path is included in the tool result.

## How it works

- `search`, `reader`, and `zread` use Z.ai Streamable HTTP MCP endpoints.
- `vision` uses the `@z_ai/mcp-server` stdio server. The package depends on `@z_ai/mcp-server@0.1.4` and falls back to `npx -y @z_ai/mcp-server@0.1.4` only if the local binary is unavailable.
- The extension registers generic tools synchronously so pi startup is fast.
- Server connections and tool discovery are lazy by default to avoid blocking pi startup on network or package-manager work.
- `session_shutdown` closes any opened MCP transports.

## Security and data flow

- Pi extensions run with your local user permissions. Review code before installing any third-party pi package.
- The extension reads `Z_AI_API_KEY` or `ZAI_API_KEY` from the environment; it does not store credentials.
- HTTP MCP calls send the key as a Bearer token to Z.ai MCP endpoints.
- Vision calls start a local stdio MCP server and pass the key in that child process environment.
- Truncated full outputs are written under your OS temp directory, not this repo.

## Verify this repo

```bash
npm install
npm run typecheck
npm audit --omit=dev
npm publish --dry-run
```

For install-path checks, use a temporary project so local `.pi/settings.json` changes do not affect another repo:

```bash
tmpdir="$(mktemp -d)"
cd "$tmpdir"
pi install -l /path/to/pi-zai-mcp
```

## Current limits

- Requires a Z.ai API key and network access for real tool calls.
- Server-specific wrapper availability depends on live MCP discovery. Generic list/call tools remain available even when discovery fails.
- Upstream MCP schemas and tool names can change.
- Verification currently consists of TypeScript typechecking, npm audit, npm dry-run packing, and pi install smoke checks; there is no dedicated unit test suite yet.

## Project map

```text
extensions/zai-mcp.ts  # public pi package entrypoint
src/index.ts           # extension implementation
package.json           # npm + pi package manifest
CHANGELOG.md           # release notes
```
