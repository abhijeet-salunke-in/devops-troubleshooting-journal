# Jenkins Username and Password Recovery (CentOS 9)

## Problem Statement

Forgot Jenkins username and/or password and was unable to log in to the Jenkins dashboard.

---

## Environment

- Operating System: CentOS 9
- Service: Jenkins
- Port: 8080

---

# Situation 1: Forgot Only Username

## Check Existing Jenkins Users

Run the following command:

```bash
ls /var/lib/jenkins/users
```

### Example Output

```text
admin
abhijeet
devops
```

The displayed directory names represent Jenkins usernames.

---

# Situation 2: Forgot Password

## Safe Jenkins Password Recovery Process

---

## Step 1: Stop Jenkins Service

```bash
sudo systemctl stop jenkins
```

---

## Step 2: Backup Jenkins Configuration

Before modifying the configuration, create a backup.

```bash
sudo cp /var/lib/jenkins/config.xml ~/config-backup.xml
```

---

## Step 3: Open Jenkins Configuration File

```bash
sudo nano /var/lib/jenkins/config.xml
```

---

## Step 4: Temporarily Disable Jenkins Security

Replace the following configuration:

```xml
<authorizationStrategy class="hudson.security.FullControlOnceLoggedInAuthorizationStrategy">
  <denyAnonymousReadAccess>true</denyAnonymousReadAccess>
</authorizationStrategy>

<securityRealm class="hudson.security.HudsonPrivateSecurityRealm">
  <disableSignup>true</disableSignup>
  <enableCaptcha>false</enableCaptcha>
</securityRealm>
```

With:

```xml
<authorizationStrategy class="hudson.security.AuthorizationStrategy$Unsecured"/>
<securityRealm class="hudson.security.SecurityRealm$None"/>
```

---

## Step 5: Start Jenkins Service

```bash
sudo systemctl start jenkins
```

---

## Step 6: Access Jenkins Dashboard

Open Jenkins in a browser:

```text
http://YOUR-IP:8080
```

Jenkins will open without authentication.

---

## Step 7: Create New Admin User

Navigate to:

```text
Manage Jenkins → Security → Users
```

Create:
- New username
- New password

---

## Step 8: Re-enable Jenkins Security (Important)

Edit the Jenkins configuration again:

```bash
sudo nano /var/lib/jenkins/config.xml
```

Replace:

```xml
<authorizationStrategy class="hudson.security.AuthorizationStrategy$Unsecured"/>
<securityRealm class="hudson.security.SecurityRealm$None"/>
```

Back with:

```xml
<authorizationStrategy class="hudson.security.FullControlOnceLoggedInAuthorizationStrategy">
  <denyAnonymousReadAccess>true</denyAnonymousReadAccess>
</authorizationStrategy>

<securityRealm class="hudson.security.HudsonPrivateSecurityRealm">
  <disableSignup>true</disableSignup>
  <enableCaptcha>false</enableCaptcha>
</securityRealm>
```

---

## Step 9: Restart Jenkins

```bash
sudo systemctl restart jenkins
```

---

## Step 10: Verify Login

- Jenkins login page appears normally
- Login using the new credentials

---

# Root Cause

The Jenkins administrator credentials were forgotten, preventing access to the Jenkins dashboard.

---

# Security Warning

Never leave Jenkins configured with:

```xml
<authorizationStrategy class="hudson.security.AuthorizationStrategy$Unsecured"/>
<securityRealm class="hudson.security.SecurityRealm$None"/>
```

This disables authentication completely and allows unrestricted admin access.

---

# Best Practices

Store credentials securely using:
- Password managers
  - Bitwarden
  - KeePass
- Secure credential vaults
- Encrypted backup files

Example:

```bash
sudo nano ~/jenkins-credentials.txt
```

---

# Verification

The issue was successfully resolved after:
- Jenkins security was temporarily disabled
- A new admin account was created
- Security settings were restored
- Login was verified with new credentials

---

# Lessons Learned

- Always maintain backup administrator credentials
- Take backups before editing Jenkins configuration files
- Never leave Jenkins unsecured after recovery
- Document recovery procedures for future incidents
- Secure credential management is essential in DevOps environments

---

# Conclusion

This troubleshooting process provides a safe and controlled method to recover Jenkins access without reinstalling Jenkins or losing configuration data.
