## What I Did

### 1. Prepared the Virtual Machine and Installed Red Hat Linux 7.x

Downloaded the RHEL 7.x ISO from the Red Hat Customer Portal (requires a developer subscription—free for non-production use). Created a new VM in VMware with at least 2 GB RAM, 2 vCPUs, and 20+ GB disk space.

Booted from the ISO and followed the graphical installer:
- Selected language and keyboard.
- Configured partitioning (automatic for simplicity).
- Set a strong root password and created a regular user with sudo privileges.
- Completed the installation and rebooted.

**Post-install basics (run as root or with `sudo`)**:
```bash
# Update the system
sudo yum update -y

# Install essential tools
sudo yum install -y vim curl wget net-tools

# Verify network and hostname
hostnamectl
ip addr show
```

*Screenshot description: RHEL 7 desktop/login screen after fresh install, showing successful boot and network connectivity.*

---

### 2. Installed and Configured Docker on RHEL 7.x

RHEL 7 uses `yum` for package management.

```bash
# Install prerequisites
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# Add Docker repository (using CentOS repo as common workaround for RHEL 7 Docker CE)
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker
sudo yum install -y docker-ce docker-ce-cli containerd.io

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to docker group (log out/in after)
sudo usermod -aG docker $USER

# Verify
docker --version
docker ps
```

*Screenshot description: Terminal showing Docker service running and `docker version` output.*

**Note**: If you encounter repository or subscription issues, ensure your RHEL subscription is active and repositories are enabled (`sudo subscription-manager repos --enable=rhel-7-server-extras-rpms` etc.).

---

### 3. Prepared the HTML5 Gateway Files and Installed the Container

Copied the PSMGWDocker directory (from your 12.6 installation media/resources) to the Linux host (e.g., via SCP, WinSCP, or shared folder).

Navigated to the directory:

```bash
cd /path/to/PSMGWDocker
chmod +x html5_installation.sh
sudo ./html5_installation.sh localimage
```

This imports the Docker image (e.g., `cahtml5gw:<version_tag>`). Check with `docker images`.

Created a certificates directory:

```bash
sudo mkdir -p /opt/cert
```

For lab use, generated self-signed certificates using OpenSSL (place required .crt/.key files in `/opt/cert` and adjust ownership/permissions as needed).

---

### 4. Launched the PSM HTML5 Gateway Docker Container

Ran the container with appropriate options for testing (adjust `<PVWA URL>`, hostname, and version tag):

```bash
sudo docker run --restart unless-stopped -ti -p 443:8443 \
  -v /opt/cert:/opt/import:ro \
  -d --cap-drop=all --cap-add={CHOWN,DAC_OVERRIDE,FOWNER,SETGID,SETUID} \
  -e AcceptCyberArkEULA=yes \
  -e EndPointAddress=https://<your-pvwa-hostname-or-ip>/passwordvault \
  --hostname <your-gateway-fqdn-or-name> \
  --name <your-gateway-name> cahtml5gw:<version_tag>
```

Verified the container is running: `docker ps`.

Opened port 443 in the VM's firewall if needed:
```bash
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload
```

*Screenshot description: `docker ps` output showing the running HTML5 Gateway container, and browser accessing https://gateway-ip (noting any self-signed cert warning for lab).*

---

### 5. Configured Integration in PVWA/PSM

In the PVWA:
- Added/configured the PSM HTML5 Gateway server (using the gateway's address/hostname).
- Updated connection components or account settings to enable HTML5 sessions.
- Tested an RDP/SSH session via the HTML5 interface in the browser.

*Screenshot description: PVWA interface showing the added HTML5 Gateway and a successful browser-based session to a target system.*

---

### 6. Basic Hardening and Verification

Checked logs:
```bash
docker logs <container-name>
```

Tested connectivity, certificate trust (accepted warnings in lab), and session functionality.

## Key Takeaways

- **Linux Fundamentals** — Comfortable with RHEL 7 installation, basic commands (`yum`, `systemctl`, file permissions), and Docker management.
- **Containerized Deployment** — Learned to run and manage the HTML5 Gateway as a Docker container, including volume mounts for certificates.
- **CyberArk Integration** — Understood how the HTML5 Gateway fits into PAM for browser-based privileged access.
- **Certificate Handling** — Practical experience with self-signed certs for testing (use proper CA-signed certs in production).
- **Troubleshooting** — Firewall, ports, logs, and common beginner pitfalls like permissions or network config.

This project builds strong foundational skills in Linux, containers, and PAM administration.

## About

A hands-on beginner project for installing and configuring the CyberArk PSM HTML5 Gateway on Red Hat Enterprise Linux 7.x with Docker. Perfect for those new to Linux seeking practical PAM experience.

**Notes**: 
- Adapt paths, hostnames, IPs, and version tags to your environment.
- For production, follow official security and certificate best practices.
- Use snapshots in your VM for easy resets during learning.

Feel free to extend this lab with load balancing, upgrades, or additional testing!

**Next Steps Ideas**: Integrate with a full CyberArk lab (PVWA + PSM), add load balancing simulation, or upgrade the gateway. Expand with Ansible for automation.

You can clone/fork this structure into a GitHub repo (add your screenshots to an `img/` folder). Replace placeholders with your actual screenshots and adjust version-specific details once you download the exact 12.6 package. Let me know if you need scripts, more detailed commands, or help generating image placeholders!
