# breakout-vm-penetration-test
A penetration test report detailing root access on the Breakout VM by exploiting a misconfigured Linux Capability on a tar binary.
1. Executive Summary
The target machine was successfully compromised. Initial access was obtained through a web-based shell exposed via Usermin. Full administrative privileges (root) were later achieved by exploiting a misconfigured Linux Capability assigned to the tar binary, specifically cap_dac_read_search, which allowed bypassing standard file permission restrictions.

2. Tools Used
Browser (Firefox/Chrome): Used to access the Usermin Web Shell on port 20000.
Linux Command Line (Bash): Used for system enumeration and exploitation.
Getcap: Used to identify binaries with elevated Linux Capabilities.
Tar: Exploited to bypass Discretionary Access Control (DAC) restrictions.
Cat: Used to read sensitive files and capture final flags.
3. Step-by-Step Exploitation Process
Step 1: Initial Access & Enumeration
After gaining shell access as the cyber user through the web shell, system enumeration began to identify privilege escalation opportunities.

Command: id

Result: Confirmed current user was cyber.

Command: sudo -l
Result: sudo was not installed, eliminating sudo-based privilege escalation.
Command:
/sbin/getcap -r / 2>/dev/null
Discovery:
Found a custom tar binary located at:
/home/cyber/tar

With the capability:

cap_dac_read_search=ep
Technical Analysis:

The cap_dac_read_search capability permits the binary to:

Bypass file read permission checks
Traverse directories regardless of ownership or permissions
Access restricted files such as those inside /root

This created a significant privilege escalation vector.

Step 3: Privilege Escalation Exploitation

The vulnerable tar binary was used to archive sensitive root-owned directories.

Command:
/home/cyber/tar -cvf /tmp/rootdir.tar /root
Explanation:

This command created an archive of the /root directory, bypassing permission restrictions due to the assigned capability.

Step 4: Extracting Sensitive Data

Once the archive was created, it was extracted into a readable location.

Command:
tar -xvf /tmp/rootdir.tar -C /tmp/

This exposed the contents of root’s home directory.

Step 5: Retrieving the Root Flag

The final objective was completed by reading the root flag.

Command:
cat /tmp/root/root.txt
Result:
Successfully retrieved the root flag, confirming full system compromise.
4. Security Issues Identified
Misconfigured Linux Capabilities on a non-standard tar binary
Excessive privilege assignment (cap_dac_read_search)
Sensitive administrative directories exposed through capability abuse
Lack of proper privilege restriction
5. Impact Assessment

This vulnerability allowed:

Unauthorized access to restricted root files
Full privilege escalation
Disclosure of sensitive system information
Complete compromise of the machine
6. Recommendations

To mitigate similar vulnerabilities:

Remove unnecessary capabilities from binaries:
setcap -r /home/cyber/tar
Restrict execution of custom binaries
Regularly audit file capabilities:
getcap -r / 2>/dev/null
Apply least privilege principles
Monitor unusual binary permissions and ownership
7. Conclusion

The Breakout VM was successfully exploited through:

Web shell access via Usermin
Enumeration of Linux Capabilities
Discovery of a vulnerable tar binary
Abuse of cap_dac_read_search
Extraction of root-owned files
Retrieval of the root flag

This demonstrates how improper Linux Capability configurations can provide attackers with powerful alternatives to traditional SUID-based privilege escalation.

Step 2: Identifying the Vulnerability (Linux Capabilities)
Traditional privilege escalation methods such as SUID binary enumeration were unsuccessful, so Linux Capabilities were investigated.
