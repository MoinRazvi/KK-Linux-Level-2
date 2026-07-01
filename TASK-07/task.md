### xFusionCorp Industries has hosted several static websites on Nautilus Application Servers in Stratos DC. There are some confidential directories in the document root that need to be password protected. Since they are using Apache for hosting the websites, the production support team has decided to use .htaccess with basic auth. There is a website that needs to be uploaded to /var/www/html/security on Nautilus App Server 2. However, we need to set up the authentication before that.

1. Create /var/www/html/security directory if doesn't exist.

2. Add a user kirsty in htpasswd and set its password to LQfKeWWxWD.

3. There is a file /tmp/index.html present on Jump Server. Copy the same to the directory you created, please make sure default document root should remain /var/www/html. Also website should work on URL http://<app-server-hostname>:80/security/
