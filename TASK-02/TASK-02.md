# 🔐 Task-02 - Configure SFTP User Access

# 🎯 Objective

Configure **SFTP (Secure File Transfer Protocol)** access on **App Server 3** by creating a dedicated SFTP user. The user should:

* 👤 Be able to authenticate using a password.
* 📁 Be restricted to **SFTP access only** (no SSH shell access).
* 🔒 Improve server security by limiting user capabilities.

This is a common security practice in production environments where users need to transfer files without being granted full shell access.

---

# 🖥️ Server Details

📍 **Server:** App Server 3 (`stapp03`)

---

# 🛠️ Steps to Perform

### 🔹 Step 1: Login to App Server 3

```bash
ssh banner@stapp03
sudo -i
```

---

### 🔹 Step 2: Create the SFTP User

Create the user **james**, assign it to the existing **ftp** group, and create a home directory.

```bash
useradd -m -G ftp james
```

---

### 🔹 Step 3: Set the User Password

```bash
echo "james:B4zNgHA7Ya" | chpasswd
```

---

### 🔹 Step 4: Open SSH Configuration

```bash
vi /etc/ssh/sshd_config
```

---

### 🔹 Step 5: Ensure the Following Lines Exist

```text
PasswordAuthentication yes
Subsystem sftp internal-sftp
```

> If `Subsystem sftp` points to `/usr/libexec/openssh/sftp-server`, replace it with:

```text
Subsystem sftp internal-sftp
```

---

### 🔹 Step 6: Add the Following Block at the End of the File

```text
Match User james
    ForceCommand internal-sftp
    PasswordAuthentication yes
    ChrootDirectory %h
    PermitTunnel no
    AllowTcpForwarding no
    X11Forwarding no
```

---

### 🔹 Step 7: Set Correct Permissions for Chroot

The **ChrootDirectory** must be owned by **root** and not writable by the user.

```bash
chown root:root /home/james
chmod 755 /home/james
```

Create a writable directory inside the home directory:

```bash
mkdir /home/james/upload
chown james:ftp /home/james/upload
```

---

### 🔹 Step 8: Validate SSH Configuration

```bash
sshd -t
```

Expected Output:

```text
(no output indicates configuration is valid)
```

---

### 🔹 Step 9: Restart SSH Service

```bash
systemctl restart sshd
```

---

# ✅ Verification

### Verify the User Exists

```bash
id james
```

Expected Output:

```text
uid=...
gid=...
groups=...,ftp
```

---

### Verify SSH Service

```bash
systemctl status sshd
```

Expected Output:

```text
active (running)
```

---

### Test SFTP Login

```bash
sftp james@localhost
```

Enter the password:

```text
B4zNgHA7Ya
```

Expected Result:

```text
Connected to localhost.
sftp>
```

---

### Verify SSH Shell is Blocked

```bash
ssh james@localhost
```

Expected Result:

```text
This service allows sftp connections only.
Connection closed.
```

---

# 🌍 Real-World Use Cases

* 📂 Secure file exchange between clients and servers.
* ☁️ Uploading website files to production servers.
* 🗄️ Sharing backups with third-party vendors.
* 🔒 Restricting users from executing Linux commands.
* 🏢 Providing secure access to business partners without exposing the server.

---

# 🧠 Key Concepts

* 🔐 SFTP (Secure File Transfer Protocol)
* 🛡️ OpenSSH Server
* 👤 User Authentication
* 📁 Chroot Jail
* 🚫 ForceCommand
* 🔒 Password Authentication

---

# 🎤 Interview Questions

### ❓ Q1. What is the difference between FTP and SFTP?

| FTP               | SFTP                |
| ----------------- | ------------------- |
| Unencrypted       | Encrypted using SSH |
| Port 21           | Port 22             |
| Less Secure       | Highly Secure       |
| Separate Protocol | Runs over SSH       |

---

### ❓ Q2. What does `ForceCommand internal-sftp` do?

It forces the user to use **SFTP only**, preventing access to an interactive SSH shell.

---

### ❓ Q3. Why is `ChrootDirectory` used?

It confines the user to a specific directory, preventing access to the rest of the filesystem.

---

### ❓ Q4. Why must the chroot directory be owned by `root`?

OpenSSH requires the chroot directory to be owned by **root** and not writable by other users to prevent privilege escalation and maintain the integrity of the chroot environment.

---

### ❓ Q5. What is the difference between SSH and SFTP?

* **SSH** provides secure remote shell access to execute commands on a server.
* **SFTP** uses SSH for transport but is limited to secure file transfer operations.

---

# 📚 Commands Learned

```bash
useradd -m -G ftp james

echo "james:B4zNgHA7Ya" | chpasswd

vi /etc/ssh/sshd_config

sshd -t

systemctl restart sshd

id james

sftp james@localhost

ssh james@localhost

chown root:root /home/james

chmod 755 /home/james
```

---

# ⚠️ Common Mistakes

* ❌ Forgetting to restart the `sshd` service after making configuration changes.
* ❌ Incorrect indentation inside the `Match User` block (SSH configuration is indentation-sensitive for readability).
* ❌ Setting the chroot directory owner to the user instead of `root`.
* ❌ Forgetting to create a writable subdirectory inside the chroot.
* ❌ Not validating the configuration with `sshd -t` before restarting the service.

---

# 💡 Production Tips

* ✅ Use **SSH key authentication** instead of passwords whenever possible.
* ✅ Restrict users to SFTP-only access unless shell access is explicitly required.
* ✅ Disable unnecessary features like X11 forwarding and TCP forwarding for SFTP users.
* ✅ Monitor SFTP activity using server logs for auditing and troubleshooting.

---

# ⭐ Key Takeaways

* ✔️ Created a dedicated SFTP user named `james`.
* ✔️ Enabled password-based authentication.
* ✔️ Restricted the user to **SFTP-only** access using `ForceCommand internal-sftp`.
* ✔️ Configured a secure chroot environment.
* ✔️ Verified both SFTP functionality and SSH shell restrictions.

---

# 🚨 Troubleshooting

### ❌ Error: `Connection closed`

* Verify the `Match User` configuration in `/etc/ssh/sshd_config`.
* Run `sshd -t` to check for syntax errors.

---

### ❌ Error: `bad ownership or modes for chroot directory`

Ensure the home directory is owned by `root` and has appropriate permissions:

```bash
chown root:root /home/james
chmod 755 /home/james
```

---

### ❌ Error: `Permission denied`

* Confirm the password is correct.
* Verify `PasswordAuthentication yes` is enabled.
* Check that the `sshd` service has been restarted after configuration changes.

---

# 🎯 Interview Tip

Interviewers often ask how you would provide secure file transfer access to external users without giving them full shell access. Mention **SFTP-only access using `ForceCommand internal-sftp`**, **chroot jails**, and **least privilege** principles.

---

# 🏆 Difficulty Level

🟡 **Medium**
