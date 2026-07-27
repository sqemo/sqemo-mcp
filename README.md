# sqemo-mcp

**MCP server for [Sqemo](https://sqemo.com)** — design ERDs with your team's naming standards.

AI agents can query and edit entities, relationships, and domains; generate physical names
from a shared team glossary; import/export SQL (7 dialects) and DBML; and compare the model
against a live database. Works with both local `.erd.json` files and ERDs stored on the
Sqemo cloud.

- npm: [`sqemo-mcp`](https://www.npmjs.com/package/sqemo-mcp)
- Official MCP Registry: `io.github.sqemo/sqemo`
- Web app: [app.sqemo.com](https://app.sqemo.com)

> Requires Node.js >= 22. Local-file tools work without any account or configuration.
> Login is needed for cloud ERD tools, and for the tools marked (Pro) below.

## Installation

### Claude Code (`.mcp.json`)

```json
{
  "mcpServers": {
    "sqemo": { "command": "npx", "args": ["-y", "sqemo-mcp"] }
  }
}
```

### Claude Desktop

Add the same `mcpServers` entry to your config file:

- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`

### Cursor (`.cursor/mcp.json`)

```json
{
  "mcpServers": {
    "sqemo": { "command": "npx", "args": ["-y", "sqemo-mcp"] }
  }
}
```

## Login (cloud ERDs and Pro tools)

```bash
npx sqemo-mcp login    # email + password → stores a refresh token
npx sqemo-mcp logout   # removes stored credentials
```

Credentials are stored in `~/.erdmaker/credentials.json` (mode 0600 on POSIX);
your password is never persisted.

## What you can do

36 tools in total.

### Read (17 tools)

| Tool | Description |
|------|-------------|
| `list_erds` / `list_workspaces` | Cloud ERDs and workspaces you belong to (login required) |
| `get_erd_overview` | Name, dialect, entity/relationship/domain/glossary stats |
| `list_entities` / `get_entity` | Entity list and full detail (attributes, keys, logical/physical mapping) |
| `list_relationships` | Relationships with endpoints and cardinality |
| `list_domains` | Domain definitions (also from workspace standard glossaries) |
| `search_dictionary` | Search the team glossary (logical/physical words, abbreviations, synonyms) |
| `check_naming` | Check a logical name against the team naming standard |
| `generate_physical_name` | Logical name → physical name via glossary + naming rules |
| `export_sql` | CREATE TABLE SQL — mysql, postgres, cubrid, oracle, sqlserver, sqlite, h2 |
| `export_dbml` | DBML text |
| `validate_erd` / `lint_erd` | Structural validation and full lint (naming drift, referential integrity, duplicates) |
| `diff_erds` | Diff two sources (files, cloud ERDs, or raw SQL/DBML text) — dry-run before imports |
| `export_alter_sql` | Migration (ALTER) script from the physical diff against a baseline — renames stay renames via stable IDs, destructive changes come commented out (Pro) |
| `list_proposals` | Glossary proposal queue status (login required) |

### Live database (2 tools)

Read-only against your own database. Both query only the information schema —
never table data — and the connection URL is used by this local process only,
never sent to Sqemo servers.

| Tool | Description |
|------|-------------|
| `introspect_db` | Import a live PostgreSQL/MySQL schema into an existing ERD (Pro) |
| `check_db_drift` | Check a live database or a schema dump against the ERD's physical model — missing/extra tables and columns, PK/FK/NOT NULL mismatches (Pro) |

### Write (17 tools)

| Tool | Description |
|------|-------------|
| `create_erd` | New ERD from scratch or from SQL/DBML text — to a file or the cloud |
| `upsert_entity` / `delete_entity` | Entity editing with automatic physical-name derivation |
| `upsert_attribute` / `delete_attribute` | Attribute editing — PK rules and FK propagation handled automatically |
| `upsert_relationship` / `delete_relationship` | Relationship editing with automatic FK derivation |
| `upsert_domain` / `delete_domain` | Domain definition editing |
| `upsert_dictionary_word` / `delete_dictionary_word` | Glossary editing (standard-linked glossaries are protected) |
| `update_naming_rules` | Naming rule editing (delimiter, case, unknown-word handling) |
| `import_sql` / `import_dbml` | Replace an ERD from parsed SQL/DBML (IDs preserved) |
| `auto_layout` | Automatic entity/table layout (dagre) |
| `propose_dictionary_word` / `withdraw_proposal` | Propose new glossary words for owner approval |

Cloud writes require owner or shared-editor permission and are protected by
version CAS with 3-way auto-merge for concurrent edits.

## CLI for CI pipelines

Offline, file-based subcommands (no login needed):

```bash
# Naming-standard check — exits 1 on violations, great as a CI gate
npx sqemo-mcp lint schema.erd.json

# Schema export to stdout
npx sqemo-mcp export schema.erd.json --format sql --dialect postgres > schema.sql
npx sqemo-mcp export schema.erd.json --format dbml > schema.dbml
```

Drift mode compares the model against a real database or a dump, and exits 1 when
they disagree (Pro, requires login):

```bash
npx sqemo-mcp lint schema.erd.json --db "$DATABASE_URL" [--db-schema public] [--strict]
npx sqemo-mcp lint schema.erd.json --schema dump.sql --dialect postgres
npx sqemo-mcp lint --erd <cloud-erd-id> --db "$DATABASE_URL" --ignore 'tmp_*'
```

GitHub Actions example:

```yaml
- run: npx sqemo-mcp lint schema.erd.json
- run: npx sqemo-mcp lint schema.erd.json --db "${{ secrets.DATABASE_URL }}"
```

## Environment variables

| Variable | Purpose |
|----------|---------|
| `ERDMAKER_HOME` | Override the credentials directory (default `~/.erdmaker`) |
| `ERDMAKER_SUPABASE_URL` | Override the API URL (defaults to the Sqemo cloud) |
| `ERDMAKER_SUPABASE_ANON_KEY` | Override the API publishable key |
| `ERDMAKER_MAX_REQUESTS_PER_MINUTE` | Per-minute request cap (default `120`, `0` disables) |
| `ERDMAKER_MAX_REQUESTS_PER_DAY` | Daily request cap (default `10000`, `0` disables) |

The request caps are a safety net against agents stuck in loops; exceeding them
returns a `rate_limited` error that tells the agent to stop and notify the user.

## Errors

All tool errors return `{ code, message }` — e.g. `not_authenticated`, `no_permission`,
`save_conflict` (retry after re-reading), `validation_failed`, `rate_limited`.

## Links

- [Sqemo](https://sqemo.com) — team naming standards + ERD design in the browser
- [Live database guide](https://sqemo.com/docs/live-database) — drift checks in CI, step by step
- [Plans](https://sqemo.com/pricing) — which tools need Pro
- [sqemo-mcp on npm](https://www.npmjs.com/package/sqemo-mcp)
- Feedback and bug reports: [issues](https://github.com/sqemo/sqemo-mcp/issues) or hello@sqemo.com

## License

[MIT](LICENSE)
