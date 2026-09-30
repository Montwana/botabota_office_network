# Botabota Office Solutions

## IT Support Troubleshooting Incidents

This document records the network troubleshooting incidents identified and resolved during the development of the Botabota Office Solutions network.

The incidents were investigated using a structured IT support approach:

1. Identify the reported problem
2. Gather information and collect evidence
3. Test connectivity
4. Isolate the fault
5. Identify the root cause
6. Apply the appropriate fix
7. Verify the resolution
8. Document the outcome

---

## Incident Overview

| Incident | Category              | Affected Area       | Status   |
| -------- | --------------------- | ------------------- | -------- |
| INC-001  | IP Configuration      | Sales workstation   | Resolved |
| INC-002  | DNS Configuration     | Finance workstation | Resolved |
| INC-005  | VLAN / DHCP / Routing | Sales network       | Resolved |

---

# INC-01 — Workstation Network Connectivity

**Category:** IP Configuration
**Priority:** Medium
**Affected Device:** SALES-PC01
**Status:** Resolved

## Reported Issue

A Sales user reported that their workstation could not access the company network or internal resources.

## Initial Investigation

The workstation was checked to determine whether the issue was related to the physical connection, IP configuration, gateway connectivity, or another network service.

The following troubleshooting sequence was followed:

* Checked the physical network connection.
* Inspected the workstation's IP configuration.
* Tested connectivity to the default gateway.
* Tested connectivity to internal network resources.
* Compared the workstation configuration with a functioning workstation.

## Findings

The workstation had an IP configuration that did not belong to the company's network.

The expected network was:

```text
Network: 192.168.10.0/24
Gateway: 192.168.10.1
```

The workstation was configured on an incorrect subnet.

## Root Cause

The workstation had been manually configured with an incorrect IP configuration instead of receiving its network settings from DHCP.

## Resolution

The workstation was returned to DHCP so that it could automatically obtain the correct:

* IP address
* Subnet mask
* Default gateway
* DNS server

## Verification

After the configuration was corrected, connectivity was tested again.

Successful tests included:

```text
Workstation → Default Gateway
Workstation → Internal Resources
Workstation → DNS
Workstation → Internal Hostname
```

The workstation successfully rejoined the correct network and normal connectivity was restored.

### Key Troubleshooting Skills

* IPv4 configuration
* Subnet identification
* DHCP
* Connectivity testing
* Fault isolation
* Configuration comparison

---

# INC-02 — Internal DNS Resolution Failure

**Category:** DNS Configuration
**Priority:** Medium
**Affected Device:** FINANCE-PC01
**Status:** Resolved

## Reported Issue

A Finance user reported that internal resources could not be accessed using their hostnames.

For example, the user could not successfully access:

```text
printer.nexa.local
```

## Initial Investigation

The workstation's network configuration was inspected first to determine whether the problem was related to general connectivity or specifically to DNS.

The following tests were performed:

```text
ipconfig
ping 192.168.10.1
ping 192.168.10.20
nslookup printer.nexa.local
ping printer.nexa.local
```

## Findings

The workstation had a valid IP address and could reach the network gateway.

The DNS server itself was also reachable.

However, hostname resolution was unsuccessful.

The workstation was configured to use the wrong DNS server:

```text
Incorrect DNS:
192.168.50.25
```

The correct Nexa DNS server was:

```text
192.168.10.20
```

The configuration was compared with a functioning Finance workstation, which helped isolate the difference.

## Root Cause

The workstation had an incorrect DNS server address configured.

Because the workstation was pointing to the wrong DNS server, internal hostnames could not be resolved.

## Resolution

The DNS configuration was corrected to:

```text
DNS Server:
192.168.10.20
```

## Verification

After correcting the DNS configuration, hostname resolution was tested again.

The following test was successful:

```text
nslookup printer.nexa.local
```

The workstation was then able to resolve the printer hostname and access the internal resource.

### Key Troubleshooting Skills

* DNS troubleshooting
* `ipconfig`
* `nslookup`
* Ping testing
* Configuration comparison
* Service isolation
* Root-cause identification

---

# INC-03 — Sales VLAN and DHCP Connectivity

**Category:** VLAN / DHCP / Inter-VLAN Routing
**Priority:** High
**Affected Area:** Sales Department
**Status:** Resolved

## Reported Issue

During the network upgrade, Sales workstations were unable to consistently obtain an IP address from the correct Sales network.

Some workstations received addresses from the wrong network, while another workstation received an APIPA address.

Example APIPA address:

```text
169.254.x.x
```

This indicated that the workstation had not successfully received an address from DHCP.

## Network Design

The upgraded network used department-based VLANs:

| VLAN | Department | Network         | Gateway      |
| ---- | ---------- | --------------- | ------------ |
| 10   | IT         | 192.168.10.0/24 | 192.168.10.1 |
| 20   | Sales      | 192.168.20.0/24 | 192.168.20.1 |
| 30   | Finance    | 192.168.30.0/24 | 192.168.30.1 |
| 40   | Servers    | 192.168.40.0/24 | 192.168.40.1 |

The DHCP server was located in the Servers VLAN:

```text
DHCP Server:
192.168.40.10
```

Because DHCP clients were located on different VLANs, DHCP relay was configured on the router.

## Investigation

The issue was investigated systematically rather than assuming that DHCP itself was the only problem.

### 1. Checked workstation IP configuration

The affected Sales workstation did not receive the expected:

```text
192.168.20.x
```

address range.

### 2. Checked switch VLAN assignments

The Sales workstations were connected to the correct access ports:

```text
Fa0/3 → Sales PC01
Fa0/4 → Sales PC02
```

Both ports were assigned to:

```text
VLAN 20
```

### 3. Checked trunk configuration

The link between SW-ACCESS and SW-CORE was configured as a trunk.

The required VLANs were allowed across the trunk:

```text
10,20,30,40
```

### 4. Checked VLAN availability

VLAN 20 was created on the required switches and was active across the network.

### 5. Checked DHCP configuration

The Sales DHCP pool was configured for:

```text
Network: 192.168.20.0/24
Gateway: 192.168.20.1
DNS: 192.168.40.20
```

### 6. Checked DHCP relay

The router was configured to forward DHCP requests from VLAN 20 to the DHCP server:

```text
ip helper-address 192.168.40.10
```

### 7. Checked inter-VLAN routing

The router-on-a-stick configuration was inspected, including the VLAN 20 subinterface.

The investigation identified an incorrect IP configuration on:

```text
GigabitEthernet0/0.20
```

## Root Cause

The VLAN 20 router subinterface had an incorrect IP address.

This prevented the Sales VLAN from using the correct default gateway and disrupted communication required for DHCP operation.

## Resolution

The VLAN 20 router subinterface was corrected to:

```text
interface gigabitEthernet0/0.20
ip address 192.168.20.1 255.255.255.0
ip helper-address 192.168.40.10
```

The configuration was then saved to the router's startup configuration.

## Verification

The Sales workstation was switched back to DHCP and successfully received an address from the correct network.

Expected configuration:

```text
IP Address: 192.168.20.x
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
DNS Server: 192.168.40.20
```

Connectivity to the Sales gateway was then tested successfully.

The incident was considered resolved after confirming that the affected workstation received the correct network configuration and could communicate with the network.

### Key Troubleshooting Skills

* VLAN troubleshooting
* DHCP troubleshooting
* DHCP relay
* Router-on-a-stick
* Inter-VLAN routing
* Trunk troubleshooting
* IPv4 subnetting
* Cisco IOS configuration

* Systematic fault isolation
---

# Troubleshooting Methodology

The incidents in this project followed a structured troubleshooting methodology rather than relying on trial and error.

```text
User Report
    ↓
Gather Information
    ↓
Check Physical Connectivity
    ↓
Inspect Configuration
    ↓
Test Connectivity
    ↓
Compare With Working Device
    ↓
Isolate the Fault
    ↓
Identify Root Cause
    ↓
Apply Fix
    ↓
Verify
    ↓
Document
```

This approach helped separate problems involving:

* End-user configuration
* IP addressing
* DNS
* DHCP
* VLAN assignment
* Trunking
* Inter-VLAN routing
* Router configuration

---

# Tools and Commands Used

The following tools and commands were used during the troubleshooting process:

### Cisco Packet Tracer

Used to simulate:

* Routers
* Switches
* Workstations
* Servers
* VLANs
* DHCP
* DNS
* Network connectivity

### Windows Network Commands

```text
ipconfig
ping
nslookup
```

### Cisco IOS Commands

```text
show vlan brief
show interfaces trunk
show running-config
show ip interface brief
```

These commands were used to inspect network configuration, verify VLAN and trunk status, and isolate configuration faults.

---

# Final Outcome

All identified incidents were resolved and verified.

The final Botabota Office Solutions network provided:

* Department-based VLAN segmentation
* Inter-VLAN routing
* Centralized DHCP
* DHCP relay
* Internal DNS
* Network printer connectivity
* Structured IP addressing
* Documented troubleshooting procedures

The troubleshooting process also demonstrated the importance of testing each layer of the network systematically before changing configurations.

---

## Skills Demonstrated

**Networking**

* IPv4 addressing
* Subnetting
* VLANs
* Trunking
* Inter-VLAN routing
* Router-on-a-stick

**Network Services**

* DHCP
* DHCP relay
* DNS
* Static addressing

**IT Support**

* Incident investigation
* Fault isolation
* Configuration comparison
* Root-cause analysis
* Service verification
* Technical documentation

**Tools**

* Cisco Packet Tracer
* Cisco IOS
* Windows network utilities

