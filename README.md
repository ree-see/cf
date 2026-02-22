# cf - Cloudflare DNS CLI

A simple, powerful CLI for managing Cloudflare DNS records.

## Install

```bash
# Download and install
curl -fsSL https://raw.githubusercontent.com/ree-see/cf/main/cf > /usr/local/bin/cf
chmod +x /usr/local/bin/cf

# Setup your API token
cf init
```

## Requirements

- `curl`
- `jq`
- Cloudflare API token with DNS edit permissions

## Usage

### Global Flags

```bash
--dry-run, -n   Show what would happen without making changes
--json, -j      Output raw JSON (works with list, zones)
```

### DNS Records

```bash
# List all records for a domain
cf list example.com
cf --json list example.com | jq length

# Add records
cf add A staging.example.com 1.2.3.4
cf add A staging.example.com 1.2.3.4 true  # with Cloudflare proxy
cf add CNAME www.example.com example.com
cf add TXT example.com "v=spf1 include:_spf.google.com ~all"
cf add MX example.com mail.example.com 10

# Update a record
cf update staging.example.com 5.6.7.8
cf update staging.example.com 5.6.7.8 A

# Delete records
cf delete staging.example.com        # all records for name
cf delete staging.example.com A      # specific type
cf --dry-run delete staging.example.com  # preview what would be deleted

# Toggle Cloudflare proxy (orange cloud)
cf proxy staging.example.com on
cf proxy staging.example.com off

# Update TTL (1 = auto)
cf ttl staging.example.com 300
cf ttl staging.example.com 1   # auto

# Search across all zones
cf search staging
cf search 192.168
```

### Backup & Restore

```bash
# Export to JSON (default)
cf export example.com > backup.json

# Export in BIND zone file format
cf export example.com bind > zone.txt

# Import from JSON
cf import example.com backup.json
cf --dry-run import example.com backup.json  # preview imports

# Clone all records from one zone to another
cf clone source.com target.com
cf --dry-run clone source.com target.com     # preview clone

# Diff live records against a backup
cf diff example.com backup.json
```

### Zone Management

```bash
# List all your zones
cf zones
cf --json zones | jq '.[0].name'

# Show zone status and settings
cf status example.com

# Show SSL certificate info
cf ssl example.com

# Purge cache
cf purge example.com                          # purge everything
cf purge example.com https://example.com/page # specific URLs

# Toggle development mode (bypasses cache for 3 hours)
cf dev-mode example.com on
cf dev-mode example.com off
```

## JSON Import Format

```json
[
  {
    "type": "A",
    "name": "staging.example.com",
    "content": "1.2.3.4",
    "proxied": true,
    "ttl": 1
  },
  {
    "type": "CNAME",
    "name": "www.example.com",
    "content": "example.com",
    "proxied": true
  },
  {
    "type": "MX",
    "name": "example.com",
    "content": "mail.example.com",
    "priority": 10
  }
]
```

## Configuration

API token is stored in `~/.config/cf/config`:

```bash
CF_API_TOKEN="your-token-here"
```

Override with environment variable: `CF_CONFIG=/path/to/config cf list example.com`

## Creating an API Token

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com/profile/api-tokens)
2. Create Token → "Edit zone DNS" template
3. Select zones you want to manage
4. Copy the token and run `cf init`

## License

MIT
