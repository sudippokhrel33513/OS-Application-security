[lab08-README.md](https://github.com/user-attachments/files/28325469/lab08-README.md)
# 🖥️ VMware ESXi — Identity & Access Management Lab (Lab 08)

> **Fanshawe College — Information Security Management (INFO6003)**
> A hands-on lab focused on configuring user accounts, roles, permissions, and LAN segments inside a VMware ESXi 5.x environment using the vSphere Client. Demonstrates enterprise-level IAM and network segmentation skills.

---

## 🛠️ Tools & Technologies Used

![VMware](https://img.shields.io/badge/VMware_ESXi-607078?style=flat-square&logo=vmware&logoColor=white)
![vSphere](https://img.shields.io/badge/vSphere_Client-607078?style=flat-square&logo=vmware&logoColor=white)
![Windows](https://img.shields.io/badge/Windows_7-0078D6?style=flat-square&logo=windows&logoColor=white)
![PuTTY](https://img.shields.io/badge/PuTTY-SSH-4EAA25?style=flat-square&logo=gnubash&logoColor=white)

| Tool | Purpose |
|---|---|
| **VMware ESXi 5.5.0** | Bare-metal hypervisor hosting virtual machines |
| **vSphere Client** | GUI management interface for ESXi |
| **PuTTY (SSH)** | Remote shell access to ESXi host |
| **Windows 7 VM** | Client machine used to access vSphere |
| **LAN Segments** | Virtual isolated network segments for VMs |

---

## 🖥️ Lab Environment

- **ESXi Host:** `10.0.0.90` — VMware ESXi 5.5.0 Build 1623387
- **Client Machine:** Windows 7 (`W7-SPOKHREL`) connected via vSphere Client
- **VMs configured:**
  - `ISM_RP-01` → `spokhrel_VM-01`
  - `ISM_RP-02` → `spokhrel_VM-02`
- **Lab focus:** User creation, role assignment, permissions, LAN segmentation, SSH access

---

## 📌 Lab Tasks & Screenshots

---

### Task 1 — Creating a User Account on VMware ESXi

**Objective:** Create a new local user account (`spokhrel157855`) on the ESXi host with shell access.

**Steps performed in vSphere Client:**
1. Navigated to **Administration → Local Users & Groups → Users**
2. Clicked **Add** to create a new user
3. Filled in user details:
   - **Login:** `spokhrel157855`
   - **User Name:** `sudip pokhrel`
   - **UID:** `1000`
   - **Shell Access:** ✅ Granted
4. Confirmed creation — user appeared alongside `root`, `vpxuser`, and `dcui`

**Recent Tasks confirmed:**
- `Create user` → `ha-folder-root` → ✅ Completed (7/12/2023 2:58:52 PM)
- `Delete user` → `ha-folder-root` → ✅ Completed (7/12/2023 2:58:22 PM)

**Why it matters:** Managing local user accounts on a hypervisor is a fundamental IAM task in enterprise environments. Shell access control is critical — granting it incorrectly gives users direct ESXi command-line access, which is a significant security risk.

![User Account Creation on ESXi](lab08/1.jpg)

---

### Task 2 — Assigning Permissions to a Virtual Machine

**Objective:** Assign Administrator role permissions to users on `spokhrel_VM-02`.

**Steps performed:**
1. Selected `spokhrel_VM-02` in the vSphere inventory
2. Navigated to the **Permissions** tab
3. Confirmed the following users have **Administrator** role on this VM:

| User/Group | Role | Defined In |
|---|---|---|
| spokhrel157855 | Administrator | 10.0.0.90 |
| vpxuser | Administrator | 10.0.0.90 |
| dcui | Administrator | 10.0.0.90 |
| root | Administrator | 10.0.0.90 |

**Recent Tasks confirmed:**
- `Set entity permission ru...` → ✅ Completed × 2 (7/12/2023 3:46 PM)

**Why it matters:** In VMware ESXi, permissions are applied at the object level (host, cluster, VM). Assigning roles per VM is a key **least privilege** practice — users should only have access to the VMs they need, not the entire host.

![VM Permissions Assignment](lab08/3.jpg)

---

### Task 3 — Role-Based Access Control on VM-01 & Verifying on Windows 7

**Objective:** Apply granular role-based permissions on `spokhrel_VM-01` and verify the Windows 7 machine identity.

**Permissions configured on `spokhrel_VM-01`:**

| User/Group | Role | Defined In |
|---|---|---|
| spokhrel157855 | **Power State** | This object |
| vpxuser | Administrator | 10.0.0.90 |
| dcui | Administrator | 10.0.0.90 |
| root | Administrator | 10.0.0.90 |

**Key difference:** `spokhrel157855` was given only **Power State** role on VM-01 (can start/stop the VM) — NOT Administrator. This demonstrates **least privilege** — the user has just enough access to do their job, no more.

**Windows 7 verification (bottom terminal):**
```cmd
net config workstation | find "name"
Computer name:      \\W7-SPOKHREL
Full Computer name: W7-spokhrel.pokhrel.ca
User name:          Administrator
```

**Recent Tasks confirmed:**
- `Set entity permission ru...` → ✅ Completed (7/12/2023 4:10 PM)
- `Update role` → ✅ Completed (7/12/2023 4:08 PM)

**Why it matters:** Assigning different roles to the same user on different VMs is a core RBAC principle. `spokhrel157855` is an Administrator on VM-02 but only has Power State access on VM-01 — demonstrating real-world access segmentation.

![RBAC and Power State Role](lab08/4.jpg)

---

### Task 4 — SSH Access to ESXi via PuTTY & New User Login Attempt

**Objective:** Attempt SSH access to the ESXi host using the newly created user and observe access control behavior.

**ESXi Host Permissions tab showed:**

| User/Group | Role | Defined In |
|---|---|---|
| spokhrel-01 | Administrator | This object |
| spokhrel157855 | Administrator | This object |
| vpxuser | Administrator | This object |
| dcui | Administrator | This object |
| root | Administrator | This object |

**PuTTY SSH session to `10.0.0.90`:**
```
login as: spokhrel-01
Using keyboard-interactive authentication.
Password:
Access denied
Using keyboard-interactive authentication.
Password:
The time and date of this login have been sent to the system logs.
~ #
```

**Key observations:**
- First password attempt was **denied** — simulating a wrong password / failed login
- Second attempt **succeeded** — logged into ESXi shell as `spokhrel-01`
- ESXi shell banner confirmed: *"The ESXi Shell can be disabled by an administrative user"*
- `~ #` prompt confirms successful root-level ESXi shell access

**Recent Tasks:**
- `Set entity permission ru...` → ✅ Completed (7/12/2023 4:29 PM)
- `Create user` → ❌ Failed — "The specified key" error

**Why it matters:** SSH access to the ESXi shell is a powerful and sensitive capability. Failed login attempts being logged to system logs is a critical **audit trail** feature. The failed user creation task also highlights the importance of error handling in IAM operations.

![SSH Access via PuTTY to ESXi](lab08/5.jpg)

---

### Task 5 — LAN Segments: Network Segmentation Theory & Findings

**Objective:** Explain the purpose of LAN segments in VMware and document why a segmentation attempt failed.

**Why LAN segments are used:**

| Benefit | Description |
|---|---|
| 🚀 Performance | Reduces broadcast traffic, increases available bandwidth |
| 🔒 Security | Isolates VMs from each other and from the physical network |
| 📋 Manageability | Simplifies network management by grouping VMs logically |
| 📈 Scalability | Easily add VMs to segments without physical reconfiguration |
| 🛡️ Fault Isolation | A problem in one segment doesn't affect others |
| 🎯 QoS | Quality of Service techniques can be applied per segment |

**Why did this fail?**

> *"Propagation is removed from ISM_RP-01S"*

The LAN segment configuration failed because **propagation was removed from `ISM_RP-01S`** — meaning the network segment settings were not inherited/propagated down to the child objects (VMs) in that resource pool. Without propagation enabled, VMs in that pool cannot receive the segment configuration from the parent object.

**Why it matters:** Understanding LAN segment propagation is critical for network architects. A misconfigured propagation setting can silently break network isolation — a serious security gap in a production VMware environment.

![LAN Segments Theory and Failure Analysis](Lab08/5.jpg)

---

## 🔁 Full Lab Workflow Summary

```
[1] Create user 'spokhrel157855' on ESXi → UID 1000, shell access granted
        ↓
[2] Assign Administrator permissions on spokhrel_VM-02 to all users
        ↓
[3] Apply RBAC on spokhrel_VM-01 → spokhrel157855 gets Power State only
        ↓
[4] SSH into ESXi via PuTTY → login as spokhrel-01 → shell access confirmed
        ↓
[5] Configure LAN segments → propagation failure identified and explained
```

---

## 📚 Key Takeaways

- **User management on ESXi** is done via vSphere Client under Local Users & Groups
- **Shell access** on ESXi should be tightly controlled — it gives direct hypervisor command-line access
- **RBAC in VMware** allows per-VM role assignment — same user can have different roles on different VMs
- **Least privilege** was demonstrated by giving `spokhrel157855` only Power State (not Admin) on VM-01
- **Failed login attempts** are logged to ESXi system logs — a key audit trail for security monitoring
- **LAN segment propagation** must be enabled for network isolation settings to apply to child VMs

---

## ⚠️ Disclaimer

> All activities were performed in a **controlled, isolated VMware ESXi lab environment** for educational purposes as part of the Fanshawe College Information Security Management (INFO6003) program. No production systems were accessed or modified.

---

## 👨‍💻 Author

**Sudip Pokhrel** | Information Security Analyst — GRC

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/sudip-pokhrel-3375291b3/)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/sudippokhrel33513)


[Lab09-README.md](https://github.com/user-attachments/files/28325675/Lab09-README.md)
# 🐧 Linux File Permissions, User Management & Password Security (Lab 09)

> **Fanshawe College — Information Security Management (INFO6003)**
> A hands-on lab exploring Linux user account management, file permissions, special permission bits (SUID, SGID, Sticky Bit), shared directory access control, and password hashing — all performed on an Ubuntu Server VM inside VMware Workstation.

---

## 🛠️ Tools & Technologies Used

![Ubuntu](https://img.shields.io/badge/Ubuntu_Server-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black)
![VMware](https://img.shields.io/badge/VMware-607078?style=flat-square&logo=vmware&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white)

| Concept | Description |
|---|---|
| **User Management** | Creating users, viewing `/etc/passwd` entries |
| **File Permissions** | Read/write/execute, owner/group/other |
| **SUID / SGID** | Special permission bits for privilege escalation control |
| **Sticky Bit** | Protects shared directories from unauthorized deletion |
| **Shadow File** | Viewing hashed passwords in `/etc/shadow` |
| **PAM (pam_unix)** | Password policy enforcement via SHA-256 hashing |
| **Shared Directory** | `/ShareAll` — multi-user shared folder access control |

---

## 🖥️ Lab Environment

- **Server VM:** Ubuntu Server (`U7-spokhrel157855`) — VMware Workstation
- **OS:** Ubuntu 12.04 LTS (Precise Pangolin) — Linux 3.8.0-29-generic x86_64
- **Users created:** `spokhrel157855`, `student-01`, `student-02`, `spokhre1-01`, `saved`
- **Shared directory:** `/ShareAll` — world-writable with sticky bit

---

## 📌 Lab Tasks & Screenshots

---

### Task 1 — User Account Review & `/etc/passwd` Exploration

**Objective:** Review all user accounts on the Ubuntu server and understand the `/etc/passwd` file structure.

**Commands used:**
```bash
ls -all /home/student-01/test
uname -a
```

**`/etc/passwd` entries visible:**

| User | UID | GID | Home | Shell |
|---|---|---|---|---|
| proxy | 13 | 13 | `/bin` | `/bin/sh` |
| www-data | 33 | 33 | `/var/www` | `/bin/sh` |
| backup | 34 | 34 | `/var/backups` | `/bin/sh` |
| list | 38 | 38 | `/var/list` | `/bin/sh` |
| irc | 39 | 39 | `/var/run/ircd` | `/bin/sh` |
| nobody | 65534 | 65534 | `nonexistent` | `/bin/sh` |
| syslog | 102 | 104 | `/` | `/bin/false` |
| messagebus | 102 | 104 | `/var/run/dbus` | `/bin/false` |
| **saved** | 1000 | 1000 | `/home/saved` | `/bin/bash` |
| **spokhrel157855** | 1001 | 1001 | `/home/spokhrel157855` | `/bin/bash` |
| **student-01** | 1002 | 1002 | `/home/student-01` | `/bin/bash` |
| **student-02** | 1003 | 1003 | `/home/student-02` | `/bin/bash` |
| **spokhre1-01** | 1004 | 1004 | `/home/spokhre1-01` | `/bin/bash` |

**File listing in `/home/student-01/test`:**
```
total 8
265031 drwxrwxr-x  2  student-01  student-01  4096  Jul 26  11:34  .
262161 drwxr-x    4  student-01  student-01  4096  Jul 26  11:33  ..
265032 -rwSrw-r--  1  student-01  student-01     0  Jul 26  11:34  file-01
```

**Key finding — `file-01` has capital `S`:**
- `s` (lowercase) = **setuid + executable** — the file runs with owner's privileges
- `S` (uppercase) = **setuid + NOT executable** — setuid is set but has no effect since the file isn't executable — a misconfiguration or intentional test

**`uname -a` confirms:** Ubuntu 12.04 LTS, Linux 3.8.0-29-generic x86_64, Aug 14 2013

![User Accounts and /etc/passwd Review](lab-09/1.jpg)

---

### Task 2 — Shared Directory (`/ShareAll`) Access Control & Deletion Testing

**Objective:** Test file creation, ownership, and deletion behavior in a shared directory to demonstrate Linux permission enforcement.

**Commands used:**
```bash
cd /
cd ShareAll
touch pokhrel
ls -all
rm root-01          # attempt 1 — wrong path
rm root-01          # attempt 2 — prompted for confirmation
ls -all             # verify result
```

**`/ShareAll` directory listing (before):**
```
524290  drwxrwxrwx  2  root        root        4096  Jul 26  12:07  .
        2  drwxr-x   25  root        root        4096  Jul 26  11:59  ..
524293  -rw-rw-r--  1  spokhrel-01  spokhrel-01    0  Jul 26  12:07  pokhrel
524291  -rw-r--r--  1  root         root           0  Jul 26  12:02  root-01
524292  -rw-rw-r--  1  student-01   student-01     0  Jul 26  12:06  student-01
```

**Deletion attempt of `root-01`:**
```bash
rm root-01
# rm: cannot remove 'root-01': No such file or directory  ← wrong path

rm root-01
# rm: remove write-protected regular empty file 'root-01'? y  ← confirmed
```

**`/ShareAll` directory listing (after):**
```
524290  drwxrwxrwx  2  root        root        4096  Jul 26  12:09  .
        2  drwxr-x   25  root        root        4096  Jul 26  11:59  ..
524293  -rw-rw-r--  1  spokhrel-01  spokhrel-01    0  Jul 26  12:07  pokhrel
524292  -rw-rw-r--  1  student-01   student-01     0  Jul 26  12:06  student-01
```

**`root-01` was deleted** because the directory was `drwxrwxrwx` (world-writable) — any user can delete any file. The sticky bit had not yet been applied.

**Why it matters:** A world-writable directory without a sticky bit is a serious security risk — any user can delete other users' files. This sets up the need for the sticky bit demonstrated in Task 3.

![Shared Directory Access Control Testing](lab-09/2.jpg)

---

### Task 3 — Sticky Bit & Root Permission Enforcement

**Objective:** Apply the sticky bit to `/ShareAll` and test whether root can override file ownership protections.

**Commands used:**
```bash
ls -all              # confirm current state
rm root-01           # attempt deletion as root
uname -a
```

**Directory listing with sticky bit applied:**
```
524290  drwxrwxrwt  2  root  root  4096  Jul 26  12:15  .
        2  drwxr-x   25  root  root  4096  Jul 26  11:59  ..
524293  -rw-rw-r--  1  spokhrel-01  spokhrel-01  0  Jul 26  12:07  pokhrel
524291  -rw-r--r--  1  root         root          0  Jul 26  12:15  root-01
```

**Note the `t` at the end of `drwxrwxrwt`** — this is the **sticky bit** applied to `/ShareAll`.

**Deletion attempt:**
```bash
rm root-01
# rm: cannot remove 'root-01': Operation not permitted
```

**Even root cannot delete `root-01`** because the sticky bit ensures only the file's **owner** can delete it — not even other administrators.

**Q&A from the lab:**
- **Q:** Which directory are you in? What changed in the prompt?
  - **A:** Root directory (`#`). The prompt changed from `user@host` to `root@host` — confirming privilege escalation
- **Q:** Why is `S` capital instead of small `s`?
  - **A:** `s` = setuid AND executable; `S` = setuid but NOT executable

**`uname -a` confirms:** Ubuntu 12.04 LTS, Linux 3.8.0-29-generic x86_64

![Sticky Bit and Root Permission Testing](lab-09/3.jpg)

---

### Task 4 — Password Hashing & `/etc/shadow` File Inspection

**Objective:** Inspect the PAM password policy and view hashed passwords in the `/etc/shadow` file.

**Commands used:**
```bash
cat /etc/pam.d/common-password | grep success=
cat /etc/shadow | grep student-03
cat /etc/shadow | grep student-04
uname -a
```

**PAM password policy output:**
```
password  [success=1 default=ignore]  pam_unix.so obscure sha256
```
This confirms passwords are hashed using **SHA-256** via `pam_unix.so` with the `obscure` flag (enforces minimum complexity).

**`/etc/shadow` entries for student accounts:**

| User | Hash Algorithm | Hash (truncated) |
|---|---|---|
| student-03 | `$1$` (MD5) | `$1$nLWT7ot$M8OT2YFeuwbl.Gnofilbn/` |
| student-04 | `$5$` (SHA-256) | `$5$2LUMVW19$WSXVY51K7Dzr/ptCfASEDIfR1FHJ2P2MCaA1zwATHO` |

**Hash prefix meanings:**
- `$1$` = **MD5** — older, weaker hashing (student-03)
- `$5$` = **SHA-256** — stronger, recommended hashing (student-04)
- `$6$` = SHA-512 (most secure, not shown here)

**Shadow file format:** `username:$algorithm$salt$hash:last_change:min:max:warn:inactive:expire`

**Why it matters:** The shadow file stores password hashes rather than plaintext. Seeing different hash algorithms (`$1$` vs `$5$`) shows that older accounts may use weaker hashing — a key finding in a security audit. Password policy enforcement via PAM is a fundamental Linux hardening technique.

![Password Hashing and Shadow File](lab-09/4.jpg)

---

### Task 5 — Q&A: Prompt Changes & SUID Bit Explanation

**Objective:** Answer theoretical questions about Linux privilege indicators and special permission bits.

**Q1: Which directory are you in now? What changes to the prompt can you see?**
> **Answer:** The current directory is the root directory (`/`). The prompt changed from `user@host:~$` to `root@host:#` — the `#` symbol indicates root-level privileges, as opposed to `$` for regular users.

**Q2: Why is this a capital `S` instead of a small `s`?**

| Symbol | Meaning |
|---|---|
| `s` (lowercase) | **setuid is set AND the file is executable** — runs with owner's privileges |
| `S` (uppercase) | **setuid is set BUT the file is NOT executable** — setuid has no practical effect |

**Real-world significance:**
- `s` on `/usr/bin/passwd` = users can change their own password (runs as root temporarily)
- `S` on a non-executable file = a misconfiguration — setuid is meaningless without execute permission
- Attackers look for SUID files as privilege escalation vectors

![SUID Bit and Prompt Change Explanation](Lab09/lab09-slide-5.jpg)

---

### Task 6 — Sticky Bit Deep Dive & Permission Denial

**Objective:** Understand the sticky bit's role on `/ShareAll` and `/tmp` directories, and confirm permission denial behavior.

**Q1: The `t` has been added — what does the sticky bit on `/ShareAll` do? Why does `/tmp` also have it?**

> **Answer:** The sticky bit on `/ShareAll` ensures that **only the file's owner** can alter or delete their own files within the directory, even though the directory is world-writable. On `/tmp`, the sticky bit serves the same purpose — it provides a secure, organised environment for shared temporary file storage by multiple users, preventing one user from deleting another's temp files.

**Q2: What happened when trying to delete another user's file? Was permission granted?**

> **Answer:** The file was **not deleted**. Permission was denied — the operation was cancelled. This is the sticky bit working correctly — `spokhrel-01` cannot delete `student-01`'s file even in a world-writable directory.

**Permission comparison:**

| Directory | Permissions | Sticky Bit | Effect |
|---|---|---|---|
| `/ShareAll` | `drwxrwxrwt` | ✅ Yes | Only owners can delete their files |
| `/tmp` | `drwxrwxrwt` | ✅ Yes | Same — protects temp files from cross-user deletion |
| `/ShareAll` (before) | `drwxrwxrwx` | ❌ No | Anyone could delete anyone's files |

**Why it matters:** The sticky bit is a critical Linux access control mechanism used on all shared directories. Without it, a world-writable directory is vulnerable to **data destruction attacks** where any user can delete others' files.

![Sticky Bit Deep Dive and Permission Denial](Lab09/lab09-slide-6.jpg)

---

## 🔁 Full Lab Workflow Summary

```
[1] Review /etc/passwd → identified 5 custom users with UIDs 1000-1004
        ↓
[2] Test /ShareAll (no sticky bit) → any user can delete any file
        ↓
[3] Apply sticky bit (drwxrwxrwt) → root cannot delete others' files
        ↓
[4] Inspect /etc/shadow → MD5 ($1$) vs SHA-256 ($5$) password hashes
        ↓
[5] Explain SUID: s(lowercase)=executable, S(uppercase)=not executable
        ↓
[6] Confirm sticky bit behavior → permission denied when deleting others' files
```

---

## 📚 Key Takeaways

- **`/etc/passwd`** stores user account info; **`/etc/shadow`** stores hashed passwords — never plaintext
- **SHA-256 (`$5$`)** is stronger than **MD5 (`$1$`)** — older accounts should be upgraded
- **PAM (`pam_unix.so`)** enforces password complexity and hashing algorithm policy
- **World-writable directories without sticky bit** are a security risk — anyone can delete anyone's files
- **Sticky bit (`t`)** on `/ShareAll` and `/tmp` ensures only file owners can delete their own files
- **SUID `s`** = runs with owner privileges; **SUID `S`** = setuid set but file not executable (misconfiguration)
- The **`#` prompt** vs **`$` prompt** is a visual indicator of root vs regular user privilege level

---

## ⚠️ Disclaimer

> All activities were performed in a **controlled, isolated VMware lab environment** for educational purposes as part of the Fanshawe College Information Security Management (INFO6003) program. No production systems were accessed or modified.

---

## 👨‍💻 Author

**Sudip Pokhrel** | Information Security Analyst — GRC

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/sudip-pokhrel-3375291b3/)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/sudippokhrel33513)
