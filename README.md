# odoo14-install-on-ubuntu-Server
# Guide to install Odoo 14 on Ubuntu Desktop

# Odoo 14 Installation on Ubuntu 20.04 LTS

Step 1: Update System
sudo apt update && sudo apt upgrade -y

Step 2: Secure Server
sudo apt install openssh-server fail2ban -y

Step 3: Create Odoo System User
sudo adduser --system --home=/opt/odoo --group odoo

Step 4: Install Dependencies
sudo apt install -y python3-pip python3-dev python-dev \
libxml2-dev libxslt1-dev zlib1g-dev libsasl2-dev \
libldap2-dev build-essential libssl-dev libffi-dev \
libmysqlclient-dev libjpeg-dev libpq-dev libjpeg8-dev \
liblcms2-dev libblas-dev libatlas-base-dev npm node-less git

sudo ln -s /usr/bin/nodejs /usr/bin/node
sudo npm install -g less less-plugin-clean-css

Step 5: PostgreSQL Setup
sudo apt install postgresql -y
sudo su - postgres
createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt odoo14
psql
ALTER USER odoo14 WITH SUPERUSER;
\q
exit

Step 6: Get Odoo 14 Source
sudo su - odoo -s /bin/bash
git clone https://www.github.com/odoo/odoo --depth 1 --branch 14.0 --single-branch .
exit

Step 7: Python Dependencies + wkhtmltopdf
sudo pip3 install -r /opt/odoo/requirements.txt

# Install wkhtmltopdf 0.12.5
sudo wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.bionic_amd64.deb
sudo dpkg -i wkhtmltox_0.12.5-1.bionic_amd64.deb
sudo apt install -f
Step 8: Configure Odoo
sudo cp /opt/odoo/debian/odoo.conf /etc/odoo.conf
sudo nano /etc/odoo.conf
Sample config:
[options]
admin_passwd = admin
db_host = False
db_port = False
db_user = odoo14
db_password = False
addons_path = /opt/odoo/addons
logfile = /var/log/odoo/odoo.log
Set permissions:
sudo chown odoo: /etc/odoo.conf
sudo chmod 640 /etc/odoo.conf

sudo mkdir /var/log/odoo
sudo chown odoo:root /var/log/odoo

Step 9: Create Odoo Service
sudo nano /etc/systemd/system/odoo.service
Paste this:
[Unit]
Description=Odoo
Documentation=http://www.odoo.com

[Service]
Type=simple
User=odoo
ExecStart=/opt/odoo/odoo-bin -c /etc/odoo.conf

[Install]
WantedBy=default.target
sudo chmod 755 /etc/systemd/system/odoo.service
sudo chown root: /etc/systemd/system/odoo.service
Step 10: Start and Access Odoo
sudo systemctl start odoo.service
sudo systemctl status odoo.service

# Enable on boot
sudo systemctl enable odoo.service
Access via browser
http://<your-server-ip>:8069
________________________________________
