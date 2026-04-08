# macOS SSH Guide — Machine to Machine

A practical, step-by-step guide to SSH from one macOS machine to another using only the built-in CLI. No third-party tools required.

---

## Prerequisites

- Both Macs on the same network (or reachable via IP/hostname)
- macOS 10.12+ (SSH is built in)
- Admin access on the target (host) machine

---

## Step 1 — Find the IP Address of the Host Mac

On the **machine you want to SSH INTO**, run:

```bash
# Try en0 first (usually Wi-Fi)
ipconfig getifaddr en0

# If that returns nothing, try en1 (Ethernet or secondary adapter)
ipconfig getifaddr en1

# Or en2
ipconfig getifaddr en2
```

**Which interface is which?**
- `en0` — usually Wi-Fi
- `en1` — usually Ethernet or Thunderbolt adapter
- `en2` — secondary adapter

You'll get back something like `192.168.1.45` — that's your target IP.

---

## Step 2 — Enable Remote Login on the Host Mac

SSH into a Mac only works if **Remote Login** is turned on.

On the **host machine** (the one you're SSHing INTO):

```bash
# Enable Remote Login via CLI
sudo systemsetup -setremotelogin on

# Verify it's active
sudo systemsetup -getremotelogin
# Expected output: Remote Login: On
```

Or via GUI:
**System Settings → General → Sharing → Remote Login → Toggle ON**

---

## Step 3 — SSH From the Client Mac

On the **machine you're SSHing FROM**:

```bash
ssh username@192.168.1.45
```

Replace:
- `username` — the macOS username on the host machine
- `192.168.1.45` — the IP you found in Step 1

You'll be prompted for the host machine's password. Enter it and you're in.

---

## Step 4 — Set Up SSH Key Authentication (Skip Passwords)

Typing a password every time gets old. Set up key-based auth instead.

**On the client machine (the one you're SSHing FROM):**

```bash
# Generate a key if you don't have one
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy your public key to the host machine
ssh-copy-id username@192.168.1.45
```

Now you can SSH without a password:

```bash
ssh username@192.168.1.45
```

---

## Step 5 — Create a Shortcut with SSH Config

Stop typing the full IP every time. Add an alias in `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

Add this block:

```
Host my-mac
  HostName 192.168.1.45
  User username
  IdentityFile ~/.ssh/id_ed25519
```

Now connect with just:

```bash
ssh my-mac
```

---

## Step 6 — Transfer Files Between Macs (Bonus)

Once SSH is working, use `scp` to copy files:

```bash
# Copy a file FROM the remote mac TO your local machine
scp username@192.168.1.45:/path/to/remote/file.txt ~/Desktop/

# Copy a file FROM local TO the remote mac
scp ~/Desktop/file.txt username@192.168.1.45:/path/to/destination/

# Copy an entire folder
scp -r ~/my-folder username@192.168.1.45:~/Documents/
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `Connection refused` | Remote Login not enabled on host — see Step 2 |
| `No route to host` | Wrong IP, or Macs on different networks |
| `Permission denied` | Wrong username or password |
| `en0` returns nothing | Try `en1` or `en2` — or run `ifconfig` to see all interfaces |
| Still can't connect | Check firewall: System Settings → Network → Firewall |

---

## Author

Ed Camacho | [GitHub](https://github.com/Worldtraveler247) | [LinkedIn](https://linkedin.com/in/worldtraveler247)
