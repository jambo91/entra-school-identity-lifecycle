# Leaver Process

## Objective

The leaver process demonstrates how Northstar Academy securely removes access when a teacher leaves the organisation.

Daniel Reed was used as the fictional departing teacher.

## Business Requirement

When a teacher leaves, the school must:

- Prevent new sign-ins immediately
- Invalidate existing authentication sessions
- Remove group-based access
- Recover assigned product licences
- Retain audit evidence of administrative actions
- Verify that the former teacher can no longer access the environment

## Step 1: Disable the Account

Daniel's account was changed from enabled to disabled.

```text
AccountEnabled: true → false
```

Disabling the account prevents new authentication attempts from succeeding.

The account was disabled instead of deleted so that the organisation could retain the identity object and associated audit information during the offboarding and retention period.

## Step 2: Revoke Active Sessions

Daniel's active sessions were revoked.

Microsoft Entra updated the `StsRefreshTokensValidFrom` property, invalidating refresh tokens issued before the new timestamp.

Account disabling and session revocation were performed as separate controls:

- Account disabling blocks new authentication.
- Session revocation helps invalidate existing sessions and forces reauthentication.

## Step 3: Test the Disabled Account

Before offboarding, Daniel successfully signed in to the Microsoft My Account portal.

After the account was disabled and sessions were revoked, another sign-in attempt was performed.

The sign-in was blocked with the following message:

```text
Your account has been locked. Contact your support person to unlock it, then try again.
```

This confirmed that the offboarding controls prevented Daniel from authenticating.

## Step 4: Remove Group-Based Access

The teacher dynamic-group rules were updated to include the account-status condition:

```text
user.accountEnabled -eq true
```

Final `DG-All-Teachers` rule:

```text
(user.employeeId -startsWith "TEACHER-") and
(user.accountEnabled -eq true)
```

Final `DG-Science-Teachers` rule:

```text
(user.employeeId -startsWith "TEACHER-") and
(user.department -eq "Science") and
(user.accountEnabled -eq true)
```

Rule validation confirmed that disabled Daniel no longer satisfied either rule.

Actual group-membership removal remained pending while Microsoft Entra processed the dynamic membership changes asynchronously.

## Step 5: Recover the Licence

Daniel's Microsoft Entra ID P2 licence was inherited from:

```text
DG-All-Teachers
```

When Microsoft Entra completes Daniel's removal from this group, the inherited P2 licence should be removed automatically and returned to the available licence pool.

This result will not be marked as passed until actual licence removal is verified.

## Validation

| Test | Expected result | Actual result |
|---|---|---|
| Disable Daniel's account | AccountEnabled changes to false | Passed |
| Revoke active sessions | Refresh tokens are invalidated | Passed |
| Attempt new sign-in | Authentication is blocked | Passed |
| Review audit log | Account change is recorded | Passed |
| Review token property | Token-valid timestamp changes | Passed |
| Validate All Teachers rule | Daniel does not satisfy the rule | Passed |
| Validate Science Teachers rule | Daniel does not satisfy the rule | Passed |
| Remove All Teachers membership | Daniel is removed asynchronously | Pending |
| Remove Science membership | Daniel is removed asynchronously | Pending |
| Remove inherited P2 licence | Licence returns to available pool | Pending |

## Processing Delay

Dynamic membership evaluation and membership processing are separate operations.

Rule validation immediately confirmed that Daniel no longer qualified for the groups. However, the visible group membership had not yet been updated.

This demonstrates an important operational consideration:

- Dynamic-group changes are asynchronous.
- Processing normally completes within several hours.
- Some changes can take more than 24 hours.
- Administrators should monitor the processing status.
- Security-critical access should not rely only on delayed group removal.

The account was already protected through immediate account disabling and session revocation while group and licence processing continued.

## Audit Evidence

The audit logs confirmed:

```text
AccountEnabled
Old value: true
New value: false
```

The audit logs also confirmed a change to:

```text
StsRefreshTokensValidFrom
```

This provided evidence that both account disabling and token revocation were performed.

## Evidence
## Evidence

- [Teacher group membership before offboarding](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/13-Daniel%20groups%20before%20off-boarding.jpeg)
- [Teacher account disabled](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/14-teacher-account-disabled.png.jpeg)
- [Teacher sessions revoked](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/15-teacher-sessions-revoked.png.jpeg)
- [Account-disabled audit evidence](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/16-account-disabled-audit-evidence.png.jpeg)
- [Disabled account failed sign-in log](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/17-disabled-account-failed-sign-in-log.png.jpeg)

## Security Considerations

- The account was disabled before access cleanup completed.
- Active sessions were explicitly revoked.
- Authentication failure was tested.
- Dynamic groups exclude disabled accounts.
- Licence removal is automated through group membership.
- Audit logs provide traceability.
- The user object was retained instead of deleted immediately.
- No password or authentication secret was published.
- Sensitive information was removed from screenshots.

## Current Status

The immediate security controls have been successfully validated.

Dynamic group removal and inherited licence removal remain pending and will be verified after Microsoft Entra finishes processing.
