# 🦜 Parrot OS — Configuration & Troubleshooting: Every Command Explained

> **Purpose:** Every configuration and troubleshooting command broken down — what it does, why you use it, what each flag means, and what you can achieve with it
> **Style:** Beginner-friendly explanations with real-world context
> **Version:** Parrot OS 7.0 "Echo" (Debian 13 base)

---

## 📋 Table of Contents

1. [How to Read This Guide](#how-to-read-this-guide)
2. [System Update Commands](#2-system-update-commands)
3. [Package Management Commands](#3-package-management-commands)
4. [Network Configuration Commands](#4-network-configuration-commands)
5. [Service Management Commands](#5-service-management-commands-systemctl)
6. [User & Permission Commands](#6-user--permission-commands)
7. [Firewall Commands](#7-firewall-commands)
8. [Disk & Storage Commands](#8-disk--storage-commands)
9. [Process & System Monitoring](#9-process--system-monitoring)
10. [Log & Journal Commands](#10-log--journal-commands)
11. [SSH Configuration Commands](#11-ssh-configuration-commands)
12. [VPN Commands](#12-vpn-commands)
13. [Wi-Fi & Wireless Commands](#13-wi-fi--wireless-commands)
14. [Boot & GRUB Commands](#14-boot--grub-commands)
15. [Shell & Environment Commands](#15-shell--environment-commands)
16. [File & Directory Commands](#16-file--directory-commands)
17. [Metasploit Setup Commands](#17-metasploit-setup-commands)
18. [Tor & AnonSurf Commands](#18-tor--anonsurf-commands)
19. [Docker Commands](#19-docker-commands)
20. [Troubleshooting Command Toolkit](#20-troubleshooting-command-toolkit)
21. [Complete Cheat Sheet](#21-complete-cheat-sheet)

---

## How to Read This Guide

Every command follows this pattern:

```
command -flag argument
│       │      │
│       │      └─ WHAT IT TARGETS (file, IP, service name...)
│       └─ WHAT THE FLAG CHANGES (behavior modifier)
└─ WHAT THE COMMAND DOES (base action)
```

Each section explains:
- 🔵 **WHAT** — what the command literally does
- 🟢 **WHY** — when and why you'd use it
- 🟡 **FLAGS** — what each flag/option means
- 🔴 **OUTPUT** — what you'll see when it runs
- ⚡ **PRO TIP** — advanced usage or common mistakes

---

## 2. System Update Commands

### 🔵 Understanding the Update Process

```
Parrot OS Update Flow:
apt update    →  Downloads package lists (metadata only — no actual packages)
apt upgrade   →  Installs newer versions of installed packages
apt autoremove → Removes packages no longer needed
```

---

```bash
sudo parrot-upgrade
```
| Part | Meaning |
|------|---------|
| `sudo` | **Run as superuser** — updates affect system files owned by root. Without sudo, permission denied. |
| `parrot-upgrade` | **Parrot's custom update wrapper** — safer than raw `apt upgrade` for Parrot OS. Runs: apt update + apt full-upgrade + apt autoremove in the correct order, with Parrot-specific checks. |

🟢 **WHY use this instead of plain `apt upgrade`:** Parrot OS has specific package dependencies. `parrot-upgrade` respects these relationships and avoids breaking the system. Always use this as your daily update command.

🔴 **What you'll see:**
```
[*] Updating package lists...
[*] Upgrading packages...
[*] Removing obsolete packages...
[*] Done. System is up to date.
```

---

```bash
sudo apt update
```
| Part | Meaning |
|------|---------|
| `sudo` | Root needed to write to /var/lib/apt/ |
| `apt` | **Advanced Package Tool** — Debian/Ubuntu/Parrot's package manager |
| `update` | **Refresh package lists** — downloads updated metadata from all repositories. Does NOT install or upgrade anything. Just checks what's available. |

🟢 **WHY:** Before installing anything, run this so apt knows the latest available versions. Without it, apt might install an outdated version.

🔴 **Output:**
```
Hit:1 https://deb.parrot.sh/parrot parrot InRelease     ← Repository reached, up to date
Get:2 https://security.debian.org trixie-security InRelease  ← Downloading updated index
Fetched 2,345 kB in 3s
Reading package lists... Done
```

⚡ **PRO TIP:** `Hit` = already have latest index. `Get` = downloading newer index. `Err` = failed to reach repository (internet/DNS problem).

---

```bash
sudo apt upgrade
```
| Part | Meaning |
|------|---------|
| `upgrade` | **Install newer versions** of all currently installed packages. Will NOT remove packages or install new ones even if needed. |

🟢 **WHY:** Keeps installed software patched and secure. Run after `apt update`.

---

```bash
sudo apt full-upgrade
```
| Part | Meaning |
|------|---------|
| `full-upgrade` | Like upgrade, but CAN remove packages if needed to complete the upgrade. Handles complex dependency changes. |

🟢 **WHY:** When a major update requires removing old packages to install new ones, `full-upgrade` handles it. `upgrade` would just skip those packages.

---

```bash
sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y
```
| Part | Meaning |
|------|---------|
| `&&` | **AND operator** — only run next command if previous succeeded. If update fails, don't try to upgrade. |
| `-y` | **Yes to all prompts** — automatically answer "yes" to "Do you want to continue? [Y/n]". Non-interactive. |
| `autoremove` | **Remove orphaned packages** — packages that were installed as dependencies but are no longer needed by anything. |

🟢 **WHY combine them:** One command that safely updates everything and cleans up. `-y` flag makes it fully automated — good for scripts.

---

```bash
sudo apt clean
```
🔵 **WHAT:** Deletes all downloaded .deb package files from `/var/cache/apt/archives/`.

🟢 **WHY:** Those .deb files are kept after installation "in case you need to reinstall." They take up disk space. After a big upgrade, run this to free hundreds of MB.

🔴 **What gets deleted:** `/var/cache/apt/archives/*.deb` — the actual installer files. Package list metadata (needed for apt to work) is NOT deleted.

---

```bash
sudo apt autoremove
```
🔵 **WHAT:** Removes packages installed automatically as dependencies for other packages, when the package that needed them has been removed.

🟢 **WHY:** When you uninstall a program, its dependencies stay behind. Over time this accumulates. `autoremove` cleans them up.

⚠️ **CAUTION:** Review what it wants to remove before confirming. Rarely, it might suggest removing something you actually use. Read the list before pressing Y.

---

## 3. Package Management Commands

```bash
sudo apt install package-name
```
| Flag/Part | Meaning |
|-----------|---------|
| `install` | Download and install a package and all its dependencies |
| `package-name` | Name of the software to install |

```bash
sudo apt install -y nmap wireshark gobuster
```
| Part | Meaning |
|------|---------|
| `-y` | Skip "Do you want to continue?" prompt — auto-yes |
| Multiple names | Install several packages at once in one command |

🟢 **WHY install multiple at once:** Single apt transaction — faster than running apt multiple times. All dependencies resolved together.

---

```bash
sudo apt install --reinstall package-name
```
| Part | Meaning |
|------|---------|
| `--reinstall` | Download and install fresh copy even if already installed |

🟢 **WHY:** When a tool is broken/corrupt — its files may have been accidentally deleted or modified. `--reinstall` restores it to original state without needing to remove first.

---

```bash
sudo apt install -f
```
| Part | Meaning |
|------|---------|
| `-f` | **Fix** broken dependencies — tries to correct broken package states |

🟢 **WHY:** After a failed installation, some packages may be "half-installed." This completes or removes them and fixes the dependency chain.

---

```bash
sudo apt remove package-name
sudo apt purge package-name
```
| Command | What It Does | When to Use |
|---------|-------------|------------|
| `remove` | Remove package binary but KEEP config files in `/etc/` | When you might reinstall later and want to keep settings |
| `purge` | Remove package AND all its config files | Complete removal — fresh start if reinstalled |

🟢 **WHY `purge` over `remove`:** If a package was misconfigured and causing problems, `purge` wipes the bad config too. Next installation starts completely fresh.

---

```bash
apt search keyword
apt search "web scanner"
```
🔵 **WHAT:** Searches package names AND descriptions for the keyword.

🟢 **WHY:** Discover tools you didn't know existed. `apt search "port scanner"` finds nmap, masscan, etc.

🔴 **Output:**
```
nmap/parrot 7.94+git20230807 amd64
The Network Mapper

masscan/parrot 1.3.2 amd64
TCP port scanner, spews SYN packets asynchronously
```

---

```bash
apt show package-name
```
🔵 **WHAT:** Shows detailed info about a package: version, size, dependencies, description, maintainer, homepage.

🟢 **WHY:** Before installing something unfamiliar, check what it is and what it depends on. Also shows if it's already installed.

---

```bash
apt list --installed
apt list --installed | grep python
```
| Part | Meaning |
|------|---------|
| `--installed` | Filter to only show installed packages |
| `\| grep python` | Pipe through grep to filter results further |

🟢 **WHY:** Find out which version of a tool is installed, or check if something is installed at all.

---

```bash
dpkg -l | grep package
dpkg -L package-name
dpkg -i package.deb
```
| Command | What It Does | Why Use It |
|---------|-------------|-----------|
| `dpkg -l \| grep X` | List installed packages matching X | Lower-level than apt, always available |
| `dpkg -L package` | List all FILES installed by a package | Find where a tool's binaries and configs are |
| `dpkg -i file.deb` | Install a .deb file directly | Install downloaded packages not in repos |

🟢 **WHY `dpkg` instead of `apt`:** When installing a .deb file you downloaded manually (e.g., VSCode, Chrome, custom tools), `dpkg -i` installs it directly. Use `apt install -f` after if dependencies are missing.

---

```bash
sudo dpkg --configure -a
```
| Part | Meaning |
|------|---------|
| `--configure` | Configure unpacked but not yet configured packages |
| `-a` | All packages needing configuration |

🟢 **WHY:** After a crash or power cut during apt install, packages may be left "unpacked but not configured." This completes the configuration. Run this when apt says "dpkg was interrupted."

---

## 4. Network Configuration Commands

```bash
ip a
# Full form:
ip address show
```
🔵 **WHAT:** Shows all network interfaces with their IP addresses, MAC addresses, and status.

🟢 **WHY:** First command to run to understand your network setup. Shows: which interfaces exist, what IPs they have, which are UP/DOWN.

🔴 **Output explained:**
```
1: lo: <LOOPBACK,UP>          ← Loopback (127.0.0.1) — internal only
inet 127.0.0.1/8           ← IPv4 loopback address

2: eth0: <BROADCAST,MULTICAST,UP>  ← Wired ethernet — UP means active
link/ether aa:bb:cc:dd:ee:ff    ← MAC address
inet 192.168.1.100/24          ← Your IP and subnet mask

3: wlan0: <BROADCAST,MULTICAST>   ← Wi-Fi interface — no UP = disconnected

4: tun0: <POINTOPOINT,UP>         ← VPN tunnel interface (when VPN active)
inet 10.10.14.5/23             ← Your VPN IP (use this as LHOST in Metasploit!)
```

⚡ **PRO TIP:** When using HackTheBox or TryHackMe, your `tun0` IP is your attack IP. Use `ip a show tun0` to find it quickly.

---

```bash
ip r
# Full form:
ip route show
```
🔵 **WHAT:** Shows the routing table — how packets are sent to different destinations.

🟢 **WHY:** Diagnose routing problems. Understand which gateway your traffic goes through. Critical when VPN is connected — check if traffic routes correctly.

🔴 **Output explained:**
```
default via 192.168.1.1 dev eth0    ← Default route: unknown destinations go to router
	192.168.1.0/24 dev eth0             ← Local network: reach directly via eth0
	10.10.10.0/23 via 10.10.14.1 dev tun0  ← VPN routes: HTB machines go through VPN
	```
	
	---
	
	```bash
	ip link set eth0 up
	ip link set eth0 down
	```
	| Part | Meaning |
	|------|---------|
	| `ip link` | Manage network interface link layer |
	| `set eth0` | Target this specific interface |
	| `up` / `down` | Bring interface up (activate) or down (deactivate) |
	
	🟢 **WHY `down` then `up`:** Soft-restart a network interface without rebooting. Fixes stuck/glitchy connections. Like unplugging and replugging a network cable.
	
	---
	
	```bash
	ip addr add 192.168.1.100/24 dev eth0
	ip addr del 192.168.1.100/24 dev eth0
	```
	| Part | Meaning |
	|------|---------|
	| `addr add` | Add an IP address to an interface |
	| `192.168.1.100/24` | IP address with subnet mask (/24 = 255.255.255.0) |
	| `dev eth0` | On this device/interface |
	
	🟢 **WHY:** Manually assign an IP when DHCP isn't available or you need a specific address. `/24` means the last octet is for hosts — 256 addresses on this subnet.
	
	---
	
	```bash
	ip route add default via 192.168.1.1
	```
	| Part | Meaning |
	|------|---------|
	| `route add` | Add entry to routing table |
	| `default` | "default" = catch-all for any destination not in routing table |
	| `via 192.168.1.1` | Send those packets to this gateway (your router) |
	
	🟢 **WHY:** After manually configuring an IP, you also need a route. Without a default route, your machine can't reach anything outside its subnet.
	
	---
	
	```bash
	sudo dhclient eth0
	```
	🔵 **WHAT:** Sends a DHCP request on eth0 — asks the router to assign an IP address automatically.
	
	🟢 **WHY:** When you manually took an interface down and back up, it might not automatically re-request an IP. Also useful when DHCP lease expired.
	
	---
	
	```bash
	nmcli device status
	nmcli connection show
	```
	| Command | What It Shows | Why Use It |
	|---------|--------------|-----------|
	| `nmcli device status` | All network devices and their state (connected/disconnected) | Quick overview of network hardware status |
	| `nmcli connection show` | All saved network connection profiles | See configured connections (Wi-Fi, VPN, Ethernet) |
	
	🟢 **WHY `nmcli` over `ip`:** NetworkManager manages connections persistently — settings survive reboots. `ip` commands are temporary and reset on reboot.
	
	---
	
	```bash
	nmcli con mod "Wired connection 1" \
	ipv4.method manual \
	ipv4.addresses "192.168.1.100/24" \
	ipv4.gateway "192.168.1.1" \
	ipv4.dns "8.8.8.8,1.1.1.1"
	nmcli con up "Wired connection 1"
	```
	| Part | Meaning |
	|------|---------|
	| `con mod` | **Connection modify** — change settings of a saved connection |
	| `"Wired connection 1"` | Name of the connection profile (use `nmcli con show` to find exact name) |
	| `ipv4.method manual` | Use static IP instead of DHCP |
	| `ipv4.addresses "192.168.1.100/24"` | Set this static IP with subnet |
	| `ipv4.gateway "192.168.1.1"` | Set the default gateway (router) |
	| `ipv4.dns "8.8.8.8,1.1.1.1"` | Use Google and Cloudflare DNS |
	| `con up` | Activate (apply) these settings now |
	
	🟢 **WHY:** Persistent static IP that survives reboots. Good for servers, lab machines, or when you need a consistent IP for port forwarding.
	
	---
	
	```bash
	ping -c 4 8.8.8.8
	ping -c 4 google.com
	```
	| Part | Meaning |
	|------|---------|
	| `ping` | Send ICMP echo requests — tests basic connectivity |
	| `-c 4` | **Count** — send exactly 4 packets then stop. Without -c, pings forever. |
	| `8.8.8.8` | Google's DNS server (tests internet connectivity by IP) |
	| `google.com` | Tests DNS resolution AND internet connectivity |
	
	🟢 **WHY two different pings:** 
	- `ping 8.8.8.8` works → internet works, IP routing works
	- `ping google.com` fails → DNS problem (can't resolve names to IPs)
	- Both fail → no internet / firewall blocking ICMP
	
	---
	
	```bash
	traceroute google.com
	mtr google.com
	```
	| Command | What It Does | Why Use It |
	|---------|-------------|-----------|
	| `traceroute` | Shows each "hop" (router) between you and destination | Find WHERE a connection problem occurs — which hop drops packets |
	| `mtr` | Like traceroute but live, updating continuously | Identify intermittent packet loss at specific hops |
	
	---
	
	```bash
	nslookup google.com
	dig google.com
	dig google.com +short
	dig @8.8.8.8 google.com
	```
	| Command | What It Shows | Why Use It |
	|---------|--------------|-----------|
	| `nslookup google.com` | DNS lookup — what IP does google.com resolve to | Check if DNS works |
	| `dig google.com` | Detailed DNS query with full response | Troubleshoot DNS, see TTL, record types |
	| `dig +short` | Just the IP answer | Quick IP lookup |
	| `dig @8.8.8.8 google.com` | Query specific DNS server | Test if a particular DNS server works |
	
	🟢 **WHY `dig @8.8.8.8`:** When `ping google.com` fails but `ping 8.8.8.8` works, test directly against a known-good DNS server to confirm it's a local DNS config issue.
	
	---
	
	```bash
	ss -tulnp
	netstat -tulnp
	```
	| Flag | Meaning |
	|------|---------|
	| `-t` | TCP connections |
	| `-u` | UDP connections |
	| `-l` | Listening ports only (services waiting for connections) |
	| `-n` | Numeric — show port numbers instead of service names (faster, unambiguous) |
	| `-p` | Show the Process using each port |
	
	🟢 **WHY:** See what's listening on your machine. Critical for:
	- Security: what services are exposed?
	- Debugging: is Metasploit really listening on 4444?
	- Conflict: is something already using the port you need?
	
	🔴 **Output explained:**
	```
	Proto  Local Address   State    PID/Program
	tcp    0.0.0.0:22      LISTEN   1234/sshd      ← SSH listening on all interfaces
	tcp    127.0.0.1:5432  LISTEN   5678/postgres  ← PostgreSQL, localhost only
	tcp    0.0.0.0:4444    LISTEN   9999/nc        ← Netcat listener (your reverse shell!)
	```
	
	---
	
	```bash
	cat /etc/resolv.conf
	```
	🔵 **WHAT:** Shows current DNS server configuration.
	
	🟢 **WHY:** When DNS stops working, check this file first. Should contain `nameserver 8.8.8.8` or similar. If empty or has wrong IP → DNS broken.
	
	```bash
	echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
	echo "nameserver 1.1.1.1" | sudo tee -a /etc/resolv.conf
	```
	| Part | Meaning |
	|------|---------|
	| `echo "nameserver 8.8.8.8"` | Create text to write |
	| `\|` | Pipe — send output of echo to next command |
	| `sudo tee /etc/resolv.conf` | Write to file (tee writes to file AND stdout, needs sudo for system files) |
	| `-a` | Append — add to end of file instead of overwriting |
	
	🟢 **WHY `tee` instead of `>`:** `sudo echo > file` doesn't work — the redirect runs as your user, not sudo. `tee` runs as sudo, so it CAN write to root-owned files.
	
	---
	
	## 5. Service Management Commands (systemctl)
	
	**Understanding systemd:**
	```
	systemd = init system (PID 1) — manages all services
	Service  = a background process (ssh, apache, postgresql, metasploit)
	Unit     = any resource systemd manages (services, mounts, timers, etc.)
	
	Service states:
	active (running)   = currently running ✅
	active (exited)    = ran successfully and finished
	inactive (dead)    = not running
	failed             = crashed or couldn't start ❌
	enabled            = will start on boot
	disabled           = won't start on boot
	```
	
	---
	
	```bash
	sudo systemctl start ssh
	```
	| Part | Meaning |
	|------|---------|
	| `systemctl` | System control — interface to systemd |
	| `start` | Start the service NOW (doesn't affect boot behavior) |
	| `ssh` | Short name for `ssh.service` |
	
	🟢 **WHY:** Start a service for immediate use without enabling it permanently. Good for testing.
	
	---
	
	```bash
	sudo systemctl stop ssh
	```
	🔵 **WHAT:** Gracefully stops the service — sends SIGTERM, waits for clean shutdown, then SIGKILL if needed.
	
	🟢 **WHY `stop` vs killing process:** `stop` lets the service clean up properly (close files, save state, release ports). Killing with `kill` is abrupt and may leave things broken.
	
	---
	
	```bash
	sudo systemctl restart ssh
	```
	🔵 **WHAT:** Stops then starts the service. Applies config changes.
	
	🟢 **WHY:** After editing a config file (like `/etc/ssh/sshd_config`), the running service doesn't know about changes. Restart loads the new config.
	
	---
	
	```bash
	sudo systemctl reload ssh
	```
	🔵 **WHAT:** Tells service to re-read its config WITHOUT restarting. Keeps existing connections alive.
	
	🟢 **WHY:** For services where restarting would disconnect users. Nginx/Apache support this — active connections stay alive while new config is applied.
	
	⚠️ **NOTE:** Not all services support reload. If unsure: `sudo systemctl reload-or-restart service`
	
	---
	
	```bash
	sudo systemctl enable ssh
	sudo systemctl disable ssh
	```
	| Command | What It Does | Immediate Effect |
	|---------|-------------|-----------------|
	| `enable` | Create symlink so service starts on every boot | Does NOT start it now |
	| `disable` | Remove symlink — won't start on boot | Does NOT stop it now |
	
	🟢 **WHY separate enable from start:** You can enable (will auto-start on boot) without starting now. You can start (running now) without enabling (won't persist after reboot).
	
	---
	
	```bash
	sudo systemctl enable --now ssh
	```
	| Part | Meaning |
	|------|---------|
	| `--now` | Enable AND start immediately in one command |
	
	🟢 **WHY:** Most efficient — configure + start in one shot. Use this when setting up a service.
	
	---
	
	```bash
	systemctl status ssh
	```
	🔵 **WHAT:** Shows comprehensive status — running state, last log lines, PID, memory usage, start time.
	
	🔴 **Output explained:**
	```
	● ssh.service - OpenBSD Secure Shell server
	Loaded: loaded (/lib/systemd/system/ssh.service; enabled)
	← "enabled" = starts on boot
	Active: active (running) since Mon 2025-01-01 10:00:00; 2h ago
	← "active (running)" = currently working
	Main PID: 1234 (sshd)
	← Process ID
	Tasks: 1 (limit: 4915)
	Memory: 4.2M
	CGroup: /system.slice/ssh.service
	└─1234 sshd: /usr/sbin/sshd -D [listener]
	
	Jan 01 10:00:00 parrot sshd[1234]: Server listening on 0.0.0.0 port 22.
	← Recent log entries
	```
	
	---
	
	```bash
	systemctl is-active ssh
	systemctl is-enabled ssh
	systemctl is-failed ssh
	```
	| Command | Output if true | Why Use |
	|---------|---------------|---------|
	| `is-active` | "active" | Script-friendly check: `if systemctl is-active ssh; then echo running; fi` |
	| `is-enabled` | "enabled" | Check boot config without reading full status |
	| `is-failed` | "failed" | Detect crashed services in scripts |
	
	---
	
	```bash
	systemctl list-units --type=service
	systemctl list-units --type=service --all
	systemctl list-units --failed
	```
	| Command | Shows |
	|---------|-------|
	| `--type=service` | All active services |
	| `--all` | All services including inactive/dead |
	| `--failed` | Only failed services |
	
	🟢 **WHY `--failed`:** Quick way to find what crashed on your system. First thing to check after a boot problem.
	
	---
	
	```bash
	sudo systemctl set-default graphical.target
	sudo systemctl set-default multi-user.target
	```
	| Target | Equivalent | What it Means |
	|--------|-----------|---------------|
	| `graphical.target` | Old runlevel 5 | Boot with GUI (KDE desktop) |
	| `multi-user.target` | Old runlevel 3 | Boot to text/terminal only (no GUI) |
	
	🟢 **WHY `multi-user`:** Saves RAM (no desktop environment). Good for servers, or when GUI is broken and you need to fix it. Also boots faster.
	
	```bash
	sudo systemctl isolate graphical.target
	```
	🔵 **WHAT:** Immediately SWITCH to graphical mode (starts GUI) without rebooting.
	
	---
	
	## 6. User & Permission Commands
	
	```bash
	sudo useradd -m -s /bin/bash -G sudo username
	```
	| Flag | Meaning | Why |
	|------|---------|-----|
	| `useradd` | Create new user account | Base command |
	| `-m` | **Make home directory** `/home/username` | Without -m, no home directory is created — user can't store files or configs |
	| `-s /bin/bash` | **Shell** — set bash as login shell | Without this, user gets `/bin/sh` or no shell at all |
	| `-G sudo` | **Groups** — add to sudo group | Allows user to run sudo commands |
	| `username` | The username to create | |
	
	🟢 **WHY ALL these flags:** `useradd` alone creates a bare account with no home, no shell, unusable. These flags make a proper usable account.
	
	---
	
	```bash
	sudo passwd username
	passwd
	```
	| Command | What It Does |
	|---------|-------------|
	| `sudo passwd username` | Set/change another user's password (requires root) |
	| `passwd` (no args) | Change YOUR OWN password |
	
	---
	
	```bash
	sudo usermod -aG sudo username
	sudo usermod -aG docker,netdev,wireshark username
	```
	| Flag | Meaning | Critical Detail |
	|------|---------|-----------------|
	| `usermod` | Modify user account | Changes properties of existing user |
	| `-a` | **Append** to groups | WITHOUT -a, it REPLACES all groups. Always use -aG together! |
	| `-G` | **Groups** — specify group(s) | Comma-separated list, no spaces |
	
	⚠️ **CRITICAL:** `usermod -G sudo user` (without -a) REMOVES user from ALL other groups and only adds sudo. Always `-aG` to APPEND.
	
	🟢 **Common groups to add:**
	```
	sudo      → Can use sudo
	docker    → Can use Docker without sudo
	wireshark → Can capture packets without sudo
	netdev    → Network device management
	bluetooth → Bluetooth access
	video     → GPU/display access
	```
	
	---
	
	```bash
	newgrp docker
	```
	🔵 **WHAT:** Starts new shell session with new group membership active — without logging out.
	
	🟢 **WHY:** After `usermod -aG docker user`, you must log out and back in for the group change to take effect. `newgrp` applies it immediately in current terminal session.
	
	---
	
	```bash
	id username
	groups username
	```
	| Command | Output | Why |
	|---------|--------|-----|
	| `id username` | `uid=1000(user) gid=1000(user) groups=1000(user),27(sudo),999(docker)` | Full UID, GID, and all groups |
	| `groups username` | `user sudo docker wireshark` | Just the group names |
	
	🟢 **WHY:** Verify that usermod worked. Confirm user is in expected groups.
	
	---
	
	```bash
	chmod 755 file
	chmod 644 file
	chmod +x script.sh
	chmod -R 755 directory/
	```
	**Understanding permission numbers:**
	```
	Permission = Owner | Group | Others
	rwx     rwx     rwx
	421     421     421
	
	755 = rwx r-x r-x = Owner:all, Group:read+execute, Others:read+execute
	644 = rw- r-- r-- = Owner:read+write, Group:read, Others:read
	600 = rw- --- --- = Owner only (SSH keys should be 600)
	777 = rwx rwx rwx = Everyone everything (DANGEROUS — never use on sensitive files)
	```
	
	| Command | What It Sets | When to Use |
	|---------|-------------|------------|
	| `chmod 755 file` | Owner:rwx Group:r-x Others:r-x | Executables, directories |
	| `chmod 644 file` | Owner:rw Group:r Others:r | Regular files, configs |
	| `chmod 600 file` | Owner:rw only | SSH private keys, password files |
	| `chmod +x file` | Add execute bit for all | Make a script runnable |
	| `chmod -R 755 dir/` | Apply 755 to directory and ALL contents | Fixing web directories |
	
	🟢 **WHY 600 for SSH keys:** If your private key is readable by others, SSH refuses to use it — "UNPROTECTED PRIVATE KEY FILE!" warning. It's a security protection.
	
	---
	
	```bash
	sudo chown user:group file
	sudo chown -R www-data:www-data /var/www/html/
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `chown` | **Change owner** of file/directory | When a file needs to be owned by different user |
	| `user:group` | New owner:new group | Can change just owner: `chown user file` or just group: `chown :group file` |
	| `-R` | **Recursive** — apply to directory AND all contents | Without -R, only the directory itself changes, not files inside |
	
	---
	
	```bash
	sudo visudo
	```
	🔵 **WHAT:** Safely edits `/etc/sudoers` — the file that controls who can use sudo and how.
	
	🟢 **WHY `visudo` instead of `nano /etc/sudoers`:** `visudo` validates syntax before saving. If you make a typo in sudoers with regular editor, you could lock yourself out of sudo permanently. `visudo` prevents that.
	
	**Key sudoers entries explained:**
	```
	username ALL=(ALL:ALL) ALL
	│        │    │    │    │
	│        │    │    │    └─ Commands: ALL = any command
	│        │    │    └─ Run as group: ALL = any group  
	│        │    └─ Run as user: ALL = any user (including root)
	│        └─ From host: ALL = any machine
	└─ Who: username
	
	username ALL=(ALL) NOPASSWD: /usr/bin/nmap
	│          │
	│          └─ Only this specific command
	└─ No password prompt when using sudo
	```
	
	---
	
	## 7. Firewall Commands
	
	```bash
	sudo ufw enable
	sudo ufw disable
	sudo ufw status verbose
	```
	| Command | What It Does | Why |
	|---------|-------------|-----|
	| `ufw enable` | Activate firewall — starts blocking by default | Apply default deny incoming rule |
	| `ufw disable` | Turn off firewall — all traffic allowed | During testing when firewall interferes |
	| `status verbose` | Show all rules + current status | See exactly what's blocked/allowed |
	
	🟢 **WHY `verbose`:** Plain `ufw status` shows less detail. `verbose` shows default policies (incoming/outgoing) + all custom rules.
	
	---
	
	```bash
	sudo ufw default deny incoming
	sudo ufw default allow outgoing
	```
	| Rule | What It Means | Why This Setup |
	|------|--------------|----------------|
	| `deny incoming` | Block ALL inbound connections by default | Nothing can connect to you unless explicitly allowed |
	| `allow outgoing` | Allow ALL outbound connections by default | You can connect to anything |
	
	🟢 **WHY this combination:** Standard security posture. You initiate connections (browsing, updates, tools) freely. But attackers can't connect to your machine unless you explicitly open a port.
	
	---
	
	```bash
	sudo ufw allow 22/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 4444/tcp
	sudo ufw deny 23/tcp
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `allow` | Open this port — let connections through | You need this service accessible |
	| `deny` | Block this port | Security hardening |
	| `22/tcp` | Port number / protocol | Port 22 = SSH, protocol TCP |
	| `4444/tcp` | Custom port | Your Metasploit reverse shell listener port |
	
	🟢 **WHY specify `/tcp`:** Some services use UDP. `allow 53/udp` for DNS. `allow 53` opens both TCP and UDP. Be specific.
	
	---
	
	```bash
	sudo ufw allow from 192.168.1.50
	sudo ufw allow from 10.10.14.0/24 to any port 22
	```
	| Rule | What It Does | Why |
	|------|-------------|-----|
	| `allow from IP` | Only allow connections from specific IP | Restrict SSH to specific machine |
	| `from NETWORK to any port 22` | Allow entire subnet to access SSH | Allow team members on VPN to SSH in |
	
	🟢 **WHY source-based rules:** Better security than opening port to world. Only YOUR IP can connect.
	
	---
	
	```bash
	sudo ufw status numbered
	sudo ufw delete 3
	sudo ufw delete allow 22/tcp
	```
	| Command | What It Does |
	|---------|-------------|
	| `status numbered` | Show rules with line numbers |
	| `delete 3` | Delete rule #3 (by number from above) |
	| `delete allow 22/tcp` | Delete specific rule by its description |
	
	---
	
	```bash
	sudo iptables -L -n -v --line-numbers
	```
	| Flag | Meaning | Why |
	|------|---------|-----|
	| `-L` | **List** all rules |  |
	| `-n` | **Numeric** — show IPs and ports as numbers, not hostnames | Faster, no DNS lookups, unambiguous |
	| `-v` | **Verbose** — show packet/byte counts and interface | See which rules are actually matching traffic |
	| `--line-numbers` | Show rule line numbers | Needed to delete specific rules by position |
	
	🟢 **WHY iptables over UFW:** UFW is a frontend — eventually uses iptables. For complex rules (NAT, port forwarding, masquerading), you need direct iptables. UFW can't do everything.
	
	---
	
	```bash
	sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
	echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `-t nat` | NAT table | Packet address translation rules |
	| `-A PREROUTING` | Append to PREROUTING chain | Rules that run BEFORE routing decisions |
	| `-p tcp` | Protocol TCP | |
	| `--dport 8080` | Destination port 8080 | Packets arriving on port 8080 |
	| `-j REDIRECT` | Jump to REDIRECT action | Redirect to different port on same machine |
	| `--to-port 80` | Redirect to port 80 | Web server listens on 80 |
	| `ip_forward = 1` | Enable packet forwarding | Required for machine to forward packets between interfaces |
	
	🟢 **WHY these:** Classic pentesting setup — redirect traffic for interception, or forward traffic through compromised machines.
	
	---
	
	## 8. Disk & Storage Commands
	
	```bash
	df -h
	```
	| Flag | Meaning | Why |
	|------|---------|-----|
	| `df` | **Disk free** — show disk space usage by filesystem | |
	| `-h` | **Human-readable** — show GB/MB instead of raw bytes | 50G is easier to read than 53687091200 |
	
	🔴 **Output explained:**
	```
	Filesystem      Size  Used Avail Use% Mounted on
	/dev/sda1        50G   23G   25G  48% /          ← Root partition
	tmpfs           3.9G     0  3.9G   0% /dev/shm   ← RAM disk
	/dev/sda2       100G   45G   50G  47% /home       ← Home directory
	```
	
	🟢 **WHY check this:** When apt fails or tools crash, often it's a full disk. Check `Use%` — if any partition is 95%+ → clean up immediately.
	
	---
	
	```bash
	du -sh /var/log/
	du -sh ~/*
	du -sh /* 2>/dev/null | sort -h
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `du` | **Disk usage** — space used by files/directories |  |
	| `-s` | **Summary** — single number for entire directory (not every file) | Without -s, lists every subdirectory |
	| `-h` | Human-readable | |
	| `~/*` | All items in home directory | Find what's taking up space |
	| `2>/dev/null` | Redirect errors to /dev/null (hide "Permission denied") | Clean output |
	| `sort -h` | Sort by human-readable sizes (largest last) | Find biggest directories |
	
	🟢 **WHY `sort -h`:** Finds your biggest space consumers. `du -sh /* 2>/dev/null | sort -h | tail -20` shows top 20 largest directories.
	
	---
	
	```bash
	lsblk
	lsblk -f
	```
	| Command | Shows | Why |
	|---------|-------|-----|
	| `lsblk` | Block devices tree (disks + partitions) with sizes | See all storage devices clearly |
	| `lsblk -f` | **Filesystem** info — adds filesystem type, UUID, mount point | Full storage picture |
	
	🔴 **Output:**
	```
	NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
	sda      8:0    0    50G  0 disk            ← Whole disk
	├─sda1   8:1    0    49G  0 part /          ← Root partition
	└─sda2   8:2    0     1G  0 part [SWAP]     ← Swap space
	sdb      8:16   1    32G  0 disk            ← USB drive
	└─sdb1   8:17   1    32G  0 part /media/usb ← USB mounted
	```
	
	---
	
	```bash
	sudo mount /dev/sdb1 /mnt/usb
	sudo umount /mnt/usb
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `mount` | Attach filesystem to directory tree | Make storage device accessible |
	| `/dev/sdb1` | Device file for the partition | First partition of second disk |
	| `/mnt/usb` | Mount point directory | Where you'll access the files |
	| `umount` | Detach filesystem | ALWAYS unmount before removing device — prevents data corruption |
	
	🟢 **WHY unmount matters:** Like safely ejecting a USB. Unmount flushes cached writes to the device. Yanking without unmounting = possible corruption.
	
	---
	
	```bash
	sudo mount -t ntfs-3g /dev/sdb1 /mnt/usb
	sudo mount -t exfat /dev/sdb1 /mnt/usb
	```
	| Part | Meaning |
	|------|---------|
	| `-t ntfs-3g` | **Type** — specify filesystem type. `ntfs-3g` = read/write NTFS (Windows drives). `exfat` = exFAT (modern USB drives). |
	
	🟢 **WHY specify type:** Linux can't auto-detect all filesystem types. Windows NTFS drives need explicit `-t ntfs-3g` flag. Must have `ntfs-3g` package installed.
	
	---
	
	```bash
	sudo fdisk -l
	sudo fdisk /dev/sdb
	```
	| Command | What It Does | When to Use |
	|---------|-------------|------------|
	| `fdisk -l` | **List** partition tables of all disks | See all disks and their partition layout |
	| `fdisk /dev/sdb` | **Interactive partition editor** for disk sdb | Create, delete, modify partitions |
	
	⚠️ **DANGER:** `fdisk /dev/sdb` can destroy data. Know which disk you're targeting before running.
	
	---
	
	```bash
	sudo dd if=/dev/sdb of=/tmp/usb_backup.img bs=4M status=progress
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `dd` | **Data duplicator** — byte-for-byte copy | Exact disk/partition clone |
	| `if=/dev/sdb` | **Input file** — source device | The disk to copy FROM |
	| `of=/tmp/usb_backup.img` | **Output file** — destination | The image file to create |
	| `bs=4M` | **Block size** 4 megabytes | Larger blocks = faster transfer. 4M is optimal for most cases. |
	| `status=progress` | Show transfer progress | Without this, dd runs silently — no idea if it's working |
	
	🟢 **WHY use dd:** Creates forensic-grade exact copy — every byte including deleted files, filesystem metadata, boot sectors. Needed for: forensic imaging, backup bootable drives, creating live USB images.
	
	⚠️ **DANGER:** `dd` is called "disk destroyer" by some. `if=` and `of=` swapped = overwrites your source. Double-check before running!
	
	---
	
	```bash
	sudo chkdsk   # Windows — not available on Linux
	sudo fsck /dev/sdb1
	sudo e2fsck -f /dev/sdb1
	```
	| Command | What It Does | When to Use |
	|---------|-------------|------------|
	| `fsck` | **Filesystem check** — scan and repair filesystem errors | After crash, power loss, or filesystem errors |
	| `e2fsck -f` | Force check ext2/3/4 filesystem | Force check even if appears clean |
	
	⚠️ **IMPORTANT:** Run fsck on UNMOUNTED partition. Running on mounted filesystem = danger of corruption.
	
	---
	
	```bash
	sudo fallocate -l 4G /swapfile
	sudo chmod 600 /swapfile
	sudo mkswap /swapfile
	sudo swapon /swapfile
	```
	| Part | Meaning | Why Each Step |
	|------|---------|---------------|
	| `fallocate -l 4G` | Create 4GB file instantly | Creates swap file container |
	| `chmod 600 /swapfile` | Owner-only permissions | Security — swap contains memory pages that may have passwords |
	| `mkswap` | Format file as swap space | Prepare it for use as virtual memory |
	| `swapon` | Enable the swap | Activate it — system can now use it |
	
	🟢 **WHY create swap:** When RAM fills up, Linux uses swap space on disk as overflow. Without swap, processes get killed (OOM killer). With swap, system stays alive but slower.
	
	---
	
	## 9. Process & System Monitoring
	
	```bash
	htop
	btop
	```
	| Tool | What It Shows | Key Features |
	|------|--------------|-------------|
	| `htop` | CPU, RAM, swap, process list | Color-coded bars, tree view, easy kill |
	| `btop` | CPU, RAM, network, disk, processes | Beautiful graphs, better visuals |
	
	**htop keyboard shortcuts:**
	```
	F9 or k  → Kill process (select with arrows first)
	F6       → Sort by column (CPU, MEM, etc.)
	F5       → Tree view (show parent-child relationships)
	/        → Search for process
	u        → Filter by user
	Space    → Tag a process (for batch operations)
	F10      → Quit
	```
	
	🟢 **WHY use htop over `top`:** Htop has color, mouse support, and much easier navigation. `top` is always available (even minimal systems) but htop is nicer to use.
	
	---
	
	```bash
	ps aux
	ps aux | grep process-name
	ps aux --sort=-%cpu | head -10
	```
	| Flag | Meaning | Why |
	|------|---------|-----|
	| `ps` | **Process status** snapshot (not live) | |
	| `a` | All processes, all users | Without a: only YOUR processes |
	| `u` | User-oriented format — shows username, CPU%, MEM% | More readable |
	| `x` | Include processes without controlling terminal | Includes background daemons |
	| `--sort=-%cpu` | Sort by CPU usage descending | Find what's hogging CPU |
	| `\| head -10` | Show only top 10 results | Focus on worst offenders |
	
	🟢 **WHY `ps aux` vs `htop`:** `ps aux` is scriptable — pipe to grep, awk, sort. Good for automation. `htop` is interactive only.
	
	---
	
	```bash
	kill PID
	kill -9 PID
	killall firefox
	pkill -f "process name"
	```
	| Command | Signal | Behavior | When to Use |
	|---------|--------|----------|------------|
	| `kill PID` | SIGTERM (15) | Polite — asks process to stop cleanly | Normal termination |
	| `kill -9 PID` | SIGKILL (9) | Force — OS kills process immediately, no cleanup | When process is frozen/won't respond to SIGTERM |
	| `killall firefox` | SIGTERM by name | Kills all processes named "firefox" | Kill all instances of an app |
	| `pkill -f "name"` | Match full command | Matches against full command line | Kill process by partial name or argument |
	
	⚠️ **WHY avoid `kill -9` normally:** It's instant — no chance to save files, close connections, or clean up. Use SIGTERM first, wait a few seconds, then SIGKILL if needed.
	
	---
	
	```bash
	free -h
	cat /proc/meminfo
	vmstat -s
	```
	| Command | What It Shows | Why |
	|---------|--------------|-----|
	| `free -h` | RAM and swap: total, used, free, available | Quick memory check |
	| `/proc/meminfo` | Detailed memory breakdown | Deeper analysis |
	| `vmstat -s` | Virtual memory statistics | Swap activity, page faults |
	
	🔴 **`free -h` output:**
	```
	total  used  free  shared  buff/cache  available
	Mem:           15Gi  4.2Gi  8.1Gi  234Mi   3.1Gi      10Gi
	Swap:          4.0Gi    0B  4.0Gi
	```
	🟢 **WHY "available" matters more than "free":** "free" = completely unused. "available" = free + memory that CAN be freed (cache). "available" is the real number of free memory.
	
	---
	
	```bash
	top
	top -u username
	```
	| Flag | Meaning |
	|------|---------|
	| `top` | Live process monitor (updates every 3 seconds) |
	| `-u username` | Show only that user's processes |
	
	**Inside top:**
	```
	P  → Sort by CPU
	M  → Sort by Memory
	k  → Kill a process (type PID)
	q  → Quit
	1  → Show individual CPU cores
	```
	
	---
	
	```bash
	watch -n 2 'ss -tulnp'
	watch -n 1 'ps aux --sort=-%cpu | head -10'
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `watch` | Run command repeatedly and display output | Monitor changing values |
	| `-n 2` | Refresh every 2 seconds | |
	| `'command'` | Command in quotes to run | Single quotes to prevent early expansion |
	
	🟢 **WHY `watch`:** Instead of manually running the same command every few seconds, `watch` does it automatically. Great for monitoring during tests.
	
	---
	
	## 10. Log & Journal Commands
	
	```bash
	journalctl -xe
	```
	| Flag | Meaning | Why |
	|------|---------|-----|
	| `journalctl` | Query systemd journal — all system logs | |
	| `-x` | **eXplanatory** — adds explanatory help text for error codes | Makes errors more understandable |
	| `-e` | Jump to **End** of log | See most recent entries immediately |
	
	🟢 **WHY this is your first debugging command:** When something breaks, `journalctl -xe` shows the most recent errors with explanations. Start here.
	
	---
	
	```bash
	journalctl -u ssh
	journalctl -u ssh -f
	journalctl -u ssh -n 50
	journalctl -u ssh --since "1 hour ago"
	journalctl -u ssh --since "2025-01-01" --until "2025-01-02"
	```
	| Flag | Meaning | Why |
	|------|---------|-----|
	| `-u ssh` | **Unit** — filter logs for ssh.service only | Focus on one service instead of all logs |
	| `-f` | **Follow** — stream new log entries live | Watch a service in real-time as it logs |
	| `-n 50` | Last 50 lines | See recent entries without scrolling through everything |
	| `--since "1 hour ago"` | Only logs from last hour | When you know roughly when problem started |
	
	🟢 **WHY `-f` is powerful:** Run `journalctl -u ssh -f` then try to SSH in another terminal. See exactly what SSH logs in real time — invaluable for debugging.
	
	---
	
	```bash
	journalctl -b
	journalctl -b -1
	journalctl -b --list-boots
	```
	| Command | What It Shows |
	|---------|--------------|
	| `-b` | Logs from current boot only |
	| `-b -1` | Logs from PREVIOUS boot (before last restart) |
	| `--list-boots` | All available boot records with timestamps |
	
	🟢 **WHY `-b -1`:** System crashed and rebooted? The logs about the crash are from last boot, not current one. `-b -1` gives you those logs.
	
	---
	
	```bash
	journalctl --disk-usage
	sudo journalctl --vacuum-size=100M
	sudo journalctl --vacuum-time=7d
	```
	| Command | What It Does | Why |
	|---------|-------------|-----|
	| `--disk-usage` | Show how much disk space journals use | Logs accumulate over time |
	| `--vacuum-size=100M` | Keep only 100MB of logs, delete rest | Free disk space |
	| `--vacuum-time=7d` | Keep only last 7 days of logs | Time-based cleanup |
	
	---
	
	```bash
	tail -f /var/log/syslog
	tail -100 /var/log/auth.log
	grep "Failed password" /var/log/auth.log
	grep "error" /var/log/syslog | tail -20
	```
	| Command | What It Shows | Why |
	|---------|--------------|-----|
	| `tail -f syslog` | Live stream of system log | General system events as they happen |
	| `auth.log` | Authentication events | SSH logins, sudo usage, password attempts |
	| `grep "Failed password"` | Failed SSH login attempts | Find brute force attacks or your own login failures |
	
	---
	
	```bash
	dmesg
	dmesg -T
	dmesg | tail -30
	dmesg | grep -i "error\|fail\|warn"
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `dmesg` | **Kernel ring buffer** — hardware and kernel messages | |
	| `-T` | **Timestamp** — human-readable time instead of seconds since boot | Much easier to correlate with other events |
	| `grep -i` | Case-insensitive grep | Catches "Error", "ERROR", "error" |
	
	🟢 **WHY dmesg:** When hardware fails (USB not working, driver error, kernel panic), the evidence is in dmesg. Also check when:
	- USB device not detected
	- Wi-Fi driver issues  
	- Disk errors
	- Kernel module problems
	
	---
	
	## 11. SSH Configuration Commands
	
	```bash
	sudo systemctl enable --now ssh
	ssh-keygen -t ed25519 -C "parrot-pentest"
	ssh-keygen -t rsa -b 4096
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `-t ed25519` | Key **type** — Ed25519 algorithm | Modern, fast, secure. Shorter keys than RSA but equally or more secure |
	| `-t rsa -b 4096` | RSA 4096-bit | Legacy compatibility. Use when ed25519 not supported |
	| `-C "comment"` | **Comment** — label added to public key | Identify which key is which. Appears in `authorized_keys` |
	
	🟢 **WHY Ed25519 over RSA:** Ed25519 keys are shorter (easier to copy), faster to generate and use, and have no known weaknesses. Use Ed25519 unless you specifically need RSA compatibility.
	
	---
	
	```bash
	ssh-copy-id -i ~/.ssh/id_ed25519.pub user@target.com
	ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 user@target.com
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `ssh-copy-id` | Copies public key to remote server's `authorized_keys` | Enables password-less key authentication |
	| `-i ~/.ssh/id_ed25519.pub` | Which **identity** (key) to copy | Explicitly choose the public key file |
	| `-p 2222` | Remote SSH **port** | When SSH runs on non-standard port |
	
	🔵 **WHAT happens:** Connects to server with password, appends your public key to `~/.ssh/authorized_keys` on server. Next SSH connection uses key (no password needed).
	
	---
	
	```bash
	nano ~/.ssh/config
	```
	**SSH Config file explained:**
	```
	Host htb                    ← Alias (type "ssh htb" to use this)
	HostName 10.10.10.100     ← Actual IP or hostname
	User root                 ← Default username
	IdentityFile ~/.ssh/htb   ← Which private key to use
	Port 22                   ← SSH port
	
	Host jump                   ← Jump server alias
	HostName jump.example.com
	User admin
	
	Host internal               ← Internal server via jump
	HostName 192.168.1.100
	ProxyJump jump             ← Connect through "jump" alias first
	```
	
	🟢 **WHY use SSH config:** Instead of `ssh -i ~/.ssh/htb_key -p 22 root@10.10.10.100`, just type `ssh htb`. Saves time, reduces errors.
	
	---
	
	```bash
	ssh -L 8080:localhost:80 user@server
	ssh -L 3306:db.internal:3306 user@jump-server
	```
	| Part | Meaning | Real Use Case |
	|------|---------|---------------|
	| `-L 8080:localhost:80` | **Local port forward** | Access web server at server:80 via YOUR localhost:8080 |
	| `-L 3306:db.internal:3306` | Forward through jump to internal DB | Access internal database through SSH tunnel |
	
	🟢 **WHY:** Access services that are only available on the remote network. Your traffic: `localhost:8080` → SSH tunnel → `server:80`. From your browser it looks like a local service.
	
	---
	
	```bash
	ssh -R 4444:localhost:4444 user@attacker.com
	```
	| Part | Meaning | Why for Pentesting |
	|------|---------|-------------------|
	| `-R 4444:localhost:4444` | **Remote port forward** | Create reverse tunnel: attacker:4444 → SSH → victim:4444 |
	
	🟢 **WHY:** When victim is behind firewall (can't receive connections), they SSH out to you. You can then reach them through that tunnel.
	
	---
	
	```bash
	ssh -D 1080 user@server -N -f
	```
	| Flag | Meaning | Why |
	|------|---------|-----|
	| `-D 1080` | **Dynamic** SOCKS proxy on port 1080 | All traffic through SOCKS5 proxy at localhost:1080 |
	| `-N` | No command — just tunnel | Don't open a shell, just keep connection for tunneling |
	| `-f` | Fork to background | Free your terminal while tunnel stays open |
	
	🟢 **WHY:** Route any application through this proxy to access server's network. Configure browser proxy to localhost:1080 → browse internal sites.
	
	---
	
	**SSHD Config important settings (`/etc/ssh/sshd_config`):**
	```
	Port 2222                    → Change from default 22
	# WHY: Reduces automated scan noise. Attackers scan port 22.
	
	PermitRootLogin no           → Disable direct root SSH
	# WHY: If attacker guesses password, they get root immediately.
	# Better: SSH as regular user, then sudo.
	
	PasswordAuthentication no    → Keys only
	# WHY: Passwords can be brute-forced. Keys cannot (unless very weak passphrase).
	
	MaxAuthTries 3               → 3 attempts then disconnect
	# WHY: Limits brute-force attempts per connection.
	
	AllowUsers username1 username2  → Whitelist
	# WHY: Even if attacker creates account somehow, SSH denies them.
	```
	
	---
	
	## 12. VPN Commands
	
	```bash
	sudo openvpn --config lab.ovpn
	sudo openvpn --config lab.ovpn --daemon
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `openvpn` | OpenVPN client | Connect to VPN server |
	| `--config lab.ovpn` | VPN configuration file | Contains server address, certificates, keys |
	| `--daemon` | Run in background | Free your terminal. VPN stays connected. |
	
	🟢 **WHY OpenVPN:** Used by HackTheBox, TryHackMe, and most corporate VPNs. The `.ovpn` file you download contains everything needed — server IP, your certificate, encryption settings.
	
	---
	
	```bash
	ip a show tun0
	```
	🟢 **WHY check tun0:** After `openvpn` starts, it creates a `tun0` (tunnel) interface. If `tun0` doesn't exist → VPN not connected. If it exists with an IP → VPN working. This IP is your `LHOST` for Metasploit payloads.
	
	---
	
	```bash
	sudo pkill openvpn
	```
	🔵 **WHAT:** Kill all openvpn processes — disconnects VPN.
	
	---
	
	```bash
	sudo wg-quick up wg0
	sudo wg-quick down wg0
	sudo wg show
	sudo systemctl enable wg-quick@wg0
	```
	| Command | What It Does | Why |
	|---------|-------------|-----|
	| `wg-quick up wg0` | Start WireGuard interface wg0 | Connect to WireGuard VPN |
	| `wg-quick down wg0` | Stop WireGuard interface | Disconnect |
	| `wg show` | Show WireGuard status | Verify connection, see peers, check handshake time |
	| `systemctl enable wg-quick@wg0` | Auto-start on boot | `@wg0` = parameter passing to service template |
	
	🟢 **WHY WireGuard over OpenVPN:** Faster (modern crypto), simpler config, less overhead, better for mobile. OpenVPN is more compatible with enterprise setups.
	
	---
	
	## 13. Wi-Fi & Wireless Commands
	
	```bash
	rfkill list all
	sudo rfkill unblock wifi
	sudo rfkill unblock all
	```
	| Command | What It Does | Why |
	|---------|-------------|-----|
	| `rfkill list all` | Show all radio kill switch states | Check if Wi-Fi is blocked at hardware or software level |
	| `rfkill unblock wifi` | Remove software block on Wi-Fi | Some laptops/distros disable Wi-Fi by software kill switch |
	
	🟢 **WHY check rfkill first:** "Hard blocked" = physical Wi-Fi switch is off (check laptop). "Soft blocked" = software disabled it. `rfkill unblock wifi` fixes soft block.
	
	---
	
	```bash
	sudo airmon-ng check kill
	sudo airmon-ng start wlan0
	sudo airmon-ng stop wlan0mon
	```
	| Command | What It Does | Why |
	|---------|-------------|-----|
	| `check kill` | Kill NetworkManager and wpa_supplicant | They fight with monitor mode — must stop them first |
	| `start wlan0` | Enable monitor mode → creates `wlan0mon` | Needed for Wi-Fi packet capture and injection |
	| `stop wlan0mon` | Disable monitor mode | Restore normal Wi-Fi operation |
	
	---
	
	```bash
	sudo service NetworkManager restart
	sudo systemctl restart NetworkManager
	```
	🟢 **WHY restart NM after airmon-ng stop:** airmon-ng killed NetworkManager. After finishing Wi-Fi testing, restart it to reconnect to Wi-Fi normally.
	
	---
	
	```bash
	nmcli radio wifi on
	nmcli radio wifi off
	nmcli dev wifi list
	nmcli dev wifi connect "SSID" password "yourpassword"
	```
	| Command | What It Does |
	|---------|-------------|
	| `radio wifi on/off` | Enable/disable Wi-Fi radio |
	| `dev wifi list` | Scan and list nearby Wi-Fi networks |
	| `wifi connect "SSID" password "pass"` | Connect to Wi-Fi network |
	
	🟢 **WHY nmcli for Wi-Fi:** Works from command line without GUI. Useful when desktop environment is broken or you're in text mode.
	
	---
	
	## 14. Boot & GRUB Commands
	
	```bash
	sudo update-grub
	```
	🔵 **WHAT:** Scans for installed operating systems and kernels, regenerates `/boot/grub/grub.cfg`.
	
	🟢 **WHY:** Run after: installing a new kernel, adding Windows to dual boot, changing GRUB settings. Without this, GRUB menu won't reflect changes.
	
	---
	
	```bash
	sudo nano /etc/default/grub
	sudo update-grub  # Always run after editing!
	```
	**Key GRUB settings explained:**
	```
	GRUB_DEFAULT=0
	# WHY: Which menu entry boots automatically. 0=first entry, "saved"=last chosen
	
	GRUB_TIMEOUT=5
	# WHY: Seconds to show menu before auto-booting. 0=instant boot (no menu shown)
	# Set to 10 if you need time to select entries
	
	GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
	# WHY: Kernel parameters for every boot
	# "quiet" = suppress most boot messages (cleaner boot)
	# "splash" = show graphical splash screen
	# Add "nomodeset" here if having display driver issues
	
	GRUB_CMDLINE_LINUX=""
	# WHY: Additional parameters added to GRUB_CMDLINE_LINUX_DEFAULT
	```
	
	---
	
	```bash
	sudo grub-install /dev/sda
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `grub-install` | Install GRUB bootloader | Write GRUB to the Master Boot Record |
	| `/dev/sda` | Target DISK (not partition!) | Install to the disk's MBR, not a partition |
	
	🟢 **WHY:** After Windows overwrites GRUB (common in dual boot), or after drive replacement. Reinstalls GRUB so Linux can boot again.
	
	⚠️ **DANGER:** Wrong disk = overwrites wrong MBR. Triple-check `fdisk -l` to confirm which disk is `/dev/sda` before running.
	
	---
	
	```bash
	sudo os-prober
	```
	🔵 **WHAT:** Scans for other operating systems (Windows, other Linux) on all disks.
	
	🟢 **WHY:** Run before `update-grub` when setting up dual boot. Finds Windows and adds it to GRUB menu automatically.
	
	---
	
	## 15. Shell & Environment Commands
	
	```bash
	nano ~/.bashrc
	source ~/.bashrc
	```
	| Command | What It Does | Why |
	|---------|-------------|-----|
	| `nano ~/.bashrc` | Edit bash configuration for YOUR user | Add aliases, PATH changes, functions |
	| `source ~/.bashrc` | **Reload** .bashrc without restarting terminal | Apply changes immediately in current session |
	
	🟢 **WHY source instead of restart:** Opening new terminal would apply changes too, but `source` does it instantly in current window.
	
	---
	
	```bash
	export PATH=$PATH:~/tools/bin
	export PATH=$PATH:/opt/newtools
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `export` | Make variable available to child processes | Shell scripts and apps can see it |
	| `PATH` | The variable listing directories searched for commands | |
	| `$PATH:~/tools/bin` | **Append** new directory to existing PATH | Keep existing paths, add new one at end |
	
	🟢 **WHY:** When you install a tool manually (not via apt), its binary might not be in a standard PATH directory. Adding its directory to PATH lets you run it by name from anywhere.
	
	⚠️ **IMPORTANT:** This only lasts for current session. Add to `~/.bashrc` to make permanent:
	```bash
	echo 'export PATH=$PATH:~/tools/bin' >> ~/.bashrc
	```
	
	---
	
	```bash
	alias ll='ls -alF'
	alias update='sudo parrot-upgrade'
	alias myip='curl -s ifconfig.me'
	alias ports='ss -tulnp'
	```
	🔵 **WHAT:** Creates a shortcut — `ll` runs `ls -alF` automatically.
	
	🟢 **WHY:** Type less, work faster, avoid remembering long flags. Add to `~/.bashrc` to make permanent.
	
	---
	
	```bash
	tmux new -s pentest
	tmux attach -t pentest
	tmux ls
	```
	| Command | What It Does | Why for Pentesting |
	|---------|-------------|-------------------|
	| `new -s pentest` | Create new session named "pentest" | Start organized workspace |
	| `attach -t pentest` | Reconnect to existing session | Reattach after disconnect — session keeps running |
	| `ls` | List all tmux sessions | See what's running |
	
	🟢 **WHY tmux is essential:** 
	1. Sessions survive SSH disconnects — tools keep running
	2. Multiple windows in one terminal — nmap in window 1, metasploit in window 2
	3. Split panes — see multiple outputs simultaneously
	4. Named sessions — organized workflow
	
	**tmux shortcuts (after pressing Ctrl+a as prefix):**
	```
	Ctrl+a c  → New window
	Ctrl+a n  → Next window
	Ctrl+a p  → Previous window
	Ctrl+a |  → Split vertically
	Ctrl+a -  → Split horizontally
	Ctrl+a d  → Detach session (keep running)
	Ctrl+a ?  → Help (all shortcuts)
	```
	
	---
	
	## 16. File & Directory Commands
	
	```bash
	ls -la
	ls -la /etc/
	```
	| Flag | Meaning | What You See Extra |
	|------|---------|-------------------|
	| `ls` | List directory contents | |
	| `-l` | **Long format** — details | Permissions, owner, size, date |
	| `-a` | **All** — show hidden files (starting with `.`) | `.bashrc`, `.ssh/`, `.config/` |
	
	🔴 **Output explained:**
	```
	drwxr-xr-x  2 user user 4096 Jan  1 10:00 Documents/
	│├──────────  │ │    │    │       │          └─ Name
	││           │ │    │    │       └─ Date modified
	││           │ │    │    └─ Size in bytes
	││           │ │    └─ Group owner
	││           │ └─ User owner
	││           └─ Hard link count
	│└─ Permissions: d=dir, r=read, w=write, x=execute
	└─ d=directory, -=file, l=symlink
	```
	
	---
	
	```bash
	find / -name "*.conf" 2>/dev/null
	find /home -type f -name "*.txt"
	find / -perm -4000 2>/dev/null
	find / -size +100M 2>/dev/null
	find / -mtime -7 2>/dev/null
	```
	| Flag | Meaning | Use Case |
	|------|---------|----------|
	| `-name "*.conf"` | Match by filename pattern | Find all config files |
	| `-type f` | **Type file** (not directory, symlink) | Filter to regular files only |
	| `-type d` | **Type directory** | Find directories only |
	| `-perm -4000` | Has SUID bit | Find privilege escalation vectors |
	| `-size +100M` | Larger than 100MB | Find space hogs |
	| `-mtime -7` | Modified in last 7 days | Find recently changed files |
	| `2>/dev/null` | Hide "Permission denied" errors | Clean output |
	
	---
	
	```bash
	grep -r "password" /etc/
	grep -r "password" /var/www/ --include="*.php"
	grep -i "error" /var/log/syslog | tail -20
	grep -n "pattern" file.txt
	```
	| Flag | Meaning | Why |
	|------|---------|-----|
	| `-r` | **Recursive** — search directories | Search entire directory tree |
	| `--include="*.php"` | Only PHP files | Limit search to relevant file types |
	| `-i` | **Case-insensitive** | Matches "error", "Error", "ERROR" |
	| `-n` | Show **line numbers** | Know where the match is in file |
	
	🟢 **WHY `grep` matters for security:** Find hardcoded passwords, misconfigurations, and sensitive data buried in files.
	
	---
	
	```bash
	cat file.txt
	cat /etc/passwd
	less file.txt
	head -20 file.txt
	tail -20 file.txt
	tail -f /var/log/syslog
	```
	| Command | What It Does | When to Use |
	|---------|-------------|------------|
	| `cat` | Print entire file to screen | Short files |
	| `less` | Scrollable viewer | Long files (q to quit, / to search) |
	| `head -20` | First 20 lines | Check file format, see beginning |
	| `tail -20` | Last 20 lines | See most recent log entries |
	| `tail -f` | Follow — stream new lines live | Watch log files in real time |
	
	---
	
	## 17. Metasploit Setup Commands
	
	```bash
	sudo systemctl start postgresql
	sudo systemctl enable postgresql
	sudo msfdb init
	```
	| Command | What It Does | Why Each Step |
	|---------|-------------|---------------|
	| `systemctl start postgresql` | Start PostgreSQL database server | Metasploit stores all scan data, credentials, sessions in PostgreSQL |
	| `systemctl enable postgresql` | Start PostgreSQL on every boot | Without this, postgresql doesn't start after reboot and msfconsole loses DB |
	| `msfdb init` | Initialize Metasploit's database | Creates the `msf` database, user, and tables. Must do this once on fresh install. |
	
	🟢 **WHY PostgreSQL matters:** Without DB, Metasploit works but: can't save scan results, no `hosts`/`services` commands, no workspace organization, no credential storage between sessions.
	
	---
	
	```bash
	msfconsole -q -x "db_status; exit"
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `-q` | Quiet (no banner) | Faster test |
	| `-x "commands"` | Execute these commands and exit | Non-interactive test |
	| `db_status` | Check database connection | Verify DB is working |
	
	🟢 **WHY run this test:** Quick way to confirm Metasploit and PostgreSQL are talking without fully opening msfconsole.
	
	🔴 **Good output:** `[*] Connected to msf. Connection type: postgresql.`
	🔴 **Bad output:** `[*] No database support`
	
	---
	
	```bash
	sudo msfdb reinit
	sudo msfdb delete
	sudo msfdb init
	```
	| Command | When to Use |
	|---------|------------|
	| `msfdb reinit` | Database exists but is broken — reset it |
	| `msfdb delete` then `msfdb init` | Complete fresh start — delete everything |
	
	---
	
	## 18. Tor & AnonSurf Commands
	
	```bash
	sudo systemctl start tor
	sudo systemctl enable tor
	sudo systemctl status tor
	```
	🔵 **WHAT:** Start/enable/check Tor daemon — the background Tor relay process.
	
	🟢 **WHY separate from AnonSurf:** Tor daemon can run without AnonSurf for specific-app proxying. AnonSurf uses the running Tor daemon but adds system-wide iptables routing.
	
	---
	
	```bash
	curl --socks5-hostname 127.0.0.1:9050 https://check.torproject.org/api/ip
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `--socks5-hostname 127.0.0.1:9050` | Route through local Tor SOCKS proxy | Test if Tor is working |
	| `check.torproject.org/api/ip` | Tor Project's IP checker | Returns JSON with your apparent IP |
	
	🟢 **WHY:** Verify that Tor is routing traffic. If result shows your real IP → Tor not working. If shows different IP → Tor working.
	
	---
	
	```bash
	sudo anonsurf start
	sudo anonsurf stop
	sudo anonsurf changeid
	sudo anonsurf myip
	sudo anonsurf status
	```
	| Command | What Happens Internally | Why Use |
	|---------|------------------------|---------|
	| `start` | Modifies iptables to redirect ALL traffic through Tor port 9050. Switches DNS to anonymous DNS. | Route all system traffic anonymously |
	| `stop` | Removes iptables rules. Restores original DNS. | Return to normal routing |
	| `changeid` | Sends NEWNYM to Tor control port → gets new circuit → new exit IP | Change apparent location |
	| `myip` | Requests your IP through Tor to external service | Verify anonymization is working |
	| `status` | Shows: Tor status, current IP, iptables state | Check everything is configured correctly |
	
	---
	
	```bash
	sudo nano /etc/proxychains4.conf
	```
	**Config file sections explained:**
	```
	# CHAIN TYPE (choose one):
	strict_chain
	# Every proxy in list used IN ORDER. If one fails, connection fails.
	# WHY USE: Maximum control over routing path.
	
	dynamic_chain  
	# Skip dead proxies. Move to next one.
	# WHY USE: More reliable when using multiple proxies.
	
	random_chain
	# Random order each connection.
	# WHY USE: Harder to trace — different route every time.
	
	# PROXY LIST (at bottom of file):
	socks5  127.0.0.1  9050    # Tor proxy
	socks4  10.10.10.1  1080   # Another SOCKS4 proxy
	http    203.0.113.1  3128  # HTTP proxy
	```
	
	```bash
	proxychains4 nmap -sT -Pn target.com
	proxychains4 ssh user@target.com
	proxychains4 sqlmap -u "http://target.com/?id=1"
	```
	🟢 **WHY `-sT` with proxychains:** Proxychains works at the TCP socket level. Raw packet scans (`-sS`, UDP, etc.) bypass the TCP socket and go directly to the network — proxychains CAN'T intercept them. Always use TCP connect scan (`-sT`) with proxychains.
	
	---
	
	## 19. Docker Commands
	
	```bash
	sudo apt install docker.io -y
	sudo systemctl enable --now docker
	sudo usermod -aG docker $USER
	newgrp docker
	```
	| Command | Why |
	|---------|-----|
	| `enable --now docker` | Start docker AND enable for boot in one command |
	| `usermod -aG docker $USER` | Run docker without sudo (security improvement) |
	| `newgrp docker` | Apply group change immediately without logout |
	
	---
	
	```bash
	docker pull kalilinux/kali-rolling
	docker run -it kalilinux/kali-rolling bash
	docker run -it --network host kalilinux/kali-rolling bash
	```
	| Part | Meaning | Why |
	|------|---------|-----|
	| `pull` | Download image from Docker Hub | Get latest version of image |
	| `run` | Create and start container | Each `run` creates NEW container |
	| `-it` | **Interactive** + **TTY** | Opens interactive terminal. Without -it, container runs and exits immediately. |
	| `--network host` | Use host's network | Container shares host's IP and interfaces. Important for network tools to see host's network. |
	| `bash` | Command to run | Start bash shell inside container |
	
	🟢 **WHY `--network host` for pentest containers:** Without it, container is on its own isolated network. With it, tools inside container can scan the same network as your host.
	
	---
	
	```bash
	docker ps
	docker ps -a
	docker images
	docker stop container_id
	docker rm container_id
	docker rmi image_name
	docker exec -it container_id bash
	docker system prune -a
	```
	| Command | What It Does | When to Use |
	|---------|-------------|------------|
	| `ps` | Running containers | See what's currently active |
	| `ps -a` | All containers (including stopped) | See everything, including exited |
	| `images` | Downloaded images | See what's taking disk space |
	| `stop` | Gracefully stop container | Normal shutdown |
	| `rm` | Remove stopped container | Clean up |
	| `rmi` | Remove downloaded image | Free disk space |
	| `exec -it ID bash` | Open new shell in RUNNING container | Debug or work in active container |
	| `system prune -a` | Remove ALL stopped containers, unused images, networks | Free space completely |
	
	---
	
	## 20. Troubleshooting Command Toolkit
	
	### 🔧 APT Problems
	
	```bash
	sudo rm /var/lib/dpkg/lock
	sudo rm /var/lib/dpkg/lock-frontend
	sudo rm /var/cache/apt/archives/lock
	sudo dpkg --configure -a
	sudo apt install -f
	```
	| Command | What It Fixes | When to Use |
	|---------|--------------|-------------|
	| `rm lock files` | **Removes lock files** left by crashed apt process | "Could not get lock" error |
	| `dpkg --configure -a` | **Configure all** half-installed packages | "dpkg was interrupted" error |
	| `apt install -f` | **Fix** broken dependencies | "Unmet dependencies" error |
	
	🟢 **WHY lock files exist:** Only one apt/dpkg process should run at a time — lock prevents two apt commands running simultaneously. If apt crashes, the lock stays and future apt commands fail. Removing lock is safe ONLY if no apt is actually running.
	
	---
	
	```bash
	sudo apt clean
	sudo rm -rf /var/lib/apt/lists/*
	sudo apt update
	```
	| Command | What It Does | Why This Fixes Issues |
	|---------|-------------|----------------------|
	| `apt clean` | Remove downloaded .deb files | Free space, remove corrupted packages |
	| `rm -rf /var/lib/apt/lists/*` | Delete all package list metadata | Forces complete re-download of fresh package lists |
	| `apt update` | Download fresh package lists | Rebuilds the lists you just deleted |
	
	🟢 **WHY:** Corrupt package list files cause mysterious apt errors. Nuking them and re-downloading always gives you clean state.
	
	---
	
	### 🔧 Network Problems
	
	```bash
	sudo systemctl restart NetworkManager
	sudo ip link set eth0 down && sudo ip link set eth0 up
	sudo dhclient eth0
	```
	| Command | What It Fixes | When to Use |
	|---------|--------------|-------------|
	| `restart NetworkManager` | Restart entire network management system | General network issues, Wi-Fi dropping |
	| `link down && link up` | Soft-reset interface | Interface shows up but no traffic |
	| `dhclient eth0` | Request new IP via DHCP | "No IP address" after interface reset |
	
	---
	
	```bash
	sudo systemctl restart systemd-resolved
	sudo systemd-resolve --flush-caches
	sudo resolvectl flush-caches
	sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
	```
	| Command | What It Fixes |
	|---------|--------------|
	| `restart systemd-resolved` | DNS resolution service crash |
	| `flush-caches` | Stale DNS cache serving wrong IPs |
	| `ln -sf` | Broken `/etc/resolv.conf` symlink (common after updates) |
	
	🟢 **WHY the symlink fix:** In modern Parrot/Debian, `/etc/resolv.conf` should be a symlink to systemd-resolved's stub. If it's a regular file, DNS may stop working after network changes. The symlink ensures it always uses the live resolver.
	
	---
	
	### 🔧 Permission Problems
	
	```bash
	sudo chown -R $USER:$USER ~/problem-directory/
	sudo chmod +x script-that-wont-run.sh
	sudo ls -la /problem/path/
	```
	| Command | What It Fixes | Why |
	|---------|--------------|-----|
	| `chown -R $USER:$USER` | Ownership issues | Files owned by root in your home = can't edit |
	| `chmod +x` | "Permission denied" when running script | Script needs execute bit |
	| `ls -la` | Shows EXACT permissions and owner | Know what you're dealing with before fixing |
	
	---
	
	### 🔧 Service Won't Start
	
	```bash
	# Debug sequence:
	systemctl status service-name          # Step 1: See current state and last errors
	journalctl -u service-name -n 50       # Step 2: More detailed logs
	journalctl -xe                         # Step 3: System-wide recent errors
	service-name --test                    # Step 4: Many services have test mode
	service-name configtest                # Step 5: Config syntax test (apache, nginx)
	```
	
	🟢 **WHY this sequence:** `status` gives quick overview. `journalctl` gives full logs. Most service failures are: config syntax errors, port already in use, missing dependency, permission issues. Logs always tell you which one.
	
	---
	
	```bash
	ss -tulnp | grep :80
	lsof -i :80
	sudo kill $(lsof -t -i:80)
	```
	| Command | What It Does | Why |
	|---------|-------------|-----|
	| `ss -tulnp \| grep :80` | Find what's using port 80 | Service can't start if port is taken |
	| `lsof -i :80` | List open file on port 80 (shows process) | More detailed than ss |
	| `kill $(lsof -t -i:80)` | Kill whatever is using port 80 | Free the port |
	
	🟢 **WHY this matters:** "Address already in use" error = another process has that port. Find and kill it first.
	
	---
	
	### 🔧 Disk Full
	
	```bash
	df -h                                  # Step 1: Which partition is full?
	du -sh /* 2>/dev/null | sort -h        # Step 2: What's taking space?
	sudo apt clean                         # Step 3: Clear package cache
	sudo journalctl --vacuum-size=100M     # Step 4: Shrink logs
	find /tmp -mtime +7 -delete            # Step 5: Delete old temp files
	rm -rf ~/.local/share/Trash/*          # Step 6: Empty trash
	```
	
	🟢 **WHY this order:** Start broad (which partition), then narrow (which directory), then clear known safe targets (cache, logs, tmp). Only then look at your own files.
	
	---
	
	### 🔧 System Log Analysis
	
	```bash
	# Find what caused a crash:
	journalctl -b -1 --priority=err        # Errors from LAST boot
	journalctl -b --since "10 minutes ago" # Recent messages
	dmesg -T | grep -i "killed\|oom"      # Out of Memory kills
	grep "segfault" /var/log/syslog        # Application crashes
	```
	
	| Command | What It Reveals |
	|---------|----------------|
	| `priority=err` | Only error-level messages — filters noise |
	| `"killed\|oom"` | OOM killer activity — what got killed due to RAM |
	| `"segfault"` | Segmentation faults — application bugs/crashes |
	
	---
	
	## 21. Complete Cheat Sheet
	
	```
	═══════════════════════════════════════════════════════
	SYSTEM UPDATE & PACKAGES
	═══════════════════════════════════════════════════════
	Update all:         sudo parrot-upgrade
	Install:            sudo apt install -y package
	Remove+config:      sudo apt purge package
	Fix broken:         sudo apt install -f && sudo dpkg --configure -a
	Find package:       apt search keyword
	Clear cache:        sudo apt clean && sudo apt autoremove
	Fix lock:           sudo rm /var/lib/dpkg/lock* && sudo dpkg --configure -a
	
	═══════════════════════════════════════════════════════
	NETWORK COMMANDS
	═══════════════════════════════════════════════════════
	See interfaces:     ip a
	See routing:        ip r
	Test internet:      ping -c 4 8.8.8.8
	Test DNS:           ping -c 4 google.com || dig google.com
	See open ports:     ss -tulnp
	Flush DNS:          sudo resolvectl flush-caches
	Restart network:    sudo systemctl restart NetworkManager
	Get new IP:         sudo dhclient eth0
	
	═══════════════════════════════════════════════════════
	SERVICE MANAGEMENT
	═══════════════════════════════════════════════════════
	Start:              sudo systemctl start service
	Stop:               sudo systemctl stop service
	Restart:            sudo systemctl restart service
	Enable+start:       sudo systemctl enable --now service
	Status:             systemctl status service
	Failed services:    systemctl list-units --failed
	Service logs:       journalctl -u service -f
	
	═══════════════════════════════════════════════════════
	PENTEST SERVICES QUICK START
	═══════════════════════════════════════════════════════
	Metasploit:         sudo systemctl start postgresql && msfconsole
	BloodHound:         sudo neo4j start && bloodhound
	Nessus:             sudo systemctl start nessusd → https://localhost:8834
	AnonSurf:           sudo anonsurf start
	VPN:                sudo openvpn --config file.ovpn
	HTTP server:        python3 -m http.server 8080
	
	═══════════════════════════════════════════════════════
	USER & PERMISSIONS
	═══════════════════════════════════════════════════════
	My user info:       id && groups
	Add to group:       sudo usermod -aG groupname $USER
	Apply group now:    newgrp groupname
	Make executable:    chmod +x script.sh
	Fix ownership:      sudo chown -R $USER:$USER ~/directory/
	SSH key perms:      chmod 600 ~/.ssh/id_rsa
	
	═══════════════════════════════════════════════════════
	DISK & FILES
	═══════════════════════════════════════════════════════
	Disk space:         df -h
	Directory size:     du -sh /directory/
	Find large files:   find / -size +100M 2>/dev/null
	SUID files:         find / -perm -4000 2>/dev/null
	Find config files:  find / -name "*.conf" 2>/dev/null
	Mount USB:          sudo mount /dev/sdb1 /mnt/usb
	Safely unmount:     sudo umount /mnt/usb
	
	═══════════════════════════════════════════════════════
	LOGS & DEBUGGING
	═══════════════════════════════════════════════════════
	Recent errors:      journalctl -xe
	Service logs:       journalctl -u service -f -n 50
	Kernel messages:    dmesg -T | tail -30
	Failed services:    systemctl list-units --failed
	Auth log:           tail -f /var/log/auth.log
	What's on port X:   ss -tulnp | grep :PORT
	
	═══════════════════════════════════════════════════════
	PROCESS MANAGEMENT
	═══════════════════════════════════════════════════════
	Live monitor:       htop OR btop
	All processes:      ps aux
	Top CPU users:      ps aux --sort=-%cpu | head -10
	Kill by PID:        kill PID (polite) || kill -9 PID (force)
	Kill by name:       pkill process-name
	Port user:          lsof -i :PORT
	
	═══════════════════════════════════════════════════════
	SSH
	═══════════════════════════════════════════════════════
	Connect:            ssh user@target
	Key auth:           ssh -i ~/.ssh/key user@target
	Custom port:        ssh -p 2222 user@target
	Local forward:      ssh -L LOCAL:REMOTE_HOST:REMOTE_PORT user@server
	SOCKS proxy:        ssh -D 1080 user@server -N -f
	Generate key:       ssh-keygen -t ed25519 -C "label"
	Copy key:           ssh-copy-id -i ~/.ssh/id_ed25519.pub user@target
	
	═══════════════════════════════════════════════════════
	QUICK TROUBLESHOOT FLOW
	═══════════════════════════════════════════════════════
	Something broken?
	1. journalctl -xe                   ← What just failed?
	2. systemctl status service         ← Is the service running?
	3. ss -tulnp | grep :PORT           ← Is port in use?
	4. df -h                            ← Disk full?
	5. free -h                          ← RAM full?
	6. ping 8.8.8.8                     ← Internet working?
	7. dig google.com                   ← DNS working?
	8. ip a                             ← Network interface up?
	```
	
	---
	
	*🦜 Parrot Security OS — Configuration & Troubleshooting: Every Command Explained*
	*📌 Add this to ~/Desktop for quick reference during work*
	*⚠️ Test configuration changes in VMs before applying to production systems*
