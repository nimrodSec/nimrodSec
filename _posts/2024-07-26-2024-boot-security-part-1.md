---
layout: post
title: Boot Security Requirements
subtitle: Why you should not trust your BIOS 
tags: [low-level security]
comments: false
mathjax: true
author: Nimrod Adam
---

In the last years, attacks against the boot process have
become more and more sophisticated. 
In comparison to malware targeting the operating system, boot-level malware is
highly persistent, often sustaining both reboots and re-installs
of a system. 
Furthermore, due to the elevated privileges of the BIOS and bootloader, a malicious boot component could
result in the sabotage of the entire system’s security!

In order to design a secure boot process, there is a need of clear
requirements for boot-level security, based on an analysis of current
boot security threats.

{: .box-warning}
It is crucial to find effective solutions for establishing a secure and verifiable boot process,
as even security focused operating systems, such as Tails or QubesOS, can not protect
from compromised platform firmware!

### The BIOS and TPM

The system Basic Input Output System (**BIOS**) is the first
piece of software executed on the CPU when a computer
is powered on. 

While historically its role was to provide
operating systems access to hardware, its current primary
role is to initialize and test hardware components \& system management functions, and then to load
the operating system. 

While there are several different types of BIOS firmware on x86
systems, most newer systems use boot firmware based on
the **UEFI** specifications. 

The BIOS has traditionally been considered the so called **root of trust**
of the computer. This means that the entire code of the BIOS is assumed to be not malicious. 
 
{: .box-error}
As the BIOS is the the first code that runs on
the CPU, the BIOS can (maliciously) modify the OS image that
it is supposed to load. Due to the privileged access it has to
all the hardware, it can also (maliciously) reprogram all periphery
devices.

*Assuming* that the BIOS is trustworthy is thus not enough. 
The *trusted computing base* (the code considered trustworthy) needs to:
- small: to minimize the attacks surface
- verifiable: to ensure the authenticity and integrity of the code  


The trusted platform module (**TPM**) aims to provide a minimal, trustworthy root of trust. 
The TPM is a discrete microchip found on most modern computer systems, tasked with
providing basic security-related functions, including securely
storing and generating encryption keys and providing assurances about the state of a system. 

{: .box-note}
Many boot security features utilize the TPM. The inner workings of it will be discussed in a future post on *measured boot*.

###  BOOT-LEVEL THREATS

The following sections attempt
to establish an understanding and categorization of *some* possible
threats to the security of the boot process, in order to formulate
according security requirements. Examining all possible types of attacks is beyond
the scope of this post. 

#### A. Bootloader Attacks

Before diving into the realm of BIOS attacks, it is
important to mention that attacks on the bootloader also exist,
which can sometimes even sabotage the underlying BIOS
security measures. 

{: .box-note}
In a nutshell, a bootloader is a program that starts whenever a device is powered on to activate the operating system. 

The most notable example of a bootloader attack is **BootHole**.
BooHole effected GRUB2, a common bootloader utilized by most Linux systems. 
It enabled attackers to gain arbitrary code execution during the
boot process via a Buffer-Overflow vulnerability, even when
Secure Boot was enabled. An exploit of the vulnerability
would also enable attackers to gain persistence thus giving them near-total control over the victim device.

{: .box-warning}
Conclusion: The BIOS must detect changes to the bootloader, and trigger a corresponding action if a change is observed!

The action can range from notifying the user, to refusing to
boot if a modification is detected.
Due to the fact that bootloaders such as GRUB are updated
regularly, managing the trustworthiness of the bootloader is not an easy task.


#### B. BIOS Backdoor & Supply Chain Attacks

{: .box-warning}
While the common assumption is that the BIOS arrives to
the user in a trusted state, there is no technical assurance that
this is the case. 

A further threat to boot security is the
installation of a backdoor by the BIOS manufacturer (**OEM**).

A backdoor, as defined by *MITRE*, is an intentional malicious
action taken to create and ultimately exploit a vulnerability
in a technology at some point within the supply chain.

{: .box-success}
While no known cases of a BIOS backdoor exist as of the
writing of this paper, targeted backdooring of devices at the
order of law enforcement agencies is technically and legally
feasible. 

Even when no intentional backdoor is installed by the OEM, the BIOS is
still susceptible to other types of supply chain attacks! NSA's TAO Operations are one prominent example. 

#### C. Bootkits

Assuming that the system arrives with the manufacturers
intended BIOS installed, and assuming that this BIOS was not backdoored by the OEM, 
there are still threats to the integrity of the system BIOS during the system's lifetime,
most notably in the form of bootkits.

Bootkits work by abusing and subverting the operating system
in the course of the initial boot process. 
A malicious modification of the BIOS code can happen in two main ways.

{: .box-error}
A. If an attacker with physical access to the device
   connects an SPI programmer to the SPI chip, they can
   replace the benign firmware contents with a malicious one.

{: .box-error}
B. If an attacker with elevated local privileges reflashes the BIOS from the operating system.

However, reflashing the BIOS from the operating system requires either a lack of proper reflashing protection implemented by the original BIOS, or an exploit of the original BIOS to get code execution rights, before the reflashing locks are applied, for example during an update. 

{: .box-warning}
Conclusion: Malicious modifications to the BIOS must be detected and trigger a corresponding action!

As the BIOS is updated more rarely, managing the trusted
state of the BIOS is less difficult then with the bootloader.

### BOOT SECURITY REQUIREMENTS

Based upon the above threats to boot-
level security, the following requirements for a secure boot
process can be formulated:

#### A. Verifiable Initial State

{: .box-note}
It is necessary that the user can verify that the installed
BIOS and bootloader are in a trusted initial state.

This can be achieved for example by open source BIOS  and bootloader
code with reproducible builds.

In this case, even when the BIOS arrives in an untrusted
state, due to the possibility of a malware infection 'en passant', the user can compile the BIOS from source and flash it to the BIOS chip, either internally, or externally via an SPI programmer.

{: .box-error}
Although the current UEFI speficitaion is open-source, the OEM implementation is not! The only way the user can reflash the BIOS is by using the binary code from the OEM. 

#### B. Verifiable Trusted State

{: .box-note}
It is necessary that the user can verify that the BIOS and the
bootloader have not been altered unknowingly.

This can be achieved for example by computing a checksum
or signature of each BIOS component and the bootloader
during each boot, and comparing those to saved, trusted values.
The technical implementation could use a TPM
to manage the trusted state.
A security token or an open
authentication app could add a second factor to establish trust in the TPM's measurements.

#### C. Verifiable Updated State

{: .box-note}
It is necessary that the user can update the BIOS and the bootloader and reestablish a trusted state in case of an infection or a new software release.

In the case of a BIOS update, this can be achieved for example by manually recompiling and verifying the new release, internally or
externally flashing it to the BIOS, recomputing the signatures
and checksums of the BIOS components and sealing them to
the TPM.

In the case of a bootloader update, the same applies, except that the update may be done via the common release channel of the operating system and that no flashing is involved.



``And this is it for the first post! In the next one, I will discuss whether UEFI Secure Boot holds up to those requirements. Some of you may already guess...`` 

{: .box-success}
I would be very happy about any questions or feedback!
 You can find my contact details [here](/nimrodSec/contact).

### Bibliography:

X. Kovah and C. Kallenberg, “How many million BIOS
would you like to infect”, 2015. 
[Online] Available: https://archive.conference.hitb.org/hitbsecconf2015ams/wp-content/uploads/2015/02/WHITEPAPER-How-Many-Million-BIOSes-Would-You-Like-To-Infect.pdf 

D. A. Cooper, W. T. Polk, A. R. Regenscheid, and
M. P. Souppaya, “BIOS protection guidelines,” National
Institute of Standards and Technology.
[Online] Available: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-147.pdf  

T. Hudson and L. Rudolph, “Thunderstrike: EFI
firmware bootkits for apple MacBooks”. [Online]. Available: https://dl.acm.org/doi/pdf/10.1145/2757667.2757673 

W. J. Heinbockel, E. R. Laderman, and G. J. Serrao, “Supply chain attacks and resiliency mitigations,” The MITRE Corporation, pp. 1–30, 2017.
[Online] Available: https://www.nist.gov/system/files/documents/2018/12/06/supply_chain_attacks_and_resiliency_mitigations_-_mitre.pdf

J. Rutkowska, “Intel x86 considered harmful,” the Invisible Things Lab, 2015. [Online]. Available:
https://blog.invisiblethings.org/papers/2015/x86_harmful.pdf

A. Robison. “There’s a hole in the boot,” Eclypsium
— Supply Chain Security for the Modern Enterprise.
(Jul. 29, 2020), [Online]. Available: https://eclypsium.com/blog/theres-a-hole-in-the-boot 

S. Gallagher. "Photos of an NSA “upgrade” factory show Cisco router getting implant". (2014).
[Online]. Available: 
https://arstechnica.com/tech-policy/2014/05/photos-of-an-nsa-upgrade-factory-show-cisco-router-getting-implant/