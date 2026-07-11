# Understanding the Layers of the Internet & Hosting a Tor Hidden Service

## https://www.youtube.com/watch?v=9ZOBfc4lHVg

> **Course:** Cyber Security — Web Fundamentals & Anonymity Networks
> **Topic:** Surface Web, Deep Web, Dark Web, Hidden Web — Concepts & Differences, plus a Practical Walkthrough of Hosting Your Own Tor (.onion) Hidden Service

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Foundation Concept: What is a URL?](#2-foundation-concept-what-is-a-url)
   - 2.1 [URL Syntax Breakdown](#21-url-syntax-breakdown)
   - 2.2 [The Golden Rule of the Internet](#22-the-golden-rule-of-the-internet)
3. [The Surface Web](#3-the-surface-web)
4. [The Deep Web](#4-the-deep-web)
5. [The Dark Web](#5-the-dark-web)
6. [The Hidden Web](#6-the-hidden-web)
7. [How a Website Can Move Between Categories](#7-how-a-website-can-move-between-categories)
8. [How the Dark Web Keeps Users Anonymous](#8-how-the-dark-web-keeps-users-anonymous)
9. [Is the Dark Web Completely Untraceable?](#9-is-the-dark-web-completely-untraceable)
10. [Practical: Hosting Your Own Tor Hidden Service (.onion site)](#10-practical-hosting-your-own-tor-hidden-service-onion-site)
    - 10.1 [Step 1: Install Tor](#101-step-1-install-tor)
    - 10.2 [Step 2: Host a Basic Website Locally with Apache](#102-step-2-host-a-basic-website-locally-with-apache)
    - 10.3 [Step 3: Run Tor and Build the Circuit](#103-step-3-run-tor-and-build-the-circuit)
    - 10.4 [Step 4: Configure the Hidden Service in torrc](#104-step-4-configure-the-hidden-service-in-torrc)
    - 10.5 [Step 5: Find Your .onion Address](#105-step-5-find-your-onion-address)
    - 10.6 [Step 6: Access It via Tor Browser](#106-step-6-access-it-via-tor-browser)
11. [What's Coming Next](#11-whats-coming-next)
12. [Summary / Quick Revision Table](#12-summary--quick-revision-table)

---

## 1. Introduction

Many rumors circulate about the "dark web" — some true, some exaggerated. This lesson aims to give a **clear, structured understanding** of the different layers of the internet: **Surface Web, Deep Web, Dark Web, and Hidden Web**, followed by a **hands-on practical**: hosting your own website as a Tor hidden service (an `.onion` address).

---

## 2. Foundation Concept: What is a URL?

Before understanding the internet's "layers," it's essential to understand the **URL (Uniform Resource Locator)**.

> A URL is a **unique address** assigned to every file, folder, video, or resource stored on the internet, allowing it to be uniquely identified and accessed. It's also referred to as a **Web Resource Unique Identifier**, or more formally, a **URI (Uniform Resource Identifier)**.

### 2.1 URL Syntax Breakdown

```
http(s)://domain-name:port/path/file?parameter=value

Example:
http://cs:81/AUTH/login.aspx?UID=129
```

| Component           | Example           | Description                                                                |
| ------------------- | ----------------- | -------------------------------------------------------------------------- |
| **Protocol**        | `http`, `https`   | The communication protocol used                                            |
| **Domain name**     | `cspathshala.com` | The website's address                                                      |
| **Port (optional)** | `80`, `81`, `443` | The specific port the site runs on; defaults to a standard port if omitted |
| **Path**            | `/AUTH/`          | Folder structure on the server                                             |
| **File**            | `login.aspx`      | The specific file/resource being accessed                                  |
| **Parameter(s)**    | `?UID=129`        | Data being passed along with the request                                   |

> Not every URL contains all of these elements, but this is the general structure.

### 2.2 The Golden Rule of the Internet

> **To access anything on the internet, you must have its exact, genuine URL. Without the URL, that resource cannot be accessed — no matter how much data exists on a server.**

This single rule is the foundation for understanding why the internet is divided into different "layers" — it's really about **which URLs are discoverable** and by whom.

---

## 3. The Surface Web

**Definition:** The portion of the internet that is **indexed by search engines** and easily accessible to anyone via a simple search.

- Search engines (Google, DuckDuckGo, Yahoo, etc.) can only show you data/URLs that they have **indexed**.
- Since you can't memorize every URL on the internet, search engines act as the "directory" that helps you find the URL for what you're looking for.
- **Estimated size:** Despite the vast amount of data on the internet, only **around 5%** of it is indexed and accessible this way — this 5% is the **Surface Web**.

**Examples:** Google, Yahoo, Facebook, and any regular website you can find via a normal search.

---

## 4. The Deep Web

**Definition:** All internet data that search engines **cannot** or **do not** display in their results — regardless of the reason.

> If a search engine can't show a particular URL/data in its results, that data falls under the **Deep Web** — this accounts for the remaining **~95%** of internet data.

**Examples of Deep Web content (completely mundane, not inherently illegal or sinister):**

- A private course portal (e.g., class recordings only accessible to enrolled students) — searching for it on Google won't reveal it, even by name.
- Your private messages, personal contact details, or private database entries.
- Any content deliberately or technically excluded from search engine indexing.

> 💡 **Key clarification:** The Deep Web is **not** inherently sinister — it includes ordinary things like private cloud storage, internal company portals, academic databases, and online banking pages. It's simply "not searchable," not "illegal."

---

## 5. The Dark Web

**Definition:** A **subset of the Deep Web** — specifically, the portion of unindexed data where **illegal activities** are known to take place.

> The Dark Web is **not a specific physical location or a distinct "part" of the internet infrastructure** — it's a **category/classification** applied to certain unindexed websites based on the nature of the activity happening on them.

- All Dark Web content is technically part of the Deep Web (since it's unindexed), but **not all Deep Web content is Dark Web** (most of it is perfectly ordinary/private, not illegal).
- Dark Web sites are typically accessed through special anonymity-focused browsers (e.g., **Tor Browser**, also called an "onion browser").

---

## 6. The Hidden Web

**Definition:** Content within the Deep Web whose **URL/address is known only to a very small, specific group of people** — sometimes just the creator(s).

**Illustrative example:**

> Three friends build a private website together. Only these three people know its URL. No search engine indexes it, and no outsider has ever discovered its address. This falls squarely into the **Hidden Web** category — because it's not merely "unindexed" (Deep Web) or "illegal" (Dark Web); its very existence and address are known to almost no one.

**Key distinction between Hidden Web and Dark Web:**

- **Hidden Web** = defined by **obscurity** (who knows the address)
- **Dark Web** = defined by **illegal activity** occurring on an unindexed site

A site can be Hidden without being Dark (e.g., a private family photo-sharing site only 3 people know about), and a site can become Dark without necessarily starting out as intentionally "hidden."

---

## 7. How a Website Can Move Between Categories

Categories are **not fixed** — the same website can shift between Surface, Deep, Hidden, and Dark depending on circumstances:

| Scenario                                                                                                                   | Resulting Category                                  |
| -------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| A hidden site's URL is shared on a public blog, and the content is legitimate, and Google indexes it                       | **Surface Web** (now indexed and searchable)        |
| That same site is later found to be selling illegal goods (e.g., weapons/drugs), and Google removes/bans it from its index | **Dark Web** (illegal activity + no longer indexed) |
| A private site's URL is known only to its creator(s)                                                                       | **Hidden Web**                                      |
| A private site's data simply isn't indexed by search engines, for any (non-illegal) reason                                 | **Deep Web**                                        |

This illustrates that these are **descriptive categories based on visibility and legality**, not fixed technical zones of the internet.

---

## 8. How the Dark Web Keeps Users Anonymous

Several layered factors make Dark Web users difficult to track:

1. **Special browser required:** Dark Web (`.onion`) sites can only be accessed through a **Tor Browser** (also called an "onion browser") — regular browsers cannot resolve `.onion` addresses.
2. **Unmemorable URLs:** `.onion` addresses are long, randomly-generated strings (ending in `.onion`) — practically impossible to memorize, unlike a normal domain name.
3. **Dynamic/changing addresses:** Onion addresses can be changed by their owner at will. Even if you find and save a working URL, it may stop working entirely by the next day if the owner rotates it.
4. **Multiple layers of proxy relay (the Tor "circuit"):** When you connect to a Tor hidden service, your traffic is routed through **multiple relay nodes** (commonly three), each of which only knows the previous and next hop in the chain — not the full path. This makes it extremely difficult to trace a connection back to the actual server or the actual user.

---

## 9. Is the Dark Web Completely Untraceable?

**No system is perfectly untraceable.** Even with the above protections, a common practical method for identifying a Dark Web site's operator is through **web application penetration testing techniques** — for example:

- Any data a user submits to a website (login forms, search fields, uploaded files, etc.) is ultimately **processed and stored on that site's backend server**, regardless of how many proxy layers protect the network connection.
- If a **vulnerability** exists in the website's code (e.g., an injection flaw), a tester/investigator could potentially exploit it to gain direct information about or access to the actual server — effectively bypassing the network-level anonymity protections, since the flaw exists at the **application layer**, not the network layer.

> 💡 **Key takeaway:** Tor's anonymity protects the **network path**, not necessarily the **application security** of the hosted website itself. A poorly coded hidden service can still be compromised through standard web vulnerabilities.

---

## 10. Practical: Hosting Your Own Tor Hidden Service (.onion site)

This walkthrough demonstrates how easy it technically is to host a basic website as a Tor hidden service — illustrating the concepts above hands-on.

> ⚠️ **Environment used:** A Kali Linux virtual machine, purely for local educational demonstration.

### 10.1 Step 1: Install Tor

```bash
sudo apt install tor
```

- `tor` is the tool that manages the Tor proxy/circuit and enables hosting a hidden service.

### 10.2 Step 2: Host a Basic Website Locally with Apache

While Tor installs, set up a simple local website to serve as the demo content:

```bash
sudo service apache2 start
```

- This starts the **Apache** web server, which by default serves HTML content on **port 80**.
- Verify it's working by visiting in a browser:
  ```
  http://localhost:80
  ```
  or
  ```
  http://127.0.0.1:80
  ```
- At this point, the site is only accessible **locally** — it has no `.onion` address yet.

### 10.3 Step 3: Run Tor and Build the Circuit

```bash
tor
```

- Running this command causes Tor to build its **circuit** (the chain of relay nodes used for anonymous routing).
- The first run can take some time to reach 100%. If it appears stuck for a long time (e.g., stuck at 30%), stop it with **Ctrl+C** and re-run `tor` — it typically resumes and completes.
- Once the circuit reaches 100%, stop the process (**Ctrl+C**) — you now need to edit Tor's configuration before it can serve your hidden service.

### 10.4 Step 4: Configure the Hidden Service in torrc

1. Navigate to Tor's configuration directory:
   ```bash
   cd /etc/tor
   ls
   ```
2. Open the configuration file (requires root permissions) using any text editor:
   ```bash
   gedit torrc
   # or
   mousepad torrc
   ```
3. Search within the file for the keyword **"hidden"** (e.g., using Ctrl+F in the editor).
4. Locate the two relevant configuration lines (commonly around lines 71–72 in a default file):

   ```
   HiddenServiceDir /var/lib/tor/hidden_service/
   HiddenServicePort 80 127.0.0.1:80
   ```

   - **`HiddenServiceDir`** — the directory where all hidden service data (including your `.onion` hostname) will be stored/managed.
   - **`HiddenServicePort`** — maps the public-facing hidden service port (`80`) to your local service (`127.0.0.1:80`, where Apache is already running).

5. **Uncomment** both lines (remove the leading `#` if present) to activate them.
6. Save the file (**Ctrl+S**) and close the editor.

> That's the entire configuration needed — no other changes are required.

### 10.5 Step 5: Find Your .onion Address

1. Navigate to the hidden service directory specified above:
   ```bash
   cd /var/lib/tor/hidden_service/
   ls
   ```
   You should see files/folders related to the hidden service, including one named **`hostname`**.
2. View the generated onion address:
   ```bash
   cat hostname
   ```
   This will output your unique `.onion` address (something like `abcdefghijklmnop.onion`).

### 10.6 Step 6: Access It via Tor Browser

1. Re-run `tor` (if not already running) to re-establish the circuit and make the hidden service live.
2. Open **Tor Browser**:
   - Launch it from its installed location (e.g., desktop icon).
   - Click **Connect** to let it establish its own connection to the Tor network.
3. Once connected, paste your `.onion` address into the address bar and press Enter.
4. You should see the **same Apache default page** you saw locally on `localhost:80` — but now it's being served as a genuine Tor hidden service.

**Important operational note:**

> The hidden service is only reachable **as long as the `tor` process keeps running** on the hosting machine. Stopping it (**Ctrl+C**) immediately takes the site offline — the Tor Browser will fail to load the page.

---

## 11. What's Coming Next

A follow-up video/lesson will cover **anonymity techniques** in more depth — including how to hide your **IP address, MAC address, and general user/region information**, to make it harder for anyone to trace activity back to you.

---

## 12. Summary / Quick Revision Table

| Concept                         | One-Line Summary                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **URL / URI**                   | The unique address required to access any specific resource on the internet                                                                 |
| **Golden Rule of the Internet** | Without the exact URL, a resource cannot be accessed, no matter how much data exists                                                        |
| **Surface Web**                 | The ~5% of internet data indexed and searchable by search engines                                                                           |
| **Deep Web**                    | The ~95% of internet data NOT indexed/shown by search engines, for any reason (not inherently illegal)                                      |
| **Dark Web**                    | A subset of the Deep Web specifically associated with illegal activity on unindexed sites                                                   |
| **Hidden Web**                  | Content whose URL is known only to a small, specific group of people (defined by obscurity, not legality)                                   |
| **Categories are fluid**        | The same website can move between Surface/Deep/Dark/Hidden depending on indexing status and content legality                                |
| **Dark Web anonymity factors**  | Special browser (Tor) required, unmemorable `.onion` URLs, dynamic/changeable addresses, multi-hop relay circuits                           |
| **Is it fully untraceable?**    | No — application-layer vulnerabilities in the hosted website can still expose the server/operator, even with network-level anonymity intact |
| **Tor**                         | The tool/network used to anonymize connections and host/access `.onion` hidden services                                                     |
| **Apache**                      | Used in the demo to serve a basic local website on port 80 before exposing it via Tor                                                       |
| **`torrc`**                     | Tor's main configuration file; `HiddenServiceDir` and `HiddenServicePort` are the two key settings for hosting a hidden service             |
| **`hostname` file**             | Located in the hidden service directory; contains your generated `.onion` address                                                           |
| **Hidden service uptime**       | Only accessible while the `tor` process is actively running on the host machine                                                             |

---

_End of Notes — Layers of the Internet & Hosting a Tor Hidden Service_
