# 🛡 Personal Security Exploration: Windows Firewall Validation Lab

## 📌 Project Overview

This project documents my hands-on exploration of Windows Firewall security on my personal laptop.

Instead of assuming the firewall works correctly, I decided to:

* Inspect my network configuration
* Verify firewall status
* Enable firewall logging
* Simulate blocked traffic
* Validate enforcement through log analysis
* Understand inbound vs outbound behavior

This was a personal security validation exercise focused on understanding **control, enforcement, detection, and visibility**.

---

# 🌐 1. Network Inspection Phase

## Commands Used

```
ipconfig
route print
```

## What I Observed

* Private IP address (192.168.x.x) → Home network
* Default gateway routes traffic through router
* VPN adapter installed but not connected
* IPv6 default route enabled
* WSL created private NAT network (172.x.x.x range)

## Key Learning

Security begins with visibility. Before testing firewall behavior, I needed to understand how traffic flows through my system.

---

# 🔥 2. Firewall Verification Phase

## Command Used

```
Get-NetFirewallProfile
```

## Findings

* Firewall enabled for Domain, Private, and Public profiles
* Default inbound behavior = Block
* Default outbound behavior = Allow
* IPv6 traffic also protected under firewall

## Key Learning

A firewall is both a prevention and enforcement control. However, enabling it is not enough — it must be validated.

---

# 📓 3. Enabling Firewall Logging

## Created Log Directory

```
C:\Users\Username\Downloads\FirewallLogs
```

## Enabled Logging

```
Set-NetFirewallProfile -Profile Domain,Private,Public -LogBlocked True
```

## Purpose

To record blocked traffic events for security visibility.

## Key Learning

Blocking without logging provides no visibility. Detection is as important as prevention.

---

# 🚫 4. Why We Tested Outbound Traffic

My laptop is behind a home router using NAT:

```
Internet → Router → Laptop
```

The router blocks unsolicited inbound internet traffic before it reaches the laptop.

If I tested inbound from the internet:

* Router would block it first
* Windows Firewall would never see it
* No log would be generated

Therefore, outbound testing was chosen because:

* Outbound traffic originates from the laptop
* Firewall directly controls it
* Easy to simulate and validate
* Guarantees enforcement visibility

---

# 🧪 5. Simulating Blocked Traffic

## Created Outbound Block Rule

```
New-NetFirewallRule -DisplayName "BlockGoogleTest" `
-Direction Outbound `
-RemoteAddress 8.8.8.8 `
-Action Block
```

## Tested Connectivity

```
Test-NetConnection -ComputerName 8.8.8.8 -Port 53
```

Result: Connection failed as expected.

## Validation

Checked firewall log file and observed DROP entries.

After validation, removed test rule:

```
Remove-NetFirewallRule -DisplayName "BlockGoogleTest"
```

---

# 📊 6. Log Analysis

Example entries observed:

```
DROP TCP <internal-ip> <external-ip> <src-port> 53

```

## Protocol Explanation

### TCP (Transmission Control Protocol)

* Reliable connection-based protocol
* Used by HTTPS, SSH, many applications
* Like making a phone call (connection established first)

### UDP (User Datagram Protocol)

* Fast, connectionless protocol
* Used by DNS, streaming, gaming
* Like shouting a message (no confirmation)

### ICMP (Internet Control Message Protocol)

* Used for diagnostics (ping)
* Network health checking

## Why Multiple Protocols Appeared

When testing DNS (port 53), the system attempted:

* UDP DNS request
* Possible TCP fallback
* ICMP connectivity test
* Background name resolution traffic

Firewall blocked each attempt and logged them.

---

# 🏠 7. Inbound vs Outbound Understanding

## Inbound Traffic

Inbound = External device attempting to connect to my laptop.

In a home setup:

* Router blocks most internet inbound traffic
* Laptop firewall only logs inbound if traffic reaches it

## Outbound Traffic

Outbound = Laptop initiating connection to external systems.

Firewall controls outbound traffic directly, which makes it ideal for testing enforcement.

---

# 🧠 Security Control Lifecycle Demonstrated

This lab validated the full security control process:

1. Visibility (Network inspection)
2. Control verification (Firewall profile check)
3. Detection activation (Enable logging)
4. Prevention (Create block rule)
5. Enforcement validation (Test connection)
6. Telemetry analysis (Review DROP logs)
7. Cleanup (Remove test rule)

Security is not assumption.
Security is verification.

---

# 🧒 Explaination Like I’m 10

My laptop is a house.

* The router is the outer gate.
* The firewall is the guard at my door.
* The log file is the guard’s notebook.

I told the guard not to let a specific person talk to anyone.
That person tried calling, shouting, and checking if I was there.
The guard stopped all attempts and wrote each one down.
I opened the notebook and confirmed he did his job.

---

# 🎯 Final Reflection

This personal exploration helped me understand:

* How routing affects security
* Why outbound testing was necessary
* The difference between TCP, UDP, and ICMP
* How firewall logging enables forensic visibility
* Why validation is essential in security

This exercise strengthened my mindset from configuration-focused to validation-focused.

Security controls must be tested, observed, and verified — not assumed.
