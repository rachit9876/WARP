## How to Set Up Cloudflare WARP VPN on Ubuntu (No Zero Trust)

### Tested on: Ubuntu 22.04+ (also works on 20.04)

---

### Step 1: Install the WARP client

```bash
curl -fsSL https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list

sudo apt update
sudo apt install cloudflare-warp
```

---

### Step 2: Fix DNS (most networks break WARP registration)

#### Set Cloudflare DNS (system-wide) to avoid hijacked DNS:

```bash
nmcli connection show  # Find your connection name (e.g., "Wi-Fi" or "KIIT-WIFI-NET.")
```

Then run:

```bash
sudo nmcli connection modify "YOUR_CONNECTION_NAME" ipv4.ignore-auto-dns yes
sudo nmcli connection modify "YOUR_CONNECTION_NAME" ipv4.dns "1.1.1.1 1.0.0.1"
sudo nmcli connection down "YOUR_CONNECTION_NAME" && sudo nmcli connection up "YOUR_CONNECTION_NAME"
```

Confirm DNS is correct:

```bash
resolvectl status
```

Make sure it shows only:

```
DNS Servers: 1.1.1.1 1.0.0.1
```

---

### Step 3: Set WARP mode (optional, but recommended)

```bash
warp-cli set-mode warp
```

This avoids using Zero Trust / Teams.

---

### Step 4: Register the client

```bash
warp-cli registration new
```

> It will open a browser window — log in or click to continue.

---

### Step 5: Connect to WARP VPN

```bash
warp-cli connect
```

Check status:

```bash
warp-cli status
```

---

### Future Usage (Quick Commands)

| Action         | Command                           |
| -------------- | --------------------------------- |
| Connect VPN    | `warp-cli connect`                |
| Disconnect VPN | `warp-cli disconnect`             |
| Status         | `warp-cli status`                 |
| Restart daemon | `sudo systemctl restart warp-svc` |

---

### Optional: Auto-connect on boot (if needed)

```bash
sudo systemctl enable warp-svc
```

---

### Tip: Avoid Zero Trust / Teams

If the app ever says you're being routed through “Zero Trust” or asks for an organization domain, run:

```bash
warp-cli set-mode warp
```

That switches you to regular WARP VPN (no login needed after registration).

---
### 🔧 Fix (If You Have Errors)

1. **Edit the repo file** you created:

```bash
sudo nano /etc/apt/sources.list.d/cloudflare-client.list
```

2. Change this line:

```
deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ plucky main
```

⬇️ Replace `plucky` with `jammy`:

```
deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ jammy main
```

3. Save (`CTRL+O`, `Enter`, `CTRL+X`) and then run:

```bash
sudo apt update
sudo apt install cloudflare-warp -y
```

4. Verify:

```bash
warp-cli --version
```

---

💡 This is a common trick: when a new Ubuntu version comes out, some vendors don’t update repos immediately, but packages from the previous LTS (jammy = 22.04) usually work fine.

---
