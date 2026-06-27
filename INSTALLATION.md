# Filip.DHCP Installation Guide

**Application:** Filip.DHCP  
**Version:** 0.2.0  
**Installer:** `Filip.DHCP-0.2.0-x64.msi`  
**Target OS:** Windows 10/11 x64, Windows Server x64  
**Publisher certificate:** `CN=SAPIENTECH Local Code Signing`  
**Certificate thumbprint:** `D4CAA00550339EDFBDF791D76BD76D3A94A75AE7`  
**License:** MIT License

---

## 1. What is Filip.DHCP?

Filip.DHCP is a small Windows command-line utility for detecting DHCP servers visible from a selected network adapter.

It can be used to check whether a network has:

- one expected DHCP server,
- more than one DHCP server,
- a possible rogue/conflicting DHCP server,
- DHCP responses from a specific custom IP range.

The current lightweight version does **not** require Npcap, Nmap, or a packet capture driver.

---

## 2. Recommended installation file

For normal users and domain deployment use only this file:

```text
Filip.DHCP-0.2.0-x64.msi
```

The MSI is a self-contained Windows Installer package. It already contains all required payload files.

You do **not** need to copy or upload these files separately for installation:

```text
filip-dhcp.exe
README.md
LICENSE.txt
SAPIENTECH-Local-CodeSigning.cer
```

They are embedded in the MSI and installed automatically.

---

## 3. What the MSI installs

The MSI installs the application into:

```text
C:\Program Files\Filip.DHCP
```

Installed files:

```text
filip-dhcp.exe
README.md
LICENSE.txt
SAPIENTECH-Local-CodeSigning.cer
```

It also creates a Start Menu shortcut:

```text
Start Menu → Filip.DHCP → Filip.DHCP
```

---

## 4. Digital signature and certificate information

The application and MSI are signed with a local/internal SAPIENTECH code-signing certificate:

```text
Subject:     CN=SAPIENTECH Local Code Signing
Thumbprint:  D4CAA00550339EDFBDF791D76BD76D3A94A75AE7
Type:        self-signed/internal code-signing certificate
```

This certificate is intended for internal/company use. It is not a public CA certificate like DigiCert, Sectigo, or GlobalSign.

### What the installer does with the certificate

During installation, the MSI imports the certificate into the local computer certificate stores:

```text
LocalMachine\Root
LocalMachine\TrustedPublisher
```

In Windows UI these stores correspond approximately to:

```text
Trusted Root Certification Authorities
Trusted Publishers
```

This allows Windows to trust the SAPIENTECH local publisher certificate on that computer.

If the certificate is already installed, the import is effectively skipped/kept as-is by Windows/certutil. The installer does not need to install a duplicate certificate.

### Important security note

Installing a certificate into Trusted Root/Trusted Publishers is a trust decision. Only install this MSI or certificate if it came from a trusted SAPIENTECH source.

For public distribution to unknown third-party users, a paid public code-signing certificate is recommended instead of a self-signed internal certificate.

---

## 5. Interactive installation

Double-click:

```text
Filip.DHCP-0.2.0-x64.msi
```

or run from Command Prompt:

```cmd
msiexec /i Filip.DHCP-0.2.0-x64.msi
```

Because the installer writes to `Program Files` and imports a machine-level certificate, administrator rights/UAC approval are required.

---

## 6. Silent installation

For silent local installation from an elevated Command Prompt:

```cmd
msiexec /i Filip.DHCP-0.2.0-x64.msi /qn /norestart
```

Silent installation with detailed log:

```cmd
msiexec /i Filip.DHCP-0.2.0-x64.msi /qn /norestart /L*v C:\Windows\Temp\Filip.DHCP-install.log
```

Recommended for troubleshooting:

```cmd
msiexec /i Filip.DHCP-0.2.0-x64.msi /passive /norestart /L*v C:\Windows\Temp\Filip.DHCP-install.log
```

`/passive` shows progress but does not require user interaction.

---

## 7. Silent uninstallation

Uninstall using the MSI file:

```cmd
msiexec /x Filip.DHCP-0.2.0-x64.msi /qn /norestart
```

Uninstall with log:

```cmd
msiexec /x Filip.DHCP-0.2.0-x64.msi /qn /norestart /L*v C:\Windows\Temp\Filip.DHCP-uninstall.log
```

Note: certificate removal policy should be decided by administrators. In many environments, trusted publisher/root certificates are managed separately by GPO and are not automatically removed with individual application uninstallers.

---

## 8. Domain / network deployment

The MSI can be deployed silently across a Windows domain.

Common deployment options:

- Group Policy Software Installation,
- Microsoft Intune,
- Microsoft Configuration Manager/SCCM,
- PDQ Deploy,
- PsExec or other remote execution tools,
- startup script running under computer/system context.

### Example: install from a network share

Put the MSI on a UNC path accessible by computers:

```text
\\server\software\Filip.DHCP\Filip.DHCP-0.2.0-x64.msi
```

Silent install command:

```cmd
msiexec /i "\\server\software\Filip.DHCP\Filip.DHCP-0.2.0-x64.msi" /qn /norestart /L*v C:\Windows\Temp\Filip.DHCP-install.log
```

### Group Policy Software Installation

Recommended path:

```text
Computer Configuration
→ Policies
→ Software Settings
→ Software installation
→ New Package
```

Use a UNC path, for example:

```text
\\server\software\Filip.DHCP\Filip.DHCP-0.2.0-x64.msi
```

Recommended assignment type:

```text
Assigned to computers
```

Computer assignment is preferred because the installer writes to `Program Files` and imports a machine-level certificate.

---

## 9. Recommended enterprise certificate deployment

Although the MSI can import the local SAPIENTECH certificate itself, in a domain environment the cleaner approach is to deploy the certificate centrally via GPO.

Recommended GPO stores:

```text
Computer Configuration
→ Policies
→ Windows Settings
→ Security Settings
→ Public Key Policies
→ Trusted Root Certification Authorities
```

and:

```text
Computer Configuration
→ Policies
→ Windows Settings
→ Security Settings
→ Public Key Policies
→ Trusted Publishers
```

Deploy this certificate file:

```text
SAPIENTECH-Local-CodeSigning.cer
```

After the certificate is trusted via GPO, the MSI can be deployed normally and Windows should recognize the publisher certificate.

---

## 10. Verifying the installation

After installation, verify files:

```cmd
dir "C:\Program Files\Filip.DHCP"
```

Expected files:

```text
filip-dhcp.exe
README.md
LICENSE.txt
SAPIENTECH-Local-CodeSigning.cer
```

Run help:

```cmd
"C:\Program Files\Filip.DHCP\filip-dhcp.exe" /help
```

Run a basic DHCP check:

```cmd
"C:\Program Files\Filip.DHCP\filip-dhcp.exe" /adapter:Ethernet /timeout:5
```

Run a custom range check:

```cmd
"C:\Program Files\Filip.DHCP\filip-dhcp.exe" /adapter:Ethernet /range:192.168.0.1-192.168.1.255 /timeout:10
```

---

## 11. Verifying the digital signature

PowerShell:

```powershell
Get-AuthenticodeSignature "C:\Program Files\Filip.DHCP\filip-dhcp.exe"
```

Expected status:

```text
Status : Valid
```

Expected signer:

```text
CN=SAPIENTECH Local Code Signing
```

Check MSI signature before installation:

```powershell
Get-AuthenticodeSignature ".\Filip.DHCP-0.2.0-x64.msi"
```

Expected status:

```text
Status : Valid
```

---

## 12. Verifying the installed certificate

Check Trusted Root:

```powershell
Get-ChildItem Cert:\LocalMachine\Root | Where-Object Thumbprint -eq "D4CAA00550339EDFBDF791D76BD76D3A94A75AE7"
```

Check Trusted Publishers:

```powershell
Get-ChildItem Cert:\LocalMachine\TrustedPublisher | Where-Object Thumbprint -eq "D4CAA00550339EDFBDF791D76BD76D3A94A75AE7"
```

If both commands return the certificate, the local SAPIENTECH publisher certificate is trusted on that computer.

---

## 13. Application usage examples

Show help:

```cmd
filip-dhcp.exe /help
```

Use automatic/current adapter:

```cmd
filip-dhcp.exe
```

Use a specific adapter:

```cmd
filip-dhcp.exe /adapter:Ethernet
```

Set timeout:

```cmd
filip-dhcp.exe /adapter:Ethernet /timeout:10
```

Run without interactive prompts:

```cmd
filip-dhcp.exe /yes
```

Scan custom DHCP server IP range:

```cmd
filip-dhcp.exe /adapter:Ethernet /range:192.168.0.1-192.168.1.255 /timeout:10
```

Experimental private-range scan:

```cmd
filip-dhcp.exe /adapter:Ethernet /experimental
```

Note: `/experimental` and `/range` are best-effort modes. DHCP is normally a broadcast-domain protocol, so these modes may not detect every DHCP server across routed networks/VLANs.

---

## 14. Troubleshooting

### Windows still warns about publisher

Possible causes:

- the certificate was not installed into Trusted Root and Trusted Publishers,
- the MSI was not run elevated,
- domain policy blocks self-signed publishers,
- Smart App Control / WDAC policy requires a public/reputable certificate.

Verify certificate stores using the commands in section 12.

### Silent install fails

Run with full log:

```cmd
msiexec /i Filip.DHCP-0.2.0-x64.msi /qn /norestart /L*v C:\Windows\Temp\Filip.DHCP-install.log
```

Then inspect:

```text
C:\Windows\Temp\Filip.DHCP-install.log
```

### DHCP check finds no servers

Check:

- correct adapter name,
- whether the adapter has IPv4 address,
- whether Windows Firewall/security tools block UDP 67/68,
- whether the DHCP server is in the same broadcast domain,
- whether `/range` should be used for a known server IP.

---

## 15. Files for administrators

Recommended release package for administrators:

```text
Filip.DHCP-0.2.0-x64.msi
INSTALLATION.md
```

Optional, if certificate is distributed separately via GPO:

```text
SAPIENTECH-Local-CodeSigning.cer
```

For basic user installation, only the MSI is required.

