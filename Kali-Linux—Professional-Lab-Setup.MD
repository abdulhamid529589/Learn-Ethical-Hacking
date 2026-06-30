# 🐧 Kali Linux — Professional Lab Setup & Customization
> **Study notes** from video tutorial (Hindi). These tips help you customize your Kali Linux machine into a professional-looking ethical hacking lab environment.

---

## Table of Contents

1. [Enable Root User](#1-enable-root-user)
2. [Check & Update System Version](#2-check--update-system-version)
3. [GUI & Desktop Customization](#3-gui--desktop-customization)
4. [Custom Terminal Banner — Figlet & Lolcat](#4-custom-terminal-banner--figlet--lolcat)
5. [Appearance, Themes & Icons](#5-appearance-themes--icons)
6. [Dock Bar — Plank](#6-dock-bar--plank)
7. [Advanced Terminal — Terminator](#7-advanced-terminal--terminator)
8. [Better Process Monitor — htop](#8-better-process-monitor--htop)
9. [Upgrade to GNOME Desktop](#9-upgrade-to-gnome-desktop)
10. [Hacker-Style Terminal (Bonus)](#10-hacker-style-terminal-bonus)

---

## 1. Enable Root User

**Why:** By default, many commands require `sudo` every time. Enabling the root user lets you work without repeated permission prompts.

### Problem Without Root
```bash
apt update
# Error: Permission denied
```

### Fix: Use sudo temporarily
```bash
sudo apt update
```

### Enable Root User Permanently

**Step 1 — Set a root password:**
```bash
sudo passwd root
# Enter your current user password
# Then set a new password for root
```

**Step 2 — Switch to root user:**
```bash
su root
# Enter the root password you just set
```

**Step 3 — Or, use sudo to get a root shell directly:**
```bash
sudo su
# Now all commands run as root — no need to prefix with sudo
```

```bash
# Example: now this works without sudo
apt get update
```

> **Security Note:** Working as root permanently is convenient but risky. Use it in isolated lab environments only.

---

## 2. Check & Update System Version

### Update Package List
```bash
sudo apt update
```

### Full System Upgrade
```bash
sudo apt upgrade -y
```

> Run these two commands regularly to keep your Kali Linux tools and packages up to date.

---

## 3. GUI & Desktop Customization

### Change Wallpaper
- Right-click on Desktop → **Desktop Settings**
- Browse available wallpapers and click **Apply**
- You can also download more wallpapers from the Kali Linux website

### Taskbar / Panel Customization
- Right-click on the Desktop → **Panel** → **Panel Preferences**
- Add/remove applets (Application Menu, Icon Bar, etc.)
- Adjust icon size, spacing, and position

### Show Hidden Files on Desktop
- Right-click on Desktop → **Desktop Settings** → enable **Show Hidden Files**

### Remove Unused Apps (Improves Performance)
- Uninstall apps you don't use via the package manager
- Removing bloat directly improves system speed and RAM usage

---

## 4. Custom Terminal Banner — Figlet & Lolcat

**Why:** Every time you open a terminal, display a colorful custom banner — looks professional and impressive.

### Install the Tools
```bash
sudo apt install figlet lolcat -y
```

### Test it Manually First
```bash
figlet -f future "Your Text Here" | lolcat
```

### Font Options for Figlet
```bash
figlet -f banner   "Your Text"   # Block style
figlet -f digital  "Your Text"   # Digital display style
figlet -f future   "Your Text"   # Futuristic style (recommended)
```

### Make the Banner Permanent (Auto-loads Every Terminal)

**Step 1 — Open your shell config file:**
```bash
# For Zsh (default on newer Kali):
nano ~/.zshrc

# For Bash (older Kali):
nano ~/.bashrc
```

**Step 2 — Scroll to the bottom and add:**
```bash
figlet -f future "  Your Name  " | lolcat
```

**Step 3 — Save and exit:**
```
Press Esc → type :wq → press Enter
```

**Step 4 — Restart terminal to see the result.**

> Every time you open a new terminal, your custom colorful banner will appear automatically.

---

## 5. Appearance, Themes & Icons

### Open Appearance Settings
- Search "Appearance" in the application menu, or run:
```bash
xfce4-appearance-settings
```

### Available Themes

| Theme | Style |
|-------|-------|
| Kali-Dark | Dark (recommended for hacking labs) |
| Kali-Light | Light/White |
| High Contrast | For accessibility |
| Windows-like | Mimics Windows UI |

### Change Icon Pack

| Icon Pack | Look |
|-----------|------|
| Flat | Minimal, default |
| GNOME | Clean, modern |
| Tango | Colorful, classic |
| High Contrast | Bold, visible |

### Font Settings
- Go to **Appearance → Fonts**
- Change the system font if the default doesn't look good
- Recommended: keep default unless you have a preference

### Settings → Menu & Buttons
- **Show images in menus** — uncheck to speed up menus
- **Show images on buttons** — uncheck for a cleaner UI
- **Editable accelerators** — enable if you want keyboard shortcut editing
- **Scaling** — keep at `1x` (higher values cause display issues)

---

## 6. Dock Bar — Plank

**Why:** Adds a macOS-style animated dock bar at the bottom (or any edge) of the screen.

### Install Plank
```bash
sudo apt install plank -y
```

### Launch Plank
- Search "Plank" in the application menu → click **Run Plank**

### Configure Plank
- Right-click the dark circle at the edge of the dock → **Preferences**

| Setting | Description |
|---------|-------------|
| Icon Zoom | Enable hover zoom effect (macOS feel) |
| Zoom Size | How much icons enlarge on hover |
| Icon Size | Default icon size on the dock |
| Alignment | Center / Start / End / Fill |
| Position | Top / Bottom / Left / Right |
| Auto-Hide | Hide dock when not in use |
| Dock Type | Solid / Transparent / etc. |

> **Tip:** Set Alignment to **Center** and Position to **Bottom** for the classic macOS dock look.

---

## 7. Advanced Terminal — Terminator

**Why:** Split your terminal into multiple panes — run commands side by side without switching windows.

### Check if Already Available (New Kali)
- Right-click inside any terminal → look for **Split Terminal Horizontal / Vertical**
- If available, you don't need Terminator

### Install Terminator (Older Kali / Not Available)
```bash
sudo apt install terminator -y
```

### Launch Terminator
- Search "Terminator" in the application menu → **Run Terminal Emulator**

### How to Split the Terminal

| Action | How |
|--------|-----|
| Split Horizontally | Right-click → **Split Horizontally** |
| Split Vertically | Right-click → **Split Vertically** |
| Close a pane | Type `exit` in that pane |
| Add more splits | Right-click any pane and split again |

### Example Layout
```
┌─────────────────────┬─────────────────────┐
│  Terminal 1         │  Terminal 2         │
│  (running nmap)     │  (running hydra)    │
├─────────────────────┴─────────────────────┤
│  Terminal 3 (monitoring with htop)        │
└───────────────────────────────────────────┘
```

---

## 8. Better Process Monitor — htop

**Why:** `top` is basic. `htop` gives a colorful, interactive, easy-to-read system monitor.

### Install htop
```bash
sudo apt install htop -y
```

### Compare: top vs htop

| Feature | `top` | `htop` |
|---------|-------|--------|
| Color display | No | Yes |
| CPU per-core bars | No | Yes |
| Memory bar graph | No | Yes |
| Mouse support | No | Yes |
| Easy to kill process | No | Yes (F9) |
| Overall readability | Basic | Much better |

### Launch htop
```bash
htop
```

### Useful htop Shortcuts

| Key | Action |
|-----|--------|
| `F2` | Setup / customize columns |
| `F3` | Search process |
| `F5` | Tree view |
| `F6` | Sort by column |
| `F9` | Kill a process |
| `F10` | Quit |
| `q` | Quit |

---

## 9. Upgrade to GNOME Desktop

**Why:** GNOME is a more modern, polished desktop environment compared to the default Xfce. It changes the login screen, app launcher, and overall feel.

### Install GNOME
```bash
sudo apt update
sudo apt install kali-desktop-gnome -y
```

- During installation, when asked about the display manager, select **gdm3**

### Reboot After Installation
```bash
sudo reboot
```

### After Reboot
- The login screen will look completely different (GDM3)
- Select your user and log in
- The desktop, app drawer, and system settings will now use GNOME

### GNOME Appearance Settings
- Right-click desktop → **Display Settings** or **Appearance**
- Set theme to **Dark**
- Choose a new wallpaper from the improved collection
- Accessibility options (large fonts, high contrast, screen reader) are available under **Accessibility**

### Desktop Icon Settings (GNOME)
Right-click desktop → **Desktop Icon Settings**:

| Option | Recommended |
|--------|-------------|
| Show Personal Folder on Desktop | Enable |
| Show Trash Icon on Desktop | Enable |
| Show External Drives on Desktop | Enable |
| Show Network Drives on Desktop | Enable |
| Dark text on icon labels | Optional |
| Open files with single click | Personal preference |
| Show image thumbnails | Always |

### Remove Old Desktop Environment (Optional — Frees Up Space)
After confirming GNOME works well, remove the old Xfce packages:
```bash
sudo apt remove kali-desktop-xfce xfce* lightdm --purge -y
sudo apt autoremove -y
```

> ⚠️ Only do this after confirming GNOME is fully working. Removing Xfce while GNOME isn't working will leave you without a desktop.

---

## 10. Hacker-Style Terminal (Bonus)

**Why:** Recreate the "movie hacker" terminal look with a live animated interface showing network status, IP, bandwidth, time, and system info.

### How to Get It
1. Open your browser and search for **"Hacker Terminal Linux"** or look for tools like **HackerTyper**, **Cool Retro Term**, or similar tools
2. Download the **64-bit Linux** version
3. Save the file

### Make the File Executable
- Right-click the downloaded file → **Properties**
- Go to the **Permissions** tab
- Check **"Allow this file to run as a program"**
- Close Properties

### Run It
- Double-click the file → select **"Run as Program"**
- A terminal-like interface launches with sound effects and animated display

### Features Available in This Terminal

| Feature | Description |
|---------|-------------|
| Network bandwidth monitor | Live upload/download stats |
| IP detection | Shows your current IP |
| Processor usage | CPU load in real time |
| Geographic location | IP-based location display |
| Time & date | Live clock display |
| Keyboard shortcut display | Shows available actions |

---

## 🗂️ Customization Summary Checklist

| # | Tip | Done? |
|---|-----|-------|
| 1 | Enable root user (`sudo passwd root`) | ☐ |
| 2 | Update & upgrade system (`apt update && apt upgrade`) | ☐ |
| 3 | Customize desktop wallpaper, panel, and icons | ☐ |
| 4 | Install figlet + lolcat, add banner to `.zshrc` | ☐ |
| 5 | Set GNOME/dark theme and preferred icon pack | ☐ |
| 6 | Install and configure Plank dock | ☐ |
| 7 | Install Terminator for split-pane terminals | ☐ |
| 8 | Install and use htop instead of top | ☐ |
| 9 | Upgrade to GNOME desktop environment | ☐ |
| 10 | Download and run hacker-style terminal | ☐ |

---

## 📦 All Commands at a Glance

```bash
# System update
sudo apt update && sudo apt upgrade -y

# Enable root
sudo passwd root

# Install banner tools
sudo apt install figlet lolcat -y

# Install dock
sudo apt install plank -y

# Install split terminal
sudo apt install terminator -y

# Install better process monitor
sudo apt install htop -y

# Install GNOME desktop
sudo apt install kali-desktop-gnome -y

# Reboot
sudo reboot

# Remove old desktop (after confirming GNOME works)
sudo apt remove kali-desktop-xfce xfce* lightdm --purge -y
sudo apt autoremove -y
```

---

*Notes compiled for educational and lab setup purposes.*