# cf - Cloudflare DNS CLI

Dead simple CLI for managing Cloudflare DNS records.

## Setup

```bash
cf init
# Enter your Cloudflare API token (needs Zone:DNS:Edit permission)
```

## Usage

```bash
# List your zones
cf zones

# List records for a domain
cf list dehouse.gg

# Add records
cf add A staging.dehouse.gg 1.2.3.4        # DNS only
cf add A staging.dehouse.gg 1.2.3.4 true   # Proxied (orange cloud)
cf add CNAME www.dehouse.gg dehouse.gg

# Update a record
cf update staging.dehouse.gg 5.6.7.8

# Delete records
cf delete staging.dehouse.gg A       # Delete specific type
cf delete staging.dehouse.gg         # Delete all records for name

# Toggle proxy
cf proxy staging.dehouse.gg on
cf proxy staging.dehouse.gg off
```

## Config

Config stored at `~/.config/cf/config` (or set `CF_CONFIG` env var).

## Requirements

- bash, curl, jq
