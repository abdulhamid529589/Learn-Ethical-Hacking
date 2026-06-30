# 🦜 Parrot OS — Complete Configuration & Troubleshooting Guide

> **Version:** Parrot OS 7.0 "Echo" (Debian 13 base, KDE Plasma)
> **Author:** Senior Cybersecurity Reference Guide
> **Purpose:** Configure every aspect of Parrot OS + fix every common problem
> **Last Updated:** 2025–2026

---

## 📋 Table of Contents

### ⚙️ CONFIGURATION
1. [First Boot Setup](#1-first-boot-setup)
2. [Package Management & Updates](#2-package-management--updates)
3. [Network Configuration](#3-network-configuration)
4. [VPN Configuration](#4-vpn-configuration)
5. [Display & GPU Drivers](#5-display--gpu-drivers)
6. [Wi-Fi Adapter Configuration](#6-wi-fi-adapter-configuration)
7. [Sound & Audio Configuration](#7-sound--audio-configuration)
8. [Firewall Configuration](#8-firewall-configuration)
9. [User & Permissions Management](#9-user--permissions-management)
10. [Shell & Terminal Configuration](#10-shell--terminal-configuration)
11. [KDE Plasma Desktop Configuration](#11-kde-plasma-desktop-configuration)
12. [Service Management (systemd)](#12-service-management-systemd)
13. [Metasploit & PostgreSQL Setup](#13-metasploit--postgresql-setup)
14. [Tor & AnonSurf Configuration](#14-tor--anonsurf-configuration)
15. [SSH Server Configuration](#15-ssh-server-configuration)
16. [Docker Configuration](#16-docker-configuration)
17. [Virtual Machine / VirtualBox Setup](#17-virtual-machine--virtualbox-setup)
18. [Dual Boot Configuration](#18-dual-boot-configuration)
19. [Disk Encryption & LUKS](#19-disk-encryption--luks)
20. [Swap & Memory Configuration](#20-swap--memory-configuration)
21. [Locale, Timezone & Keyboard](#21-locale-timezone--keyboard)
22. [Custom Tool Installation](#22-custom-tool-installation)
23. [Wordlist & SecLists Setup](#23-wordlist--seclist-setup)
24. [Hashcat GPU Setup](#24-hashcat-gpu-setup)
25. [AppArmor Configuration](#25-apparmor-configuration)

### 🔧 TROUBLESHOOTING
26. [Package & APT Problems](#26-package--apt-problems)
27. [Network & Wi-Fi Problems](#27-network--wi-fi-problems)
28. [Boot & GRUB Problems](#28-boot--grub-problems)
29. [Display & Screen Problems](#29-display--screen-problems)
30. [Sound Problems](#30-sound-problems)
31. [Metasploit Problems](#31-metasploit-problems)
32. [Permission & Root Problems](#32-permission--root-problems)
33. [USB & Hardware Problems](#33-usb--hardware-problems)
34. [VPN Problems](#34-vpn-problems)
35. [DNS & Internet Problems](#35-dns--internet-problems)
36. [KDE / Desktop Problems](#36-kde--desktop-problems)
37. [Tool-Specific Problems](#37-tool-specific-problems)
38. [Performance Problems](#38-performance-problems)
39. [Disk & Filesystem Problems](#39-disk--filesystem-problems)
40. [System Log Analysis](#40-system-log-analysis)

---

# ⚙️ CONFIGURATION SECTION

---

## 1. First Boot Setup

### 🚀 Essential First Steps After Installing Parrot OS

```bash
# Step 1: Update the entire system (ALWAYS do this first)
sudo parrot-upgrade
# parrot-upgrade is Parrot's custom safe update wrapper
# It runs: apt update → apt full-upgrade → apt autoremove

# Alternative manual update:
sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y

# Step 2: Check system info
uname -a                      # Kernel version
lsb_release -a                # Parrot version info
hostnamectl                   # System hostname and OS info
inxi -Fxz                     # Full hardware summary

# Step 3: Set hostname (optional)
sudo hostnamectl set-hostname parrot-sec
sudo nano /etc/hosts          # Add: 127.0.1.1  parrot-sec

# Step 4: Set timezone
timedatectl list-timezones | grep Asia     # Find your timezone
sudo timedatectl set-timezone Asia/Dhaka  # Set Bangladesh timezone
timedatectl status                         # Verify

# Step 5: Enable essential services
sudo systemctl enable --now ssh            # Enable SSH server
sudo systemctl enable --now NetworkManager # Enable networking
```

### 🔐 Create a Strong User Setup

```bash
# Change root password (highly recommended)
sudo passwd root

# Change your user password
passwd

# Add user to important groups
sudo usermod -aG sudo,adm,netdev,bluetooth,audio,video $USER

# Verify groups
groups $USER
id $USER
```

### 🛠️ Install Essential Tools Not Pre-installed

```bash
sudo apt install -y \
curl wget git vim nano htop \
net-tools iputils-ping traceroute \
build-essential python3-pip python3-venv \
zip unzip p7zip-full \
tmux screen \
tree locate \
exiftool \
steghide \
wfuzz ffuf \
seclists \
gobuster \
whatweb \
dnsenum dnsrecon \
enum4linux \
nbtscan \
onesixtyone \
redis-tools \
rlwrap \
jq
```

---

## 2. Package Management & Updates

### 📦 APT Package Manager Complete Reference

```bash
# === UPDATING ===
sudo apt update                          # Refresh package lists
sudo apt upgrade                         # Upgrade installed packages
sudo apt full-upgrade                    # Upgrade + remove obsolete packages
sudo parrot-upgrade                      # Parrot's safe upgrade (RECOMMENDED)
sudo apt dist-upgrade                    # Distribution upgrade

# === INSTALLING ===
sudo apt install package-name            # Install package
sudo apt install package1 package2       # Install multiple packages
sudo apt install -y package-name         # Install without prompts
sudo apt install --reinstall package     # Reinstall broken package
sudo apt install -f                      # Fix broken dependencies

# === REMOVING ===
sudo apt remove package-name             # Remove (keep config files)
sudo apt purge package-name              # Remove + delete config files
sudo apt autoremove                      # Remove unused dependencies
sudo apt autoclean                       # Clear downloaded package cache
sudo apt clean                           # Clear entire package cache

# === SEARCHING ===
apt search keyword                       # Search packages
apt show package-name                    # Show package details
apt list --installed                     # List all installed packages
apt list --installed | grep tool         # Find specific installed tool
dpkg -l | grep package                   # Alternative: list via dpkg
dpkg -L package-name                     # List files installed by package
which tool-name                          # Find where a tool is installed

# === PACKAGE SOURCES ===
cat /etc/apt/sources.list                # View main sources
ls /etc/apt/sources.list.d/             # View additional sources
sudo nano /etc/apt/sources.list          # Edit sources

# Parrot OS official sources.list:
# deb https://deb.parrot.sh/parrot/ parrot main contrib non-free non-free-firmware
# deb https://deb.parrot.sh/parrot/ parrot-backports main contrib non-free

# === DPKG DIRECT PACKAGE MANAGEMENT ===
sudo dpkg -i package.deb                 # Install .deb file directly
sudo dpkg -r package-name                # Remove package
sudo dpkg --configure -a                 # Configure all unconfigured packages
sudo dpkg --get-selections               # List all packages with status
sudo dpkg-reconfigure package-name       # Reconfigure a package

# === SNAP PACKAGES ===
sudo snap install package-name          # Install snap package
snap list                               # List installed snaps
snap refresh                            # Update all snaps

# === PIP (Python packages) ===
pip3 install package-name               # Install Python package
pip3 install -r requirements.txt        # Install from requirements file
pip3 install --upgrade package          # Upgrade Python package
pip3 list                               # List installed Python packages
```

### 🔑 Adding External Repositories

```bash
# Add a PPA or external repo key
curl -fsSL https://repo.example.com/key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/example.gpg
echo "deb [signed-by=/usr/share/keyrings/example.gpg] https://repo.example.com stable main" | sudo tee /etc/apt/sources.list.d/example.list
sudo apt update

# Example: Add GitHub CLI repo
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list
sudo apt update && sudo apt install gh
```

---

## 3. Network Configuration

### 🌐 Network Interface Configuration

```bash
# === VIEW NETWORK INFO ===
ip a                                     # Show all interfaces and IPs
ip r                                     # Show routing table
ip link show                             # Show link status
ifconfig                                 # Classic (requires net-tools)
nmcli device status                      # NetworkManager device status
nmcli connection show                    # Show all connections

# === STATIC IP (via NetworkManager CLI) ===
nmcli con mod "Wired connection 1" \
ipv4.method manual \
ipv4.addresses "192.168.1.100/24" \
ipv4.gateway "192.168.1.1" \
ipv4.dns "8.8.8.8,8.8.4.4"
nmcli con up "Wired connection 1"        # Apply settings

# === DYNAMIC IP (DHCP) ===
nmcli con mod "Wired connection 1" ipv4.method auto
nmcli con up "Wired connection 1"

# === MANUAL INTERFACE COMMANDS ===
sudo ip link set eth0 up                 # Bring interface up
sudo ip link set eth0 down               # Bring interface down
sudo ip addr add 192.168.1.100/24 dev eth0  # Add IP to interface
sudo ip addr del 192.168.1.100/24 dev eth0  # Remove IP
sudo ip route add default via 192.168.1.1   # Add default gateway
sudo dhclient eth0                       # Request DHCP lease

# === DNS CONFIGURATION ===
cat /etc/resolv.conf                     # View current DNS
sudo nano /etc/resolv.conf               # Edit DNS (temporary — may reset)

# Permanent DNS via NetworkManager:
nmcli con mod "connection-name" ipv4.dns "1.1.1.1 8.8.8.8"
nmcli con up "connection-name"

# Or edit: /etc/NetworkManager/system-connections/connection-name.nmconnection

# DNS over HTTPS (systemd-resolved):
sudo nano /etc/systemd/resolved.conf
# Add: DNS=1.1.1.1 9.9.9.9
# Add: DNSOverTLS=yes
sudo systemctl restart systemd-resolved
```

### 🔍 Network Diagnostics

```bash
ping -c 4 8.8.8.8                        # Test internet connectivity
ping -c 4 google.com                     # Test DNS resolution
traceroute google.com                    # Trace route to destination
mtr google.com                           # Interactive traceroute
nslookup google.com                      # DNS lookup
dig google.com                           # Detailed DNS lookup
dig google.com +short                    # Quick DNS lookup
host google.com                          # Simple DNS lookup
curl -I https://google.com               # Test HTTP connectivity
wget -q --spider http://google.com       # Test URL availability
ss -tulnp                                # Show listening ports
netstat -tulnp                           # Alternative (net-tools)
lsof -i :80                              # Show what's using port 80
```

---

## 4. VPN Configuration

### 🔒 OpenVPN Configuration

```bash
# Install OpenVPN
sudo apt install openvpn -y

# Connect to VPN (HackTheBox / TryHackMe / custom)
sudo openvpn --config yourfile.ovpn

# Connect in background
sudo openvpn --config yourfile.ovpn --daemon

# Connect with credentials file
sudo openvpn --config yourfile.ovpn --auth-user-pass credentials.txt

# Check VPN interface
ip a show tun0                           # VPN interface is usually tun0

# Disconnect
sudo pkill openvpn

# Auto-start on boot:
sudo cp yourfile.ovpn /etc/openvpn/client/
sudo systemctl enable openvpn-client@yourfile
sudo systemctl start openvpn-client@yourfile
```

### 🔒 WireGuard Configuration

```bash
sudo apt install wireguard -y

# Create config file
sudo nano /etc/wireguard/wg0.conf
# Paste your WireGuard config here

# Start WireGuard
sudo wg-quick up wg0

# Stop WireGuard
sudo wg-quick down wg0

# Check status
sudo wg show

# Auto-start on boot
sudo systemctl enable wg-quick@wg0
```

### 🔒 ProtonVPN / NordVPN (GUI)

```bash
# ProtonVPN CLI
sudo apt install protonvpn -y
protonvpn login
protonvpn connect --fastest               # Connect to fastest server
protonvpn connect --cc US                 # Connect to US server
protonvpn disconnect
protonvpn status

# NordVPN
curl -sSf https://downloads.nordcdn.com/apps/linux/install.sh | sh
nordvpn login
nordvpn connect
nordvpn set killswitch on                # Enable kill switch
nordvpn set dns 103.86.96.100            # Set NordVPN DNS
```

---

## 5. Display & GPU Drivers

### 🖥️ NVIDIA Driver Installation (Official Method)

```bash
# Step 1: Identify your GPU
lspci | grep -i nvidia
lspci | grep -i vga

# Step 2: Check what driver is currently active
lsmod | grep nouveau                     # Check if nouveau is loaded
nvidia-smi                               # Check if nvidia driver works

# Step 3: Install NVIDIA driver from Parrot repo
sudo apt update
sudo apt install nvidia-driver -t parrot-backports

# OR for specific driver version:
sudo apt install nvidia-driver-535
sudo apt install nvidia-driver-550

# Step 4: Reboot
sudo reboot

# Step 5: Verify installation
nvidia-smi                               # Should show GPU info
nvidia-settings                          # Open NVIDIA settings GUI

# === MANUAL INSTALLATION (for problems) ===
# Step 1: Drop to text mode
sudo systemctl set-default multi-user.target
sudo reboot

# Step 2: Blacklist nouveau
echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
echo "options nouveau modeset=0" | sudo tee -a /etc/modprobe.d/blacklist-nouveau.conf
sudo update-initramfs -u

# Step 3: Reboot again
sudo reboot

# Step 4: Install driver
sudo bash NVIDIA-Linux-x86_64-*.run

# Step 5: Return to GUI mode
sudo systemctl set-default graphical.target
sudo reboot
```

### 🖥️ AMD/Intel Driver Configuration

```bash
# AMD GPU (usually works out of the box with open-source driver)
sudo apt install firmware-amd-graphics    # AMD firmware
sudo apt install xserver-xorg-video-amdgpu  # AMD Xorg driver

# Intel integrated graphics
sudo apt install xserver-xorg-video-intel  # Intel Xorg driver
sudo apt install intel-media-va-driver     # Intel media acceleration

# Check current display driver
sudo lshw -c display
glxinfo | grep "OpenGL renderer"
inxi -G
```

### 🖥️ Multi-Monitor Setup

```bash
# List connected displays
xrandr

# Set resolution
xrandr --output HDMI-1 --mode 1920x1080

# Extend desktop (side by side)
xrandr --output HDMI-1 --auto --right-of eDP-1

# Mirror displays
xrandr --output HDMI-1 --same-as eDP-1

# KDE GUI: System Settings > Display and Monitor > Display Configuration
```

---

## 6. Wi-Fi Adapter Configuration

### 📡 Identify Your Wi-Fi Adapter

```bash
# Find your Wi-Fi card
lspci | grep -i wireless
lspci | grep -i network
lsusb                                    # For USB Wi-Fi adapters
inxi -N                                  # Network info
iwconfig                                 # Wireless interface info
iw dev                                   # Modern wireless info
ip link show                             # All interfaces
```

### 📡 Install Missing Wi-Fi Drivers

```bash
# Check if driver is missing
lshw -C network                          # Look for "UNCLAIMED" = no driver

# Install firmware packages (fixes most issues)
sudo apt install -y \
firmware-iwlwifi \          # Intel Wi-Fi
firmware-atheros \          # Atheros Wi-Fi
firmware-realtek \          # Realtek Wi-Fi
firmware-ralink \           # Ralink Wi-Fi
firmware-brcm80211 \        # Broadcom Wi-Fi
firmware-libertas \         # Marvell
firmware-misc-nonfree \     # Various
linux-headers-$(uname -r)   # Kernel headers (needed for building drivers)

sudo update-initramfs -u
sudo reboot

# Broadcom specific (common laptop Wi-Fi)
sudo apt install broadcom-sta-dkms
sudo modprobe wl                         # Load Broadcom module

# Realtek RTL8812AU (popular USB pentest adapter)
sudo apt install realtek-rtl88xxau-dkms
sudo modprobe 88XXau
```

### 📡 Monitor Mode Setup

```bash
# Enable monitor mode (for Wi-Fi pentesting)
sudo airmon-ng check kill                # Kill conflicting processes
sudo airmon-ng start wlan0               # Start monitor mode → creates wlan0mon
iwconfig wlan0mon                        # Verify monitor mode

# Manual method:
sudo ip link set wlan0 down
sudo iw wlan0 set monitor control
sudo ip link set wlan0 up

# Disable monitor mode (return to normal)
sudo airmon-ng stop wlan0mon
sudo service NetworkManager restart

# Check interface capabilities
iw phy phy0 info | grep -A 5 "Supported interface modes"
```

---

## 7. Sound & Audio Configuration

### 🔊 PipeWire / PulseAudio Configuration (Parrot 7.0)

```bash
# Parrot 7.0 uses PipeWire by default
# Check what audio system is running:
pactl info | grep "Server Name"
systemctl --user status pipewire
systemctl --user status pulseaudio

# === VOLUME CONTROL ===
pactl set-sink-volume @DEFAULT_SINK@ 100%    # Set volume to 100%
pactl set-sink-volume @DEFAULT_SINK@ +10%    # Increase volume
pactl set-sink-volume @DEFAULT_SINK@ -10%    # Decrease volume
pactl set-sink-mute @DEFAULT_SINK@ toggle    # Mute/unmute
pactl list sinks short                       # List audio output devices
pactl list sources short                     # List audio input devices

# === SELECT DEFAULT AUDIO DEVICE ===
pactl set-default-sink SINK_NAME            # Set default output
pactl set-default-source SOURCE_NAME        # Set default input (microphone)

# === ALSA (lower level) ===
alsamixer                                    # Interactive volume control
amixer set Master 80%                        # Set master volume
alsactl store                                # Save ALSA settings

# === FIX AUDIO (if not working) ===
# Restart PipeWire:
systemctl --user restart pipewire
systemctl --user restart pipewire-pulse

# Kill and restart PulseAudio:
pulseaudio --kill && pulseaudio --start

# Reload ALSA:
sudo alsactl restore

# Fix for laptop microphone/speaker issues:
sudo nano /etc/modprobe.d/alsa-base.conf
# Add: options snd-hda-intel index=0 model=auto
sudo reboot
```

---

## 8. Firewall Configuration

### 🔥 UFW (Uncomplicated Firewall)

```bash
# Install UFW
sudo apt install ufw -y

# Enable / Disable
sudo ufw enable
sudo ufw disable

# Default policies
sudo ufw default deny incoming           # Block all incoming (recommended)
sudo ufw default allow outgoing          # Allow all outgoing

# Allow specific ports
sudo ufw allow 22/tcp                    # SSH
sudo ufw allow 80/tcp                    # HTTP
sudo ufw allow 443/tcp                   # HTTPS
sudo ufw allow 4444/tcp                  # Custom (e.g. Metasploit handler)

# Allow from specific IP
sudo ufw allow from 192.168.1.0/24      # Allow entire subnet
sudo ufw allow from 10.10.10.1           # Allow specific IP

# Deny specific ports
sudo ufw deny 23/tcp                     # Block Telnet
sudo ufw deny from 192.168.1.50         # Block specific IP

# Delete rules
sudo ufw delete allow 22/tcp
sudo ufw delete 3                        # Delete rule by number

# View rules
sudo ufw status verbose                  # Detailed status
sudo ufw status numbered                 # Rules with numbers

# Reset all rules
sudo ufw reset

# Logging
sudo ufw logging on
sudo ufw logging high
sudo cat /var/log/ufw.log
```

### 🔥 iptables (Advanced)

```bash
# View current rules
sudo iptables -L -n -v --line-numbers     # All rules with details
sudo iptables -L INPUT -n -v              # INPUT chain only
sudo iptables -t nat -L -n -v             # NAT table

# Save and restore rules
sudo iptables-save > /etc/iptables/rules.v4
sudo iptables-restore < /etc/iptables/rules.v4

# Allow established connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Block an IP
sudo iptables -A INPUT -s 192.168.1.50 -j DROP

# Allow port
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Port forwarding (for pivoting)
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward  # Enable forwarding

# Flush all rules (CAREFUL — removes all firewall rules)
sudo iptables -F
sudo iptables -X
```

---

## 9. User & Permissions Management

### 👤 User Management

```bash
# Create new user
sudo useradd -m -s /bin/bash username
sudo passwd username

# Create user with specific groups
sudo useradd -m -s /bin/bash -G sudo,netdev username

# Delete user
sudo userdel -r username                 # -r removes home directory

# Modify user
sudo usermod -aG groupname username      # Add to group
sudo usermod -s /bin/zsh username        # Change shell
sudo usermod -l newname oldname          # Rename user

# List users
cat /etc/passwd | grep /home            # Real users
getent passwd                            # All users
id username                              # User info

# Switch users
su - username                            # Switch with environment
sudo -u username command                 # Run command as user
sudo su                                  # Become root
```

### 🔒 File Permissions

```bash
# Permission format: [type][owner][group][others]
# r=4, w=2, x=1

chmod 755 file          # rwxr-xr-x  (owner:rwx, group:r-x, others:r-x)
chmod 644 file          # rw-r--r--  (owner:rw, group:r, others:r)
chmod 600 file          # rw-------  (owner only, for SSH keys)
chmod 777 file          # rwxrwxrwx  (everyone — DANGEROUS)
chmod +x script.sh      # Add execute permission
chmod -R 755 directory/ # Recursive permission change

# Change ownership
sudo chown user:group file
sudo chown -R user:group directory/
sudo chown root:root /etc/sensitive_file

# Special permissions
chmod +s binary         # Set SUID bit (run as owner)
chmod +t directory/     # Set sticky bit (only owner can delete)
find / -perm -4000 2>/dev/null  # Find all SUID files

# View permissions
ls -la file
stat file
```

### 🔑 Sudo Configuration

```bash
sudo visudo                              # Safely edit /etc/sudoers

# Common sudoers entries:
# username ALL=(ALL:ALL) ALL            → Full sudo access
# username ALL=(ALL) NOPASSWD: ALL      → Sudo without password
# username ALL=(ALL) /usr/bin/nmap      → Only allow specific command

# Add user to sudo group:
sudo usermod -aG sudo username
# Verify:
sudo -l -U username
```

---

## 10. Shell & Terminal Configuration

### 🐚 Bash Configuration

```bash
# Edit bash profile
nano ~/.bashrc                           # User bash settings
nano ~/.bash_profile                     # Login shell settings
nano /etc/bash.bashrc                    # System-wide bash settings

# Apply changes immediately
source ~/.bashrc

# Useful aliases to add to ~/.bashrc:
alias ll='ls -alF'
alias la='ls -A'
alias ..='cd ..'
alias ...='cd ../..'
alias grep='grep --color=auto'
alias ports='ss -tulnp'
alias myip='curl -s ifconfig.me'
alias update='sudo parrot-upgrade'
alias cls='clear'

# Security-focused aliases:
alias nmap='nmap --reason'
alias python='python3'
alias pip='pip3'
alias msfstart='sudo service postgresql start && msfconsole'

# Add to PATH (for custom tools):
export PATH=$PATH:~/tools/bin:/opt/tools
export PATH=$PATH:~/.local/bin

# Load after editing:
source ~/.bashrc
```

### 🐚 ZSH with Oh-My-Zsh (Enhanced Shell)

```bash
# Install ZSH
sudo apt install zsh -y

# Install Oh-My-Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Set ZSH as default shell
chsh -s $(which zsh)

# Install popular plugins
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# Edit ~/.zshrc to enable plugins:
# plugins=(git zsh-autosuggestions zsh-syntax-highlighting sudo web-search)

# Apply:
source ~/.zshrc
```

### 🖥️ tmux Configuration

```bash
# Install tmux
sudo apt install tmux -y

# Create ~/.tmux.conf for better experience:
cat > ~/.tmux.conf << 'EOF'
# Change prefix to Ctrl+a
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# Split panes
bind | split-window -h
bind - split-window -v

# Mouse support
set -g mouse on

# Increase history
set -g history-limit 10000

# Better colors
set -g default-terminal "screen-256color"
EOF

# Key shortcuts:
# Ctrl+a c       — New window
# Ctrl+a n/p     — Next/prev window
# Ctrl+a |       — Split horizontal
# Ctrl+a -       — Split vertical
# Ctrl+a arrows  — Navigate panes
# Ctrl+a d       — Detach session
tmux new -s pentest              # New session named "pentest"
tmux attach -t pentest           # Reattach to session
tmux ls                          # List sessions
```

---

## 11. KDE Plasma Desktop Configuration

### 🖥️ KDE Settings & Customization

```bash
# System Settings (GUI)
systemsettings5                          # Open KDE system settings

# KDE Config locations:
~/.config/                               # Most KDE app configs
~/.local/share/                         # KDE data files
~/.config/kwinrc                        # KWindow Manager settings
~/.config/plasma-org.kde.plasma.desktop-appletsrc  # Desktop widgets
~/.config/kdeglobals                    # Global KDE settings

# Reset KDE to defaults (if broken):
rm -rf ~/.config/plasma* ~/.config/kde*
pkill plasmashell && kstart5 plasmashell

# Restart KDE Plasma shell (without logout):
plasmashell --replace &

# Fix KDE Plasma crash loop:
kwin_x11 --replace &
```

### 🎨 KDE Themes & Appearance

```bash
# Install KDE themes via command line
sudo apt install kde-full plasma-theme-oxygen

# Change theme via GUI:
# System Settings > Appearance > Global Theme

# Dark mode:
# System Settings > Appearance > Colors → Breeze Dark

# Install additional icon packs:
sudo apt install papirus-icon-theme
# Then: System Settings > Appearance > Icons → Papirus-Dark
```

---

## 12. Service Management (systemd)

### ⚙️ systemctl Complete Reference

```bash
# === SERVICE CONTROL ===
sudo systemctl start service-name        # Start a service
sudo systemctl stop service-name         # Stop a service
sudo systemctl restart service-name      # Restart a service
sudo systemctl reload service-name       # Reload config without restart
sudo systemctl status service-name       # Check service status
sudo systemctl enable service-name       # Enable on boot
sudo systemctl disable service-name      # Disable on boot
sudo systemctl enable --now service-name # Enable AND start immediately

# === VIEWING SERVICES ===
systemctl list-units --type=service      # All active services
systemctl list-units --type=service --all # All services (including inactive)
systemctl list-unit-files                # All unit files and their states
systemctl is-active service-name         # Check if running
systemctl is-enabled service-name        # Check if enabled on boot
systemctl is-failed service-name         # Check if failed

# === LOGS ===
journalctl -u service-name               # Logs for specific service
journalctl -u service-name -f            # Follow live logs
journalctl -u service-name --since "1 hour ago"
journalctl -xe                           # Recent errors (most useful for debugging)
journalctl -b                            # Logs since last boot
journalctl -b -1                         # Logs from previous boot
journalctl --disk-usage                  # Check log disk usage
sudo journalctl --vacuum-time=7d         # Keep only last 7 days of logs

# === PENTEST-SPECIFIC SERVICES ===
sudo systemctl start postgresql          # Metasploit DB
sudo systemctl start neo4j               # BloodHound DB
sudo systemctl start apache2             # Web server
sudo systemctl start ssh                 # SSH server
sudo systemctl start tor                 # Tor service
sudo systemctl start nessusd             # Nessus scanner

# === SYSTEM TARGETS (runlevels) ===
sudo systemctl isolate multi-user.target  # Switch to text mode (runlevel 3)
sudo systemctl isolate graphical.target   # Switch to GUI (runlevel 5)
sudo systemctl get-default               # Check default target
sudo systemctl set-default graphical.target  # Set default to GUI
```

---

## 13. Metasploit & PostgreSQL Setup

### 💀 Complete Metasploit Configuration

```bash
# === INITIAL SETUP ===
sudo apt install metasploit-framework -y

# Start PostgreSQL (required for Metasploit DB)
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Initialize Metasploit database
sudo msfdb init

# OR if init fails:
sudo msfdb delete
sudo msfdb init

# Verify database connection
msfconsole -q -x "db_status; exit"
# Should show: [*] Connected to msf. Connection type: postgresql

# === LAUNCH OPTIONS ===
msfconsole                               # Normal launch
msfconsole -q                            # Quiet mode (no banner)
msfconsole -x "use exploit/xxx; set RHOSTS 1.2.3.4; run"  # Run commands

# === DATABASE COMMANDS IN MSFCONSOLE ===
db_status                                # Check DB connection
workspace                                # List workspaces
workspace -a target_name                 # Create new workspace
workspace target_name                    # Switch workspace
db_nmap -sV -sC 10.10.10.1             # Nmap with results saved to DB
hosts                                    # List discovered hosts
services                                 # List discovered services
vulns                                    # List found vulnerabilities
creds                                    # List captured credentials
loot                                     # List captured loot
notes                                    # List notes

# === UPDATE METASPLOIT ===
sudo msfupdate
# OR:
sudo apt update && sudo apt upgrade metasploit-framework

# === METASPLOIT RPC (for automation) ===
sudo msfrpcd -P yourpassword -S -a 127.0.0.1
# Connect via msfrpc client or Armitage

# === RESET METASPLOIT DB ===
sudo msfdb delete
sudo msfdb init
```

---

## 14. Tor & AnonSurf Configuration

### 🧅 Tor Configuration

```bash
# === TOR SERVICE ===
sudo apt install tor -y
sudo systemctl start tor
sudo systemctl enable tor
sudo systemctl status tor

# Tor configuration file:
sudo nano /etc/tor/torrc

# Key settings:
# SOCKSPort 9050                  → SOCKS5 proxy port
# ControlPort 9051                → Control port
# HashedControlPassword HASH      → Auth password
# ExitNodes {US}                  → Prefer US exit nodes
# StrictNodes 1                   → Only use specified exit nodes
# ExcludeExitNodes {CN},{RU}      → Avoid these exit countries

# Generate HashedControlPassword:
tor --hash-password "yourpassword"

# Test Tor connectivity:
curl --socks5-hostname 127.0.0.1:9050 https://check.torproject.org/api/ip

# === ANONSURF ===
sudo anonsurf start                      # Start anonymous mode
sudo anonsurf stop                       # Stop anonymous mode
sudo anonsurf restart                    # Restart (new Tor identity)
sudo anonsurf changeid                   # Change Tor exit node
sudo anonsurf myip                       # Verify your Tor IP
sudo anonsurf status                     # Check if running
sudo anonsurf dns                        # Switch to anonymous DNS

# === PROXYCHAINS4 CONFIGURATION ===
sudo nano /etc/proxychains4.conf

# Configuration options:
# strict_chain         → Use proxies in exact order (fails if one is down)
# dynamic_chain        → Skip dead proxies
# random_chain         → Random proxy order
# proxy_dns            → Route DNS through proxy

# Add at the end of the file:
# socks5 127.0.0.1 9050  ← Tor
# socks4 proxy.example.com 1080  ← Additional proxy

# Use proxychains with any tool:
proxychains4 nmap -sT target.com        # Nmap via Tor
proxychains4 sqlmap -u http://target.com
proxychains4 ssh user@target.com
proxychains4 curl https://target.com
```

---

## 15. SSH Server Configuration

### 🔐 SSH Setup & Hardening

```bash
# Install and start SSH
sudo apt install openssh-server -y
sudo systemctl enable --now ssh

# Main config file
sudo nano /etc/ssh/sshd_config

# === KEY SECURITY SETTINGS ===
# Port 2222                        → Change default port (avoid scans)
# PermitRootLogin no               → Disable root SSH (use sudo)
# PasswordAuthentication no        → Key-only auth (most secure)
# PubkeyAuthentication yes         → Enable key auth
# AuthorizedKeysFile .ssh/authorized_keys
# X11Forwarding no                 → Disable X11 forwarding
# MaxAuthTries 3                   → Limit auth attempts
# LoginGraceTime 30                → Timeout for login
# AllowUsers username1 username2   → Whitelist users
# Protocol 2                       → Use SSHv2 only

# Apply config changes:
sudo systemctl restart ssh
sudo sshd -t                             # Test config syntax

# === SSH KEY MANAGEMENT ===
ssh-keygen -t ed25519 -C "parrot-pentest"  # Generate Ed25519 key (best)
ssh-keygen -t rsa -b 4096 -C "backup"      # Generate RSA 4096 key
cat ~/.ssh/id_ed25519.pub                   # View public key

# Copy key to remote host
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@target.com
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 user@target.com

# Manual copy
cat ~/.ssh/id_ed25519.pub | ssh user@target "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# SSH config file (client shortcuts)
nano ~/.ssh/config
# Example:
# Host htb
#   HostName 10.10.10.100
#   User root
#   IdentityFile ~/.ssh/htb_key
#   Port 22
# Then just: ssh htb

# === SSH TUNNELING ===
ssh -L 8080:localhost:80 user@server     # Local port forward
ssh -R 4444:localhost:4444 user@server  # Remote port forward
ssh -D 1080 user@server                 # Dynamic SOCKS5 proxy
ssh -N -f -L 3306:db_host:3306 user@jump  # Background tunnel
```

---

## 16. Docker Configuration

### 🐳 Docker Setup

```bash
# Install Docker
sudo apt install docker.io -y
sudo systemctl enable --now docker

# Add user to docker group (run without sudo)
sudo usermod -aG docker $USER
newgrp docker                            # Apply group change immediately

# Verify installation
docker --version
docker run hello-world

# === SECURITY TOOLS IN DOCKER ===
# Run Metasploit in Docker:
docker pull metasploitframework/metasploit-framework
docker run -it metasploitframework/metasploit-framework

# Run Kali tools in Docker:
docker pull kalilinux/kali-rolling
docker run -it --network host kalilinux/kali-rolling bash

# Run tools with network access:
docker run --rm -it --network host tool-name

# === DOCKER COMMANDS ===
docker ps                                # Running containers
docker ps -a                             # All containers
docker images                            # Downloaded images
docker pull imagename                    # Download image
docker run -it imagename bash            # Run interactively
docker exec -it containerid bash         # Attach to running container
docker stop containerid                  # Stop container
docker rm containerid                    # Remove container
docker rmi imagename                     # Remove image
docker system prune                      # Clean up everything unused
```

---

## 17. Virtual Machine / VirtualBox Setup

### 💻 VirtualBox Installation & Configuration

```bash
# Install VirtualBox
sudo apt install virtualbox virtualbox-ext-pack -y

# Add user to vboxusers group
sudo usermod -aG vboxusers $USER

# Install VirtualBox Guest Additions (inside a VM)
sudo apt install virtualbox-guest-utils virtualbox-guest-x11 -y
sudo reboot

# === NETWORK MODES FOR PENTESTING ===
# NAT          → VM has internet, host can't see VM's open ports
# Bridged      → VM gets its own IP on network (best for pentest labs)
# Host-Only    → VM and host can communicate, no internet
# Internal     → VMs communicate with each other only

# Recommended lab setup:
# AttackBox (Parrot) ← Host-Only Network → Target VMs (Metasploitable, DVWA)

# Enable nested virtualization (for running VMs inside VMs):
VBoxManage modifyvm "VM Name" --nested-hw-virt on

# Increase VM disk size:
VBoxManage modifyhd /path/to/disk.vdi --resize 51200  # 50GB

# === QEMU/KVM (Better Performance Alternative) ===
sudo apt install qemu-kvm virt-manager libvirt-daemon-system -y
sudo usermod -aG libvirt,kvm $USER
sudo systemctl enable --now libvirtd
virt-manager                             # Open GUI VM manager
```

---

## 18. Dual Boot Configuration

### 💽 GRUB Bootloader Management

```bash
# Update GRUB (after installing new OS or kernels)
sudo update-grub

# Edit GRUB settings
sudo nano /etc/default/grub

# Key GRUB settings:
# GRUB_DEFAULT=0                        → Boot first entry by default
# GRUB_TIMEOUT=10                       → Wait 10 seconds at boot menu
# GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"

# Apply changes:
sudo update-grub
sudo grub-install /dev/sda              # Reinstall GRUB to disk

# Fix dual boot clock issue (Linux/Windows time conflict):
sudo timedatectl set-local-rtc 1 --adjust-system-clock
# This tells Linux to use local time (like Windows) instead of UTC

# Add Windows to GRUB menu (if not auto-detected):
sudo apt install os-prober
sudo os-prober                          # Detect other OS
sudo update-grub

# Repair GRUB from live USB:
# 1. Boot from Parrot live USB
# 2. Find your Parrot partition:
sudo fdisk -l
# 3. Mount it:
sudo mount /dev/sdaX /mnt
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
# 4. Chroot:
sudo chroot /mnt
# 5. Reinstall GRUB:
grub-install /dev/sda
update-grub
exit
```

---

## 19. Disk Encryption & LUKS

### 🔒 LUKS Full Disk Encryption

```bash
# Check if disk is encrypted
lsblk -f                                 # Check filesystem types
cryptsetup status /dev/mapper/device     # Check LUKS status

# Create encrypted partition
sudo cryptsetup luksFormat /dev/sdX      # Format with LUKS (DESTROYS DATA)
sudo cryptsetup open /dev/sdX myencrypt # Open encrypted partition
sudo mkfs.ext4 /dev/mapper/myencrypt    # Format inside
sudo mount /dev/mapper/myencrypt /mnt   # Mount

# Close encrypted partition
sudo umount /mnt
sudo cryptsetup close myencrypt

# Add backup LUKS key (always have 2 key slots!)
sudo cryptsetup luksAddKey /dev/sdX

# Create encrypted file container (like VeraCrypt)
dd if=/dev/urandom of=secure.img bs=1M count=500   # 500MB container
sudo cryptsetup luksFormat secure.img
sudo cryptsetup open secure.img secure_container
sudo mkfs.ext4 /dev/mapper/secure_container
sudo mount /dev/mapper/secure_container /mnt/secure

# Encrypt specific files (GPG)
gpg --symmetric --cipher-algo AES256 sensitive.txt
gpg --decrypt sensitive.txt.gpg
```

---

## 20. Swap & Memory Configuration

```bash
# Check current memory and swap
free -h
swapon --show
cat /proc/swaps

# === CREATE SWAP FILE ===
sudo fallocate -l 4G /swapfile           # Create 4GB swap file
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent (add to /etc/fstab):
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Change swappiness (how aggressively Linux uses swap)
# Lower = less swap use (0-100, default 60)
sudo sysctl vm.swappiness=10            # Temporary
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf  # Permanent

# Memory info
vmstat -s                               # Virtual memory stats
cat /proc/meminfo                       # Detailed memory info
```

---

## 21. Locale, Timezone & Keyboard

```bash
# === TIMEZONE ===
timedatectl list-timezones             # List all timezones
sudo timedatectl set-timezone Asia/Dhaka    # Set Bangladesh timezone
sudo timedatectl set-timezone Asia/Kolkata  # India timezone
sudo timedatectl set-timezone Europe/London # UK timezone
timedatectl status                     # Verify

# === LOCALE (Language Settings) ===
locale                                  # Current locale settings
locale -a                               # Available locales
sudo locale-gen en_US.UTF-8            # Generate locale
sudo dpkg-reconfigure locales          # Interactive locale setup
echo 'LANG=en_US.UTF-8' | sudo tee /etc/default/locale

# === KEYBOARD LAYOUT ===
setxkbmap us                            # Set US keyboard (temporary)
setxkbmap bd                            # Bangladesh keyboard
sudo dpkg-reconfigure keyboard-configuration  # Interactive setup

# KDE keyboard settings:
# System Settings > Input Devices > Keyboard

# === FONT CONFIGURATION ===
# Install fonts for better terminal rendering:
sudo apt install fonts-firacode fonts-noto fonts-noto-cjk
fc-cache -fv                            # Refresh font cache
```

---

## 22. Custom Tool Installation

### 🛠️ Installing Tools From GitHub

```bash
# General method for installing from GitHub:
git clone https://github.com/author/toolname
cd toolname
pip3 install -r requirements.txt        # Python tools
sudo python3 setup.py install           # OR
pip3 install .                          # OR

# Create a tools directory structure:
mkdir -p ~/tools/bin ~/tools/src
export PATH=$PATH:~/tools/bin            # Add to ~/.bashrc

# === POPULAR TOOL INSTALLATIONS ===

# impacket (AD attack toolkit)
git clone https://github.com/fortra/impacket
cd impacket && pip3 install .

# LinPEAS/WinPEAS
git clone https://github.com/carlospolop/PEASS-ng
# Or download directly:
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh -o ~/tools/linpeas.sh
chmod +x ~/tools/linpeas.sh

# PayloadsAllTheThings
git clone https://github.com/swisskyrepo/PayloadsAllTheThings ~/tools/PayloadsAllTheThings

# Chisel (tunneling)
wget https://github.com/jpillora/chisel/releases/latest/download/chisel_*_linux_amd64.gz
gunzip chisel_*.gz && mv chisel_* ~/tools/bin/chisel && chmod +x ~/tools/bin/chisel

# Ligolo-ng (pivoting)
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/proxy_linux_amd64
mv proxy_linux_amd64 ~/tools/bin/ligolo-proxy && chmod +x ~/tools/bin/ligolo-proxy

# Kerbrute (AD enumeration)
wget https://github.com/ropnop/kerbrute/releases/latest/download/kerbrute_linux_amd64
chmod +x kerbrute_linux_amd64 && sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute

# pwncat-cs (advanced reverse shell handler)
pip3 install pwncat-cs

# CrackMapExec / NetExec
sudo apt install netexec
# OR from pip:
pip3 install git+https://github.com/Pennyw0rth/NetExec

# JADX (Android reverse engineering)
wget https://github.com/skylot/jadx/releases/latest/download/jadx-1.5.0.zip
unzip jadx-*.zip -d ~/tools/jadx
ln -s ~/tools/jadx/bin/jadx ~/tools/bin/jadx
```

---

## 23. Wordlist & SecLists Setup

### 📚 Setting Up Wordlists

```bash
# Install SecLists (massive collection)
sudo apt install seclists -y
# Location: /usr/share/seclists/

# Install rockyou.txt (most common password list)
sudo apt install wordlists -y
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
# Location: /usr/share/wordlists/rockyou.txt

# List all available wordlists
ls /usr/share/wordlists/
ls /usr/share/seclists/

# === ESSENTIAL WORDLISTS ===
# Passwords:
/usr/share/wordlists/rockyou.txt                          # 14M passwords
/usr/share/seclists/Passwords/darkweb2017-top10000.txt    # Top 10k

# Web directories:
/usr/share/seclists/Discovery/Web-Content/common.txt      # Common dirs
/usr/share/seclists/Discovery/Web-Content/big.txt         # Large list
/usr/share/dirbuster/directory-list-2.3-medium.txt        # DirBuster list

# Subdomains:
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/seclists/Discovery/DNS/namelist.txt

# Usernames:
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
/usr/share/seclists/Usernames/Names/names.txt

# Download additional wordlists
wget https://github.com/danielmiessler/SecLists/raw/master/Passwords/Common-Credentials/10-million-password-list-top-1000000.txt

# Create custom wordlist from a website:
cewl http://target.com -w custom_wordlist.txt -d 3

# Combine and deduplicate wordlists:
cat list1.txt list2.txt | sort -u > combined.txt

# Generate password mutations:
hashcat --stdout passwords.txt -r /usr/share/hashcat/rules/best64.rule > mutated.txt
```

---

## 24. Hashcat GPU Setup

### ⚡ Configure Hashcat for Maximum Performance

```bash
# Install Hashcat
sudo apt install hashcat -y

# Check GPU availability
hashcat -I                               # List all detected devices
clinfo                                   # OpenCL info
nvidia-smi                               # NVIDIA GPU info

# Install OpenCL for GPU acceleration
sudo apt install opencl-headers ocl-icd-opencl-dev -y

# For NVIDIA:
sudo apt install nvidia-opencl-dev -y

# For AMD:
sudo apt install mesa-opencl-icd -y

# For Intel:
sudo apt install intel-opencl-icd -y

# Test GPU cracking
hashcat -b                               # Benchmark all hash types
hashcat -b -m 0                          # Benchmark MD5 only
hashcat -b -d 1                          # Benchmark on GPU device 1

# Run hashcat with GPU:
hashcat -m 1000 -a 0 -d 1 hashes.txt rockyou.txt

# Optimize for speed:
hashcat -m 0 hashes.txt wordlist.txt -O  # Optimized kernels
hashcat -m 0 hashes.txt wordlist.txt -w 3  # Workload level 3 (high)

# If GPU not detected, try:
sudo usermod -aG video $USER             # Add user to video group
sudo reboot
```

---

## 25. AppArmor Configuration

```bash
# Check AppArmor status
sudo aa-status
sudo systemctl status apparmor

# List profiles
sudo aa-status | grep "profiles are in"
cat /etc/apparmor.d/                    # Profile directory

# Set profile modes:
sudo aa-complain /etc/apparmor.d/profile-name   # Complain mode (log only)
sudo aa-enforce /etc/apparmor.d/profile-name    # Enforce mode
sudo aa-disable /etc/apparmor.d/profile-name    # Disable profile

# Reload AppArmor profiles:
sudo systemctl reload apparmor
sudo apparmor_parser -r /etc/apparmor.d/profile

# View AppArmor logs:
sudo journalctl -u apparmor
sudo dmesg | grep apparmor
grep apparmor /var/log/syslog
```

---

# 🔧 TROUBLESHOOTING SECTION

---

## 26. Package & APT Problems

### ❌ Problem: `apt update` fails / repository errors

```bash
# Fix 1: Check internet connectivity
ping -c 3 8.8.8.8
ping -c 3 deb.parrot.sh

# Fix 2: Update package list with error details
sudo apt update 2>&1 | grep -i "err\|fail"

# Fix 3: Fix broken sources
sudo nano /etc/apt/sources.list
# Ensure it has:
# deb https://deb.parrot.sh/parrot/ parrot main contrib non-free non-free-firmware

# Fix 4: Clear apt cache and retry
sudo apt clean
sudo apt update

# Fix 5: Fix GPG key errors
sudo apt-key update
sudo gpg --keyserver keyserver.ubuntu.com --recv-keys KEY_ID
```

### ❌ Problem: `dpkg: error processing package`

```bash
# Fix 1: Configure unconfigured packages
sudo dpkg --configure -a

# Fix 2: Fix broken dependencies
sudo apt install -f
sudo apt --fix-broken install

# Fix 3: Remove corrupted package lock
sudo rm /var/lib/dpkg/lock
sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/cache/apt/archives/lock
sudo dpkg --configure -a
sudo apt update

# Fix 4: Corrupted package database
sudo mv /var/lib/dpkg/info/package.list /tmp/
sudo apt install --reinstall package

# Fix 5: Remove partially installed package
sudo dpkg --remove --force-remove-reinstreq package-name
sudo apt install -f
```

### ❌ Problem: `unmet dependencies` or `broken packages`

```bash
# Fix 1: Standard fix sequence
sudo apt update
sudo apt --fix-broken install
sudo dpkg --configure -a
sudo apt autoremove
sudo apt upgrade

# Fix 2: Force downgrade conflicting package
sudo apt install package=VERSION

# Fix 3: Use aptitude (smarter dependency resolver)
sudo apt install aptitude
sudo aptitude install problematic-package

# Fix 4: Check which packages are broken
dpkg -l | grep -E "^.H|^.U"             # H=hold, U=unknown

# Fix 5: Complete package system reset (LAST RESORT)
sudo apt clean
sudo apt update
sudo apt dist-upgrade
```

### ❌ Problem: Package manager locked

```bash
# Error: "Could not get lock /var/lib/dpkg/lock"

# Fix 1: Check if another apt process is running
ps aux | grep apt
ps aux | grep dpkg

# Fix 2: Kill the process
sudo kill PID

# Fix 3: Remove lock files (only if no apt is running!)
sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/lib/dpkg/lock
sudo rm /var/cache/apt/archives/lock
sudo dpkg --configure -a
```

---

## 27. Network & Wi-Fi Problems

### ❌ Problem: No Wi-Fi / Wi-Fi not detected

```bash
# Step 1: Identify the card
lspci | grep -i wireless
lsusb
inxi -N

# Step 2: Check driver status
lshw -C network
# Look for "UNCLAIMED" = no driver installed

# Step 3: Load the driver manually
sudo modprobe iwlwifi                    # Intel
sudo modprobe ath9k                      # Atheros
sudo modprobe rtl8xxxu                   # Realtek

# Step 4: Install firmware
sudo apt install firmware-iwlwifi firmware-atheros firmware-realtek -y
sudo update-initramfs -u
sudo reboot

# Step 5: Check if blocked by rfkill
rfkill list all
sudo rfkill unblock wifi
sudo rfkill unblock all
```

### ❌ Problem: Wi-Fi drops frequently

```bash
# Fix 1: Disable power management on Wi-Fi
sudo iwconfig wlan0 power off
# Make permanent:
echo 'ACTION=="add", SUBSYSTEM=="net", KERNEL=="wlan*", RUN+="/sbin/iwconfig %k power off"' | sudo tee /etc/udev/rules.d/70-wifi-powersave.rules

# Fix 2: Disable NetworkManager Wi-Fi power saving
sudo nano /etc/NetworkManager/conf.d/default-wifi-powersave-on.conf
# Change: wifi.powersave = 2  (2=disable, 3=enable)

# Fix 3: Set Wi-Fi to infrastructure mode
sudo iwconfig wlan0 mode Managed
sudo service NetworkManager restart

# Fix 4: Update driver
sudo apt install linux-headers-$(uname -r)
sudo apt install --reinstall firmware-iwlwifi
sudo modprobe -r iwlwifi && sudo modprobe iwlwifi
```

### ❌ Problem: No internet after VPN connection

```bash
# Fix 1: Check routing table
ip route

# Fix 2: Add default route through VPN
sudo ip route add default via VPN_GATEWAY dev tun0

# Fix 3: Check DNS
cat /etc/resolv.conf
# Add DNS:
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

# Fix 4: Flush DNS cache
sudo systemd-resolve --flush-caches
sudo resolvectl flush-caches
```

### ❌ Problem: DNS not resolving

```bash
# Fix 1: Test DNS directly
nslookup google.com 8.8.8.8             # Test with Google DNS
dig @1.1.1.1 google.com                 # Test with Cloudflare DNS

# Fix 2: Fix resolv.conf
sudo nano /etc/resolv.conf
# Add:
# nameserver 8.8.8.8
# nameserver 1.1.1.1

# Fix 3: systemd-resolved fix (common in Parrot 6.2+)
sudo systemctl enable --now systemd-resolved
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf

# Fix 4: Restart networking
sudo systemctl restart NetworkManager
sudo systemctl restart systemd-resolved
```

---

## 28. Boot & GRUB Problems

### ❌ Problem: System won't boot / drops to GRUB rescue

```bash
# At grub rescue prompt:
ls                                       # List partitions: (hd0,gpt1) etc
ls (hd0,gpt2)/                          # Check which partition has /boot
# When found:
set root=(hd0,gpt2)
set prefix=(hd0,gpt2)/boot/grub
insmod normal
normal                                   # Boot normally

# After booting, reinstall GRUB:
sudo grub-install /dev/sda
sudo update-grub
```

### ❌ Problem: Black screen after GRUB (boots to black screen)

```bash
# Fix 1: Boot with nomodeset (at GRUB menu)
# Press 'e' at GRUB entry
# Find line starting with "linux"
# Add "nomodeset" before "quiet splash"
# Press Ctrl+X or F10 to boot

# Make nomodeset permanent:
sudo nano /etc/default/grub
# Change: GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nomodeset"
sudo update-grub

# Fix 2: NVIDIA black screen
# Same as above — add nomodeset or install NVIDIA drivers first

# Fix 3: Boot to recovery mode
# At GRUB: Advanced options > Recovery mode
# Select: "Enable networking" then "Drop to root shell"
sudo apt update && sudo apt upgrade
```

### ❌ Problem: Cannot boot after NVIDIA driver install

```bash
# Boot into recovery/text mode
# Then fix NVIDIA:
sudo apt purge nvidia-*
sudo apt autoremove
sudo update-initramfs -u
sudo reboot

# OR blacklist nouveau:
sudo nano /etc/modprobe.d/blacklist-nouveau.conf
# Add:
# blacklist nouveau
# options nouveau modeset=0
sudo update-initramfs -u
sudo reboot
```

---

## 29. Display & Screen Problems

### ❌ Problem: Wrong screen resolution

```bash
# List available resolutions
xrandr

# Set resolution manually
xrandr --output HDMI-1 --mode 1920x1080
xrandr --output eDP-1 --mode 1366x768

# Add custom resolution if not listed
cvt 1920 1080 60                         # Get modeline
# Output: Modeline "1920x1080_60.00" ...
xrandr --newmode "1920x1080_60.00" [paste modeline values]
xrandr --addmode HDMI-1 "1920x1080_60.00"
xrandr --output HDMI-1 --mode "1920x1080_60.00"

# Make permanent via KDE:
# System Settings > Display and Monitor > Display Configuration
```

### ❌ Problem: Screen tearing (especially with NVIDIA)

```bash
# Fix for NVIDIA:
sudo nvidia-settings
# Display Configuration > Force Full Composition Pipeline

# Force via config:
sudo nano /etc/X11/xorg.conf.d/20-nvidia.conf
# Add:
# Section "Screen"
#   Option "metamodes" "nvidia-auto-select +0+0 { ForceFullCompositionPipeline = On }"
# EndSection

# Fix with KDE compositor:
# System Settings > Display and Monitor > Compositor
# Rendering backend: OpenGL 3.1
```

---

## 30. Sound Problems

### ❌ Problem: No sound / audio not working

```bash
# Fix 1: Check if muted
alsamixer                               # Press M to unmute channels
pactl list sinks | grep -i mute

# Fix 2: Restart audio service
systemctl --user restart pipewire
systemctl --user restart pipewire-pulse
# OR:
pulseaudio --kill && sleep 2 && pulseaudio --start

# Fix 3: Check audio devices
aplay -l                                # List playback devices
arecord -l                              # List recording devices
pactl list sinks short                  # List PulseAudio sinks

# Fix 4: Set correct output device
pactl set-default-sink SINK_NAME

# Fix 5: Reinstall audio stack
sudo apt install --reinstall pulseaudio alsa-utils
sudo apt install pipewire pipewire-pulse
sudo reboot

# Fix 6: Intel HDA fix (laptop audio)
sudo nano /etc/modprobe.d/alsa-base.conf
# Add: options snd-hda-intel index=0 model=auto
sudo reboot
```

---

## 31. Metasploit Problems

### ❌ Problem: Metasploit database not connected

```bash
# Error: "WARNING: No database support"

# Fix 1: Start PostgreSQL
sudo systemctl start postgresql
sudo systemctl status postgresql

# Fix 2: Reinitialize the database
sudo msfdb reinit

# Fix 3: Manual database setup
sudo -u postgres psql
# In psql:
CREATE USER msf WITH PASSWORD 'yourpassword';
CREATE DATABASE msf OWNER msf;
\q

# Fix 4: Check msfdb config
cat ~/.msf4/database.yml
# Should show:
# production:
#   adapter: postgresql
#   database: msf
#   username: msf
#   password: yourpassword
#   host: 127.0.0.1
#   port: 5432

# Fix 5: Reset completely
sudo msfdb delete
sudo msfdb init
msfconsole -q -x "db_status"
```

### ❌ Problem: Meterpreter session dies immediately

```bash
# Fix 1: Check firewall rules on attacker
sudo ufw allow PORT/tcp

# Fix 2: Migrate to stable process
meterpreter> ps
meterpreter> migrate PID_OF_EXPLORER   # Migrate to explorer.exe

# Fix 3: Set session handling options
msf> set ExitOnSession false
msf> set SessionRetryTotal 10

# Fix 4: Use persistent handler
msf> handler -H 0.0.0.0 -P 4444 -p windows/x64/meterpreter/reverse_tcp
```

---

## 32. Permission & Root Problems

### ❌ Problem: "Permission denied" errors

```bash
# Check file permissions
ls -la /path/to/file
stat /path/to/file

# Fix: Add execute permission
chmod +x script.sh

# Fix: Change ownership
sudo chown $USER:$USER file

# Fix: Use sudo
sudo command

# Fix: Add to required group
sudo usermod -aG groupname $USER
newgrp groupname                        # Apply without logout

# Common group fixes:
sudo usermod -aG dialout $USER          # Serial ports
sudo usermod -aG plugdev $USER          # USB devices
sudo usermod -aG netdev $USER           # Network
sudo usermod -aG video $USER            # GPU/Video
sudo usermod -aG docker $USER           # Docker
```

### ❌ Problem: Forgotten sudo password

```bash
# Boot to recovery mode from GRUB
# Select: Drop to root shell prompt

# Reset password:
passwd username
passwd root

# OR mount filesystem as read-write:
mount -o remount,rw /
passwd username
reboot
```

---

## 33. USB & Hardware Problems

### ❌ Problem: USB Wi-Fi adapter not recognized

```bash
# Check if USB device is detected
lsusb
# Look for your adapter's vendor ID

# Check kernel messages
dmesg | tail -30
dmesg | grep usb

# Install drivers for common USB adapters:

# Alfa AWUS036ACH (RTL8812AU)
sudo apt install realtek-rtl88xxau-dkms
sudo modprobe 88XXau

# TP-Link TL-WN722N v2/v3 (RTL8188EUS)
sudo apt install realtek-rtl8188eus-dkms

# Alfa AWUS036ACM (mt7612u)
sudo modprobe mt7612u

# Verify it's loaded:
iwconfig
ip a | grep wlan
```

### ❌ Problem: USB device won't mount

```bash
# Check if detected
lsblk
lsusb
dmesg | tail -20 | grep usb

# Manual mount
sudo mkdir /mnt/usb
sudo mount /dev/sdX1 /mnt/usb

# Mount NTFS (Windows drives)
sudo apt install ntfs-3g
sudo mount -t ntfs-3g /dev/sdX1 /mnt/usb

# Mount exFAT
sudo apt install exfat-fuse exfatprogs
sudo mount -t exfat /dev/sdX1 /mnt/usb

# Unmount properly
sudo umount /mnt/usb
```

---

## 34. VPN Problems

### ❌ Problem: OpenVPN won't connect

```bash
# Debug connection
sudo openvpn --config file.ovpn --verb 5   # Verbose output

# Common fixes:
# Fix 1: Install OpenVPN dependencies
sudo apt install openvpn network-manager-openvpn -y

# Fix 2: Route fix
sudo ip route del default                    # Remove conflicting route
sudo openvpn --config file.ovpn

# Fix 3: DNS fix after VPN
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

# Fix 4: TLS handshake timeout
# Add to .ovpn: tls-timeout 10
# Add: connect-retry 5 10

# Fix 5: HackTheBox specific
sudo openvpn --config lab_username.ovpn
# Check tun0: ip a show tun0
```

### ❌ Problem: Can't reach VPN target after connecting

```bash
# Check if VPN interface is up
ip a show tun0

# Check routing
ip route                                 # Should show route to target network via tun0

# Add manual route
sudo ip route add 10.10.10.0/24 via VPN_GATEWAY dev tun0

# Ping gateway
ping -c 3 10.10.10.1

# Check if firewall is blocking
sudo ufw status
sudo iptables -L
```

---

## 35. DNS & Internet Problems

### ❌ Problem: DNS resolution fails after update

```bash
# This is a common Parrot 6.2+ issue

# Fix 1: Restore resolv.conf symlink
sudo rm /etc/resolv.conf
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf

# Fix 2: Enable systemd-resolved
sudo systemctl enable --now systemd-resolved

# Fix 3: Manual DNS
sudo nano /etc/resolv.conf
# Add:
# nameserver 8.8.8.8
# nameserver 1.1.1.1
# Make it immutable to prevent overwriting:
sudo chattr +i /etc/resolv.conf

# Fix 4: NetworkManager DNS
sudo nano /etc/NetworkManager/NetworkManager.conf
# Under [main] add:
# dns=default

sudo systemctl restart NetworkManager
```

---

## 36. KDE / Desktop Problems

### ❌ Problem: KDE Plasma crashes / freezes

```bash
# Restart Plasma shell without logout
kquitapp5 plasmashell
sleep 2
kstart5 plasmashell &

# Or:
plasmashell --replace &

# Fix corrupted KDE config
rm -rf ~/.config/plasma-org.kde.plasma.desktop-appletsrc
kstart5 plasmashell &

# Fix KWin compositor crash
kwin_x11 --replace &

# Complete KDE reset
rm -rf ~/.config/plasma* \
~/.config/kde* \
~/.config/kwin* \
~/.local/share/plasma*
reboot
```

### ❌ Problem: KDE application won't open

```bash
# Check for errors in terminal
application-name &> /tmp/app_error.log
cat /tmp/app_error.log

# Fix missing libraries
sudo apt install --reinstall application-name
ldd /usr/bin/application | grep "not found"  # Find missing libs
sudo apt install missing-library

# Fix Qt/KDE framework issues
sudo apt install --reinstall libqt5core5a libkf5* qtbase5-dev
```

---

## 37. Tool-Specific Problems

### ❌ Aircrack-ng: "No such BSSID available"

```bash
# Fix: Make sure you're capturing on correct channel
sudo airodump-ng wlan0mon --bssid TARGET_BSSID -c CHANNEL -w capture

# Verify monitor mode
iwconfig wlan0mon                       # Should show Mode:Monitor
```

### ❌ Burp Suite: Certificate not trusted

```bash
# Fix: Install Burp CA certificate in browser
# 1. Start Burp Suite
# 2. Go to http://burp in browser → Download CA Certificate
# 3. In Firefox: Settings > Privacy > Certificates > Import
# 4. Check "Trust to identify websites"
# Or use FoxyProxy to easily toggle proxy on/off
```

### ❌ SQLmap: Too slow / getting blocked

```bash
sqlmap -u "URL" --delay=2 --random-agent  # Add delays, random UA
sqlmap -u "URL" --tor --check-tor          # Route through Tor
sqlmap -u "URL" --tamper=space2comment     # Use tamper scripts for WAF bypass
sqlmap -u "URL" --level=1 --risk=1        # Start conservatively
```

### ❌ Nmap: Permission denied for SYN scan

```bash
# SYN scan requires root
sudo nmap -sS target                     # Run as root
# OR use TCP connect scan (no root needed):
nmap -sT target
```

### ❌ Hashcat: No devices found / OpenCL error

```bash
# Fix 1: Install OpenCL
sudo apt install ocl-icd-opencl-dev opencl-headers -y

# Fix 2: NVIDIA OpenCL
sudo apt install nvidia-opencl-dev

# Fix 3: Run as root
sudo hashcat -m 0 hash.txt wordlist.txt

# Fix 4: Use CPU instead
hashcat -m 0 hash.txt wordlist.txt -d 1  # -d 1 = device 1
hashcat -m 0 hash.txt wordlist.txt --force  # Force CPU

# Fix 5: Check detected devices
hashcat -I
clinfo
```

### ❌ Wireshark: No interfaces / Permission denied

```bash
# Fix: Add user to wireshark group
sudo usermod -aG wireshark $USER
sudo dpkg-reconfigure wireshark-common  # Select "Yes" for non-root capture
newgrp wireshark
# Logout and login to apply

# Alternative: Run as root
sudo wireshark
```

---

## 38. Performance Problems

### ❌ System running slow

```bash
# Check CPU and memory usage
htop
btop

# Check disk I/O
iotop                                   # Install: sudo apt install iotop
iostat -x 1                             # Install: sudo apt install sysstat

# Check temperature
sensors                                 # Install: sudo apt install lm-sensors
sudo sensors-detect                     # Detect sensors

# Check for runaway processes
ps aux --sort=-%cpu | head -10          # Top CPU users
ps aux --sort=-%mem | head -10          # Top memory users

# Free memory
sync && echo 3 | sudo tee /proc/sys/vm/drop_caches  # Clear page cache

# Disable unnecessary services
sudo systemctl disable bluetooth        # Disable Bluetooth (if unused)
sudo systemctl disable cups             # Disable printer service

# SSD optimization
sudo systemctl enable fstrim.timer      # Weekly TRIM for SSDs

# Check startup time
systemd-analyze blame | head -20        # Show slow-starting services
```

---

## 39. Disk & Filesystem Problems

### ❌ Problem: Disk full

```bash
# Check disk usage
df -h                                   # Disk usage by partition
du -sh /*  2>/dev/null | sort -h        # Usage by directory
du -sh ~/* 2>/dev/null | sort -h        # Usage in home directory

# Find large files
find / -size +100M -type f 2>/dev/null  # Files over 100MB
find / -size +1G -type f 2>/dev/null    # Files over 1GB

# Clean up space
sudo apt clean                          # Clear package cache
sudo apt autoremove                     # Remove unused packages
sudo journalctl --vacuum-size=100M      # Limit journal size
docker system prune -a                  # Clean Docker (if installed)
rm -rf ~/.local/share/Trash/*           # Empty trash
find /tmp -mtime +7 -delete             # Delete old temp files

# Find and remove old kernels
dpkg --list | grep linux-image
sudo apt purge linux-image-OLD-VERSION
sudo update-grub
```

### ❌ Problem: Filesystem errors

```bash
# Check filesystem
sudo fsck /dev/sdX1                     # Check and repair
sudo fsck -y /dev/sdX1                  # Auto-fix errors

# For running filesystem (must unmount first or use live USB):
sudo umount /dev/sdX1
sudo e2fsck -f /dev/sdX1               # ext4 check
sudo ntfsfix /dev/sdX1                 # NTFS fix

# Check SMART disk health
sudo apt install smartmontools
sudo smartctl -a /dev/sda               # Full disk health report
sudo smartctl -t short /dev/sda         # Run short test
```

---

## 40. System Log Analysis

### 📋 Reading and Understanding Logs

```bash
# === KEY LOG FILES ===
/var/log/syslog              # General system messages
/var/log/auth.log            # Authentication attempts (SSH, sudo, login)
/var/log/kern.log            # Kernel messages
/var/log/dpkg.log            # Package installation/removal
/var/log/apt/history.log     # APT command history
/var/log/ufw.log             # Firewall logs
/var/log/nginx/error.log     # Nginx errors
/var/log/apache2/error.log   # Apache errors

# === READING LOGS ===
tail -f /var/log/syslog                 # Follow live
tail -100 /var/log/auth.log             # Last 100 lines
grep "error\|fail\|warn" /var/log/syslog  # Filter errors
grep "Failed password" /var/log/auth.log  # Failed SSH attempts
journalctl -xe                          # Recent systemd errors
journalctl -u service-name -f          # Follow service logs
journalctl --since "2025-01-01" --until "2025-01-02"  # Date range

# === USEFUL LOG ANALYSIS COMMANDS ===
# Find all failed SSH logins:
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Find all successful sudo uses:
grep "sudo:" /var/log/auth.log | grep "COMMAND"

# Check which packages were recently installed:
grep "install" /var/log/apt/history.log | tail -20

# Check for kernel errors:
dmesg -T | grep -i "error\|fail\|warn"
dmesg -T --level=err,warn

# Monitor real-time system activity:
watch -n 1 'ss -tulnp'                  # Watch open ports
watch -n 2 'ps aux --sort=-%cpu | head'  # Watch CPU-hungry processes
```

---

## 🆘 Emergency Recovery Procedures

### 💊 Complete System Recovery Checklist

```bash
# If system is broken/unbootable:
# 1. Boot from Parrot Live USB

# 2. Mount broken system
sudo fdisk -l
sudo mount /dev/sdaX /mnt              # Mount root partition
sudo mount /dev/sdaY /mnt/boot         # Mount boot (if separate)
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo mount --bind /run /mnt/run

# 3. Chroot into broken system
sudo chroot /mnt

# 4. Fix the issue:
apt update && apt upgrade              # Update packages
dpkg --configure -a                    # Configure broken packages
update-grub && grub-install /dev/sda  # Fix GRUB
passwd username                         # Reset password
systemctl disable broken-service        # Disable bad service

# 5. Exit and reboot
exit
sudo umount -R /mnt
sudo reboot
```

---

## 📞 Getting Help Resources

| Resource | URL | Purpose |
|----------|-----|---------|
| **Parrot Official Docs** | https://parrotsec.org/docs/ | Official documentation |
| **Parrot Community Forum** | https://community.parrotsec.org/ | Ask questions |
| **Parrot GitLab** | https://gitlab.com/ParrotSec | Bug reports |
| **Parrot Reddit** | https://reddit.com/r/ParrotOS | Community help |
| **Debian Wiki** | https://wiki.debian.org/ | Underlying system docs |
| **ArchWiki** | https://wiki.archlinux.org/ | Excellent Linux reference |
| **AskUbuntu** | https://askubuntu.com/ | Debian-based solutions |
| **LinuxQuestions** | https://linuxquestions.org/ | Community support |
| **Stack Overflow** | https://stackoverflow.com/ | Programming/scripting help |

### 🔍 Self-Help Before Asking

```bash
# Always try these first:
man command-name                         # Read the manual
command --help                           # Quick help
info command-name                        # Detailed info pages
apropos keyword                          # Find related commands
journalctl -xe                           # Check recent errors
dmesg | tail -20                         # Kernel messages
google: "parrot os [your error message]" # Search the error
```

---

*🦜 Parrot Security OS — https://parrotsec.org*
*⚠️ Always test configurations in a VM before applying to production systems*
*📌 For authorized security testing and learning ONLY*
