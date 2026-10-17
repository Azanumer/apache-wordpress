# Apache + WordPress

Production-ready Apache 2.4 VirtualHost configs for WordPress with PHP-FPM —
the classic shared-hosting / Plesk-style stack, tuned for security and speed.

## Files

| File | Purpose |
|---|---|
| `apache-wordpress.conf` | Full vhost: HTTP→HTTPS redirect, PHP-FPM via socket, TLS 1.2+, static-asset caching, deflate, security headers, blocked sensitive files |
| `security-headers.conf` | Stricter drop-in headers snippet (CSP report-only starter, 2-year HSTS with preload) |

## Requirements

- Apache 2.4 with `mpm_event`
- PHP-FPM (7.4+; the config points at the 8.2 socket — adjust to yours)
- Certbot (or any cert) for the TLS paths used

## Install

```bash
sudo cp apache-wordpress.conf /etc/apache2/sites-available/example.com.conf
sudo sed -i 's/example.com/yourdomain.com/g' /etc/apache2/sites-available/yourdomain.com.conf
# then fix the DocumentRoot and PHP-FPM socket version inside the file

sudo a2enmod ssl rewrite headers expires deflate proxy proxy_fcgi
sudo a2dismod mpm_prefork   # mpm_event + PHP-FPM only
sudo a2enmod mpm_event
sudo a2ensite yourdomain.com
sudo apachectl configtest
sudo systemctl reload apache2
```

## Tuning notes

- **mpm_event**: with PHP-FPM, Apache only serves static files and proxies
  PHP. A sane starting point in `mpm_event.conf`:
  `StartServers 2`, `MinSpareThreads 25`, `MaxSpareThreads 75`,
  `ThreadsPerChild 25`, `MaxRequestWorkers 150`.
- **www vs non-www**: the config canonicalises everything to
  `https://example.com/` (no www). If you prefer www, flip the
  `Redirect` target and the `ServerName`.
- **xmlrpc.php** is denied for hardening. Comment out that filename if you
  use Jetpack or the WordPress mobile app.
- Run `curl -sI https://yourdomain.com/wp-config.php` — you should get a
  403. If you get a 200, the `<FilesMatch>` block is not being applied.

MIT licensed.
