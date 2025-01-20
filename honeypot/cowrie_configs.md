# Cowrie Configuration Guide

This guide outlines the steps to customize your Cowrie honeypot, making it harder for attackers to detect as a honeypot to potentially enhance the quality of collected log data.

## 1. Change the Default Username

### Edit `/etc/passwd` in the Honeypot Filesystem
While in the `(cowrie-env)` environment:
```bash
sudo su cowrie
cd ~/cowrie
source cowrie-env/bin/activate
pwd
```
/home/cowrie/cowrie

Edit the fake `/etc/passwd` file to change the default username
```bash
vi honeyfs/etc/passwd
```
Locate the following line:
```plaintext
phil:x:1000:1000:Phil California,,,:/home/phil:/bin/bash
```
Change `phil` to a new username (e.g., `NEW_NAME`):
```plaintext
NEW_NAME:x:1000:1000:NEW_NAME,,,:/home/NEW_NAME:/bin/bash
```

### Edit the Fake Filesystem
Modify the honeypot's fake filesystem to reflect the new username:
```bash
# Use fsctl to access and modify the fake filesystem
bin/fsctl share/cowrie/fs.pickle

# Rename the default user directory
fs.pickle:/$ mv /home/phil /home/NEW_NAME
```
Exit `fsctl` with `CTRL+D`.

Restart Cowrie to apply the changes:
```bash
bin/cowrie restart
```

---

## 2. Customize `cowrie.cfg`

The `cowrie.cfg` file allows you to manipulate various aspects of the honeypot’s behavior.

### Edit the Configuration File
Navigate to the configuration directory and open the file:
```bash
pwd
/home/cowrie/cowrie
cd etc/
vi cowrie.cfg
```

### Key Parameters to Update

#### 1. **Hostname (output of `uname -n`)**
Set the hostname to mimic an AWS Lightsail server (172.31.0.0/16) or a generic private NAT IP:
```ini
hostname = ip-172-31-30-30
```

#### 2. **Kernel Version (output of `uname -r`)**
Set a realistic kernel version based on your server’s operating system:
```ini
kernel_version = 5.4.0-1018-aws
```
Examples:
- **AWS Lightsail Ubuntu 22**: `5.15.0-1030-aws`
- **AWS Lightsail Debian 11**: `5.10.0-17-cloud-amd64`

#### 3. **Kernel Build String (output of `uname -v`)**
Configure the kernel build string to match the fake kernel version:
```ini
kernel_build_string = #18-Ubuntu SMP Wed Jun 24 01:15:00 UTC 2020
```
Examples:
- **AWS Lightsail Ubuntu 22**: `#34-Ubuntu SMP Mon Jan 23 20:13:32 UTC 2023`
- **AWS Lightsail Debian 11**: `#1 SMP Debian 5.10.136-1 (2022-08-13)`

#### 4. **Hardware Platform (output of `uname -m`)** *(Optional)*
Update the hardware platform to ensure consistency with other configurations:
```ini
hardware_platform = x86_64
```

#### 5. **Sensor Name** *(Optional)*
Set a sensor name for internal identification:
```ini
sensor_name = puddingCup
```

#### 6. **Fake IP Address** *(Optional)*
Configure a fake IP address for incoming connections:
```ini
fake_addr = 172.31.25.140
```

---

## 3. Restart Cowrie
After making all changes, restart Cowrie to apply the updated configurations:
```bash
sudo su cowrie
cd /home/cowrie/cowrie
source cowrie-env/bin/activate
bin/cowrie restart
```

These changes help avoid being immediately identified as a default Cowrie honeypot.