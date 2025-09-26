# WN10-CC-000370
# 🔐 Vulnerability Management Lab – WN10-CC-000370

**Control Title:** The Convenience PIN for Windows 10 Must Be Disabled  
**STIG ID:** WN10-CC-000370  
**Compliance Frameworks:** DISA STIG, HIPAA, NIST 800-53, CMMC  
**Lab Environment:** Azure, Tenable.sc / Nessus, PowerShell  
**Vulnerability Category:** Authentication Hardening  
**Remediation Type:** Registry Modification

---

## 🎯 Objective

This lab demonstrates how to simulate, detect, and remediate a non-compliant configuration where **Windows 10 convenience PIN login is enabled** — a setting that reduces authentication security and violates several compliance baselines.

You will use Tenable to scan the system, then remediate using PowerShell, followed by a verification scan.

---

## 📁 Table of Contents

1. [Azure VM Setup](#azure-vm-setup)  
2. [Vulnerability Implementation](#vulnerability-implementation)  
3. [Tenable Scan Configuration](#tenable-scan-configuration)  
4. [Initial Scan Results](#initial-scan-results)  
5. [Remediation via PowerShell](#remediation-via-powershell)  
6. [Verification Steps](#verification-steps)  
7. [Security Rationale](#security-rationale)  
8. [Post-Lab Cleanup](#post-lab-cleanup)  
9. [Appendix: PowerShell Commands](#appendix-powershell-commands)

---

## ☁️ Azure VM Setup

### 🔹 VM Parameters

| Parameter         | Value                          |
|------------------|-------------------------------|
| VM Name          | `Win10-STIGLab-PIN`            |
| OS Image         | Windows 10 Pro (Gen2)          |
| VM Size          | Standard D2s v3                |
| Resource Group   | `vm-lab-disablepin`            |
| Region           | Closest Azure region           |

### 🔹 Security Best Practices

- ✅ **Use strong credentials** (avoid `labuser/Cyberlab123!`)  
- 🔐 Configure **NSG rules**:
  - Allow **RDP** (TCP 3389)
  - Optionally allow **WinRM** (TCP 5985) for remote scanning
  - Block all unnecessary ports

---

## 🔧 VM Configuration for Tenable

1. **Disable Windows Firewall**
   - Run `wf.msc` → Disable all profiles (Domain, Private, Public)

2. **Enable Remote Registry Access**

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "LocalAccountTokenFilterPolicy" -Value 1 -Type DWord -Force
```

📸 **Screenshot Placeholder:** `Screenshot_01_Azure_VM_Config.png`

---

## ⚙️ Vulnerability Implementation

### 🔸 Description

The **convenience PIN login** feature in Windows 10 allows users to authenticate using a short numeric code instead of a secure password. This introduces risk in enterprise and government environments.

### 🔸 Simulate Vulnerability

```powershell
# Enable convenience PIN login (non-compliant)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "DisablePIN" -Value 0
```

📸 **Screenshot Placeholder:** `Screenshot_02_PIN_Enabled.png`

---

## 🔍 Tenable Scan Configuration

Use the **Advanced Network Scan** template in Tenable with the following settings:

### 🔹 Settings Overview

| Category         | Value |
|------------------|-------|
| Credentials      | Local Windows admin (with RDP or WinRM access) |
| Plugins          | STIG Compliance, Windows Local Security |
| Audit File       | `Windows 10 STIG v3r4.audit` |
| Services Required| Remote Registry, Admin Shares, Server Service |
| Discovery        | ICMP, TCP full scan, fast port discovery |

📸 **Screenshot Placeholder:** `Screenshot_03_Tenable_Scan_Settings.png`

---

## 🧪 Initial Scan Results

| STIG ID         | WN10-CC-000370 |
|------------------|----------------|
| Status           | ❌ Fail        |
| Description      | PIN sign-in is enabled |
| Detected Value   | `DisablePIN = 0` |
| Required Value   | `DisablePIN = 1` |

📸 **Screenshot Placeholder:** `Screenshot_04_Scan_Result_Fail.png`

---

## 🛠️ Remediation via PowerShell

### 🔸 Apply Secure Configuration

```powershell
# Disable PIN sign-in (STIG compliant)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "DisablePIN" -Value 1
```

📸 **Screenshot Placeholder:** `Screenshot_05_PIN_Remediation_PowerShell.png`

> ℹ️ A system reboot may be required to enforce the policy.

---

## ✅ Verification Steps

### 🔸 Confirm Registry Setting

```powershell
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name "DisablePIN"
```

**Expected Output:**
```
DisablePIN : 1
```

📸 **Screenshot Placeholder:** `Screenshot_06_Verify_PIN_Disabled.png`

### 🔸 Re-run Tenable Scan

| Control | Result |
|---------|--------|
| WN10-CC-000370 | ✅ Pass |

📸 **Screenshot Placeholder:** `Screenshot_07_Scan_Result_Pass.png`

---

## 🔐 Security Rationale

Allowing users to authenticate using a convenience PIN:

- Reduces entropy and overall password strength
- May bypass enterprise identity policies
- Conflicts with multi-factor enforcement

### Compliance Alignment

| Framework      | Control Reference                          |
|----------------|---------------------------------------------|
| **DISA STIG**  | WN10-CC-000370                              |
| **HIPAA**      | §164.312(a)(2)(i) – Unique user identification |
| **NIST 800-53**| IA-2, IA-5 – Authenticator management        |
| **CMMC**       | AC.1.001 – Limit access to authorized users |

---

## 🧼 Post-Lab Cleanup

1. 🧹 Delete the Azure Resource Group:
```bash
az group delete --name vm-lab-disablepin --yes --no-wait
```

2. ❌ Remove cached credentials from Tenable or local tools

3. 🔄 Reboot the system to enforce registry policies

---

## 📎 Appendix: PowerShell Commands

| Task                | Command |
|---------------------|---------|
| Simulate PIN Enabled | `Set-ItemProperty -Path HKLM:\...\System -Name DisablePIN -Value 0` |
| Remediate (Disable)  | `Set-ItemProperty -Path HKLM:\...\System -Name DisablePIN -Value 1` |
| Verify               | `Get-ItemProperty -Path HKLM:\...\System -Name DisablePIN` |
| Enable Remote Access | `Set-ItemProperty -Path HKLM:\...\System -Name LocalAccountTokenFilterPolicy -Value 1` |

---

✅ **Lab Complete**  
You have successfully identified and remediated a vulnerable configuration related to **convenience PIN authentication** using Azure, Tenable, and PowerShell in a STIG-aligned compliance workflow.

