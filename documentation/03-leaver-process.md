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
- Retain audit evidence
- Verify that the former teacher can no longer authenticate

## Step 1: Disable the Account

Daniel's account was changed from enabled to disabled:

```text
AccountEnabled: true → false
```

The account was disabled instead of immediately deleted so that the identity object and associated audit information could be retained during the offboarding and retention period.

## Step 2: Revoke Active Sessions

Daniel's active sessions were revoked.

Microsoft Entra updated the `StsRefreshTokensValidFrom` property, invalidating refresh tokens issued before the new timestamp.

Account disabling and session revocation were used as separate controls:

- Account disabling prevented new authentication.
- Session revocation invalidated existing refresh tokens and forced reauthentication.

## Step 3: Test the Disabled Account

Before offboarding, Daniel successfully accessed the Microsoft My Account portal.

After account disabling and session revocation, another sign-in attempt was performed.

The sign-in was blocked, confirming that Daniel could no longer authenticate.

## Step 4: Remove Group-Based Access

The dynamic teacher rules included the following condition:

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

After Microsoft Entra completed asynchronous processing:

- Daniel was removed from `DG-All-Teachers`.
- Daniel was removed from `DG-Science-Teachers`.
- No manual membership removal was required.

## Step 5: Recover the Licence

Daniel's Microsoft Entra ID P2 licence was inherited from:

```text
DG-All-Teachers
```

When Microsoft Entra removed Daniel from the group, the inherited P2 licence was automatically removed and returned to the available licence pool.

No direct licence removal was required.

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
- [All Teachers membership removed](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/18-all-teachers-after-offboarding.jpeg.jpeg)
- [Science membership removed](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/19-science-teachers-after-offboarding.jpeg.jpeg)
- [Inherited P2 license removed](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/20-teacher-licence-removed-after-offboarding.jpeg.jpeg)

## Processing Delay

Dynamic-rule validation immediately confirmed that Daniel no longer qualified for the teacher groups.

Visible membership removal took longer because dynamic membership processing is asynchronous.

During the delay:

- Daniel's account remained disabled.
- His active sessions had been revoked.
- Sign-in testing confirmed that access was blocked.
- Rule validation confirmed that he no longer qualified.
- Entra eventually removed the memberships automatically.
- Group-based licence removal then completed automatically.

This demonstrates why security-critical offboarding should use immediate account controls while automated cleanup continues.

## Audit Evidence

The audit logs confirmed:

```text
AccountEnabled
Old value: true
New value: false
```

The audit logs also recorded a change to:

```text
StsRefreshTokensValidFrom
```

This provided evidence that account disabling and session revocation occurred.

## Evidence

- [Teacher group membership before offboarding](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/13-Daniel%20groups%20before%20off-boarding.jpeg)
- [Teacher account disabled](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/14-teacher-account-disabled.png.jpeg)
- [Teacher sessions revoked](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/15-teacher-sessions-revoked.png.jpeg)
- [Account-disabled audit evidence](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/16-account-disabled-audit-evidence.png.jpeg)
- [Disabled account failed sign-in log](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/17-disabled-account-failed-sign-in-log.png.jpeg)
- [All Teachers membership removed](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/18-all-teachers-after-offboarding.jpeg.jpeg)
- [Science membership removed](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/19-science-teachers-after-offboarding.jpeg.jpeg)
- [Inherited P2 licence removed](https://github.com/jambo91/entra-school-identity-lifecycle/blob/main/screenshots/20-teacher-licence-removed-after-offboarding.jpeg)

## Security Considerations

- The account was disabled before automated access cleanup completed.
- Active sessions were explicitly revoked.
- Authentication failure was tested.
- Dynamic groups excluded disabled accounts.
- Licence removal was automated through group membership.
- Audit logs provided administrative traceability.
- The user object was retained instead of immediately deleted.
- No password or authentication secret was published.
- Sensitive information was removed from screenshots.

## Final Status

✅ The teacher leaver process was completed and validated successfully.

Account access, active sessions, dynamic group memberships and the inherited P2 licence were all removed as intended.
