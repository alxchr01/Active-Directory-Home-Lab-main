# Introduction

## Overview

This project documents the design, deployment, security assessment, and
hardening of an Active Directory environment built entirely within an
isolated virtual lab.

The environment consists of a Windows Server 2025 Domain Controller, a
domain-joined Windows 11 workstation, and a Kali Linux system used for
security testing. The machines operate on an isolated VMware host-only
network, allowing Active Directory attacks and defensive techniques to be
tested safely without interacting with external systems.

The project follows the environment through its complete security lifecycle:
initial deployment and configuration, reconnaissance and enumeration,
attack simulation, detection, remediation, and security validation.

## Objectives

The primary objectives of the project are to:

- Build and administer a functional Windows Active Directory domain
- Configure users, groups, Organizational Units, DNS, and Group Policy
- Deploy and manage a domain-joined Windows workstation
- Perform Active Directory reconnaissance and enumeration
- Simulate common Active Directory attack techniques
- Analyze attack activity using Windows security telemetry
- Identify and remediate security weaknesses
- Validate mitigations by repeating security tests after hardening

## Lab Environment

| System | Operating System | Role | IP Address |
|---|---|---|---|
| DC01 | Windows Server 2025 | Domain Controller / DNS | 192.168.10.10 |
| CLIENT01 | Windows 11 | Domain Workstation | 192.168.10.20 |
| KALI01 | Kali Linux | Security Testing | 192.168.10.30 |

**Domain:** `lab.local`  
**Network:** `192.168.10.0/24`  
**Virtualization:** VMware Workstation Pro  
**Network Type:** Isolated host-only network (VMnet2)

## Project Scope

The project covers both offensive and defensive aspects of Active Directory
security. Security testing includes service discovery, domain enumeration,
credential and authentication attacks, privilege escalation, lateral
movement, and Active Directory permission abuse.

Defensive work includes Windows auditing, authentication monitoring,
PowerShell logging, analysis of relevant Windows events, account and
privilege hardening, service-account security, and Group Policy
configuration.

All security testing is performed exclusively against systems created for
this lab.

## Methodology

The project is divided into five main phases:

1. **Build** — Deploy and configure the Active Directory environment.
2. **Enumerate** — Discover systems, services, accounts, and trust
   relationships from the security-testing machine.
3. **Attack** — Reproduce selected Active Directory attack techniques in
   the controlled environment.
4. **Detect and Harden** — Analyze the resulting telemetry and implement
   appropriate security controls.
5. **Validate** — Repeat relevant tests to determine whether the implemented
   mitigations are effective.

Detailed procedures, evidence, findings, and remediation steps are documented
in the subsequent sections of the repository.
