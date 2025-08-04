# Nginx Commands for Ubuntu Server Management

This document provides a comprehensive list of commands for managing an Nginx web server on an Ubuntu server, including file management commands like `sudo rm` and other essential tasks for configuration, maintenance, and troubleshooting. Each command includes its purpose and notes for safe usage.

## Prerequisites
- **Ubuntu Server**: Ensure you have administrative access (sudo privileges).
- **Nginx Installed**: Install Nginx if not already present:
  ```bash
  sudo apt update
  sudo apt install -y nginx
  ```
- **Access**: Commands are executed via SSH or terminal on the Ubuntu server.

## 1. Installing and Updating Nginx
```bash
sudo apt update
sudo apt install -y nginx
```
- **Purpose**: Updates package lists and installs Nginx.
- **Note**: Run `sudo apt update` regularly to keep package lists current. Use `-y` to auto-confirm installation.

```bash
sudo apt upgrade -y nginx
```
- **Purpose**: Upgrades Nginx to the latest version.
- **Note**: Ensure Nginx is stopped before upgrading to avoid conflicts:
  ```bash
  sudo systemctl stop nginx
  ```

## 2. Managing Nginx Service
```bash
sudo systemctl start nginx
```
- **Purpose**: Starts the Nginx service.
- **Note**: Use after installation or to restart after configuration changes.

```bash
sudo systemctl stop nginx
```
- **Purpose**: Stops the Nginx service.
- **Note**: Useful before making major configuration changes or upgrades.

```bash
sudo systemctl restart nginx
```
- **Purpose**: Restarts Nginx, applying configuration changes.
- **Note**: Preferred over `reload` for significant changes.

```bash
sudo systemctl reload nginx
```
- **Purpose**: Reloads Nginx configuration without interrupting active connections.
- **Note**: Use for minor configuration updates.

```bash
sudo systemctl status nginx
```
- **Purpose**: Checks if Nginx is running and displays its status.
- **Note**: Look for "active (running)" in the output to confirm Nginx is operational.

## 3. Configuring Nginx
```bash
sudo nano /etc/nginx/nginx.conf
```
- **Purpose**: Opens the main Nginx configuration file for editing.
- **Note**: Use a text editor like `nano` or `vim`. Save and exit with `Ctrl+O`, `Enter`, `Ctrl+X` in `nano`. Always test configuration after editing:
  ```bash
  sudo nginx -t
  ```

```bash
sudo nano /etc/nginx/sites-available/default
```
- **Purpose**: Edits the default server block configuration.
- **Note**: Modify this file to configure virtual hosts or server settings. Link to `sites-enabled` if needed:
  ```bash
  sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
  ```

```bash
sudo nginx -t
```
- **Purpose**: Tests Nginx configuration for syntax errors.
- **Note**: Run before restarting Nginx to ensure configurations are valid.

## 4. Managing Nginx Configuration Files with `sudo rm`
```bash
sudo rm /etc/nginx/sites-enabled/old_config
```
- **Purpose**: Deletes an old or unused configuration file from `sites-enabled`.
- **Note**: Use with caution. Verify the file is no longer needed with `ls /etc/nginx/sites-enabled/`. Always test configuration after deletion:
  ```bash
  sudo nginx -t
  ```

```bash
sudo rm -rf /etc/nginx/sites-available/old_site
```
- **Purpose**: Deletes an unused site configuration from `sites-available`.
- **Note**: The `-rf` flag removes files and directories recursively. Double-check with `ls` to avoid accidental deletion.

```bash
sudo rm -rf /var/www/old_site
```
- **Purpose**: Deletes an old website's files from the web root.
- **Note**: Ensure the site is no longer needed. Back up important files before deletion:
  ```bash
  cp -r /var/www/old_site /var/www/old_site_backup
  ```

## 5. Managing Logs
```bash
sudo cat /var/log/nginx/access.log
```
- **Purpose**: Views the Nginx access log.
- **Note**: Useful for monitoring traffic and debugging requests.

```bash
sudo cat /var/log/nginx/error.log
```
- **Purpose**: Views the Nginx error log.
- **Note**: Check for errors if a site fails to load or Nginx crashes.

```bash
sudo rm -f /var/log/nginx/access.log /var/log/nginx/error.log
sudo systemctl restart nginx
```
- **Purpose**: Clears Nginx access and error logs.
- **Note**: Logs are recreated after restarting Nginx. Use to free up space or reset logs for troubleshooting.

## 6. Setting Up HTTPS with Let's Encrypt
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx
```
- **Purpose**: Installs Certbot and configures HTTPS for Nginx using Let's Encrypt.
- **Note**: Requires a domain name. Follow prompts to set up SSL. If no domain, use a self-signed certificate:
  ```bash
  sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/selfsigned.key -out /etc/ssl/certs/selfsigned.crt
  ```

## 7. Managing Web Content
```bash
sudo mkdir -p /var/www/my_site/html
sudo chown -R www-data:www-data /var/www/my_site
sudo chmod -R 755 /var/www/my_site
```
- **Purpose**: Creates a directory for a new website, sets ownership to the Nginx user (`www-data`), and applies appropriate permissions.
- **Note**: Replace `my_site` with your site name. Ensure Nginx can read the files.

```bash
sudo rm -rf /var/www/my_site/html/*
```
- **Purpose**: Deletes all files in the website's directory.
- **Note**: Use with caution. Back up important files before deletion.

## 8. Firewall Configuration
```bash
sudo apt install -y ufw
sudo ufw allow 'Nginx Full'
sudo ufw enable
```
- **Purpose**: Installs and configures UFW to allow HTTP and HTTPS traffic for Nginx.
- **Note**: Check firewall status with `sudo ufw status`. Ensure SSH is allowed to avoid lockout:
  ```bash
  sudo ufw allow OpenSSH
  ```

## 9. Testing and Debugging
```bash
curl http://localhost
```
- **Purpose**: Tests if Nginx is serving content locally.
- **Note**: Use the server's IP or domain for external testing, e.g., `curl http://68.183.177.52`.

```bash
sudo netstat -tulnp | grep nginx
```
- **Purpose**: Checks which ports Nginx is listening on.
- **Note**: Requires `net-tools` (`sudo apt install -y net-tools`). Look for ports 80 (HTTP) or 443 (HTTPS).

## 10. Backing Up Nginx Configuration
```bash
sudo cp -r /etc/nginx /etc/nginx_backup
```
- **Purpose**: Backs up the entire Nginx configuration directory.
- **Note**: Restore with `sudo cp -r /etc/nginx_backup /etc/nginx` if needed. Test configuration after restoration.

## Troubleshooting Tips
- **Configuration Errors**: Always run `sudo nginx -t` after modifying configuration files.
- **Permission Issues**: Ensure files in `/var/www/` are owned by `www-data` and have correct permissions (`755` for directories, `644` for files).
- **Log Rotation**: Use `logrotate` to manage log sizes:
  ```bash
  sudo nano /etc/logrotate.d/nginx
  ```
- **Service Not Starting**: Check status (`sudo systemctl status nginx`) and logs (`/var/log/nginx/error.log`) for errors.

## Additional Notes
- **Safety with `sudo rm`**: Always verify paths before using `sudo rm` or `sudo rm -rf` to avoid accidental deletion of critical files. Use `ls` to list contents first.
- **Regular Maintenance**: Update Nginx and Ubuntu packages regularly (`sudo apt update && sudo apt upgrade -y`).
- **Documentation**: Keep this file updated with any custom commands or configurations specific to your server setup.

For further assistance, consult the [Nginx documentation](https://nginx.org/en/docs/) or contact your server administrator.