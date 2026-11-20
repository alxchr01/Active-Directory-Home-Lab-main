# Lab Setup

## Overview

The lab was deployed using VMware Workstation Pro and consists of three
virtual machines connected through an isolated host-only virtual network.

The environment was designed to provide communication between the lab systems
while keeping security testing isolated from external networks and other
devices on the host network.

## Virtual Machines

Three virtual machines were deployed:

| Hostname | Operating System | Role | IP Address |
|---|---|---|---|
| DC01 | Windows Server 2025 | Domain Controller / DNS | 192.168.10.10 |
| CLIENT01 | Windows 11 | Domain Workstation | 192.168.10.20 |
| KALI01 | Kali Linux | Security Testing | 192.168.10.30 |

## Network Configuration

A dedicated VMware host-only network, `VMnet2`, was created for the lab.

The network uses the following configuration:

| Setting | Value |
|---|---|
| Network | `192.168.10.0/24` |
| Network Type | Host-only |
| VMware DHCP | Disabled |
| Domain Controller | `192.168.10.10` |
| Windows Client | `192.168.10.20` |
| Kali Linux | `192.168.10.30` |

VMware DHCP was disabled so that addressing could be configured manually
and remain predictable throughout the security exercises.

The host-only configuration isolates the virtual environment from the
physical network while allowing communication between the virtual machines.

## Domain Controller

A Windows Server 2025 virtual machine was created and configured as `DC01`.

The server was assigned the static address:

`192.168.10.10/24`

VMware Tools was installed to improve integration between the guest and host
systems.

DC01 was subsequently used to host Active Directory Domain Services and DNS.
The deployment and configuration of these services are documented separately
in the Active Directory configuration section.

## Windows Client

A Windows 11 virtual machine was created as the domain workstation and renamed
`CLIENT01`.

The system was assigned:

`192.168.10.20/24`

DC01 was configured as the client's DNS server so that Active Directory
domain resources could be resolved correctly.

CLIENT01 was later joined to the `lab.local` domain and used to represent a
standard enterprise workstation.

## Kali Linux

A Kali Linux virtual machine was deployed as the security-testing system.

The machine was connected to VMnet2 and assigned:

`192.168.10.30/24`

Connectivity between KALI01 and DC01 was verified before beginning security
testing.

Kali was configured with the tools required for Active Directory
reconnaissance, enumeration, and attack simulation.

## Connectivity Verification

Network connectivity from KALI01 to the Domain Controller was verified using
ICMP.

```bash
ping -c 4 192.168.10.10
