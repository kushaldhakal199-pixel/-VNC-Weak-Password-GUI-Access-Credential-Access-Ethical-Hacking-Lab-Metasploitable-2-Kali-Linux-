# 🔐 VNC Weak Password → GUI Access → Credential Access & Persistence Analysis

> **Ethical Hacking / Post-Exploitation Lab**  
> **Target:** Metasploitable 2  
> **Attacker:** Kali Linux  
> **Primary Focus:** Credential Access and Persistence Analysis

---

## 📌 Overview

This practical lab demonstrates the security risks associated with an exposed VNC service using an intentionally vulnerable **Metasploitable 2** virtual machine.

The assessment covers:

- VNC service discovery
- VNC authentication
- GUI access
- Local account enumeration
- Credential-access exposure
- SSH persistence indicators
- Authentication-log investigation
- SOC detection
- Security recommendations

> ⚠️ **Lab scope:** The assessment was performed inside an isolated VMware virtual laboratory environment.

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| Attacker | Kali Linux |
| Target | Metasploitable 2 |
| Target IP | `192.168.154.130` |
| Service | VNC |
| Port | `TCP/5900` |
| Protocol | VNC / RFB |
| VNC Server | Xtightvnc |
| Tools | Nmap, VNC Viewer |
| Network | Isolated VMware laboratory |

### Lab Topology

```text
┌─────────────────────┐
│      Kali Linux     │
│     Attacker VM     │
│    192.168.154.x    │
└──────────┬──────────┘
           │
           │ Isolated VMware Network
           ▼
┌─────────────────────┐
│   Metasploitable 2  │
│      Target VM      │
│   192.168.154.130   │
│      VNC :5900      │
└─────────────────────┘
```

### Figure 1 — Lab Environment

![Figure 1 - VMware lab environment](screenshots/01-vmware-lab-environment.png)

---

# 🎯 Objective

The objective was to investigate the security impact of an exposed VNC service and determine what information could become accessible after VNC authentication.

### Attack / Investigation Path

```text
VNC Service Discovery
        ↓
    TCP Port 5900
        ↓
 VNC Authentication
        ↓
     GUI Access
        ↓
 Account Enumeration
        ↓
 Credential Access Analysis
        ↓
 Persistence Analysis
        ↓
 Authentication Log Investigation
        ↓
 Detection & Mitigation
```

---

# 1. 🔎 VNC Service Discovery

The target was identified as:

```text
192.168.154.130
```

VNC uses:

```text
TCP/5900
```

The service was identified from Kali using Nmap.

### Command

```bash
nmap -sV 192.168.154.130
```

### Figure 2 — Nmap showing TCP/5900

![Figure 2 - Nmap VNC scan](screenshots/02-nmap-vnc-port-5900.png)

### Finding

TCP/5900 was exposed and accessible from the Kali testing machine.

---

# 2. 🖥️ VNC Server Identification

Process enumeration on Metasploitable 2 showed that the VNC server was running using **Xtightvnc**.

Relevant process:

```text
Xtightvnc :0
```

The VNC desktop was associated with display `:0`.

### Figure 3 — Xtightvnc Process

![Figure 3 - Xtightvnc process](screenshots/03-xtightvnc-process.png)

---

# 3. 🌐 Confirming Port 5900

The listening network service was also confirmed.

The system showed:

```text
0.0.0.0:5900
```

This indicates that the VNC service was listening on TCP port 5900.

### Figure 4 — TCP/5900 Listening

![Figure 4 - VNC port listening](screenshots/04-vnc-port-listening.png)

---

# 4. 🔑 VNC Authentication

From Kali Linux, the VNC client was launched against the target:

```bash
vncviewer 192.168.154.130:5900
```

The connection produced:

```text
Connected to RFB server, using protocol version 3.3
Performing standard VNC authentication
Password:
```

This confirmed that the VNC service was reachable and protected by VNC authentication.

### Figure 5 — VNC Authentication Prompt

![Figure 5 - VNC authentication](screenshots/05-vnc-authentication.png)

---

# 5. 🖥️ VNC GUI Access

Successful VNC authentication allowed access to the graphical desktop of Metasploitable 2.

This demonstrated the impact of obtaining valid VNC authentication.

### Figure 6 — Successful VNC GUI Access

![Figure 6 - VNC GUI access](screenshots/06-vnc-gui-access.png)

### Finding

An attacker with valid VNC authentication could obtain graphical access to the target system.

---

# 6. 🆔 Post-Access System Identification

After obtaining access, basic system information was examined.

### Commands

```bash
whoami
hostname
id
```

The hostname was:

```text
metasploitable
```

The `msfadmin` account was identified during the assessment.

### Figure 7 — Post-Access System Identification

![Figure 7 - System identification](screenshots/07-post-access-system-info.png)

---

# 7. 📁 Local Home Directory Enumeration

The `/home` directory was examined:

```bash
ls -la /home
```

The assessment identified user directories including:

```text
msfadmin
service
user
ftp
```

### Figure 8 — Local User Home Directories

![Figure 8 - Home directory enumeration](screenshots/08-home-directory-enumeration.png)

### Finding

The VNC session provided visibility into local user accounts and their home directories.

---

# 8. 👤 Local Account Enumeration

The local account database was examined using:

```bash
cat /etc/passwd
```

Examples identified included:

```text
root
msfadmin
postgres
user
service
ftp
www-data
backup
```

### Figure 9 — Local Account Information

![Figure 9 - Local accounts](screenshots/09-local-account-enumeration.png)

### What `/etc/passwd` Provides

The file provides account information such as:

- Username
- UID
- GID
- Home directory
- Login shell

> The `x` field does **not** represent the actual password.

---

# 9. 🧑‍💻 Interactive Login Account Identification

The following command was used:

```bash
awk -F: '$7 ~ /bash|sh$/ {print $1, $7}' /etc/passwd
```

Accounts with interactive shells included:

```text
msfadmin
postgres
user
service
```

Other service accounts used shells such as:

```text
/bin/false
/usr/sbin/nologin
```

### Figure 10 — Interactive Login Accounts

![Figure 10 - Interactive accounts](screenshots/10-interactive-login-accounts.png)

---

# 10. 🔐 Credential Access Analysis

The assessment did **not** extract or crack real passwords.

Instead, credential-access exposure was demonstrated through enumeration of:

- Local usernames
- Interactive accounts
- SSH authentication configuration
- Authentication logs

### Evidence Areas

```text
/etc/passwd
/home/msfadmin/.ssh/
/var/log/auth.log
```

### Figure 11 — Credential-Access Evidence

![Figure 11 - Credential access evidence](screenshots/11-credential-access-evidence.png)

### Finding

The compromise of an interactive VNC session can expose information useful for further authentication-related investigation.

---

# 11. 🔑 SSH Persistence Indicators

The following directory was examined:

```bash
ls -la /home/msfadmin/.ssh
```

The directory contained:

```text
authorized_keys
id_rsa
id_rsa.pub
```

### Figure 12 — Existing SSH Authentication Configuration

![Figure 12 - SSH directory](screenshots/12-ssh-directory.png)

### Explanation

- `authorized_keys` contains public keys permitted for SSH authentication.
- `id_rsa` is a private SSH key.
- `id_rsa.pub` is a public SSH key.

> The private key was **not displayed, copied, or used** during the assessment.

### Finding

The presence of SSH key infrastructure represents a persistence-related area that should be reviewed during an incident investigation.

---

# 12. 🕵️ Backdoor / Persistence Analysis

The assessment investigated existing accounts and SSH configuration for potential persistence indicators.

No new malicious account or persistent backdoor was installed.

### Investigation Areas

```text
/etc/passwd
/home/*/.ssh/
/var/log/auth.log
```

### Investigation Flow

```text
VNC Access
    ↓
Local Account Enumeration
    ↓
SSH Configuration Examination
    ↓
Authentication Log Review
    ↓
Persistence Indicators Identified
```

### Figure 13 — Persistence-Related SSH Configuration

![Figure 13 - Persistence analysis](screenshots/13-persistence-analysis.png)

### Finding

Existing SSH authentication configuration should be reviewed during incident response to determine whether unauthorized keys or accounts have been introduced.

---

# 13. 📋 Authentication Log Investigation

The authentication log was examined using:

```bash
tail -20 /var/log/auth.log
```

The logs showed failed attempts by `msfadmin` to switch to the root account.

Relevant events included:

```text
FAILED su for root by msfadmin
```

and:

```text
pam_authenticate: Authentication failure
```

### Figure 14 — Authentication Failures

![Figure 14 - Authentication log failures](screenshots/14-authentication-log-failures.png)

---

# 14. 🛡️ SOC Analysis

From a SOC analyst perspective, authentication events should be investigated and correlated with other activity.

A notable event was:

```text
FAILED su for root by msfadmin
```

Repeated authentication failures can indicate:

- Incorrect credentials
- Unauthorized privilege attempts
- Potential compromise
- Misconfiguration
- User error

The event should be correlated with:

- Login history
- Source
- Timing
- Other system activity

---

# 15. ⏱️ CRON Activity

The authentication log also contained recurring root CRON sessions:

```text
CRON: session opened for user root
CRON: session closed for user root
```

These events are **not automatically malicious** because scheduled system tasks commonly execute with root privileges.

### Figure 15 — Root CRON Activity

![Figure 15 - CRON activity](screenshots/15-cron-activity.png)

---

# 16. 🧪 Controlled Credential-Harvesting Simulation

Because the purpose of the assessment is to demonstrate credential-access risk, a controlled synthetic credential can be used instead of collecting a real password.

### Synthetic Test Data

```bash
mkdir -p ~/lab-evidence

printf 'LAB_USER=demo\nLAB_PASSWORD=FakeOnly123!\n' > ~/lab-evidence/credential-demo.txt

cat ~/lab-evidence/credential-demo.txt
```

### Figure 16 — Controlled Credential-Access Simulation

![Figure 16 - Synthetic credential simulation](screenshots/16-synthetic-credential-simulation.png)

> ⚠️ The test data is **not a real credential**.

After the demonstration:

```bash
rm ~/lab-evidence/credential-demo.txt
```

### Report Statement

A controlled credential-access simulation was performed using synthetic credentials. No real user passwords were extracted or cracked.

---

# 17. 👤 Controlled Backdoor-User Detection Simulation

A controlled temporary test account can demonstrate how an unexpected account could be identified.

If authorized root access is available:

```bash
useradd lab-backdoor-demo
```

Then enumerate local accounts:

```bash
cut -d: -f1 /etc/passwd
```

The test account should appear:

```text
lab-backdoor-demo
```

### Figure 17 — Detection of Temporary Test Account

![Figure 17 - Test account detection](screenshots/17-test-account-detection.png)

After the demonstration:

```bash
userdel lab-backdoor-demo
```

Then verify that it has been removed:

```bash
cut -d: -f1 /etc/passwd
```

### Report Statement

A temporary test account was used in the isolated laboratory to demonstrate detection of an unexpected local account. The account was removed after testing and no persistent backdoor was installed.

---

# 🔗 Attack Chain Summary

```text
VNC SERVICE
     │
     ▼
TCP PORT 5900
     │
     ▼
VNC AUTHENTICATION
     │
     ▼
GUI ACCESS
     │
     ▼
ACCOUNT ENUMERATION
     │
     ▼
CREDENTIAL ACCESS
ANALYSIS / SIMULATION
     │
     ▼
SSH PERSISTENCE
INDICATOR REVIEW
     │
     ▼
AUTHENTICATION LOG
INVESTIGATION
     │
     ▼
SOC DETECTION &
MITIGATION
```

---

# ⚠️ Security Impact

The assessment demonstrated that exposure of a VNC service can provide an attacker with interactive graphical access after successful authentication.

Once access is obtained, the attacker may be able to:

- View system information
- Identify local accounts
- Identify interactive login accounts
- Examine authentication-related configuration
- Investigate SSH configuration
- Access information available to the compromised user
- Generate authentication events that can be investigated by defenders

---

# 🔍 Detection Recommendations

A SOC should monitor for:

1. Unexpected VNC connections to TCP/5900
2. Repeated VNC authentication failures
3. Unusual login times
4. Unexpected local account creation
5. Changes to `authorized_keys`
6. Unexpected SSH keys
7. Repeated failed privilege-authentication attempts
8. Suspicious authentication events
9. Unexpected processes associated with VNC
10. Unusual root activity

---

# 🛡️ Mitigation Recommendations

Recommended defensive measures include:

- Disable VNC when it is not required.
- Do not expose VNC directly to untrusted networks.
- Restrict TCP/5900 using firewall rules.
- Use strong, unique authentication credentials.
- Prefer secure remote-access mechanisms.
- Restrict administrative access.
- Monitor authentication logs.
- Regularly review local accounts.
- Regularly review SSH `authorized_keys`.
- Remove unused accounts and keys.
- Apply security updates to remote-access software.
- Segment management services from normal user networks.

---

# 📊 Findings Summary

| Finding | Evidence | Risk / Security Meaning |
|---|---|---|
| VNC exposed | TCP/5900 | Unauthorized access opportunity |
| VNC authentication | VNC password prompt | Authentication boundary |
| GUI access | Successful VNC session | Interactive system access |
| Account exposure | `/etc/passwd` | Account enumeration |
| Interactive accounts | Login-shell analysis | Authentication reconnaissance |
| SSH configuration | `.ssh` directory | Persistence investigation area |
| Authentication failures | `auth.log` | Security event requiring investigation |
| Credential simulation | Synthetic credential | Controlled demonstration |
| Backdoor detection simulation | Temporary test account | Persistence detection demonstration |

---

# ✅ Conclusion

The assessment demonstrated the security impact of an exposed VNC service on an intentionally vulnerable Metasploitable 2 system.

TCP/5900 was identified as the VNC service, successful VNC access provided GUI access, and subsequent enumeration exposed local account and authentication-related information.

SSH configuration and authentication logs were also examined as part of persistence and incident-response analysis.

The assessment used controlled simulations for credential exposure and backdoor-user detection.

**No real passwords were harvested, cracked, or disclosed, and no persistent backdoor was installed.**

The main defensive lesson is that remote graphical services should be properly secured, restricted to trusted networks, monitored, and protected with strong authentication.

---

# 📸 Screenshot Checklist

| Figure | Evidence |
|---|---|
| 1 | VMware lab environment |
| 2 | Nmap showing TCP/5900 |
| 3 | Xtightvnc process |
| 4 | TCP/5900 listening |
| 5 | VNC authentication prompt |
| 6 | Successful VNC GUI |
| 7 | `whoami`, `hostname`, `id` |
| 8 | `/home` enumeration |
| 9 | `/etc/passwd` |
| 10 | Interactive login accounts |
| 11 | Credential-access evidence |
| 12 | `.ssh` directory |
| 13 | Persistence analysis |
| 14 | `auth.log` authentication failures |
| 15 | CRON activity |
| 16 | Controlled credential simulation |
| 17 | Controlled test-account detection |

---

# 🧾 Final Attack Summary

| Item | Result |
|---|---|
| Vulnerability / Weakness | Exposed VNC service and weak authentication |
| Port | TCP 5900 |
| Access Obtained | VNC GUI |
| Post-Exploitation Focus | Credential Access |
| Credential Access Demonstrated | Local account and authentication-information enumeration |
| Persistence Analysis | Existing SSH authentication configuration and local accounts examined |
| Detection | Authentication logs and account configuration investigated |
| Credential Harvesting | Controlled synthetic simulation only |
| Backdoor | Controlled detection simulation only; no persistent backdoor installed |
| Target | Metasploitable 2 |
| Attacker | Kali Linux |

---

## ⚠️ Disclaimer

This project was performed in an isolated, authorized virtual laboratory environment for educational and cybersecurity training purposes only.
