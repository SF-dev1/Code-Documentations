-- -

## 🗺️ ROADMAP: Nginx + PHP 8.3-FPM Setup for FIMS + Support Project

-- -

## 🧱 1. **Server Requirements**
- OS: Ubuntu 22.04 (or similar)
- RAM: 8 GB
- PHP: 8.3 with FPM
- Web server: Nginx
- DNS: Domains already pointed to your server IP

---

## 📁 2. **Folder Structure**
```
/var/www/fims-project         ← FIMS Project (shared root)
/var/www/support-project      ← Support Project
```

---

## 📦 3. **Install Required Packages**

```bash
sudo apt update
sudo apt install nginx php8.3 php8.3-fpm php8.3-cli php8.3-mysql php8.3-curl php8.3-mbstring php8.3-xml php8.3-zip unzip curl -y
```

> ✅ Also install Redis, Composer, MySQL if needed

---

## 🔧 4. **Create PHP-FPM Pools (3 Pools Total)**

**Path:** `/etc/php/8.3/fpm/pool.d/`

---

### 🔹 a. `fims-api.conf` → For `fims.skmienterprise.com`
```ini
[fims_api]
user = www-data
group = www-data
listen = /run/php/php8.3-fpm-fims-api.sock
listen.owner = www-data
listen.group = www-data

pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 3
pm.max_spare_servers = 10

chdir = /var/www/fims-project
php_admin_value[memory_limit] = 512M
```

---

### 🔹 b. `fims-webhook.conf` → For `www.skmienterprise.com/fims`
```ini
[fims_webhook]
user = www-data
group = www-data
listen = /run/php/php8.3-fpm-fims-webhook.sock
listen.owner = www-data
listen.group = www-data

pm = dynamic
pm.max_children = 20
pm.start_servers = 3
pm.min_spare_servers = 2
pm.max_spare_servers = 6

chdir = /var/www/fims-project
php_admin_value[memory_limit] = 256M
```

---

### 🔹 c. `support.conf` → For `support.sylvi.in`
```ini
[support]
user = www-data
group = www-data
listen = /run/php/php8.3-fpm-support.sock
listen.owner = www-data
listen.group = www-data

pm = dynamic
pm.max_children = 20
pm.start_servers = 3
pm.min_spare_servers = 2
pm.max_spare_servers = 6

chdir = /var/www/support-project
php_admin_value[memory_limit] = 256M
```

---

### ➕ Enable and Restart FPM

```bash
sudo systemctl restart php8.3-fpm
```

---

## 🌐 5. **Nginx Virtual Host Configurations**

**Path:** `/etc/nginx/sites-available/`

---

### 🔸 a. FIMS User Access (`fims.skmienterprise.com`)

**`fims.skmienterprise.com`**
```nginx
server {
    listen 80;
    server_name fims.skmienterprise.com;
    root /var/www/fims-project;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.3-fpm-fims-api.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    access_log /var/log/nginx/fims-api-access.log;
    error_log /var/log/nginx/fims-api-error.log;
}
```

---

### 🔸 b. FIMS Webhook (`www.skmienterprise.com/fims`)

**`www.skmienterprise.com`**
```nginx
server {
    listen 80;
    server_name www.skmienterprise.com;
    root /var/www/html;

    location /fims/ {
        root /var/www/fims-project;
        index index.php index.html;
        try_files $uri $uri/ /index.php?$query_string;

        location ~ \.php$ {
            include fastcgi_params;
            fastcgi_pass unix:/run/php/php8.3-fpm-fims-webhook.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
    }

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/webhook-access.log;
    error_log /var/log/nginx/webhook-error.log;
}
```

---

### 🔸 c. Support Project (`support.sylvi.in`)

**`support.sylvi.in`**
```nginx
server {
    listen 80;
    server_name support.sylvi.in;
    root /var/www/support-project;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.3-fpm-support.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    access_log /var/log/nginx/support-access.log;
    error_log /var/log/nginx/support-error.log;
}
```

---

## 🔗 6. **Enable Nginx Sites**

```bash
sudo ln -s /etc/nginx/sites-available/fims.skmienterprise.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/www.skmienterprise.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/support.sylvi.in /etc/nginx/sites-enabled/
```

---

## ✅ 7. **Final Commands to Apply Config**

```bash
sudo nginx -t             # test config
sudo systemctl reload nginx
sudo systemctl restart php8.3-fpm
```

---

## 🔍 8. **(Optional) Add SSL with Let's Encrypt**

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d fims.skmienterprise.com -d www.skmienterprise.com -d support.sylvi.in
```

---

## 📊 9. **Monitor PHP-FPM Pools (optional)**

Add this in each FPM pool:
```ini
pm.status_path = /status
```

Then allow `/status` in your Nginx config:
```nginx
location ~ ^/status$ {
    allow 127.0.0.1;
    deny all;
    include fastcgi_params;
    fastcgi_pass unix:/run/php/php8.3-fpm-fims-api.sock; # respective sock
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

Access: `curl http://localhost/status`

---

## 🔚 Summary of What You Have:
| Domain | Pool | Project Root | Purpose |
|--------|------|---------------|---------|
| `fims.skmienterprise.com` | `fims_api` | `/var/www/fims-project` | UI |
| `www.skmienterprise.com/fims` | `fims_webhook` | `/var/www/fims-project` | Webhooks |
| `support.sylvi.in` | `support` | `/var/www/support-project` | Support Panel |

---

Would you like me to generate the full ready-to-copy config files as `.conf` for FPM and Nginx? Perfect! You're aiming for a high-performance, scalable setup using **Nginx** and **PHP 8.3-FPM** with **three virtual hosts** and **three separate PHP-FPM pools**, split across **two project roots**.

Here’s your **step-by-step deployment roadmap**, fully explained.

---

## 🗺️ ROADMAP: Nginx + PHP 8.3-FPM Setup for FIMS + Support Project

---

## 🧱 1. **Server Requirements**
- OS: Ubuntu 22.04 (or similar)
- RAM: 8 GB
- PHP: 8.3 with FPM
- Web server: Nginx
- DNS: Domains already pointed to your server IP

---

## 📁 2. **Folder Structure**
```
/var/www/fims-project         ← FIMS Project (shared root)
/var/www/support-project      ← Support Project
```

---

## 📦 3. **Install Required Packages**

```bash
sudo apt update
sudo apt install nginx php8.3 php8.3-fpm php8.3-cli php8.3-mysql php8.3-curl php8.3-mbstring php8.3-xml php8.3-zip unzip curl -y
```

> ✅ Also install Redis, Composer, MySQL if needed

---

## 🔧 4. **Create PHP-FPM Pools (3 Pools Total)**

**Path:** `/etc/php/8.3/fpm/pool.d/`

---

### 🔹 a. `fims-api.conf` → For `fims.skmienterprise.com`
```ini
[fims_api]
user = www-data
group = www-data
listen = /run/php/php8.3-fpm-fims-api.sock
listen.owner = www-data
listen.group = www-data

pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 3
pm.max_spare_servers = 10

chdir = /var/www/fims-project
php_admin_value[memory_limit] = 512M
```

---

### 🔹 b. `fims-webhook.conf` → For `www.skmienterprise.com/fims`
```ini
[fims_webhook]
user = www-data
group = www-data
listen = /run/php/php8.3-fpm-fims-webhook.sock
listen.owner = www-data
listen.group = www-data

pm = dynamic
pm.max_children = 20
pm.start_servers = 3
pm.min_spare_servers = 2
pm.max_spare_servers = 6

chdir = /var/www/fims-project
php_admin_value[memory_limit] = 256M
```

---

### 🔹 c. `support.conf` → For `support.sylvi.in`
```ini
[support]
user = www-data
group = www-data
listen = /run/php/php8.3-fpm-support.sock
listen.owner = www-data
listen.group = www-data

pm = dynamic
pm.max_children = 20
pm.start_servers = 3
pm.min_spare_servers = 2
pm.max_spare_servers = 6

chdir = /var/www/support-project
php_admin_value[memory_limit] = 256M
```

---

### ➕ Enable and Restart FPM

```bash
sudo systemctl restart php8.3-fpm
```

---

## 🌐 5. **Nginx Virtual Host Configurations**

**Path:** `/etc/nginx/sites-available/`

---

### 🔸 a. FIMS User Access (`fims.skmienterprise.com`)

**`fims.skmienterprise.com`**
```nginx
server {
    listen 80;
    server_name fims.skmienterprise.com;
    root /var/www/fims-project;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.3-fpm-fims-api.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    access_log /var/log/nginx/fims-api-access.log;
    error_log /var/log/nginx/fims-api-error.log;
}
```

---

### 🔸 b. FIMS Webhook (`www.skmienterprise.com/fims`)

**`www.skmienterprise.com`**
```nginx
server {
    listen 80;
    server_name www.skmienterprise.com;
    root /var/www/html;

    location /fims/ {
        root /var/www/fims-project;
        index index.php index.html;
        try_files $uri $uri/ /index.php?$query_string;

        location ~ \.php$ {
            include fastcgi_params;
            fastcgi_pass unix:/run/php/php8.3-fpm-fims-webhook.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
    }

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/webhook-access.log;
    error_log /var/log/nginx/webhook-error.log;
}
```

---

### 🔸 c. Support Project (`support.sylvi.in`)

**`support.sylvi.in`**
```nginx
server {
    listen 80;
    server_name support.sylvi.in;
    root /var/www/support-project;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.3-fpm-support.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    access_log /var/log/nginx/support-access.log;
    error_log /var/log/nginx/support-error.log;
}
```

---

## 🔗 6. **Enable Nginx Sites**

```bash
sudo ln -s /etc/nginx/sites-available/fims.skmienterprise.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/www.skmienterprise.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/support.sylvi.in /etc/nginx/sites-enabled/
```

---

## ✅ 7. **Final Commands to Apply Config**

```bash
sudo nginx -t             # test config
sudo systemctl reload nginx
sudo systemctl restart php8.3-fpm
```

---

## 🔍 8. **(Optional) Add SSL with Let's Encrypt**

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d fims.skmienterprise.com -d www.skmienterprise.com -d support.sylvi.in
```

---

## 📊 9. **Monitor PHP-FPM Pools (optional)**

Add this in each FPM pool:
```ini
pm.status_path = /status
```

Then allow `/status` in your Nginx config:
```nginx
location ~ ^/status$ {
    allow 127.0.0.1;
    deny all;
    include fastcgi_params;
    fastcgi_pass unix:/run/php/php8.3-fpm-fims-api.sock; # respective sock
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

Access: `curl http://localhost/status`

---

## 🔚 Summary of What You Have:
| Domain | Pool | Project Root | Purpose |
|--------|------|---------------|---------|
| `fims.skmienterprise.com` | `fims_api` | `/var/www/fims-project` | UI |
| `www.skmienterprise.com/fims` | `fims_webhook` | `/var/www/fims-project` | Webhooks |
| `support.sylvi.in` | `support` | `/var/www/support-project` | Support Panel |

---

Would you like me to generate the full ready-to-copy config files as `.conf` for FPM and Nginx? 