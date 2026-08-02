# Website Migration to OpenLiteSpeed (Without Control Panel)

A step-by-step guide to migrate a website from a cPanel server to a standalone **OpenLiteSpeed** server without using any hosting control panel.

---

## Prerequisites

* Root or sudo SSH access
* OpenLiteSpeed installed
* MySQL/MariaDB installed
* `acme.sh` configured (if using Let's Encrypt)
* Website backup or cPanel backup available

---

# 1. Create a System User

Create a dedicated Linux user for the website.

```bash
useradd -m -s /bin/bash USERNAME
```

Set the home directory permissions.

```bash
chmod 751 /home/USERNAME
```

---

# 2. Create Website Directory Structure

Create the website root and log directories.

```bash
mkdir -p /home/USERNAME/DOMAIN/public_html /home/USERNAME/DOMAIN/logs
```

Assign ownership.

```bash
chown -R USERNAME:USERNAME /home/USERNAME
```

---

# 3. Restore Website Files

Go to the document root.

```bash
cd /home/USERNAME/DOMAIN/public_html
```

Download the cPanel backup.

```bash
wget http://SERVER_IP/cpmove-USERNAME.tar.gz
```

Extract the backup.

```bash
tar -xvf cpmove-USERNAME.tar.gz
```

Replace the empty document root with the extracted website files.

```bash
rm -rf public_html && mv cpmove-USERNAME/homedir/public_html .
```

Fix ownership.

```bash
chown -R USERNAME:USERNAME /home/USERNAME
```

---

# 4. Configure OpenLiteSpeed

Main configuration file:

```text
/usr/local/lsws/conf/httpd_config.conf
```

Create the Virtual Host directory.

```bash
mkdir -p /usr/local/lsws/conf/vhosts/DOMAIN
```

Create the Virtual Host configuration.

```bash
vim /usr/local/lsws/conf/vhosts/DOMAIN/vhost.conf
```

Use an existing Virtual Host configuration as a template.

### Recommended Permissions

```text
Directory:
/usr/local/lsws/conf/vhosts/DOMAIN
Permissions : 755
Owner       : lsadm:nobody

vhost.conf
Permissions : 750
Owner       : lsadm:nobody
```

After creating the Virtual Host:

* Add the Virtual Host definition.
* Map it to both **HTTP (80)** and **HTTPS (443)** listeners.
* Restart OpenLiteSpeed.

---

# 5. Create MySQL Database

Create the database.

```sql
CREATE DATABASE username_database;
```

Create the database user.

```sql
CREATE USER 'username_user'@'localhost' IDENTIFIED BY 'StrongPassword';
```

Grant privileges.

```sql
GRANT ALL PRIVILEGES ON username_database.* TO 'username_user'@'localhost';
```

Reload MySQL privileges.

```sql
FLUSH PRIVILEGES;
```

Import the database.

```bash
mysql username_database < /path/to/database.sql
```

---

# 6. Generate SSL Certificate (acme.sh)

Go to the acme.sh directory.

```bash
cd ~/.acme.sh
```

### Root Domain + Wildcard

```bash
./acme.sh --issue -d domain.com -d *.domain.com --dns dns_cf --reloadcmd "systemctl restart lsws"
```

### Single Domain / Subdomain

```bash
./acme.sh --issue -d sub.domain.com --dns dns_cf --reloadcmd "systemctl restart lsws"
```

Example:

```bash
./acme.sh --issue -d laravel.example.com --dns dns_cf --reloadcmd "systemctl restart lsws"
```

> **Tip:** A wildcard certificate (`*.domain.com`) can be reused for all subdomains by referencing the same certificate path in the Virtual Host configuration.

---

# 7. Final Verification

Verify the following after migration:

* Website loads successfully.
* PHP is working.
* Database connection is successful.
* File permissions are correct.
* Virtual Host configuration is loaded.
* SSL certificate is active.
* HTTP redirects to HTTPS (if required).
* Error logs are clean.

---


