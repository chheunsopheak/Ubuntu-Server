# Installing MySQL and Creating a Database on Ubuntu Server

This guide provides a step-by-step walkthrough for installing the MySQL database server on an Ubuntu machine, securing it, creating a new database, and enabling remote root access if needed.

---

## ✅ Step 1: Install MySQL Server

1. **Update your package list:**
   ```bash
   sudo apt update
   ```

2. **Install the MySQL server:**
   ```bash
   sudo apt install mysql-server -y
   ```

3. **Verify the installation:**
   ```bash
   sudo systemctl status mysql
   ```
   You should see `active (running)`.

---

## ✅ Step 2: Secure Your MySQL Installation

After installation, it is crucial to run the MySQL security script to secure your server.

1.  **Run the security script:**
    ```bash
    sudo mysql_secure_installation
    ```

2.  **Follow the prompts:**
    * **VALIDATE PASSWORD COMPONENT:** Choose whether to enforce a password policy.
    * **Set root password:** Create a strong password for the MySQL `root` user.
    * **Remove anonymous users:** Type `Y` and press Enter.
    * **Disallow root login remotely:** Type `Y` and press Enter.
    * **Remove test database and access to it:** Type `Y` and press Enter.
    * **Reload privilege tables:** Type `Y` and press Enter to apply your changes.

---

## ✅ Step 3: Create a New Database

1. **Log into MySQL shell:**
   ```bash
   sudo mysql -u root -p
   ```

2. **Create a database:**
   ```sql
   CREATE DATABASE your_database_name;
   ```

3. **Confirm:**
   ```sql
   SHOW DATABASES;
   ```

4. **Exit:**
   ```sql
   exit;
   ```

---

## ✅ Step 4: Allow Remote Connections (Optional / Dev Use)

### 4.1 Update MySQL bind address

Open the MySQL config file:

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Change:
```
bind-address = 127.0.0.1
```

To:
```
bind-address = 0.0.0.0
```

Restart MySQL:
```bash
sudo systemctl restart mysql
```

---

## ✅ Step 5: Enable Remote Root Access (Dev Only)

> ⚠️ **Not recommended for production.** Use a limited-access user instead.

1. **Log in to MySQL:**
   ```bash
   sudo mysql -u root -p
   ```

2. **Allow root from any IP:**
   ```sql
   CREATE USER 'root'@'%' IDENTIFIED BY 'your_password';
   GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
   FLUSH PRIVILEGES;
   ```

---

## ✅ Step 6: Fix Public Key Retrieval Error (DBeaver/JDBC)

If you see this error:
```
Public Key Retrieval is not allowed
```

### Option A: Update JDBC URL (dev only)
In DBeaver or your app:
```
jdbc:mysql://your-server-ip:3306/your_database?useSSL=false&allowPublicKeyRetrieval=true
```

### Option B: Use `mysql_native_password`

If you prefer a compatible authentication plugin:

1. **Load plugin if needed:**
   ```sql
   INSTALL PLUGIN mysql_native_password SONAME 'auth_native.so';
   ```

2. **Set the auth plugin:**
   ```sql
   ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'your_password';
   FLUSH PRIVILEGES;
   ```

---

## ✅ Step 7: Open MySQL Port in UFW (Firewall)

If UFW is active, allow MySQL:

```bash
sudo ufw allow 3306/tcp
sudo ufw reload
```

Also ensure your cloud provider firewall allows inbound traffic on port `3306` for your IP.

---

## ✅ Step 8: Verify Remote Connection

From DBeaver or another client:
- Host: your server IP
- Port: 3306
- User: `root` or your user
- Password: set above

Test the connection.
