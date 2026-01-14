# ngm (Nginx Manager)

Command-line tool for managing Nginx configs and server blocks. It lets you quickly manage your available and enabled websites in a visual, interactive way without remembering config paths.

## How it Works

`ngm` simplifies Nginx management by handling the standard Debian/Ubuntu layout:

- **Sources**: Configs are stored in `/etc/nginx/sites-available`.
- **Enabled**: Active sites are symlinked to `/etc/nginx/sites-enabled`.
- **Extensions**: Config files use the `.nginx` extension (hidden from the user in CLI).
- **Organization**: Subdomains (e.g., `blog.example.com` and `www.example.com`) are grouped under their main domain (`example.com`) in list views.

### Safety First

- **Validation**: Before creating or renaming a site, `ngm` ensures the name is a valid domain.
- **Config Test**: After editing, creating, or toggling sites, `ngm` runs `nginx -t` to test the configuration. The service is only reloaded if the test passes.
- **No-Reload Option**: Most commands support `--no-reload` if you want to make multiple changes before applying them.

## Installation

1. Ensure you have the required dependencies:

   - `nginx`
   - `bash` (4.0+)
   - `fzf`
   - `bat` (optional, for viewing configs)

2. Copy the script to your path or symlink it:

   ```bash
   sudo ln -s /path/to/ngm /usr/local/bin/ngm
   ```

## Usage

### List Sites

```bash
ngm ls   # List all sites with status
```

Displays all websites grouped by domain. Enabled sites show ✅, disabled show ❌. Subdomains are sorted alphabetically, with `@/www` always first.

```
Sites:
  example.com
    ├─ ✅ @/www
    └─ ❌ api
  otherdomain.org
    ├─ ✅ @/www
    ├─ ❌ blog
    └─ ✅ shop
```

### Create a Website

```bash
ngm mk example.com
```

Creates `example.com.nginx` in `sites-available`, validates the name, and opens your default editor. Tests config and reloads on exit.

### Enable/Disable Sites

```bash
ngm enable   # Interactive list of disabled sites
ngm disable  # Interactive list of enabled sites
```

Select a site to toggle its status.

### Edit and View

```bash
ngm edit     # Interactive selection to edit with $EDITOR
ngm view     # Interactive selection to view with bat
```

### Manage Configs

```bash
ngm cp old.com new.com       # Duplicate a config
ngm rename old.com new.com   # Rename a config (and symlink)
```

### Logs

```bash
ngm logs        # Interactive selection to tail site-specific logs
ngm access-log  # Tail global access log
ngm error-log   # Tail global error log
```

### Service Control

```bash
ngm status   # Show service status and site counts
ngm test     # Run nginx -t
ngm reload   # Reload service
ngm restart  # Restart service
```

## Disclaimer

This tool was vibe-coded. Use at your own risk.
