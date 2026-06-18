## What I Did

### 1. Installed VMware and Created Ubuntu Linux VM (Beginner-Friendly)

If you don't have VMware or Linux experience, start here.

1. Download and install **VMware Workstation Player** (free for personal use) or Workstation Pro from Broadcom/VMware site.
2. Download **Ubuntu 22.04 LTS Desktop ISO** from ubuntu.com.
3. Open VMware → Create a New Virtual Machine.
   - Select "Typical" → Choose the Ubuntu ISO.
   - Name it e.g., "CyberArk-HTML5-Gateway-Lab".
   - Allocate: **4-8 GB RAM**, **2-4 vCPUs**, **20-40 GB disk** (thin provisioned).
   - Finish and power on.
4. Follow the Ubuntu installer: Choose English, install Ubuntu, create a user (e.g., `labuser` with sudo), and complete setup.
5. After login, update the system:
   ```
   sudo apt update && sudo apt upgrade -y
   sudo apt install net-tools curl -y
   ```
6. Install VMware Tools for better performance (copy-paste, shared folders): In VMware menu → VM → Install VMware Tools, then follow on-screen instructions in Ubuntu.

*Screenshot idea: VMware console showing Ubuntu desktop after install.*

---

### 2. Prepared the Linux Host for HTML5 Gateway

1. Install Docker (recommended for isolation and simplicity):
   ```
   sudo apt install docker.io docker-compose -y
   sudo systemctl enable --now docker
   sudo usermod -aG docker $USER  # Log out and back in
   ```
2. Verify: `docker --version` and `docker run hello-world`.
3. (Optional but recommended) Install utilities:
   ```
   sudo apt install openssl -y
   ```

---

### 3. Downloaded and Installed CyberArk HTML5 Gateway 12.6 (Docker Method)

**Note**: Obtain the `HTML5 Gateway-Rls-12.6.x.zip` from the CyberArk Marketplace. Copy the `PSMGWDocker` directory to your Linux VM (e.g., via shared folder or SCP).

1. Navigate to the extracted `PSMGWDocker` directory.
2. Make the setup script executable:
   ```
   chmod +x html5_console.sh
   ```
3. Run the installation (use `-f` if conflicts):
   ```
   sudo ./html5_console.sh install -l
   ```
4. Create a certificates directory:
   ```
   sudo mkdir -p /opt/cert
   ```

---

### 4. Generated Self-Signed Certificates (For Lab/Testing)

**Warning**: Self-signed certs are for testing only. Use proper CA-signed certs in production.

Run these commands (adjust names/FQDN as needed, e.g., `html5gw.lab.local`):

```bash
cd /opt/cert

# Root CA
sudo openssl genrsa -out rootCA.key 4096
sudo openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 365 -out rootCA.crt

# Gateway cert
sudo openssl genrsa -out psmgw.key 2048
sudo openssl req -new -key psmgw.key -out psmgw.csr
sudo openssl x509 -req -in psmgw.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out psmgw.crt -days 365 -sha256
```

Copy necessary files (PSM/PVWA CA certs if available) into `/opt/cert`.

---

### 5. Launched the HTML5 Gateway Container

Use a command similar to this (adapt for your version/PVWA):

```bash
sudo ./html5_console.sh run -ti -p 443:8443 \
  -v /opt/cert:/opt/import:ro -d \
  -e AcceptCyberArkEULA=yes \
  -e EndPointAddress=https://your-pvwa.example.com/passwordvault \
  -e GWCert=psmgw.crt \
  -e GWKey=psmgw.key \
  -e GWCAFile=rootCA.crt \
  --name html5gw-lab \
  cahtml5gw:12.6_tag  # Replace with actual image tag from docker images
```

Verify: `docker ps` and check logs with `docker logs html5gw-lab`.

*Screenshot: Docker container running, browser accessing https://<VM-IP> showing gateway interface.*

---

### 6. Configured Firewall and Tested Basic Access

```bash
sudo ufw allow 443/tcp
sudo ufw enable
```

- From your host machine's browser, access `https://<VM-IP-or-FQDN>` (accept self-signed warning).
- In CyberArk PVWA (if available): Add/configure the PSM HTML5 Gateway server under Administration → PSM → HTML5 Gateway, then test a connection to a target account.

---

### 7. (Optional) Basic Hardening and Verification

- Review CyberArk hardening scripts in the package.
- Check services: `docker ps`, `systemctl status docker`.
- Test an end-to-end privileged session via browser.

## Key Takeaways

- **Virtualization Basics** — Comfortable creating and managing Linux VMs in VMware for lab environments.
- **Linux Fundamentals** — Package management (`apt`), permissions, services, and Docker on Ubuntu.
- **Containerized Deployment** — Using Docker for isolated, reproducible PAM components like HTML5 Gateway.
- **Certificate Management** — Importance of TLS for secure gateways and basic OpenSSL usage.
- **PAM Concepts** — Understanding HTML5 Gateway's role in browser-based secure access, JWT validation, and integration with PVWA/PSM.
- **Security Practices** — Least privilege, proper cert handling, and why production setups differ from labs (e.g., no self-signed certs).
- **Troubleshooting** — Common issues like port conflicts, cert validation, and container logs.

## About

A hands-on PAM infrastructure project deploying CyberArk PSM HTML5 Gateway 12.6 on Ubuntu Linux in VMware. Focuses on beginner accessibility while covering real-world privileged access management scenarios.

### Resources
- Official CyberArk Docs: Search for "Install PSM HTML5 Gateway" (Docker method preferred for this version).
- VMware/Ubuntu guides for setup.
- CyberArk Marketplace for the 12.6 package.

**Next Steps Ideas**: Integrate with a full CyberArk lab (PVWA + PSM), add load balancing simulation, or upgrade the gateway. Expand with Ansible for automation.

You can clone/fork this structure into a GitHub repo (add your screenshots to an `img/` folder). Replace placeholders with your actual screenshots and adjust version-specific details once you download the exact 12.6 package. Let me know if you need scripts, more detailed commands, or help generating image placeholders!
