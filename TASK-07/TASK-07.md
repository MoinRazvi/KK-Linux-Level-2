# 🔐 Task-07 - Secure an Apache Directory Using `.htaccess` (Basic Authentication)

# 🎯 Objective

Secure a confidential directory on **App Server 2** using **Apache Basic Authentication**.

The requirements are:

* 📁 Create the `/var/www/html/security` directory.
* 👤 Create an authenticated user **kirsty** with the password **LQfKeWWxWD**.
* 📄 Copy the provided `index.html` file into the secured directory.
* 🌐 Ensure the website is accessible at:

```text
http://stapp02/security/
```

Only authenticated users should be able to access the content.

---

# 🖥️ Server Details

📍 **Target Server:** App Server 2 (`stapp02`)

📍 **Source File:** Jump Host

```text
/tmp/index.html
```

📍 **Destination**

```text
/var/www/html/security
```

---

# 🛠️ Steps to Perform

## 🔹 Step 1: Copy Website Files

From the **Jump Host**:

```bash
scp /tmp/index.html steve@stapp02:/tmp/
```

---

## 🔹 Step 2: Login to App Server 2

```bash
ssh steve@stapp02
sudo -i
```

---

## 🔹 Step 3: Create the Directory

```bash
mkdir -p /var/www/html/security
```

---

## 🔹 Step 4: Copy the Website

```bash
cp /tmp/index.html /var/www/html/security/
```

---

## 🔹 Step 5: Create the Password File

Create the user **kirsty**.

```bash
htpasswd -cb /etc/httpd/.htpasswd kirsty LQfKeWWxWD
```

### Explanation

* `-c` → Create a new password file.
* `-b` → Read the password from the command line.

> ⚠️ If the file already exists, **do not use `-c`**, as it overwrites the file.

To add additional users later:

```bash
htpasswd /etc/httpd/.htpasswd username
```

---

## 🔹 Step 6: Create `.htaccess`

```bash
vi /var/www/html/security/.htaccess
```

Add:

```apache
AuthType Basic
AuthName "Restricted Area"
AuthUserFile /etc/httpd/.htpasswd
Require valid-user
```

---

## 🔹 Step 7: Allow `.htaccess`

Open Apache configuration:

```bash
vi /etc/httpd/conf/httpd.conf
```

Find:

```apache
<Directory "/var/www/html">
```

Change:

```apache
AllowOverride None
```

to

```apache
AllowOverride All
```

This enables Apache to process `.htaccess` files.

---

## 🔹 Step 8: Restart Apache

```bash
systemctl restart httpd
```

---

# ✅ Verification

## Verify Password File

```bash
cat /etc/httpd/.htpasswd
```

Expected:

```text
kirsty:xxxxxxxxxxxxxxxx
```

(The password is stored as a hash.)

---

## Verify Apache

```bash
systemctl status httpd
```

Expected:

```text
active (running)
```

---

## Verify Authentication

From the **Jump Host**:

```bash
curl http://stapp02/security/
```

Expected:

```text
401 Authorization Required
```

---

Authenticate:

```bash
curl -u kirsty:LQfKeWWxWD http://stapp02/security/
```

Expected:

```html
<html>
...
```

---

# 🌍 Real-World Use Cases

* 🔒 Protecting admin portals.
* 📂 Restricting access to confidential reports.
* 📊 Securing monitoring dashboards.
* 🧾 Limiting access to internal documentation.
* 🏢 Protecting staging environments.

---

# 🧠 Key Concepts

* 🔐 Apache Basic Authentication
* 📄 `.htaccess`
* 👤 `htpasswd`
* 🔑 Password Hashing
* 📂 Directory-Level Security

---

# 🎤 Interview Questions

### ❓ Q1. What is `.htaccess`?

`.htaccess` is a per-directory Apache configuration file used to control access, URL rewriting, authentication, and other settings without modifying the main Apache configuration.

---

### ❓ Q2. What is `htpasswd`?

`htpasswd` creates and manages username/password pairs for Apache Basic Authentication.

---

### ❓ Q3. Why is `AllowOverride All` required?

Without it, Apache ignores `.htaccess` files, so the authentication rules inside `.htaccess` will not be applied.

---

### ❓ Q4. Where should the `.htpasswd` file be stored?

Outside the web document root whenever possible (e.g., `/etc/httpd/.htpasswd`) to prevent direct web access.

---

### ❓ Q5. Is Basic Authentication secure?

Basic Authentication sends credentials encoded in Base64, **not encrypted**. It should always be used together with **HTTPS** to protect credentials in transit.

---

# 📚 Commands Learned

```bash
mkdir -p /var/www/html/security

htpasswd -cb /etc/httpd/.htpasswd kirsty LQfKeWWxWD

cp /tmp/index.html /var/www/html/security/

systemctl restart httpd

curl

curl -u

AllowOverride All
```

---

# ⚠️ Common Mistakes

* ❌ Forgetting to enable `AllowOverride All`.
* ❌ Placing `.htpasswd` inside `/var/www/html`.
* ❌ Using `-c` with `htpasswd` when adding additional users (overwrites existing users).
* ❌ Restarting Apache without verifying the configuration.
* ❌ Forgetting to copy `index.html`.

---

# 💡 Production Tips

* ✅ Store `.htpasswd` outside the document root.
* ✅ Always protect Basic Authentication with HTTPS.
* ✅ Use strong passwords and periodically rotate credentials.
* ✅ Restrict only the required directories instead of the entire website.

---

# 🚨 Troubleshooting

### ❌ Authentication Prompt Doesn't Appear

Check:

```bash
grep AllowOverride /etc/httpd/conf/httpd.conf
```

Ensure it is:

```apache
AllowOverride All
```

---

### ❌ 403 Forbidden

Verify directory permissions:

```bash
ls -ld /var/www/html/security
```

---

### ❌ Password Not Working

Recreate the user:

```bash
htpasswd /etc/httpd/.htpasswd kirsty
```

---

### ❌ Apache Doesn't Start

Validate configuration:

```bash
httpd -t
```

---

# 🎯 Interview Tip

A common interview question is:

> **"Would you use `.htaccess` or the main Apache configuration for authentication?"**

For **small directory-specific changes**, `.htaccess` is convenient.

For **production environments**, configuring authentication directly in the main Apache configuration is preferred because it offers better performance by avoiding per-request `.htaccess` lookups.

---

# ⭐ Key Takeaways

* ✔️ Secured a directory using Apache Basic Authentication.
* ✔️ Created a password-protected user with `htpasswd`.
* ✔️ Enabled `.htaccess` processing using `AllowOverride All`.
* ✔️ Deployed a website into the protected directory.
* ✔️ Verified access with authenticated and unauthenticated requests.

---

# 🏆 Difficulty Level

🟡 **Medium**

---

