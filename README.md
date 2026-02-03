                                                                        Born2beRoot :
*This project has been created as part of the 42 curriculum by yelallam.*
Born2beRoot is a system administration project that introduces the fundamentals of virtualization and server configuration. The goal is to create and configure a virtual machine from scratch, implementing strict security rules and system administration best practices.

This project provides hands-on experience with:
- Virtual machine setup and management
- Linux system administration
- Security hardening (firewall configuration, SSH hardening, password policies)
- User and group management
- Service configuration and monitoring
- Bash scripting for system monitoring

By the end of this project, you will have set up a fully functional, secure server environment following industry-standard practices.

## Instructions

### Prerequisites
- VirtualBox (or UTM for Mac M1 users)
- ISO image of Debian (latest stable) or Rocky Linux (latest stable)
- Minimum 8GB disk space for the virtual machine/but preferabally 30.80 gb if you are willing to do the bonus part [the partions]

### Installation Steps

1. Create the Virtual Machine**
   - Open VirtualBox/UTM
   - Create a new VM with appropriate specifications
   - Attach the Debian/Rocky ISO image

2. Install the Operating System**
   - Follow the installation wizard
   - Configure encrypted LVM partitions during installation
   - Set up at least 2 encrypted partitions

3.Initial System Configuration**
   ```bash
   # Update system packages
   sudo apt update && sudo apt upgrade -y  # For Debian
   # or
   sudo dnf update -y  # For Rocky
   ```

4. Configure Hostname**
   ```bash
   sudo hostnamectl set-hostname your_login42
   ```

5. Install and Configure SSH**
   ```bash
   sudo apt install openssh-server -y  # For Debian
   sudo systemctl enable ssh
   sudo systemctl start ssh
   ```
   - Edit `/etc/ssh/sshd_config` to change port to 4242
   - Disable root login via SSH

6. Configure Firewall**
   ```bash
   # For Debian (UFW)
   sudo apt install ufw -y
   sudo ufw enable
   sudo ufw allow 4242
   
   # For Rocky (firewalld)
   sudo systemctl enable firewalld
   sudo systemctl start firewalld
   sudo firewall-cmd --permanent --add-port=4242/tcp
   sudo firewall-cmd --reload
   ```

7. Set Up Password Policy**
   - Install libpam-pwquality: `sudo apt install libpam-pwquality`
   - Configure `/etc/login.defs` for password aging
   - Configure `/etc/security/pwquality.conf` for password complexity

8. Configure Sudo**
   - Edit sudoers configuration: `sudo visudo`
   - Create custom sudo log directory: `sudo mkdir -p /var/log/sudo`
   - Configure sudo settings in `/etc/sudoers.d/sudo_config`

9. Create User and Groups**
   ```bash
   sudo adduser your_login
   sudo addgroup user42
   sudo usermod -aG user42,sudo your_login
   ```

10. Set Up Monitoring Script**
    - Create `/usr/local/bin/monitoring.sh`
    - Make it executable: `sudo chmod +x /usr/local/bin/monitoring.sh`
    - Configure cron job: `sudo crontab -e`
    - Add: `*/10 * * * * /usr/local/bin/monitoring.sh`

### Running the Project

1.  Start the Virtual Machine**
   - Launch VirtualBox/UTM
   - Start your configured VM

2. Verify Configuration**
   - Check SSH service: `sudo systemctl status ssh`
   - Check firewall status: `sudo ufw status` or `sudo firewall-cmd --list-all`
   - Verify monitoring script runs every 10 minutes

3. Connect via SSH**
   ```bash
   ssh your_login@localhost -p 4242
   ```

### Getting the Signature

To generate the signature for submission:

For VirtualBox users:
```bash
# Windows
certUtil -hashfile path\to\your_vm.vdi sha1

# Linux
sha1sum ~/VirtualBox\ VMs/your_vm/your_vm.vdi

# MacOS
shasum ~/VirtualBox\ VMs/your_vm/your_vm.vdi
```

**For UTM users (Mac M1):**
```bash
shasum ~/Library/Containers/com.utmapp.UTM/Data/Documents/your_vm.utm/Images/disk-0.qcow2

Save the output to `signature.txt` in your Git repository.

## Resources

[Leave this section for your own resources]

## Project Description

### Operating System Choice: Debian

Why Debian?
- Pros:
  - Highly stable and reliable
  - Excellent documentation and community support
  - Recommended for beginners in system administration
  - Large repository of pre-compiled packages
  - Long-term support (LTS) versions available
  - Lower learning curve compared to Rocky Linux

- Cons:
  - Packages may be slightly older due to stability focus
  - Less enterprise-oriented than Rocky Linux
  - Smaller corporate backing compared to RHEL-based distributions

Why Not Rocky Linux?
- Pros:
  - Enterprise-grade distribution (RHEL clone)
  - Strong security features (SELinux)
  - Excellent for learning enterprise Linux environments
  - Compatible with RHEL ecosystem

- Cons:
  - More complex setup process
  - Steeper learning curve for beginners
  - SELinux configuration can be challenging
  - KDump setup adds unnecessary complexity for this project

### Design Choices

#### Partitioning Scheme
The project uses LVM (Logical Volume Manager) with encryption for flexible and secure storage management:

- /boot- Unencrypted boot partition (500MB)
- Encrypted LVM containing:
  - root (/)** - System files (10GB)
  - swap - Memory overflow (1GB)
  - /home - User data (5GB)
  - /var- Variable data and logs (3GB)
  - **/tmp** - Temporary files (1GB)

Benefits:
- Enhanced security through full-disk encryption
- Flexible volume resizing
- Snapshot capabilities
- Better organization of system resources

#### Security Policies

Password Policy:
- 30-day expiration period
- Minimum 2 days between password changes
- 7-day warning before expiration
- Minimum 10 characters with complexity requirements
- Protection against dictionary attacks

Sudo Configuration:
- Limited to 3 authentication attempts
- Custom error messages for failed attempts
- Comprehensive logging of all sudo commands
- TTY mode enabled for enhanced security
- Restricted PATH for sudo commands

Firewall Configuration:
- Only port 4242 open for SSH
- Default deny policy for all other incoming connections
- Protection against unauthorized access

SSH Hardening:
- Non-standard port (4242) to reduce automated attacks
- Root login disabled
- Password authentication (key-based authentication recommended for production)

#### User Management
- Separate user and root accounts
- Group-based access control (user42, sudo)
- Principle of least privilege applied

#### Services Installed
- OpenSSH Server - Secure remote access
- UFW/firewalld - Firewall management
- sudo - Privilege escalation
- libpam-pwquality - Password quality enforcement
- cron - Task scheduling for monitoring

### Technical Comparisons

#### Debian vs Rocky Linux

Aspect | Debian | Rocky Linux |
-------|--------|-------------|
Base | Independent | RHEL clone |
Package Manager | apt/dpkg | dnf/rpm | Release Cycle | ~2 years | ~6 months |
 Target Audience| General purpose | Enterprise |
 Difficulty| Beginner-friendly | Intermediate |
Security Module| AppArmor | SELinux |
Community | Large, diverse | Growing, enterprise-focused |Documentation | Extensive | Good, RHEL-compatible |
#### AppArmor vs SELinux

 Feature | AppArmor | SELinux |
---------|----------|---------|
 Approach| Path-based | Label-based |
 Complexity| Simpler to configure | More complex, more powerful |
 Default on | Debian/Ubuntu | RHEL/Fedora/Rocky |
 Learning Curve | Gentle | Steep |
 Granularity | File paths | Fine-grained contexts |
 Use Case | Desktop/simple servers | Enterprise/high-security |

AppArmor uses file paths to define security policies, making it more intuitive for beginners. **SELinux** uses security contexts and labels, providing more granular control but requiring deeper understanding.

#### UFW vs firewalld

| Feature | UFW | firewalld |
---------|-----|-----------|
nterface| Command-line focused | CLI + GUI |
Complexity | Simple, straightforward | More features, complex |
Backend | iptables/nftables | nftables |
Default on | Debian/Ubuntu | RHEL/Fedora/Rocky |
Zone Support | No | Yes |
| Dynamic Rules| Limited | Full support |
| Best For | Simple configurations | Complex network setups |

UFW(Uncomplicated Firewall) lives up to its name with simple syntax and easy rule management. **firewalld** offers zones, rich rules, and dynamic configuration but with increased complexity.


 VirtualBox vs UTM

 Feature | VirtualBox | UTM |
---------|------------|-----|
Platform Support| Windows, Linux, macOS (Intel) | macOS (especially M1/M2) |
Architecture| x86/x64 | ARM64 + x86 emulation |
Performance | Native virtualization | Native on ARM, emulated x86 |
User Interface| Traditional GUI | Modern, macOS-native |
Ease of Use| Mature, well-documented | Simpler, more intuitive |
Extension Pack | Available (USB, RDP, etc.) | Built-in features |
Community | Large, established | Growing |
License | GPL (free) + proprietary | GPL (free) |

VirtualBox is the industry standard with extensive features and cross-platform support. **UTM** is the preferred choice for Apple Silicon Macs, offering native ARM virtualization with excellent performance.


## Monitoring Script Overview

The `monitoring.sh` script displays system information every 10 minutes using the `wall` command. It shows:
- Architecture and kernel version
- Physical and virtual CPU count
- Memory usage
- Disk usage
- CPU load
- Last reboot time
- LVM status
- Active TCP connections
- Logged-in users
- Network information (IP and MAC)
- Sudo command count

## Common Commands Reference

```bash
# Check OS information
uname -a
hostnamectl

# User management
sudo adduser username
sudo usermod -aG groupname username
getent group groupname

# Password policy
sudo chage -l username
cat /etc/login.defs
cat /etc/security/pwquality.conf

# Sudo configuration
sudo visudo
cat /etc/sudoers.d/sudo_config
sudo cat /var/log/sudo/sudo.log

# Firewall
sudo ufw status numbered
sudo firewall-cmd --list-all

# SSH
sudo systemctl status ssh
cat /etc/ssh/sshd_config

# Partitions and LVM
lsblk
sudo pvs
sudo vgs
sudo lvs

# Monitoring
sudo crontab -l
cat /usr/local/bin/monitoring.sh
```

Troubleshooting

SSH Connection Refused:
- Check if SSH service is running: `sudo systemctl status ssh`
- Verify firewall allows port 4242: `sudo ufw status`
- Confirm port is 4242 in `/etc/ssh/sshd_config`

Monitoring Script Not Running:
- Check cron service: `sudo systemctl status cron`
- Verify crontab entry: `sudo crontab -l`
- Check script permissions: `ls -l /usr/local/bin/monitoring.sh`

Password Policy Not Working:
- Verify libpam-pwquality is installed
- Check `/etc/security/pwquality.conf` configuration
- Review `/etc/pam.d/common-password`


Resources :
Well Born2beroot is a project rich of knowledge which means we need to get into multiple data and resources to look for the knowledge we need well here is a board that i designed my self along my jouney to serve as
an elaborated resource for this project :  https://miro.com/app/board/uXjVGQ1dGAE=/
