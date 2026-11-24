# Case 008: Windows User Accounts and NTFS Permissions

## Overview

This case documents an investigation into Windows local user accounts, account privilege levels, NTFS access control, folder ownership, permission inheritance, and shared folder configuration.

The investigation focused on controlling access to personal data between local accounts, understanding why administrator privileges do not guarantee unrestricted access to protected system folders, and configuring folders for either private or shared use.

The work involved both graphical Windows security settings and command-line permission management using `icacls`.

**Investigation Date:** August 2026

**Status:** Partially resolved — multiple permission and account management scenarios investigated.

---

## 1. Objectives

The investigation aimed to understand and configure:

- Local Windows accounts and privilege levels.
- Administrator versus Standard User permissions.
- Private folders accessible only to a specific user.
- NTFS permissions and Access Control Lists (ACLs).
- Permission inheritance.
- Folder ownership.
- Shared folders for multiple local users.
- Public Desktop shortcuts.
- Windows system identities and protected folders.
- SMB access to shared folders from another device.
- Permission troubleshooting and recovery.

---

## 2. User Account Configuration

The Windows system used multiple local user accounts, including:

- `Xenon` — primary account with administrative privileges.
- `Family` — secondary account intended for restricted everyday use.

The investigation explored the distinction between account membership and permissions granted to individual files and folders.

### Administrator

An Administrator account can perform many system-level operations, including:

- Installing software.
- Modifying system settings.
- Managing accounts.
- Changing many file and folder permissions.
- Performing administrative maintenance.

However, administrative privileges do not automatically bypass every Windows security mechanism.

Protected system resources may still be controlled by specific identities, ownership rules, permissions, and security mechanisms.

### Standard User

A Standard User account is intended for routine tasks without unrestricted administrative privileges.

The investigation explored restricting the secondary account from accessing selected folders while preserving the ability to use the computer normally.

---

## 3. NTFS Permissions

NTFS permissions control access to files and folders on NTFS-formatted volumes.

The investigation involved examining the Security tab in folder Properties and the Advanced Security Settings interface.

Permissions investigated included:

| Permission | Purpose |
|---|---|
| Full control | Broad control over files and folders, including permission management |
| Modify | Read, write, modify, and delete access |
| Read & execute | Read files and execute permitted applications |
| List folder contents | Enumerate folder contents |
| Read | View file and folder information and contents |
| Write | Create or modify files and folders |
| Special permissions | More granular access control |

The exact permissions available to an account depend on the combination of its explicit permissions, inherited permissions, group memberships, and applicable deny rules.

---

## 4. Private Folder Access

One of the investigated scenarios involved restricting a personal folder so that only the primary account `Xenon` could access it.

The folder was located on the secondary drive.

The investigation included:

- Reviewing the folder's existing security entries.
- Examining the permissions granted to `Xenon`.
- Examining the permissions granted to `Family`.
- Reviewing inherited permissions.
- Adding or removing account entries.
- Attempting to restrict access to the secondary account.
- Testing permission changes.

### Permission configuration

The Windows Security interface was used to examine and modify entries in the folder's ACL.

The investigation highlighted the importance of identifying exactly which accounts and groups have access.

An ACL may contain entries for:

- Individual local users.
- Local groups.
- SYSTEM.
- Administrators.
- Authenticated Users.
- Users.
- CREATOR OWNER.
- Other Windows security identities.

Removing or changing one account entry does not necessarily remove access granted through another group.

### Important distinction

A user may have access to a folder even when their individual username is not explicitly listed.

For example, permissions may be granted through group membership or inherited from a parent folder.

Therefore, restricting a folder requires evaluating the complete effective permission configuration, not just the visible username entries.

---

## 5. Permission Inheritance

Permission inheritance was investigated while configuring folders on a secondary drive.

Windows can inherit permissions from parent folders.

For example:

```text
D:\
└── Parent Folder
    └── Private Folder
        └── Files
```

Permissions assigned to the parent folder may propagate to child folders and files, depending on the inheritance configuration.

The Advanced Security Settings interface provides options to:

- Disable inheritance.
- Convert inherited permissions into explicit permissions.
- Remove inherited permissions.
- Configure permissions for specific objects.
- Control how permissions apply to subfolders and files.

### Investigation observations

Some permission changes were difficult to understand because existing entries were inherited.

The investigation explored disabling inheritance and converting inherited permissions into explicit entries before further modifications.

This made it possible to examine permissions more directly, but also introduced the risk of accidentally removing permissions needed for normal operation.

### Lesson

Inheritance should not be disabled casually.

Before modifying an ACL, determine:

1. Which entries are inherited.
2. Which parent folder provides them.
3. Which entries are required for normal system or application operation.
4. Whether the desired restriction can be achieved without replacing the entire ACL.

---

## 6. Ownership

Folder ownership was another major topic.

The investigation encountered folders whose owner was not the logged-in user, including folders owned by `SYSTEM` or `TrustedInstaller`.

The owner of an object has important rights related to managing its permissions, but ownership and access permissions are separate concepts.

Changing the owner does not automatically grant every desired file operation, and granting Full Control does not necessarily mean the user owns the folder.

### Ownership investigation

The Advanced Security Settings interface was used to inspect ownership and explore changing the owner.

One scenario involved a shared folder where the primary account had been granted additional permissions, but the expected ability to create files or shortcuts was still unavailable.

This led to investigating whether the owner needed to be changed.

### Public Desktop

The Public Desktop folder was investigated as a location for shortcuts intended to appear for all local users.

The objective was to create shared shortcuts rather than placing them only on the primary user's desktop.

The folder's existing owner and permissions were examined.

The investigation demonstrated that a folder intended for shared use may have a specific system-managed security configuration.

Changing its owner or recursively replacing permissions may have unintended consequences.

A safer approach is to understand the intended folder permissions first and use the designated shared location rather than modifying protected system folder ownership without a clear requirement.

---

## 7. Protected Windows System Folders

The investigation also examined why access to some Windows system folders could not be modified easily, even while logged in as an Administrator.

Examples included:

- `C:\Windows`
- `C:\Program Files`
- `C:\Program Files (x86)`

These folders may contain entries associated with `TrustedInstaller`, SYSTEM, Administrators, and other security identities.

### TrustedInstaller

TrustedInstaller is associated with the Windows Modules Installer service and is used to protect system components.

Windows uses protected ownership and permission configurations to reduce the risk of accidental or unauthorized modifications to system files.

The investigation encountered restrictions while reviewing the permissions of protected folders.

### Important finding

Administrator privileges do not mean every protected Windows file or folder should be modified.

Taking ownership of protected system folders and replacing their ACLs may interfere with:

- Windows servicing.
- Updates.
- Application installation.
- System integrity.
- Operating system functionality.

**Changing ownership of critical Windows folders was not considered a suitable general solution.**

---

## 8. Windows Security Identities

The investigation explored several built-in Windows security identities that appeared in permission lists.

| Identity | General role |
|---|---|
| SYSTEM | Windows operating system and system-level processes |
| Administrators | Administrative group |
| Users | General local user group |
| Authenticated Users | Accounts authenticated to the system |
| CREATOR OWNER | Placeholder representing the creator/owner in applicable permission rules |
| INTERACTIVE | Users logged in interactively |
| BATCH | Processes running in batch context |
| SERVICE | Service-related security context |

These identities are not necessarily ordinary accounts that can be used for interactive login.

Some are security principals used by Windows to evaluate permissions.

### Unknown accounts

An ACL may contain a security identifier (SID) that Windows can no longer resolve to a recognizable account name.

This can happen when the original account is removed or becomes unavailable.

An unresolved SID should not automatically be assumed to be malicious.

Its origin and purpose should be investigated before removing it.

---

## 9. Effective Access

The Advanced Security Settings interface includes an Effective Access feature.

It was investigated as a method of determining what permissions a specific user actually has.

Effective Access can help evaluate permissions resulting from multiple sources, including:

- Direct permissions.
- Group memberships.
- Inheritance.
- Deny entries.

This is useful when a user appears to have access despite not being explicitly listed, or when permissions are difficult to interpret.

### Investigation lesson

The Effective Access interface is preferable to relying solely on visual inspection of individual Allow checkboxes when troubleshooting complex ACLs.

---

## 10. Shared Folders and SMB Access

The investigation included creating folders intended for different access scopes.

Examples included:

- A private folder accessible only to `Xenon`.
- A shared folder intended for all local users.
- A folder shared over the local network using SMB.

The folder permissions and network sharing configuration were examined separately.

### NTFS permissions versus share permissions

Windows network access can involve two separate permission layers:

1. NTFS permissions — control access to the files and folders on the filesystem.
2. Share permissions — control access through the network share.

The effective network access is determined by the combination of these permissions.

A share permission alone does not override a restrictive NTFS ACL.

Likewise, NTFS Full Control does not automatically mean the user can connect through SMB if the share permissions or network configuration prevent access.

### SMB client investigation

An Android SMB client was configured to connect to a Windows PC using a username and password.

The investigation included:

- Specifying the Windows PC's network address.
- Specifying a shared folder.
- Using the `Xenon` account credentials.
- Creating separate private and shared folder scenarios.
- Examining the relationship between the account used for authentication and the folder's ACL.

A successful SMB connection requires the client to authenticate using an account that has appropriate access to the share and filesystem.

---

## 11. Permission Management with `icacls`

The investigation explored using `icacls` to manage NTFS permissions from the command line.

The tool can be used to inspect and modify ACLs, including:

- Granting permissions.
- Removing permission entries.
- Resetting permissions.
- Managing inheritance.
- Applying changes recursively.

Example commands investigated included:

```cmd
icacls "D:\Xenon"
```

Inspect permissions for a folder.

```cmd
icacls "D:\Xenon" /reset
```

Reset ACLs to inherited/default permissions.

```cmd
icacls "D:\Xenon" /grant "Xenon:(OI)(CI)F" /t
```

Grant Full Control to a specified account recursively, with inheritance flags for child objects.

```cmd
icacls "D:\Xenon" /remove "Family"
```

Remove a specified account's explicit permission entry.

### Risks

Recursive permission changes can affect large numbers of files and folders.

Resetting ACLs may remove custom security configuration.

Granting Full Control recursively can unintentionally expose sensitive files or allow destructive operations.

Command-line permission changes should be applied only after reviewing the existing ACL and confirming the target path.

---

## 12. UAC and Permission Changes

The investigation explored how User Account Control (UAC) interacts with access to restricted folders.

In one scenario, accessing a restricted folder prompted for administrative authentication.

After authentication, the folder's permission configuration appeared to have changed persistently.

This led to investigating the difference between temporary elevation and permanent changes to an ACL.

### Key distinction

UAC elevation and NTFS permission modification are different operations.

A UAC prompt may allow an elevated process to perform an operation, but permission changes made during that process can potentially persist beyond the immediate access attempt.

The investigation demonstrated the importance of checking the resulting ACL after granting access rather than assuming access was temporary.

---

## 13. Permission Troubleshooting Challenges

Several difficulties were encountered during the investigation:

- Permissions were inherited from parent folders.
- Ownership and permissions were initially treated as closely related but needed to be evaluated separately.
- Administrators could not freely modify some protected system folders.
- Shared folder permissions differed from private folder permissions.
- User accounts could receive access through group memberships.
- UAC authentication could result in persistent permission changes.
- Windows displayed several built-in security identities that were not ordinary users.
- Removing one user entry did not necessarily guarantee that access was removed through other groups.
- Changing folder ownership did not automatically solve every access problem.

These challenges reinforced the need to inspect the full ACL and security context before applying changes.

---

## 14. Troubleshooting Summary

| Investigation | Outcome |
|---|---|
| Local account privilege levels | Investigated |
| Administrator vs Standard User | Investigated |
| Private folder access configuration | Investigated |
| NTFS ACL management | Investigated |
| Permission inheritance | Investigated |
| Folder ownership | Investigated |
| SYSTEM and TrustedInstaller ownership | Investigated |
| Public Desktop access | Investigated |
| Windows protected folders | Investigated |
| Built-in security identities | Investigated |
| Effective Access | Investigated |
| Share permissions vs NTFS permissions | Investigated |
| SMB access from Android | Tested as part of home network setup |
| `icacls` permission management | Investigated |
| UAC-related persistent permission changes | Investigated |

---

## 15. Lessons Learned

### 15.1 Ownership and permissions are different

Ownership identifies the owner of an object.

ACL permissions determine what users and groups are allowed or denied.

Changing one does not necessarily solve a problem involving the other.

### 15.2 Administrator is not unrestricted

Windows deliberately protects critical operating system folders.

Administrative privileges should not be treated as a reason to bypass protected ownership or replace system ACLs.

### 15.3 Understand inheritance before changing permissions

Inherited permissions can grant access through parent folders and groups.

Removing or disabling inheritance without understanding the existing configuration can make a folder inaccessible or disrupt normal operation.

### 15.4 Share permissions and NTFS permissions are separate

Network access requires evaluating both layers.

A successful local access test does not automatically prove that SMB access will work.

### 15.5 Avoid unnecessary Deny entries

Explicit Deny permissions can complicate access evaluation and override otherwise valid Allow permissions.

In many scenarios, configuring appropriate Allow entries and removing unnecessary inherited access is easier to maintain than introducing broad Deny rules.

### 15.6 Use Effective Access for verification

Effective Access can help confirm what a specific account is allowed to do.

Permission configuration should be verified using the intended account rather than relying solely on the administrator's view.

### 15.7 Avoid recursive changes to protected locations

Recursive ownership or ACL modifications may affect system files and application directories.

Changes should be limited to the intended user-managed folder whenever possible.

---

## 16. Final Outcome

The investigation improved understanding of Windows local accounts, privilege levels, NTFS permissions, inheritance, ownership, system security identities, UAC, and SMB access control.

Multiple permission scenarios were examined, including private folders, shared folders, Public Desktop, and protected Windows system locations.

The investigation was exploratory and involved several configuration attempts. Not every scenario was independently verified in a final clean configuration.

**Final status: Partially resolved — permission concepts and several configuration scenarios investigated; complete validation of all configurations not established.**

---

## 17. Future Work

Potential future investigation:

- Create a clean test environment with multiple local users.
- Build a repeatable private-folder permission configuration.
- Build a repeatable shared-folder configuration.
- Test NTFS and SMB share permissions independently.
- Verify effective permissions using Effective Access.
- Compare GUI permission changes with `icacls`.
- Document inheritance behavior using test folders.
- Test account removal and unresolved SID cleanup.
- Explore auditing access to sensitive folders.
- Develop a safe permission backup and recovery procedure.
- Document the distinction between UAC elevation, ownership, and ACL modifications with reproducible tests.

---

## Disclaimer

This case documents personal Windows administration and troubleshooting experiments.

It does not claim that every permission configuration investigated was a recommended production configuration.

Changes to ownership or permissions on Windows system folders can cause system instability, application failures, or update problems.

All examples should be tested on user-managed folders or a controlled test environment before applying them to important systems.

Sensitive usernames, device identifiers, paths, and other personal details should be anonymized before publishing screenshots or diagnostic material.
