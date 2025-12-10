# ngm (Nginx Manager)

ngm is a command-line tool for managing Nginx configs and server blocks. It lets the user quickly manage their available and eanbled websites, without trying to remember the paths of the configs and server blocks, and in a more visual way.

## Notes

- Config files will have the extension ".nginx", but that part won't be shown to or expected from the user through commands.
- No-subdomain and www subdomain will always be managed by the same server block. This will be assumed, not enforced or verified. The file name will be the domain and TLD, it won't include the "www" subdomain part. In grouped lists it will be denoted as "@/www".
- If the config check fails, Nginx service won't be reloaded, instead the user will be notified of the error. Changes will still persist.
- If fzf is exited without a selection, the command will exit, informing the user that no action was taken.
- When we say "a valid domain name", it can include subdomains or not.
- Modifying commands will have a "--no-reload" flag.
- Default editor is `$EDITOR`, or "nano" if `$EDITOR` is not set.
- fzf and bat will be assumed to exist.

## Commands

- `ngm ls`: Displays all websites (files in /etc/nginx/sites-available), grouped by their domain and TLD. Enabled sites are shown with ✅, disabled sites with ❌. For example, under the "example.com" group, there can be "@/www", "blog" and "project1.demo" subdomains. Subdomains are sorted alphabetically within each group, except "@/www" which is always shown first.
- `ngm view`: Using fzf, lists the files in the `/etc/nginx/sites-available` folder in a fuzzy-searchable way. If the user presses Enter in fzf, the file will be opened in the `bat` command line utility.
- `ngm edit`: Using fzf, lists the files in the `/etc/nginx/sites-available` folder in a fuzzy-searchable way. If the user presses Enter in fzf, the file will be opened in the default editor with sudo privileges. On exit, Nginx config will be tested and if it's valid, the service will be reloaded.
- `ngm disable`: Using fzf, lists the symlinks in the `/etc/nginx/sites-enabled` folder in a fuzzy-searchable way. If the user presses Enter in fzf, the symlink will be removed from `/etc/nginx/sites-enabled`. On exit, Nginx service will be reloaded.
- `ngm enable`: Using fzf, lists the files in the `/etc/nginx/sites-available` folder that are not symlinked to `/etc/nginx/sites-enabled`, in a fuzzy-searchable way. If the user presses Enter in fzf, the file will be symlinked to `/etc/nginx/sites-enabled`. On exit, Nginx config will be tested and if it's valid, the service will be reloaded.
- `ngm mk <SITE_NAME>`: Creates a new file named like `<SITE_NAME>.nginx` in the `/etc/nginx/sites-available` folder, ensures the name is a valid domain name, and opens it in the default editor with sudo privileges. On exit, Nginx config will be tested and if it's valid, the service will be reloaded.
- `ngm cp <SITE_NAME> <NEW_SITE_NAME>`: Duplicates the site config file with the new name, ensures the new name is a valid domain name, and opens it in the default editor with sudo privileges. On exit, Nginx config will be tested and if it's valid, the service will be reloaded.
- `ngm rename <SITE_NAME> <NEW_SITE_NAME>`: Renames the site config file (and the symlink if the site is enabled), ensures the new name is a valid domain name. On exit, Nginx config will be tested and if it's valid, the service will be reloaded.
- `ngm status`: Displays the status of the Nginx service, last reload time, the number of enabled and available sites, and the last 10 lines of the Nginx error log.
- `ngm access-log`: Displays the Nginx access log using `tail -f`.
- `ngm error-log`: Displays the Nginx error log using `tail -f`.
- `ngm logs`: Using fzf, lists the symlinks in the `/etc/nginx/sites-enabled` folder in a fuzzy-searchable way. If the user presses Enter in fzf, it will display the Nginx logs for the selected site using `tail -f`.
- `ngm reload`: Reloads the Nginx service.
- `ngm test`: Tests the Nginx config.
- `ngm help`: Displays help information.
- `ngm restart`: Restarts the Nginx service.
- `ngm stop`: Stops the Nginx service.
- `ngm start`: Starts the Nginx service.

## Implementation notes

- This will be implemented with shell scripting. Primary target is Bash. Zsh compatibility would be nice to have.
