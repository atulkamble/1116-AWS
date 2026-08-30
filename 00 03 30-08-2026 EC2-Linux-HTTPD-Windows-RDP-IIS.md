Below is a cleaned-up notebook-style document you can use for teaching/practice.

# AWS EC2 — Linux & Windows Web Server Practical

## 1. EC2 Amazon Linux — Launch

```text
EC2 >> Launch Instance

Name        : server
OS          : Amazon Linux
Instance    : t3.micro
Key Pair    : key.pem
VPC         : Default VPC
Public IP   : Enabled
Storage     : Default SSD

Security Group:
SSH         : 22
HTTP        : 80
HTTPS       : 443
```

## 2. Connect to Amazon Linux EC2

Navigate to the directory where `key.pem` is downloaded.

```text
Downloads >> Right Click >> Open in Terminal / PowerShell
```

```bash
cd Downloads

chmod 400 key.pem

ssh -i "key.pem" ec2-user@ec2-34-229-150-10.compute-1.amazonaws.com
```

> `chmod 400 key.pem` is recommended on Linux/macOS so the private key isn't accessible to other users.

---

# 3. Linux File Permissions

```text
r = Read    = 4
w = Write   = 2
x = Execute = 1
```

```text
r + w     = 4 + 2     = 6
r + w + x = 4 + 2 + 1 = 7
```

Permission structure:

```text
User | Group | Others
rwx  | rwx   | rwx
```

Example:

```text
rwxrwxrwx
777
```

Common permissions:

```text
400 = r--------    Private key
644 = rw-r--r--    Normal files
755 = rwxr-xr-x    Directories / executable files
777 = rwxrwxrwx    Full permissions
```

Commands:

```bash
chmod 755 filename

cat a.txt

nano a.txt
```

---

# 4. Install Apache Web Server

Make sure the EC2 Security Group allows:

```text
HTTP  : TCP 80
HTTPS : TCP 443
```

Install Apache:

```bash
sudo yum install httpd -y
```

Start Apache:

```bash
sudo systemctl start httpd
```

Enable Apache at boot:

```bash
sudo systemctl enable httpd
```

Check status:

```bash
sudo systemctl status httpd
```

---

# 5. Create Website

Apache default document root:

```bash
/var/www/html
```

Commands:

```bash
sudo chmod 755 /var/www/html

cd /var/www/html

pwd

sudo touch index.html

sudo nano index.html
```

Add:

```html
<h1>webserver</h1>
```

Save in Nano:

```text
Ctrl + O
Enter
Ctrl + X
```

Access using the EC2 public IPv4 address:

```text
http://<EC2-PUBLIC-IP>/
```

Example from the lab:

```text
http://98.81.181.71/
```

Expected output:

```text
webserver
```

---

# EC2 Windows Server — Launch & Connect

## 6. Launch Windows EC2

```text
EC2 >> Launch Instance

Name        : winserver
OS          : Microsoft Windows Server 2025
Instance    : t3.medium / c7i-flex.large*
Key Pair    : win.pem
VPC         : Default VPC
Public IP   : Enabled
Storage     : Default SSD

Security Group:
RDP         : 3389
HTTP        : 80
HTTPS       : 443
```

> Use an instance type that is currently available and supported in your selected AWS Region.

---

# 7. Connect to Windows Server

On your local Windows system:

```text
Microsoft Store
      ↓
Windows App
      ↓
Install / Open
```

From AWS:

```text
EC2
 ↓
Instances
 ↓
Select winserver
 ↓
Connect
 ↓
RDP Client
 ↓
Get Password
 ↓
Upload win.pem
 ↓
Decrypt Password
```

Note down:

```text
Username : Administrator
Password : <Decrypted AWS Administrator Password>
```

Then:

```text
Download Remote Desktop File
        ↓
Open RDP File
        ↓
Enter Username + Password
        ↓
Connect
```

**Do not put the decrypted Administrator password in notes, GitHub, screenshots, or shared documents.** If the password shown in your original notes is real, rotate/replace the instance credentials.

---

# 8. Install IIS Web Server

After connecting through RDP:

```text
Server Manager
     ↓
Add Roles and Features
     ↓
Next
     ↓
Role-based or feature-based installation
     ↓
Next
     ↓
Select Server
     ↓
Next
     ↓
Web Server (IIS)
     ↓
Add Features
     ↓
Next
     ↓
Next
     ↓
Install
```

Wait until IIS installation completes.

---

# 9. IIS Website Directory

Default IIS web root:

```text
C:\inetpub\wwwroot
```

Navigate to:

```text
C:
 ↓
inetpub
 ↓
wwwroot
```

Create:

```text
index.html
```

When using Notepad:

```text
File
 ↓
Save As
 ↓
File name: index.html
Save as type: All Files
```

Add:

```html
<h1>webserver</h1>
```

Save the file inside:

```text
C:\inetpub\wwwroot\index.html
```

---

# 10. Access Windows IIS Website

Make sure the Windows EC2 Security Group allows:

```text
HTTP : TCP 80
```

Open:

```text
http://<WINDOWS-EC2-PUBLIC-IP>/
```

Example from the lab:

```text
http://52.87.225.245/
```

Expected:

```text
webserver
```

---

# Final Architecture

```text
                    Internet
                       |
          +------------+------------+
          |                         |
        SSH :22                   RDP :3389
          |                         |
          v                         v
+--------------------+    +----------------------+
| Amazon Linux EC2   |    | Windows Server EC2   |
| Name: server       |    | Name: winserver      |
| Apache HTTPD       |    | IIS Web Server       |
+--------------------+    +----------------------+
          |                         |
     /var/www/html           C:\inetpub\wwwroot
          |                         |
      index.html                 index.html
          |                         |
        HTTP:80                   HTTP:80
          |                         |
          +------------+------------+
                       |
                    Browser
```

## Quick Commands — Amazon Linux

```bash
# Connect
cd Downloads
chmod 400 key.pem
ssh -i "key.pem" ec2-user@<EC2-DNS-NAME>

# Apache
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl status httpd

# Website
sudo chmod 755 /var/www/html
cd /var/www/html
sudo touch index.html
sudo nano index.html

# Verify
cat index.html
```

**Linux Web Root:** `/var/www/html`
**Windows IIS Web Root:** `C:\inetpub\wwwroot`
**Linux Remote Access:** SSH `22`
**Windows Remote Access:** RDP `3389`
**Web Traffic:** HTTP `80` / HTTPS `443`
