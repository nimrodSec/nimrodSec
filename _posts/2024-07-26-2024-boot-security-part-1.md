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
Due to the elevated privileges of the BIOS and bootloader, a malicious boot component could
result in a sabotage of the entire system’s security!

In order to design a secure boot process, there is a need of clear
requirements for boot-level security, as well as an analysis of current
boot security measures.
While advances have been made in improving platform
firmware and bootloader security, most notably in the form
of Secure Boot, ways of circumventing those protections have
been found.

{: .box-warning}
It is crucial to find effective solutions for establishing a secure and verifiable boot process,
as even security focused operating systems, such as Tails or QubesOS, can not protect
from compromised boot firmware!

### The BIOS and TPM

The system Basic Input Output System (**BIOS**) is the first
piece of software executed on the CPU when a computer
is powered on. 

While historically its role was to provide
operating systems access to hardware, its current primary
role is to initialize and test hardware components & load
the operating system. In addition, modern BIOS loads and
initializes important system management functions.

While there are several different types of BIOS firmware on x86
systems, most newer systems use boot firmware based on
the UEFI specifications. 

The BIOS has traditionally been considered the so called **root of trust**
of the computer. This means that the entire code of the BIOS is assumed to be not malicious. 
 
{: .box-error}
As the BIOS is the the first code that runs on
the CPU, the BIOS can (maliciously) modify the OS image that
it is supposed to load. Due to the privileged access it has to
all the hardware, it can (maliciously) reprogram all periphery
devices.

However, *assuming* the BIOS is trustworthy is not enough. 
The *trusted computing base* (the code considered trustworthy) needs to be much smaller,
and at best verifiable.   

The trusted platform module (**TPM**) aims to provide a trustworthy root of trust. 
It is a discrete microchip found on most modern computer systems, tasked with
providing basic security-related functions, including securely
storing and generating encryption keys and providing assurances about the state of a system. 
Many boot security features utilize the TPM for measuring 

###  BOOT-LEVEL THREATS

{: .box-note}
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
The most notable example of this
kind of attacks is **BootHole**.
BooHole effected the GRUB2 bootloader, which is utilized by most Linux systems. 
It enabled attackers to gain arbitrary code execution during the
boot process via a Buffer-Overflow vulnerability, even when
Secure Boot was enabled. An exploit of the vulnerability
would enable attackers to gain persistence thus giving them
near-total control over the victim device.

{: .box-note}
The BIOS must detect changes to the bootloader
and trigger a corresponding action if a change is observed.

The action can range from notifying the user, to refusing to
boot if a modification is detected.
Due to the fact that bootloaders such as GRUB are updated
regularly, managing the trustworthiness of the bootloader is not easy.


#### B. BIOS Backdoor & Supply Chain Attacks

{: .box-warning}
While the common assumption is that the BIOS arrives to
the user in a trusted state, there is no technical assurance that
this is the case. 

One possible threat to boot security is the
installation of a backdoor by the BIOS manufacturer (OEM).

A backdoor, as defined by MITRE, is an intentional malicious
action taken to create and ultimately exploit a vulnerability
in a technology at some point within the supply chain.

{: .box-success}
While no known cases of a BIOS backdoor exist as of the
writing of this paper, targeted backdooring of devices at the
order of law enforcement agencies is technically and legally
feasible.

Even when no intentional backdoor is installed, the BIOS is
still susceptible to other types of supply chain attacks! 

#### C. Bootkits

Assuming that the system arrives with the manufacturers
intended BIOS installed, and assuming that the  BIOS is not backdoored by the OEM, 
there are still threats to the integrity of the system BIOS during the systems lifetime,
most notably in the form of bootkits.

Bootkits work by abusing and subverting the operating system
in the course of the initial boot process. 
A malicious modification of the BIOS code can happen in two main ways.

{: .box-error}
A. If an attacker with physical access to the device
   connects an SPI programmer to the SPI chip, they can
   replace the benign firmware contents with a malicious one.
B. If an attacker with elevated local privileges reflashes the BIOS 
   from the operating system.

Reflashing the BIOS from the operating system, requires either a lack of proper reflashing protection implemented by the original BIOS, or an exploit of the original BIOS to get
code execution rights, before the reflashing locks are applied, for example during an update.

{: .box-warning}
Malicious modifications to the BIOS must be detected
and trigger a corresponding action.

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

This can be achieved for example by open source BIOS and bootloader
code with reproducible builds.
In this case, even when the BIOS arrives in an untrusted
state, due to the possibility of a malware infection 'en passant',
the user can compile the BIOS from source and flash it to
the BIOS chip, either internally, or externally via an SPI
programmer.

#### B. Verifiable Trusted State

{: .box-note}
It is necessary that the user can verify that the BIOS and the
bootloader have not been altered unknowingly.

This can be achieved for example by computing a checksum
or signature of each BIOS component and the bootloader
during each boot, and comparing those to saved, trusted values.
The technical implementation could use a TPM and its PCRs
to manage the trusted state. A security token or an open
authentication app could add a second factor to establish trust
in the TPM's measurements.

#### C. Verifiable Updated State

{: .box-note}
It is necessary that the user can update the BIOS and the
bootloader and reestablish a trusted state in case of an infection
or a new software release.

In the case of a BIOS update, this can be achieved for ex-
ample by manually recompiling and verifying the new release, internally or
externally flashing it to the BIOS, recomputing the signatures
and checksums of the BIOS components and sealing them to
the TPM.
In the case of a bootloader update, the same applies, except
that the update may be done via the common release channel
of the operating system and that no flashing is involved.

``In the next post, I will discuss whether UEFI Secure Boot holds up to those requirements.`` 

{: .box-success}
I am happy about feedback!
You can find my contact details [here](/nimrodSec/contact).

