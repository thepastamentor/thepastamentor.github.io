---
layout: default
title: CAPE Prep Guide
nav_order: 4
---

# **CAPE Preparation Guide**

This guide serves as an unofficial preparation path for the CAPE (Certified Active Directory Pentesting Expert) exam. At the time of writing, no official preparation path exists beyond the course material itself, I've left a little advice and a list that compiles recommended labs and boxes that will help build what I believe to be the necessary skills.

## The Challenge

The primary challenge when preparing for CAPE is finding suitable lab environments that replicate the exam's multi-domain, multi-machine setup. Unlike shared HTB boxes or single-machine challenges, CAPE requires hands-on experience with complex Active Directory attack chains across multiple systems. 

VulnLab's chain style labs are perfect for addressing this gap, offering dedicated multibox environments that closely simulate real enterprise AD configurations. While some VulnLab content has been ported to HTB, I strongly recommend using the original VulnLab platform while it remains available. Their chains are migrating to HTB's ProLabs tier, which is significantly more expensive and uses shared infrastructure that can complicate practice scenarios.

## Critical Skills Beyond the Course

While the course material is comprehensive and should be your primary resource, there are a few practical skills that will make your exam experience significantly smoother: Have an exploit development environment ready to go. You need to be comfortable quickly modifying and compiling exploits, and be familiar with a C2 framework. I used Adaptix during my exam, and even though I didn't rely on it heavily, knowing how to use it made certain parts very trivial.

Also, be ready to write your report, [SysReptor](https://docs.sysreptor.com/htb-reporting-with-sysreptor/) host templates you can use, which make it fairly trivial. In the case that this is your first HTB exam, get familiar with SysReptor.

## Essential Tooling

Automated tools are valuable in this exam, but manual enumeration will be your best friend. If you can understand why something works, or have a keen eye for key configs that automated tools may miss, you'll have a strong advantage.

BloodyAD was probably my star of the show where other tools fell short, or I needed to get right into configs/settings to identify attack paths. As already mentioned, a C2 can make certain parts trivial, so get familiar with one you like to use. The real key in all this is knowing your tools well enough that you aren't fumbling through documentation and furiously googling, you'll be doing that enough otherwise.

## Time Management and Preparation

I completed the exam in about 3 days, but this was a solid amount of focused work each day. The reason I could move through it relatively quickly was preparation. I had a solid base of notes and cheatsheets I'd developed over months of practice, so when I hit situations during the exam I wasn't scrambling to remember commands or googling basic techniques. The flip side of this is knowing when you're actually ready versus falling into practice paralysis. You can always do one more box, one more chain, one more certification before attempting CAPE. At some point you need to trust your preparation and just go for it. If you've gone through the course material thoroughly, practiced on the recommended chains, and feel comfortable with your enumeration methodology, you're probably ready. The exam isn't trying to trick you, it's testing whether you can apply what you've learned in a methodical way under pressure. Trust your prep work and commit to the attempt rather than endlessly postponing it.

## About the Lab List

The boxes and chains listed below aren't meant to be exam spoilers, many of the attacks aren't 1:1 with what you'll see in CAPE, and some techniques may not even appear at all. The goal here is to build strong AD enumeration and exploitation fundamentals, which is the most valuable skill you can have going into the exam. I've deliberately left out ProLabs (besides Ifrit) because I don't really think they're good prep, but you may find some benefit in Cybernetics and maybe even Offshore.

I really can't stress this enough: **the best resource for learning how to pass the exam is the course material itself.** These labs are supplementary practice to reinforce what you learn in the course, and hone your skills.

If you have any ideas, suggestions, or questions you can reach me on the HackTheBox discord as **VegeLasagne**, usually active in the **#cape** channel. If I've left something out it's probably because I've considered it too far out of scope, too web-focused (ew), too simple/complicated, or I've just plain missed it, feel free to make a suggestion to me in discord.

## Recommended Boxes/Chains

### HackTheBox
- Tombwatcher
- RustyKey
- Certificate
- Voleur
- Fluffy
- Signed
- DarkZero
- Eighteen
- Puppy
- Scepter
- EscapeTwo
- Vintage
- Manager
- Rebound
- Authority
- Escape
- Absolute
- Sekhmet

### VulnLab

- Ifrit (RTL)
- Heron
- Breach
- Delegate
- Redelegate
- Lustrous
- Puppet
- Reflection
- Sendai
- Cicada
- Trusted
- Scepter

### HackSmarter
- Triathlon
