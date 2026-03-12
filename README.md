# Active-Directory-Penetration-Testing-Lab

## 🛡️ Project Overview
This repository contains a comprehensive technical write-up of a penetration test performed on a complex, multi-domain Active Directory forest environment. The simulation follows a "Black Box" approach, starting from an unauthenticated internal position to achieving full Domain Admin privileges across three interconnected domains: `poudlard.local`, `ministere.local`, and `mangemorts.local`.

## 🏗️ Lab Infrastructure & Architecture
The environment was built to simulate a production-grade infrastructure with trust relationships and modern services:
- **Domains**: 3 Interconnected domains with bi-directional trusts.
- **Operating Systems**: Windows Server 2012 R2, 2016, and Windows 10.
- **Core Services**: ADDS (Active Directory Domain Services), ADCS (Certificate Services), DNS, SMBv1/v2, LDAP/S.
- **Attacker Platform**: Exegol (Kali-based Linux environment).



## 🛠️ Tools & Technologies Used
- **Reconnaissance**: `NetExec`, `Nmap`, `BloodHound/SharpHound`.
- **Exploitation & Pivoting**: `Impacket` Suite, `Metasploit` (EternalBlue), `PetitPotam`.
- **Credential Attacks**: `Hashcat` (Kerberoasting), `Mimikatz` (LSASS memory dump).
- **ADCS Attacks**: `Certipy` (ESC1 & Shadow Credentials).

## 🚀 The Kill Chain (Executive Summary)

### 1. Initial Access & Domain Compromise (`poudlard.local`)
- **Vulnerability**: Anonymous LDAP Bind enabled and weak password policy.
- **Action**: Discovered credentials in object descriptions. Performed **Kerberoasting** on service accounts and cracked the hash with `Hashcat`.
- **Privilege Escalation**: Leveraged `Shadow Credentials` via `AddKeyCredentialLink` ACLs to take control of a high-privilege account.

### 2. Lateral Movement & Pivot (`ministere.local`)
- **Vulnerability**: Bi-directional domain trust and legacy protocols (SMBv1).
- **Action**: Exploited **MS17-010 (EternalBlue)** on a standalone server to gain SYSTEM access.
- **Findings**: Recovered clear-text administrative credentials from forgotten configuration files (`srv-logs` credentials).

### 3. ADCS Exploitation & Domain Takeover
- **Vulnerability**: Misconfigured Certificate Template (ESC1) allowing `EnrolleeSuppliesSubject`.
- **Action**: Used **PetitPotam** to coerce NTLM authentication from a server, relayed it to the ADCS web enrollment portal via `ntlmrelayx`, and forged a certificate for the target user.

### 4. Forest Takeover (`mangemorts.local`)
- **Vulnerability**: Bi-directional Forest Trust and weak account isolation.
- **Action**: 
    - Performed **SID History** exploitation or used cross-domain credentials recovered from `srv-logs`.
    - Executed a **DCSync** attack using `secretsdump.py` to dump the KRBTGT hash of the final domain.
- **Persistence**: Forged a **Golden Ticket** using `ticketer.py` with the AES256 key of the KRBTGT account, granting permanent Domain Admin rights.

---

## 🔧 Engineering Insights & Remediation (Hardening)
*Here are the strategic fixes implemented in this lab to secure the infrastructure:*

### **1. Identity & Access Management (IAM)**
- **Disable Anonymous LDAP Binds**: Ensure `dsHeuristics` is configured to prevent unauthenticated enumeration.
- **Tiered Administrative Model**: Implementation of "Tier 0/1/2" to ensure Domain Admins never log into lower-security workstations (preventing LSASS dumping).
- **GMSA Deployment**: Migrated classic service accounts to **Group Managed Service Accounts (gMSA)** with automatic 256-bit password rotation to mitigate Kerberoasting.

### **2. Network & Protocol Security**
- **Decommission Legacy Protocols**: Complete disabling of SMBv1 (to block EternalBlue) and LLMNR/NBT-NS (to block spoofing/relay).
- **Enforce SMB Signing**: Configured GPOs to require SMB signing on all servers to prevent NTLM Relay attacks.

### **3. ADCS Hardening**
- **Template Security**: Disabled `EnrolleeSuppliesSubject` on all certificate templates.
- **Extended Protection for Authentication (EPA)**: Enabled EPA and required SSL for ADCS Web Enrollment to prevent NTLM relaying to HTTP.

### **4. Advanced Persistence & Forest Trust Defense**
- **Monitor Replication Requests**: Configured IDS/SIEM to alert on `DS-Replication-Get-Changes-All` events originating from non-Domain Controller IP addresses (Detection of DCSync).
- **KRBTGT Password Rotation**: Implemented a mandatory bi-annual reset of the **KRBTGT account password** (twice to invalidate old Golden Tickets).
- **Selective Authentication**: Hardened Forest Trusts by enabling "Selective Authentication" to limit the scope of accessible resources across domains.

---

## 📄 Full Report
The detailed step-by-step write-up (PDF) including terminal captures and exploit explanations is available in this repository.

> **Disclaimer**: This project was conducted in a private, isolated laboratory environment for educational purposes only.
