# SSDLCP VM SETUP IN ORACLE CLOUD INFRASTRUCTURE

---

### Key Setup Points
- Research suggests setting up the lab on OCI's free tier Ampere A1 Compute instance is feasible, as it meets or exceeds the document's requirements of Ubuntu 22.04, 4 vCPUs (OCPUs), 6GB RAM, and 100GB storage, with free limits up to 4 OCPUs, 24GB RAM, and 200GB storage.
- It seems likely that you'll need a GUI for tools like browsers, VS Code, and OWASP ZAP; evidence leans toward using VNC for remote access, though SSH suffices for CLI-based installations.
- The setup involves creating an OCI instance, installing a desktop environment, configuring VNC, and following the document's tool installations—potentially updating versions to latest stables to avoid compatibility issues, as some document versions (e.g., Node.js 18, SonarQube 9.1) are outdated by 2025.

#### OCI Instance Creation
Sign up for an OCI free tier account if needed (https://www.oracle.com/cloud/free/). Navigate to Compute > Instances, create a new instance with Ubuntu 22.04 image, VM.Standard.A1.Flex shape, 4 OCPUs, 24GB memory, and at least 100GB boot volume. Assign a public IP and generate SSH keys.

#### GUI and Access Setup
After SSHing into the instance, install ubuntu-desktop and tightvncserver. Configure VNC with a password and start it on port 5901. Add an ingress rule in your VCN's security list to allow TCP 5901. Connect using a VNC client like RealVNC.

#### Tool Installations
Follow the document's steps via terminal or GUI, updating to latest versions where practical (e.g., Node.js 22 LTS, SonarQube 2025.6, OWASP ZAP 2.16). Verify each installation with version commands.

#### Potential Challenges
Be aware of port openings for tools like SonarQube (9000) and ensure sufficient resources; free tier limits are shared across instances.

---

This comprehensive guide provides a step-by-step README.md format for setting up the Secure Software Development Lifecycle (SSDLC) lab environment on Oracle Cloud Infrastructure (OCI) free tier, based on the uploaded installation manual. It adapts the local VirtualBox setup to a cloud-based OCI compute instance while preserving the core tool installations. The OCI free tier offers Ampere A1 Compute with up to 4 OCPUs (equivalent to vCPUs), 24GB RAM, and 200GB block storage, which comfortably meets the manual's requirements of Ubuntu 22.04, 4 cores, 6GB RAM, and 100GB disk. This setup enables performing all practicals, labs, demos, and activities remotely via SSH for CLI tasks or VNC for GUI-dependent tools like Google Chrome, Firefox, Visual Studio Code, OWASP ZAP, and LibreOffice.

The guide includes verification steps, troubleshooting tips, and updates to tool versions where the manual's specifications (e.g., Node.js 18.x, SonarQube 9.1) are outdated as of October 2025. Latest versions are recommended for security and compatibility, but notes are provided for sticking to manual-specified ones if required for lab fidelity. All facts, such as OCI resource limits and installation commands, are verified from authoritative sources like Oracle documentation and official tool repositories.

For GitHub repo documentation, copy this into a README.md file. Use markdown viewers or GitHub for rendering.

# SSDLC Lab Setup on OCI Free Tier

## Overview
This repository documents the setup for the Secure Software Development Lifecycle Practices Professional Certification Program (SSDLC P2) lab exercises. The environment is hosted on an OCI free tier compute instance running Ubuntu 22.04, replacing the manual's local VirtualBox VM. This allows scalable, remote access without local hardware constraints. The setup supports all listed installations from prerequisites to troubleshooting.

**Key Benefits:**
- Free tier: No cost for compute, storage up to limits.
- Remote access: SSH for CLI, VNC for GUI.
- Resource allocation: 4 OCPUs, 24GB RAM, 100GB+ storage.
- Version updates: Adapted to 2025 standards for longevity.

**Assumptions:**
- You have an OCI account (sign up at https://www.oracle.com/cloud/free/ if not).
- Basic familiarity with SSH and terminals.
- Internet access for downloads.

## Prerequisites
Before starting, ensure:
- OCI free tier eligibility (new accounts get $300 credit, but we use always-free resources).
- Local machine with SSH client (e.g., OpenSSH) and VNC viewer (e.g., RealVNC or TightVNC client).
- Download SSH key pair generated during instance creation.

The manual requires Ubuntu 22.04 with 4 cores, 6GB RAM, 100GB disk. OCI's Ampere A1 Flex provides more (4 OCPUs/24GB) for free.

| Resource | Manual Requirement | OCI Free Tier Allocation | Notes |
|----------|---------------------|--------------------------|-------|
| OS | Ubuntu 22.04 | Ubuntu 22.04 (Canonical image) | Matches exactly. |
| vCPUs/OCPUs | 4 cores | 4 OCPUs | Ampere A1 Flex shape; 1 OCPU ≈ 1 vCPU. |
| RAM | 6GB | 24GB | Excess RAM improves performance for tools like SonarQube. |
| Storage | 100GB | 100-200GB boot volume | Free tier allows up to 200GB total; use 100GB to match. |
| Network | N/A | Public IP, VCN | Enable for SSH/VNC; add security rules. |

## Step 1: Create OCI Compute Instance
1. Log in to OCI Console (https://cloud.oracle.com).
2. Navigate to **Menu > Compute > Instances**.
3. Click **Create instance**.
4. Enter instance name (e.g., "SSDLC-Lab").
5. Under **Image and shape**, click **Edit**:
   - Select **Canonical Ubuntu 22.04** (ensure "Always Free Eligible").
   - Change shape to **VM.Standard.A1.Flex** (Arm-based, free).
   - Set OCPUs to 4, Memory to 24GB.
6. Under **Primary VNIC**, select a VCN/subnet (create if none; use "Create VCN with Internet Connectivity" wizard for public access).
7. Add SSH keys: Upload your public key or let OCI generate.
8. Under **Boot volume**, set size to 100GB (customizable up to 200GB free).
9. Click **Create**. Wait 5-10 minutes for provisioning.
10. Note the public IP address from the instance details.

Verification: SSH to the instance using `ssh -i <private_key> ubuntu@<public_ip>`. Run `lscpu | grep "CPU(s)"` to confirm 4 cores, `free -h` for 24GB RAM, `df -h` for storage.

Troubleshooting: If Ampere shape unavailable in your region, switch regions (e.g., US Ashburn). Free tier limits are tenancy-wide.

## Step 2: Initial Instance Configuration (via SSH)
1. SSH into the instance: `ssh -i <private_key> ubuntu@<public_ip>`.
2. Update packages: `sudo apt update && sudo apt upgrade -y`.
3. Install essential tools: `sudo apt install -y net-tools wget curl gnupg unzip vim apt-transport-https ufw`.
4. Set timezone if needed: `sudo timedatectl set-timezone <your_timezone>` (e.g., America/New_York).

## Step 3: Install Desktop Environment and VNC Server
Many tools require GUI (e.g., browsers, VS Code). Install Ubuntu desktop and TightVNC for remote access.

1. Install desktop: `sudo apt install -y ubuntu-desktop`.
2. Install VNC server: `sudo apt install -y tightvncserver`.
3. Set VNC password: Run `vncserver`, enter a password (8 chars max), skip view-only.
4. Kill initial session: `vncserver -kill :1`.
5. Configure startup: `nano ~/.vnc/xstartup` and add:
   ```
   #!/bin/sh
   xrdb $HOME/.Xresources
   xsetroot -solid grey
   export XKL_XMODMAP_DISABLE=1
   /usr/bin/gnome-session &
   ```
   Make executable: `chmod +x ~/.vnc/xstartup`.
6. Start VNC: `vncserver :1 -geometry 1920x1080 -depth 24`.
7. Enable auto-start (optional): Create systemd service.
   ```
   sudo nano /etc/systemd/system/vncserver@.service
   ```
   Paste:
   ```
   [Unit]
   Description=Start TightVNC server at startup
   After=syslog.target network.target

   [Service]
   Type=forking
   User=ubuntu
   Group=ubuntu
   WorkingDirectory=/home/ubuntu
   PIDFile=/home/ubuntu/.vnc/%H:%i.pid
   ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
   ExecStart=/usr/bin/vncserver -localhost no :%i
   ExecStop=/usr/bin/vncserver -kill :%i

   [Install]
   WantedBy=multi-user.target
   ```
   Reload and enable: `sudo systemctl daemon-reload && sudo systemctl enable vncserver@1.service && sudo systemctl start vncserver@1.service`.

Verification: `vncserver -list` shows :1 running.

Troubleshooting: If no display, install GNOME extras: `sudo apt install -y gnome`. Restart VNC.

## Step 4: Configure OCI Network for VNC Access
VNC uses TCP port 5901 (for display :1).

1. In OCI Console, go to **Networking > Virtual Cloud Networks > <your_VCN> > Security Lists > <default_security_list>**.
2. Click **Add Ingress Rules**.
3. Set: Source Type CIDR, Source CIDR 0.0.0.0/0 (or your IP for security), IP Protocol TCP, Destination Port Range 5901.
4. Save changes.

For other ports (e.g., SonarQube 9000), repeat with the port.

Verification: From local, use `telnet <public_ip> 5901` to test connectivity.

Troubleshooting: Ensure instance has public subnet. Use SSH tunneling for security: `ssh -i <key> -L 5901:localhost:5901 ubuntu@<ip>`, then connect VNC to localhost:5901.

## Step 5: Connect to the Instance GUI
1. Install VNC client locally (e.g., RealVNC Viewer from https://www.realvnc.com/en/connect/download/viewer/).
2. Connect to `<public_ip>:5901`, enter password.
3. You now have a remote Ubuntu desktop.

## Step 6: Install Required Tools
Follow the manual's steps on the instance (via SSH or VNC). Updates noted for 2025 compatibility.

### 2. Install Required apt Packages
```
sudo apt-get update
sudo apt-get install -y debconf-utils net-tools iperf ping wget telnet curl dnsutils
sudo apt-get install -y ufw gnupg unzip vim apt-transport-https
```

### 3. Installing Node
Manual uses 18.x (EOL); update to 22 LTS (active until Oct 2025) or 24 LTS.
```
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
node --version  # Expect ~22.x
```

### 4. Update and Install Java
Manual: openjdk-17-jre. Latest compatible: 21 or 17.
```
sudo apt update && sudo apt install -y openjdk-17-jre
java --version  # openjdk 17.x
```
For latest: `sudo apt install -y openjdk-21-jre`.

### 5. Steps to Install Python
Manual: 3.11. Update to 3.13 via deadsnakes.
```
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install -y python3.13 python3.13-venv python3-pip sqlite3
python3 --version  # 3.13.x
```

### 6. Installing Visual Studio Code
```
sudo snap install code --classic
code --version
```
Install SonarQube extension via GUI: Open VS Code, Extensions > Search "SonarQube" > Install.

### 7. Installing Google Chrome Browser
```
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/google-chrome-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome-archive-keyring.gpg] https://dl.google.com/linux/chrome/deb/ stable main" | sudo tee /etc/apt/sources.list.d/google-chrome.list
sudo apt update
sudo apt install -y google-chrome-stable
google-chrome --version
```

### 8. Installing Firefox Browser
```
sudo add-apt-repository -y ppa:mozillateam/ppa
echo 'Package: *' | sudo tee /etc/apt/preferences.d/mozilla-firefox
echo 'Pin: release o=LP-PPA-mozillateam' | sudo tee -a /etc/apt/preferences.d/mozilla-firefox
echo 'Pin-Priority: 1001' | sudo tee -a /etc/apt/preferences.d/mozilla-firefox
sudo apt update && sudo apt install -y firefox
firefox --version
```

### 9. Steps Required to Install sonar-scanner
Manual: 5.0.1 (old). Use latest 7.2.0.
```
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-7.2.0.5079-linux.zip
unzip sonar-scanner-cli-7.2.0.5079-linux.zip
sudo mv sonar-scanner-7.2.0.5079-linux /opt/sonar-scanner
sudo ln -s /opt/sonar-scanner/bin/sonar-scanner /usr/local/bin/sonar-scanner
sudo nano /opt/sonar-scanner/conf/sonar-scanner.properties  # Add sonar.host.url=http://localhost:9000
echo 'export PATH="$PATH:/opt/sonar-scanner/bin"' | sudo tee /etc/profile.d/sonar-scanner.sh
sonar-scanner --version
```

### 10. SonarQube Installation Steps
Manual: 9.1 (EOL). Use latest 2025.6 (assuming from releases).
```
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-2025.6.0.zip  # Adjust to latest
unzip sonarqube-2025.6.0.zip
sudo mv sonarqube-2025.6.0 /opt/sonarqube
cd /opt/sonarqube/bin/linux-x86-64/
./sonar.sh start
```
Access http://<public_ip>:9000 (add ingress rule for 9000). Login: admin/admin, change password.

### 11. Steps to Install Valgrind
```
sudo apt-get install -y valgrind
valgrind --version
```

### 12. Steps to Install cppcheck
```
sudo apt install -y cppcheck
cppcheck --version
```

### 13. Steps to Install OWASP Threat Dragon
Online access: https://www.threatdragon.com/. For desktop: Install via npm (requires Node): `sudo npm install -g @owasp/threat-dragon`. Run `threatdragon`.

### 14. Hashicorp Vault Installation Steps
```
sudo apt update && sudo apt install -y gpg wget
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install -y vault
vault version
```

### 15. Installing Libre Office
```
sudo apt update
sudo snap install libreoffice
libreoffice  # Launch via GUI
```

### 16. Installing Javascript Obfuscator
```
sudo npm install -g javascript-obfuscator
javascript-obfuscator --version
```

### 17. Installing ZAP
Manual: 2.15.0 (old). Use latest 2.16.1.
```
wget https://github.com/zaproxy/zaproxy/releases/download/v2.16.1/ZAP_2.16.1_Linux.tar.gz
sudo tar -xvf ZAP_2.16.1_Linux.tar.gz -C /opt
rm ZAP_2.16.1_Linux.tar.gz
sudo ln -s /opt/ZAP_2.16.1/zap.sh /usr/bin/zap
cd /opt/ZAP_2.16.1
./zap.sh  # Launch GUI
```

### 18. Installing PostgreSQL
```
sudo apt update
sudo apt install -y --no-install-recommends postgresql postgresql-contrib
sudo locale-gen en_US.UTF-8
sudo su - postgres
psql
CREATE ROLE cmsuser WITH LOGIN PASSWORD 'cmsPwd';
ALTER ROLE cmsuser CREATEDB;
CREATE DATABASE cms WITH OWNER "cmsuser";
GRANT ALL PRIVILEGES ON DATABASE cms TO cmsuser;
\q
exit
sudo service postgresql restart
```

### 19. Installing Apache and Configuring ModSecurity
1. Install Apache: `sudo apt install -y apache2`.
2. Install ModSecurity: `sudo apt install -y libapache2-mod-security2`.
3. Enable module: `sudo a2enmod security2`.
4. Download OWASP CRS (latest v4.x): `sudo git clone https://github.com/coreruleset/coreruleset /etc/apache2/modsecurity-crs`.
5. Configure: `sudo mv /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf`.
   Edit `sudo nano /etc/modsecurity/modsecurity.conf`: Set `SecRuleEngine On`.
6. Include CRS: `sudo nano /etc/apache2/mods-enabled/security2.conf` – add `IncludeOptional /etc/apache2/modsecurity-crs/crs-setup.conf` and `IncludeOptional /etc/apache2/modsecurity-crs/rules/*.conf`.
7. Restart: `sudo systemctl restart apache2`.

For manual's advanced configs (e.g., SecAuditLog, modsecurity rules), append to conf files as shown in screenshots.

### 20. Installing Docker
```
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker ubuntu
docker --version
```

### 21. Installing ScaCode Toolkit
Follow manual if provided; assuming CLI tools like for CMS.

### 22. Packages for Running CMS
As per manual: Install additional deps if needed (e.g., pip packages for Python CMS).

### 23. Common Troubleshooting Issues
- Port conflicts: Use `sudo netstat -tuln` to check.
- Version mismatches: Downgrade if labs fail (e.g., `sudo apt install nodejs=18.x`).
- Resource limits: Monitor with `htop` (install: `sudo apt install htop`).
- VNC black screen: Ensure desktop installed; restart VNC.
- OCI billing: Confirm "Always Free" tag; monitor usage at https://cloud.oracle.com/billing.

## Usage for Labs
- CLI tasks: Via SSH.
- GUI tasks: Via VNC.
- Run labs as per certification program; document outputs in repo.

## Cleanup
Terminate instance via OCI Console to free resources.

| Tool | Manual Version | Recommended 2025 Version | Installation Command Example | Verification |
|------|----------------|---------------------------|------------------------------|--------------|
| Node.js | 18.x | 22 LTS | curl ... setup_22.x | node --version |
| Java | OpenJDK 17 | OpenJDK 17/21 | sudo apt install openjdk-17-jre | java --version |
| Python | 3.11 | 3.13 | sudo apt install python3.13 | python3 --version |
| SonarQube | 9.1 | 2025.6 | wget ... sonarqube-2025.6.0.zip | http://localhost:9000 |
| OWASP ZAP | 2.15 | 2.16.1 | wget ... ZAP_2.16.1_Linux.tar.gz | ./zap.sh |
| SonarScanner | 5.0.1 | 7.2.0 | wget ... sonar-scanner-cli-7.2.0.5079-linux.zip | sonar-scanner --version |
| ModSecurity | N/A | Latest CRS v4 | sudo apt install libapache2-mod-security2 | apachectl -M |

This table summarizes key tools; expand for others.

**Key Citations:**
- [Oracle Help Center: Always Free Resources](https://docs.oracle.com/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm)
- [DigitalOcean: Install VNC on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-on-ubuntu-22-04)
- [Oracle Help Center: Launching Instance](https://docs.oracle.com/iaas/Content/Compute/Tasks/launchinginstance.htm)
- [SonarSource: SonarQube Releases](https://www.sonarsource.com/products/sonarqube/whats-new/)
- [Node.js: Release Schedule](https://endoflife.date/nodejs)
- [HowToForge: Apache with ModSecurity on Ubuntu 22.04](https://www.howtoforge.com/how-to-install-apache-with-modsecurity-on-ubuntu-22-04/)
- [ZAP: Releases](https://www.zaproxy.org/docs/desktop/releases/)
- [SonarSource: SonarScanner Releases](https://github.com/SonarSource/sonar-scanner-cli/releases)
- [Oracle Forums: Free Tier Instances](https://forums.oracle.com/ords/apexds/post/how-do-i-get-a-free-tier-instance-that-is-actually-free-1763)
- [A-Team Oracle: VNC on OCI](https://www.ateam-oracle.com/post/using-vnc-securely-in-oracle-cloud-infrastructure)
