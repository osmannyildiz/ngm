# AGENTS.md

This file provides context for AI coding agents working on this project.

## Project Overview

**ngm (Nginx Manager)** is a command-line tool for managing Nginx site configurations on Debian/Ubuntu servers. It simplifies working with the standard `sites-available`/`sites-enabled` layout by providing interactive commands, domain validation, and automatic config testing.

### Key Vision & Decisions

1. **Target Environment**: Linux servers with Debian/Ubuntu-style Nginx layout (not macOS).
2. **Bash 4.0+**: The script requires Bash 4.0+ for associative arrays and `mapfile`.
3. **Interactive First**: Many commands use `fzf` for interactive selection rather than requiring arguments.
4. **Safety by Default**: Config changes trigger `nginx -t` before reloading; `--no-reload` flag available to batch changes.

## File Structure

```
bin/ngm          # Main executable script (Bash)
README.md        # User-facing documentation
AGENTS.md        # This file
```

## Key Assumptions

These design decisions from the spec are **assumed, not enforced**:

1. **@/www Convention**: The root domain and `www` subdomain are managed by the same config file. The filename uses just the domain (e.g., `example.com.nginx`), shown as `@/www` in grouped lists.
2. **Config Persistence**: If `nginx -t` fails after edits, changes still persist on disk. Only the reload is skipped.
3. **Dependencies**: `fzf` and `bat`/`batcat` are assumed to be installed.
4. **Zsh Compatibility**: Spec mentions it as "nice to have" but current implementation is Bash-only (uses Bash-specific features).

## Architecture & Implementation Details

### Configuration Paths

| Constant           | Value                        | Purpose                        |
| ------------------ | ---------------------------- | ------------------------------ |
| `SITES_AVAILABLE`  | `/etc/nginx/sites-available` | Source config directory        |
| `SITES_ENABLED`    | `/etc/nginx/sites-enabled`   | Enabled configs (symlinks)     |
| `EXT`              | `.nginx`                     | File extension for all configs |
| `NGINX_ACCESS_LOG` | `/var/log/nginx/access.log`  | Global access log              |
| `NGINX_ERROR_LOG`  | `/var/log/nginx/error.log`   | Global error log               |

### Domain Parsing

The `parse_domain()` function handles:

- Simple domains: `example.com` → domain=`example.com`, subdomain=`@/www`
- Subdomains: `blog.example.com` → domain=`example.com`, subdomain=`blog`
- Compound TLDs: `example.com.tr` → domain=`example.com.tr`, subdomain=`@/www`

**Supported compound TLDs** are hardcoded in `COMPOUND_TLDS`:

```
com.tr org.tr gov.tr edu.tr co.uk org.uk gov.uk com.au net.au org.au co.nz org.nz com.br org.br
```

### Command Categories

| Category    | Commands                                               |
| ----------- | ------------------------------------------------------ |
| List        | `ls`                                                   |
| Interactive | `view`, `edit`, `enable`, `disable`                    |
| File        | `mk`, `cp`, `rename`                                   |
| Service     | `status`, `test`, `reload`, `restart`, `start`, `stop` |
| Logs        | `access-log`, `error-log`, `logs`                      |

### Key Helper Functions

| Function                  | Purpose                                          |
| ------------------------- | ------------------------------------------------ |
| `is_valid_domain()`       | Regex validation for domain names                |
| `get_site_files()`        | List all `.nginx` files in sites-available       |
| `get_enabled_sites()`     | List symlinks in sites-enabled                   |
| `get_disabled_sites()`    | Difference of sets: available - enabled          |
| `parse_domain()`          | Extract main domain and subdomain from site name |
| `display_grouped_sites()` | Group sites by domain for `ls` output            |
| `fzf_select()`            | Wrapper for interactive fzf selection            |
| `test_and_reload_nginx()` | Run `nginx -t` and reload if valid               |
| `reload_nginx()`          | Reload without testing (used by `disable`)       |
| `get_editor()`            | Returns `$EDITOR` or defaults to `nano`          |
| `get_viewer()`            | Tries `batcat`, then `bat`, then `less`          |

## Coding Conventions

1. **Error Output**: Use `print_err()` for errors (prints to stderr in red).
2. **User Feedback**: Use `print_info()` (cyan) and `print_success()` (green).
3. **Command Pattern**: Each command is a `cmd_<name>()` function, dispatched by `main()`.
4. **Privileged Operations**: Use `sudo` for file operations in `/etc/nginx/` and service control.
5. **Flag Handling**: Use `has_no_reload()` to check and `strip_no_reload()` to remove the flag from args.

## Adding New Features

### Adding a New Command

1. Create a `cmd_<name>()` function in the `COMMANDS` section (between the `== COMMANDS BEGIN ==` and `== COMMANDS END ==` markers).
2. Add the case to the `main()` function's case statement.
3. Add help text in `cmd_help()`.
4. If the command modifies configs, call `test_and_reload_nginx "$no_reload"` at the end.
5. Update README.md with usage documentation.

### Adding a New Compound TLD

Add the TLD string to the `COMPOUND_TLDS` variable (space-separated).

## Testing Notes

Don't test changes by running commands directly. Instead, ask the user to manually test the changes.

## Common Pitfalls

1. **macOS Incompatibility**: The script uses `find -printf` (GNU find) which doesn't exist on macOS.
2. **Sudo Requirements**: Most operations require sudo; the script doesn't check for root access upfront.
3. **Hardcoded Paths**: All paths assume standard Debian/Ubuntu Nginx installation.
4. **Extension Handling**: The `.nginx` extension is stripped/added automatically; don't include it in user-facing commands.
