# Networking Fundamental
IT255.002 - Networking Fundamentals

---

# **Basic Cisco Switch Configuration — Field Notes**

### **Scenario:**

You are configuring a new switch (Switch0 → renamed *OgdenSwitch*) in the Ogden Office.
Network block: `10.1.17.144/28`
Router gateway: `10.1.17.145`
Switch management IP: `10.1.17.146`

---

## 🧩 **Objective**

To perform initial configuration on a Cisco switch so it:

* Has a unique hostname and secure access.
* Is protected by passwords.
* Displays a login warning.
* Can be managed remotely (via Telnet).
* Retains its settings after reboot.

---

## ⚙️ **Step-by-Step Procedure**

### **1️⃣ Console Access**

Connect physically using a **console cable**:

* PC → RS232 port
* Switch → Console port

From the PC:

> Desktop → Terminal → Default settings (9600 bps, 8N1, None) → OK

You’ll see the switch CLI prompt:

```
Switch>
```

---

### **2️⃣ Enter Privileged EXEC Mode**

```bash
enable
```

Prompt changes to:

```
Switch#
```

Now you can view system information or enter configuration mode.

---

### **3️⃣ Enter Global Configuration Mode**

```bash
configure terminal
```

Prompt becomes:

```
Switch(config)#
```

This is where all core configurations are done.

---

### **4️⃣ Change the Hostname**

```bash
hostname OgdenSwitch
```

Purpose:

* Identifies this switch uniquely in the network.
* Appears in the prompt, confirming change:

  ```
  OgdenSwitch(config)#
  ```

---

### **5️⃣ Set Privileged Mode (Enable Secret) Password**

```bash
enable secret cisco
```

Purpose:

* Secures privileged EXEC mode access (`enable`).
* The password is stored **encrypted** in the configuration file.

---

### **6️⃣ Secure the Console Port**

```bash
line console 0
password cisco
login
exit
```

Purpose:

* Requires a password before local access.
* “login” activates password checking.
* “exit” returns to global configuration mode.

---

### **7️⃣ Secure VTY (Telnet) Lines**

#### For lines 0–4 (active remote sessions)

```bash
line vty 0 4
password cisco
login
exit
```

#### For lines 5–15 (disable extra access)

```bash
line vty 5 15
login
exit
```

Purpose:

* Limits remote Telnet access to five concurrent sessions.
* Protects unused virtual lines.

---

### **8️⃣ Set the Message of the Day (MOTD) Banner**

```bash
banner motd #
********************************************
Unauthorized access is strictly prohibited.
This device belongs to the Ogden Office Network.
All activity is monitored and logged.
********************************************
#
```

Purpose:

* Legal and administrative warning for anyone accessing the device.
* Displays before any login prompt.

---

### **9️⃣ Configure Management IP on VLAN 1**

```bash
interface vlan 1
ip address 10.1.17.146 255.255.255.240
no shutdown
exit
```

Purpose:

* Enables remote management through the network.
* VLAN 1 acts as the default management interface.

---

### **🔟 Set the Default Gateway**

```bash
ip default-gateway 10.1.17.145
```

Purpose:

* Allows the switch to communicate with devices outside its subnet (e.g., routers or remote offices).

---

### **1️⃣1️⃣ Verify Configuration**

```bash
show ip interface brief
```

Confirm that:

```
Vlan1 10.1.17.146 YES manual up up
```

If it says “administratively down,” repeat `no shutdown`.

---

### **1️⃣2️⃣ Save Configuration**

To keep changes after reboot:

```bash
copy running-config startup-config
```

or shorthand:

```bash
copy run start
```

Expected output:

```
Building configuration...
[OK]
```

---

### **1️⃣3️⃣ Test Remote Connectivity**

From **PC1x4**:

* IP Address: `10.1.17.147`
* Subnet Mask: `255.255.255.240`
* Default Gateway: `10.1.17.145`

Open Command Prompt:

```bash
ping 10.1.17.146
```

Then test Telnet access:

```bash
telnet 10.1.17.146
```

You should see:

```
********************************************
Unauthorized access is strictly prohibited.
********************************************
Password:
```

Login with `cisco` → `enable` → enter secret password → `OgdenSwitch#`

---

## ✅ **Verification Commands**

| Command                   | Description                       |
| ------------------------- | --------------------------------- |
| `show running-config`     | View all current settings         |
| `show ip interface brief` | View interfaces and IPs           |
| `show users`              | See who’s connected (console/VTY) |
| `show startup-config`     | Check saved configuration         |
| `show version`            | Displays IOS version and uptime   |

---

## 🧾 **Final Configuration Snapshot**

```bash
hostname OgdenSwitch
enable secret cisco

line console 0
 password cisco
 login

line vty 0 4
 password cisco
 login

line vty 5 15
 login

banner motd #
********************************************
Unauthorized access is strictly prohibited.
This device belongs to the Ogden Office Network.
All activity is monitored and logged.
********************************************
#

interface vlan 1
 ip address 10.1.17.146 255.255.255.240
 no shutdown

ip default-gateway 10.1.17.145
```

---

## 🧠 **Key Takeaways**

* Always secure console, VTY, and privileged modes with passwords.
* Always use `enable secret` (encrypted), never `enable password` (plain text).
* Always add a legal login banner (mandatory in production).
* VLAN 1 is default for switch management; give it a static IP.
* Save (`copy run start`) after every major change.

---

# **Project 2 — Configuring DHCP & DNS (SLC Administration Office) Note**

**Network:** `10.1.17.112/28` (mask `255.255.255.240`)
**Goal:** Configure the router as a DHCP server and the server as DNS so clients get IP, gateway and DNS automatically and can resolve hostnames.

---

## Device addressing (given)

* `SLC_DHCP_Router` — `10.1.17.113/28` (gateway / DHCP server)
* `SLC_Switch1` — `10.1.17.114/28` (switch management)
* `SLC_DNS` — `10.1.17.115/28` (DNS server)
* `Printer2` — `10.1.17.116/28` (printer)
* **DHCP pool (available):** `10.1.17.117 — 10.1.17.126`
* **Network ID:** `10.1.17.112`
* **Broadcast:** `10.1.17.127`

---

## Part A — Router: DHCP configuration (CLI)

1. Open router CLI. Enter privileged exec and global config:

```
enable
configure terminal
```

2. Exclude statically assigned addresses:

```
ip dhcp excluded-address 10.1.17.113 10.1.17.116
```

3. Create DHCP pool and set parameters:

```
ip dhcp pool SLC_Admin
 network 10.1.17.112 255.255.255.240
 default-router 10.1.17.113
 dns-server 10.1.17.115
```

4. Exit and save:

```
end
copy running-config startup-config
```

**Why:** `excluded-address` prevents assigning the router/switch/DNS/printer IPs. `default-router` sets gateway for clients. `dns-server` tells clients which DNS to use.

---

## Part B — DNS server: configure DNS records (Packet Tracer Server > Services)

1. Click the `SLC_DNS` server → **Services** tab → **DNS**.
2. Turn **DNS Service** → **On**.
3. Add A records (Name → Address):

   * `SLC_DHCP_Router` → `10.1.17.113`
   * `SLC_Switch1` → `10.1.17.114`
   * `SLC_DNS` → `10.1.17.115`
   * `Printer2` → `10.1.17.116`
4. Verify records list shows the four entries.

**Why:** Clients will query this server to resolve hostnames into IPs. The router hands out the DNS IP via DHCP.

---

## Part C — Client verification (PC2x6 and additional PC)

### On PC2x6

1. Desktop → IP Configuration → Select **DHCP**.
2. Wait for the lease. Confirm the assigned values:

   * IP: `10.1.17.117–10.1.17.126`
   * Mask: `255.255.255.240`
   * Default Gateway: `10.1.17.113`
   * DNS Server: `10.1.17.115`

### Add a second PC

1. Add PC to `SLC_Switch1`, connect with straight-through cable.
2. Desktop → IP Configuration → DHCP. Confirm it gets a unique IP from the pool.

### DNS name tests (Command Prompt)

From any DHCP client:

```
ping SLC_DHCP_Router
ping SLC_Switch1
ping SLC_DNS
ping Printer2
```

Expected: names resolve to correct IPs and replies succeed.

---

## Commands to verify (router)

* Show DHCP pools and usage:

```
show ip dhcp pool
show ip dhcp binding
show running-config | section dhcp
```

* Show interface brief:

```
show ip interface brief
```

---

## Expected results / outputs

* `show ip dhcp pool` shows `SLC_Admin`, network `10.1.17.112/28`, available addresses matching the pool.
* `show ip dhcp binding` shows leased IPs and MAC addresses for clients.
* `ping SLC_DNS` from a client resolves to `10.1.17.115`.
* `ipconfig`/IP Configuration on PC shows gateway `10.1.17.113` and DNS `10.1.17.115`.

---

## Troubleshooting checklist

* Client gets APIPA/No IP: verify router DHCP service and excluded-address not covering pool.
* Client gets IP but DNS blank: ensure `dns-server` set in DHCP pool and DNS server reachable.
* Name resolution fails: from client run `nslookup <name>` to see if DNS server responds. Confirm DNS service is ON and A records exist.
* Router cannot assign addresses: verify interface connecting to switch/router is up and switch port is up.
* Save config: `copy run start` so DHCP settings survive reboot.

---

## Screenshots to capture for submission

1. **Router CLI** showing the DHCP config (capture `show running-config | section dhcp` or the `ip dhcp excluded-address` + `ip dhcp pool` block).
2. **DNS server Services** window showing DNS service **ON** and the 4 A records.
3. **PC2x6 IP Configuration** (DHCP assigned values visible).
4. **New PC IP Configuration** (also showing DHCP assignment).
5. **Command Prompt** on a PC showing successful `ping SLC_DNS` (or `nslookup SLC_DNS`).

Add these screenshots to a Word document with short captions (what the screenshot shows and the timestamp).

---

## Submission & naming

* Save Word doc containing screenshots and short captions.
* File name suggestion: `Lastname_Firstname_Project2_DHCP_DNS.docx`
* Submit before deadline.

---

## Quick reference: full router DHCP CLI sequence

```
enable
configure terminal
ip dhcp excluded-address 10.1.17.113 10.1.17.116
ip dhcp pool SLC_Admin
 network 10.1.17.112 255.255.255.240
 default-router 10.1.17.113
 dns-server 10.1.17.115
exit
copy running-config startup-config
```

---

