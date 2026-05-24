# PowerShell WinRM Setup for Ansible

PowerShell script for configuring Windows Remote Management (WinRM) to enable Ansible automation on Windows systems.

## 🎯 Purpose

This repository contains a PowerShell script (`winrm_setup_for_ansible.ps1`) that automates the configuration of WinRM on Windows servers, allowing them to be managed by Ansible playbooks from Linux control machines.

## 📁 Contents

### winrm_setup_for_ansible.ps1
**Single-purpose script** that configures Windows for Ansible automation:
- **WinRM Service Configuration** - Enables and starts WinRM service
- **HTTPS Listener Setup** - Configures secure HTTPS transport
- **Firewall Rules** - Opens necessary firewall ports
- **Authentication Configuration** - Enables Basic and NTLM authentication
- **Certificate Management** - Creates or uses existing certificates
- **Service Restart** - Ensures all services are running properly

## 🚀 Usage

### Prerequisites
- **Windows Server 2012 R2 or later**
- **Administrator privileges** (Run as Administrator)
- **PowerShell 4.0 or higher**
- **Network connectivity** to Ansible control machine

### Running the Script

**Option 1: Interactive Execution**
```powershell
# Right-click "Run as Administrator"
.\winrm_setup_for_ansible.ps1
```

**Option 2: Command Line**
```powershell
# Open PowerShell as Administrator
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
.\winrm_setup_for_ansible.ps1
```

**Option 3: Remotely via PSExec**
```cmd
psexec \\windows-server -s powershell.exe -ExecutionPolicy Bypass -File winrm_setup_for_ansible.ps1
```

**Option 4: Group Policy/SCCM**
Deploy script across multiple Windows servers using existing deployment tools.

## 🔧 What the Script Does

### WinRM Configuration
```powershell
# Enables WinRM quick config (HTTP + HTTPS)
Enable-PSRemoting -Force
Set-Item WSMan:\localhost\Client\TrustedHosts "*" -Force

# Configures HTTPS listener
$cert = New-SelfSignedCertificate -CertStoreLocation "cert:\LocalMachine\My" -DnsName $env:COMPUTERNAME
New-Item -Path WSMan:\localhost\Listener -Transport HTTPS -Address * -CertificateThumbprint $cert.Thumbprint -Force
```

### Firewall Configuration
```powershell
# Opens WinRM ports
New-NetFirewallRule -DisplayName "WinRM HTTPS" -Direction Inbound -LocalPort 5986 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "WinRM HTTP" -Direction Inbound -LocalPort 5985 -Protocol TCP -Action Allow
```

### Service Configuration
```powershell
# Sets WinRM service to automatic startup
Set-Service WinRM -StartupType Automatic
Restart-Service WinRM
```

## 🔍 Verification

### Test WinRM Connectivity
```powershell
# Test from Windows
Test-WSMan -ComputerName localhost

# Test from Linux Ansible control machine
ansible windows_servers -m win_ping
```

### Check WinRM Listeners
```powershell
# List all WinRM listeners
winrm enumerate winrm/config/Listener

# Check WinRM configuration
winrm get winrm/config
```

### Verify from Ansible
```bash
# Create inventory file
cat > windows_inventory << EOF
[windows_servers]
windows-server.example.com ansible_user=Administrator ansible_password=SecretPassword ansible_connection=winrm ansible_winrm_transport=basic ansible_winrm_server_cert_validation=ignore
EOF

# Test connectivity
ansible -i windows_inventory windows_servers -m win_ping
```

## 🛠️ Ansible Integration

### Configure Ansible for Windows

**Install required packages:**
```bash
pip install pywinrm
```

**Create Ansible inventory:**
```ini
[windows]
windows-server-1.example.com
windows-server-2.example.com

[windows:vars]
ansible_user=Administrator
ansible_password=YourPassword
ansible_connection=winrm
ansible_winrm_transport=basic
ansible_winrm_server_cert_validation=ignore
```

**Test Windows module:**
```yaml
# test_windows.yml
---
- name: Test Windows connectivity
  hosts: windows
  tasks:
    - name: Ping Windows servers
      win_ping:

    - name: Get Windows version
      win_command: cmd.exe /c ver
      register: win_ver

    - name: Display Windows version
      debug:
        var: win_ver.stdout
```

## ⚠️ Security Considerations

### Production Security
- **Use HTTPS** instead of HTTP for production environments
- **Proper SSL certificates** instead of self-signed certificates
- **Certificate validation** should be enabled (`ansible_winrm_server_cert_validation=validate`)
- **Specific trusted hosts** instead of wildcard `*`
- **Encrypted credentials** using Ansible Vault
- **Network segmentation** - WinRM should not be exposed to internet

### Secure Configuration Example
```ini
[windows:vars]
ansible_user=DOMAIN\ansible_user
ansible_connection=winrm
ansible_winrm_transport=ntlm
ansible_winrm_server_cert_validation=validate
ansible_winrm_ca_cert=/path/to/ca-cert.pem
```

## 🔧 Troubleshooting

### Common Issues

**WinRM Not Responding**
```powershell
# Check WinRM service
Get-Service WinRM

# Restart WinRM service
Restart-Service WinRM

# Check firewall
Get-NetFirewallRule -DisplayName "*WinRM*"
```

**Authentication Failures**
```powershell
# Test WinRM manually
winrm identify -remote:https://windows-server:5986 -auth:basic

# Check trusted hosts
Get-Item WSMan:\localhost\Client\TrustedHosts
```

**Certificate Issues**
```powershell
# List certificates
Get-ChildItem -Path Cert:\LocalMachine\My

# Remove old listeners
Remove-Item -Path WSMan:\localhost\Listener -Transport HTTPS -Recurse
```

**Ansible Connection Timeout**
```bash
# Increase timeout
ansible_winrm_operation_timeout_sec=120
ansible_winrm_read_timeout_sec=150
```

## 📋 Script Features

- **Idempotent** - Can be run multiple times safely
- **Error handling** - Comprehensive error checking and logging
- **Backup configuration** - Saves existing WinRM config before changes
- **Rollback capability** - Can restore previous configuration
- **Detailed logging** - Creates log file for troubleshooting

## 🚀 Advanced Usage

### Automated Deployment
```powershell
# Deploy via Group Policy
# Link script to Computer Configuration -> Policies -> Windows Settings -> Scripts -> Startup

# Deploy via SCCM
# Create package with script and run as system
```

### Batch Configuration
```powershell
# Run against multiple servers
$servers = "server1","server2","server3"
Invoke-Command -ComputerName $servers -FilePath .\winrm_setup_for_ansible.ps1
```

## 📚 Related Resources

- **percona_client_install_ansible**: Database automation with Ansible
- **Misc_Ansible_Playbooks**: General Ansible automation
- **SysAdmin-Shell-Scripts**: Linux system administration utilities

## 🤝 Contributing

This is a single-purpose utility script. For Windows automation improvements, consider creating comprehensive Ansible playbooks.

## 📝 License

PowerShell utilities - Free to use and modify.

---

**PowerShell WinRM Setup** - Enabling cross-platform automation by connecting Windows systems to Ansible orchestration.