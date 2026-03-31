---
name: wp-security
description: Install and configure Wordfence security on a WordPress site — firewall, malware scanner, login protection, and hardening settings. Use when setting up a new site, when the user asks to "secure the site", "add security", "install Wordfence", or before going live. Also use when a security issue is suspected. Applies a layered security baseline: Wordfence + core hardening + file permission checks.
license: MIT
compatibility: Requires WordPress 6.4+, WP-CLI, WP REST API. Wordfence requires a live internet connection on the server. Not needed for local development — skip to Step 5 (hardening) for local installs.
metadata:
  author: forLavon
  version: "1.0"
  category: wordpress
allowed-tools: Bash Read Write
---

# wp-security

Full security setup: Wordfence installation + configuration + site hardening.
Run once before going live. Re-run after any suspected compromise.

## Subagent strategy

Use two subagents in parallel:
- **Agent 1**: Install and activate Wordfence, configure firewall rules
- **Agent 2**: Run site hardening (file permissions, wp-config.php flags, user audit)

Merge both reports into a single security summary.

## Step 1 — Check environment

```bash
wp eval "echo is_ssl() ? 'HTTPS: yes' : 'HTTPS: no'; echo PHP_EOL;"
wp eval "echo get_bloginfo('url'); echo PHP_EOL;"
wp core version
```

If `is_ssl()` is false on a live (non-local) site: **stop and set up SSL first**.
Wordfence's firewall is far less effective over HTTP.

## Step 2 — Install Wordfence

```bash
wp plugin install wordfence --activate
```

Verify activation:
```bash
wp plugin list --status=active | grep wordfence
```

## Step 3 — Configure Wordfence via WP-CLI

Wordfence stores its config in the `wflogs/` directory and the `wp_wfconfig` table.
Apply the recommended baseline settings:

```bash
# Enable extended protection (firewall before WordPress loads)
wp eval "wf_maybe_update_firewall_rules();" 2>/dev/null || true

# Set security level to HIGH
wp option update wf_secLevel 2

# Enable login security
wp eval "
update_option('wf_loginSecurity', [
  'blockBruteForce'        => true,
  'countFailedForUsername' => true,
  'countFailedForIP'       => true,
  'maxFailuresPerUsername' => 5,
  'maxFailuresPerIP'       => 20,
  'lockoutMins'            => 60,
  'twoFactorForRoles'      => ['administrator'],
]);
echo 'Login security configured';
"
```

For full Wordfence configuration options, see [references/wordfence-options.md](references/wordfence-options.md).

## Step 4 — Wordfence API key (optional but recommended)

A free Wordfence API key enables real-time threat intelligence:
1. Go to wordfence.com → Get Free API Key
2. Enter the site URL and email
3. Apply the key:

```bash
wp eval "
update_option('wordfence_apiKey', '<YOUR_API_KEY>');
echo 'API key set';
"
```

Skip this step for local/dev environments.

## Step 5 — Site hardening (independent of Wordfence)

### 5a — wp-config.php flags

Ensure these are set in `wp-config.php`:
```php
define('DISALLOW_FILE_EDIT', true);       // disable theme/plugin editor in admin
define('DISALLOW_FILE_MODS', true);       // disable plugin/theme installs from admin (optional - comment out during setup)
define('WP_DEBUG', false);                // never true in production
define('FORCE_SSL_ADMIN', true);          // admin over HTTPS only (live sites)
```

Check current state:
```bash
wp config get DISALLOW_FILE_EDIT 2>/dev/null || echo "not set"
wp config get WP_DEBUG
```

Set if missing:
```bash
wp config set DISALLOW_FILE_EDIT true --raw --type=constant
wp config set WP_DEBUG false --raw --type=constant
```

### 5b — Remove default content

```bash
wp post delete 1 --force 2>/dev/null || true   # delete "Hello World" post
wp user delete 1 --reassign=<admin_user_id> 2>/dev/null || true  # delete default admin if different user exists
wp option update blogdescription ''             # remove "Just another WordPress site" tagline
```

### 5c — Block sensitive file access

Add to `.htaccess` (Apache) or Nginx config. For the PHP built-in server, the router.php already blocks direct PHP execution.

For live Apache hosts:
```apache
# Block access to wp-config.php
<Files wp-config.php>
  Order deny,allow
  Deny from all
</Files>

# Block access to .git, .env, readme files
<FilesMatch "^(readme\.html|license\.txt|wp-config-sample\.php|\.env|\.git)">
  Order deny,allow
  Deny from all
</FilesMatch>

# Block PHP execution in uploads
<Directory "/wp-content/uploads">
  <Files "*.php">
    Order deny,allow
    Deny from all
  </Files>
</Directory>
```

### 5d — File permission audit

```bash
# wp-config.php should be 640 or 600 (owner read/write only)
stat -c "%a %n" wp-config.php

# wp-content/ should be 755
stat -c "%a %n" wp-content/

# Fix if needed
chmod 640 wp-config.php
chmod 755 wp-content/
find wp-content/ -type f -exec chmod 644 {} \;
find wp-content/ -type d -exec chmod 755 {} \;
```

### 5e — Admin user audit

```bash
wp user list --role=administrator --format=table
```

Flag:
- Any admin account with username `admin` or `administrator` → rename it
- More than 1 admin account → verify they're all needed

Rename default admin:
```bash
wp user update <id> --user_login=<new_name>
```

## Step 6 — Run initial Wordfence scan

```bash
# Trigger scan via WP-CLI (Wordfence 7.x+)
wp eval "
if (class_exists('wfScanner')) {
  wfScanner::startScan();
  echo 'Scan started — check WP Admin → Wordfence → Scan for results';
} else {
  echo 'Wordfence scanner not available via CLI — run from WP Admin → Wordfence → Scan';
}
"
```

## Step 7 — Verify

```bash
# Confirm Wordfence is active and firewall is enabled
wp eval "
echo 'Wordfence active: ' . (is_plugin_active('wordfence/wordfence.php') ? 'yes' : 'no') . PHP_EOL;
echo 'Firewall mode: ' . get_option('wf_howGetIPs', 'not set') . PHP_EOL;
"

# Test that wp-config.php is not web-accessible (live sites)
curl -s -o /dev/null -w "%{http_code}" "$WP_SITE_URL/wp-config.php"
# expect: 403
```

## Gotchas

- **`DISALLOW_FILE_MODS true` blocks ALL plugin/theme installs**, including from WP-CLI when `--allow-root` isn't set. Comment it out during initial setup, set it after all plugins are installed.
- **Wordfence's extended firewall protection requires PHP auto_prepend_file** to be set in `.htaccess` or `php.ini`. WP-CLI can't do this — Wordfence does it automatically on first activation through the admin UI. If running headless (no browser), set manually: `auto_prepend_file = /path/to/wp-content/wflogs/rules.php`
- **On PHP built-in server (local dev), skip Wordfence entirely.** The firewall `.htaccess` directives don't apply and the WAF rules will cause errors. Install Wordfence only on live/staging servers.
- **Wordfence free has a 30-day delay on firewall rules** vs the premium real-time feed. For a personal site, free is fine. For a business site, consider Wordfence Premium.
- **`wp user delete 1`** will fail if user ID 1 is the only admin. Always ensure another admin exists before deleting.
- **File permission audit must run as the web server user**, not root. Permissions set by root may still be unreadable by PHP. Verify with `wp eval "echo is_writable(ABSPATH);"`.

## Output

Security summary:

| Check | Status |
|---|---|
| Wordfence installed | ✓ / ✗ |
| Firewall active | ✓ / ✗ |
| Login protection | ✓ / ✗ |
| DISALLOW_FILE_EDIT | ✓ / ✗ |
| WP_DEBUG off | ✓ / ✗ |
| wp-config.php permissions | 640 / ⚠ |
| Default admin renamed | ✓ / ✗ |
| Initial scan | clean / issues found |

Flag any ✗ or ⚠ as action items before going live.
