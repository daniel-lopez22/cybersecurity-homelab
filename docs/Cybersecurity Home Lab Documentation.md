# Cybersecurity Home Lab Documentation

**Ubuntu Server + Proxmox + Wazuh SIEM**

---
## Table of Contents
1. Overview
2. Lab Architecture
3. Host and Network Prerequisites
4. Proxmox Setup and Access
5. Virtual Machine Creation Standards
6. Ubuntu Server Initial Configuration
7. SSH Access and Authentication
8. Core Service Management Commands
9. Wazuh Stack Overview
10. IP Addressing Reference
11. Important File Locations
12. Credential Handling and Storage
13. Best Practices and Operational Philosophy
14. Common Failure Scenarios (Summary)
15. Wazuh API Usage and Validation
16. Wazuh Indexer Validation
17. Wazuh Role and Permission Management
18. Wazuh Dashboard Access
19. Service Startup Order
20. Log Locations for Troubleshooting
21. System Hardening Steps Implemented
22. Backup and Snapshot Strategy
23. Recovery Procedures
24. Known Failure Scenarios (Detailed)
25. Operational Checklist
26. Resume-Relevant Skills Demonstrated
27. Lessons Learned and Design Improvements
28. Future Expansion Ideas
29. Final Notes

---
## 1. Overview
This document captures the critical steps, lessons learned, and operational knowledge gained while building a personal cybersecurity home lab.

The lab simulates a small SOC-style environment focused on:

- Linux server administration
- System hardening
- Log collection and monitoring    
- SIEM deployment and troubleshooting    
- Virtualized infrastructure management    

The goal is to enable **any technically inclined user** to reproduce this lab and avoid common pitfalls encountered during setup.

---
## 2. Lab Architecture
**Core Components**
- Hypervisor: Proxmox VE    
- Guest OS: Ubuntu Server LTS    
- Security Platform: Wazuh (Manager, Indexer, Dashboard)    
- Access Method: SSH and web dashboards    
**High-Level Flow**
- Proxmox hosts the Ubuntu Server VM    
- Ubuntu Server runs the full Wazuh stack    
- Logs are generated locally for validation    
- Analyst access is via browser and SSH    

---
## 3. Host and Network Prerequisites
**Hardware Recommendations**
- CPU with virtualization support enabled in BIOS    
- Minimum 16 GB RAM on host    
- SSD-backed storage strongly recommended    
**Network Requirements**
- LAN access via bridged networking    
- Static IPs preferred for infrastructure services    
- DNS resolution working on the local network    

---
## 4. Proxmox Setup and Access
**Accessing Proxmox Web Interface**
- URL format:    
    `https://<PROXMOX_IP>:8006`    
- Example:    
    `https://192.168.1.10:8006`    
**Default Login Realm**
- Username: `root`    
- Realm: `Linux PAM authentication`
**Important Proxmox Commands**
```bash
qm list 
qm start <VMID> 
qm shutdown <VMID> 
qm stop <VMID> 
qm snapshot <VMID> <snapshot-name> 
qm rollback <VMID> <snapshot-name>
```
**Key Tip**
- Always snapshot before:    
    - Kernel upgrades        
    - Wazuh installs        
    - Configuration changes        

---
## 5. Virtual Machine Creation Standards
**VM Configuration Guidelines**
- BIOS: OVMF (UEFI)    
- Machine Type: q35    
- CPU Type: host    
- Sockets: 1    
- Cores: 6    
- Memory: 10 GB   
- Ballooning: Disabled    
- Disk:    
    - SCSI (VirtIO)        
    - 60 to 100 GB recommended        
- Network:    
    - Bridge: vmbr0        
    - Model: VirtIO        
**Why These Matter**
- Host CPU improves performance    
- Disabled ballooning prevents SIEM instability    
- VirtIO improves disk and network throughput    

---
## 6. Ubuntu Server Initial Configuration
**Post-Install Checklist**
- Update system packages    
`sudo apt update && sudo apt upgrade -y`
- Install core utilities    
`sudo apt install -y curl wget unzip net-tools htop`
- Verify network configuration
`ip a ip route`
- Confirm hostname
`hostnamectl`

---
## 7. SSH Access and Authentication
**SSH Access from Another Machine**
`ssh <username>@<UBUNTU_VM_IP>`
Example:
`ssh daniel@192.168.1.222`
**SSH Service Management**
`sudo systemctl status ssh sudo systemctl restart ssh`
**Security Best Practice**
- Disable root SSH login    
- Use strong passwords or SSH keys    
- Restrict SSH to trusted subnets if possible    

---
## 8. Core Service Management Commands
**System Service Control**
```bash
sudo systemctl status <service> 
sudo systemctl start <service> 
sudo systemctl stop <service> 
sudo systemctl restart <service>
```
**Common Services Used**
- ssh    
- wazuh-manager    
- wazuh-indexer    
- wazuh-dashboard    

---
## 9. Wazuh Stack Overview
**Wazuh Components**
- Wazuh Manager    
    - Handles log processing and rules        
- Wazuh Indexer    
    - Stores alerts and events        
- Wazuh Dashboard    
    - Web interface for analysis        
**Service Control**
```bash
sudo systemctl restart wazuh-manager 
sudo systemctl restart wazuh-indexer 
sudo systemctl restart wazuh-dashboard
```

---
## 10. IP Addressing Reference
**Typical IP Usage**
- Proxmox Host    
    - Example: `192.168.1.222`
- Ubuntu Server VM    
    - Example: `192.168.1.147`
- Wazuh Dashboard    
    - https://<UBUNTU_VM_IP>:5601        
- Wazuh Indexer API    
    - https://<UBUNTU_VM_IP>:9200        
- Wazuh Manager API    
    - https://<UBUNTU_VM_IP>:55000        

---
## 11. Important File Locations
**Wazuh Configuration Files**
- Manager config   
`/var/ossec/etc/ossec.conf`
- API users    
`/var/ossec/api/configuration/auth/users`
- Dashboard config    
`/usr/share/wazuh-dashboard/data/wazuh/config/wazuh.yml`
**System Logs**
`/var/log/syslog /var/log/auth.log`

---
## 12. Credential Handling and Storage
**Important Rule**
- Never hardcode passwords into scripts or documentation    
**Where Credentials Are Typically Stored**
- Wazuh Dashboard credentials    
    - During installation output        
    - Stored securely offline        
- API credentials    
    - Defined during Wazuh install        
    - Used for local authentication testing        
**Best Practice Learned**
- Store credentials in:    
    - Password manager        
    - Encrypted notes        
    - Offline secure storage        

---
## 13. Best Practices and Operational Philosophy
- Always snapshot before major changes    
- Verify services after every restart    
- Test APIs locally with curl before assuming failures    
- Do not trust dashboards until backend services are verified    
- SIEM workloads require stable memory allocation    

---
## 14. Common Failure Scenarios (Summary)
**Wazuh Manager Fails to Start**
- Cause:    
    - Invalid YAML        
    - Incorrect permissions        
- Fix:    
    - Check logs        
    `journalctl -xeu wazuh-manager`
**Dashboard Loads but No Data Appears**
- Cause:    
    - Indexer authentication mismatch        
- Fix:    
    - Verify indexer user permissions        
    - Validate credentials via curl        
**Ubuntu VM Network Drops**
- Cause:    
    - DHCP lease changes        
- Fix:    
    - Use static IP assignment        

---
## 15. Wazuh API Usage and Validation
The Wazuh API is critical for validating that the manager is running correctly even when the dashboard fails.
### Wazuh API Authentication Test
`curl -k -u '<api_user>:<api_password>' \ https://127.0.0.1:55000/security/user/authenticate?raw=true`
**Expected Result**
- A long authentication token string    
- Confirms:    
    - Wazuh Manager is running        
    - API credentials are valid        
**Common Failure**
- `Unauthorized`    
    - Indicates incorrect credentials or desynchronized configuration        

---
## 16. Wazuh Indexer Validation
The indexer stores alerts and must be validated independently of the dashboard.
### Check Wazuh Indices
`curl -k -u '<indexer_user>:<indexer_password>' \ https://127.0.0.1:9200/_cat/indices/wazuh-alerts-4.x-*?v`
**Expected Output**
- Index names    
- Document count    
- Status should be `green` or `yellow`    
**If No Indices Appear**
- Manager may not be writing data    
- Indexer authentication may be broken    

---
## 17. Wazuh Role and Permission Management
Permissions directly affect dashboard visibility.
### Verify Indexer Security Roles
`curl -k -u '<admin_user>:<admin_password>' \ https://127.0.0.1:9200/_plugins/_security/api/roles`
**Important Role**
- `logstash`    
    - Required for alert ingestion       
**Lesson Learned**
- Dashboard issues are often permission issues, not service failures    

---
## 18. Wazuh Dashboard Access
### Dashboard URL
`https://<UBUNTU_VM_IP>:5601`
### Common Login Realms
- `native`    
- `internal`    
**Tip**
- Use the same realm consistently    
- Document which realm works during install    

---
## 19. Service Startup Order
Correct startup order prevents silent failures.
**Recommended Order**
1. wazuh-indexer    
2. wazuh-manager    
3. wazuh-dashboard   
```bash
sudo systemctl restart wazuh-indexer 
sudo systemctl restart wazuh-manager 
sudo systemctl restart wazuh-dashboard
```

---
## 20. Log Locations for Troubleshooting
### Wazuh Logs
`/var/ossec/logs/ossec.log /var/ossec/logs/api.log`
### Dashboard Logs

`/var/log/wazuh-dashboard/wazuh-dashboard.log`
### Indexer Logs
`/var/log/wazuh-indexer/wazuh-indexer.log`
**Rule**
- Always check logs before restarting services repeatedly    

---
## 21. System Hardening Steps Implemented
- Automatic security updates enabled    
- Unnecessary services disabled    
- SSH access restricted    
- Firewall rules verified    
### Check Active Services
`systemctl list-units --type=service --state=running`

---
## 22. Backup and Snapshot Strategy
**Proxmox Snapshots**
- Before Wazuh installation    
- Before configuration changes    
- Before OS upgrades   
**Snapshot Naming Convention**
`pre-wazuh-install post-wazuh-install pre-config-change-<date>`
**Lesson Learned**
- Snapshots saved multiple hours of recovery time    

---
## 23. Recovery Procedures
### Restore a VM Snapshot
`qm rollback <VMID> <snapshot-name>`
### Emergency Service Reset
`sudo systemctl daemon-reexec sudo systemctl daemon-reload`

---
## 24. Known Failure Scenarios (Expanded)
### Issue: Services Running but Dashboard Empty
**Cause**
- Indexer role misconfiguration    
**Fix**
- Verify logstash role permissions    
- Reapply role mappings    
### Issue: Authentication Suddenly Stops Working
**Cause**
- Password mismatch between components    
**Fix**
- Re-sync credentials    
- Validate each component independently using curl    
### Issue: Kernel Update Breaks Services
**Cause**
- Reboot required after update   
**Fix**
`sudo reboot`

---
## 25. Operational Checklist
Before assuming a failure:
- Verify VM is running    
- Verify network connectivity    
- Verify disk space    
- Verify service status    
- Verify logs    
- Verify API access    

---
## 26. Resume-Relevant Skills Demonstrated

- Linux system administration    
- Virtualization with Proxmox    
- SIEM deployment and maintenance    
- Network troubleshooting    
- Log analysis    
- Secure service configuration    

---
## 27. Lessons Learned and Design Improvements
- Use a written credential log during installation    
- Split services into separate VMs earlier    
- Implement static IPs immediately    
- Enable monitoring alerts sooner    
- Document every successful command as it happens
---
## 28. Future Expansion Ideas
- Add endpoint agents    
- Forward logs from additional VMs    
- Integrate firewall logs    
- Create detection use cases    
- Automate health checks    

---
## 29. Final Notes
This lab provided hands-on exposure to real-world infrastructure issues that cannot be learned through theory alone. Troubleshooting service dependencies, authentication failures, and network issues closely mirrors challenges faced in SOC and infrastructure roles.
