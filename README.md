# pfSense-Firewall-Traffic-Filtering-Lab

## Overview
Simulated attacker traffic from Kali Linux → Windows 11 victim, then created a pfSense firewall rule to block the attacker and verified using Wireshark + Nmap.

**Tools:** pfSense, Windows 11, Kali Linux, VMware, Wireshark, Nmap  
**Skills:** firewall rule creation, packet capture, threat simulation, network security  
**MITRE:** T1046 (Network Scanning)

<img width="1166" height="656" alt="pfsense installed" src="https://github.com/user-attachments/assets/a1783546-d653-4627-bdd1-1b18013ed464" />
<img width="2358" height="1039" alt="pfsense installe 2" src="https://github.com/user-attachments/assets/a4b0c770-c500-4dce-b54c-97fc34a464d4" />

---

## 1. Lab Setup
- pfSense VM with:
  - WAN: NAT
  - LAN: VMnet2 (Host-only) → 192.168.30.1/24
- Windows 11 (victim): 192.168.30.50
- Kali Linux (attacker): 192.168.30.51

---

## 2. Baseline Attack (Before Firewall Rule)
1. Started Wireshark on Windows (Ethernet1).
2. Display filter:
   ```
   ip.addr == 192.168.30.51
   ```
3. Kali ran:
   ```
   nmap 192.168.30.50
   ```
4. Windows Wireshark showed SYN packets from Kali → Windows (attack visible).
<img width="2307" height="1298" alt="ran nmap scan on linux" src="https://github.com/user-attachments/assets/d8a6103f-28f6-4ce5-b5b3-4604b6f01e17" />
<img width="2359" height="1282" alt="SYN SCAN WIN " src="https://github.com/user-attachments/assets/f8bc5c43-6350-4913-9f18-8467620f8c89" />

---

## 3. Create Firewall Rule (Block Kali)
pfSense → **Firewall → Rules → LAN → Add**

- **Action:** Block  
- **Interface:** LAN  
- **Protocol:** Any  
- **Source:** Address → `192.168.30.51 /32`  
- **Destination:** Any  
- **Log packets:** Enabled  
- Save → Apply
<img width="2282" height="1024" alt="configured firewall rules" src="https://github.com/user-attachments/assets/1932bd05-6660-48c5-9b11-1cba04d59fc5" />

---

## 4. Verify Block (After Rule)
1. Kali re-ran:
   ```
   nmap 192.168.30.50
   ```
2. Result:
   - All ports filtered/ignored  
   - No SYN packets reached Windows  
3. Wireshark showed **no traffic** from Kali’s IP (block successful).

---

## 5. Key Outcomes
- Successfully blocked attacker traffic at the firewall.
- Verified mitigation through packet capture + Nmap behavior.
- Demonstrated SOC-style detection → containment workflow.



