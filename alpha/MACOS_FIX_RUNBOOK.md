# Living Memory — macOS/Hermes Fix Runbook (2026-08-23)

Complete record of everything fixed to get the living-memory MCP server
running on macOS (Apple Silicon, Python 3.14 system / 3.11 Hermes venv) and
operating it end-to-end. Use this as the pattern checklist for any new
version of the living-memory system.

## 1. What was broken

| # | Symptom | Root cause |
|---|---------|-----------|
| 1 | MCP server "wouldn't launch" | `.augment/mcp.json` used `${workspaceFolder}` — a VS Code/Augment-only variable. Every other launcher passes it literally → `can't open file '.../${workspaceFolder}/...'`. The server code itself was NEVER broken. |
| 2 | No MCP in Hermes | Hermes had no servers configured at all (`hermes mcp list` → none). |
| 3 | `begin_turn` FAILED, turn INTERRUPTED, assessment RED "resolve living-process identity conflict" | Probe invented NEW identity IDs (`chat_id`, `session_id`, `living_process_id`). Durable identity on disk (`.living-memory/data/continuity_recovery/latest_identity.json`) came from Augment: `living_process_id="living-systems-architecture"` + augment chat/session. SKILL.md rule: new chat/host is NOT a new living process when durable identity matches — so adopt the stored lineage. |
| 4 | `record_response` refused twice | (a) `in_response_to` must equal active turn's `user_event_id` (read from `data/continuity_recovery/active_turn.json`); (b) `host_event_id`/`execution_id`/`transition_id` must be IDENTICAL to those used in the same turn's `begin_turn`. |
| 5 | Recall output looked empty/useless | Results live in `results` string AFTER the `RECALL RESULTS` marker; lifecycle preamble dominates raw JSON. Parse after the marker. |

## 2. Fix sequence (the pattern)

1. **Test the server directly before touching config**:
   `printf '{"jsonrpc":"2.0","id":1,"method":"initialize",...}' | python3 scripts/mcp_server.py`
   If this answers, code is fine → problem is launcher config.
2. **Never use `${workspaceFolder}`** outside VS Code/Augment. Use absolute paths.
3. **Register in Hermes**:
   `hermes mcp add living-memory --command python3 --args <ABSOLUTE path>/scripts/mcp_server.py`
   (pipe `yes` into it for the interactive enable prompt). Restart session to load tools.
4. **Identity**: always reuse durable lineage:
   ```json
   {"chat_id": "augment-chat-living-memory-20260822",
    "session_id": "augment-session-20260822",
    "living_process_id": "living-systems-architecture"}
   ```
5. **Turn cycle**: begin_turn (unique host_event_id/execution_id/transition_id)
   → poll get_receipt until SUCCEEDED → read active_turn.json user_event_id
   → record_response with SAME three IDs + in_response_to=user_event_id
   → poll receipt → turn COMPLETED.
6. **Recall**: read-only, no turn needed; parse after `RECALL RESULTS`.

## 3. Artifacts created

- **Hermes MCP registration**: `~/.hermes/config.yaml` → server `living-memory`,
  command `python3`, args `<repo>/.living-memory/scripts/mcp_server.py`, all 5 tools enabled.
- **Hermes skill**: `~/.hermes/skills/note-taking/living-memory-session/SKILL.md`
  ("Use when starting a session in the living-memory repo") — full protocol,
  identity lineage, gates, recall parsing.
- **Augment config**: left as-is (its `${workspaceFolder}` is correct there).

## 4. Universal-portability notes (for Luna or next version)

- Path handling is the only OS-specific bit: derive paths from
  `Path(__file__).parent` (server already does this correctly).
- Config variable expansion differs per host (VS Code vs Claude vs Hermes) —
  ship per-host config snippets or plain absolute paths.
- Identity must persist ACROSS hosts: `latest_identity.json` is the source of
  truth; every client must read+adopt it instead of minting IDs.
- Gates are features, not bugs: RED/NEEDS_REVIEW, lineage mismatch, and
  stale-turn refusals are the fail-closed design working. Never bypass;
  reconcile instead.
