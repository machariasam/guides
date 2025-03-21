
## Prequisites

First ensure your system is up to date and install the required dependencies:

```bash
sudo apt-get update -y
sudo apt install -y apt-transport-https npm
```

## Install MongoDB

Add MongoDB repository and install MongoDB 8.0:
```bash
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list << EOF
deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse
EOF
```
```bash
sudo apt update -y
sudo apt install -y mongodb-org
sudo systemctl enable --now mongod
```
## Install GenieACS

```bash
sudo npm install -g genieacs
```
## Configure GenieACS
** Create a system user to run GenieACS Daemons:
```bash
sudo useradd --system --no-create-home --user-group genieacs
```
Create directory to save extensions and environment file:
```bash
sudo mkdir -p /opt/genieacs/ext
sudo chown genieacs:genieacs /opt/genieacs/ext
```
Create the environment configuration file:
```bash
sudo tee /opt/genieacs/genieacs.env > /dev/null << EOF
GENIEACS_CWMP_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-cwmp-access.log
GENIEACS_NBI_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-nbi-access.log
GENIEACS_FS_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-fs-access.log
GENIEACS_UI_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-ui-access.log
GENIEACS_DEBUG_FILE=/var/log/genieacs/genieacs-debug.yaml
NODE_OPTIONS=--enable-source-maps
GENIEACS_EXT_DIR=/opt/genieacs/ext
EOF
```
Generate a secure JWT secret and add it to the environment configuration file:
```bash
sudo node -e "console.log(\"GENIEACS_UI_JWT_SECRET=\" + require('crypto').randomBytes(128).toString('hex'))" | sudo tee -a /opt/genieacs/genieacs.env > /dev/null
```
Set permissions:
```bash
sudo chown genieacs:genieacs /opt/genieacs -R
sudo chmod 600 /opt/genieacs/genieacs.env
```
## Create Systemd Service Files
Create service files for different GenieACS components:
## CWMP Service
```bash
sudo tee /etc/systemd/system/genieacs-cwmp.service > /dev/null << EOF
[Unit]
Description=GenieACS CWMP
After=network.target

[Service]
User=genieacs
EnvironmentFile=/opt/genieacs/genieacs.env
ExecStart=/usr/local/bin/genieacs-cwmp

[Install]
WantedBy=default.target
EOF
```
## NBI Service
```bash
sudo tee /etc/systemd/system/genieacs-nbi.service > /dev/null << EOF
[Unit]
Description=GenieACS NBI
After=network.target

[Service]
User=genieacs
EnvironmentFile=/opt/genieacs/genieacs.env
ExecStart=/usr/local/bin/genieacs-nbi

[Install]
WantedBy=default.target
EOF
```
## FS Service
```bash
sudo tee /etc/systemd/system/genieacs-fs.service > /dev/null << EOF
[Unit]
Description=GenieACS FS
After=network.target

[Service]
User=genieacs
EnvironmentFile=/opt/genieacs/genieacs.env
ExecStart=/usr/local/bin/genieacs-fs

[Install]
WantedBy=default.target
EOF
```
## UI Service
```bash
sudo tee /etc/systemd/system/genieacs-ui.service > /dev/null << EOF
[Unit]
Description=GenieACS UI
After=network.target

[Service]
User=genieacs
EnvironmentFile=/opt/genieacs/genieacs.env
ExecStart=/usr/local/bin/genieacs-ui

[Install]
WantedBy=default.target
EOF
```
## Configure Log Rotation
```bash
sudo tee /etc/logrotate.d/genieacs > /dev/null << EOF
/var/log/genieacs/*.log /var/log/genieacs/*.yaml {
    daily
    rotate 30
    compress
    delaycompress
    dateext
}
EOF
```
## Start GenieACS Services
Reload systemd and start the services:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now genieacs-{cwmp,fs,ui,nbi}
```
## Access GenieACS UI
Retrieve the server IP address and access GenieACS UI:
```bash
IPv4=$(ip -4 addr show enp1s0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
echo "#### GenieACS UI access: http://$IPv4:3000 ####"
```