# 🚀 Task-06 - Configure Nginx as a Reverse Proxy for Apache

# 🎯 Objective

Configure a **reverse proxy architecture** on the **Nautilus Backup Server**.

The requirements are:

* 🌐 Install **Apache HTTP Server** and configure it to listen on **port 8085**.
* ⚡ Install **Nginx** and configure it to listen on **port 8099**.
* 🔄 Configure **Nginx as a Reverse Proxy** for Apache.
* 📄 Copy the provided **index.html** file from the **Jump Host** to Apache's document root.
* ▶️ Ensure both Apache and Nginx services are running.

After completing the task, users should be able to access the website using:

```bash
curl http://stbkp01:8099
```

---

# 🖥️ Server Details

📍 **Backup Server:** `stbkp01`

📍 **Source File (Jump Host):**

```text
/home/thor/index.html
```

📍 **Apache Document Root:**

```text
/var/www/html/
```

---

# 🛠️ Steps to Perform

## 🔹 Step 1: Copy the Website

From the **Jump Host**:

```bash
scp /home/thor/index.html clint@stbkp01:/tmp/
```

---

## 🔹 Step 2: Login to Backup Server

```bash
ssh clint@stbkp01
sudo -i
```

---

## 🔹 Step 3: Install Apache & Nginx

```bash
yum install -y httpd nginx
```

---

## 🔹 Step 4: Configure Apache Port

Edit:

```bash
vi /etc/httpd/conf/httpd.conf
```

Find:

```apache
Listen 80
```

Change to:

```apache
Listen 8085
```

Do **NOT** change it to:

```apache
Listen 127.0.0.1:8085
```

The task specifically requires Apache to listen on **all interfaces**.

---

## 🔹 Step 5: Copy Website

```bash
cp /tmp/index.html /var/www/html/index.html
```

---

## 🔹 Step 6: Configure Nginx

Edit:

```bash
vi /etc/nginx/nginx.conf
```

Locate the **server** block.

Change:

```nginx
listen 80;
```

to:

```nginx
listen 8099;
```

Replace the location block with:

```nginx
location / {
    proxy_pass http://localhost:8085;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

---

## 🔹 Step 7: Validate Configurations

Apache

```bash
httpd -t
```

Expected:

```text
Syntax OK
```

---

Nginx

```bash
nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

---

## 🔹 Step 8: Start Services

```bash
systemctl enable httpd nginx

systemctl restart httpd

systemctl restart nginx
```

---

# ✅ Verification

## Verify Apache

```bash
systemctl status httpd
```

Expected:

```text
active (running)
```

---

## Verify Nginx

```bash
systemctl status nginx
```

Expected:

```text
active (running)
```

---

## Verify Apache Port

```bash
grep "^Listen" /etc/httpd/conf/httpd.conf
```

Expected:

```text
Listen 8085
```

---

## Verify Nginx Port

```bash
grep "listen" /etc/nginx/nginx.conf
```

Expected:

```text
listen 8099;
```

---

## Final Verification

From the **Jump Host**:

```bash
curl http://stbkp01:8099
```

Expected:

```html
<html>
...
```

or the contents of `index.html`.

---

# 🌍 Real-World Use Cases

* 🌐 Reverse proxy for web applications.
* 🔒 Hide backend servers from clients.
* ⚖️ Load balancing across multiple backend servers.
* 🔐 SSL/TLS termination using Nginx.
* 🚀 Performance improvements through caching and compression.

---

# 🧠 Key Concepts

* 🌐 Apache HTTP Server
* ⚡ Nginx
* 🔄 Reverse Proxy
* 📄 Document Root
* 🔌 Listening Ports
* 📡 HTTP Request Forwarding

---

# 🎤 Interview Questions

### ❓ Q1. What is a Reverse Proxy?

A reverse proxy receives client requests and forwards them to one or more backend servers. The client interacts only with the reverse proxy, not the backend application directly.

---

### ❓ Q2. Why use Nginx as a reverse proxy in front of Apache?

* Better performance for static content.
* SSL/TLS termination.
* Load balancing.
* Improved security by hiding backend servers.
* Efficient handling of concurrent connections.

---

### ❓ Q3. Difference between Forward Proxy and Reverse Proxy?

| Forward Proxy                                | Reverse Proxy                                       |
| -------------------------------------------- | --------------------------------------------------- |
| Represents the client                        | Represents the server                               |
| Used by clients to access external resources | Used by servers to receive client requests          |
| Often used for internet filtering            | Used for load balancing, SSL, caching, and security |

---

### ❓ Q4. What does `proxy_pass` do?

It forwards incoming client requests from Nginx to the specified backend server.

Example:

```nginx
proxy_pass http://localhost:8085;
```

---

### ❓ Q5. Why keep Apache on port 8085 instead of 80?

Apache serves the application internally, while Nginx listens on the public-facing port (`8099`) and forwards requests. This separation improves security and flexibility.

---

# 📚 Commands Learned

```bash
yum install httpd nginx

httpd -t

nginx -t

systemctl restart httpd

systemctl restart nginx

systemctl enable httpd nginx

scp

curl

proxy_pass
```

---

# ⚠️ Common Mistakes

* ❌ Configuring Apache to listen on `127.0.0.1:8085` instead of all interfaces.
* ❌ Forgetting to copy `index.html` to `/var/www/html/`.
* ❌ Incorrect `proxy_pass` URL.
* ❌ Forgetting to restart Nginx after editing the configuration.
* ❌ Skipping configuration validation with `httpd -t` and `nginx -t`.

---

# 💡 Production Tips

* ✅ Always validate Apache and Nginx configurations before restarting services.
* ✅ Use Nginx as the public entry point for better performance and security.
* ✅ Keep backend services on non-public ports whenever possible.
* ✅ Preserve client information using `X-Forwarded-*` headers.

---

# 🚨 Troubleshooting

### ❌ 502 Bad Gateway

Usually indicates that Nginx cannot reach Apache.

Check:

```bash
systemctl status httpd
```

and ensure Apache is listening on port `8085`.

---

### ❌ Connection Refused

Verify Apache is running:

```bash
curl http://localhost:8085
```

---

### ❌ Nginx Fails to Start

Validate the configuration:

```bash
nginx -t
```

Review logs if necessary:

```bash
journalctl -u nginx
```

---

### ❌ Apache Fails to Start

Check the configuration:

```bash
httpd -t
```

Review the logs:

```bash
journalctl -u httpd
```

---

# 🎯 Interview Tip

A favorite DevOps interview question is:

> **"Why do organizations deploy Nginx in front of Apache?"**

A good answer includes:

* Improved performance.
* SSL termination.
* Load balancing.
* Reverse proxy functionality.
* Caching and compression.
* Enhanced security by hiding backend servers.

---

# ⭐ Key Takeaways

* ✔️ Installed Apache and Nginx.
* ✔️ Configured Apache to listen on port `8085`.
* ✔️ Configured Nginx to listen on port `8099`.
* ✔️ Set up Nginx as a reverse proxy for Apache.
* ✔️ Deployed the website to Apache's document root.
* ✔️ Verified the application using `curl`.

---

# 🏆 Difficulty Level

🟡 **Medium**

---

## 💡 KodeKloud Exam Tip

This is one of the **most frequently repeated Linux Level-2 tasks**.

Remember the three key configuration changes:

1. **Apache** → `Listen 8085`
2. **Nginx** → `listen 8099`
3. **Reverse Proxy**:

```nginx
location / {
    proxy_pass http://localhost:8085;
}
```

If `curl http://stbkp01:8099` returns the contents of `index.html`, the reverse proxy is configured correctly.

---

# � Linux Level-2

# 🚀 Task-06 - Configure Nginx as a Reverse Proxy for Apache

---

# 🎯 Objective

Configure a **reverse proxy architecture** on the **Nautilus Backup Server**.

The requirements are:

* 🌐 Install **Apache HTTP Server** and configure it to listen on **port 8085**.
* ⚡ Install **Nginx** and configure it to listen on **port 8099**.
* 🔄 Configure **Nginx as a Reverse Proxy** for Apache.
* 📄 Copy the provided **index.html** file from the **Jump Host** to Apache's document root.
* ▶️ Ensure both Apache and Nginx services are running.

After completing the task, users should be able to access the website using:

```bash
curl http://stbkp01:8099
```

---

# 🖥️ Server Details

📍 **Backup Server:** `stbkp01`

📍 **Source File (Jump Host):**

```text
/home/thor/index.html
```

📍 **Apache Document Root:**

```text
/var/www/html/
```

---

# 🛠️ Steps to Perform

## 🔹 Step 1: Copy the Website

From the **Jump Host**:

```bash
scp /home/thor/index.html clint@stbkp01:/tmp/
```

---

## 🔹 Step 2: Login to Backup Server

```bash
ssh clint@stbkp01
sudo -i
```

---

## 🔹 Step 3: Install Apache & Nginx

```bash
yum install -y httpd nginx
```

---

## 🔹 Step 4: Configure Apache Port

Edit:

```bash
vi /etc/httpd/conf/httpd.conf
```

Find:

```apache
Listen 80
```

Change to:

```apache
Listen 8085
```

Do **NOT** change it to:

```apache
Listen 127.0.0.1:8085
```

The task specifically requires Apache to listen on **all interfaces**.

---

## 🔹 Step 5: Copy Website

```bash
cp /tmp/index.html /var/www/html/index.html
```

---

## 🔹 Step 6: Configure Nginx

Edit:

```bash
vi /etc/nginx/nginx.conf
```

Locate the **server** block.

Change:

```nginx
listen 80;
```

to:

```nginx
listen 8099;
```

Replace the location block with:

```nginx
location / {
    proxy_pass http://localhost:8085;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

---

## 🔹 Step 7: Validate Configurations

Apache

```bash
httpd -t
```

Expected:

```text
Syntax OK
```

---

Nginx

```bash
nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

---

## 🔹 Step 8: Start Services

```bash
systemctl enable httpd nginx

systemctl restart httpd

systemctl restart nginx
```

---

# ✅ Verification

## Verify Apache

```bash
systemctl status httpd
```

Expected:

```text
active (running)
```

---

## Verify Nginx

```bash
systemctl status nginx
```

Expected:

```text
active (running)
```

---

## Verify Apache Port

```bash
grep "^Listen" /etc/httpd/conf/httpd.conf
```

Expected:

```text
Listen 8085
```

---

## Verify Nginx Port

```bash
grep "listen" /etc/nginx/nginx.conf
```

Expected:

```text
listen 8099;
```

---

## Final Verification

From the **Jump Host**:

```bash
curl http://stbkp01:8099
```

Expected:

```html
<html>
...
```

or the contents of `index.html`.

---

# 🌍 Real-World Use Cases

* 🌐 Reverse proxy for web applications.
* 🔒 Hide backend servers from clients.
* ⚖️ Load balancing across multiple backend servers.
* 🔐 SSL/TLS termination using Nginx.
* 🚀 Performance improvements through caching and compression.

---

# 🧠 Key Concepts

* 🌐 Apache HTTP Server
* ⚡ Nginx
* 🔄 Reverse Proxy
* 📄 Document Root
* 🔌 Listening Ports
* 📡 HTTP Request Forwarding

---

# 🎤 Interview Questions

### ❓ Q1. What is a Reverse Proxy?

A reverse proxy receives client requests and forwards them to one or more backend servers. The client interacts only with the reverse proxy, not the backend application directly.

---

### ❓ Q2. Why use Nginx as a reverse proxy in front of Apache?

* Better performance for static content.
* SSL/TLS termination.
* Load balancing.
* Improved security by hiding backend servers.
* Efficient handling of concurrent connections.

---

### ❓ Q3. Difference between Forward Proxy and Reverse Proxy?

| Forward Proxy                                | Reverse Proxy                                       |
| -------------------------------------------- | --------------------------------------------------- |
| Represents the client                        | Represents the server                               |
| Used by clients to access external resources | Used by servers to receive client requests          |
| Often used for internet filtering            | Used for load balancing, SSL, caching, and security |

---

### ❓ Q4. What does `proxy_pass` do?

It forwards incoming client requests from Nginx to the specified backend server.

Example:

```nginx
proxy_pass http://localhost:8085;
```

---

### ❓ Q5. Why keep Apache on port 8085 instead of 80?

Apache serves the application internally, while Nginx listens on the public-facing port (`8099`) and forwards requests. This separation improves security and flexibility.

---

# 📚 Commands Learned

```bash
yum install httpd nginx

httpd -t

nginx -t

systemctl restart httpd

systemctl restart nginx

systemctl enable httpd nginx

scp

curl

proxy_pass
```

---

# ⚠️ Common Mistakes

* ❌ Configuring Apache to listen on `127.0.0.1:8085` instead of all interfaces.
* ❌ Forgetting to copy `index.html` to `/var/www/html/`.
* ❌ Incorrect `proxy_pass` URL.
* ❌ Forgetting to restart Nginx after editing the configuration.
* ❌ Skipping configuration validation with `httpd -t` and `nginx -t`.

---

# 💡 Production Tips

* ✅ Always validate Apache and Nginx configurations before restarting services.
* ✅ Use Nginx as the public entry point for better performance and security.
* ✅ Keep backend services on non-public ports whenever possible.
* ✅ Preserve client information using `X-Forwarded-*` headers.

---

# 🚨 Troubleshooting

### ❌ 502 Bad Gateway

Usually indicates that Nginx cannot reach Apache.

Check:

```bash
systemctl status httpd
```

and ensure Apache is listening on port `8085`.

---

### ❌ Connection Refused

Verify Apache is running:

```bash
curl http://localhost:8085
```

---

### ❌ Nginx Fails to Start

Validate the configuration:

```bash
nginx -t
```

Review logs if necessary:

```bash
journalctl -u nginx
```

---

### ❌ Apache Fails to Start

Check the configuration:

```bash
httpd -t
```

Review the logs:

```bash
journalctl -u httpd
```

---

# 🎯 Interview Tip

A favorite DevOps interview question is:

> **"Why do organizations deploy Nginx in front of Apache?"**

A good answer includes:

* Improved performance.
* SSL termination.
* Load balancing.
* Reverse proxy functionality.
* Caching and compression.
* Enhanced security by hiding backend servers.

---

# ⭐ Key Takeaways

* ✔️ Installed Apache and Nginx.
* ✔️ Configured Apache to listen on port `8085`.
* ✔️ Configured Nginx to listen on port `8099`.
* ✔️ Set up Nginx as a reverse proxy for Apache.
* ✔️ Deployed the website to Apache's document root.
* ✔️ Verified the application using `curl`.

---

# 🏆 Difficulty Level

🟡 **Medium**

---

## 💡 KodeKloud Exam Tip

This is one of the **most frequently repeated Linux Level-2 tasks**.

Remember the three key configuration changes:

1. **Apache** → `Listen 8085`
2. **Nginx** → `listen 8099`
3. **Reverse Proxy**:

```nginx
location / {
    proxy_pass http://localhost:8085;
}
```

If `curl http://stbkp01:8099` returns the contents of `index.html`, the reverse proxy is configured correctly.

---

Good catch! This is the default Nginx configuration on **RHEL/CentOS**, and in this case **you need to add the `location /` block yourself**.

Insert it **inside the `server { ... }` block**, preferably after the `include /etc/nginx/default.d/*.conf;` line.

Your `server` block should look like this:

```nginx
server {
    listen       8099;
    listen       [::]:8099;
    server_name  _;
    root         /usr/share/nginx/html;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
        proxy_pass http://localhost:8085;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    error_page 404 /404.html;
    location = /404.html {
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
    }
}
```

Then run:

```bash
nginx -t
```

If the output is:

```text
syntax is ok
test is successful
```

restart the services:

```bash
systemctl restart httpd
systemctl restart nginx
```

---

### ✅ Before testing, verify Apache

```bash
curl http://localhost:8085
```

You should see the contents of `index.html`.

Then verify Nginx:

```bash
curl http://localhost:8099
```

Finally, from the **Jump Host**:

```bash
curl http://stbkp01:8099
```

---

### 💡 KodeKloud Tip

On RHEL/CentOS, the default `nginx.conf` often **doesn't include a `location /` block** because it serves static files from the `root` directive. For reverse proxy tasks, it's perfectly normal to **add your own `location /` block** inside the `server` block.

---

### Is nginx only provides revers pxorxy and we do vice versa nginx to apache forwarding?

Great question! This is a very common interview topic.

The answer is **No**. **Nginx is not the only reverse proxy**, and **you can configure Apache as a reverse proxy to Nginx as well**.

---

# 🏗️ Scenario 1: Nginx → Apache (Most Common) ✅

```text
Client
   │
   ▼
Nginx (Port 80/443)
   │
   ▼
Apache (Port 8085)
```

This is what you're doing in this task.

Example:

```nginx
location / {
    proxy_pass http://localhost:8085;
}
```

### Why is this common?

* 🚀 Nginx is event-driven and handles thousands of concurrent connections efficiently.
* 🌐 Excellent for serving static content.
* 🔒 Often used for SSL termination.
* ⚖️ Supports load balancing.
* 📦 Forwards dynamic requests to Apache, which handles PHP or other backend processing.

---

# 🏗️ Scenario 2: Apache → Nginx ✅

This is also possible.

```text
Client
   │
   ▼
Apache (80/443)
   │
   ▼
Nginx (8080)
```

Apache can act as the reverse proxy using modules such as:

* `mod_proxy`
* `mod_proxy_http`

Example Apache configuration:

```apache
ProxyPass / http://localhost:8080/
ProxyPassReverse / http://localhost:8080/
```

In this setup, Apache forwards requests to an Nginx backend.

---

# 🏗️ Scenario 3: Nginx → Tomcat (Very Common)

```text
Client
   │
   ▼
Nginx
   │
   ▼
Tomcat
```

This is one of the most common production architectures for Java applications.

---

# 🏗️ Scenario 4: Nginx → Node.js

```text
Client
   │
   ▼
Nginx
   │
   ▼
Node.js
```

Very common for Express.js applications.

---

# 🏗️ Scenario 5: Nginx → Gunicorn (Python)

```text
Client
   │
   ▼
Nginx
   │
   ▼
Gunicorn
   │
   ▼
Flask / Django
```

A standard deployment pattern for Python web applications.

---

# 🏗️ Scenario 6: Apache → Tomcat

Apache can also proxy requests to Tomcat using:

* `mod_proxy`
* `mod_jk`
* `mod_proxy_ajp`

---

# Can Any Web Server Be a Reverse Proxy?

Yes. Many web servers and load balancers can act as reverse proxies, including:

| Software | Reverse Proxy | Load Balancer |
| -------- | ------------- | ------------- |
| Nginx    | ✅             | ✅             |
| Apache   | ✅             | ✅             |
| HAProxy  | ✅             | ✅             |
| Traefik  | ✅             | ✅             |
| Envoy    | ✅             | ✅             |
| Caddy    | ✅             | Limited       |
| IIS      | ✅             | Limited       |

---

# Why Do Companies Prefer Nginx?

Nginx is often chosen because it:

* 🚀 Handles very high concurrency with low memory usage.
* 📂 Serves static files efficiently.
* 🔒 Simplifies SSL/TLS management.
* ⚖️ Provides built-in load balancing.
* 🌐 Offers caching, compression, and request routing.

Apache is still excellent, especially when applications rely on features like `.htaccess` or modules such as `mod_php`, but Nginx is generally favored as the public-facing reverse proxy.

---

# 🎤 Interview Question

### ❓ Can Apache be a reverse proxy?

**Answer:**

Yes. Apache supports reverse proxy functionality through modules like `mod_proxy` and `mod_proxy_http`. It can forward requests to backend servers such as Nginx, Tomcat, Node.js, or another Apache instance.

Example:

```apache
ProxyPass / http://localhost:8080/
ProxyPassReverse / http://localhost:8080/
```

---

# 💡 Real-Time Architecture

In modern DevOps environments, a common architecture is:

```text
                 Internet
                     │
                     ▼
              Load Balancer (AWS ALB)
                     │
                     ▼
          Nginx (Reverse Proxy)
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   Tomcat-1     Tomcat-2     Tomcat-3
```

Here:

* **AWS ALB** distributes traffic to one or more Nginx servers.
* **Nginx** acts as the reverse proxy, handling SSL, caching, and routing.
* **Tomcat** instances run the Java application.

This combination is widely used because it scales well and separates responsibilities across the infrastructure.


---


