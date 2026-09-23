# HTTP to HTTPS Redirection in Apache and Nginx

This document explains how to enable HTTP to HTTPS redirection for a website hosted on an EC2 instance, and also how to handle redirection correctly when traffic is coming through a Load Balancer.

---

## 1. Why HTTP to HTTPS Redirection is Needed

When users open a website using `http://`, the connection is not encrypted.  
To secure the communication between the browser and the server, we use `https://`.

Benefits of HTTPS:
- Encrypts data between client and server
- Builds user trust
- Required for many modern browsers and integrations
- Helps improve website security posture

So, the goal is:

- If user opens `http://kkdevopsb7.online`
- Automatically redirect to `https://kkdevopsb7.online`

---

## 2. Pre-Requisites

Before configuring HTTPS, make sure the following are ready:

### Domain setup
Create DNS `A` records for your domain and subdomain pointing to your EC2 public IP address.

Example:
- `kkdevopsb7.online`
- `www.kkdevopsb7.online`

Both should point to the EC2 instance public IP.

### Verify DNS
Use `nslookup` to confirm domain resolution:

```bash
nslookup kkdevopsb7.online
nslookup www.kkdevopsb7.online
````

Expected result:

* Both names should resolve to your EC2 public IP address

If DNS is not resolving correctly, Certbot certificate generation may fail.

---

## 3. Install Required Packages on Apache Server

If you are using Apache on Red Hat / CentOS / Rocky / Alma / Amazon Linux style systems:

```bash
sudo dnf install -y certbot mod_ssl python3-certbot-apache
```

### Package purpose

* `certbot`
  Used to request and manage SSL certificates from Let's Encrypt

* `mod_ssl`
  Enables SSL support in Apache

* `python3-certbot-apache`
  Allows Certbot to automatically configure Apache

---

## 4. Create Apache VirtualHost Configuration for Port 80

Create a configuration file:

```bash
sudo vim /etc/httpd/conf.d/kkdevopsb7.conf
```

Add the following content:

```apache
<VirtualHost *:80>
    ServerName kkdevopsb7.online
    ServerAlias www.kkdevopsb7.online
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/kkdevopsb7_error.log
    CustomLog /var/log/httpd/kkdevopsb7-access.log combined
</VirtualHost>
```

### Explanation

* `VirtualHost *:80`
  Tells Apache to listen for HTTP traffic on port 80

* `ServerName`
  Main domain name

* `ServerAlias`
  Additional domain name such as `www`

* `DocumentRoot`
  Website files location

* `ErrorLog` and `CustomLog`
  Separate log files for troubleshooting

---

## 5. Restart Apache and Validate Configuration

Run:

```bash
sudo apachectl configtest
sudo systemctl restart httpd
```

### Important

Always run `apachectl configtest` before restart.
It checks whether your Apache configuration syntax is valid.

Expected output:

```bash
Syntax OK
```

---

## 6. Generate SSL Certificate Using Certbot

Now request the SSL certificate:

```bash
sudo certbot --apache -d kkdevopsb7.online -d www.kkdevopsb7.online
```

### What this command does

* Requests SSL certificate for both domain and `www`
* Updates Apache configuration automatically
* Can optionally configure HTTP to HTTPS redirection

During execution, Certbot may ask:

* Email address
* Terms agreement
* Whether to redirect HTTP to HTTPS

Choose redirect option if prompted.

After completion, restart Apache:

```bash
sudo systemctl restart httpd
```

---

## 7. How to Verify HTTPS

Test in browser:

* `http://kkdevopsb7.online`
* `http://www.kkdevopsb7.online`

Expected:

* Both should automatically redirect to HTTPS

Also test:

```bash
curl -I http://kkdevopsb7.online
curl -I https://kkdevopsb7.online
```

Expected for HTTP:

* Status code `301` or `302`
* Location header pointing to `https://...`

---

## 8. Apache HTTP to HTTPS Redirection Behind Load Balancer

When Apache is behind a Load Balancer, direct port 80 to 443 redirection should be handled carefully.

Reason:

* Load Balancer may terminate SSL
* Backend Apache may only receive HTTP traffic from LB
* Apache must check forwarded headers to understand the original client protocol

### Apache Redirect Rule for Load Balancer

Open Apache config:

```bash
sudo vim /etc/httpd/conf/httpd.conf
```

Add this inside VirtualHost for port 80:

```apache
<VirtualHost *:80>
    RewriteEngine On
    RewriteCond %{HTTP:X-Forwarded-Proto} =http
    RewriteRule .* https://%{HTTP:Host}%{REQUEST_URI} [L,R=permanent]
</VirtualHost>
```

### Explanation

* `RewriteEngine On`
  Enables rewrite engine

* `RewriteCond %{HTTP:X-Forwarded-Proto} =http`
  Checks whether original request from client was HTTP

* `RewriteRule .* https://%{HTTP:Host}%{REQUEST_URI} [L,R=permanent]`
  Redirects request to HTTPS permanently

### Why `X-Forwarded-Proto` is used

When a request comes through an Application Load Balancer or reverse proxy, the backend server may not directly know whether the client used HTTP or HTTPS.
The load balancer passes this information using the `X-Forwarded-Proto` header.

If this header says `http`, Apache redirects to HTTPS.

### Restart Apache

```bash
sudo systemctl restart httpd
```

---

## 9. Nginx HTTP to HTTPS Redirection Behind Load Balancer

If your backend server uses Nginx instead of Apache, configure as below.

Open Nginx config:

```bash
sudo vim /etc/nginx/nginx.conf
```

Add:

```nginx
server {
    listen 80;
    server_name _;
    if ($http_x_forwarded_proto = 'http') {
        return 301 https://$host$request_uri;
    }
}
```

### Explanation

* `listen 80`
  Nginx listens on HTTP port

* `server_name _;`
  Matches any hostname

* `$http_x_forwarded_proto`
  Reads the protocol passed by load balancer

* `return 301 https://$host$request_uri;`
  Redirects to HTTPS permanently

### Restart Nginx

```bash
sudo systemctl restart nginx
```

---

## 10. Direct Apache Redirection Without Load Balancer

If Apache is directly exposed to the internet and SSL is configured locally on the same server, you can also use a simpler redirection method.

Example:

```apache
<VirtualHost *:80>
    ServerName kkdevopsb7.online
    ServerAlias www.kkdevopsb7.online
    Redirect permanent / https://kkdevopsb7.online/
</VirtualHost>
```

### When to use this

Use this only when:

* Apache itself handles HTTPS on port 443
* There is no load balancer in front
* SSL certificate is installed on Apache server

---

## 11. Important Real-Time Notes

### Case 1: Domain not resolving

If DNS is not pointing correctly to your server, Certbot will fail.

### Case 2: Security Group not allowing traffic

Ensure EC2 security group allows:

* Port 80
* Port 443

### Case 3: Apache service not running

Check status:

```bash
sudo systemctl status httpd
```

### Case 4: Configuration issue

Check syntax:

```bash
sudo apachectl configtest
```

### Case 5: Certificate renewal

Let's Encrypt certificates expire every 90 days.

Check renewal timer:

```bash
systemctl list-timers | grep certbot
```

Manual test:

```bash
sudo certbot renew --dry-run
```

---

## 12. Complete Apache Example with HTTPS Redirection

A practical flow is:

### Step 1: Create port 80 VirtualHost

```apache
<VirtualHost *:80>
    ServerName kkdevopsb7.online
    ServerAlias www.kkdevopsb7.online
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/kkdevopsb7_error.log
    CustomLog /var/log/httpd/kkdevopsb7-access.log combined
</VirtualHost>
```

### Step 2: Validate and restart

```bash
sudo apachectl configtest
sudo systemctl restart httpd
```

### Step 3: Generate certificate

```bash
sudo certbot --apache -d kkdevopsb7.online -d www.kkdevopsb7.online
```

### Step 4: Test access

```bash
curl -I http://kkdevopsb7.online
curl -I https://kkdevopsb7.online
```

---

## 13. Load Balancer Scenario Summary

### If SSL is terminated at Load Balancer

* Client connects to LB using HTTPS
* LB forwards request to backend
* Backend checks `X-Forwarded-Proto`
* Redirect logic should use forwarded header

### If SSL is terminated on backend server

* Load Balancer may pass traffic directly
* Backend Apache/Nginx handles certificates and redirects

---

## 14. Best Practice Recommendation

For production environments:

* Use Load Balancer for SSL termination when possible
* Use valid DNS records before certificate generation
* Keep port 80 open only for redirection
* Force all user traffic to HTTPS
* Test renewal process regularly
* Validate configs before every restart

---

## 15. Final Conclusion

HTTP to HTTPS redirection is an important configuration for secure website access.

There are two common scenarios:

### Normal EC2 Apache setup

* Point domain to EC2
* Install Certbot and mod_ssl
* Create VirtualHost
* Generate certificate
* Enable redirect

### Load Balancer setup

* Use `X-Forwarded-Proto`
* Redirect only when original request is HTTP
* Configure Apache or Nginx accordingly

This ensures:

* Secure access
* Better user trust
* Proper production-ready website behavior

---
