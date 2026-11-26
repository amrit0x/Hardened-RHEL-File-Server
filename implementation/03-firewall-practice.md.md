# Practical Challenge: Secure RHEL Server Using `firewalld`

| **Format** | **Total Marks** | **Difficulty** | **Focus Area** |
| :--- | :--- | :--- | :--- |
| Hands-on Lab | **25 Marks** | Intermediate | Linux Administration & Security |

---

## 📌 Scenario

You are working as a Linux System Administrator. Your organization operates a RHEL server hosting critical resources alongside an authorized RHEL client workstation.

Your objective is to harden the server's firewall configuration so that **only the designated authorized client** can access **NFSv4** and **SSH** services.

---

## 🌐 Lab Environment

| Role | Hostname / System | IP Address | Subnet Mask |
| :--- | :--- | :--- | :--- |
| **Server** | RHEL Server | `192.168.184.137` | `/24` |
| **Authorized Client** | RHEL Client | `192.168.184.139` | `/24` |
| **Untrusted Host** | Unauthorized Client | *Any other IP* | — |

---

## 🎯 Requirements

* [ ] **NFSv4 Access:** Accessible **strictly** from the authorized client (`192.168.184.139`).
* [ ] **SSH Access:** Accessible **strictly** from the authorized client (`192.168.184.139`).
* [ ] **Least Privilege:** Unnecessary, broad service allowances (e.g., standard zone-wide service permits) must be removed.
* [ ] **Persistence:** All firewall configurations must persist across reboots and firewall reloads.
* [ ] **Availability:** Existing operational services and active administrator sessions must not be disrupted.

---

## 📋 Lab Tasks

### Task 1: Firewall Auditing *(4 Marks)*

Perform a thorough pre-configuration audit on the **RHEL Server** prior to executing any modifications.

1. Verify whether the `firewalld` systemd daemon is active and running.
2. Identify the currently active firewall zone(s) and assigned interfaces.
3. Inspect the active zone's complete running configuration.
4. Check for any pre-existing rich rules.
5. Determine whether `nfs` and `ssh` services are currently exposed globally across the zone.
6. Validate the integrity and syntax of the existing firewall configuration.

> **Expected Outcome:** Comprehensive understanding of runtime and permanent firewall states before modification.

---

### Task 2: Network Verification *(2 Marks)*

On **both** RHEL systems (Server & Authorized Client):

1. Inspect and document their respective IPv4 network configurations.
2. Confirm that both systems match the addresses outlined in the lab scenario.
3. Identify the specific network interface name handling traffic on the RHEL Server.

---

### Task 3: Configure Firewall Rules *(6 Marks)*

On the **RHEL Server**:

1. Add a rich rule permitting **TCP port 2049** (NFSv4) exclusively for source address `192.168.184.139`.
2. Add a rich rule permitting **TCP port 22** (SSH) exclusively for source address `192.168.184.139`.
3. Configure both rules with permanent persistence.

> ⚠️ **Condition:** Do not permit access to these destination ports to any other IP address or subnet.

---

### Task 4: Remove Broad Access *(4 Marks)*

Your Task 1 audit confirmed that default broad service rules for NFS and SSH are active.

1. Verify whether default `nfs` or `rpc-bind` services are permanently enabled in the active zone.
2. Remove broad `nfs` service allowances from the permanent configuration.
3. Initiate and verify a new secondary SSH connection from the authorized client (`192.168.184.139`).
4. Once client access is proven and out-of-band/terminal access is assured, remove the broad `ssh` service from the active zone.

> 🚨 **Critical Precaution:** **Never** terminate your active administration session before confirming that client SSH authentication succeeds under the new policy.

---

### Task 5: Apply and Verify Configuration *(3 Marks)*

1. Check configuration validity to prevent syntax or rule conflicts.
2. Reload `firewalld` to promote permanent settings to the runtime state.
3. Verify active runtime rich rules.
4. Verify permanent configuration files on disk.
5. Review the complete output of the active zone to confirm overall compliance.

> **Expected Outcome:** Both runtime and permanent states accurately reflect the principle of least privilege with matching rich rule sets.

---

### Task 6: Connectivity Testing *(4 Marks)*

Validate the implementation by running connectivity tests across both client profiles:

#### A. From the Authorized Client (`192.168.184.139`)
1. Mount the server's NFSv4 export to a local mountpoint.
2. Validate the active mount using `findmnt` (confirm `nfs4` protocol).
3. Read/write to the mounted share directory.
4. Establish an interactive SSH session to `192.168.184.137`.

#### B. From the Unauthorized Client (`Any other IP`)
1. Attempt a TCP connection to port `2049` (e.g., via `nc`, `curl`, or `nmap`).
2. Attempt a TCP connection to port `22`.

> **Expected Outcome:** Connections succeed seamlessly for the authorized client, while connection attempts from unauthorized hosts are dropped or rejected.

---

## 💡 Bonus: Troubleshooting & Conceptual Review

*Answer these questions conceptually without running commands:*

* **Q1:** You applied a firewall rule omitting the `--permanent` switch. Following a `firewalld` restart or system reboot, the rule vanished. Why did this occur?
* **Q2:** You successfully defined an IP-restricted SSH rich rule, yet unauthorized IP addresses can still establish SSH sessions. What configuration oversight in the zone could explain this?
* **Q3:** You modified your permanent firewall configuration via `firewall-cmd --permanent ...` but did not execute `firewall-cmd --reload`. Which configuration is currently being evaluated for incoming packets?
* **Q4:** You removed the `nfs` service from the active zone, yet outside clients can still query port 2049. What other directives or rule types inside `firewalld` might still be permitting that port?
* **Q5:** What is the technical networking distinction between specifying an IP as `192.168.184.139` vs `192.168.184.139/32` within a firewall rich rule?