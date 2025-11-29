# 🔥 pfSense Home Lab – Full Firewall Setup, DNS Fix, and Windows vs Linux Access Control

This project documents my complete pfSense home lab inside VMware Workstation.  
I built a full internal network behind pfSense, configured firewall rules, fixed a DNS issue that took nearly 3 hours, and created segmentation where **Windows 11 can browse the internet but Kali Linux cannot**.
---

# 🧱 1. Lab Environment

**Hypervisor:** VMware Workstation  
**Firewall:** pfSense Community Edition  

**Virtual Machines**
- pfSense Firewall  
- Windows 11 (allowed to browse)  
- Kali Linux (blocked from browsing)  

**Network Layout**

| Device        | IP Address      | Purpose |
|---------------|------------------|---------|
| pfSense LAN   | 192.168.30.1     | Gateway + DNS |
| Windows 11    | 192.168.30.10    | Allowed to browse |
| Kali Linux    | 192.168.30.51    | Blocked from browsing |
| LAN Network   | 192.168.30.0/24  | Internal subnet |

All devices are on **VMnet2**, behind pfSense.

---

# 🛠 2. Installing pfSense (Fresh Setup)

### Step 1 — Create pfSense VM
- 2 vCPUs  
- 2GB RAM  
- **NIC1 → WAN** (NAT or Bridged)  
- **NIC2 → LAN** (Custom VMnet2)

### Step 2 — pfSense Console Configuration
Assign interfaces:

- WAN → NIC1  
- LAN → NIC2  

Set LAN IP:
- **192.168.30.1**  
- **255.255.255.0 (/24)**
<img width="1536" height="984" alt="pfsense" src="https://github.com/user-attachments/assets/6ab019b2-1d28-4e0f-8f92-e44dfc3052b4" />


### Step 3 — Access pfSense GUI
On Windows enter:

`https://192.168.30.1`

Login:  
`admin / pfsense`
<img width="2344" height="1276" alt="dashboard" src="https://github.com/user-attachments/assets/9396c80a-fbdf-470b-84e7-80d3cfa555ff" />

Run the setup wizard.

---

# 🌐 3. Connect Windows & Kali to pfSense

### Windows 11 Network Settings
- Adapter: VMnet2  
- IP: 192.168.30.10  
- Gateway: 192.168.30.1  
- DNS: 192.168.30.1  

### Kali Linux Network Settings
- Adapter: VMnet2  
- IP: 192.168.30.51  
- Gateway: 192.168.30.1  
- DNS: 192.168.30.1  

Both route through pfSense.

---

# 💥 4. The 3-Hour Problem — No DNS, No Browsing

At one point, **neither Windows nor Kali could load websites**.

They could:
- ✔ Ping IPs like `8.8.8.8`  

But:
- ❌ Could not ping domains  
- ❌ Browsers failed  
- ❌ DNS lookups failed  

I tried:
- Firewall rules  
- NAT  
- IP reconfiguration  
- Resetting pfSense  
- Googling, Reddit, YouTube  
- Even AI tools  

**The issue was ONE check box:**

> **DNS Resolver was turned OFF.**

### Fix:
**Services → DNS Resolver → Enable → Save**

Instant fix:
✔ DNS working  
✔ Browsers working  
✔ Full internet restored  

---

# 🧩 5. Create Browsing Port Alias

**Firewall → Aliases → Ports → Add**

**Alias Name:** `Browsing`  
**Ports:**  
- 80 (HTTP)  
- 443 (HTTPS)  
- 8080 (Alternative HTTP)  
- 53 (DNS – required for name resolution)

This groups browsing ports into one object.
<img width="2324" height="1220" alt="Browsing Aliases" src="https://github.com/user-attachments/assets/368ee9ea-ad8f-4748-910b-79487cae6a1d" />

---

# 🚦 6. Allow ONLY Windows 11 to Browse the Internet

**Firewall → Rules → LAN → Add**

### Rule:
- Action: **Pass**  
- Source: **192.168.30.10**  
- Destination: **Any**  
- Protocol: **TCP/UDP**  
- Ports: **Browsing alias**  
- Description: Allow Windows 11 Browsing  
<img width="2336" height="1274" alt="rule only allowing win11 to google" src="https://github.com/user-attachments/assets/a0570e35-2552-476c-810d-f7021a0b588e" />

### Result:
✔ Windows loads websites  
✔ Windows resolves DNS  
✔ Windows only device allowed to browse  

---

# 🔴 7. Block Kali Linux From Browsing (Automatically)

Because only Windows gets the Browsing rule, Kali is blocked by default.

### Kali Behavior:
✔ Can ping IPs  
❌ Cannot resolve DNS  
❌ Cannot load websites  
❌ Ports 53, 80, 443 blocked  

This creates segmentation without VLANs.
<img width="2553" height="1354" alt="unable to search on linux " src="https://github.com/user-attachments/assets/1d393e5e-8d17-478b-b4a3-6f6ad8bc2c61" />
