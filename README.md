# Privilege Escalation Report: Exploiting Linux Capabilities

## 1. Executive Summary
The target system was successfully compromised by establishing initial access through an exposed Webmin web shell. Following this, full root administrative privileges were secured by exploiting a misconfigured Linux Capability assigned to a localized `tar` executable. Specifically, the `cap_dac_read_search` capability allowed the bypassing of Discretionary Access Control (DAC) file permission policies, leading to complete system compromise.

## 2. Tools Utilized
* **Web Browser:** Used to interact with and gain access via the Webmin Web Shell (port 20000).
* **Bash (Linux Command Line):** The primary interface for system enumeration and executing the privilege escalation chain.
* **getcap:** Utilized to scan the filesystem for binaries with elevated Linux Capabilities.
* **tar:** The vulnerable binary abused to bypass DAC restrictions and package restricted files.
* **cat:** Used to read the final extracted flags and sensitive files.

## 3. Step-by-Step Exploitation Process

### Step 1: Initial Access & System Enumeration
After securing an initial shell as the unprivileged `cyber` user, basic system enumeration was conducted to map out potential privilege escalation vectors.

* **Command:** `whoami`
    * *Result:* Verified the active session was running as the `cyber` user.
* **Command:** `sudo -l`
    * *Result:* The `sudo` utility was not installed on the system, eliminating any sudo-based escalation paths.

### Step 2: Vulnerability Discovery (Linux Capabilities)
Since traditional SUID binary enumeration did not yield any viable paths, the focus shifted toward analyzing Linux Capabilities assigned to system binaries.

* **Command:** ```bash
    getcap -r / 2>/dev/null
    ```
    * *Discovery:* Located a custom `tar` executable at `/home/cyber/tar` assigned the following capability: `cap_dac_read_search=ep`.

**Technical Analysis:**
The `cap_dac_read_search` capability grants a binary the authority to bypass standard file read permissions and directory execution (search) checks. Essentially, this allowed the custom `tar` binary to read and archive restricted files—such as those located in the `/root` directory—regardless of the `cyber` user's actual permissions. This represents a critical privilege escalation vector.

### Step 3: Exploitation via Archiving
The misconfigured `tar` binary was weaponized to package the contents of the restricted root home directory into an archive stored in a globally writable location.

* **Command:** ```bash
    /home/cyber/tar -cvf /tmp/rootdir.tar /root
    ```
    * *Explanation:* This execution successfully bypassed system permission checks and created a compressed archive of the `/root` directory inside the `/tmp` folder.

### Step 4: Extracting Sensitive Data
With the archive successfully generated in a directory accessible to our current user, the contents were extracted to reveal the restricted files.

* **Command:** ```bash
    tar -xvf /tmp/rootdir.tar -C /tmp/
    ```
    * *Result:* The `/root` directory's contents were successfully unpacked into `/tmp/root`, making them readable by the `cyber` user.

### Step 5: Retrieving the Root Flag
The exploitation chain was finalized by reading the target objective file.

* **Command:** ```bash
    cat /tmp/root/root.txt
    ```
    * *Result:* The root flag was successfully read, verifying full access to root-level data.

## 4. Impact Assessment
This misconfiguration represents a critical security failure. The vulnerability permitted:
* Complete bypass of file read restrictions.
* Unauthorized extraction of restricted root files.
* Disclosure of highly sensitive system information.
* Total compromise of the target machine's data integrity.

## 5. Remediation Recommendations
To patch this vulnerability and mitigate similar threats, it is recommended to:
1.  Remove the excessive capability from the binary using the following command: `setcap -r /home/cyber/tar`.
2.  Regularly audit system binaries using `getcap` to ensure least privilege principles are strictly enforced.
