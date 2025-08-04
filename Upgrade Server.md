
### Scenario 2: Upgrade to OpenJDK 21 (Recommended)

This procedure utilizes the `apt` package manager for a more streamlined, maintainable, and secure installation.

1.  **Update Package List:**
    Ensure the local package index is up-to-date.
    ```bash
    sudo apt update
    ```

2.  **Install OpenJDK 21:**
    Install the OpenJDK 21 Development Kit from the Ubuntu repositories.
    ```bash
    sudo apt install openjdk-21-jdk
    ```

3.  **Configure System Default Java:**
    Use `update-alternatives` to switch the default Java version to OpenJDK 21.
    ```bash
    sudo update-alternatives --config java
    sudo update-alternatives --config javac
    ```
    Select the number corresponding to `java-21-openjdk-amd64` from the list.

4.  **Set `JAVA_HOME` Environment Variable:**
    Edit `/etc/profile` and set `JAVA_HOME` to the OpenJDK 21 path.
    ```bash
    sudo nano /etc/profile
    ```
    Add or update the lines:
    ```bash
    export JAVA_HOME="/usr/lib/jvm/java-21-openjdk-amd64"
    export PATH=$PATH:$JAVA_HOME/bin
    ```
    Save and close the file.
    Save and exit nano:
    1. Press Ctrl + O to save
    2. ress Enter to confirm
    3. Press Ctrl + X to exit

5.  **Apply Changes and Verify:**
    Reload the profile and confirm the new version.
    ```bash
    source /etc/profile
    java -version
    echo $JAVA_HOME
    ```

---