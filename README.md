# Active Directory Home Lab

## Overview
A home lab simulating an enterprise Active Directory environment built from scratch using Windows Server 2025 and VirtualBox.
The OU structure is modeled after Tranquilitics, a workplace stress intelligence platform and MBS capstone project. The company uses millimeter-wave sensing technology to passively detect employee stress levels in enterprise environments. The AD environment simulates the IT infrastructure such company would need to manage its workforce.

## Environment
- Host Machine: HP EliteBook (Intel Core Ultra 7, 32GB RAM)
- Virtualization: Oracle VirtualBox
- Server OS: Windows Server 2025
- Domain: corp.local
- NetBIOS: CORP

## What Was Built
- Promoted Windows Server 2025 to Domain Controller
- Configured Active Directory Domain Services (AD DS)
- Created company OU structure based on a simulated enterprise (Tranquilitics)

## OU Structure
- Tranquilitics/IT
- Tranquilitics/Engineering
- Tranquilitics/Operations
- Tranquilitics/Sales
- Tranquilitics/Finance
- Tranquilitics/Legal
- Tranquilitics/HR

## Users Created
14 domain users across all departments with onboarding flags set (must change password on first login)

## Skills Demonstrated
- Active Directory administration
- Domain Controller promotion
- OU design and user management
- Windows Server 2025 configuration

## Group Policy Objects (GPOs)
Created GPO: Tranquilitics-Security-Baseline applied to the Tranquilitics OU

Password Policy (referencing NIST SP 800-63B):
- Minimum password length: 13 characters
- Maximum password age: 90 days
- Minimum password age: 30 days
- Password history: 10 passwords remembered
- Complexity requirements: Enabled

Account Lockout Policy:
- Lockout threshold: 5 invalid attempts
- Lockout duration: 30 minutes
- Reset counter after: 30 minutes
- Administrator account lockout: Enabled
Screen Lock Policy:
- Screen saver enabled with 10-minute timeout
- Password required on screen saver resume

## Network Reconnaissance & Log Analysis

### Attack Simulation (Kali Linux)
- Ran Nmap service version scan against the Domain Controller
- Command: `nmap -sV 192.168.10.10`
- Discovered 12 open ports, including DNS (53), Kerberos (88), LDAP (389), SMB (445), and WinRM (5985)
- Nmap fingerprinted the domain as corp.local and identified the host as a Windows Domain Controller

### Defender Response (Windows Server 2025)
- Identified that Windows Filtering Platform auditing was not enabled by default
- Enabled audit logging via PowerShell: 'auditpol /set /subcategory:"Filtering Platform Connection" /success:enable /failure:enable'
- Re-ran Nmap scan and captured 107 Event ID 5156 entries in the Security log
- Events confirmed source IP 192.168.10.20 (Kali) probing multiple ports in rapid succession
- Identified lsass.exe involvement when Kali probed port 464 (Kerberos password change)

### Key Concepts Demonstrated
- Reconnaissance is the first step of an attack chain; Nmap is a standard tool for mapping attack surface
- Windows does not log everything by default; audit policy must be deliberately configured
- Event ID 5156 (Filtering Platform Connection) captures network-level reconnaissance
- SIEM tools like Splunk correlate these events automatically in enterprise environments

## Brute Force Attack Simulation & Detection

### Attack Simulation (Kali Linux)
- Used CrackMapExec to simulate a brute force credential attack against the Domain Controller
- Targeted user account: pshava (Finance department)
- Protocol: WinRM (port 5985)
- Fired multiple failed authentication attempts using incorrect passwords

### Defender Response (Windows Server 2025)
- Enabled Logon auditing via PowerShell: `auditpol /set /subcategory:"Logon" /success:enable /failure:enable`
- Captured 6 Event ID 4625 (Failed Logon) entries in the Security log
- Events confirmed attacker source IP 192.168.10.20 (Kali), workstation name KALI, targeting corp.local\pshava
- Logon Type 3 identified as a remote network logon attempt
- Failure reason logged: Unknown user name or bad password

## Successful Breach Simulation & Detection

### Attack Simulation (Kali Linux)
- Used CrackMapExec to authenticate successfully via WinRM after brute force attempts
- Command: `crackmapexec winrm 192.168.10.10 -u pshava -p P@ssword123!`
- Result: [+] corp.local\pshava:P@ssword123! (Pwn3d!)

### Misconfigurations That Enabled the Breach
- pshava (Finance department) was added to Remote Management Users group
- Account had Elevated Token privileges despite being a non-IT user
- Both represent violations of least privilege principle

### Defender Response (Windows Server 2025)
- Captured Event ID 4624 (Successful Logon) confirming the breach
- Log showed: Account Name: pshava, Workstation Name: KALI, Source IP: 192.168.10.20
- Logon Type 3 confirmed remote network authentication
- Elevated Token: Yes flagged as additional misconfiguration

### Key Concepts Demonstrated
- A successful breach requires both valid credentials AND proper access permissions
- Least privilege violations turn a stolen password into a full remote access incident
- Event ID 4624 following a cluster of 4625s is a brute force success pattern
- KALI workstation name and 192.168.10.20 source IP provide full attacker attribution
### Key Concepts Demonstrated
- Brute force attacks generate repeated 4625 events from the same source IP targeting the same account
- Windows logs the attacker's IP, machine name, and targeted account automatically
- Logon auditing must be explicitly enabled; it is not on by default
- In enterprise environments, SIEM tools are able to correlate these events and fire automated alerts on this pattern.

## Post-Exploitation Enumeration

### Tools Used
- Evil-WinRM - interactive shell over WinRM from Kali into the Domain Controller

### Actions Taken as Compromised User (pshava)
- Established remote shell on DC: `evil-winrm -i 192.168.10.10 -u pshava -p P@ssword123!`
- Confirmed identity: `corp\pshava`
- Enumerated account privileges via `whoami /priv`
- Pulled full AD account profile via `net user pshava /domain`
- Identified sole Domain Admin account (Administrator)
- Mapped WinRM-accessible accounts via `net localgroup "Remote Management Users"`
- Dumped full domain user list via `net user /domain`

### Key Findings
- pshava had SeMachineAccountPrivilege - the ability to join rogue machines to the domain
- Only one Domain Admin exists (Administrator) - target for privilege escalation
- krbtgt account identified - high-value target for Golden Ticket attacks
- All 14 Tranquilitics user accounts were enumerated and are available for password spraying

### Key Concepts Demonstrated
- Post-exploitation is about gathering information to move deeper into the network
- A non-admin compromised account can still reveal critical domain intelligence
- krbtgt compromise enables Golden Ticket attacks - permanent, undetectable domain access
- Least privilege violations at the access level enable this entire attack chain
