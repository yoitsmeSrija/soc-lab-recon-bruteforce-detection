# Simulated Reconnaissance & SSH Brute-Force — Detection Lab

A hands-on home lab exercise simulating a common two-stage attack — network reconnaissance followed by credential brute-forcing — and detecting it using packet capture analysis and host log correlation. Built to practice the detection workflow of a SOC Analyst / Security Engineer.

## Overview
| **Attacker** | Kali Linux VM (`192.168.0.114`) |
| **Target** | Ubuntu VM (`192.168.0.112`) |
| **Tools Used** | Nmap, Hydra, Wireshark, `auth.log` |
| **Environment** | VirtualBox, internal/host-only network |
| **Date** | September 16, 2026 |

## Lab Topology

```
┌─────────────────┐        Internal/Host-only Network        ┌──────────────────┐
│   Kali Linux    │ ───────────────────────────────────────  │   Ubuntu Server  │
│   (Attacker)    │        192.168.0.0/24                    │   (Target)       │
│  192.168.0.114  │                                          |  192.168.0.112   │
│  - Nmap         │                                          │  - SSH (OpenSSH) │
│  - Hydra        │                                          │  - auth.log      │
└─────────────────┘                                          │  - Wireshark     | 
                                                             └──────────────────┘
```

## 1. Summary

On September 16, 2026, a simulated reconnaissance scan (Nmap) followed by an SSH brute-force attempt (Hydra) was conducted against an Ubuntu host from a Kali Linux VM, as part of a controlled home-lab security exercise. The activity was detected and correlated using **network-level evidence** (Wireshark packet capture) and **host-level evidence** (Ubuntu's `/var/log/auth.log`) — the two-source correlation approach used in real SOC investigations.

## 2. MITRE ATT&CK Mapping

Mapping the attack to the [MITRE ATT&CK](https://attack.mitre.org/) framework — the industry-standard reference used by SOC teams to classify adversary behavior:

| Stage | Technique | ID |

| Reconnaissance | Network Service Discovery | [T1046](https://attack.mitre.org/techniques/T1046/) |
| Credential Access | Brute Force — Password Guessing | [T1110.001](https://attack.mitre.org/techniques/T1110/001/) |

## 3. Timeline

| Time | Event |
|---|---|
| 17:20 | Nmap scan (`-sV`) launched from Kali against Ubuntu host |
| 17:27 | Scan traffic captured and identified in Wireshark |
| 17:31–17:38 | Repeated SSH connection attempts from `192.168.0.114`, most disconnecting `[preauth]` |
| 17:31:27 | Authentication failure logged for user `vboxuser` from Kali |
| 17:35 | Hydra brute-force run executed with password wordlist |

## 4. Detection Method

**Network-level (Wireshark):**
- `ip.src == 192.168.0.114` — isolated all traffic from the attacking machine
- `tcp.flags.syn == 1 && tcp.flags.ack == 0` — revealed the rapid multi-port connection pattern characteristic of an Nmap scan
- `tcp.port == 22` — showed a concentrated burst of connection attempts to the SSH service specifically

**Host-level (`/var/log/auth.log`):**
- Repeated entries from `192.168.0.114`, including `sshd: Received disconnect ... [preauth]`
- One explicit authentication failure logged for user `vboxuser`, consistent with an automated login attempt tool

Correlating both layers — what the network saw vs. what the host logged — is the core investigative habit this lab was built to practice.

## 5. Evidence

**Ubuntu `auth.log` showing repeated connection/auth attempts from the attacker IP:**
<img width="1837" height="716" alt="image" src="https://github.com/user-attachments/assets/0df275d0-14de-4244-a570-b3f1e7f6a03a" />

## 6. Impact Assessment

No real impact occurred — this was a controlled, isolated lab exercise conducted only against systems owned and operated by the analyst. In a real-world scenario, an unmitigated brute-force attempt of this kind could lead to unauthorized account access, lateral movement within the network, or full system compromise if a weak/guessable password were in use.

## 7. Root Cause

- SSH was reachable on the internal network with `PasswordAuthentication` enabled to support the exercise (disabled by default on modern Ubuntu specifically to prevent this class of attack)
- No account lockout, rate-limiting, or intrusion-prevention mechanism (e.g. fail2ban) was in place to stop repeated failed login attempts
- Several connection attempts were dropped at the pre-authentication stage, suggesting either a limited wordlist or an SSH-level restriction (e.g. `MaxAuthTries`) partially mitigating the attack

## 8. Recommendations

- Disable password-based SSH authentication in production; use key-based authentication only
- Deploy fail2ban (or equivalent) to automatically block IPs after a threshold of failed login attempts
- Restrict SSH access at the firewall level to known/trusted IP ranges
- Enable centralized logging and alerting for repeated authentication failures
- Regularly review `auth.log` (or forward to a SIEM) for early detection of brute-force patterns

## 9. Skills Demonstrated

- Building and configuring an isolated virtual lab network (VirtualBox)
- Reconnaissance and credential-attack simulation using industry-standard offensive tools (Nmap, Hydra)
- Traffic analysis and filter-writing in Wireshark
- Log analysis and correlation across network and host evidence sources
- Mapping observed behavior to the MITRE ATT&CK framework
- Writing a structured, analyst-style incident report

## 10. Conclusion

This exercise demonstrated the full detection lifecycle for a common attack pattern — reconnaissance followed by credential brute-forcing — using open-source tools and native Linux logging. It reinforced the importance of correlating network-level evidence with host-level evidence to build a complete picture of an incident, a core skill for SOC Analyst and Security Engineering roles.

---

*Lab conducted in an isolated VirtualBox environment against self-owned virtual machines only.*
