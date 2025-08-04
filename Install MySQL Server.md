# Installing MySQL and Creating a Database on Ubuntu Server

This guide provides a step-by-step walkthrough for installing the MySQL database server on an Ubuntu machine and creating a new database.

---

### Step 1: Install MySQL Server

First, you need to install the MySQL server package on your Ubuntu server.

1.  **Update your package list:**
    ```bash
    sudo apt update
    ```

2.  **Install the MySQL server:**
    ```bash
    sudo apt install mysql-server -y
    ```

3.  **Verify the installation:** Check that the MySQL service is running correctly.
    ```bash
    sudo systemctl status mysql
    ```
    You should see an output indicating `active (running)`.

### Step 2: Secure Your MySQL Installation

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

### Step 3: Create a New Database

Now that MySQL is installed and secured, you can create a new database.

1.  **Log in to the MySQL shell:** You will need to use the `root` user to perform this administrative task.
    ```bash
    sudo mysql -u root -p
    ```
    Enter the `root` password you set in the previous step when prompted.

2.  **Create the database:** Use the `CREATE DATABASE` command, replacing `your_database_name` with your desired name.
    ```sql
    CREATE DATABASE your_database_name;
    ```

3.  **Verify the creation:** You can list all databases to confirm your new database exists.
    ```sql
    SHOW DATABASES;
    ```

4.  **Exit the MySQL shell:**
    ```sql
    exit;
    ```

---

**Important Security Note:** For production applications, it is a best practice to create a dedicated user with specific privileges for your new database instead of using the `root` user. This minimizes the risk of a security breach.