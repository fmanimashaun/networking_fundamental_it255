# Networking Fundamental
IT255.002 - Networking Fundamentals

---

# 🧾 **Basic Cisco Switch Configuration — Field Notes**

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

Would you like me to format this into a **printable version (Word/PDF)** or a **cheat-sheet style (one-pager)** you can keep for future labs or submissions?
