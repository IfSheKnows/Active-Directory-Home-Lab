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

Password Policy (NIST-aligned):
- Minimum password length: 13 characters
- Maximum password age: 90 days
- Minimum password age: 30 days
- Password history: 10 passwords remembered
- Complexity requirements: Enabled

Screen Lock Policy:
- Screen saver enabled with 10-minute timeout
- Password required on screen saver resume
