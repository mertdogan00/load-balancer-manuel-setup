
# 🌐 NGINX Load Balancer Setup Guide 🚦  
*A complete all-in-one reverse proxy guide with SSL, Cloudflare IP restoration, and trusted backend communication.* 🔒

---

## 📚 Table of Contents  
- [Install NGINX & Certbot](#1️⃣-install-nginx--certbot)  
- [HTTP Backend with HTTPS Frontend](#2️⃣-http-backend-with-https-frontend)  
- [HTTPS Backend with Root/Intermediate CA](#3️⃣-https-backend-with-rootintermediate-ca)  
- [Enable & Verify Configuration](#4️⃣-enable--verify-configuration)  
- [SSL Certificate Options](#5️⃣-ssl-certificate-options-lets-encrypt)  
- [Verify End-to-End Communication](#6️⃣-verify-end-to-end-communication)  
- [SSL Toolkit for Custom CA](#7️⃣-ssl-toolkit-for-custom-ca)  
- [General NGINX Configuration](#8️⃣-general-nginx-configuration)  
- [Extra Resources](#9️⃣-extra-resources)

---

## 1️⃣ Install NGINX & Certbot

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx -y
sudo apt install certbot python3-certbot-nginx -y
```

---

## 2️⃣ HTTP Backend with HTTPS Frontend

```nginx
upstream example_backend {
    server 192.168.1.10:80;
    server 192.168.1.11:80;
    server 192.168.1.12:80;
}
```

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name example.com www.example.com;

    # Request Limit (Optional)
    # limit_req zone=client_limit burst=15 nodelay;

    # if ($limit_bypass = 0) { # for bypass (optional)
    #  limit_req zone=client_limit burst=15 nodelay;
    #  limit_req_status 429; # Too Many Requests
    # }

    # Bots Off (Optional)
    # add_header X-Robots-Tag "noindex, nofollow, nosnippet, noarchive" always;

    # robots.txt Special (Optional)
    # If you want to serve robots.txt directly without proxying, enable this block.
    # Useful if you need a visible robots.txt file for bots.
    # Example:
    # location = /robots.txt {
    #     root /var/www/html;
    # }

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://example_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

       # proxy_set_header X-Real-IP $http_cf_connecting_ip;  # Optional. Real IP if no general CF config.

    }
}
```

---

## 3️⃣ HTTPS Backend with Root/Intermediate CA

```nginx
upstream example_backend {
    server 192.168.1.10:443;
    server 192.168.1.11:443;
    server 192.168.1.12:443;
}

server {
    listen 443 ssl;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass https://example_backend;

        proxy_ssl_verify on;
        proxy_ssl_certificate /etc/ssl/nginx/load_balancer.pem;
        proxy_ssl_certificate_key /etc/ssl/nginx/load_balancer.key;
        proxy_ssl_trusted_certificate /etc/ssl/nginx/ca_chain.pem;
        proxy_ssl_verify_depth 2;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 4️⃣ Enable & Verify Configuration

```bash
sudo mkdir -p /etc/nginx/ssl
sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 5️⃣ SSL Certificate Options (Let's Encrypt)

```bash
sudo certbot --nginx -d example.com -d www.example.com  # Automatic
sudo certbot certonly --nginx -d example.com            # Manual (NGINX)
sudo certbot certonly --webroot -w /var/www/html -d example.com
sudo certbot renew --dry-run
sudo systemctl status certbot.timer
```

---

## 6️⃣ Verify End-to-End Communication

```bash
curl -vk https://example.com
curl --cacert /etc/nginx/ssl/ca_chain.pem https://192.168.1.10
```

---

## 7️⃣ SSL Toolkit for Custom CA

Use the following repo to generate your own **Root + Intermediate CA** chain for backend trust:

🔗 [mertdogan00/ssl-and-ca-toolkit](https://github.com/mertdogan00/ssl-and-ca-toolkit)

---

## 8️⃣ General NGINX Configuration

For full Cloudflare compatibility, advanced gzip, and enhanced logging, see:  
📁 [`nginx.conf`](./nginx.conf)

Highlights:
- ✅ Real IP headers restored via Cloudflare ranges
- 📊 Custom logging for real client IPs & security insights
- ⚙ Optimal performance with gzip, SSL ciphers, and HTTP tuning

---

## 9️⃣ Extra Resources

- 📘 [Ultimate Server Setup](https://github.com/mertdogan00/ultimate-server-setup) – Webmin + 2FA + SSH Hardening  
- 🔐 [Certbot Self-Hosted SSL](https://github.com/mertdogan00/certbot-self-hosted-ssl) – Let's Encrypt & Cloudflare DNS

---

🎯 Your NGINX reverse proxy is now fully secure, optimized, and scalable!
