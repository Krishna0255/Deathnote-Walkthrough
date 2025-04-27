
# Deathnote: 1 — VulnHub Walkthrough

> Inspired by one of the best anime series — *Death Note*.  
> Highly recommended to watch if you haven't already!

---

## 🖥️ 1. Enumeration

First, ensure both your attacking machine and the Deathnote VM are on the same network (use **Bridged Adapter** mode).

Find the target IP:

```bash
netdiscover -i eth0
```

✅ Discovered the IP address of the Deathnote machine.

---

## 🔍 2. Reconnaissance

Scan the target for open ports, running services, and versions:

```bash
nmap -sC -sV -Pn <IP>
```

**Result:**  
- Port **22** (SSH) — Open
- Port **80** (HTTP) — Open

Visit the web server:

```bash
http://<IP>
```

At first, the page didn't render properly.  
To fix this, I edited `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Added:

```
<IP> deathnote.vuln
```

After updating, the webpage loaded correctly — themed around Kira's page.  
There was a **hint** pointing toward a `notes.txt` file somewhere.

---

### 📂 Gobuster Directory Bruteforce

Used `gobuster` to find hidden paths:

```bash
gobuster dir -u http://deathnote.vuln -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

**Found:**
- `/robots.txt`
- `/important.jpg`

**Hints gathered:**
- A `notes.txt`
- A `user.txt`
- Possible file uploads

---

### 🔎 Nikto & WPScan

**Nikto** and **WPScan** helped discover:

- Browsable directories.
- WordPress-specific uploads in `/wp-content/uploads/2021/07/`.

Downloaded suspicious files using:

```bash
wget http://deathnote.vuln/wp-content/uploads/2021/07/<filename>
```

---

## 🎯 3. Exploitation

After gathering `note.txt` and `user.txt`, I used **Hydra** to perform SSH brute force:

```bash
hydra -L users.txt -P passwords.txt ssh://<IP>
```

✅ Successfully found valid SSH credentials.

---

## 🖥️ 4. Getting a Shell

SSH into the machine:

```bash
ssh l@deathnote.vuln
```

Inside, found:

- `user.txt` in **L's** home directory — **Brainfuck encoded**.
- `/opt/L` directory with:
  - `fake-notebook-rule`
  - `kira-case`

Decoded **user.txt** using [CyberChef](https://gchq.github.io/CyberChef/) — revealed another password.

---

### 🧠 Further Enumeration

In `/home/kira/`, found:

- `kira.txt` — **Base64 encoded**.

Decoded it:

```bash
cat kira.txt | base64 -d
```

Found that **kira** user can run **sudo** without password prompt!

---

## ⚡ 5. Privilege Escalation

Switched to **kira** and used the password found earlier.

Escalated privileges:

```bash
sudo -i
```

Now as **root**, accessed:

```bash
cd /root
cat root.txt
```

🏴 **Root flag captured!**

---

# 🏁 Conclusion

This was a fun box combining:

- Enumeration
- Brute-forcing
- Steganography
- Encoding/Decoding (Brainfuck, Base64)
- WordPress exploitation
- Privilege Escalation

---

# 📚 Skills Sharpened

- Network Scanning and Enumeration
- Web Exploitation
- Password Bruteforce
- Linux Privilege Escalation
- Encoding/Decoding Techniques

---

> "The human whose name is written in this notebook... shall root this machine."

---

