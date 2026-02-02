---
layout: default
title: Learning Active Directory
nav_order: 5
---

# **Admin First, Attack Later!**

A recent discussion in the HackTheBox Discord highlighted a common frustration: the "Introduction to Active Directory" module can feel like an overwhelming information dump. Definitions and concepts that seem abstract when you've never actually worked with AD rarely stick in any meaningful way. While the "Active Directory Enumeration & Attacks" module attempts to bridge this gap, many students find themselves struggling as they go deeper, running commands without truly understanding the relationships they're exploiting or why these attacks even work.

The problem is, if you don't understand the underlying system, you're going to have a bad time.

In environments like HTB Academy, it's easy to learn AD backwards. You jump straight into offensive techniques, memorise attack commands, take notes on tool syntax, and then hit a wall when environments change or defenses block your standard approach. You can't adapt because you never built the foundation.

Thankfully, theres a better path: learn to administer AD first, then attack it.

This perspective comes from someone with a background in administering Active Directory and then securing it. When you understand the administrative side, everything clicks into place. You can read an attack chain like a story, understanding each step and why it works. In live incidents, you can anticipate the threat actor's next move because you understand the system they're exploiting and what defensive actions will actually matter.

This post will cover:
- Why the admin-first approach makes you a better attacker
- Resources for learning AD administration (not pentesting)
- Practical examples of what to build and practice in your lab
- How to connect administrative knowledge with offensive techniques
- Why this investment pays off long-term

## Why Learn Administration First?

This may have already happened to you: you're following a writeup and see that you have a user with GenericWrite on a computer object, so you configure Resource Based Constrained Delegation and impersonate a domain admin, but you don't really know what half of that is. Without admin knowledge, it's probably complete gibberish, so you've copied the commands, watched them work, then moved on without really learning anything transferrable. With admin knowledge you understand that GenericWrite is an ACL permission that should not exist on a computer object in a secure environment, you also understand that RBCD is a legitimate feature designed for service to service auth scenarios. The attack works because AD trusts the delegation configuration you just set, and the system cannot distinguish between legitimate admin config and malicious abuse.

When you understand AD administration, you'll develop an intuition for what is normal and what is misconfigured, you'll be able to spot these naturally, and be able to adapt techniques to environments instead of just trying different tools and commands. You'll be able to enumerate better, working past BloodHound and deeper into ACLs and ACEs with manual enumeration, identifying high value targets, and knowing what questions to ask instead of blindly running tools. This overall will help you evade detection, working alongside detection and prevention tools unimpeded, and the most imporant thing for the customer, aid in your report writing, where you can effectively communicate the business impact of their misconfigurations, and give realistic and actionable remediaton advice.

## "But I Just Want To Hack Things"

I know, I know, thats fair enough, and you absolutely can learn attack techniques and pass exams like OSCP, CPTS, and CAPE without doing all these (you may struggle on CAPE though), but you'll face some struggles. First off, you won't stand out from the other people going for the job, since homelabs, especially documented ones, are often valuable in the recruitment process, and second, complex environments will confuse you as you realise you dont understand the technologies running in the environment. This is all to your benefit, and it'll even include some hacking.

Overall, once this is set up, its there to tinker around with whenever you want, and it isn't a static environment, you can add or remove bits, change configurations, and completely revamp it to whatever you want to practice.

## Resources for Learning AD Administration

The goal here isn't to become a certified administrator, all you want to do is build mental models for offensive work. You want to understand why organisations use these features.

In addition to the below, if you prefer books, you can do that, I don't really have any to recommend besides [Windows Security Internals](https://nostarch.com/windows-security-internals) from No Starch Press, it's a little more on the advanced side but if you're serious about attacking or defending AD its a great read.

### Microsoft Learn (Start Here)

Microsoft Learn is the official documentation and teaches you the *intended* use cases. Understanding what AD is supposed to makes it easier to spot when it can be or is being abused. Below are some good starter resources:
- [Windows Server Administration Fundamentals](https://learn.microsoft.com/en-us/shows/windows-server-administration-fundamentals/)
- [Active Directory Domain Services overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [Group Policy overview for Windows Server](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview)

You don't need to memorise and learn everything, skim them and get an idea of the basics, knowing how to read and use the MS Learn documentation is key here.



### YouTube

There are countless tutorials for AD on YouTube, if you're a visual learner this is great, and if you find any ambiguity in written instruction, this can bridge the gap.

This '[Active Directory Masterclass](https://www.youtube.com/watch?v=trkzBz4eaQo&list=PLt7zzBn5CwKpPzfEXeTcr6ZC2cqqmfIK3&index=1)' playlist covers the initial setup of an AD environment, which will come in handy later, [this video](https://www.youtube.com/watch?v=BqAfgquDcj0) explains Group Policy, definitely a requirement to understand, and the users channel all over is great for AD. Further to this, if you want to learn about something just search it, Kerberos auth, AD Delegation, any topic, you'll find a video on it.

### Hands-On Practice

Reading and watching will only get you so far, you'll need to get your hands dirty building your own lab, this is the fun part.

You want a lab that at a minimum consists of the following
- Domain Controller
- 2 Client Workstations
- Linux attack box
Any modern version of Windows is fine, a lot of environments haven't moved over to Server 2025 yet but you'll want to move ahead of the curve.

**Platform options**:
**VirtualBox/VMware Workstation** (Free, local)
Pros: Completely free, works offline, full control
Cons: Resource-heavy (need decent RAM), manual setup
Best for: Most people starting out
**Proxmox** (Free, requires hardware)
Pros: Professional-grade, infrastructure as code, snapshots
Cons: Steeper learning curve, needs dedicated machine
Best for: If you have spare hardware and want reproducible builds
**Cloud (Azure/AWS)** (Costs money)
Pros: No local resources needed, scales easily, realistic networking
Cons: Costs add up quickly, requires cloud platform knowledge
Best for: If you have credits or want cloud experience

These are not a replacement for things like GOAD, VulnLab, or HTB labs, you can work concurrently with these.

### Setting up the Lab

What do you want to do? Where do you want to start? Well, your basic domain setup should be your server as your domain controller, you promote it to domain controller, and then you do the rest of the things that you need to do that you are researching, right? Well, here it is anyway

**Set Up Your Domain**

Basic domain setup:
Install Windows Server as your Domain Controller.
Configure static IP and DNS.
Promote to Domain Controller (creates your forest and domain).
Choose a domain name (e.g., homelab.local).
Set DSRM password (and understand why it matters offensively later).

Join your workstations:
Configure DNS to point to your DC.
Join the domain.
Verify domain join.
Log in with domain credentials.

Pro tip: take snapshots at each major step so you can roll back cleanly.

**Create an Organisational Structure**

Don't just dump everything in the default containers, create an OU structure that would mirror a real organisation, lay it out with Web and Database in the Server category, IT, HR, Sales in the Workstations and Users categories. This matters because GPO scope will follow OU structure, understanding inheritance is critical for applying policies and exploiting misconfigurations.

What to practice:
Create the OU structure above
Create 10-15 users distributed across different OUs
Create security groups for each department (IT-Staff, HR-Staff, Sales-Staff)
Add users to their respective groups
Test: Can users log in? Do they appear in the correct OUs?

**Experiment with Group Policy**

This is where things get interesting. GPOs control everything in an AD environment, and they execute with SYSTEM privileges. Create a password policy GPO. create a GPO that maps a network drive, link it to the Users OU, does it appear when users log in? Create a GPO that blocks executables from certain paths, test it out. Create a conflicting GPO, check which one wins. Break a GPO, troubleshoot it. Link the same GPO to multiple OUs, test it.

**Service Accounts and SPNs**

Service accounts are everywhere in AD, and they're frequently misconfigured. Create a service account, create a service to use it, configure the service to run as the service account and check it works. Now create multiple service accounts, some with elevated privileges, weak passwords on some, strong on others, roast them if you're ready.

**User and Group Management**

Create a GPO linked to the Workstations OU, *Computer Configuration → Preferences → Control Panel Settings → Local Users and Groups*, Add "Workstation-LocalAdmins" to local Administrators group, Members of IT-Admins should now be local admin on workstations, test it. Nested groups create complex attack paths, you'll start to learn inherited privileges.

**Authentication**

Force Kerberos authentication, see how much harder the system is to attack but also this may start to cause issues in your environment, figure out why? Attack it with Rubeus.

**Delegation Scenarios**

Delegation is a complex topic that is often misconfigured, start to throw it into the mix. Configure unconstrained delegation on a workstation, constrained delegation, RBCD.

**Misconfigurations**

Heres the real fun stuff. Create accounts with weak passwords, without pre-authentication set, overprivileged group membership, give users random privileges, see how to exploit them. You'll want to do things like nest the wrong groups, let people write to logonscripts, give GenericAll on objects, let people edit GPOs.

### Defensive Measures

If you like, this is when you can start adding in defensive measures, like enabling logging, command line process auditing, and testing it, see what comes up. Enable Defender and start running your attacks again, throw mimikatz on a machine? Is there a way around it? Have a google.

You can mix and match here, like adding in Attack Surface Reduction rules, defender exclusions, whatever you want.

## Beyond

There's so much more to do here, so here some ideas:

- Add a second domain: create a child domain or a seperate forest with trust relationships. Start to understand transitive and non-transitive trusts, and you'll understand how compromising one domain can lead to controlling the entire forest.
- Implement Certificate Services: Install ADCS on your DC or a different server, configure certificate templates and introduce vulnerable certificates. You'll learn why certificate-based auth exists, and what makes these templates vulnerable, and how they are abused for escalation and persistence.
- Add LAPS (Local Administrator Password Solution): Implement LAPS to automatically manage local admin passwords, configure which OUs have LAPS applied, set password complexity and rotation schedules. This teaches you how organizations mitigate pass-the-hash attacks, but also shows you what happens when LAPS permissions are misconfigured.
- Introduce Microsoft Defender for Endpoint (MDE): Sign up for an M365 trial and onboard your lab machines to Defender for Endpoint. This exposes you to EDR detection, you'll see exactly what triggers alerts, what behavioural detections look like, and how attack chains are correlated. Practice your attacks against real defensive tools and learn what's actually stealthy versus what immediately raises flags.

Really, there isn't a limit to what you can do, start mixing and matching, implement logging, honeypots, forest trusts, conditional access, you don't right now want to read about how much could be written about so go and get stuck in!