# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

`cf` is a single-file Bash CLI tool (~750 lines) for managing Cloudflare DNS via the Cloudflare API v4. The entire application lives in one executable file: `./cf`.

## Running the Tool

```bash
# Direct execution (no build step)
./cf help
./cf init                           # one-time setup, stores token in ~/.config/cf/config
./cf list example.com
CF_CONFIG=/path/to/config ./cf list example.com  # override config path

# Global flags (work with any command)
./cf --dry-run add A test.example.com 1.2.3.4
./cf --json list example.com | jq length
./cf --json zones | jq '.[0].name'
```

## Architecture

The script follows a flat pattern with these layers:

1. **Config** (`load_config`): Sources `~/.config/cf/config` (or `$CF_CONFIG`) and validates `CF_API_TOKEN` is set.
2. **API wrapper** (`cf_api`): All HTTP calls go through this single function — `cf_api <METHOD> <endpoint> [extra curl args]`. It prepends the base URL and injects the Bearer token.
3. **Pagination helper** (`cf_api_paged`): Iterates all pages for a GET endpoint and returns a single JSON array. Used by `list`, `zones`, `search`, `export`, `clone`, and `diff`.
4. **Zone resolver** (`get_zone_id`): Converts a domain name to a Cloudflare Zone ID. Tries 2-, 3-, and 4-part suffixes so multi-label TLDs like `.co.uk` resolve correctly.
5. **Global flag parsing**: Strips `--dry-run`/`-n` and `--json`/`-j` from `$@` before dispatch so they work in any position.
6. **Command functions** (`cmd_*`): One function per command. Each calls `load_config`, then `get_zone_id` if needed, then `cf_api` / `cf_api_paged`.
7. **Dispatcher**: A `case` statement at the bottom routes `$1` to the appropriate `cmd_*` function, shifting args before passing.

## Commands

| Command | Description |
|---------|-------------|
| `init` | Configure API token |
| `zones` | List all zones (supports `--json`) |
| `list <domain>` | List DNS records (supports `--json`) |
| `add <type> <name> <content> [proxied]` | Add a DNS record |
| `add MX <name> <content> [priority] [proxied]` | Add MX record with priority |
| `update <name> <content> [type]` | Update a DNS record |
| `delete <name> [type]` | Delete DNS record(s) (supports `--dry-run`) |
| `proxy <name> <on\|off>` | Toggle Cloudflare proxy |
| `ttl <name> <ttl> [type]` | Update TTL |
| `search <pattern>` | Search records across all zones |
| `export <domain> [json\|bind]` | Export records to stdout |
| `import <domain> <file.json>` | Import records from JSON (supports `--dry-run`) |
| `clone <src> <dst>` | Copy all records from source to target zone (supports `--dry-run`) |
| `diff <domain> <backup.json>` | Compare live records against a backup file |
| `purge <domain> [urls]` | Purge cache |
| `dev-mode <domain> <on\|off>` | Toggle development mode |
| `ssl <domain>` | Show SSL certificate status |
| `status <domain>` | Show zone status & settings |

## Known Limitations

- **`cmd_update` and `cmd_proxy` only touch `result[0]`**: If multiple records exist for a name+type pair, only the first is affected. A warning is printed when this happens.
- **`cmd_search` is sequential**: Makes one set of paginated API calls per zone. With many zones this is slow and may hit CF rate limits.
- **`cmd_purge` URL mode**: Passes all positional args after domain as URLs directly (fixed from original `shift` bug).

## Dependencies

`curl`, `jq`, `column`, `awk`, `sed` — all must be on `$PATH`. No package manager or build toolchain needed.

## Config File

Stored at `~/.config/cf/config`, chmod 600, sourced as bash:
```bash
CF_API_TOKEN="your-token-here"
```
This means the config file is executed as shell code — avoid writing other data there.
