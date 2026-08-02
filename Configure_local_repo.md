
To install **`oracle-database-preinstall-19c`**, **`oracleasm-support`**, and **`oracleasmlib`** on an offline Oracle Linux 8.10 system with UEK (Kernel 5.15 / UEK R7), follow this multi-step guide.

> **Important Offline Note:**
> * **`oracle-database-preinstall-19c`** and its dependencies are included directly on the Oracle Linux 8 ISO.
> * **`oracleasmlib`** and **`oracleasm-support`** are **not included** on the standard Oracle Linux ISO media. You must download these two RPM files from an internet-connected machine and transfer them to your offline server (e.g., via USB or SFTP).
> * **Kernel Driver:** On UEK 5.15 (UEK R7), the `oracleasm` kernel module is already built directly into the kernel, so no separate `kmod-oracleasm` driver package is required.
> 
> 

---

## Step 1: Transfer Required External ASMLib Packages

On an internet-connected host, download the matching ASMLib packages for Oracle Linux 8 (`x86_64`) from the [Oracle Technology Network](https://www.oracle.com/linux/downloads/linux-asmlib-v8-downloads.html) or [pkgs.org](https://pkgs.org/search/?q=oracleasm-support):

1. `oracleasmlib-3.0.x` or `oracleasmlib-3.1.x` RPM
2. `oracleasm-support` RPM

Transfer these RPM files to a directory on your target system (e.g., `/tmp/oracleasm-packages/`).

---

## Step 2: Mount the Oracle Linux 8.10 ISO

1. Create a mount directory:
```bash
sudo mkdir -p /mnt/ol8iso

```


2. Mount your OL8.10 ISO image (replace `/path/to/ol8.iso` with your actual ISO file path or `/dev/cdrom`):
```bash
sudo mount -o loop,ro /path/to/OracleLinux-R8-U10-x86_64-dvd.iso /mnt/ol8iso

```


3. *(Optional)* Make the mount persistent across reboots by editing `/etc/fstab`:
```bash
echo "/path/to/OracleLinux-R8-U10-x86_64-dvd.iso /mnt/ol8iso iso9660 ro,defaults,nofail 0 0" | sudo tee -a /etc/fstab

```



---

## Step 3: Configure the Local Yum/DNF Repository

1. Backup existing online repository files so `dnf` doesn't try to connect to the internet:
```bash
sudo mkdir /etc/yum.repos.d/backup
sudo mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/backup/

```


2. Create a new repository file pointing to the ISO's `BaseOS` and `AppStream` directories:
```bash
sudo cat << 'EOF' | sudo tee /etc/yum.repos.d/ol8-local.repo
[ol8_baseos_local]
name=Oracle Linux 8 BaseOS Local ISO
baseurl=file:///mnt/ol8iso/BaseOS
gpgcheck=0
enabled=1

[ol8_appstream_local]
name=Oracle Linux 8 AppStream Local ISO
baseurl=file:///mnt/ol8iso/AppStream
gpgcheck=0
enabled=1
EOF

```


3. Clean the repository cache and verify the local repos are active:
```bash
sudo dnf clean all
sudo dnf repolist

```



---

## Step 4: Install `oracle-database-preinstall-19c`

Now install the preinstall package directly using `dnf` from your local ISO repository:

```bash
sudo dnf install -y oracle-database-preinstall-19c

```

*This command will automatically resolve and install all OS kernel parameters, limits, users (`oracle`), groups (`oinstall`, `dba`), and required pre-installation packages from the ISO.*

---

## Step 5: Install `oracleasm-support` and `oracleasmlib`

Navigate to the directory where you transferred the ASMLib RPM files in Step 1 and install them via `dnf` so local dependencies (like `kmod` or system utilities) are resolved from your mounted ISO repository:

```bash
cd /tmp/oracleasm-packages/

sudo dnf install -y oracleasm-support-*.rpm oracleasmlib-*.rpm

```

---

## Step 6: Configure and Verify Oracle ASMLib

1. Initialize the ASMLib module configuration:
```bash
sudo oracleasm configure -i

```


*Follow the interactive prompts to specify the default user (`oracle`), group (`oinstall`), start on boot (`y`), and scan for disks on boot (`y`).*
2. Load the kernel driver and initialize the ASMLib service:
```bash
sudo oracleasm init

```


3. Confirm ASMLib status:
```bash
sudo oracleasm status

```


*Expected Output:*
```text
Checking Health of Oracle ASM Library driver: OK
Checking status of Oracle ASMLib filesystem: mounted

```
