# Filip.DHCP

**Version:** 0.2.0  
**Application:** DHCP server detector / rogue DHCP helper  
**Executable:** `filip-dhcp.exe`  
**Executable SHA256:** `6f6b002f41ba21e8e597fca3d6b7ba0815c2e9fefedc4504476372abbc78f7f9`  
**Release date:** 2026-06-24  
**License:** MIT License  
**Runtime:** Windows command-line application, no Npcap required in current lightweight mode

> This SHA256 value is used by the application at startup. When `README.md` is placed in the same folder as `filip-dhcp.exe`, the application reads this document and compares the stored SHA256 value with the current executable file. If the file was modified after release packaging, the application prints a checksum warning.

---

## Purpose

Filip.DHCP is a small Windows command-line utility for detecting DHCP servers visible from a selected network adapter. Its main purpose is to help identify possible rogue or conflicting DHCP servers on a local network segment.

The application sends DHCPDISCOVER packets and waits for DHCPOFFER/DHCPACK replies. If more than one DHCP server responds, the output highlights a possible DHCP conflict.

---

## Important network limitation

DHCP is normally a broadcast-domain protocol. The standard detection mode checks DHCP servers reachable from the selected local adapter/VLAN.

The application also includes experimental unicast scanning modes, but these are best-effort only and may miss valid DHCP servers. They are useful for quick diagnostics, not as a guaranteed routed-network DHCP discovery method.

Current lightweight version does **not** require Npcap, Nmap, or a packet-capture driver.

---

## Files in release package

Recommended release folder:

```text
filip-dhcp.exe
README.md
```

Keep `README.md` next to `filip-dhcp.exe` if you want offline checksum verification at startup.

---

## Checksum verification

At startup the application prints the current SHA256 of the executable and verification status.

Possible statuses:

```text
Checksum verification: OK (README.md)
```

The executable SHA256 matches the hash stored in this README.

```text
Checksum verification: FAILED - README.md hash does not match executable
```

The executable was rebuilt, replaced, corrupted, or modified after this README was generated.

```text
Checksum verification: not checked
```

The README file is missing, unreadable, or does not contain a SHA256 hash.

After every new release build, regenerate this README or update the SHA256 value in the header.

---

## Basic usage

Run from Command Prompt or PowerShell:

```cmd
filip-dhcp.exe
```

The application displays the detected current network adapter and asks whether it should use it for DHCP detection.

If UDP port 68 binding fails, run the program from an elevated terminal as Administrator.

---

## Command-line parameters

| Parameter | Description |
|---|---|
| `/help`, `--help` | Shows help text, supported parameters, and examples. |
| `/adapter:<name>` | Uses a specific network adapter by name or description. Example: `/adapter:Ethernet` or `/adapter:eth4`. |
| `/timeout:<seconds>` | Sets DHCP response wait timeout. Default is 5 seconds. |
| `/yes`, `--yes`, `-y` | Non-interactive mode. Uses defaults without asking questions. |
| `/range:<start-ip>-<end-ip>` | Experimental scan of a custom IP range. Example: `/range:192.168.0.1-192.168.1.255`. |
| `/experimental` | Experimental scan of all RFC1918 private ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16. |

---

## Examples

Show help:

```cmd
filip-dhcp.exe /help
```

Use the current/default adapter interactively:

```cmd
filip-dhcp.exe
```

Use a specific adapter:

```cmd
filip-dhcp.exe /adapter:Ethernet
```

Use a specific adapter with longer timeout:

```cmd
filip-dhcp.exe /adapter:Ethernet /timeout:10
```

Run non-interactively:

```cmd
filip-dhcp.exe /yes
```

Scan a smaller custom range using experimental unicast mode:

```cmd
filip-dhcp.exe /adapter:Ethernet /range:192.168.0.1-192.168.1.255
```

Scan all private ranges using experimental mode:

```cmd
filip-dhcp.exe /adapter:Ethernet /experimental
```

---

## Adapter selection and validation

When `/adapter:<name>` is used, the application validates that the adapter exists and is usable.

The adapter must have:

- a usable IPv4 address,
- a MAC address,
- non-loopback network interface information.

If the adapter is missing, disabled, disconnected, or has no usable IPv4/MAC address, the program exits with an error similar to:

```text
Error: adapter 'eth4' was not found, has no IPv4/MAC address, or is not usable/up
```

---

## Detection modes

### Default broadcast mode

Default mode sends DHCPDISCOVER broadcast on the selected adapter and waits for DHCP replies.

This is the recommended mode for normal DHCP conflict detection on the current VLAN/network segment.

### Custom range mode

Enabled by:

```cmd
/range:192.168.0.1-192.168.1.255
```

This mode sends best-effort unicast DHCPINFORM packets to UDP/67 on every IP in the selected interval and listens for replies on UDP/68. The start IP must be lower than or equal to the end IP.

### Experimental private-range mode

Enabled by:

```cmd
/experimental
```

This sends best-effort unicast DHCPINFORM packets to UDP/67 and listens for replies on UDP/68 while scanning the RFC1918 private ranges:

- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

This can be very slow and should be used carefully.

---

## Output summary

The application prints:

- application version,
- build commit,
- executable SHA256,
- checksum verification status,
- selected adapter information,
- detection mode,
- received DHCP offers,
- possible conflict warning when multiple DHCP servers respond.

At the end, in an interactive terminal, the application waits for:

```text
Press any key to exit . . .
```

---

## Build from source

Requirements:

- Rust toolchain with Cargo
- Windows target environment

Build release binary:

```cmd
cargo build --release
```

Release executable:

```text
target\release\filip-dhcp.exe
```

After building, calculate the SHA256 of the release executable and update this README header.


---

## MSI installer

This release also includes:

```text
Filip.DHCP-0.2.0-x64.msi
```

The MSI installs:

- `filip-dhcp.exe` into `Program Files\Filip.DHCP`,
- `README.md`,
- `LICENSE.txt`,
- `SAPIENTECH-Local-CodeSigning.cer`,
- Start Menu shortcut `Filip.DHCP`.

During installation it also imports the local SAPIENTECH code-signing certificate into the local machine certificate stores:

```text
Trusted Root Certification Authorities
Trusted Publishers
```

The certificate import is used only for trusting this local/self-signed SAPIENTECH publisher certificate. If the certificate is already installed, Windows/certutil reports it as already present and keeps the existing trust entry.

The MSI itself is signed with the same certificate:

```text
CN=SAPIENTECH Local Code Signing
Thumbprint: D4CAA00550339EDFBDF791D76BD76D3A94A75AE7
```


---

## SIEM / logging

Filip.DHCP can write an informational detection summary after each run. This is useful for Windows Event Log collection, Wazuh agents, NetXMS/syslog pipelines, or other SIEM tools.

Windows Application log:

```cmd
filip-dhcp.exe /adapter:Ethernet /yes /eventlog
```

Syslog over UDP, default port 514:

```cmd
filip-dhcp.exe /adapter:Ethernet /yes /syslog:192.168.0.10
```

Syslog over UDP, custom port:

```cmd
filip-dhcp.exe /adapter:Ethernet /yes /syslog:192.168.0.10:5514
```

Combined Windows Event Log and syslog:

```cmd
filip-dhcp.exe /adapter:Ethernet /yes /eventlog /syslog:192.168.0.10:514
```

The Windows event is written to the Application log with source `Filip.DHCP`, type `Information`, and event ID `1000`. The MSI creates the `Filip.DHCP` event source during installation. Writing to Windows Event Log may require the app to run elevated or under a service/system context depending on local policy.

The syslog message is sent as UDP local0.info and contains a compact key/value summary, for example:

```text
Filip.DHCP DHCP detection status=OK_ONE_DHCP_SERVER adapter=Ethernet method="DHCPDISCOVER broadcast (UDP socket mode, no Npcap)" offers=1 servers=192.168.0.254
```

---

## Digital signature

The executable is signed with a local code-signing certificate:

```text
CN=SAPIENTECH Local Code Signing
Thumbprint: D4CAA00550339EDFBDF791D76BD76D3A94A75AE7
```

This is a local/self-signed certificate intended for internal use. On another Windows computer, install `SAPIENTECH-Local-CodeSigning.cer` into Trusted Root Certification Authorities and Trusted Publishers if Windows does not already trust the publisher.

---

## License

MIT License

Copyright (c) 2026 SAPIENTECH / Filip.DHCP contributors

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files, to deal in the software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the software, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
