# CentOS Stream 9 VM Black Screen Recovery After Interrupted System Update

## Problem Statement

A CentOS Stream 9 virtual machine running inside VirtualBox failed to load the desktop environment after an interrupted system update.

After booting:
- Black screen appeared
- Top panel and clock were visible
- Mouse cursor worked
- Applications and desktop icons did not load

The operating system kernel was functional, but the GNOME desktop session was partially broken.

---

# Environment

| Component           | Details         |
|---------------------|-----------------|
| Host OS             | Windows         |
| Virtualization      | VirtualBox      | 
| Guest OS            | CentOS Stream 9 |
| Desktop Environment | GNOME           |

---

# What Happened

## Stage 1 — Power Failure

The physical system lost power unexpectedly while the CentOS VM was running.

At that moment:
- System processes were active
- Files existed in memory (RAM)
- Package operations may have been in progress

---

## Stage 2 — Interrupted System Update

After power restoration:
- The VM booted again
- CentOS attempted to continue package updates

During this process, the VM was manually shut down again before the update completed.

This interrupted package transactions and caused system inconsistency.

---

# Root Cause Analysis

## Main Technical Cause

An incompatible MySQL repository was configured on the system:

```text
mysql80-community
```

The repository contained:
- EL7 packages

But the operating system was:
- CentOS Stream 9 (EL9)

DNF attempted to install packages such as:

```text
mysql-community-server-8.0.xx.el7
```

This created dependency conflicts because EL7 packages required older libraries:

```text
libcrypto.so.10
libssl.so.10
```

These libraries are not available on EL9 systems.

---

## Why the Desktop Environment Broke

The interrupted update caused:
- Partial package installation
- Incomplete dependency resolution
- Broken package synchronization
- Damaged GNOME-related packages

As a result:
- GNOME shell loaded partially
- Desktop session initialization failed

---

# Symptoms Observed

After boot:
- Black screen
- Top bar visible
- Clock visible
- Mouse cursor functional
- No desktop icons
- Applications failed to launch

This indicated:
- Linux kernel was operational
- Display manager partially worked
- GNOME session components were corrupted

---

# Recovery Process

---

## Step 1 — Access TTY Terminal

Because the GUI was unstable, a TTY terminal session was used.

### Keyboard Shortcut

```text
Ctrl + Alt + F3
```

### VirtualBox Shortcut

```text
Right Ctrl + F3
```

### Purpose

TTY mode bypasses the graphical desktop and provides direct shell access.

---

## Step 2 — Clean DNF Cache

```bash
sudo dnf clean all
```

### Purpose

- Remove corrupted metadata
- Remove stale cache
- Force fresh repository synchronization

---

## Step 3 — Synchronize Installed Packages

```bash
sudo dnf distro-sync -y
```

### Purpose

- Compare installed packages with repository versions
- Repair incomplete upgrades
- Restore package consistency

---

## Step 4 — Identify Dependency Errors

DNF reported dependency conflicts related to:

```text
mysql80-community
```

Important finding:
- EL7 repository packages were configured on an EL9 system

This revealed the incompatible repository configuration.

---

## Step 5 — Disable Incompatible Repository

The problematic repository was disabled.

### Reason

DNF could not resolve dependencies while incompatible EL7 packages remained enabled.

Disabling the repository allowed package synchronization to continue correctly.

---

## Step 6 — Perform Full System Repair

```bash
sudo dnf distro-sync -y --allowerasing
```

### Purpose

- Repair broken package dependencies
- Remove conflicting packages
- Complete interrupted transactions
- Restore package database consistency

---

## Step 7 — Reinstall GNOME Components

```bash
sudo dnf reinstall gnome-shell gdm mutter -y
```

### Purpose

- Restore damaged desktop packages
- Reinstall GNOME shell components
- Repair graphical session functionality

---

## Step 8 — Reboot System

After reboot:
- GNOME loaded successfully
- Desktop environment functioned normally
- Applications opened correctly

Issue resolved successfully.

---

# Technical Areas Involved

| Area                  | Issue                                  |
|-----------------------|----------------------------------------|
| Package Manager       | Interrupted DNF transaction            |
| Repository Management | Incompatible MySQL EL7 repository      |
| Dependency Resolution | Missing EL7 libraries                  |
| Desktop Environment   | GNOME session corruption               |
| Linux Recovery        | TTY-based troubleshooting              |
| System Repair         | DNF synchronization and package repair |

--------------

# Verification

The recovery was verified by:
- Successful GNOME desktop startup
- Functional applications
- Stable system boot
- No remaining dependency conflicts

------------

# Important Lessons Learned

## Never Interrupt Package Operations

Avoid interrupting:
- `dnf update`
- `yum update`
- Kernel upgrades
- Package installations

Interrupted package transactions can corrupt system state.

---

## Always Use Safe Shutdown

Recommended shutdown methods:

```bash
sudo shutdown now
```

Or use:
- VirtualBox ACPI Shutdown

Avoid forcefully powering off VMs during updates.

---

## Use VM Snapshots Before Major Changes

Always create snapshots before:
- Jenkins installation
- Docker/Kubernetes setup
- MySQL installation
- System upgrades

### VirtualBox Snapshot Path

```text
Machine → Take Snapshot
```

---

# Real-World Relevance

This type of recovery workflow is common in:
- Linux servers
- Cloud virtual machines
- Production infrastructure
- DevOps environments

The troubleshooting methodology involved:
1. Diagnosing symptoms
2. Isolating the problem
3. Repairing package management
4. Restoring system services
5. Verifying system health

---

# Conclusion

This incident demonstrated practical Linux recovery skills involving:
- DNF troubleshooting
- Repository management
- Dependency conflict resolution
- GNOME desktop recovery
- TTY-based administration
- System repair after interrupted updates

The system was successfully restored without reinstalling the operating system.
