## Overview
Built a 3-VM home lab in Oracle VirtualBox to practice SOC analyst skills safely — an isolated network with an attacker machine (Kali), a target server (Ubuntu), and a Windows endpoint, none of which touch my real home network.
## Setup
### 1. Installed Ubuntu Server as a guest VM
Set up Ubuntu Server 26.04.1 LTS as the third VM alongside my existing Kali Linux install, to act as the target/practice server in the lab.
![Ubuntu Server install](Screenshot%20from%202026-09-02%2022-33-37.png)
### 2. Installed Windows 10 (Enterprise Evaluation)
Added a Windows 10 VM to the lab — most companies run Windows endpoints, so practicing detection/monitoring on Windows is essential for SOC work.
![Windows 10 install](Screenshot%20from%202026-09-02%2022-59-32.png)
### 3. Configured internal networking 
Switched all three VMs' network adapters from NAT to **Internal Network** (named `soclab`), which lets the VMs talk to each other while keeping them fully isolated from the internet and my host machine's real network — a safer setup for hands-on practice.

![Internal network config 1](Pasted%20image%2020260902235331.png)

![Internal network config 2](Pasted%20image%2020260902235503.png)

![Internal network config 3](Pasted%20image%2020260902235527.png)

 ### 4. Assigned static IPs and verified connectivity
Internal Network has no DHCP, so IPs had to be assigned manually on each machine:

| VM | Role | IP Address | Interface |
|---|---|---|---|
| Kali Linux | Attacker/testing machine | 192.168.56.10 | eth0 |
| Ubuntu Server | Target/practice server | 192.168.56.11 | enp0s3 |
| Windows 10 | Endpoint monitoring practice | 192.168.56.12 | Ethernet |

Verified full connectivity with `ping` between all three machines (0% packet loss in both directions) and confirmed SSH access from Kali into Ubuntu Server.
## Troubleshooting (the part that actually taught me something) -
**Ubuntu's `cloud-init` hung for nearly an hour on boot** — it was stalling while trying to reach a cloud metadata service that doesn't exist in a local VirtualBox setup. Fixed by confirming the adapter was correctly set and letting the install fully complete rather than force-restarting early.
- **Interface names weren't consistent across OSes** — assumed `eth0` everywhere based on generic guides, but Ubuntu Server actually used `enp0s3` while Kali used `eth0` and Windows used `Ethernet`. Learned to check with `ip a` (Linux) / `Get-NetAdapter` (Windows) before assuming a name.
- **Windows Firewall silently blocked ping by default** — had to add an explicit inbound rule (`New-NetFirewallRule ... -Protocol ICMPv4`) before Kali could get a reply from the Windows VM, even though the IP and network config were already correct. 
- **SSH "connection refused" from Kali to Ubuntu** — Ubuntu Server didn't have `openssh-server` installed/running by default. Had to temporarily switch Ubuntu's adapter back to NAT to install it (`sudo apt install openssh-server`), then switch back to Internal Network and reassign the static IP, since Internal Network has no internet access for installs. 
- ## What I learned -
- How VirtualBox's Internal Network isolates VMs while still allowing controlled inter-VM communication
- Manual static IP assignment on both Linux (`ip addr add`) and Windows (`New-NetIPAddress`)
- That default OS behavior (interface naming, firewall rules, installed services) varies more than tutorials usually admit — and that diagnosing "why isn't this working" is itself a core skill, not a distraction from the "real" lab work
