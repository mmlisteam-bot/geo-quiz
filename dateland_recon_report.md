# Recon Report: dateland.co.il

## Summary
- **Target**: dateland.co.il (Israeli dating site, 500K+ users)
- **Server**: nginx
- **HTTP/2**: Enabled (200 OK)
- **CDN**: Custom CDN at cdn.datesupport.net

## Technical Findings

### Headers & Security
```
HTTP/2 200
Server: nginx
Content-Type: text/html; charset=utf-8
Cache-Control: no-store, no-cache, must-revalidate
Cookies: tid, _check, _check_pc (all secure, long expiry)
```
**Note**: No X-Frame-Options, X-Content-Type-Options, or CSP headers detected.

### Technology Stack
- **Frontend**: jQuery 3.2.1, jQuery UI 1.12.1, Fancybox 2.1.5
- **Custom JS**: common.js, longpoll.js (v654)
- **CDN**: cdn.datesupport.net (self-hosted CDN)
- **Google Services**: Google Sign-In, AdSense (ca-pub-6620147734904702)

### Sensitive File Tests
| Path | Status | Notes |
|------|--------|-------|
| /.env | 403 Forbidden | Protected |
| /.git/HEAD | 403 Forbidden | Protected |
| /wp-login.php | 404 Not Found | Not WordPress |
| /xmlrpc.php | 404 Not Found | Not WordPress |
| /robots.txt | 200 OK | Exposes structure |

### Robots.txt Analysis
```
Disallow: /*?* (dynamic params)
Disallow: /polls/, /articles/, /support/
Disallow: *.php*
Disallow: /?action=
Allow: /css/, /js/, /wl/, /site-images/
Sitemap: https://dateland.co.il/sitemap.xml.gz
```

### External Domains Discovered
- cdn.datesupport.net (JS/CSS hosting)
- accounts.google.com (OAuth)
- app.appsflyer.com (mobile attribution)
- Sister sites: 123date.me, blind-date.co.il, ahlam.net

### API Endpoints (from JS)
```javascript
REQUEST_API_HOST = ''; // Same origin
API_URL = dynamic
Endpoints use POST with JSON
```
No hardcoded secrets found in common.js.

### Google Site Verification
```
google-site-verification: 6f-nqgCUx69we3lRymJ8CWuqSmJvVUK28iNOyWhn6Ks
```

## Potential Attack Vectors

### 1. Missing Security Headers
- No CSP → XSS potential
- No X-Frame-Options → Clickjacking possible
- No HSTS → Downgrade attacks

### 2. Cookie Configuration
- Long-lived cookies (93M seconds = ~3 years)
- Session tracking via `tid` parameter
- No `SameSite` attribute visible

### 3. Information Disclosure
- Server version hidden (generic "nginx")
- JS files expose API structure
- Sitemap reveals internal paths: /polls_list, /relations, /success

### 4. Subdomain Enumeration Targets
Based on discovered patterns:
- cdn.datesupport.net
- m.dateland.co.il (mobile?)
- api.dateland.co.il (potential)
- admin.dateland.co.il (potential)

## Recommended Next Steps

1. **Subdomain Enumeration**: Use `subfinder` or `amass` on dateland.co.il
2. **Port Scanning**: Check for open ports (8080, 8443, 3306, etc.)
3. **CORS Testing**: Test for credential reflection
4. **Parameter Fuzzing**: Use `arjun` on discovered endpoints
5. **JS Secret Hunting**: Deep scan all JS files for tokens
6. **Wayback Machine**: Check for historical leaks

## Tools Used
- curl (header/file probing)
- grep (pattern extraction)
- Manual sitemap analysis

---
*Generated using recon-skills methodology*
