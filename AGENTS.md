# ArkComposer + Codex MCP Guide

This file is a generic operating guide for any Codex session working with a public ArkComposer installation.

## Public Links

- Website: [www.arkcomposer.com](http://www.arkcomposer.com)
- GitHub: [github.com/arkark2010arkark/ArkComposer](https://github.com/arkark2010arkark/ArkComposer)

## Goal

Connect Codex to ArkComposer quickly and reliably, without re-discovering the MCP setup every session.

## Recommended Connection Strategy

- Prefer live MCP access over direct file inspection whenever ArkComposer is already open.
- For Codex, prefer ArkComposer TCP MCP mode on `localhost`. It is simple, fast, and does not depend on a preconfigured desktop connector.
- Use stdio mode only when the MCP client can launch `ArkComposer.exe --mcp-mode` directly.

## How A User Starts ArkComposer MCP

1. Launch ArkComposer normally.
2. Open the left palette.
3. Click `MCP Server`.
4. Start the server.
5. Use the port values shown by ArkComposer.

Default ports in the app UI:

- TCP raw JSON-RPC: `127.0.0.1:9100`
- HTTP/SSE: `127.0.0.1:9101`

If the user changed the ports, trust the values currently shown in ArkComposer.

## Protocol Facts You Should Not Re-Learn Every Time

- ArkComposer TCP MCP uses newline-delimited JSON-RPC 2.0.
- Send `initialize` first and read exactly one response.
- Then send `notifications/initialized`. That notification has no response.
- After that, use:
  - `tools/list`
  - `tools/call`
  - `prompts/list`
  - `prompts/get`
  - `resources/list`
  - `resources/read`
- `tools/call` results often place the actual payload in `result.content[0].text` as JSON text. Parse that string again.

## Fast Path For Codex

If Codex already has a working ArkComposer MCP connector, use it.

If no connector is preconfigured, connect directly to the TCP server on `127.0.0.1:9100` and proceed immediately. Do not spend time rediscovering the protocol.

## Minimal PowerShell Client

```powershell
$client = [System.Net.Sockets.TcpClient]::new('127.0.0.1', 9100)
$stream = $client.GetStream()
$writer = New-Object System.IO.StreamWriter($stream)
$writer.NewLine = "`n"
$writer.AutoFlush = $true
$reader = New-Object System.IO.StreamReader($stream)

function Send-ArkJson($obj) {
  $json = $obj | ConvertTo-Json -Compress -Depth 20
  $writer.WriteLine($json)
}

function Read-ArkJson() {
  return ($reader.ReadLine() | ConvertFrom-Json -Depth 20)
}

Send-ArkJson @{
  jsonrpc = '2.0'
  id = 1
  method = 'initialize'
  params = @{
    protocolVersion = '2024-11-05'
    capabilities = @{}
    clientInfo = @{ name = 'codex'; version = '1.0' }
  }
}
$null = Read-ArkJson

Send-ArkJson @{
  jsonrpc = '2.0'
  method = 'notifications/initialized'
  params = @{}
}

Send-ArkJson @{
  jsonrpc = '2.0'
  id = 2
  method = 'tools/call'
  params = @{
    name = 'get_score'
    arguments = @{ summary_only = $true }
  }
}

$resp = Read-ArkJson
$score = $resp.result.content[0].text | ConvertFrom-Json
$score
```

## Mandatory First Steps In Every Live Session

1. Call `get_score` with `summary_only=true`.
2. Verify the live title, track count, and measure counts.
3. Use `get_score_range` only for the tracks and measures you actually need.
4. Edit only after reading the current score state.

Never assume:

- the score is empty
- the current song is the one you expect
- track `0` is free
- the first four measures should be overwritten
- the current key, tempo, or meter without reading them first

## Safe Editing Rules

- Prefer `add_notes_batch` over repeated `add_note` calls.
- Use `change_note` for targeted note edits instead of delete-and-recreate when possible.
- ArkComposer may automatically keep two extra measures at the end of the song to support composition flow and user convenience.
- Treat those extra two measures as normal built-in behavior, not as an error.
- Do not remove or “fix” those extra trailing measures unless the user explicitly asks for that change.
- If you insert measures, remember:
  - `add_measure.position` is `1`-based
  - it inserts before that position
- If mode matters, remember:
  - `add_measure` can set `key_fifths`
  - `add_measure` does not set `key_mode`
  - after insertion, use `set_measure_props` if you need to guarantee major/minor mode
- Verify after edits with `get_score_range` or `get_score(summary_only=true)`.

## Common Workflow Patterns

### Inspect A Song

1. `get_score(summary_only=true)`
2. `get_score_range(track_index, from_measure, to_measure)`

### Add Intro / Bridge / Ending

1. Inspect the current opening or target section first.
2. Insert measures only if needed.
3. Set measure properties if key, mode, tempo, or meter must match exactly.
4. Use `add_notes_batch`.
5. Verify the inserted passage and the join point into the existing music.

### Rewrite A Passage

1. Inspect the exact measures first.
2. Use `add_notes_batch` with `clear_measures` when replacing a whole passage.
3. Verify immediately.

### Add A New Track

1. Read the current score summary first.
2. Call `add_track`.
3. Name the track clearly.
4. Compose with `add_notes_batch`.

## Useful ArkComposer MCP Resources

If resources are exposed through the connected client, prefer the built-in docs instead of guessing:

- `arkcomposer://docs/how-to-use-llm`
- `arkcomposer://docs/score-model`
- `arkcomposer://docs/tool-workflows`

Local builds may also include a bundled guide named:

- `docs/HowToUse_ArkCompoerForLLM.md`

Note: the bundled filename may contain the `Compoer` typo. If present, use it as-is.

## Troubleshooting

### No Response On Port 9100

- Make sure ArkComposer is running.
- Make sure the MCP server was actually started from the ArkComposer UI.
- Confirm the active TCP port in the ArkComposer MCP Server dialog.
- If a previous client is stuck, fully close ArkComposer and restart it.

### The Client Connects But Nothing Happens

- Confirm you sent `initialize`.
- Confirm you then sent `notifications/initialized`.
- Confirm you are using newline-delimited JSON messages on TCP.

### HTTP/SSE Is Needed Instead Of Raw TCP

ArkComposer also exposes HTTP/SSE by default on `127.0.0.1:9101`.

The flow is:

1. `GET /sse`
2. Read the `endpoint` event
3. `POST` JSON-RPC requests to `/message?sessionId=...`

Use TCP first unless the client specifically requires HTTP/SSE.

### The Score Looks Different From Earlier Notes

Trust the live ArkComposer result, not old notes, screenshots, or cached summaries.

The source of truth is always the current response from:

- `get_score(summary_only=true)`
- `get_score_range(...)`

Also note:

- ArkComposer may automatically append two trailing measures as part of its normal composition workflow.
- This is intentional behavior for user convenience.
- It is not an MCP error and should not be “corrected” unless the user explicitly requests it.

## Practical Default

When working with ArkComposer from Codex:

1. Connect to the live MCP server.
2. Read the score summary.
3. Read only the target range.
4. Make the smallest correct edit.
5. Verify immediately.

This is the default operating procedure for any public ArkComposer installation.
