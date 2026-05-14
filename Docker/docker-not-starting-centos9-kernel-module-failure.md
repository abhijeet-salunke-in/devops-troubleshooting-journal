# Docker Not Starting in CentOS Stream 9 VM — Kernel Module Recovery Guide

## Problem Statement

Docker was successfully installed on a CentOS Stream 9 virtual machine, but the Docker service failed to start.

Attempting to start Docker produced:

```text
job for docker.service failed because the control process exited with error code.
See "systemctl status docker.service" and "journalctl -xeu docker.service" for details.
```

The Docker daemon (`dockerd`) crashed during initialization due to missing Linux networking kernel modules.

---

# Environment

| Component         | Details         |
|-------------------|-----------------|
| Host OS           | Windows         |
| Virtualization    | VirtualBox      |
| Guest OS          | CentOS Stream 9 |
| Container Runtime | Docker CE       |
| Package Manager   | DNF             |

---

# Docker Installation Command

Docker was installed using:

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

# Initial Failure

Docker service startup command:

```bash
sudo systemctl enable --now docker
```

Result:

```text
Job for docker.service failed
```

Meaning:
- Docker packages installed successfully
- Docker daemon could not initialize correctly

---

# Initial Diagnosis

## Check Docker Service Status

```bash
sudo systemctl status docker.service
```

---

## View Docker Logs

```bash
sudo journalctl -xeu docker.service
```

The logs indicated service failure but did not clearly expose the root cause.

---

# Advanced Diagnosis Using `dockerd`

To obtain detailed startup errors, Docker daemon was started manually:

```bash
sudo dockerd
```

This revealed the actual issue:

```text
failed to create NAT chain DOCKER
iptables failed
Protocol not supported
```

Additional error:

```text
Unable to initialize Netlink socket
```

---

# Technical Meaning of the Error

Docker requires Linux networking subsystems to:
- Create bridge interfaces
- Configure NAT
- Manage container networking
- Apply firewall rules

Docker attempts to create:

```text
docker0
```

bridge network during startup.

This depends on:
- iptables
- nftables
- Linux kernel networking modules

Because these components failed to initialize, Docker daemon exited immediately.

---

# Root Cause Analysis

The earlier interrupted CentOS system update damaged:

```text
kernel-modules
```

package integrity.

As a result:
- Critical Linux networking modules were missing
- iptables functionality failed
- Firewall subsystem became unstable
- Docker networking initialization failed

---

# Proof of Missing Kernel Modules

Networking module test:

```bash
sudo modprobe ip_tables
```

Result:

```text
FATAL: Module ip_tables not found
```

This confirmed that required kernel networking modules were missing or corrupted.

---

# Recovery Process

---

## Step 1 — Verify Docker Failure

Check Docker service status:

```bash
sudo systemctl status docker.service
```

View detailed logs:

```bash
sudo journalctl -xeu docker.service
```

---

## Step 2 — Run Docker Daemon Manually

```bash
sudo dockerd
```

### Purpose

Manual daemon startup exposes detailed low-level startup failures that are often hidden in systemd logs.

---

## Step 3 — Verify Kernel Networking Modules

```bash
sudo modprobe ip_tables
```

### Failure Output

```text
FATAL: Module ip_tables not found
```

This confirmed broken kernel networking support.

---

## Step 4 — Reinstall Kernel Packages

Repair damaged kernel package state:

```bash
sudo dnf reinstall kernel-modules kernel-core kernel -y
```

### Purpose

- Restore missing networking modules
- Repair corrupted kernel package installation
- Rebuild module availability

---

## Step 5 — Reboot System

```bash
sudo reboot
```

### Why Reboot Is Required

New kernel modules are properly loaded during system boot.

---

## Step 6 — Verify Module Recovery

After reboot:

```bash
sudo modprobe ip_tables
```

If no output appears, the module loaded successfully.

This indicates:
- Networking modules restored
- iptables subsystem functional

---

## Step 7 — Start Docker

```bash
sudo systemctl start docker
```

Verify service status:

```bash
sudo systemctl status docker
```

Expected result:

```text
active (running)
```

Docker daemon successfully started.

---

# Technical Root Cause Summary

| Component          | Problem                                |
|--------------------|----------------------------------------|
| Interrupted update | Corrupted kernel package state         |
| kernel-modules     | Missing networking modules             |
| iptables/nftables  | Failed initialization                  |
| firewalld          | Broken networking stack                |
| Docker daemon      | Unable to create NAT/bridge networking |

---

# Why Docker Depends on iptables

Docker networking workflow:

1. Start Docker daemon
2. Create `docker0` bridge
3. Configure NAT
4. Create firewall rules
5. Enable container communication

Docker uses:

```text
iptables
```

to create:

```text
DOCKER NAT chains
```

Without kernel networking modules:
- NAT initialization fails
- Docker networking cannot start
- Docker daemon exits immediately

---

# Important Recovery Commands

## Check Docker Status

```bash
sudo systemctl status docker
```

---

## View Detailed Docker Error

```bash
sudo dockerd
```

---

## Test Networking Module

```bash
sudo modprobe ip_tables
```

---

## Repair Kernel Packages

```bash
sudo dnf reinstall kernel-modules kernel-core kernel -y
```

---

## Reboot System

```bash
sudo reboot
```

---

## Start Docker

```bash
sudo systemctl start docker
```

---

# Verification

The issue was considered resolved after:
- `ip_tables` module loaded successfully
- Docker daemon started normally
- Docker service status showed `active (running)`
- Container networking initialized correctly

---

# Prevention Tips

## Never Interrupt

Avoid interrupting:
- `dnf update`
- Kernel upgrades
- Docker installation
- Package installation processes

Interrupted package transactions can damage kernel and networking subsystems.

---

## Always Take VM Snapshots

Before:
- Docker installation
- Kubernetes setup
- System upgrades
- Kernel updates

### VirtualBox Snapshot Path

```text
Machine → Take Snapshot
```

---

## Use Safe Shutdown Procedures

Recommended:

```bash
sudo shutdown now
```

Or:
- VirtualBox ACPI Shutdown

Avoid forcefully powering off virtual machines.

---

# DevOps and Linux Concepts Learned

| Topic                    | Skill Learned                  |
|--------------------------|--------------------------------|
| systemd                  | Service diagnosis              |
| Docker daemon            | Manual debugging               |
| Linux kernel modules     | Module verification/loading    |
| iptables/nftables        | Container networking           |
| DNF/RPM recovery         | Repairing package corruption   |
| Linux networking         | NAT and bridge troubleshooting |
| Virtual machine recovery | Interrupted update repair      |

---

# Real-World Relevance

This troubleshooting workflow closely matches real Linux infrastructure recovery procedures used by:
- System administrators
- DevOps engineers
- Platform engineers
- Cloud infrastructure teams

The recovery process involved:
1. Diagnosing service failure
2. Identifying hidden daemon errors
3. Verifying kernel functionality
4. Repairing damaged packages
5. Restoring networking stack integrity
6. Validating service recovery

---

# Conclusion

The Docker startup failure was caused by corrupted Linux kernel networking modules after an interrupted CentOS update.

The issue was successfully resolved by:
- Diagnosing the real daemon error
- Verifying missing kernel modules
- Reinstalling kernel packages
- Restoring networking subsystem functionality
- Restarting Docker services

This recovery demonstrated practical Linux infrastructure troubleshooting and Docker networking diagnostics.
