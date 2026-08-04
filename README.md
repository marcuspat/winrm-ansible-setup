# PowerShell WinRM Setup for Ansible

A single PowerShell script that prepares a Windows host for Ansible management over WinRM.

## What it actually does

`winrm_setup_for_ansible.ps1` downloads and runs Ansible's own upstream `ConfigureRemotingForAnsible.ps1` script from the `ansible/ansible` GitHub repo, then executes it with `-ExecutionPolicy ByPass`. That's the entire script:

```powershell
$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"
(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
powershell.exe -ExecutionPolicy ByPass -File $file
```

All WinRM listener setup, firewall rules, certificate generation, and authentication configuration happen inside that upstream Ansible script, not in code written in this repo. There is no Active Directory management, IIS configuration, scheduled task orchestration, or PowerShell DSC anywhere here.

## Usage

Run as Administrator on the target Windows host:

```powershell
.\winrm_setup_for_ansible.ps1
```

## Note

An earlier version of this README described WinRM listener setup, firewall rules, and certificate configuration as if they were implemented directly in this script, including fabricated code blocks using `Enable-PSRemoting`, `New-SelfSignedCertificate`, and `New-NetFirewallRule` that don't appear anywhere in the actual file. Corrected 2026-08-04. For what actually configures WinRM, see the real upstream script this repo calls: https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1
