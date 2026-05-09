# ArkComposer MCP Bootstrap Guide

This file is a short bootstrap note for Codex or other LLM clients working with a public ArkComposer installation.

The live ArkComposer MCP server is the source of truth. After connecting, prefer the prompt, resources, and tool schemas exposed by the running app over this packaged file.

## Public Links

- Website: [www.arkcomposer.com](http://www.arkcomposer.com)
- GitHub: [github.com/arkark2010arkark/ArkComposer](https://github.com/arkark2010arkark/ArkComposer)

## Connection

Start ArkComposer normally, open the left palette, and start `MCP Server`.

Default endpoints:

- TCP JSON-RPC: `127.0.0.1:9100`
- HTTP/SSE: `127.0.0.1:9101/sse`

If the ArkComposer MCP Server dialog shows different ports, use those ports.

## Mandatory Startup Flow

After connecting:

1. Send `initialize`.
2. Send `notifications/initialized`.
3. Call `prompts/list`.
4. Call `prompts/get` with `name: "arkcomposer.system"`.
5. Call `resources/list`.
6. Read the built-in docs with `resources/read`.
7. Call `tools/list` and follow the live `inputSchema`.
8. Call `get_score` with `summary_only: true` before editing.

Useful resources:

- `arkcomposer://docs/how-to-use-llm`
- `arkcomposer://docs/score-model`
- `arkcomposer://docs/tool-workflows`

Bundled fallback guide:

- `docs/HowToUse_ArkCompoerForLLM.md`

The `Compoer` spelling is historical and kept for compatibility.

## Tool Contract Reminders

- Every `tools/call.params.arguments` value must be a JSON object.
- Never send a bare array as the top-level `arguments` value.
- General tools use full-name object fields.
- Compact reads use `get_score_compact` or `get_score_range_compact`.
- Compact writes use `add_notes_compact`; arrays live inside `arguments.notes` and `arguments.clear`.
- Read compact legends use `note_fields` and `seg_fields`.
- Compact write legends use `row_fields` and `clear_row_fields`.

## Safe Editing Rule

Always read the live score first, edit only the requested target range, then verify the changed range.

Trust the live MCP response over cached notes, old screenshots, or this packaged bootstrap file.
