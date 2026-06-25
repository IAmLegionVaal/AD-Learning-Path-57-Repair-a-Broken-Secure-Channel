# AD Learning Path 57 — Repair a Broken Secure Channel

## Objective
Diagnose and repair a workstation or member-server trust failure without immediately removing it from the domain.

## Prerequisites
- Disposable domain member
- Local administrator access
- Authorized domain credential
- Healthy DNS, time, and domain controllers
- Baseline computer-account evidence

## Workflow
1. Confirm the user-facing trust error and collect System and Netlogon events.
2. Check that the member uses only internal AD DNS and can resolve LDAP/Kerberos SRV records.
3. Compare time with a domain controller.
4. Inspect the AD computer object for duplicate, disabled, or recently reset state.
5. Test the secure channel.
6. Repair it against a known healthy DC using an interactive authorized credential.
7. Restart if required and validate domain authentication, Kerberos, Group Policy, and the computer-account password timestamp.

```powershell
Get-DnsClientServerAddress -AddressFamily IPv4
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.corp.lab
w32tm.exe /stripchart /computer:dc01.corp.lab /samples:5 /dataonly
Test-ComputerSecureChannel -Verbose

$Credential = Get-Credential -Message 'Authorized domain repair account'
Test-ComputerSecureChannel -Repair -Server DC01 -Credential $Credential -Verbose
```

## Validation
```powershell
Test-ComputerSecureChannel -Verbose
nltest.exe /sc_verify:corp.lab
nltest.exe /dsgetdc:corp.lab
klist.exe
gpupdate.exe /force
Get-ADComputer $env:COMPUTERNAME -Properties PasswordLastSet,Enabled,DNSHostName
```

## Evidence
Store the original error, DNS/time checks, secure-channel failure, AD object state, repair output, post-repair authentication and policy checks, relevant events, root cause, and final pass/fail status under `evidence/`.

## Troubleshooting
- Repair fails: verify DNS, time, firewall, DC reachability, and the exact computer object.
- Duplicate name or snapshot rollback: resolve the identity conflict before repair.
- Repair succeeds then fails again: investigate cloning, reverted snapshots, or repeated account resets.

## Security notes
Use the narrowest authorized repair credential and never place it in scripts. Avoid domain remove/rejoin until the simpler repair path has been evaluated and evidence preserved.

## Cleanup
Return the disposable endpoint to a healthy baseline and remove any intentionally introduced fault.

## References
- Microsoft Learn: Secure channel troubleshooting
- Microsoft Learn: `Test-ComputerSecureChannel`

## Next activity
`AD-Learning-Path-58-Diagnose-Group-Policy-Processing`
