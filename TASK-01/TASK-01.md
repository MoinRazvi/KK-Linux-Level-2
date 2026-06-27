# 🐧 Linux Level-2 | Task-01 🌐 Apache Port & URL Redirects

# 🎯 Objective

Configure the Apache web server to:

* 🌐 Listen on **port 5000** instead of the default port.
* 🔁 Redirect **non-www** requests to **www** using a **Permanent Redirect (301)**.
* 🔁 Redirect **/blog/** to **/news/** using a **Temporary Redirect (302)**.

This task demonstrates how Apache handles URL redirection, which is commonly used during website migrations, SEO improvements, and URL restructuring.

---

# 🖥️ Server

📍 **App Server 1**

---

# 🛠️ Steps to Perform

### Step 1️⃣ - Login to App Server 1

```bash
ssh tony@stapp01
sudo -i
```

---

### Step 2️⃣ - Edit Apache Configuration

On CentOS/RHEL:

```bash
vi /etc/httpd/conf/httpd.conf
```

---

### Step 3️⃣ - Change Apache Listening Port

Find:

```apache
Listen 80
```

Change it to:

```apache
Listen 5000
```

---

### Step 4️⃣ - Add Redirect Rules

At the end of the file, add:

```apache
<VirtualHost *:5000>

    ServerName stapp01.stratos.xfusioncorp.com

    Redirect permanent / http://www.stapp01.stratos.xfusioncorp.com:5000/

</VirtualHost>

<VirtualHost *:5000>

    ServerName www.stapp01.stratos.xfusioncorp.com

    Redirect temp /blog/ http://www.stapp01.stratos.xfusioncorp.com:5000/news/

</VirtualHost>
```

---

### Step 5️⃣ - Check Apache Configuration

```bash
httpd -t
```

Expected Output

```text
Syntax OK
```

---

### Step 6️⃣ - Restart Apache

```bash
systemctl restart httpd
```

---

### Step 7️⃣ - Enable Service (Optional)

```bash
systemctl enable httpd
```

---

# ✅ Verification

Check Apache is listening on port **5000**

```bash
ss -tlnp | grep 5000
```

Expected

```text
LISTEN 0 128 *:5000
```

---

Verify Service

```bash
systemctl status httpd
```

Expected

```text
active (running)
```

---

# 🌍 Understanding Redirects

## 🔹 Permanent Redirect (301)

```apache
Redirect permanent / http://www.stapp01.stratos.xfusioncorp.com:5000/
```

Equivalent to

```apache
Redirect 301 / http://www.stapp01.stratos.xfusioncorp.com:5000/
```

✔ Browser caches the redirect

✔ Search engines update their index

✔ Best for permanent website changes

---

## 🔹 Temporary Redirect (302)

```apache
Redirect temp /blog/ http://www.stapp01.stratos.xfusioncorp.com:5000/news/
```

Equivalent to

```apache
Redirect 302 /blog/ http://www.stapp01.stratos.xfusioncorp.com:5000/news/
```

✔ Browser does not permanently cache

✔ Search engines continue indexing the old URL

✔ Useful during maintenance or testing

---

# 💡 Real-World Use Cases

✅ Redirect old URLs after website migration.

✅ Force users to use the **www** version of a website.

✅ Redirect traffic during maintenance.

✅ Improve SEO by avoiding duplicate content.

✅ Change application URLs without breaking bookmarks.

---

# 🧠 Key Concepts

* 🌐 Apache Virtual Hosts
* 🔄 URL Redirection
* 🔌 Listening Ports
* 📡 HTTP Status Codes (301 & 302)
* ⚙️ Apache Configuration Validation

---

# 🎤 Interview Questions

### Q1. What is the difference between 301 and 302 redirects?

**301 (Permanent Redirect):**

* Browser caches the redirect.
* Search engines transfer SEO ranking to the new URL.
* Used when the URL has permanently changed.

**302 (Temporary Redirect):**

* Browser does not permanently cache the redirect.
* Search engines keep indexing the original URL.
* Used for temporary changes like maintenance or testing.

---

### Q2. Why do we use VirtualHost in Apache?

Virtual Hosts allow a single Apache server to host multiple websites or domains, each with its own configuration.

---

### Q3. How do you check Apache configuration before restarting?

```bash
httpd -t
```

Always validate the configuration to avoid downtime caused by syntax errors.

---

### Q4. How do you check which port Apache is listening on?

```bash
ss -tlnp | grep httpd
```

or

```bash
ss -tlnp | grep 5000
```

---

### Q5. Where is Apache's main configuration file located on CentOS/RHEL?

```text
/etc/httpd/conf/httpd.conf
```

---

### Q6. What happens if two services try to use the same port?

The second service will fail to start because only one process can bind to a specific IP:port combination at a time.

---

# 📚 Commands Learned

```bash
vi /etc/httpd/conf/httpd.conf

httpd -t

systemctl restart httpd

systemctl status httpd

systemctl enable httpd

ss -tlnp | grep 5000
```

---

# ⭐ Key Takeaways

✔ Changed Apache listening port from **80** to **5000**.

✔ Configured a **301 Permanent Redirect** from **non-www** to **www**.

✔ Configured a **302 Temporary Redirect** from **/blog/** to **/news/**.

✔ Validated the configuration before restarting Apache.

✔ Learned the practical difference between **301** and **302** redirects.

---

# ⚠️ Common Mistakes

❌ Forgetting to change the `Listen` directive to the required port.

❌ Restarting Apache without running `httpd -t`.

❌ Using `Redirect permanent` when a temporary redirect is required.

❌ Forgetting to include the correct port (`:5000`) in the redirect URL.

❌ Defining overlapping `VirtualHost` configurations that cause unexpected behavior.

---

# 💼 Production Tips

* 🛡️ Always test configuration changes with `httpd -t` before restarting.
* 📋 Use 301 redirects only for permanent URL changes to preserve SEO.
* 🔍 Keep redirect rules simple and avoid redirect loops.
* 📖 Document all redirect changes, especially during website migrations.

---

# 🏆 Difficulty Level

🟡 **Medium**

This task requires understanding Apache configuration, Virtual Hosts, listening ports, and the correct use of HTTP redirect status codes—skills commonly expected of Linux administrators and DevOps engineers.
