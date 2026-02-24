Issue → Symptoms → Root cause → Fix → Verification

* Issues:
1. DNS using Shaw resolver / wrong DNS server
2. nslookup only worked when specifying the DNS
3. DC time vs member server time mismatch
4. Hyper-V time sync/timezone permissions
5. AADSTS700027 cert/key expired error
6. “sync already running” / no logs appearing

# Troubleshooting Log

This section documents key issues encountered during hybrid identity implementation and the structured troubleshooting approach used to resolve them.

---

## Incident 1 – Entra Connect Configuration Failure (AADSTS700027)

### Error Observed
AADSTS700027: The certificate used to sign the client assertion is expired.

### Impact
Microsoft Entra Connect failed during Azure AD connector initialization. Synchronization could not complete.

### Investigation

Checked system time on both Domain Controller and Member Server:

w32tm /query /status  
Get-Date  

Observed:
- Member server time was 2 hours behind.
- Domain Controller was using Local CMOS Clock.
- Timezone misalignment detected.
- Member server was syncing via Hyper-V Time Integration instead of domain hierarchy.

### Root Cause
Certificate-based authentication failed due to time skew between local environment and Azure.  
System time was outside the certificate validity window, causing MSAL authentication failure.

### Resolution

1. Corrected timezone configuration.
2. Disabled Hyper-V time synchronization integration.
3. Forced domain-based time synchronization:

w32tm /resync  

4. Verified authoritative source:

w32tm /query /status  

Time source correctly changed to Domain Controller.

### Verification
Re-ran Entra Connect configuration.  
AAD connector initialized successfully with no further authentication errors.

---

## Incident 2 – Domain Controller Not Syncing External Time

### Symptoms
w32tm /resync returned: “no time data available”.

### Root Cause
DNS forwarders were not configured on the Domain Controller, preventing external NTP resolution.

### Resolution

Configured DNS forwarders to router.

Validated external resolution:

nslookup time.windows.com  

Confirmed DC could resolve external hosts.

---

## Incident 3 – Users Not Appearing in Entra ID After Sync

### Symptoms
Synchronization completed successfully but no users appeared in Entra ID.

### Investigation
Verified:
- Sync scheduler running.
- No connector errors.
- Group-based filtering configured.

Discovered:
Security group used for filtering was located outside the scoped OU selected during OU filtering.

### Root Cause
Group object was not imported due to OU filtering scope mismatch.

### Resolution
Moved security group into the selected synchronization OU.
Triggered delta sync:

Start-ADSyncSyncCycle -PolicyType Delta  

### Verification
Scoped users successfully appeared in Entra ID.

---

## Incident 4 – DNS Resolution Using ISP Instead of Domain DNS

### Symptoms
nslookup returned Shaw ISP DNS server instead of internal Domain Controller DNS.

### Root Cause
Client network adapter was using router DNS instead of Domain Controller.

### Resolution
Configured client to use Domain Controller IP as primary DNS server.
Validated:

nslookup beastdc.ykt.dhruv  

Internal resolution successful.

---

## Incident 5 (Deep Dive) – Sync Engine Execution & Validation

### Scenario
After configuration, synchronization did not appear to run automatically.

### Steps Taken

Verified scheduler:

Get-ADSyncScheduler  

Triggered manual sync:

Start-ADSyncSyncCycle -PolicyType Initial  

Checked sync status:

Get-ADSyncConnectorRunStatus  

Reviewed Event Viewer logs:
Applications and Services Logs → Forefront Identity Manager → Operational

### Outcome
Confirmed successful import → sync → export cycle.
Validated user authentication in Microsoft 365 portal using synced credentials.
