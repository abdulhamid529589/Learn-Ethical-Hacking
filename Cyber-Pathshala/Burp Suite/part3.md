# Burp Suite — Proxying an Android Device for Mobile/APK Pen Testing

### Study Notes: LAN-Based Android-to-Burp Setup

---

## Index

1. [Overview & Use Case](#1-overview--use-case)
2. [Networking Concept: Why Two Private IPs Can Talk](#2-networking-concept-why-two-private-ips-can-talk)
3. [Lab Topology](#3-lab-topology)
4. [Step 1 — Configure VM Networking (Bridged Adapter)](#4-step-1--configure-vm-networking-bridged-adapter)
5. [Step 2 — Connect the Android Device to the Same Wi-Fi](#5-step-2--connect-the-android-device-to-the-same-wi-fi)
6. [Step 3 — Start Burp Suite on Kali](#6-step-3--start-burp-suite-on-kali)
7. [Step 4 — Bind Burp's Proxy Listener to All Interfaces](#7-step-4--bind-burps-proxy-listener-to-all-interfaces)
8. [Step 5 — Identify Kali's LAN IP Address](#8-step-5--identify-kalis-lan-ip-address)
9. [Step 6 — Configure the Proxy on the Android Device](#9-step-6--configure-the-proxy-on-the-android-device)
10. [Step 7 — Install Burp's CA Certificate on Android](#10-step-7--install-burps-ca-certificate-on-android)
11. [Step 8 — Verifying the Connection](#11-step-8--verifying-the-connection)
12. [Quick Reference Cheat Sheet](#12-quick-reference-cheat-sheet)
13. [Common Errors & Fixes](#13-common-errors--fixes)
14. [Notes for Modern Android Versions (7+) / App-Specific Pinning](#14-notes-for-modern-android-versions-7--app-specific-pinning)
15. [Ethical & Legal Notes](#15-ethical--legal-notes)

---

## 1. Overview & Use Case

Everything covered in earlier lessons connected **Burp ↔ your own browser** on the same machine (`127.0.0.1:8080`). This lesson extends that to a **second physical device on the same network** — an Android phone — so you can intercept and test traffic from **mobile apps and mobile browsers**, not just your desktop browser.

**Why this matters:** many real-world pentesting/bug-bounty engagements involve **Android APKs** that talk to backend APIs over HTTP/HTTPS. To test those APIs (auth flows, IDOR, business logic, etc.) the same way you test a web app, you need the same visibility Burp gives you on desktop — which means routing the phone's traffic through Burp too.

**Key clarification from the lesson:** this Android-to-Burp connection can be done over a **local area network** (as demonstrated here) or, with additional setup (port forwarding, a reachable public/VPS IP, etc.), over a **wide area network** — but LAN is the simpler and recommended starting point for lab practice.

---

## 2. Networking Concept: Why Two Private IPs Can Talk

Recap from earlier lessons: a **private IP cannot talk directly to a public IP** without a router in between. But **two private IPs on the same local network CAN talk to each other directly** — this is the key fact that makes phone-to-Burp proxying possible without any internet-facing infrastructure.

```
Kali Machine (attacker/tester)          Android Device
Private IP: 192.168.1.20                Private IP: 192.168.1.22
   │                                        │
   └──────────── same LAN / same Wi-Fi ─────┘
        (both devices can address each other directly)
```

Since both devices share the same Wi-Fi network and both have private IPs in the same subnet, the Android device can be pointed directly at Kali's IP + Burp's port — no public IP, router configuration, or port forwarding required.

---

## 3. Lab Topology

| Device          | Role                                          | Identity needed                                                       |
| --------------- | --------------------------------------------- | --------------------------------------------------------------------- |
| Kali Linux (VM) | Runs Burp Suite (the proxy)                   | Private LAN IP (e.g., `192.168.1.20`), reachable on port `8080`       |
| Android phone   | Generates traffic to be tested (apps/browser) | Connected to the **same Wi-Fi SSID** as the Kali VM's bridged network |

---

## 4. Step 1 — Configure VM Networking (Bridged Adapter)

By default, a VM's NAT adapter puts it behind an isolated virtual network (commonly `10.0.x.x`) that your phone **cannot** reach — the phone and the VM aren't on the same broadcast domain. You need a **Bridged Adapter** instead, which makes the VM appear as a full device on your actual home/office Wi-Fi network.

**In VirtualBox / VMware network settings for the Kali VM:**

1. Open the VM's **Network Settings**.
2. **Adapter 1**: set to **Bridged Adapter**, and select your real physical Wi-Fi/Ethernet adapter (the one your host machine uses to connect to your router).
3. **Adapter 2** (optional, kept for convenience/fallback): set to **NAT**, so you still have a general-purpose internet-connected interface if needed.
4. Ensure both adapters are **enabled/ticked**, then click OK.
5. Start/restart the Kali VM.

**Result:** Kali now has (potentially) two IPs — one from the bridged adapter (in your real router's subnet, e.g., `192.168.1.x`) and one from NAT (e.g., `10.0.3.x`). You'll pick the **bridged** one for this exercise, since that's the subnet your phone is actually on.

---

## 5. Step 2 — Connect the Android Device to the Same Wi-Fi

1. On the Android device, go to **Settings → Wi-Fi**.
2. Connect to the **exact same SSID** your host machine (and therefore the bridged Kali VM) is using — e.g., `YourNetwork_5G`.
3. Confirm connection is active before proceeding.

> Both devices being on the same SSID/subnet is the whole reason this works without extra routing configuration — double-check this first if anything fails later.

---

## 6. Step 3 — Start Burp Suite on Kali

```bash
# Navigate to your Burp install directory
cd ~/Desktop/BurpSuite

# Launch (Community or Professional, launch script name may vary)
./bsh
```

Click through **Next → Start Burp** to reach the main interface, same as previous lessons.

---

## 7. Step 4 — Bind Burp's Proxy Listener to All Interfaces

By default, Burp's proxy listener only binds to `127.0.0.1` (**loopback only** — reachable only from the same machine). To accept connections from another device like your phone, you must rebind it to listen on **all network interfaces**.

1. Go to **Proxy → Options** (or **Proxy Settings**, depending on Burp version).
2. Confirm **Intercept is turned OFF** for now (you don't want it stuck intercepting while you set things up).
3. Under **Proxy Listeners**, select the existing `127.0.0.1:8080` entry → click **Edit**.
4. Under **Binding**, change from **"Loopback only"** to **"All interfaces."**
5. Click **OK**, then confirm the follow-up warning dialog (Burp will warn you this exposes the listener more broadly — acceptable and expected on a private, trusted LAN for lab purposes).
6. The listener entry should now display as `*:8080` — the asterisk indicates it's listening on **every IP address the machine has** (loopback, LAN/bridged, and any other active interface), not just `127.0.0.1`.

> ⚠️ **Security note:** binding to all interfaces means _any_ device that can reach your machine's IP on port 8080 can now route traffic through your Burp instance — fine on a trusted home/lab network, but you would **not** want this listener exposed on an untrusted or public network. Revert to "Loopback only" when you're done with cross-device testing.

---

## 8. Step 5 — Identify Kali's LAN IP Address

You need Kali's **bridged-adapter IP** specifically (not the NAT IP) — this is the address your phone will actually be able to reach.

```bash
ifconfig
# or
ip a
```

- The **NAT adapter** IP typically looks like `10.0.3.x` — **not** what you want here.
- The **Bridged adapter** IP typically looks like `192.168.x.x` — **this** is the one to use.

**Tip if both interfaces are showing and it's confusing which is which:** temporarily disable the NAT adapter (via VM settings or `ip link set <interface> down`) so only the bridged interface's IP appears, confirming which one you need. Re-enable NAT afterward if you still want general internet access on that interface.

Example result used throughout this lesson: **`192.168.1.20`**

---

## 9. Step 6 — Configure the Proxy on the Android Device

1. On Android: **Settings → Wi-Fi**.
2. Tap and hold (or tap the settings/pencil icon) on your connected network → **Modify network** / **Advanced options**.
3. You may be prompted to re-enter the Wi-Fi password to make changes — enter it.
4. Scroll to **Proxy** → change from **None** to **Manual**.
5. Enter:
   - **Proxy hostname**: Kali's bridged IP (e.g., `192.168.1.20`)
   - **Proxy port**: `8080`
6. Leave other fields (like Bypass proxy for) empty unless you have a specific reason to exclude certain hosts.
7. **Save.**

**Result:** all of the Android device's HTTP/HTTPS traffic now routes through Burp on your Kali machine — but HTTPS traffic will fail/show warnings until you complete the certificate installation next, since the device doesn't yet trust Burp's re-signed certificates.

---

## 10. Step 7 — Install Burp's CA Certificate on Android

Same underlying concept as installing Burp's cert in a desktop browser (covered in Lesson 1), just via Android's certificate manager.

1. On the Android device, open any browser and navigate to:
   ```
   http://burp
   ```
   (⚠️ plain `http`, no `https`, no dots/paths — same special address as the desktop setup)
2. You'll see Burp's built-in page with a **CA Certificate** download link (usually top-right). Tap it → the certificate file downloads.
3. Go to **Settings** → search **"certificate"** → **Install a certificate** (or **Encryption & credentials → Install a certificate**, path varies by Android version/manufacturer) → **CA certificate** / **Wi-Fi certificate**.
4. You'll likely see a warning that installing a CA certificate lets the certificate owner monitor your network activity — this is expected and correct (that's literally what you're setting up for testing purposes) → tap **Install anyway**.
5. Authenticate with your fingerprint/PIN/pattern if prompted.
6. Select the downloaded certificate file (if multiple exist, choose the most recently downloaded one) → confirm installation.

**Verify installation:**

1. **Settings → Security/Certificate → Trusted credentials** (or **Certificate Management app** on some Android versions).
2. Go to the **User** tab (user-installed certs, as opposed to system-preloaded ones).
3. Confirm you see an entry from **"PortSwigger"** (Burp's parent company) — this confirms the cert is installed and trusted.

---

## 11. Step 8 — Verifying the Connection

1. On the Android device, open a browser and navigate to any test site (e.g., your own lab domain).
2. On Kali, go to **Proxy → Intercept**, turn it **ON**, and confirm you see request packets arriving that originate from the Android device (you'll recognize them by the target hostname, e.g., your test domain).
3. **Optional proof-of-concept test — modify the response:**
   - Right-click the intercepted request → **"Do intercept" → "Response to this request."**
   - Forward the request through; the corresponding **response** will now pause in Intercept.
   - Edit the response body directly — e.g., find a text string like `Home` in the HTML and change it to something else (e.g., `Ashish`), or change a heading's text.
   - Click **Forward**.
   - On the Android device's browser, the modified text now appears on the rendered page — definitive proof that Burp is actively intercepting and altering traffic between the phone and the target server.
4. Turn **Intercept back OFF** once verified, so normal browsing/testing can proceed without manually forwarding every packet.

---

## 12. Quick Reference Cheat Sheet

| Task                                   | Where / Command                                                             |
| -------------------------------------- | --------------------------------------------------------------------------- |
| Set VM to bridged networking           | VM Network Settings → Adapter 1 → Bridged Adapter                           |
| Find Kali's LAN IP                     | `ifconfig` or `ip a` (use the bridged-subnet IP, not NAT)                   |
| Rebind Burp listener to all interfaces | Proxy → Options → Proxy Listeners → Edit → Binding → All interfaces         |
| Set Android's proxy                    | Wi-Fi → Modify network → Advanced → Proxy → Manual → `<Kali IP>:8080`       |
| Get Burp's CA cert on Android          | Browser → `http://burp` → download CA Certificate                           |
| Install cert on Android                | Settings → Install a certificate → CA certificate → select downloaded file  |
| Verify cert installed                  | Settings → Trusted credentials → User tab → look for "PortSwigger"          |
| Confirm traffic flowing                | Proxy → Intercept ON → browse on phone → watch for phone's requests in Burp |

---

## 13. Common Errors & Fixes

| Symptom                                                                          | Likely Cause                                                                             | Fix                                                                                            |
| -------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Phone can't reach Kali's IP at all                                               | VM still on NAT, not bridged; or wrong IP used                                           | Re-check §4 (bridged adapter) and §8 (correct interface's IP)                                  |
| Phone Wi-Fi shows "connected, no internet" after setting proxy                   | Burp not actually running, or listener still loopback-only                               | Confirm Burp is running and listener shows `*:8080` (§7)                                       |
| HTTPS sites fail / cert warnings on Android                                      | CA certificate not installed or not trusted                                              | Redo §10 — confirm cert appears under **Trusted credentials → User**                           |
| Nothing shows up in Burp's Intercept/HTTP History                                | Proxy setting on phone didn't save, or wrong IP/port typed                               | Re-verify §9 exactly; toggle Wi-Fi off/on on the phone to force settings to re-apply           |
| Some apps still bypass Burp / show connection errors even with proxy+cert set up | The specific app uses **certificate pinning** (hardcoded trust, ignores system CA store) | See §14 below — this requires additional bypass techniques, out of scope for basic proxy setup |
| Kali shows two IPs and you're unsure which to use                                | Both NAT and Bridged adapters active simultaneously                                      | Temporarily disable NAT adapter to isolate the bridged IP, per §8                              |

---

## 14. Notes for Modern Android Versions (7+) / App-Specific Pinning

Two important real-world caveats not covered in depth in this lesson but essential to know before relying on this setup for serious app testing:

1. **System-wide "Trusted credentials" limitation (Android 7+):** starting with Android Nougat (API 24+), apps **targeting** that API level or higher **by default ignore user-installed CA certificates** (like the one you just installed) for their own network connections — even though your browser will happily trust it. This means the setup in this lesson reliably works for **browser traffic**, but many **individual APKs** will silently refuse to send traffic through Burp at all, or fail with SSL errors, unless:
   - The APK's manifest explicitly opts in to trusting user certs (`network_security_config.xml` with `<certificates src="user"/>`), or
   - You use additional tooling (e.g., **Frida**, **Objection**, or repackaging the APK) to force the app to trust user certificates or bypass its network security config — this is a deeper mobile-app-pentesting topic for a dedicated lesson.
2. **Certificate pinning:** some apps (especially banking, payment, and security-conscious apps) hardcode ("pin") the exact certificate or public key they expect from their backend server, entirely bypassing the OS certificate store. Even a perfectly installed Burp CA cert won't help here — the app will refuse to talk to "not the real server" regardless of trust settings. Bypassing pinning requires app-specific techniques (Frida scripts, patching the APK's smali code, using tools like **objection**'s `sslpinning disable`) — again, a separate advanced topic.

**Practical takeaway for your lab practice:** this lesson's setup is the correct **foundation** for mobile pentesting and will work great for testing your **browser traffic** and **simple/unhardened test APKs**. For real commercial apps with modern security practices, expect to need the additional bypass techniques above — a natural "next lesson" topic once this foundational proxy setup is solid.

---

## 15. Ethical & Legal Notes

- Only proxy and inspect traffic from **your own Android device**, running **your own apps** or apps you have explicit authorization to test (e.g., an app you built, or an in-scope target under a signed bug bounty/pentest engagement).
- Installing a third-party root CA certificate that lets a proxy decrypt your HTTPS traffic is a significant trust decision — only do this on a **dedicated testing device** (or a device/profile you don't use for personal banking, personal accounts, etc.), never on your primary daily-use phone. If you don't have a spare Android device, use an **Android emulator** (e.g., Android Studio's emulator, or Genymotion) for this kind of testing instead — safer and just as effective for most app-testing purposes.
- Remember to **revert** the proxy setting back to "None" and optionally remove the installed CA certificate when you're done testing, especially if this is a personal daily-use device — leaving an intercepting proxy configured indefinitely is both a security risk (if you rejoin other networks) and unnecessary once your testing session is complete.
- As with all Burp-based testing covered in this series: authorization is what makes this legal and legitimate. The exact same technical setup is neutral — what matters is _whose_ traffic and _whose app_ you're pointing it at.
