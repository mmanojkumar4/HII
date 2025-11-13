

---

# **README – User Management Automation Script (`create_users.sh`)**

## **1. Purpose and Design of the Script**

The purpose of `create_users.sh` is to automate the creation and setup of user accounts for new employees in a Linux environment. Instead of manually creating each user, adding groups, setting passwords, and configuring home directories, this script automates the entire process using information from a simple text file.

Each line in the input file follows the format:

```
username;group1,group2,group3
```

### **Design Highlights**

* Reads user information line-by-line and ignores empty lines or comments (lines starting with `#`).
* Splits each line into a username and a list of additional groups.
* Automatically creates missing groups.
* Creates the user (or updates if already exists) and assigns the necessary groups.
* Creates the home directory `/home/username` if it doesn’t exist, and applies correct permissions.
* Generates a random 12-character password for each user.
* Sets the user’s password and stores credentials securely in `/var/secure/user_passwords.txt` with **600 permissions**.
* Logs all successes, errors, and skipped entries to `/var/log/user_management.log` with **600 permissions**.
* Displays clear messages through a custom logging function.

The script ensures consistency, reduces human errors, and provides an audit trail through logs.

---

## **2. Step-by-Step Explanation**

Below is a simple step-by-step breakdown of what the script does:

### **Step 1: Read the input file**

* Reads each line from the file you pass as an argument.
* Example line:
  `manoj;dev,www-data`

### **Step 2: Ignore unnecessary lines**

* Skips:

  * Empty lines
  * Lines starting with `#`

### **Step 3: Clean and split each line**

* Removes all whitespace.
* Splits line into:

  * **Username** (before `;`)
  * **Groups** (after `;`)

### **Step 4: Create supplementary groups**

* Takes the comma-separated groups.
* Checks if each group already exists.
* Creates missing groups using `groupadd`.

### **Step 5: Create or update the user**

* If the user already exists:

  * The script adds new groups using `usermod`.
* If the user does NOT exist:

  * Creates the user with a home directory and assigns groups.

### **Step 6: Home directory setup**

* Ensures `/home/username` exists.
* Sets correct owner (`username:username`).
* Sets permissions to `700` (private home directory).

### **Step 7: Generate and apply password**

* Generates a random 12-character password.
* Sets password using `chpasswd`.
* Stores `username:password` securely in `/var/secure/user_passwords.txt`.

### **Step 8: Logging**

* Every action is logged with a timestamp to:

  * `/var/log/user_management.log`

### **Step 9: Finalization**

* After processing all users, the script applies correct permissions and logs completion.

---

## **3. Example Usage**

### **1. Create an input file**

`users.txt`

```
# New employees
light; sudo,dev,www-data
siyoni; sudo
manoj; dev,www-data
```

### **2. Make the script executable**

```bash
chmod +x create_users.sh
```

### **3. Run the script as root**

```bash
sudo ./create_users.sh users.txt
```

### **4. Check logs**

```bash
sudo cat /var/log/user_management.log
```

### **5. Check generated passwords**

```bash
sudo cat /var/secure/user_passwords.txt
```

---

## **4. Security Considerations**

### **Sensitive Password File**

* Passwords are stored in **plaintext** in:

  ```
  /var/secure/user_passwords.txt
  ```
* This file has **600 permissions** so only `root` can read it.
* In real environments, consider:

  * Encrypting this file
  * Using a secret manager (Vault, AWS Secrets Manager)
  * Deleting the file after distribution

### **Least Privilege**

* Only `root` should run the script.
* Log and password files are protected using:

  ```bash
  chmod 600
  ```

### **Password Strength**

* Passwords are generated using `/dev/urandom`.
* 12 characters including symbols provides good strength by default.

### **Audit Logging**

* All actions are logged to:

  ```
  /var/log/user_management.log
  ```
* This ensures traceability for compliance and auditing.

### **Home Directory Permissions**

* Home directories use:

  ```
  chmod 700
  ```
* This prevents other users from reading another user's files.

---



