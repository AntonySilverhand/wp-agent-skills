# Remote Hosting Install Guide

## Cloudways / managed hosting

1. In the Cloudways dashboard, create a new application → WordPress
2. Wait for provisioning (~2 min)
3. Get credentials: Application → Access Details → Admin Panel URL
4. Log in, go to Users → Profile → Application Passwords → Add New
5. Set `WP_SITE_URL` to the application URL (always HTTPS on Cloudways)

## cPanel hosting (SiteGround, Bluehost, etc.)

1. cPanel → Softaculous → WordPress → Install
2. Set directory to root (leave blank for domain root)
3. After install: log into WP admin, create Application Password
4. If WP-CLI is available via SSH:
   ```bash
   wp --info  # check if available
   wp user application-password create admin "skills-dev" --porcelain
   ```

## VPS with SSH (DigitalOcean, Linode, etc.)

```bash
# SSH into the server
ssh user@your-server-ip

# Install WP-CLI if not present
curl -sL https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar -o /usr/local/bin/wp
chmod +x /usr/local/bin/wp

# Assuming Apache/Nginx + MySQL already configured
wp core download --path=/var/www/html
wp config create --dbname=lavon --dbuser=lavon_db --dbpass=<pass> --dbhost=localhost
wp db create
wp core install --url=https://yourdomain.com --title="Lavon" \
  --admin_user=lavon --admin_password=<pass> --admin_email=lavon@email.com
wp user application-password create lavon "wp-skills" --porcelain
```

## Verifying remote access from local machine

```bash
# Test REST API is accessible
curl -s "https://yourdomain.com/wp-json/wp/v2/types" | jq 'keys'

# Test authenticated access
curl -s -u "lavon:<app-password>" \
  "https://yourdomain.com/wp-json/wp/v2/users/me" | jq '{name, roles}'
```
