Practical Challenge: Secure RHEL Server Using firewalld

Scenario
You are working as a Linux System Administrator. Your company has a RHEL server and an authorized RHEL client.

Your job is to configure the server's firewall so that only the authorized client can access NFSv4 and SSH.

Lab environment
System	IP Address
RHEL Server	192.168.184.137
RHEL Client	192.168.184.139
Unauthorized Client	Any other IP

Requirements
- NFSv4 must be accessible only from the authorized client.
- SSH must be accessible only from the authorized client.
- Unnecessary broad service allowances must be removed.
- Firewall rules must persist after reboot.
- Existing working services must not be disrupted.

Task 1: Firewall Auditing (4 Marks)
Perform a pre-configuration audit on the RHEL Server.
1. Check whether firewalld is running.
2. Identify the active firewall zone.
3. Inspect the complete firewall configuration.
4. Check existing rich rules.
5. Identify whether NFS and SSH are broadly allowed.
6. Check whether the firewall configuration is valid.
Expected: You should know the existing firewall configuration before making any changes.
Task 2: Network Verification (2 Marks)
On both RHEL systems:
1. Identify their IPv4 addresses.
2. Verify that the server and client IP addresses match the scenario.
3. Identify the network interface used by the server.
Task 3: Configure Firewall Rules (6 Marks)
On the RHEL Server:
1. Create a rich rule allowing TCP port 2049 only from 192.168.184.139.
2. Create a rich rule allowing TCP port 22 only from 192.168.184.139.
3. Make both rules permanent.
Condition: Do not allow these ports to every IP address.
Task 4: Remove Broad Access (4 Marks)
Your audit reveals that NFS and SSH services are already allowed broadly.
1. Check whether NFS is permanently allowed.
2. Remove broad NFS access if it is enabled and no longer required.
3. Test a new SSH connection from the authorized client.
4. Only after confirming SSH access and having a recovery method, remove broad SSH access if enabled.
Important: Do not close your existing SSH session before verifying the new connection.
Task 5: Apply and Verify Configuration (3 Marks)
1. Validate the firewall configuration.
2. Reload firewalld.
3. Verify runtime rich rules.
4. Verify permanent rich rules.
5. Inspect the complete active-zone configuration.
Expected: Runtime and permanent configurations should reflect your intended policy.
Task 6: Connectivity Testing (4 Marks)
Perform these tests:

On RHEL Client:
1. Connect to the NFSv4 share.
2. Verify the mount using findmnt.
3. Access the mounted directory.
4. Establish an SSH connection to the server.
On Unauthorized Client:
5. Attempt to connect to TCP port 2049.
6. Attempt to connect to TCP port 22.
Expected: Authorized access should work; unauthorized access should fail.

Bonus: Troubleshooting Questions
Extra Practice
Answer these without executing commands:

Q1. You added a firewall rule without --permanent. After restarting firewalld, the rule disappeared. Why?

Q2. You added a source-specific SSH rich rule, but unauthorized clients can still connect. What could be the reason?

Q3. You modified permanent firewall rules but forgot to execute firewall-cmd --reload. Which configuration is currently active?

Q4. You removed the NFS service, but port 2049 is still accessible. What other firewall configurations would you investigate?

Q5. What is the purpose of /32 in 192.168.184.139/32?


Answer(Task):
Task 1:
1. systemctl status firewalld
2. firewall-cmd --get-active-zone
3. firewall-cmd --list-all
4. firewall-cmd --permanent --zone=public --list-rich-rules
5. `firewall-cmd --permanent --zone=public --query-service=nfs`    
`firewall-cmd --permanent --zone=public --query-service=ssh`
7. firewall-cmd --check-config

Task 2:
