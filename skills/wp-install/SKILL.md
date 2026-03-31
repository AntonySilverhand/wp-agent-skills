---
name: wp-install
description: Set up WordPress for a personal site — either connect to an existing installation or install a fresh one. Use at the very start of any site project, when the user says "I don't have WordPress yet", "install WordPress for me", "set up the site", or "where do I start". Detects the OS (Windows, macOS, Linux) and environment, chooses the best install method, configures credentials, and sets the WP_SITE_URL/WP_USER/WP_APP_PASSWORD environment variables so all other wp-* skills can run.
license: MIT
compatibility: Windows (PowerShell), macOS, Linux. PHP 8.0+ required. Local: PHP built-in server + SQLite (no MySQL needed). Remote: SSH or cPanel. WP-CLI auto-downloaded.
metadata:
  author: forLavon
  version: "1.0"
  category: wordpress
allowed-tools: Bash Read Write
---

# wp-install

Onboard a WordPress site. Connects to an existing installation or installs a fresh one.
Ends with all environment variables set and verified so every other skill works immediately.

## Step 1 — Detect OS and ask where the site lives

**Detect OS first:**
```bash
# On Linux/macOS (bash/zsh)
uname -s

# On Windows (PowerShell)
$env:OS
```

Adapt all subsequent commands to the detected OS. Key differences:

| Task | Windows (PowerShell) | macOS/Linux (bash) |
|---|---|---|
| Set env var | `$env:VAR = "value"` | `export VAR="value"` |
| Path separator | `\` | `/` |
| base64 encode | `[Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes("u:p"))` | `echo -n "u:p" \| base64` |
| Background process | `Start-Process php -ArgumentList ...` | `php ... &` |
| Write file | `Add-Content` or `Out-File` | `echo >> file` |
| Download file | `Invoke-WebRequest -Uri ... -OutFile ...` | `curl -sL ... -o ...` |

For Windows users: all commands below show the bash version. Use the PowerShell equivalents from the table above, or run inside WSL2 (Windows Subsystem for Linux) for a Linux-identical experience.

Ask the user:
> "Do you have a WordPress site already, or should we set one up?
> If you have one: share the URL and admin credentials.
> If not: local (on this computer) or remote (a hosting provider)?"

Then branch:

---

## Branch A — Existing site

Collect:
- Site URL (must be HTTPS for Application Passwords to work)
- Admin username
- Application Password (guide: WP Admin → Users → Profile → Application Passwords → Add New)

Set and verify:
```bash
export WP_SITE_URL="<url>"
export WP_USER="<user>"
export WP_APP_PASSWORD="<password>"

# Verify access
curl -s -u "$WP_USER:$WP_APP_PASSWORD" "$WP_SITE_URL/wp-json/wp/v2/users/me" \
  | jq '{name, roles}'
```

If this returns the user's name and roles: done. Skip to Step 4.
If it fails: troubleshoot (see Gotchas).

---

## Branch B — Local install (recommended for development)

Use subagents to run prerequisite checks in parallel, then install.

### B.1 — Parallel prerequisite check (use subagents)

Launch subagents to check these simultaneously:
- **Agent 1**: `php --version` — need 8.0+
- **Agent 2**: `php -r "echo extension_loaded('pdo_sqlite') ? 'ok' : 'missing';"` — need SQLite
- **Agent 3**: `curl -s https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar -o /tmp/wp-cli.phar && php /tmp/wp-cli.phar --version` — WP-CLI

Collect all results. If any fail, fix before continuing:

| Missing | Fix |
|---|---|
| PHP | Install PHP 8.0+ via system package manager |
| pdo_sqlite | Arch: `sudo pacman -S php-sqlite`; Ubuntu: `sudo apt install php-sqlite3`; macOS: `brew install php` |
| WP-CLI | Already handled by the check above |

### B.2 — Install WordPress core

```bash
WP_DIR="$HOME/Sites/lavon-wp"
mkdir -p "$WP_DIR"
php /tmp/wp-cli.phar core download --path="$WP_DIR" --skip-content
```

### B.3 — Install SQLite integration (parallel with core download if possible)

```bash
# Download the official WordPress SQLite plugin
curl -sL "https://downloads.wordpress.org/plugin/sqlite-database-integration.zip" \
  -o /tmp/sqlite-wp.zip
unzip -q /tmp/sqlite-wp.zip -d /tmp/sqlite-wp/
cp -r /tmp/sqlite-wp/sqlite-database-integration/. \
  "$WP_DIR/wp-content/plugins/sqlite-database-integration/"

# Install the db.php drop-in (routes all DB calls to SQLite)
cp "$WP_DIR/wp-content/plugins/sqlite-database-integration/db.copy" \
   "$WP_DIR/wp-content/db.php"

mkdir -p "$WP_DIR/wp-content/database"
```

### B.4 — Configure

```bash
php /tmp/wp-cli.phar config create \
  --path="$WP_DIR" \
  --dbname=lavon \
  --dbuser=wp \
  --dbpass=wp \
  --dbhost=localhost \
  --skip-check \
  --force

# SQLite-specific config (append to wp-config.php)
echo "define('DB_DIR', __DIR__ . '/wp-content/database/');" >> "$WP_DIR/wp-config.php"
echo "define('DB_FILE', 'lavon.sqlite');"                   >> "$WP_DIR/wp-config.php"
```

### B.5 — Run WordPress installer

```bash
php /tmp/wp-cli.phar core install \
  --path="$WP_DIR" \
  --url="http://localhost:8080" \
  --title="Lavon" \
  --admin_user="lavon" \
  --admin_password="lavon-dev-2024" \
  --admin_email="lavon@example.com" \
  --skip-email
```

### B.6 — Start PHP built-in server

**Linux/macOS:**
```bash
php -S localhost:8080 -t "$WP_DIR" &> /tmp/wp-server.log &
echo "WordPress running at http://localhost:8080"
```

**Windows (PowerShell):**
```powershell
$WP_DIR = "$env:USERPROFILE\Sites\lavon-wp"
Start-Process php -ArgumentList "-S localhost:8080 -t $WP_DIR" -RedirectStandardOutput "$env:TEMP\wp-server.log" -WindowStyle Hidden
Write-Host "WordPress running at http://localhost:8080"
```

**Windows (simplest — use LocalWP):**
If PHP isn't available natively on Windows, the easiest path is [LocalWP](https://localwp.com/) — a free GUI app that handles everything (PHP, MySQL, SSL). After creating a site in LocalWP:
1. Click "Open Site" → note the local URL
2. Click "Admin" → Users → Profile → Application Passwords
3. Set `WP_SITE_URL`, `WP_USER`, `WP_APP_PASSWORD` and skip to Step 4

> Note: for a proper local server, replace the built-in PHP server with nginx + php-fpm.
> The built-in server is single-threaded and fine for development/testing.

### B.7 — Generate Application Password

```bash
php /tmp/wp-cli.phar user application-password create lavon "wp-skills-dev" \
  --path="$WP_DIR" --porcelain
```

Save the output — this is the Application Password.

---

## Branch C — Remote hosting install

Collect from user:
- Hosting provider (Cloudways, SiteGround, Kinsta, cPanel, etc.)
- SSH or SFTP credentials

Use subagents to run in parallel where possible:
- **Agent 1**: Check if WordPress is already present at the remote path
- **Agent 2**: Check if WP-CLI is available on the remote server

Then follow [references/remote-install.md](references/remote-install.md) for provider-specific steps.

---

## Step 4 — Set environment variables

```bash
export WP_SITE_URL="http://localhost:8080"   # or the remote URL
export WP_USER="lavon"
export WP_APP_PASSWORD="<generated password>"
```

Write these to a `.env` file in the project root so they persist across sessions:
```bash
cat > .env << EOF
WP_SITE_URL=http://localhost:8080
WP_USER=lavon
WP_APP_PASSWORD=<password>
EOF
```

## Step 5 — Final verification

```bash
curl -s -u "$WP_USER:$WP_APP_PASSWORD" \
  "$WP_SITE_URL/wp-json/wp/v2/users/me" | jq '{name, roles}'
```

Expected: `{"name": "lavon", "roles": ["administrator"]}`

## Gotchas

- **Application Passwords require HTTPS on live sites** but work over HTTP in local development. If you get 401 on a remote site, ensure HTTPS is active.
- **The SQLite drop-in (`db.php`) must be in `wp-content/`, not `wp-content/plugins/`.** If it's in the wrong place, WordPress will try to connect to MySQL and fail.
- **PHP built-in server is single-threaded.** It cannot handle the block editor's parallel requests well. Use for CLI testing only — open the site in the browser after running the installer.
- **WP-CLI with SQLite requires the `db.php` drop-in before running `core install`**, not after. If you install core first, the SQLite translator won't initialise and the installer will fail trying to connect to MySQL.
- **`--skip-check` on `config create` is required with SQLite** because WP-CLI tries to connect to MySQL to verify the config, which will always fail when using SQLite.

## Output

- Site URL and admin URL
- Admin credentials (username + password for dev)
- Application Password (save this — it's only shown once)
- Confirmation: `curl` test returns the admin user and roles
- Next step: "Run `wp-setup-theme` to scaffold the visual theme."
