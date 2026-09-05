# Linux EC2 Access Using PuTTY + Apache Web Server

## 1. Linux Distribution Basics

| Linux Distribution | Family        | Package Manager | Notes                                      |
| ------------------ | ------------- | --------------- | ------------------------------------------ |
| RHEL               | Red Hat       | `dnf` / `yum`   | Commercial; subscription-based support     |
| CentOS Stream      | Red Hat       | `dnf`           | Free, community-driven upstream of RHEL    |
| Fedora             | Red Hat       | `dnf`           | Community distribution; newer technologies |
| Debian             | Debian        | `apt`           | Community Linux distribution               |
| Ubuntu             | Debian        | `apt`           | Based on Debian; maintained by Canonical   |
| Ubuntu Server      | Debian/Ubuntu | `apt`           | Server-focused Ubuntu edition              |

### Linux Family Relationship

```text
Red Hat Family
│
├── Fedora
├── CentOS Stream
└── RHEL
     └── dnf / yum


Debian Family
│
├── Debian
└── Ubuntu
     └── Ubuntu Server
          └── apt
```

---

# 2. PuTTY

**PuTTY** is an SSH client commonly used on Windows to connect to remote Linux servers.

Default SSH port:

```text
22
```

Official download options:

* PuTTY official site: `putty.org`
* PuTTY download page: `chiark.greenend.org.uk/~sgtatham/putty/latest.html`

---

# 3. Example EC2 Linux Server

You can connect using either the EC2 **Public DNS** or **Public IPv4 address**.

Example:

```text
Public DNS:
ec2-34-224-99-61.compute-1.amazonaws.com

OR

Public IPv4:
34.224.99.61
```

> Note: Public IP/DNS can change after an EC2 stop/start unless an Elastic IP is assigned.

---

# 4. Architecture

```text
Windows PC / Windows EC2
        |
        | RDP (if using Windows EC2)
        v
+----------------------+
| Windows Machine      |
| PuTTY Installed      |
+----------+-----------+
           |
           | SSH - Port 22
           | .ppk Private Key
           v
+----------------------+
| AWS EC2 Linux        |
| Amazon Linux/Ubuntu  |
+----------+-----------+
           |
           | HTTP - Port 80
           v
      Web Browser
```

---

# 5. Prerequisites

Before connecting, verify:

```text
✓ Linux EC2 instance is running
✓ Public IPv4/Public DNS is available
✓ Security Group allows SSH TCP 22
✓ Correct username is known
✓ Correct private key (.ppk) is available
```

Common EC2 usernames:

```text
Amazon Linux  → ec2-user
Ubuntu        → ubuntu
RHEL          → ec2-user
CentOS        → centos (AMI dependent)
```

---

# 6. Connect to Linux EC2 Using PuTTY

## Step 1 — Open Windows Machine

Use either:

```text
Local Windows Computer
```

or connect to a Windows EC2 instance using:

```text
RDP → Port 3389
```

---

## Step 2 — Install PuTTY

Download and install PuTTY on Windows.

Launch:

```text
PuTTY
```

---

## Step 3 — Prepare the Linux EC2 Key

Launch the Linux EC2 instance.

Keep the `.ppk` private key securely on the Windows machine where PuTTY will run.

> Never upload a private key to a public repository or share it with students/users unnecessarily.

---

## Step 4 — Configure PuTTY

Open:

```text
PuTTY
```

Go to:

```text
Session
```

Enter:

```text
Host Name:
34.224.99.61

Port:
22

Connection Type:
SSH
```

---

## Step 5 — Configure Username

Navigate to the SSH login/username settings under:

```text
Connection
   └── Data
```

Set the username.

For Amazon Linux:

```text
ec2-user
```

For Ubuntu:

```text
ubuntu
```

---

## Step 6 — Add Private Key

Depending on the PuTTY version, navigate to the SSH authentication credentials section, for example:

```text
Connection
   └── SSH
       └── Auth
           └── Credentials
```

Select:

```text
Private key file for authentication
```

Browse and select:

```text
your-key.ppk
```

---

## Step 7 — Save the Connection

Return to:

```text
Session
```

Under:

```text
Saved Sessions
```

Enter:

```text
myconnection
```

Click:

```text
Save
```

Then:

```text
Open
```

---

# 7. Verify Linux Login

After connecting:

```bash
id
```

Example purpose:

```text
Shows:
- Current user
- UID
- GID
- Groups
```

Check current user:

```bash
whoami
```

Check hostname:

```bash
hostname
```

---

# 8. Root User / sudo Practice

Become root:

```bash
sudo -i
```

or:

```bash
sudo su -
```

Check:

```bash
whoami
```

Expected:

```text
root
```

Exit root:

```bash
exit
```

### Important

Prefer:

```bash
sudo -i
```

over:

```bash
sudo su
```

for a root login shell.

---

# 9. Root Password

If password-based local root access is specifically required for a lab:

```bash
sudo passwd root
```

> EC2 normally uses SSH key authentication. Enabling root/password SSH login is generally unnecessary and should not be used as the normal EC2 access method.

---

# 10. Create Linux User

Create user:

```bash
sudo adduser atul
```

On many distributions, you can also use:

```bash
sudo useradd -m atul
```

The `-m` option creates the user's home directory when required.

Verify:

```bash
id atul
```

Switch user:

```bash
sudo su - atul
```

or:

```bash
su - atul
```

The second command normally requires the target user's password.

Check:

```bash
whoami
pwd
```

Expected:

```text
atul
/home/atul
```

Return:

```bash
exit
```

---

# 11. Check Linux Users

Display user database:

```bash
cat /etc/passwd
```

Filter a specific user:

```bash
grep atul /etc/passwd
```

Check home directories:

```bash
cd /home
ls
```

---

# 12. Apache Web Server — Ubuntu

Ubuntu/Debian uses:

```text
apt
```

Install Apache:

```bash
sudo apt update
sudo apt install apache2 -y
```

Enable Apache at boot:

```bash
sudo systemctl enable apache2
```

Start Apache:

```bash
sudo systemctl start apache2
```

Check status:

```bash
sudo systemctl status apache2
```

---

# 13. Create Web Page

Instead of changing the entire `/var/www/html` directory permissions, use `sudo` when modifying files.

```bash
cd /var/www/html
```

Remove the default page:

```bash
sudo rm -f index.html
```

Create a page:

```bash
echo "<h1>Web Server</h1>" | sudo tee /var/www/html/index.html
```

Check:

```bash
cat /var/www/html/index.html
```

---

# 14. Test the Website

Ensure the EC2 Security Group allows:

```text
HTTP
TCP
Port 80
Source: 0.0.0.0/0
```

For an internet-facing training web server, open:

```text
http://INSTANCE-PUBLIC-IP
```

Example:

```text
http://34.224.99.61
```

Expected output:

```text
Web Server
```

---

# 15. Apache Shell Script

Create the script in your home directory:

```bash
cd ~
nano script.sh
```

Paste:

```bash
#!/bin/bash

apt update
apt install apache2 -y

systemctl enable apache2
systemctl start apache2

echo "<h1>Web Server</h1>" > /var/www/html/index.html
```

Save the file.

Make executable:

```bash
chmod +x script.sh
```

Run with root privileges:

```bash
sudo ./script.sh
```

Verify:

```bash
systemctl status apache2
```

Then access:

```text
http://INSTANCE-PUBLIC-IP
```

---

# 16. Quick Manual Method

For Ubuntu:

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl enable --now apache2
echo "<h1>Web Server</h1>" | sudo tee /var/www/html/index.html
```

Test locally:

```bash
curl http://localhost
```

Then test from a browser:

```text
http://INSTANCE-PUBLIC-IP
```

---

# 17. Amazon Linux / RHEL Equivalent

Ubuntu uses:

```bash
sudo apt install apache2 -y
```

Amazon Linux/RHEL-family systems generally use:

```bash
sudo dnf install httpd -y
```

Start the service:

```bash
sudo systemctl enable --now httpd
```

Create the page:

```bash
echo "<h1>Web Server</h1>" | sudo tee /var/www/html/index.html
```

Therefore:

```text
Ubuntu/Debian
apt → apache2 → apache2 service

Amazon Linux/RHEL
dnf → httpd → httpd service
```

---

# 18. Important Commands

```bash
id
whoami
hostname
pwd

sudo -i
exit

sudo adduser atul
sudo useradd -m atul
id atul
sudo su - atul

cat /etc/passwd
grep atul /etc/passwd

cd /home
ls

sudo apt update
sudo apt install apache2 -y

sudo systemctl enable apache2
sudo systemctl start apache2
sudo systemctl status apache2

curl http://localhost
```

---

# 19. Ports to Remember

| Service | Port |
| ------- | ---: |
| SSH     |   22 |
| HTTP    |   80 |
| HTTPS   |  443 |
| RDP     | 3389 |

---

# 20. Complete Practice Flow

```text
Windows Machine
      ↓
Install PuTTY
      ↓
Launch Linux EC2
      ↓
Allow SSH : 22
      ↓
Get Public IP / DNS
      ↓
Load .ppk Key
      ↓
Username: ec2-user / ubuntu
      ↓
SSH Login
      ↓
id / whoami / hostname
      ↓
sudo -i
      ↓
Create User
      ↓
Install Apache
      ↓
Create index.html
      ↓
Allow HTTP : 80
      ↓
Open Public IP
      ↓
Web Server Working ✓
```

## Key Points to Remember

**PuTTY**

* Windows SSH client
* Uses SSH port `22`
* Supports `.ppk` private keys

**Amazon Linux**

* Default user: `ec2-user`
* Package manager: `dnf`
* Web server package/service: `httpd`

**Ubuntu**

* Default user: `ubuntu`
* Package manager: `apt`
* Web server package/service: `apache2`

**EC2 Security Group**

* SSH → `22`
* HTTP → `80`
* HTTPS → `443`

**Security**

* Never publish `.ppk`/`.pem` private keys.
* Do not put passwords in training notes or repositories.
* Prefer key-based SSH authentication.
* Avoid unnecessarily changing `/var/www/html` permissions.
