# Configuring an Ubuntu Server with Nginx, HTTPS, and Firewall

## Introduction
This document provides a comprehensive guide to setting up an Ubuntu server (e.g., Ubuntu 20.04 or later) with Nginx as the web server, securing it with HTTPS using Let's Encrypt, and configuring the Uncomplicated Firewall (UFW) for security. This guide assumes root or sudo access and a registered domain name pointing to the server's public IP.

**Last Updated**: July 17, 2025

---

## Prerequisites
- Ubuntu server with root or sudo access.
- A registered domain name (e.g., `example.com`) with DNS configured to point to the server's public IP.
- Basic familiarity with SSH and terminal commands.

---

## 1. Update the System
Ensure the system is up-to-date to prevent compatibility issues.

1. Run the following command:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

---

## 2. Install Nginx
Nginx is a high-performance web server. Follow these steps to install and verify it.

1. **Install Nginx**:
   ```bash
   sudo apt install nginx -y
   ```

2. **Start and enable Nginx**:
   ```bash
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

3. **Verify Nginx status**:
   ```bash
   sudo systemctl status nginx
   ```
   Look for "active (running)" in the output.

4. **Test Nginx**:
   - Open a browser and navigate to `http://<server_ip>`.
   - You should see the default Nginx welcome page.
   - If inaccessible, ensure port 80 is open in your server's firewall or cloud provider's security group.

---

## 3. Configure Nginx
Nginx uses server blocks to manage websites. Configure a server block for your domain.

1. **Create a server block**:
   ```bash
   sudo nano /etc/nginx/sites-available/example.com
   ```

   Add the following configuration (replace `example.com` with your domain):
   ```nginx
   server {
      listen 80;
      server_name example.com www.example.com;

      # Redirect all HTTP traffic to HTTPS
      return 301 https://$host$request_uri;
   }

   server {
      listen 443 ssl http2;
      server_name example.com www.example.com;

      # SSL configuration (assumes Let's Encrypt certificates)
      ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
      include /etc/letsencrypt/options-ssl-nginx.conf;
      ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

      # SSL security settings
      ssl_protocols TLSv1.2 TLSv1.3;
      ssl_prefer_server_ciphers on;
      ssl_ciphers EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH;
      ssl_session_cache shared:SSL:10m;
      ssl_session_timeout 1d;
      ssl_session_tickets off;

      # Security headers
      add_header X-Frame-Options "SAMEORIGIN" always;
      add_header X-Content-Type-Options "nosniff" always;
      add_header X-XSS-Protection "1; mode=block" always;
      add_header Referrer-Policy "strict-origin-when-cross-origin" always;
      add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

      # Web root and index files
      root /var/www/example.com/html;
      index index.html index.htm;

      # Serve static files
      location / {
         try_files $uri $uri/ /index.html;
      }

      # Cache static assets
      location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg|eot|otf)$ {
         expires 365d;
         access_log off;
         add_header Cache-Control "public, immutable";
      }

      # Deny access to hidden files
      location ~ /\. {
         deny all;
      }

      # Error pages
      error_page 404 /404.html;
      location = /404.html {
         internal;
      }

      error_page 500 502 503 504 /50x.html;
      location = /50x.html {
         internal;
      }

      # Access and error logs
      access_log /var/log/nginx/example.com.access.log;
      error_log /var/log/nginx/example.com.error.log;
   }
   ```

2. **Create the web root directory**:
   ```bash
   sudo mkdir -p /var/www/example.com/html
   ```

3. **Add a sample HTML file**:
   ```bash
   sudo nano /var/www/example.com/html/index.html
   ```

   Add:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>Welcome to Example.com</title>
   </head>
   <body>
       <h1>Hello, Nginx!</h1>
   </body>
   </html>
   ```

4. **Set permissions**:
   ```bash
   sudo chown -R www-data:www-data /var/www/example.com
   sudo chmod -R 755 /var/www/example.com
   ```

5. **Enable the server block**:
   ```bash
   sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
   ```

6. **Test and reload Nginx**:
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

7. **Verify the site**:
   - Visit `http://example.com` in a browser.
   - You should see the "Hello, Nginx!" page.

---

## 4. Enable HTTPS with Let's Encrypt
Secure your website with a free SSL/TLS certificate from Let's Encrypt.

1. **Install Certbot**:
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   ```

2. **Obtain an SSL certificate**:
   ```bash
   sudo certbot --nginx -d example.com -d www.example.com
   ```

   - Provide an email address and agree to the terms.
   - Certbot will configure Nginx to redirect HTTP to HTTPS.

3. **Verify HTTPS**:
   - Visit `https://example.com`.
   - Confirm a secure connection (padlock icon in the browser).

4. **Set up automatic renewal**:
   - Test the renewal process:
     ```bash
     sudo certbot renew --dry-run
     ```
   - Verify the renewal timer:
     ```bash
     sudo systemctl status certbot.timer
     ```

---

## 5. Configure UFW Firewall
Use UFW to restrict traffic to only necessary ports.

1. **Install UFW** (if not already installed):
   ```bash
   sudo apt install ufw -y
   ```

2. **Allow necessary ports**:
   ```bash
   sudo ufw allow OpenSSH
   sudo ufw allow 'Nginx Full'
   ```

3. **Enable UFW**:
   ```bash
   sudo ufw enable
   ```

4. **Verify UFW status**:
   ```bash
   sudo ufw status
   ```

   Expected output:
   ```
   Status: active
   To                         Action      From
   --                         ------      ----
   OpenSSH                    ALLOW       Anywhere
   Nginx Full                 ALLOW       Anywhere
   ```

5. **(Optional) Deny other incoming traffic**:
   ```bash
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   ```

---

## 6. Additional Security and Optimization
Enhance Nginx security and performance with the following tweaks.

1. **Add security headers**:
   Edit `/etc/nginx/sites-available/example.com` and add:
   ```nginx
   add_header X-Frame-Options "SAMEORIGIN";
   add_header X-Content-Type-Options "nosniff";
   add_header X-XSS-Protection "1; mode=block";
   add_header Referrer-Policy "strict-origin-when-cross-origin";
   ```

2. **Hide Nginx version**:
   Edit `/etc/nginx/nginx.conf` and add in the `http` block:
   ```nginx
   server_tokens off;
   ```

3. **Optimize SSL settings**:
   Add to the server block:
   ```nginx
   ssl_protocols TLSv1.2 TLSv1.3;
   ssl_prefer_server_ciphers on;
   ssl_ciphers EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH;
   ```

4. **Test and reload Nginx**:
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

---

## 7. Monitoring and Maintenance
Regularly monitor and back up your server.

1. **Monitor logs**:
   - Access logs: `/var/log/nginx/access.log`
   - Error logs: `/var/log/nginx/error.log`
   ```bash
   tail -f /var/log/nginx/error.log
   ```

2. **Monitor server health**:
   ```bash
   sudo apt install htop -y
   htop
   ```

3. **Backup configurations**:
   ```bash
   sudo tar -czf nginx_backup.tar.gz /etc/nginx /var/www
   ```

---

## 8. Troubleshooting
- **Nginx fails to start**:
  - Check configuration: `sudo nginx -t`
  - View logs: `sudo journalctl -u nginx`
- **Firewall issues**:
  - Verify rules: `sudo ufw status`
  - Ensure cloud provider security groups allow ports 22, 80, and 443.
- **HTTPS not working**:
  - Check Certbot logs: `/var/log/letsencrypt/letsencrypt.log`
  - Verify DNS A record points to the server's IP.

---

## Conclusion
Your Ubuntu server is now configured with Nginx, secured with HTTPS via Let's Encrypt, and protected by a UFW firewall. The website is accessible at `https://example.com`. For further customization (e.g., adding PHP or Node.js), consult additional documentation or contact your system administrator.

**Note**: Replace `example.com` with your actual domain name throughout the process.

---

**Document Created**: July 17, 2025  
**Author**: Grok, xAI
