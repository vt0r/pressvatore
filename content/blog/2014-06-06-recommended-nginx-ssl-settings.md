---
date: 2014-06-06T16:24:10Z
disqus: true
share: true
title: Recommended Nginx SSL/TLS Settings
url: /2014/06/06/recommended-nginx-ssl-settings/
---

**UPDATE: 2024-01-12** - Defaulted to TLSv1.3 and newer OpenSSL options, but left older recommendations commented, just in case.
**UPDATE: 2019-01-30** - Added TLSv1.3 recommend ciphers. Also added another XSS prevention header.

---

Have you seen the recent (and not so recent) OpenSSL/TLS protocol vulnerabilities, and are you worried about securing connections to the Nginx servers you manage?

I'm going to assume you answered "yes" to that question, as it's unlikely you'd end up on this page otherwise. The whole purpose of this post is to share some high-security SSL configuration info for Nginx, so you've come to the right place.

**DISCLAIMER: YOU SHOULD NOT TRUST ME OR BLINDLY COPY AND PASTE MY CONFIGURATION INTO YOUR NGINX CONFIGURATION FILE WITHOUT UNDERSTANDING WHAT EACH OPTION DOES AND WHY EACH CONFIGURATION VALUE WAS CHOSEN. CONSIDER YOURSELF AND ANY ANIMALS HARMED BY USING MY CONFIGURATION SUFFICIENTLY WARNED**

Here is an example configuration template that should support _most_ browsers currently in use today, while preferring very high levels of security on modern browsers, and effectively mitigating most older/newer attacks:

```nginx
## HTTPS globals
ssl_protocols             TLSv1.2 TLSv1.3;
# Comment the above line and uncomment
# below two (2) for nginx versions older than 1.13.0
#ssl_protocols             TLSv1.2;
#ssl_ciphers               ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384;
# Uncomment the above one line and uncomment
# below one for OpenSSL versions older than 1.1.1
ssl_ciphers               TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384:TLS_AES_128_GCM_SHA256:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256;
ssl_prefer_server_ciphers on;
ssl_dhparam               /path/to/dh2048.pem; # >=2048-bits recommended
ssl_session_cache         shared:SSL:10m;
ssl_session_timeout       10m;
ssl_session_tickets       off;           # Requires nginx >= 1.5.9
ssl_ecdh_curve            secp384r1;     # Requires nginx >= 1.1.0
resolver                  1.1.1.1 9.9.9.9 valid=300s;
resolver_timeout          5s;
ssl_stapling              on;            # Requires nginx >= 1.3.7
ssl_stapling_verify       on;            # Requires nginx >= 1.3.7

## Headers to add inside server blocks
add_header Strict-Transport-Security "max-age=31536000";
# Add "; includeSubdomains" to the line above if all
# subdomains are secured with TLS.
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```

As you can verify by [checking 1a4.fr on the SSL Labs test site](https://www.ssllabs.com/ssltest/analyze.html?d=1a4.fr), these configuration options, combined with the latest stable versions of OpenSSL and Nginx, score an A+ while still allowing nearly all modern browsers that support TLS1.2 and up (and ECDH) to connect.

Some of the above options will not work in older versions of Nginx, and they've been marked with comments to give an idea of which versions are capable of using them.

Please feel free to send me any comments you may have on possible improvements to the configuration above.
