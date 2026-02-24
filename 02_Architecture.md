* Environment overview (DC, Member Server, Entra tenant)
* Identity flow (AD → Entra via Entra Connect Sync)
* OU / Group scoping approach (why you avoided full sync)

# Architecture Overview

## Environment Components

- 1 × Windows Server 2022 Domain Controller (AD DS, DNS)
- 1 × Windows Server 2022 Member Server (Microsoft Entra Connect)
- Microsoft Entra ID Tenant (rationabeing3346.onmicrosoft.com)
- 1 × Windows 10 Pro Client VM (DC, Hyper-V)
- 1 × Windows 10 Pro Client VM (On Personal Laptop for direct Entra ID Cloud-Users)

## Identity Model

- On-premises AD DS is the authoritative identity source.
- Microsoft Entra Connect configured for:
  - Password Hash Synchronization (PHS)
  - Scoped synchronization using OU and security group filtering

## Synchronization Scope

- Dedicated OU: EntraSync-Users
- Security Group: EntraSyncGrp (Global Security)
- Only selected test users are included to simulate a controlled enterprise rollout.

## Authentication Flow

User logs in → AD authenticates → Password hash synced to Entra ID → Cloud authentication validated.

## Key Infrastructure Dependencies

- DNS forwarders configured on the Domain Controller
- Correct NTP hierarchy
- Hyper-V time synchronization on Member Server
- Azure application certificate validation (resolved AADSTS700027)
