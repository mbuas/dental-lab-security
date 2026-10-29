# Securing Software & Integrated Systems in a Dental Laboratory

> Building in-house software and system integrations for a dental laboratory that handles patient data, then attacking my own work to find and fix weaknesses before anyone else can.

This repository is a written overview of my role and approach. It contains **no source code, configuration, network details or references to internal systems**, on purpose. Publishing those would undermine the protections described below.

---

## Table of contents

1. [My role](#my-role)
2. [Why security matters here](#why-security-matters-here)
3. [My approach: build, break, patch, repeat](#my-approach-build-break-patch-repeat)
4. [What I test for](#what-i-test-for)
5. [Project: dental scanner setup](#project-dental-scanner-setup)
6. [Project: VNC Alert](#project-vnc-alert)
7. [Principles I work by](#principles-i-work-by)
8. [Skills and tools](#skills-and-tools)

---

## My role

I work at a dental laboratory, where I design, build and maintain the software and system integrations the lab uses every day. That covers tools that connect machines, workstations and staff over the company's internal network, and software that receives, processes and stores **patient-related data** such as scans, case information and the identifiers that link them to a patient and a dental practice.

On top of building these systems, I am also responsible for securing them. In practice that means:

- **Early detection.** Spotting security weaknesses as early as possible, ideally at design time, before a system goes into daily use.
- **Identification.** Understanding exactly what a weakness is, how it could be exploited, what data or functionality it exposes and how serious the impact would be.
- **Patching.** Fixing the root cause, not just the symptom, then checking that the fix holds and hasn't introduced new problems.

I use my own knowledge and experience together with the tools available to me (analysis and reverse-engineering tools, network inspection, AI-assisted code review and testing) to do this continuously, not as a one-off audit.

---

## Why security matters here

Dental labs are an easy target to underestimate. The data they handle is sensitive:

- **Patient data is personal and medical.** Intraoral scans, case notes and identifiers count as health-related personal data. A leak harms real people and carries legal consequences under data-protection rules such as the GDPR.
- **The lab sits between many parties.** Cases come in from dental practices and go back out. Every integration point is a possible way in.
- **Lab environments are mixed.** Specialised hardware, vendor software, older workstations and newer custom tools often share one network. The weakest link sets the security level for everything.
- **The software itself is an asset.** The systems I build encode the lab's workflows and know-how. If they could be easily copied or cloned, the lab would lose value, and a cloned system with weaker controls could become another route to patient data.

So I treat two things as worth protecting: **the patient data** and **the systems and software themselves**.

---

## My approach: build, break, patch, repeat

My main method is to **attack my own work from the outside**. Once a piece of software or an integration is built and working, I stop being its developer and start acting like someone who wants to abuse it.

### 1. Build with security in mind

Before writing anything, I map out:

- what data the system will touch, where it comes from, where it goes and where it rests;
- who and what should have access (people, machines, other services), and at what level;
- which parts are exposed to the network, to removable media or to other users of the same machine.

This gives me a threat model to build against and, later, to test against.

### 2. Deconstruct it

After the system works, I take it apart the way an outsider would:

- examining what the finished, deployed product reveals about itself (files, stored data, logs, network traffic, error messages, metadata);
- analysing compiled or packaged components to see how much of the internal logic, configuration or secrets can be recovered;
- looking at how components talk to each other and whether that communication can be read, replayed or faked.

### 3. Try to replicate it

I try to **rebuild or clone** the system using only what an attacker could realistically get: a copy of the installed software, access to a workstation, or a position on the internal network. Key questions:

- Can the software be copied to another machine and run without authorisation?
- Can its behaviour be reproduced well enough to impersonate it on the network?
- Can licensing, identity or integrity checks be bypassed or removed?

If I can replicate it, so can someone else, and that becomes a finding to fix.

### 4. Try to withdraw patient data

Next I try to **extract patient data** in every way I can think of:

- reading data at rest (local storage, caches, temporary files, exports, backups);
- intercepting data in transit across the internal network;
- abusing legitimate features (exports, previews, logs, debug output, notifications) to leak more than intended;
- escalating from limited access, such as a standard user or a single workstation, to data that access should never reach;
- checking what remains after data is "deleted" or a case is closed.

### 5. Patch and verify

Every successful attack, and every attempt that came uncomfortably close, turns into a fix:

- fixing the root cause (e.g. encrypting data instead of just hiding it, removing secrets from places they don't belong, tightening access instead of obscuring it);
- re-running the same attack to confirm it no longer works;
- checking that the fix didn't break the lab's workflow, because security that gets in the way of daily work tends to get bypassed.

### 6. Repeat

Systems change: new features, new machines, new vendor updates, new staff. I repeat the cycle whenever something meaningful changes, and periodically even when nothing seems to have.

---

## What I test for

Without going into specifics, these are the main categories I check in everything I build:

| Area | What I look for |
|---|---|
| **Data at rest** | Patient data stored unencrypted, left in temporary or cache locations, or kept longer than needed |
| **Data in transit** | Unencrypted or unauthenticated traffic on the internal network; traffic that can be sniffed, replayed or spoofed |
| **Access control** | Missing or weak authentication; users or machines able to reach more than they need |
| **Secrets handling** | Credentials, keys or tokens embedded in software, configs or scripts where they can be recovered |
| **Reverse engineering & cloning** | How easily the software can be decompiled, understood, modified, copied or run without authorisation |
| **Information leakage** | Logs, error messages, notifications, filenames or metadata revealing patient or system details |
| **Integrity** | Whether components can be tampered with or replaced without the system noticing |
| **Network exposure** | Services listening more widely than necessary; trust placed in "being on the internal network" alone |

---

## Project: dental scanner setup

*Status: in progress*

### What it is

I am currently designing and building the software and integration setup around a dental scanner used in the lab. The setup covers the scanning workstation and the software that **receives, processes, organises and passes on scan data**, which is linked to individual patients and cases.

Since this system processes patient data from the start, security is part of the design from day one rather than added after it works.

### Security focus

- **Data flow mapping.** Before implementation I map every stage patient data passes through: capture, processing, storage, handover to other systems or people, and eventual removal. Each stage is a place where data could leak and needs its own protection.
- **Minimal exposure.** The setup should store and expose only the patient information each step actually needs, and nothing should linger on the workstation once it has been handed off.
- **Protecting data at rest and in transit.** Scan files and case data must not be readable by someone who gets access to the machine's storage or watches the network.
- **Controlled access.** Only the intended people and systems should be able to start scans, view results or retrieve data.
- **Resistance to copying.** The software and its configuration should not be easy to lift onto another machine and run, or to study in order to reproduce the setup.

### How I'm testing it

As parts of the setup start working, I put each one through the build → break → patch cycle described above:

- treating the scanning workstation as if it were compromised or stolen, and checking what patient data could be recovered from it;
- observing what the software sends over the network and whether that could be read or imitated;
- analysing the finished software as an outsider would, to see how much of its logic and configuration can be recovered or reused;
- trying to get patient data out through legitimate-looking routes such as exports, logs, previews and leftover files.

Findings are fixed before that component goes into routine use, and the full setup gets the same treatment again before it handles real patient cases.

---

## Project: VNC Alert

*Status: in use, actively maintained*

### What it is

VNC Alert is an internal monitoring and alerting tool I built for the lab. It runs on the **company's internal network** and uses remote-desktop (VNC) connectivity between machines to detect specific events or states and notify the right people quickly, so problems get handled without someone having to check each machine in person.

### Why it needs securing

A tool like this is useful precisely because it can see across the network, and that is also what makes it sensitive:

- **Remote-desktop access is powerful.** Anything that relies on VNC connectivity could, if misconfigured or abused, give someone a view into, or control over, other workstations, including ones where patient data is displayed or processed.
- **The internal network is not a security boundary on its own.** Being "inside" should not automatically mean being trusted. A compromised device or an unauthorised person on the network should not be able to use the tool, or the connections it relies on, as a stepping stone.
- **Alerts can leak information.** Notifications and logs must not reveal patient details or sensitive system information to anyone who happens to see them.

### Security focus

- **Limiting what the tool can reach and do** to the minimum it needs to perform its function.
- **Protecting the connections and credentials** it depends on, so they can't be read off the network or recovered from the software or its configuration.
- **Making sure the tool can't be impersonated**, so fake alerts can't be injected and real ones can't be silently suppressed.
- **Keeping alert content clean** of patient data and unnecessary internal details.

### How I test it

I attack VNC Alert as if I were an unauthorised device on the internal network or someone holding a copy of the software:

- watching its network traffic for anything readable, replayable or spoofable;
- trying to recover connection details or credentials from the deployed tool;
- trying to use the tool, or the remote-desktop access it relies on, to reach machines or screens it shouldn't;
- checking whether its behaviour can be reproduced well enough to send convincing fake alerts.

Each weakness found this way is patched and re-tested, and I repeat the process whenever the tool or the network around it changes.

---

## Principles I work by

- **Assume I'm not the only one looking.** If I can find a weakness, someone else can too, so it gets fixed and not filed away.
- **Attack my own work honestly.** I test with an outsider's mindset and resources, not with my insider knowledge of how the system is "supposed" to be used.
- **Protect the data, not just the perimeter.** Network boundaries and passwords help, but patient data should stay protected even if one layer fails.
- **Least privilege everywhere.** Every user, machine and component gets only the access it needs.
- **Collect and keep less.** Data that is never stored can't leak.
- **Security has to fit the workflow.** Lab staff need to work efficiently. Controls that create friction get bypassed, so I design protections that are as invisible as possible.
- **Security through obscurity is not enough.** Hiding how something works is a bonus, never the main defence.
- **Keep internals private.** That's why this README has no code, configuration or system details.

---

## Skills and tools

- Designing and building custom software and system integrations for a production lab environment
- Threat modelling and data-flow analysis for systems handling personal and health-related data
- Self-directed security testing of my own systems: reverse engineering, network traffic analysis, data-extraction and cloning attempts
- Root-cause remediation and re-testing of identified vulnerabilities
- Using AI-assisted tools to speed up code review, analysis and testing
- Working knowledge of data-protection requirements relevant to patient data (GDPR)

---

<sub>This document describes my work in general terms only. No patient data, source code, credentials or internal infrastructure details are included or referenced.</sub>
