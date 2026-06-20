# CyberArk-PSM-HTML5-Gateway-12.6-Lab
 
 *Scenario-based hands-on project deploying CyberArk PSM HTML5 Gateway on Linux in VMware for secure browser-based privileged access.*

## Overview

This lab guides you through setting up a CyberArk PSM HTML5 Gateway (version 12.6) on a Linux virtual machine running in VMware Workstation/Player. The HTML5 Gateway (based on Apache Guacamole) enables secure RDP/SSH connections to target systems directly from a web browser, without requiring thick clients such as RDP. 

This project includes detailed steps for installing Red Hat Enterprise Linux 7.x on VMware, prerequisites, Docker-based installation (simpler and more isolated than RPM), certificate setup for testing, basic configuration, and integration notes with a CyberArk environment (e.g., PVWA/PSM). I'll practice virtualization, Linux basics, container management, and PAM concepts.

**Scenario**: As a junior PAM administrator in a company deploying CyberArk PAM Self-Hosted. I need to provide browser-based access to privileged sessions while maintaining security (TLS, JWT validation). This lab simulates a standalone HTML5 Gateway deployment.

## Tools Used

- **VMware Workstation/Player**: Free/Pro virtualization platform to run the Linux VM
- **Red Hat Enterprise Linux 7.x — Host operating system
- **Docker**: Container platform for running the HTML5 Gateway
- **OpenSSL**: For generating self-signed certificates (lab/testing only)
- **CyberArk HTML5 Gateway 12.6 package**: From CyberArk Marketplace (requires access)
- **Web Browser**: For testing connections
- **CyberArk PAM environment** (optional for full testing) — PVWA and PSM for configuration and JWT


