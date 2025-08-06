# odoo14-install-on-ubuntu-Server

# Odoo 14 Installation on Ubuntu 20.04 LTS

### **Step 1: Update System**


sudo apt update && sudo apt upgrade -y


### **Step 2: Secure Server (Optional but Recommended)**

sudo apt install openssh-server fail2ban -y


### **Step 3: Create Odoo System User**


sudo adduser --system --home=/opt/odoo --group odoo

### **Step 4: Install Required Dependencies**


sudo apt install -y python3-pip python3-dev python-dev \
libxml2-dev libxslt1-dev zlib1g-dev libsasl2-dev \
libldap2-dev build-essential libssl-dev libffi-dev \
libmysqlclient-dev libjpeg-dev libpq-dev libjpeg8-dev \
liblcms2-dev libblas-dev libatlas-base-dev npm node-less git

---

sudo ln -s /usr/bin/nodejs /usr/bin/node
sudo npm install -g less less-plugin-clean-css


### **Step 5: PostgreSQL Setup**


sudo apt install postgresql -y
sudo su - postgres
createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo14
psql


```

Inside PostgreSQL:

ALTER USER odoo14 WITH SUPERUSER;
\q
exit


---

### **Step 6: Get Odoo 14 Source Code**


sudo su - odoo -s /bin/bash
git clone https://www.github.com/odoo/odoo --depth 1 --branch 14.0 --single-branch .
exit


### **Step 7: Install Python Requirements + wkhtmltopdf**

sudo pip3 install -r /opt/odoo/requirements.txt


Install `wkhtmltopdf` (required for PDF reports):


sudo wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.bionic_amd64.deb
sudo dpkg -i wkhtmltox_0.12.5-1.bionic_amd64.deb
sudo apt install -f -y


---

### **Step 8: Configure Odoo**


sudo cp /opt/odoo/debian/odoo.conf /etc/odoo.conf
sudo nano /etc/odoo.conf


Sample `odoo.conf`:


[options]
admin_passwd = 12345
db_host = False
db_port = False
db_user = odoo14
db_password = 123456789
addons_path = /opt/odoo/addons
logfile = /var/log/odoo/odoo.log

; Optional performance settings
limit_time_real = 3600
limit_time_cpu = 3600
limit_request = 8192
max_cron_threads = 2
workers = 2
```

Set config file permissions:


sudo chown odoo: /etc/odoo.conf
sudo chmod 640 /etc/odoo.conf


Log directory setup:


sudo mkdir /var/log/odoo
sudo chown odoo:root /var/log/odoo


### **Step 9: Create Odoo Service**


sudo nano /etc/systemd/system/odoo.service


Paste the following:


[Unit]
Description=Odoo
Documentation=http://www.odoo.com

[Service]
Type=simple
User=odoo
ExecStart=/opt/odoo/odoo-bin -c /etc/odoo.conf

[Install]
WantedBy=default.target


Set permissions:


sudo chmod 755 /etc/systemd/system/odoo.service
sudo chown root: /etc/systemd/system/odoo.service


### **Step 10: Start and Enable Odoo**


sudo systemctl start odoo.service
sudo systemctl status odoo.service
sudo systemctl enable odoo.service


Then open in your browser:


http://<your-server-ip>:8069




## Additional Tips & Fixes

### PostgreSQL Access Fix (Optional for Trust)


sudo nano /etc/postgresql/12/main/pg_hba.conf


Change:


local   all             all                                     peer


To:

local   all             all                                     trust


Then restart PostgreSQL:


sudo systemctl restart postgresql

### Install Extra Python Libraries


sudo pip3 install Werkzeug geopy requests deep_translator simplejson


### Handling Permissions (for development/debugging only)


# Unlock folder temporarily for editing
sudo chown -R <your_user>:<your_user> /opt/odoo/custom_addons/fortek_odoo

# Restore back to Odoo user after editing
sudo chown -R odoo:odoo /opt/odoo/custom_addons/fortek_odoo


If your Linux user is `furqan1`:


sudo chown -R furqan1:furqan1 /opt/odoo/addons
sudo chown -R odoo:odoo /opt/odoo/addons


### Custom Addons Folder Setup

If you create a custom module folder:


sudo mkdir -p /opt/odoo/custom_addons/fortek_odoo
sudo chown -R odoo:odoo /opt/odoo/custom_addons/fortek_odoo

Then add this to `addons_path` in `/etc/odoo.conf`:


addons_path = /opt/odoo/addons, /opt/odoo/custom_addons/fortek_odoo


### Restart Odoo and Check Logs


sudo systemctl restart odoo
sudo tail -f /var/log/odoo/odoo.log


### Optional: Increase Disk Space (VirtualBox or Cloud VM)

* Resize disk externally first.
* Then inside the VM:


sudo apt install gparted
sudo gparted
