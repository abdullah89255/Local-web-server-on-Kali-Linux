# Local-web-server-on-Kali-Linux


# 1) Quick (serve a folder) — Python 3

Fast for static files. Serves current folder on port 8000:

```bash
cd /path/to/site-folder
python3 -m http.server 8000
# open: http://localhost:8000 or http://127.0.0.1:8000
```

If you want it on port 80 (needs sudo):

```bash
sudo python3 -m http.server 80
# open: http://localhost
```

---

# 2) Standard (full web stack) — Apache2

Good if you need PHP, virtual hosts, etc.

Install Apache:

```bash
sudo apt update
sudo apt install apache2
```

Start & enable at boot:

```bash
sudo systemctl start apache2
sudo systemctl enable apache2
```

Check status:

```bash
sudo systemctl status apache2
```

Default web root: `/var/www/html/`
Place your `index.html` or `index.php` there:

```bash
sudo cp -r /path/to/your/site/* /var/www/html/
# give appropriate ownership so you can edit files:
sudo chown -R $USER:www-data /var/www/html
```

Open in browser: `http://localhost` or `http://127.0.0.1`

If Apache fails to start, inspect logs:

```bash
sudo journalctl -xeu apache2.service
sudo tail -n 100 /var/log/apache2/error.log
```

---

# 3) PHP built-in server (dev only)

If you need to run PHP quickly:

```bash
cd /path/to/php-project
php -S 0.0.0.0:8000
# open http://localhost:8000
```

---

# 4) Node.js static server

If you use node/npm:

```bash
# install http-server once globally (or use npx)
sudo npm install -g http-server
cd /path/to/site-folder
http-server -p 8000
# open http://localhost:8000
```

Or with npx (no global install):

```bash
npx http-server -p 8000
```

---

# 5) If you want a full LAMP (Linux+Apache+MySQL+PHP)

```bash
sudo apt install apache2 mariadb-server php libapache2-mod-php php-mysql
sudo systemctl restart apache2
```

Then configure MariaDB (`sudo mysql_secure_installation`) if you need databases.

---

# Troubleshooting tips

* Are ports listening?

```bash
ss -tlnp | grep ':80\|:8000'
```

* If `localhost` doesn’t resolve, use `http://127.0.0.1`.
* Permission issues: use `sudo chown -R $USER:www-data /var/www/html`.
* If firewall blocks (rare on Kali), check `ufw status` or `iptables -L`.
* Apache config test:

```bash
sudo apache2ctl configtest
```

---

## 🧩 Step 1 — Install all required packages

Run this in your terminal:

```bash
sudo apt update
sudo apt install apache2 mariadb-server php libapache2-mod-php php-mysql php-xml php-gd php-curl php-zip php-mbstring unzip wget -y
```

This installs:

* **Apache2** → web server
* **MariaDB** → database
* **PHP + common modules** → to run WordPress/PHP apps

---

## ⚙️ Step 2 — Start and enable services

```bash
sudo systemctl start apache2
sudo systemctl start mariadb
sudo systemctl enable apache2
sudo systemctl enable mariadb
```

Check Apache status:

```bash
sudo systemctl status apache2
```

✅ If “active (running)” → you’re good.

Now open in browser:
👉 [http://localhost](http://localhost)
You should see **“Apache2 Default Page”**.

---

## 🔐 Step 3 — Secure MariaDB

Set up a password and remove insecure defaults:

```bash
sudo mysql_secure_installation
```

Follow prompts:

* Set root password → **Yes**
* Remove anonymous users → **Yes**
* Disallow root login remotely → **Yes**
* Remove test database → **Yes**
* Reload privileges → **Yes**

---

## 🗄️ Step 4 — Create a database for WordPress

Enter MariaDB shell:

```bash
sudo mysql -u root -p
```

Then run these SQL commands (replace `wpuser` and `password` as you like):

```sql
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 🌐 Step 5 — Download and install WordPress

```bash
cd /tmp
wget https://wordpress.org/latest.zip
unzip latest.zip
sudo mv wordpress /var/www/html/
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress
```

---

## 🧾 Step 6 — Configure Apache for WordPress

Create a new site config:

```bash
sudo nano /etc/apache2/sites-available/wordpress.conf
```

Paste this:

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/wordpress
    ServerName localhost

    <Directory /var/www/html/wordpress/>
        AllowOverride All
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/wordpress_error.log
    CustomLog ${APACHE_LOG_DIR}/wordpress_access.log combined
</VirtualHost>
```

Save (Ctrl+O, Enter, Ctrl+X).

Enable the site and rewrite module:

```bash
sudo a2ensite wordpress.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## 🛠️ Step 7 — WordPress setup via browser

Open:
👉 **[http://localhost/wordpress](http://localhost/wordpress)**

You’ll see the WordPress setup page.

Fill in:

* Database Name: `wordpress`
* Username: `wpuser`
* Password: `password`
* Database Host: `localhost`
* Table Prefix: `wp_`

Click **Submit → Run the installation → Create admin user.**

Done! 🎉
You now have **WordPress running locally on Kali**.

---

## 🧩 Step 8 (optional) — Run any other PHP app

Place any PHP app in `/var/www/html/myapp`, then:

```bash
sudo chown -R www-data:www-data /var/www/html/myapp
sudo chmod -R 755 /var/www/html/myapp
```

Access via:
👉 [http://localhost/myapp](http://localhost/myapp)

---

Apache can’t start because the log directory **`/var/log/apache2/`** is missing.

Let’s fix it step-by-step 👇

---

### 🧰 Step 1 — Create the missing log directory

Run:

```bash
sudo mkdir -p /var/log/apache2
sudo chown -R www-data:adm /var/log/apache2
sudo chmod 750 /var/log/apache2
```

---

### 🧱 Step 2 — Verify Apache config

Just to ensure no syntax issue:

```bash
sudo apache2ctl configtest
```

If you see:

```
Syntax OK
```

✅ it’s good.

---

### 🧩 Step 3 — Restart Apache

```bash
sudo systemctl restart apache2
```

Now check again:

```bash
sudo systemctl status apache2
```

If it says `active (running)`, you’re done!
Open in browser → **[http://localhost](http://localhost)**

---

### 🧰 Step 4 (Optional but recommended)

Fix the “Could not reliably determine the server's fully qualified domain name” warning by adding this line:

```bash
echo "ServerName localhost" | sudo tee /etc/apache2/conf-available/fqdn.conf
sudo a2enconf fqdn
sudo systemctl reload apache2
```

---

### ✅ Verify it’s working

```bash
curl http://localhost
```

You should see the default Apache HTML page contents.

---

If it **still fails** after this, run:

```bash
sudo journalctl -xeu apache2.service | tail -n 20
```

