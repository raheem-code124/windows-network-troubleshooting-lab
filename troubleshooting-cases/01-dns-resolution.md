# Ticket 001 | DNS Resolution Failure

**Category:** Network Troubleshooting / DNS

**Environment:** Windows 11 Pro — Oracle VirtualBox

**Workstation:** `IT-SUPPORT-PC01`

**Incident Type:** Simulated IT Support Ticket

**Resolution Status:** Resolved and Verified

---

## 1. Incident Summary

### Problem Description

A Windows 11 workstation was intentionally configured with an incorrect IPv4 DNS server to simulate a common desktop support incident involving failed domain-name resolution.

The objective was to investigate the network configuration, determine whether the issue affected general IP connectivity or DNS specifically, identify the root cause, and restore normal DNS functionality.

### Resolution Summary

Identified a nonresponsive, manually configured DNS server (`192.0.2.1`). Testing confirmed that the specified server timed out during DNS queries, while external IPv4 connectivity remained operational.

Restored automatic DNS configuration, cleared the DNS resolver cache, and verified successful domain-name resolution.

## 2. Technical Environment

| Component | Configuration |
|---|---|
| Operating System | Windows 11 Pro |
| Virtualization | Oracle VirtualBox |
| Network Adapter | Ethernet — NAT |
| IPv4 Address | `10.0.2.15` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `10.0.2.2` |
| Working DNS Server | `10.0.0.1` |
| Simulated Incorrect DNS | `192.0.2.1` |

The workstation initially had a functioning network configuration. The DNS misconfiguration was introduced deliberately to reproduce and investigate the problem.

## 3. Problem Reproduction

Opened the network adapter configuration using `ncpa.cpl` and navigated to:

**Ethernet → Properties → Internet Protocol Version 4 (TCP/IPv4)**

Changed the preferred IPv4 DNS server from automatic configuration to:

`192.0.2.1`

This address belongs to a range reserved for documentation and testing. In this lab environment, it did not provide DNS responses.

The IPv4 address and default gateway were left unchanged to isolate the DNS configuration from general IP addressing.

### Evidence — Incorrect DNS Configuration

![Manually configured incorrect DNS server](https://github.com/raheem-code124/windows-network-troubleshooting-lab/blob/main/screenshots/09a-dns-configuration.png)

## 4. Diagnostic Investigation

### Test 1 — Inspect Network Configuration

**Command:** `ipconfig /all`

**Purpose:** Inspect the network adapter's IPv4 address, gateway, and configured DNS servers.

**Observed results:**

| Parameter | Value |
|---|---|
| IPv4 Address | `10.0.2.15` |
| Default Gateway | `10.0.2.2` |
| Manually Configured IPv4 DNS | `192.0.2.1` |

**Analysis:**

The workstation retained its expected IPv4 address and default gateway. However, the IPv4 DNS server had been changed to the test address.

This made the DNS configuration the primary area for further investigation.

### Test 2 — Query the Suspected DNS Server

**Command:** `nslookup google.com 192.0.2.1`

**Purpose:** Send a DNS lookup directly to the configured IPv4 DNS server and determine whether it responds.

**Observed result:** The DNS query timed out.

**Analysis:**

The specified DNS server did not respond to the lookup. This supported the hypothesis that the manually configured DNS server was unsuitable for the workstation.

The explicit server argument in this command is significant: it tests `192.0.2.1` directly, regardless of other DNS servers that Windows might have available.

### Test 3 — Verify External IPv4 Connectivity

**Command:** `ping 1.1.1.1`

**Purpose:** Test external IPv4 connectivity without relying on domain-name resolution.

**Observed results:**

| Metric | Result |
|---|---|
| Packets Sent | 4 |
| Packets Received | 4 |
| Packet Loss | 0% |
| Average Round-Trip Time | Approximately 17 ms |

**Analysis:**

All four ICMP echo requests received responses.

This demonstrated that the workstation could still communicate with an external IPv4 address. The evidence pointed toward a DNS-specific configuration problem instead of a complete IPv4 connectivity failure.

### Evidence — Diagnostic Results

![DNS query timeout and successful external IPv4 ping](https://github.com/raheem-code124/windows-network-troubleshooting-lab/blob/main/screenshots/09b-dns-failure-and-ping.png)

## 5. Root Cause Analysis

**Root Cause: Incorrect manual IPv4 DNS server configuration.**

The network adapter was configured to use `192.0.2.1` as its preferred IPv4 DNS server. That address did not respond to the lab's DNS lookup requests.

The diagnostic evidence established two important findings:

- A direct DNS request to the configured server timed out.
- External IPv4 connectivity remained functional.

Together, these results isolated the observed failure to the configured DNS server, without requiring unnecessary changes to the workstation's IP address, subnet mask, or gateway.

**Diagnostic scope:** Because Windows may have access to other DNS servers, including IPv6 DNS servers, the failed direct query does not establish that every possible DNS lookup on the workstation was unavailable. It specifically demonstrates failure of the tested server.

## 6. Corrective Actions

Returned to the Ethernet adapter's IPv4 properties and selected:

**Obtain DNS server address automatically**

This removed the incorrect manually assigned IPv4 DNS server.

Next, opened Command Prompt and executed:

**Command:** `ipconfig /flushdns`

**Purpose:** Clear locally cached DNS resolver entries before retesting.

Then executed:

**Command:** `ipconfig /all`

**Purpose:** Confirm that the network configuration had been restored.

**Observed result:** The IPv4 DNS configuration returned to the working server address, `10.0.0.1`.

### Evidence — Restored Configuration

![Restored automatic DNS configuration](https://github.com/raheem-code124/windows-network-troubleshooting-lab/blob/main/screenshots/10-dns-configuration-restored.png)

## 7. Resolution Verification

Following the corrective action, repeated the relevant diagnostic tests to confirm recovery.

| Verification | Command | Observed Result |
|---|---|---|
| DNS configuration | `ipconfig /all` | Incorrect IPv4 DNS server removed; working configuration restored |
| DNS resolution | `nslookup google.com` | Domain name successfully resolved |
| External IP connectivity | `ping 1.1.1.1` | Successful replies; 0% packet loss |

The successful DNS lookup demonstrated that domain-name resolution was functioning after the configuration change.

The successful ping confirmed that external IPv4 connectivity remained available.

### Evidence — Post-Resolution Testing

![Successful DNS resolution and external connectivity](https://github.com/raheem-code124/windows-network-troubleshooting-lab/blob/main/screenshots/11b-dns-resolution-and-ping.png)

## 8. Technical Lessons Learned

This scenario reinforced the value of separating DNS troubleshooting from general network connectivity testing.

A DNS timeout and a failed ping do not necessarily indicate the same type of problem. Using `nslookup` to test the specified DNS server and `ping` to test an external IP address provided evidence for narrowing the diagnosis.

The corrective action focused on the confirmed misconfiguration. Restoring automatic DNS settings addressed the cause without changing unrelated network parameters.

The final verification was essential: confirming the correct settings alone would not demonstrate successful DNS resolution. A follow-up lookup was needed to verify functionality.

## 9. Incident Closure

| Closure Item | Result |
|---|---|
| Failure reproduced | Yes |
| Diagnostic evidence collected | Yes |
| Root cause identified | Incorrect manual IPv4 DNS configuration |
| Corrective action applied | Automatic DNS configuration restored |
| DNS resolution retested | Successful |
| External IPv4 connectivity retested | Successful |
| Final Status | **Resolved** |

---

[Return to Main Project README](../README.md)
