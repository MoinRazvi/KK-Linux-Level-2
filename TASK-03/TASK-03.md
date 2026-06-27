# 🚀 Task-03 - Install & Configure Apache Tomcat


# 🎯 Objective

Install and configure **Apache Tomcat** on **App Server 3**, change the default listening port to **8088**, and deploy the provided **ROOT.war** application so it is accessible directly from the base URL.

After completing the task, the application should be accessible at:

```text
http://stapp03:8088
```

This task introduces Java application deployment using Tomcat, one of the most widely used Java application servers.

---

# 🖥️ Server Details

📍 **Tomcat Server:** App Server 3 (`stapp03`)

📍 **WAR File Location:** Jump Host

```text
/tmp/ROOT.war
```

---

# 🛠️ Steps to Perform

### 🔹 Step 1: Copy the WAR file to App Server 3

On the **Jump Host**:

```bash
scp /tmp/ROOT.war banner@stapp03:/tmp/
```

---

### 🔹 Step 2: Login to App Server 3

```bash
ssh banner@stapp03
sudo -i
```

---

### 🔹 Step 3: Install Tomcat

For CentOS/RHEL:

```bash
yum install -y tomcat tomcat-webapps tomcat-admin-webapps
```

> **Note:** Some KodeKloud environments may only have the `tomcat` package available. If `tomcat-webapps` or `tomcat-admin-webapps` are unavailable, install only `tomcat`.

---

### 🔹 Step 4: Change Tomcat Port

Edit the server configuration:

```bash
vi /etc/tomcat/server.xml
```

Locate:

```xml
<Connector port="8080"
```

Change it to:

```xml
<Connector port="8088"
```

---

### 🔹 Step 5: Deploy the Application

Copy the WAR file to Tomcat's webapps directory:

```bash
cp /tmp/ROOT.war /var/lib/tomcat/webapps/
```

Since the file is named **ROOT.war**, Tomcat automatically deploys it as the default application.

No additional configuration is required.

---

### 🔹 Step 6: Start and Enable Tomcat

```bash
systemctl enable tomcat
systemctl restart tomcat
```

---

### 🔹 Step 7: Verify Tomcat Status

```bash
systemctl status tomcat
```

Expected:

```text
active (running)
```

---

# ✅ Verification

### Verify Tomcat is Listening on Port 8088

```bash
ss -tlnp | grep 8088
```

Expected:

```text
LISTEN 0 100 *:8088
```

---

### Verify Deployment

```bash
ls /var/lib/tomcat/webapps
```

Expected:

```text
ROOT
ROOT.war
```

Tomcat automatically extracts the WAR file into the `ROOT` directory.

---

### Verify the Web Application

From the **Jump Host**:

```bash
curl http://stapp03:8088
```

Expected:

```html
<html>
...
```

or the application's HTML content.

---

# 🌍 Real-World Use Cases

* ☕ Deploying Java web applications in enterprise environments.
* 🌐 Hosting internal business applications.
* 🔄 CI/CD deployments using Jenkins or other automation tools.
* 📦 Deploying Spring Boot WAR applications.
* 🚀 Serving Java-based REST APIs and web services.

---

# 🧠 Key Concepts

* ☕ Apache Tomcat
* 📦 WAR (Web Application Archive)
* 🌐 HTTP Connector
* 📁 ROOT Application
* ⚙️ `server.xml`
* 🚀 Java Application Deployment

---

# 🎤 Interview Questions

### ❓ Q1. What is Apache Tomcat?

Apache Tomcat is an open-source Java application server that implements Java Servlet, JSP, and WebSocket technologies. It is commonly used to host Java web applications.

---

### ❓ Q2. What is a WAR file?

A **WAR (Web Application Archive)** is a packaged Java web application containing all the components needed for deployment, such as:

* Java classes
* JSP files
* HTML/CSS/JavaScript
* Libraries (`WEB-INF/lib`)
* Configuration files (`WEB-INF/web.xml`)

---

### ❓ Q3. Why is the application named `ROOT.war`?

Tomcat deploys `ROOT.war` as the default web application, making it accessible directly from the base URL:

```text
http://server:port/
```

If the file were named `myapp.war`, it would be accessible at:

```text
http://server:port/myapp
```

---

### ❓ Q4. Where does Tomcat deploy applications?

Default deployment directory:

```text
/var/lib/tomcat/webapps/
```

---

### ❓ Q5. Which file controls Tomcat's listening port?

```text
/etc/tomcat/server.xml
```

---

### ❓ Q6. How do you verify a deployed application?

You can:

* Use `curl` to access the application.
* Check the extracted directory under `/var/lib/tomcat/webapps`.
* Review Tomcat logs in `/var/log/tomcat/`.

---

# 📚 Commands Learned

```bash
yum install tomcat

vi /etc/tomcat/server.xml

cp /tmp/ROOT.war /var/lib/tomcat/webapps/

systemctl restart tomcat

systemctl enable tomcat

systemctl status tomcat

ss -tlnp | grep 8088

curl http://stapp03:8088
```

---

# ⚠️ Common Mistakes

* ❌ Forgetting to copy the WAR file from the Jump Host.
* ❌ Not changing the connector port from `8080` to `8088`.
* ❌ Placing the WAR file in the wrong directory.
* ❌ Forgetting to restart Tomcat after modifying `server.xml`.
* ❌ Testing the application before Tomcat has finished extracting the WAR file.

---

# 💡 Production Tips

* ✅ Always validate the Tomcat service status after deployment.
* ✅ Store application artifacts (WAR files) in a version-controlled artifact repository (e.g., Nexus or Artifactory).
* ✅ Monitor Tomcat logs during deployment to detect startup or application errors.
* ✅ Use a reverse proxy (e.g., Apache HTTP Server or Nginx) in front of Tomcat for production deployments.

---

# 🚨 Troubleshooting

### ❌ Tomcat Service Fails to Start

Check the service status:

```bash
systemctl status tomcat
```

Review logs:

```bash
journalctl -u tomcat
```

---

### ❌ Application Not Loading

Verify the WAR file exists:

```bash
ls -l /var/lib/tomcat/webapps/
```

Ensure the `ROOT` directory has been extracted after deployment.

---

### ❌ Port 8088 Not Listening

Verify the connector configuration in `server.xml` and restart Tomcat.

---

### ❌ Connection Refused

Ensure the Tomcat service is running and listening on port `8088`:

```bash
ss -tlnp | grep 8088
```

---

# 🎯 Interview Tip

A common interview question is:

> **"Why would you deploy an application as `ROOT.war` instead of `myapp.war`?"**

Deploying as `ROOT.war` allows users to access the application directly from the server's base URL (e.g., `http://server:8088/`) without specifying an additional context path. This is useful for primary web applications or landing pages.

---

# ⭐ Key Takeaways

* ✔️ Installed Apache Tomcat on App Server 3.
* ✔️ Configured Tomcat to listen on port `8088`.
* ✔️ Deployed the application as `ROOT.war` for direct base URL access.
* ✔️ Verified the service and application functionality using `curl`.
* ✔️ Learned the fundamentals of Java web application deployment.

---

# 🏆 Difficulty Level

🟡 **Medium**
