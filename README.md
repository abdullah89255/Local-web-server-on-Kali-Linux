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

