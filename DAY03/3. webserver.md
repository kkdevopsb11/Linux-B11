# Apache HTTPD Web Server on Amazon Linux

## 1. What is a Web Server?

A **Web Server** is software that receives requests from clients such as web browsers and sends web content back to them using protocols such as **HTTP** or **HTTPS**.

Examples of web servers:

* Apache HTTP Server
* Nginx
* Microsoft IIS
* Tomcat
* Caddy

Example:

```text
Browser
   |
   | HTTP Request
   v
Web Server
   |
   | HTTP Response
   v
Browser
```

When we enter:

```text
http://example.com
```

the browser sends an HTTP request to the web server.

The web server processes the request and returns content such as:

* HTML
* CSS
* JavaScript
* Images
* Files
* API responses

---

# 2. What is Apache HTTPD?

**Apache HTTP Server**, commonly called **Apache** or **HTTPD**, is an open-source web server maintained by the Apache Software Foundation.

On Linux systems, the Apache service is usually called:

```text
httpd
```

HTTPD stands for:

```text
HTTP Daemon
```

A daemon is a background service running continuously on a Linux system.

---

# 3. Apache HTTPD Default Ports

Apache normally works on the following ports:

| Protocol | Port |
| -------- | ---: |
| HTTP     |   80 |
| HTTPS    |  443 |

Example:

```text
HTTP
Client ---- Port 80 ----> Apache Web Server

HTTPS
Client ---- Port 443 ---> Apache Web Server
```

---

# 4. Basic Architecture

```text
User / Browser
      |
      |
      | HTTP Request
      | Port 80
      v
Internet
      |
      v
AWS Security Group
      |
      v
EC2 Instance
      |
      v
Apache HTTPD
      |
      v
/var/www/html
      |
      v
index.html
```

Apache reads website files from the document root and returns them to the browser.

---

# 5. Apache Document Root

The default Apache document root on Amazon Linux is:

```bash
/var/www/html
```

Website files are normally placed inside this directory.

Example:

```bash
/var/www/html/index.html
```

If a browser requests:

```text
http://server-ip/
```

Apache normally looks for:

```text
/var/www/html/index.html
```

---

# 6. Launch an Amazon Linux EC2 Instance

Create an EC2 instance using:

```text
AMI        : Amazon Linux
Instance   : t2.micro / t3.micro
Protocol   : SSH
SSH Port   : 22
Web Port   : 80
```

The instance must have a Public IPv4 address if the website needs to be accessed directly from the internet.

---

# 7. Configure AWS Security Group

Allow SSH access:

```text
Type       : SSH
Protocol   : TCP
Port       : 22
Source     : My IP
```

Allow HTTP access:

```text
Type       : HTTP
Protocol   : TCP
Port       : 80
Source     : 0.0.0.0/0
```

For IPv6, if required:

```text
::/0
```

HTTPS can be allowed later using:

```text
Port 443
```

---

# 8. Connect to Amazon Linux

Example:

```bash
ssh -i mykey.pem ec2-user@PUBLIC-IP
```

Example:

```bash
ssh -i devops.pem ec2-user@13.233.10.20
```

---

# 9. Check Operating System

Run:

```bash
cat /etc/os-release
```

Example output may show:

```text
Amazon Linux 2023
```

---

# 10. Update Packages

For Amazon Linux 2023:

```bash
sudo dnf update -y
```

Amazon Linux 2 commonly uses:

```bash
sudo yum update -y
```

---

# 11. Install Apache HTTPD

Amazon Linux 2023:

```bash
sudo dnf install httpd -y
```

Amazon Linux 2:

```bash
sudo yum install httpd -y
```

Verify installation:

```bash
httpd -v
```

Example:

```text
Server version: Apache/2.4.x
```

---

# 12. Check HTTPD Service Status

Run:

```bash
sudo systemctl status httpd
```

If Apache is not running, the status may show:

```text
inactive
```

---

# 13. Start Apache

```bash
sudo systemctl start httpd
```

Verify:

```bash
sudo systemctl status httpd
```

Expected status:

```text
active (running)
```

---

# 14. Enable Apache During Boot

Starting Apache manually only starts the service for the current running system.

To automatically start Apache whenever the server boots:

```bash
sudo systemctl enable httpd
```

Verify:

```bash
sudo systemctl is-enabled httpd
```

Expected output:

```text
enabled
```

---

# 15. Start and Enable Together

Instead of using two commands:

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

we can use:

```bash
sudo systemctl enable --now httpd
```

---

# 16. Verify Apache Using Browser

Find the EC2 Public IPv4 address.

Example:

```text
13.233.10.20
```

Open:

```text
http://13.233.10.20
```

If Apache is installed, running, and port 80 is allowed, the Apache default/test page or configured website should appear.

---

# 17. Create a Custom Web Page

Move to the Apache document root:

```bash
cd /var/www/html
```

Create:

```bash
sudo vi index.html
```

Add:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Web Server</title>
</head>

<body>
    <h1>Welcome to Apache Web Server</h1>
    <p>Running on Amazon Linux</p>
</body>
</html>
```

Save the file.

Open:

```text
http://PUBLIC-IP
```

Example:

```text
http://13.233.10.20
```

The custom page should now appear.

---

# 18. Create a Web Page Using echo

A quick method:

```bash
echo "<h1>Welcome to Apache Web Server</h1>" | sudo tee /var/www/html/index.html
```

Verify:

```bash
cat /var/www/html/index.html
```

---

# 19. Add Server Information

Example:

```bash
echo "<h1>Welcome to KK FUNDA DEVOPS</h1>" | sudo tee /var/www/html/index.html
```

Add hostname:

```bash
hostname
```

Create a dynamic-looking static page:

```bash
echo "<h1>Web Server: $(hostname)</h1>" | sudo tee /var/www/html/index.html
```

---

# 20. Important Apache Commands

## Start Apache

```bash
sudo systemctl start httpd
```

## Stop Apache

```bash
sudo systemctl stop httpd
```

## Restart Apache

```bash
sudo systemctl restart httpd
```

## Reload Apache

```bash
sudo systemctl reload httpd
```

## Check Apache Status

```bash
sudo systemctl status httpd
```

## Enable Apache

```bash
sudo systemctl enable httpd
```

## Disable Apache

```bash
sudo systemctl disable httpd
```

## Check Whether Enabled

```bash
sudo systemctl is-enabled httpd
```

## Check Whether Active

```bash
sudo systemctl is-active httpd
```

---

# 21. Restart vs Reload

Restart:

```bash
sudo systemctl restart httpd
```

Apache service is stopped and started again.

Reload:

```bash
sudo systemctl reload httpd
```

Apache reloads its configuration without completely stopping the service.

For many configuration changes, reload is preferred when possible.

---

# 22. Check Port 80

Run:

```bash
sudo ss -tulpn
```

or:

```bash
sudo ss -tulpn | grep :80
```

Example:

```text
LISTEN 0 511 *:80 *:* users:(("httpd",pid=1234))
```

This confirms Apache is listening on port 80.

---

# 23. Test Website from the Server

Run:

```bash
curl http://localhost
```

or:

```bash
curl 127.0.0.1
```

Expected output:

```html
<h1>Welcome to Apache Web Server</h1>
```

This is very useful for troubleshooting.

---

# 24. Request Flow

When the following URL is accessed:

```text
http://13.233.10.20
```

the request flow is:

```text
Browser
   |
   | HTTP Request
   | Destination Port: 80
   v
Internet
   |
   v
AWS Security Group
   |
   | Allows TCP 80
   v
EC2 Instance
   |
   v
Apache HTTPD
   |
   v
/var/www/html/index.html
   |
   v
HTTP Response
   |
   v
Browser
```

---

# 25. Apache Configuration Directory

Important Apache configuration files are normally located under:

```bash
/etc/httpd/
```

Main configuration file:

```bash
/etc/httpd/conf/httpd.conf
```

Additional configuration files:

```bash
/etc/httpd/conf.d/
```

View the main configuration:

```bash
sudo cat /etc/httpd/conf/httpd.conf
```

---

# 26. Important Apache Locations

| Purpose             | Location                     |
| ------------------- | ---------------------------- |
| Main configuration  | `/etc/httpd/conf/httpd.conf` |
| Extra configuration | `/etc/httpd/conf.d/`         |
| Website files       | `/var/www/html/`             |
| Access logs         | `/var/log/httpd/access_log`  |
| Error logs          | `/var/log/httpd/error_log`   |
| Service name        | `httpd`                      |

---

# 27. Apache Access Log

Apache records incoming requests in:

```bash
/var/log/httpd/access_log
```

View:

```bash
sudo cat /var/log/httpd/access_log
```

Monitor live requests:

```bash
sudo tail -f /var/log/httpd/access_log
```

Example:

```text
192.168.1.10 - - [23/Sep/2026] "GET / HTTP/1.1" 200
```

---

# 28. Apache Error Log

Errors are normally stored in:

```bash
/var/log/httpd/error_log
```

View:

```bash
sudo cat /var/log/httpd/error_log
```

Monitor continuously:

```bash
sudo tail -f /var/log/httpd/error_log
```

This is one of the most important files for troubleshooting Apache issues.

---

# 29. Check Apache Configuration

Before restarting Apache after configuration changes:

```bash
sudo apachectl configtest
```

Expected:

```text
Syntax OK
```

Another command is:

```bash
sudo httpd -t
```

Expected:

```text
Syntax OK
```

---

# 30. HTTP Status Codes

Common HTTP response codes:

| Code | Meaning               |
| ---: | --------------------- |
|  200 | OK                    |
|  201 | Created               |
|  301 | Permanent Redirect    |
|  302 | Temporary Redirect    |
|  400 | Bad Request           |
|  401 | Unauthorized          |
|  403 | Forbidden             |
|  404 | Not Found             |
|  500 | Internal Server Error |
|  502 | Bad Gateway           |
|  503 | Service Unavailable   |

Example:

```text
HTTP 200
```

means the request was processed successfully.

---

# 31. Check HTTP Response Using curl

Run:

```bash
curl -I http://localhost
```

Example:

```text
HTTP/1.1 200 OK
Server: Apache
Content-Type: text/html
```

Only headers are returned because of:

```bash
-I
```

---

# 32. Download a Website Template

Example:

```bash
cd /tmp
```

Download a website:

```bash
wget WEBSITE-URL
```

If the website is provided as a ZIP file:

```bash
unzip website.zip
```

Copy files:

```bash
sudo cp -r website/* /var/www/html/
```

Restart Apache if required:

```bash
sudo systemctl restart httpd
```

---

# 33. Install wget and unzip

If they are not installed:

```bash
sudo dnf install wget unzip -y
```

For Amazon Linux 2:

```bash
sudo yum install wget unzip -y
```

---

# 34. Check Website Files

```bash
ls -l /var/www/html
```

Example:

```text
index.html
css
js
images
```

---

# 35. File Permissions

Check:

```bash
ls -ld /var/www/html
```

and:

```bash
ls -l /var/www/html
```

The Apache process must have permission to read the website files.

Common permissions:

```bash
sudo chmod -R 755 /var/www/html
```

Avoid using:

```bash
chmod -R 777
```

unless there is a very specific requirement.

`777` gives write permissions to everyone and is generally not recommended.

---

# 36. Find Apache Processes

Run:

```bash
ps -ef | grep httpd
```

Example:

```text
root     1200  httpd
apache   1201  httpd
apache   1202  httpd
apache   1203  httpd
```

Apache normally creates multiple worker processes to handle incoming requests.

---

# 37. Apache Service Details

Run:

```bash
systemctl status httpd
```

Useful information includes:

```text
Loaded
Active
Main PID
Tasks
Memory
Logs
```

---

# 38. Check Package

Amazon Linux 2023:

```bash
dnf list installed | grep httpd
```

Alternative:

```bash
rpm -qa | grep httpd
```

---

# 39. Remove Apache

Amazon Linux 2023:

```bash
sudo dnf remove httpd -y
```

Amazon Linux 2:

```bash
sudo yum remove httpd -y
```

---

# 40. Troubleshooting Scenario

Assume:

```text
http://PUBLIC-IP
```

is not opening.

Follow the checks below.

## Step 1: Check Instance

Verify the EC2 instance is:

```text
Running
```

---

## Step 2: Check Public IP

Verify that the correct Public IPv4 address is being used.

```bash
curl ifconfig.me
```

or check the EC2 console.

---

## Step 3: Check Security Group

Verify:

```text
TCP 80
```

is allowed.

Example:

```text
HTTP
TCP
80
0.0.0.0/0
```

---

## Step 4: Check Apache Package

```bash
rpm -qa | grep httpd
```

---

## Step 5: Check Service

```bash
sudo systemctl status httpd
```

If stopped:

```bash
sudo systemctl start httpd
```

---

## Step 6: Test Locally

```bash
curl localhost
```

If `curl localhost` works but the browser cannot access the website, investigate:

```text
Security Group
Network ACL
Routing
Public IP
Firewall
```

---

## Step 7: Check Port

```bash
sudo ss -tulpn | grep :80
```

---

## Step 8: Check Website Files

```bash
ls -l /var/www/html
```

Verify:

```text
index.html
```

exists.

---

## Step 9: Check Logs

```bash
sudo tail -100 /var/log/httpd/error_log
```

and:

```bash
sudo tail -100 /var/log/httpd/access_log
```

---

# 41. Public IP Limitation

An EC2 instance's automatically assigned Public IPv4 address can change when the instance is stopped and started.

Example:

Before:

```text
13.233.10.20
```

After Stop/Start:

```text
3.110.45.60
```

This is one reason production environments should not normally depend directly on an automatically assigned EC2 public IP.

Options include:

```text
Elastic IP
Load Balancer
Route 53 DNS
```

---

# 42. Elastic IP

An **Elastic IP** is a static public IPv4 address provided by AWS.

Architecture:

```text
Internet
   |
   v
Elastic IP
   |
   v
EC2
   |
   v
Apache
```

It helps avoid changing the public address when the EC2 instance is stopped and started.

AWS charges may apply depending on Elastic IP usage, so unused addresses should not be retained unnecessarily.

---

# 43. Domain-Based Access

Instead of accessing:

```text
http://13.233.10.20
```

a domain can be used:

```text
http://example.com
```

Architecture:

```text
User
 |
 v
example.com
 |
 v
DNS
 |
 v
Public IP / Load Balancer
 |
 v
EC2
 |
 v
Apache
```

---

# 44. DNS Role

DNS converts a domain name such as:

```text
example.com
```

into an IP address such as:

```text
13.233.10.20
```

Example:

```text
example.com
       |
       v
DNS Lookup
       |
       v
13.233.10.20
       |
       v
Apache Web Server
```

---

# 45. HTTP vs HTTPS

HTTP:

```text
http://example.com
```

uses:

```text
Port 80
```

HTTPS:

```text
https://example.com
```

uses:

```text
Port 443
```

HTTPS encrypts communication between the browser and server using TLS.

Production websites should normally use HTTPS.

---

# 46. Complete Installation Script

Amazon Linux 2023:

```bash
#!/bin/bash

dnf update -y

dnf install httpd -y

systemctl start httpd

systemctl enable httpd

echo "<h1>Welcome to Apache Web Server</h1>" > /var/www/html/index.html
```

---

# 47. EC2 User Data Example

The same script can be used while creating an EC2 instance under:

```text
Advanced Details
    |
    v
User Data
```

Example:

```bash
#!/bin/bash

dnf install httpd -y

systemctl enable --now httpd

cat <<EOF > /var/www/html/index.html
<html>
<head>
    <title>Apache Server</title>
</head>

<body>
    <h1>Welcome to Apache HTTPD</h1>
    <h2>Running on Amazon Linux</h2>
</body>
</html>
EOF
```

After the EC2 instance starts:

```text
http://PUBLIC-IP
```

should display the website.

---

# 48. Manual Installation vs User Data

Manual:

```text
Launch EC2
     |
SSH
     |
Install HTTPD
     |
Start HTTPD
     |
Create Website
```

Automated:

```text
Launch EC2
     |
User Data
     |
Install HTTPD
     |
Start HTTPD
     |
Create Website
```

User Data helps automate initial server configuration.

---

# 49. Useful Commands Summary

```bash
# Install Apache
sudo dnf install httpd -y

# Start Apache
sudo systemctl start httpd

# Enable Apache
sudo systemctl enable httpd

# Start + Enable
sudo systemctl enable --now httpd

# Stop Apache
sudo systemctl stop httpd

# Restart Apache
sudo systemctl restart httpd

# Reload Apache
sudo systemctl reload httpd

# Status
sudo systemctl status httpd

# Apache version
httpd -v

# Configuration test
sudo apachectl configtest

# Check port
sudo ss -tulpn | grep :80

# Test locally
curl localhost

# HTTP headers
curl -I localhost

# Website directory
cd /var/www/html

# Main config
sudo vi /etc/httpd/conf/httpd.conf

# Access log
sudo tail -f /var/log/httpd/access_log

# Error log
sudo tail -f /var/log/httpd/error_log
```

---

# 50. Real-World Architecture

A basic environment:

```text
Internet
   |
   v
Security Group
   |
   v
EC2
   |
   v
Apache
   |
   v
Website
```

A more production-oriented architecture:

```text
                   Internet
                       |
                       v
                   Route 53
                       |
                       v
                Load Balancer
                       |
             ---------------------
             |                   |
             v                   v
          EC2-1                EC2-2
          Apache               Apache
             |                   |
             ---------------------
                       |
                       v
                    Backend
                       |
                       v
                    Database
```

This architecture can provide:

* High availability
* Load distribution
* Scalability
* DNS-based access
* HTTPS support

---

# 51. Important Interview Questions

### What is HTTPD?

HTTPD is the Apache HTTP Server daemon that handles HTTP requests.

### What is the default Apache port?

```text
80
```

### What is the HTTPS port?

```text
443
```

### What is the Apache document root?

```bash
/var/www/html
```

### What is the Apache main configuration file?

```bash
/etc/httpd/conf/httpd.conf
```

### How do you start Apache?

```bash
sudo systemctl start httpd
```

### How do you restart Apache?

```bash
sudo systemctl restart httpd
```

### How do you check Apache status?

```bash
sudo systemctl status httpd
```

### How do you check Apache configuration syntax?

```bash
sudo apachectl configtest
```

### How do you check whether port 80 is listening?

```bash
sudo ss -tulpn | grep :80
```

### Where are Apache logs stored?

```bash
/var/log/httpd/
```

Main logs:

```text
access_log
error_log
```

### How do you test Apache locally?

```bash
curl localhost
```

---

# 52. Key Points

* Apache HTTPD is a web server.
* HTTP normally uses port `80`.
* HTTPS normally uses port `443`.
* Website files are stored under `/var/www/html`.
* Apache service name is `httpd`.
* `systemctl` is used to manage the service.
* AWS Security Groups must allow the required ports.
* `access_log` records incoming requests.
* `error_log` helps identify errors.
* `curl localhost` is useful for troubleshooting.
* `apachectl configtest` should be used before applying major configuration changes.
* DNS can map a domain name to a server or load balancer.
* HTTPS should be used for secure production websites.
* User Data can automate HTTPD installation during EC2 creation.

For GitHub, a good filename would be **`apache-httpd-amazon-linux.md`**.
