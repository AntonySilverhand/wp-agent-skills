# Wordfence Configuration Options

## Key wp_options entries Wordfence uses

| Option key | Values | Recommended |
|---|---|---|
| `wf_secLevel` | 0 (low), 1 (medium), 2 (high) | 2 |
| `wordfence_apiKey` | string | set via wordfence.com |
| `wfsecurity_enabled` | true/false | true |
| `wf_loginSecurity` | array (see below) | see Step 3 |
| `wf_howGetIPs` | `REMOTE_ADDR`, `HTTP_X_FORWARDED_FOR` | `REMOTE_ADDR` unless behind proxy |
| `wf_alertEmails` | email address | site admin email |
| `wf_enableLiveTraffic` | true/false | false (performance cost) |

## Login security array structure

```php
[
  'blockBruteForce'        => true,
  'countFailedForUsername' => true,   // count fails per username
  'countFailedForIP'       => true,   // count fails per IP
  'maxFailuresPerUsername' => 5,      // lock after 5 bad passes for a username
  'maxFailuresPerIP'       => 20,     // lock IP after 20 fails
  'lockoutMins'            => 60,     // lock for 1 hour
  'twoFactorForRoles'      => ['administrator'],  // require 2FA for admins
  'blockAdminRegistration' => true,   // block new admin registration
]
```

## Recommended email alerts

```php
update_option('wf_alertEmails', 'lavon@example.com');
update_option('wf_alertOn_adminLogin', true);        // alert on admin login
update_option('wf_alertOn_block', true);             // alert on IP block
update_option('wf_alertOn_scanIssues', true);        // alert on scan findings
update_option('wf_alertOn_updateAvailable', true);   // alert on plugin updates
```

## Checking Wordfence firewall status

```bash
wp eval "
\$status = get_option('wf_wafStatus', 'unknown');
echo 'WAF status: ' . \$status . PHP_EOL;
\$rules = file_exists(WP_CONTENT_DIR . '/wflogs/rules.php') ? 'present' : 'missing';
echo 'Firewall rules file: ' . \$rules . PHP_EOL;
"
```

## Disabling Wordfence live traffic (recommended for low-resource hosting)

```bash
wp eval "update_option('wf_enableLiveTraffic', false); echo 'Live traffic disabled';"
```

## Allowlisting your own IP (prevents lockout during development)

```bash
wp eval "
\$ips = get_option('wf_whitelisted_ips', '');
\$my_ip = '<YOUR_IP>';
update_option('wf_whitelisted_ips', \$ips ? \$ips . ',' . \$my_ip : \$my_ip);
echo 'IP allowlisted';
"
```
