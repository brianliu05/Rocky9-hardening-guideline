Creating a hardening guideline based on the provided Rocky 9 server hardening commands involves detailing each step and explaining its purpose. Below is a comprehensive guideline:

---

# Rocky 9 Server Hardening Guideline

This document outlines the steps to harden a Rocky 9 server using OpenSCAP tools and the Security Content Automation Protocol (SCAP) Security Guide.

## Prerequisites

- Ensure you have administrative (root) access to the server.
- Internet connection to install the necessary packages.

## Step-by-Step Hardening Process

### 📦1. Install Required Packages

Install the OpenSCAP scanner and SCAP security guide:
```bash
yum install openscap-scanner scap-security-guide
```

### 📁2. Navigate to SCAP Content Directory

Change to the directory where SCAP content is stored:
```bash
cd /usr/share/xml/scap/ssg/content
```

### ✅3. Verify SCAP Content

Get information about the SCAP content file:
```bash
oscap info ssg-rl9-ds.xml
oscap info --profile xccdf_org.ssgproject.content_profile_cis_server_l1 ssg-rl9-ds.xml
```

### ⚙️4. Preliminary Configuration Changes

#### Set Timezone
Set the server timezone to Asia/Hong_Kong:
```bash
timedatectl set-timezone Asia/Hong_Kong
```

#### Configure SSH
Restrict SSH access to a specific user (e.g., admin):
```bash
echo "AllowUsers hkmuadmin" >> /etc/ssh/sshd_config
systemctl restart sshd
```

#### Configure Firewall
Set the default firewalld zone to 'drop':
```bash
sed -i 's/DefaultZone=public/DefaultZone=drop/g' /etc/firewalld/firewalld.conf
```
#### Allow SSH through the firewall:
```bash
firewall-cmd --add-service=ssh --permanent
firewall-cmd --zone=drop --add-rich-rule='rule protocol value="icmp" accept' --permanent
sudo systemctl restart firewalld
```

#### Reset Root Password
For security, reset the root password:
```bash
passwd root
```

#### Set UEFI Boot Loader Password
Secure the GRUB2 bootloader with a password:
```bash
grub2-setpassword
```

### 📝 5. Generate Initial Compliance Report

Generate an evaluation report for the CIS Server Level 1 profile:
```bash
oscap xccdf eval --results result.xml --profile xccdf_org.ssgproject.content_profile_cis_server_l1 ssg-rl9-ds.xml
sudo oscap xccdf generate report result.xml > report.html
```
- **result.xml**: Contains the results of the evaluation.
- **report.html**: A human-readable report generated from the results.

### 🔧6. Apply Remediations

Automatically apply remediations based on the CIS Server Level 1 profile:
```bash
sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis_server_l1 --remediate ssg-rl9-ds.xml
```
Reapply firewall settings (they may be reset):

```bash
firewall-cmd --add-service=ssh --permanent
firewall-cmd --zone=drop --add-rich-rule='rule protocol value="icmp" accept' --permanent
sudo systemctl restart firewalld
```
Reboot to apply changes:
```bash
sudo init 6
```
Note: The init 6 command reboots the system. Ensure all work is saved before running.

### 🛠️7. Generate Remediation Script  (Optional)
Generate a Bash script for remediations, which can be used for future use:
```bash
oscap xccdf generate fix --profile xccdf_org.ssgproject.content_profile_cis_server_l1 --fix-type bash --output remediations.sh ssg-rl9-ds.xml
```
### 📊8. Generate Post-Remediation Report

Re-evaluate the system and generate a final compliance report:
```bash
cd /usr/share/xml/scap/ssg/content
sudo oscap xccdf eval --results result.xml --profile xccdf_org.ssgproject.content_profile_cis_server_l1 ssg-rl9-ds.xml
sudo oscap xccdf generate report result.xml > after_report.html
```

## Notes

- **Time Synchronization**: Ensure the server's time is accurately synchronized, as this affects logs and security measures.
- **SSH Access**: Restricting SSH access to specific users enhances security by reducing potential attack vectors.
- **Firewall Configuration**: Setting a restrictive default zone and explicitly allowing necessary services helps prevent unauthorized access.
- **Password Management**: Regularly update and secure root passwords to maintain system security.

---

By following these steps, you can significantly enhance the security posture of your Rocky 9 server. Regular audits and updates to security policies and practices are also recommended to maintain a secure environment.
