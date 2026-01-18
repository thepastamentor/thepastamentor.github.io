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

As of mid-January 2025 HTB has moved to a "dedispawn" model where you get your own instance, this appears to only be for the standalone boxes, but what we may see is that the infra is changed for allow mutliple-machine labs. The pro-labs instances are still shared, and make Vulnlab a more suitable and cheaper model to access this content.

## Critical Skills Beyond the Course

While the course material is comprehensive and should be your primary resource, there are a few practical skills that will make your exam experience significantly smoother: Have an exploit development environment ready to go. You need to be comfortable quickly modifying and compiling exploits, and be familiar with a C2 framework. I used Adaptix during my exam, and even though I didn't rely on it heavily, knowing how to use it made certain parts very trivial.

Also, be ready to write your report, [SysReptor](https://docs.sysreptor.com/htb-reporting-with-sysreptor/) host templates you can use, which make it fairly trivial. In the case that this is your first HTB exam, get familiar with SysReptor. There are sample reports available too, these are a great example on how to format your report, what information to present, and how. I was knocked back on my first CPTS submission because the report was not up to par, the sample report was presented with the feedback and I was successful the second time around.

## Essential Tooling

Automated tools are valuable in this exam, but manual enumeration will be your best friend. If you can understand why something works, or have a keen eye for key configs that automated tools may miss, you'll have a strong advantage.

BloodyAD was probably my star of the show where other tools fell short, or I needed to get right into configs/settings to identify attack paths. As already mentioned, a C2 can make certain parts trivial, so get familiar with one you like to use. The real key in all this is knowing your tools well enough that you aren't fumbling through documentation and furiously googling, you'll be doing that enough otherwise.

I've since been put onto [powerview.py](https://github.com/aniqfakhrul/powerview.py) by a few people, which I did use a little in the course and not pay much attention to. On a bit more of a review it seems like a very well developed tool, and I would recommend learning how to use it for the course, you shouldn't be limiting your toolbox because you prefer one tool over another.

For pivoting, I used [ligolo-ng](https://github.com/nicocha30/ligolo-ng) thanks to how trivial I found it. I learned this tool a while ago so I still set everything up manually, and it appears to have a fair amount of automation in it, but I will always recommend learning how to set your tunnels and interface manually, since if something doesn't work you're not gonna know how to troubleshoot it. Naturally for pivoting do it however you find it easiest, but I've found ligolo really does make it trivial, especially as soon as you get to a double pivot and beyond.

## Time Management and Preparation

I completed the exam in about 3 days, but this was a solid amount of focused work each day. The reason I could move through it relatively quickly was preparation. I had a solid base of notes and cheatsheets I'd developed over months of practice, so when I hit situations during the exam I wasn't scrambling to remember commands or googling basic techniques. The flip side of this is knowing when you're actually ready versus falling into practice paralysis. You can always do one more box, one more chain, one more certification before attempting CAPE. At some point you need to trust your preparation and just go for it. If you've gone through the course material thoroughly, practiced on the recommended chains, and feel comfortable with your enumeration methodology, you're probably ready. The exam isn't trying to trick you, it's testing whether you can apply what you've learned in a methodical way under pressure. Trust your prep work and commit to the attempt rather than endlessly postponing it.

## About the Lab List

The boxes and chains listed below aren't meant to be exam spoilers, many of the attacks aren't 1:1 with what you'll see in CAPE, and some techniques may not even appear at all. The goal here is to build strong AD enumeration and exploitation fundamentals, which is the most valuable skill you can have going into the exam. I've deliberately left out ProLabs (besides Ifrit) because I don't really think they're good prep, but you may find some benefit in Cybernetics and maybe even Offshore.

I really can't stress this enough: **the best resource for learning how to pass the exam is the course material itself.** These labs are supplementary practice to reinforce what you learn in the course, and hone your skills.

If you have any ideas, suggestions, or questions you can reach me on the HackTheBox discord as **VegeLasagne**, usually active in the **#cape** channel. If I've left something out it's probably because I've considered it too far out of scope, too web-focused (ew), too simple/complicated, or I've just plain missed it, feel free to make a suggestion to me in discord.

If a box doesn't have a link, its either because it's still active on HTB, I can't find a decent writeup/video for it, or I've just forgotten.

## Recommended Boxes/Chains

### [HackTheBox](https://www.hackthebox.com/)
- Tombwatcher [(1)](https://benheater.com/hackthebox-tombwatcher/) [(2)](https://www.youtube.com/watch?v=um8b-TN76bY)
- RustyKey [(1)](https://0xdf.gitlab.io/2025/11/08/htb-rustykey.html) [(2)](https://www.youtube.com/watch?v=vkbIVr4_ZdE)
- Certificate [(1)](https://0xdf.gitlab.io/2025/10/04/htb-certificate.html) [(2)](https://www.youtube.com/watch?v=-4c5QcW3r4Q)
- Certified [(1)](https://0xdf.gitlab.io/2025/03/15/htb-certified.html) [(2)](https://www.youtube.com/watch?v=edW9sOyAS9Q)
- Voleur [(1)](https://0xdf.gitlab.io/2025/11/01/htb-voleur.html) [(2)](https://www.youtube.com/watch?v=G7FOSfZMC1I)
- Fluffy [(1)](https://0xdf.gitlab.io/2025/09/20/htb-fluffy.html) [(2)](https://www.youtube.com/watch?v=KvUC7bakm-E)
- Signed
- DarkZero
- Eighteen
- Puppy [(1)](https://0xdf.gitlab.io/2025/09/27/htb-puppy.html) [(2)](https://www.youtube.com/watch?v=QZQ-IjsI5Qk)
- Scepter [(1)](https://0xdf.gitlab.io/2025/07/19/htb-scepter.html) [(2)](https://www.youtube.com/watch?v=EEMNIIEzq24)
- EscapeTwo [(1)](https://0xdf.gitlab.io/2025/05/24/htb-escapetwo.html) [(2)](https://www.youtube.com/watch?v=fE6BYs4P1t4)
- Vintage [(1)](https://0xdf.gitlab.io/2025/04/26/htb-vintage.html) [(2)](https://www.youtube.com/watch?v=-JM--KfVhpA)
- Manager [(1)](https://0xdf.gitlab.io/2024/03/16/htb-manager.html) [(2)](https://www.youtube.com/watch?v=lT2Tv0m4k7M)
- Rebound [(1)](https://0xdf.gitlab.io/2024/03/30/htb-rebound.html) [(2)](https://www.youtube.com/watch?v=oUIoH4yBT3k)
- Authority [(1)](https://0xdf.gitlab.io/2023/12/09/htb-authority.html) [(2)](https://www.youtube.com/watch?v=7AF5riqLy-8)
- Escape [(1)](https://0xdf.gitlab.io/2023/06/17/htb-escape.html) [(2)](https://www.youtube.com/watch?v=PS2duvVcjws)
- Absolute [(1)](https://0xdf.gitlab.io/2023/05/27/htb-absolute.html) [(2)](https://www.youtube.com/watch?v=rfAmMQV_wss)
- Sekhmet [(1)](https://0xdf.gitlab.io/2023/04/01/htb-sekhmet.html) [(2)](https://www.youtube.com/watch?v=vsgPsMZx59w)

### [VulnLab](https://www.vulnlab.com/)

- Ifrit (RTL) [(1)](https://www.youtube.com/watch?v=frhZAKcOJrc)
- Heron [(1)](https://ethicxz.github.io/VL-Heron-Chain/) [(2)](https://www.youtube.com/watch?v=EmNc1vWPhTA)
- Breach [(1)](https://panosoikogr.github.io/2025/03/18/VL-Breach/) [(2)](https://www.youtube.com/watch?v=1pgS9Gxzyaw)
- Delegate [(1)](https://0xdf.gitlab.io/2025/09/12/htb-delegate.html) [(2)](https://www.youtube.com/watch?v=TdQjVjEye64)
- Baby2 [(1)](https://0xdf.gitlab.io/2025/09/26/htb-babytwo.html) [(2)](https://www.youtube.com/watch?v=a97PMfOXitY)
- Redelegate [(1)](https://0xdf.gitlab.io/2025/07/17/htb-redelegate.html)
- Lustrous [(1)](https://vuln.dev/lab-lustrous-walkthrough/) [(2)](https://www.youtube.com/watch?v=Zwsyv1MI7G4)
- Puppet [(1)](https://vuln.dev/vulnlab-puppet/)
- Reflection [(1)](https://notes.secure77.de/WriteUps/VulnLab/Reflection/Writeup) [(2)](https://www.youtube.com/watch?v=DH4dFwNTb9A)
- Sendai [(1)](https://0xdf.gitlab.io/2025/08/28/htb-sendai.html) [(2)](https://www.youtube.com/watch?v=ckkq3LcCHN8)
- Cicada [(1)](https://0xdf.gitlab.io/2025/07/03/htb-vulncicada.html)
- Trusted [(1)]()[(2)](https://www.youtube.com/watch?v=tUoqTFvbpgw&list=PLqyUgadpThTKvcfW94RxwQHFmVPxMIQGX)

### [HackSmarter](https://www.hacksmarter.org/dashboard)
- Triathlon [(1)](https://laffin.tech/writeups/triathlon-hack-smarter-writeup/) [(2)](https://www.youtube.com/watch?v=i4cYdjQg1W8)
- NorthBridge Systems [(1)](https://0xb0b.gitbook.io/writeups/hack-smarter/labs/northbridge-systems)


Last updated 18 Jan 2025.

Not written by AI. If you see emdashes, vulgar use of dot points, and vague info, run away.