# Jenkins Service Failed to Start Due to Java Version Mismatch

## Problem Statement

Jenkins service failed to start on a CentOS Stream 9 system.

Attempting to start Jenkins produced:

```text
Job for jenkins.service failed because the control process exited with error code.
See "systemctl status jenkins.service" and "journalctl -xeu jenkins.service" for details.
```

The issue was caused by an unsupported Java runtime version.

---

# Environment

| Component              | Details         |
|------------------------|-----------------|
| Operating System       | CentOS Stream 9 |
| Service                | Jenkins         |
| Service Manager        | systemd         |
| Installed Java Version | Java 17         |
| Required Java Version  | Java 21         |

---

# Initial Service Failure

Jenkins startup command:

```bash
sudo systemctl start jenkins
```

Result:

```text
Job for jenkins.service failed because the control process exited with error code.
```

This indicated:
- Jenkins startup process initiated
- Jenkins application terminated during runtime initialization

---

# Diagnostic Process

---

## Step 1 — Check Jenkins Service Status

```bash
sudo systemctl status jenkins.service
```

### Purpose

This command helps identify:
- Service state
- Exit codes
- Restart attempts
- Main process status

### Observed Output

```text
status=1/FAILURE
```

Meaning:
- Jenkins exited with an application-level failure

---

## Step 2 — Review Journal Logs

```bash
sudo journalctl -xeu jenkins.service
```

### Purpose

Used to inspect:
- Service startup logs
- Runtime errors
- Configuration issues

### Observation

Logs only displayed generic service failure messages and did not reveal the exact root cause.

---

## Step 3 — Run Jenkins Manually

Jenkins executable was started directly:

```bash
sudo /usr/bin/jenkins
```

### Why This Step Is Important

Running applications manually often exposes:
- Runtime dependency errors
- Java compatibility issues
- Plugin failures
- Port conflicts
- Permission problems

This is one of the most effective Linux troubleshooting techniques.

---

# Root Cause Found

Manual startup displayed:

```text
Running with Java 17 ...
older than the minimum required version (Java 21)
```

Additional output:

```text
Supported Java versions are: [21, 25]
```

This confirmed:

```text
Installed Java version was unsupported by the installed Jenkins release.
```

---

# Why Jenkins Failed

Modern Jenkins versions require:
- Java 21 or newer

The system contained:
- Java 17

During startup:
1. Jenkins checked Java runtime version
2. Detected unsupported Java release
3. Terminated initialization immediately

As a result:

```text
jenkins.service failed
```

---

# Recovery Process

---

## Step 1 — Install Java 21

```bash
sudo dnf install java-21-openjdk java-21-openjdk-devel -y
```

### Purpose

- Install Jenkins-supported Java runtime
- Install Java development tools

---

## Step 2 — Configure Default Java Version

```bash
sudo alternatives --config java
```

Select:

```text
java-21-openjdk
```

### Purpose

Set Java 21 as the system default Java runtime.

---

## Step 3 — Verify Active Java Version

```bash
java -version
```

### Expected Output

```text
openjdk version "21..."
```

This confirms:
- Correct Java runtime is active
- Jenkins compatibility requirements are satisfied

---

## Step 4 — Start Jenkins Service

```bash
sudo systemctl start jenkins
```

Verify service status:

```bash
sudo systemctl status jenkins
```

### Expected Result

```text
active (running)
```

Jenkins started successfully.

---

# Complete Jenkins Troubleshooting Workflow

If Jenkins fails again, follow this process.

---

## Check Service Status

```bash
sudo systemctl status jenkins.service
```

---

## Check Journal Logs

```bash
sudo journalctl -xeu jenkins.service
```

---

## Run Jenkins Manually

```bash
sudo /usr/bin/jenkins
```

This frequently reveals:
- Java errors
- Port conflicts
- Plugin corruption
- Memory problems
- Permission issues

---

## Verify Java Version

```bash
java -version
```

Ensure the installed Java version is supported by the Jenkins release.

---

## Verify Port Availability

```bash
sudo ss -tulnp | grep 8080
```

### Purpose

Check whether another service is already using port 8080.

---

## Restart Jenkins

```bash
sudo systemctl restart jenkins
```

---

## Monitor Live Logs

```bash
sudo journalctl -fu jenkins
```

Useful for:
- Real-time monitoring
- Observing startup behavior
- Detecting recurring failures

---

# Common Jenkins Startup Failure Causes

| Problem                  | Symptoms                  |
|--------------------------|---------------------------|
| Wrong Java version       | Immediate startup failure |
| Port 8080 already in use | Bind/address errors       |
| Corrupted plugins        | Startup crashes           |
| Low memory               | JVM termination           |
| Permission issues        | Access denied errors      |
| Broken Jenkins config    | Initialization failure    |
| Firewall restrictions    | Web UI inaccessible       |

---

# Technical Concepts Learned

| Area                    | Skill                      |
|-------------------------|----------------------------|
| systemd                 | Service management         |
| journalctl              | Log analysis               |
| Java runtime management | alternatives configuration |
| Jenkins runtime         | Manual startup debugging   |
| Network diagnostics     | Port verification          |
| Linux troubleshooting   | Root cause analysis        |

---

# Verification

The issue was resolved after:
- Installing Java 21
- Setting Java 21 as default runtime
- Restarting Jenkins successfully
- Verifying service health using `systemctl`

---

# Real-World Relevance

This troubleshooting process closely matches real-world Linux and DevOps operational workflows:

1. Diagnose service failure
2. Review system logs
3. Execute application manually
4. Identify runtime dependency issue
5. Repair root cause
6. Validate application recovery

This methodology is commonly used by:
- DevOps engineers
- Platform engineers
- Linux system administrators
- CI/CD infrastructure teams

---

# Conclusion

Jenkins failed because the installed release required Java 21, while the system was running Java 17.

The issue was successfully resolved by:
- Installing Java 21
- Updating the system default Java runtime
- Restarting Jenkins services
- Verifying successful service initialization

This recovery demonstrated practical Jenkins runtime troubleshooting and Java dependency management in a Linux environment.
