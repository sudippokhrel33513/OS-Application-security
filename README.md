[Lab08-README.md](https://github.com/user-attachments/files/28325469/Lab08-README.md)
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

![User Account Creation on ESXi](Lab08/1.jpg)

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

![VM Permissions Assignment](Lab08/2.jpg)

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

![RBAC and Power State Role](Lab08/3.jpg)

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

![SSH Access via PuTTY to ESXi](Lab08/4.jpg)

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

![LAN Segments Theory and Failure Analysis](Lab08/lab08-slide-5.jpg)

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
