** Prereqs
** Install Entra Connect Sync
** Select PHS + optional SSO (what you chose and why)
** OU filtering + group filtering (what you configured)

# Implementation Steps

## 1. Active Directory Preparation

- Created dedicated OU: EntraSync-Users
- Created Global Security Group: EntraSyncGrp
- Added selected test users (labuser3, labuser4)
- Verified UPN format: user@rationabeing****.onmicrosoft.com (Different from my DC Domain - YKT.dhruv)

## 2. DNS Configuration

- Configured DNS forwarders on Domain Controller
- Verified name resolution using:
  nslookup login.microsoftonline.com

## 3. Time Synchronization

- Verified NTP source using:
  w32tm /query /status
- Disabled Hyper-V time sync
- Corrected timezone mismatch
- Confirmed DC as authoritative time source

## 4. Microsoft Entra Connect Installation

- Selected Password Hash Synchronization (PHS)
- Enabled optional SSO
- Configured OU filtering
- Configured group-based filtering

## 5. Synchronization Validation

- Executed:
  Start-ADSyncSyncCycle -PolicyType Initial
  Then
  Start-ADSyncSyncCycle -PolicyType Delta (When needed)
- Verified sync status:
  Get-ADSyncScheduler
- Confirmed successful export to Entra ID
- Users were found in the Entra ID and were responding to the changes made in the DC (AD)
