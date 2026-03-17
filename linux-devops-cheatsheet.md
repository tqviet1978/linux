# 🐧 Linux DevOps Cheatsheet

> A comprehensive reference of Linux commands commonly used in DevOps workflows.

---

## 📋 Table of Contents

| # | Topic |
|---|-------|
| 1 | [📦 Package Management](#-package-management) |
| 2 | [🗄️ MySQL / MariaDB](#️-mysql--mariadb) |
| 3 | [🐘 PostgreSQL](#-postgresql) |
| 4 | [🔴 Redis](#-redis) |
| 5 | [🍃 MongoDB](#-mongodb) |
| 6 | [🔀 Git](#-git) |
| 7 | [🐳 Docker](#-docker) |
| 8 | [🌐 Nginx](#-nginx) |
| 9 | [🔐 SSH](#-ssh) |
| 10 | [☁️ SSH qua Cloudflare Tunnel](#️-ssh-qua-cloudflare-tunnel) |
| 11 | [📁 File & Directory](#-file--directory) |
| 12 | [🔍 Search & View Files](#-search--view-files) |
| 13 | [⚙️ Process & System](#️-process--system) |
| 14 | [🌍 Network](#-network) |
| 15 | [👤 User, Group, Sudo & SELinux](#-user-group-sudo--selinux) |
| 16 | [🕐 Cron Jobs](#-cron-jobs) |
| 17 | [🔧 Systemd Service](#-systemd-service) |
| 18 | [🔥 Firewall (UFW)](#-firewall-ufw) |
| 19 | [📊 Log Management & Journalctl](#-log-management--journalctl) |
| 20 | [🔄 Rsync](#-rsync) |
| 21 | [☁️ Cloudflared (Cloudflare Tunnel)](#️-cloudflared-cloudflare-tunnel) |
| 22 | [🔁 Rclone](#-rclone) |
| 23 | [🔒 Tailscale](#-tailscale) |
| 24 | [🔎 Troubleshoot Port & Process](#-troubleshoot-port--process) |
| 25 | [🐍 Python & Virtual Environment](#-python--virtual-environment) |
| 26 | [⚙️ Run Program as a Daemon Service](#️-run-program-as-a-daemon-service-systemd) |
| 27 | [🔐 SSL/TLS & Certbot](#-ssltls--certbot) |
| 28 | [🌐 curl Advanced](#-curl-advanced) |
| 29 | [✂️ Sed & Awk](#️-sed--awk) |
| 30 | [💾 Disk Management](#-disk-management) |
| 31 | [📜 Bash Scripting](#-bash-scripting) |
| 32 | [🔥 iptables](#-iptables) |
| 33 | [🔍 nmap](#-nmap) |
| 34 | [📝 Tips & Tricks](#-tips--tricks) |
| 35 | [🔖 Alias & Function in `.bashrc`](#-alias--function-in-bashrc) |
| 36 | [🟩 NVM & Node.js](#-nvm--nodejs) |

---

## 📦 Package Management

```bash
# Debian / Ubuntu
sudo apt update && sudo apt upgrade -y
sudo apt install <package>
sudo apt remove <package>           # Remove package, keep config files
sudo apt purge <package>            # Remove package + delete config files
sudo apt purge <package>*           # Remove all related packages (wildcard)
sudo apt autoremove                 # Remove unused dependencies
sudo apt autoclean                  # Remove outdated cached packages
sudo apt clean                      # Remove all downloaded package cache

# RHEL / CentOS / Fedora
sudo yum update -y
sudo dnf install <package>
sudo dnf remove <package>
sudo dnf autoremove
```

---

## 🗄️ MySQL / MariaDB

### Connect

```bash
# Login with password
mysql -u root -p
mariadb -u root -p

# MariaDB with no root password — use sudo instead of entering password
sudo mariadb -u root
sudo mariadb -u root -e "SHOW DATABASES;"    # Run command directly, no shell

# Connect to a specific database
sudo mariadb -u root mydb
mysql -u root -p mydb
```

### Create Database

```bash
# Create database with standard charset (supports Unicode, emoji)
sudo mariadb -u root <<EOF
CREATE DATABASE IF NOT EXISTS mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
EOF

# Or enter shell then run
sudo mariadb -u root
> CREATE DATABASE IF NOT EXISTS mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
> EXIT;
```

### Backup

```bash
# Backup a specific database
mysqldump -u root -p mydb > mydb_backup.sql
sudo mysqldump -u root mydb > mydb_backup.sql          # MariaDB no password

# Backup with gzip compression
mysqldump -u root -p mydb | gzip > mydb_backup_$(date +%F).sql.gz
sudo mysqldump -u root mydb | gzip > mydb_backup_$(date +%F).sql.gz

# Backup all databases
mysqldump -u root -p --all-databases > all_backup.sql

# Backup multiple databases
mysqldump -u root -p --databases db1 db2 db3 > multi_backup.sql
```

### Restore

```bash
# Restore from .sql file
mysql -u root -p mydb < mydb_backup.sql
sudo mariadb -u root mydb < mydb_backup.sql            # MariaDB no password

# Restore from .sql.gz — using zcat (keeps original file intact)
zcat mydb_backup.sql.gz | sudo mariadb -u root mydb

# Restore .sql.gz with progress bar (requires pv)
pv mydb_backup.sql.gz | zcat | sudo mariadb -u root mydb

# Restore all databases
mysql -u root -p < all_backup.sql
sudo mariadb -u root < all_backup.sql

# Quick check after restore
sudo mariadb -u root -e "SHOW TABLES FROM mydb;" | head -20
sudo mariadb -u root -e "SELECT COUNT(*) FROM mydb.some_table;"
```

> 💡 `pv` shows transfer speed and progress % when restoring large files. Install: `sudo apt install pv`

### User Management

```bash
# Create user and grant privileges
CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON mydb.* TO 'user'@'localhost';
FLUSH PRIVILEGES;

# Drop user
DROP USER 'user'@'localhost';
```

---

## 🐘 PostgreSQL

```bash
# Backup
pg_dump -U postgres mydb > mydb_backup.sql
pg_dumpall -U postgres > all_backup.sql

# Restore
psql -U postgres mydb < mydb_backup.sql

# Connect
psql -U postgres -d mydb -h localhost
```

---

## 🔴 Redis

### Connect & CLI Basics

```bash
# Connect redis-cli
redis-cli                                      # Localhost default
redis-cli -h 127.0.0.1 -p 6379               # Specify host and port
redis-cli -h 127.0.0.1 -p 6379 -a password   # With password
redis-cli --no-auth-warning -a password       # Suppress password warning

# Test connection
redis-cli ping                                 # Returns PONG if OK

# Server info
redis-cli info
redis-cli info memory                          # Memory section only
redis-cli info replication                     # Master/replica status
```

### Key Management

```bash
# List keys
KEYS *                                         # All keys (avoid on large production)
KEYS user:*                                    # Keys matching pattern
SCAN 0 MATCH user:* COUNT 100                 # Safer alternative to KEYS in production

# Basic operations
SET mykey "hello"
SET mykey "hello" EX 3600                     # With TTL of 1 hour (seconds)
GET mykey
DEL mykey
EXISTS mykey                                   # Returns 1 if exists, 0 if not
TYPE mykey                                     # string / list / hash / set / zset
TTL mykey                                      # Remaining lifetime (seconds), -1 if no TTL
EXPIRE mykey 3600                              # Set TTL on existing key
PERSIST mykey                                  # Remove TTL — key lives forever

# Rename / move
RENAME oldkey newkey
MOVE mykey 1                                   # Move key to database 1
```

### Common Data Types

```bash
# String
INCR counter                                   # Increment by 1
INCRBY counter 5                               # Increment by 5
APPEND mykey " world"

# Hash
HSET user:1 name "Viet" email "viet@example.com"
HGET user:1 name
HGETALL user:1                                 # Get all fields
HDEL user:1 email
HEXISTS user:1 name

# List
LPUSH mylist "a" "b" "c"                       # Push to head
RPUSH mylist "x"                               # Push to tail
LRANGE mylist 0 -1                             # Get all elements
LLEN mylist

# Set
SADD myset "apple" "banana" "cherry"
SMEMBERS myset
SISMEMBER myset "apple"                        # Check if element exists in set
SREM myset "banana"
```

### Flush & Backup

```bash
# Delete data (be careful!)
FLUSHDB                                        # Flush current database
FLUSHALL                                       # Flush all databases

# Backup — create RDB snapshot
redis-cli BGSAVE                               # Async backup, non-blocking
redis-cli LASTSAVE                             # Unix timestamp of last backup
# Default dump.rdb location: /var/lib/redis/dump.rdb

# View current config
redis-cli CONFIG GET dir                       # RDB storage directory
redis-cli CONFIG GET save                      # Auto-save configuration
```

### Service

```bash
sudo systemctl start redis
sudo systemctl stop redis
sudo systemctl restart redis
sudo systemctl status redis
sudo systemctl enable redis                    # Auto-start on boot

# View logs
sudo journalctl -u redis -f
tail -f /var/log/redis/redis-server.log
```

---

## 🍃 MongoDB

### Connect & mongosh

```bash
# Connect
mongosh                                        # Localhost default
mongosh "mongodb://localhost:27017"
mongosh "mongodb://user:password@localhost:27017/mydb"
mongosh "mongodb://user:password@host:27017/mydb?authSource=admin"

# Connect and run command directly
mongosh --eval "db.adminCommand({ ping: 1 })"
mongosh mydb --eval "db.users.countDocuments()"
```

### Database & Collection

```bash
# Inside mongosh shell
show dbs                                       # List all databases
use mydb                                       # Switch to / create database
db                                             # Show current database
db.dropDatabase()                              # Drop current database

show collections                               # List collections
db.createCollection("users")
db.users.drop()                                # Drop collection
db.stats()                                     # Database statistics
db.users.stats()                               # Collection statistics
```

### CRUD

```bash
# Insert
db.users.insertOne({ name: "Viet", email: "viet@example.com", age: 30 })
db.users.insertMany([{ name: "A" }, { name: "B" }])

# Find / Query
db.users.find()                                # All documents
db.users.find({ age: { $gt: 25 } })           # age > 25
db.users.find({ name: "Viet" }, { email: 1 }) # Return email field only
db.users.findOne({ name: "Viet" })
db.users.countDocuments({ age: { $gt: 25 } })

# Update
db.users.updateOne({ name: "Viet" }, { $set: { age: 31 } })
db.users.updateMany({ age: { $lt: 18 } }, { $set: { status: "minor" } })
db.users.replaceOne({ name: "Viet" }, { name: "Viet", age: 31 })  # Replace entire document

# Delete
db.users.deleteOne({ name: "Viet" })
db.users.deleteMany({ status: "inactive" })
db.users.deleteMany({})                        # Delete all documents (keeps collection)
```

### Index

```bash
db.users.createIndex({ email: 1 })            # Ascending index
db.users.createIndex({ email: 1 }, { unique: true })  # Unique index
db.users.createIndex({ name: 1, age: -1 })    # Compound index
db.users.getIndexes()                          # List all indexes
db.users.dropIndex("email_1")                 # Drop index by name
```

### Backup & Restore

```bash
# Backup
mongodump --db mydb --out /backup/mongo/
mongodump --uri "mongodb://user:pass@localhost/mydb" --out /backup/

# Compressed backup
mongodump --db mydb --archive=/backup/mydb_$(date +%F).gz --gzip

# Restore
mongorestore --db mydb /backup/mongo/mydb/
mongorestore --uri "mongodb://user:pass@localhost" /backup/mongo/

# Restore from compressed archive
mongorestore --archive=/backup/mydb_2026-03-15.gz --gzip
```

### Service & Monitoring

```bash
sudo systemctl start mongod
sudo systemctl stop mongod
sudo systemctl restart mongod
sudo systemctl status mongod
sudo systemctl enable mongod                   # Auto-start on boot

# View logs
sudo journalctl -u mongod -f
tail -f /var/log/mongodb/mongod.log

# Monitoring inside mongosh
db.serverStatus()                              # Full server info
db.serverStatus().connections                  # Current connection count
db.currentOp()                                 # Currently running operations
db.killOp(<opid>)                              # Kill an operation
```

---

## 🔀 Git

### Configure Username & Email

```bash
# Global config (applies to all repos on this machine)
git config --global user.name "Tran Quoc Viet"
git config --global user.email "your@email.com"

# Local config (current repo only — overrides global)
git config user.name "Tran Quoc Viet"
git config user.email "work@company.com"

# Check config
git config --list
git config user.name
git config user.email

# Show which file each config value comes from
git config --list --show-origin
```

> 💡 Use **local config** to separate personal and work accounts on the same machine.

### Initialize a new repo and push to remote

```bash
echo "# my-project" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/<user>/<repo>.git
git push -u origin main
```

### Connect an existing local repo to remote

```bash
git remote add origin https://github.com/<user>/<repo>.git
git branch -M main
git push -u origin main
```

> 💡 `-u` sets the upstream so future `git push` / `git pull` work without specifying the remote and branch.

### Manage remotes

```bash
git remote -v                                    # List remotes
git remote add origin <url>                      # Add remote
git remote set-url origin <new-url>              # Change remote URL
git remote remove origin                         # Remove remote
```

### Basics

```bash
git clone https://github.com/user/repo.git
git clone git@github.com:user/repo.git          # SSH

git status
git add .
git commit -m "message"
git push origin main
git pull origin main
```

### Branches

```bash
git branch                                       # List branches
git branch feature/new-feature                   # Create new branch
git checkout feature/new-feature                 # Switch to branch
git checkout -b feature/new-feature              # Create and switch
git merge feature/new-feature                    # Merge into current branch
git branch -d feature/new-feature               # Delete local branch
git push origin --delete feature/new-feature    # Delete remote branch
```

### Undo / Reset

```bash
git revert <commit-hash>                         # Safe undo (creates new commit)
git reset --soft HEAD~1                          # Undo commit, keep changes staged
git reset --hard HEAD~1                          # Undo commit, discard changes
git stash                                        # Stash current changes
git stash pop                                    # Re-apply stashed changes
```

### Tags

```bash
git tag v1.0.0
git tag -a v1.0.0 -m "Release 1.0.0"
git push origin v1.0.0
git push origin --tags
```

### Fix: Accidentally Committed Files

Use this when you've committed files that should be ignored (e.g. `__pycache__`, `.env`, build artifacts, editor config).

**Step 1 — Remove from git tracking (keep files locally):**

```bash
# Remove specific paths
git rm -r --cached src/__pycache__ tests/__pycache__ src/myapp.egg-info

# Or remove all tracked files that match .gitignore rules (after updating .gitignore)
git rm -r --cached .
git add .
```

**Step 2 — Add to `.gitignore`:**

```bash
# Python
echo -e "__pycache__/\n*.pyc\n*.pyo\n*.egg-info/\ndist/\nbuild/\n.eggs/" >> .gitignore

# Node.js
echo -e "node_modules/\ndist/\n.env\n*.log" >> .gitignore

# General
echo -e ".env\n.DS_Store\n*.log\n.idea/\n.vscode/" >> .gitignore
```

**Step 3 — Commit the fix:**

```bash
git add .gitignore
git commit -m "remove tracked build artifacts, update .gitignore"
git push
```

> 💡 **Best practice:** always create `.gitignore` before the first `git add .`, and run `git status` to review staged files before committing. GitHub maintains a collection of ready-made `.gitignore` templates at [github.com/github/gitignore](https://github.com/github/gitignore).

---

## 🐳 Docker

### Image & Container

```bash
docker build -t myapp:latest .
docker pull nginx:latest
docker images
docker rmi <image-id>

docker run -d -p 80:80 --name myapp nginx
docker run -it ubuntu /bin/bash               # Interactive mode
docker ps                                      # Running containers
docker ps -a                                   # All containers
docker stop <container>
docker start <container>
docker rm <container>
docker logs -f <container>
docker exec -it <container> /bin/bash
```

### Docker Compose

```bash
docker compose up -d                           # Start services in background
docker compose down                            # Stop and remove containers
docker compose down -v                         # Also remove volumes
docker compose logs -f
docker compose ps
docker compose restart <service>
docker compose pull                            # Pull latest images
```

### Cleanup

```bash
docker system prune -a                         # Remove all unused resources
docker volume prune                            # Remove unused volumes
docker image prune                             # Remove dangling images
```

---

## 🌐 Nginx

```bash
sudo nginx -t                                  # Test configuration
sudo systemctl reload nginx
sudo systemctl restart nginx
sudo systemctl status nginx

# View logs
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

---

## 🔐 SSH

### Basic Connection

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "email@example.com"
ssh-keygen -t rsa -b 4096 -C "email@example.com"

# Copy public key to server
ssh-copy-id user@server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server   # Specify key
ssh-copy-id -p 2222 user@server                     # Custom port

# Connect
ssh user@server
ssh -p 2222 user@server                        # Custom port
ssh -i ~/.ssh/id_rsa user@server               # Specify key file
ssh -v user@server                             # Verbose, for debugging

# SSH Tunnel (Port Forwarding)
ssh -L 3306:localhost:3306 user@server         # Local: access remote DB locally
ssh -R 8080:localhost:80 user@server           # Remote: expose local port to server
ssh -N -f -L 3306:localhost:3306 user@server   # Run in background, no shell
```

### SSH Config (~/.ssh/config)

Save connection profiles to avoid long commands:

```
Host myserver
    HostName 123.45.67.89
    User ubuntu
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

Host prod
    HostName prod.example.com
    User deploy
    IdentityFile ~/.ssh/id_rsa_prod

Host bastion
    HostName bastion.example.com
    User ec2-user
    IdentityFile ~/.ssh/id_rsa
```

```bash
# After config, just use:
ssh myserver
ssh prod
```

### Run Remote Commands via SSH

```bash
# Run a single command
ssh user@server "ls -lah /var/www"
ssh user@server "df -h"
ssh user@server "sudo systemctl status nginx"

# Run chained commands
ssh user@server "cd /var/www && git pull && systemctl restart myapp"

# Run local script on remote server
ssh user@server 'bash -s' < deploy.sh

# Run multiple commands via heredoc
ssh user@server << 'EOF'
  cd /home/ubuntu/myapp
  git pull origin main
  source venv/bin/activate
  pip install -r requirements.txt
  sudo systemctl restart myapp
  echo "Deploy done!"
EOF

# Capture remote output locally
ssh user@server "cat /var/log/nginx/error.log" > local_error.log
ssh user@server "mysqldump -u root -p'pass' mydb" > backup.sql

# Pipe local data to remote
cat backup.sql | ssh user@server "mysql -u root -p'pass' mydb"
```

### SCP — Copy Files over SSH

```bash
# Upload: local → server
scp file.txt user@server:/home/ubuntu/
scp file.txt user@server:/home/ubuntu/newname.txt    # Rename on upload
scp -r /local/folder user@server:/home/ubuntu/       # Copy entire directory
scp -P 2222 file.txt user@server:/tmp/               # Custom port

# Download: server → local
scp user@server:/home/ubuntu/file.txt .              # Download to current dir
scp user@server:/var/log/nginx/error.log ./error.log
scp -r user@server:/home/ubuntu/myapp ./myapp        # Download entire directory

# Copy between two remote servers (no local copy)
scp user1@server1:/path/file.txt user2@server2:/path/

# Useful flags
scp -C file.txt user@server:/tmp/             # Compress during transfer
scp -p file.txt user@server:/tmp/             # Preserve timestamps & permissions
scp -v file.txt user@server:/tmp/             # Verbose / debug

# Use SSH config aliases
scp file.txt myserver:/home/ubuntu/
scp -r prod:/var/backups ./backups
```

### SFTP — Interactive File Transfer

```bash
sftp user@server
sftp -P 2222 user@server

# SFTP shell commands
ls                    # List files on server
lls                   # List local files
pwd / lpwd            # Current directory (remote / local)
cd /var/www           # Change remote directory
lcd ~/Downloads       # Change local directory
get file.txt          # Download to local
get -r folder/        # Download entire directory
put file.txt          # Upload to server
put -r folder/        # Upload entire directory
rm file.txt           # Delete remote file
mkdir newfolder       # Create remote directory
exit                  # Exit
```

---

## ☁️ SSH via Cloudflare Tunnel

> Secure SSH access **without exposing port 22 to the public**, no static IP required. All traffic through Cloudflare Tunnel is fully encrypted.

### Architecture

```
Local Machine  →  cloudflared (proxy)  →  Cloudflare Edge  →  Tunnel  →  Production Server :22
```

### Production Server — Add SSH to tunnel config

```bash
# Open tunnel config file
sudo nano /etc/cloudflared/config.yml
```

Add SSH ingress rule **before** the catch-all 404:

```yaml
ingress:
  - hostname: ssh.production.com
    service: ssh://localhost:22
  # ... other rules ...
  - service: http_status:404   # catch-all — must be last
```

Restart tunnel to apply:

```bash
sudo systemctl restart cloudflared
sudo systemctl status cloudflared   # Verify tunnel is running
```

Go to **Cloudflare Dashboard → DNS** and create a record:

```
Type:     CNAME
Name:     ssh.production.com
Target:   <TUNNEL_UUID>.cfargotunnel.com
Proxy:    Proxied (orange)  ✅
```

> Get `TUNNEL_UUID` from: `cloudflared tunnel list`

---

### Local Machine — Install cloudflared + configure SSH proxy

```bash
# 1. Install cloudflared (Ubuntu/Debian)
curl -fsSL https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb \
  -o /tmp/cloudflared.deb
sudo dpkg -i /tmp/cloudflared.deb
cloudflared --version   # Verify installation

# 2. (If production server requires SSH key) Generate key and copy to server
ssh-keygen -t ed25519 -C "your@email.com"   # Creates key pair at ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub                   # Copy this output

# On production server, paste public key into authorized_keys for the target user
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo "<paste_public_key_here>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# 3. Add SSH proxy config (with IdentityFile if using key auth)
cat >> ~/.ssh/config << 'EOF'

Host ssh.production.com
    HostName ssh.production.com
    User <production_user>
    IdentityFile ~/.ssh/id_ed25519
    ProxyCommand cloudflared access ssh --hostname %h
EOF
```

SSH normally — cloudflared handles the tunnel automatically:

```bash
ssh ssh.production.com

# Or use SCP over the tunnel
scp file.txt ssh.production.com:/home/<production_user>/
```

---

### Troubleshooting

```bash
# Check tunnel is active on production server
cloudflared tunnel list
cloudflared tunnel info <TUNNEL_UUID>

# View tunnel logs on production server
sudo journalctl -u cloudflared -f

# Manually test from local (bypassing SSH config)
cloudflared access ssh --hostname ssh.production.com

# Verify DNS record is correct
dig ssh.production.com CNAME
```

---

## 📁 File & Directory

```bash
ls -lah                                        # Detailed listing
find /path -name "*.log" -mtime +7             # Files older than 7 days
find /path -type f -size +100M                 # Files larger than 100MB
du -sh /var/log/*                              # Directory sizes
df -h                                          # Disk usage
cp -r source/ dest/
mv source dest
rm -rf /path/to/dir                            # Force remove directory

# Compress / Extract
tar -czvf archive.tar.gz /path/to/dir          # Compress
tar -xzvf archive.tar.gz                       # Extract
tar -xzvf archive.tar.gz -C /target/dir        # Extract to specific directory

zip -r archive.zip /path/to/dir
unzip archive.zip
```

---

## 🔍 Search & View Files

```bash
cat file.txt
less file.txt
tail -f /var/log/syslog                        # View logs realtime
tail -n 100 file.log                           # Last 100 lines
head -n 50 file.log                            # First 50 lines

grep "error" /var/log/nginx/error.log
grep -r "keyword" /etc/                        # Recursive search
grep -i "error" file.log                       # Case-insensitive
grep -n "error" file.log                       # Show line numbers

# Combine
cat access.log | grep "404" | wc -l           # Count 404 errors
```

---

## ⚙️ Process & System

```bash
top
htop                                           # Install: apt install htop
ps aux | grep nginx
kill -9 <PID>
pkill nginx

free -h                                        # RAM
vmstat 1 5                                     # CPU/Memory stats
iostat                                         # Disk I/O

uptime
uname -a
hostname

# Time & timezone
timedatectl                                    # Show system time, timezone, NTP status
sudo timedatectl set-timezone Asia/Ho_Chi_Minh # Set timezone to Vietnam
sudo timedatectl set-ntp true                  # Enable NTP sync
timedatectl list-timezones | grep Asia         # Search available timezones
date                                           # Show current date and time
```

---

## 🌍 Network

```bash
ip addr                                        # Show IP addresses
ip route                                       # Routing table
netstat -tuln                                  # Open ports
ss -tuln                                       # Faster alternative to netstat
curl -I https://example.com                    # HTTP headers
wget https://example.com/file.zip
ping -c 4 google.com
traceroute google.com
nslookup example.com
dig example.com

# Check if port is open
nc -zv server 3306
telnet server 22

# Hardware
lspci                                          # List PCI devices (network cards, GPU...)
lsusb                                          # List connected USB devices
```

### Static IP Setup (Debian/Ubuntu)

```bash
# Open network config file
sudo nano /etc/network/interfaces
```

Add static IP config for the interface (typically `eth0` or `ens3`):

```
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

```bash
# Restart network service
sudo systemctl restart networking
# Or on older systems
sudo /etc/init.d/networking restart

# Verify IP after configuring
ip addr show eth0
ip route
```

> 💡 On Ubuntu 18.04+, use **Netplan** instead: config file at `/etc/netplan/*.yaml`, apply with `sudo netplan apply`.

---

## 👤 User, Group, Sudo & SELinux

### User Management

```bash
# Create user
useradd -m username                            # Create user with home directory
useradd -m -s /bin/bash -c "Full Name" username
adduser username                               # Interactive mode, easier

# Set / change password
passwd username
echo "username:newpassword" | chpasswd         # Non-interactive

# Modify user
usermod -aG sudo username                      # Add to sudo group
usermod -aG docker,www-data username           # Add to multiple groups
usermod -s /bin/bash username                  # Change default shell
usermod -l newname oldname                     # Rename user
usermod -d /new/home -m username               # Change home directory

# Drop user
userdel username                               # Keep home directory
userdel -r username                            # Drop user + home dir + mail

# View user info
id username                                    # UID, GID, groups
whoami                                         # Current user
who                                            # Who is logged in
w                                              # Logged in users + activity
last                                           # Login history
lastfail                                       # Failed login history
cat /etc/passwd | grep username                # User info in system
```

### Group Management

```bash
# Create & delete group
groupadd developers
groupdel developers

# Add / remove user from group
usermod -aG developers username                # Add (keeps existing groups)
gpasswd -a username developers                 # Alternative add
gpasswd -d username developers                 # Remove from group

# # View groups
groups username                                # Groups of a user
cat /etc/group | grep developers               # View group members
getent group developers

# Switch active group in current session
newgrp developers
```

### File Permissions (chmod / chown)

```bash
# chmod — change file permissions
# Numeric mode: r=4, w=2, x=1
chmod 755 file          # rwxr-xr-x (owner: rwx, group: r-x, others: r-x)
chmod 644 file          # rw-r--r-- (owner: rw, group: r, others: r)
chmod 600 file          # rw------- (owner only, use for SSH keys/secrets)
chmod 777 file          # Not recommended — everyone has full access
chmod -R 755 /var/www   # Recursive

# chmod — symbolic mode
chmod u+x script.sh     # Add execute for owner
chmod g-w file          # Remove write for group
chmod o+r file          # Add read for others
chmod a+x script.sh     # Add execute for all

# chown — change ownership
chown user file
chown user:group file
chown -R www-data:www-data /var/www/html
chown -R ubuntu:ubuntu /home/ubuntu/myapp

# Xem permission
ls -lah /var/www
stat file.txt
```

### Sudo & Sudoers

```bash
# Switch user
sudo su -                                      # Switch to root (login shell)
sudo su - username                             # Switch to another user
sudo -u www-data bash                          # Run shell as specific user
sudo -i                                        # Interactive root shell

# Run commands with sudo
sudo systemctl restart nginx
sudo -u postgres psql                          # Run command as postgres user

# Edit sudoers — always use visudo, never edit directly
sudo visudo
sudo visudo -f /etc/sudoers.d/myapp            # Recommended: separate file

# Sudoers syntax:
# user  HOST=(RUNAS) COMMANDS
ubuntu  ALL=(ALL:ALL) ALL                      # Full sudo access
deploy  ALL=(ALL) NOPASSWD: /bin/systemctl     # No password for systemctl
deploy  ALL=(ALL) NOPASSWD: ALL                # No password for everything

# View sudo privileges
sudo -l
sudo -l -U username                            # View another user's privileges (as root)

# Add user to sudo group
usermod -aG sudo username                      # Debian / Ubuntu
usermod -aG wheel username                     # RHEL / CentOS / Fedora
```

### SELinux (RHEL / CentOS / Fedora)

> **Note:** SELinux is common on RHEL/CentOS/Fedora. Debian/Ubuntu uses AppArmor instead.

```bash
# Check status
sestatus                                       # Detailed status
getenforce                                     # Enforcing / Permissive / Disabled

# Change mode temporarily (no reboot needed)
sudo setenforce 0                              # Switch to Permissive (log but don't block)
sudo setenforce 1                              # Switch to Enforcing (block violations)

# Change mode permanently — edit /etc/selinux/config
# SELINUX=enforcing   → fully active
# SELINUX=permissive  → log only, useful for debugging
# SELINUX=disabled    → completely off (requires reboot)
sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config

# View SELinux denial logs
sudo ausearch -m avc -ts recent               # Recent denials
sudo ausearch -m avc -ts today                # Today's denials
sudo journalctl | grep "SELinux is preventing"

# View context of files/processes
ls -Z /var/www/html                            # SELinux context of file
ps auxZ | grep nginx                           # Process context

# Change file context
sudo chcon -t httpd_sys_content_t /var/www/html/myfile
sudo chcon -R -t httpd_sys_content_t /var/www/html

# Restore default context
sudo restorecon -v /var/www/html/myfile
sudo restorecon -Rv /var/www/html              # Recursive

# Allow service to use non-default port
sudo semanage port -a -t http_port_t -p tcp 8080
sudo semanage port -l | grep http              # View allowed ports

# Allow httpd to make network connections (needed for reverse proxy)
sudo setsebool -P httpd_can_network_connect 1
sudo setsebool -P httpd_can_network_connect_db 1

# Auto-generate policy from denial logs
sudo ausearch -m avc -ts recent | audit2allow -M mypolicy
sudo semodule -i mypolicy.pp
```

### AppArmor (Debian / Ubuntu)

```bash
# Check status
sudo apparmor_status
sudo aa-status

# Enable/disable profile for a service
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx   # Enable enforce mode
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx  # Switch to complain (log only)
sudo aa-disable /etc/apparmor.d/usr.sbin.nginx   # Disable profile

# Reload profile after editing
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx

# View violations in logs
sudo journalctl | grep "apparmor"
sudo dmesg | grep "apparmor"
```

---

## 🕐 Cron Jobs

```bash
crontab -e                                     # Edit cron for current user
crontab -l                                     # List current crons
crontab -r                                     # Remove all crons

# Syntax: minute hour day month weekday command
# ┌─ minute   (0-59)
# │ ┌─ hour    (0-23)
# │ │ ┌─ day    (1-31)
# │ │ │ ┌─ month  (1-12)
# │ │ │ │ ┌─ weekday (0-7, 0 and 7 = Sunday)
# │ │ │ │ │
# * * * * * /path/to/command

0 2 * * * /scripts/backup.sh                  # Every day at 2:00 AM
0 0 * * 0 /scripts/weekly.sh                  # Every Sunday at midnight
*/5 * * * * /scripts/check.sh                 # Every 5 minutes
```

---

## 🔧 Systemd Service

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl enable nginx                    # Start on boot
sudo systemctl disable nginx
sudo systemctl status nginx

journalctl -u nginx -f                        # View logs service
journalctl -u nginx --since "1 hour ago"
journalctl -xe                                 # Recent error logs
```

---

## 🔥 Firewall (UFW)

```bash
sudo ufw status
sudo ufw enable
sudo ufw allow 22
sudo ufw allow 80/tcp
sudo ufw allow from 192.168.1.0/24 to any port 3306
sudo ufw deny 23
sudo ufw delete allow 80
sudo ufw reset
```

---

## 📊 Log Management & Journalctl

```bash
# View logs realtime
journalctl -f
journalctl -u nginx -f                        # Follow a specific service

# Filter by time
journalctl --since today
journalctl --since "1 hour ago"
journalctl --since "2024-01-01" --until "2024-01-31"
journalctl --since "2024-01-01 08:00:00"

# Filter by service / unit
journalctl -u nginx
journalctl -u nginx -u mysql                  # Multiple services
journalctl -u docker --since today

# Filter by severity level
journalctl -p err                             # Errors and above
journalctl -p warning
journalctl -p debug
# Levels: emerg(0) alert(1) crit(2) err(3) warning(4) notice(5) info(6) debug(7)

# Filter by process / PID
journalctl _PID=1234
journalctl _COMM=nginx                        # By process name

# Lines and format
journalctl -n 100                             # Last 100 lines
journalctl -n 50 -u nginx
journalctl -o json-pretty                     # JSON output
journalctl -o short-iso                       # ISO timestamp format

# Boot logs
journalctl -b                                 # Current boot
journalctl -b -1                              # Previous boot
journalctl --list-boots                       # List all boots

# Search
journalctl -u nginx | grep "error"
journalctl --grep="Out of memory"

# Disk usage and cleanup
journalctl --disk-usage
journalctl --vacuum-size=500M                 # Keep only 500MB
journalctl --vacuum-time=30d                  # Remove logs older than 30 days

# Rotate logs manually
logrotate -f /etc/logrotate.conf

# Delete old compressed log files
find /var/log -name "*.gz" -mtime +30 -delete
```

---

## 🔄 Rsync

```bash
# Sync local directories
rsync -avz /source/ /destination/

# Sync to remote server
rsync -avz /local/path/ user@server:/remote/path/

# Sync from remote server
rsync -avz user@server:/remote/path/ /local/path/

# Delete files at destination that no longer exist at source
rsync -avz --delete /source/ /destination/

# Dry run (preview only, no changes)
rsync -avzn /source/ /destination/
```

---

## ☁️ Cloudflared (Cloudflare Tunnel)

```bash
# Install
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 \
  -o /usr/local/bin/cloudflared
chmod +x /usr/local/bin/cloudflared

# Login
cloudflared tunnel login

# Create and manage tunnels
cloudflared tunnel create my-tunnel
cloudflared tunnel list
cloudflared tunnel info my-tunnel
cloudflared tunnel delete my-tunnel

# Run tunnel (forward traffic to localhost)
cloudflared tunnel run --url http://localhost:3000 my-tunnel

# Configure DNS route
cloudflared tunnel route dns my-tunnel subdomain.example.com

# Install as systemd service
cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
sudo systemctl status cloudflared

# View logs
journalctl -u cloudflared -f

# Quick temporary tunnel (no login required)
cloudflared tunnel --url http://localhost:8080
```

### Ví dụ file config `~/.cloudflared/config.yml`

```yaml
tunnel: <tunnel-id>
credentials-file: /home/user/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: app.example.com
    service: http://localhost:3000
  - hostname: api.example.com
    service: http://localhost:8080
  - service: http_status:404
```

```bash
# Run with config file
cloudflared tunnel --config ~/.cloudflared/config.yml run
```

---

## 🔁 Rclone

```bash
# Install
curl https://rclone.org/install.sh | sudo bash

# Configure remote (Google Drive, S3, Backblaze, SFTP...)
rclone config
rclone listremotes                            # List configured remotes

# List files
rclone ls remote:bucket/path
rclone lsd remote:                            # List directories only
rclone lsf remote:bucket                     # Concise format

# Copy (one-way, does not delete destination)
rclone copy /local/path remote:bucket/path
rclone copy remote:bucket/path /local/backup

# Sync (one-way, deletes destination files not in source)
rclone sync /local/path remote:bucket/path
rclone sync remote:bucket/path /local/path

# Move and delete
rclone move /local/path remote:bucket/path
rclone delete remote:bucket/path/file.txt
rclone purge remote:bucket/old-folder        # Delete entire remote directory

# Dry run before syncing
rclone sync /local/path remote:bucket --dry-run
rclone sync /local/path remote:bucket -v     # Verbose

# Mount remote as local filesystem
rclone mount remote:bucket /mnt/remote --daemon
fusermount -u /mnt/remote                    # Unmount

# Backup with filters
rclone copy /var/www remote:backup/www \
  --exclude "*.log" \
  --exclude ".git/**" \
  --progress

# Automated nightly backup via cron
0 2 * * * rclone sync /var/www remote:backup/www --log-file /var/log/rclone.log
```

---

## 🔒 Tailscale

```bash
# Install (Debian/Ubuntu)
curl -fsSL https://tailscale.com/install.sh | sh

# Login to Tailscale network
sudo tailscale up
sudo tailscale up --authkey=<auth-key>        # Headless, no browser needed

# View status and IP
tailscale status
tailscale ip -4                               # Show IPv4 address

# Disconnect / reconnect
sudo tailscale down
sudo tailscale up

# Exit Node (route all traffic through another node)
sudo tailscale up --exit-node=<node-ip>
sudo tailscale up --exit-node=               # Disable exit node

# Advertise current machine as Exit Node
sudo tailscale up --advertise-exit-node
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Subnet Router (share local network)
sudo tailscale up --advertise-routes=192.168.1.0/24

# Tailscale SSH
sudo tailscale up --ssh
ssh user@<tailscale-hostname>

# Ping a node in the network
tailscale ping <node-name-or-ip>

# View logs
journalctl -u tailscaled -f

# Update and logout
sudo tailscale update
sudo tailscale logout
```

---

## 🔎 Troubleshoot Port & Process

### Find process using a port

```bash
# lsof — most detailed
sudo lsof -i :5000                            # Process using port 5000
sudo lsof -i :5000 -Pn                        # -P: no port name resolve, -n: no hostname resolve
sudo lsof -i :80 -i :443                      # Multiple ports
sudo lsof -i TCP -P -n                        # All open TCP ports
sudo lsof -i UDP -P -n                        # All open UDP ports

# ss — faster than netstat
sudo ss -tuln                                 # All listening ports
sudo ss -tulnp                                # Include process name
sudo ss -tulnp | grep :5000                   # Filter specific port
sudo ss -tnp state established                # Active connections

# netstat (requires net-tools)
sudo netstat -tuln                            # All listening ports
sudo netstat -tulnp                           # Include PID/process
sudo netstat -tulnp | grep :5000
sudo netstat -an | grep ESTABLISHED           # Active connections
sudo netstat -an | grep LISTEN

# fuser — returns PID quickly
sudo fuser 5000/tcp                           # PID using port
sudo fuser -v 5000/tcp                        # Verbose
sudo fuser -k 5000/tcp                        # Kill process on port
```

### Port → PID → Service workflow

```bash
# Step 1: Find PID from port
sudo lsof -i :5000 -Pn
sudo ss -tulnp | grep :5000

# Step 2: Inspect the process
ps aux | grep <PID>
cat /proc/<PID>/cmdline | tr '\0' ' '         # Full command being run
ls -la /proc/<PID>/exe                        # Binary path

# Step 3: Find the systemd service
sudo systemctl status <PID>
# or find by process name
sudo systemctl status nginx
sudo systemctl status $(ps -p <PID> -o comm=) # Auto get name from PID
```

### View all running processes

```bash
ps aux                                        # All processes
ps aux --sort=-%cpu | head -10               # Top 10 by CPU
ps aux --sort=-%mem | head -10               # Top 10 by memory
ps aux | grep nginx                           # Filter by name

# Process tree
pstree -p                                     # Show PIDs
pstree -u                                     # Show users

# Realtime
top
htop                                          # Install: apt install htop
```

### Kill process

```bash
kill <PID>                                    # SIGTERM — graceful stop
kill -9 <PID>                                 # SIGKILL — force stop
kill -15 <PID>                                # SIGTERM (default)
pkill nginx                                   # Kill by name
pkill -9 nginx
killall node                                  # Kill all processes named "node"

# Kill process holding a port
sudo fuser -k 5000/tcp
sudo kill -9 $(sudo lsof -t -i :5000)
```

### View open files / sockets for a process

```bash
sudo lsof -p <PID>                            # All files opened by PID
sudo lsof -p <PID> | grep IPv                 # Network connections only
sudo lsof -u www-data                         # All files opened by user
sudo lsof +D /var/www                         # Which processes are using this directory
```

### Practical example

```bash
# Step 1: Find who is using port 3000
sudo lsof -i :3000 -Pn
# OUTPUT:
# COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
# node    1234  ubuntu   20u  IPv4  12345      0t0  TCP *:3000 (LISTEN)

# Step 2: Check the systemd service
sudo systemctl status 1234

# Step 3: View logs
journalctl _PID=1234 -n 50

# Step 4: Check full command and working directory
cat /proc/1234/cmdline | tr '\0' ' '
ls -la /proc/1234/cwd                         # Working directory
```

---

## 🐍 Python & Virtual Environment

### Install Python

```bash
# Check version
python3 --version
pip3 --version

# Install
sudo apt install python3 python3-pip python3-venv -y

# Install multiple Python versions (Ubuntu — deadsnakes PPA)
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.11 python3.11-venv python3.11-dev
```

### Virtual Environment (venv)

```bash
# Create venv
python3 -m venv venv                          # Creates venv/ directory
python3.11 -m venv venv                       # Use specific Python version

# Activate
source venv/bin/activate                      # Linux/macOS
# (venv) prefix appears in prompt

# Deactivate
deactivate

# Remove venv
rm -rf venv/
```

### Package Management inside venv

```bash
# After activating venv
pip install flask
pip install flask==2.3.0                      # Specific version
pip install -r requirements.txt               # Install from file

# Export installed packages
pip freeze > requirements.txt

# View packages
pip list
pip show flask                                # Details of one package

# Upgrade and uninstall
pip install --upgrade flask
pip uninstall flask
pip uninstall -r requirements.txt -y          # Batch uninstall
```

### Run Python apps

```bash
python3 app.py
python3 -m uvicorn app:app --host 0.0.0.0 --port 8000    # FastAPI/ASGI
python3 -m gunicorn app:app -w 4 -b 0.0.0.0:8000         # Flask/WSGI

# Run as module
python3 -m http.server 8080                   # Quick file server
python3 -m json.tool data.json                # Format JSON
```

---

## ⚙️ Run Program as a Daemon Service (Systemd)

Make any program (Python app, Node.js, shell script) start automatically on boot and be managed by systemd.

### Step 1: Create service file

```bash
sudo nano /etc/systemd/system/myapp.service
```

### Example: Python app with venv

```ini
[Unit]
Description=My Python App
After=network.target                          # Start after network is available
Wants=network-online.target

[Service]
Type=simple
User=ubuntu                                   # User to run the app (avoid root)
Group=ubuntu
WorkingDirectory=/home/ubuntu/myapp           # App working directory

# Use Python inside venv
ExecStart=/home/ubuntu/myapp/venv/bin/python app.py

# Or use gunicorn inside venv
# ExecStart=/home/ubuntu/myapp/venv/bin/gunicorn app:app -w 4 -b 0.0.0.0:8000

Restart=always                                # Restart automatically if crashed
RestartSec=5                                  # Wait 5 seconds before restarting
StandardOutput=journal                        # Log stdout to journald
StandardError=journal                         # Log stderr to journald

# Environment variables
Environment="ENV=production"
Environment="PORT=8000"
# Or load from .env file
EnvironmentFile=/home/ubuntu/myapp/.env

[Install]
WantedBy=multi-user.target                    # Start at normal runlevel
```

### Example: Node.js app

```ini
[Unit]
Description=My Node.js App
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/nodeapp
ExecStart=/usr/bin/node server.js
# Or use npm: ExecStart=/usr/bin/npm start
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal
EnvironmentFile=/home/ubuntu/nodeapp/.env

[Install]
WantedBy=multi-user.target
```

### Step 2: Enable and start service

```bash
# Reload systemd to pick up new service file
sudo systemctl daemon-reload

# Enable auto-start on boot
sudo systemctl enable myapp

# Start immediately
sudo systemctl start myapp

# Enable + start in one command
sudo systemctl enable --now myapp

# Check status
sudo systemctl status myapp
```

### Daily management

```bash
sudo systemctl start myapp
sudo systemctl stop myapp
sudo systemctl restart myapp
sudo systemctl reload myapp                   # Reload config without restarting process

# Enable / disable auto-start on boot
sudo systemctl enable myapp
sudo systemctl disable myapp

# View logs realtime
journalctl -u myapp -f

# View last 100 log lines
journalctl -u myapp -n 100

# View logs since last boot
journalctl -u myapp -b
```

### Step 3: Deploy and restart

```bash
# After deploying new code
cd /home/ubuntu/myapp
git pull origin main
source venv/bin/activate
pip install -r requirements.txt
deactivate

sudo systemctl restart myapp
sudo systemctl status myapp
```

### Restart policies

```ini
Restart=no              # Never restart (default)
Restart=always          # Always restart regardless of exit code
Restart=on-failure      # Only restart on non-zero exit code
Restart=on-abnormal     # Restart on signal or timeout
```

### View all services

```bash
systemctl list-units --type=service --state=running
systemctl list-units --type=service --state=failed
systemctl list-unit-files --type=service     # All services with enable/disable status
```

---

## 🔐 SSL/TLS & Certbot

```bash
# Install Certbot (Debian/Ubuntu)
sudo apt install certbot python3-certbot-nginx -y

# Issue SSL certificate (auto-configure Nginx)
sudo certbot --nginx -d example.com -d www.example.com

# Issue cert only, configure Nginx manually
sudo certbot certonly --nginx -d example.com

# Wildcard certificate (requires DNS challenge)
sudo certbot certonly --manual --preferred-challenges dns -d "*.example.com"

# Renew certificates
sudo certbot renew                            # Renew all certs near expiry
sudo certbot renew --dry-run                  # Test renewal without issuing
sudo certbot renew --cert-name example.com    # Renew specific domain

# List and delete certs
sudo certbot certificates
sudo certbot delete --cert-name example.com

# Check auto-renewal timer
sudo systemctl status certbot.timer

# Check SSL expiry from command line
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates
openssl x509 -enddate -noout -in /etc/letsencrypt/live/example.com/cert.pem
```

### Nginx SSL + Reverse Proxy config

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;     # Redirect HTTP → HTTPS
}

server {
    listen 443 ssl;
    server_name example.com www.example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 🌐 curl Advanced

```bash
# Basic GET
curl https://api.example.com/users
curl -s https://api.example.com              # Silent, no progress bar
curl -o output.html https://example.com      # Save to file

# POST with JSON
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "John", "email": "john@example.com"}'

# POST with form data
curl -X POST https://api.example.com/login \
  -d "username=admin&password=secret"

# View headers
curl -I https://example.com                  # HEAD request — headers only
curl -v https://example.com                  # Verbose — full request & response

# Custom headers
curl -H "Authorization: Bearer <token>" https://api.example.com/me
curl -H "X-API-Key: abc123" -H "Accept: application/json" https://api.example.com

# Upload file
curl -F "file=@/path/to/file.jpg" https://api.example.com/upload

# Authentication
curl -u username:password https://api.example.com

# Follow redirects and timeouts
curl -L https://short.url/abc
curl --connect-timeout 10 --max-time 30 https://api.example.com
curl --retry 3 --retry-delay 2 https://api.example.com

# Check HTTP status code
curl -o /dev/null -s -w "%{http_code}" https://example.com
curl -o /dev/null -s -w "Status: %{http_code} | Time: %{time_total}s\n" https://example.com

# PUT and DELETE
curl -X PUT https://api.example.com/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "Jane"}'

curl -X DELETE https://api.example.com/users/1 \
  -H "Authorization: Bearer <token>"

# Skip SSL verification (self-signed certs)
curl -k https://self-signed.example.com

# Use a proxy
curl -x http://proxy:8080 https://example.com

# Download multiple files
curl -O https://example.com/file1.zip \
     -O https://example.com/file2.zip
```

---

## ✂️ Sed & Awk

### Sed — Find & Replace in files/streams

```bash
# Replace text
sed 's/old/new/' file.txt                    # Replace first occurrence per line
sed 's/old/new/g' file.txt                   # Replace all occurrences (global)
sed 's/old/new/gi' file.txt                  # Case-insensitive
sed -i 's/old/new/g' file.txt                # Edit file in-place
sed -i.bak 's/old/new/g' file.txt            # Edit file, keep .bak backup

# Delete lines
sed '/pattern/d' file.txt                    # Delete lines matching pattern
sed '5d' file.txt                            # Delete line 5
sed '5,10d' file.txt                         # Delete lines 5 to 10
sed '/^$/d' file.txt                         # Delete empty lines

# Print specific lines
sed -n '5p' file.txt                         # Print line 5
sed -n '5,10p' file.txt                      # Print lines 5 to 10
sed -n '/pattern/p' file.txt                 # Print lines matching pattern

# Insert lines
sed '3a\\Add this line after line 3' file.txt
sed '3i\\Add this line before line 3' file.txt

# Real-world examples
sed -i 's/localhost/db.example.com/g' config.yml
sed -i 's/DEBUG=true/DEBUG=false/g' .env
```

### Awk — Column-based text processing

```bash
# Print specific columns ($1=col1, $NF=last col)
awk '{print $1}' file.txt
awk '{print $1, $3}' file.txt
ps aux | awk '{print $1, $2, $11}'           # User, PID, Command

# Custom delimiter (-F)
awk -F: '{print $1}' /etc/passwd             # Print usernames
awk -F, '{print $2}' data.csv                # Column 2 of CSV

# Filter by condition
awk '$3 > 100 {print}' file.txt
awk '/error/ {print}' app.log
awk 'NR>=5 && NR<=10' file.txt               # Print lines 5 to 10

# Calculate
awk '{sum += $1} END {print "Total:", sum}' numbers.txt
df -h | awk 'NR>1 {print $5, $6}'           # Disk usage

# Top 10 IPs from Nginx access log
cat /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# Top 5 processes by CPU
ps aux | awk 'NR>1 {print $3, $4, $11}' | sort -rn | head -5
```

---

## 💾 Disk Management

```bash
# Disk overview
df -h                                        # Partition usage
lsblk                                        # Block device tree
lsblk -f                                     # Include filesystem & UUID
sudo fdisk -l                                # All disk details
blkid                                        # UUID & filesystem type

# Directory size
du -sh /var/log                              # Total size of directory
du -sh /var/log/*                            # Size of each item inside
du -sh /* 2>/dev/null | sort -rh | head -10  # Top 10 largest directories
du -ah /home | sort -rh | head -20           # Top 20 largest files/folders

# Find large files
find / -type f -size +500M 2>/dev/null
find /var -type f -size +100M -exec ls -lh {} \;

# Free up disk space
sudo apt autoremove -y && sudo apt clean     # Remove unused packages and apt cache
sudo journalctl --vacuum-size=200M           # Limit journald log size
docker system prune -a                       # Remove Docker unused resources
find /var/log -name "*.gz" -mtime +7 -delete # Remove old compressed logs older than 7 days
truncate -s 0 /var/log/syslog                # Clear log file content

# Format partition (be careful!)
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.xfs /dev/sdb1

# Mount / Unmount
sudo mount /dev/sdb1 /mnt/data
sudo umount /mnt/data
sudo mount -a                                # Mount all entries in /etc/fstab

# Auto-mount on boot — add to /etc/fstab
blkid /dev/sdb1                              # Get UUID
echo "UUID=xxxx /mnt/data ext4 defaults 0 2" | sudo tee -a /etc/fstab

# Check filesystem (unmount first)
sudo fsck -y /dev/sdb1                       # Check filesystem (unmount first)

# Swap
swapon --show                                # View current swap
sudo fallocate -l 2G /swapfile               # Create 2GB swap file
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab  # Auto-mount on boot
```

---

## 📜 Bash Scripting

### Structure & variables

```bash
#!/bin/bash
set -e          # Exit immediately on error
set -u          # Error on undefined variables
set -o pipefail # Exit if any pipe command fails

# Variables
NAME="John"
TODAY=$(date +%F)                            # Assign command output to variable
FILES=$(ls /var/www)

echo "Hello $NAME, today is ${TODAY}"

# Script arguments: ./deploy.sh production v1.2.0
ENV=$1
VERSION=$2
echo "Deploying $VERSION to $ENV"
echo "All args: $@"
echo "Arg count: $#"

# Read user input
read -p "Enter your name: " USERNAME
read -sp "Enter password: " PASSWORD          # -s: hide input
```

### Conditionals

```bash
# If/else
if [ "$ENV" == "production" ]; then
  echo "Deploy production!"
elif [ "$ENV" == "staging" ]; then
  echo "Deploy staging"
else
  echo "Invalid environment"
  exit 1
fi

# Numeric comparison: -gt > | -lt < | -ge >= | -le <= | -eq == | -ne !=: -gt > | -lt < | -ge >= | -le <= | -eq == | -ne !=
if [ $COUNT -gt 10 ]; then echo "Greater than 10"; fi

# File / variable checks
[ -f "/etc/nginx/nginx.conf" ] && echo "File exists"
[ -d "/var/www" ]              && echo "Directory exists"
[ -z "$VAR" ]                  && echo "Variable is empty"
[ -n "$VAR" ]                  && echo "Variable has value"

# Check command success
if systemctl is-active --quiet nginx; then
  echo "Nginx is running"
fi
```

### Loops

```bash
# For loop
for i in 1 2 3 4 5; do echo "Number $i"; done

for file in /var/log/*.log; do
  echo "Processing: $file"
done

for i in $(seq 1 10); do echo "Loop $i"; done

# While loop
COUNT=0
while [ $COUNT -lt 5 ]; do
  echo "Count: $COUNT"
  ((COUNT++))
done

# Read file line by line
while IFS= read -r line; do
  echo "Line: $line"
done < /etc/hosts
```

### Functions & error handling

```bash
# Define function
check_service() {
  local SERVICE=$1                           # Local variable trong function
  if systemctl is-active --quiet "$SERVICE"; then
    echo "OK $SERVICE dang chay"
    return 0
  else
    echo "LOI $SERVICE khong chay"
    return 1
  fi
}

check_service nginx
check_service mysql

# Trap — run cleanup on exit or interrupt
cleanup() {
  echo "Cleaning up before exit..."
  rm -f /tmp/tmpfile
}
trap cleanup EXIT                            # Run cleanup when script exits
trap cleanup INT TERM                        # Run cleanup on Ctrl+C or kill signal
```

### Sample deploy script

```bash
#!/bin/bash
set -e

APP_DIR="/home/ubuntu/myapp"
SERVICE="myapp"
BRANCH=${1:-main}                            # Default to main if no argument passed

log() { echo "[$(date '+%H:%M:%S')] $1"; }

log "Bat dau deploy branch: $BRANCH"

cd $APP_DIR
git pull origin $BRANCH

if [ -f "requirements.txt" ]; then
  log "Cai dat Python dependencies..."
  source venv/bin/activate
  pip install -r requirements.txt -q
  deactivate
fi

log "Restarting service..."
sudo systemctl restart $SERVICE

sleep 2
if systemctl is-active --quiet $SERVICE; then
  log "Deploy successful!"
else
  log "ERROR: Service failed to start!"
  journalctl -u $SERVICE -n 20
  exit 1
fi
```

---

## 🔥 iptables

```bash
# View current rules
sudo iptables -L -v -n                       # Detailed, no hostname resolution
sudo iptables -L --line-numbers              # Include line numbers

# Chains: INPUT (incoming), OUTPUT (outgoing), FORWARD (routed traffic)

# Allow (ACCEPT)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # SSH
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT    # HTTP
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT   # HTTPS

# Block (DROP)
sudo iptables -A INPUT -p tcp --dport 3306 -j DROP    # Block MySQL from outside
sudo iptables -A INPUT -s 1.2.3.4 -j DROP             # Block specific IP

# Allow from specific IP/subnet
sudo iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT
sudo iptables -A INPUT -s 10.0.0.5 -p tcp --dport 3306 -j ACCEPT

# Essential: allow established connections and loopback
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT

# Set default policy (block everything except allowed)
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Delete rules
sudo iptables -D INPUT -p tcp --dport 80 -j ACCEPT    # Delete by content
sudo iptables -D INPUT 3                              # Delete by line number
sudo iptables -F                                      # Flush all rules

# Persist rules across reboots
sudo apt install iptables-persistent -y
sudo netfilter-persistent save

# NAT / Port forwarding
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
sudo iptables -t nat -L                      # View NAT rules
```

---

## 🔍 nmap

```bash
# Install
sudo apt install nmap -y

# Basic scan
nmap 192.168.1.1                             # Scan single host
nmap 192.168.1.0/24                          # Scan entire subnet
nmap 192.168.1.1-50                          # Scan IP range

# Port scanning
nmap -p 80,443,3306 192.168.1.1              # Specific ports
nmap -p 1-1000 192.168.1.1                   # Scan range
nmap -p- 192.168.1.1                         # All 65535 ports

# Scan types
nmap -sT 192.168.1.1                         # TCP Connect (no root required)
nmap -sS 192.168.1.1                         # SYN / stealth scan (root required)
nmap -sU 192.168.1.1                         # UDP scan
nmap -sn 192.168.1.0/24                      # Ping scan — find live hosts only

# OS and service detection
nmap -sV 192.168.1.1                         # Service version detection
nmap -O  192.168.1.1                         # OS detection (root required)
nmap -A  192.168.1.1                         # All: OS + version + traceroute

# Scan speed (T1=slowest, T5=fastest)
nmap -T3 192.168.1.0/24                      # Default, balanced
nmap -T5 192.168.1.1                         # Fastest

# Save output
nmap -oN output.txt 192.168.1.0/24           # Plain text
nmap -oX output.xml 192.168.1.0/24           # XML format

# Practical examples
# Check common ports on a server
nmap -sV -p 22,80,443,3306,5432,6379,27017 192.168.1.1

# Find all live hosts in the network
nmap -sn 192.168.1.0/24 | grep "Nmap scan report"

# Quick scan — top 100 ports
sudo nmap -sS --top-ports 100 192.168.1.1
```

---

## 📝 Tips & Tricks

```bash
# Run in background — survives terminal close
nohup ./script.sh &
screen -S mysession
tmux new -s mysession

# Command history
history
history | grep docker

# Time how long a command takes
time ./script.sh

# Redirect output
command > output.txt                           # Overwrite
command >> output.txt                          # Append
command 2>&1 | tee output.txt                  # Capture both stdout & stderr

# Environment variables
export MY_VAR="value"
echo $MY_VAR
env | grep MY_VAR
```

---

## 🔖 Alias & Function trong `.bashrc`

Tạo shortcut lệnh để tiết kiệm thời gian gõ.

### Alias — shortcut đơn giản

```bash
# Mở file .bashrc
nano ~/.bashrc

# Thêm alias vào cuối file, ví dụ:
alias ll="ls -lah"
alias gs="git status"
alias mm="cd /var/www/myapp"
alias push="git add --all; git commit -m 'X'; git push"

# Áp dụng ngay mà không cần mở terminal mới
source ~/.bashrc
```

### Function — linh hoạt hơn alias, nhận tham số

```bash
# Thêm vào ~/.bashrc
function push() {
    git add --all
    if [ -z "$1" ]; then
        git commit -m "X"          # Dùng message mặc định nếu không truyền tham số
    else
        git commit -m "$1"
    fi
    git push
}

function mkcd() {
    mkdir -p "$1" && cd "$1"       # Tạo thư mục rồi vào luôn
}
```

```bash
# Sử dụng
push                               # Commit với message "X"
push "fix login bug"               # Commit với message tùy chỉnh
mkcd /var/www/newproject
```

> 💡 Alias phù hợp cho lệnh ngắn cố định. Dùng **function** khi cần nhận tham số hoặc có logic điều kiện.

---

## 🟩 NVM & Node.js

### Cài đặt NVM

```bash
# Cài NVM (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# Áp dụng ngay mà không cần mở terminal mới
source ~/.bashrc
# Hoặc: source ~/.zshrc  (nếu dùng zsh)

# Kiểm tra cài đặt
nvm --version
```

### Quản lý phiên bản Node

```bash
# Cài Node.js
nvm install --lts                  # Cài bản LTS mới nhất (khuyến nghị)
nvm install 20                     # Cài Node 20.x
nvm install 18.20.0                # Cài phiên bản cụ thể

# Chuyển đổi phiên bản
nvm use --lts
nvm use 20
nvm use 18.20.0

# Đặt phiên bản mặc định
nvm alias default 20               # Dùng Node 20 mặc định khi mở terminal mới
nvm alias default --lts

# Xem các phiên bản đã cài
nvm ls
nvm ls-remote --lts                # Xem các bản LTS có thể cài

# Xóa một phiên bản
nvm uninstall 18.20.0
```

### npm cơ bản

```bash
node -v                            # Kiểm tra phiên bản Node
npm -v                             # Kiểm tra phiên bản npm

npm install                        # Cài dependencies từ package.json
npm install <package>              # Cài package vào dependencies
npm install -D <package>           # Cài vào devDependencies
npm install -g <package>           # Cài global (dùng được ở mọi nơi)
npm uninstall <package>
npm update

npm run dev
npm run build
npm run start

# Xem các package global đã cài
npm list -g --depth=0
```

---

*Last updated: 2026 · For DevOps Engineers*
